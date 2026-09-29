---
layout: page
title: Protokoll
permalink: /de/protocol/
lang: de
ref: protocol
nav_order: 5
---

{% include langnav.html %}

Die vollständige Spezifikation ist `docs/spec.md` in proxy-shopping-go. Diese Seite fasst das Wichtigste zusammen.

## Schlüssel

Alle Schlüssel werden aus einer einzigen Mnemonic (BIP39) abgeleitet, einer pro Zweck (alle secp256k1).

| Zweck | Pfad |
|---|---|
| Identität (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| BTC-Bestellschlüssel (Nutzer, Einkäufer) | `m/7333'/1'/{idx}'` (idx stammt aus einem Hash der Bestell-ID) |
| BTC-Bestellschlüssel (Treuhänder) | `m/7333'/2'/{idx}`. Der xpub ist öffentlich, sodass andere ihn ableiten können, während der Treuhänder offline ist |
| BTC-Wallet | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Vertrauen

| kind | Signiert von | Inhalt |
|---|---|---|
| 30500 | Koordinator (*coordinator*) | Delegation an einen Betreiber; widerrufen mit `revoked` |
| 30501 | Betreiber (*operator*) | Liste aus Region × Einkäufer × Treuhänder, empfohlene Relays und Chain-Endpunkte |
| 30502 / 30503 | Einkäufer (*shopper*) / Treuhänder (*escrow*) | Profil (Gebühren, Bargeld-Regionen, xpub, Auszahlungsadressen) |
| 10050 | alle | die Relays, auf denen man Nachrichten empfängt |

Die Version im Tag `v` entscheidet, welches Ereignis neuer ist (normalerweise die UNIX-Zeit der Signatur). Nichts läuft ab.
Ereignisse laufen sowohl über Nostr-Relays als auch über libp2p-gossipsub; Go-Knoten leiten das, was sie auf dem einen Weg erhalten, auf den anderen weiter.

## Nachrichten

Nachrichten werden mit NIP-59 verpackt (gift wrap → seal → Inhalt).
Der Inhalt ist ein **signiertes** Ereignis (kind 5400), damit ein Treuhänder es im Streitfall als Dritter prüfen kann.

- Eine Nachricht geht an mindestens k (Standard 2) der Posteingangs-Relays des Empfängers.
- Der Empfänger antwortet mit einem `ack`; der Absender sendet erneut, bis er eines erhält.
- Eine Nachricht umfasst höchstens 28000 Bytes. Große Nachweise wie Screenshots werden nur per Hash referenziert;
  im Streitfall werden sie in Teilen als `attachment`-Nachrichten an den Treuhänder gesendet.

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

T1 < T2 (Blockhöhen). Bestätigt der Nutzer den Empfang nie und ist T1 verstrichen, kann der Einkäufer das Geld allein abholen.
Verschwindet der Einkäufer, kann der Nutzer es nach T2 allein zurückholen.
Der Treuhänder muss einen Streitfall vor T1 entscheiden.

## USDC (Safe v1.4.1)

- Jede Bestellung erhält einen Safe mit drei Eigentümern und einem Schwellenwert von zwei.
- Seine Adresse ergibt sich aus CREATE2 und lässt sich daher vorab aus der Bestell-ID berechnen.
- Die Timelocks sind als Safe-Modul umgesetzt (`PSEscrowModule`).
  - Nach t1 kann der Einkäufer das Geld mit `claimByShopper` abholen.
  - Nach t2 kann der Nutzer es mit `refundToUser` zurückholen.
- Auszahlungen und Entscheidungen sind mit EIP-712 signierte SafeTxs. Die Aufteilung einer Entscheidung nutzt `MultiSendCallOnly`.

## Identität an die Schlüssel binden

Jede Bestellanfrage (`order.request`) enthält einen `key_proof`:
eine Signatur des Chain-Schlüssels, der in das Multisig eingeht (der BTC-Bestellschlüssel oder das EVM-Konto),
die zeigt, dass er derselben Person gehört wie die Nostr-Identität.
Ohne ihn könnte eine andere Identität die öffentlichen Schlüssel des Nutzers in ihre eigene Anfrage kopieren und dem Treuhänder gegenüber behaupten, der Nutzer zu sein.

## Lieferadresse

Die Adresse wird mit einem Einmalschlüssel K verschlüsselt (XChaCha20-Poly1305).

- K wird mit NIP-44 für den Einkäufer und separat für den Treuhänder verpackt.
- Die Kopie des Treuhänders ist nicht Teil der Anfrage: Sie wird dem Einkäufer in `order.escrow_key` übergeben (die Anfrage enthält nur ihren Hash).
  Der Einkäufer (oder der Nutzer) gibt sie im Streitfall an den Treuhänder weiter, sodass der Treuhänder die Adresse von Bestellungen ohne Streitfall nicht lesen kann.

## Regionen

Codes werden per Präfix abgeglichen: `JP` > `JP-13` (Tokio) > `JP-13-13104` (Shinjuku).
Die Gemeindeebene verwendet die japanischen Gemeindeschlüssel.
