---
layout: page
title: プロトコル
permalink: /protocol/
---

完全な仕様は proxy-shopping-go の `docs/spec.md` にあります。ここでは要点だけをまとめます。

## 鍵

1 つのニーモニック（BIP39）から、用途ごとの鍵を導出します（すべて secp256k1）。

| 用途 | パス |
|---|---|
| 身元（Nostr） | `m/44'/1237'/0'/0/0`（NIP-06） |
| libp2p | `m/7333'/0'/0'` |
| BTC の注文鍵（利用者・shopper） | `m/7333'/1'/{idx}'`（idx は注文 ID のハッシュから決まる） |
| BTC の注文鍵（escrow） | `m/7333'/2'/{idx}`。xpub を公開するので、escrow がオフラインでも相手が導出できる |
| BTC の財布 | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## 信頼

| kind | 署名者 | 内容 |
|---|---|---|
| 30500 | coordinator | operator への委任書。`revoked` で失効させる |
| 30501 | operator | 地域 × shopper × escrow の一覧、推奨するリレーとチェーンの接続先 |
| 30502 / 30503 | shopper / escrow | プロフィール（手数料、現金で行ける地域、xpub、受け取りアドレス） |
| 10050 | 各主体 | 自分宛てのメッセージを受け取るリレー |

新旧は `v` タグのバージョンで決めます。有効期限は持ちません。
配布は Nostr リレーと libp2p の gossipsub の両方で行い、Go ノードは一方で受け取ったものをもう一方へ中継します。

## メッセージ

NIP-59（gift wrap → seal → 中身）で包みます。
中身は **署名付き** のイベント（kind 5400）にします。紛争のとき、escrow が第三者として検証できるようにするためです。

- 受信箱のリレーのうち、k 個（既定 2）以上に送ります。
- 受け手は `ack` を返し、送り手は ack が来るまで再送します。
- 1 通は 30000 byte までです。スクリーンショットのような大きい証拠は、ハッシュだけをメッセージに載せます。
  紛争のときに、`attachment` として分けて escrow に送ります。

```
order.request → order.quote → order.accept → （入金）→ order.funded + escrow.notice
→ order.purchased → order.shipping → order.release → order.completed
紛争: dispute.open → dispute.evidence_request → dispute.evidence (+ attachment) → dispute.ruling → dispute.countersigned
通報: report（operator 宛て）
```

## BTC（P2WSH）

```
OP_IF
  OP_2 <利用者> <shopper> <escrow> OP_3 OP_CHECKMULTISIG
OP_ELSE
  OP_IF   <T1> OP_CHECKLOCKTIMEVERIFY OP_DROP <shopper> OP_CHECKSIG
  OP_ELSE <T2> OP_CHECKLOCKTIMEVERIFY OP_DROP <利用者> OP_CHECKSIG
  OP_ENDIF
OP_ENDIF
```

T1 < T2（ブロック高）です。利用者が受け取り確認をしないまま T1 を過ぎると、shopper が単独で受け取れます。
shopper が消えたときは、T2 を過ぎると利用者が単独で取り戻せます。
escrow は、紛争を T1 より前に裁定する義務を負います。

## USDC（Safe v1.4.1）

- 注文ごとに、所有者 3 人・しきい値 2 の Safe を作ります。
- アドレスは CREATE2 で決まるので、注文 ID から事前に計算できます。
- タイムロックは Safe のモジュール（`PSEscrowModule`）で実装しています。
  - t1 以降は、shopper が `claimByShopper` で受け取れます。
  - t2 以降は、利用者が `refundToUser` で取り戻せます。
- 支払いと裁定は EIP-712 で署名した SafeTx です。裁定の分割には `MultiSendCallOnly` を使います。

## 住所

届け先は使い捨ての鍵 K で暗号化します（XChaCha20-Poly1305）。

- K は、shopper 宛てと escrow 宛てに、それぞれ NIP-44 で包みます。
- escrow 宛てのものは、紛争のときに shopper（または利用者）が escrow に渡します。

## 地域

`JP` > `JP-13`（東京都）> `JP-13-13104`（新宿区）のように、前方一致で照合します。
市区町村の段は、全国地方公共団体コードを使います。
