# GameLib Referans Kılavuzu - Bölüm 4

Bu belge, `GameLib` kütüphanesindeki çeşitli C++ sınıflarının ve ilgili başlık dosyalarının ayrıntılı bir referansını sunar. Her bileşen, genel amacını, başlık dosyasındaki (.h) önemli tanımları, C++ dosyasındaki (.cpp) temel implementasyon prensiplerini ve oyundaki pratik kullanım senaryolarını açıklayacak şekilde belgelenmiştir.

Bu dosya, `client_GameLib_Referans_Part3.md` dosyasının devamı niteliğindedir ve GameLib kütüphanesinin belgelendirilmesine buradan devam edilmektedir.

## İçindekiler

* [Dış Mekan Haritalarında Su Çizimi (`MapOutdoorWater.cpp`)](#dış-mekan-haritalarında-su-çizimi-mapoutdoorwatercpp)
* [Harita ve Ortam Tür Tanımları (`MapType.h` ve `MapType.cpp`)](#harita-ve-ortam-tür-tanımları-maptypeh-ve-maptypecpp)
* [Harita İle İlgili Yardımcı Fonksiyonlar ve Sınıflar (`MapUtil.h` ve `MapUtil.cpp`)](#harita-ile-ilgili-yardımcı-fonksiyonlar-ve-sınıflar-maputilh-ve-maputilcpp)
* [Canavar Alan Bilgileri (`MonsterAreaInfo.h` ve `MonsterAreaInfo.cpp`)](#canavar-alan-bilgileri-monsterareainfoh-ve-monsterareainfocpp)
* [Fiziksel Nesne Simülasyonu (`PhysicsObject.h` ve `PhysicsObject.cpp`)](#fiziksel-nesne-simülasyonu-physicsobjecth-ve-physicsobjectcpp)
* [Özellik (Property) Veri Yönetimi (`Property.h` ve `Property.cpp`)](#özellik-property-veri-yönetimi-propertyh-ve-propertycpp)

---

### Dış Mekan Haritalarında Su Çizimi (`MapOutdoorWater.cpp`)

#### Dosyanın Genel Amacı

`MapOutdoorWater.cpp`, `CMapOutdoor` sınıfının bir parçası olarak, dış mekan haritalarındaki su yüzeylerinin yüklenmesi, animasyonu ve çizilmesi (render edilmesi) ile ilgili işlevleri barındırır. Temel amacı, animasyonlu dokular ve basit dalgalanma efektleri kullanarak dinamik ve görsel olarak çekici su yüzeyleri oluşturmaktır.

#### C++ Implementasyon Detayları (`MapOutdoorWater.cpp`)

Bu dosya, `CMapOutdoor` sınıfına ait su ile ilgili metodları içerir:

*   **`CMapOutdoor::LoadWaterTexture()`**: Su animasyonu için kullanılacak doku serisini (`d:/ymir Work/special/water/%02d.dds`) yükler. Mevcut dokuları önce temizler, ardından 30 adet dokuyu `m_WaterInstances` dizisine `CResourceManager` aracılığıyla yükler.
*   **`CMapOutdoor::UnloadWaterTexture()`**: Yüklenmiş tüm su dokularını bellekten kaldırır.
*   **`CMapOutdoor::RenderWater()`**: Görünür arazi parçalarındaki (patches) su yüzeylerini çizen ana fonksiyondur.
    *   Render durumlarını ayarlar (Z-Buffer yazımı kapalı, Alpha Blending açık, Culling yok).
    *   Zamanlayıcı (`ELTimer_GetMSec()`) ile animasyonlu su dokusunu seçer ve ayarlar.
    *   Doku koordinatlarını kamera uzayına göre ayarlamak için bir doku dönüşüm matrisi kullanır, bu da suyun kamera hareketiyle akıyormuş gibi görünmesini sağlar.
    *   Basit bir su yüksekliği dalgalanma efekti uygular.
    *   Su yamalarını iki grup halinde çizer: sise yakın olanlar (alpha blend aktif) ve sisten uzak olanlar (alpha blend potansiyel olarak kapalı).
    *   Çizim sonrası render durumlarını geri yükler.
*   **`CMapOutdoor::DrawWater(long patchnum)`**: Belirli bir arazi yamasına ait su yüzeyini, o yamaya ait vertex buffer'ı kullanarak çizer.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Görsel Zenginlik**: Dış mekan haritalarına (nehirler, göller, deniz kıyıları) gerçekçi ve canlı su efektleri ekleyerek oyun dünyasının görsel kalitesini artırır.
*   **Dinamik Ortam**: Suyun animasyonlu dokuları ve hafif dalgalanma efekti, ortamın daha dinamik ve yaşayan bir dünya gibi hissedilmesine katkıda bulunur.
*   **Atmosfer Oluşturma**: Su yüzeylerinin görünümü, haritanın genel atmosferini (örneğin, sakin bir göl veya dalgalı bir deniz) destekleyebilir.
*   **Optimizasyon**: Su çizimi, sadece görünür ve su içeren arazi yamaları için yapılır. Uzaktaki su için alfa harmanlamanın kapatılması gibi teknikler, performansı optimize etmeye yönelik olabilir.

---

### Harita ve Ortam Tür Tanımları (`MapType.h` ve `MapType.cpp`)

#### Dosyaların Genel Amacı

`MapType.h` ve `MapType.cpp` dosyaları, Metin2 istemcisindeki haritalar, ortam özellikleri (ışıklandırma, sis vb.) ve harita üzerine yerleştirilebilen çeşitli "özellik" (property) nesneleri (ağaçlar, binalar, efektler) için temel veri yapılarını, sabitleri ve yardımcı fonksiyonları tanımlar. Bu tanımlar, harita verilerinin yüklenmesi, işlenmesi ve oyun dünyasının mantıksal ve görsel olarak oluşturulması için temel teşkil eder.

#### `MapType.h` - Başlık Dosyası Tanımları

*   **`prt` Namespace**: Harita özellikleri (property) ile ilgili tanımları içerir.
    *   `EPropertyType`: Ağaç, bina, efekt gibi özellik türlerini tanımlayan enum.
    *   `c_szPropertyTypeName`, `c_szPropertyExtension`: Özellik türü isimleri ve dosya uzantıları.
    *   Fonksiyonlar: `GetPropertyType`, `GetPropertyExtension`.
    *   Özellik Veri Yapıları: `TPropertyTree`, `TPropertyBuilding`, `TPropertyEffect`, `TPropertyAmbience`, `TPropertyDungeonBlock` gibi her özellik türü için spesifik veri tutan struct'lar.
    *   `EAmbiencePlayType`: Ortam seslerinin çalma türleri.
    *   Veri Dönüşüm Fonksiyon Prototipleri: `...DataToString`, `...StringToData` (CProperty ile entegrasyon için).
*   **Ortam Tanımları**:
    *   `SEnvironmentData` / `TEnvironmentData`: Bir haritanın tüm ortam ayarlarını (ışık, sis, gökyüzü vb.) içeren ana yapı.
    *   `TEnvironmentDataMap`: Farklı ortam ayar setlerini ID ile eşleştiren map.
*   **Konum Tanımları**: `TScreenPosition`, `TPixelPosition`, `TCellPosition`.
*   **Sabitler**: Harita bölümü, öznitelik ve hücre boyutları.

#### `MapType.cpp` - C++ Implementasyon Detayları

*   **`SEnvironmentData` Metodları**: `GetFogNearDistance`, `GetFogFarDistance`.
*   **`SPixelPosition_CalculateDistanceSq3d`**: İki 3D nokta arası mesafenin karesini hesaplar.
*   **`prt` Namespace Fonksiyonları**:
    *   `GetPropertyType`, `GetPropertyExtension` implementasyonları.
    *   `IntegerNumberToString`, `FloatNumberToString`.
    *   `...DataToString`, `...StringToData` implementasyonları: `TProperty...` yapılarını `CProperty` nesnesi aracılığıyla string tabanlı anahtar-değer formatına/formatından dönüştürür. Bu, özelliklerin metin dosyalarında saklanmasını sağlar.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Harita Editörü Veri Formatı**: `prt` namespace içindeki yapılar, harita editörlerinin (örn: WorldEditor) kullandığı veri formatını tanımlar. Tasarımcılar bu yapılar aracılığıyla dünyaya nesneler (ağaç, bina vb.) yerleştirir.
*   **Dinamik Ortam Yönetimi**: `SEnvironmentData`, haritaların atmosferik koşullarını (gündüz/gece, sis, gökyüzü) tanımlar. Oyun, farklı ortam setleri arasında geçiş yapabilir (örn: mağaraya girince değişen atmosfer).
*   **Optimizasyon ve Veri Erişimi**: Mesafe hesaplama fonksiyonları, hücre tabanlı konumlandırma gibi özellikler oyun mekaniklerini ve optimizasyonu destekler.
*   **Modülerlik**: Özellik sistemi, oyuna yeni nesne türlerinin eklenmesini kolaylaştırır.

---

### Harita İle İlgili Yardımcı Fonksiyonlar ve Sınıflar (`MapUtil.h` ve `MapUtil.cpp`)

#### Dosyaların Genel Amacı

`MapUtil.h` ve `MapUtil.cpp`, harita sistemiyle ilgili çeşitli yardımcı fonksiyonları (ortam verisi yükleme, koordinat dönüşümleri, mesafe hesaplamaları) ve yumuşak geçiş animasyonları için kullanılan `CEaseOutInterpolation` sınıfını barındırır. Bu araçlar, oyun dünyasının oluşturulması ve dinamiklerinin yönetilmesinde önemli rol oynar.

#### `MapUtil.h` - Başlık Dosyası Tanımları

*   **Ortam Verisi Fonksiyonları**:
    *   `Environment_Init(SEnvironmentData& envData)`: `SEnvironmentData` yapısını varsayılan değerlerle başlatır.
    *   `Environment_Load(SEnvironmentData& envData, const char* envFileName)`: `.env` dosyasından ortam verilerini yükler.
*   **Koordinat ve Mesafe Fonksiyonları**:
    *   `GetLinearInterpolation(float begin, float end, float curRate)`: Doğrusal enterpolasyon yapar.
    *   `PixelPositionToAttributeCellPosition(...)`, `AttributeCellPositionToPixelPosition(...)`: Piksel ve harita öznitelik hücre koordinatları arasında dönüşüm yapar.
    *   `GetPixelPositionDistance(...)`: İki piksel pozisyonu arası mesafeyi hesaplar.
*   **`CEaseOutInterpolation` Sınıfı**: Yavaşlayarak duran (ease-out) enterpolasyon efekti için.
    *   `Initialize()`: Değerleri sıfırlar.
    *   `Setup(float fStart, float fEnd, float fTime)`: Enterpolasyonu ayarlar.
    *   `Interpolate(float fElapsedTime)`: Değeri günceller.
    *   `isPlaying()`: Aktif olup olmadığını kontrol eder.
    *   `GetValue()`, `GetChangingValue()`: Mevcut değeri ve değişim miktarını döndürür.

#### `MapUtil.cpp` - C++ Implementasyon Detayları

*   **`Environment_Init`**: `SEnvironmentData` yapısının tüm alanlarını (ışıklar, materyal, sis, filtreleme, rüzgar, gökyüzü, bulutlar, mercek parlaması) varsayılan değerlere ayarlar.
*   **`Environment_Load`**: `CTextFileLoader` kullanarak `.env` dosyasını okur. Dosyadaki "directionallight", "material", "fog", "filter", "skybox", "lensflare" gibi bölümleri ayrıştırarak `SEnvironmentData` yapısını doldurur.
*   **Koordinat ve Mesafe Fonksiyonları**: Bildirimlerine uygun standart matematiksel işlemleri gerçekleştirir.
*   **`CEaseOutInterpolation`**:
    *   `Setup`: Başlangıç/bitiş değerleri ve süreye göre hız ve ivme hesaplar.
    *   `Interpolate`: Kalan süreyi, hızı ve değeri günceller. Bittiğinde değeri sıfırlar (dikkat: bitiş değerinde kalmaz).

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Ortam Yönetimi**: `.env` dosyaları aracılığıyla her haritanın kendine özgü atmosferini (ışık, sis, gökyüzü) yükler ve yönetir.
*   **Koordinat Sistemleri**: Oyun mantığı (hücre tabanlı) ve grafik çizimi (piksel tabanlı) arasında koordinat dönüşümleri yapar.
*   **Mesafe Hesaplamaları**: AI, yetenek menzili, oyuncu etkileşimleri gibi oyun mekaniklerinde kullanılır.
*   **Yumuşak Animasyonlar**: `CEaseOutInterpolation`, UI animasyonları, kamera hareketleri, değer geçişleri (örn: can barı azalması) için yumuşak "ease-out" efektleri sağlar.

---

### Canavar Alan Bilgileri (`MonsterAreaInfo.h` ve `MonsterAreaInfo.cpp`)

#### Dosyaların Genel Amacı

`MonsterAreaInfo.h` ve `MonsterAreaInfo.cpp`, harita üzerinde canavarların veya canavar gruplarının nerede, nasıl ve hangi sayıda ortaya çıkacağını (spawn) belirleyen alan bilgilerini yöneten `CMonsterAreaInfo` sınıfını tanımlar. Bu, genellikle oyunun dünya editörü tarafından oluşturulan ve harita verileriyle birlikte saklanan bir yapıdır.

#### `MonsterAreaInfo.h` - Başlık Dosyası Tanımları

*   **Enum Tanımları**:
    *   `EMonsterAreaInfoType`: Alanın türünü belirtir (`MONSTER`, `GROUP`).
    *   `EMonsterDir`: Canavarların başlangıç yönünü belirtir (Kuzey, Güney, Rastgele vb.).
*   **`CMonsterAreaInfo` Sınıfı**:
    *   Temel Alan Fonksiyonları: `SetID`/`GetID`, `Clear`, `SetOrigin`/`GetOrigin`, `SetSize`/`GetSize`, `GetLeft`/`Top`/`Right`/`Bottom`.
    *   Alan Türü Fonksiyonları: `SetMonsterAreaInfoType`/`GetMonsterAreaInfoType`.
    *   Canavar Grubu Bilgileri: `SetMonsterGroupID`/`GetMonsterGroupID`, İsim, Lider Adı, Takipçi Sayısı.
    *   Tekil Canavar Bilgileri: `SetMonsterName`/`GetMonsterName`, `SetMonsterVID`/`GetMonsterVID`.
    *   Genel Canavar Ayarları: `SetMonsterCount` (rastgele pozisyonlar üretir), `SetMonsterDirection` (yön vektörü hesaplar), `RemoveAllMonsters`, `GetMonsterCount`, `GetMonsterDir`, `GetMonsterDirVector`, `GetTempMonsterPos`.
    *   Korumalı Fonksiyon: `SetLRTB` (alan sınırlarını hesaplar).
    *   Üye Değişkenler: Alan türü, grup/canavar bilgileri, sayı, yön, ID, geometri, geçici canavar pozisyonları (`m_TempMonsterPosVector`).
*   **Typedef Tanımları**: `TMonsterAreaInfoPtrVector`, `TMonsterAreaInfoPtrVectorIterator`.

#### `MonsterAreaInfo.cpp` - C++ Implementasyon Detayları

*   **`CMonsterAreaInfo::Clear()`**: Alanın başlangıç noktasını ve boyutunu geçersiz değerlere ayarlar, `RemoveAllMonsters()` çağırır.
*   **`SetOrigin`, `SetSize`**: Değerleri ayarlar ve alan sınırlarını güncellemek için `SetLRTB()` çağırır.
*   **`SetLRTB()`**: Alanın sol, üst, sağ, alt sınırlarını merkez ve yarı boyutlardan hesaplar.
*   **`SetMonsterCount(DWORD dwCount)`**: Canavar sayısını ayarlar, `m_TempMonsterPosVector`'ü temizler, yeniden boyutlandırır ve alan sınırları içinde rastgele pozisyonlar atar.
*   **`SetMonsterDirection(EMonsterDir eMonsterDir)`**: Yönü ayarlar ve `EMonsterDir`'e göre bir 2D yön vektörü (`m_v2Monsterdirection`) hesaplar (rotasyon matrisi kullanarak). `DIR_RANDOM` ise rastgele bir yön seçer.
*   **`RemoveAllMonsters()`**: Alan türünü `INVALID` yapar, ID'leri/isimleri sıfırlar/boşaltır, canavar sayısını 0 yapar, yönü kuzeye ayarlar.
*   **`GetTempMonsterPos(DWORD dwIndex)`**: Geçerli indeksteki önceden hesaplanmış canavar pozisyonunu döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Harita Editörü Entegrasyonu**: Tasarımcılar, dünya editörüyle haritalara `CMonsterAreaInfo` nesneleri yerleştirir, türünü (tekil/grup), canavar ID'sini, sayısını, yönünü ayarlar. Bu bilgiler harita dosyasıyla kaydedilir.
*   **Canavar Yaratma (Spawning)**: Oyun sunucusu, haritayı yüklerken bu bilgileri okur. Alan aktif olduğunda (oyuncu yaklaşınca vb.), belirtilen sayıda ve türde canavarı, hesaplanan rastgele pozisyonlarda ve yönlerde yaratır.
*   **Görev ve Etkinlik Yönetimi**: Görevler, belirli `CMonsterAreaInfo` bölgelerindeki canavarların öldürülmesini gerektirebilir. Etkinlikler özel canavar spawn bölgelerini aktif edebilir.
*   **Performans**: Canavarları belirli alanlara sınırlamak, kontrollü ve performanslı bir spawn mekanizması sunar. Pozisyonların önceden hesaplanması (editörde/yüklemede) spawn anındaki yükü azaltır.

---

### Fiziksel Nesne Simülasyonu (`PhysicsObject.h` ve `PhysicsObject.cpp`)

#### Dosyaların Genel Amacı

`PhysicsObject.h` ve `PhysicsObject.cpp`, oyun dünyasındaki nesneler için basit bir itme (push) ve sürtünme (friction) tabanlı fiziksel etki simülasyonu sağlayan `CPhysicsObject` sınıfını tanımlar. Ayrıca, bu simülasyonun dünya (çarpışma tespiti için `IPhysicsWorld`) ve diğer nesnelerle (çarpışma ayarı için `IObjectManager`) etkileşimini yönetmek için kullanılan arayüzleri içerir. Temel amaç, karakterlere veya nesnelere dış kuvvetler uygulandığında kısa süreli bir hareket ve yavaşlayarak durma efekti yaratmaktır.

#### `PhysicsObject.h` - Başlık Dosyası Tanımları

*   **`IPhysicsWorld` Arayüzü**: Fiziksel dünyanın çarpışma tespitini soyutlar.
    *   `static IPhysicsWorld* GetPhysicsWorld()`: Global erişim için.
    *   `virtual bool isPhysicalCollision(const D3DXVECTOR3& c_rvCheckPosition) = 0`: Çarpışma kontrolü (saf sanal).
*   **`IObjectManager` Arayüzü**: Nesnelerin diğer nesnelerle çarpışmalarını ayarlamayı soyutlar.
    *   `static IObjectManager* GetObjectManager()`: Global erişim için.
    *   `virtual void AdjustCollisionWithOtherObjects(CActorInstance* pInst) = 0`: Çarpışma ayarı (saf sanal).
*   **`CPhysicsObject` Sınıfı**:
    *   `Initialize()`: Kütle, sürtünme, hız, ivme gibi fiziksel özellikleri sıfırlar.
    *   `Update(float fElapsedTime)`: Her frame çağrılır, itme enterpolasyonlarını günceller.
    *   `isBlending()`: İtme etkisinin devam edip etmediğini kontrol eder.
    *   `SetDirection(...)`: Hareket yönünü ayarlar.
    *   `IncreaseExternalForce(const D3DXVECTOR3& c_rvBasePosition, float fForce)`: Dış kuvvet uygular, çarpışma kontrolü yaparak son pozisyonu hesaplar.
    *   `SetLastPosition(const TPixelPosition& c_rPosition, float fBlendingTime)`: İtme sonucu oluşan son pozisyon farkını ve süresini ayarlar, yumuşak geçiş için `CEaseOutInterpolation`'ları kurar.
    *   `GetLastPosition(...)`: Hesaplanan son itme ötelenmesini döndürür.
    *   `GetXMovement()`, `GetYMovement()`: Anlık X/Y hareketini döndürür.
    *   `SetActorInstance(...)`/`GetActorInstance(...)`: İlişkili `CActorInstance`'ı yönetir.
    *   Korumalı: `Accumulate(...)` (fizik adımı hesaplar), üye değişkenler (kütle, sürtünme, vektörler, `CEaseOutInterpolation` nesneleri).

#### `PhysicsObject.cpp` - C++ Implementasyon Detayları (Özet)

*   **Global Sabitler**: `c_fFrameTime` (simülasyon adım süresi), `EPSILON` (küçük değer eşiği).
*   **`CPhysicsObject::Update`**: Aktifse `m_xPushingPosition` ve `m_yPushingPosition` enterpolasyonlarını günceller.
*   **`CPhysicsObject::Accumulate`**: Bir zaman adımı için temel fizik hesaplaması yapar (sürtünme uygular, ivme ve hızı günceller, pozisyonu değiştirir). Hız ve yön zıt ise hızı sıfırlar.
*   **`CPhysicsObject::IncreaseExternalForce`**:
    *   Başlangıç ivmesi ve hızı kuvvetten hesaplar.
    *   Bir döngü içinde `Accumulate` ile hareketi simüle eder, her adımda `IPhysicsWorld::isPhysicalCollision` ile çarpışma kontrolü yapar.
    *   Çarpışma olursa fizik nesnesini sıfırlar.
    *   Nesne durursa döngüden çıkar.
    *   Sonuçta oluşan toplam ötelenme ve süre ile `SetLastPosition` çağrılır.
    *   Varsa, `IObjectManager::AdjustCollisionWithOtherObjects` çağrılır.
*   **`CPhysicsObject::SetLastPosition`**: `m_v3LastPosition`'ı ayarlar, X ve Y eksenleri için `CEaseOutInterpolation`'ları (başlangıç 0, hedef ötelenme) kurar.
*   Diğer getter/setter ve `Initialize` fonksiyonları bildirimlere uygun şekilde çalışır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Karakter Geri Sekmesi (Knockback)**: Güçlü saldırılar aldığında karakterlerin (oyuncu veya NPC) itilmesi. Saldırıyı yapan, hedefin `CPhysicsObject`'ine yön ve kuvvet uygular. Sistem, engellere çarpana kadar veya kuvvet sönümlenene kadar hedefi iter ve `CEaseOutInterpolation` ile hareketi yumuşatır.
*   **Ortam Etkileşimi**: Parçalanabilir nesnelerin (variller, sandıklar) bir patlama sonucu etrafa saçılması.
*   **Arayüzler (`IPhysicsWorld`, `IObjectManager`)**: `GameLib`'in ana oyun mantığından (çarpışma sistemi, nesne yönetimi) bağımsız kalmasını sağlar. Oyunun üst katmanları bu arayüzleri implemente ederek fizik sistemine gerekli bilgileri sağlar.
*   **Hareket Yumuşatma**: `CEaseOutInterpolation` kullanımı, itme efektlerinin ani ve keskin olması yerine daha pürüzsüz ve görsel olarak hoş görünmesini sağlar.

---

### Özellik (Property) Veri Yönetimi (`Property.h` ve `Property.cpp`)

#### Dosyaların Genel Amacı

`Property.h` ve `Property.cpp`, oyun içinde çeşitli nesne ve yapılandırma özelliklerini (property) metin tabanlı dosyalarda (genellikle `.pr`, `.prt`, `.prb` vb. uzantılı) saklamak, yüklemek ve yönetmek için kullanılan `CProperty` sınıfını tanımlar. Bu sistem, anahtar-değer çiftleri ve bu anahtarlara bağlı string listeleri (token vektörleri) temelinde çalışır. Dosyalar `YPRT` (Ymir Property) adı verilen bir başlık ve CRC kontrolü içerir.

#### `Property.h` - Başlık Dosyası Tanımları

*   **`CProperty(const char* c_pszFileName)`**: Yapıcı. Dosya adını alır.
*   **`Clear()`**: Tüm property'leri temizler.
*   **`ReadFromMemory(const void* c_pvData, int iLen, const char* c_pszFileName)`**: Bellekteki veriden property'leri okur. Formatı (fourCC, CRC) kontrol eder, `CMemoryTextFileLoader` ile satırları ayrıştırır.
*   **`GetFileName()`**: Dosya yolunu döndürür.
*   **`GetVector(const char* c_pszKey, CTokenVector& rTokenVector)`**: Anahtara bağlı string vektörünü alır.
*   **`GetString(const char* c_pszKey, const char** c_ppString)`**: Anahtara bağlı ilk string'i alır.
*   **`PutVector(const char* c_pszKey, const CTokenVector& c_rTokenVector)`**: Anahtar-vektör çifti ekler.
*   **`PutString(const char* c_pszKey, const char* c_pszString)`**: Anahtar-tek string çifti ekler.
*   **`Save(const char* c_pszFileName)`**: Property verilerini dosyaya kaydeder. `YPRT` başlığı, CRC ve anahtar-değer çiftlerini yazar. `CPropertyManager` aracılığıyla güvenli kaydetme yapar.
*   **`GetSize()`**: Property sayısını döndürür.
*   **`GetCRC()`**: Dosyanın CRC'sini döndürür.
*   **Korumalı Üyeler**: Dosya adı, CRC, token map (`m_stTokenMap`).

#### `Property.cpp` - C++ Implementasyon Detayları (Özet)

*   **Dosya Formatı**: `YPRT` (4 byte), `\r\n`, CRC (string), `\r\n`, ardından her satır: `anahtar\t"değer1"\t"değer2"...\r\n`.
*   **Okuma (`ReadFromMemory`)**: FourCC ve CRC'yi doğrular, satırları ayrıştırır. Her satırda ilk token anahtar, kalanlar değer vektörü olarak `m_stTokenMap`'e eklenir. Anahtarlar küçük harfe çevrilir.
*   **Yazma (`Save`)**: `CTempFile` kullanır. `YPRT` başlığını, hesaplanmış/mevcut CRC'yi yazar. Map'teki her anahtar-değer çiftini formatına uygun şekilde yazar. `CPropertyManager::Instance().Put()` ile asıl dosyaya taşır.
*   **CRC Yönetimi**: `Save` sırasında CRC yoksa, zaman damgası ve dosya adından `CPropertyManager` ile benzersiz bir CRC oluşturur.
*   Getter/setter'lar map üzerinde çalışır, anahtarları genellikle küçük harfe çevirerek işlem yapar.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Nesne Özellikleri**: `MapType.h`'deki `TPropertyTree`, `TPropertyBuilding` gibi yapılarla tanımlanan ağaç, bina vb. nesnelerin özelliklerini (model, boyut) `.prt`, `.prb` dosyalarında saklar.
*   **Efekt Tanımları (`.pre`)**: Özel efektlerin partikül, ses gibi detaylarını tanımlar.
*   **Ortam Sesi Tanımları (`.pra`)**: Bölgesel ortam seslerini, çalma türlerini ve dosyalarını yönetir.
*   **Zindan Blokları (`.prd`)**: Modüler zindan parçalarının özelliklerini saklar.
*   **Harita Editörü Entegrasyonu**: WorldEditor gibi araçlar bu dosyaları okur ve yazar, tasarımcıların oyun dünyasını ve nesnelerini yapılandırmasını sağlar.
*   **Veri Bütünlüğü**: CRC, dosyaların bozulmadığını veya beklenmedik şekilde değişmediğini kontrol etmeye yardımcı olur.
*   **Esnek Yapılandırma**: Anahtar-değer sistemi, çeşitli varlıklar için esnek ve genişletilebilir bir veri tanımlama yöntemi sunar.

---

### Özellik (Property) Dosyalarını Yükleme (`PropertyLoader.h` ve `PropertyLoader.cpp`)

#### Dosyaların Genel Amacı

`PropertyLoader.h` ve `PropertyLoader.cpp` dosyaları, `CDir` sınıfından türeyen `CPropertyLoader` sınıfını tanımlar. Bu sınıfın temel amacı, belirtilen bir dizin yapısı içindeki özellik (property) dosyalarını (genellikle `.pr`, `.prt`, `.prb` gibi uzantılara sahip) yinelemeli olarak taramak ve bu dosyaları `CPropertyManager` aracılığıyla sisteme kaydetmektir. Bu sayede oyun, başlangıçta veya ihtiyaç duyulduğunda ilgili tüm property verilerini yükleyebilir.

#### `PropertyLoader.h` - Başlık Dosyası Tanımları

*   **`#include "../eterbase/FileDir.h"`**: Dizin ve dosya işlemleri için temel sınıf olan `CDir`'ı dahil eder.
*   **`class CPropertyManager;`**: `CPropertyManager` sınıfının ileriye dönük bildirimi (forward declaration).
*   **`class CPropertyLoader : public CDir`**: `CDir`'dan kalıtım alır.
    *   **`CPropertyLoader()`**: Yapıcı metot.
    *   **`virtual ~CPropertyLoader()`**: Yıkıcı metot.
    *   **`void SetPropertyManager(CPropertyManager* pPropertyManager)`**: Kullanılacak `CPropertyManager` nesnesini ayarlar.
    *   **`DWORD RegisterFile(const char* c_szPathName, const char* c_szFileName)`**: Belirtilen dosya yolunu ve adını kullanarak bir property dosyasını `CPropertyManager`'a kaydeder. Dosyanın CRC'sini döndürür.
    *   **`virtual bool OnFolder(const char* c_szFilter, const char* c_szPathName, const char* c_szFileName)`**: `CDir` sınıfından gelen ve bir klasör bulunduğunda çağrılan sanal metot. Bu klasör içinde yeni bir `CPropertyLoader` örneği oluşturarak yinelemeli tarama yapar.
    *   **`virtual bool OnFile(const char* c_szPathName, const char* c_szFileName)`**: `CDir` sınıfından gelen ve bir dosya bulunduğunda çağrılan sanal metot. `RegisterFile` metodunu çağırarak dosyayı kaydeder.
    *   **`protected: CPropertyManager* m_pPropertyManager;`**: Property dosyalarını yönetmek için kullanılacak `CPropertyManager` işaretçisi.

#### `PropertyLoader.cpp` - C++ Implementasyon Detayları

*   **`CPropertyLoader::OnFolder(...)`**:
    *   Verilen filtre, yol ve klasör adını kullanarak tam bir alt klasör yolu oluşturur.
    *   Yeni bir `CPropertyLoader` nesnesi oluşturur.
    *   Bu yeni yükleyiciye mevcut `m_pPropertyManager`'ı atar.
    *   Yeni yükleyicinin `Create` metodunu (CDir sınıfından gelir) çağırarak alt klasördeki property dosyalarının taranmasını başlatır.
*   **`CPropertyLoader::OnFile(...)`**:
    *   Bulunan dosya için `RegisterFile` metodunu çağırır.
*   **`CPropertyLoader::RegisterFile(...)`**:
    *   Tam dosya yolunu oluşturur.
    *   Dosya uzantısını alır ve hem uzantıyı hem de dosya adını küçük harfe çevirir.
    *   Dosya yolundaki ters taksimleri (`\`) normal taksimlere (`/`) dönüştürür (`StringPath`).
    *   Eğer dosya adı özel `"property/reserve"` ise, bu dosya `m_pPropertyManager->LoadReservedCRC()` çağrılarak işlenir ve ayrılmış CRC değerlerini yükler.
    *   Diğer tüm durumlar için, `m_pPropertyManager->Register()` çağrılarak property dosyası kaydedilir ve başarılı olursa ilgili `CProperty` nesnesinin CRC'si döndürülür, aksi halde 0 döner.
*   **`CPropertyLoader::SetPropertyManager(...)`**: Verilen `CPropertyManager` işaretçisini `m_pPropertyManager`'a atar.
*   **Yapıcı ve Yıkıcı Metotlar**: Yapıcıda `m_pPropertyManager`'ı `NULL` olarak başlatır. Yıkıcıda özel bir işlem yapılmaz.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Toplu Property Yükleme**: Oyunun başlangıcında veya belirli bir modül yüklenirken (örneğin, yeni bir harita paketi), ilgili klasörlerdeki tüm property dosyalarının (`.prt`, `.prb`, `.pre` vb.) otomatik olarak taranıp `CPropertyManager` aracılığıyla sisteme kaydedilmesini sağlar. Bu, tek tek her dosyayı manuel olarak yükleme ihtiyacını ortadan kaldırır.
*   **Modülerlik ve Esneklik**: Yeni property dosyaları eklendiğinde veya mevcutlar güncellendiğinde, sadece ilgili klasöre yerleştirilmeleri yeterlidir. `CPropertyLoader` bu dosyaları otomatik olarak bulup yükleyecektir.
*   **Veri Organizasyonu**: Oyun verilerinin (özellikle nesne, efekt, ortam özellikleri) düzenli bir klasör yapısında tutulmasına ve yönetilmesine olanak tanır.
*   **Ön Yükleme (Preloading)**: Oyunun ihtiyaç duyacağı property verilerini önceden yükleyerek çalışma zamanındaki erişim hızını artırabilir.
*   **CRC Yönetimi Entegrasyonu**: `RegisterFile` içinde `CPropertyManager::Register` çağrısı, property dosyalarının CRC'lerini de kontrol ederek veri bütünlüğünün sağlanmasına yardımcı olur. Özel `"property/reserve"` dosyası, `LoadReservedCRC` fonksiyonu aracılığıyla, manuel olarak ayarlanmış veya önceden bilinen CRC değerlerini yüklemek için bir mekanizma sunar.

---

### Özellik (Property) Yönetimi (`PropertyManager.h` ve `PropertyManager.cpp`)

#### Dosyaların Genel Amacı

`PropertyManager.h` ve `PropertyManager.cpp` dosyaları, `CSingleton` desenini kullanarak gerçeklenen `CPropertyManager` sınıfını tanımlar. Bu sınıf, oyundaki tüm özellik (property) dosyalarının merkezi yönetimini üstlenir. Property dosyalarını bir paket dosyasından (`.epk/.eix`) veya doğrudan dosya sisteminden yükleyebilir, CRC (Cyclic Redundancy Check) değerlerine göre saklayabilir, erişim sağlayabilir ve yönetebilir. Temel amacı, property verilerine hızlı ve verimli bir şekilde erişilmesini sağlamak ve veri bütünlüğünü korumaktır.

#### `PropertyManager.h` - Başlık Dosyası Tanımları

*   **`#include "../eterPack/EterPack.h"`**: `CEterPack` ve ilgili paketleme sistemi bileşenlerini dahil eder.
*   **`class CPropertyManager : public CSingleton<CPropertyManager>`**: `CSingleton` şablon sınıfından türetilerek tekil (singleton) bir nesne olmasını sağlar.
    *   **`CPropertyManager()` / `virtual ~CPropertyManager()`**: Yapıcı ve yıkıcı metotlar.
    *   **`void Clear()`**: Yüklenmiş tüm property'leri temizler.
    *   **`void SetPack(CEterPack* pPack)`**: Dışarıdan bir `CEterPack` nesnesi atamak için (kullanılıp kullanılmadığı implementasyona bağlıdır, mevcut cpp'de doğrudan kullanılmıyor gibi görünüyor).
    *   **`bool BuildPack()`**: "property" klasöründeki dosyaları bir paket dosyasına (`property.epk`/`eix`) yazar.
    *   **`bool LoadReservedCRC(const char* c_pszFileName)`**: Belirtilen dosyadan (genellikle `property/reserve`) ayrılmış CRC listesini yükler.
    *   **`void ReserveCRC(DWORD dwCRC)`**: Verilen bir CRC değerini ayrılmış listeye ekler. Bu CRC'ler `GetUniqueCRC` tarafından üretilmez.
    *   **`DWORD GetUniqueCRC(const char* c_szSeed)`**: Verilen bir Tohum (seed) string'e göre benzersiz bir CRC üretir. Üretilen CRC, ayrılmış CRC'ler veya daha önce kaydedilmiş property CRC'leri ile çakışmaz.
    *   **`bool Initialize(const char* c_pszPackFileName = NULL)`**: Property yöneticisini başlatır. Eğer bir paket dosya adı verilirse, property'leri bu paketten yükler (`m_isFileMode = false`). Verilmezse, dosya modunda çalışır (`m_isFileMode = true`) ve dosyalar doğrudan diskten okunur.
    *   **`bool Register(const char* c_pszFileName, CProperty** ppProperty = NULL)`**: Bir property dosyasını okur, `CProperty` nesnesi oluşturur ve CRC'sine göre haritada (map) saklar. Eğer `ppProperty` verilirse, oluşturulan nesnenin işaretçisini döndürür.
    *   **`bool Get(DWORD dwCRC, CProperty** ppProperty)`**: Verilen CRC'ye sahip `CProperty` nesnesini bulur ve `ppProperty` aracılığıyla döndürür.
    *   **`bool Get(const char* c_pszFileName, CProperty** ppProperty)`**: Dosya adına göre `CProperty` nesnesini alır (aslında `Register` işlemini yapar).
    *   **`bool Put(const char* c_pszFileName, const char* c_pszSourceFileName)`**: Bir kaynak dosyayı hedef ada kopyalar, paket modundaysa pakete ekler ve ardından property'yi kaydeder.
    *   **`bool Erase(DWORD dwCRC)`**: CRC'ye göre bir property'yi siler. Dosyayı diskten ve (paket modundaysa) paketten siler, CRC'sini `property/reserve` dosyasına ekler.
    *   **`bool Erase(const char* c_pszFileName)`**: Dosya adına göre bir property'yi siler.
    *   **Korunan Üyeler**:
        *   `TPropertyCRCMap`: DWORD (CRC) anahtarlı `CProperty*` değerleri tutan map.
        *   `TCRCSet`: Ayrılmış CRC değerlerini tutan set.
        *   `m_isFileMode`: Paket modunda mı yoksa dosya modunda mı çalışıldığını belirten bayrak.
        *   `m_PropertyByCRCMap`: Yüklenmiş property'leri CRC'lerine göre saklayan map.
        *   `m_ReservedCRCSet`: Ayrılmış CRC'leri saklayan set.
        *   `m_pack`: Property paketini yöneten `CEterPack` nesnesi.
        *   `m_fileDict`: Paket dosyası için kullanılan `CEterFileDict`.

#### `PropertyManager.cpp` - C++ Implementasyon Detayları (Özet)

*   **`Initialize`**: Paket adı verilirse, `m_pack.Create` ile paketi açar. Paketteki her dosyayı (eğer `property/reserve` değilse) `Register` ile kaydeder. Paket adı verilmezse, dosya modunda kalır (WorldEditor gibi araçlar için).
*   **`BuildPack`**: "property" adında bir paket oluşturur. "property" klasöründeki tüm dosyaları bu pakete sıkıştırmadan (`COMPRESSED_TYPE_NONE`) ekler.
*   **`LoadReservedCRC`**: `CEterPackManager` aracılığıyla `property/reserve` dosyasını okur. Dosyadaki her satırı bir CRC değeri olarak `ReserveCRC` ile kaydeder.
*   **`ReserveCRC`**: Verilen CRC'yi `m_ReservedCRCSet`'e ekler.
*   **`GetUniqueCRC`**: Verilen seed ile CRC32 hesaplar. Bu CRC, `m_ReservedCRCSet` veya `m_PropertyByCRCMap`'te varsa, seed'e rastgele bir rakam ekleyerek yeni bir CRC hesaplar ve bu işlemi benzersiz bir CRC bulunana kadar tekrarlar.
*   **`Register`**: `CEterPackManager` kullanarak dosyayı bellek haritalı dosya (mapped file) olarak alır. Yeni bir `CProperty` nesnesi oluşturur ve `pProperty->ReadFromMemory` ile içeriğini okur. Eğer aynı CRC ile başka bir property zaten kayıtlıysa, eskisini silip yenisiyle değiştirir ve bunu loglar. Yoksa, yeni property'yi `m_PropertyByCRCMap`'e ekler.
*   **`Get(const char* ...)`**: Aslında dosyayı yeniden `Register` eder.
*   **`Get(DWORD ...)`**: `m_PropertyByCRCMap`'ten CRC'ye göre property'yi bulur.
*   **`Put`**: Dosyayı belirtilen hedefe kopyalar. Eğer paket modundaysa, dosyayı pakete de ekler (`m_pack.Put`). Son olarak dosyayı `Register` eder.
*   **`Erase(DWORD dwCRC)`**: Property'yi map'ten siler, `DeleteFile` ile dosyayı diskten siler. CRC'yi `ReserveCRC` ile ayrılmış listeye alır. Paket modundaysa paketten de siler (`m_pack.Delete`). Silinen property'nin CRC'sini `property/reserve` dosyasına ekler (append).
*   **`Clear`**: `m_PropertyByCRCMap`'teki tüm `CProperty` nesnelerini siler ve map'i temizler.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Merkezi Property Erişimi**: Oyunun herhangi bir yerinden, dosya adı veya CRC ile property verilerine erişim sağlar. Bu, özellikle `CPropertyLoader` tarafından toplu yüklenen property'lere erişmek için kullanılır.
*   **Paket Yönetimi (`.epk/.eix`)**: Property dosyalarını bir paket içinde toplayarak dağıtımı kolaylaştırır ve dosya sayısını azaltır. `Initialize` sırasında bu paketten yükleme yapabilir. `BuildPack` ile bu paket dosyası oluşturulabilir.
*   **Dosya Modu**: Özellikle oyun geliştirme araçları (örn: WorldEditor) için, property dosyalarının doğrudan diskten okunup yazılmasına olanak tanır, böylece değişiklikler anında yansır.
*   **CRC Tabanlı Veri Bütünlüğü ve Yönetimi**: Her property dosyası bir CRC ile tanımlanır. Bu, aynı içeriğe sahip farklı isimli dosyaların veya güncellemelerin yönetilmesine yardımcı olabilir. `GetUniqueCRC`, yeni property'ler için çakışmayan CRC'ler üretir.
*   **Ayrılmış CRC'ler (`property/reserve`)**: Belirli CRC değerlerinin özel amaçlar için ayrılmasını ve otomatik üretilen CRC'lerle çakışmamasını sağlar. Örneğin, silinmiş dosyaların CRC'leri buraya eklenerek tekrar kullanılmaları engellenebilir.
*   **Dinamik Yükleme ve Kayıt**: `Register` ve `Put` fonksiyonları, oyun çalışırken bile yeni property dosyalarının yüklenmesine veya mevcutların güncellenmesine olanak tanır.
*   **Bellek Yönetimi**: `Clear` fonksiyonu ile yüklenmiş tüm property nesnelerinin bellekteki referanslarını temizler.

---

### Irk Veri Yönetimi (`RaceData.h` ve `RaceData.cpp`)

#### Dosyaların Genel Amacı

`RaceData.h` ve `RaceData.cpp` dosyaları, oyundaki karakter ırklarına (örneğin Savaşçı, Ninja, Sura, Şaman) ait tüm görsel, yapılandırma ve davranışsal verileri yöneten `CRaceData` sınıfını tanımlar. Bu sınıf, bir ırkın temel modelini, animasyonlarını (motions), saç stillerini, vücut şekillerini, eklenti parçalarını (attaching data), çarpışma bilgilerini, duman efektlerini ve saldırı kombolarını içerir. Veriler genellikle harici dosyalardan yüklenir ve `CRaceData` nesnesi içinde çeşitli veri yapılarında saklanır. Sınıf ayrıca, bu verilere erişmek ve bunları yönetmek için kapsamlı bir arayüz sunar. `CDynamicPool` kullanarak bellek yönetimi optimize edilmiştir.

#### `RaceData.h` - Başlık Dosyası Tanımları (Özet)

*   **Enum Tanımları**:
    *   `ERaces`: Oyundaki farklı karakter ırklarını listeler (örn: `RACE_WARRIOR_M`, `RACE_ASSASSIN_W`).
    *   `EParts`: Karakterin farklı vücut veya ekipman parçalarını tanımlar (örn: `PART_MAIN`, `PART_WEAPON`, `PART_HAIR`).
    *   `SMOKE_NUM`: Duman efekti sayısı için bir sabit.
*   **Temel Veri Yapıları ve Typedef'ler**:
    *   `SMotion`: Bir animasyon verisini, yüzdesini ve ilişkili `CGraphicThing` (model) ile `CRaceMotionData` (animasyon detayları) işaretçilerini tutar.
    *   `TMotionVector`: `SMotion` nesnelerinden oluşan bir vektör.
    *   `TMotionVectorMap`: Animasyon indeksine göre `TMotionVector` tutan bir harita.
    *   `SMotionModeData`: Belirli bir animasyon moduna (örn: ayakta durma, yürüme, koşma, saldırı) ait `TMotionVectorMap`'i ve mod indeksini içerir.
    *   `TMotionModeDataMap`: Animasyon mod indeksine göre `SMotionModeData` işaretçilerini tutan bir harita.
    *   `SModelData`: `NRaceData::TAttachingDataVector` (eklenti verileri vektörü) içerir.
    *   `TModelDataMap`: Model indeksine göre `SModelData` tutan bir harita.
    *   `TComboData`: Kombo saldırılarına ait animasyon indekslerini içeren bir vektör (`TComboIndexVector`).
    *   `TNormalAttackIndexMap`, `TComboAttackDataMap`: Normal ve kombo saldırıların animasyonlarını yönetmek için haritalar.
    *   `SSkin`: Bir model parçasının kaynak ve hedef doku (skin) dosyalarını ve parça türünü tanımlar.
    *   `SHair`: Saç modelinin dosya adını ve ilişkili `SSkin` vektörünü içerir.
    *   `SShape`: Vücut şekli modelinin dosya adını ve ilişkili `SSkin` vektörünü içerir.
*   **`CRaceData` Sınıfı**:
    *   **Statik Metotlar**: `New`, `Delete`, `CreateSystem`, `DestroySystem` (CDynamicPool yönetimi için).
    *   **Yapıcı/Yıkıcı**: `CRaceData()`, `~CRaceData()`.
    *   **`Destroy()`**: Nesneyi temizler, tüm kaynakları serbest bırakır.
    *   **Erişim Metotları (Getters)**:
        *   Temel model, LOD modeli, öznitelik dosyası, animasyon listesi dosyası adları ve `CGraphicThing`/`CAttributeData` işaretçileri için.
        *   Belirli bir parçaya bağlı kemik adı (`GetAttachingBoneName`).
        *   Animasyon modu, animasyon vektörü, animasyon verisi işaretçileri (`GetMotionModeDataPointer`, `GetMotionVectorPointer`, `GetMotionDataPointer`).
        *   Eklenti verileri (`GetAttachingDataPointer`, `GetCollisionDataPointer`).
        *   Duman efekti ID'leri ve kemik adı (`GetSmokeEffectID`, `GetSmokeBone`).
        *   Saç ve şekil verileri (`FindHair`, `FindShape`).
        *   Normal ve kombo saldırı verileri (`GetNormalAttackIndex`, `GetComboDataPointer`).
    *   **Veri Yükleme ve Kayıt Metotları**:
        *   `LoadRaceData(const char* c_szFileName)`: Ana ırk veri dosyasını yükler (genellikle bir script dosyası).
        *   `RegisterMotionData(...)`: Bir animasyon dosyasını yükler, `CRaceMotionData` oluşturur ve kaydeder.
        *   `RegisterMotionMode(...)`: Yeni bir animasyon modu kaydeder.
        *   `RegisterAttachingBoneName(...)`, `ChangeAttachingBoneName(...)`: Eklenti kemik adlarını kaydeder/değiştirir.
        *   `RegisterNormalAttack(...)`, `ReserveComboAttack(...)`, `RegisterComboAttack(...)`: Saldırı animasyonlarını ve kombolarını tanımlar.
        *   `SetShapeModel(...)`, `AppendShapeSkin(...)`: Vücut şekli modellerini ve dokularını ayarlar.
        *   `SetHairSkin(...)`: Saç modellerini ve dokularını ayarlar.
    *   **Korunan Üyeler**: Irk indeksi, duman efekti ID'leri, çeşitli model ve animasyon verilerini tutan haritalar ve vektörler, temel dosya adları (model, ağaç, öznitelik, animasyon listesi), eklenti kemik adları haritası, saç ve şekil verileri haritaları.

#### `RaceData.cpp` - C++ Implementasyon Detayları (Özet)

*   **Statik Havuz Yönetimi**: `ms_kPool` (`CRaceData` için) ve `ms_MotionModeDataPool` (`SMotionModeData` için) `CDynamicPool` örnekleridir. `New`, `Delete`, `CreateSystem`, `DestroySystem` metotları bu havuzları yönetir.
*   **Veri Erişimi**: `Get...` metotları, genellikle ilgili harita veya vektör içinde arama yaparak istenen veriyi veya işaretçiyi döndürür. Bulunamaması durumunda `NULL` veya `FALSE` dönerler. Bazı metotlar, varsayılan moda (örn: `MODE_GENERAL`) düşme (fallback) mantığı içerir.
*   **Saç ve Şekil Yönetimi**: `FindHair`, `FindShape` metotları, ID'ye göre kayıtlı saç/şekil verilerini bulur. `SetHairSkin`, `SetShapeModel`, `AppendShapeSkin` metotları bu verileri ayarlar ve yapılandırır.
*   **Animasyon Yönetimi**:
    *   `RegisterMotionMode`: Yeni bir `SMotionModeData` nesnesini havuzdan alır ve `m_pMotionModeDataMap`'e ekler.
    *   `RegisterMotionData`: `CRaceMotionData::New()` ile yeni bir animasyon verisi nesnesi oluşturur, `LoadMotionData` ile dosyadan yükler ve `NEW_RegisterMotion` aracılığıyla `CRaceData`'ya kaydeder. `CResourceManager` kullanarak animasyonun `.gr2` dosyasını (`CGraphicThing`) alır.
    *   `NEW_RegisterMotion`: Verilen `CRaceMotionData` ve `CGraphicThing`'i ilgili `SMotionModeData` içindeki `MotionVectorMap`'e ekler.
    *   `GetMotionKey`: Belirli bir mod ve animasyon indeksi için genel bir animasyon anahtarı (`MOTION_KEY`) oluşturur, eğer özel modda animasyon yoksa genel moda (örn: `MODE_HORSE` veya `MODE_GENERAL`) başvurur.
*   **Saldırı Komboları**: `ReserveComboAttack` ile belirli bir kombo türü için yer ayrılır. `RegisterComboAttack` ile bu kombo dizisindeki belirli bir adıma animasyon indeksi atanır.
*   **Kaynak Yönetimi**: `GetBaseModelThing`, `GetLODModelThing`, `GetAttributeDataPtr` gibi metotlar, `CResourceManager` aracılığıyla ilgili kaynakları (genellikle `CGraphicThing` veya `CAttributeData`) talep üzerine yükler ve işaretçilerini saklar.
*   **`Destroy()` ve `__Initialize()`**: `Destroy`, tüm dinamik olarak ayrılmış bellekleri (özellikle `CRaceMotionData` nesneleri ve `SMotionModeData` nesneleri) serbest bırakır, haritaları ve vektörleri temizler. `__Initialize`, üye değişkenleri başlangıç değerlerine ayarlar.
*   `LoadRaceData` fonksiyonunun implementasyonu bu dosyada görünmüyor, muhtemelen bir script ayrıştırıcısı (parser) aracılığıyla başka bir yerde ele alınıyor ve `CRaceData`'nın public metotlarını çağırarak dolduruyor.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Karakter Oluşturma ve Gösterimi**: Bir karakter (oyuncu veya NPC) oluşturulduğunda, ilgili ırkın `CRaceData` nesnesi kullanılarak temel modeli, saç stili, vücut şekli ve diğer görsel özellikleri yüklenir ve ayarlanır.
*   **Animasyon Kontrolü**: Karakterin yürüme, koşma, durma, saldırma, büyü yapma gibi tüm hareketleri, `CRaceData` içinde tanımlanan animasyon modları ve animasyon verileri aracılığıyla yönetilir. Oyun mantığı, duruma göre uygun animasyon anahtarını talep eder.
*   **Ekipman ve Eklentiler**: Silahlar, zırhlar veya diğer eklenti nesneleri, `CRaceData` içinde tanımlanan `AttachingBoneNameMap` ve `TAttachingDataVector` kullanılarak karakter modelinin doğru kemiklerine bağlanır.
*   **Çarpışma Tespiti**: Karakterin ve eklentilerinin çarpışma geometrileri (`NRaceData::TAttachingData` içinde `ATTACHING_DATA_TYPE_COLLISION_DATA` türü) `CRaceData` üzerinden elde edilir.
*   **Karakter Özelleştirme**: Oyuncuların karakterlerinin saç stilini, rengini veya diğer görünüm özelliklerini değiştirmesi, `CRaceData`'daki `SHair` ve `SShape` verilerinin değiştirilmesi veya farklı olanlarının seçilmesiyle gerçekleştirilir.
*   **Savaş Sistemi**: Normal saldırılar ve özel kombo saldırıları, `TNormalAttackIndexMap` ve `TComboAttackDataMap` içinde tanımlanan animasyon dizileriyle yürütülür.
*   **Kaynak Optimizasyonu**: LOD (Level of Detail) modelleri (`GetLODModelThing`) ve kaynakların `CResourceManager` ile yönetilmesi, performans optimizasyonuna katkıda bulunur. `CDynamicPool` kullanımı, sık oluşturulan/yok edilen `CRaceData` ve `SMotionModeData` nesneleri için bellek verimliliği sağlar.

---

### Irk Veri Dosyası Yükleme (`RaceDataFile.cpp` içindeki `CRaceData::LoadRaceData`)

#### Fonksiyonun Genel Amacı

Bu bölüm, `CRaceData` sınıfının `LoadRaceData` metodunun `RaceDataFile.cpp` dosyasındaki implementasyonunu detaylandırır. Bu metot, bir ırka ait ana yapılandırma dosyasını (genellikle `.msm` uzantılı bir metin dosyası) okuyarak ırkın temel özelliklerini, modellerini, dokularını, eklenti bilgilerini ve diğer ilişkili verilerini yükler.

#### `CRaceData::LoadRaceData(const char* c_szFileName)` Implementasyon Detayları

*   **Dosya Yükleme**: `CTextFileLoader` kullanarak belirtilen `c_szFileName` adlı dosyayı yükler.
*   **Temel Bilgiler**: Dosyadan aşağıdaki anahtar kelimelerle temel dosya ve kemik adlarını okur:
    *   `basemodelfilename`: Ana model dosyasının adı.
    *   `treefilename`: Ağaç modeli dosyasının adı (eğer varsa, örneğin bineklere dönüşen canavarlar için).
    *   `attributefilename`: Öznitelik dosyasının adı (`.atr`).
    *   `smokebonename`: Duman efektlerinin çıkacağı kemiğin adı.
    *   `motionlistfilename`: Animasyon listesi dosyasının adı (genellikle `motlist.txt`).
*   **Duman Efektleri (`smokefilename`)**: Eğer `smokefilename` anahtarı varsa, bir token vektörü olarak duman efekti tanımlarını okur. Her tanım bir duman türü indeksi ve efekt dosya adından oluşur. `CEffectManager::Instance().RegisterEffect2` ile bu efektleri kaydeder ve ID'lerini `m_adwSmokeEffectID` dizisinde saklar.
*   **Şekil Verileri (`shapedata`)**:
    *   `shapedata` adlı bir alt düğüme (child node) geçer.
    *   `pathname`: Şekil dosyaları için genel bir yol.
    *   `shapedatacount`: Toplam şekil sayısı.
    *   Her bir şekil için (`shapedata` alt düğümü, indeks ile erişilir):
        *   `specialpath`: Geçici veya özel bir yol (önceki `pathname`'i geçersiz kılabilir).
        *   `shapeindex`: Şeklin benzersiz ID'si.
        *   `model` veya `local_model`: Şekil model dosyasının adı. `local_model` tam yol belirtirken, `model` `pathname` veya `specialpath` ile birleştirilir.
        *   `sourceskin`/`targetskin` veya `local_sourceskin`/`local_targetskin`: Model için kaynak ve hedef doku dosyaları. `local_` ön ekli olanlar tam yol belirtir. Birden fazla doku seti (`sourceskin2`/`targetskin2`) desteklenebilir.
        *   `SetShapeModel` ve `AppendShapeSkin` metotları çağrılarak bu veriler `CRaceData` içine kaydedilir.
*   **Saç Verileri (`hairdata`)**:
    *   `hairdata` adlı bir alt düğüme geçer.
    *   `pathname`: Saç dosyaları için genel bir yol.
    *   `hairdatacount`: Toplam saç modeli sayısı.
    *   Her bir saç modeli için (`hairdata` alt düğümü, indeks ile erişilir):
        *   `specialpath`: Geçici veya özel bir yol.
        *   `hairindex`: Saç modelinin ID'si.
        *   `model`, `sourceskin`, `targetskin`: Saç modeli ve doku dosyaları.
        *   `SetHairSkin` metodu çağrılarak bu veriler kaydedilir.
*   **Uçuş Hedefi Ayarlama (`flytargetadjustposition`)** (ENABLE_FLY_TARGET_POSITION tanımlıysa):
    *   `flytargetadjustposition` alt düğümünden `adjustposition` (bir `D3DXVECTOR3`) okunur ve `m_v3FlyTargetAdjustPosition` ile `m_bHasFlyTargetAdjustPosition` ayarlanır.
*   **Eklenti Verileri (`attachingdata`)**:
    *   `attachingdata` alt düğümüne geçer.
    *   `NRaceData::LoadAttachingData(TextFileLoader, &m_AttachingDataVector)` çağrılarak eklenti verileri (çarpışma küreleri, efekt eklenti noktaları vb.) okunur ve `m_AttachingDataVector` içine yüklenir.

#### Kullanım Amacı

`LoadRaceData` metodu, `CRaceManager` tarafından bir ırkın tüm temel yapılandırmasını `.msm` dosyasından yüklemek için kullanılır. Bu, karakterlerin oyunda doğru şekilde görünmesini ve davranmasını sağlayan verilerin merkezi bir noktadan yüklenmesini sağlar.

---

### Irk Yöneticisi (`RaceManager.h` ve `RaceManager.cpp`)

#### Dosyaların Genel Amacı

`RaceManager.h` ve `RaceManager.cpp`, oyundaki tüm `CRaceData` nesnelerinin yönetiminden sorumlu olan `CRaceManager` sınıfını tanımlar. `CSingleton` deseni ile tasarlanmıştır, yani oyun boyunca tek bir örneği bulunur. `CRaceManager`, ırk verilerini (modeller, animasyonlar vb.) talep üzerine yükler, saklar, erişim sağlar ve seçili olan ırkı yönetir. Ayrıca, ırk kaynak dosyalarının bulunacağı yolları yönetir ve ırkların yükseklikleri gibi ek bilgileri saklayabilir.

#### `RaceManager.h` - Başlık Dosyası Tanımları

*   **`#include "RaceData.h"`**: `CRaceData` sınıfını dahil eder.
*   **`class CRaceManager : public CSingleton<CRaceManager>`**: Tekil (singleton) `CRaceManager` sınıfı.
    *   **`TRaceDataMap`**: `DWORD` (ırk ID'si) ile `CRaceData*` (ırk verisi işaretçisi) eşleştiren bir harita türü.
    *   **Yapıcı/Yıkıcı**: `CRaceManager()`, `~CRaceManager()`.
    *   **`Create()` / `Destroy()`**: `CRaceManager`'ı ve bağımlı sistemleri (örneğin `CRaceData` ve `CRaceMotionData` için dinamik havuzlar) başlatır ve yok eder.
    *   **Irk Kayıt Metotları**:
        *   `RegisterRaceName(DWORD dwRaceIndex, const char* c_szName)`: Bir ırk ID'sini bir isimle eşleştirir.
        *   `RegisterRaceSrcName(const char* c_szName, const char* c_szSrcName)`: Bir ırk adını bir kaynak adıyla (genellikle klasör adı) eşleştirir.
    *   **Yol Yönetimi**:
        *   `SetPathName(const char* c_szPathName)`: Genel bir temel yol ayarlar.
        *   `GetFullPathFileName(const char* c_szFileName)`: Verilen dosya adına temel yolu ekleyerek tam yol döndürür.
    *   **Irk Yüksekliği**:
        *   `SetRaceHeight(int iVnum, float fHeight)`: Bir ırk VNUM'u için ek yükseklik değeri ayarlar.
        *   `float GetRaceHeight(int iVnum)`: Ayarlanan ek yüksekliği döndürür.
    *   **Irk Veri Yönetimi**:
        *   `CreateRace(DWORD dwRaceIndex)`: Belirtilen ID için boş bir `CRaceData` nesnesi oluşturur ve haritaya ekler (bu fonksiyon doğrudan yükleme yapmaz, genellikle script taraflı manuel kurulum için kullanılır).
        *   `SelectRace(DWORD dwRaceIndex)`: Belirtilen ID'ye sahip ırkı aktif/seçili ırk olarak ayarlar.
        *   `CRaceData* GetSelectedRaceDataPointer()`: Aktif/seçili `CRaceData` işaretçisini döndürür.
        *   `BOOL GetRaceDataPointer(DWORD dwRaceIndex, CRaceData** ppRaceData)`: Belirtilen ID'ye sahip `CRaceData` işaretçisini döndürür. Eğer ırk daha önce yüklenmemişse, `__LoadRaceData` çağrılarak yüklenir.
    *   **Korunan Yükleme Metotları**:
        *   `CRaceData* __LoadRaceData(DWORD dwRaceIndex)`: Asıl ırk verisi yükleme işlemini yapar.
        *   `bool __LoadRaceMotionList(CRaceData& rkRaceData, const char* pathName, const char* motionListFileName)`: Bir ırkın animasyon listesi dosyasını (`motlist.txt`) yükler.
    *   **Korunan Üyeler**: `m_RaceDataMap` (yüklenmiş ırk verilerini tutar), `m_kMap_stRaceName_stSrcName` (isim-kaynak adı eşlemesi), `m_kMap_dwRaceKey_stRaceName` (ID-isim eşlemesi), `m_kMap_iRaceKey_fRaceAdditionalHeight` (VNUM-yükseklik eşlemesi).
    *   **Özel Üyeler**: `m_strPathName` (temel yol), `m_pSelectedRaceData` (seçili ırk verisi).

#### `RaceManager.cpp` - C++ Implementasyon Detayları (Özet)

*   **Yardımcı Fonksiyonlar**:
    *   `__IsGuildRace(unsigned race)`, `__IsNPCRace(unsigned race)`: Verilen ırk ID'sinin lonca veya NPC ırkı olup olmadığını kontrol eder.
    *   `__GetRaceResourcePathes(unsigned race, std::vector <std::string>& vec_stPathes)`: Irk ID'sine ve türüne göre (`__IsGuildRace`, `__IsNPCRace` kullanarak) ırk kaynak dosyalarının aranacağı olası klasör yollarının bir listesini oluşturur (örn: `d:/ymir work/npc/`, `d:/ymir work/monster/`).
*   **`__LoadRaceData(DWORD dwRaceIndex)`**: Irk verilerini yükleyen ana fonksiyondur.
    *   Önce `m_kMap_dwRaceKey_stRaceName`'den ırk adını alır.
    *   Eğer ırk adı '#' ile başlıyorsa (LOAD_LOCAL_RESOURCE), bu yerel bir kaynak olduğunu belirtir ve yolu doğrudan kullanır.
    *   Aksi halde, `__GetRaceResourcePathes` ile olası kaynak yollarını alır.
    *   Her bir yol için, ırkın `.msm` dosyasını (örn: `d:/ymir work/npc/savasci/savasci.msm`) bulmaya çalışır. `CRaceData::LoadRaceData` (önceki bölümde detaylandırılan) ile bu dosyayı yükler.
    *   `.msm` dosyası başarıyla yüklendikten sonra, o ırka ait animasyon listesi dosyasını (genellikle `motlist.txt`, adı `.msm` içinden okunur) `__LoadRaceMotionList` ile yükler.
    *   Yükleme başarılı olursa `CRaceData` işaretçisini döndürür, aksi halde `NULL`.
*   **`__LoadRaceMotionList(...)`**: Bir ırkın animasyon listesi dosyasını (`motlist.txt`) yükler.
    *   Dosyayı `CEterPackManager` ve `CMemoryTextFileLoader` kullanarak okur.
    *   Dosyadaki her satırı (mod, tür, dosya adı, yüzde) ayrıştırır.
    *   Animasyon türü string'ini (`WAIT`, `WALK`, `ATTACK` vb.) `CRaceMotionData` içindeki sabit animasyon ID'lerine (`CRaceMotionData::NAME_WAIT` vb.) eşleştirmek için statik bir harita (`s_kMap_stType_dwIndex`) kullanır.
    *   `rkRaceData.RegisterMotionData` çağrılarak her bir animasyon dosyası ilgili `CRaceData` nesnesine kaydedilir.
    *   Standart saldırı animasyonunu (`CRaceMotionData::NAME_NORMAL_ATTACK`) kaydeder.
*   **Kayıt Metotları**: `RegisterRaceName`, `RegisterRaceSrcName`, `SetRaceHeight` gibi metotlar ilgili haritalara (map) veri ekler.
*   **`GetRaceDataPointer(...)`**: Bir ırk ID'si için `CRaceData` talep edildiğinde, önce `m_RaceDataMap` içinde var olup olmadığına bakar. Varsa, onu döndürür. Yoksa, `__LoadRaceData` ile yükler, haritaya ekler ve sonra döndürür (lazy loading).
*   **`Create()` / `Destroy()`**: `Create`, `CRaceMotionData` ve `CRaceData` için statik sistemleri (dinamik havuzlar) başlatır. `Destroy`, `__DestroyRaceDataMap` çağırarak tüm yüklenmiş `CRaceData` nesnelerini siler.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Merkezi Irk Yönetimi**: Oyundaki tüm karakter ırklarına ait verilerin tek bir noktadan yönetilmesini, yüklenmesini ve erişilmesini sağlar.
*   **Dinamik Yükleme (Lazy Loading)**: Irk verileri sadece ihtiyaç duyulduğunda (`GetRaceDataPointer` çağrıldığında) yüklenir, bu da oyunun başlangıç yükleme süresini azaltabilir.
*   **Kaynak Yolu Yönetimi**: Irkların model ve animasyon dosyalarının farklı klasör yapılarında bulunabilmesine olanak tanır (`__GetRaceResourcePathes`).
*   **Irk Yapılandırması**: Oyun sunucusundan veya scriptlerden gelen ırk ID'lerine göre doğru karakter modellerinin, animasyonlarının ve diğer özelliklerin yüklenmesini sağlar.
*   **Oyun Mantığı Entegrasyonu**: Oyunun karakter oluşturma, animasyon kontrolü, savaş mekanikleri gibi çeşitli sistemleri, ihtiyaç duydukları ırk verilerine `CRaceManager` aracılığıyla erişir.
*   **Veri Tutarlılığı**: Aynı ırk verisinin birden fazla kez yüklenmesini engeller, mevcutsa onu kullanır.

---

### Irk Animasyon Verileri ve Olayları (`RaceMotionData.h`, `RaceMotionData.cpp`, `RaceMotionDataEvent.h`)

#### Dosyaların Genel Amacı

Bu dosyalar, bir karakter ırkına ait tek bir animasyonun (motion) tüm detaylarını ve bu animasyon sırasında meydana gelebilecek olayları (event) yöneten `CRaceMotionData` sınıfını ve ilgili olay yapılarını tanımlar. `CRaceMotionData` nesneleri, bir animasyonun türünü, süresini, döngü özelliklerini, kombo zamanlamalarını, saldırı etkilerini ve animasyonun belirli noktalarında tetiklenecek ses, efekt gibi çeşitli olayları barındırır. Bu veriler genellikle o animasyona özel bir script dosyasından (`.msd` gibi) yüklenir.

#### `RaceMotionDataEvent.h` - Animasyon Olay Yapıları (Özet)

Bu başlık dosyası, `NMotionEvent` isim alanı (namespace) içinde, `SMotionEventData` adlı bir taban yapıdan türeyen çeşitli animasyon olayı veri yapılarını tanımlar. Her olay yapısı, olayın türünü, başlangıç zamanını ve olaya özgü diğer parametreleri içerir. Başlıca olay türleri şunlardır:

*   **`SMotionEventDataScreenWaving`**: Ekranda dalgalanma efekti yaratır (güç, etki alanı, süre).
*   **`SMotionEventDataScreenFlashing`**: Ekranı belirli bir renkle anlık olarak parlatır (kullanımda değil gibi görünüyor).
*   **`SMotionEventDataEffect`**: Belirli bir kemiğe bağlı veya bağımsız bir parçacık efekti (.efc) oynatır (efekt dosya adı, pozisyon, takip etme, bağımsızlık durumu).
*   **`SMotionEventDataEffectToTarget`**: Hedefe doğru bir efekt oynatır (özellikle balıkçılık gibi senaryolar için).
*   **`SMotionEventDataFly`**: Bir uçan nesne (mermi, büyü vb. için `.fly` dosyası) oluşturur ve fırlatır (uçan nesne dosya adı, eklenti kemiği, pozisyon).
*   **`SMotionEventDataAttack`**: Bir saldırı olayını tanımlar; çarpışma verilerini (`NRaceData::TCollisionData`) ve saldırı özelliklerini (`NRaceData::TAttackData`) içerir. Vuruş işleminin (hit process) aktif olup olmayacağını belirtir.
*   **`SMotionEventDataSound`**: Belirli bir ses dosyasını çalar.
*   **`SMotionEventDataCharacterShow` / `Hide`**: Karakteri görünür veya görünmez yapar.
*   **`SMotionEventDataWarp`**: Karakteri ışınlar (detayları bu yapıda doğrudan belirtilmemiş, oyun mantığına bırakılmış olabilir).
*   **`SMotionEventDataRelativeMoveOn` / `Off`**: Karaktere göreceli hareketin başlayacağını veya biteceğini belirtir, bir taban hız alabilir.

Her olay yapısı, verilerini metin dosyasından yüklemek için bir `Load` ve kaydetmek için bir `Save` metoduna sahiptir.

#### `RaceMotionData.h` - `CRaceMotionData` Sınıf Tanımları (Özet)

*   **Enum Tanımları**:
    *   `EType`: Animasyonun genel türünü belirtir (örn: `TYPE_WAIT`, `TYPE_MOVE`, `TYPE_ATTACK`, `TYPE_SKILL`).
    *   `EMode`: Animasyonun hangi durumda (modda) kullanılacağını belirtir (örn: `MODE_GENERAL`, `MODE_ONEHAND_SWORD`, `MODE_HORSE`).
    *   `EName`: Standartlaştırılmış animasyon isimlerini tanımlar (örn: `NAME_WAIT`, `NAME_RUN`, `NAME_NORMAL_ATTACK`, `NAME_SKILL` + indeks). Bu isimler, `CRaceManager::__LoadRaceMotionList` içinde `motlist.txt` dosyasındaki stringlerle eşleştirilir.
    *   `EMotionEventType`: `RaceMotionDataEvent.h` dosyasında tanımlanan olay türlerini listeler.
*   **Yapılar ve Typedef'ler**:
    *   `TComboInputData`: Kombo saldırıları için bir sonraki saldırının girilebileceği zaman aralıklarını (başlangıç, bir sonraki kombo zamanı, bitiş) tutar.
    *   `TMotionEventDataVector`: `NMotionEvent::SMotionEventData*` işaretçilerinden oluşan bir vektör, animasyona ait tüm olayları saklar.
*   **`CRaceMotionData` Sınıfı**:
    *   **Statik Metotlar**: `New`, `Delete`, `CreateSystem`, `DestroySystem` (`CDynamicPool` yönetimi için).
    *   **Temel Özellikler**: İsim (`m_eName`), tür (`m_eType`), kilitli olup olmadığı (`m_isLock`).
    *   **Dosya Bilgileri**: İlişkili animasyon dosyası adı (`m_strMotionFileName`), ses script dosyası adı (`m_strSoundScriptDataFileName`).
    *   **Süre ve Döngü**: Animasyon süresi (`m_fMotionDuration`), döngü sayısı (`m_iLoopCount`), döngü ise başlangıç/bitiş zamanları (`m_fLoopStartTime`, `m_fLoopEndTime`), döngünün iptal edilebilir olup olmadığı (`m_bCancelEnableSkill`).
    *   **Hareket Birikimi**: Animasyonun sonunda karakterin ne kadar yer değiştireceği (`m_accumulationPosition`, `m_isAccumulationMotion`).
    *   **Kombo ve Saldırı Verileri**: Kombo giriş zamanları (`m_ComboInputData`, `m_isComboMotion`), saldırı detayları (`m_MotionAttackData`, `m_isAttackingMotion` - `NRaceData::TMotionAttackData` türünde), sıçrama (splash) olayı olup olmadığı (`m_hasSplashEvent`).
    *   **Olay Yönetimi**: `m_MotionEventDataVector` ile olayları tutar. Olay sayısını, belirli bir indeksteki olayı veya saldırı olayını getirmek için metotlar sunar (`GetMotionEventDataCount`, `GetMotionEventDataPointer`, `GetMotionAttackingEventDataPointer`).
    *   **Ses Yönetimi**: `m_SoundInstanceVector` (`NSound::TSoundInstanceVector`) ile animasyonla ilişkili ses örneklerini tutar.
    *   **Yükleme/Kaydetme**: `LoadMotionData` (animasyon detaylarını ve olaylarını yükler), `LoadSoundScriptData` (ses scriptini yükler), `SaveMotionData` (World Editor için kaydetme).

#### `RaceMotionData.cpp` - C++ Implementasyon Detayları (Özet)

*   **Dinamik Havuz**: `CRaceMotionData` nesneleri için `ms_kPool` adlı bir `CDynamicPool` kullanılır.
*   **`SetName(UINT eName)`**: Verilen `EName` enum değerine göre animasyonun `EType`'ını (örn: `NAME_WAIT` ise `TYPE_WAIT`) ve `m_isLock` durumunu (saldırı, kombo, skill türleri için `TRUE`) ayarlar.
*   **`LoadMotionData(const char* c_szFileName)`**: Bir animasyonun detaylarını ve olaylarını genellikle `.msd` (Motion Script Data gibi) uzantılı bir metin dosyasından yükler:
    *   `CTextFileLoader` kullanarak dosyayı okur.
    *   Temel bilgileri (`motionfilename`, `motionduration`, `accumulation`) ayrıştırır.
    *   Alt düğümleri (`comboinputdata`, `attackingdata`, `loopdata`, `motioneventdata`) işler.
    *   `motioneventdata` bölümünde, `motioneventdatacount` kadar olayı okur. Her bir `event` alt düğümü için:
        *   `motioneventtype`'a göre uygun `NMotionEvent::SMotionEventData` türemiş nesnesini (`TMotionFlyEventData`, `TMotionEffectEventData` vb.) oluşturur.
        *   Oluşturulan nesnenin `Load` metodunu çağırarak olaya özgü verileri (örn: efekt adı, pozisyonu, süresi) yükler.
        *   Olayın başlangıç zamanını (`startingtime`) okur ve bunu frame indeksine (`dwFrame`) çevirir.
        *   Olay nesnesini `m_MotionEventDataVector`'e ekler.
    *   Dosya adından (`.msd` yerine `.mss` yaparak) ses script dosya adını türetir ve `LoadSoundScriptData`'yı çağırır.
*   **`LoadSoundScriptData(const char* c_szFileName)`**: Belirtilen `.mss` (Motion Sound Script) dosyasını `NSound::LoadSoundInformationPiece` ile okur ve ses verilerini `m_SoundInstanceVector` içine `NSound::DataToInstance` ile dönüştürerek yükler.
*   **Erişim Metotları**: `GetType`, `IsLock`, `GetLoopCount`, `GetMotionDuration` gibi metotlar, yüklenmiş verileri döndürür.
*   **`Initialize()` / `Destroy()`**: Üye değişkenleri varsayılan değerlere ayarlar ve `m_MotionEventDataVector` gibi dinamik bellek alanlarını temizler.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Detaylı Animasyon Kontrolü**: Her bir karakter animasyonunun (yürüme, koşma, saldırı, yetenek kullanımı, duygular vb.) tüm yönlerini tanımlar.
*   **Zamanlanmış Olaylar**: Animasyonların belirli anlarında efektlerin (örneğin, kılıç savrulurken çıkan parlama, büyü yaparken oluşan partiküller), seslerin (kılıç sesi, adım sesi, karakter konuşması), hasar uygulama zamanlamalarının ve diğer özel olayların (karakterin gizlenmesi, ışınlanması) tetiklenmesini sağlar.
*   **Savaş Mekanikleri**: Saldırı animasyonları (`TYPE_ATTACK`, `TYPE_COMBO`, `TYPE_SKILL`), `TMotionAttackingEventData` aracılığıyla ne zaman ve nasıl hasar verileceğini, hangi çarpışma şeklinin kullanılacağını tanımlar. Kombo sistemleri için `TComboInputData` ile bir sonraki saldırının ne zaman girilebileceği belirlenir.
*   **Görsel ve İşitsel Geri Bildirim**: Oyuncuya ve çevredekilere animasyonlarla senkronize görsel efektler ve sesler sunarak oyun deneyimini zenginleştirir.
*   **Yetenek (Skill) Sistemi**: Yeteneklerin animasyonları, bu yapı üzerinden özel efektler, sesler ve hasar uygulama zamanlamalarıyla birleştirilir.
*   **Dinamik Davranışlar**: `IsLock` özelliği, bir animasyonun başka bir hareketle kesilip kesilemeyeceğini belirleyerek karakterin daha akıcı veya kasıtlı hareket etmesini sağlar.
*   **Kaynak Yönetimi**: `CDynamicPool`, `CRaceMotionData` nesnelerinin verimli bir şekilde oluşturulup yok edilmesine yardımcı olur.

---

### Kar Efekti Yönetimi (`SnowEnvironment.h` ve `SnowEnvironment.cpp`)

#### Dosyaların Genel Amacı

`SnowEnvironment.h` ve `SnowEnvironment.cpp` dosyaları, oyun dünyasında tam ekran bir kar yağışı efekti oluşturmak ve yönetmek için kullanılan `CSnowEnvironment` sınıfını tanımlar. Bu sınıf, `CScreen` sınıfından miras alır ve kar taneciklerinin oluşturulması, güncellenmesi, çizilmesi ve isteğe bağlı bir hareket bulanıklığı (motion blur) efekti uygulanmasından sorumludur. Efektin yoğunluğu ve görünürlüğü, oyun içi ayarlara ve özel etkinlik durumlarına (örneğin, Noel etkinliği) bağlı olabilir.

#### `SnowEnvironment.h` - Başlık Dosyası Tanımları

*   **`#include "../EterLib/GrpScreen.h"`**: Temel ekran ve çizim işlevleri için `CScreen` sınıfını dahil eder.
*   **`class CSnowParticle;`**: `CSnowParticle` sınıfının ileriye dönük bildirimi.
*   **`class CSnowEnvironment : public CScreen`**:
    *   **Yapıcı/Yıkıcı**: `CSnowEnvironment()`, `~CSnowEnvironment()`.
    *   **Temel Kontrol Metotları**: `Create()` (kaynakları oluşturur), `Destroy()` (kaynakları serbest bırakır), `Enable()` (efekti açar), `Disable()` (efekti kapatır).
    *   **Güncelleme ve Çizim**: `Update(const D3DXVECTOR3& c_rv3Pos)` (efektin merkez pozisyonunu günceller), `Deform()` (kar taneciklerini günceller ve yenilerini oluşturur), `Render()` (kar taneciklerini ve blur efektini çizer).
    *   **Korunan Yardımcı Metotlar**: `__Initialize()` (üye değişkenleri sıfırlar), `__CreateBlurTexture()` (blur efekti için dokuları ve render hedeflerini oluşturur), `__CreateGeometry()` (kar tanecikleri için vertex ve index buffer'ları oluşturur), `__BeginBlur()` (blur için çizimi ara hedefe yönlendirir), `__ApplyBlur()` (blur efektini ana hedefe uygular).
    *   **Üye Değişkenler**:
        *   `m_lpOldSurface`, `m_lpOldDepthStencilSurface`: Orijinal render ve derinlik hedeflerini saklamak için.
        *   `m_lpSnowTexture`, `m_lpSnowRenderTargetSurface`, `m_lpSnowDepthSurface`: Kar taneciklerinin çizileceği ara doku ve yüzeyler (blur için).
        *   `m_lpAccumTexture`, `m_lpAccumRenderTargetSurface`, `m_lpAccumDepthSurface`: İsteğe bağlı birikimli blur (accumulation blur) için ek doku ve yüzeyler (aktif olarak kullanılmıyor gibi görünüyor).
        *   `m_pVB`, `m_pIB`: Kar tanecikleri için DirectX vertex ve index buffer'ları.
        *   `m_v3Center`: Kar efektinin merkez dünya koordinatı.
        *   `m_wBlurTextureSize`: Blur dokusunun boyutu.
        *   `m_pImageInstance`: Kar taneciği dokusu için (`snow.dds`) bir `CGraphicImageInstance`.
        *   `m_kVct_pkParticleSnow`: `CSnowParticle` işaretçilerini tutan bir `std::vector`.
        *   `m_dwParticleMaxNum`: Ekranda aynı anda bulunabilecek maksimum kar taneciği sayısı.
        *   `m_bBlurEnable`: Hareket bulanıklığı efektinin aktif olup olmadığını belirten bayrak.
        *   `m_bSnowEnable`: Kar efektinin genel olarak aktif olup olmadığını belirten bayrak.

#### `SnowEnvironment.cpp` - C++ Implementasyon Detayları

*   **Bağımlılıklar**: `CSnowParticle` (her bir kar taneciği için), `PythonSystem` (oyun ayarlarını okumak için - `ENABLE_ENVIRONMENT_EFFECT_OPTION` aktifse), `PythonBackground` (özel etkinlik durumlarını kontrol etmek için - örneğin `IsXMasShowEvent`).
*   **`Enable()` / `Disable()`**: `m_bSnowEnable` bayrağını ayarlar. `Enable()` ilk kez çağrıldığında `Create()` ile kaynakları oluşturur.
*   **`Update(const D3DXVECTOR3& c_rv3Pos)`**: Kar efektinin merkezini (`m_v3Center`) günceller. Eğer efekt kapalıysa ve partikül yoksa işlem yapmaz.
*   **`Deform()`**: Kar taneciklerinin hareketini ve ömrünü yönetir:
    *   Geçen süreyi (`fElapsedTime`) hesaplar.
    *   Aktif kameranın pozisyonu ve bakış yönüne göre yeni kar taneciklerinin doğacağı bir başlangıç bölgesi (`v3ChangedPos`) belirler.
    *   `m_kVct_pkParticleSnow` içindeki her bir `CSnowParticle` için `pSnow->Update(fElapsedTime, v3ChangedPos)` çağrılır.
    *   `pSnow->IsActivate()` `FALSE` ise (yani tanecik ömrünü tamamlamışsa), `CSnowParticle::Delete(pSnow)` ile havuzuna geri bırakılır ve vektörden silinir.
    *   Eğer `m_bSnowEnable` aktifse (ve `ENABLE_ENVIRONMENT_EFFECT_OPTION` tanımlıysa ilgili Python kontrolleri de geçerliyse), `m_dwParticleMaxNum` sınırına kadar (her çağrıda en fazla 10 tane) yeni `CSnowParticle` oluşturulur (`CSnowParticle::New()`), `pSnowParticle->Init(v3ChangedPos)` ile başlatılır ve `m_kVct_pkParticleSnow` vektörüne eklenir.
*   **Blur Efekti Çizimi (`__BeginBlur`, `__ApplyBlur`)**:
    *   `__BeginBlur()`: Eğer `m_bBlurEnable` aktifse, mevcut render hedefini (`m_lpOldSurface`) saklar ve çizimi `m_lpSnowRenderTargetSurface` üzerine yönlendirir. Bu yüzeyi temizler.
    *   `__ApplyBlur()`: Eğer `m_bBlurEnable` aktifse, `m_lpSnowTexture`'ı (kar taneciklerinin çizildiği doku) alır ve saklanmış olan `m_lpOldSurface`'e (ana ekran) tam ekran bir dörtgen (quad) üzerine çizer. Bu, basit bir hareket bulanıklığı etkisi yaratır. Koddaki yorumlanmış bölümler, daha karmaşık bir "accumulation blur" tekniğinin de denendiğini göstermektedir.
*   **`Render()`**: Kar taneciklerini çizer:
    *   Önce `__BeginBlur()` çağrılır.
    *   Maksimum (`m_dwParticleMaxNum`) veya mevcut partikül sayısı kadar (`dwParticleCount`) partikül çizilir.
    *   Vertex buffer (`m_pVB`) kilitlenir.
    *   Her bir `CSnowParticle` için `pSnow->SetCameraVertex(c_rv3Up, c_rv3Cross)` (kameraya göre billboard oryantasyonu için) ve ardından `pSnow->GetVerticies(...)` çağrılarak dört köşe vertex verisi buffer'a yazılır.
    *   Uygun DirectX render durumları ayarlanır (Z-Buffer yazımı kapalı, Alpha Blending açık, Culling kapalı).
    *   `m_pImageInstance` (kar taneciği dokusu) ayarlanır.
    *   `STATEMANAGER.DrawIndexedPrimitive` ile tüm kar tanecikleri üçgen listesi olarak çizilir.
    *   Son olarak `__ApplyBlur()` çağrılır.
*   **Kaynak Yönetimi (`Create`, `Destroy`, `__Initialize`)**:
    *   `__CreateBlurTexture()`: `m_bBlurEnable` ise `m_lpSnowTexture` ve `m_lpAccumTexture` adlı iki D3D dokusu ile bunlara karşılık gelen render hedefi yüzeylerini ve derinlik yüzeylerini oluşturur.
    *   `__CreateGeometry()`: `m_dwParticleMaxNum` kadar kar taneciğini (her biri 4 vertex, 2 üçgen) tutabilecek kapasitede `m_pVB` (vertex buffer) ve `m_pIB` (index buffer) oluşturur. Index buffer, standart quad çizim indisleriyle doldurulur.
    *   `Create()`: `Destroy()`'u çağırarak mevcut kaynakları temizler, ardından `__CreateBlurTexture` ve `__CreateGeometry`'yi çağırır. "d:/ymir work/special/snow.dds" dosyasını `CResourceManager` ile yükleyip `m_pImageInstance`'a atar.
    *   `Destroy()`: Oluşturulan tüm D3D kaynaklarını (`SAFE_RELEASE` ile) serbest bırakır. `m_kVct_pkParticleSnow` içindeki tüm `CSnowParticle` nesnelerini `stl_wipe` ile siler (bu, partiküllerin kendi `Delete` metodunu çağırır) ve `CSnowParticle::DestroyPool()` ile partikül havuzunu temizler.
    *   `__Initialize()`: Tüm işaretçileri `NULL`'a, bayrakları `FALSE`'a ayarlar ve `m_kVct_pkParticleSnow` vektörünü `m_dwParticleMaxNum` kapasitesiyle reserve eder.
*   **Yapıcı (`CSnowEnvironment()`)**: `m_bBlurEnable`'ı `FALSE` (varsayılan olarak blur kapalı), `m_dwParticleMaxNum`'ı 3000, `m_wBlurTextureSize`'ı 512 olarak ayarlar ve `__Initialize()`'ı çağırır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Atmosferik Efekt**: Oyun dünyasına kar yağışı ekleyerek kış veya soğuk iklim temalı haritalarda atmosferi güçlendirir.
*   **Görsel Çeşitlilik**: Özellikle Noel gibi özel etkinlikler sırasında etkinleştirilerek oyun dünyasına mevsimsel bir hava katar (`CPythonBackground::instance().IsXMasShowEvent()` kontrolü).
*   **Performans Odaklı Parçacık Sistemi**: Maksimum partikül sayısı (`m_dwParticleMaxNum`) ile sınırlıdır. `CSnowParticle` nesneleri bir havuzdan (`CDynamicPool` benzeri bir yapı, `CSnowParticle::New/Delete` ile yönetilir) alınıp geri bırakılır.
*   **Hareket Bulanıklığı (Motion Blur)**: İsteğe bağlı olarak (`m_bBlurEnable`) kar taneciklerine veya tüm sahneye (çizim sırasına göre) bir hareket bulanıklığı efekti ekleyerek daha yumuşak bir görünüm sağlayabilir.
*   **Ayarlanabilirlik**: `ENABLE_ENVIRONMENT_EFFECT_OPTION` derleme zamanı sabiti ve `CPythonSystem::instance().GetSnowModeOption()` Python çağrısı ile oyuncunun kar efektini açıp kapatabilmesine olanak tanır.

---

### Kar Taneciği Detayları (`SnowParticle.h` ve `SnowParticle.cpp`)

#### Dosyaların Genel Amacı

Bu dosyalar, `CSnowEnvironment` tarafından yönetilen her bir bireysel kar taneciğini temsil eden `CSnowParticle` sınıfını tanımlar. Sınıf, bir kar taneciğinin oluşturulması (havuzdan), başlatılması (rastgele pozisyon, hız, boyut), güncellenmesi (düşme ve salınım hareketi, ömür kontrolü) ve çizim için gerekli vertex verilerinin sağlanmasından sorumludur.

#### `SnowParticle.h` - Başlık Dosyası Tanımları

*   **`SParticleVertex` Yapısı**: Kar taneciği için vertex yapısını tanımlar (3D pozisyon `v3Pos`, UV koordinatları `u`, `v`).
*   **`BlurVertex` Yapısı**: (Bu dosyada tanımlanmış olsa da `CSnowEnvironment` tarafından kullanılan) Blur efekti için vertex yapısını tanımlar (3D pozisyon `pos`, `rhw`, renk `color`, UV koordinatları `tu`, `tv`).
*   **`class CSnowParticle`**:
    *   **Statik Havuz Yönetimi**: `static CSnowParticle* New()`, `static void Delete(CSnowParticle* pSnowParticle)`, `static void DestroyPool()`. Bu metotlar, `ms_kVct_SnowParticlePool` adlı statik bir `std::vector` kullanarak `CSnowParticle` nesneleri için bir nesne havuzu yönetir.
    *   **`Init(const D3DXVECTOR3& c_rv3Pos)`**: Kar taneciğini, verilen merkez pozisyon (`c_rv3Pos`) etrafında rastgele özelliklerle (pozisyon, hız, boyut, salınım parametreleri) başlatır ve `m_bActivate` bayrağını `true` yapar.
    *   **`SetCameraVertex(const D3DXVECTOR3& rv3Up, const D3DXVECTOR3& rv3Cross)`**: Kar taneciğinin billboard olarak çizilmesi için kameranın yukarı ve yan vektörlerini alarak taneciğin kendi yukarı ve yan ofset vektörlerini (`m_v3Up`, `m_v3Cross`) hesaplar.
    *   **`bool IsActivate()`**: Taneciğin aktif (görünür ve hareketli) olup olmadığını döndürür.
    *   **`Update(float fElapsedTime, const D3DXVECTOR3& c_rv3Pos)`**: Geçen süreye (`fElapsedTime`) ve referans pozisyona (`c_rv3Pos` - genellikle kamera) göre taneciğin pozisyonunu günceller. Basit bir düşme ve X-Y eksenlerinde sinüsoidal bir salınım hareketi uygular. Tanecik belirli bir mesafenin dışına çıkarsa veya çok aşağı düşerse `m_bActivate` bayrağını `false` yapar.
    *   **`GetVerticies(...)`**: Taneciğin dört köşe noktasının (`SParticleVertex`) dünya koordinatlarını ve UV değerlerini hesaplayıp döndürür.
    *   **Korunan Üyeler**: `m_bActivate` (aktiflik durumu), `m_fHalfWidth`/`Height` (boyut), `m_v3Velocity` (hız), `m_v3Position` (pozisyon), `m_v3Up`/`Cross` (kamera göreceli ofsetler), `m_fPeriod`/`curRadian`/`Amplitude` (salınım parametreleri).

#### `SnowParticle.cpp` - C++ Implementasyon Detayları

*   **Sabit**: `c_fSnowDistance = 70000.0f`: Kar taneciklerinin kamera etrafında ne kadar uzağa yayılabileceğini belirleyen bir sabit.
*   **Havuz Mekanizması (`New`, `Delete`, `DestroyPool`)**:
    *   `New()`: Havuz (`ms_kVct_SnowParticlePool`) boşsa yeni bir `CSnowParticle` oluşturur, değilse havuzdan bir tane alır.
    *   `Delete()`: Kullanılmayan `CSnowParticle` nesnesini havuza geri ekler.
    *   `DestroyPool()`: Havuzdaki tüm nesneleri siler.
*   **`Init(const D3DXVECTOR3& c_rv3Pos)`**: Taneciği başlatır:
    *   Pozisyon: `c_rv3Pos` etrafında, `c_fSnowDistance / 10.0f` yarıçaplı bir daire içinde rastgele bir X, Y konumu ve `c_rv3Pos.z`'den 1500-2000 birim yukarıda bir Z konumu.
    *   Hız: X ve Y hızı 0, Z hızı (düşme) -50 ile -200 arasında rastgele bir değer.
    *   Boyut: Genişlik ve yükseklik 2.0 ile 7.0 arasında rastgele bir değer.
    *   Salınım: Periyot, başlangıç radyanı ve genlik rastgele değerler alır.
*   **`Update(float fElapsedTime, const D3DXVECTOR3& c_rv3Pos)`**:
    *   `m_v3Position += m_v3Velocity * fElapsedTime;` ile dikey düşüşü uygular.
    *   `m_v3Position.x += m_v3Cross.x * sin(m_fcurRadian) / 10.0f;` (ve Y için benzeri) ile yatay salınım ekler. Salınım, kameranın yan (`m_v3Cross`) vektörü yönünde ve `m_fcurRadian` ile kontrol edilen sinüs fonksiyonuyla yapılır.
    *   Ömür Kontrolü: Tanecik, `c_rv3Pos.z`'nin 500 birim altına düşerse veya `c_rv3Pos` X/Y koordinatlarından `c_fSnowDistance` kadar uzaklaşırsa `m_bActivate` `false` olur.
*   **`SetCameraVertex(...)`**: Verilen kamera yukarı ve yan vektörlerini taneciğin yarı boyutuyla çarparak `m_v3Up` ve `m_v3Cross` ofsetlerini oluşturur. Bu ofsetler, `GetVerticies` içinde billboard quad'ının köşe noktalarını hesaplamak için kullanılır.
*   **`GetVerticies(...)`**: Taneciğin `m_v3Position`'ı merkez alınarak, `m_v3Up` ve `m_v3Cross` ofsetleri kullanılarak dört adet `SParticleVertex` (sol-alt, sağ-alt, sol-üst, sağ-üst) hesaplanır. UV koordinatları (0,0), (1,0), (0,1), (1,1) olarak ayarlanır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Bireysel Kar Taneciği Simülasyonu**: Her bir `CSnowParticle` nesnesi, kendi rastgele başlangıç koşulları, boyutu, düşme hızı ve salınım hareketi ile bağımsız bir kar taneciğini simüle eder.
*   **Performans İçin Nesne Havuzu**: Sıkça oluşturulup yok edilen kar tanecikleri için bir nesne havuzu kullanarak bellek tahsisi ve serbest bırakma işlemlerinin getireceği yükü azaltır.
*   **Dinamik ve Doğal Görünüm**: Rastgeleleştirilmiş başlangıç parametreleri ve salınım hareketi, kar yağışının daha doğal ve daha az tekrarlayıcı görünmesine katkıda bulunur.
*   **Billboard Çizimi**: Kar tanecikleri, her zaman kameraya dönük 2D dörtgenler (billboard) olarak çizilir. Bu, `SetCameraVertex` ve `GetVerticies` metotları ile sağlanır ve 3D modeller kullanmaya göre çok daha performanslıdır.
*   **Ömür Yönetimi**: Tanecikler belirli bir alanın dışına çıktığında veya yeterince düştüğünde devre dışı bırakılarak sonsuza kadar hesaplanmaları ve çizilmeleri engellenir, bu da performansı artırır.

