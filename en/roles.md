---
layout: page
title: Roles
permalink: /en/roles/
lang: en
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Role | Always online | Uses | Does | If it cheats |
|---|---|---|---|---|
| **user** | no | browser | orders, funds, confirms receipt, opens disputes | ― |
| **proxy shopper** | **yes** | Go node + shopper-bot | quotes, buys on the user's behalf, reports shipping; earns a fee | loses its share in the escrow's ruling; the operator removes it from the list |
| **escrow** | no | browser or Go node | rules on disputes before T1 | the operator removes it from the list; depending on their terms, its bond is confiscated and used as compensation |
| **operator** | no | browser, Go node or `psctl` | signs, per region, the list of trustworthy shopper × escrow combinations; receives reports | the coordinator revokes its delegation |
| **coordinator** | no | browser or `psctl` | signs delegations to operators | users remove it from their settings (dismissal) |

## User

1. Open the web app, create a mnemonic (12 words), write it down, and choose a passphrase that encrypts the key.
2. Enter the shop URL, the shop's region and the items.
3. Pick one of the offered shopper × escrow combinations and place the order.
4. A quote arrives. If its rate differs from your own sources by more than 3% you get a caution; above 10% a strong warning.
5. Accept and fund. The money is split between the multisig and the escrow's upfront fee.
6. When the item arrives, press "received" (after a screen showing the amount and recipient, your partial signature goes to the shopper).
7. If it doesn't arrive, open a dispute. The app gathers the evidence for you.

## Proxy shopper

- Run the Go node with `role: shopper` and connect shopper-bot (the browser automation tool).
- Configure the payment methods, currencies, regions where you can pay cash, your fee and your shop risk policy.
- Card details live only inside shopper-bot; they are never given to the node or the network.
- If the user never confirms receipt and T1 passes, you can take the funds alone.

## Escrow

- Rules only on disputes of orders that paid its upfront fee.
- It normally cannot read the delivery address. In a dispute, the shopper (or the user) hands it the decryption key.
- A ruling reaches both parties as a signed transaction; it becomes final once one of them countersigns.

## Operator

- Signs its regions (one or more), the list of combinations, and the relays and chain endpoints it recommends.
- On a report, publishes a new version of the list without the offender. The bond follows the terms it agreed with the escrow.

## Coordinator

- Issues delegations to operators. Revoking means publishing a new version with `revoked`.

## Getting listed (the registry)

The maintainer's trust registry is the GitHub repository
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **A merged pull request is the
approval:** after every merge, CI signs the new delegations and lists with the registry's keys and publishes them
(to the public Nostr relays and to https://pad01g.github.io/proxy-shopping-registry/events.json).

| You want to be | Add in a pull request | What the merge does |
|---|---|---|
| shopper | `shoppers/<name>.json` (pk, contact, description, cash regions, payments, the escrows you work with) | the registry's operator lists your shopper × escrow combinations |
| escrow | `escrows/<name>.json` (pk, contact, description, SLA days) | shoppers can name you; your combinations appear |
| operator | `operators/<name>.json` (pk, contact, description, regions) | the registry's coordinator delegates to you; you then sign your own lists |
| coordinator | `coordinators/<name>.json` (pk, contact, description) | you appear in the coordinator directory the apps offer to users (each user still chooses whom to trust) |

`pk` is your Nostr public key (64 hex chars): the web app shows it in Settings, and `psctl keys --mnemonic-file …`
prints it. Removing someone is a pull request that moves their file to `revoked/` with a reason. The registry's
README has the exact file formats and the commands.
