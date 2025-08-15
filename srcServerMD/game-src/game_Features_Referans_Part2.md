# Metin2 Oyun Sunucusu - Özellikler Referansı Part 2 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans.md`](game_Features_Referans.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) belirli oyun özellikleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [Header Dosyaları (.h)](#header-dosyaları-h)
    *   [`ClientPackageCryptInfo.h`](#clientpackagecryptinfoh)
    *   [`cube.h`](#cubeh)
    *   [`CubeManager.h`](#cubemanagerh)
    *   [`cuberenewal.h`](#cuberenewalh)
    *   [`dragon_soul_table.h`](#dragon_soul_tableh)
    *   [`DragonLair.h`](#dragonlairh)
    *   [`DragonSoul.h`](#dragonsoulh)
*   [Kaynak Kod Dosyaları (.cpp)](#kaynak-kod-dosyaları-cpp)
    *   [`ClientPackageCryptInfo.cpp`](#clientpackagecryptinfocpp)
    *   [`cmd_oxevent.cpp`](#cmd_oxeventcpp)
    *   [`cube.cpp`](#cubecpp)
    *   [`CubeManager.cpp`](#cubemanagercpp)
    *   [`cuberenewal.cpp`](#cuberenewalcpp)
    *   [`dragon_soul_table.cpp`](#dragon_soul_tablecpp)
    *   [`DragonLair.cpp`](#dragonlaircpp)
    *   [`DragonSoul.cpp`](#dragonsoulcpp)

---

## Header Dosyaları (.h)

### `ClientPackageCryptInfo.h`

*   **Amaç:** İstemci paketleri (`.epk`/`.eix`) için şifreleme anahtarlarını ve ek veri bloklarını (SDB - Supplementary Data Blocks) yöneten `CClientPackageCryptInfo` sınıfını tanımlar. Bu veriler, istemci tarafından paket bütünlüğünü doğrulamak veya belirli içerikleri (genellikle haritaya özgü) çözmek için kullanılır. SDB verilerini tutan `TSupplementaryDataBlockInfo` yapısını da içerir.
*   **Temel İşlevler/İçerik:**
    *   **`TSupplementaryDataBlockInfo` Struct:** Paket ve dosya tanımlayıcılarını (`dwPackageIdentifier`, `dwFileIdentifier`) ve SDB veri akışını (`vecSDBStream`) tutar. Veriyi istemciye göndermek için serileştirme (`Serialize`) ve boyut hesaplama (`GetSerializedSize`) metotları sağlar.
    *   **`CClientPackageCryptInfo` Sınıfı:**
        *   `LoadPackageCryptInfo(const char* pCryptInfoDir)`: Belirtilen dizindeki tüm kripto dosyalarını (`cshybridcrypt*`) yükler.
        *   `GetPackageCryptKeys(BYTE** ppData, int& iDataSize)`: Tüm yüklenmiş şifreleme anahtarlarının serileştirilmiş akışını (önbelleğe alınmış) döndürür.
        *   `GetRelatedMapSDBStreams(const char* pMapName, BYTE** ppData, int& iDataSize)`: Belirtilen (küçük harfe çevrilmiş) harita adıyla ilişkili tüm SDB'lerin serileştirilmiş akışını (önbelleğe alınmış) döndürür.
        *   `LoadPackageCryptFile(const char* pCryptFile)` (private): Tek bir kripto dosyasını yükler.
        *   Üyeler: Anahtar verilerini (`m_vecPackageCryptKeys`), harita bazlı SDB bilgilerini (`m_mapPackageSDB`), toplam anahtar paket sayısını (`m_nCryptKeyPackageCnt`) ve serileştirilmiş önbellekleri (`m_pSerializedCryptKeyStream`, `TPerFileSDBInfo::m_pSerializedStream`) tutar.
*   **Bağlantılı Dosyalar:** `ClientPackageCryptInfo.cpp`, `<unordered_map>`, `<vector>`, `<string>`.

### `cube.h`

*   **Amaç:** Eşya birleştirme/oluşturma sistemi olan Küp (Cube) ile ilgili sabitleri, veri yapılarını ve fonksiyon bildirimlerini tanımlar. Bu başlık dosyası, `#if !defined(__CUBE_RENEWAL__)` bloğu içinde yer aldığı için muhtemelen eski veya belirli bir versiyona ait Küp sistemini temsil etmektedir.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler:**
        *   `CUBE_MAX_NUM`: Küp penceresine konulabilecek maksimum eşya sayısı (24).
        *   `CUBE_MAX_DISTANCE`: Küpü kullanmak için NPC'ye olan maksimum uzaklık (1000).
    *   **`CUBE_VALUE` Struct:**
        *   Bir eşya türünü (VNUM) ve miktarını (`count`) temsil eder.
        *   Malzeme ve ödül listelerinde kullanılır.
        *   Eşitlik karşılaştırması için `operator==` tanımlanmıştır.
    *   **`CUBE_DATA` Struct:**
        *   Tek bir Küp tarifini tanımlar:
            *   `npc_vnum` (`std::vector<WORD>`): Tarifi gerçekleştirebilen NPC VNUM'ları.
            *   `item` (`std::vector<CUBE_VALUE>`): Gerekli malzemeler.
            *   `reward` (`std::vector<CUBE_VALUE>`): Üretim sonucu elde edilebilecek olası ödüller.
            *   `percent` (int): Başarı şansı (1-100).
            *   `gold` (unsigned int): Gerekli altın miktarı.
            *   `not_remove` (DWORD): Üretim sonrası silinmeyecek özel bir malzemenin VNUM'u (varsa).
        *   Metotlar:
            *   `can_make_item(LPITEM* items, WORD npc_vnum)`: Verilen eşyalar ve NPC ile bu tarifin yapılıp yapılamayacağını kontrol eder.
            *   `reward_value()`: Ödül listesinden rastgele bir ödül seçer (Not: Implementasyonu şüpheli olabilir).
            *   `remove_material(LPCHARACTER ch)`: Gerekli malzemeleri karakterin küp penceresinden siler.
    *   **Fonksiyon Bildirimleri:**
        *   `Cube_init()`: Küp sistemini başlatır, tarifleri yükler.
        *   `Cube_load(const char* file)`: Belirtilen dosyadan küp tariflerini yükler.
        *   `Cube_make(LPCHARACTER ch)`: Karakter için küp üretim işlemini gerçekleştirir.
        *   `Cube_open(LPCHARACTER ch)`, `Cube_close(LPCHARACTER ch)`: Küp penceresini açar/kapatır.
        *   `Cube_add_item`, `Cube_delete_item`, `Cube_clean_item`, `Cube_show_list`: Küp penceresindeki eşyaları yöneten fonksiyonlar.
        *   `Cube_request_result_list`, `Cube_request_material_info`: İstemciye tarif bilgilerini gönderen fonksiyonlar.
        *   `Cube_InformationInitialize`: Tarif bilgilerini istemciye göndermek üzere ön işler.
*   **Bağlantılı Dosyalar:** `cube.cpp`, `stdafx.h`.

### `CubeManager.h`

*   **Amaç:** Yenilenmiş (Renewal) Küp sistemi için merkezi yönetim sınıfı olan `CCubeManager`\'ı (singleton) ve ilgili veri yapılarını tanımlar. `#if defined(__CUBE_RENEWAL__)` gibi bir kontrol olmamasına rağmen, `cube.h` ve `cube.cpp`\'deki kontrollerle birlikte düşünüldüğünde bu dosyaların yeni sistemi temsil ettiği varsayılabilir.
*   **Temel İşlevler/İçerik:**
    *   **`CUBE_VALUE` Struct:**
        *   Eşya VNUM\'unu (`vnum`) ve sayısını (`count`) (her ikisi de `int32_t`) tutar.
        *   `operator==` ile karşılaştırma imkanı sunar.
    *   **`CUBE_DATA` Struct:**
        *   Tek bir yenilenmiş küp tarifini tanımlar:
            *   `npc_vnum` (`std::vector<int32_t>`): Tarifi gerçekleştirebilen NPC VNUM\'ları.
            *   `item` (`std::vector<CUBE_VALUE>`): Gerekli malzemeler.
            *   `reward` (`std::vector<CUBE_VALUE>`): Üretim sonucu elde edilebilecek olası ödüller.
            *   `percent` (int): Başarı şansı (1-100).
            *   `gold` (`int32_t`): Gerekli altın miktarı.
            *   `gem_point` (`int32_t`): Gerekli \"gem\" (Gaya vb. özel para birimi) miktarı.
            *   `allow_copy` (bool): İlk malzemenin efsunlarının sonuca kopyalanıp kopyalanmayacağı.
            *   `category` (`std::string`): Tarifin kategorisi.
            *   `not_remove` (int): Üretim sonrası silinmeyecek malzemenin VNUM\'u.
            *   `set_value` (int): Set eşyası oluşturma/değiştirme ile ilgili değer.
    *   **`CCubeManager` Sınıfı (Singleton):**
        *   Yapıcı (`CCubeManager()`): Sınıfı başlatır.
        *   `FN_check_cube_data(CUBE_DATA* cube_data)`: Yüklenen bir tarif verisinin geçerliliğini kontrol eder.
        *   `Cube_init()`: Küp sistemini başlatır, tarifleri (`cube.txt`) yükler.
        *   `Cube_close(LPCHARACTER ch)`: Belirtilen karakter için küp penceresini kapatır.
        *   `GetDataVector()`: Yüklenmiş tüm tariflerin (`CUBE_DATA*`) vektörünü döndürür.
        *   `RefineCube(...)`: Asıl küp üretim/geliştirme işlemini başlatan ve yöneten ana fonksiyon.
*   **Bağlantılı Dosyalar:** `CubeManager.cpp`, `stdafx.h`.

### `cuberenewal.h`

*   **Amaç:** Yenilenmiş (Renewal) Küp sistemi için temel tanımlamaları, veri yapılarını ve fonksiyon bildirimlerini içerir. Bu, `cube.h` ve `CubeManager.h`\'den farklı veya onların yerine geçen bir implementasyon olabilir.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler:** `CUBE_MAX_NUM` (24), `CUBE_MAX_DISTANCE` (1000).
    *   **`CUBE_RENEWAL_VALUE` Struct:** Eşya VNUM (`DWORD`) ve sayısını (`int`) tutar. `operator==` içerir.
    *   **`CUBE_RENEWAL_DATA` Struct:** Yenilenmiş bir küp tarifini tanımlar:
        *   `npc_vnum` (`std::vector<WORD>`): Tarifi sunan NPC VNUM\'ları.
        *   `item` (`std::vector<CUBE_RENEWAL_VALUE>`): Gerekli malzemeler.
        *   `reward` (`std::vector<CUBE_RENEWAL_VALUE>`): Olası ödüller.
        *   `category` (`std::string`): Tarif kategorisi.
        *   `percent` (int): Başarı şansı.
        *   `gold` (unsigned int): Altın maliyeti.
        *   `gem` (unsigned int): Gem (Gaya vb.) maliyeti.
        *   `allowCopy` (DWORD): Efsun/soket kopyalamaya izin verip vermediğini belirten bayrak/değer (0 veya 1 gibi).
    *   **Fonksiyon Bildirimleri:**
        *   `Cube_init()`: Sistemi başlatır, tarifleri (`cube.txt`) yükler.
        *   `Cube_load(const char *file)`: Tarif dosyasını yükler.
        *   `Cube_InformationInitialize()`: Yüklenen tarif bilgilerini istemciye gönderilecek formata hazırlar.
        *   `Cube_open(LPCHARACTER ch)`: Karakter için küp penceresini açar ve tarifleri gönderir.
        *   `Cube_close(LPCHARACTER ch)`: Karakter için küp penceresini kapatır (istemciye paket göndermez).
        *   `Cube_Make(LPCHARACTER ch, int index, int count_item, int index_item_improve)`: Üretim işlemini gerçekleştirir.
        *   `SendDateCubeRenewalPackets(LPCHARACTER ch, BYTE subheader, DWORD npcVNUM = 0)`: İstemciye `TPacketGCCubeRenewalReceive` paketi ile küp verilerini veya komutlarını gönderir.
*   **Bağlantılı Dosyalar:** `cuberenewal.cpp`, `stdafx.h`.

### `dragon_soul_table.h`

*   **Amaç:** Ejderha Taşı Simyası (Dragon Soul System) için gerekli veri yapılarını (`SApply`) ve `dragon_soul_table.txt` dosyasından okunan Simya verilerini yöneten `DragonSoulTable` sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`SApply` Struct:** Efsun türünü (`EApplyTypes`), değerini (`int`) ve isteğe bağlı olasılığını (`float`) tutar.
    *   **`DragonSoulTable` Sınıfı:**
        *   `ReadDragonSoulTableFile`: Ana yükleme fonksiyonu.
        *   `GetDragonSoulGroupName`: Simya türü (BYTE) ve grup adı (string) arasında dönüşüm yapar.
        *   `GetBasicApplys`/`GetAdditionalApplys`: Simya türüne göre temel/ek efsun listesini (`TVecApplys`) döndürür.
        *   `GetApplyNumSettings`: Simya seviyesine göre efsun sayısı ayarlarını (temel, min ek, max ek) döndürür.
        *   `GetWeight`: Simya seviyesine göre efsun ağırlığını döndürür.
        *   `GetRefineGradeValues`/`GetRefineStepValues`/`GetRefineStrengthValues`: Kademe/Saflık/Güç yükseltme değerlerini (gerekli malzeme, ücret, olasılıklar) döndürür.
        *   `GetDragonHeartExtValues`/`GetDragonSoulExtValues`: Ejderha Yüreği/Taşı çıkarma değerlerini (şarj, olasılık, yan ürün) döndürür.
        *   Özel metotlar (Dosya okuma, veri kontrolü).
        *   Üyeler (`CGroupTextParseTreeLoader`, `CGroupNode` işaretçileri, map/vektörler).
*   **Bağlantılı Dosyalar:** `dragon_soul_table.cpp`, `constants.h`, `group_text_parse_tree.h`.

### `DragonLair.h`

*   **Amaç:** Ejderha Odası (Dragon Lair) örneklerini yöneten sınıfları (`CDragonLair`, `CDragonLairManager`) tanımlar. Özel harita oluşturma, oyuncu ışınlama, ejderha ölümü ve harita çökmesi gibi olayların yönetilmesini içerir.
*   **Temel İşlevler/İçerik:**
    *   **`CDragonLair` Sınıfı:**
        *   Tek bir aktif Ejderha Odası örneğini temsil eder.
        *   Lonca ID'si, temel harita indeksi, özel örnek harita indeksi ve başlangıç zamanını saklar.
        *   `GetEstimatedTime()`: Odanın aktif kalma süresini hesaplar.
        *   `OnDragonDead()`: Odadaki ejderha öldüğünde çağrılır (loglama için).
    *   **`CDragonLairManager` Sınıfı (Singleton):**
        *   Tüm aktif Ejderha Odası örneklerini yönetir.
        *   `Start()`: Yeni bir özel harita oluşturur, lonca üyelerini ışınlar, `CDragonLair` nesnesi yaratır ve canavarları doğurur.
        *   `OnDragonDead()`: Ejderha ölüm olayını işler, haritanın çöküşünü tetikler ve örneği temizler.
        *   `GetLairCount()`: Aktif oda sayısını verir.
*   **Bağlantılı Dosyalar:** `DragonLair.cpp`, `<boost/unordered_map.hpp>`, `../../common/stl.h`.

### `DragonSoul.h`

*   **Amaç:** Ejderha Taşı Simyası (Dragon Soul System) için ana yönetim sınıfı olan `DSManager`'ı (singleton) tanımlar. Simya taşlarının oluşturulması, efsunlanması, geliştirilmesi, etkinleştirilmesi, süresi ve çıkarılması gibi tüm işlemleri yönetir. Bu işlemler için `DragonSoulTable` sınıfından veri alır.
*   **Temel İşlevler/İçerik:**
    *   **`DSManager` Sınıfı (Singleton):**
        *   `ReadDragonSoulTableFile`: Simya konfigürasyon dosyasını (`dragon_soul_table.txt`) okur.
        *   `GetDragonSoulInfo`: VNUM'dan Simya taşı bilgilerini (tür, kademe, saflık, güç) çıkarır.
        *   `GetBasePosition`/`IsValidCellForThisItem`: Simya envanteri pozisyonlarını yönetir.
        *   `GetDuration`/`LeftTime`/`IsTimeLeftDragonSoul`: Simya taşlarının süresini yönetir.
        *   `ExtractDragonHeart`: Simya taşından Ejderha Yüreği çıkarır.
        *   `PullOut`: Takılı bir Simya taşını çıkarır.
        *   `DoRefineGrade`/`DoRefineStep`/`DoRefineStrength`: Simya taşlarının seviyelerini yükseltir.
        *   `DoChangeAttribute` (`__DS_CHANGE_ATTR__`): Simya taşı efsunlarını değiştirir (opsiyonel).
        *   `DragonSoulItemInitialize`: Yeni Simya taşına başlangıç değerlerini atar.
        *   `HasActivedAllSlotsByPage`: Simya sayfasının tam aktif olup olmadığını kontrol eder.
        *   `ActivateDragonSoul`/`DeactivateDragonSoul`/`IsActiveDragonSoul`: Simya taşlarının aktif/pasif durumunu ve etkilerini yönetir.
        *   `GetDSSetGrade`/`GetDSSetValue` (`__DS_SET__`): Simya seti bonuslarını yönetir (opsiyonel).
*   **Bağlantılı Dosyalar:** `DragonSoul.cpp`, `dragon_soul_table.h`, `item.h`, `char.h`, `../../common/length.h`.

*Buraya `.h` dosyalarının belgeleri eklenecektir.*

---

## Kaynak Kod Dosyaları (.cpp)

### `ClientPackageCryptInfo.cpp`

*   **Amaç:** `CClientPackageCryptInfo` sınıfının metotlarını uygular. Kripto dosyalarını (`cshybridcrypt*`) okur, içindeki şifreleme anahtarlarını ve haritaya özgü Ek Veri Bloklarını (SDB) ayrıştırır, saklar ve istemciye göndermek üzere serileştirir/önbelleğe alır.
*   **Temel İşlevler/İçerik:**
    *   **`LoadPackageCryptFile`:** Tek bir kripto dosyasını açar, başlığından anahtar verisinin boyutunu ve SDB başlangıcını okur. Anahtar verisini okuyup `m_vecPackageCryptKeys` vektörüne ekler. Ardından SDB bölümünü okur: Her SDB paketi için paket adı hash'ini, dosya sayısını okur; her dosya için dosya adı hash'ini, ilişkili harita adını ve asıl SDB veri akışını okuyup `m_mapPackageSDB` haritasına (harita adı anahtarıyla) depolar.
    *   **`LoadPackageCryptInfo`:** Belirtilen dizindeki tüm `cshybridcrypt*` dosyalarını bulur ve her biri için `LoadPackageCryptFile` fonksiyonunu çağırır. Yükleme öncesi mevcut verileri temizler.
    *   **`GetPackageCryptKeys`:** `m_vecPackageCryptKeys` içindeki tüm anahtar verilerini ve toplam paket sayısını (`m_nCryptKeyPackageCnt`) içeren bir byte dizisi oluşturur. Bu diziyi ilk çağrıda oluşturup `m_pSerializedCryptKeyStream` içinde önbelleğe alır ve sonraki çağrılarda doğrudan bu önbelleği döndürür.
    *   **`GetRelatedMapSDBStreams`:** Verilen harita adı (küçük harfe çevrilir) için `m_mapPackageSDB` haritasında arama yapar. Bulunan `TPerFileSDBInfo` nesnesinin `GetSerializedStream` metodunu çağırır. Bu metot da benzer şekilde SDB verilerini ilk çağrıda serileştirip önbelleğe alır ve sonraki çağrılarda önbelleği döndürür.
*   **Bağlantılı Dosyalar:** `ClientPackageCryptInfo.h`, `stdafx.h`, "../../common/stl.h", `<stdio.h>`, dosya/dizin işlemleri (`dirent.h`/`xdirent.h`).

### `cmd_oxevent.cpp`

*   **Amaç:** OX Event (Bilgi Yarışması) ile ilgili GM komutlarını (`ACMD`) uygular.
*   **Temel İşlevler/İçerik:**
    *   **`ACMD(do_oxevent_show_quiz)`:** Yüklü olan OX sorularını (`quiz.lua`) komutu çalıştıran GM'e listeler (`COXEventManager::instance().ShowQuizList`).
    *   **`ACMD(do_oxevent_log)`:** Mevcut OX Event'in kazananlarını bir log dosyasına kaydeder (`COXEventManager::instance().LogWinner`). Kazanan yoksa veya işlem başarısız olursa bilgi verir.
    *   **`

### `cube.cpp`

*   **Amaç:** `cube.h`'de bildirilen Küp sistemi fonksiyonlarını ve `CUBE_DATA` metotlarını uygular. `cube.txt` dosyasından tarifleri yükler, eşya üretim mantığını çalıştırır, istemci etkileşimlerini yönetir ve sonuçları loglar. Bu dosya da `#if !defined(__CUBE_RENEWAL__)` kontrolü içerir, yani belirli bir Küp sistemi versiyonuna aittir.
*   **Temel İşlevler/İçerik:**
    *   **Global Veriler:**
        *   `s_cube_proto` (`std::vector<CUBE_DATA*>`): `cube.txt`'den yüklenen tüm tarifleri (`CUBE_DATA` nesneleri) saklayan ana vektör.
        *   `cube_info_map` (`TCubeMapByNPC`): Tarif bilgilerini istemciye göndermek için yapılandırılmış verileri tutan harita (NPC VNUM -> Ödül Listesi).
        *   `cube_result_info_map_by_npc` (`TCubeResultInfoTextByNPC`): Her NPC için istemciye gönderilecek formatlanmış tarif listesi metnini önbelleğe alan harita.
        *   `s_isInitializedCubeMaterialInformation`: İstemci için tarif bilgilerinin hazırlanıp hazırlanmadığını gösteren bayrak.
    *   **Yardımcı Fonksiyonlar (`FN_*`):**
        *   `FN_check_item_count`: Karakterin küp penceresinde belirli bir malzemeden yeterli sayıda olup olmadığını kontrol eder.
        *   `FN_remove_material`: Belirtilen sayıda malzemeyi karakterin küp penceresinden siler.
        *   `FN_find_cube`: Karakterin küp penceresindeki eşyalara ve konuşulan NPC'ye göre `s_cube_proto` içinden uygun tarifi bulur.
        *   `FN_check_valid_npc`: Bir NPC'nin küp tarifi sunup sunmadığını kontrol eder.
        *   `FN_check_cube_data`: Yüklenen bir tarifin temel verilerinin (VNUM/count > 0) geçerli olup olmadığını kontrol eder.
    *   **Tarif Yükleme (`Cube_init`, `Cube_load`):**
        *   `Cube_init`: Önceki tarifleri temizler ve `Cube_load`'u çağırır.
        *   `Cube_load`: `cube.txt` dosyasını açar, satır satır okur. Yorumları (`#`) atlar. `section`, `npc`, `item`, `reward`, `percent`, `gold`, `not_remove`, `end` gibi anahtar kelimelere göre verileri ayrıştırır. Her `section` için yeni bir `CUBE_DATA` oluşturur, ilgili verileri doldurur ve `end` ile karşılaşınca doğrular ve `s_cube_proto` vektörüne ekler.
        *   **Küp Penceresi Yönetimi (`Cube_open`, `Cube_close`, `Cube_add_item`, `Cube_delete_item`):**
            *   `Cube_open`: Gerekli kontrolleri (mesafe, başka pencere açık mı vb.) yapar. Karakterin küp durumunu ayarlar (`SetCubeNpc`) ve istemciye `cube open` komutunu gönderir.
            *   `Cube_close`: Karakterin küp durumunu sıfırlar ve istemciye `cube close` komutunu gönderir.
            *   `Cube_add_item`: İstemciden gelen istekle envanterdeki bir eşyayı karakterin küp dizisine (`GetCubeItem()`) ekler.
            *   `Cube_delete_item`: Küp dizisindeki belirtilen slottaki eşyayı kaldırır (işaretçiyi NULL yapar).
            *   Ekleme/çıkarma sonrası istemci arayüzünü güncellemek için `FN_update_cube_status` çağrılır.
        *   **Üretim Mantığı (`Cube_make`):**
            *   Uygun tarifi bulur (`FN_find_cube`).
            *   Yeterli altını kontrol eder.
            *   Malzemeleri siler (`remove_material`).
            *   Altını alır (`PointChange`).
            *   Başarı şansını (`percent`) kullanarak sonucu belirler.
            *   Başarılıysa: Ödülü seçer (`reward_value`), oyuncuya verir (`AutoGiveItem`), istemciye `cube success` komutunu gönderir ve loglar (`CubeLog`).
            *   Başarısızsa: İstemciye `cube fail` komutunu gönderir ve loglar.
        *   **İstemci Bilgi Sağlama (`Cube_InformationInitialize`, `Cube_MakeCubeInformationText`, `Cube_request_result_list`, `Cube_request_material_info`):**
            *   `Cube_InformationInitialize`: `s_cube_proto`'yu işleyerek `cube_info_map`'ı oluşturur. Aynı ödülü veren karmaşık tarifleri (`complicateMaterial`) belirler.
            *   `Cube_MakeCubeInformationText`: `cube_info_map`'ı kullanarak her tarif için istemciye uygun formatta malzeme listesi metinleri (`materialInfo.infoText`) oluşturur.
            *   `Cube_request_result_list`: İstemci isteği üzerine, NPC için olası ödülleri formatlı bir string olarak gönderir.
            *   `Cube_request_material_info`: İstemci isteği üzerine, belirli tarif(ler) için önceden formatlanmış malzeme listesi metinlerini gönderir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `cube.h`, `constants.h`, `utils.h`, `log.h`, `char.h`, `dev_log.h`, `locale_service.h`, `item.h`, `item_manager.h`, `<sstream>`, `<boost/unordered_map.hpp>`.

### `CubeManager.cpp`

*   **Amaç:** `CubeManager.h`\'de bildirilen yenilenmiş Küp sistemi sınıfı `CCubeManager`\'ın metotlarını uygular. Küp tariflerini (`cube.txt`) yükler, üretim mantığını yönetir ve istemciyle iletişim kurar.
*   **Temel İşlevler/İçerik:**
    *   **`split` Fonksiyonu:** Yardımcı bir fonksiyon. Verilen bir string\'i belirtilen ayırıcı karaktere göre böler ve sonuçları bir string vektörüne doldurur. `Cube_init` içinde tarif dosyasını ayrıştırmak için kullanılır.
    *   **`Cube_init()`:**
        *   Sistemi başlatır ve `cube.txt` dosyasını okur.
        *   Dosyayı satır satır işler, yorumları atlar.
        *   Her satırı anahtar kelimelere (`section`, `npc`, `item`, `reward`, `percent`, `gold`, `category`, `gem`, `allow_copy`, `set_value`, `not_remove`, `end`) göre ayrıştırır.
        *   `CUBE_DATA` nesneleri oluşturur ve ayrıştırılan değerleri (yeni eklenen `category`, `gem_point`, `allow_copy`, `set_value` dahil) ilgili alanlara atar.
        *   Geçerli tarifleri (`FN_check_cube_data` ile kontrol edilir) `s_cube_proto` vektörüne ekler.
    *   **`Cube_close(LPCHARACTER ch)`:**
        *   Karakterin küp NPC bilgisini sıfırlar.
        *   İstemciye küp penceresini kapatması için `CUBE_RENEWAL_CLOSE` alt başlığı ile `TPacketGCCubeRenewal` paketi gönderir.
    *   **`IsStackableItem(uint32_t dwVnum)`:** Verilen VNUM\'a sahip eşyanın istiflenebilir olup olmadığını kontrol eder.
    *   **`RefineCube(...)`:**
        *   Küp üretim/geliştirme işleminin ana mantığını içerir. İstemciden gelen parametrelerle (hedef VNUM, malzeme listesi, çarpan, geliştirme eşyası indeksi) çalışır.
        *   Gerekli kontrolleri yapar (karakter durumu, işlem sıklığı vb.).
        *   Verilen bilgilere uyan tarifi `s_cube_proto` vektöründe arar.
        *   **Malzeme ve Maliyet Kontrolü:** Gerekli malzemelerin (`CountSpecifyItem`), altının (`GetGold`) ve gemin (`GetGem`) karakterde yeterli miktarda olup olmadığını kontrol eder (çarpanı dikkate alarak).
        *   **Başarı Şansı Artırma:** Eğer `indexImprove` ile bir eşya belirtilmişse, bu eşyanın sayısını kullanarak başarı şansını artırır ve eşyayı eksiltir.
        *   **Sonuç Belirleme:** Tarifin başarı şansını (`percent`) ve (varsa) eklenen şansı kullanarak rastgele bir sayı ile üretimin başarılı olup olmayacağını belirler.
        *   **Malzeme/Maliyet Düşme:** Gerekli malzemeleri (`RemoveSpecifyItem`), altını (`PointChange`) ve gemi (`PointChange`) karakterden düşer (`not_remove` ile belirtilen hariç).
        *   **Başarılı Üretim:**
            *   `set_value` aktifse: İlgili malzeme eşyasını bulur ve set değerini (`SetItemSetValue`) ayarlar (yeni eşya oluşturmaz).
            *   `allow_copy` aktifse: Yeni bir eşya oluşturur, malzeme eşyasının efsunlarını kopyalar (`CopyAllAttrTo`) ve yeni eşyayı verir. Ana malzeme (eğer `not_remove` değilse) silinir.
            *   Normal üretim: Yeni bir ödül eşyası oluşturur (`CreateItem`) ve karaktere verir (`AddToCharacter`).
        *   **Başarısız Üretim:** Eğer `allow_copy` veya `set_value` aktifse ve `not_remove` uygulanmıyorsa, kullanılan ana malzemenin sayısını bir azaltır.
        *   **Bildirim:** İşlem sonucunu (başarı/başarısızlık) karakterin sohbet penceresine gönderir (`ChatPacket`).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `CubeManager.h`, `desc_client.h`, `buffer_manager.h`, `char_manager.h`, `locale_service.h`, `char.h`, `affect.h`, `utils.h`, `item_manager.h`, `item.h`, `config.h`, `<iostream>`, `<fstream>`, `<sstream>`, `<vector>`, `<functional>`.

### `cuberenewal.cpp`

*   **Amaç:** `cuberenewal.h`'de bildirilen yenilenmiş Küp sistemi için gerekli işlevleri uygular. Küp tariflerini (`cube.txt`) yükler, üretim mantığını yönetir ve istemciyle iletişim kurar.
*   **Temel İşlevler/İçerik:**
    *   **`Cube_init()`:**
        *   Sistemi başlatır ve `cube.txt` dosyasını okur.
        *   Dosyayı satır satır işler, yorumları atlar.
        *   Her satırı anahtar kelimelere (`section`, `npc`, `item`, `reward`, `percent`, `gold`, `category`, `gem`, `allow_copy`, `set_value`, `not_remove`, `end`) göre ayrıştırır.
        *   `CUBE_DATA` nesneleri oluşturur ve ayrıştırılan değerleri (yeni eklenen `category`, `gem_point`, `allow_copy`, `set_value` dahil) ilgili alanlara atar.
        *   Geçerli tarifleri (`FN_check_cube_data` ile kontrol edilir) `s_cube_proto` vektörüne ekler.
    *   **`Cube_close(LPCHARACTER ch)`:**
        *   Karakterin küp NPC bilgisini sıfırlar.
        *   İstemciye küp penceresini kapatması için `CUBE_RENEWAL_CLOSE` alt başlığı ile `TPacketGCCubeRenewal` paketi gönderir.
    *   **`IsStackableItem(uint32_t dwVnum)`:** Verilen VNUM\'a sahip eşyanın istiflenebilir olup olmadığını kontrol eder.
    *   **`RefineCube(...)`:**
        *   Küp üretim/geliştirme işleminin ana mantığını içerir. İstemciden gelen parametrelerle (hedef VNUM, malzeme listesi, çarpan, geliştirme eşyası indeksi) çalışır.
        *   Gerekli kontrolleri yapar (karakter durumu, işlem sıklığı vb.).
        *   Verilen bilgilere uyan tarifi `s_cube_proto` vektöründe arar.
        *   **Malzeme ve Maliyet Kontrolü:** Gerekli malzemelerin (`CountSpecifyItem`), altının (`GetGold`) ve gemin (`GetGem`) karakterde yeterli miktarda olup olmadığını kontrol eder (çarpanı dikkate alarak).
        *   **Başarı Şansı Artırma:** Eğer `indexImprove` ile bir eşya belirtilmişse, bu eşyanın sayısını kullanarak başarı şansını artırır ve eşyayı eksiltir.
        *   **Sonuç Belirleme:** Tarifin başarı şansını (`percent`) ve (varsa) eklenen şansı kullanarak rastgele bir sayı ile üretimin başarılı olup olmayacağını belirler.
        *   **Malzeme/Maliyet Düşme:** Gerekli malzemeleri (`RemoveSpecifyItem`), altını (`PointChange`) ve gemi (`PointChange`) karakterden düşer (`not_remove` ile belirtilen hariç).
        *   **Başarılı Üretim:**
            *   `set_value` aktifse: İlgili malzeme eşyasını bulur ve set değerini (`SetItemSetValue`) ayarlar (yeni eşya oluşturmaz).
            *   `allow_copy` aktifse: Yeni bir eşya oluşturur, malzeme eşyasının efsunlarını kopyalar (`CopyAllAttrTo`) ve yeni eşyayı verir. Ana malzeme (eğer `not_remove` değilse) silinir.
            *   Normal üretim: Yeni bir ödül eşyası oluşturur (`CreateItem`) ve karaktere verir (`AddToCharacter`).
        *   **Başarısız Üretim:** Eğer `allow_copy` veya `set_value` aktifse ve `not_remove` uygulanmıyorsa, kullanılan ana malzemenin sayısını bir azaltır.
        *   **Bildirim:** İşlem sonucunu (başarı/başarısızlık) karakterin sohbet penceresine gönderir (`ChatPacket`).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `cuberenewal.h`, `desc_client.h`, `buffer_manager.h`, `char_manager.h`, `locale_service.h`, `char.h`, `affect.h`, `utils.h`, `item_manager.h`, `item.h`, `config.h`, `<iostream>`, `<fstream>`, `<sstream>`, `<vector>`, `<functional>`.

### `dragon_soul_table.cpp`

*   **Amaç:** `DragonSoulTable` sınıfının metotlarını uygular. `dragon_soul_table.txt` dosyasını `CGroupTextParseTreeLoader` kullanarak okur, ayrıştırır, verileri dahili yapılarda saklar ve bu verilere erişim sağlayan metotları sunar. Yüklenen verilerin geçerliliğini kontrol eder.
*   **Temel İşlevler/İçerik:**
    *   **Global Sabitler:** `g_astGradeName`, `g_astStepName`, `g_astMaterialName` (string dizileri).
    *   **`ReadDragonSoulTableFile()`:** Ana yükleme fonksiyonu. `CGroupTextParseTreeLoader` oluşturur, dosyayı yükler, `Read*` fonksiyonlarını çağırır, grup düğümlerini önbelleğe alır ve `Check*` fonksiyonları ile doğrular.
    *   **Okuma Fonksiyonları (`ReadVnumMapper`, `ReadBasicApplys`, `ReadAdditionalApplys`):** İlgili grup düğümlerini alır, satırları/alt düğümleri işler, değerleri okur ve ilgili map'lere (`m_map_name_to_type`, `m_map_basic_applys_group` vb.) doldurur.
    *   **Doğrulama Fonksiyonları (`Check*`):** Yüklenen verilerin varlığını ve mantıksal tutarlılığını (negatif olmayan değerler, doğru liste boyutları vb.) kontrol eder, hataları loglar.
    *   **Veri Erişim Fonksiyonları (`Get*`):** Parametre olarak alınan Simya türü/seviyesi gibi bilgilere göre önbelleğe alınmış grup düğümlerinden ilgili veriyi (`GetGroupValue`/`GetGroupRow` ile) okuyup döndürür. Hataları kontrol eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `group_text_parse_tree.h`, `dragon_soul_table.h`, `item_manager.h`, `<boost/lexical_cast.hpp>`.

### `DragonLair.cpp`

*   **Amaç:** `DragonLair.h`'de tanımlanan Ejderha Odası sistemi mantığını uygular. Özel harita örneği oluşturma, oyuncuları ışınlama, canavar doğurma (`regen_do` ile), ejderha ölümünü işleme ve örnek haritanın zamanlı çöküşünü/temizlenmesini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Yardımcı Functor'lar:**
        *   `FWarpToDragronLairWithGuildMembers`: Lonca üyelerini örnek haritaya ışınlar.
        *   `FWarpToVillage`: Oyuncuları köylerine gönderir (`GoHome`).
    *   **`DragonLair_Collapse_Event` (Event):** Ejderha öldükten sonra tetiklenen zamanlı olay:
        *   Adım 0: Tamamlama süresi duyurusu gönderir, 30 sn bekler.
        *   Adım 1: Kalan oyuncuları köye ışınlar, 30 sn bekler.
        *   Adım 2: Özel haritayı yok eder (`SECTREE_MANAGER::DestroyPrivateMap`) ve `CDragonLair` nesnesini siler.
    *   **`CDragonLair::OnDragonDead`:** Ejderha öldürme olayını loglar (`LogManager::DragonSlayLog`).
    *   **`CDragonLairManager::Start`:** Örnek oluşturma sürecini yönetir: özel harita yaratır (`CreatePrivateMap`), oyuncuları ışınlar, `CDragonLair` oluşturur, `instance_regen.txt` dosyasını yükleyerek canavarları doğurur (`regen_do`).
    *   **`CDragonLairManager::OnDragonDead`:** İlgili `CDragonLair` örneğini bulur, `OnDragonDead` metodunu çağırır, `DragonLair_Collapse_Event`'i başlatır ve örneği yöneticiden siler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `DragonLair.h`, `entity.h`, `sectree_manager.h`, `char.h`, `guild.h`, `locale_service.h`, `regen.h`, `log.h`, `utils.h`.

### `DragonSoul.cpp`

*   **Amaç:** `DSManager` sınıfının metotlarını uygular. Simya sistemi işlemlerinin (yükseltme, çıkarma, etkinleştirme, efsun atama vb.) çekirdek mantığını içerir. `DragonSoulTable`'dan alınan verilere ve olasılıklara göre sonuçları belirler.
*   **Temel İşlevler/İçerik:**
    *   **Yardımcı Fonksiyonlar:** `Gamble` (olasılıklara göre rastgele seçim), `MakeDistinctRandomNumberSet` (tekrarsız rastgele seçim), VNUM ayrıştırma (`GetType`, `GetGradeIdx` vb.).
    *   **Efsun Yönetimi (`PutAttributes`, `RefreshItemAttributes`, `DragonSoulItemInitialize`):**
        *   `PutAttributes`: Yeni Simya taşına temel ve rastgele ek efsunları, tablo verisi ve ağırlığa göre hesaplanmış değerlerle atar.
        *   `RefreshItemAttributes`: Mevcut efsunların değerlerini taşın seviyesine göre günceller.
        *   `DragonSoulItemInitialize`: `PutAttributes` çağırır ve süre soketini ayarlar.
    *   **Yükseltme Mantığı (`DoRefine*`):** Malzemeleri, ücreti doğrular; tablo verisi ve `Gamble` ile sonucu belirler; malzemeleri/ücreti eksiltir; sonuç eşyasını oluşturur/yok eder; sonucu loglar ve paketle bildirir.
    *   **`DoChangeAttribute` (`__DS_CHANGE_ATTR__`):** Efsun değiştirme mantığını içerir (opsiyonel).
    *   **Etkinleştirme/Devre Dışı Bırakma (`ActivateDragonSoul`, `DeactivateDragonSoul`, `RefreshDragonSoulState`):** Aktif desteği kontrol eder, ilgili soketi ayarlar, `ModifyPoints` ile etkileri uygular/kaldırır, süre event'ini yönetir, loglar.
    *   **Çıkarma İşlemleri (`ExtractDragonHeart`, `PullOut`):** Gerekli malzemeleri kontrol eder, tablo verisi ve `Gamble` ile sonucu belirler, malzemeleri eksiltir, Ejderha Yüreği veya yan ürün oluşturabilir, sonucu loglar.
    *   **Durum Kontrolleri:** İlgili item soketlerini okur.
    *   **Set Bonusu (`GetDSSetGrade`, `GetDSSetValue` - `__DS_SET__`):** Takılı taşları kontrol eder ve set bonusunu hesaplar (opsiyonel).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `item.h`, `item_manager.h`, `unique_item.h`, `packet.h`, `desc.h`, `char.h`, `dragon_soul_table.h`, `log.h`, `DragonSoul.h`, `<boost/lexical_cast.hpp>`.

### `dungeon.h`

*   **Amaç:** Özel zindan (dungeon) örneklerini (`CDungeon`) ve bu örnekleri yöneten merkezi sınıfı (`CDungeonManager`) tanımlar. Zindanlar, genellikle görevler, etkinlikler veya özel savaş alanları için oluşturulan, ana oyun dünyasından izole edilmiş geçici haritalardır.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:** `FlagMap` (string->int), `RegenVector` (`LPREGEN` vektörü), `TPartyMap` (parti->int), `TUniqueMobMap` (string->karakter), `IdType` (uint32_t, zindan kimliği).
    *   **`CDungeon` Sınıfı:**
        *   Tek bir zindan örneğini yönetir.
        *   **Kimlik ve Harita:** `GetId()`, `GetMapIndex()`.
        *   **Oyuncu/Parti Yönetimi:** `JoinParty`, `QuitParty`, `Join`, `IncMember`, `DecMember`, `IncPartyMember`, `DecPartyMember`, `ForEachMember`.
        *   **Canavar Yönetimi:** `IncMonster`, `DecMonster`, `CountMonster`, `CountRealMonster`, `KillAll`, `Purge`, `IncKillCount`, `GetKillMobCount`, `GetKillStoneCount`.
        *   **Spawn İşlemleri:** `Spawn`, `SpawnMob`, `SpawnMob_ac_dir`, `SpawnGroup`, `SpawnNameMob`, `SpawnGotoMob`, `SpawnRegen`, `AddRegen`, `ClearRegen`, `IsValidRegen`.
        *   **Özel Varlık (Unique) Yönetimi:** `SetUnique`, `SpawnMoveUnique`, `SpawnMoveGroup`, `SpawnUnique`, `SpawnStoneDoor`, `SpawnWoodenDoor`, `KillUnique`, `PurgeUnique`, `IsUniqueDead`, `GetUniqueHpPerc`, `GetUniqueVid`, `DeadCharacter`, `UniqueSetMaxHP`, `UniqueSetHP`, `UniqueSetDefGrade`.
        *   **Işınlama/Çıkış:** `JumpAll`, `WarpAll`, `JumpParty`, `ExitAll`, `ExitAllToStartPosition`, `JumpToEliminateLocation`, `SetExitAllAtEliminate`, `SetWarpAtEliminate`, `SetWarpLocation`, `SendDestPositionToParty`.
        *   **Bayraklar ve Durum:** `Notice`, `GetFlag`, `SetFlag`, `IsUsePotion`, `IsUseRevive`, `UsePotion`, `UseRevive`, `CheckEliminated`, `IsAllPCNearTo`.
        *   **Eşya Grupları:** `CreateItemGroup`, `GetItemGroup`.
        *   **Olaylar (Event):** `deadEvent`, `exit_all_event_`, `jump_to_event_` (Zindanın boş kalması, tüm canavarların ölmesi gibi durumlarda zamanlanmış eylemleri tetiklemek için).
        *   **Üyeler:** Zindan ID'si (`m_id`), harita indeksleri (`m_lOrigMapIndex`, `m_lMapIndex`), karakter seti (`m_set_pkCharacter`), bayrak haritası (`m_map_Flag`), eşya grubu haritası (`m_map_ItemGroup`), parti haritası (`m_map_pkParty`), alan haritası referansı (`m_map_Area`), özel canavar haritası (`m_map_UniqueMob`), sayaçlar (`m_iMobKill`, `m_iStoneKill`, `m_iMonsterCount`), durum bayrakları (`m_bUsePotion`, `m_bUseRevive`, `m_bExitAllAtEliminate`, `m_bWarpAtEliminate`), olay işaretçileri, regen vektörü (`m_regen`), (`__DUNGEON_RENEWAL__` için) katılımcı haritası (`m_Participants`).
    *   **`CDungeonManager` Sınıfı (Singleton):**
        *   Tüm `CDungeon` örneklerini yönetir.
        *   `Create(long lOriginalMapIndex)`: Verilen harita indeksine dayanarak yeni bir özel zindan haritası ve `CDungeon` örneği oluşturur.
        *   `Destroy(CDungeon::IdType dungeon_id)`: Belirtilen ID'ye sahip zindanı ve ilişkili özel haritayı yok eder.
        *   `Find(CDungeon::IdType dungeon_id)`: ID'ye göre zindanı bulur.
        *   `FindByMapIndex(long lMapIndex)`: Zindan harita indeksine göre zindanı bulur.
        *   **Üyeler:** Zindan haritaları (`m_map_pkDungeon`, `m_map_pkMapDungeon`), sonraki ID (`next_id_`).
*   **Bağlantılı Dosyalar:** `dungeon.cpp`, `sectree_manager.h`, `<map>`, `<vector>`, `<unordered_map>`.

### `dungeon.cpp`

*   **Amaç:** `dungeon.h` içinde bildirilen `CDungeon` ve `CDungeonManager` sınıflarının fonksiyonlarını ve zindan sistemiyle ilgili olay işleyicilerini uygular. Özel harita oluşturma, oyuncu/parti yönetimi, canavar/nesne spawn etme, olay tabanlı durum değişiklikleri (örneğin tüm canavarlar öldüğünde çıkış/ışınlama) ve zindan örneğinin temizlenmesi gibi temel zindan mekaniklerini içerir.
*   **Temel İşlevler/İçerik:**
    *   **Zindan Yönetimi (`CDungeonManager`):**
        *   `Create`: `SECTREE_MANAGER::CreatePrivateMap` ile özel harita oluşturur, benzersiz bir ID atar, yeni bir `CDungeon` nesnesi oluşturur ve bunu hem ID'ye hem de harita indeksine göre haritalarda saklar.
        *   `Destroy`: Zindan nesnesini haritalardan kaldırır, zindanla ilişkili Quest sunucu zamanlayıcılarını iptal eder (`quest::CQuestManager::instance().CancelServerTimers`), özel haritayı yok eder (`SECTREE_MANAGER::instance().DestroyPrivateMap`) ve `CDungeon` nesnesini bellekten siler.
    *   **Zindan Başlatma/Bitirme (`CDungeon`):**
        *   `Constructor`/`Initialize`: Zindan oluşturulduğunda başlangıç değerlerini (sayaçlar, bayraklar, olay işaretçileri vb.) ayarlar ve `SECTREE_MANAGER`'dan alan haritası referansını alır.
        *   `Destructor`: Parti bağlantısını keser (`SetDungeon_for_Only_party`), regen listesini temizler (`ClearRegen`), zamanlanmış olayları iptal eder (`event_cancel`).
    *   **Oyuncu/Parti İşlemleri:**
        *   `Join`/`JoinParty`: Oyuncuları veya partileri `FWarpToDungeon` functor'ı kullanarak zindan haritasına ışınlar (`ch->WarpSet`) ve zindanın parti listesine ekler.
        *   `QuitParty`: Partiyi zindan listesinden çıkarır.
        *   `IncMember`/`DecMember`: Zindana giren/çıkan karakterleri `m_set_pkCharacter` setinde takip eder. Zindan boşaldığında (`m_set_pkCharacter.empty()`), `dungeon_dead_event` olayını belirli bir süre sonra zindanı yok etmek üzere zamanlar.
        *   `IncPartyMember`/`DecPartyMember`: Parti bazında oyuncu sayısını takip eder ve parti tamamen çıktığında `QuitParty`'yi çağırır.
    *   **Spawn İşlemleri:**
        *   `Spawn*` fonksiyonları: `CHARACTER_MANAGER::SpawnMob`/`SpawnGroup`/`SpawnMoveGroup` kullanarak canavarları, grupları veya özel "unique" varlıkları zindan haritası içinde (`m_lMapIndex`) belirtilen konumlara (genellikle alan adı veya koordinat) spawn eder. Koordinatlar, ana haritadaki göreceli konumdan özel haritadaki mutlak konuma dönüştürülür (`pkSectreeMap->m_setting.iBaseX/Y` kullanılarak).
        *   Oluşturulan canavarlara `ch->SetDungeon(this)` ile zindan işaretçisi atanır. `SpawnUnique` ve ilgili fonksiyonlar, canavarı `m_map_UniqueMob` haritasına ekler ve `AFFECT_DUNGEON_UNIQUE` etkisini verir.
        *   `SpawnRegen`: Belirtilen regen dosyasını (`.txt`) `regen_do` fonksiyonu ile zindan haritasında çalıştırır. `AddRegen`, `ClearRegen`, `IsValidRegen` ile zindan içindeki regen olaylarını yönetir.
    *   **Işınlama/Çıkış İşlemleri:**
        *   `JumpAll`/`WarpAll`/`JumpParty`: Oyuncuları haritalar arasında veya zindan içinde belirli konumlara ışınlamak için `FWarpToPosition` ve `FWarpToPositionForce` functor'larını kullanır. Bu functor'lar karakterlerin `WarpSet` veya `Show`/`Stop` metotlarını çağırır.
        *   `ExitAll`/`ExitAllToStartPosition`: Zindandaki tüm oyuncuları çıkarmak için `FExitDungeon` (kayıtlı konuma) veya `FExitDungeonToStartPosition` (başlangıç köyüne) functor'larını kullanır.
    *   **Olay Yönetimi ve `CheckEliminated`:**
        *   Bir canavar öldüğünde `DecMonster` çağrılır ve bu da `CheckEliminated`'ı tetikler.
        *   `CheckEliminated`: Canavar sayısı (`m_iMonsterCount`) sıfıra ulaştığında, önceden ayarlanmış bayraklara (`m_bExitAllAtEliminate`, `m_bWarpAtEliminate`) göre hareket eder. Belirli bir gecikme (`m_iWarpDelay`) varsa, `dungeon_exit_all_event` (tümünü çıkarma) veya `dungeon_jump_to_event` (belirtilen konuma ışınlama) olaylarını zamanlar. Gecikme yoksa eylemi hemen gerçekleştirir (`ExitAll`, `JumpToEliminateLocation`).
        *   `dungeon_*_event` fonksiyonları: Zamanlayıcı tetiklendiğinde çalışır, ilgili `CDungeon` örneğini bulur ve planlanan eylemi (çıkış veya ışınlama) yapar.
    *   **Özel Varlık (Unique) Yönetimi:**
        *   `KillUnique`/`PurgeUnique`: İlgili canavarı `m_map_UniqueMob` haritasından çıkarır ve `Dead()` (öldürür) veya `M2_DESTROY_CHARACTER` (yok eder) çağırır.
        *   `DeadCharacter`: Ölen bir karakterin unique olup olmadığını kontrol eder ve öyleyse haritadan siler.
        *   `UniqueSet*`: Unique canavarların HP veya savunma gibi özelliklerini değiştirir.
    *   **Yardımcı Fonksiyonlar/Yapılar:** Zindan içi işlemleri kolaylaştırmak için çeşitli functor'lar (örneğin, `FKillSectree`, `FPurgeSectree` ile haritadaki tüm canavarları/eşyaları temizleme; `FNotice` ile zindan içindekilere mesaj gönderme; `FCountMonster` ile canavar sayma; `FNearPosition` ile oyuncuların belirli bir alanda olup olmadığını kontrol etme) ve olay bilgi yapıları (`dungeon_id_info`) kullanılır.
    *   **`__DUNGEON_RENEWAL__` Implementasyonu:** `RegisterParticipant`, `IsParticipantRegistered`, `ClearParticipants` fonksiyonları, zindana giren oyuncuların kaydını tutar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `dungeon.h`, `char.h`, `char_manager.h`, `party.h`, `affect.h`, `packet.h`, `desc.h`, `config.h`, `regen.h`, `start_position.h`, `item.h`, `item_manager.h`, `utils.h`, `questmanager.h`, `desc_manager.h`, `log.h`, `locale_service.h`.

### `event_manager.h`

*   **Amaç:** Zamanlanmış oyun etkinliklerini (EXP bonusu, eşya düşürme oranı artışı, özel canavar spawn olayları vb.) yönetmek için `CEventManager` singleton sınıfını tanımlar. Etkinliklerin takvimini yükler, başlangıç ve bitiş zamanlarına göre otomatik olarak aktif/pasif hale getirir ve istemcilere etkinlik bilgilerini gönderir.
*   **Temel İşlevler/İçerik:**
    *   **`CEventManager` Sınıfı (Singleton):**
        *   **Yönetim:** `Initialize` (etkinlik verilerini yükler, yeniden yükleme desteği), `CancelActiveEvents` (aktif etkinlikleri durdurur, kuyrukları temizler), `UpdateEventStatus` (DB'den gelen bilgiyle etkinliği tamamlandı işaretler).
        *   **Zamanlama Kuyrukları:** `Enqueue` (etkinliği başlangıç/bitiş kuyruğuna ekler ve zamanlar), `Dequeue` (etkinliği kuyruktan çıkarır ve zamanlayıcıyı iptal eder).
        *   **Bilgi Alma/Dönüşüm:** `GetEvent` (string -> enum), `GetEventString` (enum -> string), `SendEventInfo` (istemciye etkinlik takvimini gönderir), `GetEventState` (bir etkinliğin aktif olup olmadığını Quest bayrağı üzerinden kontrol eder).
        *   **Durum Ayarlama:** `SetEventState` (etkinliği aktif/pasif yapar, ilgili `Set*Event` fonksiyonunu çağırır), `UpdateGameFlag` (Quest bayrağını yerel olarak ayarlar ve DB'ye bildirir).
        *   **Özel Etkinlik İşleyicileri:** `SetExperienceEvent`, `SetItemDropEvent`, `SetBossEvent`, `SetMetinEvent`, `SetMiningEvent`, `SetGoldFrogEvent`, `SetMoonlightEvent`, `SetHexegonalEvent`, `SetFishingEvent`, `SetHideAndSeekEvent`, `SetOXEvent`, `SetTanakaEvent` (her etkinlik türü için özel başlatma/durdurma mantığını içerir).
        *   **Yeniden Yükleme:** `SetReloadMode`, `GetReloadMode`.
    *   **Enum'lar:**
        *   `EEventTypes`: Farklı etkinlik türlerini tanımlar (örn. `EVENT_TYPE_EXPERIENCE`, `EVENT_TYPE_BOSS`).
        *   `EEvent`: Etkinliklerle ilgili sabitleri içerir (örn. özel harita indeksleri, döngü aşamaları, etkinliklerin yönetildiği sunucu kanalı `EVENT_CHANNEL`).
        *   `EQueueType`: Zamanlama kuyruklarının türlerini belirtir (`QUEUE_TYPE_START`, `QUEUE_TYPE_END`).
    *   **Üyeler:** Etkinlik adı haritası (`m_mapEventName`), yüklenen etkinlik verileri (`m_mapEvent`), başlangıç ve bitiş zamanlayıcılarını tutan kuyruk haritaları (`m_mapEventStartQueue`, `m_mapEventEndQueue`), yeniden yükleme bayrağı (`m_bReload`).
*   **Bağlantılı Dosyalar:** `event_manager.cpp`, `../../common/tables.h` (`TEventTable`), `singleton.h`, `<unordered_map>`.

### `event_manager.cpp`

*   **Amaç:** `CEventManager` sınıfının metotlarını ve zamanlama için kullanılan olay işleyici fonksiyonlarını (`EVENTFUNC`) uygular.
*   **Temel İşlevler/İçerik:**
    *   **Olay İşleyici Fonksiyonları (`EVENTFUNC`):**
        *   `warp_all_to_village_event`: Belirtilen haritadaki tüm oyuncuları başlangıç köylerine ışınlar (`FWarpAllToVillage` functor'ı kullanılır).
        *   `dynamic_spawn_cycle_event`: Boss Avı, Metin Yağmuru gibi çok aşamalı spawn etkinliklerini yönetir. Zamanlanmış aralıklarla (`PASSES_PER_SEC`) çalışır, farklı regen dosyalarını (`strRegenPath`, `strRegenPath2`) yükler ve belirli rauntlarda (`CLEAR_ENTITY_STAGE*_ROUND`) haritadaki canavarları temizler (`FKillSectree` functor'ı kullanılır).
        *   `static_spawn_cycle_event`: Madencilik, Altın Kurbağa, Tanaka gibi tek aşamalı, tekrarlayan spawn etkinliklerini yönetir. Belirli aralıklarla aynı regen dosyasını (`strRegenPath`) yükler ve başlangıçta haritayı temizler (Tanaka hariç).
        *   `queue_event_process`: Bir etkinliğin başlangıç veya bitiş zamanı geldiğinde çağrılır. İlgili `CEventManager::SetEventState` fonksiyonunu çağırarak etkinliği başlatır/durdurur ve `CEventManager::Dequeue` ile zamanlayıcıyı kuyruktan kaldırır.
    *   **`CEventManager` Metot Uygulamaları:**
        *   **`Initialize`:** Gelen etkinlik tablosunu (`TEventTable*`) döngüyle işler. Her etkinliği `m_mapEvent` haritasına ekler. Eğer sunucu etkinlik kanalıysa (`EVENT_CHANNEL`) ve etkinlik tamamlanmamışsa, başlangıç ve bitiş zamanları için `Enqueue` çağırır.
        *   **`Enqueue`:** Başlangıç/bitiş zamanına kalan süreyi hesaplar. Süre pozitifse, `queue_event_process` fonksiyonunu o süre sonunda çalıştıracak bir olay (`event_create`) oluşturur ve `LPEVENT` işaretçisini ilgili kuyruk haritasına (`m_mapEventStartQueue` veya `m_mapEventEndQueue`) ekler. Eğer başlangıç zamanı zaten geçmişse, etkinliği hemen `SetEventState(true)` ile başlatır.
        *   **`Dequeue`:** İlgili kuyruk haritasından olay işaretçisini bulur, `event_cancel` ile zamanlayıcıyı iptal eder ve haritadan siler. Bitiş kuyruğundan silme durumunda, veritabanına etkinliğin bittiğini bildirmek için `HEADER_GD_UPDATE_EVENT_STATUS` paketi gönderir.
        *   **`Set*Event` Fonksiyonları:** Her etkinlik türü için özel mantığı uygular:
            *   `UpdateGameFlag` ile Quest bayrağını günceller ve DB'ye bildirir.
            *   `BroadcastNotice` ile oyunculara duyuru yapar.
            *   EXP/Item Drop: `CPrivManager` ile sunucu geneli bonusları ayarlar.
            *   Boss/Metin/Mining/Gold Frog/Tanaka: `dynamic/static_spawn_cycle_event` olaylarını başlatarak canavar spawnlarını yönetir. Bitişlerinde `warp_all_to_village_event` ile oyuncuları ışınlar.
            *   Moonlight/Hexagonal/Fishing/HideAndSeek/OX: Sadece bayrakları ayarlar ve duyuru geçer.
            *   Tanaka: Etkinlik kanalı dışındaysa, P2P paketi ile etkinlik kanalına bildirir.
        *   **Diğer:** `CancelActiveEvents` (yeniden yükleme için temizlik), `UpdateEventStatus` (DB yanıtıyla tamamlama), `GetEventState` (Quest bayrağı kontrolü), `SendEventInfo` (istemciye takvim gönderme).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `event.h`, `event_manager.h`, `text_file_loader.h`, `locale_service.h`, `quest.h`, `questmanager.h`, `priv_manager.h`, `sectree_manager.h`, `start_position.h`, `char.h`, `regen.h`, `config.h`, `desc_client.h`, `desc_manager.h`, `log.h`, `p2p.h`.

### `exchange.h`

*   **Amaç:** Oyuncular arası eşya ve altın/çek ticaretini yöneten `CExchange` sınıfını ve ilgili sabitleri tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`EExchangeValues` Enum'u:**
        *   `EXCHANGE_ITEM_MAX_NUM`: Ticaret penceresindeki maksimum eşya slot sayısı (12 veya 24).
        *   `EXCHANGE_MAX_DISTANCE`: Ticareti başlatmak için izin verilen maksimum mesafe.
    *   **`CExchange` Sınıfı:**
        *   Bir ticaret oturumunun tek bir tarafını yönetir.
        *   **Üyeler:** Ticaret ortağı (`m_pCompany`), sahip (`m_pOwner`), teklif edilen eşyalar (`m_apItems`, `m_aItemPos`, `m_abItemDisplayPos`), altın (`m_lGold`), çek (`m_lCheque`, opsiyonel), kabul durumu (`m_bAccept`), ticaret penceresi grid'i (`m_pGrid`).
        *   **Metotlar:**
            *   `Accept(bool bIsAccept)`: Kabul durumunu ayarlar. Her iki taraf da kabul ederse ticareti tamamlamayı dener.
            *   `Cancel()`: Ticareti iptal eder.
            *   `AddGold(long lGold)` / `AddCheque(long lCheque)`: Altın/çek ekler.
            *   `AddItem(TItemPos item_pos, BYTE display_pos, ...)`: Envanterden eşya ekler.
            *   `RemoveItem(BYTE pos)`: Ticaretten eşya çıkarır.
            *   `GetOwner()`/`GetCompany()`/`GetAcceptStatus()`/`SetCompany()`: Temel bilgilere erişim.
            *   `GetItemByPosition(int i)`: Belirli slottaki eşyayı döndürür.
            *   `Done()` (private): Eşya/altın transferini gerçekleştirir.
            *   `Check(int* piItemCount)` (private): Sahibin teklif edilenleri hala bulundurduğunu kontrol eder.
            *   `CheckSpace()` (private): Karşı tarafın envanterinde yer olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `exchange.cpp`. (Dolaylı olarak: `char.h`, `item.h`, `grid.h`).

### `exchange.cpp`

*   **Amaç:** `CExchange` sınıfının metotlarını ve ticareti başlatan `CHARACTER::ExchangeStart` fonksiyonunu uygular.
*   **Temel İşlevler/İçerik:**
    *   **`exchange_packet(...)`:** İstemciye ticaret durumuyla ilgili (`HEADER_GC_EXCHANGE`) paketleri gönderen yardımcı fonksiyon. Eşya eklerken detaylı bilgi içerir.
    *   **`CHARACTER::ExchangeStart(LPCHARACTER victim)`:** Ticaret isteğini başlatır. Mesafe, meşguliyet, engel durumu gibi birçok ön kontrolü yapar. Başarılı olursa her iki taraf için `CExchange` nesnesi oluşturur ve istemcilere bildirir.
    *   **`CExchange::AddItem`:** Eşyanın ticarete uygunluğunu (anti-flag, kilit, ruh bağı vb.) kontrol eder, ticaret penceresi grid'ine ekler, eşyayı "ticarette" olarak işaretler (`SetExchanging`) ve her iki tarafa paket gönderir.
    *   **`CExchange::RemoveItem`:** Eşyayı ticaretten çıkarır, işaretini kaldırır, grid'den siler ve paket gönderir.
    *   **`CExchange::AddGold`/`AddCheque`:** Sahibin yeterli miktara sahip olup olmadığını ve (çek için) karşı tarafın limitini kontrol eder, miktarı ayarlar ve paket gönderir.
    *   **`CExchange::Accept`:** Kabul durumunu günceller ve paket gönderir. Eğer her iki taraf da kabul ettiyse, `Check()` ve `CheckSpace()` ile son kontrolleri yapar. Başarılıysa `Done()` çağrılarak transfer gerçekleştirilir, başarısızsa `Cancel()` çağrılır.
    *   **`CExchange::Check`:** Sahibin teklif ettiği altın/çeki ve eşyaların hala envanterinde olup olmadığını doğrular.
    *   **`CExchange::CheckSpace`:** Karşı tarafın envanterini geçici bir grid üzerinde simüle ederek teklif edilen tüm eşyalar için yeterli yer olup olmadığını kontrol eder.
    *   **`CExchange::Done`:** Teklif edilen eşyaları sahibinden çıkarıp (`RemoveFromCharacter`) karşı tarafa ekler (`AddToCharacter`), altın/çek transferini `PointChange` ile yapar ve işlemleri loglar (`LogManager`).
    *   **`CExchange::Cancel`:** Ticareti sonlandırır, istemcilere paket gönderir, eşyaların `SetExchanging` durumunu sıfırlar ve `CExchange` nesnelerini siler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `exchange.h`, `grid.h`, `utils.h`, `desc.h`, `desc_client.h`, `char.h`, `item.h`, `item_manager.h`, `packet.h`, `log.h`, `db.h`, `locale_service.h`, `DragonSoul.h`, `config.h`, (isteğe bağlı) `messenger_manager.h`, `GrowthPetSystem.h`.

### `fishing.h`

*   **Amaç:** Balıkçılık sistemi için temel sabitleri, veri yapılarını (balık bilgileri, olay verisi) ve fonksiyon bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler ve Enum'lar:** `MAX_FISH`, `NUM_USE_RESULT_COUNT`, `FISH_BONE_VNUM`, `SHELLFISH_VNUM`, `EARTHWORM_VNUM`, `USED_*` (balık kullanım sonuçları), `FISHING_TIME_*` (zamanlama türleri), `MAX_FISHING_TIME_COUNT`, `CAMPFIRE_MOB`, `FISHER_MOB`, `FISH_MIND_PILL_VNUM`.
    *   **`fishing_event_info` Struct:** Balıkçılık zamanlayıcısı için gerekli verileri (oyuncu ID, adım, zaman, balık ID) tutar.
    *   **Fonksiyon Bildirimleri:** `Initialize`, `CreateFishingEvent`, `Take` (oltayı çekme), `Stop` (opsiyonel), `Simulation`, `UseFish`, `Grill`, `RefinableRod`, `RealRefineRod`.
*   **Bağlantılı Dosyalar:** `fishing.cpp`, `item.h`.

### `fishing.cpp`

*   **Amaç:** `fishing.h`'de bildirilen balıkçılık fonksiyonlarını ve olay işleyicisini uygular. Balık tutma mekaniğini, balık verilerini, zamanlamayı, zorluğu ve ilgili işlemleri yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Veri Yükleme (`Initialize`):** `fishing.txt` veya `fishing.json` dosyasından balık bilgilerini (`fish_info` dizisi) ve olasılık tablolarını (`g_prob_accumulate`, `g_prob_sum`) yükler.
    *   **Balık Belirleme (`DetermineFish`):** Oyuncunun bulunduğu haritaya, premium etkilere ve event bayraklarına göre olasılık tablosundan rastgele bir balık seçer.
    *   **Balıkçılık Olayı (`fishing_event`):**
        *   Zamanlanmış olay işleyicisi.
        *   Adım 0: Oltayı sallar (`FishingReact`), tutulacak balığı belirler, küre varsa bildirir (`PredictFish`), sonraki adımı zamanlar.
        *   Adım 1+: Başarısızlık durumunu tetikler (`FishingFail`).
    *   **Oltayı Çekme (`Take`):** Oyuncu komutuyla çağrılır. Geçen süreye (`ms`), tutulan balığın zorluğuna ve oyuncunun seviyesine göre `Compute` fonksiyonu ile başarıyı belirler. Sonuca göre (`FishingSuccess` veya `FishingFail`) paket gönderir, eşya verir (`AutoGiveItem`), balık boyutunu hesaplar (`GetFishLength`) ve loglar (`FishLog`). Olta puanını günceller (`FishingPractice`).
    *   **Başarı Hesaplama (`Compute`):** Zaman adımına göre `aFishingTime` tablosundan başarı yüzdesini alır. Başarılıysa oyuncu seviyesi ile balık zorluğunu karşılaştırır.
    *   **Balık Kullanımı (`UseFish`):** `fish_info[idx].used_table` ve rastgele sayı ile sonuç eşyasını (kemik, istiridye vb.) belirler.
    *   **Pişirme (`Grill`):** Balığı alır, karşılık gelen pişmiş balık VNUM'unu (`grill_vnum`) bulur ve verir.
    *   **Olta Yönetimi:** `GetFishingLevel` (seviyeyi alır), `FishingPractice` (puanı artırır), `RefinableRod` (geliştirme kontrolü), `RealRefineRod` (geliştirme işlemi).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `fishing.h`, `constants.h`, `item_manager.h`, `config.h`, `packet.h`, `sectree_manager.h`, `char.h`, `char_manager.h`, `log.h`, `questmanager.h`, `buffer_manager.h`, `desc_client.h`, `locale_service.h`, `affect.h`, `unique_item.h`, (isteğe bağlı) `GrowthPetSystem.h`, `filesystem`, `fmt/core.h`, `nlohmann/json.hpp`.

### `GrowthPetSystem.h`

*   **Amaç:** Geliştirilebilir Pet Sistemi (Growth Pet System) için ana sınıfları (`CGrowthPetSystem`, `CGrowthPetSystemActor`), veri yapılarını (`TPetSkillTable`), sabitleri (enum'lar: `EGrowthPet*`, `EPetRevive`) ve yetenek/evrim tablolarını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler/Enum'lar:** Sistem limitleri, pencere türleri, EXP türleri, yetenek indeksleri, mühür VNUM'ları, stat artış aralıkları (`PET_*_RANGE`), yumurta bilgileri (`PET_HATCH_INFO_RANGE`), evrim malzemeleri (`PET_EVOLVE_CUBE`).
    *   **`TPetSkillTable` Struct & Sabit Tablolar:** Pet yeteneklerinin temel bilgilerini (`pet_skill_table`) ve uzmanlık değerlerini (`pet_skill_specialist_table`) tanımlar.
    *   **`CGrowthPetSystemActor` Sınıfı:** Tek bir aktif peti yönetir. Sahibi, pet karakteri, mühür eşyası ve pet bilgilerini (`TGrowthPetInfo`) tutar. Peti çağırma/gönderme, güncelleme (AI), seviye/EXP, evrim, yetenek, besleme, isim değiştirme, özellik belirleme ve buff yönetimi fonksiyonlarını içerir.
    *   **`CGrowthPetSystem` Sınıfı:** Bir karaktere ait tüm petleri yönetir. `CGrowthPetSystemActor` nesnelerini haritada tutar. Ana çağırma/gönderme işlemlerini yönetir ve `Update` metodu ile tüm aktif petleri günceller (periyodik olay ile tetiklenir).
*   **Bağlantılı Dosyalar:** `GrowthPetSystem.cpp`, `item.h`, `utils.h`.

### `GrowthPetSystem.cpp`

*   **Amaç:** `GrowthPetSystem.h`'de bildirilen Geliştirilebilir Pet Sistemi sınıflarının ve fonksiyonlarının implementasyonunu sağlar. Pet yaşam döngüsü (yumurta, çağırma, gönderme, canlandırma), gelişim (seviye, evrim, EXP, besleme), yetenek yönetimi (öğrenme, geliştirme, silme) ve buff uygulama mantığını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Çağırma/Gönderme (`Summon`/`Dismiss`):** Pet karakterini spawn eder/yok eder, pet bilgilerini mühürden yükler/kaydeder, buffları uygular/kaldırır, ilgili event'i başlatır/durdurur.
    *   **Güncelleme (`Update`/`_UpdateFollowAI`):** Petin hayatta olup olmadığını kontrol eder, otomatik yetenekleri (İyileştirme, Ölümsüzlük, Kötü etki silme) tetikler, sahibi takip etmesini sağlar.
    *   **Seviye Atlama (`SetPetLevel`, `SetExp`):** EXP kazanımını yönetir (mob/item ayrı), seviye atlama koşullarını kontrol eder, seviye atlandığında statları yaşa ve türe göre (`PET_*_RANGE`) artırır, buffları günceller.
    *   **Evrim (`EvolvePet`):** Evrim koşullarını (seviye, yaş) kontrol eder, evrim seviyesini artırır, gerekirse pet görünümünü değiştirir (`EvolvePetRace`), EXP'yi sıfırlar.
    *   **Besleme (`ItemCubeFeed`):** Farklı pencere türlerine göre (can, evrim, EXP) eşyaları kontrol eder, tüketir ve ilgili pet değerini (süre, evrim malzemesi kontrolü, EXP) günceller.
    *   **Yetenek Yönetimi (`LearnPetSkill`, `IncreasePetSkill`, `Delete*Skill`, `GetPetSkillInformation`):** Yetenek kitaplarını/malzemelerini kontrol eder, petin yetenek bilgilerini (`TGrowthPetInfo`) günceller, buffları yeniden hesaplar (`GetPetSkillInformation`) ve uygular.
    *   **Buff Yönetimi (`GiveBuff`/`ClearBuff`):** Petin statlarından ve pasif yeteneklerinden gelen etkileri `AFFECT_GROWTH_PET` türüyle karaktere ekler/kaldırır.
    *   **Canlandırma (`RevivePet`, `Revive`):** Ölü pet için gerekli malzeme sayısını (yaşa göre) hesaplar, tüketir ve petin süresini/yaşını günceller.
    *   **Özellik Yönetimi (`PetAttrChange`, `Determine`):** Petin türünü ve statlarını rastgele değiştirir veya türünü belirler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `GrowthPetSystem.h`, `config.h`, `utils.h`, `vector.h`, `char.h`, `sectree_manager.h`, `char_manager.h`, `mob_manager.h`, `../../common/VnumHelper.h`, `packet.h`, `item_manager.h`, `item.h`, `desc_client.h`, `log.h`.

### `guild_manager.h`

*   **Amaç:** Sunucudaki tüm Loncaları (`CGuild`) merkezi olarak yöneten `CGuildManager` singleton sınıfını ve lonca savaşları ile ilgili yardımcı yapıları (`CGuildWarReserveForGame`) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CGuildWarReserveForGame` Sınıfı:** Planlanmış lonca savaşı verilerini ve bahislerini tutar.
    *   **`CGuildManager` Sınıfı (Singleton):**
        *   Lonca Yaşam Döngüsü: `CreateGuild`, `FindGuild`, `LoadGuild`, `TouchGuild`, `DisbandGuild`.
        *   Oyuncu Yönetimi: `Link`/`Unlink` (oyuncu-lonca bağlantısı), `LoginMember`, `P2PLoginMember`/`LogoutMember`.
        *   Lonca Savaşları: `RequestWar`, `WaitStartWar`, `StartWar`, `EndWar`, `CancelWar`, `DeclareWar`, `RefuseWar`, `ReserveWar`, `GetReservedWar`, `DeleteReservedWar`, `ReserveWarBet`, `ReserveWarBetCheck`, `ChangeMaster` (Savaş sırasında).
        *   Lonca Skilleri: `UseSkill`, `GSP`, `GuildPointChange`.
        *   İletişim: `Packet`, `SendGuildInfoPacket`, `SendGuildDataPacket`, `SendGuildWarPacket`.
        *   Yardımcı: `ChangeGuildGrade`, `OfferGuildExp`, `Initialize`.
*   **Bağlantılı Dosyalar:** `guild_manager.cpp`, `singleton.h`, `guild.h`.

### `guild_manager.cpp`

*   **Amaç:** `guild_manager.h`'de bildirilen `CGuildManager` sınıfının metotlarını uygular. Lonca oluşturma/yok etme, üye ekleme/çıkarma, lonca savaşı başlatma/bitirme/yönetme, lonca becerileri, lonca deneyimi ve derece yönetimi gibi tüm lonca sistemi mekaniklerini içerir.
*   **Temel İşlevler/İçerik:**
    *   **Lonca Yaşam Döngüsü:**
        *   `CreateGuild`: Yeni bir lonca oluşturur, lideri ekler ve veritabanına kaydeder (`db_clientdesc->DBPacket`).
        *   `LoadGuild`: Veritabanından gelen lonca verilerini yükler ve `CGuild` nesnesi oluşturur.
        *   `TouchGuild`: Loncanın son aktif zamanını günceller.
        *   `DisbandGuild`: Lonca üyelerini çıkarır, lonca arazilerini temizler (`building::CManager`), savaşı iptal eder ve veritabanından siler.
    *   **Oyuncu Yönetimi:**
        *   `Link`/`Unlink`: Karakter (`CHARACTER`) nesnesi ile `CGuild` nesnesi arasındaki bağlantıyı kurar/keser.
        *   `LoginMember`/`P2PLoginMember`/`LogoutMember`: Oyuncu giriş/çıkışlarını ve kanal değişikliklerini yönetir, `CGuild::LoginMember`/`LogoutMember` çağırır.
    *   **Lonca Savaşları:**
        *   `RequestWar`/`WaitStartWar`/`StartWar`/`EndWar`/`CancelWar`: Lonca savaşı sürecini yönetir (savaş isteği, bekleme, başlama, bitiş, iptal). Savaştaki lonca bilgilerini (`GuildWarListContainer`) günceller, ilgili `CGuild` metotlarını çağırır ve paketleri gönderir.
        *   `DeclareWar`/`RefuseWar`: Savaş ilanı ve reddetme işlemlerini yapar, DB'ye bildirir.
        *   `ReserveWar`/`GetReservedWar`/`DeleteReservedWar`: Planlanmış lonca savaşlarını yönetir (rezervasyon, bilgi alma, silme).
        *   `ReserveWarBet`: Planlanmış savaşlar için bahisleri yönetir.
        *   `ChangeMaster`: Savaş sırasında lider değişikliği mantığını içerir.
    *   **Lonca Skilleri ve Deneyim:**
        *   `UseSkill`: Lonca becerisini kullanır (`CGuild::UseSkill`), SP düşer.
        *   `GSP`: Lonca beceri puanını döndürür.
        *   `GuildPointChange`: Lonca deneyim puanını değiştirir (`CGuild::OfferExp`).
        *   `OfferGuildExp`: Oyuncudan loncaya deneyim aktarır.
    *   **İletişim:** `Packet`, `SendGuildInfoPacket`, `SendGuildDataPacket`, `SendGuildWarPacket` gibi fonksiyonlar istemciye lonca ile ilgili güncel bilgileri ve savaş durumunu gönderir.
        *   **Diğer:** `ChangeGuildGrade` (üye rütbesi değiştirme), `Initialize` (lonca verilerini yükleme).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `guild.h`, `guild_war.cpp`, `constants.h`, `skill.h`, `char.h`, `packet.h`, `desc_client.h`, `buffer_manager.h`, `char_manager.h`, `db.h`, `guild_manager.h`, `affect.h`, `p2p.h`, `questmanager.h`, `building.h`, `locale_service.h`, `log.h`.

### `guild.h`

*   **Amaç:** Tek bir loncayı temsil eden `CGuild` sınıfını ve lonca üyeleri (`SGuildMember`), rütbeler (`TGuildGrade`), temel lonca verileri (`TGuildData`), savaş bilgileri (`SGuildWar`) gibi ilgili yapıları ve sabitleri tanımlar. Lonca üyeleri, rütbeler, yetenekler, deneyim, savaşlar, yorumlar ve mali durumun yönetimi için arayüzü bildirir.
*   **Temel İşlevler/İçerik:**
    *   **Yapılar:**
        *   `SGuildMember`: Üye PID, rütbe, sınıf, seviye, bağışlanan EXP, isim.
        *   `TGuildGrade`: Rütbe adı ve yetki bayrakları (`GUILD_AUTH_*`).
        *   `TGuildData`: Lonca ID, lider PID, seviye, EXP, isim, rütbe dizisi, yetenekler, güç (SP), ladder puanı, savaş istatistikleri, altın.
        *   `SGuildWar`: Düşman lonca ile savaş bilgisi (başlangıç zamanı, skor, durum, tür, harita indeksi).
    *   **`CGuild` Sınıfı Bildirimleri:**
        *   Üye Yönetimi: `AddMember`, `RemoveMember`, `LoginMember`, `LogoutMember`, `GetMember`, `GetMemberCount`, `GetMaxMemberCount`, `ChangeMemberGrade`, `ChangeMemberGeneral`.
        *   Rütbe/Yetki: `ChangeGradeName`, `ChangeGradeAuth`, `HasGradeAuth`.
        *   EXP/Seviye: `OfferExp`, `GuildPointChange` (seviye atlama mantığı içerir), `GetLevel`.
        *   Yetenekler: `GetSkillLevel`, `SkillLevelUp`, `UseSkill`, `ComputeGuildPoints`, `SkillRecharge`, `UpdateSkill`, `SendSkillInfoPacket`, `SkillUsableChange`.
        *   Yorumlar: `AddComment`, `DeleteComment`, `RefreshComment`.
        *   Finans: `RequestDepositMoney`, `RequestWithdrawMoney`, `RecvMoneyChange`, `GetGuildMoney`, `ChargeSP`.
        *   Savaşlar: `GuildWarPacket`, `GetGuildWarState`, `GetGuildWarType`, `CanStartWar`, `UnderWar`, `SetWarScoreAgainstTo`, `GetWarScoreAgainstTo`, `GetWarStartTime`, `RequestDeclareWar`, `DeclareWar`, `RequestRefuseWar`, `RefuseWar`, `WaitStartWar`, `CheckStartWar`, `StartWar`, `EndWar`, `ReserveWar`, `SetGuildWarMapIndex`, `GuildWarEntryAsk`, `GuildWarEntryAccept`.
        *   Davet Sistemi: `Invite`, `InviteAccept`, `InviteDeny`, `VerifyGuildJoinableCondition`.
        *   Diğer: `Initialize`, `Load`, `Save*`, `Packet`, `Chat`, `Disband`, `ChangeMasterTo`, `HasLand`.
*   **Bağlantılı Dosyalar:** `guild.cpp`, `guild_war.cpp`, `constants.h`, `skill.h`, `char.h`, `packet.h`.

### `guild.cpp`

*   **Amaç:** `guild.h`'de bildirilen `CGuild` sınıfının (savaş dışı) metotlarını uygular. Üye yönetimi, giriş/çıkış işlemleri, rütbe değişiklikleri, yorumlar, yetenekler, deneyim, finans ve lonca oluşturma/yükleme/dağıtma gibi işlevleri içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Veritabanı Etkileşimi:** Lonca oluşturma/yükleme (`guild`, `guild_grade`, `guild_member` tablolarından okuma/yazma), üye/rütbe/yetki/seviye/EXP/beceri/para değişikliklerini DB'ye kaydetme (genellikle `HEADER_GD_GUILD_*` paketleri ile DB process'ine gönderilir).
    *   **Üye Yönetimi:** `m_member` (tüm üyeler), `m_memberOnline` (oyundaki üyeler), `m_memberP2POnline` (başka sunucudaki üyeler) konteynerlarını yönetir. Oyuncu giriş/çıkışında (`LoginMember`, `LogoutMember`, `P2P*`) setleri günceller, karakterin guild pointer'ını ayarlar (`ch->SetGuild`), `CGuildManager` ile bağlantı kurar/keser ve istemcilere/P2P'ye ilgili paketleri (`GUILD_SUBHEADER_GC_LIST`, `LOGIN`, `LOGOUT`, `REMOVE`) gönderir.
    *   **Seviye/EXP Yönetimi:** `OfferExp` ile oyuncudan EXP alır, `GuildPointChange` ile lonca EXP'sini artırır, `guild_exp_table`'a göre seviye atlama kontrolü yapar, seviye atlandığında yetenek puanı verir, gücü yeniler, ladder puanı ekler ve sonucu DB'ye/istemcilere bildirir.
    *   **Yetenek Yönetimi:** `SkillLevelUp` ile yetenek seviyesini artırır (yetenek puanı kontrolü), `UpdateSkill` ile DB'den gelen güncellemeyi uygular, `ComputeGuildPoints` ile max gücü hesaplar, `SaveSkill`/`SendDBSkillUpdate` ile değişiklikleri kaydeder/iletir.
    *   **Yorum Yönetimi:** `AddComment` (cooldown içerir), `DeleteComment` (yetki kontrolü yapar), `RefreshComment` fonksiyonları `guild_comment` tablosuyla etkileşime girer ve sonucu (`GUILD_SUBHEADER_GC_COMMENTS`) ilgili oyuncuya gönderir.
    *   **Finans Yönetimi:** `RequestDepositMoney`/`WithdrawMoney` ile oyuncu ve lonca arasında para transferi isteğini DB'ye iletir. `RecvMoneyChange` DB'den gelen güncellemeyi uygular ve üyelere bildirir (`GUILD_SUBHEADER_GC_MONEY_CHANGE`). `ChargeSP` ile oyuncu parasıyla lonca gücü (SP) doldurur.
    *   **Davet Sistemi:** `Invite` ile oyuncuya davet gönderir (`GUILD_SUBHEADER_GC_GUILD_INVITE`) ve zaman aşımı için event (`m_GuildInviteEventMap`) başlatır. `InviteAccept`/`Deny` ile sonucu işler, `VerifyGuildJoinableCondition` ile katılma koşullarını (loncadan ayrılma/dağılma cezası, zaten üye olma, lonca dolu, savaşta olma) kontrol eder.
*   **Bağlantılı Dosyalar:** `guild.h`, `stdafx.h`, `utils.h`, `config.h`, `char.h`, `packet.h`, `desc_client.h`, `buffer_manager.h`, `char_manager.h`, `db.h`, `guild_manager.h`, `affect.h`, `p2p.h`, `questmanager.h`, `building.h`, `locale_service.h`, `log.h`.

### `guild_war.cpp`

*   **Amaç:** `CGuild` sınıfının lonca savaşına özgü metotlarını uygular. Savaş ilanı, reddetme, başlatma, bitirme, skorlama ve savaş haritası yönetimi gibi lonca savaşı mekaniklerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Paket Yönetimi:** `GuildWarPacket` savaş durumunu (`GUILD_SUBHEADER_GC_WAR`), `SendEnemyGuild` mevcut savaşları ve skorları (`WAR_SCORE`) üyelere gönderir.
    *   **Durum Kontrolü:** `GetGuildWarState`, `GetGuildWarType`, `UnderWar`, `UnderAnyWar` fonksiyonları `m_EnemyGuild` haritasını kontrol ederek savaş durumunu sorgular.
    *   **Savaş Başlatma/Bitirme Akışı:**
        *   `RequestDeclareWar`: DB'ye savaş ilanı isteği (`HEADER_GD_GUILD_WAR`, `GUILD_WAR_SEND_DECLARE`) gönderir.
        *   `DeclareWar`: DB'den gelen yanıtla ilanı kaydeder (`m_EnemyGuild`, `GUILD_WAR_SEND_DECLARE`). Rakip de ilan ederse veya kabul ederse `WaitStartWar` veya `StartWar`'a geçiş yapılır.
        *   `WaitStartWar`: Savaş haritası gerektiren türler için `CWarMapManager::CreateWarMap` ile harita oluşturur, P2P ile diğer sunuculara bildirir (`HEADER_GG_GUILD_WAR_ZONE_MAP_INDEX`) ve durumu `GUILD_WAR_WAIT_START` yapar.
        *   `StartWar`: Durumu `GUILD_WAR_ON_WAR` yapar, başlangıç zamanını kaydeder, üyeleri bilgilendirir ve (gerekirse) haritaya giriş daveti gönderir (`GuildWarEntryAsk`).
        *   `EndWar`: Savaş haritasını sonlandırır (`pMap->SetEnded()`), üyeleri bilgilendirir (`GUILD_WAR_END`), `m_EnemyGuild`'den kaydı siler ve gerekirse lonca skillerinin etkilerini kaldırır.
    *   **Skorlama:** `SetWarScoreAgainstTo` skoru günceller (`it->second.score`), savaş haritası varsa oradaki skoru günceller (`pMap->UpdateScore`), yoksa skor paketini yayınlar.
    *   **Savaş Haritası Yönetimi:** `SetGuildWarMapIndex` ile harita indeksini kaydeder. `GuildWarEntryAsk` ile üyelere giriş daveti gönderir (quest letter). `GuildWarEntryAccept` ile oyuncuyu haritadaki başlangıç noktasına ışınlar.
    *   **Diğer:** `ReserveWar` (planlanmış savaşlar için), `LadderPoint` yönetimi gibi işlevleri içerir.
*   **Bağlantılı Dosyalar:** `guild.h`, `stdafx.h`, `constants.h`, `utils.h`, `config.h`, `log.h`, `char.h`, `packet.h`, `desc_client.h`, `buffer_manager.h`, `char_manager.h`, `db.h`, `affect.h`, `p2p.h`, `war_map.h`, `questmanager.h`, `sectree_manager.h`, `locale_service.h`, `guild_manager.h`.

### `horse_rider.h`

*   **Amaç:** Karakterin at/binek verilerini ve temel mantığını yöneten `CHorseRider` temel sınıfını tanımlar. Her at seviyesi için statları (`THorseStat`) ve global stat dizisini (`c_aHorseStat`) içerir.
*   **Temel İşlevler/İçerik:**
    *   **`THorseStat` Struct:** Seviye gereksinimi, NPC VNUM'u, Maks Sağlık/Dayanıklılık, Temel Statlar (ST, DX, HT, IQ), SungMa Statları (opsiyonel), Hasar ve Zırh değerlerini tutar.
    *   **`c_aHorseStat` Dizisi:** Her at seviyesi (0-`HORSE_MAX_LEVEL`) için `THorseStat` bilgilerini saklar.
    *   **`CHorseRider` Sınıfı:**
        *   Üyeler: `THorseInfo m_Horse` (Seviye, Sağlık, Dayanıklılık, Binme Durumu, Sağlık Düşüş Zamanı), `m_eventStaminaRegen`, `m_eventStaminaConsume`.
        *   Getter'lar: `GetHorseLevel`, `GetHorseHealth`, `GetHorseStamina`, `GetHorseMaxHealth`, `GetHorseMaxStamina`, `GetHorseST`/`DX`/`HT`/`IQ` vb.
        *   Sanal Metotlar: `ReviveHorse`, `HorseDie`, `SendHorseInfo`, `ClearHorseInfo`, `UpdateRideTime`, `SetHorseLevel`, `StartRiding`, `StopRiding`.
        *   Diğer Metotlar: `FeedHorse`, `ResetHorseHealthDropTime`, `EnterHorse`.
*   **Bağlantılı Dosyalar:** `horse_rider.cpp`, `constants.h`.

### `horse_rider.cpp`

*   **Amaç:** `CHorseRider` sınıfının metotlarını uygular. Atın can/dayanıklılık yönetimi, binme/inme mantığı, periyodik sağlık düşüşü ve olay (event) tabanlı dayanıklılık tüketimi/yenilenmesi gibi işlevleri içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`c_aHorseStat` Değerleri:** Her at seviyesi için statların tanımlandığı global dizi.
    *   **Dayanıklılık Yönetimi (`UpdateHorseStamina`, `StartStaminaConsume/RegenEvent`, `horse_stamina_consume/regen_event`):** Binek kullanıldığında dayanıklılık düşer (`HORSE_STAMINA_CONSUME_INTERVAL`), kullanılmadığında artar (`HORSE_STAMINA_REGEN_INTERVAL`). Event'ler aracılığıyla periyodik olarak çalışır. Dayanıklılık sıfıra düşerse binici attan iner (`StopRiding`).
    *   **Sağlık Yönetimi (`UpdateHorseHealth`, `ResetHorseHealthDropTime`, `CheckHorseHealthDropTime`, `FeedHorse`):** Belirli aralıklarla (`HORSE_HEALTH_DROP_INTERVAL`) atın canı 1 azalır. `FeedHorse` ile can artırılır ve düşüş zamanlayıcısı sıfırlanır. Can sıfıra düşerse `HorseDie` çağrılır.
    *   **Binme/İnme (`StartRiding`, `StopRiding`):** `m_Horse.bRiding` bayrağını değiştirir ve ilgili dayanıklılık event'ini başlatır/durdurur.
    *   **At Ölümü/Canlandırma (`HorseDie`, `ReviveHorse`):** At öldüğünde dayanıklılık event'lerini iptal eder, dayanıklılığı sıfırlar. Canlandırıldığında canı/dayanıklılığı seviyesine göre doldurur, sağlık düşüş zamanını sıfırlar ve dayanıklılık yenilenme event'ini başlatır.
    *   **Seviye Ayarlama (`SetHorseLevel`):** Atın seviyesini, canını, dayanıklılığını ayarlar ve sağlık düşüş zamanını sıfırlar.
*   **Bağlantılı Dosyalar:** `horse_rider.h`, `stdafx.h`, `constants.h`, `utils.h`, `config.h`, `char_manager.h`.

### `horsename_manager.h`

*   **Amaç:** Oyuncuların atlarına verdiği özel isimleri yöneten `CHorseNameManager` singleton sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   `m_mapHorseNames`: Oyuncu ID'lerini at isimlerine eşleyen harita.
    *   `GetHorseName(DWORD dwPlayerID)`: Oyuncunun at ismini alır.
    *   `UpdateHorseName(DWORD dwPlayerID, const char* szHorseName, bool broadcast)`: At ismini günceller ve isteğe bağlı olarak yayınlar.
    *   `Validate(LPCHARACTER pChar)`: Geçici at isimlerinin süresini kontrol eder.
*   **Bağlantılı Dosyalar:** `horsename_manager.cpp`, `<map>`, `<string>`, `singleton.h`.

### `horsename_manager.cpp`

*   **Amaç:** `CHorseNameManager` metotlarını uygular.
*   **Orta Seviye Implementasyon Detayları:**
    *   `GetHorseName`: `m_mapHorseNames` haritasından ismi okur.
    *   `UpdateHorseName`: `m_mapHorseNames` haritasını günceller ve `BroadcastHorseName`'i çağırır.
    *   `BroadcastHorseName`: İsim güncelleme bilgisini (`HEADER_GD_UPDATE_HORSE_NAME`) DB process'ine gönderir.
    *   `Validate`: Karakterdeki `AFFECT_HORSE_NAME` etkisini ve `horse_name.valid_till` quest flag'ini kontrol eder. Süre dolmuşsa, atı gönderip tekrar çağırır, etkiyi kaldırır, `UpdateHorseName` ile ismi boş string olarak günceller.
*   **Bağlantılı Dosyalar:** `horsename_manager.h`, `stdafx.h`, `desc_client.h`, `char_manager.h`, `char.h`, `affect.h`, `utils.h`.

### `loot_filter.h`

*   **Amaç:** Oyuncuların düşen eşyaları belirli kriterlere göre otomatik olarak toplamalarını veya yoksaymalarını sağlayan bir "Loot Filter" (Ganimet Filtresi) sistemini yönetmek için `CLootFilter` sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CLootFilter` Sınıfı:**
        *   **Üye Değişkenler:**
            *   `m_LootFilterSettings[ELootFilter::LOOT_FILTER_SETTINGS_MAX]` (BYTE dizisi): Filtre ayarlarını tutar. `ELootFilter` enum'u (muhtemelen `../../common/tables.h` içinde tanımlı) farklı filtre kategorilerini ve alt ayarlarını (örn. silahlar için min/max refine, min/max seviye, hangi sınıflar için toplanacağı vb.) belirler.
            *   `m_LootFilteredItemsSet` (`std::unordered_set<DWORD>`): Filtre tarafından "görünmez" hale getirilen (yani oyuncu tarafından toplanmayacak olan) eşyaların Sanal ID'lerini (VID) saklar.
        *   **Metotlar:**
            *   `SetLootFilterSettings`: Verilen ayarlarla filtre yapılandırmasını günceller ve `m_LootFilteredItemsSet`'i temizler.
            *   `IsLootFilteredItem`: Bir eşyanın (VID ile) filtre tarafından gizlenip gizlenmediğini kontrol eder.
            *   `InsertLootFilteredItem`: Bir eşyayı (VID ile) filtreli listeye ekler (gizler).
            *   `ClearLootFilteredItems`: Filtreli (gizlenmiş) tüm eşya listesini temizler.
            *   `CanPickUpItem`: Bir `LPITEM` nesnesi alarak, mevcut filtre ayarlarına göre bu eşyanın oyuncu tarafından toplanıp toplanamayacağını belirler. Bu ana kontrol fonksiyonudur ve çeşitli `CheckTopic*` yardımcı metotlarını çağırır.
        *   **Özel (Private) `CheckTopic*` Metotları:**
            *   `CheckTopicWeapon`, `CheckTopicArmor`, `CheckTopicHead`, `CheckTopicCommon`, `CheckTopicCostume`, `CheckTopicDS`, `CheckTopicUnique`, `CheckTopicRefine`, `CheckTopicPotion`, `CheckTopicFishMining`, `CheckTopicMountPet`, `CheckTopicSkillBook`, `CheckTopicEtc`, `CheckTopicEvent`: Her biri belirli bir eşya kategorisi için filtreleme mantığını uygular. Bu metotlar, `m_LootFilterSettings` dizisindeki ilgili ayarlara (açık/kapalı durumu, min/max refine, min/max seviye, alt tür seçimi, sınıf seçimi vb.) göre eşyanın toplanıp toplanamayacağına karar verir.
*   **Bağlantılı Dosyalar:** `loot_filter.cpp` (uygulama), `stdafx.h` (genellikle), `<unordered_set>`, `../../common/tables.h` (`ELootFilter`, `EItemTypes`, `EArmorSubTypes` vb. enum tanımları için). Bu sınıf muhtemelen `CHARACTER` sınıfı içinde bir üye olarak kullanılır veya `ITEM_MANAGER` ile entegre çalışır.

### `loot_filter.cpp`

*   **Amaç:** `loot_filter.h`'de tanımlanan `CLootFilter` sınıfının metotlarını uygular. Oyuncunun ayarlarına göre hangi eşyaların toplanabileceğini belirleyen filtreleme mantığını içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı/Yıkıcı (`CLootFilter::CLootFilter()`, `CLootFilter::~CLootFilter()`):**
        *   Yapıcı, `m_LootFilterSettings` dizisini sıfırlar.
    *   **Ayar Yönetimi:**
        *   `SetLootFilterSettings`: Verilen ayarları `m_LootFilterSettings`'e kopyalar ve `ClearLootFilteredItems`'ı çağırarak önceden filtrelenmiş eşya listesini temizler.
    *   **Filtrelenmiş Eşya Yönetimi:**
        *   `IsLootFilteredItem`: `m_LootFilteredItemsSet` içinde verilen VID'nin olup olmadığını kontrol eder.
        *   `InsertLootFilteredItem`: Verilen VID'yi `m_LootFilteredItemsSet`'e ekler.
        *   `ClearLootFilteredItems`: `m_LootFilteredItemsSet`'i temizler.
    *   **Ana Toplama Kontrolü (`CanPickUpItem`):**
        *   Verilen `LPITEM`'ın türüne (`bType`) ve alt türüne (`bSubType`) göre bir `switch` ifadesi kullanarak uygun `CheckTopic*` fonksiyonunu çağırır.
        *   Bazı özel VNUM'lar veya durumlar (örn. `item->IsHairDye()`) için doğrudan belirli `CheckTopic*` fonksiyonlarını çağırabilir veya varsayılan olarak `true` döndürebilir.
    *   **Kategori Bazlı Kontrol Fonksiyonları (`CheckTopic*`):**
        *   Her `CheckTopic*` fonksiyonu (örn. `CheckTopicWeapon`, `CheckTopicArmor`), öncelikle ilgili kategorinin genel açma/kapama ayarını (`m_LootFilterSettings[CATEGORY_ON_OFF]`) kontrol eder. Kapalıysa `false` döner.
        *   Ardından, eşyanın refine seviyesi, giyme seviyesi gibi özelliklerini, `m_LootFilterSettings` dizisindeki min/max ayarlarla karşılaştırır. Bu aralıkların dışındaysa `false` döner.
        *   Silah, zırh, başlık ve yetenek kitabı gibi kategoriler için, oyuncunun hangi sınıflara ait eşyaları toplamak istediğine dair ayarlara bakar (`WEAPON_SELECT_DATA_JOB_WARRIOR` vb.). Eğer eşya, seçili sınıflardan birine aitse (`!IS_SET(item->GetAntiFlag(), ITEM_ANTIFLAG_*)`) `true` döner.
        *   `CheckTopicCommon` ve `CheckTopicCostume` gibi bazı fonksiyonlar, eşyanın daha spesifik alt türlerine (örn. ayakkabı, kolye, kemer) göre farklı ayar bayraklarını kontrol eder.
        *   `CheckTopicSkillBook`, beceri kitabının hangi sınıfa ait olduğunu `CSkillManager` üzerinden sorgulayarak ilgili sınıf filtresini uygular.
        *   Basit kategoriler (`CheckTopicDS`, `CheckTopicUnique`, `CheckTopicEtc`, `CheckTopicEvent`) genellikle sadece genel açma/kapama durumunu kontrol eder.
*   **Çalışma Prensibi:** Oyuncu, oyun arayüzünden ganimet filtresi ayarlarını yapar. Bu ayarlar `m_LootFilterSettings` dizisinde saklanır. Bir eşya yere düştüğünde ve oyuncu tarafından toplanma potansiyeli olduğunda, sistem `CanPickUpItem` fonksiyonunu bu eşya ile çağırır. Fonksiyon, eşyanın türüne ve oyuncunun ayarlarına göre eşyanın alınıp alınmayacağına karar verir. Eğer `CanPickUpItem` `false` dönerse, eşya muhtemelen otomatik olarak toplanmaz veya oyuncuya farklı şekilde gösterilir (örn. gizlenir ve `InsertLootFilteredItem` ile listeye eklenir).
*   **Bağlantılı Dosyalar:** `loot_filter.h`, `stdafx.h`, `affect.h`, `affect_flag.h`, `item.h`, `skill.h`.