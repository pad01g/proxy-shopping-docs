---
layout: page
title: How it works
permalink: /en/overview/
lang: en
ref: overview
nav_order: 2
---

{% include langnav.html %}

## What it does

When the shop you want to buy from only accepts cash or a specific payment method, you pay with crypto
(currently **BTC signet** and **USDC**) and a **proxy shopper** buys the item for you and ships it.

The money goes into a **2-of-3 multisig** per order (user, shopper, escrow).

- When the item arrives, the user and the shopper sign to pay the shopper.
- If there is a dispute, the escrow decides the split together with one of the other two.
- Even if someone stops responding, **timelocks** make sure the money ends up with one side.

## Architecture

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **Users run nothing.** Opening the public web app is enough to join.
  Keys are created in the browser and never leave it; every authorization (signature) happens in the browser.
- Users behind NAT can connect too, because the connection to a Nostr relay is outbound.
- Messages for people who are not always online (users, escrows, operators, coordinators) wait,
  still encrypted, on Nostr relays acting as mailboxes. Each message goes to several relays.
- Only the shopper needs to be always online. It runs a Go node and a browser automation tool (shopper-bot) that operates the shops.

## How trust flows

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- The user trusts **only the coordinator's public key**. Trust extends from there to operators and to shopper × escrow combinations.
- Lists are versioned and never expire. Removing someone means publishing a new version.
- A user "dismisses" a coordinator by removing it from their settings.

## Fees

| To whom | Enforceable? | How |
|---|---|---|
| shopper | yes | included in the quote |
| escrow (upfront) | yes | paid directly to the escrow together with the funding. **An escrow has no duty to arbitrate orders without the upfront fee** |
| escrow (dispute) | yes | taken from the split of the ruling |
| operator / coordinator | no | agreed outside the protocol, e.g. listing fees. What a list sells is "being found" |

The escrow's bond and its confiscation are not part of the protocol.
They are treated as terms the operator and the escrow agree on themselves; a reference contract is provided.

## Cash-only shops

A shop's location is a region code (e.g. `JP-13-13104` = Shinjuku, Tokyo).
A shopper declares the regions where it can go and pay cash, and only takes orders for shops there.

## Shop risk

Before taking an order the shopper scores the shop:
is it on the shopper's allowlist, is it HTTPS, and is its checkout a known payment gateway.
Sending card details to an arbitrary site is risky, so orders for low-scoring shops are declined.
