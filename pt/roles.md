---
layout: page
title: Papéis
permalink: /pt/roles/
lang: pt-BR
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Papel | Sempre online | Usa | Faz | Se trapacear |
|---|---|---|---|---|
| **usuário** (*user*) | não | navegador | faz pedidos, financia, confirma o recebimento, abre disputas | ― |
| **comprador** (*proxy shopper*) | **sim** | nó em Go + shopper-bot | cota, compra em nome do usuário, informa o envio; ganha uma taxa | perde sua parte na decisão do custodiante; o operador o remove da lista |
| **custodiante** (*escrow*) | não | navegador ou nó em Go | decide disputas antes de T1 | o operador o remove da lista; conforme os termos combinados, sua caução é confiscada e usada como compensação |
| **operador** (*operator*) | não | navegador, nó em Go ou `psctl` | assina, por região, a lista de combinações confiáveis comprador × custodiante; recebe denúncias | o coordenador revoga sua delegação |
| **coordenador** (*coordinator*) | não | navegador ou `psctl` | assina delegações para operadores | os usuários o removem das configurações (dispensa) |

## Usuário

1. Abra o aplicativo web, crie um mnemônico (12 palavras), anote-o e escolha uma senha que criptografa a chave.
2. Informe a URL da loja, a região da loja e os itens.
3. Escolha uma das combinações comprador × custodiante oferecidas e faça o pedido.
4. Chega uma cotação. Se a taxa de câmbio diferir das suas próprias fontes em mais de 3%, você recebe um aviso; acima de 10%, um alerta forte.
5. Aceite e financie. O dinheiro é dividido entre a multisig e a taxa antecipada do custodiante.
6. Quando o item chegar, pressione "recebido" (depois de uma tela que mostra o valor e o destinatário, sua assinatura parcial vai para o comprador).
7. Se não chegar, abra uma disputa. O aplicativo reúne as provas para você.

## Comprador

- Rode o nó em Go com `role: shopper` e conecte o shopper-bot (a ferramenta de automação do navegador).
- Configure os meios de pagamento, as moedas, as regiões onde você pode pagar em dinheiro, sua taxa e sua política de risco das lojas.
- Os dados de cartão ficam apenas dentro do shopper-bot; nunca são entregues ao nó nem à rede.
- Se o usuário nunca confirmar o recebimento e T1 passar, você pode sacar os fundos sozinho.

## Custodiante

- Decide apenas disputas de pedidos que pagaram sua taxa antecipada.
- Normalmente não consegue ler o endereço de entrega. Em uma disputa, o comprador (ou o usuário) lhe entrega a chave de decriptação.
- A decisão chega às duas partes como uma transação assinada; ela se torna definitiva quando uma delas contra-assina.

## Operador

- Assina suas regiões (uma ou mais), a lista de combinações e os relays e endpoints de blockchain que recomenda.
- Ao receber uma denúncia, publica uma nova versão da lista sem o infrator. A caução segue os termos combinados com o custodiante.

## Coordenador

- Emite delegações para operadores. Revogar significa publicar uma nova versão com `revoked`.

## Como ser listado (o registro)

O registro de confiança do mantenedor é o repositório GitHub
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Um pull request aceito (merge) é a
aprovação:** após cada merge, a CI assina as novas delegações e listas com as chaves do registro e as publica
(nos relays Nostr públicos e em https://pad01g.github.io/proxy-shopping-registry/events.json).

| Você quer ser | Adicione em um pull request | O que o merge faz |
|---|---|---|
| comprador | `shoppers/<name>.json` (pk, contato, descrição, regiões de pagamento em dinheiro, pagamentos, os custodiantes com quem você trabalha) | o operador do registro lista suas combinações comprador × custodiante |
| custodiante | `escrows/<name>.json` (pk, contato, descrição, prazo de SLA em dias) | compradores podem indicar você; suas combinações aparecem |
| operador | `operators/<name>.json` (pk, contato, descrição, regiões) | o coordenador do registro delega a você; depois você assina suas próprias listas |
| coordenador | `coordinators/<name>.json` (pk, contato, descrição) | você aparece no diretório de coordenadores que os aplicativos oferecem aos usuários (cada usuário ainda escolhe em quem confiar) |

`pk` é sua chave pública Nostr (64 caracteres hex): o aplicativo web a mostra em Configurações, e `psctl keys --mnemonic-file …`
a imprime. Remover alguém é um pull request que move o arquivo dessa pessoa para `revoked/` com um motivo. O README
do registro traz os formatos exatos dos arquivos e os comandos.
