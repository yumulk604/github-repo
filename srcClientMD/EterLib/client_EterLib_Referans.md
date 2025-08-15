# EterLib Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/EterLib/` klasöründe bulunan `EterLib` modülünün amacını, içerdiği temel bileşenleri ve bu bileşenlerin işlevlerini açıklamaktadır.

`EterLib`, Metin2 istemcisinin en temel ve kapsamlı kütüphanelerinden biridir. Genellikle oyun motorunun alt seviye fonksiyonlarını, temel veri yapılarını, kaynak yönetimini, Direct3D ile etkileşim için sarmalayıcı (wrapper) sınıfları, ağ iletişiminin bazı bileşenlerini, temel matematik ve yardımcı fonksiyonları, ve kullanıcı arayüzü (UI) olmayan çeşitli sistemleri içerir. `EterBase` üzerine kurulu olup, diğer birçok kütüphane (örneğin `EterGrnLib`, `EterPythonLib`, `UserInterface`) için temel oluşturur veya onlarla sıkı bir etkileşim içindedir.

## Genel Amaç ve İçerik

`EterLib` modülünün geniş bir sorumluluk alanı vardır ve aşağıdaki gibi çeşitli temel görevler için araçlar sunar:

*   **Kaynak Yönetimi:** Oyun içi varlıkların (modeller, dokular, sesler vb.) yüklenmesi, yönetilmesi ve serbest bırakılması.
*   **Grafik Sarmalayıcıları (Graphics Wrappers):** Direct3D nesneleri (Vertex Buffer, Index Buffer, Texture, Shader vb.) için daha kullanıcı dostu ve yönetilebilir sınıflar.
*   **Render Durum Yönetimi:** Direct3D render state'lerinin (örneğin, karıştırma modları, Z-buffer ayarları, doku filtreleme) yönetimi için bir mekanizma.
*   **Matematik ve Geometri:** Vektörler, matrisler, ışınlar (rays), sınırlayıcı kutular (bounding boxes) gibi temel matematiksel işlemler ve geometrik testler.
*   **Temel Veri Yapıları ve Algoritmalar:** Havuzlar (Pools), referans sayımı (Reference Counting), ağaçlar, listeler gibi genel amaçlı veri yapıları.
*   **Ağ İletişimi:** Düşük seviye ağ soketleri, paket başlıkları, veri akışları (streams) gibi ağ iletişiminin temel bileşenleri.
*   **Giriş (Input) Yönetimi:** Klavye ve fare girdilerinin işlenmesi.
*   **Zamanlama ve Profilleme:** Oyun içi zaman yönetimi ve performans profilleme araçları.
*   **Yardımcı Fonksiyonlar:** String işleme, dosya işlemleri, hata ayıklama gibi genel yardımcı fonksiyonlar.
*   **Grafiksel Efektler ve Bileşenler:** SkyBox, Lens Flare, Screen Filter gibi bazı temel grafiksel bileşenler.
*   **Metin İşleme ve Gösterimi:** Temel metin oluşturma ve render etme yetenekleri.

Bu modül, istemcinin kararlı ve verimli çalışması için hayati öneme sahip birçok temel sistemi barındırır.

---

## Dosya Bazlı Detaylandırma

### `StdAfx.h` ve `StdAfx.cpp`

*   **Amaç:** Bu dosyalar, `EterLib` projesi için ön derlenmiş başlık (PCH - Precompiled Header) mekanizmasını oluşturur. `EterLib` için gerekli olan çok sayıda standart sistem başlığını (Windows, Direct3D 8, DirectInput, Winsock vb.), proje içi temel başlıkları (`EterBase/StdAfx.h`, `EterBase/Debug.h`, `UserInterface/Locale_inc.h`, `EterLocale/CodePageId.h`) ve kütüphane bağımlılıklarını (`winmm.lib`, `d3d8.lib`, `d3dx8.lib`) içerir.

*   **Temel Özellikler/İçerik (`StdAfx.h`):**
    *   `#pragma once`: Başlık dosyasının tek bir derleme biriminde yalnızca bir kez dahil edilmesini sağlar.
    *   `WIN32_LEAN_AND_MEAN`: Windows başlık dosyalarından daha az kullanılan API'lerin çıkarılmasını sağlayarak derleme süresini azaltır.
    *   `_CRT_SECURE_NO_WARNINGS`: Güvenli olmayan CRT fonksiyonlarıyla ilgili uyarıları devre dışı bırakır.
    *   Çeşitli `#pragma warning(disable: ...)` direktifleri: Belirli derleyici uyarılarını (4710 - inlined olmayan fonksiyon, 4786 - uzun sembol adları, 4244 - veri kaybı olası tür dönüşümü vb.) devre dışı bırakır.
    *   **Ana Bağımlılıklar ve Dahil Edilenler:**
        *   `../UserInterface/Locale_inc.h`: Yerelleştirme ayarları.
        *   `<d3d8.h>`, `<d3dx8.h>`: Direct3D 8 ve D3DX8 yardımcı kütüphanesi başlıkları.
        *   `<dinput.h>`: DirectInput başlığı (giriş aygıtları için).
        *   `<mmsystem.h>`: Multimedya zamanlayıcıları ve diğer sistem fonksiyonları.
        *   `<process.h>`: Thread ve process yönetimi.
        *   `<stdio.h>`, `<math.h>`, `<time.h>`, `<direct.h>`, `<malloc.h>`: Standart C kütüphaneleri.
        *   `../EterBase/StdAfx.h`, `../EterBase/Debug.h`: `EterBase` modülünden temel tanımlar ve hata ayıklama araçları.
        *   `../EterLocale/CodePageId.h`: Karakter kod sayfası tanımları.
        *   `<winsock.h>`: Windows Sockets API (ağ iletişimi için).
    *   **Kütüphane Bağlantıları (`#pragma comment(lib, ...)`):**
        *   `winmm.lib`: Windows Multimedia kütüphanesi.
        *   `d3d8.lib`: Direct3D 8 kütüphanesi.
        *   `d3dx8.lib`: D3DX8 yardımcı kütüphanesi.

*   **İçerik (`StdAfx.cpp`):**
    *   `#include "stdafx.h"`: Ön derlenmiş başlık dosyasını dahil eder. Bu dosyanın kendisi `.pch` dosyasının oluşturulmasını tetikler.

*   **Kullanım Alanı:**
    *   Bu dosyalar, `EterLib` içindeki diğer tüm kaynak dosyaların derlenmesinden önce işlenir.
    *   Sık kullanılan ve nadiren değişen başlıkları tek bir yerde toplayarak ve önceden derleyerek tüm projenin derleme süresini önemli ölçüde kısaltır.
    *   Proje genelinde kullanılacak temel kütüphaneleri (Direct3D, DirectInput vb.) ve ayarları (uyarıları devre dışı bırakma gibi) tanımlar.

### `Util.h` ve `Util.cpp`

*   **Amaç:** Bu dosyalar, `EterLib` içinde çeşitli genel amaçlı yardımcı fonksiyonları, bir template sınıfını (`CTransitor`) ve özellikle metin tabanlı yapılandırma dosyalarını okuma, font ve karakter kod sayfası (code page) yönetimi ile ilgili işlevleri barındırır.

*   **`CTransitor<T>` Template Sınıfı (`Util.h`):**
    *   **Amaç:** Bir değerin (`T` tipi) belirli bir süre içinde bir başlangıç değerinden (`m_SourceValue`) bir hedef değere (`m_TargetValue`) doğru yumuşak bir geçiş (interpolasyon) yapmasını sağlar.
    *   **Tür Takma Adları (Typedefs):**
        *   `TTransitorFloat`: `float` değerler için.
        *   `TTransitorVector3`: `D3DXVECTOR3` (3D vektör) değerleri için.
        *   `TTransitorColor`: `D3DXCOLOR` (renk) değerleri için.
    *   **Temel Metotlar:**
        *   `SetActive(BOOL bActive)`: Geçişin aktif olup olmadığını ayarlar.
        *   `isActive()`: Aktif olup olmadığını döndürür.
        *   `isActiveTime(float fcurTime)`: Verilen zamanın (`fcurTime`) geçiş süresi içinde olup olmadığını kontrol eder.
        *   `SetID(DWORD dwID)`, `GetID()`: Geçişe bir kimlik atamak ve almak için.
        *   `SetSourceValue(const T& c_rSourceValue)`: Sadece başlangıç değerini ayarlar.
        *   `SetTransition(const T& c_rSourceValue, const T& c_rTargetValue, float fStartTime, float fBlendTime)`: Başlangıç ve hedef değerlerini, başlangıç zamanını ve geçiş süresini ayarlar.
        *   `GetValue(float fcurTime, T* pValue)`: Verilen zamandaki (`fcurTime`) interpole edilmiş değeri hesaplar ve `pValue`'a yazar. Zaman, başlangıç zamanından küçükse `false` döner.
    *   **Kullanım Alanı:** Animasyonlarda, kullanıcı arayüzü elemanlarının hareketlerinde veya herhangi bir değerin zamanla yumuşak bir şekilde değişmesi gereken durumlarda kullanılabilir (örneğin, bir rengin yavaşça solması, bir nesnenin pozisyonunun değişmesi).

*   **Metin Dosyası İşleme Fonksiyonları (`Util.h` ve `Util.cpp`):**
    *   `PrintfTabs(FILE* File, int iTabCount, const char* c_szString, ...)`: Verilen dosyaya, belirtilen sayıda sekme (tab) karakteri ekledikten sonra formatlı bir string yazar. Genellikle yapılandırılmış log veya debug çıktıları oluşturmak için kullanılır.
    *   `LoadTextData(const char* c_szFileName, CTokenMap& rstTokenMap)`: Belirtilen dosyadan (EterPack içinden `CEterPackManager` ile okunur) metin verilerini yükler. Her satırın iki token'dan oluştuğunu varsayar (bir anahtar ve bir değer). Token'ları küçük harfe çevirir ve `rstTokenMap` (bir `std::map<std::string, std::string>`) içine kaydeder.
    *   `LoadMultipleTextData(const char* c_szFileName, CTokenVectorMap& rstTokenVectorMap)`: Daha karmaşık metin dosyalarını yükler. "start" ve "end" bloklarını destekler. Bir anahtar ve bu anahtara karşılık gelen birden fazla string token'ını (`CTokenVector`, yani `std::vector<std::string>`) `rstTokenVectorMap` (`std::map<std::string, CTokenVector>`) içine kaydeder.
    *   `TokenToVector(CTokenVector& rVector)`: 3 elemanlı bir `CTokenVector`'ü (`std::string` vektörü) `D3DXVECTOR3`'e dönüştürür (`atof` kullanarak).
    *   `TokenToColor(CTokenVector& rVector)`: 4 elemanlı bir `CTokenVector`'ü `D3DXCOLOR`'a dönüştürür.
    *   `GOTO_CHILD_NODE` Makrosu: `CTextFileLoader::CGotoChild` nesnesi oluşturarak `CTextFileLoader` içinde belirli bir alt düğüme geçmeyi kolaylaştırır (RAII prensibiyle kapsamdan çıkıldığında geri dönülür).

*   **Font ve Karakter Kod Sayfası (Code Page) Yönetimi (`Util.h` ve `Util.cpp`):**
    *   `gs_fontFace`, `gs_codePage`: Global statik değişkenler, varsayılan font yüzünü ve kod sayfasını tutar.
    *   `EnumFontFamExProc(CONST LOGFONT* plogFont, ..., LPARAM lParam)`: Windows API `EnumFontFamiliesEx` için bir callback fonksiyonu. Belirtilen font adının (`lParam`) sistemde olup olmadığını kontrol eder.
    *   `GetCharsetFromCodePage(WORD codePage)`: Verilen Windows kod sayfasına karşılık gelen karakter setini (örneğin, `SHIFTJIS_CHARSET`, `HANGUL_CHARSET`) döndürür.
    *   `GetFontFaceFromCodePageNT(WORD codePage)` ve `GetFontFaceFromCodePage9x(WORD codePage)`: Belirli bir kod sayfasına göre Windows NT ve Windows 9x sistemleri için varsayılan/uygun font yüzlerini (font adlarını) döndürür. (Not: 9x fonksiyonundaki bazı font adları bozuk karakterlerle görünüyor, muhtemelen kodlama sorunu.)
    *   `GetDefaultCodePage()`: Mevcut varsayılan kod sayfasını (`gs_codePage`) döndürür.
    *   `GetDefaultFontFace()`: Mevcut varsayılan font yüzünü (`gs_fontFace`) döndürür.
    *   `GetFontFaceFromCodePage(WORD codePage)`: İşletim sistemi sürümünü kontrol ederek (`IsWindows9x()`, `IsWindowsNT()`) uygun `GetFontFaceFromCodePageNT` veya `GetFontFaceFromCodePage9x` fonksiyonunu çağırır.
    *   `SetDefaultFontFace(const char* fontFace)`: `gs_fontFace` global değişkenini ayarlar.
    *   `SetDefaultCodePage(DWORD codePage)`: `gs_codePage` global değişkenini ayarlar ve `gs_fontFace`'i de bu kod sayfasına uygun olanla günceller.
    *   `base64_decode(const char* str, char* resultStr)`: Base64 ile kodlanmış bir string'i çözer. (Yardımcı fonksiyonları `__base64_get` ve `__strcat1` kullanılır.)

*   **Diğer Yardımcı Fonksiyonlar (`Util.h` ve `Util.cpp`):**
    *   `GetMaxTextureWidth()` ve `GetMaxTextureHeight()`: Muhtemelen grafik kartının desteklediği maksimum doku genişliğini ve yüksekliğini döndürmek için tasarlanmıştır, ancak `Util.cpp`'nin sağlanan kısmında bu fonksiyonların implementasyonları görünmüyor. (Not: Bu fonksiyonlar `EterGrnLib/Util.cpp` içinde implemente edilmiş olabilir ve burada sadece bildirimleri kalmış olabilir.)

*   **Kullanım Alanı:**
    *   `CTransitor`: Oyun içi yumuşak geçiş efektleri ve animasyonlar için kullanılır.
    *   Metin dosyası yükleme fonksiyonları: Oyun ayarları, scriptler veya diğer yapılandırma verilerini içeren basit metin dosyalarını okumak için kullanılır.
    *   Font ve kod sayfası fonksiyonları: Uluslararasılaştırma ve yerelleştirme için, farklı dillerdeki metinlerin doğru font ve karakter setiyle görüntülenmesini sağlamak amacıyla kullanılır. İstemcinin çalıştığı dil ve işletim sistemine göre uygun fontları seçmeye yardımcı olur.
    *   `base64_decode`: Ağ üzerinden gelen veya dosyalarda saklanan base64 kodlu verileri çözmek için kullanılabilir.

### `ReferenceObject.h` ve `ReferenceObject.cpp` (`CReferenceObject` Sınıfı)

*   **Amaç:** `CReferenceObject` sınıfı, paylaşılan nesnelerin yaşam döngüsünü yönetmek için temel bir referans sayma (reference counting) mekanizması sunar. Bir nesneye kaç tane başka nesnenin referans verdiğini takip eder ve referans sayısı sıfıra düştüğünde nesnenin kendi kendini yok etmesini sağlar.

*   **`CReferenceObject` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_refCount`: Nesneye olan aktif referansların sayısını tutar.
        *   `m_destructed`: Nesnenin yok edilip edilmediğini belirten bir bayrak (aslında `OnSelfDestruct` içinde `true` yapılıyor ama asıl kontrol `m_refCount` üzerinden).
    *   **Kurucu ve Yok Edici:**
        *   `CReferenceObject()`: Kurucu. `m_refCount`'ı 0, `m_destructed`'ı `false` olarak başlatır.
        *   `virtual ~CReferenceObject()`: Sanal yok edici. Bu, `CReferenceObject`'ten türetilen sınıfların da doğru şekilde yok edilmesini sağlar.
    *   **Ana Metotlar ve İşlevler:**
        *   `AddReference()`: Nesneye bir referans ekler. Referans sayısını (`m_refCount`) artırır. Eğer bu ilk referanssa (`m_refCount == 0` iken çağrılırsa), `OnConstruct()` sanal metodunu çağırır.
        *   `AddReferenceOnly()`: Sadece referans sayısını artırır. `OnConstruct()`'ı çağırmaz. Bu, nesne zaten oşturulmuş ve referans sayısının manuel olarak artırılması gerektiği özel durumlar için olabilir.
        *   `Release()`: Nesneden bir referans kaldırır. Referans sayısını azaltır.
            *   Eğer referans sayısı 1'den büyükse, sadece sayacı azaltır.
            *   Eğer referans sayısı 1 veya daha az ise (yani bu son referans ise), `m_refCount`'ı 0 yapar ve `OnSelfDestruct()` sanal metodunu çağırır.
        *   `GetReferenceCount()`: Mevcut referans sayısını döndürür.
        *   `canDestroy()`: Nesnenin yok edilip edilemeyeceğini kontrol eder. Eğer referans sayısı 0 ise `true` döner.
    *   **Sanal Metotlar (Virtual Methods):**
        *   `virtual void OnConstruct()`: Nesneye ilk referans eklendiğinde (`AddReference` içinde) çağrılır. Türetilmiş sınıflar, nesne ilk kez kullanıma alındığında yapılması gereken özel başlatma işlemlerini burada yapabilir. Varsayılan implementasyonda `m_destructed`'ı `false` yapar.
        *   `virtual void OnSelfDestruct()`: Nesneye olan son referans kaldırıldığında (`Release` içinde) çağrılır. Türetilmiş sınıflar, nesne yok edilmeden önce yapılması gereken özel temizleme işlemlerini burada yapabilir. Varsayılan implementasyonda `m_destructed`'ı `true` yapar ve ardından `delete this;` çağırarak nesneyi bellekten siler.

*   **Kullanım Alanı:**
    *   `CReferenceObject`, `CResource` gibi birçok temel sınıfın ebeveyni olarak kullanılır.
    *   Oyun içinde dokular, modeller, sesler gibi paylaşılan kaynakların yönetiminde kritik bir rol oynar. Bir kaynak birden fazla yerde kullanılıyorsa, her kullanım bir referans ekler. Kaynak artık hiçbir yerde kullanılmıyorsa (referans sayısı sıfıra düşerse), kaynak otomatik olarak bellekten temizlenir.
    *   Bu mekanizma, "dangling pointers" (artık geçerli olmayan bir bellek alanını gösteren işaretçiler) ve bellek sızıntıları (artık kullanılmayan ama serbest bırakılmamış bellek) gibi sorunların önlenmesine yardımcı olur.

### `Resource.h` ve `Resource.cpp` (`CResource` Sınıfı)

*   **Amaç:** `CResource` sınıfı, `CReferenceObject`'ten türeyen soyut bir temel sınıftır. Oyun içinde kullanılan çeşitli kaynakların (örneğin, dokular, modeller, ses dosyaları) genel yönetimini, yüklenmesini ve yaşam döngüsünü kontrol etmek için bir arayüz ve temel altyapı sağlar. `CResourceManager` tarafından yönetilir.

*   **`CResource` Sınıfı:**
    *   **Kalıtım:** `public CReferenceObject`
    *   **`TType` (typedef `DWORD`):** Kaynak türlerini tanımlamak için kullanılan bir tür takma adıdır. Genellikle `StringToType` ile bir string'in CRC32 hash'i olarak üretilir.
    *   **Enum `EState`:** Kaynağın durumunu belirtir:
        *   `STATE_EMPTY`: Kaynak boş, henüz yüklenmemiş veya temizlenmiş.
        *   `STATE_ERROR`: Kaynak yüklenirken bir hata oluştu.
        *   `STATE_EXIST`: Kaynak başarıyla yüklendi ve mevcut.
        *   `STATE_LOAD` (kullanılmıyor gibi): Muhtemelen yükleme aşamasını belirtmek için düşünülmüş ama mevcut kodda aktif olarak kullanılmıyor.
        *   `STATE_FREE` (kullanılmıyor gibi): Muhtemelen serbest bırakılma aşamasını belirtmek için düşünülmüş.
    *   **Temel Değişkenler:**
        *   `m_stFileName`: Kaynağın dosya adını tutan `std::string`.
        *   `m_dwLoadCostMiliiSecond`: Kaynağın yüklenmesinin milisaniye cinsinden maliyeti.
        *   `me_state`: Kaynağın mevcut durumunu (`EState`) tutar.
        *   `ms_bDeleteImmediately` (static bool): Kaynaklar serbest bırakıldığında hemen silinip silinmeyeceğini belirleyen statik bir bayrak. `false` ise, `CResourceManager` tarafından silinmek üzere rezerve edilir.
    *   **Kurucu ve Yok Edici:**
        *   `CResource(const char* c_szFileName)`: Kurucu. Dosya adını ayarlar ve durumu `STATE_EMPTY` olarak başlatır.
        *   `virtual ~CResource()`: Sanal yok edici.
    *   **Ana Metotlar ve İşlevler:**
        *   `Clear()`: `OnClear()` sanal metodunu çağırır ve kaynağın durumunu `STATE_EMPTY` yapar.
        *   `static TType StringToType(const char* c_szType)`: Verilen string'in CRC32 hash'ini hesaplayarak kaynak türü (`TType`) üretir.
        *   `static TType Type()`: `CResource` sınıfının kendi türünü (`StringToType("CResource")`) döndürür.
        *   `Load()`: Kaynağı yükler. Eğer durumu `STATE_EMPTY` değilse bir şey yapmaz.
            *   `CEterPackManager::Instance().Get()` ile dosyayı EterPack içinden okumaya çalışır.
            *   Başarılı olursa, `OnLoad()` sanal metodunu çağırarak türetilmiş sınıfın asıl yükleme işlemini yapmasını sağlar.
            *   `OnLoad` başarılı olursa durumu `STATE_EXIST`, başarısız olursa `STATE_ERROR` yapar.
            *   Dosya bulunamazsa da `OnLoad(0, NULL)` çağrılır (bazı kaynaklar dosyasız oluşturulabilir).
        *   `Reload()`: Kaynağı yeniden yükler. Önce `Clear()` ile temizler, sonra `Load()`'a benzer şekilde dosyayı okuyup `OnLoad()`'ı çağırır.
        *   `ConvertPathName(const char* c_szPathName, char* pszRetPathName, int retLen)`: Verilen dosya yolundaki '/' karakterlerini '\\' ile değiştirir ve tüm karakterleri küçük harfe (`korean_tolower`) çevirir. Sonucu `pszRetPathName`'e yazar.
        *   `CreateDeviceObjects()`: Türetilmiş sınıfların grafik cihazıyla ilişkili nesneleri (örneğin, Direct3D dokuları, buffer'ları) oluşturması için sanal bir metot. Varsayılan implementasyonu `true` döner.
        *   `DestroyDeviceObjects()`: `CreateDeviceObjects` ile oluşturulan cihaz nesnelerini yok etmek için sanal bir metot.
        *   `SetDeleteImmediately(bool isSet)`: `ms_bDeleteImmediately` statik bayrağını ayarlar.
        *   `IsData() const`: Kaynağın boş olup olmadığını (`me_state != STATE_EMPTY`) kontrol eder.
        *   `IsEmpty() const`: `OnIsEmpty()` sanal metodunu çağırır.
        *   `IsType(TType type)`: `OnIsType()` sanal metodunu çağırır.
        *   `GetLoadCostMilliSecond()`: Yükleme süresini döndürür.
        *   `GetFileName() const`, `GetFileNameString() const`: Dosya adını döndürür.
    *   **Soyut Sanal Metotlar (Pure Virtual yerine `= 0` ile işaretlenmiş ama implementasyonu olanlar var, daha çok arayüz zorunluluğu):**
        *   `virtual bool OnLoad(int iSize, const void* c_pvBuf) = 0`: Türetilmiş sınıflar bu metodu implemente ederek asıl veri yükleme işlemini gerçekleştirir. `iSize` tampon boyutu, `c_pvBuf` ise veri tamponuna işaretçidir.
        *   `virtual void OnClear() = 0`: Türetilmiş sınıflar bu metodu implemente ederek kaynak temizlendiğinde yapılması gereken işlemleri (bellek serbest bırakma vb.) yapar.
        *   `virtual bool OnIsEmpty() const = 0`: Türetilmiş sınıflar bu metodu implemente ederek kaynağın gerçekten boş olup olmadığını (örneğin, iç veri yapılarının durumu) kontrol eder.
        *   `virtual bool OnIsType(TType type) = 0`: Türetilmiş sınıflar bu metodu implemente ederek kendi türlerini kontrol eder ve genellikle `CResource::OnIsType`'ı da çağırarak temel sınıf kontrolünü de yapar.
    *   **Override Edilmiş `CReferenceObject` Sanal Metotları:**
        *   `virtual void OnConstruct()`: Kaynağa ilk referans eklendiğinde çağrılır. `Load()` metodunu çağırarak kaynağın yüklenmesini tetikler.
        *   `virtual void OnSelfDestruct()`: Son referans kaldırıldığında çağrılır. Eğer `ms_bDeleteImmediately` `true` ise `Clear()` çağırır, değilse `CResourceManager::Instance().ReserveDeletingResource(this)` ile kaynağı silinmek üzere yöneticiye bildirir.

*   **Kullanım Alanı:**
    *   `CResource`, `EterLib` ve diğer kütüphanelerdeki tüm yönetilen kaynaklar için (örneğin, `CGraphicImage` (doku), `CGrannyModel` (model), `CSoundData` (ses)) temel sınıf görevi görür.
    *   Kaynakların dosya adıyla tanımlanmasını, EterPack üzerinden yüklenmesini, durumlarının takip edilmesini ve referans sayımıyla yaşam döngülerinin yönetilmesini sağlar.
    *   `CResourceManager`, bu `CResource` nesnelerini bir harita içinde tutar ve onlara erişimi sağlar.

### `ResourceManager.h` ve `ResourceManager.cpp` (`CResourceManager` Sınıfı)

*   **Amaç:** `CResourceManager` sınıfı, `CSingleton`'dan miras alarak oyun içindeki tüm `CResource` tabanlı kaynakların merkezi yönetimini sağlar. Kaynakların yüklenmesi, bulunması, türüne göre oluşturulması, önbelleğe alınması (caching), arka planda yüklenmesi ve yaşam döngülerinin yönetilmesi gibi kritik görevleri üstlenir.

*   **`CResourceManager` Sınıfı:**
    *   **Kalıtım:** `public CSingleton<CResourceManager>`
    *   **Temel Veri Yapıları (Typedefs ve Üyeler):**
        *   `TResourcePointerMap`: `DWORD` (dosya CRC'si) ile `CResource*` (kaynak işaretçisi) eşleştiren bir map. `m_pCacheMap` (statik önbellek için) ve `m_pResMap` (aktif kaynaklar için) bu türdedir.
        *   `TResourceNewFunctionPointerMap`: `std::string` (dosya uzantısı) ile `CResource* (*)(const char*)` (kaynak oluşturma fonksiyonu işaretçisi) eşleştiren bir map (`m_pResNewFuncMap`).
        *   `TResourceNewFunctionByTypePointerMap`: `int` (kaynak türü) ile kaynak oluşturma fonksiyonu işaretçisi eşleştiren bir map (`m_pResNewFuncByTypeMap`).
        *   `TResourceDeletingMap`: `CResource*` ile `DWORD` (genellikle zaman damgası) eşleştiren bir map (`m_ResourceDeletingMap`). Silinmesi beklenen kaynakları tutar.
        *   `TResourceRequestMap`: `DWORD` (CRC) ile `std::string` (dosya adı) eşleştiren bir map. `m_RequestMap` (arka plan yükleme istekleri) ve `m_WaitingMap` (yüklenmesi beklenenler) bu türdedir.
        *   `TResourceRefDecreaseWaitingMap`: `long` (zaman damgası) ile `CResource*` eşleştiren bir map (`m_pResRefDecreaseWaitingMap`). Arka planda yüklenen ve referans sayısının bir süre sonra azaltılması gereken kaynakları tutar.
        *   `ms_loadingThread` (static `CFileLoaderThread`): Arka planda dosya yükleme işlemlerini yürüten thread.
        *   `ms_mutex` (static `std::mutex`, `ENABLE_LOADING_PERFORMANCE` ile aktif): Thread güvenliği için kullanılan mutex.
    *   **Ana Metotlar ve İşlevler:**
        *   `CResourceManager()`: Kurucu.
        *   `~CResourceManager()`: Yok edici.
        *   `LoadStaticCache(const char* c_szFileName)`: Belirtilen dosyayı `GetResourcePointer` ile yükler, CRC'sini hesaplar ve `m_pCacheMap`'e ekler. Kaynağın referans sayısını artırır. Bu, oyun başlangıcında sık kullanılacak bazı kaynakları önceden yüklemek için kullanılabilir.
        *   `DestroyDeletingList()`: `CResource::SetDeleteImmediately(true)` yapar ve `__DestroyCacheMap()` ile `__DestroyDeletingResourceMap()`'i çağırarak önbelleği ve silinmeyi bekleyen kaynak listesini temizler.
        *   `Destroy()`: Tüm kaynak haritasını (`__DestroyResourceMap()`) temizler. Çağrılmadan önce `DestroyDeletingList()`'in çağrılmış olması beklenir (`assert` ile kontrol edilir).
        *   `BeginThreadLoading()`, `EndThreadLoading()`: Arka plan yükleme thread'ini (`ms_loadingThread`) başlatır ve sonlandırır.
        *   `InsertResourcePointer(DWORD dwFileCRC, CResource* pResource)`: Verilen CRC ve kaynak işaretçisini `m_pResMap`'e ekler. Eğer CRC zaten varsa hata verir.
        *   `FindResourcePointer(DWORD dwFileCRC)`: `m_pResMap` içinde verilen CRC'ye sahip bir kaynak arar. Bulursa işaretçisini, bulamazsa `NULL` döndürür.
        *   `GetResourcePointer(const char* c_szFileName)`: Bir kaynağı dosya adına göre alır. `__GetFileCRC` ile dosya adının CRC'sini hesaplar.
            *   Önce `FindResourcePointer` ile `m_pResMap`'te arar.
            *   Bulamazsa, dosya uzantısını alır ve `m_pResNewFuncMap`'ten uygun kaynak oluşturma fonksiyonunu bulur.
            *   Bulunan fonksiyonla yeni bir kaynak nesnesi oluşturur, `m_pResMap`'e ekler ve döndürür.
            *   Eğer uzantıya göre fonksiyon bulunamazsa hata verir.
        *   `GetTypeResourcePointer(const char* c_szFileName, int iType = -1)`: `GetResourcePointer`'a benzer, ancak dosya uzantısı yerine doğrudan bir tür (`iType`) ve `m_pResNewFuncByTypeMap` kullanarak kaynak oluşturur.
        *   `isResourcePointerData(DWORD dwFileCRC)`: `m_pResMap`'te verilen CRC'ye sahip bir kaynağın olup olmadığını kontrol eder (işaretçi döndürür veya `NULL`).
        *   `RegisterResourceNewFunctionPointer(const char* c_szFileExt, CResource* (*pResNewFunc)(const char* c_szFileName))`: Belirli bir dosya uzantısı için kaynak oluşturma fonksiyonunu kaydeder.
        *   `RegisterResourceNewFunctionByTypePointer(int iType, CResource* (*pNewFunc) (const char* c_szFileName))`: Belirli bir kaynak türü için kaynak oluşturma fonksiyonunu kaydeder.
        *   `DumpFileListToTextFile(const char* c_szFileName)`: `m_pResMap` içindeki tüm kaynakların dosya adlarını, boyutlarını (KB) ve yükleme sürelerini (cost) bir metin dosyasına yazar. Farklı sıralama kriterlerine göre (boyut, maliyet) listeler oluşturur.
        *   `IsFileExist(const char* c_szFileName)`: `CEterPackManager::Instance().IsFileExist()` kullanarak bir dosyanın EterPack içinde var olup olmadığını kontrol eder.
        *   `Update()`: Her frame'de çağrılır. Silinmeyi bekleyen kaynakları (`m_ResourceDeletingMap`) kontrol eder. Belirli bir süre (`c_Deleting_Wait_Time`) geçmişse ve referans sayısı 0 ise kaynağı `Clear()` ile temizler ve haritadan çıkarır.
        *   `ReserveDeletingResource(CResource* pResource)`: Bir `CResource` nesnesi `Release()` edildiğinde ve referans sayısı 0'a düştüğünde (ve `CResource::ms_bDeleteImmediately` `false` ise) bu fonksiyon çağrılır. Kaynağı `m_ResourceDeletingMap`'e ekler ve silinme zamanını kaydeder.
        *   `ProcessBackgroundLoading()`: Arka plan yükleme işlemlerini yönetir.
            *   `m_RequestMap`'teki yükleme isteklerini `ms_loadingThread`'e gönderir ve `m_WaitingMap`'e taşır.
            *   `ms_loadingThread`'den tamamlanmış yüklemeleri (`Fetch`) alır.
            *   Yüklenen dosya için `GetResourcePointer` ile kaynak nesnesini alır (veya oluşturur).
            *   Eğer kaynak boşsa (`IsEmpty()`), `OnLoad()` ile veriyi yükler, referans sayısını artırır (`AddReferenceOnly()`) ve kaynağı belirli bir süre sonra referansını azaltmak üzere `m_pResRefDecreaseWaitingMap`'e ekler.
            *   Tamamlanan isteği `m_WaitingMap`'ten çıkarır.
            *   `m_pResRefDecreaseWaitingMap`'i kontrol ederek bekleme süresi dolan kaynakların referanslarını `Release()` ile azaltır.
        *   `PushBackgroundLoadingSet(std::set<std::string>& LoadingSet)`: Verilen dosya adları setini arka plan yükleme için `m_RequestMap`'e ekler.
    *   **Yardımcı Metotlar:**
        *   `__DestroyDeletingResourceMap()`, `__DestroyResourceMap()`, `__DestroyCacheMap()`: İlgili map'leri temizleyen özel fonksiyonlar.
        *   `__GetFileCRC(const char* c_szFileName, const char** c_pszLowerFile = NULL)`: Dosya adının CRC32'sini hesaplar. Dosya yolundaki '/' karakterlerini '\\' ile değiştirir ve ismi küçük harfe çevirir. İsteğe bağlı olarak küçük harfe çevrilmiş dosya adını `c_pszLowerFile` ile dışarı verir.

*   **Global Değişken:**
    *   `g_iLoadingDelayTime`: Yükleme işlemleriyle ilgili bir gecikme süresi (muhtemelen frame başına yükleme sayısını sınırlamak için).

*   **Kullanım Alanı:**
    *   `CResourceManager`, tüm oyun kaynaklarının merkezi bir noktadan yönetilmesini sağlar.
    *   Oyun başladığında veya belirli bir seviye yüklendiğinde gerekli kaynakların yüklenmesinden, oyun sırasında ihtiyaç duyulan kaynakların anında veya arka planda getirilmesinden ve artık kullanılmayan kaynakların güvenli bir şekilde bellekten temizlenmesinden sorumludur.
    *   Farklı dosya türleri için özel kaynak oluşturma fonksiyonlarının kaydedilmesine izin vererek genişletilebilir bir yapı sunar.
    *   Arka planda yükleme (threading) özelliği, oyunun akıcılığını artırmaya yardımcı olurken, önbellekleme (caching) sık erişilen kaynaklara hızlı erişim sağlar.

### `GrpBase.h` ve `GrpBase.cpp` (`CGraphicBase` Sınıfı ve İlgili Tanımlar)

*   **Amaç:** `GrpBase.h` dosyası, `EterLib` içindeki grafiksel işlemler için temel tanımları, sık kullanılan vertex yapılarını, yardımcı fonksiyonları ve `CGraphicBase` sınıfını içerir. `CGraphicBase`, Direct3D cihazının ve temel grafiksel durumların (kamera, matrisler, viewport) yönetimi için statik metotlar ve üyeler sunan bir arayüz görevi görür. Doğrudan örneği oluşturulmaz, statik arayüzü kullanılır.

*   **Temel Tanımlar ve Yapılar (`GrpBase.h`):**
    *   **Vertex Yapıları:** Çeşitli vertex formatlarını tanımlayan çok sayıda yapı bulunur. Örneğin:
        *   `TVertex`: Pozisyon (x,y,z), renk (DWORD), doku koordinatları (u,v).
        *   `SPVertex`: Sadece pozisyon (x,y,z).
        *   `TPDVertex`: Pozisyon, renk.
        *   `TPTVertex`: Pozisyon, doku koordinatı.
        *   `TPDTVertex`: Pozisyon, renk, doku koordinatı.
        *   `TPNTVertex`: Pozisyon, normal, doku koordinatı (En sık kullanılanlardan biri).
        *   `TPNT2Vertex`: Pozisyon, normal, iki set doku koordinatı.
        *   `TPDT2Vertex`: Pozisyon, renk, iki set doku koordinatı.
    *   **Diğer Yapılar:**
        *   `SFace`: 3 adet `TIndex` (WORD) içeren bir üçgen yüz yapısı.
        *   `TTextureCoordinate`: `D3DXVECTOR2`.
        *   `TNormal`, `TPosition`: `D3DXVECTOR3`.
        *   `TDiffuse`, `TAmbient`, `TSpecular`: `DWORD`.
        *   `UDepth (TDepth)`: `float`, `long`, `DWORD` olarak erişilebilen bir union (derinlik için).
        *   `SNameInfo`: `DWORD name` ve `TDepth depth` (isim ve derinlik bilgisi, muhtemelen picking için).
        *   `SBoundBox`: Min/max koordinatları (`sx,sy,sz`, `ex,ey,ez`), mesh ve kemik indeksi içeren sınırlayıcı kutu.
    *   **Global Fonksiyonlar:**
        *   `PixelPositionToD3DXVECTOR3`, `D3DXVECTOR3ToPixelPosition`: Koordinat sistemi dönüşümleri (Y eksenini ters çevirir).
    *   `c_FillRectIndices`: Bir dörtgeni iki üçgenle çizmek için sabit indeks dizisi.

*   **`CGraphicBase` Sınıfı:**
    *   **Statik Üyeler (Önemlileri):**
        *   `ms_lpd3d`, `ms_lpd3dDevice`: Direct3D8 arayüzü ve Direct3D8 cihazı işaretçileri.
        *   `ms_lpd3dMatStack`: Direct3D matris yığını (`ID3DXMatrixStack`).
        *   `ms_d3dPresentParameter`: Cihaz oluşturulurken kullanılan sunum parametreleri.
        *   `ms_Viewport`: Mevcut Direct3D viewport'u.
        *   `ms_d3dCaps`: Direct3D cihazının yetenekleri (`D3DCAPS8`).
        *   `ms_dwD3DBehavior`: Cihaz oluşturulurken kullanılan davranış bayrakları (örn: `D3DCREATE_HARDWARE_VERTEXPROCESSING`).
        *   `ms_matIdentity`, `ms_matView`, `ms_matProj`, `ms_matInverseView`, `ms_matWorld`: Temel dönüşüm matrisleri.
        *   `ms_iWidth`, `ms_iHeight`: Ekran çözünürlüğü.
        *   `ms_fFieldOfView`, `ms_fNearY`, `ms_fFarY`, `ms_fAspect`: Projeksiyon matrisi parametreleri.
        *   `ms_lpSphereMesh`, `ms_lpCylinderMesh`: Basit geometrik şekiller için D3DX meshleri.
        *   `ms_alpd3dPDTVB[]`: `TPDTVertex` formatı için bir dizi dinamik vertex buffer (ring buffer gibi kullanılır).
        *   `ms_alpd3dDefIB[]`: Önceden tanımlanmış bazı basit geometriler (çizgi, küp vb.) için indeks buffer'ları.
    *   **Kurucu ve Yok Edici:**
        *   `CGraphicBase()`: Statik üyelerin bazılarını başlatır (örneğin, matrisleri identity yapar).
        *   `virtual ~CGraphicBase()`: Yok edici.
    *   **Ana Statik Metotlar ve İşlevler:**
        *   `GetAvailableTextureMemory()`: Kullanılabilir doku belleğini döndürür (periyodik olarak güncellenir).
        *   `GetViewMatrix()`, `GetIdentityMatrix()`: İlgili statik matrisleri döndürür.
        *   **Kamera Ayar Fonksiyonları:**
            *   `SetSimpleCamera`, `SetEyeCamera`, `SetAroundCamera`, `SetPositionCamera`: Farklı parametrelerle view matrisini (`ms_matView`) ayarlar. `CCamera` sınıfını kullanır.
            *   `MoveCamera`: Kamera pozisyonunu öteler.
            *   `GetTargetPosition`, `GetCameraPosition`: Kamera hedef ve pozisyon bilgilerini alır.
        *   **Projeksiyon Ayar Fonksiyonları:**
            *   `SetOrtho2D`, `SetOrtho3D`, `SetPerspective`: Projeksiyon matrisini (`ms_matProj`) ve ilgili parametreleri (`ms_fFieldOfView` vb.) ayarlar.
        *   **Matris Yığını İşlemleri (World Transform):**
            *   `PushMatrix()`, `PopMatrix()`: `ms_lpd3dMatStack` kullanarak dünya matrisini yığına ekler/çıkarır.
            *   `MultMatrix()`, `MultMatrixLocal()`: Mevcut dünya matrisini verilen matrisle çarpar.
            *   `Translate()`, `Rotate()`, `RotateLocal()`, `RotateYawPitchRollLocal()`, `Scale()`: Dünya matrisine temel dönüşümler uygular.
            *   `LoadMatrix()`, `GetMatrix()`, `GetMatrixPointer()`: Dünya matrisini ayarlar veya alır.
        *   `GetSphereMatrix()`: Küresel yansıma (sphere mapping) için özel bir matris oluşturur.
        *   **Ekran Efektleri:**
            *   `InitScreenEffect()`: Efekt parametrelerini sıfırlar.
            *   `SetScreenEffectWaving()`: Ekranda dalgalanma efekti parametrelerini ayarlar.
            *   `SetScreenEffectFlashing()`: Ekranda renk flaşı efekti parametrelerini ayarlar.
        *   `GetColor()`: Verilen r,g,b,a (float) değerlerinden `DWORD` renk üretir.
        *   `GetFaceCount()`, `ResetFaceCount()`: Çizilen üçgen sayısını yönetir (muhtemelen debug/profiling için).
        *   `GetLastResult()`: Son HRESULT değerini döndürür.
        *   `UpdateProjMatrix()`, `UpdateViewMatrix()`: Direct3D cihazına `ms_matProj` ve `ms_matView` matrislerini set eder.
        *   `SetViewport()`: Direct3D viewport'unu ayarlar.
        *   `GetBackBufferSize()`: Arka tampon boyutlarını alır.
        *   `IsTLVertexClipping()`, `IsFastTNL()`, `IsLowTextureMemory()`, `IsHighTextureMemory()`: Cihaz yetenekleri ve durumu hakkında bilgi döndürür.
        *   `SetDefaultIndexBuffer(UINT eDefIB)`: Önceden tanımlanmış indeks buffer'larından birini aktif hale getirir.
        *   `SetPDTStream(SPDTVertexRaw* pVertices, UINT uVtxCount)` ve `SetPDTStream(SPDTVertex* pVertices, UINT uVtxCount)`: Verilen `TPDTVertex` verilerini `ms_alpd3dPDTVB` dinamik vertex buffer'larından birine yükler ve stream kaynağı olarak ayarlar. Basit, sık değişen geometrilerin çizimi için kullanılır.
    *   **`GrpBase.cpp` İçindeki Uygulama Detayları:**
        *   `CGraphicBase` sınıfının statik üyelerinin ilk değer atamaları burada yapılır.
        *   Kamera fonksiyonları genellikle `CCamera` adlı başka bir sınıfın (bu dosyada tanımı yok, muhtemelen ayrı) statik metotlarını çağırır veya onun durumunu günceller.
        *   Matris işlemleri `D3DXMatrix` fonksiyonlarını kullanır.
        *   `SetPDTStream`, veriyi bir sonraki uygun dinamik vertex buffer'a (`D3DLOCK_DISCARD` ile) kopyalar ve `STATEMANAGER.SetStreamSource` ile ayarlar.
        *   Direct3D cihazının oluşturulması ve yok edilmesiyle ilgili fonksiyonlar (örneğin, `Create`, `Destroy`) bu dosyalarda doğrudan görünmüyor; muhtemelen `CEterSystem` veya `CWindowApplication` gibi daha üst seviye bir sınıfta ele alınıyor ve `CGraphicBase`'in statik üyeleri orada dolduruluyor.

*   **Kullanım Alanı:**
    *   `CGraphicBase` ve içerdiği tanımlar, `EterLib` ve onu kullanan diğer kütüphaneler için temel 2D/3D grafik işlemlerinin altyapısını oluşturur.
    *   Grafiksel nesnelerin render edilmesi için gerekli olan view, projection ve world matrislerinin yönetimi, temel vertex formatları, viewport ayarları ve basit çizim işlemleri için bir merkez görevi görür.
    *   Doğrudan örneklenmek yerine, statik fonksiyonları ve üyeleri aracılığıyla tüm grafiksel kod tarafından erişilir ve kullanılır.

### `Pool.h` (Nesne Havuzlama Sınıfları)

*   **Amaç:** `Pool.h` dosyası, sık sık oluşturulup yok edilen nesneler için bellek ayırma ve serbest bırakma maliyetini azaltmak amacıyla kullanılan nesne havuzlama (object pooling) mekanizmalarını tanımlar. Bu, performansı artırabilir ve bellek parçalanmasını (memory fragmentation) azaltabilir.

*   **`CDynamicPool<T>` Template Sınıfı:**
    *   **Amaç:** `T` tipindeki nesneler için basit bir dinamik havuz yönetimi sağlar.
    *   **Temel Değişkenler:**
        *   `m_kVct_pkData`: Havuz tarafından oluşturulmuş tüm `T` nesnelerinin işaretçilerini tutan bir `std::vector`.
        *   `m_kVct_pkFree`: Şu anda kullanımda olmayan (serbest) `T` nesnelerinin işaretçilerini tutan bir `std::vector`.
        *   `m_uInitCapacity`: Başlangıçta rezerve edilecek kapasite (ancak tam olarak kullanılmıyor gibi görünüyor, `reserve` çağrılıyor ama önceden doldurulmuyor).
        *   `m_uUsedCapacity`: Havuzun oluşturduğu toplam nesne sayısı.
    *   **Temel Metotlar:**
        *   `Create(UINT uCapacity)`: Belirtilen kapasite için vektörlerde yer ayırır (`reserve`).
        *   `Alloc()`: Havuzdan bir `T` nesnesi talep eder.
            *   Eğer `m_kVct_pkFree` boşsa, `new T` ile yeni bir nesne oluşturur, `m_kVct_pkData`'ya ekler ve döndürür.
            *   Eğer `m_kVct_pkFree`'de serbest nesne varsa, oradan birini alır, `m_kVct_pkFree`'den çıkarır ve döndürür.
        *   `Free(T* pkData)`: Kullanılan bir `T` nesnesini havuza geri bırakır (`m_kVct_pkFree`'ye ekler).
        *   `FreeAll()`: `m_kVct_pkData`'daki tüm nesneleri `m_kVct_pkFree`'ye taşıyarak hepsini serbest olarak işaretler.
        *   `Clear()` / `Destroy()`: Havuzdaki tüm nesneleri `delete` ile siler ve vektörleri temizler.
        *   `GetCapacity()`: Havuzun o ana kadar oluşturduğu toplam nesne sayısını (`m_kVct_pkData.size()`) döndürür.
    *   **Not:** `DYNAMIC_POOL_STRICT` makrosu tanımlıysa, `Free` içinde bazı `assert` kontrolleri (verinin geçerli olup olmadığı, zaten serbest olup olmadığı) aktifleşir.

*   **`CDynamicPoolEx<T>` Template Sınıfı:**
    *   **Amaç:** `CDynamicPool<T>`'ye çok benzer, ancak nesneleri `new T` ve `delete pkData` yerine `::operator new(sizeof(T))` ve `::operator delete(pkData)` kullanarak oluşturur ve siler. Bu, nesnelerin kurucu (constructor) ve yok edici (destructor) metotlarının çağrılmasını atlayarak daha ham bir bellek yönetimi sağlar. Bu genellikle sadece POD (Plain Old Data) türleri için veya nesne yaşam döngüsünün manuel olarak yönetildiği özel durumlar için uygundur.
    *   Yapısı ve metotları `CDynamicPool<T>` ile büyük ölçüde aynıdır, fark sadece nesne oluşturma/silme yöntemindedir (`New()` ve `Delete()` statik yardımcı metotları aracılığıyla).

*   **`CPooledObject<T>` Template Sınıfı:**
    *   **Amaç:** Belirli bir `T` tipi için global (statik) bir `CDynamicPoolEx<T>` havuzu kullanarak nesnelerin oluşturulmasını ve silinmesini yöneten bir temel sınıftır. Bu sınıftan türeyen nesneler, `new` ve `delete` operatörlerini override ederek otomatik olarak bu global havuzu kullanır.
    *   **Statik Üyeler:**
        *   `ms_kPool` (static `CDynamicPoolEx<T>`): `T` tipi için paylaşılan nesne havuzu.
    *   **Override Edilmiş Operatörler:**
        *   `void* operator new(unsigned int uiSize)`: `ms_kPool.Alloc()` çağırarak havuzdan bellek alır.
        *   `void operator delete(void* pT)`: `ms_kPool.Free((T*)pT)` çağırarak nesneyi havuza geri bırakır.
    *   **Statik Metotlar:**
        *   `DestroySystem()`: `ms_kPool.Destroy()` çağırarak global havuzu temizler.
        *   `DeleteAll()`: `ms_kPool.FreeAll()` çağırarak havuzdaki tüm nesneleri serbest olarak işaretler.
        *   `GetStaticPool()`: Statik havuza (`ms_kPool`) referans döndürür.

*   **Kullanım Alanı:**
    *   `CDynamicPool` ve `CDynamicPoolEx`: Sıkça oluşturulan ve yok edilen nesneler (örneğin, partiküller, kısa süreli efekt nesneleri, mermi gibi oyun içi varlıklar) için bellek yönetimini optimize etmek amacıyla kullanılır. `new` ve `delete` çağrılarının sıklığını azaltarak performansı artırır.
    *   `CDynamicPoolEx`: Kurucu/yok edici çağrılarının maliyetinden kaçınmak istendiğinde veya nesnelerin bellekteki yerleşimi üzerinde daha hassas kontrol gerektiğinde kullanılır.
    *   `CPooledObject`: Bir sınıfın tüm örneklerinin otomatik olarak bir nesne havuzundan yönetilmesini sağlamak için kolay bir yol sunar. Sınıfı `CPooledObject<KendiSinifi>` şeklinde bu sınıftan türetmek yeterlidir.

### `StateManager.h` ve `StateManager.cpp` (`CStateManager` Sınıfı)

*   **Amaç:** `CStateManager` sınıfı, `CSingleton`'dan türeyerek Direct3D 8 render state'lerinin, doku state'lerinin, shader'ların, transform matrislerinin, sabitlerin (constants), vertex/index buffer atamalarının, materyal ve ışıkların yönetimini merkezileştiren ve optimize eden bir sistem sunar. Temel amacı, Direct3D API çağrılarını önbelleğe alarak (caching) gereksiz state değişikliklerini önlemek ve böylece render performansını artırmaktır. (Kodun başındaki yorumlar, bunun NVIDIA tarafından geliştirilmiş olabileceğini gösteriyor.)

*   **Yardımcı Sınıflar ve Enum'lar (`StateManager.h`):**
    *   `CStreamData`: Bir vertex buffer (`LPDIRECT3DVERTEXBUFFER8`) ve onun stride'ını (bir vertex'in boyutu) bir arada tutar.
    *   `CIndexData`: Bir index buffer (`LPDIRECT3DINDEXBUFFER8`) ve başlangıç vertex indeksini (`BaseVertexIndex`) bir arada tutar.
    *   `eStateType`: Yönetilen state türlerini tanımlar (`STATE_MATERIAL`, `STATE_RENDER`, `STATE_TEXTURE`, `STATE_TEXTURESTAGE`, `STATE_VSHADER`, `STATE_PSHADER`, `STATE_TRANSFORM`, `STATE_VCONSTANT`, `STATE_PCONSTANT`, `STATE_STREAM`, `STATE_INDEX`).
    *   `CStateID`: Bir state değişikliğini temsil eder. `eStateType` ve ilgili parametreleri (örneğin, Render State türü, Texture Stage indeksi ve türü) içerir.
    *   `CStateManagerState`: Belirli bir andaki tüm yönetilen state'lerin (Render State'ler, Texture State'ler, atanan dokular, shader'lar, matrisler, sabitler, stream/index buffer'ları, materyal) bir anlık görüntüsünü (snapshot) tutar. `ResetState` metodu ile varsayılan değerlere sıfırlanabilir.

*   **`CStateManager` Sınıfı:**
    *   **Kalıtım:** `public CSingleton<CStateManager>`
    *   **Temel Değişkenler:**
        *   `m_lpD3DDev`: Yönetilen Direct3D cihazı işaretçisi.
        *   `m_CurrentState`, `m_CopyState`, `m_ChipState`: `CStateManagerState` türünden üç önemli state nesnesi:
            *   `m_CurrentState`: Kullanıcının API aracılığıyla ayarladığı hedef state'leri tutar.
            *   `m_ChipState`: Direct3D cihazındaki (donanımdaki) mevcut state'leri temsil eder (veya temsil ettiği varsayılır).
            *   `m_CopyState`: `Save...` metotlarıyla state'leri geçici olarak kaydetmek için kullanılır.
        *   `m_StateCache`: Gerçekleşmesi gereken (yani `m_CurrentState` ile `m_ChipState` arasında farklı olan) state değişikliklerini (`CStateID`) tutan bir `std::vector`. `Set...` metotları burayı doldurur.
        *   `m_bForce`: `true` ise, state karşılaştırması yapmadan tüm state'lerin cihaza gönderilmesini zorlar.
        *   `m_bScene`: `BeginScene` ve `EndScene` arasında `true` olur.
        *   `m_dwBestMinFilter`, `m_dwBestMagFilter`: Cihazın yeteneklerine göre belirlenen en iyi doku filtreleme modları (genellikle Anisotropic veya Linear).
        *   `m_kLightData` (static `SLightData`, cpp içinde): Işık verilerini tutar.
    *   **Kurulum ve Yönetim:**
        *   `CStateManager(LPDIRECT3DDEVICE8 lpDevice)`: Kurucu. Cihazı alır ve `SetDevice` çağırır.
        *   `~CStateManager()`: Yok edici. Cihaz referansını bırakır.
        *   `SetDevice(LPDIRECT3DDEVICE8 lpDevice)`: Yeni bir Direct3D cihazı ayarlar, cihaz yeteneklerini (`D3DCAPS8`) alır, en iyi filtreleme modlarını belirler ve `SetDefaultState` çağırır.
        *   `SetDefaultState()`: Tüm state nesnelerini (`m_CurrentState`, `m_CopyState`, `m_ChipState`) varsayılan değerlere sıfırlar. Temel transform matrislerini identity yapar, varsayılan materyal ve render/texture stage state'lerini ayarlar.
        *   `Restore()`: Tüm `m_CurrentState`'deki state'leri `m_bForce = true` yaparak cihaza tekrar gönderir.
        *   `BeginScene()`, `EndScene()`: Direct3D `BeginScene`/`EndScene` çağrılarını yapar ve `m_bScene` bayrağını yönetir.
    *   **State Ayarlama Metotları (`Set...`)**: Bu metotlar (`SetRenderState`, `SetTexture`, `SetTextureStageState`, `SetVertexShader`, `SetPixelShader`, `SetTransform`, `SetVertexShaderConstant`, `SetPixelShaderConstant`, `SetStreamSource`, `SetIndices`, `SetMaterial`, `SetLight`) state değişikliği taleplerini alır.
        1.  İstenen state'i `m_CurrentState` içinde günceller.
        2.  İstenen state ile `m_ChipState` (donanımdaki mevcut state) karşılaştırılır.
        3.  Eğer state farklıysa veya `m_bForce` `true` ise:
            *   İlgili Direct3D API çağrısı yapılır (örn: `m_lpD3DDev->SetRenderState`).
            *   `m_ChipState` güncellenir.
            *   **(Not:** Kodun orijinal yorumlarında belirtilen `m_StateCache` kullanımı bu implementasyonda tam olarak görünmüyor. State değişiklikleri doğrudan cihaza uygulanıyor ve sadece `m_ChipState` ile karşılaştırarak gereksiz çağrılar engelleniyor. Belki de `m_StateCache` daha önceki bir versiyonda kullanılıyordu veya farklı bir amaç için var.)
    *   **State Kaydetme/Geri Yükleme Metotları (`Save...`, `Restore...`)**: Belirli bir state'in mevcut değerini (`m_CurrentState`) `m_CopyState` içine kaydeder (`Save...`) ve daha sonra `m_CopyState`'teki değeri geri yükler (`Restore...`, bu da ilgili `Set...` metodunu çağırır).
    *   **State Alma Metotları (`Get...`)**: `m_CurrentState` içindeki mevcut state değerlerini döndürür (`GetRenderState`, `GetTexture`, `GetTextureStageState`, `GetVertexShader`, `GetPixelShader`, `GetTransform`, `GetMaterial`, `GetLight`).
    *   **Çizim (Draw) Metotları:** `DrawPrimitive`, `DrawPrimitiveUP`, `DrawIndexedPrimitive`, `DrawIndexedPrimitiveUP` gibi Direct3D çizim fonksiyonlarını sarmalar. Bunları çağırmadan önce herhangi bir bekleyen state değişikliği olup olmadığını kontrol etmezler; state'lerin önceden ayarlanmış olduğu varsayılır.
    *   `SetBestFiltering(DWORD dwStage)`: Belirlenen doku katmanı için cihazın desteklediği en iyi filtreleme ayarlarını (Anisotropic veya Linear) yapar.

*   **Kullanım Alanı:**
    *   `CStateManager`, tüm Direct3D state ayarlamaları için merkezi bir nokta sağlar.
    *   Kodun farklı yerlerinden yapılan `SetRenderState`, `SetTexture` gibi çağrıların doğrudan Direct3D API'sini çağırmak yerine `CStateManager` üzerinden yapılması beklenir (genellikle `STATEMANAGER.` makrosu veya benzeri bir singleton erişimi ile).
    *   State'leri önbelleğe alarak (mevcut state ile karşılaştırarak), aynı state'in tekrar tekrar ayarlanmasını önler ve bu sayede CPU ve GPU arasındaki iletişim yükünü azaltarak performansı artırır.
    *   `Save/Restore` mekanizması, geçici state değişiklikleri yapıp sonra önceki duruma kolayca dönmeyi sağlar.
    *   Oyunun render döngüsünde kritik bir rol oynar. 

### `NetAddress.h` ve `NetAddress.cpp` (`CNetworkAddress` Sınıfı)

*   **Amaç:** `CNetworkAddress` sınıfı, Winsock'un `SOCKADDR_IN` yapısını sarmalayarak (wrap ederek) IPv4 ağ adreslerini (IP adresi ve port numarası) yönetmek için bir arayüz sağlar. IP adreslerinin string, DWORD veya DNS adı üzerinden ayarlanmasını ve alınmasını kolaylaştırır.

*   **`CNetworkAddress` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_sockAddrIn`: Asıl adres bilgilerini tutan `SOCKADDR_IN` yapısı.
    *   **Statik Metot:**
        *   `static bool GetHostName(char* szName, int size)`: Makinenin ana bilgisayar adını (hostname) alır.
    *   **Kurucu ve Yok Edici:**
        *   `CNetworkAddress()`: Kurucu. `Clear()` çağırır.
        *   `~CNetworkAddress()`: Yok edici.
    *   **Ana Metotlar ve İşlevler:**
        *   `Clear()`: `m_sockAddrIn` yapısını sıfırlar ve `sin_family`'yi `AF_INET` olarak ayarlar.
        *   `Set(const char* c_szAddr, int port)`: Bir adres string'i (IP veya DNS adı) ve port numarası alarak adresi ayarlar.
            *   `IsIP()` ile string'in IP formatında olup olmadığını kontrol eder.
            *   Eğer IP ise `SetIP(const char*)` çağırır.
            *   Eğer IP değilse, DNS adı olduğunu varsayar ve `SetDNS()` çağırır. Başarısız olursa `false` döner.
            *   `SetPort()` ile port numarasını ayarlar.
        *   `SetLocalIP()`: IP adresini `INADDR_ANY` (genellikle 0.0.0.0, yerel makinedeki herhangi bir IP adresini temsil eder) olarak ayarlar.
        *   `SetIP(DWORD ip)`: IP adresini `DWORD` (host byte order) olarak alır ve network byte order'a çevirerek (`htonl`) ayarlar.
        *   `SetIP(const char* c_szIP)`: IP adresini string olarak alır ve `inet_addr` ile `DWORD`'a çevirerek ayarlar.
        *   `SetDNS(const char* c_szDNS)`: Verilen DNS adını `gethostbyname` ile çözümleyerek IP adresini bulur ve ayarlar. Başarısız olursa `false` döner.
        *   `SetPort(int port)`: Port numarasını alır ve network byte order'a çevirerek (`htons`) ayarlar.
        *   `GetPort()`: Port numarasını network byte order'dan host byte order'a çevirerek (`ntohs`) döndürür.
        *   `GetSize()`: `SOCKADDR_IN` yapısının boyutunu döndürür (`sizeof(m_sockAddrIn)`).
        *   `GetIP(char* szIP, int len)`: IP adresini "a.b.c.d" formatında bir string olarak `szIP` tamponuna yazar.
        *   `GetIP()`: IP adresini `DWORD` (host byte order) olarak döndürür (`ntohl` kullanarak).
        *   `operator const SOCKADDR_IN& () const`: Sınıf nesnesinin doğrudan `const SOCKADDR_IN&` türüne dönüştürülebilmesini sağlar. Bu, Winsock fonksiyonlarına parametre olarak geçirmeyi kolaylaştırır.
    *   **Yardımcı Metot:**
        *   `IsIP(const char* c_szAddr)`: Verilen string'in bir IP adresi olup olmadığını basitçe ilk karakterin rakam olup olmadığına bakarak kontrol eder (çok sağlam bir kontrol değil).

*   **Kullanım Alanı:**
    *   Ağ bağlantıları kurulurken (örneğin, `connect` veya `bind` fonksiyonları çağrılırken) hedef veya yerel adresleri belirtmek için kullanılır.
    *   IP adreslerini ve port numaralarını farklı formatlar arasında (string, DWORD) dönüştürmek ve DNS çözümlemesi yapmak için kolay bir yol sunar.
    *   Winsock API'leriyle etkileşimi basitleştirir.

### `NetDatagram.h` ve `NetDatagram.cpp` (`CNetworkDatagram` Sınıfı)

*   **Amaç:** `CNetworkDatagram` sınıfı, UDP (User Datagram Protocol) soketlerini kullanarak bağlantısız (connectionless) ağ iletişimi için bir arayüz sağlar. Belirli bir portu dinleyerek datagramları almak ve belirtilen bir adrese datagram göndermek için kullanılır.

*   **`CNetworkDatagram` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_sock`: Oluşturulan UDP soketinin tanıtıcısı (`SOCKET`).
        *   `m_fdsRecv`, `m_fdsSend`: `select` fonksiyonu tarafından kullanılan dosya tanımlayıcı setleri (`fd_set`). Soketin okunabilir veya yazılabilir durumda olup olmadığını kontrol etmek için kullanılır.
    *   **Kurucu ve Yok Edici:**
        *   `CNetworkDatagram()`: Kurucu. `__Initialize()` çağırır.
        *   `~CNetworkDatagram()`: Yok edici. `Destroy()` çağırır.
    *   **Ana Metotlar ve İşlevler:**
        *   `Destroy()`: Eğer soket geçerliyse (`m_sock != INVALID_SOCKET`), `closesocket()` ile kapatır ve `__Initialize()` çağırarak üyeleri sıfırlar.
        *   `Create(UINT uPort)`: Yeni bir UDP soketi oluşturur.
            *   `socket(AF_INET, SOCK_DGRAM, 0)` ile bir UDP soketi oluşturur.
            *   `ioctlsocket(m_sock, FIONBIO, &arg)` ile soketi non-blocking (engellemesiz) moda ayarlar.
            *   Verilen port numarasını (`uPort`) kullanarak bir `SOCKADDR_IN` yapısı oluşturur (IP adresi `INADDR_ANY` olarak ayarlanır).
            *   `bind()` fonksiyonu ile soketi bu adrese (yerel IP ve belirtilen port) bağlar. Başarısız olursa `false` döner.
        *   `Update()`: Soketin durumunu kontrol etmek için `select` fonksiyonunu kullanır.
            *   `FD_ZERO` ile `m_fdsRecv` ve `m_fdsSend` setlerini temizler.
            *   `FD_SET` ile `m_sock`'ı her iki sete de ekler.
            *   `select()` fonksiyonunu sıfır gecikme (`delay.tv_sec = 0`, `delay.tv_usec = 0`) ile çağırarak soketin okunabilir veya yazılabilir durumda olup olmadığını anında kontrol eder. Sonuçlar `m_fdsRecv` ve `m_fdsSend` setlerine yansıtılır.
        *   `CanRecv()`: `Update()` çağrıldıktan sonra, `FD_ISSET(m_sock, &m_fdsRecv)` kullanarak sokette okunacak veri olup olmadığını kontrol eder.
        *   `PeekRecvFrom(UINT uBufLen, void* pvBuf, SOCKADDR_IN* pkSockAddrIn)`: `recvfrom` fonksiyonunu `MSG_PEEK` bayrağı ile çağırır. Bu, soketteki veriyi okur ancak veriyi soket tamponundan kaldırmaz. Gönderenin adresi `pkSockAddrIn`'e yazılır. Alınan byte sayısını döndürür.
        *   `RecvFrom(UINT uBufLen, void* pvBuf, SOCKADDR_IN* pkSockAddrIn)`: `recvfrom` fonksiyonunu çağırarak soketten veri okur ve veriyi soket tamponundan kaldırır. Gönderenin adresi `pkSockAddrIn`'e yazılır. Alınan byte sayısını döndürür.
        *   `SendTo(UINT uBufLen, const void* c_pvBuf, const SOCKADDR_IN& c_rkSockAddrIn)`: `sendto` fonksiyonunu çağırarak verilen veriyi (`c_pvBuf`) belirtilen hedef adrese (`c_rkSockAddrIn`) gönderir. Gönderilen byte sayısını döndürür.
    *   **Yardımcı Metot:**
        *   `__Initialize()`: `m_sock`'ı `INVALID_SOCKET` olarak ayarlar.

*   **Kullanım Alanı:**
    *   Genellikle oyun içinde daha az güvenilir ancak daha hızlı veri aktarımı gerektiren durumlar için kullanılır (örneğin, pozisyon güncellemeleri, bazı oyun olayları).
    *   Belirli bir portu dinleyerek sunucudan veya diğer istemcilerden UDP paketleri almak ve göndermek için temel bir mekanizma sunar.
    *   Non-blocking yapısı ve `select` kullanımı, oyunun ana döngüsünü engellemeden ağ işlemlerini kontrol etmeyi mümkün kılar.

### `NetStream.h` ve `NetStream.cpp` (`CNetworkStream` Sınıfı)

*   **Amaç:** `CNetworkStream` sınıfı, istemcinin sunucu ile olan ana TCP/IP bağlantısını yöneten temel sınıftır. Veri gönderme ve alma işlemlerini, tamponlamayı (buffering), şifrelemeyi (TEA veya Cipher) ve isteğe bağlı paket sekanslamayı (packet sequencing) ele alır. Bağlantı durumlarını takip eder ve olayları (bağlantı başarılı, bağlantı koptu vb.) bildirmek için sanal fonksiyonlar kullanır.

*   **`CNetworkStream` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_sock`: TCP soketinin tanıtıcısı (`SOCKET`).
        *   `m_addr`: Bağlantı kurulan sunucunun adresi (`CNetworkAddress`).
        *   `m_recvBuf`, `m_recvBufSize`, `m_recvBufInputPos`, `m_recvBufOutputPos`: Gelen veriler için ana alım tamponu ve ilgili boyut/pozisyon bilgileri.
        *   `m_sendBuf`, `m_sendBufSize`, `m_sendBufInputPos`, `m_sendBufOutputPos`: Gönderilecek veriler için ana gönderim tamponu ve ilgili boyut/pozisyon bilgileri.
        *   `m_recvTEABuf`, `m_recvTEABufSize`, `m_recvTEABufInputPos`: Eski TEA şifrelemesi kullanılıyorsa, şifresi çözülecek gelen veriler için geçici tampon.
        *   `m_sendTEABuf`, `m_sendTEABufSize`, `m_sendTEABufInputPos`: Eski TEA şifrelemesi kullanılıyorsa, şifrelenecek giden veriler için geçici tampon.
        *   `m_isOnline`: Bağlantının aktif olup olmadığını belirten bayrak.
        *   `m_connectLimitTime`: Bağlantı denemesi için zaman aşımı süresi.
        *   **Şifreleme Üyeleri:**
            *   `m_cipher` (`Cipher`, `__IMPROVED_PACKET_ENCRYPTION__` ile aktif): Geliştirilmiş şifreleme nesnesi.
            *   `m_isSecurityMode` (eski): Güvenlik modunun (TEA) aktif olup olmadığını belirtir.
            *   `m_szEncryptKey`, `m_szDecryptKey` (eski): TEA şifreleme/deşifreleme anahtarları.
        *   **Sekanslama Üyeleri:**
            *   `m_iSequence`: Gönderilen son sekans numarası.
            *   `m_bUseSequence`: Paket sekanslamanın aktif olup olmadığını belirtir.
            *   `m_kVec_bSequenceTable`: Sekans numaraları için kullanılan tablo (muhtemelen bir tür XOR veya basit şifreleme için).
    *   **Kurucu ve Yok Edici:**
        *   `CNetworkStream()`: Kurucu. Tamponları ve diğer üyeleri başlatır. Sekans tablosunu doldurur.
        *   `~CNetworkStream()`: Yok edici. `Clear()` çağırır.
    *   **Kurulum ve Temizleme:**
        *   `SetRecvBufferSize()`, `SetSendBufferSize()`: Alım ve gönderim tamponlarının boyutlarını ayarlar (ve yeniden oluşturur).
        *   `SetSecurityMode(...)`: Şifreleme modunu ve anahtarlarını ayarlar (Eski TEA veya Cipher aktivasyonu için).
        *   `Clear()`: Soketi kapatır, tüm tamponları siler ve üyeleri sıfırlar.
        *   `ClearRecvBuffer()`: Sadece alım tamponunu temizler.
    *   **Bağlantı Yönetimi:**
        *   `Connect(...)`: Verilen adrese (IP, DNS veya `CNetworkAddress` nesnesi) ve porta TCP bağlantısı kurmayı dener.
            *   Yeni bir soket oluşturur.
            *   Non-blocking moda ayarlar.
            *   `connect()` çağırır. Non-blocking modda bu genellikle hemen `WSAEWOULDBLOCK` hatası verir.
            *   Bağlantının başarılı olup olmadığını veya zaman aşımına uğrayıp uğramadığını `Process()` içinde `select()` ile kontrol eder.
            *   Başarı (`OnConnectSuccess`) veya başarısızlık (`OnConnectFailure`) durumunda ilgili sanal fonksiyonları çağırır.
        *   `Disconnect()`: Bağlantıyı kapatır (`closesocket`), `OnDisconnect()` sanal metodunu çağırır ve durumu çevrimdışı yapar.
        *   `IsOnline()`: Bağlantının aktif olup olmadığını (`m_isOnline`) döndürür.
    *   **Veri İşleme:**
        *   `Process()`: Ana işleme döngüsü. Her frame'de çağrılır.
            *   Eğer bağlantı kuruluyorsa (`m_connectLimitTime` ayarlıysa), `select()` ile bağlantı durumunu kontrol eder.
            *   Eğer bağlantı aktifse (`IsOnline()`):
                *   `__SendInternalBuffer()` ile gönderim tamponundaki verileri göndermeye çalışır.
                *   `__RecvInternalBuffer()` ile soketten gelen verileri alım tamponuna okur ve şifreliyse çözer.
                *   Başarısız olursa (bağlantı kopması), `Disconnect()` ve `OnRemoteDisconnect()` çağırır.
                *   Başarılıysa, `OnProcess()` sanal metodunu çağırarak türetilmiş sınıfın gelen verileri işlemesini sağlar.
        *   `__RecvInternalBuffer()`: `recv()` ile soketten veri okur. Okunan veriyi `m_recvBuf`'a yazar. Eğer şifreleme aktifse (`IsSecurityMode()`), veriyi çözer (`m_cipher.Decrypt` veya `old_tea_decrypt`). Başarısız olursa `false` döner.
        *   `__SendInternalBuffer()`: `m_sendBuf`'taki gönderilecek veriyi (`m_sendBufInputPos - m_sendBufOutputPos` kadar) alır. Eğer şifreleme aktifse veriyi şifreler (`m_cipher.Encrypt` veya `old_tea_encrypt`). `send()` ile veriyi sokete gönderir. Gönderilen kısmı tampondan çıkarır (`__PopSendBuffer`). Başarısız olursa `false` döner.
        *   `__PopSendBuffer()`: Gönderim tamponunda gönderilmiş veriyi temizler (gerektiğinde `memmove` ile kaydırır).
    *   **Veri Okuma/Yazma Arayüzü:**
        *   `Peek(int len, ...)`: Alım tamponundan belirtilen boyutta veriyi okur ancak tampondan çıkarmaz.
        *   `Recv(int len, ...)`: Alım tamponundan belirtilen boyutta veriyi okur ve tampondan çıkarır.
        *   `Send(int len, ...)`: Verilen veriyi gönderim tamponuna (`m_sendBuf`) yazar. Veri hemen gönderilmez, `Process()` içindeki `__SendInternalBuffer` tarafından gönderilir.
        *   `SendSequence()`: Eğer `m_bUseSequence` aktifse, bir sekans numarası paketi gönderir.
        *   `SetPacketSequenceMode()`: Paket sekanslamayı açar/kapatır.
    *   **Şifreleme (Cipher) Metotları (`__IMPROVED_PACKET_ENCRYPTION__` ile):**
        *   `Prepare()`: Şifreleme el sıkışması (handshake) için gerekli veriyi hazırlar.
        *   `Activate()`: Karşı taraftan gelen handshake verisiyle şifrelemeyi aktifleştirir.
        *   `ActivateCipher()`: Şifrelemeyi doğrudan aktifleştirir (muhtemelen önceden paylaşılan anahtarla).
    *   **Sanal Metotlar (Olay Bildirimi):**
        *   `OnConnectSuccess()`: Bağlantı başarıyla kurulduğunda çağrılır.
        *   `OnConnectFailure()`: Bağlantı kurma başarısız olduğunda çağrılır.
        *   `OnRemoteDisconnect()`: Karşı taraf bağlantıyı kapattığında çağrılır.
        *   `OnDisconnect()`: Bağlantı herhangi bir nedenle kesildiğinde çağrılır.
        *   `OnProcess()`: `__RecvInternalBuffer` ile veri alındıktan sonra çağrılır. Türetilmiş sınıflar burada alım tamponundaki veriyi (genellikle paketleri) işler.

*   **Kullanım Alanı:**
    *   İstemcinin oyun sunucusuyla olan ana TCP bağlantısını yönetmek için temel altyapıyı sağlar.
    *   Ağ katmanının karmaşıklığını (soket yönetimi, non-blocking I/O, tamponlama, şifreleme) soyutlar.
    *   Genellikle `CPythonNetworkStream` gibi daha üst seviye bir sınıf tarafından miras alınır. Bu üst seviye sınıf, `OnProcess` içinde gelen paketleri ayrıştırır, `Send` ile giden paketleri oluşturur ve `OnConnect*`/`OnDisconnect` olaylarına göre oyun durumunu yönetir.

### `NetPacketHeaderMap.h` ve `NetPacketHeaderMap.cpp` (`CNetworkPacketHeaderMap` Sınıfı)

*   **Amaç:** `CNetworkPacketHeaderMap` sınıfı, ağ üzerinden gelen veya gönderilen paketlerin başlık (header) kimliklerini, bu paketlerin boyut bilgilerine (sabit boyutlu mu, dinamik mi, sabitse boyutu nedir) eşleyen basit bir harita (map) sağlar.

*   **`TPacketType` Yapısı (`NetPacketHeaderMap.h` içinde):**
    *   `iPacketSize`: Sabit boyutlu paketler için paketin boyutunu tutar.
    *   `isDynamicSizePacket`: Paketin boyutunun dinamik olup olmadığını (genellikle paketin kendi içinde boyut bilgisi taşıdığı durumlar) belirten bir boolean bayrak.

*   **`CNetworkPacketHeaderMap` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_headerMap`: `int` (paket başlığı) ile `TPacketType` yapısını eşleştiren bir `std::map`.
    *   **Kurucu ve Yok Edici:**
        *   `CNetworkPacketHeaderMap()`: Kurucu.
        *   `~CNetworkPacketHeaderMap()`: Yok edici.
    *   **Ana Metotlar:**
        *   `Set(int header, TPacketType rPacketType)`: Belirtilen başlık (`header`) için paket türü bilgilerini (`rPacketType`) haritaya ekler veya günceller.
        *   `Get(int header, TPacketType* pPacketType)`: Belirtilen başlığa karşılık gelen `TPacketType` bilgilerini haritadan bulur ve `pPacketType` işaretçisi aracılığıyla döndürür. Başlık bulunamazsa `false` döner.

*   **Kullanım Alanı:**
    *   `CNetworkStream` veya ondan türeyen sınıflar (örneğin `CPythonNetworkStream`), ağdan veri okurken gelen bayt akışını anlamlı paketlere ayırmak için bu haritayı kullanır.
    *   Bir paketin başlığı okunduğunda, bu haritaya başvurularak paketin geri kalanının ne kadar okunması gerektiği (sabit boyutluysa) veya boyut bilgisinin paketin neresinde aranacağı (dinamik boyutluysa) belirlenir.
    *   Bu, paket işleme mantığını merkezileştirir ve farklı paket türlerinin yönetilmesini kolaylaştırır.

Bu belgenin devamı için aşağıdaki bağlantıları takip edebilirsiniz:

*   [EterLib Referans Kılavuzu - Bölüm 2](client_EterLib_Referans_Part2.md)
*   [EterLib Referans Kılavuzu - Bölüm 3](client_EterLib_Referans_Part3.md)
*   [EterLib Referans Kılavuzu - Bölüm 4](client_EterLib_Referans_Part4.md)
*   [EterLib Referans Kılavuzu - Bölüm 5](client_EterLib_Referans_Part5.md)