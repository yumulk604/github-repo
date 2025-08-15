# GameLib Referans Kılavuzu - Bölüm 2

Bu belge, `GameLib` kütüphanesindeki çeşitli C++ sınıflarının ve ilgili başlık dosyalarının ayrıntılı bir referansını sunar. Her bileşen, genel amacını, başlık dosyasındaki (.h) önemli tanımları, C++ dosyasındaki (.cpp) temel implementasyon prensiplerini ve oyundaki pratik kullanım senaryolarını açıklayacak şekilde belgelenmiştir.

Bu dosya, `client_GameLib_Referans.md` dosyasının devamı niteliğindedir ve GameLib kütüphanesinin belgelendirilmesine buradan devam edilmektedir.

## İçindekiler

* [`CArea` Sınıfı (`Area.h` ve `Area.cpp`)](#carea-sınıfı-areah-ve-areacpp)
* [`TEMP_CAreaLoaderThread` Sınıfı (`AreaLoaderThread.h` ve `AreaLoaderThread.cpp`)](#temp_carealoaderthread-sınıfı-arealoaderthreadh-ve-arealoaderthreadcpp)
* [`CTerrain` Sınıfı (`AreaTerrain.h` ve `AreaTerrain.cpp`)](#cterrain-sınıfı-areaterrainh-ve-areaterraincpp)
* [`CDragonSoulTable` Sınıfı (`DragonSoulTable.h` ve `DragonSoulTable.cpp`)](#cdragonsoultable-sınıfı-dragonsoultableh-ve-dragonsoultablecpp)
* [`CDungeonBlock` Sınıfı (`DungeonBlock.h` ve `DungeonBlock.cpp`)](#cdungeonblock-sınıfı-dungeonblockh-ve-dungeonblockcpp)
* [`IFlyEventHandler` Arayüzü (`FlyHandler.h`)](#iflyeventhandler-arayüzü-flyhandlerh)
* [`CFlyingData` Sınıfı (`FlyingData.h` ve `FlyingData.cpp`)](#cflyingdata-sınıfı-flyingdatah-ve-flyingdatacpp)
* [`CFlyingInstance` Sınıfı (`FlyingInstance.h` ve `FlyingInstance.cpp`)](#cflyinginstance-sınıfı-flyinginstanceh-ve-flyinginstancecpp)
* [`CFlyingManager` Sınıfı (`FlyingObjectManager.h` ve `FlyingObjectManager.cpp`)](#cflyingmanager-sınıfı-flyingobjectmanagerh-ve-flyingobjectmanagercpp)
* [`IFlyTargetableObject` Arayüzü (`FlyTarget.h`)](#iflytargetableobject-arayüzü-flytargeth)
* [`CFlyTarget` Sınıfı (`FlyTarget.h` ve `FlyTarget.cpp`)](#cflytarget-sınıfı-flytargeth-ve-flytargetcpp)
* [`CFlyTrace` Sınıfı (`FlyTrace.h` ve `FlyTrace.cpp`)](#cflytrace-sınıfı-flytraceh-ve-flytracecpp)
* [`CGameEventManager` Sınıfı (`GameEventManager.h` ve `GameEventManager.cpp`)](#cgameeventmanager-sınıfı-gameeventmanagerh-ve-gameeventmanagercpp)
* [`GameType.h` ve `GameType.cpp` Dosyaları](#gametypeh-ve-gametypecpp-dosyaları)
* [`GameUtil.h` ve `GameUtil.cpp` Dosyaları](#gameutilh-ve-gameutilcpp-dosyaları)
* [`IBackground` Arayüzü (`Interface.h`)](#ibackground-arayüzü-interfaceh)
* [`CItemData` Sınıfı (`ItemData.h` ve `ItemData.cpp`)](#citemdata-sınıfı-itemdatah-ve-itemdatacpp)

### `CArea` Sınıfı (`Area.h` ve `Area.cpp`)

**Sınıfın Genel Amacı**
`CArea` sınıfı, oyun dünyasındaki belirli bir coğrafi alanı veya sektörü temsil eder. Bu alan içindeki tüm statik ve dinamik çevresel nesnelerin (binalar, ağaçlar, dekorasyonlar, efektler, çevre sesleri, zindan blokları vb.) yüklenmesinden, yönetilmesinden, güncellenmesinden ve render edilmesinden sorumludur. Bir dış mekan haritası (`CMapOutdoor`) genellikle birden fazla `CArea` nesnesinden oluşur ve oyuncu haritada hareket ettikçe bu alanlar yüklenip kaldırılır.

**Başlık Dosyası Tanımları (.h)**

*   **Temel Veri Yapıları:**
    *   **`CArea::SObjectData`**: Bir alandaki her bir "statik" nesnenin başlangıç verilerini tutar. İçeriği:
        *   `Position`: Nesnenin 3D pozisyonu.
        *   `dwCRC`: Nesnenin özelliklerini tanımlayan `CProperty` nesnesine bir CRC (Cyclic Redundancy Check) referansı. Bu property, nesnenin model dosyasını, türünü vb. içerir.
        *   `abyPortalID[PORTAL_ID_MAX_NUM]`: Nesnenin ilişkili olduğu portal ID'leri.
        *   `m_fYaw`, `m_fPitch`, `m_fRoll`: Nesnenin başlangıç rotasyonu.
        *   `m_fHeightBias`: Nesnenin Z eksenindeki yükseklik sapması.
        *   `dwRange`, `fMaxVolumeAreaPercentage`: Ambiyans (çevre sesi/efekti) nesneleri için etki menzili ve maksimum ses seviyesinin uygulanacağı iç menzil yüzdesi.
    *   **`CArea::SObjectInstance`**: `SObjectData`'dan oluşturulan, oyun dünyasında yaşayan bir nesne örneğini temsil eder. İçeriği:
        *   `dwType`: Nesnenin türü (ağaç, bina, efekt, ambiyans, zindan bloğu - `prt::PROPERTY_TYPE_*`).
        *   `pAttributeInstance`: Nesnenin çarpışma ve diğer fiziksel özelliklerini tutan `CAttributeInstance` işaretçisi.
        *   `pTree`: Eğer nesne bir ağaç ise `SpeedTreeWrapperPtr`.
        *   `pThingInstance`: Eğer nesne bir bina veya genel bir model ise `CGraphicThingInstance` işaretçisi.
        *   `isShadowFlag`: Nesnenin gölge oluşturup oluşturmadığı.
        *   `dwEffectID`, `dwEffectInstanceIndex`: Eğer nesne bir efekt ise, efektin ID'si ve `m_EffectInstanceMap` içindeki indeksi.
        *   `pAmbienceInstance`: Eğer nesne bir ambiyans ise `CArea::SAmbienceInstance` işaretçisi.
        *   `pDungeonBlock`: Eğer nesne bir zindan bloğu ise `CDungeonBlock` işaretçisi.
    *   **`CArea::SAmbienceInstance`**: Bir çevre sesi veya efektini yönetir. Pozisyon, menzil, ses çalma türü (`ONCE`, `STEP`, `LOOP`) ve ses dosyası bilgilerini içerir.

*   **Bellek Yönetimi (Statik Havuzlar):**
    *   `CArea`, `TObjectInstance`, `CAttributeInstance`, `TAmbienceInstance`, `CDungeonBlock` gibi sık oluşturulup yok edilen nesneler için statik `CDynamicPool`'ler kullanılır.
    *   Bu havuzlar `CArea::New()`, `CArea::Delete()` ve `CArea::DestroySystem()` metodları ile yönetilir.

**C++ Implementasyon Detayları (.cpp)**

*   **Yükleme Süreci (`Load`):**
    *   `Load(const char* c_szPathName)`: Belirtilen yoldaki `AreaData.txt` ve `AreaAmbienceData.txt` dosyalarını okur.
    *   `__Load_LoadObject()` ve `__Load_LoadAmbience()`: Bu dosyaları satır satır ayrıştırır, her nesne için bir `SObjectData` oluşturur ve `m_ObjectDataVector`'e ekler. Nesne özellikleri (`CProperty`) `CPropertyManager` aracılığıyla CRC kullanılarak alınır.
    *   `__Load_BuildObjectInstances()`: `m_ObjectDataVector`'deki her `SObjectData` için bir `SObjectInstance` oluşturur. `SObjectInstance`'ın türüne göre (`__SetObjectInstance_SetTree`, `__SetObjectInstance_SetBuilding` vb. çağrılarak) ilgili grafik/ses/fizik örneği oluşturulur ve ayarlanır.
        *   Binalar (`CGraphicThingInstance`) için LOD (Level of Detail) modelleri de yüklenir.
        *   Efektler (`CEffectInstance`) `CEffectManager` aracılığıyla oluşturulur.
        *   Ağaçlar (`SpeedTreeWrapper`) `CSpeedTreeForestDirectX8` aracılığıyla oluşturulur.
        *   Eğer bir nesne için attribute (`.attr`) dosyası bulunamazsa, binalar için otomatik olarak bir OBB (Oriented Bounding Box) çarpışma verisi oluşturulmaya çalışılır.

*   **Güncelleme (`Update`, `UpdateAroundAmbience`):**
    *   `__UpdateAniThingList()`: Alandaki animasyonlu `CGraphicThingInstance`'ların (`m_AniThingCloneInstanceVector`) `Update()` metodlarını ve statik olanların (`m_ThingCloneInstaceVector`) `UpdateLODLevel()` metodunu çağırır.
    *   `__UpdateEffectList()`: Alandaki tüm aktif efektleri (`m_EffectInstanceMap`) günceller. Oyuncunun pozisyonuna göre efektleri gizler/gösterir ve ömrü biten efektleri siler.
    *   `UpdateAroundAmbience(float fX, float fY, float fZ)`: Belirtilen merkez noktasına (genellikle oyuncu pozisyonu) göre alandaki tüm `SAmbienceInstance`'ların seslerini günceller (ses seviyesi, çalma durumu vb.). `SAmbienceInstance` içindeki `UpdateOnceSound`, `UpdateStepSound`, `UpdateLoopSound` metodları ile sesin çalma mantığını yönetir. `UpdateLoopSound`, mesafeye göre ses seviyesini ayarlar.

*   **Render Süreci (`Render`, `RenderEffect`, `RenderCollision`, `RenderAmbience`, `RenderDungeon`):**
    *   `Render()`: Animasyonlu nesnelerin (`m_AniThingCloneInstanceVector`) `Deform()` metodlarını çağırarak iskelet animasyonlarını günceller ve alandaki görünür ana nesneleri (`m_ThingCloneInstaceVector`) render eder.
    *   `CollectRenderingObject()` ve `CollectBlendRenderingObject()`: Diğer sistemler (örneğin `CMapOutdoor`) tarafından çağrılarak, bu alandaki opak ve saydam nesneleri ayrı listelerde toplar.
    *   `RenderEffect()`: Alandaki efektleri, render sıralarına göre (isteğe bağlı olarak sıralanarak) çizer.
    *   `RenderCollision()`: Hata ayıklama amacıyla, alandaki tüm nesnelerin çarpışma geometrilerini (AABB, OBB, sphere) çizer.
    *   `RenderAmbience()`: Hata ayıklama amacıyla, ambiyansların etki alanlarını (küreler, çizgiler) çizer. `SAmbienceInstance` içindeki `Render()` metodu da debug için ambiyansın etki alanını çizer.
    *   `RenderDungeon()`: Alandaki zindan bloklarını (`m_DungeonBlockCloneInstanceVector`) çizer. `WORLD_EDITOR` tanımlıysa, Shift tuşuna basıldığında zindan bloklarını yarı saydam çizebilir.

*   **Portal Yönetimi:**
    *   Alandaki nesneler portal ID'leri ile ilişkilendirilebilir.
    *   `EnablePortal(BOOL bFlag)`: Portal tabanlı görünürlüğü aktif/deaktif eder.
    *   `AddShowingPortalID(int iNum)`: Görünür olması istenen portal ID'lerini bir sete ekler.
    *   `RefreshPortal()`: Sadece `m_kSet_ShowingPortalID` içinde bulunan portallarla ilişkili nesnelerin görünür olmasını sağlar ve render listelerini (`m_TreeCloneInstaceVector`, `m_ThingCloneInstaceVector`, `m_DungeonBlockCloneInstanceVector`) buna göre günceller.

*   **Yenileme ve Temizleme:**
    *   `Refresh()`: `m_ObjectInstanceVector`'deki nesnelere dayanarak tüm "clone" (render/güncelleme için kullanılan) vektörlerini yeniden oluşturur. Nesnelerin bounding sphere'lerini ve çarpışma verilerini günceller.
    *   `Clear()`: Alanla ilişkili tüm örnekleri ve verileri temizler, bellek havuzlarına iade eder.

**Oyunda Kullanım Amacı ve Senaryoları**
`CArea`, oyun dünyasının modüler bir şekilde yüklenmesini ve yönetilmesini sağlayan temel bir sınıftır. Her bir alan, kendi nesnelerini, efektlerini ve seslerini barındırır ve oyuncunun çevresindeki alanlar dinamik olarak yüklenip kaldırılarak performans optimize edilir. Örneğin, bir oyuncu yeni bir bölgeye girdiğinde, o bölgeye ait `CArea` nesnesi yüklenir ve içindeki tüm ağaçlar, binalar, NPC'ler ve ortam sesleri aktif hale gelir. Oyuncu bölgeden uzaklaştığında ise bu `CArea` nesnesi bellekten kaldırılarak kaynak tasarrufu sağlanır.

### `AreaLoaderThread.h` ve `AreaLoaderThread.cpp`

**Oyundaki Kullanım Amacı:**
`TEMP_CAreaLoaderThread` sınıfı, oyun haritasının parçaları olan `CTerrain` (arazi verileri) ve `CArea` (alan içindeki nesneler, efektler vb.) nesnelerinin yükleme işlemlerini ana oyun iş parçacığından ayrı, bağımsız bir iş parçacığında (thread) eşzamansız olarak gerçekleştirmek üzere tasarlanmıştır. Amaç, özellikle büyük harita verileri yüklenirken oyunun ana döngüsünün kilitlenmesini veya yavaşlamasını engelleyerek daha akıcı bir kullanıcı deneyimi sunmaktır. Sınıf adındaki "TEMP_" ön eki, bunun geçici veya tam olarak geliştirilmemiş bir çözüm olabileceğine işaret etmektedir.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**

`TEMP_CAreaLoaderThread`, bir üretici-tüketici (producer-consumer) desenine benzer şekilde çalışır:

1.  **İstek (Request):** Ana iş parçacığı, yüklenmesini istediği `CTerrain` veya `CArea` nesnesinin bir işaretçisini `Request()` metodunu kullanarak yükleyici iş parçacığına iletir. Bu istekler, ilgili tür için ayrı bir "istek kuyruğuna" (`m_pTerrainRequestDeque` veya `m_pAreaRequestDeque`) eklenir. İstek kuyruklarına erişim, karşılık gelen mutex'ler (`m_TerrainRequestMutex`, `m_AreaRequestMutex`) ile senkronize edilir.
2.  **Sinyalleşme:** Yeni bir istek eklendiğinde, yükleyici iş parçacığını bekliyorsa uyandırmak için bir semafor (`m_hSemaphore`) kullanılır.
3.  **İşleme (Process):** Yükleyici iş parçacığı, `Execute()` metodu içindeki ana döngüsünde semaforu bekler. Sinyal aldığında, istek kuyruklarından birinden bir nesne alır ve ilgili yükleme işlemini (`ProcessTerrain()` veya `ProcessArea()`) gerçekleştirir.
    *   **`ProcessTerrain()`**: Belirtilen `CTerrain` nesnesi için arazi yükseklik haritası (`height.raw`), doku katmanları (`tile.raw`), attribute haritası (`attr.atr`), su haritası (`water.wtr`), gölge haritaları (`shadowmap.dds`, `shadowmap.raw`) ve minimap dokusu (`minimap.dds`) gibi dosyaları yükler. Yükleme işlemi tamamlandığında arazi yamalarını hesaplar (`CalculateTerrainPatch()`) ve araziyi hazır duruma getirir (`SetReady()`).
    *   **`ProcessArea()`**: Belirtilen `CArea` nesnesi için `Load()` metodunu çağırarak alan içindeki nesneleri, efektleri ve ambiyans verilerini (`AreaData.txt`, `AreaAmbienceData.txt`) yükler.
    *   **DİKKAT:** `Execute()` metodundaki `bProcessTerrain` değişkeni `true` olarak başlatılır ve mevcut kodda hiçbir zaman değiştirilmez. Bu, `ProcessArea()` fonksiyonunun pratikte **hiçbir zaman çağrılmayacağı** ve bu iş parçacığının yalnızca `CTerrain` nesnelerini yükleyeceği anlamına gelir. Alan yükleme işlevselliği ya eksik ya da kasıtlı olarak devre dışı bırakılmış olabilir.
4.  **Tamamlanma (Fetch):** Yükleme işlemi tamamlanan nesne, ilgili "tamamlanma kuyruğuna" (`m_pTerrainCompleteDeque` veya `m_pAreaCompleteDeque`) eklenir. Ana iş parçacığı, `Fetch()` metodunu kullanarak bu kuyruklardan yüklenmiş nesneleri alabilir. Tamamlanma kuyruklarına erişim de mutex'lerle (`m_TerrainCompleteMutex`, `m_AreaCompleteMutex`) korunur.
5.  **Kapatma (Shutdown):** `Shutdown()` metodu, iş parçacığını güvenli bir şekilde sonlandırmak için kullanılır.

*   **Senkronizasyon Mekanizmaları:**
    *   **Mutex'ler**: İstek ve tamamlanma kuyruklarına güvenli erişim sağlamak için kullanılır.
    *   **Semafor**: Yükleyici iş parçacığını, işlenecek yeni bir istek olduğunda uyandırmak için kullanılır.

*   **Yapay Gecikmeler:**
    *   `ProcessTerrain()` ve `ProcessArea()` fonksiyonlarında `Sleep(g_iLoadingDelayTime)` çağrıları bulunur. `g_iLoadingDelayTime` (bu dosyalarda tanımlanmamış harici bir global değişken), yükleme adımları arasına yapay gecikmeler ekler. Bu, genellikle hata ayıklama, I/O işlemlerini simüle etme veya yükleyici iş parçacığının CPU'yu aşırı tüketmesini önleme amacıyla kullanılır.

*   **Sınırlamalar ve Gözlemler:**
    *   "TEMP_" ön eki, sınıfın üretim için son hali olmayabileceğini düşündürür.
    *   `ProcessArea()` fonksiyonunun çağrılmıyor olması, sınıfın mevcut haliyle yalnızca arazi yükleme işlevselliği olarak çalıştığını gösterir.
    *   `Request` fonksiyonlarındaki `m_iRestSemCount` ile semaforu serbest bırakma şekli standart dışıdır ve gereksiz yere karmaşık görünebilir.

Bu sınıf, oyun dünyasının büyük bölümlerinin arka planda yüklenmesini sağlayarak potansiyel olarak performansı artırabilir, ancak mevcut implementasyonunda (özellikle alan yükleme kısmında) eksiklikler veya kasıtlı sınırlamalar barındırıyor gibi görünmektedir.

### `AreaTerrain.h` ve `AreaTerrain.cpp`

**Oyundaki Kullanım Amacı:**
`CTerrain` sınıfı, büyük bir dış mekan haritasının (`CMapOutdoor`) tek bir sektörünü veya "karo"sunu (tile) temsil eder. Her bir `CTerrain` nesnesi, kendi geometrisini (yükseklik haritası), yüzey dokularını (texture splatting), su yüzeylerini, attribute'larını (örneğin, yürünebilirlik), önceden hesaplanmış gölgelerini ve minimap için bir bölümünü yönetir. Bu sınıf, `AreaLoaderThread` tarafından arka planda yüklenir ve ana oyun döngüsünde render edilir.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**

`CTerrain` sınıfı, `CTerrainImpl` (PRTerrainLib kütüphanesinden) temel sınıfından miras alır ve arazi verilerinin yüklenmesi, işlenmesi ve render için hazırlanmasından sorumludur.

*   **Veri Yükleme ve Yönetimi:**
    *   **Yükseklik Haritası (`height.raw`):** `LoadHeightMap()` ile yüklenir. Arazinin temel 3D şeklini tanımlayan ham yükseklik değerlerini içerir. Yüklendikten sonra, her bir nokta için yüzey normalleri (`m_acNormalMap`) hesaplanır (`CalculateNormal()`).
    *   **Doku Karıştırma Haritası (Tile Map - `tile.raw`):** `RAW_LoadTileMap()` ile yüklenir. Arazinin her bir küçük biriminde hangi temel doku katmanının (örneğin, çimen, toprak, kaya) kullanılacağını belirtir.
    *   **Attribute Haritası (`attr.atr`):** `LoadAttrMap()` ile yüklenir. Arazinin her bir noktasının oyun mekaniği açısından özelliklerini (örneğin, `TA_BLOCK` - geçilemez, `TA_WATER` - su) içerir.
    *   **Su Haritası (`water.wtr`):** (Genellikle `CTerrainImpl` tarafından yönetilir) Su alanlarının konumunu ve su seviyelerini tanımlar.
    *   **Gölge Verileri:** `LoadShadowTexture()` ile gölge dokusunu (`shadowmap.dds`) ve `LoadShadowMap()` ile ham gölge yoğunluk verisini (`shadowmap.raw`) yükler.
    *   **Minimap Dokusu (`minimap.dds`):** `LoadMiniMapTexture()` ile bu arazi sektörüne ait minimap parçasını yükler.
    *   **Bellek Yönetimi:** `CTerrain` nesneleri, `CDynamicPool` (`ms_kPool`) kullanılarak verimli bir şekilde oluşturulur ve silinir.

*   **Doku Katmanlama (Texture Splatting):**
    *   `CTerrain` birden fazla doku katmanını (örneğin, çimen, toprak, kaya) yumuşak geçişlerle karıştırarak gerçekçi bir yüzey görünümü oluşturur.
    *   `RAW_CountTiles()`: Hangi doku katmanının (`tile.raw` içindeki değerler) ne sıklıkta kullanıldığını sayar.
    *   `RAW_GenerateSplat()`: Her bir doku katmanı (temel katman hariç) için bir alfa haritası (`m_lpAlphaTexture[]`) oluşturur. Bu alfa haritası, o katmanın nerelerde ve ne kadar opak görüneceğini belirler. Komşu tile'lara bakarak yumuşak geçişler (blending) sağlanır.
    *   `AddTexture32()`: Oluşturulan 8-bit alfa haritasını, mipmap'leri de olan bir Direct3D dokusuna (genellikle `A8R8G8B8` formatında, alfa kanalı kullanılarak) dönüştürür.

*   **Arazi Parçaları (Terrain Patches - `CTerrainPatch`):**
    *   Render performansını optimize etmek ve Seviye Detayı (LOD) uygulamak için her `CTerrain` nesnesi, `PATCH_XCOUNT` x `PATCH_YCOUNT` (örneğin, 16x16 veya 32x32) adet daha küçük parçaya (`CTerrainPatch`) bölünür.
    *   `CalculateTerrainPatch()` ve özellikle `_CalculateTerrainPatch()`: Her bir `CTerrainPatch` için vertex buffer'larını oluşturur.
        *   Bu işlem sırasında, yükseklik haritasından yükseklik değerleri ve önceden hesaplanmış normal haritasından normal vektörleri kullanılır.
        *   Parçanın genel eğimine göre türü (düz, tepelik, uçurum) belirlenir (`PATCH_TYPE_PLAIN`, `PATCH_TYPE_HILL`, `PATCH_TYPE_CLIFF`).
        *   Su haritası (`m_abyWaterMap`) ve su seviyeleri (`m_lWaterHeight`) kontrol edilir. Su varsa, su yüzeyi için ayrı vertex'ler oluşturulur. Suyun saydamlığı (alfa değeri), suyun derinliğine göre dinamik olarak hesaplanır; sığ su daha saydam, derin su daha opak olur.
        *   Oluşturulan arazi ve su vertex'leri, `CTerrainPatch` nesnesinin kendi Direct3D vertex buffer'larına yüklenir.

*   **Koordinat ve Durum:**
    *   `SetCoordinate(WORD wCoordX, WORD wCoordY)`: Bu arazi sektörünün büyük haritadaki (X, Y) koordinatlarını ayarlar.
    *   `SetReady(bool bReady)`: Arazinin yüklenip render edilmeye hazır olup olmadığını belirler.
    *   `m_pOwnerOutdoorMap`: Bu `CTerrain` nesnesinin ait olduğu ana `CMapOutdoor` nesnesine bir işaretçi tutar. Bu, komşu arazilerle etkileşim (örneğin, `WE_GetHeightMapValue` ile sınırların ötesinden yükseklik sorgulama) için önemlidir.

*   **Yardımcı Fonksiyonlar:**
    *   `GetHeight(int x, int y)`: Verilen dünya koordinatlarındaki arazi yüksekliğini üçgen enterpolasyonu ile hesaplar.
    *   `GetNormal(int ix, int iy, D3DXVECTOR3* pv3Normal)`: Verilen dünya koordinatlarındaki yüzey normalini döndürür.
    *   `isAttrOn()`, `GetAttr()`: Belirli bir koordinattaki attribute (özellik) bayraklarını sorgular.
    *   `GetWaterHeight()`: Belirli bir koordinattaki veya su numarasına göre suyun yüksekliğini döndürür.

*   **İşaretli Alanlar (`AllocateMarkedSplats`, `DeallocateMarkedSplats`):**
    *   Oyun içinde belirli alanları görsel olarak vurgulamak için (örneğin, görev hedefleri, yetenek etki alanları) arazi üzerine ek bir alfa tabanlı doku katmanı çizme yeteneği sunar.

`CTerrain` sınıfı, `GameLib` içindeki en temel ve karmaşık görsel bileşenlerden biridir. `PRTerrainLib`'den gelen temel arazi işleme yeteneklerini genişleterek, oyun motorunun ihtiyaç duyduğu spesifik yükleme, veri işleme ve render hazırlık adımlarını gerçekleştirir.

### `DragonSoulTable.h` ve `DragonSoulTable.cpp`

**Oyundaki Kullanım Amacı:**
`CDragonSoulTable` sınıfı, Ejderha Taşı (Dragon Soul) sistemi için gerekli olan tüm yapılandırma verilerini, özellikleri, üretilme ve geliştirilme (refine) parametrelerini `dragon_soul_table.txt` adlı bir dosyadan okumak, işlemek ve oyun içinde erişilebilir hale getirmekle sorumludur. Bu, Ejderha Taşlarının türlerini, temel ve rastgele gelen özelliklerini, bu özelliklerin gelme olasılıklarını, özelliklerin değer aralıklarını ve potansiyel olarak taşların seviye/kalite/saflık yükseltme mekaniklerini içerir.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**

`CDragonSoulTable`, `CGroupTextParseTreeLoader` kullanarak `dragon_soul_table.txt` dosyasını ayrıştırır. Bu dosya, Ejderha Taşı sistemiyle ilgili farklı veri setlerini içeren gruplardan oluşur.

*   **Temel Veri Yapıları ve Sabitler:**
    *   **`SApply` Yapısı:** Bir Ejderha Taşı özelliğini (efsununu) tanımlar:
        *   `apply_type`: Özelliğin türü (`CItemData::EApplyTypes` enum sabitlerinden biri).
        *   `apply_value`: Özelliğin sayısal değeri.
        *   `prob`: Özelliğin gelme olasılığı (genellikle ek özellikler için kullanılır).
    *   **`g_astGradeName[]`**: Ejderha Taşı kalitelerinin ("grade_normal", "grade_brilliant", vb.) metin karşılıklarını içerir.
    *   **`g_astStepName[]`**: Ejderha Taşı saflık seviyelerinin ("step_lowest", "step_low", vb.) metin karşılıklarını içerir.
    *   **`g_astMaterialName[]`**: Ejderha Taşı geliştirme malzemelerinin metin karşılıklarını içerir (kodda doğrudan kullanılmasa da `GetRefineStrengthValues` içinde endeksleme için referans alınır).

*   **Dosya Okuma ve Ayrıştırma (`ReadDragonSoulTableFile`):**
    *   Belirtilen `dragon_soul_table.txt` dosyasını yükler.
    *   Aşağıdaki ana bölümleri okumak için özel `Read*` fonksiyonlarını çağırır:
        *   **`ReadVnumMapper()` ("vnummapper" grubu):** Ejderha Taşı isimlerini (örneğin, "ONYX", "DIAMOND") benzersiz sayısal tür ID'lerine (`BYTE`) eşler. Bu eşlemeler `m_map_name_to_type` ve `m_map_type_to_name` içinde saklanır.
        *   **`ReadBasicApplys()` ("basicapplys" grubu):** Her Ejderha Taşı türü için garanti olarak gelen temel özellikleri (efsunları) ve bu özelliklerin değerlerini okur. Bu veriler `m_map_basic_applys_group` içinde saklanır.
        *   **`ReadAdditionalApplys()` ("additionalapplys" grubu):** Her Ejderha Taşı türü için rastgele gelebilecek ek özellikleri, bu özelliklerin değerlerini ve gelme olasılıklarını okur. Bu veriler `m_map_additional_applys_group` içinde saklanır.
    *   Ana gruplardan "applynumsettings" (özellik sayısı ayarları) ve "weighttables" (özellik değeri gelme ağırlıkları) için `CGroupNode` işaretçilerini önbelleğe alır.
    *   Yüklenen verilerin temel tutarlılığını kontrol etmek için `Check*` fonksiyonlarını çağırır.

*   **Veri Erişim Metodları (`Get*`):**
    *   Sınıf, yüklenen verilere erişim sağlamak için bir dizi `Get*` metodu sunar. Bu metodlar, genellikle Ejderha Taşı türü, kalite indeksi, saflık indeksi, güçlendirme seviyesi gibi parametreler alır ve ilgili değeri döndürür.
    *   **`GetBasicApplys()` / `GetAdditionalApplys()`**: Belirli bir Ejderha Taşı türü için temel veya ek özellik listesini döndürür.
    *   **`GetApplyNumSettings()`**: Belirli bir tür ve kalitedeki Ejderha Taşı için kaç adet temel özellik, ve en az/en çok kaç adet ek özellik geleceğini döndürür.
    *   **`GetWeight()`**: Belirli bir tür, kalite, saflık ve güçlendirme seviyesi için bir özelliğin belirli bir değere sahip olma ağırlığını (olasılığını) döndürür. Eğer Ejderha Taşı türüne özel bir ayar yoksa, "default" grubundaki ayarları kullanır.
    *   **Yorum Satırındaki `GetRefine*` ve `GetDragon*Ext*` Metodları:** Bu metodlar (`GetRefineGradeValues`, `GetRefineStepValues`, `GetRefineStrengthValues`, `GetDragonHeartExtValues`, `GetDragonSoulExtValues`), muhtemelen Ejderha Taşı kalite yükseltme, saflık yükseltme, güçlendirme (farklı malzemelerle), Ejderha Kalbi çıkarma (Dragon Heart Extraction) ve Ejderha Ruhu çıkarma (Dragon Soul Extraction) gibi sistemlerin parametrelerini (gerekli öğe sayısı, ücret, başarı/başarısızlık olasılıkları, yan ürünler vb.) okumak için tasarlanmıştır. Bu metodların ve ilişkili `Check*` metodlarının `ReadDragonSoulTableFile` içindeki çağrılarının yorum satırında olması, bu özelliklerin şu anda aktif olmayabileceğini veya geliştirme aşamasında/kaldırılmış olabileceğini gösterir.

*   **Veri Doğrulama (`Check*` Metodları):**
    *   `CheckApplyNumSettings`, `CheckWeightTables` (ve yorum satırındaki diğerleri) gibi metodlar, `dragon_soul_table.txt` dosyasından okunan verilerin temel mantıksal tutarlılığını kontrol eder. Örneğin, gerekli değerlerin dosyada olup olmadığını, olasılıkların negatif olmamasını, sayaçların geçerli aralıklarda olmasını vb. doğrularlar. Hatalı bir durum tespit edilirse `TraceError` ile log kaydı oluşturulur.

*   **Yapılandırma Dosyası (`dragon_soul_table.txt`):**
    *   Ejderha Taşı sisteminin tüm davranışını tanımlayan, `CGroupTextParseTreeLoader` formatında, hiyerarşik gruplardan oluşan bir metin dosyasıdır.
    *   Her grup, belirli bir özelliği veya sistemi yapılandırır (örneğin, "vnummapper", "basicapplys", "additionalapplys", "applynumsettings", "weighttables").

`CDragonSoulTable`, Ejderha Taşı sistemi için hayati öneme sahip tüm parametreleri merkezi bir yerden yönetir. Bu sayede, oyun dengesi ve Ejderha Taşı özellikleri, oyunun kaynak kodunu değiştirmeden, sadece `dragon_soul_table.txt` dosyasını düzenleyerek ayarlanabilir.

### `CDungeonBlock` Sınıfı

**Dosyalar:** `GameLib/DungeonBlock.h`, `GameLib/DungeonBlock.cpp`

`CDungeonBlock`, `CGraphicObjectInstance` sınıfından türetilmiş olup, oyun dünyasındaki zindan bölümlerini veya benzer statik yapı bloklarını temsil eder. Bu sınıf, bir veya daha fazla 3D modelden oluşan yapıları yüklemek, güncellemek, çizmek ve bu yapılarla etkileşim (çarpışma, yükseklik sorgulama) kurmak için kullanılır.

#### Temel Özellikler ve Sorumluluklar

*   **Model Yönetimi:** `.gr2` gibi model dosyalarından (`CGraphicThing` aracılığıyla) zindan yapısını yükler. Her bir alt model için `CDungeonModelInstance` (bir iç sınıf) örnekleri oluşturur ve bunları bir konteynerde (`m_ModelInstanceContainer`) saklar.
*   **Çizim (Rendering):**
    *   Normal çizim: Zindan bloklarını, genellikle iki doku kullanan özel bir vertex shader (`ms_pnt2VS`) ile çizer.
    *   Gölge çizimi: Zindan bloklarının basitleştirilmiş bir gölgesini, doku yerine sabit bir renk ve özel alfa karıştırma ayarları kullanarak çizer.
*   **Güncelleme:** Her çerçevede zindan bloğunun dönüşümünü ve içindeki model örneklerinin animasyon durumlarını günceller.
*   **Sınırlayıcı Hacimler (Bounding Volumes):**
    *   Kaba görünürlük testleri (culling) ve genel çarpışma tespiti için tüm bloğu çevreleyen bir sınırlayıcı küre (`m_v3Center`, `m_fRadius`) hesaplar.
    *   Daha hassas sınırlayıcı kutu bilgisi de sağlayabilir.
*   **Çarpışma Tespiti:**
    *   Işın kesişim testlerini (`Intersect`) destekler.
    *   Statik çarpışma verilerini (`CStaticCollisionDataVector`) alıp kendi dönüşümüne göre işleyebilir.
*   **Yükseklik Verisi:** Araziye benzer şekilde, belirli bir dünya (x,y) koordinatındaki zemin yüksekliğini sorgulamak için bir yükseklik öznitelik örneği (`CAttributeInstance`) tutabilir ve kullanabilir.

#### Önemli Üye Değişkenler

*   `m_v3Center`: `D3DXVECTOR3`, Bloğun sınırlayıcı küresinin merkez noktası.
*   `m_fRadius`: `float`, Bloğun sınırlayıcı küresinin yarıçapı.
*   `m_pThing`: `CGraphicThing*`, Bloğun ana model verilerini tutan işaretçi.
*   `m_ModelInstanceContainer`: `TModelInstanceContainer` (yani `std::vector<CDungeonModelInstance*>`), Bloğu oluşturan alt model örneklerini tutan vektör.
*   `m_kDeformableVertexBuffer`: `CGraphicVertexBuffer`, Deforme olabilir tepe noktası verileri için kullanılan tampon.

#### Önemli Metotlar

*   **`CDungeonBlock()` / `~CDungeonBlock()` / `Destroy()`:** Yapıcı, yıkıcı ve kaynak temizleme fonksiyonları.
*   **`bool Load(const char* c_szFileName)`:** Verilen dosya yolundan zindan bloğu modelini ve ilişkili kaynakları yükler. Başarılı olursa `true` döner.
*   **`void Update()`:** Bloğun ve içindeki model örneklerinin durumunu günceller.
*   **`void Render()`:** Zindan bloğunu normal şekilde ekrana çizer.
*   **`void OnRenderShadow()`:** Zindan bloğunun gölgesini çizer.
*   **`void BuildBoundingSphere()`:** Tüm alt modelleri dikkate alarak bloğun genel sınırlayıcı küresini hesaplar.
*   **`bool GetBoundingSphere(D3DXVECTOR3& v3Center, float& fRadius)`:** Bloğun dünya koordinatlarındaki sınırlayıcı küre bilgilerini döndürür.
*   **`void GetBoundBox(D3DXVECTOR3* pv3Min, D3DXVECTOR3* pv3Max)`:** Bloğun genel sınırlayıcı kutusunu hesaplar ve döndürür.
*   **`bool Intersect(float* pfu, float* pfv, float* pft)`:** Verilen bir ışınla (genellikle fare tıklamasıyla) bloğun kesişip kesişmediğini test eder. Kesişirse `true` döner ve kesişim bilgilerini `pfu`, `pfv`, `pft` ile doldurur.
*   **`void OnUpdateCollisionData(const CStaticCollisionDataVector* pscdVector)`:** Statik çarpışma verilerini bloğa ekler.
*   **`void OnUpdateHeighInstance(CAttributeInstance* pAttributeInstance)`:** Yükseklik sorgulamaları için kullanılacak yükseklik haritası örneğini ayarlar.
*   **`bool OnGetObjectHeight(float fX, float fY, float* pfHeight)`:** Verilen (x,y) dünya koordinatındaki zemin yüksekliğini `pfHeight` ile döndürür. Yükseklik verisi varsa `true` döner.

#### İç Sınıf: `CDungeonModelInstance`

`CGrannyModelInstance` sınıfından türeyen bu iç sınıf, zindan bloklarını oluşturan bireysel modellerin çizimini yönetir.

*   **`void RenderDungeonBlock()`:** Modeli, konum-normal-doku koordinatlarını (`TPNT2Vertex`) içeren ve iki doku kullanan (`CGrannyMaterial::TYPE_BLEND_PNT`) bir shader ile çizer.
*   **`void RenderDungeonBlockShadow()`:** Modelin gölgesini çizer. Bu çizim sırasında dokular devre dışı bırakılır, `D3DRS_TEXTUREFACTOR` ile ayarlanan bir renk kullanılır ve alfa karıştırma modları gölge efekti oluşturacak şekilde (kaynak renk sıfır, hedef renk kaynak renkle çarpılır) ayarlanır.

#### Oyundaki Kullanım Amacı

`CDungeonBlock`, genellikle zindanlar, mağaralar, binaların içleri gibi karmaşık ve statik geometrilerden oluşan oyun alanlarını oluşturmak için kullanılır. Oyuncuların ve diğer karakterlerin etkileşime girebileceği (üzerinde yürüyebileceği, çarpışabileceği) katı yapılar olarak görev yaparlar. `Load` fonksiyonu aracılığıyla farklı zindan parçaları yüklenebilir ve bir araya getirilerek büyük ve karmaşık ortamlar oluşturulabilir. Gölge çizim yeteneği, bu yapıların oyun dünyasında daha gerçekçi görünmesine katkıda bulunur.

### `IFlyEventHandler` Arayüzü

**Dosya:** `GameLib/FlyHandler.h`

`IFlyEventHandler`, uçan nesnelerin (mermiler, büyüler vb.) yaşam döngüleri boyunca meydana gelen çeşitli olaylara tepki vermek için kullanılan bir arayüz (interface) sınıfıdır. Bu arayüzü implemente eden sınıflar, uçan nesneler tarafından tetiklenecek olan olay işleyici (event handler) metotlarını tanımlayabilirler.

#### Temel Amaç

*   Uçan nesne davranışlarının sonucunda ortaya çıkan durumları (hedef bulma, hedefe çarpma, menzil dışına çıkma vb.) oyun mantığına bildirmek.
*   `CActorInstance` (genellikle fırlatan) ve `CFlyingInstance` (uçan nesnenin kendisi) arasında bir iletişim katmanı sağlamak.

#### Arayüz Metotları

Metotlar, olayı tetikleyen ana kaynağa göre gruplandırılabilir:

1.  **`CActorInstance` Tarafından Çağrılan Olaylar:**
    *   `virtual void OnSetFlyTarget()`: Uçan bir nesne için hedef başarıyla ayarlandığında çağrılır.
    *   `virtual void OnShoot(DWORD dwSkillIndex)`: Bir uçan nesne (genellikle bir yetenekle ilişkili, `dwSkillIndex` ile belirtilir) fırlatıldığında çağrılır.
    *   `virtual void OnNoTarget()`: Fırlatma sırasında geçerli bir hedef bulunamadığında çağrılır.
    *   `virtual void OnNoArrow()`: (Muhtemelen) Fırlatılacak mermi (ok vb.) kalmadığında çağrılır.

2.  **`CFlyingInstance` Tarafından Çağrılan Olaylar (Patlama/Çarpışma Durumları):**
    *   `virtual void OnExplodingOutOfRange()`: Uçan nesne, hedefine ulaşamadan maksimum menzilini aşıp patladığında/yok olduğunda çağrılır.
    *   `virtual void OnExplodingAtBackground()`: Uçan nesne, oyun dünyasındaki bir arka plan nesnesine (duvar, zemin vb.) çarparak patladığında/yok olduğunda çağrılır.
    *   `virtual void OnExplodingAtAnotherTarget(DWORD dwSkillIndex, DWORD dwVID)`: Uçan nesne, asıl hedefi yerine başka bir geçerli hedefe (canavar, oyuncu vb., `dwVID` ile tanımlanır) çarparak patladığında/yok olduğunda çağrılır. `dwSkillIndex`, ilgili yeteneği belirtir.
    *   `virtual void OnExplodingAtTarget(DWORD dwSkillIndex)`: Uçan nesne, asıl hedefine başarıyla ulaşıp patladığında/etki ettiğinde çağrılır. `dwSkillIndex`, ilgili yeteneği belirtir.

Tüm metotlar sanal (virtual) olduğundan ve varsayılan olarak hiçbir işlem yapmadığından, türetilmiş sınıflar sadece ilgilendikleri olayları işlemek üzere bu metotları geçersiz kılabilir (override).

### `CFlyingData` Sınıfı

**Dosyalar:** `GameLib/FlyingData.h`, `GameLib/FlyingData.cpp`

`CFlyingData`, oyun içinde hareket eden uçan nesnelerin (örneğin oklar, sihirli mermiler, yetenek efektleri) davranışlarını ve özelliklerini tanımlayan verileri içeren bir sınıftır. Bu veriler genellikle harici bir script dosyasından (örneğin `.mfl` uzantılı) yüklenir ve `CFlyingInstance` nesneleri tarafından kullanılır.

#### Temel Özellikler ve Sorumluluklar

*   **Fiziksel ve Davranışsal Parametreler:** Uçan nesnenin başlangıç hızı, ivmesi, yerçekimi etkisi, menzili, açısal hızı, hedef takip (homing) yeteneği, yayılma (spreading) özelliği, çarpışma davranışları (arka plana çarpma, başka hedeflere çarpma, delip geçme sayısı) gibi temel dinamiklerini tanımlar.
*   **Görsel Efektler:** Uçan nesneye bağlı "kuyruk" (tail) efektleri veya yörünge boyunca takip eden diğer görsel eklentileri (`TFlyingAttachData` yapısı ile) ve hedefe ulaştığında veya yok olduğunda oluşacak patlama efektini (`m_strBombEffectName`, `m_dwBombEffectID`) yönetir.
*   **Script Yükleme/Kaydetme:** `LoadScriptFile` ve `SaveScriptFile` metotları aracılığıyla uçan nesne tanımlarını metin tabanlı dosyalardan okuyabilir ve bu dosyalara kaydedebilir. Bu, oyun tasarımcılarının kod değişikliği yapmadan uçan nesne davranışlarını ayarlamasına olanak tanır.
*   **Bellek Yönetimi:** `CDynamicPool` kullanarak `CFlyingData` nesnelerinin verimli bir şekilde oluşturulup yok edilmesini sağlar.

#### Önemli Üye Değişkenler

*   `m_strFilename`: `std::string`, Yüklenen script dosyasının adı.
*   `m_bSpreading`: `bool`, Uçan nesnenin yayılarak hareket edip etmeyeceği.
*   `m_bMaintainParallel`: `bool`, Paralel hareketin korunup korunmayacağı.
*   `m_fInitVel`: `float`, Başlangıç hızı.
*   `m_fConeAngle`: `float`, Uçuşun konik yayılma açısı (derece cinsinden olabilir).
*   `m_fRollAngle`: `float`, Uçan nesnenin kendi ekseni etrafında yuvarlanma açısı.
*   `m_v3AngVel`: `D3DXVECTOR3`, Açısal hız (x, y, z eksenlerinde).
*   `m_fGravity`: `float`, Yerçekimi etkisi (genellikle negatif Z yönünde bir ivme).
*   `m_fBombRange`: `float`, Patlama efekti için etki yarıçapı.
*   `m_strBombEffectName`: `std::string`, Patlama anında oynatılacak efekt dosyasının adı.
*   `m_dwBombEffectID`: `DWORD`, `CEffectManager` tarafından atanan patlama efekti ID'si.
*   `m_bIsHoming`: `bool`, Hedef takip özelliğinin aktif olup olmadığı.
*   `m_fHomingMaxAngle`: `float`, Hedefe kilitlenme için izin verilen maksimum dönüş açısı (saniyede derece cinsinden olabilir).
*   `m_fHomingStartTime`: `float`, Hedef takibinin başlayacağı süre (saniye cinsinden, fırlatıldıktan sonra).
*   `m_bHitOnBackground`: `bool`, Arka plan nesnelerine çarpıp çarpmayacağı.
*   `m_bHitOnAnotherMonster`: `bool`, Diğer canavarlara/hedeflere çarpıp çarpmayacağı.
*   `m_iPierceCount`: `int`, Kaç hedefi delip geçebileceği (0 ise ilk çarpışmada durur).
*   `m_fCollisionSphereRadius`: `float`, Çarpışma tespiti için kullanılan kürenin yarıçapı (kodda yorum satırında, kullanımı aktif olmayabilir).
*   `m_fRange`: `float`, Maksimum uçuş menzili.
*   `m_v3Accel`: `D3DXVECTOR3`, Uçan nesneye etki eden sabit ivme.
*   `m_AttachDataVector`: `std::vector<TFlyingAttachData>`, Uçan nesneye bağlı ek görsel efektlerin listesi.

#### `TFlyingAttachData` Yapısı

Uçan nesneye eklenen görsel eklentilerin (efektler, kuyruklar) özelliklerini tanımlar.

*   `iType`: `int`, Eklenti türü (`FLY_ATTACH_EFFECT` veya implemente edilmemiş `FLY_ATTACH_OBJECT`).
*   `iFlyType`: `int`, Eklentinin uçuş paterni (örn: `FLY_ATTACH_TYPE_LINE`, `FLY_ATTACH_TYPE_SINE`).
*   `strFilename`: `std::string`, Eklenti için efekt (`.mse`) dosyasının adı.
*   `bHasTail`: `bool`, Eklentinin bir kuyruğu olup olmadığı.
*   `dwTailColor`: `DWORD`, Kuyruk rengi (ARGB formatında).
*   `fTailLength`: `float`, Kuyruk uzunluğu.
*   `fTailSize`: `float`, Kuyruk kalınlığı/boyutu.
*   `bRectShape`: `bool`, Kuyruğun şeklinin dikdörtgen olup olmadığı (değilse genellikle çizgi şeklinde).
*   `fRoll`: `float`, Eklentinin kendi ekseni etrafında dönüşü.
*   `fDistance`: `float`, `FLY_ATTACH_TYPE_MULTI_LINE` için çizgiler arası mesafe.
*   `fPeriod`: `float`, `FLY_ATTACH_TYPE_SINE` veya `FLY_ATTACH_TYPE_EXP` için dalga periyodu.
*   `fAmplitude`: `float`, `FLY_ATTACH_TYPE_SINE` veya `FLY_ATTACH_TYPE_EXP` için dalga genliği.

#### Önemli Metotlar

*   **`CFlyingData()` / `~CFlyingData()` / `Destroy()` / `__Initialize()`:** Yapıcı, yıkıcı, kaynak temizleme ve başlangıç değerlerini ayarlama.
*   **`bool LoadScriptFile(const char* c_szFilename)`:** Verilen script dosyasından uçan nesne verilerini yükler. `CTextFileLoader` kullanarak dosyayı ayrıştırır.
*   **`bool SaveScriptFile(const char* c_szFilename)`:** Mevcut uçan nesne verilerini belirtilen dosyaya kaydeder.
*   **`void SetBombEffect(const char* szEffectName)`:** Patlama efektini ayarlar, `CEffectManager`'a kaydeder ve ID'sini saklar.
*   **`DWORD AttachFlyEffect(int iType, const std::string& strFilename, float fRoll, float fArg1, float fArg2)`:** Uçan nesneye yeni bir görsel eklenti (efekt) ekler ve eklenti indeksini döndürür.
*   **`TFlyingAttachData& GetAttachDataReference(int iIndex)`:** Belirli bir indeksteki eklenti verisine referans döndürür.
*   **`int GetAttachDataCount()`:** Eklenmiş eklenti sayısını döndürür.
*   **`void DuplicateAttach(int iIndex)`:** Belirli bir indeksteki eklentiyi kopyalar.
*   **`void RemoveAttach(int iIndex)`:** Belirli bir indeksteki eklentiyi kaldırır.
*   **`void RemoveAllAttach()`:** Tüm eklentileri kaldırır.
*   **`static CFlyingData* New()` / `static void Delete(CFlyingData* pData)` / `static void DestroySystem()`:** `CDynamicPool` aracılığıyla bellek yönetimi için statik metotlar.

#### Oyundaki Kullanım Amacı

`CFlyingData`, oyundaki tüm fırlatılabilir nesnelerin (oklar, büyüler, özel yetenek mermileri vb.) nasıl davranacağını tanımlayan bir şablon görevi görür. Bir yetenek kullanıldığında veya bir ok fırlatıldığında, ilgili `CFlyingData` yüklenir (veya önbellekten alınır) ve bu verilere göre bir `CFlyingInstance` (uçan nesnenin oyun dünyasındaki canlı örneği) oluşturulur. Bu sayede, farklı yetenekler veya silahlar için çok çeşitli ve özelleştirilmiş mermi davranışları tasarlanabilir.

### `CFlyingInstance` Sınıfı

**Dosyalar:** `GameLib/FlyingInstance.h`, `GameLib/FlyingInstance.cpp`

`CFlyingInstance`, oyun dünyasında aktif olarak hareket eden, görsel efektlere sahip olabilen ve hedeflere çarpabilen bir uçan nesne örneğini temsil eder. Genellikle oklar, büyüler veya diğer yetenek mermileri gibi fırlatılabilir varlıklar için kullanılır. Her bir `CFlyingInstance`, davranışını ve görünümünü tanımlayan bir `CFlyingData` nesnesine referans tutar ve olayları işlemek için bir `IFlyEventHandler` kullanabilir.

#### Temel Özellikler ve Sorumluluklar

*   **Yaşam Döngüsü Yönetimi:** Bir `CFlyingInstance`, `Create()` metoduyla oluşturulur, `Update()` metoduyla her çerçevede güncellenir ve menzili bittiğinde, bir hedefe çarptığında veya başka bir nedenle yok olması gerektiğinde `Destroy()` (veya dolaylı olarak `__Explode()`) ile sonlandırılır.
*   **Fizik ve Hareket:** Kendi pozisyonunu, hızını, ivmesini ve yönelimini (rotasyon) yönetir. `CFlyingData`'dan alınan başlangıç hızı, ivme, yerçekimi gibi parametrelerle hareket eder.
*   **Hedefleme ve Homing:** Belirli bir `CFlyTarget`'a (bir nesne veya pozisyon olabilir) doğru hareket eder. Eğer `CFlyingData` içinde tanımlanmışsa, hedefini aktif olarak takip etme (homing) yeteneğine sahiptir (`AdjustDirectionForHoming`).
*   **Çarpışma Tespiti:**
    *   Ana hedefine ulaşıp ulaşmadığını kontrol eder.
    *   `CFlyingData`'da belirtilmişse, araziye, statik dünya nesnelerine (`m_pData->m_bHitOnBackground`) veya diğer karakterlere (`m_pData->m_bHitOnAnotherMonster`) çarpıp çarpmadığını kontrol eder.
    *   Delip geçme (`m_iPierceCount`) özelliğini destekler, birden fazla hedefi vurabilir.
*   **Görsel Efektler:**
    *   `CFlyingData`'da tanımlanan eklenti efektlerini (`TAttachEffectInstance` aracılığıyla) yönetir. Bu eklentiler, uçan nesneye bağlı kalan izler (`CFlyTrace`) veya diğer görsel efektler olabilir. `UpdateAttachInstance()` ile bu efektlerin pozisyon ve rotasyonları güncellenir, `RenderAttachInstance()` ile (genellikle izler) çizilir.
    *   Hedefe çarptığında veya yok olduğunda bir patlama efekti (`__Bomb()`) oluşturabilir.
*   **Olay Yönetimi:** `IFlyEventHandler` arayüzü üzerinden çeşitli olayları (hedefe çarpma, menzil dışına çıkma vb.) fırlatan karaktere veya ilgili sisteme bildirir.
*   **Bellek Yönetimi:** `CDynamicPool` kullanarak verimli bir şekilde bellekten alınır ve geri bırakılır.

#### Önemli Üye Değişkenler

*   `m_pData`: `CFlyingData*`, Bu örneğin davranışını tanımlayan veri nesnesi.
*   `m_bAlive`: `bool`, Örnegin aktif olup olmadığı.
*   `m_FlyTarget`: `CFlyTarget`, Mevcut hedef.
*   `m_pOwner`: `IActorInstance*`, Bu uçan nesneyi fırlatan karakter.
*   `m_pHandler`: `IFlyEventHandler*`, Olayları işleyecek handler.
*   `m_dwSkillIndex`: `DWORD`, İlişkili yetenek ID'si.
*   `m_iPierceCount`: `int`, Kalan delip geçme hakkı.
*   `m_v3Position`: `D3DXVECTOR3`, Dünya koordinatlarındaki mevcut pozisyon.
*   `m_v3Velocity`: `D3DXVECTOR3`, Dünya koordinatlarındaki mevcut hız.
*   `m_v3Accel`: `D3DXVECTOR3`, Uygulanan ivme.
*   `m_qRot`: `D3DXQUATERNION`, Ana yönelim.
*   `m_fRemainRange`: `float`, Kalan uçuş mesafesi.
*   `m_vecAttachEffectInstance`: `TAttachEffectInstanceVector`, Bağlı efekt örneklerinin listesi.
*   `m_HittedObjectSet`: `std::set<IActorInstance*>`, Daha önce vurulmuş karakterlerin kaydı (delip geçme için).

#### Önemli Metotlar

*   **`void Create(CFlyingData* pData, const D3DXVECTOR3& c_rv3StartPos, const CFlyTarget& c_rkTarget, bool canAttack)`:** Uçan nesne örneğini başlatır. Temel parametreleri ayarlar, `CFlyingData`'yı kullanarak ilk yönü ve hızı belirler, eklenti efektlerini oluşturur.
*   **`bool Update()`:** Her çerçevede çağrılır. Hareketi günceller, hedef takibini yapar, çarpışmaları kontrol eder (hedef, diğer karakterler, arka plan), menzili kontrol eder ve gerekirse olayları tetikler veya nesneyi yok eder. Eklenti efektlerini de günceller.
*   **`void Render()`:** Genellikle `RenderAttachInstance()` çağırarak bağlı olan `CFlyTrace` gibi iz efektlerini çizer.
*   **`void AdjustDirectionForHoming(const D3DXVECTOR3& v3TargetPosition)`:** Hedef takip mekaniğini uygular; nesnenin yönünü hedefe doğru, belirli bir açısal hız limitiyle çevirir.
*   **`void BuildAttachInstance()` / `void UpdateAttachInstance()` / `void ClearAttachInstance()`:** Uçan nesneye bağlı görsel efektlerin oluşturulmasını, güncellenmesini ve silinmesini yönetir.
*   **`void __Explode(bool bBomb = true)`:** Uçan nesneyi "ölü" olarak işaretler ve `bBomb` true ise patlama efekti oluşturur (`__Bomb()`).
*   **`void __Bomb()`:** `CFlyingData`'da tanımlı patlama efektini `CEffectManager` aracılığıyla mevcut pozisyonda oluşturur.
*   **`void SetEventHandler(IFlyEventHandler* pHandler)`:** Olay işleyiciyi ayarlar.
*   **`void SetOwner(IActorInstance* pOwner)`:** Uçan nesneyi fırlatanı ayarlar.
*   **`void SetSkillIndex(DWORD dwIndex)`:** İlişkili yetenek ID'sini ayarlar.

#### `TAttachEffectInstance` İç Yapısı

*   `iAttachIndex`: `CFlyingData` içindeki `TFlyingAttachData`'nın indeksi.
*   `dwEffectInstanceIndex`: `CEffectManager` tarafından yönetilen efekt örneğinin ID'si.
*   `pFlyTrace`: `CFlyTrace*`, Eğer bu eklentinin bir kuyruk izi varsa, onu çizecek olan `CFlyTrace` nesnesine işaretçi.

#### Oyundaki Kullanım Amacı

`CFlyingInstance`, bir karakter bir ok fırlattığında, bir büyü yaptığında veya menzilli bir yetenek kullandığında oyun dünyasında yaratılan asıl "mermi"dir. `CFlyingData`'dan aldığı direktiflerle hareket eder, hedefleriyle etkileşime girer, görsel efektlerini sergiler ve yaşam döngüsünü tamamlar. `CFlyingObjectManager` tarafından yönetilirler.

### `CFlyingManager` Sınıfı

**Dosyalar:** `GameLib/FlyingObjectManager.h`, `GameLib/FlyingObjectManager.cpp`

`CFlyingManager`, `CSingleton` tasarım desenini kullanan bir yönetici sınıfıdır. Oyundaki tüm uçan nesne tanımlarını (`CFlyingData`) ve bu tanımlardan oluşturulan aktif uçan nesne örneklerini (`CFlyingInstance`) merkezi olarak yönetmekten sorumludur.

#### Temel Özellikler ve Sorumluluklar

*   **Uçan Nesne Tanımı Yönetimi (`CFlyingData`):**
    *   `RegisterFlyingData()`: Uçan nesnelerin davranışlarını ve özelliklerini tanımlayan script dosyalarını (genellikle `.mfl`) yükler. Her bir tanım, dosya adının CRC32 değeri ile anahtarlanarak bir harita (`m_kMap_pkFlyData`) içinde saklanır.
*   **Uçan Nesne Örneği Yönetimi (`CFlyingInstance`):**
    *   `CreateFlyingInstanceFlyTarget()`: Belirli bir `CFlyingData` tanımına (CRC ile referanslanır), başlangıç pozisyonuna, hedefe (`CFlyTarget`) ve saldırı yapabilme durumuna göre yeni bir `CFlyingInstance` oluşturur. Oluşturulan örnekler bir liste (`m_kLst_pkFlyInst`) içinde tutulur.
    *   `Update()`: Aktif tüm `CFlyingInstance` örneklerinin her çerçevede güncellenmesini sağlar. Yaşam döngüsü sona eren (örneğin hedefe ulaşan veya menzili biten) örnekleri listeden çıkarır ve bellekten siler.
    *   `Render()`: Aktif tüm `CFlyingInstance` örneklerinin (genellikle eklenti izleri gibi) çizilmesini sağlar.
*   **Sunucu Kontrollü Uçan Nesneler:**
    *   `RegisterIndexedFlyData()`: Sunucudan gelen bir indeks ve uçuş türü (`EIndexFlyType`) ile bir uçan nesne tanımını (script dosyası) eşleştirir. Bu eşlemeler `m_kMap_dwIndexFlyData` içinde saklanır.
    *   `CreateIndexedFly()`: Sunucudan gelen bir indekse göre, belirli bir başlangıç ve bitiş aktörü arasında, tanımlanan uçuş türüne (`NORMAL`, `FIRE_CRACKER`, `AUTO_FIRE`) uygun şekilde bir `CFlyingInstance` oluşturur. Bu tür uçan nesneler genellikle saldırı amaçlı değildir (`canAttack = false`).
*   **Ok Kılıfı Sistemi Entegrasyonu (`ENABLE_QUIVER_SYSTEM`):**
    *   `CreateIndexedFlyingInstanceFlyTarget()`: Muhtemelen okçu karakterlerin ok kılıfından seçtiği belirli bir ok türü için, verilen indekse göre bir `CFlyingInstance` oluşturur. Bu durumda uçan nesne saldırı yapabilir (`canAttack = true`).
*   **Harita Yöneticisi Entegrasyonu:**
    *   Bir `CMapManager` işaretçisi (`m_pMapManager`) tutar. Bu, `CFlyingInstance`'ların arka plan (zemin) çarpışmalarını kontrol ederken arazi yüksekliğini sorgulayabilmesi için gereklidir.
*   **Kaynak Yönetimi:** Kullanılmayan `CFlyingData` ve `CFlyingInstance` nesnelerinin bellekten düzgün bir şekilde silinmesini sağlar (`Destroy`, `DeleteAllInstances`).

#### Önemli Üye Değişkenler

*   `m_kMap_pkFlyData`: `TFlyingDataMap` ( `std::map<DWORD, CFlyingData*>` ), Yüklenmiş `CFlyingData` tanımlarını CRC anahtarıyla saklar.
*   `m_kLst_pkFlyInst`: `TFlyingInstanceList` ( `std::list<CFlyingInstance*>` ), Aktif `CFlyingInstance` örneklerini tutar.
*   `m_kMap_dwIndexFlyData`: `TIndexFlyDataMap` ( `std::map<DWORD, TIndexFlyData>` ), Sunucu tarafından indekslenmiş uçan nesne tanımlarını saklar. `TIndexFlyData` yapısı, uçuş türünü (`EIndexFlyType`) ve ilgili `CFlyingData`'nın CRC'sini içerir.
*   `m_pMapManager`: `CMapManager*`, Harita yöneticisine bir işaretçi.
*   `m_IDCounter`: `DWORD`, Yeni oluşturulan `CFlyingInstance`'lara benzersiz bir ID atamak için kullanılan sayaç (bu ID'nin kapsamlı bir kullanımı kodda belirgin değildir).

#### `EIndexFlyType` Enum

Sunucu tarafından kontrol edilen `CreateIndexedFly` ile oluşturulan uçan nesnelerin temel davranışını belirler:

*   `INDEX_FLY_TYPE_NORMAL`: Başlangıç aktöründen bitiş aktörüne doğru standart bir uçuş.
*   `INDEX_FLY_TYPE_FIRE_CRACKER`: Başlangıç aktöründen rastgele bir yöne ve mesafeye doğru, genellikle görsel bir efekt (havai fişek gibi) amaçlı bir uçuş.
*   `INDEX_FLY_TYPE_AUTO_FIRE`: Genellikle başlangıç aktöründen bitiş aktörüne doğru daha direkt bir yörüngede otomatik bir atış.

#### Oyundaki Kullanım Amacı

`CFlyingManager`, oyundaki tüm mermi, büyü ve diğer fırlatılabilir nesnelerin oluşturulmasını, güncellenmesini, çizilmesini ve yok edilmesini düzenleyen merkezi bir sistemdir. Hem istemci tarafından tetiklenen (örneğin oyuncunun bir yetenek kullanması) hem de sunucu tarafından tetiklenen (örneğin özel olaylar veya NPC davranışları) uçan nesneleri destekler. Bu merkezi yönetim, kaynakların verimli kullanılmasını ve oyun mantığının tutarlı olmasını sağlar.

### `IFlyTargetableObject` Arayüzü

**Dosya:** `GameLib/FlyTarget.h`

`IFlyTargetableObject`, oyun dünyasında uçan mermiler veya yetenekler tarafından hedeflenebilen herhangi bir varlığın (örneğin karakterler, canavarlar) uygulaması gereken bir arayüzdür. Bu arayüz, uçan nesnelerin hedefleriyle nasıl etkileşim kuracağını standartlaştırır.

#### Temel Amaçları

*   Hedeflenebilir bir nesnenin, uçan bir mermi tarafından nişan alınacak kesin 3D pozisyonunu sağlamak.
*   Hedeflenebilir bir nesneye, bir mermi tarafından vurulduğunda hasar alma veya başka bir etki uygulama mekanizması sunmak.
*   Bir hedefin artık geçerli olmadığında (örneğin yok edildiğinde), kendisine nişan almış tüm uçan nesneleri bu durumdan haberdar etmesini sağlamak.

#### Arayüz Metotları

*   **`virtual D3DXVECTOR3 OnGetFlyTargetPosition() = 0;`**
    *   Bu saf sanal fonksiyon, implemente eden sınıf tarafından geçersiz kılınmalıdır.
    *   Hedefin anlık olarak bulunduğu ve uçan nesnenin nişan alması gereken dünya koordinatlarındaki pozisyonunu (`D3DXVECTOR3`) döndürür. Bu, genellikle karakterin merkezi veya belirli bir kemik pozisyonu olabilir.

*   **`virtual void OnShootDamage() = 0;`**
    *   Bu saf sanal fonksiyon, implemente eden sınıf tarafından geçersiz kılınmalıdır.
    *   Uçan bir nesne bu hedefe başarıyla isabet ettiğinde ve hasar uygulaması gerektiğinde çağrılır. Bu metodun içinde hasar hesaplaması, can düşürme veya diğer ilgili etkiler gerçekleştirilir.

#### Dahili Mekanizmalar (Hedef Takipçileri Yönetimi)

`IFlyTargetableObject` arayüzü, kendisini hedefleyen `CFlyTarget` nesnelerinin bir kaydını tutar (`m_FlyTargeterSet`). Bu, hedefin durumu değiştiğinde (örneğin yok olduğunda) tüm "nişancılara" haber verilmesini sağlar:

*   `protected: inline void ClearFlyTargeter();`
    *   Bu metot, hedef nesne yok edildiğinde veya artık hedeflenemez olduğunda çağrılmalıdır.
    *   `m_FlyTargeterSet` içindeki her `CFlyTarget` örneğinin `NotifyTargetClear()` metodunu çağırarak hedefin geçersizleştiğini bildirir. Sonrasında bu seti temizler.
*   `private: inline void AddFlyTargeter(CFlyTarget* pTargeter);`
    *   Bir `CFlyTarget` nesnesi bu `IFlyTargetableObject`'i hedeflemeye başladığında `CFlyTarget` tarafından çağrılır.
*   `private: inline void RemoveFlyTargeter(CFlyTarget* pTargeter);`
    *   Bir `CFlyTarget` nesnesi artık bu `IFlyTargetableObject`'i hedeflemediğinde `CFlyTarget` tarafından çağrılır.

### `CFlyTarget` Sınıfı

**Dosyalar:** `GameLib/FlyTarget.h`, `GameLib/FlyTarget.cpp`

`CFlyTarget` sınıfı, bir uçan nesnenin (örneğin `CFlyingInstance`) hedefini temsil eder. Bir hedef, ya `IFlyTargetableObject` arayüzünü uygulayan dinamik bir oyun varlığı ya da dünya üzerinde sabit bir 3D pozisyon olabilir.

#### Temel Özellikler ve Sorumluluklar

*   **Hedef Türü Yönetimi:** Hedefin bir nesne mi (`TYPE_OBJECT`) yoksa bir pozisyon mu (`TYPE_POSITION`) olduğunu `EType` enum'u ile takip eder.
*   **Nesne Hedefleri:**
    *   Eğer hedef bir nesne ise, o nesneye bir `IFlyTargetableObject*` (`m_pFlyTarget`) ile referans tutar.
    *   Hedeflenen nesneye kendisini bir "hedefleyici" (targeter) olarak kaydettirir (`m_pFlyTarget->AddFlyTargeter(this)`).
    *   Hedeflenen nesne yok edildiğinde veya bu `CFlyTarget` nesnesi yok edildiğinde/hedefi değiştiğinde kaydını sildirir (`m_pFlyTarget->RemoveFlyTargeter(this)`).
*   **Pozisyon Hedefleri:** Eğer hedef bir pozisyon ise, bu 3D pozisyonu (`m_v3FlyTargetPosition`) saklar.
*   **Dinamik Pozisyon Güncelleme:** Hedef bir nesne ise, `GetFlyTargetPosition()` çağrıldığında hedef nesnenin `OnGetFlyTargetPosition()` metodunu çağırarak en güncel pozisyonu alır ve bunu geçici olarak `m_v3FlyTargetPosition` (mutable) içinde saklar.
*   **Hedef Geçersizleşme Bildirimi:** Hedeflenen bir `IFlyTargetableObject`, `ClearFlyTargeter()` aracılığıyla bu `CFlyTarget`'ın `NotifyTargetClear()` metodunu çağırdığında, hedef türünü `TYPE_POSITION`'a dönüştürür (son bilinen pozisyonu kullanarak) ve nesne işaretçisini (`m_pFlyTarget`) `NULL` yapar. Bu, uçan nesnenin hedefsiz kalmamasını veya geçersiz bir işaretçiye erişmeye çalışmamasını sağlar.

#### `EType` Enum

*   `TYPE_NONE`: Geçerli bir hedef tanımlanmamış.
*   `TYPE_OBJECT`: Hedef, `IFlyTargetableObject` arayüzünü implemente eden bir oyun nesnesi.
*   `TYPE_POSITION`: Hedef, dünya koordinatlarında belirli bir 3D nokta.

#### Önemli Üye Değişkenler

*   `m_v3FlyTargetPosition`: `mutable D3DXVECTOR3`, Hedefin 3D pozisyonu. Nesne hedefleri için dinamik olarak güncellenir.
*   `m_pFlyTarget`: `IFlyTargetableObject*`, Hedeflenen nesneye işaretçi (eğer tür `TYPE_OBJECT` ise).
*   `m_eType`: `EType`, Hedefin türünü belirtir.

#### Önemli Metotlar

*   **`CFlyTarget()` (Kurucular):**
    *   Varsayılan kurucu (hedef `TYPE_NONE`).
    *   `CFlyTarget(IFlyTargetableObject* pFlyTarget)`: Bir nesneyi hedefler.
    *   `CFlyTarget(const D3DXVECTOR3& v3FlyTargetPosition)`: Bir pozisyonu hedefler.
    *   `CFlyTarget(const CFlyTarget& rhs)`: Kopya kurucu.
*   **`~CFlyTarget()` (Yıkıcı):** Eğer bir nesneyi hedefliyorsa, o nesneden kaydını siler.
*   **`void Clear()`:** Hedefi `TYPE_NONE` durumuna getirir.
*   **`bool IsObject()` / `bool IsPosition()` / `bool IsValidTarget()`:** Hedefin türünü veya geçerliliğini kontrol eder.
*   **`void NotifyTargetClear()`:** Hedeflenen nesne tarafından çağrıldığında, hedefi `TYPE_POSITION`'a dönüştürür.
*   **`const D3DXVECTOR3& GetFlyTargetPosition() const`:** Hedefin güncel pozisyonunu döndürür.
*   **`IFlyTargetableObject* GetFlyTarget()`:** Eğer hedef bir nesne ise, nesneye işaretçiyi döndürür (aksi halde `assert` tetiklenir).
*   **`CFlyTarget& operator=(const CFlyTarget& rhs)` (Atama Operatörü):** Bir `CFlyTarget`'ı diğerine atar, hedef nesne kayıtlarını doğru şekilde yönetir.

#### Oyundaki Kullanım Amacı

`CFlyTarget`, `CFlyingInstance` gibi uçan nesnelerin neye doğru hareket edeceğini ve potansiyel olarak neye çarpacağını tanımlamak için kullanılır. Bir yetenek kullanıldığında veya bir ok fırlatıldığında, hedefin bir karakter mi yoksa belirli bir nokta mı olduğu `CFlyTarget` aracılığıyla belirtilir. Bu yapı, dinamik ve statik hedeflerin esnek bir şekilde yönetilmesini sağlar.

### `CFlyTrace` Sınıfı

**Dosyalar:** `GameLib/FlyTrace.h`, `GameLib/FlyTrace.cpp`

`CFlyTrace` sınıfı, `CScreen`'den miras alır ve uçan nesnelerin (örneğin `CFlyingInstance`'a bağlı efektler) arkasında görsel bir iz veya "kuyruk" (trail) efekti oluşturmakla görevlidir. Bu iz, nesnenin önceki pozisyonlarını birleştirerek akıcı bir hareket illüzyonu yaratır.

#### Temel Özellikler ve Sorumluluklar

*   **Pozisyon Geçmişi Yönetimi:** Uçan nesnenin belirli bir süre boyunca (`m_fTailLength`) geçtiği pozisyonları ve bu pozisyonlara ulaştığı zaman damgalarını bir `std::deque` (`m_TimePositionDeque`) içinde saklar.
*   **İz Oluşturma ve Güncelleme:**
    *   `Create()`: Bir `CFlyingData::TFlyingAttachData` yapısından aldığı renk, uzunluk, boyut ve şekil gibi parametrelerle izin temel özelliklerini ayarlar.
    *   `UpdateNewPosition()`: Uçan nesnenin her yeni pozisyonu bu metoda bildirilir. Yeni pozisyon deque'in başına eklenirken, `m_fTailLength` süresini doldurmuş olan eski pozisyonlar deque'in sonundan çıkarılır.
*   **İz Çizimi (Rendering):**
    *   `Render()`: `m_TimePositionDeque`'deki pozisyon geçmişini kullanarak izi ekrana çizer. İz, birbirini takip eden pozisyonlar arasına çizilen dörtgenlerden (quads) oluşur.
    *   Bu dörtgenler, kameraya dönük olacak şekilde (billboard) yönlendirilir, böylece iz her açıdan düzgün görünür.
    *   İzin şekli (`m_bRectShape` true ise düz şerit, false ise uca doğru incelen) ve zamanla solma efekti (segmentin yaşına göre boyut veya alfa ayarlanarak) uygulanır.
    *   Saydamlık ve parlama efektleri için alfa karıştırma (genellikle additive blending) kullanılır.
    *   Çizimden önce basit bir görüş alanı (frustum) kontrolü ve saydamlık sorunlarını azaltmak için segmentlerin derinliğe göre sıralanması (painter's algorithm benzeri) yapılır.
*   **Bellek Yönetimi:** `CDynamicPool` kullanarak `CFlyTrace` nesnelerinin verimli bir şekilde oluşturulup yok edilmesini sağlar.

#### Önemli Üye Değişkenler

*   `m_TimePositionDeque`: `std::deque<std::pair<float, D3DXVECTOR3>>`, Zaman damgası ve pozisyon çiftlerini içeren deque. İzin geometrisini oluşturmak için kullanılır.
*   `m_bRectShape`: `bool`, `true` ise iz segmentleri sabit genişlikte, `false` ise segmentin yaşına göre incelen bir yapıda çizilir.
*   `m_dwColor`: `DWORD`, İzin ana rengi (ARGB formatında).
*   `m_fSize`: `float`, İzin başlangıçtaki temel genişliği/kalınlığı.
*   `m_fTailLength`: `float`, Bir pozisyon kaydının deque içinde kalma süresi (saniye cinsinden), dolayısıyla izin görsel uzunluğunu belirler.

#### Önemli Metotlar

*   **`CFlyTrace()` / `~CFlyTrace()` / `Destroy()` / `__Initialize()`:** Yapıcı, yıkıcı, kaynak temizleme ve başlangıç değerlerini ayarlama.
*   **`void Create(const CFlyingData::TFlyingAttachData& rFlyingAttachData)`:** Verilen eklenti verilerinden izin özelliklerini (renk, uzunluk, boyut, şekil) ayarlar.
*   **`void UpdateNewPosition(const D3DXVECTOR3& v3Position)`:** Uçan nesnenin yeni pozisyonunu kaydeder ve eskiyen pozisyonları temizler. Bu metot, genellikle ana `CFlyingInstance` tarafından periyodik olarak çağrılır.
*   **`void Render()`:** İzi, depolanan pozisyon verilerine dayanarak bir dizi birleştirilmiş ve kameraya dönük dörtgen olarak çizer. Çeşitli Direct3D çizim durumlarını yöneterek saydamlık ve renk efektlerini uygular.
*   **`static CFlyTrace* New()` / `static void Delete(CFlyTrace* pkInst)` / `static void DestroySystem()`:** `CDynamicPool` aracılığıyla bellek yönetimi için statik metotlar.

#### `TFlyVertex` (İç Yapı - `FlyTrace.cpp`'de kullanılır)

Render metodu içinde tanımlanan geçici bir yapı olup, bir iz segmentinin tepe noktalarını tanımlar:

*   `D3DXVECTOR3 p`: Tepe noktasının pozisyonu.
*   `DWORD c`: Tepe noktasının rengi.
*   `D3DXVECTOR2 t`: Tepe noktasının doku koordinatı (aktif olarak kullanılmıyor gibi görünse de tanımlıdır).

#### Oyundaki Kullanım Amacı

`CFlyTrace`, genellikle okların, büyülerin veya diğer hızlı hareket eden nesnelerin arkasında görsel bir devamlılık ve hız hissi yaratmak için kullanılır. `CFlyingInstance`'a bağlı bir efekt (`TAttachEffectInstance` içinde `pFlyTrace` olarak) olarak çalışır ve uçan nesnenin yörüngesini vurgular. Parametreleri (`CFlyingData` üzerinden) ayarlanarak farklı görünümlerde (kalın, ince, uzun, kısa, renkli) izler oluşturulabilir.

### `CGameEventManager` Sınıfı

**Dosyalar:** `GameLib/GameEventManager.h`, `GameLib/GameEventManager.cpp`

`CGameEventManager`, `CSingleton` ve `CScreen` sınıflarından türetilmiş bir yönetici sınıfıdır. Oyun ekranında meydana gelen çeşitli genel olayları, özellikle de kamera ve ekran efektlerini tetikleyen olayları (örneğin, ekran sallanması) yönetmek için tasarlanmıştır. İleride daha kapsamlı sinematik olayları da kontrol etmesi planlanmış olabilir.

#### Temel Özellikler ve Sorumluluklar

*   **Merkez Pozisyon Yönetimi:** Olayların etkisini değerlendirmek için bir referans noktası olan `m_CenterPosition`'ı tutar ve günceller. Bu genellikle ana karakterin veya kameranın odak noktasının yakınında bir yer olabilir.
*   **Ekran Sallanma Olayı İşleme:** `ProcessEventScreenWaving` metodu aracılığıyla, bir aktörün belirli bir hareketinden (animasyonundan) kaynaklanan ekran sallanma olaylarını işler. Olayın etkisi, aktörün `m_CenterPosition`'a olan mesafesine ve olayın tanımlandığı menzile göre belirlenir.
*   **Ekran Efektleri Tetikleme:** `CScreen` taban sınıfından gelen (varsayımsal) `SetScreenEffectWaving` gibi fonksiyonları kullanarak görsel ekran efektlerini başlatır.

#### Önemli Üye Değişkenler

*   `m_CenterPosition`: `TPixelPosition` (muhtemelen `D3DXVECTOR3` benzeri bir yapı), olayların işlenmesi için referans alınan merkezi bir dünya pozisyonunu saklar.

#### Önemli Metotlar

*   **`CGameEventManager()` / `~CGameEventManager()`:** Yapıcı ve yıkıcı metotlar.
*   **`void SetCenterPosition(float fx, float fy, float fz)`:** `m_CenterPosition`'ı belirtilen dünya koordinatlarına ayarlar. `fy` koordinatının `-fy` olarak atanması dikkat çekicidir; bu, muhtemelen koordinat sistemi dönüşümleriyle ilgilidir.
*   **`void Update()`:** Şu anda boş bir implementasyona sahiptir. Gelecekte zamanla gelişen veya periyodik oyun olaylarını yönetmek için kullanılabilir.
*   **`void ProcessEventScreenWaving(CActorInstance* pActorInstance, const CRaceMotionData::TScreenWavingEventData* c_pData)`:**
    *   Bir `CActorInstance` (olayı tetikleyen aktör) ve bir `TScreenWavingEventData` (sallanma olayının süresi, gücü, etki menzili gibi parametreleri içeren yapı) alır.
    *   Aktörün pozisyonunun, `m_CenterPosition` etrafındaki `c_pData->iAffectingRange` ile tanımlanan etki menzili içinde olup olmadığını kontrol eder.
    *   Eğer aktör menzil içindeyse, `SetScreenEffectWaving()` fonksiyonunu (muhtemelen `CScreen` taban sınıfından) `c_pData->fDurationTime` ve `c_pData->iPower` parametreleriyle çağırarak ekran sallanma efektini tetikler.

#### Oyundaki Kullanım Amacı

`CGameEventManager`, genellikle karakterlerin güçlü saldırıları, büyük patlamalar veya önemli oyun anları gibi durumlarda oyuncuya görsel geri bildirim sağlamak amacıyla kullanılır. Özellikle `ProcessEventScreenWaving` metodu, animasyon sistemindeki belirli olaylar (genellikle `CRaceMotionData` içinde tanımlanır) tarafından tetiklenerek, aksiyonu daha etkileyici hale getiren ekran sallanmaları gibi efektleri uygun koşullarda (mesafe kontrolü yaparak) başlatır. `m_CenterPosition`, genellikle oyuncunun kamerasının veya ana karakterinin pozisyonuna yakın bir değere ayarlanır, böylece efektler oyuncunun algı alanıyla ilişkili olur.

### `GameType.h` ve `GameType.cpp` Dosyaları

Bu dosyalar, oyun içinde kullanılan temel türleri, genel konfigürasyonları ve özellikle karakterlerle ilişkili çeşitli veri yapılarını tanımlar. En önemli bölüm, karakter ırklarına ve sınıflarına özgü saldırı, çarpışma ve eklenti verilerini yönetmek için tasarlanmış `NRaceData` isim alanıdır.

#### Global Tanımlamalar

*   **`g_fGameFPS` (`float`):** Oyunun hedeflenen saniye başına kare sayısını (FPS) tutan global bir değişkendir. `GameType.cpp` içinde `60.0f` olarak başlatılır.
*   **`MOTION_KEY` Makroları:**
    *   `MAKE_MOTION_KEY(mode, index)`: Verilen animasyon modu (`mode`) ve animasyon ana indeksi (`index`) bilgilerini paketleyerek tek bir `DWORD` (MOTION_KEY) oluşturur.
    *   `MAKE_RANDOM_MOTION_KEY(mode, index, type)`: Animasyon modu, ana indeksi ve bir alt tip/indeks (`type`) bilgisini paketleyerek bir `MOTION_KEY` oluşturur.
    *   `GET_MOTION_MODE(key)`: Verilen `MOTION_KEY`'den animasyon modunu çıkarır.
    *   `GET_MOTION_INDEX(key)`: Verilen `MOTION_KEY`'den animasyon ana indeksini çıkarır.
    *   `GET_MOTION_SUB_INDEX(key)`: Verilen `MOTION_KEY`'den animasyon alt indeksini/tipini çıkarır.
    Bu makrolar, animasyonların verimli bir şekilde kodlanması ve yönetilmesi için kullanılır.

#### `NRaceData` İsim Alanı

Bu isim alanı, karakterlerin ırklarına, sınıflarına ve cinsiyetlerine bağlı olarak değişebilen çeşitli oyun mekaniği verilerini (saldırı tanımları, çarpışma geometrileri, modellere eklenen efektler/nesneler) tanımlamak, yüklemek ve kaydetmek için kullanılır.

##### Enum Tanımları

*   **`EJobs`**: Karakter sınıflarını listeler:
    *   `JOB_WARRIOR` (Savaşçı)
    *   `JOB_ASSASSIN` (Ninja)
    *   `JOB_SURA` (Sura)
    *   `JOB_SHAMAN` (Şaman)
    *   `JOB_WOLFMAN` (Lycan)
    *   `JOB_MAX_NUM` (Maksimum sınıf sayısı)
*   **`ESex`**: Karakter cinsiyetlerini listeler:
    *   `SEX_MALE` (Erkek)
    *   `SEX_FEMALE` (Kadın)
    *   `SEX_MAX_NUM` (Maksimum cinsiyet sayısı)
*   **`EAttackType`**: Temel saldırı türlerini belirtir:
    *   `ATTACK_TYPE_SPLASH`: Alan etkili saldırı.
    *   `ATTACK_TYPE_SNIPE`: Tek bir hedefe yönelik saldırı.
*   **`EHitType`**: Bir vuruşun sonucunu veya türünü belirtir:
    *   `HIT_TYPE_NONE`: Vuruş yok.
    *   `HIT_TYPE_GREAT`: "Harika" vuruş (genellikle kritik veya güçlü vuruş).
    *   `HIT_TYPE_GOOD`: "İyi" vuruş (normal vuruş).
*   **`EMotionType`**: Animasyonların genel kategorisini belirtir:
    *   `MOTION_TYPE_NONE`: Belirli bir animasyon türü yok.
    *   `MOTION_TYPE_NORMAL`: Normal animasyon (örneğin yürüme, koşma, bekleme).
    *   `MOTION_TYPE_COMBO`: Kombo saldırı animasyonu.
    *   `MOTION_TYPE_SKILL`: Yetenek kullanım animasyonu.
*   **`ECollisionType`**: Bir çarpışma verisinin amacını belirtir:
    *   `COLLISION_TYPE_NONE`: Çarpışma verisi yok.
    *   `COLLISION_TYPE_BODY`: Karakterin ana gövde çarpışması.
    *   `COLLISION_TYPE_ATTACKING`: Saldırı sırasında kullanılan vuruş alanı.
    *   `COLLISION_TYPE_DEFENDING`: Savunma sırasında kullanılan (belki bloklama) alanı.
    *   `COLLISION_TYPE_SPLASH`: Alan etkili saldırıların etki alanı.
*   **`ECollisionShape`**: Basit çarpışma geometrilerinin şeklini belirtir:
    *   `COLLISION_SHAPE_SPHERE`: Küre şeklinde çarpışma alanı.
    *   `COLLISION_SHAPE_CYLINDER`: Silindir şeklinde çarpışma alanı (kullanımı `TCollisionData` yüklemesinde belirgin değil).
*   **`EAttachingDataType`**: Karakter modellerinin kemiklerine eklenebilen veri türlerini belirtir:
    *   `ATTACHING_DATA_TYPE_NONE`: Eklenti yok.
    *   `ATTACHING_DATA_TYPE_COLLISION_DATA`: Çarpışma verisi eklentisi.
    *   `ATTACHING_DATA_TYPE_EFFECT`: Görsel efekt eklentisi.
    *   `ATTACHING_DATA_TYPE_OBJECT`: Başka bir model/nesne eklentisi.
    *   `ATTACHING_DATA_TYPE_MAX_NUM`.

##### Temel Veri Yapıları (Structs)

*   **`SAttackData`**: Bir saldırının temel özelliklerini tanımlar.
    *   `iAttackType`: `EAttackType` türünden saldırı tipi.
    *   `iHittingType`: `EHitType` türünden vuruş tipi.
    *   `fInvisibleTime`: Saldırı sonrası karakterin görünmez kalacağı süre (saniye).
    *   `fExternalForce`: Vuruşun hedefe uygulayacağı dış kuvvet (itme/savurma).
    *   `fStiffenTime`: Vuruşun hedefte yaratacağı sersemleme/kilitlenme süresi (saniye).
    *   `iHitLimitCount`: Bu saldırının maksimum kaç hedefe vurabileceği (0 ise sınırsız veya farklı yorumlanır).
*   **`SHitData`**: Bir saldırı animasyonu içindeki belirli bir vuruş anını veya aralığını detaylandırır.
    *   `fAttackStartTime`: Vuruşun aktif olmaya başladığı zaman (animasyon süresi içinde).
    *   `fAttackEndTime`: Vuruşun aktifliğinin sona erdiği zaman.
    *   `fWeaponLength`: Silahın veya vuruşun menzilini etkileyen uzunluk.
    *   `strBoneName`: Vuruşun kaynağı olan kemik adı (eğer silahsızsa veya özel bir noktadan çıkıyorsa).
    *   `mapHitPosition` (`THitTimePositionMap`): Animasyon sırasında vuruş küresinin zamanla değişen pozisyonlarını (`CDynamicSphereInstance` olarak) tutar. Bu, hareketli saldırılarda vuruş alanının doğru takibi için kullanılır.
    *   `Load(CTextFileLoader& rTextFileLoader)`: Bu yapıyı bir metin dosyasından yükler.
*   **`SMotionAttackData`**: `SAttackData`'dan miras alır ve bir animasyonla ilişkili daha kapsamlı saldırı verilerini tutar.
    *   `iMotionType`: `EMotionType` türünden animasyon kategorisi.
    *   `HitDataContainer` (`THitDataContainer` - yani `std::vector<THitData>`): Bir animasyon sırasında birden fazla vuruş anı veya farklı vuruş alanları olabileceği için `SHitData` örneklerini bir vektörde tutar.
*   **`SCollisionData`**: Bir çarpışma alanını tanımlar.
    *   `iCollisionType`: `ECollisionType` türünden çarpışmanın amacı.
    *   `SphereDataVector` (`CSphereCollisionInstanceVector`): Çarpışma alanını oluşturan bir veya daha fazla kürenin verilerini tutar (`CSphereCollisionInstance`, `EterLib`'den gelir ve pozisyon ile yarıçap içerir).
*   **`SAttachingEffectData`**: Bir modele eklenen bir efektin özelliklerini tanımlar.
    *   `strFileName`: Efektin `.mse` gibi script dosyasının yolu.
    *   `v3Position`: Efektin eklendiği kemiğe göre yerel pozisyonu.
    *   `v3Rotation`: Efektin eklendiği kemiğe göre yerel rotasyonu (Euler açıları).
*   **`SAttachingObjectData`**: Bir modele eklenen bir alt nesnenin (örneğin bir kalkan, özel bir aksesuar) özelliklerini tanımlar.
    *   `strFileName`: Eklenecek nesnenin model dosyasının yolu.
*   **`SAttachingData`**: Çeşitli türlerdeki eklentileri genel bir yapıda toplar.
    *   `dwType`: `EAttachingDataType` türünden eklentinin tipi.
    *   `isAttaching`: Eklentinin aktif olup olmadığını belirten bayrak.
    *   `dwAttachingModelIndex`: Eklentinin bağlandığı ana model içindeki bir alt modelin indeksi (eğer varsa).
    *   `strAttachingBoneName`: Eklentinin bağlandığı kemiğin adı.
    *   `pCollisionData` (`TCollisionData*`): Eğer `dwType` `ATTACHING_DATA_TYPE_COLLISION_DATA` ise bu işaretçi geçerli olur.
    *   `pEffectData` (`TAttachingEffectData*`): Eğer `dwType` `ATTACHING_DATA_TYPE_EFFECT` ise bu işaretçi geçerli olur.
    *   `pObjectData` (`TAttachingObjectData*`): Eğer `dwType` `ATTACHING_DATA_TYPE_OBJECT` ise bu işaretçi geçerli olur.
    Bu işaretçiler, `CDynamicPool`'lerden alınan nesneleri gösterir.

##### Bellek Yönetimi (Dinamik Havuzlar)

Sıkça oluşturulup yok edilebilecek `TCollisionData`, `TAttachingEffectData`, ve `TAttachingObjectData` türleri için bellek verimliliği sağlamak amacıyla `CDynamicPool` kullanılır:
*   `g_CollisionDataPool`: `TCollisionData` nesneleri için.
*   `g_EffectDataPool`: `TAttachingEffectData` nesneleri için.
*   `g_ObjectDataPool`: `TAttachingObjectData` nesneleri için.
`DestroySystem()` fonksiyonu, oyun sonlandığında bu havuzlardaki tüm belleği serbest bırakır.

##### Veri Yükleme ve Kaydetme Fonksiyonları

`NRaceData` isim alanı, yukarıda bahsedilen veri yapılarının metin tabanlı dosyalardan (`CTextFileLoader` aracılığıyla) yüklenmesi ve bu dosyalaradüzgün formatta kaydedilmesi için bir dizi `Load*Data` ve `Save*Data` fonksiyonu sunar.

*   **Yükleme (`Load*`):**
    *   `LoadAttackData()`, `THitData::Load()`, `LoadMotionAttackData()`, `LoadCollisionData()`, `LoadEffectData()`, `LoadObjectData()`, `LoadAttachingData()`.
    *   Bu fonksiyonlar, belirli bir hiyerarşi ve anahtar kelime setine sahip metin dosyalarını ayrıştırarak ilgili yapıları doldurur.
    *   `LoadEffectData` ayrıca yüklediği efekti `CEffectManager`'a kaydeder.
    *   `LoadAttachingData`, eklenti tipine göre dinamik havuzlardan bellek ayırır ve ilgili spesifik yükleme fonksiyonunu çağırır.
*   **Kaydetme (`Save*`):**
    *   `SaveAttackData()`, `SaveMotionAttackData()`, `SaveCollisionData()`, `SaveEffectData()`, `SaveObjectData()`, `SaveAttachingData()`.
    *   Bu fonksiyonlar, mevcut veri yapılarının içeriğini, `Load*` fonksiyonlarının okuyabileceği formatta, genellikle sekmelerle (indentation) okunabilirliği artırılmış bir şekilde dosyaya yazar.

##### Oyundaki Kullanım Amacı

`GameType.h` ve özellikle `NRaceData` içindeki tanımlamalar, Metin2 istemcisinin karakterlerle ilgili çok sayıda veriyi (saldırıların nasıl çalıştığı, karakterlerin ne tür çarpışma alanlarına sahip olduğu, hangi efektlerin hangi kemiklere bağlı olduğu vb.) harici dosyalarda (genellikle `.msm` - motion script, veya ırk/karakter yapılandırma dosyaları) tanımlamasına ve yönetmesine olanak tanır. Bu, oyun tasarımcılarının ve geliştiricilerinin, oyunun temel kodunu değiştirmeden karakter davranışlarını, saldırı özelliklerini ve görsel eklentilerini kolayca düzenlemesini ve dengelemesini sağlar. Dinamik havuzların kullanımı, bu verilerin yüklenmesi ve işlenmesi sırasında bellek yönetimini optimize eder.

### `GameUtil.h` ve `GameUtil.cpp` Dosyaları

Bu dosyalar, oyun içinde sıkça ihtiyaç duyulan çeşitli matematiksel ve geometrik yardımcı fonksiyonları içerir. Başlıca odak alanları dinamik nesneler arası temel çarpışma tespiti ve 2D/3D açı/rotasyon hesaplamalarıdır.

#### Çarpışma Tespiti Fonksiyonları

Bu fonksiyonlar, hareket eden (dinamik) basit geometrik şekiller arasındaki çarpışmaları kontrol eder.

*   **`bool DetectCollisionDynamicSphereVSDynamicSphere(const CDynamicSphereInstance& c_rSphere1, const CDynamicSphereInstance& c_rSphere2)`**
    *   **Amaç:** İki dinamik (hareketli) kürenin bir sonraki pozisyonlarına doğru hareket ederken çarpışıp çarpışmadığını tespit eder.
    *   **Yöntem:**
        1.  **AABB (Axis-Aligned Bounding Box) Testi:** Öncelikle, iki kürenin hareketlerini kapsayan AABB'lerin kesişip kesişmediğini kontrol ederek hızlı bir dışlama yapar. Eğer AABB'ler kesişmiyorsa, çarpışma mümkün değildir.
        2.  **Doğru Parçası Kesişimi:** Her iki kürenin `v3LastPosition` (önceki pozisyon) ve `v3Position` (mevcut/hedef pozisyon) noktaları arasında birer doğru parçası (hareket vektörü) tanımlanır. `IntersectLineSegments` (muhtemelen `EterLib`'den) fonksiyonu kullanılarak bu iki doğru parçasının birbirine en yakın noktaları (`vA` ve `vB`) bulunur.
        3.  **Mesafe Kontrolü:** En yakın noktalar (`vA` ve `vB`) arasındaki mesafenin karesi (`D3DXVec3LengthSq(&(vA - vB))`) hesaplanır. Bu değer, iki kürenin yarıçapları toplamının (`r = c_rSphere1.fRadius + c_rSphere2.fRadius`) karesinden (`rsq = r * r`) küçük veya eşitse, kürelerin hareket yolları boyunca bir çarpışma meydana geldiği kabul edilir.
    *   **Not:** Kod içinde yorum satırına alınmış birçok alternatif çarpışma hesaplama yöntemi bulunmaktadır. Aktif olan yöntem, "sürekli çarpışma tespiti" (Continuous Collision Detection - CCD) için basitleştirilmiş bir yaklaşımdır ve kürelerin hareket yollarının yakınlığını kontrol eder.

*   **`bool DetectCollisionDynamicZCylinderVSDynamicZCylinder(const CDynamicSphereInstance& c_rSphere1, const CDynamicSphereInstance& c_rSphere2)`**
    *   **Amaç:** Esasen XY düzleminde hareket eden iki 2D dairenin çarpışmasını tespit eder. Fonksiyon "ZCylinder" olarak adlandırılsa da, Z eksenindeki pozisyonları ve hareketleri göz ardı eder. Girdi olarak `CDynamicSphereInstance` alır ancak Z bileşenlerini sıfırlayarak kullanır.
    *   **Yöntem:** `DetectCollisionDynamicSphereVSDynamicSphere` ile aynı temel adımları izler:
        1.  Kürelerin (aslında dairelerin) Z pozisyonları sıfırlanır.
        2.  2D AABB testi yapılır.
        3.  2D doğru parçalarının (`v3LastPosition`'dan `v3Position`'a, Z=0 alınarak) birbirine en yakın noktaları bulunur.
        4.  Bu noktalar arasındaki 2D mesafenin karesi, dairelerin yarıçapları toplamının karesiyle karşılaştırılır.

*   **Yorum Satırındaki Diğer Fonksiyonlar:**
    *   `DetectCollisionDynamicSphereVSStaticPlane`, `DetectCollisionDynamicSphereVSStaticSphere`, `DetectCollisionDynamicSphereVSStaticCylinder` gibi birçok fonksiyon başlığı `GameUtil.h` içinde yorum satırı olarak bulunmaktadır. Bu, bu tür çarpışmalar için fonksiyonların planlandığını ancak tamamlanmadığını veya kullanımdan kaldırıldığını gösterir.

#### Açı ve Rotasyon Yardımcı Fonksiyonları

Bu fonksiyonlar, 2D uzayda yönleri temsil eden açıları hesaplamak, aralarında enterpolasyon yapmak ve dönüş yönlerini belirlemek için kullanılır. Açılar genellikle derece cinsindendir.

*   **`float GetDegreeFromPosition(float x, float y)`**
    *   (0,0) orijininden verilen (x,y) noktasına doğru olan vektörün açısını derece cinsinden hesaplar. Genellikle Y ekseninin negatif yönü (yukarı) 0 derece kabul edilir ve açılar saat yönünde artar. `atan2` yerine `acos` ve ardından `x` koordinatının işaretine göre düzeltme yapar.
*   **`float GetDegreeFromPosition2(float sx, float sy, float ex, float ey)`**
    *   Başlangıç noktası (`sx`, `sy`) ile bitiş noktası (`ex`, `ey`) arasındaki yön vektörünün açısını `GetDegreeFromPosition` kullanarak hesaplar.
*   **`float GetInterpolatedRotation(float begin, float end, float curRate)`**
    *   `begin` (başlangıç açısı) ile `end` (hedef açı) arasında, `curRate` (0.0 ile 1.0 arasında bir oran) kadar doğrusal enterpolasyon yapar.
    *   Dönüşün her zaman en kısa yolu (saat yönü veya tersi) izlemesini sağlar. Örneğin, 350 dereceden 10 dereceye enterpolasyon yaparken, 20 derecelik yolu kullanır (340 derecelik uzun yol yerine). 360 derece sınırını (örn: 350 -> 10) doğru şekilde yönetir.
*   **`bool IsCWRotation(float begin, float end)` ve `bool IsCCWRotation(float begin, float end)`**
    *   Bu fonksiyonlar, `begin` açısından `end` açısına gitmek için basitçe saat yönü (CW) veya saat yönünün tersi (CCW) bir dönüşün gerekip gerekmediğini kontrol etmeye çalışır. Ancak implementasyonları (`IsCCWRotation` için `begin - end < 0`), 360 derece geçişlerini ve en kısa yolu her zaman doğru bir şekilde ele almayabilir. `GetInterpolatedRotation` veya `GetRotatingDirection` gibi fonksiyonlar en kısa yol tespiti için daha güvenilirdir.
*   **`bool IsCWAcuteAngle(float begin, float end)` ve `bool IsCCWAcuteAngle(float begin, float end)`**
    *   İki açı arasındaki dar açının (180 dereceden küçük) hangi yönde (CW veya CCW) olduğunu belirlemeyi amaçlar.
*   **`float GetDegreeDifference(float fSource, float fTarget)`**
    *   İki açı (`fSource`, `fTarget`) arasındaki en kısa açısal farkı (her zaman pozitif ve 0-180 derece aralığında) hesaplar.
*   **`int GetRotatingDirection(float fSource, float fTarget)` (`EDegree_Direction` enum'u ile)**
    *   `fSource` açısından `fTarget` açısına ulaşmak için en kısa dönüşün yönünü belirler.
    *   Dönüş yönü `EDegree_Direction` enum sabitlerinden biri olarak döndürülür:
        *   `DEGREE_DIRECTION_SAME`: Açılar aynı (veya çok yakın).
        *   `DEGREE_DIRECTION_RIGHT`: Saat yönünde (CW) dönüş.
        *   `DEGREE_DIRECTION_LEFT`: Saat yönünün tersine (CCW) dönüş.

#### Kamera ve Karakter Rotasyon Dönüşümleri

*   **`float CameraRotationToCharacterRotation(float fCameraRotation)`**
*   **`float CharacterRotationToCameraRotation(float fCharacterRotation)`**
    *   Bu iki fonksiyon, kamera ve karakterin Y ekseni etrafındaki rotasyonları arasında dönüşüm yapar. Her ikisi de `fmod((540.0f - rotation), 360.0f)` formülünü kullanır.
    *   Bu dönüşüm, genellikle karakterin "ileri" baktığı yön ile kameranın baktığı yön arasında 180 derecelik bir fark olduğunda veya koordinat sistemleri arasında bir uyum gerektiğinde kullanılır (örneğin, kamera karakteri arkadan takip ederken, W tuşuna basıldığında karakterin kameranın "ileri" yönüne gitmesi için). 540 (yani 360+180) kullanımı, 180 derecelik bir kaydırma ve mod alma işlemini birleştirir.

#### Oyundaki Kullanım Amacı

`GameUtil` içindeki fonksiyonlar, oyun motorunun çeşitli alt sistemleri için temel yapı taşlarıdır:
*   **Çarpışma Tespiti:** Basit saldırıların hedefe isabet edip etmediğini, karakterlerin birbirleriyle veya mermilerle çarpışıp çarpışmadığını kontrol etmek için kullanılır.
*   **Rotasyon ve Yön Hesaplamaları:** Karakterlerin ve NPC'lerin hareket yönlerini belirlemek, hedeflere doğru dönmelerini sağlamak, animasyonların doğru yöne bakmasını sağlamak, kamera hareketlerini yumuşatmak ve kullanıcı girdisine göre karakteri yönlendirmek gibi birçok durumda kritik öneme sahiptir.
*   **Enterpolasyon:** Animasyonları veya hareketleri daha akıcı hale getirmek için açıların veya pozisyonların yumuşak geçişlerle güncellenmesinde kullanılır.

### `IBackground` Arayüzü

**Dosya:** `GameLib/Interface.h`

`IBackground`, oyun dünyasının arka plan özelliklerini (özellikle belirli bir noktanın geçilebilir olup olmadığını) sorgulamak için kullanılan bir arayüz (interface) sınıfıdır. `CSingleton<IBackground>`'dan türetilmiştir, bu da ona global bir erişim noktası sağlar.

#### Temel Amaç

*   Oyun dünyasındaki belirli koordinatların durumunu (örneğin, bir karakterin o noktaya hareket edip edemeyeceği, bir nesnenin o noktaya yerleştirilip yerleştirilemeyeceği) soyut bir şekilde sorgulamak için standart bir yöntem sunmak.
*   Genellikle çarpışma tespiti, karakter hareketi veya yapay zeka (AI) yol bulma (pathfinding) algoritmaları tarafından kullanılır.

#### Arayüz Metotları

*   **`virtual bool IsBlock(int x, int y) = 0;`**
    *   Bu saf sanal (pure virtual) bir metottur ve `IBackground` arayüzünü implemente eden her sınıf tarafından tanımlanmalıdır.
    *   Verilen dünya koordinatlarındaki (`x`, `y`) bir noktanın engellenmiş (geçilemez veya dolu) olup olmadığını kontrol eder.
    *   Eğer nokta engellenmişse `true`, aksi halde `false` döndürür.

#### Oyundaki Kullanım Amacı

`IBackground` arayüzü, genellikle oyun haritasını (`CMapOutdoor` veya benzeri bir sınıf) temsil eden bir sınıf tarafından implemente edilir. Bu sayede, oyunun farklı modülleri (karakter kontrolü, AI, yetenek sistemleri vb.) haritanın belirli noktalarının geçilebilirliği hakkında bilgi alabilirler. Örneğin, bir karakterin belirli bir hedefe doğru hareket etmeye çalışmadan önce, hedefin `IsBlock` ile kontrol edilerek ulaşılabilir olup olmadığına bakılabilir.

### `CItemData` Sınıfı

**Dosyalar:** `GameLib/ItemData.h`, `GameLib/ItemData.cpp`

`CItemData` sınıfı, Metin2 istemcisindeki her bir eşyanın (item) tüm tanımlayıcı verilerini, özelliklerini, görsel kaynaklarını ve oyun mekanikleriyle ilgili parametrelerini kapsayan merkezi bir veri yapısıdır. Bu sınıf, genellikle sunucudan veya istemci tarafındaki prototip dosyalarından (örneğin, `item_proto`) yüklenen eşya bilgilerini ve bu eşyalarla ilişkili 3D modeller, ikonlar gibi grafik kaynaklarını yönetir.

#### Temel Özellikler ve Sorumluluklar

*   **Eşya Prototip Verileri:** Her `CItemData` örneği, bir eşyanın temel özelliklerini içeren bir `TItemTable` yapısını saklar. Bu yapı, eşyanın VNUM (benzersiz ID), ismi, türü, alt türü, ağırlığı, boyutu, giyilebilirlik bayrakları (`dwWearFlags`), kullanım kısıtlama bayrakları (`ullAntiFlags`), genel bayrakları (`dwFlags`), bağışıklık bayrakları (`dwImmuneFlag`), alım/satım fiyatları, seviye/stat limitleri (`aLimits`), efsunları (`aApplies`), özel değerleri (`alValues`), soketleri (`alSockets`) ve yükseltilmiş versiyonunun VNUM'u (`dwRefinedVnum`) gibi kritik bilgileri içerir.
*   **Grafik Kaynak Yönetimi:**
    *   **Modeller:** Eşyanın karakter üzerinde giyildiğinde görünen ana 3D modeli (`m_pModelThing`), varsa alternatif/alt modeli (`m_pSubModelThing`) ve yere bırakıldığında görünen modeli (`m_pDropModelThing`) için `CGraphicThing` işaretçileri tutar. Ayrıca, uzak mesafelerde performans optimizasyonu için kullanılan Seviye Detayı (LOD) modellerini (`m_pLODModelThingVector`) de yönetebilir.
    *   **İkon:** Envanterde veya diğer arayüzlerde gösterilen eşya ikonunu (`m_pIconImage` - `CGraphicSubImage*`) tutar.
    *   Bu kaynaklar, dosya isimleri (`m_strModelFileName`, `m_strIconFileName` vb.) üzerinden `CResourceManager` aracılığıyla yüklenir.
*   **Ek Bilgiler:**
    *   Eşyanın kısa açıklaması (`m_strDescription`) ve özeti (`m_strSummary`).
    *   Eşyaya bağlı olabilen ek veriler (örneğin, silahlar için çarpışma verileri) için bir `NRaceData::TAttachingDataVector` (`m_AttachingDataVector`).
*   **Veri Erişimi:** Sınıf, sakladığı tüm bu bilgilere erişmek için çok sayıda `Get*` (örneğin, `GetName()`, `GetType()`, `GetApply()`, `GetModelThing()`) ve `Is*` (örneğin, `IsAntiFlag()`, `IsEquipment()`) metodu sağlar.
*   **Bellek Yönetimi:** `CItemData` nesneleri, sık oluşturulup yok edilebilecekleri için `CDynamicPool<CItemData>` (`ms_kPool`) kullanılarak verimli bir şekilde yönetilir. `CItemData::New()` ve `CItemData::Delete()` statik metodları bu havuz üzerinden örneklerin alınmasını ve iade edilmesini sağlar. `DestroySystem()` metodu ise oyun kapanırken tüm havuzu temizler.
*   **Sistem Entegrasyonları:** Kod içinde `#if defined(ENABLE_...)` direktifleriyle sarmalanmış bölümler, Aura Sistemi (`ENABLE_AURA_COSTUME_SYSTEM`), Acce Sistemi (`ENABLE_ACCE_COSTUME_SYSTEM`), Evcil Hayvan Sistemi (`ENABLE_GROWTH_PET_SYSTEM`) gibi çeşitli opsiyonel oyun sistemleriyle entegrasyonu ve bu sistemlere özel ek veri ve fonksiyonları (örneğin, ölçekleme tabloları, özel efekt ID'leri) barındırır.

#### Önemli Enum Tanımları (`CItemData.h` içinde)

`CItemData.h` dosyası, eşyaların özelliklerini ve davranışlarını kategorize etmek için çok sayıda `enum` tanımlar:

*   **`EItemType`**: Eşyanın ana kategorisini belirtir (örneğin, `ITEM_TYPE_WEAPON`, `ITEM_TYPE_ARMOR`, `ITEM_TYPE_USE`, `ITEM_TYPE_COSTUME`, `ITEM_TYPE_DS`, `ITEM_TYPE_PET`).
*   **Alt Tür Enum'ları**:
    *   `EWeaponSubTypes`: Kılıç, Bıçak, Yay, Çift-El, Çan, Yelpaze, Ok vb.
    *   `EArmorSubTypes`: Vücut Zırhı, Kask, Kalkan, Bilezik, Ayakkabı, Kolye, Küpe vb.
    *   `ECostumeSubTypes`: Kostüm (Vücut), Kostüm (Saç), Binek Kostümü, Acce Kostümü, Silah Kostümü, Aura Kostümü vb.
    *   `EUseSubTypes`: İksir, Parşömen (Tılsım), Yükseltme Nesnesi, Sandık Anahtarı, Yem, Beceri Kitabı vb.
    *   Ayrıca `EMaterialSubTypes`, `EResourceSubTypes`, `EDragonSoulSubType`, `EMetinSubTypes` gibi birçok alt kategori için enumlar mevcuttur.
*   **`ELimitTypes`**: Eşya kullanımı için geçerli olabilecek sınırlama türleri (LIMIT_LEVEL, LIMIT_STR, LIMIT_REAL_TIME, LIMIT_DURATION vb.).
*   **`EItemAntiFlag`**: Eşyanın belirli eylemlere karşı kısıtlamalarını tanımlayan bayraklar (örneğin, `ITEM_ANTIFLAG_FEMALE` - kadın karakterler giyemez, `ITEM_ANTIFLAG_SELL` - satılamaz, `ITEM_ANTIFLAG_STACK` - istiflenemez). Bu, `uint64_t` tipindedir ve çok sayıda kısıtlamayı destekler.
*   **`EItemFlag`**: Eşyanın genel özelliklerini tanımlayan bayraklar (örneğin, `ITEM_FLAG_REFINEABLE` - yükseltilebilir, `ITEM_FLAG_STACKABLE` - istiflenebilir, `ITEM_FLAG_RARE` - nadir, `ITEM_FLAG_LOG` - kullanımı loglanır).
*   **`EWearPositions`**: Eşyaların karakter üzerinde kuşanılacağı slotları tanımlar (WEAR_BODY, WEAR_WEAPON, WEAR_RING1 vb.).
*   **`EItemWearableFlag`**: Bir eşyanın hangi genel ekipman kategorisine (vücut, kask, silah vb.) takılabileceğini belirten bayraklar.
*   **`EApplyTypes`**: Eşyaların karakterlere sağlayabileceği bonus (efsun) türlerini tanımlar (APPLY_MAX_HP, APPLY_STR, APPLY_CRITICAL_PCT, APPLY_ATTBONUS_HUMAN vb.). Çok geniş ve çeşitli efsun türlerini içerir.

#### Ana Veri Yapısı: `TItemTable`

Bu yapı, `item_proto`'dan okunan ve bir eşyanın temel özelliklerini içeren tüm alanları barındırır:

*   `dwVnum`: Eşyanın benzersiz numarası (Virtual Number).
*   `szName` / `szLocaleName`: Eşyanın kod adı ve yerelleştirilmiş (oyunda görünen) adı.
*   `bType` / `bSubType`: Eşyanın `EItemType` ve ilgili alt tür enum'undan gelen tipi.
*   `bWeight` / `bSize`: Eşyanın ağırlığı ve envanterde kapladığı slot sayısı.
*   `ullAntiFlags` / `dwFlags` / `dwWearFlags` / `dwImmuneFlag`: Yukarıda bahsedilen çeşitli bayraklar.
*   `dwIBuyItemPrice` / `dwISellItemPrice`: NPC'den alınış ve NPC'ye satış fiyatı.
*   `aLimits[ITEM_LIMIT_MAX_NUM]`: `TItemLimit` (tip, değer) dizisi olarak eşya kullanım limitleri.
*   `aApplies[ITEM_APPLY_MAX_NUM]`: `TItemApply` (tip, değer) dizisi olarak eşyanın verdiği efsunlar.
*   `alValues[ITEM_VALUES_MAX_NUM]`: Eşya tipine göre farklı anlamlar taşıyan genel amaçlı 6 adet sayısal değer.
*   `alSockets[ITEM_SOCKET_MAX_NUM]`: Eşyadaki soketlere takılı olan taşların VNUM'ları veya özel değerleri.
*   `dwRefinedVnum`: Eşya yükseltildiğinde dönüşeceği yeni eşyanın VNUM'u (0 ise yükseltilemez veya son seviyedir).
*   `wRefineSet`: Yükseltme reçetesiyle ilgili bir set ID'si.
*   `bAlterToMagicItemPct`: Eşyanın sihirli bir eşyaya (genellikle bonusları olan) dönüşme olasılığı.
*   `bSpecular`: Modelin parlaklık katsayısı (0-255 arası).
*   `bGainSocketPct`: Eşyaya soket eklenme olasılığı (veya maksimum soket sayısı, yoruma göre değişebilir).

#### Önemli Metotlar ve İşlevler

*   **`CItemData::New()` ve `CItemData::Delete()`:** Dinamik havuzdan `CItemData` örneği almak ve iade etmek için kullanılır.
*   **`Clear()`:** Bir `CItemData` örneğinin içeriğini temizler, varsayılan değerlere ayarlar.
*   **`SetItemTableData(TItemTable* pItemTable)`:** Dışarıdan bir `TItemTable` verisi alarak eşyanın temel özelliklerini ayarlar. Bu, genellikle `CItemManager` tarafından `item_proto` yüklenirken çağrılır.
*   **`SetDefaultItemData(const char* c_szIconFileName, const char* c_szModelFileName)`:** Geçerli bir `item_proto` kaydı bulunamayan veya geçici olarak oluşturulan eşyalar için varsayılan bir görünüm (genellikle bir çuval modeli ve bir ikon) atar.
*   **`__LoadFiles()` (protected):** `m_strIconFileName`, `m_strModelFileName` gibi stringlerde tutulan dosya adlarını kullanarak `CResourceManager` aracılığıyla asıl grafik kaynaklarını (modeller, ikon) yükler ve ilgili `CGraphicThing*` veya `CGraphicSubImage*` işaretçilerine atar.
*   **`Get*()` Metotları:** Eşyanın VNUM'u, ismi, tipi, alt tipi, bayrakları, limitleri, efsunları, değerleri, soketleri, modelleri, ikonu gibi sayısız özelliğine erişim sağlar. Örneğin:
    *   `GetIndex()`: `dwVnum`'ı döndürür.
    *   `GetName()`: `szLocaleName`'i döndürür.
    *   `GetType()`, `GetSubType()`: Eşya tipini ve alt tipini döndürür.
    *   `IsAntiFlag(uint64_t ullFlag)`, `IsFlag(DWORD dwFlag)`: Belirli bayrakların kurulu olup olmadığını kontrol eder.
    *   `GetApply(BYTE byIndex, TItemApply* pItemApply)`: Belirli bir indeksteki efsunu döndürür.
    *   `GetValue(BYTE byIndex)`: Belirli bir indeksteki `alValues` değerini döndürür.
    *   `GetSocket(BYTE byIndex)`: Belirli bir indeksteki soket değerini döndürür.
    *   `GetModelThing()`, `GetDropModelThing()`, `GetIconImage()`: İlgili grafik kaynaklarına işaretçi döndürür.
*   **`IsEquipment()`:** Eşyanın giyilebilir bir ekipman olup olmadığını (silah, zırh, kostüm, yüzük, kemer) kontrol eder.
*   **Sisteme Özel `Get*`/`Set*` Metotları:** `ENABLE_ACCE_COSTUME_SYSTEM`, `ENABLE_AURA_COSTUME_SYSTEM` gibi derleme zamanı bayraklarına bağlı olarak, bu sistemlerle ilgili özel verileri (örneğin, Acce için farklı ırk/cinsiyetlere göre model ölçekleri, Aura için efekt ID'si) ayarlayan ve alan metotlar içerir.

#### Oyundaki Kullanım Amacı

`CItemData`, oyun istemcisinin eşyalarla ilgili tüm bilgilere tutarlı ve merkezi bir şekilde erişmesini sağlar. Envanterde bir eşya gösterileceği zaman ikonuna, karakter bir eşyayı kuşandığında 3D modeline, bir eşyanın üzerine gelindiğinde açıklamasına ve özelliklerine, bir eşya kullanıldığında ise türüne, alt türüne ve `alValues` gibi değerlerine bakılarak ilgili işlemler yapılır. `CItemManager` gibi bir üst yönetici sınıf, tüm `CItemData` örneklerini yönetir ve gerektiğinde VNUM ile sorgulanarak ilgili eşya verilerine ulaşılmasını sağlar. Bu yapı, yeni eşyaların eklenmesini, mevcutların dengelenmesini ve eşya tabanlı oyun mekaniklerinin yönetilmesini kolaylaştırır.