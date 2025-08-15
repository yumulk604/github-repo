# EterPack Referans Kılavuzu

Bu belge, Metin2 istemcisinin `EterPack` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. `EterPack`, genellikle oyunun varlık dosyalarını (.epk, .eix uzantılı paketler) yönetmek, bu paketlerden veri okumak, şifreleme/şifre çözme işlemleri ve dosya sistemiyle ilgili soyutlamalar için kullanılır.

---

## Dosya Bazlı Detaylandırma 

### `EterPack.h` ve `EterPack.cpp` (`CEterPack` ve Yardımcı Sınıflar)

*   **Genel Amaç:** Bu dosyalar, Metin2 istemcisinin kullandığı `.eix` (indeks dosyası) ve `.epk` (veri dosyası) formatındaki paketlenmiş varlık dosyalarını oluşturmak, okumak, yönetmek ve bu dosyalardaki verilere erişmek için temel sınıfları ve fonksiyonları tanımlar. `CEterPack` sınıfı, tek bir `.eix`/`.epk` çiftini temsil ederken, `CEterFileDict` birden fazla paket içindeki dosyalara hızlı erişim için bir sözlük yapısı sunar. Sistem, LZO sıkıştırması ve çeşitli şifreleme katmanlarını (Panama, özelleştirilmiş hibrit şifreleme) destekler.

*   **`EterPack.h` - Temel Tanımlar ve Bildirimler:**
    *   **Sabitler ve Enum'lar:**
        *   `EterPack::c_PackCC`, `EterPack::c_IndexCC`: Paket ve indeks dosyaları için "EPKD" olarak tanımlanmış FourCC (tanımlayıcı) kodları.
        *   `EterPack::c_Version`: Paket formatının sürüm numarası (şu anki sürüm 2).
        *   `EterPack::c_HeaderSize`: İndeks dosyasının başlık boyutu (FourCC + Sürüm + İndeks Sayısı).
        *   `EEterPackTypes` Enum'u: Dosya adı uzunluğu (`FILENAME_MAX_LEN`), veritabanı adı uzunluğu (`DBNAME_MAX_LEN`), serbest blok boyutları ve çeşitli sıkıştırma/şifreleme türleri için sabitler tanımlar:
            *   `COMPRESSED_TYPE_NONE`: Sıkıştırma veya şifreleme yok.
            *   `COMPRESSED_TYPE_COMPRESS`: LZO ile sıkıştırılmış.
            *   `COMPRESSED_TYPE_SECURITY`: LZO ile sıkıştırılmış ve özel bir anahtarla şifrelenmiş (genellikle `COMPRESS_EPK` anahtarı ile).
            *   `COMPRESSED_TYPE_PANAMA`: Panama stream cipher ile şifrelenmiş.
            *   `COMPRESSED_TYPE_HYBRIDCRYPT`: `EterPackPolicy_CSHybridCrypt` ile yönetilen karma şifreleme.
            *   `COMPRESSED_TYPE_HYBRIDCRYPT_WITHSDB`: `COMPRESSED_TYPE_HYBRIDCRYPT` gibi, ancak ek bir "Supplementary Data Block" (SDB) içerir.
    *   **`SEterPackIndex` Yapısı (`#pragma pack(push, 4)` ile):**
        *   Bir `.eix` dosyasındaki her bir dosya girdisinin metadatasını tanımlar:
            *   `id`: Dosyanın paketteki benzersiz kimliği (indeks sırası).
            *   `filename[FILENAME_MAX_LEN + 1]`: Dosyanın paket içindeki adı.
            *   `filename_crc`: Dosya adının CRC32 özeti (hızlı arama için).
            *   `real_data_size`: Dosyanın `.epk` içinde kapladığı gerçek (genellikle hizalanmış) boyut.
            *   `data_size`: Dosyanın orijinal (veya sıkıştırılmış/şifrelenmiş verinin) boyutu.
            *   `MD5Digest[16]` (eğer `CHECKSUM_CHECK_MD5` tanımlıysa) veya `data_crc` (DWORD): Veri bütünlüğü için MD5 özeti veya CRC32.
            *   `data_position`: Dosya verisinin `.epk` dosyasındaki başlangıç konumu (offset).
            *   `compressed_type`: Dosyanın sıkıştırma/şifreleme türünü belirten `char` (yukarıdaki `EEterPackTypes` enum değerlerinden biri).
    *   **`CEterFileDict` Sınıfı:**
        *   Birden fazla `CEterPack` nesnesindeki tüm dosyalara merkezi ve hızlı erişim sağlamak için bir sözlük (dictionary) görevi görür.
        *   `Item` (iç yapı): Bir `CEterPack` işaretçisi ve o paketteki bir `SEterPackIndex` işaretçisini tutar.
        *   `TDict` (typedef `std::unordered_multimap<DWORD, Item>`): Dosya adı CRC32'sini `Item` yapılarına eşler. `unordered_multimap` kullanılması, CRC32 çakışmalarını (farklı dosya adlarının aynı CRC32'ye sahip olması) ele almak içindir.
        *   `InsertItem()`, `UpdateItem()`, `GetItem()`: Sözlüğe öğe ekleme, güncelleme ve getirme metotları.
    *   **`CEterPack` Sınıfı:**
        *   Tek bir `.eix` (indeks) ve `.epk` (veri) dosya çiftini yönetir.
        *   `Create()`: Bir paket oluşturur veya mevcut bir paketi açar, indeks dosyasını okur/işler.
        *   `DecryptIV()`: Panama şifrelemesi için kullanılan Başlatma Vektörünün (IV) şifresini çözer.
        *   `Get()` / `Get2()`: Belirtilen bir dosyayı paketten okur. Veriyi bir `CMappedFile` nesnesi aracılığıyla döndürür ve sıkıştırma/şifre çözme işlemlerini otomatik olarak yapar.
        *   `Put()`: Pakete yeni bir dosya ekler veya mevcut bir dosyayı günceller (eğer paket salt okunur değilse). Veriyi sıkıştırır/şifreler.
        *   `Delete()`: Paketten bir dosyayı (mantıksal olarak) siler (indeks girdisini serbest olarak işaretler).
        *   `Extract()`: Paketteki tüm dosyaları diske çıkarır.
        *   `IsExist()`: Bir dosyanın pakette olup olmadığını kontrol eder.
        *   `EncryptIndexFile()` / `DecryptIndexFile()`: İndeks dosyasını şifreler/şifresini çözer (genellikle LZO sıkıştırması ve basit bir XOR veya özel anahtar ile).
        *   `m_pCSHybridCryptPolicy`: `EterPackPolicy_CSHybridCrypt` türünde bir işaretçi, karma şifreleme politikasını yönetir.
        *   `m_stIV_Panama`: Panama şifrelemesi için kullanılan IV (Başlatma Vektörü).
        *   CryptoPP kütüphanesi ile entegrasyon için metotlar (Panama şifrelemesi).
    *   **`CMakePackLog` Sınıfı:**
        *   Paket oluşturma veya düzenleme işlemleri sırasında log ve hata mesajları yazmak için basit bir singleton loglama sınıfı.
        *   `.log` ve `.err` uzantılı dosyalara yazar.

*   **`EterPack.cpp` - Uygulama Detayları:**
    *   **`CMakePackLog` Uygulaması:** Standart dosya I/O operasyonları ile log ve hata dosyalarına yazar.
    *   **`CEterPack` Uygulaması:**
        *   **Yapıcı/Yok Edici:** `m_pCSHybridCryptPolicy` nesnesini oluşturur/siler. `Destroy()` metodu tüm iç veri yapılarını temizler.
        *   **`Create()` Metodu:**
            *   Verilen `.eix` (indeks) dosyasını açar. Dosya yoksa ve salt okunur değilse yenisini oluşturur.
            *   İndeks dosyasının başlığını (FourCC, Sürüm) kontrol eder.
            *   Eğer indeks dosyası şifreliyse (FourCC `CLZObject::ms_dwFourCC` ise), `CLZO::Instance().Decompress()` ile LZO açma ve şifre çözme işlemi yapar.
            *   İndeks girdilerini okur (`m_indexData`), her bir girdi için `filename_crc` kullanarak `m_DataPositionMap`'e ekler. Boş girdileri `m_FreeIndexList`'e atar.
            *   `CEterFileDict` nesnesini günceller.
            *   Salt okunur modda değilse ve indeks şifreliyse, `DecryptIndexFile()` çağrılarak indeksin şifresi çözülüp düz formatta yeniden yazılır.
        *   **`DecryptIV()` Metodu:** Verilen bir Panama anahtarı ile `m_stIV_Panama` üyesinin XOR'lanarak şifresini çözer.
        *   **`Get()` / `Get2()` Metotları (Veri Okuma):**
            1.  `FindIndex()` ile dosya adından `SEterPackIndex` bilgisini bulur.
            2.  `CMappedFile::Create()` ile `.epk` dosyasının ilgili bölümünü belleğe mapler.
            3.  Eğer `compressed_type` `COMPRESSED_TYPE_SECURITY` veya `COMPRESSED_TYPE_PANAMA` ise, veri bütünlüğü kontrolü yapılır (MD5 veya CRC32).
            4.  `compressed_type`'a göre gerekli işlemleri yapar:
                *   `COMPRESSED_TYPE_NONE`: Veri doğrudan kullanılır.
                *   `COMPRESSED_TYPE_COMPRESS`: `CLZO::Instance().Decompress()` ile LZO açma.
                *   `COMPRESSED_TYPE_SECURITY`: `CLZO::Instance().Decompress()` özel bir anahtar (`COMPRESS_EPK`) ile LZO açma ve şifre çözme.
                *   `COMPRESSED_TYPE_PANAMA`: `__Decrypt_Panama()` ile Panama stream cipher şifre çözme. Bu metot, dosya adından türetilen bir anahtar ve `m_stIV_Panama`'yı kullanır. CryptoPP kütüphanesi (Tiger, SHA1, RIPEMD128, Whirlpool, Panama) kullanılır. Genellikle dosyanın ilk 2048 baytı şifrelenir.
                *   `COMPRESSED_TYPE_HYBRIDCRYPT` / `COMPRESSED_TYPE_HYBRIDCRYPT_WITHSDB`: `m_pCSHybridCryptPolicy->DecryptMemory()` çağrılır. `_WITHSDB` durumunda ek olarak `GetSupplementaryDataBlock()` ile SDB verisi alınıp ana veriye eklenir.
            5.  Açılmış/çözülmüş veri `CMappedFile` nesnesine bağlanır ve kullanıcıya döndürülür.
        *   **`Put()` Metodu (Veri Yazma):**
            1.  Paket şifreli veya salt okunur ise hata verir.
            2.  Veriyi `packType`'a göre sıkıştırır/şifreler:
                *   LZO sıkıştırması (`COMPRESSED_TYPE_COMPRESS`).
                *   LZO + Şifreleme (`COMPRESSED_TYPE_SECURITY`).
                *   Panama şifrelemesi (`COMPRESSED_TYPE_PANAMA`) (`__Encrypt_Panama()` ile).
                *   Hibrit şifreleme (`COMPRESSED_TYPE_HYBRIDCRYPT`, `COMPRESSED_TYPE_HYBRIDCRYPT_WITHSDB`) (`m_pCSHybridCryptPolicy->EncryptMemory()` ile). `_WITHSDB` için `GenerateSupplementaryDataBlock` çağrılır.
            3.  İşlenmiş verinin CRC32 veya MD5 özetini hesaplar.
            4.  Mevcut bir dosya ise ve yeni veri eski yerine sığıyorsa, indeksi günceller. Sığmıyorsa veya dosya yeniyse, eski indeksi (varsa) serbest bırakır, `NewIndex()` ile yeni bir indeks girdisi ve `GetNewDataPosition()` ile veri için yeni bir pozisyon alır.
            5.  İndeks ve veri dosyalarına (`WriteIndex`, `WriteData`/`WriteNewData`) yazar.
        *   **`__CreateFileNameKey_Panama()` Metodu:** Dosya adından bir CRC32 hesaplar. Bu CRC32 değerine göre farklı CryptoPP hash algoritmaları (Tiger, SHA1, RIPEMD128, Whirlpool) seçilerek dosya adından 32 byte'lık bir anahtar türetilir. Bu anahtar Panama şifrelemesinde kullanılır.
        *   **İndeks ve Veri Yönetimi:** `FindIndex`, `NewIndex`, `PushFreeIndex`, `WriteIndex`, `WriteData` gibi metotlar, `.eix` ve `.epk` dosyalarının yapısını, serbest alan yönetimini ve veri bloklarının yerleşimini yönetir.
    *   **`CEterFileDict` Uygulaması:** `unordered_multimap` kullanarak dosya adı CRC'sine göre `Item` yapılarını saklar ve arar. CRC çakışması durumunda tam dosya adı karşılaştırması yapar.

*   **Kullanım Amacı:**
    *   Oyunun grafik, ses, script gibi tüm varlıklarını paketlenmiş dosyalarda tutarak dağıtımı ve yönetimi kolaylaştırır.
    *   Dosya erişimini soyutlar; istemci kodu, dosyaların diskte mi yoksa bir paket içinde mi olduğundan bağımsız olarak dosyalara erişebilir.
    *   LZO sıkıştırması ile varlıkların diskte kapladığı alanı azaltır.
    *   Çeşitli şifreleme katmanları (Panama, HybridCrypt) ile varlıkların yetkisiz erişime ve değiştirilmeye karşı korunmasına yardımcı olur.
    *   `CEterPackManager` (bu dosyalarda tanımlanmamış ama genellikle birlikte kullanılır) aracılığıyla birden fazla `.epk` dosyası aynı anda yönetilebilir ve dosya arama işlemleri öncelik sırasına göre yapılabilir.

*   **Bağımlılıklar:**
    *   CryptoPP kütüphanesi: Panama, Tiger, SHA1, RIPEMD128, Whirlpool gibi kriptografik algoritmalar için kullanılır.
    *   LZO kütüphanesi (CLZO sınıfı aracılığıyla): Veri sıkıştırma/açma işlemleri için.
    *   `EterBase`: Temel yardımcı fonksiyonlar (`utils.h`, `CRC32.h`), hata ayıklama (`Debug.h`) ve bellek eşlemli dosya (`MappedFile.h`) sınıfları için.
    *   `UserInterface/Locale_inc.h`: Yerelleştirmeyle ilgili tanımlar için.
    *   İsteğe bağlı olarak Themida SDK (`__THEMIDA__` makrosu ile).

### `EterPackCursor.h` ve `EterPackCursor.cpp` (`CEterPackCursor` Sınıfı)

*   **Amaç:** `CEterPackCursor` sınıfı, bir `CEterPack` nesnesi içindeki belirli bir dosyaya erişmek ve bu dosya üzerinde sıralı okuma işlemleri yapmak için bir imleç (cursor) veya akış (stream) benzeri bir arayüz sunar. Bir dosya paketten açıldığında, bu sınıf o dosyanın verileri üzerinde gezinmeyi ve parçalar halinde okumayı kolaylaştırır.

*   **Sınıf Yapısı ve Metotları:**
    *   **Yapıcı (`CEterPackCursor(CEterPack* pack)`)**: Bir `CEterPack` nesnesine işaretçi alarak başlatılır.
    *   **Yıkıcı (`~CEterPackCursor()`)**: `Close()` metodunu çağırarak açık olan dosyayı kapatır.
    *   **`Open(const char* filename)` (bool)**: Belirtilen dosya adını kullanarak ilişkili `CEterPack` nesnesinden dosyayı açar.
        *   `inlineConvertPackFilename()` ile dosya adı normalleştirilir.
        *   `m_pPack->Get()` metodunu çağırarak dosyayı bir `CMappedFile` (`m_file`) olarak alır ve veri işaretçisini (`m_pData`) ayarlar.
        *   Başarılı olursa `true`, aksi takdirde `false` döner.
    *   **`Close()` (void)**: Açık olan `CMappedFile` nesnesini (`m_file.Destroy()`) kapatır, veri işaretçisini (`m_pData`) `NULL` yapar ve okuma noktasını (`m_ReadPoint`) sıfırlar.
    *   **`Seek(long offset)` (void)**: Okuma noktasını (`m_ReadPoint`) dosya içinde belirtilen `offset` değerine ayarlar. Değer, dosya boyutları içinde kalacak şekilde (`max(0, min(Size(), offset))`) sınırlandırılır.
    *   **`Read(LPVOID data, long size)` (bool)**: Mevcut okuma noktasından (`m_ReadPoint`) itibaren belirtilen `size` kadar baytı, verilen `data` tamponuna okur.
        *   Eğer dosya açık değilse veya istenen okuma miktarı dosya sonunu aşıyorsa `false` döner.
        *   `memcpy` kullanarak veriyi kopyalar ve `m_ReadPoint`'i okunan bayt sayısı kadar ilerletir.
        *   Başarılı olursa `true` döner.
    *   **`Size()` (long)**: Açık olan dosyanın toplam boyutunu (`m_file.Size()`) döndürür. Dosya açık değilse 0 döner.

*   **Özel Üyeler:**
    *   `m_pPack` (`CEterPack*`): Bu imlecin ait olduğu ana paket nesnesi.
    *   `m_file` (`CMappedFile`): Paketten açılan dosyayı temsil eden bellek eşlemli dosya nesnesi.
    *   `m_pData` (`LPCVOID`): `m_file` tarafından sağlanan, dosya verilerinin başlangıcını gösteren işaretçi.
    *   `m_ReadPoint` (long): Dosya içindeki mevcut okuma pozisyonu (bayt cinsinden offset).

*   **Kullanım Amacı:**
    *   Bir `CEterPack` içindeki bir dosyayı açtıktan sonra, dosyanın tamamını bir kerede belleğe yüklemek yerine, ihtiyaç duyulan kısımlarını parça parça okumak için kullanılır.
    *   Dosya içinde rastgele erişim (belirli bir noktaya `Seek` ile gidip oradan `Read` ile okuma) imkanı sunar.
    *   Büyük dosyaların işlenmesinde veya dosya formatlarının ayrıştırılmasında (parsing) kullanışlıdır.
    *   Örneğin, bir yapılandırma dosyası, bir script dosyası veya büyük bir veri bloğu paketten bu imleç aracılığıyla okunabilir ve işlenebilir.

### `EterPackManager.h` ve `EterPackManager.cpp` (`CEterPackManager` Sınıfı)

*   **Amaç:** `CEterPackManager` sınıfı, bir singleton olarak, istemci tarafından kullanılan tüm `CEterPack` (.epk/.eix paketleri) örneklerini merkezi bir şekilde yönetir. Dosyaların bu paketlerden veya doğrudan diskten aranması, okunması, yeni paketlerin kaydedilmesi ve şifreleme anahtarları gibi paket özelliklerinin yönetilmesinden sorumludur.

*   **Sınıf Yapısı ve Temel Özellikler (`CEterPackManager`):**
    *   **Singleton:** `CSingleton<CEterPackManager>`'dan miras alır.
    *   **`SCache` Yapısı (iç yapı):** Basit bir dosya önbelleği (cache) için veri tamponunu (`m_abBufData`) ve boyutunu (`m_dwBufSize`) tutar.
    *   **`ESearchModes` Enum'u:** Dosya arama önceliğini belirler:
        *   `SEARCH_FILE_FIRST`: Önce diskte (yerel dosya sisteminde) ara, bulunamazsa paketlerde ara.
        *   `SEARCH_PACK_FIRST`: Önce kayıtlı paketlerde ara, bulunamazsa diskte ara.
    *   **Veri Yapıları (Korumalı Üyeler):**
        *   `m_FileDict` (`CEterFileDict`): Tüm kayıtlı paketlerdeki dosyaların birleşik bir sözlüğünü tutar (hızlı arama için).
        *   `m_RootPack` (`CEterPack`): Genellikle "root" veya temel bir paket için kullanılır (belirli bir dizine bağlı olmayan dosyalar için olabilir).
        *   `m_PackList` (`TEterPackList` - `std::list<CEterPack*>`): Kayıtlı `CEterPack` nesnelerinin bir listesini tutar (muhtemelen arama sırası veya yönetim kolaylığı için).
        *   `m_PackMap` (`TEterPackMap` - `std::unordered_map<std::string, CEterPack*, stringhash>`): Paket adlarını (veya .eix dosya adlarını) `CEterPack` işaretçilerine eşler.
        *   `m_DirPackMap` (`TEterPackMap`): Paketlerin ilişkilendirildiği dizin yollarını `CEterPack` işaretçilerine eşler.
        *   `m_kMap_dwNameKey_kCache` (`std::unordered_map<DWORD, SCache>`): Dosya adı CRC32'sini `SCache` nesnelerine eşleyen bir önbellek haritası.
        *   `m_csFinder` (`CRITICAL_SECTION`): Dosya arama ve erişim işlemlerini iş parçacığı güvenli (thread-safe) hale getirmek için kullanılır.
    *   **Temel Metotlar:**
        *   **Yapıcı/Yıkıcı:** `m_csFinder`'ı başlatır/siler, `__ClearCacheMap()` ile önbelleği ve `m_PackMap`'teki tüm `CEterPack` nesnelerini temizler.
        *   **`SetCacheMode()` / `SetRelativePathMode()`**: Önbellekleme modunu ve göreceli yol deneme modunu aktif eder.
        *   **`LoadStaticCache(const char* c_szFileName)`**: Belirtilen dosyayı paketlerden okur ve içeriğini `m_kMap_dwNameKey_kCache` içine yükler (eğer önbellek modu aktifse).
        *   **`SetSearchMode(bool bPackFirst)`**: Dosya arama önceliğini ayarlar.
        *   **`Get(CMappedFile& rMappedFile, const char* c_szFileName, LPCVOID* pData)` (bool)**: Ana dosya getirme fonksiyonu. `m_iSearchMode`'a göre `GetFromPack` veya `GetFromFile`'ı çağırır.
        *   **`GetFromPack(CMappedFile& rMappedFile, const char* c_szFileName, LPCVOID* pData)` (bool)**:
            *   Dosya adını `ConvertFileName` ile normalleştirir.
            *   Önce `__FindCache` ile önbellekte arar. Varsa, önbellekten döndürür.
            *   `m_FileDict.GetItem()` ile dosyayı paket sözlüğünde arar.
            *   Bulunursa, ilgili `CEterPack::Get2()` metodunu çağırarak dosyayı okur.
        *   **`GetFromFile(CMappedFile& rMappedFile, const char* c_szFileName, LPCVOID* pData)` (bool)**: Dosyayı doğrudan diskten okumaya çalışır (`CMappedFile::Create` kullanarak). Mutlak yolları (örn: `d:/ymir work/`) veya istemci kök dizinine göreceli yolları deneyebilir.
        *   **`isExist(const char* c_szFileName)` (bool)**: Bir dosyanın var olup olmadığını kontrol eder (arama moduna göre önce paket veya önce dosya).
        *   **`isExistInPack(const char* c_szFileName)` (bool)**: Bir dosyanın herhangi bir kayıtlı pakette olup olmadığını kontrol eder.
        *   **`RegisterPack(const char* c_szName, const char* c_szDirectory, const BYTE* c_pbIV = NULL)` (bool)**: Yeni bir `.eix`/`.epk` paketini yöneticiye kaydeder.
            *   Aynı isimde bir paket zaten kayıtlı değilse, yeni bir `CEterPack` nesnesi oluşturur ve `CEterPack::Create()` ile başlatır.
            *   Paketi `m_PackMap` ve `m_DirPackMap`'e ekler.
            *   `c_pbIV` ile Panama şifrelemesi için IV (Başlatma Vektörü) sağlanabilir.
        *   **`RegisterRootPack(const char* c_szName)`**: `m_RootPack` nesnesini belirtilen paket dosyasıyla başlatır.
        *   **`DecryptPackIV(DWORD key)`**: Kayıtlı tüm paketlerin `DecryptIV` metodunu çağırarak Panama IV'lerinin şifresini çözer.
        *   **Hibrit Şifreleme Yönetimi (`EterPackPolicy_CSHybridCrypt` ile ilgili):**
            *   `RetrieveHybridCryptPackKeys(const BYTE* pStream)`: Bir akıştan (stream) hibrit şifreleme anahtarlarını okur ve ilgili paketlerin politikalarına dağıtır.
            *   `RetrieveHybridCryptPackSDB(const BYTE* pStream)`: Bir akıştan hibrit şifreleme için Ek Veri Bloklarını (SDB) okur ve ilgili paketlerin politikalarına dağıtır.
            *   `WriteHybridCryptPackInfo(const char* pFileName)`: Kayıtlı paketlerdeki tüm hibrit şifreleme anahtarlarını ve SDB bilgilerini belirtilen bir dosyaya yazar (genellikle paket oluşturma/dağıtım araçları tarafından kullanılır).
        *   **`ConvertFileName(const char* c_szFileName, std::string& rstrFileName)` (int)**: Dosya yolundaki `\` karakterlerini `/` ile değiştirir, tüm harfleri küçük harfe çevirir ve yoldaki dizin sayısını döndürür.
        *   **`CompareName(const char* c_szDirectoryName, DWORD dwLength, const char* c_szFileName)` (bool)**: Bir dosya adının belirli bir dizin altında olup olmadığını kontrol eder.

*   **Kullanım Amacı:**
    *   Oyun istemcisinin ihtiyaç duyduğu tüm varlık dosyalarına (.epk paketleri içinden veya doğrudan diskten) erişim için merkezi bir arayüz sağlar.
    *   Birden fazla paket dosyasının (örneğin, `locale_xx.epk`, `pack/sound.epk`, `season1.epk` vb.) yönetilmesini ve bu paketler arasında dosya arama işlemlerinin verimli bir şekilde yapılmasını sağlar.
    *   Dosya arama sırasını yapılandırma imkanı sunarak modlama veya geliştirme süreçlerini kolaylaştırabilir (örneğin, önce yerel diskteki dosyaları kontrol et, sonra paketlere bak).
    *   Sık erişilen dosyalar için basit bir önbellekleme mekanizması sunarak performansı artırabilir.
    *   Özellikle `COMPRESSED_TYPE_HYBRIDCRYPT` ve `COMPRESSED_TYPE_PANAMA` gibi karmaşık şifreleme yöntemlerinin anahtar ve IV yönetimini merkezileştirir.

*   **Notlar:**
    *   Kod içinde `__THEMIDA__` makrosu ile korunan bölümler, Themida adlı bir kod koruma/obfuscation aracıyla entegrasyonu gösterir.
    *   `ArrangeMemoryMappedPack()` metodu yorum satırı haline getirilmiş; muhtemelen bellek eşlemli paketlerin belirli bir süre sonra bellekten kaldırılması (unmap edilmesi) gibi bir optimizasyon için düşünülmüş ancak aktif olarak kullanılmıyor.

### `EterPackPolicy_CSHybridCrypt.h` ve `EterPackPolicy_CSHybridCrypt.cpp` (`EterPackPolicy_CSHybridCrypt` Sınıfı)

*   **Amaç:** Bu sınıf, `CEterPack` sistemi içinde "Hybrid Crypt" olarak adlandırılan bir şifreleme/şifre çözme politikasını yönetir. Bu politika, dosya uzantılarına ve dosya adlarına bağlı olarak farklı simetrik şifreleme algoritmaları (Camellia, Twofish, XTEA) ve bu algoritmalara özgü anahtar/IV (Başlatma Vektörü) çiftleri kullanarak `.epk` paketlerindeki dosyaların güvenliğini sağlamayı amaçlar. Ayrıca, isteğe bağlı olarak Ek Veri Blokları (Supplementary Data Blocks - SDB) oluşturma ve yönetme yeteneğine de sahiptir.

*   **Sınıf Yapısı ve Temel Özellikler:**
    *   **`eHybridCipherAlgorithm` Enum'u:** Desteklenen şifreleme algoritmalarını tanımlar: `e_Cipher_Camellia`, `e_Cipher_Twofish`, `e_Cipher_XTEA`.
    *   **`UEncryptKey` ve `UEncryptIV` Union'ları:** Farklı şifreleme algoritmalarının gerektirdiği anahtar ve IV boyutlarını (genellikle 16 byte) tek bir union yapısında birleştirir.
    *   **`SCSHybridCryptKey` Yapısı:** Bir `UEncryptKey` ve bir `UEncryptIV` üyesini içerir.
    *   **`TCSHybridCryptKeyMap` (typedef `std::unordered_map<DWORD, TCSHybridCryptKey>`):** Dosya uzantılarının CRC32 hash'lerini (`DWORD`) `SCSHybridCryptKey` yapılarına eşler. Her dosya uzantısı için bir ana anahtar/IV çifti saklar.
    *   **`SSupplementaryDataBlockInfo` Yapısı:** Bir SDB için ilişkili harita adını (`strRelatedMapName` - kullanım amacı tam açık değil) ve SDB verisini (`vecStream`) tutar.
    *   **`TSupplementaryDataBlockMap` (typedef `std::unordered_map<DWORD, SSupplementaryDataBlockInfo>`):** Dosya adlarının CRC32 hash'lerini `SSupplementaryDataBlockInfo` yapılarına eşler.
    *   **Temel Metotlar:**
        *   **`GenerateCryptKey(std::string& rfileName)` (bool)**: Verilen dosya adının uzantısı için (eğer daha önce oluşturulmamışsa) rastgele bir anahtar ve IV çifti (`TCSHybridCryptKey`) üretir ve `m_mapHybridCryptKey` içine kaydeder.
        *   **`GetPerFileCryptKey(std::string& rfileName, eHybridCipherAlgorithm& eAlgorithm, TEncryptKey& key, TEncryptIV& iv)` (bool)**: Bir dosya için kullanılacak şifreleme algoritmasını, anahtarı ve IV'yi belirler.
            1.  Dosya adının uzantısının hash'ini kullanarak `m_mapHybridCryptKey`'den ilgili ana anahtar/IV çiftini alır.
            2.  Dosya adının tam yolunun CRC32 hash'ini hesaplar.
            3.  Şifreleme algoritmasını (`eAlgorithm`), bu dosya adı CRC32'sinin `Num_Of_Ciphers` ile modu alınarak seçer (Camellia, Twofish, XTEA arasında döngüsel bir seçim).
            4.  Alınan ana anahtar/IV'yi, dosya adı CRC32'si ile XOR'layarak o dosyaya özgü (per-file) bir anahtar ve IV türetir.
        *   **`EncryptMemory(std::string& rfilename, IN const BYTE* pSrcData, IN int iSrcLen, OUT CLZObject& zObj)` (bool)**: Verilen kaynak veriyi (`pSrcData`), `GetPerFileCryptKey` ile elde edilen algoritma, anahtar ve IV'yi kullanarak şifreler. Şifreleme için CryptoPP kütüphanesinin `CTR_Mode` (Counter Mode) operasyon modunu ve seçilen algoritmayı (Camellia, Twofish, XTEA) kullanır. Sonuç, `CLZObject` (`zObj`) içinde döndürülür.
        *   **`DecryptMemory(std::string& rfilename, IN const BYTE* pSrcData, IN int iSrcLen, OUT CLZObject& zObj)` (bool)**: `EncryptMemory`'nin tersi işlemini yapar; şifreli verinin şifresini çözer.
        *   **`IsContainingCryptKey()` (bool const)**: `m_mapHybridCryptKey`'de kayıtlı en az bir anahtar olup olmadığını kontrol eder.
        *   **Ek Veri Bloğu (SDB) Metotları:**
            *   **`GenerateSupplementaryDataBlock(...)` (bool)**: Bir dosya için SDB oluşturur. Rastgele bir boyutta (64-128 byte arası) kaynak verinin sonundan bir parça alır ve bunu `m_mapSDBMap` içinde saklar. Asıl veri (`pDestData`, `iDestLen`) SDB çıkarıldıktan sonraki kısmı işaret eder.
            *   **`GetSupplementaryDataBlock(...)` (bool)**: Bir dosya için saklanmış SDB verisini döndürür.
            *   **`IsContainingSDBFile()` (bool const)**: En az bir SDB kaydı olup olmadığını kontrol eder.
        *   **Giriş/Çıkış (I/O) Metotları:**
            *   **`WriteCryptKeyToFile(CFileBase& rFile)`**: `m_mapHybridCryptKey` içindeki tüm uzantı hash-anahtar/IV çiftlerini belirtilen dosyaya yazar.
            *   **`ReadCryptKeyInfoFromStream(IN const BYTE* pStream)` (int)**: Bir byte akışından (stream) anahtar/IV bilgilerini okuyarak `m_mapHybridCryptKey`'i doldurur.
            *   **`WriteSupplementaryDataBlockToFile(CFileBase& rFile)`**: `m_mapSDBMap` içindeki tüm SDB bilgilerini (dosya hash, ilişkili harita adı, SDB verisi) dosyaya yazar.
            *   **`ReadSupplementatyDataBlockFromStream(IN const BYTE* pStream)` (int)**: Bir byte akışından SDB bilgilerini okuyarak `m_mapSDBMap`'i doldurur (istemci tarafında `strRelatedMapName` okunmaz).

*   **Kullanım Amacı:**
    *   `CEterPack` sınıfı tarafından, `COMPRESSED_TYPE_HYBRIDCRYPT` ve `COMPRESSED_TYPE_HYBRIDCRYPT_WITHSDB` olarak işaretlenmiş dosyaların şifrelenmesi ve şifresinin çözülmesi için bir politika nesnesi olarak kullanılır.
    *   Paket oluşturma araçları (`CEterPack::Put` ve `CEterPackManager::WriteHybridCryptPackInfo`) bu sınıfı kullanarak anahtarlar üretir, verileri şifreler ve anahtar/SDB bilgilerini özel bir dosyaya veya akışa yazar.
    *   Oyun istemcisi, `CEterPackManager::RetrieveHybridCryptPackKeys` ve `RetrieveHybridCryptPackSDB` aracılığıyla bu anahtar/SDB bilgilerini yükler ve ardından `CEterPack::Get()` sırasında bu politika sınıfı, dosyaların şifresini çözmek için kullanılır.
    *   Dosya uzantısına ve dosya adına dayalı bu çok katmanlı anahtar türetme mekanizması, paket içindeki farklı dosyalar için farklı şifreleme anahtarları kullanılmasını sağlayarak güvenliği artırmayı hedefler.

*   **Bağımlılıklar:**
    *   CryptoPP kütüphanesi: Camellia, Twofish, XTEA simetrik şifreleme algoritmaları ve CTR modu için.
    *   `EterBase`: `CFileNameHelper`, `CFileBase`, `CRC32`, `lzo.h` (doğrudan kullanılmasa da `CLZObject` ile ilişkili olabilir), `Random.h`.

*   **Notlar:**
    *   SDB'nin tam amacı ve `strRelatedMapName`'in ne için kullanıldığı koddan net olarak anlaşılamamaktadır. Genellikle şifrelenmiş verinin bir kısmının ayrı tutulması, ek bir doğrulama adımı, dijital imza parçası veya özel bir veri bloğu içermesi gibi amaçlarla yapılabilir.
    *   Şifreleme için `CTR_Mode` (Counter Mode) kullanılması, şifreleme ve şifre çözme işlemlerinin paralel yapılabilmesine olanak tanır ve aynı anahtar/IV çiftiyle aynı blokların aynı şekilde şifrelenmesini sağlar (padding gerektirmez).

### `Inline.h` (Yardımcı Inline Fonksiyonlar)

*   **Amaç:** Bu başlık dosyası, dosya yolu manipülasyonu için sık kullanılan ve performans açısından inline olarak tanımlanması tercih edilen küçük yardımcı fonksiyonlar içerir.

*   **Fonksiyonlar:**
    *   **`inlinePathCreate(const char* path)`**: Verilen bir dosya yolundaki (`path`) tüm üst dizinleri oluşturmaya çalışır. Yol ayıracı olarak `/` karakterini kullanır.
        *   Yol string'ini tarar, her `/` karakterinde bir dizin segmenti belirler ve `CreateDirectory` Windows API fonksiyonunu kullanarak bu dizini oluşturur.
        *   **Not:** Sağlanan kodda `CreateDirectory` fonksiyonuna `char*` tipinde bir argüman geçilmektedir. Eğer proje Unicode olarak derleniyorsa veya `CreateDirectory`'nin `CreateDirectoryW` (Unicode versiyonu) olması bekleniyorsa bu bir uyumsuzluğa ve potansiyel hataya yol açabilir. ANSI versiyonu (`CreateDirectoryA`) `char*` ile çalışır.
    *   **`inlineConvertPackFilename(char* name)`**: Bir dosya adı/yolu string'ini `EterPack` sistemi için standart bir formata dönüştürür.
        *   String içindeki tüm ters eğik çizgileri (`\`) düz eğik çizgi (`/`) ile değiştirir.
        *   Tüm karakterleri küçük harfe çevirir (`tolower`).
        *   Bu, farklı işletim sistemlerinden veya kullanıcı girdilerinden gelebilecek yol ayrımlarını ve büyük/küçük harf farklılıklarını standartlaştırarak paket içinde tutarlı dosya adları kullanılmasını sağlar.

*   **Kullanım Amacı:**
    *   `CEterPack` veya `CEterPackManager` gibi sınıflar içinde, paketlere dosya eklerken veya paketlerden dosya çıkarırken (extract) dosya yollarını oluşturmak veya normalleştirmek için kullanılır.

### `md5.h` ve `md5.c` (MD5 Özet Algoritması)

*   **Amaç:** Bu dosyalar, RSA Data Security, Inc. tarafından geliştirilen MD5 (Message-Digest Algorithm 5) özet algoritmasının bir C implementasyonunu sağlar. MD5, verilen bir girdi verisinden (mesaj) 128-bit (16 byte) uzunluğunda benzersiz bir özet (hash veya parmak izi) üretir. Bu özet, veri bütünlüğünü doğrulamak için kullanılır.

*   **`md5.h` - Başlık Dosyası:**
    *   `MD5_CTX` yapısını tanımlar: MD5 hesaplama işlemi sırasında durumu (state) tutan bağlam (context) yapısıdır. İçerisinde ara değerleri (`buf`), işlenen bit sayısını (`i`) ve sonuç özetini (`digest`) barındırır.
    *   Temel MD5 fonksiyonlarını bildirir:
        *   `MD5Init(MD5_CTX* mdContext)`: Bir `MD5_CTX` yapısını başlatır.
        *   `MD5Update(MD5_CTX* mdContext, unsigned char* inBuf, unsigned int inLen)`: Mevcut bağlama yeni bir veri bloğu ekleyerek özeti günceller.
        *   `MD5Final(MD5_CTX* mdContext)`: MD5 hesaplamasını sonlandırır ve nihai 16 byte'lık özeti `mdContext->digest` içine yazar.
        *   `MD5Transform(UINT4* buf, UINT4* in)`: MD5 algoritmasının temel dönüşüm fonksiyonudur; 64 byte'lık bir giriş bloğunu işleyerek ara özet değerlerini günceller.

*   **`md5.c` - Kaynak Dosyası:**
    *   `MD5Init`, `MD5Update`, `MD5Final` ve `MD5Transform` fonksiyonlarının tam implementasyonunu içerir.
    *   `MD5Transform` fonksiyonu, MD5'in 4 turunu (round) ve her turdaki F, G, H, I temel fonksiyonlarını, özel sabitleri ve bit rotasyonlarını kullanarak uygular.
    *   Dosyanın başında RSA Data Security, Inc. lisans ve telif hakkı bilgileri bulunur.
    *   Kod, `#ifndef CPU386` ve `#else /* CPU386 */` blokları ile ayrılmış iki `MD5Transform` implementasyonu içerir. Sağlanan kodda genel C implementasyonu görünmektedir; `CPU386` tanımlıysa, 386/486 işlemciler için optimize edilmiş assembly versiyonunun kullanıldığı bir bölüm de (muhtemelen ayrı bir dosyada veya bu dosyanın tam halinde) mevcuttur.

*   **Kullanım Amacı:**
    *   `CEterPack` sınıfında, `CHECKSUM_CHECK_MD5` makrosu tanımlı olduğunda, `.epk` paketlerine eklenen veya paketlerden okunan dosyaların veri bütünlüğünü doğrulamak için kullanılır.
    *   Bir dosya pakete eklenirken MD5 özeti hesaplanıp `SEterPackIndex` yapısında saklanır. Dosya daha sonra paketten okunurken tekrar MD5 özeti hesaplanır ve saklanan özetle karşılaştırılır. Eğer özetler farklıysa, dosyanın bozulmuş olabileceği anlaşılır.

### `obfuscate.h` (Derleme Zamanı String Gizleme)

*   **Amaç:** Bu başlık dosyası, C++ template'leri ve makrolar kullanarak string literallerinin derleme zamanında basit bir XOR şifrelemesi ile gizlenmesini (obfuscation) sağlayan bir mekanizma sunar. Bu, programın çalıştırılabilir dosyasındaki hassas string'lerin (örneğin, API anahtarları, özel mesajlar, dosya yolları) kolayca okunmasını engelleyerek tersine mühendisliği bir miktar zorlaştırmayı hedefler.

*   **Temel Bileşenler (`ay` namespace'i içinde):**
    *   **`obfuscator<std::size_t N, char KEY>` Sınıfı (template):**
        *   Yapıcısında (`constexpr obfuscator(const char* data)`), derleme zamanında verilen `data` string'inin her karakterini belirtilen `KEY` ile XOR'layarak `m_data` üyesinde saklar.
        *   `getData()`, `getSize()`, `getKey()` gibi `constexpr` metotlarla gizlenmiş veriye ve anahtara erişim sağlar.
    *   **`obfuscated_data<std::size_t N, char KEY>` Sınıfı (template):**
        *   Bir `obfuscator` nesnesi ile başlatılır ve gizlenmiş veriyi kendi `m_data` üyesine kopyalar.
        *   `decrypt()`: Saklanan gizlenmiş veriyi aynı `KEY` ile tekrar XOR'layarak orijinal string'i elde eder.
        *   `encrypt()`: Eğer veri çözülmüşse, tekrar XOR'layarak gizler.
        *   `is_encrypted()`: Verinin o an şifreli olup olmadığını kontrol eder (genellikle string'in son karakterinin null olup olmadığına bakarak).
        *   `operator char* ()`: Sınıf nesnesinin `char*` tipine dönüştürülmesini sağlar. Bu dönüşüm sırasında otomatik olarak `decrypt()` çağrılır, böylece gizlenmiş string doğrudan kullanılabilir hale gelir.
        *   Yıkıcısında (`~obfuscated_data()`), `m_data` içeriğini sıfırlayarak orijinal veya gizlenmiş string'in bellekten silinmesine yardımcı olur.
    *   **`make_obfuscator<std::size_t N, char KEY = '.'>(const char(&data)[N])` Fonksiyonu (template `constexpr`):**
        *   Bir string literali ve isteğe bağlı bir anahtar alarak bir `obfuscator` nesnesi oluşturur. String uzunluğunu (`N`) otomatik olarak çıkarır.
    *   **`AY_OBFUSCATE(data)` ve `AY_OBFUSCATE_KEY(data, key)` Makroları:**
        *   String literallerini kolayca gizlemek için kullanılır.
        *   Derleme zamanında bir `obfuscator` oluşturur, ardından bu `obfuscator` ile bir `obfuscated_data` nesnesi (statik ömürlü bir lambda içinde) oluşturur ve bu nesneye bir referans döndürür.
        *   `AY_OBFUSCATE` varsayılan anahtar olarak `'.'` kullanırken, `AY_OBFUSCATE_KEY` özel bir anahtar belirtilmesine izin verir.
        *   String'in null karakterle sonlandığından emin olmak için bir `static_assert` içerir.

*   **Kullanım Amacı:**
    *   Program içindeki sabit string'lerin, özellikle de hassas olabilecek bilgilerin (şifreler, anahtarlar, özel sunucu adresleri, debug mesajları vb.) çalıştırılabilir dosyada doğrudan görünmesini engellemek.
    *   String gizlendiği için, basit bir string arama aracıyla bu bilgilere ulaşmak zorlaşır.
    *   Çalışma zamanında string gerektiğinde (örneğin, ekrana basılacaksa veya bir fonksiyona parametre olarak geçilecekse) `obfuscated_data` nesnesi üzerinden otomatik olarak veya manuel `decrypt()` çağrısıyla orijinal haline getirilir.

### `StdAfx.h` ve `StdAfx.cpp` (Ön Derlenmiş Başlıklar)

*   **Amaç:** Bu dosyalar, Microsoft Visual C++ projelerinde yaygın olarak kullanılan ön derlenmiş başlık (precompiled header - PCH) mekanizmasının bir parçasıdır. Temel amaçları, `EterPack` kütüphanesi içinde sıkça kullanılan ve nadiren değiştirilen standart Windows başlıklarını, diğer temel kütüphane başlıklarını (`EterBase` gibi) ve yerelleştirme tanımlarını tek bir yerde toplayarak projenin genel derleme süresini önemli ölçüde kısaltmaktır.

*   **`StdAfx.h` İçeriği:**
    *   `#pragma once`: Bu başlık dosyasının her derleme birimi (.cpp dosyası) için yalnızca bir kez dahil edilmesini sağlar, çoklu dahil etme sorunlarını önler.
    *   `#define WIN32_LEAN_AND_MEAN`: Windows başlıklarından (`windows.h`) daha az kullanılan API'leri (örneğin, RPC, OLE, Winsock'un bazı eski kısımları) hariç tutarak derleme süresini ve ön derlenmiş başlığın boyutunu azaltır.
    *   `#include "../UserInterface/Locale_inc.h"`: İstemcinin yerelleştirme ayarlarıyla (dil, bölge vb.) ilgili tanımları ve makroları içerir. Bu, `EterPack`'in bazı bölümlerinin yerelleştirilmiş metinler veya davranışlarla etkileşime girebileceğini veya bu tanımlara ihtiyaç duyan diğer başlıkları dolaylı olarak içerdiğini gösterir.
    *   `// #include <crtdbg.h>`: Yorum satırı haline getirilmiş. Aktif olsaydı, C çalışma zamanı kütüphanesinin hata ayıklama rutinlerini (örneğin, bellek sızıntısı tespiti için `_CrtDumpMemoryLeaks`) dahil ederdi.
    *   `#include <windows.h>`: Temel Windows API fonksiyonları, veri türleri ve makroları için gereklidir. `EterPack` içindeki dosya işlemleri, bellek yönetimi ve diğer alt seviye sistem etkileşimleri için kullanılır.
    *   `#include <assert.h>`: `assert()` makrosunu sağlar. Bu makro, programın belirli bir noktasında doğru olması beklenen bir koşulu kontrol etmek için kullanılır; koşul yanlışsa programı sonlandırır (genellikle debug modunda).
    *   `#include "../EterBase/StdAfx.h"`: `EterPack` kütüphanesinin temel aldığı `EterBase` kütüphanesinin ön derlenmiş başlığını dahil eder. Bu, `EterBase` tarafından sağlanan tüm temel sınıfların, yardımcı fonksiyonların ve tanımların `EterPack` içinde de erişilebilir olmasını sağlar. Bu, modülerliğin ve kod tekrarını önlemenin önemli bir parçasıdır.

*   **`StdAfx.cpp` İçeriği:**
    *   `#include "stdafx.h"`: Bu dosya yalnızca kendi başlık dosyası olan `StdAfx.h`'ı içerir. Derleyici bu .cpp dosyasını işlediğinde, `StdAfx.h` içinde listelenen tüm başlıkları derler ve sonucunda bir ön derlenmiş başlık dosyası (.pch) oluşturur. Projedeki diğer .cpp dosyaları daha sonra bu .pch dosyasını kullanarak derleme süreçlerini hızlandırır.

*   **Kullanım Amacı:**
    *   `EterPack` kütüphanesinin derleme sürelerini optimize etmek.
    *   Sık kullanılan başlıkların her .cpp dosyasında tekrar tekrar derlenmesini önlemek.
    *   Proje genelinde ortak kullanılan temel Windows API'lerine, `EterBase` kütüphanesine ve yerelleştirme tanımlarına tutarlı erişim sağlamak.