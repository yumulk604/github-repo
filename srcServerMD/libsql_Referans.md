# libsql Referans Kılavuzu

Bu dosya, `srcServer/Source/libsql` kütüphanesinin amacını, içerdiği temel bileşenleri ve fonksiyonları açıklamaktadır.

`libsql`, Metin2 sunucusunun veritabanı ile asenkron olarak iletişim kurmasını sağlayan temel kütüphanedir.

## Başlık Dosyaları (.h)

Bu dosyalar, `libsql` kütüphanesinin dışarıya sunduğu arayüzleri, sınıfları ve fonksiyon bildirimlerini içerir.

### `libsql.h`

*   **Amaç:** Kütüphanenin ana başlık dosyası olarak işlev görür.
*   **İçerik:** Sadece `AsyncSQL.h` dosyasını içerir. Bu, `AsyncSQL.h`'nin kütüphanenin temel arayüzünü tanımladığını gösterir.

### `stdafx.h`

*   **Amaç:** Ön derlenmiş başlık (precompiled header) dosyasıdır. Derleme sürelerini optimize etmek için kullanılır.
*   **İçerik:** `libthecore` kütüphanesinin `stdafx.h` dosyasını ve `AsyncSQL.h` dosyasını içerir. Bu, `libsql`'in `libthecore`'a bağımlı olduğunu gösterir.

### `AsyncSQL.h`

*   **Amaç:** MySQL veritabanı ile asenkron (eşzamansız) iletişim kurmak için temel sınıf (`CAsyncSQL`) ve ilgili veri yapılarını (`SQLResult`, `SQLMsg`) tanımlar.
*   **Temel Bileşenler:**
    *   **`SQLResult` Yapısı:** Tek bir MySQL sorgu sonucunu (veya çoklu sonuç setindeki bir sonucu) temsil eder. `MYSQL_RES` işaretçisini, satır sayısını (`uiNumRows`), etkilenen satır sayısını (`uiAffectedRows`) ve `INSERT` işlemi sonrası ID'yi (`uiInsertID`) içerir. Destructor'da `mysql_free_result` çağırarak kaynağı otomatik yönetir.
    *   **`SQLMsg` Yapısı:** Bir veritabanı sorgusu isteğini ve sonucunu içeren mesaj yapısıdır.
        *   Sorgu metnini (`stQuery`), isteğe bağlı kullanıcı verisini (`pvUserData`), sorgunun sonuç döndürüp döndürmeyeceğini (`bReturn`) ve MySQL bağlantı işaretçisini (`m_pkSQL`) tutar.
        *   Sorgu sonuçlarını (`SQLResult*` vektörü `vec_pkResult`) saklar ve `Store()`, `Get()`, `Next()` metodlarıyla bu sonuçlara erişimi yönetir.
        *   Olası MySQL hata numarasını (`uiSQLErrno`) saklar.
        *   Destructor'da içerdiği `SQLResult` nesnelerini temizler.
    *   **`CAsyncSQL` Sınıfı:** Asenkron SQL işlemlerini yöneten ana sınıftır.
        *   **Kurulum ve Bağlantı:** `Setup()` (bağlantı bilgilerini ayarlar), `Connect()` (veritabanına bağlanır), `IsConnected()`, `QueryLocaleSet()` (karakter setini ayarlar).
        *   **Sorgu İşlemleri:**
            *   `AsyncQuery()`: Sonuç beklenmeyen sorguları asenkron olarak gönderir.
            *   `ReturnQuery()`: Sonuç beklenen sorguları asenkron olarak gönderir, `pvUserData` ile ilişkilendirilebilir.
            *   `DirectQuery()`: Sorguyu *senkron* olarak çalıştırır ve sonucu hemen döndürür (asenkron modelin dışına çıkar).
        *   **Sonuç Yönetimi:** `PushResult()` (işlenen sorgu sonucunu kuyruğa ekler), `PopResult()` (sonuç kuyruğundan bir sonuç alır).
        *   **İş Parçacığı (Thread) Yönetimi:** `ChildLoop()` (sorguları işleyen ayrı iş parçacığının ana döngüsü), `Quit()` (iş parçacığını sonlandırır). Platforma göre (`__WIN32__`) `pthread` veya Windows `CRITICAL_SECTION`/`HANDLE` kullanır.
        *   **Kuyruk Yönetimi:** Sorgu (`m_queue_query`, `m_queue_query_copy`) ve sonuç (`m_queue_result`) için kuyruklar kullanır. Sorgu kuyruklarını korumak için mutex (`m_mtxQuery`, `m_mtxResult`) kullanır.
        *   **Diğer:** `EscapeString()` (SQL enjeksiyonunu önlemek için string kaçış işlemi yapar), `CountQuery()`/`CountResult()` (kuyruk boyutları), `GetSQLHandle()` (doğrudan MySQL bağlantısına erişim).
        *   İş parçacıkları arası iletişim için bir pipe (`m_aiPipe`) ve semafor (`m_sem`) kullanır.
    *   **`CAsyncSQL2` Sınıfı:** `CAsyncSQL`'den türemiştir ve ek olarak `SetLocale` metodu sunar (muhtemelen bağlantı sonrası karakter setini değiştirmek için).
*   **Bağımlılıklar:** `libthecore` (stdafx.h, log.h), `mysql` kütüphanesi, `Semaphore.h`.
*   **Kullanım Alanı:** Oyun sunucusunun veritabanı işlemlerini (oyuncu bilgileri, loglar, itemler vb.) ana oyun döngüsünü bloklamadan arka planda gerçekleştirmesini sağlar.

### `Semaphore.h`

*   **Amaç:** İş parçacıkları (thread) arasında eşzamanlılığı sağlamak için kullanılan bir semafor mekanizmasının platformdan bağımsız arayüzünü tanımlar (`CSemaphore` sınıfı).
*   **Temel Özellikler:**
    *   Platforma göre (`__WIN32__` direktifi ile) Windows semafor (`HANDLE`) veya POSIX semafor (`sem_t*`) kullanır.
    *   `Initialize()`: Semaforu başlatır.
    *   `Destroy()`/`Clear()`: Semaforu yok eder/temizler.
    *   `Wait()`: Semaforun sayacı sıfırdan büyük olana kadar bekler ve sonra sayacı bir azaltır (semaforu alır).
    *   `Release(count)`: Semaforun sayacını belirtilen `count` kadar artırır (varsayılan 1), bekleyen bir iş parçacığını uyandırabilir (semaforu bırakır).
*   **Bağımlılıklar:** Windows (`windows.h` - dolaylı), POSIX (`semaphore.h`).
*   **Kullanım Alanı:** `CAsyncSQL` içindeki iş parçacığı oluşturma ve başlatma sırasında senkronizasyon için kullanılabilir (örneğin, ana iş parçacığının, sorgu işleyen iş parçacığının hazır olmasını beklemesi).

### `Statement.h`

*   **Amaç:** MySQL C API'sinin Prepared Statement özelliğini kullanmak için bir sarmalayıcı sınıf (`CStmt`) tanımlar. Prepared Statement'lar, SQL sorgularını önceden derleyerek ve parametreleri ayrı göndererek performansı artırır ve SQL enjeksiyon riskini azaltır.
*   **`CStmt` Sınıfı Özellikleri:**
    *   `Prepare()`: Verilen SQL sorgu metnini MySQL sunucusunda hazırlar (`mysql_stmt_prepare`).
    *   `BindParam()`: Hazırlanmış sorgudaki parametre yer tutucularına (`?`) veri bağlar. Veri türünü (`enum_field_types`), verinin adresini (`void*`) ve maksimum uzunluğu alır, bunları `MYSQL_BIND` yapılarında saklar (`m_vec_param`).
    *   `BindResult()`: Sorgu sonucunda dönecek sütunları değişkenlere bağlar. Sütun türünü, değişken adresini ve maksimum uzunluğu alır, bunları `MYSQL_BIND` yapılarında saklar (`m_vec_result`).
    *   `Execute()`: Hazırlanmış ve parametreleri bağlanmış sorguyu çalıştırır (`mysql_stmt_execute`), etkilenen satır sayısını (`iRows`) döndürür.
    *   `Fetch()`: Çalıştırılan sorgunun bir sonraki satır sonucunu, `BindResult` ile bağlanmış değişkenlere çeker (`mysql_stmt_fetch`).
    *   `Error()`: Statement ile ilgili bir hata oluştuğunda loglama yapar.
    *   `Destroy()`: Ayrılmış kaynakları (statement handle, bind vektörleri) serbest bırakır.
*   **Dahili Üyeler:** `MYSQL_STMT` işaretçisi (`m_pkStmt`), sorgu metni (`m_stQuery`), parametre (`m_vec_param`) ve sonuç (`m_vec_result`) için `MYSQL_BIND` vektörleri, parametre uzunlukları (`m_puiParamLen`).
*   **Bağımlılıklar:** `AsyncSQL.h` (dolayısıyla `mysql.h`), `<string>`, `<vector>`.
*   **Kullanım Alanı:** Tekrarlanan sorguları daha verimli çalıştırmak, SQL enjeksiyonuna karşı güvenliği artırmak ve sorgu ile veriyi ayırmak için kullanılır.

### `Tellwait.h`

*   **Amaç:** İş parçacıkları (veya süreçler) arasında basit bir sinyal bekleme mekanizması için fonksiyon bildirimleri içerir. Bu mekanizma, bir iş parçacığının diğerinin belirli bir noktaya ulaştığını beklemesini veya bildirmesini sağlar.
*   **Not:** Bu dosyadaki fonksiyonlar **sadece Unix/Linux** gibi POSIX uyumlu sistemlerde (`#ifndef __WIN32__`) tanımlıdır ve kullanılır. Windows için bir karşılığı bu başlıkta yoktur.
*   **Bildirilen Fonksiyonlar:**
    *   `TELL_WAIT()`: Sinyal mekanizmasını başlatır.
    *   `WAIT_CHILD()`: Ebeveyn iş parçacığı/süreci, çocuktan sinyal gelene kadar bekler.
    *   `TELL_CHILD(pid_t pid)`: Ebeveyn, belirli bir çocuk iş parçacığına/sürecine sinyal gönderir.
    *   `WAIT_PARENT()`: Çocuk iş parçacığı/süreci, ebeveynden sinyal gelene kadar bekler.
    *   `TELL_PARENT(pid_t pid)`: Çocuk, ebeveyne sinyal gönderir.
*   **Bağımlılıklar:** Muhtemelen POSIX sinyalleri (`signal.h`) ve süreç/iş parçacığı yönetimi (`pthread.h`, `unistd.h`) ile ilgilidir (uygulaması `Tellwait.cpp`'de görülecektir).
*   **Kullanım Alanı:** `CAsyncSQL` içindeki iş parçacığı oluşturma ve başlatma sırasında senkronizasyon için kullanılabilir (örneğin, ana iş parçacığının, sorgu işleyen iş parçacığının hazır olmasını beklemesi).

--- 

## Kaynak Dosyalar (.cpp)

Bu dosyalar, başlık dosyalarında bildirilen sınıfların ve fonksiyonların gerçek implementasyonlarını içerir.

### `AsyncSQL.cpp`

*   **Implementasyon Detayları:**
    *   `CAsyncSQL` sınıfının kurucu (`CAsyncSQL()`) ve yıkıcı (`~CAsyncSQL()`) metodlarını içerir. Yıkıcı metod `Quit()` ve `Destroy()` çağırarak iş parçacığını durdurur ve kaynakları (MySQL bağlantısı, mutex'ler) temizler.
    *   `Setup()`: Bağlantı bilgilerini (host, user, pass, db, locale, port) sınıf üyelerine atar. Eğer `bNoThread` false ise, platforma özgü mutex'leri (`pthread_mutex_t` veya `CRITICAL_SECTION`) ve semaforu (`CSemaphore`) başlatır, ardından sorguları işlemek üzere `AsyncSQLThread` fonksiyonunu ayrı bir iş parçacığında çalıştırır.
    *   `Connect()`: `mysql_init()` ile MySQL bağlantısını başlatır, `mysql_options()` ile karakter seti (`MYSQL_SET_CHARSET_NAME`) ve otomatik tekrar bağlanma (`MYSQL_OPT_RECONNECT`) gibi seçenekleri ayarlar, `mysql_real_connect()` ile asıl bağlantıyı kurar. Bağlantı başarılı olursa `m_bConnected` true olur.
    *   `QueryLocaleSet()`: `mysql_set_character_set()` ile bağlantının karakter setini ayarlar.
    *   `AsyncQuery()` ve `ReturnQuery()`: Yeni bir `SQLMsg` nesnesi oluşturur, sorguyu ve diğer bilgileri (kullanıcı verisi, return flag) ayarlar, `PushQuery()` ile sorgu kuyruğuna ekler ve semaforu (`m_sem.Release()`) serbest bırakarak iş parçacığını uyandırır.
    *   `DirectQuery()`: Doğrudan `mysql_real_query()` kullanarak sorguyu çalıştırır, `mysql_store_result()` ile sonuçları alır, bunları yeni bir `SQLMsg` içine koyar (`Store()` metodu ile) ve bu mesajı döndürür. Hata durumunda `mysql_errno()` ve `mysql_error()` ile loglama yapar.
    *   `ChildLoop()`: İş parçacığının ana döngüsüdür. `m_bEnd` false olduğu sürece çalışır:
        *   `m_sem.Wait()` ile yeni bir sorgu gelene kadar bekler.
        *   `CopyQuery()` ile ana sorgu kuyruğundaki sorguları kendi işlem kuyruğuna (`m_queue_query_copy`) kopyalar (muhtemelen mutex kilidini kısa tutmak için).
        *   İşlem kuyruğundaki her sorgu için:
            *   `mysql_real_query()` ile sorguyu çalıştırır.
            *   Hata olursa (`mysql_errno()`) hata kodunu `SQLMsg`'e kaydeder.
            *   Başarılı ve sonuç bekleniyorsa (`bReturn`), `SQLMsg::Store()` ile sonuçları (`MYSQL_RES`) alır ve saklar.
            *   İşlenen `SQLMsg`'i `PushResult()` ile sonuç kuyruğuna ekler.
        *   Bağlantı kopması gibi kritik hatalarda (`CR_SERVER_GONE_ERROR`, `CR_SERVER_LOST`) döngüden çıkabilir.
    *   **Kuyruk Yönetimi:**
        *   `PushQuery()`/`PopQuery()`/`PeekQuery()`: Ana sorgu kuyruğunu (`m_queue_query`) mutex (`m_mtxQuery`) koruması altında yönetir.
        *   `PushResult()`/`PopResult()`: Sonuç kuyruğunu (`m_queue_result`) mutex (`m_mtxResult`) koruması altında yönetir.
        *   `CopyQuery()`/`PeekQueryFromCopyQueue()`/`PopQueryFromCopyQueue()`: İş parçacığının kendi sorgu kopyası kuyruğunu (`m_queue_query_copy`) yönetir.
    *   `EscapeString()`: `mysql_real_escape_string()` fonksiyonunu kullanarak SQL enjeksiyonuna karşı güvenli string kaçış işlemi sağlar.
    *   `CAsyncSQL2::SetLocale()`: Sadece locale string'ini ayarlar, asıl işlemi yapmaz (muhtemelen bağlantı sırasında veya `QueryLocaleSet` ile kullanılır).
*   **Notlar:**
    *   Platforma özgü (`#ifdef __WIN32__`) kod blokları içerir (thread oluşturma, mutex kullanımı).
    *   Kritik MySQL hatalarını loglar (`sys_err`).
    *   Basit bir profilleyici sınıfı (`cProfiler`) içerir ancak kodun içinde aktif olarak kullanılmıyor gibi görünmektedir.

### `Semaphore.cpp`

*   **Implementasyon Detayları:**
    *   `Semaphore.h`'de bildirilen `CSemaphore` sınıfının platforma özgü uygulamalarını içerir.
    *   **POSIX (Unix/Linux):**
        *   `Initialize`: `sem_init` ile isimsiz bir semafor oluşturur (başlangıç değeri 0).
        *   `Wait`: `sem_wait` fonksiyonunu çağırır. `EINTR` (kesintiye uğrayan sistem çağrısı) hatasını ele alarak beklemeye devam eder.
        *   `Release`: Belirtilen sayıda `sem_post` çağırır.
        *   `Destroy`/`Clear`: `sem_destroy` ile semaforu yok eder ve belleği serbest bırakır.
    *   **Windows:**
        *   `Initialize`: `CreateSemaphore` ile bir semafor nesnesi oluşturur (başlangıç değeri 0, maksimum değer 32).
        *   `Wait`: `WaitForSingleObject` ile semaforun sinyallenmesini (sayacının artmasını) bekler.
        *   `Release`: `ReleaseSemaphore` ile semaforun sayacını belirtilen miktar kadar artırır.
        *   `Destroy`/`Clear`: `CloseHandle` ile semafor nesnesini kapatır.

### `Statement.cpp`

*   **Implementasyon Detayları:**
    *   `Statement.h`'de bildirilen `CStmt` sınıfının metodlarını uygular.
    *   **Kurucu/Yıkıcı:** Kurucu (`CStmt()`) üyeleri NULL veya 0'a ayarlar. Yıkıcı (`~CStmt()`) `Destroy()`'u çağırır.
    *   `Destroy()`: `mysql_stmt_close()` ile statement handle'ını kapatır ve `m_puiParamLen` için ayrılan belleği `free()` ile serbest bırakır.
    *   `Error()`: `mysql_stmt_errno()` ve `mysql_stmt_error()` kullanarak statement hatasını loglar (`sys_log`).
    *   `Prepare()`: `mysql_stmt_init()` ile bir statement handle oluşturur, sorguyu saklar (`m_stQuery`), `mysql_stmt_prepare()` ile sorguyu MySQL sunucusunda hazırlar. Sorgudaki `?` sayısını sayarak parametre vektörünü (`m_vec_param`) ve parametre uzunluk dizisini (`m_puiParamLen`) boyutlandırır/başlatır. Sonuç vektörünü (`m_vec_result`) sabit bir boyutla (48) başlatır ve `mysql_stmt_bind_result()` ile bağlar.
    *   `BindParam()`: Gelen parametre bilgilerini (`type`, `p`, `iMaxLen`) sıradaki `MYSQL_BIND` yapısına (`m_vec_param` içinde) atar. Parametre uzunluğu için `m_puiParamLen` dizisindeki ilgili elemanın adresini verir. Tüm parametreler bağlandığında (`m_uiParamCount == m_vec_param.size()`), `mysql_stmt_bind_param()` ile tüm parametreleri tek seferde bağlar.
    *   `BindResult()`: Gelen sonuç bağlama bilgilerini (`type`, `p`, `iMaxLen`) sıradaki `MYSQL_BIND` yapısına (`m_vec_result` içinde) atar. Bu işlem `Prepare` sırasında zaten yapıldığı için burada sadece bilgileri doldurur.
    *   `Execute()`: Önce tüm parametrelerin bağlanıp bağlanmadığını kontrol eder. String türündeki parametrelerin uzunluklarını `strlen` ile hesaplayıp `m_puiParamLen` dizisine yazar. `mysql_stmt_execute()` ile sorguyu çalıştırır. `mysql_stmt_store_result()` ile sunucudaki tüm sonuçları istemci tarafına çeker. `mysql_stmt_num_rows()` ile satır sayısını alır ve `iRows` üyesine atar. Başarılı olursa 1 döndürür.
    *   `Fetch()`: `mysql_stmt_fetch()` fonksiyonunu çağırır ve sonucunu tersine çevirerek döndürür (API 0 döndürürse başarılı, yani `Fetch` true döndürür). Bu fonksiyon, `BindResult` ile bağlanmış değişkenlere bir sonraki satırın verilerini doldurur.

### `Tellwait.cpp`

*   **Implementasyon Detayları:**
    *   Bu dosya, **sadece Unix/Linux** gibi POSIX uyumlu sistemlerde (`#ifndef __WIN32__`) derlenir ve `Tellwait.h`'deki fonksiyonları uygular.
    *   Bir sinyal işleyici (`sig_usr`) tanımlar. Bu işleyici hem `SIGUSR1` hem de `SIGUSR2` sinyallerini yakalar ve global bir bayrak (`sigflag`) ayarlar.
    *   `TELL_WAIT()`:
        *   `SIGUSR1` ve `SIGUSR2` için `sig_usr` işleyicisini ayarlar (`signal`).
        *   Bu iki sinyali bloke etmek için bir sinyal maskesi (`newmask`) oluşturur.
        *   `sigprocmask(SIG_BLOCK, ...)` ile bu sinyalleri bloke eder ve mevcut sinyal maskesini (`oldmask`) saklar. Bu, bekleme sırasında sinyallerin hemen işlenmesini engeller.
    *   `TELL_PARENT(pid)`: Belirtilen process ID'sine (`pid`) `SIGUSR2` sinyalini gönderir (`kill`).
    *   `WAIT_PARENT()`:
        *   `sigflag` sıfır olduğu sürece `sigsuspend(&zeromask)` ile bekler. `sigsuspend` atomik olarak sinyal maskesini geçici olarak `zeromask` (yani tüm sinyallere izin veren) ile değiştirir ve bir sinyal gelene kadar bekler.
        *   `sig_usr` işleyicisi `SIGUSR1` veya `SIGUSR2` aldığında `sigflag` 1 olur ve döngü biter.
        *   `sigflag`'ı sıfırlar.
        *   `sigprocmask(SIG_SETMASK, ...)` ile orijinal sinyal maskesini (`oldmask`) geri yükler.
    *   `TELL_CHILD(pid)`: Belirtilen process ID'sine (`pid`) `SIGUSR1` sinyalini gönderir (`kill`).
    *   `WAIT_CHILD()`: `WAIT_PARENT` ile aynı mantıkla çalışır, `SIGUSR1` veya `SIGUSR2` sinyalini bekler.
*   **Not:** Bu mekanizma, genellikle `fork()` ile oluşturulan çocuk süreçler ve ebeveyn süreç arasındaki basit senkronizasyonlar için tasarlanmıştır. İş parçacıkları (threads) için de kullanılabilir, ancak daha modern eşzamanlılık mekanizmaları (mutex, condition variable, semaphore) genellikle iş parçacıkları için daha uygundur. `CAsyncSQL` içinde `pthread` kullanıldığı göz önüne alındığında, bu mekanizmanın `CAsyncSQL`'in kendi iş parçacığı yönetimiyle doğrudan entegre olup olmadığı belirsizdir, ancak potansiyel olarak başlatma aşamasında kullanılabilir.

--- 

## Proje/Derleme Dosyaları

### `Makefile`

*   **Amaç:** `libsql` kütüphanesini Unix/Linux benzeri sistemlerde derlemek için talimatlar içerir.
*   **Derleyici:** `clang++90` (Clang C++ derleyicisinin 9.0 sürümü).
*   **Çıktı:** `libsql.a` (statik kütüphane).
*   **Derleme Seçenekleri (`CFLAGS`/`CXXFLAGS`):**
    *   `-m32`: 32-bit çıktı oluşturur.
    *   `-Wall`: Tüm yaygın derleyici uyarılarını etkinleştirir.
    *   `-O2`: Optimizasyon seviyesi 2.
    *   `-pipe`: Derleme adımları arasında dosya yerine pipe kullanır (hızlandırabilir).
    *   `-D_THREAD_SAFE`: İş parçacığı güvenliği ile ilgili derleme seçeneklerini etkinleştirir.
    *   `-fno-exceptions`: C++ istisnalarını (exceptions) devre dışı bırakır.
    *   `-std=c++2a`: C++20 standardını kullanır (C++2a, C++20'nin geliştirme sırasındaki adıydı).
*   **Include Yolları (`IFLAGS`):** `/usr/local/include/` yolunu içerir (muhtemelen MySQL başlık dosyaları için).
*   **Hedefler:**
    *   `default`: Ana hedef, `libsql.a`'yı oluşturur.
    *   `$(BIN)` (`libsql.a`): Nesne dosyalarını (`.o`) derler ve `ar cru` ile statik kütüphaneyi oluşturur, `ranlib` ile indeksler.
    *   `clean`: Derlenmiş nesne dosyalarını (`.o`) ve statik kütüphaneyi (`.a`) siler.
    *   `dep`: Kaynak dosyaları arasındaki bağımlılıkları hesaplayıp `Depend` dosyasına yazar.
*   **Kaynak Dosyalar:** `AsyncSQL.cpp`, `Semaphore.cpp`, `Tellwait.cpp`, `Statement.cpp`.

### `libsql.vcxproj`

*   **Dosya Türü:** Visual Studio C++ Proje Dosyası (XML).
*   **Amaç:** `libsql` kütüphanesinin Windows (Win32) platformunda Visual Studio (v143 araç takımı - VS 2022) ile nasıl derleneceğini tanımlar.
*   **Proje Tipi:** Statik Kütüphane (`.lib`).
*   **Temel Ayarlar:**
    *   **Platform:** Win32.
    *   **Konfigürasyonlar:** `Debug` ve `Release`.
    *   **Çıktı Dosyaları:** `libsql_d.lib` (Debug), `libsql.lib` (Release). Dosyalar doğrudan `libsql` klasörüne kaydedilir (`OutDir = $(ProjectDir)`).
    *   **Ara Dizinler:** `win32/Debug/` ve `win32/Release/`.
    *   **Include Yolları:** `../../External/MySQL/6.0.2/win32` ve `../../External/include`. Bu, MySQL 6.0.2 için başlık dosyalarının `External` klasörü altında olduğunu gösterir.
    *   **Önişlemci Tanımları:** `WIN32`, `__WIN32__` ve konfigürasyona özel (`_DEBUG`/`NDEBUG`). `_USE_32BIT_TIME_T` tanımı burada da mevcut (Debug'da). `_CRT_SECURE_NO_DEPRECATE` veya `_WINSOCK_DEPRECATED_NO_WARNINGS` burada yok.
    *   **Çalışma Zamanı Kütüphanesi:** `/MTd` (Debug), `/MT` (Release).
    *   **Derlenen Kaynaklar:** `AsyncSQL.cpp`, `Semaphore.cpp`, `Statement.cpp`, `Tellwait.cpp`.
    *   **Dil Standardı:** En son C++ standardı (`stdcpplatest`).
*   **Not:** Bu dosya, kütüphanenin Windows ortamında derlenmesi için gerekli yapılandırma bilgilerini ve özellikle MySQL kütüphanesi gibi dış bağımlılıkların yerini belirtir.