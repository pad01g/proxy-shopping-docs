---
layout: page
title: Cara kerja
permalink: /id/overview/
lang: id
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Apa fungsinya

Jika toko tempat Anda ingin berbelanja hanya menerima tunai atau metode pembayaran tertentu, Anda membayar dengan kripto
(saat ini **BTC signet** dan **USDC**) dan seorang **pembeli perantara** (proxy shopper) membelikan barang itu untuk Anda lalu mengirimkannya.

Uangnya masuk ke **multisig 2-dari-3** untuk setiap pesanan (pengguna (user), pembeli perantara, escrow/penengah).

- Saat barang tiba, pengguna dan pembeli perantara menandatangani pembayaran kepada pembeli perantara.
- Jika terjadi sengketa, escrow memutuskan pembagiannya bersama salah satu dari dua pihak lainnya.
- Bahkan jika seseorang berhenti merespons, **timelock** memastikan uang itu berakhir di salah satu pihak.

## Arsitektur

```
Browser (proxy-shopping-web)                Node Go (proxy-shopping-go)
  kunci dan tanda tangan di browser     shopper / escrow / operator / relay
        │ WSS (hanya keluar)                    │ WSS           │ libp2p
        ▼                                        ▼               ▼
   relay Nostr (ditunjuk oleh operator) ◀──▶  jaringan node Go
   = pintu masuk yang selalu online + kotak surat   (gossipsub; menembus NAT lewat circuit relay v2 dan DCUtR)
```

- **Pengguna tidak menjalankan apa pun.** Cukup membuka aplikasi web publik untuk bergabung.
  Kunci dibuat di browser dan tidak pernah keluar dari sana; setiap otorisasi (tanda tangan) terjadi di browser.
- Pengguna di balik NAT juga bisa terhubung, karena koneksi ke relay Nostr bersifat keluar (outbound).
- Pesan untuk pihak yang tidak selalu online (pengguna, escrow, operator, koordinator) menunggu dalam keadaan
  tetap terenkripsi di relay Nostr yang berfungsi sebagai kotak surat. Setiap pesan dikirim ke beberapa relay.
- Hanya pembeli perantara yang harus selalu online. Ia menjalankan node Go dan alat otomatisasi browser (shopper-bot) yang mengoperasikan toko.

## Alur kepercayaan

```
coordinator (pengguna memercayai kunci publiknya)
  └─ delegasi: "operator ini boleh menerbitkan daftar"
       └─ daftar: "di wilayah ini, shopper ini dan escrow ini bisa dipercaya bersama"
```

- Pengguna hanya memercayai **kunci publik koordinator** (coordinator). Dari sana kepercayaan meluas ke operator dan ke kombinasi pembeli perantara × escrow.
- Daftar memiliki versi dan tidak pernah kedaluwarsa. Mengeluarkan seseorang berarti menerbitkan versi baru.
- Pengguna "memberhentikan" koordinator dengan menghapusnya dari pengaturan.

## Biaya

| Kepada siapa | Bisa dipaksakan? | Caranya |
|---|---|---|
| pembeli perantara | ya | termasuk dalam penawaran harga |
| escrow (di muka) | ya | dibayar langsung ke escrow bersamaan dengan pendanaan. **Escrow tidak wajib menengahi pesanan tanpa biaya di muka** |
| escrow (sengketa) | ya | diambil dari pembagian putusan |
| operator / koordinator | tidak | disepakati di luar protokol, misalnya biaya pencantuman. Yang dijual sebuah daftar adalah "bisa ditemukan" |

Jaminan (bond) escrow dan penyitaannya bukan bagian dari protokol.
Keduanya dianggap sebagai ketentuan yang disepakati sendiri oleh operator dan escrow; tersedia kontrak acuan.

## Toko khusus tunai

Lokasi toko dinyatakan dengan kode wilayah (misalnya `JP-13-13104` = Shinjuku, Tokyo).
Pembeli perantara menyatakan wilayah tempat ia bisa datang dan membayar tunai, dan hanya menerima pesanan untuk toko di sana.

## Risiko toko

Sebelum menerima pesanan, pembeli perantara memberi skor pada toko:
apakah toko ada di allowlist pembeli perantara, apakah memakai HTTPS, dan apakah checkout-nya memakai gateway pembayaran yang dikenal.
Mengirim data kartu ke sembarang situs itu berisiko, sehingga pesanan untuk toko berskor rendah ditolak.
