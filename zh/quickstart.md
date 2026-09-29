---
layout: page
title: 快速上手
permalink: /zh/quickstart/
lang: zh-Hans
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. 在本地运行整个网络（实验环境）

一个 docker compose 文件就能启动一个封闭的网络。它不连接互联网。

- 两个 Nostr 中继
- 一个 libp2p 中继
- 运行在自定义 signet 上的 bitcoind，以及一个兼容 Esplora 的 API
- 部署了 Safe 及其他合约的 anvil（EVM）
- 四家模拟商店和一个模拟的银行卡支付网关
- 两个代购者（shopper）、三个托管方（escrow）（其中一个在 NAT 之后）、两个运营者（operator）
- 公开 Web 应用和演示应用

要求：Docker（约 8 GB 内存）。

把 proxy-shopping-go 和 proxy-shopping-web 并排放在同一个父目录下，然后运行：

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

如果要从头开始，请先运行 `docker compose down -v`，把节点和中继保存的数据也一并删除
（每次启动时链都会从零开始，残留的订单数据会与链不一致）。

### 运行 e2e 测试

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

结果写入 `e2e/results/e2e-latest.md`。runner 在 NAT 内部（`home` 网络）运行，
因此所有场景都假定用户位于 NAT 之后。

| id | 场景 |
|---|---|
| a | 正常流程：BTC（日元商店）和 USDC（美元商店）。检查报价的汇率，托管方收到预付费，代购者收到货款。汇率偏差过大的报价会收到强烈警告 |
| b | 配送失败，用户发起纠纷。诚实的托管方裁定退款，用户副署（BTC）。托管方能解密地址，并以附件形式收到截图 |
| c | 托管方作出不诚实的裁决。用户举报；运营者将其从列表中移除，并依照条款没收其保证金作为赔偿（USDC） |
| d | 协调者（coordinator）撤销对某个运营者的委托，该列表中的组合随即从报价中消失 |
| e | NAT 之后的浏览器通过公开 Web 应用下单（1 对 1 消息经由 Nostr 中继传递）。状态查询通过 libp2p circuit relay 到达 NAT 之后的托管方节点 |
| f | 代购者拒绝它判定为有风险的商店，以及其地区之外的只收现金的商店。该地区内的代购者接下这笔现金订单并完成配送 |
| g | T1 之后，代购者可以单独取走资金（BTC） |
| h | 如果代购者消失，用户在 T2 之后单独取回资金（USDC） |
| i | 与诚实托管方的 USDC 纠纷。即使有人在裁决前向 Safe 转入少量金额，退款仍会执行 |
| j | 商店无法履约的订单（售罄）。代购者提出协商退款；用户核对后接受 |
| k | 如果代购者消失，用户在 T2 之后单独取回资金（BTC） |

### 在演示中体验完整流程

实验环境运行后，打开演示页面 `http://localhost:8888/`（无需修改 hosts 文件或配置证书）。
在同一个页面上，用户、托管方、运营者和协调者各自拥有自己的密钥（保存在浏览器的本地存储中），
而代购者就是始终在线的 Go 节点本身。所有路径都是真实的：Nostr 中继、bitcoind、anvil、Go 节点。

- 在顶部选择一个场景。左侧的向导会告诉你下一步由谁做什么，以及为什么。
- "转到这一步"会切换到该角色的标签页，并指向要按的按钮。表单已按场景预先填好，你只需点击并确认。
- 场景包括：正常流程（BTC / USDC）、配送失败与退款、售罄与协商退款、拒绝有风险的商店、举报不诚实的托管方并将其从列表中移除、以及代购者消失后在 T2 之后退款。
- 你也可以为每个角色各开一个窗口，例如 `?role=user` 和 `?role=escrow,operator,coordinator`（同一浏览器的窗口共享密钥和进度）。
- 在现实中，每个角色都在别处、在各自的浏览器里。演示只是把密钥分开而已。
- 仅限实验环境：任何能访问 8888 端口的人都可以使用水龙头、挖矿、时间快进和代购者的管理 API（因此它只绑定在 127.0.0.1 上）。

还有一个按照向导驱动演示的 e2e：`docker compose run --rm runner demo`。

### 使用 Web 应用

实验环境中的域名（`*.test`）只能在容器内部解析。
要在你的浏览器中使用 Web 应用：

- 在本地暴露 `edge` 容器的 443 端口（例如在 `compose.override.yaml` 中：`services: {edge: {ports: ["127.0.0.1:443:443"]}}`）。
  这也会暴露仅限实验环境使用的 `faucet.test` 和 `evm.test`（任何人都可以铸造余额、快进时间），所以只在 localhost 上暴露。
- 在 hosts 文件中把 `app.test` 和其他域名指向 127.0.0.1。

证书是自签名的。

1. 打开 `https://app.test/`，选择"新建"或"从助记词恢复"。
   设置一个用于加密密钥的口令（8 个字符或以上）（你也可以明确选择不加密，或使用 NIP-07 扩展）。
   下次使用时，用口令解锁密钥。
2. 在"下单"中输入商店 URL（例如 `https://safe-shop.test/`）、商店所在地区（例如 `JP-13-13104`）和商品（例如 `A-100`），然后搜索报价。
3. 选择一个代购者 × 托管方组合，填写收货地址并下单。
4. 收到报价后，检查汇率差异和多签地址核验的结果，然后接受。
5. 在实验环境中，用"从水龙头领取"给钱包充值，然后点击"向多签注资"。
   所有动用资金的操作（注资、付款、副署）都会经过一个显示金额和收款方的确认页面。
6. 商品送达后，点击"已收货"向代购者付款。只有在付款于链上确认之后，才会显示"已完成"。

## 2. 在公开网络上运行

架构与实验环境相同。不同之处在于：

- 使用 ACME 证书；
- 不要使用 `lab/keys` 中的密钥；
- 使用真实的 signet 和 EVM 链。

### 代购者

始终在线；运行 Go 节点和 shopper-bot。

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

- 银行卡信息只写在 shopper-bot 的配置文件（`BOT_CARDS_FILE`）中；节点永远拿不到它。
- 每家商店的操作步骤都写成一个 shopper-bot 驱动（driver）。用 AI 操作商店的驱动也具有相同的输入和输出（`PurchaseRequest` / `PurchaseResult`）。

### 托管方 / 运营者 / 协调者

它们不需要始终在线。

- 可以使用 Web 应用中的"托管方"、"运营者"和"协调者"页面。
- 如需持续运行，就以 `role: escrow` / `role: operator` 运行 Go 节点。
- 如果只需签名，也可以使用 `psctl`。

版本（`v`）默认为 UNIX 时间，所以通常不需要传入。

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. 在任何地方运行你自己的网络角色，无需许可

proxy-shopping 没有中央运营方，也无需注册。信任链只由密钥和签名过的 Nostr 事件构成，
所以你（或替你工作的 AI agent）可以在自己的城市开办一个本地市场：

1. **成为协调者：** 生成一个密钥（`psctl keys --mnemonic-file coordinator.mnemonic`）。协调者就是这么简单。
2. **委托给一个运营者**（你自己的第二个密钥，或你信任的人）：
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **为你的地区列出代购者和托管方**——你自己当代购者，朋友当托管方：
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **告诉大家信任你的协调者密钥**（在 Web 应用的"设置"中添加，MCP 服务器则用 `PS_COORDINATORS`）。
   想让所有人都能找到你，就向[登记处](https://github.com/pad01g/proxy-shopping-registry)提交一个 pull request，
   添加 `coordinators/<name>.json`；想用同样的方式为你自己的社区做审批，就 fork 这个登记处。

**在哪里运行：** 家里的一台机器就够了。节点只建立出站连接（Nostr 中继，以及位于 NAT 之后时的 libp2p
circuit relay），所以不需要开放任何端口。外出时，用 [Tailscale](https://tailscale.com/) 或任何 VPN 从手机访问
节点的管理 API 和 Web 应用；小型 VPS 也可以。

**AI agent 能做什么：** agent 可以承担运营者和代购者的日常工作——通过节点的管理 API 或 MCP 服务器
关注订单、保持列表更新、为银行卡商店驱动 shopper-bot、报告问题——而你自己的本地知识
（哪些商店、哪些地区、步行可达的只收现金的店）才是别人无法提供的部分。
skill `proxy-shopper`（`npx skills add pad01g/proxy-shopping-go`）会一步步引导 agent 完成这些。

请对你的用户坦诚：公开网络还很新，运行在 BTC signet（测试币）上，所以要等有人使用之后才会有收入。

## 参与贡献 {#contributing}

**欢迎提交 pull request**——所有仓库都欢迎：shopper-bot 的商店驱动、新的支付方式和链、
本文档的翻译、协议评审、错误修复，以及在[登记处](https://github.com/pad01g/proxy-shopping-registry)中的收录。
请在 GitHub 上开 issue 或提交 pull request。
