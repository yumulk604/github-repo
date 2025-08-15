# EffectLib Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/EffectLib/` klasöründe bulunan `EffectLib` modülünün amacını, temel mimarisini ve içerdiği ana bileşenleri açıklamaktadır.

`EffectLib`, Metin2 istemcisindeki oyun içi görsel efektlerin oluşturulması, yönetilmesi, güncellenmesi ve render edilmesi (çizilmesi) için kullanılan kütüphanedir. Bu kütüphane, yetenek (skill) efektleri, büyü efektleri, karakter veya nesne üzerindeki görsel işaretler, ortam efektleri gibi çok çeşitli görsel öğelerin temelini oluşturur.

## Genel Amaç ve Mimari

`EffectLib` kütüphanesi, modüler bir yaklaşımla farklı efekt türlerini destekleyecek şekilde tasarlanmıştır. Temel mimarisi şu bileşenler etrafında şekillenir:

*   **Efekt Verisi (`EffectData`, `*Data`):** Bir efektin nasıl görüneceğini ve davranacağını tanımlayan şablon veya prototip verileridir. Bu veriler genellikle harici dosyalardan (örneğin, özel efekt tanım dosyaları) yüklenir ve efektin temel özelliklerini (kullanılacak parçacık sistemleri, meshler, ışıklar, ömür süresi vb.) içerir.
    *   Örnekler: `EffectData.h`, `ParticleSystemData.h`, `SimpleLightData.h`, `EffectMesh.h`
*   **Efekt Elemanı (`EffectElementBase`, `*Property`):** Bir efektin yapı taşlarıdır. Parçacık yayıcılar (emitter), meshler, ışıklar gibi farklı türlerde elemanlar olabilir. Bunların özellikleri (`*Property` dosyaları) efekt verisi içinde tanımlanır.
    *   Örnekler: `EffectElementBase.h`, `ParticleProperty.h`, `EmitterProperty.h`
*   **Efekt Örneği (`EffectInstance`, `*Instance`):** Oyun dünyasında aktif olarak görünen ve güncellenen her bir efektin somut örneğidir. Bir `EffectData` şablonundan oluşturulur ve kendi yaşam döngüsüne (pozisyon, zamanlama, aktif parçacıklar vb.) sahiptir.
    *   Örnekler: `EffectInstance.h`, `ParticleSystemInstance.h`, `SimpleLightInstance.h`, `EffectMeshInstance.h`, `EffectElementBaseInstance.h`
*   **Efekt Yöneticisi (`EffectManager`):** Tüm efekt verilerini yükler, yönetir ve ihtiyaç duyulduğunda yeni efekt örnekleri (`EffectInstance`) oluşturur. Aynı zamanda aktif efekt örneklerinin güncellenmesinden ve yok edilmesinden sorumludur.
    *   Örnek: `EffectManager.h`
*   **Yardımcı Bileşenler:** Zamanlama (`FrameController`), özel güncelleme mantıkları (`EffectUpdateDecorator`), temel tür tanımları (`Type`) gibi destekleyici yapılar.

Bu yapı, farklı efekt türlerini (parçacık, mesh, ışık vb.) bir araya getirerek karmaşık görsel efektler oluşturmaya olanak tanır.

---

## Dosya Bazlı Detaylandırma

### `EffectManager.h`

*   **Amaç:** Efekt sisteminin merkezi yönetim birimidir. Efekt verilerini yüklemek, saklamak, yeni efekt örnekleri (instance) oluşturmak, mevcut örnekleri güncellemek, render etmek ve yok etmekten sorumludur.
*   **Kalıtım:** `CScreen` (muhtemelen temel çizim/ekran yönetimi sınıfı) ve `CSingleton<CEffectManager>` (Singleton deseni) sınıflarından türetilmiştir. Bu, yöneticinin sistem genelinde tekil olmasını ve çizim döngüsüyle doğrudan entegre olabilmesini sağlar.
*   **Temel Bileşenler ve İşlevler:**
    *   **Veri Yapıları:**
        *   `TEffectDataMap`: Efekt verilerini (`CEffectData*`) DWORD ID'lerine göre saklayan bir `std::map`.
        *   `TEffectInstanceMap`: Aktif efekt örneklerini (`CEffectInstance*`) benzersiz DWORD indekslerine göre saklayan bir `std::map`.
        *   `TEffectCacheMap`: Önbelleğe alınmış efekt örneklerini (`CEffectInstance*`) saklayan bir `std::map` (muhtemelen tekrar tekrar kullanılan efektler için).
    *   **Efekt Verisi Yönetimi:**
        *   `RegisterEffect(const char* c_szFileName, ...)` / `RegisterEffect2(const char* c_szFileName, ...)`: Belirtilen efekt tanım dosyasını yükler, bir `CEffectData` nesnesi oluşturur, bunu `m_kEftDataMap` içinde saklar ve bir ID (dosya adı CRC'si olabilir) döndürür. Efekt verilerinin önbelleğe alınmasını (`isNeedCache`) destekler.
        *   `GetEffectData(DWORD dwID, CEffectData** ppEffect)`: Verilen ID'ye karşılık gelen `CEffectData` işaretçisini döndürür.
        *   `__DestroyEffectDataMap()` / `__DestroyEffectCacheMap()`: Saklanan efekt verilerini ve önbelleği temizler.
    *   **Efekt Örneği (Instance) Yönetimi:**
        *   `CreateEffect(DWORD dwID, ...)` / `CreateEffect(const char* c_szFileName, ...)`: Verilen ID veya dosya adıyla ilişkilendirilmiş `CEffectData`'yı kullanarak yeni bir `CEffectInstance` oluşturur, bunu `m_kEftInstMap`'e ekler ve örneğe ait benzersiz bir indeks döndürür. Efektin başlangıç pozisyonu, rotasyonu ve parçacık ölçeği gibi parametreler alır.
        *   `CreateEffectInstance(DWORD dwInstanceIndex, DWORD dwID, ...)`: Belirli bir indeks numarasıyla (muhtemelen önceden ayrılmış) yeni bir efekt örneği oluşturur.
        *   `DestroyEffectInstance(DWORD dwInstanceIndex)`: Verilen indeksteki efekt örneğini bulur ve yok eder (`delete`).
        *   `DeactiveEffectInstance(DWORD dwInstanceIndex)` / `ActiveEffectInstance(DWORD dwInstanceIndex)`: Bir efekt örneğini geçici olarak devre dışı bırakır veya yeniden etkinleştirir. Optimizasyon (görüş alanı dışı efektler vb.) veya grafik ayarları (efektleri kapatma/açma) için kullanılabilir.
        *   `DeleteAllInstances()`: Tüm aktif efekt örneklerini yok eder.
        *   `__DestroyEffectInstanceMap()`: Efekt örneği haritasını temizler.
        *   `IsAliveEffect(DWORD dwInstanceIndex)`: Verilen indeksteki efekt örneğinin hala var olup olmadığını kontrol eder.
    *   **Güncelleme ve Render Döngüsü:**
        *   `Update()`: Tüm aktif efekt örneklerinin (`m_kEftInstMap` içindekiler) `Update()` metodunu çağırarak durumlarını (parçacık pozisyonları, animasyonlar, ömürler vb.) ilerletir.
        *   `UpdateSound()`: Efektlerle ilişkili seslerin güncellenmesini tetikler (muhtemelen `CEffectInstance` içindeki ilgili fonksiyonlar çağrılır).
        *   `Render()`: Tüm aktif ve görünür efekt örneklerinin `Render()` metodunu çağırarak ekrana çizilmelerini sağlar. `CScreen` taban sınıfının çizim döngüsüyle entegre çalışır.
        *   `GetRenderingEffectCount()`: O anda render edilen efekt sayısını döndürür.
    *   **Seçili Efekt Kontrolü (`m_pSelectedEffectInstance`):**
        *   `SelectEffectInstance(DWORD dwInstanceIndex)`: Yönetilecek (konumunu/rotasyonunu ayarlama vb.) efekt örneğini seçer.
        *   `SetEffectInstancePosition(...)`, `SetEffectInstanceRotation(...)`, `SetEffectInstanceGlobalMatrix(...)`: Seçili olan efekt örneğinin pozisyonunu, rotasyonunu veya dünya matrisini günceller.
        *   `ShowEffect()` / `HideEffect()`: Seçili efekt örneğini görünür veya gizli yapar.
    *   **Diğer Fonksiyonlar:**
        *   `Destroy()`: Yöneticinin tüm kaynaklarını (veri haritaları, örnek haritaları vb.) temizler.
        *   `GetInfo(std::string* pstInfo)`: Yöneticinin durumu hakkında bilgi (örneğin, aktif efekt sayısı) içeren bir string oluşturur.
*   **Kullanım Alanı:** Oyunun herhangi bir yerinde görsel efekt oluşturmak, güncellemek ve göstermek gerektiğinde `CEffectManager` örneği (`CEffectManager::Instance()`) kullanılır. Oyun dünyasındaki karakterler, nesneler veya ortamlar tarafından tetiklenen tüm görsel efektlerin yaşam döngüsünü merkezi olarak yönetir.

#### Implementasyon Detayları (`EffectManager.cpp`)

*   **`Update()`:** Aktif efekt örnekleri (`m_kEftInstMap`) üzerinde gezer. Her örneğin `Update()` metodunu çağırır. `isAlive()` `false` dönen örnekleri haritadan çıkarır ve `CEffectInstance::Delete()` ile bellek havuzuna iade eder.
*   **`Render()`:** İki moda sahiptir:
    *   **Sıralı (Varsayılan):** Tüm aktif örnekleri geçici bir vektöre alır, `CEffectInstance::LessRenderOrder()` kullanarak sıralar (saydamlık için) ve sonra her birinin `Render()` metodunu çağırır.
    *   **Sırasız (`m_isDisableSortRendering` true ise):** Doğrudan harita üzerinde gezer ve `Render()` çağırır (daha hızlı ama saydamlık sorunları olabilir).
*   **`RegisterEffect(...)`:** Dosya adının CRC'sini hesaplar. Eğer CRC `m_kEftDataMap`'te yoksa, `CEffectData::New()` ile nesne oluşturur, `LoadScript()` ile veriyi yükler ve haritaya ekler. `isNeedCache` true ise, aynı zamanda `CEffectInstance::New()` ile bir örnek oluşturup `m_kEftCacheMap`'e ekler.
*   **`CreateEffect(...)`:** Benzersiz bir örnek ID'si (`GetEmptyIndex()`) alır, `CreateEffectInstance()` ile örneği oluşturur, `SelectEffectInstance()` ile seçer ve verilen pozisyon/rotasyon ile dünya matrisini ayarlar (`SetEffectInstanceGlobalMatrix()`).
*   **`CreateEffectInstance(...)`:** Verilen ID ile `GetEffectData()` kullanarak `CEffectData`'yı bulur. `CEffectInstance::New()` ile örnek oluşturur, ölçekleri ayarlar ve `SetEffectDataPointer()` çağırarak örneği veriye bağlar (bu sırada alt eleman örnekleri oluşturulur). Son olarak örneği `m_kEftInstMap`'e ekler.
*   **`DestroyEffectInstance(...)`:** Verilen ID ile örneği `m_kEftInstMap`'ten bulur, haritadan siler ve `CEffectInstance::Delete()` ile havuzuna iade eder.
*   **`SelectEffectInstance(...)`:** Verilen ID ile örneği bulur ve `m_pSelectedEffectInstance` işaretçisine atar.

(Bu bölüme, EffectLib içindeki belirli dosyaların detaylı açıklamaları eklenecektir.)

### `EffectData.h`

*   **Amaç:** Bir görsel efektin şablonunu veya prototipini temsil eder. Efekti oluşturan tüm statik bileşenlerin (parçacık sistemleri, meshler, ışıklar, sesler) tanımlarını ve özelliklerini bir arada tutar.
*   **Bağımlılıklar:** `ParticleSystemData.h`, `EffectMesh.h`, `SimpleLightData.h` ve ses sistemiyle ilgili `MilesLib/Type.h` dosyalarına bağımlıdır.
*   **Temel Özellikler/İçerik (`CEffectData` Sınıfı):**
    *   **Bileşen Veri Koleksiyonları:**
        *   `TParticleVector m_ParticleVector`: Efekte ait parçacık sistemi tanımlarını (`CParticleSystemData*`) içeren bir vektör.
        *   `TMeshVector m_MeshVector`: Efekte ait mesh tanımlarını (`CEffectMeshScript*`) içeren bir vektör.
        *   `TLightVector m_LightVector`: Efekte ait ışık tanımlarını (`CLightData*`) içeren bir vektör.
        *   `NSound::TSoundInstanceVector m_SoundInstanceVector`: Efektle ilişkilendirilmiş ses örneklerini içeren bir vektör.
    *   **Yükleme Fonksiyonları:**
        *   `LoadScript(const char* c_szFileName)`: Ana efekt tanım dosyasını (muhtemelen özel bir metin formatı) yükler. Dosya içeriğini ayrıştırarak parçacık, mesh ve ışık verilerini oluşturur (aşağıdaki sanal ayırma fonksiyonlarını kullanarak) ve ilgili vektörlere ekler. Ayrıca sınırlayıcı küre bilgilerini hesaplar.
        *   `LoadSoundScriptData(const char* c_szFileName)`: Efekt için ses bilgilerini ayrı bir script dosyasından yükler ve `m_SoundInstanceVector`'ü doldurur.
    *   **Veri Erişim Metotları:**
        *   `GetParticleCount()`, `GetMeshCount()`, `GetLightCount()`: İçerdiği parçacık sistemi, mesh ve ışık sayısını döndürür.
        *   `GetParticlePointer(DWORD dwPosition)`, `GetMeshPointer(DWORD dwPosition)`, `GetLightPointer(DWORD dwPosition)`: İlgili vektördeki belirli bir indekste bulunan bileşen verisine işaretçi döndürür.
        *   `GetSoundInstanceVector()`: Ses örneği vektörüne erişim sağlar.
    *   **Sınırlayıcı Küre (Bounding Sphere):**
        *   `GetBoundingSphereRadius()`, `GetBoundingSpherePosition()`: Efektin 3D uzayda kapladığı tahmini alanı temsil eden sınırlayıcı kürenin bilgilerini döndürür. Bu, görünürlük tespiti (culling) gibi optimizasyonlar için kullanılır.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CEffectData> ms_kPool`: `CEffectData` nesnelerinin oluşturulması ve silinmesi için statik bir dinamik bellek havuzu kullanılır. Bu, sık kullanılan efekt verilerinin tekrar tekrar yüklenmesi yerine havuzdan hızlıca alınmasını/iade edilmesini sağlayarak performansı artırır.
        *   `static CEffectData* New()`: Havuzdan yeni bir `CEffectData` nesnesi alır veya oluşturur.
        *   `static void Delete(CEffectData* pkData)`: Kullanılan `CEffectData` nesnesini havuza geri iade eder.
        *   `static void DestroySystem()`: Bellek havuzunu temizler.
    *   **Sanal Ayırma Fonksiyonları (Virtual Allocators):**
        *   `AllocParticle()`, `AllocMesh()`, `AllocLight()`: `LoadScript` tarafından çağrılan sanal fonksiyonlardır. Varsayılan olarak `CParticleSystemData`, `CEffectMeshScript` ve `CLightData` nesneleri oluştururlar. Bu mekanizma, `CEffectData`'dan türetilmiş sınıfların, farklı türde veya özelleştirilmiş efekt bileşen verileri oluşturmasına olanak tanır.
        *   **Diğer Metotlar:**
        *   `Clear()`: Efekt verisinin içeriğini (vektörler vb.) temizler.
        *   `GetFileName()`: Efekt verisinin yüklendiği dosya adını döndürür.
*   **Kullanım Alanı:** `CEffectManager` tarafından, belirli bir efekt türünün nasıl oluşturulacağını tanımlayan bir şablon olarak kullanılır. Oyun başladığında veya ihtiyaç duyulduğunda efekt script dosyaları okunarak `CEffectData` nesneleri oluşturulur ve yönetici tarafından saklanır. Daha sonra bu veriler kullanılarak oyun dünyasında `CEffectInstance`'lar yaratılır.

#### Implementasyon Detayları (`EffectData.cpp`)

*   **`LoadScript(...)`:**
    *   `CTextFileLoader` ile script dosyasını yükler.
    *   Önce "boundingsphereradius" ve "boundingsphereposition" değerlerini okur.
    *   Sonra script'teki ana düğümleri ("mesh", "particle", "light") döngüyle gezer.
    *   Her düğüm için ilgili `Alloc*()` fonksiyonunu (örn: `AllocParticle()`) çağırır. Bu fonksiyon, ilgili veri sınıfının (`CParticleSystemData`) `New()` metodunu kullanarak havuzdan bir nesne alır ve bunu `CEffectData`'nın iç vektörüne (`m_ParticleVector`) ekler.
    *   Alınan nesne işaretçisi üzerinden o nesnenin kendi `LoadScript()` metodunu çağırarak, ilgili script bloğunun okunmasını o sınıfa devreder.
    *   Son olarak, efekt dosyası adına göre (örn: `effect.mse` -> `sound/effect.mss`) bir ses script dosyası arar ve varsa `LoadSoundScriptData()` ile yükler.
*   **`LoadSoundScriptData(...)`:** Harici ses kütüphanesi fonksiyonları (`NSound::*`) aracılığıyla ses bilgilerini yükler ve `m_SoundInstanceVector`'ü doldurur.
*   **`AllocParticle()`, `AllocMesh()`, `AllocLight()`:** İlgili veri sınıflarının (`CParticleSystemData`, `CEffectMeshScript`, `CLightData`) statik `New()` metotlarını çağırarak havuzdan nesne alır, bunları ilgili iç vektöre ekler ve işaretçiyi döndürür.
*   **`Clear()`:** Sınırlayıcı küre bilgilerini sıfırlar ve özel `__Clear*Vector()` fonksiyonlarını çağırarak tüm eleman verilerini (`Delete()` metodlarını çağırarak havuza iade eder) ve iç vektörleri temizler.
*   **`Delete()` (Statik):** Nesneyi havuza iade etmeden önce (`ms_kPool.Free()`) nesnenin `Clear()` metodunu çağırır.
*   **`DestroySystem()` (Statik):** Kendi havuzunu ve ayrıca `CParticleSystemData`, `CEffectMeshScript`, `CLightData` havuzlarını da yok eder.

(Bu bölüme, EffectLib içindeki belirli dosyaların detaylı açıklamaları eklenecektir.)

### `EffectInstance.h`

*   **Amaç:** Oyun dünyasında aktif olarak var olan ve çalışan bir görsel efekt örneğini temsil eder. Bir `CEffectData` şablonundan oluşturulur ve kendi dinamik durumunu (pozisyon, zamanlama, aktif parçacıklar vb.) yönetir.
*   **Kalıtım:** `CGraphicObjectInstance` sınıfından türetilmiştir. Bu, efekt örneklerinin oyunun grafik motoru ve sahne yönetimi sistemiyle entegre olmasını sağlar (güncelleme, render etme, görünürlük kontrolü vb.).
*   **Bağımlılıklar:** `EffectData.h`, `EffectElementBaseInstance.h`, `EffectMeshInstance.h`, `ParticleSystemInstance.h`, `SimpleLightInstance.h` gibi efekt bileşenlerinin örnek sınıflarına ve `EterLib/GrpObjectInstance.h`, `EterLib/Pool.h`, `MilesLib/Type.h` gibi temel kütüphane dosyalarına bağımlıdır.
*   **Temel Özellikler/İçerik (`CEffectInstance` Sınıfı):**
    *   **Yaşam Döngüsü ve Durum Yönetimi:**
        *   `isAlive()`: Efekt örneğinin hala aktif olup olmadığını (yok edilmemiş veya süresi dolmamış) kontrol eder.
        *   `SetActive()`, `SetDeactive()`: Örneği etkinleştirir veya devre dışı bırakır. Devre dışı örnekler güncellenmez ve render edilmez.
        *   `Clear()`: Örneği başlangıç durumuna sıfırlar ve içerdiği tüm bileşen örneklerini (parçacık, mesh, ışık) temizler.
        *   `m_isAlive`: Efektin aktiflik bayrağı.
        *   `m_dwFrame`: Efektin başlangıcından itibaren geçen kare veya zaman birimi sayacı.
        *   `m_fLastTime`: Son güncelleme anındaki zamanı depolar (delta zaman hesaplamaları için).
    *   **Efekt Verisi ile İlişki:**
        *   `SetEffectDataPointer(CEffectData* pEffectData)`: Bu örneğin temel alacağı `CEffectData` şablonunu ayarlar. Bu fonksiyon çağrıldığında, `CEffectData` içindeki tanımlara göre gerekli parçacık sistemi, mesh ve ışık örneklerini (`CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance`) oluşturur ve ilgili vektörlerde (`m_*InstanceVector`) saklar.
        *   `m_pkEftData`: İlişkili `CEffectData` nesnesinin işaretçisi.
    *   **Bileşen Örnekleri Koleksiyonları:**
        *   `m_ParticleInstanceVector`: Bu efekte ait aktif parçacık sistemi örneklerini tutan vektör.
        *   `m_MeshInstanceVector`: Bu efekte ait aktif mesh örneklerini tutan vektör.
        *   `m_LightInstanceVector`: Bu efekte ait aktif ışık örneklerini tutan vektör.
    *   **Güncelleme ve Render:**
        *   `OnUpdate()`: (Sanal fonksiyon) Efektin bir sonraki adıma geçmesini sağlar. İçerdiği tüm aktif bileşen örneklerinin `Update()` fonksiyonlarını çağırır.
        *   `UpdateSound()`: Efektle ilişkili seslerin güncellenmesini tetikler.
        *   `OnRender()`: (Sanal fonksiyon) Efektin ekrana çizilmesini sağlar. İçerdiği tüm aktif ve görünür bileşen örneklerinin `Render()` fonksiyonlarını çağırır.
        *   `LessRenderOrder(CEffectInstance* pkEftInst)`: Diğer bir efekt örneğiyle render sırasını karşılaştırmak için kullanılır (muhtemelen saydamlık sıralaması için).
    *   **Konumlandırma, Ölçekleme ve Dönüşüm:**
        *   `SetGlobalMatrix(const D3DXMATRIX& c_rmatGlobal)`: Efektin oyun dünyasındaki genel dönüşüm matrisini (pozisyon, rotasyon, ölçek) ayarlar. Bu matris, tüm bileşen örneklerinin konumlandırılmasında temel alınır.
        *   `GetGlobalMatrix()`: Ayarlanan global matrisi döndürür.
        *   `m_matGlobal`: Efektin dünya matrisini saklar.
        *   `SetParticleScale(float fParticleScale)` / `GetParticleScale()`: Efektin parçacıklarının genel ölçeğini ayarlar/alır.
        *   `SetMeshScale(D3DXVECTOR3 rv3MeshScale)` / `GetMeshScale()`: Efektin meshlerinin genel ölçeğini ayarlar/alır.
    *   **Sınırlayıcı Küre:**
        *   `GetBoundingSphere(D3DXVECTOR3& v3Center, float& fRadius)`: Efektin mevcut durumuna göre hesaplanmış sınırlayıcı küre bilgilerini (merkez ve yarıçap) döndürür. Bu, `CEffectData`'daki statik bilgiye ek olarak örneğin dinamik durumunu yansıtır.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CEffectInstance> ms_kPool`: `CEffectInstance` nesneleri için statik bir dinamik bellek havuzu.
        *   `static CEffectInstance* New()`: Havuzdan yeni bir `CEffectInstance` alır.
        *   `static void Delete(CEffectInstance* pkEftInst)`: Kullanılan `CEffectInstance`'ı havuza iade eder.
        *   `static void DestroySystem()`: Bellek havuzunu temizler.
        *   `static int ms_iRenderingEffectCount`: O anda render edilen efekt örneklerinin sayısını tutan statik sayaç.
        *   `static void ResetRenderingEffectCount()`: Sayacı sıfırlar.
        *   `static int GetRenderingEffectCount()`: Sayaç değerini döndürür.
*   **Kullanım Alanı:** `CEffectManager` tarafından `CreateEffect` veya benzeri fonksiyonlarla oluşturulur. Oyun döngüsü içinde `Update()` ve `Render()` fonksiyonları çağrılarak efektin canlandırılması ve görselleştirilmesi sağlanır. Efektin ömrü dolduğunda veya manuel olarak `DestroyEffectInstance` çağrıldığında yok edilir.

#### Implementasyon Detayları (`EffectInstance.cpp`)

*   **`SetEffectDataPointer(...)`:** Bu fonksiyon, şablon `CEffectData`'yı canlı bir efekte dönüştürür:
    *   `CEffectData` içindeki parçacık, mesh ve ışık tanımları üzerinde döngüyle gezer.
    *   Her tanım için ilgili `__Set*Data` (örn: `__SetParticleData`) fonksiyonunu çağırır.
    *   **`__Set*Data` Fonksiyonları:**
        *   İlgili örnek sınıfının (`CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance`) `New()` metoduyla havuzdan bir örnek alır.
        *   Örneğin `SetDataPointer()` metodunu çağırarak veri bloğunu bağlar (bu sırada örnek kendi başlatmasını yapar, örn: `CEffectMeshInstance` mesh kaynağını yükler).
        *   Örneğin `SetLocalMatrixPointer()` ile ana efektin matrisini verir.
        *   Parçacık/mesh için ölçekleri ayarlar.
        *   Oluşturulan örneği `CEffectInstance`'ın kendi `m_*InstanceVector`'üne ekler.
    *   Ses verisi işaretçisini ayarlar.
    *   Sınırlayıcı küre bilgilerini alır ve kaydeder.
*   **`OnUpdate()`:**
    *   `FEffectUpdator` functor'ını kullanarak tüm alt eleman örneklerinin (`m_*InstanceVector` içindekiler) `Update()` metodlarını çağırır.
    *   Eğer en az bir alt eleman hala aktifse (`Update()` true döndürürse), `CEffectInstance`'ın kendi `m_isAlive` bayrağını `true` yapar, aksi halde `false` yapar.
*   **`OnRender()`:**
    *   Genel saydam efekt render durumlarını (`STATEMANAGER` ile) ayarlar (AlphaBlend açık, ZWrite kapalı vb.).
    *   Tüm parçacık ve mesh örneklerinin (`m_ParticleInstanceVector`, `m_MeshInstanceVector`) `Render()` metotlarını çağırır (ışıklar çizilmez).
    *   Render durumlarını geri yükler.
*   **`Clear()`:** Tüm alt eleman örnek vektörleri üzerinde gezer, her örnek için `Delete()` (havuza iade) çağırır ve vektörleri temizler.
*   **Bellek Yönetimi:** `New`, `Delete`, `DestroySystem` fonksiyonları `CEffectData` ile benzer şekilde havuzlar ve alt eleman havuzları üzerinden çalışır.

(Bu bölüme, EffectLib içindeki belirli dosyaların detaylı açıklamaları eklenecektir.)

### `ParticleSystemData.h`

*   **Amaç:** Bir parçacık sisteminin tüm statik özelliklerini ve davranışlarını tanımlayan veri yapısıdır. Parçacıkların nasıl yayılacağını, nasıl görüneceğini ve zamanla nasıl değişeceğini belirleyen parametreleri içerir.
*   **Kalıtım:** `CEffectElementBase` sınıfından türetilmiştir. Bu, parçacık sistemi verilerinin genel efekt elemanı yapısının bir parçası olmasını ve temel yükleme/kaydetme mekanizmalarını miras almasını sağlar.
*   **Bağımlılıklar:** `EffectElementBase.h`, `EmitterProperty.h`, `ParticleProperty.h` ve `EterLib/TextFileLoader.h` dosyalarına bağımlıdır.
*   **Temel Özellikler/İçerik (`CParticleSystemData` Sınıfı):**
    *   **Temel Özellik Yapıları:**
        *   `m_EmitterProperty (CEmitterProperty)`: Parçacık yayıcısının (emitter) özelliklerini tanımlar. Bu özellikler arasında yayıcının türü (nokta, küre, koni vb.), parçacık yayma oranı, maksimum parçacık sayısı, yayılma ömrü, yayılma hızı ve açısı gibi parametreler bulunabilir.
        *   `m_ParticleProperty (CParticleProperty)`: Tekil parçacıkların özelliklerini tanımlar. Bunlar arasında parçacığın ömrü, başlangıç ve bitiş rengi, başlangıç ve bitiş boyutu, hareket özellikleri (hız, ivme, dönme hızı), kullanılacak doku (texture), harmanlama (blending) modu gibi görsel ve fiziksel nitelikler yer alır.
        *   `GetEmitterPropertyPointer()`: `m_EmitterProperty` nesnesine işaretçi döndürür.
        *   `GetParticlePropertyPointer()`: `m_ParticleProperty` nesnesine işaretçi döndürür.
    *   **Yükleme ve Temizleme (Devralınan ve Özelleştirilen Fonksiyonlar):**
        *   `OnLoadScript(CTextFileLoader& rTextFileLoader)`: (Korumalı Sanal) `CEffectElementBase`'den gelen bu fonksiyon, efekt script dosyasından parçacık sistemine özgü verileri (yayıcı ve parçacık özellikleri) okumak için özelleştirilmiştir.
        *   `OnClear()`: (Korumalı Sanal) `m_EmitterProperty` ve `m_ParticleProperty`'yi varsayılan değerlerine sıfırlar.
        *   `OnIsData()`: (Korumalı Sanal) Gerekli yayıcı ve parçacık özelliklerinin yüklenip yüklenmediğini kontrol eder.
    *   **Doku Yönetimi:**
        *   `ChangeTexture(const char* c_szFileName)`: Parçacıklar için kullanılacak doku dosyasını çalışma zamanında değiştirir. Bu, aynı parçacık davranışını farklı görsellerle kullanmaya olanak tanır.
    *   **Dekoratör Entegrasyonu:**
        *   `BuildDecorator(CParticleInstance* pInstance)`: Verilen parçacık sistemi örneği (`CParticleInstance`) için özel davranışlar veya güncellemeler ekleyen "dekoratörler" (`CEffectUpdateDecorator` türevleri) oluşturur ve ayarlar. Bu, parçacıkların hareketlerini (yerçekimi, rüzgar, hedefe yönelme vb.) veya diğer özelliklerini zamanla karmaşık şekillerde değiştirmek için kullanılır.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CParticleSystemData> ms_kPool`: `CParticleSystemData` nesneleri için statik bir dinamik bellek havuzu.
        *   `static CParticleSystemData* New()`: Havuzdan yeni bir `CParticleSystemData` nesnesi alır.
        *   `static void Delete(CParticleSystemData* pkData)`: Kullanılan nesneyi havuza iade eder.
*   **Kullanım Alanı:** `CEffectData` içinde bir veya daha fazla `CParticleSystemData` tanımlanır. Her biri, genel bir efektin parçacık tabanlı bir bileşenini oluşturur. `CEffectManager`, bir `CEffectInstance` oluştururken, bu `CParticleSystemData` tanımlarına dayanarak `CParticleSystemInstance`'lar yaratır.

### `ParticleSystemInstance.h`

*   **Amaç:** Bir `CParticleSystemData` şablonuna dayanarak oyun dünyasında aktif olan ve çalışan bir parçacık sistemini temsil eder. Parçacıkların oluşturulması, güncellenmesi, render edilmesi ve ömürlerinin yönetilmesinden sorumludur.
*   **Kalıtım:** `CEffectElementBaseInstance` sınıfından türetilmiştir. Bu, parçacık sistemi örneğinin genel efekt elemanı örneği hiyerarşisine dahil olmasını sağlar.
*   **Bağımlılıklar:** `EffectElementBaseInstance.h`, `ParticleInstance.h`, `ParticleProperty.h`, `EmitterProperty.h` ve `EterLib` içindeki `GrpScreen.h`, `StateManager.h`, `GrpImageInstance.h` gibi grafik ve durum yönetimi dosyalarına bağımlıdır.
*   **Temel Özellikler/İçerik (`CParticleSystemInstance` Sınıfı):**
    *   **Veri Bağlantısı ve Başlatma:**
        *   `OnSetDataPointer(CEffectElementBase* pElement)`: (Korumalı Sanal) Temel alınacak `CParticleSystemData` nesnesini (`pElement` olarak gelen) ayarlar. Bu `CParticleSystemData` içinden `m_pParticleProperty` ve `m_pEmitterProperty` işaretçilerini alır. Parçacıklar için kullanılacak doku(lar)ı (`CGraphicImageInstance`) yükler/ayarlar ve `m_kVct_pkImgInst` vektörüne ekler.
        *   `m_pData (CParticleSystemData*)`: İlişkili parçacık sistemi verisi.
        *   `m_pParticleProperty (CParticleProperty*)`: Kolay erişim için parçacık özelliklerine işaretçi.
        *   `m_pEmitterProperty (CEmitterProperty*)`: Kolay erişim için yayıcı özelliklerine işaretçi.
    *   **Parçacık Yönetimi:**
        *   `m_ParticleInstanceListVector (TParticleInstanceListVector)`: Aktif parçacık örneklerini (`CParticleInstance*`) tutan bir `std::vector<std::list<CParticleInstance*>>`. Dış vektör, farklı dokulara sahip parçacık gruplarını ayırabilir.
        *   `m_kVct_pkImgInst (TImageInstanceVector)`: Parçacıklar tarafından kullanılan doku örneklerini (`CGraphicImageInstance*`) tutar.
        *   `CreateParticles(float fElapsedTime)`: Belirlenen yayıcı özelliklerine (`m_pEmitterProperty`) göre geçen süre (`fElapsedTime`) içinde yeni parçacıklar (`CParticleInstance`) oluşturur. Yeni parçacıkların başlangıç durumlarını (pozisyon, hız, ömür vb.) ayarlar ve `m_ParticleInstanceListVector`'e ekler.
        *   `m_fEmissionResidue`: Parçacık yayma hızının kesirli kısımlarını bir sonraki güncellemeye taşır (daha düzgün yayılım için).
        *   `m_dwCurrentEmissionCount`: Yayılan toplam parçacık sayısını izler.
        *   `m_iLoopCount`: Yayıcının döngü sayısını (eğer döngüsel ise) izler.
    *   **Güncelleme ve Render:**
        *   `OnUpdate(float fElapsedTime)`: (Korumalı Sanal) Parçacık sisteminin durumunu günceller:
            *   `CreateParticles(fElapsedTime)` ile yeni parçacıklar oluşturur.
            *   `m_ParticleInstanceListVector` içindeki tüm `CParticleInstance`'ların `Update()` fonksiyonlarını çağırarak durumlarını (pozisyon, renk, boyut vb.) ilerletir.
            *   Ömrü dolan parçacıkları listeden çıkarır ve bellek havuzuna iade eder.
            *   Yayıcının ömrünü kontrol eder ve gerekirse sistemi sonlandırır.
        *   `OnRender()`: (Korumalı Sanal) Parçacık sistemini ekrana çizer:
            *   `ForEachParticleRendering` şablon fonksiyonunu kullanarak `m_ParticleInstanceListVector` içindeki tüm görünür parçacıkları (`InFrustum` ile kontrol edilir) render eder.
            *   Her parçacık grubu için ilgili dokuyu (`m_kVct_pkImgInst`) ayarlar (`STATEMANAGER.SetTexture`).
            *   Parçacıkların gerçek çizimi (vertex buffer oluşturma, çizim çağrısı) muhtemelen `CParticleInstance` veya `ForEachParticleRendering` içindeki bir functor aracılığıyla yapılır.
        *   `InFrustum(CParticleInstance* pInstance)`: Verilen parçacığın kamera görüş alanı (frustum) içinde olup olmadığını kontrol ederek gereksiz çizimleri engeller.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CParticleSystemInstance> ms_kPool`: `CParticleSystemInstance` nesneleri için statik bir dinamik bellek havuzu.
        *   `static CParticleSystemInstance* New()`: Havuzdan yeni bir `CParticleSystemInstance` alır.
        *   `static void Delete(CParticleSystemInstance* pkData)`: Kullanılan nesneyi havuza iade eder.
*   **Kullanım Alanı:** Bir `CEffectInstance` içinde, `CEffectData`'da tanımlanmış her bir `CParticleSystemData` için bir `CParticleSystemInstance` oluşturulur. Efektin parçacık tabanlı bileşenlerinin canlandırılması ve görselleştirilmesinden sorumludur.

### `EmitterProperty.h`

*   **Amaç:** Bir parçacık sistemindeki yayıcının (emitter) tüm davranışsal, geometrik ve zamana bağlı özelliklerini tanımlar. Parçacıkların nereden, ne zaman, ne kadar, hangi hızda, hangi yönde ve hangi başlangıç özellikleriyle (ömür, boyut vb.) yayılacağını kontrol eder.
*   **Bağımlılıklar:** `Type.h` (muhtemelen `TTimeEventTableFloat` gibi zamanla değişen değer tabloları için).
*   **Temel Özellikler/İçerik (`CEmitterProperty` Sınıfı):**
    *   **Yayıcı Temel Parametreleri:**
        *   `m_dwMaxEmissionCount (DWORD)`: Yayıcının toplamda yayabileceği maksimum parçacık sayısı.
        *   `m_fCycleLength (float)`: Yayıcı döngüsünün süresi (saniye cinsinden).
        *   `m_bCycleLoopFlag (BOOL)`: Yayıcı döngüsünün tekrarlanıp tekrarlanmayacağı.
        *   `m_iLoopCount (int)`: Döngüsel yayıcı için tekrar sayısı (örneğin, -1 sonsuz döngü olabilir).
    *   **Yayıcı Şekli ve Davranışı:**
        *   `m_byEmitterShape (BYTE)`: Yayıcının geometrik şekli (enum: `EMITTER_SHAPE_POINT`, `EMITTER_SHAPE_ELLIPSE`, `EMITTER_SHAPE_SQUARE`, `EMITTER_SHAPE_SPHERE`).
        *   `m_byEmitterAdvancedType (BYTE)`: Yayıcının gelişmiş yayma tipi (enum: `EMITTER_ADVANCED_TYPE_FREE`, `EMITTER_ADVANCED_TYPE_OUTER`, `EMITTER_ADVANCED_TYPE_INNER`). Parçacıkların şeklin içinden, dışından veya serbestçe yayılmasını belirleyebilir.
        *   `m_bEmitFromEdgeFlag (BOOL)`: Parçacıkların yayıcı şeklinin kenarından mı yoksa tüm hacminden/alanından mı yayılacağını belirtir.
        *   `m_v3EmittingSize (D3DXVECTOR3)`: Yayıcı şeklinin boyutları (örneğin, kare için en/boy/derinlik).
        *   `m_fEmittingRadius (float)`: Küresel veya eliptik yayıcılar için yarıçap.
        *   `m_v3EmittingDirection (D3DXVECTOR3)`: Parçacıkların ana yayılma yönü.
    *   **Zamana Bağlı Özellikler (`TTimeEventTableFloat` Kullanarak):**
        *   Bu özellikler, yayıcının ömrü boyunca veya bir döngü içinde değerlerinin nasıl değişeceğini tanımlayan zaman çizelgeleridir (`Type.h` içinde tanımlı `TTimeEventTableFloat`).
        *   `m_TimeEventEmittingSize`: Yayıcı boyutunun zamanla değişimi.
        *   `m_TimeEventEmittingAngularVelocity`: Yayılan parçacıkların başlangıçtaki açısal hızının zamanla değişimi.
        *   `m_TimeEventEmittingDirectionX`, `Y`, `Z`: Yayılma yönünün bileşenlerinin (X, Y, Z) zamanla değişimi.
        *   `m_TimeEventEmittingVelocity`: Yayılan parçacıkların başlangıç hızının zamanla değişimi.
        *   `m_TimeEventEmissionCountPerSecond`: Saniye başına yayılan parçacık sayısının (yayma oranının) zamanla değişimi.
        *   `m_TimeEventLifeTime`: Yayılan parçacıkların sahip olacağı başlangıç ömrünün zamanla değişimi.
        *   `m_TimeEventSizeX`, `m_TimeEventSizeY`: Yayılan parçacıkların sahip olacağı başlangıç boyutlarının (X ve Y eksenlerinde) zamanla değişimi.
    *   **Erişim Fonksiyonları:**
        *   Sınıf üyelerinin değerlerini almak için çeşitli `Get*()` metodları (örneğin, `GetMaxEmissionCount()`, `GetEmitterShape()`).
        *   Zamana bağlı özelliklerin belirli bir `fTime` anındaki değerini almak için `Get*(float fTime, float* pfValue)` şeklinde metodlar (örneğin, `GetEmissionCountPerSecond(fTime, &value)`).
    *   **Diğer Metotlar:**
        *   `Clear()`: Yayıcı özelliklerini varsayılan değerlerine sıfırlar.
*   **Kullanım Alanı:** `CParticleSystemData` içinde bir üye olarak (`m_EmitterProperty`) bulunur. `CParticleSystemInstance`, yeni parçacıklar oluştururken bu özellikleri kullanarak parçacıkların nasıl, ne zaman ve hangi başlangıç nitelikleriyle yayılacağını belirler.

### `ParticleProperty.h`

*   **Amaç:** Bir parçacık sistemindeki bireysel bir parçacığın ömrü boyunca sahip olacağı tüm görsel, fiziksel ve davranışsal özellikleri tanımlar.
*   **Bağımlılıklar:** `Type.h` (zamanla değişen değer tabloları için) ve `EterLib/GrpImageInstance.h` (doku yönetimi için).
*   **Temel Özellikler/İçerik (`CParticleProperty` Sınıfı):**
    *   **Görsel Nitelikler:**
        *   **Doku ve Animasyon:**
            *   `m_ImageVector (std::vector<CGraphicImage*>)`: Parçacığın kullanacağı doku(lar)ı tutar. Birden fazla doku, doku animasyonu için kullanılabilir.
            *   `InsertTexture(const char* c_szFileName)` / `SetTexture(const char* c_szFileName)`: Doku ekleme/ayarlama fonksiyonları.
            *   `m_byTexAniType (BYTE)`: Doku animasyon türü (enum: `TEXTURE_ANIMATION_TYPE_NONE`, `CW`, `CCW`, `RANDOM_FRAME`, `RANDOM_DIRECTION`).
            *   `m_fTexAniDelay (float)`: Doku animasyonunda kareler arası gecikme.
            *   `m_bTexAniRandomStartFrameFlag (BOOL)`: Doku animasyonunun rastgele bir kareden başlayıp başlamayacağı.
        *   **Karıştırma (Blending) ve Renk İşlemleri:**
            *   `m_bySrcBlendType (BYTE)`, `m_byDestBlendType (BYTE)`: Parçacığın arka planla nasıl karıştırılacağını belirleyen DirectX karıştırma faktörleri (örn: `D3DBLEND_SRCALPHA`).
            *   `m_byColorOperationType (BYTE)`: Parçacık renginin doku rengiyle nasıl birleştirileceğini belirleyen DirectX renk işlemi (örn: `D3DTOP_MODULATE`).
        *   **Billboard Davranışı:**
            *   `m_byBillboardType (BYTE)`: Parçacığın kameraya göre nasıl hizalanacağını belirler (her zaman kameraya dönük, Y ekseninde dönük vb.).
    *   **Hareket ve Fiziksel Özellikler:**
        *   **Rotasyon:**
            *   `m_byRotationType (BYTE)`: Parçacığın kendi etrafında nasıl döneceğini belirtir (enum: `ROTATION_TYPE_NONE`, `TIME_EVENT`, `CW`, `CCW`, `RANDOM_DIRECTION`).
            *   `m_fRotationSpeed (float)`: Sabit hızlı rotasyon için dönüş hızı.
            *   `m_wRotationRandomStartingBegin`, `m_wRotationRandomStartingEnd (WORD)`: Rastgele başlangıç rotasyonu için açı aralığı (derece cinsinden olabilir).
        *   **Genel Davranış Bayrakları:**
            *   `m_bAttachFlag (BOOL)`: `true` ise parçacık, yayıcının (veya efekt örneğinin) hareketini takip eder.
            *   `m_bStretchFlag (BOOL)`: `true` ise parçacık, hareket yönüne doğru "esner" (uzar), hızla hareket eden efektlere (kıvılcım, iz) dinamik bir görünüm katar.
    *   **Zamana Bağlı Özellikler (`TTimeEventTableFloat`, `TTimeEventTableColor`):**
        *   Bu özellikler, parçacığın ömrü boyunca değerlerinin nasıl değişeceğini tanımlayan zaman çizelgeleridir (`Type.h` içinde tanımlı).
        *   `m_TimeEventGravity (TTimeEventTableFloat)`: Parçacığa etki eden yerçekimi kuvvetinin zamanla değişimi.
        *   `m_TimeEventAirResistance (TTimeEventTableFloat)`: Parçacığa etki eden hava direncinin zamanla değişimi.
        *   `m_TimeEventScaleX`, `m_TimeEventScaleY (TTimeEventTableFloat)`: Parçacığın X ve Y eksenlerindeki boyutunun ömrü boyunca değişimi.
        *   `m_TimeEventColor (TTimeEventTableColor)` (veya World Editor için ayrı R,G,B,A kanalları): Parçacığın renginin (RGBA) ömrü boyunca değişimi.
        *   `m_TimeEventRotation (TTimeEventTableFloat)`: `ROTATION_TYPE_TIME_EVENT` seçiliyse, parçacığın rotasyon açısının ömrü boyunca değişimi.
    *   **Diğer Metotlar:**
        *   `Clear()`: Parçacık özelliklerini varsayılan değerlerine sıfırlar.
*   **Kullanım Alanı:** `CParticleSystemData` içinde bir üye olarak (`m_ParticleProperty`) bulunur. `CParticleSystemInstance` içindeki her bir `CParticleInstance`, bu `CParticleProperty`'den aldığı özelliklere göre canlandırılır ve görselleştirilir.

### `ParticleInstance.h`

*   **Amaç:** Bir parçacık sistemindeki tek bir, bireysel parçacığın oyun dünyasındaki canlı örneğini temsil eder. Kendi pozisyonunu, hızını, rengini, boyutunu, rotasyonunu ve kalan ömrünü yönetir.
*   **Bağımlılıklar:** `EffectUpdateDecorator.h` (parçacığa özel davranışlar eklemek için), `EterLib/GrpBase.h` (temel grafik türleri için), `EterLib/Pool.h` (bellek havuzu için).
*   **Temel Özellikler/İçerik (`CParticleInstance` Sınıfı):**
    *   **Dinamik Durum Bilgileri:**
        *   `m_v3Position (D3DXVECTOR3)`: Parçacığın 3D uzaydaki anlık pozisyonu.
        *   `m_v3LastPosition (D3DXVECTOR3)`: Parçacığın bir önceki frame'deki pozisyonu.
        *   `m_v3Velocity (D3DXVECTOR3)`: Parçacığın anlık hızı (yön ve büyüklük).
        *   `m_fLifeTime (float)`: Parçacığın başlangıçtaki toplam ömrü (saniye).
        *   `m_fLastLifeTime (float)`: Parçacığın kalan ömrü. Bu sıfıra ulaştığında parçacık yok edilir.
    *   **Görsel Nitelikler (Anlık Değerler):**
        *   `m_v2HalfSize (D3DXVECTOR2)`: Parçacık dörtgeninin anlık yarı boyutu (genişlik/2, yükseklik/2).
        *   `m_v2Scale (D3DXVECTOR2)`: Parçacığın anlık X ve Y ölçeği (`CParticleProperty`'deki zaman çizelgesine göre güncellenir).
        *   `m_fRotation (float)`: Parçacığın Z ekseni etrafındaki anlık rotasyon açısı.
        *   `m_dcColor (DWORDCOLOR)` veya `m_Color (D3DXCOLOR)`: Parçacığın anlık RGBA rengi (`CParticleProperty`'deki zaman çizelgesine göre güncellenir).
        *   `m_byFrameIndex (BYTE)`: Doku animasyonu için mevcut kare indeksi.
        *   `m_fLastFrameTime (float)`: Doku animasyonunda kare geçişleri için zamanlayıcı.
    *   **Fiziksel Özellikler (Anlık Değerler):**
        *   `m_fAirResistance (float)`: Parçacığa etki eden anlık hava direnci.
        *   `m_fRotationSpeed (float)`: Parçacığın anlık dönüş hızı.
        *   `m_fGravity (float)`: Parçacığa etki eden anlık yerçekimi kuvveti.
    *   **Güncelleme Mantığı:**
        *   `Update(float fElapsedTime, float fAngle)`: Parçacığın durumunu günceller:
            *   Kalan ömrü (`m_fLastLifeTime`) azaltır.
            *   Eğer bir `m_pDecorator` atanmışsa, dekoratörün `Update()` metodunu çağırarak özel fiziksel davranışları (yerçekimi, hava direnci, özel rotasyon vb.) uygular.
            *   Hıza ve geçen zamana göre `m_v3Position`'ı günceller.
            *   Doku animasyonunu (`m_byFrameIndex`, `m_fLastFrameTime`) ilerletir.
            *   Renk ve ölçeği, `CParticleProperty`'deki zaman çizelgelerine ve geçen ömür yüzdesine göre günceller.
            *   Parçacığın ömrü dolmuşsa `FALSE`, aksi halde `TRUE` döndürür.
    *   **Render için Hazırlık:**
        *   `m_ParticleMesh[4] (TPTVertex)`: Parçacığı temsil eden dörtgenin (quad) köşe (vertex) verilerini (Pozisyon, Texture koordinatları) tutar.
        *   `Transform(const D3DXMATRIX* c_matLocal, ...)`: Parçacığın `m_ParticleMesh` köşe verilerini, parçacığın anlık pozisyonu, boyutu, rotasyonu, billboard tipi ve üst efektin genel matrisi (`c_matLocal`) gibi faktörlere göre dünya koordinatlarına dönüştürür.
        *   `GetParticleMeshPointer()`: Dönüştürülmüş `m_ParticleMesh` verilerine işaretçi döndürür. Bu, `CParticleSystemInstance` tarafından vertex buffer'ı doldurmak için kullanılır.
    *   **Dekoratör Entegrasyonu:**
        *   `m_pDecorator (NEffectUpdateDecorator::CBaseDecorator*)`: Parçacığa özel davranışlar (yerçekimi, hava direnci vb.) ekleyen bir dekoratör nesnesine işaretçi. `CParticleSystemData::BuildDecorator` tarafından oluşturulur ve atanır.
    *   **Özellik İşaretçileri:**
        *   `m_pParticleProperty (CParticleProperty*)`: Bu parçacığın temel özelliklerini tanımlayan `CParticleProperty` nesnesine işaretçi.
        *   `m_pEmitterProperty (CEmitterProperty*)`: Parçacığı yayan yayıcının özelliklerine işaretçi (bazı yayıcı özellikleri parçacığın başlangıç durumunu etkileyebilir).
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CParticleInstance> ms_kPool`: `CParticleInstance` nesneleri için statik bir dinamik bellek havuzu.
        *   `static CParticleInstance* New()`: Havuzdan yeni/kullanılmamış bir `CParticleInstance` alır ve `__Initialize()` ile başlatır.
        *   `DeleteThis()`: Kullanılan `CParticleInstance`'ı havuza geri iade eder.
*   **Kullanım Alanı:** `CParticleSystemInstance` tarafından, yayıcıdan çıkan her bir parçacık için oluşturulur ve yönetilir. Parçacığın ömrü boyunca güncellenir ve render işlemine veri sağlar. Ömrü dolduğunda veya sistem tarafından yok edildiğinde bellek havuzuna geri döner.

### `EffectMesh.h`

*   **Amaç:** Mesh tabanlı efektler için hem 3D model verilerini (`CEffectMesh`) hem de bu modellerin efekt içinde nasıl davranacağını belirleyen script tabanlı özellikleri (`CEffectMeshScript`) tanımlar.
*   **Bağımlılıklar:** `EffectElementBase.h`, `Type.h` ve `EterLib` içindeki `GrpScreen.h`, `Resource.h`, `GrpImageInstance.h`, `TextFileLoader.h` gibi dosyalar.

#### `CEffectMesh` Sınıfı

*   **Rolü:** Bir 3D modelin geometrisini (vertex, index), animasyon karelerini ve ilişkili doku bilgilerini bir dosyadan (genellikle `.mse` uzantılı özel bir format) yükleyen ve yöneten bir kaynak sınıfıdır.
*   **Kalıtım:** `CResource` (Kaynak yönetimi için temel sınıf).
*   **Temel Veri Yapıları:**
    *   `SEffectFrameData (TEffectFrameData)`: Bir mesh'in tek bir animasyon karesine ait vertex, index ve görünürlük verilerini tutar.
        *   `PDTVertexVector (std::vector<TPTVertex>)`: Pozisyon, Diffuse (renk) ve Texture koordinatlarını içeren vertex verileri.
    *   `SEffectMeshData (TEffectMeshData)`: Tek bir mesh nesnesinin tüm verilerini (obje adı, doku adı, tüm animasyon kareleri (`EffectFrameDataVector`), doku işaretçileri (`pImageVector`)) barındırır. Dinamik havuzla yönetilir.
*   **Ana İşlevler:**
    *   `OnLoad(int iSize, const void* c_pvBuf)`: Dosya içeriğinden mesh verilerini yükler. Farklı dosya versiyonlarını (`__LoadData_Ver001`, `__LoadData_Ver002`) destekleyebilir.
    *   `GetFrameCount()`: Mesh animasyonunun toplam kare sayısını döndürür.
    *   `GetMeshCount()`: Yüklenen dosyadaki alt mesh sayısını (geometri sayısını) döndürür.
    *   `GetMeshDataPointer(DWORD dwMeshIndex)`: Belirli bir alt mesh'in `TEffectMeshData` yapısına işaretçi döndürür.
    *   `GetTextureVectorReference(DWORD dwMeshIndex)`: Belirli bir alt mesh'in kullandığı dokuların (`CGraphicImage*`) vektörüne referans döndürür.
*   **Üyeler:**
    *   `m_pEffectMeshDataVector (std::vector<TEffectMeshData*>)`: Yüklenen tüm alt mesh verilerini tutan vektör.

#### `CEffectMeshScript` Sınıfı

*   **Rolü:** `CEffectMesh` ile yüklenecek olan modelin dosya adını ve bu modelin efekt içinde nasıl görüneceğini ve canlandırılacağını (saydamlık, renk, billboard davranışı, doku animasyonu vb.) tanımlayan script tabanlı özellikleri yönetir.
*   **Kalıtım:** `CEffectElementBase` (Efekt elemanları için temel sınıf).
*   **Temel Veri Yapıları:**
    *   `SMeshData (TMeshData)`: `CEffectMesh` içindeki her bir alt mesh için ek görsel ve davranışsal özellikleri tanımlar.
        *   `byBillboardType`: Mesh'in kameraya göre hizalanma türü.
        *   `bBlendingEnable`, `byBlendingSrcType`, `byBlendingDestType`: Saydamlık ve karıştırma ayarları.
        *   `byColorOperationType`, `ColorFactor`: Mesh'e uygulanacak renk işlemi ve sabit renk çarpanı.
        *   `bTextureAnimationLoopEnable`, `fTextureAnimationFrameDelay`, `dwTextureAnimationStartFrame`: Mesh üzerindeki doku animasyonu için parametreler.
        *   `TimeEventAlpha (TTimeEventTableFloat)`: Mesh'in alfasının (görünürlüğünün) zamanla nasıl değişeceğini tanımlayan zaman çizelgesi.
    *   `m_MeshDataVector (TMeshDataVector)`: Her bir alt mesh için bir `TMeshData` nesnesi tutar.
*   **Ana İşlevler ve Üyeler:**
    *   `m_strMeshFileName (std::string)`: Yüklenecek olan `CEffectMesh` dosyasının adı (örn: "effect.mse").
    *   `m_isMeshAnimationLoop (BOOL)`, `m_iMeshAnimationLoopCount (int)`, `m_fMeshAnimationFrameDelay (float)`: `CEffectMesh` içindeki vertex animasyonunun döngü ayarları ve hızı.
    *   `OnLoadScript(CTextFileLoader& rTextFileLoader)`: Efekt script dosyasından (`.eff` uzantılı olabilir) `CEffectMeshScript`'e özgü bu verileri okur.
    *   Çeşitli `Get*()` fonksiyonları ile `TMeshData` içindeki özelliklere (billboard, blending, renk, alfa zaman çizelgesi, doku animasyon parametreleri) erişim sağlar.
*   **Bellek Yönetimi:** `CEffectMeshScript` nesneleri için de dinamik bir bellek havuzu (`ms_kPool`) kullanılır.
*   **Kullanım Alanı:** Bir `CEffectData` içinde `CEffectMeshScript` elemanları tanımlanır. `CEffectManager`, bir `CEffectInstance` oluştururken, bu `CEffectMeshScript` tanımlarına dayanarak ilgili `CEffectMesh` kaynağını yükler (veya mevcutsa onu kullanır) ve ardından bir `CEffectMeshInstance` yaratarak bu mesh'i efektin bir parçası olarak canlandırır.

### `EffectMeshInstance.h`

*   **Amaç:** Bir `CEffectMeshScript` ve dolayısıyla bir `CEffectMesh` verisine dayanarak oyun dünyasında aktif olan ve canlandırılan bir 3D mesh efektini temsil eder. Mesh'in vertex animasyonunu, doku animasyonlarını, görünürlüğünü ve render edilmesini yönetir.
*   **Kalıtım:** `CEffectElementBaseInstance`.
*   **Bağımlılıklar:** `EffectElementBaseInstance.h`, `FrameController.h`, `EffectMesh.h` ve `EterLib` içindeki `GrpScreen.h`, `GrpImageInstance.h`.
*   **Temel Özellikler/İçerik (`CEffectMeshInstance` Sınıfı):**
    *   **Veri Bağlantısı ve Başlatma:**
        *   `OnSetDataPointer(CEffectElementBase* pElement)`: (Korumalı Sanal) Temel alınacak `CEffectMeshScript` nesnesini (`pElement`) ayarlar (`m_pMeshScript`).
            *   `m_pMeshScript->GetMeshFileName()` ile belirtilen `.mse` dosyasını `CEffectMesh` kaynağı olarak yükler (veya alır) ve `m_roMesh`, `m_pEffectMesh` üyelerine atar.
            *   `m_pEffectMesh`'teki her bir alt mesh ve `m_pMeshScript`'teki her bir `TMeshData` için doku örneklerini (`CGraphicImageInstance`) yükler/ayarlar ve bunları `m_TextureInstanceVector` içinde saklar. Her doku seti için bir `TextureFrameController` (`TTextureInstance` içinde) başlatılır.
            *   Mesh'in vertex animasyonu için `m_MeshFrameController`'ı `m_pMeshScript`'teki ayarlara (döngü, hız) göre başlatır.
        *   `m_pMeshScript (CEffectMeshScript*)`: İlişkili mesh script verilerine işaretçi.
        *   `m_pEffectMesh (CEffectMesh*)`: Yüklenen mesh kaynak verilerine işaretçi.
        *   `m_roMesh (CEffectMesh::TRef)`: Mesh kaynağına referans sayımlı işaretçi.
    *   **Animasyon Kontrolü:**
        *   `m_MeshFrameController (CFrameController)`: `CEffectMesh` içindeki vertex animasyonunun (kare geçişlerinin) zamanlamasını ve ilerlemesini yönetir.
        *   `m_TextureInstanceVector (std::vector<TTextureInstance>)`: Her bir alt mesh'in doku animasyonlarını yönetir. `TTextureInstance` yapısı şunları içerir:
            *   `TextureFrameController (CFrameController)`: İlgili alt mesh üzerindeki doku animasyonunun zamanlamasını kontrol eder.
            *   `TextureInstanceVector (std::vector<CGraphicImageInstance*>)`: Doku animasyonu için kullanılacak doku örneklerini tutar.
    *   **Güncelleme ve Render:**
        *   `OnUpdate(float fElapsedTime)`: (Korumalı Sanal) Mesh efekt örneğinin durumunu günceller:
            *   `m_MeshFrameController.Update(fElapsedTime)` ile mesh'in vertex animasyon karesini ilerletir.
            *   `m_TextureInstanceVector` içindeki her bir `TextureFrameController`'ı güncelleyerek doku animasyon karelerini ilerletir.
            *   `m_pMeshScript`'teki `TimeEventAlpha` gibi zamanla değişen özellikleri kullanarak mesh'in anlık saydamlığını vb. günceller.
        *   `OnRender()`: (Korumalı Sanal) Mesh efektini ekrana çizer:
            *   `STATEMANAGER` aracılığıyla gerekli grafik durumlarını (karıştırma modları, Z-test, renk operasyonları) `m_pMeshScript`'teki `TMeshData`'dan alarak ayarlar.
            *   `m_pEffectMesh`'teki her bir alt mesh için:
                *   Doğru doku animasyon karesindeki dokuyu (`m_TextureInstanceVector` ve `TextureFrameController` aracılığıyla) grafik işlem hattına bağlar.
                *   `m_MeshFrameController`'dan alınan mevcut vertex animasyon karesindeki vertex verilerini (`TEffectFrameData::PDTVertexVector`) kullanarak mesh'i render eder.
                *   Gerekirse billboard hesaplamalarını uygular.
        *   `isActive()`: Mesh animasyonunun durumu ve diğer koşullara göre efektin hala aktif olup olmadığını kontrol eder.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CEffectMeshInstance> ms_kPool`: `CEffectMeshInstance` nesneleri için statik bir dinamik bellek havuzu.
*   **Kullanım Alanı:** Bir `CEffectInstance` içinde, `CEffectData`'da tanımlanmış her bir `CEffectMeshScript` için bir `CEffectMeshInstance` oluşturulur. Efektin 3D model tabanlı bileşenlerinin canlandırılması ve görselleştirilmesinden sorumludur.

### `SimpleLightData.h` (CLightData Sınıfı)

*   **Amaç:** Bir efektin parçası olarak kullanılacak basit bir nokta ışığının (point light) tüm statik özelliklerini (renk, maksimum menzil, etki süresi, mesafeye göre azalma faktörleri) ve menzilinin zamanla nasıl değişeceğini tanımlar.
*   **Kalıtım:** `CEffectElementBase`.
*   **Bağımlılıklar:** `EffectElementBase.h`, `Type.h` (özellikle `TTimeEventTableFloat` için) ve `EterLib/TextFileLoader.h`.
*   **Temel Özellikler/İçerik (`CLightData` Sınıfı):**
    *   **Temel Işık Parametreleri:**
        *   `m_fMaxRange (float)`: Işığın ulaşabileceği maksimum etki alanı (yarıçap).
        *   `m_fDuration (float)`: Işığın toplam etki süresi (saniye).
        *   `m_cAmbient (D3DXCOLOR)`: Işığın ortam (ambient) rengi.
        *   `m_cDiffuse (D3DXCOLOR)`: Işığın yaygın (diffuse) rengi.
        *   `m_fAttenuation0 (float)`, `m_fAttenuation1 (float)`, `m_fAttenuation2 (float)`: Işığın mesafeyle nasıl zayıflayacağını belirleyen DirectX azalma faktörleri (sabit, doğrusal, karesel).
    *   **Zamana Bağlı Özellikler:**
        *   `m_TimeEventTableRange (TTimeEventTableFloat)`: Işığın menzilinin (etki alanının) zamanla nasıl değişeceğini tanımlayan bir zaman çizelgesi. Bu, ışığın yanıp sönmesi, parlaklığının artıp azalması gibi etkiler için kullanılır.
        *   `GetRange(float fTime, float& rRange)`: Belirli bir `fTime` anındaki ışık menzilini `m_TimeEventTableRange`'e göre hesaplar.
    *   **Döngü Özellikleri:**
        *   `m_bLoopFlag (BOOL)`: Işık efektinin döngüsel olup olmayacağı.
        *   `m_iLoopCount (int)`: Döngüsel ise tekrar sayısı.
    *   **DirectX Entegrasyonu:**
        *   `InitializeLight(D3DLIGHT8& light)`: Bu `CLightData` nesnesindeki özelliklere göre bir DirectX `D3DLIGHT8` yapısını doldurur. Bu yapı, grafik motoru tarafından ışığı ayarlamak için kullanılır.
    *   **Yükleme ve Temizleme (Devralınan ve Özelleştirilen Fonksiyonlar):**
        *   `OnLoadScript(CTextFileLoader& rTextFileLoader)`: (Korumalı Sanal) Efekt script dosyasından ışığa özgü verileri (renkler, menzil, süre, azalma faktörleri, döngü ayarları, zamanla değişen menzil tablosu) okur.
        *   `OnClear()`: (Korumalı Sanal) Işık özelliklerini varsayılan değerlerine sıfırlar.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CLightData> ms_kPool`: `CLightData` nesneleri için statik bir dinamik bellek havuzu.
*   **Kullanım Alanı:** `CEffectData` içinde bir veya daha fazla `CLightData` tanımlanabilir. `CEffectManager`, bir `CEffectInstance` oluştururken, bu `CLightData` tanımlarına dayanarak `CLightInstance`'lar yaratır ve bu örnekler aracılığıyla oyun dünyasına dinamik ışıklar ekler.

### `SimpleLightInstance.h` (CLightInstance Sınıfı)

*   **Amaç:** Bir `CLightData` şablonuna dayanarak oyun dünyasında aktif olan bir nokta ışığı örneğini temsil eder. Işığın grafik motoruna kaydedilmesi, ömrü boyunca özelliklerinin (özellikle menzilinin) güncellenmesi ve ömrü dolduğunda sistemden kaldırılmasından sorumludur.
*   **Kalıtım:** `CEffectElementBaseInstance`.
*   **Bağımlılıklar:** `EffectElementBaseInstance.h`, `SimpleLightData.h` (içindeki `CLightData`) ve `EterLib/GrpScreen.h` (grafik motoruyla etkileşim için).
*   **Temel Özellikler/İçerik (`CLightInstance` Sınıfı):**
    *   **Veri Bağlantısı ve Başlatma:**
        *   `OnSetDataPointer(CEffectElementBase* pElement)`: (Korumalı Sanal) Temel alınacak `CLightData` nesnesini (`pElement`) ayarlar ve `m_pData` üyesine atar.
        *   `OnInitialize()`: (Korumalı Sanal) Işık örneği başlatıldığında çağrılır.
            *   `m_pData->InitializeLight()` kullanarak bir `D3DLIGHT8` yapısı oluşturur.
            *   Bu `D3DLIGHT8` yapısını `CScreen` (veya benzeri bir grafik motoru arayüzü) aracılığıyla sisteme ekler (örn: `CScreen::AddLight()`).
            *   Grafik motorundan dönen ışık kimliğini (`DWORD`) `m_LightID` içinde saklar.
            *   Döngü sayacını (`m_iLoopCount`) başlatır.
        *   `m_pData (CLightData*)`: İlişkili ışık verilerine işaretçi.
        *   `m_LightID (DWORD)`: Grafik motoru tarafından bu ışık örneğine atanan ve onu yönetmek için kullanılan kimlik.
    *   **Güncelleme ve Yaşam Döngüsü:**
        *   `OnUpdate(float fElapsedTime)`: (Korumalı Sanal) Işık örneğinin durumunu günceller:
            *   Genel efekt elemanı ömrünü (`CEffectElementBaseInstance::m_fRemainingTime`) ilerletir.
            *   `m_pData->GetRange(fElapsedTime, currentRange)` kullanarak mevcut zamana göre ışığın menzilini hesaplar.
            *   `CScreen::SetLightRange(m_LightID, currentRange)` (veya benzeri bir fonksiyonla) grafik motorundaki ışığın menzilini günceller. Gerekirse renk gibi diğer zamanla değişen özellikler de burada güncellenebilir.
            *   Döngü (`m_iLoopCount`) ve genel ömür kontrolü yapar. Efektin süresi dolmuşsa `FALSE` döndürerek kendini sonlandırmaya hazırlar.
        *   `m_iLoopCount (DWORD)`: Mevcut döngü sayısını izler.
    *   **Render:**
        *   `OnRender()`: (Korumalı Sanal) Genellikle boştur. Işıklar doğrudan "çizilmezler"; bunun yerine grafik motoru tarafından sahne aydınlatmasında kullanılırlar. Özellikleri `OnUpdate` sırasında ayarlanır.
    *   **Yok Edilme:**
        *   `OnDestroy()`: (Korumalı Sanal) Işık örneği yok edildiğinde çağrılır. `CScreen::DeleteLight(m_LightID)` (veya benzeri) ile ışığı grafik motorundan kaldırır.
    *   **Bellek Yönetimi (Dinamik Havuz):**
        *   `static CDynamicPool<CLightInstance> ms_kPool`: `CLightInstance` nesneleri için statik bir dinamik bellek havuzu.
*   **Kullanım Alanı:** Bir `CEffectInstance` içinde, `CEffectData`'da tanımlanmış her bir `CLightData` için bir `CLightInstance` oluşturulur. Efektin basit ışıklandırma bileşenlerini oyun dünyasına entegre eder.

### `EffectElementBase.h`

*   **Amaç:** `EffectLib` kütüphanesi içindeki tüm efekt elemanı **veri** sınıfları (örneğin, `CParticleSystemData`, `CEffectMeshScript`, `CLightData`) için ortak bir soyut taban sınıfı tanımlar. Bu sınıf, efekt elemanlarının sahip olabileceği temel özellikleri (başlangıç zamanı, zamanla değişen yerel pozisyon) ve temel işlemleri (veri yükleme, temizleme, veri varlığını kontrol etme) için bir arayüz sağlar.
*   **Bağımlılıklar:** `Type.h` (özellikle `TTimeEventTablePosition` gibi zamanla değişen değer tabloları için).
*   **Temel Özellikler/İçerik (`CEffectElementBase` Sınıfı):**
    *   **Soyut Arayüz (Pure Virtual Fonksiyonlar):**
        *   `virtual void OnClear() = 0`: Türetilmiş sınıfların, kendilerine ait verileri temizlemek için implemente etmesi gereken saf sanal fonksiyon.
        *   `virtual bool OnIsData() = 0`: Türetilmiş sınıfların, geçerli verilerinin yüklenip yüklenmediğini kontrol etmek için implemente etmesi gereken saf sanal fonksiyon.
        *   `virtual BOOL OnLoadScript(CTextFileLoader& rTextFileLoader) = 0`: Türetilmiş sınıfların, bir metin dosyasından (efekt script'i) kendi özel verilerini yüklemek için implemente etmesi gereken saf sanal fonksiyon.
    *   **Genel (Public) Arayüz Fonksiyonları:**
        *   `Clear()`: Dahili olarak `OnClear()`'ı çağırır.
        *   `isData()`: Dahili olarak `OnIsData()`'yı çağırır.
        *   `LoadScript(CTextFileLoader& rTextFileLoader)`: Dahili olarak `OnLoadScript()`'i çağırır. Bu, `CEffectData` gibi üst seviye sınıfların, içerdiği tüm eleman verilerini polimorfik olarak yüklemesini sağlar.
    *   **Ortak Veri Üyeleri ve Fonksiyonları:**
        *   `m_fStartTime (float)`: Bu efekt elemanının, ana efektin (`CEffectInstance`) başlangıcına göre ne kadar süre sonra aktif olacağını belirtir.
        *   `GetStartTime()`: `m_fStartTime` değerini döndürür.
        *   `m_TimeEventTablePosition (TTimeEventTablePosition)`: Efekt elemanının, ana efektin yerel koordinat sistemine göre sahip olacağı pozisyonun zamanla nasıl değişeceğini tanımlayan bir zaman çizelgesi. Bu, elemanın efekt içinde hareket etmesini sağlar.
        *   `GetPosition(float fTime, D3DXVECTOR3& rPosition)`: Belirli bir `fTime` anındaki yerel pozisyonu `m_TimeEventTablePosition`'a göre hesaplar ve `rPosition`'a yazar.
*   **Kullanım Alanı:** Doğrudan örneklenmez. `CParticleSystemData`, `CEffectMeshScript`, `CLightData` gibi sınıflar bu sınıftan türeyerek ortak başlatma zamanı, zamanla değişen pozisyon özelliklerini miras alır ve script'ten veri yükleme, temizleme gibi işlemler için standart bir arayüz sunar. `CEffectData`, bu taban sınıfı işaretçileri üzerinden farklı türdeki eleman verilerini yönetebilir.

### `EffectElementBaseInstance.h`

*   **Amaç:** `EffectLib` kütüphanesi içindeki tüm efekt elemanı **örnek** sınıfları (örneğin, `CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance`) için ortak bir soyut taban sınıfı tanımlar. Bu sınıf, efekt elemanı örneklerinin yaşam döngüsünü (veri atama, başlatma, güncelleme, render etme, yok etme), aktiflik durumunu, yerel zamanını ve ana efekte göre konumlandırılmasını yönetmek için bir arayüz ve temel altyapı sunar.
*   **Bağımlılıklar:** `EffectElementBase.h`.
*   **Temel Özellikler/İçerik (`CEffectElementBaseInstance` Sınıfı):**
    *   **Soyut Arayüz (Pure Virtual Fonksiyonlar):**
        *   `virtual void OnSetDataPointer(CEffectElementBase* pElement) = 0`: Türetilmiş sınıfın, kendisine atanan `CEffectElementBase` verisini (örneğin, `CParticleSystemData*`'ya cast ederek) alıp işlemesi için.
        *   `virtual void OnInitialize() = 0`: Türetilmiş sınıfın, örnek başlatıldığında kendi özel başlatma rutinlerini (kaynak ayırma, ilk durum ayarlama vb.) gerçekleştirmesi için.
        *   `virtual void OnDestroy() = 0`: Türetilmiş sınıfın, örnek yok edildiğinde kendi özel temizleme rutinlerini (kaynakları serbest bırakma vb.) gerçekleştirmesi için.
        *   `virtual bool OnUpdate(float fElapsedTime) = 0`: Türetilmiş sınıfın, her frame'de kendi özel güncelleme mantığını (hareket, animasyon, durum değişikliği vb.) çalıştırması için. Elemanın yaşam süresi dolmuşsa `false` döndürmelidir.
        *   `virtual void OnRender() = 0`: Türetilmiş sınıfın, kendini ekrana çizmek için kendi özel render mantığını çalıştırması için.
    *   **Genel (Public) Yaşam Döngüsü Fonksiyonları:**
        *   `SetDataPointer(CEffectElementBase* pElement)`: Temel veri bloğunu (`m_pBase`) ayarlar ve `OnSetDataPointer()`'ı çağırır.
        *   `Initialize()`: Örneği aktif hale getirir (`m_isActive=true`), zamanlayıcıları sıfırlar ve `OnInitialize()`'ı çağırır.
        *   `Destroy()`: Örneği pasif hale getirir (`m_isActive=false`) ve `OnDestroy()`'ı çağırır.
        *   `Update(float fElapsedTime)`: Elemanın yerel zamanını (`m_fLocalTime`) ilerletir. Başlangıç zamanı (`m_dwStartTime`) geldiyse ve eleman aktifse, `OnUpdate()`'ı çağırır. `OnUpdate` `false` dönerse veya kalan ömür (`m_fRemainingTime`) biterse, bu fonksiyon da `false` döndürür (genellikle elemanın yok edilmesi gerektiğini belirtir).
        *   `Render()`: Eleman aktifse ve başlama zamanı gelmişse, `OnRender()`'ı çağırır.
    *   **Durum ve Zaman Yönetimi:**
        *   `m_isActive (bool)`: Elemanın aktif olup olmadığını belirtir.
        *   `m_fLocalTime (float)`: Elemanın oluşturulmasından itibaren geçen yerel süresi.
        *   `m_dwStartTime (DWORD)`: Elemanın aktifleşeceği, ana efektin başlangıcına göre olan zaman (milisaniye).
        *   `m_fRemainingTime (float)`: Elemanın kalan ömrü. Bu, genellikle `CEffectData`'daki bir süreden veya elemanın kendi iç mantığından gelir.
        *   `m_bStart (bool)`: Elemanın `Update` döngüsüne ilk kez girip girmediğini (başlangıç zamanı koşulunu sağlayıp sağlamadığını) belirtir.
    *   **Konumlandırma:**
        *   `mc_pmatLocal (const D3DXMATRIX*)`: Ana efektin (`CEffectInstance`) dönüşüm matrisine işaretçi. Elemanların bu matrise göre yerel olarak konumlandırılmasına olanak tanır. `SetLocalMatrixPointer()` ile ayarlanır.
    *   **Ölçekleme (Ek Özellikler):**
        *   `SetParticleScale(float fParticleScale)` ve `SetMeshScale(D3DXVECTOR3 rv3MeshScale)`: Parçacık ve mesh örnekleri için özel ölçekleme ayarlama fonksiyonları ve ilgili `m_fParticleScale`, `m_v3MeshScale` üyeleri. Bu, türetilmiş sınıfların özel ölçekleme ihtiyaçlarını karşılamak için eklenmiştir.
*   **Kullanım Alanı:** Doğrudan örneklenmez. `CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance` gibi sınıflar bu sınıftan türeyerek ortak yaşam döngüsü yönetimi, zamanlama ve ana efekte göre konumlandırma altyapısını miras alır. `CEffectInstance`, bu taban sınıfı işaretçileri üzerinden farklı türdeki eleman örneklerini polimorfik olarak yönetir (günceller, render eder).

### `Type.h`

*   **Amaç:** `EffectLib` kütüphanesi genelinde kullanılan temel veri türlerini, enum sabitlerini, vertex formatlarını ve özellikle efekt özelliklerinin zamanla nasıl değişeceğini tanımlayan "Time Event Table" (Zaman Olay Tablosu) yapılarını ve ilgili yardımcı fonksiyonları tanımlar.
*   **Temel İçerikler:**
    *   **Makrolar ve Sabitler:**
        *   `Clamp(x, min, max)`: Bir değeri belirtilen aralıkta sınırlar.
        *   `GRAVITY`: Varsayılan yerçekimi vektörü.
        *   `MAX_FRAME`, `MAX_TEXTURE`: Kullanım amacı tam açık olmayan sabitler.
    *   **Vertex Formatları (FVF - Flexible Vertex Format):**
        *   `FVF_POINT` (`D3DFVF_XYZ`): Sadece pozisyon (X,Y,Z).
        *   `FVF_PT` (`D3DFVF_XYZ|D3DFVF_TEX1`): Pozisyon ve bir doku koordinatı seti (UV).
        *   `FVF_PDT` (`D3DFVF_XYZ|D3DFVF_DIFFUSE|D3DFVF_TEX1`): Pozisyon, diffuse renk (DWORD) ve bir doku koordinatı seti. `TPTVertex` olarak da bilinir.
    *   **Enum Tanımları:**
        *   `EEffectType`: Efekt türlerini listeler (`EFFECT_TYPE_PARTICLE`, `MESH`, `SIMPLE_LIGHT` vb.).
        *   `EMeshBillBoardType`: Mesh efektleri için billboard davranış türleri (`NONE`, `ALL`, `Y`, `MOVE`).
        *   `EBillBoardType`: Parçacıklar/diğer 2D elemanlar için billboard türleri (`NONE`, `ALL`, `Y`, `LIE` (yere yapışık), `2FACE`, `3FACE`).
        *   `EMovingType`: Hareket türleri (`DIRECT`, `BEZIER_CURVE`), genellikle pozisyon zaman tablolarıyla kullanılır.
    *   **`CTimeEvent<T>` Şablon Sınıfı ve `TTimeEventTable*` Türleri:**
        *   `CTimeEvent<T>`: Belirli bir `m_fTime` (zaman) anında bir `m_Value` (değer) tutan temel şablon yapısıdır.
        *   `DWORDCOLOR`: Rengi `DWORD` olarak saklayan ve üzerinde temel işlemler (çarpma, toplama) yapılabilen bir yapı.
        *   `TTimeEventType*` typedef'leri: `CTimeEvent<float>`, `CTimeEvent<DWORDCOLOR>`, `CTimeEvent<D3DXVECTOR2>`, `CTimeEvent<D3DXVECTOR3>` gibi özel `CTimeEvent` türleri tanımlar.
        *   `TTimeEventTable*` typedef'leri: Yukarıdaki `TTimeEventType*` türlerinden oluşan `std::vector`'ler için takma adlardır (örn: `TTimeEventTableFloat`, `TTimeEventTableColor`). Bunlar, bir özelliğin zaman içindeki değişimini tanımlayan anahtar kare (keyframe) listeleridir.
        *   `SEffectPosition (TEffectPosition)`: Zaman, pozisyon, hareket türü ve Bezier kontrol noktası bilgilerini içeren özel bir yapı. `TTimeEventTablePosition` bu yapılardan oluşur.
    *   **Zaman Olayı Değer Hesaplama Fonksiyonları:**
        *   `GetTimeEventBlendValue<T>(float fElapsedTime, std::vector<CTimeEvent<T> >& rVector, T* pReturnValue)`: Verilen bir zaman (`fElapsedTime`) ve bir zaman olay tablosu (`rVector`) için, o andaki değeri anahtar kareler arasında **lineer interpolasyon** yaparak hesaplar. `float`, `D3DXVECTOR2/3` ve `DWORDCOLOR` için özelleşmiş interpolasyon logiği içerir.
        *   `GetTimeEventValue<T>(float fElapsedTime, std::vector<CTimeEvent<T> >& rVector, T* pReturnValue)`: Verilen zamana en yakın (eşit veya küçük) anahtar karenin değerini **interpolasyon yapmadan** (adım fonksiyonu gibi) döndürür.
        *   `GetTimeEventLinearValuePosition(float fElapsedTime, TTimeEventTablePosition& rVector, D3DXVECTOR3* pReturnPos)`: Pozisyonlar için özel bir lineer interpolasyon fonksiyonu (muhtemelen Bezier eğrilerini de destekler).
    *   **Token Okuma Fonksiyonları (Harici Bildirimler):**
        *   `GetTokenTimeEventFloat`, `GetTokenTimeEventColor`, `GetTokenTimeEventPosition` vb.: Efekt script dosyalarından (`CTextFileLoader` aracılığıyla) ilgili zaman olay tablolarını okumak için kullanılan, bu dosyada sadece `extern` olarak bildirilmiş yardımcı fonksiyonlar.
*   **Kullanım Alanı:** Bu dosya, `EffectLib` içindeki birçok sınıfın (özellikle `EmitterProperty`, `ParticleProperty`, `EffectMeshScript`, `CLightData` gibi özellik/veri sınıfları) temelini oluşturur. Efektlerin parçacık boyutu, rengi, pozisyonu, mesh saydamlığı gibi sayısız özelliğinin zamanla dinamik olarak değişmesini sağlayan esnek bir altyapı sunar.

### `FrameController.h`

*   **Amaç:** Kare tabanlı animasyonların (örneğin, mesh vertex animasyonları veya doku (sprite) animasyonları) zamanlamasını, ilerlemesini ve döngü davranışını yönetmek için kullanılan bir yardımcı sınıftır.
*   **Temel Özellikler/İçerik (`CFrameController` Sınıfı):**
    *   **Zamanlama ve Güncelleme:**
        *   `SetFrameTime(float fTime)`: Bir animasyon karesinin ne kadar süre ekranda kalacağını (saniye cinsinden) ayarlar. Bu, animasyon hızını belirler (FPS = 1.0 / fTime).
        *   `Update(float fElapsedTime)`: Kontrolcünün durumunu günceller. Geçen süreyi (`fElapsedTime`) biriktirir. Biriken süre `m_fFrameTime`'ı aştığında, mevcut kareyi (`m_dwcurFrame`) ilerletir ve döngü kontrolü yapar.
        *   `m_fFrameTime (float)`: Bir karenin süresi.
        *   `m_fLastFrameTime (float)`: Son kare güncellemesinden bu yana geçen süreyi biriktiren sayaç.
    *   **Kare Yönetimi:**
        *   `SetMaxFrame(DWORD dwMaxFrame)`: Animasyondaki toplam kare sayısını (0'dan `dwMaxFrame-1`'e kadar) ayarlar.
        *   `SetStartFrame(DWORD dwStartFrame)`: Döngüsel animasyonlarda, son kareye ulaşıldığında geri dönülecek başlangıç karesinin indeksini ayarlar.
        *   `SetCurrentFrame(DWORD dwFrame)`: Mevcut kare indeksini doğrudan ayarlar.
        *   `GetCurrentFrame()`: Mevcut kare indeksini (`BYTE` olarak) döndürür.
        *   `m_dwcurFrame (DWORD)`: Mevcut animasyon karesinin indeksi.
        *   `m_dwMaxFrame (DWORD)`: Animasyondaki toplam kare sayısı.
        *   `m_dwStartFrame (DWORD)`: Döngü için başlangıç karesi indeksi.
    *   **Döngü Kontrolü:**
        *   `SetLoopFlag(BOOL bFlag)`: Animasyonun döngüsel olup olmayacağını (`TRUE` ise döngüsel) ayarlar.
        *   `SetLoopCount(int iLoopCount)`: Animasyonun kaç kez döngü yapacağını ayarlar. Genellikle 0 veya -1 sonsuz döngü anlamına gelir.
        *   `m_isLoop (BOOL)`: Döngü bayrağı.
        *   `m_iLoopCount (int)`: Kalan döngü sayısı.
    *   **Aktiflik Durumu:**
        *   `SetActive(BOOL bFlag)`: Kontrolcünün güncellenip güncellenmeyeceğini ayarlar.
        *   `isActive(DWORD dwMainFrame = 0)`: Kontrolcünün aktif olup olmadığını döndürür.
        *   `m_isActive (BOOL)`: Aktiflik bayrağı.
    *   **Diğer:**
        *   `Clear()`: Kontrolcünün durumunu (mevcut kare, geçen süre, aktiflik vb.) başlangıç değerlerine sıfırlar.
*   **Kullanım Alanı:** `CEffectMeshInstance` içinde hem mesh'in vertex animasyonunu (`m_MeshFrameController`) hem de her bir alt mesh'in doku animasyonunu (`TTextureInstance::TextureFrameController`) kontrol etmek için kullanılır. Aynı zamanda parçacıkların doku animasyonları için de (`CParticleInstance` içinde dolaylı olarak `CParticleSystemInstance` tarafından) kullanılabilir. Animasyonlu bir öğenin hangi karede olduğunun zamanla takip edilmesi gereken her yerde kullanılabilir.

### `EffectUpdateDecorator.h`

*   **Amaç:** Dekoratör (Decorator) tasarım desenini kullanarak `CParticleInstance` nesnelerine dinamik olarak ek davranışlar ve güncelleme mantıkları (yerçekimi, hava direnci, zamanla değer değişimi, doku animasyonu vb.) eklemek için bir çerçeve ve çeşitli somut dekoratör sınıfları tanımlar.
*   **Namespace:** Tüm ilgili sınıflar `NEffectUpdateDecorator` namespace'i içindedir.
*   **Bağımlılıklar:** `Type.h`, `EterBase/Random.h`, `EterLib/Pool.h`.
*   **Temel Bileşenler:**
    *   **`CDecoratorData` Yapısı:** Dekoratörlerin `Excute` metoduna geçen verileri (parçacık örneği, geçen süre, toplam süre) paketler.
    *   **`CBaseDecorator` Soyut Sınıfı:**
        *   Tüm dekoratörlerin temel sınıfıdır.
        *   **Zincirleme:** Dekoratörlerin bir bağlı liste (`m_NextDecorator`, `m_PrevDecorator`) oluşturarak birbirine eklenmesine olanak tanır.
        *   `Excute(const CDecoratorData& d)`: Zincirdeki tüm dekoratörlerin `__Excute()` metodunu sırayla çağırır.
        *   `Clone(...)`: Bir dekoratör zincirinin tamamını yeni bir parçacık örneği için kopyalar. İşaretçilerin yeni örneğe göre ayarlanmasını sağlar.
        *   `__Excute(const CDecoratorData& d)` (Pure Virtual): Türetilmiş sınıfın özel güncelleme mantığını uyguladığı yer.
        *   `__Clone(...)` (Pure Virtual): Türetilmiş sınıfın kendi kopyasını oluşturduğu yer.
    *   **`CTimeEventDecorator<T>` Şablon Sınıfı:**
        *   Bir `TTimeEventTable*` kullanarak bir `CParticleInstance` üye değişkeninin (`T* pData`) değerini zamanla lineer olarak interpolasyonla günceller.
        *   Çeşitli özellikler için typedef'ler içerir: `CScaleValueDecorator`, `CColorAllDecorator`, `CAirResistanceValueDecorator`, `CGravityValueDecorator`, `CRotationSpeedValueDecorator`.
    *   **Doku Animasyon Dekoratörleri:**
        *   `CTextureAnimationCWDecorator`: Doku animasyonunu saat yönünde (`CW`) ilerletir.
        *   `CTextureAnimationCCWDecorator`: Doku animasyonunu saat yönünün tersine (`CCW`) ilerletir.
        *   `CTextureAnimationRandomDecorator`: Doku animasyonunu rastgele karelerle ilerletir.
        *   Bu dekoratörler, `CParticleInstance`'ın `m_byFrameIndex` gibi üyelerini doğrudan manipüle eder.
    *   **Fiziksel Davranış Dekoratörleri:**
        *   `CAirResistanceDecorator`: Parçacığın hızını (`m_v3Velocity`), parçacığın kendi `m_fAirResistance` değerine göre azaltır.
        *   `CGravityDecorator`: Parçacığın hızına (`m_v3Velocity`), parçacığın kendi `m_fGravity` değerine göre yerçekimi uygular.
        *   `CRotationDecorator`: Parçacığın rotasyonunu (`m_fRotation`), parçacığın kendi `m_fRotationSpeed` değerine göre günceller.
*   **Bellek Yönetimi:** Birçok dekoratör sınıfı `CPooledObject<T>`'den türetilmiştir ve kendi bellek havuzları üzerinden yönetilir.
*   **Kullanım Alanı:** `CParticleSystemData::BuildDecorator` fonksiyonu, efekt script'inde tanımlanan özelliklere göre (örneğin, zamanla değişen alfa, yerçekimi etkisi, doku animasyonu) uygun dekoratör nesnelerini oluşturur ve bunları bir zincir halinde birbirine bağlar. Bu zincirin başlangıcı (`CHeaderDecorator`) `CParticleInstance`'ın `m_pDecorator` üyesine atanır. Her `CParticleInstance::Update` çağrısında, bu zincirdeki tüm dekoratörler çalıştırılarak parçacığın durumu güncellenir.

(Bu bölüme, EffectLib içindeki belirli dosyaların detaylı açıklamaları eklenecektir.) 

---

**Not:** `EffectLib` içerisindeki sınıflara ait `.cpp` implementasyon detayları için lütfen [`client_EffectLib_Referans_Part2.md`](client_EffectLib_Referans_Part2.md) dosyasına bakınız.

### `StdAfx.h`

*   **Amaç:** `EffectLib` projesi için ön derlenmiş başlık (precompiled header - PCH) dosyasıdır. Sık kullanılan ve nadiren değiştirilen başlık dosyalarını (örneğin, Windows API, temel kütüphane başlıkları) içererek derleme sürelerini kısaltmayı amaçlar.
*   **İçerik:**
    *   `#pragma once`: Bu başlık dosyasının yalnızca bir kez dahil edilmesini sağlar.
    *   Temel Kütüphane Dahili Başlıkları:
        *   `../EterBase/StdAfx.h`: `EterBase` kütüphanesinin ön derlenmiş başlığını içerir. Bu sayede `EterBase`'in temel tanımları ve sık kullanılan başlıkları da `EffectLib` için PCH'a dahil edilir.
        *   `../EterBase/Utils.h`, `../EterBase/Timer.h`, `../EterBase/CRC32.h`, `../EterBase/Debug.h`: `EterBase` kütüphanesinden çeşitli yardımcı programlar, zamanlayıcı, CRC32 hesaplama ve hata ayıklama araçları gibi sık kullanılan modülleri doğrudan dahil eder.
        *   `../EterLib/StdAfx.h`: `EterLib` kütüphanesinin ön derlenmiş başlığını içerir.
        *   `../EterLib/TextFileLoader.h`: Metin dosyalarını yüklemek için kullanılan `CTextFileLoader` sınıfını içerir. Bu, efekt script dosyalarının okunması için gereklidir.
        *   `../MilesLib/StdAfx.h`: Ses kütüphanesi olan `MilesLib`'in ön derlenmiş başlığını içerir.
    *   Yorum Satırları: Dosyanın sonunda, `EffectLib` içindeki neredeyse tüm diğer başlık dosyalarını içeren yorum satırına alınmış bir bölüm bulunur. Bu, geliştirme sırasında PCH'a nelerin dahil edilip edilmeyeceğinin denendiğini veya bir zamanlar tüm projenin PCH üzerinden derlendiğini, ancak daha sonra modülerliği artırmak için bu yaklaşımın değiştirilmiş olabileceğini gösterir. Mevcut durumda, sadece dış kütüphanelerin `StdAfx.h` dosyaları ve bazı temel modüller doğrudan PCH'a dahil edilmiştir.
*   **Kullanım Alanı:**
    *   **Derleme Süresini İyileştirme:** `StdAfx.h` içinde listelenen başlık dosyaları, proje ilk kez derlendiğinde veya `StdAfx.cpp` değiştirildiğinde bir `.pch` dosyasına önceden derlenir. Sonraki derlemelerde, bu başlıklar tekrar işlenmek yerine önceden derlenmiş bu dosya kullanılır, bu da özellikle büyük projelerde derleme süresini önemli ölçüde azaltır.
    *   **Proje Bağımlılıkları:** `EffectLib`'in hangi temel kütüphanelere (EterBase, EterLib, MilesLib) ve bu kütüphanelerin hangi temel bileşenlerine doğrudan erişimi olduğunu gösterir.

(Bu bölüme, EffectLib içindeki belirli dosyaların detaylı açıklamaları eklenecektir.)