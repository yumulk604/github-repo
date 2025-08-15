# EterLib Referans Kılavuzu - Bölüm 5

Bu dosya, `EterLib` referans kılavuzunun devamıdır.

---

## Dosya Bazlı Detaylandırma (Devamı)

### `AttributeData.h` ve `AttributeData.cpp` (`CAttributeData` Sınıfı)

*   **Amaç:** `CAttributeData` sınıfı, bir `CResource` olarak, genellikle 3D modellerle veya harita parçalarıyla ilişkili öznitelik verilerini (attribute data) bir dosyadan yüklemek ve yönetmek için kullanılır. Bu öznitelik verileri temel olarak iki türdedir: statik çarpışma verileri (çeşitli geometrik şekillerle tanımlanır) ve yükseklik verileri (bir grup vertex ile tanımlanır). Bu veriler, oyun içinde fiziksel etkileşimleri ve arazi üzerindeki hareketleri belirlemek için kullanılır.

*   **`AttributeData.h` - Tanımlar ve Bildirimler:**
    *   **`SHeightData` (veya `THeightData`) Yapısı:**
        *   Bir yükseklik verisi parçasını temsil eder.
        *   `szName[32 + 1]`: Yükseklik verisi parçasının adı.
        *   `std::vector<D3DXVECTOR3> v3VertexVector`: Yüksekliği tanımlayan vertex'lerin bir listesi. Bu vertex'ler genellikle bir üçgen ağı (mesh) veya bir poligon zinciri oluşturur.
    *   **`THeightDataVector`:** `std::vector<THeightData>` için bir typedef.
    *   **`CAttributeData` Sınıfı:**
        *   **Kalıtım:** `public CResource`. (Bu, `CAttributeData`'nın `CResourceManager` tarafından yönetilebileceği anlamına gelir.)
        *   **`TRef`:** `CRef<CAttributeData>` için bir typedef.
        *   `static TType Type()`: Sınıf için benzersiz bir tip tanımlayıcısı döndürür ("CAttributeData").
        *   Kurucu (`CAttributeData(const char* c_szFileName)`), Yıkıcı (`virtual ~CAttributeData()`).
        *   `const CStaticCollisionDataVector& GetCollisionDataVector() const`: Yüklenmiş statik çarpışma verilerinin (`CStaticCollisionData` vektörü - bkz. `CollisionData.h`) bir referansını döndürür.
        *   `const THeightDataVector& GetHeightDataVector() const`: Yüklenmiş yükseklik verilerinin (`THeightData` vektörü) bir referansını döndürür.
        *   `size_t AddCollisionData(const CStaticCollisionData& collisionData)`: Programatik olarak yeni bir çarpışma verisi ekler (nadiren kullanılır, genellikle dosyadan yüklenir).
        *   `DWORD GetHeightDataCount() const`: Yükseklik verisi parçalarının sayısını döndürür.
        *   `BOOL GetHeightDataPointer(DWORD dwIndex, const THeightData** c_ppHeightData) const`: Belirtilen indeksteki yükseklik verisi parçasına bir işaretçi döndürür.
        *   `float GetMaximizeRadius()`: Yükseklik verilerindeki vertex'lere dayanarak hesaplanmış, nesnenin kaplayabileceği maksimum bir yarıçapı döndürür (genellikle culling veya kaba testler için).
    *   **Korumalı Sanal Metotlar (`CResource` Arayüzü):**
        *   `OnLoad(int iSize, const void* c_pvBuf)`: Öznitelik verisi dosyasının içeriğini yükler ve ayrıştırır.
        *   `OnClear()`: Yüklenmiş tüm çarpışma ve yükseklik verilerini temizler.
        *   `OnIsEmpty() const`: Hiçbir çarpışma veya yükseklik verisi yüklenmemişse `true` döner.
        *   `OnIsType(TType type)`: Tip kontrolü yapar.
        *   `OnSelfDestruct()`: Nesne yok edilirken `Clear()` çağırır.
    *   **Korumalı Üyeler:**
        *   `m_fMaximizeRadius`: Hesaplanan maksimum yarıçap.
        *   `m_StaticCollisionDataVector`: `CStaticCollisionData` türünde statik çarpışma verilerini saklayan vektör.
        *   `m_HeightDataVector`: `THeightData` türünde yükseklik verilerini saklayan vektör.

*   **`AttributeData.cpp` - Uygulama Detayları:**
    *   **Dosya Formatı (`OnLoad`):**
        1.  Dosyanın başında "AttributeData" başlığının (`c_szAttributeDataFileHeader`) olup olmadığını kontrol eder.
        2.  Ardından, `DWORD` tipinde çarpışma verisi sayısını (`dwCollisionDataCount`) ve yükseklik verisi sayısını (`dwHeightDataCount`) okur.
        3.  **Çarpışma Verisi Yükleme:**
            *   `dwCollisionDataCount` kadar döngüye girer.
            *   Her bir `CStaticCollisionData` için sırayla şunları okur:
                *   `dwType`: Çarpışma tipi (örn: `COLLISION_TYPE_BOX`, `COLLISION_TYPE_SPHERE` - bkz. `CollisionData.h`).
                *   `szName`: Çarpışma parçasının adı (32 byte).
                *   `v3Position`: Çarpışma şeklinin yerel pozisyonu (`D3DXVECTOR3`).
                *   `fDimensions`: Çarpışma şeklinin boyutları (tipine göre 1, 2 veya 3 float değer).
                *   `quatRotation`: Çarpışma şeklinin yerel rotasyonu (`D3DXQUATERNION`).
            *   Bu bilgiler `m_StaticCollisionDataVector`'e eklenir.
        4.  **Yükseklik Verisi Yükleme:**
            *   `dwHeightDataCount` kadar döngüye girer.
            *   Her bir `THeightData` için sırayla şunları okur:
                *   `szName`: Yükseklik parçasının adı (32 byte).
                *   `dwPrimitiveCount`: Bu yükseklik parçasını oluşturan vertex sayısı.
                *   Vertex verileri: `dwPrimitiveCount` kadar `D3DXVECTOR3` doğrudan `rHeightData.v3VertexVector` içine okunur.
            *   Her yükseklik verisi parçası için, `m_fMaximizeRadius` güncellenir. Bu yarıçap, yüklenen vertex'lerin orijinden olan maksimum uzaklığına (her eksende ayrı ayrı, 50.0f eklenerek) göre hesaplanır.
        5.  Eğer `c_pvBuf` `NULL` ise (yani dosya içeriği yoksa), yükleme başarılı sayılır. Bu, bazı durumlarda öznitelik verisi olmayan nesneler için bir esneklik sağlar.
    *   **`OnClear`:** `m_StaticCollisionDataVector` ve `m_HeightDataVector`'ı temizler.
    *   **`OnIsEmpty`:** Her iki vektörün de boş olup olmadığını kontrol eder.
    *   **`GetMaximizeRadius`:** Hesaplanan `m_fMaximizeRadius` değerini döndürür.
    *   Kurucu, `m_fMaximizeRadius`'u 0.0f olarak başlatır.

*   **Kullanım Alanı:**
    *   Bir 3D modelin (örneğin, bir bina, bir kaya) veya bir harita parçasının sahip olduğu çarpışma geometrilerini ve yürülebilir yüzey yüksekliklerini tanımlamak için kullanılır.
    *   Bu `.atr` (genellikle uzantısı budur) dosyaları, 3D modelleme araçlarında veya özel editörlerde oluşturulabilir ve model dosyasıyla birlikte yüklenir.
    *   `CGraphicObjectInstance` veya ondan türeyen sınıflar, bu `CAttributeData` nesnesini kullanarak kendi çarpışma ve yükseklik verilerini alır.
    *   Oyun motoru, bu verileri kullanarak karakterlerin duvarlardan geçmesini engellemek, mermilerin nesnelere çarpmasını tespit etmek veya karakterlerin arazi üzerinde doğru yükseklikte durmasını sağlamak gibi fiziksel etkileşimleri yönetir.
    *   `m_fMaximizeRadius`, nesnenin genel bir sınırlayıcı yarıçapını sağlar ve daha detaylı çarpışma veya culling testlerinden önce hızlı bir ön eleme için kullanılabilir.

### `AttributeInstance.h` ve `AttributeInstance.cpp` (`CAttributeInstance` Sınıfı)

*   **Amaç:** `CAttributeInstance` sınıfı, bir `CAttributeData` nesnesinden alınan öznitelik verilerinin (özellikle yükseklik verileri) oyun dünyasındaki bir örneğini temsil eder. Bu örnek, belirli bir dünya dönüşüm matrisine (`m_matGlobal`) sahiptir ve bu sayede öznitelik verileri (vertex pozisyonları) dünya koordinatlarına dönüştürülür. Temel olarak, yüklenmiş yükseklik verileri üzerinde nokta bazlı yükseklik sorgulamaları (`GetHeight`) ve ışınla kesişim testleri (picking - `Picking`) yapmak için kullanılır. Sınıf, nesne havuzlaması (`CDynamicPool`) ile yönetilir.

*   **`AttributeInstance.h` - Tanımlar ve Bildirimler:**
    *   **`CAttributeInstance` Sınıfı:**
        *   Kurucu (`CAttributeInstance()`), Yıkıcı (`virtual ~CAttributeInstance()`).
        *   `Clear()`: Örneği temizler, yüklenmiş veri referansını ve dönüştürülmüş vertexleri siler.
        *   `IsEmpty() const`: Örnekte işlenmiş yükseklik verisi olup olmadığını kontrol eder.
        *   `GetDataFileName() const`: İlişkili `CAttributeData` nesnesinin dosya adını döndürür.
        *   **Veri ve Dönüşüm Ayarlama:**
            *   `SetObjectPointer(CAttributeData* pAttributeData)`: Bu örneğin temel alacağı `CAttributeData` nesnesini ayarlar.
            *   `RefreshObject(const D3DXMATRIX& c_rmatGlobal)`: Verilen genel dünya matrisini (`c_rmatGlobal`) kullanarak `CAttributeData`'daki yükseklik vertexlerini dünya koordinatlarına dönüştürür ve `m_v3HeightDataVector` içinde saklar. Ayrıca `m_fHeightRadius`'u günceller.
            *   `GetObjectPointer() const`: İlişkili `CAttributeData` nesnesine işaretçi döndürür.
        *   **Sorgulama Metotları:**
            *   `Picking(const D3DXVECTOR3& v, const D3DXVECTOR3& dir, float& out_x, float& out_y)`: Verilen bir ışının (başlangıç `v`, yön `dir`) bu öznitelik örneğinin yükseklik verileriyle (üçgenleriyle) kesişip kesişmediğini test eder. Kesişirse, en yakın kesişim noktasının X ve Y dünya koordinatlarını `out_x` ve `out_y`'ye yazar.
            *   `IsInHeight(float fx, float fy)`: Verilen X,Y dünya koordinatlarının, nesnenin `m_fHeightRadius` ile tanımlanan genel yükseklik etkileşim alanının içinde olup olmadığını hızlıca kontrol eder.
            *   `GetHeight(float fx, float fy, float* pfHeight)`: Verilen X,Y dünya koordinatlarında, bu öznitelik örneğinin yükseklik verileri (üçgenler) üzerindeki Z (yükseklik) değerini hesaplar ve `pfHeight`'e yazar. Birden fazla yükseklik değeri olabilecek durumlarda en yükseğini alır.
            *   `IsHeightData() const`: (Muhtemelen `IsEmpty` ile aynı işlevi görmesi amaçlanmış ancak implementasyonu eksik veya farklı olabilir.)
    *   **Korumalı Metotlar:**
        *   `SetGlobalMatrix(const D3DXMATRIX& c_rmatGlobal)`: (Bildirimi var, implementasyonu yok gibi. `RefreshObject` içinde matris doğrudan atanıyor.)
        *   `SetGlobalPosition(const D3DXVECTOR3& c_rv3Position)`: (Bildirimi var, implementasyonu yok gibi.)
    *   **Korumalı Üyeler:**
        *   `m_fCollisionRadius`: (Kullanılmıyor gibi görünüyor, `CAttributeData` çarpışma verilerini ayrı yönetir.)
        *   `m_fHeightRadius`: `CAttributeData`'dan alınan ve nesnenin yükseklik etkileşim alanının kaba bir yarıçapını belirten değer.
        *   `m_matGlobal`: Bu öznitelik örneğinin dünya dönüşüm matrisi.
        *   `std::vector< std::vector<D3DXVECTOR3> > m_v3HeightDataVector`: `CAttributeData`'dan alınan ve `m_matGlobal` ile dünya koordinatlarına dönüştürülmüş yükseklik vertexlerini saklar (her iç vektör, `THeightData`'daki bir vertex listesine karşılık gelir).
        *   `CAttributeData::TRef m_roAttributeData`: Temel öznitelik verilerini içeren `CAttributeData` nesnesine referans sayımlı işaretçi.
    *   **Statik Havuz Yönetimi:**
        *   `CreateSystem(UINT uCapacity)`, `DestroySystem()`: Nesne havuzunu (`ms_kPool`) oluşturur/yok eder.
        *   `New()`, `Delete(CAttributeInstance* pkInst)`: Havuzdan nesne alır/verir.
        *   `ms_kPool`: `CDynamicPool<CAttributeInstance>` tipinde statik nesne havuzu.

*   **`AttributeInstance.cpp` - Uygulama Detayları:**
    *   **Havuz Yönetimi:** `CreateSystem`, `DestroySystem`, `New`, `Delete` statik metotları, `ms_kPool` aracılığıyla `CAttributeInstance` nesnelerinin verimli bir şekilde oluşturulup silinmesini yönetir.
    *   **`RefreshObject` Metodu:**
        *   `m_matGlobal`'i verilen matrisle günceller.
        *   `m_roAttributeData`'dan `GetMaximizeRadius` ile `m_fHeightRadius`'u alır.
        *   `m_roAttributeData`'daki her bir `THeightData` için (yani her bir yükseklik vertex grubu için):
            *   İç `m_v3HeightDataVector`'ü yeniden boyutlandırır.
            *   `CAttributeData`'daki her bir lokal vertex'i `D3DXVec3TransformCoord` ile `m_matGlobal` matrisini kullanarak dünya koordinatlarına dönüştürür ve `m_v3HeightDataVector`'e kaydeder.
    *   **`GetHeight` Metodu:**
        1.  Örnek boşsa veya verilen X,Y koordinatları `IsInHeight` testini geçemezse (yani `m_fHeightRadius` dışındaysa) `false` döner.
        2.  Y koordinatını `-1.0f` ile çarpar (muhtemelen farklı bir koordinat sistemi uyumu için).
        3.  `m_v3HeightDataVector` içindeki her bir yükseklik üçgeni (her 3 vertex bir üçgen oluşturur) için:
            *   Hızlı bir sınırlayıcı kutu testi yapar: Eğer verilen X,Y koordinatları üçgenin X ve Y eksenlerindeki min/max sınırlarının tamamen dışındaysa, bu üçgeni atlar.
            *   `IsInTriangle2D` (`GrpMath.h`'den) fonksiyonunu kullanarak X,Y koordinatlarının üçgenin 2D izdüşümü içinde olup olmadığını kontrol eder.
            *   Eğer içindeyse:
                *   Üçgenin normal vektörünü (`v3Cross`) hesaplar.
                *   Eğer normalin Z bileşeni sıfır değilse (üçgen Y-X düzlemine paralel değilse), üçgenin düzlem denklemini (`Ax + By + Cz = D`) kullanarak verilen X,Y için Z (yükseklik) değerini hesaplar: `Z = (D - Ax - By) / C`.
                *   Hesaplanan bu yüksekliği, o ana kadar bulunan maksimum yükseklikle (`*pfHeight`) karşılaştırır ve daha büyükse günceller. `bFlag`'ı `true` yapar.
        4.  Eğer en az bir yükseklik bulunduysa `true` döner.
    *   **`Picking` Metodu:**
        1.  Örnek boşsa `false` döner.
        2.  `m_v3HeightDataVector` içindeki her bir yükseklik üçgeni için:
            *   Möller–Trumbore kesişim algoritmasına benzer bir yöntemle ışının üçgenle kesişimini test eder:
                *   Üçgenin normalini (`n`) hesaplar.
                *   Işının üçgen düzlemiyle kesiştiği noktadaki `t` parametresini (ışın denklemi `P(t) = v + t*dir` için) hesaplar.
                *   Kesişim noktasını (`x = v + t*dir`) bulur.
                *   Barycentric koordinatlara benzer bir yöntemle (cross product'ların işaretlerini kontrol ederek) kesişim noktasının üçgenin içinde olup olmadığını test eder.
                *   Eğer içindeyse ve bu ilk bulunan kesişimse veya daha önceki kesişimden daha yakınsa, bu kesişim noktasının X ve Y koordinatlarını (`nx`, `ny`) saklar. `bPicked`'ı `true` yapar.
        3.  Eğer bir kesişim bulunduysa `out_x`, `out_y` güncellenir ve `true` döner.
    *   **`Clear` Metodu:** Referansları temizler, matrisi identity yapar ve dönüştürülmüş vertex vektörünü boşaltır.
    *   Kurucu ve Yıkıcı temeldir; `Initialize` benzeri bir metot olmamasına rağmen, kurucuda ve `Clear`'da üyeler sıfırlanır.

*   **Kullanım Alanı:**
    *   Genellikle bir `CGraphicObjectInstance` (veya türevi) içinde, o nesneye ait `.atr` dosyasından yüklenen `CAttributeData`'nın bir örneğini tutmak için kullanılır.
    *   Nesne dünyaya yerleştirildiğinde veya hareket ettiğinde, `RefreshObject` metodu nesnenin güncel dünya matrisiyle çağrılarak bu öznitelik örneğinin dünya koordinatlarındaki yükseklik verileri güncellenir.
    *   Karakterlerin veya diğer nesnelerin belirli bir X,Y pozisyonunda arazi veya statik bir nesne üzerindeki doğru yüksekliğini bulmak için `GetHeight` metodu kullanılır.
    *   Fare ile dünyadaki belirli bir noktaya (özellikle arazi veya büyük statik nesneler) tıklandığında, hangi geometrik yüzeye tıklandığını ve o noktanın X,Y koordinatlarını bulmak için `Picking` metodu kullanılır. Bu, örneğin karakteri fare tıklamasıyla hareket ettirme işlevinde önemlidir.

### `BlockTexture.h` ve `BlockTexture.cpp` (`CBlockTexture` Sınıfı)

*   **Amaç:** `CBlockTexture` sınıfı, bir `CGraphicDib` (CPU tarafında tutulan bir resim) nesnesinin belirtilen bir dikdörtgen bölgesini (`m_rect`) alarak, bu bölgeyi belirli bir genişlik ve yükseklikte (`m_dwWidth`, `m_dwHeight`) bir Direct3D dokusuna (`m_lpd3dTexture`) aktarmak ve bu dokuyu 2D olarak ekrana çizmek için kullanılır. Temel olarak, CPU'da oluşturulan veya güncellenen bir resim parçasının GPU'ya verimli bir şekilde yüklenmesini ve render edilmesini sağlar. İsteğe bağlı olarak çizim sırasında kırpma (clipping) yapabilir.

*   **`BlockTexture.h` - Tanımlar ve Bildirimler:**
    *   **`CBlockTexture` Sınıfı:**
        *   **Kalıtım:** `public CGraphicBase` (Bu, `ms_lpd3dDevice` gibi statik Direct3D üyelerine erişim sağlar).
        *   Kurucu (`CBlockTexture()`), Yıkıcı (`virtual ~CBlockTexture()`).
        *   `Create(CGraphicDib* pDIB, const RECT& c_rRect, DWORD dwWidth, DWORD dwHeight)`: Verilen `CGraphicDib` nesnesini, kaynak dikdörtgeni (`c_rRect`), hedef doku genişliğini ve yüksekliğini kullanarak bir Direct3D dokusu oluşturur.
        *   `SetClipRect(const RECT& c_rRect)`: Çizim sırasında kullanılacak bir kırpma dikdörtgeni ayarlar ve kırpmayı aktif hale getirir.
        *   `Render(int ix, int iy)`: Dokuyu belirtilen ekran koordinatlarına (`ix`, `iy`) çizer. Eğer kırpma aktifse, `m_clipRect`'e göre kırpar.
        *   `InvalidateRect(const RECT& c_rsrcRect)`: Kaynak `CGraphicDib` üzerinde belirtilen `c_rsrcRect` (DIB koordinatlarında) bölgesindeki değişiklikleri Direct3D dokusuna günceller. Bu, dokunun sadece değişen kısımlarının yeniden yüklenmesini sağlar.
    *   **Korumalı Üyeler:**
        *   `m_pDIB`: Kaynak resim verilerini içeren `CGraphicDib` nesnesine işaretçi.
        *   `m_rect`: Kaynak `CGraphicDib` içinde bu blok dokusuna karşılık gelen dikdörtgen bölge.
        *   `m_clipRect`: `Render` sırasında kullanılacak kırpma dikdörtgeni.
        *   `m_bClipEnable`: Kırpmanın aktif olup olmadığını belirten bayrak.
        *   `m_dwWidth`, `m_dwHeight`: Oluşturulan Direct3D dokusunun genişliği ve yüksekliği.
        *   `m_lpd3dTexture`: Yönetilen Direct3D dokusu.

*   **`BlockTexture.cpp` - Uygulama Detayları:**
    *   **Kurucu/Yıkıcı:** Kurucu üyeleri `NULL` yapar. Yıkıcı, `m_lpd3dTexture`'ı `safe_release` ile serbest bırakır.
    *   **`Create` Metodu:**
        *   `ms_lpd3dDevice->CreateTexture` kullanarak `D3DFMT_A8R8G8B8` formatında, `D3DPOOL_MANAGED` havuzunda bir Direct3D dokusu oluşturur.
        *   Verilen `CGraphicDib`, kaynak `RECT` ve boyutları sınıf üyelerine atar.
    *   **`InvalidateRect` Metodu:**
        1.  Parametre olarak verilen `c_rsrcRect` (güncellenecek DIB bölgesi) ile bu `CBlockTexture`'ın `m_rect`'i (DIB üzerindeki kendi alanı) arasında bir kesişim olup olmadığını kontrol eder.
        2.  Kesişim bölgesini (`clipRect`, doku koordinatlarında) hesaplar.
        3.  `m_lpd3dTexture->LockRect` ile dokunun bu `clipRect` bölgesini kilitler.
        4.  Kaynak `CGraphicDib`'in ilgili piksellerini (`pdwSrc`) alır.
        5.  Kilitlenen doku bölgesindeki pikselleri (`pdwDst`) satır satır günceller:
            *   Kaynak DIB pikseli sıfır değilse (yani görünür bir renkse), bu pikselin rengini alır ve alfa değerini `0xff` (tamamen opak) yaparak hedef dokuya yazar.
            *   Kaynak DIB pikseli sıfırsa (genellikle saydam kabul edilir), hedef doku pikselini de `0` (tamamen saydam siyah) yapar.
        6.  `m_lpd3dTexture->UnlockRect` ile dokunun kilidini açar.
    *   **`SetClipRect` Metodu:** `m_bClipEnable`'ı `true` yapar ve `m_clipRect`'i ayarlar.
    *   **`Render` Metodu:**
        1.  Çizilecek dörtgenin ekran koordinatlarını (`isx`, `isy`, `iex`, `iey`) ve doku koordinatlarını (`su`, `sv`, `eu`, `ev`) hesaplar.
        2.  Eğer `m_bClipEnable` `true` ise:
            *   Dörtgenin `m_clipRect` ile kesişimini kontrol eder. Tamamen dışarıdaysa çizim yapmadan çıkar.
            *   Dörtgenin kenarları `m_clipRect` tarafından kırpılıyorsa, hem ekran koordinatlarını hem de doku koordinatlarını (UV) buna göre ayarlar, böylece sadece görünür kısım doğru dokuyla çizilir.
        3.  `TPDTVertex` tipinde 4 köşe (vertex) oluşturur. Pozisyonları hesaplanan ekran koordinatları, doku koordinatları (UV) ve `0xffffffff` (beyaz) rengiyle ayarlanır.
        4.  `CGraphicBase::SetPDTStream` ile bu vertexleri Direct3D'ye gönderir.
        5.  `CGraphicBase::SetDefaultIndexBuffer(CGraphicBase::DEFAULT_IB_FILL_RECT)` ile iki üçgen çizecek standart index buffer'ını ayarlar.
        6.  `STATEMANAGER.SetTexture(0, m_lpd3dTexture)` ile dokuyu aktif hale getirir.
        7.  `STATEMANAGER.SetVertexShader(D3DFVF_XYZ | D3DFVF_TEX1 | D3DFVF_DIFFUSE)` ile uygun FVF'yi ayarlar.
        8.  `STATEMANAGER.DrawIndexedPrimitive` ile dörtgeni (2 üçgen) çizer.

*   **Kullanım Alanı:**
    *   Genellikle kullanıcı arayüzü (UI) elemanları gibi, CPU tarafında (bir `CGraphicDib` üzerinde) oluşturulan veya dinamik olarak güncellenen 2D grafikleri ekranda göstermek için kullanılır.
    *   Örneğin, bir metin etiketinin resmi `CGraphicDib` üzerine çizildikten sonra, bu `CBlockTexture` aracılığıyla bir dokuya aktarılıp ekranda gösterilebilir.
    *   `InvalidateRect`, DIB'deki küçük değişikliklerin tüm dokuyu yeniden yüklemeden verimli bir şekilde GPU'ya aktarılmasını sağlar. Bu, özellikle sık güncellenen UI elemanları için performans açısından önemlidir.
    *   Harita üzerinde beliren geçici işaretler veya özel efektler gibi, bir ana resim atlasının belirli bir bölümünü alıp çizmek için de kullanılabilir.

### `Camera.h` ve `Camera.cpp` (`CCamera` ve `CCameraManager` Sınıfları)

*   **Amaç:** Bu dosyalar, oyun dünyasındaki 3D kameranın davranışlarını, görünümünü ve yönetimini tanımlar. `CCamera` sınıfı tek bir kamera örneğini temsil ederken, `CCameraManager` birden fazla kamera örneğini yöneten ve aktif kamerayı değiştirmeyi sağlayan bir singleton sınıfıdır.

*   **`CCamera.h` - Tanımlar ve Bildirimler:**
    *   **`CAMERA_TARGET_STANDARD`, `CAMERA_TARGET_FACE` Sabitleri:** Kamera hedefinin standart yüksekliğini ve yüze odaklanma durumundaki yüksekliğini tanımlar.
    *   **`eCameraState` Enum:** Kameranın içinde bulunduğu durumu belirtir (örneğin, `CAMERA_STATE_NORMAL`, `CAMERA_STATE_CANTGODOWN` - bir engele takılıp aşağı gidememesi, `CAMERA_STATE_SCREEN_BY_BUILDING` - bir bina tarafından engellenmesi).
    *   **`CCamera` Sınıfı:**
        *   **Temel Kamera Parametreleri:**
            *   `m_v3Eye`: Kameranın dünyadaki pozisyonu.
            *   `m_v3Target`: Kameranın baktığı noktanın pozisyonu.
            *   `m_v3Up`: Kameranın yukarı yönünü belirten vektör (genellikle `(0,0,1)`).
            *   `m_v3View`: `m_v3Target - m_v3Eye` (bakış yönü).
            *   `m_v3Cross`: `Cross(m_v3Up, m_v3View)` (sağ yön).
        *   **Matrisler:**
            *   `m_matView`: Direct3D için görünüm matrisi.
            *   `m_matInverseView`: Görünüm matrisinin tersi.
            *   `m_matBillboard`: Billboard efektleri için özel matris.
        *   **Açısal Değerler ve Mesafe:**
            *   `m_fPitch`: Kameranın X ekseni etrafındaki eğim açısı.
            *   `m_fRoll`: Kameranın Z ekseni (bakış yönü) etrafındaki dönüş açısı.
            *   `m_fDistance`: Kameranın hedefe olan mesafesi.
        *   **Çarpışma Işınları (`CRay` türünde):**
            *   Arazi ve nesne çarpışmalarını tespit etmek için çeşitli yönlerde tanımlanmış ışınlar (örn: `m_kCameraBottomToTerrainRay`, `m_kLeftObjectCollisionRay`).
            *   `m_fTerrainCollisionRadius`, `m_fObjectCollisionRadius`: Çarpışma testlerinde kullanılan yarıçaplar.
        *   **Kullanıcı Etkileşimi ve Durum:**
            *   `m_isLock`: Kameranın hareketlerinin kilitli olup olmadığını belirtir.
            *   `m_bDrag`, `m_lMousePosX`, `m_lMousePosY`: Fare ile sürükleme durumu ve pozisyonu.
            *   `m_fPitchSum`, `m_fRollSum`: Sürükleme sırasında biriken pitch ve roll değerleri.
            *   `m_fResistance`: Kamera hareketlerindeki yumuşaklık/direnç katsayısı.
            *   `m_v3AngularVelocity`, `m_v3AngularAcceleration`: Fizik tabanlı kamera hareketi için (yumuşak geçişler).
            *   `m_eCameraState`, `m_eCameraStatePrev`: Mevcut ve önceki kamera durumu.
            *   `m_fTarget_`: Hedefin yerden yüksekliği (`CAMERA_TARGET_STANDARD` gibi).
            *   `m_bProcessTerrainCollision`: Arazi çarpışma kontrolünün aktif olup olmadığı.
        *   **Statik Üyeler:** `CAMERA_MIN_DISTANCE`, `CAMERA_MAX_DISTANCE`.
        *   **Ana Metotlar (Bazıları):**
            *   `Lock()`, `Unlock()`, `IsLock()`: Kamera hareketlerini kilitler/açar.
            *   `Wheel(int nLen)`: Fare tekerleği ile mesafeyi ayarlar (`m_v3AngularVelocity.y` üzerinden).
            *   `BeginDrag()`, `Drag()`, `EndDrag()`, `IsDraging()`: Fare ile kamera döndürme işlemlerini yönetir.
            *   `SetViewParams()`, `SetEye()`, `SetTarget()`, `SetUp()`: Temel kamera parametrelerini ayarlar ve `SetViewMatrix()`'i çağırır.
            *   `SetViewMatrix()`: `m_v3Eye`, `m_v3Target`, `m_v3Up`'a göre `m_matView`, `m_matInverseView`, `m_matBillboard`, `m_fPitch`, `m_fRoll`, `m_fDistance` değerlerini ve çarpışma ışınlarını hesaplar/günceller.
            *   Çeşitli `Move...` metotları (örn: `Move()`, `Zoom()`, `MoveAlongView()`, `MoveFront()`): Kamerayı farklı şekillerde hareket ettirir.
            *   `RotateEyeAroundTarget()`, `RotateEyeAroundPoint()`: Kamerayı bir hedef veya nokta etrafında döndürür.
            *   `Pitch()`, `Roll()`, `SetDistance()`: Pitch, roll ve mesafeyi doğrudan ayarlar.
            *   `Update()`: (Bildirimi var) Kamera hareketlerini, fiziksel etkileri ve çarpışmaları işler.
            *   `ProcessTerrainCollision()`, `ProcessBuildingCollision()`: (Bildirimi var, özel) Çarpışma mantığını uygular.
            *   `SetTargetHeight()`: Hedef yüksekliğini ayarlar.
            *   `SetTerrainCollision()`: Arazi çarpışmasını aktif/pasif yapar.
    *   **`CCameraManager` Sınıfı:**
        *   **Kalıtım:** `public CSingleton<CCameraManager>`.
        *   **`ECameraNum` Enum:** Farklı kamera türleri için sabitler (örn: `DEFAULT_PERSPECTIVE_CAMERA`, `DEFAULT_ORTHO_CAMERA`).
        *   **Temel Üyeler:**
            *   `m_CameraMap` (`TCameraMap` - `std::map<BYTE, CCamera*>`): Kamera ID'lerini `CCamera` işaretçilerine eşler.
            *   `m_pCurrentCamera`, `m_pPreviousCamera`: Aktif ve bir önceki aktif kamera işaretçileri.
        *   **Ana Metotlar:**
            *   `AddCamera()`, `RemoveCamera()`: Kamera ekler/kaldırır.
            *   `GetCurrentCamera()`, `SetCurrentCamera()`: Aktif kamerayı alır/ayarlar.
            *   `ResetToPreviousCamera()`: Bir önceki kameraya döner.
            *   `isCurrentCamera()`: Belirtilen ID'nin aktif kamera olup olmadığını kontrol eder.
            *   `SetTerrainCollision()`: Aktif kameranın arazi çarpışmasını ayarlar.

*   **`Camera.cpp` - Uygulama Detayları:**
    *   **`CCamera` Uygulaması:**
        *   Kurucu: Başlangıç değerlerini ayarlar (direnç, hedef yüksekliği, çarpışma yarıçapları vb.).
        *   `SetResistance()`: Kamera hareket direncini ayarlar.
        *   Fare etkileşim metotları (`Wheel`, `Drag`): Kullanıcı girdisini `m_v3AngularVelocity` (açısal hız) değişikliklerine dönüştürerek kamera hareketlerini tetikler. `Drag` sırasında `m_fPitchSum` ve `m_fRollSum` biriktirilir.
        *   `SetViewParams`, `SetEye`, `SetTarget`, `SetUp`: Bu metotlar kameranın temel yönelimini ayarlar ve ardından `SetViewMatrix`'i çağırarak tüm türetilmiş değerleri (matrisler, açılar, ışınlar) günceller.
        *   `SetViewMatrix()`: En karmaşık metotlardan biridir. Eye, Target, Up vektörlerinden yola çıkarak:
            *   View ve Cross vektörlerini hesaplar.
            *   Pitch ve Roll açılarını (derece cinsinden) hesaplar.
            *   `D3DXMatrixLookAtRH` ile `m_matView`'ı oluşturur.
            *   `D3DXMatrixInverse` ile `m_matInverseView`'ı oluşturur.
            *   `m_matBillboard`'ı `m_matInverseView`'dan türetir (sadece rotasyon kısmı).
            *   Tüm çarpışma ve görünüm ışınlarını (`CRay`) güncel pozisyon ve yönelimlere göre ayarlar.
        *   `Move...` ve `Rotate...` metotları: `m_v3Eye` ve `m_v3Target`'ı uygun şekilde güncelleyip `SetViewMatrix`'i çağırır. `RotateEyeAroundTarget` pitch açısını (-80, 80 derece) sınırlar.
        *   `CalculateRoll()`: `m_v3View` vektörünün XY düzlemindeki açısını kullanarak `m_fRoll` değerini hesaplar.
        *   Kamera durumu (`m_eCameraState`) çarpışmalar veya engellerle karşılaşıldığında güncellenir ve bazı hareketleri kısıtlayabilir (örneğin, `CAMERA_STATE_CANTGOLEFT` durumunda sola dönüş engellenir).
    *   **`CCameraManager` Uygulaması:**
        *   Kurucu: Varsayılan perspektif ve ortografik kameraları (`DEFAULT_PERSPECTIVE_CAMERA`, `DEFAULT_ORTHO_CAMERA`) oluşturur ve `m_CameraMap`'e ekler. Aktif kamerayı perspektif olarak ayarlar.
        *   Yok edici: `m_CameraMap`'teki tüm kamera nesnelerini siler.
        *   `SetCurrentCamera()`: Aktif kamerayı değiştirirken bir önceki kamerayı saklar.
        *   Diğer metotlar `m_CameraMap` üzerinde basit işlemler veya aktif kameraya delege edilen çağrılardır.

*   **Kullanım Alanı:**
    *   `CCamera`, oyun dünyasının nasıl görüneceğini belirleyen temel bileşendir. Oyuncu karakterini takip etmek, serbest kamera modları, sinematik görünümler gibi çeşitli kamera davranışları bu sınıf üzerine inşa edilebilir.
    *   Kullanıcı girdileri (fare, klavye) genellikle bu sınıfın metotlarını çağırarak kamera hareketlerini kontrol eder.
    *   Çarpışma ışınları, kameranın duvarların veya arazinin içinden geçmesini engellemek için kullanılır. `Update` (veya `Process...Collision`) metodunda bu ışınlar test edilir ve gerekirse kamera pozisyonu düzeltilir.
    *   `m_fTarget_` ve ilgili yükseklik oranları (`m_fEyeGroundHeightRatio`, `m_fTargetHeightLimitRatio`), özellikle karakteri takip eden kameralarda, kameranın hedefe (karaktere) göre uygun bir dikey pozisyonda kalmasına yardımcı olur.
    *   `CCameraManager`, farklı oyun durumları veya kullanıcı tercihleri için farklı kamera davranışları arasında geçiş yapmayı kolaylaştırır (örn: normal oyun kamerası, özel bir yetenek için yakın çekim kamerası, harita görünümü için ortografik kamera).
    *   Billboard matrisi, 2D sprite'ların veya partiküllerin her zaman kameraya dönük görünmesini sağlamak için kullanılır.

### `CollisionData.h` ve `CollisionData.cpp` (Çarpışma Veri Yapıları ve Sınıfları)

*   **Amaç:** Bu dosyalar, oyun dünyasındaki statik nesneler için temel geometrik çarpışma şekillerini (Düzlem, Küre, Silindir, Eksenle Hizalı Sınırlayıcı Kutu - AABB, Yönlendirilmiş Sınırlayıcı Kutu - OBB) ve bu şekillerin dinamik bir küre (genellikle karakterleri veya hareketli nesneleri temsil eder) ile etkileşimini yöneten veri yapılarını ve sınıfları tanımlar. `CStaticCollisionData`, genellikle bir `.atr` dosyasından yüklenen ham çarpışma verilerini tutarken, `CBaseCollisionInstance` ve türevleri bu verileri dünya koordinatlarına dönüştürülmüş, sorgulanabilir çarpışma örneklerine çevirir. Bu örnekler, çarpışma tespiti, hareket düzeltmesi ve debug amaçlı render işlemleri için metotlar sunar. Her çarpışma örneği türü için nesne havuzları (`CDynamicPool`) kullanılır.

*   **`CollisionData.h` - Tanımlar ve Bildirimler:**
    *   **Temel Veri Yapıları (Öznitelikler - Attributes):**
        *   `TSphereData`: Dünya koordinatlarında bir kürenin pozisyonunu (`v3Position`) ve yarıçapını (`fRadius`) tutar.
        *   `TPlaneData`: Bir düzlemin dünya pozisyonunu (`v3Position`), normalini (`v3Normal`), dört köşe noktasını (`v3QuadPosition`) ve içe dönük kenar vektörlerini (`v3InsideVector` - genellikle 2D sınırlama testi için) tutar.
        *   `TAABBData`: Eksenle hizalı bir kutunun minimum (`v3Min`) ve maksimum (`v3Max`) dünya koordinatlarını tutar.
        *   `TOBBData`: Yönlendirilmiş bir kutunun lokal eksenlerdeki minimum (`v3Min`) ve maksimum (`v3Max`) noktalarını ve bu lokal eksenleri dünya koordinatlarına çeviren rotasyon matrisini (`matRot`) tutar.
        *   `TCylinderData`: Bir silindirin dünya pozisyonunu (`v3Position` - genellikle taban merkezi), yarıçapını (`fRadius`) ve yüksekliğini (`fHeight`) tutar.
    *   **`ECollisionType` Enum:** Çarpışma şeklinin türünü belirtir (`COLLISION_TYPE_PLANE`, `COLLISION_TYPE_BOX` (kullanımda değil gibi), `COLLISION_TYPE_SPHERE`, `COLLISION_TYPE_CYLINDER`, `COLLISION_TYPE_AABB`, `COLLISION_TYPE_OBB`).
    *   **`CDynamicSphereInstance` Yapısı:**
        *   Hareketli bir çarpışma küresini temsil eder. Bir önceki (`v3LastPosition`) ve mevcut (`v3Position`) dünya pozisyonunu ve yarıçapını (`fRadius`) tutar. Çarpışma testleri bu yapıya karşı yapılır.
    *   **`CStaticCollisionData` Yapısı:**
        *   Bir `.atr` dosyasından okunan tek bir statik çarpışma şeklinin ham verilerini tutar.
        *   `dwType`: `ECollisionType` türünden çarpışma tipi.
        *   `szName[32 + 1]`: Çarpışma parçasının adı.
        *   `v3Position`: Çarpışma şeklinin yerel (local) pozisyonu (ana nesneye göre).
        *   `fDimensions[3]`: Çarpışma şeklinin boyutları (örneğin, küre için yarıçap; kutu için genişlik, yükseklik, derinlik).
        *   `quatRotation`: Çarpışma şeklinin yerel rotasyonu (`D3DXQUATERNION`).
    *   **`CBaseCollisionInstance` Sınıfı (Soyut Temel Sınıf):**
        *   Farklı çarpışma şekli örnekleri için ortak bir arayüz tanımlar.
        *   `virtual void Render(D3DFILLMODE d3dFillMode = D3DFILL_SOLID) = 0`: Çarpışma şeklini debug amaçlı çizer.
        *   `MovementCollisionDynamicSphere(const CDynamicSphereInstance& s) const`: Verilen dinamik kürenin hareketi sırasında bu statik şekille çarpışıp çarpışmadığını test eder.
        *   `CollisionDynamicSphere(const CDynamicSphereInstance& s) const`: Dinamik kürenin mevcut pozisyonunda bu statik şekille çarpışıp çarpışmadığını test eder.
        *   `GetCollisionMovementAdjust(const CDynamicSphereInstance& s) const`: Çarpışma durumunda dinamik kürenin hareketini düzeltmek için gereken itme/kaydırma vektörünü döndürür.
        *   `Destroy()`: Örneği ilişkili nesne havuzuna geri bırakır.
        *   `static CBaseCollisionInstance* BuildCollisionInstance(const CStaticCollisionData* c_pCollisionData, const D3DXMATRIX* pMat)`: Bir `CStaticCollisionData` ve dünya matrisi alarak uygun tipte bir çarpışma örneği (`CSphereCollisionInstance` vb.) oluşturur ve döndürür.
        *   **Korumalı Sanal Metotlar:** `OnMovementCollisionDynamicSphere`, `OnCollisionDynamicSphere`, `OnGetCollisionMovementAdjust`, `OnDestroy`. Bu metotlar türetilmiş sınıflar tarafından kendi şekillerine özel çarpışma mantığını implemente etmek için kullanılır.
    *   **Türetilmiş Çarpışma Örneği Sınıfları:** (`CSphereCollisionInstance`, `CPlaneCollisionInstance`, `CAABBCollisionInstance`, `COBBCollisionInstance`, `CCylinderCollisionInstance`)
        *   Her biri `CBaseCollisionInstance`'dan miras alır.
        *   Kendi geometrik özniteliklerini (örn: `CSphereCollisionInstance` için `TSphereData m_attribute`) tutar.
        *   `GetAttribute()`: Kendi öznitelik verilerine erişim sağlar.
        *   Soyut temel sınıfın sanal metotlarını kendi şekillerine uygun çarpışma testleri ve render işlemleriyle implemente eder.
        *   `OnDestroy()`: Nesneyi ilgili global nesne havuzuna (`gs_sci`, `gs_pci` vb.) geri `Free` eder.

*   **`CollisionData.cpp` - Uygulama Detayları:**
    *   **Global Nesne Havuzları:** Her bir çarpışma örneği türü (`CSphereCollisionInstance` vb.) için bir `CDynamicPool` (örn: `gs_sci`, `gs_cci`) global olarak tanımlanır.
    *   **`DestroyCollisionInstanceSystem()` Fonksiyonu:** Tüm bu global nesne havuzlarını `Destroy()` ederek temizler. Genellikle program sonlanırken çağrılır.
    *   **`CBaseCollisionInstance::BuildCollisionInstance()` Metodu:**
        *   Verilen `CStaticCollisionData`'nın `dwType`'ına göre bir `switch` ifadesi kullanır.
        *   Uygun havuzdan (`gs_sci.Alloc()`, `gs_pci.Alloc()` vb.) yeni bir çarpışma örneği alır.
        *   `CStaticCollisionData`'daki yerel pozisyon, boyutlar ve rotasyonu, verilen dünya matrisi (`pMat`) ile birleştirerek çarpışma şeklinin dünya koordinatlarındaki özniteliklerini (örn: `TSphereData::v3Position`, `TPlaneData::v3QuadPosition`) hesaplar ve doldurur.
        *   Örneğin, `COLLISION_TYPE_PLANE` için:
            *   Lokal dörtgen köşe noktaları oluşturulur.
            *   Bu noktalar, `CStaticCollisionData`'nın rotasyonu, lokal pozisyonu ve verilen dünya matrisiyle çarpılarak dünya koordinatlarına dönüştürülür.
            *   Dönüştürülmüş köşe noktalarından düzlemin normali ve iç kenar vektörleri hesaplanır.
        *   Örneğin, `COLLISION_TYPE_SPHERE` için:
            *   `CStaticCollisionData`'daki lokal pozisyon dünya matrisiyle dönüştürülerek kürenin dünya merkezi bulunur.
            *   Yarıçap doğrudan `CStaticCollisionData`'dan alınır.
    *   **Türetilmiş Sınıfların Metot Uygulamaları:**
        *   **`Render()`:** Genellikle `CScreen` sınıfının (başka bir `EterLib` bileşeni) `RenderSphere`, `RenderBar3d` (düzlem/quad için), `RenderCylinder`, `RenderCube` gibi debug çizim fonksiyonlarını çağırarak ilgili şekli ekranda gösterir.
        *   **`OnDestroy()`:** Kendisini ait olduğu global havuzdan `Free` eder.
        *   **`OnCollisionDynamicSphere()` / `OnMovementCollisionDynamicSphere()`:** Her şekil türü için özel çarpışma testleri içerir:
            *   **Küre vs Küre:** Merkezler arası mesafenin yarıçaplar toplamından küçük olup olmadığını kontrol eder. `OnMovementCollisionDynamicSphere` ayrıca hareket yönünü ve mesafesini de dikkate alarak daha hassas bir test yapmaya çalışır (özellikle hızlı hareket eden nesneler için ara adımları kontrol etme mantığı da içerebilir).
            *   **Düzlem vs Küre:** Kürenin merkezinin düzlemin hangi tarafında olduğunu (normal ile dot product), düzlemin dörtgen sınırları içinde olup olmadığını (iç kenar vektörleri ile dot product) ve düzleme olan mesafesinin kürenin yarıçapından küçük olup olmadığını kontrol eder.
            *   **Silindir vs Küre:** Kürenin Z ekseninde silindirin yükseklik sınırları içinde olup olmadığını, XY düzlemindeki merkezler arası mesafenin yarıçaplar toplamından küçük olup olmadığını test eder. `IntersectLineSegments` kullanarak daha kesin bir hareketli çarpışma testi de yapabilir.
            *   **AABB/OBB vs Küre:** Kürenin merkezini AABB/OBB'nin en yakın noktasına kelepçeleyerek (clamp ederek) bu en yakın nokta ile küre merkezi arasındaki mesafenin küre yarıçapından küçük olup olmadığını kontrol eder. OBB için küre pozisyonu önce OBB'nin lokal koordinat sistemine dönüştürülür.
        *   **`OnGetCollisionMovementAdjust()`:** Çarpışma olduğunda, dinamik kürenin statik şekle girmesini engelleyecek veya onu dışarı itemeyecek bir düzeltme vektörü hesaplamaya çalışır. Genellikle çarpışma normali veya en yakın yüzeye dik bir yönde, kürenin yarıçapı kadar bir itme uygular. Bazı implementasyonlar daha karmaşık kayma (sliding) davranışları da içerebilir (ancak mevcut kodda bu daha basit görünüyor, genellikle `gc_fReduceMove` gibi bir faktörle çarpılmış bir itme vektörü döndürülüyor).

*   **Kullanım Alanı:**
    *   Bir `CGraphicObjectInstance` (veya benzeri bir oyun nesnesi) yüklendiğinde, ilişkili `.atr` dosyasındaki `CStaticCollisionData` listesi okunur.
    *   Nesnenin her bir `CStaticCollisionData`'sı ve nesnenin güncel dünya matrisi kullanılarak `CBaseCollisionInstance::BuildCollisionInstance` ile bir dizi `CBaseCollisionInstance` işaretçisi oluşturulur. Bu örnekler genellikle nesneye ait bir `CCollisionInstanceVector` içinde tutulur.
    *   Oyun döngüsünde, karakter veya diğer hareketli nesneler (bir `CDynamicSphereInstance` ile temsil edilen) bu statik çarpışma örnekleri listesiyle test edilir.
    *   `MovementCollisionDynamicSphere` ile olası bir çarpışma tespit edilirse, `GetCollisionMovementAdjust` ile karakterin pozisyonu düzeltilir, böylece nesnelerin içinden geçmesi engellenir.
    *   Bu sistem, oyun dünyasındaki temel fiziksel etkileşimleri ve katılığı sağlar.

### `ColorTransitionHelper.h` ve `ColorTransitionHelper.cpp` (`CColorTransitionHelper` Sınıfı)

*   **Amaç:** `CColorTransitionHelper` sınıfı, bir rengin (RGBA formatında, her bir bileşen 0.0f ile 1.0f arasında float olarak temsil edilir) bir başlangıç renginden (`m_fSrc...`) bir hedef renge (`m_fDst...`) belirli bir süre (`m_dwDuration`) boyunca doğrusal olarak yumuşak bir geçiş yapmasını sağlar. Bu, zamanla renk değiştiren animasyonlar veya efektler oluşturmak için kullanılır.

*   **`ColorTransitionHelper.h` - Tanımlar ve Bildirimler:**
    *   **`CColorTransitionHelper` Sınıfı:**
        *   Kurucu (`CColorTransitionHelper()`), Yıkıcı (`~CColorTransitionHelper()`).
        *   `Clear(const float& c_rfRed, ..., const float& c_rfAlpha)`: Başlangıç ve hedef renkleri belirtilen renge ayarlar, süreyi sıfırlar ve geçişi durdurur.
        *   `SetSrcColor(const float& c_rfRed, ..., const float& c_rfAlpha)`: Sadece geçişin başlangıç rengini ayarlar.
        *   `SetTransition(const float& c_rfRed, ..., const float& c_rfAlpha, const DWORD& dwDuration)`: Geçişin hedef rengini ve toplam süresini (milisaniye cinsinden) ayarlar.
        *   `GetCurColor() const`: Geçişteki mevcut anlık rengi `D3DCOLOR` (DWORD, ARGB formatında) olarak döndürür.
        *   `StartTransition()`: Geçişi başlatır, başlangıç zamanını kaydeder.
        *   `Update()`: Geçişi günceller. Geçen süreye göre mevcut rengi interpole eder. Geçiş tamamlandıysa `false`, devam ediyorsa `true` döner.
        *   `isTransitionStarted() const`: Geçişin aktif olup olmadığını döndürür.
    *   **Özel Üyeler:**
        *   `m_dwCurColor`: Hesaplanan mevcut anlık renk (`D3DCOLOR`).
        *   `m_dwStartTime`: Geçişin başladığı zaman (`GetCurrentTime()` ile alınır).
        *   `m_dwDuration`: Geçişin toplam süresi.
        *   `m_bTransitionStarted`: Geçişin aktif olup olmadığını belirten bayrak.
        *   `m_fSrcRed`, `m_fSrcGreen`, `m_fSrcBlue`, `m_fSrcAlpha`: Başlangıç renginin float bileşenleri.
        *   `m_fDstRed`, `m_fDstGreen`, `m_fDstBlue`, `m_fDstAlpha`: Hedef renginin float bileşenleri.

*   **`ColorTransitionHelper.cpp` - Uygulama Detayları:**
    *   **Kurucu/Yıkıcı:** Her ikisi de `Clear(0.0f, 0.0f, 0.0f, 0.0f)` çağırarak sınıfı varsayılan (siyah, tamamen saydam) bir duruma getirir.
    *   **`Clear` Metodu:** Tüm renk bileşenlerini (kaynak ve hedef) verilen değerlere eşitler, süreyi ve başlangıç zamanını sıfırlar, `m_dwCurColor`'ı sıfırlar ve `m_bTransitionStarted`'ı `false` yapar.
    *   **`SetSrcColor` Metodu:** Sadece kaynak renk bileşenlerini günceller.
    *   **`SetTransition` Metodu:** Hedef renk bileşenlerini ve geçiş süresini ayarlar.
    *   **`StartTransition` Metodu:** `m_bTransitionStarted`'ı `true` yapar ve `m_dwStartTime`'ı o anki zamanla günceller.
    *   **`Update` Metodu:**
        1.  Mevcut zaman ile başlangıç zamanı arasındaki farkı (`dwElapsedTime`) hesaplar.
        2.  Geçiş yüzdesini (`fpercent = dwElapsedTime / m_dwDuration`) hesaplar. Bu değer 0.0 ile 1.0 arasında sınırlandırılır.
        3.  Kaynak ve hedef renkler arasında `fpercent` oranında doğrusal interpolasyon yaparak anlık RGBA değerlerini (`fCurRed` vb.) hesaplar: `Cur = Src + (Dst - Src) * percent`.
        4.  Hesaplanan float RGBA değerlerini `0-255` aralığına ölçekleyip `D3DCOLOR` formatına (DWORD, ARGB sırası: `(A << 24) | (R << 16) | (G << 8) | B`) dönüştürerek `m_dwCurColor`'a atar.
        5.  Eğer `fpercent` 1.0 ise (yani süre dolmuşsa) VE mevcut hesaplanan renk bileşenleri hedef renk bileşenleriyle tam olarak eşleşiyorsa, `m_bTransitionStarted`'ı `false` yapar ve `false` döner (geçiş bitti).
        6.  Aksi takdirde `true` döner (geçiş devam ediyor).
    *   **`GetCurColor` Metodu:** `m_dwCurColor` üyesinin bir referansını döndürür.

*   **Kullanım Alanı:**
    *   Kullanıcı arayüzü elemanlarının renklerinin zamanla değişmesi (örneğin, bir düğmenin üzerine gelindiğinde renginin yavaşça açılması veya bir uyarının renginin yanıp sönmesi).
    *   Oyun içi efektlerde renk geçişleri (örneğin, bir nesnenin hasar aldığında kısa bir süre kırmızıya dönüp sonra normale dönmesi).
    *   Sahne geçişlerinde ekranın yavaşça siyaha veya beyaza kararması/açılması (fade in/out).
    *   Kullanımı genellikle şu şekildedir: Bir `CColorTransitionHelper` nesnesi oluşturulur, `SetSrcColor` ve `SetTransition` ile başlangıç/hedef renkler ve süre ayarlanır, `StartTransition` ile geçiş başlatılır ve her frame `Update` çağrılarak geçiş ilerletilir, `GetCurColor` ile de anlık renk değeri alınıp ilgili nesneye uygulanır. 

### `CullingManager.h` ve `CullingManager.cpp` (`CCullingManager` Sınıfı)

*   **Amaç:** `CCullingManager` sınıfı, bir singleton olarak, oyun dünyasındaki `CGraphicObjectInstance` nesnelerinin görünürlüklerini optimize etmek için bir culling (eleme) sistemi sağlar. Temel olarak, kameranın görüş alanı (frustum) içinde olmayan veya belirli sorgu kriterlerini (mesafe, ışın kesişimi) karşılamayan nesnelerin render edilmesini engelleyerek performansı artırır. Bu işlem için `SphereLib` adlı harici bir kütüphaneyi ve onun `SpherePackFactory` bileşenini kullanır.

*   **`CullingManager.h` - Tanımlar ve Bildirimler:**
    *   **`RangeTester<T>` Template Yapısı:**
        *   `SpherePackCallback`'ten miras alır.
        *   Genel amaçlı bir callback sarmalayıcısıdır. Kullanıcının sağladığı bir fonksiyon nesnesini (`T* f`) alır.
        *   `RayTraceCallback`, `RangeTestCallback`, `PointTest2dCallback` gibi kendi callback metotları içinde, eğer bir `SpherePack` (nesne) test kriterlerini karşılarsa, bu kullanıcı fonksiyonunu (`(*f)((CGraphicObjectInstance*)sphere->GetUserData())`) çağırır.
    *   **`CCullingManager` Sınıfı:**
        *   **Kalıtım:** `public CSingleton<CCullingManager>`, `public SpherePackCallback`, `private CScreen`.
            *   `SpherePackCallback`: `SpherePackFactory` tarafından tetiklenen culling olaylarını (görünürlük, ışın kesişimi vb.) işlemek için gerekli callback metotlarını implemente eder.
            *   `CScreen` (private): Görüş frustumunu oluşturmak ve güncellemek için `CScreen`'in matris ve frustum yönetimi fonksiyonlarına erişir.
        *   **`CullingHandle` Typedef'i:** `SpherePack*` için bir takma addır. Bir nesnenin culling sistemindeki temsilcisidir.
        *   **`TRangeList` Typedef'i:** `std::vector<CGraphicObjectInstance*>` için bir takma addır. Mesafe ve ışın sorgularının sonuçlarını tutmak için kullanılır.
        *   **Ana Metotlar:**
            *   `Register(CGraphicObjectInstance* obj)`: Bir `CGraphicObjectInstance`'ı culling sistemine kaydeder. Nesnenin sınırlayıcı küresini alır ve `m_Factory->AddSphere_` ile `SpherePackFactory`'ye ekler. Bir `CullingHandle` döndürür.
            *   `Unregister(CullingHandle h)`: Verilen handle'a sahip nesneyi culling sisteminden çıkarır (`m_Factory->Remove`).
            *   `Reset()`: `m_Factory->Reset()` çağırarak culling yapısını sıfırlar.
            *   `Update()`: `m_Factory->Process()` çağırır. Bu, `SpherePackFactory`'nin iç uzamsal veri yapısını (eğer nesneler hareket ettiyse) güncellemesini tetikleyebilir.
            *   **Sorgu Metotları:**
                *   `FindRange(const Vector3d& p, float radius)`: Verilen bir nokta (`p`) ve yarıçap (`radius`) içindeki tüm nesneleri bulur. Sonuçları `m_list`'e yazar. `RangeTestCallback` bu sırada tetiklenir.
                *   `FindRay(const Vector3d& p1, const Vector3d& dir)`: Verilen bir ışınla (`p1` başlangıç, `dir` yön) kesişen nesneleri bulur. Sonuçları `m_list`'e yazar.
                *   `FindRayDistance(const Vector3d& p1, const Vector3d& dir, float distance)`: `FindRay` gibidir, ancak belirtilen bir maksimum mesafe (`distance`) içindeki kesişimleri arar.
                *   `RangeTest(const Vector3d& p, float radius, SpherePackCallback* callback)`: Kullanıcının kendi `SpherePackCallback` implementasyonunu kullanarak bir mesafe testi yapmasını sağlar.
                *   `PointTest2d(const Vector3d& p, SpherePackCallback* callback)`: 2D nokta testi yapar.
            *   **`ForIn...` Template Metotları:** (`ForInRange2d`, `ForInRange`, `ForInRay`, `ForInRayDistance`)
                *   `RangeTester`'ı kullanarak, culling sorgularının sonuçları üzerinde kullanıcı tanımlı bir fonksiyonu (functor/lambda) çalıştırmak için kolay bir yol sunarlar.
        *   **Callback Metotları (`SpherePackCallback` Arayüzü):**
            *   `RayTraceCallback(...)`: `m_Factory->RayTrace` tarafından çağrılır. Işın bir nesneyle kesişirse ve mesafe kriterini (`m_RayFarDistance`) karşılıyorsa, nesneyi (`(CGraphicObjectInstance*)sphere->GetUserData()`) `m_list`'e ekler.
            *   `VisibilityCallback(...)`: `m_Factory->FrustumTest` tarafından çağrılır. Nesnenin frustum'a göre durumunu (`state`: `VS_OUTSIDE`, `VS_PARTIAL`, `VS_INSIDE`) alır.
                *   Eğer `VS_OUTSIDE` ise, `pInstance->Hide()` çağrılarak nesne gizlenir.
                *   Aksi takdirde (`VS_PARTIAL` veya `VS_INSIDE`), `pInstance->Show()` çağrılarak nesne gösterilir.
            *   `RangeTestCallback(...)`: `m_Factory->RangeTest` tarafından çağrılır. Nesne `VS_OUTSIDE` değilse, `m_list`'e eklenir.
    *   **Korumalı Üyeler:**
        *   `m_list` (`TRangeList`): Sorgu sonuçlarını tutan vektör.
        *   `m_RayFarDistance`: Işın testleri için maksimum kesişim mesafesi.
        *   `m_Factory` (`SpherePackFactory*`): `SphereLib`'den gelen, asıl uzamsal veri yapısını ve culling algoritmalarını içeren fabrika nesnesi.

*   **`CullingManager.cpp` - Uygulama Detayları:**
    *   **Kurucu:** `SpherePackFactory`'yi belirli parametrelerle (maksimum nesne sayısı, kök yarıçapı, yaprak yarıçapı, ekstra yarıçap) oluşturur. Bu parametreler, `SpherePackFactory`'nin içindeki uzamsal hiyerarşinin (muhtemelen bir tür octree) yapısını ve verimliliğini etkiler.
    *   **Yok Edici:** `m_Factory` nesnesini siler.
    *   **`Register` / `Unregister`:** `CGraphicObjectInstance`'dan sınırlayıcı küre bilgilerini alır (`GetBoundingSphere`) ve `SpherePackFactory`'nin ilgili metotlarına iletir. `SpherePack` nesnesinin `UserData` alanına `CGraphicObjectInstance` işaretçisi kaydedilir.
    *   **Callback Uygulamaları:** `VisibilityCallback`, nesneleri `Show()` veya `Hide()` yaparak doğrudan görünürlüklerini yönetir. Diğer callback'ler genellikle sonuçları `m_list`'e toplar.
    *   `COUNT_SHOWING_SPHERE` ve `SPHERELIB_STRICT` gibi `#ifdef` blokları, culling işlemleri sırasında debug amaçlı loglama ve sayım yapmak için kullanılır.

*   **Kullanım Alanı:**
    *   `CCullingManager`, oyunun ana döngüsünde (`Process` metodu aracılığıyla) her frame'de çalışarak görünür nesne setini belirler.
    *   Render edilecek `CGraphicObjectInstance`'lar culling sistemine `Register` edilir.
    *   Frustum culling, kameranın görmediği nesnelerin çizilmesini engelleyerek GPU yükünü azaltır.
    *   `FindRange` ve `FindRay` gibi sorgular, oyun mantığı içinde belirli bir alandaki nesneleri bulmak (örneğin, patlama etkileri için) veya fare ile nesne seçimi (picking) yapmak için kullanılabilir.
    *   `SpherePackFactory`'nin kullanımı, çok sayıda nesne arasında verimli culling testleri yapılmasını sağlar. 

### `Decal.h` ve `Decal.cpp` (`CDecal` Sınıfı)

*   **Amaç:** `CDecal` sınıfı, 3D bir yüzeyin üzerine yansıtılan veya "yapıştırılan" bir 2D resim (çıkartma) oluşturmak için temel altyapıyı sağlar. Bu, genellikle kan izleri, kurşun delikleri, patlama yanıkları veya zemine yansıtılan özel işaretler gibi efektler için kullanılır. Bir decal, genellikle bir merkez noktası, bir normal (yönelim), bir tanjant (rotasyon için) ve boyutları (genişlik, yükseklik, derinlik) ile tanımlanır. Bu parametreler, decal'in 3D dünyada nasıl konumlanacağını ve hangi mesh yüzeyleriyle kesişeceğini belirleyen 6 adet kırpma düzlemi (clipping plane) oluşturmak için kullanılır.

*   **`Decal.h` - Tanımlar ve Bildirimler:**
    *   **`MAX_DECAL_VERTICES` Sabiti:** Bir decal için izin verilen maksimum vertex sayısı (256).
    *   **`CDecal` Sınıfı:**
        *   Kurucu (`CDecal()`), Yıkıcı (`virtual ~CDecal()`).
        *   `Clear()`: Decal'i temizler, tüm vertex/index verilerini ve kırpma düzlemlerini sıfırlar.
        *   `virtual void Make(D3DXVECTOR3 v3Center, D3DXVECTOR3 v3Normal, D3DXVECTOR3 v3Tangent, float fWidth, float fHeight, float fDepth) = 0`: Soyut bir metottur. Türetilmiş sınıflar, bu metodu implemente ederek verilen parametrelere göre decal'in 6 kırpma düzlemini (`m_v4LeftPlane` vb.) oluşturmalıdır.
        *   `virtual void Render()`: Oluşturulan decal geometrisini (vertex ve index verilerini kullanarak) çizer.
    *   **Korumalı Üyeler:**
        *   `m_v3Center`, `m_v3Normal`: Decal'in merkezini ve normalini tutar (genellikle `Make` içinde ayarlanır).
        *   `m_v4LeftPlane`, `m_v4RightPlane`, `m_v4BottomPlane`, `m_v4TopPlane`, `m_v4FrontPlane`, `m_v4BackPlane`: Decal'in sınırlayıcı kutusunu oluşturan altı adet `D3DXPLANE`.
        *   `m_dwVertexCount`, `m_dwPrimitiveCount`: Oluşturulan decal mesh'indeki toplam vertex ve primitive (üçgen) sayısı.
        *   **`TTRIANGLEFANSTRUCT` Yapısı:** Bir üçgen yelpazesini (triangle fan) çizmek için gerekli bilgileri tutar (minimum index, vertex sayısı, primitive sayısı, vertex buffer offset'i).
        *   `m_TriangleFanStructVector` (`std::vector<TTRIANGLEFANSTRUCT>`): Decal birden fazla ayrı üçgen yelpazesinden oluşuyorsa, her birini saklar.
        *   `m_Vertices[MAX_DECAL_VERTICES]` (`TPDTVertex` dizisi): Decal'in vertex verilerini (pozisyon, renk, doku koordinatı) tutar. (Not: Doku koordinatları bu temel sınıfta doğrudan atanmıyor gibi, bu genellikle türetilmiş sınıflarda veya `Render` sırasında materyal ile ayarlanır.)
        *   `m_Indices[MAX_DECAL_VERTICES]` (`WORD` dizisi): Decal'in index verilerini tutar (üçgen yelpazeleri için).
        *   `m_cfDecalEpsilon`: Kırpma ve normal testlerinde kullanılan küçük bir tolerans değeri (varsayılan 0.25f).
    *   **Korumalı Metotlar:**
        *   `AddPolygon(DWORD dwAddCount, const D3DXVECTOR3* c_pv3Vertex, const D3DXVECTOR3* c_pv3Normal)`: Kırpma sonucu elde edilen bir poligonu (`c_pv3Vertex`) decal'in vertex ve index listesine bir üçgen yelpazesi olarak ekler.
        *   `ClipMesh(DWORD dwPrimitiveCount, const D3DXVECTOR3* c_pv3Vertex, const D3DXVECTOR3* c_pv3Normal)`: Verilen bir mesh'in üçgenlerini (`c_pv3Vertex`, `c_pv3Normal`) decal'in 6 kırpma düzlemine göre kırpar. Kırpılan ve decal normaliyle uyumlu olan poligonları `AddPolygon` ile ekler.
        *   `ClipPolygon(DWORD dwVertexCount, ..., D3DXVECTOR3* c_pv3NewNormal) const`: Bir poligonu decal'in 6 kırpma düzlemine göre (sırayla `ClipPolygonAgainstPlane` çağırarak) kırpar.
        *   `static DWORD ClipPolygonAgainstPlane(const D3DXPLANE& v4Plane, ...)`: Bir poligonu tek bir düzleme göre Sutherland-Hodgman algoritmasına benzer bir yöntemle kırpar. Düzlemin negatif tarafında kalan kısımları atar, kesişen kenarları yeniden hesaplar ve pozitif tarafta kalan veya kesişim sonucu oluşan yeni vertexleri `c_pv3NewVertex`'e yazar.

*   **`Decal.cpp` - Uygulama Detayları:**
    *   **`Clear` Metodu:** Tüm üyeleri varsayılan (genellikle sıfır) değerlere ayarlar.
    *   **`ClipMesh` Metodu:**
        1.  Giriş olarak bir mesh'in üçgenlerini (vertex ve normal listesi) alır.
        2.  Her bir üçgen için:
            *   Üçgenin normali ile decal'in ana normali (`m_v3Normal`) arasındaki açıyı kontrol eder (dot product). Eğer üçgen, decal'e yeterince "yüzeyden bakmıyorsa" (yani normalleri zıt veya çok farklı yönlerdeyse, `m_cfDecalEpsilon` toleransıyla belirlenir), bu üçgen atlanır. Bu, decal'in sadece yönelimine uygun yüzeylere yansıtılmasını sağlar.
            *   Eğer üçgenin normali uygunsa, `ClipPolygon` metodu çağrılarak bu üçgen decal'in 6 kırpma düzlemine göre kırpılır.
            *   Kırpma sonucu en az 3 vertex'ten oluşan bir poligon elde edilirse, `AddPolygon` ile bu poligon decal geometrisine eklenir.
    *   **`AddPolygon` Metodu:**
        *   Kırpılmış bir poligonun vertexlerini alır.
        *   Bu vertexleri kullanarak bir `TTRIANGLEFANSTRUCT` oluşturur ve `m_TriangleFanStructVector`'e ekler.
        *   Vertexleri `m_Vertices` dizisine kopyalar. Bu aşamada vertex renkleri (`diffuse`) varsayılan olarak `0xFFFFFFFF` (beyaz) ayarlanır. Alfa değerleri veya özel doku koordinatları bu temel sınıfta doğrudan hesaplanmıyor; bu genellikle türetilmiş sınıfların sorumluluğundadır veya render sırasında bir materyal ile yönetilir.
        *   Üçgen yelpazesi için gerekli indeksleri `m_Indices` dizisine oluşturur.
    *   **`ClipPolygon` Metodu:** Bir poligonu sırayla decal'in 6 kırpma düzlemine (`m_v4LeftPlane`'den başlayarak `m_v4FrontPlane`'e kadar) karşı `ClipPolygonAgainstPlane` ile kırpar. Her adımdan çıkan poligon bir sonraki düzlem için giriş olur.
    *   **`ClipPolygonAgainstPlane` Metodu (Statik):**
        *   Sutherland-Hodgman poligon kırpma algoritmasının bir varyasyonunu uygular.
        *   Poligonun her bir vertex'ini düzlemin pozitif mi negatif mi tarafında olduğunu sınıflandırır.
        *   Poligonun kenarlarını dolaşırken, düzlemi kesen kenarlar için kesişim noktalarını hesaplar.
        *   Düzlemin pozitif tarafında kalan orijinal vertexleri ve yeni hesaplanan kesişim vertexlerini kullanarak kırpılmış yeni poligonu oluşturur.
    *   **`Render` Metodu:**
        *   Basit bir dünya matrisi (identity) ayarlar.
        *   Uygun vertex formatını (`D3DFVF_XYZ | D3DFVF_DIFFUSE | D3DFVF_TEX1`) ayarlar.
        *   `m_TriangleFanStructVector`'deki her bir üçgen yelpazesi için `STATEMANAGER.DrawIndexedPrimitiveUP` çağırarak `m_Vertices` ve `m_Indices` dizilerindeki veriyi kullanarak decal'i çizer.

*   **Kullanım Alanı:**
    *   Bir decal nesnesi oluşturulduktan sonra, `Make` metodu çağrılarak decal'in 3D dünyadaki konumu, yönelimi ve boyutları belirlenir. Bu, kırpma düzlemlerini oluşturur.
    *   Daha sonra, decal'in yansıtılacağı hedef yüzeylerin (örneğin, bir karakter modeli veya bir duvar) vertex ve normal verileri `ClipMesh` metoduna verilir.
    *   `ClipMesh`, bu yüzeyleri decal'in sınırlarına göre kırparak decal geometrisini oluşturur.
    *   Son olarak, `Render` metodu çağrılarak oluşturulan bu decal geometrisi ekranda çizilir. Genellikle decal'e bir doku (texture) atanarak görsel efekt elde edilir; bu doku atama işlemi `Render` içinde veya bir materyal sistemi aracılığıyla yapılabilir.
    *   Sınıf, `MAX_DECAL_VERTICES` ile sınırlı sayıda vertex üretebilir, bu nedenle çok karmaşık yüzeylere veya çok büyük decal'lere uygulandığında geometrinin bir kısmı kaybolabilir.
    *   `CDecalManager` (koddaki yorum satırlarında kalmış) gibi bir yönetici sınıfı, birden fazla decal'i yönetmek, güncellemek ve render etmek için kullanılabilir.

### `DibBar.h` ve `DibBar.cpp` (`CDibBar` Sınıfı)

*   **Amaç:** `CDibBar` sınıfı, potansiyel olarak büyük bir `CGraphicDib` (CPU'da tutulan bitmap) nesnesini, Direct3D doku boyutu kısıtlamalarına (genellikle 2'nin kuvvetleri ve maksimum bir boyut, örneğin 256x256) uyacak şekilde daha küçük, yönetilebilir `CBlockTexture` parçalarına bölerek yönetir. Bu, büyük 2D resimlerin (örneğin, UI arka planları, kaydırma çubukları, büyük ikon setleri) verimli bir şekilde GPU'ya yüklenmesini ve render edilmesini sağlar.

*   **`DibBar.h` - Tanımlar ve Bildirimler:**
    *   **`CDibBar` Sınıfı:**
        *   Kurucu (`CDibBar()`), Yıkıcı (`virtual ~CDibBar()`).
        *   `Create(HDC hdc, DWORD dwWidth, DWORD dwHeight)`: Belirtilen genişlik ve yükseklikte bir `CGraphicDib` (`m_dib`) oluşturur. Ardından `__BuildTextureBlockList` ile bu DIB'i `CBlockTexture` parçalarına böler.
        *   `Invalidate()`: Tüm `CBlockTexture` parçalarının tüm alanını geçersiz kılar, böylece DIB üzerindeki değişiklikler dokulara yansıtılır.
        *   `SetClipRect(const RECT& c_rRect)`: Tüm `CBlockTexture` parçaları için bir kırpma dikdörtgeni ayarlar.
        *   `ClearBar()`: `m_dib`'in içeriğini sıfırlar (siyah yapar) ve ardından `Invalidate()` çağırarak bu değişikliği dokulara yansıtır.
        *   `Render(int ix, int iy)`: Tüm `CBlockTexture` parçalarını belirtilen ekran koordinatlarına göre sırayla çizer.
    *   **Korumalı Metotlar:**
        *   `__NearTextureSize(DWORD dwSize)`: Verilen bir boyuta en yakın veya eşit olan bir sonraki 2'nin kuvveti değerini bulur (örneğin, 100 girerse 128 döndürür).
        *   `__DivideTextureSize(DWORD dwSize, DWORD dwMax, DWORD* pdwxStep, DWORD* pdwxCount, DWORD* pdwxRest)`: Verilen bir boyutu (`dwSize`), maksimum adım boyutuna (`dwMax`) göre kaç tam adım (`pdwxCount`) ve kalan parça (`pdwxRest`) olduğuna böler.
        *   `__BuildTextureBlock(DWORD dwxPos, DWORD dwyPos, DWORD dwImageWidth, DWORD dwImageHeight, DWORD dwTextureWidth, DWORD dwTextureHeight)`: Belirtilen DIB pozisyonundan (`dwxPos`, `dwyPos`) ve boyutlarından (`dwImageWidth`, `dwImageHeight`) bir `CBlockTexture` oluşturur. Oluşturulacak doku boyutları (`dwTextureWidth`, `dwTextureHeight`) da verilir.
        *   `__BuildTextureBlockList(DWORD dwWidth, DWORD dwHeight, DWORD dwMax = 256)`: `m_dib`'i, maksimum doku boyutu `dwMax` olacak şekilde `CBlockTexture` parçalarına böler ve bu parçaları `m_kVec_pkBlockTexture` listesine ekler.
        *   `virtual void OnCreate()`: `Create` metodu sonunda çağrılan sanal bir metottur. Türetilmiş sınıflar ek başlatma işlemleri için bunu override edebilir.
    *   **Korumalı Üyeler:**
        *   `m_dib` (`CGraphicDib`): Ana resim verilerini CPU'da tutan DIB nesnesi.
        *   `m_kVec_pkBlockTexture` (`std::vector<CBlockTexture*>`): Oluşturulan `CBlockTexture` parçalarının işaretçilerini tutan vektör.
        *   `m_dwWidth`, `m_dwHeight`: `CDibBar`'ın toplam genişliği ve yüksekliği.

*   **`DibBar.cpp` - Uygulama Detayları:**
    *   **Kurucu/Yıkıcı:** Kurucu boştur. Yıkıcı, `m_kVec_pkBlockTexture` içindeki tüm `CBlockTexture` nesnelerini `stl_wipe` ile siler.
    *   **`Create` Metodu:**
        *   `m_dib.Create()` ile belirtilen boyutlarda bir DIB oluşturur.
        *   Toplam genişlik ve yüksekliği saklar.
        *   `__BuildTextureBlockList` metodunu çağırarak DIB'i parçalara ayırıp `CBlockTexture` listesini oluşturur.
        *   `OnCreate()` sanal metodunu çağırır.
    *   **`__NearTextureSize` Metodu:** Bir sayının 2'nin kuvveti olup olmadığını kontrol eder. Değilse, kendisinden büyük veya eşit olan en küçük 2'nin kuvveti değerini bulana kadar 2 ile çarparak ilerler.
    *   **`__DivideTextureSize` Metodu:** Basit bir bölme işlemi yaparak, bir boyutun maksimum bir adıma kaç kez tam bölündüğünü ve kalanı hesaplar.
    *   **`__BuildTextureBlock` Metodu:** Parametre olarak aldığı DIB bölgesi ve hedef doku boyutları ile yeni bir `CBlockTexture` nesnesi oluşturur ve onun `Create` metodunu çağırır. Başarısız olursa `NULL` döner.
    *   **`__BuildTextureBlockList` Metodu:**
        1.  `CDibBar`'ın toplam genişliğini ve yüksekliğini, verilen maksimum doku boyutu (`dwMax`, varsayılan 256) ile böler (`__DivideTextureSize` kullanarak) ve kaç tane tam boyutlu blok (`dwxCount`, `dwyCount`) ve ne kadar artık kısım (`dwxRest`, `dwyRest`) olduğunu hesaplar.
        2.  Artık kısımların doku boyutlarını `__NearTextureSize` ile 2'nin en yakın kuvvetine ayarlar (`dwxTexRest`, `dwyTexRest`).
        3.  İç içe döngülerle tam boyutlu blokları (`dwMax` x `dwMax`) oluşturur. Her biri için `__BuildTextureBlock` çağırılır ve başarılı olursa `m_kVec_pkBlockTexture`'a eklenir.
        4.  Genişlik ve yükseklik için artık kısımlar varsa, bu artık kısımlara karşılık gelen (potansiyel olarak daha küçük boyutlu) `CBlockTexture`'ları da oluşturur (örneğin, sağ kenardaki artık bloklar, alt kenardaki artık bloklar ve sağ alt köşedeki artık blok).
    *   **`Invalidate` Metodu:** `m_dwWidth` ve `m_dwHeight` boyutlarında bir `RECT` oluşturur ve `m_kVec_pkBlockTexture` içindeki her `CBlockTexture`'ın `InvalidateRect` metodunu bu tam `RECT` ile çağırır. Bu, tüm DIB'in değiştiğini varsayarak tüm blokların güncellenmesini tetikler.
    *   **`SetClipRect` Metodu:** `m_kVec_pkBlockTexture` içindeki her `CBlockTexture`'ın `SetClipRect` metodunu çağırır.
    *   **`ClearBar` Metodu:** `m_dib.GetPointer()` ile DIB'in bellek alanına erişir ve `memset` ile tüm pikselleri sıfırlar. Ardından `Invalidate()` çağırır.
    *   **`Render` Metodu:** `m_kVec_pkBlockTexture` içindeki her `CBlockTexture`'ın `Render` metodunu aynı `ix`, `iy` koordinatlarıyla çağırır. Bu, parçaların doğru pozisyonlarda çizilerek orijinal büyük resmi oluşturmasını sağlar.

*   **Kullanım Alanı:**
    *   Kullanıcı arayüzünde çok büyük arka plan resimleri, geniş bilgi panelleri veya uzun listeler gibi tek bir Direct3D dokusuna sığmayabilecek 2D grafiklerin yönetimi için idealdir.
    *   Oyun içi haritaların veya büyük dünya görünümlerinin 2D temsilleri için kullanılabilir (ancak genellikle bu tür amaçlar için daha özel tile-tabanlı sistemler tercih edilir).
    *   `CGraphicDib` üzerinde yapılan değişiklikler (örneğin, metin çizimi, dinamik grafik güncellemeleri) `Invalidate()` çağrısı ile GPU tarafındaki dokulara yansıtılabilir.
    *   Temelde, büyük bir DIB'i, donanım kısıtlamalarına uygun daha küçük doku parçalarına bölerek soyutlar ve bu parçaların bir bütün olarak render edilmesini ve yönetilmesini kolaylaştırır.