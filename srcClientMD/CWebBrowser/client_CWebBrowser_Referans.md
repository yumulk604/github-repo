# CWebBrowser Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/CWebBrowser/` klasöründe bulunan `CWebBrowser` modülünün amacını ve temel işlevlerini açıklamaktadır.

`CWebBrowser` modülü, Metin2 istemcisi içine gömülü bir web tarayıcı bileşenini yönetmek için C-stili bir arayüz sağlar. Bu modül, oyun içinde belirli bir alanda web tabanlı içeriklerin (örneğin, duyurular, etkinlik sayfaları, item-shop arayüzleri veya çevrimiçi yardım kılavuzları) gösterilmesine olanak tanır.

---

## `CWebBrowser.h` Dosyası

Bu başlık dosyası, `CWebBrowser` modülünün dışa sunduğu tüm fonksiyonları deklare eder.

*   **Amaç:** Gömülü web tarayıcı bileşeninin yaşam döngüsünü (başlatma, temizleme, yok etme), görünürlüğünü, konumunu ve yükleyeceği web sayfasını kontrol etmek için bir dizi C fonksiyonu tanımlar.
*   **Genel Bakış:** Fonksiyonlar `extern "C"` ile deklare edilmiştir, bu da C ve C++ uyumluluğu sağlar.
*   **Temel Fonksiyonlar:**
    *   **`int WebBrowser_Startup(HINSTANCE hInstance)`**
        *   **İşlevi:** Web tarayıcı bileşenini başlatır ve gerekli kaynakları ayarlar.
        *   **Parametreler:** `hInstance` - Uygulamanın örneği (instance handle).
        *   **Dönüş Değeri:** Başarılı olursa muhtemelen `0` veya pozitif bir değer, hata durumunda negatif bir değer döndürür.
    *   **`void WebBrowser_Cleanup()`**
        *   **İşlevi:** `WebBrowser_Startup` ile ayrılan kaynakların bir kısmını veya belirli geçici verileri temizler. Genellikle program sonlanmadan hemen önce çağrılır.
    *   **`void WebBrowser_Destroy()`**
        *   **İşlevi:** Web tarayıcı bileşenini tamamen yok eder ve tüm kaynaklarını serbest bırakır.
    *   **`int WebBrowser_Show(HWND parent, const char* addr, const RECT* rcWebBrowser)`**
        *   **İşlevi:** Web tarayıcısını görünür hale getirir, belirtilen URL'yi yükler ve belirtilen `parent` pencere içinde `rcWebBrowser` ile tanımlanan dikdörtgen alana konumlandırır.
        *   **Parametreler:**
            *   `parent`: Web tarayıcı penceresinin ebeveyni olacak pencerenin tanıtıcısı (`HWND`).
            *   `addr`: Yüklenecek web sayfasının URL adresi (C-string).
            *   `rcWebBrowser`: Web tarayıcısının ebeveyn pencere içindeki konumunu ve boyutunu belirten `RECT` yapısı işaretçisi.
        *   **Dönüş Değeri:** Başarılı olursa muhtemelen `0` veya pozitif bir değer, hata durumunda negatif bir değer döndürür.
    *   **`void WebBrowser_Hide()`**
        *   **İşlevi:** Web tarayıcısını gizler.
    *   **`void WebBrowser_Move(const RECT* rcWebBrowser)`**
        *   **İşlevi:** Görünür durumdaki web tarayıcısının konumunu ve/veya boyutunu `rcWebBrowser` ile belirtilen yeni değerlere göre günceller.
        *   **Parametreler:** `rcWebBrowser` - Yeni konumu ve boyutu belirten `RECT` yapısı işaretçisi.
    *   **`int WebBrowser_IsVisible()`**
        *   **İşlevi:** Web tarayıcısının o anda görünür olup olmadığını kontrol eder.
        *   **Dönüş Değeri:** Görünürse `1` (veya `true` karşılığı bir değer), gizliyse `0` (veya `false` karşılığı bir değer) döndürebilir.
    *   **`const RECT& WebBrowser_GetRect()`**
        *   **İşlevi:** Web tarayıcısının mevcut konumunu ve boyutunu içeren bir `RECT` yapısına referans döndürür.
*   **Kullanım Alanı:** Oyun arayüzünün bir parçası olarak web tabanlı içerik göstermek, oyunculara güncel bilgiler sunmak veya oyun dışı web kaynaklarına (örneğin forumlar, destek sayfaları) oyun içinden erişim sağlamak amacıyla kullanılır. Arka planda hangi web tarayıcı teknolojisinin (örn: MSHTML/Trident, CEF) kullanıldığı `CWebBrowser.c` implementasyonunda detaylandırılır.

---

## `CWebBrowser.c` Dosyası (Implementasyon Detayları)

Bu C dosyası, `CWebBrowser.h` içinde bildirilen fonksiyonların gerçek implementasyonunu içerir ve gömülü web tarayıcı bileşeninin OLE (Object Linking and Embedding) / COM (Component Object Model) teknolojisi aracılığıyla nasıl yönetildiğini gösterir.

*   **Ana Çalışma Prensibi:** Kod, Windows işletim sisteminin sağladığı COM arayüzlerini kullanarak bir Web Tarayıcı ActiveX kontrolünü (genellikle Internet Explorer'ın `CLSID_WebBrowser` ile temsil edilen MSHTML/Trident motoru) bir Win32 penceresi içine yerleştirir ve yönetir.

*   **OLE Arayüz Implementasyonları:**
    *   Tarayıcı kontrolünün, onu barındıran uygulama (host) ile iletişim kurabilmesi için zorunlu olan temel COM arayüzleri implemente edilir:
        *   `IOleClientSite`: Tarayıcının host hakkında bilgi almasını ve host ile etkileşim kurmasını sağlar.
        *   `IOleInPlaceSite`: Tarayıcının yerinde (in-place) aktivasyonunu yönetir (yani, ayrı bir pencere yerine host penceresi içinde çalışmasını).
        *   `IOleInPlaceFrame`: Tarayıcının, host penceresinin çerçevesiyle (menüler, durum çubuğu vb., bu örnekte çoğu kullanılmıyor) etkileşimini sağlar.
        *   `IDocHostUIHandler`: Tarayıcının kullanıcı arayüzü davranışlarını (sağ tık menüsü, kenarlıklar, script çalıştırma vb.) host uygulamanın özelleştirmesine olanak tanır.
    *   Her arayüz için gerekli C fonksiyonları (`Site_*`, `InPlace_*`, `Frame_*`, `UI_*`) tanımlanır.
    *   Bu fonksiyonların adresleri, statik olarak tanımlanmış VTable (Virtual Method Table) yapılarına (`MyIOleClientSiteTable`, `MyIOleInPlaceSiteTable`, `MyIOleInPlaceFrameTable`, `MyIDocHostUIHandlerTable`) atanır.
    *   **Önemli Fonksiyonlar:**
        *   `Site_QueryInterface`: Tarayıcı bu fonksiyonu çağırarak host uygulamanın implemente ettiği diğer COM arayüzlerini (`IOleInPlaceSite`, `IDocHostUIHandler` vb.) sorgular.
        *   `UI_ShowContextMenu`: Tarayıcının varsayılan sağ tık menüsünü engellemek için `S_OK` döndürür.
        *   `UI_GetHostInfo`: Tarayıcının 3D kenarlığını kaldırmak gibi UI ayarlarını yapar.
        *   `InPlace_GetWindow` ve `Frame_GetWindow`: Tarayıcıya, kendisini barındıran pencerenin `HWND`'ını bildirir.
        *   `InPlace_GetWindowContext`: Tarayıcıya `IOleInPlaceFrame` implementasyonunun adresini ve pencere bilgilerini sağlar.
        *   `InPlace_OnPosRectChange`: Tarayıcı nesnesinin yeniden boyutlandırılması gerektiğinde çağrılır ve `IOleInPlaceObject::SetObjectRects` ile boyutları günceller.
    *   **Genişletilmiş Yapılar (`_IOleClientSiteEx`, `_IOleInPlaceFrameEx`, vb.):** Her tarayıcı örneği için gerekli olan OLE arayüz nesnelerini ve ek bilgileri (özellikle host penceresinin `HWND`'ı) tek bir yapıda toplamak için kullanılır. Bu, COM nesnelerinin durumunu yönetmeyi kolaylaştırır. Bu yapılar `GlobalAlloc` ile dinamik olarak oluşturulur.

*   **Tarayıcı Gömme Süreci (`EmbedBrowserObject` fonksiyonu):**
    1.  COM kütüphanesi `OleInitialize` ile başlatılır.
    2.  Genişletilmiş OLE site yapısı (`_IOleClientSiteEx`) için bellek ayrılır.
    3.  Bu yapı içindeki OLE arayüzlerinin VTable işaretçileri ayarlanır ve host penceresinin `HWND`'ı saklanır.
    4.  `CoGetClassObject` ve `IClassFactory::CreateInstance` kullanılarak `CLSID_WebBrowser` ile tarayıcı OLE nesnesi oluşturulur ve `IOleObject` arayüzü alınır.
    5.  Oluşturulan `IOleObject` işaretçisi ve OLE site yapısının işaretçisi, host penceresinin `GWLP_USERDATA` alanında saklanır. Bu, daha sonra bu pencereye ait tarayıcı nesnesine erişmek için kullanılır.
    6.  `IOleObject::SetClientSite` ile tarayıcıya OLE site implementasyonu bildirilir.
    7.  `OleSetContainedObject` ile tarayıcının gömülü olduğu belirtilir.
    8.  `IOleObject::DoVerb` çağrısı (`OLEIVERB_SHOW` ile) tarayıcıyı etkinleştirir ve host penceresine yerleştirir.
    9.  `IOleObject::QueryInterface` ile `IWebBrowser2` arayüzü alınır. Bu arayüz, sayfa yükleme, navigasyon ve boyutlandırma gibi işlemler için kullanılır.
    10. `IWebBrowser2` arayüzü üzerinden tarayıcının başlangıç boyutları ayarlanır.

*   **API Fonksiyonlarının Çalışması:**
    *   `WebBrowser_Startup`: COM'u başlatır ve tarayıcıyı barındıracak pencere sınıfını (`WEBBROWSER_CLASSNAME`, `WebBrowser_WindowProc` ile) kaydeder.
    *   `WebBrowser_Cleanup`: Pencere sınıfının kaydını siler ve `OleUninitialize` çağırır.
    *   `WebBrowser_Show`: `WEBBROWSER_CLASSNAME` sınıfından bir pencere oluşturur (`CreateWindowEx`). Bu, `WM_CREATE` mesajını tetikler ve `WebBrowser_WindowProc` içindeki `EmbedBrowserObject` çağrılır. Ardından `DisplayHTMLPage` ile verilen URL yüklenir.
    *   `WebBrowser_Hide`: Tarayıcı penceresini (`gs_hWndWebBrowser`) gizler ve `DestroyWindow` ile yok eder.
    *   `WebBrowser_Move`: `MoveWindow` API'sini kullanarak tarayıcı penceresini taşır/yeniden boyutlandırır.
    *   `DisplayHTMLPage` / `DisplayHTMLStr`: `GWLP_USERDATA`'dan alınan `IOleObject` üzerinden `IWebBrowser2` arayüzünü sorgular ve `Navigate2` veya `IHTMLDocument2::write` fonksiyonlarını çağırarak içeriği yükler.
    *   Diğer fonksiyonlar (`WebBrowser_IsVisible`, `WebBrowser_Destroy`, `WebBrowser_GetRect`) genellikle global statik değişken (`gs_hWndWebBrowser`) veya basit API çağrıları üzerinden çalışır.

*   **Teknoloji:** Kod tamamen Win32 API ve COM teknolojilerine dayanır. MFC, ATL veya başka bir C++ framework kullanılmaz. 