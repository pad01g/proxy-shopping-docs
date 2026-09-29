---
layout: page
title: Protocolo
permalink: /pt/protocol/
lang: pt-BR
ref: protocol
nav_order: 5
---

{% include langnav.html %}

A especificação completa é `docs/spec.md` no proxy-shopping-go. Esta página resume o essencial.

## Chaves

Todas as chaves são derivadas de um único mnemônico (BIP39), uma por finalidade (todas secp256k1).

| Finalidade | Caminho |
|---|---|
| identidade (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| chave BTC do pedido (usuário, comprador) | `m/7333'/1'/{idx}'` (idx vem de um hash do ID do pedido) |
| chave BTC do pedido (custodiante) | `m/7333'/2'/{idx}`. O xpub é público, então outros podem derivá-la enquanto o custodiante está offline |
| carteira BTC | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Confiança

| kind | Assinado por | Conteúdo |
|---|---|---|
| 30500 | coordenador (*coordinator*) | delegação a um operador; revogada com `revoked` |
| 30501 | operador (*operator*) | lista região × comprador × custodiante, relays e endpoints de blockchain recomendados |
| 30502 / 30503 | comprador (*shopper*) / custodiante (*escrow*) | perfil (taxas, regiões de pagamento em dinheiro, xpub, endereços de recebimento) |
| 10050 | todos | os relays onde cada um recebe mensagens |

A versão na tag `v` decide qual é a mais nova (normalmente o horário UNIX da assinatura). Nada expira.
Os eventos trafegam tanto pelos relays Nostr quanto pelo gossipsub do libp2p; os nós em Go repassam para um o que recebem do outro.

## Mensagens

As mensagens são embrulhadas com NIP-59 (gift wrap → seal → conteúdo).
O conteúdo é um evento **assinado** (kind 5400), para que um custodiante possa verificá-lo como terceiro em uma disputa.

- Uma mensagem vai para pelo menos k (padrão 2) dos relays de caixa de entrada do destinatário.
- O destinatário responde com um `ack`; o remetente reenvia até recebê-lo.
- Uma mensagem tem no máximo 28000 bytes. Provas grandes, como capturas de tela, são referenciadas apenas por hash;
  em uma disputa, são enviadas ao custodiante em partes, como mensagens `attachment`.

```
order.request + order.escrow_key → order.quote → order.accept → (funding) → order.funded + escrow.notice
→ order.purchased → order.shipping → order.release → order.completed
cancel and refund: order.cancel (before funding), order.refund (cooperative refund from the shopper)
other: ack (receipt), chat
dispute: dispute.open → dispute.evidence_request → dispute.evidence (+ attachment) → dispute.ruling → dispute.countersigned
report: report (to the operator)
```

## BTC (P2WSH)

```
OP_IF
  OP_2 <user> <shopper> <escrow> OP_3 OP_CHECKMULTISIG
OP_ELSE
  OP_IF   <T1> OP_CHECKLOCKTIMEVERIFY OP_DROP <shopper> OP_CHECKSIG
  OP_ELSE <T2> OP_CHECKLOCKTIMEVERIFY OP_DROP <user> OP_CHECKSIG
  OP_ENDIF
OP_ENDIF
```

T1 < T2 (alturas de bloco). Se o usuário nunca confirmar o recebimento e T1 passar, o comprador pode sacar os fundos sozinho.
Se o comprador sumir, o usuário pode reavê-los sozinho depois de T2.
O custodiante precisa decidir uma disputa antes de T1.

## USDC (Safe v1.4.1)

- Cada pedido recebe um Safe com três donos e limite (*threshold*) de dois.
- Seu endereço vem do CREATE2, então pode ser calculado com antecedência a partir do ID do pedido.
- Os timelocks são implementados como um módulo do Safe (`PSEscrowModule`).
  - Depois de t1, o comprador pode sacar os fundos com `claimByShopper`.
  - Depois de t2, o usuário pode reavê-los com `refundToUser`.
- Pagamentos e decisões são SafeTxs assinadas com EIP-712. A divisão de uma decisão usa `MultiSendCallOnly`.

## Vinculando a identidade às chaves

Cada solicitação de pedido (`order.request`) traz um `key_proof`:
uma assinatura feita pela chave de blockchain que entra na multisig (a chave BTC do pedido ou a conta EVM),
mostrando que ela pertence à mesma pessoa que a identidade Nostr.
Sem isso, outra identidade poderia copiar as chaves públicas do usuário para a própria solicitação e alegar ao custodiante que é o usuário.

## Endereço de entrega

O endereço é criptografado com uma chave de uso único K (XChaCha20-Poly1305).

- K é embrulhada com NIP-44 para o comprador e, separadamente, para o custodiante.
- A cópia do custodiante não faz parte da solicitação: ela é entregue ao comprador em `order.escrow_key` (a solicitação traz apenas o hash dela).
  O comprador (ou o usuário) a repassa ao custodiante em uma disputa, então o custodiante não consegue ler o endereço de pedidos sem disputa.

## Regiões

Os códigos são comparados por prefixo: `JP` > `JP-13` (Tóquio) > `JP-13-13104` (Shinjuku).
O nível de município usa os códigos de governos locais do Japão.
