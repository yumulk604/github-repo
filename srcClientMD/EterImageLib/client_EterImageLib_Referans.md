# EterImageLib Referans Kılavuzu

Bu kılavuz, `@srcClient/Source/EterImageLib/` klasöründe bulunan `EterImageLib` modülünün amacını, içerdiği temel bileşenleri ve bu bileşenlerin işlevlerini açıklamaktadır.

`EterImageLib`, Metin2 istemcisinin çeşitli resim (imaj) formatlarını yüklemek, işlemek ve yönetmek için kullandığı temel kütüphanedir. Oyun içindeki dokular (textures), arayüz elemanları ve diğer grafiksel varlıklar için kullanılan resim dosyalarının (örneğin, TGA, JPG, DDS) okunması ve oyun motorunun kullanabileceği bir formata dönüştürülmesi bu kütüphane aracılığıyla gerçekleştirilir.

## Genel Amaç ve İçerik

`EterImageLib` modülü, aşağıdaki gibi çeşitli temel görevler için araçlar sunar:

*   **Resim Dosyası Yükleme:** Farklı formatlardaki (TGA, DDS vb.) resim dosyalarını bellekten veya diskten okuma.
*   **Resim Verisi İşleme:** Okunan resim verilerini (piksel bilgileri, boyutlar, format vb.) işleme ve saklama.
*   **Format Dönüşümü:** Gerekirse resim formatları arasında temel dönüşümler yapma (örneğin, sıkıştırılmış formatları açma).
*   **Arayüz Sağlama:** Yüklenen resim verilerini grafik motorunun (örneğin, `EterGrnLib` veya doğrudan Direct3D) kullanabileceği şekilde sunma.

Bu modül, istemcinin görsel dünyasının oluşturulmasında kritik bir rol oynar, çünkü tüm 2D ve 3D grafiklerin temelini oluşturan doku ve resimlerin yüklenmesinden sorumludur.

---

## Dosya Bazlı Detaylandırma 

### `StdAfx.h` ve `StdAfx.cpp`

*   **Amaç:** Bu dosyalar, `EterImageLib` projesi için ön derlenmiş başlık (PCH - Precompiled Header) mekanizmasını sağlar. Sık kullanılan ve nadiren değiştirilen standart sistem başlık dosyalarını (Windows API) ve proje için temel olan bazı genel başlıkları içerir. Amaç, derleme sürelerini kısaltmaktır.

*   **Temel Özellikler/İçerik (`StdAfx.h`):**
    *   `#pragma once`: Başlık dosyasının tek bir derleme biriminde yalnızca bir kez dahil edilmesini sağlar.
    *   `#pragma warning(disable:4786)`: Uzun sembol adlarıyla ilgili bir derleyici uyarısını (genellikle STL kullanımlarından kaynaklanır) devre dışı bırakır.
    *   `WIN32_LEAN_AND_MEAN`: Windows başlık dosyalarından daha az kullanılan API'lerin çıkarılmasını sağlayarak derleme süresini azaltır.
    *   **Ana Bağımlılıklar:**
        *   `"../UserInterface/Locale_inc.h"`: Yerelleştirme ile ilgili tanımlamalar için `UserInterface` modülüne bir bağımlılık.
        *   `<windows.h>`: Temel Windows API fonksiyonları.
        *   `<assert.h>`: Hata ayıklama için `assert` makrosu.
        *   `<string>`, `<vector>`: Standart C++ kütüphanesinden string ve vektör konteynerları.
    *   **`_TraceForImage` Fonksiyonu:**
        *   Değişken argüman listesi (`va_list`) alarak formatlı bir string oluşturur.
        *   Oluşturulan string'i `_DEBUG` modunda `OutputDebugString` ile hata ayıklama çıktısına, her durumda ise `printf` ile konsola yazar. Bu, resim yükleme ve işleme sırasında loglama veya hata ayıklama mesajları basmak için kullanılabilir.
    *   `#pragma warning(default:4018)`: Muhtemelen daha önce devre dışı bırakılmış olan 4018 numaralı uyarıyı (işaretli/işaretsiz uyuşmazlığı) varsayılan durumuna getirir.

*   **İçerik (`StdAfx.cpp`):**
    *   `#include "stdafx.h"`: Ön derlenmiş başlık dosyasını dahil eder. Bu dosyanın kendisi `.pch` dosyasının oluşturulmasını tetikler.
    *   `namespace { char dummy; };`: Bazı durumlarda bağlayıcının (linker) LNK4221 uyarısı vermesini (obje dosyasının genel sembol tanımlamaması) engellemek için kullanılan boş bir namespace içinde tanımlanmış bir karakter.

*   **Kullanım Alanı:**
    *   Bu dosyalar doğrudan resim işleme mantığı içermez. `EterImageLib` kütüphanesinin diğer `.cpp` dosyalarının en başına `#include "stdafx.h"` eklenerek, burada tanımlanan başlıkların ve ayarların tüm kütüphane genelinde tutarlı bir şekilde kullanılmasını ve derleme optimizasyonunu sağlar.
    *   `_TraceForImage` fonksiyonu, kütüphane içinde özel loglama ihtiyaçları için bir yardımcı araç sunar.

    ### `Image.h` ve `Image.cpp` (`CImage` Sınıfı)

*   **Amaç:** `CImage` sınıfı, bellekte tutulan 2D resim verileri için temel bir konteyner ve manipülasyon arayüzü sağlar. Bu sınıf, genellikle diğer resim formatı yükleyicileri (örneğin, `CTGAImage`, `CDXTCImage`) için bir hedef veya kaynak olarak kullanılır. Resim verilerini 32-bit renk (DWORD) dizisi olarak saklar.

*   **`TGA_HEADER` Yapısı ve Tanımları (`Image.h`):**
    *   `Image.h` dosyası, `CImage` sınıfının tanımından önce `TGA_HEADER` adında bir yapı ve TGA formatıyla ilgili bazı makro tanımları (`IMAGEDESC_ORIGIN_MASK` vb.) içerir. Bu, TGA resimlerini işlemek için gerekli başlık bilgisini tanımlar. Bu yapının `CImage` sınıfıyla doğrudan bir ilişkisi olmasa da, genellikle TGA yükleyicisi aynı kütüphane içinde olacağından buraya dahil edilmiş olabilir.
    *   `#pragma pack(push)` ve `#pragma pack(1)` direktifleri, `TGA_HEADER` yapısının bellekte byte hizalaması olmadan paketlenmesini sağlar, bu da dosya formatlarıyla çalışırken önemlidir.

*   **`CImage` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_pdwColors`: Resmin piksel verilerini tutan `DWORD` (genellikle 32-bit ARGB veya ABGR renk formatı) işaretçisi.
        *   `m_width`: Resmin piksel cinsinden genişliği.
        *   `m_height`: Resmin piksel cinsinden yüksekliği.
        *   `m_stFileName`: Resimle ilişkilendirilmiş dosya adını tutan bir string.
    *   **Kurucu ve Yok Ediciler:**
        *   `CImage()`: Varsayılan kurucu. `Initialize()` çağırır.
        *   `CImage(CImage& image)`: Kopyalama kurucusu. Verilen `CImage` nesnesinin bir kopyasını oluşturur. Yeni bir bellek alanı ayırır ve piksel verilerini kopyalar.
        *   `virtual ~CImage()`: Sanal yok edici. `Destroy()` çağırarak ayrılan belleği serbest bırakır. Sanal olması, bu sınıftan türetilmiş sınıfların da kaynaklarını doğru şekilde temizleyebilmesini sağlar.
    *   **Temel Metotlar ve İşlevler:**
        *   `Initialize()`: Üye değişkenleri (`m_pdwColors`'ı `NULL`, `m_width` ve `m_height`'ı 0) başlangıç durumlarına ayarlar.
        *   `Destroy()`: `m_pdwColors` tarafından tutulan piksel verisi için ayrılmış belleği `delete[]` ile serbest bırakır ve işaretçiyi `NULL` yapar.
        *   `Create(int width, int height)`: Belirtilen genişlik ve yükseklikte yeni bir resim tamponu oluşturur. Önce mevcut bir tampon varsa `Destroy()` ile onu temizler, ardından `width * height` boyutunda bir `DWORD` dizisi ayırır ve boyutları günceller.
        *   `Clear(DWORD color = 0)`: Resim tamponunun tamamını belirtilen `color` değeriyle (varsayılan olarak siyah veya şeffaf siyah) doldurur.
        *   `GetWidth() const`: Resmin genişliğini döndürür.
        *   `GetHeight() const`: Resmin yüksekliğini döndürür.
        *   `GetBasePointer()`: Piksel verilerinin başlangıç adresine (`m_pdwColors`) bir işaretçi döndürür.
        *   `GetLinePointer(int line)`: Resmin belirtilen satırının (`line`) başlangıcına bir işaretçi döndürür.
        *   `PutImage(int x, int y, CImage* pImage)`: Başka bir `CImage` nesnesini (`pImage`) bu resmin belirtilen `(x, y)` koordinatlarına kopyalar (çizer). Kaynak resmin bu resmin sınırları içinde olduğundan emin olmak için `assert` kullanılır.
        *   `FlipTopToBottom()`: Resmi dikey olarak ters çevirir (üst satırlarla alt satırları yer değiştirir). Bu, bazı resim formatlarının (örneğin TGA) veriyi alttan üste saklaması nedeniyle gerekebilir. Geçici bir tampon (`swap`) kullanarak satırları kopyalar.
        *   `SetFileName(const char* c_szFileName)`: Resmin dosya adını ayarlar.
        *   `GetFileNameString() const`: Resmin dosya adını döndürür.
        *   `IsEmpty() const`: Resim tamponunun oluşturulup oluşturulmadığını (`m_pdwColors`'ın `NULL` olup olmadığını) kontrol eder.
*   **Kullanım Alanı:**
    *   `CImage`, oyun içinde 32-bit renk formatında ham piksel verilerini tutmak ve bu verilere temel erişim sağlamak için kullanılır.
    *   Farklı resim dosyası formatlarını (TGA, DDS vb.) yükleyen sınıflar, bu `CImage` sınıfını hedef format olarak kullanarak yükledikleri veriyi bu yapıya dönüştürebilirler.
    *   Basit resim manipülasyonları (kopyalama, temizleme, ters çevirme) için temel işlevler sunar.
    *   Oluşturulan resim verileri daha sonra grafik motoru tarafından doku (texture) oluşturmak için kullanılabilir.

    ### `TGAImage.h` ve `TGAImage.cpp` (`CTGAImage` Sınıfı)

*   **Amaç:** `CTGAImage` sınıfı, `CImage` sınıfından miras alarak TGA (Truevision Graphics Adapter) formatındaki resim dosyalarını yüklemek ve kaydetmek için özelleşmiş işlevsellik sunar. Farklı TGA türlerini (gri tonlamalı, sıkıştırılmamış renkli, RLE sıkıştırılmış renkli) ve çeşitli bit derinliklerini (16, 24, 32-bit) işleyebilir.

*   **`CTGAImage` Sınıfı:**
    *   **Kalıtım:** `public CImage`
    *   **Enum `ETGAImageFlags`:**
        *   `FLAG_RLE_COMPRESS = (1 << 0)`: Resmin RLE sıkıştırmalı olarak kaydedilip kaydedilmeyeceğini belirten bir bayrak.
    *   **Temel Değişkenler:**
        *   `m_Header`: Yüklenen veya kaydedilecek TGA dosyasının başlık bilgilerini (`TGA_HEADER` yapısı) tutar.
        *   `m_dwFlag`: Resimle ilgili bayrakları (örneğin, `FLAG_RLE_COMPRESS`) tutar.
        *   `m_pdwEndPtr`: `SaveToDiskFile` içinde RLE sıkıştırması sırasında kullanılan bir işaretçi (ancak mevcut `TGAImage.cpp` içeriğinde tam kullanımı görünmüyor, belki de eksik veya kaldırılmış bir özellik).
    *   **Kurucu ve Yok Ediciler:**
        *   `CTGAImage()`: Varsayılan kurucu. `m_dwFlag`'ı 0 olarak ayarlar.
        *   `CTGAImage(CImage& image)`: Bir `CImage` nesnesinden `CTGAImage` oluşturur. Temel `CImage`'in verilerini kopyalar ve TGA formatının orijin beklentisi nedeniyle `FlipTopToBottom()` çağırır.
        *   `virtual ~CTGAImage()`: Sanal yok edici.
    *   **Ana Metotlar ve İşlevler:**
        *   `Create(int width, int height)`: `CImage::Create` çağrısının üzerine, `m_Header` için varsayılan TGA başlık bilgilerini (sıkıştırılmamış, 32-bit renkli, alfa kanalı için desc biti ayarlanmış) ayarlar.
        *   `LoadFromMemory(int iSize, const BYTE* c_pbMem)`: Bellekteki bir TGA dosyası verisinden resmi yükler.
            1.  İlk 18 byte'ı `m_Header`'a kopyalar.
            2.  `CImage::Create` ile resim tamponunu oluşturur.
            3.  `m_Header.imgType` (resim tipi) ve `m_Header.colorBits` (renk bit derinliği) değerlerine göre bir `switch` bloğu içinde farklı TGA formatlarını işler:
                *   **Tip 3 (Sıkıştırılmamış Gri Tonlamalı):** Her byte'ı okur ve R, G, B kanallarına aynı değeri, A'ya 255 atayarak DWORD oluşturur.
                *   **Tip 2 (Sıkıştırılmamış Renkli):**
                    *   **16-bit:** Her 2 byte'ı okur (WORD), 5 bit R, 5 bit G, 5 bit B olarak ayrıştırır, 8-bit'e genişletir ve A=255 ile DWORD oluşturur.
                    *   **24-bit:** Her 3 byte'ı (R, G, B sırasıyla) okur, A=255 ile DWORD oluşturur.
                    *   **32-bit:** Tüm piksel verisini doğrudan `memcpy` ile `m_pdwColors`'a kopyalar.
                *   **Tip 10 (RLE Sıkıştırılmış Renkli):**
                    *   Bir RLE paketi başlığını (BYTE) okur.
                    *   Eğer paket başlığının en yüksek biti 0 ise (`rle < 0x80`), sonraki `rle+1` adet piksel sıkıştırılmamıştır ve doğrudan okunur.
                    *   Eğer paket başlığının en yüksek biti 1 ise (`rle >= 0x80`), sonraki `rle-127` adet piksel, takip eden tek bir piksel değerinin tekrarıdır.
                    *   Bu işlem 24-bit ve 32-bit RLE sıkıştırmalı veriler için ayrı ayrı ele alınır. RLE taşmalarına karşı `assert` ve `printf` ile kontrol yapılır.
            4.  Yükleme tamamlandıktan sonra, TGA başlığındaki `desc` alanının 0x20 bitine (genellikle üstten-alta orijini belirtir) bakarak, eğer resim alttan-üste saklanmışsa (`!(m_Header.desc & 0x20)`), `FlipTopToBottom()` çağırarak resmi dikey olarak çevirir.
        *   `LoadFromDiskFile(const char* c_szFileName)`: Verilen dosya adından TGA resmini yükler.
            *   `CMappedFile` kullanarak dosyayı belleğe map eder.
            *   `LoadFromMemory` fonksiyonunu çağırarak yükleme işlemini gerçekleştirir.
        *   `SaveToDiskFile(const char* c_szFileName)`: Resmi belirtilen dosyaya TGA formatında kaydeder.
            *   Dosyayı yazma modunda açar.
            *   `m_Header`'ı dosyaya yazar.
            *   Eğer `m_dwFlag` içinde `FLAG_RLE_COMPRESS` ayarlanmışsa ve resim 32-bit ise RLE sıkıştırmalı kaydetmeye çalışır:
                *   Piksel satırlarını teker teker işler.
                *   Her satırda, aynı olan ardışık pikselleri sayarak veya farklı olan ardışık pikselleri sayarak RLE paketleri oluşturur ve dosyaya yazar. Hem tekrar eden piksel paketleri (en yüksek bit 1) hem de ham piksel paketleri (en yüksek bit 0) oluşturabilir.
            *   Eğer sıkıştırma istenmemişse veya uygun değilse, resmi sıkıştırılmamış olarak kaydeder (32-bit piksel verisini doğrudan dosyaya yazar).
            *   Dosyayı kapatır.
        *   `SetCompressed(bool isCompress = true)`: `m_dwFlag`'da `FLAG_RLE_COMPRESS` bitini ayarlar veya temizler.
        *   `SetAlphaChannel(bool isExist = true)`: `m_Header.desc` içindeki alfa kanalıyla ilgili bitleri (0x08) ayarlar veya temizler. Ayrıca `m_Header.colorBits`'i 32 veya 24 olarak günceller.
        *   `GetHeader()`: `m_Header`'a referans döndürür.
        *   `GetRawPixelCount(const DWORD* data)` ve `GetRLEPixelCount(const DWORD* data)`: Bu fonksiyonlar `TGAImage.h`'de deklare edilmiş ancak `TGAImage.cpp` dosyasının sağlanan kısmında implementasyonları görünmüyor. Muhtemelen RLE sıkıştırma verimliliğini hesaplamak veya RLE verisinin boyutunu tahmin etmek için kullanılmış olabilecek, ancak şu anki implementasyonda aktif olmayan veya kaldırılmış fonksiyonlardır.

*   **Kullanım Alanı:**
    *   `CTGAImage` sınıfı, Metin2 istemcisinin TGA formatındaki dokuları ve arayüz resimlerini yüklemesi ve potansiyel olarak kaydetmesi (örneğin, ekran görüntüsü alma gibi bir işlevsellik için) için kullanılır.
    *   Farklı bit derinlikleri ve sıkıştırma türleriyle başa çıkabilme yeteneği, çeşitli kaynaklardan gelen TGA dosyalarıyla uyumluluk sağlar.
    *   `CMappedFile` kullanımı, büyük resim dosyalarının diskten okunmasında verimlilik sağlayabilir.

### `DXTCImage.h` ve `DXTCImage.cpp` (`CDXTCImage` Sınıfı)

*   **Amaç:** `CDXTCImage` sınıfı, DDS (DirectDraw Surface) dosyalarını yüklemek ve bu dosyalarda bulunan DXTC (S3 Texture Compression - DXT1, DXT3, DXT5) formatlarındaki sıkıştırılmış doku verilerini açmak (decompress) için kullanılır. Ayrıca sıkıştırılmamış ARGB formatındaki DDS dosyalarını da destekler ve mipmap seviyelerini yönetebilir.

*   **Önemli Yapılar ve Enum'lar (`DXTCImage.h`):**
    *   **`EPixFormat` Enum:** Desteklenen piksel formatlarını tanımlar: `PF_ARGB`, `PF_DXT1`, `PF_DXT2` (kullanılmıyor gibi), `PF_DXT3`, `PF_DXT4` (kullanılmıyor gibi), `PF_DXT5`, `PF_UNKNOWN`.
    *   **`_XDDPIXELFORMAT` (typedef `XDDPIXELFORMAT`):** DDS dosyasının başlığında bulunan `DDS_PIXELFORMAT` yapısına çok benzer bir yapıdır. Piksel formatı, FourCC kodu, bit maskeleri gibi detaylı bilgileri içerir. `DUMMYUNIONNAMEN` makroları, C ve C++ uyumluluğu için isimsiz union'ları yönetir.
    *   **`DXTColBlock`, `DXTAlphaBlockExplicit`, `DXTAlphaBlock3BitLinear` (`DXTCImage.cpp` içinde):** DXTC sıkıştırma bloklarının yapılarını tanımlar.
        *   `DXTColBlock`: DXT1, DXT3 ve DXT5'in renk verilerini (2 renk ve 4x4 piksel için interpolasyon bitleri) içerir.
        *   `DXTAlphaBlockExplicit`: DXT3'ün alfa verilerini (her piksel için 4-bit explicit alfa) içerir.
        *   `DXTAlphaBlock3BitLinear`: DXT5'in alfa verilerini (2 referans alfa ve 4x4 piksel için 3-bit interpolasyon) içerir.
    *   **`Color8888`, `Color565` (`DXTCImage.cpp` içinde):** Piksel renklerini farklı formatlarda temsil etmek için kullanılan yardımcı yapılar.

*   **`CDXTCImage` Sınıfı:**
    *   **Temel Değişkenler:**
        *   `m_pbCompBufferByLevels[MAX_MIPLEVELS]`: Her mipmap seviyesi için sıkıştırılmış ham veri işaretçisi (dosya belleğe map edildiğinde doğrudan burayı gösterir).
        *   `m_bCompVector[MAX_MIPLEVELS]`: Her mipmap seviyesi için sıkıştırılmış verilerin bir kopyasını tutan `std::vector<BYTE>`. Veriler buraya kopyalanır.
        *   `m_nCompSize`, `m_nCompLineSz`: Sıkıştırılmış verinin toplam boyutu ve bir satırının boyutu (genellikle pitch ile ilgilidir).
        *   `m_strFormat[32]`: Piksel formatının string temsili (örneğin, "DXT1").
        *   `m_CompFormat`: `EPixFormat` türünden sıkıştırma formatı.
        *   `m_lPitch`: Yüzeyin pitch'i (bir satırın byte cinsinden uzunluğu).
        *   `m_dwMipMapCount`: Mipmap seviyesi sayısı.
        *   `m_bMipTexture`: Mipmap olup olmadığını belirten bayrak.
        *   `m_dwFlags`: DDS başlığından okunan `dwFlags` (örneğin, `DDSD_PITCH`, `DDSD_MIPMAPCOUNT`).
        *   `m_nWidth`, `m_nHeight`: Resmin piksel cinsinden genişliği ve yüksekliği.
        *   `m_xddPixelFormat`: Resmin `XDDPIXELFORMAT` yapısı.
    *   **Kurucu ve Temizleme:**
        *   `CDXTCImage()`: Kurucu. `Initialize()` çağırır.
        *   `~CDXTCImage()`: Yok edici.
        *   `Initialize()`: Üye değişkenleri varsayılan değerlerine sıfırlar/ayarlar. `m_pbCompBufferByLevels` işaretçilerini `NULL` yapar.
        *   `Clear()`: Tüm mipmap seviyeleri için `m_bCompVector`'leri temizler ve `Initialize()` çağırır.
    *   **Yükleme Metotları:**
        *   `LoadFromFile(const char* filename)`: Belirtilen dosyadan DDS resmini yükler.
            *   Dosya uzantısının ".DDS" olup olmadığını kontrol eder.
            *   `CMappedFile` kullanarak dosyayı belleğe map eder.
            *   `LoadFromMemory` fonksiyonunu çağırır.
        *   `LoadHeaderFromMemory(const BYTE* c_pbMap, int iSize)`: Bellekteki veriden sadece DDS başlığını okur ve sınıf üyelerini (genişlik, yükseklik, format, mipmap sayısı vb.) doldurur.
            *   "DDS " (dwMagic) imzasını kontrol eder (ancak kodda bu kontrol yorum satırı içinde).
            *   `DDSURFACEDESC2` yapısını okur.
            *   Piksel formatını `DecodePixelFormat` ile analiz ederek `m_CompFormat`'ı belirler.
            *   Mipmap verilerinin başlangıç işaretçilerini (`m_pbCompBufferByLevels`) ayarlar.
        *   `LoadFromMemory(const BYTE* c_pbMap, int iSize)`: Bellekteki veriden DDS resmini yükler (başlık ve piksel verileri).
            *   Önce `LoadHeaderFromMemory` çağırarak başlığı yükler.
            *   Ardından, mipmap seviyelerine göre sıkıştırılmış piksel verilerini `m_pbCompBufferByLevels`'dan `m_bCompVector`'lere kopyalar. Pitch veya linear size durumlarına göre farklı kopyalama mantıkları uygular.
    *   **Veri Kopyalama ve Açma (Decompression) Metotları:**
        *   `Copy(int miplevel, BYTE* pbDest, long lDestPitch)`: Belirtilen mipmap seviyesindeki sıkıştırılmış veriyi (`m_bCompVector[miplevel]`) hedef bir bellek alanına (`pbDest`) kopyalar. Pitch'i dikkate alır.
        *   `Decompress(int miplevel, DWORD* pdwDest)`: Ana dekompresyon fonksiyonu. `m_CompFormat`'a göre uygun olan özel dekompresyon fonksiyonunu (örn: `DecompressDXT1`) çağırır.
        *   `DecompressDXT1(int miplevel, DWORD* pdwDest)`: DXT1 formatındaki veriyi açar.
            *   Sıkıştırılmış veriyi (`m_bCompVector[miplevel]`) 4x4 bloklar halinde okur.
            *   Her `DXTColBlock` için:
                *   `GetColorBlockColors` yardımcı fonksiyonu ile 2 ana rengi (col0, col1) okur ve bunlardan 2 ara renk (col2, col3) türetir (DXT1'de col0 > col1 ise 2 ara renk, değilse 1 ara renk ve 1 transparan renk).
                *   `DecodeColorBlock` yardımcı fonksiyonu ile bloktaki her piksel için 2-bitlik indeksleri kullanarak bu 4 renkten uygun olanı seçer ve hedef `pdwDest` tamponuna 32-bit ARGB olarak yazar.
        *   `DecompressDXT3(int miplevel, DWORD* pdwDest)`: DXT3 formatındaki veriyi açar.
            *   Her 4x4 blok için önce `DXTAlphaBlockExplicit` (8 byte) okur. `DecodeAlphaExplicit` ile her piksel için 4-bitlik explicit alfa değerini alır.
            *   Ardından `DXTColBlock` (8 byte) okur ve renkleri `DecompressDXT1`'e benzer şekilde (ancak DXT3 her zaman 4 renk kullanır) açar. Alfa ve renk bilgilerini birleştirerek hedef tampona yazar.
        *   `DecompressDXT5(int miplevel, DWORD* pdwDest)`: DXT5 formatındaki veriyi açar.
            *   Her 4x4 blok için önce `DXTAlphaBlock3BitLinear` (8 byte) okur. `DecodeAlpha3BitLinear` ile 2 referans alfa değerini (alpha0, alpha1) ve her piksel için 3-bitlik interpolasyon indekslerini kullanarak 8 olası alfa değerinden birini seçer.
            *   Ardından `DXTColBlock` (8 byte) okur ve renkleri DXT3'e benzer şekilde açar. Alfa ve renk bilgilerini birleştirir.
        *   `DecompressARGB(int miplevel, DWORD* pdwDest)`: Sıkıştırılmamış ARGB formatındaki veriyi doğrudan hedef tampona kopyalar.
    *   **Yardımcı Metotlar:**
        *   `DecodePixelFormat(CHAR* strPixelFormat, XDDPIXELFORMAT* pddpf)`: `XDDPIXELFORMAT` yapısını analiz ederek `m_CompFormat`'ı ve formatın string temsilini (`strPixelFormat`) belirler. FourCC kodlarına (`MAKEFOURCC('D','X','T','1')` vb.) ve bit maskelerine bakar.
        *   `Unextract(BYTE* pbDest, int iWidth, int iHeight, int iPitch)`: Bu fonksiyonun tam amacı net değil, ancak sıkıştırılmış veriyi (muhtemelen DXT1) belirli bir pitch'e göre yeniden düzenleyerek `pbDest`'e yazıyor gibi görünüyor. İsmi, bir tür "extract" işleminin tersi olduğunu ima ediyor.
        *   `GetColorBlockColors` (inline, cpp içinde): Bir `DXTColBlock`'tan 4 adet 32-bit ARGB rengi üretir.
        *   `DecodeColorBlock` (inline, cpp içinde): Bir `DXTColBlock`'u ve üretilmiş 4 rengi kullanarak 4x4 piksellik bir alanı 32-bit ARGB formatına dönüştürür.
        *   `DecodeAlphaExplicit` (inline, cpp içinde): Bir `DXTAlphaBlockExplicit`'i kullanarak 4x4 piksellik bir alanın alfa değerlerini 32-bit ARGB formatına (alfa kanalına) yazar.
        *   `DecodeAlpha3BitLinear` (inline, cpp içinde): Bir `DXTAlphaBlock3BitLinear`'ı kullanarak 4x4 piksellik bir alanın alfa değerlerini 32-bit ARGB formatına yazar.
    *   **Yorum Satırına Alınmış Zamanlama (Timing) Fonksiyonları:**
        *   `.h` dosyasında ve `.cpp` dosyasının sonunda dekompresyon algoritmalarının performansını test etmek için kullanılan `RunTimingSession` ve `Time_Decomp5_xx` gibi fonksiyonların bildirimleri ve muhtemel implementasyonları yorum satırı içinde bulunmaktadır. Bunlar geliştirme sırasında kullanılmış olabilir.

*   **Kullanım Alanı:**
    *   `CDXTCImage`, oyun motorunun DDS dosyalarından doku yüklemesi için kritik bir bileşendir. DXTC formatları, doku verilerini sıkıştırarak diskte daha az yer kaplamasını ve GPU belleğine daha hızlı yüklenmesini sağlar.
    *   Yüklenen ve dekompres edilen piksel verileri (genellikle `Decompress` çağrıldıktan sonra elde edilen 32-bit ARGB verisi), Direct3D gibi grafik API'leri aracılığıyla doku (texture) nesneleri oluşturmak için kullanılır.
    *   Mipmap desteği, farklı mesafelerde dokuların daha iyi görünmesini ve "shimmering" artefaktlarının azalmasını sağlar.

