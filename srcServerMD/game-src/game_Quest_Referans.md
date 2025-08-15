# Metin2 Oyun Sunucusu - Görev Sistemi Referansı (`game/src`)

Bu belge, Metin2 oyun sunucusunun (`game/src`) görev (quest) sistemiyle ilgili C++ ve Lua bileşenlerini belgeler.

## İçindekiler

*   [Geliştirme Araçları](#geliştirme-araçları)
    *   [`quest/` (qc - Görev Derleyici)](#quest-qc---görev-derleyici)
*   [Lua Arayüzleri (`questlua_*`)](#lua-arayüzleri-questlua_)
    *   [`questlua_affect.cpp`](#questlua_affectcpp)
    *   [`questlua_arena.cpp`](#questlua_arenacpp)
    *   [`questlua_attr67add.cpp`](#questlua_attr67addcpp)
    *   [`questlua_ba.cpp`](#questlua_bacpp)
    *   [`questlua_building.cpp`](#questlua_buildingcpp)
    *   [`questlua_danceevent.cpp`](#questlua_danceeventcpp)
    *   [`questlua_dragonlair.cpp`](#questlua_dragonlaircpp)
    *   [`questlua_dragonsoul.cpp`](#questlua_dragonsoulcpp)
    *   [`questlua_dungeon.cpp`](#questlua_dungeoncpp)
    *   [`questlua_forked.cpp`](#questlua_forkedcpp)
    *   [`questlua_game.cpp`](#questlua_gamecpp)
    *   [`questlua_global.cpp`](#questlua_globalcpp)
    *   [`questlua_guild.cpp`](#questlua_guildcpp)
    *   [`questlua_horse.cpp`](#questlua_horsecpp)
    *   [`questlua_item.cpp`](#questlua_itemcpp)
    *   [`questlua_marriage.cpp`](#questlua_marriagecpp)
    *   [`questlua_meleylair.cpp`](#questlua_meleylaircpp)
    *   [`questlua_mgmt.cpp`](#questlua_mgmtcpp)
    *   [`questlua_monarch.cpp`](#questlua_monarchcpp)
    *   [`questlua_npc.cpp`](#questlua_npccpp)
    *   [`questlua_oxevent.cpp`](#questlua_oxeventcpp)
    *   [`questlua_party.cpp`](#questlua_partycpp)
    *   [`questlua_pc.cpp`](#questlua_pccpp)
*   [Çekirdek Görev Sistemi Sınıfları (.h)](#çekirdek-görev-sistemi-sınıfları-h)
    *   [`quest.h`](#questh)
    *   [`questevent.h`](#questeventh)
*   [Çekirdek Görev Sistemi Sınıfları (.cpp)](#çekirdek-görev-sistemi-sınıfları-cpp)
    *   [`questevent.cpp`](#questeventcpp)

## Geliştirme Araçları

### `quest/` (qc - Görev Derleyici)

*   **Amaç:** Bu klasör, Metin2'nin Görev Derleyicisi (`qc`) aracının kaynak kodunu içerir.
*   **Temel İşlev:** `qc` aracı, insanlar tarafından okunabilir/yazılabilir görev script dosyalarını (genellikle `.quest` uzantılı) alır ve bunları oyun sunucusunun Lua motoru tarafından yürütülebilecek Lua script dosyalarına (`.lua`) veya ilişkili veri dosyalarına derler. Çıktılar genellikle `object/` klasörü altında yapılandırılmış bir dizin hiyerarşisinde oluşturulur.
*   **İçerik ve Çalışma Prensibi:**
    *   `qc.cc`:
        *   **Ayrıştırma (Parsing):** Lua kütüphanesinin lexer'ını (`LexState`, `luaX_lex`) kullanarak `.quest` dosyasını token'lara ayırır.
        *   **Yapı Analizi:** Bir durum makinesi (state machine) kullanarak `.quest` dosyasının beklenen yapısını (`quest ... do`, `state ... do`, `when ... [with ...] begin ... end`, `function ... begin ... end`) kontrol eder.
        *   **Veri Çıkarımı:** Quest ismi, state isimleri, `when` koşulları (isim, argüman, `with` koşulu), `when` bloklarının gövde kodları ve `function` tanımları ayrıştırılır ve ilgili C++ map/set/vector veri yapılarında saklanır (`state_script_map`, `state_arg_script_map`, `define_state_name_set`, `all_functions` vb.).
        *   **Kontroller:**
            *   Lua Syntax Kontrolü (`check_syntax`): Üretilen Lua kod parçalarının (`with` koşulları, `when` gövdeleri, `function` gövdeleri) geçerli Lua sözdizimine sahip olup olmadığını `luaL_loadbuffer` ile kontrol eder.
            *   Fonksiyon Kullanım Kontrolü (`RegisterUsedFunction`, `CheckUsedFunction`, `load_quest_function_list`): `.quest` içinde çağrılan fonksiyonların, önceden tanımlanmış fonksiyon listesinde (`quest_functions` dosyası + `.quest` içindeki `function` tanımları) olup olmadığını kontrol eder.
            *   State Kullanım Kontrolü: `set_state` gibi komutlarla kullanılan state isimlerinin `.quest` içinde tanımlanıp tanımlanmadığını kontrol eder.
        *   **Çıktı Üretimi:** Ayrıştırılan ve kontrol edilen verileri kullanarak `object/` klasörü altında belirli bir yapıda dosyalar oluşturur:
            *   `object/state/<quest_adı>`: Quest'in state isimleri ve CRC32 değerleri ile `function` tanımlarını içeren bir Lua tablosu.
            *   `object/begin_condition/<quest_adı>`: Quest'in başlangıç koşulu (varsa).
            *   `object/<kategori>/<olay>/` veya `object/notarget/<olay>/`: Her `when` olayı için:
                *   Argümanlı `when`: `<quest>.<state>.<idx>.script` (kod), `<quest>.<state>.<idx>.when` (koşul), `<quest>.<state>.<idx>.arg` (argüman) dosyaları.
                *   Argümansız `when`: `<quest>.<state>` dosyası (tüm kod).
    *   `crc32.h`, `crc32.cc`:
        *   **İşlev:** Standart CRC32 (`get_crc32`) ve FNV-1a benzeri hızlı bir hash (`get_fast_hash`) hesaplama fonksiyonları içerir. Büyük/küçük harf duyarsız CRC32 (`get_crc32_case`) fonksiyonu yorum satırı halindedir.
        *   **Kullanım (qc):** `qc.cc` tarafından state isimlerini sayısal bir değere (CRC32) dönüştürmek için `get_crc32` kullanılır. Bu, state'leri verimli bir şekilde temsil etmek ve karşılaştırmak için kullanılır. CRC çakışmalarını önlemek için basit bir mekanizma içerir.
    *   `create_conversion.cc`:
        *   **İşlev:** `qc` tarafından `object/state/` dizininde oluşturulan state dosyalarını okur, her state ismi için `get_crc32` ile güncel CRC değerini hesaplar ve eski state indekslerini/CRC'lerini bu yeni değerlerle güncellemek için komutlar üretir (Örn: veritabanı veya dosyalar için `perl` `sed` benzeri komutlar).
        *   **Amaç:** Muhtemelen state CRC hesaplama yöntemi değiştiğinde veya mevcut state indekslerini standartlaştırmak için kullanılan tek seferlik bir **yardımcı geçiş/dönüşüm aracıdır**.
    *   Yapılandırma Dosyaları (`qc.vcxproj`, `Makefile`, `CMakeLists.txt`): `qc` aracının farklı platformlarda (Windows/Visual Studio, Linux/Make, CMake) derlenmesi için gereken proje ve yapılandırma dosyaları.
*   **Kullanım:** Geliştiriciler görev scriptlerini düzenledikten sonra, `qc` aracını kullanarak sunucunun kullanabileceği son formata dönüştürürler. Bu, oyunun çalışma zamanında değil, geliştirme/build aşamasında çalıştırılan bir araçtır.
*   **Önemli Notlar:** Bu klasördeki kodlar doğrudan çalışan oyun sunucusunun bir parçası değildir, ancak görevlerin oyuna eklenmesi için kritik bir adımdır.

### `lua_incl.h`

*   **Amaç:** Lua C API başlık dosyalarını (`lua.h`, `lauxlib.h`, `lualib.h`) C++ kodu içinden doğru bir şekilde dahil etmek için kullanılır. Özellikle, C++ derleyicileriyle uyumluluk için `extern "C"` bloğu sağlar.
*   **Temel İşlevler/İçerik:**
    *   `#if !defined(_MSC_VER) && defined(__cplusplus)` direktifi ile başlayan ve biten `extern "C"` bloğu, C++ kodu içindeyken ve Microsoft Visual C++ derleyicisi kullanılmıyorken Lua başlıklarının C bağlantı kurallarına göre dahil edilmesini sağlar. Bu, C ve C++ arasında isim mangling (isim bozma) farklılıklarından kaynaklanabilecek bağlantı (linking) hatalarını önler.
    *   Aşağıdaki Lua başlık dosyalarını içerir:
        *   `lua.h`: Lua çekirdek fonksiyonları ve temel tür tanımları.
        *   `lauxlib.h`: Lua yardımcı kütüphanesi; sık kullanılan fonksiyonlar ve makrolar.
        *   `lualib.h`: Lua standart kütüphanelerini (örn. `math`, `string`, `table`) açmak için fonksiyonlar.
*   **Kullanım:** Bu başlık dosyası, C++ projesinde Lua script motorunu entegre eden ve Lua C API'sini doğrudan kullanan `.cpp` dosyaları tarafından dahil edilir. Özellikle görev (quest) sistemi gibi Lua scriptlerinin yoğun olarak kullanıldığı yerlerde önemlidir.
*   **Bağlantılı Dosyalar:** Bu dosya doğrudan bir `.cpp` uygulamasına sahip değildir, ancak Lua entegrasyonu olan tüm C++ dosyaları tarafından (`questlua.cpp`, `questmanager.cpp` ve çeşitli görevle ilgili Lua arayüz dosyaları) dahil edilir.

## Lua Arayüzleri (`questlua_*`)

*Buraya Lua'ya açılan görev fonksiyonları (`questlua_*.cpp` dosyaları) belgelenecektir.*

### `questlua_affect.cpp`

*   **Amaç:** Karakterlere çeşitli etkileri (affect) Lua betikleri aracılığıyla eklemek, kaldırmak ve sorgulamak için fonksiyonlar sağlar. Bu etkiler genellikle geçici bonuslar, debuff'lar veya özel durumlar olabilir.
*   **Lua Ön Eki:** `affect`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `affect.add(apply_on_index, value, duration)`: Mevcut karaktere `AFFECT_QUEST_START_IDX` türünde bir etki ekler.
        *   `apply_on_index` (sayı): `aApplyInfo` dizisindeki efekt türünün indeksi (örn: `APPLY_MAX_HP`).
        *   `value` (sayı): Efektin değeri.
        *   `duration` (sayı): Efektin saniye cinsinden süresi.
        *   **Örnek:** `affect.add(affect.APPLY_MAX_HP, 500, 600)` (10 dakika boyunca +500 Maks HP verir).
    *   `affect.remove(affect_type_or_quest_index)`: Belirtilen türdeki veya mevcut göreve ait efekti kaldırır.
        *   `affect_type_or_quest_index` (sayı, opsiyonel): Kaldırılacak efektin türü. Eğer 0 veya verilmezse, mevcut görevin `AFFECT_QUEST_START_IDX` + quest_index türündeki efektini kaldırır.
    *   `affect.remove_bad()`: Karakter üzerindeki tüm "kötü" etkileri (debuff) kaldırır.
    *   `affect.remove_good()`: Karakter üzerindeki tüm "iyi" etkileri (buff) kaldırır.
    *   `affect.add_hair(apply_on_index, value, duration)`: Saç stiliyle ilgili özel bir etki (`AFFECT_HAIR`) ekler. Parametreleri `affect.add` ile benzerdir.
    *   `affect.remove_hair()`: `AFFECT_HAIR` türündeki efekti kaldırır. Varsa, kalan süresini Lua yığınına push eder.
    *   `affect.get_apply_on(affect_type)`: Belirtilen `affect_type`'a sahip bir efektin hangi `APPLY_*` türüne etki ettiğini (`wApplyOn`) döndürür. Efekt yoksa `nil` döner.
    *   `affect.get_apply_value(affect_type)`: Belirtilen `affect_type`'a sahip bir efektin değerini (`lApplyValue`) döndürür. Efekt yoksa `nil` döner.
    *   `affect.add_collect(apply_on_index, value, duration)`: Toplama görevleriyle ilgili özel bir etki (`AFFECT_COLLECT`) ekler. Parametreleri `affect.add` ile benzerdir.
    *   `affect.add_collect_point(point_type, value, duration)`: Belirli bir `POINT_*` türüne etki eden `AFFECT_COLLECT` türünde bir efekt ekler.
    *   `affect.remove_collect(apply_on_index, value)`: Belirli bir `apply_on` ve değere sahip `AFFECT_COLLECT` türündeki efekti kaldırır.
    *   `affect.remove_all_collect()`: Karakter üzerindeki tüm `AFFECT_COLLECT` türündeki efektleri kaldırır.
*   **Kayıt:** `RegisterAffectFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `affect` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `questmanager.h`, `sectree_manager.h`, `char.h`, `affect.h`, `db.h`.

### `questlua_arena.cpp`

*   **Amaç:** Oyuncular arası düello (arena) sistemiyle ilgili işlevleri Lua betiklerine açar. Düello başlatma, arena haritası tanımlama, düello listesini alma ve gözlemci ekleme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `arena`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `arena.start_duel(target_char_name, set_point)`: Mevcut karakter ile adı verilen `target_char_name` arasında `set_point` puanlık bir düello başlatır.
        *   Başarı durumuna göre 1 (başarılı), 0 (karakterler bulunamadı), 2 (karakterlerden biri zaten arenada), 3 (düello başlatılamadı) döndürür.
        *   Karakterler at üzerindeyse attan indirir.
        *   `LC_IsCanada()` kontrolüne göre farklı `StartDuel` parametreleri çağrılabilir (muhtemelen süre limiti ekler).
    *   `arena.add_map(map_idx, start_ax, start_ay, start_bx, start_by)`: Arena sistemi için yeni bir harita ve başlangıç pozisyonları tanımlar.
        *   `map_idx` (sayı): Harita indeksi.
        *   `start_ax`, `start_ay` (sayı): A oyuncusunun başlangıç koordinatları.
        *   `start_bx`, `start_by` (sayı): B oyuncusunun başlangıç koordinatları.
    *   `arena.get_duel_list()`: Devam eden tüm düelloların listesini Lua yığınına bir tablo olarak push eder. Her tablo öğesi düellodaki iki oyuncunun ismini içerir.
    *   `arena.add_observer(map_idx, obs_x, obs_y)`: Mevcut karakteri, belirtilen `map_idx`'teki bir arenaya `obs_x`, `obs_y` koordinatlarından gözlemci olarak ekler.
    *   `arena.is_in_arena(pid)`: Verilen oyuncu ID'sine (`pid`) sahip karakterin bir arenada düellocu olup olmadığını kontrol eder. 1 (arenada), 0 (değil) döndürür.
*   **Kayıt:** `RegisterArenaFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `arena` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `char.h`, `char_manager.h`, `arena.h`.

### `questlua_attr67add.cpp`

*   **Amaç:** (`__ATTR_6TH_7TH__` makrosu tanımlıysa) Eşyalara 6. ve 7. efsunları ekleme sistemiyle ilgili işlemleri Lua betiklerine açar. Bu sistem genellikle özel bir NPC arayüzü (NPC_STORAGE benzeri) üzerinden çalışır.
*   **Lua Ön Eki:** `attr67add`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `attr67add.holding()`: 6./7. efsun ekleme arayüzünde (geçici saklama alanı) işlenmeyi bekleyen bir eşya olup olmadığını kontrol eder. `true` veya `false` döndürür.
    *   `attr67add.collect()`: Arayüzdeki işlenmiş eşyayı oyuncunun envanterine alır. Yeterli yer yoksa hata mesajı verir. Başarılı toplama sonrası ilgili görev bayraklarını sıfırlar.
    *   `attr67add.success()`: Efsun ekleme işleminin başarılı olup olmadığını kontrol eder (`add_attr67.success` görev bayrağı).
        *   Eğer başarılıysa ve nadir efsun henüz eklenmemişse (`add_attr67.add` bayrağı <= 0), eşyaya `AddRareAttribute()` fonksiyonu ile nadir efsunu ekler ve `add_attr67.add` bayrağını 1 yapar.
        *   `true` (başarılı ve eşya arayüzde) veya `false` döndürür.
    *   `attr67add.item_vnum()`: Arayüzdeki eşyanın VNUM'unu döndürür. Eşya yoksa 0 döner.
    *   `attr67add.window_type()`: Kullanılan arayüz penceresinin türünü (sabit olarak `NPC_STORAGE`) döndürür.
    *   `attr67add.window_cell()`: Arayüzdeki eşyanın bulunduğu hücre indeksini (sabit olarak 0, tek eşya kullanıldığı varsayılır) döndürür.
*   **Kayıt:** `RegisterAttr67AddFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `attr67add` tablosu altına kaydedilir.
*   **Koşullu Derleme:** Bu dosyanın içeriği ve fonksiyonları sadece `__ATTR_6TH_7TH__` makrosu tanımlı olduğunda derlenir ve kullanılır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `char.h`, `item.h`, `item_manager.h`.

### `questlua_ba.cpp`

*   **Amaç:** Savaş Arenası (Battle Arena) ile ilgili temel bir işlevi Lua betiklerine açar.
*   **Lua Ön Eki:** `ba`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `ba.start(map_index)`: Belirtilen harita indeksindeki Savaş Arenası'nı başlatır.
        *   `map_index` (sayı): Savaş Arenası'nın başlatılacağı haritanın özel indeksi.
*   **Kayıt:** `RegisterBattleArenaFunctionTable()` fonksiyonu ile `ba.start` fonksiyonu Lua'da `ba` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `BattleArena.h`.

### `questlua_building.cpp`

*   **Amaç:** Lonca arazileri ve binaları ile ilgili çeşitli işlemleri Lua betiklerine açar. Arazi ID'si ve bilgilerini alma, arazi sahibi ayarlama, bir loncanın arazi sahibi olup olmadığını kontrol etme ve bina yeniden yapılandırma gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `building`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `building.get_land_id(map_index, x, y)`: Verilen harita indeksi ve koordinatlardaki arazinin ID'sini döndürür. Arazi bulunamazsa 0 döner.
    *   `building.get_land_info(land_id)`: Belirtilen `land_id`'ye sahip arazinin bilgilerini döndürür: fiyat, sahip lonca ID'si, minimum lonca seviyesi. Arazi bulunamazsa varsayılan yüksek değerler döner.
    *   `building.set_land_owner(land_id, guild_id)`: Belirtilen `land_id`'ye sahip arazinin sahibini, verilen `guild_id` olarak ayarlar. Sadece arazinin mevcut bir sahibi yoksa işlem yapar.
    *   `building.has_land(guild_id)`: Verilen `guild_id`'ye sahip loncanın bir araziye sahip olup olmadığını kontrol eder (veritabanından sorgu yaparak). `true` veya `false` döndürür.
    *   `building.reconstruct(new_building_vnum)`: Mevcut NPC'nin bulunduğu lonca arazisindeki binayı, belirtilen `new_building_vnum`'a sahip yeni bir bina ile yeniden yapılandırır. Bu fonksiyonun bir NPC üzerinden çağrılması beklenir.
*   **Kayıt:** `RegisterBuildingFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `building` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `questmanager.h`, `sectree_manager.h`, `char.h`, `guild.h`, `db.h`, `building.h`.

### `questlua_danceevent.cpp`

*   **Amaç:** Özel bir "Dans Etkinliği" ile ilgili bir işlevi Lua betiklerine açar.
*   **Lua Ön Eki:** `dance_event`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `dance_event.gohome()`: Belirli bir harita (indeks 115) üzerindeki tüm oyuncuları (GM olmayanları ve belirli bir koordinat aralığı dışındakileri) kendi başlangıç noktalarına ("evlerine") ışınlar.
        *   Bu fonksiyon, `FWarpToHome` adlı bir functor yapısını kullanarak haritadaki tüm karakterler üzerinde işlem yapar.
*   **Kayıt:** `RegisterDanceEventFunctionTable()` fonksiyonu ile `dance_event.gohome` fonksiyonu Lua'da `dance_event` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `sectree_manager.h`, `char.h`.

### `questlua_dragonlair.cpp`

*   **Amaç:** Ejderha İni (Dragon Lair) sistemiyle ilgili bir işlevi Lua betiklerine açar.
*   **Lua Ön Eki:** `DragonLair` (Dosya adında 'dl' kısaltması olsa da Lua tablosu `DragonLair` olarak kaydedilmiş)
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `DragonLair.startRaid(base_map_index)`: Mevcut karakterin bulunduğu haritada, belirtilen `base_map_index`'e bağlı olarak ve karakterin loncası için bir Ejderha İni baskını başlatır.
        *   `base_map_index` (sayı): Baskının ilişkili olduğu ana haritanın indeksi.
*   **Kayıt:** `RegisterDragonLairFunctionTable()` fonksiyonu ile `DragonLair.startRaid` fonksiyonu Lua'da `DragonLair` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `DragonLair.h`, `char.h`, `guild.h`.

### `questlua_dragonsoul.cpp`

*   **Amaç:** Ejderha Taşı (Dragon Soul) Simya sistemiyle ilgili çeşitli işlevleri Lua betiklerine açar. Simya penceresini açma, oyuncuya simya yapma yetkisi verme ve bu yetkinin durumunu kontrol etme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `ds`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `ds.open_refine_window()`: Mevcut karakter için Ejderha Taşı Simya (işleme/yükseltme) penceresini açar. Karakterin simya için yetkili (`DragonSoul_IsQualified()`) olması gerekir. Pencere, mevcut NPC üzerinden açılır.
    *   `ds.give_qualification()`: Mevcut karaktere Ejderha Taşı Simya sistemini kullanma yetkisi verir (`DragonSoul_GiveQualification()`).
    *   `ds.is_qualified()`: Mevcut karakterin Ejderha Taşı Simya sistemini kullanmaya yetkili olup olmadığını kontrol eder. 1 (yetkili) veya 0 (yetkisiz) döndürür.
    *   `ds.open_changeattr_window()`: (`__DS_CHANGE_ATTR__` tanımlıysa) Mevcut karakter için Ejderha Taşı efsun değiştirme penceresini açar. Karakterin yetkili olması gerekir ve pencere mevcut NPC üzerinden açılır.
*   **Hata Yönetimi:** Fonksiyonlar içinde `sys_err` makrosu kullanılarak `CQuestManager::instance().QuestError` üzerinden hata loglaması yapılır.
*   **Kayıt:** `RegisterDragonSoulFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `ds` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `questmanager.h`, `char.h`.

### `questlua_dungeon.cpp`

*   **Amaç:** Zindan (Dungeon) sistemiyle ilgili çok sayıda işlevi Lua betiklerine açar. Bu, zindan oluşturma, oyuncu/parti ışınlama, yaratık/obje oluşturma (spawn), zindan içi bayrak (flag) yönetimi, görev bayrağı ayarlama, zindan durumunu kontrol etme, oyuncuları zindandan çıkarma ve zindana özel NPC/yaratık yönetimi gibi geniş bir yelpazeyi kapsar.
*   **Lua Ön Eki:** `d`
*   **Temel İşlevler/Örnek Fonksiyonlar (Kategorilere Ayrılmış):**
    *   **Zindan Yönetimi ve Bilgi:**
        *   `d.new_jump(map_idx, x, y)`: Belirtilen harita indeksi için yeni bir zindan örneği oluşturur ve mevcut karakteri zindanın belirtilen x, y koordinatlarına ışınlar.
        *   `d.new_jump_all(map_idx, x, y)`: Yeni bir zindan örneği oluşturur ve mevcut karakterin bulunduğu haritadaki tüm oyuncuları zindanın belirtilen x, y koordinatlarına ışınlar. (`__DUNGEON_RENEWAL__` ile ek kayıt mantığı içerir).
        *   `d.new_jump_party(map_idx, x, y)`: Yeni bir zindan örneği oluşturur ve mevcut karakterin partisini zindanın belirtilen x, y koordinatlarına ışınlar.
        *   `d.jump_party_with_opp_leader(map_idx, xA, yA, xB, yB, opp_leader_pid)`: Yeni bir zindan örneği oluşturur ve hem mevcut karakterin partisini (xA, yA'ya) hem de `opp_leader_pid` ile belirtilen rakip liderin partisini (xB, yB'ye) zindana ışınlar.
        *   `d.join(map_idx)`: Mevcut karakteri (eğer parti lideriyse partisiyle birlikte) belirtilen `map_idx`'e sahip mevcut bir zindana katılır.
        *   `d.exit()`: Mevcut karakteri, zindana girmeden önceki kayıtlı konumuna ışınlar.
        *   `d.exit_all()`: Mevcut zindandaki tüm oyuncuları, zindana girmeden önceki kayıtlı konumlarına ışınlar.
        *   `d.exit_all_to_start_position()`: Mevcut zindandaki tüm oyuncuları, zindanın başlangıç pozisyonuna ışınlar.
        *   `d.find(map_idx)`: Verilen `map_idx` ile bir zindan örneğinin var olup olmadığını kontrol eder (`true`/`false`).
        *   `d.select(map_idx)`: Verilen `map_idx` ile zindan örneğini bulur ve mevcut görev bağlamı için seçili zindan olarak ayarlar (`true`/`false`). Bulamazsa veya `map_idx` 0 ise seçimi kaldırır.
        *   `d.get_map_index()`: Mevcut seçili zindanın harita indeksini döndürür. Zindan yoksa 0 döner.
        *   `d.purge()`: Mevcut zindandaki tüm canavarları ve taşları temizler.
        *   `d.kill_all()`: Mevcut zindandaki tüm canavarları ve taşları öldürür.
        *   `d.count_monster()`: Mevcut zindandaki canavar sayısını döndürür.
        *   `d.notice(line1, line2, ...)`: Zindandaki tüm oyunculara 10 satıra kadar duyuru mesajı gönderir.
        *   `d.set_warp_location(map_idx, x, y)`: Zindandaki tüm oyuncuların bir sonraki ışınlanma/ölüm sonrası canlanma noktasını ayarlar.
        *   `d.set_exit_all_at_eliminate(duration)`: Zindandaki tüm hedefler yok edildiğinde, belirtilen süre (saniye) sonunda tüm oyuncuların otomatik olarak zindandan çıkarılmasını ayarlar.
        *   `d.set_warp_at_eliminate(duration, map_idx, x, y, regen_file)`: Tüm hedefler yok edildiğinde, belirtilen süre sonunda tüm oyuncuların belirtilen harita/koordinatlara ışınlanmasını ve opsiyonel olarak bir `regen_file`'ın spawn edilmesini ayarlar.
        *   `d.check_eliminated()`: Zindandaki tüm hedeflerin yok edilip edilmediğini manuel olarak kontrol eder ve ilgili eylemleri (çıkış/ışınlama) tetikler.
    *   **Bayrak (Flag) Yönetimi:**
        *   `d.setf(flag_name, value)`: Mevcut zindana ait bir bayrağın değerini ayarlar.
        *   `d.getf(flag_name)`: Mevcut zindana ait bir bayrağın değerini alır.
        *   `d.getf_from_map_index(flag_name, map_idx)`: Belirtilen `map_idx`'e sahip zindanın bir bayrağının değerini alır.
        *   `d.setqf(flag_name, value)`: Mevcut zindandaki tüm oyuncuların belirtilen görev bayrağını ayarlar (bayrak adı `quest_adı.flag_name` şeklinde olur).
        *   `d.setqf2(quest_name, flag_name, value)`: Mevcut zindandaki tüm oyuncuların `quest_name.flag_name` görev bayrağını ayarlar.
    *   **Yaratık/Obje Oluşturma (Spawn) ve Yönetimi:**
        *   `d.regen_file(filename)`: Belirtilen `regen` dosyasını mevcut zindanda spawn eder (mevcut olanları silmez).
        *   `d.set_regen_file(filename)`: Belirtilen `regen` dosyasını mevcut zindanda spawn eder (mevcut olanları siler).
        *   `d.clear_regen()`: Mevcut zindandaki tüm `regen` dosyalarını ve onlardan spawn olanları temizler.
        *   `d.spawn_mob(vnum, x, y, radius, count)`: Belirtilen VNUM'a sahip canavarı belirtilen x, y koordinatlarına (opsiyonel yarıçap içinde rastgele, varsayılan 1 adet) spawn eder. İlk spawn edilen canavarın VID'sini döndürür.
        *   `d.spawn_mob_dir(vnum, x, y, direction)`: Belirtilen yöne bakacak şekilde canavar spawn eder.
        *   `d.spawn_mob_ac_dir(vnum, x, y, direction)`: Saldırgan modda ve belirtilen yöne bakacak şekilde canavar spawn eder.
        *   `d.spawn_name_mob(vnum, x, y, name)`: Belirtilen isme sahip bir canavar spawn eder.
        *   `d.spawn_goto_mob(from_x, from_y, to_x, to_y)`: Başlangıç koordinatlarından hedef koordinatlara giden (muhtemelen görünmez) bir yönlendirme NPC'si spawn eder.
        *   `d.spawn_group(group_vnum, x, y, radius, aggressive, count)`: Belirtilen canavar grubunu (VNUM'u ile) belirtilen koordinatlara (yarıçap içinde, saldırganlık ayarı ile, belirtilen sayıda grup) spawn eder. Grubun liderinin VID'sini döndürür.
        *   `d.spawn_unique(key, vnum, regen_file_path)`: Belirli bir anahtar (`key`) ile tanımlanan özel bir canavarı (boss vb.) `regen_file_path` içindeki konumda spawn eder.
        *   `d.spawn_move_unique(key, vnum, regen_file_path_from, regen_file_path_to)`: Belirli bir anahtarla tanımlanan özel canavarı bir konumdan diğerine hareket edecek şekilde spawn eder.
        *   `d.spawn_move_group(vnum, regen_file_path_from, regen_file_path_to)`: Belirli bir canavar grubunu bir konumdan diğerine hareket edecek şekilde spawn eder.
        *   `d.spawn_stone_door(key, regen_file_path)`: Belirli bir anahtarla tanımlanan bir taş kapıyı spawn eder.
        *   `d.spawn_wooden_door(key, regen_file_path)`: Belirli bir anahtarla tanımlanan bir ahşap kapıyı spawn eder.
        *   `d.set_unique(key, vid)`: Belirli bir anahtarla tanımlanan özel yaratığın VID'sini manuel olarak ayarlar.
        *   `d.get_unique_vid(key)`: Belirli bir anahtarla tanımlanan özel yaratığın VID'sini döndürür.
        *   `d.is_unique_dead(key)`: Belirli bir anahtarla tanımlanan özel yaratığın ölü olup olmadığını kontrol eder (`true`/`false`).
        *   `d.kill_unique(key)`: Belirli bir anahtarla tanımlanan özel yaratığı öldürür.
        *   `d.purge_unique(key)`: Belirli bir anahtarla tanımlanan özel yaratığı zindandan temizler.
        *   `d.purge_area(x1, y1, x2, y2)`: Belirtilen koordinat aralığındaki tüm canavarları ve taşları temizler.
        *   `d.unique_set_hp(key, hp)`: Belirli bir anahtarla tanımlanan özel yaratığın can puanını ayarlar.
        *   `d.unique_set_maxhp(key, max_hp)`: Belirli bir anahtarla tanımlanan özel yaratığın maksimum can puanını ayarlar.
        *   `d.unique_set_def_grade(key, grade)`: Belirli bir anahtarla tanımlanan özel yaratığın savunma derecesini ayarlar.
        *   `d.unique_get_hp_perc(key)`: Belirli bir anahtarla tanımlanan özel yaratığın can puanı yüzdesini döndürür.
    *   **Zindan Durumu ve İstatistik:**
        *   `d.get_kill_stone_count()`: Zindanda kırılan metin taşı sayısını döndürür.
        *   `d.get_kill_mob_count()`: Zindanda öldürülen canavar sayısını döndürür.
        *   `d.is_use_potion()`: Zindanda iksir kullanılıp kullanılmadığını kontrol eder (`true`/`false`).
        *   `d.revived()`: Zindanda yeniden canlanma kullanılıp kullanılmadığını kontrol eder (`true`/`false`).
    *   **Parti ve Oyuncu Yönetimi:**
        *   `d.set_dest(x, y)`: Mevcut karakterin partisine haritada hedef konumu gönderir (mini harita üzerinde ok işareti).
        *   `d.all_near_to(x, y)`: Zindandaki tüm oyuncuların belirtilen koordinatlara belirli bir mesafe (30 birim) içinde olup olmadığını kontrol eder (`true`/`false`).
    *   **Eşya Grubu Yönetimi (Giriş/Çıkış Kontrolü):**
        *   `d.set_item_group(group_name, size, item_vnum1, count1, item_vnum2, count2, ...)`: Zindan için bir eşya grubu tanımlar. Oyuncuların zindana girmesi veya kalması için bu gruptaki eşyalardan birine sahip olması gerekir.
        *   `d.say_diff_by_item_group(group_name, can_enter_msg, cant_enter_msg)`: Tanımlanan eşya grubuna göre, zindandaki oyunculara farklı mesajlar gösterir (gerekli eşyaya sahip olanlara `can_enter_msg`, olmayanlara `cant_enter_msg`).
        *   `d.exit_all_by_item_group(group_name)`: Tanımlanan eşya grubundaki gereksinimleri karşılamayan tüm oyuncuları zindandan çıkarır.
        *   `d.delete_item_in_item_group_from_all(group_name)`: Zindandaki oyunculardan, tanımlanan eşya grubundaki gereksinimi karşılayan ilk eşyayı siler.
    *   **Dungeon Renewal (`__DUNGEON_RENEWAL__`):**
        *   `d.clear_participants()`: Zindana kayıtlı tüm katılımcıların listesini temizler.
        *   `d.register_participant()`: Mevcut karakteri zindana katılımcı olarak kaydeder.
        *   `d.register_party_participants()`: Mevcut karakterin partisini zindana katılımcı olarak kaydeder.
        *   `d.is_registered(map_idx)`: Mevcut karakterin belirtilen harita indeksine sahip zindana kayıtlı olup olmadığını kontrol eder (`true`/`false`).
*   **Hata Yönetimi:** Fonksiyonlar içinde `sys_err` makrosu kullanılarak `CQuestManager::instance().QuestError` üzerinden hata loglaması yapılır.
*   **Kayıt:** `RegisterDungeonFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `d` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `questmanager.h`, `questlua.h`, `dungeon.h`, `char.h`, `party.h`, `buffer_manager.h`, `char_manager.h`, `packet.h`, `desc_client.h`, `desc_manager.h`, `config.h`, `db.h`, `p2p.h`.

### `questlua_forked.cpp`

*   **Amaç:** Üç Yol Savaşı (Forked War / Sungzi) etkinliğiyle ilgili çeşitli işlevleri Lua betiklerine açar. Ölüm sayısını ayarlama/alma, imparatorluk öldürme skorlarını sıfırlama, etkinliği başlatma, başlangıç pozisyonlarını ve harita indekslerini alma, harita kontrolleri, kullanıcı kaydı ve canavar temizleme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `forked`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `forked.setdeadcount()`: Mevcut karakterin Üç Yol Savaşı'ndaki yeniden canlanma hakkı sayısını, `threeway_war_dead_count` olay bayrağının değerine ayarlar.
    *   `forked.getdeadcount()`: Mevcut karakterin kalan yeniden canlanma hakkı sayısını döndürür.
    *   `forked.initkillcount()`: Mevcut karakterin imparatorluğunun Üç Yol Savaşı'ndaki öldürme skorunu sıfırlar.
    *   `forked.initforked()`: Üç Yol Savaşı etkinliğini başlatır ve rastgele harita setini ayarlar (`CThreeWayWar::Initialize`, `RandomEventMapSet`).
    *   `forked.get_sungzi_start_pos()`: Mevcut karakterin imparatorluğu için Sungzi (kale) haritasındaki başlangıç X ve Y koordinatlarını döndürür.
    *   `forked.get_pass_start_pos()`: Mevcut karakterin imparatorluğu için Geçit (pass) haritasındaki başlangıç X ve Y koordinatlarını döndürür.
    *   `forked.getsungzimapindex()`: Sungzi haritasının indeksini döndürür.
    *   `forked.getpassmapindexbyempire(empire_id)`: Belirtilen imparatorluğa ait Geçit haritasının indeksini döndürür.
    *   `forked.getpasspathbyempire(empire_id)`: Belirtilen imparatorluğa ait Geçit haritasının dosya yolunu (questlerde kullanılmak üzere) döndürür.
    *   `forked.isforkedmapindex(map_index)`: Verilen harita indeksinin bir Üç Yol Savaşı haritası olup olmadığını kontrol eder (`true`/`false`).
    *   `forked.issungzimapindex(map_index)`: Verilen harita indeksinin Sungzi haritası olup olmadığını kontrol eder (`true`/`false`).
    *   `forked.warp_all_in_map(from_map_idx, to_map_idx, x, y, delay_sec)`: Belirtilen `from_map_idx` haritasındaki tüm oyuncuları, `delay_sec` saniye sonra `to_map_idx` haritasındaki `x`, `y` koordinatlarına ışınlamak için bir zamanlayıcı başlatır.
    *   `forked.is_registered_user()`: Mevcut karakterin Üç Yol Savaşı'na kayıtlı olup olmadığını kontrol eder (`true`/`false`).
    *   `forked.register_user()`: Mevcut karakteri Üç Yol Savaşı'na kaydeder.
    *   `forked.purge_all_monsters()`: Tüm Üç Yol Savaşı haritalarındaki canavarları temizler.
*   **Kayıt:** `RegisterForkedFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `forked` tablosu altına kaydedilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `threeway_war.h`, `questlua.h`, `questmanager.h`, `char.h`, `dungeon.h`, `p2p.h`, `locale_service.h`.

### `questlua_game.cpp`

*   **Amaç:** Genel oyun mekanikleri ve sistemleriyle ilgili çeşitli işlevleri Lua betiklerine açar. Olay bayrakları, depo (safebox), nesne market (mall), lonca oluşturma, eşya düşürme, posta kutusu, aura sistemi, görünüm değiştirme ve binek yükseltme gibi farklı özelliklere erişim sağlar.
*   **Lua Ön Eki:** `game`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `game.set_event_flag(flag_name, value)`: Belirtilen isimdeki global bir olay bayrağının (event flag) değerini ayarlar.
    *   `game.get_event_flag(flag_name)`: Belirtilen isimdeki global olay bayrağının değerini alır.
    *   `game.request_make_guild()`: Mevcut karaktere lonca oluşturma arayüzünü açar (istemciye `HEADER_GC_REQUEST_MAKE_GUILD` paketi gönderir).
    *   `game.get_safebox_level()`: Mevcut karakterin depo seviyesini (sayfa sayısı) döndürür.
    *   `game.set_safebox_level(level)`: Mevcut karakterin depo seviyesini (ve boyutunu) ayarlar. Veritabanına istek gönderir.
    *   `game.open_safebox()`: Mevcut karakter için depo şifresi girme arayüzünü açar (`ShowMeSafeboxPassword` komutu).
    *   `game.open_mall()`: Mevcut karakter için nesne market şifresi girme arayüzünü açar (`ShowMeMallPassword` komutu).
    *   `game.drop_item(item_vnum, count)`: Belirtilen VNUM ve sayıda eşyayı mevcut karakterin yakınına yere atar.
    *   `game.drop_item_with_ownership(item_vnum, count, duration_sec)`: Eşyayı yere atar ancak belirli bir süre (`duration_sec`) boyunca sadece mevcut karaktere ait olacak şekilde sahiplik ayarlar. Süre 0 veya verilmezse kalıcı sahiplik olur.
    *   `game.drop_item_with_ownership_and_dice(item_vnum, count, duration_sec)`: (`__DICE_SYSTEM__` tanımlıysa) Eşyayı sahiplikle yere atar ve eğer karakter bir partideyse zar atma sistemini başlatır.
    *   `game.open_web_mall()`: Oyun içi nesne marketini açar (`do_in_game_mall` komutunu çağırır).
    *   `game.get_config(config_name)`: Belirtilen sunucu yapılandırma ayarının değerini döndürür (Örn: `game.get_config("create_with_full_set")`).
    *   `game.open_mailbox()`: (`__MAILBOX__` tanımlıysa) Mevcut karakter için posta kutusunu açar.
    *   `game.send_gm_mail(player_name, title, message, item_vnum, item_count, yang, won)`: (`__MAILBOX__` tanımlıysa) Belirtilen oyuncuya GM postası gönderir (eşya, Yang ve Won içerebilir).
    *   `game.open_aura_absorb_window()`: (`__AURA_COSTUME_SYSTEM__` tanımlıysa) Aura soğurma penceresini açar.
    *   `game.open_aura_growth_window()`: (`__AURA_COSTUME_SYSTEM__` tanımlıysa) Aura yükseltme penceresini açar.
    *   `game.open_aura_evolve_window()`: (`__AURA_COSTUME_SYSTEM__` tanımlıysa) Aura evrimleştirme penceresini açar.
    *   `game.open_changelook(type)`: (`__CHANGE_LOOK_SYSTEM__` tanımlıysa) Belirtilen tür için görünüm değiştirme penceresini açar.
    *   `game.open_mount_up_grade()`: (`__RIDING_EXTENDED__` tanımlıysa) Binek yükseltme penceresini açar.
*   **Hata Yönetimi:** Fonksiyonlar içinde `sys_err` makrosu kullanılarak `CQuestManager::instance().QuestError` üzerinden hata loglaması yapılır.
*   **Kayıt:** `RegisterGameFunctionTable()` fonksiyonu ile tüm bu fonksiyonlar Lua'da `game` tablosu altına kaydedilir.
*   **Koşullu Derleme:** Bazı fonksiyonlar (`__DICE_SYSTEM__`, `__MAILBOX__`, `__AURA_COSTUME_SYSTEM__`, `__CHANGE_LOOK_SYSTEM__`, `__RIDING_EXTENDED__`) ilgili makrolar tanımlı olduğunda derlenir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `desc_client.h`, `char.h`, `item_manager.h`, `item.h`, `cmd.h`, `packet.h`, `utils.h`, `config.h`, (koşullu olarak) `party.h`, `mount_up_grade.h`.

### `questlua_global.cpp`

*   **Amaç:** Bu dosya, Lua betiklerine genel amaçlı, herhangi bir oyun nesnesine (karakter, eşya vb.) doğrudan bağlı olmayan bir dizi yardımcı fonksiyon sunar. Bu fonksiyonlar genellikle sunucu geneli işlemler, zamanlayıcılar, duyurular, loglama ve çeşitli yardımcı araçlar için kullanılır.
*   **Lua Ön Eki:** Yok (fonksiyonlar doğrudan global isim alanına kaydedilir, örn: `say("mesaj")`).
*   **Temel İşlevler/Örnek Fonksiyonlar (Kategorilere Ayrılmış):**
    *   **Mesajlaşma ve Arayüz:**
        *   `say(mesaj)`: Mevcut karaktere standart bir konuşma balonu mesajı gönderir. Script'e `[ENTER]` ekler.
        *   `chat(mesaj)`: Mevcut karaktere genel sohbet (`CHAT_TYPE_TALKING`) mesajı gönderir.
        *   `cmdchat(mesaj)`: Mevcut karaktere komut tipi (`CHAT_TYPE_COMMAND`) sohbet mesajı gönderir.
        *   `syschat(mesaj)`: Mevcut karaktere sistem mesajı (`CHAT_TYPE_INFO`) gönderir.
        *   `notice(mesaj)`: Mevcut karaktere duyuru tipi (`CHAT_TYPE_NOTICE`) mesaj gönderir.
        *   `big_notice(mesaj)`: Mevcut karaktere büyük duyuru tipi (`CHAT_TYPE_BIG_NOTICE`) mesaj gönderir.
        *   `setleftimage(resim_yolu)`: Görev penceresinin sol tarafına resim eklemek için script komutu (`[LEFTIMAGE src;...]`) oluşturur.
        *   `settopimage(resim_yolu)`: Görev penceresinin üst tarafına resim eklemek için script komutu (`[TOPIMAGE src;...]`) oluşturur.
        *   `set_skin(skin_index)` veya `setskin(skin_index)`: Görev penceresinin arayüz stilini (`CQuestManager::SetSkinStyle`) ayarlar.
        *   `raw_script(script_string)`: Ham bir script komutunu mevcut göreve ekler (`CQuestManager::AddScript`). Test sunucusunda loglanır.
        *   `notice_all(mesaj1, mesaj2, ..., mesaj6)`: Tüm sunucuya en fazla 6 satırlık yerelleştirilmiş bir duyuru gönderir (P2P ile `HEADER_GG_LOCALE_NOTICE` ve yerel olarak `SendLocaleNotice`).
        *   `big_notice_all(mesaj)`: Tüm sunucuya büyük puntolu bir duyuru gönderir (P2P ile `HEADER_GG_BIG_NOTICE` ve yerel olarak `SendBigNotice`).
        *   `notice_in_map(mesaj, buyuk_font_mu)`: Mevcut karakterin bulunduğu haritadaki herkese duyuru (`SendNoticeMap`) gönderir.
        *   `say_in_map(map_index, script_mesaji)`: Belirli bir haritadaki tüm oyunculara bir görev script mesajı (`HEADER_GC_SCRIPT`) gönderir. Mesaja `[ENTER][DONE]` eklenir.
        *   `notice_mission(mesaj)` (`__12ZI_NOTICE__` tanımlıysa): Mevcut karaktere görev (`CHAT_TYPE_MISSION`) bildirimi gönderir.
        *   `notice_sub_mission(mesaj)` (`__12ZI_NOTICE__` tanımlıysa): Mevcut karaktere alt görev (`CHAT_TYPE_SUB_MISSION`) bildirimi gönderir.
    *   **Zamanlayıcılar:**
        *   `server_timer(isim, saniye, arguman)`: Belirli bir süre sonra çalışacak, sunucu geneli, tek seferlik bir zamanlayıcı (`quest_create_server_timer_event` ile `loop=false`) ayarlar. İsimli script'i yükler.
        *   `server_loop_timer(isim, saniye, arguman)`: Belirli aralıklarla tekrarlanacak, sunucu geneli bir zamanlayıcı (`quest_create_server_timer_event` ile `loop=true`) ayarlar. İsimli script'i yükler.
        *   `clear_server_timer(isim, arguman)`: Belirli bir sunucu zamanlayıcısını (`CQuestManager::ClearServerTimer`) temizler.
        *   `get_server_timer_arg()`: Mevcut sunucu zamanlayıcısının argümanını (`CQuestManager::GetServerTimerArg`) döndürür.
        *   `timer(saniye)`: Mevcut oyuncu için isimsiz, tek seferlik bir zamanlayıcı (`quest_create_timer_event` ile `name=""`) ayarlar.
        *   `timer(isim, saniye)`: Mevcut oyuncu için isimli, tek seferlik bir zamanlayıcı (`quest_create_timer_event`) ayarlar. İsimli script'i yükler.
        *   `loop_timer(isim, saniye)`: Mevcut oyuncu için isimli, döngüsel bir zamanlayıcı (`quest_create_timer_event` ile `loop=true`) ayarlar. İsimli script'i yükler.
        *   `cleartimer(isim)`: Mevcut oyuncunun belirtilen isimdeki zamanlayıcısını (`PC::RemoveTimer`) temizler.
    *   **Bilgi Alma ve Ayarlar:**
        *   `get_locale()` veya `LC()`: Mevcut karakterin veya varsayılan yerel ayar kodunu (`GetLocaleCodeName`) döndürür (örn: "tr", "en").
        *   `locale_quest(quest_id)`: Belirli bir görev ID'si için yerelleştirilmiş metni (`LC_QUEST`) döndürür.
        *   `number(min, max)`: Genel `::number` fonksiyonunu kullanarak belirtilen aralıkta rastgele bir tamsayı üretir.
        *   `time_to_str(timestamp)`: Unix zaman damgasını `asctime(gmtime())` ile okunabilir bir string'e çevirir.
        *   `getnpcid(npc_adi)`: NPC adına göre VNUM'unu (`CQuestManager::FindNPCIDByName`) bulur.
        *   `is_test_server()`: Global `test_server` değişkeninin durumunu döndürür.
        *   `is_speed_server()`: Global `speed_server` değişkeninin durumunu döndürür.
        *   `item_name(item_vnum)`: Eşya VNUM'una göre yerelleştirilmiş adını (`LC_ITEM_NAME`) döndürür.
        *   `mob_name(mob_vnum)`: Canavar VNUM'una göre yerelleştirilmiş adını (`LC_MOB_NAME`) döndürür.
        *   `mob_vnum(canavar_adi)`: Canavar adına göre (`CMobManager::Get(name, false)`) VNUM'unu döndürür.
        *   `skill_name(skill_vnum)`: Beceri VNUM'una göre yerelleştirilmiş adını (`LC_SKILL_NAME`) döndürür.
        *   `get_global_time()` veya `get_time()`: Mevcut global sunucu zamanını (`::get_global_time()`) (Unix timestamp) döndürür.
        *   `get_channel_id()`: Sunucunun çalıştığı kanal ID'sini (global `g_bChannel`) döndürür.
        *   `under_han(string)`: Verilen string'in Korece Hangul altında olup olmadığını (`::under_han`) kontrol eder (özel karakter kontrolü).
        *   `get_locale_base_path()`: Yerel ayar dosyalarının temel dizin yolunu (`LocaleService_GetBasePath()`) döndürür.
        *   `get_special_item_group(grup_id)`: `ITEM_MANAGER::GetSpecialItemGroup` ile özel eşya grubundaki eşyaların VNUM ve sayılarını döndürür (çiftler halinde: vnum1, count1, vnum2, count2...).
    *   **Loglama:**
        *   `sys_log(seviye, mesaj)`: Sunucu sistem loguna (`::sys_log`) mesaj yazar (seviye >= 1 ise sadece test sunucusunda). Mevcut PC ve karakter bilgisi eklenir. Test sunucusunda karaktere de bilgi mesajı gönderilir.
        *   `sys_err(mesaj)`: `quest::CQuestManager::instance().QuestError` makrosu aracılığıyla sunucu hata loguna ve mevcut karaktere bilgi mesajı olarak mesaj yazar. Mevcut PC ve karakter bilgisi eklenir.
        *   `char_log(ne_oldu_kodu, nasil, ipucu)`: Mevcut karakter için `LogManager::instance().CharLog` ile log kaydı oluşturur.
        *   `item_log(item_id, nasil, ipucu)`: Belirtilen ID'ye sahip eşya için (`ITEM_MANAGER::Find` ile bulunur) `LogManager::instance().ItemLog` ile log kaydı oluşturur.
    *   **Karakter ve NPC İşlemleri:**
        *   `command(komut_string)`: Mevcut karakter adına bir oyun içi komut (`::interpret_command`) çalıştırır.
        *   `find_pc_by_name(oyuncu_adi)`: Oyuncu adına göre (`CHARACTER_MANAGER::FindPC`) karakteri bulur ve VID'sini döndürür (0 eğer bulunamazsa).
        *   `find_pc_cond(min_seviye, max_seviye, meslek_bayragi)`: Mevcut karakterin haritasında, belirtilen seviye aralığı ve meslek bayrağına uyan başka bir oyuncu (`CHARACTER_MANAGER::FindSpecifyPC`) arar ve VID'sini döndürür.
        *   `find_npc_by_vnum(vnum)`: Mevcut karakterin haritasında belirtilen VNUM'a sahip bir NPC arar (`CHARACTER_MANAGER::GetCharactersByRaceNum` ve harita kontrolü ile) ve ilk bulunanın VID'sini döndürür (0 eğer bulunamazsa).
        *   `spawn_mob(vnum, sayi, agresif_mi)`: Mevcut karakterin yakınına (`CHARACTER_MANAGER::SpawnMobRange`) belirtilen VNUM, sayı (1-10 arası) ve agresiflik ayarında canavar spawn eder. Başarıyla spawn edilen sayıyı döndürür.
        *   `block_chat(oyuncu_adi, sure_string)`: Global `do_block_chat` komutunu çağırarak belirtilen oyuncunun sohbetini belirtilen süre boyunca engeller.
    *   **Harita ve Alan İşlemleri:**
        *   `warp_all_to_village(map_index, saniye_cinsinden_gecikme)`: Belirli bir haritadaki tüm oyuncuları, belirtilen gecikme sonunda kendi imparatorluklarının başlangıç köyüne (`g_start_position`) ışınlamak için bir olay (`warp_all_to_village_event`) oluşturur. Haritaya duyuru gönderir.
        *   `warp_to_village()`: Mevcut karakteri kendi imparatorluğunun başlangıç köyüne (`g_start_position`) ışınlar.
        *   `kill_all_in_map(map_index)`: Belirli bir haritadaki tüm canavarları (PC, normal pet ve büyüme peti hariç) `Dead()` fonksiyonunu çağırarak öldürür.
        *   `regen_in_map(map_index, dosya_adi)`: Belirli bir haritaya belirtilen `regen` dosyasını (`regen_load_in_file`) yükler.
        *   `purge_area(x1, y1, x2, y2)`: Belirtilen koordinat alanındaki tüm canavarları ve taşları (`M2_DESTROY_CHARACTER`) yok eder. İşlemi yapan NPC hariç tutulur.
        *   `warp_all_in_area_to_area(from_x1, from_y1, from_x2, from_y2, to_x1, to_y1, to_x2, to_y2)`: Bir alandaki tüm PC'leri başka bir alandaki rastgele bir konuma ışınlar. Işınlanan karakter sayısını döndürür.
        *   `add_restart_city_pos(map_index, imparatorluk_id, x, y, z)`: `SECTREE_MANAGER::instance().AddRestartCityPos` ile belirli bir harita için yeniden başlama şehir pozisyonu ekler.
        *   `set_quest_flag_in_area(bayrak_adi, deger, x1, y1, x2, y2)`: Belirtilen alan içindeki tüm PC'ler için `PC::SetFlag` ile belirtilen görev bayrağını ayarlar.
        *   `find_boss_by_vnum(vnum)` (`__MT_THUNDER_DUNGEON__` tanımlıysa): Mevcut karakterin haritasında (`SECTREE_MAP::for_each` ile `FMobCounter` kullanarak) belirtilen VNUM'a sahip yaratık sayısını döndürür.
    *   **Görev Durumu:**
        *   `set_quest_state(gorev_adi, durum_adi)`: Mevcut oyuncunun (`PC`) belirtilen görev için durumunu (`PC::SetQuestState` veya `QuestState::st`, `PC::SetCurrentQuestStateName`) ayarlar. Koşan thread'in mevcut thread olup olmadığını kontrol eder.
        *   `get_quest_state(gorev_adi)`: Mevcut oyuncunun (`PC`) belirtilen görevin `gorev_adi.__status` bayrağının değerini (`PC::GetFlag`) döndürür.
    *   **Ayrıcalıklar (Privileges):**
        *   `__give_char_priv(tip, deger)`: Mevcut karaktere (`CPrivManager::RequestGiveCharacterPriv`) belirtilen tipte ve değerde bir ayrıcalık verir.
        *   `__give_empire_priv(imparatorluk_id, tip, deger, sure_saat)`: Belirli bir imparatorluğa (`CPrivManager::RequestGiveEmpirePriv`) ayrıcalık verir.
        *   `__give_guild_priv(lonca_id, tip, deger, sure_saat)`: Belirli bir loncaya (`CPrivManager::RequestGiveGuildPriv`) ayrıcalık verir.
        *   `__get_empire_priv_string(imparatorluk_id)`: Belirli bir imparatorluğun (`CPrivManager::GetPrivByEmpireEx`) aktif ayrıcalıklarını (isim, değer, kalan süre) string olarak döndürür.
        *   `__get_empire_priv(imparatorluk_id, tip)`: Belirli bir imparatorluğun (`CPrivManager::GetPrivByEmpire`) belirtilen tipteki ayrıcalık değerini döndürür.
        *   `__get_guild_priv_string(lonca_id)`: Belirli bir loncanın (`CPrivManager::GetPrivByGuildEx`) aktif ayrıcalıklarını string olarak döndürür.
        *   `__get_guildid_byname(lonca_adi)`: Lonca adına göre (`CGuildManager::FindGuildByName`) ID'sini döndürür.
        *   `__get_guild_priv(lonca_id, tip)`: Belirli bir loncanın (`CPrivManager::GetPrivByGuild`) belirtilen tipteki ayrıcalık değerini döndürür.
    *   **Sistem ve Oyun Ayarları:**
        *   `set_bgm_volume_enable()`: `CHARACTER_SetBGMVolumeEnable` çağırır.
        *   `add_bgm_info(map_index, bgm_dosya_adi, ses_seviyesi)`: `CHARACTER_AddBGMInfo` ile belirli bir harita için arka plan müziği bilgisi ekler.
        *   `add_goto_info(isim, imparatorluk_id, map_index, x, y)`: `CHARACTER_AddGotoInfo` ile `/goto` komutu için hızlı ışınlanma noktası ekler.
        *   `enable_over9refine(from_vnum, to_vnum)`: `COver9RefineManager::instance().enableOver9Refine` ile belirli eşyalar arasında +9 üzeri basma sistemini aktif eder.
        *   `add_ox_quiz(seviye, soru, cevap_dogru_mu)`: `COXEventManager::instance().AddQuiz` ile OX Event için yeni bir soru ekler.
    *   **Özel Sistem Fonksiyonları:**
        *   `__refine_pick()`: Madencilikte (`mining::RealRefinePick`) kazma geliştirmeyi dener. Envanterdeki hücreden eşyayı alır.
        *   `__fish_real_refine_rod()`: Balıkçılıkta (`fishing::RealRefineRod`) olta geliştirmeyi dener. Envanterdeki hücreden eşyayı alır.
*   **Kayıt:** `RegisterGlobalFunctionTable()` fonksiyonu, `global_functions` (`luaL_reg` türünde) dizisindeki tüm bu fonksiyonları `lua_register` kullanarak Lua'nın global state'ine kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `sstream`, `constants.h`, `char.h`, `char_manager.h`, `log.h`, `questmanager.h`, `questlua.h`, `questevent.h`, `config.h`, `mining.h`, `fishing.h`, `priv_manager.h`, `utils.h`, `p2p.h`, `item_manager.h`, `mob_manager.h`, `start_position.h`, `over9refine.h`, `OXEvent.h`, `regen.h`, `cmd.h`, `guild.h`, `guild_manager.h`, `sectree_manager.h`, `desc.h`.

### `questlua_guild.cpp`

*   **Amaç:** Lonca sistemiyle ilgili çeşitli işlevleri Lua betiklerine açar. Lonca bilgileri, sıralama, savaşlar, bahisler ve lonca yönetimi gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `guild`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Lonca Bilgileri ve Sıralama:**
        *   `guild.get_rank()`: Mevcut karakterin loncasının sıralamadaki yerini döndürür. Lonca yoksa -1 döner.
        *   `guild.get_ladder_point()`: Mevcut karakterin loncasının merdiven puanını (ladder point) döndürür. Lonca yoksa -1 döner.
        *   `guild.high_ranking_string()`: En yüksek sıralamaya sahip loncaların listesini (mevcut karakterin loncası vurgulanarak) bir string olarak döndürür.
        *   `guild.around_ranking_string()`: Mevcut karakterin loncasının etrafındaki sıralamaya sahip loncaların listesini bir string olarak döndürür. Lonca yoksa boş string döner.
        *   `guild.name(lonca_id)` veya `guild.get_name(lonca_id)`: Verilen ID'ye sahip loncanın adını döndürür. Lonca bulunamazsa boş string döner.
        *   `guild.level(lonca_id)`: Verilen ID'ye sahip loncanın seviyesini döndürür. Lonca bulunamazsa 0 döner.
        *   `guild.get_member_count()`: Mevcut karakterin loncasının üye sayısını döndürür. Lonca yoksa 0 döner.
    *   **Lonca Savaşları:**
        *   `guild.is_war(rakip_lonca_id)`: Mevcut karakterin loncasının, belirtilen ID'ye sahip lonca ile savaşta olup olmadığını kontrol eder (`true`/`false`).
        *   `guild.war_enter(rakip_lonca_id)`: Belirtilen ID'ye sahip loncadan gelen savaş teklifini kabul eder.
        *   `guild.get_any_war()`: Mevcut karakterin loncasının savaşta olduğu herhangi bir loncanın ID'sini döndürür (0 eğer savaşta değilse).
        *   `guild.get_reserve_war_table()`: Beklemedeki (reserve) lonca savaşlarının (sadece `GUILD_WAR_TYPE_BATTLE` tipi) bir tablosunu döndürür. Her alt tablo şunları içerir: [1]=Savaş ID, [2]=Güçlü Lonca ID, [3]=Zayıf Lonca ID, [4]=Handikap.
        *   `guild.get_warp_war_list()`: Işınlanılabilen aktif savaşlardaki loncaların listesini bir tablo olarak döndürür.
    *   **Savaş Bahisleri:**
        *   `guild.war_bet(savas_id, taraf_lonca_id, miktar)`: Belirtilen savaşa, belirtilen lonca tarafına, belirtilen altın miktarında bahis yapar (Veritabanına `HEADER_GD_GUILD_WAR_BET` paketi gönderir).
        *   `guild.is_bet(savas_id)`: Mevcut karakterin hesabının belirtilen savaşa bahis yapıp yapmadığını kontrol eder (`true`/`false`).
    *   **Lonca Yönetimi:**
        *   `guild.change_master(yeni_lider_adi)`: Lonca liderliğini, belirtilen isimdeki üyeye devreder (Basit versiyon). Sadece lider çağırabilir. Durum kodları döndürür (0: İsim geçersiz, 1: Çağıran lider değil, 2: Üye bulunamadı, 3: Başarılı, 4: Lonca yok).
        *   `guild.change_master_with_limit(yeni_lider_adi, min_seviye, istifa_limiti_sn, yeni_lider_olma_limiti_sn, uye_olma_limiti_sn, nakit_mi)`: Lonca liderliğini devrederken ek kontroller ve zaman limitleri uygular (yeni liderin seviyesi, istifa/lider olma/üye olma süre kısıtlamaları). Başarı durumunda ilgili görev bayraklarını (`change_guild_master.*`) ayarlar. Detaylı durum kodları döndürür ( önceki kodlara ek olarak 5: Yeni lider offline, 6: Yeni lider seviyesi düşük, 7: Yeni liderin lider olma kısıtlaması var).
*   **Kayıt:** `RegisterGuildFunctionTable()` fonksiyonu, `guild_functions` dizisindeki tüm fonksiyonları `guild` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `desc_client.h`, `char.h`, `char_manager.h`, `utils.h`, `guild.h`, `guild_manager.h`, `db.h` (DB Client bağlantısı için).

### `questlua_horse.cpp`

*   **Amaç:** Karakterin bineği (at) ile ilgili işlevleri Lua betiklerine açar. Ata binme/inme, atı çağırma/gönderme, atın seviyesi, sağlığı, dayanıklılığı, ismi gibi bilgilere erişim ve yönetimi sağlar.
*   **Lua Ön Eki:** `horse`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Çağırma ve Binme:**
        *   `horse.is_riding()`: Karakterin ata binili olup olmadığını kontrol eder (1 evet, 0 hayır).
        *   `horse.is_summon()`: Karakterin bir atının çağrılmış olup olmadığını kontrol eder (`true`/`false`).
        *   `horse.ride()`: Karakteri mevcut atına bindirir (`CHARACTER::StartRiding`).
        *   `horse.unride()`: Karakteri attan indirir (`CHARACTER::StopRiding`).
        *   `horse.summon(uzaktan_mi, vnum, isim)`: Karakterin atını çağırır (`CHARACTER::HorseSummon`).
            *   `uzaktan_mi` (bool, opsiyonel, varsayılan `false`): Atın uzaktan gelip gelmeyeceği.
            *   `vnum` (sayı, opsiyonel, varsayılan 0): Çağrılacak özel binek VNUM'u (0 ise karakterin kayıtlı atı).
            *   `isim` (string, opsiyonel, varsayılan `NULL`): Atın adı.
        *   `horse.unsummon()`: Karakterin çağrılmış atını geri gönderir (`CHARACTER::HorseSummon(false)`).
    *   **At Bilgileri:**
        *   `horse.is_mine()`: Mevcut NPC'nin (genellikle çağrılmış at), mevcut karakterin atı olup olmadığını kontrol eder (`true`/`false`).
        *   `horse.get_level()`: Karakterin atının seviyesini döndürür.
        *   `horse.get_grade()`: Karakterin atının derecesini (grade) döndürür.
        *   `horse.get_health()`: Karakterin atının mevcut sağlık puanını döndürür.
        *   `horse.get_health_pct()`: Karakterin atının mevcut sağlık yüzdesini (%0-100) döndürür.
        *   `horse.get_stamina()`: Karakterin atının mevcut dayanıklılık puanını döndürür.
        *   `horse.get_stamina_pct()`: Karakterin atının mevcut dayanıklılık yüzdesini (%0-100) döndürür.
        *   `horse.is_dead()`: Karakterin atının ölü olup olmadığını kontrol eder (`true`/`false`).
        *   `horse.get_name()`: Karakterin atının adını (`CHorseNameManager` üzerinden) döndürür. Adı yoksa boş string döner.
    *   **At Yönetimi:**
        *   `horse.set_level(seviye)`: Karakterin atının seviyesini ayarlar (`CHARACTER::SetHorseLevel`). Puanları yeniden hesaplar ve skill seviye paketini gönderir.
        *   `horse.advance()`: Karakterin atının seviyesini bir artırır. Maksimum seviyedeyse bir şey yapmaz.
        *   `horse.revive()`: Eğer at ölü ise canlandırır (`CHARACTER::ReviveHorse`).
        *   `horse.feed()`: Atı besler (`CHARACTER::FeedHorse`). Eğer atın sağlığı zaten maksimumda ise bilgi mesajı gösterir.
        *   `horse.set_name(isim)`: Karakterin atına isim verir. İsim geçerliliğini kontrol eder (`check_name`). Başarılı olursa, ismin geçerlilik süresi için görev bayrağı (`horse_name.valid_till`) ve efekt (`AFFECT_HORSE_NAME`) ekler, `CHorseNameManager` ile ismi günceller, atı yeniden çağırır. Durum kodu döndürür (0: At seviyesi yok, 1: İsim geçersiz, 2: Başarılı).
*   **Kayıt:** `RegisterHorseFunctionTable()` fonksiyonu, `horse_functions` dizisindeki tüm fonksiyonları `horse` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `horsename_manager.h`, `char.h`, `affect.h`, `config.h`, `utils.h`.

### `questlua_item.cpp`

*   **Amaç:** Görev betiklerinin eşyalarla (item) etkileşim kurmasını sağlar. Mevcut görevin işlem yaptığı eşyayı (`current item`) seçme, eşya bilgilerini (ID, VNUM, ad, seviye, soketler, değerler, bayraklar vb.) alma, eşya özelliklerini değiştirme, eşyayı silme ve +9 üzeri basma gibi işlemleri kapsar.
*   **Lua Ön Eki:** `item`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Eşya Seçimi ve Bilgi:**
        *   `item.select(item_id)`: Verilen ID'ye sahip eşyayı bulur ve mevcut işlem eşyası olarak ayarlar (`CQuestManager::SetCurrentItem`). Başarı durumunda `true` döner.
        *   `item.select_cell(cell_index)`: Mevcut karakterin envanterindeki belirtilen hücredeki eşyayı seçer ve mevcut işlem eşyası olarak ayarlar. Başarı durumunda `true` döner.
        *   `item.is_available()`: Mevcut bir işlem eşyasının seçili olup olmadığını kontrol eder (`true`/`false`).
        *   `item.get_id()`: Mevcut işlem eşyasının ID'sini döndürür (0 eğer seçili değilse).
        *   `item.get_cell()`: Mevcut işlem eşyasının envanterdeki hücre numarasını döndürür (0 eğer seçili değilse).
        *   `item.get_vnum()`: Mevcut işlem eşyasının VNUM'unu döndürür (0 eğer seçili değilse).
        *   `item.get_name()`: Mevcut işlem eşyasının yerelleştirilmiş adını (`GetLocaleName`) döndürür (boş string eğer seçili değilse).
        *   `item.get_count()`: Mevcut işlem eşyasının sayısını döndürür (0 eğer seçili değilse).
        *   `item.get_size()`: Mevcut işlem eşyasının envanterde kapladığı boyutu döndürür (0 eğer seçili değilse).
        *   `item.get_type()`: Mevcut işlem eşyasının türünü (`ITEM_TYPE_*`) döndürür (0 eğer seçili değilse).
        *   `item.get_sub_type()`: Mevcut işlem eşyasının alt türünü (`ITEM_SUBTYPE_*`) döndürür (0 eğer seçili değilse).
        *   `item.get_level()`: Mevcut işlem eşyasının geliştirme seviyesini (`GetRefineLevel`) döndürür (0 eğer seçili değilse).
        *   `item.get_level_limit()`: Mevcut işlem eşyasının seviye limitini (sadece silah ve zırh için) döndürür.
        *   `item.get_attribute_count()`: Mevcut işlem eşyasının sahip olduğu efsun sayısını döndürür (0 eğer seçili değilse).
        *   `item.get_changelook_vnum()` (`__CHANGE_LOOK_SYSTEM__` tanımlıysa): Mevcut işlem eşyasının görünüm değiştirme VNUM'unu (`GetTransmutationVnum`) döndürür (0 eğer seçili değilse veya değiştirilmemişse).
        *   `item.get_vnum_real_time(item_vnum)`: Verilen VNUM'a sahip eşyanın `LIMIT_REAL_TIME` limitine sahip olup olmadığını kontrol eder (`true`/`false`).
    *   **Soketler, Değerler ve Bayraklar:**
        *   `item.get_socket(socket_index)`: Mevcut işlem eşyasının belirtilen indeksteki (0-2) soket değerini döndürür (0 eğer seçili değilse veya indeks geçersizse).
        *   `item.set_socket(socket_index, value)`: Mevcut işlem eşyasının belirtilen indeksteki (0-2) soket değerini ayarlar.
        *   `item.get_value(value_index)`: Mevcut işlem eşyasının belirtilen indeksteki (0-5) değerini (`GetValue`) döndürür (0 eğer seçili değilse veya indeks geçersizse).
        *   `item.set_value(index, apply_type, apply_value)`: Mevcut işlem eşyasının belirtilen indeksteki (0-ITEM_ATTRIBUTE_MAX_NUM-1) efsununu (`SetForceAttribute`) ayarlar/değiştirir.
        *   `item.has_flag(flag_degeri)`: Mevcut işlem eşyasının belirtilen bayrağa (`ITEM_FLAG_*`) sahip olup olmadığını kontrol eder (`true`/`false`).
    *   **Eşya Yönetimi ve Geliştirme:**
        *   `item.remove()`: Mevcut işlem eşyasını oyundan siler (`ITEM_MANAGER::RemoveItem`). Sadece eşya sahibi mevcut karakter ise işlem yapar. Seçili eşyayı temizler.
        *   `item.get_refine_vnum()`: Mevcut işlem eşyasının başarılı geliştirme sonrası dönüşeceği VNUM'u (`GetRefinedVnum`) döndürür (0 eğer seçili değilse).
        *   `item.next_refine_vnum(item_vnum)`: Verilen VNUM'un bir sonraki geliştirme VNUM'unu (`TItemTable::dwRefinedVnum`) döndürür (0 eğer bulunamazsa).
        *   `item.can_over9refine()`: Mevcut işlem eşyasının +9 üzeri basılıp basılamayacağını (`COver9RefineManager::canOver9Refine`) kontrol eder (0/1).
        *   `item.change_to_over9()`: Mevcut işlem eşyasını +9 üzeri basma için dönüştürür (`COver9RefineManager::Change9ToOver9`). Başarı durumunu (`true`/`false`) döndürür.
        *   `item.over9refine()`: Mevcut işlem eşyasına +9 üzeri basma işlemi uygular (`COver9RefineManager::Over9Refine`). Başarı durumunu (`true`/`false`) döndürür.
        *   `item.get_over9_material_vnum(item_vnum)`: Verilen eşya VNUM'u için +9 üzeri basma materyalinin VNUM'unu (`COver9RefineManager::GetMaterialVnum`) döndürür.
        *   `item.start_realtime_expire()`: Mevcut işlem eşyası için gerçek zamanlı süre bitimi olayını başlatır (`ITEM::StartRealTimeExpireEvent`).
        *   `item.copy_and_give_before_remove(yeni_item_vnum)`: Mevcut işlem eşyasının tüm efsunlarını (soketler dahil) belirtilen `yeni_item_vnum` ile oluşturulan yeni bir eşyaya kopyalar, eski eşyayı siler ve yeni eşyayı aynı hücreye koyar. Başarı durumunu (`true`/`false`) döndürür.
*   **Kayıt:** `RegisterITEMFunctionTable()` fonksiyonu, `item_functions` dizisindeki tüm fonksiyonları `item` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `char.h`, `item.h`, `item_manager.h`, `over9refine.h`, `log.h`, `db.h`.

### `questlua_marriage.cpp`

*   **Amaç:** Evlilik sistemiyle ilgili işlevleri Lua betiklerine açar. Evlilik teklifi, evliliği sonlandırma, evli çifti bulma, düğün haritasına katılma/yönetme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `marriage`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Evlilik Yönetimi:**
        *   `marriage.engage_to(karakter_vid)`: Mevcut karakterden, belirtilen VID'ye sahip karaktere evlilik teklifi gönderir (`CManager::RequestAdd`).
        *   `marriage.remove()`: Mevcut karakterin evliliğini sonlandırmak için istek gönderir (`CManager::RequestRemove`).
        *   `marriage.set_to_marriage()`: Mevcut karakterin evlilik durumunu 'evli' olarak ayarlar (`TMarriage::SetMarried`). Genellikle düğün töreni sonrası kullanılır.
        *   `marriage.find_married_vid()`: Mevcut karakterin evli olduğu kişinin VID'sini döndürür (0 eğer evli değilse veya eş offline ise).
        *   `marriage.get_married_time()`: Evliliğin başlangıcından bu yana geçen süreyi saniye cinsinden döndürür (0 eğer evli değilse).
    *   **Düğün Haritası Yönetimi:**
        *   `marriage.get_wedding_list()`: Devam eden düğünlerin listesini bir Lua tablosu olarak döndürür. Her alt tablo şunları içerir: [1]=Damat PID, [2]=Gelin PID, [3]=Damat Adı, [4]=Gelin Adı.
        *   `marriage.join_wedding(damat_pid, gelin_pid)`: Mevcut karakteri, belirtilen çiftin düğün haritasına ışınlar (`TMarriage::WarpToWeddingMap`). Çiftin gerçekten evli olup olmadığını kontrol eder.
        *   `marriage.warp_to_my_marriage_map()`: Mevcut karakteri, kendi düğün haritasına ışınlar (`TMarriage::WarpToWeddingMap`).
        *   `marriage.end_wedding()`: Mevcut karakterin (eğer ev sahibi ise) düğününü sonlandırmak için istek gönderir (`TMarriage::RequestEndWedding`).
        *   `marriage.wedding_dark(karanlik_mi)`: Düğün haritasını karanlık yapar veya aydınlatır (`WeddingMap::SetDark`).
        *   `marriage.wedding_snow(kar_yagsin_mi)`: Düğün haritasında kar yağdırır veya durdurur (`WeddingMap::SetSnow`).
        *   `marriage.wedding_music(calinsin_mi, muzik_dosya_adi)`: Düğün haritasında belirtilen müziği çalar veya durdurur (`WeddingMap::SetMusic`).
        *   `marriage.wedding_is_playing_music()`: Düğün haritasında müzik çalınıp çalınmadığını kontrol eder (`true`/`false`).
        *   `marriage.wedding_client_command(komut_string)`: Düğün haritasındaki tüm oyunculara belirtilen komut string'ini gönderir (`WeddingMap::ShoutInMap(CHAT_TYPE_COMMAND, ...)`).
        *   `marriage.in_my_wedding()`: Mevcut karakterin kendi düğün haritasında olup olmadığını kontrol eder (`true`/`false`).
*   **Kayıt:** `RegisterMarriageFunctionTable()` fonksiyonu, `marriage_functions` dizisindeki tüm fonksiyonları `marriage` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `char_manager.h`, `wedding.h`, `questmanager.h`, `utils.h`, `config.h`.

### `questlua_meleylair.cpp`

*   **Amaç:** (`__GUILD_DRAGONLAIR__` makrosu tanımlıysa) Meley'in İni (Ejderha İni) zindanıyla ilgili işlevleri Lua betiklerine açar. Zindana kayıt olma, girme, kontrol etme, ödül alma ve sıralama gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `MeleyLair`
*   **Koşullu Derleme:** Bu dosyadaki fonksiyonlar yalnızca `__GUILD_DRAGONLAIR__` makrosu tanımlı olduğunda kullanılabilir.
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `MeleyLair.GetRequirments()`: Zindana giriş için gereken minimum seviyeyi (`MeleyLair::MIN_LVL`) ve lonca merdiven puanı maliyetini (`MeleyLair::LADDER_POINTS_COST`) döndürür.
    *   `MeleyLair.GetParticipantsLimit()`: Zindana katılabilecek maksimum oyuncu sayısını (`MeleyLair::PARTICIPANTS_LIMIT`) döndürür.
    *   `MeleyLair.GetSubMapIndex()`: Meley'in İni zindanının harita indeksini (`MeleyLair::SUBMAP_INDEX`) döndürür.
    *   `MeleyLair.Register()`: Mevcut karakterin loncasını Meley'in İni'ne kaydeder (`CMgr::Register`). Dönüş değerleri: [1] 0=Başarılı, 1=Lonca yok, 2=Lider değil, 3=Zaten kayıtlı, 4=Seviye düşük, 5=Puan yetersiz, 6=Bekleme süresi. [2] İlk dönüş değeri 6 ise kalan bekleme süresi.
    *   `MeleyLair.IsRegistered()`: Mevcut karakterin loncasının kayıtlı olup olmadığını ve kaçıncı sırada olduğunu (`CMgr::isRegistered`) döndürür (bool, sıra_no).
    *   `MeleyLair.Enter()`: Mevcut karakteri, loncası kayıtlıysa ve sıra geldiyse Meley'in İni'ne sokar (`CMgr::Enter`). Başarı durumunu ve eğer başarısızsa sebebini (0=Başarılı, 1=Lonca yok, 2=Kayıtlı değil, 3=Sıra gelmedi, 4=Limit dolu, 5=Başka bir Meley haritası açık, 6=Zindan oluşturulamadı, 7=Yanlış harita) döndürür (bool, sonuç_kodu).
    *   `MeleyLair.IsMeleyMap()`: Mevcut karakterin bulunduğu haritanın bir Meley İni haritası olup olmadığını kontrol eder (`CMgr::IsMeleyMap`).
    *   `MeleyLair.Check()`: Zindanın durumunu kontrol eder (muhtemelen zaman aşımı, katılımcı sayısı vb.) (`CMgr::Check`).
    *   `MeleyLair.Leave()`: Mevcut karakterin loncasını ve karakteri zindandan çıkarmak için istek gönderir (`CMgr::LeaveRequest`).
    *   `MeleyLair.CanGetReward()`: Mevcut karakterin loncasının ödül alıp alamayacağını kontrol eder (`CMgr::CanGetReward`).
    *   `MeleyLair.Reward(odul_tipi)`: Mevcut karaktere ve loncasına belirtilen tipte ödülü verir (`CMgr::Reward`).
    *   `MeleyLair.OpenRanking()`: Mevcut karaktere Meley İni sıralamasını gösterir (`CMgr::OpenRanking`).
*   **Kayıt:** `RegisterMeleyLairFunctionTable()` fonksiyonu, `functions` dizisindeki tüm fonksiyonları `MeleyLair` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `MeleyLair.h`, `char.h`, `guild.h`.

### `questlua_mgmt.cpp`

*   **Amaç:** Sunucu yönetimiyle ilgili, özellikle krallık (monarch) sistemine odaklanan işlevleri Lua betiklerine açar. Krallık bilgilerini sorgulama ve kralı değiştirme gibi fonksiyonlar sunar.
*   **Lua Ön Eki:** `mgmt`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `mgmt.monarch_state(imparatorluk_index)`: Belirtilen imparatorluğun (1, 2 veya 3) kral bilgilerini (`TMonarchInfo` yapısından) döndürür: Kralın adı (string), Kralın PID'si (sayı), Seçilme tarihi (string), Krallık hazinesi (sayı).
    *   `mgmt.monarch_change_lord(imparatorluk_index, yeni_kral_pid)`: Belirtilen imparatorluğun kralını, verilen PID'ye sahip oyuncu ile değiştirmek için veritabanına istek (`HEADER_GD_CHANGE_MONARCH_LORD`) gönderir.
*   **Kayıt:** `RegisterMgmtFunctionTable()` fonksiyonu, `mgmt_functions` dizisindeki fonksiyonları `mgmt` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `questmanager.h`, `monarch.h`, `desc_client.h`.

### `questlua_monarch.cpp`

*   **Amaç:** Krallık (hükümdar/monarch) sistemiyle ilgili çeşitli işlevleri Lua betiklerine açar. Hükümdara özel yetenekler (can basma, güçlendirme), yaratık çağırma, oyuncu ışınlama, duyuru yapma, vergi toplama ve kale yönetimi gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `oh` (Dosya adı `monarch` olsa da Lua tablosu `oh` olarak kaydedilmiştir. Muhtemelen "Official Helper" veya benzeri bir anlama gelebilir ya da eski bir isimlendirme kalıntısıdır.)
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Hükümdar Yetenekleri ve Yönetimi:**
        *   `oh.takemonarchmoney(miktar)`: Mevcut karakterin imparatorluğunun hazinesinden belirtilen miktarda parayı (Yang * 10000) alır. Sadece veritabanına istek gönderir, paranın gerçekten alınıp alınmadığını kontrol etmez.
        *   `oh.isguildmaster()`: Mevcut karakterin bir lonca lideri (veya rütbesi <= 4 olan bir üye) olup olmadığını kontrol eder (1 evet, 0 hayır).
        *   `oh.ismonarch()`: Mevcut karakterin hükümdar olup olmadığını kontrol eder (1 evet, 0 hayır).
        *   `oh.monarchbless()`: Hükümdarın "Kutsama" yeteneğini kullanır. Belirli bir altın maliyeti (`MonarchHealGold` event flag'i veya 2M Yang) karşılığında, hükümdarın bulunduğu haritadaki kendi imparatorluğuna ait oyuncuların HP ve SP'lerini doldurur. Sadece hükümdar veya GM kullanabilir.
        *   `oh.monarchpowerup()`: Hükümdarın "Aslan Kükremesi" (Lion's Roar) yeteneğini kullanır. Belirli bir altın maliyeti (5M Yang) ve bekleme süresi sonrası, hükümdarın bulunduğu haritadaki kendi imparatorluğuna ait oyunculara 3 dakika boyunca +%10 saldırı gücü verir. Sadece hükümdar veya GM kullanabilir.
        *   `oh.monarchdefenseup()`: Hükümdarın "Altın Zırh" (Golden Armor) yeteneğini kullanır. Belirli bir altın maliyeti (5M Yang) ve bekleme süresi sonrası, hükümdarın bulunduğu haritadaki kendi imparatorluğuna ait oyunculara 3 dakika boyunca +%10 savunma verir. Sadece hükümdar veya GM kullanabilir.
        *   `oh.notice(mesaj)`: Hükümdarın kendi imparatorluğuna özel bir duyuru (`SendMonarchNotice`) yapmasını sağlar. Sadece hükümdar kullanabilir.
        *   `oh.info()`: Tüm imparatorlukların hükümdar bilgilerini (isim, hazine) sohbet penceresinde gösterir.
    *   **Yaratık ve Muhafız Çağırma:**
        *   `oh.spawnmob(mob_vnum)`: Belirtilen VNUM'a sahip bir yaratığı hükümdarın bulunduğu yere çağırır. Oluşan yaratığın VID'sini döndürür. Sadece hükümdar veya GM kullanabilir.
        *   `oh.spawnguard(grup_vnum, bolge_index)`: Belirtilen muhafız grubunu (VNUM ile) imparatorluğun kalesindeki belirtilen bölgeye çağırır. Çağırma maliyeti krallık hazinesinden düşülür. Sadece hükümdar veya GM, kendi kalesindeyken kullanabilir.
        *   `oh.monarch_mob(mob_vnum_string)`: `do_monarch_mob` komutunu çağırarak canavar çağırır (genellikle boss).
    *   **Işınlama ve Oyuncu Transferi:**
        *   `oh.warp(oyuncu_adi)`: Hükümdarı, aynı imparatorluktaki belirtilen oyuncunun yanına ışınlar. Belirli bir maliyeti (10k Yang) ve bekleme süresi vardır. Sadece izin verilen haritalara ışınlanılabilir.
        *   `oh.transfer(oyuncu_adi)`: Belirtilen oyuncuyu (aynı imparatorluktan ve aynı kanaldan) hükümdarın yanına ışınlar. Maliyeti ve bekleme süresi vardır. Sadece izin verilen haritalarda çalışır. Oyuncu farklı kanaldaysa veya offline ise P2P ile istek gönderir.
        *   `oh.transfer2(oyuncu_adi)`: `oh.transfer`'a benzer şekilde çalışır ancak hedef oyuncuya bir kabul/red penceresi (`monarch_transfer` görevi) gönderir.
    *   **Kale Yönetimi (Frog event ile ilgili, muhtemelen eski bir özellik):**
        *   `oh.frog_to_empire_money()`: (Yorum satırı içinde) Kale ile ilgili bir etkinlikten (altın kurbağa) elde edilen geliri imparatorluk hazinesine aktarır.
*   **Olaylar (Events):**
    *   `monarch_powerup_event`: `oh.monarchpowerup` ile verilen saldırı gücü etkisinin süresi dolduğunda etkiyi kaldırır.
    *   `monarch_defenseup_event`: `oh.monarchdefenseup` ile verilen savunma etkisinin süresi dolduğunda etkiyi kaldırır.
    *   `monarch_transfer2_event`: `oh.transfer2` ile ışınlama isteği gönderilen oyuncuya görev aracılığıyla bildirim gönderir.
*   **Kayıt:** `RegisterMonarchFunctionTable()` fonksiyonu, `Monarch_functions` dizisindeki tüm fonksiyonları `oh` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `monarch.h`, `desc_client.h`, `start_position.h`, `config.h`, `mob_manager.h`, `castle.h`, `dev_log.h`, `char.h`, `char_manager.h`, `utils.h`, `p2p.h`, `guild.h`, `sectree_manager.h`.

### `questlua_npc.cpp`

*   **Amaç:** Görev betiklerinin NPC'lerle (Non-Player Character) etkileşim kurmasını sağlayan çeşitli işlevleri Lua'ya açar. NPC bilgilerini alma (ırk, imparatorluk, lonca, seviye, isim, VID, PID, IP), NPC dükkanını açma, NPC'yi öldürme/silme, oyuncuya yakınlığını kontrol etme, NPC'yi kilitleme/kilidi açma, saldırı/hasar çarpanlarını ayarlama ve özel etkinlik (Noel Baba) NPC'leri için fonksiyonlar içerir.
*   **Lua Ön Eki:** `npc`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **NPC Bilgileri ve Durumu:**
        *   `npc.getrace()` veya `npc.get_race()`: Mevcut NPC'nin ırk (VNUM) numarasını döndürür.
        *   `npc.get_empire()`: Mevcut NPC'nin imparatorluğunu döndürür (0 eğer NPC ise, PC ise karakterin imparatorluğu).
        *   `npc.is_pc()`: Mevcut NPC'nin bir oyuncu karakteri olup olmadığını kontrol eder (`true`/`false`).
        *   `npc.get_guild()`: Mevcut NPC'nin (eğer bir PC ise) lonca ID'sini döndürür (0 eğer loncası yoksa veya NPC ise).
        *   `npc.is_quest()`: Mevcut NPC'nin, mevcut göreve (`GetCurrentQuestName`) atanmış NPC olup olmadığını kontrol eder (`true`/`false`).
        *   `npc.get_vid()`: Mevcut NPC'nin VID (Virtual ID) değerini döndürür.
        *   `npc.get_name()`: Mevcut NPC'nin adını döndürür.
        *   `npc.get_level()`: Mevcut NPC'nin seviyesini döndürür.
        *   `npc.get_rank()`: Mevcut NPC'nin canavar rütbesini (`MOB_RANK_*`) döndürür.
        *   `npc.get_type()`: Mevcut NPC'nin türünü (`CHAR_TYPE_*`) döndürür.
        *   `npc.is_metin()`: Mevcut NPC'nin bir metin taşı olup olmadığını kontrol eder (`true`/`false`).
        *   `npc.is_boss()`: Mevcut NPC'nin bir boss (`MOB_RANK_BOSS`) olup olmadığını kontrol eder (`true`/`false`).
        *   `npc.get_job()`: Mevcut NPC bir oyuncu ise mesleğini döndürür, değilse -1 döner.
        *   `npc.get_pid()`: Mevcut NPC'nin oyuncu ID'sini döndürür (NPC ise genellikle 0 veya kendi ID'si).
        *   `npc.get_exp()`: Mevcut NPC'nin (eğer bir canavarsa) verdiği tecrübe puanını döndürür.
        *   `npc.get_ip()`: Mevcut NPC bir oyuncu ise IP adresini döndürür, değilse boş string döner.
        *   `npc.get_vnum0()`: Mevcut NPC'nin ırk (VNUM) numarasını döndürür (bkz: `npc.get_race()`).
    *   **NPC Etkileşimleri:**
        *   `npc.open_shop(shop_vnum)`: Mevcut NPC'ye ait belirtilen VNUM'daki dükkanı mevcut oyuncu için açar. `shop_vnum` verilmezse NPC'nin varsayılan dükkanını (VNUM 0) açar.
        *   `npc.kill()`: Mevcut NPC'yi öldürür (`Dead()` fonksiyonunu çağırır) ve oyuncunun görev NPC ID'sini sıfırlar.
        *   `npc.purge()`: Mevcut NPC'yi oyundan tamamen siler (`M2_DESTROY_CHARACTER`) ve oyuncunun görev NPC ID'sini sıfırlar.
        *   `npc.is_near(mesafe)`: Mevcut oyuncunun mevcut NPC'ye belirtilen `mesafe` (metre cinsinden, varsayılan 10) içinde olup olmadığını kontrol eder (`true`/`false`).
        *   `npc.is_near_vid(karakter_vid, mesafe)`: Belirtilen VID'ye sahip karakterin mevcut NPC'ye `mesafe` içinde olup olmadığını kontrol eder.
        *   `npc.lock()`: Mevcut NPC'yi mevcut oyuncuya kilitler (diğer oyuncular etkileşime giremez). NPC zaten başkasına kilitliyse `false` döner.
        *   `npc.unlock()`: Mevcut NPC'nin mevcut oyuncu tarafından konulmuş kilidini açar.
        *   `npc.get_leader_vid()`: Mevcut NPC bir partideyse, parti liderinin VID'sini döndürür (0 eğer partide değilse veya lider yoksa).
        *   `npc.show_effect_on_target(efekt_dosya_yolu)`: Mevcut NPC üzerinde (eğer hedef NPC, çağıran oyuncudan farklıysa) belirtilen parçacık efektini gösterir.
    *   **Çarpan Ayarları (Hedef VID ile):**
        *   `npc.get_vid_attack_mul(hedef_vid)`: Belirtilen VID'ye sahip karakterin saldırı çarpanını (`GetAttMul`) döndürür.
        *   `npc.set_vid_attack_mul(hedef_vid, carpan_degeri)`: Belirtilen VID'ye sahip karakterin saldırı çarpanını ayarlar.
        *   `npc.get_vid_damage_mul(hedef_vid)`: Belirtilen VID'ye sahip karakterin hasar çarpanını (`GetDamMul`) döndürür.
        *   `npc.set_vid_damage_mul(hedef_vid, carpan_degeri)`: Belirtilen VID'ye sahip karakterin hasar çarpanını ayarlar.
    *   **Özel Etkinlik Fonksiyonları (Noel Baba - VNUM: `xmas::MOB_SANTA_VNUM`):**
        *   `npc.get_remain_skill_book_count()`: Noel Baba NPC'sinin verebileceği kalan beceri kitabı sayısını (`POINT_ATT_GRADE_BONUS`) döndürür.
        *   `npc.dec_remain_skill_book_count()`: Noel Baba NPC'sinin verebileceği kalan beceri kitabı sayısını bir azaltır.
        *   `npc.get_remain_hairdye_count()`: Noel Baba NPC'sinin verebileceği kalan saç boyası sayısını (`POINT_DEF_GRADE_BONUS`) döndürür.
        *   `npc.dec_remain_hairdye_count()`: Noel Baba NPC'sinin verebileceği kalan saç boyası sayısını bir azaltır.
*   **Kayıt:** `RegisterNPCFunctionTable()` fonksiyonu, `npc_functions` dizisindeki tüm fonksiyonları `npc` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `questmanager.h`, `char.h`, `party.h`, `xmas_event.h`, `char_manager.h`, `shop_manager.h`, `guild.h`, `desc.h`.

### `questlua_oxevent.cpp`

*   **Amaç:** OX Event (Bilgi Yarışması) sistemiyle ilgili işlevleri Lua betiklerine açar. Etkinliğin durumunu sorgulama, etkinliği başlatma/kapatma, soru sorma, katılımcı sayısını alma, etkinliği sonlandırma ve ödül verme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `oxevent`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `oxevent.get_status()`: OX Event'in mevcut durumunu (`OXEventStatus` enum: `OXEVENT_OPEN`, `OXEVENT_CLOSE`, `OXEVENT_QUIZ`, `OXEVENT_FINISH`) sayısal olarak döndürür.
    *   `oxevent.open()`: OX Event'i başlatır. `oxquiz.lua` betiğini çalıştırarak soruları yükler ve etkinliğin durumunu `OXEVENT_OPEN` yapar. Betik yüklenemezse 0, başarılı olursa 1 döndürür.
    *   `oxevent.close()`: OX Event girişlerini kapatır ve durumu `OXEVENT_CLOSE` yapar.
    *   `oxevent.quiz(soru_no, sure_saniye)`: Belirtilen numaralı soruyu sorar ve cevap için `sure_saniye` kadar bekler. Durumu `OXEVENT_QUIZ` yapar. Başarılıysa 1, soru bulunamazsa 0 döndürür.
    *   `oxevent.get_attender()`: OX Event'e katılan oyuncu sayısını döndürür.
    *   `oxevent.end_event()`: OX Event'i sonlandırma sürecini başlatır. Durumu `OXEVENT_FINISH` yapar ve 5 saniye sonra `COXEventManager::CloseEvent()` fonksiyonunu çağıran bir zamanlayıcı (`end_oxevent`) kurar. Bu fonksiyon, kazananları ve kalanları haritadan ışınlar.
    *   `oxevent.end_event_force()`: OX Event'i derhal sonlandırır (`COXEventManager::CloseEvent()` çağırır) ve durumu `OXEVENT_FINISH` yapar.
    *   `oxevent.give_item(item_vnum, item_sayisi)`: OX Event'e katılan (ve muhtemelen kazanan) tüm oyunculara belirtilen VNUM ve sayıda eşyayı verir.
*   **Olaylar (Events):**
    *   `end_oxevent`: `oxevent.end_event` tarafından kurulur ve süre sonunda etkinliği tamamen kapatır.
*   **Kayıt:** `RegisterOXEventFunctionTable()` fonksiyonu, `oxevent_functions` dizisindeki tüm fonksiyonları `oxevent` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `char.h`, `char_manager.h`, `OXEvent.h`, `config.h`, `locale_service.h`.

### `questlua_party.cpp`

*   **Amaç:** Oyuncuların oluşturduğu partilerle (gruplarla) ilgili işlevleri Lua betiklerine açar. Parti lideri olma durumu, parti üye sayısı, parti sohbeti, parti bayrakları, sinematik gösterme, partiye özel efekt verme ve zindan içi kontroller gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `party`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Parti Bilgileri ve Durumu:**
        *   `party.is_leader()`: Mevcut karakterin parti lideri olup olmadığını kontrol eder (`true`/`false`).
        *   `party.is_party()`: Mevcut karakterin bir partide olup olmadığını kontrol eder (`true`/`false`).
        *   `party.get_leader_pid()`: Mevcut karakterin partisinin liderinin PID'sini döndürür. Partide değilse -1 döner.
        *   `party.get_near_count()`: Mevcut karakterin partisindeki yakındaki (aynı haritada ve belirli bir mesafedeki) üye sayısını döndürür. Partide değilse 0 döner.
        *   `party.get_max_level()`: Mevcut karakterin partisindeki en yüksek seviyeli üyenin seviyesini döndürür. Partide değilse 1 döner.
        *   `party.is_in_dungeon()`: Mevcut karakterin partisinin bir zindanda olup olmadığını kontrol eder (`true`/`false`).
        *   `party.get_member_pids()`: Mevcut karakterin bulunduğu haritadaki tüm parti üyelerinin PID'lerini bir Lua tablosu olarak döndürür.
    *   **Parti İçi Etkileşim ve Komutlar:**
        *   `party.chat(mesaj)`: Mevcut karakterin partisine normal sohbet (`CHAT_TYPE_TALKING`) mesajı gönderir.
        *   `party.syschat(mesaj)`: Mevcut karakterin partisine sistem (`CHAT_TYPE_INFO`) mesajı gönderir.
        *   `party.show_cinematic(script_mesaji)`: Mevcut karakterin partisindeki yakındaki üyelere sinematik bir görev penceresi (`QUEST_SKIN_CINEMATIC`) gösterir.
        *   `party.run_cinematic(script_degeri)`: `party.show_cinematic`'e benzer, ancak script değeri `[RUN_CINEMA value;...]` formatında oluşturulur.
        *   `party.clear_ready()`: Mevcut karakterin partisindeki yakındaki üyelerin "hazır" durumunu temizler (muhtemelen bir sonraki aşamaya geçiş için kullanılır).
    *   **Parti Bayrakları (Flags):**
        *   `party.setf(bayrak_adi, deger)`: Partiye ait genel bir bayrağın değerini ayarlar.
        *   `party.getf(bayrak_adi)`: Partiye ait genel bir bayrağın değerini alır.
        *   `party.setqf(gorev_bayrak_adi, deger)`: Mevcut görevin adıyla birleştirilmiş bir bayrak adıyla (`mevcut_gorev_adi.gorev_bayrak_adi`) partideki tüm online üyelere görev bayrağı atar. Partide değilse sadece mevcut karaktere atar.
        *   `party.is_map_member_flag_lt(gorev_bayrak_adi, deger)`: Mevcut karakterin bulunduğu haritadaki parti üyelerinin, belirtilen görev bayrağı değerinin (`mevcut_gorev_adi.gorev_bayrak_adi`) verilen `deger`den küçük olup olmadığını kontrol eder. Tüm üyeler için koşul sağlanırsa `true` döner.
    *   **Partiye Etki Verme:**
        *   `party.give_buff(etki_tipi, uygulama_yeri, uygulama_degeri, bayraklar, sure, sp_maliyeti, uzerine_yazilsin_mi, kup_mu)`: Mevcut karakterin bulunduğu haritadaki parti üyelerine (veya partide değilse sadece mevcut karaktere) belirtilen özelliklerde bir etki (buff/debuff) verir.
            *   `etki_tipi` (sayı): `AFFECT_*` sabitlerinden biri.
            *   `uygulama_yeri` (sayı): `APPLY_*` sabitlerinden biri.
            *   `uygulama_degeri` (sayı): Etkinin sayısal değeri.
            *   `bayraklar` (sayı): `AFF_*` sabitlerinden biri (efektin özel davranışları).
            *   `sure` (sayı): Saniye cinsinden etki süresi.
            *   `sp_maliyeti` (sayı): Etkinin SP maliyeti (genellikle 0).
            *   `uzerine_yazilsin_mi` (bool): Aynı türden bir efekt varsa üzerine yazılsın mı?
            *   `kup_mu` (bool): Bu efekt bir küp (cube) tarafından mı verildi?
*   **Yapılar (Structs) ve Functor'lar:**
    *   `FRunCinematicSender`, `FCinematicSender`: Partideki üyelere sinematik script göndermek için kullanılan functor'lar.
    *   `FGiveBuff`: Partideki üyelere efekt vermek için kullanılan functor.
    *   `FPartyPIDCollector`: Partideki üyelerin PID'lerini toplamak için kullanılan functor.
*   **Kayıt:** `RegisterPartyFunctionTable()` fonksiyonu, `party_functions` dizisindeki tüm fonksiyonları `party` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `sstream`, `desc.h`, `party.h`, `char.h`, `questlua.h`, `questmanager.h`, `packet.h`.

### `questlua_pc.cpp`

*   **Amaç:** Oyuncu karakteri (PC) ile ilgili çok sayıda işlevi Lua betiklerine açar. Karakter bilgileri (seviye, HP, SP, para, statüler, bayraklar, konum, envanter, yetenekler, binek, evlilik durumu, GM seviyesi vb.), eşya verme/alma/kontrol etme, ışınlama, görev bayrağı yönetimi, yetenek ve statü yönetimi, premium özellikler, kostüm sistemleri ve daha birçok PC odaklı mekaniği kapsar.
*   **Lua Ön Eki:** `pc`
*   **Temel İşlevler/Örnek Fonksiyonlar (Kategorilere Ayrılmış):**
    *   **Karakter Bilgileri ve Durumu:**
        *   `pc.get_level()`: Karakterin seviyesini döndürür.
        *   `pc.get_exp()`: Karakterin mevcut tecrübe puanını döndürür.
        *   `pc.get_next_exp()`: Bir sonraki seviye için gereken tecrübe puanını döndürür.
        *   `pc.get_hp()`, `pc.getmaxhp()`, `pc.get_max_hp()`: Mevcut/maksimum HP.
        *   `pc.get_sp()`, `pc.getmaxsp()`, `pc.get_max_sp()`: Mevcut/maksimum SP.
        *   `pc.get_money()`, `pc.getgold()`, `pc.get_gold()`: Karakterin Yang miktarını döndürür.
        *   `pc.get_cheque()`: (`__CHEQUE_SYSTEM__` tanımlıysa) Karakterin Won miktarını döndürür.
        *   `pc.get_name()`: Karakterin adını döndürür.
        *   `pc.get_vid()`: Karakterin VID'sini döndürür.
        *   `pc.get_job()`: Karakterin mesleğini (JOB_*) döndürür.
        *   `pc.get_race()`: Karakterin ırk numarasını döndürür.
        *   `pc.get_sex()`: Karakterin cinsiyetini (0: Erkek, 1: Kadın) döndürür.
        *   `pc.get_empire()`: Karakterin imparatorluğunu döndürür.
        *   `pc.get_alignment()`, `pc.get_real_alignment()`: Karakterin mevcut/gerçek sıralama puanını (/10) döndürür.
        *   `pc.get_ht()`, `pc.get_iq()`, `pc.get_st()`, `pc.get_dx()`: Karakterin temel statü (VIT, INT, STR, DEX) değerlerini döndürür.
        *   `pc.get_skill_point()`: Karakterin kalan yetenek puanlarını döndürür.
        *   `pc.get_playtime()`: Karakterin toplam oynama süresini (saniye) döndürür.
        *   `pc.get_current_map_index()`, `pc.get_map_index()`: Karakterin bulunduğu haritanın indeksini döndürür.
        *   `pc.get_x()`, `pc.get_y()`: Karakterin genel x, y koordinatlarını (/100) döndürür.
        *   `pc.get_local_x()`, `pc.get_local_y()`: Karakterin harita içindeki yerel x, y koordinatlarını (/100) döndürür.
        *   `pc.get_start_location()`: Karakterin başlangıç harita indeksini ve x, y koordinatlarını döndürür.
        *   `pc.get_channel_id()`: Sunucunun çalıştığı kanal ID'sini döndürür.
        *   `pc.get_ip()`: Karakterin IP adresini döndürür.
        *   `pc.get_last_damage()`: Karakterin aldığı son hasar miktarını döndürür.
        *   `pc.get_language()`: Karakterin istemci dilini döndürür (örn: "tr", "en").
        *   `pc.is_gm()`: Karakterin GM (Game Master) olup olmadığını kontrol eder (GM_HIGH_WIZARD seviyesi ve üzeri).
        *   `pc.get_gm_level()`: Karakterin GM seviyesini döndürür.
        *   `pc.is_dead()`: Karakterin ölü olup olmadığını kontrol eder.
        *   `pc.is_riding()`: Karakterin bir bineğe binili olup olmadığını kontrol eder.
        *   `pc.can_warp()`: Karakterin ışınlanıp ışınlanamayacağını kontrol eder.
        *   `pc.exchanging()`: Karakterin ticaret yapıp yapmadığını kontrol eder.
        *   `pc.in_dungeon()`: Karakterin bir zindanda olup olmadığını kontrol eder.
        *   `pc.has_guild()`: Karakterin bir loncası olup olmadığını kontrol eder.
        *   `pc.get_guild()`: Karakterin lonca ID'sini döndürür.
        *   `pc.is_guild_master()`: Karakterin lonca lideri olup olmadığını kontrol eder.
    *   **Eşya Yönetimi:**
        *   `pc.give_item(sebep_string, vnum_veya_isim, sayi)`: Karakterin envanterine eşya verir (sadece `PC::GiveItem` çağırır, düşürmez).
        *   `pc.give_item2(vnum_veya_isim, sayi)`: Eşyayı karaktere verir, envanter doluysa yere atar. Verilen eşyanın ID'sini döndürür.
        *   `pc.give_item2_select(vnum_veya_isim, sayi)`: `pc.give_item2` gibi, ancak verilen eşyayı mevcut görev eşyası olarak seçer.
        *   `pc.give_item_from_special_item_group(grup_vnum)`: Özel eşya grubundan rastgele bir eşya verir.
        *   `pc.give_poly_marble(canavar_vnum)`: Belirtilen canavarın dönüşüm küresini verir.
        *   `pc.count_item(vnum_veya_isim)`: Envanterdeki belirtilen eşya sayısını döndürür.
        *   `pc.remove_item(vnum_veya_isim, sayi_opsiyonel)`: Envanterden belirtilen eşyayı siler.
        *   `pc.enough_inventory(item_vnum)`: Belirtilen eşya için envanterde yeterli yer olup olmadığını kontrol eder.
        *   `pc.get_empty_inventory_count()`: Boş envanter slotu sayısını döndürür.
        *   `pc.get_weapon()`: Giyili silahın VNUM'unu döndürür (0 yoksa).
        *   `pc.get_armor()`: Giyili zırhın VNUM'unu döndürür (0 yoksa).
        *   `pc.get_wear(slot_index)`: Belirtilen ekipman slotundaki eşyanın VNUM'unu döndürür.
        *   `pc.get_equip_refine_level(slot_index)`: Belirtilen ekipman slotundaki eşyanın geliştirme seviyesini döndürür.
        *   `pc.refine_equip(slot_index, max_seviye_limiti, basari_yuzdesi_ops)`: Belirtilen ekipman slotundaki eşyayı geliştirir.
        *   `pc.unequip_select(slot_index)`: Belirtilen ekipman slotundaki eşyayı çıkarır, VNUM'unu döndürür.
        *   `pc.equip_slot(envanter_hücresi)`: Belirtilen envanter hücresindeki eşyayı giyer.
        *   `pc.get_socket_items()`: Envanterdeki, içinde Metin Taşı olan eşyaların (İsim, Hücre No) listesini döndürür.
        *   `pc.get_sig_items(grup_vnum)`: Belirtilen özel eşya grubuna (SIG) ait envanterdeki eşyaların ID'lerini döndürür.
    *   **Görev ve Bayrak Yönetimi:**
        *   `pc.get_quest_flag(bayrak_adi_sonek)`: Mevcut görevin `quest_adi.bayrak_adi_sonek` bayrağının değerini alır.
        *   `pc.set_quest_flag(bayrak_adi_sonek, deger)`: Mevcut görevin `quest_adi.bayrak_adi_sonek` bayrağının değerini ayarlar.
        *   `pc.del_quest_flag(bayrak_adi_sonek)`: Mevcut görevin belirtilen bayrağını siler.
        *   `pc.get_flag(bayrak_adi)`: Global bir görev bayrağının değerini alır.
        *   `pc.set_flag(bayrak_adi, deger)`: Global bir görev bayrağının değerini ayarlar.
        *   `pc.get_another_quest_flag(baska_gorev_adi, bayrak_adi)`: Başka bir göreve ait bayrağın değerini alır.
        *   `pc.set_another_quest_flag(baska_gorev_adi, bayrak_adi, deger)`: Başka bir göreve ait bayrağın değerini ayarlar.
    *   **Tecrübe, Para ve Statü Değişimi:**
        *   `pc.give_exp(sebep_string, miktar)`: Tecrübe puanı verir (`PC::GiveExp`).
        *   `pc.give_exp2(miktar)`: Tecrübe puanı verir (`CHARACTER::PointChange(POINT_EXP, ...)`).
        *   `pc.give_exp_perc(sebep_string, seviye_icin, yuzde)`: Belirli bir seviyenin tecrübe ihtiyacının yüzdesi kadar tecrübe verir.
        *   `pc.give_gold(miktar)`: Yang verir.
        *   `pc.change_gold(miktar)`, `pc.changemoney(miktar)`, `pc.change_money(miktar)`: Yang miktarını değiştirir (pozitif/negatif).
        *   `pc.give_cheque(miktar)`: (`__CHEQUE_SYSTEM__` tanımlıysa) Won verir.
        *   `pc.change_cheque(miktar)`: (`__CHEQUE_SYSTEM__` tanımlıysa) Won miktarını değiştirir.
        *   `pc.change_alignment(yeni_deger_carpi_10)`: Sıralama puanını değiştirir.
        *   `pc.change_sp(miktar)`: SP miktarını değiştirir.
        *   `pc.set_level(yeni_seviye)`: Karakterin seviyesini, statülerini ve HP/SP'sini doğrudan ayarlar.
        *   `pc.set_ht(yeni_deger)`, `pc.set_iq(yeni_deger)`, `pc.set_st(yeni_deger)`, `pc.set_dx(yeni_deger)`: Temel statü değerlerini doğrudan ayarlar, harcanan statü puanını düşer.
        *   `pc.reset_status(stat_index)`: Belirli bir statüyü (0:VIT, 1:INT, 2:STR, 3:DEX) sıfırlar, puanları iade eder.
        *   `pc.reset_point()`: Karakterin seviyesine göre statü ve yetenek puanlarını sıfırlar.
    *   **Yetenek Yönetimi:**
        *   `pc.get_skill_group()`: Mevcut yetenek grubunu döndürür.
        *   `pc.set_skill_group(grup_no)`: Yetenek grubunu ayarlar.
        *   `pc.has_master_skill()`: En az bir usta seviye (M1+) yeteneği olup olmadığını kontrol eder.
        *   `pc.learn_grand_master_skill(skill_vnum)`: Belirtilen yeteneği Büyük Usta (G1) yapar.
        *   `pc.is_skill_book_no_delay()`: Beceri kitabı okuma bekleme süresinin aktif olup olmadığını kontrol eder.
        *   `pc.remove_skill_book_no_delay()`: Beceri kitabı okuma bekleme süresi efektini kaldırır.
        *   `pc.get_skill_level(skill_vnum)`: Belirtilen yeteneğin seviyesini döndürür.
        *   `pc.set_skill_level(skill_vnum, seviye)`: Belirtilen yeteneğin seviyesini ayarlar.
        *   `pc.clear_skill()`: Tüm yetenekleri sıfırlar.
        *   `pc.clear_sub_skill()`: Tüm alt yetenekleri (pasifler vb.) sıfırlar.
        *   `pc.clear_one_skill(skill_vnum)`: Belirtilen bir yeteneği sıfırlar.
        *   `pc.dec_skill_point()`: Yetenek puanını bir azaltır.
        *   `pc.set_skill_point(yeni_puan_miktari)`: Yetenek puanını doğrudan ayarlar.
    *   **Işınlama ve Konum:**
        *   `pc.warp(x, y, map_index_ops)`: Karakteri belirtilen koordinatlara (ve haritaya) ışınlar.
        *   `pc.warp_local(map_index, local_x, local_y)`: Karakteri belirtilen haritadaki yerel koordinatlara ışınlar.
        *   `pc.warp_exit()`: Karakteri daha önce kaydedilmiş çıkış konumuna ışınlar.
        *   `pc.set_warp_location(map_index, x, y)`: Karakterin bir sonraki ışınlanma/ölüm sonrası canlanma noktasını ayarlar.
        *   `pc.set_warp_location_local(map_index, local_x, local_y)`: `pc.set_warp_location` gibi ama yerel koordinatlarla.
        *   `pc.save_exit_location()`: Mevcut konumunu çıkış konumu olarak kaydeder.
        *   `pc.teleport(hedef_kasaba_index_veya_oyuncu_adi)`: Özel ışınlanma fonksiyonu (kasaba veya oyuncu yanına).
    *   **Lonca ve Savaş Haritası:**
        *   `pc.destroy_guild()`: Karakterin loncasını dağıtmak için istek gönderir (sadece liderse).
        *   `pc.remove_from_guild()`: Karakteri loncadan çıkarır.
        *   `pc.warp_to_guild_war_observer_position(lonca1_id, lonca2_id)`: Karakteri iki lonca arasındaki savaş haritasının gözlemci konumuna ışınlar.
        *   `pc.get_war_map()`: Karakterin bulunduğu savaş haritasının indeksini döndürür.
    *   **Binek ve At:**
        *   `pc.is_mount()`: Karakterin bir binek (mount) üzerinde olup olmadığını kontrol eder.
        *   `pc.mount(binek_vnum, sure_ops)`: Karakteri belirtilen bineğe bindirir.
        *   `pc.mount_bonus(uygulama_yeri, deger, sure)`: Bineğe ek bonus verir.
        *   `pc.unmount()`: Karakteri binekten indirir.
        *   `pc.get_horse_level()`: At seviyesini döndürür.
        *   `pc.is_horse_alive()`: Atın canlı olup olmadığını kontrol eder.
        *   `pc.revive_horse()`: Atı canlandırır.
        *   `pc.get_special_ride_vnum()`: Takılı özel binek eşyasının VNUM'unu ve soketini döndürür.
    *   **Dönüşüm (Polymorph):**
        *   `pc.is_polymorphed()`: Karakterin dönüşmüş olup olmadığını kontrol eder.
        *   `pc.remove_polymorph()`: Dönüşümü kaldırır.
        *   `pc.polymorph(canavar_vnum, sure)`: Karakteri belirtilen canavara dönüştürür.
        *   `pc.give_polymorph_book(item_vnum, skill_vnum, book_type, upgrade_type)`: Dönüşüm kitabı verir.
        *   `pc.upgrade_polymorph_book()`: Mevcut görev eşyası olan dönüşüm kitabını geliştirir.
    *   **Evlilik Sistemi:**
        *   `pc.is_engaged()`: Nişanlı olup olmadığını kontrol eder.
        *   `pc.is_married()`: Evli olup olmadığını kontrol eder.
        *   `pc.is_engaged_or_married()`: Nişanlı veya evli olup olmadığını kontrol eder.
    *   **Madencilik ve Cevher İşleme:**
        *   `pc.mining()`: Madencilik yapar (mevcut NPC ile etkileşimde).
        *   `pc.ore_refine(maliyet, basari_yuzdesi, metin_tasi_hücresi)`: Cevher işler (mevcut görev eşyası ve metin taşı ile).
        *   `pc.diamond_refine(maliyet, basari_yuzdesi)`: Elmas işler (mevcut görev eşyası ile).
    *   **Premium ve Özel Sistemler:**
        *   `pc.get_premium_remain_sec(premium_tipi)`: Belirtilen premium özelliğin kalan süresini döndürür.
        *   `pc.open_acce()`: (`__ACCE_COSTUME_SYSTEM__` tanımlıysa) Şebnem/Kanat sistem penceresini açar.
        *   `pc.get_gem()`: (`__GEM_SYSTEM__` tanımlıysa) Gaya miktarını döndürür.
        *   `pc.open_gem_shop()`: (`__GEM_SYSTEM__` tanımlıysa) Gaya Marketini açar.
        *   `pc.create_gaya(maliyet, basari_yuzdesi, metin_tasi_hücresi, parilti_tasi_sayisi)`: (`__GEM_SYSTEM__` tanımlıysa) Gaya üretir.
        *   `pc.conqueror_reset_status(stat_index)`: (`__CONQUEROR_LEVEL__` tanımlıysa) Fatih seviyesi statüsünü sıfırlar.
        *   `pc.get_conqueror_exp()`, `pc.get_conqueror_next_exp()`, `pc.get_conqueror_level()`: (`__CONQUEROR_LEVEL__` tanımlıysa) Fatih seviyesi tecrübe/seviye bilgilerini döndürür.
        *   `pc.set_conqueror_level()`: (`__CONQUEROR_LEVEL__` tanımlıysa) Fatih seviyesini ayarlar (muhtemelen seviye atlatır).
    *   **Çeşitli Fonksiyonlar:**
        *   `pc.change_sex()`: Karakterin cinsiyetini değiştirir.
        *   `pc.change_empire(yeni_imparatorluk_id)`: Karakterin imparatorluğunu değiştirir, başarı durumunu döndürür.
        *   `pc.get_change_empire_count()`: İmparatorluk değiştirme sayısını alır.
        *   `pc.set_change_empire_count()`: İmparatorluk değiştirme sayısını artırır (veya ayarlar).
        *   `pc.change_name(yeni_isim)`: Karakterin adını değiştirir, sonuç kodu döndürür.
        *   `pc.send_block_mode(mod_degeri)`: Karakterin blok modunu (saldırı engeli vb.) ayarlar.
        *   `pc.aggregate_monster()`: Çevredeki canavarları karaktere çeker.
        *   `pc.forget_my_attacker()`: Karakterin saldıranlarını unutturur.
        *   `pc.attract_ranger()`: Uzaktaki canavarları çeker.
        *   `pc.select_pid(hedef_pid)`: Görev bağlamını `hedef_pid`'ye sahip karaktere değiştirir, eski karakterin PID'sini döndürür.
        *   `pc.select_vid(hedef_vid)`: Görev bağlamını `hedef_vid`'ye sahip karaktere değiştirir, eski karakterin VID'sini döndürür.
        *   `pc.is_near_vid(hedef_vid, mesafe)`: Belirtilen VID'ye sahip karakterin belirli bir mesafe içinde olup olmadığını kontrol eder.
        *   `pc.give_lotto()`: Loto bileti verir (veritabanına kayıt ekler).
        *   `pc.get_logoff_interval()`: Son çıkıştan bu yana geçen süreyi döndürür.
        *   `pc.get_last_play()`: Son oynama zaman damgasını döndürür.
        *   `pc.get_player_id()`: Karakterin PID'sini döndürür.
        *   `pc.get_account_id()`: Karakterin hesap ID'sini döndürür.
        *   `pc.get_account()`: Karakterin hesap adını döndürür.
        *   `pc.charge_cash(miktar, odeme_tipi_string)`: Nakit yükleme isteği gönderir.
        *   `pc.give_award(item_vnum, sayi, sebep_string)`: Nesne market deposuna eşya ekler.
        *   `pc.give_award_socket(item_vnum, sayi, sebep_string, socket0, socket1, socket2)`: `pc.give_award` gibi ama soketli.
        *   `pc.get_informer_type()`: Item award için komut stringini alır (özel sistem).
        *   `pc.get_informer_item()`: Item award için item vnum'unu alır (özel sistem).
        *   `pc.get_killee_drop_pct()`: Öldürülen NPC'den eşya düşme şansı ile ilgili yüzdeleri döndürür.
        *   `pc.is_blocked(hedef_oyuncu_adi)`: (`__MESSENGER_BLOCK_SYSTEM__` tanımlıysa) Hedef oyuncunun engellenip engellenmediğini kontrol eder.
        *   `pc.is_friend(hedef_oyuncu_adi)`: (`__MESSENGER_BLOCK_SYSTEM__` tanımlıysa) Hedef oyuncunun arkadaş listesinde olup olmadığını kontrol eder.
        *   `pc.hide_costume(kostum_parcasi_index, gizle_mi)`: (`__HIDE_COSTUME_SYSTEM__` tanımlıysa) Belirtilen kostüm parçasını gizler/gösterir.
        *   `pc.change_race(yeni_irk_id)`: Karakterin ırkını değiştirir, yetenekleri sıfırlar.
        *   `pc.dead()`: Karakteri öldürür.
        *   `pc.give_full_set()`: Karaktere mesleğine uygun başlangıç ekipman seti verir ve efsunlar.
*   **Kayıt:** `RegisterPCFunctionTable()` fonksiyonu, `pc_functions` dizisindeki tüm fonksiyonları `pc` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** Çok sayıda başlık dosyası içerir (`stdafx.h`, `config.h`, `questmanager.h`, `char.h`, `item.h`, `affect.h`, `guild_manager.h`, `marriage.h`, `polymorph.h`, `log.h`, `utils.h` vb.).

## Çekirdek Görev Sistemi Sınıfları (.h)

*Buraya görev sisteminin C++ tarafındaki header dosyaları (`questmanager.h`, `questnpc.h`, `questpc.h` vb.) belgelenecektir.*

### `quest.h`

*   **Amaç:** Metin2 görev (quest) sistemi için temel tanımlamaları ve yapıları içerir. Bu dosya, görevlerin Lua betikleriyle nasıl etkileşime girdiğini ve durumlarını nasıl yönettiklerini belirleyen temel enum'ları ve struct'ları sağlar.
*   **Temel İşlevler/İçerik:**
    *   **`INDUCTION_LEVEL*` Makroları:** Muhtemelen belirli seviyelerde tetiklenen özel görevler veya sistemler için kullanılan bayraklar.
    *   **`quest` Namespace'i:**
        *   **NPC VNUM Sabitleri (`QUEST_NO_NPC`, `QUEST_ATTR_NPC_START`, `QUEST_ATTR*_NPC`):** Görevlerde kullanılabilen özel NPC ID'lerini tanımlar. `QUEST_NO_NPC` (0) genellikle bir NPC'nin gerekmediği durumlar için, diğerleri ise görevlerde özel amaçlar için sanal NPC'ler olarak kullanılabilir (örneğin, görünmez tetikleyiciler).
        *   **Görev Olay Türleri Enum'u (`QUEST_CLICK_EVENT`'den `QUEST_EVENT_COUNT`'a kadar):** Görevlerin hangi olaylarla tetiklenebileceğini tanımlar. Örneğin:
            *   `QUEST_CLICK_EVENT`: NPC'ye tıklama.
            *   `QUEST_KILL_EVENT`: Belirli bir canavarı öldürme.
            *   `QUEST_TIMER_EVENT`: Belirli bir süre geçmesi (oyuncuya özel).
            *   `QUEST_LEVELUP_EVENT`: Seviye atlama.
            *   `QUEST_LOGIN_EVENT`/`QUEST_LOGOUT_EVENT`: Oyuna giriş/çıkış.
            *   `QUEST_ITEM_USE_EVENT`: Eşya kullanma.
            *   `QUEST_SERVER_TIMER_EVENT`: Sunucu genelinde periyodik olay.
            *   Ve daha birçok olay türü...
        *   **Görev Askıya Alma Durumları Enum'u (`SUSPEND_STATE_NONE`'dan `SUSPEND_STATE_SELECT_ITEM_EX`'e kadar):** Bir görevin oyuncudan bir girdi beklerken (örneğin, bir seçenek seçimi, bir metin girişi, bir eşya seçimi) hangi durumda olduğunu belirtir. Bu durumlar, görev akışını duraklatır ve oyuncunun etkileşimini bekler.
            *   `SUSPEND_STATE_PAUSE`: Basit bir bekleme veya mesaj gösterme.
            *   `SUSPEND_STATE_SELECT`: Oyuncunun birden fazla seçenekten birini seçmesini bekleme.
            *   `SUSPEND_STATE_INPUT`: Oyuncudan metin girişi bekleme.
            *   `SUSPEND_STATE_CONFIRM`: Evet/Hayır onayı bekleme.
            *   `SUSPEND_STATE_SELECT_ITEM`: Oyuncunun envanterinden bir eşya seçmesini bekleme.
            *   `SUSPEND_STATE_SELECT_ITEM_EX`: (`__GEM_SYSTEM__` tanımlıysa) Muhtemelen eşya seçimi için genişletilmiş bir durum.
        *   **`EQuestConfirmType` Enum'u (`CONFIRM_NO`, `CONFIRM_YES`, `CONFIRM_TIMEOUT`):** `SUSPEND_STATE_CONFIRM` durumunda oyuncunun verdiği yanıt türünü belirtir.
        *   **`AStateScriptType` Struct:** Bir görev durumuna (state) ait derlenmiş Lua betik kodunu (`m_code` - `std::vector<char>`) tutar.
        *   **`AArgScript` Struct:** Bir görevin "when" bloğunu temsil eder.
            *   `arg` (std::string): `when` koşulunun argümanı (örn: `when kill.VNUM with ...` için VNUM).
            *   `when_condition` (std::vector<char>): `with` ile belirtilen ek Lua koşulunun derlenmiş kodu.
            *   `script` (AStateScriptType): `begin ... end` arasındaki ana Lua betik kodunun derlenmiş hali.
            *   `quest_index`, `state_index`: Bu betiğin hangi göreve ve hangi state'e ait olduğunu belirtir.
        *   **`QuestState` Struct:** Bir oyuncunun belirli bir görevdeki mevcut durumunu temsil eder.
            *   `co` (lua_State*): Bu görev durumu için kullanılan Lua coroutine'i.
            *   `ico` (int): Coroutine referansı (Lua registry'de).
            *   `args` (short int): Coroutine'e geçirilecek argüman sayısı.
            *   `suspend_state` (BYTE): Görevin mevcut askıya alma durumu (`SUSPEND_STATE_*`).
            *   `iIndex` (int): Görevin genel index'i.
            *   `bStart` (bool): Görev bu state'den mi başladı?
            *   `st` (int): Mevcut state'in script index'i.
            *   `_title`, `_clock_name`, `_counter_name`, `_clock_value`, `_counter_value`, `_icon_file`: Görev arayüzünde (UI) gösterilecek bilgiler (başlık, zamanlayıcı, sayaç, ikon).
            *   `chat_scripts` (std::vector<AArgScript*>): Bu state içinde tanımlanmış olan NPC konuşma (`chat`) betiklerini tutar.
*   **Bağlantılı Dosyalar:** `lua_incl.h` (Lua başlıkları için). Genellikle `questmanager.cpp`, `questnpc.cpp`, `questpc.cpp` gibi görev sistemi çekirdek dosyaları tarafından kullanılır.

### `questevent.h`

*   **Amaç:** Görev sistemi için zamanlayıcı tabanlı olayları tanımlayan yapıları ve bu olayları oluşturan/iptal eden fonksiyon bildirimlerini içerir. Bu olaylar, belirli bir süre sonunda veya periyodik olarak görev betiklerini tetiklemek için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`quest` Namespace'i:**
        *   **`EVENTINFO(quest_server_event_info)` Struct:** Sunucu genelinde çalışan görev zamanlayıcıları için bilgi tutar.
            *   `time_cycle` (int): Zamanlayıcının ne kadar sürede bir tetikleneceği (eğer döngüselse) veya ilk tetiklenme süresi. 0 ise döngüsel değildir.
            *   `npc_id` (unsigned int): Bu zamanlayıcıyla ilişkili NPC'nin VNUM'u. `QUEST_NO_NPC` olabilir.
            *   `arg` (unsigned int): Zamanlayıcı olayına özel bir argüman.
            *   `name` (char*): Zamanlayıcının adı (tanımlama ve temizleme için).
        *   **`EVENTINFO(quest_event_info)` Struct:** Oyuncuya özel görev zamanlayıcıları için bilgi tutar.
            *   `time_cycle` (int): `quest_server_event_info`'daki gibi.
            *   `player_id` (unsigned int): Bu zamanlayıcının ait olduğu oyuncunun ID'si.
            *   `npc_id` (unsigned int): Bu zamanlayıcıyla ilişkili NPC'nin VNUM'u.
            *   `name` (char*): Zamanlayıcının adı.
        *   **Fonksiyon Bildirimleri:**
            *   `quest_create_server_timer_event(...)`: Yeni bir sunucu geneli görev zamanlayıcısı oluşturur. Zamanlayıcı adı, ne zaman (`when` saniye cinsinden), ilişkili NPC, döngüsel olup olmadığı (`loop`) ve bir argüman alır. `LPEVENT` (event işaretçisi) döndürür.
            *   `quest_create_timer_event(...)`: Yeni bir oyuncuya özel görev zamanlayıcısı oluşturur. Zamanlayıcı adı, oyuncu ID'si, ne zaman, ilişkili NPC ve döngüsel olup olmadığını alır. `LPEVENT` döndürür.
            *   `CancelTimerEvent(LPEVENT* ppEvent)`: Verilen event işaretçisindeki zamanlayıcıyı iptal eder ve ilişkili belleği (özellikle `name`) temizler.
*   **Bağlantılı Dosyalar:** `questevent.cpp` (uygulama), `quest.h` (`QUEST_NO_NPC` sabiti için). `event.h` (veya benzeri bir genel olay sistemi başlığı) `EVENTINFO` makrosu ve `LPEVENT` türü için gereklidir.

## Çekirdek Görev Sistemi Sınıfları (.cpp)

*Buraya görev sisteminin C++ tarafındaki kaynak kod dosyaları (`questmanager.cpp`, `questnpc.cpp`, `questpc.cpp` vb.) belgelenecektir.*

### `questevent.cpp`

*   **Amaç:** `questevent.h` dosyasında bildirilen görev zamanlayıcı olaylarının oluşturulmasını ve çalıştırılmasını uygular. Sunucu genelindeki ve oyuncuya özel zamanlayıcıların mantığını içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`quest` Namespace'i:**
        *   **`CancelTimerEvent(LPEVENT* ppEvent)`:**
            *   Verilen olay işaretçisinden (`*ppEvent`) `quest_event_info` (veya türemiş bir yapı) bilgisini alır.
            *   Eğer `info` ve `info->name` geçerliyse, `info->name` için ayrılmış belleği `M2_DELETE_ARRAY` ile serbest bırakır.
            *   Genel `event_cancel(ppEvent)` fonksiyonunu çağırarak olayı sistemden kaldırır.
        *   **`EVENTFUNC(quest_server_timer_event)` (Olay İşleyici Fonksiyonu):**
            *   Sunucu geneli bir görev zamanlayıcısı tetiklendiğinde çalışır.
            *   `event->info`'dan `quest_server_event_info`'yu alır.
            *   `CQuestManager::instance().ServerTimer(info->npc_id, info->arg)` fonksiyonunu çağırır. Bu fonksiyon, ilgili görevin sunucu zamanlayıcı olayını işler.
            *   `ServerTimer` `false` dönerse (muhtemelen bir hata veya görevin artık geçerli olmaması durumu), olayın kısa bir süre sonra tekrar denenmesi için `passes_per_sec / 2 + 1` döndürür.
            *   Eğer zamanlayıcı döngüsel değilse (`info->time_cycle == 0`), `CQuestManager::instance().ClearServerTimerNotCancel(info->name, info->arg)` ile görev yöneticisindeki kaydını temizler ve `info->name` belleğini serbest bırakır.
            *   Döngüselse veya devam edecekse, bir sonraki tetiklenme için `info->time_cycle` değerini döndürür.
        *   **`EVENTFUNC(quest_timer_event)` (Olay İşleyici Fonksiyonu):**
            *   Oyuncuya özel bir görev zamanlayıcısı tetiklendiğinde çalışır.
            *   `event->info`'dan `quest_event_info`'yu alır.
            *   `CHARACTER_MANAGER::instance().FindByPID(info->player_id)` ile oyuncunun hala oyunda olup olmadığını kontrol eder.
                *   Oyuncu oyundaysa:
                    *   `CQuestManager::instance().Timer(info->player_id, info->npc_id)` fonksiyonunu çağırır. Bu, ilgili oyuncu için görevin zamanlayıcı olayını işler.
                    *   `Timer` `false` dönerse, olayın kısa bir süre sonra tekrar denenmesi için `passes_per_sec / 2 + 1` döndürür.
                    *   Eğer zamanlayıcı döngüsel değilse (`info->time_cycle == 0`), `END_OF_TIMER_EVENT` etiketine atlar.
                *   Oyuncu oyunda değilse (veya döngüsel olmayan zamanlayıcı bittiyse):
                    *   `END_OF_TIMER_EVENT` etiketi:
                        *   `CQuestManager::instance().GetPC(info->player_id)` ile oyuncunun görev verilerini (`PC` nesnesi) alır.
                        *   Eğer `pPC` ve `info->name` geçerliyse, `pPC->RemoveTimerNotCancel(info->name)` ile oyuncunun görev verilerinden bu zamanlayıcıyı kaldırır.
                        *   `info->name` belleğini serbest bırakır ve olay tamamen sonlandığı için `0` döndürür.
            *   Döngüselse ve oyuncu hala oyundaysa, bir sonraki tetiklenme için `info->time_cycle` değerini döndürür.
        *   **`quest_create_server_timer_event(...)`:**
            *   Yeni bir sunucu geneli görev zamanlayıcısı oluşturur.
            *   Verilen `name` için bellek ayırır (`M2_NEW char[]`) ve kopyalar.
            *   `when` (saniye) değerini `PASSES_PER_SEC` makrosuyla oyun döngüsü sayısına çevirir (`ltime_cycle`).
            *   `AllocEventInfo<quest_server_event_info>()` ile olay bilgi yapısını oluşturur, üyelerini (`npc_id`, `time_cycle` (döngüselse `ltime_cycle`, değilse 0), `arg`, `name`) doldurur.
            *   `event_create(quest_server_timer_event, info, ltime_cycle)` ile genel olay sistemine kaydeder ve dönen `LPEVENT`'i döndürür.
        *   **`quest_create_timer_event(...)`:**
            *   Yeni bir oyuncuya özel görev zamanlayıcısı oluşturur.
            *   Sunucu zamanlayıcısı oluşturmaya benzer şekilde `name` için bellek ayırır, `when`'i döngü sayısına çevirir.
            *   `AllocEventInfo<quest_event_info>()` ile olay bilgi yapısını oluşturur, üyelerini (`player_id`, `npc_id`, `name`, `time_cycle`) doldurur.
            *   `event_create(quest_timer_event, info, ltime_cycle)` ile olayı kaydeder ve `LPEVENT`'i döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `questmanager.h`, `questevent.h`. Ayrıca `event.h` (veya benzeri genel olay sistemi) ve `passes_per_sec` gibi zamanlama sabitleri için `utils.h` gibi dosyalara bağımlıdır. 