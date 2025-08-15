# GameLib Referans Kılavuzu - Bölüm 3

Bu belge, `GameLib` kütüphanesindeki çeşitli C++ sınıflarının ve ilgili başlık dosyalarının ayrıntılı bir referansını sunar. Her bileşen, genel amacını, başlık dosyasındaki (.h) önemli tanımları, C++ dosyasındaki (.cpp) temel implementasyon prensiplerini ve oyundaki pratik kullanım senaryolarını açıklayacak şekilde belgelenmiştir.

Bu dosya, `client_GameLib_Referans_Part2.md` dosyasının devamı niteliğindedir ve GameLib kütüphanesinin belgelendirilmesine buradan devam edilmektedir.

## İçindekiler

* [`CItemManager` Sınıfı (`ItemManager.h`, `ItemManager.cpp`)](#citemmanager-sınıfı-itemmanagerh-itemmanagercpp)
* [Eşya Yardımcı Fonksiyonları (`ItemUtil.h`, `ItemUtil.cpp`)](#eşya-yardımcı-fonksiyonları-itemutilh-itemutilcpp)
* [`CMapBase` Sınıfı (`MapBase.h`, `MapBase.cpp`)](#cmapbase-sınıfı-mapbaseh-mapbasecpp)
* [`CMapManager` Sınıfı (`MapManager.h`, `MapManager.cpp`)](#cmapmanager-sınıfı-mapmanagerh-mapmanagercpp)
* [`CMapOutdoor` Sınıfı (`MapOutdoor.h`, `MapOutdoor.cpp` ve ilgili dosyalar)](#cmapoutdoor-sınıfı-mapoutdoorh-mapoutdoorcpp-ve-ilgili-dosyalar)
    * [Karakter Gölgeleri (`MapOutdoorCharacterShadow.cpp`)](#karakter-gölgeleri-mapoutdoorcharactershadowcpp)
    * [Arazi İndeks Tamponları (`MapOutdoorIndexBuffer.cpp`)](#arazi-indeks-tamponları-mapoutdoorindexbuffercpp)
    * [Arazi ve Alan Yükleme (`MapOutdoorLoad.cpp`)](#arazi-ve-alan-yükleme-mapoutdoorloadcpp)
    * [Dörtlü Ağaç Yönetimi (`MapOutdoorQuadtree.cpp`)](#dörtlü-ağaç-quadtree-yönetimi-mapoutdoorquadtreecpp)
    * [Çizim İşlevleri (`MapOutdoorRender.cpp`)](#çizim-i̇şlevleri-mapoutdoorrenderercpp)
    * [Donanım Dönüşümlü Arazi Çizimi (`MapOutdoorRenderHTP.cpp`)](#donanım-dönüşümlü-arazi-çizimi-mapoutdoorrenderhtpcpp)
    * [Yazılımsal Dönüşümlü Arazi Çizimi (`MapOutdoorRenderSTP.cpp`)](#yazılımsal-dönüşümlü-arazi-çizimi-mapoutdoorrenderstpcpp)
    * [Harita Güncelleme Mantığı (`MapOutdoorUpdate.cpp`)](#harita-güncelleme-mantığı-mapoutdoorupdatecpp)

---

## `CItemManager` Sınıfı (`ItemManager.h`, `ItemManager.cpp`)

### Dosyaların Genel Amacı

`CItemManager` sınıfı, `CSingleton` tasarım desenini kullanarak oyun istemcisindeki tüm eşya prototiplerini (`CItemData` nesnelerini) merkezi olarak yönetir. Eşya verilerini yüklemek, saklamak, bunlara erişim sağlamak ve Ejderha Taşı Sistemi gibi eşya ile ilişkili alt sistemlerin yapılandırmalarını yönetmekle görevlidir.

### `.h` - Başlık Dosyası Tanımları (Özet)

`ItemManager.h` dosyasında `CItemManager` sınıfının tanımı bulunur. Temel olarak şunları içerir:

*   **Singleton Erişimi:** `Instance()` ve `Destroy()` statik metotları.
*   **Eşya Veri Yapıları:**
    *   `m_ItemMap`: `std::map<DWORD, CItemData*>` tipinde, VNUM (eşya kodu) ile `CItemData` işaretçilerini eşler.
    *   `m_vec_ItemRange`: VNUM aralığı belirten eşyalar için referanslar tutar.
    *   `m_pSelectedItemData`: Seçili eşyanın `CItemData` işaretçisi.
*   **Yükleme Fonksiyonları:** `LoadItemTable()`, `LoadItemList()`, `LoadItemDesc()`, `LoadItemScale()`, `LoadAuraScale()`, `LoadDragonSoulTable()`.
*   **Erişim Fonksiyonları:** `GetItemDataPointer()`, `MakeItemData()`, `SelectItemData()`, `GetSelectedItemDataPointer()`.
*   **Ejderha Taşı Sistemi Entegrasyonu:** `CDragonSoulTable* m_pDSTable` ve ilgili erişim/yükleme fonksiyonları.
*   **Premium Özel Pazar Entegrasyonu:** (`ENABLE_PREMIUM_PRIVATE_SHOP` tanımlıysa) `m_NameItemMap` ve `GetItemByName()`, `GetItemListByName()` fonksiyonları.

### `.cpp` - C++ Implementasyon Detayları (Özet)

`ItemManager.cpp` dosyasında `CItemManager` sınıfının fonksiyonları implemente edilir:

*   **Kurucu/Yok Edici:** `CItemManager()` başlangıç değerlerini atar, `~CItemManager()` ise `Destroy()` fonksiyonunu çağırır. `Destroy()` metodu, `m_ItemMap` içindeki tüm `CItemData` örneklerini serbest bırakır ve `m_pDSTable`\'ı siler.
*   **Veri Yükleme:**
    *   `LoadItemTable()`: `item_proto` dosyasını (sıkıştırılmış ikili veya düz metin) okur, her satır için `MakeItemData()` ile `CItemData` oluşturur/alır ve `pItemData->SetItemTableData()` ile veriyi aktarır. İkonları bulmaya çalışır.
    *   `LoadItemList()`: `item_list.txt` gibi dosyalardan VNUM, tür, ikon ve model yolu bilgilerini okur, `CItemData::SetDefaultItemData()` ile ayarlar.
    *   `LoadItemDesc()`: `item_desc.txt` dosyasından eşya açıklamalarını yükler.
    *   `LoadItemScale()` / `LoadAuraScale()`: Acce ve Aura sistemleri için model/efekt ölçeklerini yükler.
    *   `LoadDragonSoulTable()`: `CDragonSoulTable` oluşturur ve Ejderha Taşı verilerini yükler.
*   **Erişim ve Yönetim:**
    *   `MakeItemData()`: Yeni bir `CItemData` oluşturur veya mevcut olanı döndürür.
    *   `GetItemDataPointer()`: Verilen VNUM\'a ait `CItemData` işaretçisini döndürür, VNUM aralıklarını dikkate alır.
    *   `ENABLE_PREMIUM_PRIVATE_SHOP` aktifse, `GetItemByName()` ve `GetItemListByName()` isimle eşya arama işlevselliği sunar.

### Oyunda Kullanım Amacı ve Senaryoları

`CItemManager`, istemcinin oyun içindeki tüm eşyalarla ilgili detaylı bilgilere erişebilmesi için merkezi bir otorite görevi görür.
*   **Envanter/Ekipman:** Envanterdeki bir eşyanın özelliklerini (isim, ikon, açıklama, efsunlar vb.) göstermek için `GetItemDataPointer()` kullanılır.
*   **Dünya Çizimi:** Bir karakterin elindeki silahın veya zırhının 3D modelini çizmek için eşyanın `CItemData`\'sından model bilgisi alınır.
*   **Eşya Kullanımı:** Bir iksir kullanıldığında veya bir beceri kitabı okunduğunda, ilgili eşyanın `CItemData`\'sındaki bilgiler (tür, etki vb.) kullanılır.
*   **Ticaret/Pazar:** Eşya bilgilerini arayüzde sergilemek için kullanılır.
*   **Ejderha Taşı Sistemi:** Ejderha Taşlarının özellikleri, efsunları ve set etkileri gibi bilgiler `CItemManager` aracılığıyla elde edilir.

Bu merkezi yönetim, eşya verilerinin tutarlı olmasını, kolayca güncellenebilmesini ve verimli bir şekilde yönetilmesini sağlar.

---

## Eşya Yardımcı Fonksiyonları (`ItemUtil.h`, `ItemUtil.cpp`)

### Dosyaların Genel Amacı

`ItemUtil.h` ve `ItemUtil.cpp` dosyaları, eşyalarla, özellikle eşya efsunları (bonusları/apply\'ları) ve Ejderha Taşı Sistemi ile ilgili çeşitli yardımcı tanımlamalar ve fonksiyonlar içerir. Temel amaçları, string tabanlı yapılandırma adlarını oyun içinde kullanılan sayısal enum değerlerine dönüştürmek ve bazı sistemler için standart sabitler sağlamaktır.

### `.h` - Başlık Dosyası Tanımları

`ItemUtil.h` dosyası, oyun içindeki belirli sistemler için standartlaştırılmış sabit değerler (enumlar) ve yapılar sunar:

*   **Enum Tanımları:**
    *   `EDragonSoulDeckType`: Ejderha Taşı kuşanma setlerini (`DS_DECK_1`, `DS_DECK_2`).
    *   `EDragonSoulGradeTypes`: Ejderha Taşı kalite seviyelerini (`DRAGON_SOUL_GRADE_NORMAL`'dan `DRAGON_SOUL_GRADE_MYTH`'e kadar).
    *   `EDragonSoulStepTypes`: Ejderha Taşı saflık (adım) seviyelerini (`DRAGON_SOUL_STEP_LOWEST`'ten `DRAGON_SOUL_STEP_HIGHEST`'e kadar).
    *   `EDragonSoulStrengthTypes`: Ejderha Taşı güçlendirme seviyeleri için sabit (`DRAGON_SOUL_STRENGTH_MAX`).
*   **`TValueName` Yapısı:**
    *   Bir string ismi (`c_pszName`) sayısal bir değerle (`lValue`, genellikle bir enum sabiti) eşleştirmek için kullanılır.
*   **Fonksiyon Prototipi:**
    *   `long GetApplyTypeByName(const char* c_pszApplyName)`: Bir efsun (apply) isminin string halini alır ve bu isme karşılık gelen `CItemData::EApplyTypes` enum değerini döndürür.

### `.cpp` - C++ Implementasyon Detayları (Özet)

`ItemUtil.cpp` dosyasının içeriği şöyledir:

*   **`c_aApplyTypeNames` Global Dizisi:**
    *   `TValueName` yapısını kullanan statik bir global dizidir. Oyundaki tüm eşya efsunlarının string isimlerini (örneğin, "MAX_HP", "STR") `CItemData.h` içinde tanımlanmış olan `CItemData::EApplyTypes` enum sabitleriyle eşleştirir.
    *   Bu dizi, temel efsunlardan özel sistemlere kadar geniş bir yelpazeyi kapsar ve `#if defined` blokları ile derleme zamanında farklı efsun setlerini içerebilir.
    *   Dizinin sonu `{ NULL, 0 }` ile belirtilir.
*   **`GetApplyTypeByName` Fonksiyonu:**
    *   Verilen bir string efsun adını alır.
    *   `c_aApplyTypeNames` dizisi üzerinde büyük/küçük harf duyarsız bir karşılaştırma (`stricmp`) yaparak eşleşen efsun adını arar.
    *   Eşleşme bulunursa, ilgili `lValue` (sayısal enum değeri) döndürülür.
    *   Eşleşme bulunamazsa 0 (genellikle `CItemData::APPLY_NONE`) döndürülür.

### Oyunda Kullanım Amacı ve Senaryoları

`ItemUtil` dosyaları, istemcinin eşya özelliklerini, özellikle de efsunları (item applies) işlemesi için temel bir katman sağlar:

*   **Yapılandırma Esnekliği:** `GetApplyTypeByName` sayesinde, efsunlar metin tabanlı dosyalarda (örn: `item_proto`, görev scriptleri) veya sunucu mesajlarında string olarak tanımlanabilir. İstemci bu stringleri kolayca kendi içsel enum formatına çevirerek okunabilirliği ve yönetimi artırır.
*   **Ejderha Taşı Sistemi:** `EDragonSoul*` enumları, Ejderha Taşı sisteminin farklı yönlerini (setler, kaliteler, saflık seviyeleri) kod içinde tutarlı ve standart bir şekilde ifade etmek için kullanılır.
*   **Kod Okunabilirliği ve Bakımı:** Efsun isimlerini merkezi bir dizide ve eşleştirme fonksiyonunda toplamak, yeni efsunlar eklendiğinde veya mevcutlar değiştirildiğinde kodun daha kolay yönetilmesine yardımcı olur.

---

## `CMapBase` Sınıfı (`MapBase.h`, `MapBase.cpp`)

### Dosyaların Genel Amacı

`CMapBase`, `CScreen` sınıfından türetilmiş soyut bir temel sınıftır. Oyun dünyasındaki farklı harita türleri (iç mekan `CMapIndoor`, dış mekan `CMapOutdoor`) için ortak bir arayüz ve temel işlevsellik (harita adı, türü, özellikleri yükleme, hazır olma durumu takibi, çevre verileriyle etkileşim) sağlar. Türetilmiş sınıflar, soyut metotları implemente ederek kendi özel yükleme, güncelleme ve çizim mantıklarını tanımlar.

### `.h` - Başlık Dosyası Tanımları

`MapBase.h` dosyası, `CMapBase` sınıfının tanımını ve ilgili enum ve yapıları içerir:

*   **`EMAPTYPE` Enum:** Haritanın türünü belirtir (`MAPTYPE_INVALID`, `MAPTYPE_INDOOR`, `MAPTYPE_OUTDOOR`).
*   **`SSungmaAttr` Yapısı:** (`ENABLE_SUNGMA_ATTR` tanımlıysa) Haritaya özel Sungma (Fetheder) niteliklerini (str, hp, move, immune) tutar.
*   **Temel Üye Değişkenler:**
    *   `m_eType`: `EMAPTYPE` türünde harita tipi.
    *   `m_strName`, `m_strParentMapName`: Harita ve ana harita adı.
    *   `m_bReady`: Haritanın yüklenip kullanıma hazır olup olmadığını gösteren bayrak.
    *   `mc_pEnvironmentData`: Mevcut çevre ayarlarına (`TEnvironmentData`) işaretçi.
    *   `m_bHasSungmaAttr`, `m_kSungmaAttr`: Sungma özellikleri bilgileri.
*   **Soyut (Pure Virtual) Fonksiyonlar:** Türetilmiş sınıflar tarafından implemente edilmelidir.
    *   `Initialize()`, `Destroy()`
    *   `Load(float x, float y, float z)`
    *   `Update(float fX, float fY, float fZ)`
    *   `UpdateAroundAmbience(float fX, float fY, float fZ)`
    *   `GetHeight(float fx, float fy)`
    *   `OnBeginEnvironment()`, `ApplyLight()`, `OnRender()`
    *   `OnSetEnvironmentDataPtr()`, `OnResetEnvironmentDataPtr()`
*   **Genel Fonksiyonlar:** `Clear()`, `Enter()`, `Leave()`, `Render()`, `SetEnvironmentDataPtr()`, `LoadProperty()`, `LoadSungMaAttr()`, `GetName()`, `GetType()`, `GetMapDataDirectory()`.

### `.cpp` - C++ Implementasyon Detayları (Özet)

`MapBase.cpp` dosyasında `CMapBase` sınıfının soyut olmayan metotları implemente edilir:

*   **Kurucu/Yok Edici:** Başlangıç değerlerini atar (`Clear()`).
*   **`Clear()`:** Üye değişkenleri varsayılan değerlerine sıfırlar.
*   **`Enter()` / `Leave()`:** `m_bReady` bayrağını ayarlar.
*   **`Render()`:** `m_bReady` ise `OnRender()` sanal fonksiyonunu çağırır.
*   **`SetEnvironmentDataPtr()`:** `mc_pEnvironmentData`'yı ayarlar ve `OnSetEnvironmentDataPtr()` / `OnResetEnvironmentDataPtr()` sanal metotlarını çağırır.
*   **`LoadProperty()`:** Haritanın (veya ana haritanın) `MapProperty.txt` dosyasını yükler. `CTokenVectorMap` ile "maptype" ve "parentmapname" gibi özellikleri okur. "parentmapname" varsa, harita verilerini o klasörden alır.
*   **`LoadSungMaAttr()`:** (`ENABLE_SUNGMA_ATTR` aktifse) Haritanın (veya ana haritanın) `sungma_attr.txt` dosyasını yükler ve Sungma özelliklerini (`m_kSungmaAttr`) doldurur.
*   **Erişim Metotları:** `GetName()`, `GetType()`, `GetMapDataDirectory()` gibi metotlar ilgili bilgileri döndürür.

### Oyunda Kullanım Amacı ve Senaryoları

`CMapBase`, Metin2 istemcisindeki farklı harita sistemlerinin (dış mekan, iç mekan) temelini oluşturur.
*   **Harita Yönetimi:** Oyun bir haritaya geçiş yaptığında, uygun `CMapBase` türevi (örn: `CMapOutdoor`) oluşturulur. Bu nesne, `CMapBase` arayüzü üzerinden yüklenir, güncellenir ve çizilir.
*   **Veri Yeniden Kullanımı:** `LoadProperty()` ve `parentmapname` mekanizması, aynı temel varlıkları kullanan farklı haritalar oluşturmayı sağlar (örn: aynı zindanın farklı görev versiyonları), disk alanından tasarruf edilir.
*   **Özelleştirilebilirlik:** `ENABLE_SUNGMA_ATTR` gibi derleme zamanı seçenekleri veya `MapProperty.txt` ve `sungma_attr.txt` gibi yapılandırma dosyaları, haritalara özel mekanikler ve özellikler eklemeyi kolaylaştırır.
*   **Soyutlama:** Farklı harita türleri için ortak bir arayüz sağlayarak kodun daha modüler ve genişletilebilir olmasına yardımcı olur.

---

## `CMapManager` Sınıfı (`MapManager.h`, `MapManager.cpp`)

### Dosyaların Genel Amacı

`CMapManager`, `CScreen` ve `IPhysicsWorld`'den türeyen, oyun istemcisindeki haritaların ve çevre (environment) ayarlarının merkezi yönetiminden sorumlu bir singleton benzeri sınıftır. Aktif harita nesnesini (şu anda öncelikli olarak `CMapOutdoor`) oluşturur, yükler, günceller, yönetir ve harita ile ilgili sorgulamaları (yükseklik, zemin özellikleri vb.) ilgili harita nesnesine yönlendirir. Ayrıca, genel harita bilgilerini (`AtlasInfo.txt`) ve çevre ayarlarını (`.msenv` dosyaları) yönetir.

### `.h` - Başlık Dosyası Tanımları

`MapManager.h` dosyası, `CMapManager` sınıfının tanımını ve ilgili yapıları içerir:

*   **`TMapInfo` Yapısı:** Bir haritanın temel bilgilerini (isim, dünya koordinatları, boyutlar) tutar.
*   **Üye Değişkenler:**
    *   `m_pkMap`: Aktif harita nesnesine (`CMapOutdoor*`) işaretçi.
    *   `mc_pcurEnvironmentData`: Aktif çevre verisine (`TEnvironmentData*`) işaretçi.
    *   `m_EnvironmentDataMap`: `std::map<DWORD, TEnvironmentData*>` ile indekslenmiş çevre verilerini saklar.
    *   `m_kVct_kMapInfo`: `std::vector<TMapInfo>` ile `AtlasInfo.txt`'den yüklenen harita bilgilerini tutar.
    *   `m_PropertyManager`: `CPropertyManager` nesnesi.
    *   `m_Forest`: `CSpeedTreeForestDirectX8` nesnesi (SpeedTree entegrasyonu).
    *   `m_stAtlasInfoFileName`: Atlas bilgi dosyasının adı.
*   **Harita Yönetimi Fonksiyonları:** `Create()`, `Destroy()`, `LoadMap()`, `UnloadMap()`, `UpdateMap()`.
*   **Çevre Yönetimi Fonksiyonları:** `RegisterEnvironmentData()`, `SetEnvironmentData()`, `SetEnvironmentDataPtr()`, `BeginEnvironment()`, `EndEnvironment()`, `GetCurrentEnvironmentData()`.
*   **Harita Bilgisi Sorgulama Fonksiyonları:** `GetHeight()`, `GetTerrainHeight()`, `GetWaterHeight()`, `GetNormal()`, `isAttrOn()`, `GetAttr()`, `GetShadowMapColor()`, `GetMapName()`, `GetMapInfoVector()`.
*   **Diğer Fonksiyonlar:** `ReserveSoftwareTilingEnable()`, `AllocMap()` (sanal), `RefreshPortal()`, `ClearPortal()`, `AddShowingPortalID()`.
*   **`IPhysicsWorld` Arayüzü Metotları:** `Get terreno altura()`, `isAttrOn()` gibi fiziksel dünya ile ilgili sorgular için implementasyonlar (çoğunlukla `m_pkMap`'e delege eder).

### `.cpp` - C++ Implementasyon Detayları (Özet)

`MapManager.cpp` dosyasında `CMapManager` sınıfının fonksiyonları implemente edilir:

*   **Kurucu/Yok Edici:** Başlangıç değerlerini atar. `Destroy()` metodu, tüm çevre verilerini siler ve `m_pkMap`'i yok eder.
*   **`Create()`:** `AllocMap()` (şu anda doğrudan `new CMapOutdoor()`) ile `m_pkMap`'i oluşturur ve `__LoadMapInfoVector()` ile `AtlasInfo.txt`'yi yükler.
*   **`LoadMap()`:**
    1.  Mevcut haritadan `Leave()` ile çıkar.
    2.  Yeni harita adını ayarlar, haritanın özelliklerini (`LoadProperty`, `LoadSungMaAttr`) yükler.
    3.  Harita türünü kontrol eder (şu an sadece `MAPTYPE_OUTDOOR` destekleniyor gibi).
    4.  `m_pkMap->Load()` ile harita verilerini yükler.
    5.  Haritanın kendi çevre ayar dosyasını (`.msenv`) varsayılan çevre olarak kaydeder ve aktif hale getirir.
    6.  Haritaya `Enter()` ile giriş yapar.
*   **Çevre Yönetimi:**
    *   `RegisterEnvironmentData()`: `.msenv` dosyasını yükler ve `m_EnvironmentDataMap`'e ekler.
    *   `SetEnvironmentData()` / `SetEnvironmentDataPtr()`: Aktif çevre verisini (`mc_pcurEnvironmentData`) ayarlar.
    *   `BeginEnvironment()`: Aktif çevrenin ışık, sis, materyal ayarlarını Direct3D'ye uygular.
    *   `EndEnvironment()`: `BeginEnvironment`'ta yapılan değişiklikleri geri alır.
*   **`__LoadMapInfoVector()`:** `AtlasInfo.txt` dosyasını okur ve `TMapInfo` yapılarını `m_kVct_kMapInfo` vektörüne doldurur.
*   **Sorgulama Fonksiyonları:** `GetHeight`, `GetNormal` gibi fonksiyonlar, çağrıları doğrudan `m_pkMap`'e yönlendirir.

### Oyunda Kullanım Amacı ve Senaryoları

`CMapManager`, istemcinin harita ve çevreyle ilgili tüm operasyonları için merkezi bir kontrol noktasıdır:

*   **Harita Geçişleri:** Oyuncu bir haritadan diğerine geçtiğinde, `CMapManager::LoadMap()` çağrılır. Bu fonksiyon yeni haritayı yükler, ilgili çevre ayarlarını aktif hale getirir.
*   **Dünya Atmosferi:** `BeginEnvironment()` ve `EndEnvironment()` fonksiyonları, her çizim karesinde o anki haritanın atmosferik koşullarını (ışıklandırma, sis, gökyüzü) ayarlar. Farklı haritalar veya aynı haritanın farklı bölgeleri/zamanları için farklı `.msenv` dosyaları kullanılarak özgün atmosferler yaratılabilir.
*   **Harita Bilgileri:** Oyunun herhangi bir yerinden o anki haritanın yüksekliği, zemin özellikleri gibi bilgilere `CMapManager` üzerinden erişilebilir.
*   **Genel Dünya Haritası:** `AtlasInfo.txt` ve `m_kVct_kMapInfo` sayesinde, oyun istemcisi tüm dünya haritalarının birbirine göre konumlarını ve boyutlarını bilir. Bu, genel bir dünya haritası arayüzü oluşturmak veya farklı haritalar arası geçiş mantığını yönetmek için kullanılabilir.
*   **SpeedTree Entegrasyonu:** `m_Forest` üyesi üzerinden SpeedTree ağaçlarının yönetimini koordine eder.

`CMapManager`, mevcut yapısıyla `CMapOutdoor`'a odaklansa da, `AllocMap` gibi sanal fonksiyonlar gelecekte farklı harita türlerini desteklemek için bir esneklik sunar.

---

## `CMapOutdoor` Sınıfı (`MapOutdoor.h`, `MapOutdoor.cpp` ve ilgili dosyalar)

### Dosyaların Genel Amacı

`CMapOutdoor`, `CMapBase` sınıfından türeyen ve Metin2 istemcisindeki dış mekan haritalarının tüm yönlerini yöneten karmaşık ve merkezi bir sınıftır. Arazi (terrain), alanlar (area - içindeki objeler, NPC'ler vb.), su, gökyüzü, ağaçlar (SpeedTree), gölgeler, çevresel efektler gibi unsurların yüklenmesi, güncellenmesi, çizilmesi ve etkileşiminden sorumludur. Performans ve görsel kalite için LOD, quadtree gibi optimizasyon teknikleri kullanır.

### `.h` - Başlık Dosyası Tanımları (Özet)

`MapOutdoor.h` dosyası, `CMapOutdoor` sınıfının tanımını, iç içe geçmiş yapıları, enumları ve çok sayıda üye değişkeni ile fonksiyon prototipini içerir. Başlıca öğeler:

*   **Yapılar/Enumlar:** `SEffectData`, `TOutdoorMapCoordinate`, `SHeightCache`, `DWORD` tabanlı parça görünürlük bayrakları (`PART_TERRAIN`, `PART_OBJECT` vb.).
*   **Temel Üye Değişkenler:**
    *   Arazi/Alan Yönetimi: `m_pTerrain[AROUND_AREA_NUM]`, `m_pArea[AROUND_AREA_NUM]`, `m_TerrainVector`, `m_AreaVector`.
    *   Koordinatlar: `m_CurCoordinate`, `m_PrevCoordinate`, `m_lCenterX`, `m_lCenterY`.
    *   Optimizasyon: `m_pRootNode` (`CTerrainQuadtreeNode*`), `m_kHeightCache`.
    *   Çizim Bileşenleri: `m_SkyBox`, `m_LensFlare`, `m_ScreenFilter`, `m_lpCharacterShadowMapTexture`.
    *   Ayarlar: `m_fHeightScale`, `m_lViewRadius`, `m_TextureSet`, `m_SnowTextureSet`.
    *   Görünürlük: `m_dwVisiblePartFlags`, `m_bSettingTerrainVisible`.
*   **Ana Fonksiyonlar (Prototip):**
    *   `CMapBase` Türevleri: `Load()`, `Update()`, `OnRender()`, `GetHeight()`, `Destroy()`, `Initialize()`.
    *   Yükleme: `LoadSetting()`, `LoadTerrain()`, `LoadArea()`, `LoadMonsterAreaInfo()`.
    *   Güncelleme: `UpdateTerrain()`, `__UpdateArea()`, `UpdateSky()`.
    *   Çizim: `RenderTerrain()`, `RenderArea()`, `RenderBlendArea()`, `RenderSky()`, `RenderCloud()`, `RenderTree()`, `RenderEffect()`.
    *   Gölgeler: `CreateCharacterShadowTexture()`, `BeginRenderCharacterShadowToTexture()`, `EndRenderCharacterShadowToTexture()`.
    *   Quadtree: `BuildQuadTree()`, `FreeQuadTree()`.
    *   Diğer: `SetVisiblePart()`, `isAttrOn()`, `GetNormal()`.
*   **Çok sayıda özel ve yardımcı fonksiyon prototipi** (özellikle çizim, güncelleme ve yükleme için).

### `.cpp` - C++ Implementasyon Detayları (Genel Özet)

`CMapOutdoor.cpp` ve ilişkili `MapOutdoor*.cpp` dosyaları, bu karmaşık sınıfın tüm işlevselliğini implemente eder.

*   **Kurulum ve Yıkım (`Initialize`, `Destroy`):** Kaynakları başlatır ve serbest bırakır. `Destroy()` tüm alt bileşenleri (arazi, alanlar, dokular vb.) temizler.
*   **Yükleme (`MapOutdoorLoad.cpp`):**
    *   `LoadSetting()`: Harita ayar dosyasını (`<map_name>.msa` veya `Setting.txt`) okur (boyut, yükseklik ölçeği, doku seti vb.).
    *   `Load()`: Ana yükleme fonksiyonu; `LoadSetting`'i çağırır, quadtree, arazi proxy'leri, su ve gölge dokularını hazırlar. Belirtilen koordinatlara göre `Update()` ile ilk yüklemeyi tetikler.
    *   `LoadTerrain()` / `LoadArea()`: Belirli koordinatlardaki arazi/alan sektörlerini diskten yükler, ilgili nesneleri (`CTerrain`, `CArea`) oluşturur ve yönetici vektörlere (`m_TerrainVector`, `m_AreaVector`) ekler.
*   **Güncelleme (`MapOutdoorUpdate.cpp`):**
    *   `Update()`: Ana güncelleme fonksiyonu. Oyuncu pozisyonuna göre aktif arazi/alan sektörlerini belirler, gerekirse yenilerini yükler/eskilerini kaldırır (dinamik yükleme). SpeedTree, gökyüzü, gölge alıcıları ve engelleyicileri günceller.
    *   `UpdateTerrain()`: Görünür arazi yamalarını belirler, quadtree yüksekliklerini günceller.
    *   `__UpdateArea()`: Aktif alanlardaki nesneleri, gölge alıcılarını ve PC engelleyicilerini günceller.
    *   Çöp toplama mekanizması (`__UpdateGarvage` vb.) kullanılmayan sektörleri zamanla siler.
*   **Çizim (`MapOutdoorRender.cpp`, `MapOutdoorRenderHTP.cpp`, `MapOutdoorRenderSTP.cpp`):**
    *   `OnRender()`: Ana çizim fonksiyonu. Görünürlük bayraklarına (`m_dwVisiblePartFlags`) göre gökyüzü, arazi, alanlar (objeler), su, ağaçlar, efektler vb. bileşenleri uygun sırada çizer.
    *   `RenderTerrain()`: Araziyi çizer. Quadtree ile görünür yamaları belirler, LOD seviyelerine göre sıralar. Donanımsal (`__RenderTerrain_RenderHardwareTransformPatch`) veya yazılımsal (`__RenderTerrain_RenderSoftwareTransformPatch`) çizim yollarını kullanır.
    *   `RenderArea()` / `RenderBlendArea()`: Opak ve harmanlanmış objeleri (alanlardan toplanan) çizer.
    *   Diğer `Render*` fonksiyonları belirli bileşenleri (gökyüzü, bulut, ağaçlar, efektler) çizer.
*   **Quadtree Yönetimi (`MapOutdoorQuadtree.cpp`):**
    *   `BuildQuadTree()`: Arazi parçalarını hiyerarşik bir yapıda organize eden dörtlü ağacı oluşturur.
    *   `SubDivideNode()`: Düğümleri özyineli olarak alt bölümlere ayırır.
    *   `FreeQuadTree()`: Ağacı siler.
*   **Karakter Gölgeleri (`MapOutdoorCharacterShadow.cpp`):** Render-to-texture tekniği ile dinamik karakter gölgelerini ayrı bir dokuya çizer.
*   **Arazi İndeks Tamponları (`MapOutdoorIndexBuffer.cpp`):** Farklı LOD seviyeleri için optimize edilmiş Direct3D indeks tamponları oluşturur, komşu LOD'lar arası "dikiş" (stitching) mantığını içerir.

### Oyunda Kullanım Amacı ve Senaryoları

`CMapOutdoor`, Metin2'nin geniş dış mekan dünyalarını hayata geçiren temel sınıftır.
*   **Dünya Oluşturma ve Yönetimi:** Oyuncu bir dış mekan haritasına girdiğinde, `CMapOutdoor` nesnesi bu dünyayı diskten yükler (arazi yükseklik haritaları, dokular, objeler, NPC yerleşimleri vb.).
*   **Dinamik Yükleme (Streaming):** Oyuncu haritada hareket ettikçe, `CMapOutdoor` sürekli olarak oyuncunun çevresindeki yeni arazi ve alan sektörlerini yükler, artık görünür olmayanları bellekten kaldırır. Bu, çok büyük haritaların bile verimli bir şekilde yönetilmesini sağlar.
*   **Görsel Çizim:** Gökyüzünü, araziyi (LOD ve splatting teknikleriyle), su yüzeylerini, binaları, ağaçları (SpeedTree kullanarak), karakter ve nesne gölgelerini, sis gibi atmosferik efektleri ekrana çizer.
*   **Etkileşim:** Karakterin yürüdüğü zeminin yüksekliğini (`GetHeight`), bir noktadaki zemin türünü (`GetAttr`) veya bir fare tıklamasının haritada hangi noktaya isabet ettiğini (`GetPickingPointWithRay`) belirlemek için kullanılır.
*   **Optimizasyon:** Performansı artırmak için dörtlü ağaç (frustum culling ve LOD için), yükseklik önbelleği, donanımsal/yazılımsal dönüşüm seçenekleri gibi çeşitli teknikler kullanır.

Kısacası, oyuncunun dış mekanlarda gördüğü, etkileşimde bulunduğu ve deneyimlediği her şey büyük ölçüde `CMapOutdoor` sınıfı tarafından yönetilir.

---

### `CMapOutdoor` Sınıfı – Bölüm Detayları

Aşağıdaki bölümler, `CMapOutdoor` sınıfının ana işlevselliklerini implemente eden belirli `.cpp` dosyalarına odaklanır.

#### Karakter Gölgeleri (`MapOutdoorCharacterShadow.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının karakterler gibi dinamik nesneler için gölgeler oluşturma mekanizmasını içerir. "Render-to-texture" tekniğini kullanarak, gölgeleri önce ayrı bir 2D dokuya (gölge haritası) çizer, ardından bu dokuyu ana sahnedeki yüzeylere (genellikle arazi üzerine) yansıtarak gölge efekti oluşturur.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili tanımlamalar:
*   `m_lpCharacterShadowMapTexture`: `LPDIRECT3DTEXTURE8`, gölge haritası dokusu.
*   `m_lpCharacterShadowMapRenderTargetSurface`, `m_lpCharacterShadowMapDepthSurface`: Gölge haritası için render hedefi ve derinlik tamponu yüzeyleri.
*   `m_wShadowMapSize`: Gölge haritasının boyutu.
*   `SetShadowTextureSize(WORD size)`
*   `BeginRenderCharacterShadowToTexture()`
*   `EndRenderCharacterShadowToTexture()`
*   `ReleaseCharacterShadowTexture()`
*   `CreateCharacterShadowTexture()`

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`SetShadowTextureSize()`**: Gölge haritası dokusunun çözünürlüğünü ayarlar. Boyut değişirse, dokunun yeniden oluşturulması için bayrak ayarlar.
*   **`CreateCharacterShadowTexture()`**: Mevcut dokuyu serbest bırakır, grafik kartı yeteneklerini kontrol eder ve belirtilen boyutta yeni bir render hedefi dokusu (`D3DFMT_R5G6B5`) ve derinlik tamponu (`D3DFMT_D16`) oluşturur. Gölge çizimi için viewport ayarlar.
*   **`ReleaseCharacterShadowTexture()`**: Oluşturulan Direct3D kaynaklarını serbest bırakır.
*   **`BeginRenderCharacterShadowToTexture()`**:
    1.  Gerekiyorsa gölge dokusunu yeniden oluşturur.
    2.  Sanal bir ışık kaynağının perspektifinden "View" ve "Projection" (genellikle ortografik) matrislerini ayarlar.
    3.  Ana sahne ışıklandırmasını kapatır, gölge rengini `D3DRS_TEXTUREFACTOR` ile ayarlar.
    4.  Render hedefini gölge haritası dokusuna ve derinlik tamponuna değiştirir.
    5.  Gölge haritasını ve derinlik tamponunu temizler.
*   **`EndRenderCharacterShadowToTexture()`**: Render hedefini, viewport'u ve diğer render durumlarını ana sahne çizimi için eski haline getirir.

##### Oyundaki Kullanım Amacı
Karakterler gibi hareketli nesneler için dinamik gölgeler oluşturur. Gölgeleri ayrı bir (genellikle daha düşük çözünürlüklü) dokuya çizmek, performansı artırabilir. Bu doku daha sonra araziye veya diğer uygun yüzeylere yansıtılır.

---

#### Arazi İndeks Tamponları (`MapOutdoorIndexBuffer.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının dış mekan arazisini farklı Detay Seviyelerinde (LOD) verimli bir şekilde çizmek için kullandığı Direct3D İndeks Tamponlarını (`Index Buffer`) oluşturma ve yönetme mantığını içerir. Temel amaç, farklı LOD'lardaki komşu arazi parçaları arasında görsel bütünlüğü (dikiş/stitching) sağlamaktır.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili tanımlamalar:
*   `m_IndexBuffer[TERRAINPATCH_LODMAX]`: `CGraphicIndexBuffer` tipinde, her LOD seviyesi için bir indeks tamponu dizisi.
*   `SetIndexBuffer()`: İndeks tamponlarını oluşturur.
*   Çok sayıda `ADDLvl*` benzeri özel fonksiyon prototipi (header'da olmayabilir, .cpp'ye özel olabilir).

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`SetIndexBuffer()`**:
    *   `TERRAINPATCH_LODMAX` ile tanımlanan sayıda LOD seviyesi için indeks tamponları oluşturur.
    *   Her LOD seviyesi için, standart bir arazi parçasını (`TERRAIN_PATCHSIZE` x `TERRAIN_PATCHSIZE`) üçgen şeritleri (triangle strips) kullanarak çizecek köşe indekslerini hesaplar.
    *   **LOD 0 (En Yüksek Detay):** Parçadaki her dörtgeni iki üçgenle kaplar.
    *   **Düşük LOD Seviyeleri (LOD 1, 2...):** Daha az üçgen kullanır. Komşu parçaların LOD seviyelerine bağlı olarak görsel bozuklukları (boşluklar, T-bağlantıları) önlemek için "dikiş" (stitching) algoritmaları kullanılır.
        *   Bu amaçla `ADDLvl1TL`, `ADDLvl1T`, ..., `ADDLvl2M` gibi özel fonksiyonlar, parçanın kenarlarını ve köşelerini komşu parçaların LOD'una göre farklı şekillerde üçgenleyerek "etek" (skirt) benzeri ek üçgenler oluşturur.
    *   Hesaplanan indeksler `CGraphicIndexBuffer` nesnelerine yüklenir.
*   **`ADDLvl*` Fonksiyon Ailesi**: `SetIndexBuffer()` içinde çağrılırlar. Belirli bir LOD seviyesinde, arazi parçasının belirli bir bölgesini (kenar, köşe, orta) komşu parçaların LOD durumlarına göre doğru şekilde üçgenleyecek köşe indekslerini üretirler.

##### Oyundaki Kullanım Amacı
Geniş dış mekan haritalarında performansı optimize etmek için LOD sistemi kullanılır. Bu dosyadaki mantık, farklı LOD seviyelerindeki arazi parçalarının birbirine sorunsuz bir şekilde bağlanmasını sağlar. Arazi çizimi sırasında, her parçanın LOD'una uygun, önceden hesaplanmış bu indeks tamponları kullanılır.

---

#### Arazi ve Alan Yükleme (`MapOutdoorLoad.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının dış mekan haritalarının temel ayarlarını, bireysel arazi (terrain) sektörlerini, alan (area) sektörlerini (içerdikleri objelerle birlikte), çevre verilerini ve canavar ortaya çıkma (spawn) bilgilerini diskten yüklemekten sorumlu mantığı içerir.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili fonksiyon prototipleri:
*   `Load(float x, float y, float z)`
*   `LoadSetting(const char* c_szFileName)`
*   `LoadTerrain(WORD wTerrainCoordX, WORD wTerrainCoordY, const char* c_szAreaName, const char* c_szTerrainName)`
*   `LoadArea(WORD wAreaCoordX, WORD wAreaCoordY, const char* c_szAreaName)`
*   `LoadMonsterAreaInfo()` (`WORLD_EDITOR` için)
*   `GetEnvironmentDataName()`
*   `isTerrainLoaded(WORD wX, WORD wY)`, `isAreaLoaded(WORD wX, WORD wY)`
*   `AssignTerrainPtr()`
*   `ChangeTextureset(const std::string& FileName)` (`ENVIRONMENT_SYSTEM` için)

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`Load()`**: Ana yükleme fonksiyonu. Mevcut verileri temizler (`Destroy()`), `LoadSetting()` ile harita ayarlarını yükler, quadtree ve arazi proxy'lerini oluşturur, su/gölge dokularını hazırlar ve `Update()` ile ilk sektör yüklemesini tetikler.
*   **`LoadSetting()`**: `<map_name>/Setting.txt` (veya `.msa`) dosyasından harita boyutu, yükseklik ölçeği, doku seti adı, varsayılan çevre dosyası gibi temel ayarları okur. Doku setlerini (`m_TextureSet`, `m_SnowTextureSet`) yükler.
*   **`LoadTerrain()`**: Belirtilen koordinattaki arazi sektörünü yükler. `<map_data_dir>/<ID>/AreaProperty.txt` okur, `CTerrain` nesnesi oluşturur ve `height.raw`, `tile.raw`, `attr.atr`, `water.wtr`, `shadowmap.dds`/`.raw`, `minimap.dds` gibi dosyaları yükler.
*   **`LoadArea()`**: Belirtilen koordinattaki alan sektörünü yükler. `CArea` nesnesi oluşturur ve `pArea->Load()` ile o alana ait objeleri vb. yükler.
*   **`AssignTerrainPtr()`**: Oyuncunun mevcut pozisyonuna göre aktif 3x3 arazi/alan sektörlerine (`m_pTerrain`, `m_pArea`) işaretçileri atar.
*   **`LoadMonsterAreaInfo()`**: (`WORLD_EDITOR` aktifse) `regen.txt` dosyasından canavar spawn bilgilerini okur, `CMonsterAreaInfo` nesneleri oluşturur.
*   **`isTerrainLoaded()` / `isAreaLoaded()`**: Sektörün daha önce yüklenip yüklenmediğini kontrol eder.

##### Oyundaki Kullanım Amacı
Harita verilerini diskten yükleyerek oyun dünyasını oluşturur. Bu, haritanın temel yapılandırmasından (boyut, dokular), görsel detaylarına (arazi, objeler), ve oyun mekaniklerine (canavar spawnları) kadar geniş bir yelpazeyi kapsar. Yükleme işlemleri genellikle oyuncu haritaya girdiğinde veya harita üzerinde hareket ettikçe dinamik olarak (streaming) tetiklenir.

---

#### Dörtlü Ağaç (Quadtree) Yönetimi (`MapOutdoorQuadtree.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının dış mekan arazisinin optimizasyonu için kullandığı dörtlü ağaç (quadtree) yapısını oluşturma (`BuildQuadTree`), özyineli olarak alt bölümlere ayırma (`SubDivideNode`, `AllocQuadTreeNode`) ve serbest bırakma (`FreeQuadTree`) işlevlerini içerir. Dörtlü ağaçlar, görünürlük tespiti (frustum culling) ve Detay Seviyesi (LOD) yönetimi için kullanılır.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili tanımlamalar:
*   `m_pRootNode`: `CTerrainQuadtreeNode*` tipinde, dörtlü ağacın kök düğümü.
*   `BuildQuadTree()`
*   `AllocQuadTreeNode(long x0, long y0, long x1, long y1)`
*   `SubDivideNode(CTerrainQuadtreeNode* Node)`
*   `FreeQuadTree()`

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`BuildQuadTree()`**: Ana inşa fonksiyonu. Mevcut ağacı temizler (`FreeQuadTree`), tüm araziyi kapsayan kök düğümü (`m_pRootNode`) `AllocQuadTreeNode` ile oluşturur ve `SubDivideNode` ile ağacı alt bölümlere ayırır.
*   **`AllocQuadTreeNode()`**: Yeni bir `CTerrainQuadtreeNode` oluşturur, kapsadığı alanın köşe koordinatlarını, boyutunu ve sol üst köşesine karşılık gelen parça numarasını ayarlar. (Merkez/yarıçap hesaplaması kısıtlı).
*   **`SubDivideNode()`**: Verilen bir düğümü dört alt düğüme (NW, NE, SW, SE) böler. Her alt düğüm için `AllocQuadTreeNode` çağırır ve eğer alt düğüm hala yeterince büyükse, özyineli olarak `SubDivideNode` ile tekrar böler.
*   **`FreeQuadTree()`**: `m_pRootNode` ile temsil edilen ağaç yapısını siler. (Not: Mevcut implementasyonun derin ağaçlarda bellek sızıntısı riski olabilir, `CTerrainQuadtreeNode` yıkıcısının davranışına bağlıdır).

##### Oyundaki Kullanım Amacı
Dış mekan arazilerini verimli yönetmek için kullanılır:
1.  **Görünürlük Tespiti (Frustum Culling):** Kamera tarafından görülmeyen ağaç düğümleri (ve içerdikleri arazi parçaları) çizimden çıkarılır.
2.  **Detay Seviyesi (LOD) Yönetimi:** Kameraya uzak düğümler düşük detayda, yakın olanlar yüksek detayda çizilebilir.

Bu optimizasyonlar, büyük haritalarda akıcı bir oyun deneyimi için önemlidir.

---

#### Çizim İşlevleri (`MapOutdoorRender.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının dış mekan haritalarının arazi, harita nesneleri (alanlar), gökyüzü, bulutlar, ağaçlar, su ve özel efektler gibi çeşitli görsel bileşenlerinin ekrana çizilmesiyle (rendering) ilgili temel mantığı içerir. Çizim sırasını belirler ve özel çizim fonksiyonlarını çağırır.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili fonksiyon prototipleri:
*   `OnRender()`
*   `RenderTerrain()`, `RenderArea(bool bRenderAmbience = false)`, `RenderBlendArea()`
*   `RenderSky()`, `RenderCloud()`, `RenderTree()`, `RenderEffect()`, `RenderWater()`
*   `RenderScreenFiltering()`, `RenderBeforeLensFlare()`, `RenderAfterLensFlare()`
*   `SetVisiblePart(DWORD dwPartID, bool isVisible)`, `IsVisiblePart(DWORD dwPartID)`
*   `SelectIndexBuffer(BYTE byLODLevel, WORD & wPrimitiveCount, D3DPRIMITIVETYPE & ePrimitiveType)`
*   Çok sayıda yardımcı ve özel çizim fonksiyonu prototipi.

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`OnRender()`**: Ana çizim döngüsü. Kamera ve gölge matrislerini ayarlar, harmanlama işlemlerini yapar ve sırasıyla `RenderArea` (opak objeler), `RenderTree`, `RenderTerrain`, `RenderWater`, `RenderBlendArea` (harmanlanmış objeler), `RenderSky`, `RenderCloud`, `RenderEffect` gibi fonksiyonları çağırır. Görünürlük bayraklarına (`m_dwVisiblePartFlags`) göre çizim yapar.
*   **`RenderTerrain()`**: Araziyi çizer. Görüş alanı (frustum) kontrolü yapar, `__RenderTerrain_RecurseRenderQuadTree` ile görünür quadtree düğümlerini/yamalarını toplar, mesafeye göre sıralar ve donanımsal (`MapOutdoorRenderHTP.cpp`) veya yazılımsal (`MapOutdoorRenderSTP.cpp`) yama çizim fonksiyonlarını çağırır.
*   **`__RenderTerrain_RecurseRenderQuadTree()`**: Quadtree'de özyineli dolaşarak görünür arazi yamalarını belirler ve `m_PatchVector`'e ekler.
*   **`RenderArea()`**: Opak harita nesnelerini (alanlardan toplanan) çizer. Gölgeler aktifse, karakter gölgelerini gölge alıcı nesnelere uygular. Nesneler kameraya yakınlığa göre sıralanır.
*   **`RenderBlendArea()`**: Alfa harmanlaması gerektiren (transparan) nesneleri çizer. Nesneler sıralanır ve uygun render durumları ayarlanır.
*   **Diğer `Render*` Fonksiyonları**: `m_SkyBox` (gökyüzü, bulut), `CSpeedTreeForestDirectX8` (ağaçlar), `m_ScreenFilter`, `m_LensFlare` gibi ilgili sınıfların çizim metotlarını çağırır veya alanlardaki efektleri/çarpışma geometrilerini çizer. `RenderWater()` su çizimini yönetir.
*   **Görünürlük Yönetimi**: `SetVisiblePart()` ile haritanın hangi kısımlarının çizileceği kontrol edilir.

##### Oyundaki Kullanım Amacı
Dış mekan haritalarının görsel zenginliğini oluşturur. Detaylı arazi, karmaşık objeler, atmosferik efektler ve optimizasyon teknikleri (LOD, frustum culling) burada bir araya gelir. Oyuncunun dünyayı deneyimlediği şekli doğrudan etkiler, performansı dengelerken görsel kaliteyi hedefler.

---

#### Donanım Dönüşümlü Arazi Çizimi (`MapOutdoorRenderHTP.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının, GPU'nun dönüşüm ve aydınlatma (Transform & Lighting - T&L) yeteneklerini kullanarak arazi yamalarını (Hardware Transform Patch - HTP) çizmesinden sorumlu fonksiyonları içerir. Bu, CPU yükünü azaltır ve genellikle daha yüksek performans sağlar.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili fonksiyon prototipi:
*   `__RenderTerrain_RenderHardwareTransformPatch()`
*   `__HardwareTransformPatch_RenderPatchSplat(long patchnum, WORD wPrimitiveCount, D3DPRIMITIVETYPE ePrimitiveType)`
*   `__HardwareTransformPatch_RenderPatchNone(long patchnum, WORD wPrimitiveCount, D3DPRIMITIVETYPE ePrimitiveType)`

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`__RenderTerrain_RenderHardwareTransformPatch()`**: Ana HTP çizim fonksiyonu.
    *   Sis parametrelerini alır, Direct3D render/texture state'lerini ayarlar (doku koordinatları için kamera uzayı, alfa test/blend, filtreleme).
    *   Çizilecek yamaları mesafeye göre üç bölgeye ayırır: sis yok (yakın), sisli (orta), sadece sis rengi (uzak).
    *   Her bölge için LOD seviyesini belirler, `SelectIndexBuffer` ile uygun indeks tamponunu seçer.
    *   Yakın/orta bölgeler için `__HardwareTransformPatch_RenderPatchSplat()`, uzak bölge için `__HardwareTransformPatch_RenderPatchNone()` çağrılır.
    *   Render state'leri geri yükler.
*   **`__HardwareTransformPatch_RenderPatchSplat()`**: Belirli bir arazi yamasını doku katmanları (splats) ile çizer.
    *   Arazi parçasının dünya koordinatlarına göre doku dönüşüm matrislerini (`matTexTransform`, `matSplatAlphaTexTransform`, `matSplatColorTexTransform`) hesaplar ve doku aşamalarına atar.
    *   Yamanın donanım dönüşümlü köşe tamponunu (`CTerrainPatchProxy::HardwareTransformPatch_GetVertexBufferPtr()`) ayarlar.
    *   Her aktif doku katmanı (splat) için:
        *   Kar efekti aktifse kar dokularını (`m_SnowTextureSet`), değilse normal dokuları (`m_TextureSet`) kullanır.
        *   Ana doku ve splat alfa dokusunu ayarlar, `STATEMANAGER.DrawIndexedPrimitive()` ile yamayı çizer.
    *   Gölgeler aktifse (`m_bDrawShadow`), ışıklandırmayı açar, harmanlama modlarını ayarlar, statik ve dinamik gölge matrislerini/dokularını kullanarak yamayı tekrar çizer (gölgelerle).
*   **`__HardwareTransformPatch_RenderPatchNone()`**: Uzak mesafedeki, sadece sis rengiyle çizilecek yamaları dokusuz çizer.

##### Oyundaki Kullanım Amacı
GPU'nun gücünden yararlanarak dış mekan arazilerini verimli ve görsel olarak zengin çizer.
*   **Performans:** Köşe dönüşümlerini GPU'ya yaptırarak CPU yükünü azaltır.
*   **Görsel Kalite:** LOD, splatting, sis ve gölge efektleriyle detaylı haritaların akıcı görüntülenmesini sağlar.
*   **Esneklik:** `WORLD_EDITOR` için özel çizim modları ve kar efekti gibi çevresel etkiler için farklı doku setlerini destekler.

---

#### Yazılımsal Dönüşümlü Arazi Çizimi (`MapOutdoorRenderSTP.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının "Software Transform Patch" (STP) tekniğiyle ilgili fonksiyonlarını içerir. Donanımsal T&L yetenekleri sınırlı veya olmayan grafik kartlarında daha iyi performans için arazi yamalarının CPU üzerinde işlenmesini (köşe dönüşümleri, aydınlatma, sis) sağlar.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili yapılar ve fonksiyon prototipleri:
*   `SoftwareTransformPatch_SSplatVertex` ve `SoftwareTransformPatch_STLVertex` yapıları.
*   `SoftwareTransformPatch_SRenderState` yapısı (çizim boru hattı verilerini tutar).
*   `m_kSTPD`: `SoftwareTransformPatch_SData` tipinde, STP için dinamik vertex buffer'ları ve diğer verileri tutan üye.
*   `__RenderTerrain_RenderSoftwareTransformPatch()`
*   Ve çok sayıda `__SoftwareTransformPatch_*` önekli yardımcı fonksiyon.

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`__RenderTerrain_RenderSoftwareTransformPatch()`**: Ana STP çizim fonksiyonu.
    *   Render durumlarını (`__SoftwareTransformPatch_ApplyRenderState`) ve çizim verilerini (`__SoftwareTransformPatch_BuildPipeline`) hazırlar.
    *   Yamaları mesafeye göre sıralar, LOD seviyelerine göre indeks tamponlarını seçer.
    *   Yamaları sise yakın ve sisli olarak iki grupta işler, her ikisi için de `__SoftwareTransformPatch_RenderPatchSplat()` çağırır (sis parametresi farklı).
    *   Hızlı T&L desteklenmiyorsa, en uzaktaki yamalar için `__SoftwareTransformPatch_RenderPatchNone()` çağrılır.
    *   Render durumlarını geri yükler.
*   **`__SoftwareTransformPatch_RenderPatchSplat()`**: Belirli bir arazi yamasını kaplamalarıyla (splatting) çizer.
    *   `__SoftwareTransformPatch_SetTransform()` ile yamanın köşe noktalarını CPU'da ekran koordinatlarına dönüştürür, aydınlatma ve sis hesaplar.
    *   `__SoftwareTransformPatch_SetSplatStream()` ile dönüştürülmüş köşe verilerini dinamik vertex buffer'a (`m_kSTPD.m_pkVBSplat`) yükler.
    *   Her kaplama için dokuları ayarlar ve `STATEMANAGER.DrawIndexedPrimitive()` ile çizer.
    *   Gölgeler aktifse, `__SoftwareTransformPatch_SetShadowStream()` ile gölge verilerini hazırlar, gölge render durumlarını ayarlar ve yamayı gölge dokularıyla tekrar çizer.
*   **`__SoftwareTransformPatch_RenderPatchNone()`**: Uzak yamaları kaplamasız, sadece temel renkle çizer. Köşeleri CPU'da dönüştürür ve `m_kSTPD.m_pkVBNone` buffer'ına yükler.
*   **`__SoftwareTransformPatch_SetTransform()`**: CPU üzerinde köşe dönüşümü, aydınlatma, doku koordinat hesaplaması ve sis etkisi uygular.
*   **`__SoftwareTransformPatch_SetSplatStream()` / `__SoftwareTransformPatch_SetShadowStream()`**: Hesaplanan köşe verilerini uygun formatlarda dinamik vertex buffer'lara yükler.
*   **Vertex Buffer Yönetimi (`__SoftwareTransformPatch_Create()`, `Destroy()`):** STP için `D3DPOOL_SYSTEMMEM` havuzunda dinamik vertex buffer'lar oluşturur ve siler.

##### Oyundaki Kullanım Amacı
Eski veya düşük seviye donanımlarda dış mekan haritalarının verimli çizilebilmesini sağlar. CPU'yu kullanarak GPU yükünü dengelemeye yardımcı olur. Donanımsal T&L desteklenmediğinde veya yavaş olduğunda alternatif bir çizim yolu sunar.

---

#### Harita Güncelleme Mantığı (`MapOutdoorUpdate.cpp`)

##### Dosyanın Genel Amacı
Bu dosya, `CMapOutdoor` sınıfının dış mekan haritasının durumunu her oyun döngüsünde (frame) güncellemekten sorumlu fonksiyonları içerir. Bu, oyuncu pozisyonuna göre arazi/alanların dinamik yüklenmesi/kaldırılması, gölge alıcıları ve görüş engelleyicilerin belirlenmesi, gökyüzü güncellemeleri ve çöp toplama gibi optimizasyonları kapsar.

##### `.h` - Başlık Dosyası Tanımları (İlgili Kısımlar)
`MapOutdoor.h` içinde ilgili fonksiyon prototipleri:
*   `Update(float fX, float fY, float fZ)` (CMapBase'den override)
*   `UpdateTerrain(float fX, float fY)`
*   `UpdateSky()`
*   `UpdateAroundAmbience(float fX, float fY, float fZ)` (CMapBase'den override)
*   `ConvertTerrainToTnL(long lx, long ly)`
*   `UpdateQuadTreeHeights(CTerrainQuadtreeNode* Node)`
*   `AssignPatch(long lPatchNum, long x0, long y0, long x1, long y1)`
*   `__UpdateArea(D3DXVECTOR3& v3Player)`
*   Ve çöp toplama, PC blocker/shadow receiver toplama ile ilgili çeşitli yardımcı fonksiyon ve funktör tanımları/prototipleri.

##### `.cpp` - C++ Implementasyon Detayları (Özet)
*   **`Update()`**: Ana güncelleme fonksiyonu.
    *   Oyuncunun mevcut arazi sektörünü belirler.
    *   Oyuncu farklı bir yükleme bölgesine (`LOAD_SIZE_WIDTH`) geçmişse, yeni referans koordinatlarına göre çevredeki arazi (`LoadTerrain`) ve alanları (`LoadArea`) yükler/günceller, `AssignTerrainPtr()` ile aktif işaretçileri ayarlar.
    *   SpeedTree (`UpdateSystem()`), çöp toplama (`__UpdateGarvage`), arazi yamaları (`UpdateTerrain`), alanlar/objeler (`__UpdateArea`), gökyüzü (`UpdateSky`) ve yükseklik önbelleğini (`__HeightCache_Update`) günceller.
*   **`UpdateTerrain()`**: Oyuncunun pozisyonuna göre merkez arazi yamasını belirler. Merkez değişmişse `ConvertTerrainToTnL()` ile görünür yamaları hazırlar ve `UpdateQuadTreeHeights()` ile quadtree'yi günceller.
*   **`ConvertTerrainToTnL()`**: Görüş yarıçapına göre görünür arazi yamalarını belirler, proxy yamalarına doğru arazi/yama numaralarını atar (`AssignPatch`).
*   **`UpdateQuadTreeHeights()`**: Quadtree düğümlerinin sınırlayıcı küre (merkez, yarıçap) bilgilerini kapladıkları aktif yamalara göre günceller.
*   **`__UpdateArea()`**: `__Game_UpdateArea` (oyun sırasında) veya `__NEW_WorldEditor_UpdateArea` (editörde) fonksiyonlarını çağırır.
    *   `__Game_UpdateArea`: `m_PCBlockerVector` ve `m_ShadowReceiverVector` listelerini temizler. `__CollectShadowReceiver()` ile gölge alıcılarını, `__CollectCollisionPCBlocker()` ile kamera-oyuncu arası engelleyicileri toplar. `__UpdateAroundAreaList()` ile aktif alanların kendi güncellemelerini tetikler.
*   **Çöp Toplama Mekanizması (`__ClearGarvage`, `__UpdateGarvage`, `UpdateAreaList` vb.)**: (`REMOVING_COLLECTORS` tanımlı değilse) Görüş alanı dışına çıkan arazi/alan sektörlerini hemen silmek yerine bir listeye ekler ve periyodik olarak bu listeden birer eleman silerek performansı dengeler.

##### Oyundaki Kullanım Amacı
Oyun dünyasının canlı ve dinamik kalmasını sağlar. Oyuncunun çevresindeki dünyanın sürekli olarak yüklenmesi, güncellenmesi ve gereksiz kısımların temizlenmesi (streaming) gibi işlemler akıcı bir oyun deneyimi için hayati öneme sahiptir. Ayrıca, gölge ve görünürlük (occlusion) hesaplamaları için gerekli verileri toplayarak çizim (render) aşamasına hazırlar.