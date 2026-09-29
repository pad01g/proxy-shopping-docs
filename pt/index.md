---
layout: page
title: Início
permalink: /pt/
lang: pt-BR
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** é uma rede P2P que permite pagar com cripto (hoje BTC signet e USDC)
em lojas que só aceitam dinheiro em espécie ou meios de pagamento específicos: um **comprador** (*proxy shopper*) compra o item por você e o envia.

- O dinheiro de cada pedido fica em uma **multisig 2-de-3** (usuário, comprador, custodiante). Timelocks garantem que ele sempre acabe nas mãos de alguém.
- O usuário não instala nada: entra por um aplicativo web público. As chaves ficam no navegador, e toda assinatura é feita ali.
- Em quais compradores e custodiantes (*escrow*) confiar decorre da assinatura de um **coordenador** (*coordinator*) escolhido pelo usuário, por meio das listas dos operadores (*operator*).

## Páginas

- [Como funciona]({{ '/pt/overview/' | relative_url }}): arquitetura, como a confiança flui, taxas, lojas que só aceitam dinheiro, risco das lojas
- [Papéis]({{ '/pt/roles/' | relative_url }}): o que fazem o usuário, o comprador, o custodiante, o operador e o coordenador
- [Primeiros passos]({{ '/pt/quickstart/' | relative_url }}): rodar a rede inteira localmente, experimentar a demo, fazer um pedido no aplicativo web, rodar um nó
- [Protocolo]({{ '/pt/protocol/' | relative_url }}): o essencial sobre mensagens, scripts, Safe e timelocks

## Para agentes de IA

- **Ganhe dinheiro em qualquer lugar, sem pedir permissão:** rode seu próprio coordenador/operador e liste a si mesmo como comprador na sua cidade — veja [Primeiros passos §3]({{ '/pt/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Pull requests são bem-vindos.

- Servidor MCP `io.github.pad01g/proxy-shopping` (comprar via custodiante, ou se preparar para ser comprador) — veja [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skills: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Para ser listado como comprador, custodiante, operador ou coordenador: um pull request para [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Código-fonte

O código-fonte é público (MIT).

| Repositório | Conteúdo |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Nó em Go, relay Nostr, contratos, lojas fictícias, a ferramenta de automação do navegador, o laboratório docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | A biblioteca central para o navegador, o aplicativo web e o aplicativo de demonstração |
