# db Referans Kılavuzu - Bölüm 2

Bu dosya, `db_Referans.md` dosyasının devamıdır ve `srcServer/Source/db/src` klasöründeki kalan bileşenlerin belgelenmesini içerir.

---

### `CsvReader.h` / `CsvReader.cpp` (`cCsvFile`, `cCsvRow`, `cCsvAlias`, `cCsvTable` Sınıfları)

*   **Amaç:** Virgülle ayrılmış değerler (CSV) formatındaki dosyaları okumak, yazmak ve bu dosyalardaki verilere kolayca erişmek için bir dizi yardımcı sınıf sağlar.
*   **Sınıflar:**
    *   **`cCsvRow`:** Bir CSV dosyasındaki tek bir satırı temsil eder. `std::vector<std::string>` sınıfından türetilmiştir. Her bir hücreye (token) string olarak erişim sağlar. Ayrıca `AsInt()`, `AsDouble()`, `AsString()` gibi yardımcı fonksiyonlarla hücredeki veriyi doğrudan istenen tipe dönüştürme imkanı sunar. İsim tabanlı erişim için `cCsvAlias` ile birlikte kullanılabilir.
    *   **`cCsvAlias`:** CSV sütunlarına isim (alias) atayarak sütun indeksleri yerine isimlerle erişimi kolaylaştırır. `AddAlias()` ile isim ve indeks eşleşmesi eklenir. `operator[]` ile hem indeksten isme hem de isimden indekse (küçük harfe duyarsız) dönüşüm sağlar.
    *   **`cCsvFile`:** Bir CSV dosyasının tamamını temsil eder.
        *   `Load(fileName, separator, quote)`: Belirtilen dosyayı okur, satırları ve hücreleri ayrıştırır (`separator` ve `quote` karakterlerini dikkate alarak). Satırları `cCsvRow` nesneleri olarak `m_Rows` (bir `std::vector<cCsvRow*>`) içinde saklar. Tırnak içindeki özel karakterleri (virgül, tırnak işareti) ve çift tırnakla escape edilmiş tırnak işaretlerini doğru şekilde işler. Yorum satırlarını (`#`) atlar.
        *   `Save(fileName, append, separator, quote)`: Bellekteki `m_Rows` içeriğini belirtilen dosyaya yazar. Gerekirse özel karakterleri tırnak içine alır ve tırnak işaretlerini çift tırnakla escape eder.
        *   `Destroy()`: Bellekteki tüm `cCsvRow` nesnelerini siler.
        *   `operator[]`: Belirli bir satırdaki `cCsvRow` nesnesine erişim sağlar.
        *   `GetRowCount()`: Toplam satır sayısını döndürür.
    *   **`cCsvTable`:** `cCsvFile` ve `cCsvAlias`'ı birleştirerek tablo benzeri bir yapı sunar. Sütun isimleri (alias) tanımlandıktan sonra `Load()` ile dosyayı yükler. `Next()` fonksiyonu ile satır satır ilerler ve `AsInt(name)`, `AsDouble(name)`, `AsStringByIndex(index)` gibi fonksiyonlarla mevcut satırdaki verilere isim veya indeks ile erişim sağlar.
*   **Önem:** Özellikle yapılandırma dosyalarını veya basit veritabanı tablolarını CSV formatında okumak ve işlemek için kullanışlı bir araç seti sunar. Sütun isimleri (alias) kullanımı, kodun okunabilirliğini ve bakımını kolaylaştırır.
*   **Implementasyon Detayları (`CsvReader.cpp`):**
    *   `Load()` fonksiyonu, dosyayı satır satır okur ve her satırı karakter karakter işleyerek `STATE_NORMAL` ve `STATE_QUOTE` durumlarına göre hücreleri (token) ayırır.
    *   `Save()` fonksiyonu, her hücreyi kontrol eder; özel karakter içeriyorsa tırnak içine alır ve içindeki tırnakları çift tırnakla değiştirir.
    *   `cCsvAlias`, isimleri küçük harfe çevirerek `std::map` içinde saklar, böylece isimle erişim büyük/küçük harfe duyarsız olur.
*   **Bağımlılıklar:** `stdafx.h`, `CsvReader.h`, `<fstream>`, `<algorithm>`, `<cassert>`.

### `grid.h` / `grid.cpp` (`CGrid` Sınıfı)

*   **Amaç:** İki boyutlu bir ızgara (grid) yapısını temsil eder ve bu ızgara üzerinde belirli boyutlardaki boş alanları bulma, işaretleme ve temizleme işlevleri sunar. Genellikle oyun içi envanter sistemlerinde eşyaların yerleştirileceği boş yerleri bulmak için kullanılır.
*   **Veri Yapıları ve Üyeler:**
    *   `m_iWidth`, `m_iHeight` (private `int`): Izgaranın genişliğini ve yüksekliğini saklar.
    *   `m_pGrid` (private `char*`): Izgara verisini tutan tek boyutlu bir karakter dizisi. Her hücre doluysa `1` (true), boşsa `0` (false) değerini alır.
*   **Önemli Fonksiyonlar:**
    *   **`CGrid(int w, int h)` (Yapıcı):** Belirtilen genişlik ve yükseklikte yeni bir boş ızgara oluşturur.
    *   **`CGrid(CGrid* pkGrid, int w, int h)` (Kopyalayıcı Yapıcı):** Mevcut bir `CGrid` nesnesinden veri kopyalayarak yeni bir ızgara oluşturur (boyutlar farklı olabilir, kesişim alanı kopyalanır).
    *   **`~CGrid()` (Yıkıcı):** `m_pGrid` için ayrılan belleği serbest bırakır.
    *   **`Clear()`:** Tüm ızgarayı boşaltır (tüm hücreleri `0` yapar).
    *   **`FindBlank(int w, int h)`:** Izgara üzerinde sol üstten başlayarak, belirtilen `w` genişliğinde ve `h` yüksekliğinde tamamen boş olan ilk alanı arar. Bulursa, boş alanın sol üst hücresinin indeksini (tek boyutlu dizi indeksi) döndürür. Bulamazsa `-1` döndürür.
    *   **`IsEmpty(int iPos, int w, int h)`:** Verilen pozisyondan (`iPos`) başlayan `w`x`h` boyutundaki alanın tamamen boş olup olmadığını kontrol eder. Izgara sınırlarını da kontrol eder.
    *   **`Put(int iPos, int w, int h)`:** Verilen pozisyondan başlayan `w`x`h` boyutundaki alanı dolu olarak işaretler (`1` yapar). İşaretlemeden önce `IsEmpty` ile kontrol yapar, eğer alan boş değilse `false` döner ve işaretlemez. Başarılı olursa `true` döner.
    *   **`Get(int iPos, int w, int h)`:** Verilen pozisyondan başlayan `w`x`h` boyutundaki alanı boş olarak işaretler (`0` yapar). Sınır kontrolü yapar.
    *   **`Print()`:** Izgaranın mevcut durumunu konsola yazdırır (debug amaçlı).
    *   **`GetSize()`:** Izgaranın toplam hücre sayısını (`width * height`) döndürür.
*   **Önem:** Envanter, depo gibi ızgara tabanlı sistemlerde eşya yerleşimi ve boş yer kontrolü için temel bir altyapı sağlar.
*   **Bağımlılıklar:** `grid.h`, `string.h`, `stdio.h`, `libthecore/memcpy.h`, `common/stl.h`.

### `ItemIDRangeManager.h` / `ItemIDRangeManager.cpp` (`CItemIDRangeManager` Sınıfı)

*   **Amaç:** Oyunda yeni oluşturulacak eşyalar için kullanılabilir ID aralıklarını belirlemek ve yönetmek. DB sunucusu, bu sınıf aracılığıyla hangi ID aralığının boş olduğunu ve yeni eşyalar için kullanılabileceğini belirler, bu bilgiyi oyun sunucularına iletir. Bu, farklı sunucuların veya işlemlerin aynı ID'yi kullanmasını engeller. Singleton olarak tasarlanmıştır.
*   **Veri Yapıları ve Üyeler:**
    *   `cs_dwMaxItemID`: Kullanılabilecek maksimum eşya ID'si.
    *   `cs_dwMinimumRange`: Bir ID aralığının minimum boyutu (örn. 10 milyon).
    *   `cs_dwMinimumRemainCount`: Bir aralığın kullanılabilir sayılabilmesi için içinde kalması gereken minimum boş ID sayısı (örn. 10 bin).
    *   `m_listData` (private `std::list<TItemIDRangeTable>`): Kullanılabilir (boş olduğu tespit edilmiş) ID aralıklarını (`TItemIDRangeTable` yapıları) bir liste içinde saklar. `TItemIDRangeTable`, `common/tables.h` içinde tanımlıdır ve bir aralığın başlangıcını (`dwMin`), bitişini (`dwMax`) ve o aralıkta kullanılmaya başlanacak ilk ID'yi (`dwUsableItemIDMin`) içerir.
*   **Önemli Fonksiyonlar:**
    *   **`Build()`:** DB sunucusu başlatılırken çağrılır (`CClientManager::InitializeTables` içinde).
        *   Mümkün olan tüm ID aralıklarını (`cs_dwMinimumRange` boyutunda) döngü ile kontrol eder.
        *   Halihazırda `CClientManager` tarafından bilinen (kullanılan) aralığı atlar.
        *   Her potansiyel aralık için `BuildRange()` fonksiyonunu çağırır.
        *   `BuildRange()` başarılı olursa (aralık kullanılabilirse), aralığı `m_listData` listesine ekler.
    *   **`BuildRange(DWORD dwMin, DWORD dwMax, TItemIDRangeTable& range)`:**
        *   Belirtilen `dwMin` ve `dwMax` arasındaki ID aralığının kullanılabilir olup olmadığını kontrol eder.
        *   `item` ve (varsa) `private_shop_item` tablolarında bu aralıktaki en büyük ID'yi (`MAX(id)`) sorgular.
        *   Bulunan maksimum ID'den sonraki ID'yi (`dwUsableItemIDMin`) başlangıç noktası olarak belirler.
        *   Eğer aralıkta kalan boş ID sayısı `cs_dwMinimumRemainCount`'tan az ise aralığı kullanılamaz olarak işaretler (`false` döner).
        *   Ayrıca, `dwUsableItemIDMin` ile `dwMax` arasında veritabanında herhangi bir eşya olup olmadığını tekrar kontrol eder (güvenlik önlemi). Varsa `false` döner.
        *   Tüm kontrollerden geçerse, `range` parametresini doldurur ve `true` döner.
    *   **`GetRange()`:**
        *   Bir oyun sunucusuna yeni bir ID aralığı gerektiğinde (`CClientManager` tarafından) çağrılır.
        *   `m_listData` listesinin başından bir aralık alır.
        *   Alınan aralığın hala geçerli olup olmadığını (başka bir oyun sunucusu tarafından alınmamış olmasını) tüm bağlı oyun sunucularına (`CPeer::CheckItemIDRangeCollision`) sorarak kontrol eder (`FCheckCollision` functor'ı ile).
        *   Eğer aralık hala kullanılabilirse, bu aralığı (`TItemIDRangeTable`) döndürür.
        *   Eğer liste boşsa veya listedeki tüm aralıklar zaten alınmışsa, sıfırlanmış bir aralık döndürür ve hata logu basar.
    *   **`UpdateRange(DWORD dwMin, DWORD dwMax)`:**
        *   Belirli bir aralığın (muhtemelen bir oyun sunucusu tarafından kullanıldıktan sonra bir kısmı) tekrar kontrol edilip listeye eklenmesi için kullanılır. `BuildRange()`'i çağırır ve başarılı olursa listeye ekler.
*   **Önem:** Eşya oluşturma işlemlerinde ID çakışmalarını önleyerek veri bütünlüğünü sağlar. DB sunucusu, oyun sunucularına kontrollü bir şekilde benzersiz ID aralıkları dağıtır.
*   **Bağımlılıklar:** `stdafx.h`, `ItemIDRangeManager.h`, `Main.h`, `DBManager.h`, `ClientManager.h`, `Peer.h`, `common/tables.h`.

### `Lock.h` / `Lock.cpp` (`CLock` Sınıfı)

*   **Amaç:** İşletim sistemine özel mutex (karşılıklı dışlama) mekanizmalarını (Windows için `CRITICAL_SECTION`, diğerleri için `pthread_mutex_t`) sarmalayarak platformdan bağımsız bir kilit arayüzü sunar. Çoklu iş parçacığı (multi-thread) ortamlarında paylaşılan verilere veya kritik kod bölümlerine aynı anda sadece bir iş parçacığının erişmesini sağlamak için kullanılır.
*   **Veri Yapıları ve Üyeler:**
    *   `m_lock` (private `lock_t`): İşletim sistemine özgü kilit nesnesini tutar (`CRITICAL_SECTION` veya `pthread_mutex_t`).
    *   `m_bLocked` (private `bool`): Kilidin şu anda alınıp alınmadığını takip eder (debug/assert amaçlı).
*   **Önemli Fonksiyonlar:**
    *   **`Initialize()`:** Kilit nesnesini başlatır (`pthread_mutex_init` veya `InitializeCriticalSection`).
    *   **`Destroy()`:** Kilit nesnesini yok eder (`pthread_mutex_destroy` veya `DeleteCriticalSection`). Kilidin serbest bırakılmış olduğundan emin olmak için bir `assert` içerir.
    *   **`Lock()`:** Kilidi almaya çalışır. Eğer kilit başka bir iş parçacığı tarafından tutuluyorsa, kilit serbest bırakılana kadar mevcut iş parçacığını bloke eder. Kilidi başarıyla aldığında `m_bLocked`'ı `true` yapar.
    *   **`Unlock()`:** Tutulan kilidi serbest bırakır, böylece bekleyen başka bir iş parçacığı kilidi alabilir. `m_bLocked`'ı `false` yapar. Kilidin önceden alınmış olduğundan emin olmak için bir `assert` içerir.
    *   **`Trylock()`:** Kilidi almaya çalışır, ancak bloke etmez. Kilidi hemen alabilirse başarılı olur (genellikle 0 veya pozitif değer döner), alamazsa başarısız olur (genellikle negatif veya 0 döner - platforma bağlı).
*   **Önem:** DB sunucusunun olası çoklu iş parçacıklı bölümlerinde (özellikle `libsql` gibi kütüphaneler veya paylaşılan veri yapılarına erişim sırasında) veri yarışlarını (race conditions) ve tutarsızlıkları önlemek için temel bir senkronizasyon aracı sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `Lock.h`, (Windows için)`<windows.h>`, (Diğerleri için)`<pthread.h>`, `<cassert>`.

### `Monarch.h` / `Monarch.cpp` (`CMonarch` Sınıfı)

*   **Amaç:** İmparatorlukların liderlerini (Monarch/Kral), krallık seçimlerini, krallık vergilerini/paralarını yönetir. Singleton olarak tasarlanmıştır. Krallık bilgilerini, adayları ve seçim oylarını bellekte tutar, veritabanı ile senkronize eder ve oyun sunucularına bilgi sağlar.
*   **Veri Yapıları ve Üyeler (`Monarch.h`):**
    *   `TMonarchInfo`: Her imparatorluk için kralın PID'sini (`pid`), ismini (`name`), krallık parasını (`money`) ve seçilme tarihini (`date`) tutan yapı. `m_MonarchInfo` üyesinde saklanır.
    *   `MonarchElectionInfo`: Bir oyuncunun seçimde kime oy verdiğini (`pid`, `selectedpid`) tutan yapı. `m_map_MonarchElection` içinde saklanır.
    *   `MonarchCandidacy`: Seçimde aday olan oyuncunun bilgilerini (`pid`, `name`) tutan yapı. `m_vec_MonarchCandidacy` vektöründe saklanır.
    *   `MAP_MONARCHELECTION`: Oyuncu PID'sinden `MonarchElectionInfo*`'a eşleme yapan map.
    *   `VEC_MONARCHCANDIDACY`: `MonarchCandidacy` yapılarının vektörü.
*   **Önemli Fonksiyonlar (`Monarch.cpp`):**
    *   **`LoadMonarch()`:** Sunucu başlangıcında (`CClientManager::InitializeMonarch`) çağrılır. `monarch` tablosundan mevcut kralların bilgilerini (`player` tablosu ile join yaparak) okur ve `m_MonarchInfo` yapısını doldurur.
    *   **`SetMonarch(const char* name)`:** Belirtilen isimdeki oyuncuyu ilgili imparatorluğun kralı olarak ayarlar. `player` tablosundan oyuncunun PID ve imparatorluğunu bulur, `m_MonarchInfo`'yu günceller ve `monarch` tablosuna `REPLACE INTO` ile yeni kralı kaydeder.
    *   **`DelMonarch(int Empire)` / `DelMonarch(const char* name)`:** Belirtilen imparatorluğun kralını veya ismi verilen oyuncu kral ise krallığını siler. `monarch` tablosundan ilgili kaydı siler ve `m_MonarchInfo`'yu sıfırlar.
    *   **`IsMonarch(int Empire, DWORD pid)`:** Verilen oyuncunun belirtilen imparatorluğun kralı olup olmadığını kontrol eder.
    *   **`AddMoney(int Empire, int64_t Money)` / `DecMoney(int Empire, int64_t Money)`:** İlgili imparatorluğun krallık parasına ekleme/çıkarma yapar. Limiti (2 milyar) kontrol eder. Bellekteki `m_MonarchInfo.money`'yi günceller ve `monarch` tablosuna `UPDATE` sorgusu gönderir.
    *   **`TakeMoney(int Empire, DWORD pid, int64_t Money)`:** Kralın krallık kasasından para çekmesini sağlar. Kral olup olmadığını ve yeterli para olup olmadığını kontrol eder. `DecMoney` gibi parayı azaltır ve DB'yi günceller.
    *   **`AddCandidacy(DWORD pid, const char* name)` / `DelCandidacy(const char* name)`:** Krallık seçimi için aday ekler/siler. Bellekteki `m_vec_MonarchCandidacy` vektörünü günceller ve `monarch_candidacy` tablosuna `INSERT`/`DELETE` sorgusu gönderir.
    *   **`IsCandidacy(DWORD pid)`:** Verilen oyuncunun zaten aday olup olmadığını kontrol eder.
    *   **`VoteMonarch(DWORD pid, DWORD selectedpid)`:** Bir oyuncunun seçimde oy kullanmasını sağlar. Oyuncunun daha önce oy kullanıp kullanmadığını `m_map_MonarchElection` ile kontrol eder. İlk oyu ise bilgileri belleğe ekler ve `monarch_election` tablosuna `INSERT` sorgusu gönderir.
    *   **`ElectMonarch()`:** Seçim sonuçlarını hesaplar (ancak bu fonksiyonun implementasyonu eksik veya sadece oyları sayıyor gibi görünüyor, kralı belirlemiyor). `m_map_MonarchElection`'daki oyları sayarak adayların aldığı oy sayılarını hesaplar.
    *   **`GetCandidacyIndex(DWORD pid)`:** Verilen aday PID'sinin `m_vec_MonarchCandidacy` vektöründeki indeksini bulur.
*   **Önem:** İmparatorluklar arası dengeyi ve oyuncu etkileşimini sağlayan krallık sisteminin temelini oluşturur. Kralın belirlenmesi, krallık hazinesinin yönetimi ve seçim süreçlerini yönetir.
*   **Bağımlılıklar:** `Monarch.h`, `common/utils.h`, `Main.h`, `ClientManager.h`, `DBManager.h`.

### `MoneyLog.h` / `MoneyLog.cpp` (`CMoneyLog` Sınıfı)

*   **Amaç:** Oyundaki önemli para akışlarını (örn. NPC'den alış/satış, oyuncu ticareti) loglamak için kullanılır. Singleton olarak tasarlanmıştır.
*   **Veri Yapıları ve Üyeler:**
    *   `m_MoneyLogContainer` (private `MoneyLogMap[MONEY_LOG_TYPE_MAX_NUM]`): Log türüne (`MONEY_LOG_TYPE_*` enum, muhtemelen `common` içinde tanımlı) göre ayrılmış, `Vnum` (genellikle eşya Vnum'u) ve toplam `Gold` miktarını tutan map'lerden oluşan bir dizi. Aynı türdeki ve Vnum'daki logları birleştirir.
*   **Önemli Fonksiyonlar:**
    *   **`AddLog(BYTE bType, DWORD dwVnum, int iGold)`:** Belirtilen tür (`bType`) ve Vnum (`dwVnum`) için belirtilen altın miktarını (`iGold`) ilgili map'e ekler veya mevcut değeri günceller.
    *   **`Save()`:** `CClientManager::MainLoop` içinde periyodik olarak çağrılır. `m_MoneyLogContainer` içindeki birikmiş tüm logları `HEADER_DG_MONEY_LOG` paketi olarak rastgele bir oyun sunucusuna (`GetAnyPeer`) gönderir. Gönderdikten sonra log konteynerını temizler. Oyun sunucusu bu paketleri alıp `money_log` tablosuna kaydeder.
*   **Önem:** Oyun ekonomisini takip etmek, hileleri tespit etmek veya dengeyi analiz etmek için önemli para hareketlerinin kaydını tutar.
*   **Bağımlılıklar:** `stdafx.h`, `MoneyLog.h`, `ClientManager.h`, `Peer.h`, `common/tables.h` (dolaylı, `TPacketMoneyLog` ve `MONEY_LOG_TYPE_*` için).

### `NetBase.h` / `NetBase.cpp` (`CNetBase`, `CNetPoller` Sınıfları)

*   **`CNetBase`:**
    *   **Amaç:** Ağ ile ilgili sınıflar için çok temel bir üst sınıf görevi görür. Esas olarak statik bir `LPFDWATCH m_fdWatcher` üyesini tanımlar.
    *   `m_fdWatcher`: `libthecore` kütüphanesinden gelen, dosya tanımlayıcılarını (file descriptors - genellikle ağ soketleri) izleyen ve olayları (okuma/yazma hazır) bildiren mekanizmanın (fdwatch) bir işaretçisi. Tüm ağ sınıfları (örn. `CClientManager`, `CPeer`) aynı `fdwatch` nesnesini paylaşır.
*   **`CNetPoller`:**
    *   **Amaç:** `CNetBase`'den ve `singleton<CNetPoller>`'dan türeyen bu sınıf, paylaşılan `fdwatch` nesnesini oluşturmaktan (`Create`) ve yok etmekten (`Destroy`) sorumludur. Singleton yapısı, `fdwatch`'ın program boyunca sadece bir kez oluşturulmasını ve yönetilmesini sağlar.
    *   `Create()`: `m_fdWatcher` null ise `fdwatch_new()` ile yeni bir izleyici oluşturur.
    *   `Destroy()`: `fdwatch_delete()` ile izleyiciyi siler ve `thecore_destroy()` ile `libthecore` kütüphanesini temizler.
*   **Önem:** DB sunucusundaki tüm ağ operasyonları için temel olan olay izleme mekanizmasının (`fdwatch`) merkezi olarak başlatılmasını ve yönetilmesini sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `NetBase.h`, `Config.h`, `ClientManager.h` (dolaylı), `libthecore/stdafx.h` (fdwatch fonksiyonları için).

### `Peer.h` / `Peer.cpp` (`CPeer` Sınıfı)

*   **Amaç:** DB sunucusuna bağlanan her bir oyun sunucusunu (`game` core) temsil eden sınıftır. `CPeerBase`'den türetilmiştir (muhtemelen `libthecore` veya benzeri bir kütüphaneden temel ağ bağlantı işlevlerini alır). Oyun sunucusu ile DB sunucusu arasındaki ağ iletişimini (paket gönderme/alma, kodlama/çözme), bağlantı durumunu (`OnAccept`, `OnClose`) ve oyun sunucusuna özel bilgileri (kanal numarası, IP adresi, dinlediği portlar, sorumlu olduğu haritalar, atanmış eşya ID aralıkları) yönetir.
*   **Veri Yapıları ve Üyeler:**
    *   `HEADER` (struct, pragma pack(1)): Ağ paketlerinin başlığını temsil eder (`bHeader`: paket türü, `dwHandle`: bağlantı handle'ı, `dwSize`: veri boyutu).
    *   `EState` (enum): Bağlantı durumunu (`STATE_CLOSE`, `STATE_PLAYING`) belirtir.
    *   `m_state` (private `int`): Mevcut bağlantı durumu.
    *   `m_bChannel` (private `BYTE`): Bu oyun sunucusunun ait olduğu kanal numarası.
    *   `m_dwHandle` (private `DWORD`): DB sunucusu tarafından atanan benzersiz bağlantı kimliği.
    *   `m_dwUserCount` (private `DWORD`): Bu oyun sunucusundaki online oyuncu sayısı.
    *   `m_wListenPort` (private `WORD`): Oyun sunucusunun oyuncu bağlantılarını dinlediği port.
    *   `m_wP2PPort` (private `WORD`): Oyun sunucusunun P2P bağlantıları için kullandığı port.
    *   `m_alMaps` (private `long[MAX_MAP_ALLOW]`): Bu oyun sunucusunun sorumlu olduğu haritaların listesi.
    *   `m_itemRange` / `m_itemSpareRange` (private `TItemIDRangeTable`): Bu oyun sunucusuna atanmış ana ve yedek eşya ID aralıkları.
    *   `m_stPublicIP` (private `std::string`): Oyun sunucusunun genel IP adresi.
*   **Önemli Fonksiyonlar:**
    *   **`OnAccept()`/`OnConnect()`:** Yeni bir oyun sunucusu bağlantısı kabul edildiğinde/kurulduğunda çağrılır. Benzersiz bir `handle` atar (`m_dwHandle`) ve durumu `STATE_PLAYING` yapar.
    *   **`OnClose()`:** Bağlantı kapandığında çağrılır. Durumu `STATE_CLOSE` yapar ve bu sunucuya atanmış olan eşya ID aralığını (`m_itemRange`) `CItemIDRangeManager`'a geri verir (`UpdateRange`).
    *   **`PeekPacket(int& iBytesProceed, BYTE& header, DWORD& dwHandle, DWORD& dwLength, const char** data)`:** Gelen veri tamponundan (`GetRecvBuffer`) bir sonraki tam paketi (başlık, handle, boyut, veri) okumaya çalışır. Tam bir paket varsa `true` döner ve parametreleri doldurur.
    *   **`EncodeHeader(BYTE header, DWORD dwHandle, DWORD dwSize)` / `EncodeReturn(BYTE header, DWORD dwHandle)` / `Encode(const void* data, size_t size)`:** Oyun sunucusuna gönderilecek paketleri oluşturur ve gönderme tamponuna (`CPeerBase` içindeki) yazar. `EncodeHeader` başlık bilgilerini, `Encode` asıl veriyi yazar. `EncodeReturn` sadece başlık ve handle içeren yanıt paketi gönderir.
    *   **`Send()`:** Gönderme tamponundaki verileri ağ üzerinden gönderir (`CPeerBase::Send`).
    *   **`GetHandle()`:** Bağlantı handle'ını döndürür.
    *   **`Set/GetUserCount()`:** Oyun sunucusundaki oyuncu sayısını ayarlar/alır.
    *   **`Set/GetPublicIP()`:** Oyun sunucusunun IP adresini ayarlar/alır.
    *   **`Set/GetChannel()`:** Oyun sunucusunun kanalını ayarlar/alır.
    *   **`Set/GetListenPort()`:** Oyun sunucusunun dinleme portunu ayarlar/alır.
    *   **`Set/GetP2PPort()`:** Oyun sunucusunun P2P portunu ayarlar/alır.
    *   **`SetMaps(long* pl)` / `GetMaps()`:** Oyun sunucusunun sorumlu olduğu harita listesini ayarlar/alır.
    *   **`SetItemIDRange(TItemIDRangeTable itemRange)` / `SetSpareItemIDRange(TItemIDRangeTable itemRange)`:** Bu oyun sunucusuna atanacak ana ve yedek eşya ID aralıklarını ayarlar.
    *   **`CheckItemIDRangeCollision(TItemIDRangeTable itemRange)`:** Verilen ID aralığının, bu sunucunun mevcut ana ve yedek aralıklarıyla çakışıp çakışmadığını kontrol eder.
    *   **`SendSpareItemIDRange()`:** `CItemIDRangeManager`'dan yeni bir yedek ID aralığı alır, bunu mevcut yedek aralığı olarak ayarlar ve eski yedek aralığını (şimdi ana aralık oldu) oyun sunucusuna `HEADER_DG_ACK_SPARE_ITEM_ID_RANGE` paketiyle gönderir.
    *   **`GetMap(DWORD dwMapIndex)` (`#if defined(__PREMIUM_PRIVATE_SHOP__)`)**: Verilen harita indeksinin bu sunucunun sorumlu olduğu haritalar arasında olup olmadığını kontrol eder.
*   **Önem:** DB sunucusunun birden fazla oyun sunucusuyla aynı anda iletişim kurmasını ve her birini ayrı ayrı yönetmesini sağlar. Paket alışverişi ve oyun sunucularına özel durumların takibi için temel yapıdır. `CClientManager`, bağlı tüm `CPeer` nesnelerinin bir listesini tutar.
*   **Bağımlılıklar:** `stdafx.h`, `Peer.h`, `PeerBase.h` (muhtemelen `libthecore`), `ItemIDRangeManager.h`, `common/tables.h`. 


### `PeerBase.h` / `PeerBase.cpp` (`CPeerBase` Sınıfı)

*   **Amaç:** `CPeer` sınıfı için temel ağ işlevlerini sağlayan soyut bir üst sınıftır (`CNetBase`\'den türemiştir). TCP soket bağlantılarını kabul etme (`Accept`), kurma (`Connect`), kapatma (`Close`, `Disconnect`) ve veri gönderme/alma (`Send`, `Recv`, `Encode*`) için temel mekanizmaları içerir. Gelen ve giden veriler için tamponları (`LPBUFFER m_inBuffer`, `m_outBuffer` - muhtemelen `libthecore`\'dan) yönetir ve ağ olaylarını izlemek için `fdwatch` kullanır.
*   **Soyut Metodlar:** `OnAccept()`, `OnConnect()`, `OnClose()` sanal (virtual) metodlardır. Türeyen sınıfın (`CPeer`), bu olaylar gerçekleştiğinde kendi özel mantığını (`override` ederek) uygulaması gerekir.
*   **Veri Yapıları ve Üyeler:**
    *   `m_host` (protected `char[MAX_HOST_LENGTH + 1]`): Bağlantının host adı veya IP adresi.
    *   `m_fd` (protected `socket_t`): Bağlantının soket tanımlayıcısı.
    *   `m_BytesRemain` (private `int`): Gelen tamponda kalan işlenmemiş bayt sayısı.
    *   `m_outBuffer` (private `LPBUFFER`): Giden veri tamponu.
    *   `m_inBuffer` (private `LPBUFFER`): Gelen veri tamponu.
*   **Önemli Fonksiyonlar:**
    *   **`Accept(socket_t fd_accept)`:** Verilen dinleme soketinden yeni bir bağlantıyı kabul eder. Başarılı olursa:
        *   Yeni bir soket (`m_fd`) oluşturur.
        *   Soket ayarlarını yapar (buffer boyutları).
        *   Gelen ve giden tamponları (`m_inBuffer`, `m_outBuffer`) oluşturur (`buffer_new`).
        *   Soketi okuma olayları için `fdwatch`\'a ekler (`fdwatch_add_fd`).
        *   `OnAccept()` sanal metodunu çağırır.
        *   Bağlantı güvenliği kontrolü içerir (`#ifdef __PORT_SECURITY__`).
    *   **`Connect(const char* host, WORD port)`:** Belirtilen host ve porta bağlanmayı dener. Başarılı olursa:
        *   Soket (`m_fd`) oluşturur (`socket_connect`).
        *   Giden tamponu (`m_outBuffer`) oluşturur.
        *   Soketi okuma olayları için `fdwatch`\'a ekler.
        *   `OnConnect()` sanal metodunu çağırır.
    *   **`Close()`:** `OnClose()` sanal metodunu çağırır (bağlantı kapanmadan önce türeyen sınıfın temizlik yapabilmesi için).
    *   **`Disconnect()`:** Soketi `fdwatch`\'tan kaldırır (`fdwatch_del_fd`) ve soketi kapatır (`socket_close`). `m_fd`\'yi `INVALID_SOCKET` yapar.
    *   **`Destroy()`:** `Disconnect()`\'i çağırır ve tamponları siler (`buffer_delete`).
    *   **`EncodeBYTE(BYTE b)`, `EncodeWORD(WORD w)`, `EncodeDWORD(DWORD dw)`, `Encode(const void* data, DWORD size)`:** Verilen veriyi giden tampona (`m_outBuffer`) yazar (`buffer_write`) ve soketin yazma için hazır olduğunu `fdwatch`\'a bildirir (`fdwatch_add_fd` ile FDW_WRITE bayrağını ayarlar).
    *   **`Recv()`:** Soketten (`m_fd`) veri okur (`socket_read`) ve gelen tampona (`m_inBuffer`) yazar (`buffer_write_proceed`). Okunan bayt sayısını günceller (`m_BytesRemain`). Başarılı olursa 1, bağlantı kapandıysa 0, hata olursa -1 döner.
    *   **`RecvEnd(int proceed_bytes)`:** Gelen tampondaki işlenen `proceed_bytes` kadar veriyi temizler (`buffer_read_proceed`).
    *   **`GetRecvLength()`:** Gelen tamponda işlenmeyi bekleyen veri boyutunu (`m_BytesRemain`) döndürür.
    *   **`GetRecvBuffer()`:** Gelen tamponun okunabilir kısmının başlangıç adresini (`buffer_read_peek`) döndürür.
    *   **`Send()`:** Giden tampondaki (`m_outBuffer`) veriyi sokete yazmaya çalışır (`socket_write`). Tampon boş değilse ve yazma işlemi tamamlanmadıysa soketin yazma için hazır olduğunu `fdwatch`\'a bildirir.
    *   **`GetFd()`:** Soket tanımlayıcısını döndürür.
    *   **`GetHost()`:** Bağlantının host adını döndürür.
    *   **`GetSendLength()`:** Giden tamponda gönderilmeyi bekleyen veri boyutunu (`buffer_size`) döndürür.
*   **Önem:** `CPeer` gibi ağ bağlantılarını yöneten sınıflar için yeniden kullanılabilir, temel bir ağ katmanı sağlar. Bağlantı yönetimi, tamponlama ve olay tabanlı G/Ç (`fdwatch`) işlemlerini soyutlar.
*   **Bağımlılıklar:** `stdafx.h`, `PeerBase.h`, `NetBase.h`, `libthecore` (soket, tampon, fdwatch fonksiyonları).


### `PrivateShopUtils.h`

*   **Amaç:** Bu başlık dosyası, `#if defined(__PREMIUM_PRIVATE_SHOP__)` koşuluyla derlenen premium/çevrimdışı özel pazar sistemi için bir dizi `inline` yardımcı fonksiyon içerir. Bu fonksiyonlar, özel pazar ve eşyalarıyla ilgili veritabanı sorgularını oluşturmak ve sorgu sonuçlarını C++ yapılarına dönüştürmek için kullanılır. `.cpp` dosyası yoktur, fonksiyonlar doğrudan çağrıldıkları yerde derlenir.
*   **Fonksiyonlar:**
    *   **`GetPrivateShopQuery(DWORD dwOwner)`:** Verilen sahip ID'sine göre `private_shop` tablosundan pazarın ana bilgilerini çekecek `SELECT` sorgu metnini oluşturur ve döndürür. `GetTablePostfix()` fonksiyonunu kullanarak tablo ismine (varsa) yerel son eki ekler.
    *   **`GetPrivateShopItemQuery(DWORD dwOwner)`:** Verilen sahip ID'sine göre `private_shop_item` tablosundan pazardaki tüm eşyaların detaylı bilgilerini (Vnum, adet, fiyat, soketler, efsunlar, görünüm, element vb. - `#ifdef` ile kontrol edilen sistemlere bağlı olarak) çekecek `SELECT` sorgu metnini oluşturur ve döndürür.
    *   **`CreatePrivateShopTableFromRes(MYSQL_RES* pRes, TPrivateShop& rTable)`:** Bir MySQL sorgu sonucunu (`MYSQL_RES*`) alır, tek bir satır (`mysql_fetch_row`) bekler ve bu satırdaki sütunları ayrıştırarak (`str_to_number`, `strlcpy`) sonuçları verilen `TPrivateShop` yapısı referansına doldurur. Sonuç kümesi boşsa veya hata varsa `false` döner.
    *   **`CreatePrivateShopItemTableFromRes(MYSQL_RES* pRes, std::vector<TPlayerPrivateShopItem>* pVec, DWORD dwPID)`:** Bir MySQL sorgu sonucunu alır, tüm satırları döngü ile işler (`mysql_fetch_row`), her satırdaki verileri `TPlayerPrivateShopItem` yapısına doldurur ve bu yapıları verilen `std::vector` işaretçisine (`pVec`) ekler. Eşyaların sahibini (`dwPID`) de ayarlar. Sonuç kümesi boşsa veya hata varsa `false` döner.
    *   **`CopyItemData(const TPlayerPrivateShopItem& rSourceTable, TPlayerItem rTargetTable)`:** Özel pazar eşya yapısındaki (`TPlayerPrivateShopItem`) verileri, standart oyuncu eşya yapısına (`TPlayerItem`) kopyalamak için kullanılır. Muhtemelen bir eşya pazardan çekildiğinde veya satın alındığında, envantere eklenmeden önce bu fonksiyon kullanılır.
*   **Önem:** Özel pazar sisteminin veritabanı işlemleriyle ilgili tekrarlayan kodları (sorgu oluşturma, sonuç ayrıştırma) merkezi ve `inline` fonksiyonlarda toplayarak kod tekrarını azaltır ve okunabilirliği artırır. Farklı sistemlerin (`#ifdef` ile kontrol edilen) varlığına göre sorgu ve veri yapılarının dinamik olarak uyarlanmasını sağlar.
*   **Bağımlılıklar:** `stdafx.h`, `ClientManager.h`, `Main.h` (özellikle `GetTablePostfix` için), `common/tables.h` (dolaylı olarak `TPrivateShop` ve `TPlayerPrivateShopItem` yapıları için).


### `PrivManager.h` / `PrivManager.cpp` (`CPrivManager` Sınıfı)

*   **Amaç:** Oyun içinde farklı kapsamlarda (imparatorluk, lonca, karakter) uygulanan özel bonusları veya cezaları ("privilege" - ayrıcalık) yönetir. Örneğin, bir imparatorluğa eşya düşürme oranı bonusu vermek, bir loncaya EXP bonusu vermek veya bir karaktere geçici bir negatif etki uygulamak gibi. Singleton olarak tasarlanmıştır.
*   **Veri Yapıları:**
    *   `TPrivEmpireData`, `TPrivGuildData`, `TPrivCharData`: Sırasıyla imparatorluk, lonca ve karaktere özgü ayrıcalık bilgilerini (tür (`BYTE type`), değer (`int value`), bitiş zamanı (`time_t end_time_sec` - sadece lonca ve imparatorluk için), ilgili ID (empire, guild_id, pid)) ve kaldırılma durumunu (`bool bRemoved`) tutan yapılar.
    *   `m_aaPrivEmpire[MAX_PRIV_NUM][EMPIRE_MAX_NUM]` (private `TPrivEmpireData*`): İmparatorluk ayrıcalıklarını tür ve imparatorluk ID'sine göre hızlı erişim için tutan 2 boyutlu dizi. Pointer değeri aktif ayrıcalığı gösterir.
    *   `m_aPrivGuild[MAX_PRIV_NUM]` (private `PrivGuildDataMap` -> `map<DWORD, TPrivGuildData*>` dizisi): Lonca ayrıcalıklarını türlerine göre ayıran ve lonca ID'sinden ayrıcalık verisine (`TPrivGuildData*`) eşleme yapan map dizisi.
    *   `m_aPrivChar[MAX_PRIV_NUM]` (private `PrivCharDataMap` -> `map<DWORD, TPrivCharData*>` dizisi): Karakter ayrıcalıklarını türlerine göre ayıran ve karakter PID'sinden ayrıcalık verisine (`TPrivCharData*`) eşleme yapan map dizisi.
    *   `m_pqPrivEmpire`, `m_pqPrivGuild`, `m_pqPrivChar` (private `std::priority_queue`): Ayrıcalıkların bitiş zamanlarına (`time_t`) göre sıralandığı öncelik kuyrukları. En erken bitecek olan en üstte bulunur. Zamanı dolan ayrıcalıkların kaldırılması için kullanılır. Kuyruklar `pair<time_t, TPriv*Data*>` tutar.
*   **Önemli Fonksiyonlar:**
    *   **`AddEmpirePriv(BYTE empire, BYTE type, int value, time_t duration_sec)`:**
    *   **`AddGuildPriv(DWORD guild_id, BYTE type, int value, time_t duration_sec)`:**
    *   **`AddCharPriv(DWORD pid, BYTE type, int value)`:**
        *   Belirtilen kapsama (imparatorluk, lonca, karakter), belirtilen türde, değerde ve sürede (lonca ve imparatorluk için `duration_sec`, karakter için sabit süreler) yeni bir ayrıcalık ekler.
        *   Mevcut bir ayrıcalık varsa, eski ayrıcalığın `bRemoved` bayrağını `true` yapar (lonca ve imparatorluk için) veya işlemi doğrudan reddeder (karakter için).
        *   Yeni `TPriv*Data` nesnesi oluşturur.
        *   Nesneyi ilgili map/diziye ekler/günceller.
        *   Bitiş zamanını hesaplar ve `pair<time_t, TPriv*Data*>` olarak ilgili öncelik kuyruğuna (`m_pq*`) ekler.
        *   Değişikliği tüm oyun sunucularına ilgili `SendChange*Priv` fonksiyonu ile bildirir.
    *   **`Update()`:**
        *   Ana döngüde periyodik olarak çağrılır (`CClientManager::MainLoop`).
        *   Üç öncelik kuyruğunu (`m_pq*`) kontrol eder.
        *   Zamanı dolmuş (`top().first <= now`) ayrıcalıkları kuyruktan çıkarır.
        *   Eğer çıkarılan ayrıcalık kaldırılmamışsa (`!p->bRemoved`), ilgili map/diziden de kaldırır (`erase`) ve ayrıcalığın kaldırıldığını (`value=0`, `end_time_sec=0`) tüm oyun sunucularına `SendChange*Priv` ile bildirir.
        *   Zamanı dolan `TPriv*Data` nesnesini siler (`delete p`).
    *   **`SendPrivOnSetup(CPeer* peer)`:**
        *   Yeni bir oyun sunucusu bağlandığında (`CClientManager` içinde) çağrılır.
        *   Mevcut tüm aktif ayrıcalık bilgilerini (map'lerde ve dizide tutulan `TPriv*Data` nesneleri) bu yeni sunucuya `SendChange*Priv` fonksiyonları aracılığıyla gönderir.
    *   **`SendChangeEmpirePriv(BYTE empire, BYTE type, int value, time_t end_time_sec)`:**
    *   **`SendChangeGuildPriv(DWORD guild_id, BYTE type, int value, time_t end_time_sec)`:**
    *   **`SendChangeCharPriv(DWORD pid, BYTE type, int value)`:**
        *   Belirli bir ayrıcalıktaki değişikliği (ekleme, güncelleme, kaldırma) ilgili `TPacketDGChange*Priv` yapısına doldurur.
        *   `CClientManager::instance().ForEachPeer` ile tüm bağlı oyun sunucularını (`CPeer*`) dolaşır.
        *   Her oyun sunucusu için paketi `EncodeHeader` ve `Encode` ile gönderir. Paket göndermek için functor yapıları (`FSendChange*Priv`) kullanılır.
*   **Önem:** Oyun dinamiklerini geçici olarak etkileyen çeşitli bonus ve cezaların merkezi olarak yönetilmesini ve tüm oyun sunucuları arasında senkronize edilmesini sağlar. Süreye bağlı etkilerin otomatik olarak kaldırılmasını yönetir.
*   **Bağımlılıklar:** `stdafx.h`, `PrivManager.h`, `Peer.h`, `ClientManager.h`, `common/singleton.h`, `common/tables.h`.


### `ProtoReader.h` / `ProtoReader.cpp`

*   **Amaç:** Genellikle CSV (veya benzeri metin formatındaki) dosyalarda tanımlanan oyun prototip verilerini (mob_proto, item_proto gibi) okumak, ayrıştırmak ve C++ yapılarına (`TMobTable`, `TItemTable` vb.) dönüştürmek için yardımcı fonksiyonlar içerir. Bu işlem genellikle sunucu başlangıcında `ClientManagerBoot.cpp` içindeki `Initialize*Table` fonksiyonları tarafından kullanılır.
*   **Fonksiyonlar:**
    *   **`get_Item_*_Value(std::string inputString)` / `get_Mob_*_Value(std::string inputString)`:**
        *   Prototip dosyalarındaki metin tabanlı değerleri (örn: "ITEM_WEAPON", "ANTI_MUSA", "FLAG_SLOW", "RANK_BOSS", "RACE_FLAG_ANIMAL") karşılık gelen sayısal enum veya bitmask değerlerine dönüştürürler.
        *   `item_proto` ve `mob_proto` tablolarındaki çeşitli özellik sütunlarının (Type, SubType, AntiFlag, Flag, WearFlag, ImmuneFlag, LimitType, ApplyType, Rank, MobType, BattleType, Size, AIFlag, RaceFlag, ImmuneFlag) doğru şekilde doldurulmasını sağlarlar.
        *   Genellikle önceden tanımlanmış `string` dizileri üzerinde `find` veya doğrudan karşılaştırma yaparak çalışırlar.
    *   **`Set_Proto_Mob_Table(TMobTable* mobTable, cCsvTable& csvTable, std::map<int, const char*>& nameMap)`:**
        *   Bir `cCsvTable` (muhtemelen `CsvReader.h`'den) nesnesinden okunan tek bir mob verisi satırını alır.
        *   `get_Mob_*_Value()` fonksiyonlarını kullanarak metin değerlerini sayısala çevirir.
        *   Sonuçları verilen `TMobTable` yapısının ilgili alanlarına (`szName`, `szLocaleName` (isim haritasından), `bRank`, `bType`, `dwAIFlag`, `dwRaceFlag`, `dwImmuneFlag`, `dwLevel`, `dwExp`, `dwMaxHP`, `wDef`, `fDamMultiply`, `dwDropItemVnum`, `dwResistances`, `cEnchants` vb.) doldurur.
        *   Sayısal alanları `csvTable.AsString()` ile alıp `str_to_number` veya `stoi`/`stoul`/`stoull` gibi fonksiyonlarla dönüştürür.
    *   **`Set_Proto_Item_Table(TItemTable* itemTable, cCsvTable& csvTable, std::map<int, const char*>& nameMap)`:**
        *   Bir `cCsvTable` nesnesinden okunan tek bir eşya verisi satırını alır.
        *   `get_Item_*_Value()` fonksiyonlarını kullanarak metin değerlerini sayısala çevirir.
        *   Sonuçları verilen `TItemTable` yapısının ilgili alanlarına (`szName`, `szLocaleName` (isim haritasından), `bType`, `bSubType`, `dwAntiFlags`, `dwFlags`, `dwWearFlags`, `dwImmuneFlag`, `dwRefinedVnum`, `wLimitCount`, `aLimits` (limit türü/değeri), `aApplies` (efsun türü/değeri), `alValues` (eşya değerleri), `alSockets` (soket değerleri), `iGold`, `iShopBuyPrice` vb.) doldurur.
        *   Sayısal alanları `csvTable.AsString()` ile alıp `str_to_number` veya benzeri fonksiyonlarla dönüştürür.
    *   **`StringSplit(string strOrigin, string strTok)` / `trim(const string& str)`:** String işleme için yardımcı fonksiyonlar (boşlukları temizleme, string'i belirtilen ayırıcıya göre bölme).
*   **Önem:** Oyunun temel verilerini (eşyalar, moblar) tanımlayan prototip dosyalarının okunup standart C++ yapılarına (`T*Table`) dönüştürülmesini sağlayarak sunucunun bu verilere tutarlı ve verimli bir şekilde erişimini kolaylaştırır. Metin tabanlı konfigürasyonun kullanılmasını mümkün kılar.
*   **Bağımlılıklar:** `stdafx.h`, `ProtoReader.h`, `CsvReader.h`, `common/tables.h`, `<sstream>`, `<cmath>`, `<map>`.


### `QID.h`

*   **Amaç:** DB sunucusu tarafından veritabanına gönderilen eşzamansız sorguları (`CDBManager::ReturnQuery` ile gönderilenler) tanımlamak için kullanılan benzersiz kimlikleri (Query ID - QID) bir `enum` içinde barındırır. Eşzamansız bir sorgu tamamlandığında ve sonucu `CDBManager::PopResult` ile alındığında, bu QID (`SQLMsg->Get()->uiQID`) sorgunun türünü belirtir. `CClientManager::AnalyzeQueryResult`, bu QID'yi kullanarak gelen sonucu hangi `RESULT_*` fonksiyonunun işlemesi gerektiğini belirler.
*   **İçerik:**
    *   `enum QID { ... };`: Her bir enum sabiti (`QID_PLAYER`, `QID_ITEM`, `QID_QUEST`, `QID_AFFECT`, `QID_LOGIN`, `QID_SAFEBOX_LOAD`, `QID_ITEM_SAVE`, `QID_PLAYER_DELETE`, `QID_LOGIN_BY_KEY`, `QID_ITEM_AWARD_LOAD`, `QID_GUILD_RANKING`, `QID_ITEMPRICE_LOAD`, `QID_PRIVATE_SHOP`, `QID_PRIVATE_SHOP_ITEM_DELETE` vb.) farklı bir veritabanı sorgu operasyonuna karşılık gelir.
    *   İsimlendirme genellikle sorgunun amacını açıklar (oyuncu yükleme, eşya yükleme, görev kaydetme, depo yükleme, oyuncu silme, eşya ödülü alma, özel pazar eşyası kaydetme vb.).
    *   Bazı QID'ler `#ifdef` direktifleri içine alınmış olabilir (örn. `#if defined(__PREMIUM_PRIVATE_SHOP__)`), bu da ilgili sistemin aktif olup olmamasına göre derlenip derlenmeyeceğini gösterir.
*   **Önem:** Eşzamansız veritabanı sorgularının sonuçlarını güvenilir bir şekilde tanımlamak ve doğru işleyici koda yönlendirmek için temel bir mekanizmadır. Kodun okunabilirliğini ve sorgu sonuçlarının yönetimini kolaylaştırır.
*   **Bağımlılıklar:** Yok (sadece temel C++ enum tanımı içerir).


### `stdafx.h`

*   **Amaç:** "Standard Application Framework Extensions" anlamına gelir. Bu dosya, `db` projesi genelinde sıkça kullanılan ve nadiren değişen başlık dosyalarını (standart kütüphaneler, projenin diğer modüllerinden gelen temel başlıklar - `libthecore`, `common`) tek bir yerde toplar. Genellikle derleyici tarafından ön derlenmiş başlık dosyası (PCH - Precompiled Header) oluşturmak için kullanılır. PCH mekanizması, bu başlıkların her `.cpp` dosyasında tekrar tekrar işlenmesini engelleyerek derleme süresini önemli ölçüde azaltır.
*   **İçerik:**
    *   `#include "../../libthecore/include/stdafx.h"`: `libthecore` kütüphanesinin temel başlık dosyasını dahil eder. Bu, ağ soketleri, olay döngüsü (`fdwatch`), tamponlama (`buffer`) gibi düşük seviyeli çekirdek işlevlerini sağlar.
    *   **Platforma Özgü Ayarlar:** `#ifndef __WIN32__` bloğu ile Windows ve diğer (genellikle Linux/FreeBSD) sistemler için farklı ayarlamalar yapar. Örneğin, Windows dışı için `<semaphore.h>` dahil edilirken, Windows için `isdigit` ve `isspace` makroları geniş karakter versiyonlarına (`iswdigit`, `iswspace`) yönlendirilir.
    *   **`common` Kütüphanesi Başlıkları:**
        *   `length.h`: Oyun içindeki çeşitli maksimum uzunlukları (oyuncu adı, lonca adı, şifre vb.) tanımlayan sabitleri içerir.
        *   `tables.h`: Veritabanı tablolarının C++ yapıları (`TPlayerTable`, `TItemTable`, `TGuildMember`) ve ağ paketlerinin yapıları (`TPacketCGLogin`, `TPacketGDSetup`, `TPacketDGLoginSuccess`) gibi oyunun temel veri yapılarını tanımlar.
        *   `singleton.h`: Singleton tasarım desenini uygulamak için kullanılan `singleton` şablon sınıfını sağlar.
        *   `utils.h`: String manipülasyonu, zaman/tarih işlemleri, rastgele sayı üretimi gibi genel amaçlı yardımcı fonksiyonları ve makroları içerir.
        *   `stl.h`: Sık kullanılan C++ Standart Şablon Kütüphanesi (STL) başlıklarını (`<vector>`, `<map>`, `<string>`, `<algorithm>` vb.) topluca dahil eder.
        *   `service.h`: Genellikle global servis konfigürasyonları veya tanımlamaları (örn. `g_stLocaleNameColumn` gibi global değişkenler veya servis ile ilgili sabitler) içerir.
*   **Önem:** Projenin temel yapı taşlarını ve ortak bağımlılıklarını sağlar. Derleme süreçlerini hızlandırır ve kodun daha düzenli olmasını sağlar. `db/src` içindeki hemen hemen tüm `.cpp` dosyaları, ilk olarak bu dosyayı `#include` eder.
*   **`stdafx.cpp`:** Bu dosya genellikle sadece `#include "stdafx.h"` satırını içerir ve derleyicinin PCH dosyasını (`.pch`) oluşturması için kullanılır. Bu projede `stdafx.cpp` dosyası bulunmamaktadır, bu da PCH kullanımının farklı bir şekilde yapılandırılmış olabileceğini veya hiç kullanılmıyor olabileceğini gösterir (nadiren de olsa).
*   **Bağımlılıklar:** `libthecore`, `common` modülleri.


### `version.cpp`

*   **Amaç:** DB sunucusu başlatıldığında (sadece Windows dışı sistemlerde, `#ifndef __WIN32__` bloğu nedeniyle) `version.txt` adında bir dosya oluşturarak veya üzerine yazarak projenin sürüm bilgilerini kaydeder. Bu, çalışan sunucu örneğinin hangi versiyona ait olduğunu hızlıca belirlemeye yarar.
*   **Fonksiyonlar:**
    *   **`WriteVersion()`:**
        *   `version.txt` dosyasını yazma amacıyla açar (`fopen`).
        *   Dosya başarıyla açılırsa, içine sabit bir metin ("M2_V6 Project") ve derleme sırasında tanımlanan `__VERSION__` makrosundaki değeri içeren bir satır ("METIN2 Official Lieke Project: [sürüm]") yazar (`fprintf`).
        *   Dosyayı kapatır (`fclose`).
        *   Eğer dosya açılamazsa, standart hata akışına (`stderr`) bir hata mesajı yazdırır ve programı sonlandırır (`exit(0)`).
*   **Önem:** Sunucu yönetimi ve hata ayıklama süreçlerinde, hangi kod versiyonunun çalıştığını doğrulamak için basit ve etkili bir yöntem sunar.
*   **Bağımlılıklar:** `<stdio.h>`, `<stdlib.h>`.


</rewritten_file> 