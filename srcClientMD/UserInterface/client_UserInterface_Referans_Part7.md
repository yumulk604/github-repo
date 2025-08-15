# UserInterface Referans Kılavuzu - Bölüm 7

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 6'da ele alınan konuların ardından, bu bölümde istemcinin Python'a sunduğu ana `app` modülü incelenecektir.

## İçindekiler

* [`PythonApplicationModule.cpp` (Python `app` Modülü)](#pythonapplicationmodulecpp-python-app-modülü)
* [`PythonApplicationProcedure.cpp` (Pencere Mesaj İşleme)](#pythonapplicationprocedurecpp-pencere-mesaj-işleme)
* [`PythonApplicationWebPage.cpp` (Web Sayfası Yönetimi)](#pythonapplicationwebpagecpp-web-sayfası-yönetimi)
* [`PythonBackground.h`, `PythonBackground.cpp` ve `PythonBackgroundModule.cpp` (Oyun Arka Plan Yönetimi)](#pythonbackgroundh-pythonbackgroundcpp-ve-pythonbackgroundmodulecpp-oyun-arka-plan-yönetimi)
* [`PythonCharacterManager.h` ve `PythonCharacterManager.cpp` (Karakter Yönetimi)](#pythoncharactermanagerh-ve-pythoncharactermanagercpp-karakter-yönetimi)
* [`PythonCharacterManagerModule.cpp` (Python `chrmgr` Modülü)](#pythoncharactermanagermodulecpp-python-chrmgr-modülü)
* [`PythonCharacterModule.cpp` (Python `chr` Modülü)](#pythoncharactermodulecpp-python-chr-modülü)
* [`PythonChat.h` ve `PythonChat.cpp` (Sohbet Yönetimi)](#pythonchat-h-ve-pythonchat-cpp-sohbet-yönetimi)
* [`PythonChatModule.cpp` (Python `chat` Modülü)](#pythonchatmodulecpp-python-chat-modülü)
* [`PythonConfig.h` ve `PythonConfig.cpp` (Python `cfg` veya Yapılandırma Modülü)](#pythonconfigh-ve-pythonconfigcpp-python-cfg-veya-yapılandırma-modülü)

---

### `PythonApplicationModule.cpp` (Python `app` Modülü)

Bu dosya, `CPythonApplication` sınıfının ve diğer çeşitli istemci işlevlerinin Python betiklerine `app` adlı bir modül aracılığıyla sunulmasını sağlar. İstemcinin C++ tarafındaki temel fonksiyonlarının, ayarlarının ve sabitlerinin Python'dan erişilebilir ve kontrol edilebilir olmasını mümkün kılar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma**: `initapp()` fonksiyonu aracılığıyla `app` adında bir Python modülü oluşturur.
*   **Fonksiyon Köprüleme**: `CPythonApplication` sınıfının birçok metodunu (örneğin, oyun döngüsü, pencere oluşturma, kamera kontrolü, imleç yönetimi, ağ işlemleri, dosya işlemleri, yerelleştirme, performans ayarları vb.) Python'dan çağrılabilir C fonksiyonları (`app*` önekli) olarak sarmalar ve `app` modülüne kaydeder.
*   **Sabitlerin Aktarılması**: Oyunla ilgili önemli C++ sabitlerini (klavye tuş kodları, imleç türleri, kamera yönleri, yerelleştirme kodları, oyun versiyonu vb.) `app` modülüne Python tamsayı sabitleri olarak ekler.
*   **Derleme Zamanı Yapılandırmalarının Aktarılması**: İstemcinin derlenmesi sırasında aktif olan veya olmayan sistem özelliklerini (`ENABLE_*` veya `WJ_*` gibi C++ makroları) boolean (doğru/yanlış) sabitler olarak `app` modülüne ekler. Bu, Python betiklerinin belirli özelliklerin istemcide mevcut olup olmadığını kontrol etmesine olanak tanır.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **Fonksiyon Sarmalayıcıları (`app*` önekli fonksiyonlar)**:
    *   Her bir C++ fonksiyonu (genellikle `CPythonApplication::Instance().MethodAdi()`) için bir Python sarmalayıcı fonksiyonu tanımlanır.
    *   Bu sarmalayıcılar, Python'dan gelen argümanları (`PyObject* poArgs`) alır, uygun C++ türlerine dönüştürür (`PyTuple_GetString`, `PyTuple_GetInteger`, `PyTuple_GetFloat` vb.), C++ fonksiyonunu çağırır ve C++ fonksiyonunun dönüş değerini Python nesnesine (`Py_BuildValue`, `Py_BuildNone` vb.) dönüştürerek geri gönderir.
    *   **Örnekler**:
        *   `appCreate(PyObject* poSelf, PyObject* poArgs)`: `CPythonApplication::Instance().Create()` metodunu çağırır.
        *   `appSetCamera(PyObject* poSelf, PyObject* poArgs)`: `CPythonApplication::Instance().SetCamera()` metodunu çağırır.
        *   `appIsPressed(PyObject* poSelf, PyObject* poArgs)`: `CPythonApplication::Instance().IsPressed()` metodunu çağırır.
        *   `appRunPythonFile(PyObject* poSelf, PyObject* poArgs)`: `CPythonLauncher::Instance().RunFile()` metodunu çağırır.
        *   `appGetLocaleServiceName(PyObject* poSelf, PyObject* poArgs)`: `LocaleService_GetName()` fonksiyonunu çağırır.
        *   `appShowWebPage(PyObject* poSelf, PyObject* poArgs)`: `CPythonApplication::Instance().ShowWebPage()` metodunu çağırır.
        *   `appIsExistFile(PyObject* poSelf, PyObject* poArgs)`: `CEterPackManager::Instance().isExist()` veya `_access()` ile dosya varlığını kontrol eder.

*   **`initapp()` Fonksiyonu**:
    *   Bu C fonksiyonu, Python yorumlayıcısı tarafından `app` modülü ilk kez import edildiğinde otomatik olarak çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyonların isimlerini (Python'da kullanılacak isim), karşılık gelen C fonksiyon adreslerini ve argüman türlerini (`METH_VARARGS`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("app", s_methods);` çağrısı ile `app` modülünü oluşturur ve `s_methods` dizisindeki fonksiyonları bu modüle kaydeder.
    *   `PyModule_AddIntConstant(poModule, "PYTHON_CONSTANT_NAME", CPLUSPLUS_CONSTANT_VALUE);` çağrıları ile yüzlerce C++ sabiti ve derleme zamanı yapılandırma bayrağı Python modülüne eklenir. Bu sayede Python scriptleri `app.PYTHON_CONSTANT_NAME` şeklinde bu değerlere erişebilir.
        *   **Klavye Kodları**: `app.DIK_ESCAPE`, `app.VK_LEFT` vb.
        *   **İmleç Şekilleri**: `app.NORMAL`, `app.ATTACK`, `app.BUY` vb.
        *   **Yerelleştirme Kodları**: `app.LOCALE_EN`, `app.LOCALE_TR` vb.
        *   **Sistem Özellikleri (Makrolar)**: `app.ENABLE_COSTUME_SYSTEM`, `app.ENABLE_DRAGON_SOUL_SYSTEM`, `app.WJ_SHOW_MOB_INFO` vb. (bunların değeri `true` veya `false` olur).

*   **Metin Dosyası İşleme (`CTextLineLoader` Sınıfı)**:
    *   `appOpenTextFile`, `appCloseTextFile`, `appGetTextFileLineCount`, `appGetTextFileLine` fonksiyonları, `CTextLineLoader` adlı bir iç sınıfı kullanarak paketlenmiş (`.epk`/`.eix`) veya harici metin dosyalarını satır satır okumak için bir arayüz sağlar.
    *   `CTextLineLoader`, dosyayı `CEterPackManager` aracılığıyla yükler ve `CMemoryTextFileLoader` ile satır verilerine erişir.
    *   **Not**: `CMemoryTextFileLoader` tanımı bu dosyada bulunmamaktadır, muhtemelen `StdAfx.h` veya başka bir başlık dosyasından gelmektedir. Linter hatası da bu eksik tanıma işaret ediyor.

*   **Diğer Yardımcı Fonksiyonlar**:
    *   `appLoadLocaleAddr`: Şifrelenmiş bir adres dosyasını (`addrPath`) TEA algoritmasıyla çözer ve içeriğini döndürür. Bu, genellikle sunucu IP adresleri gibi hassas bilgilerin saklanması için kullanılır.
    *   `appGetImageInfo`: Bir resim dosyasının (DevIL kütüphanesi kullanılarak) yüklenip yüklenemeyeceğini ve boyutlarını döndürür.
    *   `appGetFileList`: Belirli bir filtreye uyan dosyaların listesini Windows API (`FindFirstFile`, `FindNextFile`) kullanarak döndürür.
    *   **Linter Notları**: `ilLoad`, `FindFirstFile` ve `PyString_FromString` (WCHAR* için) çağrılarında karakter türü uyumsuzlukları (char* vs wchar_t*) linter tarafından belirtilmiştir. Bu durumlar, özellikle farklı karakter setleri kullanan yerelleştirmelerde sorunlara yol açabilir veya unicode dosya adlarıyla çalışırken hatalara neden olabilir. `ShellExecute` için de benzer bir durum geçerlidir.

#### Oyunda Kullanım Amacı ve Senaryoları

`PythonApplicationModule.cpp` tarafından oluşturulan `app` modülü, Metin2 istemcisinin Python scriptleri için temel API'yi sağlar. Python, genellikle kullanıcı arayüzü (UI) mantığı, görev (quest) scriptleri ve bazı oyun içi etkileşimler için kullanılır.

*   **UI Kontrolü**: Python UI scriptleri, `app` modülünü kullanarak oyunun temel özelliklerini kontrol eder. Örneğin:
    *   `app.SetCursor(app.TALK)`: Fare imlecini konuşma ikonuna değiştirir.
    *   `app.GetTime()`: Geçerli oyun zamanını alarak UI'da bir zamanlayıcı göstermek.
    *   `app.IsPressed(app.DIK_ESC)`: ESC tuşuna basılıp basılmadığını kontrol etmek.
    *   `app.ShowWebPage("https://google.com", (0,0,800,600))`: Oyun içinde bir web sayfası göstermek.
*   **Yapılandırma ve Ayarlar**: Python scriptleri, `app` modülü aracılığıyla çeşitli oyun ayarlarını okuyabilir veya değiştirebilir.
    *   `app.SetMinFog(1000.0)`: Minimum sis mesafesini ayarlar.
    *   `app.SetFrameSkip(1)`: Frame atlamayı aktif eder.
    *   `app.GetLocaleName()`: Geçerli yerelleştirme adını alır.
*   **Bilgi Alma**: Oyun durumu hakkında bilgi almak için kullanılır.
    *   `app.GetRenderFPS()`: Anlık render FPS'ini alır.
    *   `app.GetCursorPosition()`: Fare imlecinin ekran koordinatlarını alır.
    *   `app.IsExistFile("locale/tr/locale_interface.txt")`: Belirli bir dosyanın varlığını kontrol eder.
*   **Özellik Kontrolü**: Python scriptleri, `app.ENABLE_COSTUME_SYSTEM` gibi sabitleri kontrol ederek belirli bir özelliğin istemcide aktif olup olmadığını anlayabilir ve buna göre farklı UI elemanları gösterebilir veya farklı davranışlar sergileyebilir.
*   **Geliştirme ve Hata Ayıklama**: `app.EnablePerformanceTime("RENDER_GAME", 1)` gibi fonksiyonlar geliştirme sırasında performans takibi için kullanılabilir. `app.testSetSpecularColor()` gibi test fonksiyonları da mevcuttur.
*   **Görev Scriptleri**: Görev scriptleri, belirli eylemleri tetiklemek veya bilgi almak için `app` modülünü kullanabilir (örn: `app.GetGlobalTime()` ile sunucu zamanını almak).

Kısacası, `app` modülü, C++ çekirdeği ile Python scriptleri arasında hayati bir köprü oluşturarak istemcinin esnekliğini ve modülerliğini artırır. Python'ın dinamik doğası, UI ve bazı oyun mantıklarının daha hızlı geliştirilmesine ve güncellenmesine olanak tanır.
Linter tarafından belirtilen `CMemoryTextFileLoader`'ın eksik tanımı ve karakter türü uyumsuzlukları, bu dosyanın derlenmesi ve doğru çalışması için çözülmesi gereken potansiyel sorunlardır.

---

### `PythonApplicationProcedure.cpp` (Pencere Mesaj İşleme)

Bu dosya, `CPythonApplication` sınıfının Windows işletim sisteminden gelen mesajları işleyen ana fonksiyonu olan `WindowProcedure` ve ilgili yardımcı fonksiyonları içerir. İstemcinin işletim sistemi olaylarına (fare, klavye, pencere yönetimi vb.) tepki vermesini sağlar.

#### Dosyanın Genel Amaçları

*   **Windows Mesaj Döngüsü İşleme**: `WindowProcedure` fonksiyonu, uygulamanın ana penceresine gönderilen Windows mesajlarını (örneğin, `WM_ACTIVATEAPP`, `WM_KEYDOWN`, `WM_LBUTTONDOWN`, `WM_MOUSEMOVE`, `WM_SIZE`, `WM_CLOSE` vb.) yakalar ve işler.
*   **Giriş Yönetimi**: Klavye ve fare girişlerini (tuş basımları, fare tıklamaları, fare hareketleri, fare tekerleği) işleyerek uygun oyun içi olaylara (`OnKeyDown`, `OnMouseLeftButtonDown`, `OnMouseWheel` vb.) yönlendirir.
*   **IME (Input Method Editor) Yönetimi**: Özellikle Asya dilleri gibi karmaşık karakter giriş yöntemleri için IME mesajlarını (`WM_IME_STARTCOMPOSITION`, `WM_IME_CHAR` vb.) işler ve `CPythonIME` singleton sınıfına iletir.
*   **Pencere Durumu Yönetimi**: Uygulamanın aktif/pasif olma durumunu, tam ekran/pencere modu geçişlerini, pencerenin yeniden boyutlandırılmasını ve küçültülmesini yönetir.
    *   `__SetFullScreenWindow`: Tam ekran moduna geçer.
    *   `__MinimizeFullScreenWindow`: Tam ekran modundayken pencereyi küçültür ve masaüstüne döner.
    *   `__ResetCameraWhenMinimize`: Pencere küçültüldüğünde kamera ve imleç durumunu sıfırlar.
*   **Fare Yakalama (Mouse Capture) Yönetimi**: `SafeSetCapture` ve `SafeReleaseCapture` fonksiyonları ile fare girişinin uygulama penceresi dışına taşmasını engellemek veya serbest bırakmak için `SetCapture` ve `ReleaseCapture` Windows API'lerini güvenli bir şekilde (referans sayacıyla) kullanır.
*   **Çift Tıklama Algılama**: Sol fare tuşuna yapılan çift tıklamaları algılar ve `OnMouseLeftButtonDoubleClick` olayını tetikler.

#### C++ Implementasyon Detayları

*   **`WindowProcedure(HWND hWnd, UINT uiMsg, WPARAM wParam, LPARAM lParam)`**:
    *   Gelen `uiMsg` (mesaj tipi) değerine göre `switch` ifadesiyle farklı durumları ele alır.
    *   **`WM_ACTIVATEAPP`**: Uygulama penceresi aktif olduğunda ses seviyesini geri yükler ve tam ekran modunu ayarlar; pasif olduğunda ses seviyesini kaydeder ve tam ekran modundan çıkar (gerekirse).
    *   **IME Mesajları**: `WM_INPUTLANGCHANGE`, `WM_IME_STARTCOMPOSITION`, `WM_IME_COMPOSITION`, `WM_IME_ENDCOMPOSITION`, `WM_IME_NOTIFY`, `WM_IME_SETCONTEXT`, `WM_CHAR` gibi mesajları doğrudan `CPythonIME::Instance()` üzerinden ilgili işleyici fonksiyonlara yönlendirir.
    *   **Klavye Mesajları**: `WM_KEYDOWN` için `OnIMEKeyDown` çağrılır. `WM_SYSKEYDOWN` ve `WM_SYSKEYUP` (örneğin Alt+F4, F10) kısmen işlenir.
    *   **Fare Mesajları**:
        *   `WM_LBUTTONDOWN`, `WM_LBUTTONUP`, `WM_RBUTTONDOWN`, `WM_RBUTTONUP`, `WM_MBUTTONDOWN`, `WM_MBUTTONUP`: İlgili `OnMouse*ButtonDown/Up` fonksiyonlarını çağırır. Fare yakalamayı yönetir. Sol tuş için çift tıklama mantığı içerir.
        *   Fare orta tuş işlemleri `UI::CWindowManager::Instance().RunMouseMiddleButtonDown/Up` fonksiyonlarına yönlendirilir.
        *   `WM_MOUSEWHEEL` (0x20a): Eğer CEF web tarayıcı görünür değilse `OnMouseWheel` çağrılır.
    *   **`WM_SIZE`**: Pencere boyutu değiştiğinde (restore, maximize, minimize) grafik cihazının (`m_grpDevice`) arka tamponunu yeniden boyutlandırır ve `OnSizeChange` olayını tetikler.
    *   **`WM_EXITSIZEMOVE`**: Pencere taşıma veya boyutlandırma işlemi bittiğinde çağrılır, `WM_SIZE` benzeri işlemler yapar.
    *   **`WM_SETCURSOR`**: Fare imlecinin görünürlüğünü ve şeklini donanım imleç modu aktifse ayarlar.
    *   **`WM_CLOSE`**: Debug modda doğrudan `PostQuitMessage(0)` ile uygulamayı kapatır, release modda ise `RunPressExitKey()` fonksiyonunu çağırarak oyun içinden çıkış prosedürünü başlatır.
    *   **`WM_DESTROY`**: Pencere yok edildiğinde çağrılır, şu an için özel bir işlem yapmıyor.
    *   Diğer tüm mesajlar için varsayılan olarak `CMSApplication::WindowProcedure` (muhtemelen üst sınıfın mesaj işleyicisi) çağrılır.
*   **Fare Yakalama**: `gs_nMouseCaptureRef` statik bir sayaç kullanarak `SetCapture` ve `ReleaseCapture` çağrılarının dengeli yapılmasını sağlar. Bu, özellikle iç içe kontroller veya birden fazla bileşenin fareyi yakalamak istediği durumlarda önemlidir.
*   **Tam Ekran Yönetimi**: Windows API fonksiyonları olan `ChangeDisplaySettings` ve `SetWindowPos` kullanılarak ekran çözünürlüğü ve pencere durumu ayarlanır.

#### Oyunda Kullanım Amacı ve Senaryoları

`PythonApplicationProcedure.cpp` içindeki kodlar, istemcinin işletim sistemi ve kullanıcı ile temel düzeyde etkileşim kurmasını sağlar.

*   **Kullanıcı Girişi**: Oyuncunun klavye ve fare ile yaptığı tüm eylemler (hareket, saldırı, arayüz etkileşimi, yazı yazma) bu prosedür üzerinden geçer.
*   **Pencere Yönetimi**: Oyunun tam ekran veya pencere modunda çalışması, boyutunun değiştirilmesi, simge durumuna küçültülmesi gibi temel pencere işlemleri buradan yönetilir.
*   **Uygulama Yaşam Döngüsü**: Oyunun başlatılması, aktifleşmesi, pasifleşmesi ve kapatılması gibi durumlar `WM_ACTIVATEAPP` ve `WM_CLOSE` mesajları ile kontrol edilir.
*   **IME Desteği**: Farklı dillerde metin girişi için IME sisteminin doğru çalışmasını temin eder.
*   **Grafik ve Ses Senkronizasyonu**: Pencere durumu değiştiğinde (aktif/pasif, minimize) grafik ayarlarının (kamera, imleç) ve ses düzeylerinin uygun şekilde ayarlanmasını sağlar.

Bu dosya, `CPythonApplication` sınıfının Windows API'leriyle doğrudan etkileşimde bulunduğu kritik bir katmandır ve oyunun genel kullanıcı deneyimi için temel teşkil eder.

---

### `PythonApplicationWebPage.cpp` (Web Sayfası Yönetimi)

Bu dosya, `CPythonApplication` sınıfının Chromium Embedded Framework (CEF) kullanarak oyun içinde web sayfalarını gösterme, gizleme ve yönetme işlevlerini içerir. Bu, genellikle oyun içi duyurular, etkinlikler, mağaza veya destek sayfaları gibi web tabanlı içeriklerin sunulması için kullanılır.

#### Dosyanın Genel Amaçları

*   **Web Sayfası Durum Kontrolü**: Oyun içinde bir web sayfasının o anda aktif ve görünür olup olmadığını kontrol eder.
*   **Web Sayfası Gösterme**: Belirli bir URL'yi oyun penceresi içinde tanımlanmış bir dikdörtgen alanda bir web tarayıcı görünümü olarak yükler ve gösterir.
*   **Web Sayfası Taşıma/Yeniden Boyutlandırma**: Halihazırda açık olan bir web sayfasının oyun içindeki konumunu ve boyutlarını dinamik olarak değiştirir.
*   **Web Sayfası Gizleme**: Aktif olan web sayfasını gizler ve tarayıcı kaynaklarını serbest bırakır.

#### C++ Implementasyon Detayları

*   **`bool CPythonApplication::IsWebPageMode()`**:
    *   `CefWebBrowser_IsVisible()` global fonksiyonunu çağırarak CEF tarayıcısının görünür olup olmadığını kontrol eder.
    *   Görünürse `true`, değilse `false` döner.
*   **`void CPythonApplication::ShowWebPage(const char* c_szURL, const RECT& c_rcWebPage)`**:
    *   Eğer zaten bir web sayfası görünürse (`CefWebBrowser_IsVisible()`) işlem yapmadan çıkar.
    *   Grafik cihazını (`m_grpDevice`) web tarayıcı moduna geçirir (`EnableWebBrowserMode`), bu muhtemelen render döngüsünde veya giriş yönetiminde özel bir durum oluşturur.
    *   `CefWebBrowser_Show(GetWindowHandle(), c_szURL, &c_rcWebPage)` fonksiyonunu çağırarak CEF tarayıcısını belirtilen URL ve konum/boyut ile ana oyun penceresi (`GetWindowHandle()`) üzerinde oluşturur ve gösterir.
    *   `CefWebBrowser_Show` başarısız olursa `TraceError` ile hata kaydı düşer.
    *   Fare imleç modunu donanım moduna (`CURSOR_MODE_HARDWARE`) ayarlar, bu genellikle web sayfalarıyla etkileşim için tercih edilir.
*   **`void CPythonApplication::MoveWebPage(const RECT& c_rcWebPage)`**:
    *   Eğer bir web sayfası görünürse (`CefWebBrowser_IsVisible()`):
        *   Grafik cihazının (`m_grpDevice`) web tarayıcı için ayırdığı dikdörtgen alanı günceller (`MoveWebBrowserRect`).
        *   `CefWebBrowser_Move(&c_rcWebPage)` fonksiyonunu çağırarak CEF tarayıcı penceresinin pozisyonunu ve boyutunu günceller.
*   **`void CPythonApplication::HideWebPage()`**:
    *   Eğer bir web sayfası görünürse (`CefWebBrowser_IsVisible()`):
        *   `CefWebBrowser_Hide()` fonksiyonunu çağırarak CEF tarayıcısını gizler.
        *   Grafik cihazını (`m_grpDevice`) web tarayıcı modundan çıkarır (`DisableWebBrowserMode`).
        *   Sistem ayarlarına (`m_pySystem.IsSoftwareCursor()`) göre fare imleç modunu yazılım (`CURSOR_MODE_SOFTWARE`) veya donanım (`CURSOR_MODE_HARDWARE`) moduna geri döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu fonksiyonlar, oyun geliştiricilerinin istemci içinden dinamik web içerikleri sunmasını sağlar.

*   **Oyun İçi Mağaza (Item Shop)**: Oyuncuların gerçek para veya oyun içi para birimiyle eşya satın alabileceği web tabanlı bir mağaza arayüzü göstermek.
*   **Duyurular ve Etkinlik Bilgileri**: Yeni güncellemeler, bakım duyuruları veya özel etkinlikler hakkında oyuncuları bilgilendirmek için web sayfaları kullanmak.
*   **Lider Tabloları (Rankings)**: Oyun içi sıralamaları veya turnuva sonuçlarını web formatında göstermek.
*   **Destek ve Yardım Sayfaları**: Oyuncuların sıkça sorulan sorulara, oyun kılavuzlarına veya destek talebi oluşturma formlarına erişebileceği sayfalar sunmak.
*   **Anketler ve Geri Bildirim Formları**: Oyuncuların oyun hakkındaki düşüncelerini toplamak için web tabanlı anketler göstermek.

`CPythonApplication` içindeki bu metotlar, `CefWebBrowser` modülü (muhtemelen CEF API'lerini sarmalayan bir katman) ile etkileşime girerek bu işlevselliği soyutlar ve istemcinin diğer bölümlerinin kolayca web sayfalarını yönetebilmesini sağlar. Bu özellik, oyun arayüzünün bir parçası olarak modern ve esnek web teknolojilerinden faydalanma imkanı sunar.

---

### `PythonBackground.h`, `PythonBackground.cpp` ve `PythonBackgroundModule.cpp` (Oyun Arka Plan Yönetimi)

Bu üç dosya, Metin2 istemcisinin oyun dünyasının arka planını, haritalarını (özellikle dış mekan haritaları - `CMapOutdoor`), gökyüzünü, sisini, gölgelerini, ışıklandırmasını, kar gibi çevresel efektlerini ve bunlarla ilgili Python (`background` modülü) arayüzünü yöneten `CPythonBackground` singleton sınıfını tanımlar, implemente eder ve Python'a sunar.

#### Dosyaların Genel Amaçları

*   **`PythonBackground.h` (Başlık Dosyası)**:
    *   `CPythonBackground` singleton sınıfının tanımını içerir. Bu sınıf, `CMapManager` (temel harita yönetimi) sınıfından miras alır.
    *   Oyun dünyasının görsel ve mantıksal yönleriyle ilgili çeşitli `enum` tanımları (örn: `EShadow` gölge seviyeleri, `EDayMode` gündüz/gece modu) ve yapılar (örn: `TVIEWDISTANCESET` görüş mesafesi ve sis ayarları) barındırır.
    *   Harita yükleme, koordinat dönüşümleri, gölge ayarları, görüş mesafesi yönetimi, ışıklandırma, çevresel efektler (kar, Noel ağacı), portal yönetimi, özel efektler, lonca alanları ve harita geçişleri (warp) gibi çok sayıda fonksiyon prototipini deklare eder.
    *   `CSnowEnvironment` (kar efektleri yönetimi) nesnesini ve harita ile ilgili çeşitli veri yapılarını (mevcut harita adı, taban koordinatları, portal ID'leri, hedef efektleri vb.) üye değişken olarak tutar.
    *   İsteğe bağlı derleme makroları (`ENABLE_SHADOW_RENDER_QUALITY_OPTION`, `ENABLE_OX_RENDER_AREA`, `ENABLE_ENVIRONMENT_EFFECT_OPTION`, `ENABLE_SUNGMA_ATTR`) ile kontrol edilen ek özellikler için tanımlamalar içerir.

*   **`PythonBackground.cpp` (C++ Implementasyon Dosyası)**:
    *   `CPythonBackground` sınıfının tüm metotlarının detaylı C++ implementasyonlarını içerir.
    *   Harita verilerinin (`CMapOutdoor` aracılığıyla), gölgelerin, sisin, gökyüzünün, suyun, bulutların ve diğer çevresel elemanların yüklenmesi, güncellenmesi ve render edilmesiyle ilgili mantığı yönetir.
    *   `Initialize()`: Temel ayarları yapar, `AtlasInfo.txt` dosyasının yolunu belirler.
    *   `Create()`: Harita ve kar ortamını oluşturur, `property` dosyalarını yükler (`__CreateProperty`).
    *   `Destroy()`: Harita ve kar ortamı kaynaklarını serbest bırakır.
    *   `Update(float fCenterX, float fCenterY, float fCenterZ)`: Kamera pozisyonuna göre haritayı, çevresel sesleri, kar efektlerini, portalları ve hedef efektlerini günceller.
    *   `Render()`: Ana harita render işlemini (`CMapOutdoor::Render()`) ve görünürse lonca alanlarını çizer. Kar efektleri için `m_SnowEnvironment.Deform()` çağırır.
    *   Çeşitli `Render*` fonksiyonları (`RenderSnow`, `RenderSky`, `RenderCloud`, `RenderWater`, `RenderEffect`, `RenderCharacterShadowToTexture` vb.) `CMapOutdoor` sınıfının ilgili render metotlarını çağırır.
    *   `SetShadowLevel()`, `SetShadowQualityLevel()`, `SetShadowTargetLevel()`: Gölge ayarlarını ve kalitesini `CMapOutdoor` üzerinden yapılandırır.
    *   `SelectViewDistanceNum()`, `SetViewDistanceSet()`: Görüş mesafesi ve sis ayarlarını yönetir.
    *   `Warp(DWORD dwX, DWORD dwY)`: Belirtilen global koordinatlara göre yeni haritayı yükler, mini haritayı günceller, zindan haritaları için özel ayarlar yapar.
    *   `CheckAdvancing(CInstanceBase* pInstance)`: Verilen karakterin ilerleyip ilerleyemeyeceğini, çevredeki nesnelerle çarpışma durumunu kontrol eder.
    *   Portal, lonca alanı, özel efekt ve hedef efekt yönetimi fonksiyonları.
    *   `ENABLE_ENVIRONMENT_EFFECT_OPTION` için `IsBoomMap()` gibi haritaya özel çevre efektlerini kontrol eden fonksiyonlar.

*   **`PythonBackgroundModule.cpp` (Python Arayüz Modülü)**:
    *   `CPythonBackground` sınıfının işlevlerini Python scriptlerinin kullanabilmesi için "background" adında bir Python modülü oluşturur.
    *   `initBackground()` fonksiyonu, CPython API'sini kullanarak bu modülü tanımlar ve `CPythonBackground` metotlarını sarmalayan C fonksiyonlarını (`background*` önekli) bu modüle kaydeder.
    *   Python'dan harita yükleme (`backgroundLoadMap`), çevre verisi ayarlama (`backgroundSetEnvironmentData`), gölge seviyesi (`backgroundSetShadowLevel`), görüş mesafesi (`backgroundSelectViewDistanceNum`), kar efekti (`backgroundEnableSnow`), zemin yüksekliği sorgulama (`backgroundGetHeight`) gibi birçok özelliğin kontrol edilmesini sağlar.
    *   Ayrıca, `CPythonBackground` ve `CMapOutdoor` içindeki çeşitli sabitleri (`PART_SKY`, `SHADOW_NONE`, `DISTANCE0` vb.) Python modülüne tamsayı sabitleri olarak ekler, böylece Python scriptleri bu sabitlere isimleriyle erişebilir.

#### Başlık Dosyası Tanımları (`PythonBackground.h`)

*   **Temel Sınıflar ve Miras**:
    *   `CMapManager`: Temel harita yönetimi işlevlerini sağlayan üst sınıf.
    *   `CSingleton<CPythonBackground>`: Singleton deseni ile global erişim sağlar.
*   **Önemli Enum ve Yapılar**:
    *   `EShadow`: Farklı gölge detay seviyeleri (SHADOW_NONE, SHADOW_GROUND, SHADOW_ALL vb.).
    *   `EShadowQuality` ve `EShadowTarget` (eğer `ENABLE_SHADOW_RENDER_QUALITY_OPTION` aktifse): Daha detaylı gölge kalite ve hedef ayarları.
    *   `OXArea` (eğer `ENABLE_OX_RENDER_AREA` aktifse): OX Event alanı tanımları.
    *   `DISTANCE0` - `DISTANCE4`: Farklı görüş mesafesi kademeleri.
    *   `DAY_MODE_LIGHT`, `DAY_MODE_DARK`: Gündüz ve gece modları.
    *   `TVIEWDISTANCESET`: Görüş mesafesi, sis başlangıç/bitiş mesafesi ve gökyüzü kutusu ölçeğini içeren yapı.
*   **Üye Değişkenler**:
    *   `m_SnowEnvironment`: Kar efektlerini yöneten `CSnowEnvironment` nesnesi.
    *   `m_iDayMode`: Aktif gündüz/gece modu.
    *   `m_eShadowLevel`, `m_eShadowQualityLevel`, `m_eShadowTargetLevel`: Mevcut gölge ayarları.
    *   `m_eViewDistanceNum`: Mevcut görüş mesafesi kademesi.
    *   `m_dwRenderShadowTime`: Gölge render süresini tutan değişken.
    *   `m_dwBaseX`, `m_dwBaseY`: Yüklenen haritanın başlangıç (base) global koordinatları.
    *   `m_ViewDistanceSet[NUM_DISTANCE_SET]`: Farklı kademeler için görüş mesafesi ayar dizisi.
    *   `m_kSet_iShowingPortalID`: O anda görünür olan portal ID'leri kümesi.
    *   `m_kSet_strDungeonMapName`: Zindan olarak işaretlenmiş harita isimleri kümesi.
    *   `m_kMap_dwTargetID_dwChrID`: Hedef efekt ID'si ile karakter VID'sini eşleyen harita.
    *   `m_kMap_dwID_kReserveTargetEffect`: Yükseklik verisi henüz alınamadığı için bekleyen hedef efektleri.
*   **Temel Fonksiyon Prototipleri**:
    *   `Create() / Destroy()`: Kaynakları oluşturma ve yok etme.
    *   `Update()`: Arka plan mantığını güncelleme.
    *   `Render()`: Arka planı çizme.
    *   `LoadMap()` (CMapManager'dan gelen): Harita yükleme.
    *   `Warp()`: Farklı bir haritaya veya konuma ışınlanma.
    *   Çeşitli ayar (`Set*`), sorgulama (`Get*`), render (`Render*`) ve özel efekt (`CreateTargetEffect`, `DeleteSpecialEffect`) fonksiyonları.

#### C++ Implementasyon Detayları (`PythonBackground.cpp`)

*   **Property Yükleme (`__CreateProperty`)**: Oyunun "property" klasöründen (`.prb` dosyaları - bina, ağaç gibi statik nesne tanımları) nesne özelliklerini yükler.
*   **Harita Yükleme ve Yönetimi**:
    *   `LoadMap` çağrıldığında, `CMapManager`'ın ilgili fonksiyonu kullanılır. Başarılı yükleme sonrası mini harita atlası (`CPythonMiniMap::Instance().LoadAtlas()`) güncellenir.
    *   `Warp` fonksiyonu, hedef global koordinatlara göre doğru `TMapInfo`'yu bulur, `LoadMap` ile yeni haritayı yükler, taban koordinatlarını (`m_dwBaseX`, `m_dwBaseY`) günceller ve mevcut harita adını (`m_strMapName`) ayarlar.
    *   Zindan haritaları (`m_kSet_strDungeonMapName` içinde ise) için özel ayarlar (`EnableTerrainOnlyForHeight`, `EnablePortal`) yapılır.
*   **Güncelleme Döngüsü (`Update`)**:
    *   `CMapManager::UpdateMap()` ve `CMapManager::UpdateAroundAmbience()` çağrılarak harita verileri ve çevresel sesler güncellenir.
    *   `m_SnowEnvironment.Update()` ile kar efektleri güncellenir.
    *   Portallar kontrol edilir: `CCullingManager::ForInRay` ile kameranın baktığı yöndeki portallar tespit edilir. Eğer portal listesi değişmişse, `ClearPortal()`, `AddShowingPortalID()` ve `RefreshPortal()` çağrılarak güncellenir.
    *   Hedef işaretleme efektleri (`m_kMap_dwTargetID_dwChrID`, `m_kMap_dwID_kReserveTargetEffect`) güncellenir. Eğer bir hedef efekti için yükseklik verisi hemen alınamazsa (`rkMap.GetHeight` 0 dönerse), `SReserveTargetEffect` olarak beklemeye alınır.
*   **Render İşlemleri**:
    *   `Render()` ana fonksiyonu, `CMapOutdoor::Render()`'ı çağırarak arazi, nesneler, su vb. çizer. Ayrıca aktifse lonca alanlarını çizer. Kar efektleri için `m_SnowEnvironment.Deform()` çağırır.
    *   `RenderCharacterShadowToTexture()`: Karakterlerin gölgelerini ayrı bir dokuya (texture) çizer. Bu doku daha sonra arazi üzerine yansıtılır. Gölge seviyesine ve `ENABLE_SHADOW_RENDER_QUALITY_OPTION` makrosuna göre farklı karakter grupları (sadece ana karakter veya tüm karakterler) ve farklı gölge doku boyutları kullanılır.
*   **Çarpışma Kontrolü (`CheckAdvancing`)**:
    *   Bir karakterin hareket etmeden önce, `CCullingManager` kullanılarak çevresindeki nesnelerle çarpışıp çarpışmadığı kontrol edilir.
    *   `CollisionAdjustChecker` ile nesnelerden kaçınma (avoidance) denenir. Eğer hala çarpışma varsa (`CollisionChecker`), karakterin hareketi engellenir (`BlockMovement()`).
*   **Işıklandırma**: `SetCharacterDirLight` ve `SetBackgroundDirLight` fonksiyonları, mevcut çevre verisinden (`mc_pcurEnvironmentData`) ilgili yönlü ışık bilgilerini alıp `STATEMANAGER` aracılığıyla Direct3D'ye ayarlar.

#### Python Arayüzü Implementasyon Detayları (`PythonBackgroundModule.cpp`)

*   **Modül Tanımlama (`initBackground`)**:
    *   `Py_InitModule("background", s_methods)` ile "background" adlı Python modülü oluşturulur.
    *   `s_methods` dizisi, Python'dan çağrılacak fonksiyon adlarını (örn: "LoadMap", "SetShadowLevel") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn: `backgroundLoadMap`, `backgroundSetShadowLevel`) eşler.
*   **Fonksiyon Sarmalayıcıları (`background*` önekli fonksiyonlar)**:
    *   Bu C fonksiyonları, Python'dan argümanları alır (`PyTuple_GetString`, `PyTuple_GetInteger`, `PyTuple_GetFloat` vb.), `CPythonBackground::Instance()` üzerinden ilgili C++ metodunu çağırır ve sonucu Python nesnesine (`Py_BuildValue`, `Py_BuildNone` vb.) dönüştürerek geri gönderir.
    *   **Örnekler**:
        *   `backgroundLoadMap(PyObject* poSelf, PyObject* poArgs)`: Harita adı ve başlangıç koordinatlarını Python'dan alır, `CPythonBackground::Instance().LoadMap()` çağırır.
        *   `backgroundSetEnvironmentData(PyObject* poSelf, PyObject* poArgs)`: Çevre verisi index'ini alır, `CPythonBackground::Instance().GetEnvironmentData()` ve `ResetEnvironmentDataPtr()` ile ayarlar. `ENABLE_ENVIRONMENT_EFFECT_OPTION` aktifse ve gece modu seçeneği kapalıysa gece modunu gündüze çevirir.
        *   `backgroundGetHeight(PyObject* poSelf, PyObject* poArgs)`: Koordinatları alır, `CPythonBackground::Instance().GetHeight()` ile yükseklik bilgisini alır ve Python'a float olarak döndürür.
        *   `backgroundSetShadowLevel(PyObject* poSelf, PyObject* poArgs)`: Gölge seviyesini alır ve `CPythonBackground::Instance().SetShadowLevel()` ile ayarlar.
*   **Sabitlerin Eklenmesi**:
    *   `PyModule_AddIntConstant(poModule, "PYTHON_CONSTANT_NAME", CPP_CONSTANT_VALUE)` kullanılarak C++ tarafındaki `enum` değerleri (örn: `CPythonBackground::SHADOW_ALL`, `CMapOutdoor::PART_SKY`) Python modülüne sabit olarak eklenir. Bu, Python scriptlerinin bu sabitleri `background.SHADOW_ALL` gibi kullanmasını sağlar.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonBackground` ve onun Python modülü `background`, oyun dünyasının atmosferini ve görsel tutarlılığını sağlamanın yanı sıra, performans ve oyun mekanikleriyle de yakından ilişkilidir. Örneğin, görüş mesafesi ayarları doğrudan render edilecek nesne sayısını ve dolayısıyla performansı etkiler. Zemin yükseklik verileri ise karakter hareketleri ve yeteneklerin doğru çalışması için kritik öneme sahiptir.

---

### `PythonCharacterManager.h` ve `PythonCharacterManager.cpp` (Karakter Yönetimi)

Bu dosyalar, `CPythonCharacterManager` adlı singleton sınıfını tanımlar ve implemente eder. Bu sınıf, oyundaki tüm karakter örneklerinin (`CInstanceBase` türevleri) yaşam döngüsünü, güncellemelerini, render edilmelerini ve birbirleriyle olan temel etkileşimlerini yönetmekten sorumludur. Oyuncu karakteri, diğer oyuncular, NPC'ler ve canavarlar gibi tüm dinamik varlıklar bu yönetici tarafından takip edilir.

#### Dosyaların Genel Amaçları

*   **Karakter Örneklerinin Merkezi Yönetimi**: Oyunda mevcut olan tüm karakterlerin (`CInstanceBase` nesneleri) bir kaydını tutar. Bu, karakterlerin oluşturulması, silinmesi ve erişilmesi için merkezi bir nokta sağlar.
*   **Yaşam Döngüsü Yönetimi**: Karakterlerin oluşturulmasından (`CreateInstance`), güncellenmesine (`Update`), deformasyonuna (`Deform`), render edilmesine (`Render`) ve silinmesine (`DeleteInstance`, `DeleteInstanceByFade`) kadar tüm yaşam döngüsü olaylarını yönetir.
*   **Koleksiyonlar**: Karakterleri "canlı" (`m_kAliveInstMap`) ve "ölü" (silinme animasyonu devam edenler, `m_kDeadInstList`) olarak iki ana koleksiyonda saklar.
*   **Ana Karakter Takibi**: Oyuncunun kontrol ettiği ana karakteri (`m_pkInstMain`) özel olarak takip eder.
*   **Hedef Seçimi (Picking)**: Fare ile üzerine gelinen veya seçilen karakteri (`m_pkInstPick`) belirler.
*   **Render Optimizasyonu**: Karakterleri render sırasına göre sıralar ve sadece görüş alanı içindekileri (veya belirli bir mesafedekileri) işler.
*   **Çarpışma ve Etkileşim**: Temel çarpışma ayarlarını (`AdjustCollisionWithOtherObjects`) ve PvP/Guild savaşı durumlarına göre karakterlerin görünüm güncellemelerini tetikler (`InsertPVPKey`, `ChangeGVG`).
*   **Özel Efektler**: Karakterlerle ilişkili seviye atlama gibi nokta efektlerini (`ShowPointEffect`, `RegisterPointEffect`) yönetir.

#### Başlık Dosyası Tanımları (`PythonCharacterManager.h`)

*   **Temel Sınıflar ve Miras**:
    *   `CSingleton<CPythonCharacterManager>`: Singleton deseni ile global erişim sağlar.
    *   `IAbstractCharacterManager`: Karakter yöneticisi için soyut bir arayüz implemente eder.
    *   `IObjectManager`: (Muhtemelen genel nesne yönetimi için bir arayüz, detayları `IObjectManager` tanımında gizlidir).
*   **Veri Yapıları**:
    *   `TCharacterInstanceList`: `CInstanceBase*` tipinde bir `std::list` (genellikle ölü karakterler için kullanılır).
    *   `TCharacterInstanceMap`: `DWORD` (Virtual ID) anahtarlı ve `CInstanceBase*` değerli bir `std::map` (canlı karakterler için kullanılır).
    *   `CharacterIterator`: `TCharacterInstanceMap` üzerinde gezinmek için özel bir iterator sınıfı.
*   **Önemli Üye Değişkenler**:
    *   `m_pkInstMain`: Ana oyuncu karakterinin işaretçisi.
    *   `m_pkInstPick`: Fare ile seçilen karakterin işaretçisi.
    *   `m_pkInstBind`: (Muhtemelen bir karaktere bağlanmış başka bir karakteri veya nesneyi tutar, kullanım senaryosu `SelectInstance` ile ilgilidir).
    *   `m_v2PickedInstProjPos`: Seçilen karakterin 2D ekran projeksiyon koordinatları.
    *   `m_kAliveInstMap`: Canlı karakter örneklerini VID'leri ile eşleyerek tutan map.
    *   `m_kDeadInstList`: Silinme sürecindeki karakter örneklerini tutan liste.
    *   `m_kVct_pkInstPicked`: Fare ile etkileşime girilebilecek (pickable) karakterlerin sıralı listesi.
    *   `m_adwPointEffect[POINT_MAX_NUM]`: `POINT_LEVEL`, `POINT_LEVEL_STEP` gibi olaylar için kayıtlı efekt ID'lerini tutan dizi.
    *   `m_adwVectorIndexTabNextTarget` (eğer `ENABLE_TAB_NEXT_TARGET` aktifse): Sekme tuşu ile sonraki hedefi seçme özelliği için mevcut hedef indeksini tutar.
*   **Temel Fonksiyon Prototipleri**:
    *   `Update()`, `Deform()`, `Render()`: Karakterlerin ana güncelleme ve çizim döngüleri.
    *   `CreateInstance()`, `DeleteInstance()`: Karakter örneklerini oluşturma ve silme.
    *   `SetMainInstance()`, `GetMainInstancePtr()`: Ana karakteri ayarlama ve alma.
    *   `GetInstancePtr()`: Verilen VID ile karakter örneğini alma.
    *   `OLD_GetPickedInstancePtr()`: Fare ile seçilen karakteri alma.
    *   `RenderShadowMainInstance()`, `RenderShadowAllInstances()`: Karakter gölgelerini çizme.
    *   `RefreshAllPCTextTail()`, `RefreshAllGuildMark()`: Tüm oyuncu karakterlerinin isim etiketlerini ve lonca işaretlerini yenileme.

#### C++ Implementasyon Detayları (`PythonCharacterManager.cpp`)

*   **Karakter Güncelleme (`Update`)**:
    *   Tüm canlı karakterler için `CInstanceBase::Update()` çağrılır.
    *   Ana karakter varsa, belirli bir görüş mesafesi (`CHAR_STAGE_VIEW_BOUND`) dışındaki karakterler `__DeleteBlendOutInstance` ile silinmek üzere işaretlenir ve `m_kDeadInstList`'e taşınır.
    *   `UpdateTransform()`: Karakterlerin pozisyon ve duruş güncellemelerini yapar, `CPythonBackground` ile çarpışma kontrolü (`CheckAdvancing`) yapar.
    *   `UpdateDeleting()`: `m_kDeadInstList` içindeki karakterlerin silinme animasyonlarını günceller; animasyon bittiyse karakteri tamamen siler (`CInstanceBase::Delete`).
    *   `__NEW_Pick()`: Fare ile etkileşim kurulabilecek karakterleri günceller ve seçileni belirler.
*   **Karakter Render (`Render`)**:
    *   Direct3D durumlarını ayarlar.
    *   `__RenderSortedAliveActorList()`: Canlı karakterleri `LessCharacterInstancePtrRenderOrder` (render sırasına göre) sıralar ve her biri için `Render()` ve `RenderTrace()` çağırır.
    *   `__RenderSortedDeadActorList()`: Benzer şekilde, ölü karakterleri sıralar ve render eder.
    *   Seçili karakter varsa, 3D pozisyonunu 2D ekran koordinatlarına yansıtır (`m_v2PickedInstProjPos`).
*   **Karakter Oluşturma ve Silme**:
    *   `CreateInstance(SCreateData&)`: Yeni bir `CInstanceBase` nesnesi oluşturur, `RegisterInstance` ile `m_kAliveInstMap`'e ekler ve `CInstanceBase::Create()` ile karakter verilerini (ırk, görünüm vb.) yükler. Ana karakter ise `SelectInstance` ile seçilir.
    *   `DeleteInstance(DWORD dwDelVID)`: Karakteri doğrudan `m_kAliveInstMap`'ten siler ve hafızadan temizler. `m_pkInstMain`, `m_pkInstPick`, `m_pkInstBind` işaretçilerini günceller.
    *   `__DeleteBlendOutInstance(CInstanceBase* pkInstDel)`: Karakteri solma (fade out) efektiyle silmek için `pkInstDel->DeleteBlendOut()` çağırır ve `m_kDeadInstList`'e ekler. `IAbstractPlayer::NotifyCharacterDead` ile oyuncu arayüzüne ölüm bildirimi yapar.
*   **Hedef Seçimi (Picking) Mekanizması**:
    *   `__UpdatePickedActorList()`: Etkileşime girilebilen (`CanPickInstance()`) ve görünür olan tüm karakterleri `m_kVct_pkInstPicked` listesine ekler. Ölü karakterler için sınırlayıcı kutu (`IntersectBoundingBox`), canlılar için savunma küresi (`IntersectDefendingSphere`) ile kesişim testi yapar.
    *   `__SortPickedActorList()`: `m_kVct_pkInstPicked` listesini kameraya olan mesafeye göre (`CInstanceBase_SLessCameraDistance`) sıralar.
    *   `__NEW_Pick()`: Sıralanmış listedeki karakterleri kontrol ederek fare imleci altındaki karakteri belirler. Önce sınırlayıcı kutu ile kesişen, sonra daha genel bir testle bulunan karakteri seçer. Seçilen karakter değişirse `OnSelected()` / `OnUnselected()` olaylarını tetikler.
*   **Çarpışma Ayarı (`AdjustCollisionWithOtherObjects`)**:
    *   Bir oyuncu karakterinin (`IsPC()`) diğer statik nesnelerle (PC, NPC, Düşman olmayanlar) fiziksel olarak harmanlanmış bir çarpışması (`TestPhysicsBlendingCollision`) olup olmadığını kontrol eder. Çarpışma varsa, karakterin pozisyonunu mevcut pozisyonuna geri ayarlar (`SetBlendingPosition`). Bu, karakterlerin nesnelerin içine girmesini engellemeye yardımcı olur.
*   **Durum ve Efekt Yönetimi**:
    *   `InsertPVPKey()` / `RemovePVPKey()`: İki karakter arasında PvP durumunu (düşmanlık) ayarlar veya kaldırır. Bu durum değişikliği sonrası karakterlerin isim etiketleri (`RefreshTextTail`) güncellenir.
    *   `ChangeGVG()`: Lonca savaşı durumunda olan loncaların üyelerinin isim etiketlerini günceller.
    *   `ShowPointEffect()`: Belirli bir karakter (`dwVID`) için önceden `RegisterPointEffect` ile kaydedilmiş bir efekti (örn: seviye atlama, yetenek puanı artışı) oynatır (`CInstanceBase::LevelUp`, `CInstanceBase::SkillUp`).
*   **Optimizasyon**:
    *   `IsCacheMode()`: Canlı karakter sayısına göre bir "önbellek modu"nun aktif olup olmayacağına karar verir (çok fazla karakter varsa aktifleşir, azsa devre dışı kalır). Bu modun tam olarak neyi optimize ettiği koddaki yorumlardan net anlaşılmamaktadır, muhtemelen bazı güncellemeleri basitleştirir veya render detaylarını azaltır.
    *   `EnableSortRendering(bool isEnable)`: Bu fonksiyonun içi boştur, muhtemelen gelecekteki bir özellik için veya eski bir kalıntıdır.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonCharacterManager`, oyun dünyasındaki tüm karakterlerin merkezi sinir sistemi gibidir ve istemcinin sorunsuz çalışması için kritik öneme sahiptir.

*   **Karakterlerin Görselleştirilmesi**: Her frame'de `Update()` ve `Render()` fonksiyonları çağrılarak karakterlerin animasyonları, pozisyonları güncellenir ve ekrana çizilir.
*   **Oyuncu Etkileşimi**: Fare ile bir karaktere tıklandığında, `__NEW_Pick()` mekanizması sayesinde o karakter `m_pkInstPick` olarak ayarlanır ve oyun arayüzü (UI) bu bilgiye göre (örneğin, hedef penceresini göstermek) tepki verebilir.
*   **Ağ İletişimi Entegrasyonu**: Sunucudan gelen karakter oluşturma, silme, hareket etme veya durum değiştirme paketleri işlendiğinde, `CPythonNetworkStream` gibi sınıflar `CPythonCharacterManager`'ın ilgili fonksiyonlarını (örn: `CreateInstance`, `DeleteInstance`, veya `CInstanceBase` üzerindeki hareket/efekt fonksiyonları) çağırarak oyun dünyasını senkronize eder.
*   **NPC ve Canavar Davranışları**: NPC ve canavarların yapay zekası (AI) güncellendiğinde, bu güncellemeler genellikle `CInstanceBase` türevli sınıflar içinde yönetilir ve `CPythonCharacterManager` bunların toplu olarak güncellenmesini sağlar.
*   **Savaş ve Yetenek Kullanımı**: Bir karakter başka bir karaktere saldırdığında veya bir yetenek kullandığında, hedef seçimi, mesafe kontrolü ve efektlerin uygulanması gibi işlemler `CPythonCharacterManager` ve yönettiği `CInstanceBase` nesneleri üzerinden koordine edilir.
*   **Görsel Efektler**: Oyuncunun seviye atlaması gibi önemli olaylarda `ShowPointEffect` çağrılarak ilgili görsel efektler karakterin üzerinde belirir.
*   **Oyun İçi Olaylar**: PvP turnuvaları veya lonca savaşları gibi özel olaylarda, `InsertPVPKey` ve `ChangeGVG` fonksiyonları kullanılarak karakterler arasındaki ilişkiler ve görünümleri dinamik olarak değiştirilir.
*   **Performans Yönetimi**: Uzak karakterlerin güncellenmemesi veya daha az detayla render edilmesi gibi optimizasyonlar bu sınıf aracılığıyla dolaylı olarak yönetilir.

Bu sınıf, `CInstanceBase` ile çok yakından çalışır ve `CInstanceBase`'in bireysel karakter mantığını yönetirken, `CPythonCharacterManager` tüm bu bireylerin bir arada, tutarlı bir şekilde var olmasını ve etkileşimde bulunmasını sağlar.

---

### `PythonCharacterManagerModule.cpp` (Python `chrmgr` Modülü)

Bu dosya, `CPythonCharacterManager`, `CRaceManager` ve `CRaceData` sınıflarının çeşitli işlevlerini Python betiklerine `chrmgr` adlı bir modül aracılığıyla sunar. Bu modül, karakterlerin genel görünüm ayarları, ırk verileri, efektler ve isim/başlık renkleri gibi özelliklerin Python tarafından yönetilmesini sağlar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`chrmgr`)**: Karakter yönetimi ve ırk verileriyle ilgili C++ fonksiyonlarını Python'a açan `chrmgr` modülünü oluşturur.
*   **Irk ve Görünüm Veri Yönetimi**: Irk tanımlamaları, hareket (motion) verileri, kemik bağlantıları, şekil modelleri ve kaplamalarının Python üzerinden yapılandırılmasını sağlar.
*   **Efekt ve Renk Yönetimi**: Karakter isim renkleri, unvan renkleri, genel efektler ve noktasal efektlerin kaydedilmesi ve gösterilmesi için arayüz sunar.
*   **Sabitlerin Aktarılması**: İsim renkleri (`NAMECOLOR_*`), efekt türleri (`EFFECT_*`) gibi C++ sabitlerini Python'a aktararak betiklerde kullanılabilir hale getirir.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initchrmgr()` Fonksiyonu**:
    *   Bu C fonksiyonu, Python yorumlayıcısı `chrmgr` modülünü ilk kez import ettiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyonların isimlerini (Python'da kullanılacak isim: örn. "SetEmpireNameMode"), karşılık gelen C fonksiyon adreslerini (örn. `chrmgrSetEmpireNameMode`) ve argüman türlerini (`METH_VARARGS`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("chrmgr", s_methods);` çağrısı ile `chrmgr` modülünü oluşturur ve `s_methods` dizisindeki fonksiyonları bu modüle kaydeder.
    *   `PyModule_AddIntConstant(poModule, "PYTHON_CONSTANT_NAME", CPLUSPLUS_CONSTANT_VALUE);` çağrıları ile çeşitli C++ sabitleri (örneğin `CInstanceBase::NAMECOLOR_NORMAL_MOB` sabiti `chrmgr.NAMECOLOR_MOB` olarak) Python modülüne eklenir.

*   **Fonksiyon Sarmalayıcıları (`chrmgr*` önekli fonksiyonlar)**:
    *   Her bir C++ işlevi için (genellikle `CRaceManager::Instance().MethodAdi()`, `CInstanceBase::StaticMethod()` veya `CPythonCharacterManager::Instance().MethodAdi()` çağrıları) bir Python sarmalayıcı fonksiyonu tanımlanmıştır.
    *   Bu sarmalayıcılar, Python'dan gelen argümanları (`PyObject* poArgs`) alır, `PyTuple_GetInteger`, `PyTuple_GetString` gibi fonksiyonlarla uygun C++ türlerine dönüştürür, ilgili C++ fonksiyonunu çağırır ve C++ fonksiyonunun dönüş değerini Python nesnesine (`Py_BuildValue`, `Py_BuildNone` vb.) dönüştürerek geri gönderir.
    *   **Örnekler**:
        *   `chrmgrSetEmpireNameMode(PyObject* poSelf, PyObject* poArgs)`: `CInstanceBase::SetEmpireNameMode` ve `CPythonCharacterManager::Instance().RefreshAllPCTextTail` fonksiyonlarını çağırır.
        *   `chrmgrLoadRaceData(PyObject* poSelf, PyObject* poArgs)`: `CRaceManager::Instance().GetSelectedRaceDataPointer()` ile seçili ırk verisini alır ve `pRaceData->LoadRaceData()` ile ırk verisini yükler.
        *   `chrmgrRegisterEffect(PyObject* poSelf, PyObject* poArgs)`: `CInstanceBase::RegisterEffect` statik metodunu çağırarak bir efekt türünü belirli bir kemik ve dosya yolu ile kaydeder.
        *   `chrmgrSetAffect(PyObject* poSelf, PyObject* poArgs)`: `CPythonCharacterManager::Instance().SCRIPT_SetAffect()` metodunu çağırarak belirli bir karakterin (VID ile belirtilen) bir affect durumunu (görsel efekt) ayarlar.

#### Oyunda Kullanım Amacı ve Senaryoları

`chrmgr` modülü, genellikle oyunun başlangıç aşamasında ve karakterlerle ilgili temel yapılandırmaların yüklenmesi sırasında Python betikleri tarafından kullanılır.

*   **Irk Verilerinin Yüklenmesi**: Oyunun kullandığı `msm` (motion script files) gibi betik dosyaları, `chrmgr` fonksiyonlarını (`LoadRaceData`, `RegisterMotionData`, `SetShapeModel` vb.) çağırarak her bir karakter ırkı için modelleri, animasyonları, saldırı türlerini ve diğer ırka özel verileri C++ tarafına kaydeder. Bu, genellikle `root` veya `locale` içindeki Python betikleri aracılığıyla yapılır.
*   **Görsel Ayarların Yapılandırılması**: Karakter isimlerinin ve unvanlarının hangi durumlarda (örneğin, PK, parti üyesi) hangi renkte görüneceği `chrmgr.RegisterNameColor` ve `chrmgr.RegisterTitleColor` ile ayarlanır. İmparatorluk isimlerinin gösterilip gösterilmeyeceği `chrmgr.SetEmpireNameMode` ile kontrol edilir.
*   **Efektlerin Kaydedilmesi**: Oyun içinde kullanılacak çeşitli genel efektler (örneğin, stun efekti, seviye atlama efekti parçacıkları) `chrmgr.RegisterEffect` veya `chrmgr.RegisterPointEffect` ile önceden kaydedilir.
*   **Geliştirme ve Test**: `chrmgr.GetVIDInfo` gibi fonksiyonlar, geliştirme sırasında belirli bir karakterin durumunu sorgulamak için kullanılabilir.

Bu modül, karakter sisteminin temel yapı taşlarının Python üzerinden esnek bir şekilde tanımlanmasına ve yönetilmesine olanak tanır.

---

### `PythonCharacterModule.cpp` (Python `chr` Modülü)

Bu dosya, oyundaki bireysel karakter örnekleriyle (`CInstanceBase` türevleri) ve genel karakter işlemleriyle etkileşim kurmak için Python betiklerine `chr` adlı bir modül sunar. Bu modül, karakterlerin oluşturulması, silinmesi, seçilmesi, görünümlerinin değiştirilmesi, hareketlerinin ve pozisyonlarının yönetilmesi gibi çok çeşitli işlevleri içerir.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`chr`)**: Karakter örneklerini yönetmek ve onlarla etkileşimde bulunmak için `chr` adlı bir Python modülü oluşturur.
*   **Karakter Örneği Yaşam Döngüsü**: Karakterlerin oluşturulması (`CreateInstance`), güncellenmesi (`Update`, `Deform`), render edilmesi (`Render`) ve silinmesi (`DeleteInstance`) için Python arayüzleri sağlar.
*   **Karakter Etkileşimi ve Kontrolü**: Karakter seçimi (`SelectInstance`), hareketleri (`PushOnceMotion`, `SetPixelPosition`), görünüm değişiklikleri (zırh, silah, saç: `SetArmor`, `SetWeapon`, `SetHair`), durumları (ölü/canlı: `Die`, `Revive`) ve diğer eylemleri Python'dan yönetme imkanı sunar.
*   **Bilgi Sorgulama**: Karakterlerin mevcut durumu (pozisyon, isim, ırk, tip, VID vb.) hakkında bilgi almak için fonksiyonlar içerir.
*   **Sabitlerin Aktarılması**: Hareket (motion) isimleri (`MOTION_*`), yönler (`DIR_*`), örnek tipleri (`INSTANCE_TYPE_*`), karakter parçaları (`PART_*`) ve affect (etki) türleri (`AFFECT_*`) gibi C++ sabitlerini Python'a aktarır.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initchr()` Fonksiyonu**:
    *   Python yorumlayıcısı `chr` modülünü ilk kez import ettiğinde bu fonksiyon çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyonları (örn. "CreateInstance", "SelectInstance", "SetPixelPosition") ve bunlara karşılık gelen C sarmalayıcı fonksiyonlarını (örn. `chrCreateInstance`, `chrSelectInstance`, `chrSetPixelPosition`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("chr", s_methods);` ile `chr` modülü oluşturulur ve fonksiyonlar kaydedilir.
    *   Çok sayıda C++ sabiti (`CRaceMotionData::NAME_WAIT` -> `chr.MOTION_WAIT`, `CActorInstance::TYPE_PC` -> `chr.INSTANCE_TYPE_PLAYER` gibi) `PyModule_AddIntConstant` kullanılarak modüle eklenir. Bu, Python betiklerinin bu sabitlere isimleriyle erişmesini sağlar.

*   **Fonksiyon Sarmalayıcıları (`chr*` önekli fonksiyonlar)**:
    *   Bu C fonksiyonları, Python'dan gelen argümanları (`PyObject* poArgs`) işler (`PyTuple_GetInteger`, `PyTuple_GetString`, `PyTuple_GetFloat` vb.).
    *   Genellikle `CPythonCharacterManager::Instance()` üzerinden seçili karakter örneğini (`GetSelectedInstancePtr()`) veya VID ile belirli bir karakter örneğini (`GetInstancePtr(vid)`) alırlar.
    *   Elde edilen karakter örneği (`CInstanceBase* pkInst`) üzerinde ilgili C++ metodunu (örn. `pkInst->SetArmor(form)`, `pkInst->PushOnceMotion(motion)`) çağırırlar.
    *   Bazı fonksiyonlar doğrudan `CPythonCharacterManager` üzerinde işlem yapar (örn. `chrCreateInstance` -> `CPythonCharacterManager::Instance().CreateInstance(kCreateData)`).
    *   Sonuçları (varsa) Python nesnelerine (`Py_BuildValue`, `Py_BuildNone`) dönüştürerek geri gönderirler.
    *   **Örnekler**:
        *   `chrCreateInstance(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir sanal ID (VID) ve isteğe bağlı bir sözlük (pozisyon, at VNUM'u vb. için) alır, `CInstanceBase::SCreateData` yapısını doldurur ve `CPythonCharacterManager::Instance().CreateInstance()` ile yeni bir karakter örneği oluşturur.
        *   `chrSelectInstance(PyObject* poSelf, PyObject* poArgs)`: Bir VID alır ve `CPythonCharacterManager::Instance().SelectInstance()` ile o karakteri "seçili karakter" yapar.
        *   `chrSetPixelPosition(PyObject* poSelf, PyObject* poArgs)`: Seçili karakterin piksel pozisyonunu ayarlar (`pkInst->SCRIPT_SetPixelPosition(x, y)` veya `pkInst->NEW_SetPixelPosition(TPixelPosition(x, y, z))`).
        *   `chrPushOnceMotion(PyObject* poSelf, PyObject* poArgs)`: Seçili karaktere bir kez oynatılacak bir hareket (animasyon) iter (`pkInst->PushOnceMotion(motionIndex, blendTime)`).
        *   `chrGetNameByVID(PyObject* poSelf, PyObject* poArgs)`: Verilen VID'ye sahip karakterin adını döndürür.
        *   `chrIsNPC(PyObject* poSelf, PyObject* poArgs)`: Verilen VID'ye sahip karakterin NPC olup olmadığını boolean olarak döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

`chr` modülü, istemcinin Python betikleri için karakterlerle doğrudan etkileşim kurmanın ana yoludur.

*   **Karakter Oluşturma ve Yönetimi**:
    *   Özel görevler, ara sahneler veya test amaçlı olarak Python'dan dinamik karakterler (`chr.CreateInstance`) oluşturulabilir ve silinebilir (`chr.DeleteInstance`).
*   **Kullanıcı Arayüzü Etkileşimleri**:
    *   Oyuncu bir NPC'ye tıkladığında, UI scripti `chr.GetVirtualNumber(vid)` ile NPC'nin VNUM'unu alabilir ve buna göre diyalog penceresi açabilir.
    *   Envanterdeki bir zırha tıklandığında, `chr.SetArmor(armorVnum)` (genellikle seçili ana karakter için örtük olarak) çağrılarak karakterin görünümü güncellenebilir.
    *   Yetenek çubuğundan bir yetenek seçildiğinde, ilgili hareket (`chr.PushOnceMotion(chr.MOTION_SKILL)`) tetiklenebilir.
*   **Karakter Durum Kontrolü ve Bilgilendirme**:
    *   `chr.GetNameByVID(targetVID)` ile hedefin adı alınarak UI'da gösterilebilir.
    *   `chr.GetPixelPosition(playerVID)` ile oyuncunun pozisyonu UI scriptleri tarafından takip edilebilir.
    *   `chr.IsEnemy(targetVID)` gibi fonksiyonlarla hedefin oyuncuya karşı tutumu belirlenebilir.
*   **Hareket ve Animasyon Kontrolü**:
    *   Belirli sosyal hareketler (dans, alkış) `chr.PushOnceMotion(chr.MOTION_DANCE_1)` gibi komutlarla tetiklenebilir.
    *   Karakterin duruşu (`chr.SetLoopMotion(chr.MOTION_WAIT)`), yürümesi (`chr.SetLoopMotion(chr.MOTION_WALK)`) veya koşması (`chr.SetLoopMotion(chr.MOTION_RUN)`) gibi temel hareket döngüleri ayarlanabilir.
*   **Efekt ve Görünüm Ayarları**:
    *   Karakterin saç modeli (`chr.ChangeHair`), silahı (`chr.ChangeWeapon`), zırhı (`chr.ChangeShape`) gibi görünümleri dinamik olarak değiştirilebilir.
    *   Karakterlerin üzerine fare ile gelindiğinde veya seçildiğinde vurgulamak için `chr.Select(vid)` ve `chr.Unselect(vid)` kullanılabilir, bu fonksiyonlar karakterin render modunu değiştirebilir.
*   **Geliştirme ve Hata Ayıklama**:
    *   Modül içindeki çok sayıda `test*` fonksiyonu (örn: `chrtestSetAddRenderModeRGB`, `chrFaintTest`) geliştiricilerin çeşitli karakter özelliklerini ve render modlarını hızlıca test etmesini sağlar.
    *   `chr.MoveToDestPosition` gibi fonksiyonlarla karakter hareketleri programatik olarak test edilebilir.

`chr` modülü, `CPythonCharacterManager` ile birlikte çalışarak oyun dünyasındaki tüm karakterlerin Python üzerinden yönetilmesini ve istemci tarafı mantığının büyük bir kısmının Python'da yazılmasını mümkün kılar. Bu, UI geliştirme ve oyun mekaniklerinin prototiplenmesi/uygulanması süreçlerinde esneklik sağlar.

---

### `PythonChat.h` ve `PythonChat.cpp` (Sohbet Yönetimi)

Bu dosyalar, Metin2 istemcisindeki tüm oyun içi sohbet işlevlerini yöneten `CPythonChat` singleton sınıfını ve özel mesajlaşmalar için kullanılan `CWhisper` yardımcı sınıfını tanımlar ve implemente eder. Sohbet mesajlarının alınması, farklı türlere (genel, grup, lonca, fısıltı vb.) göre renklendirilmesi, farklı sohbet pencerelerinde (setlerinde) gösterilmesi, mesajların zamanla kaybolması ve oyuncu engelleme gibi özellikler bu sınıflar tarafından yönetilir.

#### Dosyaların Genel Amaçları

*   **`PythonChat.h` (Başlık Dosyası)**:
    *   `CPythonChat` singleton sınıfının ve `CWhisper` sınıfının tanımlarını içerir.
    *   Sohbet mesajlarını (`SChatLine`), sohbet penceresi yapılandırmalarını (`SChatSet`) ve bekleyen mesajları (`SWaitChat`) temsil eden yapıları tanımlar.
    *   Sohbet türleri (`CHAT_TYPE_*`), fısıltı türleri (`WHISPER_TYPE_*`), mesaj panosu durumları (`BOARD_STATE_*`) ve hyperlink türleri (`HYPER_LINK_ITEM_*`) için `enum` tanımları barındırır.
    *   Sohbet hatları için dinamik bellek havuzları (`CDynamicPool`) kullanır.
    *   Sohbet pencereleri oluşturma, güncelleme, render etme, yapılandırma (pozisyon, boyut, filtreler), mesaj ekleme, fısıltı yönetimi ve engellenen karakterler listesi için fonksiyon prototiplerini deklare eder.
    *   `ENABLE_CHAT_LOG_FIX` ve `ENABLE_CHATTING_WINDOW_RENEWAL` gibi derleme zamanı makrolarına bağlı olarak ek özellikler ve tanımlamalar içerebilir.

*   **`PythonChat.cpp` (C++ Implementasyon Dosyası)**:
    *   `CPythonChat` ve `CWhisper` sınıflarının tüm metotlarının detaylı C++ implementasyonlarını içerir.
    *   **`CPythonChat` Implementasyonu**:
        *   `SChatLine` Yönetimi: Sohbet satırları için `CDynamicPool` kullanarak bellek tahsisi ve serbest bırakma (`SChatLine::New`, `SChatLine::Delete`). Her satırın kendine ait renkleri (`aColor`) ve eklenme zamanı (`fAppendedTime`) vardır.
        *   `TChatSet` Yönetimi: Farklı sohbet penceresi yapılandırmalarını (`m_ChatSetMap`) yönetir. Her setin pozisyonu (`m_ix`, `m_iy`), yüksekliği (`m_iHeight`), satır adımı (`m_iStep`), görünür mesajların kaydırma pozisyonu (`m_fEndPos`), panonun durumu (`m_iBoardState` - VIEW, EDIT, LOG) ve aktif sohbet türlerini belirleyen modları (`m_iMode`) bulunur.
        *   Mesaj Ekleme (`AppendChat`, `AppendChatWithDelay`): Gelen sohbet mesajlarını ana mesaj kuyruğuna (`m_ChatLineDeque`) ekler. Belirli bir sayıdan fazla mesaj biriktiğinde en eskisi silinir. Mesajlar, tüm aktif sohbet setlerinin gösterim listelerine de dağıtılır. `AppendChatWithDelay` ile mesajlar gecikmeli olarak eklenebilir.
        *   Güncelleme (`Update`): Her sohbet seti için durumuna (`BOARD_STATE_VIEW`, `BOARD_STATE_EDIT`, `BOARD_STATE_LOG`) göre `UpdateViewMode`, `UpdateEditMode` veya `UpdateLogMode` çağırır. Bu fonksiyonlar, mesajların zamanla solmasını (alpha değerinin azalması), yeni mesajların eklenmesini ve kaydırma çubuğu pozisyonuna göre gösterilecek mesajların düzenlenmesini yönetir.
        *   Render (`Render`): Her sohbet setindeki görünür mesajları (`CGraphicTextInstance::Render`) ekrana çizer.
        *   Fısıltı Yönetimi (`CreateWhisper`, `AppendWhisper`, `ClearWhisper`, `GetWhisper`, `InitWhisper`): Oyuncular arası özel mesajları yönetir. Her fısıltı oturumu bir `CWhisper` nesnesiyle temsil edilir ve `m_WhisperMap` içinde saklanır.
        *   Engelleme (`IgnoreCharacter`, `IsIgnoreCharacter`): Engellenen oyuncuların listesini (`m_IgnoreCharacterSet`) tutar.
        *   Renk Ayarları (`SetChatColor`, `GetChatColor`): Farklı sohbet türleri için varsayılan renkleri ayarlar ve sorgular.
        *   `ENABLE_CHAT_LOG_FIX` makrosu aktifse, ayrı bir sohbet kayıt (log) sistemi (`m_ChatLogLineDeque`, `m_ShowingChatLogLineList`) için benzer işlemler yapılır.
    *   **`CWhisper` Implementasyonu**:
        *   Her bir fısıltı oturumu için ayrı bir mesaj kuyruğu (`m_ChatLineDeque`) ve gösterim listesi (`m_ShowingChatLineList`) tutar.
        *   Mesaj ekleme (`AppendChat`), pozisyon/boyut ayarlama (`SetPosition`, `SetBoxSize`) ve render (`Render`) işlevlerini sağlar.
        *   Mesajlar, fısıltı penceresinin boyutuna ve kaydırma pozisyonuna (`m_fcurPosition`) göre düzenlenir (`__ArrangeChat`).
        *   `SChatLine` için kendi bellek havuzunu kullanır.

#### Başlık Dosyası Tanımları (`PythonChat.h`)

*   **Sınıflar ve Yapılar**:
    *   `CPythonChat`: Ana sohbet yöneticisi, singleton.
    *   `CWhisper`: Bireysel fısıltı oturumlarını yöneten sınıf.
    *   `CPythonChat::SChatLine`: Tek bir sohbet satırını temsil eder (metin, renk, eklenme zamanı, `CGraphicTextInstance`).
    *   `CWhisper::SChatLine`: `CWhisper` için özelleşmiş sohbet satırı yapısı.
    *   `CPythonChat::TChatSet`: Bir sohbet penceresinin yapılandırmasını ve durumunu tutar.
    *   `CPythonChat::SWaitChat`: Gecikmeli eklenecek sohbet mesajı verilerini tutar.
*   **Enum Tanımları**:
    *   `EWhisperType`: Fısıltı mesajının türünü belirtir (normal sohbet, hedef yok, engellenmiş vb.).
    *   `EHyperlinkType`: Sohbet mesajlarındaki özel link türlerini (eşya linki gibi) tanımlar.
    *   `EBoardState`: Sohbet penceresinin modunu (sadece görüntüleme, düzenleme, kayıt) belirtir.
*   **Önemli Üye Değişkenler (`CPythonChat`)**:
    *   `m_ChatLineDeque`: Tüm sohbet mesajlarının tutulduğu ana kuyruk.
    *   `m_ShowingChatLineList`: (Artık doğrudan kullanılmıyor gibi, `TChatSet` içinde her sete özel liste var).
    *   `m_ChatSetMap`: Oluşturulan tüm sohbet setlerini (pencerelerini) ID'leri ile eşleyerek tutar.
    *   `m_WhisperMap`: Aktif fısıltı oturumlarını oyuncu isimleriyle eşleyerek tutar.
    *   `m_IgnoreCharacterSet`: Engellenen oyuncu isimlerini tutan set.
    *   `m_WaitChatList`: Gecikmeli eklenecek mesajların listesi.
    *   `m_akD3DXClrChat[CHAT_TYPE_MAX_NUM]`: Her sohbet türü için D3DXCOLOR renklerini tutan dizi.
    *   `ENABLE_CHAT_LOG_FIX` ile: `m_ChatLogLineDeque`, `m_ShowingChatLogLineList`.
*   **Önemli Fonksiyon Prototipleri (`CPythonChat`)**:
    *   `CreateChatSet()`: Yeni bir sohbet penceresi yapılandırması oluşturur.
    *   `Update()`, `Render()`: Belirli bir sohbet setini günceller ve çizer.
    *   `SetBoardState()`, `SetPosition()`, `SetHeight()`, `SetStep()`, `ToggleChatMode()`, `SetEndPos()`: Sohbet seti ayarlarını değiştirir.
    *   `AppendChat()`: Normal sohbet mesajı ekler.
    *   `AppendWhisper()`: Fısıltı mesajı ekler.
    *   `IgnoreCharacter()`: Bir oyuncuyu engeller/engelini kaldırır.

#### C++ Implementasyon Detayları (`PythonChat.cpp`)

*   **Mesaj Ömrü ve Solma Efekti**: `UpdateViewMode` içinde, `c_fStartDisappearingTime` (5 saniye) süresini aşan veya maksimum satır sayısını (`c_iMaxLineCount`) geçen mesajların alpha değeri kademeli olarak azaltılır. Alpha değeri çok düşünce mesaj listeden silinir.
*   **Düzenleme Modu (Edit Mode) Alpha Ayarı**: `UpdateEditMode` içinde, düzenleme modundayken görünür olan ancak düzenlenebilir alanın dışındaki eski mesajların alpha değeri, yeni mesajlara yer açmak için daha hızlı bir şekilde ayarlanır.
*   **Mesajların Düzenlenmesi (`ArrangeShowingChat`)**: Bir sohbet setinin filtreleri (aktif modlar) veya kaydırma pozisyonu değiştiğinde, o sette gösterilecek mesaj listesi (`m_ShowingChatLineList`) yeniden oluşturulur. Ana mesaj kuyruğundan (`m_ChatLineDeque`) aktif modlara uyan mesajlar alınır ve setin yüksekliği ile kaydırma pozisyonuna göre görünür olacaklar seçilir.
*   **Kaynak Yönetimi**: `CGraphicTextInstance` nesneleri ve `SChatLine`, `CWhisper` gibi yapılar için `CDynamicPool` kullanılarak verimli bellek yönetimi yapılır. `DefaultFont_GetResource()` ile varsayılan yazı tipi kaynağı alınır.
*   **Gecikmeli Mesajlar**: `m_WaitChatList` içinde tutulan mesajlar, `CPythonChat::Update` içinde zamanları geldiğinde (`dwAppendingTime < dwcurTime`) normal `AppendChat` ile işlenir.
*   **Fısıltı Penceresi (`CWhisper`)**: Her `CWhisper` nesnesi kendi içinde `m_ChatLineDeque` tutar. `Render` fonksiyonu, fısıltı kutusunun boyutu (`m_fWidth`, `m_fHeight`) ve kaydırma pozisyonuna (`m_fcurPosition`) göre mesajları çizer. Metinler çok satırlı (`SetMultiLine(true)`) olabilir ve belirtilen genişliğe (`SetLimitWidth`) göre sığdırılır.

#### Python Arayüzü (`PythonChatModule.cpp` - Bu dosyada değil, ayrı bir modül dosyasında)

`CPythonChat` sınıfının fonksiyonları genellikle Python'a `chat` veya benzeri bir modül aracılığıyla sunulur. Bu modül, UI scriptlerinin sohbet pencerelerini oluşturmasını, yapılandırmasını, mesaj göndermesini ve almasını sağlar.

*   `chat.CreateChatSet(id)`: Yeni bir sohbet penceresi oluşturur.
*   `chat.SetChatPosition(id, x, y)`, `chat.SetChatHeight(id, height)`: Pencere boyut ve konumunu ayarlar.
*   `chat.ToggleChatMode(id, chatType, enable)`: Belirli bir sohbet türünün (örn: LONCA, GRUP) o pencerede gösterilip gösterilmeyeceğini ayarlar.
*   `chat.AppendChat(chatType, message)`: Oyun içinde bir mesaj gösterir (genellikle istemci taraflı bilgilendirme veya sistem mesajları için).
*   `chat.GetWhisper(userName)`: Belirli bir kullanıcıyla olan fısıltı geçmişini almak için kullanılır.
*   `chat.IgnorePlayer(userName)`: Oyuncuyu engelleme/engeli kaldırma.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonChat` sistemi, oyun içi iletişimin temelini oluşturur.

*   **Genel Sohbet Penceresi**: Oyuncuların genel, grup, lonca, bağırma gibi mesajlarını gördüğü ana sohbet penceresi bir `TChatSet` olarak yönetilir. Filtreler aracılığıyla oyuncu hangi tür mesajları görmek istediğini seçebilir.
*   **Fısıltı Pencereleri**: Her bir özel sohbet (fısıltı) ayrı bir `CWhisper` nesnesi ve buna bağlı bir UI penceresi ile yönetilir. `CPythonChat`, bu `CWhisper` nesnelerinin merkezi bir kaydını tutar.
*   **Sistem Mesajları**: Sunucudan gelen duyurular, eşya düşürme bilgileri, seviye atlama bildirimleri gibi bilgilendirme mesajları `CHAT_TYPE_INFO` veya `CHAT_TYPE_NOTICE` olarak `AppendChat` ile eklenir.
*   **Engelleme**: Oyuncular, rahatsız edici buldukları diğer oyuncuları `IgnoreCharacter` ile engelleyebilir, bu sayede o kişiden gelen mesajlar gösterilmez.
*   **Eşya Linkleri**: Sohbet satırları içinde özel formatlarla (`item:vnum:flag:socket0...`) eşya linkleri oluşturulabilir. Bu linklere tıklandığında `EHyperlinkType` ile tanımlanan bilgi tooltip üzerinde gösterilir.
*   **Arayüz Entegrasyonu**: Python ile yazılmış UI scriptleri, `CPythonChat`'ın sunduğu arayüzü kullanarak sohbet pencerelerini oluşturur, oyuncu girdilerini alır, mesajları sunucuya gönderir (bu kısım `CPythonNetworkStream` ile bağlantılıdır) ve gelen mesajları `CPythonChat` aracılığıyla görüntüler.

Bu sistem, oyunun sosyal etkileşim özellikleri için hayati öneme sahiptir ve oyuncuların birbirleriyle ve oyun dünyasıyla iletişim kurmasını sağlar.

---

### `PythonChatModule.cpp` (Python `chat` Modülü)

Bu dosya, `CPythonChat` sınıfının ve `CWhisper` sınıfının işlevlerini Python betiklerine `chat` adlı bir modül aracılığıyla sunar. Bu modül, oyun içi sohbet pencerelerinin (setlerinin) oluşturulması, yapılandırılması, mesajların eklenmesi, fısıltı (özel mesaj) yönetimi, oyuncu engelleme ve sohbet içi eşya bağlantılarının işlenmesi gibi temel sohbet sistemi etkileşimlerini Python üzerinden mümkün kılar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`chat`)**: `CPythonChat` sınıfının genel sohbet ve fısıltı yönetimi işlevlerini Python'a açan `chat` modülünü oluşturur.
*   **Sohbet Penceresi (Set) Yönetimi**: Sohbet pencerelerinin oluşturulması (`CreateChatSet`), güncellenmesi (`Update`), çizdirilmesi (`Render`) ve çeşitli özelliklerin (pozisyon, boyut, satır adımı, kaydırma pozisyonu, aktif sohbet türleri) Python'dan ayarlanmasını sağlar.
*   **Mesaj Ekleme ve Yönetimi**: Farklı türlerde sohbet mesajlarının (`AppendChat`, `AppendChatWithDelay`) eklenmesini ve gösterilecek mesajların düzenlenmesini (`ArrangeShowingChat`) destekler.
*   **Fısıltı (Whisper) Yönetimi**: Belirli oyuncularla özel sohbet oturumları oluşturma (`CreateWhisper`), mesaj ekleme (`AppendWhisper`), fısıltı penceresini çizdirme (`RenderWhisper`) ve yapılandırma (`SetWhisperBoxSize`, `SetWhisperPosition`) işlevlerini sunar.
*   **Oyuncu Engelleme**: Oyuncuları engelleme (`IgnoreCharacter`) ve engelli olup olmadığını kontrol etme (`IsIgnoreCharacter`) imkanı tanır.
*   **Eşya Bağlantısı (Hyperlink) İşleme**: Sohbet mesajlarındaki özel formatlı eşya bağlantılarını (`item:vnum:flag...`) çözümleyerek oyun içinde tıklanabilir ve bilgisi görüntülenebilir metin bağlantılarına (`|cffffc700|Hitem:...|h[Eşya Adı]|h|r`) dönüştürür (`GetLinkFromHyperlink`).
*   **Sabitlerin Aktarılması**: Sohbet türleri (`CHAT_TYPE_*`), fısıltı türleri (`WHISPER_TYPE_*`), pano durumları (`BOARD_STATE_*`) ve hyperlink elemanları (`HYPER_LINK_ITEM_*`) gibi C++ sabitlerini Python modülüne aktarır.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initChat()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `chat` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyon adlarını (örn. "CreateChatSet", "AppendChat") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `chatCreateChatSet`, `chatAppendChat`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("chat", s_methods);` ile `chat` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak `CHAT_TYPE_TALKING`, `WHISPER_TYPE_CHAT`, `HYPER_LINK_ITEM_VNUM` gibi çok sayıda C++ sabiti Python modülüne (`chat.CHAT_TYPE_TALKING` vb.) aktarılır.

*   **Fonksiyon Sarmalayıcıları (`chat*` önekli fonksiyonlar)**:
    *   Bu C fonksiyonları, Python'dan gelen argümanları (`PyObject* poArgs`) alır, `PyTuple_GetInteger`, `PyTuple_GetString`, `PyTuple_GetFloat` gibi fonksiyonlarla C++ türlerine dönüştürür.
    *   `CPythonChat::Instance()` üzerinden ilgili C++ metodunu çağırır (örn. `CPythonChat::Instance().CreateChatSet(iID)`).
    *   Fısıltı ile ilgili fonksiyonlar (`RenderWhisper`, `SetWhisperBoxSize` vb.) için önce `CPythonChat::Instance().GetWhisper(szName, &pWhisper)` ile ilgili `CWhisper` nesnesi alınır, sonra bu nesne üzerinde işlem yapılır.
    *   Sonuçları (varsa) Python nesnelerine (`Py_BuildValue`, `Py_BuildNone`) dönüştürerek geri gönderir.
    *   **Örnekler**:
        *   `chatSetChatColor(PyObject* poSelf, PyObject* poArgs)`: Sohbet türü ve RGB renk değerlerini alarak `CPythonChat::Instance().SetChatColor()` çağırır.
        *   `chatAppendChat(PyObject* poSelf, PyObject* poArgs)`: Sohbet türünü ve mesaj metnini alarak `CPythonChat::Instance().AppendChat()` çağırır.
        *   `chatGetLinkFromHyperlink(PyObject* poSelf, PyObject* poArgs)`: Özel formatlı bir eşya hyperlink string'ini (`item:vnum:flag...`) alır. Bu string'i `:` karakterine göre böler (`split_string`). Eğer geçerli bir eşya linki ise, `CItemManager::Instance().GetItemDataPointer()` ile eşya verisini alır ve tooltip formatında (`|c...|H...|h[İsim]|h|r`) bir string oluşturarak Python'a döndürür. Bu işlem, eşyanın temel VNUM'u, bayrakları, soketleri ve `ENABLE_` makrolarıyla tanımlanmış ek özellikleri (set item, change look, refine element, apply random, attributes) içerecek şekilde dinamik olarak yapılır.

#### Oyunda Kullanım Amacı ve Senaryoları

`chat` modülü, Python tabanlı kullanıcı arayüzü (UI) scriptlerinin oyun içi sohbet sistemini tamamen kontrol etmesini sağlar.

*   **Sohbet Arayüzü Yönetimi**: UI scriptleri, `chat.CreateChatSet` ile farklı amaçlar için sohbet pencereleri oluşturabilir (örn. ana sohbet, sistem logları). Bu pencerelerin konumu, boyutu, hangi sohbet türlerini göstereceği (`chat.ToggleChatMode`) Python tarafından dinamik olarak ayarlanır.
*   **Mesaj Gönderimi ve Alımı**: Oyuncu sohbet satırına bir mesaj yazdığında, UI bu mesajı alır ve (genellikle `CPythonNetworkStream` aracılığıyla) sunucuya gönderir. Sunucudan gelen veya istemci tarafından oluşturulan mesajlar (`chat.AppendChat`) ile ilgili sohbet pencerelerinde görüntülenir.
*   **Fısıltı Sistemi**: Oyuncular arası özel mesajlaşma arayüzü, `chat` modülünün fısıltı fonksiyonları kullanılarak oluşturulur ve yönetilir. Yeni fısıltı geldiğinde `chat.InitWhisper` ile UI'da bir buton oluşturulabilir.
*   **Eşya Paylaşımı**: Oyuncular envanterlerindeki eşyaları sohbet üzerinden paylaşmak istediklerinde, UI scripti eşyanın verilerini (VNUM, soketler, efsunlar vb.) alıp `chatGetLinkFromHyperlink` fonksiyonunun beklediği formatta bir string oluşturur ve bunu sohbet mesajına ekler. Diğer oyuncular bu linke tıkladığında, `chatGetLinkFromHyperlink` tarafından işlenerek eşya bilgisi gösterilir.
*   **Sistem Bildirimleri**: Oyun içi önemli olaylar (örn. "Bir oyuncu X eşyasını düşürdü", "Sunucu bakımı 5 dakika sonra başlayacak") `chat.AppendChat(chat.CHAT_TYPE_INFO, "Mesaj")` gibi komutlarla oyunculara bildirilir.

Bu modül, oyunun sosyal etkileşimlerinin ve bilgi akışının merkezinde yer alır ve kullanıcı arayüzünün sohbetle ilgili tüm yönlerini Python üzerinden yönetme esnekliği sunar.

---

### `PythonConfig.h` ve `PythonConfig.cpp` (Python `cfg` veya Yapılandırma Modülü)

Bu dosyalar, `ENABLE_CONFIG_MODULE` makrosu aktif olduğunda derlenen `CPythonConfig` singleton sınıfını tanımlar ve implemente eder. Bu sınıf, `minIni` kütüphanesini kullanarak `.ini` formatındaki yapılandırma dosyalarından ayar okuma ve bu dosyalara ayar yazma işlemlerini yönetir. Bu sayede oyunun çeşitli ayarları (genel, sohbet, grafik seçenekleri vb.) kalıcı olarak saklanabilir ve oyun başlangıcında yüklenebilir.

#### Dosyaların Genel Amaçları

*   **`PythonConfig.h` (Başlık Dosyası)**:
    *   `CPythonConfig` singleton sınıfının tanımını içerir.
    *   `minIni` kütüphanesinin başlık dosyasını (`minIni.h`) dahil eder.
    *   Yapılandırma dosyası içindeki bölümleri (section) temsil etmek için `EClassTypes` (örn. `CLASS_GENERAL`, `CLASS_CHAT`, `CLASS_OPTION`) enum'unu tanımlar.
    *   Yapılandırma dosyasını başlatma (`Initialize`), ayar yazma (`Write`) ve okuma (`GetString`, `GetInteger`, `GetBool`), bölüm silme (`RemoveSection`) için fonksiyon prototiplerini deklare eder.

*   **`PythonConfig.cpp` (C++ Implementasyon Dosyası)**:
    *   `CPythonConfig` sınıfının metotlarını implemente eder.
    *   `Initialize()`: Belirtilen dosya adı ile yeni bir `minIni` nesnesi oluşturur. Eğer zaten aynı dosya için bir nesne varsa işlem yapmaz, farklı bir dosya ise eskisini silip yenisini oluşturur.
    *   `GetClassNameByType()`: `EClassTypes` enum değerini `.ini` dosyasındaki karşılık gelen bölüm ismine (string) dönüştürür (örn. `CLASS_CHAT` -> "CHAT").
    *   `Write()` (çeşitli overload'lar): Belirtilen bölüme (`EClassTypes`), anahtara (`stKey`) ve değere (string, int, bool) göre ayarı `.ini` dosyasına yazar. `minIni::put()` fonksiyonunu kullanır.
    *   `GetString()`, `GetInteger()`, `GetBool()`: Belirtilen bölüm ve anahtardan ilgili türdeki değeri okur. Eğer anahtar bulunamazsa veya bir hata oluşursa varsayılan değeri (`stDefaultValue`, `iDefaultValue`, `bDefaultValue`) döndürür. `minIni::gets()`, `minIni::geti()`, `minIni::getbool()` fonksiyonlarını kullanır.
    *   `RemoveSection()`: Belirtilen bölümü ve içindeki tüm anahtarları `.ini` dosyasından siler. `minIni::del()` fonksiyonunu kullanır.
    *   Tüm okuma/yazma işlemleri öncesinde `m_pkIniFile` işaretçisinin geçerli olup olmadığını kontrol eder ve değilse hata mesajı (`TraceError`) verir.

#### Python Arayüzü (Ayrı bir modül dosyasında tanımlanır, örneğin `PythonSystemModule.cpp` içinde `systemSetting` gibi)

`CPythonConfig` sınıfının işlevleri genellikle doğrudan `config` adında bir Python modülüyle değil, daha genel bir sistem ayarları modülü (örneğin `systemSetting` veya `settings`) aracılığıyla dolaylı olarak Python'a sunulur. Bu Python fonksiyonları, `CPythonConfig::Instance()` üzerinden C++ metotlarını çağırır.

*   `systemSetting.SetFileName(fileName)`: Kullanılacak `.ini` dosyasını ayarlar (`CPythonConfig::Instance().Initialize(fileName)` çağrılır).
*   `systemSetting.SetSaveGeneral(key, value)` / `systemSetting.GetSaveGeneral(key, defaultValue)`
*   `systemSetting.SetSaveChat(key, value)` / `systemSetting.GetSaveChat(key, defaultValue)`
*   `systemSetting.SetSaveOption(key, value)` / `systemSetting.GetSaveOption(key, defaultValue)`
    *   Bu tür fonksiyonlar, `key` (string), `value` (string, int, veya bool) alır ve `CPythonConfig::Instance().Write(CPythonConfig::CLASS_GENERAL, key, value)` veya `CPythonConfig::Instance().GetString(CPythonConfig::CLASS_CHAT, key, defaultValue)` gibi uygun `CPythonConfig` metodunu çağırır.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonConfig` sistemi, oyunun çeşitli yapılandırılabilir ayarlarının kalıcı olarak saklanması ve yönetilmesi için kullanılır.

*   **Oyun Seçeneklerinin Kaydedilmesi**: Grafik ayarları (çözünürlük, gölge kalitesi, görüş mesafesi), ses ayarları (müzik, efekt sesi seviyeleri), kontrol ayarları (fare hassasiyeti, tuş atamaları) gibi oyuncunun yaptığı değişiklikler `.ini` dosyasına kaydedilir. Oyun tekrar başlatıldığında bu ayarlar okunarak uygulanır.
    *   Örnek: `systemSetting.SetSaveOption("SCREEN_WIDTH", 1920)`
*   **Sohbet Ayarlarının Kişiselleştirilmesi**: Oyuncunun farklı sohbet türleri için (genel, grup, lonca vb.) seçtiği renkler, sohbet penceresinin pozisyonu ve boyutu gibi ayarlar kaydedilebilir.
    *   Örnek: `systemSetting.SetSaveChat("NORMAL_CHAT_COLOR_R", 255)`
*   **Genel Oyun Davranışları**: Otomatik pot basma, yere düşen eşyaları otomatik toplama gibi özelliklerin aktif/pasif durumları bu sistemle saklanabilir.
    *   Örnek: `systemSetting.GetSaveGeneral("AUTO_PICKUP_ENABLED", false)`
*   **Arayüz Durumları**: Belirli UI pencerelerinin son açık kalma durumu veya pozisyonu gibi bilgiler kaydedilerek oyuncuya daha tutarlı bir deneyim sunulabilir.

Bu sistem, genellikle oyunun `system.cfg` veya benzeri bir `.ini` dosyasıyla etkileşimde bulunur. Oyuncunun yaptığı ayar değişiklikleri UI scriptleri tarafından algılanır, ilgili Python fonksiyonları çağrılarak `CPythonConfig` aracılığıyla dosyaya yazılır. Oyun başladığında ise bu ayarlar dosyadan okunarak ilgili C++ sistemlerine (örn. `CPythonSystem`, `CPythonGraphic`, `CPythonChat`) uygulanır.

---
