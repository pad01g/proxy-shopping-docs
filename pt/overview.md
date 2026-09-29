---
layout: page
title: Como funciona
permalink: /pt/overview/
lang: pt-BR
ref: overview
nav_order: 2
---

{% include langnav.html %}

## O que ele faz

Quando a loja onde você quer comprar só aceita dinheiro em espécie ou um meio de pagamento específico, você paga com cripto
(hoje **BTC signet** e **USDC**) e um **comprador** (*proxy shopper*) compra o item por você e o envia.

O dinheiro vai para uma **multisig 2-de-3** por pedido (usuário, comprador e custodiante, o *escrow*).

- Quando o item chega, o usuário e o comprador assinam para pagar o comprador.
- Se houver uma disputa, o custodiante decide a divisão junto com uma das outras duas partes.
- Mesmo que alguém pare de responder, **timelocks** garantem que o dinheiro acabe com um dos lados.

## Arquitetura

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **O usuário não roda nada.** Basta abrir o aplicativo web público para participar.
  As chaves são criadas no navegador e nunca saem dele; toda autorização (assinatura) acontece no navegador.
- Usuários atrás de NAT também conseguem se conectar, porque a conexão com um relay Nostr é de saída.
- Mensagens para quem não está sempre online (usuários, custodiantes, operadores, coordenadores) ficam aguardando,
  ainda criptografadas, em relays Nostr que funcionam como caixas de correio. Cada mensagem vai para vários relays.
- Só o comprador precisa estar sempre online. Ele roda um nó em Go e uma ferramenta de automação do navegador (shopper-bot) que opera as lojas.

## Como a confiança flui

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- O usuário confia **apenas na chave pública do coordenador** (*coordinator*). A partir dela, a confiança se estende aos operadores (*operator*) e às combinações comprador × custodiante.
- As listas têm versão e nunca expiram. Remover alguém significa publicar uma nova versão.
- O usuário "dispensa" um coordenador removendo-o das suas configurações.

## Taxas

| Para quem | Exigível? | Como |
|---|---|---|
| comprador | sim | incluída na cotação |
| custodiante (antecipada) | sim | paga diretamente ao custodiante junto com o financiamento. **O custodiante não tem obrigação de arbitrar pedidos sem a taxa antecipada** |
| custodiante (disputa) | sim | retirada da divisão da decisão |
| operador / coordenador | não | combinada fora do protocolo, por exemplo taxas de listagem. O que uma lista vende é "ser encontrado" |

A caução do custodiante e seu confisco não fazem parte do protocolo.
São tratados como termos que o operador e o custodiante combinam entre si; há um contrato de referência disponível.

## Lojas que só aceitam dinheiro

A localização de uma loja é um código de região (por exemplo `JP-13-13104` = Shinjuku, Tóquio).
O comprador declara as regiões onde pode ir e pagar em dinheiro, e só aceita pedidos de lojas nessas regiões.

## Risco das lojas

Antes de aceitar um pedido, o comprador dá uma nota para a loja:
ela está na lista de permissões (*allowlist*) do comprador, usa HTTPS, e o checkout é de um gateway de pagamento conhecido?
Enviar dados de cartão para um site qualquer é arriscado, então pedidos de lojas com nota baixa são recusados.
