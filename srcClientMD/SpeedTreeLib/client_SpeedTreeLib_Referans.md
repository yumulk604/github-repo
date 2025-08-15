# SpeedTreeLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `SpeedTreeLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. 

## Dosya Bazlı Detaylandırma

### `BoundaryShapeManager.h` ve `BoundaryShapeManager.cpp` (`CBoundaryShapeManager` Sınıfı)

*   **Amaç:** `CBoundaryShapeManager` sınıfı, dışbükey olmayan (non-convex) çokgenlerden oluşabilen karmaşık 2D sınır şekillerini yönetmek için kullanılır. Bu şekiller, genellikle `.bsf` (Boundary Shape File) uzantılı dosyalardan yüklenir. Sınıf, belirli bir (X, Y) noktasının bu yüklenmiş sınırlardan herhangi birinin içinde olup olmadığını kontrol etme ve bu sınırlar içinde rastgele bir nokta üretme yeteneği sağlar. Bu, genellikle SpeedTree ağaçlarının veya diğer bitki örtüsü türlerinin belirli alanlara yerleştirilmesini kısıtlamak veya belirli bölgelerde yoğunlaştırmak için kullanılır.

*   **`BoundaryShapeManager.h` - Temel Tanımlar ve Bildirimler:**
    *   **`SPoint` Yapısı:**
        *   `float m_afData[3]`: Bir noktanın 3D koordinatlarını (X, Y, Z) tutar.
        *   `float& operator[](int i)`: Koordinatlara dizi erişimi (örn: `point[0]` X'i verir) sağlar.
    *   **`SBoundaryShape` Yapısı:**
        *   `m_vContours` (std::vector< std::vector<SPoint> >): Bir sınır şeklini oluşturan konturların (kapalı çokgenlerin) bir listesidir. Her kontur, `SPoint` vektörüdür. Bir şekil, birden fazla ayrı konturdan (örn: bir göl ve içindeki adalar) oluşabilir.
        *   `m_afMin[3]`, `m_afMax[3]` (float): Şeklin X, Y, Z eksenlerindeki minimum ve maksimum sınırlarını (AABB - Axis-Aligned Bounding Box) tutar.
    *   **`CBoundaryShapeManager` Sınıfı:**
        *   **Genel Metotlar:**
            *   `CBoundaryShapeManager()`: Yapıcı.
            *   `~CBoundaryShapeManager()`: Yıkıcı.
            *   `LoadBsfFile(const char* pFilename)`: Belirtilen `.bsf` dosyasından sınır şekillerini yükler.
            *   `PointInside(float fX, float fY)`: Verilen (X, Y) noktasının yüklenmiş sınırlardan herhangi birinin içinde olup olmadığını kontrol eder.
            *   `RandomPoint(float& fX, float& fY)`: Yüklenmiş sınırlardan birinin içinde rastgele bir (X, Y) noktası üretir ve `fX`, `fY` referanslarına atar.
            *   `GetCurrentError()`: En son oluşan hata mesajını string olarak döndürür.
        *   **Özel (Private) Metot:**
            *   `PointInShape(SBoundaryShape& sShape, float fX, float fY)`: Verilen (X, Y) noktasının belirli bir `SBoundaryShape` içinde olup olmadığını kontrol eder.
        *   **Özel (Private) Üyeler:**
            *   `m_vBoundaries` (std::vector<SBoundaryShape>): Yüklenmiş tüm sınır şekillerini tutan bir vektör.
            *   `m_strCurrentError` (std::string): En son hata mesajını saklar.

*   **`BoundaryShapeManager.cpp` - Uygulama Detayları:**
    *   **`LoadBsfFile(const char* pszFilename)`:**
        *   Belirtilen `.bsf` dosyasını binary modda (`"rb"`) açar.
        *   Dosyadan sırasıyla okur:
            1.  Toplam sınır şekli sayısı (`nNumBoundaries`).
            2.  Her bir sınır şekli için:
                *   Kontur sayısı (`nNumContours`).
                *   Her bir kontur için:
                    *   Nokta sayısı (`nNumPoints`).
                    *   Her bir nokta için 3 float (`SPoint.m_afData`).
        *   Noktaları okurken, her bir `SBoundaryShape` için `m_afMin` ve `m_afMax` AABB değerlerini günceller.
        *   Okunan şekilleri `m_vBoundaries` vektörüne ekler.
        *   Dosya okuma hatalarını veya format hatalarını `m_strCurrentError`'a kaydeder ve `false` döndürür.
    *   **`PointInside(float fX, float fY)`:**
        *   `m_vBoundaries` vektöründeki her bir `SBoundaryShape` için `PointInShape()` metodunu çağırır.
        *   Eğer nokta herhangi bir şeklin içindeyse `true` döndürür, aksi halde `false`.
    *   **`PointInShape(SBoundaryShape& sShape, float fX, float fY)`:**
        *   Belirli bir `sShape` içindeki her bir kontur (`m_vContours[k]`) için "Ray Casting" (Işın Fırlatma) algoritmasını uygular:
            *   Verilen (X, Y) noktasından yatay bir ışın fırlatılır (genellikle pozitif X yönünde).
            *   Bu ışının kontur kenarlarıyla kaç kez kesiştiği sayılır.
            *   Eğer kesişme sayısı tek ise nokta içeridedir, çift ise dışarıdadır.
            *   Koddaki implementasyon, standart bir "point-in-polygon" testinin yaygın bir formülasyonunu kullanır. (Not: Koddaki `(sShape.m_vContours[k][i][0] - sShape.m_vContours[k][i][0])` ifadesi `0` döndürecektir, bu muhtemelen bir yazım hatasıdır ve `(sShape.m_vContours[k][j][0] - sShape.m_vContours[k][i][0])` olmalıdır. Bu düzeltilmezse, algoritma dikey kenarlar için hatalı sonuç verebilir veya sıfıra bölme hatası oluşabilir.)
    *   **`RandomPoint(float& fX, float& fY)`:**
        *   Eğer yüklü sınır şekli yoksa `false` döndürür.
        *   `m_vBoundaries` içinden rastgele bir sınır şekli seçer.
        *   Seçilen şeklin AABB'si (`sShape.m_afMin`, `sShape.m_afMax`) içinde rastgele bir (X, Y) noktası üretir (`frandom` fonksiyonu kullanılır, bu `EterBase`'den geliyor olabilir).
        *   Üretilen bu noktanın `PointInShape()` ile gerçekten şeklin içinde olup olmadığını kontrol eder.
        *   Nokta içerideyse `true` döner, ancak bu metodun mevcut haliyle, nokta dışarıda kalsa bile `false` yerine `true` dönebilir (eğer ilk denemede içerde değilse tekrar deneme mekanizması yok). Daha sağlam bir implementasyon, nokta şeklin içinde bulunana kadar veya belirli bir deneme sayısına ulaşılana kadar rastgele nokta üretmeye devam etmelidir.

*   **Kullanım Amacı:**
    *   Belirli 2D alanları tanımlamak ve bu alanlar içinde nokta sorgulamaları yapmak için kullanılır.
    *   SpeedTree gibi bitki örtüsü sistemlerinde, ağaçların veya çimenlerin sadece belirli coğrafi sınırlar içine yerleştirilmesini sağlamak.
    *   Oyuncuların veya NPC'lerin giremeyeceği yasaklı bölgeler tanımlamak.
    *   Belirli efektlerin veya olayların sadece tanımlanmış alanlarda tetiklenmesini sağlamak.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (SpeedTreeLib için)
    *   `windows.h` (Doğrudan kullanımı görünmüyor, muhtemelen `FILE` işlemleri veya eski bir bağımlılık için)
    *   `../EterBase/Random.h`: `random_range` ve `frandom` fonksiyonları için.
    *   Standart C++ kütüphaneleri (`vector`, `string`, `cstdio` için `FILE`).

### `Constants.h` (SpeedTree ve SpeedGrass Sabitleri)

*   **Amaç:** `Constants.h` dosyası, `SpeedTreeLib` kütüphanesi, özellikle de SpeedGrass alt sistemi tarafından kullanılan çeşitli sayısal sabitleri ve yapılandırma değerlerini merkezi bir yerde tanımlar. Bu sabitler, vertex shader sabit konumları, çimen render parametreleri (LOD mesafeleri, solma uzunlukları, bıçak (blade) özellikleri), matematiksel sabitler ve çimen vertex yapısının öznitelik boyutları gibi alanları kapsar.

*   **Temel Tanımlamalar ve Sabitler:**
    *   **`USE_SPEEDGRASS` Makrosu:** Bu makro, SpeedGrass ile ilgili kod bloklarının derlenip derlenmeyeceğini kontrol etmek için kullanılır. Kodda `#ifdef USE_SPEEDGRASS` direktifleriyle sarmalanmış bölümler bu makro tanımlı olduğunda aktif hale gelir.
    *   **Vertex Shader Sabit Konumları:**
        *   `c_nShaderLightPosition` (16)
        *   `c_nShaderGrassBillboard` (4)
        *   `c_nShaderGrassLodParameters` (8)
        *   `c_nShaderCameraPosition` (9)
        *   `c_nShaderGrassWindDirection` (10)
        *   `c_nShaderGrassPeriods` (11)
        *   `c_nShaderLeafAmbient` (47)
        *   `c_nShaderLeafDiffuse` (48)
        *   `c_nShaderLeafSpecular` (49)
        *   `c_nShaderLeafLightingAdjustment` (50)
        *   Bu sabitler, vertex shader programlarında belirli uniform değişkenlerin hangi sabit register'lara bağlanacağını tanımlar.
    *   **Çimen (Grass) Parametreleri:**
        *   `c_fDefaultGrassFarLod` (300.0f): Çimen için varsayılan en uzak LOD (Level of Detail) mesafesi.
        *   `c_fGrassFadeLength` (50.0f): Çimenin LOD mesafesine yaklaştıkça ne kadarlık bir mesafe içinde solacağı (fade out).
        *   `c_fMinBladeNoise` (-0.2f), `c_fMaxBladeNoise` (0.2f): Her bir çimen bıçağının pozisyonuna uygulanacak rastgele gürültü (noise) için min/max sınırlar.
        *   `c_fMinBladeThrow` (1.0f), `c_fMaxBladeThrow` (2.5f): Çimen bıçaklarının rüzgarla ne kadar "savrulacağını" belirleyen min/max faktörler.
        *   `c_fMinBladeSize` (7.5f), `c_fMaxBladeSize` (10.0f): Çimen bıçaklarının boyutu için min/max sınırlar.
        *   `c_nNumBladeMaps` (2): Kullanılacak farklı çimen bıçağı doku haritası sayısı.
        *   `c_fGrassBillboardWideScalar` (1.75f): Çimen billboard'larının ne kadar geniş görüneceğini belirleyen bir çarpan.
        *   `c_fWalkerHeight` (12.0f): Muhtemelen oyuncu veya kamera yüksekliğiyle ilgili bir sabit, çimen etkileşimleri için kullanılabilir.
        *   `c_nDefaultGrassBladeCount` (33000): Varsayılan olarak oluşturulacak çimen bıçağı sayısı.
        *   `c_nGrassRegionCount` (20): Çimenlerin bölümlere (region) ayrılmasında kullanılan bir sayı (muhtemelen bir boyuttaki region sayısı).
    *   **Matematiksel Sabitler:**
        *   `c_fPi`, `c_fHalfPi`, `c_fQuarterPi`, `c_fTwoPi`: Pi sayısı ve çeşitli katları.
        *   `c_fDeg2Rad` (57.29578f): Dereceyi radyana çevirmek için kullanılan bir sabit (Aslında `180.0f / c_fPi` olmalı, bu değer `c_fRad2Deg` gibi duruyor, yani radyanı dereceye çevirir. Kullanım yerine göre kontrol edilmeli).
        *   `c_f90` (0.5f * c_fPi): 90 derece (radyan cinsinden).
    *   **Çimen Vertex Öznitelik Boyutları (Byte Cinsinden):**
        *   `c_nGrassVertexTexture0Size` (2 * sizeof(float)): Temel doku koordinatları (UV).
        *   `c_nGrassVertexTexture1Size` (4 * sizeof(float)): Vertex indeksi, bıçak boyutu, rüzgar ağırlığı, gürültü faktörü gibi ek veriler.
        *   `c_nGrassVertexColorSize` (4 * sizeof(unsigned char)): Vertex rengi (RGBA).
        *   `c_nGrassVertexPositionSize` (3 * sizeof(float)): Vertex pozisyonu (XYZ).
        *   `c_nGrassVertexTotalSize`: Bir çimen vertex'inin toplam boyutu.
    *   **Yorum Satırı Haline Getirilmiş Bölümler:**
        *   Dosyada aydınlatma (`c_afLightDir`, `c_afLightPos`), istatistikler (`c_nStatFrameRate` vb.), arazi (`c_fTerrainBaseRepeat` vb.), rüzgar matrisleri, ağaç LOD parametreleri ve ağaçların dal (branch), yaprak (frond/leaf) ve arazi vertex öznitelik boyutları gibi birçok sabit yorum satırı (`/* ... */`) içinde bırakılmıştır. Bu, bu sabitlerin ya artık kullanılmadığını, ya farklı bir konfigürasyona ait olduğunu ya da başka bir yerde tanımlandığını gösterebilir.

*   **Kullanım Amacı:**
    *   `SpeedTreeLib` ve özellikle `SpeedGrassRT` sınıfı için yapılandırma ve render parametrelerini merkezi bir yerden sağlamak.
    *   Shader programları ile C++ kodu arasında tutarlı sabit değerlerin kullanılmasını garantilemek.
    *   Çimenlerin görünümünü, davranışını ve performansını etkileyen çeşitli ayarları kolayca değiştirilebilir kılmak.

*   **Bağımlılıklar:**
    *   Temel C++ ve matematik.
    *   (Yorum satırlarındaki `CFilename` ve `IdvVector` gibi sınıflar, IDV'nin (Interactive Data Visualization, SpeedTree'nin orijinal geliştiricisi) kendi iç kütüphanelerine işaret ediyor olabilir.)

### `SpeedGrassRT.h` ve `SpeedGrassRT.cpp` (`CSpeedGrassRT` Sınıfı)

*   **Amaç:** `CSpeedGrassRT` sınıfı, SpeedTree'nin SpeedGrass teknolojisini kullanarak oyun dünyasında çimen ve benzeri bitki örtüsünün verimli bir şekilde oluşturulması, yönetilmesi ve render edilmesi için temel işlevselliği sağlar. Bu sınıf, çimen bıçaklarının (`SBlade`) yerleştirilmesi, bu bıçakların culling (eleme) ve LOD (Level of Detail - Detay Seviyesi) işlemleri için bölgelere (`SRegion`) ayrılması, kamera pozisyonu ve yönelimine göre billboard'ların hesaplanması, rüzgar efektlerinin uygulanması ve çimen geometrisinin vertex buffer'lara yüklenmesi gibi görevleri üstlenir.

*   **`SpeedGrassRT.h` - Temel Tanımlar ve Bildirimler:**
    *   **`USE_SPEEDGRASS` Makrosu:** Tüm dosya içeriği bu makro ile sarmalanmıştır, yani SpeedGrass özelliği aktifse derlenir.
    *   **`SBlade` Yapısı:** Tek bir çimen bıçağının özelliklerini tanımlar:
        *   `m_afPos[3]` (float): Bıçağın 3D pozisyonu.
        *   `m_afNormal[3]` (float): Bıçağın bulunduğu yüzeyin normali.
        *   `m_fSize` (float): Bıçağın boyutu.
        *   `m_ucWhichTexture` (unsigned char): Kullanılacak çimen dokusunun indeksi.
        *   `m_fNoise` (float): Rüzgar salınımında kullanılacak rastgele gürültü faktörü.
        *   `m_fThrow` (float): Rüzgarın bıçağı ne kadar "savuracağını" belirleyen faktör.
        *   `m_afBottomColor[3]`, `m_afTopColor[3]` (float): Bıçağın alt ve üst kısımlarının renkleri (RGB).
    *   **`SRegion` Yapısı:** Bir grup çimen bıçağını içeren bir bölgeyi tanımlar:
        *   `m_afCenter[3]`, `m_afMin[3]`, `m_afMax[3]` (float): Bölgenin merkezi ve AABB (Axis-Aligned Bounding Box) sınırları.
        *   `m_bCulled` (bool): Bölgenin view frustum (görüş alanı) dışında olup olmadığı.
        *   `m_fCullingRadius` (float): Bölgenin culling işlemleri için kullanılan küresel yarıçapı.
        *   `m_vBlades` (std::vector<SBlade>): Bu bölgeye ait çimen bıçaklarının bir vektörü.
        *   `m_VertexBuffer` (CGraphicVertexBuffer): Bu bölgedeki çimenlerin render edilmesi için kullanılacak vertex buffer.
    *   **Statik Üye Değişkenler:**
        *   `m_fLodFarDistance`, `m_fLodTransitionLength`: Çimen LOD ayarları.
        *   `m_afUnitBillboard[12]`: Kameraya bakan birim billboard'un 4 köşe noktasının koordinatları (4 köşe * 3 float).
        *   `m_afWindDir[4]`: Rüzgar yönü ve gücü (XYZ + W=0.0f).
        *   `m_afCameraOut`, `m_afCameraUp`, `m_afCameraRight`, `m_afCameraPos`: Kamera yönelim ve pozisyon vektörleri.
        *   `m_fFieldOfView`, `m_fAspectRatio`: Kamera görüş alanı ve en-boy oranı.
        *   `m_afFrustumBox`, `m_afFrustumMin`, `m_afFrustumMax`, `m_afFrustumPlanes`: View frustum culling için kullanılan veriler.
    *   **Genel Metotlar:**
        *   `DeleteRegions()`: Oluşturulmuş tüm çimen bölgelerini siler.
        *   `GetRegions()`: Çimen bölgelerine erişim sağlar.
        *   `ParseBsfFile()`: Bir `.bsf` (Boundary Shape File) dosyasını kullanarak çimen bıçaklarını belirli sınırlar içine yerleştirir. `CBoundaryShapeManager` kullanır.
        *   `CustomPlacement()`: Özel çimen yerleştirme mantığı için bir şablon sunar.
        *   `GetUnitBillboard()`: Hesaplanan birim billboard köşe noktalarını döndürür.
        *   LOD, Wind, Camera için `Get` ve `Set` metotları.
    *   **Sanal (Virtual) Metotlar (Arazi Etkileşimi):**
        *   `virtual float Color(float fX, float fY, const float* pNormal, float* pTopColor, float* pBottomColor) const`: Verilen (X,Y) koordinatındaki ve normaldeki bir nokta için çimenin üst ve alt renklerini belirler. Dönüş değeri genellikle yüksekliğe bağlı bir interpolasyon faktörüdür. Bu metodun, arazi sisteminden renk bilgilerini alacak şekilde türetilmiş bir sınıfta implemente edilmesi beklenir.
        *   `virtual float Height(float fX, float fY, float* pNormal) const`: Verilen (X,Y) dünya koordinatındaki arazinin yüksekliğini (`return` değeri) ve o noktadaki yüzey normalini (`pNormal` aracılığıyla) döndürür. Bu metodun da arazi sistemiyle etkileşim kuracak şekilde implemente edilmesi gerekir.
    *   **Korunan (Protected) Metotlar:**
        *   `CreateRegions()`: Verilen çimen bıçağı listesinden bölgeleri oluşturur, bıçakları bölgelere atar ve her bölgenin sınırlarını hesaplar.
        *   `ComputeFrustum()`: Mevcut kamera parametrelerine göre view frustum düzlemlerini ve sınırlarını hesaplar.
        *   `ComputeUnitBillboard()`: Kameranın mevcut yönelimine göre birim billboard'u hesaplar.
        *   `ConvertCoordsToCell()`: Dünya koordinatlarını bölge gridi (ızgarası) hücre indekslerine dönüştürür.
        *   `GetRegionIndex()`: Satır ve sütun indekslerinden tek boyutlu bölge dizisi için indeks hesaplar.
        *   `OutsideFrustum()`: Bir bölgenin view frustum dışında olup olmadığını kontrol eder.

*   **`SpeedGrassRT.cpp` - Uygulama Detayları:**
    *   **Yardımcı Fonksiyonlar:**
        *   `VecInterpolate()`: İki float değer arasında doğrusal interpolasyon yapar.
        *   `VectorSinD()`, `VectorCosD()`: Derece cinsinden açı alan sinüs ve kosinüs fonksiyonları (D3DXToRadian benzeri bir dönüşüm içerirler).
    *   **Statik Üyelerin Başlatılması:** LOD, rüzgar, kamera, billboard ve frustum ile ilgili statik değişkenlere varsayılan değerler atanır.
    *   **`SBlade::SBlade()` ve `SRegion::SRegion()` Yapıcıları:** Üye değişkenleri varsayılan değerlere ayarlar.
    *   **`ParseBsfFile()`:**
        *   `CBoundaryShapeManager` kullanarak `.bsf` dosyasını yükler.
        *   Belirtilen sayıda (`nNumBlades`) çimen bıçağı oluşturmaya çalışır.
        *   Her bıçak için `cManager.RandomPoint()` ile `.bsf` sınırları içinde rastgele bir (X,Y) pozisyonu alır.
        *   `Height()` sanal metodunu çağırarak Z pozisyonunu ve yüzey normalini alır.
        *   Genel sahne sınırlayıcı kutusunu (`m_afBoundingBox`) günceller.
        *   `Color()` sanal metodunu çağırarak bıçağın alt ve üst renklerini ve boyutunu ayarlar.
        *   Rastgele bir doku indeksi, rüzgar gürültüsü ve savrulma değeri atar.
        *   Tüm oluşturulan bıçaklarla `CreateRegions()` metodunu çağırır.
    *   **`CreateRegions()`:**
        *   Mevcut bölgeleri siler ve sahne sınırlayıcı kutusuna (`m_afBoundingBox`) ve belirtilen satır/sütun sayısına göre yeni bölgeler oluşturur.
        *   Her bölgenin 2D (X,Y) sınırlarını ve merkezini hesaplar.
        *   Verilen `vSceneBlades` vektöründeki her bir çimen bıçağını, pozisyonuna göre uygun bölgeye atar.
        *   Bıçaklar bölgelere atandıktan sonra, her bölgenin Z sınırlarını (`m_afMin[2]`, `m_afMax[2]`) ve 3D culling yarıçapını günceller.
        *   Eğer `fCollisionDistance > 0.0f` ise, her bölge içindeki bıçaklar arasında basit bir 2D çarpışma tespiti yapar ve birbirine çok yakın olan bıçakları siler.
    *   **`RotateAxisFromIdentity()`:** Verilen bir eksen ve açı ile bir birim matrisinden (identity matrix) dönme matrisi oluşturur. (Bu fonksiyon doğrudan `D3DXMatrixRotationAxis` ile benzer bir işlev görür.)
    *   **`ComputeFrustum()`:**
        *   Kamera pozisyonu, yönelim vektörleri (sağ, yukarı, ileri/içeri), görüş alanı (FOV) ve LOD mesafelerini kullanarak 5 adet frustum düzlemi (uzak, üst, sol, alt, sağ) hesaplar. Her düzlem `Ax + By + Cz + D = 0` formundadır.
        *   Ayrıca, frustum'un dünya koordinatlarındaki 2D (X,Y) izdüşümünün minimum ve maksimum noktalarını (`m_afFrustumMin`, `m_afFrustumMax`) hesaplar.
    *   **`ComputeUnitBillboard()`:**
        *   Kameranın Z ekseni etrafındaki yatay açısını (`fAzimuth`) hesaplar.
        *   Bu açıyı kullanarak, YZ düzleminde tanımlanmış standart bir dörtgeni (unit billboard) döndürerek kameraya bakmasını sağlar. Döndürülmüş 4 köşe noktasının koordinatları `m_afUnitBillboard` dizisinde saklanır.
    *   **`ConvertCoordsToCell()`:** Dünya X,Y koordinatlarını, tüm çimen alanını kaplayan `m_nNumRegionCols` x `m_nNumRegionRows` boyutlarındaki bölge gridindeki hücre (region) indekslerine dönüştürür.
    *   **`Cull()`:**
        *   Önce `m_afFrustumMin` ve `m_afFrustumMax` (frustum'un 2D AABB'si) kullanılarak potansiyel olarak görülebilir bölge hücre aralığı belirlenir.
        *   Eğer tüm çimen alanı bu AABB'nin dışındaysa, tüm bölgeler culled (`m_bAllRegionsCulled = true`) edilir.
        *   Aksi halde, bu potansiyel hücre aralığındaki her bölge için `OutsideFrustum()` çağrılarak daha hassas bir culling testi yapılır.
    *   **`OutsideFrustum()`:** Bir bölgenin merkez noktasının, 5 frustum düzleminin her birine olan mesafesini, bölgenin culling yarıçapıyla karşılaştırır. Eğer merkez, herhangi bir düzlemin "dış" tarafında ve yarıçaptan daha uzaksa, bölge culled edilir. Bu, basit bir küre-düzlem kesişim testidir.

*   **Kullanım Amacı:**
    *   Geniş alanlara verimli bir şekilde çimen render etmek.
    *   Çimenleri bölgelere ayırarak sadece görünür ve LOD sınırları içindeki çimenlerin işlenmesini sağlamak.
    *   Çimenlerin araziye uyum sağlaması (yükseklik ve renk alması) için bir arayüz sunmak.
    *   Temel rüzgar ve kamera etkileşimli billboard'lar oluşturmak.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (SpeedTreeLib için)
    *   `BoundaryShapeManager.h`: `.bsf` dosyalarından çimen yerleşim sınırlarını okumak için.
    *   `Constants.h`: Çeşitli sabitler ve yapılandırma değerleri için.
    *   `../EterLib/GrpVertexBuffer.h`: `CGraphicVertexBuffer` için.
    *   DirectX (D3DXVECTOR3, D3DXMATRIX, D3DXToRadian, D3DXVec3Normalize, D3DXVec3Dot, D3DXMatrixRotationZ, D3DXVec3TransformCoord, D3DXMatrixIdentity).
    *   Standart C++ kütüphaneleri (`vector`, `cmath` için `sinf`, `cosf`, `tanf`, `sqrtf`, `atan2`).
    *   Temel C++ (`min`, `max`, `memcpy`).

### `SpeedGrassWrapper.h` ve `SpeedGrassWrapper.cpp` (`CSpeedGrassWrapper` Sınıfı)

*   **Amaç:** `CSpeedGrassWrapper` sınıfı, `CSpeedGrassRT` temel sınıfından türeyerek, SpeedGrass (çimen) sistemini oyunun özel arazi sistemi (`CMapOutdoor`) ile entegre eden bir sarmalayıcı (wrapper) görevi görür. `CSpeedGrassRT`'nin arazi etkileşimi için tanımladığı sanal (virtual) metotları (`Color` ve `Height`) implemente ederek, çimenlerin doğru yüksekliklerde ve arazi renkleriyle uyumlu bir şekilde yerleştirilmesini sağlar. Ayrıca, çimen dokusunu yüklemek ve her çimen bölgesi (`SRegion`) için vertex buffer'ları hazırlamak gibi grafik başlatma işlemlerini de gerçekleştirir.

*   **`SpeedGrassWrapper.h` - Temel Tanımlar ve Bildirimler:**
    *   **Kalıtım:** `public CSpeedGrassRT` sınıfından miras alır.
    *   **Genel Metotlar:**
        *   `CSpeedGrassWrapper()`: Yapıcı metot. `m_pMapOutdoor` ve `m_lpD3DTexure8`'i null olarak başlatır.
        *   `virtual ~CSpeedGrassWrapper()`: Yıkıcı metot (varsayılan).
        *   `SetMapOutdoor(CMapOutdoor* pMapOutdoor)`: Kullanılacak `CMapOutdoor` nesnesinin işaretçisini ayarlar.
        *   `Draw(float fDensity)`: Çimenleri çizmek için kullanılan (ancak C++ kodunda yorum satırı halinde bırakılmış) metot. Orijinal implementasyon OpenGL komutları içeriyordu.
        *   `InitFromBsfFile(const char* pFilename, unsigned int nNumBlades, unsigned int uiRows, unsigned int uiCols, float fCollisionDistance)`: Belirtilen `.bsf` dosyasından çimenleri başlatır, temel sınıfın `ParseBsfFile` metodunu çağırır ve ardından `InitGraphics()`'i çağırır.
    *   **Özel (Private) Sanal (Virtual) Metotlar (Override Edilenler):**
        *   `virtual float Color(float fX, float fY, const float* pNormal, float* pTopColor, float* pBottomColor) const override`: `CSpeedGrassRT`'den gelen bu metodu override eder. Verilen (X,Y) dünya koordinatındaki çimen bıçağının alt ve üst renklerini `m_pMapOutdoor->GetBrushColor()` kullanarak araziden alır ve ayarlar.
        *   `virtual float Height(float fX, float fY, float* pNormal) const override`: `CSpeedGrassRT`'den gelen bu metodu override eder. Verilen (X,Y) dünya koordinatındaki arazinin yüksekliğini `m_pMapOutdoor->GetHeight()` kullanarak alır ve yüzey normalini varsayılan olarak dikey (0,0,1) ayarlar.
    *   **Özel (Private) Metot:**
        *   `InitGraphics(void)`: Çimen dokusunu yükler ve her `SRegion` için vertex buffer'ları oluşturur.
    *   **Özel (Private) Üye Değişkenler:**
        *   `m_pMapOutdoor` (CMapOutdoor*): Oyunun arazi verilerine erişim için kullanılan `CMapOutdoor` nesnesine işaretçi.
        *   `m_lpD3DTexure8` (LPDIRECT3DTEXTURE8): Yüklenen çimen dokusunun Direct3D 8 arayüzüne işaretçi.
        *   `m_GrassImageInstance` (CGraphicImageInstance): Çimen dokusunu yönetmek için kullanılan bir `CGraphicImageInstance` nesnesi.

*   **`SpeedGrassWrapper.cpp` - Uygulama Detayları:**
    *   **`Draw(float fDensity)`:**
        *   Bu fonksiyonun içeriği tamamen yorum satırı (`/* ... */`) halindedir.
        *   Orijinal (yorum satırındaki) kod, OpenGL kullanarak çimenleri render etmek üzere tasarlanmıştı:
            *   `Cull()` (temel sınıftan) çağrılarak görünür bölgeler belirlenir.
            *   OpenGL durumları ayarlanır (Culling, Blend, Texture, Alpha Test).
            *   Her bir görünür bölge (`SRegion`) için, bölgenin vertex buffer'ı (`pRegions[i].m_pVertexBuffer`) bağlanır ve `glDrawArrays(GL_QUADS, ...)` ile çimen quad'ları çizilir.
            *   Render edilen üçgen sayısı hesaplanır.
        *   Mevcut Metin2 istemcisi DirectX kullandığı için bu OpenGL tabanlı çizim kodu devre dışı bırakılmıştır. Çizim işlemleri muhtemelen istemcinin ana render döngüsü içinde, `CSpeedGrassWrapper`'dan alınan verilerle ve DirectX API'si kullanılarak farklı bir yerde yapılmaktadır.
    *   **`InitFromBsfFile(...)`:**
        *   Temel sınıfın `ParseBsfFile()` metodunu çağırarak çimen bıçaklarının `.bsf` dosyasına göre yerleştirilmesini ve bölgelerin oluşturulmasını sağlar.
        *   Ardından `InitGraphics()` metodunu çağırarak grafik kaynaklarını hazırlar.
    *   **`Color(...) const` (Override):**
        *   `m_pMapOutdoor` işaretçisi geçerliyse, `m_pMapOutdoor->GetBrushColor(fX, fY, afLowColor, afHighColor)` metodunu çağırarak arazinin o noktasındaki fırça renklerini (alt ve üst) alır.
        *   Bu renkleri (Metin2'nin renk formatından RGB'ye dönüştürerek) `pBottomColor` ve `pTopColor` çıktı parametrelerine atar. Üst renk için ayrıca rastgele bir interpolasyon ve hafif bir renk sapması uygular.
        *   Renk değerlerini [0,1] aralığına sınırlar.
        *   `afLowColor[3]` (muhtemelen alfa veya yoğunluk bilgisi) değerini döndürür.
    *   **`Height(...) const` (Override):**
        *   `m_pMapOutdoor` işaretçisi geçerliyse, `m_pMapOutdoor->GetHeight(afPos)` metodunu çağırarak verilen (X,Y) koordinatındaki arazi yüksekliğini alır.
        *   `pNormal` çıktı parametresini varsayılan olarak (0,0,1) yani düz bir yukarı normal olarak ayarlar. Bu, çimenlerin her zaman dikey olarak büyüyeceği anlamına gelir, arazi eğimini dikkate almaz.
        *   Hesaplanan yüksekliği döndürür.
    *   **`InitGraphics(void)`:**
        *   Çimen dokusunu (`"D:/ymir work/special/brush_2.dds"`) `CResourceManager` aracılığıyla yükler ve `m_GrassImageInstance` üzerine ayarlar. Doku nesnesini `m_lpD3DTexure8`'e alır. (Not: Dosya yolu sabit kodlanmıştır, bu genellikle yapılandırma dosyası veya daha dinamik bir yolla çözülmelidir.)
        *   `m_nNumRegions` kadar döngüye girerek her bir `SRegion` için:
            *   Geçici bir bellek alanı (`pBuffer`) oluşturur. Bu alan, bölgedeki tüm çimen bıçaklarının vertex verilerini (her bıçak için 4 köşe) tutacak boyuttadır.
            *   Vertex verileri şunları içerir:
                *   **TexCoords0 (UV):** Her bıçağın hangi doku bölümünü kullanacağını belirler (`c_nNumBladeMaps`'e göre).
                *   **TexCoords1 (Ek Veriler):**
                    *   `x`: Billboard köşe indeksi (`c_nShaderGrassBillboard` + köşe no).
                    *   `y`: Bıçağın boyutu (`iBlade->m_fSize`).
                    *   `z`: Bıçağın rüzgar savrulma faktörü (`iBlade->m_fThrow`).
                    *   `w`: Bıçağın rüzgar gürültü faktörü (`iBlade->m_fNoise`).
                *   **Color (RGBA):** Bıçağın köşe noktalarının renkleri (`iBlade->m_afTopColor`, `iBlade->m_afBottomColor`). Alfa tam opak (0xff) ayarlanır.
                *   **Position (XYZ):** Bıçağın kök pozisyonu (`iBlade->m_afPos`). Tüm 4 köşe için aynı pozisyon kullanılır; gerçek köşe pozisyonları vertex shader'da billboard matrisi ve diğer parametrelerle hesaplanır.
            *   Bu verileri `pBuffer`'a doldurur.
            *   (Yorum satırı halinde bırakılmış kodda) `pRegion->m_VertexBuffer` (`CIdvVertexBuffer` veya `CGraphicVertexBuffer`) bu `pBuffer`'daki verilerle doldurulur ve FVF (Flexible Vertex Format) veya stride bilgileri ayarlanır.
            *   Mevcut kodda `pRegion->m_VertexBuffer.Create()` çağrısı ve `D3DFVF_XYZ | D3DFVF_DIFFUSE | D3DFVF_TEX1` FVF tanımı görünmektedir, ancak buffer'a veri yükleme kısmı tam olarak aktif değildir. Muhtemelen `CGraphicVertexBuffer`'ın `SetRawData` veya benzeri bir metoduyla bu veriler daha sonra yüklenir.
            *   Geçici `pBuffer` silinir.

*   **Kullanım Amacı:**
    *   `CSpeedGrassRT`'nin soyut arazi etkileşimlerini, Metin2'nin `CMapOutdoor` sistemiyle somutlaştırmak.
    *   Çimenlerin araziye doğru şekilde yerleşmesini (yükseklik) ve renk olarak uyum sağlamasını sağlamak.
    *   Çimen render için gerekli doku ve vertex buffer gibi grafik kaynaklarını yüklemek ve hazırlamak.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (SpeedTreeLib için)
    *   `SpeedGrassRT.h` (Temel sınıf)
    *   `../EterLib/ResourceManager.h`, `../EterLib/GrpTexture.h`, `../EterLib/GrpVertexBuffer.h`, `../EterLib/GrpScreen.h`
    *   `../GameLib/MapOutdoor.h` (CMapOutdoor için)
    *   `../EterBase/Random.h` (`GetRandom` için)
    *   `Constants.h` (SpeedGrass sabitleri için)
    *   DirectX (LPDIRECT3DTEXTURE8, D3DFVF).
    *   Standart C++ kütüphaneleri (`vector`, `stdlib`, `stdio`).

### `SpeedTreeConfig.h` (SpeedTreeRT Çalışma Zamanı Yapılandırması)

*   **Amaç:** `SpeedTreeConfig.h` dosyası, SpeedTreeRT (Runtime) kütüphanesinin temel çalışma zamanı davranışlarını ve özelliklerini kontrol eden bir dizi `#define` direktifi ve sabit değer içerir. Bu yapılandırmalar, render ayarları, aydınlatma, rüzgar efektleri, Seviye Detayı (LOD) yönetimi, yaprak yerleşimi, doku koordinatları, yukarı vektörü ve billboard modları gibi SpeedTree'nin çeşitli yönlerini etkiler. Bu dosya, istemcinin ihtiyaçlarına göre SpeedTree'nin nasıl çalışacağını önceden belirlemek için merkezi bir kontrol noktası sunar.

*   **Temel Yapılandırma Alanları ve Sabitler:**
    *   **Genel Sabitler:**
        *   `c_nNumWindMatrices` (4): Rüzgar animasyonları için kullanılacak farklı rüzgar matrisi sayısı.
        *   `c_nNumInstancesPerModel` (10): Bir ağaç modelinden oluşturulabilecek maksimum örnek (instance) sayısı. (Not: Bu sabit, `WRAPPER_FOREST_FROM_INSTANCES` moduyla ilgili olabilir.)
        *   `c_fForestSize` (200.0f): Ormanın genel boyutunu veya ağaçların yayılabileceği maksimum alanı belirten bir değer.
        *   `c_fSpacingTolerance` (30.0f): Ağaçlar arasında korunması gereken minimum boşluk toleransı.
        *   `c_nMaxPlacementIterations` (500): Ağaç yerleştirme algoritmalarının maksimum deneme sayısı.
        *   `c_nDefaultAlphaTestValue` (84): Alfa testi için varsayılan referans değeri (0-255 aralığında).
        *   `c_fNearLodFactor` (2.0f), `c_fFarLodFactor` (9.0f): Ağaçların LOD (Level of Detail) geçiş mesafelerini hesaplamak için kullanılan çarpanlar. Genellikle ağacın boyutuyla çarpılır.
        *   `c_fBenchmarkPeriod` (1.0f): Performans ölçümü (benchmark) periyodu (saniye cinsinden).
    *   **Vertex Shader Sabit Konumları:**
        *   Vertex shader programlarında kullanılacak çeşitli uniform değişkenlerin (örneğin, `c_nVertexShader_LeafLightingAdjustment` (70), `c_nVertexShader_Light` (71), `c_nVertexShader_Material` (74), `c_nVertexShader_TreePos` (52), `c_nVertexShader_CompoundMatrix` (0), `c_nVertexShader_WindMatrices` (54), `c_nVertexShader_LeafTables` (4), `c_nVertexShader_Fog` (85)) hangi sabit register'lara (constant registers) bağlanacağını tanımlar. Bu, C++ kodu ile shader programları arasında veri akışını sağlar.
    *   **Aydınlatma Ayarları:**
        *   `c_afLightPosition`, `c_afLightAmbient`, `c_afLightDiffuse`, `c_afLightSpecular`, `c_afLightGlobalAmbient`: Varsayılan yönlü ışık kaynağının pozisyonu/yönü ve temel aydınlatma bileşenlerinin (çevresel, yaygın, yansımalı, genel çevresel) renk ve yoğunluk değerlerini tanımlar.
        *   `WRAPPER_USE_STATIC_LIGHTING` vs `WRAPPER_USE_DYNAMIC_LIGHTING`: SpeedTree sarmalayıcısının statik (önceden hesaplanmış) veya dinamik (çalışma zamanında hesaplanan) aydınlatma modelini kullanıp kullanmayacağını belirler. Kodda `WRAPPER_USE_STATIC_LIGHTING` seçilidir.
    *   **Rüzgar Ayarları:**
        *   `WRAPPER_USE_GPU_WIND`, `WRAPPER_USE_CPU_WIND`, `WRAPPER_USE_NO_WIND`: Rüzgar efektlerinin GPU'da shader'lar aracılığıyla mı, CPU'da mı hesaplanacağını yoksa hiç rüzgar efekti kullanılmayacağını mı belirler. Kodda `WRAPPER_USE_NO_WIND` seçilidir, yani varsayılan olarak rüzgar efektleri devre dışıdır.
    *   **Yaprak Yerleştirme Algoritması:**
        *   `WRAPPER_USE_GPU_LEAF_PLACEMENT` vs `WRAPPER_USE_CPU_LEAF_PLACEMENT`: Ağaç yapraklarının dallara yerleştirilmesi işleminin GPU üzerinde mi yoksa CPU üzerinde mi yapılacağını seçer. Kodda `WRAPPER_USE_CPU_LEAF_PLACEMENT` seçilidir.
    *   **Doku Koordinatları:**
        *   `WRAPPER_FLIP_T_TEXCOORD`: Bu makro tanımlıysa, doku koordinatlarından T (veya V) bileşeninin ters çevrilmesi gerektiğini belirtir. Bu genellikle DirectX gibi T koordinatını yukarıdan aşağıya doğru artıran grafik API'leri için gereklidir. Kodda bu tanımlıdır.
    *   **Yukarı Vektörü (Up Vector):**
        *   `WRAPPER_UP_POS_Y` vs `WRAPPER_UP_POS_Z`: 3D dünyada "yukarı" yönünün pozitif Y ekseni mi yoksa pozitif Z ekseni mi olduğunu tanımlar. Metin2'nin koordinat sistemine uygun olarak kodda `WRAPPER_UP_POS_Z` seçilidir.
    *   **Orman Yükleme Mekanizması:**
        *   `WRAPPER_FOREST_FROM_STF` vs `WRAPPER_FOREST_FROM_INSTANCES`: Ağaçların ve orman düzeninin bir `.stf` (SpeedTree Forest) dosyasından mı yükleneceğini, yoksa ayrı ayrı ağaç modellerinin örneklenmesi (instancing) ve programatik olarak yerleştirilmesiyle mi oluşturulacağını belirler. Kodda `WRAPPER_FOREST_FROM_INSTANCES` seçilidir.
    *   **Billboard Modları:**
        *   `WRAPPER_BILLBOARD_MODE`: Uzaktaki ağaçlar için billboard kullanımını genel olarak aktif eder. Kodda bu tanımlıdır.
        *   `WRAPPER_RENDER_HORIZONTAL_BILLBOARD`: Üstten bakıldığında görünen yatay billboard'ların render edilip edilmeyeceğini kontrol eder. Kodda bu yorum satırı halindedir, yani aktif değildir.
    *   **Ek Özellikler:**
        *   `WRAPPER_RENDER_SELF_SHADOWS`: Ağaçların kendi üzerlerine gölge düşürme özelliğini aktif eder. Kodda bu tanımlıdır.
        *   `WRAPPER_USE_FOG`: Sahnedeki sis efektlerinin SpeedTree ağaçlarını etkileyip etkilemeyeceğini belirler. Kodda bu tanımlıdır.
    *   **Türetilmiş Sabitler (Derived Constants):**
        *   Yukarıdaki makro seçimlerine bağlı olarak bazı ek makrolar tanımlanır. Örneğin, `WRAPPER_USE_GPU_WIND` aktifse, ağaçların dalları (`BRANCHES_USE_SHADERS`), yaprakları (`FRONDS_USE_SHADERS`, `LEAVES_USE_SHADERS`) rüzgar efektleri için shader kullanacak şekilde ayarlanır. Benzer şekilde, `WRAPPER_USE_GPU_LEAF_PLACEMENT` aktifse, yapraklar için shader kullanımı (`LEAVES_USE_SHADERS`) etkinleştirilir.

*   **Kullanım Amacı:**
    *   SpeedTreeRT kütüphanesinin çekirdek davranışlarını ve özelliklerini oyun motorunun veya istemcinin gereksinimlerine göre özelleştirmek.
    *   Farklı donanım yeteneklerine veya performans hedeflerine göre SpeedTree ayarlarını optimize etmek (örneğin, GPU vs CPU rüzgarı).
    *   Grafik API (DirectX vs OpenGL) ve koordinat sistemi farklılıklarına uyum sağlamak.
    *   Aydınlatma, sis, gölgeleme gibi görsel efektlerin SpeedTree entegrasyonunu kontrol etmek.

*   **Bağımlılıklar:**
    *   Temel C++ önişlemci direktifleri.
    *   Bu dosyadaki tanımlar, `SpeedTreeRT.h` ve SpeedTree sarmalayıcı sınıfları (`CSpeedTreeWrapper`, `CSpeedTreeForest` vb.) tarafından kullanılır.

### `SpeedTreeForest.h` ve `SpeedTreeForest.cpp` (`CSpeedTreeForest` Sınıfı)

*   **Amaç:** `CSpeedTreeForest` sınıfı, bir oyun sahnesindeki tüm SpeedTree ağaçlarını ve bitki örtüsünü merkezi olarak yönetmek için tasarlanmıştır. Ana ağaç modellerini (genellikle `.spt` dosyalarından yüklenen şablonlar) yönetir, bu modellerden oyun dünyasında görünecek örnekler (instances) oluşturur ve siler. Ayrıca, tüm orman için geçerli olan genel rüzgar efektlerini, Seviye Detay (LOD) ayarlarını ve aydınlatma/sis gibi çevresel etkileri koordine eder. Sınıf, grafik API'sine özel bazı işlemleri (örneğin, rüzgar matrislerinin shader'lara yüklenmesi veya ağaçların çizilmesi) gerçekleştirmek için sanal (virtual) metotlar içerir, bu da DirectX veya OpenGL gibi farklı render sistemleri için özelleştirilmiş alt sınıflar oluşturulmasına olanak tanır.

*   **`SpeedTreeForest.h` - Temel Tanımlar ve Arayüz:**
    *   **Render Bit Vektörü (Makrolar):**
        *   `Forest_RenderBranches` (1 << 0): Ağaç dallarını render et.
        *   `Forest_RenderLeaves` (1 << 1): Ağaç yapraklarını (genellikle quad'lar veya daha karmaşık geometriler) render et.
        *   `Forest_RenderFronds` (1 << 2): Ağaçların eğrelti otu benzeri kısımlarını (fronds) render et.
        *   `Forest_RenderBillboards` (1 << 3): Uzaktaki ağaçlar için kullanılan 2D billboard'ları render et.
        *   `Forest_RenderAll` ((1 << 4) - 1): Tüm ağaç bileşenlerini render et.
        *   `Forest_RenderToShadow` (1 << 5): Ağaçları gölge haritası oluşturmak amacıyla render et (genellikle daha basit bir materyal ile).
        *   `Forest_RenderToMiniMap` (1 << 6): Ağaçları minimap üzerinde gösterilecek şekilde render et.
        *   Bu makrolar, `Render()` sanal metoduna parametre olarak geçirilerek çizim işleminin detaylarını kontrol eder.
    *   **Tür Tanımları (Type Aliases):**
        *   `SpeedTreeWrapperPtr`: `std::shared_ptr<CSpeedTreeWrapper>` için bir takma ad. Bir `CSpeedTreeWrapper` nesnesinin (ana model veya örnek) sahipliğini paylaşır.
        *   `SpeedTreeWrapperWeakPtr`: `std::weak_ptr<CSpeedTreeWrapper>` için bir takma ad. Döngüsel bağımlılıkları kırmak için kullanılır.
        *   `TTreeMap`: `std::map<DWORD, SpeedTreeWrapperPtr>` için bir takma ad. Ana ağaç modellerini, genellikle dosya adlarından hesaplanan bir CRC (Cyclic Redundancy Check) anahtarıyla eşleyerek saklar.
    *   **Ana Metotlar:**
        *   `CSpeedTreeForest()`: Kurucu metot. Başlangıç ayarlarını yapar (örneğin, `CSpeedTreeRT::SetNumWindMatrices`, orman sınırları).
        *   `~CSpeedTreeForest()`: Yıkıcı metot. `Clear()` çağırarak kaynakları serbest bırakır.
        *   `GetMainTree(DWORD dwCRC, SpeedTreeWrapperPtr& ppMainTree, const char* c_pszFileName)`: Belirtilen CRC'ye ve dosya adına sahip ana ağaç modelini (`CSpeedTreeWrapper`) yükler. Eğer ağaç zaten `m_pMainTreeMap` içinde varsa, onu döndürür; yoksa dosyadan (`.spt`) yükler, `m_pMainTreeMap`'e ekler ve döndürür. Yükleme `CEterPackManager` üzerinden yapılır.
        *   `GetMainTree(DWORD dwCRC)`: Sadece CRC kullanarak `m_pMainTreeMap` içinden bir ana ağaç modeli arar ve varsa döndürür.
        *   `CreateInstance(float x, float y, float z, DWORD dwTreeCRC, const char* c_szTreeName)`: Belirtilen CRC'ye sahip ana ağaç modelinden yeni bir örnek (`CSpeedTreeWrapper` instance) oluşturur. Bu örneğin pozisyonunu (x, y, z) ayarlar ve sınır küresini (`RegisterBoundingSphere`) kaydeder.
        *   `DeleteInstance(SpeedTreeWrapperPtr pTree)`: Verilen ağaç örneğini siler. Önce örneğin ait olduğu ana modeli bulur, sonra ana model üzerinden örneği kaldırır.
        *   `UpdateSystem(float fCurrentTime)`: Tüm SpeedTree sistemi için genel güncellemeleri yapar. `CSpeedTreeRT::SetTime()` ile SpeedTree motoruna geçen süreyi bildirir ve `SetupWindMatrices()` ile rüzgar matrislerini günceller.
        *   `Clear()`: `m_pMainTreeMap`'deki tüm ana ağaç modellerini temizler, bu da dolaylı olarak tüm örneklerin silinmesine yol açar (shared_ptr kullanımı sayesinde).
        *   `SetLight(const float* afDirection, const float* afAmbient, const float* afDiffuse)`: Ormandaki tüm ağaçlar için genel aydınlatma parametrelerini (ışık yönü, ortam (ambient) rengi, yaygın (diffuse) renk) ayarlar. Bu değerler `m_afLighting` üye değişkeninde saklanır.
        *   `SetFog(float fFogNear, float fFogFar)`: Orman için sis parametrelerini (sisin başladığı ve bittiği mesafeler) ayarlar. Değerler `m_afFog` üye değişkeninde saklanır.
        *   `GetExtents() const`: Ormanın X, Y, Z eksenlerindeki minimum ve maksimum sınırlarını (`m_afForestExtents`) döndürür.
        *   `GetWindStrength() const`, `SetWindStrength(float fStrength)`: Rüzgarın mevcut şiddetini alır veya ayarlar. `SetWindStrength` çağrıldığında, tüm mevcut ağaç örneklerinin (`CSpeedTreeRT::SetWindStrength` aracılığıyla) rüzgar şiddeti güncellenir.
        *   `SetupWindMatrices(float fTimeInSecs)`: Geçen zamana bağlı olarak rüzgar animasyonu için kullanılacak dönme matrislerini hesaplar. Bu matrisler daha sonra ya CPU (`CSpeedTreeRT::SetWindMatrix`) ya da GPU (`UploadWindMatrix`) tarafında rüzgar efektini uygulamak için kullanılır.
    *   **Grafik API Bağımlı Sanal Metotlar:**
        *   `virtual void UploadWindMatrix(unsigned int uiLocation, const float* pMatrix) const = 0`: Hesaplanan bir rüzgar matrisini, belirtilen shader sabit konumuna (register) yüklemek için soyut bir metottur. Bu metodun, kullanılan grafik API'sine (örneğin DirectX) özel olarak türetilmiş bir sınıfta implemente edilmesi gerekir.
        *   `virtual void Render(unsigned long ulRenderBitVector) = 0`: Ormandaki ağaçları, verilen render bit vektörüne (dallar, yapraklar, billboardlar vb. neyin çizileceğini belirten) göre çizmek için soyut bir metottur. Bu da grafik API'sine özel olarak implemente edilmelidir.
    *   **Korunan (Protected) Üyeler:**
        *   `m_pMainTreeMap` (TTreeMap): Yüklenmiş olan ana ağaç modellerini (şablonları) CRC anahtarlarıyla eşleştirerek saklar.
        *   `m_afLighting[12]`: Aydınlatma parametrelerini (yön/pozisyon, ambient, diffuse) tutan bir dizi.
        *   `m_afFog[4]`: Sis parametrelerini (yakın, uzak, ölçek) tutan bir dizi.
    *   **Özel (Private) Metot ve Üyeler:**
        *   `AdjustExtents(float x, float y, float z)`: Yeni bir ağaç eklendiğinde veya bir ağacın pozisyonu değiştiğinde, ormanın genel minimum ve maksimum sınırlarını (`m_afForestExtents`) günceller.
        *   `m_afForestExtents[6]`: Ormanın minimum ve maksimum X, Y, Z koordinatlarını saklar ([minX, minY, minZ, maxX, maxY, maxZ]).
        *   `m_fWindStrength` (float): Rüzgarın genel şiddetini tutar (0.0: rüzgar yok, 1.0: tam güçte).
        *   `m_fAccumTime` (float): `UpdateSystem` çağrıları arasında geçen toplam süreyi biriktirir, rüzgar matrisi hesaplamalarında kullanılır.

*   **`SpeedTreeForest.cpp` - Uygulama Detayları:**
    *   **Ağaç Yönetimi:**
        *   `GetMainTree`: Ana ağaç modelini dosya adından CRC üreterek veya doğrudan CRC ile haritadan (`m_pMainTreeMap`) alır. Yoksa, `CEterPackManager` aracılığıyla `.spt` dosyasını yükler, `CSpeedTreeWrapper::LoadTree` ile ağaç verisini işler ve haritaya ekler.
        *   `CreateInstance`: `GetMainTree` ile ana modeli alır, `CSpeedTreeWrapper::MakeInstance` ile bir örnek oluşturur, pozisyonunu ayarlar ve `RegisterBoundingSphere` çağırır.
        *   `DeleteInstance`: Verilen örneğin ait olduğu ana ağaç modelini bulur (`InstanceOf()`) ve ana model üzerinden örneği kaldırır.
    *   **Sistem Güncellemesi:**
        *   `UpdateSystem`: `CSpeedTreeRT::SetTime()` ile SpeedTree motoruna geçen süreyi iletir. `SetupWindMatrices()` ile rüzgar animasyon matrislerini günceller.
    *   **Rüzgar Efektleri:**
        *   `SetWindStrength`: Yeni rüzgar şiddeti farklıysa, haritadaki tüm ana ağaçlar ve onların tüm örnekleri için `CSpeedTreeRT::SetWindStrength` çağırarak rüzgar şiddetini günceller.
        *   `SetupWindMatrices`: `SpeedTreeConfig.h` içinde tanımlanan `c_nNumWindMatrices` kadar rüzgar matrisi hesaplar. Her matris için, `sinf` ve `cosf` fonksiyonları ile farklı frekanslarda zamanla değişen salınım hareketleri oluşturur. Rüzgar şiddeti (`m_fWindStrength`) arttıkça, salınımın temel açısı (`fBaseAngle`) ve frekansı (`fBaseFreq`) artar. Oluşturulan X ve Y eksenlerindeki dönme matrisleri birleştirilerek nihai rüzgar matrisi elde edilir. Bu matris, `WRAPPER_USE_CPU_WIND` tanımlıysa doğrudan `CSpeedTreeRT::SetWindMatrix` ile SpeedTree motoruna iletilir veya `WRAPPER_USE_GPU_WIND` tanımlıysa `UploadWindMatrix` sanal metodu aracılığıyla shader'a yüklenmek üzere hazırlanır. (Ancak, Metin2 istemcisindeki `SpeedTreeConfig.h` ayarlarında `WRAPPER_USE_NO_WIND` tanımlı olduğu için bu rüzgar hesaplamaları pratikte kullanılmayabilir veya farklı bir yolla yönetiliyor olabilir.)
    *   **Aydınlatma ve Sis:**
        *   `SetLight` ve `SetFog`: Aldıkları parametreleri doğrudan ilgili üye dizilere (`m_afLighting`, `m_afFog`) atarlar. Bu değerler, muhtemelen `Render` metodunun grafik API'ye özel implementasyonunda shader'lara aktarılır.
    *   **Sınır Yönetimi:**
        *   `AdjustExtents`: Verilen bir noktanın koordinatlarına göre ormanın genel minimum ve maksimum sınırlarını (`m_afForestExtents`) günceller.

*   **Kullanım Amacı:**
    *   Sahnedeki tüm SpeedTree ağaçlarının merkezi bir noktadan yüklenmesini, örneklenmesini ve yönetilmesini sağlamak.
    *   Ağaçlar için rüzgar, LOD, ışık ve sis gibi global efektleri ve parametreleri koordine etmek.
    *   Grafik API'sine özel render ve kaynak yönetimi işlemlerini soyutlayarak farklı render altyapılarına (örn: DirectX, OpenGL) uyarlanabilir bir yapı sunmak.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (SpeedTreeLib için)
    *   `SpeedTreeRT.h` (SpeedTree çalışma zamanı kütüphanesi)
    *   `SpeedTreeWrapper.h` (`CSpeedTreeWrapper` sınıfı için)
    *   `SpeedTreeConfig.h` (Yapılandırma sabitleri için)
    *   `../EterBase/Filename.h`, `../EterBase/MappedFile.h` (Dosya işlemleri için)
    *   `../EterPack/EterPackManager.h` (Paketlenmiş dosyalara erişim için)
    *   Standart C++ kütüphaneleri (`vector`, `map`, `memory` (shared_ptr, weak_ptr), `cfloat` (FLT_MAX)). 

### `SpeedTreeForestDirectX8.h` ve `SpeedTreeForestDirectX8.cpp` (`CSpeedTreeForestDirectX8` Sınıfı)

*   **Amaç:** `CSpeedTreeForestDirectX8` sınıfı, `CSpeedTreeForest` temel sınıfının DirectX 8 API'si için özelleştirilmiş bir uygulamasıdır. Temel sınıfın soyut (virtual) olarak tanımladığı render (`Render`) ve rüzgar matrisi yükleme (`UploadWindMatrix`) gibi grafik API'sine bağımlı işlevleri DirectX 8 komutlarını kullanarak gerçekleştirir. Ayrıca, SpeedTree için gerekli olan dal (branch) ve yaprak (leaf) vertex shader'larını yüklemek ve yönetmekten sorumludur. Bu sınıf, Metin2 istemcisindeki SpeedTree entegrasyonunun DirectX 8 render altyapısıyla doğrudan etkileşim kuran katmanıdır ve `CSingleton` tasarım deseni kullanılarak oyun boyunca tek bir örneğinin olması sağlanır.

*   **`SpeedTreeForestDirectX8.h` - Temel Tanımlar ve Arayüz:**
    *   **Kalıtım:**
        *   `public CSpeedTreeForest`: Orman yönetimiyle ilgili temel işlevselliği (ağaç yükleme, örnekleme, genel rüzgar/ışık/sis yönetimi vb.) miras alır.
        *   `public CGraphicBase`: Metin2'nin `EterLib` kütüphanesindeki `CGraphicBase` sınıfından türeyerek temel grafiksel nesne özelliklerini veya arayüzlerini alabilir.
        *   `public CSingleton<CSpeedTreeForestDirectX8>`: Singleton deseni implementasyonunu sağlar, böylece bu sınıftan global olarak erişilebilen tek bir örnek (`CSpeedTreeForestDirectX8::Instance()`) oluşturulur.
    *   **Ana Metotlar:**
        *   `CSpeedTreeForestDirectX8()`: Kurucu. DirectX cihaz işaretçisini (`m_pDx`) ve shader tanıtıcılarını (`m_dwBranchVertexShader`, `m_dwLeafVertexShader`) sıfırlar.
        *   `virtual ~CSpeedTreeForestDirectX8()`: Yıkıcı.
        *   `UploadWindMatrix(unsigned int uiLocation, const float* pMatrix) const override`: `CSpeedTreeForest` temel sınıfındaki sanal metodu ezer. Verilen rüzgar matrisini (`pMatrix`), belirtilen shader sabit konumuna (`uiLocation`) Metin2'nin `STATEMANAGER`'ı aracılığıyla `SetVertexShaderConstant` kullanarak yükler.
        *   `UpdateCompundMatrix(const D3DXVECTOR3& c_rEyeVec, const D3DXMATRIX& c_rmatView, const D3DXMATRIX& c_rmatProj)`: Kamera pozisyonu (`c_rEyeVec`), görüş (view) matrisi (`c_rmatView`) ve projeksiyon matrisini (`c_rmatProj`) kullanarak birleşik bir dünya-görüş-projeksiyon matrisi oluşturur. Bu matrisi hem SpeedTree motorunun kamera bilgisi için (`CSpeedTreeRT::SetCamera`) hem de vertex shader'daki ilgili sabit register (`c_nVertexShader_CompoundMatrix`) için ayarlar.
        *   `Render(unsigned long ulRenderBitVector = Forest_RenderAll) override`: `CSpeedTreeForest` temel sınıfındaki sanal metodu ezer. Belirtilen `ulRenderBitVector` maskesine göre (dallar, yapraklar, billboardlar vb.) ormanı DirectX 8 API'sini kullanarak çizer. Bu metot, render durumlarını yönetir, shader'ları ayarlar ve her bir ağaç örneğinin çizim fonksiyonlarını çağırır.
        *   `SetRenderingDevice(LPDIRECT3DDEVICE8 pDevice)`: Kullanılacak DirectX 8 render cihazını (`LPDIRECT3DDEVICE8`) ayarlar. Ayrıca, `InitVertexShaders()` fonksiyonunu çağırarak SpeedTree için gerekli vertex shader'ları yükler ve temel SpeedTree ışıklandırma ayarlarını (`CSpeedTreeRT::SetLightAttributes`, `CSpeedTreeRT::SetLightState`) yapar.
    *   **Özel (Private) Metotlar ve Üyeler:**
        *   `InitVertexShaders()`: Dal ve yapraklar için özel vertex shader'ları (`LoadBranchShader`, `LoadLeafShader` ile) yükler ve bu shader'ların tanıtıcılarını `CSpeedTreeWrapper` sınıfına bildirir.
        *   `m_pDx` (LPDIRECT3DDEVICE8): Saklanan DirectX 8 cihaz işaretçisi.
        *   `m_dwBranchVertexShader` (DWORD): Yüklenen dal/frond vertex shader'ının DirectX tanıtıcısı.
        *   `m_dwLeafVertexShader` (DWORD): Yüklenen yaprak vertex shader'ının DirectX tanıtıcısı.

*   **`SpeedTreeForestDirectX8.cpp` - Uygulama Detayları:**
    *   **Shader Yönetimi (`InitVertexShaders`, `SetRenderingDevice`):**
        *   `SetRenderingDevice` çağrıldığında, öncelikle DirectX 8 cihazı saklanır.
        *   `InitVertexShaders` tetiklenir. Bu fonksiyon, `VertexShaders.h` (veya ilişkili .cpp) içinde tanımlı olan `LoadBranchShader` ve `LoadLeafShader` fonksiyonlarını kullanarak önceden derlenmiş veya tanımlanmış olan vertex shader bytecode'larını yükler. Yüklenen shader'ların tanıtıcıları `m_dwBranchVertexShader` ve `m_dwLeafVertexShader` üyelerinde saklanır.
        *   Bu shader tanıtıcıları daha sonra `CSpeedTreeWrapper::SetVertexShaders` aracılığıyla her bir ağaç tipine atanır.
        *   `SetRenderingDevice` ayrıca, `SpeedTreeConfig.h`'den alınan varsayılan değerlerle `CSpeedTreeRT` için temel bir ışık kaynağı (0 numaralı) ayarlar.
    *   **Matris Yönetimi (`UploadWindMatrix`, `UpdateCompundMatrix`):**
        *   `UploadWindMatrix`: Temel sınıftan gelen rüzgar matrislerini, `STATEMANAGER.SetVertexShaderConstant` kullanarak doğrudan vertex shader sabitlerine yükler. Bu, rüzgar efektlerinin GPU üzerinde hesaplanmasını sağlar (eğer `WRAPPER_USE_GPU_WIND` aktifse).
        *   `UpdateCompundMatrix`: Kamera ve projeksiyon matrislerini birleştirip transpoze ederek vertex shader'daki `c_nVertexShader_CompoundMatrix` sabitine yükler. Ayrıca, SpeedTreeRT motorunun kamera pozisyonu ve yönünü bilmesi için `CSpeedTreeRT::SetCamera` çağrılır; bu, LOD hesaplamaları ve billboard'ların doğru yönlendirilmesi için kritiktir.
    *   **Render İşlemi (`Render`):**
        *   **Başlangıç:** Temel sınıfın `UpdateSystem`'i çağrılarak zaman ve rüzgar güncellenir. Eğer çizilecek ağaç yoksa işlem sonlanır. Gölge veya minimap render'ı değilse `UpdateCompundMatrix` çağrılır.
        *   **Durum Saklama ve Ayarlama:** Mevcut DirectX render durumları (`D3DRS_LIGHTING`, `D3DRS_COLORVERTEX`, `D3DRS_FOGVERTEXMODE`) saklanır. `SpeedTreeConfig.h`'deki aydınlatma moduna göre (`WRAPPER_USE_DYNAMIC_LIGHTING`) bu durumlar ayarlanır.
        *   **Ağaç Güncelleme:** Tüm ağaç örnekleri üzerinde `pInstance->Advance()` çağrılarak SpeedTreeRT'nin kendi iç güncellemelerini (LOD vb.) yapması sağlanır.
        *   **Shader Sabitleri:** Temel sınıftaki `m_afLighting` ve `m_afFog` dizileri kullanılarak ışık ve sis parametreleri shader'lara yüklenir.
        *   **Texture Stage ve Render Durumları:** Çizim türüne (normal, gölge) göre DirectX texture stage state'leri (`D3DTSS_COLOROP`, `D3DTSS_ALPHAOP`, filtreleme vb.) ve render state'leri (`D3DRS_ALPHATESTENABLE`, `D3DRS_ALPHAFUNC`, `D3DRS_CULLMODE`) dikkatlice ayarlanır. Örneğin, gölge çiziminde genellikle alfa testi ve basit doku modülasyonu kullanılırken, normal çizimde daha karmaşık modülasyonlar ve filtrelemeler aktif olabilir.
        *   **Ağaç Bileşenlerinin Çizimi:**
            1.  **Dallar (Branches):** `m_dwBranchVertexShader` aktif edilir. `Forest_RenderBranches` biti ayarlıysa, her ağaç tipi için `SetupBranchForTreeType` çağrılır ve sonra görünür olan her örneğin `RenderBranches` metodu çağrılarak dallar çizilir.
            2.  **Frondlar (Fronds):** Culling kapatılır (`D3DCULL_NONE`). `Forest_RenderFronds` biti ayarlıysa, her ağaç tipi için `SetupFrondForTreeType` çağrılır ve sonra görünür olan her örneğin `RenderFronds` metodu çağrılarak frondlar çizilir.
            3.  **Yapraklar (Leaves):** `m_dwLeafVertexShader` aktif edilir. `Forest_RenderLeaves` biti ayarlıysa, her ağaç tipi için `SetupLeafForTreeType` çağrılır, görünür olan her örneğin `RenderLeaves` metodu çağrılır ve son olarak her ağaç tipi için `EndLeafForTreeType` çağrılır. Gölge/minimap çizimi için alfa referans değeri özel olarak ayarlanabilir.
            4.  **Billboardlar (Billboards):** `WRAPPER_NO_BILLBOARD_MODE` tanımlı değilse ve `Forest_RenderBillboards` biti ayarlıysa, ışıklandırma ve vertex renkleri kapatılır. Her ağaç tipi için `SetupBranchForTreeType` (billboard'lar genellikle dal dokularını kullanır) çağrılır ve sonra görünür olan her örneğin `RenderBillboards` metodu çağrılır.
        *   **Durum Geri Yükleme:** Çizim işlemi bittikten sonra, başlangıçta saklanan DirectX render durumları geri yüklenir.

*   **Kullanım Amacı:**
    *   `CSpeedTreeForest` sınıfının soyut grafik işlemlerini DirectX 8 API'sine özgü komutlarla implemente etmek.
    *   SpeedTree ağaçlarının (dallar, yapraklar, frondlar, billboardlar) DirectX 8 ortamında doğru şekilde render edilmesini sağlamak.
    *   Vertex shader'ların yüklenmesi, ayarlanması ve SpeedTree tarafından kullanılmasını yönetmek.
    *   Rüzgar, ışık, sis ve kamera matrisleri gibi verilerin shader'lara aktarılmasını koordine etmek.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (SpeedTreeLib için)
    *   `d3d8.h`, `d3d8types.h`, `d3dx8.h` (DirectX 8 kütüphaneleri)
    *   `../EterBase/Timer.h` (`CTimer` için)
    *   `../EterLib/StateManager.h` (DirectX render durumlarını yönetmek için)
    *   `../EterLib/Camera.h` (`CCameraManager` ve `CCamera` için)
    *   `SpeedTreeForest.h` (Temel sınıf)
    *   `SpeedTreeConfig.h` (Yapılandırma sabitleri)
    *   `VertexShaders.h` (Shader yükleme fonksiyonları için, örneğin `LoadBranchShader`, `LoadLeafShader`)
    *   `CSingleton` (EterLib veya EterBase içinde tanımlı olabilir)
    *   `CGraphicBase` (EterLib içinde tanımlı olabilir)

### `SpeedTreeMaterial.h` (`CSpeedTreeMaterial` Sınıfı)

*   **Amaç:** `CSpeedTreeMaterial` sınıfı, SpeedTree ağaç bileşenlerinin (dallar, yapraklar, frondlar) DirectX 8'de render edilirken kullanılacak materyal özelliklerini (`D3DMATERIAL8`) yönetmek için basit bir sarmalayıcı (wrapper) sınıftır. SpeedTreeRT kütüphanesinden alınan materyal verilerini (ambient, diffuse, specular, emissive renkleri ve parlaklık gücü) standart bir `D3DMATERIAL8` yapısına kolayca aktarmak ve bu yapıya erişim sağlamak için kullanılır.

*   **Temel Özellikler ve Metotlar:**
    *   **`D3DMATERIAL8 m_cMaterial`:** Sınıfın temelini oluşturan DirectX 8 materyal yapısıdır. Ambient, Diffuse, Specular, Emissive renklerini (her biri RGBA `D3DCOLORVALUE`) ve Specular Power (parlaklık keskinliği) değerini içerir.
    *   **`CSpeedTreeMaterial()` (Yapıcı):** `m_cMaterial` üyesinin tüm renk bileşenlerini varsayılan olarak tam beyaz (R,G,B,A = 1.0f) ve `Power` değerini 5.0f olarak başlatır.
    *   **`Set(const float* pMaterialArray)`:** SpeedTreeRT motorundan genellikle 13 float'lık bir dizi halinde alınan materyal verilerini `m_cMaterial` yapısına kopyalar:
        *   `pMaterialArray[0..2]`: Diffuse renk (RGB)
        *   `pMaterialArray[3..5]`: Ambient renk (RGB)
        *   `pMaterialArray[6..8]`: Specular renk (RGB)
        *   `pMaterialArray[9..11]`: Emissive renk (RGB)
        *   `pMaterialArray[12]`: Specular Power (Parlaklık gücü)
        *   Tüm renklerin alfa (A) bileşeni varsayılan olarak 1.0f (tam opak) ayarlanır.
    *   **`D3DMATERIAL8* Get()`:** Saklanan `m_cMaterial` yapısının adresini döndürür. Bu, `IDirect3DDevice8::SetMaterial()` fonksiyonuna doğrudan parametre olarak verilebilir.

*   **Kullanım Amacı:**
    *   SpeedTreeRT'den gelen materyal bilgilerini DirectX 8'in anlayacağı bir formata dönüştürmek ve saklamak.
    *   Her ağaç bileşeni (dal, yaprak, frond) için ayrı materyal özelliklerinin kolayca ayarlanmasını ve render sırasında kullanılmasını sağlamak.

*   **Bağımlılıklar:**
    *   `d3d8.h`, `d3d8types.h`, `d3dx8.h` (DirectX 8 temel başlık dosyaları).
    *   `cstring` (`memcpy` için, dolaylı olarak dahil edilmiş olabilir).

### `SpeedTreeWrapper.h` ve `SpeedTreeWrapper.cpp` (`CSpeedTreeWrapper` Sınıfı)

*   **Amaç:** `CSpeedTreeWrapper` sınıfı, SpeedTreeRT kütüphanesi ile Metin2 istemcisinin DirectX 8 tabanlı render motoru arasında bir köprü görevi görerek tek bir SpeedTree ağacını (bu bir ana şablon ağaç veya bu şablondan oluşturulmuş bir örnek/instance olabilir) sarmalar ve yönetir. Ağacın `.spt` dosyasından yüklenmesi, geometrisinin (dallar, yapraklar, frondlar, billboard'lar) oluşturulup DirectX vertex ve index buffer'larına aktarılması, her bir ağaç bileşeninin uygun materyal ve dokularla render edilmesi, ağaç örneklerinin (instance) verimli bir şekilde oluşturulup yönetilmesi, Seviye Detay (LOD) hesaplamalarının yapılması, rüzgar efektlerinin uygulanması ve çarpışma verilerinin sağlanması gibi temel işlevleri yerine getirir. Aynı zamanda, Metin2'nin `CGraphicObjectInstance` sınıfından türeyerek oyun dünyasındaki diğer grafik nesneleriyle tutarlı bir arayüz sunar ve genel render döngüsüne entegre olur.

*   **`SpeedTreeWrapper.h` - Temel Tanımlar ve Arayüz:**
    *   **Kalıtım:**
        *   `public CGraphicObjectInstance`: Oyun dünyasında bir grafik nesnesi olarak temel işlevleri (pozisyon, dönüşüm, BBox, render metodları) miras alır. `OnRender`, `SetPosition`, `CalculateBBox`, `GetBoundingSphere`, `OnUpdateCollisionData` gibi sanal metodları SpeedTree'ye özel olarak implemente eder.
        *   `public std::enable_shared_from_this<CSpeedTreeWrapper>`: `shared_ptr` mekanizmasıyla güvenli kendini işaret etme (self-pointing) sağlar, özellikle instancing yönetiminde önemlidir.
    *   **Statik Üyeler:**
        *   `ms_dwBranchVertexShader` (DWORD), `ms_dwLeafVertexShader` (DWORD): Tüm ağaç örnekleri tarafından paylaşılan, sırasıyla dal/frond ve yaprak geometrileri için kullanılacak olan DirectX vertex shader'larının tanıtıcılarını (handle) tutar. Bu değerler genellikle `CSpeedTreeForestDirectX8::SetRenderingDevice` içinde yüklenip bu sınıfa bildirilir.
        *   `ms_bSelfShadowOn` (bool): Ağaçların kendi üzerine gölge (self-shadow) efektinin genel olarak aktif olup olmadığını kontrol eden bir bayraktır.
    *   **Temel Metotlar:**
        *   `CSpeedTreeWrapper()`: Kurucu. Yeni bir `CSpeedTreeRT` çekirdek nesnesi oluşturur, başlangıç pozisyonu, rüzgar ve yerel matris ayarlarını yapar.
        *   `virtual ~CSpeedTreeWrapper()`: Yıkıcı. Eğer bu bir ana modelse (instance değilse), oluşturduğu tüm DirectX buffer'larını, `m_pTextureInfo` ve `m_pGeometryCache` gibi dinamik bellek alanlarını serbest bırakır. Her durumda `m_pSpeedTree` nesnesini siler.
        *   `LoadTree(const char* pszSptFile, ...)`: Bir `.spt` ağaç tanım dosyasını yükler. SpeedTreeRT motorunu kullanarak ağaç verisini işler, `SpeedTreeConfig.h`'deki ayarlara göre (doku koordinatları, aydınlatma, rüzgar) yapılandırır, ağaç geometrisini (`CSpeedTreeRT::Compute`) oluşturur, LOD sınırlarını ve materyallerini (`CSpeedTreeMaterial` kullanarak) ayarlar, gerekli dokuları (`CGraphicImageInstance` kullanarak) yükler ve son olarak `SetupBuffers()` metodunu çağırarak DirectX için vertex/index buffer'larını hazırlar.
        *   `GetBoundingBox() const`: Ağacın SpeedTreeRT tarafından hesaplanan ham 6 float'lık (minX, minY, minZ, maxX, maxY, maxZ) sınırlayıcı kutusunu döndürür.
        *   `GetCollisionObjectCount()`, `GetCollisionObject(...)`: SpeedTreeRT'den ağacın çarpışma geometrisini (küreler, silindirler) sorgulamak için kullanılır.
        *   `Advance()`: Ağacın LOD seviyesini kamera mesafesine göre hesaplar (`m_pSpeedTree->ComputeLodLevel()`) ve eğer CPU tabanlı rüzgar aktifse rüzgar efektlerini (`m_pSpeedTree->ComputeWindEffects()`) günceller.
        *   `MakeInstance()`: Mevcut `CSpeedTreeWrapper` nesnesinden (bu bir ana model olmalıdır) bir örnek (instance) oluşturur. Yeni örnek, ana modelin `CSpeedTreeRT` nesnesinin bir kopyasını (`m_pSpeedTree->MakeInstance()`) alır ve ana modelin materyal, doku, geometri önbelleği ve DirectX buffer işaretçilerini paylaşır. Bu, bellek kullanımını ve yükleme süresini önemli ölçüde azaltır.
        *   `DeleteInstance(SpeedTreeWrapperPtr pInstance)`: Bu ana modele ait belirtilen örneği siler.
        *   `GetSpeedTree() const`: Sarmalanan `CSpeedTreeRT` nesnesine bir işaretçi döndürür.
        *   `SetWindStrength(float fStrength)`: Ağacın rüzgar hassasiyetini ayarlar.
    *   **Render ile İlgili Metotlar:**
        *   `SetupBranchForTreeType() const`, `SetupFrondForTreeType() const`, `SetupLeafForTreeType() const`: Belirli bir ağaç bileşeninin (dal, frond, yaprak) render edilmesinden önce çağrılır. İlgili materyali (`STATEMANAGER.SetMaterial`), dokuları (`STATEMANAGER.SetTexture`) ve vertex/index buffer'larını (`STATEMANAGER.SetStreamSource`, `STATEMANAGER.SetIndices`) ayarlar. Gerekirse CPU tabanlı rüzgar için vertex buffer'ı günceller.
        *   `EndLeafForTreeType()`: Yaprakların render'ı bittikten sonra, CPU ile güncellenen yaprak LOD'ları için kullanılan bayrakları sıfırlar.
        *   `RenderBranches() const`, `RenderFronds() const`, `RenderLeaves() const`, `RenderBillboards() const`: İlgili ağaç bileşenini DirectX `DrawIndexedPrimitive` veya `DrawPrimitive(UP)` komutlarıyla çizer. Çizimden önce `PositionTree()` ile ağacın dünya matrisini ayarlar ve `m_pSpeedTree->GetGeometry()` ile SpeedTreeRT'den güncel geometri bilgilerini (aktif LOD, alfa test değeri vb.) alır.
    *   **Özel (Private) Metotlar:**
        *   `SetupBuffers()`: `SetupBranchBuffers`, `SetupFrondBuffers`, `SetupLeafBuffers` alt metotlarını çağırarak tüm ağaç bileşenleri için DirectX vertex ve index buffer'larını oluşturur ve doldurur.
        *   `PositionTree() const`: Ağacın mevcut pozisyonuna göre bir dünya dönüşüm matrisi oluşturur ve bunu DirectX'e (`D3DTS_WORLD`) ve vertex shader sabitine (`c_nVertexShader_TreePos`) yükler.
        *   `LoadTexture(const char* pFilename, CGraphicImageInstance& rImage)`: `CResourceManager` kullanarak belirtilen dosyadan bir doku yükler ve verilen `CGraphicImageInstance` nesnesine atar.
        *   `SetShaderConstants(const float* pMaterial) const`: Verilen materyal dizisini ve ağacın yaprak aydınlatma ayarını ilgili vertex shader sabitlerine (`c_nVertexShader_Material`, `c_nVertexShader_LeafLightingAdjustment`) yükler.
    *   **Üye Değişkenler (Önemlileri):**
        *   `m_pSpeedTree` (CSpeedTreeRT*): SpeedTreeRT motorunun çekirdek nesnesi.
        *   `m_bIsInstance` (bool): Bu nesnenin bir ana model mi yoksa bir örnek mi olduğunu belirtir.
        *   `m_vInstances` (std::vector<SpeedTreeWrapperPtr>): Eğer ana modelse, oluşturulan örneklerin bir listesi.
        *   `m_pInstanceOf` (SpeedTreeWrapperPtr): Eğer bir örnekse, ait olduğu ana modele işaretçi.
        *   `m_pGeometryCache` (CSpeedTreeRT::SGeometry*): SpeedTreeRT'den alınan geometri verileri için bir önbellek.
        *   **DirectX Buffer'ları:** `m_pBranchVertexBuffer`, `m_pBranchIndexBuffer`, `m_pFrondVertexBuffer`, `m_pFrondIndexBuffer`, `m_pLeafVertexBuffer` (LOD seviyelerine göre bir dizi).
        *   `m_afPos[3]`: Ağacın dünya koordinatlarındaki pozisyonu.
        *   `m_afBoundingBox[6]`: Ağacın ham sınırlayıcı kutusu.
        *   `m_cBranchMaterial`, `m_cLeafMaterial`, `m_cFrondMaterial` (`CSpeedTreeMaterial` türünde): Ağacın farklı kısımları için materyal tanımları.
        *   `m_BranchImageInstance`, `m_ShadowImageInstance`, `m_CompositeImageInstance` (`CGraphicImageInstance` türünde): Sırasıyla dal, kendi üzerine düşen gölge ve kompozit (yapraklar/frondlar için ana) dokuları temsil eder.

*   **`SpeedTreeWrapper.cpp` - Uygulama Detayları:**
    *   **Buffer Kurulumu (`SetupBranchBuffers`, `SetupFrondBuffers`, `SetupLeafBuffers`):**
        *   Her ağaç bileşeni (dal, frond, yaprak) için DirectX vertex buffer'ları (ve dallar/frondlar için index buffer'ları) oluşturulur. Vertex formatları (`D3DFVF_SPEEDTREE_BRANCH_VERTEX`, `D3DFVF_SPEEDTREE_LEAF_VERTEX`) `VertexShaders.h` içinde tanımlanmıştır.
        *   Buffer'lar, SpeedTreeRT'den alınan geometri verileriyle (pozisyon, normal/renk, doku koordinatları, rüzgar ağırlıkları, GPU yaprak yerleşim indeksleri vb.) doldurulur.
        *   `WRAPPER_USE_CPU_WIND` veya `WRAPPER_USE_CPU_LEAF_PLACEMENT` tanımlıysa, ilgili buffer'lar dinamik (`D3DUSAGE_DYNAMIC`, `D3DPOOL_SYSTEMMEM`) olarak oluşturulur, aksi halde yönetilen (`D3DUSAGE_WRITEONLY`, `D3DPOOL_MANAGED`) bellekten oluşturulur.
        *   Yapraklar için her LOD seviyesine ayrı bir vertex buffer (`m_pLeafVertexBuffer[unLod]`) tahsis edilir.
    *   **Render Akışı (`OnRender`, `OnRenderPCBlocker` ve ilişkili `Render...` metodları):**
        *   `OnRender` (ve benzeri `OnRenderPCBlocker`), `CGraphicObjectInstance`'dan gelen çağrıyla tetiklenir.
        *   Gerekirse shader'ları yükler (`LoadBranchShader`, `LoadLeafShader`).
        *   `CSpeedTreeForestDirectX8::Instance().UpdateSystem()` ve `UpdateCompundMatrix()` çağrılarak genel sistem ve matrisler güncellenir.
        *   DirectX render durumları (`STATEMANAGER` aracılığıyla D3DRS ve D3DTSS) dikkatlice ayarlanır (ışıklandırma, alfa testi, culling, doku filtreleme, karıştırma modları).
        *   Sırasıyla dallar, frondlar, yapraklar ve billboardlar için `Setup...ForTreeType` ve ardından `Render...` metodları çağrılır.
        *   `Render...` metodları, `m_pSpeedTree->GetGeometry()` ile güncel LOD bilgisini alır, `PositionTree()` ile ağacın dünya matrisini ayarlar ve `STATEMANAGER.DrawIndexedPrimitive` veya `DrawPrimitive(UP)` ile çizimi gerçekleştirir.
        *   CPU tabanlı rüzgar veya yaprak yerleşimi kullanılıyorsa (`#ifdef WRAPPER_USE_CPU_WIND` vb.), ilgili `Setup...ForTreeType` veya `RenderLeaves` metodlarında vertex buffer'lar `Lock()` edilerek güncellenir.
    *   **Instancing (`MakeInstance`):**
        *   Yeni bir `CSpeedTreeWrapper` nesnesi oluşturulur.
        *   `m_pSpeedTree->MakeInstance()` ile yeni bir `CSpeedTreeRT` çekirdek örneği elde edilir.
        *   Yeni örnek, ana modelin materyal, doku (`SetImagePointer` ile referans kopyalama), geometri önbelleği ve DirectX buffer işaretçilerini devralır. Bu, bellek ve kaynak paylaşımını sağlar.
    *   **Çarpışma (`OnUpdateCollisionData`):**
        *   SpeedTreeRT'den alınan (`GetCollisionObject`) küre ve silindir şeklindeki çarpışma nesneleri, `CStaticCollisionData` yapısına dönüştürülür ve ağacın mevcut pozisyonuna göre transforme edilerek `CGraphicObjectInstance::AddCollision` ile oyunun çarpışma sistemine eklenir.

*   **Kullanım Amacı:**
    *   Her bir SpeedTree ağacını ve örneklerini oyun dünyasında yönetmek.
    *   Ağaç geometrisini SpeedTreeRT motorundan alıp DirectX buffer'larına yüklemek.
    *   Ağaçların LOD, rüzgar, materyal, doku ve shader ayarlarını yöneterek doğru şekilde render edilmesini sağlamak.
    *   Oyunun genel render döngüsüne ve çarpışma sistemine entegre olmak.

*   **Bağımlılıklar:**
    *   `StdAfx.h`
    *   `SpeedTreeRT.h` (SpeedTree çekirdek kütüphanesi)
    *   `SpeedTreeMaterial.h` (`CSpeedTreeMaterial` için)
    *   `SpeedTreeConfig.h` (Yapılandırma sabitleri)
    *   `SpeedTreeForestDirectX8.h` (`CSpeedTreeForestDirectX8` singleton örneğine erişim için)
    *   `VertexShaders.h` (Shader yükleme fonksiyonları ve FVF tanımları için)
    *   `d3d8.h`, `d3d8types.h`, `d3dx8.h` (DirectX 8)
    *   `../EterLib/GrpObjectInstance.h`, `../EterLib/GrpImageInstance.h`, `../EterLib/GrpCollisionObject.h`, `../EterLib/ResourceManager.h`, `../EterLib/Camera.h`, `../EterLib/StateManager.h` (Metin2 EterLib kütüphanesi)
    *   `../EterBase/Debug.h`, `../EterBase/Timer.h`, `../EterBase/Filename.h` (Metin2 EterBase kütüphanesi)
    *   Standart C++ kütüphaneleri (`vector`, `memory`, `list`, `string`).

### `StdAfx.h` ve `StdAfx.cpp` (Ön Derlenmiş Başlık Dosyaları)

*   **Amaç:** `StdAfx.h` ve `StdAfx.cpp` dosyaları, `SpeedTreeLib` kütüphanesi için ön derlenmiş başlık (precompiled header - PCH) mekanizmasını oluşturur. Bu mekanizma, sık kullanılan ve nadiren değişen başlık dosyalarının bir kez derlenip saklanmasını sağlayarak projenin genel derleme süresini önemli ölçüde azaltmayı hedefler.

*   **`StdAfx.h` - İçerik ve Tanımlar:**
    *   Bu dosya, `SpeedTreeLib` içindeki birçok kaynak dosya tarafından ortak olarak kullanılan başlık dosyalarını ve bazı genel tanımlamaları içerir.
    *   **`#define WIN32_LEAN_AND_MEAN`**: Windows.h başlık dosyasından daha az kullanılan API'lerin dışarıda bırakılmasını sağlar, bu da derleme sürelerini kısaltır.
    *   **`#include <assert.h>`**: Hata ayıklama sırasında koşulları kontrol etmek için kullanılan `assert()` makrosunu dahil eder.
    *   **`#include "../UserInterface/Locale_inc.h"`**: Metin2 istemcisinin `UserInterface` modülünden yerelleştirme (dil, bölge ayarları) ile ilgili başlık dosyasını dahil eder. Bu, kütüphane içinde kullanılan metinlerin veya hata mesajlarının farklı dillere çevrilebilmesi için gerekli olabilir.
    *   **`#include "SpeedTreeForestDirectX8.h"`**: `SpeedTreeLib`'in DirectX 8 için özelleştirilmiş ana orman yönetimi sınıfı olan `CSpeedTreeForestDirectX8`'in başlık dosyasını dahil eder. Bu, `StdAfx.h`'yi kullanan tüm dosyaların bu sınıfa erişimini kolaylaştırır.
    *   **NANOBEGIN / NANOEND Makroları**: Bu makrolar, "Armadillo nanomite protection" açıklamasıyla birlikte verilmiştir. Belirli kod bloklarını sarmalamak için kullanılırlar ve içlerinde derleyiciye özel assembly komutları (`__asm _emit`) barındırırlar. Bu tür makrolar genellikle yazılım koruma mekanizmalarında, tersine mühendisliği zorlaştırmak veya çok düşük seviyeli hata ayıklama/profilleme işlemleri için kullanılır. Hem Borland C++ hem de Microsoft C++ derleyicileri için farklı tanımlamalar içerirler.

*   **`StdAfx.cpp` - İşlevi:**
    *   Bu dosyanın tek amacı `StdAfx.h` dosyasını dahil etmektir (`#include "stdafx.h"`).
    *   Derleme sürecinde, derleyici bu `.cpp` dosyasını kullanarak `StdAfx.h` içinde listelenen tüm başlık dosyalarını işler ve bunların ön derlenmiş bir halini (genellikle `.pch` uzantılı bir dosya) oluşturur. Sonraki derlemelerde, diğer kaynak dosyalar doğrudan bu `.pch` dosyasını kullanarak bu başlıklara daha hızlı erişir.

*   **Kullanım Amacı:**
    *   Projenin genel derleme süresini azaltmak.
    *   Sık kullanılan başlık dosyalarını merkezi bir yerde toplamak.

*   **Bağımlılıklar:**
    *   Windows SDK (Windows.h için).
    *   Metin2 `UserInterface` kütüphanesi (`Locale_inc.h`).
    *   `SpeedTreeLib` içindeki `SpeedTreeForestDirectX8.h`.
    *   Derleyiciye özgü assembly komutları (NANOBEGIN/NANOEND için).

### `VertexShaders.h` (SpeedTree Vertex Shader Tanımları ve Yükleyicileri)

*   **Amaç:** Bu başlık dosyası, SpeedTree ağaçlarının (dallar, yapraklar, frondlar) DirectX 8'de render edilmesi için gerekli olan Esnek Vertex Formatlarını (FVF - Flexible Vertex Format), bu formatlara karşılık gelen C++ vertex yapılarını ve bu yapıları işleyen vertex shader programlarının HLSL assembly kodlarını içerir. Ayrıca, bu shader kodlarını derleyip DirectX vertex shader nesneleri oluşturan yardımcı fonksiyonları (`LoadBranchShader`, `LoadLeafShader`) da tanımlar. Bu dosya, SpeedTree geometrisinin nasıl işleneceğini ve GPU'ya nasıl gönderileceğini belirleyen temel bileşenleri merkezi bir yerde toplar.

*   **Temel İçerik ve Yapılar:**
    *   **FVF Tanımları (`D3DFVF_SPEEDTREE_BRANCH_VERTEX`, `D3DFVF_SPEEDTREE_LEAF_VERTEX`):**
        *   Bu `DWORD` sabitleri, DirectX'e bir vertex'in hangi bileşenleri (pozisyon, normal, renk, doku koordinatları vb.) içerdiğini ve bu bileşenlerin sırasını bildirir.
        *   Tanımlar, `SpeedTreeConfig.h` dosyasındaki `#define` makrolarına (örneğin, `WRAPPER_USE_DYNAMIC_LIGHTING`, `WRAPPER_RENDER_SELF_SHADOWS`, `WRAPPER_USE_GPU_WIND`, `WRAPPER_USE_GPU_LEAF_PLACEMENT`) bağlı olarak dinamik olarak değişir. Bu sayede, farklı yapılandırmalara (statik/dinamik aydınlatma, GPU tabanlı rüzgar/yaprak yerleşimi var/yok) göre optimize edilmiş vertex formatları kullanılır.
        *   Örneğin, `WRAPPER_USE_DYNAMIC_LIGHTING` aktifse vertex normali (`D3DFVF_NORMAL`) dahil edilir; değilse, önceden hesaplanmış vertex rengi (`D3DFVF_DIFFUSE`) dahil edilir.
        *   GPU tabanlı özellikler aktifse, ek doku koordinat setleri (`D3DFVF_TEXCOORDSIZE2(n)` veya `D3DFVF_TEXCOORDSIZE4(n)`) aracılığıyla rüzgar ağırlıkları, matris indeksleri veya yaprak yerleşim verileri gibi ek bilgiler vertex'e eklenir.
    *   **Vertex Yapıları (`SFVFBranchVertex`, `SFVFLeafVertex`):**
        *   Bu C++ yapıları (struct), yukarıda tanımlanan FVF'lere tam olarak karşılık gelen veri üyelerini içerir. Bu, C++ kodu tarafında vertex verilerinin kolayca oluşturulmasını ve yönetilmesini sağlar.
        *   `SFVFBranchVertex`: Dalların ve frondların vertex verilerini tutar. Üyeleri: `D3DXVECTOR3 m_vPosition`, (dinamik aydınlatma için `D3DXVECTOR3 m_vNormal` veya statik aydınlatma için `DWORD m_dwDiffuseColor`), `FLOAT m_fTexCoords[2]` (birincil doku), (aktifse `FLOAT m_fShadowCoords[2]` gölge dokusu için) ve (aktifse `FLOAT m_fWindIndex`, `FLOAT m_fWindWeight` GPU rüzgarı için).
        *   `SFVFLeafVertex`: Yaprakların vertex verilerini tutar. Üyeleri: `D3DXVECTOR3 m_vPosition`, normal/renk, birincil doku koordinatları ve (aktifse `FLOAT m_fWindIndex`, `FLOAT m_fWindWeight`, `FLOAT m_fLeafPlacementIndex`, `FLOAT m_fLeafScalarValue` GPU rüzgarı/yaprak yerleşimi için).
    *   **Vertex Shader Kodları (`g_achSimpleVertexProgram`, `g_achLeafVertexProgram`):**
        *   Bunlar, `vs.1.1` (Vertex Shader Model 1.1) versiyonunda yazılmış HLSL assembly kodlarını içeren `static const char` dizileridir.
        *   **`g_achSimpleVertexProgram` (Dal/Frond Shader):**
            *   Vertex pozisyonunu alır.
            *   `WRAPPER_USE_GPU_WIND` aktifse: Vertex'e atanan rüzgar matris indeksini (`v9.x`) ve ağırlığını (`v9.y`) kullanarak rüzgar matrislerinden (`c[54+a0.x]`) ilgili olanı seçer, vertex pozisyonunu bu matrisle çarpar ve ağırlığa göre orijinal pozisyonla interpolasyon yaparak rüzgarlı pozisyonu hesaplar.
            *   Hesaplanan pozisyonu ağacın kendi yerel pozisyonuyla (`c[52]`) birleştirir.
            *   Son olarak, birleşik dünya-görüş-projeksiyon matrisiyle (`c[0]`) vertex'i ekran koordinatlarına dönüştürür (`oPos`).
            *   Aydınlatma: `WRAPPER_USE_STATIC_LIGHTING` aktifse, vertex rengini (`v5`) doğrudan geçirir (`oD0`). Dinamik aydınlatma aktifse, vertex normalini (`v3`) ışık yönüyle (`c[71]`) çarpar (dot product) ve materyal renkleriyle (`c[72]`-`c[75]`) birleştirerek son rengi hesaplar.
            *   `WRAPPER_USE_FOG` aktifse, vertex'in kameraya olan mesafesine göre lineer sis faktörü hesaplar (`oFog`).
            *   Doku koordinatlarını (`v7` -> `oT0`, aktifse `v8` -> `oT1` gölge için) pixel shader'a geçirir.
        *   **`g_achLeafVertexProgram` (Yaprak Shader):**
            *   Dal shader'ına benzer şekilde pozisyon, rüzgar (aktifse), ağaç pozisyonu ve projeksiyon hesaplamalarını yapar.
            *   `WRAPPER_USE_GPU_LEAF_PLACEMENT` aktifse: Vertex'e atanan yaprak yerleşim indeksini (`v9.z`, `c_nVertexShader_LeafTables` başlangıçlı adresleme için) ve skalar değerini (`v9.w`) kullanarak, shader sabitlerinde tutulan yaprak billboard tablosundan (`c[a0.x]`) ofset değerlerini okur ve yaprak quad'ının köşe noktalarını oluşturur.
            *   Aydınlatma ve sis hesaplamaları dal shader'ındakine benzerdir.
            *   Birincil doku koordinatlarını (`v7` -> `oT0`) geçirir.
    *   **Shader Yükleme Fonksiyonları (`LoadBranchShader`, `LoadLeafShader`):**
        *   Bu `static` fonksiyonlar, `LPDIRECT3DDEVICE8` (DirectX 8 cihazı) alır ve ilgili shader kodunu (`g_ach...Program`) ve vertex tanımlama bildirimini (`p...ShaderDecl`) kullanarak `D3DXAssembleShader` ile shader'ı derler.
        *   Başarılı derleme sonrası, `IDirect3DDevice8::CreateVertexShader` ile DirectX vertex shader nesnesini oluşturur ve bu nesnenin tanıtıcısını (DWORD) döndürür.
        *   Eğer ilgili GPU özelliği (`WRAPPER_USE_GPU_WIND` veya `WRAPPER_USE_GPU_LEAF_PLACEMENT`/`WRAPPER_USE_GPU_WIND`) `SpeedTreeConfig.h`'de aktif değilse, bu fonksiyonlar shader oluşturmak yerine doğrudan ilgili FVF değerini döndürürler. Bu, bu durumlarda DirectX'in sabit fonksiyonlu pipeline'ının kullanılacağını gösterir.
        *   Hata durumlarında (derleme veya oluşturma hatası), kullanıcıya bir `MessageBox` (ASCII versiyonu `MessageBoxA` olmalı) ile bilgi verirler veya `Tracef` ile log kaydı tutarlar.

*   **Kullanım Amacı:**
    *   SpeedTree ağaç bileşenleri için esnek ve yapılandırılabilir vertex formatları sağlamak.
    *   Bu formatlara uygun C++ yapıları tanımlayarak kod içinde vertex verilerinin kolayca yönetilmesini sağlamak.
    *   GPU üzerinde çalışan ve SpeedTree'nin rüzgar, yaprak yerleşimi, aydınlatma ve LOD gibi özelliklerini destekleyen vertex shader programlarını tanımlamak.
    *   Bu shader'ları çalışma zamanında derleyip DirectX'e yüklemek için yardımcı fonksiyonlar sunmak.

*   **Bağımlılıklar:**
    *   `SpeedTreeConfig.h` (Yapılandırma makroları için, FVF ve shader içeriğini etkiler).
    *   `d3d8.h`, `d3dx8math.h` (DirectX 8 başlıkları, `D3DXVECTOR3`, `D3DXAssembleShader`, `CreateVertexShader` vb. için).
    *   Standart C++ kütüphaneleri (`map`, `string`, `stdio.h` için `sprintf`).
    *   Windows API (`MessageBox`).
    *   (Metin2 için `Tracef` gibi özel loglama fonksiyonları).