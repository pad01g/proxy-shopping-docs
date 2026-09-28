---
layout: page
title: Protocol
permalink: /en/protocol/
lang: en
ref: protocol
nav_order: 5
---

{% include langnav.html %}

The full specification is `docs/spec.md` in proxy-shopping-go. This page summarizes the essentials.

## Keys

All keys are derived from one mnemonic (BIP39), one per purpose (all secp256k1).

| Purpose | Path |
|---|---|
| identity (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| BTC order key (user, shopper) | `m/7333'/1'/{idx}'` (idx comes from a hash of the order ID) |
| BTC order key (escrow) | `m/7333'/2'/{idx}`. The xpub is public, so others can derive it while the escrow is offline |
| BTC wallet | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Trust

| kind | Signed by | Contents |
|---|---|---|
| 30500 | coordinator | delegation to an operator; revoked with `revoked` |
| 30501 | operator | region × shopper × escrow list, recommended relays and chain endpoints |
| 30502 / 30503 | shopper / escrow | profile (fees, cash regions, xpub, payout addresses) |
| 10050 | everyone | the relays where one receives messages |

The `v` tag version decides which is newer (normally the UNIX time of signing). Nothing expires.
Events travel over both Nostr relays and libp2p gossipsub; Go nodes forward what they receive on one to the other.

## Messages

Messages are wrapped with NIP-59 (gift wrap → seal → content).
The content is a **signed** event (kind 5400), so that an escrow can verify it as a third party in a dispute.

- A message goes to at least k (default 2) of the recipient's inbox relays.
- The recipient answers with an `ack`; the sender resends until it gets one.
- A message is at most 28000 bytes. Large evidence such as screenshots is referenced by hash only;
  in a dispute it is sent to the escrow in pieces as `attachment` messages.

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

T1 < T2 (block heights). If the user never confirms receipt and T1 passes, the shopper can take the funds alone.
If the shopper disappears, the user can take them back alone after T2.
The escrow must rule on a dispute before T1.

## USDC (Safe v1.4.1)

- Each order gets a Safe with three owners and a threshold of two.
- Its address comes from CREATE2, so it can be computed in advance from the order ID.
- The timelocks are implemented as a Safe module (`PSEscrowModule`).
  - After t1, the shopper can take the funds with `claimByShopper`.
  - After t2, the user can take them back with `refundToUser`.
- Payouts and rulings are SafeTxs signed with EIP-712. A ruling's split uses `MultiSendCallOnly`.

## Binding the identity to the keys

Each order request (`order.request`) carries a `key_proof`:
a signature by the chain key that goes into the multisig (the BTC order key or the EVM account),
showing that it belongs to the same person as the Nostr identity.
Without it, another identity could copy the user's public keys into its own request and claim to the escrow that it is the user.

## Delivery address

The address is encrypted with a one-time key K (XChaCha20-Poly1305).

- K is wrapped with NIP-44 for the shopper and, separately, for the escrow.
- The escrow's copy is not part of the request: it is handed to the shopper in `order.escrow_key` (the request carries only its hash).
  The shopper (or the user) passes it to the escrow in a dispute, so the escrow cannot read the address of orders without a dispute.

## Regions

Codes are matched by prefix: `JP` > `JP-13` (Tokyo) > `JP-13-13104` (Shinjuku).
The municipality level uses Japan's local government codes.
