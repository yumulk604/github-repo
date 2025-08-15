# Metin2 Oyun Sunucusu - Yardımcı Bileşenler Referansı Part 3 (`game/src`)

**Not:** Bu belge, [`game_Utils_Referans_Part2.md`](game_Utils_Referans_Part2.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) çeşitli yardımcı bileşenleri ve sistemleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`MarkImage.h`](#markimageh)
*   [`MarkImage.cpp`](#markimagecpp)
*   [`MarkConvert.cpp`](#markconvertcpp)
*   [`MarkManager.h`](#markmanagerh)
*   [`MarkManager.cpp`](#markmanagercpp)
*   [`messenger_manager.h`](#messenger_managerh)
*   [`messenger_manager.cpp`](#messenger_managercpp)
*   [`minilzo.h`](#minilzoh)
*   [`minilzo.c`](#minilzoc)
*   [`object_allocator.h`](#object_allocatorh)
*   [`panama.h`](#panamah)
*   [`panama.cpp`](#panamacpp)
*   [`pool.h`](#poolh)
*   [`profiler.h`](#profilerh)
*   [`protocol.h`](#protocolh)
*   [`spam.h`](#spamh)
*   [`stable_priority_queue.h`](#stable_priority_queueh)
*   [`state.h`](#stateh)
*   [`text_file_loader.h`](#text_file_loaderh)
*   [`text_file_loader.cpp`](#text_file_loadercpp)
*   [`TrafficProfiler.h`](#trafficprofilerh)
*   [`TrafficProfiler.cpp`](#trafficprofilercpp)
*   [`unique_item.h`](#unique_itemh)
*   [`unique_mob.h`](#unique_mobh)
*   [`update_limit_time.py`](#update_limit_timepy)
*   [`utils.h`](#utilsh)
*   [`utils.cpp`](#utilscpp)
*   [`vector.h`](#vectorh)
*   [`vector.cpp`](#vectorcpp)
*   [`vid.h`](#vidh)
*   [`version.cpp`](#versioncpp)

---

### `MarkImage.h`

*   **Amaç:** Lonca amblemlerini (`guild_mark.tga` gibi bir ana resim dosyası içinde) yönetmek için kullanılan sınıfları ve yapıları tanımlar. Bu, amblemlerin yüklenmesi, kaydedilmesi, tekil amblem olarak (16x12 piksel) ve daha büyük bloklar (64x48 piksel) halinde işlenmesi, sıkıştırılması ve CRC kontrollerini içerir. DevIL (Developer's Image Library) kütüphanesini resim işlemleri için kullanır.
*   **Temel İşlevler/İçerik:**
    *   **`Pixel` Typedef'i (`unsigned long`):** Genellikle 32-bit bir piksel formatını (örn. BGRA) temsil eder.
    *   **`SGuildMark` Struct:**
        *   Tek bir lonca ambleminin (16x12 piksel) verilerini tutar.
        *   `WIDTH`, `HEIGHT`, `SIZE` sabitleri.
        *   `m_apxBuf[SIZE]` (Pixel dizisi): Ham piksel verisi.
        *   `Clear()`: Amblemi şeffaf (veya belirli bir varsayılan renge) temizler.
        *   `IsEmpty()`: Amblemin tamamen boş (siyah veya şeffaf) olup olmadığını kontrol eder.
    *   **`SGuildMarkBlock` Struct:**
        *   Bir grup lonca amblemini (4x4 amblem, yani 64x48 piksel) içeren bir "blok" yapısını tanımlar. Bu bloklar, istemciye verimli bir şekilde gönderilmek üzere sıkıştırılır.
        *   `MARK_PER_BLOCK_WIDTH`, `MARK_PER_BLOCK_HEIGHT`, `WIDTH`, `HEIGHT`, `SIZE` sabitleri.
        *   `MAX_COMP_SIZE`: Bir bloğun LZO ile sıkıştırıldıktan sonraki maksimum boyutunu tanımlar.
        *   `m_apxBuf[SIZE]` (Pixel dizisi): Bloğun ham piksel verisi (kullanılmıyor gibi, sadece sıkıştırılmış veri tutuluyor).
        *   `m_abCompBuf[MAX_COMP_SIZE]` (BYTE dizisi): Bloğun LZO ile sıkıştırılmış verisi.
        *   `m_sizeCompBuf` (lzo_uint): Sıkıştırılmış verinin boyutu.
        *   `m_crc` (DWORD): Bloğun sıkıştırılmamış ham piksel verisinin CRC32 değeri.
        *   `GetCRC()`: Bloğun CRC değerini döndürür.
        *   `CopyFrom(const BYTE* pbCompBuf, DWORD dwCompSize, DWORD crc)`: Verilen sıkıştırılmış veriyi ve CRC'yi bloğa kopyalar.
        *   `Compress(const Pixel* pxBuf)`: Verilen ham piksel verisini LZO ile sıkıştırır, `m_abCompBuf`'a yazar, boyutunu `m_sizeCompBuf`'ta saklar ve `m_crc`'yi hesaplar.
    *   **`CGuildMarkImage` Sınıfı:**
        *   Ana lonca amblemi resim dosyasını (genellikle 512x512 piksel) yönetir.
        *   **Sabitler:** `WIDTH`, `HEIGHT` (ana resim boyutu), `BLOCK_ROW_COUNT`, `BLOCK_COL_COUNT`, `BLOCK_TOTAL_COUNT` (ana resimdeki blok sayısı), `MARK_ROW_COUNT`, `MARK_COL_COUNT`, `MARK_TOTAL_COUNT` (ana resimdeki toplam amblem slotu sayısı), `INVALID_MARK_POSITION`.
        *   **Yapıcı/Yıkıcı (`CGuildMarkImage()`, `~CGuildMarkImage()`):** DevIL resim handle'ını (`m_uImg`) başlatır/serbest bırakır.
        *   **Resim Yönetimi (`Create`, `Destroy`, `Build`, `Save`, `Load`):**
            *   `Create()`: Yeni bir DevIL resmi oluşturur (`ilGenImages`).
            *   `Destroy()`: Mevcut DevIL resmini siler (`ilDeleteImages`).
            *   `Build(c_szFileName)`: Belirtilen isimde yeni, boş bir TGA dosyası (512x512, BGRA formatında) oluşturur ve kaydeder.
            *   `Save(c_szFileName)`: Mevcut DevIL resmini TGA dosyası olarak kaydeder.
            *   `Load(c_szFileName)`: TGA dosyasını yükler, boyutunu doğrular, BGRA formatına dönüştürür ve `BuildAllBlocks()` ile tüm blokları sıkıştırıp CRC'lerini hesaplar.
        *   **Piksel Veri Erişimi (`PutData`, `GetData`):** DevIL fonksiyonlarını (`ilSetPixels`, `ilCopyPixels`) kullanarak ana resim üzerindeki belirli bir bölgeye piksel verisi yazar veya okur.
        *   **Sunucu Tarafı Amblem İşlemleri:**
            *   `SaveMark(DWORD posMark, BYTE* pbMarkImage)`: Belirli bir pozisyondaki (0-1279) amblemi günceller. Verilen 16x12'lik amblem verisini ana resme (`PutData`) yazar ve bu amblemi içeren bloğu yeniden sıkıştırır/CRC'sini günceller (`m_aakBlock`).
            *   `DeleteMark(DWORD posMark)`: Belirli bir pozisyondaki amblemi boş bir amblemle değiştirerek siler (`SaveMark` kullanarak).
        *   **İstemci Tarafı Blok İşlemleri:**
            *   `SaveBlockFromCompressedData(DWORD posBlock, const BYTE* pbComp, DWORD dwCompSize)`: İstemciden gelen sıkıştırılmış bir blok verisini alır, LZO ile açar (`lzo1x_decompress_safe`), ana resme yazar (`PutData`) ve `m_aakBlock` dizisindeki ilgili bloğu bu sıkıştırılmış veri ve CRC ile günceller.
        *   **Yardımcı Metotlar:**
            *   `GetEmptyPosition()`: Ana resimdeki boş bir amblem slotunun pozisyonunu bulur.
            *   `GetBlockCRCList(DWORD* crcList)`: Tüm blokların CRC değerlerini bir diziye kopyalar.
            *   `GetDiffBlocks(const DWORD* crcList, std::map<BYTE, const SGuildMarkBlock*>& mapDiffBlocks)`: İstemciden gelen CRC listesiyle sunucudaki blok CRC'lerini karşılaştırır ve farklı olan blokları `mapDiffBlocks`'a ekler.
        *   **Özel (Private) Üyeler:**
            *   `BuildAllBlocks()`: Ana resmi yükledikten sonra, tüm 64x48'lik blokları `GetData` ile okur, `SGuildMarkBlock::Compress` ile sıkıştırır ve `m_aakBlock` dizisinde saklar.
            *   `m_aakBlock[BLOCK_ROW_COUNT][BLOCK_COL_COUNT]` (`SGuildMarkBlock` dizisi): Ana resimdeki tüm sıkıştırılmış blokları ve CRC'lerini tutar.
            *   `m_apxImage[WIDTH * HEIGHT * sizeof(Pixel)]` (Pixel dizisi): Ana resmin ham piksel verilerini tutar (muhtemelen doğrudan kullanılmıyor, DevIL handle'ı üzerinden erişiliyor).
            *   `m_uImg` (ILuint): DevIL kütüphanesi tarafından kullanılan resim tanıtıcısı (handle).
*   **Bağlantılı Dosyalar:** `MarkImage.cpp` (uygulama), `minilzo.h` (LZO sıkıştırma), `<IL/il.h>` (DevIL kütüphanesi).

### `MarkImage.cpp`

*   **Amaç:** `MarkImage.h` dosyasında bildirilen `CGuildMarkImage` ve `SGuildMarkBlock` sınıflarının metotlarını uygular. DevIL kütüphanesini kullanarak lonca amblem resimlerini (TGA formatında) yükleme, oluşturma, kaydetme, piksel verilerini okuma/yazma işlemlerini gerçekleştirir. Ayrıca, amblem bloklarını LZO algoritması ile sıkıştırma, açma ve CRC32 hesaplama işlevlerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Fonksiyonlar:**
        *   `NewMarkImage()`: `CGuildMarkImage` nesnesi oluşturur (`M2_NEW` kullanarak).
        *   `DeleteMarkImage(CGuildMarkImage* pkImage)`: `CGuildMarkImage` nesnesini siler (`M2_DELETE` kullanarak).
    *   **`CGuildMarkImage` Sınıfı Metotları:**
        *   **Yapıcı/Yıkıcı:** `m_uImg`'yi `INVALID_HANDLE` olarak başlatır. Yıkıcı `Destroy()`'u çağırır.
        *   **`Destroy()`:** Geçerli bir DevIL resim handle'ı (`m_uImg`) varsa `ilDeleteImages` ile serbest bırakır.
        *   **`Create()`:** Yeni bir DevIL resim handle'ı oluşturur (`ilGenImages`).
        *   **`Save(c_szFileName)`:** `ilEnable(IL_FILE_OVERWRITE)` ile dosyanın üzerine yazmayı etkinleştirir, `ilBindImage` ile resmi bağlar ve `ilSave(IL_TGA, ...)` ile TGA formatında kaydeder.
        *   **`Build(c_szFileName)`:** Var olan resmi `Destroy` eder, yenisini `Create` eder. `ilTexImage` ile 512x512 boyutunda, 4 kanallı (BGRA), boş (siyah/şeffaf) bir resim oluşturur ve `Save` ile dosyaya yazar. Orijin sol üst köşe olarak ayarlanır (`IL_ORIGIN_UPPER_LEFT`).
        *   **`Load(c_szFileName)`:** Resmi `Destroy` eder, yenisini `Create` eder. `ilLoad` ile dosyayı yükler, boyutlarını (`WIDTH`, `HEIGHT`) kontrol eder, `ilConvertImage(IL_BGRA, IL_UNSIGNED_BYTE)` ile BGRA formatına dönüştürür ve ardından `BuildAllBlocks()` çağırarak resimdeki tüm 64x48'lik blokları sıkıştırıp CRC'lerini hesaplar.
        *   **`PutData(...)`, `GetData(...)`:** `ilBindImage` ile resmi bağlar ve `ilSetPixels` veya `ilCopyPixels` DevIL fonksiyonlarını kullanarak resmin belirli bir dikdörtgen alanına piksel verisi yazar veya bu alandan piksel verisi okur.
        *   **`SaveMark(posMark, pbImage)`:** Verilen pozisyondaki (`posMark`) 16x12'lik amblemi günceller. `PutData` ile amblemi ana resme çizer. Ardından, bu amblemin bulunduğu 64x48'lik bloğun koordinatlarını hesaplar, `GetData` ile bu bloğun güncel piksel verisini okur ve `m_aakBlock[rowBlock][colBlock].Compress()` ile bloğu yeniden sıkıştırıp CRC'sini günceller.
        *   **`DeleteMark(posMark)`:** Tamamen boş (siyah/şeffaf) bir 16x12'lik amblem oluşturur ve bunu `SaveMark` ile ilgili pozisyona yazarak amblemi siler.
        *   **`SaveBlockFromCompressedData(posBlock, pbComp, dwCompSize)`:** Verilen sıkıştırılmış blok verisini (`pbComp`) LZOManager (`CLZO::Instance()`) kullanarak `lzo1x_decompress_safe` ile açar. Açılan piksel verisini `PutData` ile ana resme yazar. Son olarak, `m_aakBlock` dizisindeki ilgili bloğu, gelen sıkıştırılmış veri, boyutu ve açılmış veriden hesaplanan yeni CRC ile günceller (`CopyFrom`).
        *   **`BuildAllBlocks()`:** Resimdeki tüm 64x48'lik blokları dolaşır. Her blok için `GetData` ile piksel verisini okur ve `m_aakBlock[row][col].Compress()` ile sıkıştırıp CRC'sini hesaplayarak saklar.
        *   **`GetEmptyPosition()`:** Tüm 16x12'lik amblem slotlarını dolaşır. Her slot için `GetData` ile piksel verisini okur, `SGuildMark::IsEmpty()` ile boş olup olmadığını kontrol eder ve ilk boş bulunan slotun pozisyonunu döndürür. Boş slot yoksa `INVALID_MARK_POSITION` döndürür.
        *   **`GetDiffBlocks(crcList, mapDiffBlocks)`:** İstemciden gelen `crcList` ile sunucudaki `m_aakBlock` dizisindeki blokların CRC'lerini karşılaştırır. Farklı CRC'ye sahip blokları `mapDiffBlocks` haritasına ekler.
        *   **`GetBlockCRCList(crcList)`:** Sunucudaki tüm blokların CRC değerlerini (`m_aakBlock[row][col].GetCRC()`) verilen `crcList` dizisine kopyalar.
    *   **`SGuildMark` Metotları:**
        *   **`Clear()`:** Amblemin tüm piksellerini `0xff000000` (muhtemelen tam opak siyah veya özel bir şeffaflık değeri) olarak ayarlar.
        *   **`IsEmpty()`:** Amblemin tüm piksellerinin `0x00000000` (tamamen şeffaf siyah) olup olmadığını kontrol eder.
    *   **`SGuildMarkBlock` Metotları:**
        *   **`GetCRC()`:** Saklanan `m_crc` değerini döndürür.
        *   **`CopyFrom(pbCompBuf, dwCompSize, crc)`:** Verilen sıkıştırılmış veriyi (`pbCompBuf`) kendi `m_abCompBuf` tamponuna kopyalar, boyutu (`dwCompSize`) ve CRC'yi (`crc`) saklar.
        *   **`Compress(pxBuf)`:** Verilen ham piksel verisini (`pxBuf`) LZOManager (`CLZO::Instance()`) kullanarak `lzo1x_1_compress` ile sıkıştırır, sonucu `m_abCompBuf`'a yazar, sıkıştırılmış boyutu `m_sizeCompBuf`'ta saklar ve orijinal piksel verisinin CRC32 değerini (`GetCRC32`) hesaplayıp `m_crc`'de saklar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `MarkImage.h`, `crc32.h`, `lzo_manager.h`.

### `MarkConvert.cpp`

*   **Amaç:** Eski tip lonca amblemi dosyalarını (`guild_mark.idx` ve `guild_mark.tga`) yeni kullanılan formata (muhtemelen `CGuildMarkManager` tarafından yönetilen ve `mark/` dizini altında ayrı dosyalar halinde saklanan format) dönüştürmek için bir yardımcı program/fonksiyon içerir. Bu işlem genellikle sunucu güncellemesi veya eski verilerin yeni sisteme aktarılması sırasında bir kereye mahsus çalıştırılır.
*   **Temel İşlevler/İçerik:**
    *   **Sabit Tanımları:**
        *   `OLD_MARK_INDEX_FILENAME "guild_mark.idx"`: Eski amblem indeks dosyasının adı.
        *   `OLD_MARK_DATA_FILENAME "guild_mark.tga"`: Eski amblem resim verilerini içeren TGA dosyasının adı (genellikle 512x? boyutunda, çok sayıda amblemi bir arada tutan bir sprite sheet).
    *   **`LoadOldGuildMarkImageFile()` (static fonksiyon):**
        *   `OLD_MARK_DATA_FILENAME` (`guild_mark.tga`) dosyasını okur.
        *   Dosya içeriğini (512x512 piksel * piksel boyutu kadar) bir bellek alanına (`Pixel* dataPtr`) yükler.
        *   Bu bellek alanının işaretçisini döndürür. Hata durumunda `NULL` döner.
    *   **`GuildMarkConvert(const std::vector<DWORD>& vecGuildID)` (ana fonksiyon):**
        *   `mark` adında bir dizin oluşturur (Windows için `_mkdir`, diğerleri için `mkdir`).
        *   Eski indeks dosyası (`guild_mark.idx`) yoksa işlemi atlar ve `true` döner (dönüştürme gereksiz).
        *   `OLD_MARK_INDEX_FILENAME` dosyasını okuma modunda açar.
        *   `LoadOldGuildMarkImageFile()` ile eski TGA resim verisini belleğe yükler.
        *   `guild_mark.idx` dosyasını satır satır okur:
            *   Her satırdan `guild_id` (lonca ID) ve `mark_id` (eski TGA dosyasındaki amblem sırası) değerlerini `sscanf` ile ayrıştırır.
            *   Eğer `guild_id`, fonksiyona parametre olarak verilen `vecGuildID` (sadece aktif veya var olan loncaların ID'leri) içinde bulunmuyorsa bu loncayı atlar.
            *   `mark_id`'den eski TGA dosyasındaki amblemin başlangıç koordinatlarını (satır `row`, sütun `col`, piksel cinsinden `sx`, `sy`) hesaplar.
            *   Hesaplanan koordinatları kullanarak eski TGA verisinden (`oldImagePtr`) 16x12 boyutundaki amblem piksellerini geçici bir `Pixel mark[SGuildMark::SIZE]` dizisine kopyalar.
            *   `CGuildMarkManager::instance().SaveMark(guild_id, (BYTE*)mark)` fonksiyonunu çağırarak kopyalanan bu amblemi yeni formatta (ilgili lonca ID'si için) kaydeder.
        *   Belleğe yüklenen eski TGA verisini (`oldImagePtr`) `free` ile serbest bırakır.
        *   Okunan dosyayı (`fp`) kapatır.
        *   Eski `guild_mark.idx` ve `guild_mark.tga` dosyalarını `.removable` uzantısı ekleyerek yeniden adlandırır (Windows için `move`, diğerleri için `mv` sistem komutları ile).
        *   Başarılı olursa `true` döner.
*   **Çalışma Prensibi:** Bu fonksiyon, sunucuda kayıtlı olan ve `vecGuildID` ile belirtilen loncalara ait eski amblemleri okur. Eski sistemde amblemler büyük bir TGA dosyasında (`guild_mark.tga`) bir sprite sheet olarak tutulurken, hangi loncanın hangi amblemi kullandığı `guild_mark.idx` dosyasında belirtilirdi. `GuildMarkConvert`, bu iki dosyayı okuyarak her bir loncanın 16x12'lik amblem verisini çıkarır ve `CGuildMarkManager`'ın `SaveMark` metodunu kullanarak yeni sisteme uygun şekilde (genellikle her lonca için ayrı bir dosya veya yeni bir merkezi TGA dosyası içinde doğru pozisyona) kaydeder. İşlem sonunda eski dosyalar yeniden adlandırılarak tekrar kullanılmaları engellenir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `MarkManager.h` (özellikle `CGuildMarkManager` ve `SGuildMark::SIZE` için). Windows için `direct.h`, diğerleri için sistem başlıkları (`sys/stat.h`, `unistd.h` gibi `mkdir` ve `access` için, `stdafx.h` üzerinden gelebilir).

---

### `MarkManager.h`

*   **Amaç:** Lonca amblemlerini (mark) ve lonca sembollerini yönetmek için `CGuildMarkManager` singleton sınıfını tanımlar. Amblem ID'lerinin lonca ID'leri ile eşleştirilmesi, amblem resim dosyalarının yüklenmesi/kaydedilmesi, boş amblem ID'lerinin yönetimi ve istemci/sunucu arasındaki amblem senkronizasyonu gibi işlemleri içerir. Ayrıca lonca sembollerinin (daha büyük, özel resimler) yüklenmesi, kaydedilmesi ve yönetilmesi işlevlerini de barındırır.
*   **Temel İşlevler/İçerik:**
    *   **`MAX_IMAGE_COUNT` Enum'u:** Yönetilebilecek maksimum amblem resim dosyası sayısı (örn: `mark_0.tga`, `mark_1.tga` vb.).
    *   **`INVALID_MARK_ID` Enum'u:** Geçersiz bir amblem ID'sini belirtir.
    *   **`TGuildSymbol` Struct:**
        *   Bir lonca sembolünün verilerini tutar.
        *   `crc` (DWORD): Sembolün ham verisinin CRC32 değeri.
        *   `raw` (std::vector<BYTE>): Sembolün ham piksel veya sıkıştırılmış verisi.
    *   **`CGuildMarkManager` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:** Boş amblem ID'lerini (`m_setFreeMarkID`) başlatır, yüklenmiş resimleri (`m_mapIdx_Image`) temizler.
        *   **Sembol Yönetimi:**
            *   `GetGuildSymbol(DWORD GID)`: Belirtilen loncaya ait sembolü döndürür.
            *   `LoadSymbol(const char* filename)`: Sembol dosyasını yükler.
            *   `SaveSymbol(const char* filename)`: Sembolleri dosyaya kaydeder.
            *   `UploadSymbol(DWORD guildID, int iSize, const BYTE* pbyData)`: Bir lonca için yeni sembol verisi yükler.
        *   **Amblem (Mark) Yönetimi:**
            *   `SetMarkPathPrefix(const char* prefix)`: Amblem dosyalarının bulunduğu dizin için bir ön ek belirler (örn: "mark").
            *   `LoadMarkIndex()`: `mark/[prefix]_index` dosyasından lonca ID-amblem ID eşleşmelerini okur ve `LoadMarkImages()` ile ilgili resim dosyalarını yükler.
            *   `SaveMarkIndex()`: Mevcut lonca ID-amblem ID eşleşmelerini indeks dosyasına yazar.
            *   `LoadMarkImages()`: İndeks dosyasında referans verilen tüm amblem resim dosyalarını (`mark/[prefix]_[imgIdx].tga`) yükler.
            *   `SaveMarkImage(DWORD imgIdx)`: Belirli bir amblem resim dosyasını kaydeder.
            *   `GetMarkImageFilename(DWORD imgIdx, std::string& path)`: Verilen indeks için amblem resim dosyasının adını oluşturur.
            *   `AddMarkIDByGuildID(DWORD guildID, DWORD markID)`: Bir lonca ID'sini bir amblem ID'si ile eşleştirir ve bu amblem ID'sini boş listeden çıkarır.
            *   `GetMarkImageCount()`: Yüklenmiş amblem resim dosyası sayısını döndürür.
            *   `GetMarkCount()`: Eşleştirilmiş (kullanımda olan) amblem sayısını döndürür.
            *   `GetMarkID(DWORD guildID)`: Bir loncaya atanmış amblem ID'sini döndürür.
        *   **Sunucu Tarafı Amblem İşlemleri:**
            *   `CopyMarkIdx(char* pcBuf)`: Tüm lonca ID-amblem ID eşleşmelerini istemciye gönderilmek üzere bir tampona kopyalar.
            *   `SaveMark(DWORD guildID, BYTE* pbMarkImage)`: Bir lonca için yeni bir amblem kaydeder. Gerekirse yeni bir amblem ID'si (`__AllocMarkID`) ayırır, amblemi ilgili `CGuildMarkImage` nesnesine kaydeder (`pkImage->SaveMark`), amblem resim dosyasını ve indeksi günceller.
            *   `DeleteMark(DWORD guildID)`: Bir loncanın amblemini siler, amblem ID'sini boşa çıkarır ve indeksi günceller.
            *   `GetDiffBlocks(DWORD imgIdx, const DWORD* crcList, std::map<BYTE, const SGuildMarkBlock*>& mapDiffBlocks)`: İstemciden gelen CRC listesi ile sunucudaki bir amblem resminin bloklarını karşılaştırır ve farklı olan blokları döndürür (istemcinin güncellemesi için).
        *   **İstemci Tarafı Amblem İşlemleri:**
            *   `SaveBlockFromCompressedData(DWORD imgIdx, DWORD idBlock, const BYTE* pbBlock, DWORD dwSize)`: İstemciden gelen sıkıştırılmış bir amblem bloğunu alır ve ilgili `CGuildMarkImage`'a kaydeder.
            *   `GetBlockCRCList(DWORD imgIdx, DWORD* crcList)`: Belirli bir amblem resmindeki tüm blokların CRC listesini alır (istemcinin sunucuyla senkronize olması için).
        *   **Özel (Private) Üyeler:**
            *   `__NewImage()`, `__DeleteImage(CGuildMarkImage* pkImgDel)`: `CGuildMarkImage` nesneleri oluşturur/siler.
            *   `__AllocMarkID(DWORD guildID)`: Bir lonca için boş bir amblem ID'si bulur ve atar.
            *   `__GetImage(DWORD imgIdx)`: Belirli bir indeksteki `CGuildMarkImage` nesnesini döndürür (yoksa yükler veya oluşturur).
            *   `m_mapIdx_Image` (std::map<DWORD, CGuildMarkImage*>): Yüklenmiş amblem resimlerini (indeks -> `CGuildMarkImage*`) tutar.
            *   `m_mapGID_MarkID` (std::map<DWORD, DWORD>): Lonca ID -> Amblem ID eşleşmelerini tutar.
            *   `m_setFreeMarkID` (std::set<DWORD>): Kullanılmayan (boş) amblem ID'lerini tutar.
            *   `m_pathPrefix` (std::string): Amblem dosyaları için yol ön eki.
            *   `m_mapSymbol` (std::map<DWORD, TGuildSymbol>): Lonca ID -> Sembol verisi eşleşmelerini tutar.
*   **Bağlantılı Dosyalar:** `MarkManager.cpp` (uygulama), `MarkImage.h` (`CGuildMarkImage` tanımı), `stdafx.h`.

---

### `MarkManager.cpp`

*   **Amaç:** `MarkManager.h` dosyasında bildirilen `CGuildMarkManager` sınıfının metotlarını uygular. Lonca amblemlerinin ve sembollerinin diskten yüklenmesi, kaydedilmesi, ID'lerinin yönetilmesi ve istemci ile sunucu arasında senkronizasyonunun sağlanması gibi temel işlevleri yerine getirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`CGuildMarkManager()`):** `m_setFreeMarkID` setini, olası tüm amblem ID'leri (0'dan `MAX_IMAGE_COUNT * CGuildMarkImage::MARK_TOTAL_COUNT - 1`'e kadar) ile doldurarak başlatır.
    *   **Yıkıcı (`~CGuildMarkManager()`):** `m_mapIdx_Image` haritasındaki tüm `CGuildMarkImage` nesnelerini `__DeleteImage` ile siler.
    *   **Dosya Adı Yönetimi:**
        *   `GetMarkImageFilename(imgIdx, path)`: `mark/[prefix]_[imgIdx].tga` formatında dosya yolu oluşturur.
        *   `SetMarkPathPrefix(prefix)`: Amblem dosyalarının saklandığı `mark/` dizini altındaki ön eki (örn: "guild", "player" vb.) ayarlar.
    *   **İndeks Dosyası Yönetimi (`LoadMarkIndex`, `SaveMarkIndex`):**
        *   `LoadMarkIndex()`: `mark/[prefix]_index` dosyasını açar. Her satırdan `guildID` ve `markID` okur, `AddMarkIDByGuildID` ile eşleşmeyi kaydeder. Sonra `LoadMarkImages()` çağırır.
        *   `SaveMarkIndex()`: `m_mapGID_MarkID` haritasındaki tüm eşleşmeleri `mark/[prefix]_index` dosyasına "guildID markID" formatında yazar.
    *   **Amblem Resim Dosyası Yönetimi:**
        *   `LoadMarkImages()`: `m_mapGID_MarkID`'deki tüm amblem ID'lerini kontrol eder. Her bir amblemin ait olduğu resim dosyası (`markID / CGuildMarkImage::MARK_TOTAL_COUNT`) için `__GetImage()` çağırarak resmin yüklenmesini (veya oluşturulmasını) tetikler.
        *   `SaveMarkImage(imgIdx)`: `__GetImage(imgIdx)->Save()` çağırarak ilgili amblem resim dosyasını diske kaydeder.
        *   `__GetImage(imgIdx)`: `m_mapIdx_Image` haritasında `imgIdx`'e karşılık gelen `CGuildMarkImage` nesnesini arar. Yoksa, `__NewImage()` ile yeni bir tane oluşturur, `GetMarkImageFilename` ile dosya adını alır, `pkImage->Load()` ile yüklemeye çalışır. Yükleme başarısız olursa (dosya yoksa), `pkImage->Build()` ile boş bir resim dosyası oluşturur ve tekrar `pkImage->Load()` yapar. Oluşturulan/yüklenen nesneyi haritaya ekler ve döndürür.
        *   **Amblem ID Yönetimi:**
            *   `AddMarkIDByGuildID(guildID, markID)`: `m_mapGID_MarkID`'ye (`guildID`, `markID`) çiftini ekler ve `markID`'yi `m_setFreeMarkID`'den siler.
            *   `GetMarkID(guildID)`: `m_mapGID_MarkID`'den `guildID`'ye karşılık gelen `markID`'yi döndürür. Bulamazsa `INVALID_MARK_ID` döner.
            *   `__AllocMarkID(guildID)`: `m_setFreeMarkID`'den en küçük uygun boş `markID`'yi alır. Bu `markID`'nin ait olduğu resim dosyasını `__GetImage()` ile (gerekirse oluşturarak) alır. `AddMarkIDByGuildID` ile eşleşmeyi kaydeder ve `markID`'yi döndürür. Boş ID yoksa `INVALID_MARK_ID` döner.
        *   **Sunucu İşlevleri:**
            *   `CopyMarkIdx(pcBuf)`: `m_mapGID_MarkID`'deki tüm (`guildID`, `markID`) çiftlerini, `WORD` (2 byte) türünde `pcBuf` tamponuna yazar. Bu, genellikle istemciye amblem bilgilerini toplu göndermek için kullanılır.
            *   `SaveMark(guildID, pbMarkImage)`: Bir loncanın amblemini kaydeder. `GetMarkID` ile loncanın mevcut bir amblemi olup olmadığını kontrol eder. Yoksa `__AllocMarkID` ile yeni bir ID alır. İlgili `CGuildMarkImage` nesnesini `__GetImage` ile alır, `pkImage->SaveMark()` ile amblem verisini (`pbMarkImage`) resme yazar. Son olarak `SaveMarkImage` ve `SaveMarkIndex` ile değişiklikleri diske kaydeder.
            *   `DeleteMark(guildID)`: Loncanın amblemini siler. `m_mapGID_MarkID`'den kaydı bulur, ilgili `CGuildMarkImage`'ı alır ve `pkImage->DeleteMark()` çağırır. `markID`'yi `m_setFreeMarkID`'ye geri ekler ve `m_mapGID_MarkID`'den siler. `SaveMarkIndex` ile indeksi günceller.
            *   `GetDiffBlocks(imgIdx, crcList, mapDiffBlocks)`: `__GetImage(imgIdx)->GetDiffBlocks()` çağırarak istemci ve sunucu arasındaki farklı amblem bloklarını bulur.
        *   **İstemci İşlevleri (Sunucu tarafında da çağrılabilir):**
            *   `SaveBlockFromCompressedData(imgIdx, posBlock, pbBlock, dwSize)`: `__GetImage(imgIdx)->SaveBlockFromCompressedData()` çağırarak sıkıştırılmış bir blok verisini kaydeder.
            *   `GetBlockCRCList(imgIdx, crcList)`: `__GetImage(imgIdx)->GetBlockCRCList()` çağırarak bir amblem resminin tüm bloklarının CRC listesini alır.
        *   **Sembol Yönetimi (`GetGuildSymbol`, `LoadSymbol`, `SaveSymbol`, `UploadSymbol`):**
            *   `GetGuildSymbol(guildID)`: `m_mapSymbol`'dan ilgili loncanın sembolünü döndürür.
            *   `LoadSymbol(filename)`: Belirtilen dosyadan sembolleri yükler. Dosya formatı: [sembol sayısı (DWORD)], ardından her sembol için [guildID (DWORD)], [veri boyutu (DWORD)], [ham veri (BYTE dizisi)]. Yüklenen verinin CRC32'si hesaplanır ve `TGuildSymbol` içinde saklanır.
            *   `SaveSymbol(filename)`: `m_mapSymbol`'daki tüm sembolleri `LoadSymbol`'da belirtilen formatta dosyaya yazar.
            *   `UploadSymbol(guildID, iSize, pbyData)`: Bir lonca için yeni sembol verisini alır, `m_mapSymbol`'da günceller veya yeni giriş oluşturur ve CRC'sini hesaplar.
        *   **Test Kodu (`#ifdef __UNITTEST__`):** `main` fonksiyonu ve yardımcı fonksiyonlar (`heartbeat`, `SaveMark`) içerir. Bu bölüm, `CGuildMarkManager`'ın temel işlevlerini (amblem kaydetme, silme, CRC listesi alma, farkları bulma) test etmek için kullanılır. DevIL ve LZO kütüphanelerini ilklendirir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `MarkManager.h`, `crc32.h`, `lzo_manager.h` (test kodu için), `MarkImage.h` (test kodu için).

---

### `messenger_manager.h`

*   **Amaç:** Oyuncular arası arkadaşlık (messenger) sistemini yönetmek için `MessengerManager` singleton sınıfını tanımlar. Arkadaş listeleri, GM listesi (`__MESSENGER_GM__` tanımlıysa) ve engellenenler listesi (`__MESSENGER_BLOCK_SYSTEM__` tanımlıysa) gibi verileri yönetir. Oyuncu giriş/çıkışlarında, arkadaş ekleme/çıkarma işlemlerinde ve P2P (Peer-to-Peer) sunucular arası iletişimde rol oynar.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:**
        *   `keyT` (std::string), `keyA` (const std::string&): Hesap/oyuncu isimleri için kısaltmalar.
        *   `KeyTSet` (std::set<std::string>): Bir oyuncunun arkadaş/GM/engellenen listesindeki isimleri tutan set.
        *   `KeyTRelation` (std::map<std::string, KeyTSet>): Bir oyuncunun (anahtar) arkadaş/GM/engellenen listesini (değer) tutan harita.
        *   Benzer şekilde `KeyGSet`, `KeyGRelation` (GM için) ve `KeyBSet`, `KeyBRelation` (engellenenler için).
    *   **`MessengerManager` Sınıfı (Singleton):**
        *   **Public Metotlar:**
            *   `P2PLogin(const std::string& account)`, `P2PLogout(const std::string& account)`: P2P üzerinden gelen giriş/çıkış bilgilerini işler, `Login`/`Logout` çağırır.
            *   `Login(const std::string& account)`: Bir oyuncu (hesap) oyuna girdiğinde çağrılır. Oyuncunun arkadaş, GM ve engellenenler listesini DB'den yükler (`LoadList`, `LoadGMList`, `LoadBlockList`). Oyuncuyu `m_set_loginAccount`'a ekler.
            *   `Logout(const std::string& account)`: Oyuncu oyundan çıktığında çağrılır. Oyuncuyu `m_set_loginAccount`'tan siler. Bu oyuncuyu listesinde bulunduran diğer online oyunculara çıkış bilgisini gönderir (`SendLogout`, `SendGMLogout`, `SendBlockLogout`). İlişkili listelerden (`m_Relation`, `m_InverseRelation` vb.) kaldırır.
            *   `RequestToAdd(LPCHARACTER ch, LPCHARACTER target)`: `ch` karakterinin `target` karakterine arkadaşlık isteği göndermesini sağlar. Quest durumu kontrolü yapar. Başarılıysa `target`'a `messenger_auth` komutu gönderir.
            *   `AuthToAdd(const std::string& account, const std::string& companion, bool bDeny)`: Arkadaşlık isteğine verilen yanıtı işler. `bDeny` false ise her iki oyuncu için de `AddToList` çağırır.
            *   `AddToList(const std::string& account, const std::string& companion)`: `account`'ın listesine `companion`'ı ekler. DB'ye kaydeder, P2P ile diğer sunuculara bildirir ve `__AddToList` çağırır.
            *   `__AddToList(...)`: İlişkili listeleri (`m_Relation`, `m_InverseRelation`) günceller ve online ise ilgili oyunculara bilgi gönderir.
            *   `RemoveFromList(const std::string& account, const std::string& companion)`: `account`'ın listesinden `companion`'ı çıkarır. DB'den siler, P2P ile bildirir ve `__RemoveFromList` çağırır.
            *   `__RemoveFromList(...)`: İlişkili listeleri günceller ve online ise oyuncuya bilgi gönderir.
            *   `IsInList(...)`, `IsFriend(...)`: Bir oyuncunun diğerini arkadaş listesinde tutup tutmadığını kontrol eder.
            *   `RemoveAllList(const std::string& account)`: Bir oyuncunun tüm arkadaş listesini temizler.
            *   (`__MESSENGER_BLOCK_SYSTEM__` için benzer `AddToBlockList`, `RemoveFromBlockList`, `IsBlocked`, `RemoveAllBlockList` metotları)
            *   `Initialize()`, `Destroy()`: Yöneticinin başlatılması ve yok edilmesi.
        *   **Private Metotlar (Paket Gönderme ve Yükleme):**
            *   `SendList(const std::string& account)`: Oyuncuya arkadaş listesini (`MESSENGER_SUBHEADER_GC_LIST`) gönderir (online/offline durumlarıyla birlikte).
            *   `SendLogin(const std::string& account, const std::string& companion)`: `account`'a, `companion`'ın oyuna girdiğini bildirir (`MESSENGER_SUBHEADER_GC_LOGIN`).
            *   `SendLogout(const std::string& account, const std::string& companion)`: `account`'a, `companion`'ın oyundan çıktığını bildirir (`MESSENGER_SUBHEADER_GC_LOGOUT`).
            *   `LoadList(SQLMsg* pmsg)`: DB'den (`messenger_list` tablosu) gelen arkadaş listesi verilerini işler, `m_Relation` ve `m_InverseRelation`'ı doldurur, `SendList` ve `SendLogin` çağırır.
            *   (`__MESSENGER_GM__` için benzer `SendGMList`, `SendGMLogin`, `SendGMLogout`, `LoadGMList` metotları `common.gmlist` tablosunu kullanır).
            *   (`__MESSENGER_BLOCK_SYSTEM__` için benzer `SendBlockList`, `SendBlockLogin`, `SendBlockLogout`, `LoadBlockList` metotları `messenger_block_list` tablosunu kullanır).
        *   **Üye Değişkenler:**
            *   `m_set_loginAccount` (KeyTSet): Oyunda olan (login yapmış) hesapların listesi.
            *   `m_Relation` (KeyTRelation): `account` -> {`companion1`, `companion2`} şeklinde arkadaş ilişkilerini tutar.
            *   `m_InverseRelation` (KeyTRelation): `companion` -> {`account1`, `account2`} şeklinde ters arkadaş ilişkilerini tutar (birisi oyuna girdiğinde kimlere haber verileceğini bulmak için).
            *   `m_set_requestToAdd` (std::set<DWORD>): Aktif arkadaşlık isteklerini (CRC32(isim1:isim2) şeklinde) tutar.
            *   (`__MESSENGER_GM__` için `m_GMRelation`, `m_InverseGMRelation`).
            *   (`__MESSENGER_BLOCK_SYSTEM__` için `m_BlockRelation`, `m_InverseBlockRelation`, `m_set_requestToBlockAdd`).
*   **Bağlantılı Dosyalar:** `messenger_manager.cpp` (uygulama), `db.h` (`SQLMsg`), `stdafx.h`, `char.h` (`LPCHARACTER`).

---

### `messenger_manager.cpp`

*   **Amaç:** `messenger_manager.h` dosyasında bildirilen `MessengerManager` sınıfının metotlarını uygular. Arkadaşlık, GM ve engelleme listelerinin veritabanı işlemlerini (yükleme, ekleme, silme), oyuncu online/offline durumlarının takibini, istemciye ilgili paketlerin gönderilmesini ve P2P üzerinden diğer sunucularla senkronizasyonu yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`Login(account)`:**
        *   Oyuncu zaten `m_set_loginAccount`'ta varsa bir şey yapmaz.
        *   `DBManager::instance().FuncQuery` kullanarak asenkron olarak `messenger_list`, (varsa) `common.gmlist` ve (varsa) `messenger_block_list` tablolarından ilgili oyuncunun listelerini çekmek için sorgu gönderir. Sorgu sonuçları `LoadList`, `LoadGMList`, `LoadBlockList` callback fonksiyonlarına gider.
        *   Oyuncuyu `m_set_loginAccount`'a ekler.
    *   **`Logout(account)`:**
        *   Oyuncu `m_set_loginAccount`'ta yoksa bir şey yapmaz.
        *   `m_set_loginAccount`'tan siler.
        *   `m_InverseRelation[account]` (ve GM/Block için olanlar) üzerinden, çıkan oyuncuyu listesinde tutan tüm online oyunculara `SendLogout` (veya `SendGMLogout`, `SendBlockLogout`) ile çıkış yaptığını bildirir.
        *   `m_Relation` (ve GM/Block için olanlar) haritalarından çıkan oyuncuyu ve onun diğer oyunculardaki girişlerini siler.
    *   **`LoadList(msg)` (Callback):**
        *   DB'den gelen (`messenger_list` tablosu) sonuçları işler.
        *   Her satır için `m_Relation[account].insert(companion)` ve `m_InverseRelation[companion].insert(account)` ile ilişkileri kaydeder.
        *   Listenin tamamı yüklendikten sonra `SendList(account)` ile oyuncuya listeyi gönderir.
        *   `m_InverseRelation[account]` üzerinden, yeni giren oyuncuyu listesinde bulunduran ve online olan diğer oyunculara `SendLogin` ile giriş yaptığını bildirir.
    *   **`RequestToAdd(ch, target)`:**
        *   İstek yapan ve hedef karakterlerin PC olup olmadığını, quest (`IsRunning`) durumlarını kontrol eder.
        *   İsimlerden CRC32 hesaplayarak bir istek ID'si (`dwComplex`) oluşturur ve `m_set_requestToAdd`'e ekler.
        *   Hedef karaktere `messenger_auth [istek_yapanin_ismi]` komutunu göndererek arkadaşlık isteğini bildirir.
    *   **`AuthToAdd(account, companion, bDeny)`:**
        *   `companion` (istek yapan) ve `account` (yanıt veren) isimlerinden CRC32 ile istek ID'sini tekrar oluşturur.
        *   `m_set_requestToAdd`'de böyle bir istek olup olmadığını kontrol eder, yoksa hata loglar.
        *   İsteği `m_set_requestToAdd`'den siler.
        *   `bDeny` (reddetme) `false` ise, karşılıklı olarak `AddToList(companion, account)` ve `AddToList(account, companion)` çağırır.
    *   **`AddToList(account, companion)`:**
        *   `companion` boşsa veya zaten listede varsa bir şey yapmaz.
        *   DB'ye `INSERT INTO messenger_list` sorgusu gönderir.
        *   `__AddToList(account, companion)` çağırır.
        *   P2P üzerinden diğer sunuculara `HEADER_GG_MESSENGER_ADD` paketi göndererek değişikliği bildirir.
    *   **`__AddToList(account, companion)`:**
        *   `m_Relation[account].insert(companion)` ve `m_InverseRelation[companion].insert(account)` ile hafızadaki listeleri günceller.
        *   `account` karakteri online ise ona bilgi mesajı gönderir.
        *   `companion` karakteri online ise `account`'a `SendLogin` ile `companion`'ın online olduğunu bildirir, değilse `SendLogout` ile offline olduğunu bildirir.
    *   **`RemoveFromList(account, companion)`:**
        *   `companion` boşsa veya listede yoksa hata mesajı gönderir.
        *   DB'ye `DELETE FROM messenger_list` sorgusu gönderir.
        *   `__RemoveFromList(account, companion)` çağırır.
        *   P2P üzerinden diğer sunuculara `HEADER_GG_MESSENGER_REMOVE` paketi gönderir.
    *   **`__RemoveFromList(account, companion)`:**
        *   `m_Relation[account].erase(companion)` ve `m_InverseRelation[companion].erase(account)` ile hafızadaki listeleri günceller.
        *   `account` karakteri online ise ona bilgi mesajı gönderir.
    *   **`SendList(account)`:**
        *   `m_Relation[account]` listesindeki her bir arkadaş için `m_set_loginAccount`'ta olup olmadığını kontrol ederek online/offline durumunu belirler.
        *   `HEADER_GC_MESSENGER` ve `MESSENGER_SUBHEADER_GC_LIST` alt başlığı ile paket oluşturur. Her arkadaş için `TPacketGCMessengerListOnline` veya `TPacketGCMessengerListOffline` yapısını ve arkadaş ismini `TEMP_BUFFER`'a yazar. Sonra bu buffer'ı oyuncuya gönderir.
    *   **`SendLogin(account, companion)` ve `SendLogout(account, companion)`:**
        *   `HEADER_GC_MESSENGER` ve `MESSENGER_SUBHEADER_GC_LOGIN` veya `_LOGOUT` alt başlığı ile paket oluşturur, `companion` ismini ve uzunluğunu ekleyerek `account` oyuncusuna gönderir.
    *   GM listesi (`__MESSENGER_GM__`) ve Engellenenler listesi (`__MESSENGER_BLOCK_SYSTEM__`) için de benzer `Load*List`, `Add*List`, `Remove*List`, `Send*List`, `Send*Login`, `Send*Logout` fonksiyonları bulunur, farklı DB tabloları ve P2P/istemci paket alt başlıkları kullanılır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `gm.h`, `messenger_manager.h`, `buffer_manager.h`, `desc_client.h`, `log.h`, `config.h`, `p2p.h`, `crc32.h`, `char.h`, `char_manager.h`, `questmanager.h`.

---

### `minilzo.h`

*   **Amaç:** LZO (Lempel-Ziv-Oberhumer) gerçek zamanlı veri sıkıştırma kütüphanesinin mini bir alt kümesi için başlık dosyasıdır. LZO1X algoritması için sıkıştırma ve açma fonksiyonlarının bildirimlerini ve gerekli bellek boyutu (`LZO1X_MEM_COMPRESS`) gibi sabitleri tanımlar. `lzo/lzoconf.h` başlığını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Versiyon Tanımı:** `MINILZO_VERSION` (0x1080).
    *   **Bellek Gereksinimi Sabitleri:**
        *   `LZO1X_MEM_COMPRESS` (`LZO1X_1_MEM_COMPRESS`): `lzo1x_1_compress` fonksiyonu için gereken çalışma belleği (`wrkmem`) boyutunu tanımlar (genellikle `16384 * sizeof(lzo_dict_t)`).
        *   `LZO1X_MEM_DECOMPRESS`: Açma fonksiyonları için ek bellek gerekmediğini belirtir (0).
    *   **Fonksiyon Bildirimleri:**
        *   **`lzo1x_1_compress(...)`:** Verilen kaynak veriyi (`src`, `src_len`) LZO1X algoritması ile sıkıştırır ve sonucu `dst`'ye yazar. Sıkıştırılmış boyutu `dst_len` işaretçisi ile döndürür. Çalışma belleği olarak `wrkmem` kullanır.
        *   **`lzo1x_decompress(...)`:** LZO1X ile sıkıştırılmış veriyi (`src`, `src_len`) açar ve sonucu `dst`'ye yazar. Açılmış boyutu `dst_len` işaretçisi ile döndürür. `wrkmem` parametresi kullanılır (ancak bu versiyonda gerekli değil).
        *   **`lzo1x_decompress_safe(...)`:** `lzo1x_decompress` ile aynı işlevi yapar ancak girdi/çıktı tampon taşmalarına ve geri bakış (`lookbehind`) hatalarına karşı ek kontroller içerir. Daha güvenlidir ancak biraz daha yavaş olabilir.
*   **Bağlantılı Dosyalar:** `minilzo.c` (uygulama), `lzo/lzoconf.h` (yapılandırma ve tür tanımları).

---

### `minilzo.c`

*   **Amaç:** `minilzo.h`'de bildirilen LZO1X sıkıştırma ve açma fonksiyonlarının implementasyonunu içerir. Bu, LZO kütüphanesinin küçük, bağımlılığı az bir alt kümesidir ve genellikle gömülü sistemlerde veya tam LZO kütüphanesinin gereksiz olduğu durumlarda kullanılır. Metin2 sunucusunda, muhtemelen `CLZOManager` tarafından veri sıkıştırma için kullanılır.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapılandırma ve Kontroller (`#include`, `#define`, `static lzo_bool` fonksiyonlar, `_lzo_config_check`):**
        *   Gerekli başlık dosyalarını (`minilzo.h`, `stdio.h`, `stdlib.h`, `string.h`, `assert.h` vb.) içerir.
        *   Derleme zamanı ve çalışma zamanı için çeşitli makrolar ve sabitler tanımlar (örn: `M1/M2/M3/M4_MAX_OFFSET`, `D_BITS`, `LZO_HASH` stratejisi, `COPY4` gibi optimizasyon makroları).
        *   Temel veri türlerinin boyutları, işaretçi hizalamaları, sistem byte sırası (endianness) gibi konfigürasyonları kontrol eden `_lzo_config_check` fonksiyonunu ve yardımcılarını içerir.
        *   LZO kütüphanesinin telif hakkı bilgisini (`__lzo_copyright`) ve versiyon fonksiyonlarını (`lzo_version`, `lzo_version_string` vb.) içerir.
        *   Adler32 checksum hesaplaması için `lzo_adler32` fonksiyonunu (ve LZO_DO* makrolarını) içerir.
        *   `memcmp`, `memcpy`, `memmove`, `memset` gibi standart C fonksiyonlarının LZO içindeki eşdeğerlerini (`lzo_memcmp` vb.) sağlar (bazı platformlarda veya derleyici ayarlarında doğrudan sistem fonksiyonları kullanılır).
    *   **Sıkıştırma (`lzo1x_1_compress`, `_lzo1x_1_do_compress`):**
        *   `_lzo1x_1_do_compress` ana sıkıştırma mantığını içerir.
        *   Sözlük (`wrkmem`) kullanarak tekrarlayan veri bloklarını bulur.
        *   Veriyi literal (olduğu gibi kopyalanan) ve eşleşme (önceki veriye referans) blokları olarak kodlar.
        *   Farklı uzunluk ve ofsetlere sahip eşleşmeler için M1, M2, M3, M4 olarak adlandırılan farklı kodlama yöntemleri kullanır.
        *   Hashing stratejisi (`LZO_HASH_LZO_INCREMENTAL_B` gibi) kullanılarak potansiyel eşleşmeler hızlıca bulunur (`DINDEX1`, `DINDEX2`, `GINDEX`).
        *   Sözlük güncellenir (`UPDATE_I`).
        *   Sıkıştırılmış veriyi çıktı tamponuna (`out`) yazar.
        *   `lzo1x_1_compress` ise `_lzo1x_1_do_compress`'i çağırır ve sıkıştırılamayan son veriyi (varsa) ve EOF (End Of File) işaretçisini çıktıya ekler.
    *   **Açma (`lzo1x_decompress`, `lzo1x_decompress_safe`):**
        *   Her iki fonksiyon da benzer bir temel mantık kullanır.
        *   Girdi tamponundaki (`in`) kontrol baytlarını okuyarak literal veya eşleşme bloğu olup olmadığını anlar.
        *   **Literal Blok:** Belirtilen uzunluktaki veriyi doğrudan girdiden çıktıya kopyalar.
        *   **Eşleşme Bloğu:** Kontrol baytlarından eşleşme uzunluğunu (`t`) ve ofsetini (`m_off`) çıkarır. Çıktı tamponunda `op - m_off` konumundaki veriyi (`m_pos`) bulur ve `t` bayt kadarını mevcut çıktı konumuna kopyalar. Farklı M1/M2/M3/M4 kodlamalarına göre ofset ve uzunluk farklı şekillerde hesaplanır.
        *   `lzo1x_decompress_safe`, girdi tamponu sonuna (`ip_end`) ve çıktı tamponu sonuna (`op_end`) ulaşıp ulaşılmadığını ve geri bakış pozisyonunun (`m_pos`) geçerli olup olmadığını (`TEST_IP`, `NEED_IP`, `TEST_OP`, `NEED_OP`, `TEST_LOOKBEHIND`) her adımda kontrol ederek taşma hatalarını önler. `lzo1x_decompress` bu kontrollerin bazılarını içermeyebilir (daha hızlı ama potansiyel olarak güvensiz).
        *   EOF işaretçisi (M4_MARKER | 1) bulunduğunda işlemi bitirir.
*   **Bağlantılı Dosyalar:** `minilzo.h` (bildirimler), `lzo/lzoconf.h` (yapılandırma), çeşitli standart C kütüphaneleri.

---

### `object_allocator.h`

*   **Amaç:** Sık kullanılan nesneler için optimize edilmiş bir bellek ayırıcı (allocator) mekanizması sağlar. Sık sık `new` ve `delete` yapılan nesneler için bellek ayırma/serbest bırakma maliyetini azaltmayı hedefler. Serbest bırakılan bellek bloklarını hemen sisteme iade etmek yerine bir "free list" içinde tutar ve yeni isteklerde öncelikle bu listeden verir. Belirli bir eşik (`FREE_TRIGGER`) aşıldığında listeyi temizler. Ayrıca, hata ayıklama (`DEBUG_ALLOC` tanımlıysa) modunda bellek sızıntılarını ve hatalarını tespit etmeye yardımcı olacak ek bilgiler (dosya adı, satır numarası, "yaş") ekler.
*   **Temel İşlevler/İçerik:**
    *   **`FreeList` Typedef'i (`std::deque<void*>`):** Serbest bırakılmış bellek bloklarının işaretçilerini tutmak için kullanılan veri yapısı.
    *   **`LateAllocator` Şablon Sınıfı:**
        *   Belirli bir nesne türü (`OBJ`) ve serbest bırakma tetikleyici sayısı (`FREE_TRIGGER`) için temel geç serbest bırakma mantığını içerir.
        *   `Alloc(size)`: Bellek ayırır. Önce `m_freeBlocks` listesini kontrol eder, boş değilse oradan bir blok alır. Boşsa, standart `::malloc` ile yeni bellek ayırır. (`DEBUG_ALLOC` aktifse, ek bir `size_t` boyutunda başlık alanı ayırır).
        *   `Free(p)`: Belleği serbest bırakır. Eğer `m_freeBlockCount` (serbest listedeki blok sayısı) `FREE_TRIGGER`'dan küçükse, bloğu `m_freeBlocks` listesine ekler. Eğer eşit veya büyükse, standart `::free` ile bloğu doğrudan sisteme iade eder. (`DEBUG_ALLOC` aktifse, serbest bırakmadan önce belleği sıfırlar ve başlık alanını da dikkate alarak `::free` çağırır).
        *   `GetFreeBlockCount()`: Serbest listesindeki blok sayısını döndürür.
        *   **Statik Üyeler:** `m_freeBlockCount`, `m_freeBlocks` (her şablon örneği için ayrı).
    *   **`ObjectAllocator` Şablon Sınıfı:**
        *   Belirli bir nesne türü (`OBJ`) için `new` ve `delete` operatörlerini aşırı yükleyerek (`operator new`, `operator delete`) `LateAllocator`'ı kullanır.
        *   Bu sınıftan türetilen sınıflar, otomatik olarak optimize edilmiş bellek ayırma/serbest bırakma mekanizmasından faydalanır.
        *   `new` operatörü `LateAllocator::Alloc`'u çağırır. (`DEBUG_ALLOC` aktifse, ek olarak `DebugAllocator::MarkAcquired` ile hata ayıklama bilgisi kaydeder).
        *   `delete` operatörü `LateAllocator::Free`'yi çağırır. (`DEBUG_ALLOC` aktifse, ek olarak `AllocTag::IncreaseAge` ile bloğun "yaşını" artırır).
        *   `GetFreeBlockCount()`: `LateAllocator`'daki sayacı döndürür.
        *   **Statik Üye:** `m_allocator` (`LateAllocator` örneği).
    *   **Makrolar:**
        *   `M2_OBJ_NEW`: `DEBUG_ALLOC` tanımlıysa `new(__FILE__, __LINE__)` (dosya/satır bilgisiyle `new`), değilse standart `new` olarak tanımlanır. `ObjectAllocator` türemiş sınıflar için bu makro kullanılmalıdır.
        *   `M2_OBJ_DELETE`, `M2_OBJ_DELETE_EX`: `DEBUG_ALLOC` tanımlıysa `delete` işlemiyle birlikte `DebugAllocator::MarkReleased` çağırarak hata ayıklama bilgisi kaydeder, değilse standart `delete` olarak tanımlanır.
*   **Kullanım:** Sıkça oluşturulup yok edilen nesne sınıfları (örn. karakterler, eşyalar, olay bilgileri) `ObjectAllocator`'dan türetilerek bellek yönetimi performansı artırılabilir. Örneğin: `class CMyObject : public ObjectAllocator<CMyObject> { ... };`. Nesne oluştururken `M2_OBJ_NEW CMyObject` ve silerken `M2_OBJ_DELETE(pMyObject)` makroları kullanılır.
*   **Bağlantılı Dosyalar:** `debug_allocator.h` (hata ayıklama için), `<deque>`, `<assert.h>`. `::malloc` ve `::free` için `stdlib.h` veya benzeri bir başlık (genellikle `stdafx.h` üzerinden).

---

### `panama.h`

*   **Amaç:** Panama sistemiyle ilgili fonksiyon bildirimlerini içerir. Bu sistem, muhtemelen istemci tarafındaki paket (*.epk, *.eix) dosyalarının bütünlüğünü doğrulamak veya belirli paketler için şifreleme/doğrulama anahtarları (IV - Initialization Vector) sağlamak için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   `PanamaLoad()`: Panama sistemi için gerekli verileri (genellikle `panama/panama.lst` dosyasından paket adları ve ilgili IV dosyalarının yollarını okuyarak) yükleyen fonksiyonun bildirimi. Yüklenen paket/IV çifti sayısını döndürür.
    *   `SendPanamaList(LPDESC d)`: Yüklenmiş olan Panama paket adı ve IV listesini, istemciye özel olarak XOR'lanmış şekilde (`TPacketGCPanamaPack` kullanarak) belirli bir istemci bağlantısına (`LPDESC d`) gönderen fonksiyonun bildirimi.
*   **Bağlantılı Dosyalar:** `panama.cpp` (uygulama).

---

### `panama.cpp`

*   **Amaç:** `panama.h`'de bildirilen `PanamaLoad` ve `SendPanamaList` fonksiyonlarını uygular. `panama.lst` dosyasını okuyarak paket adlarını ve ilgili IV dosyalarını yükler, IV verilerini belleğe yükler ve istemciye gönderir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Statik Değişken:**
        *   `s_panamaVector` (`PanamaVectorType` - `std::vector<std::pair<std::string, BYTE*>>`): Paket adını (`std::string`) ve 32 byte'lık IV verisinin işaretçisini (`BYTE*`) tutan çiftlerin vektörü.
    *   **`PanamaLoad()`:**
        *   `panama/panama.lst` dosyasını okuma modunda açar.
        *   Dosyayı satır satır okur (`fgets`). Her satırdan paket adını (`szPackName`) ve IV dosya adını (`szIVFileNameConfig`) `sscanf` ile ayrıştırır.
        *   IV dosyasının tam yolunu (`panama/[IV dosya adı]`) oluşturur.
        *   IV dosyasını ikili okuma (`"rb"`) modunda açar.
        *   32 byte'lık IV verisini dosyadan okur (`fread`) ve `abIV` dizisine atar. Okuma başarısız olursa hata loglar.
        *   Başarılı olursa, okunan IV verisini hex formatında loglar (`sys_log`).
        *   `s_panamaVector`'e yeni bir çift ekler: paket adı ve yeni ayrılmış 32 byte'lık bellek alanı (`M2_NEW BYTE[32]`).
        *   Okunan `abIV` verisini bu yeni ayrılmış bellek alanına kopyalar (`memcpy`).
        *   Tüm satırlar işlendikten sonra dosyaları kapatır ve yüklenen çift sayısını (`s_panamaVector.size()`) döndürür.
    *   **`SendPanamaList(LPDESC d)`:**
        *   Bir `TPacketGCPanamaPack` paketi oluşturur ve başlığını `HEADER_GC_PANAMA_PACK` olarak ayarlar.
        *   `s_panamaVector` üzerinde döner:
            *   Her bir çift için paket adını (`it->first`) ve IV verisini (`it->second`) `pack` yapısına kopyalar.
            *   IV verisini (`pack.abIV`) 32 bitlik DWORD dizisi olarak ele alır.
            *   Her DWORD değeri için, istemcinin `d->GetPanamaKey()` ile alınan özel anahtarı ve döngü indeksi ile oluşturulan basit bir değerle XOR'lar (`ivs[i] ^= d->GetPanamaKey() + i * 16777619`). Bu, her istemciye farklı bir IV seti gönderilmesini sağlar.
            *   Değiştirilmiş paketi `d->Packet()` ile istemciye gönderir.
*   **Çalışma Prensibi:** Sunucu başlangıcında `PanamaLoad` çağrılarak `panama.lst`'de listelenen tüm paket adları ve ilgili IV dosyalarından okunan 32 byte'lık IV verileri belleğe yüklenir. Bir istemci bağlandığında ve muhtemelen belirli bir aşamaya (örn. handshake sonrası) geldiğinde, `SendPanamaList` çağrılır. Bu fonksiyon, bellekteki her paket/IV çifti için IV verisini istemcinin Panama anahtarıyla XOR'layarak kişiselleştirir ve `HEADER_GC_PANAMA_PACK` paketiyle istemciye gönderir. İşlem sonunda eski dosyalar yeniden adlandırılarak tekrar kullanılmaları engellenir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `desc.h`, `packet.h`.

---

### `pool.h`

*   **Amaç:** Sıkça oluşturulan ve yok edilen nesneler veya belirli boyutlardaki ham bellek blokları için optimize edilmiş bellek havuzu (memory/object pool) mekanizmaları sağlar. Standart `new`/`delete` veya `malloc`/`free` çağrılarının performans maliyetini azaltmayı hedefler.
*   **Temel İşlevler/İçerik:** (`#ifdef M2_USE_POOL` ile çevrelenmiştir)
    *   **`PoolNode<T>` Struct:** Havuzdaki serbest blokları zincirleme listeyle tutmak için kullanılan düğüm yapısı.
    *   **`PoolAllocation<T>` Struct:** Havuz tarafından ayrılmış büyük bir bellek bloğunu (`chunk`), içindeki küçük blok sayısını (`num_blocks`) ve bu bloklara karşılık gelen `PoolNode` dizisini (`nodes`) tutar.
    *   **`PoolDetail<T>` Struct:** Bellek ayırma (`Alloc`) ve serbest bırakma (`Free`) işlemleri için tür bazında özelleştirme sağlar.
        *   Genel `T` türleri için `new T[]` ve `delete[]` kullanır.
        *   `void` türü için özelleştirilmiş versiyonu `::malloc` ve `::free` kullanır (ham bellek blokları için).
    *   **`ArrayPool<T>` Şablon Sınıfı:**
        *   Belirli bir boyuttaki (`array_size`) diziler (veya ham bellek blokları) için genel amaçlı, sadece büyüyebilen bir bellek havuzu.
        *   `Acquire()`: Havuzdan kullanılabilir bir blok alır. Eğer serbest blok yoksa, havuzu `Stretch` ile genişletmeye çalışır.
        *   `Release(T* p)`: Kullanılan bir bloğu havuza geri verir. Blok, `PoolNode` aracılığıyla serbest listesine eklenir.
        *   `Reserve(size_t n)`: Havuzun kapasitesini en az `n` blok içerecek şekilde ayarlar.
        *   `CleanUp()`: Havuz tarafından ayrılmış tüm büyük bellek bloklarını (`allocated_`) ve `PoolNode` dizilerini serbest bırakır.
        *   `Stretch(size_t increment)`: Havuza yeni bir büyük bellek bloğu (`chunk`) ekler, bu bloğu `array_size` boyutunda küçük bloklara böler ve her biri için `PoolNode` oluşturarak serbest listesine (`free_`) ekler.
    *   **`Pool` Typedef'i (`ArrayPool<void>`):** Ham (türsüz) bellek blokları için `ArrayPool`'un özel bir takma adı.
    *   **`MemoryPool` Sınıfı:**
        *   Farklı boyutlardaki ham bellek blokları için birden fazla `Pool` (yani `ArrayPool<void>`) nesnesini yönetir.
        *   `Acquire(size_t size)`: Belirtilen boyuta uygun bir `Pool` bulur (yoksa oluşturur) ve oradan bir bellek bloğu alır.
        *   `Release(void* p, size_t size)`: Bellek bloğunu, ait olduğu boyuttaki `Pool`'a geri verir.
    *   **`ObjectPool<T>` Şablon Sınıfı:**
        *   Belirli bir `T` türündeki nesneler için basit bir nesne havuzu. `T` türünün varsayılan kurucu (constructor) sağlaması gerekir.
        *   Dahili olarak `Pool` (yani `ArrayPool<void>`) kullanır.
        *   `Construct()`: Havuzdan (`pool_.Acquire()`) bir ham bellek bloğu alır ve bu blok üzerinde `placement new` kullanarak yeni bir `T` nesnesi oluşturur.
        *   `Destroy(T* p)`: Nesnenin yıkıcısını (`p->~T()`) manuel olarak çağırır ve ardından ham bellek bloğunu `pool_.Release(p)` ile havuza geri verir.
        *   `Reserve(size_t n)`: Dahili `Pool`'un kapasitesini ayarlar.
*   **Kullanım:** Bu yapılar, sıkça aynı türde nesne veya aynı boyutta bellek bloğu ayıran/serbest bırakan sistemlerde (örn: ağ paketleri, karakter/eşya nesneleri) bellek parçalanmasını (fragmentation) azaltmak ve ayırma/serbest bırakma işlemlerini hızlandırmak için kullanılır.
*   **Not:** Kod içindeki yorumlar, bu havuz mekanizmasının thread-safe olmadığını belirtir.
*   **Bağlantılı Dosyalar:** `<cstddef>`, `<cassert>`, `<boost/unordered_map.hpp>` (veya `<unordered_map>`), `stdlib.h` (malloc/free için). `DEBUG_ALLOC` tanımlıysa `debug_allocator.h` da dahil edilebilir.

---

### `profiler.h`

*   **Amaç:** Kodun çeşitli bölümlerinin çalışma süresini ölçmek ve analiz etmek için basit bir performans profilleme sistemi tanımlar. Bu sistem, belirli kod bloklarının ne kadar sürdüğünü veya belirli fonksiyonların toplamda ne kadar süre harcadığını ve kaç kez çağrıldığını takip etmek için kullanılabilir.
*   **Temel İşlevler/İçerik:**
    *   **`CProfiler` Sınıfı (Singleton):**
        *   Genel profilleyici yönetimini yapar.
        *   **`SProfileStackData` Struct:** Tek bir `Push`/`Pop` çağrısı arasındaki süreyi ölçmek için kullanılır (çağrı sırası, başlangıç/bitiş zamanı, isim).
        *   **`SProfileAccumData` Struct:** Bir kod bloğunun toplam çalışma süresini (`iCollapsedTime`), kaç kez çağrıldığını (`iCallingCount`) ve çağrı derinliğini (`iDepth`) biriktirerek ölçmek için kullanılır.
        *   `TProfileAccumDataMap` (boost::unordered_map<std::string, TProfileAccumData>): İsimlere göre birikmiş profil verilerini tutar.
        *   `Initialize()`: Profilleyiciyi sıfırlar (sayaçları ve derinliği).
        *   `Clear()`: Birikmiş verileri (`iCallingCount`, `iCollapsedTime`) sıfırlar, ancak isimleri ve derinlikleri korur.
        *   `Push(const char* c_szName)`, `Pop(const char* c_szName)`: `SProfileStackData` kullanarak belirli bir kod bloğunun anlık süresini ölçmek için kullanılır (yığın tabanlı).
        *   `PushAccum(const char* c_szName)`, `PopAccum(const char* c_szName)`: `SProfileAccumData` kullanarak belirli bir isimle etiketlenmiş kod bloklarının toplam süresini ve çağrı sayısını biriktirir.
        *   `Log(const char* c_pszFileName)`, `Print(FILE* fp)`: Toplanan profil verilerini (hem yığın hem de birikmiş) bir dosyaya veya standart çıktıya (stderr) yazdırır.
        *   `PrintOneStackData(const char* c_szName)`, `PrintOneAccumData(const char* c_szName)`: Belirli bir isim için profil verisini yazdırır.
        *   **Özel Üyeler:** Profil yığın verilerini (`m_ProfileStackDatas`), birikmiş verileri (`m_ProfileAccumDataMap`, `m_vec_Accum`) ve mevcut derinliği/çağrı adımını tutar.
    *   **`CProfileUnit<T>` Şablon Sınıfı:**
        *   RAII (Resource Acquisition Is Initialization) prensibini kullanarak profillemeyi kolaylaştırır.
        *   Yapıcı (`CProfileUnit(name)`): `CProfiler::instance().PushAccum(name)` çağırır.
        *   Yıkıcı (`~CProfileUnit()`): Kapsam (scope) sonlandığında otomatik olarak `PopAccum()` çağırır.
        *   `Pop()`: Profillemeyi manuel olarak durdurmak için kullanılır.
    *   **`PROF_UNIT` Makrosu:** `CProfileUnit<void>` için bir kısaltmadır. Bir kod bloğunun başına `PROF_UNIT profile_unit_name("Profile Label");` eklenerek kolayca o bloğun profilini çıkarmak için kullanılır.
*   **Bağlantılı Dosyalar:** `<boost/unordered_map.hpp>`. (Uygulama dosyası yoktur, tamamen başlık dosyası içindedir).

---

### `protocol.h`

*   **Amaç:** Ağ paketlerinde kullanılan temel veri türlerinin kodlanması (encoding) ve kodunun çözülmesi (decoding) için inline fonksiyonlar ve bir paket kodlama makrosu tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Inline Encoding Fonksiyonları:**
        *   `encode_byte(char ind)`: Tek bir byte'ı (`char`) ağ üzerinden gönderilebilecek `const char*` formatına dönüştürür (statik bir `char` dizisi kullanarak).
        *   `encode_2bytes(sh_int ind)`: İki byte'lık bir değeri (`short int` veya `WORD`) `const char*` formatına dönüştürür.
        *   `encode_4bytes(int ind)`: Dört byte'lık bir değeri (`int` veya `DWORD`) `const char*` formatına dönüştürür.
        *   **Not:** Bu fonksiyonlar thread-safe değildir, çünkü statik bir dizi kullanırlar. Sadece geçici dönüşüm için tasarlanmışlardır.
    *   **Inline Decoding Fonksiyonları:**
        *   `decode_byte(const void* a)`: Bir bellek adresinden (`void*`) tek bir `BYTE` okur.
        *   `decode_2bytes(const void* a)`: Bir bellek adresinden iki byte'lık bir `WORD` okur.
        *   `decode_4bytes(const void* a)`: Bir bellek adresinden dört byte'lık bir `INT` (veya `DWORD`) okur.
    *   **`packet_encode` Makrosu:**
        *   `#define packet_encode(buf, data, len) __packet_encode(buf, data, len, __FILE__, __LINE__)`
        *   Verilen veriyi (`data`, `len` boyutunda) belirtilen ağ tamponuna (`buf`, `LPBUFFER`) yazmak için kullanılır. Asıl işi `__packet_encode` fonksiyonu yapar.
    *   **`DEFAULT_PACKET_BUFFER_SIZE` Makrosu:**
        *   Varsayılan ağ paketi tampon boyutunu tanımlar (genellikle 65536 byte).
    *   **`__packet_encode` Inline Fonksiyonu:**
        *   `packet_encode` makrosunun gerçek uygulamasını içerir.
        *   Tamponda (`pbuf`) yeterli yer olup olmadığını kontrol eder (`buffer_has_space`).
        *   Yeterli yer varsa, veriyi (`data`) tampona yazar (`buffer_write`).
        *   Hata durumunda (yeterli yer yoksa) `false` döner (ancak hata mesajı yorum satırına alınmış).
*   **Bağlantılı Dosyalar:** (Genellikle `buffer.h` veya benzeri bir tampon yönetim başlığına ve temel tür tanımlarına (`common/length.h` gibi) ihtiyaç duyar. `assert.h` kullanılır). (Uygulama dosyası yoktur, tamamen başlık dosyası içindedir).

---

### `spam.h`

**Amacı:** Bu başlık dosyası, oyundaki sohbet mesajlarında spam (istenmeyen veya yasaklı içerik) tespiti yapmak için basit bir mekanizma sağlayan `SpamManager` singleton sınıfını tanımlar.

**Temel Bileşenler:**

*   **`SpamManager` Sınıfı (Singleton):**
    *   **Amacı:** Yasaklı kelimelerin bir listesini ve her kelime için bir "spam puanı" tutar. Verilen bir metin içinde bu kelimelerin olup olmadığını kontrol eder ve toplam spam puanını hesaplar.
    *   **Metotlar:**
        *   `GetSpamScore(const char* src, size_t len, unsigned int& score) const`: Verilen metin (`src`) içinde kayıtlı spam kelimelerini arar.
            *   Metindeki boşlukları kaldırır.
            *   Kayıtlı her spam kelimesi için `WildCaseCmp` (joker karakter ve büyük/küçük harf duyarsız karşılaştırma) kullanarak eşleşme arar.
            *   Eşleşme bulunursa, bulunan kelimenin puanını toplam `score`'a ekler ve bulunan ilk spam kelimesini (`word`) döndürür.
            *   Hiçbir spam kelimesi bulunamazsa `NULL` döndürür ve `score` 0 olur.
        *   `Clear()`: Kayıtlı tüm spam kelimelerini temizler (`m_vec_word.clear()`).
        *   `Insert(const char* str, unsigned int score = 10)`: Yeni bir spam kelimesini (`str`) ve isteğe bağlı bir puanı (`score`, varsayılan 10) listeye ekler. Eklenen kelimeyi loglar.
    *   **Özel Üyeler:**
        *   `m_vec_word` (`std::vector<std::pair<std::string, unsigned int>>`): Spam kelimelerini ve karşılık gelen puanlarını tutan bir vektör.

**Bağlantılı Dosyalar:** `utils.h` (muhtemelen `WildCaseCmp` fonksiyonu için), `<string>`, `<vector>`, `<utility>`, `<algorithm>` (std::remove_if için), `<cctype>` (isspace için). `spam.cpp` dosyası bulunmamaktadır, tüm implementasyon başlık dosyasındadır.

---

### `stable_priority_queue.h`

*   **Amaç:** Standart C++ öncelik kuyruğuna benzer bir arayüze sahip, ancak aynı önceliğe sahip elemanların göreceli sıralarını koruyan (kararlı sıralama) bir öncelik kuyruğu şablon sınıfı tanımlar. Bu, `std::list` gibi bir konteyner ve özel bir karşılaştırıcı (`Compare`) kullanılarak gerçekleştirilir.
*   **Temel İşlevler/İçerik:**
    *   **Şablon Parametreleri:**
        *   `T`: Kuyrukta saklanacak eleman türü.
        *   `Container`: Elemanları saklamak için kullanılacak temel konteyner türü (varsayılan `std::list<T>`).
        *   `Compare`: İki elemanı karşılaştırmak için kullanılacak fonksiyon nesnesi (varsayılan `std::less<typename Container::value_type>`). Bu karşılaştırıcı, kuyruğun "en yüksek" öncelikli elemanını belirler (genellikle `std::less` ile en büyük eleman).
    *   **Typedef\'ler:**
        *   `value_type`: Saklanan elemanın türü (`Container::value_type`).
        *   `size_type`: Boyut türü (`Container::size_type`).
        *   `container_type`: Kullanılan konteyner türü.
    *   **Korumalı (Protected) Üyeler:**
        *   `c` (`Container`): Elemanları tutan asıl konteyner.
        *   `comp` (`Compare`): Elemanları karşılaştırmak için kullanılan karşılaştırıcı nesnesi.
    *   **Özel (Private) Metot:**
        *   `push(InputIterator first, InputIterator end)`: Bir iteratör aralığındaki tüm elemanları kuyruğa ekler. Bu metot `explicit stable_priority_queue` kurucusunda kullanılır.
    *   **Genel (Public) Metotlar:**
        *   **`explicit stable_priority_queue(const Compare& cp = Compare(), const Container& cc = Container())`:** Kurucu. Opsiyonel bir karşılaştırıcı ve başlangıç konteyneri alabilir. Başlangıç konteynerindeki elemanları kuyruğa ekler.
        *   **`empty() const`:** Kuyruk boşsa `true` döndürür.
        *   **`size() const`:** Kuyruktaki eleman sayısını döndürür.
        *   **`top() const`:** En yüksek öncelikli elemana bir referans döndürür. Kararlı öncelik kuyruğunda, `std::list` ve `std::lower_bound` kullanıldığı için bu genellikle listenin son elemanıdır (`c.back()`).
        *   **`pop()`:** En yüksek öncelikli elemanı kuyruktan çıkarır (`c.pop_back()`).
        *   **`push(const value_type& x)`:** Yeni bir elemanı (`x`) kuyruğa ekler. Eleman, `std::lower_bound` kullanılarak karşılaştırıcıya (`comp`) göre doğru sıralı pozisyona eklenir. Bu, aynı öncelikli elemanların eklenme sıralarını korur (kararlılık).
*   **Kullanım Senaryoları:** Aynı önceliğe sahip görevlerin veya olayların işlenme sırasının önemli olduğu durumlarda kullanılabilir. Örneğin, bir görev zamanlayıcısında, aynı aciliyet seviyesine sahip görevlerin ilk gelen ilk işlenir (FIFO) mantığıyla ele alınması gerektiğinde.
*   **Bağlantılı Dosyalar:** `<list>` (varsayılan konteyner için), `<functional>` (varsayılan `std::less` için), `<algorithm>` (`std::lower_bound` için). (Tamamen başlık dosyasıdır, ayrı bir `.cpp` dosyası yoktur).

---

### `state.h`

*   **Amaç:** Oyun içindeki varlıklar veya sistemler için bir Durum Makinesi (Finite State Machine - FSM) deseni uygulamak amacıyla temel sınıfları tanımlar. Bu, bir nesnenin farklı davranış modları (durumları) arasında geçiş yapmasını ve her durumda belirli eylemleri gerçekleştirmesini sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`CState` Sınıfı (Soyut Temel Sınıf):**
        *   **Amacı:** Bir durumun arayüzünü tanımlar.
        *   **Sanal Yıkıcı (`virtual ~CState()`):** Türetilmiş sınıfların doğru şekilde yok edilmesini sağlar.
        *   **Saf Sanal Metotlar (Pure Virtual Functions):**
            *   `ExecuteBeginState()`: Bir duruma ilk kez girildiğinde çağrılacak mantığı içerir.
            *   `ExecuteState()`: Durum aktif olduğu sürece periyodik olarak (veya bir olayla tetiklenerek) çağrılacak ana mantığı içerir.
            *   `ExecuteEndState()`: Durumdan çıkılırken çağrılacak temizleme veya sonlandırma mantığını içerir.
    *   **`CStateTemplate<T>` Şablon Sınıfı (`CState`'den Türetilmiş):**
        *   **Amacı:** `CState` arayüzünün genel (generic) bir uygulamasını sağlar. `T` şablon parametresi, durum makinesine sahip olan nesnenin (sahip sınıf) türünü temsil eder.
        *   **Typedef:**
            *   `PFNSTATE`: Sahip `T` sınıfının üye fonksiyonlarına işaret eden bir tür (`void (T::*) (void)`).
        *   **Korumalı (Protected) Üyeler:**
            *   `m_pInstance` (`T*`): Durum makinesinin sahibi olan nesnenin bir işaretçisi.
            *   `m_pfnBeginState` (`PFNSTATE`): Durumun başlangıç fonksiyonuna (sahip sınıfın bir metodu) işaret eder.
            *   `m_pfnState` (`PFNSTATE`): Durumun ana yürütme fonksiyonuna işaret eder.
            *   `m_pfnEndState` (`PFNSTATE`): Durumun bitiş fonksiyonuna işaret eder.
        *   **Yapıcı (`CStateTemplate()`):** Tüm işaretçileri `0` (null) olarak başlatır.
        *   **`Set(T* pInstance, PFNSTATE pfnBeginState, PFNSTATE pfnState, PFNSTATE pfnEndState)` Metodu:**
            *   Durum nesnesini başlatır. Sahip nesne işaretçisini (`m_pInstance`) ve üç durum fonksiyonu işaretçisini (`m_pfnBeginState`, `m_pfnState`, `m_pfnEndState`) ayarlar. `assert` ile işaretçilerin geçerli olup olmadığını kontrol eder.
        *   **Sanal Metotların Geçersiz Kılınması (Override):**
            *   `ExecuteBeginState()`: `m_pInstance` üzerinden `m_pfnBeginState` ile işaret edilen fonksiyonu çağırır.
            *   `ExecuteState()`: `m_pInstance` üzerinden `m_pfnState` ile işaret edilen fonksiyonu çağırır.
            *   `ExecuteEndState()`: `m_pInstance` üzerinden `m_pfnEndState` ile işaret edilen fonksiyonu çağırır.
            *   Tüm bu metotlar, çağrı öncesinde `m_pInstance` ve ilgili fonksiyon işaretçisinin geçerli olup olmadığını `assert` ile kontrol eder.
*   **Kullanım Şekli (Örnek):**
    ```cpp
    // class CMyEntity {
    // public:
    //     void OnIdle_Begin();
    //     void OnIdle_State();
    //     void OnIdle_End();
    //     // ... other states ...
    // };
    // CStateTemplate<CMyEntity> idleState;
    // idleState.Set(this, &CMyEntity::OnIdle_Begin, &CMyEntity::OnIdle_State, &CMyEntity::OnIdle_End);
    ```
*   **Bağlantılı Dosyalar:** `<cassert>` (`assert` makrosu için). Genellikle durum makinelerini yöneten daha üst seviye bir FSM yöneticisi (örn: `CFSM` sınıfı) ile birlikte kullanılır. (Tamamen başlık dosyasıdır, ayrı bir `.cpp` dosyası yoktur).

---

### `text_file_loader.h`

*   **Amaç:** `CTextFileLoader` sınıfını tanımlar. Bu sınıf, belirli bir formatta yapılandırılmış metin dosyalarını (genellikle yapılandırma dosyaları, veri dosyaları) yüklemek, içindeki verileri gruplar ve anahtar-değer çiftleri (token vektörleri) olarak ayrıştırmak ve bu verilere hiyerarşik bir şekilde erişim sağlamak için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`TTokenVectorMap` Typedef'i:** `std::map<std::string, TTokenVector>` (Anahtar olarak string, değer olarak token vektörü).
    *   **`SGroupNode` Struct:**
        *   `strGroupName` (std::string): Grubun adı.
        *   `LocalTokenVectorMap` (TTokenVectorMap): Bu gruba ait anahtar-değer çiftlerini tutar.
        *   `pParentNode` (SGroupNode*): Üst grup düğümüne işaretçi.
        *   `ChildNodeVector` (std::vector<SGroupNode*>): Alt grup düğümlerini tutan vektör.
    *   **`CTextFileLoader` Sınıfı:**
        *   **Statik Metot:** `DestroySystem()`: `ms_groupNodePool`'u temizler.
        *   **Yapıcı/Yıkıcı:** `CTextFileLoader()`, `~CTextFileLoader()`.
        *   **Yükleme:** `Load(const char* c_szFileName)`: Metin dosyasını yükler ve ayrıştırır. `GetFileName()`: Yüklenen dosyanın adını döndürür.
        *   **Düğüm Gezintisi:**
            *   `SetTop()`: Mevcut düğümü en üst (global) düğüme ayarlar.
            *   `GetChildNodeCount()`: Mevcut düğümün alt düğüm sayısını döndürür.
            *   `SetChildNode(const char* c_szKey)`, `SetChildNode(const std::string& c_rstrKeyHead, DWORD dwIndex)`, `SetChildNode(DWORD dwIndex)`: Mevcut düğümü belirtilen alt düğüme ayarlar.
            *   `SetParentNode()`: Mevcut düğümü üst düğüme ayarlar.
            *   `GetCurrentNodeName(std::string* pstrName)`: Mevcut düğümün adını alır.
        *   **Token Erişimi:**
            *   `IsToken(const std::string& c_rstrKey)`: Belirtilen anahtarın mevcut düğümde olup olmadığını kontrol eder.
            *   `GetTokenVector(const std::string& c_rstrKey, TTokenVector** ppTokenVector)`: Belirtilen anahtara karşılık gelen token vektörünü alır.
            *   Çeşitli `GetTokenTYPE` metotları: `GetTokenBoolean`, `GetTokenByte`, `GetTokenWord`, `GetTokenInteger`, `GetTokenDoubleWord`, `GetTokenFloat`, `GetTokenVector2`, `GetTokenVector3`, `GetTokenVector4`, `GetTokenPosition`, `GetTokenQuaternion`, `GetTokenDirection`, `GetTokenColor`, `GetTokenString`. Bu metotlar, belirtilen anahtara karşılık gelen token'ı istenen veri türüne dönüştürerek döndürür.
        *   **Korumalı (Protected) Üyeler:**
            *   `LoadGroup(TGroupNode* pGroupNode)`: Bir grup düğümünü ve altındaki verileri özyinelemeli olarak yükler.
            *   `m_strFileName`, `m_dwcurLineIndex`, `mc_pData`: Dosya adı, mevcut satır indeksi ve veri işaretçisi.
            *   `m_fileLoader` (`CMemoryTextFileLoader`): Dosya içeriğini satır satır işlemek için yardımcı bir nesne.
            *   `m_globalNode` (`TGroupNode`): Kök grup düğümü.
            *   `m_pcurNode` (`TGroupNode*`): Mevcut aktif grup düğümü.
        *   **Özel (Private) Statik Üye:**
            *   `ms_groupNodePool` (`CDynamicPool<TGroupNode>`): `SGroupNode` nesneleri için bir bellek havuzu.
*   **Bağlantılı Dosyalar:** `text_file_loader.cpp` (uygulama), `../../common/d3dtype.h` (muhtemelen `D3DXVECTOR` türleri için), `../../common/pool.h` (`CDynamicPool` için), `file_loader.h` (`CMemoryTextFileLoader` ve `TTokenVector` için).

---

### `text_file_loader.cpp`

*   **Amaç:** `text_file_loader.h` içinde bildirilen `CTextFileLoader` sınıfının metotlarını uygular. Metin dosyalarını okuma, gruplar ve anahtar-değerler halinde ayrıştırma ve bu verilere erişim sağlama işlevlerini gerçekleştirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`ms_groupNodePool`:** `SGroupNode` nesneleri için statik bir `CDynamicPool` örneği. `DestroySystem()` ile temizlenir.
    *   **Yapıcı (`CTextFileLoader()`):** `m_dwcurLineIndex`'i 0 yapar, `SetTop()` çağırır, `m_globalNode`'un adını "global" ve ebeveynini `NULL` olarak ayarlar.
    *   **`Load(c_szFileName)`:**
        *   Dosya adını saklar, satır indeksini sıfırlar.
        *   Dosyayı ikili okuma modunda (`"rb"`) açar.
        *   Dosya boyutunu alır, tüm içeriği bir bellek bloğuna okur.
        *   Okunan veriyi `m_fileLoader.Bind()` ile `CMemoryTextFileLoader` nesnesine bağlar. Okunan bellek bloğunu siler.
        *   `LoadGroup(&m_globalNode)` çağırarak ayrıştırma işlemini başlatır.
    *   **`LoadGroup(pGroupNode)`:**
        *   Dosyada satır satır ilerler (`m_fileLoader.GetLineCount()`, `m_fileLoader.SplitLine()`).
        *   Boş satırları veya yorumları (`{`, `}`) atlar.
        *   **"group" Komutu:**
            *   Yeni bir `TGroupNode` oluşturur (`ms_groupNodePool.Alloc()`).
            *   Ebeveynini ve grup adını ayarlar. Mevcut düğümün alt düğüm listesine ekler.
            *   Bir sonraki satıra geçer ve özyinelemeli olarak `LoadGroup(pNewNode)` çağırır.
        *   **"list" Komutu:**
            *   Liste adını (`key`) alır.
            *   Bir sonraki satıra geçer ve `}` karakterine kadar olan tüm satırlardaki token'ları bir `TTokenVector` içinde toplar.
            *   Toplanan token vektörünü `pGroupNode->LocalTokenVectorMap`'e `key` ile ekler.
        *   **Diğer Komutlar (Anahtar-Değer):**
            *   Satırın ilk token'ını anahtar (`key`) olarak alır.
            *   Geri kalan token'ları bir `TTokenVector` içinde toplar.
            *   Bu vektörü `pGroupNode->LocalTokenVectorMap`'e `key` ile ekler.
            *   Değer yoksa hata verir (`sys_err`).
    *   **Düğüm Gezintisi Metotları (`SetTop`, `GetChildNodeCount`, `SetChildNode`, `SetParentNode`, `GetCurrentNodeName`):** `m_pcurNode` işaretçisini ve `ChildNodeVector`'ü kullanarak düğümler arasında gezinmeyi ve bilgi almayı sağlar. `assert` ile geçersiz durumları kontrol eder.
    *   **Token Erişim Metotları (`IsToken`, `GetTokenVector`, `GetTokenTYPE`):**
        *   `IsToken`: `m_pcurNode->LocalTokenVectorMap.find()` ile anahtarın varlığını kontrol eder.
        *   `GetTokenVector`: Anahtarı bulur ve `TTokenVector*` döndürür. Bulamazsa `sys_log` ile hata kaydeder.
        *   Diğer `GetTokenTYPE` metotları: `GetTokenVector`'ı çağırır, dönen vektörün boş olup olmadığını kontrol eder, ilk elemanı alır ve `str_to_number` (sayısal türler için) veya `atof` (float türleri için) gibi fonksiyonlarla istenen türe dönüştürür. Hata durumlarında `sys_log` ile kayıt düşer. Çoklu değerli token'lar (Vector2/3/4, Color) için beklenen sayıda eleman olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `../../common/stl.h` (muhtemelen `stl_lowers` için), `text_file_loader.h`.

---

### `TrafficProfiler.h`

*   **Amaç:** Ağ trafiğiyle ilgili (özellikle gelen paket başlıkları) bilgileri toplamak ve analiz etmek için bir profilleyici sistemi tanımlar. Belirli bir süre içinde hangi paket başlıklarının ne sıklıkta alındığını ve toplam boyutlarını kaydeder. Bu, ağ performansını izlemek veya belirli paket türlerinden kaynaklanan sorunları tespit etmek için kullanılabilir.
*   **Temel İşlevler/İçerik:**
    *   **`TProfileInfo` Struct:**
        *   `count` (unsigned int): Belirli bir paket başlığının kaç kez alındığını sayar.
        *   `length` (unsigned int): Belirli bir paket başlığıyla ilişkili toplam veri boyutunu (byte cinsinden) biriktirir.
        *   Yapıcı: `count` ve `length`'i 0 olarak başlatır.
    *   **`TrafficProfiler` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:** `m_bEnable`'ı `false` yapar, `m_iFlushCycle`'ı 600 saniye (10 dakika) ve `m_iLastFlushTime`'ı 0 olarak ayarlar.
        *   **Public Metotlar:**
            *   `Initialize()`: Bir şey yapmaz (muhtemelen gelecekteki kullanım için).
            *   `Destroy()`: Bir şey yapmaz.
            *   `SetEnable(bool bEnable)`: Profilleyiciyi etkinleştirir veya devre dışı bırakır.
            *   `IsEnabled() const`: Profilleyicinin etkin olup olmadığını döndürür.
            *   `Flush()`: Toplanan profil verilerini (paket başlığı, sayısı, toplam boyutu) `log/traffic_profile/[tarih_saat].txt` formatında bir dosyaya yazar ve ardından mevcut profil verilerini (`m_mapInfos`) temizler. En son boşaltma zamanını (`m_iLastFlushTime`) günceller.
            *   `Profile(BYTE header, int iLen)`: Gelen bir paketin başlığını (`header`) ve uzunluğunu (`iLen`) kaydeder. Eğer profilleyici etkinse:
                *   `m_mapInfos[header].count`'ı artırır.
                *   `m_mapInfos[header].length`'e `iLen`'i ekler.
            *   `GetFlushCycle() const`: Boşaltma döngüsü süresini (saniye cinsinden) döndürür.
            *   `SetFlushCycle(int cycle)`: Boşaltma döngüsü süresini ayarlar.
            *   `GetLastFlushTime() const`: En son boşaltma işleminin zamanını döndürür.
        *   **Private Üyeler:**
            *   `m_bEnable` (bool): Profilleyicinin aktif olup olmadığını belirten bayrak.
            *   `m_mapInfos` (`std::map<BYTE, TProfileInfo>`): Paket başlıklarını (`BYTE`) anahtar olarak, `TProfileInfo` (sayı ve toplam uzunluk) yapılarını değer olarak tutan harita.
            *   `m_iFlushCycle` (int): Profil verilerinin dosyaya ne sıklıkta yazılacağını belirleyen süre (saniye cinsinden).
            *   `m_iLastFlushTime` (int): En son profil verilerinin dosyaya yazıldığı zaman (saniye cinsinden, `time(0)` değeri).
*   **Bağlantılı Dosyalar:** `TrafficProfiler.cpp` (uygulama), `stdafx.h`, `<map>`.

---

### `TrafficProfiler.cpp`

*   **Amaç:** `TrafficProfiler.h` dosyasında bildirilen `TrafficProfiler` sınıfının metotlarını uygular. Ağ trafiği verilerini toplar, belirli aralıklarla dosyaya yazar ve profilleyici durumunu yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`TrafficProfiler::TrafficProfiler()`):**
        *   `m_bEnable`'ı `false` olarak ayarlar.
        *   `m_iFlushCycle`'ı varsayılan 600 saniye (10 dakika) olarak ayarlar.
        *   `m_iLastFlushTime`'ı 0 olarak ayarlar.
    *   **Yıkıcı (`TrafficProfiler::~TrafficProfiler()`):**
        *   Bir şey yapmaz.
    *   **`Flush()` Metodu:**
        *   `log/traffic_profile` dizininin var olup olmadığını kontrol eder, yoksa oluşturur (`mkdir` veya `_mkdir`).
        *   Geçerli zamanı (`time(0)`) alır ve `strftime` kullanarak `YYYYMMDD_HHMMSS.txt` formatında bir dosya adı oluşturur.
        *   Oluşturulan dosya adıyla `log/traffic_profile/` dizini altında bir dosya açar.
        *   `m_mapInfos` haritasındaki her bir giriş (paket başlığı ve `TProfileInfo`) için:
            *   Başlık numarasını (ondalık ve onaltılık formatta), paket sayısını ve toplam uzunluğu dosyaya yazar.
        *   Dosyayı kapatır.
        *   `m_mapInfos.clear()` ile toplanan verileri temizler.
        *   `m_iLastFlushTime`'ı geçerli zamanla günceller.
    *   **`Profile(BYTE header, int iLen)` Metodu:**
        *   `m_bEnable` `false` ise hiçbir şey yapmaz ve döner.
        *   `m_mapInfos[header].count`'ı bir artırır.
        *   `m_mapInfos[header].length`'e `iLen`'i ekler.
    *   **Diğer Metotlar (`Initialize`, `Destroy`, `SetEnable`, `IsEnabled`, `GetFlushCycle`, `SetFlushCycle`, `GetLastFlushTime`):**
        *   Genellikle basit ayarlayıcı (setter) veya alıcı (getter) fonksiyonlarıdır, `m_bEnable`, `m_iFlushCycle`, `m_iLastFlushTime` gibi üye değişkenlerini yönetirler.
        *   `Initialize` ve `Destroy` şu anda bir işlem yapmamaktadır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `TrafficProfiler.h`, `log.h` (muhtemelen `LOG_PATH` veya benzeri bir makro için), `<ctime>` (`time_t`, `strftime`, `localtime` için), `<sys/stat.h>` veya `<direct.h>` (`mkdir` için).

---

### `unique_item.h`

*   **Amaç:** Oyun içinde özel işlevlere sahip, genellikle görevler, etkinlikler veya özel sistemlerle ilişkili olan benzersiz eşyaların ve eşya gruplarının VNUM (Virtual Number - Sanal Numara) değerlerini enum sabitleri olarak tanımlar. Bu, kod içinde bu eşyalara sabit isimlerle ve VNUM'larıyla kolayca referans verilmesini sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`enum` Sabitleri:** Çok sayıda `UNIQUE_GROUP_*` ve `UNIQUE_ITEM_*` (veya sadece `ITEM_*_VNUM`) adlı sabit tanımlar.
        *   **`UNIQUE_GROUP_*`:** Belirli bir kategoriye veya işleve sahip eşya gruplarını temsil eder. Örneğin:
            *   `UNIQUE_GROUP_LUCKY_GOLD`: Şanslı Altın eşya grubu.
            *   `UNIQUE_GROUP_AUTOLOOT`: Otomatik toplama eşya grubu.
            *   `UNIQUE_GROUP_RING_OF_EXP`: Tecrübe yüzüğü grubu.
            *   `UNIQUE_GROUP_SPECIAL_RIDE`: Özel binek grubu.
            *   `DRAGON_SOUL_EXTRACTOR_GROUP`: Ejderha Ruhu Çıkarıcı grubu.
        *   **`UNIQUE_ITEM_*` veya `ITEM_*_VNUM`:** Tekil, özel eşyaların VNUM'larını tanımlar. Örneğin:
            *   `UNIQUE_ITEM_TEARDROP_OF_GODNESS`: Tanrıçanın Gözyaşı.
            *   `UNIQUE_ITEM_RING_OF_LANGUAGE`: Lisan Yüzüğü.
            *   `UNIQUE_ITEM_WHITE_FLAG`: Beyaz Bayrak.
            *   `ITEM_SKILLFORGET_VNUM`: Beceri Sıfırlama Kağıdı.
            *   `UNIQUE_ITEM_FISH_MIND`: Balıkçılık Zihni.
            *   `ITEM_HORSE_FOOD_1`, `ITEM_HORSE_FOOD_2`, `ITEM_HORSE_FOOD_3`: At yemleri.
            *   `ITEM_MARRIAGE_RING`: Evlilik Yüzüğü.
            *   `ITEM_SKILLBOOK_VNUM`: Beceri Kitabı.
            *   `ITEM_AUTO_HP_RECOVERY_S`: Otomatik HP Potu (Küçük).
        *   **Etkinlik Eşyaları:** Belirli oyun etkinlikleriyle (Yılbaşı, Sevgililer Günü, Ramazan vb.) ilgili eşyaların VNUM'ları (örn: `ITEM_NEW_YEAR_GREETING_VNUM`, `ITEM_VALENTINE_ROSE`, `ITEM_RAMADAN_CANDY`).
        *   **Özellik/Sistem Eşyaları:** Belirli oyun sistemleri veya özellikleriyle ilgili eşyalar (örn: `ITEM_ANTI_EXP_RING` (`__ANTI_EXP_RING__`), `MOBILE_MAILBOX` (`__MAILBOX__`), `CHANGE_LOOK_REVERSAL` (`__CHANGE_LOOK_SYSTEM__`)).
    *   **Koşullu Derleme (`#if defined(...)`):** Dosya, birçok `#if defined(__FEATURE_NAME__)` bloğu içerir. Bu, belirli özelliklerin (örn: `__LEADER_BOOK__`, `__RIDING_EXTENDED__`, `__AURA_COSTUME_SYSTEM__`) derleme zamanında aktif olup olmamasına göre ilgili eşya VNUM'larının dahil edilip edilmemesini sağlar.
*   **Kullanım Amacı:** Bu sabitler, oyunun çeşitli yerlerinde (görevler, eşya düşürme tabloları, NPC dükkanları, özel sistemlerin işleyişi vb.) belirli eşyaları VNUM'ları üzerinden tanımlamak ve kontrol etmek için kullanılır. Kod okunabilirliğini artırır ve VNUM'ların yanlış yazılmasından kaynaklanabilecek hataları azaltır.
*   **Bağlantılı Dosyalar:** Bu dosya, eşya VNUM'larına ihtiyaç duyan birçok farklı modül tarafından (`char_item.cpp`, `questlua_item.cpp`, çeşitli özellik sistemlerinin `.cpp` dosyaları vb.) dahil edilir.

---

### `unique_mob.h`

*   **Amaç:** Oyundaki belirli özel veya "benzersiz" kabul edilen canavarların VNUM (Virtual Number - Sanal Numara) değerlerini enum sabitleri olarak tanımlar. Bu, kod içinde bu canavarlara sabit isimlerle kolayca referans verilmesini sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`enum` Sabitleri:**
        *   `THUNDER_BOSS = 6192`: Muhtemelen "Gök Gürültüsü Ejderhası" veya benzeri bir boss canavarının VNUM'u.
        *   `THUNDER_HEALER = 6409`: Muhtemelen `THUNDER_BOSS` ile ilişkili, onu iyileştiren veya destekleyen bir canavarın VNUM'u.
*   **Kullanım Amacı:** Bu sabitler, oyun mantığında (örn: görevler, özel olaylar, boss mekanikleri) belirli canavarları VNUM'ları üzerinden tanımlamak ve onlara özel davranışlar atamak için kullanılır. Kod okunabilirliğini artırır.
*   **Linter Notu:** Bildirilen linter hatası (`#include errors detected. Please update your includePath.`) genellikle derleme ortamındaki include yollarının doğru ayarlanmamasından kaynaklanır ve bu dosyanın içeriğiyle doğrudan ilgili değildir.
*   **Bağlantılı Dosyalar:** Bu dosya, özel canavar VNUM'larına ihtiyaç duyan modüller (örn: görev dosyaları, canavar spawn mantığı, boss dövüş scriptleri) tarafından dahil edilebilir.

---

### `update_limit_time.py`

*   **Amaç:** Bu Python betiği, `limit_time.h` adında bir C++ başlık dosyası oluşturmak için kullanılır. Oluşturulan başlık dosyası, `GLOBAL_LIMIT_TIME` adında bir makro tanımlar. Bu makronun değeri, betiğin çalıştırıldığı zamandan itibaren yaklaşık 2 yıl (tam olarak 360 gün) sonrasını gösteren bir Unix zaman damgasıdır.
*   **Temel İşlevler/İçerik:**
    *   **Zaman Hesaplama:**
        *   `limitTime = time.mktime(time.localtime()) + 3600 * 24 * 180 * 2`:
            *   `time.localtime()`: Mevcut yerel zamanı bir struct olarak alır.
            *   `time.mktime(...)`: Bu struct'ı bir Unix zaman damgasına (saniye cinsinden) dönüştürür.
            *   `3600 * 24 * 180 * 2`: Saniye cinsinden 360 günlük bir süre (180 gün * 2).
            *   Bu iki değer toplanarak gelecekteki bir zaman damgası elde edilir.
    *   **Başlık Dosyası Oluşturma:**
        *   `desc` string değişkeni, oluşturulacak `limit_time.h` dosyasının şablonunu içerir.
        *   Bu şablon, `#ifndef __LIMIT_TIME__`, `#define __LIMIT_TIME__`, `#define ENABLE_LIMIT_TIME` gibi standart başlık korumalarını ve bir özellik etkinleştirme makrosunu içerir.
        *   `#define GLOBAL_LIMIT_TIME %dUL // %s`: Asıl zaman damgasını (`%dUL` formatında unsigned long olarak) ve bu zaman damgasının okunabilir tarih/saat formatını (`%s` ile `time.asctime` çıktısı) bir yorum olarak içerir.
        *   Ayrıca, zaman aşımıyla ilgili olabilecek `TIME_OVER_PONG_DOWN_RATE` (50000) ve `TIME_OVER_LOGIN_DOWN_RATE` (10000) gibi iki ek makro tanımlar.
        *   `open("limit_time.h", "w").write(desc % (limitTime, time.asctime(time.localtime(limitTime))))`: `limit_time.h` dosyasını yazma modunda açar ve `desc` şablonunu hesaplanan `limitTime` ve okunabilir zaman bilgisiyle formatlayarak dosyaya yazar.
*   **Kullanım Amacı:**
    *   Oluşturulan `limit_time.h` dosyası, muhtemelen sunucu yazılımının belirli bir süre sonra çalışmasını kısıtlamak, bir deneme sürümü süresi ayarlamak veya bir lisans geçerlilik tarihi belirlemek gibi bir zaman sınırlaması mekanizması için kullanılır.
    *   `GLOBAL_LIMIT_TIME` makrosu, sunucu kodu içinde bu son geçerlilik tarihini kontrol etmek için kullanılır.
    *   `TIME_OVER_PONG_DOWN_RATE` ve `TIME_OVER_LOGIN_DOWN_RATE` makroları, zaman aşımı durumunda sunucunun davranışını (örneğin, bağlantı düşürme oranları) etkileyebilir.
*   **Çalıştırılma Şekli:** Bu bir Python betiği olduğu için, C++ derleme sürecinin bir parçası olarak veya manuel olarak çalıştırılarak `limit_time.h` dosyasının periyodik olarak güncellenmesi amaçlanmış olabilir.
*   **Bağlantılı Dosyalar:** Bu betik doğrudan C++ koduyla bağlantılı değildir, ancak ürettiği `limit_time.h` dosyası, zaman sınırlaması kontrolü yapan C++ modülleri tarafından `#include` edilir.

---

### `utils.h`

*   **Amaç:** Oyun sunucusunda sıkça kullanılan genel amaçlı yardımcı makroları ve fonksiyon bildirimlerini içerir. Bu fonksiyonlar arasında bit işlemleri, mesafe hesaplamaları, zaman yönetimi, string işleme ve rastgele sayı üretimi gibi çeşitli yardımcı araçlar bulunur.
*   **Temel İşlevler/İçerik:**
    *   **Bit İşlem Makroları:**
        *   `IS_SET(flag, bit)`: Bir bayrak değişkeninde belirli bir bitin kurulu olup olmadığını kontrol eder.
        *   `SET_BIT(var, bit)`: Bir değişkende belirli bir biti kurar.
        *   `REMOVE_BIT(var, bit)`: Bir değişkenden belirli bir biti kaldırır.
        *   `TOGGLE_BIT(var, bit)`: Bir değişkendeki belirli bir bitin durumunu tersine çevirir.
    *   **VNUM Kontrol Makrosu:**
        *   `CHECK_VNUM_RANGE(vnum, min, max, range)`: Bir VNUM'un belirli bir aralıkta (`range` true ise) veya belirli iki değerden biri (`range` false ise) olup olmadığını kontrol eder.
    *   **Mesafe Hesaplama Fonksiyonları (inline):**
        *   `DISTANCE_SQRT(long dx, long dy)`: İki nokta arasındaki Öklid mesafesini karekök alarak hesaplar.
        *   `DISTANCE_APPROX(int dx, int dy)`: İki nokta arasındaki mesafeyi daha hızlı ama yaklaşık bir algoritma ile hesaplar (muhtemelen Manhattan veya Chebyshev mesafesine benzer bir optimizasyon).
    *   **`MAKEWORD` Makrosu (non-WIN32 için inline):**
        *   İki `BYTE` değerinden bir `WORD` oluşturur. Windows dışı sistemler için tanımlanmıştır.
    *   **Fonksiyon Bildirimleri (extern):**
        *   `set_global_time(time_t t)`: Sunucunun global zamanını ayarlar (bir zaman farkı yaratarak).
        *   `get_global_time()`: Ayarlanan global zamanı döndürür.
        *   `dice(int number, int size)`: Belirli sayıda (`number`) ve belirli bir yüz sayısına (`size`) sahip zarlar atar ve toplam sonucu döndürür (örn: 3d6 için `dice(3, 6)`).
        *   `str_lower(const char* src, char* dest, size_t dest_size)`: Bir string'i küçük harfe dönüştürür ve hedef tampona kopyalar.
        *   `skip_spaces(char** string)`: Bir string işaretçisini, string'deki baştaki boşlukları atlayacak şekilde ilerletir.
        *   Argüman Ayrıştırma Fonksiyonları: `one_argument`, `two_arguments`, `three_arguments`. Bir string'den boşluklarla ayrılmış argümanları sırayla çıkarır. Tırnak içindeki ifadeleri tek bir argüman olarak kabul eder.
        *   `first_cmd(const char* argument, char* first_arg, size_t first_arg_size, size_t* first_arg_len_result)`: Bir komut satırından ilk komutu (boşlukla ayrılmış ilk kelimeyi) küçük harfe dönüştürerek alır.
        *   `CalculateDuration(int iSpd, int iDur)`: Bir hız (`iSpd`) değerine göre bir süreyi (`iDur`) ayarlar. Muhtemelen saldırı hızı veya büyü yapma hızı gibi faktörlere bağlı olarak etki sürelerini hesaplamak için kullanılır.
        *   `gauss_random(float avg = 0, float sigma = 1)`: Ortalama (`avg`) ve standart sapma (`sigma`) ile Gauss (normal) dağılımına sahip rastgele bir sayı üretir (Box-Muller transformu kullanarak).
        *   `parse_time_str(const char* str)`: "1h30m10s" gibi bir string'i saniye cinsinden bir süreye dönüştürür.
        *   `WildCaseCmp(const char* w, const char* s)`: İki string'i karşılaştırır. İlk string (`w`) `*` (herhangi bir karakter dizisi) ve `?` (herhangi bir tek karakter) joker karakterlerini içerebilir. Karşılaştırma büyük/küçük harf duyarsızdır.
*   **Bağlantılı Dosyalar:** `<math.h>`, `utils.cpp` (fonksiyonların uygulamaları için).

---

### `utils.cpp`

*   **Amaç:** `utils.h` başlık dosyasında bildirilen genel amaçlı yardımcı fonksiyonların çoğunu uygular. String işleme, zaman yönetimi, rastgele sayı üretimi ve çeşitli hesaplama görevleri için fonksiyonlar içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`split_argument(const char *argument, std::vector<std::string> & vecArgs)`:**
        *   Verilen bir string'i (`argument`) boşluklara göre böler ve sonuçları bir `std::vector<std::string>` içine atar. Boost kütüphanesinin `boost::split` ve `boost::is_any_of(" ")` fonksiyonlarını kullanır. `boost::token_compress_on` ile birden fazla ardışık boşluğu tek bir ayraç gibi kabul eder.
    *   **Global Zaman Yönetimi:**
        *   `global_time_gap` (static int): Gerçek zaman ile sunucunun global zamanı arasındaki farkı (saniye cinsinden) tutar.
        *   `get_global_time()`: `time(0) + global_time_gap` ile mevcut global zamanı döndürür.
        *   `set_global_time(time_t t)`: `global_time_gap`'i, verilen `t` zamanına ulaşmak için gereken fark olarak ayarlar. Ayarlanan yeni global zamanı loglar.
    *   **`dice(int number, int size)`:**
        *   `number` (zar sayısı) ve `size` (her zarın yüz sayısı) parametrelerini alır.
        *   `number` kadar döngüde, her seferinde 1 ile `size` arasında rastgele bir sayı (`thecore_random() % size + 1`) üretir ve toplama ekler.
        *   Toplam sonucu döndürür. Geçersiz girişler (sayı veya boyut <= 0) için 0 döndürür.
    *   **`str_lower(const char* src, char* dest, size_t dest_size)`:**
        *   Kaynak string'i (`src`) karakter karakter okur, her karakteri `LOWER()` makrosu (muhtemelen `tolower()`) ile küçük harfe çevirir ve hedef tampona (`dest`) yazar.
        *   Hedef tamponun (`dest_size`) taşmamasını sağlar ve sonuna null karakter (`\0`) ekler. Kopyalanan karakter sayısını döndürür.
    *   **`skip_spaces(const char** string)`:**
        *   Verilen string işaretçisini (`*string`), null karaktere veya boşluk olmayan bir karaktere (`isnhspace` - muhtemelen `isspace` veya benzeri) ulaşana kadar ilerletir.
    *   **Argüman Ayrıştırma Fonksiyonları (`one_argument`, `two_arguments`, `three_arguments`):**
        *   `one_argument(const char* argument, char* first_arg, size_t first_size)`: String'in başındaki boşlukları atlar. Tırnak (`"`) işaretlerini dikkate alarak bir sonraki boşluğa kadar olan kısmı `first_arg`'a kopyalar. Kalan string'i döndürür.
        *   `two_arguments` ve `three_arguments`: `one_argument`'ı zincirleme çağırarak birden fazla argümanı ayrıştırır.
    *   **`first_cmd(const char* argument, char* first_arg, size_t first_arg_size, size_t* first_arg_len_result)`:**
        *   Baştaki boşlukları atlar. İlk boşluk olmayan karaktere kadar olan kısmı küçük harfe çevirerek `first_arg`'a kopyalar. Kopyalanan uzunluğu `first_arg_len_result`'a yazar. Kalan string'i döndürür.
    *   **`CalculateDuration(int iSpd, int iDur)`:**
        *   Verilen hız (`iSpd`) ve temel süreye (`iDur`) göre ayarlanmış bir süre hesaplar. Formül, hız 100'den küçükse süreyi artırır, büyükse azaltır. Muhtemelen bir karakterin saldırı hızı veya büyü yapma hızı gibi bir değere göre bir etkinin süresini (örneğin bir zehir veya buff süresi) ayarlamak için kullanılır.
    *   **Rastgele Sayı Üretimi:**
        *   `uniform_random(double a, double b)`: `a` ve `b` arasında düzgün dağılıma sahip rastgele bir `double` sayı üretir.
        *   `gauss_random(float avg, float sigma)`: Ortalama (`avg`) ve standart sapma (`sigma`) değerleriyle normal (Gauss) dağılıma sahip rastgele bir `float` sayı üretir. Box-Muller dönüşümünü kullanır ve verimlilik için bir sonraki Gauss sayısını statik değişkende saklar.
    *   **`parse_time_str(const char* str)`:**
        *   "1d2h30m15s" gibi bir zaman string'ini saniye cinsinden bir tamsayıya dönüştürür. `d` (gün), `h` (saat), `m` (dakika), `s` (saniye) soneklerini tanır. Geçersiz formatta -1 döndürür.
    *   **`WildCaseCmp(const char* w, const char* s)`:**
        *   Joker karakterli (`*` ve `?`) ve büyük/küçük harf duyarsız string karşılaştırması yapar.
        *   `*`: Sıfır veya daha fazla karakterle eşleşir (özyinelemeli olarak kontrol eder).
        *   `?`: Herhangi bir tek karakterle eşleşir.
        *   Diğer karakterler için `tolower()` kullanarak büyük/küçük harf duyarsız eşleşme kontrolü yapar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `<boost/algorithm/string/classification.hpp>`, `<boost/algorithm/string/split.hpp>`.

---

### `vector.h`

*   **Amaç:** 3D vektörleri temsil eden bir `VECTOR` yapısını ve bu vektörlerle ilgili temel geometrik ve trigonometrik hesaplamalar yapan yardımcı fonksiyonların bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   **`SVector` (VECTOR) Struct:**
        *   Bir 3D vektörün x, y, ve z koordinatlarını `float` türünde saklar.
            *   `float x;`
            *   `float y;`
            *   `float z;`
    *   **Fonksiyon Bildirimleri (extern):**
        *   `Normalize(VECTOR* pV1, VECTOR* pV2)`: `pV1` vektörünü normalize eder (birim vektör haline getirir) ve sonucu `pV2`'ye yazar.
        *   `GetDegreeFromPosition(float x, float y)`: Orijinden (0,0) verilen (x,y) pozisyonuna olan yönün açısını derece cinsinden hesaplar (genellikle Y ekseni 0 derece kabul edilir).
        *   `GetDegreeFromPositionXY(long sx, long sy, long ex, long ey)`: Başlangıç (`sx`, `sy`) ve bitiş (`ex`, `ey`) koordinatları verilen bir yöne ait açıyı derece cinsinden hesaplar.
        *   `GetDeltaByDegree(float fDegree, float fDistance, float* x, float* y)`: Verilen bir açı (`fDegree`) ve mesafe (`fDistance`) doğrultusunda x ve y eksenlerindeki değişimi (`*x`, `*y`) hesaplar.
        *   `GetDegreeDelta(float iDegree, float iDegree2)`: İki açı (`iDegree`, `iDegree2`) arasındaki mutlak farkı derece cinsinden hesaplar. Açıları -180 ile 180 derece aralığına normalize eder.
*   **Bağlantılı Dosyalar:** `vector.cpp` (fonksiyonların uygulamaları için).

---

### `vector.cpp`

*   **Amaç:** `vector.h` dosyasında bildirilen 3D vektör ve açı hesaplama fonksiyonlarını uygular.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Sabitler ve Makrolar:**
        *   `_PI ((float) 3.141592654f)`: Pi sayısı.
        *   `_1BYPI ((float) 0.318309886f)`: 1/Pi değeri.
        *   `DegreeToRadian(degree)`: Dereceyi radyana çevirir (`degree * (_PI / 180.0f)`).
        *   `RadianToDegree(radian)`: Radyanı dereceye çevirir (`radian * (180.0f / _PI)`).
    *   **`Normalize(VECTOR* pV1, VECTOR* pV2)`:**
        *   `pV1` vektörünün uzunluğunu (`l = sqrtf(x*x + y*y + z*z)`) hesaplar. Çok küçük uzunluklar için (0'a bölme hatasını önlemek amacıyla) `1.0e-12` ekler.
        *   `pV1`'in her bir bileşenini (x, y, z) bu uzunluğa bölerek `pV2`'ye yazar.
    *   **`DotProduct(VECTOR* pV1, VECTOR* pV2)` (static):**
        *   İki vektörün skaler çarpımını (`pV1->x * pV2->x + pV1->y * pV2->y + pV1->z * pV2->z`) hesaplar ve döndürür.
    *   **`GetDegreeFromPosition(float x, float y)`:**
        *   Verilen (x,y) koordinatlarından bir yön vektörü (`vtDir`) oluşturur (z=0).
        *   Bu yön vektörünü `Normalize` ile birim vektör haline getirir.
        *   Referans olarak bir standart yön vektörü (`vtStan` = (0,1,0), yani pozitif Y ekseni) tanımlar.
        *   `vtDir` ve `vtStan` arasındaki açıyı `acosf(DotProduct(&vtDir, &vtStan))` ile radyan cinsinden bulur ve `RadianToDegree` ile dereceye çevirir.
        *   Eğer `vtDir.x < 0.0f` ise (yön sol yarı düzlemdeyse), açıyı `360.0f - ret` olarak ayarlar (0-360 derece aralığında bir sonuç elde etmek için).
        *   Hesaplanan açıyı döndürür.
    *   **`GetDegreeFromPositionXY(long sx, long sy, long ex, long ey)`:**
        *   `GetDegreeFromPosition(ex - sx, ey - sy)` çağırarak iki nokta arasındaki yönün açısını hesaplar.
    *   **`GetDeltaByDegree(float fDegree, float fDistance, float* x, float* y)`:**
        *   Verilen açıyı (`fDegree`) `DegreeToRadian` ile radyana çevirir.
        *   `*x = fDistance * sin(fRadian)`
        *   `*y = fDistance * cos(fRadian)`
        *   Bu, standart birim çember trigonometrisine göre, y ekseninin 0 derece olduğu ve açıların saat yönünde arttığı bir koordinat sistemi varsayar.
    *   **`GetDegreeDelta(float iDegree, float iDegree2)`:**
        *   Her iki açıyı da -180 ile 180 derece aralığına normalize eder (eğer 180'den büyükse 360 çıkarır).
        *   İki normalize edilmiş açı arasındaki mutlak farkı (`fabs(iDegree - iDegree2)`) döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `vector.h`.

---

### `vid.h`

*   **Amaç:** Oyun dünyasındaki varlıkları (entity) veya nesneleri benzersiz bir şekilde tanımlamak için kullanılan `VID` (Virtual ID) sınıfını tanımlar. Bir `VID`, bir `DWORD` türünde benzersiz bir kimlik (`m_id`) ve bu kimlikle ilişkilendirilmiş bir `DWORD` türünde CRC (Döngüsel Artıklık Denetimi) değeri (`m_crc`) içerir. CRC, ID'nin geçerliliğini veya belirli bir örneğe ait olduğunu doğrulamak için kullanılabilir.
*   **Temel İşlevler/İçerik:**
    *   **`VID` Sınıfı:**
        *   **Yapıcılar:**
            *   `VID()`: Varsayılan yapıcı. `m_id` ve `m_crc`'yi 0 olarak başlatır.
            *   `VID(DWORD id, DWORD crc)`: Verilen `id` ve `crc` değerleriyle başlatır.
            *   `VID(const VID& rvid)`: Kopyalayıcı yapıcı.
        *   **Atama Operatörü (`operator=`):** Bir `VID` nesnesini diğerine kopyalar.
        *   **Karşılaştırma Operatörleri:**
            *   `operator== (const VID& rhs) const`: İki `VID` nesnesinin hem `m_id` hem de `m_crc` değerleri eşitse `true` döndürür.
            *   `operator!= (const VID& rhs) const`: İki `VID` nesnesi eşit değilse `true` döndürür.
        *   **Dönüşüm Operatörü (`operator DWORD() const`):** `VID` nesnesini doğrudan `m_id` değerine (yani `DWORD` türüne) dönüştürülmesini sağlar. Bu, `VID` nesnesinin bir `DWORD` ID beklenen yerlerde kolayca kullanılmasına olanak tanır.
        *   **`Reset()` Metodu:** `m_id` ve `m_crc` değerlerini 0'a sıfırlar.
        *   **Özel (Private) Üyeler:**
            *   `DWORD m_id`: Benzersiz kimlik.
            *   `DWORD m_crc`: Kimlikle ilişkili CRC değeri.
*   **Kullanım Senaryoları:**
    *   Oyundaki karakterler, canavarlar, eşyalar gibi varlıkların her birine benzersiz bir `VID` atanabilir.
    *   CRC değeri, bir ID'nin geçerli bir varlığa işaret edip etmediğini veya varlığın belirli bir durumda olup olmadığını kontrol etmek için bir tür güvenlik veya doğrulama mekanizması olarak kullanılabilir.
    *   Örneğin, bir karakter yok edildiğinde, ona ait `VID`'in CRC'si geçersiz kılınabilir, böylece eski ID'ye yapılan referansların geçersiz olduğu anlaşılır.
*   **Linter Notu:** Bildirilen linter hatası (`#include errors detected...`) genellikle derleme ortamındaki include yollarının doğru ayarlanmamasından kaynaklanır ve bu dosyanın içeriğiyle doğrudan ilgili değildir.
*   **Bağlantılı Dosyalar:** Bu dosya genellikle `stdafx.h` veya temel tür tanımlarını içeren başka bir başlık dosyası aracılığıyla birçok farklı modül tarafından dahil edilir.

---

### `version.cpp`

*   **Amaç:** Sunucunun versiyon bilgisini bir dosyaya yazdırmak için kullanılan `WriteVersion()` fonksiyonunu içerir. Bu fonksiyon genellikle sunucu başlatıldığında çağrılarak `version.txt` adlı bir dosyaya projenin adını ve derleme zamanında tanımlanan bir versiyon numarasını yazar.
*   **Temel İşlevler/İçerik:**
    *   **`WriteVersion()` Fonksiyonu:**
        *   Bu fonksiyon sadece Windows dışı (`#ifndef __WIN32__`) sistemlerde çalışacak şekilde tasarlanmıştır.
        *   `version.txt` dosyasını yazma modunda (`"w"`) açmaya çalışır.
        *   Dosya başarıyla açılırsa:
            *   `"M2_V6 Project\n"` stringini dosyaya yazar.
            *   `"METIN2 Official Lieke Project: %s\n"` formatlı stringi, `__VERSION__` makrosunun değeriyle birlikte dosyaya yazar.
            *   Dosyayı kapatır (`fclose`).
        *   Dosya açılamazsa (`NULL != fp` başarısız olursa):
            *   Standart hata akışına (`stderr`) "Cannot open version.txt" mesajını yazar.
            *   Programdan çıkar (`exit(0)`).
*   **Linter Notu ve `__VERSION__` Makrosu:**
    *   Linter tarafından bildirilen `identifier "__VERSION__" is undefined` hatası, `__VERSION__` makrosunun bu kaynak dosyada veya dahil ettiği başlıklarda tanımlanmamış olmasından kaynaklanır.
    *   Bu tür bir makro genellikle derleme komut satırında `-D__VERSION__="x.y.z"` şeklinde derleyiciye bir argüman olarak verilir. Bu sayede, her derlemede farklı bir versiyon numarası kolayca ayarlanabilir.
*   **Kullanım Amacı:**
    *   Sunucunun hangi versiyonunun çalıştığını kolayca tespit etmek için bir yöntem sağlar.
    *   Otomatik derleme ve dağıtım sistemlerinde versiyon takibi için kullanılabilir.
*   **Bağlantılı Dosyalar:** `<stdio.h>` (`fopen`, `fprintf`, `fclose`, `stderr`), `<stdlib.h>` (`exit`). (Windows sistemlerinde bu fonksiyonun içeriği derlenmeyeceği için `stdafx.h` veya diğer proje başlıklarına doğrudan bir bağımlılığı olmayabilir.)

---

</rewritten_file> 