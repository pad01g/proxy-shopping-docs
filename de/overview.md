---
layout: page
title: Funktionsweise
permalink: /de/overview/
lang: de
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Was es tut

Wenn das Geschäft, in dem Sie kaufen möchten, nur Bargeld oder ein bestimmtes Zahlungsmittel akzeptiert, zahlen Sie mit Kryptowährung
(derzeit **BTC signet** und **USDC**), und ein **Einkäufer** (*proxy shopper*) kauft den Artikel für Sie und verschickt ihn.

Das Geld geht pro Bestellung in ein **2-von-3-Multisig** (Nutzer, Einkäufer, Treuhänder — *escrow*).

- Wenn der Artikel ankommt, signieren Nutzer und Einkäufer, um den Einkäufer zu bezahlen.
- Bei einem Streitfall entscheidet der Treuhänder zusammen mit einer der beiden anderen Parteien über die Aufteilung.
- Auch wenn jemand nicht mehr antwortet, sorgen **Timelocks** dafür, dass das Geld bei einer Seite ankommt.

## Architektur

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **Nutzer betreiben nichts.** Die öffentliche Web-App zu öffnen genügt, um teilzunehmen.
  Die Schlüssel werden im Browser erzeugt und verlassen ihn nie; jede Autorisierung (Signatur) geschieht im Browser.
- Auch Nutzer hinter NAT können sich verbinden, denn die Verbindung zu einem Nostr-Relay ist ausgehend.
- Nachrichten für diejenigen, die nicht ständig online sind (Nutzer, Treuhänder, Betreiber, Koordinatoren), warten
  weiterhin verschlüsselt auf Nostr-Relays, die als Postfächer dienen. Jede Nachricht geht an mehrere Relays.
- Nur der Einkäufer muss ständig online sein. Er betreibt einen Go-Knoten und ein Browser-Automatisierungswerkzeug (shopper-bot), das die Shops bedient.

## Wie Vertrauen weitergegeben wird

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- Der Nutzer vertraut **nur dem öffentlichen Schlüssel des Koordinators** (*coordinator*). Von dort aus erstreckt sich das Vertrauen auf die Betreiber (*operator*) und auf die Kombinationen aus Einkäufer × Treuhänder.
- Listen sind versioniert und laufen nie ab. Jemanden zu entfernen heißt, eine neue Version zu veröffentlichen.
- Ein Nutzer „entlässt“ einen Koordinator, indem er ihn aus seinen Einstellungen entfernt.

## Gebühren

| An wen | Durchsetzbar? | Wie |
|---|---|---|
| Einkäufer | ja | im Angebot enthalten |
| Treuhänder (im Voraus) | ja | zusammen mit der Einzahlung direkt an den Treuhänder gezahlt. **Ein Treuhänder ist nicht verpflichtet, bei Bestellungen ohne Vorabgebühr zu schlichten** |
| Treuhänder (Streitfall) | ja | aus der Aufteilung der Entscheidung genommen |
| Betreiber / Koordinator | nein | außerhalb des Protokolls vereinbart, z. B. Listungsgebühren. Was eine Liste verkauft, ist „gefunden werden“ |

Die Kaution des Treuhänders und ihre Einbehaltung sind nicht Teil des Protokolls.
Sie gelten als Bedingungen, die Betreiber und Treuhänder untereinander vereinbaren; ein Referenzvertrag wird bereitgestellt.

## Nur-Bargeld-Geschäfte

Der Standort eines Geschäfts ist ein Regionscode (z. B. `JP-13-13104` = Shinjuku, Tokio).
Ein Einkäufer gibt die Regionen an, in die er gehen und bar bezahlen kann, und nimmt nur Bestellungen für Geschäfte dort an.

## Risiko von Shops

Bevor er eine Bestellung annimmt, bewertet der Einkäufer den Shop:
Steht er auf der Allowlist des Einkäufers, nutzt er HTTPS, und läuft der Checkout über ein bekanntes Zahlungs-Gateway?
Kartendaten an eine beliebige Website zu senden ist riskant, daher werden Bestellungen für schlecht bewertete Shops abgelehnt.
