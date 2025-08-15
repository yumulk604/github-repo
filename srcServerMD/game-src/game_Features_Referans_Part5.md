# Metin2 Oyun Sunucusu - Özellikler Referansı Part 5 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans_Part4.md`](game_Features_Referans_Part4.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) çeşitli özellikleri ve sistemleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`fishing_event.cpp`](#fishing_eventcpp)
*   [`fishing_event.h`](#fishing_eventh)
*   [`fly.cpp`](#flycpp)
*   [`fly.h`](#flyh)
*   [`gm.cpp`](#gmcpp)
*   [`gm.h`](#gmh)
*   [`guild.cpp`](#guildcpp)
*   [`guild.h`](#guildh)
*   [`guild_manager.cpp`](#guild_managercpp)
*   [`guild_manager.h`](#guild_managerh)
*   [`guild_war.cpp`](#guild_warcpp)
*   [`horse_rider.cpp`](#horse_ridercpp)
*   [`horse_rider.h`](#horse_riderh)
*   [`horsename_manager.cpp`](#horsename_managercpp)
*   [`horsename_manager.h`](#horsename_managerh)
*   [`item.cpp`](#itemcpp)
*   [`item.h`](#itemh)
*   [`item_addon.cpp`](#item_addoncpp)
*   [`item_addon.h`](#item_addonh)
*   [`item_attribute.cpp`](#item_attributecpp)
*   [`item_attribute.h`](#item_attributeh)
*   [`item_manager.cpp`](#item_managercpp)
*   [`item_manager.h`](#item_managerh)
*   [`item_manager_private.h`](#item_manager_privateh)
*   [`item_manager_read_tables.cpp`](#item_manager_read_tablescpp)
*   [`lzo_manager.cpp`](#lzo_managercpp)
*   [`lzo_manager.h`](#lzo_managerh)
*   [`main.cpp`](#maincpp)
*   [`map_location.cpp`](#map_locationcpp)
*   [`map_location.h`](#map_locationh)
*   [`MarkImage.cpp`](#MarkImagecpp)
*   [`MarkImage.h`](#MarkImageh)
*   [`MarkManager.cpp`](#MarkManagercpp)
*   [`MarkManager.h`](#MarkManagerh)
*   [`marriage.cpp`](#marriagecpp)
*   [`marriage.h`](#marriageh)
*   [`MeleyLair.cpp`](#MeleyLaircpp)
*   [`MeleyLair.h`](#MeleyLairh)
*   [`mining.cpp`](#miningcpp)
*   [`mining.h`](#miningh)
*   [`mob_manager.cpp`](#mob_managercpp)
*   [`mob_manager.h`](#mob_managerh)
*   [`monarch.cpp`](#monarchcpp)
*   [`monarch.h`](#monarchh)
*   [`motion.cpp`](#motioncpp)
*   [`motion.h`](#motionh)
*   [`over9refine.cpp`](#over9refinecpp)
*   [`over9refine.h`](#over9refineh)
*   [`OXEvent.cpp`](#OXEventcpp)
*   [`OXEvent.h`](#OXEventh)
*   [`packet_analysis.cpp`](#packet_analysiscpp)
*   [`packet_info.cpp`](#packet_infocpp)
*   [`PetSystem.cpp`](#PetSystemcpp)
*   [`PetSystem.h`](#PetSystemh)
*   [`polymorph.cpp`](#polymorphcpp)
*   [`polymorph.h`](#polymorphh)
*   [`pool.h`](#poolh)
*   [`priv_manager.cpp`](#priv_managercpp)
*   [`priv_manager.h`](#priv_managerh)
*   [`pvp.cpp`](#pvpcpp)
*   [`pvp.h`](#pvph)
*   [`refine.h`](#refineh)
*   [`refine.cpp`](#refinecpp)
*   [`ShipDefense.h`](#shipdefenseh)
*   [`ShipDefense.cpp`](#shipdefensecpp)

---

### `pvp.h`

*   **Amaç:** Oyuncular arası PvP (Player versus Player) düellolarını yönetmek için `CPVP` ve `CPVPManager` sınıflarını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`CPVP` Sınıfı:**
        *   Tek bir PvP örneğini temsil eder.
        *   **`TPlayer` Struct:** PvP'ye katılan bir oyuncunun bilgilerini (PID, VID, kabul durumu, intikam hakkı) tutar.
        *   **Yapıcılar:** İki oyuncu ID'si ile veya bir `CPVP` referansı ile PvP nesnesi oluşturur. Oyuncu ID'lerinden bir CRC (Cyclic Redundancy Check) değeri hesaplar.
        *   `Win(DWORD dwPID)`: Belirtilen oyuncunun kazandığını ayarlar, diğer oyuncuya intikam hakkı tanır.
        *   `CanRevenge(DWORD dwPID)`: Belirtilen oyuncunun intikam alıp alamayacağını kontrol eder.
        *   `IsFight()`: İki oyuncunun da düelloyu kabul edip etmediğini (dövüş durumu) kontrol eder.
        *   `Agree(DWORD dwPID)`: Belirtilen oyuncunun düelloyu kabul ettiğini işaretler.
        *   `SetVID(DWORD dwPID, DWORD dwVID)`: Oyuncunun VID (Virtual ID - oyundaki görünür ID'si) değerini ayarlar.
        *   `Packet(bool bDelete = false)`: PvP durumunu (başlangıç, dövüş, intikam, bitiş) ilgili oyunculara ve diğer oyunculara paketle bildirir.
        *   `SetLastFightTime()`, `GetLastFightTime()`: Son dövüş zamanını ayarlar ve alır (zaman aşımı kontrolü için).
        *   `GetCRC()`: PvP örneğinin CRC değerini döndürür.
        *   **Korunan Üyeler:** `m_players[2]` (iki oyuncunun `TPlayer` verisi), `m_dwCRC` (PvP'nin CRC'si), `m_bRevenge` (intikam durumu), `m_dwLastFightTime`.
    *   **`CPVPManager` Sınıfı (Singleton):**
        *   Tüm aktif PvP örneklerini yönetir.
        *   `CPVPSetMap` Typedef'i: Oyuncu ID'sine göre PvP örneklerini (`CPVP*`) gruplamak için kullanılır.
        *   **Yapıcı/Yıkıcı:** Singleton için standart.
        *   `Insert(LPCHARACTER pkChr, LPCHARACTER pkVictim)`: İki karakter arasında yeni bir PvP isteği oluşturur veya mevcut bir isteği kabul etmelerini sağlar.
        *   `CanAttack(LPCHARACTER pkChr, LPCHARACTER pkVictim)`: Bir karakterin diğerine saldırıp saldıramayacağını çeşitli oyun kurallarına (PK modu, imparatorluk, parti, lonca, PvP durumu vb.) göre belirler.
        *   `Dead(LPCHARACTER pkChr, DWORD dwKillerPID)`: PvP sırasında bir karakter öldüğünde çağrılır. Eğer ölüm geçerli bir PvP dövüşü sonucundaysa kazananı belirler.
        *   `GiveUp(LPCHARACTER pkChr, DWORD dwKillerPID)`: Bir karakterin PvP'den pes etmesi durumunu işler (henüz kullanılmıyor gibi).
        *   `Connect(LPCHARACTER pkChr)`, `Disconnect(LPCHARACTER pkChr)`: Oyuncu oyuna girdiğinde veya çıktığında PvP durumlarını günceller.
        *   `SendList(LPDESC d)`: Belirli bir istemciye (`d`) mevcut tüm PvP durumlarını gönderir.
        *   `Delete(CPVP* pkPVP)`: Belirtilen PvP örneğini yöneticiden siler.
        *   `Process()`: Periyodik olarak çağrılarak zaman aşımına uğramış PvP örneklerini temizler.
        *   `Find(DWORD dwCRC)`: Verilen CRC ile bir PvP örneği bulur.
        *   **Korunan Üyeler:** `m_map_pkPVP` (CRC -> `CPVP*` haritası), `m_map_pkPVPSetByID` (Oyuncu PID -> `CPVP*` seti haritası).
*   **Bağlantılı Dosyalar:** `pvp.cpp` (uygulama), `stdafx.h` (temel başlıklar), `char.h` (`LPCHARACTER`).

---

### `pvp.cpp`

*   **Amaç:** `pvp.h` dosyasında bildirilen `CPVP` ve `CPVPManager` sınıflarının metotlarını uygular. Oyuncular arası düello mantığını, saldırı izinlerini, durum güncellemelerini ve zaman aşımlarını yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPVP` Sınıfı:**
        *   **Yapıcı (`CPVP(dwPID1, dwPID2)`):** İki oyuncu ID'sini alır, küçük olanı `m_players[1]`'e, büyük olanı `m_players[0]`'a atar. İstek yapan oyuncunun (`dwPID1` veya `dwPID2`'ye göre) `bAgree` bayrağını `true` yapar. İki PID'den bir CRC hesaplar (`GetFastHash`).
        *   **`Packet(bDelete)`:** `TPacketGCPVP` paketi oluşturur. Modu `PVP_MODE_NONE` (silme), `PVP_MODE_FIGHT` (dövüş), `PVP_MODE_AGREE` (kabul) veya `PVP_MODE_REVENGE` (intikam) olarak ayarlar. Paketi oyundaki tüm istemcilere gönderir (`DESC_MANAGER::instance().GetClientSet()`).
        *   **`Agree(dwPID)`:** Verilen `dwPID`'li oyuncunun `bAgree` bayrağını `true` yapar. Eğer iki oyuncu da kabul etmişse (`IsFight()` true dönerse) `Packet()` ile dövüş durumunu bildirir.
        *   **`IsFight()`:** İki oyuncunun da `bAgree` bayrağının `true` olup olmadığını kontrol eder.
        *   **`Win(dwPID)`:** Kazanan oyuncuyu (`dwPID`) belirler. Kaybeden oyuncunun `bCanRevenge` bayrağını `true`, `bAgree` bayrağını `false` yapar. Kazanan oyuncunun `bAgree` bayrağını `true` yapar. `m_bRevenge`'i `true` yapar ve `Packet()` ile durumu bildirir.
        *   **`SetVID(dwPID, dwVID)`:** Verilen `dwPID`'ye sahip oyuncunun `dwVID`'sini günceller.
        *   **`SetLastFightTime()`:** Mevcut zamanı (`get_dword_time()`) son dövüş zamanı olarak kaydeder.
    *   **`CPVPManager` Sınıfı:**
        *   **`Insert(pkChr, pkVictim)`:** PvP isteği oluşturur.
            *   Mevcut bir PvP varsa (`Find(kPVP.m_dwCRC)`), istek yapan oyuncunun (`pkChr`) `Agree` metodunu çağırır. Dövüş başlarsa chat mesajları gönderir.
            *   Yoksa yeni bir `CPVP` nesnesi oluşturur, karakterlerin VID'lerini ayarlar, `m_map_pkPVP` ve `m_map_pkPVPSetByID` haritalarına ekler.
            *   `pkPVP->Packet()` ile durumu bildirir ve oyunculara chat/fısıltı ile bilgi mesajları gönderir.
        *   **`ConnectEx(pkChr, bDisconnect)`:** Oyuncu bağlandığında (`bDisconnect=false`) veya bağlantısı kesildiğinde (`bDisconnect=true`), bu oyuncunun dahil olduğu tüm PvP örneklerindeki VID'sini günceller (bağlantı kesilirse VID=0 olur).
        *   **`Dead(pkChr, dwKillerPID)`:** Ölen karakter (`pkChr`) ve onu öldürenin (`dwKillerPID`) bir PvP içinde olup olmadığını kontrol eder.
            *   Eğer `IsFight()` durumu aktifse, `pkPVP->Win(dwKillerPID)` çağrılır ve `true` döner.
            *   Eğer dövüş bitmiş ama üzerinden çok kısa bir süre geçmişse (15 saniye) yine `true` döner (muhtemelen bir tür "kaçış sonrası ölüm" durumu için).
            *   Aksi halde `false` döner.
        *   **`CanAttack(pkChr, pkVictim)`:** İki karakterin birbirine saldırıp saldıramayacağını belirler.
            *   Hedef NPC, ışınlanma kapısı vb. ise `false`.
            *   Aynı karakterlerse `false`.
            *   At üzerinde bazı kısıtlamalar.
            *   Gözlemci modundaysalar `false`.
            *   Harita imparatorluk koruması ve karakterin PK modu `PK_MODE_PROTECT` ise `false`.
            *   Farklı imparatorluktaysalar genellikle `true` (bazı bölgesel PK modu kontrolleriyle).
            *   Aynı partideyseler `false`.
            *   Hedef `IsKillerMode()` ise `true`.
            *   Saldıran negatif, hedef pozitif eğilimdeyse ve `g_protectNormalPlayer` aktif ve hedef `PK_MODE_PEACE` ise `false`.
            *   Saldıranın PK moduna (`PK_MODE_PEACE`, `REVENGE`, `GUILD`, `FREE`) göre çeşitli lonca, eğilim ve `SetKillerMode(true)` kontrolleri yapar.
            *   Son olarak, aktif bir PvP dövüşleri (`pkPVP->IsFight()`) varsa `true`, yoksa `beKillerMode` (PK moduna göre belirlenen) durumunu döndürür.
        *   **`Find(dwCRC)`:** `m_map_pkPVP` haritasından CRC'ye göre `CPVP` nesnesini bulur.
        *   **`Delete(pkPVP)`:** `CPVP` nesnesini `m_map_pkPVP` ve `m_map_pkPVPSetByID` haritalarından siler ve `M2_DELETE` ile belleği serbest bırakır.
        *   **`SendList(d)`:** Belirli bir istemciye (`d`) aktif tüm PvP durumlarını `TPacketGCPVP` paketleri ile gönderir.
        *   **`Process()`:** `m_map_pkPVP`'deki tüm PvP örneklerini dolaşır. `get_dword_time() - pvp->GetLastFightTime()` farkı belirli bir süreyi (600 saniye = 10 dakika) aşmışsa, `pvp->Packet(true)` ile silme paketi gönderir ve `Delete(pvp)` ile PvP'yi siler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `pvp.h`, `crc32.h`, `packet.h`, `desc.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `config.h`, `sectree_manager.h`, `buffer_manager.h`, `locale_service.h`. 

---

### `refine.h`

**Amacı:** Bu başlık dosyası, eşya geliştirme (refine) sistemini yöneten `CRefineManager` sınıfını ve ilgili sabitleri tanımlar. Eşya geliştirme tariflerinin yüklenmesi ve sorgulanması için temel arayüzü sağlar.

**Temel Bileşenler:**

*   **Enum Sabitleri:**
    *   `BLACKSMITH_MOB`: Normal demirci NPC'sinin VNUM'u.
    *   `ALCHEMIST_MOB`: %100 başarı şansıyla geliştirme yapan simyacı NPC'sinin VNUM'u.
    *   `BLACKSMITH_WEAPON_MOB`, `BLACKSMITH_ARMOR_MOB`, `BLACKSMITH_ACCESSORY_MOB`: Sırasıyla silah, zırh ve aksesuar demircilerinin VNUM'ları.
    *   `DEVILTOWER_BLACKSMITH_WEAPON_MOB`, `DEVILTOWER_BLACKSMITH_ARMOR_MOB`, `DEVILTOWER_BLACKSMITH_ACCESSORY_MOB`: Şeytan Kulesi'ndeki özel demircilerin VNUM'ları.
    *   `BLACKSMITH2_MOB`: Başka bir demirci NPC'sinin VNUM'u.
*   **`CRefineManager` Sınıfı:**
    *   **Amacı:** Eşya geliştirme tariflerini yöneten singleton bir sınıftır.
    *   **Typedef'ler:**
        *   `TRefineRecipeMap`: `DWORD` (genellikle eşya VNUM'u veya bir tarif ID'si) anahtarını `TRefineTable` (geliştirme tarifinin detayları) değerine eşleyen bir harita.
    *   **Metotlar:**
        *   `CRefineManager()`: Kurucu metot.
        *   `~CRefineManager()`: Yıkıcı metot.
        *   `Initialize(TRefineTable* table, int size)`: Verilen `TRefineTable` dizisinden geliştirme tariflerini yükler ve `m_map_RefineRecipe` haritasını doldurur.
        *   `GetRefineRecipe(DWORD id)`: Belirtilen ID'ye (genellikle kaynak eşyanın bir sonraki seviyesinin VNUM'u veya özel bir tarif ID'si) sahip geliştirme tarifini döndürür. Tarif bulunamazsa `NULL` döner.
    *   **Özel Üyeler:**
        *   `m_map_RefineRecipe` (`TRefineRecipeMap`): Yüklenen tüm geliştirme tariflerini saklar.

**Bağlantılı Dosyalar:** `constants.h` (muhtemelen `TRefineTable` yapısının tanımını içerir).

---

### `refine.cpp`

**Amacı:** Bu dosya, `refine.h` içinde tanımlanan `CRefineManager` sınıfının metotlarını uygular. Eşya geliştirme tariflerinin yüklenmesi ve sorgulanması işlevlerini yerine getirir.

**Temel İşlevler ve Implementasyon Detayları:**

*   **`CRefineManager::CRefineManager()` ve `CRefineManager::~CRefineManager()`:**
    *   Standart kurucu ve yıkıcı metotlardır. Özel bir başlatma veya temizleme işlemi yapmazlar.
*   **`CRefineManager::Initialize(TRefineTable* table, int size)`:**
    *   Bu metot, sunucu başlangıcında veya bir yeniden yükleme sırasında çağrılarak eşya geliştirme tariflerini yükler.
    *   Öncelikle, mevcut `m_map_RefineRecipe` haritasının boş olup olmadığını kontrol eder; doluysa temizler.
    *   Daha sonra, argüman olarak verilen `TRefineTable` dizisi üzerinde döngüye girer.
    *   Her bir `TRefineTable` öğesi için, tarif ID'sini (`table->id`), başarı olasılığını (`table->prob`) ve maliyetini (`table->cost`) loglar (`sys_log`).
    *   Tarif ID'sini anahtar, `TRefineTable` öğesini de değer olarak kullanarak `m_map_RefineRecipe` haritasına ekler.
    *   Son olarak, yüklenen toplam tarif sayısını loglar.
    *   Başarılı olursa `true` döndürür.
*   **`CRefineManager::GetRefineRecipe(DWORD vnum)`:**
    *   Belirtilen `vnum` (genellikle bir sonraki geliştirme seviyesinin eşya kodu veya özel bir tarif ID'si) için bir geliştirme tarifi arar.
    *   `__SOUL_SYSTEM__` makrosu tanımlı değilse ve `vnum` 0 ise doğrudan `NULL` döndürür (ruh sistemiyle ilgili bir optimizasyon veya özel durum olabilir).
    *   `m_map_RefineRecipe` haritasında `vnum`'u arar.
    *   Arama sonucunu loglar (`sys_log`), tarifin bulunup bulunmadığını belirtir.
    *   Eğer tarif bulunamazsa (`it == m_map_RefineRecipe.end()`), `NULL` döndürür.
    *   Tarif bulunursa, `TRefineTable` yapısına bir işaretçi (`&it->second`) döndürür.

**Bağlantılı Dosyalar:** `stdafx.h`, `refine.h`. 

---

### `ShipDefense.h`

**Amacı:** Bu başlık dosyası, Metin2 sunucusundaki "Gemi Savunması" (Ship Defense) adlı oyun özelliğini yönetmek için gerekli olan sabitleri, enum yapılarını ve `CShipDefense` ile `CShipDefenseManager` sınıflarını tanımlar. Bu özellik, oyuncuların belirli bir haritada dalgalar halinde gelen canavarlara karşı bir gemiyi veya önemli bir noktayı (direk) koruduğu bir zindan/etkinlik türüdür.

**Temel Bileşenler:**

*   **`ShipDefense` Namespace:**
    *   **Enum Sabitleri:**
        *   `EInstance`: Etkinliğin genel kurallarını belirler (örn: parti gerekliliği, bilet gerekliliği, tamir odunu düşmesi, bekleme süresi).
        *   `EProbability`: Olasılıkla ilgili sabitler (örn: min/max olasılık, odun tamir yüzdesi).
        *   `EVNumHelper`: Etkinlikte kullanılan özel NPC, canavar ve eşya VNUM'ları (Balıkçı, Gemi Direği, Hidra türleri, Portal, Tamir Odunu vb.).
        *   `EMapIndex`: Etkinlikle ilgili harita indeksleri (Giriş Limanı, Gemi Haritası, Çıkış Limanı).
        *   `EClearType`: Canavarları temizleme türleri (Tümünü, Boss hariç tümünü, Yumurtaları, Sahte Boss'u temizle).
        *   `EWaves`: Etkinlikteki dalga (wave) numaralandırması.
        *   `EUniqueCharPos`: Haritadaki özel/önemli karakterlerin (Direk, Hidra, Mini Hidralar, Yumurta) pozisyonlarını işaretlemek için kullanılır.
        *   `EWaveDuration`: Çeşitli zamanlayıcı süreleri (sonraki dalga gecikmesi, ilk dalga süresi, çıkış gecikmesi, yumurta bekleme süresi, lazer sığınak süresi vb.).
        *   `EBarrierWall`: Gemi üzerindeki bariyerlerin pozisyonları.
        *   `ENoticeType`: Oyunculara gönderilecek farklı duyuru türleri (dalga başlangıcı, direk HP'si, görev başarısı/başarısızlığı vb.).
        *   `EJumpTo`: Oyuncuların ışınlanabileceği yerler (Başlangıç noktası, Çıkış Limanı).
*   **`CShipDefense` Sınıfı:**
    *   **Amacı:** Tek bir Gemi Savunması etkinliği örneğini yönetir. Oyuncuların bulunduğu özel haritayı, dalga durumunu, canavar spawnlarını, zamanlayıcıları ve etkinliğe özgü diğer mekanikleri kontrol eder.
    *   **Typedef'ler:**
        *   `UNIQUE_CHAR_POSITION`: Özel karakter pozisyonları için `BYTE`.
        *   `UniqueCharacterMap`: `UNIQUE_CHAR_POSITION`'ı `LPCHARACTER`'a eşler.
        *   `LaserEffectData`: Lazer efektinin koordinatlarını tutan yapı.
        *   `LaserEffectDataMap`: `BYTE` (pozisyon) ile `LaserEffectData`'yı eşler.
    *   **Metotlar (Önemlileri):**
        *   `Destroy()`: Etkinliği sonlandırır, haritayı yok eder, olayları iptal eder.
        *   `CancelEvents()`: Etkinlikle ilgili tüm zamanlayıcı olaylarını iptal eder.
        *   `DeadCharacter(LPCHARACTER c_lpChar)`: Bir karakter öldüğünde çağrılır. Eğer ölen direk ise etkinliği sonlandırır. Diğer özel canavarların ölümünü işler.
        *   `NoticeByType(ENoticeType c_eType)`: Belirli bir duyuru türünü haritadaki tüm oyunculara gönderir.
        *   `Spawn(DWORD dwVNum, ...)`: Belirtilen VNUM'a sahip bir canavarı/NPC'yi haritada belirlenen pozisyonda spawn eder.
        *   `SpawnRegen(const char* c_szFileName, ...)`: Belirtilen regen dosyasını yükleyerek canavarları spawn eder.
        *   `SpawnHydra()`, `SpawnMiniHydra()`, `SpawnEgg()`: Özel boss ve canavarları spawn eder.
        *   `SpawnBarriers()`, `RemoveBarriers()`: Gemi etrafındaki bariyerleri oluşturur veya kaldırır.
        *   `Start()`: Etkinliği başlatır, ilk dalgayı hazırlar.
        *   `PrepareWave(BYTE c_byWave)`: Belirtilen dalgayı başlatmadan önce hazırlık yapar (duyuru, bekleme süresi).
        *   `SetWave(BYTE c_byWave)`: Asıl dalgayı başlatır, canavarları spawn eder, ilgili olayları (spawn, lazer) ayarlar.
        *   `SetShipEvent(BYTE c_byWave)`: Dalga tamamlandığında veya etkinlik bittiğinde bir sonraki aşamaya geçişi yönetir (ödül, çıkış portalı vb.).
        *   `JumpAll(EJumpTo eJumpTo)`: Haritadaki tüm oyuncuları belirtilen yere ışınlar ve etkinliği sonlandırır.
        *   `GetUniqueCharacter(BYTE c_byUniqueID)`: Belirli bir pozisyondaki özel karakteri (Direk, Hidra vb.) döndürür.
    *   **Üyeler:** Zamanlayıcılar (`LPEVENT`), harita indeksi, lider PID'si, durum, dalga numarası, özel karakterler haritası, lazer efekti verileri.
*   **`CShipDefenseManager` Sınıfı (Singleton):**
    *   **Amacı:** Tüm aktif Gemi Savunması etkinliklerini yönetir. Yeni etkinlikler oluşturur, mevcutları takip eder, oyuncuların katılımını ve ayrılmasını yönetir.
    *   **Enum `EStates`:** Etkinliğin durumları (NONE, CREATE, START, STOP).
    *   **Metotlar (Önemlileri):**
        *   `Create(LPCHARACTER c_lpChar)`: Yeni bir Gemi Savunması etkinliği oluşturur, özel haritayı yaratır, oyuncuyu ışınlar.
        *   `Start(LPCHARACTER c_lpChar)`: Oluşturulmuş bir etkinliği başlatır.
        *   `Stop(LPCHARACTER c_lpChar)`: Bir etkinliği zorla durdurur, oyuncuları dışarı atar.
        *   `Remove(DWORD c_dwLeaderPID)`: Belirli bir etkinliği listeden kaldırır ve yok eder.
        *   `GetState(DWORD c_dwLeaderPID)`: Bir etkinliğin durumunu döndürür.
        *   `IsCreated()`, `IsRunning()`, `CanJoin()`: Oyuncunun etkinlikle ilgili durumunu kontrol eder.
        *   `GetLeaderPID(LPCHARACTER c_lpChar, ...)`: Parti liderinin PID'sini veya oyuncunun PID'sini alır.
        *   `BroadcastAllianceHP()`: Gemi direğinin HP'sini oyunculara gönderir.
        *   `SetAllianceHPPct()`: Tamir odunu kullanıldığında direğin HP'sini artırır.
        *   `Join()`, `Leave()`, `Land()`: Oyuncuların etkinliğe katılması, ayrılması veya etkinlik sonrası karaya çıkmasını yönetir.
        *   `CanAttack()`, `OnKill()`: Etkinlik sırasındaki saldırı ve ölüm olaylarını işleyen trigger fonksiyonları.
        *   VNUM Kontrol Fonksiyonları (`IsHydra`, `IsMast` vb.): Belirli VNUM'ların etkinlikle ilgili özel canavar/nesne olup olmadığını kontrol eder.
        *   `IsDungeon(long c_lMapIndex)`: Verilen harita indeksinin bir Gemi Savunması haritası olup olmadığını kontrol eder.
    *   **Üyeler:** `m_mapShipDefense` (Lider PID'sini `CShipDefense*` işaretçisine eşleyen harita).

**Bağlantılı Dosyalar:** `stdafx.h` (temel başlıklar).

---

### `ShipDefense.cpp`

**Amacı:** Bu dosya, `ShipDefense.h` içinde tanımlanan `CShipDefense` ve `CShipDefenseManager` sınıflarının metotlarını uygular. "Gemi Savunması" (Ship Defense) etkinliğinin tüm mantığını, dalga yönetimini, canavar spawnlarını, oyuncu etkileşimlerini ve olay zamanlayıcılarını yönetir.

**Orta Seviye Implementasyon Detayları:**

*   **Olay (Event) Fonksiyonları:**
    *   `ClearSpawnEvent`: Belirli bir süre sonra bazı canavar türlerini (örn: Sahte Hidra, Yumurtalar) temizler.
    *   `ExitEvent`: Etkinlik bittiğinde veya direk yıkıldığında oyuncuları belirli bir süre sonra haritadan çıkarmak için geri sayım yapar ve duyuru gönderir.
    *   `ShipEvent`: Bir dalga tamamlandıktan sonra bir sonraki dalgaya geçiş için kısa bir bekleme süresi uygular.
    *   `LaserEffectEvent`: Belirli dalgalarda Mini Hidraların lazer saldırısı yapma olasılığını ve pozisyonunu yönetir. Oyuncuların lazerli alanda olup olmadığını kontrol eder.
    *   `WaveEvent`: Bir dalganın başlamasından önceki geri sayımı yönetir, oyunculara duyuru yapar ve dalga süresini kontrol eder.
    *   `SpawnEvent`: Aktif dalga boyunca periyodik olarak veya belirli koşullara göre (örn: güvertedeki canavar sayısı azaldığında) canavarları spawn eder. Farklı dalgalar için farklı spawn mantıkları içerir (örn: Yumurta, Mini Hidra spawn olasılıkları).
*   **`CShipDefense` Sınıfı Metotları:**
    *   **Yapıcı/Yıkıcı:** Etkinlik için gerekli başlangıç ayarlarını yapar (harita indeksi, lider PID, durum, dalga, özel karakterler haritasını temizleme vb.). Yıkıcı, `Destroy()`'u çağırır.
    *   `Destroy()`: Etkinlikle ilişkili özel haritayı `SECTREE_MANAGER` aracılığıyla yok eder, olayları iptal eder, üyeleri sıfırlar.
    *   `CancelEvents()`: Tüm aktif olay zamanlayıcılarını (`m_lpShipEvent`, `m_lpWaveEvent` vb.) `event_cancel` ile iptal eder.
    *   `DeadCharacter(c_lpChar)`: Ölen karakterin türüne göre işlem yapar:
        *   Eğer ölen Gemi Direği (`UNIQUE_MAST_POS`) ise: Tüm olayları iptal eder, güvertedeki tüm canavarları temizler, oyunculara direğin yıkıldığına dair duyuru yapar ve belirli bir süre sonra oyuncuları ana karaya ışınlamak için `ExitEvent` başlatır.
        *   Diğer özel karakterler (Mini Hidralar, Yumurta) ölürse `m_mapUniqueCharacter`'dan kaldırılır.
    *   `Notice(c_lpChar, ...)` ve `NoticeByType(c_eType)`: Oyunculara çeşitli formatlarda (büyük/küçük font, yerelleştirilmiş metinler) duyurular gönderir.
    *   `FindAllyCharacter()`: Haritadaki tüm canavarların hedefini Gemi Direği olarak ayarlar.
    *   `ClearMonstersByType(eClearType)`: Belirtilen türe göre haritadaki canavarları temizler (örn: tümü, boss hariç, yumurtalar).
    *   `CheckLaserPosition()`: Lazer saldırısı aktif olan Mini Hidraların pozisyonunda oyuncu olup olmadığını kontrol eder. Eğer oyuncu sığınaktaysa (lazerin çıktığı Mini Hidra'nın yanındaysa ve kısa süre önce başka bir sığınağa girmemişse) lazer efektini kaldırır.
    *   `JumpToPosition(c_lMapIndex, c_lXPos, c_lYPos)`: Haritadaki tüm oyuncuları belirtilen konuma ışınlar.
    *   `Spawn(dwVNum, ...)`: Belirtilen VNUM'lu canavarı/NPC'yi oluşturur, pozisyonunu ayarlar ve haritada gösterir.
    *   `CheckEmptyDeck()`: Gemi güvertesinde (belirli boss/özel canavarlar hariç) canavar kalıp kalmadığını kontrol eder.
    *   `SpawnRegen(c_szFileName, bOnce)`: `regen_do` fonksiyonunu kullanarak belirtilen dosyadan canavar gruplarını spawn eder.
    *   `SpawnHydra()`, `SpawnMiniHydra()`, `SpawnEgg()`: İlgili boss/özel canavarları haritadaki belirli pozisyonlarda veya rastgele pozisyonlarda spawn eder ve `m_mapUniqueCharacter` haritasına ekler. Yumurta spawn olduğunda belirli bir süre sonra kendini temizlemesi için `ClearSpawnEvent` başlatır.
    *   `SpawnBarriers()`, `RemoveBarriers()`, `RemoveFrontBarriers()`, `RemoveBackBarriers()`: Gemi etrafındaki görünmez duvarları (bariyer NPC'leri) oluşturur veya kaldırır.
    *   `Start()`: Etkinliği resmen başlatır, zamanı kaydeder, durumu `STATE_START` yapar, direk HP'sini yayınlar ve ilk dalgayı hazırlar (`PrepareWave`).
    *   `PrepareDeck()`: Etkinlik oluşturulduğunda çağrılır. Gemi Direği, Dümen gibi sabit NPC'leri ve bariyerleri spawn eder.
    *   `PrepareWave(c_byWave)`: Bir sonraki dalgaya geçmeden önce çağrılır. Önceki olayları iptal eder, oyunculara duyuru gönderir, (varsa) sahte boss'ları spawn eder ve belirli bir bekleme süresi sonunda `SetWave`'i çağırmak için `WaveEvent` başlatır.
    *   `SetWave(c_byWave)`: Asıl dalgayı başlatır. Gerekli canavarları (`SpawnRegen`, `SpawnMiniHydra` vb.) spawn eder, dalga boyunca canavar spawn etmek için `SpawnEvent` başlatır ve (gerekirse) lazer saldırıları için `LaserEffectEvent` başlatır.
    *   `SetShipEvent(c_byWave)`: Bir Hidra boss'u yenildiğinde veya tüm dalgalar tamamlandığında çağrılır. Oyunculara duyuru yapar, bir sonraki dalga için `PrepareWave`'i çağırmak üzere kısa bir `ShipEvent` başlatır veya etkinlik bittiyse ödül ve çıkış portalını spawn eder, `ExitEvent` başlatır.
    *   `JumpToQuarterDeck()`: Oyuncuları geminin belirli bir başlangıç noktasına ışınlar ve bariyerleri yeniden spawn eder.
    *   `JumpAll(eJumpTo)`: Tüm oyuncuları belirtilen varış noktasına (ana kara veya çıkış limanı) ışınlar ve `CShipDefenseManager` aracılığıyla bu etkinliği sonlandırır.
    *   `GetUniqueCharacter()`, `GetAllianceCharacter()`, `GetHydraCharacter()`, `IsUniqueMiniHydra()`: Özel karakterlere erişim ve kontrol fonksiyonları.
    *   `SetLaserEffectData()`, `GetLaserEffectDataMap()`: Lazer saldırısı bilgilerini yönetir.
*   **`CShipDefenseManager` Sınıfı Metotları:**
    *   **Yapıcı/Yıkıcı/Initialize/Destroy:** Etkinlik haritasını (`m_mapShipDefense`) yönetir.
    *   `Create(c_lpChar)`: Yeni bir Gemi Savunması etkinliği başlatır.
        *   Oyuncunun zaten bir etkinlikte olup olmadığını kontrol eder.
        *   `SECTREE_MANAGER::instance().CreatePrivateMap()` ile Gemi Savunması için özel bir harita oluşturur.
        *   Yeni bir `CShipDefense` nesnesi oluşturur ve `m_mapShipDefense`'e ekler.
        *   Parti üyelerine duyuru yapar ve (gerekirse) bekleme süresi uygular.
        *   Lideri yeni oluşturulan özel haritaya ışınlar.
    *   `Start(c_lpChar)`: Lider tarafından çağrıldığında, ilgili `CShipDefense` nesnesinin `Start()` metodunu çağırarak etkinliği başlatır.
    *   `Stop(c_lpChar)`: Etkinliği erken sonlandırır, tüm oyuncuları ana karaya ışınlar.
    *   `Remove(c_dwLeaderPID)`: Bir etkinliği ve ilişkili `CShipDefense` nesnesini siler.
    *   `GetState()`, `IsCreated()`, `IsRunning()`, `CanJoin()`: Etkinliklerin ve oyuncuların durumlarını kontrol eder.
    *   `GetLeaderPID(c_lpChar, c_bIsLeader)`: Verilen karakterin parti liderinin PID'sini veya kendi PID'sini (partisi yoksa veya kendisi liderse) döndürür.
    *   `BroadcastAllianceHP(c_lpAllianceChar, c_lpSectreeMap)`: Gemi Direği'nin canını `HEADER_GC_TARGET` paketi ile haritadaki tüm oyunculara gönderir.
    *   `SetAllianceHPPct(c_lpRepairChar, c_byPct)`: Bir oyuncu tamir odunu kullandığında çağrılır. Gemi Direği'nin HP'sini artırır, oyuncuya duyuru yapar, oyuncuyu sersemletir ve güncel HP'yi yayınlar.
    *   `Join(c_lpChar)`, `Leave(c_lpChar)`, `Land(c_lpChar)`: Oyuncuların etkinliğe katılması, etkinlikten ayrılması (ana karaya) veya etkinlik başarıyla bittiğinde çıkış limanına ışınlanmasını yönetir.
    *   `CanAttack(lpCharAttacker, lpCharVictim)`: Gemi Savunması sırasında kimin kime saldırabileceğini belirler (örn: canavarlar direğe saldırabilir, sahte Hidra'ya saldırılamaz).
    *   `OnKill(lpDeadChar, lpKillerChar)`: Bir karakter öldüğünde tetiklenir:
        *   Eğer ölen karakter bir Hidra Yumurtası ise, güvertedeki boss dışındaki tüm canavarları temizler.
        *   Eğer `SPAWN_WOOD_REPAIR` aktifse ve ölen bir Mini Hidra ise, öldüğü yere Tamir Odunu düşürür.
        *   Eğer ölen ana Hidra boss'larından biriyse, bir sonraki dalgaya geçmek için `pShipDefense->SetShipEvent()` çağrılır.
        *   Genel olarak, ölen karakterin `pShipDefense->DeadCharacter()` metodunu çağırır.
    *   VNUM Kontrol Fonksiyonları (`IsHydra`, `IsMast` vb.): Belirli VNUM'ların Gemi Savunması ile ilgili olup olmadığını kontrol eder.
    *   `IsDungeon(c_lMapIndex)`: Bir harita indeksinin Gemi Savunması için oluşturulmuş özel bir harita olup olmadığını kontrol eder.

**Bağlantılı Dosyalar:** `stdafx.h`, `ShipDefense.h`, `sectree_manager.h`, `char_manager.h`, `char.h`, `desc.h`, `party.h`, `mob_manager.h`, `regen.h`, `config.h`.

--- 