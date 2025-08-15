# EterBase Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/EterBase/` klasöründe bulunan `EterBase` modülünün amacını, içerdiği temel bileşenleri ve bu bileşenlerin işlevlerini açıklamaktadır.

`EterBase`, Metin2 istemcisinin diğer "Eter" önekli kütüphaneleri (EterLib, EterGrnLib vb.) ve genel sistemler için en temel düzeyde yardımcı sınıfları, fonksiyonları ve veri yapılarını içeren bir çekirdek kütüphanedir. Bu modül, istemcinin daha üst seviye bileşenleri tarafından yaygın olarak kullanılan düşük seviyeli işlevsellikler sağlar.

## Genel Amaç ve İçerik

`EterBase` modülü, aşağıdaki gibi çeşitli temel görevler için araçlar sunar:

*   **Yardımcı Fonksiyonlar ve Makrolar:** String işleme, dosya yolu manipülasyonu, güvenli bellek yönetimi (SAFE_DELETE, SAFE_RELEASE vb.), bit işlemleri, matematiksel yardımcılar ve genel amaçlı şablon fonksiyonları (`Utils.h`, `Stl.h`).
*   **Temel Veri Yapıları ve Desenler:** Singleton tasarım deseni (`Singleton.h`), zamanlayıcı mekanizmaları (`Timer.h`), rastgele sayı üretimi (`Random.h`).
*   **Dosya ve Dizin İşlemleri:** Dosya okuma/yazma, bellek eşlemli dosyalar, dosya ve dizin yönetimi, geçici dosya oluşturma (`FileBase.h`, `FileDir.h`, `FileLoader.h`, `Filename.h`, `MappedFile.h`, `TempFile.h`).
*   **Kriptografi ve Veri Bütünlüğü:** Basit şifreleme (TEA, özel cipher), karma algoritmaları (CRC32) ve veri gizleme/karıştırma mekanizmaları (`cipher.h`, `tea.h`, `CRC32.h`, `obfuscate.h`).
*   **Sıkıştırma:** LZO sıkıştırma algoritması için arayüz (`lzo.h`).
*   **Hata Ayıklama ve Raporlama:** Hata mesajları, loglama için temel araçlar ve özel hata durumları (`Debug.h`, `error.h`, `CPostIt.h`).
*   **Diğer Temel Bileşenler:** Standart başlıklar (`StdAfx.h`), sanal klavye kodları (`vk.h`) ve servis tanımlamaları (`ServiceDefs.h`) gibi çeşitli destekleyici elemanlar.

Bu modüldeki bileşenler, istemci kod tabanının genelinde yeniden kullanılabilirlik ve tutarlılık sağlamak amacıyla tasarlanmıştır.

---

## Dosya Bazlı Detaylandırma

### `Timer.h`

*   **Amaç:** Oyun istemcisi için temel zamanlama hizmetleri sunar. Zamanı takip etmek, geçen süreyi ölçmek ve özel zaman ayarlamaları yapmak için kullanılır.
*   **Temel Özellikler/İçerik:**
    *   **`CTimer` Sınıfı (Singleton):**
        *   `CSingleton<CTimer>` sınıfından türetilmiştir, bu da sistem genelinde tek bir `CTimer` örneği olmasını sağlar.
        *   `Advance()`: Zamanı ilerletir, mevcut zamanı günceller.
        *   `Adjust(int iTimeGap)`: Mevcut zamana bir zaman farkı (pozitif veya negatif) ekleyerek zamanı ayarlar.
        *   `SetBaseTime()`: Zamanlayıcının başlangıç (temel) zamanını ayarlar.
        *   `GetCurrentSecond()`: Mevcut zamanı saniye cinsinden (float) döndürür.
        *   `GetCurrentMillisecond()`: Mevcut zamanı milisaniye cinsinden (DWORD) döndürür.
        *   `GetElapsedSecond()`: Bir önceki `Advance()` çağrısından bu yana geçen süreyi saniye cinsinden (float) döndürür.
        *   `GetElapsedMilliecond()`: Bir önceki `Advance()` çağrısından bu yana geçen süreyi milisaniye cinsinden (DWORD) döndürür.
        *   `UseCustomTime()`: Zamanlayıcının gerçek sistem zamanı yerine özel bir zaman kaynağı kullanıp kullanmayacağını belirler (Bu fonksiyonun implementasyonu `m_bUseRealTime` değişkenini etkileyebilir, ancak detaylar `.cpp` dosyasındadır).
        *   **Korumalı Üyeler:**
            *   `m_bUseRealTime`: Gerçek zaman mı yoksa özel ayarlanmış zaman mı kullanılacağını belirten boolean bayrak.
            *   `m_dwBaseTime`: Zamanlayıcının başlangıç zamanı (milisaniye).
            *   `m_dwCurrentTime`: Mevcut zaman (milisaniye).
            *   `m_fCurrentTime`: Mevcut zaman (saniye, float).
            *   `m_dwElapsedTime`: Geçen süre (milisaniye).
    *   **Global Fonksiyonlar:**
        *   `ELTimer_Init()`: Zamanlayıcı sistemini başlatır (muhtemelen `CTimer` örneğini oluşturur veya temel ayarları yapar).
        *   `ELTimer_GetMSec()`: Geçerli zamanı milisaniye cinsinden döndürür (muhtemelen `CTimer::GetCurrentMillisecond()` için bir sarmalayıcı).
        *   `ELTimer_SetServerMSec(DWORD dwServerTime)`: Sunucudan gelen bir zaman damgasını ayarlamak için kullanılır. İstemci ve sunucu zamanını senkronize etmeye yardımcı olabilir.
        *   `ELTimer_GetServerMSec()`: Ayarlanan sunucu zamanını döndürür.
        *   `ELTimer_SetFrameMSec()`: Kare (frame) başına düşen zamanı ayarlar veya kaydeder.
        *   `ELTimer_GetFrameMSec()`: Ayarlanan kare zamanını döndürür. Bu, oyun döngüsü, animasyonlar veya fizik güncellemeleri gibi kare bazlı işlemler için önemlidir.
*   **Kullanım Alanı:** Oyun döngüsü yönetimi, animasyon zamanlamaları, olayların belirli zaman aralıklarında tetiklenmesi, performans ölçümleri ve genel zaman takibi gerektiren her türlü işlemde kullanılır.

### `Utils.h`

*   **Amaç:** Proje genelinde sıkça ihtiyaç duyulan çeşitli yardımcı fonksiyonları, makroları ve şablonları barındırır. Kod tekrarını azaltmak, güvenliği artırmak ve yaygın görevleri basitleştirmek için tasarlanmıştır.
*   **Temel Özellikler/İçerik:**
    *   **Güvenli Kaynak Yönetimi Makroları:**
        *   `SAFE_DELETE(p)`: Bir işaretçiyi siler ve NULL olarak ayarlar.
        *   `SAFE_DELETE_ARRAY(p)`: Bir dizi işaretçisini siler ve NULL olarak ayarlar.
        *   `SAFE_RELEASE(p)`: COM benzeri bir nesnenin `Release()` metodunu çağırır ve işaretçiyi NULL yapar.
        *   `SAFE_FREE_GLOBAL(p)`: `GlobalFree` ile ayrılmış belleği serbest bırakır ve işaretçiyi NULL yapar.
        *   `SAFE_FREE_LIBRARY(p)`: `FreeLibrary` ile yüklenmiş bir kütüphaneyi serbest bırakır ve işaretçiyi NULL yapar.
    *   **Bit İşlemleri Makroları:**
        *   `IS_SET(flag, bit)`: Belirli bir bitin ayarlı olup olmadığını kontrol eder.
        *   `SET_BIT(var, bit)`: Belirli bir biti ayarlar.
        *   `REMOVE_BIT(var, bit)`: Belirli bir biti kaldırır.
        *   `TOGGLE_BIT(var, bit)`: Belirli bir bitin durumunu tersine çevirir.
    *   **Dosya ve Dizin Yardımcıları:**
        *   `CreateTempFileName(const char* c_pszPrefix)`: Geçici bir dosya adı oluşturur.
        *   `GetFilePathNameExtension(...)`, `GetFileExtension(...)`, `GetFileNameParts(...)`: Dosya yollarından dosya adı, uzantısı ve yol gibi kısımları ayıklar.
        *   `GetOnlyFileName(...)`, `GetOnlyPathName(...)`: Sadece dosya adını veya sadece yolunu alır.
        *   `GetWorkingFolder(std::string& strFileName)`: Çalışma dizinini alır.
        *   `IsFile(const char* filename)`: Verilen yolun bir dosya olup olmadığını kontrol eder.
        *   `IsGlobalFileName(const char* c_szFileName)`: Dosya adının global bir yol olup olmadığını kontrol eder (muhtemelen özel bir formata göre).
        *   `MyCreateDirectory(const char* path)`: Dizin oluşturur.
        *   `RemoveAllDirectory(const char* c_szDirectoryName)`: Bir dizini ve içindekileri siler.
    *   **String İşleme Fonksiyonları:**
        *   `stl_lowers(std::string& rstRet)`: Bir string'i küçük harfe çevirir (STL string).
        *   `StringLowers(char* pString)`: Bir C-string'i küçük harfe çevirir.
        *   `StringPath(...)`: Dosya yollarını normalleştirir (örneğin, `\` karakterlerini `/` ile değiştirir).
        *   `SplitLine(...)`: Bir string'i belirtilen ayraçlara göre böler.
        *   `_getf(const char* c_szFormat, ...)`: `sprintf` benzeri formatlı string oluşturur.
        *   `CommandLineToArgv(PCHAR CmdLine, int* _argc)`: Komut satırı argümanlarını ayrıştırır.
        *   `string_join(...)` (template): Bir konteynırdaki string'leri belirli bir ayraçla birleştirir.
        *   `string_replace_all(...)` (template): Bir string içindeki tüm belirli alt string'leri başka bir string ile değiştirir.
        *   `htoi(...)`: Hexadecimal (onaltılık) string'i integer'a çevirir (char ve wchar_t için overload edilmiş).
        *   `StringExceptCharacter(...)`: Bir string'den belirli karakterleri çıkarır.
        *   `GetExcutedFileName(std::string& r_str)`: Çalıştırılan uygulamanın dosya adını alır.
    *   **Matematiksel ve Geometrik Yardımcılar (Templates):**
        *   `MIN(a, b)`, `MAX(a, b)`, `MINMAX(min, value, max)`: Temel min/max işlemleri.
        *   `fMIN(a, b)`, `fMAX(a, b)`, `fMINMAX(min, value, max)`: Float için min/max işlemleri.
        *   `EL_DegreeToRadian(T degree)`: Dereceyi radyana çevirir.
        *   `ELPlainCoord_GetRotatedPixelPosition(...)`: Verilen merkez, mesafe ve açıya göre döndürülmüş piksel pozisyonunu hesaplar.
        *   `EL_SignedDegreeToUnsignedDegree(T fSrc)`: İşaretli açıyı (örn: -90) işaretsiz eşdeğerine (örn: 270) çevirir.
        *   `ELRightCoord_ConvertToPlainCoordDegree(T srcDegree)`: Sağ el koordinat sistemindeki açıyı düzlem koordinat sistemindeki açıya çevirir (muhtemelen grafiksel dönüşümler için).
    *   **Diğer Yardımcılar:**
        *   `MAKEFOURCC(...)`: Dört karakterli kod oluşturur (genellikle dosya formatları veya kodek tanımlayıcıları için).
        *   `PrintAsciiData(const void* data, int bytes)`: Verilen bellek bölgesini ASCII olarak yazdırır (hata ayıklama için).
*   **Kullanım Alanı:** Bu dosyadaki fonksiyonlar ve makrolar, istemcinin hemen hemen her modülünde, bellek yönetiminden string manipülasyonuna, dosya işlemlerinden matematiksel hesaplamalara kadar çok çeşitli görevlerde kullanılır.

### `Stl.h`

*   **Amaç:** Standart Şablon Kütüphanesi (STL) bileşenleriyle çalışmayı kolaylaştıran yardımcı fonksiyonlar, özel yapılar ve şablonlar sunar. STL konteynerlerinin yönetimi, özel karşılaştırma funktörleri ve STL ile ilgili yaygın görevler için araçlar içerir.
*   **Temel Özellikler/İçerik:**
    *   **Uyarı Yönetimi:** `#pragma warning(disable:...)` direktifleri ile STL başlık dosyalarından kaynaklanabilecek bazı yaygın derleyici uyarılarını (örneğin, 4786, 4018, 4503) devre dışı bırakır.
    *   **STL Başlık Dosyaları:** Sık kullanılan STL başlıklarını (`<algorithm>`, `<string>`, `<vector>`, `<map>`, `<set>` vb.) içerir.
    *   **Yardımcı Fonksiyonlar ve Yapılar:**
        *   `korean_tolower(const char c)`: Korece bir karakteri küçük harfe çevirir (muhtemelen özel bir karakter seti için).
        *   `stl_static_string(const char* c_sz)`: Verilen C-string için statik bir `std::string` nesnesi döndürür (dikkatli kullanılmalıdır, birden fazla çağrıda aynı statik nesneyi döndürebilir).
        *   `stl_lowers(std::string& rstRet)`: Bir `std::string`'i küçük harfe çevirir (Bu fonksiyon `Utils.h` içinde de deklare edilmiş olabilir, burada tekrar yer alması ilginçtir).
        *   `split_string(const std::string& input, const std::string& delimiter, std::vector<std::string>& results, bool includeEmpties)`: Bir string'i belirtilen ayraçlara göre böler ve sonuçları bir `std::vector<std::string>` içine atar.
        *   `stl_sz_less` Yapısı: C-string'leri (`char*`) karşılaştırmak için bir funktör (işlev nesnesi). `std::map` veya `std::set` gibi sıralı konteynerlerde anahtar olarak C-string kullanıldığında faydalıdır.
        *   `stl_wipe(TContainer& container)` (template): Bir konteyner içindeki tüm işaretçi elemanlarını `delete` eder, işaretçileri NULL yapar ve konteyneri temizler. Konteyner işaretçi tutuyorsa bellek sızıntılarını önlemek için kullanılır.
        *   `hex2dec(TString szhex)` (template): İki karakterlik hexadecimal (onaltılık) bir string parçasını onluk (decimal) değere çevirir.
        *   `htmlColorStringToARGB(TString str)` (template): HTML renk kodunu (örn: "FF00AA") ARGB (Alpha-Red-Green-Blue) formatında bir `unsigned long` değere çevirir.
        *   `stl_wipe_second(TContainer& container)` (template): Özellikle `std::map` gibi çiftlerden oluşan konteynerler için tasarlanmıştır. Konteynerdeki her elemanın `second` üyesini (genellikle bir işaretçi) `delete` eder ve konteyneri temizler.
        *   `safe_release(T& rpObject)` (template): Bir COM benzeri nesnenin `Release()` metodunu güvenli bir şekilde çağırır ve işaretçiyi NULL yapar (`Utils.h` içindeki `SAFE_RELEASE` makrosuna benzer bir işlevsellik sunar ancak şablon fonksiyonu olarak).
        *   `DeleteVectorItem(...)` (template, overload edilmiş): Bir `std::vector`'dan belirli bir indeksteki, belirli bir aralıktaki veya belirli bir değere sahip elemanı siler.
        *   `DeleteListItem(...)` (template): Bir `std::list`'ten belirli bir değere sahip elemanı siler.
    *   **Özel Havuz (Pool) Yapıları (İleri Düzey):**
        *   `stl_stack_pool<TData, THandle, TCount>` (template): Sabit boyutlu bir yığın (stack) tabanlı bellek havuzu. Elemanları önceden ayırır ve hızlı tahsis/serbest bırakma sağlar. `THandle` tipi, havuzdaki bir elemana referans için kullanılır.
            *   `initialize(int capacity)`: Havuzu başlatır.
            *   `alloc()`: Havuzdan bir eleman tahsis eder.
            *   `free(THandle)`: Tahsis edilmiş bir elemanı havaza geri verir.
            *   `refer(THandle)`: Verilen handle ile elemana referans döndürür.
        *   `stl_circle_pool<TData, THandle, TCount>` (template): Sabit boyutlu bir dairesel (circular) bellek havuzu. `stl_stack_pool`'a benzer şekilde çalışır ancak farklı bir tahsis/serbest bırakma stratejisi kullanabilir.
            *   `create(int size)`: Havuzu oluşturur.
            *   `destroy()`: Havuzu yok eder.
            *   Diğer fonksiyonlar (`alloc`, `free`, `refer`) `stl_stack_pool`'a benzer.
    *   **String Hash Yapısı:**
        *   `stringhash`: `std::string` nesneleri için bir hash değeri hesaplayan bir funktör. `std::unordered_map` veya `std::unordered_set` gibi hash tabanlı STL konteynerlerinde `std::string` anahtarlarıyla kullanılmak üzere tasarlanmıştır. Basit bir çarpma ve XOR tabanlı hash algoritması kullanır.
*   **Kullanım Alanı:** STL konteynerlerinin daha verimli ve güvenli kullanımı, özel veri yapılarının (havuzlar gibi) oluşturulması, string işlemleri ve C-string'lerin STL konteynerlerinde anahtar olarak kullanılması gibi durumlarda bu dosyadaki araçlar devreye girer.

### `Singleton.h`

*   **Amaç:** Singleton tasarım desenini uygulamak için genel bir şablon sınıfı (`CSingleton<T>` ve `singleton<T>`) sağlar. Bu desen, bir sınıftan yalnızca tek bir örneğin oluşturulabilmesini ve bu örneğe global bir erişim noktası sunulmasını garanti eder.
*   **Temel Özellikler/İçerik:**
    *   **`CSingleton<T>` Şablon Sınıfı:**
        *   `static T* ms_singleton`: Singleton örneğini tutan statik bir işaretçi. Tüm `T` türündeki singleton nesneleri için paylaşılır.
        *   **Yapıcı (`CSingleton()`):**
            *   `ms_singleton`'un daha önce ayarlanmamış olduğunu `assert` ile kontrol eder (yani, bu türden başka bir singleton örneği oluşturulmamış olmalıdır).
            *   `ms_singleton` işaretçisini, türetilmiş sınıfın (`T`) örneğine doğru bir şekilde ayarlamak için bir offset hesaplaması yapar. Bu, `CSingleton`'ın `T`'nin bir taban sınıfı olduğu ve `T`'nin sanal kalıtım veya çoklu kalıtım gibi durumlar içerebileceği senaryolarda doğru işaretçiyi elde etmek için yapılan bir tekniktir.
        *   **Yıkıcı (`virtual ~CSingleton()`):**
            *   `ms_singleton`'un ayarlı olduğunu `assert` ile kontrol eder.
            *   `ms_singleton`'u `0` (NULL) olarak ayarlar, böylece örnek yok edildiğinde işaretçi geçersiz hale gelir.
        *   **`static T& Instance()` ve `static T& instance()`:**
            *   Singleton örneğine bir referans döndürür.
            *   `ms_singleton`'un geçerli bir örnek gösterdiğini `assert` ile kontrol eder.
        *   **`static T* InstancePtr()`:**
            *   Singleton örneğine bir işaretçi döndürür.
    *   **`singleton<T>` Şablon Sınıfı:**
        *   `CSingleton<T>` ile tamamen aynı işlevselliğe ve yapıya sahiptir. Farklı bir isimlendirme (Macar notasyonu olmayan `singleton`) sunar, muhtemelen kodlama stili tercihleri veya farklı kullanım senaryoları için eklenmiştir.
    *   **Statik Üye Tanımları:**
        *   `template <typename T> T* CSingleton <T>::ms_singleton = 0;`
        *   `template <typename T> T* singleton <T>::ms_singleton = 0;`
        *   Bu satırlar, her `CSingleton<T>` ve `singleton<T>` uzmanlaşması için statik `ms_singleton` işaretçisini tanımlar ve başlangıçta `0` (NULL) olarak ayarlar.
*   **Kullanım Şekli:**
    ```cpp
    // Örnek Kullanım
    class MyManager : public CSingleton<MyManager>
    {
    public:
        MyManager() {}
        ~MyManager() {}
        void DoSomething() { /* ... */ }
    };

    // Erişmek için:
    // MyManager::Instance().DoSomething();
    // veya
    // if (MyManager::InstancePtr())
    //     MyManager::InstancePtr()->DoSomething();
    ```
*   **Kullanım Alanı:** Günlük yöneticisi (logger), konfigürasyon yöneticisi, kaynak yöneticisi gibi sistem genelinde tek bir örneği olması gereken ve kolayca erişilebilmesi istenen sınıflar için kullanılır.

### `FileLoader.h`

*   **Amaç:** Dosyaları farklı kaynaklardan (bellek içi tamponlar veya disk) yüklemek için sınıflar tanımlar. Hem metin tabanlı hem de ikili (binary) dosyaların okunmasını destekler.
*   **Temel Özellikler/İçerik:**
    *   **`CMemoryTextFileLoader` Sınıfı:**
        *   **Amaç:** Bellekte tutulan bir veri bloğunu metin dosyası gibi satır satır işlemek için kullanılır.
        *   `Bind(int bufSize, const void* c_pvBuf)`: Verilen bellek tamponunu ve boyutunu sınıfa bağlar. Bellekteki veriyi satırlara ayırır ve `m_stLineVector` içinde saklar.
        *   `GetLineCount()`: Dosyadaki toplam satır sayısını döndürür.
        *   `CheckLineIndex(DWORD dwLine)`: Verilen satır indeksinin geçerli olup olmadığını kontrol eder.
        *   `SplitLine(DWORD dwLine, CTokenVector* pstTokenVector, const char* c_szDelimeter = " \t")`: Belirtilen satırı verilen ayraçlara (varsayılan olarak boşluk ve tab) göre böler ve token'ları `CTokenVector` (muhtemelen `std::vector<std::string>`) içine atar. Başarı durumunu `bool` olarak döndürür.
        *   `SplitLine2(DWORD dwLine, CTokenVector* pstTokenVector, const char* c_szDelimeter = " \t")`: `SplitLine`'a benzer şekilde çalışır ancak token sayısını `int` olarak döndürür.
        *   `SplitLineByTab(DWORD dwLine, CTokenVector* pstTokenVector)`: Belirtilen satırı özellikle tab karakterlerine (`\t`) göre böler.
        *   `GetLineString(DWORD dwLine)`: Belirtilen satırın tamamını `std::string` olarak döndürür.
        *   **Korumalı Üye:**
            *   `m_stLineVector`: Bellekten okunan ve satırlara bölünmüş metin verisini tutan `std::vector<std::string>`.
    *   **`CMemoryFileLoader` Sınıfı:**
        *   **Amaç:** Bellekte tutulan bir veri bloğunu ikili (binary) dosya gibi okumak için kullanılır.
        *   **Yapıcı (`CMemoryFileLoader(int size, const void* c_pvMemoryFile)`)**: Verilen boyutta ve adreste bir bellek dosyasını temsil edecek şekilde başlatılır.
        *   `Read(int size, void* pvDst)`: Mevcut pozisyondan belirtilen `size` kadar baytı `pvDst` tamponuna okur. Başarılı olursa `true` döner.
        *   `GetPosition()`: Dosya içindeki mevcut okuma pozisyonunu döndürür.
        *   `GetSize()`: Bellek dosyasının toplam boyutunu döndürür.
        *   **Korumalı Metotlar:**
            *   `IsReadableSize(int size)`: Mevcut pozisyondan itibaren istenen `size` kadar baytın okunabilir olup olmadığını kontrol eder.
            *   `GetCurrentPositionPointer()`: Mevcut okuma pozisyonundaki işaretçiyi döndürür.
        *   **Korumalı Üyeler:**
            *   `m_pcBase`: Bellek dosyasının başlangıç adresini tutan işaretçi.
            *   `m_size`: Bellek dosyasının toplam boyutu.
            *   `m_pos`: Mevcut okuma pozisyonu.
    *   **`CDiskFileLoader` Sınıfı:**
        *   **Amaç:** Diskteki bir dosyadan ikili (binary) veri okumak için kullanılır.
        *   `Open(const char* c_szFileName)`: Belirtilen isimdeki dosyayı okuma modunda (`"rb"`) açar.
        *   `Close()`: Açık dosyayı kapatır.
        *   `Read(int size, void* pvDst)`: Dosyadan belirtilen `size` kadar baytı `pvDst` tamponuna okur. Başarılı olursa `true` döner.
        *   `GetSize()`: Açılan dosyanın toplam boyutunu döndürür.
        *   **Korumalı Üyeler:**
            *   `m_fp`: C standart kütüphanesinden `FILE*` işaretçisi.
            *   `m_size`: Dosyanın boyutu.
*   **Kullanım Alanı:**
    *   `CMemoryTextFileLoader`: Genellikle `.txt`, `.csv`, `.ini` gibi metin tabanlı konfigürasyon dosyalarını veya betik dosyalarını bellekten (örneğin, bir paket dosyasından çıkarıldıktan sonra) okumak ve satır satır işlemek için kullanılır.
    *   `CMemoryFileLoader`: Belleğe yüklenmiş ikili veri bloklarını (örneğin, sıkıştırılmış veriler, resim verileri, model verileri) sıralı bir şekilde okumak için kullanılır.
    *   `CDiskFileLoader`: Doğrudan disk üzerinde bulunan ikili dosyaları (oyun varlıkları, büyük veri dosyaları vb.) okumak için standart bir yol sunar.
*   **Not:** `CTokenVector` muhtemelen `Utils.h` veya `Stl.h` içinde `typedef std::vector<std::string> TTokenVector;` şeklinde tanımlanmış bir türdür.

### `CRC32.h`

*   **Amaç:** Veri bütünlüğünü kontrol etmek amacıyla CRC32 (Cyclic Redundancy Check) özet değerlerini hesaplamak için fonksiyonlar sunar. Dosyaların veya bellek bloklarının değişip değişmediğini tespit etmek için kullanılır.
*   **Temel Özellikler/İçerik:**
    *   `GetCRC32(const char* buffer, size_t count)`: Verilen bellek tamponundaki (`buffer`) belirtilen `count` kadar baytın CRC32 özetini hesaplar ve `DWORD` olarak döndürür.
    *   `GetCaseCRC32(const char* buf, size_t len)`: Verilen tampondaki verinin büyük/küçük harfe duyarlı olmayan bir şekilde CRC32 özetini hesaplar. Bu genellikle dosya adları gibi metin tabanlı verilerin karşılaştırılmasında kullanılır.
    *   `GetHFILECRC32(HANDLE hFile)`: Açık bir dosya tanıtıcısı (`HANDLE`) üzerinden dosyanın CRC32 özetini hesaplar.
    *   `GetFileCRC32(const char* c_szFileName)`: Belirtilen dosya adındaki dosyanın CRC32 özetini hesaplar.
    *   `GetFileSize(const char* c_szFileName)`: Belirtilen dosyanın boyutunu bayt cinsinden `DWORD` olarak döndürür. (Doğrudan CRC32 ile ilgili olmasa da dosya işlemleri için bir yardımcıdır.)
*   **Bağımlılıklar:**
    *   `#include "../UserInterface/Locale_inc.h"`: Bu başlık dosyasının `UserInterface` modülündeki `Locale_inc.h` dosyasına bir bağımlılığı bulunmaktadır. Bu, temel bir kütüphane için dikkat çekici bir durumdur ve genellikle yerelleştirme ile ilgili bazı tanımlamalar veya ayarlar için olabilir.
*   **Kullanım Alanı:**
    *   Dosya bütünlüğünün doğrulanması (örneğin, oyun yaması veya güncellemesi sonrası dosyaların doğru indirilip indirilmediğini kontrol etmek).
    *   Ağ üzerinden gönderilen veri paketlerinin doğruluğunu kontrol etmek.
    *   Bellekteki önemli veri yapılarının bozulup bozulmadığını tespit etmek.

### `cipher.h`

*   **Amaç:** `#if defined(__IMPROVED_PACKET_ENCRYPTION__)` koşuluna bağlı olarak, iletişim kanallarını (özellikle ağ paketlerini) şifrelemek ve deşifrelemek için bir `Cipher` sınıfı tanımlar. Crypto++ kütüphanesini kullanarak simetrik şifreleme algoritmaları ve anahtar anlaşması mekanizmaları sağlar.
*   **Koşullu Derleme:** Bu dosyadaki `Cipher` sınıfı ve ilgili işlevsellik, yalnızca `__IMPROVED_PACKET_ENCRYPTION__` makrosu tanımlı olduğunda derlenir. Bu, geliştirilmiş paket şifrelemesinin isteğe bağlı bir özellik olduğunu gösterir.
*   **Temel Özellikler/İçerik (`Cipher` Sınıfı):**
    *   **Bağımlılıklar:** Crypto++ kütüphanesine (`cryptopp/cryptlib.h`) dayanır.
    *   **Yapıcı/Yıkıcı (`Cipher()`, `~Cipher()`):** Sınıfın kaynaklarını başlatır ve temizler (`CleanUp()`).
    *   **Anahtar Anlaşması:**
        *   `Prepare(void* buffer, size_t* length)`: Muhtemelen anahtar anlaşması sürecinin bir parçası olarak karşı tarafa gönderilecek veriyi hazırlar. `buffer` içine yazılan verinin boyutunu `length` ile döndürür.
        *   `Activate(bool polarity, size_t agreed_length, const void* buffer, size_t length)`: Karşı taraftan alınan anahtar anlaşması verilerini kullanarak şifreleme algoritmasını ve anahtarları etkinleştirir. `polarity` argümanı, anahtar anlaşması sürecindeki rolü (başlatan/yanıtlayan) belirleyebilir.
        *   `KeyAgreement* key_agreement_`: Anahtar anlaşması işlemlerini yürüten bir nesneye işaretçi.
    *   **Şifreleme/Deşifreleme:**
        *   `Encrypt(void* buffer, size_t length)`: Verilen `buffer`'daki `length` boyutundaki veriyi yerinde (in-place) şifreler. Şifrelemenin aktif (`activated_`) olmasını gerektirir.
        *   `Decrypt(void* buffer, size_t length)`: Verilen `buffer`'daki `length` boyutundaki veriyi yerinde (in-place) deşifreler. Şifrelemenin aktif (`activated_`) olmasını gerektirir.
        *   `CryptoPP::SymmetricCipher* encoder_`: Crypto++ kütüphanesinden şifreleyici nesnesi.
        *   `CryptoPP::SymmetricCipher* decoder_`: Crypto++ kütüphanesinden deşifreleyici nesnesi.
    *   **Durum Yönetimi:**
        *   `activated_`: Şifrelemenin başarılı bir şekilde başlatılıp etkinleştirildiğini gösteren bir boolean bayrak.
        *   `activated()`: `activated_` durumunu döndürür.
        *   `set_activated(bool value)`: `activated_` durumunu ayarlar.
        *   **Not:** Kod içindeki `//THEMIDA` yorumu, bu şifreleme mekanizmasının Themida yazılım koruma sistemiyle bir bağlantısı olabileceğine veya benzer prensipler kullanabileceğine işaret edebilir.
*   **Kullanım Alanı:** Özellikle istemci ile sunucu arasındaki ağ trafiğinin güvenliğini artırmak amacıyla, paketlerin içeriğini şifreleyerek yetkisiz erişime ve veri manipülasyonuna karşı koruma sağlamak için kullanılır. Bu, oyun hilelerini önlemeye ve kullanıcı verilerini korumaya yardımcı olabilir.

### `tea.h`

*   **Amaç:** TEA (Tiny Encryption Algorithm) ve muhtemelen onun bir varyantını kullanarak veri şifreleme ve deşifreleme fonksiyonları sunar. TEA, 64-bitlik veri blokları üzerinde 128-bitlik bir anahtar ile çalışan simetrik bir blok şifreleme algoritmasıdır.
*   **Temel Özellikler/İçerik:**
    *   **C Bağlantısı:** Fonksiyonlar `extern "C"` ile deklare edilmiştir, bu da C ve C++ uyumluluğu sağlar.
    *   **`TEA_KEY_LENGTH` Makrosu:** `16` olarak tanımlanmıştır, bu da 128 bitlik anahtarın byte cinsinden uzunluğuna (16 byte) karşılık gelir.
    *   **Modern (?) TEA Fonksiyonları:**
        *   `tea_encrypt(unsigned long* dest, const unsigned long* src, uint16_t keySet, int size)`: `src` işaretçisindeki `size` kadar veriyi (unsigned long blokları halinde) şifreler ve `dest` işaretçisine yazar. `keySet` parametresi, kullanılacak şifreleme anahtarını belirlemek için bir tanımlayıcı veya indeks olabilir, bu da anahtar yönetiminin soyutlandığını gösterir.
        *   `tea_decrypt(unsigned long* dest, const unsigned long* src, uint16_t keySet, int size)`: `src` işaretçisindeki `size` kadar şifreli veriyi (unsigned long blokları halinde) deşifreler ve `dest` işaretçisine yazar. `keySet` parametresi, `tea_encrypt` ile aynı şekilde anahtarı belirler.
    *   **Eski (?) TEA Fonksiyonları:**
        *   `old_tea_encrypt(unsigned long* dest, const unsigned long* src, const unsigned long* key, int size)`: `src` işaretçisindeki veriyi, doğrudan sağlanan 128-bitlik `key` (4 adet `unsigned long`) kullanarak şifreler ve `dest` işaretçisine yazar.
        *   `old_tea_decrypt(unsigned long* dest, const unsigned long* src, const unsigned long* key, int size)`: `src` işaretçisindeki şifreli veriyi, doğrudan sağlanan 128-bitlik `key` kullanarak deşifreler ve `dest` işaretçisine yazar.
*   **Notlar:**
    *   Başlık dosyasındaki yorumlar, bu implementasyonun David J. Wheeler ve Roger M. Needham tarafından geliştirilen orijinal TEA/XTEA koduna dayandığını belirtir.
    *   `keySet` parametresinin tam olarak nasıl bir anahtar yönetim şemasına işaret ettiği, ilgili `.cpp` dosyasındaki implementasyon detaylarına bakılarak daha iyi anlaşılabilir.
*   **Kullanım Alanı:** Küçük veri bloklarının hızlı bir şekilde şifrelenmesi ve deşifrelenmesi gereken durumlarda kullanılır. Bu, basit veri koruması, oyun içi verilerin hafif şifrelenmesi veya `cipher.h` içindeki gibi daha karmaşık şifreleme sistemlerinin bir parçası olarak kullanılabilir.

### `lzo.h`

*   **Amaç:** LZO (Lempel-Ziv-Oberhumer) veri sıkıştırma ve açma işlemleri için sınıflar ve arayüzler sunar. Ayrıca, sıkıştırılmış verinin isteğe bağlı olarak şifrelenmesini de destekler.
*   **Bağımlılıklar:** Harici LZO kütüphanesine (`lzo/lzo1x.h`) ve `Singleton.h` dosyasına bağımlıdır.
*   **Temel Özellikler/İçerik:**
    *   **`CLZObject` Sınıfı:**
        *   **Amaç:** Sıkıştırılacak/açılacak bir veri bloğunu, başlık bilgilerini ve ilgili işlemleri yönetir.
        *   **`SHeader` Yapısı (İç İçe):**
            *   `dwFourCC`: Dört karakterli kod (tanımlayıcı, örneğin dosya formatı için).
            *   `dwEncryptSize`: Şifrelenmiş verinin boyutu.
            *   `dwCompressedSize`: Sıkıştırılmış verinin boyutu.
            *   `dwRealSize`: Orijinal (sıkıştırılmamış) verinin boyutu.
        *   **Sıkıştırma Metotları:**
            *   `BeginCompress(const void* pvIn, UINT uiInLen)`: Verilen girdiyi sıkıştırma işlemine hazırlar.
            *   `BeginCompressInBuffer(const void* pvIn, UINT uiInLen, void* pvOut)`: Verilen girdiyi doğrudan sağlanan çıktı tamponuna sıkıştırma işlemine hazırlar.
            *   `Compress()`: Hazırlanmış veriyi sıkıştırır.
        *   **Açma Metotları:**
            *   `BeginDecompress(const void* pvIn)`: Verilen (sıkıştırılmış) girdiyi açma işlemine hazırlar.
            *   `Decompress(uint16_t uiKeySet)`: Hazırlanmış veriyi açar. `uiKeySet` ile muhtemelen önce deşifreleme yapar.
        *   **Şifreleme/Deşifreleme Metotları:**
            *   `Encrypt(uint16_t uiKeySet)`: `CLZObject` içindeki tamponlanmış veriyi (genellikle sıkıştırıldıktan sonra) belirtilen `uiKeySet` ile şifreler.
            *   `__Decrypt(uint16_t uiKeySet, BYTE* pbData)`: Verilen `pbData`'yı `uiKeySet` ile deşifreler (muhtemelen `Decompress` içinde dahili olarak kullanılır).
        *   **Tampon Yönetimi:** `GetBuffer()`, `GetSize()`, `AllocBuffer()` gibi metotlarla iç veri tamponunu yönetir.
        *   `static DWORD ms_dwFourCC`: Sınıf için varsayılan bir FourCC değeri tutar.
    *   **`CLZO` Sınıfı (Singleton):**
        *   **Amaç:** LZO sıkıştırma/açma işlemleri için genel bir arayüz ve kaynak yönetimi sağlar.
        *   `CSingleton<CLZO>`'dan türetilmiştir.
        *   `CompressMemory(CLZObject& rObj, const void* pIn, UINT uiInLen)`: Verilen `pIn` verisini `rObj` içine sıkıştırır.
        *   `CompressEncryptedMemory(CLZObject& rObj, const void* pIn, UINT uiInLen, uint16_t uiKeySet)`: Verilen `pIn` verisini `rObj` içine sıkıştırır ve ardından `uiKeySet` ile şifreler.
        *   `Decompress(CLZObject& rObj, const BYTE* pbBuf, uint16_t uiKeySet = 0)`: Verilen `pbBuf` (sıkıştırılmış ve muhtemelen şifrelenmiş) veriyi `rObj` içine açar. Gerekirse `uiKeySet` ile deşifreleme yapar.
        *   `GetWorkMemory()`: LZO algoritmasının ihtiyaç duyduğu çalışma belleğini döndürür.
        *   `m_pWorkMem`: LZO işlemleri için ayrılmış çalışma belleği işaretçisi.
*   **Kullanım Alanı:** Oyun varlıklarını (örneğin, `.epk`/`.eix` paket dosyaları içindeki dosyalar) diskte daha az yer kaplaması için sıkıştırmak, ağ üzerinden transfer edilen verinin boyutunu küçültmek ve bu verileri isteğe bağlı olarak (muhtemelen TEA algoritması ve `keySet` ile) şifrelemek için kullanılır. Bu, hem depolama verimliliği sağlar hem de veri güvenliğine katkıda bulunur.

### `Debug.h`

*   **Amaç:** Hata ayıklama, izleme (tracing) ve loglama işlemleri için kapsamlı bir fonksiyon ve makro seti sunar. Geliştirme sırasında programın davranışını izlemek, hataları tespit etmek ve önemli olayları kaydetmek için kullanılır.
*   **Temel Özellikler/İçerik:**
    *   **Loglama Fonksiyonları (Seviyeli):**
        *   `SetLogLevel(UINT uLevel)`: Global loglama seviyesini ayarlar. Sadece bu seviye veya daha önemli seviyedeki loglar işlenir.
        *   `Log(UINT uLevel, const char* c_szMsg)`: Verilen mesajı belirtilen seviyede loglar (muhtemelen satır sonu ekler).
        *   `Logn(UINT uLevel, const char* c_szMsg)`: Verilen mesajı belirtilen seviyede loglar (satır sonu eklemez).
        *   `Logf(UINT uLevel, const char* c_szFormat, ...)`: Formatlı bir mesajı belirtilen seviyede loglar (satır sonu ekler).
        *   `Lognf(UINT uLevel, const char* c_szFormat, ...)`: Formatlı bir mesajı belirtilen seviyede loglar (satır sonu eklemez).
    *   **İzleme (Trace) Fonksiyonları:**
        *   `Trace(const char* c_szMsg)`: Genel bir izleme mesajı loglar (muhtemelen varsayılan bir log seviyesiyle).
        *   `Tracen(const char* c_szMsg)`: Satır sonu eklemeden izleme mesajı loglar.
        *   `Tracef(const char* c_szFormat, ...)`: Formatlı bir izleme mesajı loglar.
        *   `Tracenf(const char* c_szFormat, ...)`: Satır sonu eklemeden formatlı bir izleme mesajı loglar.
        *   `TraceError(const char* c_szFormat, ...)`: Bir hata mesajını özel olarak formatlayıp loglar (genellikle daha görünür bir şekilde).
        *   `TraceErrorWithoutEnter(const char* c_szFormat, ...)`: `TraceError` gibi ancak satır sonu eklemez.
    *   **Mesaj Kutusu (Popup) Loglama:**
        *   `LogBox(const char* c_szMsg, const char* c_szCaption = NULL, HWND hWnd = NULL)`: Kullanıcıya bir Windows mesaj kutusu ile bir mesaj gösterir. Başlık ve ebeveyn pencere belirtilebilir.
        *   `LogBoxf(const char* c_szMsg, ...)`: Formatlı bir mesajı `LogBox` ile gösterir.
    *   **Dosyaya Loglama:**
        *   `LogFile(const char* c_szMsg)`: Mesajı önceden tanımlanmış bir log dosyasına yazar.
        *   `LogFilef(const char* c_szMessage, ...)`: Formatlı bir mesajı log dosyasına yazar.
        *   `OpenLogFile(bool bUseLogFile = true)`: Log dosyasını açar. `bUseLogFile` false ise dosya loglaması pasif hale getirilebilir.
        *   `CloseLogFile()`: Açık olan log dosyasını kapatır.
    *   **Konsol Yönetimi:**
        *   `OpenConsoleWindow()`: Hata ayıklama çıktıları için bir Windows konsol penceresi açar.
        *   `CloseConsoleWindow()`: Açılan konsol penceresini kapatır.
    *   **Genel Kurulum:**
        *   `SetupLog()`: Loglama sisteminin genel ayarlarını yapar (örneğin, log dosyası adını belirler, konsolu açar vb.).
    *   **Yardımcı Makro:**
        *   `CHECK_RETURN(flag, string)`: Eğer `flag` koşulu doğru (true) ise, `string` mesajını bir `LogBox` ile gösterir ve ardından mevcut fonksiyondan `return` ifadesiyle çıkar. Hata kontrolü ve erken çıkışlar için kullanılır.
    *   **Global Değişken:**
        *   `HWND g_PopupHwnd`: `LogBox` gibi popup pencereleri için varsayılan ebeveyn pencere tanıtıcısını tutar.
*   **Kullanım Alanı:** Geliştirme sürecinde program akışını takip etmek, değişken değerlerini görmek, hataları ayıklamak, önemli olayları ve istisnai durumları kaydetmek için istemcinin çeşitli yerlerinde kullanılır. Hem geliştirici konsoluna, hem log dosyalarına hem de kullanıcıya doğrudan mesaj kutularıyla çıktı verebilir.

(Bu bölüme, EterBase içindeki belirli dosyaların detaylı açıklamaları eklenecektir.) 

## `Poly/` Alt Klasörü

`EterBase` modülü içinde yer alan `Poly/` alt klasörü, sembolik hesaplama veya matematiksel ifade ayrıştırma/değerlendirme ile ilgili olabilecek bir polinom kütüphanesi veya benzeri bir yapı içermektedir. Bu klasördeki sınıflar, muhtemelen istemci içinde dinamik olarak hesaplanabilen veya script'ler aracılığıyla tanımlanabilen matematiksel ifadelerin işlenmesinde kullanılır. 

### `Poly/Base.h` ve `Poly/Base.cpp`

*   **Amaç:** `Poly` kütüphanesi içindeki diğer sınıflar için bir temel sınıf (`CBase`) ve temel tür tanımlayıcıları (ID'ler) sağlar.
*   **Temel Özellikler/İçerik (`Base.h`):**
    *   **Tür Tanımlayıcı Makroları:**
        *   `MID_UNKNOWN`: Bilinmeyen tür.
        *   `MID_NUMBER`: Sayısal bir değeri temsil eder.
        *   `MID_VARIABLE`: Bir değişkeni temsil eder.
        *   `MID_SYMBOL`: Bir sembolü (operatör, fonksiyon vb.) temsil eder.
        *   Özel Sayı Türleri: `MID_LONG`, `MID_SQRT`, `MID_FRACTION` gibi `MID_NUMBER` üzerine inşa edilmiş daha spesifik sayısal türler.
    *   **`CBase` Sınıfı:**
        *   `id (int)`: Nesnenin türünü belirten bir kimlik. Yukarıdaki `MID_` makrolarından birini veya bir kombinasyonunu alır.
        *   `CBase()`: Kurucu metod, `id`'yi `0` (muhtemelen `MID_UNKNOWN`) olarak başlatır.
        *   `virtual ~CBase()`: Sanal yıkıcı metod.
        *   `isSymbol()`, `isVar()`, `isNumber()`: Nesnenin `id` üyesindeki ilgili bitleri (`MID_SYMBOL`, `MID_VARIABLE`, `MID_NUMBER`) kontrol ederek türünü sorgulayan metodlar.
*   **Implementasyon Detayları (`Base.cpp`):**
    *   `CBase` sınıfının kurucu ve yıkıcı metodlarının basit implementasyonlarını içerir.
    *   `isSymbol()`, `isVar()`, `isNumber()` metodları, `id` üyesi üzerinde bitwise AND işlemi yaparak ilgili tür bayrağının ayarlı olup olmadığını kontrol eder.
*   **Kullanım Alanı:** Bu sınıf, `Poly` kütüphanesi içinde farklı türdeki matematiksel ifadeleri (sayılar, değişkenler, semboller/operatörler) polimorfik bir şekilde yönetmek için bir temel oluşturur. İfade ağacındaki (expression tree) düğümlerin ortak bir taban türüne sahip olmasını sağlar. İstemci tarafında, örneğin karakter stat hesaplamalarında, hasar formüllerinde veya script ile tanımlanan dinamik değerlerde kullanılabilir. Formüllerin ayrıştırılması ve değerlendirilmesi sırasında, ifadenin her bir bileşeninin türünü (sayı, değişken, operatör) belirlemek için `id` ve `is*()` metodları kullanılır. 

### `Poly/Symbol.h` ve `Poly/Symbol.cpp`

*   **Amaç:** Matematiksel ifadelerdeki sembolleri (operatörler: +, -, *, /, ^ ve parantezler: (, )) temsil eden `CSymbol` sınıfını ve bu semboller için tür tanımlayıcılarını (`ST_*`) ve karakter karşılıklarını (`SY_*`) tanımlar.
*   **Kalıtım:** `CBase` sınıfından türetilmiştir.
*   **Temel Özellikler/İçerik (`Symbol.h`):**
    *   **Sembol Türü Sabitleri (`ST_*`):**
        *   `ST_UNKNOWN`: Bilinmeyen sembol.
        *   `ST_PLUS`, `ST_MINUS`: Toplama, çıkarma.
        *   `ST_MULTIPLY`, `ST_DIVIDE`: Çarpma, bölme.
        *   `ST_CARET`: Üs alma (caret sembolü: ^).
        *   `ST_OPEN`, `ST_CLOSE`: Açma ve kapama parantezleri.
        *   **Not:** Bu sabitlerin değerleri (11, 12, 23, 24, 35, 06, 07) işlem önceliğini (veya gruplamayı) temsil ediyor olabilir. Onlar basamağı öncelik seviyesini gösteriyor gibi durmaktadır (örneğin, 1x toplama/çıkarma, 2x çarpma/bölme, 3x üs alma).
    *   **Sembol Karakter Sabitleri (`SY_*`):** Sembollerin karakter karşılıklarını tanımlar.
    *   **`CSymbol` Sınıfı:**
        *   `iType (int)`: Sembolün türünü (`ST_*` sabitlerinden birini) tutar.
        *   `CSymbol()`: Kurucu metod, `CBase::id`'yi `MID_SYMBOL` olarak ve `iType`'ı `ST_UNKNOWN` olarak ayarlar.
        *   `virtual ~CSymbol()`: Sanal yıkıcı metod.
        *   `static int issymbol(int ch)`: Verilen bir karakterin (`ch`) tanımlı sembollerden biri olup olmadığını kontrol eder. Eğer bir sembol ise, ilgili `ST_*` tür sabitini döndürür, değilse 0 döndürür.
        *   `SetType(int Type)`: Sembolün türünü (`iType`) ayarlar.
        *   `GetType()`: Sembolün türünü (`iType`) döndürür.
        *   `Equal(CSymbol dif)`: İki sembolün işlem önceliği seviyesinin (onlar basamağına bakarak) eşit olup olmadığını kontrol eder.
        *   `Less(CSymbol dif)`: Mevcut sembolün işlem önceliği seviyesinin, karşılaştırılan `dif` sembolünün seviyesinden daha düşük olup olmadığını kontrol eder.
*   **Implementasyon Detayları (`Symbol.cpp`):**
    *   `CSymbol` sınıfının kurucu, yıkıcı ve getter/setter metodlarının implementasyonlarını içerir.
    *   `Equal` ve `Less` metodları, `iType` değerinin 10'a bölümünü kullanarak işlem önceliği seviyeslerini karşılaştırır.
    *   `issymbol` metodu, gelen karakteri `SY_*` sabitleriyle karşılaştırarak ilgili `ST_*` türünü döndüren bir `switch` ifadesi içerir.
*   **Kullanım Alanı:** Bu sınıf, `Poly` kütüphanesi tarafından matematiksel ifadeler ayrıştırılırken (parsing) kullanılır. Bir karakter dizisi işlenirken, `issymbol` fonksiyonu ile bir karakterin operatör veya parantez olup olmadığı belirlenir. Eğer öyleyse, bir `CSymbol` nesnesi oluşturulur ve türü (`ST_*`) atanır. `GetType`, `Equal` ve `Less` metodları, ifadenin değerlendirilmesi sırasında (örneğin, işlem önceliğine göre Shunting-yard algoritması gibi bir algoritmada) operatörlerin önceliklerini karşılaştırmak için kullanılır. 

### `Poly/SymTable.h` ve `Poly/SymTable.cpp`

*   **Amaç:** `CSymTable` sınıfı, bir sembol tablosu (symbol table) girdisini temsil eder. Sembol tablosu, bir ayrıştırıcı (parser) veya derleyici tarafından karşılaşılan sembollerin (değişkenler, fonksiyon adları, sabitler vb.) bilgilerini depolamak için kullanılan bir veri yapısıdır.
*   **Temel Özellikler/İçerik (`SymTable.h`):**
    *   **`CSymTable` Sınıfı:**
        *   `dVal (double)`: Genellikle değişkenin veya sabitin sayısal değerini tutar.
        *   `token (int)`: Sembolün türünü veya ayrıştırıcı tarafından atanan benzersiz bir kimliği temsil eden bir tamsayı. Bu, `Symbol.h`'daki `ST_*` sabitleri veya `Base.h`'daki `MID_*` sabitleri olabilir.
        *   `strlex (std::string)`: Sembolün metinsel temsilini (lexeme), yani değişkenin adını veya sembolün kendisini (örneğin, "+") tutar.
        *   `CSymTable(int aTok, std::string aStr)`: Kurucu metod, `token` ve `strlex` değerlerini parametre olarak alır ve `dVal`'i 0 olarak başlatır.
        *   `virtual ~CSymTable()`: Yıkıcı metod.
*   **Implementasyon Detayları (`SymTable.cpp`):**
    *   `CSymTable` sınıfının kurucu ve yıkıcı metodlarının basit implementasyonlarını içerir. Kurucu, gelen parametreleri kullanarak ilgili üye değişkenlerini başlatır.
*   **Kullanım Alanı:** `Poly` kütüphanesi bir matematiksel ifadeyi ayrıştırırken, karşılaşığı her değişken veya sembol için bir `CSymTable` nesnesi oluşturabilir. Bu nesneler, muhtemelen bir `std::map` veya benzeri bir veri yapısında saklanarak bir sembol tablosu oluşturulur. Bu tablo, ifadenin değerlendirilmesi sırasında kullanılır:
    *   Bir değişken adı (`strlex`) ile karşılaşıldığında, tablodan ilgili `CSymTable` nesnesi bulunur ve `dVal` üyesindeki değeri kullanılır.
    *   Değişkenlere değer atamak için de bu tablo kullanılabilir (`dVal` güncellenir).
    *   `token` üyesi, sembolün türünü (operatör, değişken, sayı) hızlıca belirlemek için kullanılabilir.
    *   İstemci tarafında, örneğin bir görev script'inde `get_quest_status("görev_adı") > 0` gibi bir ifadenin değerlendirilmesinde, "görev_adı" gibi değişkenlerin veya `get_quest_status` gibi fonksiyonların bilgilerini saklamak için kullanılabilir. 

### `Poly/Poly.h` ve `Poly/Poly.cpp`

*   **Amaç:** `CPoly` sınıfı, metin tabanlı matematiksel ifadeleri (polinomlar ve daha genel aritmetik/fonksiyonel ifadeler) ayrıştırmak (parse) ve değerlendirmek (evaluate) için çekirdek işlevselliği sağlar.
*   **Temel Özellikler/İçerik (`Poly.h`):**
    *   **Token Türleri (`POLY_*` Makroları):** Ayrıştırıcı (parser) ve değerlendirici (evaluator) tarafından kullanılan token (işaret) türlerini tanımlar. Bunlar arasında temel aritmetik operatörler (`POLY_MUL`, `POLY_PLU`, `POLY_MIN`, `POLY_DIV`, `POLY_POW`, `POLY_MOD`), parantezler (`POLY_OPEN`, `POLY_CLOSE`), sayılar (`POLY_NUM`), değişkenler/tanımlayıcılar (`POLY_ID`), çeşitli matematiksel fonksiyonlar (`POLY_ROOT`, `POLY_COS`, `POLY_SIN`, `POLY_TAN`, `POLY_CSC`, `POLY_SEC`, `POLY_COT`, `POLY_LN`, `POLY_LOG`, `POLY_LOG10`, `POLY_ABS`, `POLY_FLOOR`), rastgele sayı fonksiyonları (`POLY_IRAND`, `POLY_FRAND`), min/max (`POLY_MINF`, `POLY_MAXF`) ve özel sabitler (`POLY_PI`, `POLY_EXP` - bunlar `POLY_ID` ile aynı değere sahip) bulunur.
    *   **`CPoly` Sınıfı:**
        *   `Analyze(const char* pszStr = NULL)`: Verilen string ifadeyi (`pszStr`) veya `SetStr` ile daha önce ayarlanan ifadeyi ayrıştırır. İfadeyi, `Eval()` tarafından değerlendirilebilecek bir iç temsil (muhtemelen ters Polonya notasyonu veya benzeri bir yığın tabanlı yapı) haline getirir. Başarılı olursa 0 döndürür, hata olursa hata pozisyonunu (`iErrorPos`) döndürür.
        *   `Eval()`: `Analyze` tarafından oluşturulan iç temsilin sonucunu hesaplar ve `float` olarak döndürür. Hata oluşmuşsa 0 döndürür.
        *   `SetStr(const std::string& str)`: Ayrıştırılacak ifadeyi ayarlar.
        *   `SetVar(const std::string& strName, double dVar)`: İfade içinde kullanılacak bir değişkenin (`strName`) değerini (`dVar`) ayarlar veya günceller. Sembol tablosuna (`lSymbol`) ekler.
        *   `GetVarCount()`: Tanımlanmış değişken sayısını döndürür.
        *   `GetVarName(unsigned int dwIndex)`: Belirli bir indeksteki değişkenin adını döndürür.
        *   `Clear()`: Sınıfın iç durumunu (sembol tablosu, token listesi vb.) temizler.
        *   `SetRandom(int iRandomType)`: `POLY_IRAND` ve `POLY_FRAND` fonksiyonlarının davranışını kontrol eder (`ERandomType` enum: `RANDOM_TYPE_FREELY`, `RANDOM_TYPE_FORCE_MIN`, `RANDOM_TYPE_FORCE_MAX`).
        *   **Korumalı Üyeler:**
            *   `tokenBase (std::vector<int>)`: Ayrıştırma sonucu oluşan token dizisi (postfix/RPN sırası olabilir).
            *   `numBase (std::vector<double>)`: `tokenBase`'deki `POLY_NUM` tokenlarına karşılık gelen sayısal değerler.
            *   `lSymbol (std::vector<CSymTable*>)`: Değişkenleri ve önceden tanımlanmış fonksiyonları/sabitleri tutan sembol tablosu.
            *   `SymbolIndex (std::vector<int>)`: `tokenBase`'deki `POLY_ID` tokenlarının `lSymbol`'daki indekslerini tutar.
            *   `strData (std::string)`: Ayrıştırılacak olan ham ifade.
            *   `iLookAhead`, `uiLookPos`, `iToken`, `iNumToken`: Ayrıştırıcı (lexical analyzer - `lexan`) tarafından kullanılan durum değişkenleri.
            *   `expr()`, `term()`, `factor()`, `expo()`: İfadeyi ayrıştırmak için kullanılan özyinelemeli iniş (recursive descent) parser fonksiyonları.
            *   `emit(int t, int tval)`: Ayrıştırılan tokenları `tokenBase`, `numBase`, `SymbolIndex` vektörlerine ekler.
            *   `match(int t)`: Beklenen token ile mevcut token'ı karşılaştırır.
            *   `error()`: Ayrıştırma hatası durumunu yönetir.
            *   `insert()`, `find()`: Sembol tablosu (`lSymbol`) yönetimi için fonksiyonlar.
            *   `my_irandom()`, `my_frandom()`: `SetRandom` ile ayarlanan türe göre rastgele sayı üreten yardımcı fonksiyonlar.
*   **Implementasyon Detayları (`Poly.cpp`):**
    *   **Ayrıştırıcı (`Analyze`, `lexan`, `expr`, `term`, `factor`, `expo`, `match`, `error`)**: Klasik bir özyinelemeli iniş ayrıştırıcısı uygular. `lexan` fonksiyonu ifadeyi karakter karakter okuyarak tokenlara (sayılar, değişkenler, operatörler, fonksiyonlar) ayırır. `expr`, `term`, `factor`, `expo` fonksiyonları işlem önceliğine göre ifadeyi ayrıştırır ve `emit` fonksiyonu aracılığıyla postfix (veya benzeri bir yığın tabanlı) temsili oluşturur.
    *   **Değerlendirici (`Eval`)**: `tokenBase`, `numBase` ve `SymbolIndex` (sembol tablosu değerleri için) vektörlerini kullanarak postfix ifadeyi bir yığın (`save` dizisi) üzerinde değerlendirir. Her operatör veya fonksiyon tokenı ile karşılaştığında, yığından gerekli sayıda argümanı alır, işlemi gerçekleştirir ve sonucu tekrar yığına koyar. Hata kontrolleri (sıfıra bölme, tanımsız logaritma vb.) içerir.
    *   **Sembol Tablosu Yönetimi (`init`, `insert`, `find`, `SetVar`, `GetVarName`, `GetVarCount`)**: `init` fonksiyonu, `POLY_PI`, `POLY_EXP` gibi önceden tanımlanmış sabitleri ve `POLY_COS`, `POLY_LN`, `POLY_IRAND` gibi fonksiyonları temsil eden tokenları `lSymbol` sembol tablosuna ekler. `insert` yeni değişkenleri ekler, `find` mevcut değişkenleri bulur, `SetVar` dışarıdan değişken değeri ayarlamayı sağlar.
    *   **Rastgele Sayı Üretimi (`my_irandom`, `my_frandom`, `SetRandom`)**: Belirlenen moda göre (`FREELY`, `FORCE_MIN`, `FORCE_MAX`) rastgele tamsayı veya ondalıklı sayı üretir.
*   **Kullanım Alanı:** Bu sınıf, istemci kodunda veya scriptlerde metin olarak tanımlanmış matematiksel formüllerin dinamik olarak hesaplanması gereken durumlarda kullanılır.
    *   **Karakter Stat Hesaplamaları:** Karakterin seviyesine, ekipmanına veya diğer değişkenlere bağlı karmaşık stat formüllerinin hesaplanması.
    *   **Hasar Formülleri:** Yeteneklerin veya saldırıların hasarını, rakibin savunması, karakterin saldırı gücü gibi değişkenlere göre hesaplamak.
    *   **Script Tabanlı Değerler:** Oyun içi görevlerde (quest) veya yapay zeka davranışlarında, belirli koşullara veya rastgeleliğe bağlı değerlerin hesaplanması (örneğin, `irand(1, 100) <= 50` gibi bir kontrol).
    *   **Dinamik Parametreler:** Efektlerin veya animasyonların bazı parametrelerinin (örneğin, süre, boyut) formüllerle belirlenmesi.
    Kullanım şekli genellikle şöyledir: Bir `CPoly` nesnesi oluşturulur, `SetVar` ile gerekli değişkenlerin değerleri atanır, `Analyze` ile formül string'i ayrıştırılır ve son olarak `Eval` ile sonuç hesaplanır. 

### `CPostIt.h` ve `CPostIt.cpp`

*   **Amaç:** `CPostIt` sınıfı, Windows panosunu (clipboard) kullanarak "anahtar=değer" çiftleri şeklinde basit metin tabanlı verileri depolamak ve potansiyel olarak uygulamalar arasında veya aynı uygulamanın farklı örnekleri arasında paylaşmak için bir mekanizma sunar. "Post-it notu" gibi geçici ve basit veri alışverişi için tasarlanmıştır.
*   **Temel Özellikler/İçerik:**
    *   **Özel Pano Formatı:** Her `CPostIt` örneği, kurucusuna verilen bir "uygulama adı" (`szAppName`) ile birleştirilmiş özel bir pano format adı (varsayılan olarak "YMCF_" veya "YMCF" + `szAppName`) kullanır. Bu, farklı uygulamaların veya aynı uygulamanın farklı `CPostIt` kullanımlarının birbirlerinin verilerini ezmesini engeller.
    *   **`_CPostItMemoryBlock` İç Sınıfı:**
        *   Verilerin asıl depolandığı ve yönetildiği yerdir.
        *   Panoya yazılacak veya panodan okunacak veriyi serileştirir ve deserileştirir.
        *   Veriler, "ANAHTAR=DEĞER" formatında string'ler olarak bir `std::list<CHAR*>` içinde tutulur.
        *   `Assign(HANDLE hBlock)`: Panodan okunan ham bellek bloğunu ayrıştırarak string listesini doldurur.
        *   `CreateHandle()`: String listesindeki veriyi panoya uygun tek bir bellek bloğuna dönüştürür. Format: `[Toplam String Sayısı (DWORD)][String1 Uzunluğu (WORD)][String1 Verisi][String2 Uzunluğu (WORD)][String2 Verisi]...`
        *   `Find(LPCSTR lpszKeyName)`: Belirli bir anahtara ait "ANAHTAR=DEĞER" string'ini döndürür.
        *   `Put(...)`: Yeni anahtar/değer çiftleri ekler veya mevcutları günceller.
        *   `Get(LPCSTR lpszKeyName, ...)`: Belirli bir anahtarın değerini okur.
    *   **`CPostIt` Sınıf Metotları:**
        *   `CPostIt(LPCSTR szAppName)`: Kurucu, pano format adını ayarlar.
        *   `Flush()`: Eğer veri değiştirilmişse (`m_bModified == true`), `_CPostItMemoryBlock` içindeki mevcut veriyi panoya yazar (`SetClipboardData`).
        *   `Empty()`: İlgili özel formattaki veriyi panodan temizler (`SetClipboardData` ile NULL handle göndererek) ve yerel bellek bloğunu siler.
        *   `Set(LPCSTR lpszKeyName, LPCSTR lpszData)`, `Set(LPCSTR lpszDataFormat)`, `Set(LPCSTR lpszKeyName, DWORD dwValue)`: Verilen anahtar ve değeri (veya "ANAHTAR=DEĞER" formatındaki string'i) yerel bellek bloğuna (`m_pMemoryBlock`) ekler/günceller. `m_bModified` bayrağını `true` yapar.
        *   `Get(LPCSTR lpszKeyName, LPSTR lpszData, DWORD nSize)`: İstenen anahtarın değerini `lpszData` tamponuna yazar. Eğer yerel bellek bloğu (`m_pMemoryBlock`) henüz oluşturulmamışsa veya boşsa, önce panodan (`GetClipboardData`) ilgili formatta veri okumayı dener ve `_CPostItMemoryBlock::Assign` ile yükler.
        *   `CopyTo(CPostIt* pPostIt, LPCSTR lpszKeyName)`: Mevcut `CPostIt` nesnesinden belirtilen bir anahtarın değerini, hedef `pPostIt` nesnesine `Set` metoduyla kopyalar.
        *   `Destroy()`: Yıkıcı çağrıldığında veya manuel olarak çağrıldığında, önce bekleyen değişiklikleri `Flush()` ile panoya yazar, sonra yerel bellek bloğunu temizler.
*   **Kullanım Alanı:**
    *   **Uygulamalar Arası Basit İletişim:** Aynı makinede çalışan farklı Metin2 istemci örnekleri arasında veya istemci ile harici bir hata ayıklama/yardımcı araç arasında çok basit, geçici verilerin (küçük ayarlar, durum bilgileri, komutlar) paylaşılması için kullanılabilir. Örneğin, bir istemci çökmeden hemen önce bazı kritik olmayan hata ayıklama bilgilerini panoya "post-it"leyebilir ve bu bilgi daha sonra başka bir araç veya aynı uygulamanın bir sonraki çalıştırılışı tarafından okunabilir.
    *   **Geçici Durum Saklama:** Kısa süreli, kalıcı olması gerekmeyen bilgileri (örneğin, bir oturumdaki bazı kullanıcı tercihleri veya son yapılan bir işlemle ilgili bir not) saklamak için kullanılabilir.
    *   **Hata Ayıklama:** Geliştirme sırasında, belirli modüllerin durumunu veya değişken değerlerini hızlıca panoya yazdırıp başka bir yerde (örn: metin editörü) incelemek için kullanılabilir.
    *   **Not:** Pano tabanlı olduğu için, bu mekanizma kalıcı depolama için uygun değildir. Panonun içeriği başka bir uygulama tarafından üzerine yazılabilir veya sistem kapatıldığında kaybolabilir. Korece yorumlardaki "클라이언트와 통신" (istemci ile iletişim) ve "메모 데이터를 클립보드에 저장" (bellek verisini panoya kaydet) ifadeleri bu tür bir kullanımı desteklemektedir.

### `error.h` ve `error.cpp`

*   **Amaç:** Bu dosyalar, Metin2 istemcisi için özel bir işlenmemiş istisna (unhandled exception) yöneticisi kurar. Program beklenmedik bir şekilde çöktüğünde (örneğin, erişim ihlali, sıfıra bölme gibi kritik hatalar), bu yönetici devreye girerek hata ayıklama bilgilerini bir dosyaya kaydeder ve potansiyel olarak harici bir hata raporlama aracını tetikler.
*   **Temel Özellikler/İçerik (`error.h`):**
    *   `extern void SetEterExceptionHandler();`: Bu fonksiyon, özel istisna yöneticisini (`EterExceptionFilter`) kurmak için `SetUnhandledExceptionFilter` Windows API fonksiyonunu çağırır.
*   **Implementasyon Detayları (`error.cpp`):
    *   **`SetEterExceptionHandler()` Fonksiyonu:**
        *   Windows API fonksiyonu olan `SetUnhandledExceptionFilter()` çağrısını yaparak, programda işlenmeyen bir istisna oluştuğunda `EterExceptionFilter` fonksiyonunun çağrılmasını sağlar.
    *   **`EterExceptionFilter(_EXCEPTION_POINTERS* pExceptionInfo)` Fonksiyonu:**
        *   Bu, asıl istisna işleme mantığını içerir. Bir istisna yakalandığında aşağıdaki adımları izler:
            1.  **Log Dosyası Oluşturma:** `ErrorLog.txt` adında bir dosya oluşturur (veya mevcutsa üzerine yazar).
            2.  **Hata Bilgilerini Yazma:** Bu dosyaya çeşitli kritik hata ayıklama bilgileri yazar:
                *   Çalıştırılan ana modülün (genellikle `.exe` dosyası) adı ve zaman damgası (`GetModuleFileName`, `GetTimestampForLoadedLibrary`).
                *   İstisnanın türünü belirten kod (`pExceptionInfo->ExceptionRecord->ExceptionCode`).
                *   İstisna anındaki işlemci register'larının (EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP) değerleri (`pExceptionInfo->ContextRecord`).
                *   Çağrı yığını (Call Stack): `StackWalk` Windows API fonksiyonu kullanılarak çağrı yığınındaki fonksiyon adresleri listelenir. Her adres için, o adresin hangi yüklü modüle (DLL veya EXE) ait olduğunu bulmak amacıyla `EnumerateLoadedModules` ve özel bir callback olan `EnumerateLoadedModulesProc` kullanılır.
            3.  **Harici Hata Raporlama Aracı:** `WinExec("errorlog.exe", SW_SHOW);` komutu ile `errorlog.exe` adında harici bir programı çalıştırır. Bu programın görevi, oluşturulan `ErrorLog.txt` dosyasını okumak, kullanıcıya daha anlaşılır bir hata mesajı göstermek ve/veya hata raporunu geliştiricilere göndermek olabilir.
            4.  **Programı Sonlandırma:** Fonksiyon, `EXCEPTION_EXECUTE_HANDLER` değeri döndürür. Bu, Windows'a istisnanın "işlendiğini" ve programın (genellikle bir hata mesajı gösterildikten sonra) güvenli bir şekilde sonlandırılması gerektiğini bildirir.
        *   **Not:** Kaynak kod içinde, geçmişte hata bilgilerinin sıkıştırılıp (`CLZObject`) bir sunucuya gönderilmeye çalışıldığına dair yorum satırları bulunmaktadır, ancak bu özellik mevcut sürümde aktif görünmemektedir.
*   **Kullanım Alanı:** Bu özel istisna yöneticisi, istemcinin beklenmedik çökme durumlarında devreye girer. Temel amacı, geliştiricilerin çökmelerin nedenlerini analiz edebilmesi için mümkün olduğunca fazla teknik bilgi toplamak ve kaydetmektir (`ErrorLog.txt`). Ayrıca, `errorlog.exe` aracılığıyla kullanıcıya bir çökme bildirimi sunar ve potansiyel olarak otomatik bir hata raporlama mekanizmasını tetikler. Bu, istemcinin genel kararlılığını artırmaya ve tespit edilen hataların daha hızlı çözülmesine yardımcı olur.

### `FileBase.h` ve `FileBase.cpp`

*   **Amaç:** `CFileBase` sınıfı, temel dosya G/Ç (Giriş/Çıkış) işlemleri için Windows API fonksiyonlarını sarmalayan (wrapper) bir temel sınıf sunar. Dosya oluşturma, okuma, yazma, boyut alma ve dosya içinde konumlanma (seek) gibi standart dosya operasyonları için daha basit ve nesne yönelimli bir arayüz sağlar.
*   **Temel Özellikler/İçerik (`FileBase.h`):**
    *   **`EFileMode` Enum:** Dosyanın açılma modunu tanımlar:
        *   `FILEMODE_READ`: Sadece okuma amaçlı açar.
        *   `FILEMODE_WRITE`: Okuma ve yazma amaçlı açar (yoksa oluşturur).
    *   **Ana Metotlar:**
        *   `Create(const char* filename, EFileMode mode)`: Belirtilen dosya adını ve modu kullanarak bir dosya açar veya oluşturur.
        *   `Close()`: Açık olan dosyayı kapatır.
        *   `Destroy()`: Dosyayı kapatır ve boyut bilgisini sıfırlar.
        *   `Size()`: Dosyanın boyutunu (byte cinsinden) döndürür.
        *   `SeekCur(DWORD size)`: Dosya işaretçisini mevcut konumundan belirtilen `size` kadar ileri taşır.
        *   `Seek(DWORD offset)`: Dosya işaretçisini dosyanın başından itibaren belirtilen `offset` konumuna taşır.
        *   `GetPosition()`: Dosya işaretçisinin mevcut konumunu döndürür.
        *   `Write(const void* src, int bytes)` (sanal): Verilen `src` tamponundaki `bytes` kadar veriyi dosyaya yazar.
        *   `Read(void* dest, int bytes)`: Dosyadan `bytes` kadar veriyi `dest` tamponuna okur.
        *   `GetFileName()`: Dosyanın adını döndürür.
        *   `IsNull()`: Dosya tanıtıcısının (`m_hFile`) geçerli olup olmadığını (NULL olup olmadığını) kontrol eder.
    *   **Korumalı Üyeler:**
        *   `m_mode (int)`: Dosyanın açılma modu (`EFileMode`).
        *   `m_filename (char[MAX_PATH + 1])`: Dosyanın adı.
        *   `m_hFile (HANDLE)`: Windows API dosya tanıtıcısı.
        *   `m_dwSize (DWORD)`: Dosyanın boyutu.
*   **Implementasyon Detayları (`FileBase.cpp`):
    *   **Dosya Açma/Oluşturma (`Create`)**: Windows API `CreateFile` fonksiyonunu kullanır.
        *   Okuma modu (`FILEMODE_READ`) için `OPEN_EXISTING` (dosya varsa açılır, yoksa hata verir).
        *   Yazma modu (`FILEMODE_WRITE`) için `OPEN_ALWAYS` (dosya varsa açılır, yoksa oluşturulur).
        *   Erişim hakları (`dwMode`) ve paylaşım modları (`dwShareMode`) açılma moduna göre ayarlanır. Yazma modunda hem okuma hem yazma erişimi ve paylaşımı istenir.
        *   Başarılı olursa, `GetFileSize` ile dosya boyutunu alıp `m_dwSize` üyesinde saklar.
    *   **Kapatma (`Close`, `Destroy`)**: `CloseHandle` API fonksiyonu ile dosyayı kapatır.
    *   **Konumlandırma (`SeekCur`, `Seek`, `GetPosition`)**: `SetFilePointer` API fonksiyonunu kullanır.
    *   **Okuma/Yazma (`Read`, `Write`)**: `ReadFile` ve `WriteFile` API fonksiyonlarını kullanır. `Write` işleminden sonra dosya boyutunu günceller.
*   **Kullanım Alanı:** `CFileBase` sınıfı, istemci kod tabanında standart dosya işlemleri için bir temel oluşturur. Doğrudan kullanılabilir veya daha özelleşmiş dosya işleme sınıfları (örneğin, yapılandırılmış veri dosyaları, log dosyaları, paketlenmiş varlık dosyalarıyla çalışan sınıflar) için bir ebeveyn sınıf olarak görev yapabilir. Oyun varlıklarının diskten yüklenmesi, kullanıcı ayarlarının veya oyun ilerlemesinin kaydedilmesi/yüklenmesi, çeşitli logların tutulması gibi temel dosya G/Ç operasyonlarının gerçekleştirildiği her yerde bu veya bundan türetilmiş sınıflar kullanılabilir.

### `FileDir.h` ve `FileDir.cpp`

*   **Amaç:** `CDir` sınıfı, bir dizin (klasör) içindeki dosyaları ve alt klasörleri listelemek ve bunlar üzerinde işlem yapmak için soyut bir temel sınıf sunar. Windows API'sinin `FindFirstFile` ve `FindNextFile` fonksiyonlarını kullanarak bir dizin yapısında gezinir.
*   **Tasarım:**
    *   Bu sınıf soyuttur (abstract) çünkü iki adet saf sanal (pure virtual) metot içerir: `OnFolder` ve `OnFile`. Bu, `CDir`'in doğrudan örneklenemeyeceği, bunun yerine bu iki metodu implemente eden türetilmiş bir sınıf oluşturulması gerektiği anlamına gelir. Bu tasarım, "Visitor" desenine benzer bir yaklaşım sunar; `CDir` gezinme mantığını sağlar, türetilmiş sınıf ise bulunan her öğe için ne yapılacağını tanımlar.
*   **Temel Özellikler/İçerik (`FileDir.h`):**
    *   **Ana Metotlar:**
        *   `Create(const char* c_szFilter, const char* c_szPath = "", BOOL bCheckedExtension = false)`: Belirtilen `c_szPath` dizininde, `c_szFilter` ve `bCheckedExtension` koşullarına uyan dosyaları ve klasörleri bulma işlemini başlatır. Dizin içeriğini tarar ve bulunan her öğe için `OnFolder` veya `OnFile` metodunu çağırır.
        *   `Destroy()`: Mevcut arama işlemini sonlandırır, kullanılan Windows tanıtıcısını (`m_hFind`) kapatır ve sınıfı başlangıç durumuna getirir.
    *   **Saf Sanal Metotlar (Türetilmiş Sınıf Tarafından Uygulanmalı):**
        *   `virtual bool OnFolder(const char* c_szFilter, const char* c_szPath, const char* c_szName) = 0;`: Dizin taraması sırasında bir alt klasör (`c_szName`) bulunduğunda çağrılır. `c_szPath` ana dizini, `c_szFilter` ise `Create` metoduna verilen orijinal filtreyi içerir. Eğer bu metot `false` dönerse, tüm tarama işlemi durdurulur.
        *   `virtual bool OnFile(const char* c_szPath, const char* c_szName) = 0;`: Dizin taraması sırasında bir dosya (`c_szName`) bulunduğunda çağrılır. `c_szPath` ana dizini içerir. Eğer bu metot `false` dönerse, tüm tarama işlemi durdurulur.
    *   **Yardımcı Metot:**
        *   `IsFolder()`: `m_wfd` (WIN32_FIND_DATA) içindeki bilgilere bakarak mevcut bulunan öğenin bir klasör olup olmadığını döndürür.
    *   **Korumalı Üyeler:**
        *   `m_wfd (WIN32_FIND_DATA)`: Mevcut dosya/klasör hakkındaki bilgileri tutan Windows yapısı.
        *   `m_hFind (HANDLE)`: `FindFirstFile`/`FindNextFile` tarafından kullanılan arama tanıtıcısı.
*   **Implementasyon Detayları (`FileDir.cpp`):
    *   **Dizin Taraması (`Create`)**:
        1.  Belirtilen yolu (`c_szPath`) normalize eder (gerekirse sonuna `\` ekler).
        2.  Arama sorgusunu `yol\*.*` şeklinde oluşturarak o yoldaki tüm öğeleri hedef alır.
        3.  `FindFirstFile` ile ilk öğeyi bulur.
        4.  Bir `do-while` döngüsü ve `FindNextFile` ile sonraki tüm öğeleri tarar.
        5.  `.` ve `..` girdilerini atlar.
        6.  Bulunan her öğe için `IsFolder()` ile türünü belirler.
        7.  Eğer bir klasörse, `OnFolder` çağrılır.
        8.  Eğer bir dosyaysa:
            *   Dosya uzantısını alır.
            *   `bCheckedExtension` `true` ise, dosya uzantısını `c_szFilter` ile karşılaştırır. `c_szFilter` içinde `';'` varsa (örn: "wav;mp3"), birden fazla uzantıyı kontrol edebilir. Eşleşme yoksa dosya atlanır.
            *   Filtre kontrollerinden geçen dosyalar için `OnFile` çağrılır.
        9.  `OnFolder` veya `OnFile` metodlarından herhangi biri `false` döndürürse, tarama işlemi hemen sonlandırılır.
*   **Kullanım Alanı:** `CDir` sınıfı, bir dizin hiyerarşisinde yinelemeli veya tek seviyeli olarak gezinmek ve bulunan dosya/klasörler üzerinde özelleştirilmiş işlemler yapmak için bir çerçeve sunar.
    *   **Örnek Senaryolar:**
        *   Belirli türdeki dosyaları (örn: tüm `.gr2` model dosyaları, tüm `.dds` doku dosyaları) bir varlık klasöründen ve alt klasörlerinden bulup yüklemek.
        *   Bir dizindeki tüm dosyaları belirli bir kritere göre işlemek (örn: yedekleme, silme, analiz etme).
        *   Oyunun yama (patch) mekanizmasında, güncellenecek veya eklenecek dosyaları belirlemek için dizinleri taramak.
    *   Geliştiricinin, `CDir`'den kendi sınıfını türetmesi ve `OnFolder` ile `OnFile` metodlarını, yapmak istediği spesifik göreve (örn: bulunan dosyayı bir listeye eklemek, dosya içeriğini okumak, alt klasörde tekrar `Create` çağırmak) uygun şekilde doldurması gerekir.

### `Filename.h`

*   **Amaç:** Bu başlık dosyası, dosya yollarını ve adlarını temsil etmek, manipüle etmek ve parçalarına ayırmak için `CFilename` sınıfını ve statik yardımcı fonksiyonlar içeren `CFileNameHelper` sınıfını tanımlar. Bu, dosya yolu işlemlerini daha yönetilebilir ve tutarlı hale getirir.
*   **`CFilename` Sınıfı:**
    *   **Genel Bakış:** Temelde bir `std::string` (`m_sRaw`) sarmalayıcısıdır. Dosya yollarıyla çalışmayı kolaylaştırmak için özel metotlar ve operatör aşırı yüklemeleri sunar.
    *   **Operatörler:** `std::string` gibi kullanılabilmesi için atama (`=`), karşılaştırma (`==`), birleştirme (`+`, `+=`), indeksleme (`[]`) ve tür dönüşüm (`operator std::string&`, `operator const std::string`) operatörleri tanımlanmıştır.
    *   **`std::string` Uyumluluğu:** `c_str()`, `find()`, `empty()`, `size()`, `length()`, `GetString()` gibi `std::string` metotlarına benzer arayüzler sağlar.
    *   **Yol Manipülasyonu:**
        *   `ChangeDosPath()`: Yol ayıracı olarak kullanılan `/` karakterlerini `\` ile değiştirir.
        *   `StringPath()`: Yol ayıracı olarak kullanılan `\` karakterlerini `/` ile değiştirir ve tüm yolu küçük harfe çevirir. Bu, oyun içinde standart ve platformdan bağımsız bir yol formatı sağlamak için önemlidir.
    *   **Yol Parçalama Metotları:** (Implementasyonları muhtemelen `CFileNameHelper`'ı kullanır)
        *   `GetName()`: Dosya adını uzantısız alır (örn: `/d/y/foo.bar` -> `foo`).
        *   `GetExtension()`: Dosya uzantısını alır (örn: `/d/y/foo.bar` -> `bar`).
        *   `GetPath()`: Dosyanın bulunduğu yolu alır (örn: `/d/y/foo.bar` -> `/d/y/`).
        *   `NoExtension()`: Tam yolu uzantısız alır (örn: `/d/y/foo.bar` -> `/d/y/foo`).
        *   `NoPath()`: Yolu kaldırıp sadece dosya adı ve uzantısını alır (örn: `/d/y/foo.bar` -> `foo.bar`).
*   **`CFileNameHelper` Sınıfı:**
    *   **Genel Bakış:** `CFilename` içindeki temel yol işleme mantığını `std::string` üzerinde çalışan statik (`static`) fonksiyonlar olarak sunar.
    *   **Statik Metotlar:** `ChangeDosPath`, `StringPath`, `GetName`, `GetExtension`, `GetPath`, `NoExtension`, `NoPath` metotlarının `std::string` parametreleri alan statik versiyonlarını içerir. Bu metotların implementasyonları başlık dosyasında `inline` olarak verilmiştir.
*   **Kullanım Alanı:**
    *   Kod içinde dosya yollarıyla çalışırken daha okunabilir ve amaca yönelik bir tip (`CFilename`) kullanılmasını sağlar.
    *   Dosya yollarını standart bir formata (`StringPath` ile küçük harf ve `/` ayıracı) dönüştürmek için kullanılır. Bu, özellikle farklı kaynaklardan (scriptler, konfigürasyonlar) gelen veya farklı platformlarda kullanılan yolların tutarlı olmasını sağlamak açısından önemlidir.
    *   Dosya adlarından yolu, adı veya uzantıyı kolayca ayıklamak için kullanılır (örn: varlık yükleyicilerinde, dosya tipi kontrolünde).
    *   `CFilename` nesnesi oluşturmadan doğrudan `std::string` üzerinde hızlı yol işlemleri yapmak için `CFileNameHelper` kullanılabilir.

### `grid.h` ve `grid.cc`

*   **Amaç:** `CGrid` sınıfı, 2 boyutlu basit bir ızgara (grid) yapısını temsil eder ve yönetir. Hücrelerin dolu veya boş olduğunu takip etmek için kullanılır. Özellikle envanter gibi sistemlerde öğelerin yerleştirilip yerleştirilemeyeceğini kontrol etmek için idealdir.
*   **Temel Özellikler/İçerik (`grid.h`):**
    *   **Yapıcılar:**
        *   `CGrid(int w, int h)`: Belirtilen genişlik ve yükseklikte boş bir ızgara oluşturur.
        *   `CGrid(CGrid * pkGrid, int w, int h)`: Var olan bir `pkGrid` ızgarasının içeriğini, yeni `w`x`h` boyutlarına sığdığı ölçüde kopyalayarak yeni bir ızgara oluşturur.
    *   **Ana Metotlar:**
        *   `Clear()`: Izgaradaki tüm hücreleri boş olarak işaretler.
        *   `FindBlank(int w, int h)`: Izgarada belirtilen `w`x`h` boyutlarında tamamen boş olan ilk alanı bulur ve sol üst köşesinin tek boyutlu indeksini döndürür. Boş alan bulamazsa -1 döndürür.
        *   `IsEmpty(int iPos, int w, int h)`: `iPos` indeksinden başlayan `w`x`h` boyutundaki alanın tamamen boş olup olmadığını kontrol eder.
        *   `Put(int iPos, int w, int h)`: `iPos` indeksinden başlayan `w`x`h` boyutundaki alanı dolu olarak işaretler. İşlem ancak alan tamamen boşsa başarılı olur ve `true` döndürür.
        *   `Get(int iPos, int w, int h)`: `iPos` indeksinden başlayan `w`x`h` boyutundaki alanı boş olarak işaretler.
        *   `GetSize()`: Izgaranın toplam hücre sayısını (`genişlik * yükseklik`) döndürür.
        *   `Print()`: Izgaranın mevcut durumunu (0'lar ve 1'ler şeklinde) konsola yazdırır (hata ayıklama için).
    *   **Korumalı Üyeler:**
        *   `m_iWidth (int)`, `m_iHeight (int)`: Izgaranın boyutları.
        *   `m_pGrid (char *)`: Izgara verisini tutan dinamik dizi. `0` boş, `1` (veya `true`) dolu anlamına gelir.
*   **Implementasyon Detayları (`grid.cc`):
    *   Izgara verisi (`m_pGrid`) tek boyutlu bir `char` dizisi olarak tutulur. İki boyutlu indeksleme `satir * genişlik + sütun` formülüyle yapılır.
    *   `FindBlank`, ızgarayı soldan sağa, yukarıdan aşağıya tarayarak uygun boş alanı arar.
    *   `IsEmpty`, `Put`, `Get` metotları, belirtilen `w`x`h` alanı içindeki tüm hücrelere erişmek için iç içe döngüler veya eşdeğer mantık kullanır.
    *   Sınır kontrolleri (`IsEmpty` içinde) alanın ızgara dışına taşmamasını sağlar.
*   **Kullanım Alanı:** Bu sınıf, Metin2 istemcisindeki ızgara tabanlı sistemlerin temel mantığını oluşturur:
    *   **Envanter/Depo:** Oyuncunun envanter veya depo penceresindeki boş yerleri bulmak, eşyaların belirli bir alana sığıp sığmayacağını kontrol etmek, eşyaları yerleştirmek (`Put`) ve kaldırmak (`Get`) için kullanılır.
    *   **Hızlı Erişim Çubuğu (Quickslot):** Yetenek veya eşyaların yerleştirildiği slotların dolu/boş durumunu takip etmek için kullanılabilir.
    *   **Ticaret/Pazar Pencereleri:** Ticaret veya pazar penceresine konulan eşyaların yerleşimini yönetmek için benzer bir yapı kullanılabilir.
    `CGrid`, bu sistemlerde yerleşim mantığını soyutlayarak kodun daha temiz ve yönetilebilir olmasını sağlar.

### `MappedFile.h` ve `MappedFile.cpp`

*   **Amaç:** `CMappedFile` sınıfı, `CFileBase`'den türeyerek bellek eşlemli dosya (memory-mapped file) yönetimi ve bellek içi veri bloklarıyla dosya gibi çalışma yetenekleri sunar. Özellikle büyük dosyalara verimli erişim ve oyunun paketlenmiş varlık dosyalarıyla (.epk/.eix) entegrasyon için tasarlanmıştır. LZO sıkıştırmasıyla açılmış verilere de arayüz sağlayabilir.
*   **Temel Özellikler/İçerik (`MappedFile.h`):**
    *   **Kalıtım:** `CFileBase`
    *   **`ESeekType` Enum:** `Seek` işlemi için başlangıç noktası (`BEGIN`, `CURRENT`, `END`).
    *   **Dosya Açma/Eşleme:**
        *   `Create(const char* filename)`: Dosyayı okuma modunda açar (bellek eşlemesi için ön hazırlık).
        *   `Create(const char* filename, const void** dest, int offset, int size)`: Dosyanın belirtilen bir bölümünü doğrudan `dest` işaretçisine eşler.
        *   `Map(const void** dest, int offset = 0, int size = 0)`: Dosyanın bir bölümünü veya tamamını belleğe eşler.
    *   **Bellek İçi Veri Bağlama:**
        *   `Link(DWORD dwBufSize, const void* c_pvBufData)`: Harici bir bellek bloğunu (`c_pvBufData`) `CMappedFile` nesnesine bağlayarak dosya gibi okunmasını sağlar. Bu, paketlerden çıkarılan veriler için kritiktir.
    *   **Sıkıştırma Entegrasyonu:**
        *   `BindLZObject(CLZObject* pLZObj)`: Bir `CLZObject` (LZO sıkıştırma/açma nesnesi) bağlar ve LZO nesnesinin tamponundaki sıkıştırılmış veriyi `Link` eder. `CMappedFile` doğrudan açma yapmaz, ham sıkıştırılmış veriye erişim ve konumlandırma sağlar.
        *   `BindLZObjectWithBufferedSize(CLZObject* pLZObj)`: `BindLZObject`'e benzer ancak LZO nesnesinin toplam tampon boyutunu kullanır.
    *   **Veri Erişimi ve Konumlandırma:**
        *   `Get()`: Eşlenmiş verinin başlangıç adresini döndürür.
        *   `Read(void* dest, int bytes)`: Mevcut konumdan veri okur.
        *   `Seek(DWORD offset, int iSeekType)`: Okuma/yazma konumunu ayarlar.
        *   `Size()`: Bağlı olan veri kaynağının (eşlenmiş dosya bölümü veya link edilmiş tampon) boyutunu döndürür (genellikle ham/sıkıştırılmış boyut).
        *   `GetPosition()`: Dosya içindeki eşlenmiş verinin başlangıç ofsetini döndürür.
        *   `GetSeekPosition()`: Mevcut `Seek` ile ayarlanmış pozisyonu döndürür.
        *   `GetCurrentSeekPoint()`: Mevcut `Seek` pozisyonundaki veriye doğrudan bir işaretçi döndürür.
    *   **Veri Ekleme (Dinamik İçerik):**
        *   `AppendDataBlock(const void* pBlock, DWORD dwBlockSize)`: Mevcut bağlı veriye yeni bir veri bloğu ekler, yeni bir tampon oluşturur ve `Link` ile bu yeni tampona yönlendirir. (Daha çok sunucu/araç tarafı kullanımı için olabilir).
    *   **Kaynak Yönetimi:**
        *   `Destroy()`: Tüm kaynakları (dosya tanıtıcıları, eşlenmiş bellek, LZO nesnesi, dinamik tamponlar) serbest bırakır.
*   **Implementasyon Detayları (`MappedFile.cpp`):
    *   **Bellek Eşleme:** Windows API fonksiyonları `CreateFileMapping` ve `MapViewOfFile` kullanılır. Sadece okuma (`PAGE_READONLY`, `FILE_MAP_READ`) modunda eşleme yapar.
    *   **Veri Kaynağı Önceliği:** `CMappedFile` öncelikle `Link` ile bağlanmış bir bellek bloğu (`m_pbBufLinkData`) üzerinden çalışır. Eğer bir dosya `Map` edilmişse, eşlenen bellek bölgesi de `Link` edilir.
    *   **LZO Entegrasyonu:** `BindLZObject` çağrıldığında, LZO nesnesinin içindeki ham (sıkıştırılmış) veri tamponu `Link` edilir. `CMappedFile`, bu veriyi LZO formatında olduğunu bilir ancak açma işlemini kendisi yapmaz; bu, kullanıcı kodunun sorumluluğundadır. `CMappedFile` bu ham veriye `Seek` ve `Read` arayüzü sunar.
    *   **`Size()` Metodu:** Genellikle bağlı olan ham verinin (eşlenmiş bölümün veya link edilmiş tamponun sıkıştırılmış boyutu) boyutunu döndürür, açılmış boyutu değil.
*   **Kullanım Alanı:**
    *   **Paket Dosyası Erişimi (.epk/.eix):** Metin2'de en yaygın kullanım alanıdır. Bir `.epk` paketinden bir dosya (örneğin bir doku, model veya script) okunduğunda, bu dosyanın (muhtemelen LZO ile sıkıştırılmış) içeriği belleğe yüklenir. Bu bellek bloğu bir `CMappedFile` nesnesine `Link` edilir ve eğer sıkıştırılmışsa `BindLZObject` ile LZO nesnesi de bağlanır. Daha sonra bu `CMappedFile` üzerinden veri okunur (gerekirse LZO ile açılarak).
    *   **Büyük Dosyalara Verimli Erişim:** Doğrudan diskteki büyük dosyaların (eğer paketlenmemişse) tamamını belleğe yüklemeden, ihtiyaç duyulan kısımlarına bellek eşleme yoluyla erişmek için kullanılabilir.
    *   **Geçici Bellek Akışları:** Çeşitli işlemler sırasında oluşturulan geçici bellek içi verilere dosya benzeri bir arayüzle (seek, read) erişmek için kullanılabilir.
    `CMappedFile`, farklı veri kaynaklarını (disk dosyası, bellek bloğu, sıkıştırılmış veri) tek bir arayüz altında birleştirerek kod karmaşıklığını azaltır.
