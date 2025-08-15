# libserverkey Referans Kılavuzu

Bu dosya, `srcServer/Source/libserverkey` kütüphanesinin amacını, içerdiği temel bileşenleri ve fonksiyonları açıklamaktadır.

`libserverkey`, sunucu anahtarının doğrulanması ve ilgili kriptografik işlemler için kullanılan bir kütüphanedir.

## Header Dosyaları

### `SIM.h`

*   **Amaç:** IP adresi ve MAC adresi tabanlı erişim kontrol listelerini yönetmek için `SIM` sınıfını tanımlar. Dosya adının aksine mobil SIM kartlarla bir ilgisi yoktur, muhtemelen "Server Identification Module" gibi bir anlama gelir.
*   **Temel İşlevler:**
    *   Metin tabanlı kuralları (`IP\tMAC`, `IP1~IP2`, `IP` formatlarında) ayrıştırma (`ParseLine`).
    *   Ayrıştırılmış kuralları kompakt bir binary formata dönüştürme (`MakeBinary`).
    *   Binary formatı tekrar okunabilir kurallara çevirme (`ParseBinary`).
    *   Verilen bir IP ve MAC adresinin tanımlı kurallara uyup uymadığını kontrol etme (`CheckIpAndMac`).
    *   MAC adresi format kontrolü (`checkmac`) ve MAC-binary dönüşümleri (`mac2bin`, `bin2mac`) için yardımcı fonksiyonlar içerir.
*   **Not:** Platforma özel tanımlamalar ve Korece yorumlar içerir.

### `CheckServerKey.h`

*   **Amaç:** Tek bir `inline` fonksiyon olan `CheckServerKey`'i tanımlar. Bu fonksiyon, verilen bir sunucu anahtarını (`serverKey`), IP ve MAC adresini kullanarak doğrular.
*   **İşleyiş Adımları:**
    1.  Sunucu anahtarındaki `*` karakterlerini `=` ile değiştirir (Muhtemelen Base64 uyumluluğu için).
    2.  Anahtarı Base64 formatından çözer (Platforma göre `atlenc.h` veya `base64_ssl.h` kullanılır).
    3.  `PublicKey.Gen.h`'den genel anahtarı alır.
    4.  Genel anahtarı `RSACrypto` kullanarak geri yükler.
    5.  Base64 çözülmüş veriyi RSA genel anahtarı ile deşifre eder.
    6.  Deşifre edilmiş binary veriyi `SIM::ParseBinary` ile ayrıştırarak IP/MAC kurallarını yükler.
    7.  Yüklenen kurallarla verilen IP/MAC adresinin eşleşip eşleşmediğini `SIM::CheckIpAndMac` ile kontrol eder.
*   **Bağımlılıklar:** `SIM.h`, `base64_ssl.h` (veya `atlenc.h`), `PublicKey.Gen.h`, `RSACrypto.h`.

### `base64_ssl.h`

*   **Amaç:** Base64 kodlama ve çözme işlemleri için fonksiyon bildirimleri içerir.
*   **Fonksiyonlar:**
    *   `base64_ssl`: Verilen girdiyi Base64 formatına kodlar.
    *   `unbase64_ssl`: Base64 formatındaki girdiyi çözer.
*   **Not:** Bu fonksiyonlar muhtemelen OpenSSL kullanılarak implemente edilmiştir ve `CheckServerKey.h` içinde Windows dışı platformlar için kullanılır.

### `PublicKey.Gen.h`

*   **Amaç:** Sabit olarak tanımlanmış RSA genel anahtar verisini içeren `CreatePublicKey` inline fonksiyonunu barındırır.
*   **İçerik:** Fonksiyon, `std::vector<unsigned char>` içine önceden belirlenmiş byte dizisini ekleyerek genel anahtarı oluşturur. Bu byte dizisi, muhtemelen DER formatında kodlanmış anahtar verisidir.
*   **Not:** Dosya başındaki yoruma göre anahtar 5 Şubat 2013'te oluşturulmuştur.

### `ServerKey.Gen.h`

*   **Amaç:** Sabit olarak tanımlanmış RSA özel anahtar verisini içeren `CreatePrivateKey` inline fonksiyonunu ve `PublicKey.Gen.h`'deki `CreatePublicKey` fonksiyonunun bir kopyasını barındırır.
*   **İçerik:**
    *   `CreatePrivateKey`: `std::vector<unsigned char>` içine önceden belirlenmiş byte dizisini ekleyerek özel anahtarı oluşturur. Bu, `PublicKey.Gen.h`'deki genel anahtarın eşidir.
    *   `CreatePublicKey`: `PublicKey.Gen.h`'deki fonksiyonun aynısıdır.
*   **Not:** Dosya başındaki yoruma göre anahtar 5 Şubat 2013'te oluşturulmuştur.

### `RSACrypto.h`

*   **Amaç:** RSA şifreleme/çözme işlemleri, anahtar yönetimi ve SHA1 hash hesaplaması için sınıflar ve fonksiyonlar tanımlar. OpenSSL kütüphanesini kullanır.
*   **Namespace:** `Security`
*   **Yapılar ve Sınıflar:**
    *   `Buffer`: Dinamik boyutlu bellek tamponlarını yönetir (`Alloc`, `Free`).
    *   `RSACrypto`:
        *   `PublicKey`: RSA genel anahtarını temsil eder (OpenSSL `rsa_st*` içerir).
        *   `PrivateKey`: RSA özel anahtarını temsil eder (OpenSSL `rsa_st*` içerir).
        *   **Statik Metodlar:**
            *   Şifreleme/Çözme: `EncryptPublic`, `DecryptPrivate`, `EncryptPrivate`, `DecryptPublic`.
            *   Anahtar Yönetimi: `GenerateKey`, `StorePrivateKey`, `RestorePrivateKey`, `StorePublicKey`, `RestorePublicKey` (DER formatı kullanır).
            *   Yardımcı: `PrintKey` (debug amaçlı).
    *   `SHA1`:
        *   `Digest`: Verilen `Buffer` için SHA1 hash hesaplar.
*   **Fonksiyonlar:**
    *   `InitRandomSeed`: OpenSSL rastgele sayı üretecini başlatır.
*   **Bağımlılıklar:** OpenSSL (`libeay32.lib` vs.)
*   **Not:** Dosya başındaki yoruma göre anahtar 5 Şubat 2013'te oluşturulmuştur.

## Kaynak Kod Dosyaları

### `base64_ssl.cpp`

*   **Amaç:** `base64_ssl.h`'de bildirilen `base64_ssl` (kodlama) ve `unbase64_ssl` (çözme) fonksiyonlarını OpenSSL'in BIO kütüphanesini kullanarak implemente eder.
*   **İşleyiş:**
    *   Fonksiyonlar, Base64 filtresi (`BIO_f_base64`) ile bellek tamponu (`BIO_s_mem` veya `BIO_new_mem_buf`) arasında veri akışı sağlamak için OpenSSL BIO zincirlerini kullanır.
    *   `BIO_write` (kodlama için) ve `BIO_read` (çözme için) fonksiyonları, BIO zinciri üzerinden geçerken veriyi otomatik olarak dönüştürür.
*   **Bağımlılıklar:** OpenSSL (BIO kütüphanesi).
*   **Not:** Dosya başındaki yoruma göre anahtar 5 Şubat 2013'te oluşturulmuştur. 

### `RSACrypto.cpp`

*   **Amaç:** `RSACrypto.h`'de bildirilen `RSACrypto` ve `SHA1` sınıflarının metodlarını OpenSSL fonksiyonlarını kullanarak implemente eder.
*   **Önemli Detaylar:**
    *   **OpenSSL Versiyon Uyumluluğu:** OpenSSL 1.1.0 öncesi ve sonrası API değişikliklerini yönetmek için `#if OPENSSL_VERSION_NUMBER` kontrolü yapar.
    *   **Rastgele Sayı Başlatma:** `InitRandomSeed` ve `StaticInitializer` ile OpenSSL'in rastgele sayı üretecini program başlangıcında güvenli bir şekilde başlatır.
    *   **Padding Modları:**
        *   `EncryptPublic` / `DecryptPrivate`: `RSA_PKCS1_OAEP_PADDING` kullanır.
        *   `EncryptPrivate` / `DecryptPublic`: `RSA_PKCS1_PADDING` kullanır.
    *   **Anahtar Üretimi (`GenerateKey`):** 1024-bit RSA anahtarı üretir (Public exponent: 7).
    *   **Anahtar Saklama/Yükleme:** `Store*Key` ve `Restore*Key` fonksiyonları, anahtarları standart DER formatında işlemek için OpenSSL'in `i2d_*` ve `d2i_*` fonksiyonlarını kullanır.
    *   **SHA1 Hash:** `SHA1::Digest`, OpenSSL'in `::SHA1` fonksiyonunu çağırır.
*   **Bağımlılıklar:** OpenSSL (RSA, RAND, SHA, PEM, BIO, BIGNUM vb. fonksiyonları). 

## Diğer Dosyalar ve Klasörler

*   **`Makefile`:** Linux/FreeBSD ortamında `make` komutu ile kütüphaneyi derlemek için kullanılan derleme betiği.
*   **`libserverkey.vcxproj`:** Windows ortamında Visual Studio ile kütüphaneyi derlemek için kullanılan proje dosyası.
*   **`Depend`:** Muhtemelen `Makefile` tarafından kullanılan, kaynak dosyaların bağımlılıklarını içeren dosya.
*   **`base64_ssl.o`, `RSACrypto.o`:** Derlenmiş nesne kod dosyaları (object files).
*   **`libserverkey.a`:** Derlenmiş statik kütüphane arşivi (archive file).
*   **`win32/` Klasörü:** Windows platformuna özgü dosyaları veya ayarları içerebilir (İçeriği şu anki listede görünmüyor). 