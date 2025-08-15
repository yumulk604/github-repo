# SphereLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `SphereLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır.

## Dosya Bazlı Detaylandırma 

### `frustum.h` ve `frustum.cpp` (`Frustum` Sınıfı)

*   **Amaç:** `Frustum` sınıfı, bir 3D kamera veya bakış açısından görülebilen alanı (view frustum - görüş hacmi) tanımlamak ve bu alan içinde nesnelerin görünürlüğünü test etmek (culling) için kullanılır. Bu, render edilmeyecek nesneleri erken bir aşamada eleyerek performansı artırmaya yardımcı olan temel bir grafik programlama tekniğidir. Sınıf, birleşik bir projeksiyon matrisinden frustum'u oluşturan altı düzlemi hesaplayabilir ve belirli bir küresel nesnenin bu frustum'a göre konumunu (tamamen içinde, tamamen dışında veya kısmen içinde) belirleyebilir.

*   **`frustum.h` - Temel Tanımlar ve Arayüz:**
    *   **`ViewState` Enum'u:** Bir nesnenin frustum'a göre durumunu belirtmek için kullanılır:
        *   `VS_INSIDE`: Nesne tamamen frustum'un içindedir.
        *   `VS_PARTIAL`: Nesne kısmen frustum'un içinde, kısmen dışındadır (yani frustum düzlemlerinden bir veya daha fazlasını keser).
        *   `VS_OUTSIDE`: Nesne tamamen frustum'un dışındadır.
    *   **`Frustum` Sınıfı Üyeleri:**
        *   **Genel Metotlar:**
            *   `void BuildViewFrustum(D3DXMATRIX& mat)`: Verilen bir birleşik matristen (genellikle dünya-görüş-projeksiyon matrisi) frustum'u oluşturan 6 düzlemi çıkarır ve normalize eder.
            *   `void BuildViewFrustum2(D3DXMATRIX& mat, float fNear, float fFar, float fFov, float fAspect, const D3DXVECTOR3& vCamera, const D3DXVECTOR3& vLook)`: `BuildViewFrustum`'a ek olarak, verilen kamera parametrelerini (yakın/uzak kırpma düzlemleri, dikey görüş alanı (FOV), en-boy oranı, kamera pozisyonu ve bakış yönü) kullanarak frustum'u saran bir sınırlayıcı küre (`m_v3Center`, `m_fRadius`) hesaplar. Bu küre, `ViewVolumeTest` içinde daha hızlı bir ön culling testi için kullanılabilir.
            *   `ViewState ViewVolumeTest(const Vector3d& c_v3Center, const float c_fRadius) const`: Verilen merkez noktası (`c_v3Center`) ve yarıçapı (`c_fRadius`) olan bir kürenin frustum'a göre durumunu (`ViewState`) döndürür.
        *   **Özel (Private) Üye Değişkenler:**
            *   `bool m_bUsingSphere`: `BuildViewFrustum2` çağrıldığında `true` olarak ayarlanır. `ViewVolumeTest` içinde, frustum'un sınırlayıcı küresiyle hızlı bir ön testin yapılıp yapılmayacağını kontrol eder.
            *   `D3DXVECTOR3 m_v3Center`: `BuildViewFrustum2` ile hesaplanan, frustum'u saran sınırlayıcı kürenin merkezi.
            *   `float m_fRadius`: `BuildViewFrustum2` ile hesaplanan, frustum'u saran sınırlayıcı kürenin yarıçapı.
            *   `D3DXPLANE m_plane[6]`: Frustum'u oluşturan altı düzlemi (yakın, uzak, sol, sağ, üst, alt) saklayan bir dizi. Her bir `D3DXPLANE`, `Ax + By + Cz + D = 0` denklemiyle bir düzlemi temsil eder.

*   **`frustum.cpp` - Uygulama Detayları:**
    *   **`ViewVolumeTest(...) const`:**
        *   Eğer `m_bUsingSphere` true ise, ilk olarak test edilen küre ile frustum'un önceden hesaplanmış sınırlayıcı küresi (`m_v3Center`, `m_fRadius`) arasında bir küre-küre çarpışma testi yapılır. Eğer bu iki küre birbirine değmiyorsa, test edilen kürenin frustum dışında olduğu (`VS_OUTSIDE`) varsayılır ve daha fazla hesaplamaya gerek kalmaz.
        *   Eğer bu ön test geçilirse veya `m_bUsingSphere` false ise, standart düzlem tabanlı frustum culling algoritması uygulanır:
            1.  Test edilen kürenin merkezi (`c_v3Center`), frustum'un altı düzleminin her birine göre mesafesi (`D3DXPlaneDotCoord` ile) hesaplanır.
            2.  Eğer merkezin herhangi bir düzleme olan mesafesi, `-c_fRadius`'tan (negatif yarıçap) küçük veya eşitse, küre bu düzlemin tamamen "arkasında" (negatif tarafında) kalıyor demektir ve dolayısıyla frustum'un dışındadır (`VS_OUTSIDE`).
            3.  Eğer küre hiçbir düzlemin tamamen arkasında değilse (yani tüm düzlemler için mesafe > `-c_fRadius`), bir sonraki adıma geçilir.
            4.  Eğer merkezin herhangi bir düzleme olan mesafesi, `c_fRadius`'tan (pozitif yarıçap) küçük veya eşitse, küre bu düzlemi kesiyor veya "ön" (pozitif) tarafına çok yakın demektir. Bu durumda, küre kısmen içeridedir (`VS_PARTIAL`).
            5.  Eğer yukarıdaki koşulların hiçbiri sağlanmazsa (yani küre tüm düzlemlerin "ön" tarafındadır ve hiçbirini kesmiyorsa - tüm mesafeler > `c_fRadius`), küre tamamen frustum'un içindedir (`VS_INSIDE`).
    *   **`BuildViewFrustum(D3DXMATRIX& mat)`:**
        *   Verilen birleşik matrisin (genellikle `matWorld * matView * matProjection`) elemanlarını kullanarak standart bir yöntemle 6 frustum düzlemini (yakın, uzak, sol, sağ, üst, alt) çıkarır. Her bir düzlemin denklemi, matrisin belirli satır veya sütunlarının kombinasyonlarından elde edilir.
        *   Örneğin, DirectX'te tipik bir sol el koordinat sistemi için:
            *   Sol Düzlem: `mat._14 + mat._11`, `mat._24 + mat._21`, `mat._34 + mat._31`, `mat._44 + mat._41`
            *   Sağ Düzlem: `mat._14 - mat._11`, `mat._24 - mat._21`, `mat._34 - mat._31`, `mat._44 - mat._41`
            *   (Benzer şekilde üst, alt, yakın ve uzak düzlemler için de formüller vardır. Koddaki `m_plane[0]` (yakın) ve `m_plane[1]` (uzak) tanımları biraz daha farklı görünüyor, muhtemelen normalizasyon veya matris konvansiyonuyla ilgili özel bir durum olabilir.)
        *   Hesaplanan her düzlem daha sonra `D3DXPlaneNormalize` fonksiyonu ile normalize edilir. Bu, düzlemin normal vektörünün birim uzunluğa sahip olmasını sağlar, bu da `D3DXPlaneDotCoord` ile yapılan mesafe hesaplamalarının doğru olması için önemlidir.
        *   `m_bUsingSphere` bayrağı `false` oları ayarlanır.
    *   **`BuildViewFrustum2(...)`:**
        *   İlk olarak, verilen kamera parametrelerini (yakın/uzak düzlemler `fNear`/`fFar`, dikey görüş alanı `fFov`, en-boy oranı `fAspect`) kullanarak frustum'un yaklaşık boyutlarını hesaplar.
        *   Uzak düzlemdeki yarı yükseklik (`fH = (fFar - fNear) * tan(fFov * 0.5f)`) ve yarı genişlik (`fW = fH * fAspect`) bulunur.
        *   Frustum'un kamera uzayındaki merkezi (`P`) ve bu merkezden uzak düzlemdeki bir köşeye olan vektör (`Q`) kullanılarak, frustum'u saran bir sınırlayıcı kürenin yarıçapı (`m_fRadius = D3DXVec3Length(&(P-Q))`) hesaplanır.
        *   Bu kürenin dünya uzayındaki merkezi (`m_v3Center`), kamera pozisyonuna bakış yönü boyunca bir miktar ötelenerek bulunur.
        *   Son olarak, temel `BuildViewFrustum(mat)` metodu çağrılarak frustum düzlemleri de oluşturulur.
        *   `m_bUsingSphere` bayrağı `true` olarak ayarlanır.

*   **Kullanım Amacı:**
    *   Bir kameranın görüş alanını matematiksel olarak tanımlamak.
    *   Sahnedeki nesnelerin (genellikle sınırlayıcı küreleriyle temsil edilen) bu görüş alanı içinde olup olmadığını verimli bir şekilde test etmek.
    *   Görünmeyen nesnelerin render edilmesini engelleyerek (frustum culling) oyun performansını artırmak.

*   **Bağımlılıklar:**
    *   `Stdafx.h` (SphereLib için)
    *   `vector.h` (SphereLib içindeki `Vector3d` sınıfı için)
    *   DirectX 8 (`D3DXMATRIX`, `D3DXPLANE`, `D3DXVECTOR3`, `D3DXPlaneDotCoord`, `D3DXPlaneNormalize`, `D3DXVec3LengthSq`, `D3DXVec3Length`, `tan`). 

### `sphere.h` ve `sphere.cpp` (`Sphere` ve `SphereInterface` Sınıfları)

*   **Amaç:** Bu dosyalar, 3D uzayda bir küreyi (`Sphere` sınıfı) ve bir grup 3D noktasına erişim için bir arayüzü (`SphereInterface` sınıfı) tanımlar. `Sphere` sınıfı, kürenin geometrik özelliklerini (merkez, yarıçap) saklamanın yanı sıra, bir nokta kümesini en iyi şekilde saran bir sınırlayıcı küre hesaplama (Jack Ritter algoritması), ışın-küre kesişim testleri ve bir noktanın küre içinde olup olmadığını kontrol etme gibi temel geometrik işlemleri sağlar.

*   **`SphereInterface` Sınıfı (`sphere.h`):**
    *   **Rolü:** Bu, soyut (abstract) bir temel sınıftır ve bir 3D nokta (vertex) koleksiyonuna erişim için standart bir arayüz sunar. Temel amacı, `Sphere::Compute` metodunun farklı türdeki nokta kümelerinden (örneğin, bir mesh veya basit bir nokta listesi) veri alabilmesini sağlamaktır.
    *   **Metotlar:**
        *   `SphereInterface()`: Varsayılan kurucu.
        *   `virtual ~SphereInterface()`: Sanal yıkıcı, bu arayüzden türetilmiş sınıfların doğru şekilde silinebilmesi için önemlidir.
        *   `virtual int GetVertexCount(void) const = 0`: Nokta kümesindeki toplam vertex (nokta) sayısını döndürmesi gereken saf sanal (pure virtual) bir metottur. Türetilmiş sınıflar bu metodu implemente etmelidir.
        *   `virtual bool GetVertex(int i, Vector3d& vect) const = 0`: Belirtilen indeksteki (`i`) vertex'in 3D koordinatlarını (`Vector3d` referansıyla) döndürmesi gereken saf sanal bir metottur. Başarılı olursa `true` döndürmelidir. Türetilmiş sınıflar bu metodu implemente etmelidir.

*   **`Sphere` Sınıfı (`sphere.h` ve `sphere.cpp`):**
    *   **Rolü:** 3D uzayda bir küreyi temsil eder. Merkez noktasını ve yarıçapını tutar. Çeşitli geometrik sorgulamalar ve hesaplamalar için metotlar sunar.
    *   **Üye Değişkenler:**
        *   `protected Vector3d mCenter`: Kürenin merkez noktasını saklar (`Vector3d` sınıfından bir nesne, muhtemelen `vector.h` içinde tanımlıdır).
        *   `private float mRadius`: Kürenin yarıçapını saklar.
        *   `private float mRadius2`: Kürenin yarıçapının karesini (`mRadius * mRadius`) saklar. Bazı mesafe karşılaştırmalarında karekök alma işleminden kaçınarak performansı artırmak için kullanılır.
    *   **Ana Metotlar:**
        *   **Yapıcılar:**
            *   `Sphere()`: Merkezi orijinde (0,0,0) ve yarıçapı 0 olan bir küre oluşturur.
            *   `Sphere(const Vector3d& center, float radius)`: Belirtilen merkez ve yarıçap ile bir küre oluşturur.
        *   **Ayar Metotları:**
            *   `void Set(const Vector3d& center, float radius)`: Kürenin merkezini ve yarıçapını ayarlar, ayrıca `mRadius2`'yi günceller. `#ifdef __STATIC_RANGE__` tanımlıysa, merkez koordinatlarının belirli bir aralıkta olup olmadığını `assert` ile kontrol eder.
            *   `void SetRadius(float radius)`: Sadece kürenin yarıçapını ayarlar ve `mRadius2`'yi günceller.
        *   **Erişim Metotları:**
            *   `float GetRadius(void) const`: Yarıçapı döndürür.
            *   `float GetRadius2(void) const`: Yarıçapın karesini döndürür.
            *   `const Vector3d& GetCenter(void) const`: Merkez noktasını sabit referans olarak döndürür.
        *   **Sınırlayıcı Küre Hesaplama:**
            *   `void Compute(const SphereInterface& source)`: Verilen `SphereInterface` uyumlu bir kaynaktan aldığı noktaları en iyi şekilde saran bir sınırlayıcı küre hesaplar. Bu metot, Jack Ritter tarafından "Graphics Gems" (1990) kitabında tanıtılan verimli bir algoritmayı kullanır:
                1.  **İlk Geçiş:** Nokta kümesi içinde X, Y ve Z eksenleri boyunca minimum ve maksimum koordinatlara sahip olan 6 nokta bulunur.
                2.  Bu 6 nokta arasındaki en uzak iki nokta (en büyük "span"e sahip olanlar) bulunur. Bu iki nokta, başlangıç küresinin çapını oluşturur.
                3.  Başlangıç küresinin merkezi bu iki noktanın ortası, yarıçapı ise aralarındaki mesafenin yarısı olarak hesaplanır.
                4.  **İkinci Geçiş:** Tüm noktalar tekrar taranır. Eğer bir nokta mevcut kürenin dışındaysa, küre bu noktayı da içerecek şekilde genişletilir. Yeni merkez ve yarıçap, eski küre ve dışarıdaki nokta arasında bir interpolasyonla yeniden hesaplanır. Bu işlem tüm noktalar için tekrarlanır ve sonuçta tüm noktaları içeren sıkı bir sınırlayıcı küre elde edilir.
        *   **Işın-Küre Kesişim Testleri:**
            *   `bool RayIntersection(const Vector3d& rayOrigin, const Vector3d& rayDirection, Vector3d* intersect)`: Verilen bir ışının (başlangıç noktası `rayOrigin`, yönü `rayDirection`) küreyle kesişip kesişmediğini test eder. "Graphics Gems" (p.388) kitabındaki standart algoritmayı kullanır ve ışın başlangıcı küre içindeyse oluşan bir hatayı düzeltmek için özel bir kontrol içerir (eğer ışın başlangıcı içerideyse, kesişim yönünü ters çevirir). Kesişim varsa `true` döner ve (eğer `intersect` NULL değilse) en yakın kesişim noktasını `intersect`'e yazar.
            *   `bool RayIntersection(const Vector3d& rayOrigin, const Vector3d& V, float distance, Vector3d* intersect)`: Önceki metoda ek olarak, bulunan kesişim noktasının `rayOrigin`'e olan mesafesinin verilen `distance` (mesafe) sınırları içinde olup olmadığını kontrol eder.
            *   `bool RayIntersectionInFront(const Vector3d& rayOrigin, const Vector3d& rayDirection, Vector3d* intersect)`: `RayIntersection` gibi çalışır, ancak sadece ışının pozitif yönünde (önünde) olan kesişimleri geçerli sayar (kesişim vektörü ile ışın yönünün dot product'ı pozitif olmalıdır).
        *   **Nokta İçindelik Testleri:**
            *   `bool InSphereXY(const Vector3d& pos, float distance) const`: Verilen bir `pos` noktasının, kürenin XY düzlemindeki 2D izdüşümünün, belirtilen `distance` kadar genişletilmiş alanının içinde olup olmadığını kontrol eder (Z koordinatı dikkate alınmaz).
            *   `bool InSphere(const Vector3d& pos, float distance) const`: Verilen bir `pos` noktasının, yarıçapı `distance` kadar artırılmış olan kürenin 3D'de içinde olup olmadığını kontrol eder.
        *   `void Report(void)`: Muhtemelen hata ayıklama amacıyla küre bilgilerini yazdırmak için tasarlanmış, ancak şu anki implementasyonu boştur.

*   **Kullanım Amacı:**
    *   3D nesneler için basit ve etkili bir sınırlayıcı hacim sağlamak.
    *   Işın seçimi (ray picking), çarpışma tespiti ve görünürlük testleri (culling) gibi çeşitli geometrik algoritmalarda kullanılmak.
    *   Bir nokta kümesini (örneğin bir 3D modelin vertexleri) saran minimum sınırlayıcı küreyi hesaplamak.

*   **Bağımlılıklar:**
    *   `Stdafx.h` (SphereLib için)
    *   `vector.h` (`Vector3d` sınıfı ve onun metotları için, örneğin `Dot`, `Length2`, `DistanceSq`).
    *   Standart C++ kütüphaneleri (`cmath` için `sqrtf`, `cassert` için `assert`).
    *   (Muhtemelen `stdio.h`, `string.h`, `stdlib.h` gibi dosyalar `sphere.cpp` içinde dolaylı olarak veya `Stdafx.h` üzerinden dahil ediliyor). 

### `pool.h` (`Pool<Type>` Template Sınıfı)

*   **Amaç:** `Pool<Type>` template sınıfı, belirli bir `Type` türündeki nesneler için yüksek performanslı, sabit boyutlu bir bellek havuzu yöneticisidir. Amacı, sıkça yapılan küçük nesne tahsisleri ve serbest bırakmaları sırasında standart `new` ve `delete` operatörlerinin neden olabileceği performans düşüşlerini ve bellek parçalanmasını (fragmentation) en aza indirmektir. Havuz, başlangıçta belirtilen sayıda nesne için tek bir bellek bloğu ayırır ve bu nesneleri "kullanılanlar" ve "boştakiler" olmak üzere iki ayrı çift yönlü bağlı liste üzerinden yönetir. Bu sayede nesne alma (allocate) ve iade etme (deallocate) işlemleri çok hızlı bir şekilde gerçekleştirilebilir.

*   **Template Parametresi:**
    *   `class Type`: Havuzda saklanacak nesnelerin türünü belirtir. Bu `Type` sınıfının, havuz içindeki çift yönlü bağlı liste altyapısını desteklemek için aşağıdaki gibi metotlara sahip olması beklenir (veya bu metotları miras aldığı bir temel sınıftan gelmelidir):
        *   `void SetNext(Type* pNext)`: Sonraki elemanı ayarlar.
        *   `Type* GetNext() const`: Sonraki elemanı döndürür.
        *   `void SetPrevious(Type* pPrev)`: Önceki elemanı ayarlar.
        *   `Type* GetPrevious() const`: Önceki elemanı döndürür.

*   **Ana Metotlar ve İşlevleri:**
    *   **Yapılandırma ve Temizleme:**
        *   `Pool()`: Varsayılan kurucu. Havuzu boş bir durumda başlatır.
        *   `~Pool()`: Yıkıcı. `Release()` çağırarak ayrılan tüm belleği serbest bırakır.
        *   `void Release()`: Havuz tarafından tutulan ana bellek bloğunu (`mData`) siler ve tüm işaretçileri/sayaçları sıfırlayarak havuzu ilk durumuna döndürür.
        *   `void Set(int maxitems)`: Havuzu belirtilen `maxitems` kapasitesiyle başlatır. Tek bir büyük bellek bloğu (`Type* mData = new Type[mMaxItems]`) ayırır ve bu bloktaki tüm elemanları başlangıçta "boş" (free) listesine çift yönlü bağlı olarak ekler.
    *   **Nesne Tahsisi (Allocation):**
        *   `Type* GetFreeNoLink()`: Boş listeden bir nesne alır. Bu nesne boş listeden çıkarılır ancak "kullanılanlar" listesine otomatik olarak bağlanmaz. Döndürülen nesnenin `Next` ve `Previous` işaretçileri `NULL` yapılır. Kullanım senaryosuna göre manuel olarak listeye eklenebilir. Eğer boş nesne yoksa `NULL` döndürür.
        *   `Type* GetFreeLink()`: Boş listeden bir nesne alır ve bu nesneyi otomatik olarak "kullanılanlar" listesinin başına (`mHead`) ekler. Eğer boş nesne yoksa `NULL` döndürür.
    *   **Nesne Serbest Bırakma (Deallocation):**
        *   `void Release(Type* t)`: Belirtilen `t` nesnesini (daha önce tahsis edilmiş olmalı) "kullanılanlar" listesinden çıkarır ve "boş" listesinin başına ekler. Bu işlem, ilgili bağlı liste işaretçilerinin güncellenmesini içerir.
    *   **İterasyon (Kullanılanlar Listesi Üzerinde):**
        *   `int Begin()`: Kullanılanlar listesi (`mHead`'den başlayan) üzerinde dolaşmaya başlamak için çağrılır. Dahili `mCurrent` işaretçisini listenin başına ayarlar ve o andaki kullanılan nesne sayısını (`mUsedCount`) döndürür. Genellikle bir `for` veya `while` döngüsüyle birlikte kullanılır.
        *   `Type* GetNext(bool& looped)`: Kullanılanlar listesindeki bir sonraki elemanı döndürür. `mCurrent` işaretçisini ilerletir. `looped` parametresi, iterasyonun listenin sonuna gelip (eğer `mHead`'den başlanmışsa ve `mCurrent` `NULL` olmuşsa) tekrar başına dönüp dönmediğini belirtmek için kullanılır (bu durumda `mCurrent`, `mHead`'e ayarlanır ve `looped` `true` olur).
        *   `Type* GetNext()`: `GetNext(bool& looped)` metodunun `looped` bayrağını döndürmeyen basit bir versiyonudur.
    *   **Durum Sorgulama:**
        *   `bool IsEmpty(void) const`: "Kullanılanlar" listesinin boş olup olmadığını kontrol eder (`mHead == NULL`).
        *   `int GetUsedCount(void) const`: Havuzda o anda "kullanımda" olan nesne sayısını döndürür.
        *   `int GetFreeCount(void) const`: Havuzda o anda "boşta" olan nesne sayısını döndürür.
    *   **Manuel Liste Yönetimi (Nadiren doğrudan kullanılır):**
        *   `void AddAfter(Type* e, Type* item)`: `item` nesnesini, kullanılanlar listesindeki `e` nesnesinden hemen sonraya ekler.
        *   `void AddBefore(Type* e, Type* item)`: `item` nesnesini, kullanılanlar listesindeki `e` nesnesinden hemen önceye ekler.

*   **Özel (Private) Üye Değişkenler:**
    *   `int mMaxItems`: Havuzun toplam kapasitesi (nesne sayısı).
    *   `Type* mCurrent`: Kullanılanlar listesi üzerinde iterasyon yapılırken mevcut konumu gösteren işaretçi.
    *   `Type* mData`: Havuz için tahsis edilen ana bellek bloğunun başlangıç adresi.
    *   `Type* mHead`: "Kullanılan" nesnelerin oluşturduğu çift yönlü bağlı listenin başı.
    *   `Type* mFree`: "Boş" (kullanıma hazır) nesnelerin oluşturduğu çift yönlü bağlı listenin başı.
    *   `int mUsedCount`: Kullanımdaki nesne sayısı.
    *   `int mFreeCount`: Boştaki nesne sayısı.

*   **Kullanım Amacı:**
    *   Sık sık oluşturulup silinen çok sayıda küçük nesnenin bellek yönetimini optimize etmek.
    *   Dinamik bellek tahsisinin (heap allocation) getirdiği performans yükünü azaltmak.
    *   Bellek parçalanmasının önüne geçmek.
    *   Oyunlar, parçacık sistemleri, mermi yönetimi gibi senaryolarda yaygın olarak kullanılır.

*   **Bağımlılıklar:**
    *   Template parametresi olan `Type` sınıfının belirli bir arayüze (Next/Previous işaretçileri için Get/Set metotları) sahip olması.
    *   Temel C++ (işaretçiler, `new[]`, `delete[]`).

*   **Örnek Kullanım:**
    *   `Pool<MyClass> myPool;`: `MyClass` türünden nesneler için bir havuz oluşturur.
    *   `MyClass* obj = myPool.GetFreeNoLink();`: Havuzdan bir nesne alır.
    *   `myPool.Release(obj);`: Nesneyi havuza iade eder.
    *   `for (int i = myPool.Begin(); i != -1; i = myPool.GetNext())`: Havuzdaki tüm nesneleri dolaşır.
    *   `if (myPool.IsEmpty())`: Havuzun boş olup olmadığını kontrol eder. 

### `SpherePackFactory` Sınıfı (`spherepack.h` ve `spherepack.cpp`)

*   **Amaç:** `SpherePackFactory` sınıfı, tüm Sphere Tree (Küre Ağacı) yapısının oluşturulması, yönetilmesi ve sorgulanmasından sorumlu merkezi bir fabrikadır. `SpherePack` nesnelerini (bellek havuzundan `Pool<SpherePack>`) oluşturur, bu nesneleri ağaca (aslında iki ayrı ağaca: `mRoot` ve `mLeaf`) entegre eder, ağacın dengelenmesini (yeniden hesaplama/recompute) ve güncellenmesini yönetir. Ayrıca, frustum culling, ışın izleme ve menzil testleri gibi sorgulamaların tüm ağaç üzerinde başlatılmasını sağlar. `SpherePackCallback` sınıfından türediği için varsayılan callback davranışlarını da içerir, ancak genellikle kullanıcı tarafından sağlanan özel bir callback nesnesiyle çalışır.
*   **Kalıtım:** `public SpherePackCallback`.
*   **Önemli Üye Değişkenler:**
    *   `mRoot` (SpherePack\*): "Root Tree" olarak adlandırılan ana küre ağacının kök düğümüne işaret eder. Bu ağaç genellikle daha büyük, daha statik veya daha az sıklıkla güncellenen nesneleri (veya "Leaf Tree"deki süperkürelerin temsillerini) içerir.
    *   `mLeaf` (SpherePack\*): "Leaf Tree" olarak adlandırılan ikincil küre ağacının kök düğümüne işaret eder. Bu ağaç genellikle daha küçük, daha dinamik veya sık güncellenen yaprak seviyesindeki nesneleri barındırır.
    *   `mCallback` (SpherePackCallback\*): Frustum, ışın izleme ve menzil testleri sırasında kullanılacak olan asıl callback işleyici nesnesine işaret eder. Fabrika, `SpherePackCallback`'ten türediği için kendisi de bir callback sağlayıcıdır, ancak genellikle dışarıdan bir `mCallback` atanır.
    *   `mSpheres` (Pool<SpherePack>): Tüm `SpherePack` nesnelerinin tahsis edildiği ve yönetildiği bellek havuzu.
    *   `mIntegrate` (SpherePackFifo\*): Ağaca yeni eklenen veya pozisyonu/boyutu önemli ölçüde değişen ve bu nedenle ağaç hiyerarşisine yeniden entegre edilmesi gereken `SpherePack` nesnelerini tutan bir FIFO kuyruğu.
    *   `mRecompute` (SpherePackFifo\*): Çocuklarının durumu değiştiği için sınırlayıcı küresinin yeniden hesaplanması (dengelenmesi/rebalance) gereken `SpherePack` (genellikle süperküreler) nesnelerini tutan bir FIFO kuyruğu.
    *   `mMaxRootSize` (float): "Root Tree" içindeki bir süperkürenin ulaşabileceği maksimum yarıçap.
    *   `mMaxLeafSize` (float): "Leaf Tree" içindeki bir süperkürenin ulaşabileceği maksimum yarıçap.
    *   `mSuperSphereGravy` (float): Bir süperkürenin yarıçapı yeniden hesaplandığında, çocuklarını tam olarak sarması için eklenen ekstra bir pay (boşluk). Bu, küçük hareketlerde sürekli yeniden hesaplamayı önlemeye yardımcı olur.
    *   `#if DEMO` ile çevrelenmiş `mColorCount` ve `mColors` dizisi: Demo uygulamasında yeni oluşturulan süperkürelere farklı renkler atamak için kullanılır.
*   **Ana Metotlar ve İşlevleri:**
    *   **Başlatma ve Yıkım:**
        *   `SpherePackFactory(int maxspheres, float rootsize, float leafsize, float gravy)`: Kurucu. Belirtilen parametrelerle fabrikayı başlatır: `mSpheres` bellek havuzunu `maxspheres` (çarpı 4) kapasitesiyle, `mIntegrate` ve `mRecompute` FIFO kuyruklarını ve `mRoot` ile `mLeaf` kök düğümlerini (`SPF_SUPERSPHERE | SPF_ROOTNODE` bayraklarıyla ve ilgili ağaç türü bayrağıyla) oluşturur.
        *   `virtual ~SpherePackFactory()`: Yıkıcı. `mIntegrate` ve `mRecompute` FIFO kuyruklarını siler (bellek havuzu `mSpheres` kendi yıkıcısında temizlenir).
    *   **Ana İşlem Döngüsü:**
        *   `void Process()`: Genellikle her frame çağrılan ana güncelleme metodudur. İki ana adımda çalışır:
            1.  **Yeniden Hesaplama (Recompute):** `mRecompute` kuyruğundaki tüm `SpherePack`\'leri alır. Her biri için `pack->Recompute(mSuperSphereGravy)` çağırır. Eğer `Recompute` metodu `true` döndürürse (yani süperküre boşalmışsa), `Remove(pack)` ile o süperküreyi siler.
            2.  **Entegrasyon (Integrate):** `mIntegrate` kuyruğundaki tüm `SpherePack`\'leri alır. Her biri için, ait olduğu ağaç türüne (`SPF_ROOT_TREE` veya `SPF_LEAF_TREE`) göre `Integrate(pack, mRoot, mMaxRootSize)` veya `Integrate(pack, mLeaf, mMaxLeafSize)` metodunu çağırır.
    *   **Nesne Ekleme ve Yönetimi:**
        *   `SpherePack* AddSphere_(const Vector3d& pos, float radius, void* userdata, bool isSphere, int flags)`: Bellek havuzundan (`mSpheres.GetFreeLink()`) yeni bir `SpherePack` nesnesi alır, `Init()` ile başlatır ve bayraklarına göre (`SPF_ROOT_TREE` veya `SPF_LEAF_TREE`) `AddIntegrate()` metodunu çağırarak entegrasyon kuyruğuna ekler.
        *   `void AddIntegrate(SpherePack* pack)`: Verilen `pack`\'i önce ait olduğu kök düğümün (`mRoot` veya `mLeaf`) çocuğu olarak ekler, sonra `SPF_INTEGRATE` bayrağını kurar ve `mIntegrate` FIFO\'suna `Push` eder. `pack`\'in FIFO konumunu (`mFifo2`) saklar.
        *   `void AddRecompute(SpherePack* recompute)`: Eğer `recompute` zaten `SPF_RECOMPUTE` bayrağına sahip değilse ve çocukları varsa (`GetChildCount() > 0`), `SPF_RECOMPUTE` bayrağını kurar, `mRecompute` FIFO\'suna `Push` eder ve FIFO konumunu (`mFifo1`) saklar. Eğer çocuğu yoksa, doğrudan `Remove(recompute)` ile siler.
        *   `void Integrate(SpherePack* pack, SpherePack* supersphere_root, float node_size)`: Bu karmaşık metot, verilen `pack`\'i `supersphere_root` (ya `mRoot` ya da `mLeaf`) altındaki ağaca hiyerarşik olarak yerleştirir:
            1.  `supersphere_root`\'un çocukları (yani mevcut süperküreler) arasında `pack`\'in merkezine en yakın olanı arar. İki tür \"en yakın\" tanımı kullanılır:
                *   `nearest1`: `pack`\'i tamamen içine alan mevcut en yakın süperküre.
                *   `nearest2`: `pack`\'i içine almak için en az miktarda genişlemesi gereken mevcut en yakın süperküre.
            2.  Eğer `nearest1` bulunursa (yani `pack` bir süperkürenin içine tam sığıyorsa), `pack` bu `nearest1`\'in çocuğu yapılır, bağlanma mesafesi hesaplanır ve `nearest1` yeniden hesaplanır (`Recompute`). Eğer `nearest1` bir \"Leaf Tree\" süperküresiyse, `Root Tree`\'deki karşılığı olan linkin pozisyonu/yarıçapı da güncellenir.
            3.  Eğer `nearest1` yok ama `nearest2` varsa ve `nearest2`\'yi `pack`\'i içerecek şekilde genişletmek `node_size` (yani `mMaxRootSize` veya `mMaxLeafSize`) sınırını aşmıyorsa, `nearest2` genişletilir, `pack` onun çocuğu yapılır ve benzer güncellemeler yapılır.
            4.  Eğer yukarıdaki iki durum da geçerli değilse (yani `pack` mevcut hiçbir süperküreye uygun şekilde eklenemiyorsa), `pack` etrafında yeni bir süperküre (`parent`) oluşturulur. Bu yeni `parent`, `supersphere_root`\'un çocuğu olur ve `pack` de bu yeni `parent`\'ın çocuğu olur. Eğer bu işlem \"Leaf Tree\"de gerçekleşiyorsa (`parent->HasSpherePackFlag(SPF_LEAF_TREE)`), bu yeni `parent` süperküresi için \"Root Tree\"de bir karşılık/link (`SpherePack* link = AddSphere_(...)`) oluşturulur ve `parent->SetUserData(link, true)` ile birbirlerine bağlanırlar.
            5.  Son olarak, `pack`\'in `SPF_INTEGRATE` bayrağı temizlenir.
        *   `void Remove(SpherePack* pack)`: Verilen `pack`\'i ağaçtan çıkarır. Eğer bu bir \"Leaf Tree\" süperküresi ve Root Tree\'de bir linki varsa, o linki de `Remove` eder. `pack->Unlink()` çağrılır ve nesne `mSpheres.Release(pack)` ile bellek havuzuna iade edilir. Kök düğümler (`SPF_ROOTNODE`) asla silinemez.
    *   **Sorgulama ve Test Metotları:**
        *   `void FrustumTest(const Frustum& f, SpherePackCallback* callback)`: Verilen `f` frustum\'u ile `mRoot` ağacı üzerinde görünürlük testi başlatır. `mCallback` üyesini (veya parametre olarak gelen `callback`\'i) kullanarak `mRoot->VisibilityTest`\'i çağırır.
        *   `void RayTrace(const Vector3d& p1, const Vector3d& p2, SpherePackCallback* callback)`: `p1`\'den `p2`\'ye bir ışınla `mRoot` ağacı üzerinde kesişim testi başlatır.
        *   `void RangeTest(const Vector3d& center, float radius, SpherePackCallback* callback)`: Belirtilen merkez ve yarıçapla `mRoot` ağacı üzerinde menzil testi başlatır.
        *   `void PointTest2d(const Vector3d& center, SpherePackCallback* callback)`: Belirtilen 2D nokta ile `mRoot` ağacı üzerinde test başlatır.
    *   **Callback Implementasyonları (SpherePackCallback'ten Gelen):**
        *   `virtual void VisibilityCallback(...)`, `virtual void RayTraceCallback(...)`, `virtual void RangeTestCallback(...)`, `virtual void PointTest2dCallback(...)`: Bu metotlar, `SpherePackFactory`\'nin kendisi bir callback işleyici olarak kullanıldığında çağrılır. Genellikle, test edilen `sphere`\'in `mUserData`\'sında saklanan linke (`SpherePack* link = (SpherePack*)sphere->GetUserData()`) bakarlar. Eğer bir link varsa ve bu link bir `SpherePack` ise (`sphere->IS_SPHERE` kontrolüyle), aynı testi bu linklenmiş `SpherePack` üzerinde (asıl `mCallback` kullanılarak) devam ettirirler. Bu, Root Tree ve Leaf Tree arasındaki sorgu geçişlerini sağlar.
        *   `void Reset()`: `mRoot` ve `mLeaf` ağaçlarındaki tüm düğümlerin görünürlük bayraklarını sıfırlar.

*   **Kullanım Amacı:**
    *   Kompleks bir Sphere Tree (iki seviyeli: Root ve Leaf) yapısını oluşturmak, dinamik olarak güncellemek ve yönetmek.
    *   Çok sayıda 3D nesnenin (kürelerle temsil edilen) hiyerarşik bir düzende verimli bir şekilde saklanmasını ve sorgulanmasını sağlamak.
    *   Frustum culling, ışın izleme, menzil ve nokta testleri gibi uzamsal sorguları optimize etmek.
    *   Bellek yönetimini `Pool` sınıfı üzerinden verimli bir şekilde yapmak.

*   **Bağımlılıklar:**
    *   `spherepack.h` (içindeki `SpherePack`, `SpherePackCallback`, `SpherePackFifo` tanımları).
    *   `Stdafx.h` (SphereLib için).
    *   `vector.h` (`Vector3d` için).
    *   `pool.h` (`Pool<SpherePack>` için).
    *   `sphere.h` (`Sphere` temel sınıfı için).
    *   `frustum.h` (`Frustum` sınıfı için).
    *   `assert.h`.
    *   (Metin2 için `NANOBEGIN`/`NANOEND` makroları, `EterBase/Debug.h` için `TraceError`). 

### `StdAfx.h` ve `StdAfx.cpp` Dosyaları

**Amaç:**

`StdAfx.h` ve `StdAfx.cpp` dosyaları, `SphereLib` kütüphanesi için ön derlenmiş başlık (precompiled header) mekanizmasını oluşturur. Bu mekanizma, sık kullanılan ve nadiren değişen başlık dosyalarının bir kez derlenip sonraki derlemelerde tekrar kullanılmasını sağlayarak projenin toplam derleme süresini önemli ölçüde azaltır. `StdAfx.cpp` dosyası genellikle sadece `StdAfx.h` dosyasını içerir ve ön derlenmiş başlığın oluşturulması için birincil kaynak görevi görür.

**`StdAfx.h` İçeriği ve Dahil Edilen Başlıklar:**

`StdAfx.h` dosyası, `SphereLib` içindeki diğer kaynak dosyaların ihtiyaç duyabileceği temel başlıkları ve tanımlamaları içerir:

*   `#pragma once`: Bu direktif, başlık dosyasının yalnızca bir kez derleme birimine dahil edilmesini sağlar.
*   `#define WIN32_LEAN_AND_MEAN`: Windows başlık dosyalarından nadiren kullanılan API'leri hariç tutarak derleme süresini kısaltır.
*   `#include <d3d8.h>` ve `#include <d3dx8.h>`: DirectX 8 için temel ve yardımcı kütüphane başlıkları. `SphereLib` muhtemelen grafiksel gösterimler veya DirectX ile entegrasyon için bu başlıklara ihtiyaç duyar.
*   Standart C/C++ Kütüphaneleri:
    *   `<stdio.h>`: Standart giriş/çıkış işlemleri.
    *   `<stdlib.h>`: Genel yardımcı fonksiyonlar (bellek yönetimi, dönüşümler vb.).
    *   `<string.h>`: Karakter dizisi (string) işlemleri.
    *   `<assert.h>`: Hata ayıklama için `assert` makrosu.
    *   `<math.h>`: Matematiksel fonksiyonlar.
*   Proje İçi Başlıklar:
    *   `../EterBase/StdAfx.h`: Muhtemelen `EterBase` kütüphanesindeki temel tanımlamaları ve ön derlenmiş başlığı içerir. Bu, kütüphaneler arası bağımlılıkları ve ortak tanımlamaları gösterir.
    *   `../UserInterface/Locale_inc.h`: Yerelleştirme (localization) ile ilgili tanımlamaları veya ayarları içeren bir başlık dosyası. Bu, `SphereLib`'in metin veya kaynaklarının yerelleştirilmesine yönelik bir bağlantıya sahip olabileceğini düşündürür, ancak `SphereLib`'in temel işlevi göz önüne alındığında bu dolaylı bir bağımlılık olabilir.

**`StdAfx.cpp` İçeriği:**

Bu dosya tipik olarak çok basittir ve yalnızca `StdAfx.h` başlık dosyasını içerir:

```cpp
// stdafx.cpp : source file that includes just the standard includes
// SphereLib.pch will be the pre-compiled header
// stdafx.obj will contain the pre-compiled type information

#include "stdafx.h"
```

Bu yapı, derleyicinin `SphereLib.pch` (precompiled header) dosyasını ve `stdafx.obj` nesne dosyasını oluşturmasını sağlar.

**Önemli Notlar:**

*   `SPHERELIB_STRICT` ve `<crtdbg.h>` gibi yorum satırı haline getirilmiş kısımlar, geçmişte daha katı hata ayıklama veya bellek sızıntısı tespiti için kullanılmış olabilecek yapılandırmalara işaret eder.

### `vector.h` Dosyası ve `Vector3d` Sınıfı

**Amaç:**

`vector.h` dosyası, 3D uzayda bir vektörü veya noktayı temsil etmek için kullanılan `Vector3d` adında bir sınıf tanımlar. Bu sınıf, DirectX'in `D3DXVECTOR3` yapısından türetilmiştir ve temel vektör işlemleri için kullanışlı metotlar ve operatörler sağlar. Oyun geliştirme ve 3D grafiklerde vektörler, pozisyonları, yönleri, hızları ve daha birçok fiziksel veya geometrik özelliği ifade etmek için temel bir araçtır.

**`Vector3d` Sınıfı:**

`Vector3d` sınıfı, `D3DXVECTOR3`'ten (dolayısıyla `x`, `y`, `z` float üyelerine sahiptir) kalıtım alır ve aşağıdaki gibi temel işlevsellikler sunar:

*   **Yapıcı Metotlar (Constructors):**
    *   `Vector3d()`: Boş yapıcı (varsayılan olarak başlatılmaz).
    *   `Vector3d(const Vector3d& a)`: Kopyalayıcı yapıcı.
    *   `Vector3d(float a, float b, float c)`: `x`, `y`, `z` bileşenleriyle başlatan yapıcı.

*   **Operatörler:**
    *   Karşılaştırma: `==`, `!=`
    *   Atama: `=`
    *   Aritmetik: `+` (toplama), `-` (çıkarma), `*` (skalerle çarpma), `/` (skalere bölme)
    *   Birikimli Aritmetik: `+=`, `-=`, `*=` (skalerle)
    *   Negasyon: `-` (unary)
    *   Dizi Erişimi: `[]` (indeks ile `x`, `y`, `z` bileşenlerine erişim)

*   **Erişimci (Accessor) ve Ayarlayıcı (Setter) Metotlar:**
    *   `GetX()`, `GetY()`, `GetZ()`
    *   `SetX(float t)`, `SetY(float t)`, `SetZ(float t)`
    *   `Set(float a, float b, float c)`: Tüm bileşenleri ayarlar.
    *   `Zero()`: Vektörü sıfır vektörüne ayarlar (`(0,0,0)`).

*   **Vektör İşlemleri:**
    *   `negative()`: Vektörün negatifini döndürür.
    *   `Magnitude()` / `Length()`: Vektörün büyüklüğünü (uzunluğunu) hesaplar: `sqrt(x*x + y*y + z*z)`.
    *   `Length2()`: Vektörün karesel uzunluğunu hesaplar: `x*x + y*y + z*z` (karekök almadan, performans için faydalıdır).
    *   `Normalize()`: Vektörü birim vektöre dönüştürür (uzunluğunu 1 yapar) ve orijinal uzunluğunu döndürür.
    *   `Dot(const Vector3d& a)`: İki vektörün nokta çarpımını hesaplar.
    *   `Cross(const Vector3d& a, const Vector3d& b)`: İki vektörün (`a` ve `b`) vektörel çarpımını hesaplar ve sonucu bu vektöre atar.
    *   `Lerp(const Vector3d& from, const Vector3d& to, float slerp)`: İki vektör arasında doğrusal interpolasyon yapar. `*this = from + slerp * (to - from)`.

*   **Mesafe Hesaplamaları:**
    *   `Distance(const Vector3d& a)`: Bu vektör (nokta) ile başka bir `a` vektörü (nokta) arasındaki Öklid mesafesini hesaplar.
    *   `DistanceSq(const Vector3d& a)`: İki nokta arasındaki karesel mesafeyi hesaplar (performans için karekök almaz).
    *   `Distance2d(const Vector3d& a)`: İki nokta arasındaki 2D (XY düzleminde) mesafeyi hesaplar.
    *   `DistanceSq2d(const Vector3d& a)`: İki nokta arasındaki 2D (XY düzleminde) karesel mesafeyi hesaplar.
    *   `DistanceXY(const Vector3d& a)`: `DistanceSq2d` ile aynı işlevi görür, karesel 2D mesafeyi döndürür. (Not: İsimlendirme `DistanceSqXY` olsaydı daha tutarlı olabilirdi.)

*   **Diğer Metotlar:**
    *   `IsInStaticRange() const;`: Bu metodun implementasyonu `vector.h` dosyasında verilmemiş, muhtemelen `vector.cpp` (varsa) veya başka bir yerde tanımlanmıştır. Genellikle bir vektörün belirli bir aralıkta olup olmadığını kontrol eder.

**`Vector3dVector` Typedef'i:**

Dosyanın sonunda, `std::vector<Vector3d>` için bir `typedef` tanımlanmıştır:
```cpp
typedef std::vector< Vector3d > Vector3dVector;
```
Bu, `Vector3d` nesnelerinden oluşan bir `std::vector`'ü daha kısa bir isimle (`Vector3dVector`) kullanmayı sağlar.

**Global Operatör:**

Ayrıca, bir skalerin bir `Vector3d` ile soldan çarpılabilmesi için global bir `operator*` tanımlanmıştır:
```cpp
inline Vector3d operator * (float s, const Vector3d& v)
{
    Vector3d Scaled(v.x * s, v.y * s, v.z * s);
    return(Scaled);
};
```

Bu sınıf, `SphereLib` içerisinde kürelerin merkezlerini, ışınların yönlerini veya çarpışma testleri için noktaları temsil etmek gibi çeşitli geometrik hesaplamalarda temel bir rol oynar. 