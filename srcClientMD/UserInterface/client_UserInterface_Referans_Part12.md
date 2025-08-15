# UserInterface Referans Kılavuzu - Bölüm 12

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonNetworkStreamPhaseHandShake.cpp` (Ağ Akışı El Sıkışma Aşaması)](#pythonnetworkstreamphasehandshakecpp-ağ-akışı-el-sıkışma-aşaması)
*   [`PythonNetworkStreamPhaseLoading.cpp` (Ağ Akışı Yükleme Aşaması)](#pythonnetworkstreamphaseloadingcpp-ağ-akışı-yükleme-aşaması)
*   [`PythonNetworkStreamPhaseLogin.cpp` (Ağ Akışı Giriş Aşaması)](#pythonnetworkstreamphaselogincpp-ağ-akışı-giriş-aşaması)
*   [`PythonNetworkStreamPhaseOffline.cpp` (Ağ Akışı Çevrimdışı Aşaması)](#pythonnetworkstreamphaseofflinecpp-ağ-akışı-çevrimdışı-aşaması)
*   [`PythonNetworkStreamPhaseSelect.cpp` (Ağ Akışı Karakter Seçim Aşaması)](#pythonnetworkstreamphaseselectcpp-ağ-akışı-karakter-seçim-aşaması)
*   [`PythonNonPlayer.h` (NPC/Mob Veri Tanımları)](#pythonnonplayerh-npcmob-veri-tanımları)
*   [`PythonNonPlayer.cpp` (NPC/Mob Veri Yönetimi)](#pythonnonplayercpp-npcmob-veri-yönetimi)
*   [`PythonNonPlayerModule.cpp` (NPC/Mob Python Modülü)](#pythonnonplayermodulecpp-npcmob-python-modülü)
*   [`PythonPackModule.cpp` (Paket Sistemi Python Modülü)](#pythonpackmodulecpp-paket-sistemi-python-modülü)
*   [`PythonPlayer.h` (Oyuncu Sınıfı Tanımları)](#pythonplayerh-oyuncu-sınıfı-tanımları)
*   [`PythonPlayer.cpp` (Oyuncu Sınıfı Uygulaması)](#pythonplayercpp-oyuncu-sınıfı-uygulaması)
*   [`PythonPlayerEventHandler.h` (Oyuncu Olay İşleyicisi Tanımları)](#pythonplayereventhandlerh-oyuncu-olay-i̇şleyicisi-tanımları)
*   [`PythonPlayerEventHandler.cpp` (Oyuncu Olay İşleyicisi Uygulaması)](#pythonplayereventhandlercpp-oyuncu-olay-i̇şleyicisi-uygulaması)

---

### `PythonNetworkStreamPhaseHandShake.cpp` (Ağ Akışı El Sıkışma Aşaması)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının sunucu ile istemci arasında ilk bağlantı kurulduktan sonra gerçekleşen "HandShake" (El Sıkışma) aşamasını yöneten kısmını içerir. Bu aşamanın temel amacı, istemci ile sunucu arasındaki zaman senkronizasyonunu sağlamak ve paket şifrelemesi için gerekli anahtarları değiştirmektir.

#### Dosyanın Temel Amaçları

*   **Aşama Yönetimi (`SetHandShakePhase`, `HandShakePhase`)**:
    *   `SetHandShakePhase()`: Ağ akışının mevcut durumunu "HandShake" olarak ayarlar, ilgili işlem (`m_phaseProcessFunc`) ve çıkış (`m_phaseLeaveFunc`) fonksiyonlarını belirler. Eğer `__DirectEnterMode_IsSet()` aktif değilse, Python tarafındaki `PHASE_WINDOW_LOGIN` penceresinin `OnHandShake` fonksiyonunu çağırır.
    *   `HandShakePhase()`: Bu aşamada gelen ağ paketlerini işleyen ana döngüdür. `CheckPacket()` ile paketleri kontrol eder ve gelen paketin başlığına göre ilgili işleyiciyi çağırır.
*   **Zaman Senkronizasyonu**:
    *   `HEADER_GC_HANDSHAKE` paketi alındığında (`RecvHandshakePacket()` içinde de benzer mantık var, ancak `HandShakePhase` içindeki daha temel bir ilk senkronizasyon gibi duruyor): Sunucudan gelen zaman bilgisi (`m_HandshakeData.dwTime`) ve gecikme (`m_HandshakeData.lDelta`) kullanılarak istemcinin sunucu zamanına senkronize olması sağlanır (`ELTimer_SetServerMSec`). Ardından istemci, güncellenmiş zaman bilgisini sunucuya geri gönderir. `CTimer::Instance().SetBaseTime()` ile istemci tarafındaki zamanlayıcı sıfırlanır.
    *   `HEADER_GC_HANDSHAKE_OK` paketi (`RecvHandshakeOKPacket()`): Sunucudan zaman senkronizasyonunun başarılı olduğuna dair onay paketi geldiğinde, istemci ile sunucu arasındaki son gecikme hesaplanarak istemci zamanı bir kez daha ayarlanır.
*   **Şifreleme Anahtarı Değişimi**:
    *   `HEADER_GC_HYBRIDCRYPT_KEYS` (`RecvHybridCryptKeyPacket()`): Sunucudan gelen hibrit şifreleme anahtarlarını alır ve `CEterPackManager::Instance().RetrieveHybridCryptPackKeys()` aracılığıyla EterPack yöneticisine bu anahtarları yükler. Bu, istemcinin şifreli paketleri okuyabilmesi için gereklidir.
    *   `HEADER_GC_HYBRIDCRYPT_SDB` (`RecvHybridCryptSDBPacket()`): Hibrit şifreleme için ek "Seed Database" (SDB) verilerini alır ve `CEterPackManager::Instance().RetrieveHybridCryptPackSDB()` ile EterPack yöneticisine yükler.
    *   `__IMPROVED_PACKET_ENCRYPTION__` makrosu tanımlıysa, ek olarak `HEADER_GC_KEY_AGREEMENT` (`RecvKeyAgreementPacket()`) ve `HEADER_GC_KEY_AGREEMENT_COMPLETED` (`RecvKeyAgreementCompletedPacket()`) paketleri aracılığıyla daha gelişmiş bir anahtar anlaşma süreci işletilir. Bu süreçte istemci kendi anahtarını hazırlar (`Prepare()`), sunucudan gelen anahtarla birleştirir (`Activate()`) ve şifrelemeyi aktif hale getirir (`ActivateCipher()`).
*   **Diğer Paketler**:
    *   `HEADER_GC_PHASE`: Başka bir aşamaya geçiş paketini işler.
    *   `HEADER_GC_BINDUDP`: UDP bağlantısı için bilgi alır (kullanımı yorum satırında).
    *   `HEADER_GC_PING`: Ping paketini işler.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::HandShakePhase()`**:
    *   Bu fonksiyon, "HandShake" aşamasındayken sürekli çağrılır.
    *   Gelen paket başlıklarına göre `switch` ifadesiyle ilgili paket işleme fonksiyonlarını (örn: `RecvHandshakePacket`, `RecvHybridCryptKeyPacket`) veya doğrudan işlemleri tetikler.
*   **`CPythonNetworkStream::SetHandShakePhase()`**:
    *   Ağ yöneticisini "HandShake" moduna geçirir.
    *   Bu modda hangi fonksiyonun (`HandShakePhase`) paketleri işleyeceğini ve bu moddan çıkıldığında hangi fonksiyonun (`__LeaveHandshakePhase`) çağrılacağını ayarlar.
    *   Python tarafındaki UI'a "HandShake" başladığını bildirir.
*   **`CPythonNetworkStream::RecvHandshakePacket()` ve `CPythonNetworkStream::RecvHandshakeOKPacket()`**:
    *   İstemci ve sunucu arasında zaman damgalarını değiş tokuş ederek zaman farkını en aza indirir ve istemcinin zamanını sunucu zamanıyla senkronize eder. `m_kServerTimeSync` yapısı bu senkronizasyon için kullanılır.
*   **`CPythonNetworkStream::RecvHybridCryptKeyPacket()` ve `CPythonNetworkStream::RecvHybridCryptSDBPacket()`**:
    *   Bu fonksiyonlar, sunucudan gelen şifreleme anahtarlarını ve ilgili verileri alarak, istemcinin `.epk/.eix` gibi paketlenmiş dosyaları doğru bir şekilde açabilmesi ve okuyabilmesi için `CEterPackManager`'a iletir. Şifreli client dosyalarının okunması için bu adım kritiktir.
*   **Gelişmiş Şifreleme (eğer aktifse)**:
    *   `RecvKeyAgreementPacket()` ve `RecvKeyAgreementCompletedPacket()` fonksiyonları, daha güvenli bir paket şifreleme katmanı için Diffie-Hellman benzeri bir anahtar anlaşma protokolünü uygular. Bu, sunucu ve istemcinin güvenli bir şekilde ortak bir şifreleme anahtarı oluşturmasını sağlar.

Bu dosya, istemcinin sunucuyla güvenli ve senkronize bir iletişim kurmasının ilk adımlarını oluşturur. Başarılı bir "HandShake" aşaması, sonraki aşamaların (Login, Loading, Game) sorunsuz işlemesi için temel teşkil eder.

---

### `PythonNetworkStreamPhaseLoading.cpp` (Ağ Akışı Yükleme Aşaması)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının "Loading" (Yükleme) aşamasını yöneten kısmını içerir. Bu aşama, oyuncu karakter seçimi yaptıktan ve oyuna giriş komutunu gönderdikten sonra, oyun dünyasının ve oyuncu verilerinin yüklenmesi sırasında aktif olur.

#### Dosyanın Temel Amaçları

*   **Aşama Yönetimi (`SetLoadingPhase`, `LoadingPhase`)**:
    *   `SetLoadingPhase()`: Ağ akışının mevcut durumunu "Loading" olarak ayarlar. `CPythonPlayer` verilerini temizler, efekt ve uçan nesne yöneticilerini sıfırlar. `m_phaseProcessFunc` ve `m_phaseLeaveFunc` ayarlanır.
    *   `LoadingPhase()`: Bu aşamada gelen ağ paketlerini işleyen ana döngüdür. Genellikle ana karakter verilerini, başlangıç eşyalarını ve harita bilgilerini bekler.
*   **Ana Karakter Verilerini Alma**:
    *   `HEADER_GC_MAIN_CHARACTER` ve versiyonları (`...2_EMPIRE`, `...3_BGM`, `...4_BGM_VOL`): Sunucudan ana karakterin temel bilgilerini (VID, Irk, İmparatorluk, Beceri Grubu, İsim, Başlangıç Koordinatları, BGM bilgisi) alır.
    *   `m_dwMainActorVID`, `m_dwMainActorRace` gibi sınıf üyelerini ayarlar.
    *   `CPythonPlayer::Instance()->SetName()` ve `SetMainCharacterIndex()` ile oyuncu bilgilerini setler.
    *   `PyCallClassMemberFunc(m_apoPhaseWnd[PHASE_WINDOW_LOAD], "LoadData", ...)` ile Python tarafındaki yükleme ekranına harita koordinatlarını göndererek harita yüklemesini tetikler veya `ENABLE_LOADING_PERFORMANCE` aktifse `CPythonLoading::Instance().LoadBackground()` çağırır.
    *   `Warp()` fonksiyonunu çağırarak karakteri başlangıç pozisyonuna ışınlar (arka plan yüklemesi vb.).
    *   `SendClientVersionPacket()` ile istemci versiyon bilgisini sunucuya gönderir.
*   **Başlangıç Verilerini Alma**:
    *   `HEADER_GC_PLAYER_POINTS` (`__RecvPlayerPoints()`): Oyuncunun tüm stat puanlarını (HP, MP, STR, DEX vb.) alır ve `CPythonPlayer::Instance()->SetStatus()` ile ayarlar.
    *   `HEADER_GC_ITEM_SET_EMPTY`: Envanterdeki boş bir slota temel eşya bilgisi (efsunsuz, bayraksız) geldiğinde işler.
    *   `HEADER_GC_QUICKSLOT_ADD`: Hızlı erişim slotu bilgilerini alır.
*   **Diğer Yükleme İşlemleri**:
    *   `EnableChatInsultFilter()`, `__FilterInsult()`, `IsChatInsultIn()`, `IsInsultIn()`, `LoadInsultList()`: Sohbet için küfür filtresiyle ilgili fonksiyonlardır. `locale/xx/insult.txt` gibi bir dosyadan yasaklı kelimeleri yükler.
    *   `LoadConvertTable()`: Farklı imparatorlukların birbirlerinin dilini (eğer aktifse) şifreli görmesi için kullanılan karakter dönüşüm tablolarını yükler (`locale/xx/empire_language_convert.txt` gibi).
    *   `LoadLocaleQuestVnum()`, `GetLocaleQuestVnum()`: NPC konuşmalarında veya görevlerde geçen özel yerelleştirilmiş metinleri (`[VNUM]Metin`) `locale/xx/locale_quest.txt` dosyasından yükler ve ID'ye göre getirir.
    *   `ENABLE_LOADING_TIP` aktifse:
        *   `LoadLoadingTipList()`: Yükleme ekranında gösterilecek ipuçlarının hangi haritalarda görüneceğini tanımlayan listeyi (`loading_tip_list.txt`) yükler.
        *   `LoadLoadingTipVnum()`: İpuçlarının metinlerini ID'lerine göre (`loading_tip_vnum.txt`) yükler.
        *   `GetLoadingTipVnum()`: Mevcut haritaya uygun rastgele bir ipucu metni döndürür.
    *   `__SetFieldMusicFileName()`, `__SetFieldMusicFileInfo()`, `GetFieldMusicFileName()`, `GetFieldMusicVolume()`: Haritaya özel arka plan müziği adını ve ses seviyesini yönetir.
*   **Oyuna Giriş**:
    *   `StartGame()`: `m_isStartGame` bayrağını `TRUE` yaparak oyunun başlamaya hazır olduğunu belirtir (genellikle tüm yükleme paketleri alındıktan sonra `GamePhase`'e geçişi tetikler).
    *   `SendEnterGame()`: Sunucuya oyuna giriş yapıldığını bildiren `HEADER_CG_ENTERGAME` paketini gönderir.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::LoadingPhase()`**:
    *   "Loading" aşamasında gelen paketleri `switch` ile yönlendirir. Çoğunlukla karakter, eşya ve stat bilgilerini bekler. Bu paketler alındıktan sonra genellikle `GamePhase()`'e geçilir.
*   **`CPythonNetworkStream::SetLoadingPhase()`**:
    *   Ağ yöneticisini "Loading" moduna geçirir.
    *   Önceki aşamanın kaynaklarını temizler (`m_phaseLeaveFunc.Run()`).
    *   Oyuncu verilerini (`CPythonPlayer::Instance()->Clear()`) ve diğer oyun dünyası elemanlarını (efektler, uçan nesneler) sıfırlar.
*   **`CPythonNetworkStream::RecvMainCharacter()` ve varyasyonları**:
    *   Oyuncunun oyuna girdiği karakterin ana verilerini alır. Bu, istemcinin hangi karakteri kontrol edeceğini, nerede başlayacağını ve temel özelliklerini bilmesini sağlar.
    *   Arka plan yüklemesini ve karakterin başlangıç pozisyonuna "warp" edilmesini tetikler.
*   **`CPythonNetworkStream::__RecvPlayerPoints()`**:
    *   Karakterin tüm temel statlarını (HP, SP, STR, DEX vb.) tek bir paketle alır ve oyuncu nesnesine işler.
*   **Yerelleştirme ve Filtre Yükleyicileri**:
    *   `LoadInsultList`, `LoadConvertTable`, `LoadLocaleQuestVnum`: Bu fonksiyonlar, oyunun yerelleştirme ve kullanıcı deneyimiyle ilgili önemli metin dosyalarını (`.txt`) `CEterPackManager` aracılığıyla okuyup işler ve hafızaya alır.
    *   Bu veriler daha sonra oyun içinde sohbet filtresi, farklı imparatorluk dilleri veya görev metinleri için kullanılır.

Bu dosya, oyuncunun karakterini ve oyun dünyasını yüklemek için gerekli olan kritik verilerin sunucudan alınmasını ve istemci tarafında işlenmesini sağlar. Yükleme tamamlandığında ve `SendEnterGame()` ile sunucuya bildirim yapıldığında, ağ akışı genellikle "Game" (Oyun) aşamasına geçer. 

---

### `PythonNetworkStreamPhaseLogin.cpp` (Ağ Akışı Giriş Aşaması)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının "Login" (Giriş) aşamasını yöneten kısmını içerir. Bu aşama, oyuncu kullanıcı adı ve şifresini girip sunucuya bağlanmaya çalıştığında aktif olur. Başarılı girişin ardından genellikle "Select" (Karakter Seçim) aşamasına geçilir.

#### Dosyanın Temel Amaçları

*   **Aşama Yönetimi (`SetLoginPhase`, `LoginPhase`)**:
    *   `SetLoginPhase()`: Ağ akışının mevcut durumunu "Login" olarak ayarlar. Eğer `__IMPROVED_PACKET_ENCRYPTION__` tanımlı değilse, `LocaleService_GetSecurityKey()` ile alınan anahtarla güvenlik modunu başlatır. `m_phaseProcessFunc` ve `m_phaseLeaveFunc` ayarlanır. Eğer `__DirectEnterMode_IsSet()` aktifse doğrudan giriş bilgilerini gönderir, değilse Python tarafındaki `PHASE_WINDOW_LOGIN`'in `OnLoginStart` fonksiyonunu çağırır ve karakter seçim verilerini temizler.
    *   `LoginPhase()`: Bu aşamada gelen ağ paketlerini işleyen ana döngüdür. `CheckPacket()` ile paketleri kontrol eder ve gelen paketin başlığına göre ilgili işleyiciyi çağırır.
*   **Giriş Paketlerini Gönderme**:
    *   `SendLoginPacket(const char* c_szName, const char* c_szPassword)`: Sunucuya standart `HEADER_CG_LOGIN` paketi ile kullanıcı adı ve şifreyi gönderir.
    *   `SendLoginPacketNew(const char* c_szName, const char* c_szPassword)`: Sunucuya `HEADER_CG_LOGIN2` paketi ile kullanıcı adını, `m_dwLoginKey` (sunucudan daha önce alınmışsa) ve istemci şifreleme anahtarlarını (`g_adwEncryptKey`) gönderir. Ardından, eğer `__IMPROVED_PACKET_ENCRYPTION__` tanımlı değilse, istemcinin şifreleme modunu bu anahtarlarla günceller.
*   **Gelen Giriş Yanıtlarını İşleme**:
    *   `HEADER_GC_LOGIN_SUCCESS3` (`__RecvLoginSuccessPacket3()`): Başarılı giriş yanıtı (3 karakter slotlu versiyon). Oyuncunun hesap bilgilerini (`m_akSimplePlayerInfo`), lonca ID'lerini (`m_adwGuildID`) ve lonca isimlerini (`m_astrGuildName`) alır. Sunucudan gelen `handle` ve `random_key` değerlerini `m_kMarkAuth` yapısında saklar. Python tarafındaki `PHASE_WINDOW_SELECT`'in `Refresh` fonksiyonunu çağırarak karakter seçim ekranını günceller.
    *   `HEADER_GC_LOGIN_SUCCESS4` (`__RecvLoginSuccessPacket4()`): Başarılı giriş yanıtı (4 karakter slotlu versiyon). `__RecvLoginSuccessPacket3` ile benzer şekilde çalışır, sadece `PLAYER_PER_ACCOUNT4` sabitini kullanır.
    *   `HEADER_GC_LOGIN_FAILURE` (`__RecvLoginFailurePacket()`): Başarısız giriş yanıtı. Sunucudan gelen hata mesajını alır ve Python tarafındaki `PHASE_WINDOW_LOGIN`'in `OnLoginFailure` fonksiyonunu bu mesajla çağırır.
    *   `HEADER_GC_EMPIRE` (`__RecvEmpirePacket()`): Oyuncunun bağlı olduğu imparatorluk bilgisini alır ve `m_dwEmpireID` değişkenine atar.
    *   `HEADER_GC_LOGIN_KEY` (`__RecvLoginKeyPacket()`): Sunucudan bir sonraki giriş denemesi için kullanılacak olan `m_dwLoginKey`'i alır. `m_isWaitLoginKey` bayrağını `FALSE` yapar.
*   **Bağlantı Hatalarını Yönetme**:
    *   `OnConnectFailure()`: Sunucuya bağlanılamadığında çağrılır. Eğer `__DirectEnterMode_IsSet()` aktifse aşamayı kapatır, değilse Python tarafındaki `PHASE_WINDOW_LOGIN`'in `OnConnectFailure` fonksiyonunu çağırır.
*   **Diğer Paketler**:
    *   `HEADER_GC_PHASE`: Başka bir aşamaya geçiş paketini işler.
    *   `HEADER_GC_PING`: Ping paketini işler.
    *   `HEADER_GC_HYBRIDCRYPT_KEYS`, `HEADER_GC_HYBRIDCRYPT_SDB`: "HandShake" aşamasında olduğu gibi hibrit şifreleme anahtarlarını ve SDB verilerini alır.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::LoginPhase()`**:
    *   "Login" aşamasındayken sürekli çağrılır ve sunucudan gelen paketleri `switch` ile ilgili fonksiyonlara yönlendirir.
*   **`CPythonNetworkStream::SetLoginPhase()`**:
    *   Ağ yöneticisini "Login" moduna geçirir.
    *   Gerekli ilk paketleri (kullanıcı adı/şifre) sunucuya gönderir.
    *   Python UI'ına giriş sürecinin başladığını bildirir.
*   **`CPythonNetworkStream::SendLoginPacket()` ve `CPythonNetworkStream::SendLoginPacketNew()`**:
    *   Kullanıcının girdiği bilgileri ve istemci tarafındaki bazı güvenlik bilgilerini sunucuya iletir. `SendLoginPacketNew`, `m_dwLoginKey` kullanarak daha güvenli bir giriş yöntemi sunabilir.
*   **`CPythonNetworkStream::__RecvLoginSuccessPacket3()` ve `CPythonNetworkStream::__RecvLoginSuccessPacket4()`**:
    *   Giriş başarılı olduğunda sunucudan gelen karakter listesini ve diğer hesap bilgilerini alır, bunları C++ tarafında saklar ve Python'daki karakter seçim ekranının güncellenmesini tetikler.
*   **`CPythonNetworkStream::__RecvLoginFailurePacket()`**:
    *   Giriş başarısız olduğunda (yanlış şifre, banlı hesap vb.), sunucudan gelen hata mesajını Python UI'ına iletir.

Bu dosya, oyuncunun oyuna giriş yapabilmesi için sunucuyla ilk kimlik doğrulama adımlarını gerçekleştirir ve başarılı bir girişin ardından karakter seçim ekranı için gerekli verileri hazırlar.

---

### `PythonNetworkStreamPhaseOffline.cpp` (Ağ Akışı Çevrimdışı Aşaması)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının "Offline" (Çevrimdışı) aşamasını yöneten çok kısa bir bölümünü içerir. Bu aşama, istemcinin sunucuyla herhangi bir aktif bağlantısının olmadığı veya bağlantının koptuğu durumları temsil eder.

#### Dosyanın Temel Amaçları

*   **Aşama Yönetimi (`OffLinePhase`)**:
    *   `OffLinePhase()`: Bu aşamada gelen ağ paketlerini (eğer varsa, ki pek olası değildir) işleyen ana döngüdür.
    *   Pratikte, bu fonksiyon sadece `HEADER_GC_PHASE` paketini kontrol eder. Bu, sunucudan istemcinin durumunu değiştirmesi için bir komut gelirse (örneğin, bağlantı tekrar kuruldu ve başka bir aşamaya geçiliyor) bunu yakalamak içindir.
    *   Diğer tüm paketler `RecvErrorPacket` ile hata olarak işlenir.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::OffLinePhase()`**:
    *   "Offline" durumundayken çağrılır.
    *   Temelde, sadece istemcinin tekrar çevrimiçi bir aşamaya geçmesini sağlayacak `HEADER_GC_PHASE` paketini bekler.
    *   Bu aşamada genellikle istemci sunucuya herhangi bir paket göndermez ve sunucudan da çok sınırlı sayıda paket beklenir (çoğunlukla hiç beklenmez).

Bu dosya, istemcinin ağ bağlantısı kesildiğinde veya henüz kurulmadığında bulunduğu durumu yönetir. Oldukça pasif bir aşamadır ve temel amacı, istemcinin tekrar bir ağ bağlantısı kurup aktif bir aşamaya geçmesini beklemektir. `SetOffLinePhase` gibi bir fonksiyonun olmaması, bu aşamaya genellikle bağlantı koptuğunda veya istemci ilk başlatıldığında varsayılan olarak girildiğini düşündürür.

---

### `PythonNetworkStreamPhaseSelect.cpp` (Ağ Akışı Karakter Seçim Aşaması)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının "Select" (Karakter Seçim) aşamasını yöneten kısmını içerir. Bu aşama, oyuncu başarılı bir şekilde giriş yaptıktan sonra karakterlerini gördüğü, yeni karakter oluşturabildiği, karakter silebildiği veya bir karakter seçerek oyuna başlayabildiği ekranda aktif olur.

#### Dosyanın Temel Amaçları

*   **Aşama Yönetimi (`SetSelectPhase`, `SelectPhase`)**:
    *   `SetSelectPhase()`: Ağ akışının mevcut durumunu "Select" olarak ayarlar. Eğer `ENABLE_LOADING_PERFORMANCE` aktifse, önceki yükleme iş parçacığının tamamlanmasını bekler (`CPythonLoading::Instance().JoinThreadLoading()`). `m_phaseProcessFunc` ve `m_phaseLeaveFunc` ayarlanır. Eğer `__IMPROVED_PACKET_ENCRYPTION__` tanımlı değilse, şifreleme anahtarlarıyla güvenlik modunu ayarlar. Oyuncunun daha önce imparatorluk seçip seçmediğine göre Python tarafında `SetSelectCharacterPhase` veya `SetSelectEmpirePhase` fonksiyonlarını çağırır.
    *   `SelectPhase()`: Bu aşamada gelen ağ paketlerini işleyen ana döngüdür.
*   **Sunucuya İstek Gönderme**:
    *   `SendSelectEmpirePacket(DWORD dwEmpireID)`: Oyuncunun seçtiği imparatorluk bilgisini (`HEADER_CG_EMPIRE`) sunucuya gönderir.
    *   `SendSelectCharacterPacket(BYTE Index)`: Oyuncunun seçtiği karakterin slot indeksini (`HEADER_CG_PLAYER_SELECT`) sunucuya gönderir.
    *   `SendDestroyCharacterPacket(BYTE index, const char* szPrivateCode)`: Karakter silme isteğini (`HEADER_CG_PLAYER_DESTROY`), karakterin slot indeksi ve silme şifresi (sosyal kimlik numarası) ile sunucuya gönderir.
    *   `SendCreateCharacterPacket(BYTE index, const char* name, BYTE job, BYTE shape, BYTE byCON, BYTE byINT, BYTE bySTR, BYTE byDEX)`: Yeni karakter oluşturma isteğini (`HEADER_CG_PLAYER_CREATE`), seçilen slot, isim, sınıf, görünüm ve başlangıç stat puanları ile sunucuya gönderir.
    *   `SendChangeNamePacket(BYTE index, const char* name)`: Karakter isim değiştirme isteğini (`HEADER_CG_CHANGE_NAME`), slot indeksi ve yeni isim ile sunucuya gönderir.
*   **Sunucudan Gelen Yanıtları İşleme**:
    *   `HEADER_GC_EMPIRE` (`__RecvEmpirePacket()`): Oyuncunun imparatorluk seçiminin sunucu tarafından onaylandığını ve `m_dwEmpireID`'nin güncellendiğini işler (bu fonksiyon Login aşamasındaki ile aynıdır).
    *   `HEADER_GC_LOGIN_SUCCESS3` / `HEADER_GC_LOGIN_SUCCESS4` (`__RecvLoginSuccessPacket3()`, `__RecvLoginSuccessPacket4()`): Bu paketler normalde Login aşamasında alınır, ancak bir şekilde Select aşamasında da gelirse karakter listesini günceller.
    *   `HEADER_GC_PLAYER_CREATE_SUCCESS` (`__RecvPlayerCreateSuccessPacket()`): Karakter oluşturma başarılı yanıtı. Sunucudan gelen yeni karakter bilgilerini (`kSimplePlayerInfomation`) alır, `m_akSimplePlayerInfo` dizisinde ilgili slota kaydeder ve Python tarafındaki `PHASE_WINDOW_CREATE`'in `OnCreateSuccess` fonksiyonunu çağırır.
    *   `HEADER_GC_PLAYER_CREATE_FAILURE` (`__RecvPlayerCreateFailurePacket()`): Karakter oluşturma başarısız yanıtı. Sunucudan gelen hata tipini alır ve Python tarafındaki `PHASE_WINDOW_CREATE` ve `PHASE_WINDOW_SELECT`'in `OnCreateFailure` fonksiyonlarını çağırır.
    *   `HEADER_GC_PLAYER_DELETE_WRONG_SOCIAL_ID` (`__RecvPlayerDestroyFailurePacket()`): Karakter silme başarısız (yanlış silme şifresi) yanıtı. Python tarafındaki `PHASE_WINDOW_SELECT`'in `OnDeleteFailure` fonksiyonunu çağırır.
    *   `HEADER_GC_PLAYER_DELETE_SUCCESS` (`__RecvPlayerDestroySuccessPacket()`): Karakter silme başarılı yanıtı. Sunucudan gelen silinen karakterin slot indeksini alır, `m_akSimplePlayerInfo` dizisinde ilgili slotu temizler ve Python tarafındaki `PHASE_WINDOW_SELECT`'in `OnDeleteSuccess` fonksiyonunu çağırır.
    *   `HEADER_GC_CHANGE_NAME` (`__RecvChangeName()`): İsim değişikliği başarılı yanıtı. Değişen karakterin PID'sini ve yeni ismini alır, `m_akSimplePlayerInfo`'da ilgili karakterin ismini günceller ve Python tarafındaki `PHASE_WINDOW_SELECT`'in `OnChangeName` fonksiyonunu çağırır.
    *   `HEADER_GC_HWID` (`__RecvHWIDPacket()`): Donanım ID (HWID) ban durumuyla ilgili bilgiyi alır ve `m_bChangedHWID` bayrağını ayarlar.
*   **Diğer Paketler**:
    *   `HEADER_GC_PHASE`: Aşama değişikliği.
    *   `HEADER_GC_HANDSHAKE`, `HEADER_GC_HANDSHAKE_OK`: Zaman senkronizasyonu.
    *   `HEADER_GC_HYBRIDCRYPT_KEYS`, `HEADER_GC_HYBRIDCRYPT_SDB`: Şifreleme anahtarları.
    *   `__IMPROVED_PACKET_ENCRYPTION__` ile `HEADER_GC_KEY_AGREEMENT`, `HEADER_GC_KEY_AGREEMENT_COMPLETED`: Gelişmiş şifreleme anahtar anlaşması.
    *   `HEADER_GC_PLAYER_POINT_CHANGE`: Bu aşamada gelmesi beklenmeyen bir paket, ancak gelirse alınır.
    *   `HEADER_GC_PING`: Ping paketi.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::SelectPhase()`**:
    *   "Select" aşamasındayken sürekli çağrılır ve sunucudan gelen paketleri karakter oluşturma, silme, seçme gibi işlemlere göre ilgili fonksiyonlara yönlendirir.
*   **`CPythonNetworkStream::SetSelectPhase()`**:
    *   Ağ yöneticisini "Select" moduna geçirir.
    *   Python UI'ına imparatorluk seçim veya karakter seçim ekranını göstermesi için sinyal gönderir.
*   **Paket Gönderme Fonksiyonları (`SendSelectEmpirePacket`, `SendSelectCharacterPacket`, vb.)**:
    *   Kullanıcının karakter seçim ekranında yaptığı eylemleri (imparatorluk seçme, karakter seçme, yeni karakter oluşturma, karakter silme) sunucuya iletir.
*   **Paket Alma Fonksiyonları (`__RecvPlayerCreateSuccessPacket`, `__RecvPlayerDestroySuccessPacket`, vb.)**:
    *   Sunucudan gelen yanıtları işler, C++ tarafındaki karakter verilerini günceller ve Python UI'ına sonuçları (başarılı/başarısız, hata mesajı vb.) bildirerek arayüzün güncellenmesini sağlar.

Bu dosya, oyuncunun hesap yönetimiyle ilgili önemli işlemleri (karakter oluşturma, silme, seçme) sunucuyla senkronize bir şekilde gerçekleştirmesini sağlar. Başarılı bir karakter seçimi ve `SendSelectCharacterPacket` gönderimi sonrasında sunucudan gelen `HEADER_GC_PHASE` paketi ile genellikle "Loading" (Yükleme) aşamasına geçilir. 

---

### `PythonNonPlayer.h` (NPC/Mob Veri Tanımları)

Bu başlık dosyası, oyuncu olmayan karakterler (NPC'ler) ve yaratıklar (Mob'lar) ile ilgili temel veri yapılarının, sabitlerin ve `CPythonNonPlayer` sınıfının tanımını içerir.

#### Dosyanın Temel Amaçları ve İçeriği

*   **`CPythonNonPlayer` Sınıfı**:
    *   Singleton olarak tasarlanmıştır (`CSingleton<CPythonNonPlayer>`).
    *   NPC/Mob verilerini yüklemek, saklamak ve bu verilere erişim sağlamak için metodlar içerir.
    *   Temel metodları: `LoadNonPlayerData`, `GetTable`, `GetName`, `GetEventType`, `GetMonsterName`, `GetMobLevel`, `GetMobRank` vb.
*   **`TMobTable` Yapısı (`struct SMobTable`)**:
    *   Bir NPC/Mob'un tüm özelliklerini barındıran ana veri yapısıdır.
    *   İçerdiği bazı önemli alanlar:
        *   `dwVnum`: Karakterin sanal numarası (unique ID).
        *   `szName`, `szLocaleName`: Karakterin ismi ve yerelleştirilmiş ismi.
        *   `bType`: Tipi (Canavar, NPC vb.).
        *   `bRank`: Rütbesi (PAWN, KNIGHT, BOSS, KING vb.).
        *   `bLevel`: Seviyesi.
        *   `dwGoldMin`, `dwGoldMax`: Düşürebileceği minimum/maksimum altın.
        *   `dwExp`: Vereceği tecrübe puanı.
        *   `dwMaxHP`: Maksimum can puanı.
        *   `dwAIFlag`: Yapay zeka bayrakları (Agresif, Korkak vb.).
        *   `dwRaceFlag`: Irk bayrakları (Hayvan, Ölümsüz, Şeytan vb.).
        *   `dwImmuneFlag`: Bağışıklık bayrakları (Sersemletme, Yavaşlatma vb.).
        *   `cEnchants`, `cResists`: Büyü ve direnç değerleri.
        *   `Skills`: Sahip olduğu yetenekler (`TMobSkillLevel` dizisi).
        *   `bOnClickType`: Tıklandığında gerçekleşecek olay tipi.
*   **Enum Sabitleri**:
    *   `EClickEvent`: NPC'ye tıklandığında tetiklenecek olay türlerini tanımlar (Savaş, Dükkan, Konuşma vb.).
    *   `EMobAIFlags` (`WJ_SHOW_MOB_INFO` ile aktifleşir): Mob'ların yapay zeka davranışlarını belirten bayraklar.
    *   `EMobRanks`: Mob rütbeleri.
    *   `EMobImmuneFlags`: Mob bağışıklıkları.
    *   `ERaceFlags`: Mob ırkları.
    *   `EMobEnchants`: Mob'ların sahip olabileceği büyüler (Lanet, Zehir vb.).
    *   `EMobResists`: Mob'ların direnç türleri (Kılıç, Ateş, Büyü vb.).
*   **Diğer Yapılar ve Typedef'ler**:
    *   `SMobSkillLevel`: Bir mob yeteneğinin VNUM'unu ve seviyesini tutar.
    *   `TNonPlayerDataMap`: `std::map<DWORD, TMobTable*>` tipinde, VNUM ile `TMobTable` işaretçilerini eşler.
    *   `ENABLE_SEND_TARGET_INFO` direktifi aktifse:
        *   `SMobItemDrop`: Düşen eşyanın VNUM'unu ve sayısını tutar.
        *   `TMobItemDrop`, `TMobItemDropMap`: Eşya düşürme bilgilerini saklamak için map yapıları.

Bu dosya, istemcinin oyun dünyasındaki NPC ve yaratıklarla ilgili statik verileri nasıl tanımladığını ve organize ettiğini gösterir.

---

### `PythonNonPlayer.cpp` (NPC/Mob Veri Yönetimi)

Bu C++ kaynak dosyası, `CPythonNonPlayer` sınıfının başlık dosyasında (`PythonNonPlayer.h`) tanımlanan metodlarının gerçek implementasyonlarını içerir. Temel olarak, NPC (Non-Player Character) ve mob (yaratık) verilerinin yüklenmesi, saklanması ve bu verilere erişim işlevlerini yönetir.

#### Dosyanın Ana İşlevleri ve Çalışma Prensibi

*   **Veri Yükleme (`LoadNonPlayerData`)**:
    *   Belirtilen bir dosyadan (genellikle `mob_proto` veya benzeri bir paketlenmiş dosya) NPC/Mob prototip verilerini yükler.
    *   Dosya `CEterPackManager` aracılığıyla açılır.
    *   Dosyanın geçerli bir "MMPT" (Mob Proto) formatında olup olmadığı kontrol edilir.
    *   Veriler sıkıştırılmışsa (`COMPRESS_MOB`), `CLZO` kütüphanesi kullanılarak açılır.
    *   Açılan verilerdeki her bir `TMobTable` yapısı okunur, bir kopyası oluşturulur ve `m_NonPlayerDataMap` adlı bir `std::map` içerisinde `dwVnum` (sanal numara) anahtarıyla saklanır.
*   **Veri Erişimi (`GetTable`, `GetName`, `GetEventType` vb.)**:
    *   `GetTable(DWORD dwVnum)`: Verilen VNUM'a sahip `TMobTable` yapısını `m_NonPlayerDataMap` üzerinden bulup döndürür. Bu, diğer birçok get fonksiyonu için temel erişim metodudur.
    *   Diğer `Get...` fonksiyonları (`GetMonsterName`, `GetMobLevel`, `GetMobRank`, `GetMobAIFlag`, `GetMonsterColor`, `GetMonsterHitRange`, `IsImmuneFlagByVnum` vb.), `GetTable`'ı kullanarak ilgili `TMobTable`'ı alır ve içindeki belirli bir alanı (isim, seviye, rütbe, AI bayrağı, renk, vuruş menzili, bağışıklık durumu vb.) döndürür.
    *   `GetEventTypeByVID(DWORD dwVID)`: Verilen bir VID (Virtual ID - oyun dünyasındaki bir instance'ın ID'si) için `CPythonCharacterManager` üzerinden `CInstanceBase` örneğini alır, bu örnekten VNUM'u öğrenir ve ardından `GetEventType(dwVnum)` fonksiyonunu çağırarak tıklama olay tipini döndürür.
*   **`ENABLE_SEND_TARGET_INFO` Kapsamındaki Fonksiyonlar**:
    *   Bu preprocessor direktifi aktif olduğunda, mob'lar hakkında daha detaylı bilgi sağlayan ek fonksiyonlar derlenir. Bunlar arasında:
        *   `GetMonsterMaxHP`, `GetMonsterRaceFlag`, `IsMonsterRaceFlag`, `GetMonsterLevel`, `GetMonsterDamage1`/`GetMonsterDamage2`, `GetMonsterExp`, `GetMonsterDamageMultiply`, `GetMonsterST`/`GetMonsterDX`, `IsMonsterStone`, `GetMonsterRegenCycle`/`GetMonsterRegenPercent`, `GetMonsterGoldMin`/`GetMonsterGoldMax`, `GetMonsterResist`.
        *   Eşya düşürme bilgileriyle ilgili fonksiyonlar: `AddItemDropInfo`, `ClearItemDropInfo`, `GetMonsterDropCount`, `GetMonsterDropItemVnum`, `GetMonsterDropItemCount`. Bu fonksiyonlar `m_MobItemDropMap` adlı bir map'i kullanarak, hangi mob'un hangi eşyaları ne kadar düşürdüğü bilgisini yönetir.
*   **Diğer Fonksiyonlar**:
    *   `GetMonsterScalePercent(DWORD dwVnum)`: Bir mob'un ölçek yüzdesini döndürür (minimum 50, maksimum 200).
    *   `GetMatchableMobList(...)`: Bu fonksiyonun implementasyonu yorum satırı haline getirilmiştir. Muhtemelen belirli bir seviye aralığındaki mob'ları listelemek için tasarlanmıştı.
    *   `Clear()` ve `Destroy()`: `CPythonNonPlayer` singleton'ı yok edildiğinde `m_NonPlayerDataMap` ve `m_MobItemDropMap` (aktifse) içindeki verileri temizler.

Bu dosya, istemcinin oyun içindeki tüm NPC ve yaratıklara ait statik verileri nasıl yönettiğini ve bu verilere nasıl erişim sağladığını detaylandırır. Yükleme işlemi genellikle oyun başlangıcında bir kez yapılır ve bu veriler oyun boyunca kullanılır.

---

### `PythonNonPlayerModule.cpp` (NPC/Mob Python Modülü)

Bu C++ kaynak dosyası, `CPythonNonPlayer` sınıfının C++ tarafındaki işlevlerini Python betiklerinin kullanımına sunmak için bir Python modülü (`nonplayer`) oluşturur. Bu sayede Python arayüz kodları, NPC ve yaratıklarla ilgili bilgilere C++ katmanından erişebilir.

#### Dosyanın Temel Amaçları ve Yapısı

*   **Python Sarmalayıcı (Wrapper) Fonksiyonları**:
    *   `CPythonNonPlayer` sınıfının genel kullanıma açık metodlarının çoğu için Python'dan çağrılabilir C fonksiyonları tanımlanır. Örneğin:
        *   `nonplayerGetEventType(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir VNUM alır, `CPythonNonPlayer::Instance().GetEventType()` çağırır ve sonucu Python integer olarak döndürür.
        *   `nonplayerGetEventTypeByVID(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir VID alır, `CPythonNonPlayer::Instance().GetEventTypeByVID()` çağırır.
        *   `nonplayerGetLevelByVID(PyObject* poSelf, PyObject* poArgs)`: VID ile mob seviyesini döndürür.
        *   `nonplayerGetGradeByVID(PyObject* poSelf, PyObject* poArgs)`: VID ile mob rütbesini (grade/rank) döndürür.
        *   `nonplayerGetMonsterName(PyObject* poSelf, PyObject* poArgs)`: VNUM ile mob ismini döndürür.
        *   `nonplayerLoadNonPlayerData(PyObject* poSelf, PyObject* poArgs)`: Python'dan dosya adı alarak `CPythonNonPlayer::Instance().LoadNonPlayerData()` fonksiyonunu çağırır.
    *   `ENABLE_SEND_TARGET_INFO` direktifi aktif olduğunda, bu kapsama giren `CPythonNonPlayer` metodları için de Python sarmalayıcıları eklenir (örn: `nonplayerGetMonsterMaxHP`, `nonplayerGetMonsterRaceFlag`, `nonplayerGetMonsterDropItem` vb.).
    *   Bu fonksiyonlar genellikle Python'dan argümanları `PyTuple_GetInteger`, `PyTuple_GetString` gibi fonksiyonlarla alır, `CPythonNonPlayer` singleton örneği üzerinden ilgili C++ fonksiyonunu çağırır ve sonucu `Py_BuildValue` ile Python nesnesine dönüştürerek geri verir.
*   **Python Modülü Tanımlama (`initNonPlayer`)**:
    *   `initNonPlayer()` fonksiyonu, Python C API kullanılarak `nonplayer` adında bir modül oluşturur.
    *   `PyMethodDef s_methods[]` dizisi, Python'da hangi isimle hangi C fonksiyonunun çağrılacağını tanımlar (örn: Python'daki `nonplayer.GetEventType` çağrısı C'deki `nonplayerGetEventType` fonksiyonunu çalıştırır).
    *   `Py_InitModule("nonplayer", s_methods)` ile modül başlatılır.
    *   `CPythonNonPlayer.h` içinde tanımlanmış olan çeşitli enum sabitleri (`ON_CLICK_EVENT_...`, `MOB_RANK_...`, `MOB_RESIST_...`, `RACE_FLAG_...` vb.), `PyModule_AddIntConstant` kullanılarak Python modülüne sabit değerler olarak eklenir. Bu sayede Python betikleri bu sabitleri `nonplayer.ON_CLICK_EVENT_SHOP` gibi doğrudan kullanabilir.

Bu dosya, C++ ile yazılmış NPC/Mob veri yönetim sisteminin Python betik arayüzüyle nasıl entegre edildiğini gösterir. Kullanıcı arayüzü veya oyun mantığının Python tarafında geliştirilen kısımları, bu modül aracılığıyla NPC ve yaratıkların özelliklerine ve davranışlarına dair bilgilere kolayca erişebilir. 

---

### `PythonPackModule.cpp` (Paket Sistemi Python Modülü)

Bu C++ kaynak dosyası, istemcinin paketlenmiş dosyalarını (`.epk`, `.eix`) yöneten `CEterPackManager` sınıfının temel işlevlerini Python betiklerinin kullanımına sunan `pack` adlı bir Python modülü tanımlar. Ayrıca, istemci tarafı şifreleme anahtarlarıyla ilgili bazı global değişkenler ve fonksiyonlar içerir, ancak bunlar doğrudan `pack` modülünün bir parçası olarak Python'a aktarılmaz.

#### Dosyanın Temel Amaçları ve Yapısı

*   **Python Modülü (`pack`) Tanımlama (`initpack`)**:
    *   `initpack()`: `pack` adında bir Python modülü oluşturur.
    *   Bu modül, Python betiklerinin istemcinin paketlenmiş dosyalarına erişmesini ve varlıklarını kontrol etmesini sağlar.
*   **Python'a Aktarılan Fonksiyonlar**:
    *   `packExist(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir dosya adı (`strFileName`) alır.
        *   `CEterPackManager::Instance().isExist(strFileName)` fonksiyonunu çağırarak belirtilen dosyanın paketlenmiş dosyalar içinde var olup olmadığını kontrol eder.
        *   Sonucu Python integer (1 varsa, 0 yoksa) olarak döndürür.
        *   Python'da `pack.Exist("dosya_yolu")` şeklinde kullanılır.
    *   `packGet(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir dosya adı (`strFileName`) alır.
        *   Dosya uzantısının `.py`, `.pyc` veya `.txt` olup olmadığını kontrol eder.
        *   Eğer bu uzantılardan birine sahipse, `CEterPackManager::Instance().Get(file, strFileName, &pData)` fonksiyonunu çağırarak dosyanın içeriğini bir `CMappedFile` nesnesi aracılığıyla alır.
        *   Dosya içeriğini ve boyutunu `Py_BuildValue("s#", pData, file.Size())` ile Python string'i olarak döndürür.
        *   Eğer dosya bulunamazsa veya uzantı uygun değilse bir Python istisnası (`Py_BuildException()`) oluşturur.
        *   Python'da `pack.Get("dosya_yolu")` şeklinde kullanılır.
*   **Şifreleme Anahtarları ve Fonksiyonları (`CHINA_CRYPT_KEY` bölümü)**:
    *   `g_adwEncryptKey[4]` ve `g_adwDecryptKey[4]`: Global DWORD dizileri, muhtemelen paket şifrelemesi veya ağ iletişimi şifrelemesi için kullanılır.
    *   `GetKey_20050304Myevan()`: Belirli bir algoritma ile bir anahtar dizisi (`s_adwKey`) üretir. Bu fonksiyonun adı ve içindeki sabit sayılar (1491971513, 2148941891, 3592385981) spesifik bir şifreleme şemasına işaret eder.
    *   `CAccountConnector::__BuildClientKey_20050304Myevan()`: `GetKey_20050304Myevan()` fonksiyonundan alınan anahtarı kullanarak `g_adwEncryptKey` ve `g_adwDecryptKey` global değişkenlerini doldurur. `g_adwEncryptKey`'in bir kısmı rastgele sayılarla güncellenir ve ardından `old_tea_encrypt` (muhtemelen Tiny Encryption Algorithm'ın bir varyantı) kullanılarak `g_adwDecryptKey` oluşturulur.
    *   Bu şifreleme ile ilgili kısımlar `pack` modülünün bir parçası olarak doğrudan Python'a sunulmaz, ancak istemcinin genel işleyişinde, özellikle `CEterPackManager`'ın şifreli paketleri okuyabilmesi veya `CPythonNetworkStream`'in ağ trafiğini şifreleyebilmesi için arka planda kullanılır.

Bu dosya, Python betiklerinin oyunun paketlenmiş varlıklarına (özellikle `.py`, `.pyc`, `.txt` dosyalarına) erişimini kolaylaştırırken, aynı zamanda istemcinin temel şifreleme mekanizmalarından bir parçasını barındırır. `pack.Exist` ve `pack.Get` fonksiyonları, UI scriptleri veya diğer Python tabanlı oyun mantığı tarafından sıkça kullanılır. 

---

### `PythonPlayer.h` (Oyuncu Sınıfı Tanımları)

Bu başlık dosyası, istemcideki ana oyuncu karakterini temsil eden ve yöneten `CPythonPlayer` sınıfının tanımını içerir. `CSingleton<CPythonPlayer>` ve `IAbstractPlayer` arayüzünden miras alır. Oyuncunun tüm verilerini, etkileşimlerini ve oyun dünyasındaki varlığını yönetmek için merkezi bir rol oynar.

#### Dosyanın Temel Amaçları ve İçeriği

*   **`CPythonPlayer` Sınıf Tanımı**:
    *   Oyuncuyla ilgili tüm verileri ve işlevleri barındıran ana sınıftır.
    *   Çok sayıda üye değişkeni (oyuncu statüleri, envanter, yetenekler, hedef bilgileri, hareket bayrakları vb.) ve metodu içerir.
*   **Enum Tanımları**:
    *   `CATEGORY_...`: Yetenek kategorileri (Aktif, Pasif).
    *   `STATUS_INDEX_...`: Temel statülerin indeksleri (ST, DX, IQ, HT).
    *   `MBT_...`: Fare buton türleri (Sol, Sağ, Orta).
    *   `MBF_...`: Fare butonlarına atanabilecek fonksiyonlar (Akıllı, Hareket, Kamera, Saldırı, Yetenek, Otomatik).
    *   `MBS_...`: Fare buton durumları (Tıklama, Basılı Tutma).
    *   `EMode`: Oyuncunun etkileşim modları (Pozisyona Tıklama, Eşyaya Tıklama, Aktöre Tıklama, Yetenek Kullanma vb.).
    *   `EEffect`: Oyuncuyla ilişkili efektler (örn: `EFFECT_PICK`).
    *   `EMetinSocketType`: Metin taşı soket türleri.
    *   `EKeyBoard_UD`, `EKeyBoard_LR`: Klavye yön tuşları durumları.
    *   `EPartyRole`: Parti içindeki roller.
    *   `EAutoPotionType`: Otomatik iksir türleri (HP, SP).
*   **Struct Tanımları**:
    *   `SSkillInstance` (`TSkillInstance`): Bir yeteneğin indeksi, türü, seviyesi, verimlilik yüzdeleri, bekleme süresi durumu ve aktiflik durumu gibi bilgilerini tutar.
    *   `SPlayerStatus` (`TPlayerStatus`): Oyuncunun envanterini (`aItem`), ejderha taşı envanterini (`aDSItem`), hızlı erişim slotlarını (`aQuickSlot`), yeteneklerini (`aSkill`), stat puanlarını (`m_alPoint`) ve mevcut hızlı erişim sayfasını (`lQuickPageIndex`) içeren kapsamlı bir yapıdır. `POINT_MAGIC_NUMBER` ile XOR'lanarak saklanan puanlar için `SetPoint` ve `GetPoint` metodları içerir.
    *   `SPartyMemberInfo` (`TPartyMemberInfo`): Parti üyesinin PID, VID, ismi, durumu, HP yüzdesi ve üzerindeki etkileri (buff/debuff) tutar.
    *   `SAutoPotionInfo`: Otomatik iksir sistemi için bilgileri (aktif mi, toplam/mevcut miktar, envanter slotu) tutar.
*   **Preprocessor Direktiflerine Bağlı Bölümler**:
    *   Çok sayıda `ENABLE_...` direktifi (örn: `ENABLE_PREMIUM_PRIVATE_SHOP`, `ENABLE_NEW_EQUIPMENT_SYSTEM`, `ENABLE_SPECIAL_INVENTORY_SYSTEM`, `ENABLE_MOVE_COSTUME_ATTR`, `ENABLE_ACCE_COSTUME_SYSTEM`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_KEYCHANGE_SYSTEM`, `ENABLE_FISHING_RENEWAL`, `ENABLE_GEM_SYSTEM`, `ENABLE_AURA_COSTUME_SYSTEM`, `ENABLE_GROWTH_PET_SYSTEM`, `ENABLE_CUBE_RENEWAL`) ile çeşitli oyun sistemlerine ait veri yapıları (örn: `TMoveCostumeAttrs`), enumlar ve fonksiyon prototipleri eklenir. Bu, oyunun modüler bir şekilde farklı özelliklerle derlenebilmesini sağlar.
*   **Fonksiyon Prototipleri**:
    *   **Hareket ve Etkileşim**: `NEW_MoveTo...`, `NEW_Attack`, `NEW_Fishing`, `SendClickItemPacket`, `__OnClickActor`, `__OnClickItem`, `__OnClickGround`.
    *   **Veri Yönetimi**: `SetName`, `SetRace`, `SetStatus`, `GetStatus`, `SetItemData`, `GetItemData`, `SetSkill`, `GetSkillLevel`, `SetAffect`, `RemoveAffect`.
    *   **Sistem Bildirimleri**: `NotifyDeletingCharacterInstance`, `NotifyCharacterDead`, `NotifyChangePKMode`.
    *   **Parti Yönetimi**: `AppendPartyMember`, `RemovePartyMember`, `UpdatePartyMemberInfo`.
    *   **PVP ve Özel Pazar**: `RememberChallengeInstance`, `OpenPrivateShop`.
    *   **Diğer Sistemler**: Çeşitli `ENABLE_...` direktifleri altındaki sistemlere özel `Set...WindowOpen`, `Get...ItemData` gibi fonksiyonlar.
    *   **Yardımcı ve Dahili Fonksiyonlar**: `__GetPickedActorPtr`, `__ClearReservedAction`, `__UpdateBattleStatus`, `__CanUseSkill`.
*   **Üye Değişkenleri**:
    *   `m_ppyGameWindow`: Python tarafındaki ana oyun penceresine işaretçi.
    *   `m_skillSlotDict`: Yetenek ID'sini slot indeksine eşleyen map.
    *   `m_stName`, `m_dwMainCharacterIndex`, `m_dwRace`: Oyuncunun temel kimlik bilgileri.
    *   `m_playerStatus`: `TPlayerStatus` yapısı, oyuncunun tüm dinamik verilerini tutar.
    *   Çok sayıda bayrak (flag) ve durum değişkeni (`m_isUp`, `m_isMoving`, `m_isOpenPrivateShop`, `m_isObserverMode` vb.).
    *   Fare ve klavye durumlarını, hedef bilgilerini, ayrılmış eylemleri (`m_eReservedMode`) saklayan değişkenler.
    *   `m_PartyMemberMap`, `m_ChallengeInstanceSet`, `m_RevengeInstanceSet`, `m_CantFightInstanceSet`: Parti ve PvP ile ilgili verileri tutan konteynerler.
    *   `m_vecAffectData`: Oyuncu üzerindeki aktif etkileri (buff/debuff) saklayan vektör.
    *   Preprocessor direktiflerine bağlı sistemler için ek üye değişkenleri (örn: `m_AuraItemInstanceVector`, `m_GemItemsMap`, `m_GrowthPetInfo`).

Bu başlık dosyası, oyuncu karakterinin istemci tarafındaki tüm yönlerini kapsayan çok geniş ve merkezi bir sınıfın arayüzünü tanımlar. Oyunun çekirdek mantığının büyük bir kısmı bu sınıf ve onunla etkileşen diğer yöneticiler (CharacterManager, ItemManager, NetworkStream vb.) üzerinden yürütülür.

---

### `PythonPlayer.cpp` (Oyuncu Sınıfı Uygulaması)

Bu C++ kaynak dosyası, `PythonPlayer.h` başlığında tanımlanan `CPythonPlayer` sınıfının metodlarının tam implementasyonlarını içerir. İstemcideki ana oyuncu karakterinin davranışlarını, veri yönetimini ve oyun dünyasıyla etkileşimlerini detaylı bir şekilde yönetir.

#### Dosyanın Ana İşlevleri ve Çalışma Prensibi

*   **Yapıcı (`CPythonPlayer::CPythonPlayer`) ve Yıkıcı (`CPythonPlayer::~CPythonPlayer`)**:
    *   Yapıcı, temel ayarları yapar, fare butonlarının varsayılan fonksiyonlarını atar (`MBF_SMART`, `MBF_CAMERA`), efekt dizisini sıfırlar ve bazı önemli durum değişkenlerini başlangıç değerlerine ayarlar. Ayrıca, bazı temel "Affect" (etki) indekslerini karşılık gelen yetenek indeksleriyle eşleyen `m_kMap_dwAffectIndexToSkillIndex` map'ini doldurur. `Clear()` fonksiyonunu çağırarak tüm oyuncu verilerini sıfırlar.
    *   Yıkıcı (`default`) özel bir işlem yapmaz, üye değişkenlerinin yıkıcıları otomatik çağrılır.
*   **Güncelleme (`Update`)**:
    *   Her frame'de çağrılır.
    *   Fare ile yürüme yönünü günceller (`NEW_RefreshMouseWalkingDirection`).
    *   Otomatik saldırıyı günceller (`__Update_AutoAttack`).
    *   Lonca bölgesi olaylarını kontrol eder (`__Update_NotifyGuildAreaEvent`).
    *   Eğer bir hedef pozisyona gidiliyorsa (`m_isDestPosition`), hedefe varılıp varılmadığını kontrol eder veya belirli bir süre geçmesine rağmen varılamadıysa uyarı verir (`AlarmHaveToGo`).
    *   Eğer dayanıklılık tüketiliyorsa (`m_isConsumingStamina`), mevcut dayanıklılığı azaltır ve Python UI'ını günceller.
*   **Oyuncu Veri Yönetimi**:
    *   **Statüler**: `SetStatus` (belirli bir stat puanını ayarlar ve gerekirse savaş durumunu günceller), `GetStatus` (bir stat puanını döndürür). Statü puanları `POINT_MAGIC_NUMBER` sabiti ile XOR'lanarak saklanır ve okunur (`SPlayerStatus::SetPoint`, `SPlayerStatus::GetPoint`). `__UpdateBattleStatus` fonksiyonu, saldırı gücü, isabet oranı, kaçınma oranı gibi ikincil statüleri hesaplar.
    *   **Temel Bilgiler**: `SetName`, `GetName`, `SetRace`, `GetRace`, `SetMainCharacterIndex`, `GetMainCharacterIndex`.
    *   **Silah Gücü**: `SetWeaponPower` (silahın minimum/maksimum fiziksel ve büyüsel saldırı gücünü ve ek gücünü ayarlar, ardından savaş durumunu günceller).
*   **Eşya Yönetimi (`GetItemData`, `SetItemData`, `GetItemIndex`, `GetItemCount` vb.)**:
    *   Oyuncunun envanterindeki (`m_playerStatus.aItem`), ejderha taşı envanterindeki (`m_playerStatus.aDSItem`) ve bazı sistemlere özel (`ENABLE_ATTR_6TH_7TH`) eşyaların bilgilerini yönetir.
    *   Eşyaların VNUM'unu, sayısını, bayraklarını, anti-bayraklarını, türünü, alt türünü, soketlerini, efsunlarını ve diğer özelliklerini almak ve ayarlamak için çeşitli fonksiyonlar sunar.
    *   `MoveItemData`: İki slot arasındaki eşyaları takas eder.
    *   `SendClickItemPacket`: Yerdeki bir eşyaya tıklandığında toplama isteği gönderir.
    *   Preprocessor direktiflerine (`ENABLE_...`) bağlı olarak birçok ek eşya özelliği yönetimi eklenir (örn: `SetItemUnbindTime`, `SetItemRefineElement`, `SetItemTransmutationVnum`, `SetItemSetValue`).
*   **Hızlı Erişim Slotları (`GetQuickPage`, `SetQuickPage`, `AddQuickSlot`, `DeleteQuickSlot`, `MoveQuickSlot`, `Request...QuickSlot` vb.)**:
    *   Hızlı erişim çubuğundaki slotların sayfasını değiştirir, slotlara eşya/yetenek/duygu ekler, siler, yerlerini değiştirir ve kullanılmalarını sağlar.
    *   Lokal ve global slot indeksleri arasında dönüşüm yapar (`LocalQuickSlotIndexToGlobalQuickSlotIndex`).
    *   Ağ üzerinden sunucuya hızlı slot değişikliklerini bildirir.
*   **Yetenek Yönetimi (`SetSkill`, `GetSkillIndex`, `GetSkillLevel`, `ClickSkillSlot`, `SetSkillCoolTime` vb.)**:
    *   Oyuncunun yeteneklerini (`m_playerStatus.aSkill`) ve yetenek ID'leriyle slot indeksleri arasındaki eşleşmeyi (`m_skillSlotDict`) yönetir.
    *   Yeteneklerin seviyesini, derecesini, bekleme sürelerini, aktiflik durumlarını ayarlar ve sorgular.
    *   Bir yetenek kullanıldığında (`ClickSkillSlot`) gerekli kontrolleri yapar (mana, bekleme süresi, ok vb.) ve sunucuya kullanım isteği gönderir (`__SendUseSkill`).
    *   `AffectIndexToSkillIndex` ve `AffectIndexToSkillSlotIndex`: Belirli buff/debuff (affect) ID'lerini ilgili yetenek ID'sine veya slotuna dönüştürür.
*   **Hareket ve Eylem Yönetimi (`NEW_MoveTo...`, `NEW_Attack`, `NEW_Stop`, `__OnPressSmart`, `__OnClickSmart` vb.)**:
    *   Oyuncunun klavye ve fare girdilerine göre hareket etmesini, saldırmasını, durmasını sağlar.
    *   "Akıllı" tıklama (`MBF_SMART`) ile tıklanan hedefe göre uygun eylemi (saldırı, toplama, konuşma) otomatik olarak seçer.
    *   Eylemleri bir gecikmeyle veya belirli bir koşul sağlandığında gerçekleştirmek üzere rezerve eder (`__ReserveClickItem`, `__ReserveClickActor`, `__ReserveUseSkill`).
*   **Hedef Yönetimi (`SetTarget`, `GetTargetVID`, `__GetTargetActorPtr`, `__ClearTarget` vb.)**:
    *   Oyuncunun mevcut hedefini (VID ile) ayarlar ve sorgular.
    *   Hedef değiştiğinde veya güncellendiğinde Python UI'ını bilgilendirir.
    *   Saldırı veya yetenek için uygun hedefi bulmaya çalışır (`__SearchNearTarget`).
*   **Parti Yönetimi (`AppendPartyMember`, `RemovePartyMember`, `UpdatePartyMemberInfo`, `IsPartyMemberByVID` vb.)**:
    *   Parti üyelerinin bilgilerini (`m_PartyMemberMap` içinde `TPartyMemberInfo` olarak) saklar ve günceller.
    *   Parti üyelerinin oyundaki varlıklarıyla (VID) bağlantısını kurar/keser.
*   **PVP Yönetimi (`RememberChallengeInstance`, `RememberRevengeInstance`, `IsChallengeInstance` vb.)**:
    *   Düello (`Challenge`), intikam (`Revenge`) ve savaşılamayan (`CantFight`) oyuncuların VID'lerini setlerde tutar.
*   **Diğer Sistemlerle Entegrasyon**:
    *   **Özel Pazar**: `OpenPrivateShop`, `ClosePrivateShop`, `IsOpenPrivateShop`.
    *   **Gözlemci Modu**: `SetObserverMode`, `IsObserverMode`.
    *   **Dayanıklılık (Stamina)**: `StartStaminaConsume`, `StopStaminaConsume`.
    *   **Efektler**: `RegisterEffect`, `NEW_ShowEffect`.
    *   **Duygular (Emotions)**: `ActEmotion`, `StartEmotionProcess`, `EndEmotionProcess`.
    *   **Zindan Navigasyonu**: `SetDungeonDestinationPosition`, `AlarmHaveToGo` (pusula efekti gösterir).
    *   **Etkiler (Affects/Buffs/Debuffs)**: `AddAffect`, `RemoveAffect`, `GetAffectDataIndex`, `GetAffectDuration`. `m_vecAffectData` içinde saklanır.
    *   **Python UI Etkileşimi**: `SetGameWindow` ile Python'daki ana oyun penceresine (`m_ppyGameWindow`) referans alır ve `PyCallClassMemberFunc` ile Python fonksiyonlarını çağırarak UI'ı günceller (örn: "RefreshInventory", "OnChangePKMode").
    *   Çok sayıda `ENABLE_...` direktifi ile derlenen sistemler için ilgili fonksiyonlar (Acce, Aura, Gem, Pet, Change Look vb.). Örneğin, `ENABLE_AURA_COSTUME_SYSTEM` aktifse, aura eşyalarını, seviyelerini, arıtma penceresini yöneten fonksiyonlar bulunur.
*   **Veri Temizleme (`Clear`, `ClearSkillDict`, `NEW_ClearSkillData`)**:
    *   Oyundan çıkışta veya yeni bir karaktere girerken oyuncuyla ilgili tüm verileri (statüler, envanter, yetenekler, hedefler, parti, açık pencere durumları vb.) sıfırlar.
*   **Yardımcı Fonksiyonlar**:
    *   Saldırı gücü, isabet oranı, kaçınma oranı gibi savaş değerlerini hesaplayan `__GetLevelAtk`, `__GetStatAtk`, `__GetTotalAtk` vb. fonksiyonlar.
    *   Yetenek kullanımı için kontroller yapan `__CanShot`, `__CanUseSkill`, `__CheckSkillUsable` vb. fonksiyonlar.
    *   Belirli koşulları kontrol eden (`IsDead`, `IsPoly`) ve çeşitli içsel durumları yöneten birçok private/protected (`__...`) fonksiyon.

`CPythonPlayer`, istemcinin oyuncu merkezli mantığının büyük bir kısmını üstlenir. Oyuncu girdilerini alır, bu girdilere göre eylemler başlatır, sunucudan gelen güncellemelerle oyuncu durumunu senkronize eder ve bu bilgileri oyun dünyasında ve kullanıcı arayüzünde yansıtır. Kapsamlı `ENABLE_...` direktifleri sayesinde, farklı özellik setleriyle derlenebilen esnek bir yapıya sahiptir.

---

### `PythonPlayerEventHandler.h` (Oyuncu Olay İşleyicisi Tanımları)

Bu başlık dosyası, `CPythonPlayerEventHandler` sınıfının tanımını içerir. Bu sınıf, `CActorInstance::IEventHandler` arayüzünden türemiştir ve ana oyuncu karakterinin (`CPythonPlayer` tarafından yönetilen) çeşitli oyun içi olaylarını (hareket, saldırı, yetenek kullanımı, efekt değişiklikleri vb.) işlemek ve bu olaylara karşılık gelen ağ mesajlarını sunucuya göndermekle sorumludur.

#### Dosyanın Temel Amaçları ve İçeriği

*   **`CPythonPlayerEventHandler` Sınıf Tanımı**:
    *   Singleton olarak tasarlanmıştır (`GetSingleton` static metodu ile erişilir).
    *   `CActorInstance::IEventHandler` arayüzünde tanımlanan sanal (virtual) olay işleyici metodları override eder:
        *   `OnSyncing`, `OnWaiting`, `OnMoving`, `OnMove`, `OnStop`, `OnWarp`: Karakterin farklı hareket ve pozisyon durumlarında tetiklenir.
        *   `OnClearAffects`, `OnSetAffect`, `OnResetAffect`: Karakter üzerindeki buff/debuff (etki) değişikliklerinde tetiklenir.
        *   `OnAttack`: Karakter bir saldırı animasyonu başlattığında tetiklenir.
        *   `OnUseSkill`: Karakter bir yetenek kullandığında tetiklenir.
        *   `OnUpdate`: Her frame güncellemesinde çağrılabilir (şu anki implementasyonda boş).
        *   `OnChangeShape`: Karakterin görünümü/şekli değiştiğinde (örn: dönüşüm) tetiklenir.
        *   `OnHit`: Karakter bir hedefe başarılı bir vuruş yaptığında tetiklenir.
    *   `FlushVictimList()`: Biriktirilmiş "vurulan hedef" (victim) bilgilerini sunucuya göndermek için public bir metottur.
*   **`SVictim` Yapısı (`struct`)**:
    *   Vurulan bir hedefin Sanal ID'sini (`m_dwVID`) ve son bilinen piksel koordinatlarını (`m_lPixelX`, `m_lPixelY`) saklamak için kullanılır.
*   **Üye Değişkenleri**:
    *   `m_kVctkVictim`: `SVictim` yapılarından oluşan bir `std::vector`. `OnHit` sırasında vurulan hedeflerin bilgilerini geçici olarak depolar.
    *   `m_dwPrevComboIndex`: Önceki kombo saldırısının indeksini tutar.
    *   `m_dwNextWaitingNotifyTime`, `m_dwNextMovingNotifyTime`: Sunucuya hareket durumu paketlerini gönderme sıklığını kontrol etmek için zaman damgaları.
    *   `m_kPPosPrevWaiting`: `OnWaiting` durumunda önceki pozisyonu saklar, gereksiz paket gönderimini önlemek için kullanılır.
*   **`CNormalBowAttack_FlyEventHandler_AutoClear` İç İçe Sınıfı (`private class`)**:
    *   `IFlyEventHandler` arayüzünden türemiştir.
    *   Normal yay saldırılarında (ok gibi) uçan nesnelerin olaylarını yönetir.
    *   Metodları:
        *   `Set`, `SetTarget`: Olay işleyicisini ana karakter ve hedef ile ilişkilendirir.
        *   `OnSetFlyTarget`: Uçan nesnenin hedefi belirlendiğinde çağrılır.
        *   `OnShoot`: Uçan nesne fırlatıldığında çağrılır.
        *   `OnNoTarget`, `OnExplodingOutOfRange`, `OnExplodingAtBackground`, `OnExplodingAtAnotherTarget`, `OnExplodingAtTarget`: Uçan nesnenin çeşitli sonuçlanma durumlarında (hedef yok, menzil dışı, zemine çarpma, farklı hedefe çarpma, ana hedefe çarpma) çağrılır.
    *   `m_NormalBowAttack_FlyEventHandler_AutoClear`: Bu iç içe sınıftan bir üye örneği.
*   **Public Metodlar (`GetNormalBowAttackFlyEventHandler`, `ChangeFlyTarget`)**:
    *   Yay saldırıları için özelleşmiş uçan nesne olay işleyicisini almak ve hedefini değiştirmek için kullanılır.

Bu dosya, oyuncu karakterinin aksiyonlarına ve durum değişikliklerine tepki veren, bu tepkileri sunucuyla senkronize eden ve bazı özel saldırı mekaniklerini (örn: ok saldırısı) yöneten bir olay mimarisi tanımlar.

---

### `PythonPlayerEventHandler.cpp` (Oyuncu Olay İşleyicisi Uygulaması)

Bu C++ kaynak dosyası, `PythonPlayerEventHandler.h` başlığında tanımlanan `CPythonPlayerEventHandler` sınıfının ve onun iç içe sınıfı olan `CNormalBowAttack_FlyEventHandler_AutoClear`'ın metodlarının implementasyonlarını içerir.

#### Ana Sınıf (`CPythonPlayerEventHandler`) Fonksiyonları

*   **Singleton Erişimi (`GetSingleton`)**: Standart singleton deseni ile global bir örnek döndürür.
*   **Efekt Olayları (`OnClearAffects`, `OnSetAffect`, `OnResetAffect`)**:
    *   Bu fonksiyonlar doğrudan `CPythonPlayer::Instance()` üzerinden ilgili efekt (buff/debuff) fonksiyonlarını çağırarak oyuncu üzerindeki etkileri yönetir.
*   **Hareket/Durum Olayları (`OnSyncing`, `OnWaiting`, `OnMoving`, `OnMove`, `OnStop`, `OnWarp`)**:
    *   `OnSyncing`: Oyuncunun pozisyonu sunucuyla senkronize edilirken çağrılır ve `m_kPPosPrevWaiting` güncellenir.
    *   `OnWaiting`, `OnMoving`: Bu durumlar devam ederken, belirli zaman aralıklarında (`m_dwNextWaitingNotifyTime`, `m_dwNextMovingNotifyTime` ile kontrol edilir) ve eğer karakter anlamlı bir mesafe katetmişse (`OnWaiting` için) `CPythonNetworkStream::Instance().SendCharacterStatePacket()` aracılığıyla sunucuya karakterin mevcut pozisyonunu, yönünü ve durumunu (FUNC_WAIT, FUNC_MOVE) gönderir.
    *   `OnMove`: Karakter yeni bir hareket başlattığında çağrılır ve hemen bir FUNC_MOVE paketi gönderir. Ayrıca sonraki periyodik bildirim zamanlarını sıfırlar.
    *   `OnStop`, `OnWarp`: Karakter durduğunda veya ışınlandığında çağrılır ve sunucuya FUNC_WAIT durum paketi gönderir.
*   **Saldırı Olayları (`OnAttack`, `OnUseSkill`)**:
    *   `OnAttack`: Oyuncu bir kombo saldırı hareketi (`wMotionIndex`) başlattığında, sunucuya pozisyon, yön ve FUNC_COMBO durumunu içeren bir paket gönderir. `__ATTACK_SPEED_CHECK__` tanımlıysa, saldırı hızıyla ilgili loglama yapar.
    *   `OnUseSkill`: Oyuncu bir yetenek (`uMotSkill`, `uArg` ile yetenek ID'si ve argümanı) kullandığında, sunucuya pozisyon, yön ve FUNC_SKILL durumunu içeren bir paket gönderir.
*   **Şekil Değiştirme (`OnChangeShape`)**:
    *   Karakter şekil değiştirdiğinde (dönüşüm vb.), `CPythonPlayer::Instance().NEW_Stop()` çağrılarak karakterin mevcut hareketi durdurulur.
*   **Vuruş Olayı (`OnHit`)**:
    *   Oyuncu bir hedefe (`rkActorVictim`) bir yetenekle (`uSkill`) vurduğunda çağrılır.
    *   `CPythonPlayer::Instance().SetTarget()` ile oyuncunun hedefini günceller.
    *   `isSendPacket` parametresi `TRUE` ise, `CPythonNetworkStream::Instance().SendAttackPacket()` ile sunucuya saldırı bilgisini gönderir. `ATTACK_TIME_LOG` tanımlıysa, saldırı zamanlamasıyla ilgili loglama yapar.
    *   Eğer hedef itilebiliyorsa (`IsPushing()`) ve çok büyük bir yaratık değilse (`IS_HUGE_RACE` kontrolü), `CPythonCharacterManager::Instance().AdjustCollisionWithOtherObjects()` ile çarpışma kontrolü yapılır ve hedefin son pozisyonu `SVictim` yapısı olarak `m_kVctkVictim` vektörüne eklenir.
*   **Vurulan Hedef Listesini Boşaltma (`FlushVictimList`)**:
    *   Eğer `m_kVctkVictim` listesi boş değilse, bu listedeki tüm hedeflerin VID ve pozisyon bilgilerini içeren `HEADER_CG_SYNC_POSITION` paketini oluşturur ve sunucuya gönderir. Bu, genellikle alan etkili yetenekler sonrası birden fazla hedefin pozisyonunu senkronize etmek için kullanılır. Liste daha sonra temizlenir.
*   **Yapıcı (`CPythonPlayerEventHandler`)**: Üye değişkenlerini başlangıç değerlerine ayarlar.

#### İç İçe Sınıf (`CNormalBowAttack_FlyEventHandler_AutoClear`) Fonksiyonları

Bu sınıf, ok gibi uçan nesnelerin olaylarını yönetir:

*   **Kurulum (`Set`, `SetTarget`)**: Ana `CPythonPlayerEventHandler` örneğini, saldırıyı yapan ana karakteri (`m_pInstMain`) ve hedefi (`m_pInstTarget`) ayarlar.
*   **Uçan Nesne Hedef Belirleme (`OnSetFlyTarget`)**:
    *   Okun hedefi belirlendiğinde çağrılır. Ana karakterin pozisyonunu ve yönünü alır, hedefin ID'sini ve uçan nesnenin varış noktasını içeren `HEADER_CG_FLY_TARGETING` paketini `CPythonNetworkStream` aracılığıyla sunucuya gönderir.
*   **Uçan Nesne Fırlatma (`OnShoot`)**:
    *   Ok fırlatıldığında çağrılır ve `CPythonNetworkStream::Instance().SendShootPacket()` ile sunucuya ilgili yetenek ID'sini içeren bir paket gönderir.
*   **Diğer Olaylar (`OnNoTarget`, `OnExplodingOutOfRange`, `OnExplodingAtBackground`, `OnExplodingAtAnotherTarget`, `OnExplodingAtTarget`)**:
    *   Bu fonksiyonlar uçan nesnenin çeşitli sonuçlarını (hedef yok, menzil dışı, zemine çarpma vb.) işler. `OnExplodingAtAnotherTarget` fonksiyonu, eğer ok hedeflenen kişi yerine başka birine isabet ederse, o yeni hedefe saldırı paketi gönderme (şu an yorum satırında) ve grafiksel hasar efekti gösterme mantığını içerir. `OnExplodingAtTarget` ise ana hedefe isabet durumunda (şu an yorum satırında) saldırı paketi göndermeyi içerir.

Özetle, `CPythonPlayerEventHandler`, oyuncu karakterinin dünyayla etkileşimlerinden doğan olayları yakalayıp bu olayların sonuçlarını (çoğunlukla pozisyon ve eylem güncellemeleri şeklinde) sunucuyla senkronize eden kritik bir bileşendir. Bu sayede diğer oyuncular ve sunucu, karakterin ne yaptığından haberdar olur.

</rewritten_file>