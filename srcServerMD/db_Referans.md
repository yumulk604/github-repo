# db Referans Kılavuzu

Bu dosya, `srcServer/Source/db` klasörünün amacını, içerdiği önemli bileşenleri ve çalışma prensiplerini açıklamaktadır.

`db` klasörü, Metin2 sunucu mimarisindeki **Veritabanı Sunucusu (DB Server)** bileşeninin kaynak kodunu içerir. Ana sorumluluğu, oyun sunucularından (`game`) gelen istekleri işleyerek MySQL veritabanı ile etkileşime girmek, verileri okumak, yazmak, güncellemek ve oyun sunucularına gerekli yanıtları göndermektir. Oyuncu verileri, hesap bilgileri, eşyalar, loncalar, görev ilerlemeleri gibi kalıcı tüm verilerin yönetiminden sorumludur.

Genellikle `libthecore` (çekirdek ağ fonksiyonları, olay döngüsü) ve `libsql` (veritabanı bağlantı sarmalayıcısı) kütüphanelerini yoğun olarak kullanır. Veri yapıları için `common` klasöründeki tanımlamalara güvenir.

## Header Dosyaları (`src/`)

Bu bölümde, `db/src` klasöründe bulunan ve arayüzleri, sınıfları, yapıları tanımlayan başlık dosyaları (`.h`) açıklanmaktadır.

### `BlockCountry.h`

*   **Amaç:** Belirli IP adresi aralıklarından gelen bağlantılara erişim engeli koyma (GeoIP blocking) ve bu engelleme için istisna (exception) hesap listesi yönetimi işlevselliğini sağlayan `CBlockCountry` sınıfını bildirir.
*   **Bildirilenler:**
    *   **`CBlockCountry` Sınıfı:** Singleton olarak tasarlanmıştır. `Load()`, `IsBlockedCountryIp()`, `SendBlockedCountryIp()`, `SendBlockException()`, `AddBlockException()`, `DelBlockException()` gibi metodları bildirir.
    *   `BLOCK_IP` (private struct): Engellenecek IP aralığını (başlangıç, bitiş, ülke adı) tanımlar.
*   **Önem:** IP tabanlı erişim kontrolü mekanizmasının arayüzünü tanımlar. Asıl mantık `BlockCountry.cpp` dosyasındadır.

### `Cache.h`

*   **Amaç:** DB sunucusunda sık erişilen veritabanı tablolarına ait verileri (eşyalar, oyuncu bilgileri, özel pazar listeleri) bellekte önbelleğe almak (caching) için kullanılan sınıfları bildirir. Bu, veritabanı okuma/yazma işlemlerini azaltarak sunucu performansını artırmayı hedefler.
*   **Temel Aldığı Sınıf:** Bu dosyadaki tüm önbellek sınıfları, `common/cache.h` içinde tanımlanan genel `cache<T>` template sınıfından türetilmiştir.
*   **Bildirilen Sınıflar:**
    *   **`CItemCache`:** `TPlayerItem` verilerini (tekil eşyalar) önbelleğe alır.
    *   **`CPlayerTableCache`:** `TPlayerTable` verilerini (oyuncu ana bilgileri) önbelleğe alır.
    *   **`CItemPriceListTableCache`:** `TItemPriceListTable` verilerini (Özel Pazar fiyat listeleri) önbelleğe alır. (Muhtemelen `#ifdef` ile çevrili).
    *   **`CPrivateShopCache`:** `TPrivateShop` verilerini (Premium Özel Pazar ana bilgileri) önbelleğe alır. (`#ifdef __PREMIUM_PRIVATE_SHOP__` ile çevrili).
    *   **`CPrivateShopItemCache`:** `TPlayerPrivateShopItem` verilerini (Premium Özel Pazar eşyaları) önbelleğe alır. (`#ifdef __PREMIUM_PRIVATE_SHOP__` ile çevrili).
*   **Önem:** Veritabanı etkileşimini optimize ederek DB sunucusunun performansını artırır. Bu sınıfların asıl çalışma mantığı (`OnFlush` vb.) `Cache.cpp` dosyasında implemente edilir.

### `ClientManager.h`

*   **Amaç:** DB sunucusunun ana yönetim sınıfı olan `CClientManager`'ı bildirir. Bu sınıf, oyun sunucularıyla (peer) olan bağlantıları yönetmek, gelen istekleri işlemek, veritabanı tablolarını belleğe yüklemek, önbellek mekanizmalarını (oyuncu, eşya, özel pazar vb.) kontrol etmek ve veritabanı işlemlerini koordine etmekten sorumludur.
*   **Tasarım:**
    *   `CNetBase`'den (muhtemelen temel ağ işlevleri için) ve `singleton<CClientManager>`'dan türetilmiştir. Global erişim sağlar.
*   **Önemli Veri Üyeleri (typedef ve map/vector):**
    *   `TPeerList`: Bağlı oyun sunucularının (`CPeer*`) listesi.
    *   `TPlayerTableCacheMap`, `TItemCacheMap`, `TItemCacheSetPtrMap`, `TItemPriceListCacheMap`: Oyuncu, eşya ve özel pazar fiyat listesi önbelleklerini yönetmek için kullanılan haritalar (map). `Cache.h`'deki sınıfları kullanır.
    *   `TChannelStatusMap`: Oyun kanallarının durumlarını (online/offline vb.) tutar.
    *   `TLoginData*` haritaları (`m_map_pkLoginData*`): Oyuncu giriş anahtarları (login key), kullanıcı adları ve hesap ID'leri ile ilişkili geçici giriş verilerini yönetir (`LoginData.h`).
    *   `TLogonAccountMap`: Hangi hesapların hangi oyun sunucusunda aktif olduğunu takip eder.
    *   `m_vec_mobTable`, `m_vec_itemTable`, `m_pShopTable`, `m_pRefineTable` vb.: Sunucu başlangıcında veritabanından veya dosyalardan yüklenen mob, eşya, dükkan, geliştirme gibi prototip tablolarını bellekte tutar.
    *   `m_map_pkObjectTable`: Lonca arazisi nesnelerini yönetir (`common/building.h`).
    *   `TPartyChannelMap`: Kanallar arasındaki parti bilgilerini tutar.
    *   `TEventFlagMap`: Global olay bayraklarını (event flag) yönetir.
    *   `TLogoutPlayerMap`: Oyundan çıkan ancak önbellekleri hemen temizlenmeyen oyuncuları takip eder.
    *   `m_itemRange`: Kullanılabilir eşya ID aralığını yönetir (`ItemIDRangeManager.h`).
    *   `#if defined(__PREMIUM_PRIVATE_SHOP__)` / `#if defined(__MAILBOX__)` vb. içindeki üyeler: İlgili sistemler aktifse bu sistemlere ait önbellekleri ve verileri yönetir.
*   **Önemli Metod Bildirimleri:**
    *   **`Initialize()`:** Sunucuyu başlatır, tabloları yükler, ağ dinlemesini başlatır.
    *   **`MainLoop()`:** Ana olay döngüsünü çalıştırır (muhtemelen `libthecore`'dan).
    *   **`Process()`:** Gelen ağ paketlerini ve veritabanı sorgu sonuçlarını işler.
    *   **`ProcessPackets(CPeer* peer)`:** Belirli bir oyun sunucusundan gelen paketleri işler.
    *   **`GetPlayerCache()`, `PutPlayerCache()`, `GetItemCache()`, `PutItemCache()`, `DeleteItemCache()` vb.:** Önbellek yönetimi fonksiyonları.
    *   **`InitializeTables()`, `InitializeShopTable()`, `InitializeMobTable()`, `InitializeItemTable()` vb.:** Veritabanından veya dosyalardan prototip tablolarını yükleyen fonksiyonlar.
    *   **`AddPeer()`, `RemovePeer()`, `GetPeer()`:** Oyun sunucusu bağlantı yönetimi.
    *   **`AnalyzeQueryResult()`, `AnalyzeErrorMsg()`:** Veritabanı sorgu sonuçlarını ve hatalarını işler.
    *   **`QUERY_*` Metodları (örn: `QUERY_LOGIN`, `QUERY_PLAYER_LOAD`, `QUERY_ITEM_SAVE`):** Oyun sunucusundan gelen belirli isteklere (GD_HEADER) karşılık gelen ve genellikle veritabanı sorgusu başlatan işleyici fonksiyonlar.
    *   **`RESULT_*` Metodları (örn: `RESULT_LOGIN`, `RESULT_PLAYER_LOAD`, `RESULT_ITEM_LOAD`):** Tamamlanan veritabanı sorgularının sonuçlarını işleyen ve genellikle oyun sunucusuna yanıt (DG_HEADER) gönderen işleyici fonksiyonlar.
    *   Lonca, Evlilik, Monarşi, Özel Pazar, Mailbox vb. sistemlere ait çok sayıda istek işleyici (`GuildCreate`, `MarriageAdd`, `Election`, `MyshopPricelistUpdate`, `QUERY_MAILBOX_LOAD`, `PrivateShopBuyRequest` vb.).
    *   **`ForwardPacket()`:** Bir paketi belirli bir kanala veya tüm oyun sunucularına (isteğe bağlı olarak biri hariç) iletir.
    *   **`SendNotice()`:** Tüm oyun sunucularına duyuru mesajı gönderir.
*   **Önem:** DB sunucusunun merkezi kontrol noktasıdır. Oyun sunucuları ile veritabanı arasındaki tüm iletişimi yönetir, veri tutarlılığını sağlar, önbellekleme ile performansı artırır ve çeşitli oyun sistemlerinin veritabanı tarafındaki mantığını içerir. Arayüzü, DB sunucusunun yeteneklerini ve hangi tür istekleri işleyebileceğini gösterir.

(Buraya diğer .h dosyalarının analizleri eklenecek)

## Kaynak Kod Dosyaları (`src/`)

Bu bölümde, `db/src` klasöründe bulunan ve `.h` dosyalarında bildirilen sınıfların/fonksiyonların implementasyonlarını içeren kaynak kod dosyaları (`.cpp`) açıklanmaktadır.

### `BlockCountry.cpp`

*   **Amaç:** `BlockCountry.h` dosyasında bildirilen `CBlockCountry` sınıfının metodlarını implemente eder.
*   **Implementasyon Detayları:**
    *   **`Load()`:** `iptocountry` ve `block_exception` tablolarını `CDBManager::DirectQuery` ile sorgular, sonuçları ayrıştırır ve `m_block_ip`, `m_block_exception` vektörlerine doldurur.
    *   **`IsBlockedCountryIp()`:** Gelen IP'yi `inet_aton`/`inet_addr` ile sayısal forma dönüştürür ve `m_block_ip` listesindeki aralıklara göre kontrol eder.
    *   **`Send*` Metodları:** `CPeer::EncodeHeader` ve `CPeer::Encode` kullanarak ilgili paketleri (HEADER_DG_BLOCK_COUNTRY_IP, HEADER_DG_BLOCK_EXCEPTION) oluşturup oyun sunucusuna gönderir.
    *   **`Add/DelBlockException()`:** `m_block_exception` vektörünü günceller, `strdup` ve `free` ile bellek yönetimi yapar.
    *   Yıkıcı (`~CBlockCountry`), `m_block_ip` içindeki `BLOCK_IP` nesnelerini `delete` eder.
*   **Bağımlılıklar:** `stdafx.h`, `BlockCountry.h`, `DBManager.h`, `Peer.h`, `common/tables.h` (Paket yapıları için dolaylı).

### `Cache.cpp`

*   **Amaç:** `Cache.h` dosyasında bildirilen önbellek sınıflarının (`CItemCache`, `CPlayerTableCache`, `CItemPriceListTableCache` vb.) metodlarını implemente eder. Özellikle, önbellekteki verilerin veritabanına nasıl yazılacağını (`OnFlush`) ve bazı durumlarda verilerin nasıl silineceğini (`Delete`) tanımlar.
*   **Implementasyon Detayları:**
    *   **`CItemCache::OnFlush()`:**
        *   Eşya silinmişse (`m_data.vnum == 0`), `DELETE FROM item...` sorgusu çalıştırır.
        *   Eşya güncellenmişse, `INSERT ... ON DUPLICATE KEY UPDATE ...` sorgusu oluşturur. Bu sorgu, eşyanın tüm özelliklerini (soketler, efsunlar, sistemlere özel alanlar) içerecek şekilde dinamik olarak inşa edilir.
    *   **`CPlayerTableCache::OnFlush()`:**
        *   Oyuncu verilerini kaydetmek için `UPDATE player SET ... WHERE id = ...` sorgusu oluşturur. Tüm oyuncu alanları güncellenir.
    *   **`CItemPriceListTableCache::OnFlush()`:**
        *   Sahibin eski listesini `DELETE` ile siler, ardından yeni listeyi `INSERT` ile satır satır ekler.
    *   **`CPrivateShopCache::OnFlush()` / `CPrivateShopItemCache::OnFlush()`:**
        *   Bu sınıfların `OnFlush` metodları, önbellekteki verilerin veritabanına yazılması için `OnFlush` metodunu çağırır.
*   **Bağımlılıklar:** `stdafx.h`, `Cache.h`, `common/cache.h`, `Peer.h`, `DBManager.h`, `common/tables.h` (Paket yapıları için dolaylı).

### `ClientManager.cpp`

*   **Amaç:** `ClientManager.h`'de bildirilen `CClientManager` sınıfının tüm fonksiyonlarını ve mantığını implemente eden, DB sunucusunun çekirdek kodunu içerir. Boyutu nedeniyle, ana işlevlerini ve implementasyon yaklaşımlarını özetleyerek belgeleyeceğiz.
*   **Implementasyon Detayları:**
    *   **Başlatma ve Sonlandırma (`Initialize`, `MainLoop`, `Quit`, `Destroy`):**
        *   `Initialize()`: Sunucu yapılandırmasını (`CConfig`) okur, yerelleştirme ve eşya ID aralığı gibi temel ayarları yapar, prototip tablolarını (`InitializeTables`) yükler, ağ soketini oluşturur ve dinlemeye başlar (`socket_tcp_bind`, `fdwatch_add_fd`).
        *   `MainLoop()`: Ana olay döngüsünü yönetir. `libthecore`'un `thecore_idle()` fonksiyonu ile periyodik olarak çalışır. Veritabanı sonuçlarını (`CDBManager::PopResult`) işler, zamanlanmış görevleri (önbellek temizleme, saat gönderme vb.) yürütür ve ağ olaylarını (`fdwatch`) dinler. `m_bShutdowned` bayrağı `true` olunca döngüden çıkar ve önbellekleri temizleyerek (`Flush`) kapanır.
        *   `Quit()`: `m_bShutdowned` bayrağını `true` yaparak sunucunun gracefully kapanmasını tetikler.
        *   `Destroy()`: Tüm oyun sunucusu bağlantılarını (`CPeer`) kapatır ve ağ dinleme soketini kapatır.
    *   **Tablo Yükleme (`Initialize*Table` metodları):**
        *   Sunucu başlangıcında çeşitli oyun verilerini (mob, item, shop, skill, refine vb.) veritabanından (`SQL_COMMON` veya `SQL_PLAYER` bağlantıları üzerinden `DirectQuery` ile) veya bazen dosyalardan okuyarak bellekteki ilgili vektörlere veya haritalara (`m_vec_mobTable`, `m_map_itemTableByVnum` vb.) yükler.
        *   Yerelleştirme (`InitializeLocalization`) ve Eşya ID aralığı (`InitializeNowItemID`) gibi özel başlangıç işlemleri de burada yapılır.
    *   **Ağ ve Bağlantı Yönetimi (`AddPeer`, `RemovePeer`, `GetPeer`, `ProcessPackets`, `ForwardPacket`):**
        *   `AddPeer()`: Yeni bir oyun sunucusu bağlantısını kabul eder, `CPeer` nesnesi oluşturur ve `m_peerList`'e ekler.
        *   `RemovePeer()`: Bir oyun sunucusunun bağlantısı kesildiğinde çağrılır. İlgili `CPeer` nesnesini siler, listeden çıkarır ve bu sunucuya bağlı olan hesapların (`TLogonAccountMap`) durumunu günceller.
        *   `ProcessPackets()`: Belirli bir oyun sunucusundan (`CPeer`) gelen verileri okur (`PeekPacket`), paket başlıklarına (`HEADER_GD_*`) göre ayırır ve ilgili `QUERY_*` işleyici fonksiyonuna yönlendirir (`switch` bloğu).
        *   `ForwardPacket()`: Bir paketi belirli bir kanala veya tüm bağlı oyun sunucularına (isteğe bağlı olarak biri hariç) göndermek için kullanılır.
    *   **Veritabanı Etkileşimi ve Sonuç İşleme (`AnalyzeQueryResult`, `AnalyzeErrorMsg`):**
        *   `CDBManager::PopResult()` ile alınan tamamlanmış veritabanı sorgu sonuçlarını (`SQLMsg`) işler.
        *   `AnalyzeQueryResult()`: Gelen sonucun QID'sine (`qi->iType`) göre ilgili `RESULT_*` fonksiyonuna yönlendirir. Sorguyla ilişkilendirilmiş `ClientHandleInfo` verisini kullanarak isteği başlatan oyun sunucusunu (`peer`) bulur.
        *   `AnalyzeErrorMsg()`: Veritabanı sorgu hatalarını işler (genellikle loglar).
    *   **İstek İşleme (`QUERY_*` ve `RESULT_*` metodları):**
        *   `ClientManager.h`'de listelenen her `QUERY_*` metodu, ilgili `HEADER_GD_*` paketini alır, gerekli veritabanı sorgusunu (`DirectQuery` veya `ReturnQuery`/`AsyncQuery`) oluşturur ve çalıştırır. Bazı durumlarda (`QUERY_PLAYER_COUNT` gibi) doğrudan yanıt gönderir.
        *   Her `RESULT_*` metodu, ilgili QID ile tamamlanan sorgunun sonucunu (`SQLMsg*`) alır, veriyi işler (gerekirse `common/tables.h` yapılarına doldurur), önbelleği günceller ve sonucu ilgili oyun sunucusuna `HEADER_DG_*` paketi olarak gönderir (`EncodeHeader`, `Encode`).
    *   **Önbellek Yönetimi (`Update*Cache`, `Put*Cache`, `Get*Cache`, `Flush*CacheSet` vb.):**
        *   Oyuncu, eşya, özel pazar vb. önbelleklerini periyodik olarak (`UpdatePlayerCache`, `UpdateItemCache`) veya gerektiğinde (`FlushPlayerCacheSet`) günceller ve temizler.
        *   `Put*Cache`: Yeni veya güncellenmiş veriyi önbelleğe ekler/günceller.
        *   `Get*Cache`: Önbellekten veri okur.
        *   `Delete*Cache`: Veriyi önbellekten siler ve veritabanından silinmesini tetikler.
    *   **Sistem Mantığı:** Lonca, Evlilik, Monarşi, BlockCountry, PrivManager, ItemAwardManager gibi özel sistemlerin DB sunucusunda mantığını içeren fonksiyonları implemente eder (genellikle ilgili Singleton yöneticisi sınıfları çağırarak).
    *   **Login Yönetimi (`GetLoginData*`, `InsertLoginData`, `DeleteLoginData`, `InsertLogonAccount`, `DeleteLogonAccount`):** Oyuncu giriş anahtarları, hesap bilgileri ve hangi hesabın hangi sunucuda aktif olduğu gibi bilgileri yönetir.
*   **Bağımlılıklar:** Neredeyse tüm diğer `db/src` başlıkları, `common` başlıkları (özellikle `tables.h`, `service.h`), `libthecore` (ağ, olay döngüsü), `libsql` (veritabanı), `libgame` (`grid.h`). Ayrıca çeşitli Singleton yöneticilerine (`CGuildManager`, `CPrivManager` vb.) bağımlıdır.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `ClientManagerBoot.cpp`

*   **Amaç:** `CClientManager` sınıfının DB sunucusu başlatılırken ihtiyaç duyduğu ilk yükleme (`Initialize*`) ve veritabanı tablolarını (`*Proto`) belleğe alma işlemlerini içeren fonksiyonları implemente eder. Ayrıca, prototip tablolarının bellekten tekrar veritabanına yazılması (`Mirror*TableIntoDB`) için fonksiyonlar içerir (bu genellikle oyun içi komutlarla veya özel araçlarla tetiklenir).
*   **Implementasyon Detayları:**
    *   **Prototip Tablolarının Yüklenmesi (`Initialize*Table`, `Initialize*Proto`):**
        *   `InitializeMobTable()`: `mob_proto` tablosunu okur, `TMobTable` yapılarına doldurur ve `m_vec_mobTable` vektöründe saklar.
        *   `InitializeItemTable()`: `item_proto` tablosunu okur, `TItemTable` yapılarına doldurur ve `m_vec_itemTable` vektöründe saklar. Ayrıca `m_map_itemTableByVnum` haritasını doldurur.
        *   `InitializeQuestItemTable()`: `quest_item_proto` tablosunu okur.
        *   `InitializeSkillTable()`: `skill_proto` tablosunu okur.
        *   `InitializeRefineTable()`: `refine_proto` tablosunu okur.
        *   `InitializeItemAttrTable()`: `item_attr` (normal efsunlar) tablosunu okur.
        *   `InitializeItemRareTable()`: `item_attr_rare` (nadir efsunlar) tablosunu okur.
        *   `InitializeLandTable()`: `land` (lonca arazileri) tablosunu okur.
        *   `InitializeObjectProto()`: `object_proto` (binalar vb.) tablosunu okur.
        *   `InitializeObjectTable()`: `object` (oyun dünyasındaki mevcut objeler) tablosunu okur.
    *   **Diğer Başlatma İşlemleri:**
        *   `InitializeMonarch()`: `monarch` (imparatorluk lideri) bilgilerini yükler.
        *   `InitializeMailBoxTable()` (opsiyonel, `#if defined(__MAILBOX__)`): `mailbox` tablosunu yükler.
        *   `InitializeGuildSkillTable()`: `skill_power` tablosunu yükler.
        *   `InitializeBanwordTable()`: `banword` (yasaklı kelimeler) tablosunu yükler.
        *   `InitializeAnalyzeTable()`: `analyze` (muhtemelen loglama/analiz için tablo) tablosunu okur.
        *   `InitializeBeltInventoryTable()`: `belt_inventory` tablosunu okur.
        *   `InitializeMiningSkillTable()`: `mining_skill_table` tablosunu okur.
        *   `InitializeGuildCommentTable()`: `guild_comment_table` tablosunu okur.
        *   `InitializeChangeNameTable()`: `change_name` tablosunu okur.
        *   `InitializeAttachingMaterialTable()`: `attaching_material_table` (nesne dönüştürme materyalleri) tablosunu okur.
        *   `InitializeSungmaTable()` (opsiyonel, `#if defined(__CONQUEROR_LEVEL__)`): `sungma_table` (fatih seviyesi etkileri) tablosunu okur.
    *   **Prototip Tablolarını DB'ye Yazma (`Mirror*TableIntoDB`):**
        *   `MirrorMobTableIntoDB()`: Bellekteki `m_vec_mobTable` içeriğini `mob_proto` tablosuna `REPLACE INTO` ile yazar.
        *   `MirrorItemTableIntoDB()`: Bellekteki `m_vec_itemTable` içeriğini `item_proto` tablosuna `REPLACE INTO` ile yazar.
    *   **Yardımcı Fonksiyonlar:**
        *   `LoadGroupData()`: `group` tablosunu okur.
        *   `LoadGroupGroupData()`: `group_group` tablosunu okur.
        *   `LoadPolymorphTable()`: `polymorph_item` tablosunu okur.
        *   `LoadGuildMarkImage()`: Lonca sembolü resimlerini `guild_mark` klasöründen yükler.
        *   `LoadGuildMarkCRCList()`: Lonca sembolü CRC listesini yükler.
        *   `LoadGuildWarScore()`: `guild_war_score` tablosunu okur.
        *   `InitializeLocalization()`: Yerelleştirme ayarlarını (`locale_string.txt`, `mob_names.txt`, `item_names.txt`) yükler.
        *   `GetTablePostfix()`: Konfigürasyona göre tablo son ekini (`_localetest`) döndürür.
        *   `SetMaxItemID()`/`GetMaxItemID()`: Kullanılacak maksimum eşya ID'sini ayarlar/alır.
        *   `SetMaxMobID()`/`GetMaxMobID()`: Kullanılacak maksimum mob ID'sini ayarlar/alır.
        *   `SetChinaMatrix()`: `matrix_card` (Çin pazarı için matris kartı) bilgilerini yükler.
        *   `SetTaiwanMatrix()`: `tw_matrix_card` (Tayvan pazarı için matris kartı) bilgilerini yükler.
        *   `SetLoginKey()`/`GetLoginKey()`: Oyun sunucusu ile DB sunucusu arasındaki bağlantı için kullanılan anahtarı ayarlar/alır.
        *   `SetLogDB()`: Log veritabanı bağlantısını ayarlar.
*   **Önem:** DB sunucusunun çalışması için gerekli olan tüm statik verilerin (mob, eşya, beceri prototipleri, ayarlar vb.) veritabanından okunup belleğe yüklenmesinden sorumludur. Sunucunun doğru şekilde başlatılması ve çalışması için kritik bir dosyadır.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `DBManager.h`, `ProtoReader.h`, `Config.h`, `Monarch.h`, `utils.h`, `tables.h`, `length.h`, `building.h`, `locale_inc.h`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `ClientManagerEventFlag.cpp`

*   **Amaç:** `CClientManager` sınıfının global olay bayraklarını (event flags) yönetmek için kullanılan fonksiyonlarını implemente eder. Bu bayraklar genellikle `quest` tablosunda `dwPID = 0` olarak saklanan, tüm sunucuyu etkileyen durumları (örneğin, aktif etkinlikler) temsil eder.
*   **Implementasyon Detayları:**
    *   **`LoadEventFlag()`:**
        *   DB sunucusu başlatılırken çağrılır.
        *   `quest` tablosundan `dwPID = 0` olan kayıtları sorgular.
        *   Sonuçları `m_map_lEventFlag` (map: string flag adı -> long değeri) haritasına yükler.
        *   Yüklenen her bayrağı `HEADER_DG_SET_EVENT_FLAG` paketiyle bağlı olan tüm oyun sunucularına gönderir.
    *   **`SetEventFlag(TPacketSetEventFlag* p)`:**
        *   Bir oyun sunucusundan `HEADER_GD_SET_EVENT_FLAG` paketi geldiğinde çağrılır.
        *   Gelen paketteki bayrak adı (`p->szFlagName`) ve değeri (`p->lValue`) ile `m_map_lEventFlag` haritasını günceller (eğer bayrak yoksa ekler, varsa ve değer değişmişse günceller).
        *   Eğer bir değişiklik yapıldıysa, bu değişikliği `quest` tablosuna `REPLACE INTO` sorgusu ile asenkron olarak kaydeder.
        *   Değişikliği `HEADER_DG_SET_EVENT_FLAG` paketiyle diğer tüm oyun sunucularına iletir (`ForwardPacket`).
    *   **`SendEventFlagsOnSetup(CPeer* peer)`:**
        *   Yeni bir oyun sunucusu (`peer`) DB sunucusuna bağlandığında çağrılır.
        *   `m_map_lEventFlag` haritasındaki tüm mevcut global olay bayraklarını `HEADER_DG_SET_EVENT_FLAG` paketleri halinde bu yeni oyun sunucusuna gönderir.
*   **Önem:** Sunucu genelindeki olayların durumunu (örn. OX etkinliği aktif mi?) tüm oyun sunucuları arasında senkronize etmeyi ve kalıcı olarak saklamayı sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `DBManager.h`, `Main.h`, `Config.h`, `QID.h` (Query ID'leri için).

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `ClientManagerGuild.cpp`

*   **Amaç:** `CClientManager` sınıfının lonca (guild) ile ilgili işlemlerini yöneten fonksiyonlarını implemente eder. Oyun sunucularından gelen lonca ile ilgili istek paketlerini (`HEADER_GD_*`) alır, gerekli veritabanı işlemlerini yapar ve/veya `CGuildManager` singleton sınıfını kullanarak lonca durumunu günceller, ardından sonuçları veya güncellemeleri oyun sunucularına (`HEADER_DG_*`) geri gönderir.
*   **Implementasyon Detayları (Önemli Fonksiyonlar):**
    *   **`GuildCreate(CPeer* peer, DWORD dwGuildID)`:**
        *   Yeni lonca oluşturma isteğini alır.
        *   `CGuildManager::instance().Load(dwGuildID)` çağırarak lonca verilerini belleğe yükler.
        *   `HEADER_DG_GUILD_LOAD` paketini tüm oyun sunucularına gönderir.
    *   **`GuildAddMember(CPeer* peer, TPacketGDGuildAddMember* p)`:**
        *   Lonca üyesi ekleme isteğini alır.
        *   `guild_member` tablosuna yeni üyeyi `INSERT` eder.
        *   Eklenen üyenin tam bilgilerini (`player` tablosu ile join yaparak) sorgular.
        *   `HEADER_DG_GUILD_ADD_MEMBER` paketi ile üye bilgilerini tüm oyun sunucularına gönderir.
    *   **`GuildRemoveMember(CPeer* peer, TPacketGuild* p)`:**
        *   Lonca üyesi çıkarma isteğini alır (`p->dwInfo` = çıkarılan üyenin PID'si).
        *   `guild_member` tablosundan üyeyi `DELETE` eder.
        *   Üyenin tekrar loncaya girebilmesi için bekleme süresini `quest` tablosuna (`guild_manage`, `new_withdraw_time`) `REPLACE` ile kaydeder.
        *   `HEADER_DG_GUILD_REMOVE_MEMBER` paketini tüm oyun sunucularına gönderir.
    *   **`GuildDisband(CPeer* peer, TPacketGuild* p)`:**
        *   Lonca dağıtma isteğini alır.
        *   `guild`, `guild_grade`, `guild_member`, `guild_comment` tablolarından ilgili lonca ID'sine ait tüm kayıtları `DELETE` eder.
        *   Tüm eski üyelerin loncadan ayrılma zamanını (`quest` tablosu, `guild_manage`, `new_disband_time`) `REPLACE` ile kaydeder.
        *   `HEADER_DG_GUILD_DISBAND` paketini tüm oyun sunucularına gönderir.
    *   **`GuildWar(CPeer* peer, TPacketGuildWar* p)`:**
        *   Lonca savaşı ile ilgili çeşitli durumları (`p->bWar`: `SEND_DECLARE`, `REFUSE`, `RESERVE`, `ON_WAR`, `OVER`, `END`, `CANCEL`) işler.
        *   Duruma göre `CGuildManager`'ın ilgili fonksiyonlarını (`AddDeclare`, `RemoveDeclare`, `ReserveWar`, `StartWar`, `RecvWarOver`, `RecvWarEnd`, `CancelWar`) çağırır.
        *   `HEADER_DG_GUILD_WAR` paketini (durum güncellenmiş olarak) tüm oyun sunucularına gönderir.
    *   **`GuildWarScore(CPeer* peer, TPacketGuildWarScore* p)`:**
        *   Lonca savaşı skoru güncelleme isteğini alır.
        *   `CGuildManager::instance().UpdateScore()` fonksiyonunu çağırır.
    *   **`GuildChangeLadderPoint(TPacketGuildLadderPoint* p)`:**
        *   Lonca sıralama puanı değiştirme isteğini alır.
        *   `CGuildManager::instance().ChangeLadderPoint()` fonksiyonunu çağırır.
    *   **`GuildUseSkill(TPacketGuildUseSkill* p)`:**
        *   Lonca becerisi kullanma isteğini alır.
        *   `CGuildManager::instance().UseSkill()` fonksiyonunu çağırarak beceriyi kullanır ve soğuma süresini başlatır.
        *   `SendGuildSkillUsable()` fonksiyonunu çağırarak becerinin artık kullanılamaz olduğunu tüm oyun sunucularına bildirir.
    *   **`SendGuildSkillUsable(DWORD guild_id, DWORD dwSkillVnum, bool bUsable)`:**
        *   Belirtilen lonca becerisinin kullanılabilirlik durumunu (`HEADER_DG_GUILD_SKILL_USABLE_CHANGE`) tüm oyun sunucularına gönderir.
    *   **`GuildChangeMaster(TPacketChangeGuildMaster* p)`:**
        *   Lonca lideri değiştirme isteğini alır.
        *   `CGuildManager::instance().ChangeMaster()` fonksiyonunu çağırır.
        *   Başarılı olursa `HEADER_DG_ACK_CHANGE_GUILD_MASTER` paketini gönderir.
*   **Önem:** Lonca sisteminin veritabanı tarafındaki tüm kalıcı işlemlerini (üye ekleme/çıkarma, lonca dağıtma) ve lonca durumlarının (`CGuildManager` üzerinden) yönetimini sağlar. Oyun sunucuları arasındaki lonca bilgilerinin tutarlılığını sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `DBManager.h`, `Main.h`, `Config.h`, `QID.h`, `GuildManager.h`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `ClientManagerLogin.cpp`

*   **Amaç:** `CClientManager` sınıfının oyuncu giriş (login), çıkış (logout) ve isim değiştirme işlemleriyle ilgili fonksiyonlarını implemente eder. Oyun sunucularından gelen ilgili istekleri alır, veritabanı sorguları yapar, hesap ve karakter verilerini yönetir ve sonuçları oyun sunucularına iletir.
*   **Implementasyon Detayları:**
    *   **Aktif Hesap Yönetimi (`Insert/Delete/FindLogonAccount`):**
        *   `m_map_kLogonAccount` (map: string login -> `CLoginData*`) haritasını kullanarak DB sunucusuna o anda bağlı olan hesapları takip eder.
        *   `InsertLogonAccount`: Yeni bir hesap başarıyla giriş yaptığında bu haritaya ekler. Hesabın `CLoginData` nesnesini kullanır.
        *   `DeleteLogonAccount`: Hesap çıkış yaptığında haritadan siler. `CLoginData` durumuna göre (`IsDeleted()`) nesneyi de silebilir.
        *   `FindLogonAccount`: Bir hesabın zaten bağlı olup olmadığını kontrol eder.
    *   **Login Anahtarı ile Giriş (`QUERY_LOGIN_BY_KEY`, `RESULT_LOGIN_BY_KEY`, `RESULT_PLAYER_INDEX_CREATE`, `RESULT_LOGIN` [account_index=1]):**
        *   `QUERY_LOGIN_BY_KEY`: Oyun sunucusundan gelen login anahtarı isteğini alır.
            *   Anahtarın geçerliliğini, hesabın zaten bağlı olup olmadığını, login adını ve client anahtarını kontrol eder.
            *   `player_index` tablosundan karakter ID'lerini ve imparatorluğu sorgular (`QID_LOGIN_BY_KEY`).
        *   `RESULT_LOGIN_BY_KEY`: İlk sorgu sonucunu alır.
            *   `player_index` kaydı yoksa oluşturur (`QID_PLAYER_INDEX_CREATE`).
            *   Varsa, `player` tablosundan karakter detaylarını sorgular (`QID_LOGIN`).
        *   `RESULT_PLAYER_INDEX_CREATE`: `player_index` oluşturulduktan sonra `QID_LOGIN_BY_KEY` sorgusunu tekrar tetikler.
        *   `RESULT_LOGIN` (ikinci adım): Karakter detaylarını alır (`CreateAccountPlayerDataFromRes`), hesabı aktif listesine ekler (`InsertLogonAccount`) ve `HEADER_DG_LOGIN_SUCCESS` ile tam hesap/karakter verisini (`TAccountTable`) oyun sunucusuna gönderir.
    *   **Şifre ile Giriş (`RESULT_LOGIN` [account_index=0]):** (Not: Şifre ile giriş sorgusu `login_by_password` muhtemelen `ClientManagerPlayer.cpp` veya benzeri bir dosyada başlatılıyor, burada sadece sonucu işleniyor).
        *   `account` ve `player_index` tablolarından gelen sonucu alır.
        *   Hesap bilgilerini (`TAccountTable`) oluşturur (`CreateAccountTableFromRes`) ve şifreyi doğrular.
        *   Başarılı ise, karakter detaylarını almak için `QID_LOGIN` sorgusunu (ikinci adımı) tetikler.
        *   Başarısız ise `HEADER_DG_LOGIN_WRONG_PASSWD` gönderir.
    *   **Çıkış (`QUERY_LOGOUT`):**
        *   Oyun sunucusundan gelen çıkış isteğini alır.
        *   Hesabı aktif listeden siler (`DeleteLogonAccount`).
        *   Hesaptaki tüm karakterleri "logout" listesine ekler (`InsertLogoutPlayer`, bu fonksiyon muhtemelen başka bir `.cpp` dosyasında).
    *   **İsim Değiştirme (`QUERY_CHANGE_NAME`):**
        *   Oyun sunucusundan gelen isim değiştirme isteğini alır.
        *   Yeni ismin başka bir oyuncu tarafından kullanılıp kullanılmadığını kontrol eder.
        *   Uygunsa, `player` tablosunda ismi günceller ve `change_name` flag'ini sıfırlar.
        *   Sonucu (`HEADER_DG_CHANGE_NAME` veya hata paketleri) oyun sunucusuna gönderir.
    *   **Yardımcı Fonksiyonlar:**
        *   `CreateAccountTableFromRes`: `account` ve `player_index` sorgu sonucundan `TAccountTable` yapısını oluşturur.
        *   `GetLastPlayTime`: Verilen oyuncu ID'si için son oynama zamanını `player` tablosundan sorgular.
        *   `CreateAccountPlayerDataFromRes`: `player` sorgu sonucundan `TAccountTable` içindeki karakter bilgilerini doldurur (önbellekten de faydalanır).
*   **Önem:** Oyuncu kimlik doğrulama, oturum yönetimi ve temel hesap işlemlerinin DB tarafındaki mantığını yönetir. Oyun sunucuları ile veritabanı arasındaki hesap/karakter verisi akışını sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `DBManager.h`, `Main.h`, `Config.h`, `QID.h`, `ItemAwardManager.h`, `HB.h` (Heartbeat), `Cache.h`, `common/*` (özellikle `tables.h`).

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `ClientManagerPlayer.cpp`

*   **Amaç:** `CClientManager` sınıfının oyuncu (`player`) ile ilgili temel işlemlerini (yükleme, kaydetme, oluşturma, silme) ve oyuncuya bağlı diğer verilerin (eşyalar, affectler, görevler, yüksek skorlar) yönetimini sağlayan fonksiyonları implemente eder. Ayrıca oyuncu, eşya ve özel pazar önbelleklerinin yönetiminden sorumludur.
*   **Implementasyon Detayları:**
    *   **Önbellek Yönetimi:**
        *   **Oyuncu Önbelleği (`GetPlayerCache`, `PutPlayerCache`, `FlushPlayerCacheSet`):** `m_map_playerCache` (map: `DWORD` pid -> `CPlayerTableCache*`) kullanarak `TPlayerTable` verilerini bellekte tutar. `PutPlayerCache` veriyi yazar/günceller, `FlushPlayerCacheSet` değişiklikleri DB'ye yazar ve önbelleği temizler.
        *   **Eşya Önbelleği (`GetItemCacheSet`, `PutItemCache`, `DeleteItemCache`, `FlushItemCacheSet`, `CreateItemCacheSet`):** `m_map_pkItemCacheSetPtr` (map: `DWORD` pid -> `TItemCacheSet*`) kullanarak oyuncuların eşyalarını (`TPlayerItem`, `CItemCache`) önbellekte tutar. `PutItemCache` eşyayı ekler/günceller, `DeleteItemCache` siler, `FlushItemCacheSet` tüm eşyaları DB'ye yazar ve önbelleği temizler.
        *   **(Varsa) Özel Pazar Önbelleği (`GetPrivateShopCache`, `DeletePrivateShopCache`, `FlushPrivateShopCache`, `GetPrivateShopItemCacheSet`, `DeletePrivateShopItemCacheSet`, `FlushPrivateShopItemCacheSet`):** `#if defined(__PREMIUM_PRIVATE_SHOP__)` altında özel pazar ve eşya önbelleklerini yönetir.
    *   **Oyuncu Yükleme (`QUERY_PLAYER_LOAD`, `RESULT_COMPOSITE_PLAYER`, `RESULT_PLAYER_LOAD`, `RESULT_ITEM_LOAD`, `RESULT_AFFECT_LOAD`, `RESULT_QUEST_LOAD`):**
        *   `QUERY_PLAYER_LOAD`: Oyun sunucusundan oyuncu yükleme isteğini alır (`HEADER_GD_PLAYER_LOAD`).
        *   Önce oyuncu önbelleğini (`GetPlayerCache`) kontrol eder.
        *   Önbellekte varsa, önbellekteki oyuncu (`TPlayerTable`), eşya (`TPlayerItem`), affect (`TPacketAffectElement`) ve quest (`TQuestTable`) verilerini doğrudan oyun sunucusuna gönderir.
        *   Önbellekte yoksa, `player`, `item`, `quest` ve `affect` tablolarından ilgili verileri sorgulamak için `QID_PLAYER`, `QID_ITEM`, `QID_QUEST`, `QID_AFFECT` sorgularını başlatır.
        *   `RESULT_COMPOSITE_PLAYER`: Gelen sorgu sonuçlarını (`SQLMsg*`) QID'lerine göre ilgili `RESULT_*` fonksiyonlarına yönlendirir.
        *   `RESULT_PLAYER_LOAD`: `player` sorgu sonucunu işler, `TPlayerTable` oluşturur (`CreatePlayerTableFromRes`), oyuncunun `CLoginData`'sını günceller (premium bilgileri, `SetPlay(true)`) ve `HEADER_DG_PLAYER_LOAD_SUCCESS` ile oyun sunucusuna gönderir.
        *   `RESULT_ITEM_LOAD`: `item` sorgu sonucunu işler, `TPlayerItem` listesi oluşturur (`CreateItemTableFromRes`), eşya önbelleğini doldurur (`CreateItemCacheSet`, `PutItemCache`), evcil hayvan eşyaları için ek sorgular yapabilir ve `HEADER_DG_ITEM_LOAD` ile oyun sunucusuna gönderir.
        *   `RESULT_AFFECT_LOAD`: `affect` sorgu sonucunu işler ve `HEADER_DG_AFFECT_LOAD` ile oyun sunucusuna gönderir.
        *   `RESULT_QUEST_LOAD`: `quest` sorgu sonucunu işler, `HEADER_DG_QUEST_LOAD` ile oyun sunucusuna gönderir ve bekleyen eşya ödüllerini kontrol eder (`ItemAwardManager`).
    *   **Oyuncu Kaydetme (`QUERY_PLAYER_SAVE`, `CreatePlayerSaveQuery`):**
        *   `QUERY_PLAYER_SAVE`: Oyun sunucusundan gelen kaydetme isteğini (`HEADER_GD_PLAYER_SAVE`) alır ve gelen `TPlayerTable` verisini oyuncu önbelleğine yazar (`PutPlayerCache`). Gerçek veritabanı yazma işlemi önbellek flush edildiğinde yapılır.
        *   `CreatePlayerSaveQuery`: `TPlayerTable`'dan `player` tablosu için `UPDATE` sorgusu oluşturur (önbellek flush mekanizması tarafından kullanılır).
    *   **Oyuncu Oluşturma (`__QUERY_PLAYER_CREATE`):**
        *   Karakter oluşturma isteğini (`HEADER_GD_PLAYER_CREATE`) alır.
        *   İsim kontrolü ve hesap başına karakter slotu kontrolü yapar.
        *   Yeni karakteri `player` tablosuna `INSERT` eder, `player_index` tablosunu günceller.
        *   `HEADER_DG_PLAYER_CREATE_SUCCESS` veya hata paketini gönderir.
    *   **Oyuncu Silme (`__QUERY_PLAYER_DELETE`, `__RESULT_PLAYER_DELETE`):**
        *   Karakter silme isteğini (`HEADER_GD_PLAYER_DELETE`) alır.
        *   Şifre/sosyal ID ve seviye limiti kontrolü yapar.
        *   Karakteri `player_deleted` tablosuna kopyalar.
        *   Oyuncu ve eşya önbelleklerini temizler.
        *   `player_index`, `player`, `item`, `quest`, `affect`, `guild_member`, `myshop_pricelist`, `messenger_list` gibi ilgili tüm tablolardan karakterle ilişkili kayıtları `DELETE` eder.
        *   `HEADER_DG_PLAYER_DELETE_SUCCESS` veya hata paketini gönderir.
    *   **Affect Yönetimi (`QUERY_ADD_AFFECT`, `QUERY_REMOVE_AFFECT`):**
        *   Affect ekleme/çıkarma isteklerini (`HEADER_GD_ADD_AFFECT`, `HEADER_GD_REMOVE_AFFECT`) alır ve `affect` tablosunda `REPLACE INTO` veya `DELETE` sorguları çalıştırır.
    *   **Yüksek Skor (`QUERY_HIGHSCORE_REGISTER`, `RESULT_HIGHSCORE_REGISTER`):**
        *   Yüksek skor kaydetme isteğini (`HEADER_GD_HIGHSCORE_REGISTER`) alır, `highscore` tablosunu kontrol eder ve gerekirse günceller.
    *   **Logout Yönetimi (`InsertLogoutPlayer`, `DeleteLogoutPlayer`, `UpdateLogoutPlayer`):**
        *   `m_map_logout` (map: `DWORD` pid -> `TLogoutPlayer*`) haritasını kullanarak yeni çıkış yapmış oyuncuları takip eder.
        *   `InsertLogoutPlayer`: Çıkış yapan oyuncuyu listeye ekler.
        *   `DeleteLogoutPlayer`: Tekrar giriş yapan oyuncuyu listeden çıkarır.
        *   `UpdateLogoutPlayer`: Periyodik olarak çalışır, belirli bir süre (`g_iLogoutSeconds`) geçmiş oyuncuların önbelleklerini veritabanına yazar (`Flush*CacheSet`) ve listeden siler.
    *   **Yardımcı Fonksiyonlar:** `CreateItemTableFromRes`, `CreatePlayerTableFromRes`.
*   **Önem:** Oyuncu verilerinin yüklenmesi, kaydedilmesi, oluşturulması, silinmesi ve oyuncuyla ilişkili diğer kritik verilerin (eşyalar, görevler, etkiler) veritabanı ile senkronizasyonunu yönetir. Önbellekleme mekanizmaları ile DB sunucusunun performansını artırmada önemli rol oynar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `DBManager.h`, `Main.h`, `QID.h`, `ItemAwardManager.h`, `HB.h` (Heartbeat), `Cache.h`, `common/*` (özellikle `tables.h`).

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### Sınıf: `CConfig` (`Config.h`, `Config.cpp`)

*   **Amaç:** Sunucu yapılandırma dosyasını (genellikle `CONFIG` adında, `anahtar = değer` formatında bir metin dosyası) okumak, ayrıştırmak ve saklamak için kullanılan Singleton sınıfıdır. DB sunucusunun çalışması için gerekli olan MySQL bağlantı bilgileri, portlar, tablo isimleri, oyun limitleri gibi çeşitli parametrelerin merkezi olarak yönetilmesini sağlar.
*   **Tasarım:**
    *   Singleton (`singleton<CConfig>`) olarak tasarlanmıştır, global erişim `CConfig::instance()` ile sağlanır.
*   **Veri Yapıları ve Üyeler:**
    *   `m_valueMap` (private `std::map<std::string, std::string>`): Yapılandırma dosyasından okunan anahtar-değer çiftlerini saklar.
*   **Önemli Fonksiyonlar:**
    *   **`LoadFile(const char* filename)`:**
        *   Belirtilen yapılandırma dosyasını açar ve okur.
        *   Dosyayı kelime kelime (`GetWord`) işleyerek `anahtar = değer` çiftlerini bulur.
        *   Yorum satırlarını (`//`) ve boşlukları atlar.
        *   Tırnak içindeki değerleri (`"değer"`) doğru şekilde işler.
        *   Bulunan anahtar-değer çiftlerini `m_valueMap`'e ekler.
    *   **`Search(const char* key)` (private):**
        *   `m_valueMap` içinde verilen anahtarı arar ve değerin `std::string` nesnesine pointer döndürür (bulamazsa `NULL`).
    *   **`Get(const char* key)` (private):**
        *   `Search()` kullanarak anahtarın string değerini döndürür (bulamazsa boş string).
    *   **`GetValue(const char* key, TYPE* dest)` (çeşitli türler için):**
        *   En sık kullanılan arayüz fonksiyonlarıdır.
        *   `Get()` ile string değeri alır.
        *   `str_to_number` (sayısal türler için) veya `strlcpy` (karakter dizisi için) kullanarak değeri istenen türe dönüştürür ve `dest` işaretçisine yazar.
        *   Başarılı olup olmadığını boolean olarak döndürür.
    *   **`GetParam(const char* key, int index, DWORD* Param)` (private) / `GetTwoValue(const char* key, DWORD* dest1, DWORD* dest2)**:
        *   Tek bir anahtara karşılık gelen birden fazla değeri (boşlukla ayrılmış) okumak için kullanılır (örn: `BIND_PORT = 13001 13061`). `GetParam` belirtilen index'teki değeri alır, `GetTwoValue` ilk iki değeri alır.
*   **Kullanım Alanı:** DB sunucusunun çalışması için gerekli olan tüm statik verilerin (mob, eşya, beceri prototipleri, ayarlar vb.) veritabanından okunup belleğe yüklenmesinden sorumludur. Sunucunun doğru şekilde başlatılması ve çalışması için kritik bir dosyadır.
*   **Bağımlılıklar:** `stdafx.h`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### Sınıf: `CDBManager` (`DBManager.h`, `DBManager.cpp`)

*   **Amaç:** DB sunucusunun MySQL veritabanı ile olan tüm etkileşimini yöneten Singleton sınıfıdır. Farklı amaçlar (oyuncu, hesap, genel vb.) ve farklı sorgu türleri (eşzamanlı, eşzamansız, geri dönüşlü/dönüşsüz) için veritabanı bağlantılarını (`CAsyncSQL2` nesneleri) yönetir ve sorgu gönderme/alma işlemlerini kolaylaştıran bir arayüz sunar.
*   **Tasarım:**
    *   Singleton (`singleton<CDBManager>`) olarak tasarlanmıştır, global erişim `CDBManager::instance()` ile sağlanır.
    *   Farklı veritabanı işlem türleri için `eSQL_SLOT` enum'u ile ayrılmış bağlantı havuzları kullanır (`SQL_PLAYER`, `SQL_ACCOUNT`, `SQL_COMMON`, `SQL_HOTBACKUP`).
    *   Her slot için 3 farklı `CAsyncSQL2` nesnesi tutar:
        *   `m_mainSQL`: Geri dönüş beklenen eşzamansız sorgular için (sonuçlar `PopResult` ile alınır).
        *   `m_directSQL`: Eşzamanlı (bloke eden) sorgular ve `EscapeString` için.
        *   `m_asyncSQL`: Geri dönüş beklenmeyen eşzamansız sorgular için.
*   **Veri Yapıları ve Üyeler:**
    *   `m_mainSQL`, `m_directSQL`, `m_asyncSQL` (private `CAsyncSQL2* [SQL_MAX_NUM]` dizileri): Bağlantı nesnelerini tutar.
    *   `CQueryInfo` (struct): `ReturnQuery` ile gönderilen sorgularla ilişkilendirilecek ek bilgileri (tür, kimlik, veri pointer'ı) tutar.
*   **Önemli Fonksiyonlar:**
    *   **`Connect(int iSlot, const char* host, ...)`:** Belirtilen slot için 3 `CAsyncSQL2` nesnesini oluşturur, MySQL sunucusuna bağlar (`Setup()`) ve karakter setini ayarlar (`SetLocale`).
    *   **`ReturnQuery(const char* c_pszQuery, int iType, DWORD dwIdent, void* pvData, int iSlot = SQL_PLAYER)`:**
        *   Geri dönüş beklenen bir sorguyu (`SELECT` gibi) eşzamansız olarak `m_mainSQL` üzerinden gönderir.
        *   Sorguyla birlikte bir `CQueryInfo` nesnesi (tür, kimlik, özel veri) ilişkilendirir.
        *   Sonuç `PopResult` ile alındığında bu `CQueryInfo` da elde edilir.
    *   **`AsyncQuery(const char* c_pszQuery, int iSlot = SQL_PLAYER)`:**
        *   Geri dönüş beklenmeyen bir sorguyu (`INSERT`, `UPDATE`, `DELETE` gibi) eşzamansız olarak `m_asyncSQL` üzerinden gönderir.
    *   **`DirectQuery(const char* c_pszQuery, int iSlot = SQL_PLAYER)`:**
        *   Bir sorguyu eşzamanlı (bloke ederek) `m_directSQL` üzerinden gönderir ve `SQLMsg*` sonucunu hemen döndürür.
    *   **`PopResult()` / `PopResult(eSQL_SLOT slot)`:**
        *   `m_mainSQL` havuz(lar)ından tamamlanmış bir sorgu sonucunu (`SQLMsg*`, içinde `MYSQL_RES*` ve `CQueryInfo*` bulunur) alır. Sonuç yoksa `NULL` döner.
    *   **`EscapeString(void* to, const void* from, unsigned long length, int iSlot = SQL_PLAYER)`:**
        *   `m_directSQL` bağlantısını kullanarak `mysql_real_escape_string` fonksiyonunu çağırır ve SQL injection'a karşı string'i güvenli hale getirir.
    *   **`SetLocale(const char* szLocale)` / `QueryLocaleSet()`:**
        *   Tüm bağlantıların karakter setini (`SET NAMES`) ayarlar.
    *   **`Count*()` Fonksiyonları:** Bağlantı havuzlarındaki bekleyen/tamamlanan sorgu ve sonuç sayılarını izlemek için kullanılır.
    *   **`Quit()`/`Clear()`:** Sorgu işleme döngülerini durdurur ve bağlantı nesnelerini temizler.
*   **Kullanım Alanı:** DB sunucusundaki diğer tüm sınıfların (özellikle `CClientManager`) veritabanı ile etkileşime girmesi için merkezi arayüz görevi görür. Sorgu gönderme ve sonuç alma işlemlerini yönetir.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h` (dolaylı), `libsql/AsyncSQL.h`, `<mysql/mysql.h>`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### Sınıf: `CGuildManager` (`GuildManager.h`, `GuildManager.cpp`)

*   **Amaç:** DB sunucusunda tüm loncalarla ilgili verileri (temel bilgiler, savaşlar, sıralama, beceri cooldown'ları, rezerve savaşlar) bellekte tutan ve yöneten Singleton sınıfıdır. Lonca savaşı mantığını, skorlamayı, ödüllendirmeyi, ladder puanlarını ve para işlemlerini yönetir.
*   **Tasarım:**
    *   Singleton (`singleton<CGuildManager>`) olarak tasarlanmıştır, global erişim `CGuildManager::instance()` ile sağlanır.
    *   Lonca verilerini (`m_map_kGuild`), aktif savaşları (`m_WarMap`), rezerve savaşları (`m_map_kWarReserve`), savaş ilanlarını (`m_DeclareMap`), savaş bitiş zamanlarını (`m_mapGuildWarEndTime`) ve beceri cooldown'larını (`m_SkillUseMap`) çeşitli map ve set yapılarında tutar.
    *   Savaşların bitiş zamanlarını (`m_pqOnWar`), beceri cooldown'larını (`m_pqSkill`) ve rezerve savaşların başlangıç zamanlarını (`m_pqWaitStart`) takip etmek için priority queue'lar kullanır.
    *   Lonca savaşı güç ve handikap hesaplamaları için `libpoly` kütüphanesini kullanır (`polyPower`, `polyHandicap`).
*   **Veri Yapıları ve Üyeler (Önemliler):**
    *   `m_map_kGuild` (private `kGuildMap` -> `map<DWORD, TGuild>`): Loncaların temel bilgilerini (isim, puan, w/d/l, altın, seviye) saklar.
    *   `m_WarMap` (private `GuildWarMap` -> `map<DWORD, map<DWORD, TGuildWarInfo>>`): Aktif lonca savaşlarının bilgilerini (karşı lonca ID -> `TGuildWarInfo`) saklar. GID1 < GID2 olacak şekilde saklanır.
    *   `m_pqOnWar` (private `priority_queue<pair<time_t, TGuildWarPQElement*>>`): Aktif savaşların bitiş zamanına göre sıralandığı kuyruk.
    *   `m_DeclareMap` (private `GuildDeclareInfoSet`): Bekleyen savaş ilanlarını saklar.
    *   `m_map_kWarReserve` (private `WarReserveMap` -> `map<DWORD, CGuildWarReserve*>`): Rezerve edilmiş savaşları (rezervasyon ID -> `CGuildWarReserve` nesnesi) saklar.
    *   `m_pqWaitStart` (private `priority_queue<pair<time_t, TGuildWaitStartInfo>>`): Başlaması beklenen rezerve savaşları başlangıç zamanına göre saklar.
    *   `m_pqSkill` (private `priority_queue<pair<time_t, TGuildSkillUsed>>`): Kullanılan lonca becerilerinin cooldown bitiş zamanlarını saklar.
    *   `map_kLadderPointRankingByGID` (private `kLadderPointRankingMap`): Loncaların sıralamadaki yerini saklar (sadece ilk 20).
*   **Önemli Fonksiyonlar:**
    *   **`Initialize()`:** Sunucu başlangıcında çağrılır. `guild` tablosundan verileri yükler, `CONFIG`'den savaş formüllerini alır, sıralamayı sorgular ve rezerve savaşları (`BootReserveWar`) yükler.
    *   **`Update()`:** Ana döngüde periyodik olarak çağrılır. Biten savaşları, dolan cooldown'ları ve başlaması gereken rezerve savaşları işler.
    *   **`Load(DWORD dwGuildID)` / `TouchGuild(DWORD GID)`:** Lonca verilerini yükler veya erişir/oluşturur.
    *   **`StartWar(BYTE bType, DWORD GID1, DWORD GID2, ...)`:** Yeni bir lonca savaşı başlatır, `m_WarMap` ve `m_pqOnWar`'a ekler.
    *   **`WarEnd(DWORD GID1, DWORD GID2, bool bForceDraw = false)`:** Bir savaşı sonlandırır, sonucu belirler (`ProcessDraw`/`ProcessWinLose`), ödülleri dağıtır, ladder puanlarını günceller, sıralamayı yeniler ve savaşı temizler (`RemoveWar`).
    *   **`RecvWarOver(...)` / `RecvWarEnd(...)`:** Oyun sunucularından gelen savaş bitiş/sonuç bilgilerini işler.
    *   **`UpdateScore(...)`:** Savaş sırasındaki skorları günceller.
    *   **`AddDeclare(...)` / `RemoveDeclare(...)`:** Savaş ilanlarını yönetir.
    *   **`ReserveWar(...)` / `ProcessReserveWar()` / `Bet(...)`:** Savaş rezervasyonlarını ve bahisleri yönetir.
    *   **`ChangeLadderPoint(...)`:** Lonca ladder puanını günceller.
    *   **`QueryRanking()` / `ResultRanking()` / `GetRanking()`:** Lonca sıralamasını yönetir.
    *   **`UseSkill(...)`:** Lonca becerisi kullanımını ve cooldown'unu yönetir.
    *   **`DepositMoney(...)` / `WithdrawMoney(...)` / `MoneyChange(...)`:** Lonca para işlemlerini yönetir.
    *   **`ChangeMaster(...)`:** Lonca liderini değiştirir.
    *   **`Dungeon(...)` / `DungeonCooldown(...)` / `DungeonStart(...)`:** (Varsa) Lonca zindanı durumunu yönetir.
*   **Kullanım Alanı:** `CClientManager` tarafından lonca ile ilgili paketler alındığında çağrılır. Bellekteki lonca durumlarını yönetir, zamanlanmış olayları (savaş bitişi, cooldown) işler ve veritabanı ile etkileşime girerek kalıcı değişiklikleri yapar.
*   **Bağımlılıklar:** `stdafx.h`, `Peer.h`, `Main.h`, `ClientManager.h`, `QID.h`, `Config.h`, `libsql.h`, `libpoly/Poly.h`, `<queue>`, `<utility>`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### Sınıf: `PlayerHB` (`HB.h`, `HB.cpp`)

*   **Amaç:** Oyuncu verileri için basit bir "Hot Backup" (Sıcak Yedekleme) mekanizması sağlayan Singleton sınıfıdır. Belirli aralıklarla (varsayılan olarak saatte bir) aktif oyuncuların verilerini `player` tablosundan saatlik olarak adlandırılan yedek tablolara (`hb_YYMMDDHH_player`) kopyalar.
*   **Tasarım:**
    *   Singleton (`singleton<PlayerHB>`) olarak tasarlanmıştır, global erişim `PlayerHB::instance()` ile sağlanır.
*   **Veri Yapıları ve Üyeler:**
    *   `m_map_data` (private `DataMap` -> `map<DWORD, time_t>`): Hangi oyuncunun (PID) en son ne zaman yedeklendiğini (zaman damgası) takip eder.
    *   `m_stCreateTableQuery` (private `std::string`): Ana `player` tablosunun `CREATE TABLE` ifadesini saklar (yedek tabloları oluşturmak için kullanılır).
    *   `m_stTableName` (private `std::string`): En son işlem yapılan yedek tablosunun adını saklar (yeni tablo oluşturma gerekliliğini kontrol etmek için).
    *   `m_iExpireTime` (private `int`): Bir oyuncunun verilerinin tekrar yedeklenmeden önce geçmesi gereken minimum süreyi saniye cinsinden belirler (varsayılan 3600).
*   **Önemli Fonksiyonlar:**
    *   **`Initialize()`:** DB sunucusu başlangıcında çağrılır. `SHOW CREATE TABLE player...` sorgusu ile ana oyuncu tablosunun yapısını alır ve `m_stCreateTableQuery`'de saklar.
    *   **`Put(DWORD id)`:**
        *   Oyuncu verisi kaydedildiğinde (`CClientManagerPlayer::PutPlayerCache` içinde) çağrılır.
        *   Verilen oyuncu ID'sinin `m_map_data`'da olup olmadığını ve son yedekleme zamanının `m_iExpireTime`'ı aşıp aşmadığını kontrol eder.
        *   Eğer yedekleme gerekiyorsa `Query(id)` fonksiyonunu çağırır ve `m_map_data`'daki zaman damgasını günceller.
    *   **`Query(DWORD id)` (private):**
        *   Asıl yedekleme işlemini yapar.
        *   Mevcut saate göre hedef yedek tablosunun adını (`hb_YYMMDDHH_player...`) oluşturur.
        *   Eğer hedef tablo adı değişmişse (`m_stTableName` ile karşılaştırır), `m_stCreateTableQuery`'yi kullanarak `CREATE TABLE IF NOT EXISTS` sorgusu ile yeni yedek tablosunu oluşturur (`SQL_HOTBACKUP` slotu, `DirectQuery`).
        *   `REPLACE INTO <yedek_tablo> SELECT * FROM player... WHERE id = ...` sorgusu ile oyuncunun güncel verilerini yedek tablosuna kopyalar (`SQL_HOTBACKUP` slotu, `AsyncQuery`).
*   **Kullanım Alanı:** Oyuncu verilerinin düzenli aralıklarla yedeklenmesini sağlayarak olası veri kayıplarına karşı bir güvenlik katmanı oluşturur. Özellikle sunucu çökmesi gibi durumlarda veri kurtarma amacıyla kullanılabilir.
*   **Bağımlılıklar:** `stdafx.h`, `Main.h`, `DBManager.h`.

(Buraya diğer .cpp dosyalarının analizleri eklenecek)

### `LoginData.h` / `LoginData.cpp` (`CLoginData` Sınıfı)

**Amaç:** Bir oyuncu hesabının DB sunucusundaki aktif oturum bilgilerini geçici olarak saklamak. Bu bilgiler arasında login anahtarı, client anahtarı, bağlı IP adresi, premium durumu ve diğer hesapla ilişkili veriler bulunur. `CLoginData` nesneleri `CClientManager` tarafından yönetilir.

**Kaynak Kod Dosyaları (`LoginData.cpp`)**

*   **`CLoginData::CLoginData()` (Yapıcı):** Yeni bir `CLoginData` nesnesi oluşturulduğunda çağrılır ve üye değişkenleri varsayılan değerlere ayarlar.
*   **`CLoginData::~CLoginData()` (Yıkıcı):** Nesne yok edildiğinde çağrılır.
*   **Çeşitli `Set*` ve `Get*` Metotları:** Sınıfın sakladığı verilere (login key, client key, IP, premium tipi, son oynama zamanı vb.) erişmek ve bu verileri güncellemek için kullanılır. Örneğin:
    *   `SetKey()`, `GetKey()`: Login anahtarını ayarlar/alır.
    *   `SetClientKey()`, `GetClientKey()`: Client anahtarını (session key) ayarlar/alır.
    *   `SetConnectedPeerHandle()`, `GetConnectedPeerHandle()`: Bağlı oyun sunucusunun (peer) handle'ını ayarlar/alır.
    *   `SetPremium()`, `GetPremium()`: Premium tipini ve bitiş zamanını ayarlar/alır.
    *   `SetAccountID()`, `GetAccountID()`: Hesap ID'sini ayarlar/alır.
    *   `SetLogin()`, `GetLogin()`: Hesap kullanıcı adını ayarlar/alır.
    *   `SetBillType()`, `GetBillType()`: Fatura tipini ayarlar/alır.
    *   `SetBillExpire()`, `GetBillExpire()`: Fatura bitiş zamanını ayarlar/alır.
*   **`IsKey()`, `IsClientKey()`:** Verilen anahtarların saklanan anahtarlarla eşleşip eşleşmediğini kontrol eder.
*   **`SetDeleted()`:** Hesabın silinmiş olarak işaretlenmesini sağlar.
*   **`SetLastPlayTime()`:** Son oynama zamanını günceller.

---

### `Main.h` / `Main.cpp`

**Amaç:** DB sunucusu (`db` process) uygulamasının ana giriş noktasıdır. Sunucunun başlatılması, yapılandırılması, ana döngünün çalıştırılması ve sonlandırılması ile ilgili temel fonksiyonları içerir. Global yapılandırma değişkenlerini yönetir.

**Header Dosyası (`Main.h`)**

*   **`Start()`:** Sunucuyu başlatan ana fonksiyonun prototipi.
*   **`End()`:** Sunucuyu durduran fonksiyonun prototipi (ancak implementasyonu `main` içinde).
*   **`GetTablePostfix()`:** Veritabanı tabloları için kullanılan son eki (örneğin, `_test`) döndüren fonksiyonun prototipi.
*   **`GetPlayerDBName()`:** Oyuncu veritabanının adını döndüren fonksiyonun prototipi.

**Kaynak Kod Dosyaları (`Main.cpp`)**

*   **`main()` Fonksiyonu:**
    *   Programın başlangıç noktasıdır.
    *   Sürüm bilgisini yazar (`WriteVersion`).
    *   Gerekli tüm Singleton yönetici sınıflarının (`CConfig`, `CNetPoller`, `CDBManager`, `CClientManager`, `PlayerHB`, `CGuildManager`, `CPrivManager`, `CMoneyLog`, `ItemAwardManager`, `marriage::CManager`, `CMonarch`, `CBlockCountry`, `CItemIDRangeManager`) nesnelerini oluşturur.
    *   `Start()` fonksiyonunu çağırarak sunucu yapılandırmasını ve başlatma işlemlerini gerçekleştirir. Başarısız olursa programdan çıkar.
    *   Diğer yönetici sınıflarının (`GuildManager`, `MarriageManager`, `BlockCountry`, `ItemIDRangeManager`) başlangıç ayarlarını yapar.
    *   `CClientManager::instance().MainLoop()` çağrısı ile ana olay döngüsünü başlatır. Bu döngü, ağ olaylarını dinler ve işler.
    *   Program sonlandırıldığında (örneğin `SIGTERM`), zamanlayıcıları devre dışı bırakır (`signal_timer_disable`).
    *   `DBManager::Quit()` ile veritabanı bağlantılarını kapatır.
    *   Henüz tamamlanmamış veritabanı sorgularının bitmesini bekler.
*   **`Start()` Fonksiyonu:**
    *   `conf.txt` dosyasını yükler (`CConfig::instance().LoadFile`). Başarısız olursa `false` döner.
    *   Yapılandırma dosyasından çeşitli parametreleri okur:
        *   `TEST_SERVER`: Sunucunun test modu durumunu ayarlar (`g_test_server`).
        *   `LOG`: Loglamanın aktif olup olmadığını ayarlar (`g_log`).
        *   `LOG_KEEP_DAYS`: Log dosyalarının kaç gün saklanacağını belirler.
        *   `CLIENT_HEART_FPS`: `thecore` kütüphanesi için heartbeat frekansını ayarlar.
        *   `LOCALE`: Sunucu yerelleştirmesini (`g_stLocale`) ve buna bağlı olarak yerel isim sütununu (`g_stLocaleNameColumn`) ayarlar. Bazı yerelleştirmelerde sıcak yedeklemeyi (`g_bHotBackup`) devre dışı bırakabilir.
        *   `DISABLE_HOTBACKUP`: Sıcak yedeklemeyi manuel olarak devre dışı bırakma seçeneği.
        *   `TABLE_POSTFIX`: Veritabanı tablo son ekini (`g_stTablePostfix`) ayarlar.
        *   `*_CACHE_FLUSH_SECONDS`: Oyuncu, eşya ve özel pazar fiyat listesi önbelleklerinin ne sıklıkla veritabanına yazılacağını (`g_i*CacheFlushSeconds`) belirler.
        *   `CACHE_FLUSH_LIMIT_PER_SECOND`: Saniye başına izin verilen maksimum önbellek yazma işlemi sayısını ayarlar.
        *   `PLAYER_ID_START`: Yeni oyuncular için başlangıç ID'sini ayarlar.
        *   `BIND_PORT`: DB sunucusunun dinleyeceği portu ayarlar.
        *   `DB_MYSQL_*`: Ana veritabanı (oyuncu, hesap, ortak) bağlantı bilgilerini okur.
        *   `HOTBACKUP_MYSQL_*`: Sıcak yedekleme veritabanı bağlantı bilgilerini okur (eğer `g_bHotBackup` aktifse).
    *   `thecore_init()` ile ana olay döngüsü mekanizmasını başlatır.
    *   `signal_timer_enable()` ile periyodik sinyal zamanlayıcısını başlatır.
    *   `CDBManager::instance().Connect()` ile veritabanı bağlantılarını (ana ve isteğe bağlı sıcak yedekleme) kurar.
    *   `CClientManager::instance().Initialize()` ile `ClientManager`'ı ve ağ dinleyicisini başlatır.
    *   Başarılı olursa `true` döner.
*   **`emergency_sig()` Fonksiyonu:**
    *   `SIGSEGV` (segmentation fault) veya `SIGUSR1` gibi sinyalleri yakalamak için kullanılır.
    *   Yakalanan sinyali loglar. `SIGSEGV` durumunda programı sonlandırır (`abort`).
*   **`SetTablePostfix()` / `GetTablePostfix()`:** Global `g_stTablePostfix` değişkenini ayarlar ve döndürür.
*   **`SetPlayerDBName()` / `GetPlayerDBName()`:** Global `g_stPlayerDBName` değişkenini ayarlar ve döndürür.
*   **Global Değişkenler:** `g_stTablePostfix`, `g_stLocaleNameColumn`, `g_stLocale`, `g_stPlayerDBName`, `g_bHotBackup`, `g_test_server`, `g_i*CacheFlushSeconds`, `g_log` gibi birçok global değişken bu dosyada tanımlanır ve `Start()` fonksiyonunda yapılandırma dosyasından okunan değerlerle doldurulur.

---

### `Marriage.h` / `Marriage.cpp` (`marriage::CManager` Sınıfı)

**Amaç:** Oyuncular arasındaki evlilik ilişkilerini (nişanlanma, evlenme, boşanma, sevgi puanı) ve düğün süreçlerini yönetir. Evlilik verilerini veritabanından yükler, bellekte tutar, günceller ve oyun sunucuları ile senkronize eder. Singleton deseni kullanır.

**Header Dosyası (`Marriage.h`)**

*   **`namespace marriage`:** Evlilikle ilgili tüm yapıları ve sınıfı içerir.
*   **`TWeddingInfo` Yapısı:** Düğün bitiş zamanını (`dwTime`) ve ilgili oyuncu ID'lerini (`dwPID1`, `dwPID2`) saklar.
*   **`TWedding` Yapısı:** Düğün başlangıç zamanını (`dwTime`), yapılacağı harita indeksini (`dwMapIndex`) ve oyuncu ID'lerini (`dwPID1`, `dwPID2`) saklar.
*   **`TMarriage` Yapısı:** Bir evlilik ilişkisinin detaylarını tutar:
    *   `pid1`, `pid2`: Evlenen oyuncuların ID'leri.
    *   `love_point`: Çiftin arasındaki sevgi puanı.
    *   `time`: Evliliğin gerçekleştiği zaman damgası.
    *   `is_married`: Evlilik durumunu belirtir (0: Nişanlı, 1: Evli).
    *   `name1`, `name2`: Oyuncuların isimleri.
    *   `GetOther(DWORD PID)`: Verilen oyuncunun eşinin ID'sini döndürür.
*   **Typedef'ler:**
    *   `MarriageSet`: `TMarriage*` içeren bir set.
    *   `MarriageMap`: `DWORD` (Oyuncu ID) -> `TMarriage*` eşlemesi.
    *   `RunningWeddingMap`: `std::pair<DWORD, DWORD>` (Oyuncu çifti) -> `TWedding` eşlemesi.
*   **`CManager` Sınıfı (Singleton):**
    *   `Initialize()`: Evlilik yöneticisini başlatır.
    *   `Get(DWORD dwPlayerID)`: Verilen ID'li oyuncunun evlilik bilgilerini döndürür.
    *   `IsMarried(DWORD dwPlayerID)`: Oyuncunun evli olup olmadığını kontrol eder.
    *   `Add()`: Yeni bir evlilik (nişan) ekler.
    *   `Remove()`: Bir evliliği kaldırır (boşanma).
    *   `Update()`: Evlilik bilgilerini (sevgi puanı, durum) günceller.
    *   `EngageToMarriage()`: Nişanlı çifti evli yapar.
    *   `ReadyWedding()`: Düğün için hazırlık yapar.
    *   `EndWedding()`: Düğünü sonlandırır.
    *   `OnSetup(CPeer* peer)`: Yeni bağlanan oyun sunucusuna evlilik bilgilerini gönderir.
    *   `Update()`: Zamanlanmış düğün olaylarını işler.
    *   Özel Üyeler: `m_Marriages` (tüm evlilikler), `m_MarriageByPID` (ID ile hızlı erişim), `m_pqWeddingStart` (başlayacak düğünler), `m_pqWeddingEnd` (bitecek düğünler), `m_mapRunningWedding` (devam eden düğünler).

**Kaynak Kod Dosyaları (`Marriage.cpp`)**

*   **`WEDDING_LENGTH`:** Düğün süresini saniye cinsinden tanımlayan sabit (1 saat).
*   **Karşılaştırma Operatörleri:** `TWedding` ve `TWeddingInfo` yapıları için zaman damgalarına göre karşılaştırma operatörleri tanımlanmıştır (öncelik kuyrukları için gerekli).
*   **`CManager::Initialize()`:**
    *   Veritabanından (`marriage` tablosu) tüm evlilik kayıtlarını sorgular (`is_married=0` olanlar hariç, bunlar silinir).
    *   Her kayıt için `player` tablosundan oyuncu isimlerini alır.
    *   Alınan bilgilerle `TMarriage` nesneleri oluşturur ve bunları `m_Marriages` seti ile `m_MarriageByPID` map'ine ekler.
*   **`CManager::Get()`:** `m_MarriageByPID` map'ini kullanarak verilen oyuncu ID'sine karşılık gelen `TMarriage` nesnesini bulur ve döndürür.
*   **`Align(DWORD& rPID1, DWORD& rPID2)`:** Verilen iki oyuncu ID'sini küçükten büyüğe sıralar (veritabanında tutarlılık için).
*   **`CManager::Add()`:**
    *   Oyuncuların zaten evli olup olmadığını kontrol eder.
    *   Oyuncu ID'lerini `Align` ile sıralar.
    *   Veritabanına yeni bir evlilik kaydı ekler (`love_point=0`, `is_married=0`).
    *   Yeni `TMarriage` nesnesi oluşturup bellekteki set ve map'e ekler.
    *   Tüm oyun sunucularına `HEADER_DG_MARRIAGE_ADD` paketini göndererek yeni evliliği bildirir.
*   **`CManager::Update()`:**
    *   Verilen oyuncuların evli olup olmadığını kontrol eder.
    *   ID'leri `Align` ile sıralar.
    *   Veritabanındaki ilgili evlilik kaydının `love_point` ve `is_married` değerlerini günceller.
    *   Bellekteki `TMarriage` nesnesinin ilgili alanlarını günceller.
    *   Tüm oyun sunucularına `HEADER_DG_MARRIAGE_UPDATE` paketini gönderir.
*   **`CManager::Remove()`:**
    *   Verilen oyuncuların evli olup olmadığını kontrol eder.
    *   ID'leri `Align` ile sıralar.
    *   Veritabanından ilgili evlilik kaydını siler.
    *   Bellekteki set ve map'ten ilgili `TMarriage` nesnesini kaldırır ve nesneyi siler (`delete`).
    *   Tüm oyun sunucularına `HEADER_DG_MARRIAGE_REMOVE` paketini gönderir.
*   **`CManager::EngageToMarriage()`:**
    *   Çiftin zaten evli olup olmadığını kontrol eder.
    *   ID'leri `Align` ile sıralar.
    *   Veritabanındaki kaydın `is_married` alanını 1 olarak günceller.
    *   Bellekteki `TMarriage` nesnesinin `is_married` alanını 1 yapar.
    *   Tüm oyun sunucularına `HEADER_DG_MARRIAGE_UPDATE` paketini gönderir.
*   **`CManager::ReadyWedding()`:**
    *   Belirtilen harita indeksinde, belirtilen oyuncular için bir düğün başlatır.
    *   Mevcut zaman artı `WEDDING_LENGTH` kadar bir bitiş zamanı ile `TWeddingInfo` oluşturur ve bunu `m_pqWeddingEnd` kuyruğuna ekler.
    *   Tüm oyun sunucularına `HEADER_DG_WEDDING_READY` paketini gönderir.
*   **`CManager::EndWedding()`:**
    *   Verilen oyuncular arasındaki düğünü bitirir (muhtemelen `m_mapRunningWedding`'den çıkarır).
    *   Tüm oyun sunucularına `HEADER_DG_WEDDING_END` paketini gönderir.
*   **`CManager::OnSetup()`:** Bağlanan yeni oyun sunucusuna mevcut tüm evlilikleri (`HEADER_DG_MARRIAGE_ADD`) ve devam eden düğünleri (`HEADER_DG_WEDDING_READY`) gönderir.
*   **`CManager::Update()`:**
    *   Periyodik olarak çalıştırılır.
    *   `m_pqWeddingEnd` kuyruğunu kontrol eder. Zamanı gelen düğünler için (`dwTime <= now`) `EndWedding` fonksiyonunu çağırır.
    *   (Not: `m_pqWeddingStart` ile ilgili bir işlem bu fonksiyonda görünmüyor, muhtemelen eksik veya farklı bir yerde yönetiliyor.)

### `ClientManagerHorseName.cpp`

*   **Amaç:** `CClientManager` sınıfının, oyuncuların sahip olduğu atlara isim verme ve bu isimleri yönetme işlevselliğini içeren fonksiyonlarını implemente eder.
*   **Implementasyon Detayları:**
    *   **`UpdateHorseName(TPacketUpdateHorseName* data, CPeer* peer)`:**
        *   Oyun sunucusundan `HEADER_GD_UPDATE_HORSE_NAME` paketi ile gelen at ismi güncelleme isteğini alır.
        *   `horse_name` tablosuna `REPLACE INTO` sorgusu ile oyuncunun ID'si (`data->dwPlayerID`) ve yeni at ismi (`data->szHorseName`) bilgisini yazar veya günceller.
        *   Bu değişikliği paketi gönderen oyun sunucusu hariç (`peer`) diğer tüm oyun sunucularına `HEADER_DG_UPDATE_HORSE_NAME` paketi ile iletir (`ForwardPacket`).
    *   **`AckHorseName(DWORD dwPID, CPeer* peer)`:**
        *   Oyun sunucusundan `HEADER_GD_REQ_HORSE_NAME` paketi ile gelen at ismi sorgulama isteğini alır.
        *   `horse_name` tablosundan verilen oyuncu ID'sine (`dwPID`) ait at ismini `SELECT` sorgusu ile alır.
        *   Sonucu (`TPacketUpdateHorseName` yapısı içinde, isim yoksa boş olarak) `HEADER_DG_ACK_HORSE_NAME` paketi ile isteği yapan oyun sunucusuna (`peer`) gönderir.
*   **Önem:** Oyuncuların atlarına verdikleri isimlerin kalıcı olarak saklanmasını ve oyun sunucuları arasında senkronize edilmesini sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `Main.h`, `ClientManager.h`.

### `ClientManagerParty.cpp`

*   **Amaç:** `CClientManager` sınıfının oyuncu partileriyle (gruplarıyla) ilgili durum yönetimini içeren fonksiyonlarını implemente eder. Parti oluşturma, dağıtma, üye ekleme/çıkarma ve üye bilgilerini (rol, seviye) güncelleme gibi işlemleri yönetir. DB sunucusu, partilerin durumunu bellekte (`m_map_pkChannelParty`) tutar ve oyun sunucuları arasında senkronizasyonu sağlar.
*   **Veri Yapısı (`ClientManager.h` içinde tanımlı):**
    *   `TPartyChannelMap m_map_pkChannelParty`: Kanal ID'sine göre parti bilgilerini tutan bir harita. Değeri `TPartyMap`'tir.
    *   `TPartyMap` (typedef `std::map<DWORD, TPartyMember>`): Parti liderinin PID'sinden parti üyelerine (`TPartyMember`) eşleme yapar.
    *   `TPartyMember` (typedef `std::map<DWORD, TPartyInfo>`): Parti üyesinin PID'sinden parti bilgilerine (`TPartyInfo`) eşleme yapar.
    *   `TPartyInfo` (struct): Üyenin rolünü (`bRole`) ve seviyesini (`bLevel`) tutar.
*   **Implementasyon Detayları:**
    *   Tüm fonksiyonlar (`QUERY_PARTY_*`) belirli bir oyun sunucusundan (`peer`) ve kanaldan gelen isteği alır.
    *   İlgili kanalın parti haritasını (`m_map_pkChannelParty[peer->GetChannel()]`) referans alır (`TPartyMap& pm`).
    *   Gelen isteğe göre (`CREATE`, `DELETE`, `ADD`, `REMOVE`, `STATE_CHANGE`, `SET_MEMBER_LEVEL`) bu harita üzerinde gerekli kontrolleri (parti var mı? üye var mı?) ve güncellemeleri (ekleme, silme, değer değiştirme) yapar.
    *   Yapılan değişikliği, isteği gönderen oyun sunucusu hariç (`peer`), aynı kanaldaki diğer tüm oyun sunucularına ilgili `HEADER_DG_PARTY_*` paketini ileterek (`ForwardPacket`) bildirir.
    *   Parti bilgileri sadece bellekte tutulur, bu dosyada veritabanı işlemi yapılmaz. Parti bilgileri kalıcı değildir ve sunucu yeniden başladığında veya kanal kapandığında kaybolur.
*   **Önem:** Oyun içindeki parti sisteminin temel durum yönetimini sağlar. Farklı kanallardaki veya aynı kanaldaki oyuncuların parti bilgilerinin tutarlı olmasını sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `Config.h`, `DBManager.h` (dolaylı, `ForwardPacket` için olabilir), `QID.h` (Kullanılmıyor gibi görünse de dahil edilmiş).

### `ClientManagerPrivateShop.cpp`

*   **Amaç:** `CClientManager` sınıfının, oyuncuların oluşturduğu özel pazarların (genellikle premium veya çevrimdışı pazar olarak adlandırılır) verilerini yöneten fonksiyonlarını implemente eder. Bu, pazar bilgilerinin (sahip, isim, süre) ve içindeki eşyaların (Vnum, adet, fiyat, soketler, efsunlar) yüklenmesini, kaydedilmesini, önbelleğe alınmasını ve oyun sunucuları ile senkronize edilmesini içerir. Ayrıca pazar oluşturma, eşya ekleme/çıkarma/satın alma gibi işlemlerin DB tarafındaki mantığını yürütür. (`#if defined(__PREMIUM_PRIVATE_SHOP__)` veya benzeri bir direktif ile sarmalanmış olabilir.)
*   **Önbellek Yönetimi:**
    *   **Pazar Önbelleği (`m_map_privateShopCache`, `CPrivateShopCache`):** Özel pazarın ana bilgilerini (`TPrivateShop`) oyuncu ID'si bazında önbelleğe alır. `Put/Get/Delete/Flush/UpdatePrivateShopCache` fonksiyonları ile yönetilir.
    *   **Pazar Eşya Önbelleği (`m_map_pPrivateShopItemCacheSetPtr`, `m_map_privateShopItemCache`, `CPrivateShopItemCache`):** Özel pazarlardaki eşyaların (`TPlayerPrivateShopItem`) bilgilerini pazar sahibinin ID'si ve eşyanın benzersiz ID'si bazında önbelleğe alır. `Create/Get/Put/Delete/Flush/UpdatePrivateShopItemCacheSet`, `Get/Put/DeletePrivateShopItemCache` fonksiyonları ile yönetilir.
*   **Veritabanı Sonuç İşleme (`RESULT_*`):**
    *   `RESULT_PRIVATE_SHOP_LOAD`: `private_shop` tablosundan yüklenen pazar ana bilgilerini işler, önbelleğe alır (`PutPrivateShopCache`), oyun sunucusuna gönderir ve pazar nesnesini bellekte oluşturur (`PrivateShopSpawn`).
    *   `RESULT_PRIVATE_SHOP_ITEM_LOAD`: `private_shop_item` tablosundan yüklenen pazar eşyalarını işler, eşya önbelleğini doldurur (`CreatePrivateShopItemCacheSet`, `PutPrivateShopItemCache`) ve oyun sunucusuna gönderir.
*   **Bellek Yönetimi (PrivateShop Sınıfı):**
    *   `LPPRIVATE_SHOP`: `PrivateShop.h`/`.cpp` içinde tanımlanan `CPrivateShop` sınıfının bir işaretçisi. Bu sınıf, bir özel pazarın tüm bilgilerini (eşyalar, isim, süre vb.) bellekte tutar.
    *   `m_mapPrivateShop`: Oyuncu ID'sinden `LPPRIVATE_SHOP`'a eşleme yapan harita. Aktif/bellekteki özel pazarları tutar.
    *   `PrivateShopCreate()`, `PrivateShopSpawn()`, `PrivateShopGet()`, `PrivateShopDelete()`: Bellekteki `CPrivateShop` nesnelerini oluşturur, bulur ve siler.
    *   `PrivateShopBuild()`: Oyun sunucusundan gelen `PRIVATE_SHOP_GD_SUBHEADER_BUILD` isteği ile pazar nesnesini oluşturur ve veritabanına kaydeder.
    *   `PrivateShopClose()`: Pazarın kapatılma isteğini işler, nesneyi siler ve veritabanını günceller.
    *   `PrivateShopDestroy()`: Bir `CPrivateShop` nesnesini ve içindeki tüm eşyaları bellekten tamamen temizler.
    *   `PrivateShopFetchData()`: Bir pazarın tüm verilerini (ana bilgi ve eşyalar) veritabanından veya önbellekten toplar.
*   **Paket İşleme (`ProcessPrivateShopPacket`, Alt Fonksiyonlar):**
    *   Oyun sunucusundan gelen `HEADER_GD_PRIVATE_SHOP` paketini alır.
    *   Paketin alt başlığına (`PRIVATE_SHOP_GD_SUBHEADER_*`) göre ilgili işleyici fonksiyona yönlendirir:
        *   `BUILD`: Yeni pazar oluşturur.
        *   `CLOSE`: Pazarı kapatır.
        *   `DELETE`: Pazarı tamamen siler.
        *   `DESPAWN`: Pazarı oyundan kaldırır (ama verileri kalır).
        *   `WITHDRAW_REQUEST`: Sahibin para çekme isteğini işler.
        *   `ITEM_CHECKIN_UPDATE`/`ITEM_CHECKOUT_UPDATE`: Eşya ekleme/çıkarma işlemlerinin sonucunu işler.
        *   `MODIFY_REQUEST`: Pazar bilgilerini (süre, isim vb.) değiştirme isteğini işler.
        *   `BUY_REQUEST`: Eşya satın alma isteğini işler.
        *   `ITEM_PRICE_CHANGE_REQUEST`: Eşya fiyatı değiştirme isteğini işler.
        *   `ITEM_MOVE_REQUEST`: Eşyanın pazar içindeki yerini değiştirme isteğini işler.
        *   `ITEM_CHECKIN_REQUEST`: Eşya ekleme isteğini işler.
        *   `ITEM_CHECKOUT_REQUEST`: Eşya çıkarma isteğini işler.
        *   `TITLE_CHANGE_REQUEST`: Pazar başlığını değiştirme isteğini işler.
        *   `WARP_REQUEST`: Pazarı ışınlama isteğini işler.
        *   `BUY`: Gerçek satın alma işlemini yapar, eşyayı siler, parayı aktarır.
*   **Premium Etkinlik Yönetimi (`PrivateShopStart/End/IsPremiumEvent`):** Özel pazarla ilgili premium özelliklerin (süre uzatma vb.) aktif olup olmadığını yönetir.
*   **Oyun Sunucusu ile Etkileşim:**
    *   `PrivateShopGameSpawn()`/`PrivateShopGameDespawn()`: Pazarın oyun dünyasında görünmesi/kaybolması için ilgili oyun sunucusuna paket gönderir.
    *   `GetPrivateShopPeer()`: Belirli bir kanaldaki oyun sunucusu bağlantısını bulur.
*   **Önem:** Premium veya çevrimdışı özel pazar sisteminin tüm veritabanı ve durum yönetimi mantığını içerir. Oyuncuların pazar oluşturmasını, yönetmesini ve diğer oyuncuların bu pazarlarla etkileşime girmesini sağlar. Önbellekleme ile performansı artırır.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `Cache.h`, `HB.h`, `Main.h`, `QID.h`, `PrivateShop.h`, `PrivateShopUtils.h`, `libgame/grid.h`.

### `ClientManagerWeather.cpp`

*   **Amaç:** `CClientManager` sınıfının, oyun içi hava durumu ve mevsimsel olaylarla ilgili global olay bayraklarını (event flag) yöneten fonksiyonunu implemente eder.
*   **Implementasyon Detayları:**
    *   **`GetDayMode(int Hour)`:** Verilen saate göre gündüz (`DAY`) veya gece (`NIGHT`) modunu belirler (6-20 arası gündüz).
    *   **`IsWinter(int Month)`:** Verilen aya göre kış mevsimi olup olmadığını belirler (3. aydan küçükse kış).
    *   **`UpdateWeatherInfo()`:**
        *   `CClientManager::MainLoop()` içinde periyodik olarak çağrılır (bu kodda görünmese de genellikle böyledir).
        *   Sistemin mevcut saatini alır (`localtime`).
        *   Saate göre `GetDayMode()` ile gündüz/gece durumunu belirler ve `eclipse` olay bayrağını günceller (`SetEventFlag` çağrılır).
        *   Aya göre `IsWinter()` ile kış olup olmadığını belirler.
        *   Kış durumuna göre `xmas_snow`, `xmas_tree`, `snow_texture` olay bayraklarını günceller (`SetEventFlag` çağrılır).
        *   `SetEventFlag` çağrısı, bayrak değerinde bir değişiklik varsa bunu veritabanına kaydeder ve diğer oyun sunucularına iletir.
*   **Önem:** Oyun dünyasındaki gündüz/gece döngüsünü ve kış mevsimi gibi mevsimsel olayları (ve bunlara bağlı görsel değişiklikleri) tüm sunucularda tutarlı bir şekilde yönetir.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`.

---

**Not:** `db/src` klasöründeki diğer bileşenlerin belgelemesi için lütfen `db2_Referans.md` dosyasına bakınız.
