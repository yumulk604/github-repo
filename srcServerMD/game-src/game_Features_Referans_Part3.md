# Metin2 Oyun Sunucusu - Özellikler Referansı Part 3 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans_Part2.md`](game_Features_Referans_Part2.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) belirli oyun özellikleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`MailBox.cpp`](#mailboxcpp)
*   [`MailBox.h`](#mailboxh)
*   [`marriage.cpp`](#marriagecpp)
*   [`marriage.h`](#marriageh)
*   [`MeleyLair.cpp`](#meleylaircpp)
*   [`MeleyLair.h`](#meleylairh)
*   [`minigame.h`](#minigameh)
*   [`minigame.cpp`](#minigamecpp)
*   [`minigame_catch_king.cpp`](#minigame_catch_kingcpp)
*   [`mining.h`](#miningh)
*   [`mining.cpp`](#miningcpp)
*   [`monarch.h`](#monarchh)
*   [`monarch.cpp`](#monarchcpp)
*   [`mount_up_grade.h`](#mount_up_gradeh)
*   [`mount_up_grade.cpp`](#mount_up_gradecpp)
*   [`over9refine.h`](#over9refineh)
*   [`over9refine.cpp`](#over9refinecpp)
*   [`TempleOchao.h`](#templeochaoh)
*   [`TempleOchao.cpp`](#templechaocpp)
*   [`OchaoTemple.cpp (Alternatif Versiyon)`](#ochaotemplecpp-alternatif-versiyon)

---

### `MailBox.h`

*   **Amaç:** (`__MAILBOX__` tanımlıysa) Oyuncular arası Posta Kutusu sistemini yöneten `CMailBox` sınıfını ve ilgili sabitleri (enum'lar) tanımlar. Mesaj/eşya/para gönderme, alma, okuma, silme gibi işlemler için arayüzü bildirir.
*   **Temel İşlevler/İçerik:**
    *   **`CMailBox` Sınıfı:**
        *   **Yapıcı/Yıkıcı:** `CMailBox` nesnesini oluşturur (veritabanından yüklenen verilerle), sahibi (`Owner`) ayarlar ve yok ederken sunucuya bildirir.
        *   **Statik Metotlar:**
            *   `Open(LPCHARACTER ch)`: Bir karakter için posta kutusunu açma işlemini başlatır (DB'den veri ister).
            *   `Create(LPCHARACTER ch, const TMailBoxTable* pTable, const WORD Size)`: DB'den gelen verilerle karakter için `CMailBox` nesnesini oluşturur.
            *   `UnreadData(LPCHARACTER ch)`: Okunmamış posta sayısını DB'den sorgular.
            *   `ResultUnreadData(LPCHARACTER ch, TMailBoxRespondUnreadData* data)`: Okunmamış posta sorgusunun sonucunu istemciye gönderir.
            *   `SendGMMail(...)`: Belirtilen oyuncuya GM adına posta gönderir.
        *   **Üye Metotları:**
            *   `ServerProcess`: İstemciye posta kutusuyla ilgili durum/sonuç paketleri gönderir (`HEADER_GC_MAILBOX_PROCESS`).
            *   `Write`: Başka bir oyuncuya mesaj, eşya, yang ve won içeren bir posta gönderir.
            *   `CheckPlayer`: Posta gönderilmeden önce hedef oyuncunun varlığını ve posta kutusu durumunu DB'den kontrol eder.
            *   `CheckPlayerResult`: `CheckPlayer` sorgusunun sonucunu işler ve istemciye bildirir.
            *   `AddData`: Bir postanın içeriğini (mesaj, gönderen, eşya vb.) istemciye gönderir ve postayı okunmuş olarak işaretler.
            *   `GetAllItems`: Tüm postalardaki alınabilir eşya/para/won'u almaya çalışır.
            *   `GetItem`: Belirli bir postadaki eşya/para/won'u alır.
            *   `DeleteAllMails`: İçeriği alınmış tüm postaları silmeye çalışır.
            *   `DeleteMail`: Belirli bir postayı siler (içeriği boşsa).
            *   `GetMailVec`: Posta verilerini içeren vektörün referansını döndürür.
    *   **Enum Sabitleri:**
        *   `EMAILBOX_CG`: İstemciden sunucuya gönderilen posta kutusu komut başlıkları.
        *   `EMAILBOX_GC`: Sunucudan istemciye gönderilen posta kutusu komut başlıkları ve durum kodları.
        *   `EMAILBOX_POST_*`: Posta gönderme, alma, silme işlemlerinin sonuçlarını belirten alt kodlar.
    *   **Üyeler:**
        *   `vecMailBox` (`MailVec` - `std::vector<TMailBoxTable>`): Karakterin tüm posta verilerini tutan vektör.
        *   `Owner` (`LPCHARACTER`): Posta kutusunun sahibi olan karakter.
*   **Bağlantılı Dosyalar:** `MailBox.cpp` (uygulama), `../../common/length.h`, `../../common/tables.h` (`TMailBoxTable`, `TMailBoxRespondUnreadData` vb. yapılar için). Bu sınıf, `CHARACTER` sınıfı içinde bir üye olarak tutulur.

### `MailBox.cpp`

*   **Amaç:** (`__MAILBOX__` tanımlıysa) `MailBox.h`'de bildirilen `CMailBox` sınıfının metotlarını uygular. Posta kutusu açma, posta gönderme (yasaklı kelime, para, eşya kontrolü dahil), hedef oyuncu kontrolü, posta içeriğini görüntüleme, eşya/para alma (vergi hesaplama dahil) ve posta silme işlemlerinin mantığını içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`CMailBox::CMailBox(...)`):**
        *   DB'den yüklenen `TMailBoxTable` dizisini `vecMailBox`'a kopyalar.
        *   `Owner`'ı ayarlar.
        *   Posta listesini (`HEADER_GC_MAILBOX`) istemciye gönderir.
        *   Posta kutusunun açıldığını istemciye bildirir (`ServerProcess(MAILBOX_GC_OPEN)`).
    *   **Yıkıcı (`CMailBox::~CMailBox()`):**
        *   Posta kutusunun kapandığını istemciye bildirir (`ServerProcess(MAILBOX_GC_CLOSE)`).
        *   Sahibinin posta kutusu yükleme bayrağını (`IsMailBoxLoading`) ve zaman damgasını (`SetMyMailBoxTime`) günceller.
    *   **`ServerProcess(...)`:** `TPacketMailboxProcess` paketini doldurur ve sahibin desc'ine gönderir.
    *   **`Write(...)`:**
        *   Gönderim ücreti (`EMAILBOX::MAILBOX_PRICE_YANG`) dahil yeterli Yang/Won kontrolü yapar.
        *   Alıcı adı, başlık ve mesaj için yasaklı kelime kontrolü (`CBanwordManager`) yapar.
        *   `TMailBoxTable` yapısını doldurur (gönderen, mesaj, zaman damgaları, GM durumu).
        *   Eğer eşya eklenmişse (`pos.IsValidItemPosition()`):
            *   Eşyanın geçerliliğini kontrol eder (takılı değil, kilitli değil, mühürlü değil vb.).
            *   Eşya bilgilerini (VNUM, adet, soketler, efsunlar, görünüm, element, set değeri vb.) `p.AddData`'ya kopyalar.
            *   Eşyayı karakterden siler (`pItem->RemoveFromCharacter()`, `ITEM_MANAGER::DestroyItem(pItem)`).
        *   Gönderilen para/eşya varsa `p.Message.bIsItemExist` bayrağını ayarlar.
        *   Gönderim ücretini ve gönderilen Yang/Won'u karakterden düşer (`Owner->PointChange`).
        *   Posta bilgilerini DB'ye kaydetmek için `HEADER_GD_MAILBOX_WRITE` paketi gönderir.
        *   Başarılı gönderim durumunu istemciye bildirir (`ServerProcess`).
        *   İşlemi loglar (`LogManager::MailLog`).
    *   **Statik Açma/Oluşturma/Sorgulama Fonksiyonları:**
        *   `Open`: Seviye limiti, diğer pencerelerin durumu, bekleme süresi gibi kontrolleri yapar. `ch->SetMailBoxLoading(true)` yapar ve DB'den posta verilerini istemek için `HEADER_GD_MAILBOX_LOAD` paketi gönderir.
        *   `Create`: DB'den gelen yanıtla çağrılır. Karakterin `CMailBox` nesnesini oluşturur (`ch->SetMailBox`).
        *   `UnreadData`: DB'den okunmamış posta bilgisini istemek için `HEADER_GD_MAILBOX_UNREAD` paketi gönderir.
        *   `ResultUnreadData`: DB'den gelen okunmamış posta yanıtını (`TMailBoxRespondUnreadData`) istemciye iletir.
        *   `SendGMMail`: GM komutuyla çağrıldığında, `TMailBoxTable` yapısını doldurur, `bIsGMPost`'u `true` yapar ve DB'ye gönderir. Hedef oyuncu online ise `UnreadData` ile onu bilgilendirir.
    *   **Hedef Kontrolü (`CheckPlayer`, `CheckPlayerResult`):**
        *   `CheckPlayer`: Gönderilecek oyuncu ismini DB'ye `HEADER_GD_MAILBOX_CHECK_NAME` ile gönderir.
        *   `CheckPlayerResult`: DB'den gelen yanıtı işler. Oyuncu yoksa, posta kutusu doluysa veya gönderen/alan birbirini engellemişse (`__MESSENGER_BLOCK_SYSTEM__` aktifse) uygun hata kodunu `ServerProcess(MAILBOX_GC_POST_WRITE_CONFIRM)` ile istemciye bildirir.
    *   **Posta İçeriği Alma (`AddData`):**
        *   Postanın geçerliliğini kontrol eder.
        *   Postayı okunmuş olarak işaretlemek için DB'ye `HEADER_GD_MAILBOX_CONFIRM` paketi gönderir.
        *   Postanın detaylı içeriğini (`TMailBoxAddData`) istemciye `HEADER_GC_MAILBOX_ADD_DATA` ile gönderir.
        *   İstemci arayüzünü günceller (`ServerProcess(MAILBOX_GC_ADD_DATA)`).
    *   **Eşya/Para Alma (`GetAllItems`, `GetItem`):**
        *   `GetItem`: Postanın geçerliliğini, silinmemiş olmasını ve içinde alınacak bir şey (`bIsItemExist`) olmasını kontrol eder. Vergi (`EMAILBOX::MAILBOX_TAX`) hesaplar. Yang/Won limitlerini kontrol eder. Eşya varsa envanterde yer olup olmadığını kontrol eder. Başarılıysa:
            *   Eşyayı oluşturur (`ITEM_MANAGER::CreateItem`), özelliklerini (soket, efsun vb.) kopyalar ve karaktere verir (`Owner->AutoGiveItem`).
            *   Yang/Won'u karaktere verir (`Owner->GiveGold`, `Owner->GiveCheque`).
            *   Posta içeriğini sıfırlar (`mail.AddData.*`, `mail.Message.bIsItemExist = false`).
            *   DB'ye postanın alındığını bildirmek için `HEADER_GD_MAILBOX_GET` paketi gönderir.
            *   İstemciye sonucu bildirir (`ServerProcess(MAILBOX_GC_POST_GET_ITEMS)`).
        *   `GetAllItems`: `vecMailBox` üzerinde döner ve her posta için `GetItem(..., true)` çağırır. Alınan postaların indekslerini bir buffer'a yazar ve `HEADER_GC_MAILBOX_ALL` ile istemciye tek seferde bildirir.
    *   **Posta Silme (`DeleteAllMails`, `DeleteMail`):**
        *   `DeleteMail`: Postanın geçerliliğini ve silinmemiş olmasını kontrol eder. İçinde hala eşya/para varsa silmez (`bIsItemExist`). Başarılıysa:
            *   DB'ye postayı silmek için `HEADER_GD_MAILBOX_DELETE` paketi gönderir.
            *   Postayı yerel olarak silinmiş işaretler (`mail.bIsDeleted = true`).
            *   İstemciye sonucu bildirir (`ServerProcess(MAILBOX_GC_POST_DELETE)`).
        *   `DeleteAllMails`: `vecMailBox` üzerinde döner ve her posta için `DeleteMail(..., true)` çağırır. Başarısız olan olursa işaretler ve toplu silme sonucunu istemciye bildirir (`ServerProcess(MAILBOX_GC_POST_ALL_DELETE)`).
*   **Bağlantılı Dosyalar:** `MailBox.h`, `stdafx.h`, `char.h`, `char_manager.h`, `desc.h`, `packet.h`, `item.h`, `item_manager.h`, `banword.h`, `buffer_manager.h`, `db.h`, `config.h`, `desc_client.h`, `log.h`, (isteğe bağlı) `messenger_manager.h`, `locale_service.h`, `DragonSoul.h`. 

### `marriage.h`

*   **Amaç:** Oyun içindeki evlilik sistemini yönetmek için `marriage` isim alanı altında `TMarriage` yapısını ve `CManager` singleton sınıfını tanımlar. Evli çiftlerin bilgilerini, evlilik seviyelerini, özel eşya bonuslarını, düğün süreçlerini ve eşlerin birbirine yakın olup olmadığını kontrol etme gibi işlevleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`TWeddingInfo` Struct:**
        *   `dwMapIndex`: Düğünün yapıldığı haritanın indeksi.
    *   **`MARRIAGE_POINT_PER_DAY` (extern const int):** Evlilik puanının günlük artış miktarı.
    *   **`TMarriage` Struct:**
        *   **Üye Değişkenler:**
            *   `m_pid1`, `m_pid2` (DWORD): Evli çiftin oyuncu ID'leri.
            *   `love_point` (int): Sevgi puanı.
            *   `marry_time` (time_t): Evlilik tarihi.
            *   `ch1`, `ch2` (LPCHARACTER): Evli karakterlerin işaretçileri (online iseler).
            *   `bSave` (bool): Verilerin kaydedilmesi gerekip gerekmediğini belirten bayrak.
            *   `is_married` (bool): Evliliğin resmi olup olmadığını belirtir.
            *   `name1`, `name2` (std::string): Evli çiftin isimleri.
            *   `pWeddingInfo` (TWeddingInfo*): Düğün bilgileri için işaretçi.
            *   `isLastNear` (bool): Eşlerin bir önceki kontrolde yakın olup olmadığı.
            *   `byLastLovePoint` (BYTE): Bir önceki kontroldeki sevgi puanı.
            *   `eventNearCheck` (LPEVENT): Eşlerin yakınlığını periyodik olarak kontrol eden olay.
        *   **Yapıcı/Yıkıcı:** Evlilik bilgilerini başlatır, olayları temizler.
        *   **Metotlar:**
            *   `Login(LPCHARACTER ch)`: Bir eş oyuna girdiğinde çağrılır.
            *   `Logout(DWORD pid)`: Bir eş oyundan çıktığında çağrılır.
            *   `IsOnline()`: Her iki eşin de oyunda olup olmadığını kontrol eder.
            *   `IsNear()`: Eşlerin aynı haritada olup olmadığını kontrol eder.
            *   `GetOther(DWORD PID)`: Verilen PID'ye sahip eşin diğer eşinin PID'sini döndürür.
            *   `GetMarriagePoint()`: Evlilik puanını hesaplar (günlük artış + sevgi puanı).
            *   `GetMarriageGrade()`: Evlilik puanına göre evlilik seviyesini (0-3) döndürür.
            *   `GetBonus(DWORD dwItemVnum, bool bShare, LPCHARACTER me)`: Belirli bir evlilik eşyası için bonus değerini evlilik seviyesine göre döndürür.
            *   `WarpToWeddingMap(DWORD dwPID)`: Oyuncuyu düğün haritasına ışınlar.
            *   `Save()`: Evlilik bilgilerini veritabanına kaydetmek üzere istek gönderir.
            *   `SetMarried()`: Evliliği resmi olarak ayarlar.
            *   `Update(DWORD point)`: Sevgi puanını günceller.
            *   `RequestEndWedding()`: Düğünü sonlandırmak için istek gönderir.
            *   `StartNearCheckEvent()`, `StopNearCheckEvent()`, `NearCheck()`: Eşlerin yakınlığını ve sevgi puanı değişimini periyodik olarak kontrol eden olayları yönetir.
    *   **Typedef'ler:**
        *   `WeddingSet` (std::set<std::pair<DWORD, DWORD>>): Düğün yapan çiftlerin PID'lerini tutar.
        *   `MariageMap` (std::map<DWORD, TMarriage*>): Oyuncu PID'si ile `TMarriage` nesnesi arasında eşleşme tutar.
    *   **`CManager` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:**
        *   **Metotlar:**
            *   `Initialize()`, `Destroy()`: Evlilik yöneticisini başlatır/yok eder.
            *   `Get(DWORD dwPlayerID)`: Verilen oyuncu ID'sine ait `TMarriage` nesnesini döndürür.
            *   `IsMarriageUniqueItem(DWORD dwItemVnum)`: Bir eşyanın evliliğe özel olup olmadığını kontrol eder.
            *   `IsMarried(DWORD dwPlayerID)`, `IsEngaged(DWORD dwPlayerID)`, `IsEngagedOrMarried(DWORD dwPlayerID)`: Oyuncunun evlilik durumunu kontrol eder.
            *   `RequestAdd(dwPID1, dwPID2, szName1, szName2)`: Yeni bir evlilik (nişan) eklemek için DB'ye istek gönderir.
            *   `Add(dwPID1, dwPID2, tMarryTime, szName1, szName2)`: DB'den gelen yanıta göre yeni bir evlilik oluşturur.
            *   `RequestUpdate(dwPID1, dwPID2, iUpdatePoint, byMarried)`: Evlilik bilgilerini (sevgi puanı, evlilik durumu) güncellemek için DB'ye istek gönderir.
            *   `Update(dwPID1, dwPID2, lTotalPoint, byMarried)`: DB'den gelen yanıta göre evlilik bilgilerini günceller.
            *   `RequestRemove(dwPID1, dwPID2)`: Bir evliliği sonlandırmak (boşanma) için DB'ye istek gönderir.
            *   `Remove(dwPID1, dwPID2)`: DB'den gelen yanıta göre bir evliliği sonlandırır.
            *   `Login(LPCHARACTER ch)`, `Logout(DWORD pid)`, `Logout(LPCHARACTER ch)`: Oyuncu giriş/çıkışlarında evlilik durumunu günceller.
            *   `WeddingReady(dwPID1, dwPID2, dwMapIndex)`: Düğün için haritayı hazırlar.
            *   `WeddingStart(dwPID1, dwPID2)`: Düğünü başlatır, çifti düğün haritasına ışınlar.
            *   `WeddingEnd(dwPID1, dwPID2)`: Düğünü sonlandırır.
            *   `RequestEndWedding(dwPID1, dwPID2)`: Düğünü sonlandırmak için DB'ye istek gönderir.
            *   `for_each_wedding(Func f)`: Düğün yapan tüm çiftler için bir fonksiyon çalıştırır (template metot).
        *   **Özel (Private) Üyeler:**
            *   `m_Marriages` (TR1_NS::unordered_set<TMarriage*>): Tüm evlilik nesnelerini tutar.
            *   `m_MarriageByPID` (MariageMap): PID ile evlilik nesnesi eşleşmelerini tutar.
            *   `m_setWedding` (WeddingSet): Aktif düğünleri tutar.
*   **Bağlantılı Dosyalar:** `marriage.cpp` (uygulama), `stdafx.h`, `char.h` (`LPCHARACTER`).

### `marriage.cpp`

*   **Amaç:** `marriage.h` dosyasında bildirilen `TMarriage` yapısı ve `CManager` sınıfının metotlarını uygular. Evlilik sistemiyle ilgili tüm mantığı (puan hesaplama, bonus verme, durum güncellemeleri, DB iletişimi, olay yönetimi) içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Değişkenler ve Sabitler:**
        *   `MAX_LOVE_GRADE`: Maksimum evlilik seviyesi (4).
        *   `MAX_MARRIAGE_UNIQUE_ITEM`: Evliliğe özel bonus veren maksimum eşya sayısı (6).
        *   `g_ItemBonus[]`: Evliliğe özel eşyaların VNUM'larını ve her evlilik seviyesinde verdikleri bonus miktarlarını tutan bir dizi.
        *   `MARRIAGE_POINT_PER_DAY`, `MARRIAGE_POINT_PER_DAY_FAST`: Normal ve hızlı (premium ile) günlük evlilik puanı artış miktarları.
    *   **`SendLoverInfo(LPCHARACTER ch, const string& lover_name, int love_point)` Fonksiyonu:** Eş bilgilerini (isim, sevgi puanı) istemciye gönderir (`HEADER_GC_LOVER_INFO`).
    *   **`TMarriage` Metotları:**
        *   **`~TMarriage()`:** Yakınlık kontrolü olayını durdurur, online ise eşlere boşanma komutu gönderir, düğün bilgilerini siler.
        *   **`GetMarriageGrade()`:** `GetMarriagePoint()` sonucuna göre evlilik seviyesini (0-3) belirler. (50-64: 0, 65-79: 1, 80-99: 2, 100: 3).
        *   **`GetMarriagePoint()`:** Evlilik puanını hesaplar. Temel puan 50'dir. Evli kalınan gün başına puan eklenir (normal veya premium hızlı). Ayrıca `love_point` (oyun içi aktivitelerle kazanılan) 1.000.000'a bölünerek ek puan verir. Toplam puan 100 ile sınırlıdır. Test sunucusunda `lovepoint` event flag'i ile ayarlanabilir.
        *   **`IsNear()`:** Eşlerin aynı haritada olup olmadığını kontrol eder.
        *   **`GetBonus(dwItemVnum, bShare, me)`:** Belirtilen evlilik eşyasından (`dwItemVnum`) alınacak bonusu hesaplar. `bShare` true ise, eşlerden herhangi birinde eşya varsa bonus verilir. False ise, sadece `me` karakteri dışındaki eşte varsa bonus verilir. Bonus miktarı `g_ItemBonus` dizisinden ve `GetMarriageGrade()` ile belirlenen evlilik seviyesinden alınır.
        *   **`Login(ch)`:** Oyuncu giriş yaptığında çağrılır. Eşlerin `LPCHARACTER` işaretçilerini ayarlar, evli iseler eş bilgilerini gönderir (`SendLoverInfo`), online iseler birbirlerini partner olarak ayarlar (`SetMarryPartner`) ve yakınlık kontrol olayını başlatır (`StartNearCheckEvent`). İstemcilere `lover_login` komutu gönderir.
        *   **`Logout(pid)`:** Oyuncu çıkış yaptığında çağrılır. İlgili `LPCHARACTER` işaretçisini `NULL` yapar, verileri kaydeder (`Save`), diğer eşin partnerini `NULL` yapar, yakınlık kontrol olayını durdurur (`StopNearCheckEvent`). Online kalan eşe `lover_logout` komutu gönderir.
        *   **`NearCheck()`:** Periyodik olarak çağrılır. Eşlerin yakınlık durumu değiştiyse (`IsNear()` vs `isLastNear`) istemcilere `lover_near` veya `lover_far` komutu gönderir. Sevgi puanı değiştiyse (`byLastLovePoint` vs `GetMarriagePoint()`) istemcilere `HEADER_GC_LOVE_POINT_UPDATE` paketi gönderir.
        *   **Olay Yönetimi (`near_check_event_info`, `near_check_event`, `StartNearCheckEvent`, `StopNearCheckEvent`):** `NearCheck()` fonksiyonunu periyodik olarak (5 saniyede bir) çağırmak için bir olay (event) sistemi kullanır.
        *   **`Save()`:** `bSave` bayrağı true ise `CManager::instance().RequestUpdate()` çağırarak evlilik bilgilerini (sevgi puanı, evlilik durumu) DB'ye kaydetme isteği gönderir.
        *   **`SetMarried()`:** `is_married`'i true yapar, `Save()` çağırır ve online ise eşlere bilgi gönderir.
        *   **`Update(point)`:** Sevgi puanını (`love_point`) artırır (maksimum 2 milyar ile sınırlı). `bSave`'i true yapar.
        *   **`WarpToWeddingMap(dwPID)`:** `pWeddingInfo`'daki harita indeksine oyuncuyu ışınlar.
        *   **`RequestEndWedding()`:** Düğünü bitirmek için `CManager` üzerinden DB'ye istek gönderir.
    *   **`CManager` Metotları:**
        *   **`IsMarriageUniqueItem(dwItemVnum)`:** `g_ItemBonus` dizisinde `dwItemVnum`'ı arar.
        *   **`IsMarried(dwPlayerID)`, `IsEngaged(dwPlayerID)`, `IsEngagedOrMarried(dwPlayerID)`:** `Get(dwPlayerID)` ile `TMarriage` nesnesini alıp durumunu kontrol eder.
        *   **`Align(dwPID1, dwPID2)`:** Her zaman `dwPID1 < dwPID2` olmasını sağlar (DB anahtarı için standardizasyon).
        *   **`Get(dwPlayerID)`:** `m_MarriageByPID` haritasından `TMarriage` nesnesini bulur.
        *   **DB İstekleri (`RequestAdd`, `RequestUpdate`, `RequestRemove`, `RequestEndWedding`):** İlgili DB paketlerini (`HEADER_GD_MARRIAGE_ADD` vb.) oluşturur ve `db_clientdesc->DBPacket()` ile gönderir. `RequestAdd` ve `RequestUpdate` öncesi `Align` çağırır.
        *   **DB Yanıtları (`Add`, `Update`, `Remove`):** DB'den gelen bilgilerle `TMarriage` nesnelerini oluşturur, günceller veya siler. `m_Marriages` (set) ve `m_MarriageByPID` (map) koleksiyonlarını yönetir.
        *   **`Login(ch)`, `Logout(pid)`:** `TMarriage::Login/Logout` çağırır.
        *   **Düğün Yönetimi (`WeddingReady`, `WeddingStart`, `WeddingEnd`):**
            *   `WeddingReady`: `TMarriage` nesnesine `TWeddingInfo` ekler ve düğün haritası indeksini ayarlar.
            *   `WeddingStart`: Çifti düğün haritasına ışınlar (`TMarriage::WarpToWeddingMap`), `m_setWedding`'e ekler.
            *   `WeddingEnd`: `WeddingManager::instance().End()` ile düğün haritasını sonlandırır, `TMarriage`'daki `pWeddingInfo`'yu siler, `m_setWedding`'den çıkarır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `char_manager.h`, `sectree_manager.h`, `desc_client.h`, `p2p.h`, `wedding.h`, `config.h`, `utils.h`, `questmanager.h`. 

### `MeleyLair.h`

*   **Amaç:** (`__GUILD_DRAGONLAIR__` tanımlıysa) Meley'in İni lonca zindan sistemini yönetmek için `MeleyLair` isim alanı altında sabitler, `CMgrMap` sınıfı (her bir zindan örneğini yönetir) ve `CMgr` singleton sınıfını (genel zindan yöneticisi) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`eConfig` Enum'u:** Zindanla ilgili sabitleri tanımlar:
        *   `MAP_INDEX`, `SUBMAP_INDEX`: Ana zindan haritası ve çıkışta ışınlanılacak alt harita indeksi.
        *   `MIN_LVL`: Loncaya katılım için minimum seviye.
        *   `PARTICIPANTS_LIMIT`: Zindana katılabilecek maksimum oyuncu sayısı.
        *   `LADDER_POINTS_COST`, `LADDER_POINTS_RETURN`: Zindana giriş için merdiven puanı maliyeti ve başarısızlık durumunda iade edilen miktar.
        *   `COOLDOWN_DUNGEON`: Başarılı bir zindan sonrası bekleme süresi.
        *   `NPC_VNUM`, `GATE_VNUM`, `BOSS_VNUM`, `STATUE_VNUM`, `CHEST_VNUM`: Zindandaki ana NPC, kapı, Meley (Boss), heykeller ve sandık VNUM'ları.
        *   `REWARD_ITEMCHEST_VNUM_1`, `REWARD_ITEMCHEST_VNUM_2`: Ödül sandığı VNUM'ları.
        *   `TIME_LIMIT_DUNGEON`: Zindanı tamamlamak için maksimum süre.
        *   `SEAL_VNUM_KILL_STATUE`: Heykelleri yok etmek için gereken mühür eşyasının VNUM'u.
        *   `TIME_LIMIT_TO_KILL_STATUE`: Heykelleri yok etmek için son aşamadaki süre limiti.
        *   Çeşitli `TIME_RESPAWN_*`, `MOBCOUNT_RESPAWN_*`, `MOBVNUM_RESPAWN_*`: Zindanın farklı aşamalarında yaratıkların ve taşların yeniden doğma süreleri, sayıları ve VNUM'ları.
    *   **Global Diziler:**
        *   `stoneSpawnPos[][]`: 2. aşamadaki taşların doğma koordinatları.
        *   `monsterSpawnPos[][]`: Genel yaratıkların doğma koordinatları.
    *   **`CMgrMap` Sınıfı:**
        *   Tek bir Meley İni örneğini yönetir.
        *   **Üye Değişkenler:**
            *   `map_index` (long): Bu zindan örneğinin harita indeksi.
            *   `guild_id` (DWORD): Zindanı başlatan loncanın ID'si.
            *   `time_start`, `last_stoneKilled`, `kill_stonesCount`, `kill_bossesCount`, `reward` (DWORD): Zindan başlangıç zamanı, son taşın öldürülme zamanı, öldürülen taş/boss sayısı, ödül zamanı.
            *   `dungeon_step` (BYTE): Zindanın mevcut aşaması (1-4).
            *   `v_Participants`, `v_Already`, `v_Rewards` (std::vector<DWORD>): Katılımcıların, son aşamada heykeli kırmış olanların ve ödül almış olanların oyuncu ID'leri.
            *   `pkSectreeMap` (LPSECTREE_MAP): Zindan haritasının sectree işaretçisi.
            *   `pkMainNPC`, `pkGate`, `pkBoss`, `pkStatue1`, `pkStatue2`, `pkStatue3`, `pkStatue4` (LPCHARACTER): Zindandaki önemli NPC ve yaratıkların işaretçileri.
            *   `e_pEndEvent`, `e_pWarpEvent`, `e_SpawnEvent`, `e_SEffectEvent`, `e_DestroyStatues` (LPEVENT): Zindanla ilgili çeşitli olaylar (zaman sınırı, ışınlama, yaratık doğumu, efektler, heykel yok etme).
        *   **Yapıcı/Yıkıcı:** Zindan haritasını ve lonca ID'sini alır, olayları ve değişkenleri başlatır/temizler.
        *   **Metotlar:**
            *   `GetMapIndex()`, `GetGuildID()`: İlgili bilgileri döndürür.
            *   `GetDungeonStep()`, `SetDungeonStep(BYTE bStep)`: Zindan aşamasını alır/ayarlar. `SetDungeonStep` ilgili aşama için olayları başlatır.
            *   `StartDungeonStep(BYTE bStep)`: Belirli bir zindan aşamasının mantığını (yaratık doğumu vb.) başlatır.
            *   Zaman ve öldürme sayılarını yöneten `Get/Set` metotları.
            *   `GetParticipantsCount()`, `Partecipant(bool bInsert, DWORD dwPlayerID)`, `IsPartecipant(DWORD dwPlayerID)`: Katılımcı listesini yönetir.
            *   `Spawn(...)`: Zindan haritasında belirtilen VNUM'da yaratık doğurur.
            *   `Start()`: Zindanın başlangıç NPC'lerini ve yaratıklarını (kapı, boss, heykeller) doğurur.
            *   `StartDungeon(LPCHARACTER pkChar)`: Zindanı resmi olarak başlatır, kapıyı açar, zaman sınırını başlatır, loncaya duyuru yapar.
            *   `EndDungeon(bool bSuccess, bool bGiveBack)`: Zindanı sonlandırır (başarılı veya başarısız). Lonca merdiven puanlarını ayarlar, loncaya duyuru yapar, ışınlama olayını başlatır.
            *   `EndDungeonWarp()`: Tüm katılımcıları zindandan dışarı ışınlar, haritayı yok eder ve `CMgrMap` nesnesini siler.
            *   `Damage(LPCHARACTER pkStatue)`: Heykellere hasar verildiğinde çağrılır. Heykelin canını kontrol eder ve belirli bir eşiğe ulaştığında heykele bir sonraki aşamaya geçiş için affect (`AFF_STATUE1/2/3`) ekler. Tüm heykeller hazır olduğunda `SetDungeonStep` ile bir sonraki zindan aşamasını tetikler.
            *   `OnKill(DWORD dwVnum)`: Zindandaki bir yaratık öldürüldüğünde çağrılır. Özellikle 2. ve 3. aşamalardaki taşların ve boss'ların öldürülmesini takip eder, heykellerdeki koruma affect'lerini kaldırır.
            *   `OnKillStatue(LPITEM pkItem, LPCHARACTER pkChar, LPCHARACTER pkStatue)`: 4. aşamada bir oyuncu mühür eşyası (`SEAL_VNUM_KILL_STATUE`) kullanarak bir heykeli kırdığında çağrılır. Heykele `AFF_STATUE4` affect'i ekler. Tüm heykeller kırıldığında `DungeonResult()`'ı çağırır.
            *   `DungeonResult()`: Zindanın sonucunu belirler (başarılıysa ödül sandığı doğurur, başarısızsa `EndDungeon(false, ...)` çağırır).
            *   `CheckRewarder(DWORD dwPlayerID)`, `GiveReward(LPCHARACTER pkChar, BYTE bReward)`: Ödül almış oyuncuları takip eder ve ödül sandığını verir.
            *   Önemli karakter işaretçilerini (`GetBossChar`, `GetStatue*Char`) döndüren metotlar.
    *   **`CMgr` Sınıfı (Singleton):**
        *   Genel Meley İni yöneticisi.
        *   **Üye Değişkenler:**
            *   `lMapCenterPos`, `lSubMapPos` (PIXEL_POSITION): Zindan ana haritasının ve çıkış haritasının varsayılan merkez koordinatları.
            *   `m_RegGuilds` (RegGuildsMap - `std::map<DWORD, CMgrMap*> `): Aktif zindan örneklerini (Lonca ID -> `CMgrMap*`) tutar.
        *   **Metotlar:**
            *   `Initialize()`, `Destroy()`: Yöneticiyi başlatır/yok eder (tüm aktif zindanları temizler).
            *   `Register(LPCHARACTER pkChar, int& iRes1, int& iRes2)`: Lonca liderinin zindana kaydolmasını sağlar. Lonca seviyesi, merdiven puanı, bekleme süresi gibi kontroller yapar. Başarılıysa özel bir harita oluşturur (`SECTREE_MANAGER::CreatePrivateMap`), `CMgrMap` nesnesi yaratır ve `m_RegGuilds`'e ekler. Sonucu `iRes1` ve `iRes2` ile döndürür.
            *   `isRegistered(CGuild* pkGuild, int& iCH)`: Bir loncanın zaten bir zindana kayıtlı olup olmadığını kontrol eder.
            *   `Enter(CGuild* pkGuild, LPCHARACTER pkChar, int& iLimit)`: Bir oyuncunun loncasının zindanına girmesini sağlar. Katılımcı limiti, zindan aşaması gibi kontroller yapar. Oyuncuyu zindan haritasına ışınlar.
            *   `Leave(CGuild* pkGuild, LPCHARACTER pkChar, bool bSendOut)`: Oyuncunun zindandan ayrılmasını yönetir (parti listesinden çıkarır). `bSendOut` true ise veya loncası artık zindanda değilse `WarpOut` ile dışarı ışınlar.
            *   `LeaveRequest(CGuild* pkGuild, LPCHARACTER pkChar)`: Oyuncunun zindandan ayrılma isteğini işler, her zaman dışarı ışınlar.
            *   `IsMeleyMap(long lMapIndex)`: Verilen harita indeksinin Meley zindan haritası olup olmadığını kontrol eder.
            *   `Check(CGuild* pkGuild, LPCHARACTER pkChar)`: Oyuncunun hala geçerli bir zindanda olup olmadığını kontrol eder, değilse dışarı ışınlar.
            *   `WarpOut(LPCHARACTER pkChar)`: Oyuncuyu `lSubMapPos`'a veya imparatorluk başlangıç noktasına ışınlar.
            *   `SetXYZ(...)`, `GetXYZ()`: Zindan merkez koordinatlarını ayarlar/alır.
            *   `SetSubXYZ(...)`, `GetSubXYZ()`: Çıkış haritası koordinatlarını ayarlar/alır.
            *   `Start(LPCHARACTER pkChar, CGuild* pkGuild)`: İlgili `CMgrMap` üzerinden zindanı başlatır.
            *   `Damage(LPCHARACTER pkStatue, CGuild* pkGuild)`: İlgili `CMgrMap` üzerinden heykele hasar verilmesini yönetir.
            *   `Remove(DWORD dwGuildID)`: Bir zindan örneğini `m_RegGuilds`'den siler (genellikle `EndDungeonWarp` sonrası).
            *   `OnKill(DWORD dwVnum, CGuild* pkGuild)`: İlgili `CMgrMap`'e yaratık ölüm bilgisini iletir.
            *   `OnKillStatue(LPITEM pkItem, ...)`, `OnKillCommon(LPCHARACTER pkMonster, ...)`: İlgili `CMgrMap`'e heykel kırma veya özel yaratık öldürme bilgisini iletir. Mühür düşürme mantığını içerir.
            *   `CanGetReward(LPCHARACTER pkChar, CGuild* pkGuild)`, `Reward(LPCHARACTER pkChar, ...)`: İlgili `CMgrMap` üzerinden ödül alma durumunu kontrol eder ve ödülü verir.
            *   `OpenRanking(LPCHARACTER pkChar)`: Meley zindanı sıralamasını (`log.meley_dungeon` tablosu) oyuncuya gönderir.
            *   `MemberRemoved(LPCHARACTER pkChar, CGuild* pkGuild)`: Loncasından atılan bir oyuncuyu zindandan çıkarır.
            *   `GuildRemoved(CGuild* pkGuild)`: Dağıtılan bir loncanın zindanını sonlandırır.
*   **Bağlantılı Dosyalar:** `MeleyLair.cpp` (uygulama), `../../common/service.h`, `../../common/length.h`, `../../common/item_length.h`, `../../common/tables.h`, `guild.h`, `char_manager.h`, `sectree_manager.h`.

### `MeleyLair.cpp`

*   **Amaç:** (`__GUILD_DRAGONLAIR__` tanımlıysa) `MeleyLair.h` dosyasında bildirilen `CMgrMap` ve `CMgr` sınıflarının metotlarını uygular. Meley'in İni zindanının tüm çalışma mantığını, aşamalarını, olaylarını, NPC ve yaratık yönetimini, katılımcı takibini, DB etkileşimlerini ve ödüllendirme sistemini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Global Diziler:** `stoneSpawnPos` ve `monsterSpawnPos` burada tanımlanır.
    *   **Yardımcı Struct'lar:**
        *   `FNotice`: Haritadaki tüm karakterlere duyuru mesajı göndermek için bir functor.
        *   `FExitAndGoTo`: Haritadaki tüm karakterleri zindandan çıkarmak için bir functor.
    *   **Olay Bilgi Struct'ları ve Fonksiyonları:**
        *   `r_meleystatues_info`, `r_meleystatues_event`: 4. aşamada heykelleri kırmak için verilen süreyi yönetir. Süre dolduğunda veya tüm heykeller kırıldığında `CMgrMap::DungeonResult()` çağrılır.
        *   `r_meleylimit_info`, `r_meleylimit_event`: Zindanın genel zaman sınırını (`TIME_LIMIT_DUNGEON`) veya başarılı/başarısız bitiş sonrası ışınlama süresini yönetir. Süre dolduğunda `CMgrMap::EndDungeon()` veya `CMgrMap::EndDungeonWarp()` çağrılır.
        *   `r_meleyspawn_info`, `r_meleyspawn_event`: Zindanın farklı aşamalarında (`bStep`) yaratıkların ve taşların periyodik olarak yeniden doğmasını (`dwTimeReload`) yönetir. Belirli VNUM (`dwMobVnum`) ve sayılarda (`dwMobCount`) yaratık doğurur. 2. ve 3. aşamalarda taşlar yok edildiğinde heykellere koruma affect'leri (`AFF_STATUE1/2/3`) ekler/kaldırır.
        *   `r_meleyeffect_info`, `r_meleyeffect_event`: Zindan aşamaları arasındaki geçişlerde heykellerin özel efektler (dönme, skill kullanma gibi) yapmasını sağlar.
    *   **`CMgrMap` Metotları:**
        *   **Yapıcı:** Olayları `NULL` yapar, vektörleri temizler, harita ve lonca bilgilerini ayarlar, `Start()` çağırır.
        *   **`Destroy()`:** Tüm olayları iptal eder, değişkenleri sıfırlar.
        *   **`SetDungeonStep(bStep)`:** Zindan aşamasını değiştirir. Önceki aşamanın spawn olayını iptal eder. Yeni aşamaya göre:
            *   **Aşama 1:** Normal yaratık spawn olayını (`MOBVNUM_RESPAWN_COMMON_STEP1`) başlatır. Zindan başlangıç zamanını kaydeder.
            *   **Aşama 2:** Heykel efekti olayını (aşama 1 efekti) başlatır. Bu olay bitince `StartDungeonStep(2)` çağrılır.
            *   **Aşama 3:** Heykel efekti olayını (aşama 2 efekti) başlatır. Bu olay bitince `StartDungeonStep(3)` çağrılır.
            *   **Aşama 4:** Heykelleri `SetArmada()` ile saldırılabilir yapar. Heykelleri kırma süresi için `r_meleystatues_event`'i başlatır. Lonca üyelerine duyuru yapar.
        *   **`StartDungeonStep(bStep)`:** Belirli bir zindan aşaması için spawn mantığını kurar:
            *   **Aşama 2:** `MOBVNUM_RESPAWN_COMMON_STEP2` ve `MOBVNUM_RESPAWN_STONE_STEP2` (taşlar) için spawn olayını başlatır. Heykellere `AFF_STATUE1` (veya `AFF_STATUE2`) eklenmesini yönetir.
            *   **Aşama 3:** `MOBVNUM_RESPAWN_COMMON_STEP3`, `MOBVNUM_RESPAWN_BOSS_STEP3` (küçük bosslar), `MOBVNUM_RESPAWN_SUBBOSS_STEP3` ve `MOBVNUM_RESPAWN_STONE_STEP2` (taşlar) için spawn olayını başlatır. Heykellere `AFF_STATUE1/2/3` eklenmesini yönetir.
        *   **`Start()`:** Zindan ilk oluşturulduğunda ana NPC, kapı, Meley (Boss) ve 4 heykeli `Spawn()` ile haritaya yerleştirir.
        *   **`StartDungeon(pkChar)`:** Lonca lideri zindanı başlattığında çağrılır. Katılımcı kontrolü, NPC/yaratık varlığı kontrolü yapar. Kapıyı (`pkGate`) öldürür. `SetDungeonStep(1)` ile 1. aşamayı başlatır. Zindan zaman sınırı için `r_meleylimit_event`'i başlatır.
        *   **`EndDungeon(bSuccess, bGiveBack)`:** Zindanı sonlandırır. Başarı durumuna göre loncaya duyurular yapar, lonca merdiven puanını ayarlar (`ChangeLadderPoint`), zindan bekleme süresini (`SetDungeonCooldown`) ayarlar, DB'ye zindan durumunu günceller (`pkGuild->RequestDungeon(0,0)`). Işınlama için `r_meleylimit_event`'i (warp moduyla) başlatır veya doğrudan `EndDungeonWarp` çağırır. `bGiveBack` true ve `bSuccess` true ise `LogManager::instance().MeleyLog` ile log kaydı tutar.
        *   **`EndDungeonWarp()`:** Haritadaki tüm oyuncuları `FExitAndGoTo` ile dışarı ışınlar. `CMgr::Remove()` ile zindanı yöneticiden kaldırır, haritayı `SECTREE_MANAGER::DestroyPrivateMap` ile yok eder ve `CMgrMap` nesnesini `M2_DELETE(this)` ile siler.
        *   **`Damage(pkStatue)`:** Heykele hasar verildiğinde çağrılır. Hasar, heykelin canını belli bir eşiğin (%75, %50, %1) altına düşürürse, heykelin canını bu eşiğe sabitler ve ilgili `AFF_STATUE1/2/3` affect'ini ekler. Tüm heykeller bir aşama için hazır olduğunda `SetDungeonStep()` ile sonraki aşamaya geçer.
        *   **`OnKill(dwVnum)`:** 2. ve 3. aşamalarda `MOBVNUM_RESPAWN_STONE_STEP2` (taş) veya `MOBVNUM_RESPAWN_BOSS_STEP3` (küçük boss) öldürüldüğünde çağrılır. Öldürülen taş/boss sayısını (`kill_stonesCount`, `kill_bossesCount`) artırır. Belirli sayıda taş/boss öldürüldüğünde heykellerdeki koruma affect'lerini kaldırır ve yeni taşların doğması için bir bekleme süresi (`SetLastStoneKilledTime`) ayarlar.
        *   **`OnKillStatue(pkItem, pkChar, pkStatue)`:** 4. aşamada çağrılır. Oyuncunun mühür eşyasını (`SEAL_VNUM_KILL_STATUE`) siler. Heykele `AFF_STATUE4` (kırılmış) affect'ini ekler. Daha önce bu oyuncu heykel kırmamışsa `v_Already` listesine ekler. Tüm 4 heykel de kırıldıysa `DungeonResult()` çağırılır.
        *   **`DungeonResult()`:** Heykel kırma süresi olayını iptal eder. Tüm heykeller kırılmışsa `pkBoss` ve heykelleri öldürür, ödül sandığını (`CHEST_VNUM`) doğurur ve `EndDungeon(true, true)` ile zindanı başarıyla sonlandırır. Kırılmamışsa `EndDungeon(false, false)` ile başarısız sonlandırır.
        *   **`GiveReward(pkChar, bReward)`:** Oyuncuya ödül sandığını (`REWARD_ITEMCHEST_VNUM_1` veya `_2`) verir ve `v_Rewards` listesine ekler.
    *   **`CMgr` Metotları:**
        *   **`Initialize()`, `Destroy()`:** `m_RegGuilds` haritasını temizler, `Destroy` içinde aktif zindanların `CMgrMap::Destroy()` metodunu ve haritalarını yok eder.
        *   **`Register(pkChar, iRes1, iRes2)`:** Zindan kaydı mantığını içerir. Lonca seviyesi, merdiven puanı, bekleme süresi kontrolleri yapar. Başarılıysa `SECTREE_MANAGER::CreatePrivateMap` ile özel harita oluşturur, `CMgrMap` nesnesi yaratır, `m_RegGuilds`'e ekler ve loncanın zindan bilgilerini DB'ye kaydetmesi için `pkGuild->RequestDungeon()` çağırır.
        *   **`Enter(pkGuild, pkChar, iLimit)`:** Oyuncunun zindana girişini yönetir. Gerekirse (başka bir kanalda kayıtlıysa ve bu kanalda yoksa) yeni bir zindan örneği oluşturur. Katılımcı limiti, zindan aşaması gibi kontroller yapar. Oyuncuyu zindan haritasına ışınlar (`pkChar->WarpSet`).
        *   **`Leave(pkGuild, pkChar, bSendOut)`, `LeaveRequest(pkGuild, pkChar)**: Oyuncuyu zindandan çıkarır. `pMap->Partecipant(false, ...)` ile katılımcı listesinden siler ve `WarpOut` ile dışarı ışınlar.
        *   **`OnKillCommon(pkMonster, pkChar, pkGuild)`:** 3. aşamada öldürülen normal yaratıklardan %30 şansla `SEAL_VNUM_KILL_STATUE` (heykel kırma mührü) düşürür.
        *   **`OpenRanking(pkChar)`:** `log.meley_dungeon` tablosundan en iyi 5 dereceyi çeker ve oyuncuya gönderir.
        *   **`MemberRemoved(pkChar, pkGuild)`:** Loncasından atılan oyuncu Meley haritasındaysa onu zindandan çıkarır.
        *   **`GuildRemoved(pkGuild)`:** Lonca dağıtıldığında aktif Meley zindanı varsa sonlandırır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `MeleyLair.h`, `db.h`, `log.h`, `item.h`, `char.h`, `utils.h`, `party.h`, `regen.h`, `config.h`, `packet.h`, `motion.h`, `item_manager.h`, `guild_manager.h`, `start_position.h`, `locale_service.h`, `boost/lexical_cast.hpp`.

### `minigame.h`

*   **Amaç:** Oyun içi mini oyunları ve bazı olayları (`__EVENT_BANNER_FLAG__` gibi) yönetmek için `CMiniGameManager` singleton sınıfını tanımlar. Farklı mini oyunlar ve olaylar için işlevsellik, `#ifdef` direktifleri kullanılarak modüler bir şekilde eklenmiştir.
*   **Temel İşlevler/İçerik:**
    *   **`EMapIndex` Enum'u:** Üç imparatorluğun ana harita indekslerini (`EMPIRE_A`, `EMPIRE_B`, `EMPIRE_C`) tanımlar.
    *   **`CMiniGameManager` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:**
        *   **Genel Metotlar:**
            *   `Initialize()`, `Destroy()`: Yöneticinin başlatılması ve yok edilmesi (şu anki implementasyonda boş).
            *   `SpawnEventNPC(DWORD dwVnum)`: Belirtilen VNUM'a sahip olay NPC'sini üç imparatorluğun ana haritalarındaki belirlenmiş konumlara (eğer zaten yoksa) doğurur.
        *   **Banner Olayı Metotları (`__EVENT_BANNER_FLAG__`):**
            *   `InitializeBanners()`: `data/banner/list.txt` dosyasından banner VNUM'larını ve isimlerini yükler (`BannerMap`).
            *   `SpawnBanners(int iEnable, const char* c_szBannerName)`: Belirtilen VNUM veya isimdeki banner'ları üç imparatorlukta (`data/banner/[a/b/c]/[banner_name].txt` dosyalarındaki konumlara göre) doğurur veya kaldırır (`regen_do` kullanarak). Aktif banner VNUM'unu `banner` event flag'ine kaydeder.
        *   **Kralı Yakala Mini Oyunu Metotları (`__MINI_GAME_CATCH_KING__`):**
            *   `MiniGameCatchKing(LPCHARACTER ch, const char* data, size_t uiBytes)`: İstemciden gelen Kralı Yakala paketi (`HEADER_GC_MINI_GAME_CATCH_KING`) için ana işleyici. Alt başlığa (`bSubheader`) göre ilgili fonksiyonu çağırır.
            *   `MiniGameCatchKingEventInfo(LPCHARACTER pkChar)`: Oyuncuya Kralı Yakala olayının aktif olup olmadığını bildirir.
            *   `InitializeMiniGameCatchKing(int iEnable)`: Kralı Yakala olayını başlatır veya bitirir. Olay NPC'sini (20506) doğurur/kaldırır ve tüm online oyunculara `MiniGameCatchKingEventInfo` ile bilgi gönderir.
            *   `InitializeMiniGameCatchKingEndTime(int iEndTime)`: Olayın bitiş zamanını ayarlar.
            *   `MiniGameCatchKingCheckEnd()`: Olayın bitiş zamanını kontrol eder ve gerekirse olayı sonlandırır (`enable_catch_king_event` flag'ini 0 yapar).
            *   `MiniGameCatchKingStartGame(LPCHARACTER pkChar, BYTE bSetCount)`: Oyuncu için Kralı Yakala oyununu başlatır. Gerekli eşya (Kart Seti) ve Yang kontrolü yapar, kartları karıştırır, oyun durumunu ayarlar ve başlangıç bilgisini istemciye gönderir.
            *   `MiniGameCatchKingDeckCardClick(LPCHARACTER pkChar)`: Oyuncu desteye tıkladığında çağrılır. Kalan kart sayısına göre oyuncunun eline bir kart (`bHandCard`) verir ve istemciye bildirir.
            *   `MiniGameCatchKingFieldCardClick(LPCHARACTER pkChar, BYTE bFieldPos)`: Oyuncu oyun alanındaki bir karta tıkladığında çağrılır. Elindeki kart ile tıklanan kartı karşılaştırır, puanı hesaplar, kartların durumunu (açık/kapalı, yok edilmiş) günceller, yatay/dikey sıra kontrolü yapar ve sonucu (`SUBHEADER_GC_CATCH_KING_RESULT_FIELD`) istemciye gönderir. Oyun bittiyse (`bHandCard == 6`) kapalı kartları (`SUBHEADER_GC_CATCH_KING_SET_END_CARD`) açar.
            *   `MiniGameCatchKingGetReward(LPCHARACTER pkChar)`: Oyun bittiğinde çağrılır. Skora göre ödül sandığını (`dwRewardVnum`) belirler, skoru kaydeder (`MiniGameCatchKingRegisterScore`), ödülü verir (`AutoGiveItem`) ve oyun durumunu sıfırlar. Sonucu (`SUBHEADER_GC_CATCH_KING_REWARD`) istemciye gönderir.
            *   `MiniGameCatchKingRegisterScore(LPCHARACTER pkChar, DWORD dwScore)`: Oyuncunun skorunu `log.catck_king_event` tablosuna kaydeder (eğer yeni skor öncekinden yüksekse `max_score`'u günceller, her zaman `total_score`'u artırır).
            *   `MiniGameCatchKingGetScore(lua_State* L, bool isTotal)`: Lua için, `log.catck_king_event` tablosundan en yüksek skor veya toplam skor sıralamasını (ilk 10) çeker ve tablo olarak döndürür.
            *   `MiniGameCatchKingGetMyScore(LPCHARACTER pkChar)`: Oyuncunun `log.catck_king_event` tablosundaki en yüksek skorunu döndürür.
        *   **Korunan Üyeler (`protected`):**
            *   `BannerMap` (BannerMapType - `std::map<DWORD, std::string>`): Banner VNUM -> Banner Adı eşleşmesi (`__EVENT_BANNER_FLAG__`).
            *   `m_bIsLoadedBanners` (bool): Banner listesinin yüklenip yüklenmediği (`__EVENT_BANNER_FLAG__`).
            *   `iCatchKingEndTime` (int): Kralı Yakala olayının bitiş zamanı (Unix timestamp) (`__MINI_GAME_CATCH_KING__`).
*   **Bağlantılı Dosyalar:** `minigame.cpp`, `minigame_catch_king.cpp` (uygulamalar), `stdafx.h`, `boost/unordered_map.hpp`, `../../common/stl.h`, `../../common/length.h`, `../../common/tables.h`, `packet.h`, `questmanager.h`.

### `minigame.cpp`

*   **Amaç:** `minigame.h`'de bildirilen `CMiniGameManager` sınıfının temel metotlarını ve `#ifdef` ile etkinleştirilmiş bazı olayların (örn. `__EVENT_BANNER_FLAG__`) implementasyonunu içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı/Yıkıcı:** İlgili olayların değişkenlerini başlatır/temizler (Kralı Yakala bitiş zamanı, banner yüklenme durumu).
    *   **`Initialize()`, `Destroy()`:** Şu anki implementasyonda boşlar.
    *   **`SpawnEventNPC(dwVnum)`:**
        *   Belirtilen `dwVnum`'a sahip NPC'nin üç imparatorluk haritasında (`EMPIRE_A`, `EMPIRE_B`, `EMPIRE_C`) zaten var olup olmadığını kontrol eder (`CHARACTER_MANAGER::instance().GetCharactersByRaceNum`).
        *   Eğer ilgili haritada NPC yoksa ve harita aktifse (`map_allow_find`), `CHARACTER_MANAGER::instance().SpawnMob` kullanarak NPC'yi `spawnPos` dizisindeki önceden tanımlanmış koordinatlara doğurur.
    *   **Banner Olayı (`__EVENT_BANNER_FLAG__`):**
        *   **`InitializeBanners()`:**
            *   `m_bIsLoadedBanners` false ise çalışır.
            *   `data/banner/list.txt` dosyasını `cCsvTable` ile okur.
            *   Her satırdan Banner VNUM'unu ve adını alıp `BannerMap`'e ekler.
            *   `m_bIsLoadedBanners`'ı true yapar.
            *   `banner` quest flag'i ayarlıysa, ilgili banner'ı `SpawnBanners` ile doğurur.
        *   **`SpawnBanners(iEnable, c_szBannerName)`:**
            *   Banner listesi yüklenmemişse `InitializeBanners()` çağırır.
            *   `iEnable` (VNUM) veya `c_szBannerName`'e göre `BannerMap`'ten ilgili banner'ı bulur.
            *   Önce mevcut banner NPC'lerini (`CHARACTER_MANAGER::instance().GetCharactersByRaceNum`) bulup `M2_DESTROY_CHARACTER` ile siler ve `banner` quest flag'ini 0 yapar.
            *   Eğer `iEnable` (veya bulunan VNUM) 0'dan büyükse:
                *   `banner` quest flag'ini yeni VNUM ile günceller.
                *   Üç imparatorluk haritası için (`EMPIRE_A`, `EMPIRE_B`, `EMPIRE_C`) ilgili banner konum dosyasını (`data/banner/[a/b/c]/[banner_name].txt`) `regen_do` fonksiyonu ile çalıştırarak banner NPC'lerini haritalara yerleştirir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `minigame.h`, `../../common/length.h`, `../../common/tables.h`, `p2p.h`, `locale_service.h`, `char.h`, `desc_client.h`, `desc_manager.h`, `buffer_manager.h`, `packet.h`, `questmanager.h`, `questlua.h`, `start_position.h`, `char_manager.h`, `item_manager.h`, `sectree_manager.h`, `regen.h`, `log.h`, `db.h`, `target.h`, `party.h`.

### `minigame_catch_king.cpp`

*   **Amaç:** (`__MINI_GAME_CATCH_KING__` tanımlıysa) Kralı Yakala mini oyununun tüm mantığını `CMiniGameManager` sınıfının üye fonksiyonları olarak uygular. Oyuncu etkileşimlerini (istemci paketleri), oyunun başlatılmasını, kart çekme ve açma mekaniklerini, skor hesaplamasını, ödüllendirmeyi ve olay durum yönetimini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`MiniGameCatchKing(ch, data, uiBytes)`:**
        *   İstemciden gelen `HEADER_GC_MINI_GAME_CATCH_KING` paketini işler.
        *   `TPacketCGMiniGameCatchKing` yapısındaki `bSubheader` değerine göre ilgili fonksiyonu çağırır:
            *   `0`: `MiniGameCatchKingStartGame`
            *   `1`: `MiniGameCatchKingDeckCardClick`
            *   `2`: `MiniGameCatchKingFieldCardClick`
            *   `3`: `MiniGameCatchKingGetReward`
    *   **`MiniGameCatchKingPacketFunc` (Struct):** Olay başladığında/bittiğinde tüm online oyunculara bilgi göndermek için `std::for_each` ile kullanılan bir functor.
    *   **`InitializeMiniGameCatchKing(iEnable)`:**
        *   Olay etkinleştirildiğinde (`iEnable` true): Olay NPC'sini (20506) `SpawnEventNPC` ile doğurur ve `MiniGameCatchKingEventInfo` ile tüm oyunculara bilgi gönderir.
        *   Olay devre dışı bırakıldığında (`iEnable` false): Olay NPC'lerini haritalardan siler (`CHARACTER_MANAGER::instance().GetCharactersByRaceNum`) ve tüm oyunculara bilgi gönderir.
    *   **`MiniGameCatchKingStartGame(pkChar, bSetCount)`:**
        *   Oyuncunun zaten oyunda olup olmadığını, olayın aktif olup olmadığını kontrol eder.
        *   Bahis miktarı (`bSetCount`) ve oyuncunun yeterli Yang (`GetGold`) / Kart Seti (`CountSpecifyItem(79604)`) sahibi olup olmadığını kontrol eder.
        *   Gerekli eşya ve Yang'ı oyuncudan alır (`RemoveSpecifyItem`, `PointChange`).
        *   Oyun alanını (`m_vecFieldCards`) oluşturur: Belirli sayıda farklı kart türünü (1-6, 6=Kral) ekler ve `std::shuffle` ile karıştırır.
        *   Oyuncunun karakter nesnesinde (`LPCHARACTER`) oyun durumunu ayarlar (`MiniGameCatchKingSetFieldCards`, `MiniGameCatchKingSetBetNumber`, `MiniGameCatchKingSetHandCardLeft`, `MiniGameCatchKingSetGameStatus`).
        *   Oyuncuya oyunun başladığını ve mevcut en yüksek skorunu (`SUBHEADER_GC_CATCH_KING_START`) gönderir.
    *   **`MiniGameCatchKingDeckCardClick(pkChar)`:**
        *   Oyuncunun oyunda olup olmadığını, olay aktifliğini, elinde zaten kart olup olmadığını kontrol eder.
        *   Kalan kart sayısına (`MiniGameCatchKingGetHandCardLeft`) göre bir sonraki kartı belirler (12-8: 1, 7-6: 2, 5-4: 3, 3: 4, 2: 5, 1: 6).
        *   Oyuncunun elindeki kartı (`MiniGameCatchKingSetHandCard`) ve kalan kart sayısını günceller.
        *   Oyuncuya elindeki kartı (`SUBHEADER_GC_CATCH_KING_SET_CARD`) gönderir.
    *   **`MiniGameCatchKingFieldCardClick(pkChar, bFieldPos)`:**
        *   Gerekli kontrolleri yapar (oyun durumu, olay aktifliği, geçerli pozisyon, elde kart olması, alan kartının kapalı olması).
        *   Eldeki kart (`bHandCard`) ile alan kartını (`filedCard.bIndex`) karşılaştırır:
            *   **Normal Kartlar (1-4):** `bHandCard < filedCard`: Puan yok, eldeki kart gider. `bHandCard == filedCard`: Puan = `index * 10`, eldeki kart gider, alan kartı açılır. `bHandCard > filedCard`: Puan = `index * 10`, eldeki kart kalır, alan kartı açılır.
            *   **Bomba Kartı (5):** Etrafındaki 8 komşu kartta 5 olup olmadığını kontrol eder. Varsa: Puan yok, eldeki kart gider, alan kartı kapalı kalır (eğer `bHandCard >= filedCard`). Yoksa: Normal kartlar gibi davranır.
            *   **Kral Kartı (6):** `bHandCard == filedCard` (yani alan kartı da Kral ise): Puan = 100, eldeki kart gider, alan kartı açılır. Değilse: Puan yok, eldeki kart gider, alan kartı kapalı kalır.
        *   Alan kartı açılacaksa (`bKeepFieldCard`) oyuncunun `m_vecCatchKingFieldCards` listesindeki ilgili kartın `bIsExposed` bayrağını true yapar.
        *   Açılan kartla birlikte yatay ve/veya dikey sıra tamamlanıp tamamlanmadığını kontrol eder. Tamamlanan her sıra için +10 puan ekler.
        *   Hesaplanan puanı oyuncunun skoruna (`MiniGameCatchKingSetScore`) ekler.
        *   Eldeki kart yok edilecekse (`bDestroyCard`) oyuncunun elindeki kartı sıfırlar (`MiniGameCatchKingSetHandCard(0)`).
        *   Eğer Kral kartı (6) oynandıysa ve skor 10'dan büyükse ödül alınabileceğini (`bGetReward`) işaretler ve oyunun bittiğini (`isTheEnd`) belirtir.
        *   Oyunun sonucunu (puan, açılan kart, yok edilen kart, ödül durumu vb.) `TPacketGCMiniGameCatchKingResult` ile (`SUBHEADER_GC_CATCH_KING_RESULT_FIELD`) istemciye gönderir.
        *   Eğer oyun bittiyse (`isTheEnd`), kapalı kalan tüm kartları `SUBHEADER_GC_CATCH_KING_SET_END_CARD` paketiyle istemciye gönderir.
    *   **`MiniGameCatchKingGetReward(pkChar)`:**
        *   Gerekli kontrolleri yapar (oyun durumu, olay aktifliği, elde kart olmaması, kalan kart olmaması).
        *   Skora (`dwScore`) göre ödül VNUM'unu (`dwRewardVnum`) belirler (10-399: 50930, 400-549: 50929, >=550: 50928).
        *   Skoru `MiniGameCatchKingRegisterScore` ile DB'ye kaydeder.
        *   Ödül VNUM'u varsa (`dwRewardVnum > 0`), ödülü bahis sayısı (`MiniGameCatchKingGetBetNumber`) kadar verir (`AutoGiveItem`), oyun durumunu sıfırlar ve başarı kodunu (0) ayarlar. Yoksa başarısızlık kodunu (1) ayarlar.
        *   Sonucu (`SUBHEADER_GC_CATCH_KING_REWARD`) istemciye gönderir.
    *   **`MiniGameCatchKingRegisterScore(pkChar, dwScore)`:**
        *   `log.catck_king_event` tablosundan oyuncunun mevcut en yüksek skorunu (`max_score`) sorgular.
        *   Kayıt varsa ve yeni skor (`dwScore`) eskisinden yüksekse `UPDATE` sorgusu ile `max_score`'u günceller ve `total_score`'u artırır. Yeni skor düşükse sadece `total_score`'u artırır.
        *   Kayıt yoksa `REPLACE INTO` sorgusu ile yeni kayıt ekler.
    *   **`MiniGameCatchKingGetScore(L, isTotal)` (Lua Fonksiyonu):**
        *   `log.catck_king_event` tablosundan `isTotal` parametresine göre ya `total_score` ya da `max_score`'a göre sıralı ilk 10 oyuncunun bilgilerini (isim, imparatorluk, skor) çeker ve tablo olarak döndürür.
    *   **`MiniGameCatchKingGetMyScore(pkChar)`:** `log.catck_king_event` tablosundan oyuncunun `max_score` değerini sorgular ve döndürür.
    *   **`MiniGameCatchKingEventInfo(pkChar)`:** `enable_catch_king_event` quest flag'inin değerini okur ve `SUBHEADER_GC_CATCH_KING_EVENT_INFO` paketiyle oyuncuya gönderir.
    *   **`MiniGameCatchKingCheckEnd()`:** Mevcut zamanı `iCatchKingEndTime` ile karşılaştırır. Eğer süre dolmuşsa `enable_catch_king_event` flag'ini 0 yapar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `<random>`, `<iterator>`, `<iostream>`, `<algorithm>`, `<ctime>`, `minigame.h`, `../../common/length.h`, `../../common/tables.h`, `p2p.h`, `locale_service.h`, `char.h`, `desc_client.h`, `desc_manager.h`, `buffer_manager.h`, `packet.h`, `questmanager.h`, `questlua.h`, `start_position.h`, `char_manager.h`, `item_manager.h`, `sectree_manager.h`, `regen.h`, `log.h`, `db.h`, `target.h`, `party.h`.

### `mining.h`

**Amacı:** Metin2 oyun sunucusundaki madencilik (mining) sistemiyle ilgili fonksiyon bildirimlerini ve temel yapıları içerir. Bu başlık dosyası, madencilik işlemlerinin, kazma (pickaxe) geliştirme ve cevher (ore) işleme gibi özelliklerin arayüzünü tanımlar.

**Temel İşlevleri/İçeriği:**
*   `mining` namespace'i altında madencilikle ilgili tüm fonksiyonları gruplar.
*   Madencilik olaylarını (event) oluşturmak için `CreateMiningEvent` fonksiyonunu deklare eder.
*   Maden damarlarından ham cevher VNUM'ını almak için `GetRawOreFromLoad` fonksiyonunu tanımlar.
*   Cevherlerin arıtılması (refine) işlemi için `OreRefine` fonksiyonunu içerir.
*   Maden toplarken elde edilecek parça sayısını belirleyen `GetFractionCount` fonksiyonunu barındırır.
*   Kazma (pickaxe) ile ilgili işlemler için fonksiyonlar sunar:
    *   `RealRefinePick`: Kazmayı geliştirmek için kullanılır.
    *   `CHEAT_MAX_PICK`: (Muhtemelen bir hile/test fonksiyonu) Kazmanın deneyimini maksimuma çıkarır.
*   Bir VNUM'ın maden damarı olup olmadığını kontrol eden `IsVeinOfOre` fonksiyonunu içerir.

---

### `mining.cpp`

**Amacı:** `mining.h` başlık dosyasında tanımlanan madencilik sistemi fonksiyonlarının implementasyonunu içerir. Oyuncuların maden damarlarından cevher toplamasını, kazmalarını geliştirmesini ve topladıkları cevherleri arıtmasını sağlayan mantığı gerçekleştirir.

**Temel İşlevleri/İçeriği:**
*   Madencilikle ilgili sabitler ve veri yapıları tanımlar:
    *   `MAX_ORE`: Desteklenen maksimum cevher türü sayısı.
    *   `MAX_FRACTION_COUNT`: Maden toplarken düşebilecek maksimum parça sayısı aralığı.
    *   `ORE_COUNT_FOR_REFINE`: Bir cevheri arıtmak için gereken ham cevher miktarı.
    *   `SInfo`: Cevher bilgilerini (damar VNUM, ham cevher VNUM, arıtılmış cevher VNUM) tutan yapı.
    *   `info[]`: Farklı cevher türleri için `SInfo` yapılarından oluşan bir dizi.
    *   `fraction_info[][]`: Maden toplarken düşecek parça sayısının olasılıklarını ve aralıklarını tutar.
    *   `PickGradeAddPct[]`: Kazmanın seviyesine göre madencilik başarısına eklenen yüzdeyi tutar.
    *   `SkillLevelAddPct[]`: Madencilik yeteneği seviyesine göre madencilik başarısına eklenen yüzdeyi tutar.
*   Madencilikle ilgili temel fonksiyonların implementasyonunu sağlar:
    *   `GetRawOreFromLoad`: Verilen maden damarı VNUM'ına karşılık gelen ham cevher VNUM'ını döndürür.
    *   `GetRefineFromRawOre`: Verilen ham cevher VNUM'ına karşılık gelen arıtılmış cevher VNUM'ını döndürür.
    *   `GetFractionCount`: Rastgele bir sayı üreterek ve `fraction_info` tablosunu kullanarak maden toplarken kaç parça düşeceğini belirler.
    *   `OreDrop`: Karakterin yakınına rastgele bir pozisyonda ham cevher düşürür.
    *   `GetOrePct`: Karakterin madencilik yeteneği seviyesi, kullandığı kazmanın seviyesi ve varsayılan bir yüzdeyi hesaba katarak madencilik başarı şansını hesaplar.
*   Kazma (pickaxe) ile ilgili fonksiyonların implementasyonunu içerir:
    *   `Pick_Check`: Bir eşyanın kazma olup olmadığını kontrol eder.
    *   `Pick_GetMaxExp`, `Pick_GetCurExp`: Kazmanın maksimum ve mevcut deneyim puanlarını döndürür.
    *   `Pick_IncCurExp`, `Pick_MaxCurExp`: Kazmanın mevcut deneyimini artırır veya maksimuma çıkarır.
    *   `Pick_SetPenaltyExp`: (Koşullu derleme `__REFINE_PICKAXE_RENEWAL__` ile) Başarısız kazma geliştirmesi durumunda deneyim cezası uygular.
    *   `Pick_Refinable`: Kazmanın geliştirilip geliştirilemeyeceğini kontrol eder (deneyim puanının maksimuma ulaşıp ulaşmadığına bakar).
    *   `Pick_IsPracticeSuccess`: Kazma deneyimi kazanma denemesinin başarılı olup olmayacağını belirler.
    *   `Pick_IsRefineSuccess`: Kazma geliştirme işleminin başarılı olup olmayacağını belirler.
    *   `RealRefinePick`: Kazma geliştirme işlemini gerçekleştirir. Başarılı olursa yeni, geliştirilmiş bir kazma verir; başarısız olursa eski kazmayı kaldırıp (veya deneyimini düşürüp) yerine bazen kırık bir eşya verebilir (eski veya `__REFINE_PICKAXE_RENEWAL__` implementasyonuna bağlı olarak).
    *   `CHEAT_MAX_PICK`: Bir komutla kazmanın deneyimini maksimuma çıkarır.
    *   `PracticePick`: Kazma kullanıldığında (madencilik yapıldığında) deneyim kazandırma mantığını işletir.
*   Madencilik olay (event) fonksiyonunu (`mining_event`) ve olay oluşturma (`CreateMiningEvent`) fonksiyonunu implemente eder. `mining_event` fonksiyonu, madencilik işlemi başladığında belirli bir süre sonra tetiklenir, başarı şansını hesaplar ve sonuca göre cevher düşürür veya başarısız mesajı verir. Ayrıca kazmaya deneyim kazandırır.
*   Cevher arıtma (`OreRefine`) fonksiyonunu implemente eder. Belirli sayıda ham cevheri, bir NPC aracılığıyla, belirli bir ücret ve başarı şansı karşılığında arıtılmış cevhere dönüştürür.
*   `IsVeinOfOre`: Bir VNUM'ın `info` dizisinde tanımlı maden damarlarından biri olup olmadığını kontrol eder.

**Orta Seviye İmpelentasyon Detayları:**
*   Maden damarları ve cevherler arasındaki ilişki `SInfo` yapısı ve `info` dizisi üzerinden yönetilir. Bu dizi, farklı cevher türlerinin VNUM'larını (damar, ham, arıtılmış) içerir.
*   Madencilik başarı şansı, karakterin madencilik yeteneği (`SKILL_MINING`), kullandığı kazmanın (`ITEM_PICK`) seviyesi ve `GetValue` ile erişilen eşya özelliklerine göre dinamik olarak hesaplanır.
*   Kazma geliştirme sistemi, kazmanın kendi içinde saklanan deneyim puanlarına (`GetSocket(0)`) ve `Value` değerlerine (geliştirme şansı, bir sonraki seviye için gereken deneyim vb.) dayanır.
*   Madencilik işlemi, bir olay (event) olarak tasarlanmıştır. `CreateMiningEvent` ile başlatılır ve belirli bir süre sonra `mining_event` fonksiyonu çalışarak sonucu belirler. Bu, işlemin anlık olmamasını ve bir bekleme süresi içermesini sağlar.
*   `#if defined(__CONQUEROR_LEVEL__)` ve `#if defined(__REFINE_PICKAXE_RENEWAL__)` gibi önişlemci direktifleri, oyunun farklı sürümleri veya yapılandırmaları için alternatif madencilik/kazma mekaniklerini etkinleştirmek veya devre dışı bırakmak için kullanılır.

### `monarch.h`

*   **Amaç:** İmparatorluk krallarını (monarch), krallık hazinelerini ve krallara özel yetenekleri (örn. "İmparatorun Kutsaması", "Aslan Kükremesi", "Ejderha Kayası") yönetmek için `CMonarch` singleton sınıfını tanımlar. Her imparatorluğun kralının kim olduğu, hazinesinde ne kadar para olduğu ve krallık yeteneklerinin aktif olup olmadığı gibi bilgileri tutar.
*   **Temel İşlevler/İçerik:**
    *   **`CMonarch` Sınıfı (Singleton):**
        *   Yapıcı/Yıkıcı: `CMonarch` nesnesini başlatır ve yok eder.
        *   `Initialize()`: Krallık yetenekleriyle ilgili durumları ve bekleme sürelerini sıfırlar.
        *   `HealMyEmpire(LPCHARACTER ch, DWORD price)`: Kralın, imparatorluk hazinesinden belirli bir bedel ödeyerek kendi imparatorluğundaki aynı haritada bulunan tüm oyuncuların HP ve SP'sini tamamen doldurmasını sağlar.
        *   `SetMonarchInfo(TMonarchInfo* pInfo)`: Veritabanından yüklenen krallık bilgilerini (`TMonarchInfo` yapısı: her imparatorluk için kral PID'si ve hazine miktarı) ayarlar.
        *   `IsMonarch(DWORD pid, BYTE bEmpire)`: Verilen PID'nin belirtilen imparatorluğun kralı olup olmadığını kontrol eder.
        *   `IsMoneyOk(int price, BYTE bEmpire)`: Belirtilen imparatorluğun hazinesinde `price` kadar para olup olmadığını kontrol eder.
        *   `SendtoDBAddMoney(int Money, BYTE bEmpire, LPCHARACTER ch)`: İmparatorluk hazinesine para eklenmesi için veritabanına istek gönderir.
        *   `SendtoDBDecMoney(int Money, BYTE bEmpire, LPCHARACTER ch)`: İmparatorluk hazinesinden para düşülmesi için veritabanına istek gönderir.
        *   `AddMoney(int Money, BYTE bEmpire)`: Yerel olarak (sunucu hafızasında) imparatorluk hazinesine para ekler.
        *   `DecMoney(int Money, BYTE bEmpire)`: Yerel olarak imparatorluk hazinesinden para düşer.
        *   `GetMoney(BYTE bEmpire)`: Belirtilen imparatorluğun hazinesindeki para miktarını döndürür.
        *   `GetMonarch()`: `TMonarchInfo` yapısının işaretçisini döndürür.
        *   `GetMonarchPID(BYTE Empire)`: Belirtilen imparatorluğun kralının PID'sini döndürür.
        *   `IsPowerUp(BYTE Empire)`: Belirtilen imparatorluk için "Aslan Kükremesi" (saldırı gücü artışı) yeteneğinin aktif olup olmadığını kontrol eder.
        *   `IsDefenceUp(BYTE Empire)`: Belirtilen imparatorluk için "Ejderha Kayası" (savunma gücü artışı) yeteneğinin aktif olup olmadığını kontrol eder.
        *   `GetPowerUpCT(BYTE Empire)`, `CheckPowerUpCT(BYTE Empire)`: "Aslan Kükremesi" yeteneğinin bekleme süresini alır veya kontrol eder.
        *   `GetDefenseUpCT(BYTE Empire)`, `CheckDefenseUpCT(BYTE Empire)`: "Ejderha Kayası" yeteneğinin bekleme süresini alır veya kontrol eder.
        *   `PowerUp(BYTE Empire, bool On)`: "Aslan Kükremesi" yeteneğini aktif veya pasif yapar ve bekleme süresini ayarlar.
        *   `DefenseUp(BYTE Empire, bool On)`: "Ejderha Kayası" yeteneğini aktif veya pasif yapar ve bekleme süresini ayarlar.
    *   **Özel (Private) Üyeler:**
        *   `m_MonarchInfo` (`TMonarchInfo`): Her imparatorluğun kral PID'sini ve hazine miktarını tutar.
        *   `m_PowerUp[4]`, `m_DefenseUp[4]` (int dizileri): Her imparatorluk için Aslan Kükremesi ve Ejderha Kayası yeteneklerinin aktif olup olmadığını tutar (muhtemelen bool olmalıydı).
        *   `m_PowerUpCT[4]`, `m_DefenseUpCT[4]` (int dizileri): Her imparatorluk için bu yeteneklerin bir sonraki kullanılabilir olacağı zamanı (pulse cinsinden) tutar.
    *   **Global Fonksiyon:**
        *   `IsMonarchWarpZone(int map_idx)`: Belirli bir harita indeksinin kraliyet ışınlanmaları için yasaklı bir bölge olup olmadığını kontrol eder.
*   **Bağlantılı Dosyalar:** `monarch.cpp` (uygulama), `../../common/tables.h` (`TMonarchInfo` tanımı), `stdafx.h`.

---

### `monarch.cpp`

*   **Amaç:** `monarch.h` dosyasında bildirilen `CMonarch` sınıfının metotlarını uygular. Krallık sistemiyle ilgili mantığı, özellikle krallık hazinesi yönetimi ve krallara özel yeteneklerin (İmparatorun Kutsaması, Aslan Kükremesi, Ejderha Kayası) kullanılmasını ve durumlarının takibini içerir.
*   **Temel İşlevleri/İçeriği:**
    *   Yapıcı (`CMonarch::CMonarch()`): `m_MonarchInfo`'yu sıfırlar ve `Initialize()` çağırır.
    *   `Initialize()`: Krallık yeteneklerinin durumlarını (`m_PowerUp`, `m_DefenseUp`) ve bekleme sürelerini (`m_PowerUpCT`, `m_DefenseUpCT`) sıfırlar.
    *   **`FHealMyEmpire` Struct (Functor):**
        *   `SECTREE_MANAGER::for_each` ile kullanılmak üzere tasarlanmıştır.
        *   Belirli bir imparatorluktaki (`m_bEmpire`) tüm oyuncu karakterlerinin (`IsPC()`) HP ve SP'sini tamamen doldurur ve efekt paketleri gönderir.
    *   **`HealMyEmpire(LPCHARACTER ch, DWORD price)`:**
        *   Kralın "İmparatorun Kutsaması" yeteneğini kullanmasını sağlar.
        *   Kullanıcının kral olup olmadığını (`IsMonarch`) ve yeteneğin bekleme süresinde olup olmadığını (`IsMCOK(CHARACTER::MI_HEAL)`) kontrol eder.
        *   İmparatorluk hazinesinde yeterli para olup olmadığını (`IsMoneyOk`) kontrol eder.
        *   Başarılı olursa, `FHealMyEmpire` functor'ını kullanarak kralın bulunduğu haritadaki kendi imparatorluğunun tüm oyuncularını iyileştirir.
        *   Hazine parasının düşürülmesi için DB'ye istek gönderir (`SendtoDBDecMoney`).
        *   Karakter için yetenek bekleme süresini ayarlar (`ch->SetMC(CHARACTER::MI_HEAL)`).
    *   **Krallık Bilgileri Yönetimi:**
        *   `SetMonarchInfo(TMonarchInfo* pInfo)`: Gelen `TMonarchInfo` verisini `m_MonarchInfo`'ya kopyalar. Bu genellikle sunucu başlangıcında DB'den krallık bilgileri yüklendiğinde çağrılır.
        *   `IsMonarch(DWORD pid, BYTE bEmpire)`: `m_MonarchInfo.pid` dizisinden kontrol eder.
        *   `GetMonarch()`: `m_MonarchInfo`'nun adresini döndürür.
        *   `GetMonarchPID(BYTE Empire)`: `m_MonarchInfo.pid` dizisinden ilgili imparatorluğun kralının PID'sini döndürür.
    *   **Krallık Hazinesi Yönetimi:**
        *   `IsMoneyOk(int price, BYTE bEmpire)`: `GetMoney(bEmpire)` ile mevcut parayı kontrol eder.
        *   `SendtoDBAddMoney(int Money, BYTE bEmpire, LPCHARACTER ch)`: `HEADER_GD_ADD_MONARCH_MONEY` başlığı ile DB'ye hazineye para ekleme paketi gönderir. Maksimum para limitini (2 milyar) kontrol eder.
        *   `SendtoDBDecMoney(int Money, BYTE bEmpire, LPCHARACTER ch)`: `HEADER_GD_DEC_MONARCH_MONEY` başlığı ile DB'ye hazineden para eksiltme paketi gönderir. Yetersiz bakiye durumunu kontrol eder.
        *   `AddMoney(int Money, BYTE bEmpire)`: `m_MonarchInfo.money` dizisindeki ilgili imparatorluğun hazine parasını artırır. Maksimum para limitini kontrol eder.
        *   `DecMoney(int Money, BYTE bEmpire)`: `m_MonarchInfo.money` dizisindeki parayı azaltır. Yetersiz bakiye durumunu kontrol eder.
        *   `GetMoney(BYTE bEmpire)`: `m_MonarchInfo.money` dizisinden ilgili imparatorluğun hazine parasını döndürür.
    *   **Krallık Yetenekleri (Aslan Kükremesi, Ejderha Kayası):**
        *   `IsPowerUp(BYTE Empire)`, `IsDefenceUp(BYTE Empire)`: İlgili imparatorluk için `m_PowerUp` veya `m_DefenseUp` bayrağının durumunu döndürür.
        *   `CheckPowerUpCT(BYTE Empire)`, `CheckDefenseUpCT(BYTE Empire)`: İlgili yeteneğin bekleme süresinin (`m_PowerUpCT`, `m_DefenseUpCT`) dolup dolmadığını `thecore_pulse()` ile karşılaştırarak kontrol eder.
        *   `PowerUp(BYTE Empire, bool On)`, `DefenseUp(BYTE Empire, bool On)`: İlgili yeteneğin durumunu (`m_PowerUp[Empire] = On`) ayarlar ve bekleme süresini (`thecore_pulse() + PASSES_PER_SEC(60 * 10)`) 10 dakika olarak ayarlar.
    *   **`IsMonarchWarpZone(int map_idx)` (Global Fonksiyon):**
        *   Belirli harita indekslerinin (örn. Zindanlar: 301-304, 351, 352; Şeytan Kulesi: 208; Örümcek Zindanı 2. Kat: 216; Örümcek Zindanı 3. Kat Sonu: 217) kralların özel ışınlanma yeteneklerini kullanamayacağı bölgeler olup olmadığını belirler. Harita indeksi 10000'den büyükse, 10000'e bölerek ana harita indeksini bulur.
*   **Orta Seviye İmpelentasyon Detayları:**
    *   Krallık bilgileri (`TMonarchInfo`) genellikle sunucu başlatıldığında veritabanından yüklenir ve `CMonarch::SetMonarchInfo` ile ayarlanır.
    *   Hazine işlemleri hem sunucu hafızasında (`AddMoney`, `DecMoney`) hem de veritabanında (`SendtoDBAddMoney`, `SendtoDBDecMoney`) senkronize edilir. Veritabanı işlemleri asenkron olarak tetiklenir.
    *   Krallık yeteneklerinin ("Aslan Kükremesi", "Ejderha Kayası") aktif olup olmadığı ve bekleme süreleri sunucu hafızasında tutulur. Bu yeteneklerin etkileri (saldırı/savunma artışı) muhtemelen karakterlerin stat hesaplamalarında `CMonarch::IsPowerUp` ve `CMonarch::IsDefenceUp` kontrol edilerek uygulanır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `monarch.h`, `char.h`, `sectree_manager.h`, `desc_client.h`, `dev_log.h`.

### `mount_up_grade.h`

*   **Amaç:** (`__RIDING_EXTENDED__` tanımlıysa) Bineklerin (atların) seviye atlama ve deneyim kazanma sistemini yönetmek için `CMountUpGrade` singleton sınıfını tanımlar. Bu sistem, normal at seviyelerinin ötesinde (21. seviyeden sonra) deneyim ve belirli eşyalar/para karşılığında seviye atlamayı mümkün kılar.
*   **Temel İşlevler/İçerik:**
    *   **`CMountUpGrade` Sınıfı (Singleton):**
        *   `OpenMountUpGrade(LPCHARACTER ch)`: Oyuncu için binek geliştirme penceresini açar ve mevcut durumu (seviye, deneyim, başarısızlık durumu) istemciye gönderir.
        *   `SetExp(LPCHARACTER ch)`: Belirli bir eşyayı (`HORSE_FEED_ITEM_ID`) kullanarak bineğe rastgele miktarda deneyim puanı ekler. Kullanılacak eşya sayısını, seviye atlamak için gereken maksimum deneyime ve mevcut deneyime göre optimize eder. Sonucu istemciye bildirir ve kullanılan eşya sayısını sohbet mesajıyla gösterir.
        *   `SetLevel(LPCHARACTER ch)`: Bineğin seviyesini yükseltmeye çalışır. Gerekli deneyim puanı, eşya (`HORSE_FEED_ITEM_ID` x `HORSE_FEED_LEVEL_COUNT`) ve Yang (`mount_up_grade_price_table`) kontrolü yapar. Seviye atlama işlemi rastgele bir başarısızlık şansına sahiptir (`EMountUpGradeRandFail`). Eğer önceki deneme başarısız olmuşsa (`IsMountUpGradeFail`), tekrar denemek için belirli miktarda Gem (`EMountUpGradeRetryGemCost`) gerekir. Başarılı veya başarısız olma durumunu ve sonucu istemciye bildirir, ilgili sohbet mesajını gönderir. Başarılı olursa `Update` çağırır.
        *   `Common(LPCHARACTER ch)`: `SetExp` ve `SetLevel` için ortak kontrolleri yapar (maksimum at seviyesi, binek üzerinde olmama durumu).
        *   `Chat(LPCHARACTER ch, const uint8_t type, const uint16_t value)`: Binek geliştirme sistemiyle ilgili özel sohbet mesajlarını (`HEADER_GC_MOUNT_UP_GRADE_CHAT`) istemciye gönderir.
        *   `Update(LPCHARACTER ch, const uint8_t level)`: Bineğin seviyesini (`SetHorseLevel`) bir artırır, deneyimini sıfırlar (`SetMountUpGradeExp(RESET)`), karakterin statlarını yeniden hesaplar (`ComputePoints`) ve yetenek seviyelerini günceller (`SkillLevelPacket`).
        *   `Send(LPCHARACTER ch, ...)`: Binek geliştirme penceresinin durumunu (`HEADER_GC_MOUNT_UP_GRADE`) istemciye gönderir (açma veya yenileme).
    *   **Enum'lar:**
        *   `EMountUpGradeChatType`: İstemciye gönderilecek farklı sohbet mesajı türlerini tanımlar (örn. yeterli eşya/para olmaması, başarı/başarısızlık).
        *   `EMountUpGradeItem`: Deneyim ve seviye atlama için gereken eşya (At Yemi) VNUM'ını ve adetlerini tanımlar.
        *   `EMountUpGradeRandExp`: Deneyim kazanırken eklenecek rastgele aralığı tanımlar.
        *   `EMountUpGradeRandFail`: Seviye atlarken başarısızlık şansını belirleyen rastgele aralığı tanımlar (0 veya 1).
        *   `EMountUpGradeRetryGemCost`: Başarısızlık sonrası tekrar deneme için gereken Gem miktarını (at seviyesine göre değişir) tanımlar.
        *   `EMountUpGradeFailType`: Başarısızlık durumunu (aktif/pasif) belirtir.
        *   `EMountUpGradeGCSubheaderType`, `EMountUpGradeCGSubheaderType`: Sunucu-istemci ve istemci-sunucu arasındaki paket alt başlıklarını tanımlar.
    *   **Global Sabit Diziler:**
        *   `mount_up_grade_exp_table[]`: Her bir at seviyesi için gereken maksimum deneyim puanını tutar (ilk 21 seviye görevlerle geçildiği için 0).
        *   `mount_up_grade_price_table[]`: Her bir at seviyesi için seviye atlama maliyetini (Yang) tutar (ilk 21 seviye 0).
*   **Bağlantılı Dosyalar:** `mount_up_grade.cpp` (uygulama), `../../common/service.h`, `../../common/length.h`, `stdafx.h`.

---

### `mount_up_grade.cpp`

*   **Amaç:** (`__RIDING_EXTENDED__` tanımlıysa) `mount_up_grade.h` dosyasında bildirilen `CMountUpGrade` sınıfının metotlarını uygular. Genişletilmiş binek (at) seviye atlama sisteminin tüm mantığını içerir: deneyim ekleme, seviye yükseltme denemesi, maliyet hesaplama, başarı/başarısızlık mekanizması ve istemci ile iletişim.
*   **Orta Seviye Implementasyon Detayları:**
    *   Yapıcı/Yıkıcı: Varsayılan (default) olarak tanımlanmışlardır.
    *   `OpenMountUpGrade(ch)`: Karakterin mevcut at seviyesini (`GetHorseLevel`), başarısızlık durumunu (`IsMountUpGradeFail`) ve deneyimini (`GetMountUpGradeExp`) alır ve `Send` ile istemciye `SUBHEADER_GC_MOUNT_UP_GRADE_OPEN` paketini gönderir.
    *   `Send(ch, ...)`: Verilen alt başlık ve verilerle `TPacketGCMountUpGrade` paketini oluşturur ve karakterin bağlantısına (`DESC`) gönderir. Geçersiz alt başlıkları kontrol eder.
    *   `Chat(ch, type, value)`: Verilen tür ve değer ile `TPacketGCMountUpGradeChat` paketini oluşturur ve karakterin bağlantısına gönderir.
    *   `SetExp(ch)`:
        *   `Common` kontrollerini yapar.
        *   Yeterli At Yemi (`HORSE_FEED_ITEM_ID`) olup olmadığını kontrol eder.
        *   Mevcut ve maksimum deneyimi alır (`mount_up_grade_exp_table`).
        *   Eklenecek rastgele deneyimi (`EMountUpGradeRandExp`) hesaplar.
        *   Maksimum deneyime ulaşmak için gereken At Yemi sayısını hesaplar ve oyuncunun sahip olduğu miktarla sınırlar (`used_item_count`).
        *   Yeni deneyimi hesaplar (`std::min` ile maksimum deneyimi aşmamasını sağlar) ve karaktere ayarlar (`ch->SetMountUpGradeExp`).
        *   Kullanılan At Yemlerini karakterden siler (`ch->RemoveSpecifyItem`).
        *   Kullanılan yem sayısını `Chat` ile bildirir.
        *   `Send` ile güncellenmiş durumu (`SUBHEADER_GC_MOUNT_UP_GRADE_REFRESH`) istemciye gönderir.
    *   `Common(ch)`: Maksimum at seviyesine ulaşılıp ulaşılmadığını (`>= EMisc::HORSE_MAX_LEVEL`) ve karakterin binek üzerinde olup olmadığını (`IsRiding`) kontrol eder. Binek üzerindeyse `Chat` ile uyarı gönderir.
    *   `SetLevel(ch)`:
        *   `Common` kontrollerini yapar.
        *   Mevcut/maksimum deneyimi ve seviye atlama maliyetini (`mount_up_grade_price_table`) alır.
        *   Yeterli deneyim, At Yemi (`HORSE_FEED_LEVEL_COUNT`) ve Yang olup olmadığını kontrol eder. Eksikse `Chat` ile bildirir.
        *   Eğer önceki deneme başarısız olmuşsa (`ch->IsMountUpGradeFail() > MOUNT_UP_GRADE_FAIL_OFF`):
            *   Gerekli Gem miktarını at seviyesine göre belirler (`EMountUpGradeRetryGemCost`).
            *   Yeterli Gem olup olmadığını kontrol eder. Eksikse `Chat` ile bildirir.
            *   Gem'i düşer (`ch->PointChange(POINT_GEM, -gemCost)`).
            *   Başarısızlık durumunu sıfırlar (`ch->SetMountUpGradeFail(RESET)`).
            *   `Update` çağırarak seviyeyi hemen artırır (Başarısızlık sonrası tekrar deneme = garantili başarı).
        *   Eğer önceki deneme başarısız değilse (veya Gem ile sıfırlandıysa):
            *   Başarısızlık şansını rastgele belirler (`number(MIN, MAX)` -> 0 veya 1).
            *   Eğer deneyim yeterliyse başarısızlık durumunu ayarlar (`ch->SetMountUpGradeFail(fail)`).
            *   Eğer başarısızlık durumu 0 ise (`fail < MOUNT_UP_GRADE_FAIL_ON`):
                *   `Update` çağırarak seviyeyi artırır.
        *   Her durumda Yang ve At Yemi maliyetini düşer.
        *   Sonuca göre (başarılı veya başarısız) `Chat` ile mesaj gönderir ve `Send` ile güncellenmiş durumu istemciye bildirir.
    *   `Update(ch, level)`:
        *   Karakterin binek deneyimini sıfırlar (`ch->SetMountUpGradeExp(RESET)`).
        *   Karakterin at seviyesini bir artırır (`ch->SetHorseLevel(level + 1)`).
        *   Karakterin statlarını yeniden hesaplatır (`ch->ComputePoints()`).
        *   Yetenek seviyesi paketini gönderir (`ch->SkillLevelPacket()`). Bu, muhtemelen at yeteneklerinin güncellenmesi içindir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `char.h`, `desc.h`, `mount_up_grade.h`, `<algorithm>`.

---

### `over9refine.h`

*   **Amaç:** Eşyaların +9 seviyesinin üzerine geliştirilmesi ("Over 9 Refine") sistemini yönetmek için `COver9RefineManager` singleton sınıfını tanımlar. Hangi +9 eşyanın hangi +10 (veya üstü) eşyaya dönüşebileceğini ve bu geliştirme işleminin koşullarını yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`COver9RefineManager` Sınıfı (Singleton):**
        *   **Private Üye:**
            *   `m_mapItem` (`OVER9ITEM_MAP` - `boost::unordered_map<DWORD, DWORD>`): +9 üstü geliştirmeye uygun eşyaların VNUM eşleşmelerini tutar (Anahtar: +9 eşya VNUM, Değer: +10 veya üstü eşya VNUM).
        *   **Public Metotlar:**
            *   `enableOver9Refine(DWORD dwVnumFrom, DWORD dwVnumTo)`: Belirli bir +9 eşyasının (`dwVnumFrom`) hangi +10 (veya üstü) eşyaya (`dwVnumTo`) geliştirilebileceğini sisteme kaydeder. Bu genellikle sunucu başlangıcında bir yapılandırma dosyasından okunarak çağrılır.
            *   `canOver9Refine(DWORD dwVnum)`: Verilen VNUM'a sahip bir eşyanın +9 üstü geliştirmeye uygun olup olmadığını kontrol eder. Dönüş değerleri:
                *   `1`: Eşya `dwVnum` tam olarak `m_mapItem`'da kayıtlıdır (yani +9'dur ve +10'a geçebilir).
                *   `2`: Eşya `dwVnum`'ın son hanesi 9 değildir, ancak bu eşyanın temel VNUM'ı (`dwVnum - dwVnum % 10`), `m_mapItem`'daki bir hedefin temel VNUM'ı ile eşleşir (yani eşya zaten +10 veya üzeridir ve normal geliştirmeye devam edebilir).
                *   `0`: Eşya +9 üstü geliştirme sistemiyle ilgili değildir veya +9 olup henüz dönüşümü tanımlanmamıştır.
            *   `Change9ToOver9(LPCHARACTER pChar, LPITEM item)`: +9 seviyesindeki bir eşyayı, `m_mapItem`'da tanımlı olan +10 (veya üstü) karşılığına dönüştürür. Yeni eşyayı oluşturur, eski eşyanın soketlerini ve efsunlarını kopyalar, envanterde yer kontrolü yapar, eski eşyayı siler ve yeni eşyayı envantere ekler. İşlemi loglar.
            *   `Over9Refine(LPCHARACTER pChar, LPITEM item)`: Zaten +10 veya üzerinde olan bir eşyayı, `item_proto`'da tanımlı `refined_vnum`'a göre bir sonraki seviyeye geliştirir. İşlem `Change9ToOver9` ile benzerdir, ancak hedef VNUM'ı `m_mapItem` yerine `item->GetRefinedVnum()`'dan alır.
            *   `GetMaterialVnum(DWORD baseVnum)`: +9 üstü geliştirme için gereken temel malzeme VNUM'ını döndürmek için kullanılır. Verilen `baseVnum`'ın `m_mapItem`'daki durumuna göre (+9 mu, +10 mu) ilgili temel VNUM'ı (son hanesi 0 olan) bulur.
*   **Bağlantılı Dosyalar:** `over9refine.cpp` (uygulama), `<boost/unordered_map.hpp>`, `stdafx.h`.

---

### `over9refine.cpp`

*   **Amaç:** `over9refine.h` dosyasında bildirilen `COver9RefineManager` sınıfının metotlarını uygular. +9 üstü geliştirme sisteminin mantığını içerir: Geliştirme tanımlarını kaydetme, bir eşyanın geliştirilip geliştirilemeyeceğini kontrol etme, +9'dan +10'a geçişi veya +10 ve üzerindeki geliştirmeleri gerçekleştirme ve geliştirme için gereken temel malzeme VNUM'ını bulma.
*   **Orta Seviye Implementasyon Detayları:**
    *   `enableOver9Refine(dwVnumFrom, dwVnumTo)`: Verilen +9 VNUM (`dwVnumFrom`) ve hedef +10 (veya üstü) VNUM (`dwVnumTo`) çiftini `m_mapItem` haritasına ekler.
    *   `canOver9Refine(dwVnum)`:
        *   Önce `dwVnum`'ın `m_mapItem` haritasında anahtar olarak olup olmadığına bakar. Varsa, bu eşya +9'dur ve +10'a geçebilir (dönüş 1).
        *   Eğer `dwVnum`'ın son hanesi 9 değilse (yani +9 değilse), `dwVnum`'ın son hanesini sıfırlayarak temel VNUM'ı bulur. `m_mapItem` haritasındaki tüm değerleri (hedef VNUM'lar) dolaşır. Eğer bu temel VNUM, haritadaki bir hedef VNUM'a eşitse, bu eşya zaten +10 veya üzerindedir ve normal geliştirmeye devam edebilir (dönüş 2).
        *   Diğer durumlarda 0 döndürür.
    *   `Change9ToOver9(pChar, item)`:
        *   Verilen eşyanın (`item`) VNUM'ının `m_mapItem`'da anahtar olarak olup olmadığını kontrol eder. Yoksa `false` döner.
        *   Haritadan hedef VNUM'ı (`dwVnum`) alır.
        *   `ITEM_MANAGER::CreateItem` ile hedef VNUM'a sahip yeni bir eşya (`over9`) oluşturur.
        *   Eski eşyanın soketlerini (`CopySocketTo`) ve efsunlarını (`CopyAttributeTo`) yeni eşyaya kopyalar.
        *   Karakterin envanterinde yeni eşya için boş yer arar (`GetEmptyInventory`). Boş yer yoksa `false` döner.
        *   Eski eşyayı karakterden kaldırır (`RemoveFromCharacter`). Bu genellikle eşyayı yok eder.
        *   Yeni eşyayı (`over9`) karakterin envanterine ekler (`AddToCharacter`).
        *   İşlemi `LogManager::instance().ItemLog` ile "REFINE OVER9" olarak loglar. Başarılı olursa `true` döner.
    *   `Over9Refine(pChar, item)`:
        *   Verilen eşyanın `item_proto`'sundaki `refined_vnum` değerini alır. Bu değer 0 ise (yani daha fazla geliştirilemiyorsa) `false` döner.
        *   `Change9ToOver9` ile aynı mantıkla devam eder: yeni eşyayı oluşturur, özellikleri kopyalar, yer kontrolü yapar, eskiyi siler, yeniyi ekler ve loglar.
    *   `GetMaterialVnum(baseVnum)`:
        *   Verilen `baseVnum`'ın `m_mapItem`'da anahtar olup olmadığına bakar. Varsa, bu +9 bir eşyadır ve malzeme olarak kendi temel VNUM'ı (`baseVnum - (baseVnum % 10)`) kullanılır.
        *   Yoksa, `baseVnum`'ın son hanesini sıfırlayarak temel VNUM'ı bulur. `m_mapItem`'ı dolaşarak bu temel VNUM'a dönüşen +9 eşyanın (`iter->first`) temel VNUM'ını (`iter->first - (iter->first % 10)`) malzeme olarak döndürür.
        *   Eşleşme bulunamazsa 0 döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `log.h`, `char.h`, `item_manager.h`, `item.h`, `over9refine.h`.

### `TempleOchao.h`

*   **Amaç:** (`__MT_THUNDER_DUNGEON__` tanımlıysa) Ochao Tapınağı (veya Yıldırım Zindanı) adı verilen özel bir zindan/etkinlik alanını yönetmek için `TempleOchao` isim alanı altında `CMgr` singleton sınıfını ve ilgili sabitleri tanımlar. Bu sistem, belirli bir haritada (örn. `MAP_INDEX = 354`) rastgele odalarda beliren bir koruyucu (`GUARDIAN`) ve öldürüldüğünde ortaya çıkan bir portal (`PORTAL`) mekaniğini yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`TempleOchao` Namespace:**
        *   **Enum Sabitleri:**
            *   `MAP_INDEX`: Zindanın harita indeksi.
            *   `ROOMS`: Zindandaki toplam oda sayısı.
            *   `GUARDIAN`: Koruyucu canavarının VNUM'u.
            *   `PORTAL`: Koruyucu öldüğünde çıkan portalın VNUM'u.
            *   `PORTAL_VANISH_TIME`: Portalın aktif kalma süresi (saniye).
            *   `CHECK_ACTIVITY`: Koruyucunun aktivite durumunun kontrol edilme sıklığı (saniye).
            *   `NO_ACTIVITY`: Koruyucunun hiç saldırı almazsa yer değiştireceği süre (saniye).
            *   `ATTACKED`: Koruyucu saldırı aldıktan sonra yer değiştirmeden önce bekleyeceği süre (saniye).
        *   **`CMgr` Sınıfı (Singleton):**
            *   **Public Metotlar:**
                *   `Initialize()`: Zindan yöneticisinin durum değişkenlerini (aktif oda, koruyucu/portal VID'leri, olay işaretçileri) sıfırlar.
                *   `Destroy()`: Aktif olayları iptal eder ve durum değişkenlerini sıfırlar.
                *   `Prepare()`: Zindan odalarının koordinatlarını (`m_rooms`) yükler/hesaplar ve ilk koruyucuyu doğurmak için `ClearPrepare()` çağırır.
                *   `Spawn()`: Yeni bir koruyucu doğurur. Önceki koruyucuyu siler (varsa), rastgele yeni bir oda seçer (`RandomRoom`) ve odaya yeni koruyucuyu (`GUARDIAN`) doğurur. Koruyucunun aktivite ve oda değiştirme olayını (`guardian_activity_event`) başlatır.
                *   `ClearPrepare()`: Aktif portal varsa siler, yeni bir koruyucu doğurur (`Spawn`) ve portalın kaybolma olayını iptal eder.
                *   `SetRoom(int iRoom)`, `GetRoom()`: Mevcut aktif odanın numarasını ayarlar/alır.
                *   `RandomRoom(int& iRoom, int& iX, int& iY, int& iZ)`: Koruyucunun doğacağı bir sonraki odayı mevcut odadan farklı olarak rastgele seçer ve koordinatlarını döndürür.
                *   `ChangeRoom()`: Koruyucu belirli bir süre (`NO_ACTIVITY` veya `ATTACKED`) boyunca koşullar sağlandığında (örn. hedefi yoksa) mevcut aktivite olayını iptal eder ve yeni bir odaya `Spawn` eder.
                *   `SetGuardianVID(DWORD dwVID)`, `GetGuardianVID()`: Mevcut koruyucunun VID'sini ayarlar/alır.
                *   `SetPortalVID(DWORD dwVID)`, `GetPortalVID()`: Aktif portalın VID'sini ayarlar/alır.
                *   `OnGuardianKilled(int iX, int iY, int iZ)`: Koruyucu öldürüldüğünde çağrılır. Belirtilen konuma bir portal (`PORTAL` VNUM) doğurur ve portalın belirli bir süre (`PORTAL_VANISH_TIME`) sonra kaybolup yeni koruyucunun doğmasını sağlayan `guardian_event`'i başlatır.
                *   `AttackedGuardian(LPCHARACTER ch)`: Koruyucu bir karakterden (`ch`) saldırı aldığında çağrılır. Aktivite olayının zamanlayıcısını (`next_change_time`) saldırı sonrası bekleme süresi (`ATTACKED`) olarak günceller.
            *   **Private Üyeler:**
                *   `iTempleRoom` (int): Mevcut aktif odanın numarası.
                *   `dwGuardianVID`, `dwPortalVID` (DWORD): Koruyucu ve portalın güncel VID'leri.
                *   `bOnGuardianKilled` (bool): Koruyucunun öldürülüp portalın aktif olup olmadığını belirten bayrak (Bu üye `.cpp` dosyasında kullanılmıyor gibi görünüyor, ancak `.h` dosyasında tanımlı).
            *   **Protected Üyeler:**
                *   `s_pkGuardianKilledEvent` (LPEVENT): Portalın kaybolması ve yeni koruyucunun doğmasıyla ilgili olay.
                *   `s_pkGuardianActivityEvent` (LPEVENT): Koruyucunun aktivitesini ve yer değiştirmesini yöneten olay.
*   **Bağlantılı Dosyalar:** `TempleOchao.cpp` (uygulama), `stdafx.h`. (`event.h` ve `char.h` gibi başlıklar `stdafx.h` veya `TempleOchao.cpp` üzerinden dahil edilir).

---

### `TempleOchao.cpp`

*   **Amaç:** (`__MT_THUNDER_DUNGEON__` tanımlıysa) `TempleOchao.h` dosyasında bildirilen `CMgr` sınıfının metotlarını uygular. Ochao Tapınağı (veya Yıldırım Zindanı) mekaniklerini yönetir: Koruyucunun (`GUARDIAN`) zindan haritasındaki (`MAP_INDEX`) rastgele odalarda (`ROOMS` sayısı kadar) doğması, belirli bir süre saldırı almazsa (`NO_ACTIVITY`) veya saldırı aldıktan sonra belirli bir süre geçerse (`ATTACKED`) ve hedefi yoksa yer değiştirmesi, öldürüldüğünde bir portal (`PORTAL`) açılması ve bu portalın belirli bir süre (`PORTAL_VANISH_TIME`) sonra kaybolup yeni koruyucunun doğması.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`SRooms` Struct:** Oda koordinatlarını (X, Y, Z) tutar.
    *   **`RoomMapInfo` Typedef'i (`std::map<BYTE, SRooms>`):** Oda ID'lerini `SRooms` yapılarına eşler.
    *   **`m_rooms` (static RoomMapInfo):** Zindandaki tüm odaların ID'lerini ve koordinatlarını tutan statik harita.
    *   **`CMgr::Initialize()`, `CMgr::Destroy()`:** Zindan durum değişkenlerini (aktif oda numarası, koruyucu/portal VID'leri) ve olay işaretçilerini sıfırlar. `Destroy` ayrıca aktif olayları `event_cancel` ile iptal eder.
    *   **`CMgr::Prepare()`:** Sadece `m_rooms` haritası boşsa çalışır (ilk başlatmada). `SECTREE_MANAGER`'dan zindan haritasını alır. `pos[][]` dizisindeki sabit ofsetleri kullanarak her bir odanın dünya koordinatlarını hesaplar ve `m_rooms`'a ekler. Rastgele bir başlangıç odası seçer, VID'leri sıfırlar ve `ClearPrepare()` çağırarak ilk koruyucuyu doğurur. Test sunucusunda (`test_server`) oda koordinatlarını loglar.
    *   **`guardian_event_info` Struct:** Koruyucunun aktivite olayının (`guardian_activity_event`) bilgilerini tutar. `ch` (koruyucunun `LPCHARACTER` işaretçisi), `next_change_time` (bir sonraki oda değiştirme zamanı) ve `bAttacked` (saldırı alıp almadığı) üyelerini içerir. `DynamicCharacterPtr` (muhtemelen `LPCHARACTER` için bir tür akıllı işaretçi veya sarmalayıcı) kullanılır.
    *   **`guardian_activity_event` (EVENTFUNC):** Belirli aralıklarla (`CHECK_ACTIVITY`) çalışır. Eğer `pInfo->next_change_time` dolmuşsa, mevcut koruyucuyu (`pInfo->ch`) haritadan siler (`M2_DESTROY_CHARACTER`), bir sonraki kontrol zamanını ayarlar ve `CMgr::instance().ChangeRoom()` çağırarak koruyucunun yerini değiştirir.
    *   **`CMgr::Spawn()`:** Yeni bir koruyucu doğurur. `RandomRoom` ile rastgele bir oda ve koordinatları alır. `CHARACTER_MANAGER::SpawnMob` ile koruyucuyu (`GUARDIAN`) doğurur. Başarılı olursa, mevcut koruyucunun VID'sini günceller (`SetGuardianVID`). Eğer `s_pkGuardianActivityEvent` aktifse iptal eder ve yeni koruyucu için yeni bir `guardian_activity_event` oluşturur. Test sunucusunda loglama yapar.
    *   **`CMgr::RandomRoom(...)`:** Mevcut odadan (`GetRoom()`) farklı, 1 ile `ROOMS` arasında rastgele yeni bir oda numarası (`iGenerated`) üretir. Bu yeni oda numarasını `SetRoom` ile ayarlar. Seçilen odanın koordinatlarını `m_rooms` haritasından alır ve çıktı parametrelerine yazar.
    *   **`ochao_event_info` Struct:** Portalın kaybolma olayının (`guardian_event`) bilgilerini tutar. `ch` (portalın `LPCHARACTER` işaretçisi) ve `bStep` (olayın adımını belirtir, genellikle portalın kaldırılacağı adım için 1) üyelerini içerir.
    *   **`guardian_event` (EVENTFUNC):** Portalın kaybolma süresi (`PORTAL_VANISH_TIME`) dolduğunda çalışır. Eğer `pInfo->bStep == 1` ise, portalı (`pInfo->ch`) haritadan siler, `CMgr::instance().ClearPrepare()` çağırarak yeni koruyucuyu doğurur ve olayın durumunu sıfırlar.
    *   **`CMgr::OnGuardianKilled(iX, iY, iZ)`:** Koruyucu öldürüldüğünde çağrılır. Verilen koordinatlara portal NPC'sini (`PORTAL`) doğurur. Başarılı olursa portalın VID'sini kaydeder (`SetPortalVID`). Eğer `s_pkGuardianKilledEvent` aktifse iptal eder ve portalın belirli bir süre sonra kaybolması için yeni bir `guardian_event` oluşturur.
    *   **`CMgr::ClearPrepare()`:** Varsa mevcut portalı (`GetPortalVID()`) haritadan siler. Portal VID'sini sıfırlar. Aktif `s_pkGuardianKilledEvent` varsa iptal eder. `Spawn()` çağırarak yeni bir koruyucu doğurur.
    *   **`CMgr::ChangeRoom()`:** Koruyucunun aktivite olayını (`s_pkGuardianActivityEvent`) iptal eder. Eğer koruyucunun bir hedefi (saldırdığı oyuncu) yoksa ve haritadaysa, `Spawn()` ile yeni bir odaya doğurur.
    *   **`CMgr::AttackedGuardian(LPCHARACTER pkVictim)`:** Koruyucu `pkVictim` tarafından saldırı aldığında çağrılır. `s_pkGuardianActivityEvent`'in `pInfo->bAttacked` bayrağını `true` yapar ve `pInfo->next_change_time`'ı mevcut zamana `ATTACKED` süresi ekleyerek günceller. Bu, koruyucunun saldırı altındayken hemen yer değiştirmesini engeller. `pInfo->ch`'ye saldıran karakteri atar (muhtemelen bir sonraki `guardian_activity_event`'te kullanılmak üzere).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `TempleOchao.h`, `utils.h`, `char.h`, `char_manager.h`, `sectree_manager.h`, `config.h`.

---

### `OchaoTemple.cpp` (Alternatif Versiyon)

*   **Not:** Bu dosya, `TempleOchao.cpp` (`__MT_THUNDER_DUNGEON__` tanımlıysa) dosyasından farklı bir implementasyon içerir. Muhtemelen Ochao Tapınağı mekaniğinin farklı bir sürümüne veya yapılandırmasına aittir.
*   **Amaç:** `TempleOchao.h` dosyasında bildirilen `CMgr` sınıfının metotlarını uygular (bu versiyon için). Ochao Tapınağı'ndaki En-Tai Koruyucusu'nun mekaniklerini yönetir: Koruyucunun rastgele odalarda doğması, belirli bir süre saldırı almazsa yer değiştirmesi, öldürüldüğünde bir portal açılması ve portalın belirli bir süre sonra kaybolup yeni koruyucunun doğması.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`SRooms` Struct:** Oda koordinatlarını (X, Y, Z) tutar.
    *   **`m_rooms` (static std::map<BYTE, SRooms>):** Oda ID'lerini koordinatlara eşleyen statik bir harita.
    *   **`CMgr::Initialize()`, `CMgr::Destroy()`:** Zindan durum değişkenlerini (oda no, VID'ler, olaylar, `bOnGuardianKilled`) sıfırlar veya temizler. `Destroy` içindeyken aktif olayları iptal eder (`event_cancel`).
    *   **`CMgr::Prepare()`:** Sadece ilk seferde çalışır (`m_rooms` boşsa). Harita yöneticisinden (`SECTREE_MANAGER`) Ochao Tapınağı haritasını (`TEMPLE_OCHAO_MAP_INDEX`) alır. `pos[][]` dizisindeki sabit koordinatlara göre `TEMPLE_OCHAO_ROOMS` sayısı kadar odanın koordinatlarını hesaplar ve `m_rooms` haritasına ekler. Oda bilgilerini loglar. Rastgele bir başlangıç odası seçer ve `ClearPrepare()` çağırarak ilk koruyucuyu doğurur.
    *   **`guardian_event_info` Struct:** Koruyucunun aktivite olayının (`guardian_activity_event`) bilgilerini tutar. `next_change_time` (bir sonraki oda değiştirme zamanı) ve `bAttacked` (saldırı alıp almadığı) üyelerini içerir.
    *   **`guardian_activity_event` (EVENTFUNC):** Belirli aralıklarla (`TEMPLE_OCHAO_CHECK_ACTIVITY`) çalışır. Eğer `pInfo->next_change_time` dolmuşsa, bir sonraki kontrol zamanını (`TEMPLE_OCHAO_NO_ACTIVITY` ekleyerek) ayarlar ve `CMgr::instance().ChangeRoom()` çağırarak koruyucunun yerini değiştirir. (Bu versiyonda olay içinden koruyucu silinmiyor, `ChangeRoom` içinde yapılıyor olabilir).
    *   **`CMgr::Spawn()`:** Yeni koruyucuyu doğurur. `RandomRoom` ile yeni bir oda ve koordinat belirler. `CHARACTER_MANAGER::SpawnMob` ile koruyucuyu (`TEMPLE_OCHAO_GUARDIAN`) doğurur. Başarılı olursa, varsa eski koruyucuyu (`GetGuardianVID`) siler. Yeni koruyucunun VID'sini kaydeder (`SetGuardianVID`). `guardian_activity_event`'i başlatır.
    *   **`CMgr::RandomRoom(...)`:** Mevcut odadan (`GetRoom`) farklı, 1 ile `TEMPLE_OCHAO_ROOMS` arasında rastgele yeni bir oda numarası üretir. Seçilen odanın koordinatlarını `m_rooms` haritasından alır ve çıktı parametrelerine yazar. Yeni oda numarasını loglar ve `SetRoom` ile ayarlar.
    *   **`ochao_event_info` Struct:** Portalın kaybolma olayının (`guardian_event`) bilgilerini tutar. Sadece `bStep` üyesini içerir (genellikle portalın kaldırılacağı adım için 1).
    *   **`guardian_event` (EVENTFUNC):** Portalın kaybolma süresi (`TempleOchao::PORTAL_VANISH_TIME`) dolduğunda çalışır. Eğer `pInfo->bStep == 1` ise, `CMgr::instance().ClearPrepare()` çağırarak portalı kaldırır ve yeni koruyucuyu doğurur, ardından adımını sıfırlar.
    *   **`CMgr::OnGuardianKilled(iX, iY, iZ)`:** Koruyucu öldürüldüğünde çağrılır. Verilen koordinatlara portal NPC'sini (`TempleOchao::PORTAL`) doğurur. Başarılı olursa portalın VID'sini kaydeder (`SetPortalVID`). Portalın belirli bir süre sonra kaybolması için `guardian_event`'i başlatır (`bStep = 1` ile). `bOnGuardianKilled` bayrağını `true` yapar.
    *   **`CMgr::ClearPrepare()`:** `guardian_event`'i (portal kaybolma olayı) iptal eder. `Spawn` ile yeni koruyucuyu doğurur. Varsa eski portalı (`GetPortalVID()`) siler. Portal VID'sini sıfırlar.
    *   **`CMgr::ChangeRoom()`:** Koruyucunun aktivite olayını (`s_pkGuardianActivityEvent`) iptal eder. Eğer koruyucunun bir hedefi (saldırdığı oyuncu) yoksa ve haritadaysa, `Spawn()` ile yeni bir odaya doğurur.
    *   **`CMgr::AttackedGuardian()`:** Koruyucu saldırı aldığında çağrılır. Aktivite olayının `pInfo->bAttacked` bayrağını `true` yapar ve `pInfo->next_change_time`'ı mevcut zamana `TEMPLE_OCHAO_ATTACKED` süresi ekleyerek günceller.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `char.h`, `char_manager.h`, `sectree_manager.h`, `config.h`, `TempleOchao.h`.