# UserInterface Referans Kılavuzu - Bölüm 8

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bu bölümde, istemcinin Python arayüzünün diğer önemli bileşenleri incelenecektir.

## İçindekiler

* [`PythonConfigModule.cpp` (Python `cfg` Modülü)](#pythonconfigmodulecpp-python-cfg-modülü)
* [`PythonCubeRenewal.cpp` (Yenilenmiş Küp Sistemi Python Modülü)](#pythoncuberenewalcpp-yenilenmiş-küp-sistemi-python-modülü)
* [`PythonCubeRenewal.h` (Yenilenmiş Küp Sistemi Başlık Dosyası)](#pythoncuberenewalh-yenilenmiş-küp-sistemi-başlık-dosyası)
* [`PythonEffectModule.cpp` (Python `effect` Modülü)](#pythoneffectmodulecpp-python-effect-modülü)
* [`PythonEventManager.h` (Görev/Etkinlik Yöneticisi Başlık Dosyası)](#pythoneventmanagerh-görevetkinlik-yöneticisi-başlık-dosyası)
* [`PythonEventManager.cpp` (Görev/Etkinlik Yöneticisi Implementasyonu)](#pythoneventmanagercpp-görevetkinlik-yöneticisi-implementasyonu)
* [`PythonEventManagerMoudle.cpp` (Python `event` Modülü)](#pythoneventmanagermoudlecpp-python-event-modülü)
* [`PythonExceptionSender.h` (Python İstisna Gönderici Başlık Dosyası)](#pythonexceptionsenderh-python-istisna-gönderici-başlık-dosyası)
* [`PythonExceptionSender.cpp` (Python İstisna Gönderici Implementasyonu)](#pythonexceptionsendercpp-python-istisna-gönderici-implementasyonu)
* [`PythonExchange.h` (Ticaret Sistemi Başlık Dosyası)](#pythonexchangeh-ticaret-sistemi-başlık-dosyası)
* [`PythonExchange.cpp` (Ticaret Sistemi Implementasyonu)](#pythonexchangecpp-ticaret-sistemi-implementasyonu)
* [`PythonExchangeModule.cpp` (Python `exchange` Modülü)](#pythonexchangemodulecpp-python-exchange-modülü)
* [`PythonFlyModule.cpp` (Python `fly` Modülü)](#pythonflymodulecpp-python-fly-modülü)
* [`PythonGameEventManager.h` (Oyun İçi Etkinlik Yöneticisi Başlık Dosyası)](#pythongameeventmanagerh-oyun-içi-etkinlik-yöneticisi-başlık-dosyası)

--- 

### `PythonConfigModule.cpp` (Python `cfg` Modülü)

Bu dosya, `ENABLE_CONFIG_MODULE` makrosu aktif olduğunda derlenir ve `CPythonConfig` sınıfının işlevlerini Python betiklerine `cfg` adlı bir modül aracılığıyla sunar. Bu modül, oyun yapılandırma dosyalarından (genellikle `.ini` formatında) ayar okuma, yazma ve bölüm silme işlemlerinin Python üzerinden yapılmasını sağlar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`cfg`)**: `CPythonConfig` sınıfının .ini dosyası yönetimi işlevlerini Python'a açan `cfg` modülünü oluşturur.
*   **Yapılandırma Dosyası İşlemleri**:
    *   Yapılandırma dosyasını başlatma/yükleme (`Init`).
    *   Belirli bir bölüme (section) ve anahtara (key) değer yazma (`Set`). Hem string hem de integer değerleri destekler.
    *   Belirli bir bölüm ve anahtardan string değer okuma (`Get`). Okuma sırasında varsayılan bir değer belirtilebilir.
    *   Belirli bir bölümü tüm içeriğiyle silme (`Remove`).
*   **Sabitlerin Aktarılması**: `CPythonConfig` içinde tanımlanmış yapılandırma bölümlerini Python modülüne `SAVE_GENERAL`, `SAVE_CHAT`, `SAVE_OPTION` gibi tamsayı sabitleri olarak aktarır.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initcfg()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `cfg` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyon adlarını (örn. "Init", "Set", "Get", "Remove") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `cfgInit`, `cfgSet`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("cfg", s_methods);` ile `cfg` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak C++ tarafındaki bölüm enumları (`CPythonConfig::CLASS_GENERAL` vb.) Python modülüne (`cfg.SAVE_GENERAL` vb.) sabit olarak eklenir.

*   **Fonksiyon Sarmalayıcıları (`cfg*` önekli fonksiyonlar)**:
    *   `cfgInit(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir dosya adı (string) alır ve `CPythonConfig::Instance().Initialize(szFileName)` çağırarak belirtilen .ini dosyasını yükler/başlatır.
    *   `cfgSet(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir bölüm türü (byte), bir anahtar (string) ve bir değer (string veya integer) alır. Değerin türüne göre `CPythonConfig::Instance().Write()` metodunun uygun overload'unu çağırır.
    *   `cfgGet(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir bölüm türü (byte), bir anahtar (string) ve bir varsayılan değer (string) alır. `CPythonConfig::Instance().GetString()` ile değeri okur ve Python'a string olarak döndürür.
    *   `cfgRemove(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir bölüm türü (byte) alır ve `CPythonConfig::Instance().RemoveSection()` ile o bölümü siler.

#### Oyunda Kullanım Amacı ve Senaryoları

`cfg` modülü, `CPythonConfig` ile birlikte çalışarak, oyun ayarlarının kalıcı olarak saklanması ve Python üzerinden esnek bir şekilde yönetilmesi için temel bir altyapı sunar.

*   **Ayarların Yüklenmesi**: Oyun başlangıcında, `cfg.Init("system.cfg")` çağrılarak yapılandırma dosyası yüklenir.
*   **Ayarların Okunması**: Python scriptleri, `cfg.Get(cfg.SAVE_OPTION, "ResolutionWidth", "1024")` gibi komutlarla belirli ayarları okuyabilir.
*   **Ayarların Kaydedilmesi**: Oyuncu arayüzünden yapılan değişiklikler (örneğin, grafik kalitesi, ses seviyesi) `cfg.Set(cfg.SAVE_OPTION, "ShadowLevel", 2)` gibi komutlarla dosyaya yazılır.
*   **Bölüm Sıfırlama**: Belirli bir ayar grubunu varsayılan değerlerine döndürmek için `cfg.Remove(cfg.SAVE_CHAT)` gibi bir komutla ilgili bölüm silinebilir (bu işlemden sonra genellikle varsayılan ayarlar tekrar yazılır).

Bu modül, `CPythonConfig` ile birlikte çalışarak, oyun ayarlarının kalıcı olarak saklanması ve Python üzerinden esnek bir şekilde yönetilmesi için temel bir altyapı sunar.

---

### `PythonCubeRenewal.cpp` (Yenilenmiş Küp Sistemi Python Modülü)

Bu dosya, `ENABLE_CUBE_RENEWAL` makrosu aktif olduğunda derlenir ve Metin2'deki "Küp Sistemi"nin yenilenmiş bir versiyonu için istemci tarafı mantığını ve Python arayüzünü (`cuberenewal` modülü) içerir. Bu sistem, oyuncuların belirli eşyaları bir NPC aracılığıyla birleştirerek yeni eşyalar veya ödüller elde etmesini sağlar. Dosya, küp tariflerinin bir metin dosyasından (`cube_renewal.txt` gibi) yüklenmesini, bu tariflerin Python'a sunulmasını ve sunucuyla iletişim kurarak eşya birleştirme işlemlerinin gerçekleştirilmesini yönetir.

#### Dosyanın Genel Amaçları

*   **Küp Tarifi Yükleme**: `cube_renewal.txt` (veya benzeri) bir dosyadan küp üretim tariflerini okur ve bellekte saklar.
    *   Tarif dosyası formatı: Her bir üretim (`section` ile başlar, `end` ile biter) için NPC VNUM'ları, gerekli materyaller (item vnum, count), ödül(ler) (item vnum, count), başarı yüzdesi, altın maliyeti, kategori, ve diğer özel parametreler (not_remove, set_value, gem, allow_copy) tanımlanır.
*   **Veri Doğrulama**: Yüklenen tariflerdeki temel verilerin (VNUM'lar, sayılar) geçerli olup olmadığını kontrol eder (`FN_check_cube_data`).
*   **Python Modülü Oluşturma (`cuberenewal`)**: Küp sistemi işlevlerini Python'a açan `cuberenewal` modülünü oluşturur.
*   **Python Arayüzü Sağlama**:
    *   Belirli bir NPC için mevcut küp tariflerini listeleme (`GetInfo`).
    *   Sunucuya küp birleştirme isteği gönderme (`SendRefine`).
    *   Küp penceresini kapatma paketi gönderme (`SendClosePacket`).
*   **Yardımcı Fonksiyonlar**: Eşya isimlerini alma (`szItemName`) ve bir eşyanın istiflenebilir olup olmadığını kontrol etme (`IsStackableItem`).

#### C++ Implementasyon Detayları

*   **`CUBE_DATA` Yapısı**: Bir küp tarifinin tüm bilgilerini (NPC VNUM listesi, materyal listesi, ödül listesi, yüzde, altın, kategori vb.) tutan C++ yapısı.
*   **`s_cube_proto` (static `std::vector<CUBE_DATA*>`):** Yüklenen tüm küp tariflerini saklayan statik bir vektör.
*   **`LoadFile(const char* szFileName)`**:
    *   Belirtilen dosyayı `CEterPackManager` aracılığıyla yükler.
    *   Dosya içeriğini satır satır okur (`std::stringstream`, `std::getline`).
    *   `split` yardımcı fonksiyonu ile satırları tab karakterine (`\t`) göre böler.
    *   Her bir `section` bloğu için yeni bir `CUBE_DATA` nesnesi oluşturur ve ilgili bilgileri (npc, item, reward, percent, gold, category vb.) bu nesneye doldurur.
    *   `end` ile biten her bloğun sonunda `FN_check_cube_data` ile temel bir doğrulama yapar ve geçerliyse `s_cube_proto` vektörüne ekler.
*   **`FN_check_cube_data(CUBE_DATA* cube_data)`**: Bir `CUBE_DATA` nesnesindeki NPC VNUM'larının, materyal VNUM ve sayılarının, ödül VNUM ve sayılarının sıfırdan büyük olup olmadığını kontrol eder.
*   **`szItemName(uint32_t dwVnum)` ve `IsStackableItem(uint32_t dwVnum)`**:
    *   `CItemManager` singleton'unu kullanarak verilen VNUM'a ait eşyanın adını alır veya istiflenebilir olup olmadığını kontrol eder.
    *   `szItemName` ayrıca eşya ismindeki '+' karakterinden sonrasını kırpar.
*   **Singleton Yapısı**: `CPythonCubeRenewal` sınıfı, `CSingleton` şablonu ile implemente edilmese de, Python modülü üzerinden erişilen fonksiyonları statik bir `s_cube_proto` veri yapısını kullandığı ve global fonksiyonlar (`cuberenewal*`) aracılığıyla çağrıldığı için benzer bir global erişim mantığına sahiptir. `CPythonCubeRenewal::Instance()` gibi bir yapı olmasa da, `GetDataVector()` gibi metotlar `CPythonCubeRenewal` nesnesi üzerinden çağrılır (muhtemelen Python modülünün bir yerinde bir global `CPythonCubeRenewal` nesnesi tutuluyordur veya fonksiyonlar statik üye fonksiyonlarıdır).

#### Python Arayüzü Implementasyon Detayları (`cuberenewal` Modülü)

*   **`initcuberenewal()` Fonksiyonu**:
    *   `cuberenewal` Python modülünü oluşturur.
    *   `s_methods` dizisi ile Python fonksiyon adlarını (`GetInfo`, `SendRefine`, `SendClosePacket`) C sarmalayıcı fonksiyonlarına (`cuberenewalGetInfo`, `cuberenewalSendRefine`, `cuberenewalSendClosePacket`) eşler.

*   **Python Fonksiyon Sarmalayıcıları**:
    *   `cuberenewalGetInfo(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir NPC VNUM'u alır.
        *   `CPythonCubeRenewal::Instance().GetDataVector()` (veya benzeri bir yolla `s_cube_proto` erişimi) ile tüm küp tariflerini gezer.
        *   Verilen NPC VNUM'u ile eşleşen tarifleri bulur.
        *   Her eşleşen tarif için bir Python sözlüğü (`PyDict_New`) oluşturur ve içine NPC VNUM'u, materyaller (item0, item1...), ödül, yüzde, altın, kategori, mücevher maliyeti (`gem`), `set_value` bayrağı, ödülün istiflenebilir olup olmadığı (`can_stack`), tarifin kopyalanabilir olup olmadığı (`allow_copy`) ve ödül eşyasının adını (`name`) ekler.
        *   Bu sözlükleri bir Python listesine (`PyList_New`) ekleyerek geri döndürür.
    *   `cuberenewalSendRefine(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan birleştirilecek eşyanın VNUM'unu (`vnum`), kaç adet üretileceğini (`multiplier`), geliştirme (improve) slot indeksini (`indexImprove`) ve gerekli materyallerin en fazla 5 adedinin slot pozisyonlarını (`itemReq` dizisi) alır.
        *   `CPythonNetworkStream::Instance().SendCubeRefinePacket()` ile bu bilgileri sunucuya gönderir.
    *   `cuberenewalSendClosePacket(PyObject* poSelf, PyObject* poArgs)`:
        *   `CPythonNetworkStream::Instance().SendCubeRenewalClosePacket()` ile küp penceresinin kapatıldığına dair paketi sunucuya gönderir.

#### Oyunda Kullanım Amacı ve Senaryoları

`cuberenewal` modülü, yenilenmiş küp sisteminin kullanıcı arayüzü (UI) ile etkileşimini sağlar.

*   **Küp Arayüzünün Doldurulması**: Oyuncu bir küp NPC'si ile konuştuğunda, UI scripti `cuberenewal.GetInfo(npcVnum)` çağırarak o NPC için mevcut tüm üretim tariflerini alır. Bu bilgi, UI'da bir liste veya sekmeler halinde oyuncuya sunulur. Her tarifin materyalleri, ödülü, başarı şansı ve maliyeti gösterilir.
*   **Üretim İşlemi**: Oyuncu bir tarif seçip gerekli materyalleri (envanterindeki slotlardan) ve miktarını belirledikten sonra "Üret" veya "Birleştir" butonuna tıklar.
    *   UI scripti, oyuncunun envanterindeki ilgili slot pozisyonlarını ve seçilen tarife göre diğer parametreleri toplayarak `cuberenewal.SendRefine(...)` fonksiyonunu çağırır.
    *   Sunucu bu isteği işler, başarı/başarısızlık durumuna göre envanteri günceller ve sonucu istemciye bildirir.
*   **Pencere Kapatma**: Oyuncu küp arayüzünü kapattığında, `cuberenewal.SendClosePacket()` çağrılarak sunucuya bilgi verilir. Bu, sunucu tarafında ilgili NPC ile etkileşimin sonlandırılması için gerekebilir.
*   **Veri Yönetimi**: Küp tarifleri `cube_renewal.txt` gibi harici bir dosyada tutulduğu için, oyunun güncellenmesi veya yeni tariflerin eklenmesi istemci yaması gerektirmeden sadece bu dosyanın güncellenmesiyle yapılabilir. İstemci başladığında veya gerektiğinde bu dosya okunarak tarifler yüklenir.

Bu sistem, eski küp sistemine göre daha esnek ve yönetilebilir bir yapı sunmayı amaçlar. Özellikle tariflerin metin tabanlı bir dosyadan yüklenmesi ve kategori gibi ek özelliklerin desteklenmesi, geliştiricilere ve oyun tasarımcılarına daha fazla kontrol imkanı tanır.

---

### `PythonCubeRenewal.h` (Yenilenmiş Küp Sistemi Başlık Dosyası)

Bu başlık dosyası, `ENABLE_CUBE_RENEWAL` makrosu aktif olduğunda derlenir ve yenilenmiş Küp Sistemi'nin temel veri yapılarını ve `CPythonCubeRenewal` singleton sınıfının bildirimini içerir. `CPythonCubeRenewal.cpp` dosyasındaki implementasyonun ve `cuberenewal` Python modülünün altyapısını oluşturur.

#### Dosyanın Genel Amaçları

*   **Veri Yapıları Tanımlama**:
    *   `CUBE_VALUE`: Bir eşyanın VNUM'unu ve sayısını (count) tutan basit bir yapıdır. Eşitlik operatörü (`operator==`) overload edilmiştir.
    *   `CUBE_DATA`: Bir küp üretim tarifinin tüm detaylarını içeren ana yapıdır. Şunları barındırır:
        *   `npc_vnum`: Bu tarifi sunabilecek NPC'lerin VNUM'larını içeren bir `std::vector<int32_t>`.
        *   `item`: Gerekli materyalleri (`CUBE_VALUE` türünde) tutan bir `std::vector`.
        *   `reward`: Üretim sonucu elde edilecek ödül(ler)i (`CUBE_VALUE` türünde) tutan bir `std::vector`.
        *   `percent`: Üretimin başarı şansını (yüzde olarak) tutan bir `int`.
        *   `gold`: Üretim için gerekli altın miktarını tutan bir `int32_t`.
        *   `gem`: Üretim için (varsa) ek bir para birimi/nesne maliyetini (örn. "ejderha taşı parçası") tutan bir `int32_t`.
        *   `allow_copy`: Tarifin kopyalanıp kopyalanamayacağını belirten bir `bool` (muhtemelen çoklu üretim için).
        *   `category`: Tarifin kullanıcı arayüzünde hangi kategori altında gösterileceğini belirten bir `std::string`.
        *   `not_remove`: Üretim sırasında materyallerin silinip silinmeyeceğini belirten bir `int` (muhtemelen bir bayrak).
        *   `set_value`: Özel bir değeri veya bayrağı tutan bir `int` (kullanım amacı implementasyona bağlıdır).
        *   Yapıcı (`CUBE_DATA()`) `set_value` ve `gem` üyelerini 0 olarak başlatır.
*   **`CPythonCubeRenewal` Sınıf Bildirimi**:
    *   `CSingleton<CPythonCubeRenewal>`'dan miras alır, bu da sınıfın global olarak tek bir örnekle erişilebilir olmasını sağlar.
    *   `TInfoDateCubeRenewal` (bildirimde `TInfoStrucCubeRenewal` olarak yanlış yazılmış olabilir, genellikle `TCubeResult` gibi bir şey olur) türünden bir `std::vector` için `typedef` tanımlar. Ancak bu `typedef` dosya içinde doğrudan kullanılmıyor gibi görünmektedir ve `TInfoDateCubeRenewal`'ın tanımı bu başlıkta mevcut değildir.
    *   Yapıcı (`CPythonCubeRenewal()`) ve yıkıcı (`~CPythonCubeRenewal()`) metotları bildirir.
    *   `FN_check_cube_data(CUBE_DATA* cube_data)`: Bir küp tarifinin temel doğruluğunu kontrol eden fonksiyonu bildirir.
    *   `LoadFile(const char* szFileName)`: Küp tarif dosyasını yükleyen fonksiyonu bildirir.
    *   `SetCubeRenewalHandler(PyObject* _CubeRenewalHandler)`: Bir Python nesnesini (muhtemelen UI eventlerini işleyecek bir callback veya handler) kaydetmek için bir metot bildirir.
    *   `GetDataVector()`: Yüklenmiş tüm küp tariflerini (`std::vector<CUBE_DATA*>`) döndüren bir metot bildirir.
    *   **Özel Üyeler**:
        *   `s_cube_proto`: `std::vector<CUBE_DATA*>` türünde, yüklenen tüm küp tariflerini saklayan (muhtemelen statik olması gereken ancak bildirimde statik olmayan) bir üye değişken. `CPythonCubeRenewal.cpp` dosyasındaki implementasyonda bu değişken statiktir.
        *   `m_CubeRenewalHandler`: Kaydedilen Python nesnesini tutan bir `PyObject*`.

#### C++ ve Python Arayüzü ile İlişkisi

*   Bu başlık dosyasında tanımlanan `CUBE_DATA` ve `CUBE_VALUE` yapıları, `CPythonCubeRenewal.cpp` içindeki `LoadFile` fonksiyonu tarafından tarif dosyası okunurken kullanılır.
*   `CPythonCubeRenewal` sınıfının metotları (`LoadFile`, `GetDataVector`), `cuberenewal` Python modülünün (`PythonCubeRenewal.cpp` içinde tanımlanan) C sarmalayıcı fonksiyonları (`cuberenewalGetInfo` vb.) tarafından çağrılır. Örneğin, `cuberenewalGetInfo`, `GetDataVector()` ile tarifleri alır ve Python sözlüklerine dönüştürür.
*   `s_cube_proto` (veya .cpp dosyasındaki statik versiyonu), yüklenen tariflerin merkezi depolama alanıdır.
*   `m_CubeRenewalHandler`, Python tarafındaki UI scriptleriyle (örneğin, olay bildirimleri veya özel UI güncellemeleri için) bir köprü kurmak amacıyla kullanılabilir, ancak `PythonCubeRenewal.cpp` dosyasındaki mevcut Python fonksiyonları doğrudan bu handler'ı kullanmıyor gibi görünmektedir. Daha çok, Python'dan C++'a veri akışı ve komut gönderme üzerine odaklanılmıştır.

#### Derleme Bağımlılıkları

*   `Packet.h`: Muhtemelen ağ paketleriyle ilgili temel tanımları içerir, ancak bu başlık dosyasında doğrudan kullanılmamıştır.
*   `ENABLE_CUBE_RENEWAL`: Tüm dosya içeriği bu makro ile çevrelenmiştir, yani bu özellik aktif değilse dosya hiçbir şey derlemeyecektir.

Bu başlık dosyası, Yenilenmiş Küp Sistemi'nin C++ tarafındaki veri modelini ve ana sınıf arayüzünü tanımlayarak, hem tariflerin yüklenip saklanması hem de Python modülü aracılığıyla bu verilere erişilip sunucuyla etkileşim kurulması için gerekli temeli sağlar.

---

### `PythonEffectModule.cpp` (Python `effect` Modülü)

Bu dosya, Metin2 istemcisindeki görsel efektlerin (`CEffectManager`) ve uçan nesne/efektlerin (`CFlyingManager`) Python betikleri (`effect` modülü) aracılığıyla yönetilmesini sağlar. Efektlerin kaydedilmesi, oluşturulması, güncellenmesi, çizdirilmesi, pozisyonlarının ayarlanması ve silinmesi gibi temel işlevleri Python'a açar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`effect`)**: `CEffectManager` ve `CFlyingManager` sınıflarının işlevlerini Python'a `effect` adında bir modül ile sunar.
*   **Efekt Yönetimi**:
    *   Efekt dosyalarını (`.mse` uzantılı olabilir) sisteme kaydetme (`RegisterEffect`).
    *   Kayıtlı bir efektten yeni bir örnek oluşturma (`CreateEffect`). Bu işlem genellikle seçili karakterin pozisyonunda bir efekt yaratır.
    *   Oluşturulmuş bir efekt örneğini silme (`DeleteEffect`).
    *   Bir efekt örneğinin dünyadaki pozisyonunu ayarlama (`SetPosition`).
    *   Tüm efektlerin güncellenmesi (`Update`) ve çizdirilmesi (`Render`) için genel çağrılar sunar (genellikle oyun döngüsü içinde `CPythonApplication` tarafından çağrılır, ancak Python'dan da tetiklenebilir).
*   **Uçan Nesne/Efekt Yönetimi (`CFlyingManager` ile ilgili)**:
    *   Belirli bir indeks ve tipe göre uçan nesne/efekt verisi kaydetme (`RegisterIndexedFlyData`). Bu, genellikle oklar, büyü mermileri veya deneyim/can/mana küreleri gibi hedefe doğru hareket eden görsel elementler için kullanılır.
*   **Sabitlerin Aktarılması**: Uçan nesne/efekt türleri (`INDEX_FLY_TYPE_*` ve `FLY_*`) gibi C++ sabitlerini Python modülüne aktarır, böylece scriptler bu sabitleri isimleriyle kullanabilir.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initeffect()` Fonksiyonu**:
    *   Python yorumlayıcısı `effect` modülünü ilk kez yüklediğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyon adlarını (örn. "RegisterEffect", "CreateEffect") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `effectRegisterEffect`, `effectCreateEffect`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("effect", s_methods);` ile `effect` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak `CFlyingManager` ve `CInstanceBase` (karakter örneği) sınıflarında tanımlı çeşitli uçan nesne türü sabitleri (örn. `CFlyingManager::INDEX_FLY_TYPE_NORMAL` -> `effect.INDEX_FLY_TYPE_NORMAL`, `CInstanceBase::FLY_EXP` -> `effect.FLY_EXP`) Python modülüne eklenir.

*   **Fonksiyon Sarmalayıcıları (`effect*` önekli fonksiyonlar)**:
    *   `effectRegisterEffect(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir efekt dosya adı (string) alır ve `CEffectManager::Instance().RegisterEffect()` ile efekti kaydeder.
    *   `effectCreateEffect(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir efekt adı (string) alır. `CPythonCharacterManager::Instance().GetSelectedInstancePtr()` ile o anda seçili olan karakter örneğini alır, bu karakterin pozisyonunu (`pInstance->NEW_GetPixelPosition()`) kullanarak `CEffectManager::Instance().CreateEffect()` ile efekti oluşturur. Oluşturulan efekt örneğinin ID'sini (index) Python'a integer olarak döndürür.
    *   `effectDeleteEffect(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir efekt örnek ID'si (integer) alır ve `CEffectManager::Instance().DestroyEffectInstance()` ile efekti siler.
    *   `effectSetPosition(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir efekt örnek ID'si (integer) ve X, Y, Z koordinatlarını (float) alır. `CEffectManager::Instance().SelectEffectInstance()` ve ardından `CEffectManager::Instance().SetEffectInstancePosition()` ile efektin pozisyonunu ayarlar.
    *   `effectUpdate(PyObject* poSelf, PyObject* poArgs)`: `CEffectManager::Instance().Update()` çağırır.
    *   `effectRender(PyObject* poSelf, PyObject* poArgs)`: `CEffectManager::Instance().Render()` çağırır.
    *   `effectRegisterIndexedFlyData(PyObject* poSelf, PyObject* poArgs)`: Python'dan bir indeks (integer), bir tip (integer) ve bir uçan nesne veri adı (string) alır. `CFlyingManager::Instance().RegisterIndexedFlyData()` ile bu veriyi kaydeder.

#### Oyunda Kullanım Amacı ve Senaryoları

`effect` modülü, oyun içindeki görsel efektlerin Python scriptleri aracılığıyla dinamik olarak yönetilmesini sağlar. Bu, genellikle yetenek kullanımları, karakter durum değişiklikleri, çevresel efektler ve kullanıcı arayüzü geri bildirimleri için kullanılır.

*   **Yetenek Efektleri**: Bir oyuncu veya NPC bir yetenek kullandığında, ilgili Python scripti `effect.CreateEffect("yetenek_efekti_adi")` çağırarak yeteneğin görselini oluşturabilir. Örneğin, bir ateş topu büyüsü yapıldığında, karakterin elinde bir ateş efekti belirebilir.
*   **Durum Efektleri**: Zehirlenme, sersemleme, güçlendirme (buff) gibi durumlar aktif olduğunda, karakterin etrafında veya üzerinde sürekli bir efekt (`effect.CreateEffect`) gösterilebilir. Durum sona erdiğinde efekt `effect.DeleteEffect` ile kaldırılır.
*   **Çevresel Efektler**: Belirli haritalarda veya olaylarda özel atmosferik efektler (örn. sis, parlayan portallar) bu modül kullanılarak tetiklenebilir.
*   **Uçan Nesneler (Skill Mermileri, Oklar vb.)**:
    *   Yetenek scriptleri veya karakter saldırı mantığı, belirli bir hedefe doğru hareket eden görsel bir mermi (uçan nesne) oluşturmak için `CInstanceBase` sınıfının `AttackProcess()` gibi fonksiyonları içinde `CFlyingManager`'ı kullanır. `effect.RegisterIndexedFlyData` ile bu uçan nesnelerin görsel tanımları (örn. "arrow.msf", "fireball_trail.mse") önceden sisteme kaydedilir.
    *   Örneğin, bir okçu saldırdığında, `FLY_QUIVER_ATTACK_NORMAL` gibi bir sabit kullanılarak uygun uçan ok efekti tetiklenir.
*   **Deneyim/Can/Mana Küreleri**: Bir canavar öldüğünde veya bir iksir kullanıldığında, oyuncuya doğru uçan deneyim (`effect.FLY_EXP`), can (`effect.FLY_HP_BIG`) veya mana (`effect.FLY_SP_BIG`) küreleri `CFlyingManager` tarafından yönetilir ve bu modüldeki sabitler aracılığıyla tanımlanır.
*   **UI Geri Bildirimleri**: Bir butona tıklandığında veya önemli bir olay gerçekleştiğinde anlık görsel efektler (parlama, vurgu vb.) gösterilebilir.
*   **Efektlerin Yüklenmesi**: Oyunun başlangıcında veya belirli bir seviye/harita yüklendiğinde, gerekli tüm efekt dosyaları `effect.RegisterEffect` kullanılarak toplu halde kaydedilir. Bu genellikle `loading.py` veya benzeri bir Python scriptinde yapılır.

Bu modül, `CEffectManager` ve `CFlyingManager` ile sıkı bir şekilde entegre çalışarak, oyun dünyasının görsel canlılığını ve yeteneklerin/etkileşimlerin görsel geri bildirimlerini zenginleştiren önemli bir araçtır.

--- 

### `PythonEventManager.h` (Görev/Etkinlik Yöneticisi Başlık Dosyası)

Bu başlık dosyası, Metin2 istemcisindeki görev (quest) ve çeşitli olay (event) scriptlerini yöneten `CPythonEventManager` singleton sınıfının bildirimini içerir. Görev scriptlerinin yüklenmesi, işlenmesi, metinlerin gösterilmesi, oyuncu etkileşimlerinin (cevaplar, butonlar) yönetilmesi ve çeşitli script komutlarının (kamera hareketleri, resim gösterme, mini harita işaretleri vb.) ele alınması için gerekli yapıları ve fonksiyon prototiplerini tanımlar.

#### Dosyanın Genel Amaçları

*   **`CPythonEventManager` Sınıf Bildirimi**: Singleton olarak tasarlanmış ana olay yöneticisi sınıfını deklare eder.
*   **Veri Yapıları Tanımlama**:
    *   `STextLine`: Görev penceresinde gösterilecek tek bir metin satırının yerel pozisyonunu (`ixLocal`, `iyLocal`) ve `CGraphicTextInstance` işaretçisini tutar.
    *   `TScriptTextLineList`: `STextLine` nesnelerinin bir `std::list`'idir; bir görev penceresindeki tüm metin satırlarını saklar.
    *   `TEventAnswerMap`: (Bu başlıkta tanımlı ama doğrudan kullanılmıyor gibi duruyor, muhtemelen eski bir kalıntı veya başka bir yerde kullanılıyor) Soru-cevap eşleştirmesi için bir `std::map`.
    *   `SEventSet`: Tek bir olay/görev script'i setini temsil eden ana yapıdır. İçerisinde şunları barındırır:
        *   Pozisyon (`ix`, `iy`), genişlik (`iWidth`), yerel Y pozisyonu (`iyLocal`).
        *   Durum bilgileri: Kilitli mi (`isLock`), son gecikme süresi (`lLastDelayTime`), mevcut harf indeksi (`iCurrentLetter`), geçerli metin rengi (`CurrentColor`), o anki satır metni (`strCurrentLine`).
        *   Metin satırı yönetimi: O anki metin satırı (`pCurrentTextLine`), tüm metin satırlarının listesi (`ScriptTextLineList`).
        *   Onay bekleme durumu (`isConfirmWait`), onay süresi için metin satırı (`pConfirmTimeTextLine`), onay bitiş zamanı (`iConfirmEndTime`).
        *   Script verisi (`ScriptGroup`), dosya adı (`szFileName`).
        *   Görünürlük ayarları (`iVisibleStartLine`, `iVisibleLineCount`).
        *   Toplam satır sayısı (`iTotalLineCount`), ayarlanmış satır sayısı (`iAdjustLine`).
        *   Varsayılan renk (`DiffuseColor`), varsayılan bekleme süresi (`lWaitingTime`), bir satırdaki maksimum karakter sayısı (`iRestrictedCharacterCount`).
        *   Soru cevap sayısı (`nAnswer`).
        *   Metin hizalama modu (`isTextCenterMode`), bekleme bayrağı (`isWaitFlag`).
        *   Bu olay setini işleyecek Python olay işleyicisi (`poEventHandler`).
    *   `TEventSetVector`: `SEventSet*` işaretçilerinin bir `std::vector`'üdür; tüm aktif olay setlerini tutar.
*   **Enum Tanımları**:
    *   `EVENT_POSITION_START`, `EVENT_POSITION_END`: Script komutlarının başlangıç mı yoksa bitiş etiketi mi olduğunu belirtir.
    *   `BOX_VISIBLE_LINE_COUNT`: Bir görev penceresinde aynı anda görünür olan maksimum satır sayısı.
    *   `EButtonType`: Görev penceresindeki buton türleri (İLERİ, BİTTİ, İPTAL).
    *   `EEventType`: Çok sayıda script komut türünü tanımlar (örneğin `EVENT_TYPE_LETTER`, `EVENT_TYPE_COLOR`, `EVENT_TYPE_WAIT`, `EVENT_TYPE_QUESTION`, `EVENT_TYPE_NEXT`, `EVENT_TYPE_IMAGE`, `EVENT_TYPE_SET_CAMERA`, `EVENT_TYPE_FADE_OUT`, `EVENT_TYPE_ITEM_NAME`, `EVENT_TYPE_INPUT` vb.).
*   **Fonksiyon Prototipleri**: Olay setlerini kaydetme, silme, güncelleme, render etme, script komutlarını işleme, metin ekleme, buton oluşturma, cevap seçme gibi işlemler için fonksiyonları bildirir.
*   **Üye Değişkenler**: `EventTypeMap` (string komut adlarını `EEventType` enum değerlerine eşler), `m_EventSetVector`, `m_EventSetPool`, `m_ScriptTextLinePool`, `m_poInterface` (genel Python arayüz nesnesi), `m_strLeftTimeString` (kalan süre metin formatı).

#### C++ ve Python Arayüzü ile İlişkisi

*   `CPythonEventManager`, görev script dosyalarını (.quest, .txt) okur, `script::Group` (muhtemelen `EterLib/parser.h` içinden gelen bir sınıf) kullanarak bunları komutlara ayırır ve `SEventSet` yapılarında saklar.
*   Python tarafındaki UI scriptleri (örn. `questevent.py`, `interfaceModule.py`), `CPythonEventManager`'ın fonksiyonlarını çağırarak görev pencerelerini oluşturur, olay setlerini kaydeder (`RegisterEventSet`), olay işleyicilerini (`poEventHandler`) atar ve görev akışını kontrol eder.
*   `SEventSet::poEventHandler`, C++ tarafındaki olayların (örn. buton tıklaması, fade işleminin bitmesi) Python tarafındaki ilgili UI fonksiyonlarını tetiklemesi için kullanılır (`PyCallClassMemberFunc`).
*   `EEventType` içinde tanımlanan her bir komut türü, `ProcessEventSet` fonksiyonunda özel bir işlemle ele alınır. Bu işlemler arasında metin ekleme, renk değiştirme, gecikme uygulama, Python UI fonksiyonlarını çağırma (resim gösterme, buton oluşturma, kamera ayarı yapma) gibi görevler bulunur.
*   `m_poInterface`, genellikle `interfaceModule.py` gibi ana UI kontrol scriptinin bir örneğini tutar ve genel UI işlemleri (örn. görev listesini güncelleme, zindan sonucu gösterme) için kullanılır.

#### Derleme Bağımlılıkları

*   `EterLib/parser.h`: Script dosyalarını okumak ve komutlara ayırmak için kullanılır.
*   `ENABLE_ATTR_6TH_7TH`, `ENABLE_GEM_SYSTEM` gibi makrolar, belirli script komut türlerinin (`EVENT_TYPE_INSERT_ITEM_TOOLTIP_BY_CELL`, `EVENT_TYPE_SELECT_ITEM_EX`) derlenip derlenmeyeceğini kontrol eder.

Bu başlık dosyası, oyunun görev ve olay sisteminin kalbidir. Script tabanlı görevlerin ve ara sahnelerin (cutscenes) yönetilmesi, oyuncuyla etkileşimli diyalogların sunulması ve çeşitli oyun içi olayların tetiklenmesi için karmaşık ama esnek bir yapı sunar.

--- 

### `PythonEventManager.cpp` (Görev/Etkinlik Yöneticisi Implementasyonu)

Bu dosya, `CPythonEventManager` sınıfının ve `SEventSet` yapısıyla ilişkili tüm fonksiyonların C++ implementasyonlarını içerir. Görev scriptlerinin yüklenmesi, satır satır işlenmesi, metinlerin biçimlendirilip gösterilmesi, kullanıcı etkileşimlerinin (butonlar, cevaplar) yönetilmesi ve script komutlarının (kamera kontrolü, resimler, mini harita sinyalleri, fade efektleri vb.) yürütülmesinden sorumludur.

#### Dosyanın Genel Amaçları

*   **Olay Seti Yönetimi (`SEventSet`)**:
    *   `RegisterEventSet` / `RegisterEventSetFromString`: Script dosyalarını (.quest, .txt) veya doğrudan string olarak verilen scriptleri yükler. `script::Group` kullanarak script'i ayrıştırır, bir `SEventSet` nesnesi oluşturur, bu nesneyi `m_EventSetPool`'dan alır ve `m_EventSetVector` içine kaydeder. Her olay setine benzersiz bir ID atanır.
    *   `ClearEventSeti` / `__ClearEventSetp`: Belirli bir olay setini ve onunla ilişkili tüm kaynakları (metin satırları vb.) temizler.
    *   `__InitEventSet`: Yeni bir olay setinin başlangıç değerlerini (pozisyon, renk, sayaçlar vb.) ayarlar.
*   **Script İşleme (`ProcessEventSet`)**: Aktif bir olay setindeki script komutlarını sırayla işler.
    *   `ScriptGroup.GetCmd()` ile bir sonraki komutu alır.
    *   `GetScriptEventIndex` ile komut adını `EEventType` enum değerine dönüştürür.
    *   `switch` ifadesiyle her bir `EEventType` için özel bir işlem gerçekleştirir.
*   **Metin Yönetimi ve Gösterimi**:
    *   `__InsertLine`: Görev penceresine yeni bir boş metin satırı (`CGraphicTextInstance`) ekler, pozisyonunu ayarlar ve mevcut satırı geçmiş satırlar listesine (`ScriptTextLineList`) taşır.
    *   `EVENT_TYPE_LETTER`: Script'teki metni harf harf (veya kelime kelime) mevcut satıra ekler. `iRestrictedCharacterCount` aşıldığında otomatik olarak yeni satıra geçer.
    *   `EVENT_TYPE_COLOR`, `EVENT_TYPE_COLOR256`: Metin rengini değiştirir.
    *   `EVENT_TYPE_ENTER`: Yeni bir satıra geçer.
    *   `EVENT_TYPE_TEXT_HORIZONTAL_ALIGN_CENTER`: Metinleri yatayda ortalar.
    *   `EVENT_TYPE_ITEM_NAME`, `EVENT_TYPE_MONSTER_NAME`: Verilen VNUM'a ait eşya veya canavar ismini metne ekler.
    *   `UpdateEventSet`: Olay setinin pozisyonunu günceller, onay bekleme süresini işler ve `ProcessEventSet`'i çağırarak script'i ilerletir.
    *   `RenderEventSet`: Görünür olan metin satırlarını ekrana çizer.
*   **Kullanıcı Etkileşimi**:
    *   `EVENT_TYPE_WAIT`: Script işlemesini durdurur, oyuncunun bir sonraki adıma geçmek için (genellikle bir butona tıklayarak) etkileşimde bulunmasını bekler.
    *   `EVENT_TYPE_NEXT`, `EVENT_TYPE_DONE`: "İleri" veya "Bitti" butonlarını oluşturmak için Python tarafındaki `poEventHandler->MakeNextButton` fonksiyonunu çağırır.
    *   `EVENT_TYPE_QUESTION`: Soru ve cevap seçeneklerini (`poEventHandler->MakeQuestion`, `poEventHandler->AppendQuestion`) oluşturur.
    *   `SelectAnswer`: Oyuncunun seçtiği cevabı sunucuya gönderir (`CPythonNetworkStream::SendScriptAnswerPacket`).
    *   `EVENT_TYPE_INPUT`: Python tarafında bir girdi alanı (`poEventHandler->OnInput`) oluşturulmasını tetikler.
    *   `EVENT_TYPE_CONFIRM_WAIT`: Belirli bir süre içinde onay bekleyen bir pencere oluşturur, kalan süreyi gösterir ve İPTAL butonu ekler.
*   **Görsel ve Diğer Efektler**:
    *   `EVENT_TYPE_LEFT_IMAGE`, `EVENT_TYPE_TOP_IMAGE`, `EVENT_TYPE_BACKGROUND_IMAGE`, `EVENT_TYPE_IMAGE`, `EVENT_TYPE_TITLE_IMAGE`: Python tarafındaki `poEventHandler` aracılığıyla görev penceresinde veya ekranda resimler gösterir.
    *   `EVENT_TYPE_INSERT_IMAGE`: Eşya ikonu veya özel bir resmi, başlık ve açıklama ile birlikte gösterir.
    *   `EVENT_TYPE_ADD_MAP_SIGNAL`, `EVENT_TYPE_CLEAR_MAP_SIGNAL`: Mini haritada işaretçi ekler veya temizler (`CPythonMiniMap`).
    *   `EVENT_TYPE_SET_CAMERA`, `EVENT_TYPE_BLEND_CAMERA`, `EVENT_TYPE_RESTORE_CAMERA`: Oyun kamerasını ayarlar, animasyonlu geçiş yapar veya varsayılan ayarlara döndürür (`IAbstractApplication`).
    *   `EVENT_TYPE_FADE_OUT`, `EVENT_TYPE_FADE_IN`, `EVENT_TYPE_WHITE_OUT`, `EVENT_TYPE_WHITE_IN`: Ekranı karartma/aydınlatma efektlerini tetikler (`poEventHandler`).
*   **Yardımcı Fonksiyonlar**: `GetArgumentString` (script argümanlarını okuma), `GetCameraSettingFromArgList` (kamera ayarlarını argümanlardan çekme), `__AddSpace` (satırlar arası boşluk ekleme), `SetLeftTimeString` (kalan süre metnini ayarlama).

#### C++ Implementasyon Detayları

*   **Komut Ayrıştırma ve Yürütme**: `script::Group::Create` ile script dosyası yüklenir ve `script::TCmd` yapılarına bölünür. `ProcessEventSet` döngüsü bu komutları `script::Group::GetCmd` ile alır ve işler.
*   **Dinamik Bellek Yönetimi**: `SEventSet` nesneleri için `CDynamicPool<SEventSet> m_EventSetPool` ve `CGraphicTextInstance` (metin satırları) için `CDynamicPool<CGraphicTextInstance> m_ScriptTextLinePool` kullanılır. Bu, sık sık oluşturulup silinen bu nesneler için bellek yönetimini optimize eder.
*   **Zamanlama ve Gecikmeler**: `CTimer::Instance().GetElapsedMilliecond()` ve `pEventSet->lLastDelayTime` kullanılarak `EVENT_TYPE_DELAY` ve `EVENT_TYPE_SLEEP` komutları için gecikmeler ve `EVENT_TYPE_LETTER` için harf başına bekleme süresi (`pEventSet->lWaitingTime`) yönetilir.
*   **Python Etkileşimi**: `PyCallClassMemberFunc` kullanılarak `SEventSet::poEventHandler` (genellikle bir Python UI sınıfının örneği) ve `m_poInterface` (genel arayüz modülü) üzerindeki Python fonksiyonları çağrılır. Bu, C++ olay mantığının Python UI'ını dinamik olarak kontrol etmesini sağlar.
*   **Yerelleştirme/Kod Sayfası Desteği**: `GetDefaultCodePage() == CP_1256` (Arapça kod sayfası) gibi kontrollerle metin hizalaması gibi bazı davranışlar farklı kod sayfalarına göre ayarlanır.
*   **Hata Kontrolü**: `CheckEventSetIndex` ile geçersiz olay seti ID'lerine erişim engellenir. Script ayrıştırma hataları veya eksik kaynaklar (örn. varsayılan font) `TraceError` ile loglanır.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonEventManager`, oyunun görev sisteminin temelini oluşturur ve oyunculara sunulan interaktif hikaye anlatımının çoğunu yönetir.

*   **Görev Diyalogları**: NPC'lerle konuşulduğunda, sunucudan gelen görev script'i `RegisterEventSetFromString` ile yüklenir. `ProcessEventSet` script'i yürüterek NPC'nin konuşmalarını, oyuncunun cevap seçeneklerini (`EVENT_TYPE_QUESTION`), butonları (`EVENT_TYPE_NEXT`) ve diğer ilgili komutları işler.
*   **Ara Sahneler (Cutscenes)**: Belirli oyun olaylarında veya görev adımlarında, `EVENT_TYPE_SET_CAMERA`, `EVENT_TYPE_FADE_OUT`, `EVENT_TYPE_IMAGE` gibi komutlar kullanılarak basit ara sahneler oluşturulabilir.
*   **Bilgilendirme Pencereleri**: Oyun mekanikleri hakkında bilgi vermek, yeni bir özellik tanıtmak veya bir olayı duyurmak için resimli ve metinli pencereler (`EVENT_TYPE_IMAGE`, `EVENT_TYPE_LETTER`) gösterilebilir.
*   **Özel Oyun İçi Olaylar**: `EVENT_TYPE_RUN_CINEMA` (bu komut başka bir script dosyasını yükler), `EVENT_TYPE_DUNGEON_RESULT` gibi komutlar, daha karmaşık oyun içi olayların veya arayüzlerin tetiklenmesini sağlar.
*   **Mini Harita ve Dünya Etkileşimleri**: `EVENT_TYPE_ADD_MAP_SIGNAL` ile görev hedefleri mini haritada gösterilebilir.

Bu sınıf, script tabanlı bir yaklaşımla görevlerin ve olayların esnek bir şekilde tasarlanmasına ve uygulanmasına olanak tanır. Geliştiriciler, C++ kodunu değiştirmeden yeni görevler ve etkileşimler ekleyebilirler.

--- 

### `PythonEventManagerMoudle.cpp` (Python `event` Modülü)

Bu dosya, `CPythonEventManager` sınıfının C++ tarafındaki işlevlerini Python betiklerinin kullanabilmesi için `event` adında bir Python modülü oluşturur. Bu modül, görev (quest) ve olay (event) scriptlerinin yüklenmesi, metinlerinin gösterilmesi, kullanıcı etkileşimlerinin (butonlar, cevaplar) yönetilmesi ve script ile ilişkili çeşitli UI ve oyun içi eylemlerin Python üzerinden kontrol edilmesini sağlar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`event`)**: `CPythonEventManager` sınıfının tüm genel (public) arayüzünü Python'a `event` modülü olarak sunar.
*   **Olay Seti (Event Set) Yönetimi**:
    *   Script dosyalarını (`.quest`, `.txt` vb.) veya string halindeki scriptleri yükleyip kaydetme (`RegisterEventSet`, `RegisterEventSetFromString`).
    *   Kayıtlı bir olay setini temizleme (`ClearEventSet`).
    *   Olay setine özel bir Python olay işleyici (event handler) atama (`SetEventHandler`). Bu, genellikle görev penceresini yöneten UI sınıfının bir örneğidir.
    *   Bir olay setindeki metin satırında gösterilecek maksimum karakter sayısını ayarlama (`SetRestrictedCount`).
*   **Metin ve Konumlandırma Yönetimi**:
    *   Olay setinin genel X,Y pozisyonunu ve genişliğini ayarlama (`UpdateEventSet` - Y pozisyonunu negatifler, `SetEventSetWidth`, `SetYPosition`).
    *   Olay setinin içindeki metinlerin dikey yerleşimini (Y pozisyonunu) alma ve ayarlama (`GetEventSetLocalYPosition`, `AddEventSetLocalYPosition`).
    *   Belirli bir olay setine metin ekleme (`InsertText`, `InsertTextInline` - satır içi X pozisyonu ile).
    *   Metinlerin varsayılan yazı tipi rengini ayarlama (`SetFontColor`).
*   **Gösterim ve Güncelleme**:
    *   Belirli bir olay setini güncelleme (`UpdateEventSet` - script'i işler ve pozisyonu ayarlar).
    *   Belirli bir olay setini ekrana çizdirme (`RenderEventSet`).
    *   Olay setindeki görünür metin satırlarının başlangıç indeksini ve sayısını ayarlama/alma (`SetVisibleStartLine`, `GetVisibleStartLine`, `SetVisibleLineCount`, `GetVisibleLineCount`).
*   **Script Akış Kontrolü**:
    *   Bir olay setindeki script işlemini hızlıca geçme/atlama (`Skip`).
    *   Bir olay setinin oyuncu etkileşimi için bekleyip beklemediğini sorgulama (`IsWait`).
    *   Bekleme durumunu sonlandırma (örn. fade efekti bittiğinde) (`EndEventProcess`).
    *   Bir olay setindeki tüm script komutlarını tek seferde işleme (`AllProcesseEventSet` - `AllProcessEventSet` olarak typo var).
*   **Kullanıcı Etkileşimi**:
    *   Oyuncunun bir soruya verdiği cevabı C++ tarafına ve sunucuya bildirme (`SelectAnswer`).
    *   Görev listesindeki bir butona tıklandığında sunucuya bilgi gönderme (`QuestButtonClick`).
*   **Bilgi Alma**:
    *   Bir olay setindeki toplam işlenmiş satır sayısını alma (`GetLineCount` - aslında `GetProcessedLineCount` olmalı, `iAdjustLine`'ı da ekliyor).
    *   Bir olay setindeki tek bir satırın yüksekliğini alma (`GetLineHeight`).
    *   Script dosyasındaki toplam komut sayısını (yaklaşık satır sayısı) alma (`GetTotalLineCount`).
    *   O an işlenmiş olan (gösterilmiş) satır sayısını alma (`GetProcessedLineCount`).
*   **Genel Ayarlar ve Temizlik**:
    *   `CPythonEventManager`'ın kullanacağı genel arayüz nesnesini (Python handle) ayarlama (`SetInterfaceWindow`).
    *   Onay beklerken gösterilecek "kalan süre" metninin formatını ayarlama (`SetLeftTimeString`).
    *   `CPythonEventManager`'ın tüm kaynaklarını temizleme (`Destroy`).
*   **Sabitlerin Aktarılması**: `CPythonEventManager` içinde tanımlı `BOX_VISIBLE_LINE_COUNT` ve buton türleri (`BUTTON_TYPE_NEXT`, `BUTTON_TYPE_DONE`, `BUTTON_TYPE_CANCEL`) gibi sabitleri Python modülüne aktarır.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initEvent()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `event` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyon adlarını (örn. "RegisterEventSet", "UpdateEventSet") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `eventRegisterEventSet`, `eventUpdateEventSet`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("event", s_methods);` ile `event` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak `CPythonEventManager` sınıfındaki bazı sabitler Python modülüne (`event.BOX_VISIBLE_LINE_COUNT` vb.) aktarılır.

*   **Fonksiyon Sarmalayıcıları (`event*` önekli fonksiyonlar)**:
    *   Bu C fonksiyonları, Python'dan gelen argümanları (`PyObject* poArgs`) alır (`PyTuple_GetString`, `PyTuple_GetInteger`, `PyTuple_GetObject` vb.).
    *   `CPythonEventManager::Instance()` üzerinden ilgili C++ metodunu çağırırlar.
    *   Dönüş değerlerini (varsa) Python nesnelerine (`Py_BuildValue`, `Py_BuildNone`) dönüştürerek geri gönderirler.
    *   Örneğin, `eventUpdateEventSet` fonksiyonu Python'dan olay seti indeksi (ID), X ve Y pozisyonlarını alır. `CPythonEventManager::Instance().UpdateEventSet(iIndex, ix, -iy)` çağrısını yapar (Y koordinatını negatifler, çünkü genellikle UI koordinatları yukarıdan aşağıya artarken, oyun dünyası koordinatları aşağıdan yukarıya artabilir veya tam tersi bir durum söz konusu olabilir; bu, tutarlılık sağlamak içindir).
    *   `eventQuestButtonClick` fonksiyonu, görev butonu ID'sini alıp `CPythonNetworkStream::Instance().SendScriptButtonPacket()` ile sunucuya paket gönderir.

#### Oyunda Kullanım Amacı ve Senaryoları

`event` modülü, Python ile yazılmış kullanıcı arayüzü (UI) scriptlerinin (`questevent.py`, `interfacemodule.py` vb.) `CPythonEventManager`'ın C++ tarafındaki güçlü görev ve olay işleme yeteneklerini kullanmasını sağlar. Bu sayede, oyunun görev diyalogları, ara sahneleri ve diğer script tabanlı etkileşimleri tamamen Python üzerinden yönetilebilir.

*   **Görev Penceresi Yönetimi**: UI scriptleri, bir NPC ile konuşulduğunda veya bir görev alındığında `event.RegisterEventSetFromString(script_metni)` ile yeni bir olay seti oluşturur. Bu sete bir olay işleyici (`event.SetEventHandler(olay_seti_ID, self)`) atar; bu `self`, genellikle görev penceresini yöneten Python UI sınıfının bir örneğidir.
*   **Dinamik Metin Gösterimi**: UI, `event.UpdateEventSet` ve `event.RenderEventSet` fonksiyonlarını kendi güncelleme ve çizim döngülerinde çağırarak script metninin ekranda akıcı bir şekilde gösterilmesini sağlar.
*   **Kullanıcı Girdisi İşleme**: Oyuncu görev penceresindeki bir butona (İleri, Bitti, Cevap Seçeneği) tıkladığında, UI scripti bu tıklamayı yakalar ve `event.Skip(olay_seti_ID)` (sonraki adıma geçmek için) veya `event.SelectAnswer(olay_seti_ID, cevap_numarasi)` gibi fonksiyonları çağırır.
*   **Arayüz Elemanlarının Kontrolü**: Script içindeki `[IMAGE]`, `[LEFTIMAGE]` gibi komutlar `CPythonEventManager` tarafından işlendiğinde, C++ tarafı `event.SetEventHandler` ile kaydedilmiş Python olay işleyicisinin `OnImage`, `OnLeftImage` gibi fonksiyonlarını çağırarak UI'ın ilgili resimleri göstermesini tetikler.
*   **Genel Oyun Durumu Senkronizasyonu**: `event.SetInterfaceWindow` ile ana arayüz modülünün Python referansı C++ tarafına iletilir. Bu sayede C++'daki `CPythonEventManager`, script komutları aracılığıyla genel arayüz fonksiyonlarını (örn. zindan sonucu gösterme, görev listesini yenileme) çağırabilir.

Kısacası, `event` modülü, C++'ın performanslı script işleme motoru ile Python'un esnek UI programlama yetenekleri arasında hayati bir köprü görevi görür. Bu, Metin2 gibi oyunlarda karmaşık ve zengin içerikli görev sistemlerinin oluşturulmasını mümkün kılar.

--- 

### `PythonExceptionSender.h` (Python İstisna Gönderici Başlık Dosyası)

Bu başlık dosyası, `CPythonExceptionSender` sınıfının bildirimini içerir. Bu sınıf, `IPythonExceptionSender` arayüzünden miras alır ve istemci tarafında oluşan Python istisnalarını (hatalarını) bir sunucuya göndermek amacıyla tasarlanmıştır. Ancak, mevcut kaynak kodda `Send()` metodunun içeriğinin tamamen yorum satırı haline getirildiği görülmektedir.

#### Dosyanın Genel Amaçları

*   **`CPythonExceptionSender` Sınıf Bildirimi**: Python istisnalarını toplama ve gönderme işlevselliği için arayüzü tanımlar.
    *   Yapıcı (`CPythonExceptionSender()`) ve yıkıcı (`~CPythonExceptionSender()`) metotları bildirir.
    *   `Send()`: İstisna bilgisini bir sunucuya göndermesi beklenen ana metodu bildirir.
*   **Üye Değişkenler**:\
    *   `m_kSet_dwSendedExceptionCRC`: `std::set<DWORD>` türünde bir üye değişkendir. Amacı, daha önce gönderilmiş olan istisnaların CRC32 (Döngüsel Artıklık Denetimi) değerlerini saklayarak aynı hatanın tekrar tekrar gönderilmesini engellemektir.\

#### Arayüz ve Kullanım\
*   `IPythonExceptionSender`: `CPythonExceptionSender` sınıfının implemente ettiği bir arayüzdür. Bu arayüzün tanımı bu başlıkta bulunmamaktadır, ancak genellikle Python motoru veya genel hata işleme mekanizması tarafından çağrılacak bir `SendException(const char* szExceptionString)` gibi bir metod içermesi beklenir. `CPythonExceptionSender` muhtemelen bu tür bir metod aracılığıyla istisna bilgisini alır ve `m_strExceptionString` (bu başlıkta deklare edilmemiş, ancak .cpp dosyasında kullanılıyor) gibi bir üyede saklar.\
*   Temel fikir, istemcide bir Python hatası oluştuğunda, bu hata `CPythonExceptionSender`\'a iletilir. `Send()` metodu çağrıldığında, bu hata mesajı alınır, bir CRC32 karması hesaplanır. Eğer bu karma daha önce gönderilenler listesinde (`m_kSet_dwSendedExceptionCRC`) yoksa, hata mesajı ağ üzerinden belirli bir sunucuya (kod içinde \"211.105.222.20\" olarak sabit kodlanmış) ve porta (`LocaleService_GetPythonErrorReportPort()` ile alınan) gönderilir. Gönderim başarılı olursa, CRC32 değeri sete eklenir.\

#### Derleme Bağımlılıkları\
*   Yok (Doğrudan `std::set` dışında standart kütüphanede bağımlılığı görünmüyor).\

Bu başlık dosyası, istemcideki Python hatalarının merkezi bir sunucuya raporlanması için bir mekanizma tanımlar. Ancak, `.cpp` dosyasındaki `Send()` metodunun yorum satırı olması, bu özelliğin şu anda aktif olarak kullanılmadığını veya devre dışı bırakıldığını göstermektedir.

---

### `PythonExceptionSender.cpp` (Python İstisna Gönderici Implementasyonu)

Bu dosya, `CPythonExceptionSender` sınıfının metotlarının implementasyonlarını içerir. Sınıfın temel amacı, istemcide oluşan Python kaynaklı istisna (hata) mesajlarını belirli bir sunucuya göndermektir. Ancak, bu dosyadaki `Send()` metodunun içeriğinin **tamamen yorum satırı halinde olması** dikkat çekicidir. Bu durum, özelliğin ya geliştirme aşamasında yarım kaldığını, ya test amaçlı geçici olarak kapatıldığını gösterir.

#### Dosyanın Genel Amaçları

*   **İstisna Gönderme Mantığı (Yorum Satırında)**: `Send()` metodu, eğer aktif olsaydı, şu adımları izleyecekti:\
    1.  Saklanan istisna mesajının (`m_strExceptionString` - bu üyenin nasıl doldurulduğu bu dosyada açık değildir, muhtemelen `IPythonExceptionSender` arayüzünden gelen bir çağrı ile ayarlanır) CRC32 karmasını hesaplar.\
    2.  Bu CRC32 değeri daha önce gönderilenler listesinde (`m_kSet_dwSendedExceptionCRC`) varsa, aynı hatanın tekrar gönderilmesini engellemek için işlem yapmaz.\
    3.  Hata mesajını `TraceError` ile yerel loglara yazar.\
    4.  Bir TCP soketi oluşturur.\
    5.  Soketi non-blocking (engellemesiz) moda ayarlar.\
    6.  Hedef sunucu adresini (\"211.105.222.20\") ve portunu (`LocaleService_GetPythonErrorReportPort()` ile alınan) belirler.\
    7.  Sunucuya bağlanmayı dener.\
    8.  Sunucudan bir \"bilet numarası\" (number\_ticket) almayı bekler (bu adımın amacı net değildir, bir tür sıralama veya kimlik doğrulama olabilir).\
    9.  İstisna mesajını parçalar halinde sunucuya göndermeye çalışır (en fazla 100 deneme ile).\
    10. Soketi kapatır.\
    11. Başarıyla gönderildiyse, istisnanın CRC32\'sini `m_kSet_dwSendedExceptionCRC` setine ekler.\

*   **Yapıcı ve Yıkıcı Metotlar**:\
    *   `CPythonExceptionSender()`: Sınıfın yapıcı metodu, özel bir başlatma işlemi yapmaz.\
    *   `~CPythonExceptionSender()`: Sınıfın yıkıcı metodu, özel bir temizleme işlemi yapmaz.\

#### Önemli Noktalar ve Potansiyel Durum\
*   **Devre Dışı Özellik**: `Send()` metodunun tamamen yorum satırı olması, bu istisna gönderme özelliğinin şu anda çalışmadığı anlamına gelir. Eğer bu özellik yeniden aktif edilecekse, yorumların kaldırılması ve muhtemelen güncel ağ kütüphaneleri veya API\'leri ile uyumlu hale getirilmesi gerekebilir (örneğin, `ioctlsocket`, `inet_addr` gibi fonksiyonlar modern uygulamalarda farklı şekillerde ele alınabilir).\
*   **Bağımlılıklar (Eğer Aktif Olsaydı)**: Kodun yorumlanmamış hali `winsock2.h` (soket işlemleri için), `GetCaseCRC32` (CRC32 hesaplaması için, muhtemelen EterLib veya benzeri bir kütüphaneden) ve `LocaleService` (port numarasını almak için) gibi bağımlılıklara ihtiyaç duyar.\
*   **Hata Raporlama Sunucusu**: Sabit kodlanmış IP adresi (\"211.105.222.20\"), hata raporlarının toplandığı merkezi bir sunucuyu işaret eder. Bu sunucunun erişilebilir ve çalışır durumda olması gerekirdi.\
*   **Güvenlik ve Verimlilik**: Mevcut (yorum satırındaki) kodda, gönderilen verinin şifrelenmesi veya sıkıştırılması gibi mekanizmalar bulunmamaktadır. Ayrıca, her hata için yeni bir TCP bağlantısı kurulması, çok sayıda hata oluşması durumunda verimsiz olabilir.\

Bu dosya, bir zamanlar Python hatalarını uzaktaki bir sunucuya bildirerek geliştiricilerin istemci hatalarını takip etmesine yardımcı olması planlanmış bir sistemin kalıntılarını içermektedir. Mevcut haliyle, bu işlevsellik aktif değildir.

---

### `PythonExchange.h` (Ticaret Sistemi Başlık Dosyası)

Bu başlık dosyası, Metin2 istemcisindeki oyuncular arası ticaret (takas) sistemini yöneten `CPythonExchange` singleton sınıfının bildirimini içerir. Ticaret penceresindeki her iki tarafın (oyuncunun kendisi ve ticaret yapılan diğer oyuncu - kurban/hedef) koyduğu eşyaları, parayı (Yang/Elk), çeki (eğer `ENABLE_CHEQUE_SYSTEM` aktifse) ve onay durumlarını takip etmek için gerekli veri yapılarını ve fonksiyon prototiplerini tanımlar.

#### Dosyanın Genel Amaçları

*   **`CPythonExchange` Sınıf Bildirimi**: Singleton olarak tasarlanmış ana ticaret yöneticisi sınıfını deklare eder.
*   **`TExchangeData` Yapısı Tanımlama**: Bir ticaret oturumundaki tek bir tarafın (oyuncu veya hedef) verilerini tutan yapıyı tanımlar. Bu yapı şunları içerir:
    *   `name`: Karakter adı (`CHARACTER_NAME_MAX_LEN`).
    *   `item_vnum`: Ticarete konulan eşyaların VNUM'ları (`EXCHANGE_ITEM_MAX_NUM` adet).
    *   `item_count`: Ticarete konulan eşyaların miktarları.
    *   `item_metin`: Eşyalardaki taşların VNUM'ları (her eşya için `ITEM_SOCKET_SLOT_MAX_NUM` adet).
    *   `item_attr`: Eşyalardaki efsunların türü ve değeri (her eşya için `ITEM_ATTRIBUTE_SLOT_MAX_NUM` adet).
    *   `accept`: Oyuncunun ticareti onaylayıp onaylamadığını belirten bir `BYTE` (0 veya 1).
    *   `elk`: Ticarete konulan Yang (para) miktarı (`DWORD`).
    *   `level`: Karakterin seviyesi (`int`).
    *   **İsteğe Bağlı Sistemlere Göre Eklenen Üyeler** (derleme zamanı makrolarıyla kontrol edilir):
        *   `cheque` (`ENABLE_CHEQUE_SYSTEM`): Ticarete konulan Çek miktarı (`DWORD`).
        *   `dwTransmutationVnum` (`ENABLE_CHANGE_LOOK_SYSTEM`): Eşyaların yansıtma (görünüm değiştirme) VNUM'ları.
        *   `dwRefineElement` (`ENABLE_REFINE_ELEMENT`): Eşyaların element bilgisi.
        *   `set_value` (`ENABLE_SET_ITEM`): Eşyaların set değeri bilgisi (`uint8_t`).
        *   `item_apply_random` (`ENABLE_APPLY_RANDOM`): Eşyalardaki rastgele efsunlar.
        *   `item_min_max_value` (`ENABLE_ITEM_VALUE10`): Eşyaların min/max değerleri (örn. beceri hasarı aralığı için).
*   **Sabit Tanımlama**:
    *   `EXCHANGE_ITEM_MAX_NUM`: Ticaret penceresindeki maksimum eşya slot sayısı (12 olarak tanımlanmış).
*   **Fonksiyon Prototipleri**: Ticaret durumunu başlatma/bitirme, her iki taraf için isim, seviye, para, eşya ve onay durumlarını ayarlama ve sorgulama fonksiyonlarını bildirir.

#### C++ ve Python Arayüzü ile İlişkisi

*   Bu sınıfta tanımlanan fonksiyonlar (`SetSelfName`, `GetItemVnumFromTarget` vb.), genellikle Python tarafındaki `exchange` modülü (`PythonExchangeModule.cpp` içinde tanımlanır) aracılığıyla UI scriptlerine sunulur.
*   Sunucudan gelen ticaretle ilgili ağ paketleri (`CPythonNetworkStream` tarafından işlenir) `CPythonExchange` singleton'ının ilgili metotlarını çağırarak ticaret penceresinin durumunu günceller (örn. `SetItemToTarget`, `SetAcceptToSelf`).
*   Kullanıcı arayüzü (UI) scriptleri, oyuncu ticaret penceresine eşya koyduğunda, para eklediğinde veya onayla butonuna tıkladığında, `exchange` modülü üzerinden yine `CPythonExchange` fonksiyonlarını çağırarak bu eylemleri sunucuya iletir (bu kısım genellikle `CPythonNetworkStream` üzerinden paket gönderilerek yapılır, `CPythonExchange` ise sadece istemci tarafındaki durumu tutar ve UI'a yansıtır).
*   `m_self` ve `m_victim` adlı `TExchangeData` türündeki iki üye değişken, sırasıyla oyuncunun kendi ticaret bilgilerini ve ticaret yaptığı diğer oyuncunun bilgilerini saklar.
*   `m_isTrading`: Mevcut bir ticaret oturumunun aktif olup olmadığını belirten bir boolean bayraktır.
*   `m_elk_mode`: (Bu değişkenin tam amacı başlık dosyasından net değil, ancak genellikle Yang/para giriş modunu belirtmek için kullanılır.)

#### Derleme Bağımlılıkları

*   `Packet.h`: `TPlayerItemAttribute` gibi eşya özelliklerini ve `CHARACTER_NAME_MAX_LEN`, `ITEM_SOCKET_SLOT_MAX_NUM` gibi sabitleri içeren başlık dosyası.
*   Çeşitli `ENABLE_` makroları (`ENABLE_CHEQUE_SYSTEM`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_REFINE_ELEMENT`, `ENABLE_SET_ITEM`, `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10`): Bu makrolar aktif olduğunda `TExchangeData` yapısına ek üyeler ve `CPythonExchange` sınıfına bu üyelerle ilgili ek fonksiyonlar eklenir. Bu, oyunun farklı özellik setleriyle derlenebilmesini sağlar.

Bu başlık dosyası, oyuncular arası ticaretin istemci tarafındaki veri modelini ve temel yönetim arayüzünü tanımlar. Ticaret penceresinin doğru ve senkronize bir şekilde çalışması için kritik bir bileşendir.

---

### `PythonExchange.cpp` (Ticaret Sistemi Implementasyonu)

Bu dosya, `CPythonExchange` sınıfının metotlarının C++ implementasyonlarını içerir. `CPythonExchange.h` dosyasında tanımlanan arayüzü gerçekleştirerek, oyuncular arası ticaret sisteminin istemci tarafındaki veri yönetimi ve durum takibini sağlar.

#### Dosyanın Genel Amaçları

*   **Ticaret Verilerini Yönetme**: Hem oyuncunun kendi (`m_self`) hem de ticaret yaptığı diğer oyuncunun (`m_victim`) ticaret penceresine koyduğu eşyaların (VNUM, adet, taşlar, efsunlar, isteğe bağlı sistemlere göre yansıtma, element, set değeri, rastgele efsun, min/max değer), para (Yang/Elk), Çek (eğer `ENABLE_CHEQUE_SYSTEM` aktifse), seviye ve isim bilgilerini ayarlar (`Set*` fonksiyonları) ve sorgular (`Get*` fonksiyonları).
*   **Ticaret Durumunu İzleme**:
    *   `Start()`: Bir ticaret oturumunun başladığını işaretlemek için `m_isTrading` bayrağını `true` yapar.
    *   `End()`: Ticaret oturumunun bittiğini işaretlemek için `m_isTrading` bayrağını `false` yapar.
    *   `isTrading()`: Mevcut bir ticaretin aktif olup olmadığını döndürür.
*   **Onay Durumlarını Yönetme**: Her iki tarafın da ticareti onaylayıp onaylamadığını (`m_self.accept`, `m_victim.accept`) ayarlar (`SetAcceptToSelf`, `SetAcceptToTarget`) ve sorgular (`GetAcceptFromSelf`, `GetAcceptFromTarget`).
*   **Veri Temizleme (`Clear`)**: Bir ticaret oturumu başladığında veya bittiğinde, `m_self` ve `m_victim` yapılarındaki tüm verileri sıfırlar (`memset`).
*   **Para Giriş Modu**: `SetElkMode` ve `GetElkMode` ile (muhtemelen) Yang/para girişinin aktif olup olmadığını yönetir.
*   **İsteğe Bağlı Sistem Desteği**: `ENABLE_CHEQUE_SYSTEM`, `ENABLE_SET_ITEM`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_REFINE_ELEMENT`, `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10` gibi derleme zamanı makrolarına bağlı olarak, bu sistemlerle ilgili ek eşya bilgilerini (çek, set değeri, yansıtma VNUM'u vb.) ayarlayan ve sorgulayan fonksiyonları içerir.

#### C++ Implementasyon Detayları

*   **Veri Saklama**: Tüm ticaret verileri, `m_self` ve `m_victim` adlı iki `TExchangeData` yapısında saklanır. Bu yapılar `CPythonExchange.h` içinde tanımlanmıştır.
*   **Sınır Kontrolleri**: Eşya ekleme/sorgulama fonksiyonlarında (`SetItemToTarget`, `GetItemVnumFromSelf` vb.), verilen pozisyon indeksinin (`pos`) geçerli bir aralıkta (`EXCHANGE_ITEM_MAX_NUM`) olup olmadığını kontrol eder. Geçersizse işlem yapmaz veya 0/varsayılan değer döndürür.
*   **String İşlemleri**: Karakter isimlerini ayarlarken `strncpy` kullanılır (`SetSelfName`, `SetTargetName`).
*   **Başlatma (`CPythonExchange()` Yapıcısı)**: Sınıf oluşturulduğunda `Clear()` fonksiyonunu çağırarak tüm ticaret verilerini sıfırlar ve `m_isTrading` ile `m_elk_mode` bayraklarını `false` olarak ayarlar.
*   **Fonksiyonların Yapısı**: Çoğu fonksiyon, ilgili `TExchangeData` yapısının (`m_self` veya `m_victim`) uygun üyesine doğrudan erişerek değer atar veya okur.
    *   Örnek: `SetItemToTarget(DWORD pos, DWORD vnum, WORD count)` fonksiyonu `m_victim.item_vnum[pos] = vnum;` ve `m_victim.item_count[pos] = count;` atamalarını yapar.
    *   Örnek: `GetItemVnumFromSelf(BYTE pos)` fonksiyonu `return m_self.item_vnum[pos];` değerini döndürür.
*   Onay durumu (`accept`) `BYTE` olarak alınır ancak `bool` olarak saklanır ve döndürülür.

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonExchange` sınıfı, istemcinin ticaret arayüzünün (UI) ihtiyaç duyduğu tüm verileri sağlar ve bu verilerin sunucudan gelen bilgilerle güncel tutulmasına yardımcı olur.

*   **Ticaret Başlatma**: Oyuncu başka bir oyuncuyla ticarete başladığında, sunucudan gelen bilgilerle `Start()`, `SetSelfName()`, `SetTargetName()`, `SetLevelToTarget()` gibi fonksiyonlar çağrılarak ticaret oturumu başlatılır ve temel bilgiler ayarlanır.
*   **Eşya/Para Ekleme/Çıkarma**: Bir oyuncu ticaret penceresine eşya koyduğunda veya para (Yang/Çek) eklediğinde, UI bu eylemi sunucuya bildirir. Sunucu bu değişikliği onayladıktan sonra, diğer oyuncunun istemcisine ve oyuncunun kendi istemcisine güncel bilgileri gönderir. Bu bilgiler `CPythonNetworkStream` tarafından alınır ve `CPythonExchange` sınıfının ilgili `SetItemToSelf/Target`, `SetElkToSelf/Target` vb. fonksiyonları çağrılarak yerel veri modeli güncellenir. UI, bu güncellenmiş verileri `Get*` fonksiyonları ile çekerek ticaret penceresini yeniler.
*   **Onaylama**: Bir oyuncu "Onayla" kutusunu işaretlediğinde, UI bunu sunucuya bildirir. Sunucu da bu durumu diğer oyuncuya iletir. `SetAcceptToSelf/Target` fonksiyonları ile onay durumları güncellenir. UI, `GetAcceptFromSelf/Target` ile bu durumları periyodik olarak kontrol ederek her iki tarafın da onay verip vermediğini gösterir.
*   **Ticaretin Tamamlanması/İptali**: Ticaret başarıyla tamamlandığında veya iptal edildiğinde, `End()` fonksiyonu çağrılır ve `Clear()` ile veriler sıfırlanır. UI da ticaret penceresini kapatır.

Bu sınıf, doğrudan Python'a `exchange` adlı bir modül aracılığıyla sunularak (bkz. `PythonExchangeModule.cpp`), Python tabanlı UI scriptlerinin ticaret verilerine erişmesini ve ticaret mantığını istemci tarafında yönetmesini mümkün kılar. Ancak, asıl ticaret mantığı ve güvenliği sunucu tarafında işlenir; `CPythonExchange` daha çok sunucudan gelen verilerin bir yansımasını tutar ve UI için bir arayüz sağlar.

---

### `PythonExchangeModule.cpp` (Python `exchange` Modülü)

Bu dosya, `CPythonExchange` sınıfının C++ tarafındaki ticaret (takas) sistemi işlevlerini Python betiklerinin kullanabilmesi için `exchange` adında bir Python modülü oluşturur. Bu modül, `CPythonExchange` sınıfındaki neredeyse tüm `Get*` ve bazı `Set*` fonksiyonlarını Python'a açarak, kullanıcı arayüzü (UI) scriptlerinin ticaret penceresindeki verileri okumasını ve bazı temel ayarları yapmasını sağlar.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`exchange`)**: `CPythonExchange` sınıfının ticaret bilgilerine erişim sağlayan fonksiyonlarını Python'a `exchange` modülü olarak sunar.
*   **Ticaret Verilerine Erişim**:
    *   Ticaretin aktif olup olmadığını kontrol etme (`isTrading`).
    *   Ticaret oturumunu sıfırlama/yeniden başlatma (`InitTrading`) - Bu fonksiyon `CPythonExchange::Instance().End()` çağırır, bu da mevcut ticareti sonlandırır ve verileri temizler.
    *   Her iki tarafın Yang (Elk) miktarını alma (`GetElkFromSelf`, `GetElkFromTarget`).
    *   Her iki tarafın onay durumunu alma (`GetAcceptFromSelf`, `GetAcceptFromTarget`).
    *   Her iki taraftaki belirli bir slottaki eşyanın VNUM'unu ve sayısını alma (`GetItemVnumFromSelf`, `GetItemCountFromSelf`, `GetItemVnumFromTarget`, `GetItemCountFromTarget`).
    *   Her iki tarafın adını alma (`GetNameFromSelf`, `GetNameFromTarget`).
    *   Hedef oyuncunun seviyesini alma (`GetLevelFromTarget`).
    *   Her iki taraftaki eşyaların taş (metin) soket bilgilerini alma (`GetItemMetinSocketFromSelf`, `GetItemMetinSocketFromTarget`).
    *   Her iki taraftaki eşyaların efsun (attribute) bilgilerini alma (`GetItemAttributeFromSelf`, `GetItemAttributeFromTarget`).
*   **İsteğe Bağlı Sistem Fonksiyonları**:
    *   `ENABLE_CHEQUE_SYSTEM`: Çek miktarını alma (`GetChequeFromSelf`, `GetChequeFromTarget`).
    *   `ENABLE_APPLY_RANDOM`: Rastgele efsun bilgilerini alma (`GetItemApplyRandomFromSelf`, `GetItemApplyRandomFromTarget`).
    *   `ENABLE_ITEM_VALUE10`: Eşyaların min/max değerlerini alma (`GetItemMinMaxValueFromSelf`, `GetItemMinMaxValueFromTarget`).
    *   `ENABLE_CHANGE_LOOK_SYSTEM`: Yansıtılmış eşya VNUM'larını alma (`GetChangeLookVnumFromSelf`, `GetChangeLookVnumFromTarget`).
    *   `ENABLE_REFINE_ELEMENT`: Eşyaların element bilgilerini alma (`GetItemRefineElementFromSelf`, `GetItemRefineElementFromTarget`).
    *   `ENABLE_SET_ITEM`: Eşyaların set değerlerini alma (`GetItemSetValueFromSelf`, `GetItemSetValueFromTarget`).
*   **Yang (Elk) Giriş Modu**: Yang giriş modunu sorgulama ve ayarlama (`GetElkMode`, `SetElkMode`).
*   **Sabitlerin Aktarılması**: `CPythonExchange::EXCHANGE_ITEM_MAX_NUM` sabiti, Python modülüne `exchange.EXCHANGE_ITEM_MAX_NUM` olarak eklenir.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initTrade()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `exchange` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyon adlarını (örn. "isTrading", "GetElkFromSelf") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `exchangeisTrading`, `exchangeGetElkFromSelf`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("exchange", s_methods);` ile `exchange` modülü oluşturulur.
    *   `PyModule_AddIntConstant(poModule, "EXCHANGE_ITEM_MAX_NUM", CPythonExchange::EXCHANGE_ITEM_MAX_NUM);` ile sabit değer modüle eklenir.

*   **Fonksiyon Sarmalayıcıları (`exchange*` önekli fonksiyonlar)**:
    *   Bu C fonksiyonları, Python'dan gelen argümanları (`PyObject* poArgs`) alır (genellikle `PyTuple_GetInteger` veya `PyTuple_GetByte` ile pozisyon gibi parametreler alınır).
    *   `CPythonExchange::Instance()` üzerinden ilgili C++ metodunu çağırırlar.
    *   Dönüş değerlerini Python nesnelerine (`Py_BuildValue` ile integer 'i', string 's', boolean 'b' vb.) dönüştürerek geri gönderirler.
    *   Örneğin, `exchangeGetItemVnumFromSelf(PyObject* poSelf, PyObject* poArgs)` fonksiyonu Python'dan bir pozisyon (integer) alır, `CPythonExchange::Instance().GetItemVnumFromSelf((char)pos)` çağırır ve sonucu Python integer olarak döndürür.
    *   `exchangeSetElkMode` fonksiyonu, Python'dan bir integer alır ve bunu boolean'a çevirerek `CPythonExchange::Instance().SetElkMode()` fonksiyonunu çağırır.

#### Oyunda Kullanım Amacı ve Senaryoları

`exchange` modülü, Python ile yazılmış ticaret arayüzü (UI) scriptlerinin (`trade.py` veya benzeri), oyuncular arası ticaret penceresindeki tüm verileri okumasını ve bazı temel ayarları (Yang giriş modu gibi) yapmasını sağlar.

*   **Ticaret Arayüzünün Güncellenmesi**: UI scriptleri, ticaret penceresini çizerken veya güncellerken bu modüldeki `Get*` fonksiyonlarını (örn. `exchange.GetItemVnumFromTarget(slotIndex)`, `exchange.GetElkFromSelf()`) çağırarak `CPythonExchange`'den en güncel verileri alır ve oyuncuya gösterir.
*   **Durum Kontrolü**: UI, `exchange.isTrading()` ile bir ticaretin aktif olup olmadığını kontrol edebilir.
*   **Ticaret Başlatma/Sıfırlama**: Yeni bir ticaret başladığında veya mevcut bir ticaret iptal edildiğinde, UI `exchange.InitTrading()` çağırarak `CPythonExchange` içindeki verilerin temizlenmesini ve ticaret durumunun sıfırlanmasını tetikleyebilir. (Bu fonksiyon `CPythonExchange::End()` çağırdığı için aslında sonlandırma ve temizleme yapar.)
*   **Kullanıcı Etkileşimleri**: Oyuncu ticaret penceresine eşya eklediğinde, para koyduğunda veya "Onayla" butonuna tıkladığında, UI scriptleri bu eylemleri yakalar. Bu eylemlerin sunucuya bildirilmesi genellikle `CPythonNetworkStream` üzerinden doğrudan paket gönderilerek yapılır. `exchange` modülü daha çok sunucudan gelen yanıtlar sonucu güncellenen `CPythonExchange` verilerini okumak için kullanılır.
*   **İsteğe Bağlı Özelliklerin Gösterimi**: Eğer `ENABLE_CHEQUE_SYSTEM` gibi makrolar aktifse, UI scriptleri `exchange.GetChequeFromTarget()` gibi ek fonksiyonları çağırarak bu özelliklerle ilgili bilgileri de ticaret penceresinde gösterebilir.

Bu modül, C++ tarafındaki ticaret veri yönetimini Python UI scriptleri için erişilebilir kılarak, esnek ve veri odaklı bir ticaret arayüzü geliştirilmesine olanak tanır.

---

### `PythonFlyModule.cpp` (Python `fly` Modülü)

Bu dosya, `CFlyingManager` sınıfının (Uçan Nesne Yöneticisi) temel genel güncelleme ve çizim işlevlerini Python betiklerinin kullanabilmesi için `fly` adında basit bir Python modülü oluşturur. `CFlyingManager`, oyun içinde havada süzülen veya bir hedefe doğru hareket eden görsel öğeleri (örneğin oklar, büyü mermileri, deneyim/can küreleri) yönetir.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`fly`)**: `CFlyingManager`'ın `Update` ve `Render` fonksiyonlarını Python'a açar.
*   **Uçan Nesnelerin Güncellenmesi**: Python'dan `fly.Update()` çağrıldığında, `CFlyingManager::Instance().Update()` fonksiyonunu tetikler. Bu, tüm aktif uçan nesnelerin bir sonraki durumlarını (pozisyon, ömür vb.) hesaplar.
*   **Uçan Nesnelerin Çizdirilmesi**: Python'dan `fly.Render()` çağrıldığında, `CFlyingManager::Instance().Render()` fonksiyonunu tetikler. Bu, tüm görünür uçan nesneleri ekrana çizer.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initfly()` Fonksiyonu**:
    *   Python yorumlayıcısı `fly` modülünü ilk kez yüklediğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak fonksiyon adlarını ("Update", "Render") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (`flyUpdate`, `flyRender`) tanımlar.
    *   `Py_InitModule("fly", s_methods);` ile `fly` modülü oluşturulur.

*   **Fonksiyon Sarmalayıcıları**:
    *   `flyUpdate(PyObject* poSelf, PyObject* poArgs)`: Argüman almaz ve `CFlyingManager::Instance().Update()` çağırır. `Py_BuildNone()` döndürür.
    *   `flyRender(PyObject* poSelf, PyObject* poArgs)`: Argüman almaz ve `CFlyingManager::Instance().Render()` çağırır. `Py_BuildNone()` döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

Normal şartlarda, `CFlyingManager::Update()` ve `CFlyingManager::Render()` fonksiyonları, ana oyun döngüsü içinde (genellikle `CPythonApplication` sınıfı tarafından) C++ seviyesinde zaten çağrılır. Bu nedenle, `fly` Python modülünün bu fonksiyonları ayrıca sunması, genellikle aşağıdaki gibi özel durumlar için kullanılır:

*   **Test ve Hata Ayıklama**: Geliştiriciler, belirli uçan nesne davranışlarını test ederken veya hataları ayıklarken, oyunun normal akışını değiştirmeden Python konsolu veya scriptler aracılığıyla `fly.Update()` veya `fly.Render()` çağrılarını manuel olarak tetikleyebilirler.
*   **Özel Kontrol İhtiyaçları**: Çok nadir durumlarda, belirli bir oyun içi olay veya UI etkileşimi sonucunda uçan nesnelerin normal döngü dışında güncellenmesi veya çizdirilmesi gerekebilir. Ancak bu genellikle kaçınılan bir yaklaşımdır.
*   **Script Tabanlı Efekt Sistemleri**: Eğer oyun, bazı efektlerini veya basit uçan nesnelerini tamamen Python üzerinden yönetiyorsa, bu modül bu scriptlerin `CFlyingManager` ile temel düzeyde etkileşim kurmasını sağlar.

`CFlyingManager`'ın asıl nesne oluşturma (örneğin, bir ok fırlatma, bir büyü gönderme), parametrelerini ayarlama ve silme işlemleri genellikle doğrudan C++ kodu içinden veya `CEffectManager` ile entegre bir şekilde gerçekleştirilir. `fly` modülü, bu yöneticinin genel "saat vuruşu" (tick) ve çizim çağrılarına Python'dan erişim kapısı sunar.

--- 

### `PythonGameEventManager.h` (Oyun İçi Etkinlik Yöneticisi Başlık Dosyası)

Bu başlık dosyası, `InGameEventManager` adlı bir singleton C++ sınıfını tanımlar. Bu sınıf, oyun içindeki zamanlanmış etkinlikleri (örneğin, EXP bonusu, eşya düşürme oranı artışı, özel boss etkinlikleri, balıkçılık turnuvaları vb.) ve bu etkinliklerle ilişkili olası ödülleri yönetmek için tasarlanmıştır. Doğrudan bir Python modülü oluşturmasa da, işlevleri muhtemelen başka bir Python sarmalayıcı modül aracılığıyla kullanıcı arayüzü (UI) scriptlerine sunulur.

#### Dosyanın Genel Amaçları

*   **`InGameEventManager` Sınıf Bildirimi**: Oyun içi etkinliklerin takvimini, türlerini ve ödüllerini yöneten merkezi singleton sınıfını deklare eder.
*   **Veri Yapıları Tanımlama**:
    *   **`TEventTable`**: Tek bir oyun içi etkinliğin tüm detaylarını (ID, tür, başlangıç/bitiş zamanları, özel değerler, tamamlanma durumu) içeren yapıyı tanımlar.
    *   **`SEventReward` (typedef `TEventReward`)**: Bir etkinlikle ilişkili tek bir ödül eşyasının VNUM'unu ve sayısını tutan yapıyı tanımlar.
*   **Enum Tanımları**:
    *   **`EEventTypes`**: Çeşitli oyun içi etkinlik türlerini (`EVENT_TYPE_EXPERIENCE`, `EVENT_TYPE_ITEM_DROP`, `EVENT_TYPE_BOSS`, `EVENT_TYPE_TANAKA` vb.) listeler.
    *   İsimsiz enum: `MONTH_MAX_NUM`, `DAY_MAX_NUM`, `REWARD_MAX_NUM` gibi sistem içi sabitleri tanımlar.
*   **Temel Fonksiyon Prototipleri**:
    *   Etkinlik ödül listesini bir dosyadan yükleme (`LoadEventRewardList`).
    *   Etkinlik takvimini ve isim-ID eşlemlerini oluşturma/sıfırlama (`BuildEventMap`, `BuildEventNameMap`).
    *   Dışarıdan (genellikle sunucudan) gelen etkinlik verilerini sisteme ekleme (`AddEventData`).
    *   Etkinlik adı (string) ile etkinlik türü ID'sini (enum) getirme (`GetEvent`).
    *   Etkinlik verilerinin sunucudan talep edilip edilmediğini yönetme (`SetRequestEventData`, `GetRequestEventData`).
    *   Belirli bir gün için kayıtlı etkinlik sayısını sorgulama (`GetDayEventCount`).
    *   Belirli bir gündeki belirli bir etkinliğin detaylarını (`TEventTable`) alma (`GetEventData`).
    *   Belirli bir etkinlik türü için tanımlanmış ödül bilgilerini (`TEventReward`) alma (`GetEventRewardData`).
*   **Dahili Veri Saklama Yapıları (typedefs ve özel üyeler)**:
    *   `EventVector`: `std::vector<TEventTable>` için bir takma ad.
    *   `EventMap`: `std::map<int, EventVector>` için bir takma ad; gün ID'sini o güne ait etkinlik vektörüne eşler.
    *   `m_map_Event`: `EventMap` türünde, etkinlik takvimini tutan ana veri yapısı.
    *   `m_map_EventReward`: `std::unordered_map<int, std::vector<TEventReward>>` türünde, etkinlik türü ID'sini o etkinliğe ait ödül eşyası vektörüne eşler.
    *   `m_mapEventName`: `std::unordered_map<std::string, int>` türünde, etkinlik türü isimlerini (string) etkinlik türü ID'lerine (enum) eşler.
    *   `m_bIsRequestEventDataed`: Etkinlik verilerinin sunucudan talep edilip edilmediğini belirten boolean bayrak.

#### C++ ve Python Arayüzü ile İlişkisi (Dolaylı)

Bu sınıf doğrudan Python'a sunulmaz. Ancak, oyun içi etkinlik takvimi gibi bir kullanıcı arayüzü özelliği için kritik verileri ve mantığı yönetir. UI scriptleri (Python ile yazılmış), bu C++ sınıfının işlevlerine erişmek için büyük olasılıkla başka bir Python modülünü (örneğin, genel bir `game` veya özel bir `event_calendar` modülü) kullanır. Bu sarmalayıcı modül, `InGameEventManager::Instance()` üzerinden bu sınıfa erişir ve fonksiyonlarını Python'a uygun şekilde açar.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Etkinlik Takvimi Arayüzü**: UI, bu yöneticiden aldığı verilerle oyunculara yaklaşan veya devam eden etkinlikleri (EXP bonusu, Metin Yağmuru, Boss avı vb.) bir takvim üzerinde gösterebilir.
*   **Etkinlik Detayları**: Oyuncu bir etkinliğe tıkladığında, UI `GetEventData` ile etkinliğin başlangıç/bitiş saatlerini, türünü ve `GetEventRewardData` ile olası ödüllerini görüntüleyebilir.
*   **Veri Yükleme ve Güncelleme**: İstemci başladığında veya sunucudan yeni etkinlik bilgisi geldiğinde (`AddEventData` aracılığıyla), bu yönetici verileri işler ve saklar. `LoadEventRewardList` ile de istemci tarafında tanımlı ödül listeleri yüklenebilir.

Bu başlık dosyası, oyun içi etkinliklerin istemci tarafında organize edilmesi, saklanması ve sorgulanması için merkezi bir C++ altyapısı sunar.

--- 

### `PythonGameEventManager.cpp` (Oyun İçi Etkinlik Yöneticisi Implementasyonu)

Bu dosya, `PythonGameEventManager.h` dosyasında bildirilen `InGameEventManager` sınıfının metotlarının C++ implementasyonlarını içerir. Oyun içi etkinliklerin takvimini, türlerini, ödüllerini yükleme, saklama ve sorgulama işlevlerini gerçekleştirir.

#### Dosyanın Genel Amaçları

*   **Etkinlik ve Ödül Verilerinin Yüklenmesi ve Yönetimi**: Etkinlik takvimini sunucudan gelen verilere göre oluşturur ve istemci tarafında tanımlı etkinlik ödül listelerini bir dosyadan yükler.
*   **Veri Yapılarının Başlatılması ve Bakımı**: Etkinlikleri günlere göre (`m_map_Event`), ödülleri etkinlik türüne göre (`m_map_EventReward`) ve etkinlik isimlerini ID'lere (`m_mapEventName`) eşleyen haritaları (map) yönetir.
*   **Sorgu Arayüzü Sağlama**: Belirli bir gündeki etkinlikleri, bir etkinliğin detaylarını veya bir etkinliğin ödüllerini sorgulamak için fonksiyonlar sunar.

#### C++ Implementasyon Detayları

*   **Yapıcı (`InGameEventManager::InGameEventManager()`)**:
    *   `m_bIsRequestEventDataed` bayrağını `false` olarak ayarlar.
    *   `m_map_EventReward` ve `m_mapEventName` haritalarını temizler.
    *   `BuildEventMap()` ve `BuildEventNameMap()` çağrılarak temel veri yapıları hazırlanır.

*   **`LoadEventRewardList(const char* c_szFileName)`**:
    *   Eğer ödüller daha önce yüklenmişse (`!m_map_EventReward.empty()`) tekrar yükleme yapmaz.
    *   Belirtilen yapılandırma dosyasını (`.txt` formatında beklenir) `CTextFileLoader` ile yükler.
    *   Dosyadaki her bir anahtar (etkinlik adı, örn. "EVENT_TYPE_EXPERIENCE") için, "1" den `REWARD_MAX_NUM`'a kadar olan alt anahtarları arar.
    *   Her alt anahtar için iki değer (eşya VNUM'u ve miktarı) bekler.
    *   Okunan etkinlik adını büyük harfe çevirir (`toupper`).
    *   `GetEvent()` ile string etkinlik adını sayısal ID'ye dönüştürür.
    *   Geçerli bir etkinlik ID'si ve geçerli VNUM/miktar bilgisi varsa, ödülü `m_map_EventReward` haritasına (etkinlik ID'si -> `std::vector<TEventReward>`) ekler.

*   **`BuildEventMap()`**:
    *   `m_map_Event` haritasını (gün ID'si -> `std::vector<TEventTable>`) temizler.
    *   `DAY_MAX_NUM` (31) boyunca her gün için boş bir etkinlik vektörü oluşturarak haritayı başlatır.

*   **`BuildEventNameMap()`**:
    *   `m_mapEventName` haritasını (string etkinlik adı -> etkinlik ID'si) `EEventTypes` enumundaki tüm etkinlik türleri için ilgili string ve enum değerleriyle doldurur.

*   **`AddEventData(std::vector<TEventTable>& eventVec)`**:
    *   `BuildEventMap()` çağırarak mevcut etkinlik takvimini sıfırlar.
    *   Parametre olarak gelen `eventVec` (genellikle sunucudan alınan aktif etkinliklerin listesi) içindeki her `TEventTable` öğesini işler.
    *   Her etkinliğin `startTime` (Unix zaman damgası) değerini `localtime()` ile yerel takvim gününe (`tm_mday`) çevirir.
    *   Etkinliği, hesaplanan bu gün ID'sine göre `m_map_Event` haritasındaki ilgili günün vektörüne ekler. Böylece etkinlikler günlere göre gruplanır.

*   **`GetEvent(const std::string strEventName)`**:
    *   Verilen string etkinlik adının `m_mapEventName` içinde kayıtlı olup olmadığını kontrol eder.
    *   Kayıtlıysa karşılık gelen sayısal etkinlik ID'sini, değilse `EVENT_TYPE_NONE` döndürür.

*   **`GetDayEventCount(int iDay)`**:
    *   Verilen gün (`iDay`) için `m_map_Event` haritasından ilgili günün etkinlik vektörünü alır.
    *   Bu vektörün boyutunu (o günkü etkinlik sayısını) döndürür. Geçersiz gün indeksi için 0 döndürür.

*   **`GetEventData(int eventIdx, int iDay, TEventTable& table)`**:
    *   Verilen gün (`iDay`) için `m_map_Event` haritasından ilgili günün etkinlik vektörünü alır.
    *   Eğer `eventIdx` (o gün içindeki etkinlik sırası, 0 tabanlı) bu vektörün sınırları içindeyse, ilgili `TEventTable` verisini `memcpy` kullanarak çıktı parametresi olan `table`'a kopyalar.

*   **`GetEventRewardData(int eventType, int pos, TEventReward& reward)`**: (Parametre adı `eventIdx` değil, `eventType` olmalıydı, çünkü `m_map_EventReward` haritası etkinlik türünü anahtar olarak kullanır.)
    *   Verilen etkinlik türü ID'si (`eventType`) için `m_map_EventReward` haritasından ödül vektörünü arar.
    *   Eğer bu etkinlik türü için ödüller tanımlanmışsa ve `pos` (ödül sırası, 0 tabanlı) bu vektörün sınırları içindeyse, ilgili `TEventReward` (VNUM ve miktar) bilgisini çıktı parametresi olan `reward`'a atar.

*   **Diğer Metotlar**:
    *   `SetRequestEventData(bool isRequested)` ve `GetRequestEventData()`: `m_bIsRequestEventDataed` bayrağını ayarlamak ve okumak için basit ayarlayıcı/alıcı (setter/getter) metotlarıdır.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu sınıfın implementasyonu, oyun içi etkinliklerin istemci tarafında etkili bir şekilde yönetilmesini sağlar:

*   **Etkinlik Takvimi Veri Kaynağı**: UI scriptleri, bir etkinlik takvimi arayüzü oluşturmak için bu sınıftan günlere göre gruplanmış etkinlik bilgilerini (`GetDayEventCount`, `GetEventData`) ve her bir etkinliğin detaylarını alabilir.
*   **Ödül Bilgilerinin Gösterimi**: Kullanıcılar bir etkinliğin detaylarını incelediğinde, `GetEventRewardData` ile o etkinliğe özel ödüller (eğer varsa) UI'da listelenebilir.
*   **Dinamik Etkinlik Güncellemeleri**: Sunucudan yeni etkinlik verileri geldiğinde (`AddEventData` aracılığıyla), istemcinin etkinlik takvimi güncellenir ve bu değişiklikler UI'a yansıtılabilir.
*   **Yerel Ödül Tanımları**: `LoadEventRewardList` sayesinde, bazı etkinliklerin ödülleri istemci tarafındaki bir yapılandırma dosyasından yönetilebilir, bu da bazı güncellemeler için sunucuya bağımlılığı azaltabilir veya esneklik sağlayabilir.

Bu C++ sınıfı, istemcinin oyun içi etkinlikleri anlamasını ve kullanıcıya sunmasını sağlayan temel mantığı ve veri yönetimini oluşturur. UI scriptleri, bu altyapıyı kullanarak zengin ve bilgilendirici bir etkinlik arayüzü oluşturabilir.

--- 

### `PythonGameEventManagerModule.cpp` (Python `eventMgr` Modülü)

Bu dosya, `GameLib` kütüphanesi içinde yer alan `CGameEventManager` sınıfının temel işlevlerini Python betiklerinin kullanabilmesi için `eventMgr` adında bir Python modülü oluşturur. Bu modül, oyun dünyasındaki konuma bağlı olayların veya tetikleyicilerin güncellenmesiyle ilgili gibi görünmektedir ve daha önce incelenen takvim tabanlı `InGameEventManager`'dan farklı bir amaca hizmet eder.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma (`eventMgr`)**: `CGameEventManager` sınıfının `Update` ve ilişkili `SetCenterPosition` işlevlerini Python'a açar.
*   **Konum Tabanlı Olayların Güncellenmesi**:
    *   `eventMgr.Update(float x, float y, float z)`: Python'dan bir merkez pozisyonu (x, y, z koordinatları) alır.
        *   Önce `CGameEventManager::Instance().SetCenterPosition(fx, fy, fz)` çağrılarak olay yöneticisinin odak noktası bu koordinatlara ayarlanır.
        *   Ardından `CGameEventManager::Instance().Update()` çağrılarak, bu merkez pozisyona ve diğer oyun koşullarına göre olaylar kontrol edilir ve güncellenir.

#### C++ ve Python Arayüzü Implementasyon Detayları

*   **`initeventmgr()` Fonksiyonu**:
    *   Python yorumlayıcısı `eventMgr` modülünü ilk kez yüklediğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak "Update" fonksiyon adını ve buna karşılık gelen C fonksiyon sarmalayıcısını (`eventMgrUpdate`) tanımlar.
    *   `PyObject* poModule = Py_InitModule("eventMgr", s_methods);` ile `eventMgr` modülü oluşturulur.

*   **Fonksiyon Sarmalayıcısı (`eventMgrUpdate`)**:
    *   Python'dan üç adet float argüman (fx, fy, fz) alır.
    *   Bu argümanları `CGameEventManager::Instance().SetCenterPosition()` fonksiyonuna geçirir.
    *   Sonrasında `CGameEventManager::Instance().Update()` fonksiyonunu çağırır.
    *   İşlem tamamlandığında `Py_BuildNone()` döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

`eventMgr` modülünün ve `CGameEventManager`'ın kullanım senaryoları şunları içerebilir:

*   **Oyuncu Odaklı Olaylar**: `eventMgr.Update(playerX, playerY, playerZ)` çağrısı, oyuncunun mevcut pozisyonunu merkez alarak çevresindeki olayları (örneğin, belirli bir bölgeye girince aktif olan görevler, gizli tetikleyiciler, çevresel ses efektleri) kontrol etmek ve güncellemek için kullanılabilir. Bu, genellikle her oyun döngüsünde veya oyuncu pozisyonu önemli ölçüde değiştiğinde çağrılır.
*   **Dinamik Ortam Etkileşimleri**: Oyun dünyasında belirli alanlara girildiğinde veya çıkıldığında scriptlerin tetiklenmesi, özel efektlerin oynatılması veya NPC davranışlarının değişmesi gibi dinamik etkileşimler bu sistemle yönetilebilir.
*   **Görev Tetikleyicileri**: Bazı görevler, oyuncunun belirli bir coğrafi konuma ulaşmasıyla başlayabilir veya ilerleyebilir. `CGameEventManager`, bu tür konum tabanlı görev adımlarını izleyebilir.

Bu modül, `CGameEventManager` aracılığıyla oyun dünyasının daha dinamik ve oyuncunun konumuyla etkileşimli hale gelmesine olanak tanır. Bu, genellikle `Update` fonksiyonunun oyunun ana döngüsünde (`CPythonApplication` veya benzeri bir yerden) oyuncunun güncel koordinatlarıyla çağrılmasıyla gerçekleştirilir.

--- 