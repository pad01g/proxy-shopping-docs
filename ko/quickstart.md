---
layout: page
title: 시작하기
permalink: /ko/quickstart/
lang: ko
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. 네트워크 전체를 로컬에서 실행하기 (실험 환경)

docker compose 파일 하나로 닫힌 네트워크를 띄웁니다. 인터넷에는 연결되지 않습니다.

- Nostr 릴레이 두 개
- libp2p 릴레이 하나
- 사용자 정의 signet 위의 bitcoind와 Esplora 호환 API
- Safe와 기타 컨트랙트가 배포된 anvil (EVM)
- 가상 상점 네 곳과 가상 카드 결제 게이트웨이
- 구매 대행자(shopper) 둘, 에스크로(escrow) 셋(그중 하나는 NAT 뒤), 운영자(operator) 둘
- 공개 웹 앱과 데모 앱

필요 사항: Docker (메모리 약 8 GB).

proxy-shopping-go와 proxy-shopping-web을 같은 상위 디렉터리에 나란히 두고 다음을 실행합니다.

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

처음부터 다시 시작하려면, 먼저 `docker compose down -v`를 실행해 노드와 릴레이에 저장된 데이터까지 지웁니다
(체인은 시작할 때마다 처음부터 만들어지므로, 남은 주문 데이터가 체인과 맞지 않게 됩니다).

### e2e 테스트 실행

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

결과는 `e2e/results/e2e-latest.md`에 저장됩니다. runner는 NAT 안쪽(`home` 네트워크)에서 동작하므로,
모든 시나리오는 사용자가 NAT 뒤에 있다고 가정합니다.

| id | 시나리오 |
|---|---|
| a | 정상 흐름: BTC(엔화 상점)와 USDC(달러 상점). 견적의 환율을 확인하고, 에스크로는 선불 수수료를, 구매 대행자는 대금을 받습니다. 환율이 크게 어긋난 견적에는 강한 경고가 표시됩니다 |
| b | 배송이 실패해 사용자가 분쟁을 제기합니다. 정직한 에스크로가 환불을 판정하고 사용자가 부서명합니다(BTC). 에스크로는 주소를 복호화할 수 있고 스크린샷을 첨부 파일로 받습니다 |
| c | 에스크로가 부정하게 판정합니다. 사용자가 신고하면 운영자가 목록에서 제외하고, 조건에 따라 보증금을 몰수해 보상에 씁니다(USDC) |
| d | 코디네이터(coordinator)가 운영자의 위임을 철회하면, 그 목록의 조합이 제안에서 사라집니다 |
| e | NAT 뒤의 브라우저가 공개 웹 앱으로 주문합니다(1:1 메시지는 Nostr 릴레이를 거칩니다). 상태 조회는 libp2p circuit relay를 통해 NAT 뒤의 에스크로 노드에 도달합니다 |
| f | 구매 대행자가 위험하다고 판단한 상점과, 자기 지역 밖의 현금 전용 상점을 거절합니다. 해당 지역의 구매 대행자가 현금 주문을 받아 배송합니다 |
| g | T1 이후 구매 대행자는 혼자서 자금을 가져갈 수 있습니다(BTC) |
| h | 구매 대행자가 사라지면, 사용자가 T2 이후 혼자서 자금을 되찾습니다(USDC) |
| i | 정직한 에스크로와의 USDC 분쟁. 판정 전에 누군가 Safe에 소액을 보내더라도 환불은 그대로 실행됩니다 |
| j | 상점이 처리할 수 없는 주문(품절). 구매 대행자가 합의 환불을 제안하고, 사용자가 확인한 뒤 수락합니다 |
| k | 구매 대행자가 사라지면, 사용자가 T2 이후 혼자서 자금을 되찾습니다(BTC) |

### 데모에서 전체 흐름 체험하기

**브라우저에서 바로 써 보기: <https://pad01g.github.io/proxy-shopping-web/> (모든 것이 페이지 안에서 시뮬레이션됩니다)**

실험 환경이 실행 중일 때 `http://localhost:8888/`에서 데모를 엽니다(hosts 파일이나 인증서가 필요 없습니다).
한 화면에서 사용자, 에스크로, 운영자, 코디네이터가 각자 자신의 키(브라우저의 로컬 저장소에 있음)를 가지며,
구매 대행자는 항상 온라인인 Go 노드 그 자체입니다. 경로는 모두 실제입니다: Nostr 릴레이, bitcoind, anvil, Go 노드.

- 위쪽에서 시나리오를 고릅니다. 왼쪽의 가이드가 다음에 누가 무엇을 하는지, 그리고 왜 하는지 알려 줍니다.
- "이 단계로 이동"을 누르면 해당 역할의 탭으로 바뀌고 눌러야 할 버튼을 가리킵니다. 양식은 시나리오에 맞게 미리 채워져 있어 클릭하고 확인하기만 하면 됩니다.
- 시나리오: 정상 흐름(BTC / USDC), 배송 실패와 환불, 품절과 합의 환불, 위험한 상점 거절, 부정한 에스크로의 신고와 목록 제외, 구매 대행자가 사라졌을 때 T2 이후 환불.
- 역할마다 창을 따로 열 수도 있습니다. 예: `?role=user`와 `?role=escrow,operator,coordinator` (같은 브라우저의 창들은 키와 진행 상황을 공유합니다).
- 실제로는 모든 역할이 서로 다른 곳에서 각자의 브라우저를 씁니다. 데모는 키만 분리합니다.
- 실험 환경 전용: 8888 포트에 접근할 수 있는 사람은 누구나 faucet, 채굴, 시간 이동, 구매 대행자의 관리 API를 쓸 수 있습니다(그래서 127.0.0.1에만 바인딩되어 있습니다).

가이드를 따라 데모를 조작하는 e2e도 있습니다: `docker compose run --rm runner demo`.

### 웹 앱 사용하기

실험 환경의 이름(`*.test`)은 컨테이너 안에서만 해석됩니다.
브라우저에서 웹 앱을 쓰려면:

- `edge` 컨테이너의 443 포트를 로컬에 노출합니다(예: `compose.override.yaml`에 `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  이렇게 하면 실험 환경 전용인 `faucet.test`와 `evm.test`(누구나 잔액을 발행하고 시간을 앞당길 수 있음)도 노출되므로, localhost에만 노출하세요.
- hosts 파일에서 `app.test`와 다른 이름들을 127.0.0.1로 지정합니다.

인증서는 자체 서명입니다.

1. `https://app.test/`를 열고 "새로 만들기" 또는 "단어로 복원"을 고릅니다.
   키를 암호화할 패스프레이즈(8자 이상)를 정합니다(암호화하지 않기를 명시적으로 선택하거나, NIP-07 확장을 쓸 수도 있습니다).
   다음부터는 패스프레이즈로 키 잠금을 해제합니다.
2. "주문"에서 상점 URL(예: `https://safe-shop.test/`), 상점의 지역(예: `JP-13-13104`), 상품(예: `A-100`)을 입력하고 제안을 검색합니다.
3. 구매 대행자 × 에스크로 조합을 골라 배송 주소를 입력하고 주문합니다.
4. 견적이 도착하면 환율 차이와 멀티시그 주소 검증 결과를 확인한 뒤 수락합니다.
5. 실험 환경에서는 "faucet에서 받기"로 지갑을 채운 다음 "멀티시그에 예치"를 누릅니다.
   돈이 움직이는 모든 동작(예치, 지급, 부서명)은 금액과 받는 사람을 보여 주는 화면을 거칩니다.
6. 물건이 도착하면 "수령함"을 눌러 구매 대행자에게 지급합니다. "완료"는 지급이 체인에서 확인된 뒤에만 표시됩니다.

## 2. 공개 네트워크에서 실행하기

구조는 실험 환경과 같습니다. 차이점은 다음과 같습니다.

- ACME 인증서를 사용합니다.
- `lab/keys`의 키를 사용하지 않습니다.
- 실제 signet과 EVM 체인을 사용합니다.

### 구매 대행자

항상 온라인이며 Go 노드와 shopper-bot을 실행합니다.

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

- 카드 정보는 shopper-bot의 설정 파일(`BOT_CARDS_FILE`)에만 넣습니다. 노드는 절대 받지 않습니다.
- 상점마다의 절차는 shopper-bot 드라이버로 작성합니다. AI로 상점을 조작하는 드라이버도 입력과 출력이 같습니다(`PurchaseRequest` / `PurchaseResult`).

### 에스크로 / 운영자 / 코디네이터

항상 온라인일 필요는 없습니다.

- 웹 앱의 "에스크로", "운영자", "코디네이터" 화면을 쓸 수 있습니다.
- 계속 실행하려면 `role: escrow` / `role: operator`로 Go 노드를 실행합니다.
- 서명만 한다면 `psctl`도 됩니다.

버전(`v`)의 기본값은 UNIX 시간이므로 보통은 넘기지 않습니다.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. 어디서든, 허락 없이 자신만의 네트워크 역할 운영하기

proxy-shopping에는 중앙 운영자도, 가입 절차도 없습니다. 신뢰의 사슬은 키와 서명된 Nostr 이벤트뿐이므로,
당신(또는 당신을 위해 일하는 AI 에이전트)이 자기 동네에서 지역 장터를 시작할 수 있습니다.

1. **코디네이터 되기:** 키를 만듭니다(`psctl keys --mnemonic-file coordinator.mnemonic`). 코디네이터란 그것이 전부입니다.
2. **운영자에게 위임하기** (자신의 두 번째 키든, 신뢰하는 사람이든):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **자기 지역의 구매 대행자와 에스크로를 목록에 올리기** — 자신은 구매 대행자로, 친구는 에스크로로:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **사람들에게 당신의 코디네이터 키를 신뢰하도록 알리기** (웹 앱의 설정에서 추가하거나, MCP 서버라면 `PS_COORDINATORS`).
   모두에게 발견되려면 [레지스트리](https://github.com/pad01g/proxy-shopping-registry)에 `coordinators/<name>.json`을
   추가하는 pull request를 보내세요. 자신의 커뮤니티를 위해 같은 방식으로 승인을 운영하려면 레지스트리를 fork하세요.

**어디서 실행하나:** 집에 있는 컴퓨터 한 대면 충분합니다. 노드는 나가는 연결만 만들기 때문에(Nostr 릴레이, 그리고 NAT 뒤에 있을 때는 libp2p
circuit relay) 포트를 열 필요가 없습니다. 외출 중에 휴대폰에서 노드의 관리 API와 웹 앱에 접속하려면 [Tailscale](https://tailscale.com/)이나
다른 VPN을 쓰세요. 작은 VPS도 괜찮습니다.

**AI 에이전트가 할 수 있는 일:** 에이전트는 운영자와 구매 대행자의 일상 업무를 맡을 수 있습니다 — 노드의 관리
API나 MCP 서버로 주문을 지켜보고, 목록을 최신으로 유지하고, 카드 상점용 shopper-bot을 조작하고, 문제를 신고합니다. 그리고 당신만의
지역 지식(어떤 상점, 어떤 지역, 걸어서 갈 수 있는 현금 전용 가게)은 누구도 대신 제공할 수 없는 부분입니다.
스킬 `proxy-shopper`(`npx skills add pad01g/proxy-shopping-go`)가 에이전트에게 그 과정을 안내합니다.

사용자에게 솔직하게 알려 주세요. 공개 네트워크는 새롭고 BTC signet(테스트용 코인)으로 운영되므로, 수익은 사람들이 쓰기 시작한 뒤에 생깁니다.

## 기여하기 {#contributing}

**pull request를 환영합니다** — 모든 저장소에서: shopper-bot용 상점 드라이버, 새로운 결제 수단과 체인,
이 문서의 번역, 프로토콜 리뷰, 버그 수정, [레지스트리](https://github.com/pad01g/proxy-shopping-registry) 등재 등.
GitHub에서 issue나 pull request를 열어 주세요.
