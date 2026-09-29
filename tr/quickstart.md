---
layout: page
title: Başlarken
permalink: /tr/quickstart/
lang: tr
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Tüm ağı yerelde çalıştırın (laboratuvar)

Tek bir docker compose dosyası kapalı bir ağ ayağa kaldırır. İnternete bağlanmaz.

- iki Nostr rölesi
- bir libp2p rölesi
- özel bir signet üzerinde bitcoind ve Esplora uyumlu bir API
- Safe ve diğer sözleşmelerle birlikte anvil (EVM)
- dört sahte mağaza ve sahte bir kart ödeme geçidi
- iki alışverişçi (shopper), üç emanetçi (escrow) (biri NAT arkasında), iki operatör (operator)
- herkese açık web uygulaması ve demo uygulaması

Gereksinimler: Docker (yaklaşık 8 GB bellek).

proxy-shopping-go ve proxy-shopping-web depolarını aynı üst dizinde yan yana koyun ve şunu çalıştırın:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Baştan başlamak için önce `docker compose down -v` çalıştırarak düğümlerin ve rölelerin saklanan verilerini de silin
(zincirler her başlatmada sıfırdan başlar, bu yüzden geride kalan sipariş verileri onlarla uyuşmaz).

### e2e testlerini çalıştırın

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Sonuçlar `e2e/results/e2e-latest.md` dosyasına yazılır. Runner NAT'ın içinden (`home` ağı) çalışır,
bu yüzden her senaryo kullanıcının NAT arkasında olduğunu varsayar.

| id | Senaryo |
|---|---|
| a | Sorunsuz akış: BTC (JPY ile satış yapan bir mağaza) ve USDC (USD ile satış yapan bir mağaza). Teklifteki kur kontrol edilir, emanetçi peşin ücretini alır, alışverişçiye ödeme yapılır. Kuru çok sapan bir teklif güçlü bir uyarı alır |
| b | Teslimat başarısız olur ve kullanıcı bir anlaşmazlık açar. Dürüst bir emanetçi iade kararı verir ve kullanıcı karşı imza atar (BTC). Emanetçi adresin şifresini çözebilir ve ekran görüntüsünü ek olarak alır |
| c | Bir emanetçi dürüst olmayan bir karar verir. Kullanıcı bunu şikâyet eder; operatör onu listeden çıkarır ve şartları gereği teminatına tazminat olarak el koyar (USDC) |
| d | Bir koordinatör (coordinator) bir operatörün yetki devrini geri alır ve o listenin eşleşmeleri tekliflerden kaybolur |
| e | NAT arkasındaki bir tarayıcı, herkese açık web uygulaması üzerinden sipariş verir (1:1 mesajlar Nostr röleleri üzerinden gider). Bir durum sorgusu, NAT arkasındaki emanetçi düğümüne libp2p circuit relay üzerinden ulaşır |
| f | Bir alışverişçi riskli olarak puanladığı bir mağazayı ve bölgesi dışındaki yalnızca nakit kabul eden bir mağazayı reddeder. Bölgedeki bir alışverişçi nakit siparişini alır ve teslim eder |
| g | T1'den sonra alışverişçi parayı tek başına alabilir (BTC) |
| h | Alışverişçi ortadan kaybolursa, kullanıcı T2'den sonra parayı tek başına geri alır (USDC) |
| i | Dürüst bir emanetçiyle USDC anlaşmazlığı. Karardan önce biri Safe'e küçük bir miktar gönderse bile iade yine gerçekleşir |
| j | Mağazanın karşılayamadığı bir sipariş (stokta yok). Alışverişçi iş birliğiyle iade önerir; kullanıcı kontrol edip kabul eder |
| k | Alışverişçi ortadan kaybolursa, kullanıcı T2'den sonra parayı tek başına geri alır (BTC) |

### Tüm akışı demoda deneyin

Laboratuvar çalışırken demoyu `http://localhost:8888/` adresinde açın (hosts dosyası ya da sertifika gerekmez).
Tek bir ekranda kullanıcı, emanetçi, operatör ve koordinatörün her birinin kendi anahtarı vardır (tarayıcının yerel depolamasında),
alışverişçi ise sürekli çevrimiçi olan Go düğümünün kendisidir. Yollar gerçektir: Nostr röleleri, bitcoind, anvil, Go düğümleri.

- En üstten bir senaryo seçin. Soldaki rehber sırada kimin ne yapacağını ve nedenini söyler.
- "Go to this step" o rolün sekmesine geçer ve basılacak düğmeyi gösterir. Formlar senaryoya göre önceden doldurulmuştur, yalnızca tıklayıp onaylarsınız.
- Senaryolar: sorunsuz akış (BTC / USDC), başarısız teslimat ve iade, stok tükenmesi ve iş birliğiyle iade, riskli bir mağazanın reddedilmesi, dürüst olmayan bir emanetçinin şikâyet edilip listeden çıkarılması ve alışverişçi kaybolduğunda T2'den sonra iade.
- Her rol için ayrı bir pencere de açabilirsiniz, ör. `?role=user` ve `?role=escrow,operator,coordinator` (aynı tarayıcının pencereleri anahtarları ve ilerlemeyi paylaşır).
- Gerçekte her rol başka bir yerde, kendi tarayıcısındadır. Demo yalnızca anahtarları ayırır.
- Yalnızca laboratuvar için: 8888 numaralı porta erişebilen herkes faucet'ı, madenciliği, zaman atlatmayı ve alışverişçinin yönetim API'sini kullanabilir (yalnızca 127.0.0.1'e bağlıdır).

Demoyu rehberini izleyerek yöneten bir e2e testi de vardır: `docker compose run --rm runner demo`.

### Web uygulamasını kullanın

Laboratuvar adları (`*.test`) yalnızca konteynerlerin içinde çözümlenir.
Web uygulamasını kendi tarayıcınızdan kullanmak için:

- `edge` konteynerinin 443 numaralı portunu yerelde açın (ör. `compose.override.yaml` içinde: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Bu, yalnızca laboratuvara özel `faucet.test` ve `evm.test` adreslerini de açar (herkes bakiye basabilir ve zamanı ileri alabilir), bu yüzden yalnızca localhost'ta açın.
- hosts dosyanızda `app.test` ve diğer adları 127.0.0.1'e yönlendirin.

Sertifikalar kendinden imzalıdır.

1. `https://app.test/` adresini açın ve "create new" ya da "restore from words" seçin.
   Anahtarı şifreleyen bir parola seçin (8 karakter veya daha uzun) (açıkça şifrelememeyi seçebilir ya da bir NIP-07 eklentisi kullanabilirsiniz).
   Bir dahaki sefere anahtarın kilidini parolayla açın.
2. "order" altında mağaza URL'sini (ör. `https://safe-shop.test/`), mağazanın bölgesini (ör. `JP-13-13104`) ve ürünü (ör. `A-100`) girin ve teklifleri arayın.
3. Bir alışverişçi × emanetçi eşleşmesi seçin, teslimat adresini girin ve siparişi verin.
4. Teklif geldiğinde kur farkını ve multisig adres kontrolünün sonucunu inceleyin, ardından kabul edin.
5. Laboratuvarda cüzdanınızı "get from the faucet" ile doldurun, ardından "fund the multisig" düğmesine basın.
   Para hareket ettiren her işlem (fonlama, ödeme, karşı imza) tutarı ve alıcıyı gösteren bir ekrandan geçer.
6. Ürün ulaştığında alışverişçiye ödemek için "received" düğmesine basın. "Completed" yalnızca ödeme zincirde onaylandıktan sonra görünür.

## 2. Herkese açık bir ağda çalıştırın

Mimari laboratuvarla aynıdır. Farklar:

- ACME sertifikaları kullanın;
- `lab/keys` içindeki anahtarları kullanmayın;
- gerçek bir signet ve EVM zinciri kullanın.

### Alışverişçi

Sürekli çevrimiçidir; Go düğümünü ve shopper-bot'u çalıştırır.

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

- Kart bilgileri yalnızca shopper-bot'un yapılandırma dosyasına (`BOT_CARDS_FILE`) girer; düğüm bunları hiçbir zaman almaz.
- Her mağazanın adımları bir shopper-bot sürücüsü (driver) olarak yazılır. Mağazaları yapay zekâyla kullanan bir sürücü de aynı girdi ve çıktıya sahiptir (`PurchaseRequest` / `PurchaseResult`).

### Emanetçi / operatör / koordinatör

Bunların sürekli çevrimiçi olması gerekmez.

- Web uygulamasının "Escrow", "Operator" ve "Coordinator" ekranlarını kullanabilirler.
- Kesintisiz çalıştırmak için Go düğümünü `role: escrow` / `role: operator` ile çalıştırın.
- Yalnızca imzalamak için `psctl` de işe yarar.

Sürüm (`v`) varsayılan olarak UNIX zamanıdır, bu yüzden normalde onu vermezsiniz.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Ağdaki kendi rolünüzü her yerde, izin almadan çalıştırın

proxy-shopping'in merkezi bir işletmecisi yoktur ve kaydolunacak hiçbir şey yoktur. Güven zinciri yalnızca anahtarlardan ve imzalı Nostr olaylarından oluşur,
bu yüzden siz (ya da sizin için çalışan bir yapay zekâ ajanı) kendi şehrinizde yerel bir pazar yeri başlatabilirsiniz:

1. **Koordinatör olun:** bir anahtar oluşturun (`psctl keys --mnemonic-file coordinator.mnemonic`). Koordinatör olmak bundan ibarettir.
2. **Bir operatöre yetki devredin** (kendi ikinci anahtarınız ya da güvendiğiniz biri):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Bölgeniz için alışverişçileri ve emanetçileri listeleyin** — alışverişçi olarak kendinizi, emanetçi olarak bir arkadaşınızı:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **İnsanlara koordinatör anahtarınıza güvenmelerini söyleyin** (web uygulamasının Ayarlar bölümünden ekleyin ya da MCP sunucusu için `PS_COORDINATORS`).
   Herkes tarafından bulunmak için [kayıt deposuna](https://github.com/pad01g/proxy-shopping-registry)
   `coordinators/<name>.json` ekleyen bir pull request açın; kendi topluluğunuz için onayları aynı şekilde yürütmek isterseniz kayıt deposunu fork'layın.

**Nerede çalıştırmalı:** evdeki bir bilgisayar yeterlidir. Düğüm yalnızca dışa doğru bağlantılar kurar (Nostr röleleri ve NAT arkasındayken
libp2p circuit relay), bu yüzden hiçbir port açmanız gerekmez. Dışarıdayken telefonunuzdan düğümünüzün yönetim API'sine ve web uygulamasına
erişmek için [Tailscale](https://tailscale.com/) ya da herhangi bir VPN kullanın; küçük bir VPS de iş görür.

**Bir yapay zekâ ajanı neler yapabilir:** bir ajan operatörün ve alışverişçinin günlük işlerini yürütebilir — düğümün yönetim API'si
ya da MCP sunucusu üzerinden siparişleri izlemek, listeleri güncel tutmak, kartla ödeme alan mağazalar için shopper-bot'u çalıştırmak, sorunları bildirmek — sizin
yerel bilginiz ise (hangi mağazalar, hangi bölgeler, yürüyerek gidebileceğiniz yalnızca nakit kabul eden dükkânlar) başka kimsenin sunamayacağı kısımdır.
`proxy-shopper` skill'i (`npx skills add pad01g/proxy-shopping-go`) ajana bu süreçte adım adım rehberlik eder.

Kullanıcılarınıza karşı dürüst olun: herkese açık ağ yenidir ve BTC signet (test coinleri) üzerinde çalışır, bu yüzden kazançlar insanlar kullanmaya başladıkça gelir.

## Katkıda bulunma {#contributing}

**Pull request'ler memnuniyetle karşılanır** — tüm depolarda: shopper-bot için mağaza sürücüleri, yeni ödeme yöntemleri ve zincirler,
bu belgelerin çevirileri, protokol incelemeleri, hata düzeltmeleri ve
[kayıt deposuna](https://github.com/pad01g/proxy-shopping-registry) yapılacak listelemeler. GitHub'da bir issue ya da pull request açın.
