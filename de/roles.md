---
layout: page
title: Rollen
permalink: /de/roles/
lang: de
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Rolle | Ständig online | Nutzt | Tut | Wenn betrogen wird |
|---|---|---|---|---|
| **Nutzer** (*user*) | nein | Browser | bestellt, zahlt ein, bestätigt den Empfang, eröffnet Streitfälle | ― |
| **Einkäufer** (*proxy shopper*) | **ja** | Go-Knoten + shopper-bot | erstellt Angebote, kauft im Namen des Nutzers, meldet den Versand; verdient eine Gebühr | verliert seinen Anteil in der Entscheidung des Treuhänders; der Betreiber entfernt ihn aus der Liste |
| **Treuhänder** (*escrow*) | nein | Browser oder Go-Knoten | entscheidet Streitfälle vor T1 | der Betreiber entfernt ihn aus der Liste; je nach Vereinbarung wird seine Kaution einbehalten und als Entschädigung verwendet |
| **Betreiber** (*operator*) | nein | Browser, Go-Knoten oder `psctl` | signiert pro Region die Liste vertrauenswürdiger Kombinationen aus Einkäufer × Treuhänder; nimmt Meldungen entgegen | der Koordinator widerruft seine Delegation |
| **Koordinator** (*coordinator*) | nein | Browser oder `psctl` | signiert Delegationen an Betreiber | Nutzer entfernen ihn aus ihren Einstellungen (Entlassung) |

## Nutzer

1. Öffnen Sie die Web-App, erzeugen Sie eine Mnemonic (12 Wörter), schreiben Sie sie auf und wählen Sie eine Passphrase, die den Schlüssel verschlüsselt.
2. Geben Sie die Shop-URL, die Region des Shops und die Artikel ein.
3. Wählen Sie eine der angebotenen Kombinationen aus Einkäufer × Treuhänder und geben Sie die Bestellung auf.
4. Ein Angebot trifft ein. Weicht sein Kurs um mehr als 3 % von Ihren eigenen Quellen ab, erhalten Sie einen Hinweis; über 10 % eine deutliche Warnung.
5. Annehmen und einzahlen. Das Geld wird zwischen dem Multisig und der Vorabgebühr des Treuhänders aufgeteilt.
6. Wenn der Artikel ankommt, drücken Sie „erhalten“ (nach einer Anzeige von Betrag und Empfänger geht Ihre Teilsignatur an den Einkäufer).
7. Kommt er nicht an, eröffnen Sie einen Streitfall. Die App sammelt die Nachweise für Sie.

## Einkäufer

- Betreiben Sie den Go-Knoten mit `role: shopper` und verbinden Sie shopper-bot (das Browser-Automatisierungswerkzeug).
- Konfigurieren Sie Zahlungsmittel, Währungen, die Regionen, in denen Sie bar zahlen können, Ihre Gebühr und Ihre Risikorichtlinie für Shops.
- Kartendaten bleiben ausschließlich in shopper-bot; sie werden nie an den Knoten oder das Netz weitergegeben.
- Bestätigt der Nutzer den Empfang nie und ist T1 verstrichen, können Sie das Geld allein abholen.

## Treuhänder

- Entscheidet nur Streitfälle von Bestellungen, die seine Vorabgebühr bezahlt haben.
- Er kann die Lieferadresse normalerweise nicht lesen. Im Streitfall übergibt ihm der Einkäufer (oder der Nutzer) den Entschlüsselungsschlüssel.
- Eine Entscheidung erreicht beide Parteien als signierte Transaktion; sie wird endgültig, sobald eine der Parteien gegenzeichnet.

## Betreiber

- Signiert seine Regionen (eine oder mehrere), die Liste der Kombinationen sowie die Relays und Chain-Endpunkte, die er empfiehlt.
- Nach einer Meldung veröffentlicht er eine neue Version der Liste ohne den Übeltäter. Die Kaution richtet sich nach den mit dem Treuhänder vereinbarten Bedingungen.

## Koordinator

- Stellt Delegationen an Betreiber aus. Widerrufen heißt, eine neue Version mit `revoked` zu veröffentlichen.

## Gelistet werden (das Register)

Das Vertrauensregister des Maintainers ist das GitHub-Repository
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Ein gemergter Pull Request ist die
Freigabe:** Nach jedem Merge signiert die CI die neuen Delegationen und Listen mit den Schlüsseln des Registers und veröffentlicht sie
(auf den öffentlichen Nostr-Relays und unter https://pad01g.github.io/proxy-shopping-registry/events.json).

| Sie möchten sein | Im Pull Request hinzufügen | Was der Merge bewirkt |
|---|---|---|
| Einkäufer | `shoppers/<name>.json` (pk, Kontakt, Beschreibung, Bargeld-Regionen, Zahlungsmittel, die Treuhänder, mit denen Sie arbeiten) | der Betreiber des Registers listet Ihre Kombinationen aus Einkäufer × Treuhänder |
| Treuhänder | `escrows/<name>.json` (pk, Kontakt, Beschreibung, SLA in Tagen) | Einkäufer können Sie angeben; Ihre Kombinationen erscheinen |
| Betreiber | `operators/<name>.json` (pk, Kontakt, Beschreibung, Regionen) | der Koordinator des Registers delegiert an Sie; danach signieren Sie Ihre eigenen Listen |
| Koordinator | `coordinators/<name>.json` (pk, Kontakt, Beschreibung) | Sie erscheinen im Koordinator-Verzeichnis, das die Apps den Nutzern anbieten (jeder Nutzer entscheidet weiterhin selbst, wem er vertraut) |

`pk` ist Ihr öffentlicher Nostr-Schlüssel (64 Hex-Zeichen): Die Web-App zeigt ihn unter Einstellungen an, und `psctl keys --mnemonic-file …`
gibt ihn aus. Jemanden zu entfernen ist ein Pull Request, der dessen Datei mit einer Begründung nach `revoked/` verschiebt. Das README
des Registers enthält die genauen Dateiformate und die Befehle.
