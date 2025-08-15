# Metin2 Oyun Sunucusu - Yardımcı Bileşenler Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) yardımcı sınıflarını, veri yapılarını, ağ iletişimini, konfigürasyonunu, veritabanı etkileşimini, loglamayı ve diğer destekleyici bileşenlerini belgeler.

## İçindekiler

*   [Klasörler](#klasörler)
    *   [`lzo/`](#lzo)
    *   [`perftest/`](#perftest)
*   [Geliştirme Betikleri](#geliştirme-betikleri)
    *   [`build_locale_string.py`](#build_locale_stringpy)
*   [Derleme Yapılandırması](#derleme-yapılandırması)
    *   [`CMakeLists.txt`](#cmakelists-txt)
    *   [`Depend`](#depend)
*   [GM Komutları](#gm-komutları)
    *   [`cmd_gm.cpp`](#cmd_gmcpp)
*   [Header Dosyaları (.h)](#header-dosyaları-h)
    *   [`cmd.h`](#cmdh)
    *   [`config.h`](#configh)
    *   [`crc32.h`](#crch)
    *   [`CsvReader.h`](#csvreaderh)
    *   [`db.h`](#dbh)
    *   [`debug_allocator_adapter.h`](#debug_allocator_adapterh)
    *   [`debug_allocator.h`](#debug_allocatorh)
    *   [`debug_ptr.h`](#debug_ptrh)
    *   [`desc_client.h`](#desc_clienth)
    *   [`desc_manager.h`](#desc_managerh)
*   [Kaynak Kod Dosyaları (.cpp)](#kaynak-kod-dosyaları-cpp)
    *   [`cmd.cpp`](#cmdcpp)
    *   [`config.cpp`](#configcpp)
    *   [`crc32.cpp`](#crc32cpp)
    *   [`CsvReader.cpp`](#csvreadercpp)
    *   [`db.cpp`](#dbcpp)
    *   [`desc_client.cpp`](#desc_clientcpp)
    *   [`desc_manager.cpp`](#desc_managercpp)
*   [Kimlik Doğrulama Girdi İşlemleri (input_auth.cpp)](#kimlik-doğrulama-girdi-işlemleri-input_authcpp)
*   [Veritabanı Girdi İşlemleri (input_db.cpp)](#veritabanı-girdi-işlemleri-input_dbcpp)

---

## Klasörler

### `lzo/`

*   **Amaç:** LZO (Lempel–Ziv–Oberhumer) veri sıkıştırma algoritmasının kütüphane dosyalarını içerir.
*   **Temel İşlevler/İçerik:**
    *   Bu klasör, LZO sıkıştırma ve açma işlemleri için gerekli olan header dosyalarını (`.h`) barındırır.
    *   Metin2 sunucusu, genellikle istemci ile sunucu arasındaki ağ trafiğini azaltmak için ağ paketlerini sıkıştırmak/açmak amacıyla bu kütüphanenin kullanılır. Ayrıca bazı oyun dosyalarının veya verilerinin sıkıştırılmasında da kullanılabilir.
    *   İçerdiği `.h` dosyaları (`lzo1.h`, `lzo1a.h`, `lzo1b.h`, `lzo1c.h`, `lzo1f.h`, `lzo1x.h`, `lzo1y.h`, `lzo1z.h`, `lzo2a.h`, `lzo16bit.h`, `lzoconf.h`, `lzoutil.h`) LZO kütüphanesinin farklı sıkıştırma seviyeleri ve fonksiyonları için arayüz tanımlamalarını sağlar.
    *   `Makefile` dosyaları, kütüphanenin derlenmesi için kullanılan yapılandırma dosyalarıdır. Kaynak kod dosyaları (`.c`/`.cpp`) genellikle bu klasörde bulunmaz, kütüphane genellikle önceden derlenmiş olarak sisteme eklenir veya projenin başka bir bölümünde yer alır.
*   **Önemli Notlar:** Bu, harici bir üçüncü parti kütüphanedir.

### `perftest/`

*   **Amaç:** Performans testiyle ilgili yardımcı fonksiyonları ve test kodlarını içerir. Bu klasördeki kodlar genellikle doğrudan oyunun çalışma zamanı mantığında kullanılmaz, daha çok geliştirme ve optimizasyon aşamalarında performans analizi yapmak için kullanılır.
*   **Dosyalar:**
    *   **`timeval_subtract.c`**
        *   **İşlev:** POSIX `struct timeval` tipindeki iki zaman damgası arasındaki farkı hesaplayan `timeval_subtract` fonksiyonunu içerir.
        *   **Kullanım:** Kod bloklarının veya fonksiyonların çalışma süresini mikrosaniye hassasiyetinde ölçmek için kullanılır.
        *   **İmplementasyon:** Standart C ile yazılmıştır. Negatif fark durumunda 1 döndürür.
    *   **`alloc_perf_test.cpp`**
        *   **İşlev:** Farklı bellek ayırma (allocation/deallocation) stratejilerinin performansını karşılaştırmak için bir test programı içerir.
        *   **Kullanım:** Geliştirme sırasında standart (`new`/`delete`, `malloc`/`free`) ve Metin2'nin özel (`M2_NEW`/`M2_DELETE`, `M2_MALLOC`/`M2_FREE`) bellek yöneticilerinin (normal ve debug modları dahil) hızını ölçmek için kullanılır.
        *   **İmplementasyon:** Önişlemci direktifleri (`#define CASE_...`) ile farklı test senaryoları (raw, allocator, debug allocator; new/delete veya malloc/free) seçilir. `kCount` (test edilecek örnek sayısı) kadar `Foo` nesnesi yaratılıp silinir ve geçen süre `gettimeofday` ve `timeval_subtract` kullanılarak ölçülür. Metin2'nin `../allocator.h` dosyasındaki `Allocator` sınıfını kullanır.
*   **Önemli Notlar:** Bu klasördeki kodlar genellikle doğrudan oyunun çalışma zamanı mantığında kullanılmaz, daha çok geliştirme ve optimizasyon aşamalarında performans analizi yapmak için kullanılır.

---

## Geliştirme Betikleri

### `build_locale_string.py`

*   **Amaç:** Mevcut dizindeki tüm `.cpp` dosyalarını tarayarak `TEXT("...")` makrosu içinde kullanılan metinleri (string literals) çıkarır ve bunları `locale_string.txt` adlı bir dosyaya, her satıra bir metin gelecek şekilde yazar. Oyunun yerelleştirilmesi sürecinde kullanılan metinleri toplamak için bir yardımcı araçtır.
*   **Temel İşlevler/İçerik:**
    *   `.cpp` dosyalarını tarar.
    *   Çok satırlı yorum bloklarını (`/* ... */`) atlar.
    *   `TEXT("...")` kalıbını arar.
    *   Çift tırnak arasındaki metni alır.
    *   Tekrarları önleyerek metinleri bir listede toplar.
    *   Toplanan metinleri `locale_string.txt` dosyasına belirli bir formatta yazar ve konsola basar.
*   **Kullanım:** Geliştiriciler tarafından, kod içerisindeki yerelleştirilecek metinleri otomatik olarak toplamak amacıyla kullanılır. Çalışan oyun sunucusunun bir parçası **değildir**.
*   **Not:** Betik Python 2 sözdizimi kullanıyor olabilir (`print "..."`). Python 3 ortamında çalıştırılacaksa düzenleme gerekebilir.

---

## Derleme Yapılandırması

### `CMakeLists.txt`

*   **Amaç:** Metin2 oyun sunucusunun ana yürütülebilir dosyası (`game`) için CMake derleme yapılandırmasını tanımlar. Hangi kaynak dosyaların derleneceğini, hangi kütüphanelerin bağlanacağını, gerekli başlık dosyası yollarını, derleyici ve bağlayıcı seçeneklerini belirtir.
*   **Temel İşlevler/İçerik:**
    *   **Proje Adı:** `set(PROJECT_NAME game)` ile projenin adını belirler.
    *   **Kaynak Grupları:** `set(Headers ...)` ve `set(Sources ...)` ile projeye dahil edilecek tüm başlık (`.h`) ve kaynak (`.cpp`, `.c`) dosyalarını listeler. `source_group` komutuyla bu dosyaları IDE'lerde (örneğin Visual Studio) mantıksal gruplar altında organize eder.
    *   **Yürütülebilir Hedef Tanımı:** `add_executable(${PROJECT_NAME} ${ALL_FILES})` komutu ile listelenen tüm kaynak dosyalarını kullanarak `game` adında bir yürütülebilir hedef oluşturur.
    *   **Hedef Özellikleri:** `set_target_properties` ile hedefin çeşitli özelliklerini ayarlar:
        *   `FOLDER "Servers"`: IDE'de hedefi "Servers" klasörü altına yerleştirir.
        *   `TARGET_NAME_DEBUG`: Debug derlemeleri için hedefin adını (`game_d`) ayarlar.
        *   `OUTPUT_DIRECTORY_DEBUG`: Debug derlemesinin çıktısının (oluşturulan `.exe` dosyası) nereye konulacağını (`../` yani `Source/game/`) belirtir.
        *   `MSVC_RUNTIME_LIBRARY`: Microsoft Visual C++ derleyicisi için kullanılacak çalışma zamanı kütüphanesini (Debug için `MultiThreadedDebug`) ayarlar.
    *   **Dahil Etme Dizinleri:** `target_include_directories` ile derleyicinin başlık dosyalarını arayacağı dizinleri belirtir. Bu, hem projenin diğer kütüphanelerini (`../../libserverkey`, `../../liblua/include`) hem de harici kütüphaneleri (`../../../External/include`, `../../../External/MySQL/...`) içerir.
    *   **Derleme Tanımları:** `target_compile_definitions` ile derleme sırasında kullanılacak önişlemci makrolarını tanımlar (örneğin `WIN32`, `_DEBUG`, `__WIN32__`, `_USE_32BIT_TIME_T`).
    *   **Derleme Seçenekleri:** `target_compile_options` ile derleyiciye özel bayrakları ayarlar (örneğin `/MP` - çoklu işlemci derlemesi, `/std:c++latest` - C++ standardı, `/Od` - optimizasyonları kapat, `/Zi` - debug bilgisi).
    *   **Bağlama Seçenekleri:** `target_link_options` ile bağlayıcıya özel bayrakları ayarlar (örneğin `/DEBUG`, `/MACHINE:X86`, `/SUBSYSTEM:CONSOLE`).
    *   **Bağımlılıklar:**
        *   `target_link_libraries(${PROJECT_NAME} PRIVATE libgame liblua ...)`: `game` hedefinin projenin diğer kütüphanelerine (`libgame`, `liblua`, `libpoly`, `libserverkey`, `libsql`, `libthecore`) bağlandığını belirtir.
        *   `target_link_libraries(${PROJECT_NAME} PRIVATE "${ADDITIONAL_LIBRARY_DEPENDENCIES}")`: Harici kütüphanelere (`libosp_d`, `cryptlibd`, `mysqlclient`, `ws2_32`, `DevIL-*`) bağlandığını belirtir.
    *   **Bağlama Dizinleri:** `target_link_directories` ile bağlayıcının kütüphane dosyalarını (`.lib`) arayacağı dizinleri belirtir (Harici kütüphanelerin ve `libosp`'nin yolları).

### `Depend`

*   **Amaç:** Derleme sistemi (genellikle `make`) tarafından otomatik olarak oluşturulan bir bağımlılık dosyasıdır. Hangi kaynak dosyasının (`.cpp`, `.c`) hangi başlık dosyalarına (`.h`) bağlı olduğunu listeler. Derleme sistemi, bir başlık dosyası değiştiğinde sadece ilgili kaynak dosyalarını yeniden derlemek için bu bilgiyi kullanır (artımlı derleme). Oyun mantığını veya kodunu **içermez** ve manuel olarak düzenlenmemelidir.
*   **Not:** Bu dosyanın içeriği genellikle çok uzundur ve derleme ortamına göre değişir.

---

## GM Komutları

### `cmd_gm.cpp`

*   **Amaç:** Oyun Yöneticileri (Game Masters - GM) tarafından kullanılan özel komutları (`ACMD`) uygular. Bu komutlar, sunucu yönetimi, oyuncu yönetimi, içerik oluşturma/yönetme, test etme ve hata ayıklama gibi geniş bir yelpazede işlevsellik sağlar. Normal oyuncuların kullanamadığı yetkiler gerektirir.
*   **Temel İşlev Grupları ve Önemli Komutlar:**
    *   **Oyuncu Yönetimi:** `/transfer`, `/warp`, `/dc`, `/kill`, `/stun`, `/slow`, `/purge` (yakındaki mobları silme), `/item_purge` (envanter temizleme), `/advance` (seviye ayarlama), `/set` (çeşitli değerleri ayarlama), `/setskill`, `/setskillother`, `/reset_subskill`, `/affect_remove`, `/block_chat`, `/hwid_ban`.
    *   **İçerik Oluşturma/Yönetme:** `/item`, `/mob`, `/group`, `/regen` (veya `/respawn`), `/book` (beceri kitabı), `/polymorph`, `/build` (lonca yapısı).
    *   **Sunucu/Sistem Yönetimi:** `/shutdown`, `/who`, `/online`, `/user` (oyuncu listesi), `/notice`, `/big_notice`, `/map_notice` (duyuru), `/reload` (veri yenileme), `/event_flag`, `/priv_empire`, `/priv_guild` (bonus oranları), `/clear_land` (lonca arazisi temizleme), `/special_item` (özel drop yükleme), `/pcbang_update` (PCBang IP listesi), `/siege` (kale savaşı), `/threeway_war_info` (Üç Yol Savaşı bilgisi), `/flush` (oyuncu önbelleği temizleme), `/eclipse` (ay tutulması), `/weeklyevent` (Savaş Arenası), `/event_helper` (Noel yardımcısı), `/banner` (event banner).
    *   **Test/Hata Ayıklama:** `/state`, `/state_attr` (efsun bilgisi), `/invisibility` (görünmezlik), `/observer` (gözlemci modu), `/level`, `/horse_*` (at komutları), `/mount_test`, `/private` (özel harita), `/save_attribute_to_image`, `/change_attr`, `/add_attr`, `/add_socket` (silahtaki efsun/soket), `/fishing_simul`, `/refine_*` (balıkçılık/madencilik geliştirme), `/getqf`, `/setqf`, `/delqf` (quest flag), `/forgetme`, `/aggregate`, `/attract_ranger`, `/pull_monster` (mob aggro), `/weaken` (mob zayıflatma), `/cooltime` (bekleme süresi kaldırma), `/can_dead` (ölümsüzlük), `/full_set` (tam ekipman/skill), `/use_item`, `/clear_affect`, `/kill_all` (yakındaki oyuncuları öldürme), `/drop_item`, `/dragon_soul` (Simya destesi), `/ds_list` (Simya envanteri), `/growth_pet` (Geliştirilebilir Pet).
    *   **Lonca/Evlilik/Hükümdarlık:** `/makeguild`, `/deleteguild`, `/greset` (lonca işlemleri), `/gwlist`, `/stop_guild_war`, `/cancel_guild_war` (lonca savaşı yönetimi), `/guild_state`, `/break_marriage`, `/rmcandidacy`, `/setmonarch`, `/rmmonarch` (hükümdarlık yönetimi).
*   **Önemli Notlar:** Bu dosyadaki komutların çoğu belirli bir GM seviyesi (`ch->GetGMLevel()`) gerektirir ve normal oyuncular tarafından kullanılamaz. Birçok komut, diğer modüllerle (örneğin, `CHARACTER_MANAGER`, `ITEM_MANAGER`, `quest::CQuestManager`, `CGuildManager`) etkileşime girer.
*   **Bağlantılı Dosyalar:** `stdafx.h` ve `game/src` altındaki hemen hemen tüm diğer başlık dosyalarını içerir, çünkü çok geniş bir işlevselliği kapsar.

---

## Header Dosyaları (.h)

### `cmd.h`

*   **Amaç:** Oyun sunucusunun komut yorumlama sistemi için temel tanımlamaları ve bildirimleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`ACMD` Makrosu:** Standart komut işleyici fonksiyon prototipini tanımlar: `void (name)(LPCHARACTER ch, const char *argument, int cmd, int subcmd)`.
    *   **`command_info` Struct:** Her bir oyun içi komutun özelliklerini tanımlayan yapı:
        *   `command` (const char*): Komutu tetikleyen metin (örn. "warp", "mob").
        *   `command_pointer` (fonksiyon işaretçisi): Komutu işleyecek olan `ACMD` fonksiyonu.
        *   `subcmd` (int): Aynı işleyici fonksiyonu kullanan farklı komutları ayırt etmek için alt komut numarası.
        *   `minimum_position` (int): Komutu kullanmak için gereken minimum karakter pozisyonu (`POS_*` enum sabitleri).
        *   `gm_level` (int): Komutu kullanmak için gereken minimum GM seviyesi (`GM_*` enum sabitleri).
    *   **`cmd_info[]` Bildirimi:** Tüm komut tanımlarını içeren global `command_info` dizisinin harici bildirimini yapar.
    *   **Fonksiyon Bildirimleri:**
        *   `interpret_command`: Ana komut yorumlayıcı fonksiyon.
        *   `interpreter_set_privilege`: Komut yetki seviyesini değiştirme fonksiyonu.
        *   `Shutdown`: Sunucuyu kapatma fonksiyonu.
        *   `Send*Notice`, `Broadcast*Notice`: Çeşitli duyuru gönderme fonksiyonları.
        *   Diğer yardımcı fonksiyonlar (`SendLog`, `CHARACTER_AddGotoInfo` vb.).
    *   **Enum Tanımları:** Belirli komut grupları için alt komut numaralarını (`SCMD_ACTION`, `SCMD_CMD`, `SCMD_RESTART`, `SCMD_XMAS`) tanımlar.
*   **Bağlantılı Dosyalar:** `cmd.cpp` (uygulama), `char.h` (LPCHARACTER tanımı için), `constants.h` (GM seviyeleri, pozisyonlar için).

### `config.h`

*   **Amaç:** Oyun sunucusunun global yapılandırma değişkenlerini, sabitlerini ve ilgili fonksiyon bildirimlerini tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Global Değişkenler:** Sunucunun çalışması için kritik olan çok sayıda global değişkeni (`extern`) bildirir. Bunlar arasında şunlar bulunur:
        *   **Ağ:** `mother_port`, `p2p_port`, `db_port`, `db_addr`, `g_szPublicIP`, `g_szInternalIP`, `g_stAuthMasterIP`, `g_wAuthMasterPort`.
        *   **Sunucu Genel:** `passes_per_sec`, `g_bChannel`, `g_iUserLimit`, `g_server_id`, `g_stHostname`, `g_stLocale`, `test_server`, `speed_server`.
        *   **Zamanlama Döngüleri:** `save_event_second_cycle`, `ping_event_second_cycle`.
        *   **Oyun Mekaniği:** `PK_PROTECT_LEVEL`, `gPlayerMaxLevel`, `gPlayerMaxLevelStats`, `gPlayerMaxConquerorLevel`, `guild_mark_min_level`, `VIEW_RANGE`, `VIEW_BONUS_RANGE`, `g_MaxGold`, `g_MaxCheque`, `gMaxItemCount`, `g_npcGroupRespawnRange`.
        *   **Özellik Bayrakları:** `guild_mark_server`, `g_bSkillDisable`, `g_bEmpireWhisper`, `g_bAuthServer`, `g_bCheckClientVersion`, `g_bCheckMultiHack`, `g_protectNormalPlayer`, `g_noticeBattleZone`, `gHackCheckEnable`, `g_BlockCharCreation`, ve birçok diğer boolean bayrak (örn. `g_bSoulBind`, `g_bUnlimitedCapeOfCourage`, `g_bNeverFailMetin`).
        *   **Yollar/URL'ler:** `g_stQuestDir`, `g_setQuestObjectDir`, `g_strWebMallURL`.
        *   **Güvenlik/Limitler:** `SPEEDHACK_LIMIT_COUNT`, `SPEEDHACK_LIMIT_BONUS`, `g_iSyncHackLimitCount`, `g_stAdminPageIP`, `g_stAdminPagePassword`.
    *   **Fonksiyon Bildirimleri:** Yapılandırma ve ilgili işlevler için fonksiyon prototiplerini içerir:
        *   `config_init`: Ana yapılandırma yükleme fonksiyonu.
        *   `map_allow_find`, `map_allow_copy`: İzin verilen haritaları yönetme.
        *   `get_table_postfix`: Veritabanı tablo son ekini alma.
        *   `LoadValidCRCList`, `IsValidProcessCRC`, `IsValidFileCRC`: İstemci CRC kontrolü.
        *   `CheckClientVersion`: İstemci sürüm kontrolü.
        *   `LoadStateUserCount`: Sunucu yoğunluk durumlarını yükleme.
*   **Bağlantılı Dosyalar:** `config.cpp` (uygulama), ve bu global değişkenleri veya fonksiyonları kullanan diğer birçok sunucu dosyası.

### `crc32.h`

*   **Amaç:** CRC32 (Cyclic Redundancy Check) sağlama toplamı ve hızlı bir hash değeri hesaplamak için kullanılan fonksiyonların bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   `typedef unsigned long crc_t;`: CRC değerini tutmak için kullanılan tür tanımı.
    *   `GetCRC32(const char* buffer, size_t count)`: Verilen bellek tamponu (`buffer`) ve boyutu (`count`) için standart, büyük/küçük harfe duyarlı CRC32 değerini hesaplar.
    *   `GetCaseCRC32(const char* buffer, size_t count)`: Verilen bellek tamponu için büyük/küçük harfe duyarsız CRC32 değerini hesaplar.
    *   `GetFastHash(const char* key, size_t len)`: Verilen anahtar (`key`) ve uzunluk (`len`) için hızlı bir hash değeri hesaplar (FNV-1a benzeri, CRC32 değil).
*   **Kullanım Alanları:** Veri bütünlüğü kontrolü (dosya veya paket bütünlüğü), istemci tarafı hile tespiti (dosya CRC kontrolü), veya basit hash tablosu anahtarları oluşturma.
*   **Bağlantılı Dosyalar:** `crc32.cpp`.

### `CsvReader.h`

*   **Amaç:** CSV (Comma Separated Values - Virgülle Ayrılmış Değerler) dosyalarını okumak, ayrıştırmak ve yazmak için kullanılan sınıfları tanımlar. Sütunlara isim verme (alias) ve verilere farklı türlerde erişim imkanı sunar.
*   **Temel İşlevler/İçerik:**
    *   **`cCsvAlias` Sınıfı:**
        *   Sütun indekslerine (örn. 0, 1, 2) anlamlı isimler (örn. "VNUM", "NAME") atamak için kullanılır.
        *   `AddAlias(isim, indeks)`: Bir isim-indeks eşleşmesi ekler.
        *   `operator[]`: İsimden indekse veya indeksten isme dönüşüm sağlar.
    *   **`cCsvRow` Sınıfı:**
        *   CSV dosyasının tek bir satırını temsil eder. `std::vector<std::string>`'den türemiştir.
        *   Her eleman bir sütun verisini string olarak tutar.
        *   `AsInt(indeks/isim)`, `AsDouble(indeks/isim)`, `AsString(indeks/isim)`: Belirtilen sütundaki veriyi istenen türe dönüştürerek döndürür.
    *   **`cCsvFile` Sınıfı:**
        *   Bir CSV dosyasının tamamını temsil eder. İçinde `cCsvRow*` listesi tutar.
        *   `Load(dosyaAdı, ayırıcı, tırnak)`: Dosyayı yükler, satırlara ve sütunlara ayırır. Tırnak içindeki ayırıcıları ve çift tırnakla temsil edilen tırnak karakterlerini doğru şekilde işler.
        *   `Save(dosyaAdı, ekle, ayırıcı, tırnak)`: Bellekteki veriyi dosyaya yazar. Gerekirse tırnak ve kaçış karakterleri ekler.
        *   `operator[]`: Belirli bir satıra erişim sağlar.
        *   `GetRowCount()`: Satır sayısını döndürür.
    *   **`cCsvTable` Sınıfı:**
        *   `cCsvFile` ve `cCsvAlias'ı birleştiren, tablo şeklinde CSV okumayı kolaylaştıran bir sarmalayıcıdır.
        *   `Load`: Dosyayı yükler.
        *   `AddAlias`: Sütun isimlerini tanımlar.
        *   `Next()`: İmleci bir sonraki satıra taşır.
        *   `ColCount()`: Geçerli satırdaki sütun sayısını verir.
        *   `AsInt`, `AsDouble`, `AsString`: Geçerli satırdaki sütun verisine isim veya indeks ile erişir.
*   **Kullanım Alanları:** Oyun verilerini (eşya prototipleri, mob prototipleri, ayarlar vb.) tutan CSV dosyalarını okumak veya logları/raporları CSV formatında yazmak.

### `db.h`

*   **Amaç:** Oyun sunucusunun MySQL veritabanlarıyla (oyun ve hesap) etkileşim kurmasını sağlayan temel sınıfları (`DBManager`, `AccountDB`), ilgili veri yapılarını ve sabitleri tanımlar. Asenkron ve senkron (doğrudan) veritabanı sorguları için arayüzler sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler/Enum'lar:**
        *   **Sorgu Türleri:** `QUERY_TYPE_RETURN` (sonuçları `AnalyzeReturnQuery`'de işler), `QUERY_TYPE_FUNCTION` (sonuç geldiğinde belirtilen fonksiyonu çağırır), `QUERY_TYPE_AFTER_FUNCTION` (sorgu bittikten sonra belirtilen fonksiyonu çağırır).
        *   **Sorgu Kimlikleri (`QID_*`):** Farklı `ReturnQuery` sonuçlarını ayırt etmek için kullanılır (örn. `QID_AUTH_LOGIN`, `QID_SAFEBOX_SIZE`, `QID_DB_STRING`, `QID_BLOCK_CHAT_LIST`, `QID_PCBANG_IP_LIST_CHECK`, `QID_SPAM_DB`).
    *   **`CQueryInfo` ve Türevleri:**
        *   `CReturnQueryInfo`: `ReturnQuery` için işlem türü (`iType`), hedef kimliği (`dwIdent`) ve ilişkili veriyi (`pvData`) tutar.
        *   `CFuncQueryInfo`: `FuncQuery` için çağrılacak fonksiyonu (`any_function f`) tutar.
        *   `CFuncAfterQueryInfo`: `FuncAfterQuery` için çağrılacak fonksiyonu (`any_void_function f`) tutar.
    *   **`DBManager` Sınıfı (Singleton):**
        *   Ana oyun veritabanı işlemleri için merkezi yönetici.
        *   `CAsyncSQL` (asenkron) ve `CAsyncSQL` (doğrudan, muhtemelen `CAsyncSQL2` olmalıydı) nesnelerini kullanarak veritabanı bağlantısını yönetir.
        *   **Metotlar:** `Connect`, `Query` (asenkron), `DirectQuery` (senkron), `ReturnQuery` (asenkron, sonuç özel olarak işlenir), `Process` (tamamlanan sorgu sonuçlarını işler), `AnalyzeReturnQuery` (ReturnQuery sonuçlarını QID'ye göre işler), `SendMoneyLog`, `LoginPrepare`, `SendAuthLogin`, `InsertLoginData`, `DeleteLoginData`, `GetLoginData`, `RequestBlockException`, `LoadDBString`, `GetDBString`, `GetGreetMessage`, `FuncQuery`, `FuncAfterQuery`, `EscapeString`.
    *   **`AccountDB` Sınıfı (Singleton):**
        *   Hesap veritabanı işlemleri için merkezi yönetici.
        *   `CAsyncSQL2` (hem doğrudan hem asenkron) nesnelerini kullanır.
        *   **Metotlar:** `Connect`, `ConnectAsync`, `DirectQuery`, `ReturnQuery`, `AsyncQuery`, `SetLocale`, `Process`, `AnalyzeReturnQuery` (özellikle spam DB yüklemesi için).
    *   **Diğer Yapılar:** `SUseTime`, `THighscoreRegisterQueryInfo`.
*   **Bağlantılı Dosyalar:** `db.cpp`, `../../libsql/AsyncSQL.h`, `any_function.h`, `login_data.h`.

### `debug_allocator_adapter.h`

*   **Amaç:** Gerçek bellek ayırma işini yapan bir alt allocator'ı (`Detail` şablon parametresi) sarmalayan ve bellek ayırma/bırakma işlemlerini takip ederek hata ayıklama yetenekleri ekleyen `DebugAllocatorAdapter` şablon sınıfını tanımlar. Sızıntıları, çift serbest bırakmaları ve potansiyel sınır aşımlarını tespit etmeye yardımcı olur.
*   **Temel İşlevler/İçerik:**
    *   **`AllocTag` Struct:** Bellek ayırma meta verilerini (dosya, satır, yaş (`age`), kullanım durumu (`in_use`)) tutar.
    *   **`ScopedOutputFile` Struct:** Log dosyalarını RAII prensibiyle açıp kapatan yardımcı sınıf.
    *   **`DebugAllocatorAdapter<Detail>` Sınıfı (Singleton):**
        *   `AllocMapType` (`unordered_map<void*, AllocTag>`) kullanarak ayrılan pointer'ları ve `AllocTag` bilgilerini takip eder.
        *   `StaticSetUp`/`StaticTearDown`: Adaptörü ve altındaki `Detail` allocator'ını başlatır/durdurur. `TearDown` sırasında `DumpLeakReport` çağırır.
        *   `Alloc`/`Free`: Ayırma/bırakma isteklerini `Detail`'e iletir.
        *   `MarkAcquired`: Bir pointer'ın ayrıldığını veya yeniden kullanıldığını kaydeder, `AllocTag` oluşturur/günceller ve yaşını döndürür.
        *   `MarkReleased`: Bir pointer'ın (veya `DebugPtr`'ın) serbest bırakıldığını, pointer'ı doğruladıktan sonra kaydeder ve `AllocTag`'ı günceller.
        *   `RetrieveAge`: Verilen bir pointer'ın mevcut yaşını döndürür.
        *   `Verify`: Bir pointer'ın geçerli olup olmadığını ve isteğe bağlı olarak yaşını (`age`) doğrular. Hataları loglar.
        *   `VerifyDeletion`: Bir pointer silinmeden hemen önce durumunu doğrular.
        *   `LogBoundaryCorruption`: Pointer'ın yaş bilgisinin beklenenden farklı olduğunu, olası bir sınır aşımını işaret ederek loglar.
        *   `DumpLeakReport`: Program sonlandığında hala `in_use` olarak işaretlenmiş pointer'ları (potansiyel sızıntılar) `dbgalloc_report.log` dosyasına yazar.
        *   `PrintStack` (isteğe bağlı, `#ifndef DBGALLOC_NO_STACKTRACE`): Hata durumlarında çağrı yığınını `dbgalloc.log` dosyasına yazar.
*   **Bağlantılı Dosyalar:** `debug_allocator.h` (bu dosyayı içerir), `ALLOCATOR_DETAIL_HEADER` ile belirtilen dosya, `debug_ptr.h` (kullanıyorsa), `<boost/unordered_map.hpp>` veya `<tr1/unordered_map>`, `<fstream>`, `<ctime>`.
*   **Not:** Bu dosya doğrudan değil, `debug_allocator.h` tarafından dahil edilmek üzere tasarlanmıştır.

### `debug_allocator.h`

*   **Amaç:** `DEBUG_ALLOC` önişlemci makrosu tanımlı olduğunda, C++ global bellek ayırma (`new`, `new[]`) ve serbest bırakma (`delete`, `delete[]`) operatörlerini, özel bir hata ayıklama allocator'ını (`DebugAllocator`, yani `DebugAllocatorAdapter`) kullanacak şekilde yeniden tanımlar. Dosya/satır bilgisiyle birlikte bellek işlemlerini kolaylaştırmak için `M2_NEW`/`M2_DELETE`/`M2_DELETE_ARRAY` makrolarını sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Koşullu Derleme (`#ifdef DEBUG_ALLOC`):** Tüm hata ayıklama mantığı bu makroya bağlıdır. Tanımlı değilse, standart `new`/`delete` kullanılır.
    *   **Allocator Seçimi:** `ALLOCATOR_DETAIL` ve `ALLOCATOR_DETAIL_HEADER` makroları ile hangi alt allocator implementasyonunun (`fifo_allocator.h` gibi) kullanılacağını belirler.
    *   **Adaptör Kullanımı:** `debug_allocator_adapter.h` dosyasını içerir ve `DebugAllocatorAdapter<ALLOCATOR_DETAIL>` için `DebugAllocator` typedef'ini oluşturur.
    *   **`DebugPtr` Entegrasyonu:** `debug_ptr.h` dosyasını içerir. `USE_DEBUG_PTR` makrosu tanımlıysa, ayrılan her bellek bloğunun başına bir "yaş" (`size_t`) değeri ekler.
    *   **Operatör Overload'ları:**
        *   `operator new(size_t, const char* file, size_t line)` / `operator new[](...)`: `DebugAllocator::Alloc` çağırır, `DebugAllocator::MarkAcquired` ile kaydeder. `USE_DEBUG_PTR` varsa yaş bilgisini bloğun başına yazar.
    *   **`debug_delete`/`debug_delete_array` Fonksiyonları:**
        *   `DebugAllocator::VerifyDeletion` ile doğrular.
        *   `debug_delete`: Nesne yıkıcısını çağırır (`p->~T()`).
        *   `debug_delete_array`: **Yıkıcıları çağırmaz!**
        *   `DebugAllocator::Free` çağırır (yaş başlığını dikkate alarak).
        *   `DebugAllocator::MarkReleased` ile serbest bırakıldığını işaretler.
        *   `DebugPtr` için overload'ları vardır (`USE_DEBUG_PTR` ise).
    *   **`M2_*` Makroları:** `M2_NEW`, `M2_DELETE`, `M2_DELETE_ARRAY` makroları, ilgili `new` operatörünü veya `debug_delete*` fonksiyonlarını `__FILE__` ve `__LINE__` bilgileriyle birlikte çağırır.
    *   **`M2_PTR_REF`/`M2_PTR_DEREF` Makroları:** `DebugPtr` kullanılıyorsa, pointer'a erişmeden önce yaşını doğrulamak için `DebugAllocator::Verify` çağırır.
*   **Bağlantılı Dosyalar:** `debug_allocator_adapter.h`, `debug_ptr.h`, `ALLOCATOR_DETAIL_HEADER` ile belirtilen dosya, `<cstdlib>`, `<new>`.

### `debug_ptr.h`

*   **Amaç:** `DEBUG_ALLOC` ve `USE_DEBUG_PTR` makroları tanımlı olduğunda, ham işaretçileri (`T*`) sarmalayan ve işaretçinin "yaşını" takip eden basit bir akıllı işaretçi benzeri `DebugPtr<T>` şablon sınıfını tanımlar. İşaretçiye erişmeden önce yaş kontrolü yaparak use-after-free gibi hataları tespit etmeye yardımcı olur.
*   **Temel İşlevler/İçerik:**
    *   **Koşullu Derleme (`#ifdef DEBUG_ALLOC`):** Bu başlığın içeriği sadece `DEBUG_ALLOC` tanımlıysa etkindir.
    *   **`DebugPtr<T>` Sınıfı:**
        *   `p_` (T*): Sarmalanan ham işaretçi.
        *   `age_` (size_t): İşaretçi ayrıldığında `DebugAllocator` tarafından atanan "yaş" değeri.
        *   **Yapıcılar:** Ham işaretçiden (yaşı `DebugAllocator::RetrieveAge` ile alır) veya başka `DebugPtr`'lardan oluşturulabilir.
        *   **`operator*`/`operator->`:** İşaretçiye erişmeden önce `GetVerified()` metodunu çağırır.
        *   **`Get()`/`GetAge()`:** Ham işaretçiyi ve saklanan yaşı döndürür.
        *   **`operator=`:** Atama işlemleri için.
        *   **`operator T*`:** Ham işaretçiye dönüşümü sağlar (doğrulama yapar).
        *   **`GetVerified()` (private):** En kritik metot. İşaretçiye erişimden önce çağrılır. Bellekte işaretçinin hemen önünde saklanan güncel yaş değerini okur (`*(reinterpret_cast<size_t*>(p) - 1)`). Bu değeri saklanan `age_` ile karşılaştırır. Uyuşmazlık durumunda hata loglar (`LogBoundaryCorruption`) veya durumu `DebugAllocator::Verify` ile tekrar kontrol eder ve ciddi bir sorun varsa programı çökertmeye çalışır.
    *   **Yardımcılar:**
        *   Karşılaştırma operatörleri (`==`, `!=`).
        *   `DebugPtr` için `static/const/dynamic_pointer_cast` fonksiyonları.
        *   `get_pointer` overload'u.
        *   `std::less`, `std::equal_to`, `std::hash` (veya `boost::hash`) uzmanlaşmaları (STL/Boost konteynerleriyle uyumluluk için).
*   **Bağlantılı Dosyalar:** `debug_allocator.h` (bu dosyayı içerir), `<algorithm>`.
*   **Not:** Bu dosya doğrudan değil, `debug_allocator.h` tarafından dahil edilmek üzere tasarlanmıştır ve `USE_DEBUG_PTR` makrosunun tanımlı olmasını gerektirir.

### `desc_client.h`

*   **Amaç:** Sunucunun başka bir sunucuya (DB sunucusu, Auth Master) istemci olarak bağlanmasını sağlayan `CLIENT_DESC` sınıfını tanımlar. `DESC` sınıfından türemiştir.
*   **Temel İşlevler/İçerik:**
    *   **`CLIENT_DESC` Sınıfı:**
        *   `DESC` sınıfından miras alır.
        *   `GetType()`: `DESC_TYPE_CONNECTOR` döndürür.
        *   **Metot Bildirimleri:** `Destroy`, `SetPhase`, `Connect`, `Setup`, `SetRetryWhenClosed`, `DBPacketHeader`, `DBPacket`, `Packet`, `IsRetryWhenClosed`, `Update`, `UpdateChannelStatus`, `Reset`.
        *   **Üyeler:** `m_iPhaseWhenSucceed` (başarılı bağlantı sonrası faz), `m_bRetryWhenClosed` (tekrar deneme bayrağı), zaman damgaları, `CInputDB m_inputDB` ve `CInputP2P m_inputP2P` (farklı fazlar için girdi işlemcileri).
    *   **Global Değişkenler:** `db_clientdesc` (DB sunucusuna bağlantı), `g_pkAuthMasterDesc` (Auth Master sunucusuna bağlantı).
*   **Bağlantılı Dosyalar:** `desc_client.cpp`, `desc.h`.

### `desc_manager.h`

*   **Amaç:** Sunucudaki tüm ağ bağlantı tanımlayıcılarını (`LPDESC`) merkezi olarak yöneten `DESC_MANAGER` singleton sınıfını tanımlar. Yeni bağlantıları kabul etme, mevcut bağlantıları bulma, kapatma, kullanıcı sayılarını takip etme, login anahtarlarını ve istemci paket şifreleme bilgilerini yönetme gibi işlevleri yerine getirir.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:** Deskriptörleri ve ilgili bilgileri saklamak için çeşitli konteyner türlerini tanımlar (`DESC_SET`, `CLIENT_DESC_SET`, `DESC_HANDLE_MAP`, `DESC_HANDSHAKE_MAP`, `DESC_LOGINNAME_MAP`, `DESC_HANDLE_RANDOM_KEY_MAP`).
    *   **`DESC_MANAGER` Sınıfı (Singleton):**
        *   **Yönetim Metotları:** `Initialize`, `Destroy`.
        *   **Bağlantı Kabul:** `AcceptDesc` (oyun istemcisi), `AcceptP2PDesc` (P2P).
        *   **Deskriptör Kapatma/Temizleme:** `DestroyDesc`, `DestroyClosed`.
        *   **Oluşturma:** `CreateHandshake` (istemci için benzersiz ID), `CreateConnectionDesc` (başka sunucuya bağlantı).
        *   **Bulma (`FindBy*`):** `FindByHandle`, `FindByHandshake`, `FindByCharacterName`, `FindByLoginName`, `FindByLoginKey`.
        *   **Hesap Yönetimi:** `ConnectAccount`, `DisconnectAccount` (login adı ile deskriptörü eşleştirme).
        *   **Bağlantı Denemesi:** `TryConnect` (kopan `CLIENT_DESC`'leri yeniden bağlamaya çalışır).
        *   **Kullanıcı Sayısı:** `UpdateLocalUserCount`, `GetUserCount`.
        *   **Anahtar Yönetimi:** `MakeRandomKey`, `GetRandomKey` (güvenlik için), `CreateLoginKey`, `ProcessExpiredLoginKey` (login işlemi için).
        *   **CRC Kontrolü:** `IsDisconnectInvalidCRC`, `SetDisconnectInvalidCRCMode`.
        *   **P2P Kontrolü:** `IsP2PDescExist`.
        *   **Client Package Cryptography:** `LoadClientPackageCryptInfo`, `SendClientPackageCryptKey`, `SendClientPackageSDBToLoadMap`.
        *   **Üyeler:** Deskriptörleri saklayan setler ve map'ler, sayaçlar, bayraklar ve `CClientPackageCryptInfo* m_pPackageCrypt`.
*   **Bağlantılı Dosyalar:** `desc_manager.cpp`, `desc.h`, `desc_p2p.h`, `desc_client.h`, `CLoginKey.h` (dolaylı), `CClientPackageCryptInfo.h`, `IFileMonitor.h`, `<boost/unordered_map.hpp>`, `../../common/stl.h`, `../../common/length.h`.

### `desc_p2p.h`

*   **Amaç:** Diğer sunuculardan gelen Peer-to-Peer (P2P) bağlantılarını temsil eden `DESC_P2P` sınıfını tanımlar. `DESC` sınıfından türemiştir.
*   **Temel İşlevler/İçerik:**
    *   `DESC_P2P` sınıf tanımı (`DESC`'den miras alır).
    *   Metot Bildirimleri: Yıkıcı (`~DESC_P2P`), `Destroy`, `SetPhase`, `Setup`.
*   **Bağlantılı Dosyalar:** `desc_p2p.cpp`, `desc.h`.

### `desc.h`

*   **Amaç:** Tüm ağ bağlantıları (oyun istemcileri, P2P, diğer sunucular) için temel sınıf olan `DESC`'i tanımlar. Soket yönetimi, bağlantı fazları, giriş/çıkış tamponları, handshake, paket şifreleme arayüzleri ve hesap/karakter ilişkilendirmesi gibi temel işlevleri içerir. Farklı fazlar için girdi işlemcilerini (`CInput*`) de tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`EDescType` Enum:** Bağlantı türünü belirtir (`DESC_TYPE_ACCEPTOR`, `DESC_TYPE_CONNECTOR`).
    *   **`CLoginKey` Sınıfı:** Geçici login anahtarlarını yönetir.
    *   **`DESC` Sınıfı:**
        *   **Sanal Metotlar:** `GetType`, `Destroy`, `SetPhase`.
        *   **Kurulum/Yıkım:** `Setup`, `Initialize`.
        *   **Bilgi Alma:** `GetSocket`, `GetHostName`, `GetPort`, `GetHandle`, `GetAccountTable`, `GetCharacter`, `GetAddr`, `GetUDPAddr`, `GetEmpire`, `GetLanguage`.
        *   **Paket İşleme:** `ProcessInput`, `ProcessOutput`, `Packet`, `BufferedPacket`, `LargePacket`, `RawPacket`, `ChatPacket`, `FlushOutput`.
        *   **Faz Yönetimi:** `IsPhase`.
        *   **Bağlama:** `BindAccountTable`, `BindCharacter`.
        *   **Handshake:** `StartHandshake`, `SendHandshake`, `HandshakeProcess`, `IsHandshaking`, `GetClientTime`.
        *   **Şifreleme:** Yeni (`Cipher`) veya eski (TEA) şifreleme için metotlar/arayüzler.
        *   **Yardımcılar:** `Log`, `UDPGrant`, `SetRelay`, `DelayedDisconnect`, `DisconnectOfSameLogin`, `SetAdminMode`, `IsAdminMode`, `SetPong`, `IsPong`, `SendLoginSuccessPacket`, `SendHWIDStatusPacket`, Anahtar Yönetimi, CRC Kontrolü, Client Versiyonu.
        *   **Üyeler:** Girdi işlemcileri, soket bilgileri, faz, handle, tamponlar, eventler, işaretçiler, adresler, bayraklar, anahtarlar, `Cipher` nesnesi vb.
*   **Bağlantılı Dosyalar:** `desc.cpp`, `constants.h`, `input.h`, `cipher.h` (opsiyonel).

### `dev_log.h`

*   **Amaç:** Yalnızca test sunucularında aktif olan, geliştiricilere özel bir loglama sistemi için seviye sabitlerini (bitmask) ve makroları (`LOG_*`) tanımlar. Ana loglama fonksiyonunu ve seviye kontrol fonksiyonlarını bildirir.
*   **Temel İşlevler/İçerik:**
    *   **Log Seviyesi Sabitleri:** `L_WARN`, `L_ERR`, `L_CRIT`, `L_INFO`, `L_MIN`, `L_MAX`, `L_LIB0`...`L_LIB3`, `L_DEB0`...`L_DEB3`, `L_USR0`...`L_USR3` (bit bayrakları).
    *   **`LOG_*` Makroları:** `dev_log` fonksiyonunu çağırmayı kolaylaştırır, otomatik olarak `__FILE__`, `__LINE__`, `__FUNCTION__` ve ilgili seviye sabitini argüman olarak ekler.
    *   **Fonksiyon Bildirimleri:** `dev_log`, `dev_log_add_level`, `dev_log_del_level`, `dev_log_set_level`.
*   **Bağlantılı Dosyalar:** `dev_log.cpp`.

## Kaynak Kod Dosyaları (.cpp)

### `cmd.cpp`

*   **Amaç:** Oyun sunucusunun komut yorumlayıcı mekanizmasını uygular ve tüm oyun içi komutların tanımlandığı merkezi `cmd_info` dizisini içerir.
*   **Temel İşlevler/İçerik:**
    *   **`ACMD` Fonksiyon Bildirimleri:** Diğer `cmd_*.cpp` dosyalarında tanımlanan tüm komut işleyici fonksiyonların (örn. `do_warp`, `do_item`, `do_mob`, `do_notice`) harici bildirimlerini içerir.
    *   **`cmd_info[]` Dizisi Tanımı:** Oyun motoru tarafından tanınan tüm komutları listeler. Her bir giriş şunları içerir:
        *   Komut metni (örn. "warp", "item", "notice").
        *   İlgili `ACMD` işleyici fonksiyonunun adresi.
        *   Alt komut numarası (SCMD).
        *   Gerekli minimum karakter pozisyonu (POS_*).
        *   Gerekli minimum GM seviyesi (GM_*).
        Bu dizi, komutların nasıl çağrılacağını ve kimlerin kullanabileceğini belirler.
    *   **`interpret_command(LPCHARACTER ch, const char* argument, size_t len)`:** Ana komut işleme fonksiyonu.
        *   Oyuncudan veya sistemden gelen komut metnini alır.
        *   Metni ayrıştırarak komut adını ve argümanları ayırır.
        *   `cmd_info` dizisinde komut adıyla eşleşen girdiyi arar.
        *   Eğer eşleşme bulunursa, karakterin komutu kullanmak için yeterli pozisyonda (`minimum_position`) ve yetkiye (`gm_level`) sahip olup olmadığını kontrol eder.
        *   Tüm kontroller başarılı olursa, `cmd_info` dizisinden alınan fonksiyon işaretçisini kullanarak ilgili `ACMD` fonksiyonunu çağırır ve gerekli parametreleri (karakter, argümanlar, cmd indeksi, subcmd) iletir.
        *   GM komutlarını loglar.
    *   **`interpreter_set_privilege(const char* cmd, int lvl)`:** Çalışma zamanında belirli bir komutun gerektirdiği GM seviyesini değiştirir.
    *   **`double_dollar(const char* src, ..., char* dest, ...)`:** Komut argümanlarındaki '$' karakterlerini '$$' olarak değiştirerek olası format string zafiyetlerini önlemek veya özel anlamı korumak için kullanılan bir metin işleme fonksiyonu.
*   **Çalışma Prensibi:** Oyuncu bir komut girdiğinde (`/warp oyuncu_adı` gibi), istemci bu metni sunucuya gönderir. Sunucu `interpret_command` fonksiyonunu çağırır. Bu fonksiyon, komutu (`warp`) `cmd_info` dizisinde arar, gerekli kontrolleri yapar ve eşleşen işleyici fonksiyonu (`do_warp`) uygun argümanlarla (`oyuncu_adı`) çağırır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `cmd.h`, `utils.h`, `config.h`, `char.h`, `

### `config.cpp`

*   **Amaç:** Sunucu yapılandırma dosyasını (`CONFIG` veya `CONFIG.<locale>`) okuyarak global değişkenleri başlatır, veritabanı bağlantılarını kurar ve sunucu başlangıcında gerekli diğer ayarları (IP tespiti, CRC listesi, komut yetkileri vb.) yükler.
*   **Temel İşlevler/İçerik:**
    *   **`config_init(const string& st_localeServiceName)`:**
        *   Sunucu başlatıldığında çağrılan ana yapılandırma fonksiyonudur.
        *   Yapılandırma dosyasını (`CONFIG` veya `CONFIG.<locale>`) açar.
        *   `GetIPInfo()` ile sunucunun public ve internal IP adreslerini tespit etmeye çalışır.
        *   Yapılandırma dosyasını satır satır okur, `parse_token` ile anahtar-değer çiftlerini ayırır.
        *   `TOKEN` makrosu ile anahtarları kontrol eder ve değerleri (`str_to_number`, `strlcpy` vb. kullanarak) `config.h`'de tanımlanan ilgili global değişkenlere atar (örn. `port`, `hostname`, `player_sql`, `map_allow`, `max_level`, `test_server`, `pk_protect_level`, `max_gold`, `view_range` ve diğer birçok özellik bayrağı).
        *   Veritabanı bağlantı bilgilerini (`player_sql`, `common_sql`, `log_sql`) okur ve `AccountDB`, `DBManager`, `LogManager` singleton'ları aracılığıyla ilgili veritabanlarına bağlanır.
        *   Common DB'den `locale` tablosunu okuyarak sunucunun yerelleştirme ayarını (`g_stLocale`) yapar ve `LocaleService_Init` fonksiyonunu çağırır.
        *   Common DB'den `locale` tablosundan `SKILL_POWER_BY_LEVEL` ve `SKILL_POWER_BY_LEVEL_TYPE<job>` verilerini okuyarak `CTableBySkill`'i doldurur.
        *   `LoadBanIP` ile yasaklı IP listesini yükler (eğer Auth sunucusu ise).
        *   `LocaleService_Load*` fonksiyonları ile çeşitli yerelleştirme dosyalarını (string, item, mob, skill isimleri vb.) yükler.
        *   `LoadValidCRCList` ile `CRC` dosyasından geçerli istemci CRC değerlerini yükler.
        *   `LoadStateUserCount` ile `state_user_count` dosyasından sunucu yoğunluk durumları için eşik değerleri yükler.
        *   `CWarMapManager::instance().LoadWarMapInfo` ile savaş haritası ayarlarını yükler.
        *   `CMD` dosyasını okuyarak komutların gerektirdiği GM seviyelerini `interpreter_set_privilege` ile ayarlar.
    *   **`GetIPInfo()`:** Sistemdeki ağ arayüzlerini tarayarak sunucunun public ve internal IP adreslerini bulmaya çalışır. `192.168.*` ve `10.*.*.*` adreslerini internal, diğerlerini public olarak varsayar.
    *   **`is_string_true()`:** Verilen metin değerinin boolean `true` olup olmadığını kontrol eder (sayısal olarak 0'dan büyükse veya 't' ile başlıyorsa true döner).
    *   **`map_allow_*` Fonksiyonları:** `map_allow` yapılandırma satırıyla belirtilen ve sunucuda izin verilen harita indekslerini `s_set_map_allows` setine ekler ve yönetir.
    *   **`FN_add_adminpageIP()`, `FN_log_adminpage()`:** Admin paneline erişim izni olan IP adreslerini `g_stAdminPageIP` vektörüne ekler.
    *   **`LoadValidCRCList()`:** `CRC` dosyasını okuyarak içindeki process ve dosya CRC değerlerini `s_set_dwProcessCRC` ve `s_set_dwFileCRC` setlerine yükler.
    *   **`IsValidProcessCRC()`, `IsValidFileCRC()`:** Verilen bir CRC değerinin yüklenen setlerde bulunup bulunmadığını kontrol eder.
    *   **`LoadClientVersion()`:** `VERSION` dosyasını okuyarak beklenen istemci versiyonunu `g_stClientVersion` değişkenine atar (Ancak kodda `g_stClientVersion`'ın başlangıç değeri üzerine yazılmıyor gibi duruyor, muhtemelen eski bir özellik).
    *   **`CheckClientVersion()`:** Bağlı istemcilerin sürümlerini kontrol eder (Aktif olarak çağrılmıyor gibi görünüyor).
    *   **`LoadStateUserCount()`:** `state_user_count` dosyasından sunucunun "Full" ve "Busy" durumlarına karşılık gelen kullanıcı sayılarını okur.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `constants.h`, `utils.h`, `log.h`, `db.h`, `locale_service.h`, `desc.h`, `desc_manager.h`, `p2p.h`, `char.h`, `ip_ban.h`, `war_map.h`, `skill_power.h`, `check_server.h` ve diğerleri.

### `crc32.cpp`

*   **Amaç:** `crc32.h`'de bildirilen CRC32 ve hızlı hash fonksiyonlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **`CRCTable[256]`:** CRC32 hesaplamasını hızlandırmak için kullanılan, önceden hesaplanmış değerleri içeren statik bir arama tablosu.
    *   **`GetCRC32(const char* buf, size_t len)`:**
        *   Standart CRC32 algoritmasını uygular.
        *   Verilen tamponu (`buf`) bayt bayt işler.
        *   Her bayt için, mevcut CRC değeri ve bayt değeri kullanılarak `CRCTable`'dan bir sonraki CRC değeri alınır ve CRC güncellenir.
        *   Performansı artırmak için `DO1`, `DO2`, `DO4`, `DO8`, `DO16` makroları ile döngü açma (loop unrolling) tekniği kullanılır.
        *   Sonuç olarak hesaplanan CRC değeri döndürülür.
    *   **`GetCaseCRC32(const char* buf, size_t len)`:**
        *   `GetCRC32`'ye benzer şekilde çalışır, ancak CRC hesaplaması yapmadan önce her karakteri `UPPER()` makrosu ile büyük harfe dönüştürür.
        *   Bu sayede büyük/küçük harf duyarsız bir CRC32 değeri elde edilir.
        *   Benzer şekilde `DO*CI` makroları ile döngü açma tekniği kullanılır.
    *   **`GetFastHash(const char* key, size_t len)`:**
        *   FNV-1a (Fowler–Noll–Vo) benzeri basit ve hızlı bir hash algoritması uygular.
        *   Verilen anahtarı (`key`) bayt bayt işler.
        *   Her bayt için hash değerini belirli bir sabit (`16777619`) ile çarpar ve ardından mevcut bayt ile XOR işlemine tabi tutar.
        *   Bu, CRC32'den farklı bir algoritmadır ve genellikle daha hızlıdır ancak veri bütünlüğü kontrolü için CRC32 kadar güvenilir olmayabilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `crc32.h`.

### `CsvReader.cpp`

*   **Amaç:** `CsvReader.h`'de bildirilen CSV işleme sınıflarının (`cCsvAlias`, `cCsvRow`, `cCsvFile`, `cCsvTable`) metotlarını uygular. Özellikle CSV dosyalarını okuma (`Load`) ve yazma (`Save`) sırasındaki ayrıştırma ve formatlama mantığını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Yardımcı Fonksiyonlar:**
        *   `Trim(std::string str)`: String'in başındaki ve sonundaki boşlukları (boşluk, tab, satır başı, yeni satır) kaldırır.
        *   `Lower(std::string original)`: String'i küçük harfe çevirir.
    *   **`cCsvAlias` Uygulaması:**
        *   `AddAlias`: İsmi küçük harfe çevirerek hem isim->indeks hem de indeks->isim haritalarına ekler.
        *   `operator[]`: İlgili haritalarda arama yaparak dönüşümü gerçekleştirir.
    *   **`cCsvFile::Load` Uygulaması:**
        *   Dosyayı `std::ifstream` ile açar.
        *   Satır satır okur (`getline`), boş satırları ve '#' ile başlayan yorum satırlarını atlar.
        *   Her satırı karakter karakter işler.
        *   Bir durum değişkeni (`ParseState state`) kullanarak normal mod (`STATE_NORMAL`) ve tırnak içi mod (`STATE_QUOTE`) arasında geçiş yapar.
        *   `STATE_QUOTE` modunda: Ardışık iki tırnak işaretini (`""`) tek bir tırnak karakteri olarak yorumlar. Tek bir tırnak işareti görünce `STATE_NORMAL` moduna döner. Tırnak içindeki ayırıcı karakterleri normal karakter gibi işler.
        *   `STATE_NORMAL` modunda: Ayırıcı karakter (varsayılan '') görünce o ana kadar biriken token'ı mevcut satıra (`cCsvRow`) ekler ve token'ı sıfırlar. Tırnak işareti (varsayılan ") görünce `STATE_QUOTE` moduna geçer. Diğer karakterleri token'a ekler.
        *   Satır sonunda kalan token'ı da satıra ekler.
        *   Okuma sırasında hata oluşursa (örn. dosya açılamazsa) `false` döndürür.
    *   **`cCsvFile::Save` Uygulaması:**
        *   Dosyayı `std::ofstream` ile açar (ekleme veya üzerine yazma modunda).
        *   Her satırı ve satırdaki her sütunu (token) işler.
        *   Eğer bir token özel karakterler (ayırıcı, tırnak, '\r', '\n') içeriyorsa:
            *   Token'ın başına ve sonuna tırnak işareti ekler.
            *   Token içindeki her tırnak işaretini iki tırnak işaretiyle (`""`) değiştirir.
        *   Eğer token özel karakter içermiyorsa, olduğu gibi yazar.
        *   Sütunların arasına ayırıcı karakteri ekler.
        *   Her satırın sonuna yeni satır karakteri ekler.
    *   **`cCsvTable` Uygulaması:** Metotları çoğunlukla `m_File` ve `m_Alias` nesnelerinin ilgili metotlarına çağrıları yönlendirir. `Next()` metodu mevcut satır indeksini (`m_CurRow`) artırır ve dosya sonuna gelinip gelinmediğini kontrol eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `CsvReader.h`, `<fstream>`, `<algorithm>`, `<map>`.\n\n

### `db.cpp`

*   **Amaç:** `db.h` içinde bildirilen `DBManager` ve `AccountDB` sınıflarının metotlarını uygular. Veritabanı bağlantılarını kurar, sorguları gönderir, asenkron sorgu sonuçlarını işler (login doğrulama, veri yükleme, highscore kaydı vb.) ve hesap veritabanı işlemlerini (özellikle spam filtresi) yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`DBManager` Uygulaması:**
        *   **`Connect`:** Hem asenkron (`m_sql`) hem de doğrudan (`m_sql_direct`) bağlantıları `CAsyncSQL::Setup` kullanarak kurar. Başarılı olursa `LoadDBString` çağırır.
        *   **`Query`, `DirectQuery`, `ReturnQuery`:** Sorgu string'lerini `va_list` ve `vsnprintf` ile oluşturur ve ilgili `CAsyncSQL` metoduna (`AsyncQuery`, `DirectQuery`, `ReturnQuery`) iletir. `ReturnQuery` için `CReturnQueryInfo` nesnesi oluşturulur.
        *   **`Process`:** `m_sql.PopResult` ile tamamlanmış sorgu sonuçlarını alır. Sonucun `pvUserData` alanındaki `CQueryInfo` işaretçisinin `iQueryType` değerine göre (`QUERY_TYPE_RETURN`, `QUERY_TYPE_FUNCTION`, `QUERY_TYPE_AFTER_FUNCTION`) ilgili işlemi yapar (fonksiyon çağırma veya `AnalyzeReturnQuery`'ye yönlendirme).
        *   **`AnalyzeReturnQuery`:** Asenkron `ReturnQuery` sonuçlarını işleyen ana `switch` bloğu. `qi->iType` (QID) değerine göre farklı mantıkları çalıştırır:
            *   `QID_AUTH_LOGIN`: Oyuncu login denemesini işler. Gelen kullanıcı adı/şifre ile veritabanını sorgular. Sonuçlara göre (kullanıcı var mı, şifre doğru mu, hesap durumu, dil seçimi, HWID ban durumu vb.) login işlemini başarılı (`LoginPrepare` çağrılır) veya başarısız (`LoginFailure` çağrılır) olarak sonuçlandırır. Brezilya (`LC_IsBrazil`) için özel olarak, hesap yoksa yeni hesap oluşturma sorgusu (`QID_BRAZIL_CREATE_ID`) gönderir.
            *   `QID_SAFEBOX_SIZE`: Oyuncunun depo boyutunu günceller.
            *   `QID_DB_STRING`: `string` tablosundan gelen verileri `m_map_dbstring` haritasına ve `m_vec_GreetMessage` vektörüne yükler.
            *   `QID_LOTTO`: Piyango sonucu gelen eşyayı (`pdw[0]`, `pdw[1]`) oyuncuya verir (`AutoGiveItem`) ve eşyanın soketlerine piyango ID (`uiInsertID`) ve zaman damgasını (`pdw[2]`) yazar.
            *   `QID_HIGHSCORE_REGISTER`: Highscore tablosundaki mevcut değeri kontrol eder ve eğer yeni değer daha iyiyse (`bOrder`'a göre) `REPLACE INTO` sorgusu gönderir.
            *   `QID_BLOCK_CHAT_LIST`: Engellenen oyuncuların listesini oyuncuya gönderir.
            *   `QID_PCBANG_IP_LIST_CHECK`, `QID_PCBANG_IP_LIST_SELECT`: `pcbang_ip` tablosunun güncellenip güncellenmediğini kontrol eder, güncellendiyse IP listesini `CPCBangManager`'a yükler.
            *   `QID_BRAZIL_CREATE_ID`: Brezilya için yeni hesap oluşturma sorgusunun sonucunu işler. Başarılıysa, normal `QID_AUTH_LOGIN` sorgusunu tekrar gönderir.
        *   **Login Yönetimi (`LoginPrepare`, `SendAuthLogin`, `SendLoginPing`, `InsertLoginData`, `DeleteLoginData`, `GetLoginData`):** Login verilerini (`CLoginData`) yönetir, login ping'lerini gönderir ve `db_clientdesc` üzerinden DB sunucusuna login/auth paketlerini iletir.
        *   **Diğer:** `SendMoneyLog`, `RequestBlockException`, `EscapeString`.
    *   **`AccountDB` Uygulaması:**
        *   `Connect`, `ConnectAsync`: `CAsyncSQL2::Setup` kullanarak hesap veritabanına bağlanır.
        *   `DirectQuery`, `AsyncQuery`, `ReturnQuery`: Sorguları `CAsyncSQL2` üzerinden gönderir.
        *   `Process`: Tamamlanan sorgu sonuçlarını alır ve `AnalyzeReturnQuery`'ye iletir.
        *   `AnalyzeReturnQuery`: Hesap veritabanı sorgu sonuçlarını işler.
            *   `QID_SPAM_DB`: `spam_db` tablosundan gelen spam kelimelerini ve skorlarını okuyarak `SpamManager`'ı günceller.
        *   **Spam DB Yenileme (`LoadSpamDB`, `reload_spam_event`, `s_pkReloadSpamEvent`):** Belirli aralıklarla (`g_uiSpamReloadCycle`) `QID_SPAM_DB` sorgusunu tekrar göndermek için bir event (`reload_spam_event`) tanımlar ve kullanır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `db.h`, `config.h`, `desc_client.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `item.h`, `item_manager.h`, `p2p.h`, `log.h`, `login_data.h`, `locale_service.h`, `pcbang.h`, `spam.h`, `auth_brazil.h`, `../../common/length.h`, `<sstream>`.

### `desc_client.cpp`

*   **Amaç:** `desc_client.h`'de tanımlanan `CLIENT_DESC` sınıfının metotlarını uygular. Diğer sunuculara bağlantı kurma, bağlantı durumunu yönetme, veri gönderme ve alma (farklı fazlara göre) işlemlerini gerçekleştirir.
*   **Temel İşlevler/İçerik:**
    *   **`Destroy()`:** Bağlantıyı kapatır, `fdwatch`'tan çıkarır, P2P kaydını siler. Eğer DB bağlantısıysa (`db_clientdesc`), partileri ve lonca savaşlarını temizler.
    *   **`Connect(int iPhaseWhenSucceed)`:** Belirtilen host ve porta `socket_connect` ile bağlanmayı dener. Başarılı olursa `fdwatch`'a ekler ve belirtilen faza geçer. Başarısız olursa `PHASE_CLIENT_CONNECTING` fazına geçer. 3 saniyeden kısa aralıklarla tekrar denemeyi engeller.
    *   **`Setup(LPFDWATCH _fdw, const char* _host, WORD _port)`:** Bağlantı hedefini (`host`, `port`) ve kullanılacak `fdwatch` nesnesini ayarlar. Giriş/çıkış tamponlarını (`InitializeBuffers`) oluşturur.
    *   **`SetPhase(int iPhase)`:** Deskriptörün fazını değiştirir ve faza özgü başlangıç işlemlerini yapar:
        *   `PHASE_CLIENT_CONNECTING`: Girdi işlemcisini sıfırlar.
        *   `PHASE_DBCLIENT`: Girdi işlemcisini `m_inputDB` olarak ayarlar. Sunucunun Auth sunucusu olup olmamasına göre DB sunucusuna `HEADER_GD_BOOT` ve `HEADER_GD_SETUP` paketlerini göndererek sunucu yapılandırmasını (IP, portlar, harita izinleri, bağlı oyuncu bilgileri vb.) iletir. `CPartyManager::EnablePCParty()` çağırır.
        *   `PHASE_P2P`: Girdi işlemcisini `m_inputP2P` olarak ayarlar, tamponları sıfırlar.
        *   `PHASE_CLOSE`: Girdi işlemcisini sıfırlar.
    *   **`DBPacketHeader(BYTE bHeader, DWORD dwHandle, DWORD dwSize)`/`DBPacket(...)`:** DB sunucusuna veya benzer protokol kullanan sunuculara (Auth Master) özel formatta (Header, Handle, Size, Data) paketleri çıkış tamponuna yazar.
    *   **`Packet(...)`:** Genel veri gönderme fonksiyonu, veriyi doğrudan çıkış tamponuna yazar.
    *   **`IsRetryWhenClosed()`:** Eğer sunucu kapanmıyorsa ve `m_bRetryWhenClosed` bayrağı true ise true döndürür.
    *   **`Update(DWORD t)`:** Periyodik olarak `UpdateChannelStatus` fonksiyonunu çağırır (eğer Auth sunucusu değilse).
    *   **`UpdateChannelStatus(DWORD t, bool fForce)`:** Mevcut sunucudaki kullanıcı sayısını (`DESC_MANAGER::GetUserCount`) alır, sunucu durumunu (boş, normal, yoğun, dolu) belirler ve bu bilgiyi `HEADER_GD_UPDATE_CHANNELSTATUS` paketi ile DB sunucusuna gönderir. 5 dakikada bir veya `fForce` true ise çalışır.
    *   **`Reset()`:** Bağlantıyı yeniden kurmak amacıyla deskriptörü temizler (`Destroy`) ve yeniden başlatır (`Initialize`, `InitializeBuffers`). Bağlantı hedef bilgileri korunur.
    *   **`InitializeBuffers()`:** Giriş ve çıkış tamponlarını 1MB boyutunda oluşturur.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `utils.h`, `desc_client.h`, `desc_manager.h`, `char.h`, `protocol.h`, `p2p.h`, `buffer_manager.h`, `guild_manager.h`, `db.h`, `party.h`.

### `desc_manager.cpp`

*   **Amaç:** `desc_manager.h`'de tanımlanan `DESC_MANAGER` singleton sınıfının metotlarını uygular. Yeni bağlantıları kabul etme, mevcut bağlantıları bulma, kapatma, kullanıcı sayılarını takip etme, login anahtarlarını ve istemci paket şifreleme bilgilerini yönetme gibi işlevleri yerine getirir.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:** Deskriptörleri ve ilgili bilgileri saklamak için çeşitli konteyner türlerini tanımlar (`DESC_SET`, `CLIENT_DESC_SET`, `DESC_HANDLE_MAP`, `DESC_HANDSHAKE_MAP`, `DESC_LOGINNAME_MAP`, `DESC_HANDLE_RANDOM_KEY_MAP`).
    *   **`DESC_MANAGER` Sınıfı (Singleton):**
        *   **Yönetim Metotları:** `Initialize`, `Destroy`.
        *   **Bağlantı Kabul:** `AcceptDesc` (oyun istemcisi), `AcceptP2PDesc` (P2P).
        *   **Deskriptör Kapatma/Temizleme:** `DestroyDesc`, `DestroyClosed`.
        *   **Oluşturma:** `CreateHandshake` (istemci için benzersiz ID), `CreateConnectionDesc` (başka sunucuya bağlantı).
        *   **Bulma (`FindBy*`):** `FindByHandle`, `FindByHandshake`, `FindByCharacterName`, `FindByLoginName`, `FindByLoginKey`.
        *   **Hesap Yönetimi:** `ConnectAccount`, `DisconnectAccount` (login adı ile deskriptörü eşleştirme).
        *   **Bağlantı Denemesi:** `TryConnect` (kopan `CLIENT_DESC`'leri yeniden bağlamaya çalışır).
        *   **Kullanıcı Sayısı:** `UpdateLocalUserCount`, `GetUserCount`.
        *   **Anahtar Yönetimi:** `MakeRandomKey`, `GetRandomKey` (güvenlik için), `CreateLoginKey`, `ProcessExpiredLoginKey` (login işlemi için).
        *   **CRC Kontrolü:** `IsDisconnectInvalidCRC`, `SetDisconnectInvalidCRCMode`.
        *   **P2P Kontrolü:** `IsP2PDescExist`.
        *   **Client Package Cryptography:** `LoadClientPackageCryptInfo`, `SendClientPackageCryptKey`, `SendClientPackageSDBToLoadMap`.
        *   **Üyeler:** Deskriptörleri saklayan setler ve map'ler, sayaçlar, bayraklar ve `CClientPackageCryptInfo* m_pPackageCrypt`.
*   **Bağlantılı Dosyalar:** `desc_manager.cpp`, `desc.h`, `desc_p2p.h`, `desc_client.h`, `CLoginKey.h` (dolaylı), `CClientPackageCryptInfo.h`, `IFileMonitor.h`, `<boost/unordered_map.hpp>`, `../../common/stl.h`, `../../common/length.h`.

### `desc_p2p.cpp`

*   **Amaç:** `desc_p2p.h`'de bildirilen `DESC_P2P` sınıfının metotlarını uygular. Diğer sunuculardan gelen P2P bağlantılarının kurulumunu, faz yönetimini ve temizlenmesini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`Destroy()`:** Bağlantıyı sonlandırır. Soketi kapatır, `fdwatch`'tan kaldırır ve deskriptörü `P2P_MANAGER::instance().UnregisterAcceptor()` ile P2P yöneticisinden siler. Temel sınıfın `Destroy` metodunu çağırır.
    *   **`Setup(LPFDWATCH fdw, socket_t fd, const char* host, WORD wPort)`:** Yeni kabul edilen P2P bağlantısı için deskriptör bilgilerini (fdwatch, soket, host, port) ayarlar. 1MB boyutunda giriş ve çıkış tamponları oluşturur (`buffer_new`). Soketi `fdwatch_add_fd` ile okuma için izlemeye alır. Opsiyonel olarak (`__PORT_SECURITY__` tanımlıysa) gelen bağlantının IP adresinin sunucunun kendi public IP'si (`g_szPublicIP`) ile eşleşip eşleşmediğini kontrol eder; eşleşmezse bağlantıyı kapatır (`SetPhase(PHASE_CLOSE)`). Başarılı olursa fazı `PHASE_P2P` olarak ayarlar.
    *   **`SetPhase(int iPhase)`:** Deskriptörün işlem fazını ayarlar.
        *   `PHASE_P2P`: Girdi işlemcisini statik `CInputP2P` nesnesi (`s_inputP2P`) olarak ayarlar ve tamponları sıfırlar.
        *   `PHASE_CLOSE`: Girdi işlemcisini `NULL` yapar.
        *   Diğer durumlar için hata loglar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `desc_p2p.h`, `protocol.h`, `p2p.h`, `../../common/service.h`, `config.h`.

### `desc.cpp`

*   **Amaç:** `desc.h`'de tanımlanan temel `DESC` sınıfının metotlarını uygular. Ağ bağlantılarının yaşam döngüsünü, giriş/çıkış işlemlerini, faz geçişlerini, handshake mantığını, şifreleme/çözme işlemlerini (TEA veya Cipher), ping/pong mekanizmasını ve diğer yardımcı işlevleri yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`Initialize()`:** Tüm üye değişkenleri varsayılan değerlere sıfırlar.
    *   **`Destroy()`:** Bağlantıyı temizler (event iptali, karakter bağlantısı kesme, tampon silme, logout paketi gönderme, soketi kapatma, şifreleme nesnesini temizleme).
    *   **`ping_event()`:** Periyodik ping gönderir ve pong yanıtını kontrol eder, yanıt yoksa bağlantıyı keser.
    *   **`Setup()`:** Yeni bağlantıyı başlatır (soket, adres, handle ayarla), tamponları oluşturur, `fdwatch`'a ekler, ping event'ini başlatır, ilk handshake'i gönderir ve fazı `PHASE_HANDSHAKE` yapar.
    *   **`ProcessInput()`:** Soketten veri okur, gerekirse şifreyi çözer (`Cipher::Decrypt` veya `TEA_Decrypt`) ve ilgili `m_pInputProcessor` ile işler.
    *   **`ProcessOutput()`:** Çıkış tamponundaki veriyi sokete yazar, kısmi yazmaları yönetir.
    *   **`Packet()`/`BufferedPacket()`/`LargePacket()`:** Paketleri gönderir, gerektiğinde tamponlar ve şifreler (`Cipher::Encrypt` veya `TEA_Encrypt`).
    *   **`SetPhase()`:** Bağlantı fazını değiştirir, istemciye bildirir ve doğru `m_pInputProcessor`'ü ayarlar.
    *   **`HandshakeProcess()`:** İstemci ile zaman senkronizasyonunu sağlar, hataları ve zaman aşımlarını yönetir.
    *   **Şifreleme Metotları:** Yeni şifreleme için anahtar anlaşmasını veya eski TEA için anahtar ayarlamasını yönetir.
    *   **Diğer Metotlar:** Hesap/karakter bağlama, UDP adresi yönetimi, loglama, P2P relay, gecikmeli bağlantı kesme, admin modu, anahtar yönetimi, CRC birleştirme, yardımcı paket göndericiler (`ChatPacket`, `SendLoginSuccessPacket` vb.).
*   **Bağlantılı Dosyalar:** Çok sayıda başlık dosyası içerir (`stdafx.h`, `config.h`, `desc.h`, `desc_client.h`, `desc_manager.h`, `char.h`, `protocol.h`, `packet.h`, `cipher.h`, `p2p.h`, `db.h`, `log.h` vb.).

### `dev_log.cpp`

*   **Amaç:** `dev_log.h`'de bildirilen geliştirici loglama fonksiyonlarını uygular. Yalnızca test sunucularında (`test_server == true`) ve aktif log seviyeleri (`s_log_mask`) dahilinde `DEV_LOG.log` dosyasına detaylı (zaman, seviye, dosya, satır, fonksiyon, mesaj) loglama yapar.
*   **Temel İşlevler/İçerik:**
    *   **`dev_log(...)`:**
        *   `test_server` ve `s_log_mask` kontrolü.
        *   `DEV_LOG.log` dosyasını açar.
        *   Mikrosaniye hassasiyetinde zamanı alır ve formatlar.
        *   Log seviyesini metne çevirir.
        *   Formatlanmış log mesajını (zaman, seviye, dosya, satır, fonksiyon, mesaj) oluşturur.
        *   Mesajı dosyaya yazar ve dosyayı kapatır.
    *   **`dev_log_add/del/set_level()`:** Statik `s_log_mask` değişkenini değiştirerek hangi log seviyelerinin aktif olacağını kontrol eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `dev_log.h`.

### `empire_text_convert.h`

*   **Amaç:** Farklı imparatorluklar arasındaki metin iletişimini karıştırmak/dönüştürmek için kullanılan fonksiyonları bildirir. Bu, "İmparatorluk Dili" özelliğinin temelini oluşturur.
*   **Temel İşlevler/İçerik:**
    *   `LoadEmpireTextConvertTable(DWORD dwEmpireID, const char* c_szFileName)`: Belirtilen imparatorluk ID'si için metin dönüştürme tablosunu (`.dat` dosyası) yükler.
    *   `ConvertEmpireText(DWORD dwEmpireID, char* szText, size_t len, int iPct)`: Verilen metni (`szText`), hedef imparatorluğun (`dwEmpireID`) kurallarına ve belirtilen olasılık yüzdesine (`iPct`) göre dönüştürür.
*   **Bağlantılı Dosyalar:** `empire_text_convert.cpp`.

### `empire_text_convert.cpp`

*   **Amaç:** `empire_text_convert.h`'de bildirilen imparatorluk metin dönüştürme fonksiyonlarını uygular. Dönüştürme tablolarını bellekte tutar ve karakter bazında dönüştürme işlemini gerçekleştirir.
*   **Temel İşlevler/İçerik:**
    *   **`STextConvertTable` Struct:** Her imparatorluk için büyük harf, küçük harf, Hangul, Jaum ve Moum karakter eşlemelerini tutan veri yapısı.
    *   **`g_aTextConvTable[3]`:** Üç imparatorluk için dönüştürme tablolarını saklayan global dizi.
    *   **`LoadEmpireTextConvertTable`:** Belirtilen `.dat` dosyasını okur ve verileri doğrudan `g_aTextConvTable` içindeki ilgili imparatorluğun `STextConvertTable` yapısına yükler.
    *   **`ConvertEmpireText`:**
        *   Metni karakter karakter dolaşır.
        *   `iPct` olasılığına göre karakteri dönüştürüp dönüştürmeyeceğine karar verir.
        *   Karakter çok baytlı ise (Korece/Çince): `g_iUseLocale` değişkenine bağlı olarak ya sabit kodlanmış Çince karakterlerle ya da yüklenmiş Hangul/Jaum/Moum tablolarındaki karşılıklarıyla değiştirir.
        *   Karakter tek baytlı ise (İngilizce harf): Yüklenmiş büyük/küçük harf tablolarındaki karşılığıyla değiştirir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `empire_text_convert.h`. (Dolaylı olarak `locale_service.h` ve `utils.h` içindeki `g_iUseLocale` ve `number` fonksiyonuna bağlıdır).

### `fifo_allocator.h`

*   **Amaç:** `debug_allocator` sistemi için alternatif bir arka uç bellek ayırıcı (`FifoAllocator`) tanımlar. Sık kullanılan küçük bellek blokları için basit bir FIFO (İlk Giren İlk Çıkar) havuzlama mekanizması kullanarak performansı artırmayı hedefler.
*   **Temel İşlevler/İçerik:**
    *   **`FifoAllocator` Sınıfı:**
        *   Farklı boyutlardaki serbest bırakılmış bellek bloklarını saklamak için bir harita (`PoolMapType`) kullanır. Her boyut için bir `std::deque<void*>` (havuz) tutar.
        *   Ayrılmış blokları ve boyutlarını takip etmek için ayrı bir harita (`AllocMapType`) kullanır.
        *   `Alloc(size_t size)`: İstenen boyutta bellek ayırır. İlgili havuzdaki blok sayısı belirli bir eşikten (`kWatermark`) azsa `::malloc` kullanır, aksi takdirde havuzun başından bir blok alır.
        *   `Free(void* p)`: Bloğu ilgili havuzun sonuna ekler.
        *   `TearDown()`: Uygulama sonunda tüm havuzlardaki blokları `::free` ile serbest bırakır.
*   **Kullanım:** Genellikle doğrudan kullanılmaz, `debug_allocator.h` içinde `ALLOCATOR_DETAIL` olarak tanımlanarak hata ayıklama bellek yöneticisinin davranışını değiştirir.
*   **Bağlantılı Dosyalar:** `<deque>`, `<unordered_map>` (Boost veya TR1), `debug_allocator.h`.

### `file_loader.h`

*   **Amaç:** Metin dosyalarını bellekten okumak ve satırları token'lara ayırmak için `CMemoryTextFileLoader` sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CMemoryTextFileLoader` Sınıfı:**
        *   `Bind(int bufSize, const void* c_pvBuf)`: Verilen bellek tamponundan dosya içeriğini yükler ve satırlara ayırır.
        *   `GetLineCount()`: Toplam satır sayısını döndürür.
        *   `CheckLineIndex(DWORD dwLine)`: Satır indeksinin geçerliliğini kontrol eder.
        *   `SplitLine(DWORD dwLine, std::vector<std::string>* pstTokenVector, const char* c_szDelimeter)`: Belirtilen satırı ayırıcılara göre token'lara böler (tırnak işaretlerini dikkate alır).
        *   `GetLineString(DWORD dwLine)`: Belirtilen satırın tamamını string olarak döndürür.
        *   `m_stLineVector` (protected): Yüklenen dosyanın satırlarını tutan vektör.
*   **Bağlantılı Dosyalar:** `file_loader.cpp`, `<vector>`, `<string>`.

### `file_loader.cpp`

*   **Amaç:** `CMemoryTextFileLoader` sınıfının metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **`Bind`:** Bellek tamponunu satır sonu karakterlerine (`\n`, `\r`) göre böler ve `m_stLineVector`'ü doldurur. Çok baytlı karakterleri işler.
    *   **`SplitLine`:** Satırı, belirtilen ayırıcılara göre böler. `"` karakterleri arasındaki metni tek bir token olarak kabul eder. `#` ile başlayan yorum satırlarını (eğer `#--#` değilse) atlar.
    *   Diğer metotlar (`GetLineCount`, `CheckLineIndex`, `GetLineString`) `m_stLineVector` üzerinde temel işlemler yapar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `file_loader.h`.

### `FileMonitor_FreeBSD.h`

*   **Amaç:** FreeBSD işletim sistemi için `IFileMonitor` arayüzünü uygulayan `FileMonitorFreeBSD` singleton sınıfını tanımlar. Dosya sistemi değişikliklerini (silme, yazma, yeniden adlandırma vb.) izlemek için FreeBSD'nin `kqueue` kernel olay bildirim mekanizmasını kullanır.
*   **Koşullu Derleme:** Yalnızca Windows dışı sistemlerde (FreeBSD hedefli) derlenir.
*   **Temel İşlevler/İçerik:**
    *   **`FileIOContext_FreeBSD` Struct:** İzlenen dosyanın tanıtıcısını (`fhMonitor`), olay listelerindeki indeksini ve değişiklik algılandığında çağrılacak geri arama fonksiyonunu (`pListenFunc`) tutar.
    *   **`FileMonitorFreeBSD` Sınıfı (Singleton):**
        *   `AddWatch`: Bir dosyayı izleme listesine ekler, `kqueue`'yu ayarlar.
        *   `Update`: `kevent` ile bekleyen olayları kontrol eder, algılanan değişiklikler için geri arama fonksiyonlarını çağırır.
    *   **Üyeler:** İzlenen dosyaların haritası (`m_FileLists`), `kevent` için olay listeleri (`m_MonitoredEventLists`, `m_TriggeredEventLists`), `kqueue` tanıtıcısı (`m_KernelEventQueue`).
*   **Bağlantılı Dosyalar:** `FileMonitor_FreeBSD.cpp`, `IFileMonitor.h`, FreeBSD sistem başlıkları (`unistd.h`, `sys/event.h` vb.), `boost/unordered_map.hpp`.

### `FileMonitor_FreeBSD.cpp`

*   **Amaç:** `FileMonitorFreeBSD` sınıfının metotlarını uygular.
*   **Koşullu Derleme:** Muhtemelen sadece FreeBSD üzerinde derlenir.
*   **Temel İşlevler/İçerik:**
    *   **`AddWatch` Uygulaması:** `open` ile dosyayı açar, `kqueue()` ile olay kuyruğunu oluşturur, `kevent` yapısını `EVFILT_VNODE`, `EV_ADD | EV_ENABLE | EV_ONESHOT` ve ilgili `NOTE_*` bayrakları ile doldurarak dosyayı izlemeye alır.
    *   **`Update` Uygulaması:** `kevent()` fonksiyonunu çağırarak olayları alır. Gelen olayların bayraklarını (`flags`) yorumlayarak değişiklik türünü belirler (`eFileUpdatedOptions`) ve ilgili geri arama fonksiyonunu çağırır.
    *   **Yıkıcı:** Açık dosya tanıtıcılarını ve `kqueue` tanıtıcısını kapatır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `FileMonitor_FreeBSD.h`, `log.h`.

### `fonks.py`

*   **Amaç:** Belirtilen bir C++ kaynak dosyasındaki (`char.cpp` olarak sabit kodlanmış) fonksiyon tanımlarını veya bildirimlerini düzenli ifadeler kullanarak çıkarmayı amaçlayan bir Python betiğidir. Geliştirme sırasında kod analizi veya dokümantasyon oluşturma gibi amaçlarla kullanılmış olabilir. **Çalışan oyun sunucusunun bir parçası değildir.**
*   **Temel İşlevler/İçerik:**
    *   `extract_function_names(file_path)` fonksiyonunu tanımlar.
    *   Verilen dosya yolunu okur.
    *   C++ fonksiyon imzalarını eşleştirmeye çalışan bir düzenli ifade (`function_pattern`) kullanır.
    *   `re.findall` ile eşleşen tüm fonksiyon imzalarını bulur ve bunları konsola yazdırır.
*   **Kullanım:** Geliştiriciler tarafından belirli bir C++ dosyasındaki fonksiyonları listelemek için kullanılan bir yardımcı araçtır.
*   **Bağlantılı Dosyalar:** Yok (Python betiği).

### `game.vcxproj`

*   **Amaç:** Microsoft Visual Studio'nun MSBuild sistemi için Metin2 `game` sunucusu yürütülebilir dosyasının derleme yapılandırmasını tanımlayan bir XML dosyasıdır. Projenin nasıl derleneceğini, hangi dosyaların dahil edileceğini, hangi kütüphanelerin bağlanacağını ve derleyici/bağlayıcı ayarlarının ne olacağını gibi bilgileri içerir.
*   **Temel İşlevler/İçerik:**
    *   **Yapılandırmalar:** Debug ve Release modları için ayrı ayarlar tanımlar.
    *   **Platform:** Genellikle Win32 (x86) hedefini belirtir.
    *   **Genel Ayarlar:** Proje GUID'si, adı, karakter seti gibi temel bilgileri içerir.
    *   **Dizinler:** Çıktı (`.exe`) ve ara derleme dosyalarının (`.obj`) yerlerini belirtir.
    *   **Derleyici Ayarları (`ClCompile`):** Optimizasyon seviyeleri, ek başlık dosyası yolları (`AdditionalIncludeDirectories`), önişlemci tanımları (`PreprocessorDefinitions`), çalışma zamanı kütüphanesi (`RuntimeLibrary`), C++ standardı (`LanguageStandard`) gibi ayarları içerir.
    *   **Bağlayıcı Ayarları (`Link`):** Bağlanacak ek kütüphaneleri (`AdditionalDependencies`, örn. `cryptlib`, `mysqlclient`), kütüphane arama yollarını (`AdditionalLibraryDirectories`), çıktı dosyasının adını (`OutputFile`), alt sistemi (`SubSystem`) gibi ayarları içerir.
    *   **Dosya Listeleri (`ItemGroup`):** Projeye dahil edilen tüm `.cpp` ve `.h` dosyalarını listeler.
    *   **Proje Referansları (`ProjectReference`):** `game` projesinin bağımlı olduğu diğer Visual Studio projelerini (örn. `libthecore`, `libsql`) belirtir.
*   **Not:** Bu dosya doğrudan oyun mantığını içermez, sadece derleme sürecini yönetir. CMake (`CMakeLists.txt`) kullanılıyorsa, bu dosya genellikle CMake tarafından otomatik olarak oluşturulur veya yerini alır.
*   **Bağlantılı Dosyalar:** Yok (Yapılandırma dosyası).

### `gm.h`

*   **Amaç:** Oyun Yöneticisi (GM) yetki seviyelerini ve erişim kontrolünü yönetmek için kullanılan fonksiyonların harici bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   `gm_get_level(const char* name, const char* host = NULL, const char* account = NULL)`: Bir oyuncunun adını, isteğe bağlı olarak host IP'sini ve hesap adını alarak GM seviyesini (`GM_*` sabitleri) döndüren ana fonksiyon.
    *   `gm_new_clear()`: Bellekteki GM listesini temizler.
    *   `gm_new_insert(const tAdminInfo& c_rInfo)`: `common/tables.h` içindeki `tAdminInfo` yapısını kullanarak yeni bir GM kaydı ekler.
    *   `gm_new_host_inert(const char* host)`: Genel GM host listesine bir IP adresi ekler.
    *   (Eski) `gm_insert`, `gm_host_insert` fonksiyon bildirimleri.
*   **Bağlantılı Dosyalar:** `gm.cpp`, `constants.h`.

### `gm.cpp`

*   **Amaç:** `gm.h`'de bildirilen GM yönetim fonksiyonlarını uygular. GM'lerin adlarını, yetki seviyelerini ve izin verilen IP adreslerini/hesaplarını saklar ve sorgular.
*   **Temel İşlevler/İçerik:**
    *   **Veri Saklama:** GM bilgilerini `g_map_GM` (isim -> `tGM`) map'inde ve genel host IP'lerini `g_set_Host` set'inde tutar.
    *   **GM Ekleme (`gm_new_insert`):** Veritabanından (`common.admin`) gelen `tAdminInfo` verisini kullanarak GM'i haritaya ekler. Eğer GM için özel bir `ContactIP` belirtilmemişse, genel host listesini (`g_set_Host`) kullanır.
    *   **Seviye Sorgulama (`gm_new_get_level`):**
        *   Test sunucusuysa doğrudan en yüksek yetkiyi verir.
        *   Verilen isimle GM'i haritada arar.
        *   **Lokasyona Göre Kontrol:**
            *   Avrupa/Singapur (Tayvan hariç): Sadece hesap adını kontrol eder, host IP'sini **kontrol etmez**.
            *   Diğer lokasyonlar: Host IP'sini kontrol eder (ya genel listede ya da GM'in özel `ContactIP`'si ile eşleşmeli).
        *   Tüm kontroller geçerse, GM'in yetki seviyesini döndürür, aksi takdirde `GM_PLAYER` döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `gm.h`, `locale_service.h`.

### `group_text_parse_tree.h`

*   **Amaç:** Hiyerarşik metin dosyalarını (örneğin, `group` anahtar kelimesi ve süslü parantezlerle yapılandırılmış) ayrıştırmak ve temsil etmek için kullanılan sınıfları (`CGroupNode`, `CGroupNode::CGroupNodeRow`, `CGroupTextParseTreeLoader`) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CGroupNodeRow`:** Bir veri satırını temsil eder (string vektörü). `GetValue` şablonları ile verilere tür dönüşümü yaparak erişim sağlar.
    *   **`CGroupNode`:** Ağaç yapısındaki bir düğümü (grubu) temsil eder. Alt düğümleri (`m_mapChildNodes`), anahtarlı satırları (`m_map_rows`), sütun isimlerini (`m_map_columnNameToIndex`) tutar. Alt düğümlere, satırlara ve satır içindeki değerlere (anahtar veya indeksle) erişim metotları sunar.
    *   **`CGroupTextParseTreeLoader`:** Ayrıştırma işlemini yönetir. `CMemoryTextFileLoader` kullanır, `Load` ile dosyayı okur, `GetGroup` ile üst seviye gruplara erişim sağlar.
    *   **`from_string` Şablonu:** String'leri farklı türlere dönüştürmek için yardımcı fonksiyon.
*   **Bağlantılı Dosyalar:** `group_text_parse_tree.cpp`, `file_loader.h`, `../../common/stl.h`, `<sstream>`.

### `group_text_parse_tree.cpp`

*   **Amaç:** `group_text_parse_tree.h`'de bildirilen sınıfların metotlarını uygular. Metin dosyasını ayrıştıran çekirdek mantığı içerir.
*   **Temel İşlevler/İçerik:**
    *   **`CGroupTextParseTreeLoader::Load`:** Dosyayı belleğe yükler ve `LoadGroup` ile ayrıştırmayı başlatır.
    *   **`CGroupTextParseTreeLoader::LoadGroup` (Recursive):**
        *   Satır satır okur.
        *   `{` ve `}` ile grup sınırlarını belirler.
        *   `group <group_name>` satırlarında alt `CGroupNode` oluşturur ve özyinelemeli çağrı yapar.
        *   `#--# <col_name>...` satırlarında sütun isimlerini ve indekslerini kaydeder.
        *   Diğer satırları `<key> <value>...` olarak ayrıştırır ve `CGroupNodeRow` olarak `m_map_rows`'a ekler.
        *   Grup ve anahtar isimlerini küçük harfe çevirir.
    *   **`CGroupNode` Metotları:** İlgili map'ler üzerinde arama ve iterasyon yaparak veri erişimini sağlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `group_text_parse_tree.h`, `../../common/stl.h`.

### `IFileMonitor.h`

*   **Amaç:** Dosya sistemi değişikliklerini izlemek için soyut bir arayüz (`IFileMonitor`) tanımlar. İşletim sistemine özgü implementasyonlar (örn. `FileMonitorFreeBSD.h`) için temel yapı görevi görür.
*   **Temel İşlevler/İçerik:**
    *   **`eFileUpdatedOptions` Enum'u:** İzlenebilecek dosya olay türlerini (Silindi, Değiştirildi, Yeniden Adlandırıldı vb.) tanımlar.
    *   **`PFN_FileChangeListener` Typedef'i:** Dosya değişikliği algılandığında çağrılacak geri arama fonksiyonunun (statik fonksiyon işaretçisi) türünü tanımlar.
    *   **`IFileMonitor` Struct (Arayüz):**
        *   `virtual void Update(DWORD dwPulses) = 0;`: Bekleyen dosya olaylarını kontrol etmek ve işlemek için periyodik olarak çağrılması gereken saf sanal fonksiyon.
        *   `virtual void AddWatch(const std::string& strFileName, PFN_FileChangeListener pListenerFunc) = 0;`: Belirtilen dosyayı izleme listesine ekleyen ve değişiklik durumunda çağrılacak dinleyici fonksiyonunu kaydeden saf sanal fonksiyon.
*   **Bağlantılı Dosyalar:** `<boost/unordered_map.hpp>`. (`.cpp` dosyası yoktur, sadece arayüz tanımlar.)

### `input_auth.cpp`

*   **Amaç:** Sunucu `g_bAuthServer` bayrağı ile bir kimlik doğrulama sunucusu olarak çalıştığında istemcilerden gelen ilk giriş (login) isteklerini işlemekten sorumludur. Temel amacı, istemci bağlantılarını kabul etmek, giriş bilgilerini doğrulamak ve oyun sunucusuna geçiş öncesindeki el sıkışma (handshake) sürecini yönetmektir.
*   **Temel İşlevler ve Sınıflar:**
    *   **`CInputAuth` Sınıfı**
        *   **`CInputAuth()` (Yapıcı)**
            *   Sınıf örneği oluşturulduğunda çağrılır. Özel bir başlatma yapmaz.
        *   **`Login(LPDESC d, const char* c_pData)`**
            *   İstemciden gelen `HEADER_CG_LOGIN3` paketi ile tetiklenir.
            *   Amacı: İstemcinin gönderdiği giriş bilgilerini (kullanıcı adı, şifre, istemci anahtarları) işlemek ve veritabanında doğrulamaktır.
            *   İşleyiş:
                1.  Sunucunun kimlik doğrulama sunucusu olup olmadığını kontrol eder (`g_bAuthServer`). Değilse, istemciyi atar.
                2.  Giriş bilgilerini (`login`, `passwd`) paketten alır ve temizler.
                3.  `FN_IS_VALID_LOGIN_STRING` ile kullanıcı adının geçerli karakterler içerip içermediğini kontrol eder.
                4.  Sunucu `g_bNoMoreClient` (daha fazla istemci kabul etmeme) modundaysa "SHUTDOWN" hatası gönderir.
                5.  `DESC_MANAGER` aracılığıyla aynı kullanıcı adıyla zaten bağlı bir istemci olup olmadığını kontrol eder; varsa "ALREADY" hatası gönderir.
                6.  `DESC_MANAGER::instance().CreateLoginKey(d)` ile istemci için benzersiz bir giriş anahtarı oluşturur.
                7.  `d->SetPanamaKey()` ile istemci ve sunucu arasında kullanılacak bir şifreleme anahtarı (Panama Key) ayarlar.
                8.  Eğer Brezilya yereli aktifse (`LC_IsBrazil()`) ve test sunucusu değilse, `auth_brazil(login, passwd)` fonksiyonu ile ek Brezilya'ya özgü kimlik doğrulama adımlarını çalıştırır.
                9.  Giriş bilgilerini SQL enjeksiyonuna karşı güvenli hale getirmek için `DBManager::instance().EscapeString` kullanır.
                10. `Login_IsInChannelService` ile girişin bir kanal hizmeti girişi olup olmadığını kontrol eder (genellikle `[` ile başlayan kullanıcı adları).
                11. `DBManager::instance().ReturnQuery(QID_AUTH_LOGIN, ...)` ile veritabanına asenkron bir sorgu göndererek hesap bilgilerini (şifre, sosyal ID, hesap ID, durum, premium süreleri vb.) talep eder. Şifre karşılaştırması (SHA1 veya düz metin) bu sorgu içinde yapılır.
        *   **`Analyze(LPDESC d, BYTE bHeader, const char* c_pData)`**
            *   Kimlik doğrulama (AUTH) aşamasındaki istemcilerden gelen paketleri analiz eden ana fonksiyondur.
            *   Amacı: Gelen paketin başlığına (`bHeader`) göre ilgili işleyici fonksiyonu çağırmaktır.
            *   İşleyiş:
                1.  Yine `g_bAuthServer` kontrolü yapar.
                2.  `switch (bHeader)` yapısı ile paketleri yönlendirir:
                    *   `HEADER_CG_PONG`: `Pong(d)` fonksiyonunu çağırarak istemcinin hayatta olduğunu teyit eder. (Pong fonksiyonu `input.cpp` içinde tanımlıdır ancak burada çağrılır)
                    *   `HEADER_CG_LOGIN3`: `Login(d, c_pData)` fonksiyonunu çağırarak giriş işlemini başlatır.
                    *   `HEADER_CG_HANDSHAKE`: Bu paket için özel bir işlem yapılmaz, boş bırakılmıştır. Muhtemelen el sıkışma sürecinin bir parçasıdır ve bu aşamada ek bir sunucu taraflı mantık gerektirmez.
                    *   Bilinmeyen bir başlık gelirse hata kaydı (`sys_err`) oluşturulur.
            *   Paket verisinin ne kadarının işlendiğini belirten `iExtraLen` değerini döndürür (bu implementasyonda hep 0).
        *   **Yardımcı Fonksiyonlar**
            *   **`FN_IS_VALID_LOGIN_STRING(const char* str)`**
                *   Amacı: Verilen bir kullanıcı adı string'inin geçerli karakterlerden oluşup oluşmadığını kontrol etmektir.
                *   İşleyiş: String boş veya çok kısaysa (`< 2` karakter) geçersiz sayılır. Ardından her karakteri kontrol eder; sayı veya harf ise geçerlidir. Bölgeye (`LC_IsCanada()`, `LC_IsYMIR()`, `LC_IsKorea()`, `LC_IsBrazil()`, `LC_IsJapan()`) göre izin verilen ek özel karakterler vardır (örn: `_`, `-`, `.`, `@`). Bunların dışında bir karakter varsa string geçersizdir.
            *   **`Login_IsInChannelService(const char* c_login)`**
                *   Amacı: Bir kullanıcı adının özel bir kanal hizmetine ait olup olmadığını belirlemektir.
                *   İşleyiş: Kullanıcı adının ilk karakteri `[` ise, kanal hizmeti girişi olarak kabul edilir.
        *   **Bağlantılı Dosyalar ve Bağımlılıklar**
            *   **`stdafx.h`**: Temel başlık dosyası.
            *   **`constants.h`**: Sabit değerler (örn: `LOGIN_MAX_LEN`, `PASSWD_MAX_LEN`).
            *   **`config.h`**: Sunucu yapılandırma bayrakları (örn: `g_bAuthServer`, `test_server`, `g_bNoMoreClient`).
            *   **`input.h`**: Temel girdi işleme sınıfları ve `Pong` gibi fonksiyonlar için.
            *   **`desc_client.h`**: `LPDESC` (istemci tanımlayıcı) yapısı için.
            *   **`desc_manager.h`**: `DESC_MANAGER` (istemci tanımlayıcı yöneticisi) sınıfı için (login anahtarı oluşturma, bağlı kullanıcı kontrolü).
            *   **`protocol.h`**: Ağ paket başlıkları (`HEADER_CG_LOGIN3`, `HEADER_GC_LOGIN_FAILURE` vb.) ve paket yapıları (`TPacketCGLogin3`) için.
            *   **`locale_service.h`**: Bölgeye özgü işlevler (`LC_IsCanada`, `LC_IsBrazil` vb.) için.
            *   **`auth_brazil.h`**: Brezilya'ya özgü kimlik doğrulama fonksiyonu `auth_brazil` için.
            *   **`db.h`**: `DBManager` (veritabanı yöneticisi) sınıfı ve sorgu ID'leri (`QID_AUTH_LOGIN`) için.
            *   Temel C/C++ kütüphaneleri: `string.h` (`strlen`, `strlcpy`), `ctype.h` (`isdigit`, `isalpha`).
        *   **Çalışma Prensibi**
            1.  İstemci, sunucunun kimlik doğrulama portuna bağlanır.
            2.  Bağlantı kabul edildiğinde, istemci için bir `LPDESC` oluşturulur ve `PHASE_AUTH` aşamasına geçer.
            3.  İstemci `HEADER_CG_LOGIN3` paketini kullanıcı adı, şifre ve istemci anahtarları ile gönderir.
            4.  `CInputAuth::Analyze` bu paketi alır ve `CInputAuth::Login` fonksiyonuna yönlendirir.
            5.  `Login` fonksiyonu temel kontrolleri yapar (geçerli kullanıcı adı, sunucu durumu, zaten bağlı olup olmadığı).
            6.  Bir login anahtarı ve Panama anahtarı oluşturulur.
            7.  Gerekirse bölgeye özgü kimlik doğrulama yapılır (örn: Brezilya).
            8.  Veritabanına `QID_AUTH_LOGIN` sorgusu gönderilerek hesap bilgileri talep edilir.
            9.  Veritabanından cevap (`HEADER_DG_LOGIN_SUCCESS`, `HEADER_DG_LOGIN_NOT_EXIST` vb.) `input_db.cpp` tarafından işlenir ve istemciye sonuç (örn: `HEADER_GC_AUTH_SUCCESS`) gönderilir. Başarılı olursa istemci bir sonraki aşamaya (genellikle karakter seçimi) geçer.
        *   Bu dosya, oyunun giriş sürecinin ilk ve kritik bir adımını yönetir ve yalnızca kimlik doğrulama sunucusu rolündeyken aktiftir.

### `input_db.cpp`

*   **Amaç:** Sunucu `g_bAuthServer` bayrağı ile bir kimlik doğrulama sunucusu olarak çalıştığında istemcilerden gelen ilk giriş (login) isteklerini işlemekten sorumludur. Temel amacı, istemci bağlantılarını kabul etmek, giriş bilgilerini doğrulamak ve oyun sunucusuna geçiş öncesindeki el sıkışma (handshake) sürecini yönetmektir.
*   **Temel İşlevler ve Sınıflar:**
    *   **`CInputDB` Sınıfı**
        *   **`CInputDB()` (Yapıcı)**
            *   Sınıf örneği oluşturulduğunda çağrılır. Özel bir başlatma yapmaz.
        *   **`Login(LPDESC d, const char* c_pData)`**
            *   İstemciden gelen `HEADER_CG_LOGIN3` paketi ile tetiklenir.
            *   Amacı: İstemcinin gönderdiği giriş bilgilerini (kullanıcı adı, şifre, istemci anahtarları) işlemek ve veritabanında doğrulamaktır.
            *   İşleyiş:
                1.  Sunucunun kimlik doğrulama sunucusu olup olmadığını kontrol eder (`g_bAuthServer`). Değilse, istemciyi atar.
                2.  Giriş bilgilerini (`login`, `passwd`) paketten alır ve temizler.
                3.  `FN_IS_VALID_LOGIN_STRING` ile kullanıcı adının geçerli karakterler içerip içermediğini kontrol eder.
                4.  Sunucu `g_bNoMoreClient` (daha fazla istemci kabul etmeme) modundaysa "SHUTDOWN" hatası gönderir.
                5.  `DESC_MANAGER` aracılığıyla aynı kullanıcı adıyla zaten bağlı bir istemci olup olmadığını kontrol eder; varsa "ALREADY" hatası gönderir.
                6.  `DESC_MANAGER::instance().CreateLoginKey(d)` ile istemci için benzersiz bir giriş anahtarı oluşturur.
                7.  `d->SetPanamaKey()` ile istemci ve sunucu arasında kullanılacak bir şifreleme anahtarı (Panama Key) ayarlar.
                8.  Eğer Brezilya yereli aktifse (`LC_IsBrazil()`) ve test sunucusu değilse, `auth_brazil(login, passwd)` fonksiyonu ile ek Brezilya'ya özgü kimlik doğrulama adımlarını çalıştırır.
                9.  Giriş bilgilerini SQL enjeksiyonuna karşı güvenli hale getirmek için `DBManager::instance().EscapeString` kullanır.
                10. `Login_IsInChannelService` ile girişin bir kanal hizmeti girişi olup olmadığını kontrol eder (genellikle `[` ile başlayan kullanıcı adları).
                11. `DBManager::instance().ReturnQuery(QID_AUTH_LOGIN, ...)` ile veritabanına asenkron bir sorgu göndererek hesap bilgilerini (şifre, sosyal ID, hesap ID, durum, premium süreleri vb.) talep eder. Şifre karşılaştırması (SHA1 veya düz metin) bu sorgu içinde yapılır.
        *   **`Analyze(LPDESC d, BYTE bHeader, const char* c_pData)`**
            *   Kimlik doğrulama (AUTH) aşamasındaki istemcilerden gelen paketleri analiz eden ana fonksiyondur.
            *   Amacı: Gelen paketin başlığına (`bHeader`) göre ilgili işleyici fonksiyonu çağırmaktır.
            *   İşleyiş: Büyük bir `switch` ifadesi kullanarak `bHeader` değerine göre ilgili `CInputDB` metodunu çağırır. Örneğin:
                *   `HEADER_DG_BOOT`: `Boot(c_pData)` - Sunucu başlangıç verilerini işler.
                *   `HEADER_DG_LOGIN_SUCCESS`: `LoginSuccess(m_dwHandle, c_pData)` - Başarılı giriş yapmış bir hesabın bilgilerini işler.
                *   `HEADER_DG_PLAYER_LOAD_SUCCESS`: `PlayerLoad(m_dwHandle, c_pData)` - Oyuncu karakter verilerini yükler.
                *   `HEADER_DG_ITEM_LOAD`: `ItemLoad(m_dwHandle, c_pData)` - Oyuncunun eşyalarını yükler.
                *   `HEADER_DG_QUEST_LOAD`: `QuestLoad(m_dwHandle, c_pData)` - Oyuncunun görev (quest) bayraklarını yükler.
                *   `HEADER_DG_SAFEBOX_LOAD`: `SafeboxLoad(m_dwHandle, c_pData)` - Oyuncunun deposunu (safebox) yükler.
                *   `HEADER_DG_GUILD_LOAD`: `GuildLoad(c_pData)` - Lonca bilgilerini yükler.
                *   `HEADER_DG_RELOAD_PROTO`: `ReloadProto(c_pData)` - Oyun içi prototipleri (mob, item vb.) yeniden yükler.
                *   Ve daha birçok farklı paket başlığı için özel işleyici metodlar bulunur.
            *   Bilinmeyen bir başlık gelirse hata kaydı (`sys_err`) oluşturulur.
            *   Başarılı işlemde 0, bilinmeyen paket durumunda -1 döndürür.
        *   **Paket İşleyici Metodları (Örnekler)**
            *   **`Boot(const char* data)`**: Bu, oyun sunucusu başlatıldığında DB sunucusundan gelen ilk ve en önemli paketlerden biridir.
                *   Amacı: Oyun için gerekli olan tüm temel prototip verilerini (moblar, eşyalar, dükkanlar, yetenekler, arındırma tarifleri, eşya efsunları, yasaklı kelimeler, binalar, objeler vb.), global zamanı, eşya ID aralıklarını, GM listelerini ve hükümdar bilgilerini yüklemek.
                *   İşleyiş:
                    1.  Paket versiyonunu kontrol eder.
                    2.  Sırayla `TMobTable`, `TItemTable`, `TShopTable`, `TSkillTable`, `TRefineTable`, `TItemAttrTable` gibi tabloları okur ve ilgili yönetici sınıfların (`CMobManager`, `ITEM_MANAGER`, `CShopManager`, `CSkillManager`, `CRefineManager` vb.) `Initialize` metodlarını çağırarak verileri yükler.
                    3.  Bina ve obje sistemini (`building::CManager`) başlatır.
                    4.  Global zamanı ayarlar (`set_global_time`).
                    5.  Eşya ID aralıklarını `ITEM_MANAGER`'a bildirir.
                    6.  GM listelerini ve hostlarını yükler.
                    7.  Hükümdar bilgilerini `CMonarch`'a yükler.
                    8.  Paketin sonunda bir kontrol değeri (0xffff) olup olmadığını doğrular.
                    9.  `LocaleService` kullanarak bölgeye özel dosyaları (drop item dosyaları, harita indeksleri, Dragon Soul tabloları vb.) yükler.
                    10. Küp sistemini, hareket yöneticisini (`CMotionManager`), PCBang sistemini ve kale sistemini (`castle_boot`) başlatır.
            *   **`LoginSuccess(DWORD dwHandle, const char* data)`**:
                *   DB sunucusundan başarılı bir hesap girişi bilgisi geldiğinde tetiklenir.
                *   `TAccountTable` yapısını alır.
                *   İlgili `LPDESC`'i `dwHandle` ile bulur.
                *   Hesap durumunu kontrol eder ("OK" olmalı).
                *   `GetServerLocation` ile oyuncunun karakterlerinin hangi sunucuda olduğunu belirler.
                *   `d->BindAccountTable(pTab)` ile hesap bilgilerini `LPDESC`'e bağlar.
                *   İstemciye imparatorluk bilgisini (`HEADER_GC_EMPIRE`) ve giriş başarı paketini (`SendLoginSuccessPacket`) gönderir.
                *   İstemciyi `PHASE_SELECT` (karakter seçimi) aşamasına geçirir.
            *   **`PlayerLoad(LPDESC d, const char* data)`**:
                *   Oyuncu karakter seçimi yaptığında ve DB'den karakter verileri geldiğinde tetiklenir.
                *   `TPlayerTable` yapısını alır.
                *   Harita ve koordinat bilgilerini doğrular, gerekirse başlangıç noktasına ayarlar.
                *   `CHARACTER_MANAGER::instance().CreateCharacter` ile yeni bir `CHARACTER` nesnesi oluşturur.
                *   Karakteri `LPDESC`'e bağlar (`ch->BindDesc(d)`, `d->BindCharacter(ch)`).
                *   Karakterin temel verilerini (`SetPlayerProto`), imparatorluğunu ayarlar.
                *   P2P sistemi üzerinden diğer sunuculara giriş bilgisini gönderir.
                *   Giriş loglarını kaydeder.
                *   İstemciyi `PHASE_LOADING` aşamasına geçirir ve temel karakter paketlerini (`MainCharacterPacket`, `PointsPacket`, `SkillLevelPacket`) gönderir.
                *   Görevleri (`quest::CQuestManager`) ve hızlı erişim slotlarını yükler.
            *   **`ItemLoad(LPDESC d, const char* c_pData)`**: Oyuncunun envanterindeki, ekipmanındaki ve diğer slotlarındaki eşyaları DB'den gelen verilere göre oluşturur ve karaktere ekler.
            *   **`QuestLoad(LPDESC d, const char* c_pData)`**: Oyuncunun görev ilerlemesini (quest flag'lerini) yükler.
            *   **`GuildLoad(const char* c_pData)`**, **`PartyCreate(const char* c_pData)`** gibi fonksiyonlar ilgili sistemlerin (lonca, grup) DB'den gelen verilerle güncellenmesini sağlar.
            *   **`ReloadProto(const char* c_pData)`**: `/reload p` gibi bir komutla tetiklendiğinde DB'den gelen güncel prototip (item, mob, skill vb.) tablolarını yükleyerek sunucuyu yeniden başlatmadan değişikliklerin aktif olmasını sağlar.
            *   Diğer birçok fonksiyon, evlilik, hükümdarlık, özel dükkanlar, posta kutusu gibi çeşitli oyun özellikleriyle ilgili DB yanıtlarını işler.
        *   **Bağlantılı Dosyalar ve Bağımlılıklar**
            *   **`protocol.h`**: Tüm `HEADER_DG_*` (Database-Game) ve `HEADER_GC_*` (Game-Client) paket başlıklarını ve ilgili veri yapılarını (`TPlayerTable`, `TItemTable` vb.) içerir.
            *   **Yönetici Sınıfları**: `CHARACTER_MANAGER`, `ITEM_MANAGER`, `MOB_MANAGER`, `SHOP_MANAGER`, `GUILD_MANAGER`, `PARTY_MANAGER`, `QUEST_MANAGER`, `SECTREE_MANAGER`, `CPrivManager`, `CMonarch`, `marriage::CManager` vb. DB'den gelen verilerle bu yöneticiler güncellenir.
            *   **`desc_manager.h`**: `DESC_MANAGER` ile `dwHandle` üzerinden `LPDESC` bulmak için kullanılır.
            *   **`char.h`**: `LPCHARACTER` nesnelerine veri yüklemek için.
            *   **`item.h`**: `LPITEM` nesneleri oluşturmak ve yüklemek için.
            *   **`db.h`**: `db_clientdesc` (DB sunucusuna olan bağlantı) için.
            *   **`config.h`**: `g_bAuthServer`, `test_server` gibi yapılandırma bayrakları için.
            *   **`locale_service.h`**: Bölgesel ayar dosyalarını yüklemek için.
            *   **`log.h`**: `LogManager` ile çeşitli loglamalar yapmak için.
            *   Ve daha birçok özellik (`DragonSoul.h`, `MailBox.h`, `private_shop_manager.h` vb.) için ilgili başlık dosyaları.
        *   **Çalışma Prensibi**
            1.  Oyun sunucusu, `db_clientdesc` üzerinden DB sunucusuna bir istek paketi (örn: `HEADER_GD_PLAYER_LOAD`) gönderir.
            2.  DB sunucusu bu isteği işler ve cevabını (örn: `HEADER_DG_PLAYER_LOAD_SUCCESS` ve `TPlayerTable` verisi) oyun sunucusuna geri gönderir.
            3.  Oyun sunucusunun ana döngüsünde, `db_clientdesc`'ten gelen veriler okunur ve `CInputDB::Process` fonksiyonuna iletilir.
            4.  `Process`, gelen veri akışını mantıksal paketlere böler.
            5.  Her paket için `CInputDB::Analyze` çağrılır.
            6.  `Analyze`, paketin başlığına göre ilgili `CInputDB` metodunu (örn: `CInputDB::PlayerLoad`) çağırır.
            7.  Bu metod, paketten gelen veriyi kullanarak oyun içi durumu (karakterler, eşyalar, loncalar vb.) günceller, ilgili yönetici sınıfları bilgilendirir ve gerekirse istemciye (`LPDESC` üzerinden) güncel bilgileri içeren paketler gönderir.
            8.  Sunucu başlangıcında (`Boot` fonksiyonu), DB sunucusu proaktif olarak büyük bir veri paketi gönderir ve `CInputDB` bu veriyi işleyerek sunucunun çalışır duruma gelmesi için gerekli tüm temel tanımları yükler.