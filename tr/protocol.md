---
layout: page
title: Protokol
permalink: /tr/protocol/
lang: tr
ref: protocol
nav_order: 5
---

{% include langnav.html %}

Tam belirtim proxy-shopping-go içindeki `docs/spec.md` dosyasıdır. Bu sayfa temel noktaları özetler.

## Anahtarlar

Tüm anahtarlar tek bir mnemonic'ten (BIP39) türetilir; her amaç için bir anahtar (hepsi secp256k1).

| Amaç | Yol |
|---|---|
| kimlik (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| BTC sipariş anahtarı (kullanıcı, alışverişçi) | `m/7333'/1'/{idx}'` (idx, sipariş kimliğinin hash'inden gelir) |
| BTC sipariş anahtarı (emanetçi) | `m/7333'/2'/{idx}`. xpub herkese açıktır, bu yüzden emanetçi çevrimdışıyken başkaları da türetebilir |
| BTC cüzdanı | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Güven

| kind | İmzalayan | İçerik |
|---|---|---|
| 30500 | koordinatör (coordinator) | bir operatöre yetki devri; `revoked` ile geri alınır |
| 30501 | operatör (operator) | bölge × alışverişçi × emanetçi listesi, önerilen röleler ve zincir uç noktaları |
| 30502 / 30503 | alışverişçi (shopper) / emanetçi (escrow) | profil (ücretler, nakit bölgeleri, xpub, ödeme adresleri) |
| 10050 | herkes | kişinin mesaj aldığı röleler |

Hangisinin daha yeni olduğuna `v` etiketindeki sürüm karar verir (normalde imzalama anındaki UNIX zamanı). Hiçbir şeyin süresi dolmaz.
Olaylar hem Nostr röleleri hem de libp2p gossipsub üzerinden taşınır; Go düğümleri birinden aldıklarını diğerine iletir.

## Mesajlar

Mesajlar NIP-59 ile sarılır (gift wrap → seal → içerik).
İçerik **imzalı** bir olaydır (kind 5400); böylece emanetçi bir anlaşmazlıkta onu üçüncü taraf olarak doğrulayabilir.

- Bir mesaj, alıcının gelen kutusu rölelerinden en az k tanesine (varsayılan 2) gider.
- Alıcı `ack` ile yanıt verir; gönderen, bunu alana kadar yeniden gönderir.
- Bir mesaj en fazla 28000 bayttır. Ekran görüntüleri gibi büyük kanıtlara yalnızca hash ile atıf yapılır;
  anlaşmazlıkta bunlar emanetçiye `attachment` mesajları olarak parça parça gönderilir.

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

T1 < T2 (blok yükseklikleri). Kullanıcı teslim aldığını hiç onaylamazsa ve T1 geçerse, alışverişçi parayı tek başına alabilir.
Alışverişçi ortadan kaybolursa, kullanıcı T2'den sonra parayı tek başına geri alabilir.
Emanetçi bir anlaşmazlıkta T1'den önce karar vermelidir.

## USDC (Safe v1.4.1)

- Her sipariş, üç sahibi ve ikilik eşiği olan bir Safe alır.
- Adresi CREATE2'den gelir, bu yüzden sipariş kimliğinden önceden hesaplanabilir.
- Zaman kilitleri bir Safe modülü (`PSEscrowModule`) olarak uygulanmıştır.
  - t1'den sonra alışverişçi parayı `claimByShopper` ile alabilir.
  - t2'den sonra kullanıcı parayı `refundToUser` ile geri alabilir.
- Ödemeler ve kararlar EIP-712 ile imzalanmış SafeTx'lerdir. Bir kararın paylaşımı `MultiSendCallOnly` kullanır.

## Kimliği anahtarlara bağlamak

Her sipariş isteği (`order.request`) bir `key_proof` taşır:
multisig'e giren zincir anahtarıyla (BTC sipariş anahtarı ya da EVM hesabı) atılmış,
bu anahtarın Nostr kimliğiyle aynı kişiye ait olduğunu gösteren bir imza.
Bu olmasaydı, başka bir kimlik kullanıcının açık anahtarlarını kendi isteğine kopyalayıp emanetçiye kullanıcının kendisi olduğunu iddia edebilirdi.

## Teslimat adresi

Adres tek kullanımlık bir K anahtarıyla şifrelenir (XChaCha20-Poly1305).

- K, alışverişçi için ve ayrıca emanetçi için NIP-44 ile sarılır.
- Emanetçinin kopyası isteğin parçası değildir: alışverişçiye `order.escrow_key` içinde verilir (istek yalnızca onun hash'ini taşır).
  Alışverişçi (ya da kullanıcı) bunu anlaşmazlıkta emanetçiye iletir; bu yüzden emanetçi anlaşmazlık olmayan siparişlerin adresini okuyamaz.

## Bölgeler

Kodlar önek ile eşleştirilir: `JP` > `JP-13` (Tokyo) > `JP-13-13104` (Shinjuku).
Belediye düzeyi, Japonya'nın yerel yönetim kodlarını kullanır.
