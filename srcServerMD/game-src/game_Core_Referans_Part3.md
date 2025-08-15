# Metin2 Oyun Sunucusu - Çekirdek Mekanikleri Referansı Part 3 (`game/src`)

**Not:** Bu belge, [`game_Core_Referans_Part2.md`](game_Core_Referans_Part2.md) dosyasının devamı niteliğindedir.

Bu belge, Metin2 oyun sunucusunun (`game/src`) çekirdek mekanikleriyle ilgili dosyalarını belgelemeye devam eder.

## İçindekiler

*   [`packet_info.h`](#packet_infoh)
*   [`packet_info.cpp`](#packet_infocpp)
*   [`packet_info_ws.cpp`](#packet_info_wscpp)
*   [`packet.h`](#packeth)
*   [`regen.h`](#regenh)
*   [`regen.cpp`](#regencpp)
*   [`safebox.h`](#safeboxh)
*   [`safebox.cpp`](#safeboxcpp)
*   [`sectree_manager.h`](#sectree_managerh)
*   [`sectree_manager.cpp`](#sectree_managercpp)
*   [`sectree.h`](#sectreeh)
*   [`sectree.cpp`](#sectreecpp)
*   [`sequence.h`](#sequenceh)
*   [`sequence.cpp`](#sequencecpp)
*   [`skill_power.h`](#skill_powerh)
*   [`skill_power.cpp`](#skill_powercpp)
*   [`skill.h`](#skillh)
*   [`skill.cpp`](#skillcpp)
*   [`start_position.h`](#start_positionh)
*   [`start_position.cpp`](#start_positioncpp)
*   [`stdafx.h`](#stdafxh)
*   [`trigger.cpp`](#triggercpp)

---

### `packet_info.h`

*   **Amaç:** Ağ paketleri hakkında bilgi (boyut, isim, çağrılma sayısı, yük vb.) saklamak ve yönetmek için temel sınıfları ve yapıları tanımlar. Özellikle sunucu performansını analiz etmek ve paket işleme yükünü izlemek için kullanılır. `CPacketInfo` ana sınıfını ve ondan türeyen `CPacketInfoCG` (Client->Game), `CPacketInfoGG` (Game->Game, P2P için) ve `CPacketInfoUDP` (UDP paketleri için) alt sınıflarını içerir.
*   **Temel İşlevler/İçeriği:**
    *   **`SPacketElement` (TPacketElement) Struct:** Tek bir paket türü hakkındaki bilgileri tutar:
        *   `iSize`: Paketin boyutu.
        *   `stName`: Paketin sembolik adı (örn. "Login", "Attack").
        *   `iCalled`: Paketin kaç kez işlendiği.
        *   `dwLoad`: Bu paketin işlenmesinin toplam süresi (milisaniye cinsinden).
        *   `bSequencePacket` (`__SEND_SEQUENCE__` tanımlıysa): Paketin bir sıra numarası içerip içermediğini belirtir.
    *   **`PacketMap` Typedef'i:** `std::map<int, TPacketElement*>` için bir takma ad. Paket başlığını (`header`) `TPacketElement` işaretçisine eşler.
    *   **`CPacketInfo` Sınıfı:**
        *   **Metotlar:**
            *   `Set(int header, int size, const char* c_pszName, bool bSeq)`: Yeni bir paket türünü kaydeder. Eğer `__SEND_SEQUENCE__` tanımlıysa ve `bSeq` true ise, paket boyutuna sıra numarası için ek bir byte ekler.
            *   `Get(int header, int* size, const char** c_ppszName)`: Belirli bir başlığa sahip paketin boyutunu ve adını alır, ayrıca `m_pCurrentPacket`'ı bu pakete ayarlar.
            *   `Start()`: `m_pCurrentPacket` için zamanlayıcıyı başlatır.
            *   `End()`: `m_pCurrentPacket` için zamanlayıcıyı durdurur, çağrılma sayısını ve toplam yükü günceller.
            *   `Log(const char* c_pszFileName)`: Tüm kayıtlı paketlerin istatistiklerini (isim, çağrılma sayısı, toplam yük, ortalama yük) belirtilen dosyaya yazar.
            *   `IsSequence(int header)` (`__SEND_SEQUENCE__` tanımlıysa): Paketin sıra numarası içerip içermediğini döndürür.
            *   `SetSequence(int header, bool bSeq)` (`__SEND_SEQUENCE__` tanımlıysa): Bir paketin sıra numarası durumunu ve buna bağlı olarak boyutunu günceller.
            *   `GetElement(int header)` (private): Başlığa göre `TPacketElement` döndürür.
        *   **Korunan (Protected) Üyeler:**
            *   `m_pPacketMap` (`PacketMap`): Tüm kayıtlı paket bilgilerini tutan harita.
            *   `m_pCurrentPacket` (`TPacketElement*`): Şu anda işlenmekte olan paket.
            *   `m_dwStartTime` (DWORD): `Start()` çağrıldığındaki zaman damgası.
    *   **`CPacketInfoCG` Sınıfı (CPacketInfo'dan türetilmiş):** İstemciden oyuna gelen (Client-to-Game) paketlerin bilgilerini tutar. Kurucusunda tüm CG paketlerini `Set()` ile kaydeder. Yıkıcısında `Log("packet_info.txt")` çağırır.
    *   **`CPacketInfoGG` Sınıfı (CPacketInfo'dan türetilmiş):** Oyun sunucuları arası (Game-to-Game, P2P) paketlerin bilgilerini tutar. Kurucusunda tüm GG paketlerini `Set()` ile kaydeder. Yıkıcısında `Log("p2p_packet_info.txt")` çağırır.
    *   **`CPacketInfoUDP` Sınıfı (CPacketInfo'dan türetilmiş):** UDP paketlerinin bilgilerini tutar. (Uygulaması `input_udp.cpp` dosyasındadır.)


### `packet_info.cpp`

*   **Amaç:** `packet_info.h` dosyasında tanımlanan `CPacketInfo`, `CPacketInfoCG` ve `CPacketInfoGG` sınıflarının metotlarını uygular. Bu dosya, özellikle istemciden gelen (CG) ve sunucular arası (GG) paketlerin tanımlanması ve istatistiklerinin toplanmasıyla ilgilenir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPacketInfo` Sınıfı Metotları:**
        *   Yapıcı: `m_pCurrentPacket`'ı `NULL`, `m_dwStartTime`'ı 0 yapar.
        *   Yıkıcı: `m_pPacketMap`'teki tüm `TPacketElement` nesnelerini siler (`M2_DELETE`).
        *   `Set`: Verilen başlık (`header`) zaten kayıtlı değilse, yeni bir `TPacketElement` oluşturur, bilgilerini (boyut, isim, çağrı sayısı=0, yük=0) ayarlar. Eğer `__SEND_SEQUENCE__` makrosu aktifse ve paket sıralı (`bSeq` true) ise, paketin boyutuna 1 byte (sekans numarası için) ekler. Sonra bu elementi `m_pPacketMap`'e ekler.
        *   `Get`: Verilen başlığa sahip `TPacketElement`'ı haritada bulur. Bulunursa, boyut ve isim bilgilerini çıktı parametrelerine atar, `m_pCurrentPacket`'ı bu elemente ayarlar ve `true` döndürür. Bulunamazsa `false` döndürür.
        *   `IsSequence`, `SetSequence` (`__SEND_SEQUENCE__` tanımlıysa): Paketlerin sıralı olup olmadığını kontrol eder veya ayarlar, gerektiğinde boyutlarını günceller.
        *   `GetElement`: Başlığa göre `TPacketElement`'ı haritadan bulup döndürür.
        *   `Start`: `m_pCurrentPacket`'ın `NULL` olmadığını `assert` ile kontrol eder, `get_dword_time()` ile başlangıç zamanını kaydeder.
        *   `End`: `m_pCurrentPacket`'ın çağrılma sayısını artırır ve `get_dword_time() - m_dwStartTime` ile geçen süreyi toplam yüke ekler.
        *   `Log`: Belirtilen dosyayı yazma modunda açar. Başlık satırını yazar. Ardından `m_pPacketMap` üzerinde dönerek her paket için adını, çağrılma sayısını, toplam yükünü ve ortalama yükünü (yük/çağrı) dosyaya yazdırır.
    *   **`CPacketInfoCG` Sınıfı:**
        *   Yapıcı: Çok sayıda `HEADER_CG_...` ile başlayan istemci paket başlığını, karşılık gelen paket yapılarının boyutlarını (`sizeof(TPacketCG...)`), sembolik isimlerini ve sıralı olup olmadıklarını (`bSeq` parametresi) `Set` metodunu kullanarak kaydeder. Birçok özellik (`__IMPROVED_PACKET_ENCRYPTION__`, `__ACCE_COSTUME_SYSTEM__` vb.) için koşullu derleme blokları içerir.
        *   Yıkıcı: `Log("packet_info.txt")` çağırarak istemci paket istatistiklerini dosyaya yazar.
    *   **`CPacketInfoGG` Sınıfı:**
        *   Yapıcı: Çok sayıda `HEADER_GG_...` ile başlayan sunucular arası P2P paket başlığını, karşılık gelen yapıların boyutlarını ve sembolik isimlerini `Set` metodunu kullanarak kaydeder (P2P paketleri genellikle sıralı değildir, `bSeq` false).
        *   Yıkıcı: `Log("p2p_packet_info.txt")` çağırarak P2P paket istatistiklerini dosyaya yazar.
*   **Not:** `packet_info_ws.cpp` dosyası ile `packet_info.cpp` dosyasının içeriği büyük ölçüde aynıdır veya çok benzerdir. Farklılıklar genellikle koşullu derleme direktiflerinde (`__SEND_SEQUENCE__` ve `__DISABLE_SEND_SEQUENCE__` gibi) ve `CPacketInfoCG` yapıcısındaki bazı paketlerin `bSeq` değerlerinde olabilir. Projenin yapılandırmasına veya derleme seçeneklerine bağlı olarak bu dosyalardan biri kullanılıyor olabilir.

### `packet_info_ws.cpp`

*   **Amaç:** Bu dosya, `packet_info.cpp` dosyasıyla çok benzer bir amaca hizmet eder: `packet_info.h`'de tanımlanan sınıfların metotlarını uygular ve özellikle istemciden gelen (CG) ve sunucular arası (GG) paketlerin bilgilerini kaydeder. "ws" soneki "without sequence" (sekanssız) veya belirli bir derleme yapılandırmasını (örneğin WebSocket veya farklı bir paket sıralama mekanizması) işaret ediyor olabilir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`CPacketInfo` Sınıfı Metotları:** `packet_info.cpp` dosyasındaki implementasyonla büyük ölçüde aynıdır. Temel fark, `Set` metodunun `__DISABLE_SEND_SEQUENCE__` ve `!defined(__DISABLE_SEND_SEQUENCE__)` makrolarıyla farklı şekillerde tanımlanmasıdır. Eğer `__DISABLE_SEND_SEQUENCE__` tanımlıysa, `Set` metodu `bSeq` parametresini almaz ve paket boyutuna sekans byte'ı eklenmez. Eğer tanımlı değilse (yani sekans aktifse), `bSeq` parametresini alır ve `packet_info.cpp`'deki gibi davranır.
    *   `IsSequence` ve `SetSequence` metotları da `!defined(__DISABLE_SEND_SEQUENCE__)` koşuluna bağlıdır.
    *   Diğer `CPacketInfo` metotları (`Get`, `Start`, `End`, `Log`, `GetElement`) `packet_info.cpp` ile aynıdır.
    *   **`CPacketInfoCG` Sınıfı:**
        *   Yapıcı: `packet_info.cpp`'deki `CPacketInfoCG` yapıcısına çok benzer şekilde istemci paketlerini kaydeder. Buradaki temel fark, `HEADER_CG_PHASE` paketinin `bSeq` parametresinin `false` olarak ayarlanmasıdır (oysa `packet_info.cpp`'de `true` idi). Diğer paketlerin `bSeq` değerleri genellikle aynıdır.
        *   Yıkıcı: `Log("packet_info.txt")` çağırır.
    *   **`CPacketInfoGG` Sınıfı:**
        *   Yapıcı ve Yıkıcı: `packet_info.cpp`'deki `CPacketInfoGG` ile tamamen aynıdır, P2P paketlerini kaydeder ve `p2p_packet_info.txt` dosyasına loglar.
*   **Genel Not:** İki `.cpp` dosyasının varlığı (`packet_info.cpp` ve `packet_info_ws.cpp`), muhtemelen farklı sunucu yapılandırmaları veya özellik setleri için farklı paket işleme stratejilerine (özellikle paket sıralamasıyla ilgili olarak) izin vermek amacıyla yapılmıştır. Derleme sırasında bu dosyalardan sadece biri linkleniyor olabilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `../../common/stl.h`, `constants.h`, `packet_info.h`.

### `packet.h`

*   **Amaç:** Bu dosya, Metin2 istemci-sunucu ve sunucular arası (P2P) iletişiminde kullanılan tüm ağ paketlerinin başlık (header) numaralarını ve veri yapılarını (`struct`) tanımlayan merkezi bir başlık dosyasıdır. İstemci ile oyun sunucusu (`game`) arasında gönderilen ve alınan verilerin formatını belirler.
*   **Temel İşlevler/İçeriği:**
    *   **Başlık Numaraları (Headers):**
        *   `CG_HEADERS` Enum'u: İstemciden Oyuna (Client-to-Game) gönderilen paketlerin başlık numaralarını tanımlar (örn: `HEADER_CG_LOGIN`, `HEADER_CG_MOVE`, `HEADER_CG_ATTACK`, `HEADER_CG_ITEM_USE`).
        *   `GC_HEADERS` Enum'u: Oyundan İstemciye (Game-to-Client) gönderilen paketlerin başlık numaralarını tanımlar (örn: `HEADER_GC_CHARACTER_ADD`, `HEADER_GC_CHAT`, `HEADER_GC_ITEM_SET`, `HEADER_GC_MOVE`).
        *   `GG_HEADERS` Enum'u: Oyun sunucuları arasında (Game-to-Game, P2P) gönderilen paketlerin başlık numaralarını tanımlar (örn: `HEADER_GG_LOGIN`, `HEADER_GG_LOGOUT`, `HEADER_GG_RELAY`, `HEADER_GG_GUILD`).
    *   **Paket Yapıları (Structs):**
        *   Her bir paket başlığına karşılık gelen veri yapılarını tanımlar. Bu yapılar genellikle `TPacketCG...`, `TPacketGC...`, `TPacketGG...` veya `packet_...`, `command_...` gibi öneklerle isimlendirilir.
        *   Her paket yapısının ilk üyesi genellikle paketin türünü belirten bir `BYTE` tipinde başlık numarasıdır (`bHeader` veya `header`).
        *   Yapılar, gönderilecek veya alınacak verilerin türünü, sırasını ve boyutunu tanımlar (örn. `DWORD dwVID`, `char szName[...]`, `long lX`, `TItemPos Cell`).
        *   Bazı paketler dinamik boyutludur ve boyut bilgisi içeren bir üye (`WORD size` gibi) veya alt başlık (`subheader`) içerebilirler.
    *   **Önemli Veri Türleri (Dolaylı Olarak Kullanılanlar):**
        *   `TPlayerItemAttribute`: Eşya bonuslarını tanımlar.
        *   `TItemPos`: Envanter veya ekipman penceresindeki konumu belirtir (pencere türü ve hücre indeksi).
        *   `TQuickslot`: Hızlı erişim çubuğundaki bir öğeyi tanımlar.
        *   `TSimplePlayer`: Karakter seçim ekranı gibi yerlerde kullanılan temel oyuncu bilgilerini içerir.
    *   **SubHeader'lar:** Bazı paketler (`TPacketCGShop`, `TPacketCGExchange`, `TPacketGCGuild` vb.) tek bir başlık altında farklı eylemleri ayırt etmek için `subheader` (alt başlık) kullanır.
    *   **Koşullu Derleme (`#if defined(...)`):** Dosya, çok sayıda `#if defined(__FEATURE_NAME__)` bloğu içerir. Bu, belirli oyun özelliklerinin (örn. `__ACCE_COSTUME_SYSTEM__`, `__GROWTH_PET_SYSTEM__`, `__PREMIUM_PRIVATE_SHOP__`) derleme zamanında etkinleştirilip devre dışı bırakılmasına olanak tanır. Yalnızca etkin özelliklerle ilgili paket başlıkları ve yapıları derlenir. Bu, kod tabanını modüler tutar ve gereksiz paketlerin dahil edilmesini önler.
    *   **Byte Hizalama (`#pragma pack(1)`):** Dosyanın sonunda `#pragma pack(1)` direktifi bulunur. Bu, derleyiciye yapı üyeleri arasına hizalama için ek boşluklar (padding) eklememesini söyler. Bu, ağ üzerinden gönderilen ve alınan verilerin farklı sistemlerde bile tutarlı bir şekilde yorumlanabilmesi için kritik öneme sahiptir.
*   **Genel Not:** Bu dosya, oyunun ağ iletişim protokolünün temelini oluşturur. İstemci ve sunucu arasındaki veya sunucular arasındaki tüm veri alışverişi, bu dosyada tanımlanan paket formatlarına göre yapılır. Dosyanın büyüklüğü ve içerdiği çok sayıda koşullu derleme bloğu, oyunun zaman içinde eklenen özelliklerle ne kadar geliştiğini göstermektedir.
*   **Bağlantılı Dosyalar:** Bu dosya, ağ iletişimiyle ilgili hemen hemen tüm `.cpp` dosyaları tarafından (`input_*.cpp`, `char.cpp`, `item.cpp`, `shop.cpp` vb.) ve paket bilgilerini yöneten `packet_info.cpp` gibi dosyalar tarafından dahil edilir. Ayrıca, `constants.h` gibi temel tanımları içeren dosyalara da bağımlıdır. 

### `regen.h`

**Amacı:** Bu başlık dosyası, oyun dünyasındaki canavar (mob) ve grup (group) yeniden doğma (regeneration/spawn) sistemiyle ilgili temel yapıları, sabitleri ve fonksiyon bildirimlerini içerir.

**Temel Bileşenler:**

*   **`REGEN_TYPE` Enum:** Yeniden doğma türlerini tanımlar:
    *   `REGEN_TYPE_MOB`: Tek bir canavar.
    *   `REGEN_TYPE_GROUP`: Bir canavar grubu.
    *   `REGEN_TYPE_EXCEPTION`: Yeniden doğmanın engellendiği bir alan.
    *   `REGEN_TYPE_GROUP_GROUP`: Bir grup içinde başka gruplar (iç içe gruplar).
    *   `REGEN_TYPE_ANYWHERE`: Haritanın herhangi bir yerinde rastgele.
*   **`regen` (REGEN) Struct:** Bir yeniden doğma noktasının tüm bilgilerini tutar:
    *   `prev`, `next`: Bağlı liste işaretçileri.
    *   `lMapIndex`: Harita indeksi.
    *   `type`: `REGEN_TYPE` enumundan türü.
    *   `sx`, `sy`, `ex`, `ey`: Yeniden doğma alanının başlangıç ve bitiş koordinatları.
    *   `z_section`: Z ekseni bölümü (özellikle iç mekanlar veya katmanlı yapılar için).
    *   `direction`: Başlangıç yönü.
    *   `time`: Yeniden doğma süresi (saniye cinsinden).
    *   `max_count`: Bu noktadan aynı anda en fazla kaç adet canavar/grup spawn olabileceği.
    *   `count`: Mevcut spawn olmuş canavar/grup sayısı.
    *   `vnum`: Spawn olacak canavarın veya grubun VNUM'u.
    *   `is_aggressive`: Spawn olanların agresif olup olmayacağı.
    *   `event`: Bu yeniden doğma noktasıyla ilişkili zamanlayıcı olayı (`LPEVENT`).
    *   `id`: Özellikle zindanlardaki yeniden doğma noktalarını ayırt etmek için kullanılan bir kimlik.
*   **`regen_event_info` Struct (EVENTINFO makrosu ile):** Standart yeniden doğma olayları için olay verisi. Bir `LPREGEN` (regen struct işaretçisi) içerir.
*   **`regen_exception` (REGEN_EXCEPTION) Struct:** Yeniden doğmanın yapılmayacağı istisnai bir alanı tanımlar.
    *   `prev`, `next`: Bağlı liste işaretçileri.
    *   `sx`, `sy`, `ex`, `ey`: İstisna alanının koordinatları.
    *   `z_section`: İstisna alanının Z ekseni bölümü.
*   **`dungeon_regen_event_info` Struct (EVENTINFO makrosu ile):** Zindana özgü yeniden doğma olayları için olay verisi. `LPREGEN` ve zindan kimliğini (`CDungeon::IdType dungeon_id`) içerir.
*   **Fonksiyon Bildirimleri (extern):**
    *   `regen_load(const char* filename, long lMapIndex, int base_x, int base_y)`: Bir harita için yeniden doğma verilerini dosyadan yükler.
    *   `regen_do(const char* filename, long lMapIndex, int base_x, int base_y, LPDUNGEON pDungeon, bool bOnce = true)`: Özellikle zindanlar veya tek seferlik spawn işlemleri için yeniden doğma verilerini işler. `bOnce` true ise sadece bir kez spawn eder.
    *   `regen_load_in_file(const char* filename, long lMapIndex, int base_x, int base_y)`: Dosyadan yeniden doğma verilerini yükler ve hemen tek seferlik spawn işlemi gerçekleştirir.
    *   `regen_free()`: Yüklenmiş tüm yeniden doğma verilerini ve ilişkili olayları temizler.
    *   `is_regen_exception(long x, long y)`: Verilen koordinatların bir yeniden doğma istisna alanı içinde olup olmadığını kontrol eder.
    *   `regen_reset(int x, int y)`: Yeniden doğma zamanlayıcılarını sıfırlar. Koordinat verilirse sadece o bölgedeki, verilmezse tüm zamanlayıcıları sıfırlar.

**Bağlantılı Dosyalar:** `dungeon.h`.

---

### `regen.cpp`

**Amacı:** Bu dosya, `regen.h` içinde tanımlanan yeniden doğma (regeneration) sistemi fonksiyonlarını ve mantığını uygular. Yeniden doğma dosyalarının okunması, canavar ve grupların periyodik olarak veya tek seferlik spawn edilmesi, istisnai alanların yönetimi ve ilgili olayların (event) işlenmesinden sorumludur.

**Temel İşlevler ve Implementasyon Detayları:**

*   **Global Değişkenler:**
    *   `regen_list` (`LPREGEN`): Tüm aktif yeniden doğma tanımlarının bağlı liste başı.
    *   `regen_exception_list` (`LPREGEN_EXCEPTION`): Tüm yeniden doğma istisna alanlarının bağlı liste başı.
*   **Dosya Okuma Yardımcıları:**
    *   `get_word(FILE* fp, char* buf)`: Dosyadan bir sonraki "kelimeyi" (token) okur. Boşluk, tab, yeni satır gibi ayraçları ve tırnak içindeki ifadeleri dikkate alır. `//` ile başlayan yorum satırlarını atlar.
    *   `next_line(FILE* fp)`: Dosyada bir sonraki satıra geçer.
    *   `read_line(FILE* fp, LPREGEN regen)`: Bir yeniden doğma dosyasından (örn. `map_a1/regen.txt`) tek bir satırı okur ve `REGEN` yapısını doldurur. Satırdaki verileri (`ERegenModes` enum'u ile tanımlanan modlara göre) ayrıştırır:
        *   **Tür (type):** `m` (mob), `g` (group), `ga` (aggressive group), `e` (exception), `r` (group_group), `s` (anywhere).
        *   **Koordinatlar (sx, sy, ex, ey):** Spawn alanı.
        *   **Z-Section:**
        *   **Yön (direction):**
        *   **Süre (regen_time):** `h` (saat), `m` (dakika), `s` (saniye) ile belirtilebilir.
        *   **Maksimum Sayı (max_count):**
        *   **VNUM:**
*   **Yeniden Doğma (Spawn) Mantığı:**
    *   `is_regen_exception(long x, long y)`: Verilen `x`, `y` koordinatlarının `regen_exception_list` içindeki herhangi bir istisna alanına denk gelip gelmediğini kontrol eder.
    *   `regen_spawn_dungeon(LPREGEN regen, LPDUNGEON pDungeon, bool bOnce)`: Zindana özel spawn işlemini gerçekleştirir.
        *   Gerekli sayıda (`regen->max_count - regen->count`) canavar/grup spawn eder.
        *   Spawn tipine (`REGEN_TYPE_ANYWHERE`, belirli koordinat, alan) göre `CHARACTER_MANAGER`'ın ilgili spawn fonksiyonlarını (`SpawnMobRandomPosition`, `SpawnMob`, `SpawnMobRange`, `SpawnGroup`, `SpawnGroupGroup`) kullanır.
        *   Spawn edilen canavarlara zindan bilgisini (`ch->SetDungeon(pDungeon)`) ve eğer periyodik bir spawn ise (`!bOnce`) yeniden doğma bilgisini (`ch->SetRegen(regen)`) atar.
    *   `regen_spawn(LPREGEN regen, bool bOnce)`: Normal haritalar için spawn işlemini yapar. `regen_spawn_dungeon`'a benzerdir ancak zindanla ilgili kısımları yoktur.
*   **Olay (Event) Fonksiyonları:**
    *   `dungeon_regen_event(EVENTINFO(dungeon_regen_event_info)* event_info)`: Zindan yeniden doğma olayları için tetiklenir. `regen_spawn_dungeon`'ı çağırır ve bir sonraki yeniden doğma için zamanlayıcıyı ayarlar.
    *   `regen_event(EVENTINFO(regen_event_info)* event_info)`: Normal harita yeniden doğma olayları için tetiklenir. `regen_spawn`'ı çağırır ve bir sonraki yeniden doğma için zamanlayıcıyı ayarlar.
*   **Yükleme Fonksiyonları:**
    *   `regen_do(const char* filename, long lMapIndex, int base_x, int base_y, LPDUNGEON pDungeon, bool bOnce)`: Genellikle zindanlar (`pDungeon` doluysa) veya tek seferlik spawnlar (`bOnce` true ise) için kullanılır. Dosyayı okur, `REGEN` yapılarını oluşturur. Zindanlar için `dungeon_regen_event` oluşturur ve ilk spawn'ı `regen_spawn_dungeon` ile yapar. Bazı özel harita indekslerini (114-117) atlar.
    *   `regen_load_in_file(const char* filename, long lMapIndex, int base_x, int base_y)`: Dosyadaki tüm tanımları okur ve `regen_spawn` ile tek seferlik spawn eder. Periyodik olay oluşturmaz.
    *   `regen_load(const char* filename, long lMapIndex, int base_x, int base_y)`: Haritalar için ana yeniden doğma yükleme fonksiyonudur.
        *   Dosyayı okur, her geçerli tanım için yeni bir `REGEN` nesnesi oluşturur ve global `regen_list`'e ekler.
        *   Koordinatları `base_x`, `base_y` ile ayarlar.
        *   Test sunucusunda ise (`test_server`) `CMobManager::instance().IncRegenCount` ile istatistik tutar.
        *   Eğer spawn olan bir NPC, Işınlanma Kapısı (WARP) veya Git Noktası (GOTO) ise, pozisyonunu `SECTREE_MANAGER::instance().InsertNPCPosition` ile kaydeder.
        *   Eğer yeniden doğma süresi (`regen->time`) 0 değilse (yani sürekli yeniden doğacaksa), ilk spawn'ı `regen_spawn` ile yapar ve periyodik `regen_event` oluşturur.
        *   Eğer tür `REGEN_TYPE_EXCEPTION` ise, yeni bir `REGEN_EXCEPTION` nesnesi oluşturur ve `regen_exception_list`'e ekler.
*   **Diğer Yardımcı Fonksiyonlar:**
    *   `regen_free()`: Tüm `REGEN` ve `REGEN_EXCEPTION` listelerindeki dinamik olarak ayrılmış belleği serbest bırakır ve ilişkili olayları iptal eder.
    *   `regen_reset(int x, int y)`: Aktif yeniden doğma olaylarının zamanlayıcılarını sıfırlar (hemen tetiklenmelerini sağlar). Eğer `x` ve `y` 0 değilse, sadece belirtilen dikdörtgen alan içindeki olayları sıfırlar.
*   **Global Bayrak:**
    *   `g_bNoRegen`: Eğer bu bayrak `true` ise, tüm `regen_load*` fonksiyonları erken çıkarak yeniden doğma sistemini devre dışı bırakır.

**Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `char.h`, `char_manager.h`, `regen.h`, `mob_manager.h`, `dungeon.h`, `../common/VnumHelper.h`. 

### `safebox.h`

**Amacı:** Bu başlık dosyası, oyuncunun eşya deposunu (Safebox) ve eşya marketi (Mall) deposunu yöneten `CSafebox` sınıfını tanımlar.

**Temel Bileşenler:**

*   **`CSafebox` Sınıfı:**
    *   **Amacı:** Bir oyuncuya ait depo veya eşya marketi içeriğini yönetir. Eşya ekleme, çıkarma, taşıma, altın miktarını ve depo boyutunu yönetme gibi işlevleri sağlar.
    *   **Yapıcı (`CSafebox(LPCHARACTER pkChrOwner, int iSize, DWORD dwGold)`):**
        *   Depo sahibini (`pkChrOwner`), başlangıç boyutunu (`iSize`) ve altın miktarını (`dwGold`) alır.
        *   Depo için grid (`CGrid`) nesnelerini (`v_Grid`) oluşturur (her 45 hücre için bir grid sayfası).
        *   Varsayılan pencere modunu `SAFEBOX` olarak ayarlar.
    *   **Yıkıcı (`~CSafebox()`):** `__Destroy()` metodunu çağırarak depodaki eşyaları ve gridleri temizler.
    *   **Metotlar:**
        *   `Add(DWORD dwPos, LPITEM pkItem)`: Belirtilen pozisyona bir eşya ekler.
        *   `Get(DWORD dwPos)`: Belirtilen pozisyondaki eşyayı döndürür.
        *   `Remove(DWORD dwPos)`: Belirtilen pozisyondaki eşyayı depodan çıkarır.
        *   `ChangeSize(int iSize)`: Depo boyutunu değiştirir (mevcut implementasyonda bir şey yapmıyor gibi görünüyor).
        *   `MoveItem(CellType bCell, CellType bDestCell, WORD count)`: Bir eşyayı depoda bir hücreden diğerine taşır veya aynı türden eşyaları istifler. `CellType`, `__EXTEND_SAFEBOX__` makrosuna göre `DWORD` veya `BYTE` olabilir.
        *   `GetItem(CellType bCell)`: Belirli bir hücredeki eşyayı döndürür.
        *   `Save()`: Depo bilgilerini (sadece altın) veritabanına kaydetmek için bir istek gönderir.
        *   `IsEmpty(DWORD dwPos, SizeType bSize)`: Belirtilen pozisyonun verilen boyuttaki bir eşya için boş olup olmadığını kontrol eder. `SizeType`, `__EXTEND_SAFEBOX__` makrosuna göre `DWORD` veya `BYTE` olabilir.
        *   `GetEmptySafebox(DWORD size)` (`__SAFEBOX_IMPROVING__` tanımlıysa): Belirtilen boyuttaki bir eşya için depoda boş bir yer arar ve pozisyonunu döndürür.
        *   `IsValidPosition(DWORD dwPos)`: Verilen pozisyonun depo sınırları içinde geçerli olup olmadığını kontrol eder.
        *   `SetWindowMode(BYTE bWindowMode)`: Pencere modunu ayarlar (`SAFEBOX` veya `MALL`). Bu, istemciye gönderilecek paket başlıklarını etkiler.
        *   `GetGridTotalSize() const`: Deponun toplam hücre sayısını döndürür.
    *   **Korunan (Protected) Metotlar:**
        *   `__Destroy()`: Depodaki tüm eşyaları (`m_pkItems`) ve gridleri (`v_Grid`) temizler. Eşyaların kaydedilmesini atlar ve bellekten siler.
    *   **Korunan (Protected) Üyeler:**
        *   `m_pkChrOwner` (`LPCHARACTER`): Deponun sahibi olan karakter.
        *   `m_pkItems` (`LPITEM[SAFEBOX_MAX_NUM]`): Depodaki eşyaların işaretçilerini tutan dizi. `SAFEBOX_MAX_NUM` genellikle depo + envanter toplamı gibi büyük bir sayıdır.
        *   `m_pkGrid` (`CGrid*`): Eski veya kullanılmayan bir grid işaretçisi gibi görünüyor, çünkü `v_Grid` kullanılıyor.
        *   `m_iSize` (int): Deponun toplam satır sayısı (her satır 9 hücre).
        *   `m_lGold` (long): Depodaki altın miktarı.
        *   `m_bWindowMode` (BYTE): Mevcut pencere modu (`SAFEBOX` veya `MALL`).
        *   `v_Grid` (`std::vector<std::shared_ptr<CGrid>>`): Depo sayfalarını temsil eden `CGrid` nesnelerinin vektörü. Her grid genellikle 5x9 boyutundadır.

**Bağlantılı Dosyalar:** `item.h` (LPITEM), `grid.h` (CGrid).

---

### `safebox.cpp`

**Amacı:** Bu dosya, `safebox.h` içinde tanımlanan `CSafebox` sınıfının metotlarını uygular. Oyuncu deposu ve eşya marketi deposunun işlevselliğini yönetir.

**Temel İşlevler ve Implementasyon Detayları:**

*   **Yapıcı (`CSafebox::CSafebox`) ve Yıkıcı (`CSafebox::~CSafebox`):**
    *   Yapıcı, depo sahibini, boyutunu ve altınını ayarlar, `m_pkItems` dizisini sıfırlar ve `v_Grid` için gerekli sayıda `CGrid` (5x9) nesnesi oluşturur. Pencere modunu `SAFEBOX` olarak başlatır.
    *   Yıkıcı, `__Destroy` metodunu çağırır.
*   **`CSafebox::__Destroy()`:**
    *   `m_pkItems` dizisindeki tüm geçerli eşyalar üzerinde döner.
    *   Her eşya için `SetSkipSave(true)` ayarlar (veritabanına kaydedilmesini engeller).
    *   `ITEM_MANAGER::instance().FlushDelayedSave(m_pkItems[i])` ile bekleyen kaydetme işlemlerini (varsa) hemen uygular/iptal eder.
    *   `M2_DESTROY_ITEM(m_pkItems[i]->RemoveFromCharacter())` ile eşyayı karakterden ayırır ve bellekten siler.
    *   `m_pkItems[i]` işaretçisini `NULL` yapar.
    *   `v_Grid` vektörünü temizler.
*   **`PosToPage(DWORD dwPos)` ve `PosToLocal(DWORD dwPos, DWORD Page)` Static Fonksiyonları:**
    *   `PosToPage`: Genel pozisyonu (0-MAX) sayfa numarasına çevirir (her sayfa 45 hücre).
    *   `PosToLocal`: Genel pozisyonu o sayfa içindeki yerel pozisyona çevirir.
*   **`CSafebox::Add(DWORD dwPos, LPITEM pkItem)`:**
    *   Pozisyonun geçerli olup olmadığını kontrol eder.
    *   Eşyanın pencere modunu (`m_bWindowMode`) ve hücre pozisyonunu ayarlar.
    *   Eşyayı kaydeder (`pkItem->Save()`) ve `FlushDelayedSave` çağırır.
    *   Doğru grid sayfasını (`v_Grid.at(Page)`) alır ve eşyayı o grid üzerine yerleştirir (`Put`).
    *   Eşyayı `m_pkItems` dizisine ekler.
    *   İstemciye `HEADER_GC_SAFEBOX_SET` veya `HEADER_GC_MALL_SET` paketi ile eşya bilgilerini gönderir. Koşullu derleme blokları (`__SOUL_BIND_SYSTEM__`, `__ITEM_APPLY_RANDOM__` vb.) ile farklı eşya özelliklerini pakete dahil eder.
*   **`CSafebox::Get(DWORD dwPos)`:**
    *   Pozisyon geçerliyse `m_pkItems[dwPos]`'u döndürür, değilse `nullptr`.
*   **`CSafebox::Remove(DWORD dwPos)`:**
    *   Belirtilen pozisyondaki eşyayı alır (`Get`).
    *   Eşya varsa, ilgili grid sayfasından kaldırır (`v_Grid.at(Page)->Get(...)`).
    *   Eşyayı karakterden ayırır (`pkItem->RemoveFromCharacter()`).
    *   `m_pkItems[dwPos]`'u `NULL` yapar.
    *   İstemciye `HEADER_GC_SAFEBOX_DEL` veya `HEADER_GC_MALL_DEL` paketi gönderir.
    *   Çıkarılan eşyayı döndürür.
*   **`CSafebox::Save()`:**
    *   Bir `TSafeboxTable` yapısı oluşturur, hesap ID'sini ve mevcut altın miktarını (`m_lGold`) ayarlar.
    *   `HEADER_GD_SAFEBOX_SAVE` başlığıyla bu bilgiyi veritabanı istemcisine (`db_clientdesc`) gönderir.
*   **`CSafebox::IsEmpty(DWORD dwPos, SizeType bSize)`:**
    *   Pozisyonun ait olduğu grid sayfasını bulur.
    *   O grid sayfasındaki `IsEmpty` metodunu çağırarak belirtilen pozisyonun verilen boyut için boş olup olmadığını kontrol eder.
*   **`CSafebox::GetEmptySafebox(DWORD size)` (`__SAFEBOX_IMPROVING__`):**
    *   Depodaki tüm hücreleri tek tek dolaşarak (`GetGridTotalSize()`), `IsEmpty` ile belirtilen boyutta bir eşya için boş yer arar. Bulursa hücre indeksini, bulamazsa -1 döndürür.
*   **`CSafebox::ChangeSize(int iSize)`:**
    *   Mevcut implementasyonda herhangi bir işlem yapmaz, sadece return eder. Depo boyutu değişimi farklı bir mekanizma ile yönetiliyor olabilir.
*   **`CSafebox::GetItem(CellType bCell)`:**
    *   `Get(DWORD dwPos)` ile benzer, ancak hücre indeksi `bCell` ile alınır.
*   **`CSafebox::MoveItem(CellType bCell, CellType bDestCell, WORD count)`:**
    *   Kaynak ve hedef hücrelerin geçerliliğini kontrol eder.
    *   Kaynak hücredeki eşyayı alır. Eşya yoksa veya takas durumundaysa işlem yapmaz.
    *   Taşınacak miktar (`count`) eşyanın mevcut miktarından fazlaysa işlem yapmaz.
    *   **İstifleme (Stacking):** Eğer hedef hücrede aynı VNUM'a, aynı soketlere sahip ve istiflenebilir başka bir eşya varsa, `count` kadarını o eşyanın üzerine ekler. Kaynak eşyanın miktarını azaltır veya tamamen kaldırır.
    *   **Normal Taşıma:** Eğer hedef hücre boşsa veya farklı bir eşya varsa:
        *   Kaynak ve hedef pozisyonların grid sayfalarını alır.
        *   Grid üzerinde eşyayı önce kaynaktan alır (`Get`), sonra hedefe koymaya çalışır (`Put`). Başarısız olursa geri alır. Bu kısım biraz karmaşık görünüyor ve eşyanın gerçekten taşınmasından önce sadece grid üzerinde bir kontrol gibi duruyor, asıl taşıma `Remove` ve `Add` ile yapılıyor.
        *   Eşyayı kaynak pozisyondan `Remove` ile alır ve hedef pozisyona `Add` ile ekler.
*   **`CSafebox::GetGridTotalSize() const`:**
    *   `v_Grid.size() * 45` (her grid sayfası 45 hücre) ile toplam hücre sayısını döndürür.
*   **`CSafebox::IsValidPosition(DWORD dwPos)`:**
    *   Verilen pozisyonun `GetGridTotalSize()`'dan küçük olup olmadığını kontrol eder.

**Bağlantılı Dosyalar:** `stdafx.h`, `grid.h`, `constants.h`, `safebox.h`, `packet.h`, `char.h`, `desc_client.h`, `item.h`, `item_manager.h`, `config.h`. 

### `sectree_manager.h`

**Amacı:** Bu başlık dosyası, oyun dünyasındaki haritaları sektörlere (sectree) ayıran ve bu sektörleri yöneten `SECTREE_MAP` ve `SECTREE_MANAGER` sınıflarını tanımlar. Harita özelliklerinin yüklenmesi, varlıkların konumlandırılması ve sorgulanması gibi temel harita yönetimi işlevlerini içerir.

**Temel Bileşenler:**

*   **`SMapRegion` (TMapRegion) Struct:** Bir harita bölgesinin sınırlarını, başlangıç noktasını, imparatorluğa özel başlangıç noktalarını ve harita adını tutar.
*   **`TAreaInfo` Struct:** Genellikle zindanlar içindeki belirli alanların (spawn bölgeleri, özel odalar vb.) koordinatlarını ve yönünü tanımlar.
*   **`npc_info` Struct:** NPC'nin türünü, adını ve konumunu saklar. Mini haritada gösterilecek NPC'ler için kullanılır.
*   **`SSetting` (TMapSetting) Struct:** Bir haritanın temel ayarlarını (indeks, hücre ölçeği, taban koordinatları, genişlik, yükseklik, varsayılan başlangıç noktası) içerir. `Setting.txt` dosyasından okunur.
*   **`SECTREE_MAP` Sınıfı:**
    *   **Amacı:** Tek bir harita indeksine ait tüm sektörleri (`LPSECTREE`) yönetir.
    *   **Typedef'ler:**
        *   `MapType`: `DWORD` (sektör paket ID'si) anahtarını `LPSECTREE` (sektör işaretçisi) değerine eşleyen bir harita.
    *   **Metotlar:**
        *   `SECTREE_MAP()`: Kurucu.
        *   `SECTREE_MAP(SECTREE_MAP& r)`: Kopyalayıcı kurucu.
        *   `~SECTREE_MAP()`: Yıkıcı. Sektörleri siler.
        *   `Add(DWORD key, LPSECTREE sectree)`: Haritaya yeni bir sektör ekler.
        *   `Find(DWORD dwPackage)`: Verilen paket ID'sine sahip sektörü bulur.
        *   `Find(DWORD x, DWORD y)`: Verilen koordinatlara denk gelen sektörü bulur.
        *   `Build()`: Haritadaki tüm sektörler için komşu sektör listelerini oluşturur.
        *   `for_each(Func& rfunc)`: Haritadaki tüm sektörlerde bulunan tüm varlıklar (entity) üzerinde belirli bir işlevi (`rfunc`) çalıştırır.
    *   **Üyeler:**
        *   `m_setting` (`TMapSetting`): Bu harita için yüklenmiş ayarlar.
        *   `map_` (`MapType`): Sektörleri tutan asıl harita.
*   **`EAttrRegionMode` Enum:** Bölgesel özellik (attribute) işlemlerinin modunu belirtir (`ATTR_REGION_MODE_SET`, `ATTR_REGION_MODE_REMOVE`, `ATTR_REGION_MODE_CHECK`).
*   **Typedef'ler:**
    *   `pkAreaMap`: `std::map<int, TAreaMap>` (Harita indeksini `TAreaMap`'e eşler).
    *   `NPCInfoVector`: `std::vector<npc_info>`.
    *   `NPCPositionMap`: `std::map<DWORD, NPCInfoVector>` (Harita indeksini NPC bilgilerinin vektörüne eşler).
*   **`SECTREE_MANAGER` Sınıfı (Singleton):**
    *   **Amacı:** Oyundaki tüm haritaların `SECTREE_MAP` örneklerini yönetir. Harita yükleme, varlık konumu bulma, özel harita (zindan) oluşturma/yok etme gibi genel işlemleri sağlar.
    *   **Metotlar (Önemlileri):**
        *   `GetMap(long lMapIndex)`: Belirtilen harita indeksine ait `SECTREE_MAP`'i döndürür.
        *   `Get(DWORD dwIndex, DWORD package)` / `Get(DWORD dwIndex, DWORD x, DWORD y)`: Belirli bir haritadaki sektörü döndürür.
        *   `LoadSettingFile(...)`: `Setting.txt` dosyasını okuyup `TMapSetting` yapısını doldurur.
        *   `LoadMapRegion(...)`: `Town.txt` dosyasını okuyup harita bölgesi ve başlangıç noktalarını yükler.
        *   `Build(const char* c_pszListFileName, const char* c_pszBasePath)`: Ana harita yükleme fonksiyonu. Verilen listedeki tüm haritaları ve ilgili verilerini (ayarlar, bölgeler, özellikler, regen dosyaları) yükler.
        *   `BuildSectreeFromSetting(...)`: `TMapSetting`'e göre sektörleri oluşturur.
        *   `LoadAttribute(...)`: `server_attr` dosyasından harita özelliklerini yükler.
        *   `LoadDungeon(...)`: `dungeon.txt` dosyasından zindan alan bilgilerini yükler.
        *   Konum Bulma Fonksiyonları: `GetValidLocation`, `GetSpawnPosition`, `GetSpawnPositionByMapIndex`, `GetRecallPositionByEmpire`, `GetCenterPositionOfMap`, `GetRandomLocation`, `GetMovablePosition`, `IsMovablePosition`.
        *   `CreatePrivateMap(long lMapIndex)`: Belirtilen haritanın bir kopyasını (özel zindan/alan) oluşturur ve yeni harita indeksini döndürür.
        *   `DestroyPrivateMap(long lMapIndex)`: Özel bir haritayı ve içindeki tüm varlıkları yok eder.
        *   `SendNPCPosition(LPCHARACTER ch)`: Oyuncuya mini haritada gösterilecek NPC pozisyonlarını gönderir.
        *   `InsertNPCPosition(...)`: Mini haritada gösterilecek bir NPC pozisyonu ekler.
        *   `GetEmpireFromMapIndex(long lMapIndex)`: Harita indeksine göre imparatorluk ID'sini döndürür.
        *   Varlık Temizleme Fonksiyonları: `PurgeMonstersInMap`, `PurgeStonesInMap`, `PurgeNPCsInMap`.
        *   `GetMonsterCountInMap(...)`: Haritadaki toplam canavar sayısını veya belirli bir VNUM'a sahip canavar sayısını döndürür.
        *   `ForAttrRegion(...)`: Belirli bir bölgedeki harita özelliklerini ayarlar, kaldırır veya kontrol eder.
        *   `SaveAttributeToImage(...)`: Harita özelliklerini bir TGA resmi olarak kaydeder (hata ayıklama için).
        *   `AddRestartCityPos(...)`, `GetRestartCityPos(...)`: Belirli haritalar için özel başlangıç şehir pozisyonlarını yönetir.
        *   `GeneratePreloadedEntitiesMap`, `ExtendPreloadedEntitiesMap`, `SendPreloadEntitiesPacket` (`__ENTITY_PRELOADING__` tanımlıysa): Varlıkların istemciye ön yüklenmesiyle ilgili fonksiyonlar.
    *   **Üyeler:**
        *   `m_map_pkSectree`: Harita indeksini `LPSECTREE_MAP`'e eşleyen ana harita.
        *   `m_map_pkArea`: Zindan alan bilgilerini tutar.
        *   `m_vec_mapRegion`: Yüklenmiş tüm harita bölgelerinin vektörü.
        *   `m_mapNPCPosition`: Mini harita NPC pozisyonlarını tutar.
        *   `next_private_index_map_`: Özel harita indekslerini takip etmek için kullanılır.
        *   `m_preloadedEntities` (`__ENTITY_PRELOADING__`): Ön yüklenecek varlık ırklarını tutar.

**Bağlantılı Dosyalar:** `sectree.h`, `<unordered_set>`.

---

### `sectree_manager.cpp`

**Amacı:** Bu dosya, `sectree_manager.h` içinde tanımlanan `SECTREE_MAP` ve `SECTREE_MANAGER` sınıflarının metotlarını uygular. Oyun dünyasındaki haritaların yüklenmesi, sektör bazlı yönetimi, harita özelliklerinin (attributes) işlenmesi, varlıkların (entity) konumlandırılması ve özel harita (zindan) örneklerinin oluşturulup yönetilmesinden sorumludur.

**Temel İşlevler ve Implementasyon Detayları:**

*   **`SECTREE_MAP` Sınıfı Uygulaması:**
    *   **`SECTREE_MAP::Build()`:** Bir harita yüklendikten sonra çağrılır. Haritadaki her sektör (`LPSECTREE`) için komşu sektörlerin bir listesini (`m_neighbor_list`) oluşturur. Bu, bir varlık hareket ettiğinde veya bir yetenek kullanıldığında etki alanındaki diğer sektörleri hızlıca bulmak için kullanılır.
*   **`SECTREE_MANAGER` Sınıfı Uygulaması:**
    *   **Harita Veri Yükleme Sırası (Genel):**
        1.  `SECTREE_MANAGER::Build(const char* c_pszListFileName, const char* c_pszMapBasePath)`:
            *   `map_list.txt` (veya benzeri) gibi bir liste dosyasını okur. Bu dosya, yüklenecek haritaların indekslerini ve dizin adlarını içerir.
            *   Her harita için:
                *   `LoadSettingFile`: `<map_path>/Setting.txt` dosyasından harita boyutu, taban koordinatları, hücre ölçeği gibi temel ayarları yükler.
                *   `LoadMapRegion`: `<map_path>/Town.txt` dosyasından harita bölgesi sınırlarını, genel başlangıç noktasını ve imparatorluğa özel başlangıç noktalarını yükler.
                *   Eğer harita sunucuda izinliyse (`map_allow_find`):
                    *   `BuildSectreeFromSetting`: Yüklenen ayarlara göre harita için `SECTREE_MAP` ve içindeki `LPSECTREE` nesnelerini oluşturur.
                    *   `LoadAttribute`: `<map_path>/server_attr` dosyasından sıkıştırılmış (LZO) harita özellik verilerini okur, açar ve her sektörün `CAttribute` nesnesine bağlar. Bu özellikler, bir hücrenin geçilebilir olup olmadığını, su olup olmadığını, PK yasağı olup olmadığını vb. belirler.
                    *   `regen_load`: `<map_path>/regen.txt`, `npc.txt`, `boss.txt`, `stone.txt` dosyalarını `regen.cpp`'deki fonksiyonlar aracılığıyla yükleyerek canavar, NPC, boss ve metin taşı spawn noktalarını tanımlar.
                    *   `LoadDungeon`: `<map_path>/dungeon.txt` dosyasından zindan içindeki özel alanları (örneğin, boss odası koordinatları) yükler.
                    *   `pkMapSectree->Build()`: Yüklenen haritanın sektörleri için komşu listelerini oluşturur.
                    *   `GeneratePreloadedEntitiesMap` (`__ENTITY_PRELOADING__`): Varsa, istemciye önceden gönderilecek varlık ırklarının bir listesini oluşturur.
    *   **Konum ve Bölge Fonksiyonları:**
        *   `GetMap(long lMapIndex)`: Harita indeksine göre `LPSECTREE_MAP` döndürür.
        *   `Get(DWORD dwIndex, DWORD x, DWORD y)`: Belirli bir haritadaki sektörü döndürür.
        *   `IsMovablePosition`, `GetMovablePosition`: Bir konumun geçilebilir olup olmadığını kontrol eder veya yakındaki geçilebilir bir konumu bulur.
        *   `GetValidLocation`: Verilen koordinatların geçerli bir harita ve sektör içinde olup olmadığını kontrol eder.
        *   `GetRandomLocation`: Bir harita üzerinde rastgele, geçilebilir bir konum bulur.
        *   `GetRecallPositionByEmpire`, `GetSpawnPositionByMapIndex`, `GetCenterPositionOfMap`: Çeşitli senaryolar için özel konumları (ışınlanma, başlangıç, merkez) döndürür.
    *   **Özel Harita (Zindan) Yönetimi:**
        *   `CreatePrivateMap(long lMapIndex)`: Bir ana haritanın (`lMapIndex`) bir kopyasını oluşturarak özel bir zindan/alan yaratır. Yeni bir harita indeksi (genellikle `lMapIndex * 10000 + i` formatında) atanır. Ana haritanın `SECTREE_MAP`'i kopyalanır.
        *   `DestroyPrivateMap(long lMapIndex)`: Özel bir haritayı ve içindeki tüm varlıkları sunucudan kaldırır.
    *   **Harita Özellikleri (Attribute) Yönetimi:**
        *   `ForAttrRegionCell`, `ForAttrRegionRightAngle`, `ForAttrRegionFreeAngle`, `ForAttrRegion`: Belirli bir geometrik alandaki (hücre, dikdörtgen, serbest açılı dörtgen) harita özelliklerini toplu olarak ayarlamak, kaldırmak veya kontrol etmek için kullanılır. Bu, oyun yöneticisi komutları veya özel olaylar için kullanılabilir.
        *   `SaveAttributeToImage`: Haritanın özelliklerini (engeller, su vb.) bir TGA resim dosyası olarak kaydeder. Bu, genellikle harita tasarımı ve hata ayıklama sırasında görselleştirme için kullanılır.
    *   **Varlık (Entity) Yönetimi:**
        *   `PurgeMonstersInMap`, `PurgeStonesInMap`, `PurgeNPCsInMap`: Belirli bir haritadaki tüm canavarları, metin taşlarını veya NPC'leri temizler.
        *   `GetMonsterCountInMap`: Bir haritadaki toplam canavar sayısını veya belirli bir VNUM'a sahip canavar sayısını döndürür.
    *   **NPC Pozisyonları (Mini Harita):**
        *   `InsertNPCPosition`: `regen_load` sırasında NPC, ışınlanma kapısı veya git noktası (GOTO) türünde bir varlık spawn edildiğinde, bu fonksiyon çağrılarak konumu `m_mapNPCPosition`'a eklenir.
        *   `SendNPCPosition`: Oyuncu bir haritaya girdiğinde veya istediğinde, o harita için `m_mapNPCPosition`'da kayıtlı olan NPC'lerin (tür, isim, x, y) listesini istemciye gönderir. Bu, istemcinin mini haritasında NPC ikonlarını göstermesini sağlar.
    *   **Diğer:**
        *   `GetEmpireFromMapIndex`: Harita indeksine göre hangi imparatorluğa ait olduğunu belirler.
        *   `AddRestartCityPos`, `GetRestartCityPos`: Belirli haritalar için imparatorluğa özel yeniden başlama noktalarını yönetir.
        *   `__ENTITY_PRELOADING__` ile ilgili fonksiyonlar: İstemcinin belirli haritalara girmeden önce bazı varlık türlerini (ırk numaralarıyla) önceden yüklemesini sağlayarak görünürlüğü artırmayı ve yükleme sürelerini azaltmayı hedefler.

**Bağlantılı Dosyalar:** `stdafx.h`, `sstream`, `targa.h`, `attribute.h`, `config.h`, `utils.h`, `sectree_manager.h`, `regen.h`, `lzo_manager.h`, `desc.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `item.h`, `item_manager.h`, `buffer_manager.h`, `packet.h`, `start_position.h`, `dev_log.h`. 

### `sectree.h`

**Amacı:** Bu başlık dosyası, oyun dünyasındaki bir haritanın temel yapı taşı olan sektörleri (`SECTREE`) temsil eden sınıfı ve ilgili yapıları tanımlar. Her bir sektör, belirli bir alanı kapsar ve o alandaki varlıkları (karakterler, eşyalar) ve harita özelliklerini (engeller, su vb.) yönetir.

**Temel Bileşenler:**

*   **`ESectree` Enum:** Sektör boyutlarını tanımlar (`SECTREE_SIZE = 6400`, `SECTREE_HALF_SIZE = 3200`, `CELL_SIZE = 50`).
*   **`sectree_coord` Struct:** Bir sektörün 2D koordinatlarını (x, y) tutar.
*   **`sectreeid` Union (SECTREEID):** Bir sektörün kimliğini temsil eder. Koordinatları (`coord`) veya paketlenmiş bir `DWORD` (`package`) olarak erişilebilir. Bu, sektörleri hem koordinat bazlı hem de tek bir ID ile yönetmeyi sağlar.
*   **Attribute Sabitleri:** Harita özelliklerini temsil eden bit maskesi sabitleri (`ATTR_BLOCK`, `ATTR_WATER`, `ATTR_BANPK`, `ATTR_OBJECT`).
*   **`FCollectEntity` Struct:** Bir sektördeki veya komşu sektörlerdeki varlıkları (entity) toplamak için kullanılan bir yardımcı yapıdır. `ForEachAround` gibi fonksiyonlarda geçici bir listeye varlıkları ekler ve sonra bu liste üzerinde işlem yapar. Bu, döngü sırasında setin değişmesinden kaynaklanabilecek sorunları önler.
*   **`SECTREE` Sınıfı:**
    *   **Amacı:** Oyun haritasının belirli bir 6400x6400'lük karesel alanını (sektör) temsil eder. O sektör içindeki varlıkları ve harita özelliklerini yönetir.
    *   **`find_if(_Func& func)` Template Metodu:** Komşu sektörlerdeki varlıklar arasında belirli bir koşulu (`func`) sağlayan ilk varlığı bulur ve döndürür.
    *   **`ForEachAround(_Func& func)` Template Metodu:** Mevcut sektör ve tüm komşu sektörlerdeki varlıklar üzerinde belirli bir işlevi (`func`) çalıştırır. Varlıkları önce `FCollectEntity` ile toplar, sonra topladığı liste üzerinde işlevi çağırır.
    *   **`for_each_for_find_victim(_Func& func)` Template Metodu:** Komşu sektörlerdeki varlıklar üzerinde, `func` true döndürdüğü anda aramayı durduracak şekilde bir işlevi çalıştırır. Hedef bulma optimizasyonları için kullanılır.
    *   **Metotlar (Önemlileri):**
        *   `SECTREE()`: Kurucu. `Initialize()`'ı çağırır.
        *   `~SECTREE()`: Yıkıcı. `Destroy()`'u çağırır.
        *   `Initialize()`: Sektör ID'sini, attribute işaretçisini ve PC sayısını sıfırlar.
        *   `Destroy()`: Sektördeki tüm varlıkları (hata mesajı vererek) yok eder ve attribute nesnesini (eğer klon değilse) siler.
        *   `GetID()`: Sektör ID'sini döndürür.
        *   `InsertEntity(LPENTITY ent)`: Verilen varlığı bu sektöre ekler. Varlığın eski sektöründen çıkarır, yeni sektörünü ayarlar ve PC sayısı sayaçlarını günceller.
        *   `RemoveEntity(LPENTITY ent)`: Verilen varlığı bu sektörden çıkarır. PC sayısı sayaçlarını günceller.
        *   `IncreasePC() / DecreasePC()`: Bu sektör ve komşu sektörlerdeki PC sayısını artırır/azaltır. PC sayısı 0'a düştüğünde sektördeki NPC'lerin durum makinelerini durdurabilir.
        *   `BindAttribute(CAttribute* pkAttribute)`: Sektöre harita özelliklerini içeren `CAttribute` nesnesini bağlar.
        *   `CloneAttribute(LPSECTREE tree)`: Başka bir sektörün attribute nesnesini paylaşır (özel haritalar/zindanlar için).
        *   `GetAttribute(long x, long y)`: Verilen koordinattaki harita özelliklerini döndürür.
        *   `IsAttr(long x, long y, DWORD dwFlag)`: Verilen koordinatın belirli bir özelliğe (`dwFlag`) sahip olup olmadığını kontrol eder.
        *   `GetEventAttribute(long x, long y)`: Koordinattaki olay numarasını (attribute'un üst byte'ları) döndürür.
        *   `SetAttribute(DWORD x, DWORD y, DWORD dwAttr)` / `RemoveAttribute(DWORD x, DWORD y, DWORD dwAttr)`: Belirli bir hücredeki harita özelliklerini ayarlar veya kaldırır.
    *   **Özel (Private) Metotlar:**
        *   `for_each_entity(_Func& func)`: Sadece bu sektördeki varlıklar üzerinde bir işlevi çalıştırır.
    *   **Üyeler:**
        *   `m_id` (`SECTREEID`): Sektörün kimliği.
        *   `m_set_entity` (`ENTITY_SET`): Sektördeki varlıkların (karakter, eşya vb.) seti.
        *   `m_neighbor_list` (`LPSECTREE_LIST`): Komşu sektörlerin listesi (kendisi dahil).
        *   `m_iPCCount` (int): Bu sektör ve komşu sektörlerdeki toplam oyuncu karakteri sayısı. NPC yapay zekasını optimize etmek için kullanılır.
        *   `isClone` (bool): Attribute nesnesinin klonlanıp klonlanmadığını belirtir.
        *   `m_pkAttribute` (`CAttribute*`): Bu sektörün harita özelliklerini tutan nesne.

**Bağlantılı Dosyalar:** `entity.h`.

---

### `sectree.cpp`

**Amacı:** Bu dosya, `sectree.h` içinde tanımlanan `SECTREE` sınıfının metotlarını uygular. Oyun haritasının bir sektörünü yönetir, sektör içindeki varlıkların (entity) eklenmesi/çıkarılması, oyuncu (PC) sayısının takibi ve harita özelliklerinin (attribute) sorgulanması/değiştirilmesi gibi işlevleri gerçekleştirir.

**Temel İşlevler ve Implementasyon Detayları:**

*   **Yapıcı/Yıkıcı ve Başlatma:**
    *   `SECTREE::SECTREE()`: `Initialize()` çağırır.
    *   `SECTREE::~SECTREE()`: `Destroy()` çağırır.
    *   `SECTREE::Initialize()`: Sektör ID'sini, attribute işaretçisini (`m_pkAttribute`) `NULL`'a, PC sayısını (`m_iPCCount`) 0'a ve klon durumunu (`isClone`) `false`'a ayarlar.
    *   `SECTREE::Destroy()`:
        *   Sektördeki varlık seti (`m_set_entity`) boş değilse hata mesajı verir ve setteki tüm varlıkları türlerine göre (karakter veya eşya) yok etmeye çalışır. Karakterler için eğer bir bağlantısı (`DESC`) varsa bağlantıyı kapatır, yoksa karakteri doğrudan yok eder. Eşyaları doğrudan yok eder.
        *   Varlık setini temizler (`m_set_entity.clear()`).
        *   Eğer attribute nesnesi klonlanmamışsa (`!isClone`) ve mevcutsa (`m_pkAttribute`), onu siler.
*   **Varlık Yönetimi (`InsertEntity`, `RemoveEntity`):**
    *   `InsertEntity(LPENTITY pkEnt)`:
        *   Varlığın mevcut sektörü bu sektörle aynıysa veya varlık zaten bu sektördeyse false döner.
        *   Varlığın eski sektöründen çıkarır (`pkCurTree->m_set_entity.erase(pkEnt)`).
        *   Varlığın yeni sektörünü bu sektör olarak ayarlar (`pkEnt->SetSectree(this)`).
        *   Varlığı bu sektörün setine ekler (`m_set_entity.insert(pkEnt)`).
        *   Eğer eklenen varlık bir PC ise:
            *   `IncreasePC()` çağırır.
            *   Eğer eski bir sektörü varsa `pkCurTree->DecreasePC()` çağırır.
        *   Eğer eklenen varlık bir PC değilse (NPC, canavar vb.) ve bu sektörde PC varsa (`m_iPCCount > 0`), karakterin durum makinesini başlatır (`pkChr->StartStateMachine()`), böylece yapay zeka aktifleşir.
    *   `RemoveEntity(LPENTITY pkEnt)`:
        *   Varlığı setten bulur ve siler.
        *   Varlığın sektörünü `NULL` yapar.
        *   Eğer çıkarılan varlık bir PC ise `DecreasePC()` çağırır.
*   **PC Sayısı Yönetimi (`IncreasePC`, `DecreasePC`):**
    *   `IncreasePC()`: Mevcut sektör ve tüm komşu sektörlerin `m_iPCCount` değerini 1 artırır.
    *   `DecreasePC()`: Mevcut sektör ve tüm komşu sektörlerin `m_iPCCount` değerini 1 azaltır.
        *   Eğer bir sektörün `m_iPCCount` değeri 0 veya altına düşerse (hata durumunda 0'a eşitlenir), o sektördeki tüm karakterlerin durum makinelerini durdurur (`ch->StopStateMachine()`). Bu, etrafta oyuncu yokken NPC'lerin/canavarların gereksiz yere işlem yapmasını engeller.
*   **Harita Özellikleri (Attribute) Yönetimi:**
    *   `BindAttribute(CAttribute* pkAttribute)`: Verilen `CAttribute` nesnesini sektöre bağlar.
    *   `CloneAttribute(LPSECTREE tree)`: Başka bir sektörün (`tree`) attribute nesnesini (`m_pkAttribute`) bu sektöre atar ve `isClone` bayrağını `true` yapar. Bu, özel haritaların ana haritanın özelliklerini paylaşmasını sağlar, bellek tasarrufu yapar.
    *   `SetAttribute(DWORD x, DWORD y, DWORD dwAttr)`: Koordinatları sektörün yerel koordinatlarına çevirerek `m_pkAttribute->Set` çağırır.
    *   `RemoveAttribute(DWORD x, DWORD y, DWORD dwAttr)`: Koordinatları sektörün yerel koordinatlarına çevirerek `m_pkAttribute->Remove` çağırır.
    *   `GetAttribute(long x, long y)`: Koordinatları sektörün yerel koordinatlarına çevirerek (`% SECTREE_SIZE / CELL_SIZE`) `m_pkAttribute->Get` çağırır ve özellik değerini döndürür.
    *   `IsAttr(long x, long y, DWORD dwFlag)`: `GetAttribute` ile özelliği alır ve istenen bayrağın (`dwFlag`) kurulu olup olmadığını kontrol eder.
    *   `GetEventAttribute(long x, long y)`: `GetAttribute` ile özelliği alır ve 8 bit sağa kaydırarak olay numarasını döndürür.

**Bağlantılı Dosyalar:** `stdafx.h`, `attribute.h`, `sectree_manager.h`, `char.h`, `char_manager.h`, `item.h`, `item_manager.h`, `desc_manager.h`, `packet.h`. 

### `sequence.h`

**Amacı:** Bu başlık dosyası, ağ paketleri için bir sıralama (sequence) mekanizmasıyla ilgili sabitleri ve harici bir byte dizisini tanımlar. Bu mekanizma, genellikle paketlerin doğru sırada işlenmesini sağlamak veya bazı güvenlik kontrolleri için kullanılır.

**Temel Bileşenler:**

*   **`__SEND_SEQUENCE__` Makrosu:** Bu makro tanımlıysa, sıralama mekanizması aktif hale gelir.
*   **`SEQUENCE_MAX_NUM` Sabiti:** Eğer `__SEND_SEQUENCE__` tanımlıysa, bu sabit 32768 olarak tanımlanır. Bu, `gc_abSequence` dizisinin maksimum boyutunu belirtir.
*   **`gc_abSequence` Dizisi (extern const BYTE[]):** Eğer `__SEND_SEQUENCE__` tanımlıysa, bu harici sabit byte dizisi bildirilir. Bu dizi, sıralama veya şifreleme için kullanılacak önceden tanımlanmış bir veri kümesini içerir. Asıl tanımı ve başlatılması `sequence.cpp` dosyasındadır.

**Bağlantılı Dosyalar:** Bu dosya genellikle `sequence.cpp` ve paket sıralaması kullanan ağ ile ilgili diğer modüller tarafından dahil edilir.

---

### `sequence.cpp`

**Amacı:** Bu dosya, `sequence.h` içinde bildirilen `gc_abSequence` adlı global byte dizisini tanımlar ve başlatır. Bu dizi, ağ paketlerinin sıralanması veya paket şifreleme anahtarları gibi amaçlarla kullanılabilir.

**Temel İşlevler ve Implementasyon Detayları:**

*   **`__SEND_SEQUENCE__` Makrosu:** Dosyanın içeriği, bu makronun tanımlı olup olmamasına bağlıdır. Eğer tanımlı değilse, dosya neredeyse boştur.
*   **`gc_abSequence` Dizisinin Tanımlanması:**
    *   Eğer `__SEND_SEQUENCE__` tanımlıysa, `gc_abSequence` dizisi `SEQUENCE_MAX_NUM` (32768) boyutunda bir `const BYTE` dizisi olarak tanımlanır.
    *   Dizi, çok sayıda önceden belirlenmiş onaltılık (hexadecimal) byte değeri ile başlatılır. Bu değerler, muhtemelen belirli bir algoritma ile üretilmiş veya rastgele seçilmiş sabitlerdir ve paket işleme sırasında kullanılır.

**Kullanım Senaryosu (Olası):**

Bu `gc_abSequence` dizisi, istemci ve sunucu arasında gönderilen paketlere bir sıra numarası veya basit bir XOR şifreleme anahtarı eklemek için kullanılabilir. İstemci ve sunucu aynı diziye ve aynı mantığa sahip olduğunda, paketlerin bütünlüğünü ve sırasını doğrulamak mümkün olabilir. Örneğin, gönderilen her paketin içeriği veya başlığı, bu diziden alınan bir sonraki byte ile XOR'lanabilir.

**Bağlantılı Dosyalar:** `stdafx.h`, `sequence.h`.

---

### `skill_power.h`

**Amacı:** Bu başlık dosyası, karakterlerin yetenek (skill) güçlerini ve seviyeye bağlı ek hasarlarını yöneten tabloları içeren `CTableBySkill` singleton sınıfını tanımlar.

**Temel Bileşenler:**

*   **`CTableBySkill` Sınıfı (Singleton):**
    *   **Amacı:** Yeteneklerin seviyelerine, karakterin işine (job) ve yetenek grubuna göre temel güçlerini ve karakter seviyesine göre ek yetenek hasarını saklayan tabloları yönetir.
    *   **Yapıcı (`CTableBySkill()`):**
        *   `m_aiSkillDamageByLevel` işaretçisini `NULL` olarak başlatır.
        *   `m_aiSkillPowerByLevelFromType` dizisindeki tüm işaretçileri (`JOB_MAX_NUM * 2` adet) `NULL` olarak başlatır. Bu dizi, her iş (job) ve her yetenek grubu (genellikle 2 grup) için ayrı bir yetenek gücü tablosu tutar.
    *   **Yıkıcı (`~CTableBySkill()`):**
        *   `DeleteAll()` metodunu çağırarak tüm dinamik olarak ayrılmış bellekleri serbest bırakır.
    *   **Metotlar:**
        *   `Check() const`: Tüm yetenek gücü tablolarının (`m_aiSkillPowerByLevelFromType`) düzgün bir şekilde ayarlanıp ayarlanmadığını kontrol eder. Herhangi biri `NULL` ise hata mesajı basar ve `false` döndürür.
        *   `DeleteAll()`: Tüm yetenek gücü tablolarını (`DeleteSkillPowerByLevelFromType` çağırarak) ve seviyeye bağlı ek hasar tablosunu (`DeleteSkillDamageByLevelTable` çağırarak) siler.
        *   `GetSkillPowerByLevelFromType(int job, int skillgroup, int skilllevel, bool bMob) const`: Belirtilen iş, yetenek grubu ve yetenek seviyesi için yetenek gücünü döndürür.
            *   Eğer `bMob` true ise, canavarlar için varsayılan yetenek gücü tablosundan (`m_aiSkillPowerByLevelFromType[0]`) değeri alır.
            *   Oyuncular için, iş ve yetenek grubuna göre doğru indeksi hesaplar (`(job * 2) + (skillgroup - 1)`) ve ilgili tablodan değeri döndürür.
            *   Geçersiz iş veya yetenek grubu için 0 döndürür.
        *   `SetSkillPowerByLevelFromType(int idx, const int* aTable)`: Belirli bir indeks (iş/yetenek grubu kombinasyonu) için yetenek gücü tablosunu ayarlar. Mevcut tabloyu siler, yeni bir dizi oluşturur ve verilen `aTable` içeriğini kopyalar.
        *   `DeleteSkillPowerByLevelFromType(int idx)`: Belirli bir indeksteki yetenek gücü tablosunu siler ve işaretçiyi `NULL` yapar.
        *   `GetSkillDamageByLevel(int Level) const`: Karakterin seviyesine (`Level`) göre ek yetenek hasarını döndürür. Geçersiz seviye için 0 döndürür.
        *   `SetSkillDamageByLevelTable(const int* aTable)`: Seviyeye bağlı ek yetenek hasarı tablosunu ayarlar. Mevcut tabloyu siler, yeni bir dizi oluşturur ve verilen `aTable` içeriğini kopyalar.
        *   `DeleteSkillDamageByLevelTable()`: Seviyeye bağlı ek yetenek hasarı tablosunu siler ve işaretçiyi `NULL` yapar.
    *   **Özel Üyeler:**
        *   `m_aiSkillPowerByLevelFromType[JOB_MAX_NUM * 2]` (`int*` dizisi): Her iş ve yetenek grubu için yetenek seviyesine göre (0'dan `SKILL_MAX_LEVEL`'e kadar) yetenek güçlerini tutan dinamik dizilerin işaretçilerini saklar.
        *   `m_aiSkillDamageByLevel` (`int*`): Karakter seviyesine göre (0'dan `PLAYER_MAX_LEVEL_CONST - 1`'e kadar) ek yetenek hasarını tutan dinamik dizinin işaretçisi.

**Bağlantılı Dosyalar:** `stdafx.h` (temel başlıklar), `../../common/length.h` (muhtemelen `JOB_MAX_NUM`, `SKILL_MAX_LEVEL`, `PLAYER_MAX_LEVEL_CONST` gibi sabitler için).

---

### `skill_power.cpp`

**Amacı:** Bu dosya, `skill_power.h` içinde tanımlanan `CTableBySkill` sınıfının metotlarını uygular. Yetenek gücü ve seviyeye bağlı ek hasar tablolarının oluşturulması, ayarlanması, sorgulanması ve silinmesi işlevlerini yerine getirir.

**Orta Seviye Implementasyon Detayları:**

*   **`CTableBySkill::Check() const`:**
    *   `m_aiSkillPowerByLevelFromType` dizisindeki tüm işaretçileri (her iş/yetenek grubu için) kontrol eder.
    *   Herhangi bir işaretçi `NULL` ise, standart hata akışına (`stderr`) bir hata mesajı yazdırır (örneğin, "[NO SETTING SKILL] aiSkillPowerByLevelFromType[job_index]") ve `false` döndürür.
    *   Tüm tablolar ayarlanmışsa `true` döndürür.
*   **`CTableBySkill::DeleteAll()`:**
    *   `JOB_MAX_NUM * 2` boyunca döner ve her indeks için `DeleteSkillPowerByLevelFromType(job_index)` metodunu çağırır.
    *   `DeleteSkillDamageByLevelTable()` metodunu çağırır.
*   **`CTableBySkill::GetSkillPowerByLevelFromType(int job, int skillgroup, int skilllevel, bool bMob) const`:**
    *   Eğer `bMob` (canavar) `true` ise, `m_aiSkillPowerByLevelFromType[0][skilllevel]` değerini döndürür (indeks 0 canavarlar için ayrılmıştır).
    *   Eğer `job` geçersiz (`>= JOB_MAX_NUM`) veya `skillgroup` 0 ise, 0 döndürür.
    *   Oyuncular için doğru tablo indeksini `idx = (job * 2) + (skillgroup - 1)` formülüyle hesaplar.
    *   `m_aiSkillPowerByLevelFromType[idx][skilllevel]` değerini döndürür.
*   **`CTableBySkill::SetSkillPowerByLevelFromType(int idx, const int* aTable)`:**
    *   Önce `DeleteSkillPowerByLevelFromType(idx)` ile varsa mevcut tabloyu siler.
    *   `SKILL_MAX_LEVEL + 1` boyutunda yeni bir `int` dizisi (`aiSkillTable`) oluşturur (`M2_NEW`).
    *   `memcpy` kullanarak `aTable` içeriğini bu yeni oluşturulan `aiSkillTable` dizisine kopyalar.
    *   `m_aiSkillPowerByLevelFromType[idx]` işaretçisini bu yeni `aiSkillTable` dizisine ayarlar.
*   **`CTableBySkill::DeleteSkillPowerByLevelFromType(int idx)`:**
    *   Eğer `m_aiSkillPowerByLevelFromType[idx]` `NULL` değilse, `M2_DELETE_ARRAY` ile işaret ettiği belleği serbest bırakır.
    *   Ardından `m_aiSkillPowerByLevelFromType[idx]` işaretçisini `NULL` yapar.
*   **`CTableBySkill::GetSkillDamageByLevel(int Level) const`:**
    *   Eğer `Level` geçersizse (0'dan küçük veya `PLAYER_MAX_LEVEL_CONST`'den büyük/eşitse), 0 döndürür.
    *   `m_aiSkillDamageByLevel[Level]` değerini döndürür.
*   **`CTableBySkill::SetSkillDamageByLevelTable(const int* aTable)`:**
    *   Önce `DeleteSkillDamageByLevelTable()` ile varsa mevcut tabloyu siler.
    *   `PLAYER_MAX_LEVEL_CONST` boyutunda yeni bir `int` dizisi (`aiSkillDamageTable`) oluşturur (`M2_NEW`).
    *   `memcpy` kullanarak `aTable` içeriğini bu yeni oluşturulan `aiSkillDamageTable` dizisine kopyalar.
    *   `m_aiSkillDamageByLevel` işaretçisini bu yeni `aiSkillDamageTable` dizisine ayarlar.
*   **`CTableBySkill::DeleteSkillDamageByLevelTable()`:**
    *   Eğer `m_aiSkillDamageByLevel` `NULL` değilse, `M2_DELETE_ARRAY` ile işaret ettiği belleği serbest bırakır.
    *   Ardından `m_aiSkillDamageByLevel` işaretçisini `NULL` yapar.

**Bağlantılı Dosyalar:** `stdafx.h`, `../../common/length.h`, `skill_power.h`.

---

### `skill.h`

**Amacı:** Bu başlık dosyası, oyundaki tüm yeteneklerin (skill) prototiplerini (`CSkillProto`) ve bu prototipleri yöneten `CSkillManager` sınıfını tanımlar. Ayrıca yeteneklerle ilgili çeşitli enum sabitlerini (bayraklar, VNUM'lar) içerir.

**Temel Bileşenler:**

*   **`ESkillFlags` Enum'u:** Yeteneklerin davranışlarını ve özelliklerini belirleyen bit bayraklarını tanımlar. Örneğin:
    *   `SKILL_FLAG_ATTACK`: Saldırı yeteneği.
    *   `SKILL_FLAG_USE_MELEE_DAMAGE`: Yakın dövüş hasarını kullanır.
    *   `SKILL_FLAG_SELFONLY`: Sadece kendine uygulanabilir.
    *   `SKILL_FLAG_SPLASH`: Alan etkili.
    *   `SKILL_FLAG_SLOW`, `SKILL_FLAG_STUN`, `SKILL_FLAG_POISON`: Durum etkileri.
    *   `SKILL_FLAG_TOGGLE`: Açılıp kapatılabilir yetenek.
    *   `SKILL_FLAG_PARTY` (`__SKILL_FLAG_PARTY__` ile): Parti üyelerini etkiler.
*   **Diğer Enum Sabitleri:**
    *   `SKILL_PENALTY_DURATION`: Yetenek kullanım sonrası ceza süresi.
    *   `SKILL_TYPE_HORSE`: At yetenekleri için özel bir tür.
    *   `ESkillIndexes`: Oyundaki tüm yeteneklerin VNUM (veya ID) karşılıklarını tanımlar (örn: `SKILL_SAMYEON`, `SKILL_GEOMKYUNG`, `SKILL_LEADERSHIP`, `GUILD_SKILL_START` vb.). Bu, yetenekleri kod içinde sabit isimlerle referans almayı kolaylaştırır.
*   **`SBonusSkillDamagePointMatch` Struct'ı (`__ATTR_6TH_7TH__` ile):**
    *   Yetenek VNUM'unu, o yeteneğin hasarını artıran özel bir karakter puanı (POINT) VNUM'u ile eşleştirir. 6. ve 7. efsunlarla gelen yetenek hasarı bonusları için kullanılır.
*   **`CSkillProto` Sınıfı:**
    *   **Amacı:** Tek bir yeteneğin tüm statik özelliklerini ve davranışlarını tanımlayan bir prototip (şablon) görevi görür.
    *   **Üyeler:**
        *   `szName[64]`: Yeteneğin adı.
        *   `dwVnum`: Yeteneğin benzersiz VNUM'u.
        *   `dwType`: Yeteneğin türü (örn: saldırı, destek, pasif).
        *   `bMaxLevel`: Yeteneğin maksimum seviyesi.
        *   `bLevelLimit`: Yeteneği öğrenmek için gereken minimum karakter seviyesi.
        *   `iSplashRange`: Alan etkili yetenekler için etki alanı yarıçapı.
        *   `bPointOn`, `wPointOn2`, `wPointOn3`: Yeteneğin etki ettiği karakter puanları (POINT_MAX_HP, POINT_ATT_SPEED vb.).
        *   `kPointPoly`, `kPointPoly2`, `kPointPoly3` (`CPoly`): Yeteneğin seviyesine göre etki miktarını hesaplayan polinomlar.
        *   `kSPCostPoly`, `kGrandMasterAddSPCostPoly` (`CPoly`): Yeteneğin SP maliyetini hesaplayan polinomlar.
        *   `kDurationPoly`, `kDurationPoly2`, `kDurationPoly3` (`CPoly`): Yeteneğin etki süresini hesaplayan polinomlar.
        *   `kDurationSPCostPoly` (`CPoly`): Sürekli aktif kalan yetenekler için saniye başına SP maliyetini hesaplayan polinom.
        *   `kCooldownPoly` (`CPoly`): Yeteneğin bekleme süresini hesaplayan polinom.
        *   `kMasterBonusPoly` (`CPoly`): Yetenek "Master" seviyesine ulaştığında eklenen bonusu hesaplayan polinom.
        *   `kSplashAroundDamageAdjustPoly` (`CPoly`): Alan etkili yeteneklerde merkeze uzaklığa göre hasar ayarlamasını hesaplayan polinom.
        *   `dwFlag`, `dwFlag2`: Yeteneğin davranışlarını belirleyen `ESkillFlags` bitleri.
        *   `dwAffectFlag`, `dwAffectFlag2`: Yeteneğin uyguladığı/kaldırdığı etkileri (affect) belirleyen bit bayrakları.
        *   `bLevelStep`: Yeteneğin kaç seviyede bir geliştirilebileceği.
        *   `preSkillVnum`, `preSkillLevel`: Bu yeteneği öğrenmek için ön koşul olan başka bir yetenek ve onun seviyesi.
        *   `lMaxHit`: Yeteneğin maksimum vuruş sayısı.
        *   `bSkillAttrType`: Yetenek için kullanılan özel bir özellik türü.
        *   `dwTargetRange`: Yeteneğin hedef menzili.
    *   **Metotlar:**
        *   `IsChargeSkill()`: Yeteneğin bir "charge" (biriktirme) yeteneği olup olmadığını kontrol eder (örn: `SKILL_TANHWAN`).
        *   `SetPointVar(const std::string& strName, double dVar)`: `kPointPoly`, `kPointPoly2`, `kPointPoly3`, `kMasterBonusPoly` polinomlarına bir değişken ve değerini atar.
        *   `SetDurationVar(const std::string& strName, double dVar)`: `kDurationPoly`, `kDurationPoly2`, `kDurationPoly3` polinomlarına bir değişken ve değerini atar.
        *   `SetSPCostVar(const std::string& strName, double dVar)`: `kSPCostPoly`, `kGrandMasterAddSPCostPoly` polinomlarına bir değişken ve değerini atar.
*   **`SkillProtoMap` Typedef'i:** `std::map<DWORD, CSkillProto*>` için bir takma ad. Yetenek VNUM'unu `CSkillProto` işaretçisine eşler.
*   **`CSkillManager` Sınıfı (Singleton):**
    *   **Amacı:** Oyundaki tüm `CSkillProto` örneklerini yükler, saklar ve erişim sağlar.
    *   **Metotlar:**
        *   `Initialize(TSkillTable* pTab, int iSize)`: Veritabanından veya bir dosyadan okunan `TSkillTable` dizisini kullanarak tüm yetenek prototiplerini oluşturur ve `m_map_pkSkillProto`'ya ekler.
        *   `Get(DWORD dwVnum)`: VNUM ile bir yetenek prototipi bulur.
        *   `Get(const char* c_pszSkillName)`: Yetenek adıyla bir yetenek prototipi bulur.
    *   **Üyeler:**
        *   `m_vec_bonus_skill_damage_point_match` (`std::vector<SBonusSkillDamagePointMatch>`, `__ATTR_6TH_7TH__` ile): 6. ve 7. efsun yetenek hasarı bonus eşleşmelerini tutar.
        *   `m_map_pkSkillProto` (`SkillProtoMap`): Yüklenen tüm yetenek prototiplerini saklar.

**Bağlantılı Dosyalar:** `../../libpoly/Poly.h` (`CPoly` sınıfı için), `constants.h` (muhtemelen `TSkillTable` yapısı için).

---

### `skill.cpp`

**Amacı:** Bu dosya, `skill.h` içinde tanımlanan `CSkillProto` ve `CSkillManager` sınıflarının metotlarını uygular. Yetenek prototiplerinin oluşturulması, yeteneklerin özelliklerini belirleyen polinomlara değişken atanması ve yetenek yöneticisinin başlatılması gibi işlevleri yerine getirir.

**Orta Seviye Implementasyon Detayları:**

*   **`CSkillProto` Metotları:**
    *   `SetPointVar(const std::string& strName, double dVar)`: Belirtilen değişken adını (`strName`) ve değerini (`dVar`), `kPointPoly`, `kPointPoly2`, `kPointPoly3` ve `kMasterBonusPoly` adlı `CPoly` nesnelerine atar. Bu polinomlar, yeteneğin seviyesine göre etki değerini hesaplamak için kullanılır.
    *   `SetDurationVar(const std::string& strName, double dVar)`: Benzer şekilde, `kDurationPoly`, `kDurationPoly2` ve `kDurationPoly3` (etki süresi polinomları) için değişkenleri ayarlar.
    *   `SetSPCostVar(const std::string& strName, double dVar)`: Benzer şekilde, `kSPCostPoly` ve `kGrandMasterAddSPCostPoly` (SP maliyeti polinomları) için değişkenleri ayarlar.
*   **`CSkillManager` Yapıcı (`CSkillManager()`)**: Eğer `__ATTR_6TH_7TH__` makrosu tanımlıysa, `m_vec_bonus_skill_damage_point_match` vektörünü sabit yetenek VNUM'ları ve bunlara karşılık gelen POINT (karakter puanı) VNUM'ları ile doldurur. Bu, belirli yeteneklerin hasarını artıran 6. ve 7. efsunların eşleştirilmesi için kullanılır.
*   **`CSkillManager` Yıkıcı (`~CSkillManager()`)**: `m_map_pkSkillProto` haritasındaki tüm `CSkillProto` nesnelerini siler (`M2_DELETE`).
*   **`FindPointType(const char* c_sz)` Global Fonksiyonu:**
    *   `skill.cpp` içinde tanımlı `kPointOnTypes` adlı bir global struct dizisi üzerinde arama yapar.
    *   Bu dizi, yeteneklerin etki edebileceği karakter özelliklerinin (POINT'lar) string isimlerini (örn: "MAX_HP", "ATT_SPEED") sayısal `POINT_` enum değerleriyle eşleştirir.
    *   Verilen string (`c_sz`) ile eşleşen bir isim bulursa, karşılık gelen sayısal `POINT_` değerini döndürür. Bulamazsa -1 döndürür.
*   **`CSkillManager::Initialize(TSkillTable* pTab, int iSize)`:**
    *   Bu, yetenek yöneticisinin ana başlatma fonksiyonudur. Veritabanından yüklenen `TSkillTable` dizisini alır.
    *   Her bir `TSkillTable` öğesi için yeni bir `CSkillProto` nesnesi oluşturur.
    *   `CSkillProto`'nun üyelerini (`dwVnum`, `szName`, `dwType`, `bMaxLevel`, `dwFlag`, `dwAffectFlag` vb.) `TSkillTable`'daki karşılık gelen değerlerle doldurur.
    *   Yetenek etkilerinin (`szPointOn`, `szPointOn2`, `szPointOn3`) string isimlerini `FindPointType` ile sayısal `POINT_` değerlerine çevirir ve `pkProto->bPointOn`, `pkProto->wPointOn2`, `pkProto->wPointOn3` üyelerine atar. Eğer bir `PointOn` stringi boşsa veya bulunamazsa (ve boş değilse) hata loglanır.
    *   Yetenek değerlerini, sürelerini, SP maliyetlerini, bekleme sürelerini vb. hesaplayan tüm polinom stringlerini (`szPointPoly`, `szDurationPoly`, `szSPCostPoly` vb.) `CPoly::Analyze()` metodu ile ayrıştırır. Herhangi bir polinomda sözdizimi hatası varsa, hata loglanır ve `Initialize` başarısız olur.
    *   Başarıyla oluşturulan her `CSkillProto` nesnesini geçici bir `map_pkSkillProto`'ya ekler.
    *   Eğer yükleme sırasında herhangi bir hata oluşmamışsa (`!bError`):
        *   Mevcut `m_map_pkSkillProto`'daki tüm eski `CSkillProto` nesnelerini siler ve haritayı temizler.
        *   Geçici `map_pkSkillProto`'daki yeni oluşturulmuş `CSkillProto` nesnelerini asıl `m_map_pkSkillProto`'ya taşır.
        *   Başarılı bir yeniden yükleme mesajı loglar.
    *   Hata oluşmuşsa, hata mesajı loglar.
    *   Sonuç olarak `!bError` (hata yoksa true) döndürür.
*   **`CSkillManager::Get(DWORD dwVnum)`:**
    *   `m_map_pkSkillProto` haritasında verilen VNUM'a sahip `CSkillProto`'yu arar ve bulursa işaretçisini, bulamazsa `NULL` döndürür.
*   **`CSkillManager::Get(const char* c_pszSkillName)`:**
    *   `m_map_pkSkillProto` haritasındaki tüm `CSkillProto`'lar üzerinde döner ve verilen `c_pszSkillName` ile (büyük/küçük harf duyarsız) eşleşen ilk yeteneği bulursa işaretçisini, bulamazsa `NULL` döndürür.

**Bağlantılı Dosyalar:** `stdafx.h`, `../../common/stl.h`, `constants.h`, `skill.h`, `char.h`. (Not: `char.h` muhtemelen `POINT_` enumları veya `CPoly` sınıfının kullandığı karakterle ilgili değişkenler için dahil edilmiş olabilir.)

---

### `start_position.h`

*   **Amaç:** Karakterlerin oyuna başlama pozisyonları, haritaları, krallık isimleri ve karakter oluşturma/arena dönüş pozisyonları gibi temel verileri tanımlayan global dizileri ve bu verilere erişim sağlayan inline fonksiyonları bildirir.
*   **Temel İşlevler/İçerik:**
    *   **Harici (extern) Global Diziler:**
        *   `g_nation_name[4][32]` (char): Krallık isimlerini tutar (0: boş, 1: Shinsoo, 2: Chunjo, 3: Jinno).
        *   `g_start_position[4][2]` (DWORD): Her krallığın (0: ayrılmış, 1-3: krallıklar) oyuna başlama X ve Y koordinatlarını tutar.
        *   `g_start_map[4]` (long): Her krallığın oyuna başlama harita indeksini tutar.
        *   `g_create_position[4][2]` (DWORD): Her krallık için karakter oluşturma ekranından sonraki başlangıç X ve Y koordinatlarını tutar.
        *   `arena_return_position[4][2]` (DWORD): Her krallık için arena veya benzeri özel haritalardan dönüş pozisyonlarının X ve Y koordinatlarını tutar.
        *   `g_wolfman_create_position[4][2]` (DWORD): Her krallık için Kurt Adam (Wolfman) karakterlerinin oluşturma sonrası başlangıç X ve Y koordinatlarını tutar.
    *   **Inline Erişim Fonksiyonları:**
        *   `EMPIRE_NAME(BYTE e)`: Verilen krallık ID'sine (`e`) karşılık gelen krallık adını `LC_TEXT` (yerelleştirme fonksiyonu) ile döndürür.
        *   `EMPIRE_START_MAP(BYTE e)`: Verilen krallık ID'si için başlangıç harita indeksini döndürür.
        *   `EMPIRE_START_X(BYTE e)`: Verilen krallık ID'si için başlangıç X koordinatını döndürür.
        *   `EMPIRE_START_Y(BYTE e)`: Verilen krallık ID'si için başlangıç Y koordinatını döndürür.
        *   `ARENA_RETURN_POINT_X(BYTE e)`: Verilen krallık ID'si için arena dönüş X koordinatını döndürür.
        *   `ARENA_RETURN_POINT_Y(BYTE e)`: Verilen krallık ID'si için arena dönüş Y koordinatını döndürür.
        *   `CREATE_START_X(BYTE e, BYTE j)`: Verilen krallık ID'si (`e`) ve iş (job) ID'si (`j`) için karakter oluşturma sonrası başlangıç X koordinatını döndürür. Eğer iş `JOB_WOLFMAN` ise `g_wolfman_create_position` kullanılır, aksi halde `g_create_position` kullanılır.
        *   `CREATE_START_Y(BYTE e, BYTE j)`: `CREATE_START_X`'e benzer şekilde Y koordinatını döndürür.
*   **Bağlantılı Dosyalar:** `locale_service.h` (`LC_TEXT` için), `start_position.cpp` (global dizilerin tanımları için).

---

### `start_position.cpp`

*   **Amaç:** `start_position.h` dosyasında bildirilen ve karakter başlangıç pozisyonları, haritaları, krallık isimleri gibi verileri içeren global dizileri tanımlar ve başlatır.
*   **Orta Seviye Implementasyon Detayları:**
    *   **`g_nation_name[4][32]` Dizisi:**
        *   Krallık isimlerini içerir. Örnek:
            *   `""` (Boş)
            *   `"신수국"` (Shinsoo)
            *   `"천조국"` (Chunjo)
            *   `"진노국"` (Jinno)
        *   Ayrıca yerelleştirme için `LC_TEXT` ile kullanılabilecek yorum satırları bulunur.
    *   **`g_start_map[4]` Dizisi:**
        *   Krallıkların başlangıç harita indekslerini tanımlar. Örnek:
            *   `0` (Ayrılmış)
            *   `1` (Shinsoo)
            *   `21` (Chunjo)
            *   `41` (Jinno)
    *   **`g_start_position[4][2]` Dizisi:**
        *   Her krallığın varsayılan başlangıç X ve Y koordinatlarını tanımlar.
    *   **`arena_return_position[4][2]` Dizisi:**
        *   Her krallık için arena veya benzeri özel haritalardan dönüldüğünde kullanılacak X ve Y koordinatlarını tanımlar.
    *   **`g_create_position[4][2]` Dizisi:**
        *   Her krallık için karakter oluşturulduktan sonraki varsayılan başlangıç X ve Y koordinatlarını tanımlar.
    *   **`g_create_position_canada[4][2]` Dizisi:**
        *   Muhtemelen belirli bir sunucu bölgesi (örn. Kanada) veya özel bir etkinlik için alternatif karakter oluşturma başlangıç pozisyonlarını tanımlar. Mevcut `start_position.h` dosyasında bu dizi için bir erişim fonksiyonu tanımlanmamıştır, bu nedenle doğrudan kullanılmıyor olabilir veya özel bir kod bloğu tarafından erişiliyor olabilir.
    *   **`g_wolfman_create_position[4][2]` Dizisi:**
        *   Her krallık için Kurt Adam (Lycan) karakterlerinin oluşturulduktan sonraki başlangıç X ve Y koordinatlarını tanımlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `start_position.h`.

---

### `stdafx.h`

*   **Amaç:** Bu dosya, `game` sunucusu projesi için bir ön derlenmiş başlık (precompiled header) dosyasıdır. Sık kullanılan ve nadiren değişen başlık dosyalarını içererek derleme sürelerini kısaltmayı hedefler. Aynı zamanda proje genelinde kullanılacak temel tanımlamaları, makroları ve diğer kütüphane başlıklarını merkezi bir noktada toplar.
*   **Temel İşlevler/İçerik:**
    *   **Proje Özellik Makroları (Yorumlanmış):**
        *   `//#define M2_USE_POOL`: Belirli nesne türleri için bellek havuzu kullanımını etkinleştirmek/devre dışı bırakmak için (genellikle `pool.h` ile ilgilidir).
        *   `//#define DEBUG_ALLOC`: Yığın ayırma (heap allocation) hata ayıklamasını etkinleştirmek/devre dışı bırakmak için (genellikle `debug_allocator.h` ile ilgilidir).
    *   **Dahil Edilen Temel Başlıklar:**
        *   `debug_allocator.h`: Bellek ayırma hatalarını ayıklamak için özel bir ayırıcı.
        *   `../../libthecore/include/stdafx.h`: `libthecore` kütüphanesinin kendi ön derlenmiş başlığı. Temel ağ ve sistem işlevlerini sağlar.
        *   `../../common/singleton.h`: Singleton (tek nesne) tasarım desenini uygulamak için bir şablon.
        *   `../../common/utils.h`: Çeşitli yardımcı fonksiyonlar ve makrolar (örn: string işlemleri, zaman fonksiyonları).
        *   `../../common/service.h`: Muhtemelen harici servislerle (örn: veritabanı, loglama) ilgili tanımlamalar.
    *   **Standart C++ Kütüphane Başlıkları:**
        *   `<algorithm>`: Algoritmalar (örn: `std::sort`, `std::find`).
        *   `<math.h>`: Matematiksel fonksiyonlar (örn: `sqrt`, `sin`, `cos`).
        *   `<list>`, `<map>`, `<set>`, `<queue>`, `<string>`, `<vector>`: Standart şablon kütüphanesi (STL) konteynerleri.
        *   `<ctime>`: Zaman ve tarih fonksiyonları.
        *   `<random>`: Rastgele sayı üretimi.
        *   `<float.h>`: Kayan nokta türleri için limitler ve özellikler.
    *   **Boost Kütüphanesi Başlıkları:**
        *   `<boost/unordered_map.hpp>`: Hızlı karma tablo tabanlı harita.
        *   `<boost/unordered_set.hpp>`: Hızlı karma tablo tabanlı küme.
        *   `#define TR1_NS boost`: Boost kütüphanesini `std::tr1` yerine kullanmak için bir isim alanı takma adı.
    *   **Karakter Fonksiyonları için Yeniden Tanımlama:**
        *   `#define isdigit iswdigit`: Geniş karakterler için rakam kontrolü.
        *   `#define isspace iswspace`: Geniş karakterler için boşluk kontrolü.
    *   **Proje İçi Başlıklar:**
        *   `typedef.h`: Proje genelinde kullanılan özel tür tanımlamaları (örn: `DWORD`, `BYTE`, `LPCHARACTER`).
        *   `locale.hpp`: Yerelleştirme (localization) ile ilgili fonksiyonlar ve tanımlar (örn: `LC_TEXT`).
        *   `event.h`: Zamanlayıcı olayları (event scheduling) sistemi için başlık.
    *   **Yardımcı Makrolar:**
        *   `#define PASSES_PER_SEC(sec) ((sec) * passes_per_sec)`: Saniye cinsinden bir süreyi, sunucunun saniyedeki geçiş (pass) sayısına göre dönüştürür. `passes_per_sec` genellikle sunucunun ana döngü sıklığıyla ilgilidir.
        *   `M_PI`, `M_PI_2`: Pi sayısı ve Pi/2 sabitleri (eğer tanımlı değilse).
        *   `IN`, `OUT`: Fonksiyon parametrelerinin girdi mi çıktı mı olduğunu belirtmek için kullanılan anlamsal makrolar (kodun okunabilirliğini artırır, derleyiciye etkisi yoktur).
        *   `itertype(v)`: Bir konteynerin (`v`) iteratör türünü almak için kullanılan platforma bağlı bir makro. GCC için `__typeof__((v).begin())`, diğerleri için `typeof((v).begin())` kullanır.
*   **Not:** Linter hataları, `unistd.h` (genellikle Unix benzeri sistemlerde bulunur) ve Boost kütüphane dosyalarının bulunamadığını göstermektedir. Bu, geliştirme ortamının doğru yapılandırılmadığına veya bu dosyaların projeye dahil edilmediğine işaret edebilir. Ancak bu durum, `stdafx.h` dosyasının amacını ve genel içeriğini belgelememizi engellemez.
*   **Bağlantılı Dosyalar:** Bu dosya, `game` sunucusundaki neredeyse tüm `.cpp` dosyaları tarafından ilk dahil edilen başlık dosyasıdır. Diğer tüm başlıklar genellikle bu dosya üzerinden dolaylı olarak dahil edilir.

### `typedef.h`

*   **Amaç:** Oyun sunucusunda sıkça kullanılan temel sınıflar için işaretçi takma adlarını (typedef) ve bazı temel koleksiyon türlerini tanımlar. Kod okunabilirliğini artırmak ve `USE_DEBUG_PTR` makrosu aracılığıyla hata ayıklama işaretçileri (`DebugPtr`) ile normal işaretçiler arasında geçiş yapmayı kolaylaştırmak için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **İşaretçi Takma Adları (typedefs):**
        *   `LPDESC`: `DESC` (Descriptor) sınıfı için işaretçi.
        *   `LPCLIENT_DESC`: `CLIENT_DESC` sınıfı için işaretçi.
        *   `LPDESC_P2P`: `DESC_P2P` (Peer-to-Peer Descriptor) sınıfı için işaretçi.
        *   `LPCHARACTER`: `CHARACTER` sınıfı için işaretçi.
        *   `LPITEM`: `CItem` sınıfı için işaretçi.
        *   `building::LPOBJECT`: `building::CObject` (yapı nesnesi) sınıfı için işaretçi.
        *   `LPREGEN`: `regen` struct'ı için işaretçi.
        *   `LPREGEN_EXCEPTION`: `regen_exception` struct'ı için işaretçi.
        *   `LPENTITY`: `CEntity` sınıfı için işaretçi.
        *   `LPSECTREE`: `SECTREE` sınıfı için işaretçi.
        *   `LPSECTREE_MAP`: `SECTREE_MAP` sınıfı için işaretçi.
        *   `LPDUNGEON`: `CDungeon` sınıfı için işaretçi.
        *   `LPPARTY`: `CParty` sınıfı için işaretçi.
        *   `LPPRIVATE_SHOP`: `CPrivateShop` sınıfı için işaretçi.
        *   **Not:** Tüm bu işaretçi takma adları, `#ifdef USE_DEBUG_PTR` koşuluna bağlı olarak ya `DebugPtr<ClassName>` (hata ayıklama için sarmalayıcı bir akıllı işaretçi) ya da doğrudan `ClassName*` (normal C++ işaretçisi) olarak tanımlanır.
    *   **Koleksiyon Türleri (typedefs):**
        *   `CHARACTER_VECTOR`: `std::vector<LPCHARACTER>`.
        *   `CHARACTER_LIST`: `std::list<LPCHARACTER>`.
        *   `CHARACTER_SET`: `TR1_NS::unordered_set<LPCHARACTER>` (C++11 öncesi için `std::tr1::unordered_set` veya Boost gibi bir kütüphaneden gelen `unordered_set`).
        *   `ENTITY_VECTOR`: `std::vector<LPENTITY>`.
        *   `ENTITY_SET`: `TR1_NS::unordered_set<LPENTITY>`.
        *   `LPSECTREE_LIST`: `std::list<LPSECTREE>`.
    *   **`PIXEL_POSITION` Struct'ı:**
        *   3D bir konumu (x, y, z koordinatları) temsil eder.
        *   Üyeler: `INT x, y, z;`
    *   **`EEntityTypes` Enum'u:**
        *   Oyun dünyasındaki temel varlık türlerini tanımlar.
        *   Değerler: `ENTITY_CHARACTER`, `ENTITY_ITEM`, `ENTITY_OBJECT`.
        *   `#if defined(__PREMIUM_PRIVATE_SHOP__)` koşuluna bağlı olarak `ENTITY_PRIVATE_SHOP` da eklenebilir.
*   **Linter Hataları ve Olası Nedenleri:**
    *   `qualified name is not allowed` ve `expected a ';'` : Bu hatalar genellikle `std::vector`, `std::list`, `std::unordered_set` (veya `TR1_NS::unordered_set`) gibi STL koleksiyonlarının tanımlandığı satırlarda görülür. Muhtemel nedenler:
        *   İlgili başlık dosyalarının (`<vector>`, `<list>`, `<unordered_set>`) eksik olması.
        *   `std::` veya `TR1_NS::` isim alanının doğru şekilde kullanılmaması veya bu isim alanlarının tanımlı olmaması.
    *   `name followed by '::' must be a class or namespace name`: `TR1_NS::unordered_set` satırlarında görülür. `TR1_NS` makrosunun (veya isim alanının) doğru tanımlanmamış veya derleyici tarafından tanınmıyor olması olasıdır. Bu, genellikle C++11 öncesi standartlarda `unordered_set` kullanmaya çalışırken veya belirli bir TR1 implementasyonuna (örn: Boost) bağımlılık olduğunda ortaya çıkar.
    *   `identifier "INT" is undefined`: `PIXEL_POSITION` struct'ındaki `INT x, y, z;` satırında görülür. `INT` türünün standart C++'da olmadığını, genellikle Windows programlamada (`<windows.h>` içinde `typedef int INT;` olarak) veya projenin kendi özel tür tanımlarında (`typedef int INT;` gibi bir satırla) tanımlandığını gösterir. Bu başlık dosyasında veya dahil ettiği dosyalarda bu tanım eksik olabilir.
*   **Bağlantılı Dosyalar:** Bu dosya genellikle `stdafx.h` gibi ön derlenmiş başlıklar aracılığıyla veya doğrudan birçok başka `.h` ve `.cpp` dosyası tarafından dahil edilir, çünkü temel tür tanımlarını sağlar. Linter hatalarını çözmek için `<vector>`, `<list>`, `<unordered_set>` (veya ilgili TR1/Boost başlığı) ve `INT` türünün tanımını içeren başlık (örn: `<windows.h>` veya projenin özel bir tür tanım dosyası) dahil edilmelidir. `DebugPtr` kullanılıyorsa, `debug_ptr.h` veya benzeri bir dosyanın da dahil edilmesi gerekir.

---

### `trigger.cpp`

*   **Amaç:** Karakterler (özellikle NPC'ler) için tıklama (`OnClick`) gibi olaylara yanıt veren tetikleyici (trigger) mekanizmalarını ve bazı temel yapay zeka (AI) davranışlarını uygular. Oyuncuların NPC'lerle etkileşime girmesini (konuşma, dükkan açma) ve canavarların basit hedef bulma ve boşta durma davranışlarını yönetir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Tetikleyici Fonksiyonları ve Yapıları:**
        *   `OnClickShop(TRIGGERPARAM)`: Oyuncu bir NPC'ye tıkladığında ve NPC'nin tıklama türü `ON_CLICK_SHOP` ise çağrılır. `CShopManager::instance().StartShopping()` fonksiyonunu çağırarak oyuncu için dükkan arayüzünü başlatır.
        *   `OnClickTalk(TRIGGERPARAM)`: (Bu dosyada implementasyonu yok, muhtemelen başka bir yerde veya quest sistemiyle entegre.)
        *   `OnIdleDefault(TRIGGERPARAM)`: Bir karakter boşta kaldığında çağrılır. Karakterin `OnIdle()` metodunu çağırır. Genellikle saniyede bir kez tetiklenir (`PASSES_PER_SEC(1)`).
        *   `OnAttackDefault(TRIGGERPARAM)`: (Bu dosyada implementasyonu yok, muhtemelen `battle.cpp` veya karakterin kendi saldırı AI'sı içinde ele alınıyor.)
        *   `TTriggerFunction` Struct: Bir tetikleyici fonksiyon işaretçisini (`int (*func) (TRIGGERPARAM)`) tutar.
        *   `OnClickTriggers[ON_CLICK_MAX_NUM]` Dizisi: `ON_CLICK_TYPE` enum değerlerine karşılık gelen `TTriggerFunction` yapılarını tutar. Bu dizi, bir NPC'nin `bOnClickType` değerine göre hangi fonksiyonun çağrılacağını belirler.
    *   **`CHARACTER::AssignTriggers(const TMobTable* table)` Metodu:**
        *   Bir karakter oluşturulduğunda veya yüklendiğinde, `TMobTable` (canavar prototipi) içindeki `bOnClickType` değerine göre karaktere uygun `OnClick` tetikleyici fonksiyonunu atar. Geçersiz bir `bOnClickType` varsa hata verir ve programı sonlandırır (`abort()`).
    *   **Hedef Bulma Mekanizması (`FindVictim`, `FuncFindMobVictim`):**
        *   `FuncFindMobVictim` Sınıfı (Functor):
            *   Bir karakterin (`m_pkChr`) etrafındaki (`m_iMaxDistance` mesafesindeki) en uygun saldırı hedefini bulmak için kullanılır. `SECTREE::ForEachAround` ile birlikte kullanılır.
            *   Yapıcı: Saldıracak karakteri, maksimum arama mesafesini ve başlangıç konumunu alır.
            *   `operator()`: `ForEachAround` tarafından bulunan her bir varlık (`LPENTITY`) için çağrılır.
                *   Varlık bir karakter değilse atlar.
                *   Hedef NPC ise ve saldıran karakter bir canavar değilse veya agresif değilse atlar.
                *   Hedef ölü veya görünmez ise atlar.
                *   Hedef "TERÖR" etkisindeyse ve saldıran karakter teröre bağışık değilse, seviye kontrolü yapar (düşük seviyeli hedefi atlar).
                *   Saldıran karakterin belirli krallıklara saldırmama bayrakları (`IsNoAttackShinsu` vb.) varsa, o krallıktaki hedefleri atlar.
                *   Mevcut en yakın hedeften daha yakın ve maksimum mesafe içindeyse, `m_pkChrVictim` olarak ayarlar.
                *   Eğer varlık inşa halinde bir bina ise (`IsBuilding()` ve ilgili affect flag'leri), `m_pkChrBuilding` olarak ayarlar.
            *   `GetVictim()`: En uygun hedefi döndürür. Önceliklendirme:
                *   Eğer bir bina bulunduysa ve saldıran karakterin canı yarıdan fazlaysa VEYA başka bir oyuncu/canavar hedefi bulunamadıysa, binayı hedef alır.
                *   Aksi takdirde, bulunan en yakın oyuncu/canavar hedefini döndürür.
        *   `FindVictim(LPCHARACTER pkChr, int iMaxDistance)` Fonksiyonu:
            *   `FuncFindMobVictim` nesnesi oluşturur.
            *   Karakterin bulunduğu `SECTREE` üzerinde `ForEachAround` ile `FuncFindMobVictim` functor'ını çalıştırır.
            *   Functor'ın bulduğu en uygun hedefi döndürür.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `char.h`, `sectree_manager.h`, `battle.h`, `affect.h`, `shop_manager.h`.

---