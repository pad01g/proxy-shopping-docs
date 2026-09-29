---
layout: page
title: Giao thức
permalink: /vi/protocol/
lang: vi
ref: protocol
nav_order: 5
---

{% include langnav.html %}

Đặc tả đầy đủ là `docs/spec.md` trong proxy-shopping-go. Trang này tóm tắt những điểm cốt lõi.

## Khóa

Mọi khóa đều được dẫn xuất từ một mnemonic (BIP39), mỗi mục đích một khóa (tất cả đều là secp256k1).

| Mục đích | Đường dẫn |
|---|---|
| danh tính (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| khóa BTC cho đơn hàng (người dùng, người mua hộ) | `m/7333'/1'/{idx}'` (idx lấy từ hash của mã đơn hàng) |
| khóa BTC cho đơn hàng (bên ký quỹ) | `m/7333'/2'/{idx}`. xpub được công khai, nên người khác có thể dẫn xuất nó ngay cả khi bên ký quỹ đang ngoại tuyến |
| ví BTC | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Lòng tin

| kind | Người ký | Nội dung |
|---|---|---|
| 30500 | điều phối viên (coordinator) | ủy quyền cho một nhà vận hành; thu hồi bằng `revoked` |
| 30501 | nhà vận hành (operator) | danh sách khu vực × người mua hộ × bên ký quỹ, các relay và endpoint chuỗi được đề xuất |
| 30502 / 30503 | người mua hộ (shopper) / bên ký quỹ (escrow) | hồ sơ (phí, khu vực tiền mặt, xpub, địa chỉ nhận tiền) |
| 10050 | mọi người | các relay nơi một người nhận thông điệp |

Phiên bản trong thẻ `v` quyết định cái nào mới hơn (thường là thời gian UNIX lúc ký). Không có gì hết hạn.
Sự kiện đi qua cả relay Nostr lẫn libp2p gossipsub; node Go chuyển tiếp những gì nhận được từ bên này sang bên kia.

## Thông điệp

Thông điệp được bọc bằng NIP-59 (gift wrap → seal → nội dung).
Nội dung là một sự kiện **đã ký** (kind 5400), để bên ký quỹ có thể kiểm chứng nó với tư cách bên thứ ba khi có tranh chấp.

- Một thông điệp được gửi đến ít nhất k (mặc định 2) relay hộp thư của người nhận.
- Người nhận trả lời bằng `ack`; người gửi gửi lại cho đến khi nhận được.
- Một thông điệp tối đa 28000 byte. Bằng chứng lớn như ảnh chụp màn hình chỉ được tham chiếu bằng hash;
  khi có tranh chấp, nó được gửi đến bên ký quỹ thành từng phần dưới dạng thông điệp `attachment`.

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

T1 < T2 (chiều cao khối). Nếu người dùng không bao giờ xác nhận đã nhận hàng và T1 đã qua, người mua hộ có thể một mình lấy số tiền.
Nếu người mua hộ biến mất, người dùng có thể một mình lấy lại tiền sau T2.
Bên ký quỹ phải phân xử tranh chấp trước T1.

## USDC (Safe v1.4.1)

- Mỗi đơn hàng có một Safe với ba chủ sở hữu và ngưỡng là hai.
- Địa chỉ của nó đến từ CREATE2, nên có thể tính trước từ mã đơn hàng.
- Timelock được hiện thực dưới dạng một module của Safe (`PSEscrowModule`).
  - Sau t1, người mua hộ có thể lấy tiền bằng `claimByShopper`.
  - Sau t2, người dùng có thể lấy lại tiền bằng `refundToUser`.
- Việc thanh toán và phán quyết là các SafeTx được ký bằng EIP-712. Phần chia của phán quyết dùng `MultiSendCallOnly`.

## Gắn danh tính với các khóa

Mỗi yêu cầu đặt hàng (`order.request`) mang theo một `key_proof`:
một chữ ký bằng khóa chuỗi được đưa vào multisig (khóa BTC cho đơn hàng hoặc tài khoản EVM),
chứng minh rằng nó thuộc về cùng một người với danh tính Nostr.
Nếu không có nó, một danh tính khác có thể chép khóa công khai của người dùng vào yêu cầu của mình và tuyên bố với bên ký quỹ rằng mình là người dùng đó.

## Địa chỉ giao hàng

Địa chỉ được mã hóa bằng một khóa dùng một lần K (XChaCha20-Poly1305).

- K được bọc bằng NIP-44 cho người mua hộ và, riêng biệt, cho bên ký quỹ.
- Bản dành cho bên ký quỹ không nằm trong yêu cầu: nó được giao cho người mua hộ trong `order.escrow_key` (yêu cầu chỉ mang hash của nó).
  Người mua hộ (hoặc người dùng) chuyển nó cho bên ký quỹ khi có tranh chấp, nên bên ký quỹ không thể đọc địa chỉ của những đơn hàng không có tranh chấp.

## Khu vực

Mã được so khớp theo tiền tố: `JP` > `JP-13` (Tokyo) > `JP-13-13104` (Shinjuku).
Cấp đô thị dùng mã chính quyền địa phương của Nhật Bản.
