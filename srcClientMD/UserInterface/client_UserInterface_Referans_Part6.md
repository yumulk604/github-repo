# UserInterface Referans Kılavuzu - Bölüm 6

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 5'te ele alınan konuların ardından, bu bölümde kalan C++ sınıfları, Python modülleri ve diğer ilgili bileşenler incelenecektir.

## İçindekiler

* [`ProcessScanner.h` ve `ProcessScanner.cpp` (Süreç Tarayıcı)](#processscannerh-ve-processscannercpp-süreç-tarayıcı)
* [`PythonAcce.h` ve `PythonAcce.cpp` (Kuşak Bonus Emilim Sistemi - Acce)](#pythonacceh-ve-pythonaccecpp-kuşak-bonus-emilim-sistemi---acce)
* [`PythonApplication.h` ve `PythonApplication.cpp` (Ana İstemci Uygulaması)](#pythonapplicationh-ve-pythonapplicationcpp-ana-istemci-uygulaması)
* [`PythonApplicationCamera.cpp` (Kamera Yönetimi Uygulaması)](#pythonapplicationcameracpp-kamera-yönetimi-uygulaması)
* [`PythonApplicationCursor.cpp` (Fare İmleci Yönetimi)](#pythonapplicationcursorcpp-fare-imleci-yönetimi)
* [`PythonApplicationEvent.cpp` (Uygulama Olay Yönetimi)](#pythonapplicationeventcpp-uygulama-olay-yönetimi)
* [`PythonApplicationLogo.cpp` (Başlangıç Logosu Oynatımı)](#pythonapplicationlogocpp-başlangıç-logosu-oynatımı)
* [`PythonApplicationModule.cpp` (Python `app` Modülü) →](client_UserInterface_Referans_Part7.md#pythonapplicationmodulecpp-python-app-modülü)

---

### `ProcessScanner.h` ve `ProcessScanner.cpp` (Süreç Tarayıcı)

Bu dosyalar, Metin2 istemcisinin arka planda çalışan diğer süreçleri periyodik olarak taramasını, bu süreçlerin yürütülebilir dosyalarının CRC32 değerlerini hesaplamasını ve bu bilgileri ana istemci kodunun erişebileceği bir kuyruğa eklemesini sağlayan bir mekanizma oluşturur. Temel amaç, bilinen hile programlarını veya diğer istenmeyen üçüncü parti uygulamaları tespit etmeye yardımcı olmaktır.

#### Dosyaların Genel Amaçları

*   **`ProcessScanner.h`**:
    *   `CRCPair` (`std::pair<DWORD, std::string>`) türünü tanımlar: Bir sürecin CRC32 değerini ve tam dosya yolunu bir arada tutar.
    *   Genel arayüz fonksiyonlarının prototiplerini deklare eder: `ProcessScanner_Create`, `ProcessScanner_Destroy`, `ProcessScanner_ReleaseQuitEvent`, `ProcessScanner_PopProcessQueue`.
*   **`ProcessScanner.cpp`**:
    *   Yukarıda belirtilen arayüz fonksiyonlarının implementasyonunu içerir.
    *   Arka planda çalışan bir thread (`ProcessScanner_Thread`) oluşturur ve yönetir. Bu thread, düzenli aralıklarla sistemdeki tüm süreçleri tarar (`ScanProcessList`).
    *   `ScanProcessList` fonksiyonu, Windows API (`CreateToolhelp32Snapshot`, `Process32First/Next`, `Module32First/Next`, `OpenProcess`) kullanarak süreçleri ve bu süreçlere ait modülleri listeler, her modülün dosya yolunu alır ve `GetFileCRC32` ile CRC'sini hesaplar.
    *   Hesaplanan CRC'ler ve dosya yolları, bir `CRITICAL_SECTION` ile korunan global bir kuyruğa (`gs_kVct_crcPair`) eklenir.
    *   `ProcessScanner_PopProcessQueue` fonksiyonu, ana istemci thread'inin bu kuyruktan güvenli bir şekilde veri almasını sağlar.

#### Başlık Dosyası Tanımları (`ProcessScanner.h`)

*   **`typedef std::pair<DWORD, std::string> CRCPair;`**:
    *   Bir `DWORD` (genellikle CRC32 değeri) ve bir `std::string` (genellikle sürecin tam dosya yolu) içeren bir çifti tanımlar.
*   **Fonksiyon Prototipleri**:
    *   `void ProcessScanner_Destroy();`: Tarayıcı thread'ini durdurur, olayları ve kritik bölümü temizler.
    *   `bool ProcessScanner_Create();`: Kritik bölümü, olayları başlatır ve tarayıcı thread'ini oluşturup düşük öncelikte çalıştırır.
    *   `void ProcessScanner_ReleaseQuitEvent();`: Tarayıcı thread'ine çıkış yapması için sinyal gönderir.
    *   `bool ProcessScanner_PopProcessQueue(std::vector<CRCPair>* pkVct_crcPair);`: Taranan süreçlerin CRC ve yol bilgilerini içeren kuyruktan bir sonraki grubu alır. Eğer kuyruk boşsa `false` döner.

#### C++ Implementasyon Detayları (`ProcessScanner.cpp`)

*   **Global Statik Değişkenler**:
    *   `static std::vector<CRCPair> gs_kVct_crcPair;`: Taranan süreçlerin CRC ve yol çiftlerini tutan global kuyruk.
    *   `static CRITICAL_SECTION gs_csData;`: `gs_kVct_crcPair` kuyruğuna erişimi senkronize etmek için kullanılan kritik bölüm.
    *   `static HANDLE gs_evReqExit = NULL;`: Tarayıcı thread'ine çıkış isteğini bildirmek için kullanılan olay (event) nesnesi.
    *   `static HANDLE gs_evResExit = NULL;`: Tarayıcı thread'inin başarıyla çıktığını bildirmek için kullanılan olay nesnesi.
    *   `static HANDLE gs_hThread = NULL;`: Tarayıcı thread'inin tanıtıcısı (handle).

*   **`ScanProcessList(std::map<DWORD, DWORD>& rkMap_crcProc, std::vector<CRCPair>* pkVct_crcPair)` Fonksiyonu**:
    1.  `CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0)` ile sistemdeki tüm süreçlerin anlık görüntüsünü alır.
    2.  `Process32First` ve `Process32Next` ile her bir süreci döner.
    3.  Her süreç için `OpenProcess(PROCESS_VM_READ, ...)` ile süreç tanıtıcısını alır.
    4.  `CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, processID)` ile o sürece ait modüllerin anlık görüntüsünü alır.
    5.  `Module32First` ve `Module32Next` ile her bir modülü (genellikle .exe veya .dll dosyaları) döner.
    6.  Her modülün dosya yolu (`me32.szExePath`) için `GetCRC32` ile yolun CRC'sini alır (bu, daha önce taranan yolları hızlıca atlamak için kullanılabilir).
    7.  Eğer yol daha önce `rkMap_crcProc` içinde yoksa (yani ilk kez taranıyorsa):
        *   `GetFileCRC32(me32.szExePath)` ile modül dosyasının içeriğinin CRC'sini hesaplar.
        *   Hesaplanan dosya CRC'sini `rkMap_crcProc`'a (yol CRC'si anahtar olarak) ve `CRCPair` olarak `pkVct_crcPair` vektörüne ekler.
    8.  Kaynakları (`CloseHandle`) serbest bırakır.

*   **`ProcessScanner_Thread(void* pv)` Fonksiyonu (Arka Plan Thread'i)**:
    1.  Başlangıçta rastgele bir gecikme (10-20 saniye) ile başlar (`dwDelay`).
    2.  `while (WAIT_OBJECT_0 != WaitForSingleObject(gs_evReqExit, dwDelay))` döngüsü içinde çalışır:
        *   `gs_evReqExit` olayı sinyallenene kadar veya `dwDelay` süresi dolana kadar bekler.
        *   `kVct_crcPair.clear()` ile yerel CRC listesini temizler.
        *   `ScanProcessList(kMap_crcProc, &kVct_crcPair)` ile süreçleri tarar. `kMap_crcProc`, daha önce taranmış ve CRC'si alınmış süreçleri (yol CRC'si -> dosya CRC'si eşlemesi) tutarak aynı dosyanın tekrar tekrar CRC'sinin hesaplanmasını engeller.
        *   `EnterCriticalSection(&gs_csData)` ile kritik bölüme girer.
        *   Yeni taranan `kVct_crcPair`'deki verileri global `gs_kVct_crcPair` kuyruğuna ekler.
        *   `LeaveCriticalSection(&gs_csData)` ile kritik bölümden çıkar.
        *   Bir sonraki tarama için `dwDelay`'i daha kısa bir rastgele süreye (1-11 saniye) ayarlar.
    3.  Döngüden çıkıldığında (çıkış istendiğinde), `SetEvent(gs_evResExit)` ile ana thread'e çıkışın tamamlandığını bildirir.

*   **`ProcessScanner_Create()` Fonksiyonu**:
    1.  `InitializeCriticalSection(&gs_csData)` ile kritik bölümü başlatır.
    2.  `CreateEvent` ile `gs_evReqExit` ve `gs_evResExit` olaylarını oluşturur.
    3.  `_beginthread(ProcessScanner_Thread, ...)` ile `ProcessScanner_Thread` fonksiyonunu yeni bir thread olarak başlatır.
    4.  `SetThreadPriority(gs_hThread, THREAD_PRIORITY_LOWEST)` ile thread'in önceliğini en düşüğe ayarlar, böylece oyunun performansını minimum düzeyde etkiler.
    5.  Başarılı olursa `true` döner.

*   **`ProcessScanner_Destroy()` Fonksiyonu**:
    1.  `ProcessScanner_ReleaseQuitEvent()` (yani `SetEvent(gs_evReqExit)`) ile tarayıcı thread'ine çıkmasını söyler.
    2.  `WaitForSingleObject(gs_evResExit, INFINITE)` ile thread'in tamamen çıkmasını bekler.
    3.  `CloseHandle` ile olayları kapatır ve `DeleteCriticalSection` ile kritik bölümü siler.

*   **`ProcessScanner_PopProcessQueue(std::vector<CRCPair>* pkVct_crcPair)` Fonksiyonu**:
    1.  `EnterCriticalSection(&gs_csData)` ile kritik bölüme girer.
    2.  Global `gs_kVct_crcPair` kuyruğunun içeriğini kullanıcı tarafından sağlanan `pkVct_crcPair` vektörüne kopyalar.
    3.  `gs_kVct_crcPair.clear()` ile global kuyruğu temizler.
    4.  `LeaveCriticalSection(&gs_csData)` ile kritik bölümden çıkar.
    5.  Eğer `pkVct_crcPair` boşsa (kuyrukta veri yoksa) `false`, aksi halde `true` döner.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu süreç tarayıcı mekanizması, istemci tarafında basit bir anti-hile veya istenmeyen uygulama tespiti için kullanılır:

1.  **Başlatma**: Oyun istemcisi başladığında `ProcessScanner_Create()` çağrılarak arka plan tarayıcı thread'i başlatılır.
2.  **Periyodik Tarama**: Arka plan thread'i, düşük öncelikte çalışarak düzenli aralıklarla sistemdeki tüm süreçleri ve modüllerini tarar, CRC32 değerlerini hesaplar ve bir kuyruğa ekler.
3.  **Veri Çekme ve Kontrol**: Oyunun ana mantığı (muhtemelen `PythonNetworkStream` veya benzeri bir güvenlik modülü), periyodik olarak `ProcessScanner_PopProcessQueue()` fonksiyonunu çağırarak taranan süreçlerin CRC listesini alır.
4.  **Şüpheli Süreç Tespiti**: Ana istemci, elde ettiği CRC listesini, bilinen hile programlarının veya istenmeyen uygulamaların CRC değerlerini içeren bir "kara liste" (blacklist) ile karşılaştırır.
    *   Eğer bir eşleşme bulunursa, istemci bu durumu sunucuya bildirebilir, kullanıcıya bir uyarı gösterebilir veya oyunu sonlandırabilir.
    *   Ayrıca, sadece CRC eşleşmesi değil, süreç adlarına (`CRCPair.second`) göre de filtreleme yapılabilir (örneğin, "cheatengine.exe" gibi bilinen isimler).
5.  **Durdurma**: Oyun kapatılırken `ProcessScanner_Destroy()` çağrılarak tarayıcı thread'i ve ilgili kaynaklar düzgün bir şekilde temizlenir.

Bu sistem, temel düzeyde bir güvenlik katmanı sağlar. Ancak, CRC değerlerinin değiştirilebilmesi veya süreçlerin kendilerini gizleyebilmesi gibi yöntemlerle atlatılabilir. Daha gelişmiş anti-hile sistemleri genellikle daha karmaşık teknikler (bellek bütünlüğü kontrolleri, API hook tespiti, sürücü tabanlı korumalar vb.) kullanır.

### `PythonAcce.h` ve `PythonAcce.cpp` (Kuşak Bonus Emilim Sistemi - Acce)

Bu dosyalar, Metin2 istemcisindeki "Acce" veya daha yaygın bilinen adıyla "Kuşak Bonus Emilim" sisteminin istemci tarafı mantığını ve Python arayüzünü içerir. Bu sistem, oyuncuların bir eşyadan (genellikle bir silah veya zırh) belirli bir bonus yüzdesini alıp bir Kuşak (Sash) eşyasına aktarmasına olanak tanır, böylece kuşağın özelliklerini geliştirir. Bu özellik yalnızca `ENABLE_ACCE_COSTUME_SYSTEM` makrosu tanımlı olduğunda derlenir.

#### Dosyaların Genel Amaçları

*   **`PythonAcce.h`**: `CPythonAcce` singleton sınıfının tanımını içerir. Bu sınıf, emilim arayüzüne yerleştirilen eşyaların bilgilerini, işlemin maliyetini ve sunucudan gelen potansiyel sonuç bilgilerini (hangi eşyanın oluşacağı, minimum ve maksimum emilim yüzdesi) tutar.
*   **`PythonAcce.cpp`**: `CPythonAcce` sınıfının metotlarının implementasyonunu ve `acce` adında bir Python modülü oluşturan C-API fonksiyonlarını içerir. Bu Python modülü, UI scriptlerinin (genellikle Python ile yazılmış) Kuşak Emilim sistemiyle etkileşimde bulunmasını sağlar (eşya ekleme/çıkarma, işlem yapma isteği gönderme, bilgi alma vb.).

#### Başlık Dosyası Tanımları (`PythonAcce.h`)

*   **`#if defined(ENABLE_ACCE_COSTUME_SYSTEM)`**: Tüm içeriğin koşullu derlenmesini sağlar.
*   **`#include "Packet.h"`**: `TAcceMaterial` ve `TAcceResult` gibi paket yapılarının tanımlarını içerir.
*   **`class CPythonAcce : public CSingleton<CPythonAcce>`**:
    *   **`enum EAcce`**: Sistemle ilgili sabitleri tanımlar:
        *   `ABSORPTION_SOCKET = 0`: Emilimi yapacak olan ana eşyanın (genellikle Kuşak) yerleştirildiği yuva indeksi.
        *   `ABSORBED_SOCKET = 1`: Bonusları emilecek olan eşyanın yerleştirildiği yuva indeksi.
        *   `WINDOW_MAX_MATERIALS = 2`: Emilim penceresindeki maksimum malzeme yuvası sayısı (Kuşak ve emilecek eşya için).
        *   `LIMIT_RANGE = 1000`: Genel bir sınır veya aralık sabiti (bu bağlamda özel kullanımı belirsiz).
    *   **`DWORD dwPrice`**: Emilim işleminin toplam maliyetini (Yang) tutar.
    *   **`typedef std::vector<TAcceMaterial> TAcceMaterials;`**: `TAcceMaterial` (bkz: `Packet.h`, genellikle eşyanın envanterdeki yerini ve var olup olmadığını tutar) yapılarından oluşan bir vektör türü.
    *   **Genel (Public) Metotlar**:
        *   `CPythonAcce()`: Kurucu.
        *   `virtual ~CPythonAcce()`: Yıkıcı.
        *   `void Clear()`: Emilim penceresindeki tüm verileri temizler.
        *   `void AddMaterial(DWORD dwRefPrice, BYTE bPos, TItemPos tPos)`: Belirtilen yuvaya (`bPos`) envanterdeki (`tPos`) bir eşyayı ekler ve referans fiyatı (`dwRefPrice`, sunucudan gelir) günceller.
        *   `void AddResult(DWORD dwItemVnum, DWORD dwMinAbs, DWORD dwMaxAbs)`: Sunucudan gelen, işlem sonucunda oluşacak eşyanın VNUM'unu ve emilebilecek minimum/maksimum bonus oranını ayarlar.
        *   `void RemoveMaterial(DWORD dwRefPrice, BYTE bPos)`: Belirtilen yuvadan eşyayı kaldırır ve referans fiyatı günceller.
        *   `DWORD GetPrice()`: Mevcut işlem maliyetini döndürür.
        *   `bool GetAttachedItem(BYTE bPos, BYTE& bHere, WORD& wCell)`: Belirtilen yuvadaki eşyanın varlığını (`bHere`) ve envanterdeki hücre numarasını (`wCell`) döndürür.
        *   `void GetResultItem(DWORD& dwItemVnum, DWORD& dwMinAbs, DWORD& dwMaxAbs)`: Sonuç eşyasının VNUM'unu ve emilim oranlarını döndürür.
        *   `BOOL IsCleanScroll(DWORD dwItemVnum)`: Verilen eşya VNUM'unun "Temizleme Parşömeni" olup olmadığını kontrol eder (özel VNUM'lar: 39046, 90000).
    *   **Korumalı (Protected) Üyeler**:
        *   `TAcceResult m_vAcceResult`: Sunucudan gelen sonuç eşya bilgisini tutan yapı (`Packet.h` içinde tanımlı).
        *   `TAcceMaterials m_vAcceMaterials`: Emilim penceresine yerleştirilen eşyaları (malzemeleri) tutan vektör.

#### C++ ve Python Arayüzü Implementasyon Detayları (`PythonAcce.cpp`)

*   **`CPythonAcce` Sınıfı Implementasyonu**:
    *   `Clear()`: `dwPrice`'ı sıfırlar, `m_vAcceResult` yapısını sıfırlar ve `m_vAcceMaterials` vektörünü temizleyip yeniden boyutlandırır.
    *   `AddMaterial()`: Eğer eklenen eşya ilk yuvaya (Kuşak yuvası, `bPos == 0`) ise `dwPrice`'ı günceller. İlgili yuvadaki `TAcceMaterial`'ın `bHere` (varlık durumu) ve `wCell` (envanter hücresi) bilgilerini ayarlar.
    *   `AddResult()`: `m_vAcceResult` içindeki `dwItemVnum`, `dwMinAbs`, `dwMaxAbs` alanlarını günceller.
    *   `RemoveMaterial()`: Eğer çıkarılan eşya ikinci yuvaya (emilecek eşya yuvası, `bPos == 1`) ise `dwPrice`'ı günceller. İlgili yuvadaki `TAcceMaterial`'ı sıfırlar.
    *   `GetAttachedItem()`, `GetResultItem()`, `IsCleanScroll()`: İlgili bilgileri döndüren basit erişimci metotlardır.
*   **Python Modülü Fonksiyonları (`acce*` önekli global fonksiyonlar)**:
    *   Bu fonksiyonlar, Python'dan çağrılabilen C fonksiyonlarıdır ve `CPythonAcce` singleton örneği veya `CPythonNetworkStream` aracılığıyla işlemler gerçekleştirir.
    *   `acceSendCloseRequest(PyObject* poSelf, PyObject* poArgs)`: Sunucuya Kuşak Emilim penceresini kapatma isteği gönderir (`CPythonNetworkStream::SendAcceClosePacket()`).
    *   `acceAdd(PyObject* poSelf, PyObject* poArgs)`: Python'dan gelen eşya konumu (envanter türü, hücre) ve hedef yuva bilgisini alarak sunucuya eşya ekleme isteği gönderir (`CPythonNetworkStream::SendAcceAddPacket()`).
    *   `acceRemove(PyObject* poSelf, PyObject* poArgs)`: Python'dan gelen yuva bilgisini alarak sunucuya eşya çıkarma isteği gönderir (`CPythonNetworkStream::SendAcceRemovePacket()`).
    *   `acceGetPrice(PyObject* poSelf, PyObject* poArgs)`: `CPythonAcce::Instance().GetPrice()`'ı çağırarak işlem maliyetini Python'a döndürür.
    *   `acceGetAttachedItem(PyObject* poSelf, PyObject* poArgs)`: `CPythonAcce::Instance().GetAttachedItem()`'ı çağırarak belirtilen yuvadaki eşya bilgisini Python'a döndürür.
    *   `acceGetResultItem(PyObject* poSelf, PyObject* poArgs)`: `CPythonAcce::Instance().GetResultItem()`'ı çağırarak sonuç eşya bilgisini Python'a döndürür.
    *   `acceSendRefineRequest(PyObject* poSelf, PyObject* poArgs)`: İkinci yuvada (`ABSORBED_SOCKET`) bir eşya olup olmadığını kontrol eder ve varsa sunucuya emilim işlemini başlatma isteği gönderir (`CPythonNetworkStream::SendAcceRefinePacket()`).
    *   `acceIsCleanScroll(PyObject* poSelf, PyObject* poArgs)`: Python'dan gelen eşya VNUM'unu alarak `CPythonAcce::Instance().IsCleanScroll()`'u çağırır ve sonucu Python'a döndürür.
*   **`initAcce()` Fonksiyonu**:
    *   Python C-API kullanılarak `acce` adında yeni bir Python modülü oluşturulur.
    *   Yukarıda listelenen `acce*` fonksiyonları bu modüle metot olarak kaydedilir.
    *   `CPythonAcce::EAcce` enum'undaki sabitler (`ABSORPTION_SOCKET`, `ABSORBED_SOCKET`, `WINDOW_MAX_MATERIALS`, `LIMIT_RANGE`) modüle tamsayı sabitleri olarak eklenir. Bu, Python scriptlerinin bu sabitlere isimleriyle erişebilmesini sağlar.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu sistem, Kuşak (Sash) eşyalarına bonus aktarma arayüzünün ve mantığının temelini oluşturur:

1.  **Arayüz Etkileşimi**: Oyuncu Kuşak Emilim penceresini açtığında, Python ile yazılmış UI scriptleri devreye girer.
2.  **Eşya Yerleştirme**: Oyuncu, envanterinden bir Kuşak ve bonuslarını emmek istediği başka bir eşyayı (örn. silah, zırh) sürükleyip arayüzdeki ilgili yuvalara bıraktığında, UI scriptleri `acce.Add()` fonksiyonunu çağırır. Bu fonksiyon da `CPythonNetworkStream` üzerinden sunucuya bilgi gönderir.
3.  **Sunucu Yanıtı**: Sunucu, yerleştirilen eşyaları doğrular, işlem maliyetini (`dwPrice`) ve potansiyel sonucu (`TAcceResult` - yeni kuşağın VNUM'u ve emilebilecek min/max bonus yüzdesi) istemciye geri gönderir. `CPythonNetworkStream` bu paketleri aldığında `CPythonAcce`'nin ilgili metotlarını (`AddMaterial`, `AddResult`) çağırarak istemci tarafındaki veriyi günceller.
4.  **Bilgi Gösterimi**: UI scriptleri, `acce.GetAttachedItem()`, `acce.GetResultItem()`, `acce.GetPrice()` gibi fonksiyonları çağırarak `CPythonAcce`'den güncel bilgileri alır ve arayüzde oyuncuya gösterir (yerleştirilen eşyaların ikonları, işlem maliyeti, beklenen bonus aralığı vb.).
5.  **"Temizleme Parşömeni" Kontrolü**: Eğer oyuncu özel bir "Temizleme Parşömeni" kullanıyorsa, `acce.IsCleanScroll()` ile bu parşömenin geçerliliği kontrol edilebilir. Bu tür parşömenler genellikle emilim oranını artırmak veya eşyanın kaybolmasını önlemek gibi etkilere sahip olabilir.
6.  **Emilim İşlemi**: Oyuncu "Tamam" veya "Birleştir" gibi bir butona tıkladığında, UI scriptleri `acce.SendRefineRequest()` fonksiyonunu çağırır. Bu fonksiyon, sunucuya emilim işlemini gerçekleştirme isteği gönderir.
7.  **Kapatma**: Pencere kapatıldığında `acce.SendCloseRequest()` çağrılır.

`CPythonAcce` ve `acce` Python modülü, bu karmaşık sistemin C++ çekirdek mantığı ile Python tabanlı kullanıcı arayüzü arasında bir köprü görevi görerek veri akışını ve kullanıcı etkileşimlerini yönetir.

Bu sistem, temel düzeyde bir güvenlik katmanı sağlar. Ancak, CRC değerlerinin değiştirilebilmesi veya süreçlerin kendilerini gizleyebilmesi gibi yöntemlerle atlatılabilir. Daha gelişmiş anti-hile sistemleri genellikle daha karmaşık teknikler (bellek bütünlüğü kontrolleri, API hook tespiti, sürücü tabanlı korumalar vb.) kullanır. 

### `PythonApplication.h` ve `PythonApplication.cpp` (Ana İstemci Uygulaması)

Bu iki dosya, Metin2 istemcisinin kalbi sayılabilecek `CPythonApplication` sınıfını tanımlar ve implemente eder. Bu sınıf, oyunun genel işleyişinden, kullanıcı arayüzü yönetiminden, grafik ve ses motorlarıyla etkileşimden, ağ iletişiminden ve diğer tüm alt sistemlerin koordinasyonundan sorumludur.

#### Dosyaların Genel Amaçları

*   **`PythonApplication.h`**:
    *   `CPythonApplication` sınıfının tanımını içerir. Bu sınıf, `CMSApplication` (temel pencere ve mesaj döngüsü), `CInputKeyboard` (klavye girişi) ve `IAbstractApplication` (soyut uygulama arayüzü) sınıflarından miras alır.
    *   İstemcinin çeşitli durumları, ayarları ve modları için `enum` tanımları (örn: `EDeviceState`, `ECursorMode`, `ECursorShape`, `ECameraControlDirection`, `ECameraMode`) içerir.
    *   Oyunun ana bileşenleri olan birçok Python sarmalayıcı sınıfının (örn: `CPythonPlayer`, `CPythonItem`, `CPythonNetworkStream`, `CPythonBackground`, `CPythonSystem`, `CPythonChat` vb.) ve yönetim sınıflarının (`CEffectManager`, `CItemManager`, `CRaceManager`, `CSoundManager` vb.) üye değişkenlerini deklare eder.
    *   Genel uygulama kontrolü, başlatma, sonlandırma, ana döngü, render ve güncelleme fonksiyonları, kamera yönetimi, fare ve klavye olay işleyicileri, IME (Input Method Editor) yönetimi, zaman ve FPS yönetimi gibi birçok public ve protected metot prototipini barındırır.
    *   CEF (Chromium Embedded Framework) ile web tarayıcı entegrasyonu için fonksiyon prototipleri (`ShowWebPage`, `MoveWebPage`, `HideWebPage`) içerir.
    *   Oyun içi video oynatımı (`OnLogoOpen`, `OnLogoUpdate` vb.) için DirectShow tabanlı üye değişkenler ve metotlar içerir.

*   **`PythonApplication.cpp`**:
    *   `CPythonApplication` sınıfının tüm metotlarının C++ implementasyonlarını içerir.
    *   **Kurucu (`CPythonApplication::CPythonApplication`)**: Üye değişkenleri varsayılan değerlere ayarlar, hata yakalayıcıyı (exception handler) kurar, zamanlayıcıyı başlatır ve singleton örneğini (`ms_pInstance`) ayarlar.
    *   **Yıkıcı (`CPythonApplication::~CPythonApplication`)**: Varsayılan yıkıcı kullanılır. Gerçek temizleme işlemleri `Destroy()` metodunda yapılır.
    *   **`Create()`**: Ana uygulama penceresini oluşturur, ikonları ayarlar, IME'yi başlatır, tam ekran/pencere modunu ayarlar, fare imleçlerini oluşturur, ses yöneticisini ve grafik cihazını başlatır, klavye girişini başlatır, ağ cihazını kurar ve diğer Python modüllerini (`CPythonItem`, `CPythonIME`, `CPythonTextTail` vb.) ve yöneticileri (`CLightManager`) başlatır. Ayrıca yapışkan tuşlar (StickyKeys) gibi Windows ayarlarını geçici olarak düzenler ve küre haritalarını (sphere maps) yükler.
    *   **`Destroy()`**: `Create()` ile başlatılan tüm kaynakları serbest bırakır: CEF tarayıcıyı, küre haritalarını, pencere yöneticisini, çeşitli Python modüllerini ve C++ yöneticilerini, grafik ve ses cihazlarını, fare imleçlerini yok eder. Kaydedilmiş ayarları (config, interface status) dosyaya yazar ve Windows ayarlarını geri yükler.
    *   **`Loop()`**: Oyunun ana mesaj döngüsünü yönetir. Sürekli olarak Windows mesajlarını işler (`MessageProcess()`) ve ardından ana oyun mantığını (`Process()`) çalıştırır.
    *   **`Process()`**: Her frame'de çağrılan ana mantık fonksiyonudur. Zamanlayıcıyı günceller, ağ paketlerini işler, klavye ve fare girdilerini günceller, kamerayı günceller, kaynakları, kullanıcı arayüzünü ve oyun dünyasını günceller. Frame atlama (frame skipping) mantığını uygular. Grafik motorunun `Begin()`, `End()`, `Show()` çağrılarını yaparak render işlemini tetikler ve FPS, yük gibi performans sayaçlarını günceller.
    *   **`UpdateGame()`**: Oyun dünyasının mantıksal güncellemelerini yapar. Oyuncu pozisyonunu alır, arka planı günceller, oyun olay yöneticisini, karakter yöneticisini, efekt yöneticisini, uçan nesne yöneticisini, eşya yöneticisini ve oyuncu nesnesini günceller.
    *   **`RenderGame()`**: Oyun dünyasının görselleştirilmesini gerçekleştirir. Render hedeflerini (render targets) yönetir, perspektif ve görüş alanını ayarlar, culling (görünmeyen nesnelerin çizilmemesi) işlemini yapar, karakterleri, özel dükkanları, gölgeleri, gökyüzünü, bulutları, çevreyi, suyu, karı, efektleri, eşyaları, uçan nesneleri ve PC engelleyicilerini çizer.
    *   **Giriş İşleyicileri (`OnMouseMove`, `OnLButtonDown` vb.)**: Fare ve klavye olaylarını yakalar ve bunları ilgili Python fare işleyicisine veya oyun içi eylemlere yönlendirir.
    *   **Kamera Yönetimi (`__UpdateCamera`, `RotateCamera`, `PitchCamera`, `ZoomCamera` vb.)**: Kamera pozisyonunu, açısını ve hızını yönetir. Normal, özel ve olay kameraları arasında geçiş yapabilir.
    *   **Pencere Yönetimi (`__SetFullScreenWindow`, `OnSizeChange` vb.)**: Tam ekran ve pencere modu arasında geçişi, pencere boyutlandırmasını ve aktif/minimized durumlarını yönetir.
    *   **`LoadLocaleData()`**: Belirtilen yerelleştirme yolundan eşya, NPC, yetenek tanımlarını, hakaret listesini ve diğer yerel ayarları yükler.
    *   Diğer birçok yardımcı fonksiyon (zaman yönetimi, FPS sorgulama, sunucu bağlantı bilgileri, IME kontrolü vb.) içerir.

#### Başlık Dosyası Tanımları (`PythonApplication.h`)

*   **Temel Sınıflar ve Arayüzler**:
    *   `CMSApplication`: Temel Windows uygulama işlevleri (pencere oluşturma, mesaj döngüsü).
    *   `CInputKeyboard`: Klavye girişi işleme.
    *   `IAbstractApplication`: Uygulamanın soyut arayüzü.
*   **Önemli Enum Tanımları**:
    *   `EDeviceState`: Grafik cihazının durumu (OK, SKIP, FALSE).
    *   `ECursorMode`: Fare imleci modu (HARDWARE, SOFTWARE).
    *   `ECursorShape`: Farklı oyun içi durumlar için fare imleci şekilleri (NORMAL, ATTACK, TARGET, TALK vb.).
    *   `EInfo`: Hata ayıklama bilgisi türleri (ACTOR, EFFECT, ITEM, TEXTTAIL).
    *   `ECameraControlDirection`: Kamera hareket yönleri (POSITIVE, NEGITIVE, STOP).
    *   `ECameraMode`: Kamera modları (NORMAL, STAND, BLEND).
*   **Üye Değişkenler (Yöneticiler ve Python Sarmalayıcıları)**:
    *   Grafik ve Ağ: `CGraphicDevice m_grpDevice`, `CNetworkDevice m_netDevice`, `CPythonGraphic m_pyGraphic`, `CPythonNetworkStream m_pyNetworkStream`.
    *   Oyun Sistemleri: `CRaceManager m_RaceManager`, `CItemManager m_kItemMgr`, `CEffectManager m_kEftMgr`, `CSoundManager m_SoundManager`, `CLightManager m_LightManager`, `CPythonBackground m_pyBackground`, `CPythonPlayer m_pyPlayer`, `CPythonCharacterManager m_kChrMgr`, `CPythonItem m_pyItem`, `CPythonShop m_pyShop`, `CPythonSkill m_pySkill`, `CPythonQuest m_pyQuest`, `CPythonChat m_pyChat`, `CPythonSystem m_pySystem` vb.
    *   Kullanıcı Arayüzü: `UI::CWindowManager m_kWndMgr`, `CPythonIME m_pyIme`, `CPythonTextTail m_pyTextTail`, `CPythonMiniMap m_pyMiniMap`.
    *   Diğer: `CAccountConnector m_kAccountConnector`, `CGuildMarkManager m_kGuildMarkManager`, `CMovieMan m_MovieManager`.
*   **Temel Fonksiyon Prototipleri**:
    *   `Create()`: Uygulamayı ve alt sistemleri başlatır.
    *   `Destroy()`: Tüm kaynakları serbest bırakır.
    *   `Loop()`: Ana mesaj ve oyun döngüsü.
    *   `Process()`: Ana mantık ve render tetikleme.
    *   `UpdateGame()`: Oyun dünyası güncellemeleri.
    *   `RenderGame()`: Oyun dünyası çizimi.
    *   `SetMouseHandler()`: Python tarafında fare olaylarını işleyecek nesneyi ayarlar.
    *   Kamera kontrol (`SetCamera`, `RotateCamera` vb.), zaman (`SetServerTime`, `GetGlobalTime` vb.), FPS (`SetFPS`, `GetUpdateFPS` vb.) ve pencere yönetimi fonksiyonları.

#### C++ Implementasyon Detayları (`PythonApplication.cpp`)

*   **Singleton Yapısı**: `CPythonApplication::ms_pInstance` statik üyesi ile singleton deseni uygulanır, böylece uygulamanın tek bir örneğine global olarak erişilebilir (`CPythonApplication::Instance()`).
*   **Ana Döngü (`Loop` ve `Process`)**:
    *   `Loop` fonksiyonu, Windows mesajlarını alır ve işler. Eğer mesaj yoksa `Process` fonksiyonunu çağırır.
    *   `Process` fonksiyonu, her frame'de oyunun tüm mantığını yürütür:
        1.  Zamanlayıcıyı günceller (`CTimer::Instance().Advance()`).
        2.  Ağ olaylarını işler (`m_pyNetworkStream.Process()`, lonca sembol yükleyici/indirici, hesap bağlayıcı).
        3.  Klavye ve fare girdilerini işler (`UpdateKeyboard()`, `OnMouseMove()`).
        4.  Kamerayı günceller (`__UpdateCamera()`).
        5.  Kaynak yöneticisini, kamera olaylarını, fare olaylarını ve UI olaylarını günceller (`CResourceManager::Instance().Update()`, `OnCameraUpdate()`, `OnMouseUpdate()`, `OnUIUpdate()`).
        6.  Frame atlama (frame skipping) mantığını uygular: Eğer oyun, hedeflenen FPS'in gerisinde kalıyorsa, render işlemini atlayabilir.
        7.  Render işlemi: Eğer frame atlanmıyorsa ve cihaz hazırsa, grafik arayüzünü (`OnUIRender()`, `OnMouseRender()`) ve ardından oyun dünyasını (`RenderGame()`, ancak bu `OnUIRender` içinde çağrılır gibi görünüyor) çizer.
        8.  Çizim sonrası (`m_pyGraphic.Show()`) bir sonraki frame için bekleme süresini ayarlar (`Sleep()`).
*   **Oyun ve Arayüz Güncelleme/Render Ayrımı**:
    *   `UpdateGame()`: Oyun mantığını (karakterler, NPC'ler, arka plan, efektler, eşyalar vb.) günceller.
    *   `RenderGame()`: 3D oyun dünyasını (karakterler, arazi, su, gökyüzü, efektler vb.) çizer.
    *   `OnUIUpdate()`: Arayüz elemanlarının (pencereler, scriptler) mantığını günceller (genellikle `m_kWndMgr.Update()` çağrısı).
    *   `OnUIRender()`: Arayüz elemanlarını çizer (`m_kWndMgr.Render()`). `RenderGame()` fonksiyonu genellikle `OnUIRender()` içinden, arayüzün arkasına çizilecek şekilde çağrılır.
*   **Kaynak Yönetimi**:
    *   `Create()` ve `Destroy()` fonksiyonları, oyunun ihtiyaç duyduğu tüm kaynakların (grafik, ses, ağ, Python modülleri, yöneticiler) yaşam döngüsünü yönetir.
    *   `LoadLocaleData()`: Oyuna özgü yerel verileri (eşya, NPC prototipleri, yetenek açıklamaları vb.) yükler. Bu, genellikle oyun başladığında veya bölge değiştirildiğinde çağrılır.
*   **Hata Yönetimi ve Performans**:
    *   `SetEterExceptionHandler()`: Özel bir hata yakalama mekanizması kurar.
    *   `NotifyHack()`: Şüpheli bir durum algılandığında sunucuya bildirim gönderir.
    *   Kod içinde `#if defined(__PERFORMANCE_CHECK__)` blokları ve `PERF_CHECKER_RENDER_GAME` gibi bayraklar, performans ölçümleri için zamanlayıcılar ve loglamalar ekler.
    *   Frame atlama ve dinamik görüş mesafesi ayarı (`m_pyBackground.SetViewDistanceSet`) gibi mekanizmalar, düşük performanslı sistemlerde oynanabilirliği artırmaya yöneliktir.
*   **CEF Entegrasyonu**: `CefWebBrowser_Initialize`, `CefWebBrowser_Update`, `CefWebBrowser_Destroy` gibi fonksiyonlarla (bu dosyada direkt çağrılmasa da `CefWebBrowser.h` dahil edilmiş) Chromium Embedded Framework üzerinden oyun içi web tarayıcı işlevselliği sağlanır.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonApplication`, Metin2 istemcisinin çalışmasını sağlayan merkezi orkestrasyon sınıfıdır.

1.  **Başlatma**: Oyun çalıştırıldığında, `main()` (veya eşdeğeri bir giriş noktası) `CPythonApplication::Instance().Create()` çağırarak uygulamayı başlatır. Bu, pencereyi oluşturur, grafikleri, sesi, ağı ve tüm oyun alt sistemlerini initialize eder.
2.  **Ana Döngü**: `Create()` başarılı olursa, `CPythonApplication::Instance().Loop()` çağrılır. Bu döngü, kullanıcı oyundan çıkana kadar devam eder.
3.  **Olay İşleme**: Döngü içinde, Windows mesajları (fare tıklamaları, klavye girişleri, pencere olayları) işlenir ve `OnMouseMove`, `OnKeyDown` gibi ilgili olay işleyici metotlara yönlendirilir. Bu metotlar da genellikle Python scriptlerine veya oyun içi mantığa bu girdileri iletir.
4.  **Güncelleme ve Render**: Her frame'de `Process()` fonksiyonu çağrılarak:
    *   Ağdan gelen veriler işlenir.
    *   Oyun dünyasının ve arayüzün mantığı güncellenir (`UpdateGame()`, `OnUIUpdate()`).
    *   Kamera güncellenir.
    *   Ardından oyun dünyası ve arayüz çizilir (`RenderGame()`, `OnUIRender()`).
5.  **Alt Sistemlerle Etkileşim**: `CPythonApplication` içindeki çeşitli yönetici (örn: `m_kChrMgr`, `m_kEftMgr`) ve Python sarmalayıcı (`m_pyPlayer`, `m_pyItem`) nesneleri aracılığıyla oyunun tüm özellikleriyle etkileşimde bulunulur. Örneğin, bir ağ paketi geldiğinde `m_pyNetworkStream` bunu işler ve ilgili Python fonksiyonunu veya `CPythonApplication` metodunu çağırarak oyun durumunu günceller.
6.  **Python Entegrasyonu**: Birçok `CPython*` sınıfı, C++ mantığını Python scriptlerinin kullanabileceği şekilde sarmalar. `CPythonApplication`, bu Python modüllerinin çoğunu barındırır ve yönetir. Örneğin, `SetMouseHandler(PyObject* poMouseHandler)` ile Python tarafında tanımlanmış bir fare işleyici nesnesi kaydedilebilir.
7.  **Sonlandırma**: Oyuncu oyundan çıktığında veya bir hata oluştuğunda, `Destroy()` çağrılarak tüm kaynaklar düzgün bir şekilde serbest bırakılır ve uygulama sonlanır.

Kısacası, `CPythonApplication` olmadan istemcinin çalışması mümkün değildir. Oyunun tüm temel işlevlerini bir araya getirir ve yönetir.

### `PythonApplicationCamera.cpp` (Kamera Yönetimi Uygulaması)

Bu dosya, `CPythonApplication` sınıfının kamera yönetimiyle ilgili fonksiyonlarının detaylı implementasyonunu içerir. `PythonApplication.h` başlık dosyasında tanımlanan kamera ile ilgili metotlar burada hayata geçirilir.

#### Dosyanın Genel Amaçları

*   Oyun içi kameranın güncellenmesi (`__UpdateCamera`).
*   Farklı kamera modları (`CAMERA_MODE_NORMAL`, `CAMERA_MODE_STAND`, `CAMERA_MODE_BLEND`) arasında geçiş yapılması ve bu modlara özel davranışların yönetilmesi.
*   Oyuncunun kamera üzerindeki doğrudan kontrolünü sağlayan fonksiyonlar:
    *   Kamerayı döndürme (`RotateCamera`, `MovieRotateCamera`).
    *   Kameranın eğimini ayarlama (`PitchCamera`, `MoviePitchCamera`).
    *   Kamerayı yakınlaştırma/uzaklaştırma (`ZoomCamera`, `MovieZoomCamera`).
    *   Kamera hareket hızlarını ayarlama (`SetViewDirCameraSpeed`, `SetCrossDirCameraSpeed`, `SetUpDirCameraSpeed`).
*   Özel "event" (olay) kameralarının tanımlanması (`SetEventCamera`), bu kameralar arasında yumuşak geçişlerin (`BlendEventCamera`) yapılması ve varsayılan kameraya dönülmesi (`SetDefaultCamera`).
*   Mevcut kamera ayarlarının (`SCameraSetting` yapısı) alınması (`GetCameraSetting`), ayarlanması (`SetCameraSetting`) ve bir dosyaya kaydedilip (`SaveCameraSetting`) dosyadan yüklenmesi (`LoadCameraSetting`).

#### C++ Implementasyon Detayları

*   **`__UpdateCamera()`**: Bu merkezi fonksiyon, her frame'de çağrılarak aktif kamera moduna göre kamera pozisyonunu, açısını ve diğer parametrelerini günceller. `BlendValueByLinear` fonksiyonu, `CAMERA_MODE_BLEND` modunda iki kamera ayarı arasında yumuşak geçişler için doğrusal enterpolasyon yapar. Ayrıca, kamera hareket hızlarını uygular ve ses dinleyici pozisyonunu günceller.
*   **Kamera Kontrolü**: `RotateCamera`, `PitchCamera`, `ZoomCamera` gibi fonksiyonlar, oyuncu girdilerine (genellikle fare veya klavye) yanıt olarak çağrılır ve kameranın dönüş, eğim ve zoom hızlarını anlık olarak ayarlar. `Movie*` versiyonları, muhtemelen oyun içi sinematikler için ek kontroller (örn. `VK_SCROLL` tuşu ile farklı davranış) sunar.
*   **Olay Kameraları**:
    *   `SCameraSetting` yapısı, bir kameranın tüm relevant parametrelerini (pozisyon, hedef, zoom, pitch, roll vb.) bir arada tutar.
    *   `SetEventCamera` belirli bir `SCameraSetting`'i aktif hale getirirken, `BlendEventCamera` mevcut durumdan yeni bir `SCameraSetting`'e belirli bir sürede geçiş yapar.
    *   `SetDefaultCamera` ise oyuncunun standart oyun kamerasına geri döner.
*   **Ayarları Kaydetme/Yükleme**:
    *   `SaveCameraSetting`, mevcut kamera ayarlarını bir metin dosyasına yazar. Bu, geliştiricilerin veya oyuncuların özel kamera açılarını kaydetmesine olanak tanır.
        *   **Not**: `SaveCameraSetting` içindeki `SetFileAttributes(c_szFileName, FILE_ATTRIBUTE_NORMAL);` satırı, `c_szFileName` (const char*) argümanını `SetFileAttributes`'in beklediği `LPCWSTR` (const WCHAR*) türüne dönüştürmediği için bir derleyici uyarısı/hatası verebilir. Doğru kullanım için `MultiByteToWideChar` API'si ile dönüştürme yapılmalıdır.
    *   `LoadCameraSetting`, daha önce kaydedilmiş bir kamera ayar dosyasını okuyarak kamerayı o ayarlara getirir. `CTextFileLoader` bu işlem için kullanılır.
*   **Yardımcı Fonksiyonlar**: `GetCamera` mevcut kamera parametrelerini döndürürken, `IsLockCurrentCamera` kameranın oyuncu tarafından kontrol edilip edilemeyeceğini belirtir. `GetRotation` ve `GetPitch` mevcut kameranın dönüş ve eğimini verir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyadaki fonksiyonlar, oyunun temel kamera sistemini oluşturur:
*   Oyuncunun karakterini takip eden standart üçüncü şahıs kamera.
*   Belirli oyun içi olaylar, diyaloglar veya sinematik anlar için kullanılan özel, önceden tanımlanmış veya dinamik olarak ayarlanan kameralar.
*   Kamera ayarlarını harici dosyalarda saklayarak, farklı sahneler veya haritalar için özel kamera konfigürasyonlarının kolayca yönetilmesi.
*   Fare orta tuşu ile kamera sürükleme ve tekerlek ile zoom yapma gibi yaygın kamera kontrol mekanizmalarının implementasyonu.

### `PythonApplicationCursor.cpp` (Fare İmleci Yönetimi)

Bu dosya, `CPythonApplication` sınıfının Windows fare imleçlerini yönetmeyle ilgili fonksiyonlarının implementasyonunu içerir. Oyunun farklı durumlarına göre uygun fare imlecinin gösterilmesini sağlar.

#### Dosyanın Genel Amaçları

*   Oyun içinde kullanılacak standart Windows fare imleçlerini (`.cur` dosyaları) kaynaklardan (`resource.h` aracılığıyla) yüklemek (`CreateCursors`).
*   Yüklenen imleçleri oyun sonlandığında serbest bırakmak (`DestroyCursors`).
*   Fare imlecinin görünürlüğünü kontrol etmek (`SetCursorVisible`, `GetCursorVisible`).
*   Fare imlecinin modunu (donanım destekli Windows imleci veya yazılım ile çizilen özel imleç) ayarlamak (`SetCursorMode`, `GetCursorMode`).
*   Oyunun o anki durumuna göre (örneğin, normal, saldırı, konuşma, bir nesne üzerine gelme) imlecin şeklini değiştirmek (`SetCursorNum`).

#### C++ Implementasyon Detayları

*   **`CreateCursors()`**:
    *   `PythonApplication.h` içinde tanımlı `CURSOR_COUNT` enumundaki her bir imleç türü için `resource.h` dosyasında tanımlanmış `IDC_CURSOR_*` ID'lerini kullanarak `LoadImage` API'si ile imleçleri yükler.
    *   Yüklenen `HCURSOR` (imleç handle) değerlerini `m_CursorHandleMap` adlı bir `std::map` içinde saklar.
    *   İmlecin başlangıçta görünür (`m_bCursorVisible = TRUE`) olmasını ayarlar.
*   **`DestroyCursors()`**: `m_CursorHandleMap` içindeki tüm imleçleri `DestroyCursor` API'si ile serbest bırakır.
*   **Görünürlük ve Mod**:
    *   `SetCursorVisible()`: İmlecin görünürlüğünü `ShowCursor` API'si ile ayarlar (eğer `CURSOR_MODE_HARDWARE` aktifse). `m_bLiarCursorOn` bayrağı, muhtemelen imlecin gizlenip yerine sahte bir imlecin (yazılımsal) gösterildiği durumlar için kullanılır.
    *   `SetCursorMode()`: `CURSOR_MODE_HARDWARE` (Windows'un imleci çizdiği mod) ve `CURSOR_MODE_SOFTWARE` (oyunun imleci kendisinin çizdiği mod, Windows imleci gizlenir) arasında geçiş yapar.
*   **İmleç Şekli**:
    *   `SetCursorNum(int iCursorNum)`: Aktif imleç şeklini değiştirir.
        *   `__IsContinuousChangeTypeCursor` fonksiyonu, bazı imleçlerin (örn. saldırı, konuşma) bir etkileşimden sonra hemen normale dönmeyip kalıcı olması gerekip gerekmediğini belirler. Eğer normale dönülüyorsa ve önceki imleç kalıcı bir tipse, ona geri döner (`m_iContinuousCursorNum`).
        *   `CURSOR_MODE_HARDWARE` ise, `m_CursorHandleMap`'ten ilgili `HCURSOR`'u alıp `SetCursor` API'si ile Windows'un aktif imlecini değiştirir.
        *   Python tarafındaki fare işleyicisine (`m_poMouseHandler->ChangeCursor`) imlecin değiştiğini bildirir.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Etkileşim Geri Bildirimi**: Oyuncuya, fare imlecinin hangi tür etkileşim için hazır olduğunu gösterir:
    *   `IDC_CURSOR_NORMAL`: Genel kullanım.
    *   `IDC_CURSOR_ATTACK`: Bir düşmana veya saldırılabilir nesneye işaret ederken.
    *   `IDC_CURSOR_TALK`: Bir NPC ile konuşma başlatmak için.
    *   `IDC_CURSOR_PICK`: Yerden bir eşya almak için.
    *   `IDC_CURSOR_CAMERA_ROTATE`: Kamera döndürme modunda (genellikle fare orta tuşu basılıyken).
*   **Sezgisel Arayüz**: Farklı imleçler, oyuncunun ne yapabileceğini anlamasına yardımcı olarak kullanıcı deneyimini iyileştirir.

### `PythonApplicationEvent.cpp` (Uygulama Olay Yönetimi)

Bu dosya, `CPythonApplication` sınıfının çeşitli uygulama ve kullanıcı girdi olaylarını işlemesine yönelik fonksiyonların implementasyonunu içerir. Bu fonksiyonlar, Windows mesaj döngüsünden veya oyunun ana döngüsünden gelen olayları yakalar ve ilgili alt sistemlere (Kamera, UI, Python scriptleri) yönlendirir.

#### Dosyanın Genel Amaçları

*   Kamera ve Kullanıcı Arayüzü (UI) için güncelleme (`OnCameraUpdate`, `OnUIUpdate`) ve render (`OnUIRender`) kancaları sağlamak.
*   Fare olaylarını (hareket, tuş basımları, tekerlek) yakalamak ve işlemek (`OnMouseMove`, `OnMouseMiddleButtonDown/Up`, `OnMouseWheel`, `OnMouseLeftButtonDown/Up/DoubleClick`, `OnMouseRightButtonDown/Up`).
*   Klavye olaylarını (tuşa basma, bırakma) yakalamak ve işlemek (`OnKeyDown`, `OnKeyUp`).
*   IME (Input Method Editor) olaylarını işleyerek karmaşık metin girişlerini (örn. Asya dilleri) desteklemek.
*   Python tarafında tanımlanmış fare işleyicisi için güncelleme ve render fonksiyonları çağırmak (`OnMouseUpdate`, `OnMouseRender`).

#### C++ Implementasyon Detayları

*   **Güncelleme ve Render Olayları**:
    *   `OnCameraUpdate()`: `CCameraManager` aracılığıyla aktif kamerayı günceller.
    *   `OnUIUpdate()`: `UI::CWindowManager` aracılığıyla tüm UI elemanlarını günceller.
    *   `OnUIRender()`: `UI::CWindowManager` aracılığıyla tüm UI elemanlarını çizer.
*   **Fare Olayları**:
    *   Orta tuş basma/bırakma olayları (`OnMouseMiddleButtonDown/Up`) kamera sürükleme işlemini başlatır/bitirir ve imleç şeklini/görünürlüğünü ayarlar.
    *   Fare tekerleği olayı (`OnMouseWheel`), UI tarafından işlenmezse kamera zoom işlemini tetikler (`ENABLE_MOUSE_WHEEL_TOP_WINDOW` makrosuna bağlı davranış).
    *   Fare hareket olayı (`OnMouseMove`), kamera sürükleniyorsa kamera pozisyonunu günceller ve Windows imleç pozisyonunu ayarlar. Her durumda UI yöneticisine fare hareketini bildirir.
    *   Sol ve sağ tuş olayları doğrudan `UI::CWindowManager`'ın ilgili olay işleme fonksiyonlarına yönlendirilir.
*   **Klavye Olayları**:
    *   `OnKeyDown()`: Basılan tuşa göre özel işlemler yapar (örn. `DIK_ESCAPE` için `RunPressEscapeKey`, `DIK_TAB` için hedef seçimi) ve ardından olayı UI yöneticisine iletir. Oyunun belirli durumlarında (örn. yükleme ekranı) klavye girdilerini yoksayar.
    *   `OnKeyUp()`: Tuş bırakma olayını UI yöneticisine iletir.
*   **IME Olayları**: Bir dizi `RunIME*` ve `OnIMEKeyDown` fonksiyonu, IME ile ilgili olayları (metin girişi, aday listesi, dil değiştirme vb.) `UI::CWindowManager`'a yönlendirir.
*   **Python Fare İşleyicisi**:
    *   `OnMouseUpdate()`: `UI::CWindowManager`'dan alınan fare pozisyonu ile Python fare işleyicisinin (`m_poMouseHandler`) `Update` metodunu çağırır.
    *   `OnMouseRender()`: Python fare işleyicisinin `Render` metodunu çağırır (yazılımsal imleç çizimi için).

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Girdi Merkezi**: Tüm kullanıcı girdilerini (fare, klavye) yakalayıp oyunun ilgili kısımlarına dağıtır.
*   **UI Etkileşimi**: Kullanıcının UI elemanlarıyla (pencereler, düğmeler) etkileşim kurmasını sağlar.
*   **Oyun Kontrolü**: Karakter hareketi, yetenek kullanımı, kamera kontrolü gibi temel oyun kontrollerinin girdi tarafını oluşturur.
*   **Çok Dilli Destek**: IME entegrasyonu ile farklı dillerde metin girişini mümkün kılar.

### `PythonApplicationLogo.cpp` (Başlangıç Logosu Oynatımı)

Bu dosya, `CPythonApplication` sınıfının oyun başlangıcında gösterilen logo animasyonlarını veya videolarını Microsoft DirectShow API'lerini kullanarak oynatmayla ilgili fonksiyonlarının implementasyonunu içerir.

#### Dosyanın Genel Amaçları

*   DirectShow kullanarak bir video dosyasını (genellikle logo animasyonu) yüklemek ve oynatmaya hazırlamak (`OnLogoOpen`).
*   Video oynatımı sırasında her video karesini yakalayıp bir Direct3D dokusuna (texture) aktarmak ve oynatma durumunu yönetmek (`OnLogoUpdate`).
*   Video karesini içeren dokuyu ekranda render etmek (`OnLogoRender`).
*   Video oynatımı bittiğinde veya kullanıcı tarafından kesildiğinde kullanılan tüm DirectShow kaynaklarını ve dokuyu temizlemek (`OnLogoClose`).

#### C++ Implementasyon Detayları

*   **DirectShow Entegrasyonu**:
    *   Video işleme grafiğini oluşturmak ve yönetmek için `IGraphBuilder`.
    *   Video karelerini ham veri olarak yakalamak için `ISampleGrabber`.
    *   Video oynatımını kontrol etmek için `IMediaControl`.
    *   Video penceresi özelliklerini (burada görünmez) ayarlamak için `IVideoWindow`.
    *   Video boyutu gibi bilgileri almak için `IBasicVideo`.
    *   Video oynatma olaylarını (tamamlanma, hata vb.) yakalamak için `IMediaEventEx`.
*   **`OnLogoOpen(char* szName)`**:
    *   Gerekli DirectShow COM nesnelerini (`CoCreateInstance` ile) ve video karesini tutacak `CGraphicImageTexture` (`m_pLogoTex`) nesnesini oluşturur.
    *   `RenderFile` ile video dosyasını yükler. `ISampleGrabber` filtresi, kareleri `MEDIASUBTYPE_RGB32` formatında yakalamak üzere ayarlanır.
    *   Video penceresi ekran dışına taşınır ve gizlenir, olay bildirimleri ana uygulama penceresine yönlendirilir.
*   **`OnLogoUpdate()`**:
    *   Her frame'de çağrılır. Videoyu başlatır (`m_pMediaCtrl->Run()`).
    *   `m_pSampleGrabber->GetCurrentBuffer()` ile mevcut video karesini `m_pCaptureBuffer`'a alır.
    *   Yakalanan kare verisini `m_pLogoTex` dokusuna kopyalar. Videonun en-boy oranına göre ekranda kaplayacağı alanı (`m_nLeft`, `m_nTop`, `m_nRight`, `m_nBottom`) hesaplar.
    *   `m_pMediaEvent->GetEvent()` ile videonun bitip bitmediğini (`EC_COMPLETE`) veya `ESC` tuşuna basılıp basılmadığını kontrol eder. Bunlardan biri gerçekleşirse 0 döndürerek logo oynatımını sonlandırır.
*   **`OnLogoRender()`**:
    *   `m_pLogoTex` dokusunu, `OnLogoUpdate` içinde hesaplanan koordinatlara `CPythonGraphic::instance().RenderTextureBox()` ile çizer. Doku filtrelemesi yumuşak görünüm için `D3DTEXF_LINEAR` olarak ayarlanır.
*   **`OnLogoClose()`**:
    *   Tüm DirectShow COM nesnelerini `Release()` ile serbest bırakır ve `m_pCaptureBuffer` ile `m_pLogoTex`'i siler. Doku filtreleme ayarlarını varsayılana döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Giriş Videoları**: Oyun ilk kez çalıştırıldığında veya lobiye dönüldüğünde geliştirici, yayıncı veya oyun logolarını içeren videoları göstermek için kullanılır.
*   **Markalaşma**: Oyunun kimliğini ve yapımcılarını oyuncuya sunar.
*   Genellikle kullanıcı `ESC` tuşuna basarak bu videoları atlayabilir.

