# Metin2 Oyun Sunucusu - Çekirdek Mekanikleri Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) çekirdek mekanikleriyle ilgili dosyalarını (karakter yönetimi, eşya yönetimi, hareket/sektör yönetimi, temel döngü, girdi işleme, savaş, beceriler, etkiler vb.) belgeler.

## İçindekiler

*   [Header Dosyaları (.h)](#header-dosyaları-h)
    *   [`affect_flag.h`](#affect_flagh)
    *   [`affect.h`](#affecth)
    *   [`ani.h`](#anih)
    *   [`any_function.h`](#any_functionh)
    *   [`constants.h`](#constantsh)
*   [Kaynak Kod Dosyaları (.cpp)

---

## Header Dosyaları (.h)

### `affect_flag.h`

*   **Amaç:** Karakterler veya diğer oyun nesneleri üzerindeki aktif etkileri (buff/debuff) temsil etmek için kullanılan 64 bitlik bir bayrak kümesini (bitmask) yöneten `TAffectFlag` yapısını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   Standart bit manipülasyon makroları (`IS_SET`, `SET_BIT`, `REMOVE_BIT`, `TOGGLE_BIT`) içerir (varsa tanımlamaz).
    *   `TAffectFlag` yapısı:
        *   `DWORD bits[2]`: 64 bitlik bayrak alanını tutar.
        *   Yapıcı Metotlar: Yapıyı başlatır.
        *   `IsSet(int flag)`: Belirli bir bayrağın (1'den `AFF_BITS_MAX`-1'e, muhtemelen 64'e kadar) kurulu olup olmadığını kontrol eder. Bayrağın hangi `DWORD`'de ve hangi bitte olduğunu hesaplar.
        *   `Set(int flag)`: Belirli bir bayrağı kurar.
        *   `Reset(int flag)`: Belirli bir bayrağı kaldırır.
        *   Operatörler (`=`, `==`, `!=`): Atama ve karşılaştırma işlemleri için aşırı yüklenmiştir.
*   **Kullanım:** Genellikle `CAffect` veya `CHARACTER` sınıfları içinde, bir varlığın sahip olduğu genel etki türlerini (örn. zehir, hızlanma) hızlıca yönetmek için kullanılır. Her etki türüne karşılık gelen sabit bir bayrak numarası (`AFF_...`, muhtemelen `affect.h`'de tanımlı) bu yapı üzerinde `Set` veya `Reset` ile ayarlanır.
*   **Bağlantılı Dosyalar:** `affect.h` (muhtemelen `AFF_BITS_MAX` ve `AFF_...` sabitlerini içerir), `affect.cpp`.

### `affect.h`

*   **Amaç:** Oyun içindeki karakter etkilerini (Affects - buff/debuff) yönetmek için kullanılan temel sınıfları, enum'ları ve sabitleri tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CAffect` Sınıfı:**
        *   Tek bir aktif etkiyi temsil eder.
        *   Üyeler: `dwType` (etki türü - `EAffectTypes`), `wApplyOn` (uygulandığı özellik - `POINT_...`), `lApplyValue` (etki değeri), `dwFlag` (ilişkili bayrak - `EAffectBits`), `lDuration` (süre), `lSPCost` (muhtemelen eski).
        *   Statik Metotlar (`Acquire`, `Release`): `CAffect` nesneleri için nesne havuzu yönetimi sağlar.
    *   **`EAffectTypes` Enum'u:** Oyundaki tüm farklı etki türlerini (beceriler, iksirler, sistemler, premium vb.) numaralandırır. İstemci-sunucu iletişiminde kullanılır.
    *   **`EAffectBits` Enum'u:** `TAffectFlag` (`affect_flag.h`) yapısında kullanılacak bayrakları numaralandırır. Karakterin genel durumlarını (zehirli, sersemlemiş vb.) temsil eder. `AFF_BITS_MAX` bu enum'un sonunda yer alır ve `TAffectFlag`'ın boyutunu belirler (64'ü geçmemeli).
    *   **Harici Fonksiyon Bildirimleri (`extern`):** `affect.cpp`'de tanımlanan yardımcı fonksiyonlar (`GetAffectName`, `IsAffectFlag`, `SendAffectAddPacket`, `SendAffectRemovePacket`).
    *   **`AffectVariable` Enum'u:** `INFINITE_AFFECT_DURATION` sabitini (çok uzun bir süre) tanımlar.
*   **Kullanım:** Karakterlere veya nesnelere beceri, iksir veya sistemler aracılığıyla uygulanan geçici veya kalıcı etkileri yönetmek için kullanılır. `CHARACTER` sınıfı muhtemelen bir `CAffect` listesi veya benzeri bir yapı tutar ve `TAffectFlag` ile genel durumları takip eder.
*   **Bağlantılı Dosyalar:** `affect_flag.h`, `affect.cpp`, `char.h`, `char.cpp`, `skill.cpp`, `item.cpp` vb.

### `ani.h`

*   **Amaç:** Karakter animasyonları ile ilgili, özellikle saldırı ve kombo hızlarını almak ve yönetmek için kullanılan fonksiyonları bildirir.
*   **Temel İşlevler/İçerik:**
    *   `ani_init()`: Animasyon hız verilerini yükleyen başlatma fonksiyonu.
    *   `ani_attack_speed(LPCHARACTER ch)`: Belirtilen karakterin temel saldırı hızını döndürür (silah türüne göre).
    *   `ani_print_attack_speed()`: Yüklenmiş saldırı hızlarını yazdırmak için (muhtemelen hata ayıklama amaçlı).
    *   `ani_combo_speed(LPCHARACTER ch, BYTE combo)`: Belirtilen karakterin belirli bir kombo vuruşunun animasyon hızını döndürür (at üzerinde olup olmamasına göre değişir).
*   **Bağlantılı Dosyalar:** `ani.cpp`

### `any_function.h`

*   **Amaç:** Boost kütüphanesindeki `boost::any`'ye benzer, ancak özellikle fonksiyon nesnelerini (functors) veya çağrılabilir varlıkları (callables) depolamak için özelleştirilmiş tür silme (type erasure) mekanizması sağlayan `any_function` ve `any_void_function` türlerini tanımlar. Farklı imzalara sahip fonksiyonları tek bir türde saklamak için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`any_function` (typedef _boost_func_of_SQLMsg::any):** `SQLMsg*` türünde bir argüman alan fonksiyonları/çağrılabilirleri saklamak için özelleştirilmiş bir `any` türü. Muhtemelen veritabanı sorgu sonuçlarını işleyen callback fonksiyonları için kullanılır.
    *   **`any_void_function` (typedef _boost_func_of_void::any):** Argüman almayan (`void`) fonksiyonları/çağrılabilirleri saklamak için özelleştirilmiş bir `any` türü. Genel amaçlı olay işleyicileri veya ertelenmiş görevler için kullanılabilir.
    *   **`void_binder` Sınıf Şablonu:** Bir fonksiyon nesnesini (`F`) ve bu fonksiyonun beklediği argüman türünden bir değeri (`value`) saklar. `operator()` çağrıldığında, saklanan fonksiyonu saklanan değerle çağırır. Bu, argüman alan bir fonksiyonu, argüman almayan bir fonksiyona dönüştürmek için kullanılır (argümanı bağlayarak).
    *   **`void_bind` Fonksiyon Şablonu:** `void_binder` nesnesi oluşturmak için bir yardımcı (factory) fonksiyondur.
*   **Çalışma Prensibi:** Bu başlık dosyası, `any_function.inc` dosyasını farklı makro tanımlarıyla iki kez `#include` ederek iki farklı `any` uzmanlaşması (specialization) oluşturur. Bu uzmanlaşmalar, içlerinde farklı türlerde fonksiyon nesneleri saklayabilirler, ancak dışarıya bir arayüz sunarlar. `void_binder` ise, belirli bir argümanla bir fonksiyonu "sabitlemek" için kullanılır, bu da genellikle callback mekanizmalarında işe yarar.
*   **Potansiyel Kullanım:** Olay yöneticileri (event managers), görev zamanlayıcıları (task schedulers), veya callback gerektiren asenkron işlemler (örneğin veritabanı yanıtları).
*   **Bağlantılı Dosyalar:** `any_function.inc` (asıl `any` şablonunu içerir), `<algorithm>`, `<typeinfo>`.

#### `any_function.inc` (Detay)

*   **Amaç:** Bu dosya, `any_function.h` tarafından `#include` edilerek, farklı fonksiyon imzaları için `any` sınıfının özelleşmiş (specialized) versiyonlarını üreten şablon kodunu içerir. Makrolar aracılığıyla (örneğin `func_arg_type`) hangi fonksiyon imzası için kod üretileceği kontrol edilir.
*   **Temel İşlevler/İçerik:**
    *   Tür silme (type erasure) mekanizmasının çekirdeğini oluşturur: `placeholder` soyut taban sınıfı ve asıl değeri tutan `holder` şablon sınıfını tanımlar.
    *   `any` sınıfının yapıcılarını, yıkıcısını, atama operatörlerini, `swap` fonksiyonunu ve en önemlisi, saklanan fonksiyonu çağırmak için kullanılan `operator()`'ı içerir.
*   **Önemi:** Kod tekrarını önleyerek `any_function.h`'nin farklı fonksiyon türleri için `any` benzeri sınıflar tanımlamasını sağlar.

### `banword.h`

*   **Amaç:** Yasaklı kelimeleri (ban words) yönetmek için `CBanwordManager` singleton sınıfını tanımlar. Bu sınıf, yasaklı kelimelerin listesini tutar ve metinlerde bu kelimelerin varlığını kontrol etme veya sansürleme işlevleri sunar.
*   **Temel İşlevler/İçerik:**
    *   **`CBanwordManager` Sınıfı (Singleton):**
        *   `Initialize(TBanwordTable* p, WORD wSize)`: Veritabanından veya bir tablodan alınan yasaklı kelime listesiyle yöneticiyi başlatır.
        *   `Find(const char* c_pszString)`: Verilen metnin **tam olarak** yasaklı kelime listesinde olup olmadığını kontrol eder.
        *   `CheckString(const char* c_pszString, size_t _len)`: Verilen metnin **içinde** herhangi bir yasaklı kelime geçip geçmediğini kontrol eder (büyük/küçük harf duyarsız ve alt metin olarak arama yapar).
        *   `ConvertString(char* c_pszString, size_t _len)`: Verilen metindeki yasaklı kelimeleri '*' karakteri ile değiştirerek sansürler.
    *   **`m_hashmap_words` Üyesi:** Yasaklı kelimeleri verimli bir şekilde aramak için `boost::unordered_map` (hash map) içinde saklar. Anahtar kelimenin kendisi, değer ise genellikle `true`'dur.
*   **Bağlantılı Dosyalar:** `banword.cpp`, `constants.h` (muhtemelen `TBanwordTable` tanımı için), `singleton.h`, `<boost/unordered_map.hpp>`, `<string>`.

### `battle.h`

*   **Amaç:** Oyun içindeki savaş (combat) ile ilgili temel fonksiyonların, sabitlerin ve yardımcı makroların bildirimlerini içerir. Hasar hesaplama, saldırı geçerliliği kontrolü, saldırı etkileri (stun, slow vb.) ve saldırı hızı hilesi tespiti gibi işlevler için arayüz sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`EBattleTypes` Enum'u:** Bir savaş etkileşiminin sonucunu belirtir (kullanımda görünmüyor).
    *   **Hasar Hesaplama Fonksiyonları (`Calc*Damage`)**: Yakın dövüş, ok ve büyü hasarlarını hesaplamak için fonksiyon bildirimleri.
    *   **Saldırı Değerlendirme (`CalcAttackRating`)**: Saldıranın isabet oranını ve kurbanın kaçınma oranını hesaplar.
    *   **Savaş Kontrol Fonksiyonları:** Saldırı geçerliliği (`battle_is_attackable`), maksimum saldırı mesafesi (`battle_get_max_distance`), yakın dövüş saldırısı başlatma (`battle_melee_attack`), savaşı bitirme (`battle_end`) ve mesafe geçerliliği (`battle_distance_valid*`) fonksiyonları.
    *   **Etki Uygulama Fonksiyonları/Makroları:** Normal saldırı etkileri (`NormalAttackAffect`) ve genel/beceri etkileri (`AttackAffect`, `SkillAttackAffect`) için yardımcılar.
    *   **Saldırı Hızı Hilesi Tespiti:** Saldırı hızı hesaplama (`GET_ATTACK_SPEED`), zaman kaydı (`SET_ATTACK_TIME`, `SET_ATTACKED_TIME`) ve hile kontrolü (`IS_SPEED_HACK`) için fonksiyon/makro bildirimleri.
*   **Bağlantılı Dosyalar:** `battle.cpp`, `char.h`.

### `block_country.h`

*   **Amaç:** Ülke bazlı IP engelleme ve istisna yönetimi için fonksiyon bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   `add_blocked_country_ip(TPacketBlockCountryIp* data)`: Engellenmiş IP aralığı bilgisini sisteme ekler.
    *   `block_exception(TPacketBlockException* data)`: Belirli bir kullanıcı hesabı için IP engelleme istisnası ekler veya kaldırır.
    *   `is_blocked_country_ip(const char* user_ip)`: Verilen bir IP adresinin engellenmiş aralıklardan birine düşüp düşmediğini kontrol eder.
    *   `is_block_exception(const char* login)`: Verilen bir kullanıcı hesabının engelleme istisnası listesinde olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `block_country.cpp`, `packet_info.h` (muhtemelen `TPacket*` tanımları için).

### `buff_on_attributes.h`

*   **Amaç:** Belirli giyilen eşyaların efsunlarını (attributes) yüzdesel olarak artıran bir buff'ı yönetmek için `CBuffOnAttributes` sınıfını tanımlar. Buff aktifken, ilgili eşyalar giyilip çıkarıldığında veya buff değeri değiştiğinde karakterin statlarını dinamik olarak günceller.
*   **Temel İşlevler/İçerik:**
    *   **`CBuffOnAttributes` Sınıfı:**
        *   Buff sahibini, buff türünü ve etkilenecek eşya slotlarını (`WEAR_*`) tutar.
        *   `On(BYTE bValue)` / `Off()`: Buff'ı belirtilen yüzdeyle açar veya kapatır.
        *   `ChangeBuffValue(BYTE bNewValue)`: Buff'ın yüzdesini değiştirir.
        *   `AddBuffFromItem(LPITEM pItem)` / `RemoveBuffFromItem(LPITEM pItem)`: İlgili slotlara eşya giyildiğinde/çıkarıldığında çağrılarak, o eşyanın efsunlarından gelen buff etkisini ekler/kaldırır.
        *   `GiveAllAttributes()`: Mevcut birikmiş efsunlara göre tüm buff etkilerini karaktere uygular.
        *   Birikmiş efsunları (tür -> toplam değer) bir harita (`m_map_additional_attrs`) içinde saklar.
*   **Kullanım Alanı:** Örneğin, bir beceri aktifken sadece silah ve zırhtaki efsunların %10 daha fazla etki etmesi gibi mekanizmalar oluşturmak için kullanılabilir.
*   **Bağlantılı Dosyalar:** `buff_on_attributes.cpp`, `char.h`, `item.h`, `<vector>`, `<map>`.

### `buffer_manager.h`

*   **Amaç:** Geçici bellek tamponlarını (`LPBUFFER`, muhtemelen `libthecore`'dan) yönetmek için `TEMP_BUFFER` isimli bir RAII sarmalayıcı sınıfı tanımlar. Tampon oluşturma, veri yazma/okuma ve otomatik silme işlemlerini kolaylaştırır.
*   **Temel İşlevler/İçerik:**
    *   **`TEMP_BUFFER` Sınıfı:**
        *   Yapıcı: Belirtilen boyutta bir tampon oluşturur (`buffer_new`).
        *   Yıkıcı: Kapsam dışına çıkıldığında tamponu otomatik olarak siler (`buffer_delete`).
        *   `write()`: Tampona veri yazar.
        *   `read_peek()`: Tampondaki veriyi okuma pozisyonunu ilerletmeden okur.
        *   `size()`: Tampondaki veri boyutunu döndürür.
        *   `reset()`: Tamponu sıfırlar.
        *   `getptr()`: İçerideki ham `LPBUFFER` işaretçisini döndürür.
*   **Kullanım Alanı:** Özellikle ağ paketleri oluştururken veya geçici veri işleme sırasında sıkça kullanılır. Otomatik silme özelliği sayesinde bellek sızıntılarını önlemeye yardımcı olur.
*   **Bağlantılı Dosyalar:** `buffer_manager.cpp`, `buffer.h` (`libthecore`).

### `constants.h`

*   **Amaç:** Oyun sunucusu genelinde kullanılan temel sabitleri, veri yapılarını (struct'lar), enum'ları ve global sabit veri tablolarının harici (`extern`) bildirimlerini tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabit Tanımları:** `ADDRESS_MAX_LEN`, `FORTUNE_MAX_NUM`, `STONE_INFO_MAX_NUM`, `MAX_EXP_DELTA_OF_LEV`, `ARROUND_COORD_MAX_NUM` gibi sayısal sabitler.
    *   **Enum Tanımları:** `EMonsterChatState`, `EItemMiscSubTypes`, `EGuildWarType` gibi oyun içi durumları ve türleri temsil eden sembolik sabitler.
    *   **Struct Tanımları:** `SMobRankStat`, `SMobStat`, `SBattleTypeStat`, `SJobInitialPoints`, `Coord`, `SApplyInfo`, `SStoneDropInfo`, `TGuildWarInfo` gibi oyun verilerini gruplayan yapılar.
    *   **Extern Veri Tablosu Bildirimleri:** `constants.cpp` dosyasında tanımlanan ve oyun mekaniklerinde kullanılan global dizilerin ve haritaların bildirimleri. Örnekler:
        *   Deneyim Tabloları: `exp_table`, `conqueror_exp_table`, `exp_pet_table`, `guild_exp_table`.
        *   Statü/Puan Tabloları: `JobInitialPoints`, `aApplyInfo`, `aiSkillPowerByLevel`, `aiPolymorphPowerByLevel`, `aiSkillPrecision`.
        *   Mob/Savaş Verileri: `MobRankStats`, `BattleTypeStats`, `aiMobEnchantApplyIdx`, `aiMobResistsApplyIdx`, `aSkillAttackAffectProbByRank`.
        *   Parti/Seviye Farkı Bonusları: `party_exp_distribute_table`, `aiPercentByDeltaLev`, `*aiPartyBonusExpPercentByMemberCount`.
        *   Eşya/Efsun Verileri: `g_map_itemAttr`, `g_map_itemRare`, `aiItemMagicAttributePercent*`, `aiWeaponSocketQty`, `aiArmorSocketQty`, `aiAccessorySocket*`.
        *   Diğer: `aArroundCoords`, `aiSkillBookCount*`, `aiExpLossPercents`, `aStoneDrop`, `c_apszEmpireNames`, `c_apszPrivNames`, `KOR_aGuildWarInfo`.
    *   **Makro Tanımları:** `PERCENT_LVDELTA`, `CALCULATE_VALUE_LVDELTA` gibi hesaplama kısayolları.
    *   **Fonksiyon Bildirimleri:** `FN_get_apply_type` gibi yardımcı fonksiyonların prototipleri.
*   **Bağlantılı Dosyalar:** `constants.cpp` (uygulama) ve bu sabitleri, yapıları veya veri tablolarını kullanan hemen hemen tüm diğer `game/src` dosyaları.

### `event.h`

*   **Amaç:** Oyun sunucusundaki zamanlanmış olayları (timed events) yönetmek için temel yapıları, tür tanımlarını ve fonksiyon bildirimlerini içerir. Bu sistem, belirli bir süre sonra veya periyodik olarak çalışması gereken eylemleri (örn. buff süreleri, canavar yeniden doğma, görev zamanlayıcıları) zamanlamak için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`event_info_data` Struct:** Tüm olaylara özgü verilerin türetilmesi gereken temel sınıf. İsteğe bağlı olarak (`M2_USE_POOL`) verimli bellek yönetimi için nesne havuzu (`MemoryPool`) kullanır.
    *   **`EVENT` Struct:** Tek bir zamanlanmış olayı temsil eder. Şunları içerir:
        *   `func` (`TEVENTFUNC`): Olay gerçekleştiğinde çağrılacak işleyici fonksiyonun işaretçisi.
        *   `info` (`event_info_data*`): Olaya özgü verileri tutan işaretçi.
        *   `q_el` (`TQueueElement*`): Olayın kuyruktaki temsilcisinin işaretçisi.
        *   `is_force_to_end` (char): Olay işlenirken dışarıdan iptal edilip edilmediğini belirten bayrak.
        *   `is_processing` (char): Olayın o anda işlenip işlenmediğini belirten bayrak.
        *   `ref_count` (size_t): `boost::intrusive_ptr` ile referans sayımı için sayaç.
    *   **`LPEVENT` Typedef:** `EVENT` nesnelerinin ömrünü otomatik olarak yönetmek için kullanılan referans sayımlı akıllı işaretçi (`boost::intrusive_ptr<EVENT>`).
    *   **`TEVENTFUNC` Typedef:** Olay işleyici fonksiyonlarının imza türü: `long (LPEVENT event, long processing_time)`. Dönüş değeri, bir sonraki çalışma zamanı (vuruş/pulse cinsinden) veya olayı bitirmek için 0/negatif bir değerdir.
    *   **Makrolar:**
        *   `EVENTFUNC(name)`: Olay işleyici fonksiyonlarını kolayca tanımlamak için kullanılır.
        *   `EVENTINFO(name)`: `event_info_data`'dan türeyen olay bilgi yapılarını kolayca tanımlamak için kullanılır.
        *   `event_create(func, info, when)`: `event_create_ex` fonksiyonu için bir kısayoldur.
    *   **Fonksiyon Bildirimleri:**
        *   `AllocEventInfo<T>()`: Olay bilgi verisi (`event_info_data` türevi) ayırmak için şablon fonksiyon (havuz veya `M2_NEW` kullanır).
        *   `event_destroy()`: Kuyruktaki tüm olayları yok eder.
        *   `event_process(int pulse)`: Zamanı gelen olayları işleyen ana fonksiyon.
        *   `event_count()`: Bekleyen olayların sayısını döndürür.
        *   `event_create_ex(func, info, when)`: Belirtilen süre (`when`) sonra çalışacak yeni bir olay oluşturur.
        *   `event_cancel(LPEVENT* event)`: Bekleyen bir olayı iptal eder.
        *   `event_processing_time(LPEVENT event)`: Bir olayın ne kadar süredir işlendiğini (vuruş cinsinden) döndürür.
        *   `event_time(LPEVENT event)`: Bir olayın bir sonraki çalışmasına kalan süreyi (vuruş cinsinden) döndürür.
        *   `event_reset_time(LPEVENT event, long when)`: Mevcut bir olayın çalışma zamanını yeniden ayarlar.
    *   **Referans Sayımı Fonksiyonları:** `intrusive_ptr_add_ref`, `intrusive_ptr_release`.
*   **Bağlantılı Dosyalar:** `event.cpp`, `<boost/intrusive_ptr.hpp>`, (isteğe bağlı) `pool.h`. (Dolaylı olarak: `event_queue.h`).

---

## Kaynak Kod Dosyaları (.cpp)

### `affect.cpp`

*   **Amaç:** `affect.h`'de tanımlanan `CAffect` sınıfı için nesne havuzu (object pool) yönetimini uygular.
*   **Temel İşlevler/İçerik:**
    *   **Nesne Havuzu Yönetimi:**
        *   `CAffect::Acquire()`: `CAffect` nesnelerini bellekten ayırır. Derleme yapılandırmasına (`DEBUG_ALLOC` makrosu) bağlı olarak ya `boost::object_pool` kullanarak havuzdan bir nesne alır ya da doğrudan `M2_NEW` ile yeni bir nesne oluşturur. Bu, sık sık `CAffect` nesnesi oluşturma/yok etme maliyetini azaltmaya yardımcı olur.
        *   `CAffect::Release(CAffect* p)`: Kullanımı biten `CAffect` nesnelerini havuza geri verir (`boost::object_pool::free`) veya belleği serbest bırakır (`M2_DELETE`).
*   **Önemli Not:** Bu dosya, `affect.h`'de `extern` olarak bildirilen yardımcı fonksiyonların (`GetAffectName`, `IsAffectFlag`, `SendAffectAddPacket`, `SendAffectRemovePacket` vb.) tanımlarını **içermez**. Bu fonksiyonlar genellikle karakterlerle ilgili işlemleri içerdiğinden muhtemelen `char_affect.cpp` gibi karakter yönetimiyle ilgili bir dosyada tanımlanmıştır.
*   **Bağlantılı Dosyalar:** `affect.h`, `stdafx.h`, (isteğe bağlı) `boost/pool/object_pool.hpp`.

### `ani.cpp`

*   **Amaç:** `ani.h`'de bildirilen fonksiyonları uygular. Karakterlerin ırklarına, kullandıkları silah türlerine, kombo numaralarına ve at üzerinde olup olmamalarına göre saldırı animasyon hızlarını `.msa` dosyalarından yükler ve yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`ANI` Sınıfı:**
        *   `m_speed[MAIN_RACE_MAX_NUM][2][WEAPON_NUM_TYPES][9]`: Tüm ırklar, atlı/atsız durumlar, silah türleri ve 8 komboya kadar olan animasyon hızlarını (milisaniye cinsinden) depolayan çok boyutlu bir dizi. 0. indeks genellikle en hızlı kombo hızını depolar.
        *   `load()`: Tüm ırklar için animasyon hız verilerini yükler (`load_one_race` çağırır).
        *   `load_one_race()`: Belirli bir ırk için tüm silah türlerinin animasyon hızlarını yükler (`load_one_weapon` çağırır).
        *   `load_one_weapon()`: Belirli bir silah türü, kombo numarası ve at durumu için ilgili `.msa` dosyasını (`data/pc/.../*.msa`) okur ve hızı alır (`FN_attack_speed_from_file` kullanır).
        *   `attack_speed()`: Parametre olarak verilen ırk, silah, kombo ve at durumuna göre önceden yüklenmiş hızı `m_speed` dizisinden döndürür.
    *   **`s_ANI` Statik Nesnesi:** `ANI` sınıfının tekil (singleton) bir örneği. Tüm genel fonksiyonlar bu nesne üzerinden çalışır.
    *   **`FN_attack_speed_from_file()`:** Bir `.msa` dosyasını açar, içindeki `DirectInputTime` anahtarını arar ve karşılık gelen değeri (float) okuyup milisaniye cinsinden (int) döndürür.
    *   **Genel Arayüz Fonksiyonları (`ani_init`, `ani_attack_speed`, `ani_combo_speed`, `ani_print_attack_speed`):** `s_ANI` nesnesinin ilgili metotlarını çağıran sarmalayıcı (wrapper) fonksiyonlardır. `ani_attack_speed` ve `ani_combo_speed` karakter nesnesinden (`LPCHARACTER`) gerekli bilgileri (ırk, silah, at durumu) alır.
*   **Çalışma Prensibi:** Sunucu başlatıldığında `ani_init()` çağrılır. Bu fonksiyon `s_ANI.load()`'u tetikler, o da ilgili `.msa` dosyalarını okuyarak tüm olası animasyon hızlarını `s_ANI.m_speed` dizisine yükler. Oyun sırasında bir karakterin saldırı hızı gerektiğinde (`ani_attack_speed` veya `ani_combo_speed` çağrıldığında), bu diziye hızlıca erişilerek önceden yüklenmiş değer döndürülür. Bu, her seferinde dosya okuma ihtiyacını ortadan kaldırır.
*   **Bağlantılı Dosyalar:** `ani.h`, `stdafx.h`, `char.h`, `item.h`, `dev_log.h`.

### `banword.cpp`

*   **Amaç:** `banword.h`'de bildirilen `CBanwordManager` sınıfının metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **`Initialize`:** Verilen `TBanwordTable` dizisindeki tüm kelimeleri `m_hashmap_words` hash map'ine ekler ve bir log mesajı gönderir.
    *   **`Find`:** Doğrudan hash map üzerinde `find` işlemini kullanarak tam eşleşme arar.
    *   **`CheckString`:**
        1.  Girdi metnini küçük harfe çevirir (Büyük/küçük harf duyarsız kontrol için).
        2.  Hash map'teki her yasaklı kelime için, girdi metninin içinde bu kelimenin geçip geçmediğini arar.
        3.  Arama sırasında metni karakter karakter veya iki byte'lık karakterler (Çince/Korece gibi diller için `is_twobyte`) halinde dolaşır ve `strncmp` ile alt metin eşleşmesi kontrolü yapar.
    *   **`ConvertString`:**
        1.  Hash map'teki her yasaklı kelime için girdi metnini dolaşır.
        2.  `CheckString`'e benzer şekilde, metni karakter karakter veya iki byte'lık karakterler halinde ilerler.
        3.  Eğer bir yasaklı kelime ile eşleşme bulunursa, eşleşen bölümü `memset` kullanarak '*' karakterleriyle doldurur.
*   **Kullanım Alanları:** Oyuncu isimleri, sohbet mesajları, lonca isimleri gibi kullanıcı girdilerini kontrol etme ve filtrelemek için kullanılır.
*   **Bağlantılı Dosyalar:** `banword.h`, `stdafx.h`, `constants.h`, `config.h`, `<string>`, `<algorithm>`.

### `battle.cpp`

*   **Amaç:** `battle.h`'de bildirilen savaş mekanikleriyle ilgili fonksiyonları uygular. Hasar hesaplamalarının detaylarını, saldırı etkilerinin uygulanmasını, mesafe kontrollerini ve çeşitli savaş senaryolarının (PvP, PvM, lonca savaşları, arena vb.) yönetimini içerir.
*   **Temel İşlevler/İçerik:**
    *   **Mesafe ve Saldırı Geçerliliği (`battle_distance_valid*`, `battle_is_attackable`):** İki karakterin birbirine saldırıp saldıramayacağını ve mesafenin uygun olup olmadığını çeşitli kurallara (güvenli bölge, PK modu, arena, lonca savaşı vb.) göre kontrol eder.
    *   **Yakın Dövüş Saldırısı (`battle_melee_attack`):** Saldırı animasyonu, zamanlama, mesafe ve geçerlilik kontrollerini yaptıktan sonra asıl hasar hesaplama ve uygulama için `battle_hit` fonksiyonunu çağırır.
    *   **Hasar Hesaplama (`Calc*Damage`, `CalcAttackRating`, `CalcAttBonus`):**
        *   Saldıranın ve kurbanın statları (STR, DEX, INT), seviyeleri, giydiği eşyalar (silah, zırh), bonuslar (Yarı İnsanlara Karşı Güç, Kritik Vuruş vb.), dirençler (Kılıç Savunması, Büyü Savunması vb.), beceri etkileri ve diğer faktörleri (ırk, sınıf, mesafe vb.) dikkate alarak karmaşık hasar hesaplamalarını yapar.
        *   `CalcAttackRating`, isabet/kaçınma oranını DEX ve seviyeye göre belirler.
        *   `CalcAttBonus`, türe/sınıfa özel bonusları ve dirençleri uygular.
        *   Farklı saldırı türleri (yakın dövüş, ok, büyü) için ayrı hesaplama mantıkları bulunur.
    *   **Asıl Vuruş ve Etkiler (`battle_hit`, `NormalAttackAffect`):**
        *   `battle_hit`, yakın dövüş hasarını hesaplar (`CalcMeleeDamage`), standart saldırı etkilerini uygular (`NormalAttackAffect`), element ve silah türü dirençlerini hesaba katar ve son hasarı kurbana uygular (`pkVictim->Damage()`).
        *   `NormalAttackAffect`, normal saldırılarda şansa bağlı olarak zehir, kanama, sersemletme, yavaşlatma gibi etkileri uygular.
    *   **Saldırı Hızı ve Hile Tespiti (`GET_ATTACK_SPEED`, `IS_SPEED_HACK`, vb.):** Karakterin animasyon hızına ve bonuslarına göre minimum saldırı süresini hesaplar ve oyuncuların bu süreden daha hızlı saldırması durumunda bunu tespit etmeye çalışır.
*   **Bağlantılı Dosyalar:** `battle.h`, `stdafx.h`, `utils.h`, `config.h`, `char.h`, `item.h`, `mob_manager.h`, `pvp.h`, `guild.h`, `affect.h`, `arena.h`, `castle.h`, `sectree.h`, `ani.h`, `locale_service.h` ve diğerleri.

### `block_country.cpp`

*   **Amaç:** `block_country.h`'de bildirilen IP engelleme ve istisna yönetimi fonksiyonlarını uygular. Engellenen IP aralıklarını ve istisna hesaplarını bellekte saklar ve kontrol mekanizmalarını sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Veri Yapıları:** Engellenen IP aralıklarını (`T_BLOCK_IP` struct'ı içeren `s_blocked_ip` vektörü) ve istisna kullanıcı adlarını (`s_block_exception` seti) bellekte tutar.
    *   **`add_blocked_country_ip`:** Gelen IP aralığını `s_blocked_ip` vektörüne ekler.
    *   **`block_exception`:** Gelen komuta göre istisna listesine (`s_block_exception`) kullanıcı ekler veya çıkarır.
    *   **`is_blocked_country_ip`:** Verilen IP adresini sayısal formata çevirir ve `s_blocked_ip` listesindeki aralıklara düşüp düşmediğini kontrol eder.
    *   **`is_block_exception`:** Verilen kullanıcı adının `s_block_exception` set'inde olup olmadığını kontrol eder.
*   **Çalışma Prensibi:** Sunucu başlangıcında veya komutlarla engelli IP aralıkları ve istisnalar belleğe yüklenir. Kullanıcı bağlantı isteği geldiğinde, önce istisna listesi kontrol edilir, ardından IP adresi engelli listesiyle karşılaştırılır ve bağlantı kararı verilir.
*   **Bağlantılı Dosyalar:** `block_country.h`, `stdafx.h`, `constants.h`, `dev_log.h`, network kütüphaneleri, `<vector>`, `<set>`, `<string>`.

### `buff_on_attributes.cpp`

*   **Amaç:** `buff_on_attributes.h`'de bildirilen `CBuffOnAttributes` sınıfının metotlarını uygular. Eşya giyme/çıkarma ve buff değeri değişimi durumlarında efsunları toplama, çıkarma ve karakterin statlarına uygulama mantığını içerir.
*   **Temel İşlevler/İçerik:**
    *   **`On`:** Buff aktif edildiğinde, belirtilen slotlardaki tüm eşyaların efsunlarını okur, türlerine göre toplar (`m_map_additional_attrs`) ve buff yüzdesini uygulayarak sonucu karaktere ekler (`ApplyPoint`).
    *   **`Off`:** Buff kapatıldığında, `m_map_additional_attrs`'daki her efsun türü için, mevcut buff yüzdesiyle hesaplanan değeri karakterden çıkarır (`ApplyPoint` negatif değerle).
    *   **`AddBuffFromItem` / `RemoveBuffFromItem`:** Eşya giyilip çıkarıldığında, sadece değişen eşyanın efsunlarını `m_map_additional_attrs` haritasına ekler/çıkarır ve karakterin statlarındaki *farkı* hesaplayarak günceller. Bu, tüm eşyaları tekrar taramaktan daha verimlidir.
    *   **`ChangeBuffValue`:** Eski buff değerini karakterden çıkarır, buff yüzdesini günceller ve (ideal olarak) yeni değeri uygular (mevcut kodda uygulama kısmı eksik görünüyor, muhtemelen dışarıdan `GiveAllAttributes` çağrılması bekleniyor).
    *   **`GiveAllAttributes`:** Haritadaki tüm birikmiş efsunlara mevcut buff yüzdesini uygulayarak karakterin statlarını günceller.
*   **Çalışma Prensibi:** Bu sınıf, bir karakter nesnesi içinde belirli bir buff türü için örneklenir. Buff `On` ile açıldığında, ilgili eşyalardaki efsunlar toplanır ve etkisi uygulanır. Karakter eşya değiştirdiğinde `AddBuffFromItem`/`RemoveBuffFromItem` çağrılarak sadece fark kadar güncelleme yapılır. Buff değeri değişirse veya kapanırsa, etkiler buna göre ayarlanır.
*   **Bağlantılı Dosyalar:** `buff_on_attributes.h`, `stdafx.h`, `item.h`, `char.h`, `tables.h`, `<algorithm>`.

### `buffer_manager.cpp`

*   **Amaç:** `buffer_manager.h`'de bildirilen `TEMP_BUFFER` sınıfının metotlarını uygular. Temel olarak `libthecore`'daki ilgili `buffer_*` fonksiyonlarına çağrıları yönlendirir.
*   **Temel İşlevler/Metotlar:**
    *   Yapıcı: `buffer_new` ile tamponu oluşturur.
    *   Yıkıcı: `buffer_delete` ile tamponu siler.
    *   Diğer Metotlar: `buffer_read_peek`, `buffer_write`, `buffer_size`, `buffer_reset` gibi fonksiyonları çağırır.
*   **Bağlantılı Dosyalar:** `buffer_manager.h`, `stdafx.h`.

### `cmd_emotion.cpp`

*   **Amaç:** Karakterlerin sosyal eylemlerini ve duygusal ifadelerini (öpücük, tokat, danslar, tezahürat vb.) yöneten sunucu tarafı komutlarını (`ACMD`) uygular. Oyuncuların bu eylemleri tetiklemesini, koşulların kontrol edilmesini ve animasyonların istemcilere bildirilmesini sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`emotion_types[]` Dizisi:** Her bir duygu ifadesi için sunucu/istemci komut adlarını, gerekli bayrakları (hedef gerekli mi, sadece kadınlar mı, karşı cins mi, silah bırakma gerekli mi - `NEED_TARGET`, `WOMAN_ONLY`, `OTHER_SEX_ONLY`, `SELF_DISARM`, `TARGET_DISARM`) ve ek animasyon gecikmesini tanımlayan statik bir yapı dizisi.
    *   **`ACMD(do_emotion)`:** Tüm duygu komutları için ana işleyici fonksiyon.
        *   Temel kontroller yapar: At üzerinde mi, yakın zamanda hareket etmiş veya saldırmış mı?
        *   Duygu Maskesi takma gereksinimini `CHARACTER_CanEmotion` ile kontrol eder.
        *   Cinsiyet kısıtlamalarını kontrol eder (`WOMAN_ONLY`, `OTHER_SEX_ONLY`).
        *   Hedef gerektiren eylemler için hedef karakteri bulur ve kontrolleri (mesafe, PC durumu, at durumu) yapar.
        *   Karşılıklı onay gerektiren PC'ler arası eylemler (`NEED_PC` bayrağı) için `s_emotion_set` üzerinden onayı veya evlilik durumunu kontrol eder.
        *   Silah bırakma (`*_DISARM`) bayraklarını kontrol eder (Not: Kodun bu bayrakları kontrol etmesine rağmen, silahların gerçekten bırakılıp bırakılmadığını kontrol etme mantığı eksik görünüyor).
        *   Tüm kontroller geçerse, `CHAT_TYPE_COMMAND` türünde ve `HEADER_GC_CHAT` başlığıyla bir paket oluşturur (örneğin, `"kiss 12345 67890"`). Bu paket `PacketAround` ile yakındaki istemcilere gönderilerek animasyonun tetiklenmesi sağlanır.
    *   **`ACMD(do_emotion_allow)`:** Bir oyuncunun başka bir oyuncuya (`DWORD` ile belirtilen VID) kendisine karşı çiftli duygu ifadesi kullanma izni vermesini sağlayan komutu işler. İzin verilen çifti `s_emotion_set`'e ekler. Arena ve Mesaj Engelleme (`__MESSENGER_BLOCK_SYSTEM__`) kontrolleri içerir.
    *   **`CHARACTER_CanEmotion(CHARACTER& rch)`:** Bir karakterin duygu ifadesi kullanıp kullanamayacağını belirler. Varsayılan olarak izin verilmez, ancak genel ayar (`g_bDisableEmotionMask`), Evlilik Haritası veya Duygu Maskesi (`UNIQUE_ITEM_EMOTION_MASK*`) takılıysa izin verilir.
    *   **`s_emotion_set`:** Çiftli duygu ifadeleri için geçici onayları (izin veren VID, izin verilen VID) saklayan `std::set`. Eylem gerçekleştirildiğinde bu setten giriş silinmez, ancak `do_emotion` içindeki kontrol sadece varlığına bakar.
*   **Çalışma Prensibi:** Oyuncu bir duygu komutu (/öpücük gibi) girdiğinde, ilgili `ACMD(do_emotion)` fonksiyonu çağrılır. Sunucu, komuta karşılık gelen `emotion_types` girdisindeki bayrakları ve oyuncu/hedef durumunu kontrol eder. Tüm koşullar sağlanırsa, sunucu istemciye animasyonu oynatması için özel bir komut gönderir (`HEADER_GC_CHAT` ile). Çiftli eylemler için `/duygu_izni` komutuyla önceden onay alınması gerekebilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `char.h`, `char_manager.h`, `motion.h`, `packet.h`, `buffer_manager.h`, `unique_item.h`, `wedding.h`, `config.h`, (isteğe bağlı) `messenger_manager.h`.

### `cmd_general.cpp`

*   **Amaç:** Oyuncuların kullandığı çok çeşitli genel komutları (`ACMD`) uygular. Bu komutlar karakter eylemleri, oyun durumu yönetimi, karakter özelleştirme, sosyal etkileşimler, sistem etkileşimleri (depo, pazar, küp vb.), hükümdar komutları ve diğer yardımcı işlevleri kapsar.
*   **Temel İşlev Grupları ve Önemli Komutlar:**
    *   **At/Binek:** `do_user_horse_ride` (bin/in), `do_user_horse_back` (gönder), `do_user_horse_feed` (besle), `do_ride` (genel bin/in), `do_unmount`.
    *   **Oyun/Oturum:** `do_cmd` (çıkış/karakter seçimi), `do_shutdown` (sunucu kapatma - GM), `do_restart` (yeniden başlama), `do_escape` (sıkışınca ışınlanma).
    *   **Statü/Beceri:** `do_stat_reset`, `do_stat_minus`, `do_stat`, `do_conqueror_point` (Fatih puanı), `do_skillup` (beceri yükseltme), `do_guildskillup` (lonca becerisi - lider).
    *   **Sosyal/Etkileşim:** `do_pvp` (PvP isteği), `do_ungroup` (partiden ayrılma), `do_war` (savaş ilanı - lider), `do_nowar` (savaş reddi/barış - lider), `do_pkmode` (PK modu değiştirme), `do_messenger_auth` (haberci isteği onayı), `do_setblockmode` (engelleme modu), `do_view_equip` (ekipman görme - GM), `do_party_request` (parti isteği gönder), `do_party_request_accept`/`deny` (parti isteği kabul/red), `do_dice` (zar atma).
    *   **Depo/Market/Mağaza:** `do_safebox_close`/`password`/`change_password` (Depo işlemleri), `do_mall_password`/`close` (Nesne Market Deposu), `do_close_shop` (özel pazar kapatma), `do_in_game_mall` (Nesne Market web arayüzünü açma), `do_click_mall` (Nesne Market şifre penceresini açma).
    *   **Hükümdar:** `do_monarch_warpto` (ışınlanma), `do_monarch_transfer` (çekme), `do_monarch_info` (bilgi), `do_elect` (seçim arayüzü), `do_monarch_tax` (vergi ayarlama), `do_monarch_mob` (canavar çağırma).
    *   **Hareket/Gözlemci:** `do_set_walk_mode`, `do_set_run_mode`, `do_observer_exit`.
    *   **Diğer/Yardımcı:** `do_fishing` (balık tutma), `do_console` (istemci konsolu), `do_detaillog` (detaylı log - GM), `do_monsterlog` (canavar log - GM), `do_costume` (kostüm bilgisi), `do_hair` (saç affect bilgisi), `do_inventory` (envanter listesi - GM), `do_gift` (hediye bildirimi), `do_cube` (küp sistemi), `do_hide_costume` (kostüm gizleme), `do_move_channel` (kanal değiştirme), `do_sort_special_inventory` (özel envanter sıralama).
*   **Zamanlanmış Olaylar (`timed_event`, `shutdown_event`):** Oyundan çıkış, karakter değiştirme ve sunucu kapatma gibi işlemlerin belirli bir süre sonra gerçekleşmesini sağlayan event mekanizmalarını içerir. Çeşitli kontroller (savaş durumu, hack kontrolü) ile bu işlemlerin anında gerçekleşmesini engelleyebilir.
*   **Bağlantılı Dosyalar:** Çok sayıda başlık dosyası içerir (`stdafx.h`, `char.h`, `item.h`, `packet.h`, `affect.h`, `pvp.h`, `party.h`, `guild_manager.h`, `questmanager.h`, `config.h`, `utils.h` vb.).

### `constants.cpp`

*   **Amaç:** `constants.h` dosyasında bildirilen global sabit veri tablolarını ve dizilerini tanımlar ve ilk değerlerini atar. Oyun mekaniklerinin ve dengesinin temelini oluşturan somut sayısal değerleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **Veri Tablolarının Başlatılması:** Oyunun temel işleyişi için kritik olan birçok statik veri tablosunu tanımlar:
        *   `JobInitialPoints`: Karakter sınıflarının başlangıç statüleri ve seviye başına kazanımları.
        *   `MobRankStats`, `BattleTypeStats`: Mob rütbeleri ve savaş türlerine göre istatistiksel ayarlamalar.
        *   `exp_table_common`, `conqueror_exp_table`, `exp_pet_table_common`, `guild_exp_table`: Oyuncu, Fatih, Pet ve Lonca seviyeleri için gereken deneyim miktarları.
        *   `aiPercentByDeltaLev*`: Seviye farkına göre EXP kazanım yüzdeleri.
        *   `party_exp_distribute_table`, `*aiPartyBonusExpPercentByMemberCount`: Parti EXP dağılımı ve bonusları.
        *   `aArroundCoords`: Alan etkili işlemler için göreceli koordinatlar.
        *   `aiMobEnchantApplyIdx`, `aiMobResistsApplyIdx`, `aiMobElementsApplyIdx`: Mob özelliklerinin efsun türü (`APPLY_*`) sabitlerine eşlenmesi.
        *   `aApplyInfo`: Efsun türü (`APPLY_*`) sabitlerinin karakter puan türü (`POINT_*`) sabitlerine eşlenmesi.
        *   `aiItemMagicAttributePercent*`, `aiItemAttributeAddPercent`: Eşya efsunlarının gelme ve eklenme olasılıkları.
        *   `aiWeaponSocketQty`, `aiArmorSocketQty`, `aiSocketPercentByQty`: Eşya türlerine göre soket sayıları ve ekleme yüzdeleri.
        *   `aiExpLossPercents`: Ölümde kaybedilen EXP yüzdeleri.
        *   `aiSkillPowerByLevel*`: Yetenek seviyesine göre güç çarpanları.
        *   `aiPolymorphPowerByLevel`, `aiSkillPrecision`: Dönüşüm ve Fatih seviyesi yetenek hassasiyeti değerleri.
        *   `aiSkillBookCount*`, `aiGrandMasterSkillBook*`: Yetenek kitabı gereksinimleri.
        *   `aStoneDrop`: Metin taşı düşürme bilgileri.
        *   `c_apszEmpireNames`, `c_apszPrivNames`: İmparatorluk ve bonus isimleri.
        *   `KOR_aGuildWarInfo`: Lonca savaşı harita ve skor bilgileri.
        *   `aiAccessorySocket*`: Aksesuar soketiyle ilgili yüzdeler ve süreler.
        *   `c_aApplyTypeNames`: Efsun isimlerinin (string) `APPLY_*` sabitlerine eşlenmesi.
        *   Sisteme özgü sabitler (Aura, Dawnmist Zindanı, Ruh Sistemi vb.).
    *   **Global Haritaların Başlatılması:** `g_map_itemAttr`, `g_map_itemRare`, `g_map_SungmaTable` haritaları tanımlanır (veriler muhtemelen başka yerden yüklenir).
    *   **Yardımcı Fonksiyonlar:**
        *   `FN_get_apply_type(const char* apply_type_string)`: Verilen efsun ismine (string) karşılık gelen `APPLY_*` sabitini döndürür. `c_aApplyTypeNames` dizisini kullanır.
        *   `GetAuraRefineInfo`: Aura sistemiyle ilgili bilgileri döndürür.
*   **Önemli Notlar:** Bu dosyadaki tablolar oyunun temel dengesini (EXP eğrisi, statü kazanımı, düşme oranları, yetenek gücü vb.) doğrudan etkiler. Bazı tablolar (`exp_table`, `aiPercentByDeltaLev` gibi) işaretçi olarak tanımlanır ve gerçek değerleri `locale_service.cpp` gibi başka dosyalarda yerelleştirme ayarlarına göre atanır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `char.h`.

### `entity.h`

*   **Amaç:** Oyundaki tüm varlıklar (karakterler, eşyalar, nesneler vb.) için soyut temel sınıf olan `CEntity`'yi tanımlar. Bir varlığın temel özelliklerini (tip, pozisyon, sektör ağacı bağlantısı, ağ tanımlayıcısı) ve diğer varlıklarla olan görünürlük ilişkisini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`CEntity` Sınıfı:**
        *   **Saf Sanal Metotlar:** `EncodeInsertPacket(LPENTITY entity)` ve `EncodeRemovePacket(LPENTITY entity)`. Türetilmiş sınıfların, bir varlığın başka bir varlığın görüş alanına nasıl girdiğini (insert) ve çıktığını (remove) belirleyen ağ paketlerini göndermek için bu metotları implemente etmesi zorunludur.
        *   **Tip Yönetimi:** `Initialize(int type)`, `SetType(int type)`, `GetType()`, `IsType(int type)`.
        *   **Yaşam Döngüsü:** `Initialize()`, `Destroy()`. `Destroy` metodu, türetilmiş sınıfın yıkıcısında çağrılmalıdır.
        *   **Pozisyon Yönetimi:** `GetX()`, `GetY()`, `GetZ()`, `GetXYZ()`, `SetXYZ(...)`.
        *   **Sektör Ağacı (Sectree) Yönetimi:** `GetSectree()`, `SetSectree(LPSECTREE tree)`, `UpdateSectree()` (Görünürlüğü günceller).
        *   **Görünürlük Yönetimi:**
            *   `ENTITY_MAP m_map_view`: Görüş alanındaki diğer varlıkları (`LPENTITY`) ve onların "görüş yaşını" (`int`) saklayan harita.
            *   `ViewInsert(LPENTITY entity, bool recursive)`: Bir varlığı görüş haritasına ekler.
            *   `ViewRemove(LPENTITY entity, bool recursive)`: Bir varlığı görüş haritasından çıkarır.
            *   `ViewCleanup(bool recursive)`: Kendi görüş haritasındaki tüm varlıklardan kendini kaldırır ve kendi haritasını temizler.
            *   `ViewReencode()`: Görüş alanındaki tüm varlıklar için `EncodeRemovePacket` ve `EncodeInsertPacket` çağrılarını tetikleyerek görünümü tamamen yeniden gönderir (örn. dönüşüm sonrası).
            *   `GetViewAge()`, `m_iViewAge`: Görünürlük güncelleme döngülerini takip etmek için kullanılan yaş sayacı.
        *   **Ağ İşlemleri:**
            *   `BindDesc(LPDESC _d)`, `GetDesc()`: Varlığı bir ağ bağlantısı tanımlayıcısıyla ilişkilendirir.
            *   `PacketAround(const void* data, int bytes, LPENTITY except)`: Görüş alanındaki (`m_map_view`) varlıklara ve kendine paket gönderir.
            *   `PacketView(const void* data, int bytes, LPENTITY except)`: Sadece görüş alanındaki (`m_map_view`) varlıklara ve kendine paket gönderir (Observer modu dikkate alınır).
        *   **Harita İndeksi:** `SetMapIndex(long l)`, `GetMapIndex()`.
        *   **Gözlemci Modu:** `SetObserverMode(bool bFlag)`, `IsObserverMode()`.\
*   **Bağlantılı Dosyalar:** `entity.cpp`, `entity_view.cpp`. (Dolaylı olarak: `sectree.h`, `desc.h`, `common/tables.h`).

### `entity.cpp`

*   **Amaç:** `entity.h`'de tanımlanan `CEntity` temel sınıfının saf sanal olmayan metotlarını uygular. Başlatma, yok etme, tip yönetimi, paket gönderme yardımcıları ve gözlemci modu değiştirme mantığını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Yapıcı/Yıkıcı:** `Initialize()` çağırır. Yıkıcı, `Destroy()`'ın çağrılıp çağrılmadığını kontrol eder.
    *   **`Initialize()`:** Tüm üye değişkenleri varsayılan değerlere (örn. tip=-1, pos=0, view age=0, harita index=0, observer=false) sıfırlar.
    *   **`Destroy()`:** `ViewCleanup()` çağırarak varlığın diğerlerinin görüş alanından temizlenmesini sağlar ve `m_bIsDestroyed` bayrağını ayarlar. Türetilmiş sınıf yıkıcılarında çağrılmalıdır.
    *   **Tip Yönetimi:** `SetType`, `GetType`, `IsType` için basit implementasyonlar.
    *   **Paket Gönderme Yardımcıları (`FuncPacketAround`, `FuncPacketView`, `PacketAround`, `PacketView`):**
        *   `FuncPacketAround` ve `FuncPacketView` functor'ları, bir varlık listesi (şu anki implementasyonda `m_map_view`) üzerinde gezinerek, `GetDesc()` ile alınan tanımlayıcı üzerinden `Packet()` metodunu çağırır ve belirtilen veriyi gönderir. İsteğe bağlı olarak bir varlığı (`except`) atlayabilir.
        *   `PacketAround` ve `PacketView`, bu functor'ları kullanarak `m_map_view` içindeki varlıklara ve varlığın kendisine (`this`) paket gönderir. `PacketAround` şu anki implementasyonda direkt `PacketView`'i çağırıyor.
        *   Gözlemci modundaki (`m_bIsObserver == true`) varlıklar, `PacketView` ile sadece kendilerine gönderilen paketleri alır, diğerlerine göndermez.
    *   **`SetObserverMode(bool bFlag)`:** İlgili bayrakları (`m_bIsObserver`, `m_bObserverModeChange`) ayarlar ve görünürlük değişikliğini hemen uygulamak için `UpdateSectree()`'yi çağırır. Eğer varlık bir karakterse, `CHAT_TYPE_COMMAND` ile bilgilendirme mesajı gönderir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `desc.h`, `sectree_manager.h`.

### `entity_view.cpp`

*   **Amaç:** `CEntity` sınıfının görünürlük yönetimiyle ilgili metotlarını uygular. Özellikle, varlıkların birbirlerinin görüş alanına nasıl girdiğini (`ViewInsert`), çıktığını (`ViewRemove`), tamamen temizlendiğini (`ViewCleanup`) ve periyodik olarak nasıl güncellendiğini (`UpdateSectree`) belirler. Bu işlemler sırasında ilgili `EncodeInsertPacket`/`EncodeRemovePacket` çağrılarını tetikler.
*   **Temel İşlevler/İçerik:**
    *   **`ViewCleanup(bool recursive)`:** Kendi görüş haritasındaki (`m_map_view`) her varlık (`entity`) için `entity->ViewRemove(this, recursive)` çağırarak, `this` varlığının onların haritasından çıkarılmasını sağlar. Sonra kendi `m_map_view` haritasını temizler.
    *   **`ViewReencode()`:** Mevcut görünümü tamamen yeniden gönderir. Önce `EncodeRemovePacket(this)` ile kendini diğerlerinden siler, sonra `EncodeInsertPacket(this)` ile tekrar ekler. Ardından `m_map_view` içindeki her varlık için `EncodeRemovePacket` ve (eğer gözlemci değilse) `EncodeInsertPacket` çağırır. Ayrıca, eğer karşıdaki varlık gözlemci değilse, `entity->EncodeInsertPacket(this)` çağırarak kendini onlara tekrar gösterir.
    *   **`ViewInsert(LPENTITY entity, bool recursive)`:** `entity`\'yi `m_map_view`\'a ekler. Eğer zaten varsa, sadece `m_iViewAge`\'ini günceller. Eğer yeni ekleniyorsa:
        *   Haritaya ekler (`m_map_view.insert`).
        *   Eğer `entity` gözlemci değilse, `entity->EncodeInsertPacket(this)` çağırarak `this`\'in `entity` tarafından görülmesini sağlar.
        *   Eğer `recursive` true ise, `entity->ViewInsert(this, false)` çağırarak `this`\'in de `entity`\'nin görüş haritasına eklenmesini sağlar.
    *   **`ViewRemove(LPENTITY entity, bool recursive)`:** `entity`\'yi `m_map_view`\'dan siler.
        *   Eğer `entity` gözlemci değilse, `entity->EncodeRemovePacket(this)` çağırarak `this`\'in `entity` tarafından artık görülmemesini sağlar.
        *   Eğer `recursive` true ise, `entity->ViewRemove(this, false)` çağırarak `this`\'in de `entity`\'nin görüş haritasından çıkarılmasını sağlar.
    *   **`CFuncViewInsert` Functor:** `UpdateSectree` içinde `SECTREE::ForEachAround` ile kullanılır.
        *   Çağrıldığı varlık (`ent`) ile `m_me` arasındaki mesafeyi kontrol eder (`DISTANCE_APPROX`).
        *   Eğer mesafe `VIEW_RANGE + VIEW_BONUS_RANGE` içindeyse, `m_me->ViewInsert(ent)` çağırır.
        *   Opsiyonel `#ifdef __REDUCED_ENTITY_VIEW__` derleme bayrağı varsa, NPC\'lerin görüşünü kısıtlayan ek mantık içerir (örn. moblar sadece oyuncuları görür, şifacılar parti üyelerini görür vb.).
        *   Eğer her iki varlık da karakterse ve `ent` bir NPC ise, `ent`\'in state machine\'ini başlatır.
    *   **`UpdateSectree()`:** Periyodik olarak çağrılan ana görünürlük güncelleme fonksiyonu:
        *   `m_iViewAge` sayacını artırır.
        *   `CFuncViewInsert` functor\'ı ve `GetSectree()->ForEachAround()` kullanarak çevredeki varlıkları tarar. Menzil içindekiler için `ViewInsert` çağrılır (bu işlem var olanların yaşını günceller, yenileri ekler).
        *   Gözlemci modu değişikliği (`m_bObserverModeChange`) varsa: Yaşı güncellenmeyenleri `ViewRemove` ile çıkarır. Mod normale döndüyse kalanlara `EncodeInsertPacket` gönderir, gözlemci moduna geçildiyse kalanlara `EncodeRemovePacket` gönderir.
        *   Gözlemci modu değişmediyse ve varlık gözlemci değilse: Yaşı güncellenmeyen (yani menzil dışına çıkan veya artık görülmemesi gereken) varlıkları `ViewRemove` ile haritadan çıkarır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `char.h`, `sectree_manager.h`, `config.h`.

### `event.cpp`

*   **Amaç:** `event.h`'de bildirilen zamanlanmış olay sistemi fonksiyonlarını uygular. Olay kuyruğunu yönetir, olayları oluşturur, iptal eder, işler ve ömürlerini referans sayımıyla yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Global Olay Kuyruğu (`cxx_q`):** Tüm bekleyen olayları zamanlarına göre sıralı tutan statik bir `CEventQueue` nesnesi.
    *   **Nesne Havuzları (`#ifdef M2_USE_POOL`):** `event_info_data` ve `EVENT` nesneleri için statik `MemoryPool` ve `ObjectPool` tanımları.
    *   **`event_create_ex`:** Yeni bir `EVENT` nesnesi oluşturur (havuzdan veya `M2_NEW` ile), verilen işleyici fonksiyonu (`func`) ve bilgi verisini (`info`) atar. Olayı `cxx_q.Enqueue` ile belirtilen süre (`when`, vuruş cinsinden) sonrasına zamanlar ve kuyruk elemanı işaretçisini (`q_el`) saklar.
    *   **`event_cancel`:** İptal edilmek istenen olayı (`*ppevent`) alır. Eğer olay şu anda işleniyorsa (`is_processing`), `is_force_to_end` bayrağını ayarlar ve kuyruk elemanını iptal olarak işaretler. Eğer işlenmiyorsa, sadece kuyruk elemanını iptal olarak işaretler. Başarılı iptal sonrası `*ppevent` işaretçisini `NULL` yapar.
    *   **`event_reset_time`:** Olay işlenmiyorsa, mevcut kuyruk elemanını iptal eder ve olayı yeni `when` zamanıyla tekrar kuyruğa ekler.
    *   **`event_process`:** Ana olay işleme fonksiyonu. `thecore_heart->pulse` ile alınan mevcut sunucu vuruş sayısını kullanarak çalışır:
        1.  `cxx_q.GetTopKey()` ile kuyruktaki en yakın olayın zamanını kontrol eder.
        2.  Zamanı gelmiş olayları (`pulse >= key`) bir döngü içinde işler:
            a.  `cxx_q.Dequeue()` ile olayı kuyruktan çıkarır.
            b.  Eğer olay iptal edilmişse (`pElem->bCancel`), kuyruk elemanını siler ve sonraki olaya geçer.
            c.  Olayın `is_processing` bayrağını `TRUE` yapar.
            d.  Olayın işleyici fonksiyonunu (`the_event->func`) çağırır. Fonksiyona olay işaretçisini (`LPEVENT`) ve işleme süresini (`processing_time`) geçirir.
            e.  İşleyici fonksiyondan dönen değeri (`new_time`) alır.
            f.  Eğer `new_time <= 0` ise veya olay işlenirken iptal edildiyse (`is_force_to_end`), olay sonlanır. `q_el` `NULL` yapılır. `EVENT` nesnesinin yıkıcısı, `info` verisini siler. Referans sayımı sıfıra ulaştığında `EVENT` nesnesi de silinir.
            g.  Eğer `new_time > 0` ise, olay periyodiktir. `cxx_q.Enqueue` ile olay `new_time` kadar vuruş sonrasına tekrar zamanlanır ve `is_processing` `FALSE` yapılır.
    *   **`event_destroy`:** Kuyruktaki tüm olayları temizler.
    *   **Referans Sayımı:** `intrusive_ptr_add_ref` ve `intrusive_ptr_release` fonksiyonları, `LPEVENT` için referans sayımını yönetir. Sayım sıfıra düştüğünde `EVENT` nesnesi silinir (veya havuza iade edilir).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `event_queue.h`, `event.h`. (Dolaylı olarak: `pool.h`, `thecore.h`).

*Buraya `game/src` altındaki çekirdek mekaniklerle ilgili diğer kaynak kod dosyalarının belgeleri eklenecektir.* 

### `FSM.h`

*   **Amaç:** Genel amaçlı bir Sonlu Durum Makinesi (Finite State Machine - FSM) için temel sınıf olan `CFSM`'yi tanımlar. Durum geçişlerini ve güncellemelerini yönetmek için bir çerçeve sağlar. `state.h` içindeki `CState` sınıfını kullanır.
*   **Temel İşlevler/İçerik:**
    *   **`CFSM` Sınıfı:**
        *   Üyeler: `m_pCurrentState`, `m_pNewState`, `m_stateInitial` (başlangıç durumu).
        *   `Update()`: Durum makinesini günceller (durum geçişi yapar ve mevcut durumu çalıştırır).
        *   `IsState()`: Mevcut durumu kontrol eder.
        *   `GotoState()`: Yeni bir duruma geçişi talep eder.
        *   Sanal başlangıç durumu metotları (`Begin/State/EndStateInitial`) özelleştirmeye izin verir.
*   **Bağlantılı Dosyalar:** `FSM.cpp`, `state.h`.

### `FSM.cpp`

*   **Amaç:** `FSM.h`'de bildirilen `CFSM` sınıfının temel metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **Yapıcı (`CFSM::CFSM()`):** Başlangıç durumunu (`m_stateInitial`) ayarlar ve `CStateTemplate` ile metotlara bağlar.
    *   **`CFSM::Update()`:** `GotoState` çağrılmışsa durum geçişini yönetir (eski durumun sonunu, yeni durumun başlangıcını çağırır) ve her zaman mevcut durumun `ExecuteState` metodunu çağırır.
    *   **`CFSM::IsState()`:** Pointer karşılaştırması yapar.
    *   **`CFSM::GotoState()`:** `m_pNewState` değişkenini ayarlar.
*   **Bağlantılı Dosyalar:** `FSM.h`, `<cassert>`, `<cstdlib>`.

### `item_attribute.cpp`

*   **Amaç:** `CItem` sınıfının eşya bonusları (attributes/applies) ile ilgili metotlarını uygular. Normal bonus ekleme, rastgele bonus verme, bonus değiştirme, nadir bonus ekleme/değiştirme ve (eğer tanımlıysa) rastgele temel bonus (`__ITEM_APPLY_RANDOM__`) yönetimi gibi işlevleri içerir.
*   **Temel İşlevler/Metotlar:**
    *   `GetAttributeSetIndex()`: Eşyanın türü ve alt türüne göre hangi bonus setinin (silah, zırh, kostüm vb.) uygulanacağını belirleyen indeksi döndürür.
    *   `HasAttr(wApply)`, `HasApply(wApply)`, `HasRareAttr(wApply)`: Eşyanın belirli bir normal bonusu, prototipte tanımlı sabit bonusu veya nadir (6/7) bonusu içerip içermediğini kontrol eder.
    *   `AddAttribute(wApply, sValue)`, `AddAttr(wApply, wLevel)`: Eşyaya belirli bir türde ve değerde/seviyede normal bonus ekler (eğer boş slot varsa ve bonus zaten mevcut değilse).
    *   `PutAttributeWithLevel(wLevel)`, `PutAttribute(aiAttrPercentTable)`: Belirli bir seviyeye veya olasılık tablosuna göre rastgele bir normal bonus türü seçip eşyaya ekler.
    *   `ChangeAttribute(aiChangeProb)`: Eşyanın mevcut normal bonuslarını temizler ve aynı sayıda yeni rastgele bonus ekler (isteğe bağlı olasılık tablosu ile).
    *   `ChangeAttributeValue()`: Mevcut bonusların türünü koruyarak sadece değerlerini rastgele seviyelere göre günceller.
    *   `ClearAttribute()`, `ClearAllAttribute()`: Eşyanın normal veya tüm (normal + nadir) bonuslarını temizler.
    *   `GetAttributeCount()`: Eşyadaki normal bonus sayısını döndürür.
    *   `FindAttribute(wType)`, `RemoveAttributeAt(index)`, `RemoveAttributeType(wType)`: Belirli bir normal bonusu bulur veya kaldırır.
    *   `SetAttributes(c_pAttribute)`, `SetAttribute(i, wType, sValue)`, `SetForceAttribute(i, wType, sValue)`: Eşyanın bonuslarını doğrudan ayarlar. `SetForceAttribute` tüm slotlara erişim sağlar.
    *   `CopyAttributeTo(pItem)`: Eşyanın bonuslarını başka bir eşyaya kopyalar.
    *   `GetRareAttrCount()`, `ChangeRareAttribute()`, `AddRareAttribute()`: Nadir (6/7. efsun) bonusların sayısını alır, mevcut olanları rastgele yenileriyle değiştirir veya yeni bir tane rastgele ekler.
    *   **`__ITEM_APPLY_RANDOM__` ile ilgili:**
        *   `SetRandomApply(slot, type, value, path)`, `SetForceRandomApply(slot, type, value, path)`, `SetRandomApplies(c_pApplyRandom)`: Rastgele temel bonusları ayarlar.
        *   `GetNextRandomApplies()`: Eşyanın bir sonraki geliştirme seviyesindeki rastgele temel bonus değerlerini hesaplar ve döndürür.
        *   `CopyAppliesRandomTo(pItem)`: Rastgele temel bonusları başka bir eşyaya kopyalar.
*   **Çalışma Prensibi:** Metotlar, `CItem` nesnesinin `m_aAttr` (normal ve nadir bonuslar için) ve `m_aApplyRandom` (rastgele temel bonuslar için) dizileri üzerinde çalışır. Bonus ekleme veya değiştirme işlemleri, genellikle `g_map_itemAttr` (normal), `g_map_itemRare` (nadir) veya `CApplyRandomTable` (rastgele temel) gibi global veri yapılarından alınan bilgilere (bonus türleri, olasılıklar, seviyeye göre değerler) dayanır. Rastgele seçimler için `number()` veya `std::random` kullanılır. Bonuslarda yapılan değişiklikler `UpdatePacket()` ile istemciye bildirilir ve `Save()` ile veritabanına kaydedilmesi için işaretlenir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `log.h`, `item.h`, `char.h`, `desc.h`, `item_manager.h`, `config.h`.

### `item_manager.h`

*   **Amaç:** `ITEM_MANAGER` singleton sınıfını ve eşya düşürme grupları (`CSpecialItemGroup`, `CMobItemGroup` vb.), özel eşya/efsun grupları ve diğer eşya ile ilgili yardımcı sınıfları/yapıları tanımlar. Eşya prototiplerinin yönetimi, yeni eşya oluşturma, eşya bulma/kaydetme/silme, düşürme hesaplamaları ve çeşitli eşya sistemi konfigürasyonlarının (Şans Kutusu, Set Eşyaları, Rastgele Temel Bonuslar vb. - preprocessor direktiflerine bağlı olarak) arayüzünü sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Düşürme Grubu ve Özel Grup Sınıfları (İç İçe veya Yardımcı):**
        *   `CSpecialAttrGroup`: Belirli bir VNUM'a bağlı özel efsun setlerini (`SpecialAttrInfoVector`) ve bir efekt dosya adını tutar.
        *   `CSpecialItemInfo`: Bir özel eşya grubundaki tek bir eşyanın VNUM'unu, sayısını ve nadirlik oranını tutar.
        *   `CSpecialItemGroup`: Genel özel eşya gruplarını yönetir. Türleri (`NORMAL`, `PCT`, `QUEST`, `SPECIAL`) olabilir. Eşyaların (`ItemsVector`) ve birikimli düşme olasılıklarının (`m_vecProbs`) listesini tutar.
        *   `SMobItemGroupInfo`: Yaratık düşürme grubundaki bir eşyanın VNUM'unu, sayısını ve nadirlik oranını tutar.
        *   `CMobItemGroup`: Belirli bir yaratığın (`m_dwMobVnum`) öldürme başına (`m_iKillDrop`) düşürebileceği eşya grubunu (`ItemGroupInfoVector`) ve olasılıklarını (`ProbsVector`) tanımlar.
        *   `SDropItemGroupInfo`, `CDropItemGroup`: Belirli bir yaratık için tanımlanmış daha genel düşürme gruplarını yönetir.
        *   `SLevelItemGroupInfo`, `CLevelItemGroup`: Belirli bir seviye limitine ulaşıldığında aktif olan düşürme gruplarını yönetir.
        *   `SThiefGroupInfo`, `CBuyerThiefGlovesItemGroup`: Hırsız Eldiveni gibi özel eşyalar giyildiğinde aktif olan düşürme gruplarını yönetir.
    *   **`ITEM_MANAGER` Sınıfı (Singleton):**
        *   **Başlatma/Yok Etme:** `Initialize(TItemTable* table, int size)` (eşya prototiplerini `item_proto`'dan yükler), `Destroy()`, `GracefulShutdown()`.
        *   **Eşya ID Yönetimi:** `GetNewID()`, `SetMaxItemID()`, `SetMaxSpareItemID()`. (Uygulamaları `item_manager_idrange.cpp`'dedir).
        *   **Eşya Oluşturma/Yok Etme:** `CreateItem()` (ana eşya oluşturma fonksiyonu), `DestroyItem()` (eşyayı tamamen siler), `RemoveItem()` (karakterden/yerden kaldırıp siler).
        *   **Eşya Bulma:** `Find(id)`, `FindByVID(vid)`, `GetTable(vnum)` (prototipi alır), `GetVnum(name, &vnum)`, `GetVnumByOriginalName(name, &vnum)`.
        *   **Eşya Kaydetme:** `DelayedSave()`, `FlushDelayedSave()`, `SaveSingleItem()`, `Update()` (periyodik kaydetme).
        *   **Düşürme Mekanizmaları:** `GetDropPct()` (temel düşürme yüzdesini hesaplar), `CreateDropItem()` (olasılıklara göre düşecek eşyaları oluşturur ve listeye ekler), `CreateQuestDropItem()` (görev ve olay eşyalarını düşürür).
        *   **Konfigürasyon Dosyası Okuma Metot Bildirimleri:** `ReadCommonDropItemFile`, `ReadEtcDropItemFile`, `ReadSpecialDropItemFile` vb. (Uygulamaları `item_manager_read_tables.cpp`'dedir).
        *   **Diğer Yardımcı Metotlar:** `GetRefineFromVnum()`, `GetSpecialItemGroup()`, `GetSpecialAttrGroup()`, `GetMaskVnum()`, `CopyAllAttrTo()`.
        *   **Sisteme Özel Bölümler (Preprocessor ile):**
            *   `__ITEM_APPLY_RANDOM__`: `CApplyRandomTable* m_pApplyRandomTable` ve ilgili metotlar (`ReadApplyRandomTableFile`, `GetApplyRandom`, `GetApplyRandomValue`).
            *   `__LUCKY_BOX__`: `CLuckyBoxGroup* m_pLuckyBox` ve ilgili metotlar (`ReadLuckyBoxFile`, `GetLuckyBoxGroup`).
            *   `__SET_ITEM__`: Set eşyası veri yapıları (`ItemSetValueMap`, `ItemSetItemMap`) ve getirme metotları (`LoadSetItemTable`, `GetItemSetValueMap`, `GetItemSetItemMap`).
            *   `__EXTENDED_RELOAD__`: Düşürme tablolarını yeniden yükleme metotları (`ReloadMobDropItemGroup`, `ReloadSpecialItemGroup`).
            *   `__SEND_TARGET_INFO__`: `GetMonsterItemDropMap` (bir canavarın olası tüm düşürmelerini listeler), `CreateDropItemVector` (olasılıksız direkt düşürme listesi oluşturur).
        *   **Veri Üyeleri:** `m_vec_prototype` (eşya prototipleri), `m_VIDMap` (VID -> LPITEM), `m_map_pkItemByID` (ID -> LPITEM), çeşitli düşürme grubu/özel grup haritaları (`m_map_pkDropItemGroup`, `m_map_pkSpecialItemGroup` vb.), `m_set_pkItemForDelayedSave` (ertelenmiş kaydetme için set).
    *   **`M2_DESTROY_ITEM(ptr)` Makrosu:** `ITEM_MANAGER::Instance().DestroyItem(ptr)` için bir kısayoldur.
*   **Bağlantılı Dosyalar:** `item_manager.cpp`, `item_manager_read_tables.cpp`, `item_manager_idrange.cpp`, `item.h`, `constants.h` (özellikle `TItemTable` için), `singleton.h`, ve preprocessor direktiflerine bağlı olarak `item_apply_random_table.h` gibi diğer başlıklar.

---

### `item_manager.cpp`

*   **Amaç:** `item_manager.h`'de bildirilen `ITEM_MANAGER` singleton sınıfının temel işlevlerini ve bazı yardımcı fonksiyonları uygular. Eşya prototiplerini yükleme, eşya oluşturma (ID ve VID atama dahil), eşyaları bulma, kaydetme, silme ve karmaşık düşürme (drop) hesaplamalarının ana mantığını içerir.
*   **Temel İşlevler/Metotlar:**
    *   **Yapıcı/Yıkıcı:** Üye değişkenleri başlatır. `Destroy()` metodu, canlı tüm eşyaları ve yüklenmiş özel grupları (eğer varsa ve bellekten ayrılmışlarsa) temizler.
    *   **`Initialize(TItemTable* table, int size)`:** Veritabanından yüklenen ham eşya prototip tablosunu (`item_proto`) alır, `m_vec_prototype`'a kopyalar. Eşya VNUM'larına göre hızlı erişim için `m_map_vid` (VNUM -> TItemTable kopyası) ve geliştirme zincirleri için `m_map_ItemRefineFrom` (gelişmiş VNUM -> ham VNUM) haritalarını doldurur. Görev eşyalarını ve bazı özel eşyaları (`ITEM_COSTUME` tipinde `COSTUME_MOUNT` veya `ITEM_PET` gibi) `CQuestManager`'a kaydeder.
    *   **`CreateItem(vnum, count, id, bTryMagic, iRarePct, bSkipSave)`:** Çekirdek eşya oluşturma fonksiyonu:
        *   Verilen VNUM için prototipi (`TItemTable`) alır. Gerekirse VNUM maskelemesini (`GetMaskVnum`) uygular.
        *   Yeni bir `CItem` nesnesi oluşturur (M2_USE_POOL tanımlıysa havuzdan, değilse `M2_NEW` ile).
        *   Eşyaya benzersiz bir ID atar (`GetNewID()` ile) veya `id` parametresinden gelen mevcut ID'yi kullanır. Yeni ID atanırsa `bIsNewItem` true olur.
        *   Eşyaya benzersiz bir VID (`m_dwVIDCount`) atar.
        *   Oluşturulan eşyayı `m_VIDMap` (VID ile erişim) ve `m_map_pkItemByID` (ID ile erişim, eğer ID varsa ve `bSkipSave` false ise) haritalarına ekler.
        *   Eşyanın sayısını (`count`) ayarlar (yığınlanabilir olup olmamasına ve prototipteki `ITEM_FLAG_MAKECOUNT` bayrağına göre).
        *   Yeni oluşturulan (`id == 0`) ve belirli türdeki eşyalar için özel başlangıç ayarları yapar:
            *   `ITEM_UNIQUE` veya `IsRideItem()`: Kullanım süresi için soket ayarlar ve süre sonu olayını başlatır.
            *   `ITEM_AUTO_HP_RECOVERY_*`, `ITEM_AUTO_SP_RECOVERY_*`: Kullanım süresi/miktarı için soket ayarlar.
            *   Zaman sınırlı eşyalar (`LIMIT_REAL_TIME`, `LIMIT_TIMER_BASED_ON_WEAR`): Süre sonu olaylarını başlatır.
            *   `ITEM_GACHA`: Limit değerini sokete yazar.
            *   `ITEM_SOUL`: Ruh taşı kullanım süresi olayını başlatır.
            *   `ITEM_BLEND`: Karışım değerlerini ayarlar.
            *   `__ITEM_APPLY_RANDOM__` aktifse: Prototipteki `APPLY_RANDOM` türündeki bonuslar için `GetApplyRandom` ile rastgele temel bonusları belirler ve eşyaya ekler.
            *   `__ITEM_VALUE10__` aktifse: Prototipteki `alNewSockets` değerlerine göre rastgele min/max aralığında değerler atar.
            *   Prototipte `sAddonType` varsa `item->ApplyAddon()` çağırır.
            *   `bTryMagic` true ise ve `iRarePct` olasılığı tutarsa, eşyaya rastgele normal bonuslar ekler (`item->AlterToMagicItem()`).
            *   Prototipteki `bGainSocketPct` olasılığı tutarsa eşyaya soket ekler (`item->AlterToSocketItem()`).
            *   Beceri Kitabı (50300) veya Unutma Kitabı (50301/50302) ise, rastgele bir beceri VNUM'unu 0. soketine atar.
            *   Ejderha Taşı ise `DSManager::instance().DragonSoulItemInitialize()` çağırır.
            *   Aura Kostümü ise başlangıç seviye/EXP bilgisini soketine yazar.
            *   Eğer eşya bir `SpecialItemGroup`'a aitse ve bu grup bir `SpecialAttrGroup`'a (`dwAttrVnum`) işaret ediyorsa, o gruptaki sabit efsunları eşyaya `SetForceAttribute` ile ekler.
    *   **Eşya Kaydetme Mekanizması:**
        *   `DelayedSave(item)`: Eşyayı hemen kaydetmek yerine `m_set_pkItemForDelayedSave` setine ekler.
        *   `FlushDelayedSave(item)`: Belirli bir eşyayı ertelenmiş kaydetme setinden çıkarıp hemen `SaveSingleItem` ile kaydeder.
        *   `SaveSingleItem(item)`: Eşyanın verilerini `TPlayerItem` yapısına kopyalar (ID, pencere, pozisyon, VNUM, adet, soketler, efsunlar vb.) ve `HEADER_GD_ITEM_SAVE` paketiyle DB'ye gönderir.
        *   `Update()`: Periyodik olarak çağrılır, `m_set_pkItemForDelayedSave` setindeki eşyaları `SaveSingleItem` ile kaydeder (eğer `ITEM_FLAG_SLOW_QUERY` yoksa).
    *   **Eşya Silme/Kaldırma:**
        *   `RemoveItem(item, reason)`: Bir eşyayı karakterden veya yerden kaldırır (ilgili logları atar) ve ardından `M2_DESTROY_ITEM` (yani `DestroyItem`) ile tamamen yok eder.
        *   `DestroyItem(item)`: Bir eşyayı sistemden tamamen kaldırır. Eğer eşyanın ID'si varsa ve `SkipSave` değilse, DB'ye `HEADER_GD_ITEM_DESTROY` paketi gönderir. Eşyayı `m_VIDMap` ve `m_map_pkItemByID`'den çıkarır ve belleği siler (`M2_DELETE`).
    *   **Eşya Bulma Metotları:** `Find(id)`, `FindByVID(vid)`, `GetTable(vnum)` (prototipi alır, VNUM aralıklarını da destekler), `RealNumber(vnum)` (VNUM'un prototip dizisindeki indeksini bulur), `GetVnum(name, &vnum)` (yerelleştirilmiş isme göre), `GetVnumByOriginalName(name, &vnum)` (orijinal isme göre).
    *   **Düşürme (Drop) Hesaplamaları:**
        *   `GetDropPct(ch, killer, &deltaPercent, &randRange)`: Temel düşürme yüzdesini (`deltaPercent`) ve rastgele sayı aralığını (`randRange`) hesaplar. Oyuncunun seviyesi, mob rütbesi, premium etkileri (Hırsız Eldiveni, Nesne Market Bonusu), karakterin `POINT_ITEM_DROP_BONUS` değeri ve `CPrivManager` (GM bonusları) gibi faktörleri dikkate alır.
        *   `CreateDropItem(ch, killer, vec_item)`: Ana düşürme mantığını içerir. `GetDropPct` ile temel olasılıkları alır. `g_vec_pkCommonDropItem` (genel düşürme), `m_map_pkDropItemGroup` (mob'a özel grup), `m_map_pkMobItemGroup` (mob kill drop), `m_map_pkLevelItemGroup` (seviye limiti), `m_map_pkGloveItemGroup` (hırsız eldiveni), `m_map_dwEtcItemDropProb` (etc item) gibi yüklenmiş tüm düşürme tablolarını ve gruplarını dolaşır. Her bir potansiyel düşürme için, hesaplanan olasılığa göre (`iPercent >= number(1, iRandRange)`) `CreateItem` çağırarak eşyayı oluşturur ve `vec_item` vektörüne ekler. Metin taşları ve bazı özel olay düşürmelerini de (At beceri kitabı vb.) yönetir. Son olarak `CreateQuestDropItem`'ı çağırır.
        *   `CreateQuestDropItem(ch, killer, vec_item, deltaPercent, randRange)`: Aktif görevler ve oyun içi olaylarla (Noel, Ay Festivali, Sevgililer Günü, Cadılar Bayramı vb.) ilgili özel eşyaların düşürülmesini yönetir. Genellikle `quest::CQuestManager::instance().GetEventFlag()` ile aktif olayları kontrol eder ve sabit VNUM'lar veya VNUM listelerinden rastgele seçim yaparak eşya düşürür. Ayrıca "Karakter Taşı" ve "Arındırma Kutusu" gibi özel olay düşürme fonksiyonlarını çağırır.
    *   **Diğer Yardımcı Metotlar:**
        *   `GetRefineFromVnum(vnum)`: Bir eşyanın hangi VNUM'dan geliştirildiğini (ham halini) `m_map_ItemRefineFrom` haritasından bulur.
        *   `GetSpecialItemGroup(vnum)`, `GetSpecialAttrGroup(vnum)`: VNUM ile özel eşya veya özel efsun gruplarını bulur.
        *   `GetMaskVnum(vnum)`: VNUM maskeleme tablosundan (`m_map_new_to_ori`) orijinal VNUM'u bulur.
        *   `CopyAllAttrTo(oldItem, newItem)`: Bir eşyanın tüm efsunlarını, soketlerini ve (`__ITEM_APPLY_RANDOM__`, `__ITEM_VALUE10__`, `__REFINE_ELEMENT_SYSTEM__` aktifse) ilgili diğer özelliklerini başka bir eşyaya kopyalar.
    *   **Özel Etkinlik Düşürme Fonksiyonları:** `DropEvent_CharStone_SetValue`, `__DropEvent_CharStone_GetDropPercent`, `__DropEvent_CharStone_DropItem`, `DropEvent_RefineBox_SetValue`, `__DropEvent_RefineBox_GetDropItem`, `__DropEvent_RefineBox_DropItem` gibi statik fonksiyonlar ve yapılar, belirli oyun içi etkinliklerin düşürme mantığını ve yapılandırmasını yönetir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `item_manager.h`, `utils.h`, `config.h`, `char.h`, `char_manager.h`, `desc_client.h`, `db.h`, `log.h`, `skill.h`, `text_file_loader.h`, `priv_manager.h`, `questmanager.h`, `unique_item.h`, `safebox.h`, `blend_item.h`, `dev_log.h`, `locale_service.h`, `DragonSoul.h` ve preprocessor direktiflerine bağlı olarak (`__CUBE_RENEWAL__` yoksa) `cube.h`.

### `item.h`

*   **Amaç:** `CItem` sınıfının tanımını içerir. Bu sınıf, oyundaki her bir eşyanın (item) temel özelliklerini, durumunu ve davranışlarını yönetmek için bir arayüz ve veri yapısı sağlar.
*   **Temel İşlevler/İçerik:**
    *   `CItem` sınıfı, `CEntity` sınıfından türemiştir.
    *   Eşyanın VNUM'u, ID'si, sahibi (owner), penceresi (window), konumu (cell), sayısı (count), bayrakları (flags), soketleri, efsunları (attributes) gibi temel verilerini tutar.
    *   Eşya oluşturma, yok etme, karakterden kaldırma/ekleme, yere atma/alma, giyme/çıkarma gibi temel işlemleri için metot bildirimleri içerir.
    *   Eşyanın özelliklerini (bonuslar, giyme koşulları vb.) sorgulamak için metotlar sunar.
    *   Eşya ile ilgili olayları (event) yönetmek için (örneğin, yok olma, süre bitimi) LPEVENT üyeleri ve ilgili metotları barındırır.
    *   Preprocessor direktifleri (`#if defined(...)`) ile aktifleşen birçok sisteme (örneğin, `__CHANGE_LOOK_SYSTEM__`, `__SOUL_BIND_SYSTEM__`, `__AURA_COSTUME_SYSTEM__`, `__REFINE_ELEMENT_SYSTEM__`) özel üye değişkenler ve metot bildirimleri içerir.
    *   Eşya türlerini (örn. `IsWeapon()`, `IsArmor()`, `IsCostume()`) ve alt türlerini kontrol etmek için yardımcı metotlar sunar.
    *   Soket yönetimi, efsun yönetimi, nadir efsun yönetimi ve aksesuar soketleri (refine) gibi özelliklerin arayüzünü tanımlar.
*   **Bağlantılı Dosyalar:** `item.cpp` (uygulama), `item_manager.h` (yönetici sınıfı).

---

### `item.cpp`

*   **Amaç:** `item.h` dosyasında bildirilen `CItem` sınıfının metotlarını ve ilgili yardımcı fonksiyonları uygular.
*   **Temel İşlevler/İçerik:**
    *   **Yapıcı (Constructor) ve Yıkıcı (Destructor):** Eşyanın başlangıç değerlerini ayarlar ve kaynakları serbest bırakır.
    *   **`Initialize`, `Destroy`:** Eşyanın başlatılması ve yok edilmesi için temel mantığı içerir. `Destroy`, ilgili tüm event'leri iptal eder.
    *   **Paketleme (`EncodeInsertPacket`, `EncodeRemovePacket`, `UpdatePacket`, `UsePacketEncode`):** Eşyanın istemciye görünmesi, kaybolması, güncellenmesi veya kullanılması durumlarında gönderilecek ağ paketlerini oluşturur.
    *   **Durum Yönetimi (`SetProto`, `SetFlag`, `AddFlag`, `RemoveFlag`, `SetCount`, `SetCell`, `SetWindow`, `SetID`):** Eşyanın temel özelliklerini ayarlar ve günceller. `SetCount(0)` özel bir durumdur ve eşyayı yok edebilir.
    *   **Karakter Etkileşimi (`RemoveFromCharacter`, `AddToCharacter`, `EquipTo`, `Unequip`, `FindEquipCell`):** Eşyanın karakter envanterine eklenmesi, çıkarılması, giyilmesi ve çıkarılması işlemlerini yönetir. `EquipTo` ve `Unequip`, karakterin istatistiklerini (`ModifyPoints`) ve görünümünü (`SetPart`) günceller.
    *   **Yer (Ground) Etkileşimi (`RemoveFromGround`, `AddToGround`):** Eşyanın yerden alınması ve yere bırakılması işlemlerini yönetir, `SECTREE_MANAGER` ile etkileşim kurar.
    *   **Bonus ve Efsun Yönetimi (`ModifyPoints`, `GetSockets`, `SetSocket`, `GetAttributes`, `SetAttributes`, `AddAttribute`, `ChangeAttribute`, `ClearAttribute`, `ApplyAddon`, `IsAccessoryForSocket`, `AccessorySocketDegrade` vb.):** Eşyanın sahip olduğu bonusları (hem prototiptekiler hem de efsunlar) karaktere uygular veya geri alır. Soket ve efsun ekleme, değiştirme, silme işlemlerini gerçekleştirir. Aksesuar geliştirme (refine) sisteminin mantığını içerir.
    *   **Süre ve Olay Yönetimi (`StartDestroyEvent`, `StartUniqueExpireEvent`, `StartTimerBasedOnWearExpireEvent`, `StartRealTimeExpireEvent`, `StartAccessorySocketExpireEvent`, `StopUniqueExpireEvent` vb. ve ilgili event fonksiyonları):** Eşyaların belirli bir süre sonra yok olması, kullanım süresinin dolması veya aksesuar seviyesinin düşmesi gibi olayları yönetir.
    *   **Sistemlere Özel İşlevler:** Preprocessor direktifleriyle ayrılmış çok sayıda sisteme özel mantık içerir. Örnekler:
        *   **`__CHANGE_LOOK_SYSTEM__`:** Eşya görünümü değiştirme (`SetTransmutationVnum`).
        *   **`__ACCE_COSTUME_SYSTEM__`:** Kostüm aksesuarı sistemi, emilim bonusları.
        *   **`__AURA_COSTUME_SYSTEM__`:** Aura kostüm sistemi, aura güçlendirici ve emilim mantığı.
        *   **`__REFINE_ELEMENT_SYSTEM__`:** Elementli güçlendirme sistemi.
        *   **`__SOUL_BIND_SYSTEM__`:** Ruha bağlama sistemi ve mühür açma zamanlayıcısı.
        *   **`__PREMIUM_PRIVATE_SHOP__`:** Özel pazar sistemiyle entegrasyon.
        *   **`__SET_ITEM__`:** Set eşya bonusları.
    *   **Yardımcı Fonksiyonlar (`GetType`, `GetSubType`, `GetVnum`, `GetName`, `GetLocaleName`, `IsStackable`, `IsEquipable`, `CanUsedBy` vb.):** Eşya hakkında çeşitli bilgileri döndüren veya kontroller yapan metotlar.
*   **Çalışma Prensibi:**
    *   Bir `CItem` nesnesi, `ITEM_MANAGER::CreateItem` ile oluşturulur ve bir prototipe (`TItemTable`) bağlanır.
    *   Eşyanın durumu (sahibi, konumu, sayısı, efsunları vb.) üye değişkenlerde tutulur.
    *   Karakterle veya dünyayla etkileşime girdiğinde (giyilme, kullanılma, düşme vb.) ilgili metotları çağrılır.
    *   Bu metotlar, oyun mantığını uygular, karakter istatistiklerini günceller, olayları başlatır/durdurur ve istemciye gerekli paketleri gönderir.
    *   Eşyalar, `ITEM_MANAGER::DelayedSave` ile periyodik olarak veya önemli değişikliklerde veritabanına kaydedilir.
    *   Birçok özellik ve sistem, `#if defined(...)` blokları içinde yer alır, bu da sunucunun farklı özellik setleriyle derlenebilmesini sağlar.
*   **Bağlantılı Dosyalar:** `item.h` (tanım), `item_manager.cpp` (eşya oluşturma/yok etme/kaydetme).

### `locale_service.h`

*   **Amaç:** Oyun sunucusunun yerelleştirme (localization) hizmetleri için gerekli fonksiyon bildirimlerini, enum tanımlarını (`eLocalization`) ve global değişken bildirimlerini içerir. Farklı bölgelere ve dillere uyum sağlamak için kullanılan temel arayüzü tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Fonksiyon Bildirimleri:**
        *   `LocaleService_Init`: Yerelleştirme hizmetini başlatır.
        *   `LocaleService_Load*`: Yerelleştirilmiş çeşitli veri dosyalarını (string, quest, item, mob, skill, quiz, empire text) yüklemek için fonksiyonlar.
        *   `LocaleService_TransferDefaultSetting`: Varsayılan ayarları aktarır.
        *   `LocaleService_GetBasePath`/`GetMapPath`/`GetQuestPath`: İlgili bölgeye özgü dosya yollarını döndürür.
        *   `LC_GetLocalType`: Aktif yerelleştirme türünü döndürür.
        *   `LC_Is*`: Belirli bir yerelleştirmenin aktif olup olmadığını kontrol eden yardımcı fonksiyonlar (örn. `LC_IsJapan`, `LC_IsEurope`, `LC_IsWorldEdition`).
    *   **`eLocalization` Enum'u:** Desteklenen tüm yerelleştirme bölgelerini (örn. `LC_YMIR`, `LC_JAPAN`, `LC_TURKEY`, `LC_EUROPE`) numaralandırır.
*   **Bağlantılı Dosyalar:** `locale_service.cpp` (uygulama), `../../common/service.h`. Bu başlık dosyasını kullanan diğer `game/src` dosyaları.

### `locale_service.cpp`

*   **Amaç:** `locale_service.h`'de bildirilen yerelleştirme fonksiyonlarını uygular. Farklı bölgeler için başlatma mantığını, yerelleştirilmiş dosyaların yüklenmesini ve bölgeye özgü karakter adı/kodlama kontrollerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yerelleştirme Başlatma (`LocaleService_Init`, `__LocaleService_Init_*`, `LC_InitLocalization`):**
        *   `LocaleService_Init`, verilen servis adına göre ilgili `__LocaleService_Init_REGION` fonksiyonunu çağırır.
        *   Her `__LocaleService_Init_*` fonksiyonu, o bölgeye özel global değişkenleri (`g_stLocale`, `g_stServiceBasePath`, `g_stQuestDir`, `g_stServiceMapPath`, `g_setQuestObjectDir`, `g_stLocaleFilename`, `PK_PROTECT_LEVEL`) ve fonksiyon işaretçilerini (`check_name`, `is_twobyte`) ayarlar.
        *   `LC_InitLocalization`, string bölge adını `eLocalization` enum değerine (`g_eLocalType`) çevirir.
        *   `LocaleService_TransferDefaultSetting`, atanmamışsa varsayılan fonksiyon işaretçilerini ve tabloları (örn. `exp_table`) ayarlar.
    *   **Dosya Yükleme (`LocaleService_Load*`):** İlgili fonksiyonlar, belirlenen bölge yolundaki (`g_stServiceBasePath/country/...`) metin dosyalarını okur ve verileri belleğe yükler (bu yükleme işlemleri genellikle başka modüllerdeki fonksiyonlar aracılığıyla yapılır, örn. `LoadLocaleString`, `LoadLocaleQuest`).
    *   **Karakter Adı/Kodlama Kontrolleri (`check_name_*`, `is_twobyte_*`, `check_name_independent`):**
        *   `check_name_REGION`: Bölgeye özgü karakter seti kurallarına (izin verilen karakterler, uzunluk) ve yasaklı kelimelere (`CBanwordManager`) / mob isimlerine (`CMobManager`) göre isim geçerliliğini kontrol eder.
        *   `is_twobyte_REGION`: Bölgeye özgü karakter kodlamasına göre bir karakterin çift baytlı olup olmadığını belirler.
    *   **Yardımcı Fonksiyonlar (`LC_GetLocalType`, `LC_Is*`):** Global `g_eLocalType` değişkenini döndürür veya karşılaştırır.
*   **Çalışma Prensibi:** Sunucu başlatıldığında `LocaleService_Init` çağrılır ve aktif bölgeye göre tüm ayarlar yapılır. Ardından `LocaleService_Load*` fonksiyonları ile metinler ve veriler yüklenir. Oyun sırasında karakter oluşturma gibi işlemlerde `check_name` fonksiyonu, metin işleme sırasında `is_twobyte` fonksiyonu kullanılır. Diğer modüller `LC_Is*` fonksiyonları ile aktif bölgeye göre farklı davranışlar sergileyebilir.
*   **Bağlantılı Dosyalar:** `locale_service.h`, `stdafx.h`, `constants.h`, `banword.h`, `utils.h`, `mob_manager.h`, `empire_text_convert.h`, `config.h`, `skill_power.h` ve yerelleştirilmiş verileri yükleyen diğer modüllerin başlık dosyaları.

### `locale.hpp`

*   **Amaç:** Yerelleştirilmiş metinleri, görev adlarını, eşya adlarını, yaratık adlarını, yetenek adlarını ve quiz metinlerini yüklemek ve bunlara erişmek için kullanılan `extern "C"` fonksiyon bildirimlerini ve bu fonksiyonlar için kısayol makrolarını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Global Değişken Bildirimi:**
        *   `g_iUseLocale` (extern int): Hangi yerelleştirmenin aktif olduğunu belirten bir global değişken (kullanımı `locale.cpp` içinde doğrudan görünmese de, sistemin başka bir yerinde ayarlanabilir veya eski bir yapı olabilir).
    *   **Fonksiyon Bildirimleri:**
        *   `GetLocaleCodeName(BYTE bLocale)`: Verilen `LOCALE_*` byte değerine karşılık gelen kısa yerelleştirme kodunu (örn. "en", "tr", "kr") döndürür.
        *   `LoadLocaleString`, `FindLocaleString`: Genel yerelleştirilmiş metin dosyalarını yükler ve bir anahtar string'e karşılık gelen yerelleştirilmiş metni bulur.
        *   `LoadLocaleQuest`, `FindLocaleQuest`: Görev metinlerini yükler ve VNUM'a göre bulur.
        *   `LoadLocaleItemName`, `FindLocaleItemName`: Eşya isimlerini yükler ve VNUM'a göre bulur.
        *   `LoadLocaleMobName`, `FindLocaleMobName`: Yaratık isimlerini yükler ve VNUM'a göre bulur.
        *   `LoadLocaleSkillName`, `FindLocaleSkillName`: Yetenek isimlerini yükler ve VNUM'a göre bulur.
        *   `LoadLocaleQuiz`, `FindLocaleQuiz`: OX Quiz metinlerini yükler ve VNUM'a göre bulur.
        *   `LocalePetSkillInit`, `LocalePetSkillFind` (eğer `__GROWTH_PET_SYSTEM__` tanımlıysa): Pet yetenek isimlerini yükler ve VNUM'a göre bulur.
    *   **Makro Tanımları:**
        *   `LC_CODE(locale)`: `GetLocaleCodeName` için kısayol.
        *   `LC_TEXT(string)`, `LC_STRING(string, locale)`: `FindLocaleString` için kısayollar.
        *   `LC_QUEST(vnum, locale)`, `LC_ITEM_NAME(vnum, locale)`, `LC_MOB_NAME(vnum, locale)`, `LC_SKILL_NAME(vnum, locale)`, `LC_QUIZ(vnum, locale)`: İlgili `FindLocale*` fonksiyonları için kısayollar.
        *   `LC_LOCALE_PET_SKILL_TEXT(vnum, locale)` (eğer `__GROWTH_PET_SYSTEM__` tanımlıysa): `LocalePetSkillFind` için kısayol.
*   **Bağlantılı Dosyalar:** `locale.cpp` (uygulama), `../../common/length.h`. Oyunun çeşitli yerlerinde yerelleştirilmiş metinlere erişmek için bu başlık dosyasını kullanan diğer dosyalar.

### `locale.cpp`

*   **Amaç:** `locale.hpp`'de bildirilen fonksiyonları uygular. Çeşitli yerelleştirilmiş metin dosyalarını (`locale_string.txt`, `locale_quest.txt`, `item_names.txt`, `mob_names.txt`, `skill_names.txt`, `locale_quiz.txt` ve eğer `__GROWTH_PET_SYSTEM__` tanımlıysa `pet_skill_names.txt`) okur, ayrıştırır ve bu verileri global haritalarda saklar. Bu haritalar üzerinden VNUM veya anahtar string ile arama yaparak ilgili yerelleştirilmiş metni döndürme işlevselliğini sağlar.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Veri Yapıları:**
        *   `gLocaleStringMapType[LOCALE_MAX_NUM]` (std::map<std::string, std::string>): Genel metinler için. Anahtar: orijinal metin, Değer: yerelleştirilmiş metin.
        *   `gLocaleQuestMapType[LOCALE_MAX_NUM]` (std::map<DWORD, std::string>): Görev metinleri için. Anahtar: VNUM, Değer: yerelleştirilmiş metin.
        *   `gLocaleItemNameMapType[LOCALE_MAX_NUM]` (std::map<DWORD, std::string>): Eşya isimleri için.
        *   `gLocaleMobNameMapType[LOCALE_MAX_NUM]` (std::map<DWORD, std::string>): Yaratık isimleri için.
        *   `gLocaleSkillNameMapType[LOCALE_MAX_NUM]` (std::map<DWORD, std::string>): Yetenek isimleri için.
        *   `gLocaleQuizMapType[LOCALE_MAX_NUM]` (std::map<DWORD, std::string>): OX Quiz metinleri için.
        *   `localePetSkill[LOCALE_MAX_NUM]` (std::map<uint32_t, std::string>) (eğer `__GROWTH_PET_SYSTEM__` tanımlıysa): Pet yetenek isimleri için.
        *   `c_stDefaultString`: "NoName" gibi varsayılan bir string, eğer aranan metin bulunamazsa döndürülür.
    *   **Dosya Okuma ve Ayrıştırma (`LoadLocale*` fonksiyonları):**
        *   `LoadLocaleString`: `locale_string.txt` dosyasını okur. Tırnak işaretleri ve kaçış karakterlerini (`\n`, `\"`) işlemek için `FindEndQuote` ve `ConvertLocaleStrings` gibi yardımcı fonksiyonlar kullanır. Satırları anahtar-değer çiftleri olarak ayrıştırır.
        *   `LoadLocaleQuest`, `LoadLocaleQuiz`: Sekme (`\t`) ile ayrılmış, genellikle `VNUM\tMETIN` formatındaki dosyaları `fgets` ve `strtok` kullanarak okur.
        *   `LoadLocaleItemName`, `LoadLocaleMobName`, `LoadLocaleSkillName`, `LocalePetSkillInit`: Sekme (`\t`) ile ayrılmış CSV benzeri dosyaları `cCsvTable` sınıfı aracılığıyla okur. Genellikle ilk sütun VNUM, ikinci sütun isimdir. `CItemVnumHelper::IsDragonSoul` ile Ejderha Taşı Simyası eşyaları için VNUM aralığı kontrolü yapar.
        *   Tüm yükleme fonksiyonları, ilgili global haritayı hedef `bLocale` için temizler ve sonra yeni verilerle doldurur.
    *   **Metin Bulma (`FindLocale*` fonksiyonları):**
        *   Verilen VNUM veya string anahtarını ve `bLocale` (yerelleştirme kodu) parametresini kullanarak ilgili global haritada arama yapar.
        *   Eğer bir eşleşme bulunursa, haritadan alınan yerelleştirilmiş metni döndürür.
        *   Bulunamazsa, `c_stDefaultString`'i (VNUM tabanlı aramalarda) veya orijinal arama string'ini (string tabanlı aramalarda) döndürür.
    *   **`GetLocaleCodeName(BYTE bLocale)`:** Verilen `LOCALE_*` enum değerine karşılık gelen iki harfli ülke kodunu (örn., "tr", "en", "kr") bir `switch` ifadesiyle döndürür.
*   **Çalışma Prensibi:** Sunucu başlangıcında, `locale_service.cpp` içindeki `LocaleService_Load*` fonksiyonları bu dosyada tanımlanan `LoadLocale*` fonksiyonlarını çağırarak, aktif bölgeye ait metin dosyalarını yükler. Yüklenen tüm yerelleştirilmiş metinler, `LOCALE_MAX_NUM` boyutundaki global harita dizilerinde, ilgili `bLocale` indeksine göre saklanır. Oyun sırasında, bir eşyanın, görevin, yaratığın adı veya herhangi bir genel metin gerektiğinde, `locale.hpp`'deki `LC_*` makroları (veya doğrudan `FindLocale*` fonksiyonları) kullanılarak bu haritalardan hızlıca çekilir.
*   **Bağlantılı Dosyalar:** `locale.hpp` (bildirimler), `stdafx.h`, `../../common/length.h`, `../../common/VnumHelper.h`, `CsvReader.h` (CSV dosyalarını okumak için), `locale_service.h` (bu dosyadaki fonksiyonları çağıran sistem).