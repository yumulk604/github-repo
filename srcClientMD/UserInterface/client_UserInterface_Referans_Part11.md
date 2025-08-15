# UserInterface Referans Kılavuzu - Bölüm 11

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonMountUpGrade.h` (Gelişmiş Binek Yükseltme Sistemi Başlık Dosyası)](#pythonmountupgradeh-gelişmiş-binek-yükseltme-sistemi-başlık-dosyası)
*   [`PythonMountUpGrade.cpp` (Gelişmiş Binek Yükseltme Sistemi Implementasyonu ve `mupgrd` Modülü)](#pythonmountupgradecpp-gelişmiş-binek-yükseltme-sistemi-implementasyonu-ve-mupgrd-modülü)
*   [`PythonNetworkDatagram.h` (UDP Ağ İletişim Sistemi Başlık Dosyası - Yorum Satırında)](#pythonnetworkdatagramh-udp-ağ-iletişim-sistemi-başlık-dosyası---yorum-satırında)
*   [`PythonNetworkDatagram.cpp` (UDP Ağ İletişim Sistemi Implementasyonu - Yorum Satırında)](#pythonnetworkdatagramcpp-udp-ağ-iletişim-sistemi-implementasyonu---yorum-satırında)
*   [`PythonNetworkDatagramModule.cpp` (UDP Ağ İletişim Python Modülü - Yorum Satırında)](#pythonnetworkdatagrammodulecpp-udp-ağ-iletişim-python-modülü---yorum-satırında)
*   [`PythonNetworkStream.h` (Ağ Akışı Yönetimi Başlık Dosyası)](#pythonnetworkstreamh-ağ-akışı-yönetimi-başlık-dosyası)
*   [`PythonNetworkStream.cpp` (Ağ Akışı Yönetimi Implementasyonu)](#pythonnetworkstreamcpp-ağ-akışı-yönetimi-implementasyonu)
*   [`PythonNetworkStreamModule.cpp` (Ağ Akışı Python Modülü)](#pythonnetworkstreammodulecpp-ağ-akışı-python-modülü)

---

### `PythonMountUpGrade.h` (Gelişmiş Binek Yükseltme Sistemi Başlık Dosyası)

Bu başlık dosyası, `ENABLE_RIDING_EXTENDED` makrosu etkinleştirildiğinde kullanılan `CPythonMountUpGrade` adlı bir singleton C++ sınıfını tanımlar. Bu sınıf, oyundaki bineklerin (atlar ve diğer binekler) seviye atlama ve deneyim (EXP) kazanma işlemlerinin istemci tarafındaki mantığını yönetir. Aynı zamanda, bu sistemle ilgili çeşitli sabitleri, enum yapılarını ve binek seviyelerine göre EXP ve ücret tablolarını içerir.

#### Dosyanın Genel Amaçları

*   **`CPythonMountUpGrade` Sınıf Bildirimi**:
    *   `CSingleton<CPythonMountUpGrade>`'den miras alır.
    *   Binek seviyesi (`m_HorseLevel`), yükseltme başarısızlık durumu (`m_MountUpGradeFail`) ve mevcut binek deneyimi (`m_MountUpGradeExistingExp`) gibi verileri tutar.
    *   Bu verileri ayarlamak (`Set*`) ve almak (`Get*`, `Is*`) için metotlar sunar.
    *   Mevcut binek seviyesine göre bir sonraki seviye için gerekli EXP'yi (`GetMountUpGradeNecessaryExp`) ve yükseltme ücretini (`GetMountUpGradePrice`) hesaplayan metotlar içerir.
    *   Python UI ile etkileşim için bir pencere nesnesi (`m_window`) tutar ve bu pencereyi ayarlamak (`SetWindow`), yok etmek (`DestroyWindow`), yenilemek (`Refresh`) ve UI'a sohbet mesajları göndermek (`Chat`) için metotlar sağlar.
    *   Sistem durumunu sıfırlamak (`Reset`) için bir metot içerir.
*   **Enum Tanımları**:
    *   `EMountUpGradeChatType`: Binek yükseltme sistemiyle ilgili sohbet mesajı türlerini tanımlar (örneğin, binek üzerindeyken engelleme, yeterli Yang/yem olmaması, başarılı/başarısız yükseltme yüzdesi, yem ile EXP kazanımı).
    *   `EMountUpGradeChatValue`: Sohbet mesajları için ek bir değer (şu anda sadece `CHAT_TYPE_NO_VALUE`).
    *   `EMountUpGradeReset`: Sıfırlama işlemi için bir sabit (`RESET`).
    *   `EMountUpGradeItem`: Binek yemi eşya ID'si (`HORSE_FEED_ITEM_ID`), yem başına kazanılacak EXP miktarı (`HORSE_FEED_EXP_COUNT`) ve seviye atlamak için gereken yem sayısı (`HORSE_FEED_LEVEL_COUNT`) gibi eşya ile ilgili sabitleri tanımlar.
    *   `EMountUpGradeRetryGemCost`: Yükseltme başarısız olduğunda tekrar deneme için gereken Won (gem) miktarını binek seviyesine göre belirleyen sabitler.
    *   `EMountUpGradeTooltip`: Yükseltme arayüzündeki bilgi baloncuğunda gösterilecek maliyet türünü (Yang veya Won) belirtir.
    *   `EMountUpGradeFailType`: Yükseltmenin başarısız olup olmadığını belirten durumlar (`MOUNT_UP_GRADE_FAIL_OFF`, `MOUNT_UP_GRADE_FAIL_ON`).
    *   `EMountUpGradeGCSubheaderType`: Sunucudan istemciye gönderilen binek yükseltme paketi alt başlıklarını tanımlar (pencere açma, yenileme).
    *   `EMountUpGradeCGSubheaderType`: İstemciden sunucuya gönderilen binek yükseltme paketi alt başlıklarını tanımlar (EXP verme, seviye atlatma).
*   **Global Sabit Diziler**:
    *   `mount_up_grade_exp_table[HORSE_MAX_LEVEL + 1]`: Her bir binek seviyesi için bir sonraki seviyeye geçmek için gereken toplam deneyim puanını tutan bir dizi.
    *   `mount_up_grade_price_table[HORSE_MAX_LEVEL + 1]`: Her bir binek seviyesi için seviye atlama ücretini (Yang) tutan bir dizi.

#### C++ ve Python Arayüzü ile İlişkisi

*   `CPythonMountUpGrade` sınıfı, binek yükseltme verilerini ve mantığını C++ tarafında yönetir.
*   `m_window` üye değişkeni aracılığıyla Python'da oluşturulmuş bir UI penceresi ile etkileşim kurar (`PyCallClassMemberFunc` kullanılır).
*   `CPythonMountUpGrade.cpp` dosyasında, bu sınıfın işlevlerini Python betiklerine sunan `mupgrd` adında bir Python modülü tanımlanır. Bu modül, Python tabanlı UI betiklerinin binek yükseltme verilerine erişmesini, sunucuya istek göndermesini ve sistem durumunu yönetmesini sağlar.
*   Enum'larda tanımlanan sabitler, Python modülüne de aktarılarak hem C++ hem de Python tarafında tutarlı bir şekilde kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Binek Seviye Atlama Arayüzü**: Oyuncunun bineklerini geliştirebileceği bir arayüzün (genellikle bir NPC veya özel bir pencere aracılığıyla) arka plan mantığını yönetir.
*   **EXP ve Ücret Hesaplama**: Bineğin mevcut seviyesine göre bir sonraki seviye için ne kadar EXP gerektiğini ve seviye atlama işleminin ne kadar Yang/Won tutacağını hesaplar.
*   **Yem Kullanımı**: Oyuncuların binek yemi kullanarak bineklerine EXP kazandırmasını sağlar.
*   **Sunucu İletişimi**: Binek EXP verme veya seviye atlatma gibi istekleri `CPythonNetworkStream` aracılığıyla sunucuya gönderir.
*   **UI Güncelleme ve Geri Bildirim**: Sunucudan gelen yanıtlara veya oyun içi olaylara göre (örneğin, seviye atlama başarılı/başarısız, yeterli malzeme yok) UI'ı günceller ve oyuncuya sohbet mesajları aracılığıyla bilgi verir.

Bu başlık dosyası, gelişmiş binek yükseltme sisteminin C++ tarafındaki temel veri yapılarını, arayüzlerini ve sabitlerini tanımlayarak sistemin düzgün çalışması için bir çerçeve oluşturur.

---

### `PythonMountUpGrade.cpp` (Gelişmiş Binek Yükseltme Sistemi Implementasyonu ve `mupgrd` Modülü)

Bu C++ kaynak dosyası, `ENABLE_RIDING_EXTENDED` makrosu etkinleştirildiğinde, `PythonMountUpGrade.h` dosyasında bildirilen `CPythonMountUpGrade` singleton C++ sınıfının tüm metotlarının C++ implementasyonlarını ve bu sınıfın işlevlerini Python betiklerine sunan `mupgrd` adlı bir Python modülünü içerir.

#### `CPythonMountUpGrade` Sınıfı Implementasyon Detayları

*   **Yapıcı ve Yıkıcı (`CPythonMountUpGrade`, `~CPythonMountUpGrade`)**:
    *   Her ikisi de `Reset()` metodunu çağırarak sınıfın üye değişkenlerini başlangıç durumuna getirir.
*   **`Reset()`**:
    *   `m_HorseLevel` (binek seviyesi), `m_MountUpGradeFail` (yükseltme başarısızlık durumu) ve `m_MountUpGradeExistingExp` (mevcut binek EXP'si) değişkenlerini `CPythonMountUpGrade::EMountUpGradeReset::RESET` (genellikle 0) değerine sıfırlar.
*   **Pencere Yönetimi (`SetWindow`, `DestroyWindow`, `Refresh`)**:
    *   `SetWindow(PyObject* ppyObject)`: Python'dan gelen UI pencere nesnesini `m_window` üye değişkenine atar.
    *   `DestroyWindow()`: `m_window` işaretçisini `nullptr` yaparak Python penceresiyle olan bağlantıyı koparır.
    *   `Refresh()`: `m_window`'a bağlı Python penceresinin `Refresh` adlı bir metodunu çağırır (`PyCallClassMemberFunc`). Bu, UI'ın güncellenmesini tetikler.
*   **Sohbet Mesajı (`Chat`)**:
    *   `Chat(const uint8_t type, const uint8_t value)`: Verilen `type` (mesaj türü) ve `value` (ek değer) ile `m_window`'a bağlı Python penceresinin `Chat` adlı bir metodunu çağırır. Eğer `type` geçersizse hata mesajı basar.
*   **Veri Ayarlama ve Alma Metotları**:
    *   `SetHorseLevel(const uint8_t level)`: `m_HorseLevel`'ı ayarlar.
    *   `GetHorseLevel() const`: `m_HorseLevel`'ı döndürür.
    *   `SetMountUpGradeFail(const uint8_t fail)`: `m_MountUpGradeFail` durumunu ayarlar.
    *   `IsMountUpGradeFail() const`: `m_MountUpGradeFail` durumunu döndürür.
    *   `SetMountUpGradeExistingExp(const uint32_t exp)`: `m_MountUpGradeExistingExp`'i ayarlar.
    *   `GetMountUpGradeExistingExp() const`: `m_MountUpGradeExistingExp`'i döndürür.
*   **Hesaplama Metotları**:
    *   `GetMountUpGradeNecessaryExp() const`: `PythonMountUpGrade.h` dosyasındaki `mount_up_grade_exp_table` dizisini kullanarak mevcut binek seviyesinin (`GetHorseLevel() + 1`) bir sonraki seviye için gerektirdiği toplam EXP miktarını döndürür.
    *   `GetMountUpGradePrice() const`: `PythonMountUpGrade.h` dosyasındaki `mount_up_grade_price_table` dizisini kullanarak mevcut binek seviyesinin (`GetHorseLevel() + 1`) seviye atlama ücretini döndürür.

#### `mupgrd` Python Modülü

Bu bölüm, `CPythonMountUpGrade` sınıfının C++ fonksiyonlarını Python betiklerinin kullanabileceği şekilde sarmalayan C fonksiyonlarını ve modül başlatma fonksiyonunu (`initmountupgrade`) içerir.

*   **Python'a Açılan Fonksiyonlar**:
    *   `SetWindow(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir UI nesnesi alır ve `CPythonMountUpGrade::Instance().SetWindow()` ile C++ tarafına kaydeder.
    *   `DestroyWindow(PyObject* poSelf, PyObject* poArgs)`: `CPythonMountUpGrade::Instance().DestroyWindow()` çağırır.
    *   `GetHorseLevel(PyObject* poSelf, PyObject* poArgs)`: `CPythonMountUpGrade::Instance().GetHorseLevel()` sonucunu Python integer olarak döndürür.
    *   `GetMountUpGradePrice(PyObject* poSelf, PyObject* poArgs)`: `CPythonMountUpGrade::Instance().GetMountUpGradePrice()` sonucunu Python integer olarak döndürür.
    *   `IsMountUpGradeFail(PyObject* poSelf, PyObject* poArgs)`: `CPythonMountUpGrade::Instance().IsMountUpGradeFail()` sonucunu Python integer (boolean gibi) olarak döndürür.
    *   `GetMountExistingExp(PyObject* poSelf, PyObject* poArgs)` (Python'daki adı `GetMountUpGradeExistingExp`'ten farklı): `CPythonMountUpGrade::Instance().GetMountUpGradeExistingExp()` sonucunu Python integer olarak döndürür.
    *   `GetMountNecessaryExp(PyObject* poSelf, PyObject* poArgs)` (Python'daki adı `GetMountUpGradeNecessaryExp`'ten farklı): `CPythonMountUpGrade::Instance().GetMountUpGradeNecessaryExp()` sonucunu Python integer olarak döndürür.
    *   `Reset(PyObject* poSelf, PyObject* poArgs)`: `CPythonMountUpGrade::Instance().Reset()` çağırır.
    *   `Send(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir alt başlık (`iSubHeader`) alır ve `CPythonNetworkStream::Instance().MountUpGrade(iSubHeader)` ile sunucuya binek yükseltme isteği gönderir.

*   **`initmountupgrade()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `mupgrd` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, yukarıda bahsedilen C fonksiyonlarını Python isimleriyle (`"Send"`, `"Reset"`, `"GetHorseLevel"` vb.) eşleştirir.
    *   `Py_InitModule("mupgrd", s_methods)` ile `mupgrd` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak `PythonMountUpGrade.h` dosyasında tanımlanan çeşitli enum sabitleri (örneğin, `EMountUpGradeCGSubheaderType`, `EMountUpGradeChatType`, `EMountUpGradeItem` içindeki değerler) Python modülüne (`mupgrd.MOUNT_UP_GRADE_EXP`, `mupgrd.CHAT_TYPE_BANN_WHILE_MOUNTING`, `mupgrd.HORSE_FEED_ITEM_ID` vb.) aktarılır. Bu sayede Python betikleri bu sabitleri doğrudan kullanabilir.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **UI-Mantık Bağlantısı**: Python'da yazılmış binek geliştirme arayüzü (penceresi), `mupgrd` modülü aracılığıyla C++ tarafındaki binek verilerine (seviye, EXP, maliyet) erişir ve bunları kullanıcıya gösterir.
*   **Kullanıcı Etkileşimleri**: Kullanıcı arayüzdeki butonlara tıkladığında (örneğin, "EXP Ver", "Seviye Atlat"), Python betikleri `mupgrd.Send()` fonksiyonunu çağırarak ilgili isteği sunucuya iletir.
*   **Durum Güncellemeleri**: Sunucudan gelen binek bilgileri (örneğin, yeni seviye, güncel EXP) C++ tarafındaki `CPythonMountUpGrade` nesnesini günceller. Ardından `CPythonMountUpGrade::Refresh()` veya `CPythonMountUpGrade::Chat()` çağrılarak UI'ın da bu değişiklikleri yansıtması sağlanır.
*   **Sabit Değer Erişimi**: Python betikleri, binek yemi ID'si, sohbet mesajı türleri gibi sabit değerlere `mupgrd` modülü üzerinden erişerek C++ tarafıyla tutarlı bir şekilde çalışır.

Bu dosya, gelişmiş binek yükseltme sisteminin C++ mantığı ile Python UI arasındaki köprüyü kurarak, kullanıcı arayüzünün binek geliştirme işlemlerini doğru bir şekilde yönetmesini ve sunucuyla iletişim kurmasını sağlar.

---

### `PythonNetworkDatagram.h` (UDP Ağ İletişim Sistemi Başlık Dosyası - Yorum Satırında)

**Uyarı: Bu dosyadaki tüm içerik yorum satırı (`/* ... */`) içindedir. Bu, UDP tabanlı ağ iletişim sisteminin mevcut kod tabanında devre dışı bırakılmış veya kullanımdan kaldırılmış olabileceğini gösterir.**

Bu başlık dosyası, normalde `CPythonNetworkDatagram` adlı bir singleton C++ sınıfını tanımlardı. Bu sınıf, User Datagram Protocol (UDP) üzerinden sunucu ve diğer istemcilerle (P2P benzeri) iletişimi yönetmek için tasarlanmış gibi görünmektedir. Temel amacı, özellikle karakter hareketleri gibi sık güncellenen ancak kayıp toleransı olan verilerin gönderilip alınmasını sağlamaktır.

#### Dosyanın (Yorum İçindeki) Genel Amaçları

*   **`CPythonNetworkDatagram` Sınıf Bildirimi**:
    *   `CSingleton<CPythonNetworkDatagram>`'den miras alırdı.
    *   Sunucu ile UDP bağlantısı kurma, veri gönderme (`SendToServer`) ve alma (`m_NetReceiver`) işlevlerini içerirdi.
    *   Diğer istemcilerle (UDP göndericileri - `CNetDatagramSender`) iletişim kurmak için gönderici kaydetme (`RegisterSender`), silme (`DeleteSender`), seçme (`Select`) ve seçilenlere veri gönderme (`SendToSenders`) mekanizmalarını barındırırdı.
    *   Gelen UDP paketlerini işlemek için bir `Process()` döngüsü ve belirli paket türlerini (örneğin, `HEADER_CC_STATE_WALKING`) ayrıştırmak için metotlar (`CheckPacket`, `RecvStateWalkingPacket`) içerirdi.
    *   Karakter durum paketlerini (örneğin, yürüme) göndermek için `SendCharacterStatePacket` gibi özel bir metot sunardı.
*   **Veri Yapıları ve Tür Tanımları**:
    *   `TNetSenderMap`: DWORD (ID) ile `CNetDatagramSender` işaretçilerini eşleyen bir `std::map`.
    *   `TNetSenderList`: `CNetDatagramSender` işaretçilerinden oluşan bir `std::list`.
    *   `m_NetSenderMap`: Kayıtlı UDP göndericilerini tutardı.
    *   `m_NetSenderList`: `SendToSenders` için geçici olarak seçilen göndericileri tutardı.
    *   `m_NetSender`: Sunucuya veri göndermek için kullanılan `CNetDatagramSender` nesnesi.
    *   `m_NetReceiver`: Sunucudan veya diğer istemcilerden veri almak için kullanılan `CNetDatagramReceiver` nesnesi.
    *   `m_NetSenderPool`: `CNetDatagramSender` nesneleri için bir dinamik bellek havuzu (`CDynamicPool`).
*   **Bağımlılıklar**:
    *   `../EterLib/NetDatagramReceiver.h` ve `../EterLib/NetDatagramSender.h`: EterLib içindeki UDP alıcı ve gönderici sınıfları.
    *   `Packet.h`: Ağ paket başlıklarını ve yapılarını tanımlayan dosya.

#### (Yorum İçindeki) C++ ve Python Arayüzü ile İlişkisi

*   `CPythonNetworkDatagram` sınıfı, UDP ağ iletişiminin C++ mantığını yönetirdi.
*   `PythonNetworkDatagramModule.cpp` dosyasında (o da tamamen yorum satırında), bu sınıfın bazı işlevlerini (muhtemelen sistemi etkinleştirme/devre dışı bırakma) Python'a açan `udp` adında bir Python modülü tanımlanmaya çalışılmış.

#### (Yorum İçindeki) Oyunda Kullanım Amacı ve Senaryoları

*   **Hızlı Durum Güncellemeleri**: Karakterlerin yürüme gibi sık değişen durum bilgilerini TCP yerine UDP üzerinden göndererek ağ yükünü azaltmayı ve daha akıcı bir hareket senkronizasyonu sağlamayı hedeflerdi. UDP'nin bağlantısız yapısı ve paket kaybı olasılığı, bu tür "önemli ama kritik olmayan" veriler için kabul edilebilir olabilirdi.
*   **P2P Benzeri İletişim (Potansiyel)**: `RegisterSender`, `SendToSenders` gibi fonksiyonlar, istemcilerin doğrudan birbirleriyle (sunucu aracılığıyla değil) UDP üzerinden iletişim kurabileceği bir yapıyı işaret ediyor olabilir. Bu, örneğin, yakındaki oyuncular arasında konum bilgilerinin paylaşılması için kullanılabilirdi.
*   **Paket İşleme**: Gelen UDP paketlerini `Process()` içinde sürekli kontrol eder, bilinen paket başlıklarına göre ilgili işleme fonksiyonunu çağırırdı.

Bu dosyadaki yorumlanmış kod, oyun için alternatif bir ağ iletişim katmanı (UDP) denemesini veya planını göstermektedir. Ancak, tamamen yorum satırında olması, bu sistemin ya tamamlanmadığını, ya performans/güvenilirlik sorunları nedeniyle terk edildiğini ya da belirli bir derleme yapılandırmasıyla aktif hale getirilmek üzere bekletildiğini düşündürmektedir. Özellikle `SendCharacterStatePacket` içindeki `//SendToSenders(...)` satırı ve `SendToSenders` içindeki geçici kod notu, sistemin tam olarak entegre edilmediğine işaret eder.

---

### `PythonNetworkDatagram.cpp` (UDP Ağ İletişim Sistemi Implementasyonu - Yorum Satırında)

**Uyarı: Bu dosyadaki tüm içerik yorum satırı (`/* ... */`) içindedir. Bu, UDP tabanlı ağ iletişim sisteminin mevcut kod tabanında devre dışı bırakılmış veya kullanımdan kaldırılmış olabileceğini gösterir.**

Bu C++ kaynak dosyası, normalde `PythonNetworkDatagram.h` dosyasında bildirilen (ancak orada da yorum satırında olan) `CPythonNetworkDatagram` singleton C++ sınıfının metotlarının implementasyonlarını içerirdi. Temel amacı, UDP üzerinden ağ paketlerini gönderme, alma ve işleme mantığını gerçekleştirmektir.

#### (Yorum İçindeki) `CPythonNetworkDatagram` Sınıfı Implementasyon Detayları

*   **`CDatagramPacketHeaderMap` Sınıfı**:
    *   `CNetworkPacketHeaderMap`'ten türeyen özel bir iç sınıf.
    *   UDP için beklenen paket başlıklarını ve boyutlarını kaydetmek için kullanılırdı. Örnekte sadece `HEADER_CC_STATE_WALKING` için bir kayıt bulunmaktadır.
*   **`Destroy()`**:
    *   `m_NetSenderPool` (UDP göndericileri için bellek havuzu) temizlenirdi.
*   **Paket Kontrolü (`CheckPacket`)**:
    *   `m_NetReceiver` (UDP alıcısı) bağlı olup olmadığını kontrol ederdi.
    *   Gelen veriden paket başlığını okur (`Peek`) ve `CDatagramPacketHeaderMap` kullanarak bilinen bir paket olup olmadığını doğrular.
*   **Paket İşleme (`Process`)**:
    *   `m_NetReceiver.Process()` döngüsü içinde gelen UDP paketlerini sürekli kontrol ederdi.
    *   `CheckPacket` ile geçerli bir paket başlığı alınırsa, `switch` yapısı ile paket türüne göre ilgili işleme fonksiyonunu (örneğin, `HEADER_CC_STATE_WALKING` için `RecvStateWalkingPacket`) çağırırdı. Diğer bazı başlıklar (örneğin, `HEADER_CC_STATE_WAITING`, `HEADER_CC_EVENT_NORMAL_ATTACKING`) listelenmiş ancak işlenmemiş olarak bırakılmış.
*   **Sunucu Bağlantı Ayarları**:
    *   `SetConnection(const char* c_szIP, WORD wPortIndex)`: `m_NetSender` (sunucuya UDP göndericisi) için hedef IP ve portu ayarlardı.
    *   `SetRecvBufferSize(DWORD dwSize)`: `m_NetReceiver` için alım tampon boyutunu ayarlardı.
    *   `SendToServer(const void* c_pBuffer, DWORD dwSize)`: `m_NetSender` üzerinden sunucuya veri gönderirdi.
    *   `Bind(DWORD dwAddress, WORD wPortIndex)`: `m_NetReceiver`'ı belirtilen adres ve porta bağlayarak veri almaya hazır hale getirirdi.
*   **Diğer İstemcilerle (Sender) İletişim**:
    *   `RegisterSender(DWORD dwID, DWORD dwAddress, WORD wPortIndex)`: Yeni bir `CNetDatagramSender` nesnesini `m_NetSenderPool`'dan alır, verilen adres ve porta ayarlar ve `dwID` ile `m_NetSenderMap`'e kaydederdi.
    *   `DeleteSender(DWORD dwID)`: Belirtilen ID'ye sahip göndericiyi `m_NetSenderMap`'ten siler ve nesneyi `m_NetSenderPool`'a geri verirdi.
    *   `Select(DWORD dwID)`: Belirtilen ID'ye sahip göndericiyi `m_NetSenderList`'e eklerdi (çoklu gönderim için).
    *   `SendToSenders(const void* c_pBuffer, DWORD dwSize)`: `m_NetSenderList`'teki tüm göndericilere (veya geçici bir not olarak belirtildiği gibi, `m_NetSenderMap`'teki tüm kayıtlı göndericilere) verilen tamponu gönderir ve ardından listeyi temizlerdi.
    *   `GetSenderPointer(DWORD dwID, CNetDatagramSender** ppSender)`: Verilen ID'ye sahip göndericinin işaretçisini `m_NetSenderMap`'ten alırdı.
*   **Karakter Durum Paketi İşlemleri**:
    *   `SendCharacterStatePacket(...)`: Verilen karakter ID'si (VID), komut zamanı, hedef pozisyon, rotasyon ve fonksiyon/argüman bilgileriyle bir `TPacketCCState` (yürüme durumu paketi) oluştururdu. Bu paket normalde `SendToSenders` ile gönderilmek üzere tasarlanmış gibi görünse de, ilgili satır yorum içindeydi.
    *   `RecvStateWalkingPacket()`: `m_NetReceiver`'dan bir `TPacketCCState` paketi okurdu. Paket içindeki VID ile ilgili karakter örneğini (`CInstanceBase`) `CPythonCharacterManager`'dan alır ve karakterin UDP durumunu (`PushUDPState`) güncellerdi. Bu, karakterin pozisyonunu, rotasyonunu ve animasyon durumunu senkronize ederdi.
*   **Yapıcı ve Yıkıcı (`CPythonNetworkDatagram`, `~CPythonNetworkDatagram`)**:
    *   Boş implementasyonlara sahipti.

#### (Yorum İçindeki) Oyunda Kullanım Amacı ve Senaryoları

*   **Hareket Verisi Aktarımı**: En belirgin kullanım alanı, karakterlerin yürüme (`HEADER_CC_STATE_WALKING`) gibi hareket verilerinin UDP üzerinden diğer istemcilere veya sunucuya iletilmesiydi. Bu, TCP'ye göre daha düşük gecikme sağlayabilirdi.
*   **İstemciler Arası Eş Zamanlama**: `SendToSenders` mekanizması, bir istemcinin kendi hareket veya durum bilgisini çevresindeki diğer istemcilere doğrudan (veya sunucu üzerinden yönlendirilerek) göndermesini sağlayarak, oyuncuların birbirlerinin hareketlerini daha pürüzsüz görmesine yardımcı olabilirdi.
*   **UDP Paket Ayrıştırma**: Sistem, gelen UDP verilerini alır, bilinen paket başlıklarına göre ayırır ve oyun mantığını buna göre güncellerdi (örneğin, başka bir karakterin yeni pozisyonunu işlemek).

Dosyanın tamamen yorum satırında olması, bu UDP tabanlı iletişim altyapısının geliştirme aşamasında kalmış, test edilmiş ancak son sürüme dahil edilmemiş veya başka bir sistemle değiştirilmiş olabileceğini düşündürmektedir. Özellikle `SendCharacterStatePacket` içindeki `//SendToSenders(...)` satırı ve `SendToSenders` içindeki geçici kod notu, sistemin tam olarak entegre edilmediğine işaret eder.

---

### `PythonNetworkDatagramModule.cpp` (UDP Ağ İletişim Python Modülü - Yorum Satırında)

**Uyarı: Bu dosyadaki tüm içerik yorum satırı (`/* ... */`) içindedir. Bu, UDP tabanlı ağ iletişim sisteminin Python arayüzünün mevcut kod tabanında devre dışı bırakılmış veya kullanımdan kaldırılmış olabileceğini gösterir.**

Bu C++ kaynak dosyası, normalde `CPythonNetworkDatagram` C++ sınıfının bazı temel işlevlerini Python betiklerine sunacak olan `udp` adlı bir Python modülünü tanımlardı. Ancak, dosyadaki ilgili kod blokları tamamen yorum satırı içine alınmıştır.

#### (Yorum İçindeki) Dosyanın Genel Amaçları

*   **Python Modülü Tanımlama (`initudp`)**:
    *   `udp` adında bir Python modülü oluşturmayı hedeflerdi.
    *   Bu modül, `CPythonNetworkDatagram` sınıfının işlevlerine Python'dan erişim sağlamak için C fonksiyonlarını (wrapper) içerirdi.
*   **Python'a Fonksiyon Aktarma**:
    *   `udpEnable(PyObject* poSelf, PyObject* poArgs)`: `CPythonNetworkDatagram::Instance().Enable()` fonksiyonunu çağırarak UDP sistemini Python'dan etkinleştirmeyi amaçlardı. (Not: `Enable()` ve `Disable()` fonksiyonları `CPythonNetworkDatagram` sınıf tanımında bulunmuyor, bu da modülün sınıf tanımıyla tam senkronize olmadığını veya eksik olduğunu gösterebilir.)
    *   `udpDisable(PyObject* poSelf, PyObject* poArgs)`: `CPythonNetworkDatagram::Instance().Disable()` fonksiyonunu çağırarak UDP sistemini Python'dan devre dışı bırakmayı amaçlardı.

#### (Yorum İçindeki) Python Modülü (`udp`) Fonksiyonları

*   `udp.Enable()`: C++ tarafındaki UDP iletişimini (muhtemelen paket alımını ve gönderimini) başlatırdı.
*   `udp.Disable()`: C++ tarafındaki UDP iletişimini durdururdu.

#### (Yorum İçindeki) Oyunda Kullanım Amacı ve Senaryoları

*   **Dinamik Kontrol**: Eğer UDP sistemi opsiyonel veya test aşamasında bir özellikse, Python betikleri aracılığıyla oyun içinden veya başlangıç yapılandırmasında bu sistemin dinamik olarak etkinleştirilip devre dışı bırakılmasına olanak tanıyabilirdi.
*   **Hata Ayıklama/Test**: Geliştiricilerin test senaryolarında UDP iletişimini kolayca açıp kapatmalarını sağlayabilirdi.

Dosyanın tamamının yorum satırında olması, `CPythonNetworkDatagram` sistemiyle birlikte bu Python arayüzünün de ya hiç tamamlanmadığını, ya kullanımdan kaldırıldığını ya da gelecekteki bir entegrasyon için bekletildiğini güçlü bir şekilde göstermektedir. `CPythonNetworkDatagram` sınıfında `Enable`/`Disable` metotlarının eksikliği de bu durumu destekler niteliktedir.

---

### `PythonNetworkStream.h` (Ağ Akışı Yönetimi Başlık Dosyası)

Bu başlık dosyası, Metin2 istemcisinin sunucu ile olan temel ağ iletişimini yöneten `CPythonNetworkStream` sınıfını tanımlar. Bu sınıf, `CNetworkStream` sınıfından türetilmiştir ve bir singleton olarak tasarlanmıştır, yani oyun boyunca sadece bir örneği bulunur. Oyunun farklı aşamaları (login, select, game vb.) arasındaki geçişleri, paket gönderimini/alımını ve ağ ile ilgili çeşitli yardımcı fonksiyonları yönetir.

**Temel İşlevleri ve Özellikleri:**

*   **Ağ Bağlantısı Yönetimi:** Login sunucusuna, oyun sunucusuna ve mark (lonca sembolü) sunucusuna bağlanma işlemlerini yönetir.
*   **Paket İşleme:** Sunucudan gelen paketleri alır, türlerine göre ayrıştırır ve ilgili işleyicilere yönlendirir. Ayrıca istemciden sunucuya paket gönderimini de sağlar.
*   **Oyun Aşamaları (Phases):** Oyunun farklı durumlarını (örneğin, "OffLine", "Login", "Select", "Loading", "Game") yönetir ve bu aşamalara özgü işlemleri tetikler. Her aşama için özel `PyObject` pencere nesneleri tutar.
*   **Veri Gönderimi (Sending Packets):** Karakter hareketleri, saldırılar, yetenek kullanımları, envanter işlemleri, sohbet mesajları, görev etkileşimleri, ticaret, lonca işlemleri gibi çok sayıda oyun içi eylemi sunucuya iletmek için özel paket gönderme fonksiyonları içerir.
*   **Veri Alımı (Receiving Packets):** Sunucudan gelen ve oyun durumunu güncelleyen (karakter bilgileri, eşya güncellemeleri, diğer oyuncuların hareketleri, sohbet mesajları vb.) çeşitli paketleri işlemek için `Recv*` ön ekli fonksiyonlar barındırır.
*   **Karakter ve Hesap Bilgileri:** Oyuncunun hesap ve karakter bilgilerini (slot bilgileri, ID, isim, seviye, statüler vb.) saklar ve Python tarafına iletir.
*   **Lonca Sembolü (Mark) Yönetimi:** Lonca sembollerinin yüklenmesi ve indirilmesi ile ilgili işlemleri koordine eder.
*   **Yardımcı Fonksiyonlar:** Küfür filtresi, dil çeviri tabloları, hata yönetimi, ping gönderimi gibi çeşitli yardımcı işlevler sunar.
*   **Python Entegrasyonu:** C++ tarafındaki ağ olaylarını ve verilerini Python tarafındaki UI ve oyun mantığına iletmek için `PyObject` işaretçileri ve Python çağrıları kullanır. Özellikle `m_poHandler` (genel olay işleyici) ve `m_apoPhaseWnd` (aşama pencereleri) bu entegrasyonda önemli rol oynar.
*   **Ön Ekli Sistemler Desteği:** `ENABLE_` ile başlayan çok sayıda makro aracılığıyla oyuna eklenmiş çeşitli sistemlere (örneğin, `ENABLE_ACCE_COSTUME_SYSTEM`, `ENABLE_GROWTH_PET_SYSTEM`, `ENABLE_MAILBOX`, `ENABLE_PREMIUM_PRIVATE_SHOP` vb.) ait paket gönderme ve alma fonksiyonlarını içerir.

**Önemli Üye Değişkenler (Bazıları):**

*   `m_strPhase`: Mevcut oyun aşamasını tutan string (örneğin, "Game", "Login").
*   `m_poHandler`: Python tarafındaki ana olay işleyici nesnesine işaretçi.
*   `m_apoPhaseWnd[PHASE_WINDOW_NUM]`: Her oyun aşaması için Python pencere nesnelerine işaretçiler dizisi.
*   `m_akSimplePlayerInfo`: Hesaptaki karakterlerin temel bilgilerini tutan dizi.
*   `m_dwMainActorVID`, `m_dwMainActorRace`, `m_dwMainActorEmpire`, `m_dwMainActorSkillGroup`: Ana oyuncu karakterinin VID'i, ırkı, imparatorluğu ve yetenek grubu.
*   `m_dwGuildID`, `m_dwEmpireID`: Oyuncunun lonca ID'si ve imparatorluk ID'si.
*   `m_kMarkAuth`: Lonca sembolü sunucusu için kimlik doğrulama bilgileri.
*   `m_kInsultChecker`: Küfür filtresi nesnesi.
*   `m_phaseProcessFunc`, `m_phaseLeaveFunc`: Mevcut aşamanın işleme ve ayrılma fonksiyonlarına işaret eden fonksiyon nesneleri.

**Önemli Fonksiyon Grupları:**

*   `Set[PhaseName]Phase()`: Belirli bir oyun aşamasını başlatır (örneğin, `SetLoginPhase()`, `SetGamePhase()`).
*   `Send*Packet()`: Sunucuya belirli bir türde paket gönderir (örneğin, `SendChatPacket()`, `SendItemUsePacket()`).
*   `Recv*Packet()`: Sunucudan gelen belirli bir türde paketi işler (örneğin, `RecvChatPacket()`, `RecvItemSetPacket()`). Bu fonksiyonlar genellikle `protected` olarak tanımlanır ve içsel paket işleme mantığını içerir.
*   `GetAccountCharacterSlotDatau()`, `GetAccountCharacterSlotDataz()`: Karakter slot bilgilerini Python tarafına iletmek için kullanılır.
*   `ConnectLoginServer()`, `ConnectGameServer()`: Sunuculara bağlantı kurar.

Bu başlık dosyası, istemcinin sunucuyla etkileşiminin bel kemiğini oluşturur ve oyunun dinamik olarak güncellenmesini, oyuncu eylemlerinin sunucuya iletilmesini ve sunucudan gelen bilgilerin oyuna yansıtılmasını sağlar.

---

### `PythonNetworkStream.cpp` (Ağ Akışı Yönetimi Implementasyonu)

Bu dosya, `PythonNetworkStream.h` içinde tanımlanan `CPythonNetworkStream` sınıfının ve yardımcı sınıf olan `CMainPacketHeaderMap`'in implementasyonunu içerir. İstemcinin sunucu ile iletişim kurmasını sağlayan temel mantığı barındırır. Paketlerin gönderilmesi, alınması, işlenmesi ve oyunun farklı aşamaları arasındaki geçişler burada yönetilir.

**Ana Çalışma Prensipleri:**

1.  **Paket Başlık Haritası (`CMainPacketHeaderMap`):**
    *   Bu iç sınıf, sunucudan gelen paket başlıklarını (header) ve bu başlıklara karşılık gelen paket yapılarını (boyut ve dinamik/statik olma durumu) eşleştirir.
    *   Yapılandırıcısında (`CMainPacketHeaderMap()`), `Set()` fonksiyonu aracılığıyla bilinen tüm sunucu-istemci (GC) paket başlıkları ve karşılık gelen `TPacketType` bilgileri tanımlanır. Bu, gelen verinin doğru şekilde okunup yorumlanabilmesi için kritik öneme sahiptir.
    *   `STATIC_SIZE_PACKET` ve `DYNAMIC_SIZE_PACKET` enumları, paketlerin sabit mi yoksa değişken boyutlu mu olduğunu belirtir. Dinamik boyutlu paketler genellikle bir boyut bilgisi içeren ek bir başlık alanına sahiptir.

2.  **Aşama Yönetimi (Phase Management):**
    *   `CPythonNetworkStream`, oyunun farklı durumlarını (örneğin, Offline, Handshake, Login, Select, Loading, Game) `m_strPhase` değişkeninde saklar.
    *   Her aşama için bir "process" fonksiyonu (`m_phaseProcessFunc`) ve bir "leave" fonksiyonu (`m_phaseLeaveFunc`) bulunur. Bu fonksiyonlar, `CFuncObject` kullanılarak atanır.
    *   `SetOffLinePhase()`, `SetHandShakePhase()`, `SetLoginPhase()`, `SetSelectPhase()`, `SetLoadingPhase()`, `SetGamePhase()` gibi fonksiyonlar, ilgili aşamaya geçişi sağlar, `m_strPhase`'i günceller ve uygun process/leave fonksiyonlarını ayarlar.
    *   `RecvPhasePacket()` fonksiyonu, sunucudan `HEADER_GC_PHASE` paketi geldiğinde çağrılır ve `packet_phase.phase` değerine göre ilgili aşama ayarlama fonksiyonunu tetikler.

3.  **Paket Gönderme:**
    *   `Send*Packet()` genel adlandırma kuralına sahip çok sayıda fonksiyon (örneğin, `SendChatPacket()`, `SendItemUsePacket()`, `SendAttackPacket()`) bulunur.
    *   Bu fonksiyonlar, C++ tarafında ilgili paket yapısını (örneğin, `TPacketCGText`, `TPacketCGItemUse`) doldurur ve ardından `CNetworkStream::Send()` metodunu çağırarak veriyi sunucuya gönderir.
    *   Bazı gönderme işlemleri doğrudan Python'dan tetiklenebilir (örneğin, Python UI üzerinden bir butona tıklama).

4.  **Paket Alma ve İşleme:**
    *   `OnProcess()` ana döngüde çağrılır ve `m_phaseProcessFunc.Run()` ile mevcut aşamanın paket işleme fonksiyonunu çalıştırır.
    *   Her aşamanın kendi process fonksiyonu (örneğin, `LoginPhase()`, `GamePhase()`) `CheckPacket()`'i çağırarak gelen veriyi kontrol eder.
    *   `CheckPacket()`:
        *   Gelen verinin en az bir paket başlığı kadar olup olmadığını kontrol eder (`Peek()`).
        *   Başlığı okur ve `CMainPacketHeaderMap` kullanarak paketin türünü (boyut, dinamik/statik) belirler.
        *   Paketin tamamının alınıp alınmadığını kontrol eder.
        *   Başarılı olursa, alınan paket başlığını `pRetHeader` ile döndürür.
    *   Aşama process fonksiyonları, `CheckPacket()`'ten dönen başlığa göre bir `switch-case` yapısı veya `if-else if` blokları kullanarak ilgili `Recv*Packet()` fonksiyonunu çağırır.
    *   `Recv*Packet()` fonksiyonları (örneğin, `RecvChatPacket()`, `RecvItemSetPacket()`):
        *   `Recv()` metodunu kullanarak paketin geri kalanını okur.
        *   Paket içeriğini yorumlar.
        *   Gerekirse Python tarafındaki UI veya oyun mantığını güncellemek için `PyCallClassMemberFunc()` ile `m_poHandler` veya ilgili aşama penceresi (`m_apoPhaseWnd`) üzerinden Python fonksiyonlarını çağırır.
        *   Oyun içi durumları günceller (örneğin, envanter, karakter statüleri).

**Önemli Fonksiyonlar ve Mekanizmalar:**

*   **`CPythonNetworkStream::ConnectLoginServer()`, `CPythonNetworkStream::ConnectGameServer()`:** İlgili sunuculara bağlantı kurar.
*   **`CPythonNetworkStream::SetLoginInfo()`, `CPythonNetworkStream::SetLoginKey()`:** Login bilgileri ve anahtarını saklar.
*   **`CPythonNetworkStream::CheckPacket()`:** Gelen veriyi analiz ederek geçerli bir paket olup olmadığını ve türünü belirler.
*   **`CPythonNetworkStream::RecvPhasePacket()`:** Oyun aşaması değişim paketlerini işler.
*   **`CPythonNetworkStream::RecvPingPacket()`:** Ping paketlerini işler ve pong paketi gönderir.
*   **`CPythonNetworkStream::OnProcess()`:** Ana ağ işleme döngüsünü yönetir.
*   **Çeşitli `Recv*Packet()` ve `Send*Packet()` fonksiyonları:** Belirli oyun mekanikleriyle ilgili paketleri işler ve gönderir (örneğin, karakter hareketleri, eşya işlemleri, sohbet, ticaret, lonca, parti vb.).
*   **`CPythonNetworkStream::SetHandler()`, `CPythonNetworkStream::SetPhaseWindow()`:** Python tarafındaki işleyici ve pencere nesnelerini C++ tarafına kaydeder.
*   **`CPythonNetworkStream::GetAccountCharacterSlotDatau()`, `CPythonNetworkStream::GetAccountCharacterSlotDataz()`:** Karakter seçme ekranındaki karakter bilgilerini Python'a sağlar.
*   **`CPythonNetworkStream::UploadMark()`, `CPythonNetworkStream::UploadSymbol()`:** Lonca sembollerini yükler.
*   **`CPythonNetworkStream::__DownloadMark()`, `CPythonNetworkStream::__DownloadSymbol()`:** Lonca sembollerini indirir.
*   **Hata Yönetimi ve Çıkış Fonksiyonları:** `ExitApplication()`, `ExitGame()`, `LogOutGame()`, `AbsoluteExitGame()`, `AbsoluteExitApplication()` gibi fonksiyonlar oyunun veya uygulamanın düzgün bir şekilde sonlandırılmasını sağlar.

Bu dosya, istemcinin sunucuyla senkronize bir şekilde çalışmasını, oyuncu eylemlerinin sunucuya doğru bir şekilde iletilmesini ve sunucudan gelen bilgilerin oyun dünyasına ve kullanıcı arayüzüne doğru bir şekilde yansıtılmasını sağlayarak tüm çevrimiçi oyun deneyiminin temelini oluşturur.

---

### `PythonNetworkStreamCommand.cpp` (Ağ Akışı Sunucu Komutları İşleme)

Bu dosya, `CPythonNetworkStream` sınıfının bir parçası olarak, sunucudan istemciye gönderilen metin tabanlı komutları işleyen `ServerCommand` metodunu ve istemci tarafından yürütülebilecek komutlar için (şu anda işlevsiz olan) `ClientCommand` metodunu içerir. Ayrıca, komut satırlarını ayrıştırmak için yardımcı fonksiyonlar (`SkipSpaces`, `OneArgument`, `SplitToken`) barındırır.

#### Dosyanın Genel Amaçları

*   **Sunucu Komutlarını Ayrıştırma ve Yönlendirme (`ServerCommand`)**: Sunucudan gelen bir string (metin komutu) alır. Bu komutu önce Python tarafındaki `BINARY_ServerCommand_Run` fonksiyonuna iletmeyi dener. Eğer Python tarafı bu komutu işlemezse (veya bir Python işleyici bağlı değilse), komutu boşluklara göre ayırır (`SplitToken`) ve ilk kelimeyi (ana komut) kullanarak C++ içinde bir dizi `if/else if` bloğu ile işler.
*   **İstemci Komutları için Altyapı (`ClientCommand`)**: Oyuncunun sohbet penceresine `/` ile yazdığı komutları işlemek için bir yer tutucu olarak bulunur, ancak mevcut implementasyonu her zaman `false` döndürür, yani istemci taraflı komutlar bu dosya üzerinden aktif olarak işlenmez (muhtemelen Python veya başka bir sistem tarafından ele alınır).
*   **Metin Ayrıştırma Yardımcıları**:
    *   `SkipSpaces(char** string)`: Verilen string işaretçisini, baştaki boşluk karakterlerini atlayacak şekilde ilerletir.
    *   `OneArgument(char* argument, char* first_arg)`: Bir argüman stringinden ilk "kelimeyi" (boşluk veya tırnak işaretleriyle ayrılmış) alır, küçük harfe çevirir ve `first_arg`'a kopyalar. Kalan stringi döndürür.
    *   `SplitToken(const char* c_szLine, CTokenVector* pstTokenVector, const char* c_szDelimeter)`: Verilen satırı, belirtilen ayırıcıya (varsayılan olarak boşluk) göre token'lara (kelimelere) böler ve bunları `pstTokenVector` içine atar. Tırnak içindeki ifadeleri tek bir token olarak kabul eder.

#### `ServerCommand` İçinde İşlenen Önemli Komutlar

`ServerCommand` fonksiyonu, sunucudan gelen ve oyun istemcisinde çeşitli eylemleri tetikleyen birçok farklı komutu işleyebilir. Bazı örnekler:

*   **`quit`**: İstemciyi kapatır.
*   **`BettingMoney`**: (İşlevselliği yorum satırında) Muhtemelen bir bahis sistemiyle ilgili para miktarını ayarlardı.
*   **`RefreshMonsterDropInfo` (`ENABLE_SEND_TARGET_INFO`)**: Belirli bir canavarın düşürebileceği eşya bilgilerini yenilemek için Python tarafındaki `BINARY_RefreshTargetMonsterDropInfo` fonksiyonunu çağırır.
*   **`gift`**: Hediye kutusu arayüzünü göstermek için Python tarafındaki `Gift_Show` fonksiyonunu çağırır.
*   **`SnowTexture` (`ENVIRONMENT_SYSTEM`)**: Haritada kar dokularını etkinleştirmek/devre dışı bırakmak için `CPythonBackground` üzerinden harita doku setini değiştirir.
*   **`MyPrivShopOpen` (`ENABLE_MYSHOP_DECO`)**: Kişisel pazarın (belirli dekorasyonlarla) açılması için Python tarafındaki `MyPrivShopOpen` fonksiyonunu çağırır.
*   **`cube` (Alt komutlar: `open`, `close`, `info`, `success`, `fail`, `r_list`, `m_info`)**: Küp (üretim/dönüşüm) sistemiyle ilgili çeşitli işlemleri (pencere açma/kapama, bilgi güncelleme, sonuç listesi, malzeme listesi) Python tarafındaki `BINARY_Cube_*` fonksiyonları aracılığıyla yönetir.
*   **`ObserverCount`, `ObserverMode`**: Lonca savaşlarında izleyici sayısı ve modunu günceller/ayarlar.
*   **`OXAreaRender` (`ENABLE_OX_RENDER_AREA`)**: OX etkinliği alanının belirli bir bölümünü renklendirmek için `CPythonBackground` fonksiyonunu çağırır.
*   **`RefineFailedType` (`ENABLE_REFINE_MSG_ADD`)**: Eşya geliştirme başarısız olduğunda nedenini belirten bir mesaj için Python fonksiyonunu çağırır.
*   **`StoneDetect`**: Bir metin taşının (veya benzeri bir nesnenin) oyuncuya göre konumunu (mesafe ve açı) bularak ilgili efekti göstermek için `CEffectManager`'ı kullanır.
*   **`StartStaminaConsume`, `StopStaminaConsume`**: Dayanıklılık (stamina) tüketimini başlatır veya durdurur.
*   **`messenger_auth`**: Arkadaşlık isteği geldiğinde Python tarafındaki `OnMessengerAddFriendQuestion` fonksiyonunu çağırır.
*   **`combo`**: Oyuncunun kombo yetenek bayrağını ayarlar.
*   **`setblockmode`**: Oyuncunun blok modunu (muhtemelen saldırı/davet engelleme) ayarlamak için Python fonksiyonunu çağırır.
*   **Duygu (Emotion) Komutları**: `french_kiss`, `kiss`, `slap`, `clap`, `cheer1`, `cheer2`, `dance1`, `dance2`, `dig_motion` ve bir `s_emotionDict` içinde tanımlı diğer birçok duygu komutunu işler. Bu komutlar, belirtilen karakterin (VID ile) ilgili duyguyu gerçekleştirmesini sağlar (`CInstanceBase::ActDualEmotion` veya `CInstanceBase::ActEmotion`).
*   **Bilinmeyen Komutlar**: Eğer komut yukarıdakilerden hiçbiri değilse ve `s_emotionDict` içinde de bulunmuyorsa, bir hata mesajı basılır.

#### C++ ve Python Arayüzü ile İlişkisi

*   `ServerCommand` öncelikle gelen komutu Python tarafındaki `BINARY_ServerCommand_Run` fonksiyonuna devretmeye çalışır. Bu, oyun geliştiricilerinin C++ kodunu değiştirmeden Python betikleriyle yeni sunucu komutları eklemesine veya mevcut olanları değiştirmesine olanak tanır.
*   Eğer Python tarafı komutu işlemezse, C++ içindeki `if/else if` blokları devreye girer. Bu bloklar da genellikle doğrudan oyun mantığını değiştirmek yerine, `PyCallClassMemberFunc` aracılığıyla `m_apoPhaseWnd[PHASE_WINDOW_GAME]` (aktif oyun penceresi) veya diğer ilgili Python nesnelerinin belirli metotlarını çağırır. Bu, C++\'ın oyunun temel sistemlerini yönetirken, UI ve bazı oyun özellikleri mantığının Python\'da kalmasını sağlar.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Dinamik Oyun Olayları**: Sunucu, oyun içinde belirli olayları tetiklemek (örneğin, bir GM duyurusu, bir sistem mesajı, bir efektin oynatılması) veya oyuncunun arayüzünde değişiklik yapmak (örneğin, bir pencere açmak, bir bilgiyi güncellemek) için bu komutları gönderir.
*   **Sistem Entegrasyonu**: Küp sistemi, binek sistemi, OX etkinliği gibi çeşitli oyun sistemleri, sunucu tarafından gönderilen komutlarla istemci tarafında yönetilir ve güncellenir.
*   **Duygu ve İfadeler**: Diğer oyuncuların veya NPC\'lerin duygusal ifadeleri (dans etme, alkışlama vb.) sunucudan gelen komutlarla tetiklenir.
*   **Hata Ayıklama ve Yönetimsel İşlevler**: Bazı komutlar (örneğin, `quit`) yönetimsel veya hata ayıklama amaçlı olabilir.
*   **Genişletilebilirlik**: Python işleyicisine öncelik verilmesi, oyunun sunucu taraflı komut setinin C++ kodunu yeniden derlemeden Python üzerinden kolayca genişletilmesini sağlar.

Bu dosya, sunucunun istemci üzerinde daha karmaşık ve çeşitli eylemler gerçekleştirmesine olanak tanıyan esnek bir komut işleme mekanizması sunar. İstemcinin sadece ham paket verilerini değil, aynı zamanda daha yüksek seviyeli, metin tabanlı direktifleri de anlamasını ve bunlara tepki vermesini sağlar.

---

### `PythonNetworkStreamEvent.cpp` (Ağ Akışı Olayları İşleme)

Bu dosya, `CPythonNetworkStream` sınıfının ağ bağlantısıyla ilgili temel olayları ve bazı oyun içi olayları işleyen metotlarını içerir.

#### Dosyanın Genel Amaçları

*   **Bağlantı Kesilmesi Olayları (`OnRemoteDisconnect`, `OnDisconnect`)**: Sunucu tarafından bağlantının kesilmesi veya genel bir bağlantı kesilmesi durumunda çağrılacak fonksiyonları tanımlar.
*   **Betik Olayı Başlatma (`OnScriptEventStart`)**: Sunucudan bir betik olayının (genellikle görevlerle ilgili) başlatılması komutu geldiğinde çağrılır.

#### İşlenen Olaylar ve Metotlar

*   **`void CPythonNetworkStream::OnRemoteDisconnect()`**
    *   Sunucu tarafından bağlantı sonlandırıldığında `CNetworkStream` tarafından çağrılır.
    *   Bu fonksiyon, `m_poHandler` (genellikle ana Python arayüzünü veya giriş ekranını yöneten nesne) üzerindeki `SetLoginPhase` adlı Python fonksiyonunu çağırır. Bu, oyuncuyu tekrar giriş ekranına yönlendirir.
*   **`void CPythonNetworkStream::OnDisconnect()`**
    *   Yerel olarak bir bağlantı kesilmesi durumu oluştuğunda (örneğin, ağ kablosu çekildiğinde veya istemci kendi bağlantısını kapattığında) `CNetworkStream` tarafından çağrılır.
    *   Bu fonksiyonun içi boştur. Genellikle `OnRemoteDisconnect` daha spesifik bir durum olduğu için orada işlem yapılır veya bağlantı kesilme mantığı diğer `Set*Phase` fonksiyonları içinde genel olarak ele alınır.
*   **`void CPythonNetworkStream::OnScriptEventStart(int iSkin, int iIndex)`**
    *   Sunucudan `HEADER_GC_SCRIPT` paketi ile bir betik (quest) olayı başlatma isteği geldiğinde çağrılır.
    *   `iSkin`: Genellikle görevin veya olayın arayüz temasını (skin) belirtir.
    *   `iIndex`: Betik veya görev numarasını belirtir.
    *   Bu fonksiyon, `m_apoPhaseWnd[PHASE_WINDOW_GAME]` (aktif oyun arayüzü penceresi) üzerindeki `OpenQuestWindow` adlı Python fonksiyonunu `(iSkin, iIndex)` argümanlarıyla çağırır. Bu, istemci tarafında ilgili görev penceresinin açılmasını veya güncellenmesini tetikler.

#### C++ ve Python Arayüzü ile İlişkisi

*   Bu dosyadaki C++ metotları, doğrudan Python tarafındaki UI ve oyun mantığı fonksiyonlarını çağırır (`PyCallClassMemberFunc`).
*   `OnRemoteDisconnect`, oyuncuyu Python tabanlı giriş ekranına (`SetLoginPhase`) yönlendirir.
*   `OnScriptEventStart`, Python tabanlı görev arayüzünü (`OpenQuestWindow`) tetikler.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Bağlantı Kaybı Yönetimi**: Oyuncunun sunucuyla bağlantısı koptuğunda, oyunun kilitlenmesini önlemek ve oyuncuyu tekrar giriş yapabileceği bir duruma (giriş ekranı) döndürmek için `OnRemoteDisconnect` kullanılır.
*   **Görev Sistemi Entegrasyonu**: Sunucu, bir oyuncuya yeni bir görev verdiğinde veya mevcut bir görevin bir sonraki adımını tetiklediğinde `OnScriptEventStart` aracılığıyla istemciye bilgi gönderir. İstemci de bu bilgiyle ilgili görev penceresini açar veya günceller, oyuncuya görev hakkında bilgi sunar.

Bu dosya, temel ağ olaylarını ve bazı önemli oyun içi olaylarını Python tarafındaki daha üst seviye mantığa bağlayarak istemcinin bu durumlara uygun tepkiler vermesini sağlar. Özellikle bağlantı sorunları ve görev sistemi gibi kritik özelliklerin düzgün çalışması için önemlidir.

---

### `PythonNetworkStreamModule.cpp` (Ağ Akışı Python Modülü)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının ve ilgili ağ işlevlerinin büyük bir kısmını Python betiklerine sunan `net` adlı bir Python modülü tanımlar. İstemcinin sunucuyla olan etkileşimlerini Python seviyesinde yönetmek için bir köprü görevi görür.

#### Dosyanın Genel Amaçları

*   **Python Modülü (`net`) Tanımlama**: `CPythonNetworkStream` singleton örneğine erişerek C++ tarafındaki ağ fonksiyonlarını Python'dan çağrılabilir hale getiren çok sayıda sarmalayıcı (wrapper) C fonksiyonu içerir.
*   **Ağ İşlevlerini Python'a Aktarma**: Oyuncu girişi, karakter seçimi, oyun içi aksiyonlar (hareket, sohbet, eşya kullanımı, ticaret vb.), bağlantı yönetimi ve daha birçok ağ işlemini Python betiklerinin kullanımına sunar.
*   **Veri Alışverişi**: Python'dan alınan argümanları C++ fonksiyonlarına iletir ve C++ fonksiyonlarından dönen sonuçları Python veri tiplerine dönüştürür.
*   **Sistem ve Özellik Entegrasyonu**: `ENABLE_` ile başlayan makrolar aracılığıyla oyuna eklenmiş çeşitli sistemlerin (örneğin, Evcil Hayvan Sistemi, Aura Sistemi, Posta Kutusu, Özel Pazar, Eşya Yağmalama Filtresi vb.) ağ ile ilgili fonksiyonlarını da Python'a açar.

#### `net` Python Modülü Fonksiyon Grupları (Örnekler)

Bu modül, çok geniş bir fonksiyon yelpazesi sunar. İşte bazı temel kategoriler ve örnek fonksiyonlar:

*   **Bağlantı ve Oturum Yönetimi**:
    *   `net.ConnectTCP(ip, port)`: Belirtilen IP ve porta TCP bağlantısı kurar (genellikle login sunucusu).
    *   `net.SetLoginInfo(username, password)`: Kullanıcı adı ve şifreyi ayarlar.
    *   `net.SendLoginPacket(username, password)`: Giriş paketini sunucuya gönderir.
    *   `net.SendSelectEmpirePacket(empire_id)`: İmparatorluk seçimini gönderir.
    *   `net.SendSelectCharacterPacket(slot_index)`: Karakter seçimini gönderir.
    *   `net.SendEnterGamePacket()`: Oyuna giriş isteğini gönderir.
    *   `net.LogOutGame()`: Oyundan çıkış yapar (karakter seçme ekranına döner).
    *   `net.ExitGame()`: Oyundan çıkar.
    *   `net.ExitApplication()`: Uygulamayı kapatır.
    *   `net.Disconnect()`: Mevcut bağlantıyı keser.
    *   `net.IsConnect()`: Bağlantının aktif olup olmadığını kontrol eder.
*   **Oyuncu ve Karakter Bilgileri**:
    *   `net.GetMainActorVID()`: Ana karakterin VID'ini alır.
    *   `net.GetMainActorRace()`: Ana karakterin ırkını alır.
    *   `net.GetAccountCharacterSlotDataInteger(slot, type)`: Karakter slotundaki belirli bir bilgiyi (integer) alır.
    *   `net.GetAccountCharacterSlotDataString(slot, type)`: Karakter slotundaki belirli bir bilgiyi (string) alır.
*   **Oyun İçi Eylemler (Paket Gönderimi)**:
    *   **Sohbet ve İfade**: `net.SendChatPacket(message, type)`, `net.SendWhisperPacket(target_name, message)`, `net.SendEmoticon(emoticon_id)`.
    *   **Karakter Hareket ve Eylemleri**: `net.SendCharacterPositionPacket(direction)`. (Not: Asıl hareket paketleri genellikle `CPythonPlayer` üzerinden daha sık gönderilir).
    *   **Eşya Yönetimi**: `net.SendItemUsePacket(item_pos)`, `net.SendItemMovePacket(src_pos, dst_pos, count)`, `net.SendItemPickUpPacket(item_vid)`, `net.SendItemDropPacket(item_pos, count_or_money)`.
    *   **Ticaret (Exchange)**: `net.SendExchangeStartPacket(target_vid)`, `net.SendExchangeItemAddPacket(item_pos, display_pos)`, `net.SendExchangeElkAddPacket(money)`, `net.SendExchangeAcceptPacket()`.
    *   **Dükkan (Shop)**: `net.SendShopEndPacket()`, `net.SendShopBuyPacket(slot_index)`, `net.SendShopSellPacket(slot_index)`.
    *   **Güvenli Depo (Safebox)**: `net.SendSafeboxCheckinPacket(inventory_pos, safebox_pos)`, `net.SendSafeboxCheckoutPacket(safebox_pos, inventory_pos)`.
    *   **Görevler (Quest)**: `net.SendQuestConfirmPacket(answer, pid)`, `net.SendQuestInputStringPacket(text)`.
    *   **Lonca (Guild)**: `net.SendAnswerMakeGuildPacket(guild_name)`, `net.SendGuildAddMemberPacket(target_vid)`, `net.UploadMark(file_name)`, `net.UploadSymbol(file_name)`.
    *   **Parti (Party)**: `net.SendPartyInvitePacket(target_vid)`, `net.SendPartyExitPacket()`.
*   **Arayüz ve İstemci Ayarları**:
    *   `net.SetHandler(pyObject)`: Ana Python olay işleyici nesnesini ayarlar.
    *   `net.SetPhaseWindow(phase_enum, pyObject)`: Belirli bir oyun aşaması için Python pencere nesnesini ayarlar.
    *   `net.SetServerInfo(info_string)`: Sunucu bilgisini (genellikle `serverinfo.py` içeriği) ayarlar.
    *   `net.LoadInsultList(file_name)`: Küfür filtresi için kelime listesini yükler.
*   **Özel Sistem Fonksiyonları**:
    *   **Evcil Hayvan (`ENABLE_GROWTH_PET_SYSTEM`)**: `net.SendPetHatchingPacket(name, egg_slot)`, `net.SendPetFeedPacket(feed_item_slot, target_pet_slot)`.
    *   **Posta Kutusu (`ENABLE_MAILBOX`)**: `net.SendPostWrite(to_name, title, message, item_pos, yang, won)`, `net.SendPostGetItems(mail_index)`.
    *   **Aura Sistemi (`ENABLE_AURA_COSTUME_SYSTEM`)**: `net.SendAuraRefineCheckIn(item_pos, aura_window_pos, aura_refine_type)`, `net.SendAuraRefineAccept(aura_refine_type)`.
    *   **Eşya Yağmalama Filtresi (`ENABLE_LOOTING_SYSTEM`)**: `net.SendLootingSettings(settings_dict)`.
    *   **Görünüm Değiştirme (`ENABLE_CHANGE_LOOK_SYSTEM`)**: `net.SendChangeLookCheckIn(item_pos, change_look_slot)`, `net.SendChangeLookAccept()`.
    *   Ve daha birçok sisteme özel fonksiyon (`ENABLE_MYSHOP_DECO`, `ENABLE_PREMIUM_PRIVATE_SHOP`, `ENABLE_ATTR_6TH_7TH`, `ENABLE_GEM_SYSTEM` vb.).

#### `initnet()` Fonksiyonu

*   Bu C fonksiyonu, Python yorumlayıcısı tarafından `net` modülü ilk kez yüklendiğinde çağrılır.
*   `PyMethodDef s_methods[]` dizisi, yukarıda bahsedilen C sarmalayıcı fonksiyonlarını Python isimleriyle (`"ConnectTCP"`, `"SendChatPacket"`, `"GetMainActorVID"` vb.) eşleştirir.
*   `Py_InitModule("net", s_methods)` ile `net` modülünü ve içindeki tüm fonksiyonları Python ortamına kaydeder.
*   Ayrıca, `CHAT_TYPE_*` gibi bazı önemli C++ sabitlerini `PyModule_AddIntConstant` kullanarak doğrudan `net` modülüne (`net.CHAT_TYPE_TALKING` vb.) aktarır, böylece Python betikleri bu sabitleri kolayca kullanabilir.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Python UI -> C++ Ağ Katmanı İletişimi**: Python'da yazılmış kullanıcı arayüzü (UI) elemanlarından gelen etkileşimler (örneğin, bir butona tıklama, bir metin girişi) `net` modülü aracılığıyla C++ tarafındaki `CPythonNetworkStream` fonksiyonlarına yönlendirilir ve sunucuya uygun paketlerin gönderilmesi sağlanır.
*   **Oyun Mantığının Yönetimi**: Python betikleri, oyunun akışını (giriş, karakter seçimi, oyun içi etkileşimler) `net` modülü fonksiyonlarını çağırarak yönetir.
*   **İstemci Yapılandırması**: Oyun ayarları, sunucu bilgileri, olay işleyicileri gibi istemci tarafı yapılandırmaları Python'dan `net` modülü aracılığıyla C++ tarafına iletilir.
*   **Genişletilebilirlik**: Yeni oyun özellikleri veya sistemleri eklendiğinde, bu özelliklerle ilgili ağ fonksiyonları `PythonNetworkStreamModule.cpp` dosyasına eklenerek ve `initnet()` içinde kaydedilerek kolayca Python tarafının kullanımına sunulabilir.

Bu dosya, Metin2 istemcisinin C++ tabanlı ağ altyapısı ile Python tabanlı kullanıcı arayüzü ve oyun mantığı arasında hayati bir bağlantı noktasıdır. Python'un esnekliği ile C++'ın performansını bir araya getirerek karmaşık ağ etkileşimlerinin yönetilmesini kolaylaştırır.

### `PythonNetworkStreamPhaseGame.cpp` (Oyun Aşaması Ağ Akışı)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının oyunun "Game" (Oyun) aşamasıyla ilgili ağ paketlerini işleyen kısmını içerir. Oyuncunun oyun dünyasıyla etkileşimde bulunduğu (karakter hareketleri, envanter yönetimi, sohbet, ticaret, görevler, yetenek kullanımı, vb.) ana oyun döngüsü sırasında sunucudan gelen verilerin çoğunu yönetir ve oyuncu eylemlerini sunucuya gönderir.

#### Dosyanın Temel Amaçları

*   **Oyun Aşaması Paket İşleme (`GamePhase` fonksiyonu)**: İstemcinin "Game" (Oyun) aşamasındayken sunucudan gelen çeşitli paketleri (örneğin, karakter ekleme/güncelleme/silme, eşya bilgileri, sohbet mesajları, hareketler, yetenekler) ayrıştıran ve ilgili işlevleri çağıran ana döngüyü içerir.
*   **Arayüz Güncelleme Tetikleyicileri**: Sunucudan gelen verilere göre oyun arayüzünün çeşitli pencerelerini (envanter, karakter durumu, yetenekler, lonca, görevler vb.) yenilemek için Python tarafına çağrılar yapar (örneğin, `__RefreshInventoryWindow`, `__RefreshStatus`).
*   **Oyuncu Eylemlerini Gönderme**: Oyuncunun yaptığı çeşitli eylemleri (karakter hareketi, yetenek kullanımı, sohbet mesajı gönderme, eşya kullanma, ticaret teklifleri vb.) sunucuya iletmek için paket gönderme fonksiyonlarını içerir.
*   **Oyun Durumu Yönetimi**: Warp (ışınlanma), PVP durumları, düello başlangıcı gibi oyun durumlarını yönetir.
*   **Veri Senkronizasyonu**: Oyuncunun ve diğer karakterlerin pozisyonları, can/mana gibi durumları ve diğer önemli verilerin sunucu ile senkronize edilmesini sağlar.
*   **Hata ve Bildirim Yönetimi**: Hack denemelerini bildirme, hata paketlerini işleme gibi işlevleri de barındırır.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **`CPythonNetworkStream::GamePhase()`**:
    *   Bu fonksiyon, oyun aşamasında sürekli olarak çağrılır ve ağdan gelen paketleri kontrol eder.
    *   `CheckPacket(&header)` ile bir sonraki paketin başlığını okur.
    *   Gelen paketin `header` değerine göre bir `switch` ifadesi içinde ilgili `Recv...` fonksiyonunu çağırarak paketi işler (örneğin, `RecvCharacterAppendPacket`, `RecvItemSetPacket`, `RecvChatPacket`).
    *   Belirli aralıklarla veya olaylarla tetiklenerek çeşitli arayüz pencerelerinin (`m_isRefreshCharacterWnd`, `m_isRefreshInventoryWnd` vb. bayraklar aracılığıyla) Python tarafında güncellenmesini sağlar.

*   **`Recv...` Fonksiyonları (Örnekler)**:
    *   `RecvCharacterAppendPacket()`: Yeni bir karakterin haritaya eklendiği bilgisini alır ve karakteri oyun dünyasında oluşturur.
    *   `RecvCharacterUpdatePacket()`: Mevcut bir karakterin bilgilerinin (pozisyon, can, vb.) güncellendiği paketi işler.
    *   `RecvItemSetPacket()` / `RecvItemUpdatePacket()`: Envanter veya ekipmanla ilgili eşya bilgilerini alır ve günceller.
    *   `RecvChatPacket()`: Diğer oyunculardan veya sistemden gelen sohbet mesajlarını işler ve arayüzde gösterir.
    *   `RecvPointChange()`: Oyuncunun veya bir hedefin HP, MP, EXP gibi puanlarının değiştiği bilgisini alır ve arayüzü günceller.
    *   `RecvWarpPacket()`: Oyuncunun farklı bir haritaya veya konuma ışınlandığı bilgisini işler ve yeni haritayı yükler.
    *   `RecvShopPacket()`: NPC dükkanlarının içeriğini alır ve dükkan arayüzünü açar/günceller.
    *   `RecvQuestInfoPacket()`: Görevlerle ilgili bilgileri (yeni görev, görev güncellemesi) alır ve görev arayüzünü günceller.
    *   `RecvGuild()`: Lonca ile ilgili çeşitli bilgileri (üye listesi, lonca seviyesi, beceriler) işler.
    *   `RecvParty...()` fonksiyonları: Parti davetleri, üye ekleme/çıkarma, durum güncellemeleri gibi partiyle ilgili paketleri yönetir.

*   **`Send...` Fonksiyonları (Örnekler)**:
    *   `SendCharacterStatePacket()`: Karakterin hareket (yürüme, koşma) durumunu sunucuya bildirir.
    *   `SendUseSkillPacket()`: Oyuncunun bir yetenek kullandığını sunucuya iletir.
    *   `SendChatPacket()`: Oyuncunun yazdığı sohbet mesajını sunucuya gönderir.
    *   `SendItemUsePacket()` (genellikle `PythonPlayerModule.cpp` üzerinden çağrılır): Oyuncunun bir eşya kullandığını bildirir.
    *   `SendAttackPacket()`: Oyuncunun bir saldırı yaptığını (belirli bir hareket ve hedefle) sunucuya iletir.
    *   `SendTargetPacket()`: Oyuncunun bir hedef seçtiğini sunucuya bildirir.

*   **Arayüz Yenileme Fonksiyonları (`__Refresh...Window`)**:
    *   Bu fonksiyonlar (örneğin, `__RefreshInventoryWindow`, `__RefreshStatus`, `__RefreshSkillWindow`), genellikle ilgili `m_isRefresh...Wnd` boolean bayrağını `true` yapar.
    *   `GamePhase` döngüsünün sonunda, bu bayraklar kontrol edilir ve eğer `true` ise `PyCallClassMemberFunc` aracılığıyla Python'daki `game.py` (veya benzeri bir UI yönetim betiği) içindeki ilgili `Refresh...` fonksiyonu çağrılarak arayüz güncellenir.
    *   Bu, C++ tarafındaki veri değişikliklerinin kullanıcı arayüzüne yansıtılmasını sağlar.

*   **Discord RPC Entegrasyonu (`ENABLE_DISCORD_RPC`)**:
    *   Eğer bu makro tanımlıysa, oyunun farklı aşamalarına (giriş, karakter seçimi, oyun içi) göre Discord Rich Presence güncellemeleri yapılır.
    *   `Discord_Update()` fonksiyonu, oyuncunun mevcut durumu, seviyesi, karakter adı gibi bilgileri Discord'da gösterir.

#### Önemli Yapılar ve Kavramlar

*   **Paket Başlıkları (`HEADER_GC_...`)**: Sunucudan gelen her paket türünü tanımlayan sabitlerdir. `GamePhase` içindeki `switch` ifadesi bu başlıkları kullanarak doğru işleyici fonksiyonu seçer.
*   **Python Etkileşimi (`PyCallClassMemberFunc`)**: C++ tarafındaki olaylara veya veri değişikliklerine yanıt olarak Python'da tanımlanmış UI fonksiyonlarını çağırmak için kullanılır. Genellikle `m_apoPhaseWnd[PHASE_WINDOW_GAME]` üzerinden çağrı yapılır, bu da oyun aşamasındaki ana Python pencere nesnesine işaret eder.
*   **Durum Bayrakları (`m_isRefresh...Wnd`)**: Belirli arayüz pencerelerinin yenilenmesi gerekip gerekmediğini takip etmek için kullanılan boolean değişkenlerdir. Bu, gereksiz yere sürekli arayüz güncellemesi yapılmasını engeller ve performansı artırır.
*   **Yerel ve Global Pozisyonlar**: Fonksiyonlar (`__GlobalPositionToLocalPosition`, `__LocalPositionToGlobalPosition`) aracılığıyla sunucunun kullandığı global koordinatlar ile istemcinin kullandığı yerel harita koordinatları arasında dönüşüm yapılır.

Bu dosya, oyunun temel oynanış mekaniklerinin ağ üzerinden yönetilmesi ve kullanıcı arayüzü ile senkronize edilmesi açısından kritik bir rol oynar. Sunucudan gelen her türlü oyun olayı burada karşılanır ve oyuncunun eylemleri buradan sunucuya doğru yola çıkar.

### `PythonNetworkStreamPhaseGameActor.cpp` (Oyun Aşaması Aktör Ağ Akışı)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının oyunun "Game" (Oyun) aşamasında karakterlerin (aktörlerin) oluşturulması, güncellenmesi ve yönetilmesiyle ilgili ağ paketlerini işleyen bölümünü içerir. Esasen, sunucudan gelen karakter verilerini alıp oyun dünyasında görselleştirmek ve bu karakterlerin durumlarını senkronize tutmakla sorumludur.

#### Dosyanın Temel Amaçları

*   **Karakter Oluşturma ve Güncelleme**: Sunucudan gelen "karakter ekleme" (append) ve "karakter güncelleme" (update) paketlerini işleyerek oyun dünyasına yeni karakterler ekler veya mevcut karakterlerin bilgilerini (pozisyon, görünüm, hız, durum vb.) günceller.
*   **Karakter Silme**: Sunucudan gelen "karakter silme" (delete) paketini işleyerek karakterleri oyun dünyasından kaldırır.
*   **Pozisyon Yönetimi**: Karakterlerin global ve lokal pozisyonları arasında dönüşüm yapar.
*   **Hareket ve Senkronizasyon**: Karakter hareketlerini ve pozisyon senkronizasyon paketlerini işler.
*   **Sahiplik (Ownership)**: NPC'lerin veya mob'ların kime ait olduğu bilgisini yönetir.
*   **Görünüm ve Ekipman**: Karakterlerin zırh, silah, saç gibi görünür ekipmanlarını ve diğer durum bayraklarını (affect flags) ayarlar.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **Koordinat Dönüşümleri**:
    *   `CPythonNetworkStream::__GlobalPositionToLocalPosition(LONG& rGlobalX, LONG& rGlobalY)`: Sunucudan gelen global dünya koordinatlarını, istemcinin harita üzerinde kullandığı lokal koordinatlara çevirir. Bu, `CPythonBackground` sınıfı aracılığıyla yapılır.
    *   `CPythonNetworkStream::__LocalPositionToGlobalPosition(LONG& rLocalX, LONG& rLocalY)`: Lokal koordinatları global koordinatlara çevirir.

*   **Paket Alım Fonksiyonları (`Recv...`)**:
    *   **`RecvCharacterAppendPacket()` ve `RecvCharacterAdditionalInfo()`**:
        *   Bu iki fonksiyon birlikte çalışarak yeni bir karakterin oyuna eklenmesini yönetir.
        *   `TPacketGCCharacterAdd` paketi temel bilgileri (tip, hız, ırk, pozisyon, etki bayrakları vb.) içerir.
        *   Eğer karakter PC veya özel bir NPC değilse (`kNetActorData.m_bType != CActorInstance::TYPE_PC && !IsNPCType(kNetActorData.m_bType)`), ismi `CPythonNonPlayer` üzerinden alınır ve `__RecvCharacterAppendPacket` hemen çağrılır.
        *   Eğer PC veya özel bir NPC ise, bu ilk bilgiler `s_kNetActorData` statik değişkeninde saklanır ve ardından `TPacketGCCharacterAdditionalInfo` paketi beklenir. Bu ikinci paket, karakterin ismini, seviyesini, loncasını, ekipmanlarını vb. daha detaylı bilgileri içerir. Bu paket alındıktan sonra `s_kNetActorData` bu yeni bilgilerle güncellenir ve `__RecvCharacterAppendPacket` çağrılır.
        *   `IsInvisibleRace()` ile bazı özel görünmez ırkların işlenmesi atlanır.
    *   **`RecvCharacterAppendPacketNew()`**: Muhtemelen daha yeni veya farklı bir karakter ekleme paketi formatını işler. Doğrudan tüm bilgileri (isim dahil) tek pakette alır.
    *   **`RecvCharacterUpdatePacket()` ve `RecvCharacterUpdatePacketNew()`**:
        *   Mevcut bir karakterin bilgilerini (ekipman, hız, etki bayrakları, PK modu, seviye vb.) günceller.
        *   `__RecvCharacterUpdatePacket` fonksiyonunu çağırarak güncelleme işlemini gerçekleştirir.
    *   **`__RecvCharacterAppendPacket(SNetworkActorData* pkNetActorData)`**:
        *   Asıl karakter oluşturma işlemini yapar.
        *   `m_rokNetActorMgr->AppendActor(*pkNetActorData)` çağrısıyla `CNetworkActorManager` üzerinden yeni bir aktör (karakter instance'ı) oluşturulur.
        *   Eğer eklenen karakter ana oyuncu ise, oyuncunun ırkı ayarlanır, silah gücü hesaplanır (`__SetWeaponPower`), harita ismi gösterilir ve lonca ID'si güncellenir.
    *   **`__RecvCharacterUpdatePacket(SNetworkUpdateActorData* pkNetUpdateActorData)`**:
        *   `m_rokNetActorMgr->UpdateActor(*pkNetUpdateActorData)` çağrısıyla `CNetworkActorManager` üzerinden mevcut bir aktörün verileri güncellenir.
        *   Eğer güncellenen karakter ana oyuncu ise, lonca ID'si güncellenir, silah gücü yeniden hesaplanır ve çeşitli arayüz pencerelerinin (`__RefreshStatus`, `__RefreshEquipmentWindow` vb.) yenilenmesi tetiklenir.
    *   **`RecvCharacterDeletePacket()`**:
        *   `m_rokNetActorMgr->RemoveActor(chrDelPacket.dwVID)` çağrısıyla belirtilen VID'ye sahip karakteri `CNetworkActorManager` üzerinden siler.
        *   Ayrıca, eğer bu karakterin bir özel dükkan tabelası varsa, onu da kaldırır.
    *   **`RecvCharacterMovePacket()`**:
        *   Karakterin hareket bilgilerini (hedef pozisyon, rotasyon, hareket tipi, süre) alır.
        *   Global pozisyonu lokale çevirir ve `m_rokNetActorMgr->MoveActor(kNetMoveActorData)` ile karakterin hareketini başlatır.
    *   **`RecvOwnerShipPacket()`**:
        *   Bir karakterin (genellikle bir mob) sahipliğinin başka bir karaktere (genellikle oyuncu) geçtiğini bildirir. `m_rokNetActorMgr->SetActorOwner()` ile bu bilgi işlenir.
    *   **`RecvSyncPositionPacket()`**:
        *   Sunucudan gelen pozisyon senkronizasyon paketlerini işler. Birden fazla karakterin pozisyonunu tek pakette güncelleyebilir.
        *   Her karakter için global pozisyonu lokale çevirir ve `m_rokNetActorMgr->SyncActor()` ile karakterin pozisyonunu anında günceller.

*   **Yardımcı Fonksiyonlar**:
    *   **`__CanActMainInstance()`**: Ana oyuncu karakterinin hareket edip edemeyeceğini kontrol eder.
    *   **`__ClearNetworkActorManager()`**: `CNetworkActorManager` içindeki tüm aktörleri temizler.
    *   **`__SetWeaponPower(IAbstractPlayer& rkPlayer, DWORD dwWeaponID)`**: Oyuncunun elindeki silaha göre saldırı gücü değerlerini (`minPower`, `maxPower`, `minMagicPower`, `maxMagicPower`, `addPower`) hesaplar ve `CPythonPlayer` nesnesine setler. Bu, `CItemManager` ve `CPythonPlayer` üzerinden eşya bilgilerini okuyarak yapılır.
    *   **`IsNPCType(BYTE bType)`**: Verilen tipin bir NPC (NPC, At, Pet) olup olmadığını kontrol eder.

#### Önemli Yapılar ve Kavramlar

*   **`SNetworkActorData`**: Yeni bir karakter eklendiğinde sunucudan gelen tüm bilgileri tutan bir yapıdır.
*   **`SNetworkUpdateActorData`**: Mevcut bir karakter güncellendiğinde sunucudan gelen bilgileri tutan bir yapıdır.
*   **`SNetworkMoveActorData`**: Bir karakterin hareket bilgilerini tutan yapıdır.
*   **`CNetworkActorManager` (`m_rokNetActorMgr`)**: Oyun dünyasındaki tüm ağ bağlantılı aktörlerin (karakterlerin) yönetiminden sorumlu olan sınıftır. Karakterlerin oluşturulması, güncellenmesi, silinmesi ve hareket ettirilmesi gibi işlemleri bu sınıf üzerinden yapılır.
*   **VID (Virtual ID)**: Her karakteri benzersiz şekilde tanımlayan sanal bir kimliktir.
*   **Ekipman Parçaları (`CHR_EQUIPPART_ARMOR`, `CHR_EQUIPPART_WEAPON` vb.)**: Karakterin giydiği zırh, silah gibi ekipmanların ID'lerini tutar.
*   **Etki Bayrakları (`m_kAffectFlags`)**: Karakter üzerindeki çeşitli etkileri (zehir, hızlanma vb.) bit flag olarak tutar.

Bu dosya, oyuncunun ve çevresindeki diğer karakterlerin oyun dünyasında doğru bir şekilde görüntülenmesi ve hareket etmesi için `PythonNetworkStreamPhaseGame.cpp` ile yakın bir şekilde çalışır ve temel karakter yönetimi işlevlerini sağlar.

### `PythonNetworkStreamPhaseGameItem.cpp` (Oyun Aşaması Eşya Ağ Akışı)

Bu C++ kaynak dosyası, `CPythonNetworkStream` sınıfının oyunun "Game" (Oyun) aşamasında eşyalarla (item) ilgili ağ paketlerini işleyen bölümünü içerir. Envanter, depo (safebox), eşya marketi (mall), hızlı erişim slotları (quickslot) ve yerdeki eşyalarla ilgili sunucudan gelen verileri yönetir ve oyuncunun eşyalarla ilgili eylemlerini sunucuya gönderir.

#### Dosyanın Temel Amaçları

*   **Envanter Yönetimi**:
    *   Sunucudan gelen eşya bilgilerini (yeni eşya ekleme, eşya güncelleme, eşya silme) alıp oyuncunun envanterine uygular.
    *   Oyuncunun eşya kullanma, eşya taşıma, eşya bırakma (yere atma/silme) gibi eylemlerini sunucuya iletir.
*   **Depo (SafeBox) Yönetimi**:
    *   Depodaki eşya bilgilerini alır ve günceller.
    *   Oyuncunun depoya eşya koyma (check-in), depodan eşya alma (check-out), depodaki eşyaları taşıma gibi işlemlerini sunucuya gönderir.
    *   Depo şifresiyle ilgili hataları işler.
*   **Eşya Marketi (Mall) Yönetimi**:
    *   Eşya marketindeki (nesne market deposu) eşya bilgilerini alır ve günceller.
    *   Oyuncunun eşya marketinden eşya alma (checkout) işlemini sunucuya iletir.
*   **Yerdeki Eşyalar**:
    *   Yere düşen yeni eşyaların bilgilerini alır ve oyun dünyasında oluşturur.
    *   Yerden kalkan eşyaların silinmesi bilgisini işler.
    *   Yerdeki eşyaların sahiplik (ownership) bilgisini günceller.
    *   Eğer `ENABLE_DROP_DESTROY_TIME` aktifse, yerdeki eşyaların kaybolma süresini yönetir.
*   **Hızlı Erişim Slotları (QuickSlot)**:
    *   Hızlı erişim slotlarına eşya veya yetenek ekleme, silme ve taşıma bilgilerini sunucudan alır ve oyuncu verilerine uygular.
    *   Oyuncunun hızlı erişim slotlarında yaptığı değişiklikleri sunucuya gönderir.
*   **Özel Efektler ve Sesler**:
    *   Eşya kullanımıyla (iksir vb.) veya diğer olaylarla (başarı, başarısızlık, seviye atlama efektleri) ilgili özel efektlerin gösterilmesi için sunucudan gelen paketleri işler.
    *   Eşya kullanım ve düşürme seslerini tetikler.
*   **Diğer Eşya Etkileşimleri**:
    *   NPC dükkanlarından eşya alma/satma işlemlerini sunucuya iletir.
    *   Ejderha Taşı Simyası (Dragon Soul Refine) ile ilgili paketleri işler.
    *   `ENABLE_LUCKY_BOX` aktifse, şans kutusu işlemlerini yönetir.
    *   `ENABLE_GEM_SYSTEM` aktifse, mücevher dükkanı işlemlerini yönetir.

#### Anahtar Fonksiyonlar ve İşleyiş

*   **Depo (SafeBox) Fonksiyonları**:
    *   `RecvSafeBoxSetPacket()`: Depoya yeni bir eşya eklendiğinde veya mevcut bir eşya güncellendiğinde çağrılır. `CPythonSafeBox::Instance()->SetItemData()` ile depodaki eşya verisini ayarlar.
    *   `RecvSafeBoxDelPacket()`: Depodan bir eşya silindiğinde çağrılır. `CPythonSafeBox::Instance()->DelItemData()` ile eşyayı siler.
    *   `RecvSafeBoxWrongPasswordPacket()`: Yanlış depo şifresi girildiğinde Python tarafında hata mesajı gösterir.
    *   `RecvSafeBoxSizePacket()`: Depo açıldığında boyut bilgisini alır ve `CPythonSafeBox::Instance()->OpenSafeBox()` ile depoyu açar, Python tarafında da pencereyi açar.
    *   `SendSafeBoxCheckinPacket()`: Envanterden depoya eşya koyma isteği gönderir.
    *   `SendSafeBoxCheckoutPacket()`: Depodan envantere eşya alma isteği gönderir.
    *   `SendSafeBoxItemMovePacket()`: Depo içinde eşya taşıma isteği gönderir.

*   **Eşya Marketi (Mall) Fonksiyonları**:
    *   `RecvMallOpenPacket()`: Eşya marketi deposu açıldığında boyut bilgisini alır ve `CPythonSafeBox::Instance()->OpenMall()` ile marketi açar, Python tarafında da pencereyi açar.
    *   `RecvMallItemSetPacket()`: Markete yeni bir eşya eklendiğinde veya mevcut bir eşya güncellendiğinde çağrılır. `CPythonSafeBox::Instance()->SetMallItemData()` ile eşya verisini ayarlar.
    *   `RecvMallItemDelPacket()`: Marketten bir eşya silindiğinde çağrılır. `CPythonSafeBox::Instance()->DelMallItemData()` ile eşyayı siler.
    *   `SendMallCheckoutPacket()`: Marketten envantere eşya alma isteği gönderir.

*   **Envanter ve Eşya Fonksiyonları**:
    *   `RecvItemSetPacket()` / `RecvItemSetEmptyPacket()`: Envanterdeki bir slota yeni bir eşya geldiğinde veya güncellendiğinde çağrılır. `IAbstractPlayer::GetSingleton().SetItemData()` ile oyuncunun eşya verisini ayarlar. `highlight` bayrağı varsa Python tarafında eşyayı vurgular.
    *   `RecvItemUsePacket()`: Bir eşya kullanıldığında (genellikle tüketilebilirler) çağrılır ve envanteri yeniler.
    *   `RecvItemUpdatePacket()`: Envanterdeki bir eşyanın sayısını, soketlerini, efsunlarını, cevherlerini vb. günceller. `IAbstractPlayer::GetSingleton().SetItemCount()`, `SetItemMetinSocket()`, `SetItemAttribute()` gibi fonksiyonları kullanır.
    *   `RecvItemGroundAddPacket()`: Yere yeni bir eşya düştüğünde çağrılır. `CPythonItem::Instance()->CreateItem()` ile yerdeki eşyayı oluşturur. `ENABLE_ITEM_DROP_RENEWAL` ile detaylı efsun/soket bilgisi de gelebilir.
    *   `RecvItemGroundDelPacket()`: Yerdeki bir eşya kaybolduğunda çağrılır. `CPythonItem::Instance()->DeleteItem()` ile eşyayı siler.
    *   `RecvItemOwnership()`: Yerdeki bir eşyanın sahipliğini ayarlar. `CPythonItem::Instance()->SetOwnership()`.
    *   `SendItemUsePacket()`: Oyuncunun bir eşyayı kullanma isteğini sunucuya gönderir. Öncesinde ticaret, dükkan veya saldırı durumları gibi kontroller yapar.
    *   `SendItemUseToItemPacket()`: Bir eşyayı başka bir eşya üzerinde kullanma (örn: kutsama kağıdı) isteği gönderir.
    *   `SendItemDropPacket()` / `SendItemDropPacketNew()`: Eşyayı yere atma/silme isteği gönderir (para ile birlikte).
    *   `SendItemMovePacket()`: Envanter içinde veya envanter ile ekipman arasında eşya taşıma isteği gönderir.
    *   `SendItemPickUpPacket()`: Yerdeki bir eşyayı toplama isteği gönderir.

*   **Hızlı Erişim (QuickSlot) Fonksiyonları**:
    *   `RecvQuickSlotAddPacket()`: Hızlı slota yeni bir atama (eşya/beceri) eklendiğinde çağrılır. `IAbstractPlayer::GetSingleton().AddQuickSlot()`.
    *   `RecvQuickSlotDelPacket()`: Hızlı slottan bir atama silindiğinde çağrılır. `IAbstractPlayer::GetSingleton().DeleteQuickSlot()`.
    *   `RecvQuickSlotMovePacket()`: Hızlı slotlar arasında atama taşındığında çağrılır. `IAbstractPlayer::GetSingleton().MoveQuickSlot()`.
    *   `SendQuickSlotAddPacket()`, `SendQuickSlotDelPacket()`, `SendQuickSlotMovePacket()`: Oyuncunun hızlı slot değişikliklerini sunucuya gönderir.

*   **Özel Efekt ve Ses Fonksiyonları**:
    *   `RecvSpecialEffect()`: Sunucudan gelen özel efekt paketini işler (örn: iksir kullanma efekti, kritik vuruş efekti, seviye atlama efekti). `CInstanceBase::AttachSpecialEffect()` veya `CreateSpecialEffect()` ile efekti karakter üzerinde veya pozisyonda oynatır.
    *   `RecvSpecificEffect()`: Dosya yoluyla belirtilen özel bir efekti oynatır.
    *   `__PlayInventoryItemUseSound()`, `__PlayInventoryItemDropSound()`, `__PlaySafeBoxItemDropSound()`, `__PlayMallItemDropSound()`: İlgili eşya etkileşimlerinde sesleri çalar.

*   **Dükkan (Shop) Fonksiyonları**:
    *   `SendShopEndPacket()`: Dükkanı kapatma isteği gönderir.
    *   `SendShopBuyPacket()`: Dükkandan eşya satın alma isteği gönderir.
    *   `SendShopSellPacket()` / `SendShopSellPacketNew()`: Envanterdeki bir eşyayı dükkana satma isteği gönderir.

*   **Diğer**:
    *   `RecvDragonSoulRefine()`: Ejderha Taşı Simyası arayüzüyle ilgili sunucu cevaplarını işler (başarılı, başarısız, pencere açma).
    *   `RecvLuckyBoxPacket()` / `SendLuckyBoxActionPacket()`: Şans kutusu sistemiyle ilgili paketleri işler.
    *   `Recv...GemShop...()` / `Send...GemShop...()`: Mücevher dükkanı sistemiyle ilgili paketleri işler.

#### Önemli Yapılar ve Kavramlar

*   **`TItemPos`**: Bir eşyanın konumunu (pencere tipi - envanter, depo vb. ve slot indeksi) tanımlayan yapıdır.
*   **`TItemData`**: Bir eşyanın tüm bilgilerini (VNUM, adet, soketler, efsunlar, cevherler, bayraklar vb.) içeren yapıdır.
*   **`CPythonPlayer`**: Oyuncunun envanter, hızlı slot gibi verilerini tutar.
*   **`CPythonSafeBox`**: Oyuncunun depo ve eşya marketi (mall) verilerini tutar.
*   **`CPythonItem`**: Yerdeki eşyaların yönetiminden ve eşya ses/efektlerinden sorumludur.
*   **`__RefreshInventoryWindow()`**, **`__RefreshSafeboxWindow()`**, **`__RefreshMallWindow()`**: İlgili arayüz pencerelerinin Python tarafında güncellenmesini tetikleyen yardımcı fonksiyonlardır.

Bu dosya, oyuncunun oyun içindeki eşyalarla olan tüm etkileşimlerinin ağ üzerinden yönetilmesinde ve bu değişikliklerin kullanıcıya doğru bir şekilde yansıtılmasında merkezi bir rol oynar.