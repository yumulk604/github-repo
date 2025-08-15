# ScriptLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `ScriptLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. 

## Dosya Bazlı Detaylandırma

### `PythonDebugModule.h` ve `PythonDebugModule.cpp` (Python `dbg` Modülü)

*   **Amaç:** Bu dosyalar, Python betiklerine C++ tarafındaki çeşitli hata ayıklama ve loglama araçlarını sunmak için `dbg` adında bir Python modülü oluşturur. Bu sayede Python kodu içerisinden doğrudan C++ loglama mekanizmalarına (`LogBox`, `Trace`, `TraceError` vb.) mesaj gönderilebilir ve özel hata mesajları kaydedilebilir.

*   **`PythonDebugModule.h` - Temel Bildirimler:**
    *   `void initdbg();`: `dbg` Python modülünü başlatan fonksiyonun bildirimini içerir. Bu fonksiyon, `Py_InitModule` çağrısı ile Python'a C++ fonksiyonlarını kaydeder.

*   **`PythonDebugModule.cpp` - Uygulama Detayları:**
    *   **Harici Değişken:**
        *   `extern IPythonExceptionSender* g_pkExceptionSender;`: Python tarafından fırlatılan istisnaları (exceptions) C++ tarafında yakalayıp işleyebilecek bir arayüze (muhtemelen istemcinin ana hata yönetim sisteminin bir parçası) işaret eden global bir gösterici.
    *   **Python'a Açılan Fonksiyonlar:**
        *   `PyObject* dbgLogBox(PyObject* poSelf, PyObject* poArgs)`:
            *   Python'dan çağrıldığında `LogBox()` C++ fonksiyonunu tetikler.
            *   Python argümanlarından bir mesaj (`szMsg`) ve isteğe bağlı bir başlık (`szCaption`) alır. Başlık verilmezse, sadece mesajla `LogBox` çağrılır.
            *   Python'a `Py_BuildNone()` (Python `None` nesnesi) döndürür. Hata durumunda Python istisnası oluşturur.
        *   `PyObject* dbgTrace(PyObject* poSelf, PyObject* poArgs)`:
            *   Python'dan `Trace()` C++ fonksiyonunu çağırır.
            *   Python argümanlarından bir mesaj (`szMsg`) alır.
            *   Python'a `Py_BuildNone()` döndürür.
        *   `PyObject* dbgTracen(PyObject* poSelf, PyObject* poArgs)`:
            *   Python'dan `Tracen()` C++ fonksiyonunu çağırır (genellikle satır sonu karakteri olmadan loglama yapar).
            *   Python argümanlarından bir mesaj (`szMsg`) alır.
            *   Python'a `Py_BuildNone()` döndürür.
        *   `PyObject* dbgTraceError(PyObject* poSelf, PyObject* poArgs)`:
            *   Python'dan `TraceError()` C++ fonksiyonunu çağırır.
            *   Python argümanlarından bir mesaj (`szMsg`) alır.
            *   Python'a `Py_BuildNone()` döndürür.
        *   `PyObject* dbgRegisterExceptionString(PyObject* poSelf, PyObject* poArgs)`:
            *   Python'dan alınan bir hata mesajını (`szMsg`) `g_pkExceptionSender` arayüzü üzerinden C++ tarafındaki hata toplama mekanizmasına kaydeder.
            *   Bu, Python'da oluşan bir hatanın C++ tarafında daha detaylı bir şekilde loglanmasını veya bir hata raporlama sistemine gönderilmesini sağlayabilir.
            *   Python'a `Py_BuildNone()` döndürür.
    *   **Modül Başlatma Fonksiyonu:**
        *   `void initdbg()`:
            *   `PyMethodDef s_methods[]` dizisi ile Python'a açılacak C++ fonksiyonlarını ve Python'da hangi isimlerle çağrılacaklarını tanımlar:
                *   `"LogBox"` -> `dbgLogBox`
                *   `"Trace"` -> `dbgTrace`
                *   `"Tracen"` -> `dbgTracen`
                *   `"TraceError"` -> `dbgTraceError`
                *   `"RegisterExceptionString"` -> `dbgRegisterExceptionString`
            *   `Py_InitModule("dbg", s_methods)` çağrısı ile `dbg` isimli Python modülünü bu metotlarla birlikte oluşturur ve Python yorumlayıcısına kaydeder.

*   **Kullanım Amacı:**
    *   Python betikleri geliştirilirken veya çalışırken hata ayıklama ve bilgi loglama işlemlerini kolaylaştırmak.
    *   Python tarafında oluşan önemli olayların veya hataların, istemcinin genel loglama ve hata raporlama sistemiyle entegre bir şekilde kaydedilmesini sağlamak.
    *   Örneğin, bir Python UI betiği, bir hata oluştuğunda `dbg.LogBox("UI Hatası: Buton bulunamadı", "Hata")` şeklinde bir mesaj kutusu gösterebilir veya `dbg.TraceError("Kritik UI hatası oluştu.")` ile log dosyasına yazabilir.

*   **Python Arayüzü Örnekleri:**
    ```python
    import dbg

    dbg.LogBox("Bu bir test mesajıdır.", "Test Başlığı")
    dbg.Trace("Bu bir trace mesajıdır.")
    dbg.Tracen("Bu da bir tracen mesajıdır,")
    dbg.Tracen(" aynı satırda devam eder.")
    dbg.TraceError("Bu bir hata mesajıdır.")
    dbg.RegisterExceptionString("Örnek bir Python istisna mesajı.")
    ```

*   **Bağımlılıklar:**
    *   `StdAfx.h` (ve dolayısıyla Python C API başlıkları - `Python.h`).
    *   `IPythonExceptionSender` arayüzü (harici olarak tanımlı).
    *   `EterBase` veya benzeri bir kütüphaneden gelen `LogBox`, `Trace`, `Tracen`, `TraceError` gibi loglama fonksiyonları.

### `PythonLauncher.h` ve `PythonLauncher.cpp` (`CPythonLauncher` Sınıfı)

*   **Amaç:** `CPythonLauncher` sınıfı, bir singleton olarak, C++ uygulaması içinden Python yorumlayıcısını yönetmek ve Python betiklerini çalıştırmak için merkezi bir arayüz sağlar. Python'u başlatır, ana Python modülünü ve sözlüğünü (dictionary) ayarlar, farklı kaynaklardan (doğrudan satır, dosya, bellek, derlenmiş .pyc dosyası) Python kodu çalıştırabilir ve Python hata ayıklama (tracing) ve hata raporlama mekanizmalarını destekler.

*   **`PythonLauncher.h` - Temel Tanımlar ve Bildirimler:**
    *   **Sınıf Tanımı:**
        *   `CPythonLauncher : public CSingleton<CPythonLauncher>`: `CPythonLauncher` sınıfı, `EterBase` kütüphanesinden gelen `CSingleton` şablonunu kullanarak singleton deseniyle tasarlanmıştır. Bu, uygulama boyunca yalnızca bir `CPythonLauncher` örneğinin olmasını sağlar.
    *   **Genel Metotlar:**
        *   `CPythonLauncher()`: Yapıcı metot, `Py_Initialize()` çağırarak Python yorumlayıcısını başlatır.
        *   `~CPythonLauncher()`: Yıkıcı metot, `Clear()`'ı çağırır.
        *   `Clear()`: `Py_Finalize()` çağırarak Python yorumlayıcısını sonlandırır ve kaynakları serbest bırakır.
        *   `Create(const char* c_szProgramName = "eter.python")`: Python program adını ayarlar, ana modülü (`__main__`) ve sözlüğünü oluşturur, izleme fonksiyonunu (debug modunda) ayarlar ve temel Python modüllerini (`__main__`, `sys`) import eder.
        *   `SetTraceFunc(int (*pFunc)(PyObject* obj, PyFrameObject* f, int what, PyObject* arg))`: Python yorumlayıcısı için özel bir izleme (trace) fonksiyonu ayarlanmasını sağlar. `PyEval_SetTrace` fonksiyonunu çağırır.
        *   `RunLine(const char* c_szLine)`: Verilen tek bir Python kod satırını çalıştırır.
        *   `RunFile(const char* c_szFileName)`: Belirtilen dosyadan Python betiğini yükler ve çalıştırır (`EterPack` üzerinden).
        *   `RunMemoryTextFile(const char* c_szFileName, UINT uFileSize, const VOID* c_pvFileData)`: Bellekteki bir metin tabanlı Python betiğini çalıştırır.
        *   `RunCompiledFile(const char* c_szFileName)`: Önceden derlenmiş bir Python dosyasını (`.pyc`) çalıştırır.
        *   `GetError()`: En son oluşan Python hatasının açıklamasını string olarak döndürür.
    *   **Korunan (Protected) Üyeler:**
        *   `m_poModule` (PyObject*): Python'un ana (`__main__`) modülüne işaretçi.
        *   `m_poDic` (PyObject*): Ana modülün sözlüğüne (global namespace) işaretçi.

*   **`PythonLauncher.cpp` - Uygulama Detayları:**
    *   **Global İzleme Tamponu:**
        *   `std::string g_stTraceBuffer[512]`: Python çağrı yığınını (call stack) izlemek için kullanılan bir string dizisi.
        *   `int	g_nCurTraceN`: `g_stTraceBuffer` içindeki mevcut izleme kaydı sayısını tutar.
    *   **`Traceback()` Fonksiyonu:**
        *   Bir Python hatası oluştuğunda çağrılır.
        *   `g_stTraceBuffer`'daki çağrı yığını bilgilerini ve `PyErr_Fetch()` ile alınan Python hata detaylarını birleştirir.
        *   Sonucu `LogBoxf` ile bir mesaj kutusunda gösterir ve ayrıca `Tracef` ile loglar.
    *   **`TraceFunc(PyObject* obj, PyFrameObject* f, int what, PyObject* arg)` Fonksiyonu:**
        *   Python yorumlayıcısı tarafından her fonksiyon çağrısı (`PyTrace_CALL`), geri dönüşü (`PyTrace_RETURN`) veya istisnası (`PyTrace_EXCEPTION`) sırasında çağrılan bir C callback fonksiyonudur.
        *   `PyTrace_CALL`: Fonksiyon adı, dosya adı ve satır numarasını `g_stTraceBuffer`'a ekler.
        *   `PyTrace_RETURN`: `g_nCurTraceN` sayacını azaltır.
        *   `PyTrace_EXCEPTION`: İstisna bilgilerini `g_stTraceBuffer`'a ekler.
        *   Bu fonksiyon, `#ifdef _DEBUG` koşuluyla `Create()` içinde `PyEval_SetTrace` ile ayarlanır.
    *   **`Create()` Metodu:**
        *   `Py_SetProgramName()` ile Python program adını ayarlar (varsayılan "eter.python").
        *   Debug modunda (`_DEBUG` tanımlıysa) `PyEval_SetTrace(TraceFunc, NULL)` ile yukarıda bahsedilen `TraceFunc`'ı Python izleme fonksiyonu olarak ayarlar.
        *   `PyImport_AddModule("__main__")` ile ana modülü alır/oluşturur ve `m_poModule`'e atar.
        *   `PyModule_GetDict(m_poModule)` ile ana modülün sözlüğünü alır ve `m_poDic`'e atar. Bu sözlük, betiklerin global değişkenlerini tutar.
        *   `__builtin__` modülünü import eder ve `TRUE` (1), `FALSE` (0) sabitlerini bu modüle ekler. Ardından `__builtin__` modülünü `m_poDic`'e `__builtins__` anahtarıyla ekler. Bu, Python betiklerinin `TRUE` ve `FALSE` gibi sabitlere erişmesini sağlar.
        *   `RunLine("import __main__")` ve `RunLine("import sys")` çağrılarıyla temel modüllerin ana sözlüğe yüklenmesini sağlar.
    *   **Betik Çalıştırma Metotları:**
        *   `RunLine(const char* c_szSrc)`: `PyRun_String()` fonksiyonunu kullanarak verilen C-string'ini Python kodu olarak çalıştırır. Hata oluşursa `Traceback()` çağrılır.
        *   `RunFile(const char* c_szFileName)`:
            *   `CEterPackManager::Instance().Get()` ile dosyayı `EterPack` paketlerinden veya diskten yükler.
            *   Dosya içeriğini `RunMemoryTextFile()`'a göndererek çalıştırır.
        *   `RunMemoryTextFile(const char* c_szFileName, UINT uFileSize, const VOID* c_pvFileData)`:
            *   Bellekteki veriden CR (carriage return, `\r`) karakterlerini temizler (sadece LF - line feed, `\n` bırakır).
            *   `Py_CompileString()` ile metin tabanlı kodu Python bytecode'una derler.
            *   `PyEval_EvalCode()` ile derlenmiş kodu `m_poDic` (global sözlük) bağlamında çalıştırır.
        *   `RunCompiledFile(const char* c_szFileName)`:
            *   `.pyc` (derlenmiş Python) dosyasını binary modda açar.
            *   `_PyMarshal_ReadLongFromFile()` ile dosyanın başındaki "magic number"ı okur ve `PyImport_GetMagicNumber()` ile Python yorumlayıcısının beklediği magic number ile karşılaştırır. Uyuşmazsa hata verir.
            *   Yine `_PyMarshal_ReadLongFromFile()` ile zaman damgasını okur (ama kullanmaz).
            *   `_PyMarshal_ReadLastObjectFromFile()` (muhtemelen `PythonMarshal.cpp`'den gelen özel bir fonksiyon) ile dosyadaki ana kod nesnesini (code object) okur.
            *   `PyEval_EvalCode()` ile bu kod nesnesini çalıştırır. Hata oluşursa `Traceback()` çağrılır.
    *   **`GetError()` Metodu:** `PyErr_Fetch()` ile mevcut Python hata bilgilerini alır ve eğer hata mesajı bir string ise onu döndürür.
    *   `NANOBEGIN` ve `NANOEND` makroları bazı fonksiyonların başında ve sonunda kullanılmıştır, bu genellikle kod koruma/obfuscation (karartma) teknikleriyle ilgilidir.

*   **Kullanım Amacı:**
    *   İstemcinin çeşitli bölümlerinde (örn: UI, görevler, özel sistemler) Python betiklerini çalıştırmak için birincil mekanizmadır.
    *   Python'un C++ uygulamasına gömülmesini (embedding) ve yönetilmesini sağlar.
    *   Geliştirme ve hata ayıklama süreçlerinde Python hatalarının daha kolay takip edilmesine yardımcı olur.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (ve dolayısıyla Python C API başlıkları - `Python.h`, `frameobject.h`).
    *   `../EterPack/EterPackManager.h`: Dosya yükleme işlemleri için.
    *   `../EterBase/Singleton.h`: `CSingleton` taban sınıfı için.
    *   `PythonMarshal.cpp`'den `_PyMarshal_ReadLongFromFile` ve `_PyMarshal_ReadLastObjectFromFile` fonksiyonları (bu dosyada bildirilmemiş olsalar da `RunCompiledFile` içinde kullanılıyorlar). 

### `PythonMarshal.h` ve `PythonMarshal.cpp` (Python Nesne De-serialization)

*   **Amaç:** Bu dosyalar, Python'un `marshal` formatında serileştirilmiş Python nesnelerini, özellikle derlenmiş Python kodunu (`.pyc` dosyaları), bir dosyadan veya bellek bloğundan okumak (de-serialize/unmarshal) için gerekli fonksiyonları sağlar. Bu, CPython'un dahili `marshal.c` dosyasından türetilmiş veya ona çok benzeyen bir C implementasyonudur ve temel olarak `.pyc` dosyalarının yüklenmesini destekler. Yazma (serileştirme) işlemleri bu dosyalarda bulunmamaktadır.

*   **`PythonMarshal.h` - Temel Bildirimler:**
    *   `PyObject* _PyMarshal_ReadObjectFromFile(FILE* fp)`: Verilen dosya akışından bir Python nesnesi okur.
    *   `PyObject* _PyMarshal_ReadLastObjectFromFile(FILE* fp)`: Bir dosya akışından *son* Python nesnesini okur. Küçük dosyalar için optimize edilmiş olabilir (dosyayı tek seferde belleğe alarak).
    *   `long _PyMarshal_ReadLongFromFile(FILE* fp)`: Dosya akışından 4 byte'lık bir long integer okur.

*   **`PythonMarshal.cpp` - Uygulama Detayları:**
    *   **Temel Sabitler (Tür Kodları):**
        *   `TYPE_NULL ('0')`, `TYPE_NONE ('N')`, `TYPE_INT ('i')`, `TYPE_STRING ('s')`, `TYPE_CODE ('c')` gibi Python nesne türlerini temsil eden tek karakterlik kodlar tanımlanmıştır. Bu kodlar, serileştirilmiş veri akışında hangi türde bir nesnenin takip ettiğini belirtir.
    *   **`RFILE` Yapısı:**
        *   Dosyadan (`FILE* fp`) veya bir bellek tamponundan (`char* ptr`, `char* end`) okuma yapmak için kullanılan bir yardımcı yapıdır. Bu, hem `_PyMarshal_ReadObjectFromFile` hem de `_PyMarshal_ReadObjectFromString` fonksiyonlarının aynı temel okuma mantığını (`r_object`) kullanmasına olanak tanır.
    *   **Temel Okuma Fonksiyonları:**
        *   `r_byte(RFILE* p)`: Tek bir byte okur.
        *   `r_string(char* s, int n, RFILE* p)`: `n` byte'lık bir diziyi okur.
        *   `r_short(RFILE* p)`: 2 byte'lık bir short integer okur.
        *   `r_long(RFILE* p)`: 4 byte'lık bir long integer okur.
        *   `r_long64(RFILE* p)`: 8 byte'lık bir long integer okur ve platforma bağlı olarak Python `int` veya `long` nesnesi döndürür.
    *   **`PyObject* r_object(RFILE* p)` (Ana De-serialization Fonksiyonu):**
        *   Bu fonksiyon, `RFILE` yapısından bir tür kodu okuyarak başlar.
        *   Okunan tür koduna göre bir `switch` ifadesi kullanarak ilgili Python nesnesini oluşturur ve döndürür:
            *   **Basit Türler:** `TYPE_NULL` (C `NULL` döndürür), `TYPE_NONE` (`Py_None`), `TYPE_STOPITER` (`PyExc_StopIteration`), `TYPE_ELLIPSIS` (`Py_Ellipsis`).
            *   **Sayısal Türler:**
                *   `TYPE_INT`: `r_long()` ile 4 byte okur ve `PyInt_FromLong()` ile Python `int` nesnesi oluşturur.
                *   `TYPE_INT64`: `r_long64()` ile 8 byte okur.
                *   `TYPE_LONG`: `r_long()` ile basamak sayısını okur, ardından `_PyLong_New()` ile Python `long` (büyük tamsayı) nesnesi oluşturur ve `r_short()` ile her bir basamağı okur.
                *   `TYPE_FLOAT`: Uzunluğu okur, ardından string olarak float değerini okur, `atof()` ile double'a çevirir ve `PyFloat_FromDouble()` ile Python `float` nesnesi oluşturur.
                *   `TYPE_COMPLEX` (eğer `WITHOUT_COMPLEX` tanımlı değilse): Reel ve imajiner kısımları string olarak okur, `atof()` ile double'a çevirir ve `PyComplex_FromCComplex()` ile Python `complex` nesnesi oluşturur.
            *   **Dizi Türleri:**
                *   `TYPE_STRING`: `r_long()` ile string uzunluğunu okur, `PyString_FromStringAndSize()` ile boş bir Python string oluşturur ve `r_string()` ile içeriği doldurur.
                *   `TYPE_UNICODE` (eğer `Py_USING_UNICODE` tanımlıysa): Uzunluğu okur, ham byte'ları bir tampona okur ve `PyUnicode_DecodeUTF8()` ile UTF-8'den Python `unicode` nesnesine çevirir.
                *   `TYPE_TUPLE`: `r_long()` ile eleman sayısını okur, `PyTuple_New()` ile boş bir tuple oluşturur ve ardından her eleman için rekürsif olarak `r_object()` çağırarak tuple'ı doldurur.
                *   `TYPE_LIST`: `TYPE_TUPLE`'a benzer şekilde `PyList_New()` ve `PyList_SetItem()` kullanarak liste oluşturur.
            *   **Eşleme Türü:**
                *   `TYPE_DICT`: `PyDict_New()` ile boş bir sözlük oluşturur. Ardından, `TYPE_NULL` ile işaretlenen sonlandırıcıya kadar anahtar ve değerler için rekürsif olarak `r_object()` çağırır ve `PyDict_SetItem()` ile sözlüğe ekler.
            *   **`TYPE_CODE` (Kod Nesneleri):**
                *   Bu, `.pyc` dosyalarının yüklenmesi için en önemli kısımdır.
                *   Kısıtlı çalışma modunda (`PyEval_GetRestricted()`) kod nesnelerinin de-serialize edilmesini engeller.
                *   Sırasıyla `argcount`, `nlocals`, `stacksize`, `flags` (kısa tamsayılar olarak) gibi bir kod nesnesinin çeşitli özniteliklerini okur.
                *   Ardından, `code` (bytecode string'i), `consts` (sabitler tuple'ı), `names` (isimler tuple'ı), `varnames` (yerel değişken isimleri), `freevars` (serbest değişkenler), `cellvars` (hücre değişkenleri), `filename` (dosya adı), `name` (fonksiyon/modül adı) ve `lnotab` (satır numarası tablosu) gibi diğer bileşenleri rekürsif olarak `r_object()` çağırarak okur.
                *   `firstlineno` (ilk satır numarası) da kısa tamsayı olarak okunur.
                *   Tüm bu bilgilerle `PyCode_New()` fonksiyonunu çağırarak yeni bir Python kod nesnesi oluşturur.
    *   **Dışa Aktarılan Fonksiyonlar:**
        *   `_PyMarshal_ReadShortFromFile(FILE* fp)`: `r_short` için bir sarmalayıcıdır.
        *   `_PyMarshal_ReadLongFromFile(FILE* fp)`: `r_long` için bir sarmalayıcıdır. Bu fonksiyon, `CPythonLauncher::RunCompiledFile` içinde `.pyc` dosyasının magic number'ını ve zaman damgasını okumak için kullanılır.
        *   `_PyMarshal_ReadObjectFromFile(FILE* fp)`: `r_object` için bir sarmalayıcıdır.
        *   `_PyMarshal_ReadLastObjectFromFile(FILE* fp)`: Optimize edilmiş bir okuyucudur. Dosya boyutunu kontrol eder; eğer dosya yeterince küçükse (`SMALL_FILE_LIMIT`, `REASONABLE_FILE_LIMIT` sabitleriyle karşılaştırılır), tüm dosyayı belleğe okur ve ardından `_PyMarshal_ReadObjectFromString` çağırır. Bu, özellikle küçük `.pyc` dosyaları için byte byte okumaktan daha hızlıdır. `CPythonLauncher::RunCompiledFile` içinde ana kod nesnesini okumak için kullanılır.
        *   `_PyMarshal_ReadObjectFromString(char* str, int len)`: `r_object` için bellek tamponundan okuma yapan bir sarmalayıcıdır.

*   **Kullanım Amacı:**
    *   Bu modül, `CPythonLauncher` tarafından `.pyc` dosyalarını (Python'un derlenmiş bytecode'u) yüklemek ve çalıştırmak için kullanılır.
    *   Python'un kendi `marshal` formatıyla uyumlu bir şekilde çalışarak, önceden derlenmiş Python betiklerinin C++ uygulamasından verimli bir şekilde yüklenmesini sağlar.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (ve dolayısıyla Python C API başlıkları - `Python.h`, `longintrepr.h`).
    *   Standart C kütüphaneleri (`stdio.h` için `FILE`, `sys/stat.h` için `struct stat`). 

### `PythonUtils.h` ve `PythonUtils.cpp` (Python Yardımcı Fonksiyonları)

*   **Amaç:** Bu dosyalar, Python C API ile çalışmayı kolaylaştırmak için bir dizi yardımcı fonksiyon ve makro sunar. Temel olarak, Python tuple nesnelerinden çeşitli C++ veri tiplerine (int, long, float, string vb.) güvenli bir şekilde veri almak ve C++ kodundan Python sınıflarının metotlarını çağırmak için sarmalayıcı (wrapper) fonksiyonlar içerirler.

*   **`PythonUtils.h` - Temel Bildirimler:**
    *   **Makro:**
        *   `SET_EXCEPTION(x)`: `PyErr_SetString(PyExc_RuntimeError, #x)` şeklinde, verilen `x` ifadesini bir string'e dönüştürerek Python `RuntimeError` istisnası ayarlar.
    *   **Tuple Okuma Fonksiyonları (Bildirimler):**
        *   `PyTuple_GetString(PyObject* poArgs, int pos, char** ret)`
        *   `PyTuple_GetInteger(PyObject* poArgs, int pos, unsigned char* ret)`
        *   `PyTuple_GetInteger(PyObject* poArgs, int pos, int* ret)`
        *   `PyTuple_GetInteger(PyObject* poArgs, int pos, WORD* ret)`
        *   `PyTuple_GetByte(PyObject* poArgs, int pos, unsigned char* ret)`
        *   `PyTuple_GetUnsignedInteger(PyObject* poArgs, int pos, unsigned int* ret)`
        *   `PyTuple_GetLong(PyObject* poArgs, int pos, long* ret)`
        *   `PyTuple_GetUnsignedLong(PyObject* poArgs, int pos, unsigned long* ret)`
        *   `PyTuple_GetLongLong(PyObject* poArgs, int pos, long long* ret)`
        *   `PyTuple_GetFloat(PyObject* poArgs, int pos, float* ret)`
        *   `PyTuple_GetDouble(PyObject* poArgs, int pos, double* ret)`
        *   `PyTuple_GetObject(PyObject* poArgs, int pos, PyObject** ret)`
        *   `PyTuple_GetBoolean(PyObject* poArgs, int pos, bool* ret)`
        *   Bu fonksiyonlar, bir Python tuple (`poArgs`) içinden belirtilen pozisyondaki (`pos`) elemanı alıp, hedeflenen C++ tipine (`ret` işaretçisi aracılığıyla) dönüştürmeye çalışır. Başarılı olursa `true`, aksi halde (pozisyon geçersizse, tip uyuşmazlığı varsa vb.) `false` döner.
    *   **Python Sınıf Metodu Çağırma Fonksiyonları (Bildirimler):**
        *   `PyCallClassMemberFunc(PyObject* poClass, const char* c_szFunc, PyObject* poArgs)`: Bir Python sınıf örneğinin (`poClass`) belirtilen C-string isimli (`c_szFunc`) metodunu `poArgs` argümanlarıyla çağırır. Dönüş değerini atar.
        *   `PyCallClassMemberFunc(PyObject* poClass, const char* c_szFunc, PyObject* poArgs, bool* pisRet)`: Yukarıdakine benzer, ancak metodun dönüş değerini (eğer sayısal ise 0 olup olmamasına göre, değilse her zaman true) `pisRet` boolean işaretçisine yazar.
        *   `PyCallClassMemberFunc(PyObject* poClass, const char* c_szFunc, PyObject* poArgs, long * plRetValue)`: Metodun sayısal dönüş değerini `plRetValue` long işaretçisine yazar.
        *   `PyCallClassMemberFunc_ByPyString(PyObject* poClass, PyObject* poFuncName, PyObject* poArgs)`: Metot adı Python string nesnesi (`poFuncName`) olarak verilir. Dönüş değerini atar.
        *   `PyCallClassMemberFunc(PyObject* poClass, PyObject* poFunc, PyObject* poArgs)`: Çağrılacak metot doğrudan bir Python fonksiyon nesnesi (`poFunc`) olarak verilir. Dönüş değerini atar.
    *   **Python İstisna ve Nesne Oluşturma Yardımcıları (Bildirimler):**
        *   `Py_BuildException(const char * c_pszErr = NULL, ...)`: Formatlı bir string alarak Python `RuntimeError` istisnası oluşturur ve ayarlar. Eğer `c_pszErr` NULL ise mevcut hatayı temizler. `Py_BuildNone()` döndürür.
        *   `Py_BadArgument()`: `PyErr_BadArgument()` çağırır ve `NULL` döndürür.
        *   `Py_BuildNone()`: `Py_None` nesnesini referans sayısını artırarak döndürür.
        *   `Py_BuildEmptyTuple()`: Boş bir Python tuple (`()`) oluşturur ve döndürür (Bu fonksiyonun bildirimi var ancak implementasyonu `PythonUtils.cpp` içinde görünmüyor).

*   **`PythonUtils.cpp` - Uygulama Detayları:**
    *   **Global Değişken:**
        *   `IPythonExceptionSender * g_pkExceptionSender = NULL;`: Python'dan C++'a istisna bilgilerini göndermek için kullanılan bir arayüze işaretçi (`PythonDebugModule.cpp`'deki ile aynı).
    *   **Python 2.7 Uyumluluk Makroları:**
        *   `PyLong_AsLong` -> `PyLong_AsLongLong`
        *   `PyLong_AsUnsignedLongLong` -> `PyLong_AsUnsignedLongLong` (bu zaten aynı)
        *   `PyLong_AsUnsignedLong` -> `(unsigned long)PyLong_AsLongLong`
        *   Bu makrolar, Python 2.x serisindeki `long` ve `int` birleşmesinden kaynaklanan bazı C API fonksiyon isim değişikliklerini veya davranışlarını yönetmek için olabilir. Python 2.7 hedeflendiği için `PyLong_AsLongLong` genellikle 64-bit tamsayıları işler.
    *   **Tuple Okuma Fonksiyonları (Uygulama):**
        *   Tüm `PyTuple_Get*` fonksiyonları benzer bir yapı izler:
            1.  İstenen pozisyonun tuple sınırları içinde olup olmadığını kontrol eder.
            2.  `PyTuple_GetItem(poArgs, pos)` ile tuple'dan ilgili Python nesnesini alır.
            3.  Alınan Python nesnesini uygun `PyLong_AsLong`, `PyFloat_AsDouble`, `PyString_AsString` gibi Python C API fonksiyonlarıyla hedef C++ tipine dönüştürür.
            4.  Tip dönüşümü veya erişim sırasında bir sorun olmazsa `true` döner.
    *   **Python Sınıf Metodu Çağırma Fonksiyonları (Uygulama):**
        *   `__PyCallClassMemberFunc_ByCString`, `__PyCallClassMemberFunc_ByPyString`, `__PyCallClassMemberFunc` (ön ekli olanlar asıl işi yapan özel yardımcı fonksiyonlardır):
            1.  Sınıf nesnesinin (`poClass`) varlığını kontrol eder.
            2.  Metot adıyla (`c_szFunc` veya `poFuncName`) ya da doğrudan fonksiyon nesnesiyle (`poFunc`) `PyObject_GetAttrString` veya `PyObject_GetAttr` kullanarak Python sınıfından metot nesnesini alır.
            3.  Alınan metot nesnesinin çağrılabilir (`PyCallable_Check`) olup olmadığını kontrol eder.
            4.  `PyObject_CallObject(poFunc, poArgs)` ile metodu verilen argümanlarla çağırır.
            5.  Eğer çağrı sırasında bir Python istisnası oluşursa (`!poRet`), `g_pkExceptionSender` kullanılarak istisna bilgisi gönderilmeye çalışılır ve `PyErr_Print()` ile Python hata yığını yazdırılır.
            6.  Başarılı olursa çağrının dönüş değerini (`poRet`) `ppoRet` işaretçisi aracılığıyla dışarı verir.
            7.  Referans sayımlarını yönetir (`Py_DECREF`, `Py_XDECREF`).
        *   Dışa açık `PyCallClassMemberFunc` varyasyonları bu iç fonksiyonları çağırır ve dönüş değerlerini (varsa) uygun şekilde işler veya atar.
    *   **Diğer Yardımcılar:**
        *   `Py_BuildException()`: `vsnprintf` kullanarak formatlı hata mesajı oluşturur ve `PyErr_SetString` ile ayarlar.
        *   `Py_BadArgument()`: Standart "Bad Argument" Python hatasını ayarlar.
        *   `Py_BuildNone()`: `Py_None` döndürür.
        *   `Py_ReleaseNone()`: Bu fonksiyonun `PythonUtils.h`'de bildirimi yok ama `PythonUtils.cpp` içinde tanımlanmış. `Py_None`'ın referans sayısını azaltır.

*   **Kullanım Amacı:**
    *   C++ tarafında Python ile etkileşim kuran kodun daha okunabilir ve yönetilebilir olmasını sağlar.
    *   Python C API'sinin karmaşıklığını bir nebze gizleyerek, tuple'dan veri okuma ve metot çağırma gibi yaygın işlemleri standartlaştırır.
    *   Hata yönetimini merkezileştirir (örneğin, metot çağrılarında `PyErr_Print` ve `g_pkExceptionSender` kullanımı).

*   **Bağımlılıklar:**
    *   `StdAfx.h` (ve dolayısıyla Python C API başlıkları).
    *   `IPythonExceptionSender` arayüzü.

### `Resource.h` ve `Resource.cpp` (`CPythonResource` Sınıfı)

*   **Amaç:** `CPythonResource` sınıfı, bir singleton olarak, istemcinin kullandığı çeşitli oyun kaynaklarının (`CResource` türevleri) merkezi bir şekilde yönetilmesinden sorumludur. `CResourceManager`'ı kullanarak, farklı dosya uzantılarına sahip kaynakların nasıl yükleneceğini (hangi C++ sınıfının örneklenerek oluşturulacağını) tanımlar. Ayrıca, kaynakların listesini dökme ve tüm kaynakları temizleme gibi genel yönetim işlevleri sunar.

*   **`Resource.h` - Temel Tanımlar ve Bildirimler:**
    *   **Enum `EResourceTypes`:**
        *   `RES_TYPE_UNKNOWN`: Bilinmeyen bir kaynak türünü belirtmek için tanımlanmıştır, ancak `CPythonResource` içinde aktif olarak kullanılmıyor gibi görünmektedir.
    *   **Sınıf Tanımı:**
        *   `CPythonResource : public CSingleton<CPythonResource>`: `CPythonResource`, `CSingleton` şablonu aracılığıyla bir singleton olarak tasarlanmıştır.
    *   **Genel Metotlar:**
        *   `CPythonResource()`: Yapıcı metot. Çeşitli dosya uzantıları için kaynak oluşturma fonksiyonlarını `m_resManager`'a kaydeder.
        *   `~CPythonResource()`: Yıkıcı metot.
        *   `Destroy()`: Oyunun çeşitli alt sistemlerini (aktörler, efektler, UI elemanları vb.) ve `m_resManager` aracılığıyla tüm yüklenmiş kaynakları temizler.
        *   `DumpFileList(const char* c_szFileName)`: `m_resManager` tarafından yönetilen tüm kaynakların dosya adlarını belirtilen metin dosyasına yazdırır.
    *   **Korunan (Protected) Üyeler:**
        *   `m_resManager` (CResourceManager): `EterLib` kütüphanesinden gelen ve asıl kaynak yönetimini yapan `CResourceManager` nesnesi.

*   **`Resource.cpp` - Uygulama Detayları:**
    *   **Kaynak Oluşturma Fabrika Fonksiyonları:**
        *   `CResource* NewImage(const char* c_szFileName)`: `CGraphicImage` nesnesi oluşturur.
        *   `CResource* NewSubImage(const char* c_szFileName)`: `CGraphicSubImage` nesnesi oluşturur.
        *   `CResource* NewText(const char* c_szFileName)`: `CGraphicText` (font dosyaları için) nesnesi oluşturur.
        *   `CResource* NewThing(const char* c_szFileName)`: `CGraphicThing` (genellikle `.gr2` 3D modelleri için) nesnesi oluşturur.
        *   `CResource* NewEffectMesh(const char* c_szFileName)`: `CEffectMesh` (efektler için mesh verileri) nesnesi oluşturur.
        *   `CResource* NewAttributeData(const char* c_szFileName)`: `CAttributeData` (çarpışma ve yükseklik verileri) nesnesi oluşturur.
        *   Bu fonksiyonlar, `CResourceManager::RegisterResourceNewFunctionPointer` ile belirli dosya uzantılarına bağlanır.
    *   **`CPythonResource::CPythonResource()` (Yapıcı):**
        *   `m_resManager.RegisterResourceNewFunctionPointer()` çağrıları aracılığıyla dosya uzantılarını yukarıdaki fabrika fonksiyonlarıyla eşleştirir:
            *   `.sub` -> `NewSubImage`
            *   `.dds`, `.jpg`, `.tga`, `.png`, `.bmp` -> `NewImage`
            *   `.fnt` -> `NewText`
            *   `.gr2` -> `NewThing`
            *   `.mde`, `.msa`, `.mse` -> `NewEffectMesh` (farklı efekt mesh formatları olabilir)
            *   `.mdatr` -> `NewAttributeData`
    *   **`CPythonResource::Destroy()`:**
        *   Bu metot, istemcinin kapanışı sırasında çağrılarak tüm kaynakları ve kaynaklarla ilişkili sistemleri temizler. Çok sayıda statik `DestroySystem()` çağrısı içerir:
            *   **Instance Sistemleri:** `CFlyingInstance`, `CActorInstance`, `CArea`, `CGraphicExpandedImageInstance`, `CGraphicImageInstance`, `CGraphicMarkInstance`, `CGraphicThingInstance`, `CGrannyModelInstance`, `CGraphicTextInstance`, `CEffectInstance`, `CWeaponTrace`, `CFlyTrace`. Bu çağrılar, bu türdeki tüm aktif örneklerin (instance) temizlenmesini sağlar.
            *   `m_resManager.DestroyDeletingList()`: `CResourceManager`'ın silinmeyi bekleyen kaynak listesini işler.
            *   **Data Sistemleri:** `CFlyingData`, `CItemData`, `CEffectData`, `CEffectMesh::SEffectMeshData`, `CRaceData`, `NRaceData`, `CRaceMotionData`. Bu çağrılar, bu türdeki tüm paylaşılan kaynak verilerinin temizlenmesini sağlar.
            *   `m_resManager.Destroy()`: Son olarak, `CResourceManager`'ın kendisini ve kalan tüm kaynakları temizler.
    *   **`CPythonResource::DumpFileList(const char* c_szFileName)`:**
        *   `m_resManager.DumpFileListToTextFile(c_szFileName)` fonksiyonunu çağırarak, `CResourceManager` tarafından o anda yüklenmiş olan tüm kaynakların bir listesini (genellikle dosya yolları) belirtilen dosyaya yazdırır. Bu, hata ayıklama veya kaynak kullanımını analiz etmek için faydalıdır.

*   **Kullanım Amacı:**
    *   İstemcinin genel kaynak yönetim altyapısını oluşturur. `CResourceManager`'ı yapılandırarak, farklı dosya türlerinin nasıl yükleneceğini ve hangi C++ sınıflarıyla temsil edileceğini belirler.
    *   Oyun başlatıldığında, bu singleton örneği oluşturulur ve kaynak yöneticisi ayarlanır.
    *   Oyun sırasında, kodun çeşitli yerleri `CResourceManager::Instance().GetResourcePointer(filename)` gibi çağrılarla kaynaklara erişir. `CResourceManager`, dosya uzantısına bakarak `CPythonResource` tarafından kaydedilmiş uygun `New*` fonksiyonunu çağırır ve kaynağı yükler veya önbellekten döndürür.
    *   Oyun kapatılırken, `CPythonResource::Destroy()` metodu çağrılarak tüm kaynakların düzenli bir şekilde serbest bırakılması sağlanır.

*   **Bağımlılıklar:**
    *   `StdAfx.h` (ScriptLib için)
    *   `../EterLib/Resource.h`, `../EterLib/ResourceManager.h`
    *   Çeşitli kütüphanelerden kaynak sınıfları: `CGraphicImageInstance`, `CGraphicTextInstance`, `CAttributeData` (`EterLib`'den), `CGraphicThing`, `CGraphicThingInstance` (`EterGrnLib`'den), `CEffectMesh`, `CEffectInstance`, `CEffectData` (`EffectLib`'den), `CWeaponTrace`, `CRaceData`, `CActorInstance`, `CItemData` vb. (`GameLib`'den).

### `StdAfx.h` ve `StdAfx.cpp` (Ön Derlenmiş Başlıklar ve İstisna Gönderici)

*   **Amaç:** Bu dosyalar, `ScriptLib` kütüphanesi için ön derlenmiş başlık (PCH) mekanizmasını sağlar ve Python betiklerinden C++ tarafına detaylı hata/istisna bilgilerini göndermek için temel bir altyapı oluşturur (`IPythonExceptionSender` arayüzü).

*   **`StdAfx.h` İçeriği:**
    *   **Dahil Edilen Temel Kütüphane Başlıkları:**
        *   `../EterLib/StdAfx.h`: `EterLib`'in PCH'si.
        *   `../EterGrnLib/StdAfx.h`: `EterGrnLib`'in PCH'si.
        *   `../UserInterface/Locale_inc.h`: Yerelleştirme ile ilgili tanımlamalar.
    *   **Python C API Başlıkları:**
        *   `Python.h` ve Python C API'sinin çeşitli diğer bileşenleri (`node.h`, `grammar.h`, `token.h`, `parsetok.h`, `errcode.h`, `compile.h`, `symtable.h`, `eval.h`, `marshal.h`) dahil edilir. `_DEBUG` makrosu için özel bir sıralama ile Python başlıklarının doğru şekilde dahil edilmesi sağlanır.
    *   **`ScriptLib` İçi Başlıklar:**
        *   `PythonUtils.h`
        *   `PythonLauncher.h`
        *   `PythonMarshal.h`
        *   `Resource.h`
    *   **Fonksiyon Bildirimi:**
        *   `void initdbg();`: `PythonDebugModule.cpp` içinde tanımlanan `dbg` Python modülünü başlatan fonksiyon.
    *   **`IPythonExceptionSender` Arayüzü:**
        *   Bu arayüz (interface class), Python tarafından C++ tarafına istisna ve hata mesajlarının nasıl iletileceğini tanımlar.
        *   **Üye Değişken (protected):**
            *   `m_strExceptionString` (std::string): Toplanan hata/istisna mesajlarını biriktirir.
        *   **Public Metotlar:**
            *   `Clear()`: `m_strExceptionString`'i temizler.
            *   `RegisterExceptionString(const char* c_szString)`: Verilen string'i `m_strExceptionString`'e ekler.
            *   `virtual void Send() = 0;`: Saf sanal (pure virtual) bir metottur. Bu arayüzü implemente eden somut bir sınıf, bu metodu tanımlayarak biriktirilen hata mesajının nasıl işleneceğini (örn: log dosyasına yazma, kullanıcıya gösterme) belirler.
    *   **Global Değişken ve Fonksiyon Bildirimi (İstisna Gönderici için):**
        *   `extern IPythonExceptionSender* g_pkExceptionSender;`: `IPythonExceptionSender` arayüzünü implemente eden bir nesneye işaret eden global bir gösterici. Bu gösterici, `ScriptLib` içindeki çeşitli modüllerin (örn: `PythonLauncher`, `PythonDebugModule`) istisna bilgilerini kaydetmek için kullanılır.
        *   `void SetExceptionSender(IPythonExceptionSender* pkExceptionSender);`: `g_pkExceptionSender` global göstericisini ayarlamak için kullanılan fonksiyon.

*   **`StdAfx.cpp` İçeriği:**
    *   `#include "stdafx.h"`: PCH mekanizması için gereklidir.
    *   **`SetExceptionSender` Fonksiyonu Tanımı:**
        *   `void SetExceptionSender(IPythonExceptionSender* pkExceptionSender)`: Argüman olarak aldığı `pkExceptionSender` işaretçisini global `g_pkExceptionSender` değişkenine atar. Bu, genellikle uygulamanın başlatılması sırasında, ana uygulama tarafından sağlanacak somut bir istisna gönderici nesnesiyle çağrılır.

*   **Kullanım Amacı:**
    *   `StdAfx.h`, `ScriptLib` içindeki tüm `.cpp` dosyalarının ortak olarak kullandığı başlıkları ve Python C API'sini içererek derleme sürelerini kısaltır.
    *   `IPythonExceptionSender` arayüzü ve `g_pkExceptionSender` global değişkeni, Python'da bir hata oluştuğunda veya C++'dan Python çağrıları başarısız olduğunda, bu hataların detaylarının (çağrı yığını, hata mesajları vb.) merkezi bir şekilde toplanıp, istemcinin ana hata yönetim sistemi tarafından işlenmesine olanak tanır.
    *   `SetExceptionSender` fonksiyonu, istemcinin ana bölümünün, `ScriptLib`'in kullanacağı özel bir hata/istisna işleyici (handler) sağlamasına imkan verir.

*   **Bağımlılıklar:**
    *   `EterLib`, `EterGrnLib`, `UserInterface` kütüphaneleri.
    *   Python C API (Python 2.7).
    *   `ScriptLib` içindeki diğer modüller (`PythonUtils`, `PythonLauncher` vb.).