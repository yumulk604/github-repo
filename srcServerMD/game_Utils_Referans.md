# Metin2 Oyun Sunucusu - Yardımcı Bileşenler Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) yardımcı sınıflarını, veri yapılarını, ağ iletişimini, konfigürasyonunu, veritabanı etkileşimini, loglamayı ve diğer destekleyici bileşenlerini belgeler.

## İçindekiler

*   [Klasörler](#klasörler)
*   [Header Dosyaları (.h)](#header-dosyaları-h)
*   [Kaynak Kod Dosyaları (.cpp)](#kaynak-kod-dosyaları-cpp)

## Klasörler

### `lzo/`

*   **Amaç:** LZO (Lempel–Ziv–Oberhumer) veri sıkıştırma algoritmasının kütüphane dosyalarını içerir.
*   **Temel İşlevler/İçerik:**
    *   Bu klasör, LZO sıkıştırma ve açma işlemleri için gerekli olan header dosyalarını (`.h`) barındırır.
    *   Metin2 sunucusu, genellikle istemci ile sunucu arasındaki ağ trafiğini azaltmak için ağ paketlerini sıkıştırmak/açmak amacıyla bu kütüphaneyi kullanır. Ayrıca bazı oyun dosyalarının veya verilerinin sıkıştırılmasında da kullanılabilir.
    *   İçerdiği `.h` dosyaları (`lzo1.h`, `lzo1a.h`, `lzo1b.h`, `lzo1c.h`, `lzo1f.h`, `lzo1x.h`, `lzo1y.h`, `lzo1z.h`, `lzo2a.h`, `lzo16bit.h`, `lzoconf.h`, `lzoutil.h`) LZO kütüphanesinin farklı sıkıştırma seviyeleri ve fonksiyonları için arayüz tanımlamalarını sağlar.
    *   `Makefile` dosyaları, kütüphanenin derlenmesi için kullanılan yapılandırma dosyalarıdır. Kaynak kod dosyaları (`.c`/`.cpp`) genellikle bu klasörde bulunmaz, kütüphane genellikle önceden derlenmiş olarak sisteme eklenir veya projenin başka bir bölümünde yer alır.
*   **Önemli Notlar:** Bu, harici bir üçüncü parti kütüphanedir.

### `perftest/`

*   **Amaç:** Performans testiyle ilgili yardımcı fonksiyonları ve test kodlarını içerir. Bu klasördeki kodlar genellikle doğrudan oyunun çalışma zamanı mantığında kullanılmaz, daha çok geliştirme ve optimizasyon aşamalarında performans analizi yapmak için kullanılır.
*   **Dosyalar:**
    *   **`timeval_subtract.c`**
        *   **İşlev:** POSIX `struct timeval` tipindeki iki zaman damgası arasındaki farkı hesaplayan `timeval_subtract` fonksiyonunu içerir.
        *   **Kullanım:** Kod bloklarının veya fonksiyonların çalışma süresini mikrosaniye hassasiyetinde ölçmek için kullanılır.
        *   **İmplementasyon:** Standart C ile yazılmıştır. Negatif fark durumunda 1 döndürür.
    *   **`alloc_perf_test.cpp`**
        *   **İşlev:** Farklı bellek ayırma (allocation/deallocation) stratejilerinin performansını karşılaştırmak için bir test programı içerir.
        *   **Kullanım:** Geliştirme sırasında standart (`new`/`delete`, `malloc`/`free`) ve Metin2'nin özel (`M2_NEW`/`M2_DELETE`, `M2_MALLOC`/`M2_FREE`) bellek yöneticilerinin (normal ve debug modları dahil) hızını ölçmek için kullanılır.
        *   **İmplementasyon:** Önişlemci direktifleri (`#define CASE_...`) ile farklı test senaryoları (raw, allocator, debug allocator; new/delete veya malloc/free) seçilir. `kCount` (test edilecek örnek sayısı) kadar `Foo` nesnesi yaratılıp silinir ve geçen süre `gettimeofday` ve `timeval_subtract` kullanılarak ölçülür. Metin2'nin `../allocator.h` dosyasındaki `Allocator` sınıfını kullanır.
*   **Önemli Notlar:** Bu klasördeki kodlar genellikle doğrudan oyunun çalışma zamanı mantığında kullanılmaz, daha çok geliştirme ve optimizasyon aşamalarında performans analizi yapmak için kullanılır.

## Header Dosyaları (.h)

*Buraya `game/src` altındaki yardımcı header dosyalarının belgeleri eklenecektir.*

## Kaynak Kod Dosyaları (.cpp)

*Buraya `game/src` altındaki yardımcı kaynak kod dosyalarının belgeleri eklenecektir.* 