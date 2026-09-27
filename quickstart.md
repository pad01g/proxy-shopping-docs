---
layout: page
title: 使い方
permalink: /quickstart/
---

## 1. 手元で網をまるごと動かす（lab）

docker compose ひとつで、閉じた網を立てられます。インターネットには接続しません。

- Nostr リレー 2 台
- libp2p の relay
- 独自 signet の bitcoind と Esplora 互換 API
- anvil（EVM）と Safe などのコントラクト
- 架空の店 4 つと、仮想のカード決済ゲートウェイ
- shopper 2 人、escrow 3 人（うち 1 人は NAT の内側）、operator 2 人
- 公開用の Web 画面

必要なもの: Docker（メモリ 8 GB 程度）。

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web   # proxy-shopping-go の隣に置く
cd proxy-shopping-go
docker compose up -d --build
```

### e2e を流す

```sh
docker compose run --rm runner         # 全シナリオ（10 分弱）
docker compose run --rm runner a e     # 選んで流す
```

結果は `e2e/results/e2e-latest.md` に出ます。runner は NAT の内側（`home` ネットワーク）から動くので、
利用者は NAT の内側にいる前提で試されます。

| id | シナリオ |
|---|---|
| a | 正常系。BTC（JPY の店）と USDC（USD の店）。見積のレートの検証、escrow の前払い手数料、shopper への支払い。レートが大きくずれた見積に強い警告が出ること |
| b | 配達に失敗し、紛争を申し立てる。誠実な escrow が返金を裁定し、利用者が連署する（BTC）。escrow が住所を復号でき、スクリーンショットを添付として受け取ること |
| c | escrow が不正な裁定をする。利用者が通報し、operator が一覧から外して、規約の bond を没収し補償する（USDC） |
| d | coordinator が operator の委任を失効させ、その一覧の組み合わせが候補から消える |
| e | NAT の内側のブラウザが、公開の Web 画面から注文する。NAT の内側の escrow には、libp2p の relay 経由で届く |
| f | shopper が、危険と判定した店と、地域外の現金店の注文を断る。地域内の shopper は、現金店の注文を受けて届ける |
| g | T1 を過ぎると、shopper が単独で受け取れる（BTC） |
| h | shopper が消えても、T2 を過ぎれば利用者が単独で取り戻せる（USDC） |

### Web 画面を触る

lab の名前（`*.test`）はコンテナの中でしか引けません。
ブラウザで触るときは、次の 2 つをします。

- `edge` コンテナの 443 番を手元に出す。
- hosts ファイルで `app.test` などを 127.0.0.1 に向ける。

証明書は自己署名です。

1. `https://app.test/` を開き、「ニーモニックを作る」か「取り込む」を選ぶ。
2. 「注文する」で、店の URL（例: `https://safe-shop.test/`）、店の地域（例: `JP-13-13104`）、商品（例: `A-100`）を入れて候補を探す。
3. shopper と escrow の組み合わせを選び、届け先を入れて注文する。
4. 見積が届いたら、レートの差と多重署名のアドレスの検証結果を見て承諾する。
5. lab では「蛇口から受け取る」で残高を入れてから、「多重署名に入金する」を押す。
6. 届いたら「受け取った」を押すと、shopper に支払われる。

## 2. 公開網で動かす

構成は lab と同じです。違いは次のとおりです。

- 証明書を ACME のものにする。
- `lab/keys` の鍵は使わない。
- チェーンを本物の signet と EVM にする。

### shopper

常時オンラインで、Go ノードと shopper-bot を動かします。

```yaml
role: shopper
name: my-shopper
network: ps-main
mnemonic_file: /keys/shopper.mnemonic
nostr: {relays: ["wss://relay.example"], k: 2}
trust: {coordinators: ["<coordinator の公開鍵>"]}
chain:
  btc: {network: signet, esplora: "https://mempool.space/signet/api"}
shopper:
  bot_url: "http://shopper-bot:7000"
  cash_regions: [JP-13]
  risk: {allowlist: [shop.example], known_gateways: [pay.example], threshold: 70}
```

- カード情報は shopper-bot の設定ファイル（`BOT_CARDS_FILE`）にだけ書き、ノードには渡しません。
- 店ごとの操作は shopper-bot の driver として書きます。AI で操作する driver に替えるときも、入出力（`PurchaseRequest` / `PurchaseResult`）は同じです。

### escrow / operator / coordinator

常時オンラインでなくて構いません。

- Web 画面の「エスクロー」「オペレータ」「コーディネータ」から操作できます。
- 常時動かしたい場合は、Go ノードを `role: escrow` / `role: operator` で動かします。
- 署名だけなら `psctl` でもできます。

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator の公開鍵> --version 1 --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --version 1 --publish wss://relay.example
```
