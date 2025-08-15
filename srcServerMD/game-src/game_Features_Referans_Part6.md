# Metin2 Oyun Sunucusu - Özellikler Referansı Part 6 (`game/src`)

**Not:** Bu belge, [`game_Features_Referans_Part5.md`](game_Features_Referans_Part5.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) çeşitli özellikleri ve sistemleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`shop_manager.h`](#shop_managerh)
*   [`shop_manager.cpp`](#shop_managercpp)
*   [`shop.h`](#shoph)
*   [`shop.cpp`](#shopcpp)
*   [`shopEx.h`](#shopexh)
*   [`shopEx.cpp`](#shopexcpp)
*   [`SpeedServer.h`](#speedserverh)
*   [`SpeedServer.cpp`](#speedservercpp)
*   [`target.h`](#targeth)
*   [`target.cpp`](#targetcpp)
*   [`threeway_war.h`](#threeway_warh)
*   [`threeway_war.cpp`](#threeway_warcpp)
*   [`war_map.h`](#war_maph)
*   [`war_map.cpp`](#war_mapcpp)

---

### `shop_manager.h`

**Amacı:** Bu başlık dosyası, oyundaki tüm dükkan (shop) işlemlerini yöneten `CShopManager` singleton sınıfını tanımlar. NPC dükkanlarının ve oyuncu tarafından oluşturulan özel dükkanların (tezgahların) yönetimi, eşya alım satım işlemleri gibi temel işlevleri içerir.

**Temel Bileşenler:**

*   **`CShopManager` Sınıfı (Singleton):**
    *   **Amacı:** Oyun içindeki tüm dükkanları merkezi olarak yönetir.
    *   **Typedef'ler:**
        *   `TShopMap`: `DWORD` (genellikle dükkan VNUM'u veya oyuncu VID'si) anahtarını `CShop*` (dükkan nesnesi işaretçisi) değerine eşleyen bir harita.
    *   **Metotlar (Önemlileri):
        *   `Initialize(TShopTable* table, int size)`: Veritabanından yüklenen NPC dükkan bilgilerini (`TShopTable`) alarak dükkanları başlatır.
        *   `Destroy()`: Tüm dükkan nesnelerini silerek belleği temizler.
        *   `Get(DWORD dwVnum)`: Verilen VNUM'a sahip NPC dükkanını döndürür.
        *   `GetByNPCVnum(DWORD dwNPCVnum)`: Belirli bir NPC VNUM'una bağlı dükkanı döndürür.
        *   `StartShopping(LPCHARACTER pkChr, LPCHARACTER pkShopKeeper, int iShopVnum = 0)`: Bir oyuncunun bir NPC dükkanıyla etkileşime başlamasını sağlar.
        *   `StopShopping(LPCHARACTER ch)`: Oyuncunun bir dükkanla etkileşimini sonlandırır.
        *   `Buy(LPCHARACTER ch, BYTE pos)`: Oyuncunun dükkandan bir eşya satın almasını işler.
        *   `Sell(LPCHARACTER ch, WORD wCell, WORD wCount = 0, BYTE bType = 0)`: Oyuncunun bir eşyayı dükkana satmasını işler.
        *   `CreatePCShop(LPCHARACTER ch, TShopItemTable* pTable, WORD wItemCount)`: Bir oyuncu için özel bir dükkan (tezgah) oluşturur.
        *   `FindPCShop(DWORD dwVID)`: Verilen oyuncu VID'sine sahip özel dükkanı bulur.
        *   `DestroyPCShop(LPCHARACTER ch)`: Bir oyuncunun özel dükkanını kaldırır.
        *   `ReadShopTableEx(const char* stFileName)`: Gelişmiş dükkan özelliklerini (`shop_table_ex.txt`) okur ve uygular (genellikle `#if defined(__SHOPEX_RENEWAL__)` ile derleme zamanı koşuluna bağlıdır).
    *   **Özel Üyeler:**
        *   `m_map_pkShop`: VNUM'a göre NPC dükkanlarını tutan harita.
        *   `m_map_pkShopByNPCVnum`: NPC VNUM'una göre NPC dükkanlarını tutan harita.
        *   `m_map_pkShopByPC`: Oyuncu VID'sine göre özel dükkanları (tezgahları) tutan harita.
    *   **`__GEM_SYSTEM__` ile ilgili metotlar (Eğer tanımlıysa):
        *   `InitializeGemShop(TGemShopTable* table, int size)`: Cevher dükkanı verilerini başlatır.
        *   `GemShopGetRandomId(DWORD dwRow)`: Belirli bir satırdan rastgele bir cevher eşya ID'si alır.
        *   `GemShopGetVnumById(int id)`, `GemShopGetCountById(int id)`, `GemShopGetPriceById(int id)`: ID'ye göre cevher eşyasının VNUM, adet ve fiyatını döndürür.
    *   **Özel Üyeler (`__GEM_SYSTEM__` için):
        *   `m_iGemShopTableSize`: Cevher dükkanı tablosunun boyutu.
        *   `m_pGemShopTable`: Cevher dükkanı eşyalarını tutan dizi.

**Bağlantılı Dosyalar:** `stdafx.h` (temel başlıklar), `shop.h` (`CShop` sınıfı tanımı ve ilgili yapılar).

---

### `shop_manager.cpp`

**Amacı:** Bu dosya, `shop_manager.h` içinde tanımlanan `CShopManager` sınıfının metotlarını uygular. NPC dükkanlarının ve oyuncu tezgahlarının oluşturulması, eşya alım satım mantığı, dükkan verilerinin yüklenmesi ve yönetilmesi gibi işlevleri yerine getirir.

**Orta Seviye Implementasyon Detayları:**

*   **Yapıcı (`CShopManager()`) ve Yıkıcı (`~CShopManager()`):**
    *   Standart kurucu ve yıkıcı. Yıkıcı, `Destroy()` metodunu çağırarak tüm dükkan nesnelerini temizler.
*   **`Initialize(TShopTable* table, int size)`:**
    *   Veritabanından gelen `TShopTable` dizisini kullanarak NPC dükkanlarını oluşturur.
    *   Her bir dükkan için yeni bir `CShop` nesnesi yaratır ve `Create()` metodunu çağırır.
    *   Oluşturulan dükkanları `m_map_pkShop` (dükkan VNUM'u ile) ve `m_map_pkShopByNPCVnum` (NPC VNUM'u ile) haritalarına ekler.
    *   Ardından `ReadShopTableEx` fonksiyonunu çağırarak genişletilmiş dükkan ayarlarını (`shop_table_ex.txt`) yükler.
*   **`#if defined(__GEM_SYSTEM__)` Blokları:**
    *   `InitializeGemShop(TGemShopTable* table, int size)`: Cevher sistemi aktifse, cevher dükkanı için eşya bilgilerini yükler ve `m_pGemShopTable` dizisini doldurur.
    *   `GemShopGetRandomId(DWORD dwRow)`: Belirli bir satırdaki cevher eşyaları arasından rastgele birini seçer.
    *   `GemShopGetVnumById(int id)`, `GemShopGetCountById(int id)`, `GemShopGetPriceById(int id)`: Verilen ID'ye sahip cevher eşyasının bilgilerini döndürür.
*   **`Destroy()`:**
    *   `m_map_pkShop` içindeki tüm `CShop` nesnelerini siler. Yinelenen silmeleri önlemek için bir `boost::unordered_set` kullanır (çünkü farklı anahtarlar aynı `CShop` nesnesine işaret edebilir, özellikle `ReadShopTableEx` sonrası).
    *   Haritaları temizler.
*   **`Get(DWORD dwVnum)` ve `GetByNPCVnum(DWORD dwNPCVnum)`:**
    *   İlgili haritalardan (sırasıyla `m_map_pkShop` ve `m_map_pkShopByNPCVnum`) dükkan nesnesini bulur ve döndürür.
*   **`StartShopping(LPCHARACTER pkChr, LPCHARACTER pkChrShopKeeper, int iShopVnum)`:**
    *   Oyuncunun bir NPC dükkanıyla alışverişe başlamasını sağlar.
    *   Mesafe kontrolü (`SHOP_MAX_DISTANCE`) yapar.
    *   Ticaret penceresi engellerini kontrol eder (`PreventTradeWindow`).
    *   Verilen `iShopVnum` veya `pkChrShopKeeper`'ın `GetRaceNum()`'ı ile dükkanı bulur.
    *   Oyuncuyu dükkanın misafir listesine ekler (`pkShop->AddGuest()`) ve oyuncunun `SetShopOwner()` metodunu ayarlar.
*   **Oyuncu Dükkanları (PCShop) Yönetimi:**
    *   `FindPCShop(DWORD dwVID)`: `m_map_pkShopByPC` haritasından oyuncuya ait dükkanı bulur.
    *   `CreatePCShop(LPCHARACTER ch, TShopItemTable* pTable, WORD wItemCount)`: Oyuncu için yeni bir `CShop` nesnesi oluşturur, `SetPCShop()` ile oyuncuyu sahibi olarak ayarlar, eşyaları `SetShopItems()` ile yükler ve `m_map_pkShopByPC` haritasına ekler.
    *   `DestroyPCShop(LPCHARACTER ch)`: Oyuncunun dükkanını `m_map_pkShopByPC` haritasından siler ve `CShop` nesnesini yok eder. Eşya kopyalama sorunlarını önlemek için `ch->SetMyShopTime()` çağrılır.
*   **`StopShopping(LPCHARACTER ch)`:**
    *   Oyuncunun aktif dükkanını alır ve `shop->RemoveGuest(ch)` ile misafir listesinden çıkarır. `ch->SetMyShopTime()` çağrılır.
*   **`Buy(LPCHARACTER ch, BYTE pos)`:**
    *   Oyuncunun dükkandan eşya satın alma isteğini işler.
    *   Mesafe kontrolü yapar.
    *   Aktif dükkanın (`ch->GetShop()`) `Buy()` metodunu çağırır.
    *   Satın alma sonucu başarılı değilse (`SHOP_SUBHEADER_GC_OK` değilse) oyuncuya bir hata paketi gönderir.
*   **`Sell(LPCHARACTER ch, WORD wCell, WORD wCount, BYTE bType)`:**
    *   Oyuncunun envanterindeki bir eşyayı NPC dükkanına satma isteğini işler.
    *   Mesafe ve çeşitli kontroller yapar (eşya kilitli mi, giyili mi, satılabilir mi (`ITEM_ANTIFLAG_SELL`)).
    *   Eşyanın satış fiyatını hesaplar (genellikle eşya fiyatının 1/5'i eksi vergi).
    *   Vergi hesaplar ve kraliyet hazinesine eklenmesi için `CMonarch::instance().SendtoDBAddMoney()` çağrılır.
    *   Oyuncunun altını günceller (`ch->PointChange(POINT_GOLD, ...)`).
    *   Eşyayı envanterden kaldırır veya sayısını azaltır (`ITEM_MANAGER::instance().RemoveItem()` veya `item->SetCount()`).
    *   Maksimum altın sınırını kontrol eder (`g_MaxGold`).
*   **`ReadShopTableEx(const char* stFileName)`:**
    *   `shop_table_ex.txt` dosyasını okuyarak gelişmiş/çok sekmeli dükkanları yapılandırır.
    *   `CGroupTextParseTreeLoader` kullanarak dosyayı ayrıştırır.
    *   "shopnpc" grubundan NPC VNUM'larını ve bunlara karşılık gelen dükkan gruplarının adlarını okur.
    *   Her dükkan grubu için `ConvertToShopItemTable` fonksiyonunu çağırarak eşyaları, fiyatları, para birimini (Altın, İkincil Para Birimi, Eşya, EXP vb.) ve sıralama türünü okur.
    *   Eğer bir NPC için hem normal dükkan hem de genişletilmiş dükkan tanımlanmışsa hata verir.
    *   Mevcut `CShop` nesnesini `CShopEx` (çok sekmeli dükkan) nesnesine dönüştürür veya yeni bir `CShopEx` oluşturur.
    *   Okunan sekmeleri (`TShopTableEx`) `CShopEx::AddShopTable()` ile dükkana ekler.
*   **`ConvertToShopItemTable(IN CGroupNode* pNode, OUT TShopTableEx& shopTable)`:**
    *   `ReadShopTableEx` tarafından çağrılır. Bir dükkan sekmesinin (`CGroupNode`) verilerini ayrıştırarak `TShopTableEx` yapısını doldurur.
    *   Dükkan VNUM'unu, adını, para birimini (`coinType`) ve eşyalarını ("items" alt grubu) okur.
    *   Eşyaların sıralama düzenini (`sort` - Artan/Azalan/Yok) okur ve uygular.
    *   Eşyaları dükkan gridine (`CGrid`) yerleştirir.

**Bağlantılı Dosyalar:** `stdafx.h`, `shop_manager.h`, `shop.h`, `shopEx.h`, `constants.h`, `char.h`, `item.h`, `item_manager.h`, `config.h`, `locale_service.h`, `db.h`, `packet.h`, `log.h`, `questmanager.h`, `monarch.h`, `group_text_parse_tree.h` ve Boost kütüphaneleri.

---

### `shop.h`

**Amacı:** Bu başlık dosyası, tek bir dükkan örneğini temsil eden `CShop` sınıfını tanımlar. Hem NPC dükkanları hem de oyuncu tarafından açılan özel dükkanlar (tezgahlar) için temel yapıyı ve işlevselliği sağlar.

**Temel Bileşenler:**

*   **`SHOP_MAX_DISTANCE` Sabiti:** Oyuncunun bir dükkanla etkileşimde bulunabileceği maksimum mesafeyi tanımlar.
*   **`CShop` Sınıfı:**
    *   **`shop_item` Struct'ı:** Dükkandaki her bir eşyayı temsil eder. İçeriği:
        *   `vnum`: Eşyanın VNUM'u.
        *   `price`: Eşyanın altın cinsinden fiyatı.
        *   `price_cheque`: Eşyanın çek (Cheque) cinsinden fiyatı (eğer `__CHEQUE_SYSTEM__` tanımlıysa).
        *   `count`: Eşyanın adedi.
        *   `itemid`: Eğer bir oyuncu dükkanındaki gerçek bir eşyaysa, bu eşyanın benzersiz ID'si.
        *   `pkItem`: Eğer bir oyuncu dükkanındaki gerçek bir eşyaysa, bu eşyanın `LPITEM` işaretçisi.
    *   **Metotlar (Önemlileri):
        *   `Create(DWORD dwVnum, DWORD dwNPCVnum, TShopItemTable* pItemTable)`: Bir NPC dükkanı oluşturur ve verilen eşya tablosuyla (`pItemTable`) doldurur.
        *   `SetShopItems(TShopItemTable* pTable, WORD wItemCount)`: Dükkanın eşyalarını verilen tabloya göre ayarlar. Hem NPC hem de PC dükkanları için kullanılır.
        *   `SetPCShop(LPCHARACTER ch)`: Dükkanı bir oyuncu dükkanı (tezgah) olarak ayarlar ve sahibini (`ch`) belirler.
        *   `IsPCShop()`: Dükkanın bir oyuncu dükkanı olup olmadığını kontrol eder.
        *   `AddGuest(LPCHARACTER ch, DWORD owner_vid, bool bOtherEmpire)`: Bir oyuncuyu dükkanın misafir listesine ekler ve dükkan bilgilerini oyuncuya gönderir.
        *   `RemoveGuest(LPCHARACTER ch)`: Bir oyuncuyu misafir listesinden çıkarır.
        *   `Buy(LPCHARACTER ch, BYTE pos)`: Oyuncunun dükkandan belirtilen pozisyondaki eşyayı satın alma işlemini gerçekleştirir.
        *   `BroadcastUpdateItem(BYTE pos)`: Belirli bir pozisyondaki eşyanın durumu değiştiğinde (örn: satıldığında) tüm misafirlere güncelleme paketi gönderir.
        *   `GetNumberByVnum(DWORD dwVnum)`: Dükkandaki belirli bir VNUM'a sahip toplam eşya sayısını döndürür.
        *   `IsSellingItem(DWORD itemID)`: Dükkanda belirli bir `itemID`'ye sahip bir eşyanın satılıp satılmadığını kontrol eder (genellikle oyuncu dükkanları için).
        *   `GetVnum()`: Dükkanın VNUM'unu döndürür.
        *   `GetNPCVnum()`: Dükkanın sahibi olan NPC'nin VNUM'unu döndürür.
        *   `IsSoldOut()`: Oyuncu dükkanındaki tüm eşyaların satılıp satılmadığını kontrol eder.
    *   **Korunan Metotlar:**
        *   `Broadcast(const void* data, int bytes)`: Verilen veriyi dükkandaki tüm misafirlere gönderir.
    *   **Korunan Üyeler:**
        *   `m_dwVnum`: Dükkanın VNUM'u.
        *   `m_dwNPCVnum`: Sahip NPC'nin VNUM'u.
        *   `m_pGrid`: Dükkan arayüzündeki eşyaların yerleşimini yönetmek için kullanılan `CGrid` nesnesi.
        *   `m_map_guest` (`GuestMapType`): Dükkanı görüntüleyen oyuncuları (misafirleri) ve onların farklı imparatorluktan olup olmadığını tutan bir harita (`TR1_NS::unordered_map<LPCHARACTER, bool>`).
        *   `m_itemVector` (`std::vector<SHOP_ITEM>`): Dükkandaki eşyaları tutan bir vektör.
        *   `m_pkPC`: Eğer bir oyuncu dükkanıysa, sahip oyuncunun `LPCHARACTER` işaretçisi.

**Bağlantılı Dosyalar:** `stdafx.h` (temel başlıklar), `grid.h` (`CGrid` sınıfı tanımı).

---

### `shop.cpp`

**Amacı:** Bu dosya, `shop.h` içinde tanımlanan `CShop` sınıfının metotlarını uygular. Tek bir dükkanın (NPC veya oyuncu tezgahı) işlevlerini, eşya listesini yönetme, oyuncu etkileşimlerini (misafir ekleme/çıkarma, eşya satın alma) ve dükkan arayüzünün güncellenmesini sağlar.

**Orta Seviye Implementasyon Detayları:**

*   **Yapıcı (`CShop::CShop()`)**: `m_dwVnum`, `m_dwNPCVnum` ve `m_pkPC` üyelerini sıfırlar. Dükkan eşyalarının yerleşimi için bir `CGrid` nesnesi (5x9 boyutunda) oluşturur.
*   **Yıkıcı (`CShop::~CShop()`)**: Dükkandaki tüm misafirlere dükkanın kapandığına dair bir paket (`SHOP_SUBHEADER_GC_END`) gönderir. Misafirlerin `SetShop(NULL)` ile dükkan bağlantısını keser. `CGrid` nesnesini siler.
*   **`SetPCShop(LPCHARACTER ch)`**: Dükkanı bir oyuncu dükkanı olarak ayarlar ve `m_pkPC` üyesine sahip karakteri atar.
*   **`IsSoldOut()`**: Oyuncu dükkanındaki tüm `SHOP_ITEM`'ların `pkItem` üyesinin `NULL` olup olmadığını kontrol ederek dükkanın boşalıp boşalmadığını belirler.
*   **`Create(DWORD dwVnum, DWORD dwNPCVnum, TShopItemTable* pTable)`**: Bir NPC dükkanını başlatır.
    *   `m_dwVnum` ve `m_dwNPCVnum` üyelerini ayarlar.
    *   `pTable` (dükkan eşya tablosu) üzerinden geçerek satılacak eşya sayısını belirler.
    *   `SetShopItems()` metodunu çağırarak eşyaları dükkana yükler.
*   **`SetShopItems(TShopItemTable* pTable, WORD wItemCount)`**: Dükkanın eşya listesini (`m_itemVector`) doldurur.
    *   `m_pGrid->Clear()` ile dükkan gridini temizler.
    *   `m_itemVector`'ı `SHOP_HOST_ITEM_MAX_NUM` boyutuna ayarlar.
    *   Her bir eşya için:
        *   Eğer oyuncu dükkanıysa (`m_pkPC` doluysa), eşyayı oyuncunun envanterinden (`m_pkPC->GetItem(pTable->pos)`) alır.
        *   Değilse ve `pTable->vnum` geçerliyse, `ITEM_MANAGER`'dan eşya prototipini alır.
        *   Dükkan gridinde (`m_pGrid`) boş bir yer bulur (`FindBlank`). PC dükkanları için `pTable->display_pos` kullanılır.
        *   `m_itemVector`'daki ilgili `SHOP_ITEM` yapısını doldurur (vnum, count, price, itemid, pkItem).
        *   NPC dükkanlarındaki eşyaların fiyatı, eşya prototipindeki `dwShopBuyPrice` veya `ITEM_FLAG_COUNT_PER_1GOLD` bayrağına göre hesaplanır.
*   **`Buy(LPCHARACTER ch, BYTE pos)`**: Oyuncunun dükkandan eşya satın alma işlemini yönetir.
    *   Pozisyonun geçerli olup olmadığını ve oyuncunun misafir listesinde olup olmadığını kontrol eder.
    *   Satın alınacak `SHOP_ITEM`'ı alır.
    *   Fiyatın geçerli olup olmadığını kontrol eder (negatif veya sıfır olmamalı).
    *   Oyuncu dükkanıysa, satılan eşyanın (`pkSelectedItem`) hala var olup olmadığını ve sahibinin doğru olup olmadığını kontrol eder (hack girişimi tespiti).
    *   Fiyatı hesaplar (farklı imparatorluktaki oyuncular için NPC dükkanlarında fiyat 3 katıdır).
    *   Oyuncunun yeterli altını/çeki olup olmadığını kontrol eder.
    *   Satın alınacak eşyayı oluşturur (`ITEM_MANAGER::instance().CreateItem()`) veya PC dükkanıysa mevcut `r_item.pkItem`'ı kullanır.
    *   Oyuncunun envanterinde boş yer olup olmadığını kontrol eder (Dragon Soul ve özel envanterler için ayrı kontroller).
    *   Oyuncudan parayı düşer (`ch->PointChange(POINT_GOLD, ...)`).
    *   Vergi hesaplar ve uygular (`quest::CQuestManager::instance().GetEventFlag("trade_tax")` veya `personal_shop` event flag'i).
    *   NPC dükkanıysa, vergiyi kraliyete ekler (`CMonarch::instance().SendtoDBAddMoney()`).
    *   Eşyayı oyuncuya verir (`item->AddToCharacter(...)`).
    *   Eğer oyuncu dükkanıysa:
        *   Satıcının hızlı slotunu günceller.
        *   Eşyayı satıcının envanterinden çıkarır.
        *   Satıcıya parayı verir.
        *   Satıcıya vergi bilgilendirmesi yapar.
        *   Vergiyi satıcının imparatorluğunun kraliyetine ekler.
        *   `BroadcastUpdateItem(pos)` ile dükkandaki eşya listesini günceller.
        *   Dükkan boşaldıysa (`IsSoldOut()`), satıcının dükkanını kapatır (`m_pkPC->CloseMyShop()`).
    *   İşlemleri loglar (`LogManager::instance().ItemLog()`, `GoldBarLog()`).
    *   Başarılıysa `SHOP_SUBHEADER_GC_OK` döndürür, aksi halde uygun bir hata kodu döndürür.
*   **`AddGuest(LPCHARACTER ch, DWORD owner_vid, bool bOtherEmpire)`**: Bir oyuncuyu dükkana misafir olarak ekler.
    *   Oyuncunun zaten başka bir etkileşimde (ticaret, başka dükkan) olup olmadığını kontrol eder.
    *   Oyuncuyu `m_map_guest`'e ekler ve `ch->SetShop(this)` ile oyuncunun aktif dükkanını ayarlar.
    *   `TPacketGCShop` ve `TPacketGCShopStart` paketlerini hazırlar.
    *   `TPacketGCShopStart` paketindeki eşya listesini `m_itemVector`'dan doldurur. Farklı imparatorluk misafirleri için fiyatlar 3 katı olarak ayarlanır.
    *   Eşyaların bonuslarını (soketler, efsunlar, dönüşüm, element vb.) pakete ekler.
    *   Paketleri oyuncuya gönderir.
*   **`RemoveGuest(LPCHARACTER ch)`**: Bir misafiri dükkandan çıkarır.
    *   Oyuncuyu `m_map_guest`'ten siler ve `ch->SetShop(NULL)` yapar.
    *   Oyuncuya `SHOP_SUBHEADER_GC_END` paketi gönderir.
*   **`Broadcast(const void* data, int bytes)`**: Verilen paketi `m_map_guest`'teki tüm misafirlere gönderir.
*   **`BroadcastUpdateItem(BYTE pos)`**: Bir eşya satıldığında (genellikle PC dükkanlarında) çağrılır.
    *   `SHOP_SUBHEADER_GC_UPDATE_ITEM` paketini hazırlar.
    *   Güncellenen pozisyondaki `SHOP_ITEM` bilgilerini (vnum, bonuslar, fiyat, adet) pakete ekler. Eğer eşya tamamen satıldıysa vnum 0 olarak ayarlanır.
    *   Paketi `Broadcast()` ile tüm misafirlere gönderir.
*   **`GetNumberByVnum(DWORD dwVnum)`**: Dükkandaki belirli bir VNUM'a sahip tüm eşyaların toplam sayısını döndürür.
*   **`IsSellingItem(DWORD itemID)`**: Verilen `itemID`'ye sahip bir eşyanın dükkanda satılıp satılmadığını kontrol eder (oyuncu dükkanları için kullanılır).

**Bağlantılı Dosyalar:** `stdafx.h`, `shop.h`, `constants.h`, `char.h`, `item.h`, `item_manager.h`, `config.h`, `locale_service.h`, `packet.h`, `log.h`, `db.h`, `questmanager.h`, `monarch.h`, `grid.h`.

---

### `shopEx.h`

**Amacı:** Bu başlık dosyası, `CShop` sınıfından türeyen ve çok sekmeli, farklı para birimlerini destekleyen gelişmiş dükkanları (`CShopEx`) tanımlar. `shop_table_ex.txt` dosyasından okunan dükkanlar genellikle bu sınıfı kullanır.

**Temel Bileşenler:**

*   **`SShopTableEx` Struct'ı (`SShopTable`'dan Türer):**
    *   Normal `SShopTable` yapısını genişletir.
    *   `name` (`std::string`): Dükkan sekmesinin adını tutar.
    *   `coinType` (`EShopCoinType`): Bu sekmede kullanılacak para birimini belirtir (örn: Altın, İkincil Para Birimi, Eşya, EXP).
*   **`ShopTableExVector` Typedef'i:** `TShopTableEx` yapılarından oluşan bir vektör (`std::vector<TShopTableEx>`).
*   **`CShopEx` Sınıfı (`CShop`'tan Türer):**
    *   **Amacı:** Birden fazla sekmesi olabilen ve her sekmede farklı para birimi kullanılabilen gelişmiş NPC dükkanlarını temsil eder.
    *   **Metotlar (Önemlileri):
        *   `Create(DWORD dwVnum, DWORD dwNPCVnum)`: Gelişmiş dükkan nesnesini belirtilen VNUM ve NPC VNUM'u ile başlatır.
        *   `AddShopTable(TShopTableEx& shopTable)`: Dükkana yeni bir sekme (`TShopTableEx`) ekler. Aynı VNUM'a sahip sekmelerin eklenmesini engeller.
        *   `AddGuest(LPCHARACTER ch, DWORD owner_vid, bool bOtherEmpire)`: Oyuncuyu gelişmiş dükkana misafir olarak ekler. `SHOP_SUBHEADER_GC_START_EX` paketini ve tüm sekmelerin bilgilerini gönderir.
        *   `SetPCShop(LPCHARACTER ch)`: Geçersiz kılınmıştır, `CShopEx` PC dükkanı olamaz.
        *   `IsPCShop()`: Her zaman `false` döndürür.
        *   `Buy(LPCHARACTER ch, BYTE pos)`: Oyuncunun belirli bir sekmedeki ve pozisyondaki eşyayı satın alma işlemini yönetir. Farklı para birimlerine göre kontroller yapar.
        *   `IsSellingItem(DWORD itemID)`: Geçersiz kılınmıştır, `CShopEx` için anlamlı değildir.
        *   `GetTabCount()`: Dükkandaki toplam sekme sayısını döndürür.
    *   **Özel Üyeler:**
        *   `m_vec_shopTabs` (`ShopTableExVector`): Dükkanın tüm sekmelerini (`TShopTableEx`) tutan vektör.

**Bağlantılı Dosyalar:** `stdafx.h`, `shop.h` (`CShop` ve temel dükkan yapıları), `typedef.h` (muhtemelen `EShopCoinType` enum tanımı).

---

### `shopEx.cpp`

**Amacı:** Bu dosya, `shopEx.h` içinde tanımlanan `CShopEx` sınıfının metotlarını uygular. Çok sekmeli ve farklı para birimleriyle çalışan gelişmiş NPC dükkanlarının işlevselliğini sağlar.

**Orta Seviye Implementasyon Detayları:**

*   **`Create(DWORD dwVnum, DWORD dwNPCVnum)`:**
    *   Dükkanın ana VNUM'unu (`m_dwVnum`) ve sahip NPC'nin VNUM'unu (`m_dwNPCVnum`) ayarlar.
*   **`AddShopTable(TShopTableEx& shopTable)`:**
    *   Verilen `shopTable` (yeni sekme) bilgilerini `m_vec_shopTabs` vektörüne ekler.
    *   Eklenmeden önce, aynı VNUM veya NPC VNUM'una sahip bir sekmenin zaten var olup olmadığını kontrol eder; varsa eklemez ve `false` döndürür.
*   **`AddGuest(LPCHARACTER ch, DWORD owner_vid, bool bOtherEmpire)`:**
    *   `CShop::AddGuest` metoduna benzer şekilde oyuncuyu misafir olarak ekler (`m_map_guest`).
    *   `SHOP_SUBHEADER_GC_START_EX` alt başlığı ile bir `TPacketGCShop` paketi hazırlar.
    *   `TPacketGCShopStartEx` yapısını doldurur (sahip VID'si, sekme sayısı).
    *   Her bir dükkan sekmesi (`TShopTableEx`) için `TPacketGCShopStartEx::TSubPacketShopTab` yapısını oluşturur:
        *   Sekme adını ve para birimini (`coin_type`) ayarlar.
        *   Sekmedeki her bir eşya için VNUM, adet ve fiyatı ayarlar.
        *   Fiyat, sekmenin para birimine ve oyuncunun farklı imparatorluktan olup olmamasına göre ayarlanır (Altın için 3 katı, diğer para birimleri için sabit).
        *   Eşya bonusları (efsun, soket vb.) bu aşamada pakete *eklenmez* (varsayılan olarak sıfırlanır), çünkü `CShopEx` genellikle NPC dükkanıdır ve satılan eşyalar prototiplerden oluşturulur, oyuncu envanterindeki gibi dinamik bonuslara sahip olmazlar.
    *   Hazırlanan paketleri (ana paket, `TPacketGCShopStartEx` ve tüm sekmelerin verileri) oyuncuya gönderir.
*   **`Buy(LPCHARACTER ch, BYTE pos)`:**
    *   Gelen `pos` (pozisyon) bilgisinden sekme indeksini (`tabIdx`) ve sekme içindeki eşya pozisyonunu (`slotPos`) hesaplar.
    *   Sekme indeksinin geçerli olup olmadığını kontrol eder.
    *   Oyuncunun misafir listesinde olup olmadığını kontrol eder.
    *   İlgili sekme (`shopTab`) ve eşyayı (`r_item`) alır.
    *   Eşya fiyatının geçerli olup olmadığını kontrol eder.
    *   Fiyatı (`dwPrice`) alır.
    *   Sekmenin para birimine (`shopTab.coinType`) göre oyuncunun yeterli paraya/eşyaya/deneyime sahip olup olmadığını kontrol eder:
        *   `SHOP_COIN_TYPE_GOLD`: Oyuncunun altınını kontrol eder (farklı imparatorluk için 3 katı fiyat).
        *   `SHOP_COIN_TYPE_SECONDARY_COIN`: Oyuncunun envanterindeki ikincil para birimi (`ITEM_SECONDARY_COIN`) sayısını kontrol eder.
        *   `SHOP_COINT_TYPE_ITEM` (`__SHOPEX_RENEWAL__` ile): Oyuncunun envanterinde, fiyat olarak belirtilen VNUM'daki eşyadan yeterli adette olup olmadığını kontrol eder.
        *   `SHOP_COINT_TYPE_EXP` (`__SHOPEX_RENEWAL__` ile): Oyuncunun deneyim puanını kontrol eder.
    *   Yeterli değilse uygun bir hata kodu (`SHOP_SUBHEADER_GC_NOT_ENOUGH_MONEY`, `_EX`, `_ITEM`, `_EXP`) döndürür.
    *   Satın alınacak eşyayı `ITEM_MANAGER::instance().CreateItem()` ile oluşturur.
    *   Oyuncunun envanterinde boş yer olup olmadığını kontrol eder.
    *   Para birimine göre oyuncudan ilgili değeri düşer (`ch->PointChange()`, `ch->RemoveSpecifyTypeItem()`, `ch->RemoveSpecifyItem()`).
    *   Eşyayı oyuncunun envanterine ekler.
    *   İşlemi loglar.
    *   Başarılıysa `SHOP_SUBHEADER_GC_OK` döndürür.

**Bağlantılı Dosyalar:** `stdafx.h`, `shopEx.h`, `shop.h`, `constants.h`, `char.h`, `item.h`, `item_manager.h`, `packet.h`, `log.h`, `db.h`, `desc.h`, `locale_service.h`.

---

### `SpeedServer.h`

**Amacı:** Bu başlık dosyası, "Hız Sunucusu" (Speed Server) özelliklerini, özellikle de imparatorluklara göre belirli zamanlarda (haftanın günleri, tatiller) aktifleşen deneyim (EXP) bonusu tablolarını yönetmek için `CSpeedServerManager` ve `CSpeedServerEmpireExp` sınıflarını tanımlar.

**Temel Bileşenler:**

*   **İmparatorluk Sabitleri:** `EMPIRE_NONE`, `EMPIRE_RED`, `EMPIRE_YELLOW`, `EMPIRE_BLUE`.
*   **`HME` Sınıfı:**
    *   Saat (`hour`), dakika (`min`) ve EXP bonus yüzdesini (`exp`) tutar.
    *   Karşılaştırma operatörleri (`==`, `<`) ve atama operatörü içerir.
*   **`Date` Sınıfı:**
    *   Yıl (`year`), ay (`mon`) ve günü (`day`) tutar.
    *   Karşılaştırma operatörleri (`==`, `<`) içerir.
*   **`CSpeedServerEmpireExp` Sınıfı:**
    *   **Amacı:** Tek bir imparatorluk için haftanın günlerine ve özel tatillere göre EXP bonus tablolarını yönetir.
    *   **Metotlar:**
        *   `Initialize(BYTE empire)`: Belirli bir imparatorluk için EXP bonus yöneticisini başlatır, dosya adını (`exp_bonus_table_[empire].txt`) ayarlar ve varsayılan haftalık EXP tablolarını doldurur, ardından `LoadExpTable()` çağırır.
        *   `GetWdayExpTable(int wday)`: Belirli bir haftanın günü (`wday`) için EXP bonus zamanlamalarını (`std::list<HME>`) döndürür.
        *   `SetWdayExpTable(int wday, HME hme)`: Belirli bir güne yeni bir EXP bonus zamanlaması ekler ve tabloyu dosyaya yazar (`WriteExpTable()`).
        *   `GetHolidayExpTable(Date date, bool& is_exist)`: Belirli bir tarih için tatil EXP bonus zamanlamalarını döndürür. `is_exist` ile tarihin kayıtlı olup olmadığını belirtir.
        *   `SetHolidayExpTable(Date date, HME hme)`: Belirli bir tatil tarihine yeni bir EXP bonus zamanlaması ekler ve tabloyu dosyaya yazar.
        *   `InitWdayExpTable(int wday)`: Belirli bir günün EXP tablosunu temizler.
        *   `InitHolidayExpTable(Date date)`: Belirli bir tatil gününün EXP tablosunu temizler (eğer yoksa oluşturur).
        *   `GetCurrentExpPriv(int& duration, bool& is_change)`: Mevcut zamana göre aktif olan EXP bonusunu (`HME`) ve bir sonraki değişikliğe kadar kalan süreyi (`duration`) hesaplar. Bonusun bir önceki kontrolden bu yana değişip değişmediğini (`is_change`) belirtir.
        *   `WriteExpTable()`: Mevcut haftalık ve tatil EXP tablolarını ilgili imparatorluğun dosyasına yazar.
    *   **Özel Metotlar:**
        *   `LoadExpTable()`: İlgili imparatorluğun EXP bonus tablosu dosyasını okur.
        *   `LoadWdayExpTable(int wday, char* str)`: Dosyadan okunan bir satırı ayrıştırarak belirli bir günün EXP tablosunu doldurur.
    *   **Üyeler:**
        *   `empire` (BYTE): Yönetilen imparatorluk ID'si.
        *   `file_name[256]` (char): EXP bonus tablosu dosyasının adı.
        *   `current_hme` (`HME`): En son kontrol edilen aktif EXP bonusu.
        *   `holiday_map` (`std::map<Date, std::list<HME>>`): Tatil tarihlerini ve o tarihlerdeki EXP bonus zamanlamalarını tutar.
        *   `wday_exp_table[7]` (`std::list<HME>` dizisi): Haftanın 7 günü için EXP bonus zamanlamalarını tutar.
*   **`CSpeedServerManager` Sınıfı (Singleton):**
    *   **Amacı:** Tüm imparatorluklar için `CSpeedServerEmpireExp` nesnelerini yönetir ve genel bir arayüz sağlar.
    *   **Metotlar:**
        *   `Initialize()`: Tüm imparatorluklar için (`EMPIRE_MAX_NUM`'a kadar) `CSpeedServerEmpireExp::Initialize()` çağırır.
        *   İmparatorluk bazında `GetWdayExpTableOfEmpire`, `SetWdayExpTableOfEmpire`, `InitWdayExpTableOfEmpire`, `GetHolidayExpTableOfEmpire`, `SetHolidayExpTableOfEmpire`, `InitHolidayExpTableOfEmpire`, `WriteExpTableOfEmpire`, `GetCurrentExpPrivOfEmpire` gibi `CSpeedServerEmpireExp` metotlarına yönlendirme yapan metotlar içerir.
    *   **Özel Üyeler:**
        *   `Empire[EMPIRE_MAX_NUM]` (`CSpeedServerEmpireExp` dizisi): Her imparatorluk için bir `CSpeedServerEmpireExp` nesnesi tutar.

**Bağlantılı Dosyalar:** `../../common/length.h` (muhtemelen `EMPIRE_MAX_NUM` için), `<list>`, `<map>` (STL koleksiyonları için).

---

### `SpeedServer.cpp`

**Amacı:** Bu dosya, `SpeedServer.h` içinde tanımlanan `CSpeedServerManager` ve `CSpeedServerEmpireExp` sınıflarının metotlarını uygular. İmparatorluklara özel, zamana bağlı EXP bonusu tablolarının yüklenmesi, kaydedilmesi ve sorgulanması işlevlerini yerine getirir.

**Orta Seviye Implementasyon Detayları:**

*   **`CSpeedServerEmpireExp` Sınıfı Metotları:**
    *   **`Initialize(BYTE e)`:**
        *   İmparatorluk ID'sini (`empire`) ve dosya adını (`exp_bonus_table_[empire].txt`) ayarlar.
        *   Haftanın günleri için varsayılan EXP bonus zamanlamalarını `wday_exp_table`'a ekler (örn: Hafta içi 18:00'de %50, 24:00'te %100; Hafta sonu 18:00'de %100, 24:00'te %150 gibi).
        *   `LoadExpTable()` çağırarak dosyadan özel ayarları yükler (varsa varsayılanların üzerine yazar).
    *   **`LoadWdayExpTable(int wday, char* str)`:**
        *   Belirli bir gün için (`wday`) EXP bonus tablosunu (`str` içindeki veriden) yükler.
        *   Satırı `;` karakterine göre ayırır, her bir bölümü saat:dakika ve exp yüzdesi olarak ayrıştırır (`strtok`, `str_to_number`) ve `wday_exp_table[wday]` listesine `HME` nesnesi olarak ekler.
    *   **`WriteExpTable()`:**
        *   İlgili imparatorluğun EXP bonus dosyasını (`file_name`) yazma modunda açar.
        *   Haftanın her günü için (`wday_exp_table`) kaydedilmiş `HME` (saat, dakika, exp) bilgilerini dosyaya yazar (örn: "MON 18:0 50; 24:0 100;").
        *   `holiday_map`'teki her tatil için tarih ve o tarihe ait `HME` bilgilerini dosyaya yazar (örn: "HOLIDAY 2023.12.25 18:0 100; 24:0 150;").
    *   **`LoadExpTable()`:**
        *   İlgili imparatorluğun EXP bonus dosyasını okuma modunda açar.
        *   Dosyayı satır satır okur.
        *   Her satırın ilk kelimesine (`token_string`) göre işlem yapar:
            *   Eğer "SUN", "MON", ..., "SAT" ise, `LoadWdayExpTable` çağırarak ilgili günün EXP tablosunu yükler.
            *   Eğer "HOLIDAY" ise, tarihi (Y.M.D formatında) ve o tarihe ait EXP bonus zamanlamalarını ayrıştırır, `holiday_map`'e ekler.
    *   **`GetWdayExpTable(int wday)`**: İlgili `wday_exp_table[wday]` listesini döndürür.
    *   **`SetWdayExpTable(int wday, HME hme)`**: Verilen `HME` bilgisini `wday_exp_table[wday]` listesine ekler ve `WriteExpTable()` ile dosyayı günceller.
    *   **`GetHolidayExpTable(Date date, bool& is_exist)`**: `holiday_map`'te verilen tarihi arar. Bulunursa `is_exist`'i `true` yapar ve ilgili `std::list<HME>`'yi döndürür. Bulunamazsa `is_exist`'i `false` yapar ve geçersiz bir referans döndürebilir (kullanımına dikkat edilmeli).
    *   **`SetHolidayExpTable(Date date, HME hme)`**: `holiday_map`'te verilen tarihi arar. Bulunursa, `hme` bilgisini o tarihin listesine ekler. Sonra `WriteExpTable()` ile dosyayı günceller.
    *   **`InitWdayExpTable(int wday)`**: `wday_exp_table[wday].clear()` ile ilgili günün listesini temizler.
    *   **`InitHolidayExpTable(Date date)`**: `holiday_map`'te verilen tarihi arar. Bulunursa listesini temizler, bulunmazsa o tarih için boş bir liste oluşturur.
    *   **`GetCurrentExpPriv(int& duration, bool& is_change)`:**
        *   Mevcut sistem zamanını (`localtime`, `time`) alır.
        *   Mevcut tarihi `Date` nesnesi olarak oluşturur.
        *   `holiday_map`'te mevcut tarih için bir giriş olup olmadığını kontrol eder.
        *   Eğer tatil günü ise tatil tablosunu, değilse haftanın ilgili gününün tablosunu (`wday_exp_table[datetime->tm_wday]`) kullanır.
        *   Kullanılacak listedeki `HME` girişlerini dolaşarak, mevcut zamandan (`total_sec`) sonraki ilk EXP bonus zamanlamasını (`hme`) bulur.
        *   `duration` değişkenine, bir sonraki EXP bonus değişikliğine kadar kalan saniye sayısını atar.
        *   Bulunan `hme`'nin bir önceki `current_hme` ile aynı olup olmadığını kontrol ederek `is_change` değişkenini ayarlar.
        *   `current_hme`'yi günceller ve bulunan `hme`'yi döndürür.
*   **`CSpeedServerManager` Sınıfı Metotları:**
    *   Çoğu metot, parametre olarak aldığı imparatorluk ID'sine göre ilgili `Empire[empire]` nesnesinin aynı isimli metodunu çağırır (örneğin, `GetWdayExpTableOfEmpire` -> `Empire[empire].GetWdayExpTable`).
    *   **`Initialize()`**: Tüm imparatorluklar için `Empire[i].Initialize(i)` çağırır.

**Bağlantılı Dosyalar:** `stdafx.h`, `SpeedServer.h`, `locale_service.h`, `<time.h>`. Dosya işlemleri için standart C kütüphaneleri (`stdio.h`, `string.h`) ve sayı dönüşümleri için `utils.h` (muhtemelen `str_to_number` içerir) kullanılır.

---

### `target.h`

*   **Amaç:** Oyuncular için görev (quest) hedeflerini yönetmek amacıyla `CTargetManager` singleton sınıfını ve ilgili yapıları tanımlar. Bu sistem, oyunculara belirli bir konumu veya varlığı (NPC/mob) hedef olarak gösterir, hedefin konumunu günceller ve hedefe ulaşıldığında veya hedef yok olduğunda görev sistemine bilgi verir.
*   **Temel İşlevler/İçerik:**
    *   **`ETargetTypes` Enum'u:**
        *   `TARGET_TYPE_POS`: Hedefin bir harita konumu (X, Y) olduğunu belirtir.
        *   `TARGET_TYPE_VID`: Hedefin bir varlık ID'si (Virtual ID - VID) olduğunu belirtir (örn: bir NPC veya canavar).
    *   **`TargetInfo` Struct'ı (EVENTINFO makrosu ile):**
        *   Tek bir hedef olayının tüm bilgilerini tutar.
        *   `iID` (int): Hedefin benzersiz kimliği (istemciye gönderilir).
        *   `dwPID` (DWORD): Hedefin atandığı oyuncunun PID'si.
        *   `dwQuestIndex` (DWORD): Bu hedefle ilişkili görevin indeksi.
        *   `szTargetName[33]` (char): Görev içindeki hedefin adı (Lua tarafında kullanılır).
        *   `szTargetDesc[33]` (char): Hedefin açıklaması (isteğe bağlı, istemcide gösterilebilir).
        *   `iType` (int): `ETargetTypes` enumundan hedefin türü.
        *   `iArg1`, `iArg2` (int): Hedef türüne göre argümanlar:
            *   `TARGET_TYPE_POS`: `iArg1` = X koordinatı, `iArg2` = Y koordinatı.
            *   `TARGET_TYPE_VID`: `iArg1` = Hedef varlığın VID'si.
        *   `iMapIndex` (int): Hedefin bulunduğu haritanın indeksi.
        *   `iOldX`, `iOldY` (int): Hedefin istemciye gönderilen son bilinen X ve Y koordinatları.
        *   `bSendToClient` (bool): Bu hedefin istemciye gönderilip gönderilmeyeceğini belirtir.
        *   **Yapıcı:** Tüm üyeleri varsayılan değerlere (0, false, boş string) başlatır.
    *   **`ListEventMap` Typedef'i:** `std::map<DWORD, std::list<LPEVENT>>` için bir takma ad. Oyuncu PID'sini, o oyuncuya ait aktif hedef olaylarının (`LPEVENT`) bir listesine eşler.
    *   **`CTargetManager` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:** `m_iID`'yi 0 olarak başlatır.
        *   **Metotlar:**
            *   `CreateTarget(DWORD dwPID, DWORD dwQuestIndex, const char* c_pszTargetName, int iType, int iArg1, int iArg2, int iMapIndex, const char* c_pszTargetDesc = NULL, int iSendFlag = 1)`: Belirtilen oyuncu için yeni bir hedef oluşturur veya mevcut bir hedefi günceller. Bir olay (`target_event`) oluşturur ve bunu `m_map_kListEvent`'e ekler. İstemciye `SendTargetCreatePacket` ile bilgi gönderir.
            *   `DeleteTarget(DWORD dwPID, DWORD dwQuestIndex, const char* c_pszTargetName)`: Belirtilen oyuncu için bir hedefi veya belirli bir göreve ait tüm hedefleri siler. İlgili olayı iptal eder ve istemciye `SendTargetDeletePacket` ile bilgi gönderir.
            *   `Logout(DWORD dwPID)`: Bir oyuncu oyundan çıktığında çağrılır. O oyuncuya ait tüm hedef olaylarını iptal eder ve `m_map_kListEvent`'ten kaldırır.
            *   `GetTargetInfo(DWORD dwPID, int iType, int iArg1)`: Belirli bir oyuncu, hedef türü ve argümanına uyan ilk `TargetInfo`'yu döndürür.
            *   `GetTargetEvent(DWORD dwPID, DWORD dwQuestIndex, const char* c_pszTargetName)`: Belirli bir oyuncu, görev indeksi ve hedef adına uyan `LPEVENT`'i döndürür.
        *   **Korumalı (Protected) Üyeler:**
            *   `m_map_kListEvent` (`ListEventMap`): Oyuncuların aktif hedef olaylarını saklar.
            *   `m_iID` (int): Yeni hedeflere benzersiz ID atamak için kullanılan bir sayaç.
*   **Bağlantılı Dosyalar:** `target.cpp` (uygulama), `event.h` (`EVENTINFO`, `LPEVENT`), `stdafx.h`.

---

### `target.cpp`

*   **Amaç:** `target.h` dosyasında bildirilen `CTargetManager` sınıfının metotlarını ve hedef olayının (`target_event`) mantığını uygular. Görev hedeflerinin oluşturulması, güncellenmesi, silinmesi ve oyuncunun hedefe ulaşıp ulaşmadığının kontrol edilmesinden sorumludur.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Paket Gönderme Fonksiyonları:**
        *   `SendTargetCreatePacket(LPDESC d, TargetInfo* info)`: İstemciye yeni bir hedefin oluşturulduğunu bildiren `HEADER_GC_TARGET_CREATE` paketini gönderir.
        *   `SendTargetUpdatePacket(LPDESC d, int iID, int x, int y)`: Bir hedefin konumunun güncellendiğini bildiren `HEADER_GC_TARGET_UPDATE` paketini gönderir.
        *   `SendTargetDeletePacket(LPDESC d, int iID)`: Bir hedefin silindiğini bildiren `HEADER_GC_TARGET_DELETE` paketini gönderir.
    *   **`target_event` (EVENTFUNC):**
        *   Periyodik olarak (varsayılan olarak saniyede bir) çalışan ana hedef olay fonksiyonudur.
        *   `TargetInfo` içindeki `dwPID` ile karakteri (`pkChr`) bulur.
        *   Hedefin harita indeksi oyuncunun mevcut harita indeksiyle eşleşmiyorsa, olayı bir süre sonra tekrar çalışacak şekilde ayarlar.
        *   Hedef türüne (`TARGET_TYPE_POS` veya `TARGET_TYPE_VID`) göre hedefin güncel konumunu (`x`, `y`) belirler.
            *   `TARGET_TYPE_VID`: Hedef karakteri (`tch`) bulur, varsa konumunu alır.
        *   Oyuncunun hedefe olan mesafesini (`iDist`) hesaplar.
        *   Eğer mesafe 500 birimden küçük veya eşitse, `CQuestManager::instance().Target(pkChr->GetPlayerID(), info->dwQuestIndex, info->szTargetName, "arrive")` çağırarak göreve "ulaşıldı" bilgisi gönderir.
        *   Eğer hedef `TARGET_TYPE_VID` ise ve hedef karakter (`tch`) artık yoksa (ölmüş veya kaybolmuşsa), göreve "die" (öldü) bilgisi gönderir ve hedefi `CTargetManager::DeleteTarget` ile siler.
        *   Hedefin konumu değişmişse (`x != info->iOldX || y != info->iOldY`) ve istemciye gönderilmesi gerekiyorsa (`info->bSendToClient`), `SendTargetUpdatePacket` ile yeni konumu gönderir.
        *   Olayın bir sonraki çalışma süresini, hedefe olan mesafeye göre dinamik olarak ayarlar (yakınsa daha sık, uzaksa daha seyrek).
    *   **`CTargetManager::CreateTarget(...)`:**
        *   Hedefin atanacağı karakteri (`pkChr`) PID ile bulur.
        *   Karakterin haritası, hedefin haritasıyla eşleşmiyorsa işlem yapmaz.
        *   Aynı oyuncu, aynı görev ve aynı hedef adı için zaten bir hedef varsa, mevcut hedefi günceller (ID'yi, türü, argümanları, açıklamayı değiştirir) ve istemciye eski hedefi silip yenisini oluşturma paketleri gönderir.
        *   Yeni bir hedefse, yeni bir `TargetInfo` oluşturur, bilgilerini ayarlar ve `event_create` ile `target_event`'i oluşturarak `m_map_kListEvent`'e ekler. İstemciye `SendTargetCreatePacket` gönderir.
    *   **`CTargetManager::DeleteTarget(...)`:**
        *   Oyuncunun hedef listesinden (`m_map_kListEvent[dwPID]`) belirtilen görev indeksi ve hedef adına uyan hedefi/hedefleri bulur.
        *   Bulunan her hedef için, eğer istemciye gönderilmişse `SendTargetDeletePacket` gönderir.
        *   `event_cancel` ile olayı iptal eder ve listeden siler.
    *   **`CTargetManager::GetTargetEvent(...)`:**
        *   Verilen parametrelere uyan aktif hedef olayını (`LPEVENT`) `m_map_kListEvent`'ten arar ve bulursa döndürür.
    *   **`CTargetManager::GetTargetInfo(...)`:**
        *   Verilen oyuncu PID'si, hedef türü ve `iArg1` (genellikle VID) ile eşleşen ilk `TargetInfo` yapısını arar ve bulursa döndürür.
    *   **`CTargetManager::Logout(DWORD dwPID)`:**
        *   Oyuncu oyundan çıktığında, o oyuncuya ait tüm hedef olaylarını `event_cancel` ile iptal eder ve `m_map_kListEvent`'ten oyuncunun girişini siler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `questmanager.h`, `sectree_manager.h`, `char.h`, `char_manager.h`, `desc.h`, `packet.h`, `target.h`. 

---

### `threeway_war.h`

*   **Amaç:** "Üç Yol Savaşı" (Three-Way War) etkinliğini yönetmek için `CThreeWayWar` singleton sınıfını ve ilgili yapıları/fonksiyonları tanımlar. Bu etkinlik, genellikle üç krallığın (empire) birbiriyle savaştığı özel haritalarda (Sungzi ve Pass haritaları) gerçekleşir.
*   **Temel İşlevler/İçerik:**
    *   **`ForkedSungziMapInfo` Struct:**
        *   Sungzi (Kutsal Topraklar/Kale) haritasının bilgilerini tutar:
            *   `m_iForkedSung`: Harita indeksi.
            *   `m_iForkedSungziStartPosition[3][2]`: Üç krallık için başlangıç pozisyonları.
            *   `m_stMapName`: Haritanın dosya adı/yolu için kullanılan isim.
            *   `m_iBossMobVnum`: Bu haritada ortaya çıkacak boss canavarının VNUM'u.
    *   **`ForkedPassMapInfo` Struct:**
        *   Geçit (Pass) haritalarının bilgilerini tutar:
            *   `m_iForkedPass[3]`: Üç krallığın geçit haritalarının indeksleri.
            *   `m_iForkedPassStartPosition[3][2]`: Üç krallık için bu haritalardaki başlangıç pozisyonları.
            *   `m_stMapName[3]`: Üç krallığın geçit haritalarının dosya adları/yolları için kullanılan isimler.
    *   **`CThreeWayWar` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:** `Initialize()` çağırır, haritaları temizler.
        *   **Yönetim:**
            *   `Initialize()`: Savaş durumu verilerini sıfırlar (skorlar, kayıtlı kullanıcılar vb.).
            *   `LoadSetting(const char* szFileName)`: Belirtilen yapılandırma dosyasından (`threeway_war.txt` gibi) Sungzi ve Pass haritalarının ayarlarını yükler.
        *   **Skor Yönetimi:**
            *   `GetKillScore(BYTE empire) const`: Belirtilen krallığın öldürme skorunu döndürür.
            *   `SetKillScore(BYTE empire, int count)`: Belirtilen krallığın öldürme skorunu ayarlar.
        *   **Canlanma Jetonu (Revive Token) Yönetimi:**
            *   `SetReviveTokenForPlayer(DWORD PlayerID, int count)`: Bir oyuncu için canlanma jetonu sayısını ayarlar.
            *   `GetReviveTokenForPlayer(DWORD PlayerID)`: Oyuncunun canlanma jetonu sayısını döndürür.
            *   `DecreaseReviveTokenForPlayer(DWORD PlayerID)`: Oyuncunun canlanma jetonu sayısını bir azaltır.
        *   **Harita Bilgisi Erişimi:**
            *   `GetEventPassMapInfo() const`: Mevcut aktif etkinlik için geçit haritası bilgilerini döndürür.
            *   `GetEventSungZiMapInfo() const`: Mevcut aktif etkinlik için Sungzi haritası bilgilerini döndürür.
            *   `IsThreeWayWarMapIndex(int iMapIndex) const`: Verilen harita indeksinin bir Üç Yol Savaşı haritası olup olmadığını kontrol eder.
            *   `IsSungZiMapIndex(int iMapIndex) const`: Verilen harita indeksinin bir Sungzi haritası olup olmadığını kontrol eder.
        *   **Etkinlik Yönetimi:**
            *   `RandomEventMapSet()`: Rastgele bir Sungzi ve Pass haritası konfigürasyonunu aktif etkinlik olarak ayarlar (quest flag'leri aracılığıyla).
            *   `RegisterUser(DWORD PlayerID)`: Bir oyuncuyu savaşa kayıtlı olarak işaretler.
            *   `IsRegisteredUser(DWORD PlayerID) const`: Bir oyuncunun savaşa kayıtlı olup olmadığını kontrol eder.
            *   `onDead(LPCHARACTER pChar, LPCHARACTER pKiller)`: Bir karakter öldüğünde çağrılır; skorları günceller, savaşın gidişatını yönetir (krallıkların elenmesi, boss'un ortaya çıkması vb.).
            *   `SetRegenFlag(int flag)`, `GetRegenFlag() const`: Savaş sırasındaki canavar yeniden doğma aşamasını yönetir.
            *   `RemoveAllMonstersInThreeWay() const`: Tüm Üç Yol Savaşı haritalarındaki canavarları temizler.
        *   **Özel (Private) Üyeler:**
            *   `KillScore_[3]` (int): Üç krallığın öldürme skorları.
            *   `RegenFlag_` (int): Canavar yeniden doğma aşaması bayrağı.
            *   `MapIndexSet_` (std::set<int>): Tüm Üç Yol Savaşı harita indekslerini tutar.
            *   `PassInfoMap_` (std::vector<ForkedPassMapInfo>): Yüklenen tüm olası geçit haritası konfigürasyonlarını tutar.
            *   `SungZiInfoMap_` (std::vector<ForkedSungziMapInfo>): Yüklenen tüm olası Sungzi haritası konfigürasyonlarını tutar.
            *   `RegisterUserMap_` (boost::unordered_map<DWORD, DWORD>): Savaşa katılan oyuncuların ID'lerini tutar.
            *   `ReviveTokenMap_` (boost::unordered_map<DWORD, int>): Oyuncuların kalan canlanma jetonlarını tutar.
    *   **Yardımcı Global Fonksiyonlar:**
        *   `GetSungziMapPath()`, `GetPassMapPath(BYTE bEmpire)`: Aktif etkinlik haritalarının dosya yollarını döndürür.
        *   `GetPassMapIndex(BYTE bEmpire)`, `GetSungziMapIndex()`: Aktif etkinlik haritalarının indekslerini döndürür.
        *   `GetSungziStartX(BYTE bEmpire)`, `GetSungziStartY(BYTE bEmpire)`, `GetPassStartX(BYTE bEmpire)`, `GetPassStartY(BYTE bEmpire)`: Krallıkların aktif etkinlik haritalarındaki başlangıç koordinatlarını döndürür.
*   **Bağlantılı Dosyalar:** `threeway_war.cpp` (uygulama), `stdafx.h`, `<boost/unordered_map.hpp>`, `../../common/stl.h`.

---

### `threeway_war.cpp`

*   **Amaç:** `threeway_war.h` dosyasında tanımlanan `CThreeWayWar` sınıfının metotlarını ve Üç Yol Savaşı etkinliğiyle ilgili yardımcı fonksiyonları uygular. Etkinlik ayarlarının yüklenmesi, skorların takibi, oyuncu ölümlerinin işlenmesi, harita yönetimi ve canavar doğumu gibi işlevleri içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Event Fonksiyonu (`regen_mob_event`):**
        *   Belirli bir Sungzi haritasında canavarların yeniden doğmasını tetikleyen bir event'tir.
        *   `threeway_war_choice` quest flag'ine göre farklı `regen.txt` dosyalarını (`regen00.txt` veya `regen00_[choice].txt`) yükler.
    *   **`CThreeWayWar::Initialize()`:** Skorları, RegenFlag'ı ve kayıtlı oyuncu/canlanma jetonu haritalarını sıfırlar.
    *   **Skor ve Canlanma Jetonu Metotları:** `GetKillScore`, `SetKillScore`, `SetReviveTokenForPlayer`, `GetReviveTokenForPlayer`, `DecreaseReviveTokenForPlayer` ilgili verileri basitçe okur veya günceller.
    *   **`CThreeWayWar::LoadSetting(szFileName)`:**
        *   Belirtilen yapılandırma dosyasını (genellikle `threeway_war.txt`) okur.
        *   "sungzi:" ile başlayan satırlardan `ForkedSungziMapInfo` yapılarını okur ve `SungZiInfoMap_` vektörüne ekler. Harita indeksini `MapIndexSet_`'e kaydeder.
        *   "pass:" ile başlayan satırlardan `ForkedPassMapInfo` yapılarını okur ve `PassInfoMap_` vektörüne ekler. Harita indekslerini `MapIndexSet_`'e kaydeder.
    *   **Harita Bilgisi Erişim Metotları:** `GetEventPassMapInfo`, `GetEventSungZiMapInfo`, `IsThreeWayWarMapIndex`, `IsSungZiMapIndex` ilgili harita bilgilerini `PassInfoMap_`, `SungZiInfoMap_` ve `MapIndexSet_` üzerinden sorgular. `GetEvent*MapInfo` metotları mevcut aktif harita setini belirlemek için `threeway_war_pass_idx` ve `threeway_war_sungzi_idx` quest flag'lerini kullanır.
    *   **`CThreeWayWar::RandomEventMapSet()`:** `PassInfoMap_` ve `SungZiInfoMap_` vektörlerinden rastgele birer indeks seçerek ilgili quest flag'lerini ayarlar ve böylece bir sonraki savaş için harita setini belirler.
    *   **Kullanıcı Kayıt Metotları:** `RegisterUser` ve `IsRegisteredUser` metotları `RegisterUserMap_` üzerinden oyuncu kaydını yönetir.
    *   **`GetKillValue(level)` Fonksiyonu:** Öldürülen oyuncunun seviyesine göre bir öldürme puanı değeri döndürür (örn: 50-59 seviye arası 1 puan, 60-69 arası 2 puan).
    *   **`CThreeWayWar::onDead(pChar, pkKiller)`:**
        *   Ölen karakter PC değilse veya GM seviyesindeyse (test sunucusu hariç) veya `RegenFlag_ == -1` ise işlem yapmaz.
        *   Ölen oyuncunun canlanma jetonunu azaltır.
        *   Ölüm Sungzi haritasında gerçekleşmediyse veya öldüren PC değilse/aynı krallıktansa skor güncellemesi yapmaz.
        *   Öldürenin krallığının skorunu `GetKillValue` ile artırır.
        *   Belirli aralıklarla (her 5 skorda bir veya test sunucusunda her zaman) haritadaki oyunculara skor duyurusu yapar.
        *   **Savaşın Gidişatı:**
            *   **`RegenFlag_ == 0` (İlk Aşama):**
                *   `threeway_war_kill_count` quest flag'indeki zafer skorunu kontrol eder.
                *   Eğer bir krallık hariç diğer iki krallığın skoru bu zafer skorunun altındaysa (yani iki krallık elenmişse), bu elenen krallığı belirler (`bLoseEmpire`).
                *   Elenen krallığın skorunu -1 yapar, diğerlerinin skorunu sıfırlar.
                *   Elenen krallığın oyuncularını kendi başlangıç haritalarına ışınlamak için `warp_all_to_map_my_empire_event` event'leri oluşturur (Sungzi ve Pass haritalarından).
                *   Haritadaki oyunculara ve genel duyuruyla elenen krallığı bildirir.
                *   Canavarların yeniden doğması için `regen_mob_event` oluşturur.
                *   `RegenFlag_`'ı 1 yapar.
            *   **`RegenFlag_ == 1` (İkinci Aşama - Boss Aşaması):**
                *   Kalan iki krallıktan hangisinin `threeway_war_kill_count` zafer skoruna ulaştığını kontrol eder (`nVictoryEmpireIndex`).
                *   Eğer bir kazanan varsa, diğer kaybeden krallığın oyuncularını da başlangıç haritalarına ışınlar.
                *   Kazanan krallığın oyuncularına Sungzi haritasının boss'unu yenmeleri gerektiğini bildiren bir script gönderir.
                *   `threeway_war_boss_count` quest flag'i kadar boss canavarını (`GetEventSungZiMapInfo().m_iBossMobVnum`) Sungzi haritasında rastgele konumlarda doğurur.
                *   `RegenFlag_`'ı -1 yaparak savaşın bu aşamasını sonlandırır (skor sayımı durur).
    *   **`CThreeWayWar::RemoveAllMonstersInThreeWay()`:** `MapIndexSet_` içindeki tüm haritaları dolaşarak PC olmayan tüm karakterleri (`ENTITY_CHARACTER`) öldürür (`ch->Dead()`).
    *   **Global Yardımcı Fonksiyonlar:** `GetSungziMapPath`, `GetPassMapPath`, `GetPassMapIndex`, `GetSungziMapIndex`, `GetSungziStartX`, `GetSungziStartY`, `GetPassStartX`, `GetPassStartY` gibi fonksiyonlar `CThreeWayWar::instance()` üzerinden ilgili bilgileri alarak kolay erişim sağlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `threeway_war.h`, `../../common/length.h`, `../../common/tables.h`, `p2p.h`, `locale_service.h`, `packet.h`, `char.h`, `questmanager.h`, `questlua.h`, `start_position.h`, `char_manager.h`, `sectree_manager.h`, `regen.h`, `log.h`, `config.h`. 

---

### `war_map.h`

*   **Amaç:** Lonca savaşları için özel olarak oluşturulan haritaları (`CWarMap`) ve bu haritaları yöneten `CWarMapManager` singleton sınıfını tanımlar. Bu sistem, iki lonca arasındaki savaşın türüne (normal veya bayrak kapmaca) göre harita oluşturma, katılımcı yönetimi, skor takibi, olay yönetimi (başlangıç, bitiş, zaman aşımı) ve ödül hesaplama gibi işlevleri içerir.
*   **Temel İşlevler/İçerik:**
    *   **`EWarMapTypes` Enum'u:**
        *   `WAR_MAP_TYPE_NORMAL`: Normal lonca savaşı haritası.
        *   `WAR_MAP_TYPE_FLAG`: Bayrak kapmaca tipi lonca savaşı haritası.
    *   **`EWarMapIndex` Enum'u:**
        *   `WAR_MAP_INDEX_NORMAL = 110`: Normal savaş haritasının temel harita indeksi.
        *   `WAR_MAP_INDEX_FLAG = 111`: Bayrak savaşı haritasının temel harita indeksi.
    *   **`SWarMapInfo` (TWarMapInfo) Struct:**
        *   Bir savaş haritası türünün temel bilgilerini tutar:
            *   `bType` (BYTE): Savaş haritası türü (`EWarMapTypes`).
            *   `lMapIndex` (long): Temel harita indeksi (`EWarMapIndex`).
            *   `posStart[3]` (PIXEL_POSITION): Takımlar için başlangıç pozisyonları (genellikle 0 ve 1 kullanılır, 2 gözlemci veya başka bir amaç için olabilir).
    *   **`warmap` Namespace'i:**
        *   Bayrak kapmaca savaşı için bayrakların (taşınabilir ve sabit taban) VNUM'larını tanımlar.
        *   `WAR_FLAG_VNUM_START`, `WAR_FLAG_VNUM_END`: Taşınabilir bayrak VNUM aralığı.
        *   `WAR_FLAG_VNUM0`, `WAR_FLAG_VNUM1`, `WAR_FLAG_VNUM2`: Belirli taşınabilir bayrak VNUM'ları.
        *   `WAR_FLAG_BASE_VNUM`: Sabit bayrak üssü VNUM'u.
        *   `IsWarFlag(DWORD dwVnum)`: Verilen VNUM'un taşınabilir bir savaş bayrağı olup olmadığını kontrol eder.
        *   `IsWarFlagBase(DWORD dwVnum)`: Verilen VNUM'un sabit bir bayrak üssü olup olmadığını kontrol eder.
    *   **`CWarMap` Sınıfı:**
        *   Belirli bir lonca savaşı için oluşturulmuş özel bir harita örneğini yönetir.
        *   **`STeamData` Struct:** Savaşan her bir lonca için veri tutar:
            *   `dwID`: Lonca ID'si.
            *   `pkGuild`: Lonca nesnesine işaretçi.
            *   `iMemberCount`: Haritadaki mevcut lonca üye sayısı.
            *   `iUsePotionPrice`: Lonca üyelerinin savaş sırasında kullandığı iksirlerin toplam maliyeti.
            *   `iScore`: Loncanın savaş skoru.
            *   `pkChrFlag`: (Bayrak savaşında) Loncanın taşıdığı bayrağın karakter işaretçisi.
            *   `pkChrFlagBase`: (Bayrak savaşında) Loncanın bayrak üssünün karakter işaretçisi.
            *   `set_pidJoiner`: Savaşa katılan tüm üyelerin oyuncu ID'lerini tutan set (birikmiş katılım sayısı için).
        *   **Yapıcı/Yıkıcı:** Haritayı başlatır (takım verileri, olaylar), savaş bittiğinde kaynakları temizler.
        *   **Üye Yönetimi (`IncMember`, `DecMember`, `UpdateUserCount`):** Oyuncuların haritaya giriş/çıkışını yönetir, üye sayılarını günceller, gözlemci modunu ayarlar.
        *   **Skor ve Durum Yönetimi (`UpdateScore`, `CheckScore`, `SetEnded`, `CheckWarEnd`, `Timeout`, `Draw`):** Skorları günceller, savaşın bitiş koşullarını kontrol eder (skor, zaman aşımı, üye kalmaması), savaşı sonlandırır, ödülleri belirler.
        *   **Olay Yönetimi (`SetBeginEvent`, `SetTimeoutEvent`, `SetEndEvent`, `SetResetFlagEvent`):** Savaşın farklı aşamaları için olayları (timer) ayarlar ve iptal eder.
        *   **Bayrak Yönetimi (Bayrak savaşında) (`AddFlag`, `AddFlagBase`, `RemoveFlag`, `IsFlagOnBase`, `ResetFlag`):** Bayrakları ve üslerini haritaya yerleştirir, kaldırır, konumlarını kontrol eder.
        *   **Bilgi ve Paketleme (`Packet`, `Notice`, `SendWarPacket`, `SendScorePacket`):** Haritadaki oyunculara bilgi paketleri ve duyurular gönderir.
        *   **Yardımcılar (`GetTeamIndex`, `GetGuildID`, `GetGuild`, `GetType`, `GetMapIndex`, `GetGuildOpponent`, `GetWinnerGuild`, `UsePotion`, `GetRewardGold`, `OnKill`, `ExitAll`).**
    *   **`CWarMapManager` Sınıfı (Singleton):**
        *   Tüm aktif lonca savaşı haritalarını yönetir.
        *   `LoadWarMapInfo(filename)`: Savaş haritası türlerinin yapılandırma bilgilerini (harita indeksleri, başlangıç pozisyonları) yükler.
        *   `CreateWarMap(...)`: Yeni bir lonca savaşı için özel bir harita örneği (`CWarMap`) oluşturur ve yönetilen haritalar listesine ekler. `SECTREE_MANAGER::CreatePrivateMap` kullanarak haritanın fiziksel kopyasını oluşturur.
        *   `DestroyWarMap(CWarMap* pMap)`: Bir `CWarMap` örneğini ve ilişkili özel haritayı yok eder.
        *   `Find(lMapIndex)`: Verilen harita indeksine sahip aktif bir `CWarMap` bulur.
        *   `IsWarMap(lMapIndex)`: Bir harita indeksinin savaş haritası olup olmadığını kontrol eder.
        *   `GetWarMapInfo(lMapIndex)`: Bir harita türünün (`SWarMapInfo`) bilgilerini döndürür.
        *   `GetStartPosition(...)`: Belirli bir savaş haritası türü için bir takımın başlangıç pozisyonunu döndürür.
        *   `OnShutdown()`: Sunucu kapanırken tüm aktif savaşları berabere bitirir.
        *   `for_each(Func f)`: Tüm aktif savaş haritaları üzerinde bir fonksiyon uygular.
*   **Bağlantılı Dosyalar:** `war_map.cpp` (uygulama), `constants.h`, `guild.h`, `stdafx.h`, `char.h`, `item.h`.

### `war_map.cpp`

*   **Amaç:** `war_map.h` içinde bildirilen `CWarMap` ve `CWarMapManager` sınıflarının metotlarını uygular. Lonca savaşı haritalarının oluşturulması, oyuncu yönetimi, skor takibi, olay işleme ve bayrak mekanikleri gibi işlevleri gerçekleştirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Olay Fonksiyonları (`war_begin_event`, `war_end_event`, `war_timeout_event`, `war_reset_flag_event`):**
        *   `war_map_info` yapısını kullanarak ilgili `CWarMap` örneğine erişirler.
        *   `war_begin_event`: Savaşın periyodik olarak bitip bitmediğini kontrol eder (`CheckWarEnd`).
        *   `war_end_event`: Savaş bittiğinde çağrılır. İlk adımda tüm oyuncuları haritadan çıkarır (`ExitAll`), ikinci adımda haritayı yok eder (`CWarMapManager::instance().DestroyWarMap`).
        *   `war_timeout_event`: Savaş zaman aşımına uğradığında çağrılır (`pMap->Timeout()`).
        *   `war_reset_flag_event`: Bayrak savaşında bayraklar sıfırlandığında çağrılır (`pMap->AddFlag(0); pMap->AddFlag(1);`).
    *   **`CWarMap::CWarMap(...)` (Yapıcı):**
        *   Harita bilgilerini (`m_kMapInfo`, `m_WarInfo`) ayarlar.
        *   İki takımın (`m_TeamData`) verilerini başlatır, lonca nesnelerini (`CGuildManager::instance().TouchGuild`) alır.
        *   Başlangıç olayını (`war_begin_event`) ayarlar.
        *   Bayrak savaşı ise (`WAR_MAP_TYPE_FLAG`), bayrak üslerini ve bayrakları haritaya ekler (`AddFlagBase`, `AddFlag`).
    *   **`CWarMap::~CWarMap()` (Yıkıcı):**
        *   Tüm olayları iptal eder (`event_cancel`).
        *   Haritadaki tüm karakterlerin bağlantısını keser (`DESC_MANAGER::instance().DestroyDesc`).
    *   **`CWarMap::IncMember(ch)`, `CWarMap::DecMember(ch)`:**
        *   Oyuncunun loncasına göre doğru takıma ekler/çıkarır veya gözlemci yapar.
        *   Üye sayılarını (`STeamData::iMemberCount`, `STeamData::set_pidJoiner`) günceller.
        *   Zaman aşımı olayını iptal eder/başlatır.
        *   Bayrak savaşında, çıkan oyuncu bayrağı taşıyorsa bayrağı düşürür.
        *   Savaşın bitip bitmediğini kontrol eder (`CheckWarEnd`).
        *   Tüm katılımcılara güncel kullanıcı sayılarını gönderir (`UpdateUserCount`).
    *   **`CWarMap::CheckWarEnd()`:**
        *   Savaş zaten bitmişse (`m_bEnded`) veya zaman aşımı süreci başlamışsa (`m_pkTimeoutEvent`) bir şey yapmaz.
        *   Takımlardan birinin üyesi kalmamışsa, 60 saniyelik bir zaman aşımı olayı başlatır (`war_timeout_event`). Duyuru yapar.
        *   Aksi halde skor kontrolü yapar (`CheckScore`).
    *   **`CWarMap::Timeout()`:**
        *   Zaman aşımı olayı tetiklendiğinde çağrılır.
        *   Eğer savaş 5 dakikadan kısa sürmüşse berabere biter.
        *   Aksi halde, üyesi kalmayan takım kaybeder, diğeri kazanır. Eğer iki takımın da üyesi varsa, skora göre kazanan belirlenir.
        *   Kazanan loncaya ödül altını (`GetRewardGold`) hesaplar ve `CGuildManager::instance().RequestWarOver` ile savaşı sonlandırır.
    *   **`CWarMap::CheckScore()`:**
        *   Savaşın başlangıcından 30 saniye geçmemişse veya skorlar eşitse bir şey yapmaz.
        *   Takımlardan biri belirlenen bitiş skoruna (`m_WarInfo.iEndScore`) ulaşmışsa kazananı belirler, ödülü hesaplar ve `CGuildManager::instance().RequestWarOver` ile savaşı sonlandırır.
    *   **`CWarMap::SetEnded()`:**
        *   Savaş resmi olarak bittiğinde çağrılır (genellikle `CGuildManager` tarafından).
        *   Bayrak savaşında tüm bayrakları ve üsleri kaldırır.
        *   10 saniye sonra haritanın kapanması için `war_end_event` başlatır.
    *   **`CWarMap::OnKill(killer, ch)`:**
        *   Bir oyuncu öldüğünde çağrılır.
        *   Normal savaşta, öldürenin loncasına skor ekler (`SendGuildWarScore`).
        *   Bayrak savaşında, ölen oyuncu bayrağı taşıyorsa bayrağı öldüğü yere düşürür (`AddFlag`).
    *   **Bayrak Mekanikleri (`AddFlagBase`, `AddFlag`, `RemoveFlag`, `IsFlagOnBase`, `ResetFlag`):**
        *   Bayrakları ve üslerini belirli VNUM'larla (`warmap::WAR_FLAG_VNUM0/1`, `warmap::WAR_FLAG_BASE_VNUM`) haritada oluşturur (`CHARACTER_MANAGER::instance().SpawnMob`).
        *   Bayrakların `POINT_STAT`'ını lonca ID'sine ayarlar.
        *   Bir bayrak alındığında (`RemoveFlag`), karakter ölür.
        *   Bayrakların kendi üslerinde olup olmadığını kontrol eder.
        *   Tüm bayrakları sıfırlamak için kullanılır (`ResetFlag`), bu işlem tüm oyunculardaki bayrak taşıma efektini kaldırır ve bayrakları başlangıç konumlarına yerleştirir.
    *   **`CWarMapManager::LoadWarMapInfo(...)`:**
        *   Savaş haritası türlerinin (normal ve bayrak) temel bilgilerini (harita indeksi, başlangıç koordinatları) sabit olarak koddan yükler. Dosya okuma işlemi yapılmaz.
    *   **`CWarMapManager::CreateWarMap(...)`:**
        *   `SECTREE_MANAGER::instance().CreatePrivateMap` ile belirtilen temel harita indeksinden özel bir kopya oluşturur.
        *   Yeni bir `CWarMap` nesnesi oluşturur ve bunu `m_mapWarMap` haritasına ekler.
    *   **`CWarMapManager::DestroyWarMap(pMap)`:**
        *   `CWarMap` nesnesini siler ve `SECTREE_MANAGER::instance().DestroyPrivateMap` ile özel haritayı yok eder.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `war_map.h`, `sectree_manager.h`, `char.h`, `char_manager.h`, `affect.h`, `item.h`, `config.h`, `desc.h`, `desc_manager.h`, `guild_manager.h`, `buffer_manager.h`, `db.h`, `packet.h`, `locale_service.h`.