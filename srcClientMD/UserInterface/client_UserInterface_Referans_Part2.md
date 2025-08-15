# UserInterface Referans Kılavuzu - Bölüm 2

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 1'de ele alınan konuların ardından, bu bölümde kalan C++ sınıfları, Python modülleri ve diğer ilgili bileşenler incelenecektir.

## İçindekiler

* [`CefClientV8ExtensionHandler.h` ve `CefClientV8ExtensionHandler.cpp`](#cefclientv8extensionhandlerh-ve-cefclientv8extensionhandlercpp)
* [`CefWebBrowser.h` ve `CefWebBrowser.cpp`](#cefwebbrowserh-ve-cefwebbrowsercpp)
* [`CheckLatestFiles.h` ve `CheckLatestFiles.cpp`](#checklatestfilesh-ve-checklatestfilescpp)
* [`CRC32_inc.h`](#crc32_inch)
* [`EventHandler.h`](#eventhandlerh)
* [`GameType.h` ve `GameType.cpp`](#gametypeh-ve-gametypecpp)
* [`GuildMarkDownloader.h` ve `GuildMarkDownloader.cpp`](#guildmarkdownloaderh-ve-guildmarkdownloadercpp)

*(İçerikler eklendikçe güncellenecektir)*

### `CefClientV8ExtensionHandler.h` ve `CefClientV8ExtensionHandler.cpp`

#### Sınıfın Genel Amacı

`ClientV8ExtensionHandler` sınıfı, Chromium Embedded Framework (CEF) içinde çalışan JavaScript kodu ile istemcinin C++ kodu arasında bir iletişim köprüsü kurmak için tasarlanmıştır. `CefV8Handler` arayüzünü implemente ederek, JavaScript'ten çağrılabilen "yerel (native)" fonksiyonlar tanımlanmasını sağlar. Bir JavaScript fonksiyonu C++ tarafına kaydedildiğinde ve JavaScript'ten çağrıldığında, bu sınıfın `Execute` metodu tetiklenir. Bu sayede, web arayüzünden oyunun C++ mantığına veri gönderilebilir veya C++ fonksiyonları tetiklenebilir.

#### Başlık Dosyası Tanımları (`CefClientV8ExtensionHandler.h`)

`CefClientV8ExtensionHandler.h` dosyası, `ClientV8ExtensionHandler` sınıfının yapısını tanımlar.

*   **`struct ClientV8ExtensionHandler : public CefV8Handler`**:
    *   Sınıf, CEF'in V8 JavaScript motoru ile etkileşim için temel olan `CefV8Handler` arayüzünden kalıtım alır.
*   **Yapıcı `ClientV8ExtensionHandler(CefRefPtr<CefApp> app)`**:
    *   Bir `CefApp` referansını parametre olarak alır ve saklar. Bu, genellikle ana CEF uygulama nesnesine erişim için kullanılır.
*   **`bool Execute(...) override`**:
    *   `CefV8Handler` arayüzünden gelen ve override edilen en önemli metottur. JavaScript'ten bir yerel fonksiyon çağrıldığında bu metot çalışır.
    *   Parametreleri:
        *   `const CefString& name`: Çağrılan JavaScript fonksiyonunun adı.
        *   `CefRefPtr<CefV8Value> object`: JavaScript'teki `this` nesnesi.
        *   `const CefV8ValueList& arguments`: JavaScript fonksiyonuna geçirilen argümanların listesi.
        *   `CefRefPtr<CefV8Value>& retval`: C++'tan JavaScript'e bir değer döndürmek için kullanılır.
        *   `CefString& exception`: Bir hata oluşursa JavaScript'e bir istisna mesajı döndürmek için kullanılır.
*   **Özel Üye Değişkenler**:
    *   `CefRefPtr<CefApp> app`: Ana CEF uygulama nesnesine bir referans tutar.
*   **`IMPLEMENT_REFCOUNTING(ClientV8ExtensionHandler)`**:
    *   CEF nesneleri için standart referans sayım mekanizmasını sağlar.

#### C++ Implementasyon Detayları (`CefClientV8ExtensionHandler.cpp`)

`CefClientV8ExtensionHandler.cpp` dosyası, `Execute` metodunun ve yapıcının implementasyonunu içerir.

*   **`ClientV8ExtensionHandler::ClientV8ExtensionHandler(CefRefPtr<CefApp> app)`**:
    *   Yapıcı, kendisine parametre olarak verilen `app` referansını sınıfın üye değişkenine atar.
*   **`bool ClientV8ExtensionHandler::Execute(...)`**:
    *   Bu metot, çağrılan JavaScript fonksiyonunun adına (`name`) göre dallanır.
    *   **`if (name == "ChangeTextInJS")`**:
        *   Eğer çağrılan fonksiyonun adı `"ChangeTextInJS"` ise:
        *   Argüman kontrolü yapılır: Fonksiyonun bir adet string argüman alması beklenir (`arguments.size() == 1 && arguments[0]->IsString()`).
        *   Eğer argümanlar uygunsa:
            *   `const CefString text = arguments[0]->GetStringValue();`: İlk argüman olan string değeri alınır.
            *   `CefRefPtr<CefFrame> frame = CefV8Context::GetCurrentContext()->GetBrowser()->GetMainFrame();`: Mevcut JavaScript bağlamından ana tarayıcı çerçevesi (frame) elde edilir.
            *   `std::string jscall = "ChangeText('"; jscall += text; jscall += "');";`: Yeni bir JavaScript kodu string'i oluşturulur. Bu string, `ChangeText` adında başka bir JavaScript fonksiyonunu, `ChangeTextInJS`'e gelen `text` argümanıyla çağırır.
            *   `frame->ExecuteJavaScript(jscall, frame->GetURL(), 0);`: Oluşturulan bu yeni JavaScript kodu, tarayıcının ana çerçevesinde çalıştırılır.
            *   Metot `true` döndürerek çağrının başarıyla işlendiğini belirtir. Bu özel durumda, C++ fonksiyonu JavaScript'e doğrudan bir değer (`retval`) döndürmez, bunun yerine başka bir JavaScript fonksiyonunu tetikler.
    *   Eğer çağrılan fonksiyon adı eşleşmezse veya argümanlar geçersizse, metot `false` döndürerek çağrının bu handler tarafından işlenmediğini belirtir.

#### Oyunda Kullanım Amacı ve Senaryoları

`ClientV8ExtensionHandler`, CEF tabanlı web arayüzlerinden oyunun C++ tarafıyla çift yönlü bir iletişim kurmak için hayati bir rol oynar:

1.  **JavaScript'ten C++ Fonksiyonlarını Çağırma**:
    *   Web sayfasındaki JavaScript kodu, `CefRegisterExtension` ile kaydedilmiş ve bu handler'a bağlanmış fonksiyonları çağırabilir (örneğin, `window.app.ChangeTextInJS("merhaba");`).
    *   `Execute` metodu bu çağrıyı alır.

2.  **Veri Aktarımı ve İşleme**:
    *   JavaScript'ten C++'a veri (argümanlar aracılığıyla) gönderilebilir.
    *   C++ bu veriyi işleyebilir, oyun mantığını tetikleyebilir veya oyun durumunu değiştirebilir.

3.  **C++'tan JavaScript'e Geri Bildirim/Çağrı**:
    *   `Execute` metodu, `retval` parametresi aracılığıyla doğrudan JavaScript'e bir sonuç değeri döndürebilir.
    *   Bu örnekteki `ChangeTextInJS` fonksiyonunda olduğu gibi, C++ tarafı aldığı bilgiye göre tarayıcı çerçevesinde yeni JavaScript kodları da çalıştırabilir (`frame->ExecuteJavaScript`). Bu, C++'ın web sayfasındaki elementleri dinamik olarak değiştirmesini veya başka JavaScript fonksiyonlarını tetiklemesini sağlar.

4.  **Örnek Senaryo (`ChangeTextInJS`)**:
    *   Bir web arayüzünde bir JavaScript olayı (örneğin, bir butona tıklama) `app.ChangeTextInJS("Yeni Başlık")` gibi bir çağrı yapabilir.
    *   `ClientV8ExtensionHandler::Execute` bu çağrıyı alır.
    *   `Execute` metodu, aldığı `"Yeni Başlık"` string'ini kullanarak `ChangeText('Yeni Başlık');` şeklinde yeni bir JavaScript komutu oluşturur.
    *   Bu komut, web sayfasında tanımlı olan `ChangeText` adlı JavaScript fonksiyonunu çalıştırır ve bu fonksiyon da muhtemelen sayfadaki bir metin elementini günceller.

Bu sınıf, `CefClientApp` içinde `CefRegisterExtension` ile birlikte kullanılarak, CEF'in render sürecinde çalışan JavaScript kodunun oyunun C++ mantığıyla etkileşim kurmasına olanak tanır ve web tabanlı UI'ların oyunla daha derinlemesine entegre olmasını sağlar.

### `CefWebBrowser.h` ve `CefWebBrowser.cpp`

#### Dosyaların Genel Amacı

`CefWebBrowser.h` ve `CefWebBrowser.cpp` dosyaları, Chromium Embedded Framework (CEF) kullanarak oyun istemcisi içinde bir web tarayıcı penceresi oluşturmak, yönetmek ve göstermek için gerekli olan C-tarzı global fonksiyonları ve temel altyapıyı sağlar. Bu dosyalar, CEF'in başlatılması, ana tarayıcı penceresinin (bir Windows penceresi olarak) oluşturulması, belirli bir URL'nin yüklenmesi, pencerenin gösterilmesi/gizlenmesi/taşınması ve CEF'in sonlandırılması gibi işlemleri kapsar. Doğrudan bir sınıf yapısı sunmak yerine, `extern "C"` bloğu içinde tanımlanan ve oyunun C++ veya Python tarafından çağrılabilecek bir API seti sunar.

#### Başlık Dosyası Tanımları (`CefWebBrowser.h`)

`CefWebBrowser.h` dosyası, dışarıya açılan C fonksiyonlarının prototiplerini içerir.

*   **`#include "CefClientApp.h"` ve `#include "CefClientHandler.h"`**: Daha önce analiz ettiğimiz `ClientApp` (CEF uygulama süreçlerini yönetir) ve `ClientHandler` (tarayıcı yaşam döngüsü ve olaylarını yönetir) başlık dosyalarını dahil eder. Bu, `CefWebBrowser.cpp` içinde bu sınıfların örneklerinin oluşturulacağını gösterir.
*   **`extern "C"` Bloğu**: Bu blok içindeki fonksiyonlar, C bağlantı kurallarına göre derlenir. Bu, genellikle C++ kodu ile diğer diller (veya farklı C++ derleyicileriyle derlenmiş modüller) arasında uyumluluk sağlamak için kullanılır. Python'dan `ctypes` veya benzeri bir kütüphane ile bu fonksiyonlara erişim kolaylaşır.
    *   **`int CefWebBrowser_Startup(HINSTANCE hInstance)`**: CEF'i ve web tarayıcı altyapısını başlatır. Uygulamanın `HINSTANCE`'ını alır. Bir pencere sınıfını kaydeder.
    *   **`void CefWebBrowser_Cleanup()`**: CEF'i ve ayrılan kaynakları temizler, pencere sınıfının kaydını siler.
    *   **`void CefWebBrowser_Destroy()`**: Mevcut aktif web tarayıcı penceresini yok eder (aslında `CefWebBrowser_Hide` çağırır).
    *   **`int CefWebBrowser_Show(HWND parent, const char* addr, const RECT* rcWebBrowser)`**: Belirtilen `parent` penceresinin bir alt penceresi olarak yeni bir CEF tarayıcı penceresi oluşturur, verilen `addr` (URL) adresini yükler ve `rcWebBrowser` ile belirtilen boyut ve konumda gösterir.
    *   **`void CefWebBrowser_Hide()`**: Mevcut web tarayıcı penceresini gizler ve yok eder.
    *   **`void CefWebBrowser_Move(const RECT* rcWebBrowser)`**: Mevcut web tarayıcı penceresini yeni `rcWebBrowser` koordinatlarına ve boyutuna taşır.
    *   **`int CefWebBrowser_IsVisible()`**: Web tarayıcı penceresinin o anda görünür olup olmadığını kontrol eder.
    *   **`const RECT& CefWebBrowser_GetRect()`**: Web tarayıcı penceresinin mevcut `RECT` (dikdörtgen) bilgilerini döndürmesi beklenir ancak implementasyonu `CefWebBrowser.cpp` dosyasında bulunmamaktadır.

#### C++ Implementasyon Detayları (`CefWebBrowser.cpp`)

`CefWebBrowser.cpp` dosyası, `.h` dosyasında tanımlanan fonksiyonların ve yardımcı mekanizmaların implementasyonunu içerir.

*   **Global Değişkenler**:
    *   `ClientHandler* g_handler = nullptr;`: Global bir `ClientHandler` işaretçisi. `CefWebBrowser_Show` içinde oluşturulan `ClientHandler` örneğine atanır.
    *   `static const char* CEF_WEBBROWSER_CLASSNAME = "WEBBROWSER";`: Tarayıcı penceresi için kullanılacak Windows pencere sınıfının adı.
    *   `static HINSTANCE gs_hInstance = nullptr;`: Uygulamanın `HINSTANCE`'ını saklar.
    *   `static HWND gs_hWndCefWebBrowser = nullptr;`: Oluşturulan CEF tarayıcı penceresinin handle'ını saklar.
    *   `static HWND gs_hWndParent = nullptr;`: Tarayıcı penceresinin ebeveyn penceresinin handle'ını saklar.

*   **Pencere Prosedürleri (Window Procedures)**:
    *   `LRESULT CALLBACK MessageWndProc(...)`: `CreateMessageWindow` tarafından oluşturulan "mesaj-sadece" penceresi için basit bir pencere prosedürü.
    *   `HWND CreateMessageWindow(HINSTANCE hInstance)`: CEF'in bazı süreçler arası iletişim veya mesajlaşma mekanizmaları için ihtiyaç duyabileceği, görünmeyen bir "mesaj-sadece" penceresi oluşturur. (Not: Bu kodda doğrudan CEF için kullanıldığı açıkça belirtilmese de, CEF'in eski versiyonlarında veya belirli konfigürasyonlarda bu tür pencereler gerekebilirdi. Ancak modern CEF'te `multi_threaded_message_loop` ile bu genellikle gereksizdir.)
    *   `LRESULT CALLBACK CefWebBrowser_WindowProc(...)`: `gs_hWndCefWebBrowser` için ana pencere prosedürü. `WM_DESTROY` ve `WM_SIZE` mesajlarını basitçe ele alır.

*   **Ana Fonksiyonlar**:
    *   **`int CefWebBrowser_Startup(HINSTANCE hInstance)`**:
        1.  Uygulamanın `hInstance`'ını `gs_hInstance`'a kaydeder.
        2.  `CefMainArgs` ve `ClientApp` oluşturur.
        3.  `CefExecuteProcess` çağırır. Eğer bu fonksiyon 0 veya daha büyük bir değer döndürürse, mevcut sürecin CEF için ayrılmış bir alt süreç (örneğin, render veya GPU süreci) olduğu anlamına gelir ve ana uygulama bu noktada `exit()` ile sonlanmalıdır.
        4.  `CEF_WEBBROWSER_CLASSNAME` adıyla bir Windows pencere sınıfı (`WNDCLASSEX`) tanımlar ve `RegisterClassEx` ile kaydeder. Bu sınıf, `gs_hWndCefWebBrowser` için kullanılacaktır.
        5.  Başarılı olursa 1 döndürür.

    *   **`void CefWebBrowser_Cleanup()`**:
        1.  Eğer `gs_hInstance` geçerliyse, `UnregisterClass` ile `CEF_WEBBROWSER_CLASSNAME` pencere sınıfının kaydını siler.
        2.  `CefShutdown()` çağırarak CEF'i kapatır ve kaynaklarını serbest bırakır.

    *   **`int CefWebBrowser_Show(HWND hParent, const char* addr, const RECT* rc)`**:
        1.  Eğer zaten bir tarayıcı penceresi (`gs_hWndCefWebBrowser`) varsa bir şey yapmadan 0 döndürür.
        2.  `gs_hWndParent`'ı saklar.
        3.  `CreateWindowEx` kullanarak `CEF_WEBBROWSER_CLASSNAME` sınıfından, `hParent`'ın alt penceresi olarak `gs_hWndCefWebBrowser`'ı oluşturur. Boyut ve konum `rc`'den alınır.
        4.  Pencere oluşturma başarısız olursa 0 döndürür.
        5.  `CefMainArgs`, `ClientApp` tekrar oluşturulur (bu kısım `Startup` ile tekrar ediyor gibi görünse de, `CefInitialize` öncesi her zaman `CefExecuteProcess` kontrolü önemlidir).
        6.  `CefSettings` ayarlanır:
            *   `multi_threaded_message_loop = true;`: CEF'in kendi mesaj döngüsünü ayrı bir thread'de çalıştırmasını sağlar. Bu, ana uygulama mesaj döngüsüyle çakışmaları önler ve entegrasyonu kolaylaştırır.
            *   `log_severity = LOGSEVERITY_DISABLE;`: Debug olmayan buildlerde loglamayı kapatır.
            *   `no_sandbox = true;`: Sandbox özelliğini devre dışı bırakır. Sandbox, güvenliği artırır ancak bazı durumlarda (özellikle eski sistemlerde veya dosya sistemi erişimi gibi ihtiyaçlarda) sorun çıkarabilir. Geliştirme sırasında veya özel ihtiyaçlar için kapatılabilir.
            *   `cache_path`: Önbellek dosyalarının saklanacağı yolu kullanıcıya özel bir geçici dizin olarak ayarlar (`C:/Users/%USERNAME%/AppData/Local/Temp/m2CefBrowser`).
        7.  `CefInitialize` çağrılarak CEF çalışma zamanı başlatılır.
        8.  `CefWindowInfo info` ve `CefBrowserSettings b_settings` oluşturulur.
        9.  Yeni bir `ClientHandler` örneği oluşturulur ve global `g_handler`'a atanır.
        10. Yüklenecek `path` (URL), `addr` parametresinden veya komut satırı argümanı (`--url`) varsa oradan alınır.
        11. `info.SetAsChild(gs_hWndCefWebBrowser, rect)`: `CefWindowInfo`'yu, tarayıcının `gs_hWndCefWebBrowser` penceresinin içine gömülecek şekilde ayarlar.
        12. `CefBrowserHost::CreateBrowser` çağrılarak asıl tarayıcı örneği oluşturulur ve belirtilen `path` yüklenir.
        13. `ShowWindow(gs_hWndCefWebBrowser, SW_SHOW)` ve `UpdateWindow` ile pencere görünür hale getirilir.
        14. Eğer `multi_threaded_message_loop` false ise (ki bu örnekte true ayarlanmış), `CefRunMessageLoop()` çağrılır. Bu, ana thread'i bloke ederdi. True olduğu için bu blok atlanır.
        15. Ebeveyn pencereye odak geri verilir (`SetFocus(gs_hWndCefWebBrowser)` yerine `SetFocus(gs_hWndParent)` daha mantıklı olabilir, ancak kodda `gs_hWndCefWebBrowser`'a odaklanıyor).
        16. Başarılı olursa 1 döndürür. (Not: `CefShutdown()` çağrısı `Show` fonksiyonunun sonunda olmamalı, genellikle `Cleanup` içinde olmalıdır. Burada olması, her `Show` sonrası CEF'in kapanmasına neden olabilir, bu muhtemelen bir hata veya özel bir kullanım senaryosudur.)

    *   **`void CefWebBrowser_Move(const RECT* rc)`**:
        *   `MoveWindow` Windows API fonksiyonunu kullanarak `gs_hWndCefWebBrowser`'ı belirtilen yeni `RECT`'e taşır.

    *   **`void CefWebBrowser_Hide()`**:
        1.  Eğer `gs_hWndCefWebBrowser` yoksa bir şey yapmaz.
        2.  `ShowWindow(gs_hWndCefWebBrowser, SW_HIDE)` ile pencereyi gizler.
        3.  `DestroyWindow(gs_hWndCefWebBrowser)` ile pencereyi yok eder.
        4.  `gs_hWndCefWebBrowser`'ı `nullptr` yapar.
        5.  Ebeveyn pencereye (`gs_hWndParent`) odaklanır.

    *   **`int CefWebBrowser_IsVisible()`**:
        *   `gs_hWndCefWebBrowser`'ın `nullptr` olup olmadığını kontrol ederek görünürlük durumu hakkında bilgi verir.

    *   **`void CefWebBrowser_Destroy()`**:
        *   Doğrudan `CefWebBrowser_Hide()` fonksiyonunu çağırır.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu C API'si, oyunun herhangi bir yerinden CEF tabanlı bir web tarayıcısını kolayca entegre etmek için kullanılır:

1.  **Başlatma**: Oyun istemcisi başladığında, genellikle ana uygulama başlatma kodunda `CefWebBrowser_Startup(hInstance)` çağrılır. Bu, CEF'i ve gerekli Windows altyapısını hazırlar.
2.  **Tarayıcı Gösterme**: Oyuncu item-shop'u açmak, bir web tabanlı duyuruyu görüntülemek veya başka bir web içeriğiyle etkileşim kurmak istediğinde, oyun mantığı `CefWebBrowser_Show(hOyunPenceresi, "https://example.com", &tarayiciBoyutu)` gibi bir çağrı yapar. Bu, oyun penceresinin içinde belirtilen URL'yi yükleyen bir web tarayıcısı oluşturur.
3.  **Etkileşim ve Yönetim**:
    *   Tarayıcı penceresinin boyutu veya konumu değişmesi gerektiğinde `CefWebBrowser_Move` çağrılır.
    *   Tarayıcının görünür olup olmadığını kontrol etmek için `CefWebBrowser_IsVisible` kullanılır.
4.  **Kapatma/Gizleme**: Kullanıcı web arayüzünü kapattığında veya ihtiyaç kalmadığında `CefWebBrowser_Hide` (veya `CefWebBrowser_Destroy`) çağrılarak tarayıcı penceresi kapatılır ve kaynakları serbest bırakılır.
5.  **Temizleme**: Oyun istemcisi kapatılırken, `CefWebBrowser_Cleanup()` çağrılarak tüm CEF altyapısı düzgün bir şekilde sonlandırılır.

`CefWebBrowser.cpp`'deki `CefShutdown()` çağrısının `CefWebBrowser_Show` fonksiyonunun sonunda yer alması, eğer birden fazla tarayıcı penceresi açılıp kapanacaksa veya aynı tarayıcı örneği farklı URL'lerle yeniden kullanılacaksa sorun yaratabilir. Tipik olarak `CefInitialize` ve `CefShutdown` uygulama ömrü boyunca birer kez çağrılır. Eğer her `Show` bir `Initialize` ve `Shutdown` döngüsü ise, bu performans sorunlarına ve beklenmedik davranışlara yol açabilir. Kodun bu kısmı, kullanım senaryosuna göre dikkatle incelenmelidir.

### `CheckLatestFiles.h` ve `CheckLatestFiles.cpp`

#### Modülün Genel Amacı

`CheckLatestFiles` modülü, `#if defined(CHECK_LATEST_DATA_FILES)` önişlemci direktifi ile şartlı olarak derlenen bir özelliktir. Temel amacı, oyun istemcisi başlatıldığında belirli önemli veri dosyalarının bütünlüğünü ve güncelliğini doğrulamaktır. Bu doğrulama, dosyaların CRC32 checksum (özet) değerlerini, `CRC32_inc.h` başlık dosyasında tanımlanmış olan beklenen değerlerle karşılaştırarak yapılır. Eğer bir dosya eksikse, okunamıyorsa veya CRC32 özeti eşleşmiyorsa, bu modül genellikle oyunun daha fazla ilerlemesini engelleyerek bir hata durumu oluşturur ve istemcinin kapanmasına neden olabilir. Kontrol işlemi, oyunun ana iş parçacığını (thread) etkilememek için düşük öncelikli ayrı bir iş parçacığında çalıştırılır.

#### Başlık Dosyası Tanımları (`CheckLatestFiles.h`)

`CheckLatestFiles.h` dosyası, modülün dışarıya açılan fonksiyonlarını tanımlar.

*   **`#if defined(CHECK_LATEST_DATA_FILES)`**: Tüm içerik bu direktifle sarmalanmıştır, yani bu özellik sadece `CHECK_LATEST_DATA_FILES` makrosu tanımlıysa derlenir.
*   **`bool CheckLatestFiles(void);`**: Dosya kontrol sürecini başlatan ana fonksiyondur. Genellikle oyun başlangıcında çağrılır.
*   **`bool CheckLatestFiles_PollEvent(void);`**: Dosya kontrol süreci sonucunda bir hata (örneğin, CRC uyuşmazlığı) tespit edilip edilmediğini ve istemcinin sonlandırılması gerekip gerekmediğini sorgulamak için kullanılır.

#### C++ Implementasyon Detayları (`CheckLatestFiles.cpp`)

`CheckLatestFiles.cpp` dosyası, dosya kontrol mekanizmasının asıl mantığını içerir.

*   **`#include "resource.h"`**: Muhtemelen hata mesajları için string kaynaklarını (`IDS_ERR_CANNOT_READ_FILE`, `IDS_ERR_NOT_LATEST_FILE` gibi) içerir.
*   **`static struct SCHECKFILELIST s_astCRC32FileList[] = { ... };`**:
    *   Kontrol edilecek dosyaların bir listesini tutan statik bir dizi tanımlar. Her eleman (`SCHECKFILELIST` yapısı) dosya adını (`LPCSTR szFileName`), beklenen CRC32 değerini (`DWORD dwCRC32`) ve dosya boyutunu (`ULONGLONG ullSize` - bu örnekte kullanılmıyor gibi görünüyor) içerir.
    *   **`#include "CRC32_inc.h"`**: Bu satır, `s_astCRC32FileList` dizisinin asıl verilerini (dosya adları ve CRC32 değerleri) içeren harici bir başlık dosyasını dahil eder. Bu dosya, projenin build süreci sırasında otomatik olarak veya manuel olarak güncellenen, kontrol edilecek dosyaların ve checksum'larının bir listesini barındırır.
*   **`static bool gs_bQuit = false;`**: Eğer bir dosya hatası tespit edilirse `true` olarak ayarlanacak global bir bayraktır. `CheckLatestFiles_PollEvent` bu değeri döndürür.
*   **`bool CheckLatestFiles_PollEvent(void)`**:
    *   Basitçe `gs_bQuit` global değişkeninin değerini döndürür. Oyunun ana döngüsü bu fonksiyonu çağırarak dosya kontrolünde bir sorun olup olmadığını anlayabilir.
*   **`bool CheckFileCRC32(LPCSTR szFileName, DWORD dwCRC32)`**:
    *   Belirtilen dosyanın CRC32 değerini hesaplar ve beklenen `dwCRC32` ile karşılaştırır.
    *   `_access(szFileName, 4)`: Dosyanın okunabilir olup olmadığını kontrol eder. Okunamazsa, `ApplicationStringTable_GetStringz` ile yerelleştirilmiş bir hata mesajı alınır, `ApplicationSetErrorString` ile bu mesaj bir yere kaydedilir (muhtemelen kullanıcıya gösterilmek üzere) ve `false` döner.
    *   `DWORD dwLocalCRC32 = GetFileCRC32(szFileName);`: Dosyanın CRC32 değerini hesaplayan bir fonksiyonu çağırır (bu fonksiyonun implementasyonu bu dosyada yoktur, harici bir kütüphane veya yardımcı fonksiyondur).
    *   Eğer hesaplanan CRC32 beklenenle uyuşmazsa, benzer şekilde bir hata mesajı oluşturulur ve `false` döner.
    *   Tüm kontroller başarılıysa `true` döner.
*   **`UINT CALLBACK CheckLatestFilesEntry(void* pThis)`**:
    *   Bu fonksiyon, dosya kontrollerini gerçekleştiren ayrı bir iş parçacığının giriş noktasıdır.
    *   `::Sleep(500);`: Başlangıçta kısa bir bekleme yapar.
    *   `s_astCRC32FileList` dizisindeki her dosya için `CheckFileCRC32` fonksiyonunu çağırır.
    *   Eğer herhangi bir dosya için `CheckFileCRC32` `false` dönerse, `gs_bQuit` bayrağını `true` olarak ayarlar ve döngüden çıkar.
    *   İş parçacığı sonlandığında 0 döndürür.
*   **`bool CheckLatestFiles(void)`**:
    *   `_beginthreadex` kullanarak `CheckLatestFilesEntry` fonksiyonunu yeni bir iş parçacığında başlatır.
    *   `::SetThreadPriority(hThread, THREAD_PRIORITY_LOWEST);`: Oluşturulan iş parçacığının önceliğini en düşüğe ayarlar, böylece oyunun ana performansını minimum düzeyde etkiler.
    *   Kontrol sürecinin başlatıldığını belirtmek için `true` döndürür (kontrolün sonucunu değil).

#### Oyunda Kullanım Amacı ve Senaryoları

1.  **Başlangıç Kontrolü**: Oyun istemcisi ilk açıldığında, `CheckLatestFiles()` fonksiyonu çağrılarak önemli oyun dosyalarının (grafikler, konfigürasyonlar, scriptler vb.) bozuk veya eski olmadığından emin olunur.
2.  **Arka Plan İşlemi**: Kontroller, kullanıcı arayüzünü veya oyunun yüklenmesini engellememek için düşük öncelikli bir arka plan iş parçacığında yürütülür.
3.  **Hata Durumu**:
    *   Eğer `CheckFileCRC32` bir tutarsızlık (okuma hatası veya CRC uyuşmazlığı) tespit ederse, `gs_bQuit` `true` olur.
    *   Oyunun ana döngüsü veya uygun bir başka bölümü periyodik olarak `CheckLatestFiles_PollEvent()` fonksiyonunu çağırarak bu durumu kontrol eder.
    *   Eğer `gs_bQuit` `true` ise, oyun genellikle kullanıcıya bir hata mesajı gösterir (muhtemelen `ApplicationSetErrorString` ile ayarlanan mesaj) ve sonlanır. Bu, oyuncuların bozuk veya modifiye edilmiş dosyalarla oyuna girmesini ve potansiyel çökme, hile veya uyumsuzluk sorunları yaşamasını engeller.
4.  **Güncelleme ve Bakım**: Bu sistem, ayrıca oyuncuların oyun dosyalarını en son resmi sürüme güncellemelerini sağlamak için de bir mekanizma olarak işlev görebilir. Patcher (yama programı) güncellemeleri yaptıktan sonra bu kontrol, güncellemenin doğru yapıldığını teyit edebilir.

Bu modül, istemcinin stabilitesi ve güvenliği için önemli bir önleyici tedbirdir. `CHECK_LATEST_DATA_FILES` makrosu, geliştirme veya özel durumlar için bu kontrolü devre dışı bırakma esnekliği sunar.

### `CRC32_inc.h`

#### Dosyanın Genel Amacı

`CRC32_inc.h` dosyası, `CheckLatestFiles.cpp` modülü tarafından kullanılan `s_astCRC32FileList` dizisini doldurmak için gereken verileri içeren bir başlık dosyasıdır. Ancak, sağlanan içeriğe göre, bu dosya doğrudan dosya adları ve CRC32 değerlerini içermek yerine, **`#include "../EterBase/CRC32.h"`** direktifini içeriyor.

Bu durum şu anlama gelir:

*   `CRC32_inc.h` kendisi bir veri listesi tanımlamaz.
*   Bunun yerine, projenin başka bir yerinde bulunan (`../EterBase/CRC32.h`) bir dosyayı dahil eder.
*   Asıl dosya adı ve CRC32 değer listesinin `../EterBase/CRC32.h` dosyasında (veya bu dosyanın dolaylı olarak dahil ettiği başka dosyalarda) tanımlanmış olması beklenir. Bu, `CheckLatestFiles.cpp`'deki `s_astCRC32FileList` dizisinin elemanlarının `{ "dosya_adi", 0xCRCDEGERI },` formatında listelendiği yer olacaktır.

#### Oyunda Kullanım Amacı ve Senaryoları

`CheckLatestFiles.cpp` bağlamında, `CRC32_inc.h` (ve dolayısıyla `../EterBase/CRC32.h`) tarafından sağlanan veriler, oyun istemcisinin önemli dosyalarının bütünlüğünü doğrulamak için kullanılır. Build işleminde veya bir yama (patch) oluşturma sürecinde, belirli istemci dosyalarının CRC32 değerleri hesaplanır ve bu başlık dosyasına (veya onun işaret ettiği dosyaya) uygun formatta yazılır. Oyun başladığında, `CheckLatestFiles` modülü bu listedeki her dosyanın mevcut CRC32'sini hesaplar ve listedeki beklenen değerle karşılaştırır.

### `EventHandler.h`

#### Sınıfın Genel Amacı

`EventHandler` sınıfı, zamana dayalı ve isteğe bağlı olarak tekrarlayan olayları (event) yönetmek için tasarlanmış bir singleton (tek örnek) C++ sınıfıdır. Belirli bir fonksiyonun (callback), tanımlanmış bir gecikmeyle (`wait`) ve belirli bir sayıda (`count`) çalıştırılmasını sağlar. Her olay bir isimle (`name`) kaydedilir ve bu isim üzerinden yönetilebilir. Sınıf, `std::function` kullanarak genel amaçlı fonksiyonları olay olarak kabul edebilir ve zamanlamayı oyunun genel zamanlayıcısını (`CPythonApplication::Instance().GetGlobalTime()`) kullanarak yapar.

#### Başlık Dosyası Tanımları (`EventHandler.h`)

*   **`#include <functional>`**: `std::function` kullanmak için dahil edilmiştir.
*   **`#include "PythonApplication.h"`**: `CPythonApplication::Instance().GetGlobalTime()` (kısaca `now` makrosu ile erişilir) fonksiyonuna erişim için dahil edilmiştir. Bu, oyunun genel zamanını almak için kullanılır.
*   **`#define now CPythonApplication::Instance().GetGlobalTime()`**: Zamanı almak için kısa bir makro tanımlar.
*   **`struct HandlerEventInfo`**: Bir olayın tüm bilgilerini tutan yapıdır.
    *   `HandlerEventInfo(const std::function<void()> &fn, const int &_wait, const int &_count)`: Kurucu metot. Fonksiyonu, bekleme süresini (milisaniye veya oyun zaman birimi cinsinden) ve tekrar sayısını alır. `time` değişkenini, mevcut zamana `wait` ekleyerek ilk tetiklenme zamanını ayarlar.
    *   `int wait`: Olayın tekrarları arasındaki bekleme süresi.
    *   `int count`: Olayın kaç kez daha çalıştırılacağı. `-1` veya benzeri bir değer sonsuz tekrar anlamına gelebilir (kodda `0` olunca siliniyor).
    *   `int bind`: Olayın kaç kez tetiklendiğini sayar (kullanımı `Proccess` içinde `bind++` olarak görülüyor ancak başka bir yerde aktif kullanılmıyor gibi).
    *   `float time`: Olayın bir sonraki tetiklenme zamanı.
    *   `std::function<void()> func`: Tetiklenecek olan asıl fonksiyon.
*   **`class EventHandler : public CSingleton<EventHandler>`**:
    *   Sınıf, global erişim için `CSingleton` şablonundan türetilmiştir.
    *   **Yapıcı `EventHandler()` ve Yıkıcı `virtual ~EventHandler()`**: Her ikisi de `Destroy()` metodunu çağırır.
    *   **`void Destroy()`**: `EventInfoMap`'i temizleyerek tüm kayıtlı olayları siler.
    *   **`void AddEvent(const std::string& name, const std::function<void()> &func, const int &wait, const int &count)`**:
        *   Yeni bir olay ekler. Eğer aynı isimde bir olay zaten varsa, önce onu siler (`DeleteEvent(name)`).
        *   `_HAS_CXX17` makrosuna bağlı olarak `std::make_unique` veya `std::auto_ptr` kullanarak yeni bir `HandlerEventInfo` nesnesi oluşturur ve `EventInfoMap`'e (bir `std::map`) ekler.
    *   **`void DeleteEvent(const std::string& name)`**:
        *   Belirtilen isimdeki olayı `EventInfoMap`'ten siler.
    *   **`void DeleteProccess()`**: (Not: Bu fonksiyonun adı "DeleteProcess" olmalıydı, bir yazım hatası var gibi görünüyor.)
        *   Bu fonksiyon, `count` değeri 0 veya daha az olan olayları silmeye çalışır. Ancak implementasyonu biraz karmaşık ve potansiyel olarak verimsizdir. İç içe döngüler ve `break` kullanımı, haritanın ortasından eleman silerken sorunlara yol açabilir veya beklenen şekilde çalışmayabilir. Harita üzerinde iterasyon yaparken eleman silmek dikkatlice yapılmalıdır.
    *   **`void Proccess()`**: (Not: Bu fonksiyonun adı "Process" olmalıydı.)
        *   Bu metot, periyodik olarak (muhtemelen oyunun ana döngüsünde) çağrılmalıdır.
        *   `EventInfoMap`'teki tüm olayları kontrol eder.
        *   Eğer bir olayın `count`'u 0 ise, onu siler.
        *   Eğer bir olayın `time`'ı (bir sonraki tetiklenme zamanı) mevcut zamandan (`now`) küçük veya eşitse:
            *   `bind` sayacını artırır.
            *   `count`'u azaltır.
            *   Olayın `func`'ını çağırır.            
            *   Bir sonraki tetiklenme zamanını (`time`) `now + static_cast<float>(event.second->wait)` olarak günceller.
            *   Eğer `count` 0 veya daha az olmuşsa, olayı siler.
    *   **`bool FindEvent(const std::string & name)`**:
        *   Verilen isimde bir olayın `EventInfoMap`'te olup olmadığını kontrol eder.
    *   **`GetHandler(const std::string& name)`**:
        *   `_HAS_CXX17` makrosuna göre `std::unique_ptr<HandlerEventInfo>*` veya `std::auto_ptr<HandlerEventInfo>*` döndürür.
        *   Belirtilen isimdeki olayın `HandlerEventInfo` nesnesine bir işaretçi döndürür. Bulamazsa `nullptr` döndürür.
*   **Özel Üye Değişken**:
    *   `#ifdef _HAS_CXX17` bloğuna göre `std::map<std::string, std::unique_ptr<HandlerEventInfo>> EventInfoMap;` veya `std::map<std::string, std::auto_ptr<HandlerEventInfo>> EventInfoMap;`
    *   Olayları isimleriyle eşleştirerek saklayan haritadır. Akıllı işaretçiler (`unique_ptr` veya `auto_ptr`) bellek yönetimini kolaylaştırır. (Not: `std::auto_ptr` C++11'den sonra kullanımdan kaldırılmıştır ve C++17 desteği varsa `std::unique_ptr` tercih edilir.)

#### Oyunda Kullanım Amacı ve Senaryoları

`EventHandler` sınıfı, oyun içinde zamanlanmış görevleri veya periyodik eylemleri yönetmek için çok kullanışlı bir araçtır:

1.  **Geçici Efektler**: Bir UI elemanının belirli bir süre sonra kaybolması (örneğin, bir bildirim mesajı).
    *   `EventHandler::Instance().AddEvent("HideNotification", [&]() { notificationWindow->Hide(); }, 5000, 1);` // 5 saniye sonra 1 kez çalışır.

2.  **Periyodik Güncellemeler**: Belirli aralıklarla bir durumu kontrol etmek veya bir değeri güncellemek.
    *   `EventHandler::Instance().AddEvent("UpdatePlayerBuffs", std::bind(&MyClass::RefreshBuffDisplay, this), 1000, -1);` // Her 1 saniyede bir sürekli çalışır (count için özel bir değer -1 ise).

3.  **Animasyon Adımları**: Basit, zamana dayalı animasyonların adımlarını tetiklemek.
    *   Bir nesneyi birkaç saniye içinde yavaşça hareket ettirmek için birden fazla küçük gecikmeli olay eklenebilir.

4.  **Gecikmeli İşlemler**: Bir eylemin hemen değil, kısa bir süre sonra yapılmasını sağlamak.
    *   Oyuncu bir yetenek kullandığında, yeteneğin görsel efektinin X milisaniye sonra başlaması için bir olay eklenebilir.

5.  **Görev Zamanlayıcıları**: Belirli bir süre aktif olan görevlerin takibi.

`Proccess()` metodunun oyunun ana güncelleme döngüsünde düzenli olarak çağrılması, olayların doğru zamanlarda tetiklenmesi için kritik öneme sahiptir. `DeleteProccess()` fonksiyonunun mevcut implementasyonu, harita üzerinde iterasyon yaparken eleman silmenin getirdiği karmaşıklıklar nedeniyle gözden geçirilmelidir; daha güvenli bir yaklaşım, silinecek olayları bir listeye ekleyip iterasyon bittikten sonra topluca silmek olabilir.

### `GameType.h` ve `GameType.cpp`

#### Dosyaların Genel Amacı

`GameType.h` ve `GameType.cpp` dosyaları, Metin2 istemcisinde kullanılan temel veri yapılarını, sabitleri, enumları ve bazı yardımcı fonksiyonları tanımlar. Bu dosyalar, oyun içindeki envanter yönetimi, eşya pozisyonları, karakter etkileri (affect), ekipman slotları, pencere türleri, hızlı erişim çubuğu, varsayılan font yönetimi, guild sembol yolları ve çeşitli oyun mekanikleriyle ilgili temel tanımlamalar için bir merkez görevi görür. Özellikle `#if defined(ENABLE_...)` gibi önişlemci direktifleri aracılığıyla birçok isteğe bağlı sistemin (örneğin, kostüm sistemleri, özel envanterler, yeni ekipman slotları, aura sistemi vb.) tanımlarını ve sabitlerini içerir.

#### Başlık Dosyası Tanımları (`GameType.h`)

`GameType.h` dosyası, oyun genelinde kullanılacak birçok temel tanımı barındırır:

*   **`struct SAffects`**: Karakter üzerindeki aktif etkileri (buff/debuff) bit flag'leri olarak yönetmek için basit bir yapı.
*   **Global String Değişkenleri (extern)**:
    *   `g_strGuildSymbolPathName`: Lonca sembollerinin bulunduğu dizin yolunu tutar.
*   **Sabitler**:
    *   İsim ve dosya adı maksimum uzunlukları (`c_Name_Max_Length`, `c_FileName_Max_Length` vb.).
    *   Envanter sayfa boyutu ve sayısı (`c_Inventory_Page_Size`, `c_Inventory_Page_Count`).
    *   Çeşitli özel envanterlerin (beceri kitabı, materyal vb.) sayfa boyutu ve sayısı (`#if defined(ENABLE_SPECIAL_INVENTORY_SYSTEM)`).
    *   Ekipman slot sayıları ve başlangıç indeksleri (`c_Equipment_Count`, `c_Equipment_Start`).
    *   Kostüm slotları, Ejderha Taşı Simyası ekipman slotları, kemer envanteri gibi birçok özelliğe özel slot başlangıç/bitiş indeksleri ve sayıları (genellikle `#if defined` blokları içinde).
    *   Para limitleri (`CHEQUE_MAX`, `GOLD_MAX`).
*   **Enum Tanımları**:
    *   `ELevelTypes` (`ENABLE_CONQUEROR_LEVEL`): Temel seviye ve fatih seviyesi gibi seviye türleri.
    *   `ETopWindowTypes` (`WJ_ENABLE_TRADABLE_ICON`): Üstte kalan pencere türleri (ticaret, depo vb.).
    *   `EKeySettings` (`ENABLE_KEYCHANGE_SYSTEM`): Ayarlanabilir tuş atamaları için eylemler.
    *   `ESkillBookComb` (`ENABLE_SKILLBOOK_COMB_SYSTEM`): Beceri kitabı birleştirme slot sayısı.
    *   `ELootFilter` (`ENABLE_LOOTING_SYSTEM`): Otomatik toplama sistemi için filtre ayarları.
    *   `ESlotType`: Envanter, beceri, dükkan gibi genel slot türleri.
    *   `EWindows`: Envanter, ekipman, depo gibi ana pencere türleri.
    *   `EDSInventoryMaxNum`: Ejderha Taşı Simyası envanter ve arındırma penceresi maksimum slot sayıları.
    *   `EChangeLookType`, `EChangeLookSlots`, `EChangeLookPrice`, `EChangeLookItems` (`ENABLE_CHANGE_LOOK_SYSTEM`): Eşya görünümü değiştirme sistemiyle ilgili türler, slotlar, fiyatlar ve eşyalar.
    *   `ERefineElement` (`ENABLE_REFINE_ELEMENT`): Elementer arındırma sistemiyle ilgili sabitler.
    *   `EAuraRefineInfoSlot`, `EAuraWindowType`, `EAuraSlotType`, `EAuraRefineInfoType` (`ENABLE_AURA_COSTUME_SYSTEM`): Aura kostüm sistemiyle ilgili slotlar, pencere türleri ve arındırma bilgileri.
    *   `ESetItemType` (`ENABLE_SET_ITEM`): Set eşya bonus türleri.
    *   `ECombSlotType` (`ENABLE_MOVE_COSTUME_ATTR`): Kostüm efsun aktarma arayüzü slot türleri.
    *   `EDSRefineType` (`ENABLE_DS_CHANGE_ATTR`): Ejderha Taşı Simyası efsun değiştirme türleri.
    *   `EAttr67` (`ENABLE_ATTR_6TH_7TH`): 6. ve 7. efsun sistemi için NPC depo slot sayısı.
    *   `ITEM_SOCKET_SLOT_MAX_NUM`, `ITEM_ATTRIBUTE_SLOT_MAX_NUM` gibi eşya soket ve efsun slot sayıları.
*   **Yapılar (Structs)**:
    *   **`SItemPos` (TItemPos)**: Bir eşyanın pencere türü (`window_type`) ve hücre numarasını (`cell`) tutan temel yapı. Eşya konumlarını temsil eder. `IsValidCell`, `IsEquipCell` gibi yardımcı metotları vardır.
    *   `SQuickSlot` (TQuickSlot): Hızlı erişim çubuğundaki bir slotun türünü ve pozisyonunu tutar.
    *   `TPlayerItemAttribute`: Bir eşya efsununun türünü (`wType`) ve değerini (`sValue`) tutar. (`ENABLE_APPLY_RANDOM` için `bPath` da içerebilir).
    *   `SItemPriceType` (TItemPriceType) (`ENABLE_CHEQUE_SYSTEM`): Bir eşyanın Yang ve Won (Cheque) cinsinden fiyatını tutar.
    *   `packet_item` (TItemData): Bir eşyanın ağ üzerinden veya envanterde saklanırken kullanılan detaylı yapısıdır. Eşya VNUM'ı, sayısı, bayrakları, soketleri, efsunları ve çeşitli sistemlere (görünüm değiştirme, ruh bağlama, elementer arındırma, set eşyası vb.) ait özel verileri içerir.
    *   `packet_shop_item` (TShopItemData): Dükkandaki bir eşyanın bilgilerini tutar (vnum, fiyat, adet, efsunlar vb.).
    *   `SGrowthPetInfo` (TGrowthPetInfo) (`ENABLE_GROWTH_PET_SYSTEM`): Geliştirilebilir evcil hayvan sistemi için hayvanın tüm bilgilerini (ID, seviye, HP, EXP, yetenekler vb.) tutan yapı.
    *   `SAttr67AddData` (TAttr67AddData) (`ENABLE_ATTR_6TH_7TH`): 6. ve 7. efsun ekleme işlemi için gerekli verileri tutan yapı.
*   **Fonksiyon Protototipleri**:
    *   `DefaultFont_Startup`, `DefaultFont_Cleanup`, `DefaultFont_SetName`, `DefaultFont_GetResource`, `DefaultItalicFont_GetResource`: Varsayılan oyun fontlarını yönetmek için fonksiyonlar.
    *   `SetGuildSymbolPath`, `GetGuildSymbolFileName`: Lonca sembolü dosya yollarını ayarlamak ve almak için.
    *   `SlotTypeToInvenType`, `WindowTypeToSlotType`: Slot türleri ve pencere türleri arasında dönüşüm yapmak için.
    *   `ApplyTypeToPointType`, `PointTypeToApplyType` (`ENABLE_DETAILS_UI`): Eşya efsun türleri (`APPLY_`) ile karakter puan türleri (`POINT_`) arasında dönüşüm yapmak için.
    *   `GetAuraRefineInfo` (`ENABLE_AURA_COSTUME_SYSTEM`): Aura seviyesine göre arındırma bilgilerini almak için.

#### C++ Implementasyon Detayları (`GameType.cpp`)

`GameType.cpp` dosyası, `.h` dosyasında deklare edilen global değişkenlerin ve fonksiyonların implementasyonlarını içerir.

*   **Global Değişkenler**:
    *   `g_strResourcePath`, `g_strImagePath`: Kaynak ve resim dosyaları için varsayılan yollar (genellikle "d:/ymir work/" gibi geliştirme ortamına özgü yollar).
    *   `g_strGuildSymbolPathName`: Lonca sembollerinin temel yolu, `SetGuildSymbolPath` ile değiştirilebilir.
    *   `gs_strDefaultFontName`, `gs_strDefaultItalicFontName`: Varsayılan ve italik font dosyalarının adları.
    *   `gs_pkDefaultFont`, `gs_pkDefaultItalicFont`: Yüklenen varsayılan font kaynaklarına işaretçiler.
    *   `gs_isReloadDefaultFont`: Fontların yeniden yüklenmesi gerekip gerekmediğini belirten bir bayrak.
*   **Varsayılan Font Yönetimi Fonksiyonları**:
    *   `DefaultFont_Startup()`: `gs_pkDefaultFont`'u `NULL` yapar.
    *   `DefaultFont_Cleanup()`: Eğer `gs_pkDefaultFont` yüklü ise referansını serbest bırakır.
    *   `DefaultFont_SetName(const char* c_szFontName)`: Verilen font adına göre varsayılan ve italik font dosyası adlarını günceller ve yeniden yükleme bayrağını (`gs_isReloadDefaultFont`) ayarlar.
    *   `ReloadDefaultFonts()`: `CResourceManager` kullanarak font dosyalarını (hem normal hem italik) yükler veya yeniden yükler, eski fontların referanslarını serbest bırakır ve yenilerini atar.
    *   `DefaultFont_GetResource()`, `DefaultItalicFont_GetResource()`: Gerekirse `ReloadDefaultFonts`'u çağırarak ilgili font kaynağının işaretçisini döndürür.
*   **Lonca Sembolü Fonksiyonları**:
    *   `SetGuildSymbolPath(const char* c_szPathName)`: `g_strGuildSymbolPathName`'i günceller (örneğin, "mark/default/").
    *   `GetGuildSymbolFileName(DWORD dwGuildID)`: Verilen lonca ID'sine göre sembol dosyasının tam adını (örneğin, "mark/default/001.jpg") oluşturur ve döndürür.
*   **Slot Türü ve Pencere Türü Dönüşüm Fonksiyonları**:
    *   `c_aSlotTypeToInvenType[]`: `ESlotType` enum değerlerini ilgili envanter penceresi türüne (`EWindows`) eşleyen bir dizi.
    *   `SlotTypeToInvenType(BYTE bSlotType)`: Yukarıdaki diziyi kullanarak verilen slot türünü envanter penceresi türüne dönüştürür.
    *   `c_aWndTypeToSlotType[]`: `EWindows` enum değerlerini ilgili slot türüne (`ESlotType`) eşleyen bir dizi.
    *   `WindowTypeToSlotType(BYTE bWindowType)`: Yukarıdaki diziyi kullanarak verilen pencere türünü slot türüne dönüştürür. Bu eşlemeler, birçok `#if defined` bloğu ile sistemlere özel slotları da içerir.
*   **Efsun Türü ve Puan Türü Dönüşüm Fonksiyonları (`ENABLE_DETAILS_UI`)**:
    *   `SApplyInfo` yapısı: `CItemData::APPLY_` türünü bir `POINT_` türüne eşler.
    *   `aApplyInfo[]`: `CItemData::MAX_APPLY_NUM` kadar `TApplyInfo` içeren bir dizi. Her `APPLY_` türü için karşılık gelen `POINT_` türünü tanımlar. Bu dizi çok kapsamlıdır ve oyundaki tüm temel efsunları, canavar bonuslarını, dirençleri, beceri hasar bonuslarını vb. içerir. Birçok yeni sistem için `#if defined` blokları ile genişletilmiştir.
    *   `ApplyTypeToPointType(WORD wApplyType)`: `aApplyInfo` dizisini kullanarak bir `APPLY_` türünü `POINT_` türüne dönüştürür.
    *   `PointTypeToApplyType(WORD wPointType)`: `aApplyInfo` dizisinde ters arama yaparak bir `POINT_` türünü `APPLY_` türüne dönüştürür.
*   **Aura Arındırma Bilgileri Fonksiyonu (`ENABLE_AURA_COSTUME_SYSTEM`)**:
    *   `s_aiAuraRefineInfo[][]`: Aura seviyelerine göre arındırma bilgilerini (gerekli EXP, malzeme, altın, başarı şansı vb.) tutan statik bir 2D dizi.
    *   `GetAuraRefineInfo(BYTE bLevel)`: Verilen aura seviyesine uygun arındırma bilgilerini `s_aiAuraRefineInfo` dizisinden bulup döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyalar, istemcinin temel veri tanımlamalarının çoğunu içerdiği için projenin en sık başvurulan ve en önemli başlık dosyalarından biridir. Özellikle yeni bir UI elemanı, envanter türü veya eşya ile ilgili bir özellik ekleneceği zaman bu dosyalarda değişiklik yapılması veya buradaki tanımların kullanılması gerekir.

### `GuildMarkDownloader.h` ve `GuildMarkDownloader.cpp`

#### Sınıfın Genel Amacı

`CGuildMarkDownloader` sınıfı, Metin2 istemcisinin sunucudan lonca sembollerini (hem standart lonca işaretlerini/marklarını hem de daha büyük lonca amblemlerini/sembollerini) indirmek için kullandığı özel bir ağ akışıdır (`CNetworkStream`). `CSingleton` olarak tasarlandığı için oyun içinde global olarak erişilebilir. Temel görevi, sunucuya bağlanmak, gerekli lonca sembolü bilgilerini (indeks listeleri, CRC'ler) talep etmek ve gelen sembol verilerini alıp diske kaydetmektir. Bu işlem tamamlandığında, `CPythonCharacterManager` aracılığıyla oyundaki tüm lonca sembollerinin yenilenmesini tetikler.

#### Başlık Dosyası Tanımları (`GuildMarkDownloader.h`)

`GuildMarkDownloader.h` dosyası, `CGuildMarkDownloader` sınıfının yapısını, üye değişkenlerini ve fonksiyon prototiplerini tanımlar.

*   **Kalıtım**: `CNetworkStream`'den ve `CSingleton<CGuildMarkDownloader>`'dan kalıtım alır.
*   **Enum Tanımları**:
    *   `STATE_OFFLINE`, `STATE_LOGIN`, `STATE_COMPLETE`: İndirme işleminin farklı durumlarını temsil eder.
    *   `TODO_RECV_NONE`, `TODO_RECV_MARK`, `TODO_RECV_SYMBOL`: Yapılacak işin türünü (hiçbir şey, standart mark indirme, büyük sembol indirme) belirtir.
*   **Yapıcı ve Yıkıcı**: `CGuildMarkDownloader()`, `virtual ~CGuildMarkDownloader()`.
*   **Bağlantı Fonksiyonları**:
    *   `Connect(const CNetworkAddress& c_rkNetAddr, DWORD dwHandle, DWORD dwRandomKey)`: Standart lonca işaretlerini indirmek üzere sunucuya bağlanır.
    *   `ConnectToRecvSymbol(const CNetworkAddress& c_rkNetAddr, DWORD dwHandle, DWORD dwRandomKey, const std::vector<DWORD>& c_rkVec_dwGuildID)`: Belirli loncaların büyük sembollerini indirmek üzere sunucuya bağlanır.
*   **`Process()`**: Ağ olaylarını ve durum makinesini işler.
*   **Özel (Private) Fonksiyonlar**:
    *   **Ağ Olayı İşleyicileri**: `OnConnectFailure`, `OnConnectSuccess`, `OnRemoteDisconnect`, `OnDisconnect`.
    *   **Başlatma ve Durum Yönetimi**: `__Initialize`, `__StateProcess`, `__OfflineState_Set`, `__CompleteState_Set`, `__LoginState_Set`.
    *   **Paket İşleme**: `__GetPacketSize`, `__DispatchPacket`.
    *   **Login Durumu İşleyicileri**:
        *   `__LoginState_Process`: Login durumundaki genel paket akışını yönetir.
        *   `__LoginState_RecvPhase`: Sunucudan gelen faz bilgisini alır ve yapılacak işe göre ilgili listeyi (mark indeksi veya sembol CRC) talep eder.
        *   `__LoginState_RecvHandshake`: Sunucuyla el sıkışma işlemini yapar ve istemci bilgilerini gönderir.
        *   `__LoginState_RecvPing`: Sunucudan gelen ping'e pong ile cevap verir.
        *   `__LoginState_RecvMarkIndex`: Sunucudan gelen lonca mark indeksi listesini alır ve `CGuildMarkManager`'a kaydeder.
        *   `__LoginState_RecvMarkBlock`: Sunucudan gelen sıkıştırılmış mark bloğunu alır, açar ve `CGuildMarkManager` aracılığıyla kaydeder.
        *   `__LoginState_RecvSymbolData`: Sunucudan gelen lonca sembolü verisini alır ve dosyaya yazar.
        *   `__LoginState_RecvKeyAgreement`, `__LoginState_RecvKeyAgreementCompleted` (`__IMPROVED_PACKET_ENCRYPTION__` tanımlıysa): Gelişmiş paket şifrelemesi için anahtar anlaşması adımlarını işler.
    *   **Gönderme Fonksiyonları**: `__SendMarkIDXList`, `__SendMarkCRCList`, `__SendSymbolCRCList`.
*   **Özel (Private) Üye Değişkenler**:
    *   `m_dwHandle`, `m_dwRandomKey`: Sunucu tarafından verilen oturum bilgileri.
    *   `m_dwTodo`: Yapılacak indirme işleminin türü.
    *   `m_kVec_dwGuildID`: Sembolleri indirilecek loncaların ID listesi.
    *   `m_eState`: Mevcut indirme durumu.
    *   `m_currentRequestingImageIndex`: Mark indirme sırasında talep edilen bir sonraki mark imajının indeksi.
    *   `m_pkMarkMgr`: `CGuildMarkManager` işaretçisi (ancak kodda doğrudan atanıp kullanılmıyor gibi görünüyor, genellikle `CGuildMarkManager::Instance()` üzerinden erişiliyor).
    *   `m_dwBlockIndex`, `m_dwBlockDataSize`, `m_dwBlockDataPos`: Mark bloklarını alırken kullanılan geçici değişkenler (kullanımları tam olarak belirgin değil, muhtemelen eski veya tamamlanmamış bir mantığın parçası).

#### C++ Implementasyon Detayları (`GuildMarkDownloader.cpp`)

`GuildMarkDownloader.cpp` dosyası, sınıfın fonksiyonlarının detaylı implementasyonunu içerir.

*   **`SMarkIndex` Yapısı**: `MARK_BUG_FIX` kapsamında tanımlanmış, `guild_id` ve `mark_id` içeren basit bir yapı. Ancak bu yapı, dosyanın geri kalanında aktif olarak kullanılmıyor gibi görünüyor.
*   **Yapıcı**: Alıcı ve gönderici tampon boyutlarını ayarlar, `__Initialize()` çağırır.
*   **Yıkıcı**: `__OfflineState_Set()` çağırarak durumu sıfırlar.
*   **`Connect` / `ConnectToRecvSymbol`**: Durumu sıfırlar, bağlantı bilgilerini (`m_dwHandle`, `m_dwRandomKey`) ve yapılacak işi (`m_dwTodo`) ayarlar, `CNetworkStream::Connect` ile asıl bağlantıyı kurar. `ConnectToRecvSymbol` ayrıca indirilecek lonca ID'lerini de saklar.
*   **`Process()`**: Ana ağ akışını (`CNetworkStream::Process()`) ve ardından kendi durum makinesini (`__StateProcess()`) işler. Başarısız olursa bağlantıyı keser.
*   **Ağ Olayı İşleyicileri**: Bağlantı başarılı/başarısız olduğunda veya kesildiğinde uygun durum ayarlama fonksiyonlarını (`__OfflineState_Set`, `__LoginState_Set`) çağırır.
*   **`__Initialize()`**: Tüm üye değişkenleri başlangıç değerlerine sıfırlar.
*   **`__StateProcess()`**: `m_eState`'e göre ilgili durum işleme fonksiyonunu (şu anda sadece `__LoginState_Process`) çağırır. `STATE_COMPLETE` ise `false` döner (işlem bitti).
*   **Durum Ayarlama Fonksiyonları**:
    *   `__OfflineState_Set()`: `__Initialize()` çağırır.
    *   `__CompleteState_Set()`: Durumu `STATE_COMPLETE` yapar ve `CPythonCharacterManager::instance().RefreshAllGuildMark()` çağırarak oyundaki tüm lonca işaretlerinin yenilenmesini tetikler.
    *   `__LoginState_Set()`: Durumu `STATE_LOGIN` yapar.
*   **`__LoginState_Process()`**: Gelen paketin başlığını okur, paket boyutunu belirler ve `__DispatchPacket` ile ilgili işleyiciye yönlendirir. Şifreleme modu aktifse, başlık 0 ise bunu özel bir durum olarak ele alır.
*   **`__GetPacketSize(UINT header)`**: Gelen paket başlığına göre beklenen paket boyutunu döndürür.
*   **`__DispatchPacket(UINT header)`**: Gelen paket başlığına göre ilgili `__LoginState_Recv...` fonksiyonunu çağırır.
*   **Login Durumu Alıcı Fonksiyonları**:
    *   **`__LoginState_RecvHandshake()`**: Sunucudan el sıkışma paketini alır ve istemcinin `HEADER_CG_MARK_LOGIN` paketini (handle ve random key içerir) gönderir.
    *   **`__LoginState_RecvPing()`**: Sunucudan gelen ping'e `HEADER_CG_PONG` ile yanıt verir.
    *   **`__LoginState_RecvPhase()`**: Sunucudan gelen faz bilgisini alır. Eğer faz `PHASE_LOGIN` ise:
        *   Gerekirse güvenlik modunu ayarlar (`SetSecurityMode`).
        *   `m_dwTodo`'ya göre ya `__SendMarkIDXList()` (mark indirme için) ya da `__SendSymbolCRCList()` (sembol indirme için) çağırır.
    *   **`__LoginState_RecvMarkIndex()`**: Sunucudan gelen `TPacketGCMarkIDXList` paketini alır. Bu paket, kaç adet mark indeksi olduğunu ve toplam buffer boyutunu içerir. Ardından her bir (lonca ID, mark ID) çiftini okur ve `CGuildMarkManager::Instance().AddMarkIDByGuildID()` ile kaydeder. Tüm indeksler alındıktan sonra `CGuildMarkManager::Instance().LoadMarkImages()` çağrılarak mark imajları için yerel bir yapı hazırlanır ve ardından `__SendMarkCRCList()` ile ilk mark imajının CRC listesini talep eder.
    *   **`__LoginState_RecvMarkBlock()`**: Sunucudan gelen `TPacketGCMarkBlock` paketini alır. Bu paket, hangi imaja ait olduğu (`imgIdx`), kaç blok içerdiği (`count`) ve toplam buffer boyutunu içerir. Her blok için pozisyonunu, sıkıştırılmış boyutunu ve sıkıştırılmış veriyi okur. `CGuildMarkManager::Instance().SaveBlockFromCompressedData()` ile bu veriyi kaydeder. Eğer bloklar alındıysa, `CGuildMarkManager::Instance().SaveMarkImage()` ile tam mark imajını diske yazar ve `CResourceManager` aracılığıyla bu imajı yeniden yükler. Eğer hala talep edilecek mark imajı varsa `__SendMarkCRCList()` çağrılır, yoksa `__CompleteState_Set()` ile işlem tamamlanır.
    *   **`__LoginState_RecvSymbolData()`**: Sunucudan gelen `TPacketGCGuildSymbolData` paketini alır. Bu paket, lonca ID'sini ve sembol verisinin boyutunu içerir. Ardından sembol verisini okur, `g_strGuildSymbolPathName` altına (gerekirse dizin oluşturarak) lonca ID'sine göre bir dosya adı (`GetGuildSymbolFileName`) oluşturur ve bu dosyaya sembol verisini yazar.
    *   Gelişmiş şifreleme (`__IMPROVED_PACKET_ENCRYPTION__`) aktifse, `__LoginState_RecvKeyAgreement` ve `__LoginState_RecvKeyAgreementCompleted` fonksiyonları anahtar değişim sürecini yönetir.
*   **Gönderme Fonksiyonları**:
    *   **`__SendMarkIDXList()`**: Sunucuya `HEADER_CG_MARK_IDXLIST` paketi göndererek tüm lonca marklarının (guild_id, mark_id) listesini talep eder.
    *   **`__SendMarkCRCList()`**: `CGuildMarkManager`'dan sıradaki mark imajının (`m_currentRequestingImageIndex`) blok CRC listesini alır ve `HEADER_CG_MARK_CRCLIST` paketi ile sunucuya gönderir. Eğer tüm imajlar için CRC listesi gönderilmişse `__CompleteState_Set()` çağrılır.
    *   **`__SendSymbolCRCList()`**: `m_kVec_dwGuildID` listesindeki her lonca ID'si için yerel sembol dosyasının CRC'sini ve boyutunu hesaplar (`GetFileCRC32`, `GetFileSize`) ve `HEADER_CG_GUILD_SYMBOL_CRC` paketi ile sunucuya gönderir.

#### Oyunda Kullanım Amacı ve Senaryoları

1.  **İstemci Başlangıcı / Login Sonrası**: Genellikle istemci sunucuya bağlandıktan ve karakter seçimi yapıldıktan sonra, eksik veya güncellenmesi gereken lonca işaretleri (mark) varsa, bu sınıf aracılığıyla sunucudan talep edilir. `Connect()` metodu çağrılır.
    *   Önce `__SendMarkIDXList` ile tüm loncaların (guild_id, mark_id) eşleşmeleri alınır.
    *   Sonra `__SendMarkCRCList` ile her bir mark imajı için yerel CRC'ler gönderilir. Sunucu, farklı CRC'ye sahip blokları `HEADER_GC_MARK_BLOCK` ile gönderir.
    *   Alınan bloklar birleştirilip kaydedilir ve imaj yeniden yüklenir.
2.  **Oyun İçi Sembol İhtiyacı**: Oyuncu, daha önce sembolü görülmemiş bir loncanın üyesini gördüğünde veya bir lonca arazi_savaş alanına girdiğinde, eksik olan büyük lonca sembolleri (amblemleri) anlık olarak talep edilebilir. `ConnectToRecvSymbol()` metodu, belirli lonca ID'leri listesiyle çağrılır.
    *   `__SendSymbolCRCList` ile listelenen her lonca için yerel sembol dosyasının CRC'si gönderilir.
    *   Sunucu, farklı CRC'ye sahip veya istemcide hiç olmayan sembollerin tam verisini `HEADER_GC_GUILD_SYMBOL_DATA` ile gönderir.
    *   Alınan sembol verisi doğrudan dosyaya yazılır.
3.  **Güncelleme ve Yenileme**: İndirme işlemi tamamlandığında (`__CompleteState_Set`), `CPythonCharacterManager::instance().RefreshAllGuildMark()` çağrısı ile oyundaki karakterlerin üzerindeki ve UI elemanlarındaki tüm lonca işaretleri güncellenir, böylece yeni indirilen veya güncellenen semboller görünür hale gelir.

Bu sınıf, istemcinin lonca sembollerini dinamik olarak sunucudan almasını sağlayarak, tüm sembollerin istemciyle birlikte dağıtılması gerekliliğini ortadan kaldırır ve sembollerin güncellenmesini kolaylaştırır.

--- 