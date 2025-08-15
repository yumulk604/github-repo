# Metin2 Oyun Sunucusu - Görev Sistemi Referansı (`game/src`) - Bölüm 2

Bu belge, Metin2 oyun sunucusunun (`game/src`) görev (quest) sistemiyle ilgili C++ ve Lua bileşenlerini belgeler.

## İçindekiler

*   [Lua Arayüzleri (`questlua_*`)](#lua-arayüzleri-questlua_)
    *   [`questlua_pet.cpp`](#questlua_petcpp)
    *   [`questlua_quest.cpp`](#questlua_questcpp)
    *   [`questlua_shipdefense_mgr.cpp`](#questlua_shipdefense_mgrcpp)
    *   [`questlua_speedserver.cpp`](#questlua_speedservercpp)
    *   [`questlua_target.cpp`](#questlua_targetcpp)
    *   [`questlua_templeochao.cpp`](#questlua_templeochaocpp)
*   [Çekirdek Lua Entegrasyonu (`questlua.cpp`)](#çekirdek-lua-entegrasyonu-questluacpp)
    *   [`questlua.cpp`](#questluacpp)
*   [Çekirdek Görev Sistemi Bileşenleri](#çekirdek-görev-sistemi-bileşenleri)
    *   [`questlua.h`](#questluah)
    *   [`questmanager.h`](#questmanagerh)
    *   [`questmanager.cpp`](#questmanagercpp)
    *   [`questnpc.h`](#questnpch)
    *   [`questnpc.cpp`](#questnpccpp)
    *   [`questpc.h`](#questpch)
    *   [`questpc.cpp`](#questpccpp)

### `questlua_pet.cpp`

*   **Amaç:** (`__PET_SYSTEM__` makrosu tanımlıysa) Evcil Hayvan (Pet) sistemiyle ilgili işlevleri Lua betiklerine açar. Evcil hayvan çağırma, geri gönderme, çağrılmış hayvan sayısını alma, belirli bir hayvanın çağrılı olup olmadığını kontrol etme ve evcil hayvan üzerinde efekt gösterme gibi fonksiyonları içerir.
*   **Lua Ön Eki:** `pet`
*   **Koşullu Derleme:** Bu dosyadaki fonksiyonlar yalnızca `__PET_SYSTEM__` makrosu tanımlı olduğunda kullanılabilir.
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `pet.summon(mob_vnum, pet_ismi, uzaktan_gelsin_mi)`: Belirtilen VNUM'a sahip bir evcil hayvanı, verilen isimle çağırır. `uzaktan_gelsin_mi` (bool, opsiyonel, varsayılan `false`) parametresi hayvanın uzaktan gelip gelmeyeceğini belirler. İşlem mevcut görev eşyası (`GetCurrentItem()`) üzerinden yürütülür. Başarılı olursa çağrılan hayvanın VID'sini, olmazsa 0 döndürür.
    *   `pet.unsummon(mob_vnum)`: Belirtilen VNUM'a sahip çağrılmış evcil hayvanı geri gönderir.
    *   `pet.count_summoned()`: Karakterin o anda çağrılmış olan evcil hayvanlarının sayısını döndürür.
    *   `pet.is_summon(mob_vnum)`: Belirtilen VNUM'a sahip evcil hayvanın çağrılmış olup olmadığını kontrol eder (`true`/`false`).
    *   `pet.spawn_effect(mob_vnum, efekt_dosya_yolu)`: Belirtilen VNUM'a sahip çağrılmış evcil hayvan üzerinde belirtilen parçacık efektini gösterir.
*   **Kayıt:** `RegisterPetFunctionTable()` fonksiyonu, `pet_functions` dizisindeki tüm fonksiyonları `pet` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `horsename_manager.h` (dahil edilmiş ancak doğrudan kullanılmıyor gibi), `char.h`, `affect.h` (dahil edilmiş ancak doğrudan kullanılmıyor gibi), `config.h` (dahil edilmiş ancak doğrudan kullanılmıyor gibi), `utils.h` (dahil edilmiş ancak doğrudan kullanılmıyor gibi), `PetSystem.h`.

### `questlua_quest.cpp`

*   **Amaç:** Görevlerin (quest) kendi durumlarını yönetmek ve kontrol etmek için temel işlevleri Lua betiklerine açar. Görev durumunu (state) değiştirme, görev arayüzündeki bilgileri (başlık, zamanlayıcı, sayaç, ikon) ayarlama, görevi başlatma/bitirme, coroutine'i yield etme ve diğer oyuncuların görev bağlamında işlem yapma gibi kritik fonksiyonları içerir.
*   **Lua Ön Eki:** `q`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Görev Durumu ve Akışı:**
        *   `q.setstate(yeni_durum_adi)` veya `q.set_state(yeni_durum_adi)`: Mevcut görevin durumunu (state) belirtilen isimdeki duruma değiştirir. Görev yöneticisinden durumun indeksini alır ve oyuncunun mevcut görev durumunu günceller.
        *   `q.yield()`: Mevcut görevin Lua coroutine'ini askıya alır (yield). Bu, oyuncudan bir girdi beklerken veya belirli bir olayın gerçekleşmesini beklerken kullanılır. `other_pc_block` içindeyken yield yapmayı engeller.
        *   `q.start()`: Mevcut görevin başlangıç bayrağını ayarlar.
        *   `q.done()`: Mevcut görevin bitiş bayrağını ayarlar.
        *   `q.no_send()`: Muhtemelen görev durumu değişikliği veya arayüz güncellemelerinin istemciye hemen gönderilmesini engeller (`CQuestManager::SetNoSend`).
    *   **Görev Arayüzü Yönetimi:**
        *   `q.set_title(baslik_metni)`: Mevcut görevin başlığını ayarlar.
        *   `q.set_title2(gorev_adi, baslik_metni)`: Belirtilen başka bir görevin başlığını ayarlar.
        *   `q.set_clock_name(saat_adi)`: Görev arayüzündeki zamanlayıcı (clock) için gösterilecek adı ayarlar.
        *   `q.set_clock_value(saniye_cinsinden_deger)`: Görev arayüzündeki zamanlayıcının değerini ayarlar.
        *   `q.set_counter_name(sayac_adi)`: Görev arayüzündeki sayaç (counter) için gösterilecek adı ayarlar.
        *   `q.set_counter_value(sayac_degeri)`: Görev arayüzündeki sayacın değerini ayarlar.
        *   `q.set_icon(ikon_dosya_yolu)`: Görev arayüzünde gösterilecek ikon dosyasının yolunu ayarlar.
    *   **Görev Bilgisi:**
        *   `q.getcurrentquestindex()`: Mevcut çalışan görevin indeksini (numarasını) döndürür.
    *   **Diğer Oyuncu Bağlamı:**
        *   `q.begin_other_pc_block(hedef_oyuncu_pid)`: Belirtilen PID'ye sahip oyuncunun görev bağlamında işlem yapmaya başlar. Bu, bir oyuncunun başka bir oyuncu adına görev fonksiyonlarını çalıştırmasına olanak tanır (dikkatli kullanılmalıdır).
        *   `q.end_other_pc_block()`: Başka bir oyuncunun görev bağlamında işlem yapmayı bitirir ve orijinal oyuncunun bağlamına döner.
*   **Kayıt:** `RegisterQuestFunctionTable()` fonksiyonu, `quest_functions` dizisindeki tüm fonksiyonları `q` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`.

### `questlua_shipdefense_mgr.cpp`

*   **Amaç:** (`__SHIP_DEFENSE__` makrosu tanımlıysa) Gemi Savunması (Hydra) etkinliğini yönetmek için Lua fonksiyonları sağlar. Etkinlik örneği oluşturma, başlatma, katılma, ayrılma, bitirme, durum kontrolü, yapılandırma ayarlarını sorgulama ve ittifak HP'sini ayarlama gibi işlevleri içerir.
*   **Lua Ön Eki:** `ship_defense_mgr`
*   **Koşullu Derleme:** Bu dosyadaki fonksiyonlar yalnızca `__SHIP_DEFENSE__` makrosu tanımlı olduğunda kullanılabilir.
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `ship_defense_mgr.create()`: Mevcut karakter için yeni bir Gemi Savunması etkinliği örneği oluşturur (`CShipDefenseManager::Create`).
    *   `ship_defense_mgr.start()`: Mevcut karakter için oluşturulmuş Gemi Savunması etkinliğini başlatır (`CShipDefenseManager::Start`). Etkinlik zaten çalışıyorsa veya oluşturulmamışsa hata verir.
    *   `ship_defense_mgr.join()`: Mevcut karakteri aktif bir Gemi Savunması etkinliğine katar (`CShipDefenseManager::Join`).
    *   `ship_defense_mgr.leave()`: Mevcut karakteri bulunduğu Gemi Savunması etkinliğinden çıkarır (`CShipDefenseManager::Leave`).
    *   `ship_defense_mgr.land()`: Mevcut karakteri Gemi Savunması etkinliğinden karaya (muhtemelen başlangıç noktasına) ışınlar (`CShipDefenseManager::Land`).
    *   `ship_defense_mgr.is_created()`: Mevcut karakter için bir Gemi Savunması örneğinin oluşturulup oluşturulmadığını kontrol eder (`true`/`false`).
    *   `ship_defense_mgr.is_running()`: Mevcut karakterin Gemi Savunması etkinliğinin çalışıp çalışmadığını kontrol eder (`true`/`false`).
    *   `ship_defense_mgr.need_party()`: Etkinliğe katılmak için parti gerekip gerekmediğini (yapılandırma sabiti `ShipDefense::NEED_PARTY`) döndürür.
    *   `ship_defense_mgr.need_ticket()`: Etkinliğe katılmak için bilet gerekip gerekmediğini (yapılandırma sabiti `ShipDefense::NEED_TICKET`) döndürür.
    *   `ship_defense_mgr.spawn_wood_repair()`: Tamir için odunların spawn olup olmayacağını (yapılandırma sabiti `ShipDefense::SPAWN_WOOD_REPAIR`) döndürür.
    *   `ship_defense_mgr.require_cooldown()`: Etkinliğe tekrar katılmak için bekleme süresi gerekip gerekmediğini (yapılandırma sabiti `ShipDefense::REQUIRE_COOLDOWN`) döndürür.
    *   `ship_defense_mgr.set_alliance_hp_pct(yuzde)`: Gemi Savunmasındaki ittifakın (muhtemelen Hydra'nın) can puanı yüzdesini ayarlar (`CShipDefenseManager::SetAllianceHPPct`).
    *   `ship_defense_mgr.can_join()`: Mevcut karakterin etkinliğe katılma koşullarını sağlayıp sağlamadığını kontrol eder (`CShipDefenseManager::CanJoin`).
*   **Kayıt:** `RegisterShipDefenseManagerFunctionTable()` fonksiyonu, `functions` dizisindeki tüm fonksiyonları `ship_defense_mgr` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questlua.h`, `questmanager.h`, `ShipDefense.h`.

### `questlua_speedserver.cpp`

*   **Amaç:** Hız Sunucusu (Speed Server) etkinliği kapsamında, imparatorluklara özel olarak haftanın günleri veya belirli tarihler (tatiller) için EXP bonus oranlarını ve geçerlilik sürelerini ayarlamak ve sorgulamak için Lua fonksiyonları sağlar.
*   **Lua Ön Eki:** `speedserver`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   **Haftalık EXP Ayarları:**
        *   `speedserver.get_wday(imparatorluk, gun_index)`: Belirtilen imparatorluk (1-3) ve gün indeksi (1-7, Pazar=1) için tanımlanmış EXP bonus zaman dilimlerini döndürür. Her dilim için bitiş saati, bitiş dakikası ve EXP yüzdesini (üçlü gruplar halinde) döndürür.
        *   `speedserver.set_wday(imparatorluk, gun_index, bitis_saati, bitis_dakikasi, exp_yuzdesi)`: Belirtilen imparatorluk ve gün için yeni bir EXP bonus zaman dilimi ekler/ayarlar.
        *   `speedserver.init_wday(imparatorluk, gun_index)`: Belirtilen imparatorluk ve gün için tüm EXP bonus ayarlarını temizler.
    *   **Tatil EXP Ayarları:**
        *   `speedserver.get_holiday(imparatorluk, yil, ay, gun)`: Belirtilen imparatorluk ve tarih (yıl, ay, gün) için tanımlanmış EXP bonus zaman dilimlerini döndürür (get_wday gibi).
        *   `speedserver.set_holiday(imparatorluk, yil, ay, gun, bitis_saati, bitis_dakikasi, exp_yuzdesi)`: Belirtilen imparatorluk ve tarih için yeni bir EXP bonus zaman dilimi ekler/ayarlar.
        *   `speedserver.init_holiday(imparatorluk, yil, ay, gun)`: Belirtilen imparatorluk ve tarih için tüm EXP bonus ayarlarını temizler.
    *   **Mevcut Durum Sorgulama:**
        *   `speedserver.get_current_exp_priv(imparatorluk)`: Belirtilen imparatorluk için o an geçerli olan EXP bonusunu ve bir sonraki değişikliğe kadar kalan süreyi döndürür. Dönüş değerleri: [1] Bitiş Saati, [2] Bitiş Dakikası, [3] EXP Yüzdesi, [4] Kalan Süre (saniye), [5] Bir değişiklik olup olmadığı (bool).
*   **Veri Yapıları:**
    *   `HME (Hour, Minute, Exp)`: EXP bonus zaman dilimlerini temsil eden iç yapı.
    *   `Date`: Tarihleri temsil eden iç yapı.
*   **Kayıt:** `RegisterSpeedServerFunctionTable()` fonksiyonu, `speed_server_functions` dizisindeki tüm fonksiyonları `speedserver` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `SpeedServer.h`, `questlua.h`, `questmanager.h`.

### `questlua_target.cpp`

*   **Amaç:** Oyuncuya görevlerle ilgili hedefleri (belirli bir konum veya bir karakter/NPC) göstermek için Lua fonksiyonları sağlar. Mini haritada bir okla veya dünyada bir efektle hedef belirleme, hedefi silme ve hedefin VID'sini sorgulama gibi işlevleri içerir.
*   **Lua Ön Eki:** `target`
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `target.pos(hedef_etiketi, yerel_x, yerel_y, sure_saniye_ops, arg_string_ops, tip_ops)`: Mevcut karakterin bulunduğu haritada, yerel koordinatlarla (`yerel_x`, `yerel_y`) belirtilen bir konumu hedef olarak ayarlar.
        *   `hedef_etiketi` (string): Hedefin adı/etiketi (silmek için kullanılır).
        *   `yerel_x`, `yerel_y` (sayı): Haritanın başlangıç pozisyonuna göre x ve y koordinatları (100 ile çarpılır).
        *   `sure_saniye_ops` (sayı, opsiyonel): Hedefin ne kadar süreyle gösterileceği.
        *   `arg_string_ops` (string, opsiyonel): Hedefe tıklanınca çalıştırılacak özel bir argüman/komut.
        *   `tip_ops` (sayı, opsiyonel, varsayılan 1): Hedefin görsel tipi.
    *   `target.vid(hedef_etiketi, karakter_vid, arg_string_ops, tip_ops)`: Belirtilen VID'ye sahip bir karakteri (PC veya NPC) hedef olarak ayarlar.
        *   `karakter_vid` (sayı): Hedeflenecek karakterin VID'si.
        *   Diğer parametreler `target.pos` ile benzerdir.
    *   `target.npc(hedef_etiketi, npc_vid, arg_string_ops, tip_ops)`: `target.vid` ile aynı işleve sahiptir (eski veya alternatif isimlendirme).
    *   `target.pc(hedef_etiketi, pc_vid, arg_string_ops, tip_ops)`: `target.vid` ile aynı işleve sahiptir (eski veya alternatif isimlendirme).
    *   `target.delete(hedef_etiketi)`: Mevcut görev için belirtilen etikete sahip hedefi siler.
    *   `target.clear()`: Mevcut görev için tanımlanmış tüm hedefleri siler.
    *   `target.id(hedef_etiketi)`: Belirtilen etikete sahip hedefin, eğer bir VID hedefiyse (`TARGET_TYPE_VID`), VID'sini döndürür. Konum hedefi veya hedef yoksa 0 döndürür.
*   **İçsel Çalışma:**
    *   `CTargetManager::instance().CreateTarget` fonksiyonu ile hedefler oluşturulur.
    *   Hedefler oyuncu ID'si, görev indeksi ve hedef etiketi ile benzersiz olarak tanımlanır.
    *   Hedef bilgileri (`TargetInfo`) bir olay (event) olarak saklanır.
*   **Kayıt:** `RegisterTargetFunctionTable()` fonksiyonu, `target_functions` dizisindeki tüm fonksiyonları `target` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `char.h`, `sectree_manager.h`, `target.h`.

### `questlua_templeochao.cpp`

*   **Amaç:** (`__MT_THUNDER_DUNGEON__` makrosu tanımlıysa) Ochao Tapınağı zindanı ile ilgili temel bir başlatma işlevini Lua betiklerine açar.
*   **Lua Ön Eki:** `temple_ochao`
*   **Koşullu Derleme:** Bu dosyadaki fonksiyonlar yalnızca `__MT_THUNDER_DUNGEON__` makrosu tanımlı olduğunda kullanılabilir.
*   **Temel İşlevler/Örnek Fonksiyonlar:**
    *   `temple_ochao.initialize()`: Ochao Tapınağı yöneticisini (`TempleOchao::CMgr`) hazırlar (`Prepare`). Bu muhtemelen zindanın başlamadan önceki ilk kurulum adımlarını gerçekleştirir.
*   **Kayıt:** `RegisterTempleOchaoFunctionTable()` fonksiyonu, `temple_ochao_functions` dizisindeki fonksiyonları `temple_ochao` Lua tablosu altına kaydeder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `TempleOchao.h`, (`__MT_THUNDER_DUNGEON__` tanımlıysa: `questlua.h`, `questmanager.h`).

## Çekirdek Lua Entegrasyonu (`questlua.cpp`)

### `questlua.cpp`

*   **Amaç:** Bu dosya, görev sisteminin Lua motoru ile entegrasyonunun temelini oluşturur. Çeşitli `questlua_*` dosyalarında tanımlanan Lua fonksiyon tablolarını kaydeder, görev betiklerini çalıştırmak için yardımcı fonksiyonlar sağlar, parti üyeleri veya belirli alanlardaki karakterler üzerinde işlem yapmak için kullanılan functor (işlev nesnesi) tanımlarını içerir ve Lua betiklerini yürütme/değerlendirme işlevlerini barındırır.
*   **Ana Fonksiyonlar ve Yapılar:**
    *   **`ScriptToString(script_kodu)`:** Verilen bir Lua script kodunu (`return ...` formatında) çalıştırır ve sonucunu string olarak döndürür. Hata durumunda log kaydı oluşturur.
    *   **Functor'lar (İşlev Nesneleri):** Farklı senaryolarda karakter grupları üzerinde işlem yapmak için kullanılırlar:
        *   `FSetWarpLocation`: Karakterin ışınlanma noktasını ayarlar.
        *   `FSetQuestFlag`: Karakterin görev bayrağını ayarlar.
        *   `FPartyCheckFlagLt`: Parti üyelerinin belirli bir görev bayrağının verilen değerden küçük olup olmadığını kontrol eder.
        *   `FPartyChat`: Parti üyelerine sohbet mesajı gönderir.
        *   `FPartyClearReady`: Parti üyelerinin "hazır" efektini kaldırır.
        *   `FSendPacket`: Bir karaktere (entity) ağ paketi gönderir.
        *   `FSendPacketToEmpire`: Belirli bir imparatorluktaki karakterlere ağ paketi gönderir.
        *   `FWarpEmpire`: Belirli bir imparatorluktaki karakterleri ışınlar.
        *   `FBuildLuaGuildWarList`: Devam eden lonca savaşlarının listesini Lua tablosu olarak oluşturur.
    *   **`IsScriptTrue(script_kodu, boyut)`:** Verilen Lua script kodunu çalıştırır ve sonucunun `true` olup olmadığını döndürür. Hata durumunda log kaydı oluşturur.
    *   **`combine_lua_string(L, s)`:** Lua yığınındaki tüm string/sayı değerlerini bir `ostringstream` içinde birleştirir.
    *   **Highscore Fonksiyonları (`highscore_show`, `highscore_register`):** Yüksek skor tablolarını gösterme ve yeni skor kaydetme işlemlerini veritabanı üzerinden yapar.
    *   **Member Fonksiyonları (`member_chat`, `member_clear_ready`, `member_set_ready`):** Mevcut görev bağlamındaki parti üyesi (`GetCurrentPartyMember`) üzerinde işlem yapar (sohbet, hazır durumu).
    *   **Mob Fonksiyonları (`mob_spawn`, `mob_spawn_group`):** Belirtilen konuma canavar veya canavar grubu spawn eder. Spawn edilen ilk canavarın/liderin VID'sini döndürür. Opsiyonel olarak agresiflik, sayı ve belirli bir süre sonra otomatik silinme (`purge_time_event`) ayarlayabilir.
    *   **`CQuestManager::AddLuaFunctionTable(isim, fonksiyon_dizisi)`:** Verilen Lua fonksiyon dizisini (`luaL_reg`) belirtilen isim altında global bir Lua tablosu olarak kaydeder.
    *   **`CQuestManager::BuildStateIndexToName(gorev_adi)`:** Derlenmiş state dosyalarından state isimlerini ve indekslerini okuyarak Lua tarafında state indeksi -> state adı eşleşmesini kurar.
    *   **`CQuestManager::InitializeLua()`:** Görev sisteminin Lua motorunu başlatır. Temel Lua kütüphanelerini açar, tüm `Register*FunctionTable()` fonksiyonlarını çağırarak Lua arayüzlerini kaydeder, `settings.lua`, `questlib.lua`, `locale.lua` gibi temel betik dosyalarını yükler ve `object/state/` dizinindeki tüm görev state dosyalarını yükleyip işler.
    *   **State Yönetimi Fonksiyonları (`GotoSelectState`, `GotoConfirmState`, `GotoSelectItemState`, `GotoSelectItemExState`, `GotoInputState`, `GotoPauseState`, `GotoEndState`):** Bir görev betiği `yield` ettiğinde, hangi durumda askıya alındığını (`SUSPEND_STATE_*`) belirler ve istemciye uygun script komutunu (`[QUESTION ...]`, `[CONFIRM_WAIT ...]`, `[SELECT_ITEM]`, `[INPUT]`, `[NEXT]`, `[DONE]`) gönderir. `GotoConfirmState` ayrıca bir zaman aşımı olayı (`confirm_timeout_event`) kurar.
    *   **`CQuestManager::OpenState(gorev_adi, state_index)`:** Belirtilen görev ve state için yeni bir Lua coroutine oluşturur ve `QuestState` yapısını döndürür.
    *   **`CQuestManager::RunState(qs)`:** Verilen `QuestState`'deki coroutine'i devam ettirir (`lua_resume`). Coroutine'in dönüş değerine göre görevin bitip bitmediğini veya hangi durumda askıya alındığını belirler ve ilgili `Goto*State` fonksiyonunu çağırır. Hata durumunda log kaydı oluşturur.
    *   **`CQuestManager::CloseState(qs)`:** Görev durumuyla ilişkili Lua coroutine referansını kaldırır.
*   **Genel Akış:** `InitializeLua` tüm görevleri ve Lua fonksiyonlarını yükler. Bir olay (örn. NPC tıklaması) tetiklendiğinde, ilgili görev için `OpenState` ile bir coroutine başlatılır. `RunState` coroutine'i çalıştırır. Eğer script oyuncu etkileşimi gerektiriyorsa (`select`, `input`, `confirm` vb. dönerse), uygun `Goto*State` fonksiyonu çağrılır ve `RunState` `true` döner. Oyuncu etkileşimde bulunduğunda (örn. seçim yaptığında), `CQuestManager` ilgili fonksiyonu (`Select`, `Input` vb.) çağırır, bu fonksiyon coroutine'e oyuncunun girdisini argüman olarak iletir ve tekrar `RunState`'i çağırır. Script normal şekilde biterse (`return 0` ve `lua_gettop(qs.co) == 0`), `GotoEndState` çağrılır ve `RunState` `false` döner.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `sstream`, `questmanager.h`, `questlua.h`, `config.h`, `desc.h`, `char.h`, `char_manager.h`, `buffer_manager.h`, `db.h`, `xmas_event.h`, `locale_service.h`, `regen.h`, `affect.h`, `guild.h`, `guild_manager.h`, `sectree_manager.h`. Neredeyse tüm `questlua_*` dosyalarındaki kayıt fonksiyonlarını çağırır.

## Çekirdek Görev Sistemi Bileşenleri

### `questlua.h`

*   **Amaç:** Bu başlık dosyası, `questlua_*` dosyalarında tanımlanan çeşitli Lua fonksiyon tablolarını kaydetmek için kullanılan `Register*FunctionTable()` fonksiyonlarının bildirimlerini içerir. Ayrıca, görev sistemi içinde Lua ile etkileşimde kullanılan yardımcı yapılar (functor'lar) ve fonksiyon prototiplerini barındırır.
*   **Temel İçerik:**
    *   **Fonksiyon Bildirimleri (`Register*FunctionTable()`):**
        *   `RegisterPCFunctionTable()`, `RegisterNPCFunctionTable()`, `RegisterTargetFunctionTable()`, `RegisterAffectFunctionTable()`, `RegisterBuildingFunctionTable()`, `RegisterMarriageFunctionTable()`, `RegisterITEMFunctionTable()`, `RegisterDungeonFunctionTable()`, `RegisterQuestFunctionTable()`, `RegisterPartyFunctionTable()`, `RegisterHorseFunctionTable()`, `RegisterPetFunctionTable()` (`__PET_SYSTEM__`), `RegisterGuildFunctionTable()`, `RegisterGameFunctionTable()`, `RegisterArenaFunctionTable()`, `RegisterGlobalFunctionTable(lua_State* L)`, `RegisterForkedFunctionTable()`, `RegisterMonarchFunctionTable()`, `RegisterOXEventFunctionTable()`, `RegisterMgmtFunctionTable()`, `RegisterBattleArenaFunctionTable()`, `RegisterDanceEventFunctionTable()`, `RegisterDragonLairFunctionTable()`, `RegisterSpeedServerFunctionTable()`, `RegisterDragonSoulFunctionTable()`, `RegisterMeleyLairFunctionTable()` (`__GUILD_DRAGONLAIR__`), `RegisterTempleOchaoFunctionTable()` (`__MT_THUNDER_DUNGEON__`), `RegisterShipDefenseManagerFunctionTable()` (`__SHIP_DEFENSE__`), `RegisterAttr67AddFunctionTable()` (`__ATTR_6TH_7TH__`).
        *   Bu fonksiyonlar, kendi modüllerindeki Lua fonksiyonlarını (`luaL_reg` dizileri aracılığıyla) ilgili Lua tabloları altına (`pc`, `npc`, `target`, `q`, `pet` vb.) kaydetmek için `questlua.cpp` içindeki `CQuestManager::InitializeLua()` tarafından çağrılır.
    *   **Yardımcı Fonksiyon Bildirimleri:**
        *   `combine_lua_string(lua_State* L, std::ostringstream& s)`: Lua yığınındaki string/sayıları birleştirmek için kullanılır.
    *   **Functor (İşlev Nesnesi) Tanımları:**
        *   `FSetWarpLocation`: Karakterin ışınlanma noktasını ayarlamak için.
        *   `FSetQuestFlag`: Karakterin görev bayrağını ayarlamak için.
        *   `FPartyCheckFlagLt`: Parti üyelerinin bayrak değerini kontrol etmek için.
        *   `FPartyChat`: Parti üyelerine sohbet mesajı göndermek için.
        *   `FPartyClearReady`: Parti üyelerinin hazır durumunu temizlemek için.
        *   `FSendPacket`: Bir entity'ye paket göndermek için.
        *   `FSendPacketToEmpire`: Belirli bir imparatorluktaki entity'lere paket göndermek için.
        *   `FWarpEmpire`: Belirli bir imparatorluktaki karakterleri ışınlamak için.
        *   `FBuildLuaGuildWarList`: Lua için lonca savaş listesi oluşturmak için.
    *   **Olay Bilgi Yapısı ve Fonksiyon Bildirimi:**
        *   `EVENTINFO(warp_all_to_map_my_empire_event_info)`: Belirli bir imparatorluktaki oyuncuları başka bir haritaya ışınlama olayının bilgi yapısı.
        *   `EVENTFUNC(warp_all_to_map_my_empire_event)`: Yukarıdaki olayı işleyen fonksiyon bildirimi.
*   **Bağlantılı Dosyalar:** `quest.h`, `buffer_manager.h`. Bu dosya `questlua.cpp` ve tüm `questlua_*` dosyaları tarafından dahil edilir.

### `questmanager.h`

*   **Amaç:** Bu başlık dosyası, Metin2 sunucusundaki tüm görev (quest) sistemi mantığını yöneten merkezi `CQuestManager` sınıfını tanımlar. Görevlerin yüklenmesi, çalıştırılması, durum yönetimi, olay işleme ve Lua betikleriyle etkileşim gibi temel işlevlerden sorumludur.
*   **Temel Bileşenler:**
    *   **`CQuestManager` Sınıfı:**
        *   **Amacı:** Oyun içindeki görevlerin yönetimini üstlenen singleton bir sınıftır.
        *   **Sorumlulukları:**
            *   Lua betik motorunu başlatır ve yönetir (`lua_State* L`).
            *   Oyuncuya özel görev verilerini (`PC` sınıfı) ve durumlarını (`QuestState`) takip eder.
            *   Görevlerle ilişkili NPC'leri (`m_mapNPC`) ve olayları yönetir.
            *   Oyuncu girişi, canavar öldürme, eşya kullanma gibi çeşitli görev olaylarını işler.
            *   Global görev bayraklarını (`m_mapEventFlag`) yönetir.
            *   Görev betiklerini yükler ve çalıştırır.
            *   İstemciye görev arayüzü komutları gönderir.
    *   **Önemli Enumlar:**
        *   `QUEST_SKIN_NOWINDOW`: Görev arayüzü penceresi yok.
        *   `QUEST_SKIN_NORMAL`: Standart görev arayüzü penceresi.
        *   `QUEST_SKIN_SCROLL`: Kaydırma çubuklu görev arayüzü penceresi (genellikle uzun metinler için).
        *   `QUEST_SKIN_CINEMATIC`: Sinematik tarzda görev arayüzü.
    *   **Önemli Typedef'ler:**
        *   `TEventNameMap`: Olay isimlerini (string) tamsayı (`int`) karşılıklarına eşler.
        *   `PCMap`: Oyuncu ID'lerini (`unsigned int`) `PC` nesnelerine eşler.
    *   **Anahtar Public Metotlar (Özet):**
        *   `Initialize() / Destroy()`: Görev yöneticisini başlatır ve sonlandırır.
        *   `InitializeLua()`: Lua ortamını hazırlar.
        *   `GetLuaState()`: Aktif Lua durumunu döndürür.
        *   `OpenState() / CloseState() / RunState()`: Görev durumlarının yaşam döngüsünü yönetir.
        *   `GetPC() / GetPCForce()`: Bir oyuncunun görev verilerine (`PC` nesnesi) erişim sağlar.
        *   **Olay İşleme Fonksiyonları:** `Login()`, `Logout()`, `Timer()`, `Click()`, `Kill()`, `Damage()`, `Die()`, `LevelUp()`, `UseItem()`, `PickupItem()`, `SIGUse()`, `TakeItem()`, `Unmount()` vb. Bu fonksiyonlar ilgili oyun olayları gerçekleştiğinde çağrılır ve görev mantığını tetikler.
        *   `RegisterQuest() / GetQuestIndexByName() / GetQuestNameByIndex()`: Görevlerin sisteme kaydedilmesini ve isim/indeks ile sorgulanmasını sağlar.
        *   `SetEventFlag() / GetEventFlag()`: Oyun genelindeki görevle ilgili olay bayraklarını ayarlar ve okur.
        *   `ExecuteQuestScript()`: Belirtilen bir görev betiğini çalıştırır.
        *   `AddScript() / SendScript()`: İstemciye gönderilecek görev arayüzü komutlarını biriktirir ve gönderir.
        *   `AddServerTimer() / ClearServerTimer() / ServerTimer()`: Sunucu taraflı zamanlayıcıları yönetir.
        *   `Reload()`: Görev sistemini yeniden yükler.
        *   `BeginOtherPCBlock() / EndOtherPCBlock()`: Bir oyuncunun görev bağlamını geçici olarak başka bir oyuncu üzerinden çalıştırmak için kullanılır.
*   **Bağlantılı Dosyalar:** `questnpc.h`, `<boost/unordered_map.hpp>`. (Ayrıca `ITEM`, `CHARACTER`, `CDungeon` için sınıf bildirimleri içerir.)

### `questmanager.cpp`

*   **Amaç:** Bu dosya, `questmanager.h` içinde tanımlanan `CQuestManager` sınıfının ve yardımcı fonksiyonlarının implementasyonunu içerir. Görev sisteminin tüm çalışma mantığı, olayların işlenmesi, Lua entegrasyonu ve veri yönetimi burada gerçekleştirilir.
*   **Temel İşlevler ve Implementasyon Detayları:**
    *   **Başlatma (`Initialize`, `InitializeLua`):**
        *   `CQuestManager` oluşturulduğunda, `Initialize()` metodu çağrılır.
        *   `InitializeLua()`: Yeni bir Lua durumu (state) oluşturur, temel Lua kütüphanelerini ve görev sistemi için özel olarak yazılmış Lua fonksiyonlarını (örneğin, `questlua_*` dosyalarındaki) kaydeder.
        *   `m_mapEventName`: "click", "kill", "timer" gibi olay isimlerini sayısal sabitlerle eşleyen bir harita doldurulur. Bu, betiklerde olaylara referans vermeyi kolaylaştırır.
        *   `questnpc.txt` dosyası okunarak NPC'lerin görev betikleriyle olan ilişkileri `m_mapNPC` ve `m_mapNPCNameID` haritalarına yüklenir. Her NPC için bir `NPC` nesnesi oluşturulur ve bu nesne, o NPC ile tetiklenebilecek görev betiklerini tutar.
        *   Bazı başlangıç olay bayrakları (`event_flag`) ayarlanır.
    *   **Lua Etkileşimi ve Betik Çalıştırma:**
        *   `ExecuteQuestScript()`: Bir görevin belirli bir durumuna (state) ait Lua betiğini çalıştıran merkezi fonksiyondur.
            *   Betik kodunu alır ve Lua motorunda çalıştırılmak üzere yükler (`luaL_loadbuffer`).
            *   Performans için bir önbellekleme mekanizması (`__codecache`) kullanır; daha önce derlenmiş betikler tekrar derlenmez.
            *   Oyuncunun (`PC`) görev durumunu (`QuestState`) ayarlar ve Lua coroutine'ini başlatır/devam ettirir.
        *   `OpenState()`: Bir görev durumu için yeni bir Lua coroutine'i oluşturur.
        *   `RunState()`: Mevcut görev durumunun Lua coroutine'ini devam ettirir. Betik `suspend` durumuna geçtiğinde (örneğin, oyuncudan bir seçim beklerken), bu fonksiyon kontrolü geri alır. Oyuncudan girdi geldiğinde tekrar çağrılarak betiği devam ettirilir.
        *   `CloseState()`: Bir görev durumunu ve ilişkili Lua coroutine'ini temizler.
    *   **Olay Yönetimi:**
        *   `CQuestManager`, oyun içinde gerçekleşen çeşitli olaylara (event) tepki verir:
            *   `Login()`: Oyuncu oyuna girdiğinde.
            *   `Logout()`: Oyuncu oyundan çıktığında.
            *   `Kill()`: Oyuncu bir canavar öldürdüğünde. İlgili NPC (canavar) ve genel (`QUEST_NO_NPC`) için olay tetiklenir. Parti üyeleri için `OnPartyKill` de çağrılabilir.
            *   `Timer()`: Bir oyuncuyla ilişkili bir görev zamanlayıcısı dolduğunda.
            *   `Click()`: Oyuncu bir NPC'ye tıkladığında. NPC'nin `OnClick` veya varsa `OnChat` betiği çalıştırılır.
            *   `UseItem()`: Oyuncu bir eşyayı kullandığında. Eşyanın VNUM'u ile ilişkili NPC'nin (`m_mapNPC[item->GetVnum()]`) `OnUseItem` betiği çalıştırılır.
            *   `SIGUse()`: Özel eşya grubu (Special Item Group) kullanıldığında.
            *   `TakeItem()`: Oyuncu bir NPC'den eşya aldığında.
            *   `PickupItem()`: Oyuncu yerden bir eşya topladığında.
            *   `LevelUp()`: Oyuncu seviye atladığında.
            *   `AttrIn() / AttrOut()`: Oyuncu belirli bir bölgeye (attribute) girdiğinde/çıktığında.
            *   `Target()`: Bir hedefe yönelik görev eylemi gerçekleştirildiğinde.
            *   `Damage()`: Oyuncu hasar aldığında veya verdiğinde.
            *   `Die()`: Oyuncu öldüğünde.
        *   Bu olay fonksiyonları genellikle oyuncunun `PC` nesnesini alır, gerekli kontrolleri yapar ve ilgili NPC'nin (`NPC` sınıfı instance'ı) olay işleyici fonksiyonunu çağırır. Bu NPC fonksiyonları da sonuçta Lua betiklerini tetikler.
    *   **Görev Durumları ve Oyuncu Etkileşimleri:**
        *   Görevler genellikle birden fazla durumdan (state) oluşur. `CQuestManager` bu durumlar arasındaki geçişleri yönetir.
        *   `Select()`, `Input()`, `Confirm()`, `SelectItem()`: Oyuncunun görev arayüzü üzerinden yaptığı seçimler, girdiği metinler veya onaylamalar bu fonksiyonlar aracılığıyla işlenir.
            *   Bu fonksiyonlar, ilgili oyuncunun çalışmakta olan görev betiğinin `SUSPEND_STATE_SELECT`, `SUSPEND_STATE_INPUT` gibi bir durumda askıda olduğunu kontrol eder.
            *   Oyuncunun girdisi Lua yığınına (stack) aktarılır ve `RunState()` çağrılarak askıdaki betik devam ettirilir.
    *   **NPC Yönetimi:**
        *   `m_mapNPC`: NPC VNUM'larını `NPC` nesnelerine eşler. Her `NPC` nesnesi, farklı olaylar ("click", "kill" vb.) için farklı görev betiklerini tutabilir.
        *   `RegisterNPCVnum()`: Dosya sisteminde belirli bir VNUM için görev dosyaları olup olmadığını kontrol eder ve varsa bu VNUM'u kayıtlı NPC'ler arasına ekler.
        *   NPC ile ilgili olaylar (tıklama, konuşma) genellikle `m_mapNPC[npc_vnum]` üzerinden ilgili `NPC` nesnesine yönlendirilir.
    *   **Olay Bayrakları (Event Flags):**
        *   `m_mapEventFlag`: Oyun genelindeki olay durumlarını (örneğin, bir event aktif mi, drop oranları nedir vb.) tutan bir haritadır.
        *   `SetEventFlag()`: Bir olay bayrağının değerini değiştirir. Bu değişiklik, oyun mekaniklerini doğrudan etkileyebilir (örneğin, `mob_item` bayrağı canavarlardan düşen eşya oranını, `eclipse` bayrağı gece/gündüz döngüsünü etkiler). Ayrıca event NPC'lerinin spawn/despawn edilmesi gibi işlemleri de tetikleyebilir.
        *   `GetEventFlag()`: Bir olay bayrağının mevcut değerini döndürür.
        *   `RequestSetEventFlag()`: Bir olay bayrağı değişikliğinin kalıcı olması için veritabanı sunucusuna istek gönderir.
        *   `SendEventFlagList()`: Bir oyuncuya (genellikle GM) mevcut tüm olay bayraklarını ve değerlerini listeler.
        *   `BroadcastEventFlagOnLogin()`: Oyuncu giriş yaptığında, aktif olan bazı önemli olay bayraklarını (örneğin, `xmas_snow`) istemciye komut olarak gönderir, böylece istemci tarafında ilgili görsel efektler aktifleşir.
    *   **İstemci İletişimi (`AddScript`, `SendScript`):**
        *   Görev betikleri çalışırken, oyuncunun istemcisinde arayüz değişiklikleri (pencere açma, metin gösterme, seçenek sunma vb.) yapılması gerekebilir.
        *   `AddScript()`: Bu tür istemci komutlarını (`[WINDOW_TITLE]Başlık`, `[TEXT]Metin` gibi özel formatlı stringler) bir ara belleğe (`m_strScript`) ekler.
        *   `SendScript()`: Ara bellekte biriken komutları `HEADER_GC_SCRIPT` paketiyle istemciye gönderir.
        *   `SetSkinStyle()`: Gönderilecek komutlarla birlikte hangi görev penceresi stilinin (`QUEST_SKIN_NORMAL`, `QUEST_SKIN_SCROLL` vb.) kullanılacağını belirler.
    *   **Sunucu Zamanlayıcıları (`AddServerTimer`, `ClearServerTimer`, `ServerTimer`):**
        *   Belirli bir süre sonra sunucuda bir görev eyleminin tetiklenmesini sağlar. Örneğin, "5 dakika sonra X NPC'sini Y haritasında spawn et" gibi.
        *   `m_mapServerTimer`: Aktif sunucu zamanlayıcılarını ve ilişkili olaylarını tutar.
        *   `ServerTimer()`: Bir zamanlayıcı dolduğunda çağrılan olay işleyicisidir ve genellikle belirli bir NPC ile ilişkili bir görev betiğini tetikler.
    *   **Hata Yönetimi ve Yeniden Yükleme:**
        *   `QuestError()`: Görev sistemi içinde bir hata oluştuğunda loglama yapar.
        *   `WriteRunningStateToSyserr()`: Bir Lua hatası durumunda, hangi görevde ve durumda hata oluştuğuna dair bilgi loglar.
        *   `Reload()`: `/reload q` gibi bir komutla tetiklenir. Çalışan tüm görevleri durdurur, Lua durumunu kapatıp yeniden başlatır, NPC ve görev bilgilerini tekrar yükler. Bu, sunucuyu yeniden başlatmadan görevlerde değişiklik yapmayı sağlar.
    *   **Diğer Önemli Fonksiyonlar:**
        *   `GetQuestStateIndex() / GetQuestStateName()`: Görev ismi ve durum ismi/indeksi arasında dönüşüm yapar.
        *   `CanStartQuest()`: Bir görevin başlama koşullarının (genellikle `begin_condition` altındaki betikler) oyuncu için uygun olup olmadığını kontrol eder.
        *   `LoadStartQuest()`: Bir görevin başlama koşulu betiğini yükler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `buffer_manager.h`, `packet.h`, `desc_client.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `questmanager.h`, `lzo_manager.h`, `item.h`, `config.h`, `xmas_event.h`, `target.h`, `party.h`, `locale_service.h`, `dungeon.h`, `<fstream>`. 

### `questnpc.h`

*   **Amaç:** `quest` namespace'i içinde `NPC` sınıfını tanımlar. Bu sınıf, görev sistemi bağlamında bir NPC'yi temsil eder ve o NPC ile ilişkili çeşitli olaylar (tıklama, öldürme, konuşma vb.) için görev betiklerine referanslar tutar.
*   **Temel İşlevler/İçerik:**
    *   **`NPC` Sınıfı:**
        *   NPC'nin VNUM'unu (`m_vnum`) saklar.
        *   Görev olaylarını (`QUEST_CLICK_EVENT` gibi sabitlerle temsil edilen), quest indekslerini, state indekslerini ve bunlara karşılık gelen Lua betiklerini (veya argümanlı betikleri) ilişkilendiren haritalar (`m_mapOwnQuest`, `m_mapOwnArgQuest`) içerir.
        *   Çeşitli olaylar (`OnClick`, `OnKill`, `OnTimer`, `OnLogin` vb.) için metotlar sağlar. Bu metotlar genellikle `CQuestManager` tarafından çağrılır ve ilgili NPC için uygun betiğin çalıştırılmasını tetikler.
        *   Oyuncunun mevcut görev durumlarına ve NPC'nin sahip olduğu betiklere göre ilgili görevleri verimli bir şekilde bulmak için kullanılan `MatchingQuest` şablon fonksiyonunu içerir.
        *   `LoadStateScript`: Betik verilerini dosyalardan yüklemek için yardımcı bir metot.
        *   `Set`: NPC nesnesini başlatır. VNUM'una ve betik adına (genellikle VNUM'dan türetilir) göre dosya sisteminden ilgili tüm görev betiklerini yükler.
    *   **Typedef'ler:** `AQuestScriptType`, `QuestMapType`, `AArgQuestScriptType`, `ArgQuestMapType`. Bu typedef'ler, görev betiklerini olay, görev indeksi ve state bazında organize etmek için kullanılır.
    *   **Enum'lar:** `QUEST_START_STATE_INDEX`, `QUEST_CHAT_STATE_INDEX` gibi özel görev state'leri için sabitler içerir.
*   **Bağlantılı Dosyalar:** `questpc.h` (PC sınıfı tanımı ve görev yapıları için). Bu dosya `questmanager.h` tarafından dahil edilir.

### `questnpc.cpp`

*   **Amaç:** `questnpc.h` içinde tanımlanan `NPC` sınıfının metotlarını uygular. NPC'ler için görev betiklerinin yüklenmesini ve oyun içindeki olaylara yanıt olarak bu betiklerin tetiklenmesini yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`NPC::Set`:** NPC için görev betiklerini okur. Kayıtlı tüm olay isimleri (`CQuestManager::m_mapEventName`) ve nesne dizinleri (`g_setQuestObjectDir`) üzerinde döner. Her olay için, `object_dir/script_name/event_name/` altında ilgili betik dosyalarını arar. `opendir`/`readdir` kullanarak betik dosyalarını (`*.start`, `*.when`, `*.arg`, `*.script` gibi uzantılarla) bulur ve her biri için `LoadStateScript` fonksiyonunu çağırır.
    *   **`NPC::LoadStateScript`:** Betik dosya adını ayrıştırarak görev adını, state adını ve (varsa) argüman indeksini/tipini ("when", "arg", "script") belirler. Betik içeriğini (kod, 'when' koşulu veya argüman string'i) okur ve uygun haritaya (`m_mapOwnQuest` veya `m_mapOwnArgQuest`) yerleştirir. Görev/state isimlerini indekslere çevirmek için `CQuestManager`'ı kullanır.
    *   **Olay İşleyici Metotlar (`OnClick`, `OnKill`, `OnTimer` vb.):**
        *   Çoğu olay işleyici, işi `HandleEvent` veya `HandleReceiveAllEvent` fonksiyonlarına devreder.
        *   `HandleEvent`: Genellikle olayın belirli bir NPC ile ilişkili olduğu durumlarda (örn. *bu* NPC'ye tıklama) kullanılır. Oyuncunun mevcut görev durumları ve bu NPC'nin ilgili olay için sahip olduğu betikler (hem mevcut state için hem de başlangıç state'i için) arasında eşleşme arar. Eşleşen betikleri `CQuestManager::ExecuteQuestScript` aracılığıyla çalıştırır.
        *   `HandleReceiveAllEvent`: Genellikle genel olaylar (seviye atlama, oyundan çıkma) veya `QUEST_NO_NPC` gibi özel NPC'ler için (örn. herhangi bir canavar öldüğünde) kullanılır. Belirli bir olay için bu NPC ile ilişkili *tüm* görevleri kontrol eder. Oyuncunun o görevde uygun bir state'de olup olmadığını veya görevin başlangıç state'inin çalıştırılıp çalıştırılamayacağını kontrol eder ve uygun betikleri çalıştırır.
        *   `HandleReceiveAllNoWaitEvent`: `HandleReceiveAllEvent`'e benzer, ancak Login/Logout gibi olaylar için kullanılır. Bu olaylarda çalıştırılan betiklerin askıya alınmasına (yield) izin verilmez; eğer betik hemen bitmezse zorla sonlandırılır.
    *   **`OnTarget`:** Özel olarak `QUEST_TARGET_EVENT` olayını işler. `m_mapOwnArgQuest` haritasında görev indeksi, state, hedef adı ve fiil ('verb') ile eşleşen betikleri arar. Eğer bir eşleşme bulunursa ve (varsa) 'when' koşulu sağlanırsa, ilgili betiği çalıştırır.
    *   **`OnChat`:** `QUEST_CHAT_EVENT` olayını işler. Oyuncunun mevcut görev durumları veya başlayabileceği görevler için geçerli olan tüm 'chat' betiklerini ('when' koşulları dahil) toplar. Bu betiklerin argümanlarını (`arg` alanları, genellikle konuşma seçenekleri) bir `select()` Lua fonksiyon çağrısı içinde birleştirir ve geçici bir görev (`QUEST_CHAT_TEMP_QUEST`) üzerinden çalıştırarak oyuncuya seçenek olarak sunar.
    *   **`MatchingQuest` (Şablon Fonksiyon):** İlgili görevleri bulmak için kullanılan çekirdek mantıktır. Oyuncunun aktif görevleri listesi (sıralı) ve NPC'nin belirli bir olay için sahip olduğu görev betikleri haritası (sıralı) üzerinde aynı anda ilerler. Görev ID'leri eşleştiğinde `fMatch` functor'ını, NPC'de olup oyuncuda olmayan (başlama potansiyeli olan) görevler için `fMiss` functor'ını çağırır. Farklı olay türleri için farklı `fMatch`/`fMiss` functor'ları kullanılır (örn. `FuncMatchHandleEvent`, `FuncMissChatEvent`).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `questmanager.h`, `profiler.h`, `config.h`, `char.h`, `desc.h`, `<fstream>`, `<sstream>`. `CQuestManager` sınıfını yoğun bir şekilde kullanır. 

### `questpc.h`

*   **Amaç:** `quest` namespace'i içinde `PC` sınıfını tanımlar. Bu sınıf, bir oyuncunun (Player Character) görevlerle ilgili tüm verilerini ve durumlarını yönetir. Oyuncunun sahip olduğu görevler, görev bayrakları (flag), zamanlayıcılar (timer), bekleyen ödüller ve çalışan görev betiği hakkındaki bilgileri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`RewardData` Struct:**
        *   Görev ödüllerini temsil eder (EXP veya Eşya).
        *   `type` (`REWARD_TYPE_EXP`, `REWARD_TYPE_ITEM`), `value1` (EXP miktarı veya eşya VNUM'u), `value2` (eşya sayısı) üyelerini içerir.
    *   **`PC` Sınıfı:**
        *   **Enum'lar:**
            *   `QUEST_SEND_*`: İstemciye hangi görev bilgilerinin (başlık, saat, sayaç, ikon) gönderileceğini belirten bit maskesi sabitleri.
            *   `EQuestLen`: Görev başlığı, saat adı gibi stringlerin maksimum uzunlukları.
            *   `EQuestType`, `EQuestSkin` (`__QUEST_RENEWAL__` tanımlıysa): Görev tipleri (ana, alt, olay vb.) ve görev penceresi skin'leri için enumlar.
        *   **Typedef'ler:**
            *   `QuestInfo`: `map<unsigned int, QuestState>` için bir takma ad. Görev indeksini `QuestState` nesnesine eşler.
            *   `QuestInfoIterator`: `QuestInfo` haritası için iterator.
        *   **Üye Değişkenler (Önemlileri):**
            *   `m_dwID`: Oyuncunun kimliği (PID).
            *   `m_QuestInfo`: Oyuncunun sahip olduğu tüm görevlerin durumlarını (`QuestState`) tutan harita.
            *   `m_RunningQuestState` (`QuestState*`): Şu anda çalışmakta olan görevin `QuestState` işaretçisi.
            *   `m_stCurQuest`: Çalışan görevin adı.
            *   `m_FlagMap` (`map<string, int>`): Oyuncunun görevlerle ilgili tüm bayraklarını (flag) ve değerlerini tutar (örn: "gorev_adi.durum_adi" = 1).
            *   `m_FlagSaveMap`: Veritabanına kaydedilmesi gereken bayrakları geçici olarak tutar.
            *   `m_TimerMap` (`map<string, LPEVENT>`): Oyuncuya özel görev zamanlayıcılarını tutar.
            *   `m_vRewardData` (`vector<RewardData>`): Oyuncuya verilecek bekleyen ödülleri tutar.
            *   `m_bLoaded`: Oyuncunun görev verilerinin yüklenip yüklenmediğini belirtir.
            *   `m_QuestStateChange` (`vector<TQuestStateChangeInfo>`): Bekleyen görev durumu değişikliklerini tutar.
        *   **Metotlar (Önemlileri):**
            *   `SetID()`: Oyuncu ID'sini ayarlar.
            *   `SetFlag() / GetFlag() / DeleteFlag()`: Görev bayraklarını ayarlar, alır veya siler. Değişiklikleri `m_FlagSaveMap`'e de yazar.
            *   `SetQuest()`: Bir görevi oyuncu için aktif hale getirir, `m_RunningQuestState`'i ayarlar.
            *   `EndRunning() / CancelRunning()`: Çalışan görevi sonlandırır veya iptal eder.
            *   `SetQuestState()`: Bir görevin durumunu değiştirir. Değişikliği hemen uygulamaz, `m_QuestStateChange` listesine ekler.
            *   `DoQuestStateChange()`: `m_QuestStateChange` listesindeki bekleyen durum değişikliklerini işler (`LeaveState`, `EnterState`, `Letter` olaylarını tetikler).
            *   `AddTimer() / RemoveTimer() / ClearTimer()`: Oyuncuya özel görev zamanlayıcılarını yönetir.
            *   `SetCurrentQuestTitle()`, `SetCurrentQuestClockName()` vb.: Çalışan görevin istemcideki görünümünü (başlık, saat, sayaç, ikon) ayarlar ve `SendQuestInfoPacket()` için ilgili bayrakları işaretler.
            *   `SendQuestInfoPacket()`: İstemciye güncellenmiş görev bilgilerini gönderir.
            *   `Save()`: `m_FlagSaveMap`'teki bayrakları veritabanına kaydetmek üzere paketler.
            *   `GiveItem() / GiveExp()`: Ödül olarak eşya veya EXP verilmesini `m_vRewardData`'ya ekler.
            *   `Reward()`: `m_vRewardData`'daki bekleyen ödülleri karaktere verir.
            *   `Build()`: `m_FlagMap`'teki "__status" bayraklarından `m_QuestInfo`'yu yeniden oluşturur.
            *   `ClearQuest()`: Belirli bir göreve ait tüm bayrakları siler ve state'ini sıfırlar.
            *   `SetConfirmWait() / ClearConfirmWait() / IsConfirmWait()`: Başka bir oyuncudan görev onayı bekleme durumunu yönetir.
*   **Bağlantılı Dosyalar:** `quest.h` (temel görev yapıları için), `questpc.cpp` (uygulama).

### `questpc.cpp`

*   **Amaç:** `questpc.h` içinde tanımlanan `PC` sınıfının metotlarını uygular. Oyuncunun görev verilerinin yönetimi, görev bayraklarının işlenmesi, zamanlayıcıların kontrolü, görev durum değişikliklerinin ve ödüllerin yönetilmesi gibi işlevleri gerçekleştirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`PC::PC()`):** Tüm üye değişkenleri varsayılan değerlerine başlatır (örneğin, `m_RunningQuestState` NULL, `m_bLoaded` false).
    *   **Bayrak Yönetimi (`SetFlag`, `GetFlag`, `DeleteFlag`):**
        *   `SetFlag`: Bir bayrağın değerini `m_FlagMap`'te ayarlar veya günceller. Eğer değer 0 ise, `DeleteFlag` çağrılır. `bSkipSave` false ise, bayrağı `m_FlagSaveMap`'e de ekleyerek veritabanına kaydedilmek üzere işaretler.
        *   `GetFlag`: `m_FlagMap`'ten bayrağın değerini okur. Yoksa 0 döndürür.
        *   `DeleteFlag`: Bayrağı `m_FlagMap`'ten siler ve değerini 0 olarak `SaveFlag` ile kaydeder.
        *   `SaveFlag`: Bayrağı ve değerini `m_FlagSaveMap`'e ekler veya günceller.
    *   **Görev Durumu Yönetimi:**
        *   `SetCurrentQuestStateName`: Lua'dan çağrıldığında, çalışan görevin durumunu isme göre ayarlar.
        *   `SetQuestState`: Bir görevin durumunu (isim veya indeks ile) değiştirir. Eğer mevcut durumdan farklıysa, `AddQuestStateChange`'i çağırır.
        *   `AddQuestStateChange`: Görev durumu değişikliğini (`TQuestStateChangeInfo`) `m_QuestStateChange` vektörüne ekler. Asıl değişiklik `DoQuestStateChange`'de yapılır.
        *   `DoQuestStateChange`: `m_QuestStateChange` vektöründeki tüm bekleyen durum değişikliklerini işler. Her değişiklik için:
            *   `CQuestManager::LeaveState` (eski durum için) çağrılır.
            *   Oyuncunun `m_QuestInfo`'sundaki state güncellenir.
            *   Yeni state için "__status" bayrağı ayarlanır.
            *   `CQuestManager::EnterState` (yeni durum için) çağrılır.
            *   Eğer bayrak başarıyla ayarlandıysa, `CQuestManager::Letter` çağrılır.
    *   **Çalışan Görev Yönetimi (`SetQuest`, `EndRunning`, `CancelRunning`):**
        *   `SetQuest`: Belirtilen görevi oyuncunun çalışan görevi olarak ayarlar. `m_stCurQuest` ve `m_RunningQuestState`'i günceller. Görevin "__status" bayrağını mevcut state'e ayarlar.
        *   `EndRunning`: Çalışan görevi sonlandırır. NPC kilidini açar (varsa), bekleyen ödülleri verir (`Reward()`), veritabanına kaydeder (`Save()`), istemciye son görev bilgilerini gönderir (`SendQuestInfoPacket()`). Eğer state değişmişse `LeaveState`/`EnterState`/`Letter` olaylarını tetikler. Bekleyen `DoQuestStateChange`'leri çalıştırır.
        *   `CancelRunning`: Çalışan görevi iptal eder, `m_RunningQuestState`'i sıfırlar.
    *   **İstemci Bilgilendirme (`SendQuestInfoPacket`, `SetCurrentQuest*` fonksiyonları):**
        *   `SetCurrentQuestTitle`, `SetCurrentQuestClockName` vb. fonksiyonlar, çalışan görevin (`m_RunningQuestState`) ilgili alanını günceller ve `m_iSendToClient` bit maskesinde ilgili bayrağı işaretler.
        *   `SendQuestInfoPacket`: `m_iSendToClient`'da işaretlenmiş bilgilere göre `packet_quest_info` paketini oluşturur ve istemciye gönderir. `__QUEST_RENEWAL__` tanımlıysa görev tipi ve onay durumu da pakete eklenir.
    *   **Zamanlayıcı Yönetimi (`AddTimer`, `RemoveTimer`, `ClearTimer`):**
        *   `AddTimer`: Verilen isimde bir zamanlayıcıyı oyuncunun `m_TimerMap`'ine ekler (varsa eskisini siler).
        *   `RemoveTimer`: Zamanlayıcıyı haritadan siler ve ilişkili event'i iptal eder (`CancelTimerEvent`).
        *   `RemoveTimerNotCancel`: Zamanlayıcıyı sadece haritadan siler, event'i iptal etmez.
        *   `ClearTimer`: Tüm zamanlayıcıları iptal eder ve haritayı temizler.
    *   **Veritabanı Kayıt (`Save`):**
        *   `m_FlagSaveMap`'teki tüm bayrakları (görev adı, state adı ve değer olarak ayrıştırarak) `TQuestTable` yapılarına dönüştürür ve `HEADER_GD_QUEST_SAVE` paketi ile DB sunucusuna gönderir. Ardından `m_FlagSaveMap`'i temizler.
    *   **Ödül Yönetimi (`GiveItem`, `GiveExp`, `Reward`):**
        *   `GiveItem`, `GiveExp`: Verilecek ödülü (eşya veya EXP) `m_vRewardData` vektörüne ekler. Eğer aynı etiketli ödül daha önce verilmişse (`GetFlag` ile kontrol edilir), `m_bIsGivenReward`'ı true yapar.
        *   `Reward`: `m_vRewardData`'daki tüm ödülleri karaktere uygular (`ch->PointChange(POINT_EXP, ...)` veya `ch->AutoGiveItem(...)`). `m_bIsGivenReward` true ise "zaten ödül alındı" mesajı gönderir. Sonra `m_vRewardData`'yı temizler.
    *   **Diğer Fonksiyonlar:**
        *   `Build`: Başlangıçta, `m_FlagMap`'teki "__status" bayraklarını okuyarak `m_QuestInfo` haritasını (oyuncunun sahip olduğu görevlerin state'lerini) oluşturur.
        *   `ClearQuest`: Belirtilen bir göreve ait tüm bayrakları siler ve state'ini 0 yapar.
        *   `SendFlagList`: Oyuncunun tüm bayraklarını (ve "__status" bayrakları için state isimlerini) GM'e sohbet mesajı olarak gönderir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `questmanager.h`, `packet.h`, `buffer_manager.h`, `char.h`, `desc.h`, `<fstream>`, `<sstream>`. `CQuestManager` sınıfını yoğun bir şekilde kullanır. 