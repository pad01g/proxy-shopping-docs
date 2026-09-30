---
layout: page
title: Bắt đầu
permalink: /vi/quickstart/
lang: vi
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Chạy toàn bộ mạng trên máy của bạn (lab)

Một tệp docker compose duy nhất dựng lên một mạng khép kín. Nó không kết nối ra internet.

- hai relay Nostr
- một relay libp2p
- bitcoind trên một signet tùy chỉnh, và một API tương thích Esplora
- anvil (EVM) với Safe và các hợp đồng khác
- bốn cửa hàng giả lập và một cổng thanh toán thẻ giả lập
- hai người mua hộ (shopper), ba bên ký quỹ (escrow) (một bên đứng sau NAT), hai nhà vận hành (operator)
- ứng dụng web công khai và ứng dụng demo

Yêu cầu: Docker (khoảng 8 GB bộ nhớ).

Đặt proxy-shopping-go và proxy-shopping-web cạnh nhau trong cùng một thư mục cha rồi chạy:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Để bắt đầu lại từ đầu, hãy chạy `docker compose down -v` trước để xóa luôn dữ liệu đã lưu của các node và relay
(các chuỗi khởi động lại từ con số không mỗi lần chạy, nên dữ liệu đơn hàng còn sót lại sẽ không khớp với chúng).

### Chạy kiểm thử e2e

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Kết quả được ghi vào `e2e/results/e2e-latest.md`. Runner chạy từ bên trong NAT (mạng `home`),
nên mọi kịch bản đều giả định người dùng đứng sau NAT.

| id | Kịch bản |
|---|---|
| a | Luồng suôn sẻ: BTC (một cửa hàng JPY) và USDC (một cửa hàng USD). Tỷ giá trong báo giá được kiểm tra, bên ký quỹ nhận phí trả trước, người mua hộ được trả tiền. Báo giá có tỷ giá lệch xa sẽ nhận cảnh báo mạnh |
| b | Giao hàng thất bại và người dùng mở tranh chấp. Một bên ký quỹ trung thực phán quyết hoàn tiền và người dùng ký xác nhận (BTC). Bên ký quỹ giải mã được địa chỉ và nhận ảnh chụp màn hình dưới dạng tệp đính kèm |
| c | Một bên ký quỹ phán quyết không trung thực. Người dùng báo cáo; nhà vận hành loại nó khỏi danh sách và, theo điều khoản của họ, tịch thu tiền đặt cọc của nó để bồi thường (USDC) |
| d | Một điều phối viên (coordinator) thu hồi ủy quyền của một nhà vận hành, và các cặp trong danh sách đó biến mất khỏi các đề xuất |
| e | Một trình duyệt đứng sau NAT đặt hàng qua ứng dụng web công khai (thông điệp 1:1 đi qua relay Nostr). Một truy vấn trạng thái đến được node của bên ký quỹ đứng sau NAT qua circuit relay của libp2p |
| f | Một người mua hộ từ chối một cửa hàng mà họ chấm là rủi ro, và một cửa hàng chỉ nhận tiền mặt nằm ngoài khu vực của họ. Một người mua hộ trong khu vực nhận đơn tiền mặt đó và giao hàng |
| g | Sau T1, người mua hộ có thể một mình lấy số tiền (BTC) |
| h | Nếu người mua hộ biến mất, người dùng một mình lấy lại tiền sau T2 (USDC) |
| i | Tranh chấp USDC với một bên ký quỹ trung thực. Việc hoàn tiền vẫn được thực hiện ngay cả khi ai đó gửi một khoản nhỏ vào Safe trước khi có phán quyết |
| j | Một đơn hàng mà cửa hàng không đáp ứng được (hết hàng). Người mua hộ đề nghị hoàn tiền hợp tác; người dùng kiểm tra và chấp nhận |
| k | Nếu người mua hộ biến mất, người dùng một mình lấy lại tiền sau T2 (BTC) |

### Thử toàn bộ luồng trong bản demo

**Dùng thử ngay trong trình duyệt: <https://pad01g.github.io/proxy-shopping-web/> (mọi thứ được mô phỏng trong trang).**

Khi lab đang chạy, mở bản demo tại `http://localhost:8888/` (không cần tệp hosts hay chứng chỉ).
Trên cùng một màn hình, người dùng, bên ký quỹ, nhà vận hành và điều phối viên mỗi bên có khóa riêng (trong bộ nhớ cục bộ của trình duyệt),
còn người mua hộ chính là node Go luôn trực tuyến. Các đường đi là thật: relay Nostr, bitcoind, anvil, node Go.

- Chọn một kịch bản ở trên cùng. Phần hướng dẫn bên trái cho biết ai làm gì tiếp theo, và vì sao.
- "Go to this step" chuyển sang tab của vai trò đó và chỉ vào nút cần nhấn. Các biểu mẫu đã được điền sẵn theo kịch bản, nên bạn chỉ cần nhấn và xác nhận.
- Các kịch bản: luồng suôn sẻ (BTC / USDC), giao hàng thất bại và hoàn tiền, hết hàng và hoàn tiền hợp tác, từ chối một cửa hàng rủi ro, một bên ký quỹ không trung thực bị báo cáo và loại khỏi danh sách, và hoàn tiền sau T2 khi người mua hộ biến mất.
- Bạn cũng có thể mở mỗi vai trò trong một cửa sổ riêng, ví dụ `?role=user` và `?role=escrow,operator,coordinator` (các cửa sổ của cùng một trình duyệt dùng chung khóa và tiến độ).
- Trong thực tế, mỗi vai trò ở một nơi khác nhau, trong trình duyệt riêng của mình. Bản demo chỉ tách riêng các khóa.
- Chỉ dành cho lab: bất kỳ ai truy cập được cổng 8888 đều có thể dùng faucet, đào khối, tua thời gian và API quản trị của người mua hộ (nó chỉ được bind vào 127.0.0.1).

Cũng có một bài e2e điều khiển bản demo bằng cách làm theo phần hướng dẫn của nó: `docker compose run --rm runner demo`.

### Dùng ứng dụng web

Các tên miền của lab (`*.test`) chỉ phân giải được bên trong các container.
Để dùng ứng dụng web từ trình duyệt của bạn:

- Mở cổng 443 của container `edge` ra máy cục bộ (ví dụ trong `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Việc này cũng mở ra `faucet.test` và `evm.test` vốn chỉ dành cho lab (ai cũng có thể tạo số dư và tua thời gian), nên chỉ mở trên localhost.
- Trỏ `app.test` và các tên khác về 127.0.0.1 trong tệp hosts của bạn.

Các chứng chỉ là tự ký.

1. Mở `https://app.test/` và chọn "create new" hoặc "restore from words".
   Chọn một passphrase (từ 8 ký tự trở lên) để mã hóa khóa (bạn cũng có thể chủ động chọn không mã hóa, hoặc dùng một tiện ích mở rộng NIP-07).
   Lần sau, mở khóa bằng passphrase đó.
2. Trong "order", nhập URL cửa hàng (ví dụ `https://safe-shop.test/`), khu vực của cửa hàng (ví dụ `JP-13-13104`) và món hàng (ví dụ `A-100`), rồi tìm các đề xuất.
3. Chọn một cặp người mua hộ × bên ký quỹ, nhập địa chỉ giao hàng và đặt hàng.
4. Khi báo giá đến, kiểm tra độ lệch tỷ giá và kết quả kiểm tra địa chỉ multisig, rồi chấp nhận.
5. Trong lab, nạp tiền vào ví bằng "get from the faucet", rồi nhấn "fund the multisig".
   Mọi thao tác chuyển tiền (nạp, trả, ký xác nhận) đều đi qua một màn hình hiển thị số tiền và người nhận.
6. Khi hàng đến, nhấn "received" để trả tiền cho người mua hộ. "Completed" chỉ hiện ra sau khi giao dịch thanh toán được xác nhận trên chuỗi.

## 2. Chạy trên mạng công khai

Kiến trúc giống như lab. Những điểm khác:

- dùng chứng chỉ ACME;
- không dùng các khóa trong `lab/keys`;
- dùng signet thật và một chuỗi EVM thật.

### Người mua hộ

Luôn trực tuyến; chạy node Go và shopper-bot.

```yaml
role: shopper
name: my-shopper
network: ps-main
mnemonic_file: /keys/shopper.mnemonic
nostr: {relays: ["wss://relay.example"], k: 2}
trust: {coordinators: ["<coordinator public key>"]}
chain:
  btc: {network: signet, esplora: "https://mempool.space/signet/api"}
shopper:
  bot_url: "http://shopper-bot:7000"
  cash_regions: [JP-13]
  risk: {allowlist: [shop.example], known_gateways: [pay.example], threshold: 70}
```

- Thông tin thẻ chỉ nằm trong tệp cấu hình của shopper-bot (`BOT_CARDS_FILE`); node không bao giờ nhận được chúng.
- Các bước cho từng cửa hàng được viết thành một driver của shopper-bot. Một driver dùng AI để thao tác cửa hàng cũng có cùng đầu vào và đầu ra (`PurchaseRequest` / `PurchaseResult`).

### Bên ký quỹ / nhà vận hành / điều phối viên

Những vai trò này không cần trực tuyến liên tục.

- Họ có thể dùng các màn hình "Escrow", "Operator" và "Coordinator" của ứng dụng web.
- Để chạy liên tục, hãy chạy node Go với `role: escrow` / `role: operator`.
- Nếu chỉ để ký, `psctl` cũng dùng được.

Phiên bản (`v`) mặc định là thời gian UNIX, nên thông thường bạn không cần truyền vào.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Tự vận hành vai trò của bạn trong mạng, ở bất cứ đâu, không cần xin phép

proxy-shopping không có nhà vận hành trung tâm và không có gì phải đăng ký. Chuỗi lòng tin chỉ là các khóa và các sự kiện Nostr đã ký,
nên bạn (hoặc một AI agent làm việc cho bạn) có thể mở một khu chợ địa phương ngay tại thành phố của mình:

1. **Trở thành điều phối viên:** tạo một khóa (`psctl keys --mnemonic-file coordinator.mnemonic`). Điều phối viên chỉ có vậy.
2. **Ủy quyền cho một nhà vận hành** (khóa thứ hai của chính bạn, hoặc một người bạn tin tưởng):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Đưa người mua hộ và bên ký quỹ trong khu vực của bạn vào danh sách** — chính bạn làm người mua hộ, một người bạn làm bên ký quỹ:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Nói với mọi người hãy tin khóa điều phối viên của bạn** (thêm nó trong phần Cài đặt của ứng dụng web, hoặc `PS_COORDINATORS` cho MCP server).
   Để ai cũng tìm thấy bạn, hãy mở một pull request đến [registry](https://github.com/pad01g/proxy-shopping-registry)
   thêm `coordinators/<name>.json`; để tự vận hành việc chấp thuận cho cộng đồng của riêng bạn theo cùng cách, hãy fork registry.

**Chạy ở đâu:** một máy tính ở nhà là đủ. Node chỉ tạo các kết nối đi ra (relay Nostr, và circuit relay của libp2p
khi nó đứng sau NAT), nên bạn không cần mở cổng nào. Dùng [Tailscale](https://tailscale.com/) hoặc bất kỳ VPN nào để truy cập
API quản trị của node và ứng dụng web từ điện thoại khi bạn ra ngoài; một VPS nhỏ cũng dùng được.

**AI agent có thể làm gì:** một agent có thể đảm nhận công việc thường ngày của nhà vận hành và người mua hộ — theo dõi đơn hàng qua API quản trị
của node hoặc MCP server, cập nhật danh sách, điều khiển shopper-bot cho các cửa hàng nhận thẻ, báo cáo sự cố — còn hiểu biết
địa phương của chính bạn (cửa hàng nào, khu vực nào, những cửa hàng chỉ nhận tiền mặt mà bạn có thể đi bộ tới) là phần không ai khác có thể mang lại.
Skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) hướng dẫn agent từng bước.

Hãy trung thực với người dùng của bạn: mạng công khai còn mới và chạy trên BTC signet (tiền thử nghiệm), nên thu nhập chỉ đến khi có người sử dụng.

## Đóng góp {#contributing}

**Pull request luôn được hoan nghênh** — ở mọi repository: driver cửa hàng cho shopper-bot, phương thức thanh toán và chuỗi mới,
bản dịch của tài liệu này, đánh giá giao thức, sửa lỗi, và đăng ký vào
[registry](https://github.com/pad01g/proxy-shopping-registry). Hãy mở một issue hoặc pull request trên GitHub.
