---
layout: page
title: Primeiros passos
permalink: /pt/quickstart/
lang: pt-BR
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Rodar a rede inteira localmente (laboratório)

Um único arquivo docker compose sobe uma rede fechada. Ela não se conecta à internet.

- dois relays Nostr
- um relay libp2p
- bitcoind em um signet próprio, e uma API compatível com Esplora
- anvil (EVM) com o Safe e os demais contratos
- quatro lojas fictícias e um gateway fictício de pagamento com cartão
- dois compradores (*shopper*), três custodiantes (*escrow*, um deles atrás de NAT), dois operadores (*operator*)
- o aplicativo web público e o aplicativo de demonstração

Requisitos: Docker (cerca de 8 GB de memória).

Coloque proxy-shopping-go e proxy-shopping-web lado a lado no mesmo diretório pai e execute:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Para recomeçar do zero, execute antes `docker compose down -v` para apagar também os dados armazenados pelos nós e relays
(as blockchains começam do zero a cada inicialização, então dados de pedidos que sobrarem ficariam inconsistentes com elas).

### Rodar os testes e2e

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Os resultados vão para `e2e/results/e2e-latest.md`. O runner funciona de dentro do NAT (a rede `home`),
então todos os cenários supõem que o usuário está atrás de NAT.

| id | Cenário |
|---|---|
| a | Caminho feliz: BTC (uma loja em JPY) e USDC (uma loja em USD). A taxa de câmbio da cotação é verificada, o custodiante recebe sua taxa antecipada, o comprador é pago. Uma cotação com taxa muito fora recebe um alerta forte |
| b | A entrega falha e o usuário abre uma disputa. Um custodiante honesto decide pelo reembolso e o usuário contra-assina (BTC). O custodiante consegue decriptar o endereço e recebe a captura de tela como anexo |
| c | Um custodiante decide de forma desonesta. O usuário o denuncia; o operador o remove da lista e, conforme os termos combinados, confisca sua caução como compensação (USDC) |
| d | Um coordenador (*coordinator*) revoga a delegação de um operador, e as combinações daquela lista desaparecem das ofertas |
| e | Um navegador atrás de NAT faz um pedido pelo aplicativo web público (mensagens 1:1 passam pelos relays Nostr). Uma consulta de status chega ao nó do custodiante atrás de NAT pelo circuit relay do libp2p |
| f | Um comprador recusa uma loja que ele avalia como arriscada, e uma loja que só aceita dinheiro fora da sua região. Um comprador da região aceita o pedido em dinheiro e entrega |
| g | Depois de T1, o comprador pode sacar os fundos sozinho (BTC) |
| h | Se o comprador sumir, o usuário reavê os fundos sozinho depois de T2 (USDC) |
| i | Disputa em USDC com um custodiante honesto. O reembolso é executado mesmo que alguém envie um valor pequeno para o Safe antes da decisão |
| j | Um pedido que a loja não conseguiu atender (esgotado). O comprador oferece um reembolso cooperativo; o usuário confere e aceita |
| k | Se o comprador sumir, o usuário reavê os fundos sozinho depois de T2 (BTC) |

### Experimentar o fluxo completo na demo

**Experimente no seu navegador: <https://pad01g.github.io/proxy-shopping-web/> (tudo simulado dentro da página).**

Com o laboratório rodando, abra a demo em `http://localhost:8888/` (não precisa de arquivo hosts nem de certificados).
Em uma única tela, o usuário, o custodiante, o operador e o coordenador têm cada um sua própria chave (no armazenamento local do navegador),
e o comprador é o próprio nó em Go sempre online. Os caminhos são reais: relays Nostr, bitcoind, anvil, nós em Go.

- Escolha um cenário no topo. O guia à esquerda diz quem faz o quê em seguida, e por quê.
- "Ir para esta etapa" muda para a aba daquele papel e aponta o botão a ser pressionado. Os formulários vêm preenchidos para o cenário, então você só clica e confirma.
- Cenários: caminho feliz (BTC / USDC), entrega com falha e reembolso, esgotado e reembolso cooperativo, loja arriscada recusada, custodiante desonesto denunciado e removido da lista, e reembolso depois de T2 quando o comprador some.
- Você também pode abrir uma janela por papel, por exemplo `?role=user` e `?role=escrow,operator,coordinator` (janelas do mesmo navegador compartilham as chaves e o progresso).
- Na realidade, cada papel está em outro lugar, no seu próprio navegador. A demo separa apenas as chaves.
- Somente no laboratório: qualquer pessoa que alcance a porta 8888 pode usar o faucet, a mineração, o avanço no tempo e a API de administração do comprador (ela escuta apenas em 127.0.0.1).

Há também um e2e que conduz a demo seguindo o guia: `docker compose run --rm runner demo`.

### Usar o aplicativo web

Os nomes do laboratório (`*.test`) só resolvem dentro dos contêineres.
Para usar o aplicativo web a partir do seu navegador:

- Exponha localmente a porta 443 do contêiner `edge` (por exemplo, em `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Isso também expõe `faucet.test` e `evm.test`, exclusivos do laboratório (qualquer um pode criar saldo e avançar o tempo), então exponha só em localhost.
- Aponte `app.test` e os outros nomes para 127.0.0.1 no seu arquivo hosts.

Os certificados são autoassinados.

1. Abra `https://app.test/` e escolha "criar nova" ou "restaurar a partir das palavras".
   Escolha uma senha (8 caracteres ou mais) que criptografa a chave (você também pode escolher explicitamente não criptografá-la, ou usar uma extensão NIP-07).
   Da próxima vez, desbloqueie a chave com a senha.
2. Em "pedido", informe a URL da loja (por exemplo `https://safe-shop.test/`), a região da loja (por exemplo `JP-13-13104`) e o item (por exemplo `A-100`), e busque ofertas.
3. Escolha uma combinação comprador × custodiante, informe o endereço de entrega e faça o pedido.
4. Quando a cotação chegar, confira a diferença de taxa de câmbio e o resultado da verificação do endereço da multisig, e então aceite.
5. No laboratório, abasteça sua carteira com "obter do faucet" e depois pressione "financiar a multisig".
   Toda ação que movimenta dinheiro (financiar, pagar, contra-assinar) passa por uma tela que mostra o valor e o destinatário.
6. Quando o item chegar, pressione "recebido" para pagar o comprador. "Concluído" só aparece depois que o pagamento é confirmado na blockchain.

## 2. Rodar em uma rede pública

A arquitetura é a mesma do laboratório. As diferenças:

- use certificados ACME;
- não use as chaves de `lab/keys`;
- use um signet real e uma blockchain EVM real.

### Comprador

Sempre online; roda o nó em Go e o shopper-bot.

```yaml
role: shopper
name: my-shopper
network: ps-main
mnemonic_file: /keys/shopper.mnemonic
nostr: {relays: ["wss://relay.example"], k: 2}
trust: {coordinators: ["<coordinator public key>"]}
chain:
  btc: {network: signet, esplora: "https://mempool.space/signet/api"}
shopper:
  bot_url: "http://shopper-bot:7000"
  cash_regions: [JP-13]
  risk: {allowlist: [shop.example], known_gateways: [pay.example], threshold: 70}
```

- Os dados de cartão ficam apenas no arquivo de configuração do shopper-bot (`BOT_CARDS_FILE`); o nó nunca os recebe.
- Os passos de cada loja são escritos como um driver do shopper-bot. Um driver que usa IA para operar as lojas tem a mesma entrada e saída (`PurchaseRequest` / `PurchaseResult`).

### Custodiante / operador / coordenador

Estes não precisam estar sempre online.

- Podem usar as telas "Escrow", "Operator" e "Coordinator" do aplicativo web.
- Para funcionar continuamente, rode o nó em Go com `role: escrow` / `role: operator`.
- Só para assinar, o `psctl` também serve.

A versão (`v`) tem como padrão o horário UNIX, então normalmente você não a informa.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Rode seu próprio papel na rede, em qualquer lugar, sem pedir permissão

O proxy-shopping não tem operador central nem cadastro. A cadeia de confiança é feita só de chaves e eventos Nostr assinados,
então você (ou um agente de IA trabalhando para você) pode abrir um mercado local na sua própria cidade:

1. **Seja um coordenador:** crie uma chave (`psctl keys --mnemonic-file coordinator.mnemonic`). Um coordenador é só isso.
2. **Delegue a um operador** (uma segunda chave sua, ou alguém de confiança):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Liste compradores e custodiantes para a sua região** — você mesmo como comprador, um amigo como custodiante:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Peça às pessoas que confiem na chave do seu coordenador** (adicionando-a nas Configurações do aplicativo web, ou em `PS_COORDINATORS` no servidor MCP).
   Para ser encontrado por todos, abra um pull request no [registro](https://github.com/pad01g/proxy-shopping-registry)
   adicionando `coordinators/<name>.json`; para conduzir aprovações da mesma forma para a sua própria comunidade, faça um fork do registro.

**Onde rodar:** uma máquina em casa basta. O nó só faz conexões de saída (relays Nostr e, quando está atrás de NAT, um
circuit relay do libp2p), então você não abre nenhuma porta. Use o [Tailscale](https://tailscale.com/) ou qualquer VPN para acessar
a API de administração do seu nó e o aplicativo web pelo celular quando estiver fora; um VPS pequeno também serve.

**O que um agente de IA pode fazer:** um agente pode cuidar da rotina do operador e do comprador — acompanhar pedidos pela API de administração
do nó ou pelo servidor MCP, manter as listas atualizadas, conduzir o shopper-bot nas lojas que aceitam cartão, relatar problemas — e o seu
conhecimento local (quais lojas, quais regiões, quais lojas que só aceitam dinheiro ficam a uma caminhada de você) é a parte que ninguém mais pode oferecer.
A skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) guia um agente por esse processo.

Seja honesto com seus usuários: a rede pública é nova e roda em BTC signet (moedas de teste), então os ganhos só vêm quando as pessoas começarem a usá-la.

## Contribuindo {#contributing}

**Pull requests são bem-vindos** — em todos os repositórios: drivers de lojas para o shopper-bot, novos meios de pagamento e blockchains,
traduções desta documentação, revisões do protocolo, correções de bugs e listagens no
[registro](https://github.com/pad01g/proxy-shopping-registry). Abra uma issue ou um pull request no GitHub.
