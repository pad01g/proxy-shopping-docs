---
layout: page
title: Vai trò
permalink: /vi/roles/
lang: vi
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Vai trò | Luôn trực tuyến | Dùng gì | Làm gì | Nếu gian lận |
|---|---|---|---|---|
| **người dùng** (user) | không | trình duyệt | đặt hàng, nạp tiền, xác nhận đã nhận hàng, mở tranh chấp | ― |
| **người mua hộ** (proxy shopper) | **có** | node Go + shopper-bot | báo giá, mua thay người dùng, báo cáo việc giao hàng; nhận phí | mất phần của mình trong phán quyết của bên ký quỹ; nhà vận hành loại khỏi danh sách |
| **bên ký quỹ** (escrow) | không | trình duyệt hoặc node Go | phân xử tranh chấp trước T1 | nhà vận hành loại khỏi danh sách; tùy theo điều khoản, tiền đặt cọc bị tịch thu và dùng để bồi thường |
| **nhà vận hành** (operator) | không | trình duyệt, node Go hoặc `psctl` | ký, theo từng khu vực, danh sách các cặp người mua hộ × bên ký quỹ đáng tin; nhận báo cáo vi phạm | điều phối viên thu hồi ủy quyền |
| **điều phối viên** (coordinator) | không | trình duyệt hoặc `psctl` | ký các ủy quyền cho nhà vận hành | người dùng xóa khỏi phần cài đặt (bãi nhiệm) |

## Người dùng

1. Mở ứng dụng web, tạo một mnemonic (12 từ), ghi lại, và chọn một passphrase để mã hóa khóa.
2. Nhập URL của cửa hàng, khu vực của cửa hàng và các món hàng.
3. Chọn một trong các cặp người mua hộ × bên ký quỹ được đề xuất và đặt hàng.
4. Báo giá được gửi đến. Nếu tỷ giá lệch hơn 3% so với nguồn của chính bạn, bạn sẽ nhận được lưu ý; lệch hơn 10% sẽ là cảnh báo mạnh.
5. Chấp nhận và nạp tiền. Số tiền được chia giữa multisig và phí trả trước cho bên ký quỹ.
6. Khi hàng đến, nhấn "đã nhận" (sau một màn hình hiển thị số tiền và người nhận, chữ ký một phần của bạn được gửi cho người mua hộ).
7. Nếu hàng không đến, hãy mở tranh chấp. Ứng dụng sẽ thu thập bằng chứng giúp bạn.

## Người mua hộ

- Chạy node Go với `role: shopper` và kết nối shopper-bot (công cụ tự động hóa trình duyệt).
- Cấu hình các phương thức thanh toán, loại tiền, những khu vực bạn có thể trả tiền mặt, mức phí và chính sách rủi ro cửa hàng của bạn.
- Thông tin thẻ chỉ nằm bên trong shopper-bot; chúng không bao giờ được giao cho node hay cho mạng.
- Nếu người dùng không bao giờ xác nhận đã nhận hàng và T1 đã qua, bạn có thể một mình lấy số tiền.

## Bên ký quỹ

- Chỉ phân xử tranh chấp của những đơn hàng đã trả phí trước cho mình.
- Bình thường không thể đọc địa chỉ giao hàng. Khi có tranh chấp, người mua hộ (hoặc người dùng) sẽ đưa khóa giải mã cho bên ký quỹ.
- Phán quyết đến tay cả hai bên dưới dạng một giao dịch đã ký; nó trở thành chung cuộc khi một trong hai bên ký xác nhận (countersign).

## Nhà vận hành

- Ký các khu vực của mình (một hoặc nhiều), danh sách các cặp, cùng các relay và endpoint chuỗi mà mình đề xuất.
- Khi nhận báo cáo vi phạm, phát hành phiên bản mới của danh sách không còn kẻ vi phạm. Tiền đặt cọc được xử lý theo điều khoản đã thỏa thuận với bên ký quỹ.

## Điều phối viên

- Cấp ủy quyền cho các nhà vận hành. Thu hồi nghĩa là phát hành một phiên bản mới có `revoked`.

## Được đưa vào danh sách (registry)

Registry tin cậy của người duy trì dự án là repository GitHub
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Một pull request được merge chính là sự
chấp thuận:** sau mỗi lần merge, CI ký các ủy quyền và danh sách mới bằng khóa của registry rồi phát hành chúng
(lên các relay Nostr công khai và lên https://pad01g.github.io/proxy-shopping-registry/events.json).

| Bạn muốn làm | Thêm trong pull request | Việc merge sẽ làm gì |
|---|---|---|
| người mua hộ | `shoppers/<name>.json` (pk, liên hệ, mô tả, khu vực tiền mặt, phương thức thanh toán, các bên ký quỹ bạn làm việc cùng) | nhà vận hành của registry đưa các cặp người mua hộ × bên ký quỹ của bạn vào danh sách |
| bên ký quỹ | `escrows/<name>.json` (pk, liên hệ, mô tả, số ngày SLA) | người mua hộ có thể chỉ định bạn; các cặp của bạn xuất hiện |
| nhà vận hành | `operators/<name>.json` (pk, liên hệ, mô tả, khu vực) | điều phối viên của registry ủy quyền cho bạn; sau đó bạn tự ký danh sách của mình |
| điều phối viên | `coordinators/<name>.json` (pk, liên hệ, mô tả) | bạn xuất hiện trong danh bạ điều phối viên mà các ứng dụng đưa ra cho người dùng (mỗi người dùng vẫn tự chọn tin ai) |

`pk` là khóa công khai Nostr của bạn (64 ký tự hex): ứng dụng web hiển thị nó trong phần Cài đặt, và `psctl keys --mnemonic-file …`
in ra nó. Loại ai đó là một pull request chuyển tệp của họ vào `revoked/` kèm lý do. README của registry
có định dạng tệp chính xác và các lệnh.
