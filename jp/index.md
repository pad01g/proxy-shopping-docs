---
layout: page
title: トップ
permalink: /jp/
lang: ja
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** は、暗号通貨（いまは BTC signet と USDC）で、現金や特定の電子決済しか受け付けない店の買い物を、
**proxy shopper**（代理購入者）に代わりにしてもらうための P2P 網です。

- お金は注文ごとの **2-of-3 マルチシグ**（利用者・shopper・escrow）に入り、タイムロックで最後は必ず誰かの手に戻ります。
- 利用者は何もインストールせず、公開された Web 画面から参加できます。鍵はブラウザの中にあり、署名もブラウザで行います。
- どの shopper と escrow を信頼するかは、利用者が選んだ **coordinator** の署名から、operator の一覧を通じて決まります。

## ページ

- [仕組み]({{ '/jp/overview/' | relative_url }}): 構成、信頼の流れ、手数料、現金の店、店の危険度
- [役割]({{ '/jp/roles/' | relative_url }}): 利用者・shopper・escrow・operator・coordinator がそれぞれ何をするか
- [使い方]({{ '/jp/quickstart/' | relative_url }}): 手元で網をまるごと動かす、デモ画面で通しで試す、Web 画面で注文する、ノードを立てる
- [プロトコル]({{ '/jp/protocol/' | relative_url }}): メッセージ、スクリプト、Safe、タイムロックの要点

## AI agent 向け

- MCP サーバー `io.github.pad01g/proxy-shopping`（escrow を通して買う、shopper になる準備をする）。機械向けの概要は [llms.txt]({{ '/llms.txt' | relative_url }})。
- スキル: `npx skills add pad01g/proxy-shopping-go`（`proxy-shopping-buyer`、`proxy-shopper`）。
- shopper・escrow・operator・coordinator として載るには、[proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry) に pull request を出す。マージされたことが承認になる。

## ソース

ソースは公開しています（MIT）。

| レポジトリ | 中身 |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Go ノード、Nostr リレー、コントラクト、架空の店、自動操作ツール、docker compose の検証環境 |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | ブラウザ用の中核ライブラリ、Web 画面、デモ画面 |
