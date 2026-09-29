---
layout: page
title: Home
permalink: /en/
lang: en
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** is a P2P network that lets you pay with crypto (currently BTC signet and USDC)
at shops that only accept cash or specific payment methods: a **proxy shopper** buys the item for you and ships it.

- The money for each order sits in a **2-of-3 multisig** (user, shopper, escrow). Timelocks make sure it always ends up with someone.
- Users install nothing: they join through a public web app. Keys stay in the browser, and every signature is made there.
- Which shoppers and escrows to trust follows from the signature of a **coordinator** the user chooses, through the operators' lists.

## Pages

- [How it works]({{ '/en/overview/' | relative_url }}): architecture, how trust flows, fees, cash-only shops, shop risk
- [Roles]({{ '/en/roles/' | relative_url }}): what the user, shopper, escrow, operator and coordinator each do
- [Getting started]({{ '/en/quickstart/' | relative_url }}): run the whole network locally, try the demo, order in the web app, run a node
- [Protocol]({{ '/en/protocol/' | relative_url }}): the essentials of messages, scripts, Safe and timelocks

## For AI agents

- MCP server `io.github.pad01g/proxy-shopping` (buy through the escrow, or plan to become a shopper) — see [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skills: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Get listed as a shopper, escrow, operator or coordinator: a pull request to [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Source

The source code is public (MIT).

| Repository | Contents |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Go node, Nostr relay, contracts, fake shops, the browser automation tool, the docker compose lab |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | The browser core library, the web app and the demo app |
