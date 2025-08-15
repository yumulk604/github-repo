# UserInterface Referans Kılavuzu - Bölüm 3

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 2'de ele alınan konuların ardından, bu bölümde kalan C++ sınıfları, Python modülleri ve diğer ilgili bileşenler incelenecektir.

## İçindekiler

* [`GuildMarkUploader.h` ve `GuildMarkUploader.cpp`](#guildmarkuploaderh-ve-guildmarkuploadercpp)
* [`HWIDManager.h` ve `HWIDManager.cpp`](#hwidmanagerh-ve-hwidmanagercpp)
* [`imssapi.h` (Miles Sound System Internal API)](#imssapih-miles-sound-system-internal-api)
* [`InstanceBase.h` ve `InstanceBase.cpp`](#instancebaseh-ve-instancebasecpp)

*(İçerikler eklendikçe güncellenecektir)*

### `GuildMarkUploader.h` ve `GuildMarkUploader.cpp`

#### Sınıfın Genel Amacı

`CGuildMarkUploader` sınıfı, Metin2 istemcisinden sunucuya lonca sembollerini (hem standart 16x12 piksel lonca işaretlerini/marklarını hem de daha büyük 64x128 piksel lonca amblemlerini/sembollerini) yüklemek (upload) için kullanılan özel bir ağ akışıdır (`CNetworkStream`). `CSingleton` olarak tasarlanmıştır. Temel görevi, yerel bir dosyadan lonca sembolünü yüklemek, sunucuya bağlanmak, kimlik doğrulama bilgilerini göndermek ve ardından sembol verisini sunucuya iletmektir. Yükleme işlemi tamamlandığında veya bir hata oluştuğunda bağlantıyı sonlandırır.

#### Başlık Dosyası Tanımları (`GuildMarkUploader.h`)

`GuildMarkUploader.h` dosyası, `CGuildMarkUploader` sınıfının yapısını, üye değişkenlerini ve fonksiyon prototiplerini tanımlar.

*   **Kalıtım**: `CNetworkStream`'den ve `CSingleton<CGuildMarkUploader>`'dan kalıtım alır.
*   **Enum Tanımları**:
    *   `STATE_OFFLINE`, `STATE_LOGIN`, `STATE_COMPLETE`: Yükleme işleminin farklı durumlarını temsil eder.
    *   `ERROR_NONE`, `ERROR_CONNECT`, `ERROR_LOAD`, `ERROR_WIDTH`, `ERROR_HEIGHT`: Yükleme sırasında oluşabilecek hata türlerini belirtir.
    *   `SEND_TYPE_MARK`, `SEND_TYPE_SYMBOL`: Yüklenecek sembolün türünü (standart mark veya büyük sembol) belirtir.
*   **Yapıcı ve Yıkıcı**: `CGuildMarkUploader()`, `virtual ~CGuildMarkUploader()`.
*   **Bağlantı Fonksiyonları**:
    *   `Connect(const CNetworkAddress& c_rkNetAddr, DWORD dwHandle, DWORD dwRandomKey, DWORD dwGuildID, const char* c_szFileName, UINT* peError)`: Standart lonca işaretini (mark) belirtilen dosyadan yükleyerek sunucuya göndermek üzere bağlanır.
    *   `ConnectToSendSymbol(const CNetworkAddress& c_rkNetAddr, DWORD dwHandle, DWORD dwRandomKey, DWORD dwGuildID, const char* c_szFileName, UINT* peError)`: Büyük lonca sembolünü (amblemini) belirtilen dosyadan yükleyerek sunucuya göndermek üzere bağlanır.
*   **Durum ve İşlem Fonksiyonları**:
    *   `Process()`: Ağ olaylarını ve durum makinesini işler.
    *   `Disconnect()`: Bağlantıyı sonlandırır ve durumu sıfırlar.
    *   `IsCompleteUploading()`: Yükleme işleminin tamamlanıp tamamlanmadığını kontrol eder (aslında `STATE_OFFLINE` olup olmadığını kontrol eder, bu da işlemin bittiği anlamına gelir).
*   **Özel (Private) Fonksiyonlar**:
    *   **Dosya İşlemleri**:
        *   `__Save(const char* c_szFileName)`: Verilen dosyaya lonca markını kaydetmeyi amaçlar (ancak kod içinde yorum satırı halinde bırakılmış, aktif değil).
        *   `__Load(const char* c_szFileName, UINT* peError)`: Standart lonca markını (16x12) belirtilen dosyadan yükler (`DevIL` kütüphanesi kullanılır), boyut kontrolü yapar ve `m_kMark.m_apxBuf` içine kopyalar.
        *   `__LoadSymbol(const char* c_szFileName, UINT* peError)`: Büyük lonca sembolünü (64x128) belirtilen dosyadan yükler. Önce `DevIL` ile boyut kontrolü yapar, sonra dosyayı binary olarak okuyup `m_pbySymbolBuf` içine alır ve CRC32 değerini hesaplar.
    *   **Ağ Olayı İşleyicileri**: `OnConnectFailure`, `OnConnectSuccess`, `OnRemoteDisconnect`, `OnDisconnect`.
    *   **Başlatma ve Durum Yönetimi**: `__Inialize` (yazım hatası var, Initialize olmalı), `__StateProcess`, `__OfflineState_Set`, `__CompleteState_Set`, `__LoginState_Set`.
    *   **Paket Analizi**: `__AnalyzePacket(UINT uHeader, UINT uPacketSize, bool (CGuildMarkUploader::*pfnDispatchPacket)())`: Belirli bir başlık ve boyuttaki paketi analiz edip ilgili işleyici fonksiyona yönlendirir.
    *   **Login Durumu İşleyicileri**:
        *   `__LoginState_Process`: Login durumundaki gelen paketleri analiz eder.
        *   `__LoginState_RecvPhase`: Sunucudan gelen faz bilgisini alır. `PHASE_LOGIN` ise, `m_dwSendType`'a göre ya `__SendMarkPacket()` ya da `__SendSymbolPacket()` çağırır.
        *   `__LoginState_RecvHandshake`: Sunucuyla el sıkışma işlemini yapar ve istemci bilgilerini gönderir.
        *   `__LoginState_RecvPing`: Sunucudan gelen ping'e pong ile cevap verir.
        *   `__LoginState_RecvKeyAgreement`, `__LoginState_RecvKeyAgreementCompleted` (`__IMPROVED_PACKET_ENCRYPTION__` tanımlıysa): Gelişmiş paket şifrelemesi için anahtar anlaşması adımlarını işler.
    *   **Gönderme Fonksiyonları**:
        *   `__SendMarkPacket()`: Yüklü olan standart lonca markını (`m_kMark`) `HEADER_CG_MARK_UPLOAD` paketi ile sunucuya gönderir.
        *   `__SendSymbolPacket()`: Yüklü olan büyük lonca sembolünü (`m_pbySymbolBuf`) `HEADER_CG_GUILD_SYMBOL_UPLOAD` paketi ile (başlık + veri) sunucuya gönderir ve işlemi tamamlar (`__CompleteState_Set`).
*   **Özel (Private) Üye Değişkenler**:
    *   `m_eState`: Mevcut yükleme durumu.
    *   `m_dwSendType`: Yüklenecek sembolün türü.
    *   `m_dwGuildID`, `m_dwHandle`, `m_dwRandomKey`: Lonca ID'si ve sunucu tarafından verilen oturum bilgileri.
    *   `SGuildMark m_kMark`: Standart lonca markının piksel verilerini tutan yapı.
    *   `BYTE* m_pbySymbolBuf`: Büyük lonca sembolünün ham byte verilerini tutan işaretçi.
    *   `DWORD m_dwSymbolBufSize`: `m_pbySymbolBuf`'ın boyutu.
    *   `DWORD m_dwSymbolCRC32`: Yüklenen sembol dosyasının CRC32 değeri.

#### C++ Implementasyon Detayları (`GuildMarkUploader.cpp`)

`GuildMarkUploader.cpp` dosyası, sınıfın fonksiyonlarının detaylı implementasyonunu içerir.

*   **`__VTUNE__` Makrosu**: Kodun büyük bir kısmı `#ifdef __VTUNE__ #else ... #endif` bloğu içindedir. VTune bir performans analiz aracıdır; bu, VTune ile profil oluşturma sırasında bu sınıfın devre dışı bırakılmasını sağlıyor olabilir.
*   **DevIL Kütüphanesi Kullanımı**: Lonca sembolü dosyalarını yüklemek ve boyutlarını kontrol etmek için `DevIL` (Developer's Image Library) kullanılır (`ilGenImages`, `ilBindImage`, `ilLoad`, `ilGetInteger`, `ilConvertImage`, `ilCopyPixels`, `ilDeleteImages`).
*   **Yapıcı**: Tampon boyutlarını ayarlar ve `__Inialize()` çağırır.
*   **Yıkıcı**: `__OfflineState_Set()` çağırarak durumu sıfırlar.
*   **`Connect` / `ConnectToSendSymbol`**:
    *   Durumu sıfırlar, ağ tampon boyutlarını ayarlar.
    *   `CNetworkStream::Connect` ile bağlantıyı kurar. Başarısız olursa hata kodu ayarlar ve `false` döner.
    *   Gönderilecek sembol türünü (`m_dwSendType`), oturum bilgilerini ve lonca ID'sini ayarlar.
    *   İlgili yükleme fonksiyonunu (`__Load` veya `__LoadSymbol`) çağırır. Başarısız olursa hata kodu ayarlar ve `false` döner.
*   **Dosya Yükleme (`__Load`, `__LoadSymbol`)**:
    *   `__Load`: `DevIL` kullanarak 16x12 boyutunda bir resim dosyası yükler, BGRA formatına çevirir ve piksel verilerini `m_kMark.m_apxBuf`'a kopyalar. Boyutlar eşleşmezse veya yükleme başarısız olursa hata kodu ayarlar.
    *   `__LoadSymbol`: Önce `DevIL` ile resmin 64x128 boyutunda olduğunu doğrular. Ardından dosyayı binary modda açar, tüm içeriğini `m_pbySymbolBuf`'a okur, boyutunu `m_dwSymbolBufSize`'a kaydeder ve dosyanın CRC32'sini hesaplayıp `m_dwSymbolCRC32`'ye atar. Yükleme veya boyut hatası olursa hata kodu ayarlar.
*   **`Process()`, Ağ Olay İşleyicileri, Durum Yönetimi**: `CGuildMarkDownloader`'a benzer şekilde çalışır; gelen paketleri işler ve durum makinesini yönetir. Başarısızlık durumunda veya işlem tamamlandığında `__OfflineState_Set()` çağrılarak bağlantı sonlandırılır ve kaynaklar temizlenir (`m_pbySymbolBuf` silinir).
*   **Paket Gönderme (`__SendMarkPacket`, `__SendSymbolPacket`)**:
    *   `__SendMarkPacket`: `m_kMark.m_apxBuf`'taki veriyi `TPacketCGMarkUpload` paketiyle gönderir.
    *   `__SendSymbolPacket`: `TPacketCGSymbolUpload` başlığını ve ardından `m_pbySymbolBuf`'taki sembol verisini gönderir. Gönderim sonrası hemen `CNetworkStream::__SendInternalBuffer()` çağrısı yaparak verinin anında gönderilmesini zorlar ve ardından `__CompleteState_Set()` ile işlemi tamamlar.
*   **Gelen Paket İşleme**: `__LoginState_Process` fonksiyonu, `__AnalyzePacket` yardımcı fonksiyonunu kullanarak `HEADER_GC_PHASE`, `HEADER_GC_HANDSHAKE`, `HEADER_GC_PING` ve (varsa) şifreleme paketlerini bekler ve ilgili `__LoginState_Recv...` metodlarına yönlendirir. Bu metodlar, `CGuildMarkDownloader`'dakilere benzer şekilde sunucuyla ilk iletişim adımlarını gerçekleştirir. `__LoginState_RecvPhase` içinde `PHASE_LOGIN` alındığında, asıl sembol yükleme paketi (`__SendMarkPacket` veya `__SendSymbolPacket`) gönderilir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu sınıf, oyuncuların (genellikle lonca liderlerinin) oyun içinden kendi loncaları için özel işaret (mark) veya sembol (amblemi) yüklemelerine olanak tanıyan bir arayüzün istemci tarafı mantığını oluşturur.

1.  **Kullanıcı Arayüzü Etkileşimi**: Oyuncu, oyun içindeki bir arayüzden bir resim dosyası seçer (örneğin, 16x12 boyutunda bir `.tga` veya `.jpg` dosyası mark için, 64x128 boyutunda bir dosya sembol için).
2.  **Bağlantı ve Yükleme**:
    *   Arayüz, seçilen dosya adı ve lonca ID'si ile birlikte `CGuildMarkUploader::Instance().Connect()` veya `CGuildMarkUploader::Instance().ConnectToSendSymbol()` fonksiyonunu çağırır.
    *   Sınıf, dosyayı yerelden yükler, boyutunu ve formatını kontrol eder.
    *   Sunucuya bağlanır, kimlik doğrulama yapar.
    *   Başarılı kimlik doğrulamanın ardından resim verisini sunucuya gönderir.
3.  **Sonuç**: Sunucu, yüklenen sembolü alır, işler (örneğin, uygun formatta kaydeder, diğer oyunculara dağıtılmak üzere hazırlar). `CGuildMarkUploader` işlemi tamamlar ve bağlantıyı keser. Yükleme sırasında bir hata oluşursa (dosya boyutu yanlış, sunucuya bağlanılamıyor vb.), `peError` parametresi üzerinden arayüze bilgi verilir.

Bu sınıf, `CGuildMarkDownloader`'ın tersi bir işlev görerek, istemciden sunucuya sembol verisi gönderir ve loncaların kendilerini görsel olarak kişiselleştirmelerine olanak tanır.

### `HWIDManager.h` ve `HWIDManager.cpp`

#### Sınıfın Genel Amacı

`CHWIDManager` sınıfı, Windows Kayıt Defteri'nden (Registry) bilgisayarın benzersiz Donanım Kimliği'ni (Hardware ID - HWID) almak için kullanılan bir singleton (tek örnek) yöneticisidir. Alınan bu HWID, genellikle oyun istemcisi tarafından sunucuya gönderilerek hesap güvenliği, bot tespiti veya belirli kullanıcıların makinelerini tanımlama gibi amaçlarla kullanılır. Sınıf, `CSingleton<CHWIDManager>`'dan türeyerek global erişim sağlar.

#### Başlık Dosyası Tanımları (`HWIDManager.h`)

`HWIDManager.h` dosyası, `CHWIDManager` sınıfının yapısını ve arayüzünü tanımlar.

*   **`#include <string>`**: `std::string` ve `std::wstring` kullanmak için dahil edilmiştir.
*   **`//#include "../EterBase/Singleton.h"`**: `CSingleton` başlık dosyasının dahil edilmesi yorum satırı haline getirilmiş. Bu, projenin başka bir yerinde `CSingleton`'ın zaten tanımlı olduğu veya farklı bir şekilde dahil edildiği anlamına gelebilir. Ancak sınıf tanımında `public CSingleton<CHWIDManager>` kullanıldığı için bu başlığın bir şekilde erişilebilir olması gerekir.
*   **`#include <windows.h>`**: Windows API fonksiyonlarını (örneğin, Kayıt Defteri işlemleri için `HKEY`, `LSTATUS`, `RegOpenKeyExW`, `RegQueryValueExW`, `RegCloseKey`) kullanmak için dahil edilmiştir.
*   **`class CHWIDManager : public CSingleton<CHWIDManager>`**:
    *   Sınıf, global erişim için `CSingleton` şablonundan türetilmiştir.
    *   **Yapıcı `CHWIDManager()`**: HWID'yi kayıt defterinden okuma işlemini başlatır.
    *   **Yıkıcı `~CHWIDManager()`**: Kaynakları serbest bırakır (bu örnekte özel bir işlem yapmıyor).
    *   **`std::string GetHWID()`**: Alınan HWID'yi `std::string` olarak döndürür.
*   **Özel (Private) Üyeler**:
    *   **`long GetStringRegKey(HKEY hKey, const std::wstring& strValueName, std::wstring& strValue, const std::wstring& strDefaultValue)`**: Belirtilen Kayıt Defteri anahtarından bir string değeri okumak için yardımcı bir fonksiyondur.
    *   **`std::string m_strHWID`**: Okunan HWID'yi saklamak için kullanılan string değişkeni.

#### C++ Implementasyon Detayları (`HWIDManager.cpp`)

`HWIDManager.cpp` dosyası, `CHWIDManager` sınıfının fonksiyonlarının gerçek implementasyonlarını içerir.

*   **`CHWIDManager::CHWIDManager()` (Yapıcı)**:
    1.  `HKEY hKey;`: Kayıt Defteri anahtarını tutmak için bir `HKEY` değişkeni tanımlar.
    2.  `RegOpenKeyExW(HKEY_LOCAL_MACHINE, L"SOFTWARE\\Microsoft\\Cryptography", 0, KEY_READ | KEY_WOW64_64KEY, &hKey)`:
        *   `HKEY_LOCAL_MACHINE` altında `SOFTWARE\Microsoft\Cryptography` anahtarını okuma erişimiyle (`KEY_READ`) açmaya çalışır.
        *   `KEY_WOW64_64KEY` bayrağı, 64-bit bir sistemde 32-bit bir uygulamanın 64-bit Kayıt Defteri görünümüne erişmesini sağlar (veya tam tersi, uygulamanın bit mimarisine göre doğru görünümü hedefler). Bu, `MachineGuid` değerinin doğru yerden okunmasını garantilemek için önemlidir.
    3.  Eğer `RegOpenKeyExW` başarılı olursa (`res == ERROR_SUCCESS`):
        *   `std::wstring temp;`: Geçici bir `std::wstring` oluşturur.
        *   `GetStringRegKey(hKey, L"MachineGuid", temp, L"");`: `GetStringRegKey` yardımcı fonksiyonunu çağırarak açılan anahtardan `MachineGuid` adlı değeri okur ve `temp` içine atar.
        *   Eğer `GetStringRegKey` de başarılı olursa:
            *   `m_strHWID.assign(temp.begin(), temp.end());`: Okunan `std::wstring` (`temp`) içeriğini `std::string` olan `m_strHWID`'ye kopyalar (karakter seti dönüşümü dolaylı olarak gerçekleşir).
    4.  `RegCloseKey(hKey)`: Açılan Kayıt Defteri anahtarını kapatır.

*   **`CHWIDManager::~CHWIDManager()` (Yıkıcı)**:
    *   Bu örnekte herhangi bir özel işlem yapmaz. Kayıt Defteri anahtarı zaten yapıcı içinde kapatılmıştır.

*   **`LONG CHWIDManager::GetStringRegKey(HKEY hKey, const std::wstring& strValueName, std::wstring& strValue, const std::wstring& strDefaultValue)`**:
    1.  `strValue = strDefaultValue;`: `strValue` parametresini varsayılan değerle başlatır.
    2.  `WCHAR szBuffer[512];`: Değeri okumak için 512 karakterlik bir `WCHAR` tamponu tanımlar.
    3.  `DWORD dwBufferSize = sizeof(szBuffer);`: Tamponun boyutunu bayt cinsinden ayarlar.
    4.  `RegQueryValueExW(hKey, strValueName.c_str(), 0, NULL, (LPBYTE)szBuffer, &dwBufferSize)`:
        *   Belirtilen `hKey`'den `strValueName` ile eşleşen değeri okur ve `szBuffer`'a yazar.
    5.  Eğer `RegQueryValueExW` başarılı olursa (`ERROR_SUCCESS == nError`):
        *   `strValue = szBuffer;`: Okunan değeri `strValue`'ya atar.
    6.  Fonksiyon, `RegQueryValueExW`'nin dönüş kodunu (`nError`) döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

`CHWIDManager`, oyun istemcisinin çalıştığı bilgisayara özgü bir kimlik elde etmek için kullanılır:

1.  **Başlangıçta HWID Alınması**: Oyun istemcisi başlatıldığında, `CHWIDManager` singleton örneği otomatik olarak oluşturulur ve yapıcı metodu çalışarak Kayıt Defteri'nden `MachineGuid`'i okur. Bu `MachineGuid`, genellikle Windows kurulumu sırasında oluşturulan ve makineye özgü (ancak donanım değişikliği veya sanallaştırma ile değişebilen) bir tanımlayıcıdır.
2.  **Sunucuya Gönderme**: Elde edilen `m_strHWID` (veya `GetHWID()` ile alınan değer), genellikle giriş (login) paketleriyle (`AccountConnector.cpp` içinde `TPacketCGLogin3` gibi) veya diğer ağ paketleriyle sunucuya gönderilir.
3.  **Sunucu Tarafı Kullanımı**:
    *   **Hesap Güvenliği**: Sunucu, bir hesaba farklı HWID'lerden sık sık giriş yapılıp yapılmadığını kontrol ederek şüpheli aktiviteleri tespit edebilir.
    *   **Kısıtlamalar**: Belirli HWID'lere erişim kısıtlaması getirebilir (örneğin, yasaklanmış bir oyuncunun farklı hesaplarla aynı makineden girmesini zorlaştırmak).
    *   **Bot Tespiti**: Çok sayıda hesabın aynı HWID üzerinden oyuna girmesi, bot aktivitesi belirtisi olabilir.
    *   **Ödüller/Etkinlikler**: Belirli etkinliklerde, her HWID'nin yalnızca bir kez ödül almasını sağlamak için kullanılabilir.

`CHWIDManager`, istemcinin çalıştığı makineyi tanımlamak için basit ama etkili bir yöntem sunar. `MachineGuid`'in %100 değişmez veya taklit edilemez olmadığı unutulmamalıdır, ancak birçok durumda yeterli bir tanımlayıcı görevi görür.

### `imssapi.h` (Miles Sound System Internal API)

#### Dosyanın Genel Amacı

`imssapi.h` dosyası, RAD Game Tools tarafından geliştirilen Miles Sound System (MSS) adlı kapsamlı bir ses motorunun dahili veya alt seviye API (Uygulama Programlama Arayüzü) başlık dosyasıdır. Metin2 istemcisi gibi oyunlar ve multimedya uygulamaları tarafından ses çalma, miksaj, 3B ses efektleri, MIDI işleme, ses akışı (streaming) ve daha birçok gelişmiş ses özelliğini yönetmek için kullanılır. Bu başlık dosyası, çok sayıda fonksiyon, yapı, tür tanımı ve makro bildirimi içerir. Oldukça büyük bir dosya olup (2000 satırdan fazla), Miles Sound System SDK'sının bir parçasıdır ve genellikle `mss.h` gibi diğer temel MSS başlık dosyalarına bağımlıdır.

#### Temel Yapılar, Tanımlar ve API Konseptleri

`imssapi.h` içinde birçok temel konsept ve yapı tanımlanmıştır:

*   **`VOC` Yapısı**: Creative Labs'in `.VOC` ses dosyası formatının başlık yapısını tanımlar.
*   **Platforma Özel Derleme**: Kod, `IS_WIN32`, `IS_MAC`, `IS_LINUX`, `IS_DOS` gibi önişlemci direktifleri kullanılarak farklı platformlara özgü API çağrılarını ve davranışları yönetir. Örneğin, Windows için `HWND`, `HANDLE` gibi tipler ve mutex yönetimi için `CreateMutex`, `WaitForSingleObject` gibi fonksiyonlar kullanılır.
*   **Mutex (Kilitleme) Mekanizmaları**: `InMilesMutex()` ve `OutMilesMutex()` gibi makrolar, kütüphane içindeki kritik bölümlere erişimde iş parçacığı güvenliğini (thread safety) sağlamak için kullanılır. Bu, özellikle çoklu iş parçacıklı ortamlarda önemlidir.
*   **API Fonksiyon İsimlendirme**: Fonksiyonların büyük çoğunluğu `AIL_API_` önekiyle başlar (örneğin, `AIL_API_startup`, `AIL_API_allocate_sample_handle`). "AIL", muhtemelen "Audio Interface Library" anlamına gelir.
*   **Handle (Tanıtıcı) Tabanlı Kaynak Yönetimi**: Kütüphane, ses sürücüleri, ses örneklemleri, MIDI sekansları gibi kaynakları yönetmek için opak tanıtıcılar (handles) kullanır. Örnekler:
    *   `HDIGDRIVER`: Dijital ses sürücüsü tanıtıcısı.
    *   `HSAMPLE`: Bir ses örneklemi (sound sample) tanıtıcısı.
    *   `HMDIDRIVER`: MIDI sürücüsü tanıtıcısı.
    *   `HSEQUENCE`: Bir MIDI sekansı tanıtıcısı.
    *   `HPROVIDER`: Bir 3B ses veya filtre sağlayıcısı tanıtıcısı.
    *   `H3DSAMPLE`: Bir 3B ses örneklemi tanıtıcısı.
    *   `HSTREAM`: Bir ses akışı (stream) tanıtıcısı.
    *   `HDLSDEVICE`: Bir DLS (Downloadable Sounds) cihazı tanıtıcısı.
    Bu tanıtıcılar, ilgili `AIL_API_open_...`, `AIL_API_allocate_...` fonksiyonlarıyla elde edilir ve `AIL_API_close_...`, `AIL_API_release_...` fonksiyonlarıyla serbest bırakılır.
*   **Geri Çağrı (Callback) Fonksiyonları**: Zamanlayıcı olayları, ses örnekleminin sonu (End Of Sample - EOS), MIDI olayları gibi asenkron durumları işlemek için çeşitli geri çağrı fonksiyon türleri tanımlanmıştır (örneğin, `AILTIMERCB`, `AILSAMPLECB`, `AILSEQUENCECB`, `AILSTREAMCB`). Kullanıcı, bu türden kendi fonksiyonlarını kütüphaneye kaydedebilir.
*   **Temel Veri Tipleri**: Dosyada `S8`, `U16`, `S32`, `U32`, `F32` gibi temel veri tipleri kullanılır. Bu tiplerin tanımları `imssapi.h` içinde bulunmaz; muhtemelen `mss.h` gibi ana Miles Sound System başlık dosyasında platformdan bağımsız olacak şekilde tanımlanmışlardır. Bu dosyanın tek başına derlenmeye çalışılması durumunda bu tiplerin tanımsız olması linter hatalarına yol açar.

#### Ana Fonksiyon Grupları (Özet)

`imssapi.h` içindeki API fonksiyonları genel olarak aşağıdaki kategorilere ayrılabilir:

*   **Genel Başlatma, Kapatma ve Ayarlar**: `AIL_API_startup()`, `AIL_API_shutdown()`, `AIL_API_set_preference()`, `AIL_API_last_error()`.
*   **Zamanlayıcı (Timer) Servisleri**: `AIL_API_register_timer()`, `AIL_API_start_timer()`.
*   **Dijital Ses (Wave) Servisleri**:
    *   Sürücü Yönetimi: `AIL_API_open_digital_driver()`, `AIL_API_waveOutOpen()` (Win32).
    *   Örneklem (Sample) Yönetimi: `AIL_API_allocate_sample_handle()`, `AIL_API_set_sample_file()`, `AIL_API_start_sample()`, `AIL_API_set_sample_volume_pan()`, `AIL_API_register_EOS_callback()`.
    *   Ana Ses Kontrolü: `AIL_API_set_digital_master_volume_level()`.
*   **3B Ses (M3D) Servisleri**:
    *   Sağlayıcı ve Örneklem Yönetimi: `AIL_API_open_3D_provider()`, `AIL_API_allocate_3D_sample_handle()`.
    *   Konumlandırma: `AIL_API_set_3D_position()`, `AIL_API_set_3D_orientation()`.
    *   Dinleyici Yönetimi: `AIL_API_open_3D_listener()`.
*   **MIDI (XMIDI) Servisleri**:
    *   Sürücü ve Sekans Yönetimi: `AIL_API_open_XMIDI_driver()`, `AIL_API_allocate_sequence_handle()`, `AIL_API_init_sequence()`, `AIL_API_start_sequence()`.
    *   Kontrol: `AIL_API_set_sequence_tempo()`, `AIL_API_set_sequence_volume()`.
*   **Akış (Streaming) Servisleri**: `AIL_API_open_stream()`, `AIL_API_service_stream()`, `AIL_API_set_stream_volume_pan()`.
*   **DLS (İndirilebilir Sesler) Servisleri**: `AIL_API_DLS_open()`, `AIL_API_DLS_load_file()`.
*   **"Quick" API (Basitleştirilmiş Fonksiyonlar)**: `AIL_API_quick_startup()`, `AIL_API_quick_load()`, `AIL_API_quick_play()`.
*   **Redbook CD Ses Servisleri**: `AIL_API_redbook_open()`, `AIL_API_redbook_play()`.
*   **Dosya G/Ç ve Yardımcı Fonksiyonlar**: `AIL_API_file_read()`, `AIL_API_WAV_info()`, `AIL_API_compress_ADPCM()`.
*   **Bellek Dosyası Yardımcıları**: `AIL_mem_open()`, `AIL_mem_printf()`.
*   **Platforma Özgü Düşük Seviye Fonksiyonlar ve Makrolar**: Çeşitli bellek, string ve pointer işlemleri için makrolar.

#### Metin2 İstemcisindeki Olası Kullanım Alanları

Metin2 istemcisi, `imssapi.h` (ve dolayısıyla Miles Sound System) aracılığıyla aşağıdaki gibi çeşitli ses işlemlerini gerçekleştirebilir:

*   **Arka Plan Müziği**: Farklı haritalar ve durumlar için arka plan müziklerini çalmak.
*   **Ses Efektleri (SFX)**: Karakter yetenekleri, canavar sesleri, kullanıcı arayüzü tıklamaları, ortam sesleri gibi çok sayıda kısa ses efektini çalmak.
*   **3B Konumsal Ses**: Oyun dünyasındaki seslerin oyuncunun konumuna ve yönelimine göre gerçekçi bir şekilde duyulmasını sağlamak.
*   **Ses Seviyesi Kontrolü**: Ana ses seviyesi, müzik sesi, efekt sesi gibi ayarların yönetimi.
*   **Dinamik Ses Yönetimi**: Oyun içi olaylara bağlı olarak seslerin başlatılması ve durdurulması.
*   **Ses Dosyası Yükleme ve Kaynak Yönetimi**.

#### Önemli Notlar (Bağımlılıklar ve Linter Hataları Hakkında)

*   `imssapi.h` dosyası, Miles Sound System SDK'sının sadece bir parçasıdır ve derlenebilmesi için `mss.h` gibi diğer Miles başlık dosyalarına ihtiyaç duyar. Bu diğer başlıklar, `S8`, `U32`, `HDIGDRIVER` gibi temel türleri tanımlar.
*   Bu dosyanın incelenmesi sırasında karşılaşılan çok sayıda linter hatası (tanımsız değişkenler vb.), büyük olasılıkla bu harici bağımlılıkların eksikliğinden kaynaklanmaktadır.
*   Bu belgeleme, `imssapi.h` dosyasında bildirilen genel işlevlere odaklanır. Metin2'nin bu fonksiyonları spesifik olarak nasıl kullandığı, bu API çağrılarının yapıldığı `.cpp` dosyalarının incelenmesiyle daha net anlaşılacaktır.

### `InstanceBase.h` ve `InstanceBase.cpp`

#### Sınıfın Genel Amacı

`CInstanceBase` sınıfı, Metin2 istemcisindeki tüm "canlı" ve etkileşimli varlıkların (oyuncular, NPC'ler, canavarlar, binekler, bazı özel nesneler) temelini oluşturan merkezi bir sınıftır. Her bir `CInstanceBase` örneği, oyun dünyasında bir karakteri veya nesneyi temsil eder. Bu sınıf, varlıkların oluşturulması, yok edilmesi, güncellenmesi, render edilmesi, hareketleri, saldırıları, durumları (affect'ler), ekipmanları, görünüşleri ve oyun dünyasıyla etkileşimleri gibi çok geniş bir yelpazede işlevselliği yönetir. `CActorInstance` sınıfını (grafiksel temsil ve animasyon için) ve `CAffectFlagContainer`'ı (durum etkileri için) içinde barındırır. Aynı zamanda `CSingleton` olmamasına rağmen, statik bir havuz (`ms_kPool`) üzerinden örnekleri yönetilir ve `CPythonCharacterManager` aracılığıyla bu örneklere erişilir.

#### Başlık Dosyası Tanımları (`InstanceBase.h`)

`InstanceBase.h` dosyası, `CInstanceBase` sınıfının yapısını, iç içe geçmiş yapılarını (örneğin, `SHORSE`), enumlarını, sabitlerini ve fonksiyon prototiplerini tanımlar.

*   **`struct SCreateData`**: Bir `CInstanceBase` örneği oluşturmak için gereken tüm başlangıç verilerini (tip, VID, ırk, pozisyon, ekipman, isim vb.) içeren yapıdır.
*   **Enumlar**:
    *   **`EDirection`**: 8 ana yönü tanımlar.
    *   **Hareket ve Eylem Fonksiyon Kodları**: `FUNC_WAIT`, `FUNC_MOVE`, `FUNC_ATTACK` gibi sunucudan gelen durum paketlerindeki eylem türlerini tanımlar. `FUNC_SKILL` bir bit maskesi olarak kullanılır.
    *   **`EAffects` (AFFECT\_...)**: Karakterler üzerindeki temel etkileri (zehir, görünmezlik, hız vb.) tanımlayan eski enum.
    *   **`ENewAffects` (NEW\_AFFECT\_...)**: Daha sonra eklenmiş veya daha spesifik etkileri (item bonusları, görev etkileri, polymorph vb.) tanımlayan enum.
    *   **`EStoneSmoke`**: Metin taşlarının can seviyesine göre çıkardığı duman efektlerini tanımlar.
    *   **`EBuildingAffect`**: Lonca binalarının yapım ve geliştirme efektlerini tanımlar.
    *   **Silah Tutuş Tipleri**: `WEAPON_DUALHAND`, `WEAPON_ONEHAND`, `WEAPON_TWOHAND`.
    *   **İmparatorluk ID'leri**: `EMPIRE_NONE`, `EMPIRE_A`, `EMPIRE_B`, `EMPIRE_C`.
    *   **İsim Renkleri (`ENameColorIndex`)**: Karakter tipine (MOB, NPC, PC), imparatorluğa, PK durumuna göre isim rengi indekslerini tanımlar.
    *   **Alignment Tipleri**: `ALIGNMENT_TYPE_WHITE`, `ALIGNMENT_TYPE_NORMAL`, `ALIGNMENT_TYPE_DARK`.
    *   **Emoticon ID'leri**: Oyun içi ifade ikonlarının ID'lerini tanımlar.
    *   **Başlık (Title) ID'leri**: Karakterlerin isimlerinin üzerinde çıkan başlıkların ID'lerini tanımlar (örneğin, Zalim, Kahraman).
    *   **Efekt ID'leri (`EEffects`)**: Silah parlama efektleri, seviye atlama efekti, yetenek kullanım efektleri gibi genel oyun içi efektleri tanımlar.
    *   **Hasar Bayrakları (`DamageFlag`)**: Hasarın türünü (normal, zehir, kritik vb.) belirten bit bayrakları.
    *   **`EFlyEffects`**: Uçan efektlerin (EXP kazanımı, can/mana dolumu göstergeleri) türlerini tanımlar.
    *   **Düello Durumları**: `DUEL_NONE`, `DUEL_CANNOTATTACK`, `DUEL_START`.
*   **Statik Fonksiyonlar ve Üyeler**:
    *   `DestroySystem()`, `CreateSystem()`: `CInstanceBase` örnek havuzunu yönetir.
    *   `RegisterEffect()`, `RegisterTitleName()`, `RegisterNameColor()`, `RegisterTitleColor()`: Efekt, başlık ve renk tanımlarını kaydetmek için kullanılır.
    *   `ms_kPool`: `CInstanceBase` örneklerini tutan statik `CDynamicPool`.
    *   `ms_adwCRCAffectEffect`: Efekt ID'lerini CRC değerleriyle eşleştiren dizi.
*   **`struct SHORSE`**: Bir `CInstanceBase`'in binek (at) bilgilerini ve binek `CActorInstance`'ını yöneten iç içe geçmiş yapıdır.
    *   `m_isMounting`: Bineğe binilip binilmediğini belirtir.
    *   `m_pkActor`: Bineğin kendi `CActorInstance` işaretçisi.
    *   `Create()`, `Destroy()`, `SetAttackSpeed()`, `SetMoveSpeed()`, `GetLevel()`, `CanUseSkill()`, `CanAttack()` gibi binek yönetimi fonksiyonları içerir.
*   **Üye Değişkenler (Önemli Olanlar)**:
    *   `m_GraphicThingInstance`: Karakterin grafiksel temsili, animasyonları ve temel hareket mantığını yöneten `CActorInstance` nesnesi.
    *   `m_stName`: Karakterin adı.
    *   `m_awPart[CRaceData::PART_MAX_NUM]`: Karakterin giydiği ekipmanların (zırh, silah, saç vb.) ID'lerini tutan dizi.
    *   `m_dwLevel`, `m_dwRace`, `m_dwVID`, `m_dwEmpireID`, `m_dwGuildID`: Karakterin seviyesi, ırkı, sanal ID'si, imparatorluğu ve lonca ID'si.
    *   `m_sAlignment`, `m_byPKMode`: Karakterin derecesi ve PK modu.
    *   `m_kAffectFlagContainer`: Karakter üzerindeki aktif etkileri yöneten `CAffectFlagContainer` nesnesi.
    *   `m_kQue_kCmdNew`: Sunucudan gelen hareket ve eylem komutlarını işlemek için kullanılan bir komut kuyruğu.
    *   `m_kHorse`: Binek bilgilerini tutan `SHORSE` yapısı.
*   **Ana Fonksiyon Grupları (Prototip Olarak)**:
    *   Oluşturma/Yok Etme: `Create()`, `Destroy()`.
    *   Güncelleme ve Render: `Update()`, `Transform()`, `Deform()`, `Render()`.
    *   Veri Ayarlama/Alma: `SetNameString()`, `SetRace()`, `SetAlignment()`, `SetArmor()`, `SetWeapon()`, `GetLevel()`, `GetRace()`, `GetVirtualID()`.
    *   Durum Kontrolleri: `IsPC()`, `IsNPC()`, `IsDead()`, `IsMountingHorse()`, `IsWalking()`.
    *   Hareket: `NEW_Goto()`, `EndWalking()`, `SetRunMode()`, `SetWalkMode()`.
    *   Saldırı ve Yetenek: `NEW_UseSkill()`, `NEW_Attack()`, `RunNormalAttack()`, `RunComboAttack()`.
    *   Etkileşim ve Çarpışma: `AvoidObject()`, `IsBlockObject()`, `IntersectBoundingBox()`.
    *   Ağ Paket İşleme: `PushTCPState()`, `StateProcess()`.
    *   Efekt ve Görünüm: `SCRIPT_SetAffect()`, `AttachTextTail()`, `SetHair()`, `ChangeArmor()`.

#### C++ Implementasyon Detayları (`InstanceBase.cpp`)

`InstanceBase.cpp` dosyası, `CInstanceBase` sınıfının ve içindeki `SHORSE` yapısının tüm fonksiyonlarının detaylı implementasyonunu içerir. Dosyanın büyüklüğü nedeniyle tüm detaylara girmek mümkün olmasa da, ana işleyiş prensipleri şunlardır:

*   **`IsWall()` ve `GetMountLevelByVnum()`**: Yardımcı global fonksiyonlar. İlki belirli ırkların duvar olup olmadığını kontrol eder, ikincisi binek VNUM'una göre bineğin seviyesini (normal, savaş, askeri) döndürür.
*   **`CInstanceBase::SHORSE` Implementasyonu**:
    *   `Create()`: Yeni bir binek `CActorInstance` oluşturur, ırkını, şeklini ve temel özelliklerini ayarlar.
    *   `Destroy()`: Binek `CActorInstance`'ını yok eder.
    *   `GetLevel()`: Bineğin VNUM'una göre seviyesini döndürür (yukarıdaki global fonksiyonu kullanır).
    *   `CanUseSkill()`, `CanAttack()`: Bineğin seviyesine göre yetenek kullanıp kullanamayacağını veya saldırı yapıp yapamayacağını belirler.
    *   `Render()`, `Deform()`: Bineğin `CActorInstance`'ının ilgili fonksiyonlarını çağırır.
*   **Statik Üyelerin Başlatılması**: `ms_dwUpdateCounter`, `ms_kPool` gibi statik üyeler burada tanımlanır ve başlatılır.
*   **Çarpışma ve Zemin Etkileşimleri**:
    *   `__Background_GetWaterHeight()`, `__Background_IsWaterPixelPosition()`, `__GetBackgroundHeight()`: `CPythonBackground` singleton'ını kullanarak zemin ve su yüksekliği bilgilerini alır.
    *   `AvoidObject()`, `IsBlockObject()`: `m_GraphicThingInstance` üzerinden çarpışma kontrollerini yapar.
*   **Görünürlük ve Gizlilik**:
    *   `IsInvisibility()`, `IsStealth()`: `AFFECT_INVISIBILITY` ve `AFFECT_EUNHYEONG` etkilerine bakarak karakterin görünmez olup olmadığını kontrol eder. `__MainCanSeeHiddenThing()` ile ana karakterin GM olup gizli nesneleri görüp göremediği de hesaba katılır.
*   **Durum Kontrolleri**: `IsGameMaster()`, `IsSameEmpire()`, `GetAlignmentType()`, `GetPKMode()`, `IsKiller()`, `IsInSafe()` gibi birçok fonksiyon, karakterin çeşitli durumlarını sorgular.
*   **Efekt Yönetimi**:
    *   `OnSelected()`, `OnUnselected()`, `OnTargeted()`, `OnUntargeted()`: Seçim ve hedefleme efektlerini (`__AttachSelectEffect`, `__AttachTargetEffect`) yönetir.
    *   `__AttachEffect()`, `__DetachEffect()`: Genel efekt ekleme/kaldırma mekanizmaları.
    *   `__EffectContainer_...` fonksiyonları: Daha organize bir şekilde efekt ID'lerini yönetir.
    *   `__GetRefinedEffect()`: Zırh ve silahların parlama efektlerini, eşyanın artı basma seviyesine ve türüne göre ayarlar.
*   **Yaratma (`Create`)**:
    *   `SCreateData` yapısındaki tüm bilgileri alarak yeni bir karakter örneği oluşturur: Irk, VID, pozisyon, görünüm (zırh, silah, saç), isim, seviye, imparatorluk, lonca, PK modu, etkiler vb.
    *   Eğer ana karakter ise `SetMainInstance()` çağrılır.
    *   At üzerinde başlıyorsa `MountHorse()` çağrılır.
    *   İsim etiketi (`AttachTextTail()`) eklenir.
    *   Çarpışma küresi kaydedilir (`RegisterBoundingSphere()`).
*   **Ekipman ve Görünüm Değişiklikleri**:
    *   `SetRace()`, `SetArmor()`, `SetShape()`, `SetHair()`, `SetWeapon()`, `ChangeArmor()`, `ChangeWeapon()`, `ChangeHair()`: Karakterin ırkını, zırhını, genel görünümünü (shape, polimorf), saçını ve silahını ayarlar/değiştirir. Bu fonksiyonlar `m_GraphicThingInstance`'ın ilgili metodlarını çağırarak grafiksel güncellemeyi yapar ve `m_awPart` dizisini günceller. Zırh ve silah değişikliklerinde parlama efektleri de güncellenir.
*   **Hareket ve Pozisyon Yönetimi**:
    *   `SCRIPT_SetPixelPosition()`: Karakterin pozisyonunu doğrudan ayarlar.
    *   `NEW_GetPixelPosition()`, `NEW_GetCurPixelPositionRef()`: Mevcut pozisyonu alır.
    *   `NEW_SetDstPixelPosition()`: Hedef pozisyonu ayarlar.
    *   `PushTCPState()`: Sunucudan gelen hareket/eylem komutlarını (`SCommand`) bir kuyruğa (`m_kQue_kCmdNew`) ekler. Komutlar, sunucu zaman damgası ve yerel zaman damgası arasındaki fark (`m_nAverageNetworkGap`) dikkate alınarak zamanlanmış bir şekilde işlenir.
    *   `StateProcess()`: Komut kuyruğunu işler. Zamanı gelen komutları kuyruktan çıkarır ve `eFunc` değerine göre (FUNC_WAIT, FUNC_MOVE, FUNC_ATTACK, FUNC_SKILL vb.) ilgili eylemi başlatır.
        *   Eğer hedef uzaksa, karakteri yürüyüş moduna geçirir (`StartWalking()`) ve hedef pozisyona doğru hareket ettirir. `m_kMovAfterFunc` yapısı, hareket bittikten sonra yapılacak asıl eylemi (örneğin saldırı) saklar.
        *   Eğer hedef yakınsa, eylemi doğrudan gerçekleştirir (bekleme, saldırı, yetenek kullanma).
    *   `MovementProcess()`: Karakterin yürüme/koşma sırasındaki dönüşlerini, zeminle etkileşimini ve toz efekti gibi detayları yönetir. Eğer `m_isGoing` (hareket halinde) ise ve hedefe ulaşılmışsa, `m_kMovAfterFunc`'ta saklanan eylemi gerçekleştirir.
    *   `Update()`: Ana güncelleme fonksiyonudur. `StateProcess()`, `m_GraphicThingInstance.PhysicsProcess()`, `AttackProcess()`, `MovementProcess()`, `m_GraphicThingInstance.MotionProcess()` gibi birçok alt süreci çağırır. Zemin yüksekliğini ve gölge rengini günceller.
*   **Saldırı ve Yetenek Kullanımı**:
    *   `CanAttack()`, `CanUseSkill()`: Karakterin mevcut durumuna (polimorf, binek, giyilen eşya) göre saldırı yapıp yapamayacağını veya yetenek kullanıp kullanamayacağını kontrol eder.
    *   `NEW_UseSkill()`: Belirtilen yeteneği kullanır, uygun animasyonu (`m_GraphicThingInstance.InterceptOnceMotion` veya `InterceptLoopMotion`) başlatır.
    *   `RunNormalAttack()`, `RunComboAttack()`: Normal saldırı veya kombo saldırısı başlatır.
    *   `ProcessHitting()`: Saldırı animasyonları sırasındaki belirli bir anda çağrılarak, hedefe hasar verilip verilmediğini ve ilgili efektlerin (vuruş efekti, ses) tetiklenmesini sağlar.
*   **Animasyon ve Hareket Modları**:
    *   `RefreshState()`: Karakterin hareket modunu (MODE_GENERAL, MODE_HORSE, MODE_ONEHAND_SWORD vb.) silah tipine, binek durumuna ve özel durumlara (polimorf, balıkçılık) göre ayarlar ve belirtilen animasyonu başlatır.
    *   `SetMotionMode()`: `m_GraphicThingInstance` üzerinden hareket modunu ayarlar.
*   **Diğer Önemli Fonksiyonlar**:
    *   `Revive()`, `Stun()`, `Die()`: Karakterin canlanma, sersemleme, ölme gibi durumlarını yönetir.
    *   `SetAffectFlagContainer()`: Karakter üzerindeki tüm etkileri (buff/debuff) ayarlar ve ilgili görsel efektleri (`__SetAffect()`) tetikler.
    *   `__Initialize()`: Bir `CInstanceBase` örneği yeniden havuza döndüğünde veya ilk oluşturulduğunda çağrılarak tüm üye değişkenleri varsayılan değerlerine sıfırlar.

#### Oyunda Kullanım Amacı ve Senaryoları

`CInstanceBase`, oyun dünyasındaki hemen hemen her dinamik varlığı temsil ettiği için sayısız senaryoda kullanılır:

1.  **Karakter Oluşturma**: Oyuncu oyuna girdiğinde veya yeni bir karakter/NPC/canavar görüş alanına girdiğinde, sunucudan gelen bilgilerle (`SCreateData`) bir `CInstanceBase` örneği oluşturulur.
2.  **Hareket**: Oyuncu karakterini hareket ettirdiğinde veya bir NPC/canavar hareket ettiğinde, `PushTCPState` ve `StateProcess/MovementProcess` mekanizmaları devreye girer.
3.  **Saldırı/Yetenek**: Bir karakter saldırdığında veya yetenek kullandığında, ilgili fonksiyonlar (`NEW_Attack`, `NEW_UseSkill`) çağrılır, animasyonlar oynatılır ve `ProcessHitting` ile hasar uygulanır.
4.  **Görünüm Değişiklikleri**: Oyuncu zırhını, silahını değiştirdiğinde veya bir buff/debuff aldığında, `ChangeArmor`, `ChangeWeapon`, `SetAffectFlagContainer` gibi fonksiyonlar çağrılarak karakterin görünümü ve durumu güncellenir.
5.  **Etkileşimler**: Bir oyuncu bir NPC'ye tıkladığında, `IsTargetableInstance` gibi kontroller yapılır.
6.  **Binek Kullanımı**: Oyuncu bineğe bindiğinde/indiğinde `MountHorse()`/`DismountHorse()` çağrılır ve karakterin hareket modu, hızı ve görünümü buna göre ayarlanır.
7.  **Ölüm ve Canlanma**: Bir karakter öldüğünde `Die()`, canlandığında `Revive()` fonksiyonları ile ilgili animasyonlar ve durum değişiklikleri yönetilir.

`CInstanceBase`, istemci tarafındaki oyun mantığının ve görsel sunumun kesişim noktasında yer alan, son derece kritik ve karmaşık bir sınıftır.

### `InstanceBaseBattle.cpp`

#### Dosyanın Genel Amacı

`InstanceBaseBattle.cpp` dosyası, `CInstanceBase` sınıfının dövüş ve etkileşim mekanikleriyle ilgili temel fonksiyonlarını içerir. Bu fonksiyonlar, karakterler arası mesafe hesaplamaları, hedefleme mantığı, saldırı menzili kontrolleri, yeteneklerin ve saldırıların hedeflere yönlendirilmesi gibi işlevleri kapsar. Temelde, bir karakterin başka bir karakterle veya belirli bir pozisyonla olan ilişkisini (mesafe, açı vb.) belirlemek ve bu bilgilere göre aksiyon almak için gereken yardımcı fonksiyonları sağlar.

#### Temel Fonksiyonlar ve İşleyiş

*   **Yardımcı Geometri Fonksiyonları**:
    *   **`NEW_UnsignedDegreeToSignedDegree(float fUD)`**: 0-360 derece aralığındaki bir açıyı -180 ile +180 derece aralığına dönüştürür.
    *   **`NEW_GetSignedDegreeFromDirPixelPosition(const TPixelPosition& kPPosDir)`**: Verilen bir yön vektöründen (TPixelPosition) imzalı bir açı (derece cinsinden) hesaplar. (0, -1, 0) vektörünü (genellikle kuzey veya ileri yön) referans alır.
*   **Uçan Efekt (Fly Target) Yönetimi**:
    *   **`IsFlyTargetObject()`**: Karakterin bir uçan efekt (örneğin, bir ok veya büyü) için hedef olup olmadığını kontrol eder.
    *   **`GetFlyTargetDistance()`**: Uçan efektin hedefe olan mesafesini döndürür.
    *   **`ClearFlyTargetInstance()`**: Karakterin uçan efekt hedefini temizler.
    *   **`SetFlyTargetInstance(CInstanceBase& rkInstDst)`**: Karakteri, belirtilen `rkInstDst` örneğine bir uçan efekt hedefi olarak ayarlar.
    *   **`AddFlyTargetPosition(const TPixelPosition& c_rkPPosDst)`**: Uçan efektin gideceği bir hedef pozisyon ekler.
    *   **`AddFlyTargetInstance(CInstanceBase& rkInstDst)`**: Uçan efektin hedefleyeceği bir `CInstanceBase` örneği ekler. Bu, özellikle çoklu hedefli yetenekler için kullanılabilir.
*   **Mesafe Hesaplama Fonksiyonları**:
    *   **`NEW_GetDistanceFromDestInstance(CInstanceBase& rkInstDst)`**: Mevcut karakter ile `rkInstDst` karakteri arasındaki 2D mesafeyi hesaplar.
    *   **`NEW_GetDistanceFromDestPixelPosition(const TPixelPosition& c_rkPPosDst)`**: Mevcut karakterin pozisyonu ile belirtilen `c_rkPPosDst` pozisyonu arasındaki 2D mesafeyi hesaplar.
    *   **`NEW_GetDistanceFromDirPixelPosition(const TPixelPosition& c_rkPPosDir)`**: Verilen bir yön vektörünün (aslında bir fark vektörü) 2D uzunluğunu hesaplar.
*   **Açı (Rotation) Hesaplama Fonksiyonları**:
    *   **`NEW_GetRotation()`**: Karakterin mevcut açısını (-180, +180 aralığında) döndürür.
    *   **`NEW_GetRotationFromDirPixelPosition(const TPixelPosition& c_rkPPosDir)`**: Verilen bir yön vektörüne karşılık gelen açıyı hesaplar.
    *   **`NEW_GetRotationFromDestPixelPosition(const TPixelPosition& c_rkPPosDst)`**: Mevcut karakterden belirtilen hedef pozisyona olan yönün açısını hesaplar.
    *   **`NEW_GetRotationFromDestInstance(CInstanceBase& rkInstDst)`**: Mevcut karakterden `rkInstDst` karakterine olan yönün açısını hesaplar.
*   **Hedef Seçim ve Alan Etkili Yetenekler İçin Yardımcı Fonksiyonlar**:
    *   **`NEW_GetRandomPositionInFanRange(CInstanceBase& rkInstTarget, TPixelPosition* pkPPosDst)`**: `rkInstTarget` karakterine doğru olan açının etrafında rastgele bir pozisyon (bir yelpaze dilimi içinde) hesaplar. Genellikle alan etkili yeteneklerin görsel efektleri veya isabet noktaları için kullanılır.
    *   **`NEW_GetFrontInstance(CInstanceBase** ppoutTargetInstance, float fDistance)`**: Karakterin baktığı yönde, belirli bir `fDistance` mesafesi içinde ve belirli bir yelpaze açısı içinde kalan en yakın saldırılabilir hedefi bulur. Yelpaze açısı, hedefin uzaklığına göre dinamik olarak değişir (yakındaki hedefler için daha dar, uzaktakiler için daha geniş bir yelpaze).
    *   **`NEW_GetInstanceVectorInFanRange(float fSkillDistance, CInstanceBase& rkInstTarget, std::vector<CInstanceBase*>* pkVct_pkInst)`**: `rkInstTarget` karakterine doğru olan yönde, `fSkillDistance` mesafesi içinde ve belirli bir yelpaze açısı içinde kalan tüm saldırılabilir hedefleri bir vektöre doldurur. Yelpaze açısı, hedeflerin uzaklığına göre dinamik olarak ayarlanır.
    *   **`NEW_GetInstanceVectorInCircleRange(float fSkillDistance, std::vector<CInstanceBase*>* pkVct_pkInst)`**: Karakterin etrafında, `fSkillDistance` yarıçaplı bir daire içinde kalan tüm saldırılabilir hedefleri bir vektöre doldurur.
*   **Tıklama Mesafesi Kontrolleri**:
    *   **`NEW_IsClickableDistanceDestPixelPosition(const TPixelPosition& c_rkPPosDst)`**: Belirtilen pozisyonun karakter tarafından tıklanabilir (etkileşim kurulabilir) bir mesafede olup olmadığını kontrol eder (varsayılan 150.0f birim).
    *   **`NEW_IsClickableDistanceDestInstance(CInstanceBase& rkInstDst)`**: `rkInstDst` karakterinin mevcut karakter tarafından tıklanabilir bir mesafede olup olmadığını kontrol eder. Bu mesafe, hedefin türüne (PC, NPC, okçu karakteri vb.) göre değişir.
*   **Yetenek ve Saldırı Başlatma/Yönetimi**:
    *   **`NEW_UseSkill(UINT uSkill, UINT uMot, UINT uMotLoopCount, bool isMovingSkill)`**: Bir yetenek kullanımını başlatır. Karakterin durumunu (ölü, sersemlemiş vb.) kontrol eder. `isMovingSkill` ise karakteri yürüyüş moduna geçirir. Belirtilen yetenek animasyonunu (`CRaceMotionData::NAME_SKILL + uMot`) oynatır.
    *   **`NEW_Attack()`**: Karakterin mevcut baktığı yöne doğru saldırı başlatır.
    *   **`NEW_Attack(float fDirRot)`**: Belirtilen `fDirRot` açısına doğru saldırı başlatır. Karakterin durumunu kontrol eder. Polimorf durumunda `InputNormalAttack`, değilse `InputComboAttack` çağırır.
    *   **`NEW_AttackToDestPixelPositionDirection(const TPixelPosition& c_rkPPosDst)`**: Belirtilen hedef pozisyona doğru saldırı başlatır.
    *   **`NEW_AttackToDestInstanceDirection(CInstanceBase& rkInstDst)`**: Belirtilen hedef karaktere doğru saldırı başlatır.
    *   **`AttackProcess()`**: `m_GraphicThingInstance.CanCheckAttacking()` (saldırı kontrolü yapılıp yapılamayacağı) doğru ise, çevredeki tüm saldırılabilir karakterleri (`IsAttackableInstance`) kontrol eder ve `CheckAttacking` ile çarpışma/menzil testi yapar. En son etkileşim kurulan hedefi `m_dwLastDmgActorVID`'de saklar.
    *   **`InputNormalAttack(float fAtkDirRot)`**: Normal saldırı komutunu `CActorInstance`'a iletir.
    *   **`InputComboAttack(float fAtkDirRot)`**: Kombo saldırı komutunu `CActorInstance`'a iletir ve `__ComboProcess()`'i çağırır.
    *   **`RunNormalAttack(float fAtkDirRot)`**: Hareketi durdurur ve `CActorInstance` üzerinden normal saldırıyı başlatır.
    *   **`RunComboAttack(float fAtkDirRot, DWORD wMotionIndex)`**: Hareketi durdurur ve `CActorInstance` üzerinden kombo saldırısını (belirli bir kombo adımıyla) başlatır.
*   **Çarpışma ve Engelleme Kontrolleri (`CheckAdvancing`)**:
    *   Karakter hareket halindeyken (özellikle ana karakter) diğer karakterler veya harita nesneleriyle çarpışıp çarpışmadığını kontrol eder.
    *   Eğer `m_GraphicThingInstance.CanSkipCollision()` ise (örneğin, GM karakteri veya bazı özel durumlar), çarpışma kontrolünü atlar.
    *   Diğer karakterlerle çarpışma (`TestActorCollision`) tespit edilirse, hareketi engeller (`BlockMovement()`) veya pozisyonu ayarlar (`AdjustDynamicCollisionMovement`).
    *   Arkaplan (harita) nesneleriyle çarpışma, haritanın öznitelik katmanındaki (`CTerrainImpl::ATTRIBUTE_BLOCK`) bilgilere göre yapılır. Belirli adımlarla ileriye doğru pozisyonlar kontrol edilir.
*   **Saldırı Kontrolü (`CheckAttacking`)**:
    *   Karakterin bir hedefe saldırıp saldıramayacağını kontrol eder. Güvenli bölge (`IsInSafe()`) kontrolleri yapar.
    *   `m_GraphicThingInstance.AttackingProcess()` ile asıl saldırı menzili ve çarpışma testini `CActorInstance` seviyesinde gerçekleştirir.
*   **Durum Sorgulamaları**:
    *   `isNormalAttacking()`, `isComboAttacking()`, `IsUsingSkill()`, `IsUsingMovingSkill()`, `CanCancelSkill()`: `CActorInstance` üzerinden karakterin mevcut saldırı veya yetenek kullanım durumunu sorgular.
    *   `CanAttackHorseLevel()`: Binek üzerindeyken bineğin saldırı yapıp yapamayacağını kontrol eder.
    *   `IsAffect(UINT uAffect)`: Belirli bir etkinin (buff/debuff) karakter üzerinde aktif olup olmadığını kontrol eder.
*   **Vuruş İşleme (`ProcessHitting` - Yorum Satırı Halinde)**: Bu fonksiyonlar (iki overload) normalde bir saldırı animasyonunun belirli bir vuruş anında çağrılarak hedefe hasar verilmesini sağlamak için kullanılır, ancak kaynak kodda yorum satırı halinde bırakılmışlar. Bu işlevsellik muhtemelen `CActorInstance::ProcessHittingEffect()` veya benzer bir mekanizma ile `CActorInstance` tarafında ele alınmaktadır.
*   **Karakterin Ölmesi, Canlanması, Sersemlemesi**:
    *   `Revive()`: Karakteri canlandırır, hareketini durdurur ve at eğerleri gibi detayları tekrar ekler.
    *   `Stun()`: Karakteri sersemletir, hareketini durdurur ve sersemleme efekti ekler.
    *   `Die()`: Karakteri öldürür, at eğerlerini çıkarır, üzerindeki tüm etkileri temizler, seçim/hedefleme efektlerini kaldırır ve ölüm animasyonunu başlatır.
    *   `Hide()` / `Show()`: Karakteri yavaşça görünmez yapar veya görünür hale getirir (alpha blend ile).
*   **Optimizasyon (`ENABLE_OPTIMIZATION`)**:
    *   `AttackProcessOnlySelf()`: Eğer bu makro tanımlıysa ve optimizasyon aktifse, sadece ana karakterin veya ana karaktere saldıranların saldırı süreçleri işlenir. Bu, kalabalık sahnelerde performansı artırabilir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyadaki fonksiyonlar, oyunun dövüş sisteminin temelini oluşturur:

1.  **Hedef Seçimi**: Oyuncu bir canavara tıkladığında, `NEW_IsClickableDistanceDestInstance` ile mesafesi kontrol edilir. Saldırı komutu verildiğinde, `NEW_AttackToDestInstanceDirection` ile karakter hedefe doğru döner ve saldırır.
2.  **Yetenek Kullanımı**: Bir yetenek kullanıldığında, `NEW_UseSkill` çağrılır. Eğer yetenek alan etkiliyse, `NEW_GetInstanceVectorInFanRange` veya `NEW_GetInstanceVectorInCircleRange` ile etki alanındaki hedefler belirlenir.
3.  **Otomatik Saldırı/Hareket**: Canavarlar veya NPC'ler hedeflerine saldırırken veya hareket ederken bu fonksiyonları kullanır. Örneğin, bir canavar oyuncuya saldırırken `NEW_GetDistanceFromDestInstance` ile mesafeyi kontrol eder, menzile girdiğinde `NEW_AttackToDestInstanceDirection` ile saldırır.
4.  **Çarpışma Yönetimi**: Oyuncu hareket ederken, `CheckAdvancing` fonksiyonu sayesinde diğer oyunculara, NPC'lere veya engellere takılır.
5.  **Mesafe ve Açıya Bağlı Yetenekler**: Bazı yeteneklerin çalışması için karakterin hedefe belirli bir açıyla bakması veya belirli bir mesafede olması gerekebilir. Bu kontroller bu dosyadaki fonksiyonlarla yapılır.
6.  **Uçan Projeleriller**: Okçu saldırıları veya bazı büyülerde, `SetFlyTargetInstance` ve ilgili fonksiyonlar kullanılarak merminin (ok, büyü efekti) hedefi belirlenir.

Bu dosya, `CInstanceBase`'in "beyninin" dövüşle ilgili kısmını oluşturur ve `CActorInstance` (grafik ve animasyon) ile `CPythonCharacterManager` (genel karakter yönetimi) arasında bir köprü görevi görür.

### `InstanceBaseEffect.cpp`

#### Dosyanın Genel Amacı

`InstanceBaseEffect.cpp` dosyası, `CInstanceBase` sınıfının karakterler üzerinde ve etrafında görünen çeşitli görsel efektlerin, metin etiketlerinin (isim, seviye, unvan), hasar göstergelerinin ve durum belirteçlerinin (PvP, imparatorluk vb.) yönetimiyle ilgili fonksiyonlarını içerir. Bu dosya, oyun içi görsel geri bildirimlerin ve karakterlerin kimliklerinin önemli bir parçasını oluşturur.

#### Temel Fonksiyonlar ve İşleyiş

*   **Statik Üyeler ve Global Değişkenler**:
    *   `ms_fDustGap`, `ms_fHorseDustGap`: Toz efekti aralıkları.
    *   `ms_adwCRCAffectEffect[CInstanceBase::EFFECT_NUM]`: `EFFECT_NUM` kadar efekt türü için CRC (efekt dosyası) ID'lerini tutan dizi.
    *   `ms_astAffectEffectAttachBone[EFFECT_NUM]`: Her efekt türünün hangi kemik (bone) üzerine ekleneceğini belirten string dizisi.
    *   `g_akD3DXClrTitle[CInstanceBase::TITLE_NUM]`: Unvan renklerini tutan global dizi.
    *   `g_akD3DXClrName[CInstanceBase::NAMECOLOR_NUM]`: İsim renklerini tutan global dizi.
    *   `g_TitleNameMap`: Unvan ID'lerini unvan yazılarına eşleyen global map. (`ENABLE_GENDER_ALIGNMENT` tanımlıysa cinsiyete göre ayrı unvanlar tutabilir).
    *   `g_GuildLeaderGradeNameMap`: Lonca lideri rütbe isimlerini tutan global map (`ENABLE_GUILD_LEADER_GRADE_NAME` tanımlıysa).
    *   `g_kSet_dwPVPReadyKey`, `g_kSet_dwPVPKey`, `g_kSet_dwGVGKey`, `g_kSet_dwDUELKey`: PvP, GvG ve düello durumlarını takip etmek için kullanılan global setler. Her bir set, ilgili etkileşimde olan karakter çiftlerinin ID'lerinden üretilen benzersiz anahtarları tutar.
    *   `g_isEmpireNameMode`: İmparatorluk isim modu aktif mi değil mi belirten global boolean.
*   **Renk ve İsim Modu Yönetimi**:
    *   **`SetEmpireNameMode(bool isEnable)`**: İmparatorluk isim modunu açar/kapatır. Bu mod aktifken, karakter isim renkleri normal PC/NPC/MOB renkleri yerine imparatorluğa özel renklere ayarlanır.
    *   **`GetIndexedNameColor(UINT eNameColor)`**: Belirtilen indeksteki ismi rengini döndürür.
*   **Hasar Efekti Yönetimi**:
    *   **`AddDamageEffect(DWORD damage, BYTE flag, BOOL bSelf, BOOL bTarget)`**: Verilen hasar miktarını ve türünü (flag: kritik, zehir, ıskalama vb.) bir kuyruğa (`m_DamageQueue`) ekler. Bu fonksiyon, hasarın ana karakter tarafından mı verildiği (`bSelf`) veya ana karaktere mi verildiği (`bTarget`) bilgisini de alır. Sadece `CPythonSystem::Instance().IsShowDamage()` aktifse çalışır.
    *   **`ProcessDamage()`**: Hasar kuyruğundan sıradaki hasarı alır ve görsel olarak gösterir.
        *   Iskalama veya blok durumlarında (`DAMAGE_DODGE`, `DAMAGE_BLOCK`) ilgili "miss" efektini oluşturur.
        *   Diğer durumlarda, hasarın türüne (normal, zehir, kanama, yanma) ve hedefe göre (kendine hasar, hedefe hasar) uygun efekt CRC'sini (`rdwCRCEft`) ve doku önekini (`strDamageType`) belirler.
        *   Hasar miktarını rakamlara ayırarak her rakam için ayrı bir doku (`d:/ymir work/effect/affect/damagevalue/[strDamageType][rakam].dds`) kullanır ve bu dokuları `CEffectManager` aracılığıyla hasar efekti üzerine uygular.
        *   Rakamları yan yana, kamera açısına göre düzgün görünecek şekilde konumlandırır.
*   **Genel ve Özel Efektler**:
    *   **`AttachSpecialEffect(DWORD effect, float fScale)`**: Verilen özel efekti karaktere ekler (`__AttachEffect` çağırır).
    *   **`LevelUp()`**: Seviye atlama efektini (`EFFECT_LEVELUP`) ekler.
    *   **`SkillUp()`**: Yetenek geliştirme efektini (`EFFECT_SKILLUP`) ekler.
    *   **`CreateSpecialEffect(DWORD iEffectIndex, float fScale)`**: Belirli bir indeksteki efekti, karakterin mevcut transformasyon matrisini kullanarak oluşturur.
*   **Efekt Konteyneri (`__EffectContainer_...`)**:
    *   Efekt anahtarları (genellikle `EFFECT_` enum değerleri) ile o anda aktif olan efekt ID'lerini eşleştiren bir map (`m_kEffectContainer.m_kDct_dwEftID`) yönetir.
    *   `__EffectContainer_AttachEffect(DWORD dwEftKey)`: Belirtilen anahtarla ilişkili efekti ekler (eğer zaten ekli değilse).
    *   `__EffectContainer_DetachEffect(DWORD dwEftKey)`: Belirtilen anahtarla ilişkili efekti kaldırır.
*   **İmparatorluk, Seçim ve Hedef Efektleri**:
    *   **`__AttachEmpireEffect(DWORD eEmpire)`**: Karakterin imparatorluğuna göre (`EFFECT_EMPIRE + eEmpire`) bir efekt ekler. Ana karakter GM değilse ve hedef farklı imparatorluktaysa bu efekt görünür. Görünmezlik, warp, obje olma gibi durumlarda eklenmez.
    *   **`__AttachSelectEffect()` / `__DetachSelectEffect()`**: Karakter seçildiğinde/seçim kaldırıldığında `EFFECT_SELECT` efektini yönetir.
    *   **`__AttachTargetEffect()` / `__DetachTargetEffect()`**: Karakter hedeflendiğinde/hedef kaldırıldığında `EFFECT_TARGET` efektini yönetir.
*   **Metin Taşı Duman Efektleri (`__StoneSmoke_...`)**:
    *   Metin taşlarının can seviyesine göre (AFFECT_STONE_SMOKE... flag'leri ile) farklı duman efektleri göstermesini sağlar. `m_GraphicThingInstance.AttachSmokeEffect()` ile efekti ekler.
*   **Alfa (Görünürlük) Yönetimi**:
    *   **`SetAlpha(float fAlpha)`**: Karakteri blend moduna alır ve alfa değerini ayarlar.
    *   **`UpdateDeleting()`**: Karakter silinirken yavaşça kaybolmasını (alpha değerini azaltarak) sağlar.
    *   **`DeleteBlendOut()`**: Karakteri silinmeye hazırlarken blend moduna alır, alfa değerini 1.0 yapar ve isim etiketini kaldırır.
*   **PvP Durum Yönetimi (`PVPKeySystem`)**:
    *   `__GetPVPKey(DWORD dwVIDSrc, DWORD dwVIDDst)`: İki karakter VID'sinden (küçük olan önce gelecek şekilde sıralanır) benzersiz bir hash anahtarı üretir. Bu anahtar, iki karakter arasındaki PvP/GvG/Düello durumunu takip etmek için kullanılır.
    *   `InsertPVPKey`, `InsertPVPReadyKey`, `RemovePVPKey`, `InsertGVGKey`, `RemoveGVGKey`, `InsertDUELKey`: İlgili global setlere (örneğin `g_kSet_dwPVPKey`) anahtar ekler veya çıkarır.
    *   `__FindPVPKey`, `__FindPVPReadyKey`, `__FindGVGKey`, `__FindDUELKey`: Verilen karakterler/loncalar arasında ilgili bir durumun (PvP, GvG, Düello) aktif olup olmadığını kontrol eder.
    *   `IsPVPInstance(CInstanceBase& rkInstSel)`: Seçili karakterle PvP veya GvG durumunda olup olmadığını kontrol eder. Düello modu aktifse her zaman `true` döner.
*   **İsim ve Unvan Rengi/İçeriği**:
    *   **`GetNameColor()`**: `GetNameColorIndex()` sonucuna göre uygun ismi rengini döndürür.
    *   **`GetNameColorIndex()`**: Karakterin tipine (PC, NPC, MOB), PK durumuna, ana karakterle ilişkisine (aynı imparatorluk, PvP durumu, parti üyesi), düello durumuna göre uygun isim rengi indeksini belirler.
    *   **`GetTitleColor()`**: Karakterin alignment derecesine (`GetAlignmentGrade()`) göre uygun unvan rengini döndürür.
*   **Metin Etiketi (TextTail) Yönetimi**:
    *   **`AttachTextTail()`**: Karaktere isim, lonca ve seviye etiketlerini ekler. `CPythonTextTail` singleton'ını kullanır. Binek üzerindeyse etiketin yüksekliği ayarlanır. Lonca lideri rütbe adı varsa onu da gösterir.
    *   **`DetachTextTail()`**: Karakterin metin etiketlerini kaldırır.
    *   **`UpdateTextTailLevel(DWORD level)`**: Karakterin seviye etiketini günceller. PC, MOB ve Pet için farklı renkler kullanabilir (`WJ_SHOW_MOB_INFO`, `ENABLE_GROWTH_PET_SYSTEM` tanımlarına göre).
    *   **`UpdateTextTailConquerorLevel(DWORD level)`**: Şampiyon seviyesi etiketini günceller (`ENABLE_CONQUEROR_LEVEL` tanımlıysa).
    *   **`RefreshTextTail()`**: Karakterin isim rengini ve unvanını günceller. Unvan, `g_TitleNameMap`'ten ve karakterin alignment derecesine göre alınır.
    *   **`RefreshTextTailTitle()`**: Sadece `RefreshTextTail()`'i çağırır.
*   **Efekt (Affect) Yönetimi**:
    *   **`__ClearAffectFlagContainer()`**: Karakterin `m_kAffectFlagContainer`'ını temizler.
    *   **`__ClearAffects()`**: Karakter üzerindeki tüm görsel efektleri (`m_adwCRCAffectEffect` dizisindekileri) kaldırır ve `m_kAffectFlagContainer`'ı temizler. Metin taşları için `__StoneSmoke_Destroy()` çağrılır. `CActorInstance::__OnClearAffects()`'i de çağırır.
    *   **`__SetNormalAffectFlagContainer()`**: Yeni bir `CAffectFlagContainer` alarak mevcut etkileri günceller. Değişen her etki için `__SetAffect` ve `CActorInstance::__OnSetAffect`/`__OnResetAffect` çağrılır.
    *   **`__SetStoneSmokeFlagContainer()`**: Metin taşı için duman efektlerini `CAffectFlagContainer`'daki bayraklara göre ayarlar.
    *   **`SetAffectFlagContainer()`**: Gelen `CAffectFlagContainer`'a göre karakterin etkilerini günceller (normal karakterler, metin taşları ve binalar için farklı davranır).
    *   **`SCRIPT_SetAffect(UINT eAffect, bool isVisible)`**: Belirli bir etkiyi görünür/görünmez yapar (`__SetAffect` çağırır).
    *   **`__SetReviveInvisibilityAffect()`**: Canlanma sonrası geçici görünmezlik efektini ayarlar.
    *   **`__Assassin_SetEunhyeongAffect()`**: Suikastçıların "Eunhyeong" (gizlenme) yeteneği efektini ayarlar. Ana karakter veya GM ise yarı saydam, değilse tamamen görünmez yapar ve efektleri gizler.
    *   **`__Shaman_SetParalysis()`**: Şamanların "Paralysis" (felç) efektini ayarlar (`CActorInstance::SetParalysis`).
    *   **`__Warrior_SetGeomgyeongAffect()`**: Savaşçıların "Geomgyeong" (Kılıç Aurası) yeteneği efektini ayarlar. Silah parlama efekti ekler ve saldırı menzilini artırır (`CActorInstance::SetReachScale`).
    *   **`__SetSoulAffect()`**: Ruh taşı efektlerini (`AFFECT_SOUL_RED`, `AFFECT_SOUL_BLUE`, `AFFECT_SOUL_MIX`) ayarlar (`ENABLE_SOUL_SYSTEM` tanımlıysa).
    *   **`__SetAffect(UINT eAffect, bool isVisible)`**: Belirli bir etki türü için (`AFFECT_` enum değerleri) ilgili efekti (`EFFECT_AFFECT + eAffect`) ekler veya kaldırır. Bazı özel etkiler (YMIR, CHEONGEUN, EUNHYEONG, STUN vb.) için özel durum kontrolleri içerir.
*   **Emoticon (İfade Simgesi) Yönetimi**:
    *   **`IsPossibleEmoticon()`**: Yeni bir emoticon gösterilip gösterilemeyeceğini kontrol eder (önceki emoticon üzerinden belirli bir süre geçip geçmediğine bakar).
    *   **`SetFishEmoticon()`**: Balık tutma emoticonunu gösterir.
    *   **`SetFishingFailEmoticon()`**: Başarısız balık tutma emoticonunu gösterir (`ENABLE_FISHING_RENEWAL` tanımlıysa).
    *   **`SetEmoticon(UINT eEmoticon)`**: Belirtilen emoticonu (`EFFECT_EMOTICON + eEmoticon`) karakterin başının üzerinde gösterir.
*   **Toz Efekti Yönetimi**:
    *   **`SetDustGap(float fDustGap)`**, **`SetHorseDustGap(float fDustGap)`**: Yürüme ve atla gitme sırasındaki toz efektlerinin ne sıklıkta çıkacağını ayarlar.
*   **Efekt Ekleme/Kaldırma Yardımcıları**:
    *   **`__DetachEffect(DWORD dwEID)`**: Verilen efekt ID'sini `CActorInstance` üzerinden kaldırır.
    *   **`__AttachEffect(UINT eEftType, float fScale)`**: Belirtilen türdeki efekti (`ms_adwCRCAffectEffect[eEftType]`) karaktere ekler. `ms_astAffectEffectAttachBone` dizisindeki bilgiye göre belirli bir kemiğe veya genel olarak karaktere ekleyebilir. Görünmezlik durumunda eklenmez.
*   **Kombo Efekti (`__ComboProcess`)**: Yorum satırı halinde bırakılmış bir fonksiyondur. Muhtemelen kombo vuruşlarında özel efektler (örneğin, AFFECT_HWAYEOM varsa ateş efekti) göstermek için tasarlanmıştı.
*   **Efekt, Unvan ve Renk Kayıt Fonksiyonları**:
    *   **`RegisterEffect(UINT eEftType, const char* c_szEftAttachBone, const char* c_szEftName, bool isCache)`**: Belirli bir efekt türü için efekt dosyasını (`c_szEftName`) ve ekleneceği kemiği kaydeder. `CEffectManager::RegisterEffect2` ile efekt CRC'sini alır ve `ms_adwCRCAffectEffect` dizisine atar.
    *   **`RegisterTitleName(...)`**: Unvan ID'lerini unvan metinlerine eşler.
    *   **`RegisterGuildLeaderGradeName(...)`**: Lonca lideri rütbe ID'lerini rütbe metinlerine eşler.
    *   **`RegisterNameColor(...)`**: İsim rengi indekslerini RGB değerlerine eşler.
    *   **`RegisterTitleColor(...)`**: Unvan rengi indekslerini RGB değerlerine eşler.

#### Oyunda Kullanım Amacı ve Senaryoları

1.  **Görsel Geri Bildirimler**: Oyuncu seviye atladığında (`LevelUp()`), yetenek geliştirdiğinde (`SkillUp()`), hasar aldığında veya verdiğinde (`AddDamageEffect`/`ProcessDamage()`) çeşitli görsel efektler bu dosyadaki fonksiyonlar aracılığıyla gösterilir.
2.  **Karakter Kimliği**: Karakterlerin isimleri, seviyeleri, unvanları ve lonca bilgileri `AttachTextTail` ve ilgili güncelleme fonksiyonları (`RefreshTextTail`, `UpdateTextTailLevel`) ile karakterlerin üzerinde gösterilir. İsim ve unvan renkleri, karakterin durumu, imparatorluğu ve PvP ilişkilerine göre dinamik olarak değişir.
3.  **Durum Efektleri (Buff/Debuff)**: Zehirlenme, yavaşlama, hızlanma gibi durum etkileri (`__SetAffect`) karakter üzerinde ilgili görsel efektlerin (parlama, ikon vb.) çıkmasını sağlar.
4.  **Özel Yetenek Efektleri**: Savaşçıların Kılıç Aurası, Suikastçıların Gizlenmesi gibi yeteneklerin özel görsel sunumları bu dosyadaki fonksiyonlarla yönetilir.
5.  **PvP ve İmparatorluk Tanımlaması**: Karakterler arasındaki PvP durumu veya farklı imparatorluklara ait olma durumu, isim renkleri ve özel imparatorluk efektleri (`__AttachEmpireEffect`) ile oyuncuya bildirilir.
6.  **Emoticonlar**: Oyuncuların veya NPC'lerin duygularını ifade etmek için kullandığı ifade simgeleri (`SetEmoticon`) bu dosya üzerinden yönetilir.
7.  **Başlangıç Ayarları**: Oyun ilk açıldığında `RegisterEffect`, `RegisterTitleName`, `RegisterNameColor` gibi fonksiyonlar çağrılarak sistemde kullanılacak tüm efektler, unvanlar ve renkler tanımlanır. Bu tanımlamalar genellikle Python tarafından tetiklenir.

Bu dosya, `CInstanceBase`'in görsel sunum katmanının önemli bir bölümünü oluşturur ve oyun dünyasını daha canlı ve anlaşılır hale getiren birçok detayı yönetir.

--- 

*(Bu bölümdeki içerikler tamamlanmıştır. Devamı için [UserInterface Referans Kılavuzu - Bölüm 4](./client_UserInterface_Referans_Part4.md), [UserInterface Referans Kılavuzu - Bölüm 5](./client_UserInterface_Referans_Part5.md) ve [UserInterface Referans Kılavuzu - Bölüm 6](./client_UserInterface_Referans_Part6.md) dosyalarına bakınız.)*