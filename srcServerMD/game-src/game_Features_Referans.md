# Metin2 Oyun Sunucusu - Özellikler Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) belirli oyun özellikleriyle (Arena, Savaş Haritası, Evlilik vb.) ilgili dosyalarını belgeler.

## İçindekiler

*   [Header Dosyaları (.h)](#header-dosyaları-h)
    *   [`arena.h`](#arenah)
    *   [`auth_brazil.h`](#auth_brazilh)
    *   [`BattleArena.h`](#battlearenah)
    *   [`char_manager.h`](#char_managerh)
*   [Kaynak Kod Dosyaları (.cpp)](#kaynak-kod-dosyaları-cpp)
    *   [`arena.cpp`](#arenacpp)
    *   [`auth_brazil.cpp`](#auth_brazilcpp)
    *   [`BattleArena.cpp`](#battlearenacpp)
    *   [`char_manager.cpp`](#char_managercpp)

---

## Header Dosyaları (.h)

### `arena.h`

*   **Amaç:** PvP (Oyuncuya Karşı Oyuncu) düellolarının yapıldığı Arena sistemini yönetmek için gerekli sınıfları (`CArena`, `CArenaMap`, `CArenaManager`) ve ilgili yapıları (örneğin `MEMBER_IDENTITY` enum'u) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`MEMBER_IDENTITY` Enum'u:** Bir oyuncunun arenadaki rolünü belirtir (Düellocu, Gözlemci, Yok).
    *   **`CArena` Sınıfı:** Tek bir arena örneğini temsil eder.
        *   İki düellocunun PID'lerini (`m_dwPIDA`, `m_dwPIDB`), başlangıç/zaman aşımı olaylarını (`m_pEvent`, `m_pTimeOutEvent`), başlangıç ve gözlemci noktalarını, set/skor sayılarını ve gözlemci haritasını (`m_mapObserver`) tutar.
        *   Düello başlatma/bitirme, üyelik kontrolü, ölüm durumunu işleme (`OnDead`), gözlemci yönetimi gibi metotları içerir.
    *   **`CArenaMap` Sınıfı:** Belirli bir harita (`m_dwMapIndex`) içindeki birden fazla `CArena` örneğini yönetir.
        *   `CArena` nesnelerinin bir listesini (`m_listArena`) tutar.
        *   Haritaya arena ekleme, boş bir arena bulup düello başlatma, haritadaki tüm düelloları bitirme, Lua için düello listesi alma, saldırı geçerliliğini kontrol etme, ölüm durumunu işleme ve gözlemci yönetimi gibi metotlar sağlar.
    *   **`CArenaManager` Sınıfı (Singleton):** Tüm `CArenaMap` nesnelerini yöneten merkezi yöneticidir.
        *   Harita indeksine göre `CArenaMap` işaretçilerini tutan bir harita (`m_mapArenaMap`) içerir.
        *   Düello başlatma, arena tanımlarını ekleme, düello listelerini alma, saldırı geçerliliğini kontrol etme, ölüm durumlarını işleme, gözlemcileri yönetme ve arenalarda kısıtlanmış eşyaları kontrol etme (`IsLimitedItem`) gibi genel arayüzleri sunar.
*   **Bağlantılı Dosyalar:** `arena.cpp`, `char.h`, `event.h`, `singleton.h`, `<map>`, `<list>`, `<lua.h>`.

### `auth_brazil.h`

*   **Amaç:** Brezilya'ya özgü harici bir kimlik doğrulama sunucusuna (muhtemelen OnGame - `auth.ongame.com.br`) yapılan istekler için fonksiyon bildirimlerini ve dönüş kodlarını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Dönüş Kodu Sabitleri (`AUTH_BRAZIL_*`):** Kimlik doğrulama sonucunu belirten sabitler (Başarılı, Hatalı Şifre, ID Yok, Sunucu Hatası, Flash Kullanıcı).
    *   **`auth_brazil(const char* login, const char* pwd)`:** Verilen kullanıcı adı ve şifre ile harici sunucuda kimlik doğrulama işlemini gerçekleştiren ana fonksiyon.
    *   **`auth_brazil_inc_query_count()`:** Yapılan kimlik doğrulama sorgularının sayısını artıran bir sayaç fonksiyonu.
    *   **`auth_brazil_log()`:** Sorgu sayısını bir log dosyasına (`AUTH_COUNT.log`) yazan fonksiyon.
*   **Bağlantılı Dosyalar:** `auth_brazil.cpp`.

### `BattleArena.h`

*   **Amaç:** Belirli haritalarda (190, 191, 192) periyodik olarak gerçekleşen, canavar dalgalarına karşı savunma yapılan "Savaş Arenası" etkinliğini (muhtemelen kale savunması) yönetmek için `CBattleArena` singleton sınıfını ve ilgili sabitleri/enum'ları tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler:** Arena harita indeksleri (`nBATTLE_ARENA_MAP`), regen dosyası yolları (`strRegen`).
    *   **`BATTLEARENA_STATUS` Enum'u:** Arenanın durumunu belirtir (Kapalı, Savaşta, Bitti).
    *   **`CBattleArena` Sınıfı (Singleton):**
        *   Etkinlik zamanlaması (`m_pEvent`), durum (`m_status`), harita/imparatorluk bilgisi (`m_nMapIndex`, `m_nEmpire`) ve zorla bitirme bayrağını (`m_bForceEnd`) tutar.
        *   Etkinliği başlatma (`Start`), bitirme (`End`), zorla bitirme (`ForceEnd`), durum kontrolü (`IsRunning`) ve canavar/taş spawn etme (`SpawnLastBoss`, `SpawnRandomStone`) metotlarını içerir.
*   **Bağlantılı Dosyalar:** `BattleArena.cpp`, `constants.h`, `event.h`, `singleton.h`, `<string>`.

### `belt_inventory_helper.h`

*   **Amaç:** Kemer Envanteri sistemi için statik yardımcı fonksiyonlar sağlayan bir sınıf (`CBeltInventoryHelper`) tanımlar. Kemer seviyesine göre slot kullanılabilirliğini kontrol etme, kemer envanterini temizleme ve bir eşyanın kemer envanterine yerleştirilip yerleştirilemeyeceğini belirleme gibi işlevler sunar.
*   **Temel İşlevler/İçerik:**
    *   `GetBeltGradeByRefineLevel`: Kemerin + basma seviyesini dahili bir "dereceye" dönüştürür.
    *   `GetAvailableRuleTableByGrade`: Hangi kemer derecesinde hangi slotların açılabileceğini tanımlayan bir tablo döndürür.
    *   `IsAvailableCell`: Belirli bir kemer envanteri hücresinin, takılı olan kemerin derecesine göre kullanılabilir olup olmadığını kontrol eder.
    *   `IsExistItemInBeltInventory`, `GetItemCount`: Kemer envanterinde eşya olup olmadığını veya kaç tane olduğunu kontrol eder.
    *   `ClearBelt`: Kemer envanterindeki tüm eşyaları ana envantere taşımaya çalışır (yeterli yer varsa).
    *   `CanMoveIntoBeltInventory`: Bir eşyanın kemer envanterine taşınıp taşınamayacağını kontrol eder (İksirler, özel eşyalar vb.).
*   **Not:** Bu dosya sadece statik metotlar içerir, `.cpp` karşılığı yoktur.
*   **Bağlantılı Dosyalar:** `char.h`, `item.h`.

### `blend_item.h`

*   **Amaç:** "Karışım Eşyaları" (Blend Items - Muhtemelen Şebnemler) ile ilgili fonksiyonları bildirir. Bu eşyaların tanımlarını yükleme, oluşturulduklarında/kullanıldıklarında rastgele değerler atama ve bir eşyanın karışım eşyası olup olmadığını kontrol etme işlevlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   `MAX_BLEND_ITEM_VALUE`: Bir karışım eşyasının sahip olabileceği olası sonuç seviyelerinin sayısını tanımlayan sabit (genellikle 5).
    *   `Blend_Item_init()`: Karışım eşyası sistemini başlatır (verileri yükler).
    *   `Blend_Item_load(char* file)`: Karışım eşyası tanımlarını bir dosyadan (`blend.txt`) yükler.
    *   `Blend_Item_set_value(LPITEM item)`: Verilen bir karışım eşyasına, önceden tanımlanmış olasılıklara göre rastgele soket değerleri (Uygulama Tipi, Değer, Süre) atar.
    *   `Blend_Item_find(DWORD item_vnum)`: Verilen bir eşya VNUM'unun yüklenmiş karışım eşyası tanımlarında mevcut olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `blend_item.cpp`, `item.h`.

### `BlueDragon_Binder.h`

*   **Amaç:** Mavi Ejderha (Nemere) boss'unun beceri parametrelerini ve taş etkilerini Lua script'lerinden (`BlueDragonSetting`) okumak için kullanılan fonksiyonları bildirir.
*   **Temel İşlevler/İçerik:**
    *   **`BLUEDRAGON_STONE_EFFECT` Enum'u:** Savaş alanındaki taşların olası etkilerini (Savunma, Saldırı, Yenilenme vb.) tanımlar.
    *   **`BlueDragon_GetSkillFactor`:** Lua'daki `BlueDragonSetting` tablosundan, değişken sayıda anahtar kullanarak belirli bir beceri parametresinin (örneğin hasar, alan) sayısal değerini alır.
    *   **`BlueDragon_GetRangeFactor`:** Lua tablosundaki aralıklara (min/max) göre, verilen bir değere (örneğin Ejderhanın HP yüzdesi) karşılık gelen bir yüzde değeri (örneğin ek hasar yüzdesi) döndürür.
    *   **`BlueDragon_GetIndexFactor`:** Lua tablosundaki belirli bir index'e (örneğin taş index'i) ve anahtara (örneğin "pct") karşılık gelen değeri alır.
*   **Bağlantılı Dosyalar:** `BlueDragon_Binder.cpp`, `questmanager.h`.

### `BlueDragon_Skill.h`

*   **Amaç:** Mavi Ejderha'nın belirli becerilerinin (Nefes, Zayıf Nefes, Deprem) hedef seçimi ve etkilerini uygulayan functor (fonksiyon nesnesi) struct'larını tanımlar. Bu functor'lar genellikle alan taraması (`SECTOR::for_each`) sırasında hedefler için çağrılır.
*   **Temel İşlevler/İçerik:**
    *   **`FSkillBreath` Struct'ı:** Ana nefes saldırısını uygular. Rastgele hedef sınıf/cinsiyet seçer, Lua'dan alınan parametrelere göre yüzdelik ve temel hasarı hesaplar, hedefe hasar verir ve görsel efekt gönderir.
    *   **`FSkillWeakBreath` Struct'ı:** Daha basit bir nefes saldırısını uygular (sınıf/cinsiyet bonusu olmadan).
    *   **`FSkillEarthQuake` Struct'ı:** Deprem saldırısını uygular. Rastgele sınıf/cinsiyet seçer, Lua'dan alınan parametrelere göre hasar ve sersemletme süresi hesaplar, hedefe hasar verir, sersemletme uygular ve hedefi iter.
*   **Not:** Bu dosya sadece struct tanımları ve uygulama mantığını içerir, `.cpp` karşılığı yoktur.
*   **Bağlantılı Dosyalar:** `BlueDragon_Binder.h`, `char.h`, `affect.h`, `battle.h`, `sectree_manager.h`.

### `BlueDragon.h`

*   **Amaç:** Mavi Ejderha (Nemere) boss'unun ana savaş mantığıyla ilgili fonksiyonları bildirir.
*   **Temel İşlevler/İçerik:**
    *   `BlueDragon_StateBattle(LPCHARACTER)`: Mavi Ejderha'nın savaş durumundaki ana mantığını (beceri seçimi ve kullanımı) yürüten fonksiyon.
    *   `UseBlueDragonSkill(LPCHARACTER, unsigned int)`: Belirtilen indeksteki Mavi Ejderha becerisini kullanan fonksiyon. Etkileri uygular ve bir sonraki kullanım zamanını döndürür.
    *   `BlueDragon_Damage(LPCHARACTER me, LPCHARACTER attacker, int dam)`: Mavi Ejderha veya savaş alanındaki taşlar hasar aldığında/verdiğinde çağrılan, taşların aktifliğine/türüne göre özel hasar modifikasyonlarını uygulayan fonksiyon.
*   **Bağlantılı Dosyalar:** `BlueDragon.cpp`, `char.h`.

### `building.h`

*   **Amaç:** Lonca arazileri üzerine inşa edilebilen yapılar (objeler) ve bu arazilerin yönetimini sağlayan sınıfları (`CObject`, `CLand`, `CManager`) tanımlar. Yapıların verilerini, prototiplerini, arazi bilgilerini ve bunlarla ilişkili NPC'leri yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Namespace `building`:** İlgili tüm sınıfları içerir.
    *   **`CObject` Sınıfı:** Tek bir binayı/yapıyı temsil eder.
        *   Yapının verisini (`TObject`), prototipini (`TObjectProto`), VID'sini, ait olduğu araziyi (`CLand`) ve varsa ilişkili NPC'yi (`LPCHARACTER`) tutar.
        *   Yapıyı oyuna ekleme/kaldırma (`Show`, `Destroy`), istemciye bildirme (`EncodeInsertPacket`/`Remove`), NPC'sini yaratma (`RegenNPC`) ve özel etkilerini uygulama/kaldırma (`ApplySpecialEffect`/`Remove`) metotlarını içerir.
    *   **`CLand` Sınıfı:** Bir lonca arazisini temsil eder.
        *   Arazi verisini (`TLand`), sahibi lonca ID'sini ve üzerindeki objelerin listesini (`map<DWORD, LPOBJECT>`) tutar.
        *   Objeleri ID, VID, VNUM, Grup veya NPC'ye göre bulma (`FindObject*`), ekleme (`InsertObject`), silme (`DeleteObject`), veritabanına oluşturma/silme isteği gönderme (`RequestCreateObject`/`Delete`) metotlarını içerir.
        *   Arazi sahibini değiştirme (`SetOwner`, `RequestUpdate`) ve araziyi temizleme (`ClearLand`) işlevlerini sunar.
        *   Duvar oluşturma/silme (`RequestCreateWall*`/`Delete`) yardımcı metotlarını içerir.
    *   **`CManager` Sınıfı (Singleton):** Tüm arazileri ve objeleri yöneten merkezi sınıftır.
        *   Obje prototiplerini (`map<DWORD, TObjectProto*>`), arazileri (`map<DWORD, CLand*>`) ve objeleri (ID ve VID'ye göre `map<DWORD, LPOBJECT>`) saklar.
        *   Veritabanından gelen prototip (`LoadObjectProto`), arazi (`LoadLand`) ve obje (`LoadObject`) verilerini yükler ve ilgili nesneleri oluşturur/yönetir.
        *   Arazi/obje bulma (`FindLand*`, `FindObjectByVID`), silme (`DeleteObject`), kaydını silme (`UnregisterObject`) ve arazi sahibini güncelleme (`UpdateLand`) gibi global yönetim fonksiyonları sağlar.
        *   Sunucu başlangıcında objeleri gösterme/NPC yaratma (`FinalizeBoot`) ve istemciye arazi listesini gönderme (`SendLandList`) işlevlerini içerir.
        *   Arazi temizleme işlemlerini (`ClearLand*`) başlatır.
*   **Bağlantılı Dosyalar:** `building.cpp`, `constants.h`, `sectree_manager.h`, `item_manager.h`, `char.h`, `guild.h`, `guild_manager.h`, `desc.h`, `packet.h`, `entity.h`, `singleton.h`, `../../common/building.h`, `<vector>`, `<map>`, `<cmath>`.

### `castle.h`

*   **Amaç:** İmparatorluklar arası kale savaşları (Castle Siege) sistemini yönetmek için gerekli sabitleri, enum'ları, yapıları (`CASTLE_DATA`) ve fonksiyon bildirimlerini tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler ve Enum'lar:** Kale haritaları (`CASTLE_MAP`), maksimum NPC sayıları (`MAX_CASTLE_*`), kurbağa bedeli/VNUM'u (`CASTLE_FROG_*`), savaş durumu (`CASTLE_STATE`) gibi değerleri tanımlar.
    *   **`CASTLE_DATA` Struct'ı:** Bir imparatorluğun kalesine ait kurbağa, koruma (bölge/gruba göre), kule karakter işaretçilerini ve savaş/taş spawn event'lerini tutar.
    *   **Fonksiyon Bildirimleri:** Kale verilerini yükleme/kaydetme (`castle_boot`/`save`), savaşı başlatma/bitirme (`castle_siege`/`start`/`end`), NPC spawn etme (`castle_spawn_*`), NPC ölümlerini işleme (`castle_*_die`), NPC sayısını alma (`castle_*_count`), koruma kiralama maliyeti/VNUM kontrolü, saldırı izni kontrolü (`castle_can_attack`), kurbağayı paraya çevirme (`castle_frog_to_empire_money`) gibi işlevleri bildirir.
*   **Bağlantılı Dosyalar:** `castle.cpp`, `constants.h`, `char.h`, `event.h`.

### `changelook.h`

*   **Amaç:** Görünüm Değiştirme (Transmutation / Yansıtma) sistemini yöneten `CChangeLook` sınıfını tanımlar. Bu sınıf, görünüm değiştirme penceresindeki eşyaları, işlemi ve kuralları yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`CChangeLook` Sınıfı:**
        *   Görünüm değiştirme penceresini açar (`Open`).
        *   Eşyaları pencere slotlarına ekler/çıkarır (`ItemCheckIn`/`Out`).
        *   Ücretsiz geçiş biletini yönetir (`FreeItemCheckIn`/`Out`).
        *   İşlemi onaylar (`Accept`): Ücreti alır, görünümü aktarır (`SetTransmutationVnum`), kaynak eşyayı/bileti siler.
        *   Pencereyi temizler (`Clear`).
        *   Eşyaların slotlara konulup konulamayacağını ve birbirleriyle uyumlu olup olmadığını kontrol eden kuralları içerir (`CanAddItem`, `CheckItem`, `CanAddPassItem`).
        *   İşlem ücretini hesaplar (`GetPrice`).
    *   **Enum'lar:** Değiştirme türünü (`EChangeLookType`), slotları (`EChangeLookSlots`) ve ücretleri (`EChangeLookPrice`) tanımlar.
*   **Not:** Kod `#if defined(__CHANGE_LOOK_SYSTEM__)` ile çevrelenmiştir.
*   **Bağlantılı Dosyalar:** `changelook.cpp`, `char.h`, `item.h`, `<array>`.

### `check_server.h`

*   **Amaç:** Sunucuya bağlanan istemcilerin IP adreslerini, önceden tanımlanmış sunucu anahtarlarına (`keys_`) göre doğrulamak için statik bir sınıf (`CheckServer`) tanımlar. Harici bir fonksiyon (`CheckServerKey`) kullanarak IP ve anahtar eşleşmesini kontrol eder. Belirli IP'lerin veya anahtarların sunucuya erişimini kontrol etmek için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`CheckServer` Sınıfı (Statik Üyeler):**
        *   `AddServerKey(const char* serverKey)`: Doğrulama için geçerli bir sunucu anahtarı ekler.
        *   `CheckIp(const char* ip)`: Verilen IP adresini, eklenen tüm anahtarlarla `CheckServerKey` fonksiyonunu kullanarak doğrular. Herhangi bir eşleşme bulunursa başarılı olur.
        *   `IsFail()`: Doğrulama işleminin genel sonucunu (başarılı/başarısız) döndürür.
    *   **`_USE_SERVER_KEY_` Direktifi:** Bu direktif tanımlı değilse, `CheckIp` fonksiyonu kontrolü atlar ve her zaman başarılı olur.
*   **Bağlantılı Dosyalar:** `check_server.cpp`, `CheckServerKey.h` (harici), `<string>`, `<vector>`.

### `cipher.h`

*   **Amaç:** Geliştirilmiş paket şifrelemesi (`__IMPROVED_PACKET_ENCRYPTION__`) aktif olduğunda, istemci ve sunucu arasındaki iletişimi şifrelemek için kullanılan `Cipher` sınıfını bildirir. CryptoPP kütüphanesini kullanarak anahtar anlaşması ve şifreleme/çözme işlemlerini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`Cipher` Sınıfı:**
        *   `Prepare()`: Diffie-Hellman anahtar anlaşması sürecini başlatır ve karşı tarafa gönderilecek genel anahtar verisini hazırlar.
        *   `Activate()`: Karşı taraftan gelen anahtar verisi ile anlaşmayı tamamlar, paylaşılan sırrı oluşturur ve şifreleyici/çözücüyü kurar.
        *   `Encrypt(void* buffer, size_t length)`: Verilen veriyi şifreler.
        *   `Decrypt(void* buffer, size_t length)`: Verilen veriyi çözer.
        *   `activated()`: Şifrelemenin aktif olup olmadığını döndürür.
        *   `IsKeyPrepared()`: Anahtar anlaşması sürecinin başlatılıp başlatılmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `cipher.cpp`, `cryptopp/cryptlib.h`, `KeyAgreement` (ileri bildirim).

### `char_manager.h`

*   **Amaç:** Oyundaki tüm `CHARACTER` nesnelerini yöneten `CHARACTER_MANAGER` singleton sınıfını bildirir. Karakter oluşturma, yok etme, bulma, spawn etme ve genel karakter yönetimi için arayüz sağlar.
*   **Temel İşlevler/İçerik:**
        *   **Singleton Yapısı:** Tek bir örnek olmasını sağlar (`singleton<CHARACTER_MANAGER>`).
        *   **Karakter Yönetimi:** Oluşturma/Yok Etme (`CreateCharacter`, `DestroyCharacter`), Bulma (`Find`, `FindPC`, `FindByPID`, `FindSpecifyPC`, `GetCharactersByRaceNum`), Spawn Etme (`SpawnMob`, `SpawnGroup` vb.) fonksiyonlarını bildirir.
        *   **Güncelleme ve Durum:** Oyun döngüsü güncellemesi (`Update`) ve karakter durum makinesi yönetimi (`AddToStateList`, `RemoveFromStateList`) için metotlar.
        *   **Gecikmeli Kayıt:** Veritabanına yazma işlemlerini optimize etmek için gecikmeli kayıt mekanizması (`DelayedSave`, `FlushDelayedSave`, `ProcessDelayedSave`).
        *   **Global Oranlar:** EXP, Altın, Eşya Düşme gibi global oranları ayarlama/alma (`Set*Rate`, `Get*Rate`) fonksiyonları.
        *   **Yardımcı Yapılar:** Karakterleri saklamak için haritalar (`pkCharMap`, `NAME_MAP`), setler (`CHARACTER_SET`, `pkCharRaceSet`) ve iterator (`CharacterVectorInteractor`) tanımlar.
        *   **Üye Değişkenleri:** Karakter haritaları, durum/kayıt setleri, VID sayacı, global oranlar, seçili taş, bekleyen yok etme listesi gibi üyeleri bildirir.
*   **Bağlantılı Dosyalar:** `singleton.h`, `vid.h`, `stl.h`, `length.h`, `CHARACTER` (ileri bildirim).

### `char.h`

*   **Amaç:** Oyun dünyasındaki tüm dinamik varlıkları (oyuncular, NPC'ler, canavarlar, metin taşları, binekler vb.) temsil eden merkezi `CHARACTER` sınıfını ve bu sınıfla ilişkili temel veri yapılarını, sabitleri (enum'lar) ve fonksiyon prototiplerini tanımlar. Bu dosya, bir karakterin sahip olabileceği tüm özellikleri, durumları ve gerçekleştirebileceği eylemleri tanımlayan arayüzü sağlar.
*   **Temel İçerik:**
    *   **Includes & Forward Declarations:** Gerekli standart kütüphaneleri (`stl.h`, `<boost/unordered_map.hpp>`), temel oyun yapılarını (`entity.h`, `FSM.h`, `constants.h`, `packet.h`), sistem modüllerini (`affect.h`, `horse_rider.h`, `mining.h`, `cube.h`) ve ileriye dönük sınıf bildirimlerini (örn. `CParty`, `CGuild`, `CDungeon`, `CSafebox`, `CShop`, `CItem`, `CAffect`) içerir.
    *   **Önemli Sabitler ve Enum'lar:**
        *   `EPointTypes` (`POINT_*`): Karakterin sahip olduğu tüm puan türlerini (HP, SP, Statüler, Dirençler, Bonuslar, Özel Sistem Puanları vb.) numaralandırır.
        *   `EPKModes` (`PK_MODE_*`): Oyuncunun PK (Player Killer) modlarını tanımlar (Barış, Serbest, Lonca vb.).
        *   `EPositions` (`POS_*`): Karakterin duruş pozisyonlarını tanımlar (Ayakta, Oturuyor, Savașıyor, Ölü vb.).
        *   `DamageFlag`: Hasar türünün ek özelliklerini (Kritik, Delici, Zehir, Blok vb.) belirtmek için kullanılan bit flag'leri.
        *   `EDamageType`: Hasarın ana türünü (Normal, Büyü, Ok, Ateş vb.) belirtir.
        *   `EBlockAction`: Oyuncunun engelleyebileceği etkileşimleri (Ticaret, PM, Grup Daveti vb.) tanımlar.
        *   `MAIN_RACE_*`: Ana karakter sınıflarının numaraları.
        *   `FLY_*`: Uçan nesne/efekt türleri (EXP, HP/SP Küresi, Beceri Efekti).
        *   Çeşitli sabitler: Zehir süresi, binek türleri (`eMountType`), beceri grupları, safebox sayfa boyutu, yapay zeka flag'leri (`AI_FLAG_*`), anlık durum flag'leri (`INSTANT_FLAG_*`).
    *   **Önemli Yapılar (Structs):**
        *   `character_point`: Karakterin kalıcı (veritabanına kaydedilen) temel puanlarını ve bilgilerini (Seviye, EXP, HP/SP, Statüler, Altın, Becer Grubu vb.) tutan yapı.
        *   `character_point_instant`: Karakterin anlık, geçici veya hesaplanmış puanlarını ve durumlarını (Maks HP/SP, Hızlar, Savunma/Saldırı Değerleri, Anlık Flag'ler, Kuşanılan Parçalar, Bağlı NPC/Sistem işaretçileri vb.) tutan yapı.
        *   `DynamicCharacterPtr`: Bir karakter işaretçisini (LPCHARACTER) ID ve PC/NPC bilgisiyle birlikte güvenli bir şekilde saklamak ve daha sonra bulmak için kullanılan yardımcı yapı (Özellikle event bilgileri içinde kullanılır).
        *   `TSkillUseInfo`: Bir becerinin kullanım bilgilerini (Cooldown, Vuruş Sayısı, Hedef VID vb.) takip eden yapı.
        *   `AffectContainerList`, `AffectStackMap`: Affect (etki) listesini ve yığınlanabilir affect sayaçlarını tutan veri yapıları.
        *   `MobSkillEventMap`: Canavar becerilerinin gecikmeli vuruş eventlerini saklayan harita.
        *   `TBattleInfo`, `TDamageMap`: Savaş sırasında hedeflere verilen hasarı ve aggro'yu takip etmek için kullanılan yapılar.
        *   `Trigger`: Belirli olaylarda (örn. tıklama) tetiklenecek fonksiyonları tutan yapı.
    *   **`CHARACTER` Sınıfı Bildirimi:**
        *   **Miras Alma:** `CEntity`, `CFSM`, `CHorseRider` sınıflarından türetilmiştir.
        *   **Sanal Fonksiyonlar (Override):** `CEntity`'den gelen `EncodeInsertPacket` ve `EncodeRemovePacket` fonksiyonlarını override eder.
        *   **Durum Makinesi (`CStateTemplate`, `State*` fonksiyonları):** Karakterin farklı durumları (`Idle`, `Move`, `Battle`, `Horse`, `Flag` vb.) için durum nesnelerini ve bu durumların mantığını çalıştıran fonksiyon bildirimlerini içerir (`char_state.cpp`'de uygulanır).
        *   **Üye Değişkenleri (Önemliler):**
            *   Temel Bilgiler: `m_dwPlayerID`, `m_vid`, `m_stName`, `m_points` (kalıcı), `m_pointsInstant` (anlık), `m_bCharType`, `m_pkMobData`, `m_pkMobInst`.
            *   Konum/Hareket: `m_posStart`, `m_posDest`, `m_posWarp`, `m_posRegen`, `m_posExit`, `m_bWalking`, `m_bNowWalking`, `m_dwMoveStartTime`, `m_dwMoveDuration`, `m_pkChrSyncOwner`.
            *   Puanlar/Statüler: (Doğrudan `m_points` ve `m_pointsInstant` içinde).
            *   Eşyalar: `m_PlayerSlots` (unique_ptr veya pointer), `m_pSkillLevels`, `m_quickslot`.
            *   Affectler: `m_list_pkAffect`, `m_afAffectFlag`, `m_map_affectStack`.
            *   Sistem İşaretçileri: `m_pkParty`, `m_pGuild`, `m_pkDungeon`, `m_pWarMap`, `m_pkExchange`, `m_pkShop`, `m_pkMyShop`, `m_pPrivateShop`, `m_pkSafebox`, `m_pkMall`, `m_pkChrMarried`, `m_pWeddingMap`, `m_chHorse`, `m_chRider`, `m_petSystem`, `m_GrowthPetSystem`, `m_pkChangeLook`, `m_pkMailBox`.
            *   Savaş/Hedef: `m_kVIDVictim`, `m_pkChrTarget`, `m_map_kDamage`, `m_iAlignment`, `m_bPKMode`, `m_dwKillerPID`.
            *   Beceriler: `m_SkillUseInfo`, `m_adwMobSkillCooltime`.
            *   Eventler: `m_pkDeadEvent`, `m_pkRecoveryEvent`, `m_pkSaveEvent`, `m_pkFishingEvent`, `m_pkMiningEvent` gibi birçok `LPEVENT` işaretçisi.
            *   Diğer: `m_bItemLoaded`, `m_bIsLoadedAffect`, `m_bBlockMode`, `m_dwPolymorphRace`, `m_dwMountVnum`, `m_dwQuestNPCVID`, `m_aiPremiumTimes`.
        *   **Üye Fonksiyon Bildirimleri (Kategorize Edilmiş - Ana Başlıklar):**
            *   Yapıcı/Yıkıcı/Oluşturma (`CHARACTER`, `~CHARACTER`, `Create`, `Destroy`).
            *   Temel Bilgi Ayarlama/Alma (`Set*`/`Get*` - Name, Race, Job, Level, Empire, GMLevel vb.).
            *   Puan Yönetimi (`GetPoint`, `SetPoint`, `PointChange`, `ComputePoints`, `ApplyPoint`, `UpdatePointsPacket`).
            *   Konum/Hareket/Görünürlük (`Show`, `Hide`, `Goto`, `Move`, `WarpSet`, `SetRotation`, `SendMovePacket`, `EncodeInsertPacket`, `EncodeRemovePacket`, `UpdateSectree`).
            *   Eşya Yönetimi (`SetItem`, `GetItem`, `IsEmptyItemGrid`, `UseItem`, `EquipItem`, `UnequipItem`, `DropItem`, `PickupItem`, `GiveItem`, `MoveItem`, `SwapItem`, `RefineItem`, `AutoGiveItem`, `FindSpecifyItem`, `RemoveSpecifyItem` vb.).
            *   Beceri Yönetimi (`SkillLevelUp`, `SkillLevelDown`, `UseSkill`, `ComputeSkill`, `GetSkillLevel`, `SetSkillGroup`, `LearnSkillByBook` vb.).
            *   Affect Yönetimi (`AddAffect`, `RemoveAffect`, `FindAffect`, `IsAffectFlag`, `ComputeAffect`, `RefreshAffect`).
            *   Savaş Mekanikleri (`Damage`, `Attack`, `Dead`, `Revive`, `Stun`, `Shoot`, `FlyTarget`, `SetVictim`, `GetVictim`, `UpdateAggrPoint`, `SetPKMode`).
            *   Durum Makinesi (`GotoState`, `UpdateStateMachine`, `StateIdle`, `StateMove`, `StateBattle` bildirimleri).
            *   Sosyal Sistemler (`SetParty`, `RequestToParty`, `SetGuild`, `SetMarryPartner`).
            *   Özel Sistemler (`SetPolymorph`, `MountVnum`, `UnMount`, `SetShop`, `OpenMyShop`, `SetRefineNPC`, `LoadSafebox`, `LoadMall`, `OpenAcce`, `Set*CostumeHidden`, `mining`, `fishing`).
            *   Paket Gönderme (`ChatPacket`, `EffectPacket`, `UpdatePacket`, `PointsPacket`, `SkillLevelPacket`).
            *   Event Yönetimi (`Start*Event`, `Stop*Event` fonksiyonları).
            *   Yardımcı/Kontrol Fonksiyonları (`IsPC`, `IsNPC`, `IsMonster`, `IsDead`, `CanMove`, `CanFight`, `IsImmune`, `IsHack`, `CanWarp`, `IsInSafezone`).
    *   **Yardımcı Fonksiyonlar/Global'ler:** `GET_SEX`, `GetMountLevelByVnum`, `GetRandomSkillVnum`, `CAN_ENTER_ZONE`, `IS_MOUNTABLE_ZONE` gibi sınıf dışı yardımcı fonksiyon bildirimleri.
*   **Bağlantılı Dosyalar:** `char.cpp` (ana uygulama dosyası), `stdafx.h`, ve `char.h`'yi include eden neredeyse tüm diğer `game/src` dosyaları.

---

## Kaynak Kod Dosyaları (.cpp)

### `arena.cpp`

*   **Amaç:** `arena.h`'de bildirilen Arena sistemi sınıflarının ve fonksiyonlarının implementasyonunu sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`CArena` Uygulaması:**
        *   Yapıcı: Başlangıç ve gözlemci noktalarını ayarlar.
        *   `StartDuel`: Oyuncuların PID'lerini ve set sayısını kaydeder, oyuncuları arena başlangıç noktalarına ışınlar, düello başlangıç geri sayımı (`ready_to_start_event`) ve zaman aşımı (`duel_time_out`) için event'ler oluşturur, oyuncuların HP/SP'sini doldurur.
        *   `EndDuel`: Aktif event'leri iptal eder, oyuncuları (varsa) normal PK moduna döndürür, HP/SP'lerini doldurur, `SetArena(nullptr)` ile arena bağlantısını keser ve oyuncuları/gözlemcileri başlangıç noktalarına ışınlar. `Clear` ile arena durumunu sıfırlar.
        *   `OnDead`: Ölen oyuncuya göre kazanan tarafın skorunu (`m_dwSetPointOfA` veya `m_dwSetPointOfB`) artırır. Set sayısı tamamlandıysa düelloyu bitirmek için event tetikler (state 2), tamamlanmadıysa yeni raundu başlatmak için event tetikler (state 3). Oyunculara ve gözlemcilere bilgilendirme mesajları gönderir.
        *   `OnDisconnect`: Düelloculardan biri oyundan düşerse, diğer oyuncuya bilgi verip düelloyu sonlandırır.
        *   Gözlemci Yönetimi (`AddObserver`, `RegisterObserverPtr`, `RemoveObserver`, `SendPacketToObserver`, `SendChatPacketToObserver`): Gözlemcileri ekler, çıkarır, doğrular ve onlara paket/mesaj gönderir.
    *   **`CArenaMap` Uygulaması:**
        *   `AddArena`: Yeni bir `CArena` nesnesi oluşturur ve listeye ekler (koordinat çakışması yoksa).
        *   `StartDuel`: Listesindeki boş bir `CArena` bulup onun `StartDuel` metodunu çağırır.
        *   Diğer Metotlar (`EndAllDuel`, `EndDuel`, `GetDuelList`, `CanAttack`, `OnDead`, `AddObserver`, `RegisterObserverPtr`, `IsMember`): Genellikle listedeki uygun `CArena` örneğini bulup ilgili metodu çağırmaya dayanır.
    *   **`CArenaManager` Uygulaması:**
        *   `AddArena`: İlgili `CArenaMap`'i bulur (yoksa oluşturur) ve onun `AddArena` metodunu çağırır.
        *   `StartDuel`: Tüm `CArenaMap`'leri dolaşarak düelloyu başlatabilecek birini bulur.
        *   `IsLimitedItem`: Haritanın arena haritası olup olmadığını kontrol eder ve öyleyse, verilen eşya VNUM'unun kısıtlı (yasaklı) eşyalar listesinde olup olmadığına bakar.
        *   Diğer Metotlar: Genellikle ilgili `CArenaMap`'i bulup metodu ona devreder.
    *   **Event Fonksiyonları (`ready_to_start_event`, `duel_time_out`):**
        *   `ready_to_start_event`: Düello başlangıç geri sayımını yönetir, `HEADER_GC_DUEL_START` paketini gönderir, rauntları yeniden başlatır veya düelloyu bitirir.
        *   `duel_time_out`: Süre bittiğinde düelloyu sonlandırır ve oyuncuları ışınlamak için event tetikler.
*   **Bağlantılı Dosyalar:** `arena.h`, `stdafx.h`, `constants.h`, `config.h`, `packet.h`, `desc.h`, `buffer_manager.h`, `start_position.h`, `questmanager.h`, `char.h`, `char_manager.h`, `<memory>`.

### `auth_brazil.cpp`

*   **Amaç:** `auth_brazil.h`'de bildirilen fonksiyonları uygular. Belirtilen kullanıcı adı ve şifre ile Brezilya kimlik doğrulama sunucusuna (`auth.ongame.com.br`) bir HTTP GET isteği gönderir ve yanıtı ayrıştırarak sonucu döndürür.
*   **Temel İşlevler/İçerik:**
    *   **`FN_md5(const char* src)`:** Verilen metnin MD5 hash'ini hesaplayan yardımcı fonksiyon (Şifreyi göndermeden önce hashlemek için).
    *   **`FN_make_request(...)`:** Belirtilen kullanıcı adı ve MD5 hash'lenmiş şifre ile `auth.ongame.com.br` adresine gönderilecek HTTP GET isteği metnini oluşturur.
    *   **`FN_parse_reply(char* reply)`:** Kimlik doğrulama sunucusundan gelen HTTP yanıtının son satırını (genellikle "true", "false", "unknown" veya "flash" gibi bir metin) ayrıştırır ve ilgili `AUTH_BRAZIL_*` sabitini döndürür.
    *   **`auth_brazil(const char* login, const char* pwd)`:**
        1.  `auth.ongame.com.br` sunucusuna TCP soket bağlantısı kurar.
        2.  Soket için zaman aşımı ayarlar.
        3.  HTTP GET isteğini gönderir.
        4.  Sunucudan gelen yanıtı okur.
        5.  Yanıtı ayrıştırır ve sonucu (`AUTH_BRAZIL_*` sabiti) döndürür.
    *   **Sayaç ve Loglama:** Yapılan sorgu sayısını sayar ve belirli aralıklarla `AUTH_COUNT.log` dosyasına yazar.
*   **Çalışma Prensibi:** Oyuncu giriş yaparken (muhtemelen login sunucusunda veya oyun sunucusunun erken aşamalarında), bu fonksiyon çağrılarak kullanıcının kimlik bilgileri harici bir servis üzerinden doğrulanır. Bu, oyunun kendi veritabanı yerine merkezi bir kimlik doğrulama sistemi kullanmasını sağlar.
*   **Bağlantılı Dosyalar:** `auth_brazil.h`, `stdafx.h`, MD5 kütüphanesi (`md5.h` veya `xmd5.h`), socket fonksiyonları (muhtemelen `libthecore`'dan), `<stdio.h>`, `<string.h>`, (Unix-like) `<unistd.h>`, `<stdint.h>`.

### `BattleArena.cpp`

*   **Amaç:** `BattleArena.h`'de bildirilen `CBattleArena` sınıfının fonksiyonlarını uygular. Savaş Arenası etkinliğinin zamanlamasını, durum yönetimini, canavar/taş spawn mekanizmalarını ve oyuncu ışınlamalarını yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`FWarpToHome` Functor'ı:** Etkinlik bittiğinde haritadaki oyuncuları imparatorluk başlangıç noktalarına ışınlar.
    *   **`battle_arena_event` Event Fonksiyonu:** Etkinliğin farklı aşamalarını (duyurular, canavar spawn, canavar kontrolü, boss spawn, bitiş ve ışınlama) zamanlanmış adımlar halinde yöneten ana mantığı içerir. Quest flag'lerini (`battle_arena`) ve `SECTREE_MANAGER` aracılığıyla canavar/taş yönetimini kontrol eder.
    *   **`Start`:** Etkinliği başlatır, duyuruları yapar ve ana zamanlama event'ini (`battle_arena_event`) oluşturur.
    *   **`End`:** Event'i iptal eder, durumu ve quest flag'ini sıfırlar.
    *   **`ForceEnd`:** Etkinliği erken bitiş aşamasına sokan bir event tetikler.
    *   **`SpawnRandomStone`, `SpawnLastBoss`:** Belirlenen VNUM'lar ve haritaya özel koordinatlarla Metin taşlarını veya Boss'u spawn eder.
*   **Çalışma Prensibi:** Bu sistem, genellikle GM komutuyla veya otomatik bir zamanlayıcıyla tetiklenir (`Start`). Ardından `battle_arena_event` belirli aralıklarla çalışarak duyuruları yapar, canavarları/taşları spawn eder, haritadaki durumu kontrol eder ve etkinlik bittiğinde oyuncuları ışınlayıp arenayı sıfırlar (`End`).
*   **Bağlantılı Dosyalar:** `BattleArena.h`, `stdafx.h`, `constants.h`, `start_position.h`, `char_manager.h`, `char.h`, `sectree_manager.h`, `regen.h`, `questmanager.h`, `locale_service.h`.

### `blend_item.cpp`

*   **Amaç:** `blend_item.h`'de bildirilen fonksiyonları uygular. `blend.txt` dosyasından verileri yükler, karışım eşyası sonuçları için olasılık dağılımını tanımlar ve bu rastgele değerleri eşya soketlerine uygular.
*   **Temel İşlevler/İçerik:**
    *   **`BLEND_ITEM_INFO` Struct'ı:** Bir karışım eşyasının tanımını (VNUM, Uygulama Tipi, Olası Değerler/Süreler) tutar.
    *   **`s_blend_info` (vector):** Yüklenen karışım eşyası tanımlarını saklar.
    *   **`Blend_Item_load`:** `blend.txt` dosyasını ayrıştırarak karışım eşyası tanımlarını okur ve `s_blend_info` vektörüne ekler.
    *   **`FN_random_index`, `FN_ECS_random_index`:** Karışım eşyasının sonucunu (kaçıncı seviye bonus vereceğini) belirlemek için ağırlıklı rastgele seçim yapan fonksiyonlar.
    *   **`Blend_Item_set_value`:** Eşya oluşturulduğunda veya belirli bir olayda çağrılarak, `blend.txt`'deki tanıma ve rastgele seçim fonksiyonlarına göre eşyanın soketlerine (Tip, Değer, Süre) uygun bonus değerlerini atar.
    *   **`Blend_Item_find`:** Bir VNUM'un karışım eşyası olup olmadığını kontrol eder.
*   **Çalışma Prensibi:** Sunucu başlarken `Blend_Item_init` ile `blend.txt` okunur. Oyuncu bir karışım eşyası elde ettiğinde (örneğin ürettiğinde) `Blend_Item_set_value` çağrılarak eşyaya rastgele bir bonus seviyesi atanır ve bu bilgi eşyanın soketlerine kaydedilir.
*   **Bağlantılı Dosyalar:** `blend_item.h`, `stdafx.h`, `constants.h`, `log.h`, `dev_log.h`, `locale_service.h`, `item.h`, `char.h`.

### `BlueDragon_Binder.cpp`

*   **Amaç:** `BlueDragon_Binder.h`'de bildirilen Lua parametre okuma fonksiyonlarını uygular. `quest::CQuestManager` üzerinden Lua state'ine erişir ve `BlueDragonSetting` global tablosundan gerekli değerleri alır.
*   **Temel İşlevler/İçerik:**
    *   **`BlueDragon_GetSkillFactor`:** Lua state'ini kullanarak `BlueDragonSetting` tablosunda iç içe geçmiş anahtarları takip eder ve son sayısal değeri döndürür.
    *   **`BlueDragon_GetRangeFactor`:** Lua tablosundaki bir diziyi gezer, verilen değerin hangi min/max aralığına düştüğünü bulur ve o aralığa karşılık gelen yüzde değerini döndürür.
    *   **`BlueDragon_GetIndexFactor`:** Lua tablosundaki belirli bir index ve anahtara karşılık gelen sayısal değeri okur.
*   **Çalışma Prensibi:** Mavi Ejderha becerileri tetiklendiğinde (`BlueDragon_Skill.h` içindeki functor'lar), bu fonksiyonlar çağrılarak becerinin o anki duruma (Ejderhanın HP'si, hedefin sınıfı vb.) göre sahip olacağı hasar, süre, alan gibi parametreler dinamik olarak Lua scriptinden okunur.
*   **Bağlantılı Dosyalar:** `BlueDragon_Binder.h`, `stdafx.h`, `questmanager.h`.

### `BlueDragon.cpp`

*   **Amaç:** `BlueDragon.h`'de bildirilen fonksiyonları uygular. Mavi Ejderha'nın HP yüzdesine göre beceri kullanım sırasını belirler, Lua'dan alınan parametrelerle becerileri tetikler ve Ejderha/taşlar arasındaki hasar etkileşimlerini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`UseBlueDragonSkill`:** Verilen beceri indeksine göre ilgili functor'ı (`BlueDragon_Skill.h`) oluşturur, haritadaki hedeflere uygular, Lua'dan alınan bekleme süresini Ejderhanın HP'sine göre ayarlar ve bir sonraki kullanım zamanını döndürür. Deprem becerisi sonrası en uzak hedefi dövüşe alır.
    *   **`BlueDragon_StateBattle`:** Ejderhanın HP yüzdesine göre beceri öncelik sırasını belirler. Bekleme süresi dolan ilk öncelikli beceriyi `UseBlueDragonSkill` ile kullanır, bir sonraki kullanım zamanını ayarlar ve hareket paketini gönderir.
    *   **`BlueDragon_Damage`:** Mavi Ejderha veya etrafındaki taşlar hasar aldığında/verdiğinde çağrılır. Aktif taşların türüne (Saldırı/Savunma Bonusu) ve sayısına göre veya saldıran binek türüne göre Lua'dan alınan değerlerle hasarı modifiye eder.
*   **Çalışma Prensibi:** Mavi Ejderha karakteri (`CHARACTER` nesnesi) savaş durumundayken, muhtemelen kendi event döngüsü içinde düzenli olarak `BlueDragon_StateBattle` fonksiyonunu çağırır. Bu fonksiyon, Ejderhanın durumuna göre hangi beceriyi kullanacağına karar verir ve `UseBlueDragonSkill` aracılığıyla beceriyi tetikler. Beceri etkileri (`BlueDragon_Skill.h`) ve hasar modifikasyonları (`BlueDragon_Damage`), dinamik olarak `BlueDragon_Binder` aracılığıyla Lua scriptinden okunan parametrelere göre çalışır. Bu, boss mekaniklerinin kod değişikliği yapmadan Lua üzerinden ayarlanabilmesini sağlar.
*   **Bağlantılı Dosyalar:** `BlueDragon.h`, `stdafx.h`, `char.h`, `mob_manager.h`, `sectree_manager.h`, `battle.h`, `affect.h`, `BlueDragon_Binder.h`, `BlueDragon_Skill.h`, `packet.h`, `motion.h`.

### `building.cpp`

*   **Amaç:** `building.h`'de bildirilen bina/arazi sistemi sınıflarının (`CObject`, `CLand`, `CManager`) metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **`CObject` Uygulaması:** Yapıların yok edilmesi, istemciye bildirilmesi, gösterilmesi, NPC'lerinin yaratılması ve özel etkilerinin (örn. lonca üye bonusu) uygulanması/kaldırılması mantığını içerir.
    *   **`CLand` Uygulaması:** Arazi üzerindeki objelerin yönetimi (ekleme, bulma, silme), veritabanı ile iletişim (oluşturma, silme, güncelleme istekleri), arazi temizleme ve duvar örme/silme işlemlerinin mantığını içerir. Obje yerleştirirken çakışma kontrolleri yapar.
    *   **`CManager` Uygulaması:** Prototip, arazi ve obje verilerinin veritabanından yüklenmesi, ilgili nesnelerin oluşturulması ve haritalarda saklanması, global arama/yönetim fonksiyonları, sunucu başlangıç işlemleri (`FinalizeBoot`) ve istemciye veri gönderme (`SendLandList`, `UpdateLand`) mantığını içerir.
*   **Çalışma Prensibi:** Sunucu başlarken `CManager`, veritabanından tüm arazi, obje prototipi ve mevcut obje bilgilerini yükler (`Load*`). `FinalizeBoot` ile objeler oyuna dahil edilir. Oyuncular bir haritaya girdiğinde `SendLandList` ile arazi bilgileri gönderilir. Loncalar arazi satın aldığında/kaybettiğinde `UpdateLand` çağrılır. Oyuncular veya sistem (örn. görevler) `RequestCreateObject`/`DeleteObject` ile obje ekleme/kaldırma isteğinde bulunur, bu istekler veritabanına iletilir ve başarılı olursa `LoadObject`/`DeleteObject` ile oyun dünyası güncellenir.
*   **Bağlantılı Dosyalar:** `building.h`, `stdafx.h`, `constants.h`, `sectree_manager.h`, `item_manager.h`, `buffer_manager.h`, `config.h`, `packet.h`, `char.h`, `char_manager.h`, `guild.h`, `guild_manager.h`, `desc.h`, `desc_manager.h`, `desc_client.h`, `questmanager.h`, `<cmath>`.

### `castle.cpp`

*   **Amaç:** `castle.h`'de bildirilen kale savaşı fonksiyonlarını uygular. Kale verilerini yükler/kaydeder, savaş durumunu yönetir, ilgili event'leri (savaş süresi, taş spawn) oluşturur/iptal eder, NPC'leri (kurbağa, koruma, kule) spawn eder/yok eder ve savaş kurallarını (saldırı izni, ölüm sonuçları) uygular.
*   **Temel İşlevler/İçerik:**
    *   **Veri Yönetimi (`s_castle`, `castle_boot`/`save`):** Her imparatorluk için kale verilerini (aktif kurbağalar, korumalar) `castle_data.txt` dosyasından yükler ve periyodik olarak kaydeder.
    *   **Durum Yönetimi (`s_siege_state`, `s_sige_empire`, `castle_siege`/`start`/`end`):** Mevcut savaşın hangi imparatorlukta olduğunu ve hangi aşamada (`NONE`, `STRUGGLE`, `END`) olduğunu takip eder. Savaş başlatıldığında kuleleri spawn eder ve event'leri başlatır, bittiğinde event'leri iptal eder ve kuleleri yok eder.
    *   **Event İşleyicileri (`castle_siege_event`, `castle_stone_event`):**
        *   `castle_siege_event`: Savaşın süresini takip eder, duyurular yapar ve süre dolduğunda savaşı sonlandırır.
        *   `castle_stone_event`: Savaş sırasında periyodik olarak rastgele Metin taşları spawn eder.
    *   **NPC Yönetimi (`castle_spawn_*`, `castle_*_die`, `castle_*_count`):** Kurbağa, koruma ve kuleleri belirlenen pozisyonlara/bölgelere spawn eder, `s_castle` veri yapısında takip eder ve öldüklerinde kayıtları günceller (kurbağa için ödül verir, kule için savaşın bitip bitmediğini kontrol eder).
    *   **Savaş Kuralları (`castle_can_attack`):** Kale haritalarındaki saldırı izinlerini savaş durumuna ve karakterlerin imparatorluklarına göre belirler.
    *   **Diğer İşlevler:** Koruma kiralama maliyeti, kurbağayı paraya çevirme gibi yardımcı fonksiyonları içerir.
*   **Çalışma Prensibi:** Sunucu başlarken `castle_boot` ile veri yüklenir. Bir GM komutuyla `castle_siege` çağrılarak savaş başlatılır. `castle_start_siege` durumu ayarlar, kuleleri spawn eder ve `castle_siege_event` ile `castle_stone_event`'i başlatır. Event'ler periyodik olarak çalışır. Oyuncular kulelere saldırır. Kuleler öldüğünde `castle_tower_die` çağrılır, tüm kuleler ölürse savaş biter. Eğer süre dolarsa `castle_siege_event` savaşı bitirir. Savaş bittiğinde `castle_end_siege` çağrılarak durum sıfırlanır, event'ler iptal edilir.
*   **Bağlantılı Dosyalar:** `castle.h`, `stdafx.h`, `constants.h`, `config.h`, `char_manager.h`, `start_position.h`, `monarch.h`, `questlua.h`, `log.h`, `char.h`, `sectree_manager.h`, `<stdio.h>`.

### `changelook.cpp`

*   **Amaç:** `changelook.h`'de bildirilen `CChangeLook` sınıfının metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **Pencere ve Eşya Yönetimi (`Open`, `ItemCheckIn`/`Out`, `FreeItemCheckIn`/`Out`, `Clear`):** Karakter için `CChangeLook` nesnesi oluşturur, eşyaların pencereye eklenip çıkarılmasını yönetir ve istemciye ilgili paketleri (`HEADER_GC_CHANGE_LOOK_*`) gönderir.
    *   **İşlem Onayı (`Accept`):** Eşyaların ve ücretin geçerliliğini kontrol eder. Yangı düşer, sol eşyanın `TransmutationVnum` değerini ayarlar, sağdaki eşyayı ve (varsa) bileti `ITEM_MANAGER::RemoveItem` ile siler.
    *   **Kural Uygulamaları (`CanAddItem`, `CheckItem`, `CanAddPassItem`):** Bir eşyanın hangi slota konulabileceğini (tür, alt tür kontrolü) ve iki eşyanın birbiriyle uyumlu olup olmadığını (tür, alt tür, anti-flag, cinsiyet uyumu; kostüm-normal eşya geçişleri dahil) kontrol eden detaylı mantığı içerir. Ücretsiz geçiş biletini doğrular.
*   **Çalışma Prensibi:** Oyuncu NPC veya bir eşya ile etkileşime girerek sistemi başlattığında (`Open`), karakterle ilişkilendirilmiş bir `CChangeLook` nesnesi oluşturulur. Oyuncu envanterinden eşyaları pencereye sürüklediğinde `ItemCheckIn` çağrılır, kurallar kontrol edilir ve başarılıysa eşya sanal olarak pencereye yerleştirilir (istemciye paket gönderilir). "Kabul et" butonuna basıldığında `Accept` çağrılır, son kontroller yapılır, işlem gerçekleştirilir ve kullanılan eşyalar silinir.
*   **Bağlantılı Dosyalar:** `changelook.h`, `stdafx.h`, `char.h`, `item.h`, `item_manager.h`, `unique_item.h`, `desc.h`, `packet.h`.

### `char_affect.cpp`

*   **Amaç:** Karakterler üzerindeki anlık veya süreli etkileri (Affect - buff, debuff, zehir, özel durumlar vb.) yöneten fonksiyonları uygular. `CHARACTER` sınıfının etki (affect) ile ilgili metotlarını içerir. Bu dosya, becerilerin, iksirlerin veya diğer oyun mekaniklerinin karakterler üzerindeki geçici veya kalıcı etkilerini yönetmekten sorumludur.
*   **Temel İşlevler/İçerik:**
    *   **Etki Yönetimi (`FindAffect`, `AddAffect`, `RemoveAffect`, `ClearAffect`):** Karakter üzerinde belirli bir etkiyi arar, yeni bir etki ekler (varsa üzerine yazma seçeneğiyle), belirli bir etkiyi veya türdeki tüm etkileri kaldırır ve karakterin tüm (veya belirli koşullara uyan) etkilerini temizler. Ekleme/kaldırma işlemleri hem karakterin statülerini günceller hem de istemciye ve veritabanına gerekli bilgileri gönderir.
    *   **Periyodik Güncelleme (`affect_event`, `UpdateAffect`, `ProcessAffect`, `StartAffectEvent`):** Düzenli aralıklarla çalışan bir event (`affect_event`) kullanarak etkilerin sürelerini azaltır, varsa SP maliyetlerini düşer, otomatik HP/SP yenilenmesi gibi sürekli etkileri işler ve süresi dolan veya koşulları artık sağlanmayan (örneğin premium süresi biten, lonca savaşı biten) etkileri kaldırır.
    *   **Hesaplama (`ComputeAffect`, `RefreshAffect`):** Bir etkinin karakterin statülerine (Point'ler ve Affect Flag'leri - `m_afAffectFlag`) olan etkisini uygular veya geri alır. `RefreshAffect` tüm mevcut etkileri yeniden hesaplar. Özel etkiler (`SKILL_MUYEONG`, `SKILL_CHEONUN` gibi) için ek event'leri tetikleyebilir/durdurabilir.
    *   **Kayıt/Yükleme (`SaveAffect`, `LoadAffect`):** Karakterin kalıcı etkilerini (bazı özel etkiler hariç - `IS_NO_SAVE_AFFECT`) veritabanına kaydeder ve karakter oyuna girdiğinde geri yükler. Yükleme sırasında eşyaya bağlı etkilerin (örn. otomatik iksir, ruh taşı) geçerliliğini kontrol eder.
    *   **Grup Halinde Kaldırma (`RemoveGoodAffect`, `RemoveBadAffect`, `IsGoodAffect`):** Önceden tanımlanmış listelere göre "iyi" veya "kötü" olarak kabul edilen etkileri topluca kaldırır (örn. Arena başlangıcında).
    *   **Etki Yığınlama (`Set/Get/ClearAffectStack`):** Bazı etkilerin (örn. zehir) kaç kez üst üste uygulandığını takip etmek için bir mekanizma sunar (`m_map_affectStack`).
    *   **Paketleme (`SendAffectAddPacket`, `SendAffectRemovePacket`):** Etki ekleme/kaldırma bilgilerini istemciye (`HEADER_GC_AFFECT_ADD`/`REMOVE`) ve veritabanına (`HEADER_GD_ADD_AFFECT`/`REMOVE_AFFECT`) gönderen yardımcı fonksiyonlar.
*   **Çalışma Prensibi:** Beceriler, iksirler veya sistem olayları `AddAffect` fonksiyonunu çağırarak karakterlere etki ekler. Bu etkiler `ComputeAffect` ile karakterin değerlerini (HP, SP, saldırı hızı vb.) ve durumlarını (sersemleme, görünmezlik vb. - `m_afAffectFlag`) anında değiştirir. `affect_event` periyodik olarak çalışarak etkilerin sürelerini yönetir, bedellerini alır ve süresi dolanları `ProcessAffect` ile kaldırır. `RemoveAffect` ile etkiler manuel olarak veya süreleri dolduğunda kaldırılır ve etkileri geri alınır. `SaveAffect` ve `LoadAffect` ile etkilerin kalıcılığı sağlanır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `affect.h`, `packet.h`, `buffer_manager.h`, `desc_client.h`, `battle.h`, `guild.h`, `utils.h`, `locale_service.h`, `lua_incl.h`, `arena.h`, `horsename_manager.h`, `item.h`, `DragonSoul.h`.

### `char_aura.cpp`

*   **Amaç:** Aura Kostüm Sistemi'nin (`__AURA_COSTUME_SYSTEM__`) sunucu taraflı işlevlerini uygular. Bu sistem, özel aura kostümlerinin seviye atlaması, evrimleşmesi ve diğer eşyaların bonuslarını emmesi (absorbe etmesi) gibi işlevleri içerir. Bu dosya, bu işlemlerin yapıldığı özel arayüz (Aura Refine Window) ile ilgili tüm sunucu mantığını yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Arayüz Yönetimi (`OpenAuraRefineWindow`, `AuraRefineWindowClose`, `IsAuraRefineWindowOpen`, `IsAuraRefineWindowCanRefine`):** Oyuncunun belirli bir Aura İşleme Penceresi türünü (Absorb, Growth, Evolve) açmasını, kapatmasını ve işlem yapmaya uygun olup olmadığını (NPC'ye yakınlık, pencerenin açık olması vb.) kontrol eder. İstemciye ilgili paketleri gönderir (`AURA_SUBHEADER_GC_OPEN`/`CLOSE`).
    *   **Eşya Yerleştirme/Çıkarma (`AuraRefineWindowCheckIn`, `AuraRefineWindowCheckOut`):** Oyuncunun envanterinden Aura penceresindeki slotlara eşya eklemesini ve çıkarmasını yönetir. Pencere türüne göre katı kurallar uygular (doğru eşya türü mü, kostüm mü, malzeme mi, seviyesi uygun mu, kilitli mi, takılı mı vb.). Eşyaları işlem sırasında kilitler/kilidini açar (`Lock`) ve istemciye durumu bildiren paketler (`AURA_SUBHEADER_GC_SET_ITEM`/`CLEAR_SLOT`/`CLEAR_ALL`) gönderir. Ayrıca işlem öncesi sonuç bilgisini (`AURA_SUBHEADER_GC_REFINE_INFO`) gönderebilir.
    *   **Bilgi Hesaplama (`__GetAuraRefineInfo`, `__CalcAuraRefineInfo`, `__GetAuraEvolvedRefineInfo`):** Aura kostümlerinin mevcut seviye/deneyimini (`ITEM_SOCKET_AURA_CURRENT_LEVEL` soketinden) okur, malzeme eklendiğinde sonraki seviye/deneyimi hesaplar ve evrimleşme için uygun olup olmadığını kontrol eder. Bu hesaplamalar için `constants.cpp`'deki `GetAuraRefineInfo` ile seviye tablolarına başvurur.
    *   **İşlem Gerçekleştirme (`AuraRefineWindowAccept`):** Oyuncu "Onayla" butonuna bastığında seçilen pencere türüne göre ana işlemi yürütür:
        *   **Absorb (Emilim):** Hedef aura kostümüne, malzeme eşyasının VNUM'unu (`ITEM_SOCKET_AURA_DRAIN_ITEM_VNUM`), bonuslarını (`CopyAttributeTo`), rastgele efsunlarını (`CopyAppliesRandomTo`) ve min/max değerlerini (`CopyMinMaxValuesTo` - eğer varsa) kopyalar. Malzeme eşyasını siler.
        *   **Growth (Büyüme/Seviye Atlama):** Hedef aura kostümünün seviye ve deneyim soketini (`ITEM_SOCKET_AURA_CURRENT_LEVEL`), kullanılan malzeme miktarına göre hesaplayarak günceller. Malzeme eşyasını kısmen veya tamamen siler.
        *   **Evolve (Evrimleşme):** Gerekli altın (`GetGold`), malzeme (`CountSpecifyItem`) ve aura kostümünün durumunu (maks seviye/deneyim) kontrol eder. Belirlenen şansa (`AURA_REFINE_INFO_EVOLVE_PCT`) göre: Başarılı olursa, yeni (daha yüksek seviyeli) bir aura kostümü oluşturur, eski kostümün özelliklerini aktarır, seviye soketini günceller, eski kostümü ve malzemeyi siler, altını alır (`PointChange`). Başarısız olursa, sadece malzemeyi kullanır/siler.
    *   Tüm işlemler sonucunda eşyalar güncellenir, silinir veya oluşturulur (`ITEM_MANAGER`), oyuncunun altını (`PointChange`) ayarlanır ve işlem loglanır (`LogManager::instance().ItemLog`). Son olarak pencere temizlenir ve istemciye bildirilir.
*   **Çalışma Prensibi:** Oyuncu bir NPC veya eşya ile etkileşime girerek Aura İşleme Penceresini (`OpenAuraRefineWindow`) açar. Envanterinden ilgili aura kostümünü ve malzemeyi pencereye sürükler (`AuraRefineWindowCheckIn`). Sistem, eşyaların uygunluğunu kontrol eder ve önizleme bilgisi sunabilir. Oyuncu "Onayla" dediğinde (`AuraRefineWindowAccept`), sunucu seçilen işleme (Absorb, Growth, Evolve) göre son kontrolleri yapar (altın, malzeme, şans vb.), işlemi gerçekleştirir, envanteri günceller ve sonucu oyuncuya bildirir.
*   **Not:** Bu sistem `#if defined(__AURA_COSTUME_SYSTEM__)` direktifi ile derlemeye dahil edilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `item.h`, `desc.h`, `log.h`, `item_manager.h`, `buffer_manager.h`, `unique_item.h`, `utils.h`, `constants.h`. (Linter hataları göz ardı edilmiştir).

### `char_battle.cpp`

*   **Amaç:** `CHARACTER` sınıfının savaşla ilgili temel fonksiyonlarını uygular. Saldırı başlatma, hasar hesaplama (fiziksel, büyüsel, menzilli), beceri kullanma, hedef yönetimi, aggro (tehdit) takibi, ölüm mekanikleri, tecrübe ve eşya dağıtımı, PK (Oyuncu Katili) modları ve hizalama (alignment) gibi savaşın ana unsurlarını yönetir. Bu dosya, oyunun savaş mekaniklerinin çekirdeğidir.
*   **Temel İşlevler/İçerik:**
    *   **Saldırı Yönetimi (`Attack`, `CanBeginFight`, `BeginFight`, `CanFight`, `battle_is_attackable`):** Bir karakterin başka bir karaktere saldırıp saldıramayacağını kontrol eder (durum, pozisyon, PK kuralları, kale savaşı kuralları vb.), saldırı durumunu başlatır ve saldırı eylemini tetikler. Hız hilesi kontrolleri içerir.
    *   **Hasar Hesaplama ve Uygulama (`Damage`, `battle_melee_attack`, `CalcArrowDamage`, `CalcMagicDamage`, `CalcMeleeDamage`):** Farklı saldırı türleri için temel hasarı hesaplar. `Damage` fonksiyonu, gelen ham hasarı alır ve hedefin savunma, direnç, bonus, kritik/delici vuruş, blok/savuşturma, yansıtma, mana kalkanı gibi birçok faktörü hesaba katarak nihai hasarı belirler ve uygular. Hasar bilgilerini (`SendDamagePacket`) ve efektleri (`EffectPacket`) gönderir. Özel sistemlere (Mavi Ejderha, Meley, Nemere, Ochao vb.) özgü hasar modifikasyonları içerir.
    *   **Beceri Kullanımı (`ComputeSkill`, `Shoot`, `FlyTarget`, `CFuncShoot`):** Becerilerin hedeflere uygulanmasını yönetir. `Shoot`, menzilli saldırıları/becerileri işler, `FlyTarget` mermi/hedef bilgilerini gönderir. Beceriye özel kaynak tüketimini (SP, ok) ve etkileri uygular.
    *   **Ölüm ve Sonuçları (`Dead`, `RewardlessDead`, `Stun`, `IsDead`, `dead_event`, `DeathPenalty`, `ItemDropPenalty`):** Karakterin ölümünü işler. Kötü etkileri kaldırır, PK/Lonca Savaşı/Arena sonuçlarını (hizalama düşüşü, eşya düşürme, tecrübe kaybı) uygular. Ölüm loglarını kaydeder, dirilme/yok olma event'ini (`dead_event`) başlatır. `Stun` sersemlemeyi yönetir. `RewardlessDead` ödül vermeden ölümü sağlar.
    *   **Tecrübe ve Eşya Dağıtımı (`Reward`, `DistributeExp`, `GiveExp`, `RewardGold`, `NPartyExpDistribute`):** Canavar öldürüldüğünde tecrübe ve altın dağıtır. Tecrübe hesaplaması seviye farkı, bonuslar (premium, eşya, evlilik, parti) gibi etkenleri içerir. Parti içi dağıtımı ve düşen eşyaların sahipliğini/zarını yönetir.
    *   **Hedef ve Tehdit Yönetimi (`SetVictim`, `GetVictim`, `GetNearestVictim`, `GetHighestDpsVictim`, `UpdateAggrPoint`, `UpdateAggrPointEx`, `ChangeVictimByAggro`, `m_map_kDamage`):** Karakterin hedefini belirler. Canavarlar için, saldıranlardan alınan hasarı (`m_map_kDamage`) ve tehdit puanlarını (`iAggro`) takip ederek hedef değiştirmeyi yönetir.
    *   **PK Modu ve Hizalama (`SetPKMode`, `GetPKMode`, `SetKillerMode`, `IsKillerMode`, `UpdateKillerMode`, `UpdateAlignment`, `GetAlignment`, `GetRealAlignment`, `ShowAlignment`):** Oyuncunun PK modunu (Barış, Lonca, Serbest vb.), Katil Modu'nu ve Hizalama puanını yönetir.
    *   **Özel Savaş Komutları (`ForgetMyAttacker`, `AggregateMonster`, `AttractRanger`, `PullMonster`):** Cesaret Pelerini gibi eşyalarla tetiklenen, çevredeki canavarları manipüle etme işlevlerini içerir.
*   **Çalışma Prensibi:** Bir karakter `Attack` fonksiyonunu çağırdığında, hedef ve saldırı türüne göre ilgili hasar hesaplama fonksiyonları (`battle_melee_attack`, `ComputeSkill`, `Shoot`) tetiklenir. `Damage` fonksiyonu bu hasarı alır, hedef ve saldırganın durumuna göre modifiye eder ve hedefin HP'sini düşürür. Canavarlarda `UpdateAggrPoint` ile tehdit listesi güncellenir. HP sıfıra düşerse `Dead` fonksiyonu çağrılır; ölüm cezaları, tecrübe/eşya dağıtımı ve dirilme/yok olma süreci yönetilir. Tüm bu süreç boyunca PK modu, hizalama, özel durumlar (arena, savaş vb.) sürekli kontrol edilir.
*   **Not:** Dosya, çok sayıda `#if defined` bloğu ile birçok isteğe bağlı savaş özelliğini içerir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `char.h`, `item.h`, `skill.h`, `battle.h`, `packet.h`, `affect.h`, `party.h`, `guild.h`, `pvp.h`, `arena.h`, `castle.h`, `threeway_war.h`, `dungeon.h`, `questmanager.h`, `item_manager.h`, `char_manager.h`, `mob_manager.h`, `log.h`, ve çeşitli özel sistem header'ları (`BlueDragon.h`, `MeleyLair.h`, `TempleOchao.h`, `ShipDefense.h`, `GrowthPetSystem.h`, `CubeManager.h` vb.).

### `char_cards.cpp`

*   **Amaç:** Karakterler için bir kart mini oyununu (Muhtemelen Okey Kart Oyunu - `okey_event` tablosuna göre) uygular. Oyuncuların kart setleri kullanarak oynadığı, kartları topladığı, kombinasyonlar oluşturarak puan kazandığı ve puanına göre ödül aldığı bir sistemi yönetir.
*   **Temel İşlevler/İçerik:**
    *   **Oyun Yönetimi (`Cards_open`, `Cards_clean_list`, `CardsEnd`):** Oyunu başlatır (gerekirse altın ve kart seti - VNUM 79506 - alır), oyun verilerini (`character_cards`) sıfırlar ve oyun bittiğinde puanına göre ödül verir (VNUM 50267-50269) ve veritabanına log kaydı (`okey_event`) düşer.
    *   **Kart İşlemleri (`Cards_pullout`, `CardsDestroy`, `CardsAccept`, `CardsRestore`):** Desteden kart çekmeyi, eldeki kartı atmayı, eldeki kartı oyun alanına kabul etmeyi ve alandaki kartı ele geri almayı yönetir.
    *   **Kart Veri Yapıları (`character_cards`, `randomized_cards`):** Oyuncunun elindeki kartları (`cards_in_hand`), oyun alanındaki kartları (`cards_in_field`), kalan kart sayısını (`cards_left`) ve toplam puanı (`points`) `character_cards` yapısında tutar. Çekilen kartların tekrar çekilmemesi için `randomized_cards` dizisini kullanır.
    *   **Rastgeleleştirme (`RandomizeCards`, `CardWasRandomized`):** Desteden rastgele bir kart (tip 1-3, değer 1-8) çeker ve daha önce çekilip çekilmediğini kontrol eder.
    *   **Ödül Mantığı (`CheckReward`, `TypesAreSame`, `ValuesAreSame`, `CardsMatch`, `GetLowestCard`, `ResetField`, `RestoreField`):** Oyun alanındaki 3 kartın kombinasyonunu kontrol eder (aynı tip/değer, sıralı vb.), uygunsa puan (`field_points`) hesaplar, toplam puana ekler ve alanı temizler (`ResetField`). Kombinasyon yoksa kartları ele geri döndürür (`RestoreField`).
    *   **İstemci İletişimi (`SendUpdatedInformations`, `SendReward`):** Oyunun mevcut durumunu ve ödül bilgisini `CHAT_TYPE_COMMAND` üzerinden `cards open/info/finfo/reward` komutlarıyla istemciye gönderir.
    *   **Sıralama (`GetGlobalRank`, `GetRundRank`):** Veritabanındaki `okey_event` tablosundan genel ve mevcut tur için en yüksek puanlı oyuncuları çekerek sıralama bilgisi oluşturur.
*   **Çalışma Prensibi:** Oyuncu `Cards_open` ile oyunu başlatır. `Cards_pullout` ile rastgele kart çeker. `CardsAccept` ile kartları oyun alanına yerleştirir, `CardsDestroy` ile istemediği kartları atar. Alan dolduğunda `CheckReward` ile kombinasyon kontrol edilir. Puan kazanılırsa alan sıfırlanır, kazanılmazsa kartlar ele döner. Kalan kart bittiğinde veya oyuncu bitirdiğinde `CardsEnd` ile toplam puana göre ödül verilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `utils.h`, `log.h`, `db.h`, `dev_log.h`, `locale_service.h`, `config.h`, `packet.h`, `item.h`, `item_manager.h`.

### `char_change_empire.cpp`

*   **Amaç:** Oyuncu hesaplarının imparatorluk değiştirmesi işlevini uygular. Belirli kısıtlamaları (lonca üyeliği, evlilik) kontrol eder ve başarılı olursa ilgili hesabın imparatorluğunu veritabanında günceller.
*   **Temel İşlevler/İçerik:**
    *   **`ChangeEmpire(BYTE empire)`:** Ana imparatorluk değiştirme fonksiyonu.
        *   Hedef imparatorluğun mevcut imparatorlukla aynı olup olmadığını kontrol eder.
        *   Hesaba bağlı tüm oyuncu ID'lerini (`player_index` tablosu) alır.
        *   Hesaptaki herhangi bir karakterin bir loncaya üye olup olmadığını (`guild_member` tablosu) kontrol eder. Lonca üyesi varsa işlemi iptal eder (kod 2).
        *   Hesaptaki herhangi bir karakterin evli veya nişanlı olup olmadığını (`marriage::CManager`) kontrol eder. Evli/nişanlıysa işlemi iptal eder (kod 3).
        *   Tüm kontroller geçerse, `player_index` tablosundaki hesabın `empire` değerini yeni imparatorluk ID'si ile günceller.
        *   Değişiklik sayısını kaydetmek için `SetChangeEmpireCount` fonksiyonunu çağırır.
    *   **`GetChangeEmpireCount()`:** Verilen hesabın daha önce kaç kez imparatorluk değiştirdiğini `change_empire` tablosundan sorgular.
    *   **`SetChangeEmpireCount()`:** İmparatorluk değiştirme işlemi başarılı olduğunda `change_empire` tablosuna kayıt ekler veya mevcut kaydın sayacını artırır.
    *   **`GetAID()`:** Verilen oyuncu ID'sine göre hesap ID'sini bulan yardımcı fonksiyon.
*   **Çalışma Prensibi:** Oyuncu imparatorluk değiştirme talebinde bulunduğunda `ChangeEmpire` fonksiyonu çağrılır. Fonksiyon, önce oyuncunun mevcut imparatorluğunu, ardından lonca ve evlilik durumunu kontrol eder. Eğer herhangi bir kısıtlama yoksa, veritabanındaki `player_index` tablosunu güncelleyerek hesabın imparatorluğunu değiştirir ve bu değişikliği `change_empire` tablosuna kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `db.h`, `guild_manager.h`, `marriage.h`, `service.h`.

### `char_dragonsoul.cpp`

*   **Amaç:** Karakter sınıfının Ejderha Taşı Simyası (Dragon Soul - DS) sistemiyle ilgili işlevlerini uygular. Özellikle DS setlerinin (deck) aktivasyonu, deaktivasyonu, nitelik kontrolü ve arındırma penceresi yönetimi gibi temel etkileşimleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **Başlatma ve Temizleme (`DragonSoul_Initialize`, `DragonSoul_CleanUp`):** Karakter oyuna girdiğinde (`Initialize`) kaydedilmiş aktif seti (`AFFECT_DRAGON_SOUL_DECK_*`) yeniden etkinleştirir. Çıkışta (`CleanUp`) tüm aktif DS eşyalarını devre dışı bırakır.
    *   **Set Yönetimi (`DragonSoul_GetActiveDeck`, `DragonSoul_IsDeckActivated`, `DragonSoul_ActivateDeck`, `DragonSoul_DeactivateAll`):** Aktif olan DS setini (0 veya 1) döndürür, herhangi bir setin aktif olup olmadığını kontrol eder. Belirli bir seti aktif hale getirir (önce `DeactivateAll` çağrılır, sonra ilgili `AFFECT_DRAGON_SOUL_DECK_*` eklenir ve `DSManager::instance().ActivateDragonSoul` ile eşya bonusları uygulanır). `DeactivateAll`, tüm set etkilerini kaldırır ve `DSManager::instance().DeactivateDragonSoul` ile bonusları geri alır.
    *   **Nitelik Kontrolü (`DragonSoul_IsQualified`, `DragonSoul_GiveQualification`):** Karakterin DS sistemini kullanmaya yetkili olup olmadığını (`AFFECT_DRAGON_SOUL_QUALIFIED` affect'i) kontrol eder ve yetki verir.
    *   **Set Bonusu (`DragonSoul_HandleSetBonus` - `#if defined(__DS_SET__)`):** Aktif setteki tüm slotlar doluysa ve set bonusu özelliği aktifse, `DSManager` üzerinden setin seviyesini (`iSetGrade`) ve bonus değerlerini alarak karakterin statülerine (`ApplyPoint`) uygular/kaldırır. Setin aktifliğini `NEW_AFFECT_DS_SET` affect'i ile takip eder.
    *   **Arındırma Penceresi (`DragonSoul_RefineWindow_Open`, `DragonSoul_RefineWindow_ChangeAttr_Open`, `DragonSoul_RefineWindow_Close`, `DragonSoul_RefineWindow_CanRefine`):** DS arındırma ve (eğer aktifse - `#if defined(__DS_CHANGE_ATTR__)`) efsun değiştirme pencerelerinin açılmasını/kapanmasını yönetir. Pencereyi açan NPC/Entity bilgisini (`m_pointsInstant.m_pDragonSoulRefineWindowOpener`) saklar ve istemciye ilgili paketleri (`HEADER_GC_DRAGON_SOUL_REFINE`) gönderir.
*   **Çalışma Prensibi:** Oyuncu DS kullanma hakkı kazandığında (`GiveQualification`) `AFFECT_DRAGON_SOUL_QUALIFIED` alır. Oyuncu bir DS setini (`ActivateDeck`) seçtiğinde, eski set devre dışı bırakılır, yeni setin affect'i eklenir ve `DSManager` aracılığıyla o setteki taşların bonusları karaktere uygulanır. Eğer set bonusu aktifse (`__DS_SET__`), `HandleSetBonus` ile ek bonuslar da uygulanır. Oyuncu seti değiştirdiğinde veya devre dışı bıraktığında (`DeactivateAll`), tüm bonuslar geri alınır. Arındırma penceresi NPC etkileşimi ile açılır/kapanır ve sunucu bu durumu takip eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `item.h`, `desc.h`, `DragonSoul.h`, `log.h`, `config.h`, `affect.h`, `packet.h`.

### `char_gem.cpp`

*   **Amaç:** Gaya (Gem) Sistemi'ni yönetir. Bu sistem, oyuncuların Metin taşlarını özel bir para birimi olan Gaya'ya dönüştürmesini (`CraftGemItems`), bu Gaya ile özel bir mağazadan (`Gem Market`) eşya almasını (`MarketGemItems`, `BuyItemsGemMarket`) ve bu mağazanın içeriğini yönetmesini (`RefreshItemsGemMarket`, `InfoGemMarker`) sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Yapılandırma Yükleme (`LoadGemSystem`):** Sunucu başlarken `gem.txt` dosyasından Gaya Mağazası'nda çıkabilecek eşyaları, maliyetlerini, Gaya üretim oranlarını, gereken malzemeleri (Işıltılı Taş - `glimmerstone`, Pazar Genişletme/Yenileme eşyaları - `gem_expansion`/`gem_refresh`) ve Yang maliyetini yükler.
    *   **Oyuncu Mağazası Yönetimi (`UpdateItemsGemMarker0`, `UpdateItemsGemMarker`, `RefreshItemsGemMarket`, `InfoGemMarker`, `ClearGemMarket`, `CheckItemsFull`):** Her oyuncu için ayrı bir Gaya Mağazası listesi tutar (`gem/<oyuncu_adı>_gem_info.txt`). Oyuncu ilk kez açtığında veya yenilediğinde (`RefreshItemsGemMarket`), `gem.txt`'deki listeden rastgele 9 farklı eşya seçilerek bu dosyaya yazılır. `InfoGemMarker`, dosyadaki mevcut eşyaları ve açık slotları okuyup istemciye (`CHAT_TYPE_COMMAND`, `GemMarket*` komutları) gönderir.
    *   **Gaya Üretimi (`CraftGemItems`):** Oyuncunun envanterindeki Metin taşını, belirli sayıda Işıltılı Taş ve Yang karşılığında, belirli bir olasılıkla (`Prob_Gem`) Gaya puanına (`POINT_GEM`) dönüştürür. Başarılı/başarısız olma durumunu bildirir, malzemeleri siler.
    *   **Mağaza Kullanımı (`MarketGemItems`, `BuyItemsGemMarket`, `CheckSlotGemMarket`, `UpdateSlotGemMarket`):** Oyuncunun Gaya Mağazası'ndaki bir slottan eşya almasını sağlar. Eğer slot kilitliyse (3-8 arası slotlar) ve oyuncuda Pazar Genişletme eşyası varsa, önce slot açılır (`UpdateSlotGemMarket`), ardından eşya alınabilir. Satın alma işlemi Gaya puanını düşürür ve eşyayı oyuncuya verir (`AutoGiveItem`).
    *   **Mağaza Yenileme (`RefreshItems`):** Oyuncu Pazar Yenileme eşyasını kullanarak mevcut mağaza listesini manuel olarak yenileyebilir.
    *   **Zamanlayıcı (`StartCheckTimeMarket`, `check_time_market_event`):** Oyuncunun Gaya Mağazası'nın otomatik olarak yenilenmesi için bir zamanlayıcı (`gem_system.gem_refresh_time` quest flag'i, varsayılan 5 saat) başlatır ve takip eder. Süre dolduğunda mağazayı yeniler ve yeni zamanı ayarlar.
*   **Çalışma Prensibi:** Sistem `LoadGemSystem` ile yapılandırmayı yükler. Oyuncu NPC ile etkileşime girdiğinde (`InfoGemMarker`), oyuncuya özel mağaza dosyası okunur (yoksa `UpdateItemsGemMarker` ile oluşturulur) ve içerik istemciye gönderilir. Oyuncu, Metin taşı ve Işıltılı Taş kullanarak (`CraftGemItems`) Gaya puanı kazanır. Kazandığı Gaya ile mağazadan (`MarketGemItems`) eşya satın alabilir. Kapalı slotları Pazar Genişletme ile açabilir. Mağaza içeriğini Pazar Yenileme ile veya otomatik zamanlayıcı (`check_time_market_event`) ile yenileyebilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `utils.h`, `config.h`, `char.h`, `item.h`, `item_manager.h`, `locale_service.h`, `questmanager.h`, `packet.h`, `db.h`.

### `char_horse.cpp`

*   **Amaç:** Karakterin at (horse) veya binek (mount) ile etkileşimlerini yönetir. At çağırma, binme/inme, atın seviyesi, sağlığı, dayanıklılığı ve özel yetenekleriyle ilgili işlevleri içerir. `CHorseRider` sınıfının karakter üzerindeki arayüzünü sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Binme/İnme (`StartRiding`, `StopRiding`):** Ata veya bineğe binme ve inme işlemlerini yönetir. Koşulları kontrol eder (canlı mı, dönüşmüş mü, harita uygun mu, at/binek durumu uygun mu). Binildiğinde at karakterini gizler (`HorseSummon(false)`), binek VNUM'unu ayarlar (`MountVnum`). İnildiğinde `quest::CQuestManager::instance().Unmount` çağrılır, binek VNUM'u sıfırlanır ve at karakteri tekrar çağrılır (`HorseSummon(true)`).
    *   **At Çağırma/Gönderme (`HorseSummon`, `horse_dead_event`):** Oyuncunun atını temsil eden `CHARACTER` nesnesini (`m_chHorse`) yaratır (`CHARACTER_MANAGER::instance().SpawnMob`) veya yok eder (`M2_DESTROY_CHARACTER`). Atın VNUM'unu (`GetMyHorseVnum`), adını (`CHorseNameManager`), seviyesini ve ölü/canlı durumunu ayarlar. At öldüğünde `horse_dead_event` ile belirli bir süre sonra yok olmasını sağlar.
    *   **At/Binici İlişkisi (`SetRider`, `GetRider`, `ClearHorseInfo`):** Oyuncu karakteri ile at karakteri arasındaki çift yönlü bağlantıyı yönetir (oyuncunun `m_chHorse` işaretçisi, atın `m_chRider` işaretçisi).
    *   **Durum Bildirimi (`SendHorseInfo`):** Atın seviyesi, sağlık ve dayanıklılık durumunu (derecelendirilmiş olarak) istemciye `CHAT_TYPE_COMMAND` (`horse_state`) ile gönderir.
    *   **At VNUM Belirleme (`GetMyHorseVnum`):** Oyuncunun sahip olduğu atın seviyesine ve lonca durumuna (lider mi, üye mi) göre doğru at VNUM'unu hesaplar.
    *   **At Ölümü/Canlandırma (`HorseDie`, `ReviveHorse`):** At öldüğünde `CHorseRider`'daki ilgili fonksiyonu çağırır ve atı gönderir. At canlandırıldığında `CHorseRider`'daki fonksiyonu çağırır ve atı tekrar çağırır.
    *   **At Yetenekleri (`CanUseHorseSkill`):** Oyuncunun at üzerinde yetenek kullanıp kullanamayacağını atın/bineğin seviyesine veya türüne göre kontrol eder.
    *   **Seviye Senkronizasyonu (`SetHorseLevel`):** Atın seviyesini hem `CHorseRider`'da hem de karakterin `SKILL_HORSE` yetenek seviyesinde günceller.
*   **Çalışma Prensibi:** Oyuncu ata binmek istediğinde (`StartRiding`), sistem kontrolleri yapar, `CHorseRider` durumunu günceller, `MountVnum` ile karakterin görünümünü değiştirir ve mevcut at karakterini (`m_chHorse`) gizler. Oyuncu indiğinde (`StopRiding`), `MountVnum` sıfırlanır ve at karakteri tekrar görünür hale getirilir (`HorseSummon`). At çağırma (`HorseSummon(true)`) ve gönderme (`HorseSummon(false)`) işlemleri at karakterinin yaratılmasını/yok edilmesini yönetir. Atın durumu (`SendHorseInfo`) periyodik olarak istemciye iletilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `packet.h`, `guild.h`, `vector.h`, `questmanager.h`, `item.h`, `horsename_manager.h`, `locale_service.h`, `arena.h`, `VnumHelper.h`.

### `char_item.cpp`

*   **Amaç:** `CHARACTER` sınıfının eşyalarla ilgili tüm fonksiyonlarını uygular. Bu, envanter yönetimi (ana, Ejderha Taşı, özel envanterler), eşya kullanma, kuşanma/çıkarma, düşürme/toplama, takas etme, eşya üretme/geliştirme (refine), metin taşı ekleme/çıkarma ve eşyalarla tetiklenen özel sistem etkileşimlerini içerir. Kısacası, oyuncunun oyun dünyasındaki nesnelerle etkileşiminin büyük bir kısmını yönetir.
*   **Temel İşlev Grupları ve Önemli Fonksiyonlar:**
    *   **Envanter Erişimi ve Yönetimi:**
        *   `GetItem(TItemPos Cell)` / `SetItem(TItemPos Cell, LPITEM pItem)`: Belirtilen pozisyondaki eşyayı alır veya ayarlar. Tüm eşya manipülasyonlarının temelidir.
        *   `Get*InventoryItem`: Özel envanterlerdeki (Ejderha Taşı, Beceri Kitabı, Yükseltme, Taş, Hediye Kutusu) eşyaları alır.
        *   `IsEmptyItemGrid(TItemPos Cell, BYTE bSize, ...)`: Belirtilen alanın envanterde boş olup olmadığını kontrol eder.
        *   `GetEmptyInventory(BYTE size)` / `GetEmpty*Inventory(BYTE size)`: Belirtilen boyuttaki bir eşya için envanterde (veya özel envanterde) ilk boş slotu bulur.
        *   `MoveItem(TItemPos Cell, TItemPos DestCell, WORD count)`: Eşyaları envanter içinde veya envanterler arasında taşır (yığınlama, ayırma dahil).
        *   `SwapItem(UINT bCell, UINT bDestCell)`: İki envanter slotundaki eşyaları doğrudan değiştirir.
    *   **Eşya Kullanımı (`UseItem`, `UseItemEx`):**
        *   `UseItemEx(LPITEM item, ...)`: Eşyanın türüne ve alt türüne göre kullanım mantığını yönlendirir. İksirler (HP/SP, etki), parşömenler (ışınlanma, sıfırlama), beceri kitapları (`SkillLevelUp`), görev eşyaları (`quest::CQuestManager`), özel yüzük/kolyeler (dönüşüm, evlilik ışınlanması), at çağırma, şebnemler (`Blend_Item_set_value`), saç stilleri, polimorf (`DoPolymorph`), kamp ateşi, balıklar, özel sandıklar/kutular (rastgele ödül - `AutoGiveItem`), Simya arayüzü (`DragonSoul_RefineWindow_Open`) ve VNUM'a özel birçok durumu (event eşyaları vb.) ele alır.
    *   **Eşya Kuşanma/Çıkarma (`EquipItem`, `UnequipItem`, `CanEquipNow`, `CanUnequipNow`):**
        *   `EquipItem(LPITEM item, ...)`: Eşyayı uygun giyim slotuna (`WEAR_*`) takar. Kısıtlamaları (seviye, sınıf, cinsiyet) kontrol eder, bonusları (`StartAffect`, `ApplyPoint`) uygular, varsa set bonuslarını günceller (`CheckEquipmentSets`).
        *   `UnequipItem(LPITEM item)`: Giyilen eşyayı çıkarıp envantere taşır, bonusları geri alır.
        *   `CanEquipNow`/`CanUnequipNow`: Eşyanın o anda takılıp/çıkarılıp çıkarılamayacağını kontrol eder (ticaret, pazar durumu vb.).
    *   **Geliştirme (Refine) ve Metin Taşları:**
        *   `SetRefineNPC(LPCHARACTER ch)`: Etkileşimdeki demirci NPC'sini ayarlar.
        *   `DoRefine(LPITEM item, ...)`: Normal eşya basma işlemini yapar (başarı şansı, malzeme kontrolü, sonuç - `TransformRefineItem`, `NotifyRefineSuccess`/`Fail`).
        *   `DoRefineWithScroll(LPITEM item)`: Kutsama/Demirci Kağıdı gibi parşömenlerle basma işlemini yapar (`enum_RefineScrolls`).
        *   `DoRefineSoul(LPITEM item)`: Muhtemelen Ruh Taşı veya binek eşyası basma işlemini yönetir.
        *   `RefineInformation(...)`: Geliştirme penceresi için gerekli bilgileri (malzemeler, oran, ücret) istemciye gönderir.
        *   `RefineItem(LPITEM pkItem, LPITEM pkTarget)`: Cevherleri/taşları silaha/zırha ekler.
        *   `DetachMetin(LPITEM pkItem, LPITEM pkTarget)`: Metin taşını silahtan/zırhtan ayırır (başarı/başarısızlık olasılığıyla).
    *   **Eşya Düşürme/Toplama (`DropItem`, `PickupItem`, `DropGold`, `DropCheque`):**
        *   `DropItem(...)`: Eşyayı yere bırakır (düşürülebilirlik kontrolü içerir).
        *   `DropGold`/`DropCheque`: Yere altın veya Won bırakır.
        *   `PickupItem(DWORD dwVID)`: Yerdeki bir eşyayı alır (sahiplik, parti kuralları - `NPartyPickupDistribute`, envanter kontrolü).
    *   **Eşya Verme/Alma (`GiveItem`, `ReceiveItem`, `CanReceiveItem`, `AutoGiveItem`):**
        *   `GiveItem(...)`: Doğrudan başka bir oyuncuya eşya verir.
        *   `CanReceiveItem(...)`: Bir oyuncunun başka birinden eşya alıp alamayacağını kontrol eder.
        *   `ReceiveItem(...)`: Başka bir oyuncudan eşya alır.
        *   `AutoGiveItem(...)`: Görev ödülü, sandık vb. durumlarda eşyayı doğrudan oyuncunun envanterine verir.
    *   **Eşya Bulma/Sayma/Silme (`FindSpecifyItem`, `CountSpecifyItem`, `RemoveSpecifyItem`, `FindItemByID`, `RemoveItem`):**
        *   Envanterde belirli VNUM'a/ID'ye sahip eşyayı bulur, sayar veya belirtilen miktarda siler.
    *   **Özel Sistem Etkileşimleri:**
        *   `BuffOnAttr_...`: Eşya efsunlarına bağlı özel buff'ları yönetir.
        *   `ItemProcess_Hair`/`Polymorph`: Saç/Dönüşüm eşyası kullanımını yönetir.
        *   `AutoRecoveryItemProcess`: Otomatik HP/SP iksirlerinin periyodik etkisini yönetir.
        *   `DoRefineElement`/`RefineElementInformation`: Elementel efsun basma sistemini yönetir.
        *   `SoulItemProcess`: Ruh sistemiyle ilgili eşya işlemlerini yönetir.
        *   `Attr67Add`: 6./7. Efsun ekleme sistemini yönetir.
*   **Genel Çalışma Prensibi:** Oyuncu eşyalarla etkileşime girdiğinde (sağ tıklama, sürükleme, NPC etkileşimi, yerden alma), bu dosyadaki ilgili fonksiyonlar çağrılır. Fonksiyonlar geçerlilik kontrolleri yapar, eşya verilerini ve karakter statülerini günceller, envanteri düzenler ve sonucu istemciye bildirir. Veritabanı ve loglama işlemleri de gerektiğinde yapılır.
*   **Not:** Dosya çok büyüktür ve birçok `#ifdef` bloğu ile modüler sistemler içerir. `UseItemEx` gibi fonksiyonlar çok sayıda VNUM'a özel hard-coded mantık barındırabilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `char.h`, `item.h`, `item_manager.h`, `packet.h`, `affect.h`, `skill.h`, `db.h`, `log.h`, `questmanager.h`, `fishing.h`, `party.h`, `dungeon.h`, `marriage.h`, `polymorph.h`, `blend_item.h`, `castle.h`, `BattleArena.h`, `safebox.h`, `shop.h` ve diğerleri.

### `char_manager.cpp`

*   **Amaç:** `char_manager.h`'de bildirilen `CHARACTER_MANAGER` sınıfının metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **Yapıcı/Yıkıcı:** Değişkenleri başlatır, bazı varsayılan ırk numaralarını kaydeder. Yıkıcı `Destroy`'u çağırır.
    *   **Karakter Yaşam Döngüsü (`CreateCharacter`, `DestroyCharacter`, `AllocVID`):** Yeni bir VID atar, karakter nesnesini oluşturur/yok eder, karakteri ilgili haritalara (VID, PID, İsim) ekler/çıkarır. Bekleyen yok etme (`PendingDestroy`) mantığını uygular.
    *   **Karakter Bulma (`Find`, `FindByPID`, `FindPC`, `FindSpecifyPC`):** İlgili haritalarda arama yapar. `FindSpecifyPC` iş sınıfı, harita ve seviye aralığına göre rastgele bir PC bulur.
    *   **Spawn Mekanizmaları (`SpawnMob`, `SpawnMobRange`, `SpawnGroup`, `SpawnGroupGroup`, `SpawnMoveGroup`, `SpawnMobRandomPosition`):**
        *   `SpawnMob`: Tek bir karakter (NPC/Canavar) yaratır. Harita, pozisyon (engel kontrolü dahil), prototip (`CMobManager`), özellikler (imparatorluk, rotasyon) ayarlar ve karakteri görünür hale getirir (`CHARACTER::Show`).
        *   `SpawnGroup`/`SpawnGroupGroup`: Gruptaki her üye için `SpawnMobRange` çağırır, grup üyeleri için otomatik parti oluşturur. Zindan veya Metin Taşı ile ilişkilendirebilir.
        *   `SpawnMoveGroup`: Bir grubu spawn edip hedef bir noktaya hareket ettirir.
    *   **Güncelleme (`Update`):** Oyun döngüsünde çağrılır. Tüm PC'ler için `CHARACTER::UpdateCharacter` ve durum listesindeki (`m_set_pkChrState`) karakterler için `CHARACTER::UpdateStateMachine` fonksiyonlarını çağırır. Periyodik olarak sohbet sayaçlarını sıfırlar ve canavar öldürme loglarını (`KillLog`) DB'ye gönderir. Bekleyen yok etme işlemlerini gerçekleştirir.
    *   **Gecikmeli Kayıt (`DelayedSave`, `FlushDelayedSave`, `ProcessDelayedSave`):** Kaydedilmesi gereken karakterleri `m_set_pkChrForDelayedSave` setine ekler. `ProcessDelayedSave` periyodik olarak çağrılarak setteki karakterlerin `SaveReal` metodunu çağırır.
    *   **Irk Numarası Yönetimi (`RegisterRaceNum`, `RegisterRaceNumMap`, `UnregisterRaceNumMap`, `GetCharactersByRaceNum`):** Belirli VNUM'lara sahip karakterlerin hızlı bulunabilmesi için `m_map_pkChrByRaceNum` haritasını yönetir.
    *   **Global Oran Yönetimi (`Get*Rate`):** Premium durumu ve Çin yorgunluk sistemi gibi faktörleri göz önüne alarak ilgili global oranı döndürür.
    *   **Banner Sistemi (`InitializeBanners`, `SpawnBanners`):** Yapılandırma dosyalarına göre banner NPC'lerini spawn eder/yok eder.
*   **Genel Çalışma Prensibi:** `CHARACTER_MANAGER`, oyundaki tüm karakter örneklerinin merkezi deposudur. Yeni karakterler `CreateCharacter` ile eklenir, `DestroyCharacter` ile kaldırılır. Oyun döngüsü `Update` fonksiyonunu çağırarak tüm karakterlerin güncellenmesini sağlar. Canavar/NPC spawn işlemleri `Spawn*` fonksiyonları aracılığıyla yapılır. Karakter bulma işlemleri için çeşitli arama fonksiyonları sunar. Global oyun oranlarını yönetir ve veritabanı ile loglama işlemlerine yardımcı olur.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char_manager.h`, `constants.h`, `utils.h`, `desc.h`, `char.h`, `mob_manager.h`, `party.h`, `regen.h`, `p2p.h`, `dungeon.h`, `db.h`, `config.h`, `xmas_event.h`, `questmanager.h`, `questlua.h`, `locale_service.h`, `VnumHelper.h`, `boost/bind.hpp`.

### `char_quickslot.cpp`

*   **Amaç:** Karakterin Hızlı Erişim (Quickslot) çubuğunu yönetir. Oyuncuların eşyaları, becerileri veya komutları bu slotlara atamasını, silmesini veya yer değiştirmesini sağlar. Envanterdeki değişikliklere göre kısayolları günceller.
*   **Temel İşlevler/İçerik:**
    *   **`SyncQuickslot(...)`:** Envanterdeki bir eşyanın yeri değiştiğinde veya kaldırıldığında, bu eşyaya bağlı tüm hızlı erişim slotlarını günceller veya siler.
    *   **`GetQuickslot(...)`:** Belirtilen pozisyondaki hızlı erişim slotunun verisine erişim sağlar.
    *   **`SetQuickslot(...)`:** Belirtilen pozisyona yeni bir hızlı erişim ataması yapar (eşya, beceri, komut). Geçerlilik kontrolü yapar ve istemciye `HEADER_GC_QUICKSLOT_ADD` paketi gönderir.
    *   **`DelQuickslot(...)`:** Belirtilen pozisyondaki hızlı erişim slotunu temizler ve istemciye `HEADER_GC_QUICKSLOT_DEL` paketi gönderir.
    *   **`SwapQuickslot(...)`:** İki hızlı erişim slotunun içeriğini değiştirir ve istemciye `HEADER_GC_QUICKSLOT_SWAP` paketi gönderir.
    *   **`ChainQuickslotItem(...)`:** Bir eşya (özellikle yığınlanabilir) başka bir eşya ile birleştirildiğinde veya yeri değiştiğinde, eski pozisyona bağlı kısayolu yeni eşyanın pozisyonuna günceller.
*   **Çalışma Prensibi:** İstemci hızlı erişim çubuğunda değişiklik yaptığında sunucuya paket gönderir. Sunucu isteği işler, karakterin `m_PlayerSlots` verisini günceller ve onay paketini geri gönderir. Envanterdeki eşya hareketleri sırasında `SyncQuickslot` veya `ChainQuickslotItem` çağrılarak kısayollar otomatik güncellenir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `char.h`, `desc.h`, `packet.h`, `item.h`.

### `char_resist.cpp`

*   **Amaç:** Karakterlerin zehir, ateş ve kanama gibi periyodik hasar (Damage over Time - DoT) etkilerini ve çeşitli etkilere karşı dirençlerini yönetir. Canavarların temel dirençlerini uygular.
*   **Temel İşlevler/İçerik:**
    *   **DoT Hasar Oranları (`GetPoisonDamageRate`, `GetBleedingDamageRate`):** Zehir ve kanama hasarının oranını karakterin Mob Rank'ına ve direncine göre hesaplar.
    *   **DoT Eventleri (`poison_event`, `fire_event`, `bleeding_event`):** Periyodik olarak tetiklenerek ilgili DoT hasarını (`CHARACTER::Damage`) uygular ve etki süresini/sayacını yönetir.
    *   **DoT Uygulama (`AttackedByFire`, `AttackedByPoison`, `AttackedByBleeding`):** İlgili DoT etkisini karaktere uygular (`AddAffect`) ve periyodik hasar event'ini başlatır. Seviye farkına göre direnç olasılığı içerir.
    *   **DoT Kaldırma (`RemoveFire`, `RemovePoison`, `RemoveBleeding`):** İlgili DoT affect'ini ve hasar event'ini kaldırır.
    *   **Canavar Dirençleri (`ApplyMobAttribute`):** Canavar yaratıldığında `mob_proto`'daki temel efsun ve direnç değerlerini karaktere uygular (`ApplyPoint`).
    *   **Bağışıklık Kontrolü (`IsImmune`):** Karakterin belirli bir etkiye (sersemletme, yavaşlatma vb.) karşı bağışıklığı olup olmadığını (`m_pointsInstant.dwImmuneFlag`) %90 ihtimalle kontrol eder.
*   **Çalışma Prensibi:** Bir saldırı DoT etkisi uyguladığında (`AttackedBy*`), karakter üzerine affect eklenir ve periyodik hasar veren bir event (`*_event`) başlatılır. Event, süre dolduğunda veya etki kaldırıldığında (`Remove*`) sona erer. `ApplyMobAttribute` canavarın başlangıç direncini ayarlar. `IsImmune` fonksiyonu, debuff uygulanmadan önce bağışıklık kontrolü için kullanılır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `config.h`, `char.h`, `char_manager.h`, `affect.h`, `locale_service.h`.

### `char_skill.cpp`

*   **Amaç:** `CHARACTER` sınıfının becerilerle ilgili tüm fonksiyonlarını uygular. Bu, beceri öğrenme, seviye atlatma (puan, kitap, görev ile), beceri sıfırlama, beceri gücünü ve etkilerini hesaplama, beceri kullanma (bekleme süresi, SP maliyeti, hedefleme, alan etkileri), beceri gruplarını yönetme, pasif beceri etkilerini uygulama ve özel beceri mekaniklerini (örneğin, Zincirleme Şimşek, Görünmezlik, at becerileri, canavar becerileri) içerir. Kısacası, karakterlerin aktif ve pasif yeteneklerinin tüm yaşam döngüsünü ve etkileşimlerini yönetir.
*   **Temel İşlev Grupları ve Önemli Fonksiyonlar:**
    *   **Beceri Öğrenme ve Seviye Yönetimi:**
        *   `IsLearnableSkill(DWORD dwSkillVnum)`: Bir becerinin öğrenilip öğrenilemeyeceğini kontrol eder (sınıf, seviye, beceri grubu, öncül beceri).
        *   `LearnSkillByBook(DWORD dwSkillVnum, BYTE bProb)`: Beceri kitabıyla seviye atlatmayı yönetir. Tecrübe puanı gereksinimi, bekleme süresi (`GetSkillNextReadTime`), başarı olasılığı (veya okunan kitap sayısı), özel bonuslar (`AFFECT_SKILL_BOOK_BONUS`) gibi faktörleri içerir.
        *   `LearnGrandMasterSkill(DWORD dwSkillVnum)`: Ruh Taşı ile Grand Master seviyesine yükseltmeyi yönetir. Benzer şekilde bekleme süresi, kitap sayısı gereksinimi ve başarı olasılığı içerir.
        *   `SkillLevelUp(DWORD dwVnum, BYTE bMethod)`: Becerinin seviyesini 1 artırır. Yönteme (puan, kitap, görev) göre kontroller yapar, beceri puanını düşürür (gerekirse), seviyeyi ve master türünü (`SetSkillLevel`) günceller. Belirli seviyelerde (17-19, 20-29, 30-39) şansa bağlı olarak direkt Master/Grand Master/Perfect Master yapma mekanizması içerir.
        *   `SkillLevelDown(DWORD dwVnum)`: Becerinin seviyesini 1 düşürür (sadece normal seviyeler için), beceri puanını geri verir.
        *   `ResetSkill()` / `ClearSubSkill()` / `ResetOneSkill(DWORD dwVnum)`: Tüm aktif/pasif becerileri veya tek bir beceriyi sıfırlayarak harcanan puanları geri verir.
        *   `SetSkillGroup(BYTE bSkillGroup)`: Karakterin aktif beceri grubunu değiştirir.
        *   `SkillLevelPacket()`: Mevcut beceri seviyelerini istemciye gönderir (`HEADER_GC_SKILL_LEVEL`).
    *   **Beceri Bilgisi ve Gücü:**
        *   `GetSkillLevel(DWORD dwVnum)`: Belirtilen becerinin mevcut seviyesini döndürür.
        *   `GetSkillMasterType(DWORD dwVnum)`: Becerinin ustalık seviyesini (Normal, Master, GM, P) döndürür.
        *   `GetUsedSkillMasterType(DWORD dwVnum)`: Becerinin kullanım anındaki efektif ustalık seviyesini döndürür (Ruh Taşı kullanılmadıysa GM yerine M dönebilir).
        *   `GetSkillPower(DWORD dwVnum, BYTE bLevel)`: Becerinin efektif gücünü hesaplar (genellikle seviyeye göre % cinsinden). Dil yüzüğü gibi özel durumları ve lonca becerilerini ele alır.
        *   `GetSkillNextReadTime(DWORD dwVnum)` / `SetSkillNextReadTime(...)`: Beceri kitabı/Ruh Taşı okuma bekleme süresini yönetir.
    *   **Beceri Kullanımı ve Cooldown:**
        *   `CanUseSkill(DWORD dwSkillVnum)`: Karakterin mevcut sınıfı/grubu için bu beceriyi kullanıp kullanamayacağını kontrol eder. At becerileri ve diğer özel becerileri de içerir.
        *   `IsUsableSkillMotion(DWORD dwMotionIndex)`: Belirli bir animasyon indeksinin mevcut karakter tarafından kullanılıp kullanılamayacağını kontrol eder.
        *   `UseSkill(DWORD dwVnum, LPCHARACTER pkVictim, bool bUseGrandMaster)`: Bir beceriyi kullanmak için ana giriş noktasıdır. Çeşitli kontroller (durum, binek, SP/HP, cooldown) yapar, kaynakları tüketir, `m_SkillUseInfo`'yu güncelleyerek cooldown'u başlatır ve ilgili `ComputeSkill` fonksiyonunu çağırır. Toggle becerilerini (aç/kapa) ve Şarj becerilerini (örn. TANHWAN) yönetir.
        *   `ComputeCooltime(int time)`: Büyü Hızı'na göre bekleme süresini kısaltır.
        *   `TSkillUseInfo::UseSkill(...)` / `HitOnce(...)`: Belirli bir becerinin kullanıldığını işaretler, sonraki kullanılabilir zamanı ayarlar ve çoklu vuruşlu beceriler için vuruş sayacını yönetir.
        *   `CheckSkillHitCount(const BYTE SkillID, const VID TargetVID)`: İstemciden gelen vuruş paketlerinin geçerliliğini (bir hedefe maksimum kaç kez vurulabileceğini) kontrol ederek hileleri önlemeye çalışır.
    *   **Beceri Etkisi Hesaplama ve Uygulama:**
        *   `ComputeSkill(DWORD dwVnum, LPCHARACTER pkVictim, BYTE bSkillLevel)`: Bir becerinin belirli bir hedefe etkisini hesaplar ve uygular. `CSkillProto`'dan formülleri alır, `SetPolyVarForAttack` ile değişkenleri (saldırı gücü, statlar vb.) ayarlar, `CPoly::Eval` ile sonuçları (hasar, süre, miktar) hesaplar ve uygular (doğrudan `PointChange` veya `AddAffect` ile). Alan etkili beceriler için `FuncSplashDamage` veya `FuncSplashAffect` kullanır. Özel durumları (örn. MUYEONG, binek becerileri, kendi kendine kullanılanlar) ele alır.
        *   `ComputeSkillAtPosition(DWORD dwVnum, const PIXEL_POSITION& posTarget, BYTE bSkillLevel)`: Bir becerinin doğrudan bir konuma uygulanmasını sağlar (genellikle alan etkili canavar becerileri için). `FuncSplashDamage` veya `FuncSplashAffect` kullanarak o konum etrafındaki hedeflere etki eder.
        *   `ComputePassiveSkill(DWORD dwVnum)`: Pasif becerilerin bonuslarını (`ApplyPoint`) hesaplar ve uygular.
        *   `SetPolyVarForAttack(LPCHARACTER ch, CSkillProto* pkSk, LPITEM pkWeapon)`: Beceri formüllerinde kullanılacak `wep`, `mtk`, `mwep` gibi değişkenleri karakterin silahına veya canavarın saldırı değerlerine göre ayarlar.
        *   `FuncSplashDamage`: Alan etkili hasar becerileri için functor. Belirlenen alan içindeki geçerli hedeflere hasar hesaplar (`CalcBattleDamage`, dirençler, kritik vb. dahil) ve uygular. Vuruş sayacı (`m_pInfo->HitOnce`), statü etkileri (sersemletme, yavaşlatma, zehir vb. - `SkillAttackAffect`), knockback/crush gibi mekanikleri içerir. Zincirleme Şimşek için sonraki hedefi bulup event tetikler. HP/SP emme uygular.
        *   `FuncSplashAffect`: Alan etkili buff/debuff becerileri için functor. Belirlenen alan içindeki geçerli hedeflere `AddAffect` ile etki uygular.
    *   **Özel Beceri Mekanikleri:**
        *   Zincirleme Şimşek (`ChainLightningEvent`, `GetChainLightningMaxCount`): Bir hedeften diğerine seken şimşek saldırısını event tabanlı olarak yönetir.
        *   Görünmezlik (`SetAffectedEunhyung`, `SKILL_EUNHYUNG`, `RemoveInvisibleVictim`): Görünmezlik durumunu ve canavarların hedef kaybetme mantığını yönetir.
        *   Hava Kılıcı (`skill_muyoung_event`, `StartMuyeongEvent`, `StopMuyeongEvent`): Periyodik olarak yakındaki hedeflere saldıran event'i yönetir.
        *   Geri Dönüş (`skill_gwihwan_event`): Başarı şansına göre oyuncuyu imparatorluk başlangıç noktasına ışınlayan event'i yönetir.
        *   At Becerileri (`CanUseHorseSkill`): At üzerindeyken kullanılabilecek becerileri kontrol eder ve uygular (genellikle `ComputeSkill` içinde).
    *   **Canavar Becerileri (`UseMobSkill`, `CanUseMobSkill`, `GetMobSkill`, `mob_skill_hit_event`):** Canavarların beceri kullanma mantığını yönetir. Cooldown (`m_adwMobSkillCooltime`), kullanım koşulları ve `ComputeSkillAtPosition` aracılığıyla etkilerin uygulanmasını içerir. Gecikmeli vuruşlar için event kullanır.
*   **Çalışma Prensibi:** Beceriler `skill_proto`'dan yüklenen tanımlara göre çalışır. Öğrenme/seviye atlama karakterin `m_pSkillLevels` dizisini günceller. Beceri kullanıldığında (`UseSkill`), SP/HP maliyeti düşülür, cooldown (`m_SkillUseInfo`) başlatılır ve `ComputeSkill` çağrılır. `ComputeSkill`, `CSkillProto`'daki formülleri (`CPoly`) kullanarak hasar/etki miktarını/süresini hesaplar ve hedefe uygular (`PointChange`, `AddAffect`, `Damage`). Alan etkili beceriler `SECTOR::ForEachAround` ve functor'lar (`FuncSplashDamage`/`Affect`) aracılığıyla çalışır. Pasif beceriler `ComputePassiveSkill` ile doğrudan statüleri etkiler. Özel beceriler ve canavar becerileri genellikle event'ler aracılığıyla yönetilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `item.h`, `item_manager.h`, `affect.h`, `battle.h`, `packet.h`, `desc.h`, `desc_manager.h`, `sectree_manager.h`, `mob_manager.h`, `party.h`, `buffer_manager.h`, `guild.h`, `log.h`, `unique_item.h`, `questmanager.h`, `utils.h`, `constants.h`, `start_position.h`, `BlueDragon.h` (varsa), `ShipDefense.h` (varsa) ve diğer potansiyel sistem header'ları. (Linter hataları göz ardı edilmiştir).

### `char_state.cpp`

*   **Amaç:** `CHARACTER` sınıfının durum makinesi (state machine) mantığını uygular. Karakterin içinde bulunduğu duruma (Boşta, Hareket Halinde, Savaşta, Bayrak Taşıma/Bırakma, At) göre davranışlarını belirler ve durumlar arası geçişleri yönetir. Bu dosya, NPC ve canavarların yapay zekasının temelini oluşturur ve oyuncu karakterlerinin bazı otomatik davranışlarını (örn. hareket tamamlama, stamina yönetimi) kontrol eder.
*   **Temel İşlevler ve Durumlar:**
    *   **Durum Yönetimi (`GotoState`, `UpdateStateMachine`, `m_stateIdle`, `m_stateMove`, `m_stateBattle` vb.):** Karakterin mevcut durumunu değiştirir (`GotoState`) ve oyun döngüsünde `UpdateStateMachine` ile mevcut durumun mantığını çalıştırır. Ana durumlar `Idle`, `Move` ve `Battle`'dır. Ayrıca özel durumlar (`Flag`, `FlagBase`, `Horse`) bulunur.
    *   **Boşta Durumu (`StateIdle`, `__StateIdle_Stone`, `__StateIdle_NPC`, `__StateIdle_Monster`):**
        *   Karakterin yapacak bir şeyi olmadığında çalışır.
        *   **Taşlar (`__StateIdle_Stone`):** HP'si azaldıkça belirli aralıklarda canavar grupları spawn eder (`CHARACTER_MANAGER::instance().SpawnGroup`).
        *   **NPC'ler (`__StateIdle_NPC`):** Rastgele gezinme (`Goto`), takip etme (`Follow` - eğer koruduğu biri varsa), koruma NPC'lerinin (`IsGuardNPC`) yakındaki düşman canavarları araması (`FuncFindGuardVictim`) ve saldırması, Pet/Binek davranışları (genellikle ayrı yönetilir), özel event NPC'leri (örn. Noel Baba - `xmas::MOB_SANTA_VNUM`) için özel mantıklar içerir.
        *   **Canavarlar (`__StateIdle_Monster`):** Hedef arama (`FindVictim`, `IsAggressive` kontrolü), hedef yoksa veya agresif değilse rastgele gezinme (`Goto`), koruduğu karakteri (`GetProtege`) takip etme, korkak (`IsCoward`) ise kaçma (`CowardEscape`) mantığını içerir.
    *   **Hareket Durumu (`StateMove`):**
        *   Karakter bir noktaya (`m_posDest`) hareket ederken çalışır.
        *   Geçen süreye göre pozisyonu günceller (`Move`).
        *   Oyuncular için stamina tüketimini (`PointChange(POINT_STAMINA, ...)`), takas mesafesini kontrol eder.
        *   Canavarlar için hedefi takip etme mantığını (`__CHARACTER_GotoNearTarget`) içerebilir.
        *   Hedefe ulaşıldığında (`bMovementFinished`) Boşta (`StateIdle`) veya Savaş (`StateBattle`) durumuna geçiş yapar.
    *   **Savaş Durumu (`StateBattle`):**
        *   Karakterin bir hedefi (`GetVictim`) olduğunda çalışır.
        *   Hedef kontrolü yapar (ölü mü, mesafesi uygun mu).
        *   Korkak (`IsCoward`) ise kaçmaya çalışır.
        *   Agresif canavarlar hedef öldüğünde yeni hedef arar (`FindVictim`).
        *   Canavar çağırma yeteneği (`CanSummonMonster`) varsa ve koşullar uygunsa canavar çağırır (`SpawnMobRange`, `CPartyManager`).
        *   Hedef menzil dışındaysa yaklaşmaya çalışır (`__CHARACTER_GotoNearTarget`).
        *   Saldırı bekleme süresini (`m_dwLastAttackTime`, `CalculateDuration`) kontrol eder.
        *   Berserker (`IsBerserker`), GodSpeeder (`IsGodSpeeder`) gibi özel AI flag'lerine göre buffları aktif eder (`SetBerserk`, `SetGodSpeed`).
        *   Canavar becerisi (`HasMobSkill`, `CanUseMobSkill`, `UseMobSkill`) kullanma mantığını içerir.
        *   Normal saldırı yapar (`Attack`, `SendMovePacket(FUNC_ATTACK, ...)`).
        *   Özel boss mantıklarını (örn. Mavi Ejderha - `BlueDragon_StateBattle`) çağırabilir.
    *   **Bayrak Durumları (`StateFlag`, `StateFlagBase`):**
        *   Lonca savaşlarındaki bayrak taşıma/bırakma mekaniği ile ilgilidir.
        *   `StateFlag` (Bayrak NPC'si): Yakındaki uygun oyuncuya bayrak affect'ini verir (`AFFECT_WAR_FLAG`, `AFF_WAR_FLAG*`), haritadan bayrağı kaldırır.
        *   `StateFlagBase` (Bayrak Bırakma Alanı NPC'si): Yakındaki bayrak taşıyan düşman oyuncuyu algılar, skoru günceller (`SendGuildWarScore`), bayrağı sıfırlar (`pMap->ResetFlag()`).
    *   **At Durumu (`StateHorse`):**
        *   At karakterinin (`IsHorse()`) davranışını yönetir.
        *   Sürekli olarak binicisini (`GetRider`) belirli bir mesafede takip eder (`Follow`).
        *   Binici yoksa kendini yok eder (`M2_DESTROY_CHARACTER`).
    *   **Yapay Zeka Flag'leri (`IsAggressive`, `IsCoward`, `IsBerserker`, `IsNoAttack*` vb.):** Karakterin davranışlarını (saldırganlık, korkaklık, belirli imparatorluklara saldırmama vb.) belirleyen flag'leri kontrol eder ve ayarlar.
*   **Çalışma Prensibi:** Karakterin mevcut durumu `m_eCurrentState` değişkeninde tutulur. Oyun döngüsünde `UpdateStateMachine` çağrılır, bu da mevcut duruma karşılık gelen `State*` fonksiyonunu çalıştırır. Bu fonksiyonlar, karakterin çevresini (hedef, mesafe, HP durumu vb.) ve AI flag'lerini kontrol ederek uygun eylemi (hareket etme, saldırma, bekleme, beceri kullanma) gerçekleştirir ve bir sonraki durum geçişini veya mevcut durumun süresini (`m_dwStateDuration`) ayarlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `utils.h`, `vector.h`, `char.h`, `battle.h`, `char_manager.h`, `packet.h`, `motion.h`, `party.h`, `affect.h`, `buffer_manager.h`, `questmanager.h`, `p2p.h`, `item_manager.h`, `mob_manager.h`, `exchange.h`, `sectree_manager.h`, `xmas_event.h`, `guild_manager.h`, `war_map.h`, `locale_service.h`, `BlueDragon.h` (varsa), `MeleyLair.h` (varsa), `TempleOchao.h` (varsa), `unique_mob.h`.

### `char.cpp` - Bölüm 1: Yaratma, Yok Etme ve Temel Özellikler

*   **Amaç:** Bu bölüm, `CHARACTER` sınıfının temel yaşam döngüsünü (oluşturma, yok etme) ve bir karakterin kimliğini tanımlayan ana özellikleri (isim, seviye, ırk, sınıf, imparatorluk) ayarlayan fonksiyonları içerir.
*   **Önemli Fonksiyonlar:**
    *   **`CHARACTER::CHARACTER()` / `CHARACTER::~CHARACTER()`:** Yapıcı ve yıkıcı metotlar. Karakter nesnesi oluşturulduğunda değişkenleri (işaretçileri null yapma, varsayılan değerleri atama) başlatır ve nesne yok edildiğinde ayrılan belleği (örneğin, `m_pSkillLevels`, `m_pAffect`, `m_pPlayerSlots`) serbest bırakır, event'leri iptal eder ve karakteri temizler (`Destroy` çağrılır).
    *   **`Create(const char* c_pszName, DWORD vid, bool isPC)`:** Karakter nesnesinin temel başlatılmasını yapar. Verilen VID'yi atar, ismi kopyalar, PC veya NPC olmasına göre ilk kurulumları yapar (örneğin, PC için `m_pSkillLevels` ve `m_pAffect` belleklerini ayırır).
    *   **`Destroy()`:** Karakteri oyun dünyasından tamamen kaldırmadan önceki temizlik işlemlerini yapar. Binekten iner (`StopRiding`), ticareti/pazarı kapatır (`CloseMyShop`, `GetExchange()->Cancel`), partiden/loncadan ayrılır (`SetParty(NULL)`, `guild->LogoutMember`), event'leri (`event_cancel`) durdurur, sahip olduğu pointerları (örneğin, `m_pkChrTarget`, `m_pkChrShopOwner`) temizler ve karakteri `CHARACTER_MANAGER`'dan kaldırır (`CHARACTER_MANAGER::instance().UnregisterPC` veya `UnregisterNPC`). Bellek temizliği genellikle yıkıcı (`~CHARACTER()`) tarafından yapılır.
    *   **`SetRace(BYTE race)`:** Karakterin ırkını (VNUM'unu) ayarlar.
    *   **`SetJob(unsigned job)`:** Karakterin sınıfını ayarlar. Genellikle ırk numarasından türetilir (`RaceToJob`).
    *   **`SetLevel(BYTE level)`:** Karakterin seviyesini ayarlar. Seviye değişimine bağlı olarak puanları yeniden hesaplar (`ComputePoints`) ve istemciye bildirir (`UpdatePacket`).
    *   **`SetEmpire(BYTE bEmpire)`:** Karakterin ait olduğu imparatorluğu ayarlar.
    *   **`SetName(const char *name)`:** Karakterin ismini ayarlar. İsim değişikliği paketini (`HEADER_GC_CHARACTER_UPDATE`) gönderir.
    *   **`Decoded()`:** İstemcinin karakter verilerini tamamen aldığını ve karakterin oyun dünyasında tamamen aktif hale geldiğini işaretlemek için kullanılır (`SetPlayerProto(true)`).

### `char.cpp` - Bölüm 2: Konum, Görünürlük ve Hareket

*   **Amaç:** Bu bölüm, karakterin oyun dünyasındaki fiziksel varlığını yöneten fonksiyonları içerir. Karakteri görünür kılma/gizleme, pozisyonunu (ayakta, oturuyor) ayarlama, hareket ettirme (yürüme/koşma), anında ışınlama ve bu değişiklikleri diğer oyunculara bildirme gibi işlevler burada bulunur.
*   **Önemli Fonksiyonlar:**
    *   **`Show(long lMapIndex, long x, long y, ...)`:** Karakteri belirtilen harita ve koordinatlarda oyun dünyasına ekler. Karakteri `SECTREE_MANAGER`'a kaydeder, görünürlük listelerini (`m_known_list`) günceller, etraftaki karakterlere ekleme paketini (`EncodeInsertPacket`) gönderir ve (isteğe bağlı) spawn animasyonunu tetikler. Başlangıç HP/SP gibi değerleri de ayarlayabilir.
    *   **`Hide()`:** Karakteri oyun dünyasından gizler. `SECTREE_MANAGER`'dan kaydını siler, görünürlük listelerini temizler ve etraftaki karakterlere kaldırma paketini (`EncodeRemovePacket`) gönderir.
    *   **`SetPosition(int pos)`:** Karakterin duruş pozisyonunu (ayakta, oturuyor vb. - `POS_STANDING`, `POS_SITTING`) ayarlar ve pozisyon değişikliği paketini (`HEADER_GC_CHARACTER_POSITION`) gönderir.
    *   **`SetRotation(float fRot)`**, **`SetRotationToXY(long x, long y)`:** Karakterin baktığı yönü ayarlar. Genellikle hareket veya saldırı sırasında kullanılır ve `SendMovePacket` ile diğer oyunculara bildirilir.
    *   **`Goto(long x, long y)`:** Karakteri hedef koordinatlara doğru hareket ettirmek için hareket emrini başlatır. Başlangıç (`m_posStart`) ve hedef (`m_posDest`) pozisyonlarını kaydeder, hareket süresini (`m_dwMoveDuration`) karakterin hızına göre hesaplar, karakteri `StateMove` durumuna geçirir ve ilk hareket paketini (`SendMovePacket`) gönderir. Yol üzerinde engel olup olmadığını kontrol eder (`SECTREE_MANAGER::instance().IsMovablePosition`).
    *   **`Move(long x, long y)`:** Karakterin anlık koordinatlarını günceller. `StateMove` durumunda periyodik olarak çağrılır. Karakterin `SECTREE` içindeki yerini günceller (`UpdateSectree`).
    *   **`WarpSet(long x, long y, ...)`:** Karakteri anında belirtilen koordinatlara ışınlar. Gerekirse harita değiştirme (`StartWarp`), kanal değiştirme (`MoveChannel`) veya özel ışınlanma (`PIXEL_POSITION_PTR`) mantığını tetikler. Işınlanma sonrası görünürlüğü günceller.
    *   **`SendMovePacket(BYTE bFunc, BYTE bArg, ...)`:** Karakterin hareket bilgilerini (başlama, durma, yönelme vb.) içeren `HEADER_GC_MOVE` paketini oluşturur ve `PacketAround` ile etraftaki oyunculara gönderir.
    *   **`EncodeInsertPacket(LPENTITY entity)`**, **`EncodeRemovePacket(LPENTITY entity)`:** Görüş alanına giren veya çıkan bir varlık (`entity`) için `HEADER_GC_CHARACTER_ADD` veya `HEADER_GC_CHARACTER_REMOVE` paketlerini oluşturur ve doğrudan o varlığın sahibine (`entity->GetDesc()`) gönderir. `EncodeInsertPacket`, karakterin VNUM, isim, pozisyon, görünüm (eşyalar, saç, affectler, lonca vb.), hız, durum gibi birçok bilgisini içerir.
    *   **`UpdateSectree()`:** Karakterin `SECTREE` içindeki konumunu günceller. Bu işlem sırasında, yeni girilen sektörlerdeki varlıklarla veya eski sektörlerden çıkan varlıklarla görünürlük (`AddView`/`RemoveView`) güncellemelerini tetikler.

### `char.cpp` - Bölüm 3: Puan ve Statü Yönetimi

*   **Amaç:** Bu bölüm, karakterin temel ve anlık istatistiklerini (HP, SP, EXP, Seviye, Statüler - STR, DEX, CON, INT), saldırı/savunma değerlerini, hızlarını ve diğer birçok sayısal özelliğini yöneten fonksiyonları içerir. Puanları okuma, değiştirme, bonusları uygulama/kaldırma ve bu değişiklikleri hesaplayıp bildirme işlemleri burada yapılır.
*   **Önemli Fonksiyonlar:**
    *   **Puan Okuma (`GetPoint`, `GetLimitPoint`, `GetRealPoint`, `GetPolymorphPoint`, `GetMaxHP`, `GetMaxSP`, `GetHPPct`, `GetSPPct`, `GetLevel`, `GetAlignment`, `GetRealAlignment`, `GetExp`, `GetGold`, `GetCheque` vb.):** Karakterin çeşitli anlık veya temel puan değerlerini döndürür.
        *   `GetPoint(type)`: En sık kullanılan fonksiyondur. Belirtilen tür (`POINT_*` sabitleri) için tüm bonuslar (statüler, eşyalar, beceriler, affect'ler, premium vb.) hesaplanmış **anlık efektif değeri** döndürür.
        *   `GetLimitPoint(type)`: Bir puanın ulaşabileceği maksimum değeri (genellikle `limit_value` tablosundan) döndürür. Hareket hızı, saldırı hızı gibi değerler için önemlidir.
        *   `GetRealPoint(type)`: Genellikle temel (bonussuz) değeri veya polimorf durumundaki orijinal değeri almak için kullanılır.
        *   `GetPolymorphPoint(type)`: Dönüşüm (polimorf) sırasında geçerli olan puan değerini döndürür.
    *   **Puan Ayarlama ve Değiştirme (`SetPoint`, `PointChange`, `SetLevel`, `SetExp`, `GiveExp`, `GiveGold`, `PayGold`):**
        *   `SetPoint(type, val)`: Belirtilen puan türünün temel değerini doğrudan ayarlar. Genellikle karakter yaratılırken veya seviye atlandığında statü puanları için kullanılır.
        *   `PointChange(type, amount, bAmount, bBroadcast)`: En kritik fonksiyonlardan biridir. Belirtilen puan türünü `amount` kadar artırır veya azaltır.
            *   Sınırları kontrol eder (`GetLimitPoint`).
            *   Ölüm/canlanma gibi özel durumları tetikleyebilir (HP için).
            *   Değişikliğin diğer puanları nasıl etkileyeceğini hesaplamak için `ComputePoints()`'i çağırır.
            *   Değişikliği oyuncuya ve/veya etraftakilere bildirmek için `UpdatePointsPacket()`'i çağırır (`bBroadcast` parametresi).
            *   `bAmount` true ise `amount` mutlak değişim miktarıdır, false ise yeni değerdir (nadiren kullanılır).
        *   `SetLevel`, `SetExp`, `GiveExp`: Seviye ve tecrübe puanını yönetir. `GiveExp` tecrübe verirken seviye atlama kontrolü yapar.
        *   `GiveGold`, `PayGold`: Yang miktarını yönetir.
    *   **Bonus Uygulama/Kaldırma (`ApplyPoint`, `RemovePoint`):**
        *   Eşya efsunları (`ApplyTypes`), beceri etkileri (`affect.h`), polimorf veya diğer sistemlerden gelen geçici veya kalıcı bonusları karakterin puanlarına uygular (`ApplyPoint`) veya geri alır (`RemovePoint`). Bu fonksiyonlar doğrudan `m_pointsInstant` yapısındaki ilgili bonus alanlarını günceller.
    *   **Hesaplama (`ComputePoints`):**
        *   Karakterin tüm anlık efektif puanlarını (`GetPoint` ile okunacak değerleri) hesaplayan merkezi fonksiyondur. Statülerden (`points.st`, `ht`, `dx`, `iq`), eşya bonuslarından (`m_pointsInstant.iItemApply*`), beceri etkilerinden (`m_pointsInstant.pSkills`), polimorf durumundan (`m_pointsInstant.dwPolymorphRace`), affect'lerden ve diğer özel durumlardan (binek, evlilik, premium vb.) gelen tüm katkıları toplayarak nihai değerleri (`m_pointsInstant.*`) hesaplar. Genellikle `PointChange`, `EquipItem`, `UnequipItem`, `AddAffect`, `RemoveAffect` gibi fonksiyonlardan sonra çağrılır.
    *   **Bildirim (`UpdatePointsPacket`, `ChatPacket`, `EffectPacket`, `SpecificEffectPacket`):**
        *   `UpdatePointsPacket`: Belirli bir puanın (`POINT_*`) veya tüm puanların (`POINT_NONE`) yeni değerini istemciye `HEADER_GC_POINT_CHANGE` paketi ile gönderir.
        *   `ChatPacket`: Oyuncuya sistem mesajları göndermek için kullanılır (`HEADER_GC_CHAT`).
        *   `EffectPacket`: Karakter üzerinde standart bir görsel efekt (örn. seviye atlama, zehirlenme - `SE_*` sabitleri) oynatmak için `HEADER_GC_EFFECT` paketi gönderir.
        *   `SpecificEffectPacket`: Dosya adı belirtilen özel bir görsel efekti oynatmak için `HEADER_GC_SPECIFIC_EFFECT` paketi gönderir.
    *   **Diğer (`UpdateAlignment`, `ResetPlayTime`, `ResetPoint`, `ComputeRefineFee` vb.):**
        *   `UpdateAlignment`: Eğilim puanını yönetir, periyodik olarak günceller ve affect uygular.
        *   `ResetPlayTime`: Oyuncunun oynama süresini sıfırlar (genellikle Çin yorgunluk sistemi için).
        *   `ResetPoint`: Karakterin statülerini ve becerilerini sıfırlar (Seviye 1 durumuna döner, eşyaları çıkarır).
        *   `ComputeRefineFee`: Eşya basma ücretini hesaplar.

### `char.cpp` - Bölüm 4: Etkileşimler ve Sosyal Sistemler

*   **Amaç:** Bu bölüm, karakterin diğer karakterlerle veya oyun dünyasıyla doğrudan etkileşime girdiği (tıklama, ticaret, pazar kurma) ve sosyal yapılarla (parti, lonca, evlilik) olan ilişkilerini yöneten fonksiyonları kapsar.
*   **Önemli Fonksiyonlar:**
    *   **Doğrudan Etkileşimler:**
        *   `OnClick(LPCHARACTER pkChrCauser)`: Başka bir karakter (`pkChrCauser`) bu karaktere tıkladığında tetiklenir. Hedef NPC ise ilgili görev (`quest::CQuestManager::Click`), dükkan (`SetShop`) veya özel NPC fonksiyonunu (demirci - `SetRefineNPC`, depo - `OpenSafebox`) çağırır. Hedef oyuncu ise ve özel pazar (`GetMyShop`) açıksa, pazarı açar. Canavar veya başka bir oyuncuysa, hedef olarak ayarlar (`SetTarget`).
        *   `Exchange(LPCHARACTER victim)`: Başka bir oyuncuyla takas işlemi başlatır (`exchange->Open`).
        *   `OpenMyShop(const char* c_pszSign, TShopItemTable* pTable, WORD wItemCount)`: Oyuncunun özel pazarını (offline shop) açar (`CShopManager::instance().CreateMyShop`). Gerekli kontrolleri yapar (isim, eşya sayısı).
        *   `BuildPrivateShop(const char* c_szTitle, ...)`: Oyuncunun yere kurduğu tezgâhı (`CPrivateShop`) oluşturur.
        *   `mining(LPCHARACTER chLoad)`: Madencilik eylemini yönetir. Kazma kontrolü yapar, cevher damarını (`chLoad`) hedefler, başarı/başarısızlık durumunu işler ve event'i (`kill_ore_load_event`) yönetir.
        *   `fishing(uint8_t fg_success)`: Balık tutma eylemini yönetir. Olta/yem kontrolü yapar, `fishing_event`'i başlatır/durdurur, başarı/başarısızlık durumunu ve yakalanan balığı/eşyayı işler.
    *   **Parti Yönetimi:**
        *   `SetParty(LPPARTY pkParty)`: Karakterin parti üyeliğini ayarlar veya kaldırır. Parti üyelerine güncelleme paketi gönderir.
        *   `RequestToParty(LPCHARACTER leader)`: Parti liderine katılma isteği gönderir (`party_request_event`).
        *   `AcceptToParty(LPCHARACTER member)`: Partiye katılma isteğini kabul eder (`CPartyManager::instance().AddMember`).
        *   `DenyToParty(LPCHARACTER member)`: Partiye katılma isteğini reddeder.
        *   `PartyInvite(LPCHARACTER pchInvitee)`: Başka bir oyuncuyu partiye davet eder (`party_invite_event`).
        *   `PartyInviteAccept(LPCHARACTER pchInvitee)`: Parti davetini kabul eder.
        *   `PartyInviteDeny(DWORD dwPID)`: Parti davetini reddeder.
        *   `IsPartyJoinableCondition(...)`, `IsPartyJoinableMutableCondition(...)`: Bir oyuncunun partiye katılıp katılamayacağını kontrol eden statik ve dinamik koşulları (seviye farkı, harita, zindan durumu vb.) değerlendirir.
    *   **Lonca Yönetimi:**
        *   `SetGuild(CGuild* pGuild)`: Karakterin lonca üyeliğini ayarlar veya kaldırır. Lonca adını/rütbesini etraftakilere bildirir (`SendGuildName`).
        *   `SendGuildName(DWORD dwGuildID)`: Karakterin lonca adını `HEADER_GC_GUILD` paketi ile istemciye gönderir. Loncasızsa veya lonca ID'si 0 ise boş isim gönderir.
        *   `CanAttack(LPCHARACTER pkTarget, bool bCheckParty)`: Lonca savaşı, imparatorluk, PK modu gibi kurallara göre hedefe saldırılıp saldırılamayacağını kontrol eder (Bu fonksiyonun bir kısmı `char_battle.cpp` içinde olabilir, ancak lonca kontrolü burada yer alabilir).
    *   **Evlilik Sistemi:**
        *   `SetMarryPartner(LPCHARACTER ch)`: Karakterin evli olduğu partneri ayarlar.
        *   `GetMarryPartner()`: Evli olunan karakteri döndürür.
        *   `GetMarriageBonus(DWORD dwItemVnum, bool bSum)`: Evlilik eşyalarından (Yüzük vb.) gelen bonusları hesaplar. Eşler yakındaysa veya harita uygunsa bonusları toplar.
        *   `NearMarry()`: Evlilik partnerinin yakında olup olmadığını kontrol eder.

### `char.cpp` - Bölüm 5: Eventler, Zamanlayıcılar ve Durum Makinesi Kontrolü

*   **Amaç:** Bu bölüm, karakterin zamanla tetiklenen eylemlerini (HP/SP yenilenmesi, affect süreleri, balık tutma, madencilik, periyodik kaydetme) ve durum makinesinin kontrolünü yöneten fonksiyonları içerir. Event mekanizması, belirli süreler sonunda veya periyodik olarak fonksiyonların çalıştırılmasını sağlar.
*   **Önemli Fonksiyonlar:**
    *   **Event Başlatma/Durdurma/Yönetme:**
        *   `StartAffectEvent()`: HP/SP yenilenmesi ve affect sürelerini takip eden ana `recovery_event`'i başlatır.
        *   `StopAffectEvent()`: `recovery_event`'i durdurur.
        *   `StartSaveEvent()`: Periyodik karakter kaydetme (`save_event`) event'ini başlatır.
        *   `StopSaveEvent()`: `save_event`'i durdurur.
        *   `StartStaminaConsume()`, `StopStaminaConsume()`: Koşarken stamina tüketimini başlatan/durduran mekanizmayı kontrol eder (doğrudan event olmasa da zamanla ilişkilidir).
        *   `fishing_event()` (event fonksiyonu): Balık tutma süresini, başarı şansını ve sonuçlarını yönetir. `event_create` ile başlatılır, `event_cancel` ile durdurulur.
        *   `kill_ore_load_event()` (event fonksiyonu): Maden damarının (bir tür NPC) yok olma süresini yönetir.
        *   `recovery_event()` (event fonksiyonu): En önemli periyodik eventlerden biri. `UpdateAffect()`'i çağırarak affect sürelerini günceller, HP/SP yenilenmesini (`自然力`, `VitalityRecoveryPerPulse`) uygular, stamina yenilenmesini yapar, zehir/ateş hasarını uygular (ilgili eventleri tetikler), premium sürelerini kontrol eder, Çin yorgunluk sistemini günceller. Kendini periyodik olarak tekrar (`PASSES_PER_SEC(SECFUNC_PULSE_PER_SEC)`) çağırır.
        *   `save_event()` (event fonksiyonu): Karakter verilerini periyodik olarak veritabanına kaydeder (`Save()`). Kendini tekrar çağırır.
        *   `warp_npc_event()` (event fonksiyonu): Işınlayıcı NPC'lerin bekleme süresini yönetir.
        *   `check_speedhack_event()` (event fonksiyonu): Belirli aralıklarla hız hilesi kontrolü yapar.
        *   `destroy_when_idle_event()` (event fonksiyonu): Belirli bir süre boşta kalan NPC'leri (maden damarı gibi) yok eder.
        *   `party_request_event()`, `party_invite_event()`: Parti istek/davet sürelerini yönetir ve süresi dolduğunda isteği/daveti iptal eder.
        *   `move_channel_event()`: Kanal değiştirme işleminin gecikmesini yönetir ve süre dolduğunda asıl ışınlanmayı tetikler.
        *   `private_shop_warp_event()`: Özel pazara ışınlanma gecikmesini yönetir.
    *   **Durum Makinesi Kontrolü:**
        *   `StartStateMachine(int iNextPulse)`: Karakterin durum makinesini (`UpdateStateMachine`) belirli bir gecikmeyle (`passes_per_sec * iNextPulse / 100`) ilk kez çalışacak şekilde ayarlar. Genellikle karakter yaratıldığında veya haritaya girdiğinde çağrılır.
        *   `UpdateStateMachine(DWORD dwPulse)`: Oyun döngüsünde çağrılır ve karakterin mevcut durumuna (`m_eCurrentState`) göre ilgili `State*()` fonksiyonunu (örn. `StateIdle`, `StateMove`, `StateBattle`, `char_state.cpp` içinde tanımlı) çalıştırır. Bu fonksiyonlar karakterin o anki davranışını belirler.
        *   `SetNextStatePulse(int iNextPulse)`: `State*()` fonksiyonları içinden çağrılarak, bir sonraki durum makinesi güncellemesinin ne kadar süre sonra (`passes_per_sec * iNextPulse / 100`) çalışacağını ayarlar. Bu, durumun ne kadar süreceğini (`m_dwStateDuration`) belirler.
    *   **Genel Karakter Güncelleme:**
        *   `UpdateCharacter(DWORD dwPulse)`: Oyun döngüsünde her karakter için çağrılan ana güncelleme fonksiyonu. Durum makinesini (`UpdateStateMachine`), görünürlüğü (`UpdateSectree`), hareketleri (`UpdateTime`), saldırı zamanlamasını ve diğer periyodik kontrolleri (örn. `CheckSpeedHack`) tetikler. Bu fonksiyon, karakterin canlı ve dinamik olmasını sağlar.

### `char.cpp` - Bölüm 6: Özel Sistemler ve Diğer İşlevler

*   **Amaç:** Bu bölüm, karakterin belirli oyun sistemleriyle (Polimorf, Binek, Depo, Nesne Market, Kostümler, Aksesuar Sistemi, Özel Pazar vb.) etkileşimini sağlayan fonksiyonları ve çeşitli yardımcı/kontrol işlevlerini içerir.
*   **Önemli Fonksiyonlar:**
    *   **Polimorf ve Binek:**
        *   `SetPolymorph(DWORD dwRaceNum, bool bMaintainStat)`: Karakteri belirtilen VNUM'a dönüştürür. Görünümü (`UpdatePacket`), hızları ve (isteğe bağlı) statüleri günceller. Dönüşümü bitirmek için VNUM 0 ile çağrılır.
        *   `MountVnum(DWORD vnum)`: Karakterin üzerine bindiği bineğin VNUM'unu ayarlar. Görünümü günceller, at/binek karakterini yönetir (`HorseSummon`).
        *   `UnMount(bool bUnequipItem)`: Karakteri bindiği attan/binekten indirir. Binek VNUM'unu sıfırlar, at karakterini gösterir.
        *   `CanUseHorseSkill()`: At/binek üzerindeyken beceri kullanılıp kullanılamayacağını kontrol eder (genellikle `char_horse.cpp` ile bağlantılı).
        *   `SummonPetFromItem(LPITEM item)`: Eşyadan pet çağırma işlemini başlatır (`CPetSystem`).
    *   **Depo ve Nesne Market:**
        *   `LoadSafebox(int iSize, DWORD dwGold, int iItemCount, TPlayerItem* pItems)`: Oyuncunun depo bilgilerini yükler (`CSafebox`).
        *   `ChangeSafeboxSize(BYTE bSize)`: Depo boyutunu değiştirir.
        *   `LoadMall(int iItemCount, TPlayerItem* pItems)`: Nesne Market deposu bilgilerini yükler (`CMall`).
        *   `SetSafeboxSize(int iSize)`: Depo boyutunu ayarlar.
    *   **Kostümler ve Aksesuarlar (Acce):**
        *   `SetPart(BYTE bPartPos, WORD wVal)`: Karakterin görünümünü etkileyen parçaları (saç, zırh görünümü vb. - `PART_*` sabitleri) ayarlar.
        *   `GetPart(BYTE bPartPos)`: Belirtilen parçanın mevcut değerini döndürür.
        *   `GetOriginalPart(BYTE bPartPos)`: Dönüşüm/kostüm gibi etkiler olmadan orijinal parça değerini döndürür.
        *   `OpenAcce(bool bCombination)`: Aksesuar (Acce) Efsunlama/Emme penceresini açar.
        *   `AcceIsSameGrade`, `GetAcceCombinePrice`, `AddAcceMaterial`, `RemoveAcceMaterial`, `AcceCombine`, `AcceAbsorb`, `CleanAcceAttr`: Aksesuar sisteminin işlevlerini yönetir (malzeme ekleme/çıkarma, birleştirme, emme, temizleme).
        *   `Set*CostumeHidden(bool hidden)`: Belirli kostüm türlerinin (Beden, Saç, Aksesuar, Silah, Aura) görünürlüğünü ayarlar.
        *   `MoveCostumeAttr`, `SetBonusTransfer`: Kostüm efsun transferi sistemini yönetir.
    *   **Özel Dükkanlar ve Pazarlar:**
        *   `SetShop(LPSHOP pkShop)`: Karakterin etkileşimde olduğu NPC dükkanını ayarlar.
        *   `GetShop()`: Mevcut dükkanı döndürür.
        *   `GetMyShop()`: Oyuncunun özel pazarını (offline shop) döndürür.
        *   `GetPrivateShop()`: Oyuncunun yere kurduğu tezgâhı döndürür.
        *   `ClosePrivateShopPanel`, `RemovePrivateShopItem`, `GetPrivateShopItem`, `ChangePrivateShopItemPrice`, `ChangePrivateShopItemPos`, `WithdrawPrivateShop`: Özel Pazar/Tezgah yönetimi fonksiyonları.
        *   `OpenShopSearch`: Pazar Arama sistemini açar.
        *   `SetPremiumPrivateShopBonus`: Premium Pazar bonusunu ayarlar.
        *   `GemShopBuy`, `OpenGemShop`, `CreateGem`: Gaya Mağazası sistemi ile etkileşimleri yönetir (`char_gem.cpp` ile bağlantılı).
    *   **Çeşitli Sistemler ve Yardımcı Fonksiyonlar:**
        *   `SetQuestNPCID`, `SetQuestItemPtr`, `GetQuestFlag`, `SetQuestFlag`: Görev sistemi (`questmanager.h`) ile etkileşim için arayüz sağlar.
        *   `IsHack(bool bSendMsg, bool bCheckShopOwner, int limittime)`: Çeşitli hile kontrolleri yapar (saldırı hızı, hareket hızı, işlem zamanlaması vb.).
        *   `ChangeLanguage(BYTE bLanguage)`: Karakterin oyun içi dilini değiştirir.
        *   `IsRaceFlag(DWORD dwBit)`: Karakterin ırkının belirli bir bayrağa (`RACE_FLAG_*`) sahip olup olmadığını kontrol eder.
        *   `GetMobElement(BYTE bElement)`: Canavarın belirli bir elemente karşı direncini/gücünü döndürür.
        *   `ConfirmWithMsg`: Oyuncuya onay penceresi gönderir (Evet/Hayır).
        *   `GetPremiumRemainSeconds`: Premium üyeliklerin (EXP, Yang, Eşya Düşürme) kalan süresini döndürür.
        *   `WarpToPID`: Başka bir oyuncunun yanına ışınlanır.
        *   `PreventTradeWindow`: Ticaret, Pazar, Depo gibi pencerelerin açılmasını engelleyen durumları kontrol eder.
        *   `SetBlockMode`, `SetBlockModeForce`: Oyuncunun hareket/etkileşim engel modunu ayarlar.
        *   `SetComboSequence`, `SetLastComboTime`, `SetValidComboInterval`, `IncreaseComboHackCount`, `SkipComboAttackByTime`: Combo saldırı sistemi ve hile kontrolünü yönetir.
        *   `GetSkillPowerByLevel`: Verilen seviyeye göre beceri gücünü hesaplar.
        *   `IsInBlockedArea`: Karakterin girilmesi yasak bir bölgede olup olmadığını kontrol eder.
        *   `MoveChannel`, `StartMoveChannel`: Kanal değiştirme işlemlerini yönetir.
        *   `SetConqueror`: Fatih seviyesi durumunu ayarlar.
        *   `GetSungMaWill`: SungMa irade puanlarını döndürür.
        *   `ItemExpireUpdate`: Süreli eşyaların kalan süresini günceller.
        *   `SortSpecialInventoryItems`: Özel envanterleri (Beceri Kitabı, Yükseltme vb.) sıralar.
    *   **Senkronizasyon ve Görünürlük Listeleri:**
        *   `SetSyncOwner(LPCHARACTER ch, ...)`: Özellikle Pet/Binek gibi karakterlerin sahibini ayarlar ve senkronizasyon listesini (`m_syncOwnerList`) yönetir.
        *   `IsSyncOwner(LPCHARACTER ch)`: Verilen karakterin bu karakterin senkronizasyon sahibi olup olmadığını kontrol eder.

*Bu, `char.cpp` dosyasının ana işlev gruplarının bir özetidir. Dosya, burada listelenmeyen daha birçok küçük yardımcı fonksiyon ve özel durum içerebilir.*

### `check_server.cpp`

*   **Amaç:** `check_server.h`'de bildirilen `CheckServer` sınıfının statik üyelerini (`keys_` ve `fail_`) başlatır.
*   **Temel İşlevler/İçerik:**
    *   `keys_` vektörünü boş olarak başlatır.
    *   `fail_` boolean değişkenini varsayılan olarak `true` (başarısız) olarak başlatır.
*   **Bağlantılı Dosyalar:** `check_server.h`.

### `cipher.cpp`

*   **Amaç:** `cipher.h`'de bildirilen `Cipher` sınıfını ve yardımcı sınıfları uygular. Diffie-Hellman anahtar anlaşması yapar, paylaşılan sırra göre rastgele şifreleme algoritmaları seçer ve CTR modunda şifreleme/çözme işlemlerini gerçekleştirir.
*   **Temel İşlevler/İçerik:**
    *   **Anahtar Anlaşması (`DH2KeyAgreement`):** CryptoPP `DH2` kullanarak Unified Diffie-Hellman ile paylaşılan bir sır (`shared_`) oluşturur. RFC 5114 1024-bit grubu kullanılır.
    *   **Algoritma Seçimi (`BlockCipherAlgorithm`, `BlockCipherDetail`):**
        *   Paylaşılan sırra dayalı olarak rastgele iki farklı blok şifreleme algoritması (`RC6`, `MARS`, `Twofish` (varsayılan), `Serpent`, `CAST256`, `IDEA`, `3DES`, `Camellia`, `SEED`, `RC5`, `Blowfish`, `TEA`, `SHACAL2`) seçer.
        *   Seçilen algoritmalar için anahtarları ve IV'leri paylaşılan sırdan türetir.
    *   **`Cipher` Uygulaması:**
        *   `Prepare`: `DH2KeyAgreement::Prepare` çağırılır.
        *   `Activate`: `DH2KeyAgreement::Agree` çağrılır, başarılıysa `SetUp` çağrılır.
        *   `SetUp`: Algoritmaları seçer, anahtar/IV türetir ve `polarity`'ye göre birini `encoder_` (CTR modu şifreleyici), diğerini `decoder_` (CTR modu çözücü) olarak ayarlar.
*   **Not:** Kod `#if defined(__IMPROVED_PACKET_ENCRYPTION__)` ile çevrelenmiştir ve CryptoPP kütüphanesine bağımlıdır.
*   **Bağlantılı Dosyalar:** `cipher.h`, `stdafx.h`, çeşitli CryptoPP başlıkları.

*Buraya `game/src` altındaki diğer özelliklerle ilgili dosyaların belgeleri eklenecektir