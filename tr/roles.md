---
layout: page
title: Roller
permalink: /tr/roles/
lang: tr
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Rol | Sürekli çevrimiçi | Kullandığı | Yaptığı | Hile yaparsa |
|---|---|---|---|---|
| **kullanıcı** (user) | hayır | tarayıcı | sipariş verir, fonlar, teslim alındığını onaylar, anlaşmazlık açar | ― |
| **vekil alışverişçi** (proxy shopper) | **evet** | Go düğümü + shopper-bot | fiyat teklifi verir, kullanıcı adına satın alır, kargoyu bildirir; ücret kazanır | emanetçinin kararında payını kaybeder; operatör onu listeden çıkarır |
| **emanetçi** (escrow) | hayır | tarayıcı ya da Go düğümü | anlaşmazlıklarda T1'den önce karar verir | operatör onu listeden çıkarır; şartlarına bağlı olarak teminatına el konur ve tazminat olarak kullanılır |
| **operatör** (operator) | hayır | tarayıcı, Go düğümü ya da `psctl` | bölge bazında güvenilir alışverişçi × emanetçi eşleşmelerinin listesini imzalar; şikâyetleri alır | koordinatör yetki devrini geri alır |
| **koordinatör** (coordinator) | hayır | tarayıcı ya da `psctl` | operatörlere yetki devirlerini imzalar | kullanıcılar onu ayarlarından kaldırır (görevden alma) |

## Kullanıcı

1. Web uygulamasını açın, bir mnemonic (12 kelime) oluşturun, not edin ve anahtarı şifreleyecek bir parola (passphrase) seçin.
2. Mağazanın URL'sini, mağazanın bölgesini ve ürünleri girin.
3. Önerilen alışverişçi × emanetçi eşleşmelerinden birini seçin ve siparişi verin.
4. Bir fiyat teklifi gelir. Kuru kendi kaynaklarınızdan %3'ten fazla farklıysa bir uyarı notu, %10'un üzerindeyse güçlü bir uyarı alırsınız.
5. Kabul edin ve fonlayın. Para, multisig ile emanetçinin peşin ücreti arasında bölünür.
6. Ürün ulaştığında "received" düğmesine basın (tutarı ve alıcıyı gösteren bir ekranın ardından kısmi imzanız alışverişçiye gider).
7. Ulaşmazsa bir anlaşmazlık açın. Uygulama kanıtları sizin için toplar.

## Vekil alışverişçi

- Go düğümünü `role: shopper` ile çalıştırın ve shopper-bot'u (tarayıcı otomasyon aracı) bağlayın.
- Ödeme yöntemlerini, para birimlerini, nakit ödeyebileceğiniz bölgeleri, ücretinizi ve mağaza risk politikanızı yapılandırın.
- Kart bilgileri yalnızca shopper-bot'un içinde durur; düğüme ya da ağa asla verilmez.
- Kullanıcı teslim aldığını hiç onaylamazsa ve T1 geçerse, parayı tek başınıza alabilirsiniz.

## Emanetçi

- Yalnızca peşin ücretini ödemiş siparişlerin anlaşmazlıklarında karar verir.
- Normalde teslimat adresini okuyamaz. Anlaşmazlık durumunda alışverişçi (ya da kullanıcı) ona şifre çözme anahtarını verir.
- Karar her iki tarafa da imzalı bir işlem olarak ulaşır; taraflardan biri karşı imza (countersign) attığında kesinleşir.

## Operatör

- Bölgelerini (bir ya da daha fazla), eşleşmeler listesini ve önerdiği röleleri ve zincir uç noktalarını imzalar.
- Bir şikâyet geldiğinde, kuralı çiğneyeni çıkararak listenin yeni bir sürümünü yayımlar. Teminat, emanetçiyle kararlaştırdığı şartlara göre işlem görür.

## Koordinatör

- Operatörlere yetki devri verir. Geri almak, `revoked` içeren yeni bir sürüm yayımlamak demektir.

## Listelenmek (kayıt deposu)

Proje sorumlusunun güven kaydı (registry), GitHub deposu
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry)'dir. **Birleştirilen (merge edilen) bir pull request
onay demektir:** her birleştirmeden sonra CI, yeni yetki devirlerini ve listeleri kayıt deposunun anahtarlarıyla imzalar ve yayımlar
(herkese açık Nostr rölelerine ve https://pad01g.github.io/proxy-shopping-registry/events.json adresine).

| Olmak istediğiniz | Pull request'e ekleyin | Birleştirme ne yapar |
|---|---|---|
| alışverişçi | `shoppers/<name>.json` (pk, iletişim, açıklama, nakit bölgeleri, ödemeler, birlikte çalıştığınız emanetçiler) | kayıt deposunun operatörü alışverişçi × emanetçi eşleşmelerinizi listeler |
| emanetçi | `escrows/<name>.json` (pk, iletişim, açıklama, SLA gün sayısı) | alışverişçiler sizi belirtebilir; eşleşmeleriniz görünür |
| operatör | `operators/<name>.json` (pk, iletişim, açıklama, bölgeler) | kayıt deposunun koordinatörü size yetki devreder; ardından kendi listelerinizi siz imzalarsınız |
| koordinatör | `coordinators/<name>.json` (pk, iletişim, açıklama) | uygulamaların kullanıcılara sunduğu koordinatör dizininde görünürsünüz (her kullanıcı yine kime güveneceğini kendisi seçer) |

`pk`, Nostr açık anahtarınızdır (64 onaltılık karakter): web uygulaması onu Ayarlar'da gösterir, `psctl keys --mnemonic-file …`
de yazdırır. Birini çıkarmak, dosyasını bir gerekçeyle `revoked/` klasörüne taşıyan bir pull request'tir. Kayıt deposunun
README dosyasında tam dosya biçimleri ve komutlar bulunur.
