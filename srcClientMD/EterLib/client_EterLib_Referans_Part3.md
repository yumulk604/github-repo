# EterLib Referans Kılavuzu - Bölüm 3

Bu dosya, `EterLib` referans kılavuzunun devamıdır.

---

## Dosya Bazlı Detaylandırma (Devamı) 

### `msctf.h` (Microsoft Text Services Framework Başlığı)

*   **Amaç:** Bu dosya, Microsoft'un Text Services Framework (TSF) için COM arayüzlerini, yapılarını ve sabitlerini içeren standart bir Windows SDK başlık dosyasıdır. TSF, Windows'un gelişmiş metin girişi ve dil işleme yeteneklerini (IME'ler, konuşma tanıma, el yazısı tanıma vb.) uygulamalara sunan bir sistem bileşenidir.

*   **İçerik:**
    *   Çok sayıda COM arayüz tanımı içerir (örneğin, `ITfThreadMgr`, `ITfDocumentMgr`, `ITfContext`, `ITfKeyEventSink`, `ITfCompositionSink`, `ITfLangBarMgr`, `ITfCandidateListUIElement` vb.).
    *   TSF ile ilgili çeşitli yapılar (örneğin, `TF_LANGUAGEPROFILE`, `TF_SELECTION`, `TF_DISPLAYATTRIBUTE`), enumlar (`TfAnchor`, `TfActiveSelEnd`) ve sabitler tanımlar.
    *   Uygulamaların TSF yöneticileriyle (thread yöneticisi, belge yöneticisi, dil çubuğu yöneticisi, kategori yöneticisi vb.) etkileşim kurmasını sağlayan fonksiyonları (örneğin, `TF_CreateThreadMgr`, `TF_CreateInputProcessorProfiles`) bildirir.

*   **`EterLib` İçindeki Rolü:**
    *   Bu başlık dosyasının `EterLib` içine dahil edilmesi, Metin2 istemcisinin muhtemelen Windows'un IME (Input Method Editor) sistemini desteklediğini gösterir.
    *   Oyun içindeki metin giriş alanlarının (örneğin, sohbet penceresi), özellikle Asya dilleri (Çince, Japonca, Korece) gibi karmaşık karakter girişi gerektiren dillerde kullanıcıların işletim sistemi IME'lerini kullanarak metin girmelerine olanak tanımak için TSF arayüzlerini kullanması muhtemeldir.
    *   `EterLib` içindeki `IME.h` ve `IME.cpp` gibi dosyalar, bu `msctf.h` başlığında tanımlanan TSF arayüzlerini kullanarak IME etkileşimlerini yöneten asıl kodu içerebilir.

*   **Dokümantasyon Notu:** Bu dosya bir Windows sistem bileşeninin başlığı olduğundan ve çok kapsamlı olduğundan (12000 satırdan fazla), tüm arayüzlerin ve yapıların detaylı olarak belgelenmesi bu kılavuzun kapsamı dışındadır. Önemli olan, `EterLib`'in gelişmiş metin girişi (özellikle IME) desteği için Text Services Framework'e dayandığını anlamaktır. 

### `parser.h` ve `parser.cpp` (`script::Group` Sınıfı ve Yardımcıları)

*   **Amaç:** Bu dosyalar, basit bir komut tabanlı script veya yapılandırma formatını ayrıştırmak için `script` isim alanı altında `Group` sınıfını ve ilgili veri yapılarını (`TArg`, `TCmd`) tanımlar ve uygular. Ayrıca, yerelleştirmeye (locale) duyarlı bazı string işleme yardımcı fonksiyonları içerir.

*   **`parser.h` - Tanımlar ve Bildirimler:**
    *   **`script::TArg`**: Bir argümanı temsil eder (isim ve değer, ikisi de `std::string`).
    *   **`script::TCmd`**: Bir komutu temsil eder (isim `std::string`, argüman listesi `TArgList` - yani `std::list<TArg>`).
    *   **`script::Group` Sınıfı:**
        *   `Create(const std::string& stSource)`: Verilen string kaynağını ayrıştırır.
        *   `GetCmd(TCmd& cmd)`: Sıradaki komutu alır ve listeden çıkarır.
        *   `ReadCmd(TCmd& cmd)`: Sıradaki komutu okur ancak listeden çıkarmaz.
        *   `GetError()`: Ayrıştırma hatasını döndürür.

*   **`parser.cpp` - Uygulama Detayları:**
    *   **Yerelleştirme Yardımcıları:** `LocaleString_FindChar`, `LocaleString_RightTrim`, `LocaleString_Skip` gibi fonksiyonlar, string işlemlerini (karakter arama, boşluk temizleme, boşluk atlama) Windows kod sayfalarına ve çok baytlı karakterlere (`CharNextExA`, `CharPrevExA`) dikkat ederek yapar.
    *   **Argüman Ayrıştırma (`Group::GetArg`):**
        *   Bir komutun argüman string'ini (`arg1;değer1|arg2;değer2...`) işler.
        *   `;` karakterini isim/değer ayıracı, `|` karakterini argüman ayıracı olarak kullanır.
        *   İsim ve değerleri ayıklar, sağdaki boşlukları temizler ve `TArg` nesneleri oluşturup listeye ekler.
        *   İsim ve değer uzunluklarını kontrol eder.
    *   **Komut Ayrıştırma (`Group::Create`):**
        *   Verilen tüm metin kaynağını satır satır veya komut komut işler.
        *   Her komut için önce komut adını (ilk boşluğa kadar) ayıklar.
        *   Satırın geri kalanını `GetArg` fonksiyonuna vererek argüman listesini oluşturur.
        *   Komut adı ve argüman listesiyle bir `TCmd` nesnesi oluşturup dahili bir liste olan `m_cmdList`'e ekler.
    *   **Komut Erişimi (`GetCmd`, `ReadCmd`):**
        *   `GetCmd`, `m_cmdList`'in başındaki komutu döndürür ve listeden kaldırır.
        *   `ReadCmd`, `m_cmdList`'in başındaki komutu döndürür ancak listeden kaldırmaz.

*   **Varsayılan Script Formatı:** Bu parser'ın işlediği format genellikle şu şekildedir:
    ```
    komut_adı1 argüman_adı1;argüman_değeri1|argüman_adı2;argüman_değeri2
    komut_adı2 argüman_adı3;argüman_değeri3
    # Yorum satırları (muhtemelen desteklenmiyor veya Create içinde atlanıyor)
    komut_adı3
    ```
    *   Her satır bir komut temsil eder.
    *   İlk kelime komut adıdır.
    *   Geri kalan kısım argümanlardır.
    *   Argümanlar `isim;değer` şeklinde ve birbirlerinden `|` ile ayrılır.

*   **Kullanım Alanı:**
    *   Bu parser, basit yapılandırma dosyalarını veya sıralı komutlar içeren script dosyalarını okumak için kullanılır.
    *   Örneğin, bir UI penceresinin ilk açılış durumunu ayarlayan komutlar, bir efektin sıralı adımlarını tanımlayan komutlar veya basit görev scriptleri bu formatta olabilir.
    *   `Group::Create` ile dosya/string yüklenir ve ayrıştırılır. Ardından bir döngü içinde `Group::GetCmd` çağrılarak her komut sırayla alınır ve işlenir. Komutun adı (`cmd.name`) kontrol edilerek ne yapılacağına karar verilir ve argümanlara (`cmd.argList`) bakılarak işlem detaylandırılır. 

### `Mutex.h` ve `Mutex.cpp` (`Mutex` Sınıfı)

*   **Amaç:** Bu dosyalar, Windows'un Kritik Bölge (Critical Section) senkronizasyon mekanizmasını sarmalayan (wrap eden) basit bir `Mutex` sınıfı tanımlar ve uygular. Bu sınıf, çoklu iş parçacıklı (multi-threaded) ortamlarda paylaşılan kaynaklara erişimi kontrol etmek ve senkronize etmek için kullanılır.

*   **`Mutex.h` - Tanımlar ve Bildirimler:**
    *   `Mutex`: Kritik Bölge'yi sarmalayan sınıf.
        *   `Mutex()`: Kurucu metot. Kritik Bölge'yi başlatır.
        *   `~Mutex()`: Yıkıcı metot. Kritik Bölge'yi siler.
        *   `Lock()`: Kritik Bölge'ye girmeye çalışır (gerekirse bekler).
        *   `Unlock()`: Kritik Bölge'den çıkar.
        *   `Trylock()`: Kritik Bölge'ye girmeyi dener (beklemez). *(Not: Bu fonksiyonun implementasyonu `.cpp` dosyasında bulunmamaktadır.)*
    *   **Özel Üye:**
        *   `lock`: Windows `CRITICAL_SECTION` yapısı.

*   **`Mutex.cpp` - Uygulama Detayları:**
    *   **Windows API Kullanımı:** Sınıf, Windows API'sinin Kritik Bölge fonksiyonlarını kullanır:
        *   Kurucu: `InitializeCriticalSection(&lock)`
        *   Yıkıcı: `DeleteCriticalSection(&lock)`
        *   `Lock()`: `EnterCriticalSection(&lock)`
        *   `Unlock()`: `LeaveCriticalSection(&lock)`
    *   **Basit Sarmalayıcı:** Sınıf, Kritik Bölge kullanımını daha nesne yönelimli bir hale getirir ve RAII (Resource Acquisition Is Initialization) prensibini uygular; `Mutex` nesnesi oluşturulduğunda Kritik Bölge otomatik olarak başlatılır ve nesne yok edildiğinde otomatik olarak silinir.

*   **Kullanım Alanı:**
    *   `Mutex` sınıfı, birden fazla iş parçacığının (thread) aynı anda erişebileceği paylaşılan verilere veya kaynaklara erişimi senkronize etmek için kullanılır.
    *   Örneğin, bir veri yapısını (liste, harita vb.) birden fazla thread'in aynı anda değiştirmesini önlemek için kullanılır. Bir thread, veri yapısına erişmeden önce `mutex.Lock()` çağırır, işlemi tamamladıktan sonra `mutex.Unlock()` çağırarak diğer thread'lerin erişimine izin verir.
    *   Bu, veri bozulmalarını (data corruption) ve yarış koşullarını (race conditions) önlemeye yardımcı olur.
    *   `CThread` sınıfıyla oluşturulan iş parçacıklarının paylaşılan kaynakları güvenli bir şekilde kullanmasını sağlamak için kritik öneme sahiptir. 

### `MSApplication.h` ve `MSApplication.cpp` (`CMSApplication` Sınıfı)

*   **Amaç:** Bu dosyalar, standart bir Windows uygulamasının temel yapısını, başlatılmasını ve mesaj döngüsünü yöneten `CMSApplication` sınıfını tanımlar ve uygular. Bu sınıf, `CMSWindow`'dan miras alır.

*   **`MSApplication.h` - Tanımlar ve Bildirimler:**
    *   `CMSApplication`: Windows uygulamasının ana sınıfı.
        *   Kalıtım: `public CMSWindow`.
        *   `CMSApplication()`: Kurucu metot.
        *   `~CMSApplication()`: Yıkıcı metot.
        *   `Initialize(HINSTANCE hInstance)`: Uygulamanın instance tanıtıcısını ayarlar.
        *   `MessageLoop()`: Ana Windows mesaj döngüsünü başlatır.
        *   `IsMessage()`: Mesaj kuyruğunda bekleyen mesaj olup olmadığını kontrol eder.
        *   `MessageProcess()`: Mesaj kuyruğundan bir mesaj alır ve işler.
    *   **Korumalı Metotlar:**
        *   `ClearWindowClass()`: Pencere sınıflarını temizlemek için (kullanımda değil gibi).
        *   `WindowProcedure(HWND hWnd, UINT uiMsg, WPARAM wParam, LPARAM lParam)`: Uygulamanın ana pencere mesajlarını işleyen fonksiyon (override).

*   **`MSApplication.cpp` - Uygulama Detayları:**
    *   **Başlatma (`Initialize`):** Uygulamanın `HINSTANCE`'ını (genellikle `WinMain`'den alınır) saklar. Bu, pencere sınıfı kaydı gibi işlemler için gereklidir.
    *   **Mesaj Döngüsü (`MessageLoop`, `IsMessage`, `MessageProcess`):**
        *   `MessageProcess`, `GetMessage` ile mesaj kuyruğundan bir mesaj çeker. `GetMessage` 0 döndürdüğünde (yani `WM_QUIT` alındığında) `false` döndürür.
        *   Alınan mesajlar `TranslateMessage` (klavye mesajlarını çevirir) ve `DispatchMessage` (mesajı ilgili pencere prosedürüne gönderir) ile işlenir.
        *   `MessageLoop`, `MessageProcess` `false` döndürene kadar (yani `WM_QUIT` alınana kadar) döngüye devam eder.
        *   `IsMessage`, `PeekMessage` kullanarak mesaj kuyruğunu kontrol eder ve uygulamanın boşta kalıp kalmadığını anlamak için kullanılabilir.
    *   **Pencere Prosedürü (`WindowProcedure`):**
        *   `WM_CLOSE` mesajı (kullanıcı pencereyi kapatmaya çalıştığında gönderilir) alındığında, `PostQuitMessage(0)` fonksiyonunu çağırır. Bu, mesaj kuyruğuna `WM_QUIT` mesajını gönderir ve `MessageLoop`'un sonlanmasına neden olur.
        *   Diğer tüm mesajlar, işlenmek üzere temel sınıf olan `CMSWindow`'un `WindowProcedure` metoduna iletilir.

*   **Kullanım Alanı:**
    *   `CMSApplication`, genellikle oyunun veya uygulamanın ana giriş noktası (`WinMain`) tarafından oluşturulan ilk nesnedir.
    *   Uygulamanın Windows ile temel etkileşimini kurar: instance tanıtıcısını yönetir, ana mesaj döngüsünü çalıştırır ve uygulamanın kapatılma isteğini (`WM_CLOSE`) ele alır.
    *   Bir oyun döngüsü genellikle `IsMessage` ile mesaj olup olmadığını kontrol eder; eğer mesaj yoksa oyun mantığı ve render işlemleri yapılır, mesaj varsa `MessageProcess` çağrılarak Windows mesajları işlenir. Alternatif olarak, `MessageLoop` doğrudan kullanılabilir ve oyun mantığı/render işlemleri başka bir mekanizma (örneğin, bir zamanlayıcı veya ayrı bir thread) ile tetiklenebilir. 

### `MSWindow.h` ve `MSWindow.cpp` (`CMSWindow` Sınıfı)

*   **Amaç:** `CMSWindow` sınıfı, standart bir Windows penceresi (HWND) oluşturmak, yönetmek ve temel pencere işlemlerini (gösterme, gizleme, boyutlandırma, konumlandırma, mesaj işleme) gerçekleştirmek için bir sarmalayıcı (wrapper) sınıfı sunar. Windows API çağrılarını basitleştirir ve pencere sınıfı kaydını yönetir.

*   **`MSWindow.h` - Tanımlar ve Bildirimler:**
    *   **`CMSWindow` Sınıfı:**
        *   Kurucu (`CMSWindow()`), Yıkıcı (`~CMSWindow()`).
        *   `Destroy()`: Pencereyi yok eder.
        *   `Create(const char* c_szName, int brush, DWORD cs, DWORD ws, HICON hIcon, int iCursorResource)`: Yeni bir pencere oluşturur. Parametreler pencere başlığını, arka plan fırçasını, sınıf stilini (class style), pencere stilini (window style), ikonunu ve fare imlecini belirler.
        *   `Show()`, `Hide()`, `SetVisibleMode(bool isVisible)`: Pencerenin görünürlüğünü yönetir.
        *   `SetPosition(int x, int y)`, `SetCenterPosition()`: Pencere konumunu ayarlar.
        *   `SetText(const char* c_szText)`: Pencere başlığını ayarlar.
        *   `AdjustSize(int width, int height)`: İstemci alanının (client area) belirtilen boyutta olmasını sağlayacak şekilde pencere boyutunu ayarlar (kenarlıklar ve başlık çubuğu dahil).
        *   `SetSize(int width, int height)`: Pencerenin toplam boyutunu ayarlar.
        *   `IsVisible()`, `IsActive()`: Pencerenin görünürlük ve aktiflik durumunu sorgular.
        *   `GetMousePosition(POINT* ppt)`: Fare imlecinin pencereye göre istemci koordinatlarını alır.
        *   `GetClientRect(RECT* prc)`, `GetWindowRect(RECT* prc)`: İstemci alanı ve pencere dikdörtgenlerini alır.
        *   `GetScreenWidth()`, `GetScreenHeight()`: Ana ekranın çözünürlüğünü alır.
        *   `GetWindowHandle()`: Pencerenin `HWND`'ını döndürür.
        *   `GetInstance()`: Uygulamanın `HINSTANCE`'ını döndürür (statik üye).
        *   `WindowProcedure(HWND hWnd, UINT uiMsg, WPARAM wParam, LPARAM lParam)`: Sanal pencere mesaj işleme fonksiyonu. Temel mesajları (örneğin, `WM_SIZE`, `WM_ACTIVATEAPP`) işler ve diğerlerini `DefWindowProc`'a iletir.
        *   `OnSize(WPARAM wParam, LPARAM lParam)`: `WM_SIZE` mesajı için sanal işleyici.
    *   **Korumalı Metotlar:**
        *   `RegisterWindowClass(...)`: Belirtilen parametrelerle bir Windows pencere sınıfını kaydeder veya zaten kayıtlıysa mevcut olanı kullanır. Benzersiz bir sınıf adı üretir ve `ms_stWCSet` içinde takip eder.
    *   **Üyeler:**
        *   `m_hWnd`: Pencerenin `HWND`'ı.
        *   `m_rect`: Genellikle boyutlandırma için kullanılan bir `RECT`.
        *   `m_isActive`, `m_isVisible`: Pencerenin aktiflik ve görünürlük bayrakları.
    *   **Statik Üyeler:**
        *   `ms_stWCSet` (`TWindowClassSet` - `std::set<char*, stl_sz_less>`): Kayıtlı pencere sınıfı adlarını tutan bir set.
        *   `ms_hInstance`: Uygulamanın `HINSTANCE`'ı. `CMSApplication` tarafından set edilmesi beklenir.

*   **`MSWindow.cpp` - Uygulama Detayları:**
    *   **Global Pencere Prosedürü (`MSWindowProcedure`):**
        *   Bu, Windows API'sine `WNDCLASS` kaydı sırasında verilen asıl callback fonksiyonudur.
        *   `GetWindowLongPtr(hWnd, GWLP_USERDATA)` ile `CMSWindow` örneğini alır (pencere oluşturulurken `SetWindowLongPtr` ile ayarlanır).
        *   Mesajı ilgili `CMSWindow` örneğinin `WindowProcedure` sanal metoduna yönlendirir.
    *   **`CMSWindow::WindowProcedure`:**
        *   `WM_SIZE`: `OnSize` metodunu çağırır.
        *   `WM_ACTIVATEAPP`: `m_isActive` bayrağını günceller.
        *   Diğer mesajlar `DefWindowProc` ile varsayılan işleme bırakılır.
    *   **`CMSWindow::OnSize`:**
        *   Pencere küçültüldüğünde (`SIZE_MINIMIZED`), `m_isActive` ve `m_isVisible`'ı `false` yapar.
        *   Diğer durumlarda (`SIZE_MAXIMIZED`, `SIZE_RESTORED` vb.) `true` yapar.
    *   **`CMSWindow::Create`:**
        *   Mevcut bir pencere varsa `Destroy()` eder.
        *   `RegisterWindowClass` ile pencere sınıfını kaydeder (veya mevcut olanı bulur).
        *   `CreateWindowEx` (veya `CreateWindow`) ile pencereyi oluşturur.
        *   `SetWindowLongPtr` ile `this` işaretçisini pencerenin kullanıcı verisi olarak saklar, böylece `MSWindowProcedure` doğru sınıf örneğini bulabilir.
    *   **`CMSWindow::RegisterWindowClass`:**
        *   Verilen stil, fırça ve prosedürden benzersiz bir sınıf adı oluşturur (örneğin, "eter - s%x:b%x:p:%x").
        *   Bu sınıf adının `ms_stWCSet` içinde olup olmadığını kontrol eder. Varsa, mevcut kayıtlı adı döndürür.
        *   Yoksa, yeni bir `WNDCLASS` yapısı doldurur (ikon, imleç, arka plan fırçası vb. ayarlar) ve `RegisterClass` ile Windows'a kaydeder. Yeni sınıf adını `ms_stWCSet`'e ekler ve döndürür.
    *   **Diğer Metotlar:** Çoğunlukla ilgili Windows API fonksiyonlarını (`ShowWindow`, `SetWindowPos`, `GetClientRect`, `GetCursorPos`, `ScreenToClient` vb.) sarmalar ve sınıfın `m_hWnd`, `m_isVisible`, `m_isActive` gibi üyelerini günceller.
    *   `SetCenterPosition`: Pencerenin istemci alanının boyutlarını alır ve ekranın ortasına konumlandırır.
    *   `AdjustSize`: İstenen istemci alanı boyutuna ulaşmak için `AdjustWindowRectEx` kullanarak pencerenin toplam boyutunu hesaplar ve `MoveWindow` ile ayarlar.

*   **Kullanım Alanı:**
    *   `CMSWindow`, `EterLib` ve onu kullanan uygulamalarda (özellikle oyun istemcisi gibi) temel pencere oluşturma ve yönetimi için bir alt yapı sağlar.
    *   `CMSApplication` sınıfı, uygulamanın ana penceresi için `CMSWindow`'dan miras alır.
    *   Oyunun render penceresi veya hata ayıklama, araç pencereleri gibi yardımcı pencereler oluşturmak için kullanılabilir.
    *   Windows API'sinin karmaşıklığını bir ölçüde gizleyerek daha nesne yönelimli bir pencere yönetimi sunar.
    *   Pencere sınıfı kaydını otomatikleştirir ve aynı özelliklere sahip sınıfların tekrar tekrar kaydedilmesini önler. 

### `GrpMath.h` ve `GrpMath.cpp` (Grafik Matematik Yardımcıları)

*   **Amaç:** Bu dosyalar, `EterLib` içinde çeşitli 2D ve 3D matematiksel işlemler, özellikle de dönüşümler (rotasyon, öteleme), matris işlemleri ve geometrik testler için bir dizi yardımcı fonksiyon tanımlar ve uygular.

*   **`GrpMath.h` - Fonksiyon Bildirimleri ve Inline Fonksiyonlar:**
    *   `CrossProduct2D(float x1, float y1, float x2, float y2)`: İki 2D vektörün cross product (vektörel çarpımının z bileşeni gibi düşünülebilir) değerini hesaplar. Genellikle bir noktanın bir doğruya göre hangi tarafta olduğunu veya iki vektör arasındaki yönü belirlemek için kullanılır.
    *   `IsInTriangle2D(float ax, float ay, ..., tx, ty)`: Bir noktanın (t) bir 2D üçgenin (a, b, c) içinde olup olmadığını kontrol eder. Barycentric koordinatlar veya cross product temelli bir yöntem kullanır.
    *   `D3DXVec3Rotation(D3DXVECTOR3* pvtOut, const D3DXVECTOR3* c_pvtSrc, const D3DXQUATERNION* c_pqtRot)`: Bir 3D vektörü (`c_pvtSrc`) verilen bir quaternion (`c_pqtRot`) ile döndürür ve sonucu `pvtOut`'a yazar.
    *   `D3DXVec3Translation(D3DXVECTOR3* pvtOut, const D3DXVECTOR3* c_pvtSrc, const D3DXVECTOR3* c_pvtTrans)`: Bir 3D vektörünü (`c_pvtSrc`) verilen bir öteleme vektörü (`c_pvtTrans`) kadar öteler (aslında sadece toplama işlemi yapar, ismi biraz yanıltıcı olabilir).
    *   `GetRotationFromMatrix(D3DXVECTOR3* pRotation, const D3DXMATRIX* c_pMatrix)`: Bir 3x3 veya 4x4 rotasyon matrisinden Euler açılarını (X, Y, Z eksenleri etrafındaki dönüşler) çıkarır.
    *   `GetPivotAndRotationFromMatrix(D3DXMATRIX* pMatrix, D3DXVECTOR3* pPivot, D3DXVECTOR3* pRotation)`: Bir 4x4 dönüşüm matrisinden hem pivot (öteleme/pozisyon) vektörünü hem de rotasyon Euler açılarını çıkarır.
    *   `ExtractMovement(D3DXMATRIX* pTargetMatrix, D3DXMATRIX* pSourceMatrix)`: Kaynak matrisin (`pSourceMatrix`) rotasyon ve öteleme bileşenlerini çıkarıp bu bileşenlerden yeni bir dönüşüm matrisi oluşturarak `pTargetMatrix`'e yazar.
    *   **Inline Fonksiyonlar:**
        *   `D3DXVec3Blend`: İki 3D vektör arasında lineer interpolasyon yapar.
        *   `D3DXQuaternionBlend`: İki quaternion arasında lineer interpolasyon yapar (normalize edilmemiş).
        *   `ClampDegree`: Bir açıyı [0, 360) derece aralığına sıkıştırır.
        *   `GetVector3Distance`: İki 3D vektör arasındaki mesafenin karesini hesaplar (2D mesafe gibi, z bileşeni yok sayılır - bu muhtemelen bir hata veya özel bir kullanım içindir, fonksiyon adı yanıltıcı).
        *   `SafeRotationNormalizedArc`: İki normalize edilmiş vektör (vFrom, vTo) arasındaki en kısa dönüşü temsil eden bir quaternion oluşturur. Paralel ve anti-paralel durumları özel olarak ele alır.
        *   `RotationNormalizedArc`: `SafeRotationNormalizedArc` ile aynı işlevi görür ancak özel durum kontrolleri yoktur. Paralel veya anti-paralel vektörlerde tanımsız davranışlara yol açabilir.
        *   `RotationArc`: İki vektör (normalize olmaları gerekmez) arasındaki en kısa dönüşü temsil eden bir quaternion oluşturur. Önce vektörleri normalize eder, sonra `RotationNormalizedArc` çağırır.
        *   `square_distance_between_linesegment_and_point`: Bir nokta ile bir doğru parçası arasındaki en kısa mesafenin karesini hesaplar.
        *   `Vec3TransformQuaternionSafe`, `Vec3TransformQuaternion`: Bir 3D vektörü bir quaternion ile döndürür. (Implementationları biraz farklı görünüyor ama amaçları aynı gibi. `Safe` versiyonu geçici bir değişken kullanıyor.)

*   **`GrpMath.cpp` - Uygulama Detayları:**
    *   **`CrossProduct2D`:** Basitçe `x1*y2 - y1*x2` formülünü uygular.
    *   **`IsInTriangle2D`:** Üç kenar vektörü ile test noktasından köşelere giden vektörlerin cross product'larını hesaplar. Eğer tüm cross product'ların işaretleri aynıysa (veya biri sıfırsa ve nokta segment üzerindeyse) nokta üçgenin içindedir.
    *   **`D3DXVec3Rotation`:** Kaynak vektörü bir quaternion (`qtSrc`) olarak temsil eder (w=0). Ardından `qtRet = c_pqtRot * qtSrc * conjugate(c_pqtRot)` formülünü kullanarak döndürülmüş vektörü hesaplar. Sonucun x, y, z bileşenlerini `pvtOut`'a yazar.
    *   **`GetRotationFromMatrix`:** Matris elemanlarını kullanarak `atan2f` fonksiyonları aracılığıyla X, Y ve Z eksenleri etrafındaki dönüş açılarını (Euler açıları) hesaplar. Gimbal lock (singularity) durumu için özel kontrol içerir (cx ~ 0 olduğunda).
    *   **`GetPivotAndRotationFromMatrix`:** Önce `GetRotationFromMatrix` gibi rotasyon açılarını hesaplar. Ardından matrisin son satırındaki öteleme değerlerini (`_41`, `_42`, `_43`) `pPivot` vektörüne atar.
    *   **`ExtractMovement`:** `GetPivotAndRotationFromMatrix` kullanarak rotasyon ve öteleme bilgilerini alır. Ardından `D3DXMatrixRotationX/Y/Z` ve `D3DXMatrixTranslation` fonksiyonları ile ayrı ayrı rotasyon ve öteleme matrisleri oluşturur. Son olarak bu matrisleri çarparak sadece hareket bilgisini içeren bir matrisi `pTargetMatrix`'e yazar (orijinal matrisin ölçekleme veya shear gibi diğer bileşenlerini atar).

*   **Kullanım Alanı:**
    *   Bu fonksiyonlar, oyun içindeki nesnelerin konumlandırılması, döndürülmesi, animasyonu ve çarpışma tespiti gibi birçok temel grafik ve oyun mekaniği hesaplamasında kullanılır.
    *   Vektör ve quaternion dönüşümleri, 3D modellerin ve kameraların yönelimini ayarlamak için kritik öneme sahiptir.
    *   Matrislerden rotasyon ve pozisyon bilgilerini çıkarma, animasyon verilerini veya nesne hiyerarşilerini işlerken gereklidir.
    *   2D kesişim testleri, UI elemanları veya 2D projeksiyonlarla yapılan bazı geometrik sorgulamalarda kullanılabilir.
    *   Doğru parçası-nokta mesafe hesaplaması, çarpışma tespiti veya AI davranışları için faydalıdır. 

### `GrpObjectInstance.h` ve `GrpObjectInstance.cpp` (`CGraphicObjectInstance` Sınıfı)

*   **Amaç:** `CGraphicObjectInstance` sınıfı, oyun dünyasındaki çeşitli grafiksel nesnelerin (örneğin, karakterler, binalar, efektler) temel özelliklerini ve davranışlarını yöneten soyut bir temel sınıftır. Pozisyon, rotasyon, ölçek, görünürlük, çarpışma, culling (görünürlük tespiti), yükseklik haritası etkileşimi ve render işlemleri gibi ortak işlevleri içerir.

*   **`GrpObjectInstance.h` - Tanımlar, Bildirimler ve Enum'lar:**
    *   **Enum (İsimsiz - Nesne Türleri):** `THING_OBJECT`, `TREE_OBJECT`, `ACTOR_OBJECT`, `EFFECT_OBJECT`, `DUNGEON_OBJECT` gibi nesne türlerini tanımlayan sabitler. Muhtemelen RTTI (Runtime Type Information) veya nesne filtreleme için kullanılır.
    *   **Enum (İsimsiz - Portal):** `PORTAL_ID_MAX_NUM`. Bir nesnenin ilişkilendirilebileceği maksimum portal sayısını tanımlar.
    *   **`CGraphicObjectInstance` Sınıfı:**
        *   **Kalıtım:** `public CGraphicCollisionObject`. (Bu, nesnenin çarpışma algılama yeteneklerine sahip olduğunu gösterir.)
        *   **Kurucu ve Yıkıcı:** `CGraphicObjectInstance()`, `virtual ~CGraphicObjectInstance()`.
        *   **Soyut Metotlar:**
            *   `virtual int GetType() const = 0`: Türetilmiş sınıfın nesne türünü (yukarıdaki enum'lardan biri) döndürmesini zorunlu kılar.
            *   `virtual bool GetBoundingSphere(D3DXVECTOR3& v3Center, float& fRadius) = 0`: Nesnenin sınırlayıcı küresinin (bounding sphere) merkezini ve yarıçapını döndürmelidir. Culling için kullanılır.
            *   `virtual void OnRender() = 0`: Nesnenin ana render işlemini gerçekleştirir.
            *   `virtual void OnBlendRender() = 0`: Nesnenin alfa karıştırmalı (alpha blended) render işlemini gerçekleştirir (genellikle saydam veya yarı saydam nesneler için).
            *   `virtual void OnRenderToShadowMap() = 0`: Nesnenin gölge haritasına render işlemini gerçekleştirir.
            *   `virtual void OnRenderShadow() = 0`: Nesnenin ürettiği gölgeyi render eder.
            *   `virtual void OnRenderPCBlocker() = 0`: Nesnenin PC engelleyici (blocker) geometrisini render eder (muhtemelen oyuncuların içinden geçemeyeceği alanları belirtmek için).
            *   `virtual void OnUpdateCollisionData(...) = 0`: Çarpışma verisi güncellendiğinde çağrılır.
            *   `virtual void OnUpdateHeighInstance(...) = 0`: Yükseklik örneği güncellendiğinde çağrılır.
            *   `virtual bool OnGetObjectHeight(...) = 0`: Belirli bir X,Y koordinatındaki nesne yüksekliğini döndürür.
        *   **Sanal Metotlar (Varsayılan Implementasyonlu):** `OnInitialize`, `OnClear`, `OnUpdate`, `OnDeform`.
        *   **Temel Dönüşüm (Transform) Metotları:**
            *   `GetPosition`, `GetScale`, `GetRotation`, `GetYaw`, `GetPitch`, `GetRoll`.
            *   `SetPosition`, `SetScale`, `SetRotation` (float veya Yaw/Pitch/Roll), `SetRotationQuaternion`, `SetRotationMatrix`.
        *   **Genel Yaşam Döngüsü ve Render Metotları:**
            *   `Clear()`: Nesneyi başlangıç durumuna sıfırlar.
            *   `Update()`: Nesnenin durumunu günceller (animasyon, mantık vb.).
            *   `Render()`: Ana render metodunu çağırır (`OnRender`).
            *   `BlendRender()`: Blend render metodunu çağırır (`OnBlendRender`).
            *   `RenderToShadowMap()`, `RenderShadow()`, `RenderPCBlocker()`: İlgili özel render metotlarını çağırır.
            *   `Deform()`: Nesne geometrisini deforme eder (örneğin, animasyon için vertex blending).
            *   `Transform()`: Pozisyon, rotasyon ve ölçek bilgilerine dayanarak `m_worldMatrix`'i hesaplar.
        *   **Görünürlük:** `Show()`, `Hide()`, `isShow()`.
        *   **Kamera Engelleme:** `BlockCamera()`, `SetBlockCamera(bool)`.
        *   **Işın Testi (Ray Test):** `isIntersect(const CRay& c_rRay, ...)`: Verilen bir ışının nesneyle kesişip kesişmediğini kontrol eder.
        *   **Sınırlayıcı Kutu (Bounding Box):** Dünya koordinatlarındaki (`GetWTBBoxVertex`) ve yerel koordinatlardaki (`GetTBBoxMin/Max`, `GetBBoxMin/Max`) sınırlayıcı kutu bilgilerini döndürür.
        *   **Matrisler:** `GetTransform()` (`m_worldMatrix`), `GetWorldMatrix()`.
        *   **Portal Yönetimi:** `SetPortal(DWORD dwIndex, int iID)`, `GetPortal(DWORD dwIndex)`.
        *   **Culling:** `UpdateBoundingSphere()`, `RegisterBoundingSphere()`: Nesnenin sınırlayıcı küresini günceller ve culling yöneticisine kaydeder.
        *   **Statik Çarpışma:** `AddCollision`, `ClearCollision`, `CollisionDynamicSphere`, `MovementCollisionDynamicSphere`, `GetCollisionMovementAdjust`, `UpdateCollisionData`. Statik çarpışma geometrileri eklemeyi ve dinamik kürelerle çarpışma testleri yapmayı sağlar.
        *   **Yükseklik Verisi:** `SetHeightInstance`, `ClearHeightInstance`, `UpdateHeightInstance`, `IsObjectHeight`, `GetObjectHeight`. Nesnenin arazi yükseklik verisiyle etkileşimini yönetir.
        *   **Acce (Aksesuar?) Ölçekleme:** `SetScaleWorld`, `SetAcceScale`. Muhtemelen giyilebilir aksesuarların ölçeklendirilmesiyle ilgili.
    *   **Üyeler:**
        *   `m_v3Position`, `m_v3Scale`: Nesnenin pozisyonu ve ölçeği.
        *   `m_fYaw`, `m_fPitch`, `m_fRoll`: Euler açıları (rotasyon).
        *   `m_mRotation`: Rotasyon matrisi.
        *   `m_isVisible`: Görünürlük bayrağı.
        *   `m_worldMatrix`: Nihai dünya dönüşüm matrisi.
        *   `m_BlockCamera`: Kamerayı engelleyip engellemediği.
        *   `m_v4TBBox`, `m_v3TBBoxMin/Max`, `m_v3BBoxMin/Max`: Sınırlayıcı kutu verileri.
        *   `m_abyPortalID`: Portal kimlikleri dizisi.
        *   `m_CullingHandle`: Culling yöneticisindeki tanıtıcı.
        *   `m_StaticCollisionInstanceVector`: Nesneye ait statik çarpışma örnekleri.
        *   `m_pHeightAttributeInstance`: Yükseklik verisi örneği.
        *   `m_bAttachedAcceRace`, `m_v3ScaleAcce`, `m_matAbsoluteTrans`, `m_matScale`, `m_matScaleWorld`: Aksesuar ölçekleme ile ilgili üyeler.

*   **`GrpObjectInstance.cpp` - Uygulama Detayları:**
    *   **Başlatma (`OnInitialize`, `Clear`):** `Clear` metodu, nesnenin pozisyonunu, rotasyonunu, ölçeğini varsayılan değerlere ayarlar, görünürlüğü `true` yapar, culling handle'ını temizler, portal ID'lerini sıfırlar ve türetilmiş sınıfın `OnClear` metodunu çağırır.
    *   **Render Akışı:** Ana `Render`, `BlendRender`, `RenderToShadowMap` gibi metotlar, öncelikle `isShow()` ile görünürlük kontrolü yapar ve ardından ilgili `On...` sanal metodunu çağırarak asıl çizim işini türetilmiş sınıfa bırakır.
    *   **Güncelleme Akışı (`Update`, `Deform`, `Transform`):**
        *   `Update`, `OnUpdate` sanal metodunu çağırır ve ardından `UpdateBoundingSphere` ile culling için sınırlayıcı küreyi günceller.
        *   `Deform`, görünürse `OnDeform`'u çağırır.
        *   `Transform`, `m_mRotation` ve `m_v3Position`'a göre `m_worldMatrix`'i hesaplar (ölçekleme `m_matScaleWorld` ile önceden uygulanmış gibi görünüyor).
    *   **Dönüşüm Ayarları:** `SetRotation` metotları D3DX matris fonksiyonlarını (`D3DXMatrixRotationZ`, `D3DXMatrixRotationYawPitchRoll`, `D3DXMatrixRotationQuaternion`) kullanarak `m_mRotation` matrisini günceller.
    *   **Işın Testi (`isIntersect`):** Bu fonksiyonun implementasyonu genellikle türetilmiş sınıflarda (örneğin, model verisine erişerek) yapılır. Temel sınıfın implementasyonu eksiktir veya sadece bir prototiptir.
    *   **Sınırlayıcı Kutu:** Yerel ve dünya koordinatlarındaki kutu bilgileri genellikle `OnUpdate` veya `Transform` içinde türetilmiş sınıf tarafından güncellenir.
    *   **Çarpışma (`AddCollision`, `CollisionDynamicSphere` vb.):** `m_StaticCollisionInstanceVector` içindeki çarpışma örnekleri üzerinde döngüye girerek testleri gerçekleştirir.
    *   **Yükseklik (`GetObjectHeight`):** `m_pHeightAttributeInstance` geçerliyse, türetilmiş sınıfın `OnGetObjectHeight` metodunu çağırır.

*   **Kullanım Alanı:**
    *   `CGraphicObjectInstance`, oyun dünyasında yer alan ve render edilen hemen hemen tüm nesneler (karakterler, binalar, efektler, ağaçlar vb.) için bir temel teşkil eder.
    *   Nesnelerin temel 3D özelliklerini (pozisyon, rotasyon, ölçek) ve durumlarını (görünürlük) yönetir.
    *   Render, güncelleme, deformasyon, çarpışma gibi temel işlevler için sanal bir arayüz tanımlar ve türetilmiş sınıfların bu işlevleri kendi nesne türlerine göre özelleştirmesini sağlar.
    *   Culling, portal sistemi, yükseklik haritası etkileşimi gibi sistemlerle entegrasyonu sağlar.
    *   Oyun motorunun nesne yönetimi ve render boru hattının (pipeline) temel bir parçasıdır. 

### `GrpPixelShader.h` ve `GrpPixelShader.cpp` (`CPixelShader` Sınıfı)

*   **Amaç:** Bu dosyalar, Direct3D 8 için pixel shader (piksel gölgelendirici) programlarını yönetmek üzere `CPixelShader` sınıfını tanımlar ve uygular. Pixel shader'lar, bir nesnenin yüzeyindeki her bir pikselin nihai rengini hesaplamak için kullanılan küçük programlardır ve gelişmiş aydınlatma, doku karıştırma ve diğer görsel efektleri oluşturmak için kullanılırlar.

*   **`GrpPixelShader.h` - Tanımlar ve Bildirimler:**
    *   **`CPixelShader` Sınıfı:**
        *   **Kalıtım:** `public CGraphicBase`. (Ancak `CGraphicBase`'in statik üyelerini kullanır, doğrudan bir bağlantısı yok gibi görünüyor).
        *   Kurucu (`CPixelShader()`), Yıkıcı (`~CPixelShader()`).
        *   `Destroy()`: Oluşturulan pixel shader'ı siler.
        *   `CreateFromDiskFile(const char* c_szFileName)`: Belirtilen dosyadan (genellikle `.psh` veya benzeri assembly formatında) bir pixel shader derler ve oluşturur.
        *   `Set()`: Bu pixel shader'ı Direct3D render pipeline'ı için aktif hale getirir (`STATEMANAGER` aracılığıyla).
    *   **Korumalı Metot:**
        *   `Initialize()`: Üyeleri (özellikle `m_handle`) başlangıç durumuna getirir.
    *   **Üye:**
        *   `m_handle`: Oluşturulan Direct3D pixel shader'ının tanıtıcısı (`DWORD`).

*   **`GrpPixelShader.cpp` - Uygulama Detayları:**
    *   **Başlatma ve Yok Etme:**
        *   Kurucu `Initialize()`'ı çağırır, `m_handle`'ı 0 yapar.
        *   Yıkıcı `Destroy()`'ı çağırır.
        *   `Destroy()`, eğer `m_handle` geçerliyse ve Direct3D cihazı (`ms_lpd3dDevice`) mevcutsa, `ms_lpd3dDevice->DeletePixelShader(m_handle)` çağırarak Direct3D kaynağını serbest bırakır.
    *   **Oluşturma (`CreateFromDiskFile`):**
        1.  Varsa mevcut shader'ı `Destroy()` ile siler.
        2.  `D3DXAssembleShaderFromFile` fonksiyonunu kullanarak verilen dosyadaki pixel shader assembly kodunu derler. Derlenmiş kod `lpd3dxShaderBuffer` içine, hatalar (varsa) `lpd3dxErrorBuffer` içine yazılır.
        3.  Eğer derleme başarısız olursa (`FAILED`) `false` döner.
        4.  `CDirect3DXBuffer` (muhtemelen `lpd3dxBuffer`'ı sarmalayan RAII bir sınıf) ile buffer yönetimini kolaylaştırır.
        5.  `ms_lpd3dDevice->CreatePixelShader` fonksiyonunu çağırarak derlenmiş shader kodundan (`(DWORD*)shaderBuffer.GetPointer()`) bir Direct3D pixel shader nesnesi oluşturur ve tanıtıcısını (`m_handle`) alır.
        6.  Oluşturma başarısız olursa `false` döner.
        7.  Başarılı olursa `true` döner.
    *   **Aktif Etme (`Set`):**
        *   `STATEMANAGER.SetPixelShader(m_handle)` çağrısını yaparak bu shader'ı render pipeline'ı için aktif pixel shader olarak ayarlar.

*   **Kullanım Alanı:**
    *   `CPixelShader` sınıfı, özel pixel tabanlı efektler uygulamak için kullanılır.
    *   Genellikle bir materyalin veya özel bir render geçişinin parçası olarak kullanılır. Örneğin, belirli bir nesne türü için özel bir aydınlatma modeli, doku karıştırma efekti veya post-processing efekti uygulamak amacıyla bir pixel shader dosyası (`.psh`) oluşturulur.
    *   Bu dosya `CreateFromDiskFile` ile yüklenir ve bir `CPixelShader` nesnesi oluşturulur.
    *   Render sırasında, ilgili nesneler çizilmeden önce `CPixelShader::Set()` metodu çağrılarak bu özel pixel shader aktif hale getirilir. Ardından çizim yapılır ve gerekirse varsayılan shader'a geri dönülür. 

    ### `GrpRatioInstance.h` ve `GrpRatioInstance.cpp` (`CGraphicRatioInstance` Sınıfı)

*   **Amaç:** Bu dosyalar, genellikle 0.0 ile 1.0 arasında bir değeri (oranı) temsil eden ve bu değerin zaman içinde bir başlangıç değerinden hedef değere yumuşak bir şekilde geçiş yapmasını (blend/interpolate) sağlayan `CGraphicRatioInstance` sınıfını tanımlar ve uygular. Bu, genellikle animasyonları veya efektleri karıştırmak (blend etmek) için kullanılır.

*   **`GrpRatioInstance.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicRatioInstance` Sınıfı:**
        *   Kurucu (`CGraphicRatioInstance()`), Yıkıcı (`~CGraphicRatioInstance()`).
        *   `Clear()`: Oranı ve zamanlama bilgilerini sıfırlar.
        *   `SetRatioReference(const float& ratio)`: Mevcut oranı anında belirtilen değere ayarlar (başlangıç ve hedef oranı da bu değere eşitler).
        *   `BlendRatioReference(DWORD blendTime, const float& ratio)`: Belirtilen `blendTime` (milisaniye cinsinden) süresi içinde mevcut orandan hedef `ratio` değerine doğru bir geçiş başlatır.
        *   `Update()`: Geçiş (blend) işlemini mevcut zamana göre günceller.
        *   `GetCurrentRatioReference() const`: O anki (interpole edilmiş) oranı döndürür.
    *   **Korumalı Metot:**
        *   `GetTime()`: Mevcut zamanı milisaniye cinsinden alır (genellikle `CTimer::Instance().GetCurrentMillisecond()`).
    *   **Üyeler:**
        *   `m_curRatio`: O anki hesaplanmış oran değeri.
        *   `m_srcRatio`: Geçişin başlangıç oranı değeri.
        *   `m_dstRatio`: Geçişin hedef oranı değeri.
        *   `m_baseTime`: Geçişin başladığı zaman damgası.
        *   `m_blendTime`: Geçişin tamamlanması için gereken toplam süre.

*   **`GrpRatioInstance.cpp` - Uygulama Detayları:**
    *   **Başlatma (`Clear`):** Tüm oranları (`m_curRatio`, `m_srcRatio`, `m_dstRatio`) 0.0f'a ve zamanları (`m_baseTime`, `m_blendTime`) 0'a ayarlar.
    *   **Anında Ayarlama (`SetRatioReference`):** Mevcut, başlangıç ve hedef oranları doğrudan verilen değere eşitler. Herhangi bir geçiş olmaz.
    *   **Geçiş Başlatma (`BlendRatioReference`):**
        *   Geçerli zamanı `m_baseTime` olarak kaydeder.
        *   İstenen geçiş süresini `m_blendTime` olarak kaydeder.
        *   O anki `m_curRatio` değerini `m_srcRatio` olarak saklar.
        *   Yeni hedef oranı `m_dstRatio` olarak ayarlar.
    *   **Güncelleme (`Update`):**
        *   Geçerli zaman ile başlangıç zamanı arasındaki farkı (`elapsedTime`) hesaplar.
        *   Eğer geçen süre (`elapsedTime`) toplam geçiş süresinden (`m_blendTime`) küçükse:
            *   Başlangıç ve hedef arasındaki farkı (`diff`) hesaplar.
            *   Geçişin ne kadarının tamamlandığını (`rate = elapsedTime / m_blendTime`) hesaplar.
            *   Lineer interpolasyon formülü (`m_curRatio = diff * rate + m_srcRatio`) kullanarak `m_curRatio`'yu günceller.
        *   Eğer geçen süre geçiş süresine eşit veya büyükse, `m_curRatio`'yu doğrudan `m_dstRatio`'ya eşitler (geçiş tamamlanmıştır).
    *   **Zaman Alma (`GetTime`):** `CTimer::Instance().GetCurrentMillisecond()` çağırarak sistem zamanını alır.

*   **Kullanım Alanı:**
    *   `CGraphicRatioInstance`, genellikle iki durum arasında yumuşak bir geçiş gerektiren durumlarda kullanılır. Örneğin:
        *   **Animasyon Karıştırma (Blending):** İki farklı animasyon arasında yumuşak bir geçiş yapmak için. `m_curRatio`, ikinci animasyonun ne kadar ağırlıkla karıştırılacağını belirleyebilir (0.0 = sadece ilk animasyon, 1.0 = sadece ikinci animasyon).
        *   **Efekt Yoğunluğu:** Bir efektin (örneğin, parlama, renk değişimi) yoğunluğunu zamanla artırmak veya azaltmak için.
        *   **LOD (Level of Detail) Geçişleri:** Farklı detay seviyesindeki modeller arasında yumuşak bir geçiş (cross-fade) yapmak için.
    *   Bir işlem başlatıldığında `BlendRatioReference` çağrılarak hedef oran ve geçiş süresi belirlenir. Ardından her frame'de `Update()` çağrılarak `m_curRatio` güncellenir ve bu oran değeri, ilgili işlemde (animasyon karıştırma, renk interpolasyonu vb.) kullanılır.
    
    ### `GrpScreen.h` ve `GrpScreen.cpp` (`CScreen` Sınıfı)

*   **Amaç:** `CScreen` sınıfı, 2D ve 3D temel geometrik şekillerin (çizgi, kutu, küp, küre, silindir vb.) çizilmesi, ekran temizleme işlemleri, Direct3D cihaz yönetimi (kayıp cihazı geri yükleme), projeksiyon/ters projeksiyon işlemleri (dünya koordinatlarından ekran koordinatlarına ve tersi) ve view frustum (görüş hacmi) yönetimi gibi çeşitli ekran ve çizimle ilgili işlevleri bir araya getirir. Debug çizimleri, basit UI elemanları veya özel efektler için kullanılabilir.

*   **`GrpScreen.h` - Tanımlar ve Bildirimler:**
    *   **Kalıtım:** `public CGraphicCollisionObject`. Bu, ekranın kendisinin de bir çarpışma nesnesi olabileceğini veya çarpışma ile ilgili bazı temel özellikleri miras aldığını gösterir.
    *   **`CScreen` Sınıfı:**
        *   Kurucu (`CScreen()`), Yıkıcı (`~CScreen()`).
        *   **Ekran Temizleme ve Gösterme:**
            *   `ClearDepthBuffer()`: Sadece derinlik tamponunu temizler.
            *   `Clear()`: Renk, derinlik ve stencil tamponlarını `ms_clearColor`, `ms_clearDepth`, `ms_clearStencil` ile temizler.
            *   `Begin()`: `CGraphicBase::Begin()` çağırır (muhtemelen `BeginScene` içerir).
            *   `End()`: `CGraphicBase::End()` çağırır (muhtemelen `EndScene` içerir).
            *   `Show(HWND hWnd = NULL)`: Render edilmiş sahneyi belirtilen pencereye (veya varsayılan pencereye) sunar (`Present`).
            *   `Show(RECT* pSrcRect)`: Kaynak dikdörtgeni kullanarak sunar.
            *   `Show(RECT* pSrcRect, HWND hWnd)`: Kaynak dikdörtgen ve hedef pencereyi kullanarak sunar.
        *   **2D Çizim Metotları:** (Genellikle 3D versiyonlarının Z değerini sabit tutan sarmalayıcılarıdır)
            *   `RenderLine2d`, `RenderBox2d`, `RenderBar2d`, `RenderGradationBar2d`, `RenderCircle2d`.
        *   **3D Çizim Metotları:**
            *   `RenderLine3d`, `RenderBox3d`, `RenderBar3d` (iki versiyon), `RenderGradationBar3d`, `RenderCircle3d`.
            *   `RenderLineCube`, `RenderCube` (iki versiyon: eksen hizalı ve rotasyonlu).
            *   `RenderTextureBox`: Bir dokuyu 2D bir dörtgen üzerine çizer.
            *   `RenderBillboard`: Bir pozisyonda kameraya bakan bir dörtgen (billboard) çizer.
            *   `DrawMinorGrid`, `DrawGrid`: Izgara çizer.
            *   `RenderD3DXMesh`, `RenderSphere`, `RenderCylinder`: Önceden oluşturulmuş veya D3DX yardımcı fonksiyonlarıyla oluşturulmuş meshleri/şekilleri çizer.
            *   `RenderTriangle3d` (`ENABLE_FOV_OPTION` ile): Basit bir 3D üçgen çizer.
            *   `RenderMiniMapFilter` (`ENABLE_FOV_OPTION` ile): Minimap üzerinde bir filtre (örneğin görüş konisi) çizmek için kullanılır.
        *   **Render Durumu Ayarları:** (`STATEMANAGER` üzerinden)
            *   `SetColorOperation`, `SetDiffuseOperation`, `SetBlendOperation`: Genel render durumlarını ayarlar.
            *   `SetOneColorOperation`, `SetAddColorOperation`: Belirli renk işlemleri için kısayollar.
            *   `SetDiffuseColor` (DWORD veya r,g,b,a): Çizimler için varsayılan diffuse rengini ayarlar (`ms_diffuseColor`).
            *   `SetClearColor`, `SetClearDepth`, `SetClearStencil`: Ekran temizleme renk/değerlerini ayarlar.
        *   **Fare ve Projeksiyon:**
            *   `SetCursorPosition(int x, int y, int hres, int vres)`: Ekran koordinatlarındaki fare pozisyonundan dünyaya doğru bir picking ray (seçim ışını) oluşturur.
            *   `GetCursorPosition(float* px, float* py, float* pz)`: Seçim ışınının belirli bir mesafedeki dünya koordinatlarını verir (genellikle bir düzlemle kesişim).
            *   `GetCursorXYPosition`, `GetCursorZPosition`: Ayrı ayrı X, Y veya Z koordinatlarını alır.
            *   `GetPickingPosition(float t, ...)`: Seçim ışını üzerinde `t` parametresine karşılık gelen dünya koordinatını alır.
            *   `ProjectPosition(float x, float y, float z, ...)`: Dünya koordinatlarını ekran koordinatlarına dönüştürür.
            *   `UnprojectPosition(float x, float y, float z, ...)`: Ekran koordinatlarını dünya koordinatlarına dönüştürür.
        *   **Cihaz Yönetimi:**
            *   `IsLostDevice()`: Direct3D cihazının kaybolup kaybolmadığını kontrol eder.
            *   `RestoreDevice()`: Kaybolan cihazı geri yüklemeye çalışır.
        *   **Görüş Hacmi (View Frustum):**
            *   `BuildViewFrustum()`: Mevcut view ve projection matrislerinden görüş hacmini (frustum) oluşturur. Bu, nesnelerin görünür olup olmadığını test etmek (frustum culling) için kullanılır.
            *   `static void Identity()`: Muhtemelen `CGraphicBase` içindeki matris yığınını sıfırlar.
            *   `static Frustum& GetFrustum()`: Hesaplanan görüş hacmini döndürür.
    *   **Statik Üyeler:**
        *   `ms_diffuseColor`: 2D/3D çizim fonksiyonlarında kullanılan varsayılan renk.
        *   `ms_clearColor`, `ms_clearStencil`, `ms_clearDepth`: `Clear()` fonksiyonu için varsayılan temizleme değerleri.
        *   `ms_frustum`: Oyun dünyasının görünür alanını tanımlayan frustum nesnesi.

*   **`GrpScreen.cpp` - Uygulama Detayları:**
    *   **Çizim Fonksiyonları Genel Yapısı:**
        *   Çoğu 2D/3D çizim fonksiyonu (`RenderLine3d`, `RenderBar3d`, `RenderCube` vb.) `SPDTVertexRaw` (pozisyon, diffuse renk, UV koordinatları içeren basit vertex yapısı) tipinde bir vertex array oluşturur.
        *   `SetPDTStream(vertices, count)` çağrılarak bu vertex verileri dinamik bir vertex buffer'a yüklenir.
        *   `STATEMANAGER` kullanılarak gerekli render durumları ayarlanır (Texture yok, Vertex Shader `D3DFVF_XYZ | D3DFVF_DIFFUSE | D3DFVF_TEX1` vb.).
        *   Eğer gerekiyorsa (`RenderLineCube`, `RenderCube`), `SetDefaultIndexBuffer` ile önceden tanımlanmış bir index buffer ayarlanır.
        *   Son olarak, `STATEMANAGER.DrawPrimitive` veya `STATEMANAGER.DrawIndexedPrimitive` çağrılarak çizim yapılır.
        *   Bazı çizim fonksiyonları (`RenderLine3d`, `RenderBox3d`) `GRAPHICS_CAPS_CAN_NOT_DRAW_LINE` isimli bir global boolean değişkene göre çizimi atlayabilir. Bu, bazı eski grafik kartlarının düzgün çizgi çizememesiyle ilgili bir optimizasyon/önlem olabilir.
    *   **`RenderCube` (Rotasyonlu Versiyon):** Köşe noktalarını önce küpün merkezine göre öteler, sonra verilen rotasyon matrisi ile çarpar ve tekrar eski pozisyonuna öteler. Bu sayede küp kendi merkezi etrafında döner.
    *   **`RenderD3DXMesh`, `RenderSphere`, `RenderCylinder`:** Bu fonksiyonlar, D3DX kütüphanesinin sağladığı hazır mesh oluşturma fonksiyonlarını (`D3DXCreateSphere`, `D3DXCreateCylinder`) veya yüklenmiş bir `LPD3DXMESH`'i kullanır. Çizimden önce fill mode (wireframe, solid) ayarlanabilir. `RenderD3DXMesh` içinde `CD3DXMeshRenderingOption` adında yerel bir yardımcı sınıf kullanılarak render durumları (özellikle `D3DRS_FILLMODE`) geçici olarak ayarlanır ve çizim sonrası eski haline getirilir.
    *   **Fare ve Projeksiyon (`SetCursorPosition`, `ProjectPosition`, `UnprojectPosition`):**
        *   `SetCursorPosition`: `D3DXVec3Unproject` fonksiyonunu kullanarak, fare imlecinin ekran koordinatlarından (yakın ve uzak projeksiyon düzlemlerinde) iki nokta hesaplar. Bu iki nokta, dünyada bir ışın (picking ray) tanımlar. Bu işlem `CCamera` sınıfından alınan view, projection matrislerini ve viewport bilgilerini kullanır.
        *   `ProjectPosition`: `D3DXVec3Project` kullanarak dünya koordinatlarını ekran koordinatlarına dönüştürür.
        *   `UnprojectPosition`: `D3DXVec3Unproject` kullanarak ekran koordinatlarını (ve bir Z derinliğini) dünya koordinatlarına dönüştürür.
    *   **Ekran Gösterme (`Show`):** `CGraphicBase::Show` metodunu çağırır. Bu da muhtemelen Direct3D `Present()` fonksiyonunu çağırır.
    *   **Cihaz Yönetimi (`IsLostDevice`, `RestoreDevice`):** `CGraphicBase` içindeki `IsLostDevice()` ve `RestoreDevice()` fonksiyonlarını çağırır. Bunlar, Direct3D cihazının `TestCooperativeLevel` ve `Reset` metodlarını kullanarak cihaz durumunu yönetir.
    *   **View Frustum (`BuildViewFrustum`):** `CCamera::Instance().GetViewMatrix()` ve `CCamera::Instance().GetProjectionMatrix()` ile güncel view ve projection matrislerini alır. Bu iki matrisi çarparak view-projection matrisini elde eder. Sonra `ms_frustum.BuildViewFrustum` metodunu bu birleşik matris ile çağırarak frustum düzlemlerini hesaplar.

*   **Kullanım Alanları:**
    *   Oyun içi debug bilgileri çizmek (çarpışma kutuları, yollar, görüş alanları vb.).
    *   Basit 2D UI elemanları (sağlık barları, arka planlar) çizmek.
    *   Geçici görsel efektler veya işaretçiler oluşturmak.
    *   Fare ile dünyadaki nesneleri seçmek (picking).
    *   Nesnelerin ekranda görünüp görünmediğini kontrol etmek (frustum culling).

### `GrpText.h` ve `GrpText.cpp` (`CGraphicText` Sınıfı)

*   **Amaç:** `CGraphicText` sınıfı, bir font dosyasından (`.fnt`) bir font kaynağı oluşturmak ve yönetmek için kullanılır. Aslında, temel işi yapan `CGraphicFontTexture` sınıfı için bir sarmalayıcı ve kaynak yöneticisi (`CResource`) entegrasyonu görevi görür. Font dosyası adından font adını, boyutunu ve stilini (italik, kalın) ayrıştırır ve bu bilgileri kullanarak bir `CGraphicFontTexture` nesnesi oluşturur.

*   **`GrpText.h` - Tanımlar ve Bildirimler:**
    *   **Kalıtım:** `public CResource`. Bu, `CGraphicText`'in bir kaynak olarak yönetilebileceğini (`CResourceManager` tarafından yüklenebilir, referans sayımı yapılabilir vb.) gösterir.
    *   **`TRef`:** `CRef<CGraphicText>` için bir typedef. Kaynakların referans sayımlı işaretçilerle yönetilmesini sağlar.
    *   **`static TType Type()`:** Sınıf için benzersiz bir tip tanımlayıcısı döndürür (RTTI benzeri bir sistem için).
    *   **Kurucu ve Yıkıcı:** `CGraphicText(const char* c_szFileName)`, `virtual ~CGraphicText()`.
    *   **Cihaz Nesneleri:**
        *   `virtual bool CreateDeviceObjects()`: Grafik cihazıyla ilişkili kaynakları (font dokusu) oluşturur. `m_fontTexture.CreateDeviceObjects()` çağırır.
        *   `virtual void DestroyDeviceObjects()`: Grafik cihazıyla ilişkili kaynakları serbest bırakır. `m_fontTexture.DestroyDeviceObjects()` çağırır.
    *   `GetFontTexturePointer()`: Dahili `CGraphicFontTexture` nesnesine bir işaretçi döndürür.
    *   **Korumalı Sanal Metotlar (`CResource` Arayüzü):**
        *   `OnLoad(int iSize, const void* c_pvBuf)`: Kaynağın (fontun) yüklenmesini gerçekleştirir. Asıl işi font dosya adını ayrıştırmak ve `m_fontTexture.Create()`'i çağırmaktır.
        *   `OnClear()`: Kaynağı temizler. `m_fontTexture.Destroy()` çağırır.
        *   `OnIsEmpty() const`: Kaynağın boş olup olmadığını kontrol eder. `m_fontTexture.IsEmpty()` sonucunu döndürür.
        *   `OnIsType(TType type)`: Verilen tipin bu sınıfın tipiyle eşleşip eşleşmediğini kontrol eder.
    *   **Korumalı Üye:**
        *   `m_fontTexture`: Asıl font oluşturma ve doku yönetimi işini yapan `CGraphicFontTexture` nesnesi.

*   **`GrpText.cpp` - Uygulama Detayları:**
    *   **Kurucu:** `CResource` kurucusunu dosya adıyla çağırır.
    *   **`OnLoad` Metodu (Font Dosya Adı Ayrıştırma):**
        *   Bu metodun asıl görevi, verilen `.fnt` dosya adından fontun gerçek adını, boyutunu ve stilini (italik/bold) çıkarmaktır.
        *   Dosya adı formatı şöyledir:
            *   `font_ismi.fnt`: Varsayılan boyut (12) ve stil (normal).
            *   `font_ismi:BOYUT.fnt`: Belirtilen boyut, normal stil. (Örnek: `gulim:18.fnt`)
            *   `font_ismi:BOYUTi.fnt`: Belirtilen boyut, italik stil. (Örnek: `gulim:14i.fnt`)
            *   `font_ismi:BOYUTb.fnt`: Belirtilen boyut, bold stil.
            *   `font_ismi:BOYUTib.fnt` veya `font_ismi:BOYUTbi.fnt`: Belirtilen boyut, italik ve bold stil.
        *   `strrchr` ile dosya adında `:` karakterini arar. Varsa, `:` öncesi font adı, `:` sonrası ise boyut ve stil bilgisidir.
        *   Sayısal karakterleri (`isdigit`) okuyarak boyutu alır (`atoi`).
        *   Boyuttan sonra `i` harfi varsa italik, `b` harfi varsa bold olarak işaretler.
        *   Eğer `:` yoksa, `.fnt` uzantısından önceki kısım font adı olarak alınır ve varsayılan boyut (12) kullanılır.
        *   Son olarak, ayrıştırılan bu bilgilerle (`strName`, `size`, `bItalic`, `bBold`) `m_fontTexture.Create(strName, size, bItalic, bBold)` fonksiyonunu çağırarak font dokusunu oluşturur.
    *   **Diğer `CResource` Metotları:** Basitçe `m_fontTexture` üzerindeki karşılık gelen metotları çağırır (`Destroy`, `IsEmpty`).
    *   **`Type` ve `OnIsType`:** Kaynak yönetim sistemi için tip bilgisini sağlar.

*   **Kullanım Alanı:**
    *   `CGraphicText`, oyun içinde kullanılacak fontları (`.fnt` dosyaları) `CResourceManager` aracılığıyla yüklemek için bir arayüz görevi görür.
    *   Bir fonta ihtiyaç duyulduğunda (örneğin, bir UI elemanı metin gösterecekse), `CResourceManager::GetResource(font_dosya_adi)` çağrılır. Bu, eğer yüklenmemişse `CGraphicText::OnLoad`'ı tetikler.
    *   Yüklenen `CGraphicText` nesnesinden `GetFontTexturePointer()` ile `CGraphicFontTexture` alınır. Bu `CGraphicFontTexture` nesnesi, metinleri ekrana çizmek için gereken asıl doku ve karakter bilgilerini içerir (örneğin, `CGraphicTextInstance` bu nesneyi kullanır).
    *   Temelde, font dosyası adlandırma kuralına dayalı bir yapılandırma katmanı ve `CGraphicFontTexture` için bir kaynak sarmalayıcısıdır.

### `GrpTextInstance.h` ve `GrpTextInstance.cpp` (`CGraphicTextInstance` Sınıfı)

*   **Amaç:** `CGraphicTextInstance` sınıfı, belirli bir metin dizesini (`std::string`) ve bir `CGraphicText` (font kaynağı) nesnesini kullanarak ekrana çizilebilecek bir metin örneği oluşturur ve yönetir. Metnin rengi, konumu, hizalaması, maksimum uzunluğu, çok satırlı olup olmaması, dış çizgi (outline) gibi çeşitli özelliklerini kontrol eder. Ayrıca metin içi etiketleri (hyperlink, renk değişimi vb.) işleyebilir.

*   **`GrpTextInstance.h` - Tanımlar ve Bildirimler:**
    *   **`TPool`:** `CDynamicPool<CGraphicTextInstance>` için bir typedef. Bu sınıfın nesnelerinin bir havuzdan yönetildiğini gösterir.
    *   **Enum'lar:**
        *   `EHorizontalAlign`: `HORIZONTAL_ALIGN_LEFT`, `CENTER`, `RIGHT`.
        *   `EVerticalAlign`: `VERTICAL_ALIGN_TOP`, `CENTER`, `BOTTOM`.
    *   **Statik Hyperlink Metotları:**
        *   `Hyperlink_UpdateMousePos(int x, int y)`: Fare pozisyonunu günceller (hyperlink tespiti için).
        *   `Hyperlink_GetText(char* buf, int len)`: Üzerine gelinen hyperlink metnini alır.
    *   **Kurucu ve Yıkıcı:** `CGraphicTextInstance()`, `virtual ~CGraphicTextInstance()`.
    *   **Yaşam Döngüsü:** `Destroy()`, `Update()`, `Render(RECT* pClipRect = NULL)`.
    *   **Görsel Özellikler ve Ayarlar:**
        *   `ShowCursor()`, `HideCursor()`.
        *   `ShowOutLine()`, `HideOutLine()`.
        *   `SetColor(float r, float g, float b, float a = 1.0f)` / `SetColor(DWORD color)`.
        *   `SetOutlineColor(...)`.
        *   `SetHorizonalAlign(int hAlign)`, `SetVerticalAlign(int vAlign)`.
        *   `SetMax(int iMax)`: Gösterilecek maksimum karakter sayısı.
        *   `SetTextPointer(CGraphicText* pText)`: Kullanılacak font kaynağını ayarlar.
        *   `SetValueString(const std::string& c_stValue)`, `SetValue(const char* c_szValue, size_t len = -1)`: Gösterilecek metni ayarlar.
        *   `SetPosition(float fx, float fy, float fz = 0.0f)`.
        *   `SetSecret(bool Value)`: Metni `*` karakterleriyle gizler.
        *   `SetOutline(bool Value)`: Dış çizgi kullanımını açar/kapatır.
        *   `SetFeather(bool Value)`: Kenar yumuşatma (font feathering) kullanılıp kullanılmayacağını ayarlar (global `c_fFontFeather` değerini etkiler gibi görünüyor).
        *   `SetMultiLine(bool Value)`: Çok satırlı metin desteğini açar/kapatır.
        *   `SetLimitWidth(float fWidth)`: Çok satırlı metinler için maksimum genişlik sınırı.
    *   **Bilgi Alma Metotları:**
        *   `GetTextSize(int* pRetWidth, int* pRetHeight)`: Metnin piksel cinsinden genişlik ve yüksekliğini döndürür.
        *   `GetValueStringReference() const`: Metin dizesine referans döndürür.
        *   `GetTextLineCount()`: Satır sayısını döndürür.
        *   `GetLineHeight()`, `SetLineHeight()`: Satır yüksekliğini alır/ayarlar.
        *   `PixelPositionToCharacterPosition(int iPixelPosition)`: Verilen piksel pozisyonuna karşılık gelen karakter indeksini döndürür.
        *   `GetHorizontalAlign()`.
    *   **Koşullu Derleme (`WJ_MULTI_TEXTLINE`, `ENABLE_TEXT_IMAGE_LINE`):**
        *   `CheckMultiLine()`, `DisableEnterToken()` (`WJ_MULTI_TEXTLINE` ile): Gelişmiş çok satırlı metin işleme.
        *   `SImage` yapısı ve `m_imageVector` (`ENABLE_TEXT_IMAGE_LINE` ile): Metin içine resim ekleme desteği.
    *   **Korumalı Metotlar:**
        *   `__Initialize()`: Üyeleri sıfırlar.
        *   `__DrawCharacter(...)`: Verilen karakteri `m_pCharInfoVector` ve `m_dwColorInfoVector`'a ekler.
        *   `__GetTextPos(...)`: Belirli bir karakter indeksindeki metnin göreceli pozisyonunu hesaplar.
        *   `__GetTextTag(...)`: Metin içindeki özel etiketleri (`|c`, `|h` vb.) ayrıştırır.
    *   **Korumalı Yapılar:** `SHyperlink`, `SImage`.
    *   **Üyeler:**
        *   Renkler: `m_dwTextColor`, `m_dwOutlineColor`.
        *   Boyutlar: `m_textWidth`, `m_textLineHeight`.
        *   Hizalama ve Sınırlar: `m_hAlign`, `m_vAlign`, `m_iMax`, `m_fLimitWidth`.
        *   Durum Bayrakları: `m_isCursor`, `m_isSecret`, `m_isMultiLine`, `m_isOutline`, `m_fFontFeather` (bu üye feather değerini tutar, `SetFeather` global `c_fFontFeather`'ı değiştirmez, isimler kafa karıştırıcı olabilir).
        *   Veri: `m_stText` (orijinal string), `m_v3Position`.
        *   Güncelleme Bayrakları: `m_isUpdate`, `m_isUpdateFontTexture`.
        *   `m_roText`: Kullanılan `CGraphicText` font kaynağı.
        *   `m_pCharInfoVector`: Her bir karakterin font dokusundaki bilgilerini tutan vektör.
        *   `m_dwColorInfoVector`: Her bir karakterin rengini tutan vektör.
        *   `m_hyperlinkVector`: Metindeki hyperlink bilgilerini tutar.
        *   `m_imageVector` (koşullu): Metindeki resim bilgilerini tutar.
        *   `m_vMultiTextLine` (koşullu): Çok satırlı metinler için alt `CGraphicTextInstance`'ları tutar.
    *   **Statik Havuz Yönetimi:** `CreateSystem`, `DestroySystem`, `New`, `Delete`, `ms_kPool`.

*   **`GrpTextInstance.cpp` - Uygulama Detayları:**
    *   **Statik Hyperlink İşleme:** `gs_mx`, `gs_my` fare pozisyonunu, `gs_hyperlinkText` ise üzerine gelinen hyperlink'in metnini tutar.
    *   **`Update()` Metodu (Ana Mantık):**
        *   Eğer `m_isUpdate` `false` ise (yani metin veya özellikleri değiştiyse) çalışır.
        *   Font (`m_roText`) ayarlanmamışsa veya boşsa erken çıkar.
        *   Karakter bilgi vektörlerini (`m_pCharInfoVector`, `m_dwColorInfoVector`), hyperlink ve resim vektörlerini temizler.
        *   `m_stText` (orijinal metin) üzerinde döngüye girer:
            *   **Token Ayrıştırma (`FindToken`, `ReadToken`):** `@1234` formatındaki token'ları bularak kod sayfası (`dataCodePage`) bilgisini çıkarır. `@9999` UTF-8 anlamına gelir.
            *   Tokenlar arasındaki metni `Ymir_MultiByteToWideChar` ile `wchar_t` dizisine dönüştürür.
            *   **Gizli Metin (`m_isSecret`):** Eğer aktifse, tüm karakterleri `*` olarak çizer.
            *   **Arapça İşleme (`defCodePage == CP_ARABIC`):**
                *   `Arabic_MakeShape` ile Arapça karakterlerin doğru birleştirilmiş şekillerini oluşturur.
                *   Arapça metni sağdan sola doğru işlerken İngilizce (veya diğer soldan sağa) kısımları ve metin etiketlerini doğru sırada ele almak için karmaşık bir mantık içerir.
            *   **Normal Metin İşleme (Arapça değilse):**
                *   `wchar_t` dizisindeki her karakter için:
                    *   `GetTextTag` ile metin etiketlerini (`|cRRGGBB`, `|hURL^text|h` vb.) kontrol eder.
                    *   **Renk Etiketi (`|c`):** `dwColor`'ı günceller.
                    *   **Hyperlink Etiketi (`|h`):** `SHyperlink` nesnesi oluşturup `m_hyperlinkVector`'a ekler.
                    *   **Resim Etiketi (`|Image_filename|`):** (`ENABLE_TEXT_IMAGE_LINE` ile) `CResourceManager::LoadResource` ile resmi yükler, `CGraphicImageInstance` oluşturur ve `m_imageVector`'a ekler.
                    *   **Diğer Etiketler:** `ProcessTextTag` (muhtemelen `TextTag.h`'den) ile işlenir.
                    *   Normal karakterler için `__DrawCharacter` çağrılarak karakter bilgileri ve renkleri ilgili vektörlere eklenir. Bu fonksiyon aynı zamanda `m_textWidth` ve `m_textLineHeight`'ı günceller.
        *   `m_isUpdate` `true` yapılır.
    *   **`Render()` Metodu:**
        *   Font (`m_roText`) veya karakter bilgisi yoksa erken çıkar.
        *   Render durumlarını ayarlar (alpha blend, Z-test/write, culling, filtering).
        *   Font dokusunu (`pFontTexture->GetTexture()`) ayarlar.
        *   Hesaplanmış `m_textWidth` ve `m_textLineHeight`'a göre hizalama (`m_hAlign`, `m_vAlign`) yapar ve başlangıç çizim pozisyonunu (`sx`, `sy`) hesaplar.
        *   `m_pCharInfoVector` üzerinde döngüye girerek her karakteri çizer:
            *   Karakterin font dokusundaki UV koordinatlarını (`pCharInfo->s`, `t`, `eS`, `eT`) ve ekrandaki pozisyonunu (`px`, `py`, `ex`, `ey`) hesaplar.
            *   Eğer `m_fLimitWidth` ayarlıysa ve karakter bu sınıra taşıyorsa, yeni satıra geçer.
            *   Dış çizgi (`m_isOutline`) aktifse, karakteri önce `m_dwOutlineColor` ile 4 yöne (yukarı-sol, yukarı-sağ, aşağı-sol, aşağı-sağ) 1 piksel kaydırarak çizer.
            *   Ardından asıl karakteri `m_dwColorInfoVector`'dan alınan renkle çizer.
            *   Çizim, `SPDTVertex` (Pozisyon, Diffuse, Texture1) tipinde 4 köşe noktası (quad) oluşturup `CGraphicBase::SetPDTStream` ve `STATEMANAGER.DrawPrimitiveUP` (veya benzeri) ile yapılır.
            *   Hyperlinkler için fare pozisyonu kontrol edilir ve `gs_hyperlinkText` güncellenir.
        *   (`ENABLE_TEXT_IMAGE_LINE` ile) `m_imageVector`'daki resimleri çizer.
    *   **`__DrawCharacter`:** `CGraphicFontTexture::GetCharacterInfomation` ile karakter bilgilerini alır, `m_dwColorInfoVector` ve `m_pCharInfoVector`'a ekler, `m_textWidth` ve `m_textLineHeight`'ı günceller.
    *   **`__GetTextPos`:** Belirli bir indekse kadar olan karakterlerin toplam genişliğini ve o anki satır yüksekliğini hesaplayarak göreceli pozisyonu verir.
    *   **`__GetTextTag`:** `ParseTextTag` (muhtemelen `TextTag.h`'den) fonksiyonunu kullanarak metin etiketlerini ayrıştırır.
    *   **Diğer Metotlar:** Çoğunlukla ilgili üye değişkenleri ayarlar ve `m_isUpdate`'i `false` yaparak `Update()`'in sonraki çağrıda çalışmasını tetikler.
    *   **Havuz Yönetimi:** `ms_kPool` kullanılarak nesnelerin verimli bir şekilde oluşturulup silinmesini sağlar.

*   **Kullanım Alanı:**
    *   Oyun içindeki hemen hemen tüm metin gösterimleri (UI elemanları, karakter isimleri, sohbet mesajları, item açıklamaları vb.) için kullanılır.
    *   `CGraphicText` ile yüklenen font kaynağını kullanarak metinleri belirli bir pozisyonda, renkte ve stilde render eder.
    *   Metin içi biçimlendirme (renk, hyperlink) ve özel karakter işleme (Arapça, gizli metin) yetenekleri sunar.
    *   Performans için nesne havuzu (object pool) kullanır.

### `GrpTexture.h` ve `GrpTexture.cpp` (`CGraphicTexture` Sınıfı)

*   **Amaç:** `CGraphicTexture` sınıfı, Direct3D doku nesneleri (`LPDIRECT3DTEXTURE8`) için temel bir sarmalayıcı (wrapper) sınıftır. Dokuyla ilgili temel bilgileri (genişlik, yükseklik, Direct3D doku işaretçisi) tutar ve doku kaynaklarının yönetimi için temel işlevleri (yok etme, başlatma) sağlar. Genellikle doğrudan kullanılmak yerine, `CGraphicImage` veya `CGraphicRenderTargetTexture` gibi daha özelleşmiş doku sınıfları için bir temel sınıf olarak görev yapar.

*   **`GrpTexture.h` - Tanımlar ve Bildirimler:**
    *   **Kalıtım:** `public CGraphicBase`. (Bu, `CGraphicBase`'in sağladığı statik Direct3D cihazı (`ms_lpd3dDevice`) gibi temel grafik kaynaklarına erişim sağlar.)
    *   `virtual bool IsEmpty() const`: Dokunun geçerli bir Direct3D doku nesnesi içerip içermediğini kontrol eder (`m_bEmpty` bayrağını döndürür).
    *   `GetWidth() const`, `GetHeight() const`: Dokunun piksel cinsinden genişliğini ve yüksekliğini döndürür.
    *   `SetTextureStage(int stage) const`: Bu dokuyu belirtilen Direct3D doku aşamasına (texture stage) ayarlar (`STATEMANAGER` aracılığıyla).
    *   `GetD3DTexture() const`: Dahili Direct3D doku nesnesine (`LPDIRECT3DTEXTURE8`) bir işaretçi döndürür.
    *   `DestroyDeviceObjects()`: Direct3D doku nesnesini serbest bırakır (`safe_release`).
    *   **Korumalı Kurucu ve Yıkıcı:** `CGraphicTexture()`, `virtual ~CGraphicTexture()`. Bu, sınıfın genellikle miras yoluyla kullanılmasının amaçlandığını gösterir.
    *   **Korumalı Metotlar:**
        *   `Destroy()`: `DestroyDeviceObjects()` çağırır ve ardından `Initialize()` ile üyeleri sıfırlar.
        *   `Initialize()`: Üye değişkenleri (`m_lpd3dTexture`'ı `NULL`, boyutları 0, `m_bEmpty`'yi `true`) başlangıç durumuna getirir.
    *   **Korumalı Üyeler:**
        *   `m_bEmpty`: Dokunun boş olup olmadığını belirten bayrak.
        *   `m_width`, `m_height`: Dokunun boyutları.
        *   `m_lpd3dTexture`: Asıl Direct3D doku nesnesine işaretçi.

*   **`GrpTexture.cpp` - Uygulama Detayları:**
    *   **`DestroyDeviceObjects()`:** `m_lpd3dTexture` işaretçisi geçerliyse `safe_release` makrosu ile Direct3D doku kaynağını serbest bırakır.
    *   **`Destroy()`:** Önce `DestroyDeviceObjects()` ile cihazla ilişkili kaynağı siler, sonra `Initialize()` ile sınıf üyelerini sıfırlar.
    *   **`Initialize()`:** Tüm üyeleri varsayılan (boş/geçersiz) değerlerine ayarlar.
    *   **`IsEmpty()`:** `m_bEmpty` bayrağının değerini döndürür.
    *   **`SetTextureStage(int stage)`:** `STATEMANAGER.SetTexture(stage, m_lpd3dTexture)` çağırarak bu dokuyu ilgili doku aşamasına atar. Bu, render sırasında shader'ların bu dokuyu kullanabilmesini sağlar.
    *   **`GetD3DTexture()`:** `m_lpd3dTexture` işaretçisini döndürür.
    *   **`GetWidth()`, `GetHeight()`:** `m_width` ve `m_height` üyelerini döndürür.
    *   **Kurucu:** `Initialize()` çağırarak başlar.
    *   **Yıkıcı:** Herhangi bir özel işlem yapmaz (temizlik `Destroy` veya `DestroyDeviceObjects` ile yönetilir).

*   **Kullanım Alanı:**
    *   `CGraphicTexture`, `EterLib` içinde doku tabanlı kaynakların (resimler, font dokuları, render hedefleri) temelini oluşturur.
    *   Doğrudan örneklenmek yerine, türetilmiş sınıflar (örneğin, `CGraphicImage` bir resim dosyasından doku yüklerken) bu sınıfı genişletir.
    *   Türetilmiş sınıflar, `m_lpd3dTexture` üyesini kendi yöntemleriyle (örneğin, dosyadan yükleyerek veya programatik olarak oluşturarak) doldurur ve `m_width`, `m_height`, `m_bEmpty` üyelerini buna göre günceller.
    *   Render sırasında, bir nesnenin materyali bu sınıfın `SetTextureStage` metodunu kullanarak dokuyu uygun bir doku ünitesine bağlar.
