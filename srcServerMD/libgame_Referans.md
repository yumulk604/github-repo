# libgame Referans Kılavuzu

Bu dosya, `srcServer/Source/libgame` kütüphanesinin amacını, içerdiği temel bileşenleri ve fonksiyonları açıklamaktadır.

`libgame`, Metin2 sunucusunun temel oyun mekaniklerini, karakterler, eşyalar, yetenekler gibi oyun öğeleriyle ilgili paylaşılan veri yapılarını ve yardımcı fonksiyonları, ayrıca Lua script motoru için C++ binding'lerini içeren merkezi bir kütüphanedir.

## Header Dosyaları (`include/`)

Bu klasör, kütüphanenin dışarıya sunduğu temel arayüzleri içerir.

### `grid.h`

*   **Amaç:** İki boyutlu bir ızgarayı (grid) temsil eden ve üzerinde yer bulma/yerleştirme işlemleri yapan `CGrid` sınıfını tanımlar. Özellikle envanter, depo gibi sistemler için kullanılır.
*   **`CGrid` Sınıfı:**
    *   Belirtilen boyutlarda bir ızgara oluşturur (`m_pGrid` char dizisi ile).
    *   `FindBlank(w, h)`: Verilen boyuttaki ilk boş alanı bulur.
    *   `IsEmpty(pos, w, h)`: Belirtilen alanın boş olup olmadığını kontrol eder.
    *   `Put(pos, w, h)`: Belirtilen alana yerleştirme yapar (dolu işaretler).
    *   `Get(pos, w, h)`: Belirtilen alandan kaldırma yapar (boş işaretler).
    *   `Clear()`: Izgarayı temizler.
    *   `GetSize()`: Toplam hücre sayısını döndürür.
    *   Özel sistemlere (`__PREMIUM_PRIVATE_SHOP__`) ait olabilecek ek pozisyon metodları içerebilir. 

### `attribute.h`

*   **Amaç:** İki boyutlu bir alandaki (genellikle harita) her hücreye bir veya daha fazla nitelik (attribute) atamak ve yönetmek için `CAttribute` sınıfını tanımlar.
*   **`EDataType` Enum:** Hücre başına nitelik verisinin saklama türünü belirtir (`D_DWORD`, `D_WORD`, `D_BYTE`).
*   **`CAttribute` Sınıfı:**
    *   Belirtilen boyutlarda bir nitelik haritası oluşturur.
    *   Mevcut bir `DWORD` dizisinden veri okuyarak ve en uygun veri türünü seçerek (`BYTE`, `WORD`, `DWORD`) harita oluşturabilir.
    *   `Set(x, y, attr)`: Belirtilen hücreye nitelik atar/ekler.
    *   `Remove(x, y, attr)`: Belirtilen hücreden niteliği kaldırır.
    *   `Get(x, y)`: Hücrenin nitelik değerini alır.
    *   `GetDataType()`, `GetDataPtr()`: Veri türünü ve ham veri işaretçisini döndürür.
    *   `CopyRow(y, row)`: Belirtilen satırın verisini kopyalar.
    *   Veriyi `void* data` içinde tutar ve `dataType`'a göre `BYTE**`, `WORD**`, `DWORD**` işaretçileriyle erişimi kolaylaştırır.
*   **Kullanım Alanı:** Harita bölgelerine özellikler atamak (güvenli bölge, geçilemez alan, zemin türü vb.). 

### `targa.h`

*   **Amaç:** Targa (.tga) resim dosyası formatını temsil etmek (`TGA_HEADER`) ve basit TGA dosyaları oluşturup kaydetmek (`CTargaImage`) için yapılar ve bir sınıf tanımlar.
*   **`TGA_HEADER` Yapısı:** TGA dosya başlığının alanlarını tanımlar (`pragma pack(1)` ile paketlenmiş).
*   **`CTargaImage` Sınıfı:**
    *   `Create(x, y)`: Belirtilen boyutlarda yeni bir TGA resmi için bellek ayırır ve başlığı başlatır.
    *   `GetBasePointer(line)`: Belirtilen satırın piksel verisine işaretçi döndürür (doğrudan piksel manipülasyonu için).
    *   `Save(filename)`: Mevcut resim verisini .tga dosyası olarak kaydeder.
*   **Olası Kullanım Alanları:** Dinamik mini harita üretimi, debug amaçlı veri görselleştirme veya eski/kullanılmayan bir özellik. 

## Kaynak Kod Dosyaları (`src/`)

Bu klasör, `include/` altındaki header dosyalarında bildirilen sınıfların implementasyonlarını içerir.

### `grid.cc`

*   **Amaç:** `CGrid` sınıfının metodlarını implemente eder.
*   **Detaylar:**
    *   Izgara verisini (`m_pGrid`) dinamik olarak ayrılmış bir `char` dizisinde tutar (0: boş, 1: dolu).
    *   `FindBlank`: Izgarayı tarayarak ve her pozisyon için `IsEmpty`'i çağırarak boş yer arar.
    *   `Put`/`Get`: İlgili hücreleri 1 veya 0 olarak ayarlar.
    *   `IsEmpty`: Sınır kontrolleri yapar ve ilgili alandaki tüm hücrelerin 0 olup olmadığını kontrol eder.
    *   Bellek yönetimi için `new`/`delete []` ve veri kopyalama/sıfırlama için `memset`/`thecore_memcpy` kullanır.
*   **Bağımlılıklar:** `grid.h`, `../../libthecore/include/memcpy.h`, `../../common/stl.h` (muhtemelen `std::min` için). 

### `attribute.cc`

*   **Amaç:** `CAttribute` sınıfının metodlarını implemente eder.
*   **Detaylar:**
    *   **Bellek Yönetimi ve Optimizasyon:**
        *   `Alloc`: `dataType`'a (`D_BYTE`, `D_WORD`, `D_DWORD`) göre `malloc` ile ham bellek (`data`) ayırır ve 2D erişim için satır işaretçileri (`bytePtr` vb.) oluşturur.
        *   "Akıllı" Kurucu (`CAttribute(attr, ...)`): Başlangıç verisinin tekdüze olup olmadığını kontrol eder. Tekdüze ise bellek ayırmaz, sadece `defaultAttr` kullanır. Farklı değerler varsa, minimum gerekli veri türünü (`BYTE`, `WORD`, `DWORD`) belirleyerek bellek ayırır ve veriyi kopyalar/dönüştürür.
    *   **Nitelik İşlemleri:**
        *   `Set`/`Remove`: Bitwise OR (`SET_BIT`) ve AND NOT (`REMOVE_BIT`) makrolarını kullanarak belirli nitelik bitlerini ekler veya kaldırır. Bu, bir hücrenin birden fazla bayrak tutabilmesini sağlar.
        *   `Get`: Hücrenin nitelik değerini (veya `defaultAttr`) döndürür.
    *   `CopyRow`: Belirtilen satırı kopyalar (BYTE/WORD için implementasyon eksik görünüyor).
*   **Bağımlılıklar:** `attribute.h`, `../../libthecore/include/stdafx.h` (muhtemelen temel türler ve `assert` için), `../../libthecore/include/memcpy.h`. 

### `targa.cc`

*   **Amaç:** `CTargaImage` sınıfının metodlarını implemente eder.
*   **Detaylar:**
    *   `Create`: 32-bit renk derinliğine (`colorBits = 32`), sıkıştırılmamış formata (`imgType = 2`) ve sol-üst orijine (`desc = 0x20`) sahip bir TGA başlığı oluşturur. Piksel verisi için `new char[]` ile bellek ayırır (piksel başına `sizeof(DWORD)` varsayılır).
    *   `GetBasePointer`: 32-bit piksel formatına göre satır başlangıç adresini hesaplar.
    *   `Save`: `fopen` ve `fwrite` kullanarak başlığı ve piksel verisini dosyaya yazar.
    *   Bellek yönetimi için `new`/`delete[]` ve `memset` kullanır.
*   **Bağımlılıklar:** `targa.h`, `../../libthecore/include/stdafx.h`. 

## Diğer Dosyalar ve Klasörler

*   **`Makefile`:** Linux/FreeBSD ortamında `make` komutu ile `libgame.a` statik kütüphanesini derlemek için kullanılan betik (muhtemelen sadece `attribute.o`, `grid.o`, `targa.o` nesnelerini içerir).
*   **`libgame.vcxproj`:** Windows ortamında Visual Studio ile `libgame.lib` statik kütüphanesini derlemek için kullanılan proje dosyası.
*   **`Depend`:** `Makefile` tarafından kullanılan, `src/` içindeki kaynak dosyaların bağımlılıklarını içeren dosya.
*   **`attribute.o`, `grid.o`, `targa.o`:** `src/` klasöründeki `.cc` dosyalarının derlenmiş nesne kodları.
*   **`lib/` Klasörü:** Muhtemelen derlenmiş statik kütüphane (`libgame.a` veya `libgame.lib`) arşivini içerir.
*   **`win32/` Klasörü:** Windows platformuna özgü derleme çıktılarını (`.obj`, `.lib`) veya ayarları içerebilir. 