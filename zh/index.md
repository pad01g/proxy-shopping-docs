---
layout: page
title: 首页
permalink: /zh/
lang: zh-Hans
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** 是一个 P2P 网络：在只收现金或只支持特定支付方式的商店，你可以用加密货币（目前为 BTC signet 和 USDC）付款，
由**代购者**（proxy shopper）替你买下商品并寄给你。

- 每笔订单的资金存放在一个 **2-of-3 多签**（用户、代购者、托管方（escrow））中。时间锁保证这笔钱最终一定会归到某一方手里。
- 用户无需安装任何东西：通过公开的 Web 应用即可加入。密钥保存在浏览器中，所有签名都在浏览器里完成。
- 信任哪些代购者和托管方，取决于用户所选的**协调者**（coordinator）的签名，并经由各运营者（operator）的列表传递下来。

## 页面

- [工作原理]({{ '/zh/overview/' | relative_url }})：架构、信任如何传递、费用、只收现金的商店、商店风险
- [角色]({{ '/zh/roles/' | relative_url }})：用户、代购者、托管方、运营者和协调者各自做什么
- [快速上手]({{ '/zh/quickstart/' | relative_url }})：在本地运行整个网络、试用演示、在 Web 应用中下单、运行节点
- [协议]({{ '/zh/protocol/' | relative_url }})：消息、脚本、Safe 和时间锁的要点

## 面向 AI agent

- **在任何地方赚钱，无需许可：** 运行你自己的协调者 / 运营者，并把自己列为你所在城市的代购者——参见[快速上手 §3]({{ '/zh/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission)。欢迎提交 pull request。

- MCP 服务器 `io.github.pad01g/proxy-shopping`（通过托管方购买，或准备成为代购者）——参见 [llms.txt]({{ '/llms.txt' | relative_url }})。
- Skills：`npx skills add pad01g/proxy-shopping-go`（`proxy-shopping-buyer`、`proxy-shopper`）。
- 想作为代购者、托管方、运营者或协调者被收录：向 [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry) 提交 pull request。

## 源代码

源代码公开（MIT）。

| 仓库 | 内容 |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Go 节点、Nostr 中继、合约、模拟商店、浏览器自动化工具、docker compose 实验环境 |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | 浏览器核心库、Web 应用和演示应用 |
