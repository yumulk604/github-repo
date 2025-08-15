# Metin2 Oyun Sunucusu - Özellikler Referansı Part 7 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans_Part6.md`](game_Features_Referans_Part6.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) çeşitli özellikleri ve sistemleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`wedding.h`](#weddingh)
*   [`wedding.cpp`](#weddingcpp)
*   [`xmas_event.h`](#xmas_eventh)
*   [`xmas_event.cpp`](#xmas_eventcpp)

---

### `wedding.h`

*   **Amaç:** Oyuncuların evlenmeleri için özel olarak oluşturulan düğün haritalarını (`WeddingMap`) ve bu haritaları yöneten `WeddingManager` singleton sınıfını tanımlar. Bu sistem, düğün haritasının oluşturulması, katılımcıların (davetlilerin) yönetimi, harita içi özel efektlerin (hava durumu, müzik) kontrolü ve törenin sonlandırılması gibi işlevleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`marriage` Namespace'i:** Düğünle ilgili tüm sınıfları ve sabitleri içerir.
    *   **`WEDDING_MAP_INDEX` Sabiti:** Düğün haritasının temel harita indeksini (81) tanımlar.
    *   **`charset_t` Typedef'i:** `CHARACTER_SET` için bir takma ad.
    *   **`WeddingMap` Sınıfı:**
        *   Belirli bir evlilik töreni için oluşturulmuş özel bir düğün haritası örneğini yönetir.
        *   **Yapıcı (`WeddingMap(dwMapIndex, dwPID1, dwPID2)`):** Harita indeksini ve evlenen çiftin oyuncu ID'lerini (`dwPID1`, `dwPID2`) alır.
        *   **Yıkıcı (`~WeddingMap()`):** Harita sonlandırma olayını (`m_pEndEvent`) iptal eder.
        *   **Metotlar:**
            *   `GetMapIndex()`: Haritanın indeksini döndürür.
            *   `WarpAll()`: Haritadaki tüm oyuncuları kayıtlı geri dönüş konumlarına ışınlar.
            *   `DestroyAll()`: Haritadaki tüm oyuncuların bağlantısını keser veya karakterlerini yok eder.
            *   `Notice(const char* psz)`: Haritadaki tüm oyunculara duyuru mesajı gönderir.
            *   `SetEnded()`: Düğün töreninin bittiğini işaretler, oyunculara duyuru yapar, belirli seviyedeki katılımcılara ödül eşyası verir ve haritanın kapanması için bir olay (`wedding_end_event`) başlatır.
            *   `IncMember(LPCHARACTER ch)`, `DecMember(LPCHARACTER ch)`: Oyuncuları haritaya (davetli listesine) ekler/çıkarır. Düşük seviyeli oyuncuları gözlemci moduna alır.
            *   `IsMember(LPCHARACTER ch)`: Bir karakterin haritada olup olmadığını kontrol eder.
            *   `SetDark(bool bSet)`: Haritanın hava durumunu karanlık/aydınlık olarak ayarlar ve oyunculara bildirir.
            *   `SetSnow(bool bSet)`: Haritada kar efektini açar/kapatır ve oyunculara bildirir.
            *   `SetMusic(bool bSet, const char* szMusicFileName)`: Haritada özel bir müzik çalar/durdurur ve oyunculara bildirir.
            *   `IsPlayingMusic()`: Haritada müzik çalınıp çalınmadığını döndürür.
            *   `SendLocalEvent(LPCHARACTER ch)`: Yeni katılan bir oyuncuya haritanın mevcut durumunu (hava durumu, müzik) gönderir.
            *   `ShoutInMap(BYTE type, const char* szMsg)`: Haritadaki tüm oyunculara belirli bir türde sohbet mesajı gönderir.
        *   **Özel Metot:** `__BuildCommandPlayMusic(...)`: Müzik çalma komut string'ini oluşturur.
        *   **Özel Üyeler:** `m_dwMapIndex`, `m_pEndEvent` (LPEVENT), `m_set_pkChr` (davetli karakterler), `m_isDark`, `m_isSnow`, `m_isMusic`, `dwPID1`, `dwPID2` (evlenen çift), `m_stMusicFileName`.
    *   **`WeddingMapMap` Typedef'i:** `std::map<DWORD, WeddingMap*>` için bir takma ad. Aktif düğün haritalarını (harita indeksi -> `WeddingMap*`) tutar.
    *   **`WeddingManager` Sınıfı (Singleton):**
        *   Tüm aktif düğün haritalarını yönetir.
        *   **Metotlar:**
            *   `IsWeddingMap(DWORD dwMapIndex)`: Verilen harita indeksinin bir düğün haritası olup olmadığını kontrol eder.
            *   `Request(DWORD dwPID1, DWORD dwPID2)`: Evlenmek isteyen iki oyuncu için yeni bir düğün haritası oluşturma isteği başlatır. `__CreateWeddingMap` ile haritayı oluşturur ve DB'ye `HEADER_GD_WEDDING_READY` paketi gönderir.
            *   `End(DWORD dwMapIndex)`: Belirli bir düğün haritasındaki töreni sonlandırır (`WeddingMap::SetEnded` çağırır).
            *   `DestroyWeddingMap(WeddingMap* pMap)`: Bir `WeddingMap` örneğini ve ilişkili özel haritayı yok eder.
            *   `Find(DWORD dwMapIndex)`: Verilen harita indeksine sahip aktif bir `WeddingMap` bulur.
        *   **Özel Metot:** `__CreateWeddingMap(DWORD dwPID1, DWORD dwPID2)`: `SECTREE_MANAGER::CreatePrivateMap` kullanarak düğün haritasının özel bir kopyasını oluşturur, `WeddingMap` nesnesini yaratır, haritaya NPC'leri (`regen_do` ile `metin2_map_wedding_01/npc.txt` dosyasından) yerleştirir ve harita indeksini döndürür.
        *   **Özel Üyeler:** `m_mapWedding` (`WeddingMapMap`).
*   **Bağlantılı Dosyalar:** `wedding.cpp` (uygulama), `marriage.h` (muhtemelen evlilikle ilgili temel yapılar), `stdafx.h`, `char.h`, `event.h`.

---

### `wedding.cpp`

*   **Amaç:** `wedding.h` içinde bildirilen `WeddingMap` ve `WeddingManager` sınıflarının metotlarını uygular. Düğün haritalarının oluşturulması, oyuncu yönetimi, harita içi olayların kontrolü (müzik, hava durumu) ve törenin sonlandırılması gibi işlevleri yerine getirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Olay Fonksiyonu (`wedding_end_event`):**
        *   `wedding_map_info` yapısını kullanarak ilgili `WeddingMap` örneğine erişir.
        *   İlk adımda (`info->iStep == 0`) haritadaki tüm oyuncuları ışınlar (`pMap->WarpAll()`) ve 15 saniye sonra tekrar çalışacak şekilde ayarlanır.
        *   İkinci adımda düğün haritasını yok eder (`WeddingManager::instance().DestroyWeddingMap(pMap)`).
    *   **`WeddingMap::WeddingMap(...)` (Yapıcı):**
        *   Harita indeksini, evlenen çiftin PID'lerini saklar, olayları ve bayrakları (dark, snow, music) sıfırlar.
    *   **`WeddingMap::~WeddingMap()` (Yıkıcı):**
        *   `m_pEndEvent`'i iptal eder.
    *   **`WeddingMap::SetEnded()`:**
        *   Eğer zaten bir bitiş olayı varsa hata loglar.
        *   Yeni bir `wedding_end_event` oluşturur.
        *   Haritadaki oyunculara törenin bittiğini ve ışınlanacaklarını duyurur.
        *   10. seviye ve üzeri (evlenen çift hariç) katılımcılara hediye eşya (VNUM 27002, 5 adet) verir (`ch->AutoGiveItem`).
    *   **`WeddingMap::Notice(psz)`:**
        *   Haritadaki tüm oyunculara (`m_set_pkChr`) `FNotice` functor'ını kullanarak duyuru mesajı gönderir.
    *   **`WeddingMap::WarpAll()`:**
        *   Haritadaki tüm PC'leri (`m_set_pkChr`) `FWarpEveryone` functor'ını kullanarak kayıtlı konumlarına ışınlar (`ch->ExitToSavedLocation()`).
    *   **`WeddingMap::DestroyAll()`:**
        *   Haritadaki tüm karakterleri (`m_set_pkChr`) `FDestroyEveryone` functor'ını kullanarak oyundan atar (eğer bağlantısı varsa `DESC_MANAGER::instance().DestroyDesc`, yoksa `M2_DESTROY_CHARACTER`). Karakterleri setten silerek döngü yapar.
    *   **`WeddingMap::IncMember(ch)`, `WeddingMap::DecMember(ch)`:**
        *   Karakteri `m_set_pkChr` setine ekler/çıkarır.
        *   `IncMember`'da, oyuncuya haritanın mevcut durumunu (hava, müzik) gönderir (`SendLocalEvent`). Seviyesi 10'dan küçükse gözlemci moduna alır.
        *   `DecMember`'da, seviyesi 10'dan küçükse gözlemci modundan çıkarır.
    *   **Efekt Ayarlama Metotları (`SetMusic`, `SetDark`, `SetSnow`):**
        *   İlgili bayrağı (`m_isMusic`, `m_isDark`, `m_isSnow`) günceller.
        *   Haritadaki tüm oyunculara (`ShoutInMap`) ilgili komut paketini gönderir ("PlayMusic ...", "DayMode dark/light", "xmas_snow 1/0").
    *   **`WeddingMap::SendLocalEvent(ch)`:**
        *   Karaktere, haritanın mevcut karanlık, kar ve müzik durumunu bildiren komut paketlerini gönderir.
    *   **`WeddingManager::__CreateWeddingMap(dwPID1, dwPID2)`:**
        *   `SECTREE_MANAGER::instance().CreatePrivateMap(WEDDING_MAP_INDEX)` ile düğün haritasının özel bir kopyasını oluşturur.
        *   Yeni bir `WeddingMap` nesnesi oluşturur ve `m_mapWedding`'e ekler.
        *   Düğün haritası için NPC'leri (`metin2_map_wedding_01/npc.txt` dosyasından) `regen_do` ile haritaya yerleştirir.
    *   **`WeddingManager::DestroyWeddingMap(pMap)`:**
        *   `pMap->DestroyAll()` ile haritadaki tüm karakterleri temizler.
        *   `m_mapWedding`'den haritayı siler.
        *   `SECTREE_MANAGER::instance().DestroyPrivateMap()` ile özel haritayı yok eder.
        *   `WeddingMap` nesnesini siler.
    *   **`WeddingManager::Request(dwPID1, dwPID2)`:**
        *   `map_allow_find(WEDDING_MAP_INDEX)` ile düğün haritasının mevcut olup olmadığını kontrol eder.
        *   `__CreateWeddingMap` ile yeni bir düğün haritası oluşturur.
        *   Veritabanına (`db_clientdesc`) `HEADER_GD_WEDDING_READY` paketi göndererek evliliğin hazır olduğunu bildirir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `desc_client.h`, `desc_manager.h`, `char_manager.h`, `sectree_manager.h`, `config.h`, `char.h`, `wedding.h`, `regen.h`, `locale_service.h`.

---

### `xmas_event.h`

*   **Amaç:** Yılbaşı (Noel) etkinliğiyle ilgili fonksiyonları ve sabitleri (NPC VNUM'ları) tanımlar. Bu, etkinlik bayraklarına (quest flag) göre çeşitli oyun içi değişiklikleri (kar yağışı, müzik, özel NPC'lerin doğması/kalkması) tetiklemeyi içerir.
*   **Temel İşlevler/İçerik:**
    *   **`xmas` Namespace'i:** Yılbaşı etkinliğiyle ilgili tüm bildirimleri içerir.
    *   **NPC VNUM Sabitleri:**
        *   `MOB_SANTA_VNUM = 20031`: Noel Baba NPC'sinin VNUM'u.
        *   `MOB_XMAS_TREE_VNUM = 20032`: Yılbaşı Ağacı NPC'sinin VNUM'u.
        *   `MOB_XMAS_FIRWORK_SELLER_VNUM = 9004`: Havai Fişek Satıcısı NPC'sinin VNUM'u.
    *   **Fonksiyon Bildirimleri:**
        *   `ProcessEventFlag(const std::string& name, int prev_value, int value)`: Bir etkinlik bayrağının (`name`) değeri değiştiğinde (`prev_value` -> `value`) çağrılır. Bayrak adına göre ilgili işlemleri tetikler.
        *   `SpawnSanta(long lMapIndex, int iTimeGapSec)`: Belirli bir haritada (`lMapIndex`) Noel Baba'nın belirli aralıklarla (`iTimeGapSec`) yeniden doğması için bir olay (event) oluşturur.
        *   `SpawnEventHelper(bool spawn)`: `true` ise Havai Fişek Satıcısı NPC'lerini belirli konumlarda doğurur, `false` ise tüm Havai Fişek Satıcılarını haritalardan kaldırır.
*   **Bağlantılı Dosyalar:** `xmas_event.cpp` (uygulama), `stdafx.h`.

### `xmas_event.cpp`

*   **Amaç:** `xmas_event.h` içinde bildirilen Yılbaşı etkinliği fonksiyonlarını uygular. Etkinlik bayraklarındaki değişikliklere yanıt verir, Noel Baba ve diğer etkinlik NPC'lerinin doğmasını/kalkmasını yönetir, istemcilere hava durumu gibi komutlar gönderir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`ProcessEventFlag(name, prev_value, value)`:**
        *   `name` (bayrak adı) ve `value` (yeni değer) parametrelerine göre farklı işlemler yapar:
            *   **`xmas_snow`, `xmas_boom`, `xmas_song`, `xmas_tree` (ağaç VNUM'u değil, genel bir ağaç etkinliği bayrağı olabilir):**
                *   Oyundaki tüm istemcilere (`DESC_MANAGER::instance().GetClientSet()`) `CHAT_TYPE_COMMAND` ile `"[bayrak_adı] [yeni_değer]"` şeklinde bir komut gönderir. Bu, istemcide kar yağışı, havai fişek efekti, müzik veya yılbaşı ağacıyla ilgili görsel/işitsel bir değişikliği tetikler.
                *   Eğer `name == "xmas_boom"` ise:
                    *   `value` true ve `prev_value` false ise (`etkinlik başladı`): `SpawnEventHelper(true)` çağırarak Havai Fişek Satıcılarını doğurur.
                    *   `value` false ve `prev_value` true ise (`etkinlik bitti`): `SpawnEventHelper(false)` çağırarak Havai Fişek Satıcılarını kaldırır.
                *   Eğer `name == "xmas_tree"` ise:
                    *   `value > 0` ve `prev_value == 0` ise (`ağaç etkinliği başladı`): Harita 61'de (genellikle ilk köy) belirli koordinatlarda `MOB_XMAS_TREE_VNUM` VNUM'lu Yılbaşı Ağacını doğurur (eğer zaten yoksa).
                    *   `prev_value > 0` ve `value == 0` ise (`ağaç etkinliği bitti`): Tüm `MOB_XMAS_TREE_VNUM` VNUM'lu Yılbaşı Ağaçlarını haritalardan kaldırır.
            *   **`xmas_santa`:**
                *   `value == 0`: Tüm `MOB_SANTA_VNUM` VNUM'lu Noel Baba NPC'lerini haritalardan kaldırır.
                *   `value == 1`: Eğer harita 61 erişilebilir durumdaysa, `xmas_santa` bayrağını `2` yapar ve eğer haritada Noel Baba yoksa, harita 61'de rastgele bir pozisyonda bir Noel Baba doğurur (`SpawnMobRandomPosition`). (Bu, `SpawnSanta` fonksiyonuyla periyodik doğma ayarlanmadan önce manuel bir doğurma gibi görünüyor.)
                *   `value == 2`: Bir şey yapmaz (muhtemelen Noel Baba'nın aktif olduğunu belirten bir durum).
    *   **`spawn_santa_event` (EVENTFUNC):**
        *   `spawn_santa_info` yapısından harita indeksini alır.
        *   `xmas_santa` quest flag'i 0 ise (etkinlik kapalıysa) veya haritada zaten bir Noel Baba varsa bir şey yapmaz.
        *   Belirtilen haritada (`lMapIndex`) rastgele bir pozisyonda `MOB_SANTA_VNUM` VNUM'lu Noel Baba'yı doğurur. Başarısız olursa 5 saniye sonra tekrar dener.
    *   **`SpawnSanta(lMapIndex, iTimeGapSec)`:**
        *   `spawn_santa_info` oluşturur ve harita indeksini ayarlar.
        *   `spawn_santa_event` olayını `iTimeGapSec` saniye aralıklarla çalışacak şekilde oluşturur. Test sunucusunda bu süre 60'a bölünür.
    *   **`SpawnEventHelper(spawn)`:**
        *   `spawn == true` ise:
            *   Önceden tanımlanmış bir pozisyon listesi (`positions`) üzerinden geçer.
            *   Her pozisyon için, eğer ilgili harita erişilebilir durumdaysa (`map_allow_find`), haritanın temel pozisyonunu alır ve belirtilen göreceli koordinatlarda `MOB_XMAS_FIRWORK_SELLER_VNUM` VNUM'lu Havai Fişek Satıcısını doğurur.
        *   `spawn == false` ise:
            *   Tüm `MOB_XMAS_FIRWORK_SELLER_VNUM` VNUM'lu Havai Fişek Satıcılarını haritalardan kaldırır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `xmas_event.h`, `desc.h`, `desc_manager.h`, `sectree_manager.h`, `char.h`, `char_manager.h`, `questmanager.h`. 