---
layout: home
title: proxy-shopping
---

**proxy-shopping** は、暗号通貨（いまは BTC signet と USDC）で、現金や特定の電子決済しか受け付けない店の買い物を、
**proxy shopper**（代理購入者）に代わりにしてもらうための P2P 網です。

- お金は注文ごとの **2-of-3 マルチシグ**（利用者・shopper・escrow）に入り、タイムロックで最後は必ず誰かの手に戻ります。
- 利用者は何もインストールせず、公開された Web 画面から参加できます。鍵はブラウザの中にあり、署名もブラウザで行います。
- どの shopper と escrow を信頼するかは、利用者が選んだ **coordinator** の署名から、operator の一覧を通じて決まります。

## ページ

- [仕組み]({{ '/overview/' | relative_url }}): 構成、信頼の流れ、手数料、現金の店、店の危険度
- [役割]({{ '/roles/' | relative_url }}): 利用者・shopper・escrow・operator・coordinator がそれぞれ何をするか
- [使い方]({{ '/quickstart/' | relative_url }}): 手元で網をまるごと動かす、Web 画面で注文する、ノードを立てる
- [プロトコル]({{ '/protocol/' | relative_url }}): メッセージ、スクリプト、Safe、タイムロックの要点

## ソース

レポジトリは非公開です（この文書だけ公開しています）。

| レポジトリ | 中身 |
|---|---|
| proxy-shopping-go | Go ノード、Nostr リレー、コントラクト、架空の店、自動操作ツール、docker compose の検証環境 |
| proxy-shopping-web | ブラウザ用の中核ライブラリと Web 画面 |
