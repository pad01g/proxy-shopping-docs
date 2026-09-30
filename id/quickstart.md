---
layout: page
title: Memulai
permalink: /id/quickstart/
lang: id
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Jalankan seluruh jaringan secara lokal (lab)

Satu berkas docker compose menyalakan jaringan tertutup. Jaringan ini tidak terhubung ke internet.

- dua relay Nostr
- satu relay libp2p
- bitcoind di signet khusus, dan API yang kompatibel dengan Esplora
- anvil (EVM) dengan Safe dan kontrak lainnya
- empat toko tiruan dan satu gateway pembayaran kartu tiruan
- dua pembeli perantara (shopper), tiga escrow (satu di balik NAT), dua operator
- aplikasi web publik dan aplikasi demo

Kebutuhan: Docker (sekitar 8 GB memori).

Letakkan proxy-shopping-go dan proxy-shopping-web berdampingan di direktori induk yang sama, lalu jalankan:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Untuk mulai dari awal, jalankan `docker compose down -v` terlebih dahulu agar data tersimpan milik node dan relay ikut terhapus
(chain dimulai dari nol setiap kali dinyalakan, jadi sisa data pesanan akan tidak cocok dengannya).

### Jalankan tes e2e

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Hasilnya masuk ke `e2e/results/e2e-latest.md`. Runner bekerja dari dalam NAT (jaringan `home`),
sehingga setiap skenario mengasumsikan pengguna berada di balik NAT.

| id | Skenario |
|---|---|
| a | Jalur normal: BTC (toko JPY) dan USDC (toko USD). Kurs dalam penawaran harga diperiksa, escrow menerima biaya di muka, pembeli perantara dibayar. Penawaran dengan kurs yang jauh menyimpang mendapat peringatan keras |
| b | Pengiriman gagal dan pengguna membuka sengketa. Escrow yang jujur memutuskan pengembalian dana dan pengguna ikut menandatangani (BTC). Escrow bisa mendekripsi alamat dan menerima tangkapan layar sebagai lampiran |
| c | Escrow memutuskan secara tidak jujur. Pengguna melaporkannya; operator mengeluarkannya dari daftar dan, sesuai ketentuan mereka, menyita jaminannya sebagai kompensasi (USDC) |
| d | Koordinator (coordinator) mencabut delegasi seorang operator, dan kombinasi dari daftar tersebut hilang dari penawaran |
| e | Browser di balik NAT memesan lewat aplikasi web publik (pesan 1:1 lewat relay Nostr). Kueri status sampai ke node escrow di balik NAT melalui libp2p circuit relay |
| f | Pembeli perantara menolak toko yang ia nilai berisiko, dan toko khusus tunai di luar wilayahnya. Pembeli perantara di wilayah itu menerima pesanan tunai tersebut dan mengirimkannya |
| g | Setelah T1, pembeli perantara bisa mengambil dana sendiri (BTC) |
| h | Jika pembeli perantara menghilang, pengguna mengambil kembali dananya sendiri setelah T2 (USDC) |
| i | Sengketa USDC dengan escrow yang jujur. Pengembalian dana tetap terlaksana meskipun seseorang mengirim sejumlah kecil ke Safe sebelum putusan |
| j | Pesanan yang tidak bisa dipenuhi toko (stok habis). Pembeli perantara menawarkan pengembalian dana kooperatif; pengguna memeriksanya dan menerimanya |
| k | Jika pembeli perantara menghilang, pengguna mengambil kembali dananya sendiri setelah T2 (BTC) |

### Coba seluruh alurnya di demo

**Coba di peramban Anda: <https://pad01g.github.io/proxy-shopping-web/> (semuanya disimulasikan di dalam halaman).**

Selagi lab berjalan, buka demo di `http://localhost:8888/` (tidak perlu berkas hosts maupun sertifikat).
Dalam satu layar, pengguna, escrow, operator, dan koordinator masing-masing punya kunci sendiri (di penyimpanan lokal browser),
dan pembeli perantaranya adalah node Go yang selalu online itu sendiri. Jalurnya nyata: relay Nostr, bitcoind, anvil, node Go.

- Pilih skenario di bagian atas. Panduan di sebelah kiri menjelaskan siapa melakukan apa berikutnya, dan mengapa.
- "Ke langkah ini" berpindah ke tab peran tersebut dan menunjuk tombol yang harus ditekan. Formulir sudah terisi sesuai skenario, jadi Anda cukup mengklik dan mengonfirmasi.
- Skenario: jalur normal (BTC / USDC), pengiriman gagal dan pengembalian dana, stok habis dan pengembalian dana kooperatif, toko berisiko ditolak, escrow tidak jujur dilaporkan dan dikeluarkan dari daftar, serta pengembalian dana setelah T2 saat pembeli perantara menghilang.
- Anda juga bisa membuka satu jendela per peran, misalnya `?role=user` dan `?role=escrow,operator,coordinator` (jendela dari browser yang sama berbagi kunci dan kemajuan).
- Dalam kenyataan, setiap peran berada di tempat lain, di browsernya sendiri. Demo hanya memisahkan kuncinya.
- Khusus lab: siapa pun yang bisa menjangkau port 8888 dapat memakai faucet, mining, lompatan waktu, dan admin API pembeli perantara (hanya terikat ke 127.0.0.1).

Ada juga e2e yang menjalankan demo dengan mengikuti panduannya: `docker compose run --rm runner demo`.

### Memakai aplikasi web

Nama-nama lab (`*.test`) hanya bisa di-resolve di dalam kontainer.
Untuk memakai aplikasi web dari browser Anda:

- Buka port 443 kontainer `edge` secara lokal (misalnya di `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Ini juga membuka `faucet.test` dan `evm.test` yang khusus lab (siapa pun bisa mencetak saldo dan memajukan waktu), jadi buka hanya di localhost.
- Arahkan `app.test` dan nama lainnya ke 127.0.0.1 di berkas hosts Anda.

Sertifikatnya self-signed.

1. Buka `https://app.test/` lalu pilih "buat baru" atau "pulihkan dari kata-kata".
   Pilih frasa sandi (minimal 8 karakter) yang mengenkripsi kunci (Anda juga bisa secara eksplisit memilih untuk tidak mengenkripsinya, atau memakai ekstensi NIP-07).
   Lain kali, buka kunci dengan frasa sandi tersebut.
2. Di bagian "pesanan", masukkan URL toko (misalnya `https://safe-shop.test/`), wilayah toko (misalnya `JP-13-13104`) dan barangnya (misalnya `A-100`), lalu cari penawaran.
3. Pilih kombinasi pembeli perantara × escrow, masukkan alamat pengiriman, dan buat pesanan.
4. Saat penawaran harga datang, periksa selisih kurs dan hasil pemeriksaan alamat multisig, lalu terima.
5. Di lab, isi dompet Anda dengan "ambil dari faucet", lalu tekan "danai multisig".
   Setiap tindakan yang memindahkan uang (pendanaan, pembayaran, tanda tangan balasan) melewati layar yang menampilkan jumlah dan penerima.
6. Saat barang tiba, tekan "diterima" untuk membayar pembeli perantara. "Selesai" baru muncul setelah pembayaran terkonfirmasi di chain.

## 2. Menjalankan di jaringan publik

Arsitekturnya sama dengan lab. Perbedaannya:

- gunakan sertifikat ACME;
- jangan gunakan kunci di `lab/keys`;
- gunakan signet dan chain EVM yang sungguhan.

### Pembeli perantara (shopper)

Selalu online; menjalankan node Go dan shopper-bot.

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

- Data kartu hanya dimasukkan ke berkas konfigurasi shopper-bot (`BOT_CARDS_FILE`); node tidak pernah menerimanya.
- Langkah-langkah untuk setiap toko ditulis sebagai driver shopper-bot. Driver yang memakai AI untuk mengoperasikan toko memiliki masukan dan keluaran yang sama (`PurchaseRequest` / `PurchaseResult`).

### Escrow / operator / koordinator

Peran-peran ini tidak harus selalu online.

- Mereka bisa memakai layar "Escrow", "Operator", dan "Coordinator" di aplikasi web.
- Untuk berjalan terus-menerus, jalankan node Go dengan `role: escrow` / `role: operator`.
- Untuk menandatangani saja, `psctl` juga bisa.

Versi (`v`) secara bawaan adalah waktu UNIX, jadi biasanya tidak perlu Anda berikan.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Jalankan peran jaringan Anda sendiri, di mana saja, tanpa izin

proxy-shopping tidak punya operator pusat dan tidak ada pendaftaran. Rantai kepercayaannya hanyalah kunci dan event Nostr yang ditandatangani,
sehingga Anda (atau agen AI yang bekerja untuk Anda) bisa memulai pasar lokal di kota Anda sendiri:

1. **Jadilah koordinator:** buat sebuah kunci (`psctl keys --mnemonic-file coordinator.mnemonic`). Hanya itulah koordinator.
2. **Delegasikan kepada operator** (kunci kedua milik Anda sendiri, atau seseorang yang Anda percaya):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Cantumkan pembeli perantara dan escrow untuk wilayah Anda** — Anda sendiri sebagai pembeli perantara, seorang teman sebagai escrow:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Minta orang-orang memercayai kunci koordinator Anda** (tambahkan di Pengaturan aplikasi web, atau `PS_COORDINATORS` untuk server MCP).
   Agar ditemukan semua orang, buka pull request ke [registri](https://github.com/pad01g/proxy-shopping-registry)
   yang menambahkan `coordinators/<name>.json`; untuk menjalankan persetujuan bagi komunitas Anda sendiri dengan cara yang sama, fork registrinya.

**Di mana menjalankannya:** satu mesin di rumah sudah cukup. Node hanya membuat koneksi keluar (relay Nostr, dan libp2p
circuit relay ketika berada di balik NAT), jadi Anda tidak perlu membuka port. Gunakan [Tailscale](https://tailscale.com/) atau VPN apa pun untuk menjangkau
admin API node dan aplikasi web dari ponsel saat Anda bepergian; VPS kecil juga bisa.

**Apa yang bisa dilakukan agen AI:** agen bisa menjalankan rutinitas operator dan pembeli perantara — memantau pesanan lewat admin API
node atau server MCP, menjaga daftar tetap mutakhir, mengendalikan shopper-bot untuk toko yang menerima kartu, melaporkan masalah — sedangkan pengetahuan
lokal Anda sendiri (toko mana, wilayah mana, toko khusus tunai yang bisa Anda datangi dengan berjalan kaki) adalah bagian yang tidak bisa ditawarkan orang lain.
Skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) memandu agen melakukannya.

Bersikaplah jujur kepada pengguna Anda: jaringan publik ini masih baru dan berjalan di BTC signet (koin uji coba), jadi penghasilan baru datang ketika orang-orang mulai memakainya.

## Berkontribusi {#contributing}

**Pull request dipersilakan** — di semua repositori: driver toko untuk shopper-bot, metode pembayaran dan chain baru,
terjemahan dokumentasi ini, tinjauan protokol, perbaikan bug, dan pencantuman di
[registri](https://github.com/pad01g/proxy-shopping-registry). Buka issue atau pull request di GitHub.
