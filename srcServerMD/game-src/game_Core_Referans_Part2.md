# Metin2 Oyun Sunucusu - Çekirdek Mekanikleri Referansı Part 2 (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) çekirdek mekanikleriyle ilgili dosyaların belgelenmesine devam eder.

## İçindekiler

*   [`log.h`](#logh)
*   [`log.cpp`](#logcpp)
*   [`main.cpp`](#maincpp)
*   [`malloc_allocator.h`](#malloc_allocatorh)
*   [`map_location.h`](#map_locationh)
*   [`map_location.cpp`](#map_locationcpp)
*   [`mob_manager.h`](#mob_managerh)
*   [`mob_manager.cpp`](#mob_managercpp)
*   [`motion.h`](#motionh)
*   [`motion.cpp`](#motioncpp)
*   [`p2p.h`](#p2ph)
*   [`p2p.cpp`](#p2pcpp)

---

### `log.h`

*   **Amaç:** Oyun sunucusundaki çeşitli önemli olayları (eşya hareketleri, karakter eylemleri, giriş/çıkışlar, para işlemleri, hack girişimleri, küp kullanımı, GM komutları vb.) bir veritabanına kaydetmek için `LogManager` singleton sınıfını tanımlar. Farklı log türleri için özel metot bildirimleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`GOLDBAR_HOW` Enum'u:** Altın külçesi (veya benzeri değerli eşya) loglarında işlemin nasıl yapıldığını belirtmek için kullanılır (örn. `PERSONAL_SHOP_BUY`, `SHOP_SELL`, `EXCHANGE_TAKE`, `QUEST`).
    *   **`LogManager` Sınıfı (Singleton):**
        *   `Connect`: Log veritabanına bağlantı kurar.
        *   `IsConnected`: Veritabanı bağlantısının durumunu kontrol eder.
        *   Çeşitli Log Metotları (Örnekler):
            *   `ItemLog`: Eşya ile ilgili olayları (alma, düşürme, kullanma vb.) loglar.
            *   `CharLog`: Karakterle ilgili genel olayları loglar.
            *   `LoginLog`: Oyuncu giriş/çıkışlarını loglar.
            *   `MoneyLog`: Para (yang/won) değişikliklerini loglar.
            *   `HackLog`, `HackCRCLog`, `SpeedHackLog`: Hile girişimlerini loglar.
            *   `GoldBarLog`: Altın külçesi hareketlerini loglar.
            *   `CubeLog`: Küp sistemi kullanımını loglar.
            *   `GMCommandLog`: GM'ler tarafından kullanılan komutları loglar.
            *   `WhisperLog`: Fısıltı mesajlarını loglar.
            *   `ChangeNameLog`: İsim değişikliği işlemlerini loglar.
            *   `RefineLog`: Eşya geliştirme (basma) işlemlerini loglar.
            *   `ShoutLog`: Bağırma mesajlarını loglar.
            *   `LevelLog`: Seviye atlama olaylarını loglar.
            *   `BootLog`: Sunucunun başlatılma bilgilerini loglar.
            *   `FishLog`: Balık tutma olaylarını loglar.
            *   `QuestRewardLog`: Görev ödüllerini loglar.
            *   `DetailLoginLog`: Daha detaylı giriş/çıkış bilgilerini loglar.
            *   `DragonSlayLog`: Ejderha öldürme olaylarını loglar.
            *   `MeleyLog` (`__GUILD_DRAGONLAIR__`): Meley Zindanı logları.
            *   `MailLog` (`__MAILBOX__`): Posta kutusu işlemlerini loglar.
            *   `AcceLog` (`__ACCE_COSTUME_SYSTEM__`): Kostüm aksesuarı (kuşak) emdirme logları.
        *   `Query` (private): Asenkron SQL sorgusu göndermek için kullanılan özel bir metot.
    *   **Üyeler:**
        *   `m_sql` (`CAsyncSQL`): Veritabanı işlemleri için asenkron SQL nesnesi.
        *   `m_bIsConnect` (bool): Veritabanı bağlantı durumu.
*   **Bağlantılı Dosyalar:** `log.cpp` (uygulama), `../../libsql/AsyncSQL.h`, `any_function.h`, `singleton.h`.

### `log.cpp`

*   **Amaç:** `log.h`'de bildirilen `LogManager` sınıfının metotlarını uygular. Veritabanı bağlantısını yönetir ve çeşitli oyun içi olaylar için log kayıtlarını oluşturan SQL sorgularını formatlayıp asenkron olarak veritabanına gönderir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Bağlantı Yönetimi:**
        *   `Connect`: Verilen host, port, kullanıcı, şifre ve veritabanı bilgileriyle `m_sql.Setup()` kullanarak veritabanına bağlanmaya çalışır. Bağlantı durumunu `m_bIsConnect`'te saklar.
        *   `IsConnected`: `m_bIsConnect` değerini döndürür.
    *   **Sorgu Gönderme:**
        *   `Query(const char* c_pszFormat, ...)`: `va_list` kullanarak değişken argümanlı formatlanmış bir SQL sorgusu oluşturur. Test sunucusunda (`test_server` true ise) sorguyu `sys_log` ile de loglar. Oluşturulan sorguyu `m_sql.AsyncQuery()` ile veritabanına asenkron olarak gönderir.
    *   **Log Metotlarının Uygulanması:**
        *   Her bir public log metodu (örn. `ItemLog`, `CharLog`, `LoginLog` vb.) `Query` metodunu çağırarak özel bir `INSERT DELAYED INTO` veya `INSERT INTO` SQL sorgusu oluşturur.
        *   Sorgular genellikle logun türünü, zamanını (`NOW()` veya `CURDATE()`, `CURTIME()`), olayı gerçekleştiren karakterin ID'si (`who`, `pid`), koordinatları (`x`, `y`), ilgili eşya/değer (`what`), işlemin detayı (`how`), ek açıklama (`hint`), IP adresi ve bazen VNUM gibi bilgileri içerir.
        *   `c_pszHint`, `szCommand`, `item_name`, `pszText` (ShoutLog için), `message` (WhisperLog için) gibi metin tabanlı girdiler, SQL enjeksiyonunu önlemek için `m_sql.EscapeString()` ile işlenir ve sonucu `__escape_hint` statik buffer'ına yazılır.
        *   Tablo isimleri genellikle `get_table_postfix()` fonksiyonundan dönen bir sonek ile (örn. `log%s`, `loginlog%s`) dinamik olarak oluşturulur. Bu, muhtemelen sunucu veya kanal bazında ayrı log tabloları tutulmasını sağlar.
        *   Bazı loglar (`HackLog`, `HackCRCLog`, `BootLog`, `CommandLog` GM için) sunucu host adını (`g_stHostname`) veya kanal bilgisini (`g_bChannel`) da kaydeder.
        *   `LevelLog`: `LC_IsEurope()` kontrolüne göre farklı formatta log atar (Avrupa için `account_id` ve `pid` ekler).
        *   `DetailLoginLog`: Girişte `INVALID` tipinde bir log atar, çıkışta bu logu `VALID` olarak günceller ve oynama süresini (`TIMEDIFF`) hesaplar.
        *   `MeleyLog` (`__GUILD_DRAGONLAIR__`): Önce `SELECT` ile mevcut kaydı kontrol eder, duruma göre `INSERT` veya `UPDATE` yapar.
    *   **Statik Buffer:**
        *   `__escape_hint[1024]`: `EscapeString` için kullanılan statik bir karakter dizisi. Bu, teorik olarak çoklu thread ortamında sorun yaratabilir, ancak `AsyncSQL`'in işleyişine bağlıdır.
*   **Çalışma Prensibi:** `LogManager` örneği oluşturulduğunda veritabanı bağlantısı kurulur. Oyun içinde loglanması gereken bir olay meydana geldiğinde, ilgili `LogManager` metodu çağrılır. Bu metot, olaya özgü verileri alarak bir SQL `INSERT` sorgusu oluşturur, metin verilerini güvenli hale getirir ve sorguyu asenkron olarak veritabanına gönderir. Loglar, `log`, `loginlog`, `money_log` gibi farklı tablolara veya aynı tablonun farklı bölümlerine (postfix ile) yazılır.
*   **Bağlantılı Dosyalar:** `log.h`, `stdafx.h`, `constants.h`, `config.h`, `char.h`, `desc.h`, `item.h`, `locale_service.h`. (`__GUILD_DRAGONLAIR__` tanımlıysa `db.h`).

### `main.cpp`

*   **Amaç:** Oyun sunucusunun ana giriş noktasıdır (`main` fonksiyonunu içerir). Sunucunun başlatılması, temel bileşenlerin (çeşitli yönetici (manager) sınıfları) ilklendirilmesi, ana döngünün (olay işleme, ağ G/Ç, karakter güncellemeleri) çalıştırılması ve sunucunun düzgün bir şekilde kapatılması işlemlerini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Başlangıç (`main` fonksiyonu ve `start` fonksiyonu):**
        *   Komut satırı argümanlarının işlenmesi (port, log seviyesi, yerelleştirme servis adı vb. `start` fonksiyonu içinde `getopt` ile - artık kullanılmıyor gibi görünüyor, doğrudan `config_init` çağrılıyor).
        *   `WriteVersion`: Sunucu versiyon bilgisini yazar.
        *   Çok sayıda yönetici sınıfının (örn: `SECTREE_MANAGER`, `CHARACTER_MANAGER`, `ITEM_MANAGER`, `CShopManager`, `DBManager`, `LogManager`, `P2P_MANAGER`, `CGuildManager`, `DESC_MANAGER`, `quest::CQuestManager`, `DSManager` vb.) global nesneleri oluşturulur ve kurucu fonksiyonları çağrılır.
        *   `thecore_init`: `thecore` kütüphanesini (ana sunucu döngüsü, olay yönetimi, zamanlayıcılar için) başlatır ve `heartbeat` fonksiyonunu periyodik çalışacak şekilde ayarlar.
        *   Ağ soketleri (TCP için `mother_port`, P2P için `p2p_port`) oluşturulur (`socket_tcp_bind`), `fdwatch`'a (`main_fdw`) okunabilir olaylar için eklenir. UDP soketi (`udp_socket`) `#if !defined(__UDP_BLOCK__)` ile koşullu olarak oluşturulur.
        *   Veritabanı sunucusuna bağlantı (`db_clientdesc`) `DESC_MANAGER` aracılığıyla kurulur.
        *   Eğer sunucu bir Auth sunucusu ise ve master/slave konfigürasyonu varsa, master auth sunucusuna P2P bağlantısı kurulur.
        *   `signal_timer_enable`: Sinyal tabanlı zamanlayıcıları etkinleştirir.
        *   Çeşitli oyun sistemleri ilklendirilir: `MessengerManager`, `CGuildManager`, `fishing`, `COXEventManager`, `CSpeedServerManager` (eğer `speed_server` aktifse), Küp sistemi (`Cube_init` veya `CCubeManager::Instance().Cube_init()`), `Blend_Item_init`, `ani_init`, `PanamaLoad`.
        *   `Metin2Server_Check` (opsiyonel): Sunucu geçerliliğini kontrol eder.
        *   Eğer oyun sunucusu ise (`!g_bAuthServer`), Meley ve Ochao zindanları gibi özel sistemler ilklendirilir.
        *   `DESC_MANAGER::LoadClientPackageCryptInfo`: İstemci paket şifreleme bilgilerini (`package/` dizininden) yükler.
    *   **Ana Döngü (`idle` fonksiyonu):**
        *   `thecore_idle()`: İşlenecek olay olup olmadığını veya sunucunun kapanması gerekip gerekmediğini kontrol eder. Geçen "pulse" sayısını döndürür.
        *   `heartbeat`: Her "pulse" için çağrılır.
        *   `CHARACTER_MANAGER::instance().Update()`: Karakterlerle ilgili güncellemeleri yapar.
        *   `db_clientdesc->Update()`: Veritabanı bağlantısıyla ilgili bekleyen işlemleri günceller.
        *   `io_loop(main_fdw)`: Ağ giriş/çıkış işlemlerini yönetir.
        *   `log_rotate()`: Log dosyalarını yönetir (döndürme vb.).
        *   Periyodik olarak (saniyede bir) performans istatistiklerini (`pt_log`) loglar.
    *   **Kalp Atışı (`heartbeat` fonksiyonu):**
        *   `event_process(pulse)`: `thecore` tarafından yönetilen zamanlanmış olayları işler.
        *   Periyodik İşlemler (saniyede bir veya daha seyrek):
            *   Auth sunucusu ise Brezilya için özel loglama, süresi dolmuş login key'lerinin işlenmesi.
            *   Oyun sunucusu ise veritabanına oyuncu sayısını bildirme.
            *   `login_sim` (giriş simülasyonu) ile ilgili bekleyen işlemlerin gönderilmesi.
            *   `g_vec_save` (kaydedilecek oyuncu verileri) kuyruğundan verilerin DB'ye gönderilmesi.
            *   `CHARACTER_MANAGER::instance().ProcessDelayedSave()`: Gecikmeli karakter kayıt işlemlerini yapar.
            *   `FileMonitorFreeBSD::Instance().Update()`: Dosya değişikliklerini izler (FreeBSD için).
            *   `ITEM_MANAGER::instance().Update()`: Eşya güncellemelerini yapar.
            *   `DESC_MANAGER::instance().UpdateLocalUserCount()`: Yerel oyuncu sayısını günceller.
        *   `DBManager::instance().Process()`, `AccountDB::instance().Process()`, `CPVPManager::instance().Process()`: İlgili yöneticilerin bekleyen işlemlerini yapar.
        *   Kapatma Süreci (`g_bShutdown` aktif ise): Belirli zaman aralıklarıyla oyunculara bağlantı kesme bildirimi gönderir, bağlantıları zorla keser ve `thecore_shutdown()` ile sunucuyu kapatır.
    *   **Ağ G/Ç Döngüsü (`io_loop` fonksiyonu):**
        *   `DESC_MANAGER::instance().DestroyClosed()`: Kapatılmış (`PHASE_CLOSE`) bağlantıları temizler.
        *   `DESC_MANAGER::instance().TryConnect()`: Bekleyen bağlantı isteklerini işler.
        *   `fdwatch(fdw, 0)`: Soketlerdeki olayları bekler.
        *   Yeni bağlantıları kabul eder: `DESC_MANAGER::instance().AcceptDesc()` (istemci için), `DESC_MANAGER::instance().AcceptP2PDesc()` (P2P için).
        *   Mevcut bağlantılardan gelen verileri okur ve işler (`d->ProcessInput()`).
        *   Mevcut bağlantılara veri yazar (`d->ProcessOutput()`).
        *   Bağlantı hatalarında veya kopmalarında descriptor'ı `PHASE_CLOSE` durumuna getirir.
    *   **Kapatma İşlemleri (`destroy` fonksiyonu ve `main` içindeki kapatma bloğu):**
        *   `g_bShutdown` `true` yapılır, yeni istemci kabulü durdurulur (`g_bNoMoreClient`).
        *   Auth sunucusu ise DB sorgu kuyruğunun boşalması beklenir.
        *   Arena ve OXEvent yöneticileri sonlandırılır (`Destroy`).
        *   Sinyal zamanlayıcısı devre dışı bırakılır.
        *   Premium özel pazar gibi sistemler sonlandırılır.
        *   `CHARACTER_MANAGER` ve `ITEM_MANAGER` düzgün kapatma (`GracefulShutdown`) işlemlerini yapar.
        *   DB ve P2P bağlantılarındaki çıktı tamponları boşaltılır (`FlushOutput`).
        *   Diğer tüm yönetici sınıflarının `Destroy` metotları çağrılarak kaynakları serbest bırakılır.
        *   `regen_free`, soketlerin kapatılması, `fdwatch_delete`, `event_destroy`, `CTextFileLoader::DestroySystem`, `thecore_destroy` çağrıları ile tüm sistemler sonlandırılır.
    *   **Hata Yönetimi:**
        *   `ShutdownOnFatalError()`: Kritik bir hata oluştuğunda çağrılır. Oyunculara bildirim gönderir ve sunucunun belirli bir süre sonra kapanmasını zamanlar.
        *   `ContinueOnFatalError()`: Kritik hata loglanır, stack trace (eğer `USE_STACKTRACE` tanımlıysa) alınır.
    *   **Sunucu Geçerlilik Kontrolü (`Metin2Server_Check`, `Metin2Server_IsInvalid`):**
        *   `#ifdef _USE_SERVER_KEY_` ile koşullu derlenir.
        *   `g_szPublicIP` adresini `CheckServer::CheckIp` ile veya harici bir sunucuya bağlanarak kontrol eder. Başarısız olursa `g_isInvalidServer` `true` yapılır.
*   **Orta Seviye Implementasyon Detayları:**
    *   Sunucu, çok sayıda singleton deseniyle tasarlanmış yönetici sınıfı üzerinden modüler bir yapıda çalışır.
    *   Ana işlem döngüsü ve olay yönetimi `thecore` adlı bir kütüphane tarafından sağlanır. Bu kütüphane, periyodik görevler (heartbeat), zamanlanmış olaylar (event) ve asenkron G/Ç (fdwatch) için temel altyapıyı sunar.
    *   Ağ iletişimi `DESC` (descriptor) nesneleri üzerinden yönetilir. Her bağlantı bir `DESC` nesnesiyle temsil edilir ve `DESC_MANAGER` tarafından yönetilir.
    *   Koşullu derleme direktifleri (`#include "..."` öncesi `#if defined(...)` veya dosya içinde `#ifdef ... #endif`) ile sunucuya farklı özellik setleri dahil edilebilir veya çıkarılabilir (örn. `__FILEMONITOR__`, `__GUILD_DRAGONLAIR__`, `__CUBE_RENEWAL__`, `__PREMIUM_PRIVATE_SHOP__`, `__UDP_BLOCK__`).
    *   Global değişkenler (örn: `g_bShutdown`, `g_bAuthServer`, `mother_port`, `p2p_port`, `main_fdw`, `tcp_socket`, `udp_socket`, `p2p_socket`, `s_dwProfiler`, `g_isInvalidServer`) sunucunun genel durumunu, yapılandırmasını ve çalışma zamanı bilgilerini tutmak için kullanılır.
*   **Bağlantılı Dosyalar:** `stdafx.h` (ön derlenmiş başlık), `constants.h` (sabitler), `config.h` (yapılandırma), `event.h` (olay sistemi), `packet.h` (paket tanımları) ve `main.cpp` dosyasının başında `#include` edilen diğer tüm yönetici sınıflarının başlık dosyaları (örn: `desc_manager.h`, `item_manager.h`, `char_manager.h`, `questmanager.h`, `guild_manager.h` vb.). Ayrıca, `minilzo.h` (sıkıştırma), `locale_service.h` (yerelleştirme) gibi yardımcı kütüphaneler ve özellik başlıkları da dahil edilir. Boost kütüphanesinden `boost/bind.hpp` gibi bileşenler de kullanılmaktadır.

### `malloc_allocator.h`

*   **Amaç:** Standart C Kütüphanesi (CRT) fonksiyonları olan `::malloc` ve `::free` kullanarak temel bir bellek ayırıcı (allocator) sınıfı olan `MallocAllocator`'ı tanımlar. Bu sınıf, özel bellek yönetimi stratejileri gerektirmeyen durumlar için basit bir arayüz sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`MallocAllocator` Sınıfı:**
        *   **Yapıcı/Yıkıcı (`MallocAllocator()`, `~MallocAllocator()`):** Boş yapıcı ve yıkıcı metotlar.
        *   **`SetUp()`, `TearDown()`:** Boş metotlar, muhtemelen daha karmaşık ayırıcılar için bir arayüz standardı sağlamak amacıyla eklenmiştir.
        *   **`Alloc(size_t size)`:** Belirtilen boyutta (`size`) bir bellek bloğu ayırmak için global `::malloc` fonksiyonunu çağırır ve ayrılan bloğun işaretçisini döndürür.
        *   **`Free(void* p)`:** Daha önce `Alloc` ile ayrılmış olan bir bellek bloğunu (`p`) serbest bırakmak için global `::free` fonksiyonunu çağırır.
*   **Kullanım Senaryosu:** Bu sınıf,STL konteynerları veya özel veri yapıları için özel bir bellek ayırıcı gerektiğinde, ancak karmaşık bir strateji yerine sadece standart `malloc`/`free` işlevlerinin kullanılması yeterli olduğunda bir sarmalayıcı olarak kullanılabilir.
*   **Bağlantılı Dosyalar:** Bu dosya genellikle başka bir sistem tarafından (örneğin, özel bir `new`/`delete` operatörü veya bir konteyner sınıfı) dahil edilir. Linter hataları, `stdlib.h` veya benzeri bir başlığın `malloc` ve `free` için dahil edilmediğini gösteriyor, ancak bu genellikle `stdafx.h` veya projenin genel başlıkları aracılığıyla sağlanır.

### `map_location.h`

*   **Amaç:** Farklı oyun haritalarının veya bölgelerinin hangi sunucu adreslerinde (IP ve port) barındırıldığını takip etmek için kullanılan `CMapLocation` singleton sınıfını tanımlar. Bu, özellikle çoklu sunucu (multi-server) mimarilerinde veya haritaların farklı sunuculara dağıtıldığı durumlarda önemlidir.
*   **Temel İşlevler/İçerik:**
    *   **`SLocation` (typedef `TLocation`) Struct:**
        *   `addr` (long): Sunucunun IP adresini (genellikle `inet_addr` ile dönüştürülmüş `unsigned long` formatında) saklar.
        *   `port` (WORD): Sunucunun port numarasını saklar.
    *   **`CMapLocation` Sınıfı (Singleton):**
        *   **Metot Bildirimleri:**
            *   `Get(long x, long y, long& lMapIndex, long& lAddr, WORD& wPort)`: Verilen dünya koordinatları (x, y) için ilgili harita indeksini (`SECTREE_MANAGER` kullanarak) bulur ve bu harita indeksine karşılık gelen sunucu adresini (IP ve port) döndürür.
            *   `Get(int iIndex, long& lAddr, WORD& wPort)`: Doğrudan bir harita indeksi (`iIndex`) için sunucu adresini (IP ve port) döndürür.
            *   `Insert(long lIndex, const char* c_pszHost, WORD wPort)`: Belirli bir harita indeksi (`lIndex`) için sunucu host adını (`c_pszHost`) ve portunu (`wPort`) kaydeder. Host adı `inet_addr` ile sayısal IP adresine dönüştürülür.
        *   **Korunan (Protected) Üye:**
            *   `m_map_address` (`std::map<long, TLocation>`): Harita indekslerini (`long`) ilgili sunucu konum bilgilerine (`TLocation`) eşleyen bir harita.
*   **Bağlantılı Dosyalar:** `map_location.cpp` (uygulama), `../../common/stl.h` (muhtemelen `std::map` ve `singleton` şablonu için), `singleton.h` (doğrudan veya dolaylı olarak).

### `map_location.cpp`

*   **Amaç:** `map_location.h` dosyasında bildirilen `CMapLocation` sınıfının metotlarını uygular. Harita indeksleri ile sunucu adresleri (IP ve port) arasındaki eşlemeyi yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Değişken:**
        *   `CMapLocation g_mapLocations;`: `CMapLocation` sınıfının global singleton örneği.
    *   **`CMapLocation::Get(long x, long y, long& lIndex, long& lAddr, WORD& wPort)`:**
        *   Verilen dünya koordinatlarını (x, y) kullanarak `SECTREE_MANAGER::instance().GetMapIndex(x, y)` ile harita indeksini (`lIndex`) alır.
        *   Ardından, bu harita indeksini kullanarak diğer `Get` metodunu (`Get(lIndex, lAddr, wPort)`) çağırır ve sonucu döndürür.
    *   **`CMapLocation::Get(int iIndex, long& lAddr, WORD& wPort)`:**
        *   Verilen harita indeksi (`iIndex`) için `m_map_address` haritasında bir arama yapar.
        *   Eğer harita indeksi 0 ise veya haritada bulunamazsa, bir hata mesajı loglanır (`sys_log`) ve `false` döndürülür. Hata durumunda, mevcut tüm harita-sunucu eşlemelerini de loglar.
        *   Harita indeksi bulunursa, ilgili `TLocation` yapısından `addr` (IP adresi) ve `port` (port numarası) değerleri çıktı parametrelerine atanır ve `true` döndürülür.
    *   **`CMapLocation::Insert(long lIndex, const char* c_pszHost, WORD wPort)`:**
        *   Yeni bir `TLocation` nesnesi oluşturur.
        *   Verilen host adını (`c_pszHost`) `inet_addr()` fonksiyonu ile `unsigned long` formatındaki bir IP adresine dönüştürür ve `loc.addr`'a atar.
        *   Verilen port numarasını `loc.port`'a atar.
        *   Oluşturulan `TLocation` nesnesini, verilen harita indeksi (`lIndex`) ile birlikte `m_map_address` haritasına ekler (`std::make_pair` kullanarak).
        *   Ekleme işlemini `sys_log` ile loglar.
*   **Çalışma Prensibi:** Sunucu başlatılırken veya yapılandırma dosyaları okunurken, `Insert` metodu çağrılarak hangi harita indeksinin hangi sunucu IP'si ve portunda olduğu bilgisi `m_map_address` haritasına yüklenir. Oyun sırasında, bir karakterin başka bir haritaya geçmesi gerektiğinde veya farklı bir harita/bölge ile etkileşim gerektiğinde, `Get` metotları kullanılarak o haritanın barındırıldığı sunucunun adresi öğrenilir. Bu bilgi, P2P iletişimi veya istemcinin doğru sunucuya yönlendirilmesi için kullanılabilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `map_location.h`, `sectree_manager.h`. `inet_addr` fonksiyonu için genellikle bir ağ başlık dosyası (örn. `arpa/inet.h` veya `winsock2.h` `stdafx.h` üzerinden) gereklidir.

### `mob_manager.h`

*   **Amaç:** Yaratık (mob) ve NPC prototiplerini, örneklerini ve gruplarını yönetmek için kullanılan sınıfları ve yapıları tanımlar. `CMob` sınıfı, bir yaratık türünün temel özelliklerini (örneğin, `TMobTable` üzerinden) ve yeteneklerini tutar. `CMobInstance` sınıfı, bir yaratığın oyundaki belirli bir örneğine ait durum bilgilerini (örneğin, son saldırı zamanı, özel durumlar) saklar. `CMobGroup` ve `CMobGroupGroup` sınıfları, yaratıkların gruplar halinde tanımlanmasını ve yönetilmesini sağlar. `CMobManager` singleton sınıfı ise tüm bu yaratık prototiplerini, gruplarını yükler, saklar ve erişimini sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`SMobSplashAttackInfo` (TMobSplashAttackInfo) Struct:** Bir yaratık yeteneğinin alan etkili (splash) saldırısının zamanlamasını (`dwTiming`) ve etki mesafesini (`dwHitDistance`) tanımlar.
    *   **`SMobSkillInfo` (TMobSkillInfo) Struct:** Bir yaratık yeteneğinin VNUM'ını (`dwSkillVnum`), seviyesini (`bSkillLevel`) ve bu yeteneğe bağlı alan etkili saldırı bilgilerini (`vecSplashAttack`) tutar.
    *   **`CMob` Sınıfı:**
        *   Tek bir yaratık türünün (prototipinin) bilgilerini içerir.
        *   `m_table` (`TMobTable`): Yaratığın tüm temel özelliklerini (isim, seviye, HP, saldırı gücü, savunma, düşüreceği eşyalar, yetenekler vb.) içeren ana tablo.
        *   `m_mobSkillInfo[MOB_SKILL_MAX_NUM]` (`TMobSkillInfo` dizisi): Yaratığın sahip olabileceği maksimum sayıda yeteneğin detaylı bilgisini (alan etkili saldırıları dahil) saklar.
        *   `AddSkillSplash()`: Belirli bir yeteneğe alan etkili saldırı bilgisi ekler.
        *   `GetLocaleName()`: Yaratığın yerelleştirilmiş ismini döndürür.
    *   **`CMobInstance` Sınıfı:**
        *   Oyun dünyasında var olan bir yaratık örneğinin (instance) ek durum bilgilerini tutar.
        *   `m_posLastAttacked` (`PIXEL_POSITION`): Yaratığın en son saldırıya uğradığı pozisyon.
        *   `m_dwLastAttackedTime` (DWORD): Yaratığın en son saldırıya uğradığı zaman.
        *   `m_dwLastWarpTime` (DWORD): Yaratığın en son ışınlandığı zaman.
        *   `m_IsBerserk`, `m_IsGodSpeed`, `m_IsRevive` (bool): Yaratığın Berserk, GodSpeed veya Revive durumlarında olup olmadığını belirten bayraklar.
    *   **`CMobGroupGroup` Sınıfı:**
        *   Bir "grup grubunu" temsil eder. Bu grup, birden fazla `CMobGroup`'u içerebilir ve bu gruplardan birini olasılığa dayalı olarak seçmek için kullanılır.
        *   `m_dwVnum` (DWORD): Grup grubunun VNUM'ı.
        *   `m_vec_dwMemberVnum` (std::vector<DWORD>): Bu grup grubuna dahil olan `CMobGroup` VNUM'larını tutar.
        *   `m_vec_iProbs` (std::vector<int>): (`ADD_MOB_GROUP_GROUP_PROB` ile) Üye grupların seçilme olasılıklarını kümülatif olarak tutar.
        *   `AddMember()`: Grup grubuna olasılıkla birlikte bir üye grup ekler.
        *   `GetMember()`: Olasılıklara göre grup grubundan bir üye `CMobGroup` VNUM'ı seçer ve döndürür.
    *   **`CMobGroup` Sınıfı:**
        *   Bir yaratık grubunu temsil eder. Bu grup, birden fazla yaratık VNUM'ını içerebilir.
        *   `m_dwVnum` (DWORD): Grubun VNUM'ı.
        *   `m_stName` (std::string): Grubun adı.
        *   `m_vec_dwMemberVnum` (std::vector<DWORD>): Bu gruba dahil olan yaratık VNUM'larını tutar.
        *   `Create()`: Grubu VNUM ve isim ile oluşturur.
        *   `GetMemberVector()`: Üye yaratık VNUM'larının vektörünü döndürür.
        *   `GetMemberCount()`: Gruptaki üye sayısını döndürür.
        *   `AddMember()`: Gruba bir üye yaratık VNUM'ı ekler.
    *   **`CMobManager` Sınıfı (Singleton):**
        *   Tüm yaratık prototiplerini ve gruplarını yönetir.
        *   **Typedef:** `iterator` (std::map<DWORD, CMob*>::iterator).
        *   `Initialize()`: `TMobTable` dizisinden yaratık prototiplerini yükler, `group.txt` ve `group_group.txt` dosyalarından yaratık gruplarını yükler.
        *   `Destroy()`: Yüklenmiş tüm verileri temizler (henüz implemente edilmemiş).
        *   `LoadGroup()`: `group.txt` dosyasından `CMobGroup`'ları yükler.
        *   `LoadGroupGroup()`: `group_group.txt` dosyasından `CMobGroupGroup`'ları yükler.
        *   `GetGroup()`: Verilen VNUM'a sahip `CMobGroup` nesnesini döndürür.
        *   `GetGroupFromGroupGroup()`: Verilen `CMobGroupGroup` VNUM'ından rastgele bir `CMobGroup` VNUM'ı döndürür.
        *   `Get(DWORD dwVnum)`: VNUM ile yaratık prototipini (`CMob`) döndürür.
        *   `Get(const char* c_pszName, bool bIsAbbrev)`: İsimle (tam veya kısaltılmış) yaratık prototipini döndürür.
        *   `begin()`, `end()`: Yaratık prototipi haritası için iterator'lar döndürür.
        *   `RebindMobProto()`: Bir karakterin (NPC/yaratık) prototipini yeniden bağlar.
        *   `IncRegenCount()`: Belirli bir VNUM için yeniden doğma (regen) sayacını artırır (istatistiksel amaçlı).
        *   `DumpRegenCount()`: Yeniden doğma sayaçlarını bir dosyaya yazar.
        *   **Özel (Private) Üyeler:**
            *   `m_map_pkMobByVnum` (std::map<DWORD, CMob*>): VNUM -> `CMob` prototipi eşleşmelerini tutar.
            *   `m_map_pkMobByName` (std::map<std::string, CMob*>): Yaratık Adı -> `CMob` prototipi eşleşmelerini tutar.
            *   `m_map_pkMobGroup` (std::map<DWORD, CMobGroup*>): Grup VNUM -> `CMobGroup` eşleşmelerini tutar.
            *   `m_map_pkMobGroupGroup` (std::map<DWORD, CMobGroupGroup*>): GrupGrup VNUM -> `CMobGroupGroup` eşleşmelerini tutar.
            *   `m_mapRegenCount` (std::map<DWORD, double>): Yeniden doğma sayaçlarını tutar.
*   **Bağlantılı Dosyalar:** `mob_manager.cpp` (uygulama), `stdafx.h`, `../../common/tables.h` (`TMobTable` tanımı).

### `mob_manager.cpp`

*   **Amaç:** `mob_manager.h` dosyasında bildirilen `CMob`, `CMobInstance`, `CMobGroup`, `CMobGroupGroup` ve `CMobManager` sınıflarının metotlarını uygular. Yaratık prototiplerinin ve gruplarının yüklenmesi, saklanması, erişimi ve bazı yardımcı işlevleri içerir.
*   **Temel İşlevleri/İçeriği:**
    *   **`CMob` Sınıfı Metotları:**
        *   Yapıcı (`CMob()`): `m_table`'ı sıfırlar, `m_mobSkillInfo` dizisini temizler.
        *   `AddSkillSplash()`: Belirli bir yetenek indeksine alan etkili saldırı bilgisi (zamanlama, mesafe) ekler ve loglar.
    *   **`CMobInstance` Sınıfı Metotları:**
        *   Yapıcı (`CMobInstance()`): Özel durum bayraklarını (`m_IsBerserk` vb.) false yapar, son saldırı ve ışınlanma zamanlarını mevcut zamana ayarlar.
    *   **`CMobManager` Sınıfı Metotları:**
        *   **`Initialize(TMobTable* pTable, int iSize)`:**
            *   Verilen `TMobTable` dizisinden tüm yaratık prototiplerini yükler. Her prototip için bir `CMob` nesnesi oluşturur, verileri kopyalar ve `m_map_pkMobByVnum` ile `m_map_pkMobByName` haritalarına ekler.
            *   Yaratık bilgilerini (VNUM, isim, seviye, HP, DEF, EXP, düşüreceği eşya, yetenek sayısı) loglar.
            *   Eğer yaratık bir NPC, Warp veya Goto karakter türündeyse, `CHARACTER_MANAGER::RegisterRaceNum()` ile VNUM'ını kaydeder.
            *   `CQuestManager::RegisterNPCVnum()` ile VNUM'ını görev sistemi için kaydeder.
            *   `LocaleService_GetBasePath()` kullanarak `group.txt` ve `group_group.txt` dosyalarının yollarını belirler ve `LoadGroup()` ile `LoadGroupGroup()` fonksiyonlarını çağırarak yaratık gruplarını yükler. Yükleme başarısız olursa sunucuyu kapatır (`thecore_shutdown()`).
            *   `CHARACTER_MANAGER::for_each_pc()` (aslında tüm karakterler üzerinde dönmeli, isim yanıltıcı) ile `RebindMobProto()` çağırarak mevcut tüm yaratıkların prototiplerini günceller/yeniden bağlar.
        *   **`RebindMobProto(LPCHARACTER ch)`:** Eğer karakter bir oyuncu değilse (`IsPC()` false ise), `Get(ch->GetRaceNum())` ile prototipini alır ve `ch->SetProto()` ile karaktere atar.
        *   **`Get(DWORD dwVnum)`:** `m_map_pkMobByVnum` haritasından VNUM'a göre `CMob` prototipini döndürür.
        *   **`Get(const char* c_pszName, bool bIsAbbrev)`:** `m_map_pkMobByName` haritasından yaratık ismine göre (tam eşleşme veya `bIsAbbrev` true ise kısmi ön ek eşleşmesi) `CMob` prototipini döndürür.
        *   **`IncRegenCount(BYTE bRegenType, DWORD dwVnum, int iCount, int iTime)`:** Yeniden doğma türüne (`REGEN_TYPE_MOB`, `REGEN_TYPE_GROUP`, `REGEN_TYPE_GROUP_GROUP`) göre ilgili yaratıkların veya grup üyelerinin `m_mapRegenCount` haritasındaki günlük yeniden doğma oranlarını (yaklaşık olarak) hesaplar ve artırır. Bu fonksiyon, haritalardaki `regen.txt` dosyaları işlenirken çağrılır ve sunucudaki toplam yaratık yoğunluğu hakkında bir fikir verir.
        *   **`DumpRegenCount(const char* c_szFilename)`:** `m_mapRegenCount` haritasındaki VNUM ve hesaplanmış yeniden doğma sayısı verilerini belirtilen dosyaya yazar.
        *   **`GetGroupFromGroupGroup(DWORD dwVnum)`:** `m_map_pkMobGroupGroup`'dan belirtilen VNUM'a sahip grup grubunu bulur ve bu grubun `GetMember()` metodunu çağırarak olasılığa göre bir üye `CMobGroup` VNUM'ı döndürür.
        *   **`GetGroup(DWORD dwVnum)`:** `m_map_pkMobGroup` haritasından VNUM'a göre `CMobGroup` nesnesini döndürür.
        *   **`LoadGroupGroup(const char* c_pszFileName)`:**
            *   `CTextFileLoader` kullanarak belirtilen dosyayı (`group_group.txt`) yükler.
            *   Dosyadaki her bir grup grubu tanımı için:
                *   `vnum` okur.
                *   Yeni bir `CMobGroupGroup` nesnesi oluşturur.
                *   Sırayla "1", "2", ... şeklinde etiketlenmiş token'ları okur. Her token bir `CMobGroup` VNUM'ını ve isteğe bağlı olarak bir olasılık değerini içerir. Bunları `pkGroup->AddMember()` ile grup grubuna ekler.
                *   Oluşturulan grup grubunu `m_map_pkMobGroupGroup` haritasına ekler.
        *   **`LoadGroup(const char* c_pszFileName)`:**
            *   `CTextFileLoader` kullanarak belirtilen dosyayı (`group.txt`) yükler.
            *   Dosyadaki her bir grup tanımı için:
                *   `vnum` ve lider yaratığın (`leader`) VNUM'ını okur.
                *   Yeni bir `CMobGroup` nesnesi oluşturur, `Create()` ile VNUM ve ismini ayarlar, lideri `AddMember()` ile ekler.
                *   Sırayla "1", "2", ... şeklinde etiketlenmiş token'ları okur. Her token bir yaratık VNUM'ını içerir. Bunları `pkGroup->AddMember()` ile gruba ekler.
                *   Oluşturulan grubu `m_map_pkMobGroup` haritasına ekler. Grup bilgilerini loglar.
*   **Orta Seviye İmpelentasyon Detayları:**
    *   Yaratık prototipleri, sunucu başlangıcında `game` çekirdeği tarafından yüklenen `mob_proto.txt` (veya benzeri bir dosya) içeriğinin `TMobTable` yapıları halinde `Initialize` metoduna verilmesiyle yüklenir.
    *   Yaratık grupları (`group.txt`) ve grup grupları (`group_group.txt`), metin tabanlı dosyalardan `CTextFileLoader` sınıfı yardımıyla okunur. Bu dosyalar, hangi yaratıkların birlikte veya belirli bir olasılıkla doğacağını tanımlar.
    *   `IncRegenCount` ve `DumpRegenCount` fonksiyonları, oyun dünyasındaki yaratıkların yeniden doğma sıklığını analiz etmek için kullanılan istatistiksel araçlardır. `regen.txt` dosyalarındaki bilgilerle bu sayaçlar güncellenir.
    *   `ADD_MOB_GROUP_GROUP_PROB` makrosu ile koşullu derlenen kod, `CMobGroupGroup`'ların üyelerini olasılığa dayalı olarak seçmesini sağlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `char_manager.h`, `item_manager.h`, `mob_manager.h`, `regen.h`, `sectree.h`, `text_file_loader.h`, `questmanager.h`, `locale_service.h`, `../common/VnumHelper.h`.

### `motion.h`

*   **Amaç:** Karakterlerin ve yaratıkların hareket (motion) ve animasyon verilerini yönetmek için sınıflar, enum'lar ve makrolar tanımlar. `.msa` (Metin2 Sanat Animasyonu) dosyalarından hareket verilerini yüklemek, saklamak ve erişmek için bir yapı sunar. Farklı hareket modları (genel, tek elli kılıç, at üstü vb.) ve genel hareket türleri (yürüme, koşma, saldırı, özel yetenekler vb.) için tanımlamalar içerir.
*   **Temel İşlevler/İçerik:**
    *   **`EMotionMode` Enum'u:** Karakterin veya yaratığın hangi hareket modunda olduğunu belirtir (örn. `MOTION_MODE_GENERAL`, `MOTION_MODE_ONEHAND_SWORD`, `MOTION_MODE_HORSE`). Bu, genellikle kuşanılan silaha veya duruma göre değişir.
    *   **`EPublicMotion` Enum'u:** Genel olarak kullanılabilen hareket türlerini tanımlar (örn. `MOTION_WAIT`, `MOTION_WALK`, `MOTION_RUN`, `MOTION_NORMAL_ATTACK`, `MOTION_SPECIAL_1` ila `MOTION_SPECIAL_5`).
    *   **`CMotion` Sınıfı:**
        *   Tek bir hareket (animasyon) dosyasının (`.msa`) verilerini temsil eder.
        *   `LoadFromFile()`: Genel bir `.msa` dosyasını yükler, hareket süresini (`m_fDuration`) ve birikimli hareket vektörünü (`m_vec3Accumulation`, `m_isAccumulation`) okur.
        *   `LoadMobSkillFromFile()`: Bir yaratığın yetenek hareketini (`.msa`) yükler. Dosyadan hareket süresini okur ve yetenekle ilgili özel olay verilerini (örn. `MOTION_EVENT_TYPE_SPECIAL_ATTACKING` için vuruş zamanlaması ve mesafesi) ayrıştırarak `CMob::AddSkillSplash()` ile kaydeder.
        *   `GetDuration()`: Hareketin süresini döndürür.
        *   `GetAccumVector()`: Hareketin birikimli yer değiştirme vektörünü döndürür (koşma gibi hareketler için önemlidir).
        *   `IsEmpty()`: Hareket verisinin boş olup olmadığını kontrol eder.
        *   **Korunan (Protected) Üyeler:** `m_isEmpty`, `m_fDuration`, `m_isAccumulation`, `m_vec3Accumulation`.
    *   **`MOTION_KEY` Typedef'i (`DWORD`):** Bir hareketi benzersiz şekilde tanımlayan bir anahtar.
    *   **Makrolar (`MAKE_MOTION_KEY`, `GET_MOTION_MODE` vb.):** `MOTION_KEY` oluşturmak ve bu anahtardan hareket modu, indeksi ve alt indeksi çıkarmak için kullanılır.
    *   **`CMotionSet` Sınıfı:**
        *   Belirli bir karakter türü (job) veya yaratık VNUM'ı için bir dizi `CMotion` nesnesini (farklı hareket modları ve türleri için) bir harita içinde tutar.
        *   `Insert()`: Bir `MOTION_KEY` ile bir `CMotion` nesnesini haritaya ekler.
        *   `Load()`: Belirtilen `.msa` dosyasını yükleyip `CMotion` nesnesi oluşturur ve verilen mod/hareket için haritaya ekler.
        *   `GetMotion()`: Verilen `MOTION_KEY`'e karşılık gelen `CMotion` nesnesini döndürür.
        *   **Korunan (Protected) Üye:** `m_map_pkMotion` (std::map<DWORD, CMotion*>).
    *   **`CMotionManager` Sınıfı (Singleton):**
        *   Tüm karakter türleri ve yaratıklar için `CMotionSet`'leri yönetir.
        *   `Build()`: Oyuncu karakterleri (farklı ırklar ve silah modları için) ve tüm yaratıklar için hareket setlerini (`.msa` dosyalarından ve yaratıkların `motlist.txt` dosyalarından) yükler.
        *   `GetMotionSet()`: Verilen VNUM (karakter job veya yaratık VNUM) için `CMotionSet`'i döndürür.
        *   `GetMotion()`: Verilen VNUM ve `MOTION_KEY` için belirli bir `CMotion` nesnesini döndürür.
        *   `GetMotionDuration()`: Belirli bir hareketin süresini döndürür.
        *   `GetNormalAttackDuration()` (`POLYMORPH_BUG_FIX`): Bir yaratığın normal saldırı hareketinin süresini döndürür.
        *   **Korunan (Protected) Üyeler:**
            *   `m_map_pkMotionSet` (std::map<DWORD, CMotionSet*>): Karakter job/yaratık VNUM -> `CMotionSet` eşleşmelerini tutar.
            *   `m_map_normalAttackDuration` (std::map<DWORD, float>) (`POLYMORPH_BUG_FIX`): Yaratık VNUM -> Normal saldırı süresi eşleşmelerini tutar.
*   **Bağlantılı Dosyalar:** `motion.cpp` (uygulama), `../../common/d3dtype.h` (`D3DVECTOR` tanımı), `stdafx.h`.

### `motion.cpp`

*   **Amaç:** `motion.h` dosyasında bildirilen `CMotion`, `CMotionSet` ve `CMotionManager` sınıflarının metotlarını uygular. Oyuncu karakterleri ve yaratıklar için hareket (animasyon) verilerinin `.msa` dosyalarından ve `motlist.txt` dosyalarından yüklenmesini, saklanmasını ve yönetilmesini sağlar. Yaratık yetenekleriyle ilişkili özel hareket olaylarını (vuruş zamanlamaları gibi) da işler.
*   **Temel İşlevleri/İçeriği:**
    *   **Statik Yardımcı Fonksiyonlar:**
        *   `MSA_GetNormalAttackDuration(const char* msaPath)` (`POLYMORPH_BUG_FIX`): Verilen `.msa` dosyasını okur ve içindeki "MotionDuration" değerini döndürür.
        *   `MOB_GetNormalAttackDuration(TMobTable* mobTable)` (`POLYMORPH_BUG_FIX`): Bir yaratığın `motlist.txt` dosyasını okur, "NORMAL_ATTACK" türündeki tüm hareketlerin `.msa` dosyalarını inceler ve en kısa "MotionDuration" değerini döndürür.
        *   `GetMotionFileName(TMobTable* mobTable, EPublicMotion motion)`: Bir yaratığın (`mobTable`) `motlist.txt` dosyasından, belirtilen genel hareket türü (`EPublicMotion`) için ilgili `.msa` dosyasının tam yolunu bulur ve döndürür. `motlist.txt` formatı: `MODE TYPE FILENAME PERCENT`.
        *   `LoadMotion(CMotionSet* pMotionSet, TMobTable* mob_table, EPublicMotion motion)`: `GetMotionFileName` ile `.msa` dosyasının adını alır, yeni bir `CMotion` nesnesi oluşturur, `LoadFromFile` ile yükler ve `pMotionSet`'e ekler. Koşma hareketi için birikimli veri olup olmadığını kontrol eder.
        *   `LoadSkillMotion(CMotionSet* pMotionSet, CMob* pMob, EPublicMotion motion)`: Belirli bir özel yetenek hareketi (`MOTION_SPECIAL_1` vb.) için `GetMotionFileName` ile `.msa` dosyasının adını alır. Yeni bir `CMotion` nesnesi oluşturur, `LoadMobSkillFromFile` ile yükler (bu sırada yetenek olayları `pMob->AddSkillSplash` ile kaydedilir) ve `pMotionSet`'e ekler.
    *   **`CMotionManager` Metotları:**
        *   Yapıcı/Yıkıcı: Hareket setlerini temizler.
        *   `GetMotionSet()`: `m_map_pkMotionSet`'ten VNUM'a göre `CMotionSet` döndürür.
        *   `GetMotion()`: `GetMotionSet` üzerinden ilgili `CMotion` nesnesini döndürür.
        *   `GetMotionDuration()`: `GetMotion` üzerinden hareket süresini döndürür.
        *   `GetNormalAttackDuration()` (`POLYMORPH_BUG_FIX`): `m_map_normalAttackDuration`'dan önceden hesaplanmış normal saldırı süresini döndürür.
        *   **`Build()`:**
            *   Tüm ana oyuncu karakter ırkları (`MAIN_RACE_MAX_NUM`) için döngüye girer.
            *   Her ırk için bir `CMotionSet` oluşturur ve `m_map_pkMotionSet`'e ekler.
            *   Her ırkın genel (`general`), ata binmiş (`horse`) ve kullandığı silahlara özel (örn. `onehand_sword`, `twohand_sword`, `bow`, `bell`, `fan`, `claw`) yürüme (`walk.msa`) ve koşma (`run.msa`) hareketlerini standart dosya yollarından (`data/pc/[ırk]/[mod]/run.msa` gibi) `CMotionSet::Load` ile yükler.
            *   Ardından, `CMobManager`'dan tüm yaratık prototiplerini alır.
            *   Her yaratık için (eğer `szFolder` tanımlıysa):
                *   Yeni bir `CMotionSet` oluşturur ve `m_map_pkMotionSet`'e yaratık VNUM'ı ile ekler.
                *   `LoadMotion` kullanarak yürüme, koşma ve normal saldırı hareketlerini yükler.
                *   `LoadSkillMotion` kullanarak `MOTION_SPECIAL_1`'den `MOTION_SPECIAL_5`'e kadar olan özel yetenek hareketlerini yükler.
                *   (`POLYMORPH_BUG_FIX`) `MOB_GetNormalAttackDuration` ile yaratığın normal saldırı süresini hesaplar ve `m_map_normalAttackDuration`'a kaydeder.
    *   **`CMotionSet` Metotları:**
        *   Yapıcı/Yıkıcı: İçerdiği `CMotion` nesnelerini siler.
        *   `GetMotion()`: `m_map_pkMotion`'dan `MOTION_KEY`'e göre `CMotion` nesnesini döndürür.
        *   `Insert()`: Verilen `MOTION_KEY` ile `CMotion` nesnesini `m_map_pkMotion`'a ekler.
        *   `Load()`: Verilen dosya adından `CMotion` yükler ve `Insert` ile ekler.
    *   **`CMotion` Metotları:**
        *   Yapıcı: `m_isEmpty`'yi `true`, `m_fDuration` ve `m_vec3Accumulation`'ı sıfır yapar.
        *   **`LoadMobSkillFromFile(const char* c_pszFileName, CMob* pMob, int iSkillIndex)`:**
            *   `CTextFileLoader` kullanarak `.msa` dosyasını yükler.
            *   Dosyadan "motionduration" değerini okur ve `m_fDuration`'a atar.
            *   Dosyadaki "motioneventdata" bölümünü işler:
                *   Her bir "event" için "motioneventtype" okur.
                *   Eğer tip `MOTION_EVENT_TYPE_SPECIAL_ATTACKING` ise:
                    *   "spheredata" bölümünden "position" (vuruşun göreceli konumu) ve "startingtime" (vuruş zamanı) okur.
                    *   `pMob->AddSkillSplash(iSkillIndex, ...)` çağırarak bu vuruş bilgisini (zamanlama ve vuruşun y eksenindeki mesafesi) yaratığın ilgili yetenek bilgisine ekler.
            *   Yaratığın `m_mobSkillInfo` dizisindeki ilgili yetenek için VNUM ve seviyeyi ayarlar.
            *   `m_isEmpty`'yi `false` yapar.
        *   **`LoadFromFile(const char* c_pszFileName)`:**
            *   `CTextFileLoader` kullanarak `.msa` dosyasını yükler.
            *   Dosyadan "motionduration" (`m_fDuration`) ve eğer varsa "accumulation" (`m_vec3Accumulation`, `m_isAccumulation`) değerlerini okur.
            *   `m_isEmpty`'yi `false` yapar.
        *   `GetDuration()`, `GetAccumVector()`, `IsEmpty()`: İlgili üye değişkenlerin değerlerini döndürür.
    *   **`EMotionEventType` Enum'u:** `.msa` dosyalarındaki olay türlerini tanımlar (örn. `MOTION_EVENT_TYPE_EFFECT`, `MOTION_EVENT_TYPE_SPECIAL_ATTACKING`, `MOTION_EVENT_TYPE_SOUND`).
*   **Orta Seviye İmpelentasyon Detayları:**
    *   Hareketler (animasyonlar), `.msa` uzantılı metin tabanlı dosyalarda tanımlanır. Bu dosyalar, hareketin süresini, yer değiştirme miktarını (birikim) ve hareket sırasında tetiklenecek olayları (efektler, vuruşlar, sesler) içerir.
    *   Oyuncu karakterlerinin hareketleri, karakter ırkına ve kuşanılan silah moduna göre `data/pc/` dizini altındaki belirli dosya yollarından yüklenir.
    *   Yaratıkların hareketleri, her yaratığın kendi `data/monster/[klasör_adı]/motlist.txt` dosyasında listelenen `.msa` dosyalarından yüklenir. `motlist.txt`, hangi hareket türünün (örn. WALK, RUN, NORMAL_ATTACK, SPECIAL) hangi `.msa` dosyasına karşılık geldiğini belirtir.
    *   `POLYMORPH_BUG_FIX` ile işaretlenmiş kod blokları, özellikle dönüşüm (polymorph) sırasında yaratıkların saldırı süreleriyle ilgili bir hatayı düzeltmek için eklenmiş gibi görünmektedir. Bu düzeltme, normal saldırı sürelerini önceden hesaplayıp bir haritada saklamayı içerir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `../../common/stl.h`, `constants.h`, `motion.h`, `text_file_loader.h`, `mob_manager.h`, `char.h`.

### `p2p.h`

*   **Amaç:** Sunucular veya oyun kanalları arasındaki Eşler Arası (Peer-to-Peer - P2P) iletişimi yönetmek için `P2P_MANAGER` singleton sınıfını ve ilgili yapıları tanımlar. Farklı sunucu instansları arasında oyuncu verilerinin (login, logout, konum vb.) ve bazı sosyal sistemlerin (lonca, grup, messenger, evlilik) senkronizasyonunu sağlar.
*   **Temel İşlevler/İçeriği:**
    *   **`_CCI` (Channel Character Info) Struct:** Farklı bir kanalda/sunucuda bulunan bir karakter hakkındaki temel bilgileri tutar:
        *   `szName`: Karakter adı.
        *   `dwPID`: Karakterin Oyuncu ID'si.
        *   `bEmpire`: İmparatorluğu.
        *   `lMapIndex`: Bulunduğu haritanın indeksi.
        *   `bChannel`: Bulunduğu kanal.
        *   `bLanguage`: Karakterin dil ayarı.
        *   `pkDesc`: Bu bilgiyi gönderen P2P bağlantısının deskriptörü.
    *   **`P2P_MANAGER` Sınıfı (Singleton):**
        *   **Bağlantı Yönetimi:**
            *   `RegisterAcceptor() / UnregisterAcceptor()`: Gelen P2P bağlantılarını kaydeder/kaldırır.
            *   `RegisterConnector() / UnregisterConnector()`: Başlatılan P2P bağlantılarını kaydeder/kaldırır.
            *   `EraseUserByDesc(LPDESC d)`: Belirli bir P2P bağlantısıyla ilişkili tüm kullanıcı bilgilerini siler.
            *   `FlushOutput()`: Tüm P2P bağlantılarındaki bekleyen verileri gönderir.
        *   **Veri Gönderimi ve Senkronizasyon:**
            *   `Boot(LPDESC d)`: Yeni bir P2P bağlantısı kurulduğunda, mevcut sunucudaki tüm oyuncuların login bilgilerini bu bağlantıya gönderir.
            *   `Send(const void* c_pvData, int iSize, LPDESC except = NULL)`: Verilen datayı, belirtilen deskriptör hariç tüm P2P peer'lerine gönderir.
            *   `Login(LPDESC d, const TPacketGGLogin* p)`: Bir oyuncunun başka bir P2P peer'den geldiği bilgisini alır, yerel olarak kaydeder ve ilgili sistemleri (lonca, grup, messenger) günceller. Ayrıca imparatorluk bazlı kullanıcı sayısını artırır.
            *   `Logout(const char* c_pszName)` / `Logout(CCI* pkCCI)`: Bir oyuncunun P2P ağından ayrıldığı bilgisini işler, yerel kayıtlardan siler ve ilgili sistemleri günceller. İmparatorluk bazlı kullanıcı sayısını azaltır.
        *   **Bilgi Alma:**
            *   `Find(const char* c_pszName)`: İsimle P2P kullanıcısını bulur.
            *   `FindByPID(DWORD pid)`: PID ile P2P kullanıcısını bulur.
            *   `GetPeer(DWORD dwP2PPort)`: Belirli bir P2P portu üzerinden bağlı olan peer'in deskriptörünü döndürür.
            *   `GetCount()`: Mevcut sunucudaki (g_bChannel) toplam oyuncu sayısını (imparatorluklara göre) döndürür.
            *   `GetPIDCount()`: P2P üzerinden bilinen toplam farklı oyuncu (CCI) sayısını döndürür.
            *   `GetEmpireUserCount(int idx)`: Belirli bir imparatorluktaki (mevcut kanalda) kullanıcı sayısını döndürür.
            *   `GetDescCount()`: Aktif P2P bağlantı sayısını döndürür.
            *   `GetP2PHostNames(std::string& hostNames)`: Bağlı tüm P2P peer'lerinin host ve port bilgilerini bir string'e yazar.
        *   **Dahili Veri Yapıları:**
            *   `m_set_pkPeers`: Aktif P2P bağlantılarının (LPDESC) kümesi.
            *   `m_map_pkCCI`: Karakter adına göre `CCI` yapılarının haritası.
            *   `m_map_dwPID_pkCCI`: Oyuncu ID'sine göre `CCI` yapılarının haritası.
            *   `m_aiEmpireUserCount`: Her imparatorluk için mevcut kanaldaki kullanıcı sayısını tutan dizi.

### `p2p.cpp`

*   **Amaç:** `p2p.h` dosyasında tanımlanan `P2P_MANAGER` sınıfının metotlarını uygular. Sunucular arası iletişimi, oyuncu bilgilerinin senkronizasyonunu ve P2P bağlantı yaşam döngüsünü yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Başlangıç (`P2P_MANAGER::P2P_MANAGER`)**: İmparatorluk kullanıcı sayılarını sıfırlar.
    *   **Boot (`P2P_MANAGER::Boot`):** Yeni bir P2P bağlantısı (`LPDESC d`) kurulduğunda çağrılır. Mevcut sunucudaki `CHARACTER_MANAGER`'dan tüm oyuncuların listesini alır ve her biri için bir `TPacketGGLogin` paketi oluşturarak yeni bağlanan peer'e gönderir. Bu, yeni bağlanan sunucunun mevcut oyuncular hakkında bilgi sahibi olmasını sağlar.
    *   **Bağlantı Kaydı (`RegisterAcceptor`, `RegisterConnector`):** Yeni P2P deskriptörlerini `m_set_pkPeers` kümesine ekler ve `Boot` işlemini tetikler. `RegisterConnector` ayrıca karşı tarafa kendi P2P portunu ve kanal bilgisini içeren bir `TPacketGGSetup` paketi gönderir.
    *   **Bağlantı Kesilmesi (`UnregisterAcceptor`, `UnregisterConnector`, `EraseUserByDesc`):** P2P deskriptörü `m_set_pkPeers` kümesinden çıkarılır. `EraseUserByDesc` ile bu deskriptör üzerinden giriş yapmış tüm kullanıcılar `Logout` edilir.
    *   **Login (`P2P_MANAGER::Login`):**
        *   Bir `TPacketGGLogin` paketi (başka bir sunucudan gelen oyuncu bilgisi) alındığında çağrılır.
        *   Oyuncunun (`CCI`) zaten `m_map_pkCCI` veya `m_map_dwPID_pkCCI`'de olup olmadığını kontrol eder.
        *   Eğer yeni bir oyuncuysa, yeni bir `CCI` nesnesi oluşturur, bilgilerini doldurur ve haritalara ekler. Eğer oyuncu mevcut sunucunun kanalı (`g_bChannel`) üzerinden geliyorsa, `m_aiEmpireUserCount` güncellenir.
        *   Oyuncunun harita indeksi, deskriptörü ve kanalı güncellenir.
        *   `CGuildManager`, `CPartyManager` ve (eğer yeni bir CCI ise) `MessengerManager`'ın P2P login fonksiyonları çağrılarak bu sistemlerin senkronize olması sağlanır.
    *   **Logout (`P2P_MANAGER::Logout`):**
        *   Belirtilen bir `CCI` veya karakter adıyla çağrılır.
        *   Eğer oyuncu mevcut sunucunun kanalı üzerinden çıkış yapıyorsa `m_aiEmpireUserCount` azaltılır.
        *   `CGuildManager`, `CPartyManager`, `MessengerManager` ve `marriage::CManager`'ın P2P logout fonksiyonları çağrılır.
        *   `CCI` nesnesi haritalardan silinir ve bellekten serbest bırakılır (`M2_DELETE`).
    *   **Oyuncu Arama (`FindByPID`, `Find`):** İlgili haritalarda (PID veya isme göre) `CCI` nesnesini arar. Brezilya lokali için isimle arama (`Find`) yapılırken isim küçük harfe çevrilir ve başındaki/sonundaki boşluklar temizlenir (`trim_and_lower`).
    *   **Peer Arama (`GetPeer`):** Verilen P2P portuna sahip `LPDESC`'yi `m_set_pkPeers` içinde arar.
    *   **Kullanıcı Sayıları (`GetCount`, `GetEmpireUserCount`):** `m_aiEmpireUserCount` dizisini kullanarak mevcut kanaldaki toplam veya imparatorluk bazlı oyuncu sayılarını döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `../../common/stl.h`, `constants.h`, `config.h`, `p2p.h`, `desc_p2p.h`, `char.h`, `char_manager.h`, `sectree_manager.h`, `guild_manager.h`, `party.h`, `messenger_manager.h`, `marriage.h`, `utils.h`, `locale_service.h`, `<sstream>`.

// ... existing code ... 