---
layout: page
title: Start
permalink: /de/
lang: de
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** ist ein P2P-Netz, mit dem Sie in Geschäften, die nur Bargeld oder bestimmte Zahlungsmittel akzeptieren,
mit Kryptowährung bezahlen können (derzeit BTC signet und USDC): Ein **Einkäufer** (*proxy shopper*) kauft den Artikel für Sie und verschickt ihn.

- Das Geld jeder Bestellung liegt in einem **2-von-3-Multisig** (Nutzer, Einkäufer, Treuhänder). Timelocks sorgen dafür, dass es immer bei jemandem ankommt.
- Nutzer installieren nichts: Sie nehmen über eine öffentliche Web-App teil. Die Schlüssel bleiben im Browser, und jede Signatur wird dort erstellt.
- Welchen Einkäufern und Treuhändern (*escrow*) man vertraut, ergibt sich aus der Signatur eines vom Nutzer gewählten **Koordinators** (*coordinator*) über die Listen der Betreiber (*operator*).

## Seiten

- [Funktionsweise]({{ '/de/overview/' | relative_url }}): Architektur, wie Vertrauen weitergegeben wird, Gebühren, Nur-Bargeld-Geschäfte, Risiko von Shops
- [Rollen]({{ '/de/roles/' | relative_url }}): was Nutzer, Einkäufer, Treuhänder, Betreiber und Koordinator jeweils tun
- [Erste Schritte]({{ '/de/quickstart/' | relative_url }}): das ganze Netz lokal betreiben, die Demo ausprobieren, in der Web-App bestellen, einen Knoten betreiben
- [Protokoll]({{ '/de/protocol/' | relative_url }}): das Wichtigste zu Nachrichten, Skripten, Safe und Timelocks

## Für KI-Agenten

- **Überall verdienen, ohne Erlaubnis:** Betreiben Sie Ihren eigenen Koordinator/Betreiber und tragen Sie sich selbst als Einkäufer in Ihrer Stadt ein — siehe [Erste Schritte §3]({{ '/de/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Pull Requests sind willkommen.

- MCP-Server `io.github.pad01g/proxy-shopping` (über den Treuhänder kaufen oder den Einstieg als Einkäufer planen) — siehe [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skills: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Als Einkäufer, Treuhänder, Betreiber oder Koordinator gelistet werden: ein Pull Request an [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Quellcode

Der Quellcode ist öffentlich (MIT).

| Repository | Inhalt |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Go-Knoten, Nostr-Relay, Verträge, fiktive Shops, das Browser-Automatisierungswerkzeug, die docker-compose-Testumgebung |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | Die Kernbibliothek für den Browser, die Web-App und die Demo-App |
