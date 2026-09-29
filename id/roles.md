---
layout: page
title: Peran
permalink: /id/roles/
lang: id
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Peran | Selalu online | Memakai | Tugas | Jika berbuat curang |
|---|---|---|---|---|
| **pengguna** (user) | tidak | browser | memesan, mendanai, mengonfirmasi penerimaan, membuka sengketa | ― |
| **pembeli perantara** (proxy shopper) | **ya** | node Go + shopper-bot | memberi penawaran harga, membeli atas nama pengguna, melaporkan pengiriman; mendapat biaya | kehilangan bagiannya dalam putusan escrow; operator mengeluarkannya dari daftar |
| **escrow** (penengah) | tidak | browser atau node Go | memutuskan sengketa sebelum T1 | operator mengeluarkannya dari daftar; tergantung ketentuan mereka, jaminannya disita dan dipakai sebagai kompensasi |
| **operator** | tidak | browser, node Go, atau `psctl` | menandatangani, per wilayah, daftar kombinasi pembeli perantara × escrow yang tepercaya; menerima laporan | koordinator mencabut delegasinya |
| **koordinator** (coordinator) | tidak | browser atau `psctl` | menandatangani delegasi kepada operator | pengguna menghapusnya dari pengaturan (pemberhentian) |

## Pengguna

1. Buka aplikasi web, buat mnemonic (12 kata), catat, dan pilih frasa sandi (passphrase) yang mengenkripsi kunci.
2. Masukkan URL toko, wilayah toko, dan barangnya.
3. Pilih salah satu kombinasi pembeli perantara × escrow yang ditawarkan, lalu buat pesanan.
4. Penawaran harga datang. Jika kursnya berbeda lebih dari 3% dari sumber Anda sendiri, Anda mendapat peringatan; di atas 10%, peringatan keras.
5. Terima dan danai. Uangnya dibagi antara multisig dan biaya di muka untuk escrow.
6. Saat barang tiba, tekan "diterima" (setelah layar yang menampilkan jumlah dan penerima, tanda tangan parsial Anda dikirim ke pembeli perantara).
7. Jika barang tidak tiba, buka sengketa. Aplikasi mengumpulkan buktinya untuk Anda.

## Pembeli perantara

- Jalankan node Go dengan `role: shopper` dan hubungkan shopper-bot (alat otomatisasi browser).
- Atur metode pembayaran, mata uang, wilayah tempat Anda bisa membayar tunai, biaya Anda, dan kebijakan risiko toko Anda.
- Data kartu hanya ada di dalam shopper-bot; tidak pernah diberikan ke node maupun jaringan.
- Jika pengguna tidak pernah mengonfirmasi penerimaan dan T1 lewat, Anda bisa mengambil dananya sendiri.

## Escrow

- Hanya memutuskan sengketa pesanan yang telah membayar biaya di mukanya.
- Biasanya ia tidak bisa membaca alamat pengiriman. Saat sengketa, pembeli perantara (atau pengguna) memberinya kunci dekripsi.
- Putusan sampai ke kedua pihak sebagai transaksi bertanda tangan; putusan menjadi final setelah salah satu dari mereka ikut menandatanganinya.

## Operator

- Menandatangani wilayahnya (satu atau lebih), daftar kombinasi, serta relay dan endpoint chain yang ia rekomendasikan.
- Jika ada laporan, ia menerbitkan versi baru daftar tanpa pelanggar. Jaminan mengikuti ketentuan yang ia sepakati dengan escrow.

## Koordinator

- Menerbitkan delegasi kepada operator. Mencabut berarti menerbitkan versi baru dengan `revoked`.

## Agar tercantum (registri)

Registri kepercayaan milik pengelola adalah repositori GitHub
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Pull request yang di-merge adalah
persetujuannya:** setelah setiap merge, CI menandatangani delegasi dan daftar baru dengan kunci registri lalu menerbitkannya
(ke relay Nostr publik dan ke https://pad01g.github.io/proxy-shopping-registry/events.json).

| Anda ingin menjadi | Tambahkan dalam pull request | Hasil merge |
|---|---|---|
| pembeli perantara | `shoppers/<name>.json` (pk, kontak, deskripsi, wilayah tunai, pembayaran, escrow yang bekerja sama dengan Anda) | operator registri mencantumkan kombinasi pembeli perantara × escrow Anda |
| escrow | `escrows/<name>.json` (pk, kontak, deskripsi, hari SLA) | pembeli perantara bisa menunjuk Anda; kombinasi Anda muncul |
| operator | `operators/<name>.json` (pk, kontak, deskripsi, wilayah) | koordinator registri mendelegasikan kepada Anda; selanjutnya Anda menandatangani daftar Anda sendiri |
| koordinator | `coordinators/<name>.json` (pk, kontak, deskripsi) | Anda muncul di direktori koordinator yang ditawarkan aplikasi kepada pengguna (setiap pengguna tetap memilih sendiri siapa yang dipercaya) |

`pk` adalah kunci publik Nostr Anda (64 karakter hex): aplikasi web menampilkannya di Pengaturan, dan `psctl keys --mnemonic-file …`
mencetaknya. Mengeluarkan seseorang dilakukan dengan pull request yang memindahkan berkasnya ke `revoked/` beserta alasannya. README registri
memuat format berkas dan perintah yang persis.
