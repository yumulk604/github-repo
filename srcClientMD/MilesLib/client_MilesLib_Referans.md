# MilesLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `MilesLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. `MilesLib`, Miles Sound System (MSS) kütüphanesini kullanarak oyun içi 2D ses efektleri, 3D pozisyonel sesler ve müzik/ses akışları (streaming) gibi çeşitli ses işlemlerini yönetmek için bir C++ arayüzü ve sarmalayıcı katmanı sağlar.

---

## Dosya Bazlı Detaylandırma 

### `SoundBase.h` ve `SoundBase.cpp` (`CSoundBase` Sınıfı)

*   **Amaç:** `CSoundBase` sınıfı, `MilesLib` içindeki diğer ses yönetimi sınıfları için temel altyapıyı ve ortak statik üyeleri sağlar. Miles Sound System'in (MSS) başlatılması, kapatılması ve yüklenen ses dosyalarına ait verilerin (`CSoundData`) merkezi bir haritada (`ms_dataMap`) saklanmasından sorumludur. Referans sayımı (`ms_iRefCount`) mekanizması ile MSS'nin yalnızca ihtiyaç duyulduğunda başlatılıp, artık kullanılmadığında kapatılmasını güvence altına alır.

*   **`SoundBase.h` - Temel Tanımlar ve Bildirimler:**
    *   **Typedef'ler:**
        *   `SProvider` (struct `TProvider`): Bir MSS ses sağlayıcısının (provider) adını (`char* name`) ve handle'ını (`HPROVIDER hProvider`) tutar.
        *   `TSoundDataMap`: `std::map<DWORD, CSoundData*>` - Dosya CRC32 değerini anahtar olarak kullanarak `CSoundData` işaretçilerini saklar.
    *   **Statik Üye Değişkenler:**
        *   `ms_iRefCount` (int): `CSoundBase` veya türevlerinden kaç örneğin aktif olduğunu sayar. MSS'nin başlatılması/kapatılması için kullanılır.
        *   `ms_DIGDriver` (HDIGDRIVER): Miles Sound System'in dijital ses sürücüsü için handle. (Bu dosyada doğrudan kullanılmıyor gibi görünse de, genellikle MSS ile ilgili genel bir değişkendir).
        *   `ms_pProviderDefault` (TProvider*): Varsayılan ses sağlayıcısına işaretçi.
        *   `ms_ProviderVector` (`std::vector<TProvider>`): Kullanılabilir tüm ses sağlayıcılarını tutan bir vektör.
        *   `ms_dataMap` (TSoundDataMap): Yüklenmiş tüm ses dosyalarının verilerini (CRC -> `CSoundData*`) saklar.
        *   `ms_bInitialized` (bool): MSS'nin başlatılıp başlatılmadığını belirten bir bayrak. (Mevcut kodda `Initialize` ve `Destroy` mantığı doğrudan `ms_iRefCount`'a dayanıyor, bu değişkenin aktif kullanımı görünmüyor.)
    *   **Metot Bildirimleri:**
        *   `Initialize()`: Referans sayısını artırır. Eğer ilk çağrıysa, MSS'yi (`AIL_startup()`) başlatır, `miles` klasörünü redist directory olarak ayarlar, provider listesini ve data map'i temizler.
        *   `Destroy()`: Referans sayısını azaltır. Eğer son örnekse, `ms_dataMap` içindeki tüm `CSoundData` nesnelerini siler ve MSS'yi (`AIL_shutdown()`) kapatır.
        *   `AddFile(DWORD dwFileCRC, const char* filename)`: Verilen dosya adı için yeni bir `CSoundData` nesnesi oluşturur, dosyayı yükler ve `ms_dataMap`'e CRC ile ekler.
        *   `GetFileCRC(const char* filename)`: Verilen dosya adının CRC32 değerini hesaplar.

*   **`SoundBase.cpp` - Uygulama Detayları:**
    *   **`Initialize()`:**
        *   `ms_iRefCount`'u artırır.
        *   Eğer `ms_iRefCount` 1'den büyükse (yani zaten başlatılmışsa) hemen çıkar.
        *   `AIL_set_redist_directory("miles")`: Miles Sound System için gerekli DLL'lerin veya diğer dosyaların bulunduğu `miles` alt klasörünü belirtir.
        *   `AIL_startup()`: Miles Sound System'i başlatır.
        *   `ms_ProviderVector` ve `ms_dataMap`'i temizler (yeni bir başlangıç için).
    *   **`Destroy()`:**
        *   `ms_iRefCount`'u azaltır.
        *   Eğer `ms_iRefCount` hala 0'dan büyükse (yani başka aktif örnekler varsa) hemen çıkar.
        *   `ms_dataMap` üzerinde gezinerek tüm `CSoundData` işaretçilerini alır ve `delete` ile bellekten siler, ardından map'i temizler.
        *   `AIL_shutdown()`: Miles Sound System'i kapatır.
    *   **`GetFileCRC(const char* filename)`:** `EterBase` veya benzeri bir kütüphaneden gelen `GetCRC32` fonksiyonunu kullanarak dosya adının (string) CRC32'sini hesaplar.
    *   **`AddFile(DWORD dwFileCRC, const char* filename)`:**
        *   Yeni bir `CSoundData` nesnesi oluşturur.
        *   `pSoundData->Assign(filename)` ile ses dosyasını yükleyip `CSoundData` nesnesine atar (Bu `Assign` metodu `SoundData.cpp` içinde tanımlıdır).
        *   Oluşturulan `CSoundData` nesnesini `dwFileCRC` anahtarıyla `ms_dataMap`'e ekler.

*   **Kullanım Amacı:**
    *   Bu sınıf, doğrudan örneklenmek yerine, genellikle `CSoundManager`, `CSoundManager2D`, `CSoundManager3D` gibi daha özelleşmiş ses yöneticisi sınıfları tarafından miras alınır veya statik metotları kullanılır.
    *   Oyun boyunca yüklenen tüm ses dosyalarının (`.wav`, `.mp3` vb.) verilerini merkezi bir yerde tutar ve bu verilere CRC aracılığıyla hızlı erişim sağlar.
    *   Miles Sound System'in oyunun yaşam döngüsü boyunca yalnızca bir kez başlatılıp kapatılmasını yönetir, kaynak sızıntılarını ve çakışmaları önler.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h` - Stdafx.h üzerinden dolaylı olarak dahil edilir).
    *   `SoundData.h` (`CSoundData` sınıfı).
    *   `EterBase/CRC32.h` (veya benzeri, `GetCRC32` için).
    *   Standart C++ kütüphaneleri (`map`, `vector`, `string`).

### `SoundData.h` ve `SoundData.cpp` (`CSoundData` Sınıfı)

*   **Amaç:** `CSoundData` sınıfı, tek bir ses dosyasının (örneğin .wav, .mp3) ham verilerini, meta verilerini (boyut, çalma süresi) ve bu verilere erişimi yönetir. Ses dosyalarını diskten veya `EterPack` paketlerinden okuyabilir, Miles Sound System (MSS) aracılığıyla farklı formatları (PCM WAV, ADPCM WAV, MPEG Layer 3) dekompres edebilir ve dekompres edilmiş ham ses verilerini bellekte tutar. Ayrıca, yüklenen ses verileri için referans sayımı (`m_iRefCount`) ve son erişim zamanı (`m_dwAccessTime`) gibi bilgileri de yönetir.

*   **`SoundData.h` - Temel Tanımlar ve Bildirimler:**
    *   **Enum:**
        *   `FLAG_DATA_SIZE`: Ses verisinin başında boyut bilgisi olup olmadığını belirten bir bayrak.
        *   `SOUND_FILE_MAX_NUM`: Aynı anda EterPack üzerinden AIL callback'leri ile okunabilecek maksimum dosya sayısı (varsayılan 5).
    *   **Statik Metotlar:**
        *   `SetPackMode()`: MSS'nin dosya okuma işlemlerini `EterPack` sistemi üzerinden yapması için özel AIL callback fonksiyonlarını (aşağıda listelenen `open_callback` vb.) ayarlar.
    *   **Üye Değişkenler:**
        *   `m_filename[128]` (char): Ses dosyasının adı.
        *   `m_iRefCount` (int): Bu ses verisine kaç aktif referans olduğunu sayar. Verinin ne zaman bellekten silinebileceğini belirlemek için kullanılır.
        *   `m_dwAccessTime` (DWORD): Ses verisine en son ne zaman erişildiğini gösteren zaman damgası (milisaniye cinsinden).
        *   `m_dwPlayTime` (DWORD): Sesin toplam çalma süresi (milisaniye cinsinden).
        *   `m_size` (ULONG): Dekompres edilmiş ham ses verisinin boyutu (byte cinsinden).
        *   `m_data` (LPVOID): Dekompres edilmiş ham ses verisine işaretçi.
        *   `m_flag` (long): `FLAG_DATA_SIZE` gibi bayrakları tutar.
        *   `m_assigned` (bool): Bu `CSoundData` nesnesine bir dosya atanıp atanmadığını belirtir.
    *   **Statik AIL Callback Fonksiyon Bildirimleri (private):**
        *   `open_callback()`: `EterPack`'ten dosya açmak için.
        *   `close_callback()`: Açılan dosyayı kapatmak için.
        *   `seek_callback()`: Dosya içinde belirli bir konuma gitmek için.
        *   `read_callback()`: Dosyadan veri okumak için.
    *   **Statik Yardımcı Fonksiyon Bildirimleri (private):**
        *   `isSlotIndex()`: Verilen indeksin geçerli bir `ms_SoundFile` slotu olup olmadığını kontrol eder.
        *   `GetEmptySlotIndex()`: `ms_SoundFile` dizisinde boş bir slot indeksi bulur.
    *   **Statik Üye Değişkenler (private):**
        *   `ms_isSoundFile[SOUND_FILE_MAX_NUM]` (bool): `EterPack` callback'leri için kullanılan dosya slotlarının dolu/boş durumunu tutar.
        *   `ms_SoundFile[SOUND_FILE_MAX_NUM]` (CMappedFile): `EterPack` callback'leri için `CMappedFile` nesnelerini tutan bir dizi.
    *   **Temel Metotlar:**
        *   `Assign(const char* filename)`: Sınıfa bir dosya adı atar.
        *   `Get()`: Ses verisine işaretçi döndürür, referans sayısını artırır. Eğer veri yüklenmemişse `ReadFromDisk()` çağırır.
        *   `GetSize()`: Ses verisinin boyutunu döndürür.
        *   `Release()`: Referans sayısını azaltır.
        *   `GetAccessTime()`, `GetFileName()`, `SetPlayTime()`, `GetPlayTime()`.

*   **`SoundData.cpp` - Uygulama Detayları:**
    *   **`ReadFromDisk()`:**
        *   `AIL_file_read(m_filename, FILE_READ_WITH_SIZE)` ile dosyayı okur. `FILE_READ_WITH_SIZE` bayrağı, dönen verinin başında dosya boyutunun (S32 olarak) bulunacağını belirtir.
        *   `AIL_file_type()` ile dosya formatını (PCM WAV, ADPCM WAV, MP3) belirler.
        *   **Formatlara Göre İşleme:**
            *   `AILFILETYPE_PCM_WAV`: Veri zaten ham PCM formatındadır. `m_data` doğrudan okunan tampona işaret eder, `m_size` baştaki boyut bilgisinden alınır ve `m_flag`'a `FLAG_DATA_SIZE` eklenir.
            *   `AILFILETYPE_ADPCM_WAV`: `AIL_WAV_info()` ile bilgi alınır ve `AIL_decompress_ADPCM()` ile ses verisi ham PCM formatına dekompres edilir. Orijinal sıkıştırılmış veri (`s`) `AIL_mem_free_lock()` ile serbest bırakılır.
            *   `AILFILETYPE_MPEG_L3_AUDIO` (MP3): `AIL_decompress_ASI()` ile ses verisi ham PCM formatına dekompres edilir. Orijinal sıkıştırılmış veri serbest bırakılır.
        *   Bilinmeyen formatlar için hata verir.
    *   **`EterPack` Entegrasyonu için AIL Callback Fonksiyonları:**
        *   `SetPackMode()`: `AIL_set_file_callbacks()` fonksiyonunu çağırarak MSS'nin dosya G/Ç işlemlerini aşağıdaki callback'ler üzerinden yapmasını sağlar.
        *   `open_callback()`: `CEterPackManager::Instance().Get()` kullanarak `EterPack`'ten dosyayı açar (`CMappedFile` olarak). Boş bir `ms_SoundFile` slotu bulur ve dosya handle'ı olarak bu slotun indeksini döndürür.
        *   `close_callback()`: İlgili `CMappedFile` nesnesini (`ms_SoundFile[file_handle].Destroy()`) kapatır ve slotu boş olarak işaretler.
        *   `seek_callback()`: `CMappedFile::Seek()` kullanarak paket içindeki dosyada gezinir.
        *   `read_callback()`: `CMappedFile::Read()` kullanarak paket içindeki dosyadan veri okur.
    *   **Yapıcı (`CSoundData()`):** Üye değişkenleri başlangıç değerlerine ayarlar (referans sayısı 0, veri NULL vb.).
    *   **Yıkıcı (`~CSoundData()`):** `Destroy()` metodunu çağırır.
    *   **`Destroy()`:** Eğer `m_data` (ham ses verisi) yüklenmişse, `AIL_mem_free_lock(m_data)` ile Miles tarafından ayrılan belleği serbest bırakır.
    *   **`Get()`:** Referans sayısını artırır, son erişim zamanını günceller. Eğer `m_data` NULL ise (yani veri henüz yüklenmemişse) `ReadFromDisk()` çağırarak yükler. Eğer `FLAG_DATA_SIZE` ayarlıysa, verinin başındaki boyut bilgisini atlayarak gerçek ses verisine işaretçiyi döndürür.
    *   **`Release()`:** Referans sayısını azaltır, son erişim zamanını günceller.

*   **Kullanım Amacı:**
    *   `CSoundBase::ms_dataMap` içinde, dosya CRC'si ile eşleştirilerek saklanır.
    *   Bir ses efekti veya müzik çalınacağı zaman, ilgili `CSoundData` nesnesi bu haritadan alınır ve `Get()` metodu ile ham ses verisine erişilir.
    *   Referans sayımı, aynı ses verisinin birden fazla ses örneği (sound instance) tarafından aynı anda kullanılmasına ve yalnızca tüm referanslar bittiğinde bellekten silinmesine olanak tanır.
    *   `SetPackMode()` ve AIL callback'leri, oyunun varlık dosyalarının paketlenmiş (packed) formatta dağıtılmasını ve ses dosyalarının bu paketlerden şeffaf bir şekilde okunmasını sağlar.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h`).
    *   `EterPackManager` (`EterPack` kütüphanesinden).
    *   `CMappedFile` (`EterBase` kütüphanesinden).
    *   `ELTimer_GetMSec` (`EterBase` zamanlayıcı fonksiyonu).
    *   Standart C++ kütüphaneleri (`string.h` için `strncpy`).

### `SoundInstance.h`, `SoundInstance2D.cpp`, `SoundInstance3D.cpp`, `SoundInstanceStream.cpp` (Ses Örneği Sınıfları)

*   **Genel Amaç:** Bu dosyalar, çalınacak olan her bir aktif sesin (efekt veya müzik) bir örneğini temsil eden sınıfları tanımlar. `SoundInstance.h` içinde tanımlanan `ISoundInstance` soyut arayüzü, tüm ses örneklerinin sahip olması gereken temel işlevleri (çal, durdur, ses ayarı vb.) belirtir. Bu arayüzden türeyen `CSoundInstance2D`, `CSoundInstance3D` ve `CSoundInstanceStream` sınıfları ise sırasıyla 2D sesler, 3D pozisyonel sesler ve akıtılan (streamed) sesler için özelleşmiş implementasyonlar sunar.

*   **`SoundInstance.h` - Arayüz ve Sınıf Bildirimleri:**
    *   **`ISoundInstance` (Soyut Sınıf):**
        *   `CSoundBase`'den miras alır (ancak `SoundBase.h` doğrudan dahil edilmemiş, muhtemelen `Stdafx.h` üzerinden geliyor).
        *   **Saf Sanal (Pure Virtual) Metotlar:**
            *   `Initialize()`: Ses örneğini kullanıma hazırlar (örn: Miles handle ayırma).
            *   `Destroy()`: Ayrılan kaynakları serbest bırakır.
            *   `SetSound(CSoundData* pSound)`: Çalınacak ses verisini atar.
            *   `Play(int iLoopCount = 1, DWORD dwPlayCycleTimeLimit = 0)`: Sesi çalar. `iLoopCount` tekrar sayısını, `dwPlayCycleTimeLimit` ise aynı sesin ne kadar sürede bir tekrar çalınabileceğini belirler (kısa süreli tekrarları engellemek için).
            *   `Pause()`: Sesi duraklatır.
            *   `Resume()`: Duraklatılmış sesi devam ettirir.
            *   `Stop()`: Sesi tamamen durdurur ve başa sarar.
            *   `GetVolume(float& rfVolume)`: Mevcut ses seviyesini alır.
            *   `SetVolume(float volume)`: Ses seviyesini ayarlar (0.0 - 1.0).
            *   `IsDone()`: Sesin çalması bitmiş mi kontrol eder.
            *   `SetPosition(float x, float y, float z)`: (3D sesler için) ses kaynağının pozisyonunu ayarlar.
            *   `SetOrientation(float x_face, ...)`: (3D sesler için) ses kaynağının yönelimini ayarlar.
            *   `SetVelocity(float x, float y, float z, float fMagnitude)`: (3D sesler için) ses kaynağının hızını ayarlar.
    *   **`CSoundInstance2D` (Somut Sınıf):**
        *   `ISoundInstance`'dan miras alır.
        *   Üyeler: `HSAMPLE m_sample` (Miles 2D sample handle), `CSoundData* m_pSoundData`.
        *   2D sesler için `ISoundInstance` metotlarını implemente eder. 3D'ye özgü metotlar genellikle `assert` ile işaretlenir veya boş bırakılır.
    *   **`CSoundInstance3D` (Somut Sınıf):**
        *   `ISoundInstance`'dan miras alır.
        *   Üyeler: `H3DSAMPLE m_sample` (Miles 3D sample handle), `CSoundData* m_pSoundData`.
        *   3D pozisyonel sesler için `ISoundInstance` metotlarını implemente eder. `SetOrientation` implemente edilmemiş olabilir.
        *   Ek olarak `UpdatePosition(float fElapsedTime)` metodu bildirilir (pozisyon güncellemesi için).
    *   **`CSoundInstanceStream` (Somut Sınıf):**
        *   `ISoundInstance`'dan miras alır.
        *   Üyeler: `HSTREAM m_stream` (Miles stream handle).
        *   Akıtılan sesler (müzik vb.) için `ISoundInstance` metotlarını implemente eder. `SetSound` genellikle doğrudan kullanılmaz. 3D'ye özgü metotlar boş bırakılır.
        *   Ek olarak `SetStream(HSTREAM stream)` ve `IsData()` (stream handle'ının geçerli olup olmadığını kontrol eder) metotları bildirilir.

*   **`SoundInstance2D.cpp` - `CSoundInstance2D` Uygulaması:**
    *   `Initialize()`: `AIL_allocate_sample_handle(ms_DIGDriver)` ile bir 2D sample handle alır.
    *   `Destroy()`: Varsa `m_pSoundData`'yı `Release()` eder ve `AIL_release_sample_handle(m_sample)` ile Miles handle'ını serbest bırakır.
    *   `SetSound(CSoundData* pSoundData)`: Verilen `CSoundData`'dan ses verisini alır (`pSoundData->Get()`). `AIL_init_sample()` ve `AIL_set_sample_file()` ile sesi Miles sample'ına yükler. Önceki `m_pSoundData` varsa onu `Release()` eder ve yeni `pSoundData`'yı saklar.
    *   `Play(int iLoopCount, ...)`: `AIL_set_sample_loop_count()` ve `AIL_start_sample()` çağırır.
    *   `Pause()`: `AIL_stop_sample()` çağırır (Miles'ta pause için stop kullanılır, resume ile devam eder).
    *   `Resume()`: `AIL_resume_sample()` çağırır.
    *   `Stop()`: `AIL_end_sample()` çağırır (sample handle'ı geçersiz kılar, tekrar kullanmak için yeniden initialize/set etmek gerekir). `m_sample`'ı NULL yapar.
    *   `SetVolume(float volume)`: `AIL_set_sample_volume_pan()` ile sesi ve pan'ı (burada pan 0.5 - merkez) ayarlar.
    *   `IsDone()`: `AIL_sample_status(m_sample) == SMP_DONE` kontrolü yapar.

*   **`SoundInstance3D.cpp` - `CSoundInstance3D` Uygulaması:**
    *   `Initialize()`: `AIL_allocate_3D_sample_handle(ms_pProviderDefault->hProvider)` ile bir 3D sample handle alır.
    *   `Destroy()`: `m_pSoundData`'yı `Release()` eder ve `AIL_release_3D_sample_handle(m_sample)` çağırır.
    *   `SetSound(CSoundData* pSoundData)`: `pSoundData->Get()` ile ses verisini alır. `AIL_set_3D_sample_file()` ile sesi Miles 3D sample'ına yükler. Önceki `m_pSoundData`'yı `Release()` eder. Başlangıç pozisyonunu (0,0,0) ayarlar ve otomatik pozisyon güncellemeyi kapatır (`AIL_auto_update_3D_position(m_sample, 0)`).
    *   `Play(int iLoopCount, DWORD dwPlayCycleTimeLimit)`: `ELTimer_GetMSec()` ile zaman kontrolü yapar. Eğer `dwPlayCycleTimeLimit` dolmamışsa çalmaz. `AIL_set_3D_sample_loop_count()` ve `AIL_start_3D_sample()` çağırır.
    *   `Pause()`: `AIL_stop_3D_sample()`.
    *   `Resume()`: `AIL_resume_3D_sample()`.
    *   `Stop()`: `AIL_end_3D_sample()`.
    *   `SetVolume(float volume)`: `AIL_set_3D_sample_volume()`.
    *   `IsDone()`: `AIL_3D_sample_status(m_sample) == SMP_DONE`.
    *   `SetPosition(float x, float y, float z)`: `AIL_set_3D_position(m_sample, x, y, -z)` (Z ekseni ters çevrilmiş).
    *   `SetVelocity(float fx, ...)`: `AIL_set_3D_velocity()` ve ardından `AIL_auto_update_3D_position(m_sample, 1)` ile otomatik güncellemeyi açar.
    *   `UpdatePosition(float fElapsedTime)`: `AIL_update_3D_position(m_sample, fElapsedTime)` çağırır.

*   **`SoundInstanceStream.cpp` - `CSoundInstanceStream` Uygulaması:**
    *   `Initialize()`: Genellikle boştur, stream açma işlemi `SoundManagerStream` tarafından yapılır.
    *   `Destroy()`: `AIL_close_stream(m_stream)` çağırır.
    *   `SetStream(HSTREAM stream)`: Dışarıdan (genellikle `CSoundManagerStream`'den) alınan `HSTREAM` handle'ını saklar.
    *   `Play(int count, ...)`: `AIL_set_stream_loop_count()` ve `AIL_start_stream()`.
    *   `Pause()`: `AIL_pause_stream(m_stream, 1)`.
    *   `Resume()`: `AIL_pause_stream(m_stream, 0)`.
    *   `Stop()`: `AIL_close_stream(m_stream)` ve `m_stream`'i NULL yapar.
    *   `SetVolume(float volume)`: `AIL_set_stream_volume_levels(m_stream, volume, volume)` (sol ve sağ kanalı aynı ayarlar).
    *   `IsDone()`: `AIL_stream_status(m_stream) == -1` (Miles'ta stream bitince veya hata olunca -1 döner).
    *   `IsData()`: `m_stream` handle'ının geçerli olup olmadığını kontrol eder.
    *   `SetSound()` metodu `true` döner ama bir işlem yapmaz.
    *   3D ile ilgili metotlar boştur.

*   **Kullanım Amacı:**
    *   Bu sınıfların örnekleri, `CSoundManager` (veya 2D/3D/Stream türevleri) tarafından oluşturulur ve yönetilir.
    *   Bir ses çalınmak istendiğinde, uygun tipte bir `ISoundInstance` nesnesi alınır, `SetSound()` ile ses verisi atanır ve `Play()` ile çalınır.
    *   3D sesler için pozisyon ve hız bilgileri oyun dünyasındaki nesnelerin konumlarına göre güncellenir.
    *   Stream'ler genellikle arka plan müzikleri veya uzun ortam sesleri için kullanılır.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h`).
    *   `SoundBase.h` (ve dolayısıyla `SoundData.h`).
    *   `Stdafx.h` (genel başlıklar ve MSS için).
    *   `CSoundManager2D.h`, `CSoundManager3D.h` (ilgili .cpp dosyalarında `Initialize` ve `SetSound` gibi fonksiyonlarda `ms_DIGDriver` ve `ms_pProviderDefault` gibi statik üyelere erişim için dahil edilmiş olabilirler, ancak bu statik üyeler `CSoundBase`'den gelmektedir).
    *   `EterBase/Timer.h` (`ELTimer_GetMSec` için `SoundInstance3D.cpp`'de).

### `SoundManager.h` ve `SoundManager.cpp` (`CSoundManager` Sınıfı)

*   **Genel Amaç:** `CSoundManager` sınıfı, bir singleton olarak, `MilesLib` kütüphanesinin genel ses yönetimi arayüzünü sunar. 2D ses efektleri, 3D pozisyonel sesler ve müzik akışları (streaming) için ayrı ayrı yönetici sınıfları (`CSoundManager2D`, `CSoundManager3D`, `CSoundManagerStream`) içerir ve bu alt sistemler üzerinden tüm ses işlemlerini koordine eder. Ses seviyelerini (genel ses, müzik), dinleyici pozisyonunu ve yönelimini yönetir, müzikler için fade-in/fade-out efektleri sağlar ve belirli seslerin çok sık çalınmasını önlemek için bir frekans kontrolü uygular.

*   **`SoundManager.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıf:** `CSingleton<CSoundManager>`.
    *   **Enum (`EMusicState`):**
        *   `MUSIC_STATE_OFF`: Müzik çalmıyor.
        *   `MUSIC_STATE_PLAY`: Müzik normal çalıyor.
        *   `MUSIC_STATE_FADE_IN`: Müzik sesi yavaşça artarak başlıyor.
        *   `MUSIC_STATE_FADE_OUT`: Müzik sesi yavaşça azalarak bitiyor.
        *   `MUSIC_STATE_FADE_LIMIT_OUT`: Müzik sesi belirli bir limite kadar yavaşça azalıyor.
    *   **Struct (`SMusicInstance` -> `TMusicInstance`):** Her bir müzik akışı örneğinin durumunu tutar:
        *   `dwMusicFileNameCRC` (DWORD): Çalan müzik dosyasının CRC32 değeri.
        *   `MusicState` (EMusicState): Müziğin mevcut fade durumu.
        *   `fVolume` (float): Müziğin anlık ses seviyesi.
        *   `fLimitVolume` (float): `FADE_LIMIT_OUT` için hedef ses seviyesi.
        *   `fVolumeSpeed` (float): Fade işlemleri için ses seviyesi değişim hızı.
    *   **Statik Üye Değişkenler:**
        *   `ms_SoundManager2D` (CSoundManager2D): 2D ses yöneticisi örneği.
        *   `ms_SoundManager3D` (CSoundManager3D): 3D ses yöneticisi örneği.
        *   `ms_SoundManagerStream` (CSoundManagerStream): Müzik/ses akışı yöneticisi örneği.
    *   **Üye Değişkenler:**
        *   `m_bInitialized` (BOOL): Ses yöneticisinin başlatılıp başlatılmadığı.
        *   `m_isSoundDisable` (BOOL): Seslerin geçici olarak devre dışı bırakılıp bırakılmadığı (örn: oyun arka plana alındığında).
        *   `m_fxPosition`, `m_fyPosition`, `m_fzPosition` (float): Dinleyicinin (genellikle oyuncunun kamerası) 3D uzaydaki pozisyonu.
        *   `m_fSoundScale`, `m_fAmbienceSoundScale` (float): 3D seslerin ve ortam seslerinin uzaklıkla ne kadar zayıflayacağını belirleyen ölçek faktörleri.
        *   `m_fSoundVolume`, `m_fMusicVolume` (float): Genel ses efekti ve müzik ses seviyeleri (0.0 - 1.0).
        *   `m_fBackupMusicVolume`, `m_fBackupSoundVolume` (float): Sesler devre dışı bırakıldığında önceki ses seviyelerini saklamak için.
        *   `m_MusicInstances[CSoundManagerStream::MUSIC_INSTANCE_MAX_NUM]` (TMusicInstance dizisi): Aktif müzik akışlarının durumlarını saklar.
        *   `m_PlaySoundHistoryMap` (`std::map<std::string, float>`): Belirli ses dosyalarının en son ne zaman çalındığını (zaman damgası) saklar (frekans kontrolü için).
    *   **Temel Metot Bildirimleri:**
        *   `Create()`: Alt ses yöneticilerini (`2D`, `3D`, `Stream`) başlatır.
        *   `Destroy()`: Alt ses yöneticilerini yok eder.
        *   `SetPosition()`, `SetDirection()`: 3D dinleyici pozisyonunu ve yönelimini ayarlar.
        *   `Update()`: Müzik fade işlemlerini günceller ve 3D dinleyici pozisyonunu (0,0,0 - göreceli koordinatlar için) ayarlar.
        *   Ses Seviyesi Yönetimi: `SetSoundVolume()`, `SetMusicVolume()`, `SaveVolume()`, `RestoreVolume()`, `GetSoundVolume()`, `GetMusicVolume()`.
        *   Ses Çalma: `PlaySound2D()`, `PlaySound3D()`, `PlayAmbienceSound3D()`, `PlayCharacterSound3D()`.
        *   Müzik Yönetimi: `PlayMusic()`, `FadeInMusic()`, `FadeOutMusic()`, `FadeLimitOutMusic()`, `FadeOutAllMusic()`.
        *   `UpdateSoundInstance()`: (Muhtemelen animasyon sisteminden gelen) belirli bir karede çalınacak sesleri yönetir.

*   **`SoundManager.cpp` - Uygulama Detayları:**
    *   **Yapıcı (`CSoundManager()`):** Temel değişkenleri başlangıç değerlerine (ses seviyeleri 1.0, pozisyon 0,0,0 vb.) ayarlar. Müzik örneklerinin durumunu `MUSIC_STATE_OFF` yapar.
    *   **`Create()` / `Destroy()`:** İlgili alt ses yöneticilerinin `Initialize()` ve `Destroy()` metotlarını çağırır.
    *   **`Update()`:**
        *   `ms_SoundManager3D.SetListenerPosition(0.0f, 0.0f, 0.0f)`: Dinleyici pozisyonunu (0,0,0) olarak ayarlar. Bu, `PlaySound3D` gibi fonksiyonlarda ses kaynağı pozisyonlarının dinleyiciye göreceli olarak hesaplanacağı anlamına gelir.
        *   `m_MusicInstances` dizisini tarayarak aktif müziklerin fade durumlarını günceller. Ses seviyelerini `rMusicInstance.fVolumeSpeed` oranında artırır/azaltır ve hedef seviyeye ulaşıldığında durumu değiştirir veya müziği durdurur.
    *   **Ses Seviyesi Ayarları:**
        *   `__ConvertRatioVolumeToApplyVolume()` ve `__ConvertGradeVolumeToApplyVolume()`: Kullanıcı arayüzünden gelen oransal (0.0-1.0) veya kademeli (örn: 0-5) ses ayarlarını, Miles Sound System'in daha logaritmik bir tepki vermesi için dönüştürür (genellikle `pow(10.0f, (-1.0f + fRatioVolume))` formülü kullanılır).
        *   `SetSoundVolume()` ve `__SetMusicVolume()`: Ayarlanan ses seviyesini ilgili değişkende saklar ve müzik için aktif tüm stream'lerin sesini günceller.
        *   `SaveVolume()` / `RestoreVolume()`: Oyun odağını kaybettiğinde/kazandığında sesleri kapatıp/açmak için kullanılır. Mevcut ses seviyelerini yedekler, sesi sıfırlar ve `m_isSoundDisable` bayrağını ayarlar.
    *   **Ses Çalma Metotları:**
        *   `PlaySound2D()`: `ms_SoundManager2D.GetInstance()` ile bir 2D ses örneği alır, ses seviyesini ayarlar ve çalar.
        *   `PlaySound3D()` / `PlayAmbienceSound3D()`: `ms_SoundManager3D.SetInstance()` ile bir 3D ses örneği alır/oluşturur. Ses kaynağının pozisyonunu dinleyici pozisyonuna (`m_fxPosition` vb.) ve ses ölçeğine (`m_fSoundScale` veya `m_fAmbienceSoundScale`) göre normalize ederek ayarlar, sesi ayarlar ve çalar.
        *   `PlayCharacterSound3D()`: `PlaySound3D`'ye benzer, ancak ek olarak `bCheckFrequency` bayrağı `TRUE` ise `m_PlaySoundHistoryMap` kullanarak aynı sesin çok kısa aralıklarla (varsayılan 0.3 saniye) çalınmasını engeller. Ayrıca, belirli bir mesafeden (`s_fLimitDistance`) daha uzaktaki sesleri çalmaz.
    *   **Müzik Yönetimi Metotları:**
        *   `PlayMusic(dwIndex, filename, volume, speed)`: Belirtilen indeksteki müzik stream'ini ayarlar, sesini ve çalma durumunu `TMusicInstance` yapısında günceller.
        *   `FadeInMusic()`: Boş bir müzik slotu bulur veya mevcut bir müziği durdurup üzerine yazar. Müziği düşük sesle başlatır ve `MUSIC_STATE_FADE_IN` durumuna geçirir.
        *   `FadeOutMusic()` / `FadeLimitOutMusic()` / `FadeOutAllMusic()`: İlgili müzik(ler)in durumunu `MUSIC_STATE_FADE_OUT` veya `MUSIC_STATE_FADE_LIMIT_OUT` olarak ayarlar ve `Update()` döngüsünde seslerinin yavaşça azalmasını sağlar.
        *   `GetMusicIndex()`: Verilen dosya adına göre aktif müzik listesinde arama yaparak indeksini bulur.
    *   `UpdateSoundInstance()`: Bu metotlar, genellikle karakter animasyonları veya efektlerle senkronize sesleri çalmak için kullanılır. Bir `TSoundInstanceVector` (muhtemelen animasyon verisinden gelir) içindeki sesleri, mevcut kare (`dwcurFrame`) ile eşleşiyorsa çalar. 3D versiyonu, sesin pozisyonunu da dikkate alır.

*   **Kullanım Amacı:**
    *   Oyunun herhangi bir yerinden `CSoundManager::Instance().PlaySound2D("effect.wav")` gibi çağrılarla kolayca ses çalınmasını sağlar.
    *   Arka plan müziklerini ve ortam seslerini yönetir.
    *   Ses ayarları menüsü gibi kullanıcı arayüzü elemanları, bu sınıfın ses seviyesi ayarlama fonksiyonlarını çağırır.
    *   Karakterlerin hareketleri, yetenekleri veya çevresel olaylarla ilişkili 3D seslerin doğru pozisyonlarda ve doğru zamanlarda çalınmasını koordine eder.

*   **Bağımlılıklar:**
    *   Miles Sound System API (alt yöneticiler aracılığıyla).
    *   `SoundManagerStream.h`, `SoundManager2D.h`, `SoundManager3D.h`.
    *   `SoundInstance.h` (arayüz için).
    *   `Type.h` (muhtemelen `NSound::TSoundInstanceVector` gibi yapılar için).
    *   `EterBase/Singleton.h`, `EterBase/Timer.h`.
    *   Standart C++ kütüphaneleri (`map`, `string`, `math.h`).

### `SoundManager2D.h` ve `SoundManager2D.cpp` (`CSoundManager2D` Sınıfı)

*   **Amaç:** `CSoundManager2D` sınıfı, `CSoundBase`'den miras alarak, pozisyonel olmayan 2D ses efektlerinin yönetimi için özelleşmiştir. Sabit sayıda (`INSTANCE_MAX_COUNT`, varsayılan 4) `CSoundInstance2D` örneğini bir havuzda tutar ve bu havuzdaki boş (çalması bitmiş) bir örneği kullanarak yeni ses efektlerini çalar. Bu, kaynakların verimli kullanılmasını ve aynı anda çalabilecek 2D ses sayısının sınırlanmasını sağlar.

*   **`SoundManager2D.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıf:** `CSoundBase`.
    *   **Enum:**
        *   `INSTANCE_MAX_COUNT`: Havuzda tutulacak maksimum `CSoundInstance2D` sayısı (varsayılan 4).
    *   **Üye Değişkenler (protected):**
        *   `ms_Instances[INSTANCE_MAX_COUNT]` (CSoundInstance2D dizisi): 2D ses örneklerini tutan havuz.
    *   **Metot Bildirimleri:**
        *   `Initialize()`: `CSoundBase::Initialize()`'ı çağırır, Miles dijital ses sürücüsünü (`ms_DIGDriver`) açar (eğer daha önce açılmamışsa) ve havuzdaki tüm `CSoundInstance2D` örneklerini başlatır.
        *   `Destroy()`: Havuzdaki tüm `CSoundInstance2D` örneklerini yok eder, `ms_DIGDriver`'ı kapatır ve `CSoundBase::Destroy()`'ı çağırır.
        *   `GetInstance(const char* filename)`: Belirtilen dosya adı için çalmaya uygun bir `ISoundInstance` (aslında `CSoundInstance2D*`) döndürür.

*   **`SoundManager2D.cpp` - Uygulama Detayları:**
    *   **`Initialize()`:**
        *   Önce `CSoundBase::Initialize()` çağrılır.
        *   Eğer `ms_DIGDriver` (statik, `CSoundBase`'den gelir) zaten ayarlanmışsa, tekrar açmaz.
        *   `AIL_open_digital_driver(44100, 16, 2, 0)` ile 44.1 kHz, 16 bit, stereo dijital ses sürücüsü açılır ve `ms_DIGDriver`'a atanır.
        *   `ms_Instances` dizisindeki her bir `CSoundInstance2D` için `Initialize()` çağrılır.
    *   **`Destroy()`:**
        *   `ms_Instances` dizisindeki her bir `CSoundInstance2D` için `Destroy()` çağrılır.
        *   `AIL_close_digital_driver(ms_DIGDriver)` ile sürücü kapatılır ve `ms_DIGDriver` NULL yapılır.
        *   `CSoundBase::Destroy()` çağrılır.
    *   **`GetInstance(const char* c_pszFileName)`:**
        *   Dosya adının CRC'si (`dwFileCRC`) hesaplanır.
        *   `CSoundBase::ms_dataMap`'te bu CRC ile eşleşen `CSoundData` aranır.
        *   Bulunamazsa, `CSoundBase::AddFile()` ile yeni bir `CSoundData` oluşturulur ve `ms_dataMap`'e eklenir.
        *   `ms_Instances` havuzunda çalması bitmiş (`IsDone() == true`) bir `CSoundInstance2D` aranır. Arama, bir önceki aramada kalınan yerden (`static DWORD k`) devam eder (round-robin benzeri bir yaklaşım).
        *   Boş bir örnek bulunursa, `pkInst->SetSound(pkSoundData)` ile yeni ses verisi bu örneğe atanır ve örnek döndürülür.
        *   Boş bir örnek bulunamazsa `NULL` döndürülür (bu durumda ses çalınamaz).

*   **Kullanım Amacı:**
    *   Kullanıcı arayüzü sesleri, kısa oyun içi efektler gibi pozisyona bağlı olmayan seslerin çalınması için kullanılır.
    *   `CSoundManager`, `PlaySound2D` metodunda bu sınıfın `GetInstance` metodunu çağırarak bir ses örneği alır ve çalar.
    *   Havuzlama mekanizması, sıkça çalınan kısa sesler için performanslı bir çözüm sunar.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h`).
    *   `SoundBase.h`, `SoundInstance.h` (`CSoundInstance2D` için).
    *   `Stdafx.h`.

### `SoundManager3D.h` ve `SoundManager3D.cpp` (`CSoundManager3D` Sınıfı)

*   **Amaç:** `CSoundManager3D` sınıfı, `CSoundBase`'den miras alarak, 3D pozisyonel ses efektlerinin yönetimi için özelleşmiştir. Miles Sound System'in 3D yeteneklerini kullanarak, ses kaynaklarının ve dinleyicinin (listener) 3D uzaydaki pozisyonlarına, yönelimlerine ve hızlarına göre seslerin duyulmasını sağlar. Sabit sayıda (`INSTANCE_MAX_COUNT`, varsayılan 32) `CSoundInstance3D` örneğini bir havuzda tutar ve bu havuzdaki boş bir örneği kullanarak yeni 3D ses efektlerini çalar. Ayrıca, belirli ses örneklerini geçici olarak kilitleme/açma mekanizması sunar.

*   **`SoundManager3D.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıf:** `CSoundBase`.
    *   **Enum'lar:**
        *   `INSTANCE_MAX_COUNT`: Havuzda tutulacak maksimum `CSoundInstance3D` sayısı (varsayılan 32).
        *   `MAX_PROVIDERS`: Sorgulanacak maksimum Miles 3D ses sağlayıcı sayısı (varsayılan 32).
    *   **Üye Değişkenler (protected):**
        *   `m_bLockingFlag[INSTANCE_MAX_COUNT]` (bool dizisi): Havuzdaki her bir ses örneğinin kilitli olup olmadığını belirtir. Kilitli örnekler `SetInstance` tarafından yeni sesler için seçilmez.
        *   `m_Instances[INSTANCE_MAX_COUNT]` (CSoundInstance3D dizisi): 3D ses örneklerini tutan havuz.
        *   `m_pListener` (H3DPOBJECT): Miles 3D dinleyici nesnesine handle. Genellikle oyuncunun kamera pozisyonunu temsil eder.
        *   `m_bInit` (bool): `CSoundManager3D`'nin başarıyla başlatılıp başlatılmadığını gösterir.
    *   **Metot Bildirimleri:**
        *   `Initialize()`: `CSoundBase::Initialize()`'ı çağırır. Uygun bir Miles 3D ses sağlayıcısı (özellikle "Miles Fast 2D Positional Audio") arar ve açar. Bir 3D dinleyici (`m_pListener`) oluşturur ve havuzdaki tüm `CSoundInstance3D` örneklerini başlatır.
        *   `Destroy()`: Havuzdaki tüm örnekleri ve dinleyiciyi yok eder, 3D sağlayıcıyı kapatır ve `CSoundBase::Destroy()`'ı çağırır.
        *   `SetInstance(const char* c_szFileName)`: Belirtilen dosya adı için çalmaya uygun bir `CSoundInstance3D` örneğinin havuzdaki indeksini döndürür. Boş (çalması bitmiş ve kilitli olmayan) bir örnek arar.
        *   `GetInstance(DWORD dwIndex)`: Verilen indeksteki `ISoundInstance`'ı (aslında `CSoundInstance3D*`) döndürür.
        *   Dinleyici Kontrolleri: `SetListenerDirection()`, `SetListenerPosition()`, `SetListenerVelocity()`.
        *   Örnek Kilitleme: `Lock(int iIndex)`, `Unlock(int iIndex)`.

*   **`SoundManager3D.cpp` - Uygulama Detayları:**
    *   **`Initialize()`:**
        *   `CSoundBase::Initialize()` çağrılır.
        *   Eğer `ms_pProviderDefault` (statik, `CSoundBase`'den) zaten ayarlıysa (yani bir 3D sağlayıcı daha önce açılmışsa) işlemi tamamlar.
        *   `AIL_enumerate_3D_providers()` ile mevcut tüm 3D ses sağlayıcılarını listeler.
        *   Listelenen sağlayıcılar arasında özellikle `"Miles Fast 2D Positional Audio"` adlı sağlayıcıyı arar ve `ms_pProviderDefault` olarak atar.
        *   Eğer varsayılan sağlayıcı bulunamazsa veya `AIL_open_3D_provider()` ile açılamazsa, `CSoundBase::Destroy()` çağırılır ve `false` döndürülür.
        *   `AIL_open_3D_listener()` ile bir dinleyici nesnesi oluşturulur ve `m_pListener`'a atanır.
        *   Dinleyicinin başlangıç pozisyonu (0,0,0) olarak ayarlanır.
        *   Havuzdaki (`m_Instances`) tüm `CSoundInstance3D` örnekleri için `Initialize()` çağrılır ve kilit bayrakları (`m_bLockingFlag`) `false` yapılır.
    *   **`Destroy()`:**
        *   Eğer `m_bInit` `false` ise (yani başlatılmamışsa) bir şey yapmaz.
        *   Havuzdaki tüm örnekleri `Destroy()` eder.
        *   `AIL_close_3D_listener(m_pListener)` ile dinleyiciyi kapatır.
        *   `AIL_close_3D_provider(ms_pProviderDefault->hProvider)` ile 3D sağlayıcıyı kapatır.
        *   `CSoundBase::Destroy()` çağrılır.
    *   **Dinleyici Metotları (`SetListenerDirection`, `SetListenerPosition`, `SetListenerVelocity`):** İlgili AIL 3D fonksiyonlarını (`AIL_set_3D_orientation`, `AIL_set_3D_position`, `AIL_set_3D_velocity`) çağırarak dinleyici özelliklerini ayarlar. Z ekseni genellikle Miles için ters çevrilir (`-fzDir`, `-fZ`).
    *   **`SetInstance(const char* c_pszFileName)`:**
        *   `CSoundManager2D::GetInstance`'a benzer şekilde çalışır: ses dosyasını yükler (`CSoundData`).
        *   Havuzda çalması bitmiş (`IsDone() == true`) **ve kilitli olmayan (`!m_bLockingFlag[index]`)** bir `CSoundInstance3D` arar. Arama, bir önceki aramada kalınan yerden devam eder.
        *   Boş ve kilitli olmayan bir örnek bulunursa, `pkInst->SetSound(pkSoundData)` ile ses verisi atanır ve örneğin indeksi döndürülür.
        *   Bulunamazsa veya `SetSound` başarısız olursa -1 döndürür.
    *   `Lock(int iIndex)` / `Unlock(int iIndex)`: Verilen indeksteki `m_bLockingFlag` değerini ayarlar.

*   **Kullanım Amacı:**
    *   Oyun dünyasındaki karakterlerin yetenek sesleri, ayak sesleri, çevresel efektler gibi kaynağın pozisyonuna göre duyulması gereken tüm sesler için kullanılır.
    *   `CSoundManager`, `PlaySound3D`, `PlayAmbienceSound3D`, `PlayCharacterSound3D` gibi metotlarında bu sınıfın `SetInstance` ve `GetInstance` metotlarını çağırarak 3D ses örnekleri alır ve yönetir.
    *   Dinleyici pozisyonu ve yönelimi, genellikle ana oyun döngüsünde kamera pozisyonuna göre `CSoundManager` tarafından güncellenir.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h`).
    *   `SoundBase.h`, `SoundInstance.h` (`CSoundInstance3D` için).
    *   `Stdafx.h`.

### `SoundManagerStream.h` ve `SoundManagerStream.cpp` (`CSoundManagerStream` Sınıfı)

*   **Amaç:** `CSoundManagerStream` sınıfı, `CSoundBase`'den miras alarak, genellikle müzik gibi uzun ses dosyalarının diskten parça parça okunarak (streaming) çalınmasını yönetir. Sabit sayıda (`MUSIC_INSTANCE_MAX_NUM`, varsayılan 3) `CSoundInstanceStream` örneğini bir dizide tutar. Bu, aynı anda çalınabilecek stream sayısını sınırlar.

*   **`SoundManagerStream.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıf:** `CSoundBase`.
    *   **Enum:**
        *   `MUSIC_INSTANCE_MAX_NUM`: Tutulacak maksimum `CSoundInstanceStream` sayısı (varsayılan 3).
    *   **Üye Değişkenler (protected):**
        *   `m_Instances[MUSIC_INSTANCE_MAX_NUM]` (CSoundInstanceStream dizisi): Stream örneklerini tutan dizi.
    *   **Metot Bildirimleri:**
        *   `Initialize()`: `CSoundBase::Initialize()`'ı çağırır, Miles dijital ses sürücüsünü (`ms_DIGDriver`) açar (eğer daha önce açılmamışsa) ve dizideki tüm `CSoundInstanceStream` örneklerini başlatır.
        *   `Destroy()`: Dizideki tüm `CSoundInstanceStream` örneklerini durdurur (`Stop()`) ve `CSoundBase::Destroy()`'ı çağırır.
        *   `SetInstance(DWORD dwIndex, const char* filename)`: Belirtilen indeksteki stream örneği için verilen dosyayı açar ve Miles `HSTREAM` handle'ını ayarlar.
        *   `GetInstance(DWORD dwIndex)`: Belirtilen indeksteki `CSoundInstanceStream` örneğini döndürür.
        *   `CheckInstanceIndex(DWORD dwIndex)` (protected): Verilen indeksin geçerli olup olmadığını kontrol eder.

*   **`SoundManagerStream.cpp` - Uygulama Detayları:**
    *   **`Initialize()`:**
        *   Önce `CSoundBase::Initialize()` çağrılır.
        *   Eğer `ms_DIGDriver` (statik, `CSoundBase`'den gelir) zaten ayarlanmışsa, tekrar açmaz.
        *   `AIL_open_digital_driver(44100, 16, 2, 0)` ile 44.1 kHz, 16 bit, stereo dijital ses sürücüsü açılır ve `ms_DIGDriver`'a atanır.
        *   `m_Instances` dizisindeki her bir `CSoundInstanceStream` için `Initialize()` çağrılır.
    *   **`Destroy()`:**
        *   `m_Instances` dizisindeki her bir `CSoundInstanceStream` için `Stop()` çağrılarak aktif stream'ler durdurulur.
        *   `CSoundBase::Destroy()` çağrılır.
    *   **`SetInstance(DWORD dwIndex, const char* filename)`:**
        *   `CheckInstanceIndex()` ile indeksin geçerliliği kontrol edilir.
        *   `AIL_open_stream(ms_DIGDriver, filename, 0)` ile belirtilen dosya için bir Miles stream açılır.
        *   Stream başarıyla açılırsa (`hStream != NULL`), `m_Instances[dwIndex].SetStream(hStream)` ile ilgili `CSoundInstanceStream` örneğine bu handle atanır.
        *   Başarılı olursa `true`, olmazsa `false` döner.
    *   **`GetInstance(DWORD dwIndex)`:**
        *   `CheckInstanceIndex()` ile indeksin geçerliliği kontrol edilir.
        *   Geçerliyse `&m_Instances[dwIndex]` adresini, değilse `NULL` döndürür.

*   **Kullanım Amacı:**
    *   Arka plan müziklerinin ve potansiyel olarak uzun ortam seslerinin çalınması için kullanılır.
    *   `CSoundManager`, müzik çalma ve fade işlemleri için bu sınıfın `SetInstance` ve `GetInstance` metotlarını kullanır.
    *   Sınırlı sayıda stream örneği (örneğin, bir ana müzik, bir de geçici özel bölge müziği gibi senaryolar için) tutarak kaynak kullanımını kontrol altında tutar.

*   **Bağımlılıklar:**
    *   Miles Sound System API (`mss.h`).
    *   `SoundBase.h`, `SoundInstance.h` (`CSoundInstanceStream` için).
    *   `Stdafx.h`.

### `Stdafx.h` ve `Stdafx.cpp` (Ön Derlenmiş Başlıklar)

*   **Amaç:** Bu dosyalar, `MilesLib` kütüphanesi için ön derlenmiş başlık (precompiled header - PCH) mekanizmasını oluşturur. Amaçları, sık kullanılan ve nadiren değişen başlık dosyalarını (özellikle Miles Sound System'in ana başlığı `mss.h`, Windows API başlıkları ve `EterBase` gibi diğer temel kütüphane başlıkları) tek bir yerde toplayarak kütüphanenin genel derleme süresini kısaltmaktır.

*   **`Stdafx.h` İçeriği:**
    *   `#pragma once`: Bu başlık dosyasının her derleme birimi (.cpp dosyası) için yalnızca bir kez dahil edilmesini sağlar.
    *   **Derleyici Uyarılarının Devre Dışı Bırakılması:**
        *   `#pragma warning(disable:4786)`: Genellikle STL kullanıldığında, sembol adlarının 255 karakter sınırını aşması durumunda oluşan uyarıyı devre dışı bırakır.
        *   `#pragma warning(disable:4100)`: Fonksiyon parametrelerinin fonksiyon gövdesinde kullanılmaması durumunda oluşan "referanslanmayan resmi parametre" uyarısını devre dışı bırakır.
        *   `#pragma warning(disable:4201)`: İsimsiz struct/union kullanıldığında oluşan uyarıyı (nonstandard extension used : nameless struct/union) devre dışı bırakır (özellikle `mss.h` gibi harici kütüphane başlıkları için gerekebilir).
    *   **Ana Başlık Dosyaları:**
        *   `#include <mss.h>`: Miles Sound System kütüphanesinin ana başlık dosyası. Tüm MSS API fonksiyonlarını, veri yapılarını ve sabitlerini kullanılabilir hale getirir.
        *   `#include <windows.h>`: Temel Windows API fonksiyonları ve veri türleri için gereklidir.
    *   **Proje İçi Başlık Dosyaları:**
        *   `#include "../EterBase/CRC32.h"`: `EterBase` kütüphanesinden CRC32 hesaplama fonksiyonlarını içerir.
        *   `#include "../EterBase/Utils.h"`: `EterBase` kütüphanesinden çeşitli yardımcı fonksiyonları ve makroları içerir.
        *   `#include "../EterBase/Debug.h"`: `EterBase` kütüphanesinden hata ayıklama, loglama (`Logf`, `Tracen` vb.) ve `assert` gibi makroları içerir.
        *   `#include "../UserInterface/Locale_inc.h"`: İstemcinin yerelleştirme (dil, bölge ayarları) ile ilgili tanımlamalarını ve makrolarını içerir.
    *   **Armadillo Nanomite Koruması Makroları (`NANOBEGIN`, `NANOEND`):**
        *   Bu makrolar, `#ifndef NANOBEGIN` kontrolü ile tanımlanır ve hem Borland C++ (`__BORLANDC__`) hem de Microsoft Visual C++ (`__asm _emit`) için farklı assembly direktifleri içerir.
        *   `NANOBEGIN`: `0xEB, 0x03, 0xD6, 0xD7, 0x01` byte dizisini assembly olarak ekler.
        *   `NANOEND`: `0xEB, 0x03, 0xD6, 0xD7, 0x00` byte dizisini assembly olarak ekler.
        *   Bu makrolar, genellikle Armadillo gibi bir yazılım koruma/lisanslama aracı tarafından "nanomite" adı verilen çok küçük, şifrelenmiş veya sanallaştırılmış kod bloklarını işaretlemek için kullanılır. Bu, tersine mühendisliği zorlaştırmak ve kodun kritik bölümlerini korumak amacıyla yapılır.

*   **`Stdafx.cpp` İçeriği:**
    *   `#include "stdafx.h"`: Bu satır, Visual C++ derleyicisinin `Stdafx.h` içinde listelenen tüm başlıkları kullanarak ön derlenmiş başlık dosyasını (`.pch`) oluşturmasını tetikler.
    *   `namespace { char dummy; }; // solve warning lnk4221`: Bu satır, bazı durumlarda bir kütüphane projesi yalnızca başlık dosyalarından veya şablonlardan oluşuyormuş gibi göründüğünde ve derleyici/linker boş bir `.obj` dosyası ürettiğinde ortaya çıkabilen LNK4221 ("boş arşiv üyesi") bağlayıcı uyarısını çözmek için eklenmiş yaygın bir tekniktir. İsimsiz bir namespace içinde dummy bir değişken tanımlayarak, derleyicinin en az bir sembol içeren bir `.obj` dosyası oluşturmasını sağlar.

*   **Kullanım Amacı:**
    *   `MilesLib` kütüphanesi içindeki diğer tüm `.cpp` dosyaları, derleme sürecini hızlandırmak için ilk satırlarında `#include "Stdafx.h"` ifadesini içermelidir.
    *   MSS API'sine ve diğer sık kullanılan temel fonksiyonlara merkezi bir erişim noktası sağlar.
    *   Kod koruma mekanizmaları için gerekli makroları barındırır.

*   **Bağımlılıklar:**
    *   Miles Sound System SDK (özellikle `mss.h` ve ilgili kütüphane dosyaları).
    *   Windows Platform SDK.
    *   `EterBase` kütüphanesi.
    *   `UserInterface` (özellikle `Locale_inc.h`).

### `Type.h` ve `Type.cpp` (`NSound` Namespace)

*   **Amaç:** Bu dosyalar, `NSound` namespace'i altında, genellikle karakter animasyonları veya belirli zamanlamalarla senkronize edilecek ses olaylarını tanımlamak, bu bilgileri metin tabanlı dosyalardan okumak/yazmak ve çalışma zamanında kullanılacak formata dönüştürmek için gerekli veri yapılarını (`TSoundData`, `TSoundInstance`) ve yardımcı fonksiyonları sağlar.

*   **`Type.h` - Veri Yapıları ve Fonksiyon Bildirimleri:**
    *   **Namespace:** `NSound`
    *   **Global Değişken:**
        *   `extern std::string strResult;`: Dosya yükleme/kaydetme gibi işlemlerin sonuç veya hata mesajlarını tutmak için kullanılır.
    *   **Struct'lar:**
        *   **`SSoundData` (typedef `TSoundData`):**
            *   `fTime` (float): Sesin çalmaya başlayacağı zaman (saniye cinsinden).
            *   `strSoundFileName` (std::string): Çalınacak ses dosyasının adı.
            *   `fSoundVolume` (float): Sesin çalınacağı göreceli ses seviyesi (0.0 - 1.0). Bu değer, genel ses ayarlarından bağımsız olarak o anki ses için özel bir seviye belirtebilir.
        *   **`SSoundInstance` (typedef `TSoundInstance`):**
            *   `dwFrame` (DWORD): Sesin çalmaya başlayacağı animasyon karesi numarası.
            *   `strSoundFileName` (std::string): Çalınacak ses dosyasının adı.
            *   `fSoundVolume` (float): Sesin çalınacağı göreceli ses seviyesi.
    *   **Typedef'ler:**
        *   `TSoundDataVector`: `std::vector<TSoundData>`
        *   `TSoundInstanceVector`: `std::vector<TSoundInstance>`
    *   **Fonksiyon Bildirimleri:**
        *   `LoadSoundInformationPiece(const char* c_szFileName, TSoundDataVector& rSoundDataVector, const char* c_szPathHeader = NULL)`: Belirtilen `.mss` (veya benzeri) yapıdaki metin dosyasından ses bilgilerini okuyarak `TSoundDataVector`'ü doldurur. `c_szPathHeader` ile ses dosyası adlarına bir ön ek eklenebilir.
        *   `SaveSoundInformationPiece(const char* c_szFileName, TSoundDataVector& rSoundDataVector)`: Verilen `TSoundDataVector` içeriğini belirtilen dosyaya özel bir formatta kaydeder.
        *   `DataToInstance(const TSoundDataVector& c_rSoundDataVector, TSoundInstanceVector* pSoundInstanceVector)`: Zaman tabanlı `TSoundDataVector`'ü, kare tabanlı `TSoundInstanceVector`'e dönüştürür.
        *   `GetResultString()`: En son işlem sonucunu/hatasını string olarak döndürür.
        *   `SetResultString(const char* c_pszStr)`: İşlem sonucunu/hatasını ayarlar.

*   **`Type.cpp` - Fonksiyon Uygulamaları:**
    *   **`LoadSoundInformationPiece()`:**
        *   `CTextFileLoader::Cache()` (muhtemelen `EterLib` veya `EterBase`'den) ile dosyayı yükler.
        *   Dosyadan "soundpositionenable" anahtar kelimesini arar ve değerini (float) okur. Bu değer, `rSoundDataVector`'deki tüm elemanların `fSoundVolume` alanına atanır (bu kullanım biraz kafa karıştırıcıdır; `fSoundVolume` genellikle ses seviyesi için kullanılır, pozisyonla doğrudan ilgili olmayabilir, ancak kodda bu şekilde yorumlanmış).
        *   "sounddatacount" anahtar kelimesini okuyarak toplam ses verisi sayısını alır ve `rSoundDataVector`'ü bu boyutta yeniden boyutlandırır.
        *   Döngü içinde "sounddata00", "sounddata01" gibi başlıkları arar. Her başlık için iki token bekler: ilki `fTime`, ikincisi ses dosyasının adı.
        *   Eğer `c_szPathHeader` verilmişse, okunan dosya adına bu başlığı ekler.
        *   Okunan bilgileri `rSoundDataVector`'e doldurur.
        *   İşlem sonucunu `SetResultString` ile ayarlar.
    *   **`SaveSoundInformationPiece()`:**
        *   Eğer `rSoundDataVector` boşsa ve dosya diskte mevcutsa, dosyayı `_unlink()` ile siler.
        *   Aksi takdirde, dosyayı metin yazma modunda (`"wt"`) açar.
        *   Dosyaya "ScriptType CharacterSoundInformation" ve "SoundDataCount" gibi başlık bilgileri yazar.
        *   `rSoundDataVector`'deki her bir `TSoundData` elemanını "SoundDataXX zaman "dosya_adı"" formatında dosyaya yazar.
    *   **`DataToInstance()`:**
        *   Sabit bir FPS değeri (60) üzerinden saniye başına kare süresini (`c_fFrameTime`) hesaplar.
        *   Verilen `c_rSoundDataVector`'deki her `TSoundData` için:
            *   `dwFrame = (DWORD)(c_rSoundData.fTime / c_fFrameTime)` formülüyle zamanı kare numarasına dönüştürür.
            *   Dosya adını ve ses seviyesini `TSoundInstance` yapısına kopyalar.
            *   Oluşturulan `TSoundInstance`'ı `pSoundInstanceVector`'e ekler.
    *   **Kullanım Amacı:**
        *   `NSound` namespace'i ve içindeki yapılar (`TSoundData`, `TSoundInstance`) ile fonksiyonlar, oyun içi ses olaylarını (özellikle animasyonlarla senkronize olanları) tanımlamak, yönetmek ve harici dosyalardan yüklemek için bir sistem sunar.
        *   Geliştiricilere veya tasarımcılara, ses dosyalarını belirli zamanlamalara (saniye cinsinden) veya animasyon karelerine bağlayan metin tabanlı yapılandırma dosyaları (örneğin, `.mss` uzantılı dosyalar) oluşturma imkanı tanır.
        *   `LoadSoundInformationPiece` fonksiyonu, bu yapılandırma dosyalarını okuyarak ses olayı verilerini (`TSoundDataVector`) çalışma zamanında kullanılabilir hale getirir.
        *   `SaveSoundInformationPiece` fonksiyonu, bu verilerin tekrar diske yazılmasını sağlar.
        *   `DataToInstance` fonksiyonu, zaman tabanlı ses tanımlamalarını (`TSoundData`) kare tabanlı tanımlamalara (`TSoundInstance`) dönüştürerek animasyon sistemleriyle daha kolay entegrasyon sağlar.
        *   Temel olarak, karakterlerin veya diğer oyun içi varlıkların animasyonları sırasında (örneğin, adım sesleri, saldırı efektleri, yetenek kullanım sesleri) doğru anlarda seslerin çalınmasını sağlamak amacıyla, bu ses olaylarını tanımlayan harici listeleri işlemek için kullanılır.
        *   `GetResultString` ve `SetResultString` fonksiyonları, dosya yükleme/kaydetme veya veri dönüştürme işlemlerinin başarı durumunu veya olası hataları bildirmek için bir mekanizma sunar.