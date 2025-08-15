# UserInterface Referans Kılavuzu - Bölüm 10

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonLoadingUtils.hpp` (İstemci Yükleme Yardımcı Tanımları)](#pythonloadingutilshpp-istemci-yükleme-yardımcı-tanımları)
*   [`PythonLoading.hpp` (İstemci Yükleme Veri ve Sabitleri)](#pythonloadinghpp-istemci-yükleme-veri-ve-sabitleri)
*   [`PythonLoading.h` (İstemci Yükleme Yöneticisi Sınıfı)](#pythonloadingh-istemci-yükleme-yöneticisi-sınıfı)
*   [`PythonLoading.cpp` (İstemci Yükleme Yöneticisi Implementasyonu ve Python `loading` Modülü)](#pythonloadingcpp-istemci-yükleme-yöneticisi-implementasyonu-ve-python-loading-modülü)
*   [`PythonMailBox.h` (Python Posta Kutusu Sistemi Başlık Dosyası)](#pythonmailboxh-python-posta-kutusu-sistemi-başlık-dosyası)
*   [`PythonMailBox.cpp` (Python Posta Kutusu Sistemi Implementasyonu ve `mail` Modülü)](#pythonmailboxcpp-python-posta-kutusu-sistemi-implementasyonu-ve-mail-modülü)
*   [`PythonMessenger.h` (Python Messenger Sistemi Başlık Dosyası)](#pythonmessengerh-python-messenger-sistemi-başlık-dosyası)
*   [`PythonMessenger.cpp` (Python Messenger Sistemi Implementasyonu ve `messenger` Modülü)](#pythonmessengercpp-python-messenger-sistemi-implementasyonu-ve-messenger-modülü)
*   [`PythonMiniMap.h` (Python Mini Harita Sistemi Başlık Dosyası)](#pythonminimaph-python-mini-harita-sistemi-başlık-dosyası)
*   [`PythonMiniMap.cpp` (Python Mini Harita Sistemi Implementasyonu ve `miniMap` Modülü)](#pythonminimapcpp-python-mini-harita-sistemi-implementasyonu-ve-minimap-modülü)
*   [`PythonMiniMapModule.cpp` (Python Mini Harita Modülü)](#pythonminimapmodulecpp-python-mini-harita-modülü)

---

### `PythonLoadingUtils.hpp` (İstemci Yükleme Yardımcı Tanımları)

Bu başlık dosyası, `ENABLE_LOADING_PERFORMANCE` makrosu etkinleştirildiğinde, istemcinin yükleme sürecini optimize etmek için kullanılan çeşitli veri yapılarını ve yardımcı şablon fonksiyonunu tanımlar. Özellikle `CPythonLoading` sınıfı ve ilgili veri tanımlamaları (`PythonLoading.hpp`) tarafından kullanılır.

#### Dosyanın Genel Amaçları

*   **Veri Yapısı Tanımlamaları**: Oyunun yükleme aşamasında kullanılan temel veri türleri için `typedef` ve `struct` tanımları içerir. Bu yapılar, sesler, efektler, karakter ırkları, hareketler (animasyonlar), yetenekler, harita isimleri, unvanlar, renkler, duygu ikonları ve lonca binaları gibi farklı oyun varlıklarını gruplandırmak ve yönetmek için kullanılır.
    *   `SSoundData`: Ses dosyası adı ve sesin çalınacağı eşya türünü içerir. `SoundDataVector` olarak `std::vector` içinde saklanır.
    *   `SEffectData`: Efekt türü, efektin bağlanacağı kemik adı, efekt dosyası yolu ve önbelleğe alınıp alınmayacağını belirtir. `EffectDataVector` olarak `std::vector` içinde saklanır.
    *   `SFlyEffectData`: Uçan efektler için indeks, tip ve dosya adını içerir. `FlyEffectDataVector` olarak `std::vector` içinde saklanır.
    *   `SEmoticonEffectData`: `SEffectData`'dan miras alır ve ek olarak duygu (emoticon) komutunu içerir. `EmotionEffectDataVector` olarak `std::vector` içinde saklanır.
    *   `SRaceData`: Irk indeksi, model dosyası adı ve ırk verilerinin bulunduğu dosya yolunu içerir. `RaceDataVector` olarak `std::vector` içinde saklanır.
    *   `SMotionData`: Hareket (animasyon) indeksi, animasyon dosyası adı ve rastgele seçilme yüzdesini içerir. `MotionDataVector` olarak `std::vector` içinde saklanır. `SkillGradeMotionData` ise yetenek seviyesine bağlı hareketler için `std::pair<BYTE, std::string>` olarak tanımlanmıştır.
    *   `SkillIndexVector`: Yetenek indekslerini tutan bir `std::vector<DWORD>`'dur. `SkillIndexVectorMap` (karakter sınıfı ve yetenek grubuna göre) ve `GuildSkillIndexVectorMap` (lonca yetenekleri için) gibi `std::map` yapıları içinde kullanılır.
    *   `DungeonMapNameVector`: Zindan harita isimlerini tutan bir `std::vector<std::string>`'dir.
    *   `TitleNameVector`: Unvan isimlerini tutan bir `std::vector<std::string>`'dir. `ENABLE_GENDER_ALIGNMENT` aktifse, cinsiyete özel unvanlar için `PairedTitleName` yapısı kullanılır.
    *   `NameColorMap`, `TitleColorMap`: İsim ve unvan renklerini `BYTE` (indeks) ve `D3DCOLORVALUE` (renk değeri) eşleşmesi olarak tutan `std::map` yapılarıdır.
    *   `EmotionIconVector`: Duygu ikonu indeksi ve ikon dosya yolunu eşleştiren bir `std::map`'tir.
    *   `GuildBuildingMap`: Lonca bina türü (string) ile model dosya adını eşleştiren bir `std::map<std::string_view, std::string>`'dir.
*   **Yardımcı Fonksiyon Şablonu**:
    *   `SplitString`: Verilen bir karakter dizisini (string_view) belirtilen bir ayırıcı karaktere göre böler ve her bir bölüm için kullanıcı tanımlı bir fonksiyonu çağırır. Bu, genellikle metin tabanlı yapılandırma dosyalarını okurken kullanılır.
*   **Python Nesnesi Yardımcı Fonksiyonu**:
    *   `GetLocaleInfoString`: Python'daki `localeInfo` modülünden verilen bir anahtara karşılık gelen yerelleştirilmiş metni almak için bir fonksiyon prototipi sunar.

#### C++ ve Python Arayüzü ile İlişkisi

Bu dosya doğrudan Python'a bir modül sunmaz. Tanımladığı veri yapıları, `CPythonLoading` sınıfının C++ implementasyonunda ve `PythonLoading.hpp` dosyasındaki statik veri tanımlamalarında kullanılır. `SplitString` gibi yardımcı fonksiyonlar, C++ tarafında veri işleme için kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Optimize Edilmiş Veri Yükleme**: `CPythonLoading` sistemi, oyun başladığında veya harita değiştirildiğinde ihtiyaç duyulan çeşitli oyun varlıklarını (sesler, efektler, animasyonlar vb.) önceden tanımlanmış bu yapılar aracılığıyla verimli bir şekilde yüklemek için kullanılır.
*   **Yapılandırılmış Veri Yönetimi**: Farklı türdeki oyun verilerini tutarlı ve organize bir şekilde yönetmek için standartlaştırılmış yapılar sunar.
*   **Kod Okunabilirliği ve Bakımı**: Yükleme mantığında kullanılan karmaşık veri türlerini basitleştirerek kodun daha okunabilir ve bakımı daha kolay olmasına yardımcı olur.

Bu dosya, istemcinin performanslı yükleme altyapısının temel veri tanımlarını içerir ve diğer yükleme bileşenleri için bir temel oluşturur.

---

### `PythonLoading.hpp` (İstemci Yükleme Veri ve Sabitleri)

Bu başlık dosyası, `ENABLE_LOADING_PERFORMANCE` makrosu etkinleştirildiğinde, istemcinin optimize edilmiş yükleme süreci (`CPythonLoading` sınıfı tarafından yönetilir) için gerekli olan çok sayıda sabit, enum, global değişken ve statik veri koleksiyonunu tanımlar. `PythonLoadingUtils.hpp` dosyasında tanımlanan veri yapılarını kullanarak bu koleksiyonları oluşturur.

#### Dosyanın Genel Amaçları

*   **Sabit ve Enum Tanımları**:
    *   `EEmotionIcon`: Duygu (emoticon) ikonları için enum.
    *   `EComboType`, `EComboIndex`: Karakter saldırı komboları için tür ve indeks enumları.
    *   `EDustGap`: Karakter ve at hareketlerinde oluşacak toz efektleri için aralık sabitleri.
    *   `EHorseSkill`: Ata özel yetenekler için hareket indeksi sabitleri.
    *   `EGuildSkill`: Lonca yetenekleri için hareket indeksi sabitleri.
    *   `ELanguageSkill`: İmparatorluk dili yetenekleri için sabitler.
    *   `ENPCListToken`: `npclist.txt` dosyasındaki sütun (token) sıralamasını tanımlayan enum.
    *   `EGuildBuildingToken`: Lonca bina listesi dosyasındaki sütun sıralamasını tanımlayan enum.
    *   `EGuildBuildingMaterialItem`, `EGuildBuildingMaterialIndex`: Lonca binası yapım malzemeleri ve bunların indeksleri için enumlar.
    *   `g_iSafeSleep`, `g_bCacheMotionData`: Yükleme sırasında kullanılacak bekleme süresi ve hareket verilerinin önbelleğe alınıp alınmayacağı gibi genel ayarlar.
    *   `g_stNPCList_FileName`, `g_stRaceHeight_FileName`: NPC listesi ve ırk yükseklik verilerinin okunacağı dosya adları.
*   **Statik Veri Koleksiyonları (Global Değişkenler)**:
    *   `g_vUseSoundData`, `g_vDropSoundData`: `SSoundData` yapılarından oluşan vektörler; eşya kullanım ve düşme seslerini tanımlar.
    *   `g_vEffectData`: `SEffectData` yapılarından oluşan vektör; genel oyun efektlerini (toz, vuruş, iyileştirme, buff vb.) tanımlar.
    *   `g_vFlyEffectData`: `SFlyEffectData` yapılarından oluşan vektör; uçan efektleri (EXP kazanımı, büyü okları vb.) tanımlar.
    *   `g_vEmotionEffectData`: `SEmoticonEffectData` yapılarından oluşan vektör; duygu (emoticon) efektlerini ve komutlarını tanımlar.
    *   `g_vRaceData`: `SRaceData` yapılarından oluşan vektör; karakter ırklarının model dosyalarını ve yollarını tanımlar.
    *   `g_vIntroMotionData`, `g_vGeneralMotionData`, `g_vNewGeneralMotionData`, `g_vActionMotionData`, `g_vWeddingMotionData`, `g_vFishingMotionData`, `g_vGuildSkillMotionData`: `SMotionData` yapılarından oluşan vektörler; karakterlerin farklı durumlar (giriş ekranı, genel, aksiyon, evlilik, balıkçılık) ve lonca yetenekleri için animasyonlarını tanımlar.
    *   `g_vMapPlayerSkillIndex`: Karakter sınıfları ve yetenek gruplarına göre oyuncu yetenek indekslerini içeren karmaşık bir `std::map`.
    *   `g_vSupportSkillIndex`: Destek (pasif) yetenek indekslerini içeren bir vektör.
    *   `g_vMapGuildSkillIndex`: Lonca yeteneklerini "PASSIVE" ve "ACTIVE" olarak ayıran bir `std::map`.
    *   `g_vDungeonMapName`: Zindan haritalarının isimlerini içeren bir vektör.
    *   `g_vTitleName`: Oyuncu unvanlarını içeren bir vektör. `ENABLE_GUILD_LEADER_GRADE_NAME` aktifse lonca lideri unvanları için `g_vGuildLeaderGradeTitleName` de bulunur.
    *   `g_mNameColor`, `g_mTitleNameColor`: Karakter ve NPC isimleri ile unvan renklerini tanımlayan `std::map`'ler.
    *   `g_vEmotionIcon`: Duygu ikonlarının dosya yollarını içeren bir `std::map`.
    *   `g_mGuildBuilding`: Lonca bina türlerini ve ilgili model adlarını içeren bir `std::map`.
*   **Yardımcı Fonksiyonlar**:
    *   `GetGuildMaterialIndex()`: Verilen malzeme VNUM'una göre lonca binası malzeme indeksini döndürür.
    *   `GetLocaleInfoString()`: `localeInfo` Python modülünden yerelleştirilmiş metin alır (prototipi `PythonLoadingUtils.hpp`'de, implementasyonu burada).
    *   `IsNumber()`: Verilen bir string'in tamamen sayılardan oluşup oluşmadığını kontrol eder.
*   **Global Değişken Tanımları (extern)**:
    *   Yetenek indeksleri (`c_iSkillIndex_*`) ve duygu ikonları haritası (`m_kMap_iEmotionIndex_pkIconImage`) gibi bazı global değişkenlerin `extern` bildirimlerini içerir. Bu değişkenlerin asıl tanımları genellikle `.cpp` dosyalarında yer alır.

#### C++ ve Python Arayüzü ile İlişkisi

Bu dosya, doğrudan Python'a bir modül sunmaz. İçerdiği sabitler ve statik veri koleksiyonları, `CPythonLoading` sınıfının C++ implementasyonu tarafından oyun verilerini yüklemek ve yapılandırmak için kullanılır. `GetLocaleInfoString` gibi fonksiyonlar, C++ tarafının Python ile etkileşimde bulunmasına yardımcı olur.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Merkezi Veri Tanımlama**: Oyunun yükleme aşamasında ihtiyaç duyulan çok sayıda konfigürasyon verisini (efekt yolları, animasyon dosyaları, yetenek bağlantıları vb.) tek bir merkezi yerde toplar.
*   **Optimize Edilmiş Yükleme Altyapısı**: `CPythonLoading` sınıfı, bu dosyada tanımlanan verileri kullanarak oyun kaynaklarını (ses, grafik, animasyon) verimli bir şekilde yükler.
*   **Oyun Mantığı Entegrasyonu**: Karakterlerin hangi animasyonları kullanacağı, hangi seslerin çalınacağı, hangi efektlerin gösterileceği gibi oyunun temel işleyişiyle ilgili birçok detay bu verilerle belirlenir.
*   **Kolay Yapılandırma ve Bakım**: Yeni oyun varlıkları (örneğin, yeni bir efekt veya animasyon) eklemek veya mevcutları değiştirmek, genellikle bu dosyada ilgili veri koleksiyonunu güncellemekle mümkün olur, bu da bakımı kolaylaştırır.

Bu dosya, istemcinin yükleme sürecinde kullanılacak temel oyun verilerinin ve yapılandırmalarının büyük bir kısmını içerir ve `CPythonLoading` sisteminin düzgün çalışması için kritik öneme sahiptir.

---

### `PythonLoading.h` (İstemci Yükleme Yöneticisi Sınıfı)

Bu başlık dosyası, `ENABLE_LOADING_PERFORMANCE` makrosu etkinleştirildiğinde kullanılan `CPythonLoading` adlı bir singleton C++ sınıfını tanımlar. Bu sınıf, istemcinin çeşitli oyun verilerini (sesler, efektler, NPC'ler, karakter ırkları ve animasyonları, harita bilgileri, unvanlar, renkler vb.) ayrı bir iş parçacığında (thread) yükleyerek oyunun başlangıç ve harita geçiş sürelerini optimize etmekten sorumludur.

#### Dosyanın Genel Amaçları

*   **`CPythonLoading` Sınıf Bildirimi**: `CSingleton<CPythonLoading>`'den miras alan ve optimize edilmiş yükleme işlemlerini yöneten sınıfı deklare eder.
*   **Temel Kontrol Fonksiyonları (Public)**:
    *   `BeginThreadLoading()`: Veri yükleme işlemini ayrı bir iş parçacığında başlatır.
    *   `JoinThreadLoading()`: Başlatılan yükleme iş parçacığının tamamlanmasını bekler.
    *   `EndThreadLoading()`: Yükleme iş parçacığını sonlandırır ve ilgili kaynakları (örneğin, `CResourceManager`) temizler.
    *   `LoadBackground()`: Belirli oyuncu koordinatlarına göre arka planı ve ilgili oyun verilerini yükler, karakter yeteneklerini kaydeder ve oyunu başlatır.
    *   `RegisterSkills()`: Verilen karakter sınıfı, yetenek grubu ve imparatorluğa göre oyuncunun yeteneklerini `CPythonPlayer`'a kaydeder.
    *   `LoadGuildBuilding()`: Verilen bir dosyadan lonca binası listesini yükler ve `CPythonGuild`'e kaydeder.
*   **Özel (Private) Yükleme Fonksiyonları**:
    *   `__LoadData()`: Ana yükleme fonksiyonu; diğer tüm özel `__Load*` fonksiyonlarını çağırarak verileri sırayla yükler. Bu fonksiyon genellikle ayrı bir thread'de çalışır.
    *   `__Initialize()`: Temel başlangıç ayarlarını yapar (örneğin, `CInstanceBase` için toz efekti aralıkları, `CPythonPlayer` için temel efektler).
    *   `__LoadSound()`: Eşya kullanım ve düşme seslerini `PythonLoading.hpp`'deki tanımlamalara göre `CPythonItem`'a kaydeder.
    *   `__LoadEffect()`: Genel oyun efektlerini, uçan efektleri ve duygu efektlerini `PythonLoading.hpp`'deki tanımlamalara göre `CInstanceBase`, `CFlyingManager` ve `CPythonNetworkStream`'e kaydeder.
    *   `__LoadNPCList()`: `npclist.txt` dosyasını okuyarak NPC VNUM'larını ve model adlarını `CRaceManager`'a kaydeder.
    *   `__LoadRaceHeight()`: `race_height.txt` dosyasını okuyarak ırk yüksekliklerini `CRaceManager`'a kaydeder.
    *   `__LoadRace()`: `PythonLoading.hpp`'de tanımlı ırk verilerine göre her bir karakter ırkını (`CRaceData`) oluşturur, ilgili `.msm` (model) ve `.msa` (animasyon) dosyalarını yükler. Her karakter sınıfı için özel `__Load<KarakterSınıfı>Ex` fonksiyonlarını çağırır.
    *   `__LoadWarriorEx()`, `__LoadAssassinEx()`, `__LoadSuraEx()`, `__LoadShamanEx()`, `__LoadWolfmanEx()`: Belirli karakter sınıfları için genel, yetenek, duygu, savaş modu (tek el, çift el, yay vb.), balıkçılık ve at animasyonlarını (`.msa` dosyaları) `PythonLoading.hpp`'deki tanımlamalara göre yükler ve `CRaceData`'ya kaydeder. Ayrıca, kemik bağlantı noktalarını (örneğin, silahın bağlanacağı kemik) tanımlar.
    *   `__LoadComboType()`, `__LoadHorseComboType`: Farklı silah ve savaş modları için saldırı kombo sıralamalarını `CRaceData`'ya kaydeder.
    *   `__SetIntroMotions()`, `__SetGeneralMotion()`, `__SetNewGeneralMotion()`: Karakterler için giriş ekranı ve genel durum animasyonlarını yükler.
    *   `__LoadMotionData()`, `__RegisterCacheMotionData()`: Verilen yoldan hareket (animasyon) dosyalarını yükler ve isteğe bağlı olarak `CResourceManager` ile önbelleğe alır.
    *   `__RegisterEmotionAnis()`, `__LoadAction()`, `__LoadWedding()`, `__LoadFishing()`: Duygu, aksiyon, evlilik ve balıkçılık animasyonlarını yükler.
    *   `__RegisterDungeonMapName()`: Zindan harita isimlerini `CPythonBackground`'a kaydeder.
    *   `__RegisterTitleName()`: Oyuncu unvanlarını `CInstanceBase`'e kaydeder. `ENABLE_GUILD_LEADER_GRADE_NAME` aktifse lonca lideri unvanlarını da kaydeder.
    *   `__RegisterColor()`: İsim ve unvan renklerini `CInstanceBase`'e kaydeder.
    *   `__RegisterEmotionIcon()`: Duygu ikonlarını `m_kMap_iEmotionIndex_pkIconImage` haritasına yükler.
*   **Üye Değişkenler**:
    *   `ms_mutex`: Yükleme işlemleri sırasında iş parçacığı güvenliği için kullanılan bir `std::mutex`.
    *   `ms_thread`: Veri yükleme işlemlerini yürüten `std::thread` nesnesi.
    *   `ms_thread_load`, `ms_thread_joined`: İş parçacığının yüklenip yüklenmediğini ve ana iş parçacığına katılıp katılmadığını belirten atomik boolean bayraklar.

#### C++ ve Python Arayüzü ile İlişkisi

Bu sınıfın C++ fonksiyonları, oyunun yükleme mantığını yönetir. Python'a doğrudan bir arayüz sunmaz, ancak `CPythonLoading.cpp` dosyasında bu sınıfın bazı işlevlerini (özellikle `RegisterSkills`) kullanan `loading` adlı bir Python modülü tanımlanır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Hızlı Başlangıç ve Harita Geçişi**: Oyun verilerini ayrı bir iş parçacığında yükleyerek ana iş parçacığının bloke olmasını engeller, böylece oyunun daha hızlı başlamasına ve harita geçişlerinin daha akıcı olmasına katkıda bulunur.
*   **Merkezi Yükleme Yönetimi**: İstemcinin ihtiyaç duyduğu tüm temel verilerin (modeller, animasyonlar, sesler, efektler, yapılandırma dosyaları) yüklenmesini merkezi bir yerden yönetir.
*   **Performans İyileştirmesi**: Özellikle `ENABLE_LOADING_PERFORMANCE` aktif olduğunda, kaynakların organize bir şekilde yüklenmesini sağlayarak genel istemci performansını artırmayı hedefler.
*   **Ön Yükleme ve Önbellekleme**: Gerekli verileri önceden yükleyerek ve `CResourceManager` aracılığıyla önbelleğe alarak oyun içi takılmaları azaltmaya yardımcı olur.

Bu sınıf, modern oyun istemcilerinde yaygın olarak kullanılan, yükleme sürelerini kısaltmaya yönelik optimize edilmiş bir yükleme yöneticisinin temelini oluşturur.

---

### `PythonLoading.cpp` (İstemci Yükleme Yöneticisi Implementasyonu ve Python `loading` Modülü)

Bu dosya, `ENABLE_LOADING_PERFORMANCE` makrosu etkinleştirildiğinde, `PythonLoading.h` dosyasında bildirilen `CPythonLoading` singleton C++ sınıfının tüm metotlarının C++ implementasyonlarını ve `CPythonLoading` sınıfının bazı işlevlerini Python betiklerine sunan `loading` adlı bir Python modülünü içerir. Temel amacı, istemcinin oyun verilerini (sesler, efektler, NPC'ler, karakter ırkları ve animasyonları, harita bilgileri, unvanlar, renkler, lonca binaları vb.) verimli bir şekilde, genellikle ayrı bir iş parçacığında yüklemektir.

#### `CPythonLoading` Sınıfı Implementasyon Detayları

*   **Yapıcı ve Yıkıcı (`CPythonLoading`, `~CPythonLoading`)**:
    *   Yapıcı, iş parçacığı bayraklarını (`ms_thread_load`, `ms_thread_joined`) `false` olarak başlatır.
    *   Yıkıcı, `EndThreadLoading()` çağırarak iş parçacığının düzgün bir şekilde sonlandırılmasını sağlar.

*   **İş Parçacığı Yönetimi (`BeginThreadLoading`, `JoinThreadLoading`, `EndThreadLoading`)**:
    *   `BeginThreadLoading()`: Eğer `ms_thread_load` `false` ise, `__LoadData()` özel metodunu çalıştıracak yeni bir `std::thread` oluşturur ve `ms_thread_load`'u `true` yapar. Kısa bir bekleme (`Sleep(g_iSafeSleep)`) uygular.
    *   `JoinThreadLoading()`: Eğer iş parçacığı birleştirilebilir (`joinable()`) ve henüz birleştirilmemişse (`!ms_thread_joined`), `ms_thread.join()` ile ana iş parçacığının yükleme iş parçacığının bitmesini beklemesini sağlar ve `ms_thread_joined`'ı `true` yapar.
    *   `EndThreadLoading()`: Eğer iş parçacığı birleştirilebilirse onu birleştirir. Bayrakları sıfırlar ve `CResourceManager` singleton'ının yıkıcılarını çağırarak kaynakları temizler.

*   **Ana Veri Yükleme Fonksiyonu (`__LoadData`)**:
    *   Bu statik metod, bir `std::lock_guard<std::mutex>` kullanarak `ms_mutex` ile iş parçacığı güvenliğini sağlar.
    *   Sırasıyla aşağıdaki özel yükleme fonksiyonlarını çağırır:
        *   `__Initialize()`: Temel başlatma işlemleri.
        *   `__LoadSound()`: Ses dosyası yollarını ayarlar.
        *   `__LoadEffect()`: Efektleri kaydeder.
        *   `__LoadNPCList()`: `npclist.txt` dosyasından NPC verilerini yükler.
        *   `__LoadRaceHeight()`: `race_height.txt` dosyasından ırk yüksekliklerini yükler.
        *   `__LoadRace()`: Karakter ırklarını ve animasyonlarını yükler.
        *   `__RegisterDungeonMapName()`: Zindan harita isimlerini kaydeder.
        *   `__RegisterTitleName()`: Oyuncu unvanlarını kaydeder.
        *   `__RegisterGuildLeaderGradeName()` (`ENABLE_GUILD_LEADER_GRADE_NAME` ile): Lonca lideri unvanlarını kaydeder.
        *   `__RegisterColor()`: İsim ve unvan renklerini kaydeder.
        *   `__RegisterEmotionIcon()`: Duygu ikonlarını yükler.
    *   `__LOADING_PERFORMANCE_CHECK__` makrosu aktifse, her bir yükleme adımının ne kadar sürdüğünü ölçer ve `perf_all_load.txt` dosyasına yazar.

*   **Arka Plan ve Oyun Başlatma (`LoadBackground`)**:
    *   Verilen oyuncu X ve Y koordinatlarına göre `CPythonNetworkStream::Instance().Warp()` çağırır.
    *   Ana karakterin sınıf, yetenek grubu ve imparatorluk bilgilerini alarak `RegisterSkills()` fonksiyonunu çağırır.
    *   `CPythonBackground` için görüş mesafesini ayarlar ve harita hazırsa global merkez pozisyonunu günceller.
    *   `CPythonNetworkStream::Instance().StartGame()` ile oyunun başlamasını tetikler.

*   **Detaylı Yükleme Fonksiyonları (`__Initialize`, `__LoadSound`, `__LoadEffect`, vb.)**:
    *   Bu fonksiyonlar, `PythonLoading.hpp` dosyasında tanımlanan statik veri koleksiyonlarını (örn: `g_vUseSoundData`, `g_vEffectData`, `g_vRaceData`) kullanarak ilgili oyun yöneticilerine (örn: `CPythonItem`, `CInstanceBase`, `CRaceManager`, `CFlyingManager`) verileri kaydeder.
    *   `__LoadNPCList`, `__LoadRaceHeight`, `LoadGuildBuilding`: Metin tabanlı (`.txt`) yapılandırma dosyalarını `CEterPackManager`, `CMappedFile` ve `CMemoryTextFileLoader` (linter hatası bu sınıfla ilgili, muhtemelen başlık dosyası eksik veya yanlış yerde include edilmiş) kullanarak okur, satırları ve sütunları ayrıştırır ve verileri ilgili yöneticilere kaydeder. `LoadGuildBuilding` ayrıca Python listesi ve sözlüğü oluşturarak `CPythonGuild`'e veri aktarır.
    *   `__LoadRace` ve karakter sınıflarına özel `__Load<Sınıf>Ex` fonksiyonları (`__LoadWarriorEx`, `__LoadAssassinEx` vb.): Her karakter sınıfı için model (`.msm`), temel animasyonlar (`.msa`), farklı silah türlerine göre savaş animasyonları, yetenek animasyonları, duygu, aksiyon, evlilik, balıkçılık ve at animasyonlarını yükler. `__LoadMotionData`, `__RegisterCacheMotionData` gibi yardımcı fonksiyonları kullanır. Kombo türlerini (`__LoadComboType`, `__LoadHorseComboType`) ve ekipmanların bağlanacağı kemik isimlerini (`RegisterAttachingBoneName`) ayarlar.

*   **Yetenek Kaydı (`RegisterSkills`)**:
    *   Verilen karakter sınıfı (`c_bJob`), yetenek grubu (`c_bSkillGroupIndex`) ve imparatorluğa (`c_bEmpire`) göre `g_vMapPlayerSkillIndex` (oyuncu yetenekleri), `g_vSupportSkillIndex` (destek yetenekleri) ve `g_vMapGuildSkillIndex` (lonca yetenekleri) gibi `PythonLoading.hpp`'deki haritalardan ve vektörlerden yetenek ID'lerini alarak `CPythonPlayer::Instance().SetSkill()` ile oyuncuya atar. Dil yetenekleri de imparatorluğa göre ayarlanır.

#### `loading` Python Modülü

Bu bölüm, `CPythonLoading` sınıfının bazı işlevlerini Python betiklerine açan `loading` adlı bir Python modülünü tanımlar ve başlatır.

*   **`initLoading()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `loading` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyonları tanımlar:
        *   `loadingRegisterSkill`: C++ tarafındaki `CPythonLoading::Instance().RegisterSkills()` fonksiyonunu çağıran bir sarmalayıcıdır. Python'dan karakter sınıfı, yetenek grubu ve imparatorluk alır.
    *   `Py_InitModule("loading", s_methods);` ile `loading` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak çeşitli sabitler (iş sınıfları, ırk numaraları, at yetenekleri, yükleme türleri) Python modülüne (`loading.JOB_WARRIOR`, `loading.RACE_WARRIOR_M` vb.) aktarılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Performans Odaklı Veri Yükleme**: İstemcinin ihtiyaç duyduğu büyük miktardaki oyun verisini (modeller, animasyonlar, sesler, efektler, metin dosyaları) ayrı bir iş parçacığında ve organize bir şekilde yükleyerek oyunun açılışını ve harita geçişlerini hızlandırır.
*   **Merkezi Yapılandırma**: Oyunun temel varlıklarının (karakterler, NPC'ler, yetenekler) nasıl yükleneceği ve hangi dosyaların kullanılacağı `PythonLoading.hpp`'deki statik veriler ve bu dosyadaki yükleme fonksiyonları aracılığıyla merkezi olarak yönetilir.
*   **Python Entegrasyonu**: `loading` modülü aracılığıyla, Python scriptlerinin (özellikle karakter oluşturma veya oyun başlangıcı ile ilgili scriptlerin) oyuncu yeteneklerini C++ tarafındaki yükleme mekanizmasıyla senkronize bir şekilde ayarlamasına olanak tanır.
*   **Kaynak Yönetimi**: Yükleme işlemleri tamamlandığında veya sonlandırıldığında `CResourceManager` gibi kaynak yöneticilerin temizlenmesini sağlar.
*   **Hata Ayıklama ve Profilleme**: `__LOADING_PERFORMANCE_CHECK__` gibi makrolar sayesinde yükleme sürelerinin detaylı bir şekilde analiz edilmesine imkan tanır.

Bu dosya, `UserInterface` modülünün en karmaşık ve kritik performans iyileştirmelerinden birini içerir. Oyun verilerinin verimli yüklenmesi, genel kullanıcı deneyimi için hayati öneme sahiptir. Linter hatalarının `CMemoryTextFileLoader` ile ilgili olması, bu sınıfın başlık dosyasının `StdAfx.h` veya `PythonLoading.cpp` içinde doğru bir şekilde include edilmemiş olabileceğine işaret eder.

---

### `PythonMailBox.h` (Python Posta Kutusu Sistemi Başlık Dosyası)

Bu başlık dosyası, `ENABLE_MAILBOX` makrosu etkinleştirildiğinde kullanılan `CPythonMailBox` adlı bir singleton C++ sınıfını ve onunla ilişkili veri yapılarını tanımlar. Bu sistem, oyuncuların birbirlerine mesaj ve eşya göndermelerini sağlayan oyun içi posta kutusu işlevselliğinin istemci tarafındaki mantığını yönetir.

#### Dosyanın Genel Amaçları

*   **`CPythonMailBox` Sınıf Bildirimi**:
    *   `CSingleton<CPythonMailBox>`'den miras alır.
    *   Posta kutusundaki postaları (`vecMail` içinde) yönetir.
    *   Posta ekleme (`AddMail`), posta alma (`GetMail`), tüm postaları temizleme (`Destroy`) gibi temel yönetim fonksiyonlarını içerir.
*   **`SMailBoxAddData` Yapısı**:
    *   Bir postanın detaylı içeriğini (gönderen, mesaj, Yang, Won, eşya VNUM'u, eşya sayısı, eşya soketleri, eşya efsunları vb.) tutar.
    *   Bu yapı, `#if defined` blokları aracılığıyla çeşitli sistemlere (örneğin `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_REFINE_ELEMENT`, `ENABLE_SET_ITEM`, `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10`) bağlı olarak ek alanlar içerebilir.
    *   Yapıcı ve yıkıcı metotlara sahiptir.
*   **`SMailBox` Yapısı**:
    *   Bir postanın genel bilgilerini (gönderilme zamanı, silinme zamanı, başlık, GM postası olup olmadığı, eşya içerip içermediği, onaylanıp onaylanmadığı) ve `SMailBoxAddData` tipinde bir işaretçi (`AddData`) içerir.
    *   `ResetAddData()` metodu ile `AddData`'nın işaret ettiği belleği serbest bırakır.
    *   Yapıcı ve yıkıcı metotlara sahiptir.
*   **Tür Tanımları (`typedef`)**:
    *   `MailVec`: `std::vector<SMailBox*>` için bir tür takma adıdır ve posta listesini tutmak için kullanılır.
*   **Enum Tanımları**:
    *   `EMAILBOX_GC`: Sunucudan istemciye gönderilen posta kutusu ile ilgili paket başlıklarını (opcode) tanımlar (örneğin, `MAILBOX_GC_OPEN`, `MAILBOX_GC_SET`, `MAILBOX_GC_ADD_DATA`).
    *   `EMAILBOX_CG`: İstemciden sunucuya gönderilen posta kutusu ile ilgili paket başlıklarını tanımlar (örneğin, `MAILBOX_CG_CLOSE`, `MAILBOX_CG_GET_ITEMS`).
    *   `EMAILBOX_POST_ALL_DELETE`, `EMAILBOX_POST_ALL_GET_ITEMS`, `EMAILBOX_POST_DELETE_FAIL`, `EMAILBOX_POST_GET_ITEMS`, `EMAILBOX_POST_WRITE`: Posta işlemlerinin (tümünü silme, tüm eşyaları alma, tek posta silme, eşya alma, posta yazma) sonuçlarını belirten durum kodlarını tanımlar (örneğin, `POST_ALL_DELETE_OK`, `POST_GET_ITEMS_FAIL_BLOCK_CHAR`, `POST_WRITE_INVALID_NAME`).

#### C++ ve Python Arayüzü ile İlişkisi

*   `CPythonMailBox` sınıfı ve içindeki yapılar, posta kutusu verilerini C++ tarafında yönetir.
*   `CPythonMailBox.cpp` dosyasında, bu sınıfın bazı fonksiyonları ve veri yapıları kullanılarak `mail` adında bir Python modülü oluşturulur. Bu modül, Python tabanlı UI betiklerinin posta kutusu verilerine erişmesini ve bazı temel işlemleri yapmasını sağlar.
*   Enum'larda tanımlanan sabitler, Python modülüne de aktarılarak sunucu ile iletişimde ve UI'da durum gösteriminde kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Posta Alma ve Görüntüleme**: Sunucudan gelen postaları `SMailBox` ve `SMailBoxAddData` yapılarında saklar ve UI'da listelenmesini sağlar.
*   **Posta İçeriği Detayları**: Oyuncunun bir postayı seçtiğinde, gönderen, mesaj, para miktarı ve ekli eşyanın detayları (efsunlar, soketler vb.) bu yapılar aracılığıyla UI'a iletilir.
*   **Posta İşlemleri**: Oyuncunun posta silme, eşya alma gibi eylemlerini sunucuya iletmek ve sunucudan gelen yanıtları işlemek için `EMAILBOX_CG` ve `EMAILBOX_GC` opcode'larını kullanır.
*   **Durum Yönetimi**: Posta işlemlerinin başarılı olup olmadığını (`POST_WRITE_OK`, `POST_GET_ITEMS_FAIL` vb.) enum sabitleri aracılığıyla takip eder ve UI'da uygun geri bildirimlerin verilmesini sağlar.

Bu başlık dosyası, istemcinin posta kutusu sistemi için gerekli olan tüm veri yapılarını, arayüzleri ve durum kodlarını tanımlayarak sistemin C++ ve Python katmanları arasında tutarlı bir şekilde çalışmasını sağlar.

---

### `PythonMailBox.cpp` (Python Posta Kutusu Sistemi Implementasyonu ve `mail` Modülü)

Bu C++ kaynak dosyası, `ENABLE_MAILBOX` makrosu etkinleştirildiğinde, `PythonMailBox.h` dosyasında bildirilen `CPythonMailBox` singleton sınıfının metotlarını ve ilişkili yapılarının (`SMailBox`, `SMailBoxAddData`) yapıcı/yıkıcılarını implemente eder. Ayrıca, `CPythonMailBox` sınıfının işlevlerini Python betiklerine sunan `mail` adlı bir Python modülü tanımlar.

#### `CPythonMailBox` Sınıfı Implementasyon Detayları

*   **Yapıcı (`CPythonMailBox::CPythonMailBox()`)**: Boş bir yapıcıdır, özel bir başlatma yapmaz.
*   **Yıkıcı (`CPythonMailBox::~CPythonMailBox()`)**: `Destroy()` metodunu çağırarak tüm posta verilerini temizler.
*   **`Destroy()`**: `vecMail` (postaları tutan vektör) içindeki tüm `SMailBox` işaretçilerini dolaşır, her bir `SMailBox` nesnesini `delete` ile siler ve ardından vektörü temizler (`vecMail.clear()`).
*   **`GetMailVec()`**: Posta vektörüne (`vecMail`) sabit bir referans döndürür.
*   **`ResetAddData(const BYTE Index)`**: Belirtilen indeksteki postanın (`SMailBox`) `bIsConfirm` bayrağını `true` yapar ve postanın `SMailBoxAddData` işaretçisi (`AddData`) varsa onu sıfırlar (`mail->ResetAddData()`). Bu genellikle bir posta okunduğunda veya içeriği alındığında çağrılır.
*   **`AddMail(CPythonMailBox::SMailBox* mail)`**: Yeni bir `SMailBox` işaretçisini `vecMail` vektörüne ekler.
*   **`GetMail(const BYTE Index)`**: Verilen indeksteki `SMailBox` işaretçisini `vecMail` vektöründen döndürür. Eğer indeks geçersizse `nullptr` döndürür.
*   **`GetMailAddData(const BYTE Index)`**: Belirtilen indeksteki postanın `SMailBoxAddData` işaretçisini döndürür. Eğer posta veya `AddData` yoksa `nullptr` döndürür.

#### `SMailBox` Yapısı Implementasyonu

*   **Yapıcı (`CPythonMailBox::SMailBox::SMailBox(...)`)**: Aldığı parametrelerle (gönderilme zamanı, silinme zamanı, başlık, GM postası mı, eşya var mı, onaylandı mı) üye değişkenleri başlatır. `AddData` işaretçisini başlangıçta `nullptr` yapar.
*   **Yıkıcı (`CPythonMailBox::SMailBox::~SMailBox()`)**: `ResetAddData()` metodunu çağırarak `AddData` tarafından tutulan belleği serbest bırakır.
*   **`ResetAddData()`**: `AddData` işaretçisi `nullptr` değilse, işaret ettiği `SMailBoxAddData` nesnesini `delete` ile siler ve `AddData`'yı `nullptr` yapar.

#### `SMailBoxAddData` Yapısı Implementasyonu

*   **Yapıcı (`CPythonMailBox::SMailBoxAddData::SMailBoxAddData(...)`)**: Aldığı parametrelerle (gönderen, mesaj, Yang, Won, eşya VNUM, eşya sayısı vb.) üye değişkenleri başlatır. Eşya soketleri (`alSockets`), rastgele efsunlar (`aApplyRandom`), min/max değerler (`alMinMaxValue`) ve normal efsunlar (`aAttr`) gibi dizi tipindeki verileri `std::memcpy` kullanarak kopyalar.
*   **Yıkıcı (`CPythonMailBox::SMailBoxAddData::~SMailBoxAddData()`)**: Boş bir yıkıcıdır, çünkü bu yapı içinde dinamik olarak ayrılan başka bir bellek yönetmez (diziler sabit boyutludur ve `std::string` kendi belleğini yönetir).

#### `mail` Python Modülü

Bu bölüm, `CPythonMailBox` sınıfının işlevlerini Python'a açan fonksiyonları tanımlar.

*   **`initmail()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `mail` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyonları tanımlar:
        *   `mailGetMailAddData(PyObject* poSelf, PyObject* poArgs)`: Verilen indeksteki postanın gönderenini, mesajını, Yang ve Won miktarını Python demeti olarak döndürür.
        *   `mailGetMailData(PyObject* poSelf, PyObject* poArgs)`: Şu an için sadece `Py_BuildNone()` döndürüyor, yani implemente edilmemiş veya farklı bir amaç için yer tutucu olabilir.
        *   `mailGetMailDict(PyObject* poSelf, PyObject* poArgs)`: Tüm postaların genel bilgilerini (indeks, gönderilme zamanı, silinme zamanı, başlık, GM postası mı, eşya var mı, onaylandı mı) içeren bir Python sözlüğü döndürür.
        *   `mailGetMailItemData(PyObject* poSelf, PyObject* poArgs)`: Verilen indeksteki postada bulunan eşyanın VNUM'unu ve sayısını döndürür. Eşya yoksa `None` döner.
        *   `mailGetItemApplyRandom(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_APPLY_RANDOM` ile): Belirli bir posta ve efsun slot indeksi için rastgele efsun türünü ve değerini döndürür.
        *   `mailGetItemMinMaxValue(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_ITEM_VALUE10` ile): Belirli bir posta ve min/max değer indeksi için eşyanın min/max değerini döndürür.
        *   `mailGetMailItemAttribute(PyObject* poSelf, PyObject* poArgs)`: Belirli bir posta ve efsun slot indeksi için eşyanın efsun türünü ve değerini döndürür.
        *   `mailGetMailItemMetinSocket(PyObject* poSelf, PyObject* poArgs)`: Belirli bir posta ve soket indeksi için eşyanın metin taşı soket değerini döndürür.
        *   `mailGetItemChangeLookVnum(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_CHANGE_LOOK_SYSTEM` ile): Belirli bir postadaki eşyanın görünüm değiştirme VNUM'unu döndürür.
        *   `mailGetItemRefineElement(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_REFINE_ELEMENT` ile): Belirli bir postadaki eşyanın element bilgisini döndürür.
        *   `mailGetItemSetValue(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_SET_ITEM` ile): Belirli bir postadaki eşyanın set bonus değerini döndürür.
    *   `Py_InitModule("mail", s_methods);` ile `mail` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak `PythonMailBox.h` dosyasında tanımlanan `EMAILBOX_GC`, `EMAILBOX_POST_ALL_DELETE`, `EMAILBOX_POST_ALL_GET_ITEMS`, `EMAILBOX_POST_DELETE_FAIL`, `EMAILBOX_POST_GET_ITEMS`, `EMAILBOX_POST_WRITE` enumlarındaki tüm sabitler Python modülüne (`mail.MAILBOX_GC_OPEN`, `mail.POST_WRITE_OK` vb.) aktarılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **İstemci Veri Yönetimi**: Sunucudan gelen posta verilerini C++ tarafında `CPythonMailBox` içinde saklar ve yönetir.
*   **UI Güncellemesi**: Python tabanlı posta kutusu arayüzü, `mail` modülü aracılığıyla `CPythonMailBox`'tan posta listesini, posta detaylarını ve ekli eşya bilgilerini çekerek kullanıcıya gösterir.
*   **Durum Kodları**: Sunucudan gelen işlem sonuçları (örneğin, posta gönderme başarılı, eşya alma başarısız - envanter dolu) ve istemciden sunucuya gönderilecek komutlar için tanımlanan sabitler, hem C++ hem de Python tarafında tutarlı bir iletişim sağlar.
*   **Eşya Detayları**: Posta ile gelen eşyaların tüm özelliklerinin (soketler, efsunlar, görünüm, element vb.) doğru bir şekilde kullanıcıya sunulmasını sağlar.

Bu dosya, posta kutusu sisteminin istemci tarafındaki mantığının çekirdeğini oluşturur ve C++ ile Python arasındaki köprüyü kurarak UI'ın bu sistemle etkileşimini mümkün kılar.

---

### `PythonMessenger.h` (Python Messenger Sistemi Başlık Dosyası)

Bu başlık dosyası, `ENABLE_MESSENGER` makrosu etkinleştirildiğinde kullanılan `CPythonMessenger` adlı bir singleton C++ sınıfını ve onunla ilişkili veri yapılarını tanımlar. Bu sistem, oyuncuların birbirlerine mesaj ve eşya göndermelerini sağlayan oyun içi mesajlaşma işlevselliğinin istemci tarafındaki mantığını yönetir.

#### Dosyanın Genel Amaçları

*   **`CPythonMessenger` Sınıf Bildirimi**:
    *   `CSingleton<CPythonMessenger>`'den miras alır.
    *   Arkadaşları (`m_FriendNameMap`), GM'leri (`m_GMNameMap`), engellenen kullanıcıları (`m_BlockNameMap`) ve lonca üyesi durumlarını (`m_GuildMemberStateMap`) yönetir.
    *   Mesajlaşma işlevlerini içerir: arkadaş ekleme, arkadaş kaldırma, GM ekleme, GM kaldırma, engellenen kullanıcı ekleme, engellenen kullanıcı kaldırma, arkadaş listesi güncelleme, arkadaş listesi temizleme, lonca üyesi ekleme, lonca üyesi kaldırma, lonca üyesi listesi güncelleme, lonca üyesi listesi temizleme.
*   **Arkadaş Yönetimi**:
    *   `RemoveFriend(const char* c_szName)`: Verilen ismi `m_FriendNameMap` setinden siler.
    *   `OnFriendLogin(const char* c_szName)`: Verilen ismi `m_FriendNameMap`'e ekler. Eğer `m_poMessengerHandler` ayarlıysa, Python'daki `OnLogin` fonksiyonunu `MESSENGER_GRUOP_INDEX_FRIEND` grup indeksi ve isim ile çağırır.
    *   `OnFriendLogout(const char* c_szName)`: Verilen ismi `m_FriendNameMap`'e ekler (Bu, `OnFriendLogin` ile aynı davranışı sergiliyor ve bir mantık hatası olabilir; normalde listeden silme veya durumu güncelleme beklenir). Eğer `m_poMessengerHandler` ayarlıysa, Python'daki `OnLogout` fonksiyonunu çağırır.
    *   `IsFriendByName(const char* c_szName)`: `m_FriendNameMap` içinde verilen ismin olup olmadığını kontrol eder.
*   **GM Yönetimi** (`ENABLE_MESSENGER_GM` ile):
    *   `OnGMLogin(const char* c_szName)`: İsmi `m_GMNameMap`'e ekler ve UI'daki `OnLogin`'i `MESSENGER_GROUP_INDEX_GM` ile çağırır.
    *   `OnGMLogout(const char* c_szName)`: İsmi `m_GMNameMap`'e ekler ve UI'daki `OnLogout`'u çağırır. (Yine `Login` ile benzer şekilde sete ekleme yapılıyor.)
*   **Engellenen Kullanıcı Yönetimi** (`ENABLE_MESSENGER_BLOCK` ile):
    *   `OnBlockLogin(const char* c_szName)`: İsmi `m_BlockNameMap`'e ekler. Engellenen kullanıcılar her zaman çevrimdışı görüneceği için Python'daki `OnLogout` fonksiyonunu `MESSENGER_GROUP_INDEX_BLOCK` ile çağırır.
    *   `OnBlockLogout(const char* c_szName)`: İsmi `m_BlockNameMap`'e ekler ve UI'daki `OnLogout`'u çağırır.
    *   `IsBlockFriendByName(const char* c_szName)`: Verilen ismin `m_BlockNameMap`'te olup olmadığını kontrol eder.
    *   `RemoveBlock(const char* c_szName)`: İsmi `m_BlockNameMap`'ten siler ve Python'daki `OnRemoveList` fonksiyonunu çağırır.
*   **Lonca Üyesi Yönetimi**:
    *   `AppendGuildMember(const char* c_szName)`: Eğer isim `m_GuildMemberStateMap`'te yoksa, üyeyi çevrimdışı (`LogoutGuildMember`) olarak ekler.
    *   `RemoveGuildMember(const char* c_szName)`: İsmi `m_GuildMemberStateMap`'ten siler ve UI'daki `OnRemoveList`'i `MESSENGER_GRUOP_INDEX_GUILD` ile çağırır.
    *   `RemoveAllGuildMember()`: `m_GuildMemberStateMap`'i temizler ve UI'daki `OnRemoveAllList`'i çağırır.
    *   `LoginGuildMember(const char* c_szName)`: İlgili üyenin durumunu `m_GuildMemberStateMap`'te 1 (çevrimiçi) olarak ayarlar ve UI'daki `OnLogin`'i çağırır.
    *   `LogoutGuildMember(const char* c_szName)`: İlgili üyenin durumunu `m_GuildMemberStateMap`'te 0 (çevrimdışı) olarak ayarlar ve UI'daki `OnLogout`'u çağırır.
    *   `RefreshGuildMember()`: `m_GuildMemberStateMap`'teki tüm üyeler için mevcut durumlarına göre UI'daki `OnLogin` veya `OnLogout` fonksiyonlarını çağırarak listeyi yeniler.

#### C++ ve Python Arayüzü ile İlişkisi

*   `CPythonMessenger` sınıfı ve içindeki yapılar, mesajlaşma verilerini C++ tarafında yönetir.
*   `CPythonMessenger.cpp` dosyasında, bu sınıfın bazı fonksiyonları ve veri yapıları kullanılarak `messenger` adında bir Python modülü oluşturulur. Bu modül, Python tabanlı UI betiklerinin mesajlaşma verilerine erişmesini ve bazı temel işlemleri yapmasını sağlar.
*   Enum'larda tanımlanan sabitler, Python modülüne de aktarılarak sunucu ile iletişimde ve UI'da durum gösteriminde kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Olay Yönetimi**: Sunucudan gelen arkadaşlarla, lonca üyeleriyle, GM'lerle veya engellenenlerle ilgili olayları (giriş, çıkış, ekleme, silme) işler.
*   **UI Bildirimi**: Bu olayları Python'daki UI işleyicisine (`m_poMessengerHandler`) ileterek arayüzün anlık olarak güncellenmesini sağlar. Örneğin, bir arkadaş giriş yaptığında Messenger penceresindeki ismi parlar veya lonca listesi güncellenir.
*   **Python Entegrasyonu**: `messenger` modülü aracılığıyla, Python UI betiklerinin C++ tarafındaki mesajlaşma verilerine erişmesini ve bazı işlemleri tetiklemesini sağlar.

Bu dosya, mesajlaşma sisteminin C++ mantığını ve Python ile olan etkileşimini merkezileştirir.

---

### `PythonMessenger.cpp` (Python Messenger Sistemi Implementasyonu ve `messenger` Modülü)

Bu C++ kaynak dosyası, `PythonMessenger.h` dosyasında bildirilen `CPythonMessenger` singleton sınıfının metotlarını implemente eder. Ayrıca, bu sınıfın işlevlerini Python betiklerine sunan `messenger` adlı bir Python modülünü tanımlar.

#### `CPythonMessenger` Sınıfı Implementasyon Detayları

*   **Yapıcı (`CPythonMessenger::CPythonMessenger()`)**: `m_poMessengerHandler`'ı `NULL` olarak başlatır.
*   **Yıkıcı (`CPythonMessenger::~CPythonMessenger()`)**: Özel bir temizleme işlemi yapmaz (muhtemelen `Destroy()` manuel olarak çağrılır).
*   **`Destroy()`**: Arkadaş listesini (`m_FriendNameMap`) ve lonca üyesi durum haritasını (`m_GuildMemberStateMap`) temizler.
*   **`SetMessengerHandler(PyObject* poHandler)`**: Python tarafından gönderilen UI olay işleyici nesnesini `m_poMessengerHandler` üye değişkenine atar.

*   **Arkadaş Yönetimi**:
    *   `RemoveFriend(const char* c_szName)`: Verilen ismi `m_FriendNameMap` setinden siler.
    *   `OnFriendLogin(const char* c_szName)`: Verilen ismi `m_FriendNameMap`'e ekler. Eğer `m_poMessengerHandler` ayarlıysa, Python'daki `OnLogin` fonksiyonunu `MESSENGER_GRUOP_INDEX_FRIEND` grup indeksi ve isim ile çağırır.
    *   `OnFriendLogout(const char* c_szName)`: Verilen ismi `m_FriendNameMap`'e ekler (Bu, `OnFriendLogin` ile aynı davranışı sergiliyor ve bir mantık hatası olabilir; normalde listeden silme veya durumu güncelleme beklenir). Eğer `m_poMessengerHandler` ayarlıysa, Python'daki `OnLogout` fonksiyonunu çağırır.
    *   `IsFriendByName(const char* c_szName)`: `m_FriendNameMap` içinde verilen ismin olup olmadığını kontrol eder.

*   **GM Yönetimi** (`ENABLE_MESSENGER_GM` ile):
    *   `OnGMLogin(const char* c_szName)`: İsmi `m_GMNameMap`'e ekler ve UI'daki `OnLogin`'i `MESSENGER_GROUP_INDEX_GM` ile çağırır.
    *   `OnGMLogout(const char* c_szName)`: İsmi `m_GMNameMap`'e ekler ve UI'daki `OnLogout`'u çağırır. (Yine `Login` ile benzer şekilde sete ekleme yapılıyor.)

*   **Engellenen Kullanıcı Yönetimi** (`ENABLE_MESSENGER_BLOCK` ile):
    *   `OnBlockLogin(const char* c_szName)`: İsmi `m_BlockNameMap`'e ekler. Engellenen kullanıcılar her zaman çevrimdışı görüneceği için Python'daki `OnLogout` fonksiyonunu `MESSENGER_GROUP_INDEX_BLOCK` ile çağırır.
    *   `OnBlockLogout(const char* c_szName)`: İsmi `m_BlockNameMap`'e ekler ve UI'daki `OnLogout`'u çağırır.
    *   `IsBlockFriendByName(const char* c_szName)`: Verilen ismin `m_BlockNameMap`'te olup olmadığını kontrol eder.
    *   `RemoveBlock(const char* c_szName)`: İsmi `m_BlockNameMap`'ten siler ve Python'daki `OnRemoveList` fonksiyonunu çağırır.

*   **Lonca Üyesi Yönetimi**:
    *   `AppendGuildMember(const char* c_szName)`: Eğer isim `m_GuildMemberStateMap`'te yoksa, üyeyi çevrimdışı (`LogoutGuildMember`) olarak ekler.
    *   `RemoveGuildMember(const char* c_szName)`: İsmi `m_GuildMemberStateMap`'ten siler ve UI'daki `OnRemoveList`'i `MESSENGER_GRUOP_INDEX_GUILD` ile çağırır.
    *   `RemoveAllGuildMember()`: `m_GuildMemberStateMap`'i temizler ve UI'daki `OnRemoveAllList`'i çağırır.
    *   `LoginGuildMember(const char* c_szName)`: İlgili üyenin durumunu `m_GuildMemberStateMap`'te 1 (çevrimiçi) olarak ayarlar ve UI'daki `OnLogin`'i çağırır.
    *   `LogoutGuildMember(const char* c_szName)`: İlgili üyenin durumunu `m_GuildMemberStateMap`'te 0 (çevrimdışı) olarak ayarlar ve UI'daki `OnLogout`'u çağırır.
    *   `RefreshGuildMember()`: `m_GuildMemberStateMap`'teki tüm üyeler için mevcut durumlarına göre UI'daki `OnLogin` veya `OnLogout` fonksiyonlarını çağırarak listeyi yeniler.

#### `messenger` Python Modülü

Bu bölüm, `CPythonMessenger` sınıfının işlevlerini Python'a açan C fonksiyonlarını ve modül başlatma fonksiyonunu içerir.

*   **Python'a Açılan Fonksiyonlar**:
    *   `messengerRemoveFriend(PyObject* poSelf, PyObject* poArgs)`: Python'dan alınan ismi `CPythonMessenger::Instance().RemoveFriend()` ile siler.
    *   `messengerIsFriendByName(PyObject* poSelf, PyObject* poArgs)`: Python'dan alınan ismin arkadaş listesinde olup olmadığını `CPythonMessenger::Instance().IsFriendByName()` ile kontrol eder ve sonucu Python boolean olarak döndürür.
    *   `messengerIsBlockFriendByName(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_MESSENGER_BLOCK` ile): Engellenenler listesini kontrol eder.
    *   `messengerRemoveBlock(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_MESSENGER_BLOCK` ile): Engellenenler listesinden siler.
    *   `messengerDestroy(PyObject* poSelf, PyObject* poArgs)`: `CPythonMessenger::Instance().Destroy()` çağırır.
    *   `messengerRefreshGuildMember(PyObject* poSelf, PyObject* poArgs)`: `CPythonMessenger::Instance().RefreshGuildMember()` çağırır.
    *   `messengerSetMessengerHandler(PyObject* poSelf, PyObject* poArgs)`: Python'dan gelen UI olay işleyici nesnesini `CPythonMessenger::Instance().SetMessengerHandler()` ile ayarlar.
    *   `messengerGetFriendNames(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_MAILBOX` ile): `CPythonMessenger::Instance().m_FriendNameMap` içindeki tüm arkadaş isimlerini bir Python demeti (tuple) olarak döndürür. Bu fonksiyonun `ENABLE_MAILBOX` ile koşullandırılması ilginçtir ve messenger ile posta kutusu arasında bir bağlantı veya paylaşılan bir arkadaş listesi mantığı olabileceğini düşündürür.

*   **`initMessenger()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `messenger` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, yukarıda bahsedilen C fonksiyonlarını Python isimleriyle eşleştirir.
    *   `Py_InitModule("messenger", s_methods);` ile `messenger` modülü oluşturulur ve Python'a sunulur.

#### Gözlemler ve Olası İyileştirmeler

*   `OnFriendLogin`, `OnFriendLogout`, `OnGMLogin`, `OnGMLogout`, `OnBlockLogin`, `OnBlockLogout` fonksiyonları, ilgili setlere (örn: `m_FriendNameMap`) sadece ekleme (`insert`) yapıyor. Bu, aynı ismin birden çok kez eklenmesine yol açmaz (çünkü `std::set` benzersiz elemanlar tutar), ancak çıkış (logout) durumlarında genellikle listeden silme veya durum güncelleme beklenir. Bu durum, mevcut implementasyonun eksik veya farklı bir mantıkla çalıştığını gösterebilir. Özellikle arkadaş ve GM listeleri için çıkış durumlarında sadece UI'da haber veriliyor, C++ tarafındaki veri yapısı güncellenmiyor gibi duruyor.
*   Engellenen kullanıcılar için `OnBlockLogin` fonksiyonu, kullanıcı giriş yapsa bile UI'da `OnLogout` çağırarak çevrimdışı gösteriyor. Bu, istenen bir davranıştır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Olay Yönetimi**: Sunucudan gelen arkadaşlarla, lonca üyeleriyle, GM'lerle veya engellenenlerle ilgili olayları (giriş, çıkış, ekleme, silme) işler.
*   **UI Bildirimi**: Bu olayları Python'daki UI işleyicisine (`m_poMessengerHandler`) ileterek arayüzün anlık olarak güncellenmesini sağlar. Örneğin, bir arkadaş giriş yaptığında Messenger penceresindeki ismi parlar veya lonca listesi güncellenir.
*   **Python Entegrasyonu**: `messenger` modülü aracılığıyla, Python UI betiklerinin C++ tarafındaki mesajlaşma verilerine erişmesini ve bazı işlemleri tetiklemesini sağlar.

Bu dosya, mesajlaşma sisteminin C++ mantığını ve Python ile olan etkileşimini merkezileştirir.

---

### `PythonMiniMap.h` (Python Mini Harita Sistemi Başlık Dosyası)

Bu başlık dosyası, oyun içi mini harita ve atlas (büyük harita) sisteminin istemci tarafındaki temel sınıfı olan `CPythonMiniMap`'i tanımlar. `CScreen`'den ve `CSingleton<CPythonMiniMap>`'den miras alır, bu da onun bir ekran öğesi olarak render edilebildiğini ve oyun içinde tek bir örneğinin olduğunu gösterir.

#### Dosyanın Genel Amaçları

*   **`CPythonMiniMap` Sınıf Bildirimi**:
    *   Mini harita ve atlasın oluşturulması, güncellenmesi, render edilmesi ve yok edilmesi için temel fonksiyonları içerir.
    *   Mini harita ölçeği, merkez pozisyonu, boyutu gibi ayarların yönetilmesini sağlar.
    *   Oyuncuları, NPC'leri, canavarları, warp portallarını ve diğer özel işaretleri mini harita üzerinde göstermek için veri yapıları ve fonksiyonlar barındırır.
    *   Atlas (büyük harita) görünümünü, üzerindeki işaretleri ve lonca bölgelerini yönetir.
    *   Yol noktaları (waypoint) ve sinyal noktaları ekleme/kaldırma işlevselliği sunar.
    *   Python UI ile etkileşim için arayüzler sağlar (örneğin, atlas penceresini kaydetmek).
*   **Enum Tanımları**:
    *   `EMPIRE_NUM`: İmparatorluk sayısını tanımlar.
    *   `MINI_WAYPOINT_IMAGE_COUNT`, `WAYPOINT_IMAGE_COUNT`, `TARGET_MARK_IMAGE_COUNT`: Farklı türdeki harita işaretleri için kullanılacak animasyonlu görsel sayısını belirtir.
    *   `TYPE_OPC`, `TYPE_NPC`, `TYPE_MONSTER`, `TYPE_WARP`, `TYPE_WAYPOINT`, `TYPE_PARTY`, `TYPE_EMPIRE`, `TYPE_TARGET`, `TYPE_PET` (`ENABLE_GROWTH_PET_SYSTEM` ile): Harita üzerinde gösterilebilecek farklı varlık türlerini tanımlayan enum.
*   **Veri Yapıları**:
    *   `TAtlasMarkInfo`: Atlas üzerindeki bir işaretin (NPC, warp, waypoint) türünü, ID'sini, dünya ve ekran koordinatlarını, metnini ve ilişkili karakter ID'sini (varsa) tutar.
    *   `TGuildAreaInfo`: Lonca bölgelerinin ID'sini, dünya koordinatlarını, boyutlarını ve render için ekran koordinatlarını içerir.
    *   `SObserver`: Mini harita üzerinde hareket eden gözlemcilerin (genellikle parti üyeleri) kaynak, hedef ve mevcut pozisyonlarını, hareket zamanlamalarını tutar.
    *   `TMarkPosition`: Mini harita üzerindeki bir varlığın (PC, NPC, canavar) ekran koordinatlarını ve isim rengini içerir.
    *   `SPartyPlayerPosition` (`WJ_SHOW_PARTY_ON_MINIMAP` ile): Atlas üzerinde parti üyelerinin pozisyonunu, ismini, dönüşünü ve özel işaret görselini tutar.
*   **Tür Tanımları (`typedef`)**:
    *   `TInstanceMarkPositionVector`: `TMarkPosition` yapılarından oluşan bir vektör.
    *   `TAtlasMarkInfoVector`: `TAtlasMarkInfo` yapılarından oluşan bir vektör.
    *   `TGuildAreaInfoVector`: `TGuildAreaInfo` yapılarından oluşan bir vektör.

#### C++ ve Python Arayüzü ile İlişkisi

*   `CPythonMiniMap` sınıfı, mini harita ve atlasın tüm C++ mantığını yönetir.
*   Python tarafına doğrudan bir modül sunmasa da, `CPythonMiniMap.cpp` içerisinde bu sınıfın bazı fonksiyonlarını çağıran ve Python UI betiklerinin mini harita/atlas ile etkileşime girmesini sağlayan `miniMap` (veya benzer bir isimde) bir modül tanımlanır. Bu etkileşimler genellikle harita üzerinde tıklama bilgilerini almak, yol noktaları eklemek veya atlas penceresini kontrol etmek gibi işlevleri içerir.
*   `RegisterAtlasWindow`, `OpenAtlasWindow`, `SetAtlasCenterPosition` gibi fonksiyonlar Python'dan gelen UI olay işleyici nesnesiyle (`m_poHandler`) etkileşim kurar.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Navigasyon Yardımı**: Oyunculara mevcut konumlarını, çevrelerindeki önemli varlıkları (NPC'ler, portallar, diğer oyuncular) ve hedefleri göstererek oyun dünyasında gezinmelerine yardımcı olur.
*   **Görev Takibi ve Yol Noktaları**: Görev hedeflerini veya oyuncunun belirlediği özel noktaları harita üzerinde işaretleyerek kolayca bulunmalarını sağlar.
*   **Stratejik Bilgi**: Lonca bölgeleri gibi stratejik öneme sahip alanları atlas üzerinde gösterir.
*   **Parti Üyelerini Takip**: Parti üyelerinin konumlarını mini harita veya atlas üzerinde göstererek koordinasyonu kolaylaştırır.
*   **Ölçeklenebilir Görünüm**: Mini haritanın yakınlaştırma/uzaklaştırma (`ScaleUp`, `ScaleDown`) özelliği ile oyuncu farklı detay seviyelerinde etrafını görebilir.
*   **Atlas Kullanımı**: Daha geniş bir bölgeyi görmek, uzak NPC'leri veya warp portallarını bulmak için atlas (tam ekran harita) görünümü sunar.
*   **Performans**: Mini harita dokularını (`CTerrain::GetMiniMapTexture()`) ve diğer grafikleri verimli bir şekilde render etmeye çalışır. `MINIMAP_SMOOTH_ZOOM` gibi makrolar kullanıcı deneyimini iyileştirmeye yönelik özellikler ekler.

Bu başlık dosyası, Metin2 istemcisinin mini harita ve atlas işlevselliğinin kalbi niteliğindedir. Oyun içi yön bulma ve çevresel farkındalık için kritik öneme sahiptir.

---

### `PythonMiniMap.cpp` (Python Mini Harita Sistemi Implementasyonu ve `miniMap` Modülü)

Bu C++ kaynak dosyası, `PythonMiniMap.h` dosyasında bildirilen `CPythonMiniMap` singleton sınıfının metotlarını implemente eder. Aynı zamanda, bu sınıfın işlevlerini Python betiklerine sunan `miniMap` (veya benzer bir isimde, kaynak kodda spesifik bir modül ismi açıkça belirtilmemişse de genellikle bu şekilde adlandırılır) adlı bir Python modülünü tanımlar. Dosya, mini haritanın ve atlasın (büyük harita) oluşturulması, güncellenmesi, çizdirilmesi (render edilmesi), kullanıcı etkileşimlerinin yönetilmesi ve çeşitli harita işaretlerinin (oyuncular, NPC'ler, canavarlar, portallar, yol noktaları vb.) işlenmesiyle ilgili tüm mantığı içerir.

#### `CPythonMiniMap` Sınıfı Implementasyon Detayları

*   **Temel İşlevler**:
    *   `Create()`: Mini harita ve atlas için gerekli olan grafik kaynaklarını (resimler, işaretleyiciler, vertex/index buffer'lar) yükler ve başlangıç ayarlarını yapar. Örneğin, `D:/ymir work/ui/` altındaki `minimap_image_filter.dds`, `minimap_camera.dds`, `playermark.sub`, `whitemark.sub` gibi görselleri ve yol noktası görsellerini (`mini_waypointXX.sub`, `waypointXX.sub`, `targetmarkXX.sub`) yükler.
    *   `Destroy()`: `CPythonMiniMap` örneği yok edilirken tüm kaynakları serbest bırakır.
    *   `Update(float fCenterX, float fCenterY)`: Ana karakterin pozisyonuna göre mini haritadaki diğer varlıkların (oyuncular, NPC'ler, canavarlar, portallar, parti üyeleri) pozisyonlarını günceller. Bu fonksiyon, varlıkların mini harita sınırları içinde olup olmadığını kontrol eder ve uygun şekilde `m_OtherPCPositionVector`, `m_PartyPCPositionVector`, `m_NPCPositionVector`, `m_MonsterPositionVector`, `m_WarpPositionVector` gibi vektörlere ekler. Gözlemcilerin (observer) pozisyonlarını da günceller. Ayrıca, atlas üzerindeki hedef (target) işaretlerinin pozisyonlarını da günceller.
    *   `Render(float fScreenX, float fScreenY)`: Mini haritanın kendisini (arazi dokuları) ve üzerindeki tüm işaretleri (oyuncu, NPC, canavar, portal, hedef işaretleri, kamera açısı göstergesi) ekrana çizer. DirectX state'lerini (TextureStageState, RenderState) ayarlayarak farklı çizim efektleri uygular.
    *   `SetCenterPosition(float fCenterX, float fCenterY)`: Mini haritanın merkezini ana karakterin dünya koordinatlarına göre ayarlar ve ilgili arazi dokularını yükler.
    *   `SetScale(float fScale)`: Mini haritanın yakınlaştırma seviyesini ayarlar. `ScaleUp()` ve `ScaleDown()` bu fonksiyonu çağırarak ölçeği değiştirir. `MINIMAP_SMOOTH_ZOOM` makrosu aktifse, yumuşak bir geçişle ölçek değişimi yapılır.
    *   `SetMiniMapSize(float fWidth, float fHeight)`: Mini haritanın ekran üzerindeki genişlik ve yüksekliğini ayarlar.

*   **Atlas (Büyük Harita) Yönetimi**:
    *   `LoadAtlas()`: Mevcut haritanın atlasını (`atlas.sub`) ve atlas üzerindeki işaret bilgilerini (`harita_adı_point.txt`) yükler.
    *   `UpdateAtlas()`: Atlas üzerindeki ana oyuncu işaretinin pozisyonunu ve lonca bölgesi bilgilerini günceller.
    *   `RenderAtlas(float fScreenX, float fScreenY)`: Atlası ve üzerindeki NPC, warp, yol noktası, oyuncu ve lonca bölgesi işaretlerini ekrana çizer.
    *   `ClearAtlasMarkInfo()`: Atlas üzerindeki NPC ve warp işaret bilgilerini temizler.
    *   `RegisterAtlasMark(BYTE byType, const char* c_szName, long lx, long ly)`: Harita için yüklenen `_point.txt` dosyasındaki NPC ve warp bilgilerini kaydeder. `ENABLE_ATLAS_MARK_INFO` makrosu aktifse, işaret bilgileri doğrudan NPC vnum'larına göre `CPythonNonPlayer`'dan alınır.
    *   `__LoadAtlasMarkInfo()`: Mevcut haritaya ait `_point.txt` dosyasını okuyarak NPC ve warp noktalarını `m_AtlasNPCInfoVector` ve `m_AtlasWarpInfoVector` içerisine yükler.
    *   `GetAtlasInfo(...)`: Atlas üzerinde belirli bir ekran koordinatına tıklanıldığında, o noktadaki varlık (oyuncu, NPC, warp, yol noktası, lonca bölgesi) hakkında bilgi döndürür.
    *   `GetAtlasSize(...)`: Yüklenmiş atlasın boyutlarını döndürür.

*   **İşaret ve Varlık Yönetimi**:
    *   `AddObserver(DWORD dwVID, float fSrcX, float fSrcY)`, `MoveObserver(DWORD dwVID, float fDstX, float fDstY)`, `RemoveObserver(DWORD dwVID)`: Mini harita üzerinde hareket eden gözlemcileri (genellikle parti üyeleri için kullanılır) yönetir. Bu gözlemcilerin pozisyonları `Update` fonksiyonunda enterpolasyonla güncellenir.
    *   `AddWayPoint(BYTE byType, DWORD dwID, float fX, float fY, std::string strText, DWORD dwChrVID = 0)`, `RemoveWayPoint(DWORD dwID)`: Harita üzerinde yol noktaları (örneğin görev hedefleri) ekler veya kaldırır.
    *   `CreateTarget(int iID, const char* c_szName)`, `CreateTarget(int iID, const char* c_szName, DWORD dwVID)`, `UpdateTarget(int iID, int ix, int iy)`, `DeleteTarget(int iID)`: Özel hedef (target) işaretleri oluşturur, günceller ve siler. Bu işaretler mini harita ve atlas üzerinde görünebilir.
    *   `AddSignalPoint(float fX, float fY)`, `ClearAllSignalPoint()`: Oyuncunun tıkladığı noktalara geçici sinyal işaretleri (genellikle bir ping efekti) ekler ve temizler.
    *   `GetPickedInstanceInfo(...)`: Mini harita üzerinde belirli bir ekran koordinatına tıklanıldığında, o noktadaki karakter (oyuncu, NPC, canavar) hakkında bilgi döndürür.

*   **Lonca Bölgesi Yönetimi**:
    *   `ClearGuildArea()`: Lonca bölgesi bilgilerini temizler.
    *   `RegisterGuildArea(DWORD dwID, DWORD dwGuildID, long x, long y, long width, long height)`: Sunucudan gelen lonca bölgesi bilgilerini kaydeder.
    *   `GetGuildAreaID(DWORD x, DWORD y)`: Verilen dünya koordinatının hangi lonca bölgesine ait olduğunu döndürür.

*   **Python Etkileşimleri**:
    *   `RegisterAtlasWindow(PyObject* poHandler)`, `UnregisterAtlasWindow()`: Python'dan gelen atlas penceresi UI nesnesini kaydeder/kaldırır.
    *   `OpenAtlasWindow()`: Kayıtlı Python atlas penceresini gösterir.
    *   `SetAtlasCenterPosition(int x, int y)`: Atlas penceresinin merkezini ayarlar.

*   **Koşullu Derleme Makroları**:
    *   `ENABLE_ATLAS_MARK_INFO`: Atlas üzerindeki NPC/Warp bilgilerinin nasıl yükleneceğini değiştirir.
    *   `WJ_SHOW_PARTY_ON_MINIMAP`: Parti üyelerinin atlas üzerinde özel işaretlerle gösterilmesini sağlar. `AddPartyPositionInfo` ve `RemovePartyPositionInfo` fonksiyonları bu özellik için eklenir.
    *   `ENABLE_FOV_OPTION`: Mini haritada kamera görüş açısını (Field of View) gösteren bir efekt ekler.
    *   `MINIMAP_SMOOTH_ZOOM`: Mini harita yakınlaştırma/uzaklaştırma işlemlerine yumuşak bir animasyon ekler.
    *   `ENABLE_GROWTH_PET_SYSTEM`: Büyüyen evcil hayvanların mini haritada gösterilmesi için destek ekler.
    *   `ENABLE_MAP_TELEPORT`: Atlas üzerinde tıklanan noktanın koordinatlarını `GetAtlasInfo` ile döndürmeye yarayabilir.

#### Python Modülü (`miniMap`) Fonksiyonları (Tahmini)

`PythonMiniMap.cpp` dosyasında `miniMap` adlı bir Python modülü tanımlanır ve bu modül aracılığıyla Python betikleri `CPythonMiniMap` sınıfının belirli işlevlerine erişebilir. Tipik olarak sunulan fonksiyonlar şunları içerebilir (C++ fonksiyon adları ve Python'a export edilen adlar farklılık gösterebilir):

*   `IsAtlas()`: Atlasın yüklenip yüklenmediğini kontrol eder.
*   `Show()` / `Hide()`: Mini haritayı gösterir/gizler.
*   `ShowAtlas()` / `HideAtlas()`: Atlası gösterir/gizler.
*   `ScaleUp()` / `ScaleDown()`: Mini harita ölçeğini büyütür/küçültür.
*   `GetPickedInstanceInfo(float fScreenX, float fScreenY)`: Mini haritada tıklanan yerdeki karakter bilgisini alır.
*   `GetAtlasInfo(float fScreenX, float fScreenY)`: Atlasta tıklanan yerdeki bilgi (NPC adı, koordinat, lonca ID'si vb.) alır.
*   `GetAtlasSize()`: Atlasın boyutlarını alır.
*   `AddWayPoint(int type, int id, float x, float y, string name)`: Haritaya yol noktası ekler.
*   `RemoveWayPoint(int id)`: Yol noktasını kaldırır.
*   `CreateTarget(int id, string name)` / `CreateTargetVID(int id, string name, int vid)`: Özel hedef oluşturur.
*   `UpdateTarget(int id, int x, int y)`: Hedef pozisyonunu günceller.
*   `DeleteTarget(int id)`: Hedefi siler.
*   `AddSignalPoint(float x, float y)`: Haritaya sinyal noktası ekler (ping).
*   `ClearAllSignalPoint()`: Tüm sinyal noktalarını temizler.
*   `RegisterAtlasWindow(object window)`: Atlas UI penceresini C++ tarafına kaydeder.
*   `SetAtlasCenterPosition(int x, int y)`: Atlas penceresinin merkezini kaydırır.
*   `GetGuildAreaID(float x, float y)`: Verilen koordinattaki lonca ID'sini alır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Ana Mini Harita**: Oyun ekranının bir köşesinde sürekli görünen, oyuncunun yakın çevresini, NPC'leri, canavarları ve portalları gösteren harita.
*   **Atlas (Büyük Harita)**: Genellikle bir tuşa basarak (örn: 'M') açılan tam ekran harita. Daha geniş bir alanı, tüm NPC ve warp noktalarını, görev hedeflerini ve lonca bölgelerini gösterir.
*   **Tıklama ile Bilgi Alma**: Mini harita veya atlas üzerinde bir noktaya tıklandığında o noktadaki varlık hakkında bilgi (isim, koordinat vb.) gösterilir.
*   **Yol Bulma**: Görev hedefleri veya oyuncu tarafından işaretlenen noktalar (waypoint) harita üzerinde gösterilerek oyuncunun hedefine ulaşması kolaylaştırılır.
*   **Parti Yönetimi**: Parti üyelerinin konumları harita üzerinde gösterilerek takım oyunu desteklenir (`WJ_SHOW_PARTY_ON_MINIMAP`).
*   **Lonca Savaşları**: Loncaların sahip olduğu bölgeler atlas üzerinde işaretlenerek stratejik bir görünüm sunulur.

Bu dosya, istemcinin harita ile ilgili tüm görsel ve mantıksal işlemlerini yürüten merkezi bir bileşendir. Performans, özellikle çok sayıda varlığın olduğu durumlarda kritik öneme sahiptir. Çeşitli koşullu derleme seçenekleri, farklı sunucu yapılandırmalarına veya oyun özelliklerine uyum sağlamak için esneklik sunar.

---

### `PythonMiniMapModule.cpp` (Python Mini Harita Modülü)

Bu C++ kaynak dosyası, `CPythonMiniMap` C++ sınıfının işlevlerini Python betiklerine sunan `miniMap` adlı bir Python modülünü tanımlar. Bu modül, Python tabanlı UI (Kullanıcı Arayüzü) betiklerinin mini harita ve atlas (büyük harita) üzerinde çeşitli işlemleri gerçekleştirmesini sağlar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Tanımlama**: `CPythonMiniMap` sınıfının public metotlarını Python fonksiyonları olarak sarmalayarak `miniMap` adında bir Python modülü oluşturur.
*   **Arayüz Sağlama**: Python UI kodunun mini haritanın ölçeğini değiştirmesine, merkez pozisyonunu ayarlamasına, boyutunu belirlemesine, güncellemesine, render etmesine, göstermesine/gizlemesine olanak tanır.
*   **Atlas İşlevleri**: Atlasın yüklenmesi, güncellenmesi, render edilmesi, gösterilmesi/gizlenmesi ve atlas üzerindeki bilgilere (tıklanan yerdeki varlık, atlas boyutu vb.) erişim gibi atlas ile ilgili işlevleri Python'a açar.
*   **İşaret Yönetimi**: Yol noktası (waypoint) ekleme ve kaldırma gibi işlemleri Python üzerinden yapılabilir hale getirir.
*   **UI Etkileşimi**: Atlas UI penceresinin C++ tarafına kaydedilmesi ve lonca bölgesi ID'lerinin sorgulanması gibi UI ile doğrudan etkileşimli fonksiyonlar sunar.
*   **Sabitlerin Aktarılması**: `CPythonMiniMap` sınıfında tanımlanan harita varlık türü sabitlerini (örneğin, `TYPE_NPC`, `TYPE_MONSTER`) Python modülüne aktarır, böylece Python tarafında bu sabitler kullanılabilir.

#### Python'a Aktarılan Fonksiyonlar (`miniMap` Modülü)

Aşağıda, `miniMap` modülü aracılığıyla Python'a sunulan temel fonksiyonlar ve kısa açıklamaları bulunmaktadır:

*   **Mini Harita Temel Kontrolleri**:
    *   `miniMap.SetScale(float scale)`: Mini haritanın yakınlaştırma oranını ayarlar.
    *   `miniMap.ScaleUp()`: Mini haritayı bir kademe yakınlaştırır.
    *   `miniMap.ScaleDown()`: Mini haritayı bir kademe uzaklaştırır.
    *   `miniMap.SetMiniMapSize(float width, float height)`: Mini haritanın ekran üzerindeki genişlik ve yüksekliğini ayarlar.
    *   `miniMap.SetCenterPosition(float centerX, float centerY)`: Mini haritanın merkezini belirtilen dünya koordinatlarına ayarlar.
    *   `miniMap.Destroy()`: Mini harita kaynaklarını serbest bırakır.
    *   `miniMap.Create()`: Mini harita kaynaklarını oluşturur ve yükler.
    *   `miniMap.Update(float centerX, float centerY)`: Mini haritayı günceller (genellikle ana karakterin pozisyonuyla).
    *   `miniMap.Render(float screenX, float screenY)`: Mini haritayı belirtilen ekran koordinatlarına çizer.
    *   `miniMap.Show()`: Mini haritayı görünür yapar.
    *   `miniMap.Hide()`: Mini haritayı gizler.
    *   `miniMap.isShow()`: Mini haritanın görünür olup olmadığını boolean olarak döndürür.
    *   `miniMap.GetInfo(float screenX, float screenY)`: Mini harita üzerinde belirtilen ekran koordinatına tıklandığında oradaki varlık hakkında bilgi (bulunup bulunmadığı, isim, dünya koordinatları (metre cinsinden), isim rengi) döndürür.

*   **Atlas (Büyük Harita) Kontrolleri**:
    *   `miniMap.LoadAtlas()`: Mevcut haritanın atlasını yükler.
    *   `miniMap.UpdateAtlas()`: Atlası günceller.
    *   `miniMap.RenderAtlas(float screenX, float screenY)`: Atlası belirtilen ekran koordinatlarına çizer.
    *   `miniMap.ShowAtlas()`: Atlası görünür yapar.
    *   `miniMap.HideAtlas()`: Atlası gizler.
    *   `miniMap.isShowAtlas()`: Atlasın görünür olup olmadığını boolean olarak döndürür.
    *   `miniMap.IsAtlas()`: Mevcut harita için bir atlasın yüklenmiş olup olmadığını boolean olarak döndürür.
    *   `miniMap.GetAtlasInfo(float screenX, float screenY)`: Atlas üzerinde belirtilen ekran koordinatına tıklandığında oradaki varlık hakkında bilgi (bulunup bulunmadığı, isim, dünya koordinatları (metre cinsinden), isim rengi, lonca ID'si) döndürür.
    *   `miniMap.GetAtlasSize()`: Yüklenmiş atlasın piksel cinsinden genişlik ve yüksekliğini döndürür.

*   **Yol Noktası (Waypoint) Yönetimi**:
    *   `miniMap.AddWayPoint(int id, float x, float y, string name)`: Belirtilen ID, dünya koordinatları ve isim ile bir yol noktası ekler. (Tür otomatik olarak `TYPE_WAYPOINT` atanır).
    *   `miniMap.RemoveWayPoint(int id)`: Belirtilen ID'ye sahip yol noktasını kaldırır.

*   **UI ve Diğer Etkileşimler**:
    *   `miniMap.RegisterAtlasWindow(object window)`: Python'da oluşturulmuş atlas UI penceresi nesnesini C++ tarafındaki `CPythonMiniMap`'e kaydeder.
    *   `miniMap.UnregisterAtlasWindow()`: Kayıtlı atlas UI penceresini C++ tarafından kaldırır.
    *   `miniMap.GetGuildAreaID(float x, float y)`: Belirtilen dünya koordinatının ait olduğu lonca bölgesinin ID'sini döndürür. Eğer bir lonca bölgesine ait değilse genellikle `0xFFFFFFFF` gibi bir değer döndürür.

*   **Python Modülüne Aktarılan Sabitler**:
    *   `miniMap.TYPE_OPC`
    *   `miniMap.TYPE_OPCPVP`
    *   `miniMap.TYPE_OPCPVPSELF`
    *   `miniMap.TYPE_NPC`
    *   `miniMap.TYPE_MONSTER`
    *   `miniMap.TYPE_WARP`
    *   `miniMap.TYPE_WAYPOINT`
    *   `miniMap.TYPE_PARTY`
    *   `miniMap.TYPE_EMPIRE`

#### `initMiniMap()` Fonksiyonu

Bu fonksiyon, Python yorumlayıcısı tarafından `miniMap` modülü ilk kez `import` edildiğinde çağrılır. Görevleri:
1.  `PyMethodDef s_methods[]` dizisi aracılığıyla yukarıda listelenen C fonksiyonlarını Python fonksiyon isimleriyle eşleştirir.
2.  `Py_InitModule("miniMap", s_methods)` çağrısıyla `miniMap` modülünü Python çalışma zamanına tanıtır.
3.  `PyModule_AddIntConstant` kullanarak `CPythonMiniMap` sınıfındaki `TYPE_*` enum sabitlerini modüle ekler, böylece Python betikleri bu sabitlere `miniMap.TYPE_NPC` gibi erişebilir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu modül, oyunun Python tabanlı UI katmanının (örneğin, mini harita penceresi, atlas penceresi, görev takip arayüzü) C++ tarafındaki mini harita ve atlas sistemiyle etkileşim kurmasını sağlar. Örneğin:
*   Mini harita penceresindeki yakınlaştırma/uzaklaştırma butonları `miniMap.ScaleUp()` ve `miniMap.ScaleDown()` fonksiyonlarını çağırır.
*   Atlas penceresi açıldığında `miniMap.ShowAtlas()` çağrılır.
*   Oyuncu bir görevi takip etmeye başladığında, görev hedefi `miniMap.AddWayPoint()` ile haritaya eklenebilir.
*   Mini harita veya atlas üzerinde fare ile bir yere tıklandığında, UI betikleri `miniMap.GetInfo()` veya `miniMap.GetAtlasInfo()` ile o noktadaki bilgiyi alıp kullanıcıya gösterebilir.

`PythonMiniMapModule.cpp`, C++ mini harita altyapısı ile Python UI arasındaki köprüyü kurarak oyunun harita özelliklerinin kullanıcı arayüzü üzerinden kontrol edilmesini mümkün kılar.

---

</rewritten_file> 