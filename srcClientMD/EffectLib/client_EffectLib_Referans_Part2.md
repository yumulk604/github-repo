# EffectLib Referans Kılavuzu - Bölüm 2: C++ Implementasyon Detayları

Bu belge, `EffectLib` modülündeki çeşitli sınıfların `.cpp` dosyalarında bulunan implementasyon detaylarına odaklanmaktadır. Başlık dosyaları (`.h`) için temel açıklamalar ana `client_EffectLib_Referans.md` dosyasında bulunabilir.

## `ParticleSystemData.cpp` Implementasyon Detayları

`ParticleSystemData.cpp` dosyası, `CParticleSystemData` sınıfının işlevselliğini uygular. Temel özellikleri şunlardır:

*   **Bellek Yönetimi**: `CParticleSystemData` nesnelerinin verimli bir şekilde oluşturulması ve silinmesi için statik bir `CDynamicPool<CParticleSystemData>` (`ms_kPool`) kullanılır. `New()` metodu bu havuzdan bir nesne ayırırken, `Delete()` metodu nesneyi temizleyip havuza geri gönderir. `DestroySystem()` ise tüm havuzu temizler.
*   **Özellik Erişimi**: `GetEmitterPropertyPointer()` ve `GetParticlePropertyPointer()` metodları, sırasıyla gömülü `CEmitterProperty` ve `CParticleProperty` nesnelerine doğrudan erişim sağlar.
*   **Script Yükleme (`OnLoadScript`)**: Bu metod, `CTextFileLoader` aracılığıyla bir partikül sistemi script dosyasından verileri ayrıştırmaktan sorumludur.
    *   "emitterproperty" bölümü altındaki yayıcı (emitter) özelliklerini okur:
        *   Temel yayıcı parametreleri: `maxemissioncount`, `cyclelength`, `cycleloopenable`, `loopcount`.
        *   Yayıcı şekli ve tipi: `emittershape`, `emitteradvancedtype`.
        *   Yayılma hacmi: `emittingsize`, `emittingradius`, `emitteremitfromedgeflag`.
        *   Yayılma yönü: `emittingdirection`.
        *   Yayıcı özellikleri için zaman tabanlı olaylar: `timeeventemittingsize`, `timeeventemittingangularvelocity`, `timeeventemittingdirectionx/y/z`, `timeeventemittingvelocity`, `timeeventemissioncountpersecond`, `timeeventlifetime`, `timeeventsizex`, `timeeventsizey`. Token bulunamazsa varsayılan değerler veya boş olay listeleri atanır.
    *   "particleproperty" bölümü altındaki partikül özelliklerini okur:
        *   Karıştırma (blending) ve renk işlemleri: `srcblendtype`, `destblendtype`, `coloroperationtype`.
        *   Billboard ve rotasyon: `billboardtype`, `rotationtype`, `rotationspeed`, `rotationrandomstartingbegin`, `rotationrandomstartingend`.
        *   Bağlanma (attachment) ve esneme (stretching): `attachenable`, `stretchenable`.
        *   Doku animasyonu: `texanitype`, `texanidelay`, `texanirandomstartframeenable`.
        *   Fiziksel özellikler (yerçekimi, hava direnci): `gravity` (veya `timeeventgravity`), `airresistance` (veya `timeeventairresistance`). Bu özellikler için hem tek bir float değeri hem de zaman-olay tablolarını destekler.
        *   Partikül ölçeği için zaman tabanlı olaylar: `timeeventscalex`, `timeeventscaley`.
        *   Partikül rengi ve alfası için zaman tabanlı olaylar: `timeeventcolorred`, `timeeventcolorgreen`, `timeeventcolorblue`, `timeeventalpha`.
            *   Kod, `WORLD_EDITOR` makrosunun tanımlı olup olmamasına bağlı olarak renk olaylarını farklı şekilde işler. Tanımlı değilse (tipik istemci yapılarında), R, G, B, A zaman olay tablolarını ayrı ayrı okur ve ardından bunları tek bir `m_ParticleProperty.m_TimeEventColor` tablosunda birleştirir. Bu birleştirme süreci, bireysel renk/alfa kanallarındaki tüm benzersiz zaman noktalarını toplamayı ve ardından bu zaman noktalarının her birinde renk değerlerini enterpole ederek birleştirilmiş bir `TTimeEventTypeColor` olay listesi oluşturmayı içerir.
*   **Başlatma ve Temizleme**:
    *   `CParticleSystemData()` (Kurucu): Üye değişkenlerini başlatır, öncelikle `m_fBoundingSphereRadius` için varsayılan değerleri ayarlar ve `m_ImageVector`'ü temizler.
    *   `Clear()`: `CEmitterProperty` ve `CParticleProperty`'nin tüm özelliklerini varsayılan değerlerine sıfırlar ve `m_ImageVector`'ü temizler. Bu, bir nesne havuza geri bırakılmadan önce çağrılır.
*   **Sınırlayıcı Küre Hesaplanması (`UpdateBoundingSphere`)**: Bu metod, partikül sistemi için sınırlayıcı küre yarıçapını hesaplar. Yalnızca `m_EmitterProperty.m_fEmittingRadius` ve sabit bir değeri dikkate aldığı için (özellikle hareketli partiküller veya karmaşık yayıcı şekilleriyle) partikül sisteminin gerçek boyutunu doğru bir şekilde yansıtmayabilecek bir yer tutucu veya çok temel bir uygulama gibi görünmektedir.
*   **Doku Yönetimi**:
    *   `InsertTexture(const char* c_szFileName)`: `m_ImageVector`'e bir doku dosya adı ekler.
    *   `GetTexture(DWORD dwIndex)`: Verilen indeksteki doku için bir resim işaretçisi (`CGraphicImage::Ptr`) alır. Resim zaten mevcut değilse yüklemek için `CResourceManager`'a güvenir gibi görünmektedir.

Sınıf, script dosyasından belirli veri türlerini ayrıştırmak için `GetTokenTimeEventFloat`, `GetTokenTimeEventColor`, `GetTokenVector3`, `GetTokenByte` vb. (muhtemelen `CTextFileLoader`'ın üyeleri veya global yardımcı fonksiyonlar) gibi yardımcı fonksiyonlara büyük ölçüde dayanır. Script'te belirli bir token bulunamazsa genellikle varsayılan değerler atanır, bu da sağlamlığı artırır.

## `ParticleSystemInstance.cpp` Implementasyon Detayları

*   **Bellek Yönetimi (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: Diğer efekt sınıflarına benzer şekilde, `CParticleSystemInstance` örneklerinin verimli bellek tahsisi için bir `CDynamicPool` kullanır. `DestroySystem()` ayrıca `CParticleInstance::DestroySystem()` çağrısını yaparak bireysel partikül havuzlarının da temizlenmesini sağlar.
*   **Parçacık Oluşturma (`CreateParticles`)**: Bu metod, `CEmitterProperty`'ye dayalı olarak yeni partiküller yaymaktan sorumludur.
    *   `m_pEmitterProperty`'den alınan `EmissionCountPerSecond`, geçen süre ve `m_fEmissionResidue` (kesirli partikül sayılarını zamanla sorunsuz bir şekilde işlemek için) temelinde mevcut karede kaç partikül oluşturulacağını hesaplar.
    *   Yayıcı özelliğinde tanımlanan `MaxEmissionCount`'a uyar.
    *   `m_pEmitterProperty` ve `m_pData` (`CParticleSystemData`) üzerinden mevcut yerel zamanda (`m_fLocalTime`) çeşitli partikül özelliklerini alır:
        *   `ParticleLifeTime`: Yaşam süresi sıfırsa, partikül oluşturulmaz.
        *   `EmittingSize`: Yayılma için ek bir boyut ofseti.
        *   `Position`: `CParticleSystemData`'dan mevcut zamandaki temel pozisyon.
        *   `EmittingDirectionX/Y/Z`: Temel yayılma yönü bileşenleri.
        *   `EmittingVelocity`: Temel yayılma hızı.
        *   `ParticleSizeX/Y`: `m_fParticleScale` ile ölçeklenmiş temel partikül boyutu.
    *   Billboard türü `BILLBOARD_TYPE_LIE` ise ve bir yerel matris (`mc_pmatLocal`) mevcutsa, yerel matrisin yönelimine göre bir başlangıç `fLieRotation` hesaplar. Bu, "yatan" partikülleri bir yüzeyle hizalamak için kullanılır.
    *   Oluşturulacak her partikül (`iCreatingCount`) için:
        *   Havuzundan yeni bir `CParticleInstance` ayrılır.
        *   Partikül örneğine `m_pParticleProperty` ve `m_pEmitterProperty` işaretçileri atanır.
        *   **Yaşam Süresi**: Hesaplanan `fLifeTime`'dan ayarlanır.
        *   **Pozisyon**: `m_pEmitterProperty`'den alınan `EmitterShape` ile belirlenir:
            *   `EMITTER_SHAPE_POINT`: Pozisyon (0,0,0).
            *   `EMITTER_SHAPE_ELLIPSE`: Bir elips üzerinde veya içinde rastgele pozisyon (normalize edilmiş rastgele vektör, `m_fEmittingRadius` veya bunun rastgele bir kesri ile ölçeklenir, artı `fEmittingSize`).
            *   `EMITTER_SHAPE_SQUARE`: `m_v3EmittingSize` kutusu içinde rastgele pozisyon, artı `fEmittingSize`.
            *   `EMITTER_SHAPE_SPHERE`: Bir küre üzerinde veya içinde rastgele pozisyon (elipse benzer ancak 3D'de).
        *   Partikülün pozisyonu daha sonra `_v3TimePosition` (mevcut zamanda `CParticleSystemData`'dan pozisyon) ile kaydırılır.
        *   Bir yerel matris (`mc_pmatLocal`) mevcutsa ve partikül bağlı değilse (`!m_pParticleProperty->m_bAttachFlag`), pozisyonu bu matrisle dönüştürülür. `m_v3StartPosition`'ı ayarlamak için `v3TimePosition` da dönüştürülür.
        *   **Hız ve Yön**:
            *   (0,0,0) olarak başlatılır.
            *   `EMITTER_ADVANCED_TYPE_INNER`: Hız, `v3TimePosition`'a (yayılma merkezi) doğru içe yönlendirilir.
            *   `EMITTER_ADVANCED_TYPE_OUTER`:
                *   `EMITTER_SHAPE_POINT` ise: Hız rastgele bir 3D vektördür.
                *   Değilse: Hız, `v3TimePosition`'dan dışa doğru yönlendirilir.
            *   Temel `_v3Velocity` (yayıcı özelliklerinden) eklenir. Bir yerel matris mevcutsa ve bağlı değilse, bu temel hız yerel matrisle dönüştürülür (bir normal olarak, yani çeviri olmaz).
            *   `m_pEmitterProperty->m_v3EmittingDirection`'a dayalı rastgele ofsetler (bileşenler > 0 ise) her hız bileşenine eklenir.
            *   Son olarak, toplam hız `fVelocity` (yayıcı özelliklerinden) ile ölçeklenir.
        *   **Boyut**: `m_v2HalfSize`, hesaplanan `v2HalfSize`'dan ayarlanır.
        *   **Rotasyon**: Başlangıç rotasyonu, `m_wRotationRandomStartingBegin` ve `m_wRotationRandomStartingEnd` arasında rastgele bir değerdir. `BILLBOARD_TYPE_LIE` ise, `fLieRotation` eklenir.
        *   **Doku Animasyonu**:
            *   `m_byFrameIndex` başlatılır.
            *   `m_byTextureAnimationType`, `CParticleProperty`'den ayarlanır.
            *   `TEXTURE_ANIMATION_TYPE_RANDOM_DIRECTION` ayarlanmışsa ve birden fazla kare varsa, rastgele CW veya CCW animasyonunu seçer ve başlangıç karesini buna göre ayarlar.
            *   `m_bTexAniRandomStartFrameFlag` doğruysa, rastgele bir başlangıç karesi seçilir.
        *   Yeni partikül bir listeye (`pDecoratorList`, daha sonra `m_ParticleInstanceListVector`'e taşınır) eklenir.
        *   Dekoratörler (`m_pDecorator`) oluşturulur ve yeni partikül örneğine uygulanır.
    *   `m_dwCurrentEmissionCount` güncellenir.
*   **Güncelleme (`OnUpdate`)**:
    *   `m_fLocalTime`'ı artırır.
    *   Yayıcının döngüsünün tekrarlanıp tekrarlanmayacağını (`m_pEmitterProperty->IsLoop()`) kontrol eder. Döngü yapıyorsa ve döngü uzunluğuna ulaşılmışsa, `m_fLocalTime` ayarlanır ve `m_iLoopCount` artırılır.
    *   Yayıcı hala aktifse (`m_dwCurrentEmissionCount < m_pEmitterProperty->GetMaxEmissionCount()` ve döngü koşulları karşılanıyorsa):
        *   Yeni partiküller oluşturmak için `CreateParticles(fElapsedTime)` çağrılır.
    *   `m_ParticleInstanceListVector` (muhtemelen doku başına bir liste olan partikül listelerini tutar) üzerinde yinelenir. Her partikül için:
        *   Partikülün `Update(fElapsedTime)` metodu çağrılır.
        *   `Update` `false` döndürürse (partikül ölmüşse), `CParticleInstance::Delete()` kullanılarak silinir ve listeden çıkarılır.
    *   Fonksiyon, herhangi bir partikül hala hayattaysa veya yayıcı hala aktifse `true`, aksi takdirde `false` (sistem örneğinin temizlenmeye hazır olabileceğini gösterir) döndürür.
*   **Render (`OnRender`)**:
    *   Ortak render durumlarını ayarlar (örneğin, `D3DRS_ZWRITEENABLE = FALSE`).
    *   `m_ParticleInstanceListVector` ve karşılık gelen `m_kVct_pkImgInst` (resim örnekleri) üzerinde yinelenir.
    *   Her doku grubu için:
        *   `STATEMANAGER.SetTexture()` kullanarak dokuyu ayarlar.
        *   `m_pParticleProperty->m_bySrcBlendType` ve `m_pParticleProperty->m_byDestBlendType`'a göre harmanlama işlemlerini ayarlar.
        *   `m_pParticleProperty->m_byColorOperationType`'a göre renk işlemini ayarlar.
        *   `m_pParticleProperty->m_byBillboardType` ve `m_pParticleProperty->m_bAttachFlag`'e göre bir renderleyici funktör seçer:
            *   `NParticleRenderer::TwoSideRenderer`: `BILLBOARD_TYPE_2FACE` için. Açılı iki dörtgen çizer.
            *   `NParticleRenderer::ThreeSideRenderer`: `BILLBOARD_TYPE_3FACE` için. Açılı üç dörtgen çizer.
            *   `NParticleRenderer::NormalRenderer`: Bağlı değilse diğer billboard türleri için.
            *   `NParticleRenderer::AttachRenderer`: `m_bAttachFlag` doğruysa. Yerel matrisi kullanır.
        *   Seçilen renderleyici ile `ForEachParticleRendering` çağrılır. Bu şablon fonksiyonu, mevcut listedeki partiküller üzerinde yinelenir, `InFrustum` kontrolü yapar ve görünür partiküller için renderleyici funktörünü çağırır.
*   **Veri Ayarlama (`OnSetDataPointer`)**:
    *   `CParticleSystemData` atandığında çağrılır.
    *   `CParticleSystemData`, `CParticleProperty` ve `CEmitterProperty` işaretçilerini saklar.
    *   `CParticleSystemData`'nın `m_ImageVector`'ünde belirtilen dokuları `m_kVct_pkImgInst`'e ( `CGraphicImageInstance` vektörü) yükler.
    *   `m_pData->BuildDecorator(this)` kullanarak dekoratörler oluşturur.
*   **Yok Etme (`Destroy`, `Clear`)**:
    *   `Destroy()`, `Clear()`'ı ve ardından `CEffectElementBaseInstance::Destroy()`'ı çağırır.
    *   `Clear()`: `m_ParticleInstanceListVector`'deki tüm aktif `CParticleInstance`'ları siler, vektörü temizler, `m_kVct_pkImgInst`'i temizler, dekoratörleri siler ve durum değişkenlerini sıfırlar.
*   **Sınırlayıcı Kutu/Küre**:
    *   `GetBoundingBox(D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Tüm aktif partiküllerin AABB'sini hesaplar.
    *   `UpdateBoundingSphere()`: Parçacıklarının AABB'sine ve yerel matrise göre kendi sınırlayıcı küresini günceller.
*   **Frustum Eleme (`InFrustum`)**: Verilen bir `CParticleInstance`'ın `CGraphicObjectInstance::IsInFrustum()` kullanarak görünüm frustumunun içinde olup olmadığını kontrol eder.
*   **Parçacık Render Funktörleri (`NParticleRenderer`)**:
    *   `TwoSideRenderer`, `ThreeSideRenderer`, `NormalRenderer`, `AttachRenderer`: Bu yapılar `ForEachParticleRendering` tarafından kullanılan funktörlerdir. Her birinin bir `CParticleInstance*` alan bir `operator()`'ı vardır.
    *   `operator()`'ları içinde, `pInstance->Transform(...)` (billboard türü için uygun matris/rotasyon ile) çağırır ve ardından partikülü (`TPTVertex`'lerden oluşan bir dörtgen) çizmek için `STATEMANAGER.DrawPrimitiveUP(...)` çağırır.

Sınıf, birden fazla `CParticleInstance` nesnesinin yaşam döngüsünü yönetir, yayıcı özelliklerine göre oluşturulmalarını düzenler, bireysel durumlarını günceller ve doku başına toplu işleme ve uygun billboarding teknikleriyle verimli bir şekilde render eder.

## `ParticleInstance.cpp` Implementasyon Detayları

`ParticleInstance.cpp`, tek bir partikülün yaşam döngüsünü, davranışını ve render için hazırlanmasını yöneten `CParticleInstance` sınıfının metodlarını içerir.

*   **Bellek Yönetimi (`ms_kPool`, `New`, `DeleteThis`, `DestroySystem`)**:
    *   `CParticleInstance` nesneleri için bir `CDynamicPool` (`ms_kPool`) kullanılır.
    *   `New()`: Havuzdan bir nesne ayırır.
    *   `DeleteThis()`: Dahili `Destroy()` metodunu çağırır (bu kod parçasında gösterilmeyen, muhtemelen temel sınıftan gelen veya iç temizlik için olan bir metod) ve ardından nesneyi `ms_kPool`'a geri verir.
    *   `DestroySystem()`: Statik bellek havuzunu temizler.

*   **Yorum Satırları (`CRayParticleInstance`)**: Kodda `CRayParticleInstance` için yorum satırına alınmış bir bölüm bulunur. Bu, daha önce var olan veya planlanmış ancak şu an aktif olmayan özel bir ışın tipi partikül türüne işaret edebilir.

*   **`GetRadiusApproximation()`**: Partikülün yaklaşık yarıçapını, ölçeklenmiş yarı boyutlarının toplamı (`m_v2HalfSize.y * m_v2Scale.y + m_v2HalfSize.x * m_v2Scale.x`) olarak hesaplar. Bu, kaba bir görünürlük kontrolü (culling) veya etkileşim tespiti için kullanılabilir.

*   **`Update(float fElapsedTime, float fAngle)`**: Tek bir partikülün durumunu güncelleyen ana fonksiyondur.
    *   `m_fLastLifeTime` (kalan yaşam süresi), `fElapsedTime` (geçen süre) kadar azaltılır. Sıfırın altına düşerse, partikül ölü kabul edilir ve fonksiyon `false` döndürür.
    *   `fLifePercentage` (partikülün ömrünün ne kadarının geçtiği) hesaplanır.
    *   `m_pDecorator->Excute(CDecoratorData(fLifePercentage, fElapsedTime, this))` çağrılır. Bu çağrı, partikülün özelliklerini değiştiren dekoratörlerin (yerçekimi, hava direnci, renk değişimi vb. – `EffectUpdateDecorator.h/cpp` içinde tanımlanır) çalıştırıldığı yerdir.
    *   `m_v3LastPosition`, mevcut `m_v3Position` ile güncellenir.
    *   `m_v3Position`, `m_v3Velocity * fElapsedTime` ile güncellenir.
    *   **Yayıcı/Başlangıç Noktası Etrafında Dönüş (eğer `fAngle` sıfır değilse)**:
        *   Bu `fAngle` parametresi, partiküle `m_v3StartPosition` etrafında uygulanan harici bir dönüş gibi görünmektedir (muhtemelen dönen bir yayıcıdan veya üst bir dönüşümden kaynaklanır).
        *   Eğer `m_pParticleProperty->m_bAttachFlag` doğruysa (partikül bağlıysa, muhtemelen XY düzleminde 2D dönüş):
            *   Sinüs ve kosinüs kullanarak `m_v3StartPosition` etrafında 2D dönüş hesaplanır.
        *   Değilse (bağlı değilse, 3D dönüş):
            *   Partikülün pozisyonunu `m_v3StartPosition`'a göre `m_pParticleProperty->m_v3ZAxis` (eğer ayarlanmamışsa varsayılan olarak (0,0,1) olur, kendi koordinat sisteminde bir Z ekseni dönüşü anlamına gelir, sonra geri ötelenir) etrafında döndürmek için kuaternion matematiği kullanılır.
    *   Partikül hala hayattaysa `true` döndürür.

*   **`Transform(const D3DXMATRIX* c_matLocal)`**: Partikülün köşe noktalarını (vertices) mevcut durumuna ve billboard türüne göre render için hazırlar.
    *   Doku rengini modüle etmek için kullanılan `D3DRS_TEXTUREFACTOR` render durumunu `m_dcColor` (veya `WORLD_EDITOR` tanımlıysa `m_Color`) kullanarak ayarlar.
    *   Partikül dörtgeninin yönelimini tanımlayacak `v3Up` ve `v3Cross` vektörlerini hesaplar.
    *   **Eğer `m_pParticleProperty->m_bStretchFlag` değilse (normal billboard partikül)**:
        *   Mevcut kameranın Yukarı (Up) ve Çapraz (Cross) vektörlerini alır.
        *   `m_pParticleProperty->m_byBillboardType`'a göre bir `switch` ifadesi çalışır:
            *   `BILLBOARD_TYPE_LIE`: Partikülün (örneğin yerel uzayında XY düzleminde) düz durmasını sağlamak için `v3Up` ve `v3Cross`, `m_fRotation`'a göre hesaplanır.
            *   `BILLBOARD_TYPE_2FACE`, `BILLBOARD_TYPE_3FACE`, `BILLBOARD_TYPE_Y`:
                *   `v3Up` başlangıçta dünya Z-yukarı ((0,0,1)) olarak ayarlanır.
                *   Ters dönmeyi engellemek için kameranın bakış yönüne göre `v3Up` ayarlanıyor gibi görünmektedir.
                *   `v3Cross`, `v3Up` ile kameranın XY düzlemine izdüşürülen bakış vektörünün çapraz çarpımıyla hesaplanır.
                *   Eğer `m_fRotation` sıfır değilse, `v3Up` ve `v3Cross` ayrıca bunların birbirine dik bir eksen etrafında döndürülür (partikülü, kameranın partikülün Y-billboard düzlemine izdüşürülen bakış yönü etrafında etkili bir şekilde döndürür).
            *   `BILLBOARD_TYPE_ALL` (ve varsayılan durum):
                *   Eğer `m_fRotation` 0 ise, `v3Up` = -kameraÇapraz, `v3Cross` = kameraYukarı (standart kameraya bakan billboard).
                *   Eğer `m_fRotation` sıfır değilse, kameranın Yukarı ve Çapraz vektörlerini kameranın Bakış vektörü etrafında `m_fRotation` kadar döndürmek için kuaternion matematiği kullanır. Bu, partikülün kameraya bakmasını ancak kameradan partiküle doğru bir eksen etrafında döndürülmesine izin verir.
    *   **Else (eğer `m_pParticleProperty->m_bStretchFlag` doğruysa)**:
        *   `v3Up`, `m_v3Position - m_v3LastPosition` (hareket yönü) olarak ayarlanır.
        *   `c_matLocal` mevcutsa, `v3Up` bir normal olarak onunla dönüştürülür (esnemeyi dönüştürülmüş hareketle hizalamak için).
        *   `v3Cross`, kameranın bakış vektörü ile `v3Up`'ın çapraz çarpımıyla hesaplanır (dörtgeni esneme yönüne dik ve kameraya bakacak şekilde yapmak için).
        *   `v3Up` daha sonra `v3Cross` ve bakış vektörünün çapraz çarpımıyla yeniden hesaplanır (`v3Cross`'a dik ve aynı zamanda bir şekilde kamerayla hizalı olmasını sağlamak için).
        *   Hem `v3Up` hem de `v3Cross` normalize edilir.
        *   `v3Up`, `m_v2HalfSize.y * m_v2Scale.y` ile ölçeklenir.
        *   `v3Cross`, `m_v2HalfSize.x * m_v2Scale.x` ile ölçeklenir. `v3Cross`'u daha da ölçeklendirmek için `v3Velocity`'nin uzunluğunda bir `log1p` fonksiyonu kullanılır, bu da "esneme" etkisini (daha yüksek hızlarda daha uzun) yaratır.
    *   **Köşe Noktası Hesaplanması**:
        *   Partikül dörtgeninin (`m_ParticleMesh`) dört köşe noktası, son `v3Up`, `v3Cross` ve `m_v3Position` kullanılarak hesaplanır.
        *   `TPTVertex` (Pozisyon, Diffuse Renk, Doku koordinatları) kullanılır. Diffuse renk beyaza (0xffffffff) ayarlanır, çünkü gerçek renk modülasyonu `TEXTUREFACTOR` ile yapılır. Doku koordinatları standarttır ((0,0), (1,0), (0,1), (1,1)).
        *   `c_matLocal` mevcutsa ve partikül bağlı değilse (`!m_pParticleProperty->m_bAttachFlag`), partikülün köşe noktaları `c_matLocal` (üst efektin matrisi) ile dönüştürülür.

*   **`Transform(const D3DXMATRIX* c_matLocal, const float c_fZRotation)` (Aşırı Yüklenmiş Fonksiyon)**:
    *   Bu aşırı yüklenmiş fonksiyon özellikle `BILLBOARD_TYPE_2FACE` ve `BILLBOARD_TYPE_3FACE` içindir.
    *   Daha basittir: kameranın Yukarı ve Çapraz vektörlerini alır.
    *   Daha sonra bu vektörleri kameranın Bakış vektörü etrafında `c_fZRotation` kadar kuaternion matematiği kullanarak döndürür.
    *   Köşe noktası hesaplamasının geri kalanı, diğer `Transform` fonksiyonunun esneme olmayan kısmına benzer şekilde, bu döndürülmüş Yukarı ve Çapraz vektörleri kullanır. Bu, çok yüzlü partiküllerin kamera bakışına göre sabit yönelimlere sahip olmasına ve ardından döndürülmesine olanak tanır.

*   **Kurucu (`CParticleInstance()`)**:
    *   Varsayılan üye değerlerini ayarlamak için `__Initialize()` metodunu çağırır.

*   **`__Initialize()`**: 
    *   Çoğu üye değişkenini sıfıra veya varsayılan durumlara sıfırlar (`m_v3Position`, `m_v3LastPosition`, `m_v3Velocity`, renkler, rotasyon, ölçek, yaşam süreleri vb.).
    *   `m_pDecorator`, `NULL` olarak ayarlanır.
    *   `m_v3ZAxis`, `(0.0f, 0.0f, 1.0f)` olarak ayarlanır.

*   **`Destroy()`**: 
    *   `m_pDecorator`'ı `NULL` olarak ayarlar. Bu, dekoratörlerin başka bir yerde (muhtemelen `CParticleSystemInstance` temizlendiğinde veya dekoratör sisteminin havuzu tarafından) yönetildiğini (silindiğini) gösterir.

Sınıf, bir partikülün bireysel yaşamından, fiziksel güncellemelerinden (büyük ölçüde dekoratörlere devredilmiştir) ve çeşitli billboard ve esneme efektlerini destekleyerek render için ekrana bakan dörtgeninin hesaplanmasından sorumludur.

## `EmitterProperty.cpp` Implementasyon Detayları

`EmitterProperty.cpp` dosyası, `CEmitterProperty` sınıfının metodlarını uygular. Genellikle zamanla değişen yayıcı özelliklerini almak için kullanılan erişimci (getter) fonksiyonlarını ve sınıfın başlangıç durumunu ayarlayan fonksiyonları içerir.

*   **Erişimci Fonksiyonlar (Accessors)**:
    *   `GetEmitterShape()`: `m_byEmitterShape` üyesini döndürür.
    *   `GetEmitterAdvancedType()`: `m_byEmitterAdvancedType` üyesini döndürür.
    *   `isEmitFromEdge()`: `m_bEmitFromEdgeFlag` üyesini döndürür.

*   **Zaman-Olay Değeri Alıcıları (Time-Event Value Getters)**:
    *   `GetEmittingSize(float fTime, float* pfValue)`
    *   `GetEmittingAngularVelocity(float fTime, float* pfValue)`
    *   `GetEmittingDirectionX(float fTime, float* pfValue)`
    *   `GetEmittingDirectionY(float fTime, float* pfValue)`
    *   `GetEmittingDirectionZ(float fTime, float* pfValue)`
    *   `GetEmittingVelocity(float fTime, float* pfValue)`
    *   `GetEmissionCountPerSecond(float fTime, float* pfValue)`
    *   `GetParticleLifeTime(float fTime, float* pfValue)`
    *   `GetParticleSizeX(float fTime, float* pfValue)`
    *   `GetParticleSizeY(float fTime, float* pfValue)`
    *   Yukarıdaki tüm bu fonksiyonlar, genel bir yardımcı fonksiyon olan `GetTimeEventBlendValue(fTime, m_TimeEventTable, pfValue)` çağrısı yapar. Bu fonksiyon (muhtemelen `Type.cpp` gibi bir yardımcı dosyada tanımlıdır), parametre olarak verilen zaman (`fTime`), ilgili zaman olay tablosu (örneğin, `m_TimeEventEmittingSize`) ve bir float işaretçisi (`pfValue`) alır. Belirtilen zamanda tablodan enterpolasyonla hesaplanan değeri `*pfValue` içine yazar. Dosyadaki yorum satırları, daha önce `GetTimeEventBlendValue<TTimeEventTableFloat, float>(...)` gibi şablonlu bir versiyonun olabileceğini veya denendiğini düşündürmektedir.

*   **`Clear()` Metodu**:
    *   Tüm üye değişkenlerini varsayılan başlangıç değerlerine sıfırlar.
        *   `m_dwMaxEmissionCount` -> 0
        *   `m_fCycleLength` -> 0.0f, `m_bCycleLoopFlag` -> false, `m_iLoopCount` -> 0
        *   `m_byEmitterShape` -> `EMITTER_SHAPE_POINT`
        *   `m_byEmitterAdvancedType` -> `EMITTER_ADVANCED_TYPE_FREE`
        *   `m_bEmitFromEdgeFlag` -> false
        *   `m_v3EmittingSize`, `m_v3EmittingDirection` -> D3DXVECTOR3(0.0f, 0.0f, 0.0f)
        *   `m_fEmittingRadius` -> 0.0f
    *   Tüm `TTimeEventTableFloat` türündeki vektörlerin (örneğin, `m_TimeEventEmittingSize`, `m_TimeEventEmittingVelocity` vb.) içeriğini `clear()` metodu ile temizler.

*   **Kurucu (`CEmitterProperty()`)**:
    *   Nesne oluşturulduğunda tüm özellikleri varsayılan değerlerle başlatmak için doğrudan `Clear()` metodunu çağırır.

*   **Yıkıcı (`~CEmitterProperty()`)**:
    *   Boştur. Sınıfın doğrudan yönettiği ve `std::vector`'lerin otomatik temizliği veya nesnenin kendisi için kullanılan bellek havuzu sistemi dışında özel bir kaynak temizliği gerektiren dinamik olarak ayrılmış üyesi bulunmamaktadır.

`CEmitterProperty.cpp`'deki temel mantık, zamana bağlı değerleri almak için harici `GetTimeEventBlendValue` fonksiyonuna dayanır, bu da bu sınıftaki getter metodlarının implementasyonlarını bu yardımcı fonksiyona yapılan kısa sarmalayıcılar (wrapper) haline getirir.

## `ParticleProperty.cpp` Implementasyon Detayları

`ParticleProperty.cpp` dosyası, `CParticleProperty` sınıfının partikül özelliklerini yönetme işlevlerini uygular. Bu, doku yönetimi, özelliklerin varsayılan değerlere sıfırlanması ve özelliklerin bir nesneden diğerine kopyalanmasını içerir.

*   **Doku Yönetimi**:
    *   `InsertTexture(const char* c_szFileName)`:
        *   Verilen dosya adı için `CResourceManager::Instance().GetResourcePointer(c_szFileName)` kullanarak bir `CGraphicImage*` (grafik resim işaretçisi) alır.
        *   Bu resim işaretçisini `m_ImageVector` adlı vektöre ekler.
        *   Eğer `WORLD_EDITOR` makrosu tanımlıysa, doku dosya adını ayrıca `m_TextureNameVector` adlı bir string vektöründe saklar. Bu, muhtemelen editör ortamında dosya adlarına daha kolay erişim veya yeniden yükleme gibi işlemler için kullanılır.
    *   `SetTexture(const char* c_szFileName)`:
        *   Eğer `m_ImageVector` zaten birden fazla doku içeriyorsa bir `assert` ile programı durdurur (hata ayıklama modunda) ve `false` döndürür. Bu, bu fonksiyonun genellikle tek bir ana doku ayarlamak veya mevcut tüm dokuları değiştirmek için tasarlandığını gösterir.
        *   `m_ImageVector`'ü (ve `WORLD_EDITOR` tanımlıysa `m_TextureNameVector`'ü) temizler.
        *   Yeni dokuyu eklemek için `InsertTexture()` fonksiyonunu çağırır.
        *   Başarılı olursa `true` döndürür.

*   **`Clear()` Metodu**:
    *   Tüm üye değişkenlerini tanımlanmış varsayılan durumlarına sıfırlar:
        *   Rotasyonla ilgili özellikler (`m_byRotationType`, `m_fRotationSpeed`, `m_wRotationRandomStartingBegin`, `m_wRotationRandomStartingEnd`) sıfırlanır.
        *   Bayraklar (`m_bAttachFlag`, `m_bStretchFlag`) `false` olarak ayarlanır.
        *   Karıştırma (blending) özellikleri: `m_bySrcBlendType`, `BLACK_COLOR` makrosu tanımlıysa `D3DBLEND_BOTHSRCALPHA`'ya, aksi takdirde `D3DBLEND_SRCALPHA`'ya ayarlanır. `m_byDestBlendType` `D3DBLEND_ONE`'a ayarlanır. `m_byColorOperationType` `D3DTOP_MODULATE`'e ayarlanır.
        *   `m_byBillboardType`, `BILLBOARD_TYPE_NONE` olarak ayarlanır.
        *   Doku animasyon özellikleri (`m_byTexAniType`, `m_fTexAniDelay`, `m_bTexAniRandomStartFrameFlag`) varsayılan değerlerine (animasyon yok, 0.05s gecikme, rastgele başlangıç karesi yok) ayarlanır.
        *   Daha önce doğrudan float üyesi olabilecek `m_fGravity` ve `m_fAirResistance` için yorum satırları bulunur; bunlar artık zaman olay tablolarıyla yönetilmektedir.
        *   Tüm zaman olay tabloları (`m_TimeEventGravity`, `m_TimeEventAirResistance`, `m_TimeEventScaleX`, `m_TimeEventScaleY`, `WORLD_EDITOR` tanımlıysa `m_TimeEventColorRed/Green/Blue/Alpha`, değilse `m_TimeEventColor`, `m_TimeEventRotation`) kendi `.clear()` metodları kullanılarak temizlenir.
        *   `m_ImageVector` (ve `WORLD_EDITOR` tanımlıysa `m_TextureNameVector`) temizlenir.

*   **Kurucu (`CParticleProperty()`)**:
    *   Boştur. `Clear()` metodu burada çağrılmaz. Bu, üyelerin `Clear()` açıkça çağrılmadıkça veya bir script dosyasından yüklenmedikçe başlangıçta belirsiz değerlere sahip olabileceği anlamına gelir. (Ancak, `CEmitterProperty` gibi diğer sınıfların kurucularında `Clear()` çağrıldığı göz önüne alındığında, burada da çağrılması beklenir veya bu bir eksiklik olabilir. Sadece bu kod parçasına dayanarak yorum yapılmıştır.)

*   **Yıkıcı (`~CParticleProperty()`)**:
    *   Boştur. `std::vector` kendi belleğini yönettiği ve resim işaretçileri `CResourceManager` tarafından yönetildiği için burada açık bir dinamik bellek temizleme işlemine gerek yoktur.

*   **Atama Operatörü (`operator=`)**:
    *   `CParticleProperty` nesnesinin tüm üyelerinin `c_ParticleProperty` kaynağından mevcut nesneye derin kopyasını (deep copy) sağlar.
    *   Bu, tüm temel türleri, bayrakları ve önemli olarak tüm `std::vector` üye değişkenlerinin (zaman olay tabloları, resim vektörü, doku adı vektörü) içeriklerini kopyalar.

Bu implementasyon, doku yükleme ve yönetimi için `CResourceManager`'a dayanır. `WORLD_EDITOR` için koşullu derleme, editörün doku adlarını referans veya yeniden yükleme kolaylığı için saklayabileceğini, istemcinin ise sadece resim işaretçilerine ihtiyaç duyabileceğini gösterir. Sınıf, `Clear()` metodu aracılığıyla özelliklerin bilinen bir varsayılan duruma sıfırlanmasını sağlar.

## `EffectMesh.cpp` Implementasyon Detayları

`EffectMesh.cpp` dosyası, hem 3D model verilerini yükleyen ve saklayan `CEffectMesh` sınıfının hem de bu modellerin bir efekt içinde nasıl davranacağını tanımlayan `CEffectMeshScript` sınıfının implementasyonlarını içerir.

### `CEffectMesh::SEffectMeshData` (CEffectMesh İç Yapısı)

Bu yapı, `CEffectMesh` içinde tek bir alt mesh'e (bir `.mse` dosyasındaki bir geometri parçası) ait verileri tutar.

*   **Bellek Havuzu (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: `SEffectMeshData` örnekleri için `CDynamicPool` kullanarak bellek yönetimi yapar. `Delete` metodu, nesneyi havuza iade etmeden önce içerdiği `EffectFrameDataVector` ve `pImageVector`'ü temizler.
*   **Üyeler**:
    *   `szObjectName`: Mesh nesnesinin adı.
    *   `szDiffuseMapFileName`: Diffuse (yaygın) doku dosyasının adı. Bu, tek bir doku dosyası olabileceği gibi, bir `.ifl` (resim listesi dosyası) ile tanımlanan bir resim dizisi de olabilir.
    *   `EffectFrameDataVector (std::vector<TEffectFrameData>)`: Her bir animasyon karesi için vertex animasyon verilerini içeren bir vektör.
    *   `pImageVector (std::vector<CGraphicImage*>)`: Kullanılan dokular için `CGraphicImage` kaynaklarına işaretçiler tutan bir vektör.

### `CEffectMesh` Sınıfı (Mesh Veri Kaynağı)

Bu sınıf, bir `.mse` dosyasından 3D model geometrisini, animasyonunu ve doku bilgilerini yükleyen ve yöneten bir kaynak (`CResource`) sınıfıdır.

*   **Erişimciler (Accessors)**:
    *   `GetFrameCount()`: Mesh animasyonunun toplam kare sayısını (`m_iFrameCount`) döndürür.
    *   `GetMeshCount()`: Dosyadaki alt mesh sayısını (`m_pEffectMeshDataVector` boyutu) döndürür.
    *   `GetMeshDataPointer(DWORD dwMeshIndex)`: Belirtilen alt mesh indeksi için `SEffectMeshData` işaretçisi döndürür.
    *   `GetTextureVectorPointer(DWORD dwMeshIndex)` / `GetTextureVectorReference(DWORD dwMeshIndex)`: Belirli bir alt mesh'in doku listesine (`pImageVector`) işaretçi veya referans döndürür.

*   **RTTI (`Type`, `OnIsType`)**: `CResource`'dan miras alınan temel Çalışma Zamanı Tip Bilgisi (RTTI) işlevlerini uygular.

*   **`OnLoad(int iSize, const void* c_pvBuf)` (Yükleme Mantığı)**:
    *   Bir bellek tamponundan (genellikle kaynak yöneticisi tarafından yüklenen bir `.mse` dosyasının içeriği) mesh verilerini yükleyen ana fonksiyondur.
    *   Dosya sürümünü belirlemek için bir başlık (header) string'i okur.
    *   İki farklı dosya formatını destekler:
        *   Başlık "EffectData" ise: `__LoadData_Ver001()` çağrılır.
        *   Başlık "MDEData002" ise: `__LoadData_Ver002()` çağrılır.
    *   Başarılı yükleme durumunda `m_isData` bayrağını `true` yapar.

*   **`__LoadData_Ver002(int iSize, const BYTE* c_pbBuf)` (Versiyon 2 Yükleyici)**:
    *   Geometri sayısını (`m_iGeomCount`) ve kare sayısını (`m_iFrameCount`) okur.
    *   `m_pEffectMeshDataVector` vektörünü `m_iGeomCount` boyutuna ayarlar.
    *   Her bir geometri (alt mesh `n`) için:
        *   Yeni bir `SEffectMeshData` (`pMeshData`) ayırır.
        *   `szObjectName` (32 byte) ve `szDiffuseMapFileName` (128 byte) okur.
        *   `pMeshData->EffectFrameDataVector`'ü `m_iFrameCount` boyutuna ayarlar.
        *   Her bir kare `i` için:
            *   `TEffectFrameData& rFrameData` referansını alır.
            *   `byChangedFrame` (BYTE), `fVisibility` (float), `dwVertexCount` (köşe noktası sayısı), `dwIndexCount` (indeks sayısı), `dwTextureVertexCount` (doku köşe noktası sayısı) okur.
            *   Ham köşe pozisyonlarını, köşe indekslerini, doku koordinatlarını ve doku indekslerini geçici vektörlere (`v3VertexVector`, `iIndexVector` vb.) okur.
            *   Son `rFrameData.PDTVertexVector (std::vector<TPTVertex>)` vektörünü oluşturur:
                *   `dwIndexCount` kadar döner. Her bir çıktı köşe noktası için:
                    *   Pozisyonu `v3VertexVector`'den `iIndexVector[j]` kullanarak alır.
                    *   UV koordinatını `v3TextureVertexVector`'den `iTextureIndexVector[j]` kullanarak alır.
                    *   `rVertex.position` ve `rVertex.texCoord`'a atar.
                    *   **Önemli**: V doku koordinatını ters çevirir: `rVertex.texCoord.y *= -1;`. Bu, farklı UV koordinat sistemleri (örneğin, DirectX ve modelleme araçları) arasındaki uyumu sağlamak için yaygın bir işlemdir.
        *   Alt mesh için doku yükleme (`pMeshData->pImageVector`):
            *   `szDiffuseMapFileName` dosyasının bir `.ifl` dosyası (animasyonlu dokular için resim listesi) olup olmadığını belirler.
            *   Eğer `.ifl` ise:
                *   `.ifl` dosyasının içeriğini almak için `CEterPackManager` kullanır.
                *   `.ifl` dosyasını ayrıştırmak için `CMemoryTextFileLoader` kullanır.
                *   `.ifl` dosyasındaki her satır (dosya adı) için tam doku yolunu oluşturur ve `CGraphicImage*` almak için `CResourceManager` kullanır, ardından bunu `pMeshData->pImageVector`'e ekler.
            *   Değilse (tek bir doku dosyasıysa):
                *   `szDiffuseMapFileName` için `CGraphicImage*` almak üzere `CResourceManager` kullanır ve `pMeshData->pImageVector`'e ekler.
        *   Yüklenen `pMeshData`'yı `m_pEffectMeshDataVector[n]`'e atar.

*   **`__LoadData_Ver001(int iSize, const BYTE* c_pbBuf)` (Versiyon 1 Yükleyici)**:
    *   Versiyon 2'ye benzer bir yapıya sahiptir ancak verilerin okunma ve yapılandırılma şeklinde farklılıklar vardır; özellikle köşe noktası bileşenleri (normaller, renkler farklı şekilde işlenebilir veya olmayabilir).
    *   `m_iGeomCount`, `m_iFrameCount` okur.
    *   Geometriler ve kareler arasında döngü yapar.
    *   `byChangedFrame`, `fVisibility`, `dwVertexCount`, `dwIndexCount` okur.
    *   Eğer `byChangedFrame` doğruysa doğrudan `TPDTVertex` verisini okur, aksi takdirde bir önceki karenin verilerinden (`PDTVertexVector`) kopyalar. Bu, Versiyon 1'in tam köşe noktası verilerini yalnızca anahtar kareler (keyframes) için saklayabileceğini gösterir.
    *   Doku yüklemesi Versiyon 2'ye benzerdir (hem `.ifl` hem de tek doku dosyalarını destekler).

*   **Diğer `CEffectMesh` Metodları**:
    *   `GetMeshElementPointer()`: Bir `TEffectMeshData**` almak için yardımcı bir fonksiyondur.
    *   Kurucu/Yıkıcı: `m_isData`'nın başlatılmasını ve `m_pEffectMeshDataVector`'ün temizlenmesini yönetir.
    *   `OnClear()`: Verileri serbest bırakmak için yıkıcı tarafından çağrılır.
    *   `OnDestroy()`: Ayrıca `OnClear()`'ı çağırır.

### `CEffectMeshScript` Sınıfı (Meshler için Efekt Elemanı)

Bu sınıf, bir mesh kaynağının bir efekt içinde nasıl davranacağını (görünürlük, animasyon, renklendirme vb.) tanımlayan script tabanlı özellikleri yönetir.

*   **Bellek Havuzu (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: `CEffectMeshScript` örnekleri için `CDynamicPool` kullanarak bellek yönetimi yapar. `Delete` metodu, `m_MeshDataVector`'ü temizler.

*   **Script Yükleme (`OnLoadScript`)**:
    *   Mesh script özelliklerini okumak için `CTextFileLoader` kullanır.
    *   `MeshFileName` (kullanılacak `.mse` dosyasının adı) okur.
    *   Mesh animasyon özelliklerini okur: `MeshAnimationLoopEnable`, `MeshAnimationLoopCount`, `MeshAnimationFrameDelay`.
    *   Alt mesh sayısını (`dwMeshCount`) okur ve `m_MeshDataVector`'ü yeniden boyutlandırır.
    *   Her bir alt mesh için:
        *   `BillboardType` okur.
        *   Karıştırma (blending) özelliklerini okur: `BlendingEnable`, `BlendingSrcType`, `BlendingDestType`.
        *   `ColorOperationType` ve `ColorFactor` okur.
        *   Doku animasyon özelliklerini okur: `TextureAnimationLoopEnable`, `TextureAnimationFrameDelay`, `TextureAnimationStartFrame`.
        *   `TimeEventAlpha` tablosunu okumak için `GetTokenTimeEventFloat` kullanır.

*   **Erişimciler (Accessors)**:
    *   Belirli bir alt mesh indeksi (`dwMeshIndex`) için `m_MeshDataVector`'den özellikleri almak üzere bir dizi `Get*()` metodu sunar:
        *   `GetBillboardType()`, `isBlendingEnable()`, `GetBlendingSrcType()`, `GetBlendingDestType()`, `isTextureAlphaEnable()` (`TimeEventAlpha`'nın boş olup olmadığını kontrol eder), `GetColorOperationType()`, `GetColorFactor()`, `GetTimeTableAlphaPointer()`, `isTextureAnimationLoop()`, `GetTextureAnimationFrameDelay()`, `GetTextureAnimationStartFrame()`.

*   **Diğer `CEffectMeshScript` Metodları**:
    *   `ReserveMeshData(DWORD dwMeshCount)`: `m_MeshDataVector`'ü yeniden boyutlandırır.
    *   `CheckMeshIndex(DWORD dwMeshIndex)`: Bir mesh indeksinin geçerliliğini kontrol eder.
    *   `GetMeshDataPointer(DWORD dwMeshIndex, TMeshData** ppMeshData)`: Bir alt mesh için `TMeshData` işaretçisi alır.
    *   Kurucu: `Clear()` metodunu çağırır.
    *   `Clear()`: Script özelliklerini varsayılan değerlere sıfırlar ve `m_MeshDataVector`'ü temizler.
    *   `OnClear()`: Dahili `Clear()` metodunu çağırır.
    *   `OnIsData()`: `m_strMeshFileName`'in boş olup olmadığını kontrol eder.

Bu dosya, hem ham mesh verilerinin düşük seviyeli yüklenmesinden ve saklanmasından (`CEffectMesh`) hem de bu mesh'in bir efektin parçası olarak nasıl davrandığını kontrol eden daha üst seviye script özelliklerinden (`CEffectMeshScript`) sorumludur. Bu ayrım, tek bir mesh kaynağının birden fazla efekt içinde farklı davranışlarla kullanılmasına olanak tanır.

## `EffectMeshInstance.cpp` Implementasyon Detayları

`EffectMeshInstance.cpp`, bir `CEffectMeshScript` ve dolayısıyla bir `CEffectMesh` verisine dayanarak oyun dünyasında aktif olan ve canlandırılan bir 3D mesh efektini temsil eden `CEffectMeshInstance` sınıfının metodlarını uygular.

*   **Bellek Yönetimi (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: `CEffectMeshInstance` nesneleri için standart `CDynamicPool` kullanımı. `Delete` metodu, nesneyi havuzuna iade etmeden önce örneğin kendi `Destroy()` metodunu çağırır.

*   **`isActive()`**: Mesh örneğinin aktif olup olmadığını belirler.
    *   Temel sınıf (`CEffectElementBaseInstance::isActive()`) aktif değilse `false` döndürür.
    *   Ana mesh animasyon kontrolcüsü (`m_MeshFrameController`) aktif değilse `false` döndürür.
    *   `m_TextureInstanceVector` (alt meshlerin doku animasyonları) üzerinde döner. Eğer mevcut ana mesh animasyon karesi için herhangi bir alt meshin `TextureFrameController`'ı aktifse `true` döndürür.
    *   Aksi halde `false` döndürür. Bu, mesh örneğinin yalnızca temel sınıfı aktifse, vertex animasyonu aktifse VE mevcut vertex karesi için en az bir doku animasyonu aktifse aktif olduğu anlamına gelir.

*   **`OnUpdate(float fElapsedTime)`**: Örneğin durumunu günceller.
    *   `!isActive()` ise hemen `false` döndürür.
    *   Eğer `m_MeshFrameController` aktifse, onu `fElapsedTime` ile günceller.
    *   `m_TextureInstanceVector` üzerinde döner. Her bir doku örneği için, eğer `TextureFrameController`'ı mevcut ana mesh karesi için aktifse, o kontrolcüyü günceller.
    *   Her zaman `true` döndürür gibi görünmektedir (eğer başlangıçta aktifse); sonlandırma `isActive()` kontrolü ile yapılır.

*   **`OnRender()`**: Aktif olan tüm alt meshleri render eder.
    *   `!isActive()` ise geri döner.
    *   `m_roMesh` referansından `CEffectMesh*` işaretçisini alır.
    *   `pEffectMesh->GetMeshCount()` kadar (her bir alt mesh için `i` indeksiyle) döner:
        *   Mevcut alt mesh için `TextureFrameController`'ı alır. Eğer bu kontrolcü, mevcut ana mesh animasyon karesi için aktif değilse, bu alt mesh atlanır.
        *   `m_pMeshScript`'ten `iBillboardType`'ı alır.
        *   `m_matWorld` (dünya matrisi) birim matris olarak başlatılır.
        *   **Billboard Mantığı**:
            *   `MESH_BILLBOARD_TYPE_ALL`: Kameraya bakan bir billboard matrisi oluşturur, ek olarak X ekseni etrafında 90 derece döndürülür (genellikle XY düzlemlerinin kameraya bakması için).
            *   `MESH_BILLBOARD_TYPE_Y`: Y ekseni etrafında kameraya bakacak şekilde bir billboard matrisi oluşturur (sadece Y ekseninde döner). Tersine çevrilmiş kamera görüntü matrisinin ilgili kısımlarını kopyalar.
            *   `MESH_BILLBOARD_TYPE_MOVE`: Mesh'in yerel Y eksenini (0, -1, 0) hareket yönüyle (mevcut pozisyon - son pozisyon) hizalamak için bir rotasyon hesaplar. `SafeRotationNormalizedArc` fonksiyonunu kullanır.
        *   **Karıştırma (Blending)**: `m_pMeshScript`'ten alınan `isBlendingEnable()`, `GetBlendingSrcType()`, `GetBlendingDestType()` değerlerine göre alt mesh için `D3DRS_ALPHABLENDENABLE`, `D3DRS_SRCBLEND`, `D3DRS_DESTBLEND` durumlarını ayarlar.
        *   **Dünya Matrisi Oluşturma**:
            *   Mesh elemanının mevcut yerel pozisyonunu (`Position`) `m_pMeshScript->GetPosition(m_fLocalTime, ...)` ile alır.
            *   `m_v3MeshScale` kullanarak bir ölçekleme matrisi (`matTemp`) oluşturur.
            *   Yerel `Position` değerini, `m_matWorld`'ün (zaten billboard rotasyonunu içerir) öteleme kısmına ekler.
            *   `m_matWorld`'ü ölçekleme matrisi (`matTemp`) ile çarpar.
            *   Son olarak, `m_matWorld`'ü `*mc_pmatLocal` (ana efekt örneğinin global matrisi) ile çarpar.
            *   Son `m_matWorld` matrisini `STATEMANAGER.SetTransform(D3DTS_WORLD, ...)` ile ayarlar.
        *   **Renk ve Alfa**:
            *   `m_pMeshScript`'ten `ColorOperationType` ve `ColorFactor` alır; `D3DTSS_COLOROP` ve `D3DRS_TEXTUREFACTOR` (ColorFactor ile) ayarlar.
            *   `TimeEventAlpha` tablosunu alır. Eğer mevcut ve boş değilse, `GetTimeEventBlendValue` kullanarak `fAlpha` değerini hesaplar.
        *   **Alt Mesh'in Render Edilmesi**:
            *   Mevcut alt mesh ve mevcut ana mesh animasyon karesi için `CEffectMesh::TEffectMeshData* pMeshData` ve `CEffectMesh::TEffectFrameData& rFrameData` alır.
            *   `rTextureFrameController`'dan mevcut doku animasyon karesini (`dwcurTextureFrame`) alır.
            *   Eğer `dwcurTextureFrame` geçerliyse, `m_TextureInstanceVector[i].TextureInstanceVector`'den `CGraphicImageInstance*` alır ve `STATEMANAGER.SetTexture()` ile dokuyu ayarlar.
            *   Script tabanlı alfa (`fAlpha`) ile mesh karesinin kendi görünürlüğünü (`rFrameData.fVisibility`) birleştirir ve `Color.a` değerine atar. Bu son renk ile `D3DRS_TEXTUREFACTOR`'ü günceller.
            *   Vertex shader FVF'sini `D3DFVF_XYZ | D3DFVF_TEX1` (TPTVertex) olarak ayarlar.
            *   `rFrameData.PDTVertexVector`'deki vertexleri kullanarak `STATEMANAGER.DrawPrimitiveUP(D3DPT_TRIANGLELIST, ...)` ile alt mesh'i çizer.

*   **`OnSetDataPointer(CEffectElementBase* pElement)`**: Örneği `CEffectMeshScript` verileriyle başlatır.
    *   `pElement`'i `CEffectMeshScript*` türüne dönüştürür ve `m_pMeshScript`'e atar.
    *   `pMesh->GetMeshFileName()` ile mesh dosya adını alır.
    *   `CResourceManager` kullanarak `CEffectMesh*` kaynağını alır ve `m_pEffectMesh` ile `m_roMesh` (referans sayımlı işaretçi) üyelerine atar. Mesh kaynağı bulunamazsa geri döner.
    *   `m_MeshFrameController`'ı başlatır:
        *   Maksimum kare sayısını, kare süresini, döngü bayrağını, döngü sayısını `m_pEffectMesh` ve `m_pMeshScript`'ten alarak ayarlar.
    *   `m_TextureInstanceVector`'ü başlatır (mesh sayısına göre yeniden boyutlandırır):
        *   Her bir alt mesh `j` için:
            *   Alt mesh için `CEffectMeshScript::TMeshData* pMeshData` alır.
            *   Bu alt mesh için `m_pEffectMesh`'ten doku vektörünü (`std::vector<CGraphicImage*>* pTextureVector`) alır.
            *   `m_TextureInstanceVector[j].TextureFrameController`'ı, `pTextureVector`'deki doku sayısı (maksimum kare), `pMeshData`'dan kare gecikmesi, döngü bayrağı ve başlangıç karesi ile başlatır.
            *   `m_TextureInstanceVector[j].TextureInstanceVector`'ü doldurur:
                *   `pTextureVector`'deki her `CGraphicImage*` için, havuzundan bir `CGraphicImageInstance` ayırır, resim işaretçisini ayarlar ve bu vektöre ekler.

*   **Temizleme için Yardımcı Fonksiyonlar (`CEffectMeshInstance_DeleteImageInstance`, `CEffectMeshInstance_DeleteTextureInstance`)**:
    *   Bunlar, `Clear()` ve `OnDestroy()` metodlarında `std::for_each` ile kullanılan global fonksiyonlardır. Havuzdaki `CGraphicImageInstance` nesnelerini doğru bir şekilde serbest bırakmak ve `TTextureInstance` yapılarındaki `TextureInstanceVector`'leri temizlemek için kullanılırlar.

*   **`OnInitialize()`**: Boştur. Başlatma işlemleri `OnSetDataPointer` içinde yapılır.

*   **`OnDestroy()`**: `Clear()` metodunu çağırır.

*   **`Clear()`**:
    *   `m_pEffectMesh`'i `NULL` yapar ve `m_roMesh`'i temizler.
    *   `m_TextureInstanceVector` üzerinde döner. Her bir `TTextureInstance` için:
        *   İçindeki `TextureInstanceVector`'deki tüm `CGraphicImageInstance`'ları serbest bırakmak için `std::for_each` ve `CEffectMeshInstance_DeleteImageInstance` kullanır.
        *   `TextureInstanceVector`'ü temizler.
    *   `m_TextureInstanceVector`'ün kendisini temizler.

*   **Kurucu (`CEffectMeshInstance()`)**: `Clear()` metodunu çağırır ve `m_v3MeshScale`'i (1,1,1) olarak ayarlar.

*   **Yıkıcı (`~CEffectMeshInstance()`)**: `Clear()` metodunu çağırır.

Bu sınıf, bir `CEffectMesh` kaynağında tanımlanan potansiyel olarak birden fazla alt meshin animasyonunu (hem vertex hem de doku) ve render edilmesini, bir `CEffectMeshScript` içinde ayarlanan davranışsal özelliklere göre düzenler. Her bir alt mesh için billboard, karıştırma ve renk/alfa modülasyonunu yönetir.

## `SimpleLightData.cpp` Implementasyon Detayları

`SimpleLightData.cpp`, bir efektin parçası olarak kullanılacak basit bir nokta ışığının statik özelliklerini tanımlayan `CLightData` sınıfının implementasyonunu içerir.

*   **Bellek Havuzu (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: `CLightData` nesneleri için standart `CDynamicPool` kullanımı. `Delete` metodu, nesneyi havuza iade etmeden önce `Clear()` metodunu çağırır.

*   **`OnClear()`**: `CEffectElementBase`'den gelen sanal `Clear` fonksiyonunu uygular.
    *   Işık özelliklerini varsayılan değerlere sıfırlar:
        *   `m_fMaxRange` = 300.0f
        *   `m_cAmbient` = (0.5, 0.5, 0.5, 1.0)
        *   `m_cDiffuse` = (0.0, 0.0, 0.0, 1.0) (Varsayılan diffuse rengi siyah?)
        *   `m_fDuration` = 1.0f
        *   `m_fAttenuation0` = 0.0f, `m_fAttenuation1` = 0.1f, `m_fAttenuation2` = 0.0f (Varsayılan olarak ağırlıklı lineer zayıflama)
        *   `m_bLoopFlag` = false, `m_iLoopCount` = 0
    *   `m_TimeEventTableRange` vektörünü temizler.

*   **`GetRange(float fTime, float& rRange)`**: Belirli bir zamanda ışığın menzilini hesaplar.
    *   Eğer `m_TimeEventTableRange` boşsa, menzil basitçe `m_fMaxRange` olur (>= 0 olarak sınırlandırılır).
    *   Aksi takdirde, zaman tablosundan enterpolasyonla değeri (muhtemelen 0.0 ile 1.0 arasında) almak için `GetTimeEventBlendValue(fTime, m_TimeEventTableRange, &rRange)` çağrılır.
    *   Bu enterpolasyonlu değer, nihai menzili elde etmek için `m_fMaxRange` ile çarpılır.
    *   Sonuç `rRange`, >= 0 olacak şekilde sınırlandırılır.
    *   Manuel lineer enterpolasyon mantığını gösteren yorum satırına alınmış bir blok bulunur; bu mantık artık `GetTimeEventBlendValue` yardımcı fonksiyonu tarafından ele alınmaktadır.

*   **`OnIsData()`**: Sanal fonksiyonu uygular. Basitçe `true` döndürür, bu da ışık verilerinin yüklendikten sonra her zaman geçerli kabul edildiğini gösterir (veya doğrulama yükleme sırasında gerçekleşir).

*   **`OnLoadScript(CTextFileLoader& rTextFileLoader)`**: Sanal script yükleme fonksiyonunu uygular.
    *   `CTextFileLoader`'dan ışık özelliklerini okur:
        *   `duration` (float, varsayılan 1.0f)
        *   `loopflag` (boolean, varsayılan false)
        *   `loopcount` (integer, varsayılan 0)
        *   `ambientcolor` (D3DXCOLOR, zorunlu)
        *   `diffusecolor` (D3DXCOLOR, zorunlu)
        *   `maxrange` (float, zorunlu)
        *   `attenuation0`, `attenuation1`, `attenuation2` (float, zorunlu)
    *   `timeeventrange` yüklemeyi denemek için `GetTokenTimeEventFloat` kullanır. Başarısız olursa veya token mevcut değilse, `m_TimeEventTableRange` temizlenir.
    *   Tüm zorunlu tokenlar bulunursa `true`, aksi takdirde `false` döndürür.

*   **Kurucu (`CLightData()`)**: Varsayılan değerlerle başlatmak için `Clear()` metodunu çağırır.

*   **Yıkıcı (`~CLightData()`)**: Boştur.

*   **`GetDuration()`**: `m_fDuration` için basit bir erişimci.

*   **`InitializeLight(D3DLIGHT8& light)`**: `CLightData` özelliklerine göre bir `D3DLIGHT8` (DirectX 8 ışık yapısı) yapısını doldurur.
    *   `light.Type`'ı `D3DLIGHT_POINT` olarak ayarlar.
    *   `Ambient`, `Diffuse`, `Attenuation0/1/2` kopyalar.
    *   Başlangıç yerel pozisyonunu almak için `GetPosition(0.0f, position)` (`CEffectElementBase`'den miras alınır) çağırır ve `light.Position`'ı ayarlar.
    *   Başlangıç menzilini almak için `GetRange(0.0f, light.Range)` çağırır.

Bu sınıf, basit bir nokta ışığı efektinin statik özelliklerini (renkleri, zayıflaması, süresi, döngü davranışı ve menzilinin zamanla nasıl değişeceği dahil) yüklemekten ve saklamaktan sorumludur. `InitializeLight` metodu, bu özellikleri temel alınan grafik API'sinin (bu durumda DirectX 8) ihtiyaç duyduğu formata çevirmek için uygun bir yol sağlar.

## `SimpleLightInstance.cpp` Implementasyon Detayları

`SimpleLightInstance.cpp`, oyun dünyasında aktif bir nokta ışığı örneğini temsil eden ve `CLightData` ile `CLightManager` arasında köprü görevi gören `CLightInstance` sınıfının metodlarını uygular.

*   **Bellek Havuzu (`ms_kPool`, `New`, `Delete`, `DestroySystem`)**: `CLightInstance` nesneleri için standart `CDynamicPool` kullanımı. `Delete` metodu, `Destroy()` metodunu çağırır.

*   **`OnSetDataPointer(CEffectElementBase* pElement)`**: Örneği `CLightData` verilerini kullanarak başlatır.
    *   Önce mevcut durumu temizlemek için `Destroy()` metodunu çağırır (özellikle yeniden başlatma durumlarında önemlidir).
    *   `pElement`'i `CLightData*` türüne dönüştürür ve `m_pData` üyesine atar.
    *   `m_iLoopCount`'u `m_pData->GetLoopCount()` değerinden alır.
    *   Yerel bir `D3DLIGHT8` yapısı (`Light`) oluşturur.
    *   `m_pData->InitializeLight(Light)` çağrısı ile bu yapıyı doldurur.
    *   `CLightManager::Instance().RegisterLight(LIGHT_TYPE_DYNAMIC, &m_LightID, Light)` çağrısı ile bu ışığı merkezi `CLightManager`'a kaydeder. Yönetici tarafından atanan benzersiz ID, `m_LightID` içinde saklanır.

*   **`OnUpdate(float fElapsedTime)`**: Işığın özelliklerini `CLightManager` içinde günceller.
    *   Temel sınıftan `isActive()` kontrolü yapar. Aktif değilse `Destroy()` çağırır ve `false` döndürür.
    *   **Döngü/Süre Kontrolü**:
        *   `m_fLocalTime`'ın (elemanın başlangıcından beri geçen süre) `m_pData->GetDuration()`'ı aşıp aşmadığını kontrol eder.
        *   Süre aşıldıysa:
            *   Döngünün etkin olup olmadığını (`m_pData->isLoop()`) ve `m_iLoopCount`'un (bir azaltıldıktan sonra) sıfır olup olmadığını kontrol eder.
            *   Döngü devam ediyorsa: `m_fLocalTime`'ı süreyi çıkararak sıfırlar ve `m_iLoopCount`'un (eğer başlangıçta -1 ise) negatif olmamasını sağlar.
            *   Döngü duruyorsa (etkin değilse veya `m_iLoopCount` sıfıra ulaşmışsa): `Destroy()` çağırır, `m_iLoopCount`'u 1 yapar (muhtemelen döngü kontrolüne tekrar girmeyi önlemek için?) ve örneğin bittiğini belirtmek için `false` döndürür.
    *   **Işık Özellik Güncellemesi**:
        *   `CLightManager`'dan `m_LightID` kullanarak gerçek `CLight` nesnesine bir işaretçi (`pLight`) alır.
        *   Eğer `pLight` geçerliyse:
            *   Işığın Ambient ve Diffuse renklerini doğrudan `m_pData`'dan alarak ayarlar (Bu renkler bu implementasyonda zamanla değişmiyor gibi görünmektedir).
            *   Mevcut zaman için hesaplanan menzili almak üzere `m_pData->GetRange(m_fLocalTime, fRange)` çağırır.
            *   Işık yöneticisindeki menzili güncellemek için `pLight->SetRange(fRange)` çağırır.
            *   Zamana göre yerel pozisyonu almak için `m_pData->GetPosition(m_fLocalTime, pos)` çağırır.
            *   Bu `pos` değerini ana efektin matrisi (`mc_pmatLocal`) ile dönüştürür.
            *   Işık yöneticisindeki pozisyonu güncellemek için `pLight->SetPosition(pos.x, pos.y, pos.z)` çağırır.
    *   Örnek hala aktifse `true` döndürür.

*   **`OnRender()`**: Boştur. Işıklar doğrudan render edilmez; etkileri, `OnUpdate` sırasında ayarlanan özellikler kullanılarak grafik motoru tarafından sahne render edilirken uygulanır.

*   **`OnInitialize()`**: Temel sınıf `Initialize` tarafından çağrılan sanal fonksiyon.
    *   `m_LightID`'yi 0 yapar.
    *   `m_dwRangeIndex`'i 0 yapar (bu indeks, `OnUpdate` içindeki yorum satırına alınmış menzil harmanlama mantığıyla ilgili görünmektedir).

*   **`OnDestroy()`**: Temel sınıf `Destroy` tarafından çağrılan sanal fonksiyon.
    *   Eğer `m_LightID` sıfırdan farklıysa (yani bir ışık kaydedilmişse), ışığı yöneticiden kaldırmak için `CLightManager::Instance().DeleteLight(m_LightID)` çağırır.

*   **Kurucu (`CLightInstance()`)**: `Initialize()` metodunu çağırır (bu da `OnInitialize`'ı çağırır).

*   **Yıkıcı (`~CLightInstance()`)**: `Destroy()` metodunu çağırır (bu da `OnDestroy`'u çağırır).

Bu sınıf, statik `CLightData` ile aktif `CLightManager` arasında bir aracı görevi görür. Oluşturulduğunda ışığı kaydeder, ömrü boyunca menzilini ve pozisyonunu zamana ve ana efektin dönüşümüne göre günceller, döngülemeyi yönetir ve yok edildiğinde ışığı sistemden kaldırır.

## `EffectElementBase.cpp` Implementasyon Detayları

`EffectElementBase.cpp`, tüm efekt elemanı **veri** sınıflarının miras aldığı soyut taban sınıfı olan `CEffectElementBase` için ortak işlevselliği uygular.

*   **`GetPosition(float fTime, D3DXVECTOR3& rPosition)`**: Efekt elemanının belirli bir zamandaki (`fTime`) yerel pozisyonunu `m_TimeEventTablePosition` zaman çizelgesine göre hesaplar.
    *   Kenar durumlarını ele alır:
        *   Tablo boşsa (0,0,0) döndürür.
        *   Tabloda tek bir giriş varsa, o girişin pozisyonunu döndürür.
        *   `fTime` ilk girişten önceyse, ilk girişin pozisyonunu döndürür.
        *   `fTime` son girişten sonraysa, son girişin pozisyonunu döndürür.
    *   `fTime`'dan küçük olmayan ilk elemanı sıralı `m_TimeEventTablePosition` içinde verimli bir şekilde bulmak için `std::lower_bound` kullanır.
    *   Bulunan elemana (`result`) ve bir önceki elemana (`rPrev`) işaret eden iterator'lar alır.
    *   Hareket türünü (`iMovingType`) *önceki* anahtar kareden (`rPrevEffectPosition`) alır.
    *   **Enterpolasyon**:
        *   Eğer `MOVING_TYPE_DIRECT` ise: `rPrevEffectPosition.m_vecPosition` ve `rEffectPosition.m_vecPosition` arasında `fTime`'a göre lineer enterpolasyon yapar.
        *   Eğer `MOVING_TYPE_BEZIER_CURVE` ise: `rPrevEffectPosition.m_vecPosition` (başlangıç noktası), `rPrevEffectPosition.m_vecPosition + rPrevEffectPosition.m_vecControlPoint` (kontrol noktası - önceki pozisyona göre tanımlandığına dikkat edin) ve `rEffectPosition.m_vecPosition` (bitiş noktası) kullanarak ikinci dereceden Bezier enterpolasyonu yapar. Enterpolasyon parametresi `ft`'yi hesaplar.
    *   Hesaplanan pozisyonu `rPosition` içine kaydeder.

*   **Yorum Satırına Alınmış Fonksiyonlar**: `isVisible`, `GetAlpha` ve `GetScale` fonksiyonları yorum satırına alınmıştır. Bu, görünürlük, alfa ve ölçek gibi özelliklerin bir noktada temel eleman özellikleri olabileceğini ancak muhtemelen türetilmiş sınıflara (alfa için `CEffectMeshScript` gibi) taşındığını veya farklı şekilde ele alındığını düşündürür.

*   **`isData()`**: Saf sanal `OnIsData()` fonksiyonunu çağıran genel (public) bir sarmalayıcı fonksiyondur.

*   **`Clear()`**: Genel (public) bir sarmalayıcı fonksiyondur.
    *   `m_fStartTime`'ı 0.0f olarak sıfırlar.
    *   Saf sanal `OnClear()` fonksiyonunu çağırır.

*   **`LoadScript(CTextFileLoader& rTextFileLoader)`**: Script yükleme için genel (public) bir sarmalayıcı fonksiyondur.
    *   Opsiyonel `starttime` token'ını okur (varsayılanı 0.0f).
    *   Eğer varsa `timeeventposition` token'ını okur:
        *   `m_TimeEventTablePosition`'ı temizler.
        *   Token vektörünü ayrıştırır. `DIRECT` için üçlü (zaman, tip, x, y, z) veya `BEZIER_CURVE` için altılı (zaman, tip, x, y, z, cx, cy, cz) bekler.
        *   String token'ları float'a dönüştürmek için `atof` kullanır.
        *   `TEffectPosition` yapılarını oluşturur ve bunları `m_TimeEventTablePosition`'a ekler. Tip token'ı tanınmazsa `FALSE` döndürür.
    *   Türetilmiş sınıfların kendi özel verilerini yüklemesine izin vermek için saf sanal `OnLoadScript(rTextFileLoader)` fonksiyonunu çağırır. Türetilmiş sınıfın yükleme sonucunu döndürür.

*   **`GetStartTime()`**: `m_fStartTime` için erişimci (accessor) fonksiyonudur.

*   **Kurucu (`CEffectElementBase()`)**: `m_fStartTime`'ı 0.0f olarak başlatır.

*   **Yıkıcı (`~CEffectElementBase()`)**: Sanal yıkıcıdır (sanal fonksiyonlara sahip temel sınıflar için önemlidir), ancak implementasyonu boştur.

Bu temel sınıf, efekt elemanları için ortak işlevsellik sağlar: bir başlangıç zamanı ofseti, zaman tabanlı yerel pozisyon enterpolasyonu (lineer ve Bezier hareketini destekler) ve türetilmiş sınıflar tarafından uygulanan saf sanal fonksiyonlar aracılığıyla veri yükleme/temizleme için bir çerçeve sunar.

## `EffectElementBaseInstance.cpp` Implementasyon Detayları

`EffectElementBaseInstance.cpp`, tüm efekt elemanı **örnek** sınıflarının miras aldığı soyut taban sınıfı olan `CEffectElementBaseInstance` için ortak yaşam döngüsü yönetimi işlevselliğini uygular.

*   **`Update(float fElapsedTime)`**: Örneği güncellemek için genel (public) sarmalayıcı fonksiyon.
    *   Elemanın başlangıç gecikmesinin (`m_fStartTime`) geçip geçmediğini belirten `m_bStart` bayrağını kontrol eder.
    *   Eğer `m_bStart` doğruysa (gecikme bittiyse):
        *   `fElapsedTime` değerini `m_fElapsedTime` içinde saklar (bu, muhtemelen türetilmiş sınıfların özel kullanımı içindir, çünkü `fElapsedTime` zaten `OnUpdate`'e de geçilir).
        *   `m_fLocalTime`'ı (elemanın *gerçekten* başladığından beri geçen süre) artırır.
        *   Saf sanal `OnUpdate(fElapsedTime)` fonksiyonunu çağırır ve sonucunu (elemanın hala hayatta olup olmadığını belirtir) döndürür.
    *   Eğer `m_bStart` yanlışsa (hala gecikme periyodundaysa):
        *   `m_fRemainingTime`'ı (başlangıçtaki gecikme süresi) `fElapsedTime` kadar azaltır.
        *   Eğer `m_fRemainingTime` sıfıra veya altına düşerse, `m_bStart`'ı `true` yapar.
        *   `true` döndürür (örnek, gecikme süresi boyunca hala "hayatta" kabul edilir).

*   **`Render()`**: Render işlemi için genel (public) sarmalayıcı fonksiyon.
    *   Eğer `m_bStart` yanlışsa (eleman başlangıç gecikmesini geçmediyse) hemen geri döner.
    *   `mc_pmatLocal`'ın (ana matris işaretçisi) NULL olmadığını `assert` ile kontrol eder.
    *   Saf sanal `OnRender()` fonksiyonunu çağırır.

*   **`SetLocalMatrixPointer(const D3DXMATRIX* c_pMatrix)`**: `mc_pmatLocal` işaretçisini ana efektin dünya matrisine ayarlar.

*   **`SetDataPointer(CEffectElementBase* pElement)`**: Veri işaretçisini ayarlamak için genel (public) sarmalayıcı fonksiyon.
    *   İşaretçiyi `m_pBase` içinde saklar.
    *   Mevcut zamanı (`CTimer::Instance().GetCurrentMillisecond()`) `m_dwStartTime` içine kaydeder (Not: Bu üye başka bir yerde kullanılmıyor gibi görünüyor, zamanlama için `m_fLocalTime` kullanılıyor).
    *   **Başlangıç Zamanı Yönetimi**:
        *   `pElement->GetStartTime()` (gecikme değeri) alır ve `m_fRemainingTime` içinde saklar.
        *   Başlangıç zamanı <= 0 ise, `m_bStart`'ı hemen `true` yapar.
        *   Aksi takdirde, `m_bStart`'ı `false` yapar.
    *   Saf sanal `OnSetDataPointer(pElement)` fonksiyonunu çağırır.

*   **Aktif Durum Yönetimi (`isActive`, `SetActive`, `SetDeactive`)**: `m_isActive` bayrağı için basit erişimci (accessor) ve değiştirici (mutator) fonksiyonlar.

*   **Ölçek Ayarlayıcıları (`SetParticleScale`, `SetMeshScale`)**: `m_fParticleScale` ve `m_v3MeshScale` üyelerini ayarlar. Bunlar muhtemelen türetilmiş sınıflar (`CParticleSystemInstance`, `CEffectMeshInstance`) tarafından kullanılır.

*   **`Initialize()`**: Örnek durumunu sıfırlamak için genel (public) fonksiyon.
    *   `mc_pmatLocal`'ı `NULL` yapar.
    *   `m_isActive`'i `true` yapar.
    *   Zaman değişkenlerini (`m_fLocalTime`, `m_dwStartTime`, `m_fElapsedTime`) sıfırlar.
    *   Başlangıç gecikme durumunu (`m_bStart`, `m_fRemainingTime`) sıfırlar.
    *   Ölçek faktörlerini (`m_fParticleScale`, `m_v3MeshScale`) 1.0 yapar.
    *   Saf sanal `OnInitialize()` fonksiyonunu çağırır.

*   **`Destroy()`**: Örneği temizlemek için genel (public) fonksiyon.
    *   Saf sanal `OnDestroy()` fonksiyonunu çağırır.
    *   Durumu sıfırlamak için `Initialize()` fonksiyonunu çağırır (potansiyel olarak havuzdan yeniden kullanıma hazırlamak için).

*   **Kurucu (`CEffectElementBaseInstance()`)**: (Gösterilmiyor, ancak tipik olarak `Initialize()` çağırır).
*   **Yıkıcı (`~CEffectElementBaseInstance()`)**: (Gösterilmiyor, ancak tipik olarak `Destroy()` çağırır veya havuz yönetimine güvenir).

Bu temel sınıf örneği, efekt elemanları için çekirdek yaşam döngüsü yönetimini sağlar: başlangıç gecikmelerini yönetme, yerel zamanı takip etme, aktif durumu yönetme, ana matrise erişim sağlama, ölçek ayarlama ve türetilmiş sınıfların kendi özel mantıkları için uygulaması gereken arayüzü (`OnUpdate`, `OnRender`, `OnInitialize`, `OnDestroy`, `OnSetDataPointer`) tanımlar.

## `Type.cpp` Implementasyon Detayları

`Type.cpp`, `EffectLib` içinde zamanla değişen float değer tablolarını (`TTimeEventTableFloat`) işlemek için yardımcı fonksiyonlar içerir.

*   **`GetTokenTimeEventFloat(CTextFileLoader& rTextFileLoader, const char* c_szKey, TTimeEventTableFloat* pTimeEventTableFloat)`**:
    *   Bu yardımcı fonksiyon, bir `CTextFileLoader`'dan belirli bir anahtar (`c_szKey`) ile ilişkilendirilmiş bir `TTimeEventTableFloat` (her biri `float m_fTime` ve `float m_Value` içeren `TTimeEventTypeFloat` yapılarından oluşan bir `std::vector`) okumak için tasarlanmıştır.
    *   Metin dosyası yükleyicisinden `c_szKey` anahtarıyla ilişkilendirilmiş string token vektörünü almaya çalışır. Token vektörü bulunamazsa `FALSE` döndürür.
    *   Çıktı tablosunu (`pTimeEventTableFloat`) temizler.
    *   Çıktı tablosunu, token vektörünün boyutunun yarısı kadar yeniden boyutlandırır (tokenların zaman/değer çiftleri halinde geldiğini varsayarak).
    *   Token vektöründe ikişer ikişer (`i += 2`) ilerleyerek döner.
    *   Çıktı tablosundaki her giriş için ilk token'ı `m_fTime`'a ve ikinci token'ı `m_Value`'ya dönüştürmek için `atof` (ASCII'den float'a) kullanır.
    *   Başarılı ayrıştırma durumunda `TRUE` döndürür.

*   **`InsertItemTimeEventFloat(TTimeEventTableFloat* pTable, float fTime, float fValue)`**:
    *   Bu fonksiyon, yeni bir zaman-değer çiftini mevcut ve muhtemelen zamana göre *sıralı* bir `TTimeEventTableFloat` içine ekler.
    *   Eklenecek olan `fTime` değerinden daha büyük `m_fTime` değerine sahip ilk elemanı bulana kadar tablo (`pTable`) içinde döner.
    *   Verilen `fTime` ve `fValue` ile yeni bir `TTimeEventTypeFloat` yapısı (`TimeEvent`) oluşturur.
    *   Yeni elemanı iterator pozisyonundan (`itor`) *önce* eklemek için `pTable->insert(itor, TimeEvent)` kullanır, böylece tablonun zamana göre sıralı düzenini korur.

Bu dosya, `EffectLib` genelinde zamanla değişen kayan noktalı özellikleri temsil etmek için yaygın olarak kullanılan `TTimeEventTableFloat` yapısını işlemek için özel yardımcı program fonksiyonları sağlar. `GetTokenTimeEventFloat`, bu tabloları script dosyalarından yüklemeyi yönetirken, `InsertItemTimeEventFloat` ise `GetTimeEventBlendValue` gibi enterpolasyon fonksiyonları için kritik olan zaman sıralı düzeni koruyarak yeni anahtar karelerin eklenmesine olanak tanır.

## `FrameController.cpp` Implementasyon Detayları

`FrameController.cpp`, kare tabanlı animasyonların (örneğin, mesh vertex animasyonları veya doku animasyonları) zamanlamasını ve ilerlemesini yöneten `CFrameController` sınıfının metodlarını uygular.

*   **`Update(float fElapsedTime)`**: Kare kontrolcüsünün durumunu geçen süreye göre günceller.
    *   `m_fLastFrameTime`'ı (bir sonraki kareye kalan süre) `fElapsedTime` kadar azaltır.
    *   En fazla 20 iterasyonla sınırlı bir `for` döngüsü kullanır. Bu, `fElapsedTime` çok büyükse veya `m_fFrameTime` sıfıra yakınsa potansiyel sonsuz döngüleri önlemek veya gerekirse tek bir güncelleme çağrısında birden fazla kare ilerlemesini işlemek için bir güvenlik önlemi veya optimizasyon görevi görür.
    *   Döngü içinde:
        *   `m_fLastFrameTime`'ın sıfırın altına düşüp düşmediğini (yani bir kare geçişinin zamanının gelip gelmediğini) kontrol eder.
        *   Eğer evet ise:
            *   `m_fFrameTime`'ı `m_fLastFrameTime`'a geri ekler (*sonraki* kare için zamanlayıcıyı ayarlar).
            *   `m_dwcurFrame`'ı (mevcut kare) artırır.
            *   `m_dwcurFrame`'ın `m_dwMaxFrame`'a ulaşıp ulaşmadığını veya aşıp aşmadığını kontrol eder:
                *   Eğer evet ise, döngü koşullarını kontrol eder:
                    *   Eğer `m_isLoop` doğruysa ve `m_iLoopCount` (bir azaltıldıktan sonra) sıfır değilse:
                        *   Eğer başlangıçta sonsuz (-1) idiyse `m_iLoopCount`'un negatif kalmamasını sağlar.
                        *   `m_dwcurFrame`'ı 0'a sıfırlar (Not: Başlık dosyasındaki veya `m_dwStartFrame`'ın amacından farklı olarak kod burada 0 kullanıyor olabilir).
                    *   Değilse (döngü devre dışıysa veya döngü sayısı bittiyse):
                        *   `m_iLoopCount`'u 1 yapar (keyfi görünüyor, belki yeniden girişi önlemek için?).
                        *   `m_dwcurFrame`'ı 0'a sıfırlar.
                        *   `m_isActive`'i `false` yapar.
                        *   Hemen geri döner (bu güncellemede daha fazla kare ilerlemesini işlemeyi durdurur).
        *   Eğer `m_fLastFrameTime` hala negatif değilse, iç döngüden `break` ile çıkar (bu güncellemede daha fazla kare geçişine gerek yoktur).

*   **Ayarlayıcılar/Alıcılar (Setters/Getters)**:
    *   `SetCurrentFrame(DWORD dwFrame)`: `m_dwcurFrame`'ı ayarlar.
    *   `GetCurrentFrame()`: `m_dwcurFrame`'ı (`BYTE` olarak) döndürür.
    *   `SetMaxFrame(DWORD dwMaxFrame)`: `m_dwMaxFrame`'ı ayarlar.
    *   `SetFrameTime(float fTime)`: `m_fFrameTime`'ı ayarlar ve ayrıca `m_fLastFrameTime`'ı `fTime`'a sıfırlar.
    *   `SetStartFrame(DWORD dwStartFrame)`: `m_dwStartFrame`'ı ayarlar. (Not: Gösterilen `Update` döngüsünde şu anda kullanılmıyor).
    *   `SetLoopFlag(BOOL bFlag)`: `m_isLoop`'u ayarlar.
    *   `SetLoopCount(int iLoopCount)`: `m_iLoopCount`'u ayarlar.
    *   `SetActive(BOOL bFlag)`: `m_isActive`'i ayarlar.

*   **`isActive(DWORD dwMainFrame)`**: Potansiyel bir `dwMainFrame` bağımlılığını dikkate alarak kontrolcünün aktif olup olmadığını kontrol eder.
    *   Eğer `dwMainFrame` (muhtemelen ana mesh animasyonu gibi bir üst kontrolcünün karesi) `m_dwStartFrame`'dan küçükse `false` döndürür. Bu, bu kare kontrolcüsünün (örneğin bir doku animasyonunun) başlangıcını ana animasyonun belirli bir karesine kadar geciktirmeye olanak tanır.
    *   Aksi takdirde, `m_isActive` değerini döndürür.

*   **`Clear()`**: Kontrolcü durumunu sıfırlar.
    *   `m_isActive`'i `true` yapar.
    *   `m_dwcurFrame`'ı 0 yapar.
    *   `m_fLastFrameTime`'ı 0.0f yapar.

*   **Kurucu (`CFrameController()`)**: Tüm üyeleri varsayılan değerlere (etkin olmayan döngü, 0 kare, 0 zaman vb.) başlatır, ancak başlangıçta `m_isActive`'i `true` yapar.

*   **Yıkıcı (`~CFrameController()`)**: Boştur.

`CFrameController`, kare tabanlı animasyonları yönetir. Geçen süreye ve tanımlanmış bir kare süresine (`m_fFrameTime`) göre mevcut kareyi takip eder. Belirli bir sayıda (veya sonsuz) döngüyü destekler ve yalnızca belirli bir ana kareden (`m_dwStartFrame` ile `isActive` içinde kullanılır) sonra başlamak üzere geciktirilebilir. `Update` döngüsü, önemli bir süre geçtiyse tek bir çağrıda potansiyel olarak birden fazla kare ilerlemesini yönetir.

## `EffectUpdateDecorator.cpp` Implementasyon Detayları

`EffectUpdateDecorator.cpp`, `CParticleInstance` nesnelerine dinamik olarak eklenen fiziksel davranışları (hava direnci, yerçekimi, dönüş) uygulayan dekoratör sınıflarının temel işlevlerini içerir.

*   **Namespace (`NEffectUpdateDecorator`)**: Tüm kod bu namespace içindedir.
*   **Odak**: Bu dosya, başlık dosyasında tanımlanan fizik tabanlı dekoratörler (`CAirResistanceDecorator`, `CGravityDecorator`, `CRotationDecorator`) için `__Clone` ve `__Excute` metodlarını uygular. Zaman olayına dayalı dekoratörler (`CTimeEventDecorator` şablonu ve onun örnekleri olan `CScaleValueDecorator`, `CColorAllDecorator` gibi) ve doku animasyon dekoratörleri (`CTextureAnimation*Decorator`), daha basit veya şablon tabanlı oldukları için muhtemelen tamamen başlık dosyasında (`EffectUpdateDecorator.h`) inline olarak uygulanmıştır.

*   **`CAirResistanceDecorator`**:
    *   `__Clone(CParticleInstance* pfi, CParticleInstance* pi)`:
        *   Kaynak partiküldeki (`pfi`) `m_fAirResistance` değerini hedef partiküle (`pi`) kopyalar.
        *   *Yeni* bir `CAirResistanceDecorator` örneği ayırır ve döndürür. Not: Bu, bir bellek havuzu kullanmaz; bu durum, bu dekoratörlerin partiküllerden daha az sıklıkla oluşturulup yok edildiği varsayımıyla bir tasarım tercihi veya bir tutarsızlık olabilir.
    *   `__Excute(const CDecoratorData& d)`:
        *   Partikülün hızını (`d.pInstance->m_v3Velocity`) `(1.0f - d.pInstance->m_fAirResistance)` ile ölçekleyerek hava direncini uygular. Bu basit bir lineer sürüklenme modelidir.

*   **`CGravityDecorator`**:
    *   `__Clone(CParticleInstance* pfi, CParticleInstance* pi)`:
        *   `m_fGravity` değerini `pfi`'den `pi`'ye kopyalar.
        *   *Yeni* bir `CGravityDecorator` örneği ayırır ve döndürür.
    *   `__Excute(const CDecoratorData& d)`:
        *   Partikülün hızının Z bileşenini (`d.pInstance->m_v3Velocity.z`) `(d.pInstance->m_fGravity * d.fElapsedTime)` kadar azaltarak yerçekimini uygular. Z ekseninin dikey eksen olduğunu varsayar.

*   **`CRotationDecorator`**:
    *   `__Clone(CParticleInstance* pfi, CParticleInstance* pi)`:
        *   `m_fRotationSpeed` değerini `pfi`'den `pi`'ye kopyalar.
        *   *Yeni* bir `CRotationDecorator` örneği ayırır ve döndürür.
    *   `__Excute(const CDecoratorData& d)`:
        *   Partikülün dönüş açısını (`d.pInstance->m_fRotation`) `(d.pInstance->m_fRotationSpeed * d.fElapsedTime)` kadar artırarak dönüşü uygular.

`__Clone` metodları, yeni bir partikül örneği için bir dekoratör zincirinin kopyalanması gerektiğinde önemlidir; ilgili durumun (örneğin, o partikül için belirli yerçekimi veya hava direnci değeri, potansiyel olarak zincirde daha önceki bir zaman olay dekoratörü tarafından ayarlanmış olabilir) kopyalanmasını sağlar. `__Excute` metodları, her bir dekoratör tarafından uygulanan gerçek kare başına mantığı içerir.

## `StdAfx.cpp` Implementasyon Detayları

*   **Amaç:** Bu C++ dosyası, `StdAfx.h` başlık dosyasını dahil ederek ön derlenmiş başlık (`.pch`) dosyasının oluşturulmasını tetikler.
*   **İçerik:** Genellikle sadece `#include "StdAfx.h"` satırını içerir. Derleyici bu `.cpp` dosyasını işlerken, `StdAfx.h` içinde listelenen tüm başlıkları derler ve bir `.pch` dosyası oluşturur.
*   **Kullanım Alanı:** Doğrudan oyun mantığında bir rolü yoktur. Temel amacı, Visual Studio gibi C++ derleyicilerinin ön derlenmiş başlık özelliğini kullanarak projenin genel derleme süresini kısaltmaktır. Projedeki diğer `.cpp` dosyaları derlenirken, `StdAfx.h'ta zaten işlenmiş olan başlıklar için bu `.pch` dosyası kullanılır.

---