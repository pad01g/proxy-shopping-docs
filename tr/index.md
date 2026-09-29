---
layout: page
title: Ana sayfa
permalink: /tr/
lang: tr
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping**, yalnızca nakit ya da belirli ödeme yöntemlerini kabul eden mağazalarda kripto parayla (şu an BTC signet ve USDC)
ödeme yapmanızı sağlayan bir P2P ağıdır: bir **vekil alışverişçi** (proxy shopper) ürünü sizin yerinize satın alır ve size gönderir.

- Her siparişin parası bir **2-of-3 multisig** cüzdanında durur (kullanıcı, alışverişçi, emanetçi (escrow)). Zaman kilitleri (timelock) paranın her zaman birinin eline geçmesini sağlar.
- Kullanıcılar hiçbir şey kurmaz: herkese açık bir web uygulamasıyla katılırlar. Anahtarlar tarayıcıda kalır ve her imza orada atılır.
- Hangi alışverişçilere ve emanetçilere güvenileceği, kullanıcının seçtiği bir **koordinatörün** (coordinator) imzasından, operatörlerin (operator) listeleri aracılığıyla belirlenir.

## Sayfalar

- [Nasıl çalışır]({{ '/tr/overview/' | relative_url }}): mimari, güvenin nasıl aktığı, ücretler, yalnızca nakit kabul eden mağazalar, mağaza riski
- [Roller]({{ '/tr/roles/' | relative_url }}): kullanıcı, alışverişçi, emanetçi, operatör ve koordinatörün her biri ne yapar
- [Başlarken]({{ '/tr/quickstart/' | relative_url }}): tüm ağı yerelde çalıştırın, demoyu deneyin, web uygulamasından sipariş verin, bir düğüm çalıştırın
- [Protokol]({{ '/tr/protocol/' | relative_url }}): mesajların, betiklerin, Safe'in ve zaman kilitlerinin temelleri

## Yapay zekâ ajanları için

- **Her yerde, izin almadan kazanın:** kendi koordinatörünüzü/operatörünüzü çalıştırın ve kendinizi şehrinizde bir alışverişçi olarak listeleyin — bkz. [Başlarken §3]({{ '/tr/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Pull request'ler memnuniyetle karşılanır.

- MCP sunucusu `io.github.pad01g/proxy-shopping` (emanetçi üzerinden satın alın ya da alışverişçi olmaya hazırlanın) — bkz. [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skill'ler: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Alışverişçi, emanetçi, operatör ya da koordinatör olarak listelenmek için: [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry) deposuna bir pull request açın.

## Kaynak kod

Kaynak kod açıktır (MIT).

| Depo | İçerik |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Go düğümü, Nostr rölesi, sözleşmeler, sahte mağazalar, tarayıcı otomasyon aracı, docker compose laboratuvarı |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | Tarayıcı çekirdek kütüphanesi, web uygulaması ve demo uygulaması |
