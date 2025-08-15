# UserInterface Referans Kılavuzu - Bölüm 16

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`Test.h`](#testh)
*   [`UserInterface.cpp`](#userinterfacecpp)
*   [`UserInterface.rc`](#userinterfacerc)
*   [`Version.h`](#versionh)
*   [`Version.py`](#versionpy)

---

### `Test.h`

**Dosyanın Amacı ve İşlevselliği**

`Test.h`, istemcinin test sunucusu modunda olup olmadığını belirten `__IS_TEST_SERVER_MODE__` adlı global bir boolean değişkeni tanımlar. Bu değişken, geliştirme ve test aşamalarında belirli özellikleri aktif etmek veya devre dışı bırakmak için kullanılır.

**Temel Fonksiyonlar ve Değişkenler**

*   `extern bool __IS_TEST_SERVER_MODE__;`: Test sunucusu modunun aktif olup olmadığını gösteren harici boolean değişken. Eğer `true` ise, istemci test sunucusu ayarlarıyla çalışır; `false` ise normal (canlı) sunucu ayarlarıyla çalışır. Bu değişken genellikle `UserInterface.cpp` içinde ayarlanır ve projenin çeşitli yerlerinde `#ifdef` veya `if` koşullarıyla kontrol edilerek farklı davranışlar sergilenmesini sağlar.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, oyunun geliştirme veya test sürümlerinde, normal oyuncularda bulunmayan veya farklı çalışan özellikleri (örneğin, debug komutları, özel loglamalar, test NPC'leri vb.) yönetmek için kullanılır. Geliştiriciler, bu değişkeni kullanarak canlı sunucuyu etkilemeden yeni özellikleri test edebilirler.

---

### `UserInterface.cpp`

**Dosyanın Amacı ve İşlevselliği**

`UserInterface.cpp`, Metin2 istemcisinin ana giriş noktasını (`WinMain`) ve temel başlatma, yapılandırma ve Python entegrasyonu işlemlerini içeren merkezi bir dosyadır. İstemcinin çalışması için gerekli olan kütüphaneleri yükler, paket (pack) sistemini başlatır, Python yorumlayıcısını hazırlar, ana oyun betiğini çalıştırır ve istemci sonlandığında kaynakları temizler. Ayrıca, yerelleştirme (locale) ayarları, komut satırı argümanlarının işlenmesi, hata yönetimi ve çeşitli sistem kontrolleri (MD5 dosya kontrolü, en son dosya kontrolü vb.) gibi önemli görevleri yerine getirir.

**Ana Çalışma Prensipleri ve Temel Fonksiyonlar**

*   **`WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)`**: Windows uygulamalarının ana giriş noktasıdır. Burası programın başladığı yerdir.
    *   Komut satırı argümanlarını işler (örneğin, `/timestamp`, `--locale`, `--force-set-locale`).
    *   Gerekli kontrolleri yapar (örneğin, `NEEDED_COMMAND_ARGUMENT`, `NEEDED_COMMAND_CLIPBOARD` ile yama üzerinden başlatılma kontrolü, `CheckPythonLibraryFilenames` ile Python kütüphane dosyalarının varlığı).
    *   `CefWebBrowser_Startup` ile gömülü web tarayıcısını başlatır.
    *   `Main(hInstance, lpCmdLine)` fonksiyonunu çağırarak asıl istemci mantığını başlatır.
    *   `CefWebBrowser_Cleanup` ile web tarayıcısını temizler.
    *   Hata mesajlarını gösterir ve programı sonlandırır.
*   **`Main(HINSTANCE hInstance, LPSTR lpCmdLine)`**: İstemcinin ana mantığını yönetir.
    *   Rastgele sayı üreteci için tohum (seed) ayarlar.
    *   Loglama seviyesini ayarlar.
    *   Gerekirse yama güncelleyiciyi (`patchupdater.exe`) çalıştırır.
    *   DevIL (görüntü kütüphanesi) başlatır.
    *   `Setup(lpCmdLine)` fonksiyonunu çağırarak zamanlayıcı ve Granny3D loglama ayarlarını yapar.
    *   Konsol penceresini ve log dosyalarını açar (debug modunda).
    *   LZO sıkıştırma kütüphanesini, EterPackManager'ı ve HWIDManager'ı başlatır.
    *   `ENABLE_CONFIG_MODULE` aktifse `CPythonConfig` ile yapılandırma dosyasını (`config.cfg`) yükler.
    *   `PackInitialize("pack")` ile oyunun veri paketlerini yükler.
    *   `LocaleService_LoadGlobal` ile yerelleştirme ayarlarını yükler ve varsayılan kod sayfasını ayarlar.
    *   `CPythonApplication` örneği oluşturur ve başlatır.
    *   `CPythonLauncher` ve `CPythonExceptionSender` oluşturur.
    *   `RunMainScript(pyLauncher, lpCmdLine)` fonksiyonunu çağırarak Python ana betiğini çalıştırır.
    *   Uygulama sonlandığında kaynakları temizler (`app->Clear()`, `pyLauncher.Clear()`, `app->Destroy()`).
*   **`RunMainScript(CPythonLauncher& pyLauncher, const char* lpCmdLine)`**: Python yorumlayıcısını hazırlar ve ana Python betiğini (`system.py` veya `rootlib` üzerinden `system` modülü) çalıştırır.
    *   Gerekli tüm Python modüllerini (`initpack`, `initdbg`, `initapp`, `initsystem`, `initchr` vb.) başlatır. Bu fonksiyonlar, C++ tarafındaki çeşitli sistemleri (paket yöneticisi, grafik, ağ, karakter, envanter, görevler vb.) Python'a bağlar.
    *   `__DEBUG__`, `__USE_CYTHON__`, `__COMMAND_LINE__` gibi global Python değişkenlerini ayarlar.
    *   Ana Python sistem betiğini çalıştırır.
*   **`PackInitialize(const char* c_pszFolder)`**: Oyunun veri paketlerini (`.epk`, `.eix`) yükler. `DISABLE_INDEX_FILE` makrosuna bağlı olarak ya `Index` dosyasını okuyarak ya da sabit kodlanmış bir liste üzerinden paketleri kaydeder.
*   **`ApplicationStringTable_Initialize`, `ApplicationStringTable_GetString`, `ApplicationStringTable_GetStringz`**: Uygulama içindeki metinlerin yerelleştirilmesini yönetir. Metinleri `metin2client.dat` dosyasından veya Windows kaynaklarından yükler.
*   **`CheckPythonLibraryFilenames()`**: Temel Python kütüphane dosyalarının (`.pyc`) `lib` klasöründe bulunup bulunmadığını kontrol eder.
*   **`CreateMetin2GameMutex()`, `DestroyMetin2GameMutex()`**: Oyunun aynı anda birden fazla kopyasının çalışmasını engellemek için bir mutex oluşturur ve yönetir.
*   **`CheckMD5Filenames()`**: `ENABLE_MD5_FILE_CHECK` aktifse, önemli istemci dosyalarının MD5 karmalarını kontrol ederek değiştirilip değiştirilmediğini doğrular.
*   **`Setup(LPSTR lpCmdLine)`**: Yüksek çözünürlüklü zamanlayıcıyı (`timeBeginPeriod`) ayarlar ve Granny3D animasyon sistemi için loglama geri çağırım fonksiyonunu (`GrannyError`) belirler.

**Önemli Sistem Entegrasyonları**

*   **Python Entegrasyonu**: İstemcinin kullanıcı arayüzü ve bazı oyun mantığı Python ile yazılmıştır. Bu dosya, Python yorumlayıcısını başlatır, C++ fonksiyonlarını Python'a açar ve ana Python betiklerini çalıştırır.
*   **Paket Sistemi (`CEterPackManager`)**: Oyun varlıkları (modeller, dokular, sesler vb.) sıkıştırılmış paket dosyalarında saklanır. Bu dosya, paket yöneticisini başlatarak bu varlıklara erişimi sağlar.
*   **Yerelleştirme (`LocaleService`)**: Oyun metinleri ve bazı bölgesel ayarlar `locale.cfg` ve `metin2client.dat` gibi dosyalar aracılığıyla yönetilir.
*   **Grafik ve Ses Motorları**: Gerekli grafik (`devil.lib`, `granny2.lib`, `SpeedTreeRT.lib` vb.) ve ses (`mss32.lib`) kütüphanelerini bağlar ve başlatır.
*   **Ağ İletişimi**: Python tarafındaki ağ modüllerinin (`initnet`) başlatılmasını tetikler.
*   **Hata Yönetimi ve Loglama**: Hata durumlarını yakalar, log dosyalarına yazar (`syserr.txt`, `log.txt`) ve kullanıcıya bilgi mesajları gösterir.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, Metin2 istemcisinin "beyni" gibidir. Oyunun başlaması, çalışması ve sonlanması için gerekli olan tüm temel adımları koordine eder. Kullanıcı arayüzünün yüklenmesinden, oyun dünyasının oluşturulmasına, sunucuyla iletişimin kurulmasına kadar birçok kritik sürecin başlangıç noktasını oluşturur. Oyuncu `Metin2Client.exe`'yi çalıştırdığında, ilk olarak bu dosyadaki `WinMain` fonksiyonu devreye girer ve istemcinin yaşam döngüsünü başlatır.

---

### `UserInterface.rc`

**Dosyanın Amacı ve İşlevselliği**

`UserInterface.rc`, Metin2 istemcisinin Windows kaynaklarını tanımlayan bir kaynak betik dosyasıdır. Bu dosya, uygulamanın ikonu, imleçleri (cursor), sürüm bilgileri, iletişim kutuları (dialog box) ve çeşitli diller için yerelleştirilmiş metin dizeleri (string table) gibi kaynakları içerir. Microsoft Visual C++ derleyicisi tarafından kullanılarak bu kaynakları yürütülebilir dosyaya (`.exe` veya `.dll`) gömer.

**Temel İçerik ve Yapısı**

*   **`#include "resource.h"`**: Kaynak ID'lerinin tanımlandığı başlık dosyasını dahil eder.
*   **Dil Tanımları (`LANGUAGE`, `SUBLANG_DEFAULT`, `pragma code_page`)**: Farklı diller (Japonca, Çince, Korece, İngilizce) için kaynak bölümleri tanımlar. Her dil için uygun kod sayfası belirtilir.
*   **String Table (`STRINGTABLE`)**: Uygulama içinde kullanılacak metin dizelerini ID'lerle eşleştirir.
    *   `IDS_APP_NAME`: Uygulama başlığı (örn: "Metin 2").
    *   `IDS_POSSESSIVE_MORPHENE`: İyelik eki (örn: "'s", "의").
    *   `IDS_WARN_BAD_DRIVER`: Hatalı sürücü uyarısı.
    *   `IDS_WARN_NO_TNL`: TnL (Transform and Lighting) desteği olmayan grafik kartı uyarısı.
    *   `IDS_ERR_CANNOT_READ_FILE`: Dosya okuma hatası mesajı.
    *   `IDS_ERR_NOT_LATEST_FILE`: Dosyanın güncel olmadığına dair hata mesajı.
    *   `IDS_ERR_MUST_LAUNCH_FROM_PATCHER`: Oyunun yama (patcher) üzerinden başlatılması gerektiğine dair uyarı.
*   **Dialog (`IDD_SELECT_LOCALE DIALOGEX`)**: Korece kaynaklar altında tanımlanmış bir "Yerel Ayar Seçimi" (SELECT LOCALE) iletişim kutusu bulunur. Bu kutu, başlangıç ve çıkış butonları ile yerel ayarların listelendiği bir liste kutusu içerir. (Ancak pratikte bu iletişim kutusunun kullanılıp kullanılmadığı `UserInterface.cpp` kodundan anlaşılmalıdır.)
*   **Cursor (`IDC_CURSOR_NORMAL`, `IDC_CURSOR_ATTACK`, vb.)**: Oyun içinde kullanılacak çeşitli fare imleçlerini (`.cur`, `.ani` dosyaları) tanımlar. Normal imleç, saldırı imleci, konuşma imleci, kapı imleci gibi farklı durumlar için ayrı imleçler belirtilmiştir.
*   **Icon (`IDI_METIN2 ICON`)**: Uygulamanın ana ikonunu (`Metin2Client_01.ico`) tanımlar. Bu, görev çubuğunda, masaüstü kısayolunda vb. görünen ikondur.
*   **Version Information (`VS_VERSION_INFO`)**: Yürütülebilir dosyanın sürüm bilgilerini (dosya sürümü, ürün sürümü, şirket adı, telif hakkı vb.) tanımlar. Bu bilgiler, dosya özelliklerinde görünür.
*   **`TEXTINCLUDE` Bölümleri**: Genellikle Visual Studio tarafından yönetilen, kaynak düzenleyicinin ihtiyaç duyduğu bilgileri içerir.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, istemcinin kullanıcıya sunduğu görsel ve metinsel bazı temel unsurları sağlar:
*   **Uygulama İkonu**: Oyuncunun istemciyi tanımasını sağlar.
*   **Fare İmleçleri**: Oyuncunun fare ile etkileşimde bulunduğu nesne veya duruma göre farklı imleçler göstererek geri bildirim sağlar (örneğin, bir NPC ile konuşulabilirse konuşma balonu imleci, saldırı yapılabilecek bir düşman için kılıç imleci).
*   **Hata Mesajları ve Uyarılar**: Oyun çalışırken oluşabilecek bazı standart hata durumlarında (örn: sürücü uyumsuzluğu, güncel olmayan dosya) kullanıcıya kendi dilinde bilgi mesajları gösterilmesini sağlar.
*   **Sürüm Bilgisi**: İstemci dosyasının sürümünü ve diğer tanımlayıcı bilgilerini işletim sistemine ve kullanıcılara sunar.

Bu kaynaklar, istemcinin daha kullanıcı dostu ve yerelleştirilmiş bir deneyim sunmasına yardımcı olur.

---

### `Version.h`

**Dosyanın Amacı ve İşlevselliği**

`Version.h`, Metin2 istemcisinin C++ tarafındaki sürüm bilgilerini tanımlayan bir başlık dosyasıdır. Bu dosya, genellikle otomatik bir süreçle (örneğin, bir Python betiği olan `Version.py` tarafından) güncellenir ve istemcinin derleme zamanındaki spesifik sürüm numarasını içerir.

**Temel İçerik ve Yapısı**

*   **`#define VER_FILE_VERSION 1,4,48360,0`**: Dosya sürümünü virgülle ayrılmış dört sayı olarak tanımlar (Major, Minor, Build, Revision). Bu format, Windows kaynak dosyalarında (`.rc`) sürüm bilgisi için kullanılır.
*   **`#define VER_FILE_VERSION_STR "1.4.48360.0"`**: Dosya sürümünü bir metin dizesi olarak tanımlar. Bu, loglama veya kullanıcı arayüzünde sürüm gösterme gibi amaçlarla kullanılabilir.
*   **`int METIN2_GET_VERSION() { return 48360; }`**: İstemcinin ana sürüm numarasını (genellikle build numarası) döndüren bir inline fonksiyon tanımlar. Bu fonksiyon, kod içinde programatik olarak sürüm bilgisine erişmek için kullanılabilir.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, istemcinin hangi sürümünün kullanıldığını belirlemek için kritik öneme sahiptir.
*   **Yama (Patching) Sistemi**: Yama sunucusu, oyuncunun istemci sürümünü bu bilgiyle karşılaştırarak güncellemelerin gerekli olup olmadığını belirleyebilir.
*   **Hata Ayıklama ve Destek**: Bir sorun oluştuğunda, oyuncunun hangi istemci sürümünü kullandığını bilmek, sorunun kaynağını bulmada ve çözmede yardımcı olur.
*   **Sunucu Uyumluluğu**: Sunucu, belirli bir minimum istemci sürümünü zorunlu kılabilir. Bu dosyadaki sürüm bilgisi, istemcinin uyumlu olup olmadığını kontrol etmek için kullanılabilir.
*   **Bilgilendirme**: Sürüm numarası, oyun içinde veya log dosyalarında gösterilerek geliştiricilere ve bazen oyunculara bilgi verebilir.

`Version.py` betiği, genellikle bir sürüm kontrol sisteminden (örneğin Perforce'dan `p4 changes` komutuyla) en son değişiklik numarasını alarak bu dosyayı günceller, böylece her derleme, kaynak kodunun o anki durumuyla eşleşen bir sürüm numarasına sahip olur.

---

### `Version.py`

**Dosyanın Amacı ve İşlevselliği**

`Version.py`, Metin2 istemcisinin C++ başlık dosyası olan `Version.h`'yi otomatik olarak güncellemek için kullanılan bir Python betiğidir. Temel amacı, sürüm kontrol sisteminden (bu örnekte Perforce gibi görünüyor) en son değişiklik (changelist) numarasını alıp bu numarayı `Version.h` dosyasına yazmaktır. Bu sayede, her derlemede istemcinin sürüm bilgileri güncel kalır.

**Ana Çalışma Prensipleri**

1.  **Gerekli Modüllerin İçe Aktarılması**:
    *   `os`: İşletim sistemiyle ilgili fonksiyonlar için (bu betikte doğrudan kullanılmıyor gibi görünse de genellikle bu tür betiklerde bulunur).
    *   `re`: Düzenli ifadeler (regex) kullanarak metin işleme için. `Version.h` dosyasındaki mevcut sürüm numarasını bulmak için kullanılır.
    *   `sys`: Sistemle ilgili parametreler ve fonksiyonlar için (bu betikte doğrudan kullanılmıyor).
    *   `subprocess as sp`: Harici komutları çalıştırmak ve çıktılarını almak için. `p4 changes` komutunu çalıştırmak için kullanılır.

2.  **`P4_GetVersion(path)` Fonksiyonu**:
    *   Belirtilen yoldaki Perforce deposundan en son değişiklik numarasını almak için `p4 changes -m1 %s` komutunu çalıştırır.
    *   `sp.Popen` ile komutu çalıştırır ve çıktısını (`stdout`) yakalar.
    *   Komutun çıktısını (genellikle değişiklik numarası ve diğer bilgileri içeren bir metin) döndürür.

3.  **Mevcut Sürümün Okunması**:
    *   `try-except` bloğu içinde `Version.h` dosyasını açıp okumaya çalışır.
    *   `re.search("\d+;", oldData)` ile `Version.h` içindeki `METIN2_GET_VERSION()` fonksiyonundaki sayısal sürüm numarasını (örneğin `48360;`) bulmaya çalışır.
    *   Bulunursa, bu sayıyı `oldVersion` değişkenine atar. Dosya bulunamazsa veya desen eşleşmezse `oldVersion` 0 olarak kalır.

4.  **Yeni Sürümün Alınması**:
    *   `P4_GetVersion("..\..\...")` çağrısıyla (proje kök dizinini hedeflediği varsayılarak) Perforce'dan en son değişiklik bilgilerini alır.
    *   Gelen metni `split()` ile böler ve ikinci elemanı (genellikle değişiklik numarası) alıp `int()` ile sayıya çevirerek `newVersion` değişkenine atar.

5.  **Sürüm Karşılaştırması ve `Version.h` Güncellemesi**:
    *   `if oldVersion != newVersion:`: Eğer Perforce'dan alınan yeni sürüm numarası `Version.h` dosyasındaki mevcut sürümden farklıysa, `Version.h` dosyasını yazma modunda (`"wb"`) açar.
    *   Dosyaya `#define VER_FILE_VERSION`, `#define VER_FILE_VERSION_STR` ve `int METIN2_GET_VERSION()` satırlarını yeni sürüm numarasıyla güncelleyerek yazar.

**Dosyanın Kullanım Amacı (Derleme Sürecinde)**

Bu Python betiği, genellikle istemcinin derleme (build) sürecinin bir parçası olarak çalıştırılır:
*   **Otomatik Sürüm Numaralandırma**: Her yeni derleme yapıldığında, kaynak kodunun en son sürüm kontrol sistemi değişikliğine karşılık gelen güncel bir sürüm numarasına sahip olmasını sağlar.
*   **Tutarlılık**: `Version.h` dosyasındaki sürüm bilgisinin her zaman projenin mevcut durumuyla senkronize olmasını garantiler.
*   **İzlenebilirlik**: Derlenmiş istemci dosyalarının hangi kaynak kodu sürümüne ait olduğunun kolayca takip edilmesine yardımcı olur.

Bu betik sayesinde, geliştiricilerin `Version.h` dosyasını manuel olarak güncellemesine gerek kalmaz, bu da hata olasılığını azaltır ve geliştirme sürecini daha verimli hale getirir. 