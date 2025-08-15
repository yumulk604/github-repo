# EffectLib Referans Kılavuzu - Bölüm 3: Ek C++ Implementasyon Detayları

Bu belge, `EffectLib` modülündeki bazı önemli sınıfların `.cpp` dosyalarında bulunan implementasyon detaylarına devam etmektedir.

## `EffectManager.cpp` Implementasyon Detayları

`EffectManager.cpp`, `CEffectManager` sınıfının metodlarını uygulayarak efekt sisteminin merkezi yönetimini hayata geçirir.

*   **Bilgi Sağlama (`GetInfo`)**:
    *   Oyun içi hata ayıklama veya performans izleme amacıyla çeşitli efekt verisi ve örnek havuzlarının (`ms_kPool` kapasiteleri) ve aktif efekt/veri haritalarının boyutları hakkında bilgi içeren bir string oluşturur.

*   **Ses Güncelleme (`UpdateSound`)**:
    *   Aktif tüm efekt örnekleri (`m_kEftInstMap`) üzerinde gezer ve her birinin `UpdateSound()` metodunu çağırır. Bu, efektlerle senkronize ses olaylarının (başlama, bitme, döngü) yönetilmesini sağlar.

*   **Efekt Yaşam Kontrolü (`IsAliveEffect`)**:
    *   Verilen `dwInstanceIndex` ile bir efekt örneğinin `m_kEftInstMap` içinde var olup olmadığını ve `isAlive()` metodunun `true` döndürüp döndürmediğini kontrol eder.

*   **Genel Güncelleme (`Update`)**:
    *   `m_kEftInstMap` içindeki tüm aktif efekt örnekleri üzerinde döngüyle gezer.
    *   Her bir `CEffectInstance`'ın `Update()` metodunu çağırarak bireysel durumlarını (parçacık pozisyonları, animasyon kareleri, ömürleri vb.) ilerletir.
    *   Eğer bir `CEffectInstance`'ın `Update()` metodu (veya `isAlive()` kontrolü) artık aktif olmadığını belirtirse (örneğin, ömrü dolmuşsa), bu örnek `m_kEftInstMap`'ten silinir ve `CEffectInstance::Delete()` çağrılarak bellek havuzuna iade edilir.

*   **Render (`Render`)**:
    *   Render işleminden önce temel doku durumlarını sıfırlar (`STATEMANAGER.SetTexture(0, NULL)`).
    *   İki render modunu destekler:
        *   **Sıralı Render (Varsayılan, `!m_isDisableSortRendering`)**:
            *   Tüm aktif efekt örneklerini geçici bir `std::vector<CEffectInstance*>` (`s_kVct_pkEftInstSort`) içine kopyalar.
            *   Bu vektörü, `CEffectManager_LessEffectInstancePtrRenderOrder` funktörünü (bu da `CEffectInstance::LessRenderOrder` metodunu çağırır) kullanarak `std::sort` ile sıralar. Bu sıralama genellikle saydam efektlerin doğru çizilmesi (arkadan öne) için gereklidir.
            *   Sıralanmış vektördeki her bir efekt örneği için `CEffectManager_FEffectInstanceRender` funktörünü (`std::for_each` ile) çağırarak `Render()` metodunu tetikler.
        *   **Sırasız Render (`m_isDisableSortRendering` true ise)**:
            *   Doğrudan `m_kEftInstMap` üzerinde gezer ve her efekt örneğinin `Render()` metodunu çağırır. Bu, sıralama adımını atladığı için daha hızlı olabilir ancak saydamlık sorunlarına yol açabilir.

*   **Efekt Verisi Kaydı (`RegisterEffect`, `RegisterEffect2`)**:
    *   Verilen dosya adının (`c_szFileName`) küçük harfe dönüştürülmüş CRC32 hash'ini hesaplar (`dwCRC`).
    *   `m_kEftDataMap` içinde bu CRC ile bir `CEffectData` olup olmadığını kontrol eder.
        *   Eğer varsa ve `isExistDelete` `true` ise, mevcut `CEffectData`'yı siler ve haritadan kaldırır.
        *   Eğer varsa ve `isExistDelete` `false` ise (veya `RegisterEffect2` çağrılmışsa), efektin zaten kayıtlı olduğunu varsayarak `true` döndürür.
    *   Eğer CRC ile kayıtlı veri yoksa veya silinmişse:
        *   `CEffectData::New()` ile havuzdan yeni bir `CEffectData` nesnesi alır.
        *   `pkEftData->LoadScript(c_szFileName)` ile efekt script dosyasını yükler. Yükleme başarısız olursa hata verir, nesneyi havuza iade eder ve `false` döndürür.
        *   Yüklenen `CEffectData` nesnesini `dwCRC` anahtarıyla `m_kEftDataMap`'e ekler.
        *   Eğer `isNeedCache` `true` ise ve `m_kEftCacheMap`'te bu CRC ile bir örnek yoksa, `CEffectInstance::New()` ile bir örnek oluşturur, `SetEffectDataPointer()` ile veriyi bağlar ve bu örneği `m_kEftCacheMap`'e ekler. Bu, sık kullanılan efektlerin bir örneğini önbellekte tutarak yeniden oluşturma maliyetini azaltır.
    *   `RegisterEffect2`, `RegisterEffect`'i `isExistDelete = false` ile çağırır ve hesaplanan CRC'yi de döndürür.

*   **Efekt Örneği Oluşturma (`CreateEffect`, `CreateEffectInstance`)**:
    *   `CreateEffect(const char* c_szFileName, ...)`: Dosya adının CRC'sini hesaplar ve `CreateEffect(DWORD dwID, ...)` fonksiyonunu çağırır.
    *   `CreateEffect(DWORD dwID, ...)`:
        *   `GetEmptyIndex()` ile `m_kEftInstMap` için benzersiz bir örnek indeksi (`iInstanceIndex`) alır.
        *   `CreateEffectInstance(iInstanceIndex, dwID, fParticleScale, c_pv3MeshScale)` fonksiyonunu çağırarak asıl örneği oluşturur.
        *   Oluşturulan örneği `SelectEffectInstance(iInstanceIndex)` ile seçili örnek yapar.
        *   Verilen pozisyon (`c_rv3Position`) ve rotasyon (`c_rv3Rotation`) değerlerinden bir dünya matrisi oluşturur ve `SetEffectInstanceGlobalMatrix()` ile bu matrisi seçili örneğe atar.
        *   Örnek indeksini (`iInstanceIndex`) döndürür.
    *   `CreateEffectInstance(DWORD dwInstanceIndex, DWORD dwID, ...)`:
        *   Verilen `dwID` ile `GetEffectData()` kullanarak `CEffectData` işaretçisini alır. Veri bulunamazsa hata verir ve çıkar.
        *   `CEffectInstance::New()` ile havuzdan yeni bir `CEffectInstance` alır.
        *   Parçacık ve mesh ölçeklerini ayarlar.
        *   `pEffectInstance->SetEffectDataPointer(pEffect)` çağrısı ile efekt verisini örneğe bağlar. Bu adımda, `CEffectInstance` kendi içindeki parçacık, mesh ve ışık örneklerini oluşturur.
        *   Oluşturulan `CEffectInstance`'ı `dwInstanceIndex` anahtarıyla `m_kEftInstMap`'e ekler.

*   **Efekt Örneği Yok Etme (`DestroyEffectInstance`)**:
    *   Verilen `dwInstanceIndex` ile `m_kEftInstMap`'te örneği bulur.
    *   Bulunamazsa `false` döndürür.
    *   Bulunursa, örneği haritadan siler ve `CEffectInstance::Delete()` çağrısı ile havuzuna iade eder.

*   **Efekt Örneği Aktifleştirme/Deaktifleştirme (`ActiveEffectInstance`, `DeactiveEffectInstance`)**:
    *   Verilen `dwInstanceIndex` ile `m_kEftInstMap`'te örneği bulur.
    *   Bulunursa, örneğin `SetActive(true)` veya `SetDeactive(true)` metodunu çağırır. Bu fonksiyonlar, özellikle `ENABLE_GRAPHIC_ON_OFF` gibi makrolar tanımlı olduğunda derlenir ve oyun içi grafik ayarlarıyla efektlerin topluca açılıp kapatılmasına olanak tanır.

*   **Güvensiz Efekt Örneği Oluşturma (`CreateUnsafeEffectInstance`)**:
    *   Bu metod, `CEffectData` işaretçisini dışarıdan alır ve bu veriyi kullanarak bir `CEffectInstance` oluşturur, ancak bu örneği `m_kEftInstMap`'e **eklemez**. Döndürülen `CEffectInstance** ppEffectInstance` işaretçisi üzerinden örnek yönetimi çağıran koda bırakılır. Bu, muhtemelen özel durumlar veya yöneticinin standart yaşam döngüsü dışında yönetilmesi gereken efektler için kullanılır.

*   **Efekt Örneği Seçimi ve Manipülasyonu (`SelectEffectInstance`, `SetEffectTextures`, vb.)**:
    *   `SelectEffectInstance(DWORD dwInstanceIndex)`: `m_kEftInstMap`'ten örneği bulur ve `m_pSelectedEffectInstance` işaretçisine atar. Sonraki `SetEffectInstancePosition` gibi fonksiyonlar bu seçili örnek üzerinde çalışır.
    *   `SetEffectTextures(DWORD dwID, std::vector<std::string> textures)`: Verilen `dwID` ile `CEffectData`'yı bulur ve bu verinin içindeki mesh veya parçacık elemanlarının dokularını `textures` vektöründeki dosya adlarıyla değiştirmeye çalışır. Bu, aynı efekt yapısını farklı görünümlerle kullanmak için çalışma zamanında doku değişikliğine izin verir. `CEffectData::ChangeTexture` veya benzeri bir alt fonksiyonu çağırır.

*   **Efekt Verisi Alma (`GetEffectData`)**:
    *   Verilen `dwID` (CRC) ile `m_kEftDataMap`'ten `CEffectData` işaretçisini bulur ve çıktı parametresi `ppEffect`'e atar.

*   **Temizleme ve Yok Etme (`__DestroyEffectDataMap`, `__DestroyEffectCacheMap`, `__DestroyEffectInstanceMap`, `DeleteAllInstances`, `Destroy`)**:
    *   Bu fonksiyonlar, yöneticinin sahip olduğu veri ve örnek haritalarını temizler. Elemanları kendi `Delete` fonksiyonlarını çağırarak havuzlarına iade ederler. `Destroy`, tüm bu temizleme işlemlerini sırayla yapar.

*   **Boş İndeks Bulma (`GetEmptyIndex`)**:
    *   `m_kEftInstMap` için kullanılmayan bir sonraki uygun anahtarı (indeks) üretir. `m_dwEmptyInstanceIndex` sayacını artırarak basit bir şekilde çalışır.

**Oyun İçi Kullanım Alanları:**

`CEffectManager`, oyun istemcisindeki tüm görsel efektlerin beyni ve merkezi kontrol noktasıdır. Oyundaki kullanım alanları şunlardır:

1.  **Efekt Yükleme ve Kaydı:** Oyun başladığında veya belirli bir seviye/harita yüklendiğinde, gerekli tüm efekt scriptleri (`.eff` veya benzeri uzantılı dosyalar) `RegisterEffect` aracılığıyla okunur, `CEffectData` nesneleri olarak işlenir ve `CEffectManager` içinde saklanır. Bu, "Yıldırım Büyüsü", "Patlama Efekti", "Işınlanma Parıltısı" gibi tüm efekt şablonlarının hazır olmasını sağlar.
2.  **Efekt Oluşturma:** Bir oyuncu yetenek kullandığında, bir canavar öldüğünde, bir eşya parladığında veya herhangi bir görsel geri bildirim gerektiğinde, oyun mantığı `CEffectManager::Instance().CreateEffect(...)` fonksiyonunu çağırır. Bu fonksiyon, ilgili `CEffectData` şablonunu kullanarak oyun dünyasında belirli bir pozisyonda ve rotasyonda yeni bir `CEffectInstance` yaratır.
3.  **Yaşam Döngüsü Yönetimi:** `CEffectManager::Update()` fonksiyonu, oyunun ana döngüsünün her bir adımında çağrılır. Bu, tüm aktif `CEffectInstance`'ların güncellenmesini, yani parçacıkların hareket etmesini, animasyonların ilerlemesini ve ömürlerinin azalmasını sağlar. Ömrü dolan efektler otomatik olarak temizlenir.
4.  **Render Yönetimi:** `CEffectManager::Render()` fonksiyonu, oyunun render döngüsünde çağrılır. Tüm aktif ve görünür `CEffectInstance`'ların ekrana çizilmesini sağlar. Saydamlık gibi görsel kaliteyi artırmak için efektleri sıralayabilir.
5.  **Efekt Önbellekleme:** Sık kullanılan veya anında tepki vermesi gereken efektler (`isNeedCache` ile işaretlenmiş olanlar), birer örnekleri `m_kEftCacheMap` içinde önbelleğe alınabilir. Bu, `CreateEffect` çağrıldığında doğrudan bu önbellekten bir kopya alınmasını sağlayarak (veya önbellekteki veriyi kullanarak yeni bir örnek oluşturarak) performansı artırabilir.
6.  **Dinamik Kontrol:** Oyun, seçili efektler üzerinde `SelectEffectInstance` ve ardından `SetEffectInstancePosition`, `SetEffectInstanceRotation` gibi fonksiyonlarla dinamik kontrol sağlayabilir. Bu, örneğin bir karaktere bağlı kalan bir aura efektinin karakterle birlikte hareket etmesi veya bir hedefe kilitlenen bir büyü efektinin hedefini takip etmesi gibi senaryolarda kullanılabilir.
7.  **Grafik Ayarları Entegrasyonu:** `ActiveEffectInstance` ve `DeactiveEffectInstance` gibi fonksiyonlar, oyunun grafik ayarlarıyla entegre olabilir. Örneğin, "Efektleri Göster/Gizle" gibi bir seçenek, bu fonksiyonlar aracılığıyla tüm efekt örneklerini topluca etkinleştirebilir veya devre dışı bırakabilir.
8.  **Hata Ayıklama ve İzleme:** `GetInfo` metodu, geliştiricilere aktif efekt sayısı, bellek kullanımı gibi konularda bilgi sağlayarak hata ayıklama ve performans optimizasyonu süreçlerine yardımcı olur.

Kısacası, `CEffectManager`, oyun dünyasını görsel olarak zenginleştiren ve oyuncuya geri bildirim sağlayan tüm dinamik görsel öğelerin verimli bir şekilde yönetilmesinden sorumludur.

## `EffectData.cpp` Implementasyon Detayları

`EffectData.cpp`, `CEffectData` sınıfının metodlarını uygulayarak bir efekt şablonunun nasıl yüklendiğini, saklandığını ve yönetildiğini tanımlar.

*   **Bellek Yönetimi (`New`, `Delete`, `DestroySystem` - Statik Metotlar)**:
    *   `New()`: `CEffectData::ms_kPool` (statik `CDynamicPool`) üzerinden yeni bir `CEffectData` nesnesi ayırır.
    *   `Delete(CEffectData* pkData)`: Verilen `CEffectData` nesnesinin `Clear()` metodunu çağırarak içindeki tüm bileşen verilerini temizler (ve bileşenlerin kendi havuzlarına iade edilmesini sağlar), ardından nesneyi `ms_kPool`'a geri verir.
    *   `DestroySystem()`: `CEffectData`'nın kendi bellek havuzunu (`ms_kPool.Destroy()`) ve ayrıca bağımlı olduğu tüm temel efekt elemanı veri sınıflarının (`CParticleSystemData`, `CEffectMeshScript`, `CLightData`) statik `DestroySystem()` metotlarını çağırarak onların da havuzlarını temizler. Bu, program sonlanırken tüm efekt sistemi kaynaklarının düzgün bir şekilde serbest bırakılmasını sağlar.

*   **Script Yükleme (`LoadScript`)**:
    *   `m_strFileName` üyesine dosya adını atar ve `CFileNameHelper::StringPath` ile yolu normalleştirir.
    *   Bir `CTextFileLoader` nesnesi oluşturur ve verilen `c_szFileName` dosyasını yükler. Yükleme başarısız olursa `false` döndürür.
    *   Script dosyasından önce "boundingsphereradius" (float) ve "boundingsphereposition" (D3DXVECTOR3) token'larını okumaya çalışır. Bulunamazlarsa varsayılan (0) değerlere ayarlanırlar.
    *   `TextFileLoader.GetChildNodeCount()` ile script dosyasındaki ana düğüm (efekt elemanı) sayısını alır ve her biri için döngüye girer:
        *   `TextFileLoader.SetChildNode(i)` ile i. çocuğa odaklanır.
        *   Düğümün adını (`strName`) alır.
        *   Eğer düğüm adı "mesh" ise:
            *   `AllocMesh()` (aşağıda açıklanmıştır) ile bir `CEffectMeshScript` nesnesi alır.
            *   Nesnenin `Clear()` metodunu çağırarak önceki verilerden arındırır.
            *   `pMesh->LoadScript(TextFileLoader)` çağrısı ile, script dosyasının o bölümünün işlenmesini `CEffectMeshScript` sınıfına devreder.
        *   Eğer düğüm adı "particle" ise:
            *   `AllocParticle()` ile bir `CParticleSystemData` nesnesi alır.
            *   `Clear()` ve ardından `pParticleSystemData->LoadScript(TextFileLoader)` çağrıları yapılır.
        *   Eğer düğüm adı "light" ise:
            *   `AllocLight()` ile bir `CLightData` nesnesi alır.
            *   `Clear()` ve ardından `pLightData->LoadScript(TextFileLoader)` çağrıları yapılır.
        *   Her eleman işlendikten sonra `TextFileLoader.SetParentNode()` ile bir üst seviyeye dönülür.
    *   **Ses Yükleme**:
        *   Efekt dosyasının adından (`m_strFileName`) yola çıkarak bir ses script dosyası adı türetmeye çalışır. Örneğin, "d:/ymir work/effect/fire.eff" ise "sound/effect/fire.mss" gibi bir dosya arar. (`CFileNameHelper::NoExtension` ve string manipülasyonları kullanılır).
        *   Eğer geçerli bir ses dosyası adı türetilebilirse, `LoadSoundScriptData()` fonksiyonunu bu adla çağırır.

*   **Ses Scripti Yükleme (`LoadSoundScriptData`)**:
    *   `NSound::TSoundDataVector SoundDataVector`: Geçici bir ses verisi vektörü.
    *   `NSound::LoadSoundInformationPiece(c_szFileName, SoundDataVector)`: Harici ses kütüphanesi (`MilesLib` veya benzeri bir arayüz üzerinden) fonksiyonunu çağırarak verilen `.mss` (Metin2 Sound Script) dosyasından ses bilgilerini okur ve `SoundDataVector`'ü doldurur.
    *   `NSound::DataToInstance(SoundDataVector, &m_SoundInstanceVector)`: Okunan ham ses verilerini (`SoundDataVector`), efekt örneği tarafından kullanılabilecek ses örneği bilgilerine (`m_SoundInstanceVector`) dönüştürür.
    *   Not: `LoadSoundInformationPiece` `true` (başarılı) dönerse bu fonksiyon `false` döndürüyor, bu bir mantık hatası veya özel bir durum olabilir. Genellikle başarılı yüklemede `true` dönmesi beklenir.

*   **Efekt Elemanı Tahsis Metotları (`AllocParticle`, `AllocMesh`, `AllocLight`)**:
    *   Bu sanal fonksiyonlar (`CEffectData.h` içinde `virtual` olarak tanımlanmıştır), ilgili efekt elemanı veri sınıfının statik `New()` metodunu çağırarak havuzdan bir nesne alır (örn: `CParticleSystemData::New()`).
    *   Alınan nesneyi `CEffectData`'nın kendi içindeki ilgili vektöre (`m_ParticleVector`, `m_MeshVector`, `m_LightVector`) ekler.
    *   Alınan nesne işaretçisini döndürür. Bu, `LoadScript` içinde kullanılır.

*   **Erişimci Metotlar (`GetLightCount`, `GetLightPointer`, `GetParticleCount`, vb.)**:
    *   İlgili iç vektörlerin boyutlarını veya belirli bir indeksteki eleman işaretçilerini döndüren basit getter fonksiyonlarıdır. Sınırlayıcı küre bilgilerini (`GetBoundingSphereRadius`, `GetBoundingSpherePosition`) ve dosya adını (`GetFileName`) almak için de metotlar bulunur.

*   **Temizleme Metotları (`__ClearParticleDataVector`, `__ClearLightDataVector`, `__ClearMeshDataVector`, `Clear`)**:
    *   `__Clear*Vector()`: İlgili vektördeki tüm elemanlar için kendi `Delete()` (havuza iade) metotlarını çağırır (`std::for_each` kullanarak) ve ardından vektörü temizler (`.clear()`).
    *   `Clear()`: Sınırlayıcı küre bilgilerini varsayılan değerlere sıfırlar ve yukarıdaki üç `__Clear*Vector()` metodunu çağırarak tüm bileşen verilerini temizler.

*   **Kurucu (`CEffectData()`)**:
    *   Sınırlayıcı küre üyelerini (`m_fBoundingSphereRadius`, `m_v3BoundingSpherePosition`) varsayılan olarak sıfırlar. Diğer vektörler `std::vector` oldukları için otomatik olarak boş başlatılır.

**Oyun İçi Kullanım Alanları:**

`CEffectData` ve onun `.cpp` implementasyonu, oyun istemcisindeki görsel efektlerin "şablonlarını" veya "tasarımlarını" tanımlar ve yönetir.

1.  **Efekt Tanımlama:** Oyun tasarımcıları veya sanatçıları, metin tabanlı script dosyaları (`.eff` uzantılı olabilir) oluşturarak bir efektin nasıl görüneceğini ve davranacağını tanımlar. Bu scriptler, efektin hangi parçacık sistemlerini, 3D meshleri veya ışıkları içereceğini, bu bileşenlerin özelliklerini (renk, boyut, hız, ömür, doku, animasyon vb.) ve zamanla nasıl değişeceğini belirtir.
2.  **Kaynak Yükleme:** Oyun başladığında veya bir bölgeye girildiğinde, `CEffectManager`, bu efekt script dosyalarını `CEffectData::LoadScript` metodunu kullanarak yükler. `LoadScript`, script dosyasını satır satır ayrıştırır:
    *   Efektin genel sınırlayıcı küre bilgilerini okur.
    *   "particle", "mesh", "light" gibi anahtar kelimelerle tanımlanmış her bir efekt bileşeni için, ilgili veri sınıfından (`CParticleSystemData`, `CEffectMeshScript`, `CLightData`) bir nesne oluşturur (`Alloc*` metodları ile).
    *   Oluşturulan bu bileşen nesnesinin kendi `LoadScript` metodunu çağırarak, script'in o bölümünün detaylarını işlemesini sağlar. Bu, modüler bir yükleme süreci oluşturur.
    *   Ayrıca, efektle ilişkili ses tanımlarını içeren ayrı bir `.mss` dosyasını da yükleyebilir.
3.  **Veri Saklama:** Yüklenen ve işlenen bu `CEffectData` nesneleri, `CEffectManager` tarafından bir harita içinde (genellikle dosya adı CRC'si ile anahtarlanmış) saklanır. Bu sayede, aynı efekt tekrar tekrar kullanılacağında script dosyasının yeniden yüklenmesine gerek kalmaz.
4.  **Efekt Örneği Oluşturma İçin Şablon:** Bir efektin oyun dünyasında gösterilmesi gerektiğinde (`CEffectManager::CreateEffect` çağrıldığında), yönetici ilgili `CEffectData` nesnesini bulur. Bu `CEffectData` nesnesi, yeni oluşturulacak `CEffectInstance` için bir şablon görevi görür. `CEffectInstance`, bu şablondaki bilgilere (parçacık tanımları, mesh tanımları vb.) bakarak kendi aktif bileşenlerini (örneğin, `CParticleSystemInstance`'lar) oluşturur.
5.  **Bellek Verimliliği:** `CDynamicPool` kullanımı, sık oluşturulan ve yok edilen `CEffectData` nesneleri (ve bunların bileşenleri olan `CParticleSystemData` vb.) için bellek ayırma ve serbest bırakma işlemlerinin maliyetini azaltır. Bu, özellikle çok sayıda farklı efektin olduğu veya efektlerin sıkça tetiklendiği oyunlarda performansı artırır.
6.  **Modülerlik ve Genişletilebilirlik:** `AllocParticle`, `AllocMesh`, `AllocLight` gibi sanal fonksiyonlar, gelecekte farklı türde efekt bileşenleri eklemek (örneğin, özel bir "Şerit Efekti Verisi") için bir altyapı sunar. `CEffectData`'dan türetilmiş yeni bir sınıf, bu `Alloc*` fonksiyonlarını override ederek kendi özel veri türlerini oluşturabilir.

Kısacası, `EffectData.cpp`'deki mantık, ham script verilerini oyun motorunun anlayabileceği yapılandırılmış `CEffectData` nesnelerine dönüştürür. Bu nesneler, oyun dünyasında görünen canlı efektlerin (`CEffectInstance`) temelini oluşturur.

## `EffectInstance.cpp` Implementasyon Detayları

`EffectInstance.cpp`, `CEffectInstance` sınıfının metodlarını uygulayarak bir efekt şablonundan (`CEffectData`) oyun dünyasında aktif, canlı bir efekt örneğinin nasıl oluşturulduğunu, güncellendiğini, render edildiğini ve yönetildiğini tanımlar.

*   **Bellek Yönetimi ve Sayaçlar (`New`, `Delete`, `DestroySystem`, `ResetRenderingEffectCount`, `GetRenderingEffectCount`)**:
    *   `New()`: `CEffectInstance::ms_kPool` (statik `CDynamicPool`) üzerinden yeni bir `CEffectInstance` nesnesi ayırır.
    *   `Delete(CEffectInstance* pkEftInst)`: Verilen `CEffectInstance` nesnesinin `Clear()` metodunu çağırarak içerdiği tüm alt eleman örneklerini (parçacık, mesh, ışık) temizler ve kendi havuzlarına iade eder, ardından ana `CEffectInstance` nesnesini `ms_kPool`'a geri verir.
    *   `DestroySystem()`: `CEffectInstance`'ın kendi bellek havuzunu ve ayrıca bağımlı olduğu tüm alt eleman örnek sınıflarının (`CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance`) statik `DestroySystem()` metotlarını çağırarak onların da havuzlarını temizler.
    *   `ResetRenderingEffectCount()` / `GetRenderingEffectCount()`: `ms_iRenderingEffectCount` statik sayacını sıfırlar veya değerini döndürür. Bu sayaç, `OnRender()` içinde artırılarak bir frame'de kaç efektin render edildiğini takip etmek için kullanılır (performans izleme/hata ayıklama için).

*   **Render Sıralaması (`LessRenderOrder`)**:
    *   Bu metod, iki `CEffectInstance`'ı render sırasına göre karşılaştırmak için kullanılır. Mevcut implementasyon, doğrudan `m_pkEftData` işaretçilerini karşılaştırır (`return (m_pkEftData < pkEftInst->m_pkEftData);`). Bu, efektlerin efekt veri adreslerine göre sıralanacağı anlamına gelir, ki bu genellikle saydamlık için istenen bir sıralama değildir. Saydamlık sıralaması genellikle efektin kameraya olan mesafesine veya belirli bir öncelik değerine göre yapılır. Bu implementasyon, daha basit bir sıralama stratejisi veya özel bir kullanım durumu olabilir.

*   **Ses Güncelleme (`UpdateSound`)**:
    *   Eğer `m_pSoundInstanceVector` (efekt verisinden gelen ses bilgileri) geçerliyse, `CSoundManager::Instance().UpdateSoundInstance()` çağrılır. Bu, efektin mevcut dünya pozisyonu (`m_matGlobal`'den alınır), efektin kendi iç zamanlayıcısı (`m_dwFrame`) ve ses bilgileri kullanılarak 3D seslerin doğru bir şekilde çalınmasını ve güncellenmesini sağlar. `m_dwFrame` her çağrıda artırılır.

*   **Efekt Elemanı Güncelleme (`OnUpdate`)**:
    *   `Transform()` (muhtemelen `CGraphicObjectInstance` temel sınıfından gelir ve `m_matGlobal` gibi matrisleri günceller) çağrılır.
    *   `FEffectUpdator` adında bir functor yapısı kullanılır:
        *   Bu functor, bir `CEffectElementBaseInstance*` alır ve onun `Update(fElapsedTime)` metodunu çağırır. Eğer herhangi bir eleman `Update()` çağrısında `true` (yani hala aktif) dönerse, functor'ın `isAlive` bayrağını `true` yapar.
        *   Geçen süre (`fElapsedTime`), `WORLD_EDITOR` makrosu tanımlıysa `CTimer::Instance().GetElapsedSecond()` ile, değilse `CTimer::Instance().GetCurrentSecond() - m_fLastTime` ile hesaplanır.
        *   `std::for_each` kullanılarak `m_ParticleInstanceVector`, `m_MeshInstanceVector` ve `m_LightInstanceVector` içindeki tüm alt eleman örnekleri bu functor ile güncellenir.
    *   `CEffectInstance`'ın kendi `m_isAlive` bayrağı, `FEffectUpdator`'ın son `isAlive` durumuna göre ayarlanır (yani, en az bir alt eleman aktifse ana efekt de aktif kalır).
    *   `m_fLastTime`, bir sonraki `Update` çağrısında delta zamanı hesaplamak için güncellenir.

*   **Render (`OnRender`)**:
    *   Saydam efektler için tipik olan genel render durumlarını `STATEMANAGER` aracılığıyla ayarlar ve kaydeder (örneğin, `D3DRS_ALPHABLENDENABLE = true`, `D3DRS_ZWRITEENABLE = false`, `D3DRS_CULLMODE = D3DCULL_NONE`).
    *   Doku aşaması durumlarını (Texture Stage States), doku renginin bir `TFACTOR` (muhtemelen efektin genel alfa/renk modülasyonu için) ile modüle edilmesi için ayarlar.
    *   Vertex shader formatını `D3DFVF_XYZ | D3DFVF_TEX1` (TPTVertex) olarak ayarlar.
    *   `std::for_each` ve `std::mem_fn` kullanarak `m_ParticleInstanceVector` ve `m_MeshInstanceVector` içindeki tüm elemanların `Render()` metotlarını çağırır. Işık örnekleri (`m_LightInstanceVector`) render edilmez, çünkü ışıklar sahneyi etkiler ama doğrudan çizilmezler.
    *   Render işlemi bittikten sonra, `OnRender` başında kaydedilen tüm render durumlarını geri yükler.
    *   `ms_iRenderingEffectCount`'u artırır.

*   **Durum Yönetimi (`isAlive`, `SetActive`, `SetDeactive`)**:
    *   `isAlive()`: `m_isAlive` bayrağını döndürür.
    *   `SetActive()` / `SetDeactive()`: Tüm alt eleman örneklerinin (`m_ParticleInstanceVector`, `m_MeshInstanceVector`, `m_LightInstanceVector`) `SetActive()` veya `SetDeactive()` metotlarını `std::for_each` ve `std::mem_fn` kullanarak çağırır.

*   **Alt Eleman Örneği Oluşturma Yardımcıları (`__SetParticleData`, `__SetMeshData`, `__SetLightData`)**:
    *   Bu özel (private) metotlar, `SetEffectDataPointer` içinde kullanılır.
    *   Her biri, ilgili eleman veri türünü (`CParticleSystemData*`, `CEffectMeshScript*`, `CLightData*`) alır.
    *   İlgili örnek sınıfının (`CParticleSystemInstance`, `CEffectMeshInstance`, `CLightInstance`) statik `New()` metoduyla havuzdan bir örnek alır.
    *   Oluşturulan örneğin `SetDataPointer()` metodunu çağırarak veri bloğunu bağlar.
    *   Örneğin `SetLocalMatrixPointer(&m_matGlobal)` ile ana efektin dünya matrisini verir, böylece alt elemanlar ana efekte göre doğru konumlanır.
    *   Parçacık ve mesh örnekleri için `SetParticleScale()` veya `SetMeshScale()` ile ana efektin ölçeklerini uygular.
    *   Oluşturulan örneği `CEffectInstance`'ın kendi ilgili vektörüne (`m_ParticleInstanceVector` vb.) ekler.

*   **Efekt Verisi Atama (`SetEffectDataPointer`)**:
    *   Bu, bir `CEffectData` şablonundan canlı bir `CEffectInstance` oluşturmanın anahtarıdır.
    *   `m_isAlive` bayrağını `true` yapar.
    *   `m_pkEftData` işaretçisini verilen `pEffectData`'ya ayarlar.
    *   `m_fLastTime`'ı günceller ve `CEffectData`'dan sınırlayıcı küre bilgilerini (`m_fBoundingSphereRadius`, `m_v3BoundingSpherePosition`) alır.
    *   Eğer sınırlayıcı küre yarıçapı pozitifse, `CGraphicObjectInstance::RegisterBoundingSphere()` ile bu örneği muhtemelen frustum culling (görüş alanı dışı eleme) gibi optimizasyonlar için grafik sistemine kaydeder.
    *   `pEffectData` içindeki tüm parçacık, mesh ve ışık tanımları üzerinde döngüye girer.
    *   Her tanım için ilgili `__SetParticleData`, `__SetMeshData` veya `__SetLightData` yardımcı metodunu çağırarak karşılık gelen alt eleman örneklerini oluşturur ve ayarlar.
    *   `m_pSoundInstanceVector` işaretçisini `pEffectData`'dan gelen ses örneklerine ayarlar.

*   **Sınırlayıcı Küre Alma (`GetBoundingSphere`)**:
    *   Efektin dünya koordinatlarındaki sınırlayıcı küre merkezini, `m_matGlobal` (efektin dünya matrisi) ve `m_v3BoundingSpherePosition` (veriden gelen yerel ofset) kullanarak hesaplar. Yarıçap doğrudan `m_fBoundingSphereRadius`'tan alınır. Bu, `CGraphicObjectInstance`'ın gerektirdiği bir fonksiyondur ve frustum culling için kullanılır.

*   **Temizleme (`Clear`)**:
    *   Tüm alt eleman örnek vektörleri (`m_ParticleInstanceVector`, `m_MeshInstanceVector`, `m_LightInstanceVector`) üzerinde gezer.
    *   Her bir vektördeki tüm örnekler için ilgili `Delete` metodunu (`CParticleSystemInstance::Delete`, vb.) çağırarak örnekleri kendi bellek havuzlarına iade eder.
    *   Vektörleri `.clear()` ile temizler.
    *   Diğer üye değişkenlerini (`m_pkEftData`, `m_pSoundInstanceVector`, `m_isAlive` vb.) varsayılan durumlarına sıfırlar.
    *   Eğer bir sınırlayıcı küre kaydı yapılmışsa, `CGraphicObjectInstance::UnregisterBoundingSphere()` ile kaydı siler.

*   **Kurucu ve Yıkıcı (`CEffectInstance()`, `~CEffectInstance()`)**:
    *   Kurucu (`CEffectInstance()`): `m_isAlive`'ı `false` yapar, `m_pkEftData` ve `m_pSoundInstanceVector`'ü `NULL` yapar, parçacık ve mesh ölçeklerini varsayılan (1.0f) yapar.
    *   Yıkıcı (`~CEffectInstance()`): `Clear()` metodunu çağırır.

**Oyun İçi Kullanım Alanları:**

`CEffectInstance` ve onun `.cpp` implementasyonu, oyun dünyasında fiilen görünen, hareket eden ve ses çıkaran her bir görsel efektin ta kendisidir.

1.  **Canlı Efekt Temsili:** Bir büyü yapıldığında, bir patlama olduğunda veya bir karakter özel bir yetenek kullandığında, `CEffectManager` tarafından bir `CEffectData` şablonu kullanılarak bir `CEffectInstance` oluşturulur. Bu örnek, efektin o andaki tüm dinamik durumunu (aktif parçacıkların pozisyonları, mesh animasyonunun mevcut karesi, ışığın o anki parlaklığı vb.) tutar.
2.  **Dinamik Güncelleme:** Oyunun her bir frame'inde, `CEffectManager` tüm aktif `CEffectInstance`'ların `OnUpdate()` metodunu çağırır. Bu metod, örneğin içindeki tüm parçacık sistemlerini, meshleri ve ışıkları günceller. Parçacıklar hareket eder, renkleri/boyutları değişir, mesh animasyonları ilerler, ışıkların menzili zamanla değişebilir. Efektin genel ömrü de burada takip edilir.
3.  **Görselleştirme (Render):** `CEffectManager` tarafından `OnRender()` metodu çağrıldığında, `CEffectInstance` ekrana çizilir. Bu, içerdiği tüm görünür parçacıkların ve meshlerin uygun dokular, saydamlık ayarları ve billboard davranışlarıyla çizilmesini içerir. Işıklar doğrudan çizilmez ancak özellikleri `CLightInstance` tarafından güncellenerek sahne aydınlatmasına katkıda bulunur.
4.  **Etkileşim ve Konumlandırma:** Efektin oyun dünyasındaki pozisyonu, rotasyonu ve ölçeği `m_matGlobal` dünya matrisi ile belirlenir. Bu matris, `CEffectManager` tarafından veya doğrudan oyun mantığı tarafından ayarlanabilir. Bu sayede efektler karakterlere, nesnelere veya belirli dünya koordinatlarına bağlanabilir ve onlarla birlikte hareket edebilir.
5.  **Yaşam Süresi Yönetimi:** Her `CEffectInstance`'ın (ve içindeki elemanların) bir ömrü vardır. `OnUpdate` metodu, bu ömrü takip eder. Efektin veya tüm alt elemanlarının ömrü dolduğunda, `m_isAlive` bayrağı `false` olur ve `CEffectManager` bu örneği bir sonraki güncelleme döngüsünde temizler, kaynaklarını (bellek havuzuna iade ederek) serbest bırakır.
6.  **Ses Entegrasyonu:** `UpdateSound` metodu sayesinde, efektin görsel bileşenleriyle senkronize olarak sesler çalınabilir. Örneğin, bir patlama efekti hem görsel bir şok dalgası hem de bir patlama sesi üretebilir.
7.  **Optimizasyon:** Sınırlayıcı küre (`BoundingSphere`) kullanımı, efektin kamera görüş alanı dışında olup olmadığını hızlıca kontrol ederek gereksiz güncelleme ve render işlemlerini atlamaya (frustum culling) yardımcı olur.

Özetle, `EffectInstance.cpp`'deki mantık, bir efektin teorik tanımını (`CEffectData`) alıp onu oyun dünyasında yaşayan, nefes alan, değişen ve oyuncu tarafından algılanan bir varlığa dönüştürür. 