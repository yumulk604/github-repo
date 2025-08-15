# UserInterface Referans Kılavuzu - Bölüm 1

Bu belge, Metin2 istemcisinin `UserInterface` (Kullanıcı Arayüzü) kütüphanesindeki C++ sınıfları, Python modülleri, UI betik dosyaları ve ilgili diğer bileşenlerin ayrıntılı bir referansını sunar. Her bileşen, genel amacını, başlık dosyasındaki (.h) önemli tanımları, C++ dosyasındaki (.cpp) temel implementasyon prensiplerini, Python arayüzlerini ve oyun içindeki pratik kullanım senaryolarını açıklayacak şekilde belgelenmiştir.

`UserInterface` modülü, oyunun pencereleri, envanter, karakter ekranı, sohbet, görevler, mağaza gibi tüm görsel ve etkileşimli elemanlarından sorumludur. Aynı zamanda Python betikleri aracılığıyla C++ koduyla sıkı bir entegrasyon içindedir.

## İçindekiler

* [Alt Klasörler](#alt-klasörler)
    * [Icons/](#icons)
    * [Cursors/](#cursors)
* [Temel Soyut Arayüzler ve Singleton](#temel-soyut-arayüzler-ve-singleton)
    * [`AbstractSingleton.h`](#abstractsingletonh)
    * [`AbstractApplication.h`](#abstractapplicationh)
    * [`AbstractCharacterManager.h`](#abstractcharactermanagerh)
    * [`AbstractChat.h`](#abstractchath)
    * [`AbstractPlayer.h`](#abstractplayerh)
* [`AccountConnector.h` ve `AccountConnector.cpp`](#accountconnectorh-ve-accountconnectorcpp)
* [`AffectFlagContainer.h` ve `AffectFlagContainer.cpp`](#affectflagcontainerh-ve-affectflagcontainercpp)
* [`CameraProcedure.cpp` (CCamera Sınıfının Parçası)](#cameraprocedurecpp-ccamera-sınıfının-parçası)
* [`CefClientApp.h` ve `CefClientApp.cpp`](#cefclientapph-ve-cefclientappcpp)
* [`CefClientHandler.h` ve `CefClientHandler.cpp`](#cefclienthandlerh-ve-cefclienthandlercpp)

*(Bu bölümdeki içerikler tamamlanmıştır. Devamı için [UserInterface Referans Kılavuzu - Bölüm 2](./client_UserInterface_Referans_Part2.md), [UserInterface Referans Kılavuzu - Bölüm 3](./client_UserInterface_Referans_Part3.md), [UserInterface Referans Kılavuzu - Bölüm 4](./client_UserInterface_Referans_Part4.md), [UserInterface Referans Kılavuzu - Bölüm 5](./client_UserInterface_Referans_Part5.md) ve [UserInterface Referans Kılavuzu - Bölüm 6](./client_UserInterface_Referans_Part6.md) dosyalarına bakınız.)*

---

## Alt Klasörler

`UserInterface` ana klasörü altında, belirli UI varlık türlerini organize etmek için kullanılan bazı alt klasörler bulunur.

### Icons/

Bu klasör, Metin2 istemci uygulamasının kendisi için kullanılan ikon dosyalarını (örneğin, masaüstü kısayolu, pencere başlığı için `.ico` formatında) barındırır. Oyun içindeki diğer tüm ikonlar (eşyalar, yetenekler, buff/debuff vb.) genellikle veri paketlerinde (`.epk/.eix`) veya diğer kaynak klasörlerinde bulunur.

### Cursors/

Bu klasör, oyun içinde kullanılan fare imleçlerinin (mouse cursor) grafiklerini içerir. Bu imleçler, oyunun farklı durumlarına (örneğin, normal işaretçi, bir düşmanı veya NPC'yi hedefleme, tıklanabilir bir arayüz elemanı üzerinde olma) göre değişiklik gösterebilir. İmleç grafikleri genellikle `.tga` veya `.dds` gibi alfa saydamlığını destekleyen formatlardadır.

---

## Temel Soyut Arayüzler ve Singleton

`UserInterface` modülü, oyunun diğer temel sistemleriyle etkileşim kurmak için bir dizi soyut arayüz (interface) kullanır. Bu arayüzler, genellikle `TAbstractSingleton` şablonundan türeyerek singleton deseniyle erişilebilir hale getirilir. Bu yaklaşım, UI kodunun, temel sistemlerin somut implementasyonlarından bağımsız kalmasını sağlar ve modülerliği artırır.

### `AbstractSingleton.h`

#### Dosyanın Genel Amacı

Bu başlık dosyası, `TAbstractSingleton<T>` adında bir C++ şablon sınıfı tanımlar. Bu şablon, projedeki çeşitli yönetici sınıfları veya global olarak erişilmesi gereken servisler için temel bir singleton (tek örnek) deseni implementasyonu sunar. Singleton deseni, bir sınıftan yalnızca bir tek örnek oluşturulabilmesini ve bu örneğe global bir erişim noktası sağlanmasını garanti eder.

#### Şablon Sınıfı Tanımları

*   **`template <typename T> class TAbstractSingleton`**:
    *   **`static T* ms_singleton`**: Her bir `T` uzmanlaşması için o tipe ait tekil örneğin işaretçisini tutan statik bir üye. Program başlangıcında `0` (NULL) olarak ilklendirilir.
    *   **Yapıcı `TAbstractSingleton()`**: Bir `TAbstractSingleton` nesnesi oluşturulduğunda çağrılır.
        *   `assert(!ms_singleton)`: `ms_singleton`'un daha önce başka bir örnek tarafından ayarlanmadığını kontrol eder (bir tipten sadece bir singleton olabilir).
        *   `int offset = (int)(T*)1 - (int)(CSingleton <T>*) (T*) 1;`: Bu satır, `T` tipinin ve `CSingleton<T>` (muhtemelen başka bir temel singleton sınıfı) tipinin bellek düzeni arasındaki bir ofseti hesaplar. Bu, `T`'nin `CSingleton<T>`'den türediği ve `TAbstractSingleton`'un bu iki farklı singleton implementasyonu arasında bir köprü veya uyumluluk katmanı görevi gördüğü durumlarda, doğru `this` işaretçisini elde etmek için bir yöntemdir.
        *   `ms_singleton = (T*)((int)this + offset);`: `ms_singleton` işaretçisini, mevcut nesnenin (`this`) hesaplanan ofset ile ayarlanmış adresine atar.
    *   **Yıkıcı `virtual ~TAbstractSingleton()`**: Singleton nesnesi yok edildiğinde çağrılır.
        *   `assert(ms_singleton)`: `ms_singleton`'un hala geçerli bir örneği işaret ettiğini kontrol eder.
        *   `ms_singleton = 0;`: İşaretçiyi sıfırlayarak singleton örneğinin artık mevcut olmadığını belirtir.
    *   **`__forceinline static T& GetSingleton()`**: Singleton örneğine bir referans döndüren statik bir metot.
        *   `assert(ms_singleton != NULL)`: Örnek alınmadan önce `ms_singleton`'un geçerli bir adresi işaret ettiğinden emin olur.
        *   `return (*ms_singleton);`: Saklanan singleton örneğine referans döndürür.

#### Kullanım Amacı ve Senaryoları

Bu şablon, `UserInterface` içindeki ve potansiyel olarak istemcinin diğer modüllerindeki çeşitli "Abstract" arayüz sınıfları (`IAbstractApplication`, `IAbstractPlayer` vb.) için temel oluşturur. Bu arayüzlerin somut implementasyonları (genellikle `UserInterface.cpp` veya ilgili Python modüllerini sarmalayan C++ sınıflarında bulunur) bu şablondan türeyerek kendilerini singleton olarak kaydederler. Böylece, istemcinin herhangi bir yerinden `IAbstractPlayer::GetSingleton().SomePlayerFunction()` gibi bir çağrı ile oyuncu ile ilgili işlemlere erişilebilir.

Ofset hesaplaması içeren yapıcı, projenin evrimi sırasında farklı singleton implementasyonları arasında uyumluluk sağlamak veya belirli derleyici/çoklu kalıtım senaryolarını ele almak için eklenmiş özel bir durumdur.

### `AbstractApplication.h`

#### Dosyanın Genel Amacı

Bu başlık dosyası, `IAbstractApplication` adında soyut bir arayüz sınıfı tanımlar. Bu arayüz, `UserInterface` modülünün ve potansiyel olarak diğer istemci modüllerinin, temel uygulama seviyesindeki işlevlere ve bilgilere (örneğin, fare konumu, zaman yönetimi, kamera kontrolü, IME olayları) erişebilmesi için standart bir yol sunar. `TAbstractSingleton<IAbstractApplication>`'dan türetilmiştir, bu da ona global singleton erişimi sağlar.

#### Arayüz Tanımları

*   **`struct SCameraPos`**: Basit bir yapı.
    *   `float m_fUpDir`, `m_fViewDir`, `m_fCrossDir`: Kameranın yukarı, bakış ve yan yön vektörlerinin büyüklüklerini veya bileşenlerini tutar.
*   **`struct SCameraSetting`**: Kamera durumunu tanımlayan bir yapı.
    *   `D3DXVECTOR3 v3CenterPosition`: Kameranın baktığı veya etrafında döndüğü merkez nokta.
    *   `SCameraPos kCmrPos`: Yukarıdaki `SCameraPos` yapısı.
    *   `float fRotation`: Kameranın Y ekseni etrafındaki dönüşü (yaw).
    *   `float fPitch`: Kameranın X ekseni etrafındaki eğimi (pitch).
    *   `float fZoom`: Kameranın yakınlaştırma seviyesi.
*   **Saf Sanal Fonksiyonlar (Pure Virtual Functions)**: Bu arayüzü implemente edecek somut bir sınıfın sağlaması gereken fonksiyonlardır.
    *   **Girdi ve Zaman**:
        *   `virtual void GetMousePosition(POINT* ppt) = 0;`: Fare imlecinin mevcut ekran koordinatlarını alır.
        *   `virtual float GetGlobalTime() = 0;`: Oyunun başlangıcından itibaren geçen toplam süreyi (saniye cinsinden) alır.
        *   `virtual float GetGlobalElapsedTime() = 0;`: Bir önceki frame'den bu yana geçen süreyi (saniye cinsinden) alır.
    *   **Render ve Sunucu Zamanı**:
        *   `virtual void SkipRenderBuffering(DWORD dwSleepMSec) = 0;`: Render döngüsünde bir bekleme veya atlama mekanizması sağlar.
        *   `virtual void SetServerTime(time_t tTime) = 0;`: İstemcinin zamanını sunucu zamanıyla senkronize eder.
    *   **Kamera Kontrolü**:
        *   `virtual void SetCenterPosition(float fx, float fy, float fz) = 0;`: Genellikle ana karakterin pozisyonunu veya kameranın odaklanacağı genel bir dünya koordinatını ayarlar.
        *   `virtual void SetEventCamera(const SCameraSetting& c_rCameraSetting) = 0;`: Belirtilen `SCameraSetting` yapılandırmasını anında kameraya uygular.
        *   `virtual void BlendEventCamera(const SCameraSetting& c_rCameraSetting, float fBlendTime) = 0;`: Mevcut kamera durumundan belirtilen `SCameraSetting`'e `fBlendTime` süresi boyunca yumuşak bir geçiş yapar.
        *   `virtual void SetDefaultCamera() = 0;`: Kamerayı varsayılan oyun içi takip veya serbest kamera moduna geri döndürür.
    *   **IME (Input Method Editor) Yönetimi**:
        *   `virtual void RunIMEUpdate() = 0;`: IME durumunu günceller.
        *   `virtual void RunIMETabEvent() = 0;`: IME içinde Tab tuşu olayını işler.
        *   `virtual void RunIMEReturnEvent() = 0;`: IME içinde Enter/Return tuşu olayını işler.
        *   `virtual void RunIMEChangeCodePage() = 0;`: IME için kod sayfasını değiştirir.
        *   `virtual void RunIMEOpenCandidateListEvent() = 0;`: IME aday listesi penceresini açar.
        *   `virtual void RunIMECloseCandidateListEvent() = 0;`: IME aday listesi penceresini kapatır.
        *   `virtual void RunIMEOpenReadingWndEvent() = 0;`: IME okuma penceresini açar.
        *   `virtual void RunIMECloseReadingWndEvent() = 0;`: IME okuma penceresini kapatır.

#### Kullanım Amacı ve Senaryoları

`IAbstractApplication` arayüzü, `UserInterface`'in oyunun çekirdek uygulama döngüsü ve temel sistemleriyle etkileşimde bulunması için merkezi bir noktadır. Örneğin:

*   UI elemanları, fare pozisyonunu alarak etkileşimleri yönetebilir.
*   Animasyonlar veya zamanla değişen UI efektleri, `GetGlobalTime` veya `GetGlobalElapsedTime` kullanarak kendilerini güncelleyebilir.
*   Özel oyun içi olaylar veya ara sahneler, `SetEventCamera` ve `BlendEventCamera` aracılığıyla kamera perspektifini kontrol edebilir.
*   Sohbet penceresi gibi metin giriş alanları, IME fonksiyonları aracılığıyla çeşitli dillerde doğru metin girişini destekleyebilir.

Bu arayüzün somut implementasyonu genellikle ana uygulama sınıfı (`CApp` veya benzeri) tarafından sağlanır.

### `AbstractCharacterManager.h`

#### Dosyanın Genel Amacı

Bu başlık dosyası, `IAbstractCharacterManager` adında soyut bir arayüz sınıfı tanımlar. Bu arayüz, `UserInterface` gibi modüllerin, oyundaki tüm karakter örneklerini (hem oyuncular hem de NPC'ler) yöneten merkezi bir sistemle etkileşim kurması için bir yol sunar. `TAbstractSingleton<IAbstractCharacterManager>`'dan türetilmiştir.

#### Arayüz Tanımları

*   **Bağımlılıklar**: `class CInstanceBase;` (ileri bildirim). `CInstanceBase`, oyundaki bir karakterin veya nesnenin temel temsilidir.
*   **Saf Sanal Fonksiyonlar**:
    *   `virtual void Destroy() = 0;`: Karakter yöneticisinin sahip olduğu tüm kaynakları temizler.
    *   `virtual CInstanceBase* GetInstancePtr(DWORD dwVID) = 0;`: Argüman olarak verilen Sanal ID (`dwVID`) ile eşleşen karakter örneğine (`CInstanceBase` işaretçisi) bir işaretçi döndürür.

#### Kullanım Amacı ve Senaryoları

`IAbstractCharacterManager` arayüzü, UI'nin belirli karakterlerle ilgili bilgileri alması veya onlarla etkileşimde bulunması gerektiğinde kullanılır:

*   **Hedefleme Sistemi**: Oyuncu bir karakteri hedef aldığında, UI bu karakterin bilgilerini göstermek için `GetInstancePtr` ile o karakterin örneğini alabilir.
*   **Kaynak Yönetimi**: Oyun sonlandığında veya harita değiştiğinde, `Destroy` metodu çağrılarak tüm karakter örneklerinin düzgün bir şekilde temizlenmesi sağlanır.

Bu arayüz, karakter verilerinin merkezi bir yerden yönetilmesini ve UI'nin bu verilere tutarlı bir şekilde erişmesini sağlar.

### `AbstractChat.h`

#### Dosyanın Genel Amacı

Bu başlık dosyası, `IAbstractChat` adında soyut bir arayüz sınıfı tanımlar. Bu arayüz, `UserInterface` modülünün veya oyunun diğer bölümlerinin, oyun içi sohbet sistemine mesaj göndermesi için basit bir yol sağlar. `TAbstractSingleton<IAbstractChat>`'dan türetilmiştir.

#### Arayüz Tanımları

*   **Saf Sanal Fonksiyonlar**:
    *   `virtual void AppendChat(int iType, const char* c_szChat) = 0;`:
        *   `int iType`: Eklenecek sohbet mesajının türünü belirtir (normal, fısıltı, sistem vb.).
        *   `const char* c_szChat`: Eklenecek asıl metin mesajı.

#### Kullanım Amacı ve Senaryoları

`IAbstractChat` arayüzü, oyunun herhangi bir yerinden sohbet penceresine mesaj yazdırmak için kullanılır:

*   **Oyuncu Sohbeti**: Oyuncunun yazdığı normal mesajlar, fısıltılar, parti veya lonca mesajları bu fonksiyon aracılığıyla sohbet sistemine iletilir.
*   **Sistem Mesajları**: Oyun tarafından oyuncuya iletilen bilgilendirme mesajları bu fonksiyon kullanılarak gösterilir.

Bu basit arayüz, sohbet mesajlarının merkezi bir şekilde işlenmesini sağlar.

### `AbstractPlayer.h`

#### Dosyanın Genel Amacı

Bu başlık dosyası, `IAbstractPlayer` adında soyut bir arayüz sınıfı tanımlar. Bu arayüz, `UserInterface` modülünün ve diğer ilgili sistemlerin, oyuncunun ana karakteriyle ilgili çeşitli bilgilere erişmesi, durumunu değiştirmesi ve karakterle ilgili olayları bildirmesi için merkezi bir iletişim noktası sunar. `TAbstractSingleton<IAbstractPlayer>`'dan türetilmiştir.

#### Arayüz Tanımları

*   **Bağımlılıklar**: `class CInstanceBase;` (ileri bildirim).
*   **Saf Sanal Fonksiyonlar (Önemli Gruplar)**:
    *   **Ana Karakter Yönetimi**: `GetMainCharacterIndex`, `SetMainCharacterIndex`, `IsMainCharacterIndex`, `GetName`, `SetRace`.
    *   **Durum (Status) Bilgileri**: `GetStatus` (HP, MP, Seviye, EXP vb.).
    *   **Dayanıklılık (Stamina)**: `StartStaminaConsume`, `StopStaminaConsume`.
    *   **Parti İşlemleri**: `IsPartyMemberByVID`, `PartyMemberVIDToPID`, `IsSamePartyMember`.
    *   **Eşya (Item) Yönetimi**: `SetItemData`, `SetItemCount`, `SetItemMetinSocket`, `SetItemAttribute`, ve `#ifdef` ile eklenen yeni sistem fonksiyonları (`SetItemTransmutationVnum`, `SetItemApplyRandom` vb.). Ayrıca `GetItemIndex`, `GetItemFlags`, `GetItemCount`, `IsEquipItemInSlot`.
    *   **Hızlı Erişim Yuvaları (QuickSlots)**: `AddQuickSlot`, `DeleteQuickSlot`, `MoveQuickSlot`.
    *   **Saldırı Gücü**: `SetWeaponPower`.
    *   **Hedefleme ve Bildirimler**: `SetTarget`, `NotifyCharacterUpdate`, `NotifyCharacterDead`, `NotifyDeletingCharacterInstance`, `NotifyChangePKMode`.
    *   **Özel Durumlar**: `SetObserverMode`, `SetComboSkillFlag`.
    *   **Duygu (Emotion) Yönetimi**: `StartEmotionProcess`, `EndEmotionProcess`.
    *   **Ana Karakter Örneği Erişimi**: `NEW_GetMainActorPtr` (`CInstanceBase*` döndürür).

#### Kullanım Amacı ve Senaryoları

`IAbstractPlayer` arayüzü, UI'nin oyuncuyla ilgili hemen hemen her şeye erişmesi ve onu etkilemesi için kritik bir rol oynar:

*   **Karakter Bilgi Penceresi**: Oyuncunun adını, statülerini, ekipmanlarını göstermek için bu arayüzdeki `Get` fonksiyonları kullanılır.
*   **Envanter Yönetimi**: Eşyaların görüntülenmesi, taşınması, kullanılması gibi işlemler bu arayüz üzerinden oyuncu verilerini günceller.
*   **Oyun İçi Olaylar**: Oyuncunun ölmesi, seviye atlaması gibi olaylar UI'ye bu arayüzdeki `Notify` fonksiyonları aracılığıyla bildirilir.

Bu arayüz, `UserInterface`'in oyunun temel mantığı olan `GameLib` veya doğrudan oyuncu verilerini tutan `CPythonPlayer` ile sıkı bir şekilde entegre olmasını sağlar.

### `AccountConnector.h` ve `AccountConnector.cpp`

#### Sınıfın Genel Amacı

`CAccountConnector` sınıfı, Metin2 istemcisinin hesap sunucusuyla (login sunucusu) iletişim kurmasını sağlayan temel bileşendir. Kullanıcının hesap adı ve şifresiyle giriş yapma, bağlantı durumunu yönetme, sunucudan gelen paketleri işleme ve bu süreçleri Python tarafına bildirme gibi kritik görevleri üstlenir. Temelde bir durum makinesi (state machine) prensibiyle çalışarak bağlantının farklı aşamalarını (offline, handshake, authentication) yönetir. `CNetworkStream` sınıfından türemiştir ve ağ iletişimi için gerekli temel fonksiyonları devralır. Aynı zamanda `CSingleton<CAccountConnector>`'dan türeyerek global erişilebilir bir yapıda tasarlanmıştır.

#### Başlık Dosyası Tanımları (`AccountConnector.h`)

`AccountConnector.h` dosyası, `CAccountConnector` sınıfının yapısını, üye değişkenlerini ve fonksiyon prototiplerini tanımlar.

*   **`enum STATE`**: Bağlantının mevcut durumunu belirtmek için kullanılır:
    *   `STATE_OFFLINE`: Bağlantı yok veya kesilmiş.
    *   `STATE_HANDSHAKE`: Sunucuyla ilk el sıkışma aşaması.
    *   `STATE_AUTH`: Kimlik doğrulama (login) aşaması.

*   **Önemli Üye Değişkenler**:
    *   `UINT m_eState`: Mevcut bağlantı durumunu tutar (`STATE` enum değerlerinden biri).
    *   `std::string m_strID`: Kullanıcının hesap ID'sini (kullanıcı adı) saklar.
    *   `std::string m_strPassword`: Kullanıcının şifresini saklar.
    *   `std::string m_strAddr`: Bağlanılacak ana oyun sunucusunun adresini saklar.
    *   `int m_iPort`: Bağlanılacak ana oyun sunucusunun port numarasını saklar.
    *   `PyObject* m_poHandler`: Python tarafında tanımlanmış ve ağ olaylarını (başarılı/başarısız giriş, bağlantı hataları vb.) işleyecek olan nesneye (genellikle bir Python sınıf örneği) işaret eder.
    *   `BOOL m_isWaitKey`: Matrix Card gibi ek güvenlik önlemleri için bekleme durumunu belirten bir bayrak (bu kodda tam olarak aktif kullanılmıyor gibi görünse de değişken olarak mevcut).

*   **Anahtar Fonksiyon Prototipleri**:
    *   `void SetHandler(PyObject* poHandler)`: Python olay işleyici nesnesini ayarlar.
    *   `void SetLoginInfo(const char* c_szName, const char* c_szPwd)`: Giriş yapılacak kullanıcı adı ve şifreyi ayarlar.
    *   `void ClearLoginInfo(void)`: Saklanan şifre bilgisini temizler.
    *   `bool Connect(const char* c_szAddr, int iPort, const char* c_szAccountAddr, int iAccountPort)`: Belirtilen hesap sunucusu adres ve portuna bağlantı başlatır. Ana oyun sunucusu adres ve portu da daha sonra kullanılmak üzere saklanır.
    *   `void Disconnect()`: Mevcut bağlantıyı sonlandırır.
    *   `void Process()`: Ağ olaylarını işler (gelen paketleri kontrol eder, bağlantı durumunu günceller).

#### C++ Implementasyon Detayları (`AccountConnector.cpp`)

`AccountConnector.cpp` dosyası, `.h` dosyasında tanımlanan fonksiyonların gerçek implementasyonlarını içerir.

*   **Bağlantı Süreci ve Durum Yönetimi**:
    *   `Connect()`: Bağlantı isteği bu fonksiyonla başlar. İlk olarak bazı şifreleme anahtarlarını (`__BuildClientKey` veya `__BuildClientKey_20050304Myevan`) oluşturur ve ardından `CNetworkStream::Connect` ile belirtilen hesap sunucusuna bağlanır. Bağlantı başarılı olursa `OnConnectSuccess` çağrılır ve durum `STATE_HANDSHAKE`'e geçer.
    *   `Process()`: Düzenli olarak çağrılan bu fonksiyon, `CNetworkStream::Process` ile temel ağ işlemlerini yaptıktan sonra `__StateProcess`'i çağırır.
    *   `__StateProcess()`: Mevcut `m_eState`'e göre ilgili durum işleyici fonksiyonunu (`__HandshakeState_Process` veya `__AuthState_Process`) çağırır.
    *   `__HandshakeState_Set()`, `__AuthState_Set()`, `__OfflineState_Set()`: Bağlantı durumunu değiştiren yardımcı fonksiyonlardır.

*   **Paket İşleme**:
    *   Her durum işleyici (`__HandshakeState_Process`, `__AuthState_Process`), o aşamada beklenen paketleri `__AnalyzePacket` veya `__AnalyzeVarSizePacket` kullanarak analiz eder.
    *   `__AnalyzePacket()`: Sabit boyutlu paketleri işler. Gelen veriyi `Peek` ile kontrol eder, eğer beklenen `uHeader` ile eşleşiyorsa ve yeterli veri (`uPacketSize`) gelmişse, belirtilen üye fonksiyon işaretçisini (`pfnDispatchPacket`) çağırarak paketi işler.
    *   `__AnalyzeVarSizePacket()`: Değişken boyutlu paketleri (başlığında boyut bilgisi olan) işler. Benzer şekilde başlığı ve ardından paketin tamamını kontrol edip ilgili işleyici fonksiyonu çağırır.
    *   Çeşitli `__AuthState_Recv*` fonksiyonları (örneğin, `__AuthState_RecvPhase`, `__AuthState_RecvHandshake`, `__AuthState_RecvAuthSuccess`, `__AuthState_RecvAuthFailure`): Sunucudan gelen spesifik paketleri (`TPacketGC*` yapıları) alır, içeriğini işler ve gerekirse sunucuya yanıt paketleri (`TPacketCG*`) gönderir. Örneğin, `__AuthState_RecvPhase` içinde `PHASE_AUTH` geldiğinde, istemci `TPacketCGLogin3` paketini kullanıcı adı, şifre, HWID ve şifreleme anahtarlarıyla doldurup sunucuya gönderir.

*   **Python Etkileşimi**:
    *   `m_poHandler` üye değişkeni, C++ tarafındaki önemli olayları Python katmanına bildirmek için kullanılır.
    *   `PyCallClassMemberFunc(m_poHandler, "PythonMetodAdi", Py_BuildValue("(format)", arg1, ...))` fonksiyonu aracılığıyla Python nesnesinin metodları çağrılır.
        *   `OnConnectFailure()`: Bağlantı kurulamadığında Python'daki `OnConnectFailure` metodunu çağırır.
        *   `__AuthState_RecvAuthSuccess()`: Başarılı giriş sonrası, ana oyun sunucusuna bağlanmak için `CPythonNetworkStream::Connect` çağrılır. Eğer `TPacketGCAuthSuccess` içinde `bResult` false ise, Python'daki `OnLoginFailure` çağrılır.
        *   `__AuthState_RecvAuthFailure()`: Giriş başarısız olduğunda, Python'daki `OnLoginFailure` metodunu sunucudan gelen hata mesajıyla çağırır.
        *   `OnRemoteDisconnect()` ve `OnDisconnect()`: Bağlantı koptuğunda ilgili Python olaylarını tetikleyebilir (örneğin, `OnExit`).

*   **Şifreleme ve Güvenlik**:
    *   `__BuildClientKey()`: `#if !defined(__IMPROVED_PACKET_ENCRYPTION__)` durumunda çağrılır. Rastgele `g_adwEncryptKey` üretir ve sabit bir anahtar (`"JyTxtHljHJlVJHorRM301vf@4fvj10-v"`) kullanarak `old_tea_encrypt` ile `g_adwDecryptKey`'i oluşturur. Bu anahtarlar muhtemelen paket şifrelemesi için kullanılır.
    *   `__BuildClientKey_20050304Myevan()`: Belirli yerelleştirmeler (`LocaleService_IsYMIR()`) için farklı bir anahtar oluşturma mekanizmasıdır (detayları `AccountConnector.h` içinde sadece prototip olarak var, implementasyonu bu dosyada yok).
    *   `g_adwEncryptKey` ve `g_adwDecryptKey`: Global şifreleme/deşifreleme anahtarları (`extern` olarak tanımlı, muhtemelen başka bir dosyada değerleri atanıyor). `TPacketCGLogin3` paketi gönderilirken `LoginPacket.adwClientKey` alanına `g_adwEncryptKey` değerleri kopyalanır.
    *   `__IMPROVED_PACKET_ENCRYPTION__`: Bu makro tanımlıysa, `__AuthState_RecvKeyAgreement` ve `__AuthState_RecvKeyAgreementCompleted` fonksiyonları aracılığıyla Diffie-Hellman benzeri bir anahtar anlaşması protokolü kullanılır (`Prepare`, `Activate`, `ActivateCipher` fonksiyonları çağrılır). Bu, daha güvenli bir simetrik şifreleme anahtarı değişimi sağlar.
    *   `HybridCrypt` Paketleri (`__AuthState_RecvHybridCryptKeys`, `__AuthState_RecvHybridCryptSDB`): `CEterPackManager` aracılığıyla EterPack paketlerinin şifrelenmesi için ek anahtar ve veri bloklarını alır. Bu, istemcinin kullandığı veri paketlerinin içeriğinin korunmasına yardımcı olur.
    *   `dwLoginKey`: `__AuthState_RecvAuthSuccess` içinde sunucudan gelen bu anahtar, `CEterPackManager::DecryptPackIV` ile paketlerin IV (Initialization Vector) değerlerini çözmek için kullanılır.
    *   `CHWIDManager::instance().GetHWID()`: Donanım kimliğini (Hardware ID) alıp giriş paketine ekler. Bu, hesap güvenliği için ek bir katman olabilir.

#### Oyunda Kullanım Amacı ve Senaryoları

`CAccountConnector`, oyuncu oyunu başlattığında ve giriş ekranına geldiğinde devreye girer:

1.  Oyuncu kullanıcı adı ve şifresini girip "Giriş Yap" butonuna tıkladığında, Python UI scriptleri bu bilgileri alır ve `CAccountConnector::SetLoginInfo` ile C++ tarafına iletir.
2.  Ardından `CAccountConnector::Connect` çağrılarak hesap sunucusuna bağlantı işlemi başlatılır.
3.  Bağlantı kurulduktan sonra, istemci ve sunucu arasında bir dizi paket alışverişi gerçekleşir (Handshake, Ping/Pong, Login paketleri, şifreleme anahtarı anlaşmaları).
4.  Giriş başarılı olursa (`__AuthState_RecvAuthSuccess`), hesap sunucusu istemciye ana oyun sunucusuna bağlanması için gerekli bilgileri (veya bir onay) gönderir. `CAccountConnector` bu noktada genellikle bağlantısını keser ve ana oyun sunucusuyla iletişimi `CPythonNetworkStream` devralır.
5.  Giriş başarısız olursa (`__AuthState_RecvAuthFailure`), Python tarafındaki `OnLoginFailure` fonksiyonu çağrılarak kullanıcıya hata mesajı gösterilir (örneğin, "Yanlış şifre", "Hesap bulunamadı").
6.  Bağlantı sırasında herhangi bir ağ hatası oluşursa (`OnConnectFailure`, `OnRemoteDisconnect`), ilgili Python fonksiyonları çağrılarak kullanıcı bilgilendirilir veya oyun uygun şekilde sonlandırılır.

Bu sınıf, istemcinin ilk kimlik doğrulama adımını gerçekleştirmesi ve güvenli bir şekilde oyun dünyasına geçiş yapabilmesi için hayati öneme sahiptir.

### `AffectFlagContainer.h` ve `AffectFlagContainer.cpp`

#### Sınıfın Genel Amacı

`CAffectFlagContainer` sınıfı, genellikle karakterler veya nesneler üzerinde aktif olan çeşitli durumları, etkileri veya "buff/debuff"ları temsil etmek için kullanılan bir bit alanı (bitfield) yöneticisidir. Sabit bir boyutta (varsayılan olarak 64 bit) bir dizi bayrağı (flag) verimli bir şekilde saklamak ve yönetmek için tasarlanmıştır. Her bir bit, belirli bir durumun (örneğin, zehirlenme, hız artışı, sersemletme) aktif olup olmadığını temsil edebilir.

#### Başlık Dosyası Tanımları (`AffectFlagContainer.h`)

`AffectFlagContainer.h` dosyası, `CAffectFlagContainer` sınıfının yapısını ve arayüzünü tanımlar.

*   **Sabitler**:
    *   `BIT_SIZE`: Konteyner tarafından yönetilebilen toplam bit sayısını tanımlar (varsayılan 64).
    *   `BYTE_SIZE`: Bitleri saklamak için gereken bayt sayısını hesaplar. `BIT_SIZE / 8` ve eğer `BIT_SIZE` 8'in katı değilse ek bir bayt içerir.

*   **Anahtar Fonksiyon Prototipleri**:
    *   `CAffectFlagContainer()`: Kurucu metot, bit alanını temizler.
    *   `~CAffectFlagContainer()`: Yıkıcı metot.
    *   `void Clear()`: Tüm bitleri (bayrakları) 0'a (pasif duruma) ayarlar.
    *   `void CopyInstance(const CAffectFlagContainer& c_rkAffectContainer)`: Başka bir `CAffectFlagContainer` örneğinin tüm bayraklarını mevcut örneğe kopyalar.
    *   `void Set(UINT uPos, bool isSet)`: Belirtilen pozisyondaki (`uPos`) biti (bayrağı) ayarlar (`isSet = true`) veya temizler (`isSet = false`).
    *   `bool IsSet(UINT uPos) const`: Belirtilen pozisyondaki bitin ayarlı (aktif) olup olmadığını kontrol eder.
    *   `void CopyData(UINT uPos, UINT uByteSize, const void* c_pvData)`: Verilen bir ham veri bloğundan (`c_pvData`) belirtilen sayıda baytı (`uByteSize`), konteyner içindeki belirtilen bit pozisyonundan (`uPos`) başlayarak kopyalar. Bu, genellikle ağdan gelen paketlenmiş bayrak verilerini yüklemek için kullanılır.
    *   `void ConvertToPosition(unsigned* uRetX, unsigned* uRetY) const`: Konteynerin ilk 8 baytını (64 bit) iki adet 32-bit `unsigned int` (genellikle X ve Y koordinatları olarak yorumlanabilir) olarak döndürür. Bu, bayrakların belirli bir amaç için (örneğin, özel bir efektin konumu) kullanıldığı özel bir senaryoyu işaret eder.

*   **Özel Üyeler**:
    *   `typedef unsigned char Element`: Bitleri saklamak için kullanılan temel veri tipini tanımlar.
    *   `Element m_aElement[BYTE_SIZE]`: Bit bayraklarını içeren asıl dizi.

#### C++ Implementasyon Detayları (`AffectFlagContainer.cpp`)

`AffectFlagContainer.cpp` dosyası, `.h` dosyasında tanımlanan fonksiyonların gerçek implementasyonlarını içerir.

*   **`Clear()`**: `memset` kullanarak `m_aElement` dizisinin tüm baytlarını 0 yapar, böylece tüm bayraklar temizlenir.
*   **`CopyInstance()`**: `memcpy` kullanarak kaynak konteynerin `m_aElement` dizisini hedef konteynerin dizisine kopyalar.
*   **`Set(UINT uPos, bool isSet)`**:
    *   Öncelikle `uPos`'un geçerli bir bit pozisyonu olup olmadığını kontrol eder (maksimum `BYTE_SIZE * 8`).
    *   `uPos / 8` ile ilgili baytı (`rElement`) bulur.
    *   `BYTE(1 << (uPos & 7))` ile o bayt içinde doğru biti hedefleyen bir maske (`bMask`) oluşturur. (`uPos & 7` işlemi, `uPos % 8` ile aynıdır ve bitin bayt içindeki pozisyonunu verir).
    *   `isSet` true ise, `rElement |= bMask` (OR işlemi) ile biti 1 yapar.
    *   `isSet` false ise, `rElement &= ~bMask` (AND NOT işlemi) ile biti 0 yapar.
*   **`IsSet(UINT uPos) const`**:
    *   `uPos`'un geçerliliğini kontrol eder.
    *   İlgili baytı (`c_rElement`) ve bit maskesini (`bMask`) `Set` fonksiyonundaki gibi bulur.
    *   `(c_rElement & bMask)` işlemi ile bitin 1 olup olmadığını kontrol eder. Eğer sonuç sıfır değilse bit ayarlı demektir ve `true` döndürür.
*   **`CopyData(UINT uPos, UINT uByteSize, const void* c_pvData)`**:
    *   `c_pvData`'yı `BYTE*` (işaretsiz karakter işaretçisi) olarak ele alır.
    *   Belirtilen `uPos`'tan başlayarak `uByteSize * 8` (toplam bit sayısı) kadar döngü kurar.
    *   Her bir kaynak bitini okur (`*c_pbData & bMask`) ve `Set()` fonksiyonunu kullanarak hedef `CAffectFlagContainer` içindeki karşılık gelen bite yazar.
    *   Her baytın tüm bitleri işlendiğinde bir sonraki bayta geçer.
*   **`ConvertToPosition(unsigned* uRetX, unsigned* uRetY) const`**:
    *   `m_aElement` dizisinin başlangıcını bir `DWORD*` (32-bit işaretsiz tamsayı işaretçisi) olarak yorumlar.
    *   İlk `DWORD`'ü (`pos[0]`) `*uRetX`'e atar.
    *   İkinci `DWORD`'ü (`pos[1]`) `*uRetY`'e atar.
    *   Bu, 64 bitlik bayrak alanının ilk 32 bitini X koordinatı, sonraki 32 bitini Y koordinatı olarak kullanmak gibi özel bir durumu yönetir.

#### Oyunda Kullanım Amacı ve Senaryoları

`CAffectFlagContainer`, oyunda bir karakterin veya nesnenin sahip olabileceği çok sayıda ikili durumu (aktif/pasif) kompakt bir şekilde saklamak için kullanılır.

*   **Karakter Etkileri (Buffs/Debuffs)**: Bir karakterin zehirlenmiş, hızlanmış, yavaşlamış, sersemlemiş gibi durumları bu bayraklarla temsil edilebilir. Örneğin, AFFECT_POISON için 10. bit, AFFECT_SPEEDUP için 15. bit kullanılabilir. Sunucu bu bayrakları istemciye gönderdiğinde, istemci `CAffectFlagContainer::CopyData` ile bu bilgiyi yükleyebilir.
*   **UI Gösterimi**: Kullanıcı arayüzü, bir karakterin `CAffectFlagContainer`'ını kontrol ederek (`IsSet` ile) hangi etkilerin aktif olduğunu belirleyebilir ve buna göre ilgili ikonları (örneğin, ekranın köşesindeki buff ikonları) gösterebilir.
*   **Oyun Mantığı**: Oyun mantığı, bir karakterin belirli bir bayrağa sahip olup olmadığını kontrol ederek özel davranışlar uygulayabilir (örneğin, sersemlemişse hareket edememesi).
*   **Özel Kullanımlar (`ConvertToPosition`)**: `ConvertToPosition` fonksiyonunun varlığı, bu bayrak konteynerinin bazen genel durum bayrakları dışında, örneğin bir yeteneğin etki edeceği X,Y koordinatları gibi daha spesifik verileri taşımak için de kullanılabileceğini gösterir. Ancak bu, genellikle daha az yaygın bir kullanımdır ve ana amaç çoklu durum bayraklarını yönetmektir.

Bu sınıf, özellikle ağ üzerinden durum bilgilerinin verimli bir şekilde iletilmesi ve istemci tarafında kolayca yönetilmesi gerektiğinde kullanışlıdır.

### `CameraProcedure.cpp` (CCamera Sınıfının Parçası)

#### Dosyanın Genel Amacı

`CameraProcedure.cpp` dosyası, oyun içi kameranın (`CCamera` sınıfı) çeşitli hareket ve çarpışma mantıklarını içeren fonksiyon implementasyonlarını barındırır. Bu dosyadaki kodlar, kameranın oyun dünyasındaki arazi ve diğer nesnelerle nasıl etkileşime girdiğini, nasıl güncellendiğini ve oyuncu kontrollerine veya oyun olaylarına nasıl tepki verdiğini yönetir. Bu dosya, genellikle `../EterLib/Camera.h` gibi bir başlık dosyasında tanımlanan `CCamera` sınıfının bir parçasıdır ve o sınıfa ait üye fonksiyonları implemente eder.

#### Temel Fonksiyonlar ve İşleyiş

*   **`CCamera::ProcessTerrainCollision()`**:
    *   Kameranın oyun dünyasındaki araziyle çarpışmasını yönetir.
    *   `CPythonBackground::Instance().GetPickingPointWithRayOnlyTerrain()` fonksiyonunu kullanarak kameradan araziye doğru ışınlar gönderir ve çarpışma noktalarını tespit eder.
    *   Eğer bir çarpışma tespit edilirse, kameranın gözlem noktasını (`m_v3Eye`) ayarlar, böylece kamera arazinin altına girmez veya içine saplanmaz.
    *   Kameranın durumunu `CAMERA_STATE_CANTGODOWN` gibi değerlere ayarlayarak diğer sistemlere bilgi verebilir.

*   **`CCamera::ProcessBuildingCollision()`**:
    *   Kameranın binalar veya diğer büyük statik objelerle çarpışmasını engellemek için tasarlanmıştır. (Not: Koddaki bazı kısımlar yorum satırı veya geliştirme aşamasında olabilir.)
    *   `CCullingManager` aracılığıyla kameranın etrafındaki potansiyel çarpışma nesnelerini tespit eder.
    *   `CameraCollisionChecker` adında bir yardımcı yapı (struct) kullanarak dinamik küre çarpışmalarını kontrol eder.
    *   Çarpışma durumunda, kameranın açısal hızını (`m_v3AngularVelocity`) ayarlayarak kameranın objelerin içine girmesini engellemeye çalışır ve yumuşak bir şekilde etrafından dolaşmasını sağlamayı hedefler.

*   **`CCamera::Update()`**:
    *   Kameranın her frame'de güncellenmesini sağlar.
    *   `m_v3AngularVelocity` (açısal hız) değerini kullanarak kamerayı hedefin etrafında döndürür ve yakınlaştırma/uzaklaştırma yapar.
    *   Kamera mesafesini `CCamera::CAMERA_MIN_DISTANCE` ve `CCamera::CAMERA_MAX_DISTANCE` (muhtemelen `CCamera` sınıfının statik sabitleri) arasında sınırlar.
    *   Eğer `m_bProcessTerrainCollision` (muhtemelen `CCamera` sınıfının bir üye değişkeni) `true` ise `ProcessTerrainCollision()` fonksiyonunu çağırır.
    *   Kameranın hedef yüksekliğini, mevcut zoom seviyesine göre ayarlar (`CAMERA_TARGET_STANDARD` ve `CAMERA_TARGET_FACE` sabitleri arasında bir interpolasyon yaparak).
    *   `#ifdef __20040725_CAMERA_WORK__` bloğu altında sinematik kamera hareketleri için ek mantık içerebilir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyadaki prosedürler, `CCamera` sınıfının ayrılmaz bir parçası olarak, oyuncuya akıcı ve sorunsuz bir kamera deneyimi sunmak için kritik öneme sahiptir:

*   Oyuncu karakterini takip ederken kameranın duvarların veya arazinin içine girmesini engeller.
*   Fare veya klavye ile yapılan kamera kontrollerine (döndürme, yakınlaştırma) tepki verir.
*   Oyun içindeki özel olaylar veya ara sahneler için tanımlanmış kamera hareketlerini gerçekleştirebilir.
*   Kameranın minimum ve maksimum zoom mesafelerini korur.

Bu fonksiyonlar, genellikle oyunun ana döngüsü içinde `CCamera` nesnesinin kendi `Update` metodu çağrıldığında dolaylı olarak çalıştırılır.

### `CefClientApp.h` ve `CefClientApp.cpp`

#### Sınıfın Genel Amacı

`CefClientApp` sınıfı (ve onu implemente eden `ClientApp` sınıfı), Metin2 istemcisine Chromium Embedded Framework (CEF) entegrasyonunun bir parçasını oluşturur. CEF, uygulamalara web tarayıcı motoru (Chromium) gömmeyi sağlar. `ClientApp`, özellikle CEF'in "render process" (işleme süreci) tarafındaki olaylarını ve yaşam döngüsünü yönetmek için tasarlanmıştır. Temel amacı, JavaScript (CEF içinde çalışan web sayfalarında) ile C++ (oyun istemcisi kodu) arasında bir köprü kurmaktır.

#### Başlık Dosyası Tanımları (`CefClientApp.h`)

`CefClientApp.h` dosyası, `ClientApp` sınıfının tanımını içerir.

*   **`class ClientApp : public CefApp, public CefRenderProcessHandler`**:
    *   `CefApp`: CEF'in genel uygulama seviyesi geri çağrılarını (callback) yönetmek için temel arayüz.
    *   `CefRenderProcessHandler`: CEF'in render sürecine özgü geri çağrılarını yönetmek için arayüz. `ClientApp` bu arayüzü implemente ederek render sürecindeki olayları işleyebilir.
*   **`CefRefPtr<CefRenderProcessHandler> GetRenderProcessHandler() override`**:
    *   CEF tarafından çağrıldığında `this` (yani `ClientApp` örneğini) döndürür. Bu, render süreci olaylarının bu sınıf tarafından işleneceğini belirtir.
*   **`void OnWebKitInitialized() override`**:
    *   CEF render sürecinde WebKit (Chromium'un render motoru) başlatıldığında çağrılan önemli bir geri çağrı fonksiyonudur. JavaScript uzantılarının kaydedilmesi gibi işlemler genellikle burada yapılır.
*   **`IMPLEMENT_REFCOUNTING(ClientApp)`**:
    *   CEF nesneleri için referans sayımını implemente eden bir makrodur.

#### C++ Implementasyon Detayları (`CefClientApp.cpp`)

`CefClientApp.cpp` dosyası, `ClientApp` sınıfının fonksiyonlarını implemente eder.

*   **`ClientApp::ClientApp() = default;`**:
    *   Varsayılan kurucu metot.
*   **`void ClientApp::OnWebKitInitialized()`**:
    *   Bu fonksiyon içinde, özel bir JavaScript uzantısı (`v8/app`) CEF'e kaydedilir.
    *   `app_code` adlı bir string, JavaScript tarafında `app` adında bir nesne ve bu nesne altında `ChangeTextInJS` adında bir fonksiyon tanımlar.
    *   Bu JavaScript fonksiyonu (`app.ChangeTextInJS`), `native function ChangeTextInJS();` bildirimiyle C++ tarafında implemente edilecek bir yerel (native) fonksiyona çağrı yapar.
    *   `CefRegisterExtension("v8/app", app_code, new ClientV8ExtensionHandler(this))` satırı, bu JavaScript kodunu ve onunla ilişkili yerel C++ işleyicisini (`ClientV8ExtensionHandler`, muhtemelen başka bir dosyada tanımlıdır) CEF'e kaydeder.

#### Oyunda Kullanım Amacı ve Senaryoları

`CefClientApp`, oyun içinde web teknolojilerini (HTML, CSS, JavaScript) kullanarak çeşitli arayüzler veya özellikler oluşturmak için kullanılır:

*   **Gelişmiş UI Elemanları**: Karmaşık kullanıcı arayüzleri (örneğin, web tabanlı item shop, duyuru panoları, karmaşık bilgi ekranları) CEF kullanılarak tasarlanabilir.
*   **JavaScript-C++ İletişimi**: Web arayüzündeki JavaScript kodu, `ClientApp` aracılığıyla kaydedilen uzantılar sayesinde oyunun C++ tarafındaki fonksiyonları çağırabilir (örneğin, bir eşya satın alma isteği göndermek, karakter bilgilerini C++'dan çekmek).
*   **Dinamik İçerik**: Oyun içi olaylara veya verilere göre dinamik olarak güncellenen web tabanlı içerikler gösterilebilir.

Özetle, `CefClientApp`, CEF'in render sürecini yönetir ve JavaScript ile C++ arasında köprü kurarak oyun istemcisinin web teknolojileriyle zenginleştirilmesine olanak tanır.

### `CefClientHandler.h` ve `CefClientHandler.cpp`

#### Sınıfın Genel Amacı

`ClientHandler` sınıfı, Chromium Embedded Framework (CEF) entegrasyonunun önemli bir parçasıdır ve özellikle CEF tarayıcı örneğinin yaşam döngüsü olaylarını yönetir. Her bir CEF tarayıcı penceresi için bir `ClientHandler` örneği oluşturulabilir. Temel amacı, tarayıcı penceresinin oluşturulması, kapatılması gibi olayları ele almak ve tarayıcı örneğine erişim sağlamaktır.

#### Başlık Dosyası Tanımları (`CefClientHandler.h`)

`CefClientHandler.h` dosyası, `ClientHandler` sınıfının yapısını ve arayüzünü tanımlar.

*   **`class ClientHandler : public CefClient, public CefLifeSpanHandler`**:
    *   `CefClient`: CEF tarayıcı istemcisi için genel bir arayüzdür. Bu arayüz, tarayıcıyla ilgili çeşitli geri çağrıları (örneğin, yaşam döngüsü, bağlam menüsü, yükleme olayları) almak için farklı "handler" (işleyici) arayüzlerini döndürme yöntemleri sağlar.
    *   `CefLifeSpanHandler`: Tarayıcı penceresinin oluşturulması (`OnAfterCreated`), kapanması (`DoClose`, `OnBeforeClose`) gibi yaşam döngüsü olaylarını yönetmek için bir arayüzdür. `ClientHandler` bu arayüzü implemente eder.
*   **Yapıcı `ClientHandler()`**:
    *   Sınıf örneği oluşturulduğunda çağrılır. `m_BrowserHwnd`'yi (tarayıcı penceresinin handle'ı) `nullptr` olarak ilklendirir.
*   **Erişim Fonksiyonları**:
    *   `CefRefPtr<CefBrowser> GetBrowser()`: Yönetilen CEF tarayıcı örneğine bir referans sayımlı işaretçi (`CefRefPtr`) döndürür.
    *   `CefWindowHandle GetBrowserHwnd()`: Yönetilen tarayıcı penceresinin işletim sistemi seviyesindeki pencere handle'ını döndürür.
*   **`CefClient` Metotları (Override Edilmiş)**:
    *   `virtual CefRefPtr<CefLifeSpanHandler> GetLifeSpanHandler() override`: CEF'e, yaşam döngüsü olaylarını bu `ClientHandler` örneğinin yöneteceğini bildirir (`return this;`).
*   **`CefLifeSpanHandler` Metotları (Override Edilmiş)**:
    *   `virtual bool DoClose(CefRefPtr<CefBrowser> browser) override`: Tarayıcı kapatılmak üzereyken çağrılır. `false` döndürmek, tarayıcının normal şekilde kapatılmasına izin verir. `true` döndürmek kapatma işlemini engelleyebilir.
    *   `virtual void OnAfterCreated(CefRefPtr<CefBrowser> browser) override`: Bir tarayıcı penceresi başarıyla oluşturulduktan hemen sonra çağrılır.
    *   `virtual void OnBeforeClose(CefRefPtr<CefBrowser> browser) override`: Bir tarayıcı penceresi kapatılmadan hemen önce çağrılır. Kaynakların serbest bırakılması için uygun bir yerdir.
*   **Korunan Üye Değişkenler**:
    *   `CefRefPtr<CefBrowser> m_Browser`: Bu `ClientHandler` tarafından yönetilen asıl tarayıcı nesnesine işaret eder.
    *   `CefWindowHandle m_BrowserHwnd`: Tarayıcı penceresinin handle'ını saklar.
*   **`IMPLEMENT_REFCOUNTING(ClientHandler)`**:
    *   CEF nesneleri için standart referans sayımı mekanizmasını implemente eden bir makrodur. CEF, nesnelerin bellek yönetimini referans sayımı ile yapar.

#### C++ Implementasyon Detayları (`CefClientHandler.cpp`)

`CefClientHandler.cpp` dosyası, `.h` dosyasında tanımlanan fonksiyonların gerçek implementasyonlarını içerir.

*   **`ClientHandler::ClientHandler() : m_BrowserHwnd(nullptr) {}`**:
    *   Kurucu metot, `m_BrowserHwnd` üye değişkenini `nullptr` olarak başlatır.
*   **`bool ClientHandler::DoClose(CefRefPtr<CefBrowser> browser)`**:
    *   Bu implementasyonda her zaman `false` döndürür. Bu, tarayıcının kapatılma isteğini engellemez ve tarayıcının normal şekilde kapatılmasına izin verir. Eğer belirli koşullarda kapatmayı engellemek veya özel bir işlem yapmak gerekseydi, bu fonksiyonun mantığı değiştirilebilirdi.
*   **`void ClientHandler::OnAfterCreated(CefRefPtr<CefBrowser> browser)`**:
    *   Bu fonksiyon, yeni bir tarayıcı penceresi oluşturulduğunda çağrılır.
    *   `if (!m_Browser.get())`: Eğer bu `ClientHandler` örneği henüz bir ana tarayıcıya sahip değilse (yani `m_Browser` boşsa), gelen `browser` örneğini kendi ana tarayıcısı olarak ayarlar.
    *   `m_Browser = browser;`: Tarayıcı işaretçisini saklar.
    *   `m_BrowserHwnd = browser->GetHost()->GetWindowHandle();`: Tarayıcı penceresinin handle'ını alır ve saklar. Bu, pop-up pencereler değil, ana alt pencere (child window) için yapılır.
*   **`void ClientHandler::OnBeforeClose(CefRefPtr<CefBrowser> browser)`**:
    *   Bir tarayıcı penceresi kapatılmadan hemen önce çağrılır.
    *   `if (m_BrowserHwnd == browser->GetHost()->GetWindowHandle())`: Eğer kapatılmakta olan tarayıcı, bu `ClientHandler` tarafından yönetilen ana tarayıcı ise:
        *   `m_Browser = nullptr;`: Tarayıcıya olan referansı serbest bırakır. Bu, CEF'in tarayıcı nesnesini ve ilişkili kaynakları düzgün bir şekilde yok edebilmesi için önemlidir.

#### Oyunda Kullanım Amacı ve Senaryoları

`ClientHandler`, oyun içinde gömülü CEF tarayıcı örneklerinin davranışını ve yaşam döngüsünü yönetmek için kullanılır:

1.  **Tarayıcı Oluşturma**: Oyun içinde yeni bir web içeriği gösterilmesi gerektiğinde (örneğin, bir web tabanlı duyuru penceresi, item mall arayüzü), bir CEF tarayıcı örneği oluşturulur. Bu oluşturma sırasında veya hemen sonrasında, `ClientHandler::OnAfterCreated` çağrılır. Bu aşamada, `ClientHandler` tarayıcıya bir referans tutar ve pencere handle'ını saklar. Bu handle, tarayıcı penceresini oyunun arayüzüne entegre etmek veya boyutunu/konumunu yönetmek için kullanılabilir.

2.  **Tarayıcı Kapatma**: Kullanıcı web arayüzünü kapattığında veya ilgili oyun özelliği sonlandığında, CEF tarayıcı örneği kapatılır.
    *   `ClientHandler::DoClose` çağrılır. Varsayılan olarak kapatmaya izin verir.
    *   `ClientHandler::OnBeforeClose` çağrılır. Bu, `ClientHandler`'ın tarayıcıya olan referansını (`m_Browser = nullptr;`) serbest bırakması için kritik bir andır, böylece CEF bellek sızıntısı olmadan tarayıcıyı temizleyebilir.

3.  **Olay Yönetimi**: `ClientHandler`, `CefClient` arayüzü üzerinden `GetLifeSpanHandler` dışında başka handler'lar da döndürebilir (örneğin, `GetContextMenuHandler`, `GetDisplayHandler`, `GetLoadHandler`). Bu sayede, sağ tıklama menüleri, konsol mesajları, sayfa yükleme durumları gibi daha birçok tarayıcı olayını C++ tarafında ele almak mümkün olur. Bu örnekte sadece `LifeSpanHandler` üzerinde durulmuştur.

4.  **Tarayıcı Erişimi**: Oyunun C++ kodu, bir `ClientHandler` örneği üzerinden `GetBrowser()` metodunu kullanarak ilişkili CEF tarayıcı nesnesine erişebilir. Bu erişim, tarayıcıya komutlar göndermek (örneğin, farklı bir URL yüklemek, JavaScript çalıştırmak) veya tarayıcı hakkında bilgi almak için kullanılabilir.

Özetle, `ClientHandler`, CEF tarayıcılarının oyun istemcisi içindeki temel yaşam döngüsünü yöneten, tarayıcı olaylarına yanıt veren ve C++ ile tarayıcı arasındaki etkileşime aracılık eden bir köprü görevi görür. 

---

**Devamı için bakınız: [UserInterface Referans Kılavuzu - Bölüm 2](./client_UserInterface_Referans_Part2.md)**

*(Devamı için [UserInterface Referans Kılavuzu - Bölüm 3](./client_UserInterface_Referans_Part3.md), [UserInterface Referans Kılavuzu - Bölüm 4](./client_UserInterface_Referans_Part4.md), [UserInterface Referans Kılavuzu - Bölüm 5](./client_UserInterface_Referans_Part5.md) ve [UserInterface Referans Kılavuzu - Bölüm 6](./client_UserInterface_Referans_Part6.md) dosyalarına bakınız.)** 