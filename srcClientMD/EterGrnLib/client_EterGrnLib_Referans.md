# EterGrnLib Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/EterGrnLib/` klasöründe bulunan `EterGrnLib` modülünün amacını, içerdiği temel bileşenleri ve bu bileşenlerin işlevlerini açıklamaktadır.

`EterGrnLib`, Metin2 istemcisinin 3D grafiklerinin temel taşlarından biridir ve büyük olasılıkla Granny 3D animasyon kütüphanesiyle entegrasyonu sağlar. Bu modül, 3D modellerin (.gr2 formatı), animasyonların, materyallerin, mesh'lerin ve sahne içindeki "şeylerin" (things) yüklenmesi, yönetilmesi, güncellenmesi ve render edilmesi gibi kritik görevleri üstlenir. Karakterler, yaratıklar, ortam nesneleri ve diğer tüm 3D görsel öğeler bu kütüphane aracılığıyla hayata geçirilir.

## Genel Amaç ve İçerik

`EterGrnLib` modülü, aşağıdaki gibi çeşitli temel görevler için araçlar sunar:

*   **Model ve Mesh Yönetimi:** `.gr2` dosyalarından 3D modellerin ve bu modelleri oluşturan mesh'lerin yüklenmesi, işlenmesi ve saklanması.
*   **Animasyon Kontrolü:** Modeller için animasyon verilerinin yüklenmesi, animasyonların oynatılması, karıştırılması (blending) ve zamanlaması.
*   **Materyal ve Doku Yönetimi:** Modellerin yüzey özelliklerini tanımlayan materyallerin ve bu materyallerle ilişkili dokuların yönetimi.
*   **Sahne Yönetimi (Scene Graph):** Oyun dünyasındaki 3D nesnelerin ("ThingInstance") hiyerarşik bir yapıda yönetilmesi, konumlandırılması ve güncellenmesi.
*   **Render İşlemleri:** Modellerin ve diğer 3D öğelerin grafik motoru aracılığıyla ekrana çizilmesi.
*   **Çarpışma Tespiti:** Modeller için çarpışma verilerinin işlenmesi ve çarpışma testleri için arayüzler.
*   **Seviye Detayı (LOD - Level of Detail):** Performansı optimize etmek için modellerin farklı detay seviyelerinin yönetilmesi.
*   **Yardımcı Fonksiyonlar:** Kütüphane içinde kullanılan çeşitli matematiksel ve yardımcı işlevler.

Bu modüldeki bileşenler, istemcinin görsel dünyasının zengin ve dinamik bir şekilde sunulmasını sağlar.

---

## Dosya Bazlı Detaylandırma

### `StdAfx.h` ve `StdAfx.cpp`

*   **Amaç:** Bu dosyalar, `EterGrnLib` projesi için ön derlenmiş başlık (PCH) mekanizmasını oluşturur. Sık kullanılan ve nadiren değiştirilen sistem başlıklarını (Granny 3D) ve proje içi temel başlıkları (`EterBase` modülünden bazı başlıklar ve `EterGrnLib` içindeki `Util.h`) içerir.
*   **Temel Özellikler/İçerik (`StdAfx.h`):**
    *   `#pragma once`: Başlık dosyasının tek bir derleme biriminde yalnızca bir kez dahil edilmesini sağlar.
    *   `#pragma warning(disable:4786)`: Uzun sembol adlarıyla ilgili bir derleyici uyarısını devre dışı bırakır.
    *   **Ana Bağımlılıklar:**
        *   `<granny.h>`: Granny 3D animasyon kütüphanesinin ana başlık dosyası. `EterGrnLib`'in temelini oluşturur.
        *   `"../UserInterface/Locale_inc.h"`: Yerelleştirme ayarları için `UserInterface` modülüne bağımlılık.
        *   `"../EterBase/Utils.h"`: `EterBase`'den genel yardımcı fonksiyonlar.
        *   `"../EterBase/Debug.h"`: `EterBase`'den hata ayıklama ve loglama araçları.
        *   `"../EterBase/Stl.h"`: `EterBase`'den STL yardımcıları.
        *   `"Util.h"`: `EterGrnLib` içinde bulunan yerel bir yardımcı dosya.
    *   **Armadillo Nanomite Koruması:** `NANOBEGIN` ve `NANOEND` makroları, potansiyel bir yazılım koruma katmanıyla ilgili assembly komutları içerir.
*   **İçerik (`StdAfx.cpp`):**
    *   `#include "stdafx.h"`: Ön derlenmiş başlık dosyasını dahil eder.
    *   `namespace { char dummy; };`: LNK4221 bağlayıcı uyarısını (bir obje dosyasının genel sembol tanımlamaması durumu) çözmek için kullanılan bir tekniktir.
*   **Kullanım Alanı:** Doğrudan oyun mantığında bir rolü yoktur. Temel amacı, Visual Studio'da ön derlenmiş başlıkları kullanarak `EterGrnLib` kütüphanesinin ve ona bağımlı diğer projelerin derleme sürelerini önemli ölçüde kısaltmaktır. `.pch` dosyası, sık kullanılan başlıkların her `.cpp` dosyasında tekrar tekrar işlenmesini engeller. 

### `Util.h` ve `Util.cpp`

*   **Amaç:** `EterGrnLib` içinde kullanılan bazı temel yardımcı fonksiyonları ve veri yapılarını tanımlar. Özellikle Granny 3D mesh'leriyle ilgili durum sorgulamaları ve materyal verilerini gruplamak için kullanılır.
*   **Temel Özellikler/İçerik (`Util.h`):**
    *   **`GrannyMeshIsDeform(granny_mesh* pgrnMesh)` Fonksiyonu:**
        *   Verilen bir Granny mesh'inin (`pgrnMesh`) deforme olabilir bir yapıda olup olmadığını kontrol eder.
        *   Genellikle, bir mesh'in iskeletsel animasyonla (skinned animation) etkileşime girip girmediğini belirlemek için kullanılır. Rijit olmayan (non-rigid) mesh'ler deforme olabilir kabul edilir.
    *   **`SMaterialData` Yapısı:**
        *   Bir materyale ait temel verileri bir arada tutar:
            *   `CGraphicImage* pImage`: Materyalin kullandığı ana dokuya (diffuse map) bir işaretçi. `CGraphicImage` sınıfı muhtemelen başka bir kütüphanede (örn: `EterImageLib`) tanımlıdır.
            *   `float fSpecularPower`: Materyalin speküler (parlaklık/yansıma) gücünü belirler.
            *   `BOOL isSpecularEnable`: Materyalde speküler yansımanın aktif olup olmadığını gösteren bir bayrak.
            *   `BYTE bSphereMapIndex`: Küre haritası (sphere mapping) kullanımı için bir indeks. Bu, çevresel yansımalar veya özel materyal efektleri için kullanılabilir.
    *   **Yorum Satırına Alınmış Fonksiyonlar:**
        *   `GrannyMeshGetTextureAnimation` ve `GrannyMeshIsTextureAnimation` adında iki fonksiyon bildirimi yorum satırı içinde bulunmaktadır. Bunlar, geçmişte mesh'ler için UV (doku koordinatı) animasyonlarını sorgulamak ve özelliklerini almak için kullanılmış olabilecek fonksiyonlardır.
*   **Implementasyon Detayları (`Util.cpp`):**
    *   `GrannyMeshIsDeform` fonksiyonu, Granny 3D API'sinin `GrannyMeshIsRigid()` fonksiyonunu çağırır. Eğer `GrannyMeshIsRigid()` `true` (mesh rijit, yani deforme olmuyor) döndürürse, `GrannyMeshIsDeform` `false` döndürür. Aksi takdirde (mesh rijit değilse, yani deforme olabilirse) `true` döndürür.
    *   Yorum satırına alınmış `GrannyMeshGetTextureAnimation` fonksiyonunun implementasyonu, Granny mesh'inin `ExtendedData` alanından özel bir veri yapısı (`SUVData`) okuyarak UV animasyon bayrağını ve UV kayma hızlarını almaya çalıştığını gösterir.
*   **Kullanım Alanı:**
    *   `GrannyMeshIsDeform`: Model yükleme veya işleme sırasında, bir mesh'in animasyon sistemine nasıl dahil edileceğini (örneğin, skinning uygulanıp uygulanmayacağını) belirlemek için kullanılır.
    *   `SMaterialData`: Modellerin materyal bilgileri yüklenirken veya `CGrannyMaterial` gibi sınıflar içinde materyal özelliklerini taşımak ve yönetmek için kullanılır. Bu yapı, render sırasında materyal parametrelerini grafik motoruna iletmek için de bir aracı olabilir. 

### `Material.h` ve `Material.cpp`

*   **Amaç:** `CGrannyMaterial` ve `CGrannyMaterialPalette` sınıflarını tanımlayarak Granny 3D materyallerinin yüklenmesini, yönetilmesini, özelliklerinin ayarlanmasını ve Direct3D 8 render state'lerinin bu materyallere göre yapılandırılmasını sağlar.
*   **`CGrannyMaterial` Sınıfı:**
    *   **Kalıtım:** `EterLib::CReferenceObject` (Referans sayımı için).
    *   **Genel Bakış:** Tek bir Granny materyalini temsil eder, dokularını, speküler özelliklerini, saydamlık ayarlarını ve render state yönetimi için fonksiyon işaretçilerini tutar.
    *   **Statik Üyeler ve Metotlar:**
        *   `ms_akSphereMapInstance[]`: Küre haritası dokularını (`CGraphicImageInstance`) saklamak için statik bir dizi.
        *   `CreateSphereMap()`, `DestroySphereMap()`: Küre haritalarını yüklemek ve temizlemek için statik metotlar.
        *   `ms_matSpecular`, `ms_v3SpecularTrans`: Genel bir speküler aydınlatma efekti için kullanılabilen statik bir matris ve translasyon vektörü. `TranslateSpecularMatrix()` ile güncellenir.
    *   **Materyal Türleri (`EType` Enum):**
        *   `TYPE_DIFFUSE_PNT`: Standart opak veya alfa-testli materyal.
        *   `TYPE_BLEND_PNT`: Alfa karıştırmalı (saydam) materyal.
    *   **Temel Metotlar:**
        *   `CGrannyMaterial()`: Kurucu, `Initialize()` çağırır.
        *   `CreateFromGrannyMaterialPointer(granny_material* pgrnMaterial)`: Bir Granny materyal yapısından `CGrannyMaterial` nesnesini başlatır. Dokuları (`__GetImagePointer` ile yükler), materyal adını analiz ederek ve Granny materyal flag'lerini kontrol ederek `m_eType` ve `m_bTwoSideRender` gibi özellikleri ayarlar.
        *   `SetImagePointer(int iStage, CGraphicImage* pImage)`: Belirli bir doku katmanına (0: diffuse, 1: opacity/sphere) bir imaj atar.
        *   `SetSpecularInfo(BOOL bFlag, float fPower, BYTE uSphereMapIndex)`: Speküler ve küre haritası özelliklerini ayarlar. Bu metot aynı zamanda `m_pfnApplyRenderState` ve `m_pfnRestoreRenderState` fonksiyon işaretçilerini uygun render fonksiyonlarına (`__ApplyDiffuseRenderState` veya `__ApplySpecularRenderState`) yönlendirir.
        *   `ApplyRenderState()`: Kayıtlı fonksiyon işaretçisi (`m_pfnApplyRenderState`) aracılığıyla mevcut materyal türü için uygun Direct3D render state'lerini uygular.
        *   `RestoreRenderState()`: `ApplyRenderState` ile yapılan değişiklikleri geri alır.
        *   `GetImagePointer(int iStage)`, `GetD3DTexture(int iStage)`, `GetDiffuseTexture()`, `GetOpacityTexture()`: İlgili doku veya imaj işaretçilerini döndürür.
        *   `IsTwoSided()`: Materyalin çift taraflı render edilip edilmeyeceğini döndürür.
    *   **Render State Uygulama Detayları (`__Apply...`, `__Restore...` metotları):**
        *   `__ApplyDiffuseRenderState()`: `EterLib::STATEMANAGER` aracılığıyla temel render state'lerini ayarlar:
            *   1. Doku katmanına diffuse dokusunu atar (`STATEMANAGER.SetTexture(0, ...)`).
            *   2. Doku katmanını temizler.
            *   Alfa-blend ayarlarını (`D3DRS_ALPHABLENDENABLE`, `D3DRS_SRCBLEND`, `D3DRS_DESTBLEND`) materyal türüne (`TYPE_BLEND_PNT`) ve diffuse dokusunun alfa kanalına göre yapar.
            *   `m_bTwoSideRender` ise culling'i kapatır (`D3DCULL_NONE`).
        *   `__ApplySpecularRenderState()`: `__ApplyDiffuseRenderState` üzerine ek olarak:
            *   2. Doku katmanına küre haritası dokusunu (`ms_akSphereMapInstance[m_bSphereMapIndex]`) atar.
            *   2. Doku katmanı için doku koordinat üretimi (`D3DTSS_TCI_CAMERASPACEREFLECTIONVECTOR`) ve doku transform matrisi (`D3DTS_TEXTURE1` olarak `ms_matSpecular`) ayarlarını yapar.
            *   Renk ve alfa operasyonlarını (`D3DTOP_MODULATE`, `D3DTOP_ADD`) iki doku katmanını birleştirmek için ayarlar.
            *   Gerekirse speküler renk (`g_fSpecularColor`) için materyal rengi ayarlarını yapar.
        *   İlgili `Restore` fonksiyonları bu state'leri varsayılan değerlere döndürür.
    *   **Doku Yükleme (`__GetImagePointer`):**
        *   `CResourceManager::Instance().GetResourcePointer()` kullanarak doku dosyalarını yükler.
        *   `SUPPORT_LOCAL_TEXTURE` bloğu ile, modelin bulunduğu yerel bir alt klasörden doku aramayı destekler.
*   **`CGrannyMaterialPalette` Sınıfı:**
    *   **Amaç:** Bir modelin veya bir grup modelin kullanabileceği `CGrannyMaterial` örneklerinden oluşan bir koleksiyonu (paleti) yönetir. Bu, aynı materyallerin tekrar tekrar oluşturulmasını engeller.
    *   **Temel Metotlar:**
        *   `RegisterMaterial(granny_material* pgrnMaterial)`: Verilen bir Granny materyali için palettede zaten bir `CGrannyMaterial` olup olmadığını kontrol eder (`IsEqual`). Yoksa, yeni bir `CGrannyMaterial` oluşturur, `CreateFromGrannyMaterialPointer` ile başlatır ve palete (`m_mtrlVector`) ekler. Bulunan veya yeni oluşturulan materyalin indeksini döndürür.
        *   `SetMaterialImagePointer(const char* c_szMtrlName, CGraphicImage* pImage)`: Palet içindeki materyallerden, Granny materyal adı (`pgrnMaterial->Name`) eşleşen ilk materyali bulur ve onun `SetImagePointer` metodunu çağırır.
        *   `SetMaterialData(const char* c_szMtrlName, const SMaterialData& c_rkMaterialData)`: İsimle eşleşen materyali bulur ve `SMaterialData` yapısındaki bilgileri (imaj, speküler güç, speküler aktifliği, küre haritası indeksi) kullanarak materyalin özelliklerini günceller. Özellikle `SetSpecularInfo` ve `SetImagePointer` çağrılarını yapar.
        *   `SetSpecularInfo(const char* c_szMtrlName, BOOL bEnable, float fPower)`: İsimle eşleşen materyalin speküler bilgilerini `SetSpecularInfo` ile ayarlar.
        *   `GetMaterialRef(DWORD mtrlIndex)`: Verilen indeksteki `CGrannyMaterial`'e referans döndürür.
*   **Kullanım Alanı:**
    *   `CGrannyMaterial`, bir Granny 3D modelindeki her bir materyalin oyun motoru tarafından nasıl işleneceğini ve render edileceğini tanımlar. Dokuların atanması, saydamlık, speküler yansımalar ve küre haritaları gibi görsel özellikler bu sınıf üzerinden yönetilir.
    *   `CGrannyMaterialPalette`, bir model yüklenirken o modelin kullandığı tüm materyalleri verimli bir şekilde yönetmek için kullanılır. Aynı materyalin birden fazla kopya oluşturulmasını engelleyerek bellek tasarrufu sağlar ve materyal yönetimini merkezileştirir.
    *   Bu sınıflar, model verilerini grafik motorunun anlayabileceği ve işleyebileceği bir formata dönüştürür ve Direct3D render state'lerini dinamik olarak ayarlayarak istenen görsel efektlerin elde edilmesini sağlar. `SMaterialData` yapısı, dışarıdan (örneğin, scriptler veya özel konfigürasyon dosyaları aracılığıyla) materyal özelliklerinin daha kolay ayarlanabilmesi için bir arayüz sunar. 

### `Mesh.h` ve `Mesh.cpp`

*   **Amaç:** `CGrannyMesh` sınıfı, Granny 3D modelinin bir parçasını oluşturan tek bir mesh'i (çokgen ağı) temsil eder. Mesh geometrisini (vertex ve index verileri), materyal atamalarını, deformasyon (skinning) yeteneklerini ve render için üçgen gruplarını yönetir.
*   **Vertex Formatı (`granny_pnt3322_vertex` ve `GrannyPNT3322VertexType`):**
    *   Oyun içinde kullanılan standart vertex formatını tanımlar: Pozisyon (3 float), Normal (3 float), UV0 (2 float - ana doku), UV1 (2 float - ikincil doku, örn: lightmap, detay haritası).
    *   `GrannyPNT3322VertexType`, bu formatı Granny 3D kütüphanesine bildiren bir veri yapısıdır.
*   **`CGrannyMesh` Sınıfı:**
    *   **Mesh Türleri (`EType` Enum):**
        *   `TYPE_RIGID`: Deforme olmayan (non-skinned) mesh.
        *   `TYPE_DEFORM`: İskeletsel animasyonla deforme olan (skinned) mesh.
    *   **Üçgen Grupları (`TTriGroupNode` Yapısı):**
        *   Bir mesh içindeki, aynı materyali kullanan üçgen alt kümelerini temsil eder.
        *   `idxPos`: Grubun genel index buffer'ındaki başlangıç indeksi.
        *   `triCount`: Gruptaki üçgen sayısı.
        *   `mtrlIndex`: Grubun kullandığı materyalin `CGrannyMaterialPalette` içindeki indeksi.
        *   `pNextTriGroupNode`: Aynı render türüne sahip bir sonraki gruba işaretçi (opak/saydam grupları ayrı listelerde tutmak için).
    *   **Ana Metotlar ve İşlevler:**
        *   `CreateFromGrannyMeshPointer(granny_skeleton* pgrnSkeleton, granny_mesh* pgrnMesh, int vtxBasePos, int idxBasePos, CGrannyMaterialPalette& rkMtrlPal)`: Bir Granny mesh yapısından `CGrannyMesh` nesnesini başlatır.
            *   Verilen `pgrnMesh`'i saklar, `vtxBasePos` ve `idxBasePos`'u (bu mesh'in birleşik bir vertex/index buffer'ındaki ofsetleri) ayarlar.
            *   `GrannyNewMeshBinding` ile mesh'i iskelete bağlar.
            *   Mesh rijit değilse (`!GrannyMeshIsRigid`), `GrannyNewMeshDeformer` ile bir deformer oluşturur. Bu deformer, Granny'nin vertex formatından `GrannyPNT3322VertexType` formatına dönüşümü ve pozisyon/normal deformasyonunu (`GrannyDeformPositionNormal`) yapacak şekilde ayarlanır.
            *   Mesh adının "2x" ile başlayıp başlamadığına bakarak çift taraflı render (`m_isTwoSide`) özelliğini belirler.
            *   `LoadMaterials()` ile materyallerini `rkMtrlPal`'e kaydeder.
            *   `LoadTriGroupNodeList()` ile üçgen gruplarını materyal türlerine göre oluşturur.
        *   `LoadIndices(void* dstBaseIndices)`: Mesh'in indekslerini `GrannyCopyMeshIndices` kullanarak hedef belleğe kopyalar.
        *   `LoadPNTVertices(void* dstBaseVertices)` / `NEW_LoadVertices(void* dstBaseVertices)`: Rijit mesh'ler için vertex verilerini (`GrannyPNT3322VertexType` formatında) `GrannyCopyMeshVertices` kullanarak hedef belleğe kopyalar. Deforme olan mesh'ler için bu fonksiyonlar bir şey yapmaz.
        *   `DeformPNTVertices(void* dstBaseVertices, D3DXMATRIX* boneMatrices, granny_mesh_binding* pgrnMeshBinding) const`: İskeletsel animasyon için skinning işlemini gerçekleştirir. `GrannyDeformVertices` API'sini kullanarak, orijinal vertex'leri, verilen kemik matrislerini ve mesh bağlama bilgilerini temel alarak deforme eder ve sonucu hedef vertex buffer'ına yazar.
        *   `CanDeformPNTVertices() const`: Mesh'in skinning ile deforme edilip edilemeyeceğini döndürür (`m_canDeformPNTVertex`).
        *   `GetTriGroupNodeList(CGrannyMaterial::EType eMtrlType) const`: Belirli bir materyal türüne (opak, saydam) ait üçgen grubu listesinin başlangıcını döndürür. Bu, render sırasında farklı geçişlerde (pass) doğru çizim yapılmasını sağlar.
    *   **Materyal ve Üçgen Grubu Yükleme (`LoadMaterials`, `LoadTriGroupNodeList`):**
        *   `LoadMaterials()`: Mesh'in kullandığı her `granny_material` için `CGrannyMaterialPalette::RegisterMaterial()` çağırarak materyalleri palete kaydeder ve indekslerini `m_mtrlIndexVector` içinde saklar. Ayrıca, saydam bir materyal olup olmadığını `m_bHaveBlendThing` bayrağında tutar.
        *   `LoadTriGroupNodeList()`: `GrannyGetMeshTriangleGroups` ile Granny'den üçgen gruplarını alır. Her grup için bir `TTriGroupNode` oluşturur, indeks başlangıcını (`idxPos`), üçgen sayısını (`triCount`) ve materyal paletindeki materyal indeksini (`mtrlIndex`) ayarlar. Oluşturulan `TTriGroupNode`'u, materyalinin türüne (`CGrannyMaterial::EType`) göre ilgili `m_triGroupNodeLists` (örneğin, `m_triGroupNodeLists[TYPE_DIFFUSE_PNT]` veya `m_triGroupNodeLists[TYPE_BLEND_PNT]`) bağlı listesinin başına ekler.
*   **Kullanım Alanı:**
    *   `CGrannyMesh`, bir 3D modelin (örn: `CGrannyModel`) render edilebilir geometrik ve materyal bileşenlerini temsil eder.
    *   Model yükleme sırasında, her bir Granny mesh'i için bir `CGrannyMesh` nesnesi oluşturulur.
    *   Render sırasında, `CGrannyModelInstance` (veya benzeri bir sınıf) `CGrannyMesh`'in üçgen grubu listelerini (`GetTriGroupNodeList`) kullanarak mesh'i çizer. Önce opak gruplar, sonra saydam gruplar gibi farklı çizim geçişleri uygulanabilir.
    *   Animasyonlu karakterler veya nesneler için, `DeformPNTVertices` metodu her karede çağrılarak vertex'lerin kemik pozisyonlarına göre güncellenmesi (skinning) sağlanır.
    *   Vertex ve index verileri genellikle birleşik (batched) büyük vertex/index buffer'larına yüklenir; `m_vtxBasePos` ve `m_idxBasePos` bu büyük buffer'lardaki ofsetleri tutar.
    *   Bu sınıflar, model verilerini grafik motorunun anlayabileceği ve işleyebileceği bir formata dönüştürür ve Direct3D render state'lerini dinamik olarak ayarlayarak istenen görsel efektlerin elde edilmesini sağlar. `SMaterialData` yapısı, dışarıdan (örneğin, scriptler veya özel konfigürasyon dosyaları aracılığıyla) materyal özelliklerinin daha kolay ayarlanabilmesi için bir arayüz sunar. 

### `Model.h` ve `Model.cpp`

*   **Amaç:** `CGrannyModel` sınıfı, bir Granny 3D model dosyasından (.gr2) yüklenen tüm statik model verilerini temsil eder. Bu, modelin içerdiği tüm mesh'leri (`CGrannyMesh`), bu mesh'lerin kullandığı materyalleri (`CGrannyMaterialPalette`), ve bu mesh'lere ait birleşik Direct3D vertex ve index buffer'larını içerir.

*   **`CGrannyModel` Sınıfı:**
    *   **Kalıtım:** `CReferenceObject` (Referans sayımı için).
    *   **Yapısal Üyeler:**
        *   `SMeshNode`: Model içindeki bir mesh'i ve onun render listesindeki bir sonraki mesh'i işaret eden bir bağlı liste düğüm yapısıdır. Render sırasında mesh'leri materyal ve mesh tipine göre verimli bir şekilde dolaşmak için kullanılır.
            *   `iMesh`: Modelin mesh dizisindeki (`m_meshs`) indeksi.
            *   `pMesh`: İlgili `CGrannyMesh` nesnesine işaretçi.
            *   `pNextMeshNode`: Aynı render kategorisindeki bir sonraki `SMeshNode`'a işaretçi.
    *   **Temel Değişkenler:**
        *   `m_pgrnModel`: Yüklenen orijinal Granny 3D model verisine (`granny_model`) işaretçi.
        *   `m_meshs`: Modelin içerdiği tüm `CGrannyMesh` nesnelerini tutan bir dizi.
        *   `m_pntVtxBuf`: Rijit (deforme olmayan) mesh'ler için kullanılan `CGraphicVertexBuffer`.
        *   `m_idxBuf`: Tüm mesh'ler için ortak kullanılan `CGraphicIndexBuffer`.
        *   `m_meshNodes`: `SMeshNode`'lardan oluşan bir dizi (bellek havuzu).
        *   `m_meshNodeLists[CGrannyMesh::TYPE_MAX_NUM][CGrannyMaterial::TYPE_MAX_NUM]`: Farklı mesh tipleri (RIGID, DEFORM) ve materyal tipleri (DIFFUSE_PNT, BLEND_PNT) için `SMeshNode` bağlı listelerinin başlangıçlarını tutan iki boyutlu bir dizi. Bu, render sırasında belirli özelliklere sahip mesh'lerin hızlıca seçilmesini sağlar.
        *   `m_deformVtxCount`, `m_rigidVtxCount`, `m_vtxCount`: Deforme olan, rijit ve toplam vertex sayıları.
        *   `m_idxCount`: Toplam index sayısı.
        *   `m_canDeformPNVertices`: Modelin deforme olabilir vertex'lere sahip olup olmadığını belirten bir bayrak.
        *   `m_kMtrlPal`: Modele ait `CGrannyMaterialPalette` (materyal koleksiyonu).
        *   `m_bHaveBlendThing`: Modelin saydam (alpha blended) materyale sahip bir mesh içerip içermediğini gösterir.
        *   `m_dwFvF`: Modelin vertex formatını (D3DFVF) belirten bir değer.
    *   **Ana Metotlar ve İşlevler:**
        *   `CreateFromGrannyModelPointer(granny_model* pgrnModel)`: Verilen bir `granny_model` işaretçisinden `CGrannyModel`'i başlatır. `LoadMeshs()` çağırarak mesh'leri ve materyallerini yükler/oluşturur. Ayrıca toplam vertex ve index sayılarını hesaplar.
        *   `CreateDeviceObjects()`: `LoadPNTVertices()` ve `LoadIndices()` çağırarak Direct3D vertex ve index buffer'larını oluşturur ve doldurur.
        *   `DestroyDeviceObjects()`: Oluşturulan Direct3D buffer'larını serbest bırakır.
        *   `Destroy()`: Tüm kaynakları temizler (mesh'ler, materyal paleti, Granny model verisi).
        *   `LoadMeshs()`: Modeldeki her `granny_mesh` için bir `CGrannyMesh` nesnesi oluşturur (`m_meshs` dizisine). Her mesh için `CGrannyMesh::CreateFromGrannyMeshPointer` çağırılır. Bu sırada, mesh'lerin vertex ve index sayıları toplanır, deforme olup olmadıkları kontrol edilir ve D3D vertex formatı (`m_dwFvF`) belirlenir. Son olarak, `AppendMeshNode` kullanılarak mesh'ler render için `m_meshNodeLists` içindeki uygun listelere eklenir.
        *   `LoadPNTVertices()`: Rijit mesh'lere ait vertex verilerini `m_pntVtxBuf` içine `CGrannyMesh::LoadPNTVertices` kullanarak yükler.
        *   `LoadIndices()`: Tüm mesh'lere ait index verilerini `m_idxBuf` içine `CGrannyMesh::LoadIndices` kullanarak yükler.
        *   `AppendMeshNode(CGrannyMesh::EType eMeshType, CGrannyMaterial::EType eMtrlType, int iMesh)`: Belirli bir mesh'i (`iMesh` indeksi ile) ve onun türünü (`eMeshType`, `eMtrlType`) alarak, `m_meshNodes` havuzundan bir `SMeshNode` alır ve bu node'u `m_meshNodeLists` içindeki uygun bağlı listenin başına ekler.
        *   `GetMeshPointer(int iMesh)`: Belirtilen indeksteki `CGrannyMesh`'e işaretçi döndürür.
        *   `GetPNTD3DVertexBuffer()`: Rijit mesh'ler için kullanılan Direct3D vertex buffer'ını döndürür.
        *   `GetD3DIndexBuffer()`: Ortak Direct3D index buffer'ını döndürür.
        *   `GetMeshNodeList(CGrannyMesh::EType eMeshType, CGrannyMaterial::EType eMtrlType) const`: Belirtilen mesh ve materyal türüne sahip mesh'lerin bağlı liste başlangıcını döndürür. Bu, render sırasında belirli türdeki mesh'leri (örneğin, önce tüm opak rijit mesh'ler, sonra tüm opak deforme mesh'ler vb.) verimli bir şekilde çizmek için kullanılır.
        *   `GetMaterialPalette() const`: Modelin materyal paletine (`CGrannyMaterialPalette`) const referans döndürür.
        *   `CanDeformPNTVertices() const`: Modelin herhangi bir deforme edilebilir mesh içerip içermediğini kontrol eder.
        *   `DeformPNTVertices(void* dstBaseVertices, D3DXMATRIX* boneMatrices, const std::vector<granny_mesh_binding*>& c_rvct_pgrnMeshBinding) const`: Modeldeki tüm deforme olabilir mesh'ler için `CGrannyMesh::DeformPNTVertices` çağırarak skinning işlemini gerçekleştirir ve sonucu `dstBaseVertices`'e yazar.
*   **Kullanım Alanı:**
    *   `CGrannyModel`, bir .gr2 dosyasından yüklenen ve oyun dünyasında birden çok kez örneklenebilecek (bkz. `CGrannyModelInstance`) bir modelin temel geometrik ve materyal verilerini tutar.
    *   Render işlemi için gerekli olan birleşik vertex/index buffer'larını ve materyal bilgilerini sağlar.
    *   Mesh'leri, render verimliliği için türlerine (rijit/deforme) ve materyal türlerine (opak/saydam) göre organize eder.
    *   Bir modelin yüklenmesi genellikle şu adımları izler:
        1.  Granny 3D API kullanılarak .gr2 dosyası yüklenir (`granny_model`).
        2.  `CGrannyModel::CreateFromGrannyModelPointer` ile bu `granny_model`'dan `CGrannyModel` oluşturulur. Bu aşamada `CGrannyMesh`'ler ve `CGrannyMaterialPalette` oluşturulur.
        3.  `CGrannyModel::CreateDeviceObjects` ile Direct3D vertex ve index buffer'ları oluşturulur ve doldurulur.
    *   Oyun dünyasındaki her bir görünür model örneği (`CGrannyModelInstance`), bir `CGrannyModel` işaretçisine sahip olur ve bu statik veriyi kullanarak kendini çizer ve anime eder.

### `ModelInstance.h` (`CGrannyModelInstance` Sınıfı)

*   **Amaç:** `CGrannyModelInstance` sınıfı, oyun dünyasında bir `CGrannyModel`'in (statik model verisi) canlı, anime edilebilir ve render edilebilir bir örneğini temsil eder. Her bir karakter, canavar veya dinamik 3D nesne, genellikle bir `CGrannyModelInstance` ile ifade edilir.

*   **`CGrannyModelInstance` Sınıfı:**
    *   **Kalıtım:** `CGraphicCollisionObject` (Çarpışma tespiti yetenekleri için).
    *   **Statik Üyeler ve Metotlar (Bellek Yönetimi):**
        *   `ms_kPool`: `CGrannyModelInstance` nesneleri için bir `CDynamicPool` (bellek havuzu) kullanarak verimli bellek yönetimi sağlar.
        *   `New()`: Havuzdan yeni bir `CGrannyModelInstance` nesnesi alır veya oluşturur.
        *   `Delete(CGrannyModelInstance* pkInst)`: Verilen nesneyi havuza geri bırakır.
        *   `DestroySystem()`: Bellek havuzunu temizler.
    *   **Temel Değişkenler:**
        *   `m_pModel`: Bu örneğin temel aldığı statik `CGrannyModel`'e işaretçi.
        *   `m_pgrnModelInstance`: Granny 3D kütüphanesine ait gerçek model örneği (`granny_model_instance`). İskeletin anlık pozunu, animasyon kontrollerini vb. tutar.
        *   `m_pgrnCtrl`: Aktif animasyon için Granny 3D kontrolcüsü (`granny_control`). Animasyonun zamanlamasını, döngülerini ve ağırlığını yönetir.
        *   `m_pgrnAni`: Mevcut olarak bağlı olan Granny 3D animasyon verisine (`granny_animation`) işaretçi.
        *   `m_meshMatrices`: Modelin her bir mesh'i için hesaplanmış dünya (world) dönüşüm matrislerini tutan bir dizi. Bu matrisler, mesh'leri modelin genel pozisyonuna ve iskeletin deformasyonuna göre doğru konumlandırır.
        *   `mc_pParentInstance`, `m_iParentBoneIndex`: Bu model örneğinin başka bir `CGrannyModelInstance`'a (ebeveyn) ve onun belirli bir kemiğine (`m_iParentBoneIndex`) bağlanmasını sağlar (attachment).
        *   `m_fLocalTime`: Animasyonun yerel zamanını saniye cinsinden tutar.
        *   `m_fSecondsElapsed`: Son güncellemeden bu yana geçen süreyi tutar.
        *   `m_kMtrlPal`: Modele özel materyal değişikliklerini (`SetMaterialImagePointer`, `SetMaterialData`) yönetmek için bir `CGrannyMaterialPalette` kopyası veya örneği. Bu, aynı modelin farklı örneklerinin farklı dokulara veya materyal ayarlarına sahip olabilmesini sağlar.
        *   `m_pgrnWorldPoseReal`: Granny 3D dünya pozu (`granny_world_pose`). İskeletin kemiklerinin son dünya koordinatlarındaki pozisyonlarını ve oryantasyonlarını içerir.
        *   `m_vct_pgrnMeshBinding`: Modelin mesh'lerini iskelete bağlayan Granny 3D mesh bağlama (`granny_mesh_binding`) nesnelerinin bir vektörü.
        *   `m_pkSharedDeformableVertexBuffer`, `m_kLocalDeformableVertexBuffer`, `m_isDeformableVertexBuffer`: Deforme olabilen vertex'ler için dinamik vertex buffer yönetimini sağlar. Performans için paylaşımlı bir buffer kullanabilir veya kendi lokal buffer'ını oluşturabilir.
    *   **Ana Metot Grupları (Detayları ilgili `.cpp` dosyalarında açıklanacaktır):**
        *   **Kurucu/Yok Edici ve Başlatma:** `CGrannyModelInstance()`, `~CGrannyModelInstance()`, `Clear()`, `__Initialize()`.
        *   **Device Object Yönetimi:** `CreateDeviceObjects()`, `DestroyDeviceObjects()`. Özellikle dinamik vertex buffer (`__CreateDynamicVertexBuffer`, `__DestroyDynamicVertexBuffer`) ve Granny model örneği (`__CreateModelInstance`, `__DestroyModelInstance`) ile ilgilenir.
        *   **Güncelleme (Update):**
            *   `Update(DWORD dwAniFPS)`: Ana güncelleme fonksiyonu, animasyon hızını sınırlar.
            *   `UpdateLocalTime(float fElapsedTime)`: Animasyonun yerel zamanını ilerletir.
            *   `UpdateTransform(D3DXMATRIX* pMatrix, float fSecondsElapsed)`: Modelin genel dünya matrisini ayarlar ve `UpdateSkeleton` ile `Deform` (veya `DeformNoSkin`) çağırır.
            *   `UpdateSkeleton(const D3DXMATRIX* c_pWorldMatrix, float fLocalTime)`: Granny API'lerini (`GrannySetModelClock`, `GrannySampleModelAnimationsAccelerated`) kullanarak iskeletin pozunu günceller.
            *   `Deform(const D3DXMATRIX* c_pWorldMatrix)`: Vertex skinning işlemini gerçekleştirir. `UpdateWorldPose`, `UpdateWorldMatrices` ve `DeformPNTVertices` çağırır.
            *   `DeformNoSkin(const D3DXMATRIX* c_pWorldMatrix)`: Skinning yapmadan sadece dünya pozunu günceller.
        *   **Render (Çizim):** `RenderWithOneTexture()`, `RenderWithTwoTexture()`, `BlendRenderWithOneTexture()`, `BlendRenderWithTwoTexture()`, `RenderWithoutTexture()`. Bu fonksiyonlar, `CGrannyModel`'den alınan mesh listelerini ve materyalleri kullanarak modeli çizer.
        *   **Model Erişimi ve Ayarları:**
            *   `SetMainModelPointer()`: Ana `CGrannyModel`'i ve paylaşımlı deformasyon buffer'ını ayarlar.
            *   `SetLinkedModelPointer()`: Başka bir modelin iskeletine bağlanacak şekilde modeli ayarlar.
            *   `SetMaterialImagePointer()`, `SetMaterialData()`, `SetSpecularInfo()`: Modele ait materyallerin özelliklerini (doku, speküler güç vb.) dinamik olarak değiştirir.
        *   **Animasyon (Motion) Kontrolü:**
            *   `SetMotionPointer()`: Yeni bir animasyon başlatır (karıştırma süresi, döngü sayısı, hız oranı ile).
            *   `ChangeMotionPointer()`: Mevcut animasyonu kesip yenisine geçer.
            *   `CopyMotion()`: Başka bir model örneğinin animasyonunu kopyalar.
            *   `IsMotionPlaying()`: Animasyonun oynatılıp oynatılmadığını kontrol eder.
        *   **Zaman Kontrolü:** `SetLocalTime()`, `GetLocalTime()`.
        *   **Kemik (Bone) ve Bağlama (Attaching):**
            *   `GetBoneMatrixPointer()`: Belirli bir kemiğin matrisini döndürür.
            *   `GetMeshMatrixPointer()`: Belirli bir mesh'in dünya matrisini döndürür.
            *   `GetBoneIndexByName()`: İsme göre kemik indeksi bulur.
            *   `SetParentModelInstance()`: Bu örneği başka bir örneğin kemiğine bağlar.
        *   **Çarpışma Tespiti (Collision Detection):** `Intersect()`, `GetBoundBox()`.
    *   **Yardımcı Metotlar:**
        *   `__CreateWorldPose()`, `__DestroyWorldPose()`: `granny_world_pose` oluşturma/yok etme.
        *   `__CreateMeshBindingVector()`, `__DestroyMeshBindingVector()`: Mesh bağlama vektörünü yönetir.
        *   `__GetDeformableD3DVertexBufferPtr()`: Deforme edilebilir vertex buffer'ına erişim.
        *   `UpdateWorldPose()`: `GrannyBuildWorldPose` kullanarak kemiklerin dünya matrislerini hesaplar.
        *   `UpdateWorldMatrices()`: Her bir mesh için son render matrislerini hesaplar.
        *   `DeformPNTVertices()`: `CGrannyModel::DeformPNTVertices` veya lokal implementasyon ile vertex'leri deforme eder.
*   **Kullanım Alanı:**
    *   Oyun sahnesindeki her bir 3D varlık (karakter, NPC, canavar, hareketli dekor objesi vb.) için bir `CGrannyModelInstance` kullanılır.
    *   Modelin hangi animasyonu oynatacağını, nerede duracağını, hangi materyal özelliklerine sahip olacağını ve nasıl render edileceğini yönetir.
    *   Birden fazla `CGrannyModelInstance`, aynı `CGrannyModel` verisini paylaşarak bellekten tasarruf edebilir, ancak her biri kendi animasyon durumuna, pozisyonuna ve materyal ayarlarına sahip olabilir.
    *   Animasyon sistemi, kemiklere nesne bağlama (örneğin, bir karakterin eline kılıç takma) ve çarpışma tespiti gibi özellikler bu sınıf üzerinden sağlanır.

### `ModelInstance.cpp` (Temel Kurulum ve Yönetim)

Bu dosya, `CGrannyModelInstance` sınıfının temel başlatma, temizleme ve bellek yönetimi fonksiyonlarını içerir.

*   **Bellek Yönetimi (Statik Fonksiyonlar):**
    *   `CGrannyModelInstance* CGrannyModelInstance::New()`: `ms_kPool` (dinamik bellek havuzu) kullanarak yeni bir `CGrannyModelInstance` nesnesi oluşturur ve döndürür.
    *   `void CGrannyModelInstance::Delete(CGrannyModelInstance* pkInst)`: Verilen `CGrannyModelInstance` nesnesini temizler (`pkInst->Clear()`) ve ardından bellek havuzuna geri bırakır (`ms_kPool.Free(pkInst)`).
    *   `void CGrannyModelInstance::DestroySystem()`: `ms_kPool` bellek havuzunu tamamen yok eder. Genellikle program sonlanırken çağrılır.
*   **Kurucu ve Yok Edici:**
    *   `CGrannyModelInstance::CGrannyModelInstance()`: Kurucu metot. `m_pModel`'i `NULL` olarak ayarlar ve ardından `__Initialize()` fonksiyonunu çağırarak tüm üye değişkenleri varsayılan durumlarına getirir.
    *   `CGrannyModelInstance::~CGrannyModelInstance()`: Yok edici metot. `Clear()` fonksiyonunu çağırarak model örneğiyle ilişkili tüm kaynakları serbest bırakır.
*   **Başlatma ve Temizleme:**
    *   `void CGrannyModelInstance::__Initialize()`: Model örneğinin tüm üye değişkenlerini sıfırlar veya varsayılan değerlerine ayarlar. Örneğin:
        *   Eğer `m_pModel` daha önce bir modele işaret ediyorsa, `m_pModel->Release()` ile referansını bırakır.
        *   `m_pModel`, `mc_pParentInstance`, `m_pgrnModelInstance`, `m_pgrnWorldPoseReal`, `m_meshMatrices`, `m_pgrnCtrl`, `m_pgrnAni` gibi işaretçileri `NULL` yapar.
        *   `m_iParentBoneIndex`, `material_data_`, `m_dwOldUpdateFrame` gibi değişkenleri sıfırlar.
    *   `void CGrannyModelInstance::Clear()`: Daha kapsamlı bir temizleme işlemidir. `__DestroyModelInstance()` (Granny model örneğini yok eder), `__DestroyMeshMatrices()` (mesh matrislerini siler), `DestroyDeviceObjects()` (dinamik vertex buffer'ı yok eder) ve `__Initialize()` (tüm değişkenleri sıfırlar) çağırır. Bu, model örneğinin tamamen sıfırlanmasını ve kaynaklarının serbest bırakılmasını sağlar.
*   **Durum Kontrolü:**
    *   `bool CGrannyModelInstance::IsEmpty()`: Model örneğinin geçerli bir modele (`m_pModel`) ve mesh matrislerine (`m_meshMatrices`) sahip olup olmadığını kontrol eder. Eğer bunlardan biri eksikse `true` (boş) döner.
*   **Device Object Yönetimi:**
    *   `bool CGrannyModelInstance::CreateDeviceObjects()`: `__CreateDynamicVertexBuffer()` çağırarak (eğer gerekiyorsa) deforme olabilir vertex'ler için dinamik bir vertex buffer oluşturur.
    *   `void CGrannyModelInstance::DestroyDeviceObjects()`: `__DestroyDynamicVertexBuffer()` çağırarak oluşturulmuş dinamik vertex buffer'ı serbest bırakır.
*   **Model ve Materyal Erişimi (Basit Yönlendirmeler):**
    *   `CGrannyModel* CGrannyModelInstance::GetModel()`: Bağlı olan `CGrannyModel` işaretçisini (`m_pModel`) döndürür.
    *   `void CGrannyModelInstance::SetMaterialImagePointer(const char* c_szImageName, CGraphicImage* pImage)`: `m_kMtrlPal` (yerel materyal paleti) üzerinden belirtilen isimdeki materyalin dokusunu ayarlar.
    *   `void CGrannyModelInstance::SetMaterialData(const char* c_szImageName, const SMaterialData& c_rkMaterialData)`: `m_kMtrlPal` üzerinden belirtilen isimdeki materyalin verilerini (doku, speküler vb.) ayarlar ve `material_data_` üye değişkenini günceller.
    *   `void CGrannyModelInstance::SetSpecularInfo(const char* c_szMtrlName, BOOL bEnable, float fPower)`: `m_kMtrlPal` üzerinden belirtilen isimdeki materyalin speküler ayarlarını yapar.
*   **Zaman Yönetimi:**
    *   `void CGrannyModelInstance::SetLocalTime(float fLocalTime)`: Animasyonun yerel zamanını (`m_fLocalTime`) ayarlar.
    *   `int CGrannyModelInstance::ResetLocalTime()`: Animasyonun yerel zamanını `0.0f`'a sıfırlar.
    *   `float CGrannyModelInstance::GetLocalTime()`: Mevcut yerel animasyon zamanını döndürür.
*   **Ebeveyn Ataçlama (Attaching):**
    *   `void CGrannyModelInstance::SetParentModelInstance(const CGrannyModelInstance* c_pParentModelInstance, const char* c_szBoneName)`: Bu model örneğini, başka bir `CGrannyModelInstance`'ın (`c_pParentModelInstance`) belirtilen isme (`c_szBoneName`) sahip kemiğine bağlar. Önce kemik isminden kemik indeksini bulur, sonra diğer `SetParentModelInstance` fonksiyonunu çağırır.
    *   `void CGrannyModelInstance::SetParentModelInstance(const CGrannyModelInstance* c_pParentModelInstance, int iBone)`: Bu model örneğini, `c_pParentModelInstance`'ın `iBone` indeksli kemiğine bağlamak için `mc_pParentInstance` ve `m_iParentBoneIndex` üyelerini ayarlar.

### `ModelInstanceModel.cpp` (Model Atama ve Kaynak Yönetimi)

Bu dosya, `CGrannyModelInstance`'ın bir `CGrannyModel` (statik model verisi) ile nasıl ilişkilendirildiğini, Granny 3D kütüphanesine özgü kaynakların (model instance, world pose, mesh binding) nasıl oluşturulup yönetildiğini ve dinamik vertex buffer'ların nasıl ele alındığını içerir.

*   **Model Atama Fonksiyonları:**
    *   `void CGrannyModelInstance::SetMainModelPointer(CGrannyModel* pModel, CGraphicVertexBuffer* pkSharedDeformableVertexBuffer)`: Birincil model olarak `pModel`'i ayarlar. `SetLinkedModelPointer` fonksiyonunu çağırır, ancak iskelet için ayrı bir örnek belirtmez (yani bu modelin kendi iskeletini kullanacağını varsayar).
    *   `void CGrannyModelInstance::SetLinkedModelPointer(CGrannyModel* pkModel, CGraphicVertexBuffer* pkSharedDeformableVertexBuffer, CGrannyModelInstance** ppkSkeletonInst)`: Bu, `CGrannyModelInstance`'ı bir `CGrannyModel` (`pkModel`) ile ilişkilendiren ana fonksiyondur.
        1.  Önce `Clear()` çağrılarak mevcut durum temizlenir.
        2.  `m_pModel`'e yeni `pkModel` atanır ve referans sayısı artırılır (`m_pModel->AddReference()`).
        3.  Eğer `pkSharedDeformableVertexBuffer` verilmişse, paylaşımlı deformasyon buffer'ı olarak ayarlanır (`__SetSharedDeformableVertexBuffer`). Yoksa, bu örnek için yerel bir dinamik deformasyon buffer'ı oluşturulur (`__CreateDynamicVertexBuffer`).
        4.  `__CreateModelInstance()` çağrılarak Granny 3D model örneği (`granny_model_instance`) oluşturulur.
        5.  Eğer `ppkSkeletonInst` (başka bir model örneğinin iskeletini kullanmak için işaretçi) geçerliyse:
            *   `m_ppkSkeletonInst` ayarlanır.
            *   `__CreateWorldPose(*ppkSkeletonInst)`: Verilen iskelet örneğinin dünya pozunu kullanacak şekilde ayarlanır.
            *   `__CreateMeshBindingVector(*ppkSkeletonInst)`: Mesh'leri verilen iskelet örneğine bağlar.
        6.  Eğer `ppkSkeletonInst` geçerli değilse (model kendi iskeletini kullanıyorsa):
            *   `__CreateWorldPose(NULL)`: Kendi iskeleti için yeni bir dünya pozu oluşturur.
            *   `__CreateMeshBindingVector(NULL)`: Mesh'leri kendi iskeletine bağlar.
        7.  `__CreateMeshMatrices()` çağrılarak mesh'ler için dünya matrislerini tutacak bellek alanı ayrılır.
        8.  `ResetLocalTime()` ile animasyon zamanı sıfırlanır.
        9.  `m_kMtrlPal.Copy(pkModel->GetMaterialPalette())`: Modelin materyal paletinin bir kopyası, bu örneğin yerel materyal paletine (`m_kMtrlPal`) alınır. Bu, örneğe özel materyal değişikliklerine izin verir.
*   **Granny Kaynak Oluşturma/Yok Etme (Yardımcı Fonksiyonlar):**
    *   `void CGrannyModelInstance::__CreateModelInstance()`: `m_pModel`'deki `granny_model` verisini kullanarak `GrannyInstantiateModel()` ile bir `granny_model_instance` oluşturur ve `m_pgrnModelInstance`'a atar.
    *   `void CGrannyModelInstance::__DestroyModelInstance()`: `m_pgrnModelInstance`'ı `GrannyFreeModelInstance()` ile serbest bırakır.
    *   `void CGrannyModelInstance::__CreateWorldPose(CGrannyModelInstance* pkSkeletonInst)`: Eğer `pkSkeletonInst` (paylaşılan iskelet) verilmemişse, `m_pgrnModelInstance`'ın kaynak iskeletini alarak `GrannyNewWorldPose()` ile yeni bir `granny_world_pose` oluşturur ve `m_pgrnWorldPoseReal`'e atar. Eğer paylaşılan bir iskelet varsa, bu fonksiyon bir şey yapmaz (o iskeletin dünya pozu kullanılır).
    *   `void CGrannyModelInstance::__DestroyWorldPose()`: `m_pgrnWorldPoseReal`'i `GrannyFreeWorldPose()` ile serbest bırakır.
    *   `bool CGrannyModelInstance::__CreateMeshBindingVector(CGrannyModelInstance* pkDstModelInst)`: Modelin (`m_pModel`) mesh'lerini hedef bir iskelete (`pkDstModelInst` verilmişse onun iskeleti, yoksa modelin kendi iskeleti) bağlamak için `granny_mesh_binding` nesneleri oluşturur. Her mesh için `GrannyNewMeshBinding()` çağırılır ve sonuçlar `m_vct_pgrnMeshBinding` içinde saklanır.
    *   `void CGrannyModelInstance::__DestroyMeshBindingVector()`: `m_vct_pgrnMeshBinding` içindeki tüm `granny_mesh_binding`'leri `GrannyFreeMeshBinding()` ile serbest bırakır.
    *   `void CGrannyModelInstance::__CreateMeshMatrices()`: Modelin mesh sayısı kadar `D3DXMATRIX` için bellek ayırır ve `m_meshMatrices`'e atar.
    *   `void CGrannyModelInstance::__DestroyMeshMatrices()`: `m_meshMatrices` için ayrılan belleği siler.
*   **Dinamik Vertex Buffer Yönetimi:**
    *   `void CGrannyModelInstance::__SetSharedDeformableVertexBuffer(CGraphicVertexBuffer* pkSharedDeformableVertexBuffer)`: `m_pkSharedDeformableVertexBuffer`'ı ayarlar, böylece bu örnek paylaşımlı bir buffer kullanır.
    *   `bool CGrannyModelInstance::__IsDeformableVertexBuffer()`: Paylaşımlı bir buffer kullanılıyorsa veya yerel deformasyon buffer'ı boş değilse `true` döner.
    *   `IDirect3DVertexBuffer8* CGrannyModelInstance::__GetDeformableD3DVertexBufferPtr()`: Aktif deformasyon vertex buffer'ının Direct3D işaretçisini döndürür (paylaşımlı veya yerel).
    *   `CGraphicVertexBuffer& CGrannyModelInstance::__GetDeformableVertexBufferRef()`: Aktif deformasyon vertex buffer'ına (`CGraphicVertexBuffer`) referans döndürür.
    *   `void CGrannyModelInstance::__CreateDynamicVertexBuffer()`: Eğer paylaşımlı bir buffer kullanılmıyorsa ve modelde deforme olabilir vertex'ler varsa (`m_pModel->GetDeformVertexCount() > 0`), `m_kLocalDeformableVertexBuffer` için `D3DFVF_XYZ | D3DFVF_NORMAL | D3DFVF_TEX1` formatında bir vertex buffer oluşturur.
    *   `void CGrannyModelInstance::__DestroyDynamicVertexBuffer()`: `m_kLocalDeformableVertexBuffer`'ı yok eder ve `m_pkSharedDeformableVertexBuffer`'ı `NULL` yapar.
*   **Diğer Yardımcı ve Erişim Fonksiyonları:**
    *   `granny_world_pose* CGrannyModelInstance::__GetWorldPosePtr() const`: Kullanılacak `granny_world_pose` işaretçisini döndürür. Eğer bu örnek başka bir iskelete bağlıysa (`m_ppkSkeletonInst`), o iskeletin dünya pozunu, aksi halde kendi `m_pgrnWorldPoseReal`'ini döndürür.
    *   `const granny_int32x* CGrannyModelInstance::__GetMeshBoneIndices(unsigned int iMeshBinding) const`: Belirtilen mesh bağlama indeksi için kemik indekslerini döndürür (`GrannyGetMeshBindingToBoneIndices`).
    *   `DWORD CGrannyModelInstance::GetDeformableVertexCount()`: Modeldeki deforme olabilir vertex sayısını `m_pModel` üzerinden döndürür.
    *   `DWORD CGrannyModelInstance::GetVertexCount()`: Modeldeki toplam vertex sayısını `m_pModel` üzerinden döndürür.
    *   `bool CGrannyModelInstance::GetBoneIndexByName(const char* c_szBoneName, int* pBoneIndex) const`: İsme göre kemik indeksi bulmak için Granny API'si `GrannyFindBoneByName` kullanılır.
    *   `const float* CGrannyModelInstance::GetBoneMatrixPointer(int iBone) const`: Belirtilen kemiğin dünya matrisini `GrannyGetWorldPose4x4` ile `__GetWorldPosePtr()` üzerinden alır.
    *   `const float* CGrannyModelInstance::GetCompositeBoneMatrixPointer(int iBone) const`: Belirtilen kemiğin birleşik (composite) dünya matrisini `GrannyGetWorldPoseComposite4x4` ile alır. Bu genellikle iskeletin lokal pozisyonunu da içerir.
    *   `bool CGrannyModelInstance::GetMeshMatrixPointer(int iMesh, const D3DXMATRIX** c_ppMatrix) const`: Belirtilen mesh'in `m_meshMatrices` dizisinden dünya matrisini döndürür.

Bu dosyadaki fonksiyonlar, bir model örneğinin temelini oluşturur ve onu animasyon, güncelleme ve render işlemleri için hazırlar.

### `ModelInstanceMotion.cpp` (Animasyon Kontrolü)

Bu dosya, `CGrannyModelInstance` sınıfının animasyon (motion) yönetimiyle ilgili fonksiyonlarını içerir. Animasyonların atanması, değiştirilmesi, kopyalanması ve durum sorgulamaları bu dosyada gerçekleştirilir.

*   **Animasyon Atama ve Değiştirme:**
    *   `void CGrannyModelInstance::SetMotionPointer(const CGrannyMotion* pMotion, float blendTime, int loopCount, float speedRatio)`: Belirtilen `pMotion` animasyonunu model örneğine atar.
        1.  Eğer mevcut bir animasyon kontrolcüsü (`m_pgrnCtrl`) varsa, bu kontrolcünün `blendTime` (saniye cinsinden) süresi içinde yumuşak bir şekilde sonlanması (ease-out) ayarlanır (`GrannySetControlEaseOutCurve`, `GrannyCompleteControlAt`). Kontrolcü, tamamlandığında serbest bırakılmak üzere işaretlenir (`GrannyFreeControlIfComplete`).
        2.  Yeni animasyon (`pMotion->GetGrannyAnimationPointer()`) için mevcut yerel zaman (`GetLocalTime()`) ile `GrannyPlayControlledAnimation` kullanılarak yeni bir kontrolcü (`m_pgrnCtrl`) oluşturulur.
        3.  Yeni kontrolcünün hızı (`speedRatio`) ve döngü sayısı (`loopCount`) ayarlanır (`GrannySetControlSpeed`, `GrannySetControlLoopCount`).
        4.  Eğer bu ilk animasyonsa, ease-in ve ease-out devre dışı bırakılır. Değilse ve `blendTime > 0.0f` ise, yeni animasyonun `blendTime` süresi içinde yumuşak bir şekilde başlaması (ease-in) ayarlanır (`GrannySetControlEaseInCurve`).
        5.  Yeni kontrolcü, kullanılmadığında otomatik olarak serbest bırakılması için işaretlenir (`GrannyFreeControlOnceUnused`).
    *   `void CGrannyModelInstance::ChangeMotionPointer(const CGrannyMotion* pMotion, int loopCount, float speedRatio)`: Mevcut animasyonu anında keserek (`GrannyCompleteControlAt` ile mevcut zamandan `fSkipTime` kadar öncesine ayarlanır) yeni bir animasyona (`pMotion`) geçer. Yeni animasyon için ease-in/out devre dışı bırakılır. Bu, genellikle animasyonlar arasında ani ve keskin geçişler için kullanılır.
*   **Animasyon Kopyalama:**
    *   `void CGrannyModelInstance::CopyMotion(CGrannyModelInstance* pModelInstance, bool bIsFreeSourceControl)`: Başka bir `CGrannyModelInstance`'ın (`pModelInstance`) mevcut animasyon durumunu bu örneğe kopyalar.
        1.  Eğer kaynak modelin animasyonu oynamıyorsa işlem yapılmaz.
        2.  Bu örneğin mevcut `m_pgrnCtrl`'ü varsa serbest bırakılır.
        3.  Kaynak modelin animasyon (`m_pgrnAni`) ve kontrolcü ayarları (hız, döngü sayısı, mevcut ham yerel saat - `GrannyGetControlRawLocalClock`) bu örneğe kopyalanır.
        4.  Yeni oluşturulan kontrolcü için ease-in `true`, ease-out `false` olarak ayarlanır.
        5.  Eğer `bIsFreeSourceControl` `true` ise, kaynak modelin animasyon kontrolcüsü serbest bırakılır.
*   **Animasyon Durum Sorgulama:**
    *   `bool CGrannyModelInstance::IsMotionPlaying()`: Model örneğinde bir animasyonun aktif olarak oynatılıp oynatılmadığını kontrol eder. `m_pgrnCtrl` geçerliyse ve `GrannyControlIsComplete(m_pgrnCtrl)` `false` ise `true` döner.
*   **Animasyon Sonu Ayarı:**
    *   `void CGrannyModelInstance::SetMotionAtEnd()`: Eğer bir animasyon kontrolcüsü (`m_pgrnCtrl`) varsa, animasyonun yerel saatini doğrudan sonuna ayarlar (`GrannyGetControlLocalDuration` ile süreyi alıp `GrannySetControlRawLocalClock` ile ayarlar). Bu, animasyonu son karesine anında atlatır.

Bu fonksiyonlar, Granny 3D kütüphanesinin animasyon kontrol (`granny_control`) ve zamanlama mekanizmalarını kullanarak model örneklerinin dinamik olarak anime edilmesini sağlar. Özellikle `SetMotionPointer`, animasyonlar arasında yumuşak geçişler (blending / ease-in/out) yapmak için karmaşık ama esnek bir yapı sunar.

### `ModelInstanceRender.cpp` (Render İşlemleri)

Bu dosya, `CGrannyModelInstance` sınıfının render (çizim) işlemlerinden sorumlu fonksiyonlarını içerir. Modelin ekranda nasıl görüneceğini belirleyen temel mantığı barındırır.

**Temel İşlevleri:**

*   **Render Fonksiyonları:**
    *   `RenderWithOneTexture()`: Modeli tek bir doku kullanarak çizer. Hem deforme olabilen (skinned) hem de rigid (sabit) meshleri işler. `CGrannyMaterial::TYPE_DIFFUSE_PNT` malzeme tipini kullanır.
    *   `BlendRenderWithOneTexture()`: Modeli tek bir doku kullanarak ve alpha blending etkinleştirilmiş şekilde çizer. `CGrannyMaterial::TYPE_BLEND_PNT` malzeme tipini kullanır.
    *   `RenderWithTwoTexture()`: Modeli iki doku katmanı kullanarak çizer (örneğin, bir ana doku ve bir detay veya ışık haritası). `CGrannyMaterial::TYPE_DIFFUSE_PNT` malzeme tipini kullanır, ancak materyal ayarları ikinci dokuyu da hesaba katacak şekilde yapılandırılır (genellikle `CGrannyMaterial` içindeki `__ApplySpecularRenderState` gibi özel bir render state fonksiyonu ile).
    *   `BlendRenderWithTwoTexture()`: İki doku ve alpha blending ile çizim yapar. `CGrannyMaterial::TYPE_BLEND_PNT` malzeme tipini kullanır.
    *   `RenderWithoutTexture()`: Modeli doku kullanmadan, genellikle düz renk veya özel efektler (örneğin, gölgeler için depth map'e çizerken) için çizer.
*   **Mesh Çizim Yardımcıları:**
    *   `RenderMeshNodeListWithOneTexture(CGrannyMesh::EType eMeshType, CGrannyMaterial::EType eMtrlType)`
    *   `RenderMeshNodeListWithTwoTexture(CGrannyMesh::EType eMeshType, CGrannyMaterial::EType eMtrlType)`
    *   `RenderMeshNodeListWithoutTexture(CGrannyMesh::EType eMeshType, CGrannyMaterial::EType eMtrlType)`:
        Bu özel fonksiyonlar, belirli bir mesh tipi (`CGrannyMesh::EType` - DEFORM veya RIGID) ve malzeme tipi (`CGrannyMaterial::EType`) için modelin mesh listesini (`m_pModel->GetMeshNodeList()`) işler.
        *   Her bir mesh (`pMeshNode->pMesh`) için:
            *   Modelin genel index buffer'ını (`m_pModel->GetD3DIndexBuffer()`) ve mesh'in bu buffer'daki başlangıç pozisyonunu (`vtxMeshBasePos`) ayarlar (`STATEMANAGER.SetIndices`).
            *   Mesh'e ait önceden hesaplanmış dünya matrisini (`m_meshMatrices[pMeshNode->iMesh]`) Direct3D'ye bildirir (`STATEMANAGER.SetTransform(D3DTS_WORLD, ...)`).
            *   Mesh'in üçgen gruplarını (`TTriGroupNode`, `pMesh->GetTriGroupNodeList(eMtrlType)`) dolaşır.
            *   Her üçgen grubu (`pTriGroupNode`) için:
                *   İlgili materyali yerel materyal paletinden (`m_kMtrlPal.GetMaterialRef(pTriGroupNode->mtrlIndex)`) alır.
                *   Eğer `material_data_` (muhtemelen dışarıdan anlık olarak materyal özelliklerini değiştirmek için kullanılan bir yapı) içinde bir imaj belirtilmemişse ve speküler güç değişmişse, materyalin speküler ayarlarını günceller (`rkMtrl.SetSpecularInfo`).
                *   Materyalin render durumlarını uygular (`rkMtrl.ApplyRenderState()`). Bu, doku atamalarını, alpha blending ayarlarını, culling modunu vb. yapar. `RenderMeshNodeListWithTwoTexture` durumunda, dokular doğrudan `STATEMANAGER.SetTexture()` ile atanır. `RenderMeshNodeListWithoutTexture`'da ise herhangi bir doku atanmaz.
                *   Direct3D `DrawIndexedPrimitive` çağrısı ile asıl çizim işlemini gerçekleştirir.
                *   Materyalin render durumlarını geri yükler (`rkMtrl.RestoreRenderState()`) (eğer `ApplyRenderState` kullanıldıysa).
*   **Deformasyon (Skinning Olmadan):**
    *   `DeformNoSkin(const D3DXMATRIX* c_pWorldMatrix)`: Bu fonksiyon, modelin dünya pozunu (`UpdateWorldPose()`) ve mesh matrislerini (`UpdateWorldMatrices(c_pWorldMatrix)`) günceller, ancak vertex deformasyonunu (skinning) uygulamaz. Genellikle basit, hareket etmeyen veya vertex animasyonu olmayan modeller ya da sadece iskeletin güncellenmesi gereken durumlar için kullanılabilir.
*   **Vertex Shader Ayarı:**
    *   Çoğu render fonksiyonu, `STATEMANAGER.SetVertexShader(ms_pntVS)` çağrısı ile `TPNTVertex` (Position, Normal, Texture) formatındaki vertexler için uygun olan vertex shader'ı ayarlar. `ms_pntVS` muhtemelen `CGrannyModelInstance` sınıfının statik bir üyesidir.
*   **VertexBuffer ve IndexBuffer Yönetimi:**
    *   Çizimden önce uygun vertex buffer'ları (deforme olabilen meshler için `__GetDeformableD3DVertexBufferPtr()`, rigid meshler için `m_pModel->GetPNTD3DVertexBuffer()`) ve index buffer'ı (`m_pModel->GetD3DIndexBuffer()`) ayarlar.
    *   `STATEMANAGER.SetStreamSource()` ile Direct3D'ye bu buffer'ları ve vertex boyutunu bildirir.

**Çalışma Prensibi:**

`CGrannyModelInstance`'ın render süreci, `CGrannyModel`'den alınan statik mesh ve materyal bilgilerini, örneğe özgü anlık hesaplanmış dünya matrisleri ve materyal ayarlarıyla birleştirerek gerçekleşir.
Render fonksiyonları çağrıldığında:
1.  Öncelikle modelin geçerli olup olmadığı kontrol edilir (`IsEmpty()`).
2.  Gerekli vertex shader ayarlanır.
3.  Kullanılacak vertex buffer'lar (deforme olabilen ve/veya rigid) Direct3D'ye tanıtılır.
4.  Ardından, ilgili `RenderMeshNodeList...` fonksiyonlarından uygun olanı çağrılır. Bu fonksiyonlar, modelin mesh listesini (mesh türüne ve malzeme türüne göre filtrelenmiş) dolaşır.
5.  Her bir mesh için, o mesh'e ait dünya matrisi ayarlanır ve içindeki üçgen grupları (genellikle materyale göre gruplanmış) tek tek işlenir.
6.  Her bir üçgen grubu için ilgili materyal ayarları uygulanır ve `DrawIndexedPrimitive` ile üçgenler çizilir.

Bu yapı, farklı render senaryolarını (tek dokulu, çift dokulu, alpha blending'li vb.) esnek bir şekilde yönetmeyi sağlar. `STATEMANAGER` üzerinden yapılan Direct3D çağrıları, render state yönetimini merkezileştirir. Debug amaçlı `Granny_RenderBoxBones` gibi fonksiyonlar da içerebilir (`#ifdef _TEST`).

### `ModelInstanceUpdate.cpp` (Güncelleme ve Deformasyon)

Bu dosya, `CGrannyModelInstance` sınıfının her oyun döngüsünde (frame) modelin durumunu güncelleyen fonksiyonlarını içerir. Bu, animasyon zamanının ilerletilmesi, iskeletin (skeleton) güncellenmesi, kemiklerin dünya pozlarının (world pose) hesaplanması, mesh'ler için nihai dönüşüm matrislerinin oluşturulması ve son olarak vertex'lerin deforme edilmesi (skinning) işlemlerini kapsar.

*   **Ana Güncelleme Fonksiyonları:**
    *   `void CGrannyModelInstance::Update(DWORD dwAniFPS)`: Modelin animasyonunu belirli bir FPS'e (kare/saniye) göre günceller.
        *   `GetLocalTime()` ile mevcut animasyonun yerel zamanını alır ve bunu `ANIFPS_MAX` (genellikle 120 gibi yüksek bir değer) ile çarparak bir "yüksek çözünürlüklü" frame sayısına dönüştürür (`c_dwCurUpdateFrame`).
        *   `dwAniFPS` (istenen animasyon FPS'i, örneğin 30) kullanarak bir adım değeri (`ANIFPS_STEP`) hesaplar.
        *   Eğer mevcut yüksek çözünürlüklü frame, bir önceki güncellemeden bu yana yeterince ilerlemediyse (yani aynı `dwAniFPS` adımına denk geliyorsa), gereksiz güncellemeyi atlar. Bu, animasyon hızını `dwAniFPS` ile sınırlamaya yardımcı olur.
        *   `m_dwOldUpdateFrame` güncellenir.
        *   `GrannyFreeCompletedModelControls(m_pgrnModelInstance)`: Tamamlanmış Granny animasyon kontrolcülerini serbest bırakır.
        *   `GrannySetModelClock(m_pgrnModelInstance, GetLocalTime())`: Granny model örneğinin saatini mevcut yerel animasyon zamanına ayarlar. Bu, Granny'nin animasyonları doğru zamanda örneklemesi için kritik öneme sahiptir.
    *   `void CGrannyModelInstance::UpdateLocalTime(float fElapsedTime)`: Basitçe, `m_fSecondsElapsed`'i (son kareden bu yana geçen süre) günceller ve `m_fLocalTime`'a (animasyonun toplam yerel zamanı) ekler.
    *   `void CGrannyModelInstance::UpdateTransform(D3DXMATRIX* pMatrix, float fSecondsElapsed)`: Modelin genel dünya (world) matrisini (`pMatrix`) kullanarak Granny model örneğinin kök dönüşümünü günceller. `GrannyUpdateModelMatrix` API'si bu iş için kullanılır. Bu fonksiyon, iskelet ve mesh'lerin bu genel dönüşüme göre doğru konumlandırılmasını sağlar.
*   **Deformasyon (Skinning) Fonksiyonları:**
    *   `void CGrannyModelInstance::Deform(const D3DXMATRIX* c_pWorldMatrix)`: Modelin vertex'lerini iskelet animasyonuna göre deforme eder (skinning).
        1.  Modelin boş olup olmadığını kontrol eder (`IsEmpty()`).
        2.  `UpdateWorldPose()`: İskeletin kemiklerinin dünya koordinatlarındaki nihai pozlarını hesaplar.
        3.  `UpdateWorldMatrices(c_pWorldMatrix)`: Her bir mesh için, `c_pWorldMatrix` (modelin genel dünya matrisi) ve hesaplanan kemik pozlarını kullanarak nihai render matrislerini oluşturur.
        4.  Eğer model deforme edilebilir vertex'lere sahipse (`m_pModel->CanDeformPNTVertices()`):
            *   Deforme edilebilir vertex buffer'ını alır (`__GetDeformableVertexBufferRef()`).
            *   Bu buffer'ı kilitleyip (`LockRange`) vertex verilerine bir işaretçi (`pntVertices`) alır.
            *   `DeformPNTVertices(pntVertices)` çağırarak asıl skinning işlemini yapar ve sonucu kilitlenmiş buffer'a yazar.
            *   Buffer'ın kilidini açar (`Unlock`).
*   **İskelet ve Dünya Pozu Güncelleme:**
    *   `void CGrannyModelInstance::UpdateSkeleton(const D3DXMATRIX* c_pWorldMatrix, float /*fLocalTime*/)`: Bu fonksiyon, `DeformNoSkin` gibi, skinning yapmadan sadece iskeletin dünya pozunu ve mesh matrislerini günceller. `UpdateWorldPose()` ve `UpdateWorldMatrices(c_pWorldMatrix)` çağırır.
    *   `void CGrannyModelInstance::UpdateWorldPose()`: İskeletin kemiklerinin dünya matrislerini hesaplayan ana fonksiyondur.
        *   Eğer model başka bir iskelete bağlıysa (`m_ppkSkeletonInst` ve `*m_ppkSkeletonInst != this`), bu model için dünya pozu hesaplaması yapılmaz (bağlı olduğu iskeletin pozu kullanılır).
        *   Statik bir `CGrannyLocalPose` nesnesi (`s_SharedLocalPose`) kullanarak paylaşımlı bir lokal poz buffer'ı alır. Bu, her güncellemede bellek ayırma/bırakma maliyetini azaltır.
        *   `GrannyGetSourceSkeleton()` ile model örneğinin iskeletini alır.
        *   Eğer model başka bir modele/kemiğe bağlıysa (`mc_pParentInstance`), ebeveyn kemiğin matrisini alır.
        *   `GrannySampleModelAnimationsAccelerated(m_pgrnModelInstance, pgrnSkeleton->BoneCount, pAttachBoneMatrix, pgrnLocalPose, __GetWorldPosePtr())`: Bu kritik Granny API çağrısı, model örneğindeki aktif animasyon kontrolcülerini kullanarak iskeletin lokal pozunu (`pgrnLocalPose`) örnekler ve bunu ebeveyn kemik matrisiyle birleştirerek nihai dünya pozunu (`__GetWorldPosePtr()` ile erişilen `m_pgrnWorldPoseReal`) hesaplar.
        *   `GrannyFreeCompletedModelControls(m_pgrnModelInstance)`: Tekrar tamamlanmış kontrolcüleri temizler.
*   **Mesh Matrislerini Güncelleme:**
    *   `void CGrannyModelInstance::UpdateWorldMatrices(const D3DXMATRIX* c_pWorldMatrix)`: Her bir mesh için nihai dünya matrisini (`m_meshMatrices`) hesaplar.
        *   `GrannyGetWorldPoseComposite4x4Array(__GetWorldPosePtr())` ile tüm kemiklerin birleşik (composite) dünya matrislerini alır.
        *   Modeldeki her bir mesh (`m_pModel->GetMeshPointer(i)`) için:
            *   Eğer mesh deforme olabiliyorsa (`pMesh->CanDeformPNTVertices()`), mesh'in dünya matrisi doğrudan modelin genel dünya matrisine (`*c_pWorldMatrix`) eşitlenir (çünkü skinning işlemi vertex'leri zaten dünya uzayına taşıyacaktır).
            *   Eğer mesh rijit ise (deforme olmuyorsa), o mesh'in bağlı olduğu kemiğin (`*boneIndices`) dünya matrisi ile modelin genel dünya matrisi (`*c_pWorldMatrix`) çarpılarak mesh'in nihai dünya matrisi hesaplanır.
*   **Vertex Deformasyon Yardımcısı:**
    *   `void CGrannyModelInstance::DeformPNTVertices(void* pvDest)`: Asıl skinning işlemini `CGrannyModel` sınıfının `DeformPNTVertices` metoduna devreder. Bu metoda, hedef vertex buffer işaretçisi (`pvDest`), kemiklerin birleşik dünya matrisleri (`GrannyGetWorldPoseComposite4x4Array(__GetWorldPosePtr())`) ve mesh bağlama bilgileri (`m_vct_pgrnMeshBinding`) verilir.

Bu dosyadaki fonksiyonlar, bir `CGrannyModelInstance`'ın her karede canlı ve dinamik kalmasını sağlar, animasyonları ilerletir ve modeli render edilmeye hazır hale getirir.

### `ModelInstanceCollisionDetection.cpp` (Çarpışma Tespiti)

Bu dosya, `CGrannyModelInstance` sınıfının çarpışma tespitiyle ilgili fonksiyonlarını içerir. Özellikle, modelin genel sınırlayıcı kutusunu (bounding box) hesaplama ve bir ışın (ray) ile modelin kesişip kesişmediğini kontrol etme işlevlerini barındırır.

*   **Sınırlayıcı Kutu (Bounding Box) Hesaplama:**
    *   `void CGrannyModelInstance::GetBoundBox(D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Model örneğinin eksenlere hizalı genel sınırlayıcı kutusunu (Axis-Aligned Bounding Box - AABB) hesaplar.
        1.  `vtMin` ve `vtMax` vektörlerini başlangıçta çok geniş/dar değerlerle başlatır.
        2.  Modeldeki her bir mesh (`m_pModel->GetGrannyModelPointer()->MeshBindings[m].Mesh`) için:
            *   Mesh'in kemik bağlama bilgilerini (`pgrnMesh->BoneBindings`) dolaşır. Her bir kemik bağlaması, o kemiğe bağlı mesh parçasının kendi lokal Yönlendirilmiş Sınırlayıcı Kutusunu (Oriented Bounding Box - OBB) tanımlar (`rgrnBoneBinding.OBBMin`, `rgrnBoneBinding.OBBMax`).
            *   İlgili kemiğin anlık dünya matrisini alır (`GrannyGetWorldPose4x4(__GetWorldPosePtr(), boneIndices[b])`). `boneIndices[b]` ile doğru kemik matrisine ulaşılır.
            *   `MakeBoundBox()` yardımcı fonksiyonunu çağırarak bu OBB'yi kemiğin dünya matrisiyle dönüştürür ve ortaya çıkan 8 köşe noktasının minimum ve maksimum x,y,z değerlerini genel `vtMin` ve `vtMax` ile karşılaştırarak günceller.
    *   `void CGrannyModelInstance::MakeBoundBox(TBoundBox* pBoundBox, const float* mat, const float* OBBMin, const float* OBBMax, D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Bu bir yardımcı fonksiyondur.
        *   Verilen bir OBB (`OBBMin`, `OBBMax`), bir dönüşüm matrisi (`mat`) ve genel bir AABB'nin min/max köşelerini (`vtMin`, `vtMax`) alır.
        *   OBB'nin 2 köşe noktasını (genellikle min ve max köşeleri yeterli olur çünkü dönüşüm sonrası AABB hesaplanacaktır) `mat` ile çarparak dünya koordinatlarına dönüştürür. Sonuçları `pBoundBox` yapısının `sx, sy, sz` (başlangıç) ve `ex, ey, ez` (bitiş) üyelerine yazar.
        *   Bu dönüştürülmüş noktaların x,y,z değerlerini, genel `vtMin` ve `vtMax` değerleriyle karşılaştırarak AABB'yi genişletir.
*   **Işın Kesişimi (Ray Intersection):**
    *   `bool CGrannyModelInstance::Intersect(const D3DXMATRIX* c_pMatrix, float* pu, float* pv, float* pt)`: Verilen bir ışın (implicit olarak `ms_vtPickRayOrig` ve `ms_vtPickRayDir` gibi statik üyelerden alınır) ile modelin kesişip kesişmediğini kontrol eder. `c_pMatrix` muhtemelen ışının modelin lokal uzayına dönüştürülmesinde kullanılacak ek bir matristir (veya modelin dünya matrisidir).
        1.  Eğer `m_pgrnModelInstance` geçerli değilse `false` döner.
        2.  Genel bir AABB (`vtMin`, `vtMax`) hesaplamak için `GetBoundBox`'a benzer bir mantık kullanır:
            *   Modeldeki her mesh'in her kemik bağlaması için OBB'lerini dünya uzayına dönüştürerek genel bir AABB oluşturur. Bu işlem sırasında `TBoundBox` yapıları için bir statik bellek havuzu (`s_boundBoxPool`) kullanılır.
        3.  `IntersectCube()` (muhtemelen `CGraphicObject` veya `CGraphicCollisionObject` temel sınıfından gelen bir fonksiyon) çağrılarak, hesaplanan bu genel AABB ile ışının kesişip kesişmediği kontrol edilir.
        4.  **Not:** Fonksiyonun sonunda yorum satırına alınmış daha detaylı bir kesişim testi logiği bulunmaktadır. Bu bölüm, eğer genel AABB ile kesişim varsa, her bir `TBoundBox` (yani her kemiğe bağlı OBB) ile ayrı ayrı `IntersectBoundBox` testi yapar. Eğer bu da başarılı olursa, mesh rijit değilse direkt `true` döner. Eğer mesh rijit ise, `IntersectMesh` fonksiyonu çağrılarak ışının mesh'in üçgenleriyle doğrudan kesişip kesişmediği test edilir. Bu detaylı test şu an aktif değildir ve fonksiyon sadece genel AABB testiyle `true` dönmektedir. `*pt` (kesişim mesafesi) güncellenir ama `*pu`, `*pv` (kesişim noktasının barycentric koordinatları) güncellenmez.
*   **Mesh Matris Erişimi (Tekrar):**
    *   `bool CGrannyModelInstance::GetMeshMatrixPointer(int iMesh, const D3DXMATRIX** c_ppMatrix) const`: Bu fonksiyon `ModelInstanceModel.cpp`'de de bulunmaktaydı. Belirtilen `iMesh` indeksindeki mesh'e ait ilk kemiğin dünya matrisini döndürür (`GrannyGetWorldPose4x4(__GetWorldPosePtr(), __GetMeshBoneIndices(iMesh)[0])`). Bu, bir mesh'in genel bir dönüşüm matrisini almak için kullanılır, özellikle rijit mesh'ler için anlamlıdır.

Bu dosya, oyun içindeki fareyle tıklama (picking) gibi etkileşimler için temel çarpışma tespiti altyapısını sağlar. `GetBoundBox` ile elde edilen AABB, daha kaba ve hızlı çarpışma ön testleri için kullanılabilirken, `Intersect` (ve yorumdaki detaylı kısım) daha hassas testler için bir başlangıç noktası sunar.

### `Thing.h` ve `Thing.cpp` (`CGraphicThing` Sınıfı)

*   **Amaç:** `CGraphicThing` sınıfı, bir Granny 3D dosyasından (.gr2) yüklenen grafiksel bir "şey"i temsil eder. Bu "şey", birden fazla 3D modeli (`CGrannyModel`) ve bu modellerle ilişkilendirilebilecek birden fazla animasyonu (`CGrannyMotion`) içerebilir. `CGraphicThing`, bu modelleri ve animasyonları yükleyen ve yöneten bir kaynak (`CResource`) olarak işlev görür.

*   **`CGraphicThing` Sınıfı:**
    *   **Kalıtım:** `CResource` (Kaynak yönetimi sistemiyle entegrasyon için).
    *   **Temel Değişkenler:**
        *   `m_pgrnFile`: Yüklenen Granny dosyasının tamamına işaretçi (`granny_file`).
        *   `m_pgrnFileInfo`: Yüklenen Granny dosyasındaki bilgileri (model sayısı, animasyon sayısı, doku bilgileri vb.) içeren yapıya işaretçi (`granny_file_info`).
        *   `m_models`: Dosyadan yüklenen `CGrannyModel` nesnelerini tutan bir dizi.
        *   `m_motions`: Dosyadan yüklenen `CGrannyMotion` nesnelerini tutan bir dizi.
        *   `m_pgrnAni`: Bu değişken `Thing.h`'de tanımlı ancak `Thing.cpp`'de kullanılmıyor gibi görünüyor. Muhtemelen artık kullanılmayan bir kalıntıdır.
    *   **Ana Metotlar ve İşlevler:**
        *   `CGraphicThing(const char* c_szFileName)`: Kurucu. `CResource` kurucusunu çağırır ve `Initialize()`'ı çağırır.
        *   `~CGraphicThing()`: Yok edici. `Clear()` çağırarak kaynakları temizler.
        *   `Initialize()`: Tüm üye işaretçilerini (`m_pgrnFile`, `m_pgrnFileInfo`, `m_models`, `m_motions` vb.) `NULL` olarak ayarlar.
        *   `OnClear()`: Kaynakları temizler. `m_motions` ve `m_models` dizilerini siler, `GrannyFreeFile(m_pgrnFile)` ile Granny dosyasını serbest bırakır ve `Initialize()`'ı çağırır. `CResource` temel sınıfı tarafından çağrılır.
        *   `OnLoad(int iSize, const void* c_pvBuf)`: Kaynak yöneticisi tarafından çağrılır. Verilen bellek tamponundan (`c_pvBuf`, `iSize`) `GrannyReadEntireFileFromMemory` kullanarak bir `granny_file` yükler. Ardından `GrannyGetFileInfo` ile dosya bilgilerini alır. Başarılı olursa `LoadModels()` ve `LoadMotions()` fonksiyonlarını çağırır.
        *   `LoadModels()`: `m_pgrnFileInfo`'daki model sayısı kadar `CGrannyModel` nesnesi oluşturur (`m_models` dizisine). Her model için `CGrannyModel::CreateFromGrannyModelPointer` çağırarak modeli başlatır. **SUPPORT_LOCAL_TEXTURE** bloğu, modelin bulunduğu dizini (`gs_modelLocalPath`) belirleyerek, materyallerin o dizindeki dokuları aramasını sağlar. Son olarak, artık ihtiyaç duyulmayan bazı Granny dosya bölümlerini (`GrannyStandardRigidVertexSection`, `GrannyStandardRigidIndexSection` vb.) bellekten serbest bırakır (`GrannyFreeFileSection`).
        *   `LoadMotions()`: `m_pgrnFileInfo`'daki animasyon sayısı kadar `CGrannyMotion` nesnesi oluşturur (`m_motions` dizisine). Her animasyon için `CGrannyMotion::CreateFromGrannyAnimationPointer` çağırarak animasyonu başlatır.
        *   `CreateDeviceObjects()`: `CResource`'dan gelen sanal bir fonksiyondur. İçerdiği tüm `CGrannyModel`'ların (`m_models`) `CreateDeviceObjects()` fonksiyonlarını çağırarak modellerin Direct3D vertex/index buffer'larını oluşturmasını sağlar.
        *   `DestroyDeviceObjects()`: `CResource`'dan gelen sanal bir fonksiyondur. İçerdiği tüm `CGrannyModel`'ların (`m_models`) `DestroyDeviceObjects()` fonksiyonlarını çağırarak Direct3D buffer'larını serbest bırakır.
        *   `CheckModelIndex(int iModel) const`: Verilen model indeksinin geçerli olup olmadığını kontrol eder.
        *   `GetModelPointer(int iModel)`: Belirtilen indeksteki `CGrannyModel` işaretçisini döndürür.
        *   `GetModelCount() const`: Dosyadaki model sayısını döndürür.
        *   `CheckMotionIndex(int iMotion) const`: Verilen animasyon indeksinin geçerli olup olmadığını kontrol eder.
        *   `GetMotionPointer(int iMotion)`: Belirtilen indeksteki `CGrannyMotion` işaretçisini döndürür.
        *   `GetMotionCount() const`: Dosyadaki animasyon sayısını döndürür.
        *   `GetTextureCount() const`: Dosyada referans verilen doku sayısını döndürür.
        *   `GetTexturePath(int iTexture)`: Belirtilen indeksteki dokunun orijinal dosya yolunu döndürür.
        *   `OnIsEmpty() const`: Kaynağın yüklenip yüklenmediğini (`m_pgrnFile`'ın geçerli olup olmadığını) kontrol eder.
        *   `OnIsType(TType type)`: Kaynak türünün `CGraphicThing` olup olmadığını kontrol eder.
        *   `CGraphicThing::Type()`: Sınıfın tür kimliğini döndürür.

*   **Kullanım Alanı:**
    *   `CGraphicThing`, oyun içinde kullanılan bir 3D varlığın (karakter, canavar, silah, nesne vb.) tüm statik grafik verilerini (modeller ve animasyonlar) tek bir kaynakta toplar.
    *   Kaynak yöneticisi (`ResourceManager`) aracılığıyla .gr2 dosyaları yüklenir ve `CGraphicThing` nesneleri oluşturulur.
    *   Oyun dünyasında bu grafiksel varlığı canlandırmak için `CGraphicThingInstance` (bir sonraki belgeleyeceğimiz sınıf) kullanılır. `CGraphicThingInstance`, bir `CGraphicThing` işaretçisi tutarak onun modellerine ve animasyonlarına erişir ve bunları kullanarak kendini anime eder ve render eder.
    *   Bir `CGraphicThing` nesnesi, birden fazla `CGraphicThingInstance` tarafından paylaşılabilir, böylece aynı model ve animasyon verileri bellekte sadece bir kez tutulur.

### `ThingInstance.h` ve `ThingInstance.cpp` (`CGraphicThingInstance` Sınıfı)

*   **Amaç:** `CGraphicThingInstance` sınıfı, bir `CGraphicThing` kaynağının oyun dünyasındaki canlı, görünen ve etkileşime giren örneğini temsil eder. Bu sınıf, birden fazla model örneğini (`CGrannyModelInstance`), Seviye Detayı (LOD) kontrolcülerini (`CGrannyLODController`), farklı durumlara (örneğin, belirli bir animasyon sırasında kullanılacak özel modeller) karşılık gelen `CGraphicThing` referanslarını ve animasyon yönetimini bir araya getirir. Temel olarak, sahnede gördüğümüz karakter, NPC, canavar veya diğer dinamik 3D nesnelerin grafiksel temsilidir.

*   **`CGraphicThingInstance` Sınıfı:**
    *   **Kalıtım:** `CGraphicObjectInstance` (Sahne grafiği, dönüşüm matrisi, görünürlük kontrolü gibi temel grafik nesne özelliklerini sağlar).
    *   **Yapısal Üyeler:**
        *   `SModelThingSet`: Bir model parçasının (örneğin, ana gövde, bir zırh parçası) farklı LOD seviyeleri için `CGraphicThing` referanslarını tutan bir yapı.
            *   `m_pLODThingRefVector`: `CGraphicThing::TRef*` (yani `CGraphicThing` referansına işaretçi) vektörü. İndeks, LOD seviyesine karşılık gelir.
    *   **Statik Üyeler ve Metotlar (Bellek Yönetimi):**
        *   `ms_kPool`: `CGraphicThingInstance` nesneleri için `CDynamicPool`.
        *   `New()`: Havuzdan yeni bir örnek alır.
        *   `Delete(CGraphicThingInstance* pkInst)`: Örneği temizleyip havuza geri bırakır.
        *   `CreateSystem()`, `DestroySystem()`: Bellek havuzunu oluşturur ve yok eder.
    *   **Temel Değişkenler:**
        *   `m_bUpdated`: Bu karenin güncellenip güncellenmediğini belirten bayrak.
        *   `m_fLastLocalTime`, `m_fLocalTime`, `m_fDelay`, `m_fSecondElapsed`, `m_fAverageSecondElapsed`: Zamanlama ve animasyon ilerlemesi ile ilgili değişkenler. `m_fDelay` muhtemelen animasyon başlangıcını geciktirmek için kullanılır.
        *   `m_fRadius`, `m_v3Center`, `m_v3Min`, `m_v3Max`: Nesnenin sınırlayıcı küre (bounding sphere) ve sınırlayıcı kutu (bounding box) bilgileri. Bunlar görünürlük testi (culling) ve basit çarpışma tespiti için kullanılır.
        *   `m_LODControllerVector`: Her bir ana model parçası (genellikle 0 indeksi ana modeldir) için bir `CGrannyLODController` işaretçisi tutan vektör. LOD yönetimi bu kontrolcüler aracılığıyla yapılır.
        *   `m_modelThingSetVector`: Model parçalarının farklı LOD seviyeleri için `CGraphicThing` referanslarını içeren `TModelThingSet` vektörü. Örneğin, `m_modelThingSetVector[0]` ana modelin LOD setini, `m_modelThingSetVector[1]` bir zırh parçasının LOD setini tutabilir.
        *   `m_roMotionThingMap`: Belirli bir hareket anahtarıyla (`DWORD`) ilişkilendirilmiş özel `CGraphicThing` referanslarını tutan bir map. Bu, örneğin karakter koşarken farklı bir modelin (belki daha düşük poligonlu veya özel efektli) kullanılmasını sağlayabilir.
    *   **Ana Metotlar ve İşlevler:**
        *   **Kurulum ve Temizleme:**
            *   `CGraphicThingInstance()`: Kurucu. `OnInitialize()` çağırır.
            *   `~CGraphicThingInstance()`: Yok edici. `Clear()` çağırır.
            *   `OnInitialize()`: Tüm üye değişkenleri varsayılan değerlerine sıfırlar.
            *   `OnClear()`: Tüm kaynakları temizler. `m_LODControllerVector`, `m_modelThingSetVector` ve `m_roMotionThingMap` içindeki tüm nesneleri ve referansları siler/serbest bırakır. `CGraphicObjectInstance::OnClear()`'ı çağırır.
            *   `ReserveModelInstance(int iCount)`: `m_LODControllerVector` için belirtilen sayıda yer ayırır ve `NULL` kontrolcüler oluşturur.
            *   `ReserveModelThing(int iCount)`: `m_modelThingSetVector` için belirtilen sayıda `TModelThingSet` oluşturur.
        *   **Kayıt (Registration) Fonksiyonları:**
            *   `RegisterModelThing(int iModelThing, CGraphicThing* pModelThing)`: Belirli bir model parçası (`iModelThing`) için temel `CGraphicThing`'i (genellikle LOD 0) kaydeder. `m_modelThingSetVector`'da ilgili `TModelThingSet`'i bulur veya oluşturur ve `pModelThing`'i LOD 0 olarak ekler.
            *   `RegisterLODThing(int iModelThing, CGraphicThing* pModelThing)`: Belirli bir model parçasının (`iModelThing`) bir sonraki LOD seviyesi için `CGraphicThing`'i kaydeder. İlgili `TModelThingSet`'in `m_pLODThingRefVector`'üne ekler.
            *   `RegisterMotionThing(DWORD dwMotionKey, CGraphicThing* pMotionThing)`: Belirli bir hareket anahtarı (`dwMotionKey`) için özel bir `CGraphicThing` kaydeder (`m_roMotionThingMap`'e ekler).
        *   **Model ve Animasyon Ayarları:**
            *   `SetModelInstance(int iDstModelInstance, int iSrcModelThing, int iSrcModel, int iSkelInstance = DONTUSEVALUE)`: Belirli bir model örneği yuvasına (`iDstModelInstance`), belirtilen model parçasından (`iSrcModelThing`) ve o parçanın içindeki modelden (`iSrcModel`) bir `CGrannyModelInstance` oluşturur ve ayarlar. `iSkelInstance`, bu modelin hangi başka model örneğinin iskeletini kullanacağını belirtir (eğer `DONTUSEVALUE` değilse). Bu fonksiyon, `m_LODControllerVector`'daki ilgili `CGrannyLODController`'ı kullanarak modeli ve LOD verilerini ayarlar.
            *   `SetMotion(DWORD dwMotionKey, float blendTime = 0.0f, int loopCount = 0, float speedRatio = 1.0f)`: Belirtilen hareket anahtarı (`dwMotionKey`) ile ilişkili animasyonu başlatır.
                *   Önce `m_roMotionThingMap`'ten bu anahtara karşılık gelen özel bir `CGraphicThing` olup olmadığına bakar. Varsa, o `Thing`'deki ilk animasyonu alır.
                *   Yoksa veya özel `Thing`'de animasyon yoksa, temel `Thing`'den (`GetBaseThingPtr()`) animasyonu alır (`GetMotionPointer(dwMotionKey)`).
                *   Bulunan animasyonu tüm LOD kontrolcülerine (`m_LODControllerVector`) `SetMotionPointer` ile uygular.
            *   `ChangeMotion(DWORD dwMotionKey, int loopCount = 0, float speedRatio = 1.0f)`: `SetMotion`'a benzer, ancak animasyonlar arası geçişi anında yapar (`ChangeMotionPointer` kullanarak).
            *   `SetEndStopMotion()`, `SetMotionAtEnd()`: Tüm LOD kontrolcülerindeki animasyonları anında sonlandırır veya son karesine ayarlar.
        *   **Güncelleme ve Deformasyon:**
            *   `OnUpdate()`: Her karede çağrılır. `m_fLocalTime`'ı `m_fSecondElapsed` ile günceller. `UpdateLODLevel()`'ı çağırır. Eğer görünürse (`isShow()`) ve gecikme (`m_fDelay`) bitmişse, `m_LODControllerVector`'daki tüm kontrolcülerin `Update()` fonksiyonunu çağırır.
            *   `OnDeform()`: Görünürse ve gecikme bitmişse, tüm LOD kontrolcülerinin `Deform()` fonksiyonunu çağırarak vertex skinning işlemini tetikler.
            *   `DeformNoSkin()`: Tüm LOD kontrolcülerinin `DeformNoSkin()` fonksiyonunu çağırır.
            *   `UpdateLODLevel()`: `m_LODControllerVector`'daki tüm kontrolcülerin `UpdateLODLevel()` fonksiyonunu çağırarak, mesafeye göre uygun LOD seviyesini seçmelerini sağlar.
            *   `UpdateTime()`: `m_fAverageSecondElapsed` hesaplaması yapar ve zamanı günceller.
        *   **Render:**
            *   `OnRender()`: Görünürse, tüm LOD kontrolcülerinin `Render()` fonksiyonunu çağırır.
            *   `OnBlendRender()`: Görünürse, tüm LOD kontrolcülerinin `BlendRender()` fonksiyonunu çağırır (saydam nesneler için).
            *   `OnRenderToShadowMap()`: Gölge haritasına çizim için tüm LOD kontrolcülerinin `RenderToShadowMap()` fonksiyonunu çağırır.
            *   `OnRenderShadow()`: Gölge çizimi için `Render()`'ı çağırır.
            *   `OnRenderPCBlocker()`: PC engelleyici (diğer oyuncuları gizleme) için `RenderPCBlocker()`'ı çağırır.
            *   `RenderWithOneTexture()`, `RenderWithTwoTexture()`, `BlendRenderWithOneTexture()`, `BlendRenderWithTwoTexture()`: İlgili render modları için tüm LOD kontrolcülerinin uygun render fonksiyonlarını çağırır.
        *   **Çarpışma ve Sınırlayıcı Kutu:**
            *   `Intersect(float* pu, float* pv, float* pt)`: Tüm LOD kontrolcülerinin `Intersect()` fonksiyonunu çağırır ve herhangi biriyle kesişim olursa `true` döner. İlk kesişen modelin sonucunu döndürür.
            *   `GetBoundBox(D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Tüm LOD kontrolcülerinin `GetBoundBox()` fonksiyonunu çağırır ve hepsini kapsayan genel bir AABB hesaplar.
            *   `GetBoundBox(DWORD dwModelInstanceIndex, D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Belirli bir model örneğinin (`dwModelInstanceIndex`) sınırlayıcı kutusunu alır.
            *   `CalculateBBox()`: Dönüştürülmüş sınırlayıcı kutuyu (`m_v3TBBoxMin`, `m_v3TBBoxMax`) hesaplar.
            *   `GetBoundingSphere()`, `GetBoundingAABB()`: Nesnenin sınırlayıcı küresini veya AABB'sini hesaplar/döndürür.
        *   **Kemik (Bone) ve Ataçlama (Attaching):**
            *   `AttachModelInstance(int iDstModelInstance, const char* c_szBoneName, int iSrcModelInstance)`: Bu örnek içindeki bir model örneğini (`iSrcModelInstance`), başka bir model örneğinin (`iDstModelInstance`) belirtilen kemiğine (`c_szBoneName`) bağlar.
            *   `AttachModelInstance(int iDstModelInstance, const char* c_szBoneName, CGraphicThingInstance& rSrcInstance, int iSrcModelInstance)`: Başka bir `CGraphicThingInstance`'ın (`rSrcInstance`) model örneğini (`iSrcModelInstance`), bu örneğin model örneğinin (`iDstModelInstance`) belirtilen kemiğine bağlar.
            *   `DetachModelInstance(int iDstModelInstance, CGraphicThingInstance& rSrcInstance, int iSrcModelInstance)`: `AttachModelInstance`'ın tersini yapar.
            *   `FindBoneIndex(int iModelInstance, const char* c_szBoneName, int* iRetBone)`: Belirli bir model örneği içinde isme göre kemik indeksi bulur.
            *   `GetBoneMatrix(DWORD dwModelInstanceIndex, DWORD dwBoneIndex, D3DXMATRIX** ppMatrix)`: Belirli bir model örneğindeki kemiğin dünya matrisini alır.
            *   `GetCompositeBoneMatrix(DWORD dwModelInstanceIndex, DWORD dwBoneIndex, D3DXMATRIX** ppMatrix)`: Belirli bir model örneğindeki kemiğin birleşik dünya matrisini alır.
        *   **Diğer:**
            *   `GetHeight()`: Nesnenin yüksekliğini hesaplar.
            *   `ReloadTexture()`: Tüm LOD kontrolcülerinin dokularını yeniden yükler.

*   **Kullanım Alanı:**
    *   `CGraphicThingInstance`, oyun dünyasındaki dinamik 3D nesnelerin merkezi yönetim birimidir.
    *   Bir veya daha fazla `CGraphicThing` kaynağını kullanarak karmaşık varlıklar oluşturur (örneğin, karakter + zırh + silah).
    *   LOD (Seviye Detayı) yönetimini `CGrannyLODController` aracılığıyla gerçekleştirir.
    *   Animasyonları yönetir ve farklı durumlara göre özel modeller kullanabilir (`m_roMotionThingMap`).
    *   Sahne grafiği içindeki konumunu, yönelimini ve ölçeğini yönetir (`CGraphicObjectInstance`'dan gelen özellikler).
    *   Render, güncelleme, deformasyon, çarpışma tespiti gibi temel işlemleri içerdiği `CGrannyLODController`'lara devreder.

### `Motion.h` ve `Motion.cpp` (`CGrannyMotion` Sınıfı)

*   **Amaç:** `CGrannyMotion` sınıfı, bir Granny 3D animasyon verisini (`granny_animation`) sarmalayan (wrap eden) basit bir sınıftır. Bu sınıf, animasyon verisine erişimi kolaylaştırır ve bazı temel animasyon özelliklerini (isim, süre, text track'ler) sorgulamak için arayüz sağlar.

*   **`CGrannyMotion` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_pgrnAni`: Sarmalanan `granny_animation` verisine işaretçi.
    *   **Ana Metotlar ve İşlevler:**
        *   `CGrannyMotion()`: Kurucu. `Initialize()` çağırır.
        *   `~CGrannyMotion()`: Yok edici. `Destroy()` çağırır (ancak `Destroy` şu anda sadece `Initialize` çağırıyor, yani aslında bir şey serbest bırakmıyor; bu, `m_pgrnAni`'nin sahipliğinin bu sınıfta olmadığını gösterir).
        *   `Initialize()`: `m_pgrnAni` işaretçisini `NULL` yapar.
        *   `Destroy()`: `Initialize()` çağırır. Gerçek animasyon verisinin silinmesinden sorumlu değildir.
        *   `IsEmpty()`: `m_pgrnAni` işaretçisinin geçerli olup olmadığını kontrol eder.
        *   `BindGrannyAnimation(granny_animation* pgrnAni)`: Verilen `granny_animation` işaretçisini `m_pgrnAni` üyesine atar. Bu sınıfın boş olduğu varsayılır (`assert(IsEmpty())`).
        *   `GetGrannyAnimationPointer() const`: Saklanan `granny_animation` işaretçisini döndürür. `CGrannyModelInstance::SetMotionPointer` gibi fonksiyonlar bu metodu kullanarak Granny animasyon verisine erişir.
        *   `GetName() const`: Animasyonun adını (`m_pgrnAni->Name`) döndürür.
        *   `GetDuration() const`: Animasyonun süresini (`m_pgrnAni->Duration`) saniye cinsinden döndürür.
        *   `GetTextTrack(const char* c_szTextTrackName, int* pCount, float* pArray) const`: Animasyon içindeki belirtilen isme (`c_szTextTrackName`) sahip metin izini (text track) arar.
            *   Animasyonun ilk track grubundaki (`m_pgrnAni->TrackGroups[0]`) metin izlerini (`pTrack->TextTracks`) dolaşır.
            *   Her metin izindeki girişleri (`rTextTrack.Entries`) kontrol eder.
            *   Eğer bir girişin metni (`rTextTrack.Entries[j].Text`) aranan isimle eşleşirse, o girişin zaman damgasını (`rTextTrack.Entries[j].TimeStamp`) verilen `pArray` dizisine ekler ve `pCount`'u artırır. Bu, animasyonun belirli zamanlarında tetiklenmesi gereken olaylar (örneğin, adım sesi, efekt başlangıcı) için kullanılabilir.

*   **Kullanım Alanı:**
    *   `CGrannyMotion`, `CGraphicThing` tarafından bir .gr2 dosyasından yüklenen her bir animasyon için oluşturulur.
    *   `CGrannyModelInstance` (veya `CGrannyLODController`), animasyonları oynatmak istediğinde, ilgili `CGrannyMotion` nesnesinden `GetGrannyAnimationPointer()` ile `granny_animation` verisini alır ve Granny 3D API'lerine (örneğin, `GrannyPlayControlledAnimation`) iletir.
    *   Animasyon süresini veya belirli olayların zamanlamasını almak için `GetDuration()` ve `GetTextTrack()` gibi metotlar kullanılır.
    *   `CGrannyMotion` sınıfı, animasyon verisinin kendisini yönetmez veya değiştirmez; sadece mevcut bir `granny_animation` verisine bir arayüz sağlar. Animasyon verisinin asıl yönetimi ve serbest bırakılması muhtemelen `CGraphicThing` veya Granny kütüphanesinin kendisi tarafından yapılır.

### `LODController.h` ve `LODController.cpp` (`CGrannyLODController` Sınıfı)

*   **Amaç:** `CGrannyLODController` sınıfı, Seviye Detayı (Level of Detail - LOD) mekanizmasını yönetmek için kullanılır. Bir grafik nesnesinin (genellikle bir `CGraphicThingInstance`'ın bir parçası) farklı mesafelerde farklı detay seviyelerine sahip modellerini (`CGrannyModelInstance`) kontrol eder. Kameraya olan uzaklığa göre en uygun modeli seçer, bu model üzerinden animasyon, güncelleme ve render işlemlerini gerçekleştirir. Ayrıca, birden fazla model örneği tarafından paylaşılabilecek deformasyon vertex buffer'larını yöneterek bellek kullanımını optimize eder.

*   **`CGrannyLODController` Sınıfı:**
    *   **Kalıtım:** `CGraphicBase` (Temel grafik sınıfı).
    *   **Statik Üyeler ve Metotlar:**
        *   `ms_isMinLODModeEnable`: Minimum LOD modunu etkinleştirmek için statik bir bayrak. Etkinleştirildiğinde, mesafe ne olursa olsun en düşük detay seviyesi kullanılır (performans testi veya düşük sistemler için olabilir).
        *   `SetMinLODMode(bool isEnable)`: `ms_isMinLODModeEnable` bayrağını ayarlar.
        *   `gs_vbs[]`, `gs_emptyVB`: Paylaşımlı deformasyon vertex buffer'larını (`CGraphicVertexBuffer*`) yönetmek için kullanılan statik diziler ve boş bir vertex buffer.
        *   `__AllocDeformVertexBuffer()`, `__FreeDeformVertexBuffer()`: Belirli bir vertex sayısına uygun paylaşımlı bir vertex buffer'ı havuzdan alır veya havuza geri bırakır.
        *   `__ReserveSharedVertexBuffers()`: Belirli bir boyutta belirli sayıda paylaşımlı vertex buffer'ı önceden oluşturur.
        *   `GrannyCreateSharedDeformBuffer()`, `GrannyDestroySharedDeformBuffer()`: Oyun başlangıcında paylaşımlı buffer havuzunu oluşturur ve oyun sonunda yok eder.
    *   **Yapısal Üyeler (Functor'lar):**
        *   `FSetLocalTime`, `FUpdateTime`, `FUpdateLODLevel`, `FRenderWithOneTexture`, `FBlendRenderWithOneTexture`, `FRenderWithTwoTexture`, `FBlendRenderWithTwoTexture`, `FRenderToShadowMap`, `FRenderShadow`, `FDeform`, `FDeformNoSkin`, `FDeformAll`, `FCreateDeviceObjects`, `FDestroyDeviceObjects`, `FBoundBox`, `FResetLocalTime`, `FReloadTexture`, `FSetMotionPointer`, `FChangeMotionPointer`, `FEndStopMotionPointer`: Bu yapılar (functor), genellikle `std::for_each` ile birlikte kullanılmak üzere tasarlanmıştır. Bir `CGrannyLODController` işaretçisi alıp ilgili fonksiyonu çağıran `operator()` içerirler. Bu, `CGraphicThingInstance`'ın sahip olduğu `m_LODControllerVector`'daki tüm kontrolcüler üzerinde kolayca işlem yapılmasını sağlar.
    *   **Temel Değişkenler:**
        *   `m_que_pkModelInst`: Bu LOD kontrolcüsünün yönettiği farklı detay seviyelerindeki `CGrannyModelInstance` işaretçilerini tutan bir `std::deque`. İndeks 0 genellikle en yüksek detay seviyesidir.
        *   `m_pCurrentModelInstance`: Şu anda aktif (render edilen ve güncellenen) olan `CGrannyModelInstance`'a işaretçi.
        *   `m_bLODLevel`: Mevcut aktif olan LOD seviyesinin indeksi.
        *   `m_fLODDistance`: Bu kontrolcünün LOD hesaplaması için kullanılan merkezden uzaklığı.
        *   `m_dwLODAniFPS`: Bu LOD seviyesindeki animasyonun FPS sınırı.
        *   `m_pAttachedParentModel`: Eğer bu LOD kontrolcüsü başka bir LOD kontrolcüsüne (ebeveyn) bağlıysa (örneğin, bir zırh modelinin ana karakter modeline bağlanması gibi), ebeveyn kontrolcüye işaret eder.
        *   `m_AttachedModelDataVector`: Bu LOD kontrolcüsüne bağlı olan diğer LOD kontrolcülerinin (`pkLODController`) ve hangi kemiğe (`strBoneName`) bağlandıklarının bilgisini tutan bir vektör (`TAttachingModelData`).
        *   `m_pkSharedDeformableVertexBuffer`: Bu LOD kontrolcüsünün modelleri için ayrılmış (ve potansiyel olarak paylaşılan) deformasyon vertex buffer'ına işaretçi.
    *   **Ana Metotlar ve İşlevler:**
        *   **Kurulum ve Temizleme:**
            *   `CGrannyLODController()`: Kurucu. Üyeleri varsayılan değerlere ayarlar.
            *   `~CGrannyLODController()`: Yok edici. Paylaşımlı vertex buffer'ı serbest bırakır (`__FreeDeformVertexBuffer`) ve `Clear()`'ı çağırır.
            *   `Clear()`: Yönetilen tüm `CGrannyModelInstance`'ları siler (`m_que_pkModelInst`), bağlı modellerle ilişkisini keser (`m_AttachedModelDataVector`, `m_pAttachedParentModel`) ve üyeleri sıfırlar.
        *   **Model Ekleme ve Yönetimi:**
            *   `AddModel(CGraphicThing* pThing, int iSrcModel, CGrannyLODController* pSkelLODController = NULL)`: Verilen `CGraphicThing`'den (`pThing`) belirtilen modeli (`iSrcModel`) bu LOD kontrolcüsüne yeni bir LOD seviyesi olarak ekler.
                *   Yeni bir `CGrannyModelInstance` oluşturur.
                *   Modelin deformasyon vertex sayısına göre paylaşımlı bir vertex buffer ayırır (`__ReserveSharedDeformableVertexBuffer`).
                *   Eğer `pSkelLODController` verilmişse, modeli o kontrolcünün iskeletine bağlar (`SetLinkedModelPointer`), aksi halde kendi iskeletini kullanır (`SetMainModelPointer`).
                *   Oluşturulan `CGrannyModelInstance`'ı `m_que_pkModelInst` deque'sine ekler.
                *   Eğer bu eklenen ilk modelse (`m_que_pkModelInst.size() == 1`), onu `m_pCurrentModelInstance` olarak ayarlar.
            *   `__ReserveSharedDeformableVertexBuffer(DWORD deformableVertexCount)`: Belirtilen sayıda deforme olabilir vertex için uygun boyutta paylaşımlı bir vertex buffer'ı `__AllocDeformVertexBuffer` ile alır ve `m_pkSharedDeformableVertexBuffer`'a atar (eğer daha önce atanmamışsa).
        *   **Ataçlama (Attaching):**
            *   `AttachModelInstance(CGrannyLODController* pSrcLODController, const char* c_szBoneName)`: Başka bir LOD kontrolcüsünü (`pSrcLODController`) bu kontrolcünün belirtilen kemiğine (`c_szBoneName`) bağlar. Bağlanan kontrolcüyü `m_AttachedModelDataVector`'e ekler ve bağlı kontrolcünün `m_pAttachedParentModel`'ini ayarlar. Ayrıca, aktif model örneği üzerinden `RefreshAttachedModelInstance` çağırarak Granny seviyesinde ataçlamayı gerçekleştirir.
            *   `DetachModelInstance(CGrannyLODController* pSrcLODController)`: Verilen kontrolcünün ataçlamasını kaldırır. `m_AttachedModelDataVector`'den çıkarır ve `m_pAttachedParentModel`'ini sıfırlar.
            *   `RefreshAttachedModelInstance()`: `m_pCurrentModelInstance` (aktif model örneği) üzerinden, `m_AttachedModelDataVector`'deki tüm bağlı kontrolcülerin aktif model örneklerini doğru kemiklere bağlar (`SetParentModelInstance`).
        *   **LOD Yönetimi:**
            *   `SetLODLimits(float fNearLOD, float fFarLOD)`: LOD geçişleri için yakın ve uzak mesafe limitlerini ayarlar (ancak mevcut kodda bu parametreler kullanılmıyor gibi görünüyor). `m_fLODDistance`'ı `fFarLOD`'a ayarlar.
            *   `SetLODLevel(BYTE bLODLevel)`: LOD seviyesini manuel olarak ayarlar. Verilen `bLODLevel` indeksindeki modeli `m_que_pkModelInst`'ten alıp `m_pCurrentModelInstance` yapar ve bağlı modelleri günceller (`RefreshAttachedModelInstance`).
            *   `UpdateLODLevel(float fDistanceFromCenter, float fDistanceFromCamera)`: Kameraya olan mesafeye göre uygun LOD seviyesini otomatik olarak belirler ve `SetLODLevel`'i çağırır.
                *   `ms_isMinLODModeEnable` aktifse, en düşük LOD seviyesini (`m_que_pkModelInst.size() - 1`) seçer.
                *   Aksi halde, `m_fLODDistance` (genellikle `SetLODLimits` ile ayarlanan uzak limit) ve `fDistanceFromCamera` (kameradan gerçek uzaklık) kullanarak bir LOD seviyesi hesaplar. Daha uzaktaki nesneler için daha yüksek indeksli (daha düşük detaylı) modeller seçilir.
            *   `GetLODLevel()`: Mevcut aktif LOD seviyesini (`m_bLODLevel`) döndürür.
        *   **Güncelleme, Render, Deformasyon vb. (Yönlendirme):**
            *   `Update(float fElapsedTime, float fDistanceFromCenter, float fDistanceFromCamera)`: Önce `UpdateLODLevel` ile doğru modeli seçer, sonra `UpdateTime` ile aktif modelin zamanını günceller.
            *   `UpdateTime(float fElapsedTime)`: Aktif model örneğinin (`m_pCurrentModelInstance`) `UpdateLocalTime` ve `Update` fonksiyonlarını çağırır.
            *   `UpdateSkeleton(const D3DXMATRIX* c_pWorldMatrix, float fElapsedTime)`: Aktif model örneğinin `UpdateTransform` fonksiyonunu çağırır.
            *   `Deform(const D3DXMATRIX* c_pWorldMatrix)`, `DeformNoSkin(const D3DXMATRIX* c_pWorldMatrix)`, `DeformAll(const D3DXMATRIX* c_pWorldMatrix)`: Aktif model örneğinin ilgili deformasyon fonksiyonunu çağırır.
            *   `RenderWithOneTexture()`, `RenderWithTwoTexture()`, `BlendRenderWithOneTexture()`, `BlendRenderWithTwoTexture()`, `RenderToShadowMap()`, `RenderShadow()`: Aktif model örneğinin ilgili render fonksiyonunu çağırır.
            *   `ReloadTexture()`: Aktif model örneğinin `ReloadTexture` fonksiyonunu çağırır.
            *   `GetBoundBox(D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Aktif model örneğinin `GetBoundBox` fonksiyonunu çağırır.
            *   `Intersect(const D3DXMATRIX* c_pMatrix, float* u, float* v, float* t)`: Aktif model örneğinin `Intersect` fonksiyonunu çağırır.
            *   `SetLocalTime(float fLocalTime)`, `ResetLocalTime()`, `SetMotionPointer()`, `ChangeMotionPointer()`, `SetMotionAtEnd()`: Aktif model örneğinin ilgili zaman ve animasyon kontrol fonksiyonlarını çağırır.
        *   **Diğer:**
            *   `CreateDeviceObjects()`, `DestroyDeviceObjects()`: Aktif model örneğinin ilgili device object fonksiyonlarını çağırır.
            *   `isModelInstance()`: Aktif bir model örneğinin (`m_pCurrentModelInstance`) olup olmadığını kontrol eder.
            *   `GetModelInstance()`: Aktif model örneğini döndürür.
            *   `HaveBlendThing()`: Aktif model örneğinin saydam materyal içerip içermediğini kontrol eder.

*   **Kullanım Alanı:**
    *   `CGrannyLODController`, `CGraphicThingInstance` içinde her bir model parçası (gövde, zırh vb.) için kullanılır.
    *   Uzaklığa göre otomatik olarak model detay seviyesini ayarlar, böylece uzaktaki nesneler daha az poligonlu modellerle çizilerek performans artışı sağlanır.
    *   Bir `CGraphicThingInstance`'a ait tüm LOD kontrolcüleri, `std::for_each` ve yukarıda bahsedilen Functor yapıları kullanılarak toplu olarak güncellenir veya render edilir.
    *   Paylaşımlı deformasyon vertex buffer mekanizması, aynı anda ekranda çok sayıda benzer karakter olduğunda bellek kullanımını önemli ölçüde azaltır.
    *   Model ataçlama (`AttachModelInstance`) mekanizması, farklı kontrolcülere ait modellerin birbirine bağlanmasını (örneğin, bir zırhın karaktere bağlanması) ve iskeletlerini paylaşmasını sağlar.
