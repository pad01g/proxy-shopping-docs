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

レポジトリは非公開です。アクセス権がある場合は、proxy-shopping-go と proxy-shopping-web を同じ親ディレクトリに並べて置き、次を実行します。

```sh
cd proxy-shopping-go
docker compose up -d --build
```

やり直すときは `docker compose down -v` で、ノードやリレーの保存データごと消してから起動します
（チェーンは起動のたびに初めからになるので、古い注文のデータが残っていると食い違います）。

### e2e を流す

```sh
docker compose run --rm runner         # 全シナリオ（10 分ほど）
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
| e | NAT の内側のブラウザが、公開の Web 画面から注文する（1 対 1 のメッセージは Nostr リレー経由）。NAT の内側の escrow ノードへは、libp2p の circuit relay で状態の問い合わせが届く |
| f | shopper が、危険と判定した店と、地域外の現金店の注文を断る。地域内の shopper は、現金店の注文を受けて届ける |
| g | T1 を過ぎると、shopper が単独で受け取れる（BTC） |
| h | shopper が消えても、T2 を過ぎれば利用者が単独で取り戻せる（USDC） |
| i | 誠実な escrow による USDC の紛争。裁定の前に誰かが Safe に少額を送り付けても、返金を執行できる |
| j | 店が在庫切れで買えなかった注文。shopper が協力的な払い戻しを申し出て、利用者が確かめて受け入れる |
| k | shopper が消えても、T2 を過ぎれば利用者が単独で取り戻せる（BTC） |

### Web 画面を触る

lab の名前（`*.test`）はコンテナの中でしか引けません。
ブラウザで触るときは、次の 2 つをします。

- `edge` コンテナの 443 番を手元に出す（例: `compose.override.yaml` に `services: {edge: {ports: ["127.0.0.1:443:443"]}}`）。
  lab 専用の `faucet.test` と `evm.test`（誰でも残高を作ったり時刻を進めたりできる）も一緒に見えるので、手元だけに出すこと。
- hosts ファイルで `app.test` などを 127.0.0.1 に向ける。

証明書は自己署名です。

1. `https://app.test/` を開き、「新しく作る」か「復元用の単語を入れる」を選ぶ。
   鍵を暗号化するパスフレーズ（8 文字以上）を決める（暗号化しないことを明示的に選ぶこともできる。NIP-07 の拡張も使える）。
   次に開いたときは、パスフレーズで鍵を開く。
2. 「注文する」で、店の URL（例: `https://safe-shop.test/`）、店の地域（例: `JP-13-13104`）、商品（例: `A-100`）を入れて候補を探す。
3. shopper と escrow の組み合わせを選び、届け先を入れて注文する。
4. 見積が届いたら、レートの差と多重署名のアドレスの検証結果を見て承諾する。
5. lab では「蛇口から受け取る」で残高を入れてから、「多重署名に入金する」を押す。
   入金・支払い・連署など資金を動かす操作は、金額と宛先を示す確認の画面を通る。
6. 届いたら「受け取った」を押すと、shopper に支払われる。「完了」は、チェーン上で支払いが確認できてから表示される。

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

バージョン（`v`）は既定で UNIX 時刻になるので、ふつうは指定しません。

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator の公開鍵> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```
