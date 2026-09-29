---
layout: page
title: 协议
permalink: /zh/protocol/
lang: zh-Hans
ref: protocol
nav_order: 5
---

{% include langnav.html %}

完整规范见 proxy-shopping-go 中的 `docs/spec.md`。本页概括其要点。

## 密钥

所有密钥都由同一个助记词（BIP39）派生，每种用途一个（均为 secp256k1）。

| 用途 | 路径 |
|---|---|
| 身份（Nostr） | `m/44'/1237'/0'/0/0`（NIP-06） |
| libp2p | `m/7333'/0'/0'` |
| BTC 订单密钥（用户、代购者） | `m/7333'/1'/{idx}'`（idx 由订单 ID 的哈希得出） |
| BTC 订单密钥（托管方） | `m/7333'/2'/{idx}`。xpub 是公开的，因此即使托管方离线，其他人也能派生出它 |
| BTC 钱包 | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## 信任

| kind | 签名者 | 内容 |
|---|---|---|
| 30500 | 协调者（coordinator） | 对运营者的委托；用 `revoked` 撤销 |
| 30501 | 运营者（operator） | 地区 × 代购者 × 托管方列表、推荐的中继和链端点 |
| 30502 / 30503 | 代购者（shopper） / 托管方（escrow） | 资料（费用、现金地区、xpub、收款地址） |
| 10050 | 所有人 | 自己接收消息所用的中继 |

由 `v` 标签的版本决定哪个更新（通常是签名时的 UNIX 时间）。没有任何东西会过期。
事件同时通过 Nostr 中继和 libp2p gossipsub 传播；Go 节点会把从一方收到的内容转发到另一方。

## 消息

消息用 NIP-59 封装（gift wrap → seal → 内容）。
内容是一个**已签名**的事件（kind 5400），这样托管方在纠纷中可以作为第三方验证它。

- 一条消息至少发往接收者收件中继中的 k 个（默认 2 个）。
- 接收者用 `ack` 回复；发送者会一直重发，直到收到回复。
- 一条消息最多 28000 字节。截图等大型证据只以哈希引用；
  在纠纷中会以 `attachment` 消息分块发给托管方。

```
order.request + order.escrow_key → order.quote → order.accept → (funding) → order.funded + escrow.notice
→ order.purchased → order.shipping → order.release → order.completed
cancel and refund: order.cancel (before funding), order.refund (cooperative refund from the shopper)
other: ack (receipt), chat
dispute: dispute.open → dispute.evidence_request → dispute.evidence (+ attachment) → dispute.ruling → dispute.countersigned
report: report (to the operator)
```

## BTC（P2WSH）

```
OP_IF
  OP_2 <user> <shopper> <escrow> OP_3 OP_CHECKMULTISIG
OP_ELSE
  OP_IF   <T1> OP_CHECKLOCKTIMEVERIFY OP_DROP <shopper> OP_CHECKSIG
  OP_ELSE <T2> OP_CHECKLOCKTIMEVERIFY OP_DROP <user> OP_CHECKSIG
  OP_ENDIF
OP_ENDIF
```

T1 < T2（区块高度）。如果用户一直不确认收货且 T1 已过，代购者可以单独取走资金。
如果代购者消失，用户在 T2 之后可以单独取回资金。
托管方必须在 T1 之前对纠纷作出裁决。

## USDC（Safe v1.4.1）

- 每笔订单对应一个有三个所有者、阈值为二的 Safe。
- 它的地址来自 CREATE2，因此可以根据订单 ID 预先计算出来。
- 时间锁以 Safe 模块（`PSEscrowModule`）的形式实现。
  - t1 之后，代购者可以用 `claimByShopper` 取走资金。
  - t2 之后，用户可以用 `refundToUser` 取回资金。
- 付款和裁决都是用 EIP-712 签名的 SafeTx。裁决的分配使用 `MultiSendCallOnly`。

## 把身份绑定到密钥

每个订单请求（`order.request`）都带有一个 `key_proof`：
由进入多签的链上密钥（BTC 订单密钥或 EVM 账户）作出的签名，
证明该密钥与 Nostr 身份属于同一个人。
没有它，另一个身份就可以把用户的公钥复制到自己的请求里，并向托管方声称自己就是该用户。

## 收货地址

地址用一次性密钥 K 加密（XChaCha20-Poly1305）。

- K 分别用 NIP-44 为代购者和托管方各封装一份。
- 托管方的那份不在请求中：它通过 `order.escrow_key` 交给代购者（请求中只带它的哈希）。
  发生纠纷时，由代购者（或用户）把它交给托管方，因此对于没有纠纷的订单，托管方无法读取地址。

## 地区

代码按前缀匹配：`JP` > `JP-13`（东京都）> `JP-13-13104`（新宿区）。
市区町村一级使用日本的地方公共团体代码。
