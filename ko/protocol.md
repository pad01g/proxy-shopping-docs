---
layout: page
title: 프로토콜
permalink: /ko/protocol/
lang: ko
ref: protocol
nav_order: 5
---

{% include langnav.html %}

전체 명세는 proxy-shopping-go의 `docs/spec.md`입니다. 이 페이지는 그 핵심을 요약합니다.

## 키

모든 키는 하나의 니모닉(BIP39)에서 용도별로 하나씩 파생됩니다(모두 secp256k1).

| 용도 | 경로 |
|---|---|
| 신원 (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| BTC 주문 키 (사용자, 구매 대행자) | `m/7333'/1'/{idx}'` (idx는 주문 ID의 해시에서 나옴) |
| BTC 주문 키 (에스크로) | `m/7333'/2'/{idx}`. xpub이 공개되어 있으므로, 에스크로가 오프라인이어도 다른 사람이 파생할 수 있음 |
| BTC 지갑 | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## 신뢰

| kind | 서명자 | 내용 |
|---|---|---|
| 30500 | 코디네이터(coordinator) | 운영자에 대한 위임; `revoked`로 철회 |
| 30501 | 운영자(operator) | 지역 × 구매 대행자 × 에스크로 목록, 추천 릴레이와 체인 엔드포인트 |
| 30502 / 30503 | 구매 대행자(shopper) / 에스크로(escrow) | 프로필 (수수료, 현금 지역, xpub, 지급 주소) |
| 10050 | 모두 | 자신이 메시지를 받는 릴레이 |

어느 쪽이 더 새로운지는 `v` 태그의 버전으로 정합니다(보통 서명 시점의 UNIX 시간). 만료되는 것은 없습니다.
이벤트는 Nostr 릴레이와 libp2p gossipsub 양쪽으로 전달되며, Go 노드는 한쪽에서 받은 것을 다른 쪽으로 전달합니다.

## 메시지

메시지는 NIP-59로 감쌉니다(gift wrap → seal → 내용).
내용은 **서명된** 이벤트(kind 5400)이므로, 분쟁 시 에스크로가 제3자로서 이를 검증할 수 있습니다.

- 메시지는 받는 사람의 수신 릴레이 중 최소 k개(기본값 2)로 보내집니다.
- 받는 사람은 `ack`로 응답하고, 보낸 사람은 응답을 받을 때까지 다시 보냅니다.
- 메시지는 최대 28000바이트입니다. 스크린샷 같은 큰 증거는 해시로만 참조하고,
  분쟁 시 `attachment` 메시지로 조각내어 에스크로에게 보냅니다.

```
order.request + order.escrow_key → order.quote → order.accept → (funding) → order.funded + escrow.notice
→ order.purchased → order.shipping → order.release → order.completed
cancel and refund: order.cancel (before funding), order.refund (cooperative refund from the shopper)
other: ack (receipt), chat
dispute: dispute.open → dispute.evidence_request → dispute.evidence (+ attachment) → dispute.ruling → dispute.countersigned
report: report (to the operator)
```

## BTC (P2WSH)

```
OP_IF
  OP_2 <user> <shopper> <escrow> OP_3 OP_CHECKMULTISIG
OP_ELSE
  OP_IF   <T1> OP_CHECKLOCKTIMEVERIFY OP_DROP <shopper> OP_CHECKSIG
  OP_ELSE <T2> OP_CHECKLOCKTIMEVERIFY OP_DROP <user> OP_CHECKSIG
  OP_ENDIF
OP_ENDIF
```

T1 < T2 (블록 높이). 사용자가 수령을 확인하지 않고 T1이 지나면, 구매 대행자가 혼자서 자금을 가져갈 수 있습니다.
구매 대행자가 사라지면, 사용자는 T2 이후 혼자서 자금을 되찾을 수 있습니다.
에스크로는 T1 이전에 분쟁을 판정해야 합니다.

## USDC (Safe v1.4.1)

- 주문마다 소유자 3명, 임계값 2인 Safe가 만들어집니다.
- 주소는 CREATE2로 정해지므로, 주문 ID로부터 미리 계산할 수 있습니다.
- 타임락은 Safe 모듈(`PSEscrowModule`)로 구현됩니다.
  - t1 이후 구매 대행자는 `claimByShopper`로 자금을 가져갈 수 있습니다.
  - t2 이후 사용자는 `refundToUser`로 자금을 되찾을 수 있습니다.
- 지급과 판정은 EIP-712로 서명된 SafeTx입니다. 판정의 분배에는 `MultiSendCallOnly`를 사용합니다.

## 신원과 키의 연결

각 주문 요청(`order.request`)에는 `key_proof`가 들어 있습니다.
멀티시그에 들어가는 체인 키(BTC 주문 키 또는 EVM 계정)로 만든 서명으로,
그 키가 Nostr 신원과 같은 사람의 것임을 보여 줍니다.
이것이 없으면 다른 신원이 사용자의 공개 키를 자기 요청에 복사해 넣고, 에스크로에게 자신이 그 사용자라고 주장할 수 있습니다.

## 배송 주소

주소는 일회용 키 K로 암호화됩니다(XChaCha20-Poly1305).

- K는 NIP-44로 구매 대행자용과, 별도로 에스크로용으로 각각 감쌉니다.
- 에스크로용 사본은 요청에 포함되지 않습니다. `order.escrow_key`로 구매 대행자에게 전달됩니다(요청에는 그 해시만 들어 있습니다).
  분쟁 시 구매 대행자(또는 사용자)가 이를 에스크로에게 넘기므로, 분쟁이 없는 주문의 주소는 에스크로가 읽을 수 없습니다.

## 지역

코드는 접두사로 비교합니다: `JP` > `JP-13` (도쿄도) > `JP-13-13104` (신주쿠구).
시구정촌 단위에는 일본의 지방자치단체 코드를 사용합니다.
