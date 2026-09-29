---
layout: page
title: Erste Schritte
permalink: /de/quickstart/
lang: de
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Das ganze Netz lokal betreiben (Testumgebung)

Eine einzige docker-compose-Datei startet ein geschlossenes Netz. Es verbindet sich nicht mit dem Internet.

- zwei Nostr-Relays
- ein libp2p-Relay
- bitcoind auf einem eigenen Signet und eine Esplora-kompatible API
- anvil (EVM) mit Safe und den übrigen Verträgen
- vier fiktive Shops und ein fiktives Karten-Zahlungs-Gateway
- zwei Einkäufer (*shopper*), drei Treuhänder (*escrow*, einer davon hinter NAT), zwei Betreiber (*operator*)
- die öffentliche Web-App und die Demo-App

Voraussetzungen: Docker (etwa 8 GB Arbeitsspeicher).

Legen Sie proxy-shopping-go und proxy-shopping-web nebeneinander in dasselbe übergeordnete Verzeichnis und führen Sie aus:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Um von vorn zu beginnen, führen Sie zuerst `docker compose down -v` aus, damit auch die gespeicherten Daten der Knoten und Relays gelöscht werden
(die Chains beginnen bei jedem Start neu, übrig gebliebene Bestelldaten würden also nicht zu ihnen passen).

### Die e2e-Tests ausführen

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Die Ergebnisse landen in `e2e/results/e2e-latest.md`. Der Runner arbeitet innerhalb des NAT (das Netz `home`),
daher geht jedes Szenario davon aus, dass der Nutzer hinter NAT sitzt.

| id | Szenario |
|---|---|
| a | Normalfall: BTC (ein Shop in JPY) und USDC (ein Shop in USD). Der Kurs des Angebots wird geprüft, der Treuhänder erhält seine Vorabgebühr, der Einkäufer wird bezahlt. Ein Angebot mit stark abweichendem Kurs erzeugt eine deutliche Warnung |
| b | Die Lieferung schlägt fehl und der Nutzer eröffnet einen Streitfall. Ein ehrlicher Treuhänder entscheidet auf Erstattung und der Nutzer zeichnet gegen (BTC). Der Treuhänder kann die Adresse entschlüsseln und erhält den Screenshot als Anhang |
| c | Ein Treuhänder entscheidet unehrlich. Der Nutzer meldet ihn; der Betreiber entfernt ihn aus der Liste und behält gemäß ihrer Vereinbarung seine Kaution als Entschädigung ein (USDC) |
| d | Ein Koordinator (*coordinator*) widerruft die Delegation eines Betreibers, und die Kombinationen dieser Liste verschwinden aus den Angeboten |
| e | Ein Browser hinter NAT bestellt über die öffentliche Web-App (1:1-Nachrichten laufen über Nostr-Relays). Eine Statusabfrage erreicht den Treuhänder-Knoten hinter NAT über das libp2p-Circuit-Relay |
| f | Ein Einkäufer lehnt einen Shop ab, den er als riskant bewertet, sowie ein Nur-Bargeld-Geschäft außerhalb seiner Region. Ein Einkäufer in der Region übernimmt die Bargeld-Bestellung und liefert |
| g | Nach T1 kann der Einkäufer das Geld allein abholen (BTC) |
| h | Verschwindet der Einkäufer, holt der Nutzer das Geld nach T2 allein zurück (USDC) |
| i | USDC-Streitfall mit einem ehrlichen Treuhänder. Die Erstattung wird auch dann ausgeführt, wenn jemand vor der Entscheidung einen kleinen Betrag an den Safe sendet |
| j | Eine Bestellung, die der Shop nicht erfüllen konnte (ausverkauft). Der Einkäufer bietet eine einvernehmliche Erstattung an; der Nutzer prüft sie und nimmt an |
| k | Verschwindet der Einkäufer, holt der Nutzer das Geld nach T2 allein zurück (BTC) |

### Den ganzen Ablauf in der Demo ausprobieren

Während die Testumgebung läuft, öffnen Sie die Demo unter `http://localhost:8888/` (keine hosts-Datei und keine Zertifikate nötig).
Auf einem Bildschirm haben Nutzer, Treuhänder, Betreiber und Koordinator jeweils einen eigenen Schlüssel (im lokalen Speicher des Browsers),
und der Einkäufer ist der ständig laufende Go-Knoten selbst. Die Wege sind echt: Nostr-Relays, bitcoind, anvil, Go-Knoten.

- Wählen Sie oben ein Szenario. Die Anleitung links sagt, wer als Nächstes was tut und warum.
- „Zu diesem Schritt“ wechselt zum Tab der jeweiligen Rolle und zeigt auf die zu drückende Schaltfläche. Die Formulare sind für das Szenario vorausgefüllt, Sie klicken und bestätigen also nur.
- Szenarien: Normalfall (BTC / USDC), fehlgeschlagene Lieferung und Erstattung, ausverkauft und einvernehmliche Erstattung, abgelehnter riskanter Shop, unehrlicher Treuhänder gemeldet und aus der Liste entfernt, sowie Erstattung nach T2, wenn der Einkäufer verschwindet.
- Sie können auch ein Fenster pro Rolle öffnen, z. B. `?role=user` und `?role=escrow,operator,coordinator` (Fenster desselben Browsers teilen die Schlüssel und den Fortschritt).
- In Wirklichkeit ist jede Rolle woanders, in ihrem eigenen Browser. Die Demo trennt nur die Schlüssel.
- Nur Testumgebung: Wer Port 8888 erreichen kann, kann Faucet, Mining, Zeitsprung und die Admin-API des Einkäufers nutzen (er lauscht nur auf 127.0.0.1).

Es gibt auch einen e2e-Test, der die Demo anhand ihrer Anleitung durchspielt: `docker compose run --rm runner demo`.

### Die Web-App verwenden

Die Namen der Testumgebung (`*.test`) werden nur innerhalb der Container aufgelöst.
Um die Web-App aus Ihrem Browser zu nutzen:

- Geben Sie Port 443 des Containers `edge` lokal frei (z. B. in `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Damit werden auch die nur für die Testumgebung gedachten `faucet.test` und `evm.test` erreichbar (jeder kann Guthaben erzeugen und die Zeit vorstellen), geben Sie ihn also nur auf localhost frei.
- Lassen Sie `app.test` und die anderen Namen in Ihrer hosts-Datei auf 127.0.0.1 zeigen.

Die Zertifikate sind selbstsigniert.

1. Öffnen Sie `https://app.test/` und wählen Sie „neu erstellen“ oder „aus Wörtern wiederherstellen“.
   Wählen Sie eine Passphrase (8 Zeichen oder mehr), die den Schlüssel verschlüsselt (Sie können auch ausdrücklich auf Verschlüsselung verzichten oder eine NIP-07-Erweiterung verwenden).
   Beim nächsten Mal entsperren Sie den Schlüssel mit der Passphrase.
2. Geben Sie unter „Bestellung“ die Shop-URL (z. B. `https://safe-shop.test/`), die Region des Shops (z. B. `JP-13-13104`) und den Artikel (z. B. `A-100`) ein und suchen Sie nach Angeboten.
3. Wählen Sie eine Kombination aus Einkäufer × Treuhänder, geben Sie die Lieferadresse ein und bestellen Sie.
4. Wenn das Angebot eintrifft, prüfen Sie die Kursabweichung und das Ergebnis der Multisig-Adressprüfung und nehmen Sie dann an.
5. Füllen Sie in der Testumgebung Ihre Wallet mit „vom Faucet holen“ und drücken Sie dann „Multisig einzahlen“.
   Jede Aktion, die Geld bewegt (Einzahlen, Bezahlen, Gegenzeichnen), führt über eine Anzeige von Betrag und Empfänger.
6. Wenn der Artikel ankommt, drücken Sie „erhalten“, um den Einkäufer zu bezahlen. „Abgeschlossen“ erscheint erst, wenn die Zahlung on-chain bestätigt ist.

## 2. In einem öffentlichen Netz betreiben

Die Architektur ist dieselbe wie in der Testumgebung. Die Unterschiede:

- ACME-Zertifikate verwenden;
- nicht die Schlüssel aus `lab/keys` verwenden;
- ein echtes Signet und eine echte EVM-Chain verwenden.

### Einkäufer

Ständig online; betreibt den Go-Knoten und shopper-bot.

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

- Kartendaten stehen nur in der Konfigurationsdatei von shopper-bot (`BOT_CARDS_FILE`); der Knoten erhält sie nie.
- Die Schritte für jeden Shop werden als shopper-bot-Treiber geschrieben. Ein Treiber, der Shops mit KI bedient, hat dieselbe Ein- und Ausgabe (`PurchaseRequest` / `PurchaseResult`).

### Treuhänder / Betreiber / Koordinator

Diese müssen nicht ständig online sein.

- Sie können die Bildschirme „Escrow“, „Operator“ und „Coordinator“ der Web-App nutzen.
- Für den Dauerbetrieb starten Sie den Go-Knoten mit `role: escrow` / `role: operator`.
- Nur zum Signieren genügt auch `psctl`.

Die Version (`v`) ist standardmäßig die UNIX-Zeit, Sie müssen sie also normalerweise nicht angeben.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Ihre eigene Rolle im Netz betreiben, überall, ohne Erlaubnis

proxy-shopping hat keinen zentralen Betreiber und keine Anmeldung. Die Vertrauenskette besteht nur aus Schlüsseln und signierten Nostr-Ereignissen,
also können Sie (oder ein KI-Agent, der für Sie arbeitet) einen lokalen Marktplatz in Ihrer eigenen Stadt starten:

1. **Werden Sie Koordinator:** Erzeugen Sie einen Schlüssel (`psctl keys --mnemonic-file coordinator.mnemonic`). Mehr ist ein Koordinator nicht.
2. **Delegieren Sie an einen Betreiber** (Ihren eigenen zweiten Schlüssel oder jemanden, dem Sie vertrauen):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Listen Sie Einkäufer und Treuhänder für Ihre Region** — sich selbst als Einkäufer, eine Freundin oder einen Freund als Treuhänder:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Bitten Sie andere, Ihrem Koordinator-Schlüssel zu vertrauen** (in den Einstellungen der Web-App hinzufügen oder per `PS_COORDINATORS` für den MCP-Server).
   Um von allen gefunden zu werden, öffnen Sie einen Pull Request an das [Register](https://github.com/pad01g/proxy-shopping-registry),
   der `coordinators/<name>.json` hinzufügt; um Freigaben für Ihre eigene Community auf dieselbe Weise abzuwickeln, forken Sie das Register.

**Wo betreiben:** Ein Rechner zu Hause genügt. Der Knoten baut nur ausgehende Verbindungen auf (Nostr-Relays und, hinter NAT, ein libp2p-
Circuit-Relay), Sie öffnen also keine Ports. Nutzen Sie [Tailscale](https://tailscale.com/) oder ein beliebiges VPN, um unterwegs vom Handy
auf die Admin-API Ihres Knotens und die Web-App zuzugreifen; ein kleiner VPS geht auch.

**Was ein KI-Agent tun kann:** Ein Agent kann die Routine von Betreiber und Einkäufer übernehmen — Bestellungen über die Admin-API
des Knotens oder den MCP-Server verfolgen, die Listen aktuell halten, shopper-bot für Kartenzahlungs-Shops steuern, Probleme melden —, und Ihr
Wissen vor Ort (welche Shops, welche Regionen, welche Nur-Bargeld-Geschäfte Sie zu Fuß erreichen) ist das, was niemand sonst bieten kann.
Die Skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) führt einen Agenten durch den Ablauf.

Seien Sie ehrlich zu Ihren Nutzern: Das öffentliche Netz ist neu und läuft auf BTC signet (Testmünzen), Einnahmen kommen also erst, wenn Menschen es nutzen.

## Mitwirken {#contributing}

**Pull Requests sind willkommen** — in allen Repositories: Shop-Treiber für shopper-bot, neue Zahlungsmittel und Chains,
Übersetzungen dieser Dokumentation, Reviews des Protokolls, Fehlerbehebungen und Einträge im
[Register](https://github.com/pad01g/proxy-shopping-registry). Öffnen Sie ein Issue oder einen Pull Request auf GitHub.
