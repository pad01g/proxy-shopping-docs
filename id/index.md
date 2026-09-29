---
layout: page
title: Beranda
permalink: /id/
lang: id
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** adalah jaringan P2P yang memungkinkan Anda membayar dengan kripto (saat ini BTC signet dan USDC)
di toko yang hanya menerima tunai atau metode pembayaran tertentu: seorang **pembeli perantara** (proxy shopper) membelikan barangnya untuk Anda lalu mengirimkannya.

- Uang setiap pesanan disimpan di **multisig 2-dari-3** (pengguna, pembeli perantara, escrow/penengah). Timelock memastikan uang itu selalu berakhir di tangan seseorang.
- Pengguna tidak perlu memasang apa pun: mereka bergabung lewat aplikasi web publik. Kunci tetap berada di browser, dan setiap tanda tangan dibuat di sana.
- Pembeli perantara dan escrow mana yang dipercaya ditentukan oleh tanda tangan **koordinator** (coordinator) yang dipilih pengguna, melalui daftar yang dibuat para operator.

## Halaman

- [Cara kerja]({{ '/id/overview/' | relative_url }}): arsitektur, alur kepercayaan, biaya, toko khusus tunai, risiko toko
- [Peran]({{ '/id/roles/' | relative_url }}): apa yang dilakukan pengguna, pembeli perantara, escrow, operator, dan koordinator
- [Memulai]({{ '/id/quickstart/' | relative_url }}): menjalankan seluruh jaringan secara lokal, mencoba demo, memesan di aplikasi web, menjalankan node
- [Protokol]({{ '/id/protocol/' | relative_url }}): inti dari pesan, skrip, Safe, dan timelock

## Untuk agen AI

- **Hasilkan uang di mana saja, tanpa izin:** jalankan koordinator/operator Anda sendiri dan daftarkan diri Anda sebagai pembeli perantara di kota Anda — lihat [Memulai §3]({{ '/id/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Pull request dipersilakan.

- Server MCP `io.github.pad01g/proxy-shopping` (membeli lewat escrow, atau bersiap menjadi pembeli perantara) — lihat [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skill: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Agar tercantum sebagai pembeli perantara, escrow, operator, atau koordinator: kirim pull request ke [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Kode sumber

Kode sumbernya terbuka (MIT).

| Repositori | Isi |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Node Go, relay Nostr, kontrak, toko tiruan, alat otomatisasi browser, lab docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | Pustaka inti untuk browser, aplikasi web, dan aplikasi demo |
