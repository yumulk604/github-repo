# EterLib Referans Kılavuzu - Bölüm 2

Bu dosya, `EterLib` referans kılavuzunun devamıdır.

---

## Dosya Bazlı Detaylandırma (Devamı) 

### `NetDevice.h` ve `NetDevice.cpp` (`CNetworkDevice` Sınıfı)

*   **Amaç:** Bu dosyalar, Windows Sockets API (WSA) başlatma ve sonlandırma işlemlerini yöneten temel bir sınıf olan `CNetworkDevice`'ı tanımlar ve uygular. Ağ iletişimi yapacak herhangi bir modülün WSA'yı doğru bir şekilde başlatıp temizlemesi için bir altyapı sunar.

*   **`NetDevice.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CNetworkDevice`: Ağ aygıtı işlemlerini (özellikle WSA yönetimi) sarmalayan ana sınıf.
        *   `CNetworkDevice()`: Kurucu metot.
        *   `~CNetworkDevice()`: Yıkıcı metot.
        *   `Destroy()`: WSA'yı temizler.
        *   `Create()`: WSA'yı başlatır.
        *   `Initialize()`: Sınıf üyelerini başlangıç durumuna getirir.
        *   `m_isWSA`: WSA'nın başarıyla başlatılıp başlatılmadığını gösteren bir boolean bayrak.

*   **`NetDevice.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **WSA Yaşam Döngüsü Yönetimi:** `Create()` metodu içinde `WSAStartup()` çağrılarak Winsock API kullanımı için gerekli ortam başlatılır. `Destroy()` metodu (ve dolayısıyla yıkıcı) içinde ise `WSACleanup()` çağrılarak bu ortam sonlandırılır ve kaynaklar serbest bırakılır.
    *   **Durum Kontrolü:** `m_isWSA` bayrağı, `WSAStartup`'ın başarılı olup olmadığını ve `WSACleanup`'ın çağrılması gerekip gerekmediğini takip eder. Bu, gereksiz API çağrılarını ve olası hataları önler.
    *   **İdempotans:** `Create()` çağrıldığında önce `Destroy()` çağrılır, bu da mevcut bir WSA oturumu varsa temizlenmesini ve ardından yeniden başlatılmasını sağlar.

*   **Önemli Fonksiyonlar (`NetDevice.cpp`):**
    *   `CNetworkDevice::Create()`: `WSAStartup(MAKEWORD(1, 1), ...)` fonksiyonunu çağırarak WSA'nın 1.1 sürümünü başlatmaya çalışır. Başarı durumunda `m_isWSA`'yı `true` yapar.
    *   `CNetworkDevice::Destroy()`: Eğer `m_isWSA` `true` ise, `WSACleanup()` çağırır.
    *   `CNetworkDevice::Initialize()`: `m_isWSA`'yı `false` olarak ayarlar.

*   **Kullanım Alanı:**
    *   `CNetworkDevice` sınıfı, istemci tarafında ağ iletişimi yapacak üst seviye sınıflar (örneğin, `CNetworkStream` veya UDP soketlerini yöneten diğer sınıflar) tarafından genellikle bir ön koşul olarak kullanılır.
    *   Uygulama başladığında bir örneği oluşturulup `Create()` metodu çağrılarak WSA'nın hazır hale getirilmesini ve uygulama sonlandığında örneği yok edilerek (`Destroy()` aracılığıyla) WSA'nın düzgün bir şekilde kapatılmasını sağlar. Bu, ağ fonksiyonlarının doğru çalışması ve sistem kaynaklarının düzgün yönetimi için kritik öneme sahiptir.

### `NetDatagramReceiver.h` ve `NetDatagramReceiver.cpp` (`CNetDatagramReceiver` Sınıfı)

*   **Amaç:** Bu dosyalar, UDP (User Datagram Protocol) üzerinden gelen bağlantısız datagramları almak için kullanılan `CNetDatagramReceiver` sınıfını tanımlar ve uygular. Belirli bir portu dinleyerek gelen verileri bir tamponda yönetir.

*   **`NetDatagramReceiver.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CNetDatagramReceiver`: UDP datagram alımını yöneten ana sınıf.
        *   `CNetDatagramReceiver()`: Kurucu metot.
        *   `~CNetDatagramReceiver()`: Yıkıcı metot.
        *   `Clear()`: Sınıf üyelerini ve arabelleği temizler.
        *   `Bind(DWORD dwAddress, WORD wPortIndex)`: Belirtilen IP adresi (genellikle `INADDR_ANY`) ve porta bir UDP soketi bağlar.
        *   `isBind()`: Soketin bağlı olup olmadığını kontrol eder.
        *   `Process()`: Soketten gelen veriyi okumaya çalışır ve iç alım arabelleğine yazar.
        *   `Recv(void* pBuffer, int iSize)`: Alım arabelleğinden belirtilen boyutta veriyi okur ve arabellekten çıkarır.
        *   `Peek(void* pBuffer, int iSize)`: Alım arabelleğinden belirtilen boyutta veriyi okur ancak arabellekten çıkarmaz.
        *   `SetRecvBufferSize(int recvBufSize)`: Alım arabelleğinin boyutunu ayarlar.
    *   **Önemli Üyeler:**
        *   `m_isBind`: Soketin bağlı olup olmadığını belirten bayrak.
        *   `m_Socket`: UDP soket tanıtıcısı.
        *   `m_SockAddr`: Soketin bağlandığı adres bilgileri (`SOCKADDR_IN`).
        *   `m_recvBuf`, `m_recvBufSize`: Alım arabelleği ve boyutu.
        *   `m_recvBufCurrentPos`, `m_recvBufCurrentSize`: Alım arabelleğindeki geçerli okuma pozisyonu ve arabellekteki mevcut veri boyutu.

*   **`NetDatagramReceiver.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Soket Oluşturma ve Bağlama:** `Bind` metodu, `socket(AF_INET, SOCK_DGRAM, 0)` ile bir UDP soketi oluşturur, `ioctlsocket` ile non-blocking moda alır ve `bind` ile belirtilen porta (ve genellikle `INADDR_ANY` adresine) bağlar.
    *   **Veri Alımı:** `Process` metodu, `recvfrom` fonksiyonunu kullanarak bağlı soketten veri almaya çalışır. Alınan veri `m_recvBuf` adlı iç arabelleğe yazılır ve `m_recvBufCurrentSize` güncellenir.
    *   **Arabellek Yönetimi:** `Recv` metodu, `Peek` metodunu kullanarak veriyi istenen `pBuffer`'a kopyalar ve ardından `m_recvBufCurrentPos`'u ilerleterek verinin "tüketildiğini" işaretler. `Peek` ise sadece kopyalama yapar, pozisyonu değiştirmez.
    *   **Arabellek Boyutlandırma:** `SetRecvBufferSize` metodu, alım arabelleğinin (`m_recvBuf`) dinamik olarak yeniden boyutlandırılmasına olanak tanır.

*   **Önemli Fonksiyonlar (`NetDatagramReceiver.cpp`):**
    *   `CNetDatagramReceiver::Bind(...)`: Soketi oluşturur, non-blocking yapar ve belirtilen porta bağlar.
    *   `CNetDatagramReceiver::Process()`: `recvfrom` kullanarak soketten veri okur ve alım arabelleğine depolar.
    *   `CNetDatagramReceiver::Recv(...)`: Alım arabelleğinden veriyi okur ve arabellekteki okuma pozisyonunu ilerletir.
    *   `CNetDatagramReceiver::Peek(...)`: Alım arabelleğinden veriyi okur ancak okuma pozisyonunu değiştirmez.

*   **Kullanım Alanı:**
    *   `CNetDatagramReceiver`, genellikle sunucudan veya diğer istemcilerden gelen UDP paketlerini dinlemek ve almak için kullanılır. Örneğin, oyun içindeki hızlı pozisyon güncellemeleri, anlık bildirimler gibi daha az güvenilirlik gerektiren ama düşük gecikme beklenen verilerin alınmasında tercih edilebilir.
    *   Bir portu dinleyerek gelen datagramları alır ve bu datagramların daha sonra işlenmek üzere bir arabellekte tutulmasını sağlar. Uygulama, `Process()` metodunu periyodik olarak çağırarak yeni gelen verileri kontrol eder ve `Recv()` veya `Peek()` ile bu verilere erişir.

### `NetDatagramSender.h` ve `NetDatagramSender.cpp` (`CNetDatagramSender` Sınıfı)

*   **Amaç:** Bu dosyalar, UDP (User Datagram Protocol) üzerinden belirli bir hedefe bağlantısız datagram göndermek için kullanılan `CNetDatagramSender` sınıfını tanımlar ve uygular.

*   **`NetDatagramSender.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CNetDatagramSender`: UDP datagram gönderimini yöneten ana sınıf.
        *   `CNetDatagramSender()`: Kurucu metot.
        *   `~CNetDatagramSender()`: Yıkıcı metot.
        *   `isSocket()`: Soketin ayarlanıp ayarlanmadığını (yani gönderime hazır olup olmadığını) kontrol eder.
        *   `SetSocket(const char* c_szIP, WORD wPortIndex)`: Hedef IP adresini (string olarak) ve port numarasını alarak gönderim için bir soket ve hedef adresi ayarlar.
        *   `SetSocket(DWORD dwAddress, WORD wPortIndex)`: Hedef IP adresini (DWORD olarak) ve port numarasını alarak gönderim için bir soket ve hedef adresi ayarlar.
        *   `Send(const void* pBuffer, int iSize)`: Ayarlanan hedefe belirtilen tampondaki veriyi gönderir.
    *   **Önemli Üyeler:**
        *   `m_isSocket`: Soketin ve hedef adresin ayarlanıp ayarlanmadığını belirten bayrak.
        *   `m_dwAddress`: Hedef IP adresi (DWORD formatında).
        *   `m_wPortIndex`: Hedef port numarası.
        *   `m_Socket`: UDP soket tanıtıcısı.
        *   `m_SockAddr`: Hedef adres bilgileri (`SOCKADDR_IN`).

*   **`NetDatagramSender.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Soket ve Hedef Belirleme:** `SetSocket` metotları (hem string IP hem de DWORD IP alan versiyonları) çağrıldığında, `socket(AF_INET, SOCK_DGRAM, 0)` ile bir UDP soketi oluşturulur. Verilen IP adresi ve port numarası kullanılarak `m_SockAddr` (`SOCKADDR_IN` yapısı) doldurulur. Bu yapı, `Send` metodunda `sendto` fonksiyonuna hedef olarak verilir.
    *   **Veri Gönderimi:** `Send` metodu, `sendto` Windows API fonksiyonunu kullanarak `pBuffer` içindeki `iSize` boyutundaki veriyi, daha önce `SetSocket` ile belirlenmiş olan `m_SockAddr` hedefine gönderir.
    *   **Durum Kontrolü:** `m_isSocket` bayrağı, `SetSocket` başarıyla çağrıldıktan sonra `true` olur ve `Send` metodunun çağrılabilmesi için bir ön koşul olarak `assert` ile kontrol edilir.

*   **Önemli Fonksiyonlar (`NetDatagramSender.cpp`):**
    *   `CNetDatagramSender::SetSocket(DWORD dwAddress, WORD wPortIndex)`: UDP soketini oluşturur ve hedef `SOCKADDR_IN` yapısını verilen IP (DWORD) ve port ile doldurur.
    *   `CNetDatagramSender::Send(const void* pBuffer, int iSize)`: `sendto` kullanarak hazırlanan veriyi belirlenen hedefe gönderir.

*   **Kullanım Alanı:**
    *   `CNetDatagramSender`, istemciden sunucuya veya başka bir istemciye UDP üzerinden hızlı ve bağlantısız veri paketleri göndermek için kullanılır. `CNetDatagramReceiver`'ın karşıtıdır.
    *   Örneğin, oyuncunun hareket komutları, anlık aksiyon bildirimleri gibi verilerin gönderiminde tercih edilebilir. Kullanıcı öncelikle `SetSocket` ile hedef sunucunun veya alıcının IP adresini ve portunu belirler. Ardından, gönderilecek veriyi `Send` metoduyla yollar.
    *   Her bir `CNetDatagramSender` örneği genellikle tek bir hedefe veri göndermek üzere yapılandırılır.

### `PathStack.h` ve `PathStack.cpp` (`CPathStack` Sınıfı)

*   **Amaç:** Bu dosyalar, geçerli çalışma dizinini (current working directory) bir yığın (stack) üzerinde yönetmek ve dizinler arası geçişleri kolaylaştırmak için `CPathStack` sınıfını tanımlar ve uygular.

*   **`PathStack.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CPathStack`: Çalışma dizini yığınını yöneten ana sınıf.
        *   `CPathStack()`: Kurucu metot. `SetBase()` çağırır.
        *   `~CPathStack()`: Yıkıcı metot. `MoveBase()` çağırır.
        *   `SetBase()`: O anki geçerli çalışma dizinini temel (başlangıç) dizin olarak kaydeder.
        *   `MoveBase()`: Geçerli çalışma dizinini daha önce `SetBase()` ile kaydedilmiş olan temel dizine değiştirir.
        *   `Push()`: O anki geçerli çalışma dizinini yığına (deque'in başına) ekler.
        *   `Pop()`: Yığının en üstündeki (deque'in başındaki) dizin yolunu alır, geçerli çalışma dizinini bu yola değiştirir ve ardından bu yolu yığından çıkarır. Yığın boşsa `false` döner.
        *   `Move(const char* c_szPathName)`: Geçerli çalışma dizinini belirtilen `c_szPathName` yoluna değiştirir.
        *   `GetCurrentPathName(std::string* pstCurPathName)`: O anki geçerli çalışma dizinini `pstCurPathName` string'ine yazar.
    *   **Önemli Üyeler:**
        *   `m_stBasePathName`: `SetBase()` ile ayarlanan temel çalışma dizinini tutan `std::string`.
        *   `m_stPathNameDeque`: `Push()` ile eklenen çalışma dizini yollarını tutan bir `std::deque<std::string>`.

*   **`PathStack.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Dizin Yönetimi API Kullanımı:** Sınıf, geçerli çalışma dizinini almak için `_getcwd` ve değiştirmek için `_chdir` Windows API fonksiyonlarını kullanır.
    *   **Yığın Mekanizması:** `std::deque` veri yapısı bir yığın gibi kullanılır. `Push()` metodu `push_front()` ile mevcut dizini deque'in önüne ekler. `Pop()` metodu ise `front()` ile en son eklenen (en üstteki) dizini alır, `_chdir` ile o dizine geçer ve `pop_front()` ile yığından çıkarır.
    *   **Temel Dizin:** `m_stBasePathName`, genellikle sınıfın örneği oluşturulduğunda veya belirli bir başlangıç noktası ayarlanmak istendiğinde mevcut çalışma dizinini saklar. Yıkıcı metot (`~CPathStack()`) `MoveBase()` çağırarak, nesne kapsam dışına çıktığında çalışma dizininin bu temel dizine geri dönmesini sağlar (RAII prensibine benzer bir kullanım).

*   **Önemli Fonksiyonlar (`PathStack.cpp`):**
    *   `CPathStack::Push()`: `_getcwd` ile mevcut dizini alır ve `m_stPathNameDeque.push_front()` ile yığına ekler.
    *   `CPathStack::Pop()`: Yığından bir dizin yolu alır (`m_stPathNameDeque.front()`), `_chdir` ile o dizine geçer ve yolu yığından (`m_stPathNameDeque.pop_front()`) çıkarır.
    *   `CPathStack::SetBase()`: `_getcwd` ile mevcut dizini alır ve `m_stBasePathName`'e atar.
    *   `CPathStack::MoveBase()`: `_chdir` ile `m_stBasePathName`'e geçer.

*   **Kullanım Alanı:**
    *   `CPathStack` sınıfı, programın farklı bölümlerinde geçici olarak çalışma dizininin değiştirilmesi gerektiğinde kullanışlıdır. Örneğin, belirli bir klasördeki dosyaları işlemek için o klasöre geçici olarak `Move()` veya `Push()`+`Move()` kombinasyonu ile geçilip, işlem bittikten sonra `Pop()` veya `MoveBase()` ile önceki veya temel dizine kolayca geri dönülebilir.
    *   Özellikle dosya yükleme/kaydetme, kaynak arama veya harici araçları çağırma gibi işlemlerde göreceli yollarla çalışırken mevcut çalışma dizininin doğru olmasını sağlamak için kullanılır.
    *   Bir nesnenin ömrü boyunca belirli bir temel çalışma dizininden sapmaları yönetmek ve nesne yok edildiğinde otomatik olarak bu temel dizine geri dönülmesini sağlamak için RAII (Resource Acquisition Is Initialization) benzeri bir desenle kullanılabilir. 

### `Profiler.h` (Performans Profilleme Araçları)

*   **Amaç:** Bu başlık dosyası, kodun belirli bölümlerinin çalışma zamanlarını ve çağrı sayılarını ölçmek için kullanılan performans profilleme araçlarını içerir. Temel olarak iki tür profilleme mekanizması sunar: kapsam tabanlı otomatik profilleme için `AUTO_PROFILER` makrosu ve yorumlanmış kod içinde detaylı bir `CProfiler` sınıfı.

*   **Temel Özellikler ve Tanımlamalar:**

    *   **`AUTO_PROFILER(name)` Makrosu:**
        *   `#ifdef _DEBUG` direktifi ile yalnızca hata ayıklama (debug) modunda aktif olacak şekilde tasarlanmıştır.
        *   Kullanıldığı kod bloğunun (kapsamın) süresini ölçmek için kullanılır. Kapsamın başına `name` ile bir profil kaydı başlatır ve kapsam sona erdiğinde otomatik olarak durdurur.
        *   Arka planda `CAutoProfiler` adlı bir yardımcı sınıf kullanır.

    *   **`CAutoProfiler` Sınıfı:**
        *   `AUTO_PROFILER` makrosu tarafından kullanılır.
        *   Kurucu metodu (`CAutoProfiler(const char* c_szName)`), `CProfiler::Instance().Push(c_szName)` çağırarak `c_szName` adlı profil kaydını başlatır.
        *   Yıkıcı metodu (`~CAutoProfiler()`), `CProfiler::Instance().Pop(m_szName)` çağırarak ilgili profil kaydını sonlandırır.
        *   Bu RAII (Resource Acquisition Is Initialization) deseni, profil kaydının kapsam sonunda otomatik olarak kapatılmasını garanti eder.

    *   **`CProfiler` Sınıfı (Yorum Satırları İçinde):**
        *   *Not: Bu sınıfın tanımı ve metotlarının büyük bir kısmı dosya içinde `/* ... */` yorum blokları arasında yer almaktadır. Bu, bu sınıfın eski, deneysel veya şu anda aktif olarak kullanılmayan bir özellik olabileceğini gösterir. Aşağıdaki açıklamalar bu yorumlanmış koda dayanmaktadır.*
        *   `CSingleton<CProfiler>` olarak tasarlanmıştır.
        *   **Veri Yapıları:**
            *   `SProfileStackData`: Bir fonksiyonun/kod bloğunun çağrı derinliğini (`iCallStep`), başlangıç (`iStartTime`) ve bitiş (`iEndTime`) zamanlarını (milisaniye cinsinden `ELTimer_GetMSec()` ile) ve adını (`strName`) tutar.
            *   `SProfileAccumulationData`: Bir fonksiyonun/kod bloğunun toplam çalışma süresini (`iCollapsedTime`), toplam çağrılma sayısını (`iCallingCount`), son başlangıç zamanını (`iStartTime`) ve adını (`strName`) tutar.
        *   **Ana Metotlar (Yorumlanmış):**
            *   `Push(const char* c_szName)`: `SProfileStackData` için bir profil kaydı başlatır.
            *   `Pop(const char* c_szName)`: `SProfileStackData` için bir profil kaydını sonlandırır ve süreyi hesaplar.
            *   `PushAccumulation(const char* c_szName)`: `SProfileAccumulationData` için bir ölçüm periyodunu başlatır.
            *   `PopAccumulation(const char* c_szName)`: `SProfileAccumulationData` için ölçüm periyodunu sonlandırır, toplam süreye ekler ve çağrı sayısını artırır.
            *   `Clear()`: Tüm profil verilerini (hem stack hem de accumulation) sıfırlar.
            *   `ProfileByConsole()`: Toplanan tüm `SProfileStackData` verilerini konsola yazdırır.
            *   `ProfileOneStackDataByConsole(const char* c_szName)`: Belirli bir `SProfileStackData` kaydını konsola yazdırır.
            *   `ProfileOneAccumulationDataByConsole(const char* c_szName)`: Belirli bir `SProfileAccumulationData` kaydını konsola yazdırır.
            *   `ProfileByScreen()`: Hem stack hem de accumulation verilerini ekrana `CGraphicTextInstance` kullanarak grafiksel olarak çizer. (Font olarak `"ü.fnt"` kullanmaya çalışır, bu muhtemelen bozuk bir font adı veya kodlama sorunudur).
        *   **Veri Saklama:**
            *   `m_ProfileStackDatas[]`: `SProfileStackData` nesneleri için sabit boyutlu bir dizi (`STACK_DATA_MAX_NUM`).
            *   `m_ProfileAccumulationDataMap`: İsimlerle (`std::string`) `TProfileAccumulationData` nesnelerini eşleyen bir `std::map`.
            *   `m_GraphicTextInstanceMap`: Ekrana çizim için isimlerle `CGraphicTextInstance*` işaretçilerini eşleyen bir `std::map`.

    *   **Global `FrameProfiler` Fonksiyonları:**
        *   `FrameProfiler_Create()`
        *   `FrameProfiler_Destroy()`
        *   `FrameProfiler_Begin(const char* c_szName)`
        *   `FrameProfiler_End(const char* c_szName)`
        *   `FrameProfiler_Render()`
        *   Bu fonksiyonlar, muhtemelen (bu dosyada tanımlanmamış) bir `CFrameProfiler` sınıfının arayüzünü oluşturur. Genellikle tüm bir frame'in veya frame içindeki ana mantıksal bölümlerin (örneğin, Update, Render) profilini çıkarmak için kullanılırlar. `Begin` ve `End` çağrıları arasına alınan kodun süresini ölçer ve `Render` ile sonuçları ekranda gösterir.

*   **Kullanım Alanı:**
    *   **`AUTO_PROFILER` Makrosu:** Geliştirme sırasında, belirli fonksiyonların veya kod bloklarının ne kadar sürdüğünü hızlıca anlamak için kullanılır. `#ifdef _DEBUG` sayesinde release buildlerde herhangi bir performans etkisi yaratmaz. Kapsam tabanlı olduğu için kullanımı kolaydır ve profil kaydının sonlandırılmasını unutma riskini ortadan kaldırır.
    *   **Yorumlanmış `CProfiler` Sınıfı:** Eğer aktif edilirse, daha detaylı ve kalıcı profil bilgileri (toplam süre, çağrı sayısı) toplamak ve bu bilgileri oyun içinde ekranda veya konsolda görüntülemek için kullanılabilir. Oyunun darboğazlarını (bottlenecks) bulmada yardımcı olabilir.
    *   **`FrameProfiler` Fonksiyonları:** Oyunun ana döngüsündeki büyük bölümlerin (örneğin, fizik, yapay zeka, render) genel performansını izlemek ve zaman içindeki değişimlerini gözlemlemek için kullanılır. Bu, optimizasyon çalışmalarında hangi ana sistemlere odaklanılması gerektiğini belirlemeye yardımcı olabilir.
    *   Genel olarak, `Profiler.h` içindeki araçlar, geliştiricilerin oyunun performansını analiz etmelerine ve optimizasyon için potansiyel alanları belirlemelerine olanak tanır. 

### `Ray.h` (`CRay` Sınıfı)

*   **Amaç:** Bu başlık dosyası, 3D uzayda bir ışını (ray) temsil eden `CRay` sınıfını tanımlar. Işınlar, genellikle çarpışma tespiti, nesne seçimi (picking) ve diğer geometrik sorgulamalar için kullanılır.

*   **Temel Özellikler ve Tanımlamalar (`CRay` Sınıfı):**
    *   **Üye Değişkenler:**
        *   `m_v3Start`: Işının 3D uzaydaki başlangıç noktasını temsil eden `D3DXVECTOR3`.
        *   `m_v3End`: Işının hesaplanmış bitiş noktasını temsil eden `D3DXVECTOR3` (`m_v3Start + m_fRayRange * m_v3Direction`).
        *   `m_v3Direction`: Işının yönünü gösteren normalize edilmiş `D3DXVECTOR3`.
        *   `m_fRayRange`: Işının başlangıç noktasından itibaren ne kadar uzağa gideceğini belirten `float` cinsinden menzili.
    *   **Kurucu Metotlar:**
        *   `CRay(const D3DXVECTOR3& v3Start, const D3DXVECTOR3& v3Dir, float fRayRange)`: Başlangıç noktası, yön vektörü ve menzil ile bir ışın oluşturur. Yön vektörünü normalize eder ve bitiş noktasını hesaplar.
        *   `CRay()`: Varsayılan kurucu metot.
    *   **Ayarlayıcı (Setter) Metotlar:**
        *   `SetStartPoint(const D3DXVECTOR3& v3Start)`: Işının başlangıç noktasını ayarlar.
        *   `SetDirection(const D3DXVECTOR3& v3Dir, float fRayRange)`: Işının yönünü ve menzilini ayarlar. Verilen yön vektörünü normalize eder ve bu yeni değerlere göre bitiş noktasını (`m_v3End`) yeniden hesaplar.
    *   **Alıcı (Getter) Metotlar:**
        *   `GetStartPoint(D3DXVECTOR3* pv3Start) const`: Işının başlangıç noktasını `pv3Start` işaretçisine yazar.
        *   `GetDirection(D3DXVECTOR3* pv3Dir, float* pfRayRange) const`: Işının normalize edilmiş yön vektörünü `pv3Dir`'e ve menzilini `pfRayRange`'e yazar.
        *   `GetEndPoint(D3DXVECTOR3* pv3End) const`: Işının hesaplanmış bitiş noktasını `pv3End`'e yazar.
    *   **Operatörler:**
        *   `const CRay& operator=(const CRay& rhs)`: Atama operatörü. Bir `CRay` nesnesinin değerlerini diğerine kopyalar, kopyalama sırasında yön vektörünü normalize eder ve bitiş noktasını günceller.

*   **Ana Çalışma Prensibi:**
    *   `CRay` sınıfı, bir başlangıç noktası, bir yön ve bir menzil ile tanımlanan sınırlı bir doğru parçasını temsil eder.
    *   Yön vektörü (`m_v3Direction`) her zaman birim vektör (normalize edilmiş) olarak tutulur. Bu, menzil ile çarpıldığında doğru uzunlukta bir segment elde edilmesini sağlar.
    *   Bitiş noktası (`m_v3End`), başlangıç noktasına menzil kadar normalize edilmiş yön vektörünün eklenmesiyle (`m_v3Start + m_fRayRange * m_v3Direction`) dinamik olarak hesaplanır ve saklanır.
    *   Metotlar, `assert(fRayRange >= 0)` ile menzilin negatif olmamasını sağlar.

*   **Kullanım Alanı:**
    *   **Çarpışma Tespiti (Collision Detection):** Bir ışının oyun dünyasındaki nesnelerle (karakterler, arazi, binalar vb.) kesişip kesişmediğini test etmek için kullanılır. Örneğin, bir merminin hedefe isabet edip etmediği, bir karakterin belirli bir yönde engelle karşılaşıp karşılaşmadığı bu şekilde belirlenebilir.
    *   **Nesne Seçimi (Picking):** Fare imlecinin 3D dünyada hangi nesnenin üzerine geldiğini belirlemek için kullanılır. Genellikle kamera pozisyonundan fare imlecinin ekran pozisyonuna doğru bir ışın gönderilir ve bu ışının kestiği ilk nesne seçilmiş kabul edilir.
    *   **Görüş Hattı (Line-of-Sight, LOS) Kontrolleri:** İki nokta arasında doğrudan bir görüş hattı olup olmadığını (yani arada bir engel olup olmadığını) kontrol etmek için kullanılır. Örneğin, bir yapay zeka karakterinin oyuncuyu görüp göremediğini anlamak için kullanılabilir.
    *   **Işın İzleme (Ray Tracing) Temelleri:** Basit yansıma veya gölge hesaplamaları gibi daha gelişmiş grafik tekniklerinde temel bir bileşen olarak kullanılabilir (ancak bu kütüphane bağlamında bu kadar karmaşık bir kullanım beklenmeyebilir).
    *   Genel olarak, 3D uzayda yönlü bir sorgulama veya test yapılması gereken her durumda `CRay` sınıfı faydalı bir araçtır. 

### `Ref.h` (`CRef<T>` Template Sınıfı)

*   **Amaç:** Bu başlık dosyası, `CReferenceObject`'ten türeyen ve referans sayımı (reference counting) ile yönetilen nesneler için bir akıllı işaretçi (smart pointer) olan `CRef<T>` template sınıfını tanımlar. Temel amacı, bu tür nesnelerin ömürlerini otomatik olarak yöneterek bellek sızıntılarını ve geçersiz işaretçi ("dangling pointer") sorunlarını önlemektir.

*   **Temel Özellikler ve Tanımlamalar (`CRef<T>` Sınıfı):**
    *   **Template Parametresi `T`:** `CRef` tarafından yönetilecek nesnenin türünü belirtir. Bu türün `CReferenceObject`'ten türemiş olması beklenir.
    *   **Üye Değişken:**
        *   `m_pObject`: Yönetilen `CReferenceObject` (veya ondan türeyen `T` türünden) nesnesine işaret eden bir ham işaretçi (`CReferenceObject*`).
    *   **Kurucu Metotlar:**
        *   `CRef()`: Varsayılan kurucu. `m_pObject`'i `NULL` olarak başlatır.
        *   `CRef(CReferenceObject* pObject)`: Bir `CReferenceObject` işaretçisi alarak `CRef` nesnesini başlatır. Eğer `pObject` `NULL` değilse, onun `AddReference()` metodunu çağırır.
        *   `CRef(const CRef& c_rRef)`: Kopyalayıcı kurucu. Başka bir `CRef` nesnesini alarak yeni bir `CRef` oluşturur. Kaynak `CRef`'in işaret ettiği nesnenin `AddReference()` metodunu çağırır.
    *   **Yıkıcı Metot (`~CRef()`):**
        *   `Clear()` metodunu çağırır. Bu, `CRef` nesnesi kapsam dışına çıktığında, eğer bir nesneyi işaret ediyorsa o nesnenin `Release()` metodunun çağrılmasını sağlar.
    *   **Atama Operatörleri (`operator=`):**
        *   `void operator=(CReferenceObject* pObject)`: `CRef` nesnesine yeni bir `CReferenceObject` atar. Eski işaret edilen nesnenin `Release()`'ini, yeni nesnenin ise `AddReference()`'ını çağırır.
        *   `void operator=(const CRef& c_rRef)`: Bir `CRef` nesnesini diğerine atar. Benzer şekilde referans sayılarını yönetir.
    *   **Ana Metotlar:**
        *   `Clear()`: `m_pObject`'i `NULL` yapar. Eğer daha önce geçerli bir nesneyi işaret ediyorsa, o nesnenin `Release()` metodunu çağırır.
        *   `IsNull() const`: `m_pObject`'in `NULL` olup olmadığını kontrol eder.
        *   `SetPointer(CReferenceObject* pObject)`: İşaretçiyi `pObject` olarak ayarlar, eski ve yeni nesnelerin referans sayılarını uygun şekilde yönetir.
        *   `T* GetPointer() const`: Saklanan ham işaretçiyi `T*` türüne dönüştürerek döndürür.
        *   `T* operator->() const`: İşaret edilen `T` türündeki nesnenin üyelerine `->` operatörü ile erişim sağlar. İşaretçinin `NULL` olmaması için `assert` kontrolü içerir.
    *   **Yardımcı Yapı (`FClear`):**
        *   `struct FClear`: Bir `CRef<T>` nesnesini parametre olarak alıp onun `Clear()` metodunu çağıran bir fonksiyon nesnesidir (functor). Genellikle `std::for_each` gibi STL algoritmalarıyla bir `CRef` koleksiyonunu temizlemek için kullanılabilir.

*   **Ana Çalışma Prensibi (RAII - Resource Acquisition Is Initialization):**
    *   `CRef<T>` sınıfı, RAII prensibini uygular. Bir `CReferenceObject` işaretçisi `CRef` nesnesine verildiğinde (kurucu veya atama yoluyla), `CRef` bu kaynağın sahibi olur ve referans sayısını artırır.
    *   `CRef` nesnesi yok edildiğinde (örneğin, bir fonksiyon bloğundan çıkıldığında), yıkıcısı otomatik olarak kaynağın referans sayısını azaltır (`Release()` çağrısı).
    *   Eğer referans sayısı sıfıra düşerse, `CReferenceObject`'in kendi `OnSelfDestruct()` metodu tetiklenir ve nesne genellikle kendini bellekten siler.
    *   Bu otomatik yönetim, geliştiricinin manuel olarak `AddReference()` ve `Release()` çağrılarını takip etme yükünü azaltır ve hata yapma olasılığını düşürür.

*   **Kullanım Alanı:**
    *   `CRef<T>`, `EterLib` ve `EterGrnLib` gibi kütüphanelerde `CResource`'dan türeyen kaynakların (örneğin, `CGraphicImage`, `CGrannyModel`) güvenli bir şekilde yönetilmesi için yaygın olarak kullanılır.
    *   Bir fonksiyon bir kaynağa geçici olarak ihtiyaç duyduğunda, kaynağı bir `CRef` nesnesi içinde tutabilir. Fonksiyon bittiğinde `CRef` yok edilir ve kaynağın referansı otomatik olarak düşürülür.
    *   Sınıf üyeleri olarak `CReferenceObject` tabanlı nesneleri tutarken bellek sızıntılarını önlemek için idealdir.
    *   Kısacası, referans sayımıyla yönetilen herhangi bir nesnenin ömrünü kolay ve güvenli bir şekilde yönetmek için kullanılır. 

### `RenderTarget.h` ve `RenderTarget.cpp` (`CRenderTarget` Sınıfı)

*   **Koşullu Derleme:** Bu sınıfla ilgili kodlar, yalnızca `RENDER_TARGET` makrosu tanımlı olduğunda derlenir.
*   **Amaç:** `CRenderTarget` sınıfı, bir 3D modeli (genellikle bir karakter) ve isteğe bağlı bir arka plan resmini, oyun dünyasından ayrı olarak özel bir dokuya (render target texture) render etmek için bir mekanizma sağlar. Bu sonuç dokusu daha sonra genellikle kullanıcı arayüzünde (UI) bir önizleme olarak kullanılır.

*   **`RenderTarget.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CRenderTarget(DWORD dwWidth, DWORD dwHeight)`: Belirtilen genişlik ve yükseklikte bir render hedefi oluşturur.
    *   `~CRenderTarget()`: Yıkıcı metot.
    *   `SetVisibility(bool bShow)`: Render hedefinin görünürlüğünü ve render edilip edilmeyeceğini ayarlar.
    *   `RenderTexture() const`: Oluşturulan render hedefi dokusunu (muhtemelen ekrana veya bir UI elemanına) çizer.
    *   `SetRenderingRect(RECT* pRect) const`: Render hedefi dokusunun çizileceği ekran/alan dikdörtgenini ayarlar.
    *   `SelectModel(DWORD dwIndex)`: Belirtilen `dwIndex` (genellikle bir karakter veya NPC ID'si) ile ilişkili modeli yükler ve render için hazırlar.
    *   `CreateBackground(const char* c_szImgPath, DWORD dwWidth, DWORD dwHeight)`: Belirtilen yoldan bir arka plan resmi yükler ve boyutlandırır.
    *   `RenderBackground() const`: Yüklenmişse arka planı render hedefi dokusuna çizer.
    *   `UpdateModel()`: Modelin animasyonunu ve dönüşünü günceller.
    *   `DeformModel() const`: Modelin deformasyonunu (vertex blending vb.) uygular.
    *   `RenderModel() const`: Seçili modeli render hedefi dokusuna çizer.
    *   `SetZoom(bool bZoom)`: Modele uygulanan zoom seviyesini artırır veya azaltır.
    *   `CreateTextures() const`, `ReleaseTextures() const`: Render hedefiyle ilişkili Direct3D doku kaynaklarını oluşturur ve serbest bırakır.
    *   **Önemli Üyeler:**
        *   `m_pModelInstance`: Render edilecek modelin bir örneğini (`CInstanceBase`) tutan `std::unique_ptr`.
        *   `m_pBackgroundImage`: Arka plan resminin bir örneğini (`CGraphicImageInstance`) tutan `std::unique_ptr`.
        *   `m_pRenderTargetTexture`: Asıl render işleminin yapıldığı hedef dokuyu (`CGraphicRenderTargetTexture`) tutan `std::unique_ptr`.
        *   `m_fModelRotation`: Modelin kendi etrafındaki dönüş açısı.
        *   `m_fEyeY`, `m_fTargetY`, `m_fTargetHeight`, `m_fZoomY`: Modelin render edilmesi için kullanılan kamera ve zoom parametreleri.
        *   `m_bShow`: Render hedefinin gösterilip gösterilmeyeceğini belirten bayrak.

*   **`RenderTarget.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Başlatma:** Kurucu metot, belirtilen boyutlarda bir `CGraphicRenderTargetTexture` oluşturur. Bu doku, D3DFMT_X8R8G8B8 formatında renk ve D3DFMT_D16 formatında derinlik tamponuna sahip olur.
    *   **Model Seçimi ve Hazırlığı (`SelectModel`):**
        *   Verilen `dwIndex` (ırk/NPC ID'si) ile yeni bir `CInstanceBase` (karakter modeli) örneği oluşturulur.
        *   Modelin varsayılan animasyonu (genellikle `CRaceMotionData::NAME_WAIT`) ayarlanır ve sürekli render edilmesi için `SetAlwaysRender(true)` yapılır.
        *   Modelin yüksekliğine göre kamera parametreleri (`m_fEyeY`, `m_fTargetHeight`) hesaplanır.
        *   Modelin render edilmesi için özel bir kamera (`CCameraManager::SHOPDECO_CAMERA`) geçici olarak ayarlanır.
    *   **Arka Plan Yönetimi (`CreateBackground`, `RenderBackground`):**
        *   `CreateBackground`, verilen dosya yolundan bir `CGraphicImage` yükler, bunu bir `CGraphicImageInstance` içine atar ve belirtilen genişlik/yüksekliğe ölçeklendirir.
        *   `RenderBackground`, eğer görünürse (`m_bShow`) ve bir arka plan resmi yüklenmişse, `m_pRenderTargetTexture`'ı aktif render hedefi olarak ayarlar, temizler ve arka plan resmini üzerine çizer.
    *   **Model Render (`UpdateModel`, `DeformModel`, `RenderModel`):**
        *   `UpdateModel`, modelin dönüş açısını günceller ve `Transform()` ile `RotationProcess()` çağırarak modelin matrislerini ve durumunu günceller.
        *   `DeformModel`, modelin vertexlerini animasyona göre deforme eder.
        *   `RenderModel`, eğer görünürse ve model yüklenmişse:
            1.  `m_pRenderTargetTexture`'ı aktif render hedefi olarak ayarlar.
            2.  Eğer arka plan yoksa, hedef dokuyu temizler. Aksi takdirde, arka planın üzerine çizim yapılacağı varsayılır.
            3.  Derinlik tamponunu (`ClearDepthBuffer`) temizler.
            4.  Sis (fog) efektini geçici olarak kapatır.
            5.  `SHOPDECO_CAMERA` adlı özel kamerayı aktif eder.
            6.  Viewport'u render hedefi dokusunun boyutlarına göre ayarlar.
            7.  Modelin pozisyonunu, yönünü ve ölçeğini ayarlar (genellikle merkezde, belirli bir zoom ve rotasyonla).
            8.  Modeli (`m_pModelInstance->Render()`) çizer.
            9.  Önceki kamera ve viewport ayarlarını geri yükler, sis durumunu eski haline getirir.
    *   **Sonuç Render (`RenderTexture`):** `m_pRenderTargetTexture->Render()` çağrısı, `CRenderTarget` üzerinde biriktirilen görüntünün (arka plan + model) nihai olarak (muhtemelen bir UI elemanının üzerine) çizilmesini sağlar.
    *   **Kaynak Yönetimi:** `std::unique_ptr` kullanımı, model, arka plan ve render hedefi dokusu gibi kaynakların ömrünü otomatik olarak yönetir ve sızıntıları önler.

*   **Kullanım Alanı:**
    *   **Karakter Önizleme:** Karakter oluşturma ekranında, envanterde giyilen eşyaların karakter üzerinde nasıl durduğunu göstermek için veya NPC etkileşimlerinde karakterin bir portresini göstermek amacıyla kullanılır.
    *   **Eşya Önizleme:** Mağaza arayüzlerinde veya envanterde 3D eşya modellerini izole bir şekilde render edip kullanıcıya göstermek için kullanılabilir.
    *   **UI Elemanları:** Oyunun ana 3D dünyasından bağımsız olarak, kullanıcı arayüzünün bir parçası olarak dinamik 3D içerik göstermek gerektiğinde kullanılır.
    *   Temelde, bir 3D sahneyi ana render döngüsünden ayrı bir dokuya render etme ve bu dokuyu daha sonra 2D bir arayüzde kullanma ihtiyacı duyulan her senaryoda `CRenderTarget` sınıfı devreye girer. 

### `RenderTargetManager.h` ve `RenderTargetManager.cpp` (`CRenderTargetManager` Sınıfı)

*   **Koşullu Derleme:** Bu sınıfla ilgili kodlar, tıpkı `CRenderTarget` gibi, yalnızca `RENDER_TARGET` makrosu tanımlı olduğunda derlenir.
*   **Amaç:** `CRenderTargetManager` sınıfı, `CSingleton` desenini kullanarak oyundaki tüm `CRenderTarget` nesnelerinin merkezi bir şekilde yönetilmesini sağlar. Bu, birden fazla render hedefinin oluşturulması, güncellenmesi, render edilmesi ve kaynaklarının yönetilmesi işlemlerini kolaylaştırır.

*   **`RenderTargetManager.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CRenderTargetManager()`: Kurucu metot.
    *   `~CRenderTargetManager()`: Yıkıcı metot, `Destroy()` çağırır.
    *   `GetRenderTarget(uint8_t index)`: Belirtilen indekse sahip `CRenderTarget` nesnesine bir `std::shared_ptr` döndürür. Bulamazsa `nullptr` döner.
    *   `CreateRenderTarget(uint8_t index, int width, int height)`: Belirtilen indeks, genişlik ve yükseklik ile yeni bir `CRenderTarget` oluşturur ve yöneticiye kaydeder. Başarısız olursa (örn: indeks geçersiz veya zaten kullanılıyor) `false` döner.
    *   `CreateRenderTargetTextures()`: Yönetilen tüm `CRenderTarget`'ların Direct3D doku kaynaklarını oluşturur.
    *   `ReleaseRenderTargetTextures()`: Yönetilen tüm `CRenderTarget`'ların Direct3D doku kaynaklarını serbest bırakır.
    *   `Destroy()`: Tüm yönetilen `CRenderTarget` nesnelerini ve tuttukları kaynakları temizler.
    *   `DeformModels()`: Yönetilen tüm `CRenderTarget`'lardaki modellerin deformasyonunu günceller.
    *   `UpdateModels()`: Yönetilen tüm `CRenderTarget`'lardaki modellerin durumunu (animasyon, rotasyon vb.) günceller.
    *   `RenderBackgrounds()`: Yönetilen tüm `CRenderTarget`'ların arka planlarını kendi hedef dokularına çizer.
    *   `RenderModels()`: Yönetilen tüm `CRenderTarget`'lardaki modelleri kendi hedef dokularına çizer.
    *   **Önemli Üyeler:**
        *   `m_renderTargets`: `uint8_t` (indeks) ile `std::shared_ptr<CRenderTarget>` arasında eşleme yapan bir `std::unordered_map`. `CRenderTarget` nesnelerini saklar.

*   **`RenderTargetManager.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Render Hedefi Oluşturma ve Saklama:**
        *   `CreateRenderTarget`, yeni bir `CRenderTarget` nesnesini `std::make_shared` ile oluşturur ve bunu `m_renderTargets` haritasına verilen `index` ile ekler. İndeksin 0'dan büyük olması ve daha önce kullanılmamış olması kontrol edilir.
        *   `GetRenderTarget`, haritada verilen indeksi arar ve varsa karşılık gelen `std::shared_ptr<CRenderTarget>`'ı döndürür.
    *   **Toplu İşlemler:**
        *   `DeformModels`, `UpdateModels`, `RenderBackgrounds`, `RenderModels`, `CreateRenderTargetTextures`, `ReleaseRenderTargetTextures` gibi metotlar, `m_renderTargets` haritasındaki tüm `CRenderTarget` örnekleri üzerinde döngüye girerek her birinin ilgili metodunu (örneğin, `elem.second->DeformModel()`) çağırır.
    *   **Kaynak Yönetimi:**
        *   `CRenderTarget` nesneleri `std::shared_ptr` içinde saklandığı için, `m_renderTargets.clear()` çağrıldığında (yani `Destroy()` içinde) veya bir `shared_ptr` kapsam dışına çıktığında/sıfırlandığında, `CRenderTarget`'ların referans sayıları azalır. Referans sayısı sıfıra düştüğünde `CRenderTarget` yıkıcısı çalışır ve kendi içindeki kaynakları (model, arka plan, render hedefi dokusu) `std::unique_ptr`'lar aracılığıyla serbest bırakır.
        *   Bu, `CRenderTargetManager`'ın yönettiği tüm render hedeflerinin kaynaklarının merkezi ve otomatik bir şekilde yönetilmesini sağlar.

*   **Kullanım Alanı:**
    *   Oyun içinde birden fazla bağımsız 3D önizleme alanına ihtiyaç duyulduğunda (örneğin, karakter seçimi ekranında birden fazla karakterin gösterilmesi, farklı UI pencerelerinde farklı eşya veya karakter önizlemeleri) `CRenderTargetManager` kullanılır.
    *   Belirli bir indekse sahip bir `CRenderTarget`'a erişmek, onu oluşturmak veya üzerindeki işlemleri tetiklemek için merkezi bir arayüz sunar.
    *   Oyunun genel durumu değiştiğinde (örneğin, grafik ayarları değiştirildiğinde, çözünürlük değiştiğinde veya oyun kapatılırken) tüm aktif render hedeflerinin doku kaynaklarını toplu olarak yeniden oluşturmak (`CreateRenderTargetTextures`) veya serbest bırakmak (`ReleaseRenderTargetTextures`, `Destroy`) gibi genel yönetim görevleri için idealdir.
    *   Oyunun ana render döngüsünde, `UpdateModels`, `DeformModels`, `RenderBackgrounds`, ve `RenderModels` gibi toplu güncelleme ve render fonksiyonları çağrılarak tüm aktif render hedeflerinin içeriğinin güncel tutulması ve kendi dokularına render edilmesi sağlanır. Bu dokular daha sonra ilgili `CRenderTarget::RenderTexture()` çağrılarıyla ekrana yansıtılır. 

### `ScreenFilter.h` ve `ScreenFilter.cpp` (`CScreenFilter` Sınıfı)

*   **Amaç:** Bu dosyalar, tüm ekranı belirli bir renkle ve alfa (alpha) karıştırma (blending) efektiyle kaplayarak bir filtre uygulamak için kullanılan `CScreenFilter` sınıfını tanımlar ve uygular. Bu sınıf, `CScreen` sınıfından miras alır.

*   **`ScreenFilter.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CScreenFilter()`: Kurucu metot.
    *   `~CScreenFilter()`: Yıkıcı metot.
    *   `SetEnable(BOOL bFlag)`: Filtrenin aktif olup olmayacağını ayarlar.
    *   `SetBlendType(BYTE bySrcType, BYTE byDestType)`: Renk karıştırma için kaynak ve hedef blend faktörlerini ayarlar (örneğin, `D3DBLEND_SRCALPHA`).
    *   `SetColor(const D3DXCOLOR& c_rColor)`: Filtrenin uygulanacağı rengi (RGBA olarak) ayarlar.
    *   `Render()`: Filtreyi ekrana çizer.
    *   **Önemli Üyeler:**
        *   `m_bEnable`: Filtrenin aktif olup olmadığını belirten boolean bayrak.
        *   `m_bySrcType`, `m_byDestType`: Direct3D blend faktörleri.
        *   `m_Color`: Filtre rengi (`D3DXCOLOR`).

*   **`ScreenFilter.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Başlatma:** Kurucu metotta, filtre başlangıçta devre dışı bırakılır (`m_bEnable = false`), varsayılan blend türleri standart alfa karıştırması (`D3DBLEND_SRCALPHA`, `D3DBLEND_INVSRCALPHA`) ve renk tamamen saydam siyah olarak ayarlanır.
    *   **Filtre Kontrolü:**
        *   `SetEnable(BOOL bFlag)`: Bu fonksiyonun mevcut implementasyonunda `m_bEnable` her zaman `false` olarak ayarlanmaktadır. Bu, filtrenin normal kullanımda devre dışı kalacağı anlamına gelir ve bir hata veya kasıtlı bir davranış olabilir.
        *   `SetBlendType` ve `SetColor` metotları, filtrenin karıştırma modunu ve rengini dinamik olarak değiştirmek için kullanılır.
    *   **Render İşlemi (`Render()`):**
        1.  Filtre aktif değilse (`!m_bEnable`) işlem yapılmaz.
        2.  `STATEMANAGER` kullanılarak mevcut projeksiyon ve view transform matrisleri ile bazı render state'leri (alfa karıştırma, kaynak/hedef blend faktörleri) kaydedilir.
        3.  Transform matrisleri identity olarak ayarlanır, böylece ekran koordinatlarında çizim yapılabilir.
        4.  Alfa karıştırma (`D3DRS_ALPHABLENDENABLE`) aktif edilir ve `m_bySrcType` ile `m_byDestType` kullanılarak blend state'leri ayarlanır.
        5.  `SetOrtho2D` ile 2D ortografik projeksiyon ayarlanır.
        6.  `SetDiffuseColor` ile `m_Color` üyesinde saklanan filtre rengi ayarlanır.
        7.  `RenderBar2d(0, 0, CScreen::ms_iWidth, CScreen::ms_iHeight)` çağrılarak tüm ekranı kaplayan, ayarlanan renkte ve blend modunda bir dörtgen çizilir.
        8.  Kaydedilen tüm transform matrisleri ve render state'leri geri yüklenir.

*   **Kullanım Alanı:**
    *   Oyun içinde tam ekran renk efektleri oluşturmak için kullanılır. Örneğin:
        *   **Fade In/Out Efektleri:** Ekranı yavaşça siyaha veya başka bir renge karartmak ya da aydınlatmak için.
        *   **Hasar Bildirimi:** Karakter hasar aldığında kısa süreliğine ekranı kırmızıya boyamak gibi görsel geri bildirimler için.
        *   **Atmosferik Efektler:** Belirli bir bölgede veya durumda (örneğin, zehirlenme) ekrana hafif bir renk tonu vermek için.
    *   `CScreenFilter`, `CScreen`'den türediği için `CScreen`'in temel 2D çizim yeteneklerini kullanır. `SetEnable` fonksiyonundaki mevcut durum, bu filtrenin kullanılmadan önce kodda bir değişiklik yapılmasını gerektirebilir.

### `SkyBox.h` ve `SkyBox.cpp` (`CSkyBox` ve İlgili Sınıflar)

*   **Amaç:** Bu dosyalar, oyun dünyasının arka planını oluşturan, dinamik renk geçişlerine ve hareketli bulutlara sahip olabilen bir gökyüzü kutusu (skybox) sistemini tanımlar ve uygular.

*   **Temel Sınıflar ve Yapılar (`SkyBox.h`):**
    *   **`SColor`, `TGradientColor`, `TVectorGradientColor`**: Renkleri ve bir başlangıç-bitiş rengi arasındaki geçişleri tanımlayan temel veri yapılarıdır.
    *   **`CSkyObjectQuad`**: Skybox yüzeylerini veya bulut katmanlarını oluşturan bireysel dörtgenleri (quads) temsil eder. Her bir köşesi için renk ve zamanla renk geçişi (`CColorTransitionHelper` kullanarak) yeteneklerine sahiptir.
    *   **`CSkyObject`**: `CSkyBox` için soyut bir temel sınıftır. Skybox'ın genel pozisyonunu, bulut katmanının temel özelliklerini (doku, ölçek, kaydırma hızı), render modlarını ve doku yönetimini içerir. Türetilmiş sınıfların `Destroy`, `Render`, `Update` gibi fonksiyonları implemente etmesini bekler.
    *   **`CSkyBox`**: `CSkyObject`'ten miras alır ve asıl skybox mantığını içerir. Skybox'ın altı yüzünü (`m_Faces`), bu yüzlerin dokularını, dikey renk geçişlerini ve bulut katmanının detaylı ayarlarını yönetir.

*   **`SkyBox.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**

    *   **`CSkyObjectQuad` Uygulaması:**
        *   Dörtgenin köşe renklerini ve bu renkler arasındaki yumuşak geçişleri `CColorTransitionHelper` aracılığıyla yönetir.
        *   `Render` metodu, `TPDTVertex` formatındaki köşe verilerini kullanarak dörtgeni çizer.

    *   **`CSkyObject` Uygulaması:**
        *   Skybox'ı kamera pozisyonuyla senkronize ederek kamerayla birlikte hareket etmesini sağlar (`Update` metodu).
        *   Doku yükleme (`GenerateTexture`) ve silme (`DeleteTexture`) için `CResourceManager`'ı kullanır.
        *   `TSkyObjectFace` iç yapısı, bir yüzü oluşturan `CSkyObjectQuad`'ların güncellenmesi ve render edilmesi işlemlerini toplu halde yapar.

    *   **`CSkyBox` Uygulaması:**
        *   **Geometri Oluşturma (`Refresh`):** Skybox'ın altı yüzünü oluşturan dörtgenlerin (quads) köşe pozisyonlarını, normallerini ve doku koordinatlarını `SetSkyObjectQuadVertical` ve `SetSkyObjectQuadHorizon` yardımcı fonksiyonları aracılığıyla hesaplar. Bu, skybox'ın kübik yapısını oluşturur.
        *   **Doku Yönetimi:** Her bir yüz için (`SetFaceTexture`) ve bulut katmanı için (`SetCloudTexture`) ayrı dokular yüklenebilir. Bu dokular `m_GraphicImageInstanceMap` içinde saklanır.
        *   **Renk ve Geçişler:**
            *   `SetSkyColor`: Skybox yüzeylerinin köşe renkleri ve bu renkler arasındaki geçişler ayarlanır. `TVectorGradientColor` kullanılarak karmaşık renk desenleri oluşturulabilir.
            *   `SetCloudColor`: Bulut katmanının renk geçişleri ayarlanır.
            *   `StartTransition`: Ayarlanan tüm renk geçişlerini başlatır.
        *   **Bulut Sistemi:**
            *   Ayrı bir bulut katmanı (`m_FaceCloud`) yönetilir.
            *   Bulutların doku ölçeği (`SetCloudTextureScale`), genel ölçeği (`SetCloudScale`), yerden yüksekliği (`SetCloudHeight`) ve kaydırma hızı (`SetCloudScrollSpeed`) ayarlanabilir.
            *   `Update` metodunda bulut doku koordinatları kaydırma hızına göre güncellenerek hareket efekti verilir.
        *   **Güncelleme (`Update`):**
            *   Skybox'ın pozisyonunu günceller (kamerayı takip eder).
            *   Tüm renk geçişlerini (skybox yüzleri ve bulutlar) günceller.
            *   Bulutların kayma hareketini günceller.
        *   **Render İşlemleri:**
            *   `Render()`: Skybox'ın altı yüzünü çizer. Her yüz için, atanmış bir doku varsa dokulu (`SKY_RENDER_MODE_TEXTURE`), yoksa sadece vertex renkleriyle (`SKY_RENDER_MODE_DIFFUSE`) çizim yapar. Render sırasında Z-buffer genellikle devre dışı bırakılır ve skybox en geride çizilir.
            *   `RenderCloud()`: Bulut katmanını çizer. Alfa karıştırma (alpha blending) kullanılır ve bulutlar genellikle skybox yüzeylerinin üzerinde çizilir. Doku matrisi, bulutların kayma efektini yansıtacak şekilde güncellenir.
        *   **Kaynak Temizleme (`Destroy`, `Unload`):** Yüklenen tüm dokuları ve diğer kaynakları serbest bırakır.

*   **Kullanım Alanı:**
    *   Oyun dünyasının gökyüzünü ve genel atmosferini oluşturmak için temel bir bileşendir.
    *   Farklı zaman dilimlerini (gündüz, gece, gün batımı vb.) veya hava durumlarını (açık, bulutlu) yansıtmak için dinamik renk geçişleri ve farklı bulut dokuları kullanılabilir.
    *   Hareket eden bulutlar sayesinde daha canlı ve gerçekçi bir gökyüzü efekti sağlar.
    *   Skybox, genellikle kameranın etrafında sabit bir mesafede kalacak şekilde güncellenir, böylece oyuncu hareket ettikçe sonsuz bir gökyüzü illüzyonu yaratılır.

### `TargaResource.h` ve `TargaResource.cpp` (`CTargaResource` Sınıfı)

*   **Amaç:** Bu dosyalar, Targa (.tga) formatındaki resim dosyalarını bir `EterLib` kaynağı (`CResource`) olarak yüklemek, yönetmek ve bu resimlerin verilerine erişim sağlamak için `CTargaResource` sınıfını tanımlar ve uygular.

*   **`TargaResource.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CTargaResource`: TGA resim kaynağını temsil eden ana sınıf.
        *   Kalıtım: `public CResource`.
        *   `TRef`: `CRef<CTargaResource>` için bir tür takma adı (typedef), akıllı işaretçi kullanımını kolaylaştırır.
        *   `static TType Type()`: Bu kaynak sınıfının türünü ("CTargaResource") döndürür.
        *   `CTargaResource(const char* c_pszFileName)`: Kurucu metot, dosya adını alır.
        *   `~CTargaResource()`: Yıkıcı metot.
        *   `GetMemPtr()`: Yüklenen TGA resminin ham piksel verilerine (DWORD işaretçisi) erişim sağlar.
        *   `GetRect(DWORD& w, DWORD& h)`: Resmin genişliğini (`w`) ve yüksekliğini (`h`) alır.
        *   `GetTgaHeader()`: TGA dosyasının başlık bilgilerini (`TGA_HEADER` yapısı) döndürür.
    *   **Korumalı Sanal Metotlar (Override):**
        *   `OnLoad(int iSize, const void* c_pvBuf)`: Asıl TGA yükleme işlemini gerçekleştirir.
        *   `OnClear()`: Kaynak temizlendiğinde çağrılır.
        *   `OnIsEmpty() const`: Kaynağın boş (veri içermiyor) olup olmadığını kontrol eder.
        *   `OnIsType(TType type)`: Kaynağın belirtilen türde olup olmadığını kontrol eder.
    *   **Önemli Üyeler:**
        *   `image`: TGA resim verilerini ve işlevlerini içeren bir `CTGAImage` nesnesi.

*   **`TargaResource.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Bağımlılık:** `CTargaResource`, TGA dosyalarını işlemek için `EterImageLib` kütüphanesindeki `CTGAImage` sınıfını kullanır.
    *   **Yükleme (`OnLoad`):** Verilen bellek tamponundaki (`c_pvBuf`) veriyi kullanarak `m_image.LoadFromMemory()` çağrısıyla TGA resmini yükler.
    *   **Temizleme (`OnClear`):** `m_image.Clear()` çağrısıyla iç `CTGAImage` nesnesinin kaynaklarını serbest bırakır.
    *   **Durum Kontrolü (`OnIsEmpty`):** `m_image.IsEmpty()` ile `CTGAImage` nesnesinin geçerli bir resim verisi içerip içermediğini kontrol eder.
    *   **Veri Erişimi:** `GetMemPtr`, `GetRect`, `GetTgaHeader` gibi metotlar, iç `CTGAImage` nesnesinin ilgili metotlarını çağırarak TGA verilerine erişim sağlar.

*   **Kullanım Alanı:**
    *   `CTargaResource`, TGA formatındaki resim dosyalarının `CResourceManager` tarafından standart bir kaynak olarak yönetilebilmesini sağlar.
    *   Genellikle `.tga` uzantılı dosyalar için `CResourceManager`'a kaydedilir ve istendiğinde dosya adıyla yüklenir.
    *   Yüklendikten sonra, bu sınıf aracılığıyla TGA resminin ham piksel verilerine, boyutlarına ve başlık bilgilerine erişilebilir.
    *   Elde edilen ham piksel verileri, daha sonra bir `CGraphicImage` nesnesine yüklenerek veya doğrudan bir Direct3D dokusu (`LPDIRECT3DTEXTURE8`) oluşturularak grafiksel işlemlerde kullanılabilir. Bu sınıf, TGA verilerini okuma ve temel bilgilere erişim katmanını sağlar, ancak doğrudan render edilebilir bir grafik nesnesi değildir.

### `TextBar.h` ve `TextBar.cpp` (`CTextBar` Sınıfı)

*   **Amaç:** Bu dosyalar, bir `CDibBar` (Device Independent Bitmap Bar - muhtemelen bir bellek içi bitmap üzerine GDI çizimi yapmak için bir sarmalayıcı) üzerine GDI (Graphics Device Interface) fonksiyonlarını kullanarak metin çizmek için `CTextBar` sınıfını tanımlar ve uygular.

*   **`TextBar.h` - Temel İşlevler ve Tanımlamalar:**
    *   `CTextBar`: GDI metin çizim yeteneklerini `CDibBar` üzerine ekleyen ana sınıf.
        *   Kalıtım: `public CDibBar`.
        *   `CTextBar(int fontSize, bool isBold)`: Kurucu metot. Font boyutunu ve kalınlık durumunu alır.
        *   `~CTextBar()`: Yıkıcı metot.
        *   `TextOut(int ix, int iy, const char* c_szText)`: Belirtilen koordinatlara metin çizer.
        *   `SetTextColor(int r, int g, int b)`: Metin rengini ayarlar.
        *   `GetTextExtent(const char* c_szText, SIZE* p_size)`: Verilen metnin geçerli font ile boyutlarını hesaplar.
    *   **Korumalı Metotlar:**
        *   `__SetFont(int fontSize, bool isBold)`: Belirtilen özelliklere göre bir GDI fontu oluşturur ve ayarlar.
        *   `OnCreate()`: Sınıf oluşturulduğunda çağrılan başlatma fonksiyonu (muhtemelen `CDibBar`'dan sanal).
    *   **Önemli Üyeler:**
        *   `m_hFont`: Oluşturulan GDI fontunun tanıtıcısı (`HFONT`).
        *   `m_hOldFont`: `m_hFont` seçilmeden önceki orijinal fontun tanıtıcısı.
        *   `m_fontSize`: Kullanılacak fontun boyutu.
        *   `m_isBold`: Fontun kalın olup olmayacağı.

*   **`TextBar.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Font Oluşturma ve Yönetimi (`__SetFont`, `OnCreate`, Kurucu, Yıkıcı):**
        *   `__SetFont`, `LOGFONT` yapısını kullanarak istenen boyut, kalınlık, karakter seti (varsayılan kod sayfasına göre `GetCharsetFromCodePage`) ve font yüzü (varsayılan kod sayfasına göre `GetFontFaceFromCodePage`) ile bir GDI fontu oluşturur (`CreateFontIndirect`).
        *   Oluşturulan font, `CDibBar`'ın Device Context'ine (`m_dib.GetDCHandle()`) `SelectObject` ile seçilir. Eski font saklanır.
        *   `OnCreate` içinde, `m_dib.SetBkMode(TRANSPARENT)` ile metin çiziminin arka planı saydam yapılır ve `__SetFont` çağrılır.
        *   Yıkıcıda, DC'ye orijinal font geri yüklenir.
    *   **Metin İşlemleri:**
        *   `SetTextColor`, `::SetTextColor` GDI fonksiyonunu kullanarak metin rengini ayarlar.
        *   `TextOut`, metni `m_dib.TextOut()` (muhtemelen `CDibBar` içindeki bir metot) aracılığıyla DIB üzerine çizer ve DIB'in güncellenmesi için `Invalidate()` çağırır.
        *   `GetTextExtent`, `GetTextExtentPoint32` GDI fonksiyonunu kullanarak metnin piksel cinsinden genişliğini ve yüksekliğini hesaplar.

*   **Kullanım Alanı:**
    *   `CTextBar` sınıfı, dinamik olarak oluşturulan metinleri bir bellek içi bitmap'e (DIB) render etmek için kullanılır.
    *   Oyun içinde anlık olarak değişen metinlerin (örneğin, oyuncu isimleri, can barları üzerindeki sayılar, sohbet mesajları, debug bilgileri) bir resim yüzeyine aktarılması gereken durumlarda kullanışlıdır.
    *   Bu DIB daha sonra bir Direct3D dokusuna dönüştürülerek oyun dünyasında 3D bir nesne üzerinde veya kullanıcı arayüzünde 2D bir eleman olarak gösterilebilir. Temelde, GDI'nin metin render etme yeteneklerini, doğrudan Direct3D'ye metin çizmek yerine bir ara bitmap yüzeyi üzerinden kullanmayı sağlar.
    *   Font seçimi için `EterLib/Util.h` içindeki kod sayfası ve font yüzü belirleme fonksiyonlarından faydalanır.

### `TextFileLoader.h` ve `TextFileLoader.cpp` (`CTextFileLoader` Sınıfı)

*   **Amaç:** Bu dosyalar, hiyerarşik bir yapıya sahip (iç içe gruplar içeren) metin tabanlı yapılandırma dosyalarını yüklemek, ayrıştırmak ve bu dosyalardaki verilere kolayca erişmek için `CTextFileLoader` sınıfını tanımlar ve uygular.

*   **`TextFileLoader.h` - Temel Yapılar ve Tanımlamalar:**
    *   **`SGroupNode` (veya `TGroupNode`) Yapısı:**
        *   Dosya içindeki bir "grup" veya "blok"u temsil eder. Her düğümün bir adı vardır.
        *   İçinde anahtar-değer çiftleri şeklinde token'ları saklar (anahtar `std::string`, değer `CTokenVector` - yani `std::vector<std::string>`).
        *   Hiyerarşiyi sağlamak için ebeveyn (`pParentNode`) ve çocuk (`ChildNodeVector`) düğüm işaretçilerine sahiptir.
        *   Nesne havuzlaması (`CDynamicPool`) ile yönetilir.
    *   **`CGotoChild` Sınıfı:**
        *   RAII deseniyle, `CTextFileLoader` içinde geçici olarak bir alt düğüme geçip, kapsam sonunda otomatik olarak ebeveyn düğüme dönmeyi kolaylaştıran bir yardımcı sınıftır.
    *   **`CTextFileLoader` Sınıfı:**
        *   `Load(const char* c_szFileName)`: Belirtilen dosyayı yükler ve ayrıştırır.
        *   Düğüm Gezinme Metotları: `SetTop`, `GetChildNodeCount`, `SetChildNode` (isim veya indeks ile), `SetParentNode`, `GetCurrentNodeName`.
        *   Token Erişim Metotları: Belirli bir anahtara sahip token'ları çeşitli veri türlerinde (bool, byte, int, float, string, D3DXVECTOR, D3DXCOLOR vb.) almak için çok sayıda `GetTokenTYPE` metodu sunar.
        *   Önbellekleme: `SetCacheMode` ve `Cache` metotları ile yüklenen dosyaların önbelleğe alınmasını destekler.

*   **`TextFileLoader.cpp` - Ana Çalışma Prensibi ve Implementasyon Detayları:**
    *   **Dosya Okuma ve Ayrıştırma (`Load`, `LoadGroup`):**
        *   `Load` metodu, `CEterPackManager` aracılığıyla dosyayı bir bellek tamponuna yükler.
        *   `LoadGroup` adlı rekürsif fonksiyon, metin dosyasını satır satır işler.
        *   `CMemoryTextFileLoader::SplitLine2` kullanarak her satırı boşluklara göre token'lara ayırır.
        *   `{` ve `}` karakterlerini grup (blok) başlangıcı ve bitişi olarak yorumlar.
        *   `group <grup_adı>` satırlarıyla yeni `SGroupNode`'lar oluşturarak hiyerarşik bir ağaç yapısı kurar.
        *   Diğer satırları anahtar-değer (ilk token anahtar, geri kalanlar değerler vektörü) olarak yorumlayıp mevcut `SGroupNode` içine kaydeder.
    *   **Düğüm Yönetimi (`SGroupNode`):**
        *   Grup adları küçük harfe çevrilir ve CRC32 hash'i (`GenNameKey`) anahtar olarak kullanılır.
        *   `SGroupNode` örnekleri, performans için `CDynamicPool` (nesne havuzu) kullanılarak oluşturulur ve silinir.
    *   **Token Dönüşümü (`GetTokenTYPE` metotları):**
        *   İstenen token bulunduğunda, `CTokenVector` içindeki string değerler `atoi`, `atof`, `strtoul`, `D3DXVecXFromString`, `sscanf` gibi standart C/C++ ve D3DX fonksiyonları kullanılarak hedef veri türüne dönüştürülür.
    *   **Önbellekleme (`Cache`, `ms_isCacheMode`):**
        *   Eğer önbellekleme modu (`ms_isCacheMode`) aktifse, `Cache` fonksiyonu daha önce yüklenmiş bir `CTextFileLoader` örneğini dosya adı hash'ine göre statik bir haritadan (`ms_kMap_dwNameKey_pkTextFileLoader`) alıp tekrar kullanır. Aktif değilse veya ilk yüklemedeyse, yeniden yükler veya yeni örnek oluşturur.

*   **Kullanım Alanı:**
    *   `CTextFileLoader`, istemci tarafında çeşitli yapılandırma dosyalarını, script benzeri dosyaları veya hiyerarşik veri içeren metin dosyalarını okumak için kullanılır. Örneğin:
        *   Karakterlerin, NPC'lerin veya eşyaların özelliklerini tanımlayan dosyalar.
        *   Görev (quest) scriptleri veya diyaloglar.
        *   Kullanıcı arayüzü (UI) elemanlarının pozisyonlarını, boyutlarını veya özelliklerini belirleyen dosyalar.
        *   Efektlerin veya animasyonların sıralamasını ve parametrelerini içeren dosyalar.
    *   Sağladığı hiyerarşik gezinme ve çeşitli türlerde veri okuma yetenekleri sayesinde karmaşık metin tabanlı veri formatlarının kolayca işlenmesini mümkün kılar.
    *   `CGotoChild` sınıfı, belirli bir grup içindeki verilere erişirken kodun okunurluğunu artırır.

### `TextTag.h` ve `TextTag.cpp` (Metin Etiketi İşleme Fonksiyonları)

*   **Amaç:** Bu dosyalar, metinler içinde özel anlamlar taşıyan etiketleri (örneğin, renk değiştirme, köprü oluşturma, resim ekleme) tanımak, ayrıştırmak ve işlemek için kullanılan bir dizi fonksiyon ve bir enum tanımlar ve uygular.

*   **`TextTag.h` - Tanımlar ve Bildirimler:**
    *   **`enum` (İsimsiz):** Metin etiketlerinin türlerini tanımlar:
        *   `TEXT_TAG_PLAIN`: Normal metin.
        *   `TEXT_TAG_TAG`: `||` (düz `|` karakterini temsil eder).
        *   `TEXT_TAG_COLOR`: Renk etiketi (`|cRRGGBBAA`).
        *   `TEXT_TAG_HYPERLINK_START`, `TEXT_TAG_HYPERLINK_END`: Köprü başlangıç (`|H`) ve bitiş (`|h`) etiketleri.
        *   `TEXT_TAG_RESTORE_COLOR`: Önceki renge dönme etiketi (`|r`).
        *   `TEXT_TAG_IMAGE_START`, `TEXT_TAG_IMAGE_END` (`ENABLE_TEXT_IMAGE_LINE` ile): Resim başlangıç (`|I`) ve bitiş (`|i`) etiketleri.
    *   **Harici Fonksiyon Bildirimleri (`extern`):**
        *   `GetTextTag`: Bir sonraki etiketi ve bilgilerini alır.
        *   `GetTextTagOutputString`: Etiketleri işleyerek/kaldırarak render edilecek metni oluşturur.
        *   `GetTextTagOutputLen`: Render edilecek metnin uzunluğunu hesaplar.
        *   `FindColorTagEndPosition`, `FindColorTagStartPosition`: Renk etiketlerinin başlangıç/bitiş pozisyonlarını bulur.
        *   `GetTextTagInternalPosFromRenderPos`: Render edilmiş pozisyondan orijinal metin pozisyonuna çevrim yapar.

*   **`TextTag.cpp` - Uygulama Detayları:**
    *   **`GetTextTag`:** Metin akışında `|` karakterini arar ve sonraki karaktere göre etiket türünü belirler. Renk etiketleri için renk kodunu `extraInfo`'ya çıkarır.
    *   **`GetTextTagOutputString` ve `GetTextTagOutputLen`:** Metni etiketlere göre ayrıştırır. Renk ve geri yükleme etiketlerini atlarken, `TEXT_TAG_PLAIN` ve `TEXT_TAG_TAG` karakterlerini sayar/çıktıya ekler. Ancak, köprü (`|H...|h`) veya resim (`|I...|i`) etiketleri arasındaki metinleri yok sayar (render edilmez/sayılmaz).
    *   **`FindColorTagStartPosition` ve `FindColorTagEndPosition`:** Belirli bir pozisyondan başlayarak geriye veya ileriye doğru gidip ilgili renk etiketinin (`|c` veya `|r`) yerini bulmaya çalışır. Arapça kod sayfası için özel bir mantık içerir.
    *   **`GetTextTagInternalPosFromRenderPos`:** Render edilmiş metindeki bir pozisyona (örneğin, fare tıklamasıyla elde edilen karakter indeksi) karşılık gelen, etiketler dahil orijinal metindeki pozisyonu bulur. Bu, özellikle tıklanan metnin hangi etikete ait olduğunu (örneğin, hangi köprüye tıklandığını) belirlemek için kullanışlıdır.

*   **Kullanım Alanı:**
    *   Bu fonksiyonlar, oyun içinde metinlerin zengin biçimlendirme ile gösterilmesi için kullanılır.
    *   **Renklendirme:** Eşya isimleri, NPC isimleri, sistem mesajları gibi metinlerin belirli bölümlerini farklı renklerde göstermek için `|cRRGGBBAA` ve `|r` etiketleri kullanılır.
    *   **Köprüler (Hyperlinks):** Metin içinde tıklanabilir alanlar oluşturmak için `|H...|h` etiketleri kullanılır. Örneğin, bir eşya ismine tıklandığında eşya bilgi penceresinin açılması bu şekilde sağlanabilir. Köprü etiketi genellikle `|H<tür>:<veri>|h<görünen_metin>|h` formatındadır.
    *   **Satır İçi Resimler (`ENABLE_TEXT_IMAGE_LINE` ile):** Metin arasına küçük ikonlar veya resimler eklemek için `|I...|i` etiketleri kullanılabilir.
    *   Metin renderlama motorları, bu fonksiyonları kullanarak metni ayrıştırır, etiketlere göre renkleri veya diğer durumları ayarlar ve sadece render edilmesi gereken karakterleri ekrana çizer.

### `Thread.h` ve `Thread.cpp` (`CThread` Sınıfı)

*   **Amaç:** Bu dosyalar, Windows üzerinde yeni bir iş parçacığı (thread) oluşturmak ve yönetmek için soyut bir temel sınıf olan `CThread`'ı tanımlar ve uygular. Belirli bir görevi ana iş parçacığından ayrı olarak çalıştırmak için bir altyapı sunar.

*   **`Thread.h` - Tanımlar ve Bildirimler:**
    *   `CThread`: Soyut thread temel sınıfı.
        *   `CThread()`: Kurucu metot.
        *   `Create(void* arg)`: Yeni bir thread oluşturur ve başlatır. `arg` parametresi thread'in `Execute` metoduna geçirilir.
    *   **Korumalı Metotlar:**
        *   `static UINT CALLBACK EntryPoint(void* pThis)`: `_beginthreadex` için gereken statik giriş noktası fonksiyonu.
        *   `virtual UINT Setup() = 0`: Thread çalışmaya başlamadan önce çağrılan sanal başlatma fonksiyonu. Türetilmiş sınıf implemente etmelidir.
        *   `virtual UINT Execute(void* arg) = 0`: Thread'in ana iş mantığını içeren sanal fonksiyon. Türetilmiş sınıf implemente etmelidir.
        *   `UINT Run(void* arg)`: `Setup` ve `Execute`'u çağıran dahili çalıştırma fonksiyonu.
        *   `Arg() const`, `Arg(void* arg)`: Thread argümanını almak ve ayarlamak için.
    *   **Üyeler:**
        *   `m_hThread`: Oluşturulan thread'in Windows tanıtıcısı (`HANDLE`).
        *   `m_pArg`: Thread'e `Create` ile geçirilen ve `Execute`'a aktarılan argüman işaretçisi.
        *   `m_uThreadID`: Oluşturulan thread'in kimlik numarası (ID).

*   **`Thread.cpp` - Uygulama Detayları:**
    *   **Thread Oluşturma (`Create`):**
        *   `_beginthreadex` Windows API fonksiyonunu kullanarak yeni bir iş parçacığı başlatır.
        *   `EntryPoint` fonksiyonunu thread'in başlangıç adresi olarak verir.
        *   `this` işaretçisini ve `arg` parametresini `EntryPoint`'a argüman olarak geçirir.
        *   Başarılı olursa thread tanıtıcısını (`m_hThread`) saklar ve thread önceliğini `THREAD_PRIORITY_NORMAL` olarak ayarlar.
    *   **Thread Çalışma Akışı (`EntryPoint`, `Run`):**
        *   Yeni thread başladığında `EntryPoint` çağrılır.
        *   `EntryPoint`, `CThread` nesnesinin `Run` metodunu çağırır.
        *   `Run`, önce türetilmiş sınıfın implemente ettiği `Setup()` metodunu çağırır.
        *   `Setup()` başarılı olursa (0 dışında bir değer dönerse), `Run` türetilmiş sınıfın implemente ettiği `Execute(arg)` metodunu çağırarak asıl thread görevini başlatır.
        *   `Execute` metodunun dönüş değeri, thread'in çıkış kodu olur.

*   **Kullanım Alanı:**
    *   `CThread` sınıfı, ana uygulama akışını engellemeden arka planda belirli görevleri yürütmek için kullanılır.
    *   Örneğin:
        *   **Dosya G/Ç:** Büyük dosyaları okuma veya yazma işlemleri.
        *   **Ağ İletişimi:** Ağdan veri alma veya gönderme işlemleri (bazı `CNetworkStream` benzeri yapılar thread kullanabilir).
        *   **Arka Plan Hesaplamaları:** Karmaşık veya zaman alan hesaplamalar.
    *   Kullanmak için:
        1.  `CThread` sınıfından yeni bir sınıf türetilir.
        2.  `Setup()` ve `Execute(void* arg)` sanal metotları override edilir.
        3.  `Setup()` içine thread başlamadan yapılması gereken hazırlık kodları yazılır.
        4.  `Execute()` içine asıl thread iş mantığı (örneğin, bir döngü içinde dosya okuma) yazılır.
        5.  Türetilmiş sınıfın bir örneği oluşturulur ve `Create(arg)` metodu çağrılarak thread başlatılır.

### `lineintersect_utils.h` ve `lineintersect_utils.cpp` (Doğru Kesişim Yardımcıları)

*   **Kaynak:** Bu kod, Graham Rhodes tarafından yazılan ve "Game Programming Gems II" kitabında yer alan "Fast, Robust Intersection of 3D Line Segments" bölümüne dayanmaktadır.
*   **Amaç:** Bu dosyalar, 3D uzaydaki iki doğru parçasının (line segment) kesişimini veya birbirine en yakın noktalarını sağlam ve hızlı bir şekilde hesaplamak için kullanılan bir dizi yardımcı fonksiyon tanımlar ve uygular.

*   **`lineintersect_utils.h` - Fonksiyon Bildirimleri:**
    *   **`IntersectLineSegments(...)` (çeşitli overload'lar):** İki doğru parçasını (A1-A2, B1-B2) alır. Opsiyonel olarak doğruların sonsuz kabul edilip edilmeyeceğini ve bir tolerans değeri (`epsilon`) alabilir. Çıktı olarak, her segment üzerindeki diğerine en yakın noktaları, isteğe bağlı olarak segmentler arasındaki en yakın genel noktayı/vektörü ve gerçek bir kesişim olup olmadığını verir.
    *   **`FindNearestPointOnLineSegment(...)`:** Verilen bir noktanın (B) bir doğru parçasına (A1-A2) en yakın noktasını bulur.
    *   **`FindNearestPointOfParallelLineSegments(...)`:** İki paralel doğru parçasının birbirine en yakın noktalarını bulur.
    *   **`AdjustNearestPoints(...)`:** Sonsuz doğrular için hesaplanan en yakın nokta parametrelerini (`s`, `t`), sonlu doğru parçaları için geçerli olacak şekilde ayarlar.

*   **`lineintersect_utils.cpp` - Uygulama Detayları:**
    *   **Matematiksel Temel:** Fonksiyonlar, vektör cebiri (nokta çarpımı, vektör çıkarma, uzunluk karesi vb.) ve parametrik doğru denklemlerini kullanır. İki doğru arasındaki en yakın noktaları bulmak için genellikle doğrusal denklem sistemleri çözülür.
    *   **Sağlamlık ve Özel Durumlar:** Kod, sayısal kararlılık ve sağlamlık için tasarlanmıştır. Şu özel durumları ele alır:
        *   Bir veya iki doğru parçasının çok kısa (dejenere) olması.
        *   Doğru parçalarının paralel olması (bu durumda özel bir mantıkla temsilci en yakın noktalar bulunur).
    *   **Parametre Hesaplama ve Sınırlama:** İki doğruya en yakın noktalara karşılık gelen parametreler (`s` ve `t`) hesaplanır. Eğer doğrular sonlu segmentler olarak ele alınıyorsa, bu parametreler `[0, 1]` aralığına sıkıştırılır. Eğer orijinal parametreler bu aralığın dışındaysa, en yakın nokta segmentin uç noktalarından birindedir ve buna göre ayarlamalar yapılır (`AdjustNearestPoints`).
    *   **Tolerans (`epsilon`, `MY_EPSILON`):** Kayan nokta hatalarını yönetmek ve "neredeyse paralel" veya "neredeyse kesişen" durumları belirlemek için küçük bir tolerans değeri kullanılır.

*   **Kullanım Alanı:**
    *   Bu fonksiyonlar, 3D uzayda iki doğru parçasının geometrik ilişkisini hassas bir şekilde belirlemek için kullanılır.
    *   **Çarpışma Tespiti (Collision Detection):** Özellikle ışın (`CRay`) veya hareket eden küçük nesneler ile diğer nesneler veya çevre arasındaki kesişimleri test etmek için kullanılabilir.
    *   **Yapay Zeka (AI):** Görüş hattı (line-of-sight) kontrollerinde, iki nokta arasındaki yolun engellenip engellenmediğini doğruların kesişimine bakarak test etmek için kullanılabilir.
    *   **Geometrik Sorgulamalar:** İki nesnenin yörüngesinin kesişip kesişmediğini veya birbirlerine ne kadar yaklaştıklarını belirlemek gibi genel 3D geometri problemlerinde kullanılır.
    *   Sağladığı sağlamlık, özellikle kayan nokta hatalarına duyarlı olabilecek kesişim hesaplamalarında önemlidir.

</rewritten_file> 