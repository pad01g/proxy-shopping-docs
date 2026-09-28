---
layout: page
title: 仕組み
permalink: /jp/overview/
lang: ja
ref: overview
nav_order: 2
redirect_from: /overview/
---

{% include langnav.html %}

## 何ができるか

買いたい店が現金や特定の電子決済しか受け付けないとき、暗号通貨（いまは **BTC signet** と **USDC**）で
**proxy shopper**（代理購入者）に代わりに買ってもらい、届けてもらう。

お金は注文ごとの **2-of-3 マルチシグ**（利用者・shopper・escrow）に入れる。

- 届いたら、利用者と shopper の署名で shopper に払う。
- 揉めたら、escrow がもう一方と組んで配分を決める。
- 誰かが応答しなくなっても、**タイムロック**で最後は必ずどちらかに戻る。

## 構成

```
ブラウザ（proxy-shopping-web）             Go ノード（proxy-shopping-go）
  鍵はブラウザ内・署名もブラウザ       shopper / escrow / operator / relay
        │ WSS（外向きのみ）                      │ WSS           │ libp2p
        ▼                                         ▼               ▼
   Nostr リレー群（オペレータが一覧で指定） ◀──▶  Go ノード同士の網
   = 常時オンラインの口 + メールボックス          （gossipsub、NAT 越えは circuit relay v2 と DCUtR）
```

- **利用者は何も立てなくてよい。** 公開されている Web 画面を開くだけで網に参加できる。
  鍵はブラウザの中で作られ、外に出ない。操作の許可（署名）はすべてブラウザで行う。
- NAT の内側からでも、Nostr リレーへは外向きの接続なのでつながる。
- 常時オンラインでない人（利用者・escrow・operator・coordinator）宛てのメッセージは、
  暗号化したまま Nostr リレー（メールボックス）に預ける。複数のリレーへ同時に送る。
- 常時オンラインが必要なのは shopper だけ。shopper は Go ノードと、店を操作する自動操作ツール（shopper-bot）を動かす。

## 信頼の流れ

```
coordinator（利用者が公開鍵を信頼する）
  └─ 委任書: 「この operator に一覧を作らせる」
       └─ 一覧: 「この地域では、この shopper とこの escrow の組み合わせが信頼できる」
```

- 利用者が信頼するのは **coordinator の公開鍵だけ**。そこから operator、shopper と escrow の組み合わせへ間接的に信頼が伸びる。
- 一覧はバージョンで管理し、有効期限は持たない。外すときは新しい版を出す。
- 利用者は、coordinator を設定から外すことで「解任」できる。

## 手数料

| 誰に | 強制できるか | 方法 |
|---|---|---|
| shopper | できる | 見積に含める |
| escrow（前払い） | できる | 入金と同時に escrow へ直接払う。**前払いの無い注文について、escrow は仲裁の義務を負わない** |
| escrow（紛争時） | できる | 裁定の配分から差し引く |
| operator / coordinator | できない | プロトコルの外で、掲載料などを決める。一覧が売っているのは「見つけてもらえること」 |

escrow の預かり金（bond）とその没収は、プロトコルには入れていない。
operator と escrow が独自に結ぶ規約として扱い、参考実装のコントラクトを用意している。

## 現金しか使えない店

店の所在地は地域コードで表す（例: `JP-13-13104` = 東京都新宿区）。
shopper は現金で買いに行ける地域を宣言し、その地域の店の注文だけを受ける。

## 店の危険度

shopper は注文を受ける前に店を点数化する。
点数は、許可リストに載っているか、HTTPS か、既知の決済ゲートウェイか、で決まる。
カード情報を任意のサイトに送る危険があるので、点数が低い店の注文は断る。
