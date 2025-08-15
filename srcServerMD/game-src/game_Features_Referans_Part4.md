# Metin2 Oyun Sunucusu - Özellikler Referansı Part 4 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans_Part3.md`](game_Features_Referans_Part3.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) belirli oyun özellikleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`OXEvent.h`](#oxeventh)
*   [`OXEvent.cpp`](#oxeventcpp)
*   [`party.h`](#partyh)
*   [`party.cpp`](#partycpp)
*   [`pcbang.h`](#pcbangh)
*   [`pcbang.cpp`](#pcbangcpp)
*   [`PetSystem.h`](#petsystemh)
*   [`PetSystem.cpp`](#petsystemcpp)
*   [`polymorph.h`](#polymorphh)
*   [`polymorph.cpp`](#polymorphcpp)
*   [`priv_manager.h`](#priv_managerh)
*   [`priv_manager.cpp`](#priv_managercpp)
*   [`private_shop_manager.h`](#private_shop_managerh)
*   [`private_shop_manager.cpp`](#private_shop_managercpp)
*   [`private_shop_util.h`](#private_shop_utilh)
*   [`private_shop_util.cpp`](#private_shop_utilcpp)
*   [`private_shop.h`](#private_shoph)
*   [`private_shop.cpp`](#private_shopcpp)

---

### `OXEvent.h`

*   **Amaç:** Metin2 OX Yarışması etkinliğini yönetmek için `COXEventManager` singleton sınıfını ve ilgili yapıları (soru, durum enum'ları vb.) tanımlar. Oyuncuların etkinliğe katılımını, soruları, cevapları ve ödülleri yönetir.
*   **Temel İşlevler/İçeriği:**
    *   `OXEVENT_MAP_INDEX`: OX etkinliğinin yapıldığı haritanın indeksi (113).
    *   **`tag_Quiz` Struct:** Bir OX sorusunu temsil eder. İçeriği:
        *   `level`: Sorunun zorluk seviyesi.
        *   `Quiz[256]`: Soru metni.
        *   `answer`: Sorunun doğru cevabı (true/false).
    *   **`OXEventStatus` Enum:** OX etkinliğinin farklı durumlarını tanımlar:
        *   `OXEVENT_FINISH`: Etkinlik bitti.
        *   `OXEVENT_OPEN`: Etkinlik açık, oyuncu kabul ediliyor.
        *   `OXEVENT_CLOSE`: Etkinlik kapalı, yeni oyuncu kabul edilmiyor.
        *   `OXEVENT_QUIZ`: Soru sorma aşamasında.
        *   `OXEVENT_ERR`: Hata durumu.
    *   **`OXArea` Enum (`__OX_RENDER_AREA__` tanımlıysa):** Cevap alanlarını (O veya X) belirtmek için kullanılır.
    *   `MapEventChar` Typedef: `std::map<DWORD, DWORD>` için bir takma ad. Katılımcıların veya izleyicilerin Player ID'lerini saklamak için kullanılır.
    *   `QuizVector` Typedef: `std::vector<std::vector<tag_Quiz>>` için bir takma ad. Soruları seviyelerine göre gruplandırarak saklar.
    *   **`COXEventManager` Sınıfı (Singleton):**
        *   **Özel Üyeler:**
            *   `m_map_char`: Etkinlik haritasındaki tüm karakterlerin (katılımcı + izleyici) listesi.
            *   `m_map_attender`: Etkinliğe katılan yarışmacıların listesi.
            *   `m_map_miss`: Yanlış cevap veren ve izleyici alanına ışınlanacak yarışmacıların listesi.
            *   `m_vec_quiz`: Yüklenecek soruların vektörü.
            *   `m_timedEvent`: Soru süresi ve cevap kontrolü için zamanlayıcı.
        *   **Korumalı Metotlar:**
            *   `CheckAnswer()`: (Parametresiz versiyonu, muhtemelen cpp'de farklı bir implementasyon için kalmış olabilir veya eski bir versiyondan kalmadır.)
            *   `EnterAudience(LPCHARACTER pChar)`: Karakteri izleyici olarak etkinliğe alır.
            *   `EnterAttender(LPCHARACTER pChar)`: Karakteri yarışmacı olarak etkinliğe alır.
        *   **Genel Metotlar:**
            *   `Initialize()`, `Destroy()`: Yöneticinin başlatılması ve sonlandırılması.
            *   `GetStatus()`, `SetStatus(OXEventStatus status)`: Etkinlik durumunu alır ve ayarlar (Quest flag'i üzerinden).
            *   `LoadQuizScript(const char* szFileName)`: (Bu metot başlık dosyasında bildirilmiş ancak cpp dosyasında implementasyonu görünmüyor, muhtemelen lua veya başka bir script üzerinden dolaylı çağrılıyor veya eksik bir özellik.)
            *   `Enter(LPCHARACTER pChar)`: Bir karakterin OX haritasına girişini yönetir (koordinatlara göre katılımcı veya izleyici olarak ayırır).
            *   `CloseEvent()`: Etkinliği sonlandırır, tüm karakterleri kendi imparatorluklarının başlangıç noktasına ışınlar.
            *   `ClearQuiz()`: Yüklü olan tüm soruları temizler.
            *   `AddQuiz(unsigned char level, const char* pszQuestion, bool answer)`: Yeni bir soru ekler.
            *   `ShowQuizList(LPCHARACTER pChar)`: Yüklü soruları bir karaktere listeler (genellikle GM komutu için).
            *   `Quiz(unsigned char level, int timelimit)`: Belirli bir seviyeden rastgele bir soru seçer, duyurur ve zamanlayıcıyı başlatır.
            *   `GiveItemToAttender(DWORD dwItemVnum, WORD count)`: Kalan tüm katılımcılara belirtilen eşyayı verir.
            *   `CheckAnswer(bool answer)`: Oyuncuların doğru alanda (O veya X) olup olmadığını kontrol eder, yanlış cevap verenleri `m_map_miss` listesine ekler.
            *   `WarpToAudience()`: Yanlış cevap verenleri (m_map_miss) izleyici alanına ışınlar.
            *   `LogWinner()`: Kalan son katılımcıları (kazananları) loglar.
            *   `RenderArea(OXArea eArea) const` (`__OX_RENDER_AREA__` tanımlıysa): Oyuncuların ekranında O/X alanlarını görsel olarak işaretler/boyar.
            *   `GetAttenderCount()`: Anlık katılımcı sayısını döndürür.

### `OXEvent.cpp`

*   **Amaç:** `OXEvent.h` dosyasında bildirilen `COXEventManager` sınıfının metotlarını uygular. OX Yarışması'nın tüm mantığını, oyuncu yönetimini, soru sorma/cevaplama süreçlerini, zamanlayıcıları ve ödüllendirmeyi içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Durum Yönetimi:** `GetStatus` ve `SetStatus` metotları, `quest::CQuestManager` üzerinden `oxevent_status` adlı bir event flag'ini kullanarak etkinliğin genel durumunu (bitmiş, açık, kapalı, soru soruluyor) yönetir. Bu, quest scriptlerinin de OX durumunu kontrol edebilmesini sağlar.
    *   **Oyuncu Girişi:** `Enter` metodu, karakterin OX haritasına hangi koordinatlardan girdiğine bakarak onu ya yarışmacı (`EnterAttender`) ya da izleyici (`EnterAudience`) olarak kaydeder.
    *   **Soru Yönetimi:**
        *   `AddQuiz`: Soruları `m_vec_quiz` vektörüne seviyelerine göre ekler.
        *   `ClearQuiz`: Tüm soruları temizler.
        *   `Quiz`: Belirli bir seviyeden rastgele bir soru seçer, genel duyuru yapar ve `oxevent_timer` adlı bir event'i başlatır. Bu event, cevaplama süresini ve sonraki adımları kontrol eder.
    *   **Zamanlayıcı (`oxevent_timer`):**
        *   3 aşamalı bir zamanlayıcıdır:
            1.  "Cevap 10 saniye sonra açıklanacak" duyurusu yapar.
            2.  Doğru cevabı duyurur (`O` veya `X`), `CheckAnswer` çağrılır, yanlış cevap verenlerin 5 saniye sonra ışınlanacağı duyurulur. Eğer `__OX_RENDER_AREA__` tanımlıysa, doğru cevap alanı görsel olarak işaretlenir.
            3.  `WarpToAudience` çağrılarak yanlış cevap verenler ışınlanır. Etkinlik durumu `OXEVENT_CLOSE` olarak ayarlanır (bir sonraki soruya kadar). `__OX_RENDER_AREA__` tanımlıysa işaretleme kaldırılır.
    *   **Cevap Kontrolü (`CheckAnswer`):**
        *   Katılımcıların (`m_map_attender`) haritadaki koordinatlarını kontrol eder. Belirlenen doğru bölgenin (O veya X için tanımlanmış dikdörtgen alanlar) dışında kalanlar yanlış cevap vermiş sayılır.
        *   Yanlış cevap verenler `m_map_attender` listesinden çıkarılır ve `m_map_miss` listesine eklenir. Başarısız efekti gösterilir.
        *   Doğru cevap verenlere "Doğru!" mesajı gönderilir, sevinme animasyonu (`cheer1`/`cheer2`) ve başarılı efekti gösterilir.
    *   **Işınlama (`WarpToAudience`):** `m_map_miss` listesindeki tüm karakterleri OX haritasındaki rastgele izleyici pozisyonlarına ışınlar.
    *   **Etkinlik Kapatma (`CloseEvent`):** Aktif zamanlayıcıyı iptal eder. OX haritasındaki tüm karakterleri (`m_map_char`) kendi imparatorluklarının başlangıç noktasına ışınlar ve tüm listeleri temizler.
    *   **Ödüllendirme (`GiveItemToAttender`):** Etkinlik sonunda kalan tüm katılımcılara (`m_map_attender`) belirtilen VNUM'daki eşyadan belirtilen sayıda verir ve loglar.
    *   **Kazanan Loglama (`LogWinner`):** Kalan katılımcıları "LastManStanding" olarak loglar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `config.h`, `questmanager.h`, `start_position.h`, `packet.h`, `buffer_manager.h`, `log.h`, `char.h`, `char_manager.h`, `OXEvent.h`, `desc.h`.

---

### `party.h`

*   **Amaç:** Oyuncu partileri (grupları) için temel sınıfları (`CPartyManager`, `CParty`) ve ilgili enum'ları (parti rolleri, EXP dağıtım modları, sabitler) tanımlar. Parti oluşturma, yönetme, üye ekleme/çıkarma, P2P işlemleri, parti bonusları, zindan etkileşimleri ve diğer parti ile ilgili işlevlerin başlıklarını içerir.
*   **Temel İşlevler/İçerik:**
    *   **Enum'lar:**
        *   `PARTY_ENOUGH_MINUTE_FOR_EXP_BONUS`, `PARTY_HEAL_COOLTIME_LONG`, `PARTY_HEAL_COOLTIME_SHORT`, `PARTY_MAX_MEMBER`, `PARTY_DEFAULT_RANGE`: Parti mekanikleriyle ilgili temel sabitler.
        *   `EPartyRole`: Parti üyelerinin alabileceği rolleri tanımlar (Lider, Atakçı, Tank, Destekçi vb.).
        *   `EPartyExpDistributionModes`: Parti tecrübe puanı dağıtım modlarını belirtir (eşit, eşit olmayan).
        *   `EPartyMessages`: Parti içinde karakterler arası gönderilebilecek mesaj türleri (Saldır, Geri Dön vb.).
    *   **`CPartyManager` Sınıfı (Singleton):**
        *   Partilerin genel yönetimini üstlenir.
        *   `TPartyMap` (DWORD -> LPPARTY), `TPCPartySet` (LPPARTY seti) gibi typedef'ler içerir.
        *   `Initialize()`: Yöneticiyi başlatır.
        *   `EnablePCParty()`, `DisablePCParty()`: PC partilerini etkinleştirir/devre dışı bırakır.
        *   `CreateParty(LPCHARACTER pkLeader)`: Bir lider karakter ile yeni bir parti oluşturur.
        *   `DeleteParty(LPPARTY pParty)`: Belirtilen partiyi siler.
        *   `DeleteAllParty()`: Tüm PC partilerini siler.
        *   `SetParty(LPCHARACTER pkChr)`: Bir karakteri bir partiye atar (muhtemelen P2P senkronizasyonu sonrası).
        *   `SetPartyMember(DWORD dwPID, LPPARTY pParty)`: Bir oyuncu ID'sini bir partiyle eşleştirir veya eşleşmeyi kaldırır.
        *   **P2P Metotları:** `P2PLogin`, `P2PLogout`, `P2PCreateParty`, `P2PDeleteParty`, `P2PJoinParty`, `P2PQuitParty` gibi metotlar, sunucular arası parti bilgilerinin senkronizasyonu için kullanılır.
    *   **`CParty` Sınıfı:**
        *   Tek bir partiyi temsil eder.
        *   **`SMember` Struct:** Parti üyesinin bilgilerini tutar (karakter işaretçisi, yakınlık durumu, rolü, seviyesi, adı).
        *   `TMemberMap` (DWORD -> TMember): Parti üyelerini (Player ID -> Üye Bilgisi) saklar.
        *   `TFlagMap` (std::string -> int): Partiyle ilgili özel durum bayraklarını (genellikle zindanlar için) saklar.
        *   **Üye Yönetimi:** `P2PJoin`, `P2PQuit`, `Join`, `Quit`, `Link`, `Unlink` metotları ile parti üyeleri eklenir, çıkarılır, karakter nesneleri partiye bağlanır/ayrılır.
        *   `UpdateOnlineState`, `UpdateOfflineState`: Parti üyelerinin online/offline durumlarını günceller.
        *   `GetLeaderPID()`, `GetLeaderCharacter()`, `GetMemberCount()`, `GetNearMemberCount()`: Parti lideri ve üye sayıları hakkında bilgi verir.
        *   **Paket Gönderme:** `SendPartyJoinOneToAll`, `SendPartyInfoOneToAll` gibi çok sayıda `Send*` metodu, parti ile ilgili durum değişikliklerini (üye ekleme, çıkarma, bilgi güncelleme, link/unlink) istemcilere iletmek için kullanılır.
        *   **Bonus ve Roller:**
            *   `ComputePartyBonusExpPercent()`, `ComputePartyBonusAttackGrade()`, `ComputePartyBonusDefenseGrade()`: Parti bonuslarını hesaplar.
            *   `SetRole()`, `GetRole()`, `IsRole()`: Parti üyelerine rol atar ve rollerini sorgular.
            *   `ComputeRolePoint()`: Liderin liderlik seviyesine ve üyenin rolüne göre bonus puanlarını hesaplar ve karaktere uygular.
        *   **İşlevler:** `HealParty()`, `SummonToLeader()` gibi özel parti yeteneklerini içerir.
        *   `Update()`: Partinin durumunu periyodik olarak günceller (yakın üyeleri kontrol etme, bonusları yeniden hesaplama vb.).
        *   **Zindan Entegrasyonu:** `SetDungeon()`, `GetDungeon()`, `SetDungeon_for_Only_party()`, `GetDungeon_for_Only_party()` metotları ile partinin bir zindanla ilişkilendirilmesini sağlar.
        *   `GetFlag()`, `SetFlag()`: Zindan ilerlemesi gibi özel durumları takip etmek için bayrakları yönetir.
        *   `SetParameter()`, `GetExpDistributionMode()`: Tecrübe puanı dağıtım modunu ayarlar.
        *   **Şablon Fonksiyonları:** `ForEachMember`, `ForEachOnlineMember`, `ForEachNearMember` gibi fonksiyonlar, parti üyeleri üzerinde belirli işlemleri kolayca yapmak için kullanılır.
    *   **`FPartyDropDiceRoll` Struct (`__DICE_SYSTEM__` tanımlıysa):** Parti içinde eşya düşürme durumunda zar atma sistemini yönetir.
*   **Bağlantılı Dosyalar:** `party.cpp` (uygulama), `char.h`, `item.h` (opsiyonel).

### `party.cpp`

*   **Amaç:** `party.h` dosyasında bildirilen `CPartyManager` ve `CParty` sınıflarının metotlarını uygular. Oyuncu partilerinin oluşturulması, dağıtılması, üye yönetimi, P2P senkronizasyonu, bonus hesaplamaları, rol atamaları, durum güncellemeleri ve istemciye paket gönderimleri gibi tüm parti mekaniklerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPartyManager` Metotları:**
        *   `Initialize()`: `m_bEnablePCParty`'yi `false` olarak ayarlar.
        *   `DeleteAllParty()`: `m_set_pkPCParty` içindeki tüm partileri siler.
        *   `CreateParty(pLeader)`: Yeni bir `CParty` nesnesi oluşturur. Lider PC ise DB'ye `HEADER_GD_PARTY_CREATE` paketi gönderir, lideri partiye ekler ve PC partisi olarak işaretler. NPC ise sadece partiye ekler. Lideri partiye `Link` eder.
        *   `DeleteParty(pParty)`: DB'ye `HEADER_GD_PARTY_DELETE` paketi gönderir, `CParty` nesnesini siler.
        *   `SetPartyMember(dwPID, pParty)`: `m_map_pkParty` haritasını günceller. Bir oyuncuyu bir partiye atar veya var olan atamayı kaldırır.
        *   **P2P İşlemleri:** `P2PLogin`, `P2PLogout` gibi fonksiyonlar, diğer sunuculardan gelen parti ile ilgili olayları işler (örneğin, başka bir sunucudaki parti üyesinin online/offline olması, partiye katılması/ayrılması). Bu fonksiyonlar genellikle ilgili `CParty` nesnesinin metotlarını çağırır.
    *   **`CParty` Metotları:**
        *   **Yapıcı/Yıkıcı/Başlatma:**
            *   `Initialize()`: Parti ile ilgili değişkenleri (EXP dağıtım modu, lider PID, roller, bonuslar, zamanlayıcılar vb.) varsayılan değerlere ayarlar.
            *   `Destroy()`: Parti dağıtıldığında çağrılır. Tüm event'leri iptal eder, bonusları kaldırır, üyelere parti dağıldı bilgisini gönderir, üye karakterlerinin parti referanslarını `NULL` yapar ve üye haritasını temizler.
        *   **Üye Yönetimi:**
            *   `Join(dwPID)` / `P2PJoin(dwPID)`: Bir oyuncuyu (PID ile) partiye ekler. Eğer PC partisi ise, `CPartyManager`'a bildirir, herkese yeni üye bilgisini (`SendPartyJoinOneToAll`) ve yeni üyeye parti parametrelerini (`SendParameter`) gönderir. DB'ye `HEADER_GD_PARTY_ADD` paketi yollar.
            *   `Quit(dwPID)` / `P2PQuit(dwPID)`: Bir oyuncuyu partiden çıkarır. Herkese ayrılan üye bilgisini (`SendPartyRemoveOneToAll`) gönderir, üyenin bonuslarını kaldırır (`RemoveBonusForOne`), karakterin parti referansını `NULL` yapar. Eğer ayrılan lider ise `CPartyManager::DeleteParty` çağrılır. DB'ye `HEADER_GD_PARTY_REMOVE` paketi yollar.
            *   `Link(pkChr)`: Bir `LPCHARACTER` nesnesini partideki ilgili üyeyle eşleştirir. Parti güncelleme event'ini (`party_update_event`) başlatır. Eğer mini haritada parti gösterme özelliği aktifse (`__WJ_SHOW_PARTY_ON_MINIMAP__`) pozisyon gönderme event'ini de başlatır. Tüm üyelere ve yeni bağlanan üyeye link/bilgi paketlerini gönderir.
            *   `Unlink(pkChr)`: Bir `LPCHARACTER` nesnesinin partideki üye ile bağını koparır. Herkese unlink paketini gönderir.
        *   **Paket Gönderme:** Çok sayıda `SendParty*` fonksiyonu, parti durumlarını (`TPacketGCPartyAdd`, `TPacketGCPartyRemove`, `TPacketGCPartyLink`, `TPacketGCPartyUnlink`, `TPacketGCPartyUpdate`) ilgili istemcilere gönderir.
        *   **Durum Güncelleme (`Update()`):**
            *   Periyodik olarak çağrılır (event ile).
            *   Lider karakterin varlığını kontrol eder.
            *   Parti üyelerinin lidere yakın olup olmadığını (`bNear`) hesaplar.
            *   Liderin liderlik seviyesini (`m_iLeadership`) alır.
            *   EXP, atak, defans bonuslarını (`ComputePartyBonusExpPercent`, `ComputePartyBonusAttackGrade`, `ComputePartyBonusDefenseGrade`) yeniden hesaplar.
            *   Uzun süreli parti bonusunu (`m_iLongTimeExpBonus`) kontrol eder ve uygular.
            *   Her üye için `ComputeRolePoint` ile rol bonuslarını günceller.
            *   Parti iyileştirme (`HealParty`) yeteneğinin kullanılabilirliğini günceller.
            *   Gerekirse tüm üyelere güncel parti bilgilerini (`SendPartyInfoOneToAll`) gönderir.
        *   **Rol ve Bonus Hesaplama (`ComputeRolePoint`):**
            *   Liderin liderlik seviyesine (`m_iLeadership`) ve üyenin aktif rolüne (`bRole`) göre çeşitli puan türlerinde (örn: `POINT_PARTY_ATTACKER_BONUS`) bonusları hesaplar ve `ch->PointChange` ile karaktere uygular veya kaldırır.
        *   **EXP Dağıtımı ve Parametreler:**
            *   `SetParameter(iMode)`: EXP dağıtım modunu (`m_iExpDistributionMode`) ayarlar ve tüm üyelere yeni parametreyi (`SendParameterToAll`) gönderir.
            *   `SetExpCentralizeCharacter(dwPID)`: EXP'nin merkezileştirileceği karakteri ayarlar.
        *   **Diğer İşlevler:**
            *   `ChatPacketToAllMember()`: Tüm parti üyelerine sohbet mesajı gönderir.
            *   `SummonToLeader(pid)`: Belirli bir üyeyi liderin yanına ışınlar (liderlik yeteneği gerektirir).
            *   `HealParty()`: Yakındaki tüm parti üyelerini iyileştirir (liderlik yeteneği ve bekleme süresi gerektirir).
            *   `GetNextOwnership()`: Eşya sahipliği için sıradaki üyeyi belirler.
            *   `SetFlag()`, `GetFlag()`: Zindan gibi özel durumlar için bayrakları yönetir.
    *   **Event Fonksiyonları:**
        *   `party_update_event`: `CParty::Update()` fonksiyonunu periyodik olarak çağırır.
        *   `party_position_event` (`__WJ_SHOW_PARTY_ON_MINIMAP__`): `CParty::SendPositionInfo()`'yu çağırarak parti üyelerinin mini haritadaki pozisyonlarını gönderir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `char.h`, `party.h`, `char_manager.h`, `config.h`, `p2p.h`, `desc_client.h`, `dungeon.h`, `unique_item.h`, `buffer_manager.h`.

### `pcbang.h`

*   **Amaç:** PC Bang (İnternet Kafe) sistemiyle ilgili IP adreslerini ve ID'lerini yönetmek için `CPCBangManager` singleton sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:**
        *   `PCBang_IP` (unsigned long): Bir IP adresini sayısal olarak temsil eder.
        *   `PCBang_ID` (unsigned long): Bir PC Bang ID'sini temsil eder.
    *   **`CPCBangManager` Sınıfı (Singleton):**
        *   **Özel Üyeler:**
            *   `m_map_ip` (std::map<PCBang_IP, PCBang_ID>): Sayısal IP adreslerini PC Bang ID'leriyle eşleştiren bir harita.
            *   `m_minSavablePlayTime` (time_t): Kaydedilebilir minimum oyun süresi.
            *   `__GetIDFromString(const char* c_szID)`: String olarak verilen ID'yi sayısal `PCBang_ID`'ye dönüştürür.
            *   `__GetIPFromString(const char* c_szIP)`: String olarak verilen IP adresini sayısal `PCBang_IP`'ye dönüştürür.
        *   **Genel Metotlar:**
            *   `CPCBangManager()`: Kurucu metot, `m_minSavablePlayTime`'ı başlatır.
            *   `InsertIP(const char* c_szID, const char* c_szIP)`: Verilen ID ve IP çiftini `m_map_ip`'e ekler.
            *   `Log(const char* c_szIP, DWORD pid, time_t playTime)`: Belirtilen IP adresinden bağlanan oyuncunun (pid) oynama süresini loglar. Sadece oyun süresi `m_minSavablePlayTime`'dan büyükse ve IP `m_map_ip`'te kayıtlıysa loglama yapılır.
            *   `RequestUpdateIPList(PCBang_ID id)`: Veritabanından PC Bang IP listesini güncelleme isteği gönderir. `id` 0 ise genel bir kontrol yapar, değilse belirli bir ID'ye ait IP'leri seçer (genellikle Kore veya Ymir lokalleri için).
            *   `IsPCBangIP(const char* c_szIP)`: Verilen IP adresinin kayıtlı bir PC Bang IP'si olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `pcbang.cpp` (uygulama), `stdafx.h`.

### `pcbang.cpp`

*   **Amaç:** `pcbang.h`'de bildirilen `CPCBangManager` sınıfının metotlarını uygular. PC Bang IP'lerini yönetme, string'den sayısal IP/ID dönüşümleri, oyun süresi loglama ve IP listesi güncelleme isteklerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPCBangManager::CPCBangManager()`:**
        *   `m_minSavablePlayTime`'ı 10 saniye olarak ayarlar.
        *   `m_map_ip` haritasını temizler.
    *   **`CPCBangManager::InsertIP(c_szID, c_szIP)`:**
        *   `__GetIPFromString` ve `__GetIDFromString` kullanarak string girdileri sayısal formatlara dönüştürür.
        *   Dönüştürülmüş IP ve ID çiftini `m_map_ip` haritasına ekler.
    *   **`CPCBangManager::Log(c_szIP, pid, playTime)`:**
        *   Eğer `playTime`, `m_minSavablePlayTime`'dan küçükse `false` döner.
        *   Verilen `c_szIP`'yi `__GetIPFromString` ile sayısal IP'ye dönüştürür ve `m_map_ip`'te arar.
        *   IP bulunamazsa `false` döner.
        *   Bulunursa, `LogManager::instance().PCBangLoginLog()` fonksiyonunu çağırarak PC Bang ID'si, IP adresi, oyuncu ID'si (pid) ve oynama süresini loglar.
        *   Başarılı olursa `true` döner.
    *   **`CPCBangManager::RequestUpdateIPList(id)`:**
        *   Sadece Ymir (`LC_IsYMIR()`) veya Kore (`LC_IsKorea()`) lokalleri için çalışır.
        *   Eğer `id` 0 ise, `DBManager` aracılığıyla `QID_PCBANG_IP_LIST_CHECK` sorgusu ile veritabanındaki `pcbang_ip` tablosunun durumunu kontrol etmek için "show table status" sorgusu gönderir.
        *   Eğer `id` 0 değilse, `DBManager` aracılığıyla `QID_PCBANG_IP_LIST_SELECT` sorgusu ile `pcbang_ip` tablosundan belirtilen `id`'ye ait `pcbang_id` ve `ip` sütunlarını seçen bir sorgu gönderir.
    *   **`CPCBangManager::IsPCBangIP(c_szIP)`:**
        *   `c_szIP` null ise `false` döner.
        *   `__GetIPFromString` ile IP'yi sayısal forma dönüştürür ve `m_map_ip`'te arar.
        *   Bulunamazsa `false`, bulunursa `true` döner.
    *   **`CPCBangManager::__GetIDFromString(c_szID)`:**
        *   `str_to_number` fonksiyonunu kullanarak string ID'yi `PCBang_ID` (unsigned long) türüne dönüştürür.
    *   **`CPCBangManager::__GetIPFromString(c_szIP)`:**
        *   `sscanf` kullanarak IP stringini (örn: "192.168.1.10") dört ayrı integer değere (`nums[0]`-`nums[3]`) ayırır.
        *   Bu dört sayıyı bit kaydırma (bit-shifting) ve OR işlemleriyle tek bir `unsigned long` (PCBang_IP) değerine birleştirir (örn: `(nums[0] << 24) | (nums[1] << 16) | (nums[2] << 8) | nums[3]`).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `config.h`, `locale_service.h`, `log.h`, `db.h`, `pcbang.h`.

### `PetSystem.h`

*   **Amaç:** Oyunculara ait evcil hayvan (pet) sistemini yönetmek için `CPetActor` ve `CPetSystem` sınıflarını tanımlar. Petlerin çağrılması, kaybolması, takip etmesi, binilebilmesi ve sahibine bonuslar vermesi gibi işlevleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`SPetAbility` Struct:** (Boş bir yapı, muhtemelen gelecekteki pet yetenekleri için bir yer tutucu.)
    *   **`CPetActor` Sınıfı:** Tek bir evcil hayvanı temsil eder.
        *   **`EPetOptions` Enum:** Petlerin sahip olabileceği seçenekleri bit flag olarak tanımlar (`EPetOption_Followable`, `EPetOption_Mountable`, `EPetOption_Summonable`, `EPetOption_Combatable`).
        *   **Yapıcı/Yıkıcı:** Pet aktörünü sahibi, VNUM'u ve seçenekleriyle başlatır. Yıkıcı, peti `Unsummon` eder.
        *   `Update(DWORD deltaTime)`: Petin davranışını (örn: takip etme) günceller.
        *   `_UpdateFollowAI()`: Petin sahibini takip etme yapay zekasını yönetir.
        *   `_UpdatAloneActionAI()`: Petin yalnızken yapacağı rastgele hareketlerin yapay zekasını yönetir.
        *   `Follow(float fMinDistance)`: Petin sahibine belirli bir mesafeye kadar yaklaşmasını sağlar.
        *   `GetCharacter()`, `GetOwner()`, `GetVID()`, `GetVnum()`: Petin karakter nesnesini, sahibini, VID'sini ve VNUM'unu döndürür.
        *   `HasOption(EPetOptions option)`: Petin belirli bir seçeneğe sahip olup olmadığını kontrol eder.
        *   `SetName(const char* petName)`: Petin adını ayarlar (genellikle sahibinin adıyla ilişkilendirilir).
        *   `Mount()`, `Unmount()`: Petin binek olarak kullanılmasını sağlar veya binek durumunu sonlandırır.
        *   `Summon(const char* petName, LPITEM pSummonItem, bool bSpawnFar)`: Peti çağırır. Belirli haritalarda çağırma engellenebilir. Çağırma eşyasını (`pSummonItem`) kaydeder.
        *   `Unsummon()`: Peti ortadan kaldırır, verdiği bonusları temizler ve çağırma eşyasının durumunu günceller.
        *   `IsSummoned()`: Petin çağrılmış olup olmadığını kontrol eder.
        *   `SetSummonItem(LPITEM pItem)`: Peti çağıran eşyayı ayarlar. `USE_ACTIVE_PET_SEAL_EFFECT` tanımlıysa, mühür eşyasının aktiflik soketini günceller.
        *   `GetSummonItemVID()`, `GetSummonItemVnum()`: Çağırma eşyasının VID ve VNUM'unu döndürür.
        *   `GiveBuff()`: Petin çağırma eşyasından gelen bonusları sahibine uygular.
        *   `ClearBuff()`: Petin verdiği bonusları sahibinden kaldırır.
        *   `CheckBuff(const CPetActor* pPetActor)`: Belirli petlerin (VNUM'a göre) sadece özel durumlarda (örn: zindanda) bonus vermesini kontrol eder.
    *   **`CPetSystem` Sınıfı:** Bir oyuncuya ait tüm petleri yönetir.
        *   `TPetActorMap` (boost::unordered_map<DWORD, CPetActor*>): Pet VNUM'larını `CPetActor` işaretçileriyle eşleştiren bir harita.
        *   **Yapıcı/Yıkıcı:** Pet sistemini sahibiyle başlatır. Yıkıcı, tüm petleri `Destroy` eder.
        *   `GetByVID(DWORD vid)`, `GetByVnum(DWORD vnum)`: Pet aktörünü VID veya VNUM ile bulur.
        *   `GetSummonItemVID()`, `GetSummonItemVnum()`: Çağrılmış olan bir petin çağırma eşyasının VID/VNUM'unu döndürür.
        *   `Update(DWORD deltaTime)`: Tüm çağrılmış petlerin `Update` fonksiyonlarını çağırır. Belirli bir periyotta çalışır (`m_dwUpdatePeriod`).
        *   `Destroy()`: Tüm pet aktörlerini siler ve güncelleme event'ini iptal eder.
        *   `CountSummoned()`: Çağrılmış olan pet sayısını döndürür.
        *   `SetUpdatePeriod(DWORD ms)`: Pet güncelleme sıklığını ayarlar.
        *   `Summon(DWORD mobVnum, LPITEM pSummonItem, ...)`: Belirtilen VNUM'daki peti çağırır. Eğer bu VNUM için bir `CPetActor` yoksa yenisini oluşturur. Pet güncelleme event'ini başlatır.
        *   `Unsummon(DWORD mobVnum, bool bDeleteFromList)`: Belirli bir VNUM'daki peti geri çağırır. İsteğe bağlı olarak listeden de silebilir.
        *   `UnsummonAll()`: Oyuncuya ait tüm çağrılmış petleri geri çağırır.
        *   `DeletePet(DWORD mobVnum)`: Belirli bir VNUM'daki peti sistemden siler.
        *   `RefreshBuff()`: Tüm çağrılmış petlerin bonuslarını yeniler (tekrar uygular).
*   **Bağlantılı Dosyalar:** `PetSystem.cpp` (uygulama), `char.h`, `item.h`.

### `PetSystem.cpp`

*   **Amaç:** `PetSystem.h` dosyasında bildirilen `CPetActor` ve `CPetSystem` sınıflarının metotlarını uygular. Petlerin çağrılması, hareketleri, bonusları, yapay zekası ve genel yönetimini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`petsystem_update_event` (Event Fonksiyonu):** `CPetSystem::Update` metodunu periyodik olarak (saniyenin 1/4'ü) çağırır.
    *   **`CPetActor` Metotları:**
        *   **Yapıcı (`CPetActor(...)`):** VNUM, sahip, seçenekler gibi temel bilgileri ayarlar. Karakter (`m_pkChar`) ve VID başlangıçta sıfırdır.
        *   **Yıkıcı (`~CPetActor()`):** Peti `Unsummon` eder.
        *   **`SetName(name)`:** Çağrılmışsa petin `CHARACTER` nesnesinin adını ayarlar.
        *   **`Mount()`, `Unmount()`:** Eğer pet `EPetOption_Mountable` seçeneğine sahipse, sahibinin `MountVnum` veya `StopRiding` metotlarını çağırır.
        *   **`Unsummon(...)`:** Pet çağrılmışsa, `ClearBuff()` ile bonuslarını temizler, `SetSummonItem(NULL)` ile çağırma eşyasıyla bağını koparır (ve `USE_ACTIVE_PET_SEAL_EFFECT` aktifse mühür soketini günceller), sahibinin puanlarını yeniden hesaplatır (`ComputePoints`) ve petin `CHARACTER` nesnesini `M2_DESTROY_CHARACTER` ile yok eder.
        *   **`Summon(...)`:** Çağırma işlemini gerçekleştirir. `IS_BLOCKED_PET_SUMMON_MAP` ile harita kontrolü yapar. Sahibinin yakınına `CHARACTER_MANAGER::instance().SpawnMob` ile peti spawn eder, `SetPet()` ile pet olarak işaretler, imparatorluğunu ayarlar, adını ayarlar ve çağırma eşyasını `SetSummonItem` ile kaydeder. Sahibinin puanlarını yeniden hesaplatır.
        *   **`_UpdateFollowAI()`:** Petin sahibini takip etme mantığını içerir.
            *   Sahiple arasındaki mesafeyi (`fDist`) hesaplar.
            *   `RESPAWN_DISTANCE`'dan uzaksa, sahibinin yakınına ışınlar (`m_pkChar->Show`).
            *   `START_FOLLOW_DISTANCE`'dan uzaksa, `Follow()` metodunu çağırarak sahibine doğru hareket eder. `START_RUN_DISTANCE`'dan da uzaksa koşarak takip eder.
            *   Yakınsa `FUNC_WAIT` ile bekler.
        *   **`_UpdatAloneActionAI(fMinDist, fMaxDist)`:** Pet sahibinden belirli bir mesafedeyken rastgele pozisyonlara gitmesini sağlar (şu anki implementasyonda tam aktif olmayabilir).
        *   **`Update(deltaTime)`:** Ana güncelleme fonksiyonu. Sahibinin veya petin ölü olup olmadığını, çağırma eşyasının geçerliliğini kontrol eder. Eğer sorun varsa peti `Unsummon` eder. Takip edilebilir (`EPetOption_Followable`) ise `_UpdateFollowAI` çağırır.
        *   **`Follow(fMinDistance)`:** Petin sahibine `fMinDistance` mesafesine kadar yaklaşmasını sağlar. `SetRotationToXY` ile sahibine döner ve `Goto` ile hareket eder.
        *   **`SetSummonItem(pItem, updateSocket)`:** Çağırma eşyasının VID ve VNUM'unu saklar. `USE_ACTIVE_PET_SEAL_EFFECT` tanımlıysa, eşyanın `PET_SEAL_ACTIVE_SOCKET_IDX` soketini `true` (çağrılı) veya `false` (geri çağrılmış) olarak ayarlar.
        *   **`GiveBuff()`:** Çağırma eşyası (`m_dwSummonItemVID`) üzerinden `item->ModifyPoints(true)` çağırarak sahibine bonusları uygular. `__SET_ITEM__` aktifse set bonuslarını da yeniler.
        *   **`ClearBuff()`:** Çağırma eşyasının VNUM'u (`m_dwSummonItemVnum`) üzerinden `ITEM_MANAGER`'dan eşya prototipini alır ve prototipteki tüm `aApplies` (bonuslar) için sahibinden `ApplyPoint` ile negatif değerde bonus uygulayarak etkilerini geri alır.
        *   **`CheckBuff(pPetActor)`:** Belirli VNUM'lara sahip petlerin (örn: 34004, 34009) sadece sahibi bir zindandayken bonus vermesini sağlar.
    *   **`CPetSystem` Metotları:**
        *   **Yapıcı (`CPetSystem(owner)`):** Sahibi (`m_pkOwner`) ve varsayılan güncelleme periyodunu (`m_dwUpdatePeriod`) ayarlar.
        *   **Yıkıcı (`~CPetSystem()`):** `Destroy()` çağırır.
        *   **`Destroy()`:** `m_petActorMap`'teki tüm `CPetActor` nesnelerini siler ve `petsystem_update_event`'i iptal eder.
        *   **`Update(deltaTime)`:** Periyodik event tarafından çağrılır. `m_petActorMap`'teki tüm çağrılmış ve geçerli petlerin `CPetActor::Update()` metodunu çağırır. Geçersiz hale gelmiş (sahibi ölmüş, eşyası kaybolmuş vb.) petleri `v_garbageActor` listesine ekleyip sonra `DeletePet` ile siler.
        *   **`DeletePet(mobVnum)` veya `DeletePet(petActor)`:** Belirtilen peti `m_petActorMap`'ten bulup siler.
        *   **`Unsummon(vnum, bDeleteFromList)`:** `GetByVnum` ile peti bulur, `CPetActor::Unsummon` çağırır. Eğer başka çağrılı pet kalmadıysa `petsystem_update_event`'i iptal eder.
        *   **`Summon(...)`:** `GetByVnum` ile peti arar, yoksa yeni bir `CPetActor` oluşturup `m_petActorMap`'e ekler. `CPetActor::Summon` ile peti çağırır. Eğer `petsystem_update_event` aktif değilse başlatır.
        *   `GetByVID()`, `GetByVnum()`: `m_petActorMap` üzerinden petleri VID veya VNUM ile arar.
        *   `CountSummoned()`: `m_petActorMap`'teki çağrılmış pet sayısını döndürür.
        *   `RefreshBuff()`: Tüm çağrılmış petlerin `CPetActor::GiveBuff()` metodunu çağırarak bonuslarını yeniler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `vector.h`, `char.h`, `sectree_manager.h`, `char_manager.h`, `mob_manager.h`, `PetSystem.h`, `VnumHelper.h`, `packet.h`, `item_manager.h`, `item.h`, `config.h`.

### `polymorph.h`

*   **Amaç:** Karakterlerin canavarlara dönüşmesini (polymorph) sağlayan ve bu dönüşümle ilgili bonusları, becerileri ve eşyaları (dönüşüm kitabı) yöneten `CPolymorphUtils` singleton sınıfını tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Sabitler:**
        *   `POLYMORPH_SKILL_ID` (129): Dönüşüm becerisinin ID'si.
        *   `POLYMORPH_BOOK_ID` (50322): Dönüşüm kitabının eşya VNUM'u.
    *   **`POLYMORPH_BONUS_TYPE` Enum:** Dönüşüm sonrası kazanılabilecek bonus türlerini tanımlar (`POLYMORPH_NO_BONUS`, `POLYMORPH_ATK_BONUS`, `POLYMORPH_DEF_BONUS`, `POLYMORPH_SPD_BONUS`).
    *   **`CPolymorphUtils` Sınıfı (Singleton):**
        *   **Özel Üyeler:**
            *   `m_mapSPDType` (boost::unordered_map<DWORD, DWORD>): Hız bonusu veren canavar VNUM'larını tutar.
            *   `m_mapATKType` (boost::unordered_map<DWORD, DWORD>): Saldırı bonusu veren canavar VNUM'larını tutar.
            *   `m_mapDEFType` (boost::unordered_map<DWORD, DWORD>): Savunma bonusu veren canavar VNUM'larını tutar.
        *   **Genel Metotlar:**
            *   `CPolymorphUtils()`: Kurucu metot, bonus haritalarını (şu an için sadece `m_mapSPDType`'a 101 ve 1901 VNUM'ları ekleniyor) başlatır.
            *   `GetBonusType(DWORD dwVnum)`: Verilen canavar VNUM'una göre hangi tür bonusun (atak, defans, hız) verileceğini döndürür.
            *   `PolymorphCharacter(LPCHARACTER pChar, LPITEM pItem, const CMob* pMob)`: Karakteri (`pChar`), verilen dönüşüm kitabını (`pItem`) kullanarak belirtilen canavara (`pMob`) dönüştürür. Dönüşüm süresini ve şansını karakterin dönüşüm beceri seviyesine ve kitaptaki bilgilere göre hesaplar. Başarılı olursa `AFFECT_POLYMORPH` etkisini ve ilgili bonus etkisini (atak, defans, hız) karaktere ekler.
            *   `UpdateBookPracticeGrade(LPCHARACTER pChar, LPITEM pItem)`: Dönüşüm kitabının alıştırma sayısını (socket 1) azaltır. Sıfıra ulaşırsa kullanıcıya bilgi verir.
            *   `GiveBook(LPCHARACTER pChar, DWORD dwMobVnum, DWORD dwPracticeCount, BYTE BookLevel, BYTE LevelLimit)`: Karaktere yeni bir dönüşüm kitabı (`POLYMORPH_BOOK_ID`) verir. Kitabın soketlerini (0: dönüşülecek mob vnum, 1: alıştırma sayısı, 2: kitap seviyesi) ayarlar.
            *   `BookUpgrade(LPCHARACTER pChar, LPITEM pItem)`: Dönüşüm kitabının seviyesini (socket 2) artırır ve alıştırma sayısını (socket 1) yeni seviyeye göre günceller.
*   **Bağlantılı Dosyalar:** `polymorph.cpp` (uygulama), `char.h`, `item.h`, `mob_manager.h`, `affect.h`.

### `polymorph.cpp`

*   **Amaç:** `polymorph.h`'de bildirilen `CPolymorphUtils` sınıfının metotlarını uygular. Karakter dönüşümü, dönüşüm kitabı yönetimi ve bonus hesaplamaları gibi işlevleri içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPolymorphUtils::CPolymorphUtils()`:**
        *   Kurucu metotta, `m_mapSPDType` haritasına örnek olarak VNUM 101 (vahşi köpek) ve 1901 (kurt) için girişler eklenir. Diğer bonus türleri için (`m_mapATKType`, `m_mapDEFType`) başlangıçta haritalar boştur, muhtemelen veritabanından veya başka bir konfigürasyon dosyasından doldurulması beklenir.
    *   **`CPolymorphUtils::GetBonusType(dwVnum)`:**
        *   Verilen `dwVnum`'ı sırayla hız, saldırı ve savunma bonusu haritalarında arar. Eşleşme bulunursa ilgili bonus türünü döndürür, bulunamazsa `POLYMORPH_NO_BONUS` döner.
    *   **`CPolymorphUtils::PolymorphCharacter(pChar, pItem, pMob)`:**
        *   Karakterin `POLYMORPH_SKILL_ID` (129) ID'li dönüşüm becerisinin seviyesine (`bySkillLevel`) ve ustalık türüne göre dönüşüm süresini (`dwDuration`) belirler (Normal: 10, Master: 15, GM: 20, PM: 25 dakika).
        *   Dönüşüm şansını (`iPolyPercent`) hesaplar: `Karakter Seviyesi - Mob Seviyesi + Kitap Seviyesi (Socket 2) + 29 + Dönüşüm Beceri Seviyesi`.
        *   Şans %0 veya daha düşükse ya da rastgele bir sayı şanstan büyükse dönüşüm başarısız olur.
        *   Başarılı olursa, karaktere `AFFECT_POLYMORPH` türünde, `POINT_POLYMORPH` noktasına `pMob->m_table.dwVnum` (dönüşülen canavarın VNUM'u) değerini ve hesaplanan süreyi içeren bir etki ekler.
        *   Dönüşüm bonus yüzdesini (`dwBonusPercent`) hesaplar: `Dönüşüm Beceri Seviyesi + Kitap Seviyesi (Socket 2)`.
        *   `GetBonusType` ile canavarın bonus türünü belirler ve ilgili bonus için (örn: `POINT_ATT_BONUS`) `AFFECT_POLYMORPH` türünde, hesaplanan bonus yüzdesini ve (süre-1) dakika süreli ikinci bir etki ekler.
    *   **`CPolymorphUtils::UpdateBookPracticeGrade(pChar, pItem)`:**
        *   Dönüşüm kitabının (`pItem`) `socket(1)` değerini (kalan alıştırma sayısı) bir azaltır. Eğer zaten 0 ise kullanıcıya bilgi verir.
    *   **`CPolymorphUtils::GiveBook(pChar, dwMobVnum, dwPracticeCount, BookLevel, LevelLimit)`:**
        *   `pChar->AutoGiveItem(POLYMORPH_BOOK_ID, 1)` ile karaktere dönüşüm kitabı verir.
        *   Kitabın soketlerini ayarlar:
            *   `socket(0)`: `dwMobVnum` (dönüşülecek canavarın VNUM'u).
            *   `socket(1)`: `dwPracticeCount` (kalan alıştırma sayısı).
            *   `socket(2)`: `BookLevel` (kitabın seviyesi).
        *   Eğer `dwMobVnum` geçersizse hata loglar.
    *   **`CPolymorphUtils::BookUpgrade(pChar, pItem)`:**
        *   Dönüşüm kitabının `socket(2)` değerini (kitap seviyesi) bir artırır.
        *   `socket(1)` değerini (kalan alıştırma sayısı) yeni kitap seviyesinin 50 katı olarak ayarlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `mob_manager.h`, `affect.h`, `item.h`, `polymorph.h`.

### `priv_manager.h`

*   **Amaç:** Oyundaki karakterler, loncalar ve imparatorluklar için özel ayrıcalıklar veya bonuslar (örn: eşya düşürme oranı, EXP bonusu vb.) yönetmek için `CPrivManager` singleton sınıfını tanımlar. Bu ayrıcalıkların verilmesi, kaldırılması ve sorgulanması için metotlar içerir.
*   **Temel İşlevler/İçerik:**
    *   **`SPrivGuildData` Struct:** Lonca ayrıcalıklarının değerini (`value`) ve bitiş zamanını (`end_time_sec`) tutar.
    *   **Typedef'ler:**
        *   `PrivCharMap` (std::map<DWORD, int>): Karakter PID'lerini ayrıcalık değerleriyle eşleştirir.
        *   `PrivGuildMap` (std::map<DWORD, SPrivGuildData>): Lonca ID'lerini `SPrivGuildData` ile eşleştirir.
    *   **`CPrivManager` Sınıfı (Singleton):**
        *   **`SPrivEmpireData` Struct (İç Sınıf):** İmparatorluk ayrıcalıklarının değerini (`m_value`) ve bitiş zamanını (`m_end_time_sec`) tutar.
        *   **Genel Metotlar (Veritabanı İstekleri):**
            *   `RequestGiveGuildPriv(...)`: Bir loncaya belirli bir türde, değerde ve sürede ayrıcalık verilmesi için veritabanına istek gönderir (`HEADER_GD_REQUEST_GUILD_PRIV`).
            *   `RequestGiveEmpirePriv(...)`: Bir imparatorluğa belirli bir türde, değerde ve sürede ayrıcalık verilmesi için veritabanına istek gönderir (`HEADER_GD_REQUEST_EMPIRE_PRIV`).
            *   `RequestGiveCharacterPriv(...)`: Bir karaktere belirli bir türde ve değerde ayrıcalık verilmesi için veritabanına istek gönderir (`HEADER_GD_REQUEST_CHARACTER_PRIV`).
        *   **Genel Metotlar (Ayrıcalık Verme/Kaldırma):**
            *   `GiveGuildPriv(...)`: Bir loncaya doğrudan ayrıcalık verir (genellikle veritabanından gelen yanıtla çağrılır). Süresi ve değeri sınırlar dahilinde tutar. İlgili lonca bulunursa duyuru yapar ve loglar.
            *   `GiveEmpirePriv(...)`: Bir imparatorluğa doğrudan ayrıcalık verir. Duyuru yapar ve loglar.
            *   `GiveCharacterPriv(...)`: Bir karaktere doğrudan ayrıcalık verir ve loglar.
            *   `RemoveGuildPriv(...)`, `RemoveEmpirePriv(...)`, `RemoveCharacterPriv(...)`: İlgili hedef için belirli bir türdeki ayrıcalığı kaldırır.
        *   **Genel Metotlar (Ayrıcalık Sorgulama):**
            *   `GetPriv(LPCHARACTER ch, BYTE type)`: Bir karakter için belirli bir türdeki efektif ayrıcalık değerini döndürür. Karakterin kendi ayrıcalığını, imparatorluğunun ayrıcalığını ve loncasının ayrıcalığını kontrol eder ve en yüksek pozitif değeri (veya varsa negatif karakter ayrıcalığını) döndürür. `UNIQUE_ITEM_NO_BAD_LUCK_EFFECT` eşyası negatif etkileri engelleyebilir.
            *   `GetPrivByEmpire(BYTE bEmpire, BYTE type)`: Belirli bir imparatorluk için ayrıcalık değerini döndürür.
            *   `GetPrivByGuild(DWORD guild_id, BYTE type)`: Belirli bir lonca için ayrıcalık değerini döndürür.
            *   `GetPrivByCharacter(DWORD pid, BYTE type)`: Belirli bir karakter için ayrıcalık değerini döndürür.
            *   `GetPrivByEmpireEx(...)`, `GetPrivByGuildEx(...)`: Ayrıcalık verilerini (`SPrivEmpireData` veya `SPrivGuildData` işaretçisi) döndürür (değer ve bitiş zamanını içerir).
        *   **Özel Üyeler:**
            *   `m_aakPrivEmpireData[MAX_PRIV_NUM][EMPIRE_MAX_NUM]`: İmparatorluk ayrıcalıklarını tutan 2 boyutlu dizi.
            *   `m_aPrivGuild[MAX_PRIV_NUM]`: Lonca ayrıcalıklarını (tür -> lonca haritası) tutan dizi.
            *   `m_aPrivChar[MAX_PRIV_NUM]`: Karakter ayrıcalıklarını (tür -> karakter haritası) tutan dizi.
*   **Bağlantılı Dosyalar:** `priv_manager.cpp` (uygulama), `constants.h`, `char.h`, `guild.h`.

### `priv_manager.cpp`

*   **Amaç:** `priv_manager.h`'de bildirilen `CPrivManager` sınıfının metotlarını uygular. Karakter, lonca ve imparatorluk ayrıcalıklarının yönetilmesi, veritabanı ile iletişim, değerlerin alınması ve uygulanması işlevlerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yardımcı Fonksiyonlar:**
        *   `GetEmpireName(int priv)`: İmparatorluk ID'sine karşılık gelen yerelleştirilmiş adı döndürür (`c_apszEmpireNames`).
        *   `GetPrivName(int priv)`: Ayrıcalık türü ID'sine karşılık gelen yerelleştirilmiş adı döndürür (`c_apszPrivNames`).
    *   **`CPrivManager::CPrivManager()`:**
        *   İmparatorluk ayrıcalıkları dizisini (`m_aakPrivEmpireData`) sıfırlar.
    *   **Veritabanı İstekleri (`RequestGive*Priv`):**
        *   Verilen parametreleri (tür, değer, süre) sınırlar içinde tutar (`MINMAX`).
        *   İlgili paket yapısını (`TPacketGiveGuildPriv`, `TPacketGiveEmpirePriv`, `TPacketGiveCharacterPriv`) doldurur.
        *   `db_clientdesc->DBPacket()` aracılığıyla ilgili başlıkla (`HEADER_GD_REQUEST_*_PRIV`) veritabanına paketi gönderir.
    *   **Ayrıcalık Verme (`Give*Priv`):**
        *   Genellikle veritabanından gelen yanıt üzerine çağrılır.
        *   Verilen değerleri ve süreyi tekrar sınırlar içinde tutar.
        *   İlgili veri yapısını (`m_aakPrivEmpireData`, `m_aPrivGuild`, `m_aPrivChar`) günceller.
        *   İmparatorluk veya lonca ayrıcalığı ise `SendRestrictedNotice` ile sunucu geneli duyuru yapar (ayrıcalığın adı ve değeriyle birlikte).
        *   Eğer `bLog` bayrağı true ise `LogManager` aracılığıyla işlemi loglar (`GUILD_PRIV`, `EMPIRE_PRIV`, `CHARACTER_PRIV`).
    *   **Ayrıcalık Kaldırma (`Remove*Priv`):**
        *   İlgili veri yapısındaki (harita veya dizi) girişi bulur ve değeri 0, süreyi 0 olarak ayarlar veya haritadan siler.
    *   **Ayrıcalık Sorgulama (`GetPriv*`):**
        *   `GetPriv(ch, type)`: En karmaşık sorgulama fonksiyonudur. Önce karakterin kendi ayrıcalığını (`GetPrivByCharacter`) alır. Eğer bu değer negatifse ve karakterde `UNIQUE_ITEM_NO_BAD_LUCK_EFFECT` yoksa doğrudan bu negatif değeri döndürür. Aksi takdirde, karakter ayrıcalığı, genel imparatorluk ayrıcalığı (imparatorluk 0), karakterin kendi imparatorluğunun ayrıcalığı ve (varsa) loncasının ayrıcalığı arasındaki en yüksek pozitif değeri hesaplayıp döndürür.
        *   `GetPrivByEmpire(bEmpire, type)`: `GetPrivByEmpireEx` kullanarak ilgili `SPrivEmpireData`'yı alır ve içindeki `m_value`'yu döndürür. Bulamazsa 0 döner.
        *   `GetPrivByGuild(guild_id, type)`: `m_aPrivGuild` haritasından ilgili lonca ve tür için girişi arar. Bulursa `value` değerini, bulamazsa 0 döner.
        *   `GetPrivByCharacter(pid, type)`: `m_aPrivChar` haritasından ilgili karakter ve tür için girişi arar. Bulursa değeri, bulamazsa 0 döner.
        *   `GetPrivByEmpireEx(...)`, `GetPrivByGuildEx(...)`: İlgili veri yapısının (`SPrivEmpireData` veya `SPrivGuildData`) işaretçisini döndürür, bulunamazsa `NULL` döner.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `priv_manager.h`, `char.h`, `desc_client.h`, `guild.h`, `guild_manager.h`, `unique_item.h`, `utils.h`, `log.h`.

### `private_shop_manager.h`

*   **Amaç:** Oyuncuların kurduğu kişisel pazar tezgahlarını (private shop) yöneten `CPrivateShopManager` singleton sınıfını tanımlar. Pazar kurma, kapatma, eşya ekleme/çıkarma, alışveriş yapma ve pazar arama gibi işlevlerin arayüzünü sağlar.
*   **Temel İşlevler/İçerik:**
    *   **Typedef'ler:**
        *   `TPrivateShopMap` (std::unordered_map<DWORD, std::unique_ptr<CPrivateShop>>): Oyuncu PID'lerini `CPrivateShop` nesnelerine (unique_ptr ile) eşler.
        *   `TPrivateShopVIDMap` (std::unordered_map<DWORD, LPPRIVATE_SHOP>): Pazar tezgahı VID'lerini `CPrivateShop` işaretçilerine eşler.
        *   `TMarketPriceMap` (std::unordered_map<DWORD, TItemPrice>): (Kullanılmıyor gibi görünüyor) Muhtemelen eşya ID'lerini pazar fiyatlarıyla eşleştirmek için tasarlanmış.
        *   `TItemList` (std::unordered_set<LPITEM>): Pazar arama için kullanılan eşya işaretçileri seti.
        *   `TSubTypeItemMap` (std::unordered_map<BYTE, TItemList>): Eşya alt türlerini `TItemList`'e eşler (arama için).
        *   `TTypeItemMap` (std::unordered_map<BYTE, TSubTypeItemMap>): Eşya ana türlerini `TSubTypeItemMap`'e eşler (arama için).
    *   **`CPrivateShopManager` Sınıfı (Singleton):**
        *   `Destroy()`: Tüm özel pazar nesnelerini temizler.
        *   `CreatePrivateShop(DWORD dwPID)`: Belirtilen oyuncu için yeni bir `CPrivateShop` nesnesi oluşturur ve haritalara ekler.
        *   `GetPrivateShop(DWORD dwPID)`, `GetPrivateShopByOwnerName(const char* c_szOwnerName)`, `GetPrivateShopByVID(DWORD dwVID)`: Özel pazarları sahibinin PID'si, adı veya pazarın VID'si ile bulur.
        *   `DeletePrivateShop(DWORD dwPID)`: Belirtilen oyuncuya ait özel pazarı sistemden siler.
        *   `AllocVID()`: Yeni bir pazar tezgahı için benzersiz bir VID (Virtual ID) üretir.
        *   `BuildPrivateShop(...)`: Bir oyuncunun özel pazar kurma isteğini işler. `CPrivateShop` nesnesi oluşturur, eşyaları hazırlar ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_CREATE`) pazar oluşturma isteği gönderir.
        *   `BuildPrivateShopResult(...)`: Veritabanından gelen pazar kurma sonucunu işler. Başarılıysa eşyaları oyuncudan pazara aktarır, başarısızsa eşyaları iade eder ve pazarı siler.
        *   `ClosePrivateShop(LPCHARACTER pShopOwner)`: Bir oyuncunun pazarını kapatma isteğini işler. Sahibinin pazara yakınlığını kontrol eder. Eşyaları sahibine geri aktarır (`TransferItems`) ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_DELETE`) pazar silme isteği gönderir.
        *   `SpawnPrivateShop(...)`: Sunucu başlangıcında veya başka bir nedenle veritabanından yüklenen pazar bilgilerini kullanarak oyunda pazar tezgahını oluşturur.
        *   `DespawnPrivateShop(DWORD dwPID)`: Belirtilen pazarı oyundan kaldırır ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_DESPAWN`) güncelleme gönderir.
        *   `StopShopping(LPCHARACTER pShopViewer)`: Bir oyuncunun bir pazara bakmayı bırakmasını sağlar.
        *   `ItemCheckin(LPCHARACTER pOwner, const TPlayerPrivateShopItem* c_pShopItem)`: Oyuncunun kişisel pazar depolama alanına eşya eklemesini işler. Eşyayı oyuncudan alır ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_ITEM_CHECKIN_UPDATE`) check-in güncellemesi gönderir.
        *   `ItemCheckout(LPCHARACTER pOwner, WORD wSrcPos, TItemPos TDstPos)`: Oyuncunun kişisel pazar depolama alanından eşya çekmesini işler. Eşyayı oyuncuya verir ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_ITEM_CHECKOUT_UPDATE`) check-out güncellemesi gönderir.
        *   `ItemTransaction(LPCHARACTER pCustomer, TPlayerPrivateShopItem* c_pShopItem)`: Bir müşterinin bir pazardan eşya satın alma işlemini işler. Kontrolleri yapar (yer, para vb.) ve başarılıysa eşyayı müşteriye verir, ücreti düşer ve veritabanına (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_BUY`) satın alma bilgisi gönderir.
        *   `SendItemTransaction(...)`, `SendItemTransactionFailedResult(...)`: Satın alma işleminin sonucunu veritabanına iletmek için yardımcı fonksiyonlar.
        *   `SendItemTransfer(const TPlayerItem* c_pItem)`: (Kullanımı net değil, muhtemelen P2P eşya transferi için olabilir).
        *   `SendItemExpire(LPITEM pItem)`: Süresi dolan bir pazar eşyasının bilgisini veritabanına gönderir (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_ITEM_EXPIRE`).
        *   `SendShopSearchUpdate(DWORD dwShopID, BYTE bState, int iSpecificItemPos)`: Pazar arama sistemine bir pazarın veya içindeki belirli bir eşyanın durumu hakkında güncelleme gönderir (hem istemcilere `SUBHEADER_GC_PRIVATE_SHOP_SEARCH_UPDATE` hem de P2P ile `HEADER_GG_PRIVATE_SHOP_ITEM_SEARCH_UPDATE`).
        *   `AddSearchItem(LPITEM pItem)`, `RemoveSearchItem(LPITEM pItem)`: Pazar arama için kullanılan dahili eşya listesini (`m_map_searchItem`) günceller.
        *   `SearchItem(LPDESC pDesc, TPrivateShopSearchFilter& rFilter, bool bUseFilter, DWORD dwCustomerPID)`: Gelen filtrelere göre pazarlardaki eşyaları arar ve sonuçları (`TPrivateShopSearchData` listesi) istek yapan istemciye (`SUBHEADER_GC_PRIVATE_SHOP_SEARCH_RESULT`) veya (P2P için) veritabanına (`HEADER_GG_PRIVATE_SHOP_ITEM_SEARCH_RESULT`) gönderir.
        *   **Özel Üyeler:**
            *   `m_dwVIDCount`: Pazar tezgahları için VID üretici sayacı.
            *   `m_map_privateShop`: Aktif özel pazarları (sahip PID -> Pazar nesnesi) tutar.
            *   `m_map_privateShopVID`: Aktif özel pazarları (Pazar VID -> Pazar nesnesi) tutar.
            *   `m_map_searchItem`: Pazar araması için eşyaları tür/alt türe göre gruplandırılmış şekilde tutar.
*   **Bağlantılı Dosyalar:** `private_shop_manager.cpp` (uygulama), `private_shop.h`, `item.h`, `packet.h`.

### `private_shop_manager.cpp`

*   **Amaç:** `private_shop_manager.h` dosyasında bildirilen `CPrivateShopManager` sınıfının metotlarını uygular. Özel pazar oluşturma, silme, eşya işlemleri (alım, satım, check-in/out), pazar arama ve veritabanı/P2P iletişimi gibi tüm yönetim işlevlerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Pazar Yönetimi:**
        *   `CreatePrivateShop`: Yeni bir `CPrivateShop` nesnesi oluşturur, `AllocVID` ile VID atar ve `m_map_privateShop` (PID anahtarlı) ile `m_map_privateShopVID` (VID anahtarlı) haritalarına ekler.
        *   `GetPrivateShop`, `GetPrivateShopByOwnerName`, `GetPrivateShopByVID`: İlgili haritaları kullanarak pazarları bulur.
        *   `DeletePrivateShop`: Pazarı her iki haritadan da siler.
    *   **Pazar Kurma (`BuildPrivateShop`, `BuildPrivateShopResult`):**
        *   `BuildPrivateShop`: `CreatePrivateShop` çağırır, gelen eşya listesini (`c_vec_shopItem`) `TPlayerPrivateShopItem` formatına dönüştürür, `CPrivateShop` nesnesini temel bilgilerle (`SetID`, `SetVnum`, `SetOwnerName` vb.) ayarlar ve eşyaları `Initialize` ile pazara yükler. Ardından `TPrivateShop` ve `TPlayerPrivateShopItem` verilerini içeren bir paketle (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_CREATE`) veritabanına pazar oluşturma isteği gönderir.
        *   `BuildPrivateShopResult`: Veritabanından gelen yanıta göre işlem yapar. Başarılıysa eşyaları oyuncudan alır (`RemoveFromCharacter`), pazara bağlar (`BindPrivateShop`), sahibinin pazar eşya listesini (`SetPrivateShopItem`) günceller ve pazarı `Show` ile görünür yapar. Başarısızsa, eşyaların pazar bağını koparır ve `DeletePrivateShop` ile pazarı siler.
    *   **Pazar Kapatma (`ClosePrivateShop`):**
        *   Sahibinin pazara yakın olup olmadığını kontrol eder.
        *   `pPrivateShop->TransferItems(pOwner)` ile eşyaları sahibine geri aktarmaya çalışır.
        *   Başarılı olursa, pazarı görüntüleyen diğer oyuncuların pencerelerini kapatır (`CleanShopViewers`), veritabanına silme isteği (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_DELETE`) gönderir ve `DeletePrivateShop` ile pazarı siler.
    *   **Pazar Yükleme/Kaldırma (`SpawnPrivateShop`, `DespawnPrivateShop`):**
        *   `SpawnPrivateShop`: Veritabanından gelen pazar ve eşya bilgileriyle `CreatePrivateShop` çağırır, `Initialize` ile eşyaları yükler ve `Show` ile pazarı görünür yapar.
        *   `DespawnPrivateShop`: Veritabanına kaldırma isteği (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_DESPAWN`) gönderir ve `DeletePrivateShop` çağırır.
    *   **Eşya İşlemleri:**
        *   `ItemCheckin`: Eşyayı oyuncudan alır (`RemoveItem`), sahibinin pazar eşya listesini günceller (`SetPrivateShopItem`) ve veritabanına check-in güncellemesi (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_ITEM_CHECKIN_UPDATE`) gönderir.
        *   `ItemCheckout`: Eşyayı bulur (yoksa oluşturur), oyuncuya eklemeye çalışır (`AddToCharacter`). Başarılı olursa, sahibinin pazar eşya listesinden çıkarır (`RemovePrivateShopItem`) ve veritabanına check-out güncellemesi (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_ITEM_CHECKOUT_UPDATE`) gönderir.
        *   `ItemTransaction`: Müşterinin eşyayı alacak yeri ve parası olup olmadığını kontrol eder. Başarılıysa eşyayı müşteriye verir (`AddToCharacter`), parayı düşer (`PointChange`), pazardan eşyayı kaldırır (`RemoveItem` - eğer pazar bu core'daysa) ve veritabanına satın alma bilgisi (`HEADER_GD_PRIVATE_SHOP`, `PRIVATE_SHOP_GD_SUBHEADER_BUY`) gönderir.
    *   **Pazar Arama (`AddSearchItem`, `RemoveSearchItem`, `SearchItem`, `FilterItem`):**
        *   `AddSearchItem`/`RemoveSearchItem`: Bir eşya pazara eklendiğinde veya kaldırıldığında, hızlı arama için kullanılan `m_map_searchItem` haritasını (Türe -> Alt Türe -> Eşya Seti) günceller.
        *   `FilterItem`: Bir eşyanın verilen arama filtresine (`TPrivateShopSearchFilter`) uyup uymadığını kontrol eder (VNUM, tür, alt tür, seviye, statü, anti-flag, bonuslar vb.).
        *   `SearchItem`: Gelen filtreye göre pazarlardaki eşyaları arar ve sonuçları (`TPrivateShopSearchData` listesi) istek yapan istemciye (`SUBHEADER_GC_PRIVATE_SHOP_SEARCH_RESULT`) veya (P2P için) veritabanına (`HEADER_GG_PRIVATE_SHOP_ITEM_SEARCH_RESULT`) gönderir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `private_shop_manager.h`, `private_shop.h`, `private_shop_util.h`, `char.h`, `char_manager.h`, `desc_client.h`, `mob_manager.h`, `config.h`, `db.h`, `item_manager.h`, `item.h`, `utils.h`, `entity.h`, `sectree_manager.h`, `p2p.h`, `buffer_manager.h`, `desc_manager.h`, `DragonSoul.h`, `log.h`. 

### `private_shop_util.h`

*   **Amaç:** Özel pazar (private shop) sistemi için çeşitli yardımcı fonksiyonların ve veri yapısı dönüştürme fonksiyonlarının bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   **`format_number<T>(T value)` (Şablon Fonksiyonu):** Verilen bir sayısal değeri, binlik ayraçları (nokta) ile formatlayarak string olarak döndürür.
    *   **Harici Fonksiyon Bildirimleri:**
        *   `CanBuildPrivateShop(LPCHARACTER ch)`: Karakterin bulunduğu haritada özel pazar kurup kuramayacağını kontrol eder.
        *   `CheckTradeWindows(LPCHARACTER ch)`: Karakterin herhangi bir ticaret penceresinin (takas, depo, kendi pazarı vb.) açık olup olmadığını kontrol eder.
        *   `GetEmptyInventory(LPCHARACTER pChar, LPITEM pItem)`: Karakterin envanterinde (normal, ejderha taşı veya `__SPECIAL_INVENTORY_SYSTEM__` tanımlıysa ilgili özel envanterler) belirtilen eşya için boş yer olup olmadığını kontrol eder ve uygun pozisyonu döndürür.
        *   **`CopyItemData` Overload'ları:**
            *   `CopyItemData(LPITEM pItem, TPlayerItem& rTargetTable)`: `LPITEM`'dan `TPlayerItem`'a eşya verilerini kopyalar.
            *   `CopyItemData(LPITEM pItem, TPlayerPrivateShopItem& rTargetTable)`: `LPITEM`'dan `TPlayerPrivateShopItem`'a eşya verilerini kopyalar (pazar fiyatı, check-in zamanı dahil).
            *   `CopyItemData(LPITEM pItem, TPrivateShopItemData& rTargetTable)`: `LPITEM`'dan `TPrivateShopItemData`'ya eşya verilerini kopyalar.
            *   `CopyItemData(LPITEM pItem, TPrivateShopSearchData& rTargetTable)`: `LPITEM`'dan pazar arama sonuçları için kullanılan `TPrivateShopSearchData`'ya veri kopyalar (pazar ID'si, sahip adı dahil).
            *   `CopyItemData(const TPlayerPrivateShopItem& rSourceTable, LPITEM pItem, LPCHARACTER pOwner = nullptr)`: `TPlayerPrivateShopItem`'dan `LPITEM`'a eşya verilerini kopyalar.
            *   `CopyItemData(const TPlayerPrivateShopItem& rSourceTable, TPrivateShopItemData& rTargetTable)`: `TPlayerPrivateShopItem`'dan `TPrivateShopItemData`'ya veri kopyalar.
*   **Bağlantılı Dosyalar:** `../../common/length.h`, `../../common/tables.h`, `packet.h`.

### `private_shop_util.cpp`

*   **Amaç:** `private_shop_util.h` dosyasında bildirilen özel pazar yardımcı fonksiyonlarını uygular.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CanBuildPrivateShop(ch)`:**
        *   `__PRIVATE_SHOP_BUILD_MARKET_PLACE__` makrosu tanımlıysa sadece 392 ID'li haritada (genellikle özel pazar haritası) pazar kurulmasına izin verir.
        *   Tanımlı değilse, A1 (Shinsoo), B1 (Chunjo), C1 (Jinno) imparatorluklarının birinci köy haritalarında pazar kurulmasına izin verir.
        *   Diğer haritalarda `false` döner.
    *   **`CheckTradeWindows(ch)`:**
        *   Karakterin takas (`GetExchange`), depo (`IsOpenSafebox`), kendi pazarı (`GetMyShop`), NPC pazarı (`GetShopOwner`), küp (`IsCubeOpen`) veya lonca aracılığıyla item basma (`IsRefineThroughGuild`) pencerelerinden herhangi biri açıksa `false` döner, aksi halde `true` döner.
    *   **`GetEmptyInventory(pChar, pItem)`:**
        *   Eşyanın türüne göre (`IsDragonSoul`, `IsSkillBook` vb. - `__SPECIAL_INVENTORY_SYSTEM__` makrosuna bağlı olarak) ilgili envanterde boş yer arar (`GetEmptyDragonSoulInventory`, `GetEmptySkillBookInventory` vb.) ve bulursa pozisyonu, bulamazsa -1 döndürür.
    *   **`CopyItemData` Fonksiyonları:**
        *   Bu fonksiyonlar, kaynak yapıdaki (`LPITEM` veya `TPlayerPrivateShopItem`) tüm temel eşya bilgilerini (ID, sahip, VNUM, sayı, pozisyon, pencere, soketler, efsunlar) hedef yapıya (`TPlayerItem`, `TPlayerPrivateShopItem`, `TPrivateShopItemData`, `TPrivateShopSearchData` veya `LPITEM`) kopyalar.
        *   Kopyalama sırasında `__CHANGE_LOOK_SYSTEM__`, `__REFINE_ELEMENT_SYSTEM__`, `__SET_ITEM__`, `__ITEM_APPLY_RANDOM__`, `__ITEM_VALUE10__` gibi derleme zamanı makrolarına bağlı olarak ek veriler (görünüm VNUM'u, element, set değeri, rastgele efsunlar, min/max değerler) de kopyalanır.
        *   `LPITEM`'dan kopyalama yaparken, eşyanın sahibi karakterse karakterin PID'sini, bir özel pazara aitse pazarın ID'sini sahip olarak atar.
        *   `TPlayerPrivateShopItem`'dan `LPITEM`'a kopyalarken, eşyanın pazar fiyatını, check-in zamanını ve diğer pazarla ilgili özelliklerini de `LPITEM` nesnesine ayarlar.
    *   **Not:** Dosyanın başında yorum satırı içinde özel pazar sistemiyle ilgili çeşitli bilgilendirme/hata mesajlarının İngilizce metinleri listelenmiştir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `private_shop_util.h`, `packet.h`, `item.h`, `char.h`, `private_shop.h`, `private_shop_manager.h`.

### `private_shop.h`

*   **Amaç:** Tek bir oyuncu tarafından kurulan özel pazar tezgahını (`CPrivateShop`) temsil eden sınıfı tanımlar. Bu sınıf, `CEntity`'den miras alır ve pazarın eşyalarını, görüntüleyenlerini ve genel durumunu yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`CPrivateShop` Sınıfı (`CEntity` miraslı):**
        *   **Yapıcı/Yıkıcı:** Pazar nesnesini başlatır (`CEntity::Initialize(ENTITY_PRIVATE_SHOP)`) ve temizler (görüntüleyenleri ve eşyaları yok eder).
        *   `Show(long lX, long lY, long lZ, long lMapIndex)`: Pazarı belirtilen koordinatlarda ve haritada görünür hale getirir. `SECTREE_MANAGER` kullanarak entity yönetimini yapar.
        *   `Initialize(const std::vector<TPlayerPrivateShopItem>& c_vec_shopItem, bool bRespawn = false)`: Pazarı verilen eşya listesiyle başlatır. Eşyaları oluşturur (eğer `bRespawn` true ise) veya var olanları bulur, pazar içine yerleştirir (`CGrid` kullanarak pozisyon kontrolü yapar) ve `m_map_shopItem`'a ekler. `CPrivateShopManager::Instance().AddSearchItem` ile arama listesine ekler.
        *   `TransferItems(LPCHARACTER pShopOwner)`: Pazardaki tüm eşyaları sahibinin envanterine geri aktarmaya çalışır. Envanterde yeterli yer olup olmadığını kontrol eder (`CGrid` ve `GetEmptyInventory` kullanarak). Başarılı olursa eşyaları aktarır ve veritabanına güncelleme gönderir.
        *   `GetItemCount()`, `GetItemContainer()`: Pazardaki eşya sayısını ve eşya haritasını (`m_map_shopItem`) döndürür.
        *   `CleanItems()`: Pazardaki tüm eşyaları temizler (arama listesinden çıkarır ve yok eder).
        *   `GetItem(WORD wPos)`: Belirtilen pozisyondaki eşyayı döndürür.
        *   `RemoveItem(WORD wPos)`: Belirtilen pozisyondaki eşyayı pazardan kaldırır, arama listesinden çıkarır ve görüntüleyenlere güncelleme paketi (`SUBHEADER_GC_PRIVATE_SHOP_REMOVE_ITEM`) gönderir. Pazar arama güncellemesi (`SendShopSearchUpdate`) de yapılır.
        *   `ItemCheckin(const TPlayerPrivateShopItem& c_rShopItem)`: Pazara yeni bir eşya ekler (genellikle oyuncunun pazar depolama alanından). Eşyayı oluşturur, `m_map_shopItem`'a ekler, arama listesine ekler ve görüntüleyenlere güncelleme paketi (`SUBHEADER_GC_PRIVATE_SHOP_ADD_ITEM`) gönderir.
        *   `ChangeItemPrice(WORD wPos, long long llGold, DWORD dwCheque)`: Bir eşyanın fiyatını günceller ve görüntüleyenlere paket (`SUBHEADER_GC_PRIVATE_SHOP_ITEM_PRICE_CHANGE`) gönderir.
        *   `MoveItem(WORD wPos, WORD wChangePos)`: Bir eşyayı pazar içinde farklı bir pozisyona taşır ve görüntüleyenlere paket (`SUBHEADER_GC_PRIVATE_SHOP_ITEM_MOVE`) gönderir.
        *   `ChangeTitle(const char* c_szTitle)`: Pazarın başlığını değiştirir ve etraftaki oyunculara güncelleme paketi (`SUBHEADER_GC_PRIVATE_SHOP_TITLE`) gönderir.
        *   **Getter/Setter Metotları:** `SetVID`, `GetVID`, `SetID`, `GetID`, `SetVnum`, `GetVnum`, `SetOwnerName`, `GetOwnerName`, `SetTitle`, `GetTitle`, `SetTitleType`, `GetTitleType`, `SetState`, `GetState`, `SetPageCount`, `GetPageCount`, `SetClosing`, `IsClosing` gibi metotlarla pazarın çeşitli özelliklerine erişim ve değişiklik imkanı sunar.
        *   `AddShopViewer(LPCHARACTER pShopViewer)`: Bir oyuncuyu pazarı görüntüleyenler listesine (`m_set_shopViewer`) ekler ve oyuncuya pazar bilgilerini içeren başlangıç paketini (`SUBHEADER_GC_PRIVATE_SHOP_START`) gönderir.
        *   `RemoveShopViewer(LPCHARACTER pShopViewer)`: Bir oyuncuyu görüntüleyenler listesinden çıkarır ve oyuncuya pazarın kapandığına dair paket (`SUBHEADER_GC_PRIVATE_SHOP_END`) gönderir.
        *   `CleanShopViewers()`: Tüm görüntüleyenleri listeden çıkarır ve ilgili paketleri gönderir.
        *   `BroadcastPacket(const void* c_pData, int iSize)`: Pazardaki tüm görüntüleyenlere belirtilen paketi gönderir.
        *   **Korumalı Metotlar (CEntity'den override):**
            *   `EncodeInsertPacket(LPENTITY pEntity)`: Bir entity (oyuncu) pazarın görüş alanına girdiğinde gönderilecek olan pazar oluşturma paketini (`SUBHEADER_GC_PRIVATE_SHOP_ADD_ENTITY`) hazırlar.
            *   `EncodeRemovePacket(LPENTITY pEntity)`: Bir entity pazarın görüş alanından çıktığında gönderilecek olan pazar silme paketini (`SUBHEADER_GC_PRIVATE_SHOP_DEL_ENTITY`) hazırlar.
        *   **Özel Üyeler:**
            *   `m_map_shopItem` (std::map<WORD, LPITEM>): Pazar pozisyonlarını (`WORD`) `LPITEM` işaretçileriyle eşleştiren harita.
            *   `m_set_shopViewer` (boost::unordered_set<LPCHARACTER>): Pazarı o an görüntüleyen karakterlerin (müşterilerin) listesi.
            *   `m_dwID`: Pazar sahibinin Oyuncu ID'si.
            *   `m_dwVID`: Pazarın sanal ID'si (entity ID).
            *   `m_dwVnum`: Pazarın görünümünü belirleyen mob VNUM'u.
            *   `m_strOwnerName`: Pazar sahibinin adı.
            *   `m_strTitle`: Pazarın başlığı.
            *   `m_bTitleType`: Pazar başlığının türü/stili.
            *   `m_bState`: Pazarın anlık durumu (açık, kapalı, düzenleniyor vb.).
            *   `m_bPageCount`: Pazarın sahip olduğu sayfa sayısı.
            *   `m_bIsClosing`: Pazarın kapatılma sürecinde olup olmadığını belirten bayrak.
*   **Bağlantılı Dosyalar:** `private_shop.cpp` (uygulama), `../../common/tables.h`, `../../common/length.h`, `../../libgame/include/grid.h`, `entity.h`, `item.h`.

### `private_shop.cpp`

*   **Amaç:** `private_shop.h` dosyasında bildirilen `CPrivateShop` sınıfının metotlarını uygular. Bireysel bir özel pazarın oluşturulması, eşya yönetimi, oyuncu etkileşimleri (görüntüleme, satın alma - dolaylı olarak `CPrivateShopManager` üzerinden), durum değişiklikleri ve ağ paketlerinin gönderilmesi gibi işlevleri içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`CPrivateShop::CPrivateShop()`):**
        *   `CEntity::Initialize(ENTITY_PRIVATE_SHOP)` çağırır.
        *   Görüntüleyen listesini (`m_set_shopViewer`) ve eşya haritasını (`m_map_shopItem`) temizler.
        *   Pazar özelliklerini (ID, VID, VNUM, sahip adı, başlık, durum vb.) varsayılan değerlere ayarlar.
    *   **Yıkıcı (`CPrivateShop::~CPrivateShop()`):**
        *   `CleanShopViewers()` ile tüm görüntüleyenlerin bağlantısını keser.
        *   Pazardaki tüm eşyaları `CPrivateShopManager`'ın arama listesinden çıkarır (`RemoveSearchItem`) ve `M2_DESTROY_ITEM` ile yok eder.
        *   `CEntity::Destroy()` çağırır ve entity'yi sectree'den kaldırır.
    *   **`Show(lX, lY, lZ, lMapIndex)`:**
        *   Pazarı oyun dünyasında görünür hale getirir. `SECTREE_MANAGER` üzerinden doğru sectree'yi bulur, pazarın pozisyonunu ve harita indeksini ayarlar. Eğer sectree değişmişse veya ilk defa ekleniyorsa, eski sectree'den çıkarıp yenisine ekler (`EncodeInsertPacket`, `InsertEntity`, `UpdateSectree`). Sadece pozisyonu güncelleniyorsa `ViewReencode` çağırır.
    *   **`Initialize(c_vec_shopItem, bRespawn)`:**
        *   Pazar eşyalarını yükler. Her pazar sayfası için bir `CGrid` oluşturarak eşyaların doğru yerleştirilip yerleştirilemeyeceğini kontrol eder.
        *   Eğer `bRespawn` true ise (genellikle sunucu başlangıcında veritabanından yüklenirken), eşyaları `ITEM_MANAGER::instance().CreateItem` ile yeniden oluşturur.
        *   Aksi halde (`bRespawn` false, yani oyuncu manuel pazar kuruyorsa), eşyaları `ITEM_MANAGER::Instance().Find` ile oyuncunun envanterinden bulur.
        *   `CopyItemData` (util dosyasından) ile eşya bilgilerini `LPITEM` nesnesine aktarır.
        *   Eşyaları `m_map_shopItem`'a ekler ve `CPrivateShopManager::Instance().AddSearchItem` ile pazar arama sistemine kaydeder.
    *   **`TransferItems(pShopOwner)`:**
        *   Pazar kapatılırken eşyaları sahibine geri verir.
        *   Pazardaki eşyaları türlerine göre (normal envanter, ejderha taşı, özel envanterler) ayırır.
        *   Sahibinin envanterini `CGrid` kullanarak temsil eder, mevcut eşyaları bu grid'e yerleştirir.
        *   Ardından pazar eşyalarını bu grid'lerde boş yer bularak yerleştirmeye çalışır. Yer bulunamazsa hata verir ve `false` döner.
        *   Tüm eşyalar için yer bulunursa, `GetEmptyInventory` ile kesin pozisyonu alır, eşyayı `AddToCharacter` ile sahibine verir, pazardan (`RemoveItem`) ve sahibinin pazar eşya listesinden (`RemovePrivateShopItem`) çıkarır, loglar ve `CPrivateShopManager` aracılığıyla transfer bilgisini gönderir.
    *   **Eşya Yönetimi (`GetItem`, `RemoveItem`, `ItemCheckin`, `ChangeItemPrice`, `MoveItem`):**
        *   `GetItem`: Pozisyona göre eşya bulur.
        *   `RemoveItem`: Eşyayı `m_map_shopItem`'dan siler, `CPrivateShopManager`'dan arama listesinden çıkarır. Eğer eşya hala pazara bağlıysa (P2P işlemiyle alınmamışsa) `M2_DESTROY_ITEM` ile yok eder. Görüntüleyenlere `SUBHEADER_GC_PRIVATE_SHOP_REMOVE_ITEM` paketi gönderir ve pazar arama durumunu günceller.
        *   `ItemCheckin`: Gelen `TPlayerPrivateShopItem` verisiyle yeni bir `LPITEM` oluşturur, pazara bağlar, `m_map_shopItem`'a ekler, arama listesine ekler ve görüntüleyenlere `SUBHEADER_GC_PRIVATE_SHOP_ADD_ITEM` paketi gönderir.
        *   `ChangeItemPrice`: Eşyanın fiyatını günceller, loglar ve görüntüleyenlere `SUBHEADER_GC_PRIVATE_SHOP_ITEM_PRICE_CHANGE` paketi gönderir.
        *   `MoveItem`: Eşyayı pazar içinde farklı bir slota taşır, loglar ve görüntüleyenlere `SUBHEADER_GC_PRIVATE_SHOP_ITEM_MOVE` paketi gönderir.
    *   **Pazar Özellikleri (`ChangeTitle`, `SetState`):**
        *   `ChangeTitle`: Pazar başlığını günceller ve etraftaki oyunculara `SUBHEADER_GC_PRIVATE_SHOP_TITLE` paketi gönderir.
        *   `SetState`: Pazarın durumunu (açık, kapalı vb.) ayarlar. Eğer durum `STATE_CLOSED` ise `CPrivateShopManager` üzerinden pazarı siler. Aksi halde, durumu günceller ve pazar arama sistemine bilgi verir. Görüntüleyen oyuncuların pazar durumunu günceller.
    *   **Görüntüleyici Yönetimi (`AddShopViewer`, `RemoveShopViewer`, `CleanShopViewers`, `BroadcastPacket`):**
        *   `AddShopViewer`: Bir oyuncuyu görüntüleyenler listesine ekler, oyuncuya pazar bilgilerini (`SUBHEADER_GC_PRIVATE_SHOP_START` paketi) gönderir.
        *   `RemoveShopViewer`: Bir oyuncuyu listeden çıkarır ve `SUBHEADER_GC_PRIVATE_SHOP_END` paketi gönderir.
        *   `CleanShopViewers`: Tüm görüntüleyenleri çıkarır.
        *   `BroadcastPacket`: Tüm aktif görüntüleyenlere belirtilen paketi yollar.
    *   **Entity Paketleri (`EncodeInsertPacket`, `EncodeRemovePacket`):**
        *   `EncodeInsertPacket`: Pazarın bir oyuncunun görüş alanına girdiğinde gönderilecek `SUBHEADER_GC_PRIVATE_SHOP_ADD_ENTITY` paketini oluşturur.
        *   `EncodeRemovePacket`: Pazarın bir oyuncunun görüş alanından çıktığında gönderilecek `SUBHEADER_GC_PRIVATE_SHOP_DEL_ENTITY` paketini oluşturur.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `private_shop.h`, `private_shop_manager.h`, `private_shop_util.h`, `char.h`, `char_manager.h`, `desc.h`, `item.h`, `item_manager.h`, `log.h`, `p2p.h`, `sectree_manager.h`, `DragonSoul.h`, `config.h`, `buffer_manager.h`, `desc_client.h`, `../../libgame/include/grid.h`.

---

</rewritten_file> 