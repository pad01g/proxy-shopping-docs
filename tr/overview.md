---
layout: page
title: Nasıl çalışır
permalink: /tr/overview/
lang: tr
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Ne yapar

Alışveriş yapmak istediğiniz mağaza yalnızca nakit ya da belirli bir ödeme yöntemi kabul ediyorsa, siz kripto parayla ödersiniz
(şu an **BTC signet** ve **USDC**), bir **vekil alışverişçi** (proxy shopper) de ürünü sizin yerinize satın alıp size gönderir.

Para, her sipariş için ayrı bir **2-of-3 multisig** cüzdanına gider (kullanıcı (user), alışverişçi, emanetçi (escrow)).

- Ürün ulaştığında kullanıcı ve alışverişçi, alışverişçiye ödeme yapmak için birlikte imza atar.
- Anlaşmazlık olursa emanetçi, diğer iki taraftan biriyle birlikte paranın nasıl bölüneceğine karar verir.
- Biri yanıt vermeyi bıraksa bile **zaman kilitleri** (timelock) paranın taraflardan birine geçmesini sağlar.

## Mimari

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **Kullanıcılar hiçbir şey çalıştırmaz.** Katılmak için herkese açık web uygulamasını açmak yeterlidir.
  Anahtarlar tarayıcıda oluşturulur ve oradan hiç çıkmaz; her yetkilendirme (imza) tarayıcıda gerçekleşir.
- NAT arkasındaki kullanıcılar da bağlanabilir, çünkü Nostr rölesine yapılan bağlantı dışa doğrudur (outbound).
- Sürekli çevrimiçi olmayanlara (kullanıcılar, emanetçiler, operatörler (operator), koordinatörler (coordinator)) giden mesajlar,
  hâlâ şifreli hâlde, posta kutusu görevi gören Nostr rölelerinde bekler. Her mesaj birkaç röleye gönderilir.
- Yalnızca alışverişçinin sürekli çevrimiçi olması gerekir. Alışverişçi bir Go düğümü ve mağazaları kullanan bir tarayıcı otomasyon aracı (shopper-bot) çalıştırır.

## Güven nasıl akar

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- Kullanıcı **yalnızca koordinatörün açık anahtarına** güvenir. Güven oradan operatörlere ve alışverişçi × emanetçi eşleşmelerine uzanır.
- Listelerin sürümleri vardır ve hiçbir zaman süreleri dolmaz. Birini çıkarmak, yeni bir sürüm yayımlamak demektir.
- Kullanıcı bir koordinatörü ayarlarından kaldırarak onu "görevden alır".

## Ücretler

| Kime | Zorla uygulanabilir mi? | Nasıl |
|---|---|---|
| alışverişçi | evet | fiyat teklifine dahildir |
| emanetçi (peşin) | evet | fonlamayla birlikte doğrudan emanetçiye ödenir. **Emanetçinin, peşin ücreti ödenmemiş siparişlerde hakemlik yapma yükümlülüğü yoktur** |
| emanetçi (anlaşmazlık) | evet | kararın paylaşımından alınır |
| operatör / koordinatör | hayır | protokol dışında kararlaştırılır, örneğin listeleme ücretleri. Bir listenin sattığı şey "bulunabilir olmak"tır |

Emanetçinin teminatı (bond) ve bu teminata el konulması protokolün parçası değildir.
Bunlar operatör ile emanetçinin kendi aralarında kararlaştırdığı şartlar olarak ele alınır; örnek bir sözleşme sağlanmıştır.

## Yalnızca nakit kabul eden mağazalar

Bir mağazanın konumu bir bölge kodudur (ör. `JP-13-13104` = Shinjuku, Tokyo).
Alışverişçi, gidip nakit ödeyebileceği bölgeleri beyan eder ve yalnızca oradaki mağazalar için sipariş alır.

## Mağaza riski

Alışverişçi bir siparişi almadan önce mağazayı puanlar:
mağaza alışverişçinin izin listesinde (allowlist) mi, HTTPS kullanıyor mu ve ödeme sayfası bilinen bir ödeme geçidi mi.
Kart bilgilerini rastgele bir siteye göndermek risklidir, bu yüzden düşük puanlı mağazalara yönelik siparişler reddedilir.
