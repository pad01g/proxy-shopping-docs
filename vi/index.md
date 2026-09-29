---
layout: page
title: Trang chủ
permalink: /vi/
lang: vi
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** là một mạng P2P cho phép bạn trả bằng tiền mã hóa (hiện tại là BTC signet và USDC)
tại những cửa hàng chỉ nhận tiền mặt hoặc một số phương thức thanh toán nhất định: một **người mua hộ** (proxy shopper) sẽ mua món hàng thay bạn và gửi đến cho bạn.

- Tiền của mỗi đơn hàng nằm trong một **multisig 2-of-3** (người dùng, người mua hộ, bên ký quỹ (escrow)). Timelock bảo đảm tiền cuối cùng luôn về tay một ai đó.
- Người dùng không cần cài gì cả: họ tham gia qua một ứng dụng web công khai. Khóa nằm trong trình duyệt, và mọi chữ ký đều được tạo ở đó.
- Việc tin tưởng người mua hộ và bên ký quỹ nào được suy ra từ chữ ký của một **điều phối viên** (coordinator) do người dùng chọn, thông qua danh sách của các nhà vận hành (operator).

## Các trang

- [Cách hoạt động]({{ '/vi/overview/' | relative_url }}): kiến trúc, cách lòng tin được truyền đi, phí, cửa hàng chỉ nhận tiền mặt, rủi ro của cửa hàng
- [Vai trò]({{ '/vi/roles/' | relative_url }}): người dùng, người mua hộ, bên ký quỹ, nhà vận hành và điều phối viên mỗi bên làm gì
- [Bắt đầu]({{ '/vi/quickstart/' | relative_url }}): chạy toàn bộ mạng trên máy của bạn, thử bản demo, đặt hàng trong ứng dụng web, chạy một node
- [Giao thức]({{ '/vi/protocol/' | relative_url }}): những điểm cốt lõi về thông điệp, script, Safe và timelock

## Dành cho AI agent

- **Kiếm tiền ở bất cứ đâu, không cần xin phép:** tự chạy điều phối viên (coordinator) / nhà vận hành (operator) của riêng bạn và đưa chính mình vào danh sách làm người mua hộ ở thành phố của bạn — xem [Bắt đầu §3]({{ '/vi/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Pull request luôn được hoan nghênh.

- MCP server `io.github.pad01g/proxy-shopping` (mua hàng qua bên ký quỹ, hoặc chuẩn bị trở thành người mua hộ) — xem [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skill: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Được đưa vào danh sách với vai trò người mua hộ, bên ký quỹ, nhà vận hành hoặc điều phối viên: gửi một pull request đến [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Mã nguồn

Mã nguồn được công khai (MIT).

| Repository | Nội dung |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Node Go, relay Nostr, hợp đồng, cửa hàng giả lập, công cụ tự động hóa trình duyệt, môi trường lab docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | Thư viện lõi cho trình duyệt, ứng dụng web và ứng dụng demo |
