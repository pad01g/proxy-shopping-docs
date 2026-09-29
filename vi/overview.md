---
layout: page
title: Cách hoạt động
permalink: /vi/overview/
lang: vi
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Nó làm gì

Khi cửa hàng bạn muốn mua chỉ nhận tiền mặt hoặc một phương thức thanh toán nhất định, bạn trả bằng tiền mã hóa
(hiện tại là **BTC signet** và **USDC**) và một **người mua hộ** (proxy shopper) sẽ mua món hàng thay bạn rồi gửi đến cho bạn.

Tiền được chuyển vào một **multisig 2-of-3** riêng cho mỗi đơn hàng (người dùng (user), người mua hộ, bên ký quỹ (escrow)).

- Khi hàng đến nơi, người dùng và người mua hộ cùng ký để trả tiền cho người mua hộ.
- Nếu có tranh chấp, bên ký quỹ quyết định cách chia tiền cùng với một trong hai bên còn lại.
- Ngay cả khi có ai đó ngừng phản hồi, **timelock** bảo đảm tiền cuối cùng sẽ về tay một bên.

## Kiến trúc

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **Người dùng không phải chạy gì cả.** Chỉ cần mở ứng dụng web công khai là tham gia được.
  Khóa được tạo trong trình duyệt và không bao giờ rời khỏi đó; mọi sự cho phép (chữ ký) đều diễn ra trong trình duyệt.
- Người dùng đứng sau NAT cũng kết nối được, vì kết nối đến relay Nostr là kết nối đi ra (outbound).
- Thông điệp gửi cho những người không trực tuyến liên tục (người dùng, bên ký quỹ, nhà vận hành (operator), điều phối viên (coordinator)) sẽ chờ,
  vẫn ở dạng mã hóa, trên các relay Nostr đóng vai trò hộp thư. Mỗi thông điệp được gửi đến nhiều relay.
- Chỉ người mua hộ cần trực tuyến liên tục. Họ chạy một node Go và một công cụ tự động hóa trình duyệt (shopper-bot) để thao tác trên các cửa hàng.

## Lòng tin được truyền đi thế nào

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- Người dùng **chỉ tin khóa công khai của điều phối viên**. Từ đó lòng tin mở rộng đến các nhà vận hành và đến các cặp người mua hộ × bên ký quỹ.
- Danh sách có phiên bản và không bao giờ hết hạn. Muốn loại ai đó thì phát hành một phiên bản mới.
- Người dùng "bãi nhiệm" một điều phối viên bằng cách xóa nó khỏi phần cài đặt của mình.

## Phí

| Trả cho ai | Có cưỡng chế được không? | Cách thức |
|---|---|---|
| người mua hộ | có | đã tính trong báo giá |
| bên ký quỹ (trả trước) | có | trả thẳng cho bên ký quỹ cùng lúc với việc nạp tiền. **Bên ký quỹ không có nghĩa vụ phân xử những đơn hàng không trả phí trước** |
| bên ký quỹ (khi tranh chấp) | có | trích từ phần chia theo phán quyết |
| nhà vận hành / điều phối viên | không | thỏa thuận ngoài giao thức, ví dụ phí niêm yết. Thứ mà một danh sách bán là "được người khác tìm thấy" |

Tiền đặt cọc (bond) của bên ký quỹ và việc tịch thu nó không thuộc giao thức.
Chúng được xem là điều khoản do nhà vận hành và bên ký quỹ tự thỏa thuận với nhau; có sẵn một hợp đồng tham khảo.

## Cửa hàng chỉ nhận tiền mặt

Vị trí của cửa hàng là một mã khu vực (ví dụ `JP-13-13104` = Shinjuku, Tokyo).
Người mua hộ khai báo những khu vực mà họ có thể đến và trả tiền mặt, và chỉ nhận đơn cho các cửa hàng ở đó.

## Rủi ro của cửa hàng

Trước khi nhận đơn, người mua hộ chấm điểm cửa hàng:
cửa hàng có nằm trong danh sách cho phép (allowlist) của người mua hộ không, có dùng HTTPS không, và trang thanh toán có phải là một cổng thanh toán đã biết không.
Gửi thông tin thẻ đến một trang web bất kỳ là rủi ro, nên các đơn cho cửa hàng có điểm thấp sẽ bị từ chối.
