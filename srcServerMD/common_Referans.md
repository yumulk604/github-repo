# common Referans Kılavuzu

Bu dosya, `srcServer/Source/common` klasörünün amacını ve içerdiği paylaşılan dosyaları açıklamaktadır.

`common` klasörü, Metin2 sunucusunun farklı bileşenleri (`db`, `game` ve potansiyel olarak diğerleri) tarafından ortak olarak kullanılan temel veri yapılarını, sabitleri, yardımcı fonksiyonları, makroları ve tanımlamaları içerir. Kod tekrarını önlemek ve tutarlılığı sağlamak açısından kritik bir rol oynar.

## Header Dosyaları

### `building.h`

*   **Amaç:** Lonca arazileri ve bu arazilere yerleştirilebilen yapılar/objeler ile ilgili temel veri yapılarını tanımlar.
*   **Namespace:** `building`
*   **Yapılar:**
    *   **`TLand`:** Bir arazi parçasını temsil eder (ID, harita indeksi, konum, boyut, sahip lonca ID, lonca seviye limiti, fiyat).
    *   **`TObjectMaterial`:** Bir obje için gerekli malzemeyi tanımlar (Item Vnum, Miktar).
    *   **`TObjectProto`:** Bir obje türünün şablonunu/prototipini tanımlar (Vnum, fiyat, gerekli malzemeler, yükseltme bilgileri, can, bölge, ilişkili NPC, grup bilgileri, bağımlılıklar).
    *   **`TObject`:** Dünyaya yerleştirilmiş spesifik bir obje örneğini temsil eder (ID, arazi ID, Vnum, konum, rotasyon, mevcut can).
*   **Sabitler:**
    *   `OBJECT_MATERIAL_MAX_NUM`: Bir prototipte tanımlanabilecek maksimum malzeme sayısı.
*   **Kullanım Alanı:** Lonca arazisi sistemi, bina inşa/yükseltme mekanikleri.

### `cache.h`

*   **Amaç:** Belirli bir veri türü (`T`) için basit bir "write-back" önbellek (cache) mekanizması sağlayan `cache<T>` template sınıfını tanımlar.
*   **İşlev:**
    *   Veriyi (`m_data`) bellekte tutar.
    *   Veri güncellendiğinde (`Put`) bir bayrak (`m_bNeedQuery`) ayarlar.
    *   Belirli bir süre (`m_expireTime`) geçince veya manuel olarak `Flush()` çağrılınca, veriyi kalıcı depoya yazmak için türetilmiş sınıfta implemente edilmesi gereken saf sanal `OnFlush()` metodunu çağırır.
    *   `Get()`: Önbellekteki veriye erişim sağlar ve isteğe bağlı olarak son erişim zamanını günceller.
    *   `CheckTimeout()`: Verinin ne kadar süredir güncellenmediğini kontrol eder.
    *   `CheckFlushTimeout()`: Flush işleminin zaman aşımına uğrayıp uğramadığını kontrol eder.
*   **Kullanım Alanı:** Sık erişilen verilerin (oyuncu, lonca vb.) veritabanı erişimini azaltmak için bellekte tutulması ve değişikliklerin periyodik olarak veya gerektiğinde veritabanına yazılması.

### `d3dtype.h`

*   **Amaç:** DirectX (D3DX) kütüphanesinde kullanılan temel matematiksel türlerin (`D3DXVECTOR2`, `D3DXVECTOR3`, `D3DXVECTOR4`, `D3DXQUATERNION`, `D3DXCOLOR`) tanımlarını sağlar.
*   **İçerik:** 2D, 3D, 4D vektörler, quaternion (rotasyon için) ve renk (RGBA float) yapılarını tanımlar.
*   **Sunucudaki Varlık Nedeni:** Muhtemelen istemci ile paylaşılan kod modüllerinin veya veri yapılarının bu türlere ihtiyaç duymasından kaynaklanır. İstemci grafik için DirectX kullandığından, bu türlerin sunucuda da tanımlı olması veri alışverişini veya ortak kodu kolaylaştırabilir. Alternatif olarak, başka bir kütüphane bağımlılığı veya eski bir kalıntı olabilir.

### `item_length.h`

*   **Amaç:** Eşyalarla ilgili yüzlerce sabiti, limiti, türü, alt türü, bayrağı ve diğer sınıflandırmaları tanımlayan çok kapsamlı bir başlık dosyasıdır.
*   **Ana İçerikler (Enum ve Sabitler):**
    *   **Genel Limitler (`EItemMisc`):** İsim uzunluğu, değer/efsun/soket sayısı, maks. adet, nitelik sayısı vb.
    *   **Ana Eşya Türleri (`EItemTypes`):** Silah, Zırh, Kullanılabilir, Malzeme, Metin, Konteyner, Balık, Kostüm, Simya Taşı, Evcil Hayvan vb.
    *   **Alt Türler (`EWeaponSubTypes`, `EArmorSubTypes`, `ECostumeSubTypes`, `EUseSubTypes`, `EMaterialSubTypes`, `EPetSubTypes` vb.):** Ana türlerin detaylı alt kategorileri.
    *   **Simya Sistemi (`EDragonSoul*`):** Yuva, Sınıf, Kademe ve ilgili diğer sabitler.
    *   **Eşya Bayrakları (`EItemFlag`):** Eşyanın davranışını tanımlayan özellikler (Geliştirilebilir, İstiflenebilir, Benzersiz vb.).
    *   **Eşya Anti-Bayrakları (`EItemAntiFlag`):** Kullanım kısıtlamaları (Sınıf, Cinsiyet, Ticaret, Düşürme, Satma vb.).
    *   **Giyilebilirlik Bayrakları (`EItemWearableFlag`):** Hangi ekipman yuvasına takılabileceğini belirtir.
    *   **Limit Türleri (`ELimitTypes`):** Kullanım için gereken koşullar (Seviye, Stat, Süre vb.).
    *   **Diğer Sistemler:** Geliştirme (`ERefineType`), Aksesuar (`EAcceInfo`), Aura (`EItemAura*`), Ganimet Filtresi (`ELootFilter`), Ruh Sistemi (`ESoul*`) gibi birçok sisteme özel enum'lar içerir.
*   **Önem:** Eşya sisteminin temelini oluşturan tanımlamaları içerir. Eşyalarla ilgili her türlü geliştirme için kritik bir referanstır. `#ifdef` kullanımı, farklı özellik setlerine sahip sunucu yapılandırmalarını destekler.

### `length.h`

*   **Amaç:** Eşyaların ötesinde, oyunun geneliyle ilgili çok sayıda sabit, limit, enum ve temel veri yapılarını tanımlar.
*   **Ana İçerikler (Enum ve Sabitler):**
    *   **Genel Limitler/Boyutlar:** İsim/şifre uzunlukları, envanter/kasa/kemer boyutları, maks. oyuncu/mob/lonca seviyesi, maks. yang/çek, giyilebilir slot sayısı, hızlı erişim sayısı vb.
    *   **Oyun Mekanikleri/Durumları:** Cinsiyet, Yönler, Zorluk Seviyeleri, Sınıflar, Irk Bayrakları, Sohbet Türleri, Karakter Pozisyonları, GM Seviyeleri, Mob Rütbe/Tür/Boyut/Yapay Zeka/Stat/Direnç/Element, Beceri Seviyeleri, Lonca Savaşı Durumları, Premium Türleri, Özel Efektler, Pencere Türleri vb.
    *   **Efsunlar (`EApplyTypes`):** Oyundaki tüm efsun türlerini tanımlayan çok kapsamlı enum.
    *   **Sistemlere Özel Sabitler:** Mücevher, Özel Pazar, Beceri Kitabı Kombinasyonu, Element Sistemi, 6/7 Efsun, Simya, Aura, Ganimet Filtresi vb. için tanımlar.
    *   **Diğer:** `SItemPos` (Eşya konumu yapısı), `EShopCoinType` (Dükkan para birimi), `ELocale` (Yerelleştirme).
*   **Önem:** Oyunun neredeyse tüm sistemlerinin sayısal sınırlarını, kategorilerini ve sabitlerini belirler. Oyun dengesi ve özellikleri büyük ölçüde bu dosyaya bağlıdır. `#ifdef` kullanımı ile farklı sunucu yapılandırmaları desteklenir.

### `noncopyable.h`

*   **Amaç:** Kendisinden `private` olarak türeyen sınıfların kopyalanmasını (kopyalama kurucusu ve atama operatörü) derleme zamanında engellemek için kullanılan bir yardımcı sınıftır.
*   **İşleyiş:** Kopyalama kurucusunu ve atama operatörünü `private` olarak bildirir (implemente etmez). Türetilmiş sınıflar için derleyici tarafından otomatik olarak bu fonksiyonlar oluşturulmaya çalışıldığında, `private` erişim nedeniyle derleme hatası alınır.
*   **Kullanım Alanı:** Kopyalanması anlamsız veya tehlikeli olan sınıflar (Singletonlar, kaynak yöneticileri vb.) için temel sınıf olarak kullanılır.

### `pool.h`

*   **Amaç:** Sıkça oluşturulan ve yok edilen `T` türündeki nesneler için dinamik bir nesne havuzu (object pool) sağlayan `CDynamicPool<T>` template sınıfını tanımlar.
*   **İşleyiş:**
    *   Havuzdaki nesneleri (`CPoolNode<T>`) çift yönlü bağlı listelerle yönetir: biri boşta olanlar (`m_pFreeList`), diğeri kullanımda olanlar (`m_pUsedList`) için.
    *   `Alloc()`: Önce serbest listeden bir nesne alır, yoksa `new` ile yeni bir tane oluşturur ve kullanılan listeye ekler.
    *   `Free()`: Nesneyi kullanılan listeden çıkarıp serbest listeye ekler (`delete` etmez).
    *   `Clear()`: Tüm listelerdeki nesneleri `delete` ile gerçekten yok eder ve havuzu sıfırlar.
    *   `FreeAll()`: Kullanımdaki tüm nesneleri serbest listesine taşır.
*   **Faydası:** Sürekli `new`/`delete` çağrılarının maliyetini azaltır, bellek parçalanmasını önlemeye yardımcı olur ve performansı artırabilir.
*   **Kullanım Alanı:** Karakter, mob, item, paket gibi sık yaratılıp/yok edilen nesnelerin yönetimi.

### `service.h`

*   **Amaç:** Sunucunun derleme zamanı yapılandırmasını yönetir. `#define` direktifleri aracılığıyla hangi sistemlerin ve özelliklerin aktif/pasif olacağını belirler.
*   **İşleyiş:** Kodun farklı yerlerindeki `#ifdef FEATURE_NAME ... #endif` blokları, bu dosyada ilgili `FEATURE_NAME` makrosunun tanımlı olup olmadığına göre derlemeye dahil edilir veya edilmez.
*   **İçerik:** Oyunun hemen her yönüyle ilgili çok sayıda yapılandırma makrosu içerir:
    *   Bölge/Dil Ayarları
    *   Para Birimleri (Won, Gaya)
    *   Sistemler (Simya, Kostüm, Pet, Binek, Element, Ruh, Kemer, Özel Envanter, Ganimet Filtresi, Gacha, Küp vb.)
    *   Ekipman Türleri (Kuşak, Tılsım, Eldiven)
    *   Karakter Seçenekleri (Kurt Adam, Fatih Seviyesi)
    *   Eşya Özellikleri (6/7 Efsun, 6 Soket, Görünüm Değiştirme)
    *   Harita/Zindanlar
    *   Etkinlikler
    *   Arayüz Özellikleri
    *   Ağ/Güvenlik Ayarları
    *   Yorum satırı halindeki makrolar, devre dışı bırakılmış özellikleri gösterir.
*   **Önem:** Sunucunun hangi özellik setleriyle derleneceğini kontrol eden merkezi dosyadır. Yeni sistemler eklerken/çıkarırken veya farklı sunucu versiyonları oluştururken kullanılır.

### `singleton.h`

*   **Amaç:** Singleton tasarım desenini uygulamak için `singleton<T>` template sınıfını sağlar. Bu desen, bir sınıftan yalnızca tek bir nesne oluşturulmasını ve buna global erişim sağlanmasını garanti eder.
*   **İşleyiş:**
    *   Tekil nesneye işaret eden statik bir pointer (`ms_singleton`) tutar.
    *   Kurucu, `ms_singleton`'ı oluşturulan türetilmiş sınıf nesnesine ayarlar (ve birden fazla nesne oluşturulmasını `assert` ile engeller).
    *   Yıkıcı, `ms_singleton`'ı `NULL` yapar.
    *   `instance()` / `Instance()` (statik metodlar): Tekil nesneye referans döndürür.
    *   `instance_ptr()` (statik metod): Tekil nesneye işaretçi döndürür.
*   **Kullanım:** Bir sınıf `public singleton<BuSınıf>` şeklinde türetilerek Singleton yapılır. Global erişim `BuSınıf::instance()` veya `BuSınıf::instance_ptr()` ile sağlanır.
*   **Kullanım Alanı:** Oyun boyunca tek olması gereken yönetim sınıfları (Karakter Yöneticisi, Eşya Yöneticisi vb.), konfigürasyon, loglama gibi global kaynaklar.

### `stl.h`

*   **Amaç:** Standart Şablon Kütüphanesi (STL) kullanımını kolaylaştıran çeşitli yardımcı fonksiyonlar, makrolar ve tanımlamalar içerir.
*   **İçerik:**
    *   Yaygın STL başlıklarını (`vector`, `string`, `map`, `list` vb.) içerir.
    *   `itertype`: Konteyner iterator türünü almak için platforma özgü makro.
    *   `stl_lowers`: String'i küçük harfe çevirme fonksiyonu.
    *   `stringhash`: `std::string` için hash fonksiyon nesnesi (hash tabanlı konteynerler için).
    *   `erase_if`: Bir konteynerden koşula uyan elemanları silme fonksiyonu.
    *   `wipe`: İşaretçi konteynerindeki tüm elemanları `delete` edip konteyneri temizleme fonksiyonu.
    *   `wipe_second`: Map gibi konteynerlerde `second` üyesi işaretçi olan elemanları `delete` edip konteyneri temizleme fonksiyonu.
    *   Platforma özel `MIN`/`MAX`/`MINMAX` (Windows) veya `void_mem_fun*` (diğer, eski C++ tarzı üye fonksiyon işaretçisi sarmalayıcıları) tanımları.
*   **Kullanım Alanı:** STL ile çalışmayı basitleştirmek, kod tekrarını azaltmak ve özellikle ham işaretçilerle çalışırken bellek güvenliğini artırmak. 

### `tables.h`

*   **Amaç:** Oyun sunucusu (GS) ve veritabanı sunucusu (DB) arasındaki iletişim paketlerinin başlıklarını (headers) ve bu paketlerde kullanılan veya doğrudan veritabanı tablolarını temsil eden veri yapılarının (`struct`) tanımlarını içerir. Sunucu mimarisi boyunca veri temsili için bir sözleşme görevi görür.
*   **Ana İçerikler:**
    *   **Paket Başlıkları (`GD_HEADERS`, `DG_HEADERS`):** GS -> DB (`GD_`) ve DB -> GS (`DG_`) yönlerinde gönderilen farklı paket türleri için sayısal tanımlayıcılar (`enum`). Bu başlıklar, isteğin veya yanıtın türünü belirtir (örn: `HEADER_GD_PLAYER_LOAD`, `HEADER_DG_LOGIN_SUCCESS`).
    *   **Veri Yapıları (`struct`):** Oyun içi varlıklar (oyuncu, eşya, mob, beceri, dükkan, lonca vb.) ve ağ paketleri için C-tarzı yapılar tanımlar. Bu yapılar, verilerin bellekte nasıl düzenleneceğini tam olarak belirler. Önemli örnekler:
        *   `TAccountTable`: Hesap bilgilerini ve bağlı karakter özetlerini içerir.
        *   `TPlayerTable`: Bir oyuncunun veritabanındaki tüm detaylı bilgilerini temsil eder (statüler, konum, envanter, beceriler, görevler vb.).
        *   `TPlayerItem`: Bir eşyanın tüm özelliklerini (Vnum, miktar, soketler, efsunlar, sahip vb.) tanımlar.
        *   `TMobTable`: Bir mob prototipinin özelliklerini içerir.
        *   `TItemTable`: Bir eşya prototipinin özelliklerini içerir.
        *   `TSkillTable`: Bir beceri prototipinin özelliklerini içerir.
        *   `TShopTable`: Bir dükkanın içeriğini tanımlar.
        *   Çok sayıda `TPacket...` yapısı: Belirli `GD_` veya `DG_` başlıklarıyla ilişkilendirilmiş paketlerin veri yükünü tanımlar (örn: `TLoginPacket`, `TPacketGuildWar`, `TPacketMarriageAdd`).
    *   **`#pragma pack(1)`:** Bu direktif, yapıların derleyici tarafından otomatik olarak hizalama baytları eklenmeden paketlenmesini sağlar. Ağ üzerinden gönderilen veya veritabanına yazılan verilerin boyutunun ve düzeninin tutarlı olması için kritik öneme sahiptir.
    *   **`#ifdef` Koşulları:** `service.h` dosyasındaki makrolara bağlı olarak, belirli özelliklere (`__MAILBOX__`, `__PREMIUM_PRIVATE_SHOP__`, `__CONQUEROR_LEVEL__` vb.) ait yapıların derlemeye dahil edilip edilmeyeceğini kontrol eder.
*   **Önem:** Sunucu bileşenleri arasındaki veri iletişiminin ve veritabanı etkileşimlerinin temelini oluşturur. Veri yapılarının ve paket başlıklarının merkezi tanımını sağlar. `#pragma pack(1)` kullanımı, farklı sistemler arasında bile veri tutarlılığını garantiler. 

### `utils.h`

*   **Amaç:** Çeşitli temel veri türleri (bool, char, short, int, long, long long, float, double, long double ve bunların işaretsiz versiyonları) için güvenli string'den sayıya dönüştürme işlemleri sağlayan `inline` fonksiyonlar koleksiyonunu içerir.
*   **İşlev:**
    *   Her veri türü için `str_to_number(TYPE& out, const char* in)` şeklinde bir fonksiyon şablonu sunar.
    *   Giriş string'inin (`in`) `NULL` veya boş olup olmadığını kontrol eder. Boşsa `false` döner.
    *   C standart kütüphane fonksiyonlarını (`strtol`, `strtoul`, `strtof`, `strtod`, `strtoull` vb.) kullanarak string'i ilgili sayısal türe dönüştürür.
    *   Dönüştürülen değeri `out` referans parametresine atar.
    *   Dönüşüm başarılıysa (giriş string'i geçerliyse) `true`, değilse `false` döner.
    *   `long double` için olan versiyon `#ifdef __FreeBSD__` bloğu içindedir, yani sadece FreeBSD sistemlerinde derlenir.
*   **Kullanım Alanı:** Metin tabanlı verilerden (konfigürasyon dosyaları, veritabanı sonuçları, ağ mesajları) sayısal değerleri okurken güvenli ve standart bir dönüştürme yöntemi sunar. `atoi`, `atof` gibi daha az güvenli fonksiyonlara alternatif oluşturur.

### `VnumHelper.h`

*   **Amaç:** Belirli eşya (Item) ve yaratık (Mob) Vnum'larının (Virtual Number - Sanal Numara) özel türlere ait olup olmadığını kontrol etmek için statik yardımcı (`helper`) fonksiyonlar içeren sınıfları tanımlar.
*   **İçerik:**
    *   **`CItemVnumHelper` Sınıfı:**
        *   `IsPhoenix(vnum)`, `IsRamadanMoonRing(vnum)`, `IsHalloweenCandy(vnum)`, `IsHappinessRing(vnum)`, `IsLovePendant(vnum)`, `IsMagicRing(vnum)`, `IsNazarPendant(vnum)`, `IsGemCandy(vnum)` gibi özel/etkinlik eşyalarını Vnum'larına göre kontrol eden `static const bool` fonksiyonlar içerir.
        *   `IsUniqueItem(vnum)`: Yukarıdaki özel eşyalardan herhangi biri olup olmadığını topluca kontrol eder.
        *   `IsDragonSoul(vnum)`: Bir Vnum'un Simya Taşı aralığında olup olmadığını kontrol eder (`__DS_7_SLOT__` makrosuna göre aralık değişebilir).
    *   **`CMobVnumHelper` Sınıfı:**
        *   `IsPhoenix(vnum)`, `IsIcePhoenix(vnum)`, `IsReindeerYoung(vnum)` gibi özel binek/pet Vnum'larını kontrol eden fonksiyonlar içerir.
        *   `IsPetUsingPetSystem(vnum)`: Pet sistemi tarafından kullanılan özel petlerden biri olup olmadığını kontrol eder.
        *   `IsRamadanBlackHorse(vnum)`: Etkinliğe özel at Vnum'larını kontrol eder.
        *   `IsNPCType(type)`: Bir karakter türünün (`CHAR_TYPE_*`) NPC, at veya pet olup olmadığını kontrol eder.
    *   **`CVnumHelper` Sınıfı:** Şu anda boş, muhtemelen gelecekteki genel Vnum yardımcı fonksiyonları için bir yer tutucu.
*   **Kullanım Alanı:** Kodun farklı yerlerinde belirli Vnum'lara özel mantık uygulanması gerektiğinde (örneğin, bir eşyanın tekil olup olmadığını kontrol etmek, bir mob'un pet sistemine dahil olup olmadığını anlamak) bu yardımcı fonksiyonlar kullanılır. Bu, Vnum'ları doğrudan kod içinde sabit olarak yazmak yerine daha okunaklı ve yönetilebilir bir yol sunar. 