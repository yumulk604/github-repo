# libthecore Referans Kılavuzu

Bu dosya, `srcServer/Source/libthecore` kütüphanesinin amacını, içerdiği temel bileşenleri ve fonksiyonları açıklamaktadır.

`libthecore`, Metin2 sunucusunun en temel çekirdek işlevlerini (ağ iletişimi, olay yönetimi, temel veri yapıları vb.) sağlayan kütüphanedir.

---

## `include/` Klasörü Dosyaları

Bu klasör, `libthecore` kütüphanesinin dışarıya sunduğu başlık dosyalarını (.h) içerir. Diğer modüller bu dosyaları kullanarak `libthecore` fonksiyonlarına ve yapılarına erişir.

### `buffer.h`

*   **Amaç:** Dinamik boyutlu bellek tamponları (`BUFFER`) oluşturmak, yönetmek ve veri okuyup yazmak için fonksiyonlar sağlar.
*   **Temel Özellikler:** Tampon oluşturma/silme/sıfırlama, boyut/boş alan sorgulama, boyut ayarlama, ham veri yazma/okuma, belirli türlerde (byte, word, dword) veri okuma, okuma/yazma pozisyonunu ilerletmeden veri önizleme (peek) ve ardından ilerletme (proceed).
*   **Kullanım Alanı:** Ağ paketi oluşturma/parçalama, genel veri akışı yönetimi.

### `crypt.h`

*   **Amaç:** Simetrik blok şifreleme algoritmaları için şifreleme ve şifre çözme fonksiyonları bildirir.
*   **Desteklenen Algoritmalar:** TEA, GOST, DES.
*   **Kullanım Alanı:** Ağ trafiğini veya hassas verileri şifrelemek.

### `DES_table.h`

*   **Amaç:** `crypt.h` içinde bildirilen DES algoritmasının iç uygulaması tarafından kullanılan statik arama tablolarını (S-kutuları, anahtar permütasyonları) içerir.
*   **Not:** Bu dosya doğrudan dış kullanıma yönelik bir arayüz sunmaz.

### `fdwatch.h`

*   **Amaç:** Dosya tanıtıcılarını (genellikle soketler) G/Ç olayları (okuma/yazma) için izlemek amacıyla platforma özel (Windows için `select`, diğerleri için `kqueue`) G/Ç multiplexing mekanizmaları için bir soyutlama katmanı sağlar.
*   **Temel Özellikler:** İzleyici oluşturma/silme, izlenecek dosya tanıtıcısı ekleme/silme, olayları bekleme (`fdwatch`), olayları kontrol etme/temizleme, olayla ilişkili veriyi alma.
*   **Kullanım Alanı:** Sunucunun asenkron ağ olay döngüsünün (event loop) temelini oluşturur, birden fazla bağlantıyı verimli bir şekilde yönetir.

### `hangul.h`

*   **Amaç:** Korece karakterler (Hangul) üzerinde işlem yapmak için makrolar (`ishan` vb.) ve yardımcı fonksiyonlar (`check_han`, `under_han` vb.) içerir.
*   **Kullanım Alanı:** Korece metinlerin sunucu tarafında doğru işlenmesi (sohbet, isimler vb.).

### `heart.h`

*   **Amaç:** Periyodik olarak belirli bir fonksiyonu (`HEARTFUNC`) çağıran bir "kalp atışı" (heartbeat) veya ana döngü zamanlayıcı mekanizması sağlar.
*   **Temel Özellikler:** Kalp atışı nesnesi (`HEART`) oluşturma/silme (`std::unique_ptr` ile yönetilir), bekleme süresi hesaplama/bekleme (`heart_idle`), kalp atışını ilerletip fonksiyonu tetikleme (`heart_beat`).
*   **Kullanım Alanı:** Sunucunun ana olay döngüsü, periyodik görevler, zaman aşımları.

### `kstbl.h`

*   **Amaç:** KS X 1001 standardındaki 2350 Hangul karakteriyle ilgili bir arama tablosu (`KStbl`) bildirir.
*   **Kullanım Alanı:** Korece karakter işleme rutinleri tarafından kullanılır.

### `log.h`

*   **Amaç:** Sunucu için genel bir loglama sistemi arayüzü sağlar.
*   **Temel Özellikler:** Log sistemini başlatma/kapatma, log dosyalarını döndürme, log seviyelerini ayarlama, log saklama süresini belirleme, sistem (`sys_log`) ve hata (`sys_err` makrosu ile) logları yazma.
*   **Kullanım Alanı:** Olayları, hataları ve bilgileri kaydetmek, hata ayıklama ve sunucu izleme.

### `main.h`

*   **Amaç:** `libthecore` kütüphanesinin ana yaşam döngüsünü (başlatma, ana döngü, kapatma) ve zamanlama mekanizmasını yöneten fonksiyonları/değişkenleri bildirir.
*   **Temel Özellikler:** Kütüphaneyi başlatma (`thecore_init`), ana döngü (`thecore_idle`), kapatma/yok etme (`thecore_shutdown`, `thecore_destroy`), kalp atışı (`thecore_heart`, `thecore_pulse`), zamanlama (`thecore_time`), kapatma durumu kontrolü (`thecore_is_shutdowned`).
*   **Kullanım Alanı:** Sunucu uygulamasının ana iskeletini ve olay döngüsünü oluşturur.

### `memcpy.h`

*   **Amaç:** Platforma özel, potansiyel olarak optimize edilmiş bellek kopyalama (`memcpy`) işlevi sağlar.
*   **Temel Özellikler:** Windows dışı sistemlerde çalışma zamanında en iyi `memcpy` implementasyonunu seçme mekanizması sunabilir (`thecore_memcpy` fonksiyon işaretçisi), Windows'ta standart `memcpy` kullanılır.
*   **Kullanım Alanı:** Bellek kopyalama performansını optimize etmek.

### `signal.h`

*   **Amaç:** İşletim sistemi sinyallerini yakalamak ve yönetmek için fonksiyonlar sağlar.
*   **Temel Özellikler:** Sinyal işleyicilerini ayarlama (`signal_setup`), sinyallerle ilgili zamanlayıcıları yönetme (`signal_timer_enable`/`disable`).
*   **Kullanım Alanı:** Sunucunun düzgün kapatılması, çökme yönetimi, dış kontrol sinyallerine yanıt verme.

### `socket.h`

*   **Amaç:** TCP ve UDP ağ soketlerini yönetmek için temel fonksiyonları sağlar.
*   **Temel Özellikler:** Soket oluşturma, bağlama (`bind`), bağlantı kabul etme (`accept`), bağlanma (`connect`), veri okuma/yazma (`read`/`write`), soket kapatma (`close`), çeşitli soket seçeneklerini ayarlama (bloklama, tampon boyutları, linger vb.).
*   **Kullanım Alanı:** Sunucunun tüm ağ iletişiminin temelini oluşturur.

### `stdafx.h`

*   **Amaç:** Ön derlenmiş başlık (precompiled header) dosyası. Sık kullanılan sistem ve kütüphane başlıklarını ve `libthecore`'un diğer tüm başlıklarını içererek derleme sürelerini optimize eder.
*   **Not:** Genellikle `.cpp` dosyalarında ilk include edilen dosyadır.

### `typedef.h`

*   **Amaç:** Platformlar arası tutarlılık için temel veri türleri ve takma adlar (alias) tanımlar.
*   **Temel Özellikler:** `DWORD`, `BYTE`, `WORD`, `socket_t`, sabit boyutlu tamsayılar (`int32_t` vb.) gibi türleri platforma göre tanımlar.
*   **Kullanım Alanı:** Kod tabanında tutarlı temel tür kullanımını sağlar.

### `utils.h`

*   **Amaç:** Genel amaçlı yardımcı fonksiyonlar ve makrolar koleksiyonu sunar.
*   **Temel Özellikler:** Güvenli bellek yönetimi makroları, string işleme fonksiyonları, zaman/tarih yardımcıları, matematiksel fonksiyonlar (min/max, random), liste yönetimi makroları, hata ayıklama araçları (`printdata`, `core_dump`).
*   **Kullanım Alanı:** Kod tekrarını azaltmak ve yaygın görevleri basitleştirmek.

### `xdirent.h`

*   **Amaç:** POSIX uyumlu dizin gezinme fonksiyonlarını (`opendir`, `readdir` vb.) Windows platformu için sağlar.
*   **Kullanım Alanı:** Platformdan bağımsız dizin işlemleri.

### `xgetopt.h`

*   **Amaç:** Komut satırı argümanlarını ayrıştırmak için `getopt` fonksiyonunu sağlar.
*   **Kullanım Alanı:** Sunucu başlangıç parametrelerini işlemek.

### `xmd5.h`

*   **Amaç:** MD5 özet (hash) algoritmasını hesaplamak için fonksiyonlar sağlar.
*   **Temel Özellikler:** Standart MD5 adımları (`MD5Init`, `MD5Update`, `MD5Final`) ve dosya/veri için yardımcı fonksiyonlar (`lutil_md5_file`, `lutil_md5_data`).
*   **Kullanım Alanı:** Veri bütünlüğü kontrolü, dosya doğrulama.

---

## `lib/` Klasörü

*   **Amaç:** `libthecore` kütüphanesinin **derlenmiş statik kütüphane dosyasını** içerir.
*   **İçerik:** `libthecore.a` (Unix/Linux için statik kütüphane).
*   **Kullanım:** Ana sunucu uygulamaları derlenirken linkleme aşamasında kullanılır. 

---

## `src/` Klasörü

*   **Amaç:** Bu klasör, `include/` klasöründe bildirilen fonksiyonların ve yapıların gerçek C kaynak kodu uygulamalarını (`.c` dosyaları) içerir. Kütüphanenin asıl mantığı ve işlevselliği bu dosyalarda yer alır.
*   **Not:** Bu klasörde ayrıca `.o` uzantılı derlenmiş nesne dosyaları ve derleme sürecini yöneten `Makefile` gibi dosyalar da bulunabilir. 

### `buffer.c`

*   **Implementasyon Detayları:**
    *   Sık kullanılan boyutlar için bellek havuzu (`normalized_buffer_pool`) kullanarak tampon oluşturma/silme işlemlerini optimize eder.
    *   Yazma sırasında yeterli alan yoksa `realloc` kullanarak tampon boyutunu dinamik olarak artırır (`buffer_realloc`).
    *   Okuma/yazma işlemleri için işaretçi (`read_point`, `write_point`) ve uzunluk (`length`) takibi yapar.
    *   Okuma işlemi tampon sonuna geldiğinde tamponu sıfırlar (`buffer_reset`).
    *   Basit veri türlerini (byte, word, dword) okumak için kolaylaştırıcı fonksiyonlar içerir (hizalama sorunlarına dikkat edilmeli).
    *   Bellek havuzunu yönetmek için yardımcı fonksiyonlar içerir (`buffer_pool_free`, `buffer_larger_pool_free`). 

### `des.c`

*   **Implementasyon Detayları:**
    *   `crypt.h` içinde bildirilen DES (Data Encryption Standard) blok şifreleme algoritmasını uygular.
    *   Performans için optimize edilmiş, bit düzeyinde işlemler ve önceden hesaplanmış S-kutuları (`DES_table.h`) kullanır.
    *   Ana fonksiyonlar: `DES_ECB_mode` (ECB modunda şifreleme/çözme), `DES_Encrypt` (CBC modunda şifreleme), `DES_Decrypt` (CBC modunda şifre çözme).
    *   İşlemci mimarisinden bağımsızlık için `BYTES_TO_DWORD`/`DWORD_TO_4BYTES` makrolarını içerir.

### `fdwatch.c`

*   **Implementasyon Detayları:**
    *   `fdwatch.h`'deki asenkron G/Ç olay izleme arayüzünü gerçekleştirir.
    *   Platforma bağlı olarak verimli olay bildirimi için `kqueue` (BSD/macOS) veya standart `select` (Windows/Diğer) mekanizmalarını kullanır.
    *   Ana fonksiyonlar: `fdwatch_new` (izleyici oluşturma), `fdwatch` (olayları bekleme), `fdwatch_add_fd` (izlenecek soket ekleme), `fdwatch_del_fd` (soket çıkarma), `fdwatch_check_event` (gelen olayı kontrol etme).
    *   Windows için Winsock başlatma/kapatma (`win32_init`/`deinit`) içerir.
    *   Sunucunun çok sayıda bağlantıyı verimli yönetmesini sağlayan olay döngüsünün temelini oluşturur.

### `gost.c`

*   **Implementasyon Detayları:**
    *   `crypt.h` içinde bildirilen GOST 28147-89 blok şifreleme algoritmasını uygular.
    *   Optimize edilmiş S-kutuları (`k1`-`k8`, birleştirilmiş `k87` vb.) ve 32 raundluk Feistel yapısını kullanır.
    *   `GOST_Init` fonksiyonu ile S-kutusu tabloları hazırlanmalıdır.
    *   Ana fonksiyonlar: `GOST_Encrypt` (CFB benzeri modda şifreleme), `GOST_Decrypt` (CFB benzeri modda şifre çözme).

### `hangul.c`

*   **Implementasyon Detayları:**
    *   `hangul.h`'de bildirilen Korece (Hangul) karakter işleme fonksiyonlarını gerçekleştirir.
    *   EUC-KR karakter kodlamasını temel alır.
    *   Ana fonksiyonlar:
        *   `is_hangul`: Verilen 2 byte'lık dizinin geçerli bir Hangul karakteri olup olmadığını kontrol eder (B0A1-C8FE aralığı).
        *   `check_han`: Bir string'in tamamen geçerli Hangul karakterlerinden, harflerden veya rakamlardan mı oluştuğunu kontrol eder.
        *   `first_han`: Verilen bir Hangul karakterinin ilk ünsüzünü (choseong) döndürmeye çalışır (KS X 1001 `KStbl` tablosunu kullanarak).
        *   `under_han`: Verilen bir string'in son Hangul karakterinin bir alt ünsüzü (jongseong) olup olmadığını kontrol eder (`KStbl` tablosunu kullanarak). Bu, Korece'de kelime sonuna eklenen "-i/-ga" gibi eklerin belirlenmesinde kullanılır.
    *   Sunucu tarafında Korece metinlerin (isimler, sohbet vb.) doğru işlenmesi için gereklidir.

### `heart.c`

*   **Implementasyon Detayları:**
    *   `heart.h`'de tanımlanan periyodik kalp atışı (heartbeat) zamanlayıcısını uygular.
    *   Belirlenen mikro saniye (`opt_usec`) aralığında periyodik olarak bir fonksiyon (`HEARTFUNC`) çağırmak için kullanılır.
    *   Ana fonksiyonlar:
        *   `heart_new`: Yeni bir kalp atışı nesnesi (`HEART`, `std::unique_ptr` ile yönetilir) oluşturur, hedef fonksiyonu ve periyodu ayarlar.
        *   `heart_delete`: Kalp atışı nesnesini siler ve kaynakları serbest bırakır.
        *   `heart_idle`: Bir sonraki kalp atışına kadar ne kadar süre bekleneceğini hesaplar ve sunucuyu o süre kadar uyutur (`thecore_sleep`). Aynı zamanda, olası gecikmeler nedeniyle kaçırılan kalp atışı sayısını (`missed_pulse`) hesaplar ve döndürür. Bu, ana döngünün zamanında çalışmayan görevleri telafi etmesine olanak tanır.
    *   `gettimeofday` ile hassas zaman ölçümü yapar ve `timediff`/`timeadd` (muhtemelen `utils.c`'de) gibi yardımcı fonksiyonları kullanır.
    *   Sunucunun ana olay döngüsünün (`thecore_idle` içinde kullanılır) ve periyodik görevlerin zamanlamasının temelini oluşturur.

### `kstbl.c`

*   **Implementasyon Detayları:**
    *   Bu dosya, `kstbl.h` içinde bildirilen `KStbl` dizisinin tanımını içerir.
    *   `KStbl`, KS X 1001 (EUC-KR) standardındaki 2350 adet tam Hangul hecesinin her biri için öncül (choseong), orta sesli (jungseong) ve soncul (jongseong) bileşen bilgilerini kodlayan büyük bir statik `unsigned int` dizisidir.
    *   `hangul.c` içindeki fonksiyonlar (`first_han`, `under_han`) tarafından Korece karakter analizleri için kullanılır.
    *   Doğrudan çağrılacak bir fonksiyon içermez, sadece veri sağlar.

### `log.c`

*   **Implementasyon Detayları:**
    *   `log.h` içinde bildirilen sunucu loglama sistemini uygular.
    *   Üç ana log dosyası yönetir: `syslog` (genel sistem logları), `syserr` (hatalar) ve `PTS` (özel amaçlı, potansiyel olarak oyuncu takip sistemi).
    *   Ana fonksiyonlar:
        *   `log_init`/`log_destroy`: Log sistemini başlatır/kapatır, log dosyalarını açar/kapatır.
        *   `sys_log`: Belirtilen seviyeye göre `syslog`'a ve isteğe bağlı olarak `stdout`'a log yazar.
        *   `_sys_err` (`sys_err` makrosu ile kullanılır): Hata mesajlarını hem `syserr` hem de `syslog` dosyalarına ve Windows'ta `stdout`'a yazar.
        *   `log_rotate`: Gün veya saat değişiminde eski log dosyalarını yeniden adlandırarak (örneğin, `syslog.20230101`) arşivler ve yenisini oluşturur.
        *   `log_file_delete_old`: Belirlenen gün sayısından (`log_keep_days`) daha eski log dosyalarını otomatik olarak siler.
    *   Log dizinini (`./log` varsayılan) ve saklama süresini ayarlama imkanı sunar.

### `main.c`

*   **Implementasyon Detayları:**
    *   `main.h` içinde bildirilen `libthecore` kütüphanesinin ana yaşam döngüsü fonksiyonlarını uygular.
    *   Ana fonksiyonlar:
        *   `thecore_init`: Kütüphaneyi başlatır. Rastgele sayı üretecini tohumlar, sinyal yöneticilerini (`signal_setup`) ayarlar, log sistemini (`log_init`) ve PID dosyasını (`pid_init`) başlatır, GOST şifreleme S-kutularını (`GOST_Init`) hazırlar ve kalp atışı mekanizmasını (`heart_new`) oluşturur.
        *   `thecore_shutdown`: Kütüphanenin ana döngüsünün durması için `shutdowned` bayrağını ayarlar.
        *   `thecore_idle`: Ana olay döngüsünün bir adımını temsil eder. Kalp atışı mekanizmasını (`heart_idle`) çağırarak uyur ve kaçırılan vuruş sayısını döndürür. Kapatma bayrağını kontrol eder.
        *   `thecore_destroy`: Kütüphane kapatılırken çağrılır. PID dosyasını siler (`pid_deinit`) ve log sistemini kapatır (`log_destroy`).
    *   Zamanlama (`thecore_time`, `thecore_pulse`), kapatma durumu kontrolü (`thecore_is_shutdowned`) ve basit bir dahili sayaç (`thecore_tick`) için yardımcı fonksiyonlar sağlar.
    *   `thecore_profiler` dizisi (kullanımı bu dosyada tam açık değil) ve `thecore_invalid` (geçersiz durum kontrolü) gibi ek değişkenler içerir.

### `memcpy.c`

*   **Implementasyon Detayları:**
    *   Bu dosya, `memcpy.h` içinde bildirilen `thecore_memcpy` fonksiyon işaretçisi ile ilgilidir.
    *   Dosyanın büyük bölümü, farklı işlemci mimarilerinde (MMX, SSE vb.) `memcpy` optimizasyonları üzerine detaylı yorumlar içerir (xine/mplayer kodlarından alınma).
    *   Ancak, mevcut kodda (en azından `#ifndef __WIN32__` kapsamında), `thecore_memcpy` işaretçisi doğrudan standart C kütüphanesinin `memcpy` fonksiyonuna atanmıştır. Bu, özel optimizasyonların şu anda aktif olarak kullanılmadığını veya platforma özgü olduğunu gösterebilir.
    *   Amacı potansiyel olarak platforma özel optimize edilmiş bellek kopyalama sağlamaktır, ancak mevcut durumda standart implementasyon kullanılmaktadır.

### `signal.c`

*   **Implementasyon Detayları:**
    *   `signal.h` içinde bildirilen işletim sistemi sinyal yönetimi fonksiyonlarını uygular.
    *   Implementasyon platforma özgüdür:
        *   **Windows:** Fonksiyonlar boş bırakılmıştır, sinyal yönetimi farklı şekilde ele alınır.
        *   **FreeBSD:** Standart Unix `signal` ve `setitimer` fonksiyonlarını kullanır.
    *   `signal_setup` (FreeBSD): Sunucunun düzgün kapatılması (`SIGHUP`, `SIGINT`, `SIGTERM` için `hupsig`), periyodik kontrol (`SIGVTALRM` için `checkpointing`), çocuk işlem yönetimi (`SIGCHLD` için `reap`) ve hata ayıklama (`SIGUSR1` için `usrsig`/`core_dump`) gibi yaygın sinyaller için işleyiciler ayarlar. `SIGPIPE` ve `SIGALRM` yok sayılır.
    *   `signal_timer_enable`/`disable` (FreeBSD): `SIGVTALRM` sinyalini tetikleyen sanal zamanlayıcıyı yönetir.

### `socket.c`

*   **Implementasyon Detayları:**
    *   `socket.h` içinde bildirilen temel TCP ve UDP soket fonksiyonlarını uygular.
    *   Platformdan bağımsız (Unix/Windows) soket işlemleri için sarmalayıcılar sunar.
    *   Ana fonksiyonlar:
        *   `socket_read`/`socket_udp_read`: TCP/UDP soketlerinden veri okur, bloklamayan G/Ç için hata kontrolü yapar (EAGAIN, EWOULDBLOCK).
        *   `socket_write`/`socket_write_tcp`: Veriyi TCP soketine yazar, tam gönderimi sağlamak için döngü kullanır.
        *   `socket_bind`/`socket_tcp_bind`/`socket_udp_bind`: Belirtilen IP ve porta TCP veya UDP soketi bağlar.
        *   `socket_accept`: Gelen TCP bağlantılarını kabul eder.
        *   `socket_connect`: Uzak bir sunucuya TCP bağlantısı kurar.
        *   `socket_close`: Soketi kapatır.
        *   Soket seçeneklerini ayarlamak için yardımcı fonksiyonlar: `socket_nonblock`, `socket_block`, `socket_lingeroff`, `socket_lingeron`, `socket_rcvbuf`, `socket_sndbuf`, `socket_timeout`, `socket_reuse`, `socket_keepalive`.
    *   Windows (`WSAEWOULDBLOCK`) ve Unix (`EAGAIN`, `EWOULDBLOCK`) sistemleri için bloklamayan G/Ç hata kodlarını ele alır.

### `tea.c`

*   **Implementasyon Detayları:**
    *   `crypt.h` içinde bildirilen TEA (Tiny Encryption Algorithm) blok şifreleme algoritmasını uygular.
    *   32 raundluk bir Feistel ağı kullanır.
    *   `DELTA` (0x9E3779B9) sabitini ve basit bit kaydırma/XOR operasyonlarını içerir.
    *   Ana fonksiyonlar:
        *   `tea_code`/`tea_decode`: 8 byte'lık tek bir bloğu şifreler/çözer (iç kullanım için inline).
        *   `TEA_Encrypt`/`TEA_Decrypt`: Verilen veriyi 8 byte'lık bloklar halinde şifreler/çözer. Veri boyutu 8'in katı değilse sıfırlarla doldurma (padding) yapar.
    *   Genellikle istemci-sunucu arasındaki temel paket şifrelemesi için kullanılır.

### `tea.s`

*   **Not:** Bu dosya, `tea.c` dosyasının derlenmiş assembly (GAS - GNU Assembler formatı) kodunu içerir. 
*   Muhtemelen derleyici tarafından otomatik olarak oluşturulmuştur veya belirli bir mimari için elle optimize edilmiş olabilir.
*   Doğrudan okunması veya düzenlenmesi genellikle gerekli değildir, `tea.c` kaynak kodunun derlenmiş halidir.

### `utils.c`

*   **Implementasyon Detayları:**
    *   `utils.h` içinde bildirilen genel amaçlı yardımcı fonksiyonları ve makroları uygular.
    *   **Zaman Fonksiyonları:** `gettimeofday` (Windows için polyfill), `timediff`, `timeadd`, `thecore_sleep` (platforma göre `select` veya `Sleep`), `thecore_msleep`, `get_float_time`, `get_dword_time`, `time_str`.
    *   **String Fonksiyonları:** `trim_and_lower` (boşlukları temizle ve küçük harfe çevir), `lower_string` (küçük harfe çevir), `str_dup` (string kopyası oluştur), `is_abbrev` (kısaltma kontrolü), `parse_token` (':' ile ayrılmış token/değer çiftini ayır).
    *   **Dosya Fonksiyonları:** `filesize` (dosya boyutunu alır).
    *   **Matematik Fonksiyonları:** `MIN`, `MAX`, `MINMAX`, `MINLL`, `MAXLL`, `MINMAXLL`, `thecore_random` (platforma göre `rand` veya `random`), `number_ex` (belirtilen aralıkta rastgele sayı, hata kontrolü ile), `fnumber` (belirtilen aralıkta rastgele float sayı).
    *   **Hata Ayıklama:** `printdata` (veriyi hex ve ASCII formatında yazdırır), `core_dump_unix` (Unix'te çökme dökümü oluşturur, Windows'ta boş).
    *   **Diğer:** `tm_calc` (belirli bir tarihe gün ekler/çıkarır).

### `xdirent.c`

*   **Implementasyon Detayları:**
    *   Bu dosya, **sadece Windows** platformunda (`#ifdef __WIN32__`) derlenir.
    *   `xdirent.h` içinde bildirilen POSIX uyumlu dizin gezinme fonksiyonlarını (`opendir`, `closedir`, `readdir`, `rewinddir`) Windows API'larını (`_findfirst`, `_findnext`, `_findclose`) kullanarak taklit eder.
    *   Amacı, dizin listeleme işlemlerini platformdan bağımsız hale getirmektir.
    *   `DIR` yapısını Windows'un `_finddata_t` yapısı ve dosya arama `handle`'ı etrafında tanımlar.

### `xgetopt.c`

*   **Implementasyon Detayları:**
    *   Bu dosya, **sadece Windows** platformunda (`#ifdef __WIN32__`) derlenir.
    *   `xgetopt.h` içinde bildirilen standart POSIX `getopt` fonksiyonunu (komut satırı argümanlarını ayrıştırmak için) Windows ortamı için uygular.
    *   Kısa seçenekleri (`-a`, `-b`, `-n <değer>`) destekler, ancak uzun seçenekleri (`--option`) veya GNU uzantılarını desteklemez.
    *   `optarg` (seçenek argümanı) ve `optind` (işlenen sıradaki argüman indeksi) global değişkenlerini yönetir.
    *   `--` ile seçeneklerin sonlandırılmasını destekler.

### `xmd5.c`

*   **Implementasyon Detayları:**
    *   Bu dosya, FreeBSD `libmd` kütüphanesinden türetilmiştir ve **FreeBSD dışındaki** platformlarda (`#ifndef __FreeBSD__`) derlenir.
    *   `xmd5.h` içinde bildirilen MD5 (Message Digest Algorithm 5) özetleme fonksiyonlarını uygular.
    *   Standart MD5 adımlarını içerir: `MD5Init` (bağlamı başlatır), `MD5Update` (veriyi işler), `MD5Final` (sonucu hesaplar ve bağlamı temizler).
    *   `MD5Transform`: MD5 algoritmasının çekirdek dönüşüm fonksiyonu.
    *   `byteReverse`: Büyük-endian/küçük-endian sistemler arası uyumluluk için byte sırasını tersine çevirir.
    *   Kolay kullanım için yardımcı fonksiyonlar sunar:
        *   `MD5End`: `MD5Final` sonucunu okunabilir hex string formatına çevirir.
        *   `lutil_md5_file`: Bir dosyanın MD5 özetini hesaplar.
        *   `lutil_md5_data`: Bir bellek bloğunun MD5 özetini hesaplar.

---

## `win32/` Klasörü

*   **Amaç:** Bu klasör, `libthecore` kütüphanesinin Windows platformuna özgü yapılandırma ve derleme dosyalarını içerir.
*   **İçerik:** Genellikle Visual Studio proje dosyaları (`.vcxproj`, `.sln` - bu projede yok gibi görünüyor), derleme çıktıları (`Debug/`, `Release/` alt klasörleri) ve platforma özel diğer dosyaları barındırır.

### `win32/Debug/` Klasörü ve İçeriği

Bu alt klasör, kütüphanenin **Debug** konfigürasyonunda Windows üzerinde derlenmesi sırasında Visual Studio (MSVC) tarafından oluşturulan ara dosyaları ve çıktıları içerir. Gördüğümüz bazı örnekler:

*   **`microsoft/STL/std.compat.ixx.ifc.dt.*` Dosyaları:**
    *   Bunlar MSVC derleyicisinin C++20 Modülleri özelliği ile ilgili oluşturduğu ara dosyalardır. Özellikle C++ Standart Kütüphanesi'nin uyumluluk modülü (`std.compat`) ile ilgilidirler.
    *   `.json` dosyaları modül bağımlılıkları ve bilgileri gibi meta verileri içerir.
    *   `.command` dosyası, ilgili modül dosyasını oluşturmak için kullanılan derleyici komutunu saklar.
    *   Bu dosyalar doğrudan kütüphane kaynak kodunun bir parçası değildir ve genellikle geliştiricilerin doğrudan etkileşimde bulunması veya değiştirmesi gerekmez. Derleme sürecini desteklemek için kullanılırlar.

*   **`microsoft/STL/std.ixx.ifc.dt.*` Dosyaları:**
    *   `std.compat.*` dosyalarına benzer şekilde, bunlar da MSVC derleyicisinin ana C++20 Standart Kütüphane modülü (`std`) için oluşturduğu ara derleme dosyalarıdır.
    *   Aynı şekilde `.json` meta verilerini ve `.command` derleyici komutunu içerirler.
    *   Bunlar da kütüphanenin asıl kaynak kodları değil, derleme sürecinin otomatik ürettiği dosyalardır.

---

## Proje Dosyaları

### `libthecore.vcxproj`

*   **Dosya Türü:** Visual Studio C++ Proje Dosyası (XML).
*   **Amaç:** `libthecore` kütüphanesinin Windows (Win32) platformunda Visual Studio (v143 araç takımı - VS 2022) ile nasıl derleneceğini tanımlar.
*   **Proje Tipi:** Statik Kütüphane (`.lib`).
*   **Temel Ayarlar:**
    *   **Platform:** Win32.
    *   **Konfigürasyonlar:** `Debug` ve `Release`.
    *   **Çıktı Dosyaları:** `lib/libthecore_d.lib` (Debug), `lib/libthecore.lib` (Release).
    *   **Ara Dizinler:** `win32/Debug/` ve `win32/Release/`.
    *   **Include Yolları:** `include/` ve `../../External/include`.
    *   **Önişlemci Tanımları:** `WIN32`, `__WIN32__`, `_CRT_SECURE_NO_DEPRECATE`, `_USE_32BIT_TIME_T`, `_WINSOCK_DEPRECATED_NO_WARNINGS` ve konfigürasyona özel (`_DEBUG`/`NDEBUG`).
    *   **Çalışma Zamanı Kütüphanesi:** `/MTd` (Debug), `/MT` (Release).
    *   **Derlenen Kaynaklar:** `src/` klasöründeki tüm `.c` dosyaları (C++ olarak derlenir).
    *   **Dil Standardı:** En son C++ standardı (`stdcpplatest`).
*   **Not:** Bu dosya, kütüphanenin Windows ortamında derlenmesi için kritik yapılandırma bilgilerini içerir.