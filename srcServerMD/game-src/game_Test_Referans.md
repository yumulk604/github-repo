# Metin2 Oyun Sunucusu - Test Kodları Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) çeşitli test modüllerini, yardımcı programlarını ve birim testlerini belgeler.

## İçindekiler

*   [`test_allocator.cpp`](#test_allocatorcpp)
*   [`test_stacktrace.cpp`](#test_stacktracecpp)
*   [`test_window.cpp`](#test_windowcpp)
*   [`test.cpp`](#testcpp)

---

### `test_allocator.cpp`

*   **Amaç:** `object_allocator.h` içinde tanımlanan `ObjectAllocator` bellek ayırma mekanizmasının doğru çalıştığını doğrulamak için birim testleri içerir. Google Test (gtest) çatısını kullanır.
*   **Temel İşlevler/İçerik:**
    *   **`T` ve `T2` Struct'ları:**
        *   `ObjectAllocator`'dan türetilmiş basit test yapılarıdır. Farklı boyutlarda iç dizilere (`v[3]` ve `v[128]`) sahiptirler, bu da farklı boyutlardaki nesneler için ayırıcının test edilmesini sağlar.
    *   **`TEST(allocator, object_allocator)` Test Durumu:**
        *   `T` türünden iki nesne (`p1`, `p2`) oluşturur (`M2_OBJ_NEW`).
        *   Bu nesneleri siler (`M2_OBJ_DELETE`).
        *   Her silme işleminden sonra `T::GetFreeBlockCount()` ile serbest blok sayısının arttığını doğrular (`EXPECT_TRUE`).
        *   Belirli sayıda (`eTEST_COUNT = 10000`) `T` nesnesi oluşturur ve ardından siler.
        *   Tüm silme işlemlerinden sonra serbest blok sayısının beklenen bir değere (varsayılan olarak `FREE_TRIGGER` olan 128'e kadar) ulaştığını doğrular.
    *   **`TEST(allocator, object_allocator_2)` Test Durumu:**
        *   Farklı türlerden (`T` ve `T2`) nesneler oluşturup silerek, her tür için `ObjectAllocator`'ın kendi serbest blok sayacını doğru yönettiğini doğrular.
    *   **`TEST(allocator, character_alloc)` Test Durumu (Yorum Satırında):**
        *   Bu test durumu `#if 0` ile devre dışı bırakılmıştır.
        *   Yorumlarda belirtildiği üzere, `CHARACTER` nesnelerinin doğrudan `new` ile veya `M2_OBJ_DELETE` ile `ObjectAllocator` üzerinden test edilmesi, singleton bağımlılıkları nedeniyle oyunun çökmesine neden olmaktadır. Bu nedenle bu test aktif değildir.
*   **Genel Çalışma Prensibi:**
    *   Testler, `M2_OBJ_NEW` ve `M2_OBJ_DELETE` makrolarını kullanarak `ObjectAllocator` üzerinden nesneler oluşturur ve siler.
    *   Her adımdan sonra `ObjectAllocator<NesneTuru>::GetFreeBlockCount()` ile serbest bırakılan ve yeniden kullanılmayı bekleyen bellek bloklarının sayısını kontrol ederek ayırıcının beklendiği gibi davrandığını (serbest bırakılan blokları listeye eklediğini, belirli bir eşiğe kadar hemen sisteme iade etmediğini) doğrular.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `<gtest/gtest.h>`, `<execinfo.h>`, `<cxxabi.h>`, `<stdlib.h>` (`malloc`, `free` için), `../../libgame/include/attribute.h`, `../../libgame/include/targa.h`, `../../common/d3dtype.h`, `utils.h`, `minilzo.h`, `sectree.h`, `sectree_manager.h`, `vector.h`, `lzo_manager.h`, `char_manager.h`, `desc_manager.h`, `questmanager.h`, `arena.h`, `pvp.h`, `cmd.h`. Standart C/C++ kütüphaneleri (`stdio.h`, `string.h`, `stdlib.h`, `stat.h`, `dirent.h` veya Windows için `windows.h`).

---

### `test_stacktrace.cpp`

*   **Amaç:** Çağrı yığını (stack trace) bilgilerini elde etme ve C++ sembollerini okunabilir fonksiyon isimlerine dönüştürme (demangling) işlevlerini test eder. Bu tür bir işlevsellik, hata ayıklama sırasında çökmelerin veya hataların kaynağını bulmak için çok önemlidir. Google Test (gtest) çatısını kullanır.
*   **Temel İşlevler/İçerik:**
    *   **Dahil Edilen Başlıklar:**
        *   `stdafx.h`: Projenin ön derlenmiş başlığı.
        *   `<gtest/gtest.h>`: Google Test kütüphanesi.
        *   `<execinfo.h>`: `backtrace` ve `backtrace_symbols` fonksiyonları için (genellikle GNU/Linux sistemlerinde bulunur).
        *   `<cxxabi.h>`: `abi::__cxa_demangle` fonksiyonu için (C++ ABI, sembol çözme).
    *   **`TEST(utils, stacktrace)` Test Durumu:**
        *   `backtrace(array, 200)`: Mevcut çağrı yığınındaki en fazla 200 adres bilgisini `array` dizisine kaydeder.
        *   `backtrace_symbols(array, size)`: Bu adresleri sembolik stringlere (genellikle `dosya_adı(fonksiyon_adı+offset) [adres]` formatında) dönüştürür.
        *   `size > 0` olduğunu doğrular (`EXPECT_TRUE`).
        *   Her bir sembolik string üzerinde döner:
            *   Stringi ayrıştırarak C++ 'mangle' edilmiş fonksiyon adını (`begin_name`), ofsetini (`begin_offset`) ve diğer kısımları çıkarır. Bu, `<...>`, `+...` ve `>` karakterlerini arayarak yapılır.
            *   Eğer gerekli kısımlar bulunursa, `abi::__cxa_demangle` fonksiyonunu kullanarak 'mangle' edilmiş fonksiyon adını (`begin_name`) okunabilir bir C++ fonksiyon adına (`funcname`) dönüştürmeye çalışır.
            *   Dönüştürme başarılı olursa (`status == 0`), çözülmüş ismi kullanır, aksi halde orijinal 'mangle' edilmiş ismi kullanır.
            *   (Yorum satırına alınmış) `printf` ile ayrıştırılmış ve çözülmüş bilgileri yazdırabilirdi.
        *   Ayrılan bellekleri (`funcname`, `strings`) serbest bırakır.
    *   **`TEST(utils, stacktrace2)` Test Durumu:**
        *   `backtrace` ve `backtrace_symbols` ile çağrı yığınını alır.
        *   `size > 0` olduğunu doğrular.
        *   Sembol çözme (demangling) yapmadan, ham sembolik stringleri doğrudan `printf` ile yazdırır.
        *   Ayrılan belleği (`strings`) serbest bırakır.
*   **Genel Çalışma Prensibi:**
    *   Bu testler, `execinfo.h` başlığındaki standart GNU/Linux fonksiyonlarını kullanarak programın mevcut çağrı yığınını elde eder.
    *   Elde edilen ham sembolleri (adresler ve 'mangle' edilmiş isimler) `cxxabi.h` içindeki `__cxa_demangle` fonksiyonu aracılığıyla insan tarafından okunabilir C++ fonksiyon isimlerine dönüştürme yeteneğini test eder.
    *   İkinci test durumu, ham sembollerin doğrudan alınabildiğini gösterir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `<gtest/gtest.h>`, `<execinfo.h>`, `<cxxabi.h>`, `<stdlib.h>` (`malloc`, `free` için).

---

### `test_window.cpp`

*   **Amaç:** Bu dosya, çeşitli harita verilerini (çarpışma verileri, harita öznitelikleri, renk haritaları) işlemek, dönüştürmek ve test etmek için bir komut satırı arayüzü ve yardımcı fonksiyonlar içerir. `SECTREE_MANAGER` ve ilgili harita yapılarının temel işlevlerini test etmek veya harita verilerini sunucu için uygun formata getirmek amacıyla kullanılır. Normal oyun sunucusu çalışması sırasında kullanılmaz, bir geliştirme/test aracıdır.
*   **Temel İşlevler/İçerik:**
    *   **Global Değişkenler ve Yapılar:**
        *   `COLLISION_TYPE_...` Enum'u: Çarpışma veri türlerini tanımlar (PLANE, BOX, SPHERE, CYLINDER).
        *   `TSphereData`, `TPlaneData`, `TCylinderData`: Çarpışma geometrilerine ait verileri tutan yapılar.
        *   `g_pkMapSectree` (`LPSECTREE_MAP`): Test edilen haritanın sektör haritası işaretçisi.
        *   Diğer global değişkenler (`g_bx`, `g_by`, `main_fdw`, `udp_socket`, `g_bShutdown`) muhtemelen `thecore` kütüphanesi veya daha eski testler için kalıntılardır.
    *   **Yardımcı ve Dönüştürme Fonksiyonları:**
        *   `ConvertAttribute(c_pszCollisionDataFileName, c_pszMapDirectory)`: Belirtilen harita dizini için `Setting.txt` dosyasını okur, sektörleri oluşturur, `server_attr` (harita öznitelikleri) dosyalarını (`.atr`) ve çarpışma verilerini (`.m2cd` - `c_pszCollisionDataFileName` ile belirtilen) okuyarak `g_pkMapSectree`'deki sektörlerin özniteliklerini ayarlar. Son olarak, işlenmiş öznitelikleri yeni bir sıkıştırılmış `server_attr` dosyasına ve bir test `test.tga` resmine kaydeder.
        *   `ReadMapAttribute(dwAreaX, dwAreaY, c_pszFileName)`: Belirli bir alan koordinatına (`dwAreaX`, `dwAreaY`) ait `.atr` dosyasını okur ve içindeki öznitelik verilerini ilgili sektörlere (`g_pkMapSectree` kullanarak) yazar.
        *   `ReadCollisionData(c_pszFileName, iBaseX, iBaseY)`: `.m2cd` formatındaki çarpışma verisi dosyasını okur. Dosyadaki düzlem (PLANE), küre (SPHERE) ve silindir (CYLINDER) çarpışma nesnelerini işler. `ProcessLine` ve `ProcessSphere` fonksiyonlarını kullanarak bu geometrilerin kapladığı alanları `g_pkMapSectree`'deki sektör özniteliklerine engel (ATTR_BLOCK gibi) olarak işaretler (`BlockAttribute` fonksiyonu ile).
        *   `ProcessLine(sx, sy, ex, ey, f)`: İki nokta arasında bir çizgi üzerindeki tüm hücrelerde `f` fonksiyonunu çalıştıran bir Bresenham çizgi algoritması benzeri bir implementasyon.
        *   `ProcessSphere(x, y, fRadius, f)`: Belirli bir merkez ve yarıçapa sahip bir daire içindeki hücrelerde `f` fonksiyonunu çalıştırır.
        *   `BlockAttribute(x, y)`: Verilen dünya koordinatındaki hücrenin özniteliğine engel bayrağı ekler.
        *   `ReadColorMap(c_pszFileName)`: `.cmp` (renk haritası/tile haritası) dosyasını okur ve kullanılan tile setlerini analiz eder (kaç farklı tile seti kullanıldığını sayar ve yazdırır).
        *   `ReadAllMap(c_pszDirName)` / `ReadColorMapRecursive(c_pszDirName)`: Belirtilen bir dizindeki tüm `.atr` veya `.cmp` dosyalarını yinelemeli olarak bulup işleyen fonksiyonlar.
        *   `ConvertAttribute2(filename)`: Mevcut bir `server_attr` dosyasını okur, LZO ile açar, içindeki öznitelik verilerinden belirli (muhtemelen eski veya istenmeyen) bitleri temizler, tekrar LZO ile sıkıştırır ve `.new` uzantılı yeni bir dosyaya yazar.
        *   `SectreeTest(LPSECTREE pSec)`: Bir sektörün özniteliklerini konsola karakter ('X' veya ' ') olarak yazdıran bir test fonksiyonu.
        *   `GetFindFileList(...)` (Windows'a özgü): Belirli bir dizindeki dosyaları filtreleyerek listeler.
    *   **`main(int argc, char** argv)` Fonksiyonu:**
        *   Temel sunucu bileşenlerinin (SECTREE_MANAGER, DESC_MANAGER, CHARACTER_MANAGER, CQuestManager, CArenaManager, CPVPManager, LZOManager) örneklerini oluşturur.
        *   `thecore_init` ile `libthecore` kütüphanesini başlatır.
        *   Kullanıcıdan standart girdi (stdin) ile komutlar alır.
        *   Alınan komutlara göre ilgili fonksiyonları çağırır:
            *   **`a <collision_data_file> <map_directory>`:** `ConvertAttribute` fonksiyonunu çağırarak harita özniteliklerini ve çarpışma verilerini işler, `server_attr` oluşturur.
            *   **`c <filename>`:** `ConvertAttribute2` fonksiyonunu çağırarak mevcut bir `server_attr` dosyasını dönüştürür.
            *   **`b`:** Bir karakter oluşturur ve `interpret_command` ile bir buffer overflow testi yapar (valgrind veya gdb ile kullanılmak üzere tasarlanmıştır).
            *   **`q`:** Programı sonlandırır.
    *   **Boş Fonksiyonlar/Stubs:** `ContinueOnFatalError()`, `ShutdownOnFatalError()`, `heartbeat(...)`, `Metin2Server_IsInvalid()` gibi fonksiyonlar, tam bir sunucu ortamı olmadan bu test dosyasının derlenebilmesi için boş bırakılmış veya basit implementasyonlar içerir.
*   **Genel Çalışma Prensibi:**
    *   Bu dosya, genellikle Metin2 harita verilerini (özellikle `server_attr` ve çarpışma bilgileri) hazırlamak veya hata ayıklamak için kullanılan bir araçtır.
    *   `ConvertAttribute` ana işlev olup, istemci tarafı harita dosyalarından (`.atr`, `.m2cd`) sunucu tarafı `server_attr` dosyasını oluşturur.
    *   Diğer fonksiyonlar bu sürece yardımcı olur veya belirli veri türlerini test eder/dönüştürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `../../libgame/include/attribute.h`, `../../libgame/include/targa.h`, `../../common/d3dtype.h`, `utils.h`, `minilzo.h`, `sectree.h`, `sectree_manager.h`, `vector.h`, `lzo_manager.h`, `char_manager.h`, `desc_manager.h`, `questmanager.h`, `arena.h`, `pvp.h`, `cmd.h`. Standart C/C++ kütüphaneleri (`stdio.h`, `string.h`, `stdlib.h`, `stat.h`, `dirent.h` veya Windows için `windows.h`).

---

### `test.cpp`

*   **Amaç:** Bu dosya, `test_window.cpp` dosyasıyla büyük ölçüde aynı veya tamamen aynı içeriğe sahip görünmektedir. Çeşitli harita verilerini (çarpışma verileri, harita öznitelikleri, renk haritaları) işlemek, dönüştürmek ve test etmek için bir komut satırı arayüzü ve yardımcı fonksiyonlar içerir. Normal oyun sunucusu çalışması sırasında kullanılmaz, bir geliştirme/test aracıdır.
*   **Temel İşlevler/İçerik:**
    *   İçerik ve işlevsellik açısından `test_window.cpp` ile aynıdır. Lütfen `test_window.cpp` için yapılan belgelemeye bakınız.
    *   **Olası Durum:** Bu dosya, `test_window.cpp`'nin bir kopyası, eski bir versiyonu veya geliştirme sırasında farklı bir amaçla oluşturulup sonradan aynı içeriğe sahip olmuş bir dosya olabilir. Proje yapısında zaman zaman bu tür fazlalıklar oluşabilir.
*   **Bağlantılı Dosyalar:** `test_window.cpp` ile aynıdır: `stdafx.h`, `../../libgame/include/attribute.h`, `../../libgame/include/targa.h`, `../../common/d3dtype.h`, `utils.h`, `minilzo.h`, `sectree.h`, `sectree_manager.h`, `vector.h`, `lzo_manager.h`, `char_manager.h`, `desc_manager.h`, `questmanager.h`, `arena.h`, `pvp.h`, `cmd.h`. Standart C/C++ kütüphaneleri.

--- 