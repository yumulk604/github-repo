# UserInterface Referans Kılavuzu - Bölüm 5

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 4'te ele alınan konuların ardından, bu bölümde kalan C++ sınıfları, Python modülleri ve diğer ilgili bileşenler incelenecektir.

## İçindekiler

* [`md5.h` (MD5 Kriptografik Karma Fonksiyonu)](#md5h-md5-kriptografik-karma-fonksiyonu)
* [`metin2client.exe.manifest` (Uygulama Manifest Dosyası)](#metin2clientexemanifest-uygulama-manifest-dosyası)
* [`minIni.c` (INI Dosyası Okuma/Yazma Kütüphanesi)](#mininic-ini-dosyası-okumayazma-kütüphanesi)
* [`MovieMan.h` ve `MovieMan.cpp` (Video Oynatma Yöneticisi)](#moviemanh-ve-moviemancpp-video-oynatma-yöneticisi)
* [`NetworkActorManager.h` ve `NetworkActorManager.cpp` (Ağ Aktör Yöneticisi)](#networkactormanagerh-ve-networkactormanagercpp-ağ-aktör-yöneticisi)
* [`Packet.h` (Ağ Paket Tanımları)](#packeth-ağ-paket-tanımları)
* [`ProcessCRC.h` ve `ProcessCRC.cpp` (Süreç ve Dosya CRC Hesaplama)](#processcrch-ve-processcrccpp-süreç-ve-dosya-crc-hesaplama)

---

### `md5.h` (MD5 Kriptografik Karma Fonksiyonu)

#### Dosyanın Genel Amacı

`md5.h` başlık dosyası, MD5 (Message Digest Algorithm 5) algoritmasının bir C++ implementasyonunu sağlar. MD5, verilen herhangi bir uzunluktaki veriden 128 bitlik (16 byte) sabit boyutlu bir karma (hash) değeri üreten yaygın bir kriptografik karma fonksiyonudur. Bu karma değer, genellikle veri bütünlüğünü doğrulamak (bir dosyanın veya mesajın değiştirilip değiştirilmediğini kontrol etmek) veya verileri benzersiz bir şekilde tanımlamak için kullanılır. Ancak, MD5'in artık kriptografik olarak güvenli olmadığı ve çarpışma (collision) saldırılarına karşı savunmasız olduğu bilinmektedir, bu nedenle güvenlik açısından hassas uygulamalarda (örneğin şifre saklama) kullanılmamalıdır.

#### Başlık Dosyası Tanımları (`md5.h`)

*   **Temel MD5 Fonksiyonları ve Sabitleri**:
    *   `F(x, y, z)`, `G(x, y, z)`, `H(x, y, z)`, `I(x, y, z)`: MD5 algoritmasının temel mantıksal fonksiyonları.
    *   `ROTATE_LEFT(x, n)`: Bir değeri sola doğru belirtilen bit sayısı kadar döndürür.
    *   `FF`, `GG`, `HH`, `II`: MD5'in dört turunda kullanılan dönüşüm makroları. Bu makrolar, temel fonksiyonları, veri bloklarını, dönüş sabitlerini ve döndürme miktarlarını birleştirir.
    *   `MD5_S11` - `MD5_S44`: Her turdaki her adım için özel döndürme miktarları.
    *   `PADDING`: MD5 algoritmasının, işlenecek veriyi 64 byte'lık bloklara tamamlamak için kullandığı standart dolgu baytları dizisi.
*   **Tür Tanımları**:
    *   `BYTE`: `unsigned char`.
    *   `POINTER`: `unsigned char*`.
    *   `UINT2`: `unsigned short int` (2 byte).
    *   `UINT4`: `unsigned long int` (4 byte).
*   **`MD5` Sınıfı**:
    *   **`struct __context_t` (özel üye)**:
        *   `state[4]`: (ABCD) 128 bitlik ara karma durumunu tutan dört adet `UINT4`.
        *   `count[2]`: İşlenen toplam bit sayısını tutan iki adet `UINT4` (64 bitlik bir sayaç).
        *   `buffer[64]`: Gelen veriyi geçici olarak saklamak için kullanılan 64 byte'lık tampon.
    *   **Statik Yardımcı Fonksiyonlar (özel)**:
        *   `MD5Transform(UINT4 state[4], unsigned char block[64])`: MD5 algoritmasının temel dönüşüm fonksiyonu. Mevcut `state`'i (karma durumunu) bir 64 byte'lık `block` verisine göre günceller.
        *   `Encode(unsigned char* output, UINT4* input, unsigned int len)`: `UINT4` dizisini `unsigned char` dizisine (little-endian formatında) dönüştürür.
        *   `Decode(UINT4* output, unsigned char* input, unsigned int len)`: `unsigned char` dizisini `UINT4` dizisine (little-endian formatında) dönüştürür.
    *   **Genel (Public) Arayüz**:
        *   `MD5()`: Kurucu metot, `Init()`'i çağırır.
        *   `void Init()`: Yeni bir MD5 işlemi başlatır, `context`'i sıfırlar ve standart başlangıç karma değerlerini (`state`) yükler.
        *   `void Update(unsigned char* input, unsigned int inputLen)`: Devam eden bir MD5 işlemine `inputLen` uzunluğundaki `input` veri bloğunu ekler ve `context`'i günceller.
        *   `void Final()`: MD5 işlemini sonlandırır. Gerekli dolguyu (padding) ekler, son dönüşümü yapar ve 16 byte'lık nihai karma değerini `digestRaw` üyesine yazar. Ardından `writeToString()`'i çağırır.
        *   `BYTE digestRaw[16]`: Hesaplanan 16 byte'lık ham MD5 karma değeri.
        *   `char digestChars[33]`: `digestRaw`'daki ham karma değerinin okunabilir onaltılık (hexadecimal) string formatına dönüştürülmüş hali (32 karakter + null sonlandırıcı).
        *   `char* digestFile(char* filename)`: Bir dosyanın içeriğinin MD5 karmasını hesaplar.
        *   `char* digestMemory(BYTE* memchunk, int len)`: Bellekteki bir veri bloğunun MD5 karmasını hesaplar.
        *   `char* digestString(char* string)`: Bir C-string'inin MD5 karmasını hesaplar.
    *   **Yardımcı Fonksiyon (özel)**:
        *   `void writeToString()`: `digestRaw`'daki 16 byte'lık ham karma değerini, `digestChars` içine 32 karakterlik onaltılık bir string olarak formatlar.

#### C++ Implementasyon Detayları

Sınıfın içindeki statik fonksiyonlar (`MD5Transform`, `Encode`, `Decode`), MD5 algoritmasının RFC 1321'de belirtilen adımlarını takip eder:

1.  **Başlatma (`Init`)**: Durum (state) değişkenleri (A, B, C, D) sabit başlangıç değerleriyle yüklenir.
2.  **Güncelleme (`Update`)**: Girdi verisi 64 byte'lık bloklar halinde işlenir. Her blok için `MD5Transform` çağrılır. Bu fonksiyon, 4 tur (round) boyunca her biri 16 adımdan oluşan karmaşık bir dizi işlem gerçekleştirir. Her adım, temel mantıksal fonksiyonlardan birini (F, G, H, I), bir alt blok verisini, bir toplama sabitini ve bir döndürme miktarını kullanır.
3.  **Sonlandırma (`Final`)**:
    *   Orijinal mesajın sonuna dolgu (padding) bitleri eklenir, böylece toplam uzunluk 512 bitin (64 byte) katından 64 bit eksik olur.
    *   Orijinal mesajın uzunluğu (64 bit olarak) eklenir.
    *   Sonuçta ortaya çıkan A, B, C, D durum değişkenleri birleştirilerek 128 bitlik MD5 karma değeri oluşturulur.
    *   `digestRaw`'a bu 16 byte'lık ham değer yazılır ve `writeToString` ile onaltılık string formatına çevrilir.

#### Oyunda Kullanım Amacı ve Senaryoları

MD5, Metin2 istemcisinde çeşitli amaçlar için kullanılabilir:

*   **Dosya Bütünlüğü Kontrolü**: Oyun istemcisi, önemli dosyaların (örneğin, veri paketleri, yapılandırma dosyaları) indirme veya güncelleme sonrasında bozulmadığını veya değiştirilmediğini doğrulamak için MD5 karmalarını kullanabilir. Sunucudan beklenen MD5 karması alınır ve yerel dosyanınkiyle karşılaştırılır.
*   **Veri Paketlerinin Kimlik Doğrulaması**: İstemci, sunucudan aldığı bazı veri paketlerinin veya mesajların bütünlüğünü kontrol etmek için kullanılabilir, ancak bu genellikle daha güvenli yöntemlerle (örneğin, HMAC) birleştirilir.
*   **Ağ Protokolleri**: Bazı eski ağ protokollerinde, basit kimlik doğrulama veya mesaj bütünlüğü için MD5 kullanılmış olabilir.
*   **Şifre Saklama (ÖNERİLMEZ)**: Geçmişte şifrelerin MD5 karmaları saklanmış olabilir, ancak bu artık güvenli bir pratik değildir. Modern sistemler bcrypt, scrypt veya Argon2 gibi daha güçlü karma algoritmaları kullanır. Metin2'nin eski bir kod tabanında bu tür bir kullanım kalmış olabilir, ancak ideal değildir.
*   **Benzersiz Kimlik Üretimi**: Belirli oyun içi veriler veya yapılandırmalar için benzersiz bir tanımlayıcı üretmek amacıyla kullanılabilir.

`md5.h` içindeki `MD5` sınıfı, bu karma işlemlerini gerçekleştirmek için kullanışlı bir sarmalayıcı (wrapper) sağlar.

### `metin2client.exe.manifest` (Uygulama Manifest Dosyası)

#### Dosyanın Genel Amacı

`metin2client.exe.manifest` dosyası, `metin2client.exe` yürütülebilir dosyası için bir uygulama manifestidir. Bu XML tabanlı dosya, Windows işletim sistemine uygulama hakkında önemli bilgiler sağlar. Bu bilgiler, uygulamanın nasıl çalıştırılacağını, hangi Windows özelliklerine ihtiyaç duyduğunu ve hangi uyumluluk ayarlarının uygulanması gerektiğini kontrol eder.

#### Dosya Yapısı ve Sözdizimi (XML)

Manifest dosyası standart XML formatındadır ve belirli bir şemayı takip eder.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <!-- Uyumluluk Bölümü -->
  <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
    <application>
      <!-- Desteklenen İşletim Sistemleri -->
      <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"/> <!-- Windows 10 -->
      <supportedOS Id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"/> <!-- Windows 8.1 -->
      <supportedOS Id="{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}"/> <!-- Windows 8 -->
      <supportedOS Id="{35138b9a-5d96-4fbd-8e2d-a2440225f93a}"/> <!-- Windows 7 -->
      <supportedOS Id="{e2011457-1546-43c5-a5fe-008deee3d3f0}"/> <!-- Windows Vista -->
    </application>
  </compatibility>
  <!-- Güven Bilgisi Bölümü -->
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v2">
    <security>
      <requestedPrivileges>
        <!-- İstenen Çalıştırma Seviyesi -->
        <requestedExecutionLevel level='requireAdministrator' uiAccess='false' />
      </requestedPrivileges>
    </security>
  </trustInfo>
</assembly>
```

#### Anahtar Elemanlar ve Kullanım Amaçları

*   **`<assembly>`**: Manifest dosyasının kök elemanıdır.
    *   `xmlns="urn:schemas-microsoft-com:asm.v1"`: XML isim alanını tanımlar.
    *   `manifestVersion="1.0"`: Manifest sürümünü belirtir.
*   **`<compatibility>`**: Uygulamanın farklı Windows sürümleriyle uyumluluğunu tanımlar.
    *   `xmlns="urn:schemas-microsoft-com:compatibility.v1"`: Uyumluluk bölümü için XML isim alanını tanımlar.
    *   **`<application>`**: Uyumluluk ayarlarının uygulama için geçerli olduğunu belirtir.
        *   **`<supportedOS Id="{GUID}"/>`**: Uygulamanın tam olarak desteklediği ve test edildiği Windows işletim sistemi sürümlerini listeler. Her `Id` özniteliği, belirli bir Windows sürümünü temsil eden benzersiz bir GUID'dir.
            *   Windows 10: `{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}`
            *   Windows 8.1: `{1f676c76-80e1-4239-95bb-83d0f6d0da78}`
            *   Windows 8: `{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}`
            *   Windows 7: `{35138b9a-5d96-4fbd-8e2d-a2440225f93a}`
            *   Windows Vista: `{e2011457-1546-43c5-a5fe-008deee3d3f0}`
            Bu liste, Windows'un uygulamayı bu sürümler için tasarlanmış gibi çalıştırmasına yardımcı olur ve eski sürümler için uygulanan uyumluluk katmanlarını (shims) devre dışı bırakabilir.
*   **`<trustInfo>`**: Uygulamanın güvenlik ve güvenilirlik bilgilerini tanımlar.
    *   `xmlns="urn:schemas-microsoft-com:asm.v2"`: Güven bilgisi bölümü için XML isim alanını tanımlar.
    *   **`<security>`**: Güvenlikle ilgili ayarları içerir.
        *   **`<requestedPrivileges>`**: Uygulamanın çalışmak için ihtiyaç duyduğu ayrıcalık seviyesini belirtir.
            *   **`<requestedExecutionLevel level='requireAdministrator' uiAccess='false' />`**:
                *   `level='requireAdministrator'`: Uygulamanın çalışabilmesi için yönetici (administrator) ayrıcalıkları gerektiğini belirtir. Eğer kullanıcı standart bir kullanıcıysa, uygulama başlatıldığında Kullanıcı Hesabı Denetimi (UAC) istemi görüntülenerek yönetici onayı istenir.
                *   `uiAccess='false'`: Uygulamanın UI otomasyonu için güvenli masaüstüne erişim gibi özel UI ayrıcalıklarına ihtiyaç duymadığını belirtir.

#### Oyuna Etkileri

Bu manifest dosyasının `metin2client.exe` üzerinde aşağıdaki önemli etkileri vardır:

1.  **Uyumluluk Modu**: `supportedOS` etiketleri sayesinde, Windows oyunu listelenen modern işletim sistemleriyle uyumlu olarak kabul eder. Bu, Windows'un gereksiz yere eski işletim sistemleri için tasarlanmış uyumluluk düzeltmeleri uygulamasını engelleyerek potansiyel performans sorunlarını veya hataları azaltabilir.
2.  **Yönetici Ayrıcalıkları**: `requestedExecutionLevel level='requireAdministrator'` ayarı, oyunun her zaman yönetici ayrıcalıklarıyla çalışmasını talep eder. Bu, oyunun sistem dosyalarına erişmesi, belirli kayıt defteri ayarlarını değiştirmesi veya bazı donanım özelliklerini (özellikle eski anti-hile sistemleri veya grafik ayarları için) tam olarak kullanabilmesi için gerekli olabilir. Eğer oyun yönetici olarak çalışmazsa, bazı işlevleri düzgün çalışmayabilir veya hiç çalışmayabilir. Kullanıcı, oyunu her başlattığında bir UAC onayı vermek zorunda kalabilir (eğer UAC aktifse ve kullanıcı standart bir hesaba sahipse).

Özetle, bu manifest dosyası, `metin2client.exe`'nin modern Windows sistemlerinde doğru şekilde çalışmasını ve gerekli sistem ayrıcalıklarına sahip olmasını sağlamak için kritik bir yapılandırma bileşenidir.

### `minIni.c` (INI Dosyası Okuma/Yazma Kütüphanesi)

#### Dosyanın Genel Amacı

`minIni.c` dosyası, INI (Initialization) dosyalarını okumak ve yazmak için platformdan bağımsız, küçük boyutlu bir C kütüphanesinin implementasyonunu içerir. INI dosyaları, genellikle program ayarlarını, yapılandırma parametrelerini ve basit verileri saklamak için kullanılan metin tabanlı dosyalardır. Tipik bir INI dosyası, bölümler ([SectionName]), anahtarlar (Key=Value) ve yorumlar (;) içerir. Bu kütüphane, bu tür dosyalardan veri okumak ve bu dosyalara veri yazmak için temel fonksiyonları sağlar. "minIni" adı, minimal bir INI kütüphanesi olduğunu ima eder ve genellikle gömülü sistemler veya kaynakların kısıtlı olduğu ortamlar için uygundur, ancak genel amaçlı uygulamalarda da kullanılabilir.

#### Temel Fonksiyonlar ve İşleyiş

Kütüphane, aşağıdaki gibi ana işlev gruplarını yerine getiren fonksiyonlar tanımlar:

*   **Değer Okuma Fonksiyonları**:
    *   `ini_gets(Section, Key, DefValue, Buffer, BufferSize, Filename)`: Belirtilen `Section` ve `Key` altındaki string değeri okur ve `Buffer`'a yazar. Değer bulunamazsa `DefValue` kullanılır.
    *   `ini_getl(Section, Key, DefValue, Filename)`: Belirtilen `Section` ve `Key` altındaki değeri long integer (uzun tamsayı) olarak okur. Hem onluk hem de onaltılık (0x ile başlayan) sayıları destekler.
    *   `ini_getf(Section, Key, DefValue, Filename)`: (Eğer `INI_REAL` tanımlıysa) Belirtilen `Section` ve `Key` altındaki değeri ondalık sayı (float/double) olarak okur.
    *   `ini_getbool(Section, Key, DefValue, Filename)`: Belirtilen `Section` ve `Key` altındaki değeri boolean (doğru/yanlış) olarak okur. "y", "yes", "t", "true", "1" gibi değerleri doğru; "n", "no", "f", "false", "0" gibi değerleri yanlış olarak yorumlar.
*   **Bölüm ve Anahtar Listeleme Fonksiyonları**:
    *   `ini_getsection(idx, Buffer, BufferSize, Filename)`: Dosyadaki `idx` numaralı bölümün adını `Buffer`'a yazar.
    *   `ini_getkey(Section, idx, Buffer, BufferSize, Filename)`: Belirtilen `Section` altındaki `idx` numaralı anahtarın adını `Buffer`'a yazar.
*   **Değer Yazma Fonksiyonları** (Eğer `INI_READONLY` tanımlı değilse):
    *   `ini_puts(Section, Key, Value, Filename)`: Belirtilen `Section` ve `Key` altına string `Value` değerini yazar. Eğer `Key` NULL ise tüm bölümü siler. Eğer `Value` NULL ise belirtilen anahtarı siler.
    *   `ini_putl(Section, Key, Value, Filename)`: Belirtilen `Section` ve `Key` altına long integer `Value` değerini yazar.
    *   `ini_putf(Section, Key, Value, Filename)`: (Eğer `INI_REAL` tanımlıysa ve `INI_READONLY` tanımlı değilse) Belirtilen `Section` ve `Key` altına ondalık sayı `Value` değerini yazar.
*   **INI Dosyasında Gezinme/Tarama Fonksiyonu** (Eğer `INI_NOBROWSE` tanımlı değilse):
    *   `ini_browse(Callback, UserData, Filename)`: INI dosyasındaki her bir anahtar/değer çifti için belirtilen `Callback` fonksiyonunu çağırır. Bu, dosyanın tüm içeriğini işlemek için kullanılabilir.
*   **Dosya G/Ç Soyaçekimi (Abstraction)**:
    *   Kütüphane, gerçek dosya okuma/yazma işlemleri için soyutlanmış fonksiyonlar kullanır: `ini_openread`, `ini_openwrite`, `ini_openrewrite`, `ini_close`, `ini_read`, `ini_write`, `ini_remove`, `ini_rename`, `ini_tell`, `ini_seek`. Bu fonksiyonların platforma veya dosya sistemine özgü implementasyonları `INI_FILETYPE` tanımına bağlı olarak (genellikle `minIni.h` içinde `#define` edilir) sağlanmalıdır. Metin2 kaynak kodunda `INI_FILETYPE` muhtemelen standart C `FILE*` olarak tanımlanmıştır ve bu fonksiyonlar `fopen`, `fclose`, `fgets`, `fputs` vb. standart C kütüphane fonksiyonlarına yönlendirilmiştir.

#### Implementasyon Detayları

*   **Karakter Kodlaması**: Kütüphane, `_UNICODE` veya `INI_ANSIONLY` gibi makrolara bağlı olarak hem ANSI hem de Unicode karakter setlerini destekleyebilir.
*   **Yorumlar ve Boşluklar**: Satır başındaki ';' veya '#' karakterlerini yorum olarak kabul eder ve satır sonundaki yorumları temizler. Değerlerin etrafındaki başındaki ve sonundaki boşlukları temizler.
*   **Değer Tırnakları**: Değerler çift tırnak içine alınmışsa, tırnakları kaldırır ve içindeki kaçış karakterlerini (örneğin `\"`) işleyebilir (`QUOTE_DEQUOTE`). Değerleri yazarken, eğer boşluk veya özel karakterler içeriyorsa otomatik olarak tırnak içine alabilir (`QUOTE_ENQUOTE`).
*   **Satır Sonları**: Farklı platformlar için satır sonu karakterlerini (`INI_LINETERM`) doğru şekilde yönetmeye çalışır.
*   **Geçici Dosya Kullanımı**: INI dosyasına yazma işlemleri genellikle güvenli bir şekilde yapılır. Mevcut dosya okunur, değişiklikler uygulanarak yeni bir geçici dosyaya yazılır ve ardından orijinal dosya silinip geçici dosyanın adı orijinal dosyanın adına değiştirilir. Bu, yazma sırasında bir hata oluşursa orijinal dosyanın bozulmasını engeller.
*   **Tampon Boyutları**: `INI_BUFFERSIZE` (genellikle `minIni.h` içinde tanımlanır) gibi sabitler, satır okuma ve işleme için kullanılan ara tamponların boyutunu belirler.

#### Oyunda Kullanım Amacı ve Senaryoları

`minIni.c` kütüphanesi, Metin2 istemcisinde çeşitli yapılandırma ve ayar dosyalarını yönetmek için idealdir:

1.  **İstemci Ayarları (`metin2.cfg` veya benzeri dosyalar)**:
    *   **Grafik Ayarları**: Çözünürlük, gölge kalitesi, görüş mesafesi, anti-aliasing gibi ayarlar INI dosyalarında saklanabilir ve oyun başlangıcında `ini_getl`, `ini_getbool` gibi fonksiyonlarla okunabilir. Kullanıcı ayarlarda değişiklik yaptığında `ini_putl`, `ini_putbool` ile kaydedilebilir.
    *   **Ses Ayarları**: Ana ses seviyesi, müzik sesi, efekt sesi gibi ayarlar.
    *   **Kontrol Ayarları**: Klavye kısayolları, fare hassasiyeti.
    *   **Arayüz Ayarları**: Pencere pozisyonları, sohbet filtresi ayarları, oyun içi ipuçlarının gösterilip gösterilmeyeceği.
2.  **Yerelleştirme Bilgileri**: Belirli arayüz metinlerinin veya bölgesel ayarların küçük bir kısmı INI dosyalarında tutulabilir (ancak daha büyük yerelleştirme verileri genellikle özel paket formatlarındadır).
3.  **Sunucu Listeleri veya Bağlantı Bilgileri**: Nadiren de olsa, bağlanılacak sunucuların listesi veya varsayılan bağlantı portları gibi bilgiler INI dosyalarında saklanabilir.
4.  **Hata Ayıklama ve Geliştirici Ayarları**: Geliştirme sırasında, belirli özellikleri açıp kapatmak veya hata ayıklama parametrelerini ayarlamak için INI dosyaları kullanılabilir.
5.  **Modül Yapılandırmaları**: İstemcinin belirli alt modülleri (örneğin, özel bir loglama sistemi, üçüncü parti bir kütüphane) kendi yapılandırmalarını INI dosyaları aracılığıyla yönetebilir.

`minIni.c` kütüphanesi, bu tür ayarların kolayca okunup yazılabilmesi için basit ve etkili bir çözüm sunar. Küçük boyutu ve az bağımlılığı sayesinde istemciye kolayca entegre edilebilir.

### `MovieMan.h` ve `MovieMan.cpp` (Video Oynatma Yöneticisi)

Bu dosyalar, Metin2 istemcisinde çeşitli video dosyalarını (örneğin, oyun içi tanıtımlar, şirket logoları, eğitim videoları) oynatmak için kullanılan `CMovieMan` singleton sınıfını tanımlar ve uygular. Microsoft DirectShow API'lerini kullanarak video dosyalarını yükler, render eder, seslerini çalar ve kullanıcı etkileşimleriyle (örneğin, videoyu atlama) ilgilenir.

#### Dosyaların Genel Amaçları

*   **`MovieMan.h`**: `CMovieMan` sınıfının arayüzünü, video oynatma ile ilgili sabitleri (örneğin, `MOVIEMAN_FADE_DURATION`), DirectShow arayüzleri için ileriye dönük bildirimleri (forward declarations) ve video codec'leri için bazı GUID sabitlerini tanımlar.
*   **`MovieMan.cpp`**: `CMovieMan` sınıfının tüm fonksiyonlarının implementasyonunu içerir. Bu, DirectDraw yüzeyleri oluşturma, DirectShow filtre grafiği oluşturma (manuel veya otomatik), video akışlarını bir yüzeye render etme, ses çalma ve video geçiş efektleri (örneğin, fade-out) gibi işlevleri kapsar.

#### Başlık Dosyası Tanımları (`MovieMan.h`)

*   **Sabitler**:
    *   `MOVIEMAN_FADE_DURATION`: Video sonunda kullanılacak fade-out efektinin süresini milisaniye cinsinden tanımlar (1300ms).
    *   `MOVIEMAN_SKIPPABLE_YES`: Bir videonun kullanıcı tarafından (örneğin, ESC tuşuyla) atlanabilir olup olmadığını belirtir (true).
    *   `MOVIEMAN_POSTEFFECT_FADEOUT`: Video sonunda uygulanacak post-efekt türünü belirtir (şu an için sadece fade-out).
*   **DirectShow Arayüzleri (İleriye Dönük Bildirimler)**:
    *   `IDirectDraw`, `IDirectDrawSurface`, `IDirectDrawMediaStream`
    *   `IGraphBuilder`, `IBasicAudio`, `IMultiMediaStream`, `IAMMultiMediaStream`
    Bu arayüzler, DirectShow ve DirectDraw kütüphanelerinin temel bileşenleridir ve video/ses işleme için kullanılır.
*   **Codec GUID'leri**:
    *   `CLSID_MP43DMOCodec`, `CLSID_MP4VideoCodec`, `CLSID_MP3AudioCodec`, `CLSID_DIrectSoundRenderer`: Belirli video ve ses codec'leri ile DirectSound renderleyici için COM Sınıf Tanımlayıcıları (CLSID). Bunlar, DirectShow filtre grafiğinde doğru codec'lerin yüklenmesi için kullanılır.
*   **`CMovieMan` Sınıfı (Singleton)**:
    *   **Kurucu (`CMovieMan()`)**: `CoInitialize(NULL)` çağırarak COM kütüphanesini başlatır. Temel üyeleri (örneğin `m_pPrimarySurface`, `m_pBasicAudio`) null olarak ayarlar.
    *   **Yıkıcı (`~CMovieMan()`)**: `CoUninitialize()` çağırarak COM kütüphanesini serbest bırakır.
    *   **Genel (Public) Metotlar**:
        *   `ClearToBlack()`: Oyun penceresini siyaha boyar.
        *   `PlayLogo(const char *pcszName)`: Belirtilen isimdeki logo videosunu oynatır.
        *   `PlayIntro()`: Tanıtım videosunu oynatır (atlanabilir, fade-out efektiyle).
        *   `PlayTutorial(LONG nIdx)`: Belirtilen indeksteki eğitim videosunu oynatır (atlanabilir, fade-out efektiyle).
    *   **Özel (Private) Üyeler**:
        *   `m_usingRGB32`: Birincil yüzeyin 32-bit RGB formatında olup olmadığını tutar.
        *   `m_movieWidth`, `m_movieHeight`: Oynatılan videonun orijinal genişlik ve yüksekliği.
        *   `m_movieRect`: Videonun ekranda kaplayacağı `RECT` alanı.
        *   `m_pPrimarySurface`: DirectDraw birincil yüzeyine işaretçi.
        *   `m_pBasicAudio`: DirectShow `IBasicAudio` arayüzüne işaretçi (ses kontrolü için).
    *   **Özel (Private) Yardımcı Metotlar**:
        *   `FillRect(RECT& fillRect, DWORD fillColor)`: Belirtilen `RECT` alanını belirtilen renkle doldurur (DirectDraw veya GDI kullanarak).
        *   `GDIFillRect(RECT& fillRect, DWORD fillColor)`: GDI kullanarak bir alanı renkle doldurur (DirectDraw başarısız olursa fallback).
        *   `GDIBlt(IDirectDrawSurface *pSrcSurface, RECT *pDestRect)`: GDI kullanarak bir yüzeyi hedef `RECT`'e çizer (DirectDraw Blt başarısız olursa fallback).
        *   `GetWindowRect(RECT& windowRect)`: Ana oyun penceresinin ekran koordinatlarındaki `RECT`'ini alır.
        *   `CalcMovieRect(int srcWidth, int srcHeight, RECT& movieRect)`: Videonun en-boy oranını koruyarak oyun penceresine sığacak şekilde video `RECT`'ini hesaplar.
        *   `CalcBackgroundRect(const RECT& movieRect, RECT& upperRect, RECT& lowerRect)`: Videonun kaplamadığı üst/alt veya sol/sağ boşlukları (`letterbox`/`pillarbox`) temsil eden `RECT`'leri hesaplar.
        *   `PlayMovie(const char *cpFileName, const bool bSkipAllowed = FALSE, const int nPostEffectID = 0, const DWORD dwPostEffectData = 0)`: Asıl video oynatma mantığını içeren ana özel fonksiyon.
        *   `BuildFilterGraphManually(...)`: Belirli codec'ler (örneğin MPG, MP43) için manuel olarak DirectShow filtre grafiği oluşturur.
        *   `RenderFileToMMStream(...)`: Bir video dosyasını `IMultiMediaStream`'e render eder.
        *   `RenderStreamToSurface(...)`: Bir `IMultiMediaStream`'i bir DirectDraw yüzeyine render eder ve oynatır.
        *   `RenderPostEffectFadeOut(...)`: Video sonunda fade-out efekti uygular.
        *   Debug için `#ifdef _DEBUG` ile korunan `AddToRot` ve `RemoveFromRot` (şu an yorum satırı).

#### C++ Implementasyon Detayları (`MovieMan.cpp`)

*   **Video Dosya Yolları**: `LOGO_PMANG_FILE`, `INTRO_FILE` gibi çeşitli `#define`'lar ile sabit video dosyası isimleri tanımlanmıştır.
*   **DirectDraw ve COM Başlatma**: `CMovieMan` kurucusunda `CoInitialize` ile COM başlatılır ve yıkıcısında `CoUninitialize` ile sonlandırılır. `PlayMovie` içinde DirectDraw nesnesi (`pDD`) oluşturulur ve birincil yüzey (`m_pPrimarySurface`) elde edilir.
*   **Pencere ve Yüzey Yönetimi**:
    *   `ClearToBlack()`: Oyun penceresini siyaha boyar.
    *   `FillRect()`: Verilen bir `RECT`'i belirli bir renkle doldurur. Önce DirectDraw `Blt` ile dener, olmazsa GDI `FillRect`'e geçer.
    *   `CalcMovieRect()`, `CalcBackgroundRect()`: Videonun ekranda doğru şekilde ortalanması ve en-boy oranının korunması için gerekli geometrik hesaplamaları yapar.
*   **Video Oynatma Mantığı (`PlayMovie`)**:
    1.  DirectDraw nesnesi ve birincil yüzey oluşturulur.
    2.  `RenderFileToMMStream` çağrılarak video dosyası bir `IMultiMediaStream`'e yüklenir.
        *   `RenderFileToMMStream` içinde `CoCreateInstance(CLSID_AMMultiMediaStream, ...)` ile bir `IAMMultiMediaStream` nesnesi oluşturulur.
        *   `Initialize`, `AddMediaStream` (video ve ses için) çağrılarak akışlar ayarlanır.
        *   Dosya uzantısına göre (`.mpg`, `.mp43` vb.) ya `BuildFilterGraphManually` ile özel codec'ler kullanılarak filtre grafiği manuel oluşturulur ya da `OpenFile` ile DirectShow'un otomatik filtre grafiği oluşturması sağlanır.
        *   `BuildFilterGraphManually`, `CoCreateInstance` ile `Source Filter`, `Splitter`, `Video Decoder`, `Audio Decoder` gibi filtreleri oluşturur ve `pGraphBuilder->Connect` ile birbirine bağlar. `QueryInterface(IID_IBasicAudio, ...)` ile ses kontrolü için `m_pBasicAudio` elde edilir.
    3.  Elde edilen `IMultiMediaStream`'den birincil video akışı (`IDirectDrawMediaStream`) alınır.
    4.  Video boyutları (`m_movieWidth`, `m_movieHeight`) alınır.
    5.  Video içeriğinin çizileceği bir off-screen DirectDraw yüzeyi (`pSurface`) oluşturulur.
    6.  `RenderStreamToSurface` çağrılarak video bu yüzeye render edilir ve oynatılır.
        *   `RenderStreamToSurface` içinde `pDDStream->CreateSample(pSurface, ...)` ile bir video sample'ı oluşturulur.
        *   `pMMStream->SetState(STREAMSTATE_RUN)` ile oynatma başlatılır.
        *   Bir döngü içinde `pSample->Update(0, NULL, NULL, NULL) == S_OK` kontrol edilerek her kare güncellenir.
        *   Her kare `m_pPrimarySurface->Blt()` ile ekrana (hesaplanan `movieRect` içine) çizilir. Başarısız olursa `GDIBlt()` kullanılır.
        *   Eğer `bSkipAllowed` ise ve kullanıcı ESC, Space veya sol fare tuşuna basarsa döngüden çıkılır.
        *   Oynatma bittikten sonra `nPostEffectID`'ye göre (örneğin `MOVIEMAN_POSTEFFECT_FADEOUT`) `RenderPostEffectFadeOut` çağrılır.
        *   `pMMStream->SetState(STREAMSTATE_STOP)` ile oynatma durdurulur.
*   **Fade-Out Efekti (`RenderPostEffectFadeOut`)**:
    1.  O anki video karesi bir ara belleğe (`pCopiedSrcSurBuf`) kopyalanır.
    2.  Belirlenen `fadeOutDuration` süresince, her adımda:
        *   Kopyalanan karedeki piksellerin renkleri, `fadeOutColor`'a doğru (alpha blending benzeri) interpolasyonla karartılır.
        *   `m_pBasicAudio->put_Volume()` ile ses seviyesi de kademeli olarak azaltılır.
        *   Sonuç, ekrana çizilir.
    3.  Süre sonunda tüm ekran `fadeOutColor` ile doldurulur.
*   **Kaynak Temizleme**: Kullanılan tüm DirectShow ve DirectDraw arayüzleri (`Release()` çağrılarak) serbest bırakılır. Mesaj kuyruğu temizlenir.

#### Oyunda Kullanım Amacı ve Senaryoları

`CMovieMan`, oyunun çeşitli noktalarında video tabanlı içerik göstermek için kullanılır:

1.  **Başlangıç Logoları**: Oyun ilk açıldığında şirket (Ymir, Gameforge vs.) veya yayıncı (EA, IAH, GameOn) logolarını gösterir. `PlayLogo()` fonksiyonu bu amaçla kullanılır. (Örn: `LOGO_PMANG_FILE "ymir.mpg"`)
2.  **Tanıtım Videosu (Intro)**: Oyuna giriş yapmadan önce genellikle bir tanıtım videosu oynatılır. `PlayIntro()` fonksiyonu `INTRO_FILE "intro.mpg"` dosyasını oynatır. Bu videolar genellikle kullanıcı tarafından atlanabilir.
3.  **Yasal Uyarı Videoları**: Bazı bölgeler için yasal uyarıları içeren videolar (`LEGAL_FILE_00 "legal00.mpg"`) gösterilebilir.
4.  **Eğitim Videoları (Tutorials)**: Oyunun belirli mekaniklerini veya özelliklerini anlatan kısa eğitim videoları (`TUTORIAL_0 "TutorialMovie\Tutorial0.mpg"`) oynatılabilir. `PlayTutorial()` fonksiyonu bu işlevi görür.

`CMovieMan`, bu videoları tam ekran veya pencereye uygun şekilde ölçekleyerek, sesleriyle birlikte senkronize bir şekilde oynatır. Kullanıcının videoyu atlama seçeneği (genellikle ESC tuşu ile) sunulabilir. Video bitiminde yumuşak bir geçiş için fade-out efekti uygulanabilir. Bu sınıf, DirectShow'un karmaşıklığını soyutlayarak video oynatma işlevini istemci içinde daha kolay kullanılabilir hale getirir.

### `NetworkActorManager.h` ve `NetworkActorManager.cpp` (Ağ Aktör Yöneticisi)

Bu dosyalar, Metin2 istemcisinde oyun dünyasındaki tüm ağ bağlantılı aktörlerin (diğer oyuncular, NPC'ler, canavarlar) durumlarını, pozisyonlarını ve görünümlerini yöneten `CNetworkActorManager` sınıfını ve ilgili veri yapılarını tanımlar ve uygular. Sunucudan gelen verileri işleyerek bu aktörlerin istemci tarafında senkronize bir şekilde oluşturulmasından, güncellenmesinden ve kaldırılmasından sorumludur. Ayrıca, ana karakterin pozisyonunu takip ederek hangi aktörlerin görünür olması gerektiğini belirler.

#### Dosyaların Genel Amaçları

*   **`NetworkActorManager.h`**:
    *   `SNetworkActorData`: Bir aktörün tüm ağ bilgilerini (VID, isim, tip, pozisyon, görünüm, etkiler, seviye vb.) içeren ana veri yapısını tanımlar.
    *   `SNetworkMoveActorData`: Bir aktörün hareket bilgilerini (hedef pozisyon, rotasyon, süre, hareket fonksiyonu) içeren yapıyı tanımlar.
    *   `SNetworkUpdateActorData`: Bir aktörün görünüm ve durum güncellemelerini (zırh, silah, saç, hız, seviye, PK modu, etkiler vb.) içeren yapıyı tanımlar.
    *   `CNetworkActorManager`: Ağ aktörlerini yöneten ana sınıfın arayüzünü tanımlar.
*   **`NetworkActorManager.cpp`**: `CNetworkActorManager` sınıfının tüm fonksiyonlarının ve `SNetworkActorData` yapısının pozisyon güncelleme ve kopyalama mantığının implementasyonunu içerir.

#### Başlık Dosyası Tanımları (`NetworkActorManager.h`)

*   **`SNetworkActorData` Yapısı**:
    *   **Temel Bilgiler**: `m_stName` (string), `m_bType` (aktör tipi), `m_dwVID` (Virtual ID), `m_dwStateFlags` (durum bayrakları, örn. ölü, baygın), `m_dwEmpireID`, `m_dwRace`, `m_bJob` (iş/sınıf, `ENABLE_GENDER_ALIGNMENT` ile).
    *   **Hareket ve Pozisyon**: `m_dwMovSpd` (hareket hızı), `m_dwAtkSpd` (saldırı hızı), `m_fRot` (rotasyon), `m_lCurX`, `m_lCurY` (mevcut pozisyon), `m_lSrcX`, `m_lSrcY` (kaynak pozisyon), `m_lDstX`, `m_lDstY` (hedef pozisyon).
    *   **Zamanlama**: `m_dwServerSrcTime` (sunucudan gelen kaynak zamanı), `m_dwClientSrcTime` (istemcinin kaynak zamanı), `m_dwDuration` (hareket süresi).
    *   **Görünüm**: `m_dwArmor`, `m_dwWeapon`, `m_dwHair`, `m_dwAcce` (aksesuar, `ENABLE_ACCE_COSTUME_SYSTEM` ile), `m_dwAura` (`ENABLE_AURA_COSTUME_SYSTEM` ile), `m_dwArrow` (ok kılıfı, `ENABLE_QUIVER_SYSTEM` ile).
    *   **Diğer Durumlar**: `m_dwOwnerVID` (sahip VID'i, örn. petler için), `m_sAlignment` (eğilim), `m_byPKMode` (PK modu), `m_dwMountVnum` (binek VNUM'u), `m_dwGuildID`, `m_bGuildLeaderGrade` (`ENABLE_GUILD_LEADER_GRADE_NAME` ile), `m_dwLevel`, `m_dwConquerorLevel` (`ENABLE_CONQUEROR_LEVEL` ile), `m_dwAIFlag` (AI bayrağı, `WJ_SHOW_MOB_INFO` ile), `m_bLanguage`, `m_bRefineElementType` (`ENABLE_REFINE_ELEMENT` ile).
    *   **Etkiler**: `m_kAffectFlags` (`CAffectFlagContainer`).
    *   **Metotlar**:
        *   `SNetworkActorData()`: Kurucu, varsayılan değerleri ayarlar.
        *   `SetDstPosition(dwServerTime, lDstX, lDstY, dwDuration)`: Hedef pozisyonu ve hareket süresini ayarlar.
        *   `SetPosition(lPosX, lPosY)`: Aktörün pozisyonunu anında ayarlar.
        *   `UpdatePosition()`: Mevcut pozisyonu kaynak ve hedef arasında enterpole eder.
        *   Kopyalama kurucusu ve atama operatörü (`__copy__`).

*   **`SNetworkMoveActorData` Yapısı**:
    *   `m_dwVID`, `m_dwTime` (hareket başlangıç zamanı), `m_lPosX`, `m_lPosY` (hedef koordinatlar), `m_fRot`, `m_dwFunc` (hareket fonksiyonu, örn. `FUNC_MOVE`), `m_dwArg` (fonksiyon argümanı), `m_dwDuration` (hareket süresi).

*   **`SNetworkUpdateActorData` Yapısı**:
    *   `m_dwVID` ve güncellenecek çeşitli alanlar: `m_dwGuildID`, `m_dwArmor`, `m_dwWeapon`, `m_dwHair`, `m_dwAcce`, `m_dwAura`, `m_dwArrow`, `m_dwMovSpd`, `m_dwAtkSpd`, `m_sAlignment`, `m_dwLevel`, `m_dwConquerorLevel`, `m_byPKMode`, `m_dwMountVnum`, `m_bGuildLeaderGrade`, `m_bLanguage`, `m_bRefineElementType`, `m_dwStateFlags`, `m_kAffectFlags`.

*   **`CNetworkActorManager` Sınıfı (`CReferenceObject`'dan türemiş)**:
    *   **Kurucu/Yıkıcı**: `CNetworkActorManager()`, `~CNetworkActorManager()`.
    *   **Genel (Public) Metotlar**:
        *   `Destroy()`: Tüm aktörleri ve yönetici durumunu temizler.
        *   `SetMainActorVID(dwVID)`: Ana oyuncu karakterinin VID'ini ayarlar.
        *   `RemoveActor(dwVID)`: Belirtilen VID'ye sahip aktörü kaldırır.
        *   `AppendActor(const SNetworkActorData& c_rkNetActorData)`: Yeni bir aktör ekler veya mevcut olanı günceller.
        *   `UpdateActor(const SNetworkUpdateActorData& c_rkNetUpdateActorData)`: Bir aktörün görünüm ve durumunu günceller.
        *   `MoveActor(const SNetworkMoveActorData& c_rkNetMoveActorData)`: Bir aktörü hareket ettirir.
        *   `SyncActor(dwVID, lPosX, lPosY)`: Bir aktörün pozisyonunu anında senkronize eder.
        *   `SetActorOwner(dwOwnerVID, dwVictimVID)`: Bir aktörün sahibini ayarlar.
        *   `Update()`: Her frame çağrılır, aktör pozisyonlarını günceller ve görünürlüklerini yönetir.
    *   **Korumalı (Protected) Üyeler**:
        *   `m_dwMainVID`: Ana oyuncu karakterinin VID'i.
        *   `m_lMainPosX`, `m_lMainPosY`: Ana karakterin son bilinen pozisyonu.
        *   `m_kNetActorDict`: `std::map<DWORD, SNetworkActorData>`, VID'leri `SNetworkActorData` yapılarına eşleyen ana veri saklama alanı.
    *   **Korumalı (Protected) Yardımcı Metotlar**:
        *   `__OLD_Update()`: Eski güncelleme mantığı (muhtemelen `Update()` tarafından çağrılır).
        *   `__UpdateMainActor()`: Ana karakterin pozisyonunu günceller.
        *   `__IsVisiblePos(lPosX, lPosY)`: Verilen pozisyonun ana karaktere göre görünür olup olmadığını kontrol eder (`CHAR_STAGE_VIEW_BOUND` sabitini kullanır).
        *   `__IsVisibleActor(const SNetworkActorData&)`: Verilen aktörün görünür olup olmadığını kontrol eder.
        *   `__IsMainActorVID(dwVID)`: Verilen VID'in ana karaktere ait olup olmadığını kontrol eder.
        *   `__RemoveAllGroundItems()`: Yerdeki tüm item'ları temizler (`CPythonItem::DeleteAllItems`).
        *   `__RemoveAllActors()`: Tüm aktörleri hem bu yöneticiden hem de `CPythonCharacterManager`'dan siler.
        *   `__RemoveDynamicActors()`: PC, NPC, Canavar, Binek olmayan (yani dinamik olarak eklenmiş) aktörleri siler.
        *   `__RemoveCharacterManagerActor(SNetworkActorData&)`: Aktörü `CPythonCharacterManager`'dan kaldırır (ana karakterse direkt, değilse fade ile).
        *   `__FindActorData(dwVID)`: Verilen VID'ye sahip `SNetworkActorData`'yı bulur (header'da var ama cpp'de implementasyonu yok gibi görünüyor, muhtemelen map erişimi direkt yapılıyor).
        *   `__AppendCharacterManagerActor(SNetworkActorData&)`: `SNetworkActorData`'dan `CInstanceBase::SCreateData` oluşturarak `CPythonCharacterManager`'a yeni bir instance ekler.
        *   `__FindActor(SNetworkActorData&)` ve `__FindActor(SNetworkActorData&, lDstX, lDstY)`: `CPythonCharacterManager`'dan bir instance bulur, yoksa ve görünürse oluşturur.
        *   `__GetCharacterManager()`: `CPythonCharacterManager::Instance()`'ı döndürür.

#### C++ Implementasyon Detayları (`NetworkActorManager.cpp`)

*   **`SNetworkActorData::UpdatePosition()`**: Zaman farkına (`dwElapsedTime`) ve toplam süreye (`m_dwDuration`) göre mevcut pozisyonu (`m_lCurX`, `m_lCurY`) kaynak (`m_lSrcX`, `m_lSrcY`) ve hedef (`m_lDstX`, `m_lDstY`) pozisyonları arasında lineer olarak enterpole eder.
*   **`SNetworkActorData::SetDstPosition()`**: Yeni bir hareket için kaynak pozisyonunu mevcut pozisyona, hedef pozisyonunu ve zamanlama bilgilerini de gelen parametrelere ayarlar.
*   **`CNetworkActorManager::Update()`**:
    *   `__UpdateMainActor()` ile ana karakterin mevcut dünya koordinatlarını (`m_lMainPosX`, `m_lMainPosY`) günceller.
    *   `m_kNetActorDict` içindeki tüm aktörler için `rkNetActorData.UpdatePosition()` çağırarak pozisyonlarını enterpole eder.
    *   Eğer bir aktör `CPythonCharacterManager`'da yoksa ve `__IsVisibleActor` kontrolünden geçerse, `__AppendCharacterManagerActor` ile oyun dünyasına eklenir.
*   **`__IsVisibleActor()` ve `__IsVisiblePos()`**: Bir aktörün veya pozisyonun ana karakterin `CHAR_STAGE_VIEW_BOUND` ile tanımlanan görüş alanı içinde olup olmadığını kontrol eder. "Duvar" tipindeki bazı özel ırklar (`IsWall(race)`) veya "her zaman göster" efektine sahip olanlar her zaman görünür kabul edilir.
*   **`AppendActor()`**:
    *   Eğer eklenen aktör ana karakterse ve binek durumu değişmiyorsa, önce `__RemoveDynamicActors()` ve `__RemoveAllGroundItems()` çağrılarak etraftaki diğer dinamik varlıklar temizlenir. Bu, genellikle yeni bir haritaya geçildiğinde veya karakter seçimi yapıldığında ana karakterin ilk kez eklendiği senaryolarda olur.
    *   Aktör verisi `m_kNetActorDict`'e eklenir/güncellenir.
    *   Eğer aktör görünürse, `__AppendCharacterManagerActor` ile `CPythonCharacterManager`'a da eklenir.
*   **`__AppendCharacterManagerActor()`**:
    *   `SNetworkActorData`'dan `CInstanceBase::SCreateData` yapısına ilgili tüm verileri kopyalar (VID, tip, seviye, ırk, pozisyon, isim, etkiler, zırh, silah vb.).
    *   Eğer aynı VID ile zaten bir instance varsa ve binek durumu değişiyorsa (örneğin attan inip binme), pozisyonun üzerine yazılmaması için eski pozisyon korunur. Eski instance silinir.
    *   `CPythonCharacterManager::CreateInstance()` ile yeni instance oluşturulur.
    *   Eğer oluşturulan instance ana karakterse, `IAbstractPlayer::SetMainCharacterIndex()` çağrılır.
    *   Eğer aktörün devam eden bir hareketi varsa (`dwElapsedTime < rkNetActorData.m_dwDuration`), `pNewInstance->PushTCPState()` ile bu hareket instance'a iletilir.
*   **`RemoveActor()`**: Aktörü hem `m_kNetActorDict`'ten hem de `__RemoveCharacterManagerActor` aracılığıyla `CPythonCharacterManager`'dan kaldırır.
*   **`UpdateActor()`**: `m_kNetActorDict`'teki ve `CPythonCharacterManager`'daki (eğer varsa) aktörün zırh, silah, saç, hız, seviye, PK modu, etkiler gibi çeşitli özelliklerini günceller.
*   **`MoveActor()`**:
    *   `SNetworkActorData`'daki hedef pozisyon, rotasyon ve zamanlama bilgilerini günceller (`rkNetActorData.SetDstPosition()`).
    *   Eğer aktör `CPythonCharacterManager`'da varsa, `pkInstFind->PushTCPState()` ile yeni hareket komutunu (hedef, rotasyon, fonksiyon, argüman, süre) instance'a iletir. Bu, instance'ın kendi içinde yumuşak bir hareket enterpolasyonu yapmasını sağlar.
*   **`SyncActor()`**: Aktörün pozisyonunu hem `SNetworkActorData`'da hem de `CInstanceBase`'de (`NEW_SyncPixelPosition()`) anında belirtilen koordinatlara ayarlar. Bu, genellikle ani pozisyon düzeltmeleri için kullanılır.
*   **`SetActorOwner()`**: Bir aktörün (örneğin bir petin veya çağrılmış bir yaratığın) sahibini ayarlar. Hem `SNetworkActorData`'da hem de `CInstanceBase`'de (`NEW_SetOwner()`) güncellenir.

#### Oyunda Kullanım Amacı ve Senaryoları

`CNetworkActorManager`, istemcinin oyun dünyasını sunucuyla senkronize tutmasının temel direğidir:

1.  **Karakter Girişi ve Harita Değişimi**: Oyuncu oyuna girdiğinde veya harita değiştirdiğinde, `AppendActor` ile ana karakter ve etraftaki görünür diğer aktörler (oyuncular, NPC'ler, canavarlar) `m_kNetActorDict`'e ve `CPythonCharacterManager` aracılığıyla oyun dünyasına eklenir.
2.  **Aktörlerin Görünürlük Yönetimi**: Oyuncu haritada hareket ettikçe, `Update` fonksiyonu ana karakterin pozisyonuna göre hangi aktörlerin görüş alanına girdiğini veya çıktığını belirler. Görüş alanına girenler `__AppendCharacterManagerActor` ile dünyaya eklenir, çıkanlar ise (dolaylı olarak `CPythonCharacterManager` tarafından yönetilir, bu sınıf direkt kaldırma yapmaz, sadece ekleme yapar) görünmez olur veya kaldırılır.
3.  **Hareket Senkronizasyonu**: Diğer aktörler hareket ettiğinde, sunucu hareket paketleri gönderir. İstemci bu paketleri aldığında `MoveActor` çağrılır. Bu fonksiyon, aktörün `CInstanceBase` nesnesine yeni hedefi ve hareket parametrelerini iletir, böylece aktör ekranda yumuşak bir şekilde hareket eder. `SNetworkActorData::UpdatePosition` ise bu yöneticinin kendi içindeki pozisyon bilgisini güncel tutar.
4.  **Görünüm ve Durum Güncellemeleri**: Oyuncuların zırhları, silahları değiştiğinde, seviye atladıklarında, yeni etkiler kazandıklarında vb. sunucu güncelleme paketleri gönderir. `UpdateActor` çağrılarak bu değişiklikler hem `SNetworkActorData`'da hem de `CInstanceBase`'de güncellenir, böylece oyuncu bu değişiklikleri görsel olarak fark eder.
5.  **Aktörlerin Eklenmesi/Kaldırılması**: Yeni bir canavar belirdiğinde veya bir oyuncu oyundan çıktığında, sunucu ilgili paketleri gönderir. `AppendActor` veya `RemoveActor` çağrılarak bu aktörler oyun dünyasına eklenir veya kaldırılır.
6.  **Pozisyon Düzeltmeleri**: Ağ gecikmeleri veya başka sebeplerle istemci ve sunucu pozisyonları arasında uyumsuzluk oluşursa, sunucu bir senkronizasyon paketi gönderebilir. `SyncActor` çağrılarak aktörün pozisyonu anında sunucudaki pozisyona çekilir.

Bu sınıf, `CPythonCharacterManager` ile yakın bir işbirliği içinde çalışır. `CNetworkActorManager` sunucudan gelen ham ağ verilerini tutar ve bu verilerin ne zaman ve nasıl `CPythonCharacterManager`'a (yani oyun dünyasındaki grafiksel temsillere) yansıtılacağına karar verir.

### `Packet.h` (Ağ Paket Tanımları)

#### Dosyanın Genel Amacı

`Packet.h` dosyası, Metin2 istemcisi ile oyun sunucusu (ve bazen istemciler arası) arasındaki ağ iletişiminde kullanılan tüm veri paketlerinin başlıklarını (header) ve yapılarının (struct) tanımlandığı merkezi başlık dosyasıdır. Oyun içindeki her türlü etkileşim, durum güncellemesi, komut ve bilgi alışverişi bu paketler aracılığıyla gerçekleştirilir. Bu dosya, ağ protokolünün istemci tarafındaki temelini oluşturur.

#### Başlık Dosyası Tanımları (`Packet.h`)

Dosyanın içeriği genel olarak şu bölümlerden oluşur:

1.  **Dahil Edilen Dosyalar**: `"../GameLib/RaceData.h"`, `"PythonNonPlayer.h"`, `"Locale_inc.h"` gibi diğer önemli başlık dosyalarını içerir. Bu, paket yapılarında kullanılacak tür tanımlarına (örneğin, karakter özellikleri, yerelleştirme sabitleri) erişim sağlar.

2.  **`TPacketHeader` Tanımı**:
    *   `typedef BYTE TPacketHeader;`: Tüm paketlerin ilk byte'ının (başlık/header) türünü `BYTE` (genellikle `unsigned char`) olarak tanımlar. Bu başlık, paketin türünü ve dolayısıyla nasıl işleneceğini belirtir.

3.  **İstemciden Sunucuya Paket Başlıkları (`enum CG_HEADERS`)**:
    *   İstemcinin sunucuya gönderdiği eylemler ve istekler için kullanılan tüm başlık sabitlerini tanımlar.
    *   Örnekler:
        *   `HEADER_CG_LOGIN`: Giriş isteği.
        *   `HEADER_CG_ATTACK`: Saldırı komutu.
        *   `HEADER_CG_CHAT`: Sohbet mesajı gönderme.
        *   `HEADER_CG_CHARACTER_MOVE`: Karakter hareket isteği.
        *   `HEADER_CG_ITEM_USE`: Eşya kullanma isteği.
        *   `HEADER_CG_SHOP`: Dükkan etkileşimi.
        *   `HEADER_CG_WHISPER`: Özel mesaj gönderme.
        *   `HEADER_CG_HANDSHAKE`: Genellikle bağlantının başlangıcında kullanılan bir el sıkışma paketi.

4.  **Sunucudan İstemciye Paket Başlıkları (`enum GC_HEADERS`)**:
    *   Sunucunun istemciye gönderdiği bilgiler, güncellemeler ve olaylar için kullanılan tüm başlık sabitlerini tanımlar.
    *   Örnekler:
        *   `HEADER_GC_CHARACTER_ADD`: Yeni bir karakterin oyun dünyasına eklendiğini bildirir.
        *   `HEADER_GC_CHARACTER_DEL`: Bir karakterin dünyadan kaldırıldığını bildirir.
        *   `HEADER_GC_CHARACTER_MOVE`: Bir karakterin hareket ettiğini bildirir.
        *   `HEADER_GC_CHAT`: Bir sohbet mesajını iletir.
        *   `HEADER_GC_LOGIN_SUCCESS3`/`LOGIN_SUCCESS4`: Başarılı girişi ve karakter listesini bildirir.
        *   `HEADER_GC_ITEM_SET`: Envanterdeki bir eşya bilgisini ayarlar.
        *   `HEADER_GC_STUN`: Bir karakterin sersemlediğini bildirir.
        *   `HEADER_GC_DEAD`: Bir karakterin öldüğünü bildirir.
        *   `HEADER_GC_PLAYER_POINTS`: Oyuncunun statü puanlarını günceller.
        *   `HEADER_GC_HANDSHAKE`: Genellikle bağlantının başlangıcında sunucudan gelen bir el sıkışma yanıtı veya isteği.

5.  **Genel Sabitler ve Boyut Tanımları (`enum`)**:
    *   Oyun içinde kullanılan çeşitli maksimum değerleri ve sınırları tanımlar.
    *   Örnekler:
        *   `ID_MAX_NUM`, `PASS_MAX_NUM`: Kullanıcı adı ve şifre maksimum uzunlukları.
        *   `CHAT_MAX_NUM`: Maksimum sohbet mesajı uzunluğu.
        *   `PLAYER_PER_ACCOUNT3`/`PLAYER_PER_ACCOUNT4`: Hesap başına karakter sayısı.
        *   `SHOP_HOST_ITEM_MAX_NUM`: Bir dükkandaki maksimum eşya sayısı.
        *   `GUILD_NAME_MAX_LEN`: Lonca adı maksimum uzunluğu.
        *   `WEAR_MAX_NUM`: Giyilebilir eşya yuvası sayısı (`CItemData::WEAR_MAX_NUM` ile eşlenik).

6.  **Paket Yapıları (`struct`)**:
    *   `#pragma pack(push)` ve `#pragma pack(1)` direktifleri arasında tanımlanırlar. Bu, yapıların üyelerinin bellekte bitişik olarak (aralarında boşluk olmadan) yerleştirilmesini sağlar. Bu, ağ üzerinden gönderilen verinin boyutunu optimize etmek ve farklı platformlar/derleyiciler arasında tutarlılık sağlamak için kritik öneme sahiptir.
    *   Her bir `CG_HEADER` ve `GC_HEADER` için genellikle karşılık gelen bir `typedef struct` tanımlanır. Bu yapılar, paketin başlığını ve taşıdığı verileri içerir.
    *   **Örnek CG Paket Yapıları**:
        *   `TPacketCGLogin`: Giriş için kullanıcı adı ve şifreyi içerir.
        *   `TPacketCGAttack`: Saldırı tipi, hedef VID'i ve bazı CRC bilgileri içerir.
        *   `TPacketCGChat`: Sohbet mesajının uzunluğunu ve türünü içerir (asıl mesaj genellikle bu paketi takip eden ham veri olarak gönderilir).
        *   `TPacketCGMove`: Hareket fonksiyonu, argümanı, rotasyon, hedef koordinatlar ve süreyi içerir.
        *   `TPacketCGItemUse`: Kullanılacak eşyanın envanter pozisyonunu içerir.
    *   **Örnek GC Paket Yapıları**:
        *   `TPacketGCCharacterAdd`: Dünyaya eklenecek karakterin VID'i, pozisyonu, açısı, tipi, ırk numarası, hızları, durum bayrakları ve etki bayraklarını içerir.
        *   `TPacketGCChat`: Sohbet mesajının boyutu, türü, gönderen VID'i ve imparatorluğunu içerir (asıl mesaj genellikle bu paketi takip eden ham veri olarak gönderilir).
        *   `TPacketGCPlayerPoints`: Oyuncunun tüm temel statü ve puanlarını (HP, SP, STR, INT vb.) içeren bir dizi (`points[POINT_MAX_NUM]`) barındırır.
        *   `TPacketGCItemSet`: Bir envanter/ekipman yuvasına yerleştirilecek eşyanın detaylarını (VNUM, adet, soketler, efsunlar vb.) içerir.

7.  **Önişlemci Direktifleri (`#if defined(...)`)**: Dosya içinde çok sayıda `#if defined(ENABLE_FEATURE_XYZ)` gibi direktif bulunur. Bunlar, istemcinin derleme zamanında belirli sistemlerin (örneğin, `ENABLE_GEM_SYSTEM`, `ENABLE_ACCE_COSTUME_SYSTEM`, `ENABLE_GROWTH_PET_SYSTEM`) aktif olup olmamasına bağlı olarak ilgili paket başlıklarının ve yapılarının derlenip derlenmeyeceğini kontrol eder. Bu, farklı sunucu yapılandırmaları veya istemci sürümleri için esneklik sağlar.

#### C++ Implementasyon Detayları

*   **Veri Türleri**: Paket yapılarında `BYTE`, `WORD`, `DWORD`, `char[]`, `float`, `long` gibi temel C++ veri tipleri kullanılır.
*   **Hizalama (`#pragma pack(1)`)**: Bu direktif, yapı üyeleri arasında derleyicinin ekleyebileceği boşlukları (padding) kaldırarak yapı boyutunu küçültür. Ağ iletişimi için standart bir pratiktir.
*   **Dinamik Boyutlu Paketler**: Bazı paketler (örneğin sohbet paketleri) sabit bir başlık yapısını takiben değişken uzunlukta veri içerebilir. Bu tür paketlerde, yapı genellikle verinin uzunluğunu belirten bir alan içerir (`WORD size` veya `WORD length` gibi).

#### Oyunda Kullanım Amacı ve Senaryoları

`Packet.h`, istemcinin sunucuyla etkileşim kurduğu her senaryoda temel bir rol oynar:

*   **Giriş ve Karakter Yönetimi**: Oyuna giriş yapma, karakter seçme, oluşturma, silme.
*   **Dünya Etkileşimi**: Hareket etme, diğer karakterleri görme, NPC'lerle konuşma, canavarlara saldırma.
*   **Envanter ve Eşya Yönetimi**: Eşya kullanma, taşıma, düşürme, toplama, ticaret yapma, dükkanları kullanma.
*   **Sosyal Etkileşimler**: Sohbet etme, özel mesaj gönderme, parti kurma, lonca işlemleri.
*   **Durum Güncellemeleri**: Karakterin can, mana, tecrübe puanı gibi değerlerinin güncellenmesi, etkilerin (buff/debuff) uygulanması.
*   **Sistem Mesajları**: Sunucudan gelen duyurular, görev bilgileri.

Özetle, `Packet.h` dosyası, Metin2 ağ protokolünün istemci tarafındaki sözleşmesidir. Sunucu ve istemci, bu dosyada tanımlanan formatlara göre veri gönderip alarak oyun dünyasının senkronize ve etkileşimli kalmasını sağlar. Bu dosyadaki herhangi bir değişiklik, sunucu tarafındaki karşılık gelen tanımlarla uyumlu olmalıdır, aksi takdirde iletişim hataları ve oyun sorunları ortaya çıkar.

### `ProcessCRC.h` ve `ProcessCRC.cpp` (Süreç ve Dosya CRC Hesaplama)

Bu dosyalar, çalışan Metin2 istemci sürecinin (process) bellekteki imajının ve disktaki yürütülebilir dosyasının (.exe) CRC32 (Cyclic Redundancy Check) değerlerini hesaplamak için fonksiyonlar sağlar. Ayrıca, bu CRC değerlerinden türetilen ve basit bir anti-hile mekanizmasının parçası olarak kullanılabilecek "sihirli küp" (magic cube) adı verilen veri parçalarını üretir.

#### Dosyaların Genel Amaçları

*   **`ProcessCRC.h`**: İlgili fonksiyonların (`GetExeCRC`, `BuildProcessCRC`, `GetProcessCRCMagicCubePiece`) prototiplerini deklare eder.
*   **`ProcessCRC.cpp`**: Bu fonksiyonların implementasyonunu içerir. Temel olarak, Windows API (`Toolhelp32Snapshot` ve `ReadProcessMemory`) kullanarak işlem belleği hakkında bilgi toplar, CRC32 hesaplamaları yapar ve bu CRC'leri bir XOR tablosuyla birleştirerek "sihirli küp" parçalarını oluşturur.

#### Başlık Dosyası Tanımları (`ProcessCRC.h`)

*   `extern bool GetExeCRC(DWORD& r_dwProcCRC, DWORD& r_dwFileCRC);`: Çalışan sürecin bellekteki CRC'sini (`r_dwProcCRC`) ve disktaki .exe dosyasının CRC'sini (`r_dwFileCRC`) hesaplar ve döndürür.
*   `extern void BuildProcessCRC();`: `GetExeCRC`'yi çağırarak elde edilen CRC değerlerinden `abCRCMagicCube` dizisini doldurur.
*   `extern BYTE GetProcessCRCMagicCubePiece();`: `abCRCMagicCube` dizisinden ve `abCRCXorTable` dizisinden bir sonraki "sihirli küp" parçasını hesaplayıp döndürür ve bir sonraki parça için indeksi günceller.

#### C++ Implementasyon Detayları (`ProcessCRC.cpp`)

*   **Statik Global Değişkenler**:
    *   `static BYTE abCRCMagicCube[8]`: Süreç ve dosya CRC'lerinden türetilen 8 byte'lık bir dizi. Bu, "sihirli küpün" temelini oluşturur.
    *   `static BYTE abCRCXorTable[8]`: `abCRCMagicCube` ile XOR'lanacak sabit bir 8 byte'lık dizi. Bu, basit bir karıştırma (obfuscation) adımıdır.
    *   `static BYTE bMagicCubeIdx`: `abCRCMagicCube` ve `abCRCXorTable` dizilerinde bir sonraki kullanılacak byte'ın indeksini tutar.

*   **`stristr(const char* big, const char* little)` Fonksiyonu**:
    *   Standart `strstr` fonksiyonunun büyük/küçük harf duyarsız (_case-insensitive_) bir versiyonudur. Verilen `big` string içinde `little` string'ini arar.

*   **`GetProcessInformation(std::string& exeFileName, LPCVOID* ppvAddress)` Fonksiyonu**:
    1.  `CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, GetCurrentProcessId())` ile mevcut sürece ait modüllerin bir anlık görüntüsünü alır.
    2.  `GetExcutedFileName()` (muhtemelen `GameLib` veya `EterLib` içinde tanımlı bir yardımcı fonksiyon) ile çalışan .exe dosyasının adını alır.
    3.  `Module32First` ve `Module32Next` ile modül listesini dolaşır.
    4.  Her modülün yolunda (`me32.szExePath`) çalışan .exe dosyasının adını `stristr` ile arar.
    5.  Eşleşme bulunursa, modülün yolunu `exeFileName`'e, modülün bellekteki başlangıç adresini (`me32.modBaseAddr`) `ppvAddress`'e atar ve `true` döndürür.
    6.  Bulunamazsa `false` döndürür.

*   **`GetProcessMemoryCRC(LPCVOID c_pvBaseAddress)` Fonksiyonu**:
    1.  `GetCurrentProcess()` ile mevcut sürecin bir tanıtıcısını (handle) alır.
    2.  1MB boyutunda bir tampon (`pBuf`) ayırır.
    3.  `ReadProcessMemory()` kullanarak, verilen `c_pvBaseAddress` (genellikle .exe'nin bellekteki başlangıç adresi) adresinden başlayarak 1MB'a kadar olan bellek içeriğini `pBuf`'a okur.
        *   `ERROR_PARTIAL_COPY` hatası alınırsa (genellikle okunacak bölge 1MB'den küçükse veya erişim sorunları varsa), bunu bir hata olarak kabul etmez ve okunan kadarını işler.
    4.  Eğer okuma başarılıysa, okunan `dwBytesRead` kadar verinin CRC32 değerini `GetCRC32()` (muhtemelen `EterBase` içinde tanımlı bir CRC32 hesaplama fonksiyonu) ile hesaplar ve döndürür.
    5.  Hata durumunda 0 döndürür.

*   **`__GetExeCRC(DWORD& r_dwProcCRC, DWORD& r_dwFileCRC)` Fonksiyonu**:
    1.  `GetProcessInformation()` ile .exe dosyasının adını ve bellekteki başlangıç adresini alır.
    2.  Eğer bilgiler alınabildiyse, `GetProcessMemoryCRC()` ile bellekteki sürecin CRC'sini (`r_dwProcCRC`) hesaplar. Aksi halde `r_dwProcCRC`'yi 0 yapar.
    3.  `GetFileCRC32()` (muhtemelen `EterBase` içinde tanımlı) ile .exe dosyasının diskteki halinin CRC'sini (`r_dwFileCRC`) hesaplar.
    4.  `true` döndürür.

*   **`BuildProcessCRC()` Fonksiyonu**:
    1.  `LocaleService_IsHONGKONG()` veya `LocaleService_IsTAIWAN()` kontrolü yapar. Eğer bu bölgelerden biri aktifse, CRC hesaplamasını atlar, `abCRCMagicCube` dizisini sıfırlar ve fonksiyondan çıkar. Bu, belirli bölgelerde bu anti-hile mekanizmasının devre dışı bırakıldığını gösterir.
    2.  `__GetExeCRC()` ile süreç ve dosya CRC'lerini alır.
    3.  Eğer CRC'ler başarıyla alınmışsa:
        *   `abCRCMagicCube` dizisinin 8 byte'ını, süreç CRC'sinin ve dosya CRC'sinin 4'er byte'ını (önce düşük byte'lar, sonra yüksek byte'lar) alarak doldurur.
        *   Örneğin: `abCRCMagicCube[0] = LSB of ProcCRC`, `abCRCMagicCube[1] = LSB of FileCRC`, `abCRCMagicCube[2] = 2nd LSB of ProcCRC` vb.
    4.  `bMagicCubeIdx`'i 0'a ayarlar.

*   **`GetProcessCRCMagicCubePiece()` Fonksiyonu**:
    1.  `abCRCMagicCube` dizisinden `bMagicCubeIdx` indeksindeki byte'ı alır.
    2.  Bu byte'ı `abCRCXorTable` dizisinden yine `bMagicCubeIdx` indeksindeki byte ile XOR'lar.
    3.  `bMagicCubeIdx`'i bir artırır. Eğer 7'yi geçerse (yani 8 olursa), tekrar 0'a döner (dairesel bir şekilde diziyi kullanır).
    4.  XOR'lanmış sonucu (`bPiece`) döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyaların temel amacı, istemcinin bellekte veya diskte yetkisiz bir şekilde değiştirilip değiştirilmediğini tespit etmeye yönelik basit bir kontrol mekanizması sağlamaktır:

1.  **Başlangıçta CRC Hesaplama**: Oyun istemcisi başladığında, `BuildProcessCRC()` fonksiyonu çağrılarak hem çalışan sürecin bellekteki ilk 1MB'lık kısmının hem de diskteki `.exe` dosyasının CRC32 değerleri hesaplanır. Bu değerler, `abCRCMagicCube` adlı bir diziye özel bir formatla saklanır.

2.  **"Sihirli Küp" Parçalarının Üretimi**: `GetProcessCRCMagicCubePiece()` fonksiyonu, `abCRCMagicCube` dizisindeki byte'ları sırayla alıp, sabit bir XOR tablosuyla (`abCRCXorTable`) birleştirerek 8 byte'lık "sihirli küp" parçaları üretir. Bu parçalar, muhtemelen sunucuya gönderilen bazı önemli paketlere (örneğin, saldırı paketi `TPacketCGAttack` içinde `bCRCMagicCubeProcPiece` ve `bCRCMagicCubeFilePiece` olarak) eklenir.

3.  **Sunucu Tarafı Doğrulama (Tahmini)**: Sunucu, istemciden gelen bu "sihirli küp" parçalarını alır. Sunucu da aynı mantıkla istemcinin orijinal `.exe` dosyasının CRC'sini ve beklenen bellek CRC'sini (veya bir tolerans aralığını) biliyor olabilir. Gelen parçaları kendi hesapladığı değerlerle karşılaştırarak veya parçaların beklenen bir deseni takip edip etmediğini kontrol ederek istemcinin değiştirilip değiştirilmediği hakkında bir çıkarım yapabilir.

4.  **Hile Tespiti**: Eğer istemcinin belleği bir hile programı tarafından değiştirilmişse (örneğin, kod enjeksiyonu yapılmışsa) veya `.exe` dosyası modifiye edilmişse, hesaplanan CRC değerleri orijinalinden farklı olacaktır. Bu da `GetProcessCRCMagicCubePiece()` tarafından üretilen parçaların sunucunun beklediği değerlerden farklı olmasına yol açabilir, böylece potansiyel bir hile kullanımı tespit edilebilir.

**Sınırlamalar ve Notlar**:
*   Bu yöntem, CRC32'nin çarpışma (collision) olasılığı ve bellek taramasının sadece ilk 1MB ile sınırlı olması gibi nedenlerle sofistike hilelere karşı çok güçlü bir koruma sağlamaz.
*   `LocaleService` kontrolü, bu mekanizmanın bazı bölgelerde (Hong Kong, Tayvan) devre dışı bırakıldığını gösterir; bunun nedeni yasal kısıtlamalar veya farklı anti-hile stratejileri olabilir.
*   `abCRCXorTable` kullanımı, CRC değerlerini basitçe gizlemeye yönelik bir adımdır.

Özetle, `ProcessCRC.h` ve `ProcessCRC.cpp`, istemcinin bütünlüğünü doğrulamak ve temel düzeyde hile tespitine yardımcı olmak için CRC kontrolleri ve bu kontrollerden türetilen veri parçalarını kullanan bir sistemin parçasıdır.

---

*(Bu bölümdeki içerikler tamamlanmıştır. Devamı için [UserInterface Referans Kılavuzu - Bölüm 6](./client_UserInterface_Referans_Part6.md) dosyasına bakınız.)*