---
layout: page
title: Getting started
permalink: /en/quickstart/
lang: en
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Run the whole network locally (lab)

A single docker compose file brings up a closed network. It does not connect to the internet.

- two Nostr relays
- a libp2p relay
- bitcoind on a custom signet, and an Esplora-compatible API
- anvil (EVM) with Safe and the other contracts
- four fake shops and a fake card payment gateway
- two shoppers, three escrows (one behind NAT), two operators
- the public web app and the demo app

Requirements: Docker (about 8 GB of memory).

Put proxy-shopping-go and proxy-shopping-web side by side in the same parent directory and run:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

To start over, run `docker compose down -v` first to delete the nodes' and relays' stored data as well
(the chains start from scratch on every start, so leftover order data would disagree with them).

### Run the e2e tests

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Results go to `e2e/results/e2e-latest.md`. The runner works from inside the NAT (the `home` network),
so every scenario assumes the user is behind NAT.

| id | Scenario |
|---|---|
| a | Happy path: BTC (a JPY shop) and USDC (a USD shop). The quote's rate is checked, the escrow gets its upfront fee, the shopper gets paid. A quote whose rate is far off gets a strong warning |
| b | Delivery fails and the user opens a dispute. An honest escrow rules a refund and the user countersigns (BTC). The escrow can decrypt the address and receives the screenshot as an attachment |
| c | An escrow rules dishonestly. The user reports it; the operator removes it from the list and, under their terms, confiscates its bond as compensation (USDC) |
| d | A coordinator revokes an operator's delegation, and that list's combinations disappear from the offers |
| e | A browser behind NAT orders through the public web app (1:1 messages go over Nostr relays). A status query reaches the escrow node behind NAT through the libp2p circuit relay |
| f | A shopper declines a shop it scores as risky, and a cash-only shop outside its region. A shopper in the region takes the cash-only order and delivers |
| g | After T1, the shopper can take the funds alone (BTC) |
| h | If the shopper disappears, the user takes the funds back alone after T2 (USDC) |
| i | USDC dispute with an honest escrow. The refund still executes even if someone sends a small amount to the Safe before the ruling |
| j | An order the shop could not fill (sold out). The shopper offers a cooperative refund; the user checks it and accepts |
| k | If the shopper disappears, the user takes the funds back alone after T2 (BTC) |

### Try the whole flow in the demo

With the lab running, open the demo at `http://localhost:8888/` (no hosts file or certificates needed).
On one screen, the user, escrow, operator and coordinator each have their own key (in the browser's local storage),
and the shopper is the always-online Go node itself. The paths are real: Nostr relays, bitcoind, anvil, Go nodes.

- Pick a scenario at the top. The guide on the left says who does what next, and why.
- "Go to this step" switches to that role's tab and points at the button to press. Forms are prefilled for the scenario, so you only click and confirm.
- Scenarios: happy path (BTC / USDC), failed delivery and refund, sold out and cooperative refund, a risky shop declined, a dishonest escrow reported and removed from the list, and a refund after T2 when the shopper disappears.
- You can also open one window per role, e.g. `?role=user` and `?role=escrow,operator,coordinator` (windows of the same browser share the keys and the progress).
- In reality every role is somewhere else, in its own browser. The demo separates only the keys instead.
- Lab only: anyone who can reach port 8888 can use the faucet, mining, time warp and the shopper's admin API (it is bound to 127.0.0.1 only).

There is also an e2e that drives the demo by following its guide: `docker compose run --rm runner demo`.

### Use the web app

The lab names (`*.test`) resolve only inside the containers.
To use the web app from your browser:

- Expose port 443 of the `edge` container locally (e.g. in `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  This also exposes the lab-only `faucet.test` and `evm.test` (anyone can mint balances and move time forward), so expose it on localhost only.
- Point `app.test` and the other names at 127.0.0.1 in your hosts file.

The certificates are self-signed.

1. Open `https://app.test/` and choose "create new" or "restore from words".
   Choose a passphrase (8 characters or more) that encrypts the key (you can also explicitly choose not to encrypt it, or use a NIP-07 extension).
   Next time, unlock the key with the passphrase.
2. Under "order", enter the shop URL (e.g. `https://safe-shop.test/`), the shop's region (e.g. `JP-13-13104`) and the item (e.g. `A-100`), and search for offers.
3. Pick a shopper × escrow combination, enter the delivery address and place the order.
4. When the quote arrives, check the rate difference and the result of the multisig address check, then accept.
5. In the lab, fill your wallet with "get from the faucet", then press "fund the multisig".
   Every action that moves money (funding, paying, countersigning) goes through a screen showing the amount and recipient.
6. When the item arrives, press "received" to pay the shopper. "Completed" appears only after the payment is confirmed on chain.

## 2. Run on a public network

The architecture is the same as the lab. The differences:

- use ACME certificates;
- don't use the keys in `lab/keys`;
- use a real signet and EVM chain.

### Shopper

Always online; runs the Go node and shopper-bot.

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

- Card details go only in shopper-bot's config file (`BOT_CARDS_FILE`); the node never gets them.
- Each shop's steps are written as a shopper-bot driver. A driver that uses AI to operate shops has the same input and output (`PurchaseRequest` / `PurchaseResult`).

### Escrow / operator / coordinator

These need not be always online.

- They can use the "Escrow", "Operator" and "Coordinator" screens of the web app.
- To run continuously, run the Go node with `role: escrow` / `role: operator`.
- For signing only, `psctl` works too.

The version (`v`) defaults to the UNIX time, so you normally don't pass it.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```
