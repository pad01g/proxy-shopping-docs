---
layout: page
title: Protokol
permalink: /id/protocol/
lang: id
ref: protocol
nav_order: 5
---

{% include langnav.html %}

Spesifikasi lengkapnya adalah `docs/spec.md` di proxy-shopping-go. Halaman ini merangkum intinya.

## Kunci

Semua kunci diturunkan dari satu mnemonic (BIP39), satu untuk setiap keperluan (semuanya secp256k1).

| Keperluan | Path |
|---|---|
| identitas (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| kunci pesanan BTC (pengguna (user), pembeli perantara (shopper)) | `m/7333'/1'/{idx}'` (idx berasal dari hash ID pesanan) |
| kunci pesanan BTC (escrow) | `m/7333'/2'/{idx}`. xpub-nya publik, sehingga pihak lain bisa menurunkannya saat escrow sedang offline |
| dompet BTC | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Kepercayaan

| kind | Ditandatangani oleh | Isi |
|---|---|---|
| 30500 | koordinator (coordinator) | delegasi kepada operator; dicabut dengan `revoked` |
| 30501 | operator | daftar wilayah × pembeli perantara × escrow, relay dan endpoint chain yang direkomendasikan |
| 30502 / 30503 | pembeli perantara / escrow | profil (biaya, wilayah tunai, xpub, alamat pembayaran) |
| 10050 | semua | relay tempat seseorang menerima pesan |

Versi pada tag `v` menentukan mana yang lebih baru (biasanya waktu UNIX saat penandatanganan). Tidak ada yang kedaluwarsa.
Event berjalan lewat relay Nostr maupun libp2p gossipsub; node Go meneruskan apa yang diterima di satu jalur ke jalur lainnya.

## Pesan

Pesan dibungkus dengan NIP-59 (gift wrap → seal → isi).
Isinya adalah event yang **ditandatangani** (kind 5400), agar escrow bisa memverifikasinya sebagai pihak ketiga dalam sengketa.

- Pesan dikirim ke setidaknya k (bawaan 2) relay kotak masuk milik penerima.
- Penerima membalas dengan `ack`; pengirim mengirim ulang sampai menerimanya.
- Satu pesan paling besar 28000 byte. Bukti besar seperti tangkapan layar hanya dirujuk lewat hash-nya;
  saat sengketa, bukti itu dikirim ke escrow dalam potongan-potongan sebagai pesan `attachment`.

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

T1 < T2 (tinggi blok). Jika pengguna tidak pernah mengonfirmasi penerimaan dan T1 lewat, pembeli perantara bisa mengambil dana sendiri.
Jika pembeli perantara menghilang, pengguna bisa mengambilnya kembali sendiri setelah T2.
Escrow harus memutuskan sengketa sebelum T1.

## USDC (Safe v1.4.1)

- Setiap pesanan mendapat sebuah Safe dengan tiga pemilik dan ambang dua tanda tangan.
- Alamatnya berasal dari CREATE2, sehingga bisa dihitung terlebih dahulu dari ID pesanan.
- Timelock diimplementasikan sebagai modul Safe (`PSEscrowModule`).
  - Setelah t1, pembeli perantara bisa mengambil dana dengan `claimByShopper`.
  - Setelah t2, pengguna bisa mengambilnya kembali dengan `refundToUser`.
- Pembayaran dan putusan adalah SafeTx yang ditandatangani dengan EIP-712. Pembagian dalam putusan memakai `MultiSendCallOnly`.

## Mengikat identitas ke kunci

Setiap permintaan pesanan (`order.request`) membawa `key_proof`:
tanda tangan oleh kunci chain yang masuk ke multisig (kunci pesanan BTC atau akun EVM),
yang menunjukkan bahwa kunci itu milik orang yang sama dengan identitas Nostr-nya.
Tanpa ini, identitas lain bisa menyalin kunci publik pengguna ke dalam permintaannya sendiri dan mengaku kepada escrow bahwa dialah penggunanya.

## Alamat pengiriman

Alamat dienkripsi dengan kunci sekali pakai K (XChaCha20-Poly1305).

- K dibungkus dengan NIP-44 untuk pembeli perantara dan, secara terpisah, untuk escrow.
- Salinan untuk escrow bukan bagian dari permintaan: salinan itu diserahkan ke pembeli perantara dalam `order.escrow_key` (permintaan hanya membawa hash-nya).
  Pembeli perantara (atau pengguna) menyerahkannya ke escrow saat sengketa, sehingga escrow tidak bisa membaca alamat pesanan yang tidak bersengketa.

## Wilayah

Kode dicocokkan berdasarkan awalan: `JP` > `JP-13` (Tokyo) > `JP-13-13104` (Shinjuku).
Tingkat kota/kabupaten memakai kode pemerintah daerah Jepang.
