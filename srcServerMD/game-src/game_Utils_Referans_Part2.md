**Not:** Bu belge, [`game_Utils_Referans.md`](game_Utils_Referans.md) dosyasının devamı niteliğindedir.

### `input_login.cpp`

*   **Amaç:** İstemciden gelen giriş (login) ve karakter yönetimi (seçme, oluşturma, silme, oyuna girme) ile ilgili ağ paketlerini işleyen `CInputLogin` sınıfının metotlarını uygular. Aynı zamanda lonca sembol/amblemi yükleme ve istemci versiyon kontrolü gibi işlemleri de yönetir. Bu dosya, `CInputProcessor` sınıfından türeyen `CInputLogin` sınıfı aracılığıyla istemci-sunucu iletişiminin giriş aşamasını yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`CInputLogin` Sınıf Metotları:**
        *   **`Login(LPDESC d, const char* data)`:** Kullanıcı adı ve şifre ile normal giriş isteğini işler (`HEADER_CG_LOGIN`). Sunucu durumu (kapanış, kullanıcı limiti) kontrolü yapar ve DB process'ine `HEADER_GD_LOGIN` paketi gönderir.
        *   **`LoginByKey(LPDESC d, const char* data)`:** Login key (muhtemelen Auth sunucusundan alınan bir oturum anahtarı) ile giriş isteğini işler (`HEADER_CG_LOGIN2`). IP engelleme kontrolü, sunucu durumu kontrolü yapar, `LPDESC` üzerinde login key ve güvenlik anahtarını ayarlar ve DB process'ine `HEADER_GD_LOGIN_BY_KEY` paketi gönderir.
        *   **`CharacterSelect(LPDESC d, const char* data)`:** Oyuncunun karakter seçimi isteğini işler (`HEADER_CG_CHARACTER_SELECT`). Hesap bilgilerini ve seçilen karakterin geçerliliğini kontrol eder (isim değiştirme gerekliliği vb.). DB process'ine `HEADER_GD_PLAYER_LOAD` paketi göndererek karakterin yüklenmesini talep eder.
        *   **`CharacterCreate(LPDESC d, const char* data)`:** Yeni karakter oluşturma isteğini işler (`HEADER_CG_CHARACTER_CREATE`). Karakter ismi ve görünüm geçerliliğini kontrol eder. `NewPlayerTable2` yardımcı fonksiyonunu kullanarak yeni karakterin temel bilgilerini (`TPlayerTable`) doldurur ve DB process'ine `HEADER_GD_PLAYER_CREATE` paketi gönderir.
        *   **`CharacterDelete(LPDESC d, const char* data)`:** Karakter silme isteğini işler (`HEADER_CG_CHARACTER_DELETE`). Hesap bilgilerini, seçilen karakteri ve sosyal güvenlik kodunu (private code) kontrol eder. DB process'ine `HEADER_GD_PLAYER_DELETE` paketi gönderir.
        *   **`ChangeName(LPDESC d, const char* data)`:** Karakter ismi değiştirme isteğini işler (`HEADER_CG_CHANGE_NAME`). Hesabın ve karakterin durumu (isim değiştirme hakkı) kontrol edilir, yeni ismin geçerliliği `check_name` ile doğrulanır ve DB process'ine `HEADER_GD_CHANGE_NAME` paketi gönderilir.
        *   **`Entergame(LPDESC d, const char* data)`:** Oyuncunun seçtiği karakterle oyuna giriş isteğini işler (`HEADER_CG_ENTERGAME`).
            *   Karakterin pozisyonunu doğrular ve gerekirse güvenli bir noktaya taşır.
            *   `CGuildManager::instance().LoginMember(ch)` ile lonca bilgilerini günceller.
            *   Karakteri haritada gösterir (`ch->Show`).
            *   İstemci fazını `PHASE_GAME` olarak ayarlar.
            *   Kostüm seçenekleri, özel duygular gibi özellikler için Quest flag'lerini kontrol eder ve uygular.
            *   Affect'leri yükler (`ch->LoadAffect`).
            *   At, Pet gibi sistemleri başlatır (`ch->EnterHorse`, `ch->EnterPet`).
            *   Çeşitli olayları (Save, Recovery, SpeedHack) başlatır.
            *   `CPVPManager`, `MessengerManager`, `CPartyManager`, `building::CManager`, `marriage::CManager` gibi yöneticilere oyuncunun giriş yaptığını bildirir.
            *   İstemciye zaman, kanal bilgisi, selamlama mesajı ve bonus bilgilerini gönderir.
            *   Premium üyelik affect'lerini uygular.
            *   İstemci versiyonunu kontrol eder (`g_bCheckClientVersion`).
            *   GM ise konsolu aktif eder.
            *   Harita tipine göre (WarMap, WeddingMap, Dungeon, Arena, OXEvent) özel işlemleri gerçekleştirir.
            *   At ismi varsa yükler, yoksa DB'den talep eder.
            *   Savaş bölgesi uyarısı gönderir.
        *   **`Empire(LPDESC d, const char* c_pData)`:** Oyuncunun imparatorluk seçimi isteğini işler (`HEADER_CG_EMPIRE`). Seçilen imparatorluğun geçerliliğini ve hesabın durumunu (daha önce imparatorluk seçilmiş mi, karakter var mı) kontrol eder. DB process'ine `HEADER_GD_EMPIRE_SELECT` paketi gönderir.
        *   **`GuildSymbolUpload(LPDESC d, const char* c_pData, size_t uiBytes)`:** Lonca sembolü yükleme isteğini işler (`HEADER_CG_GUILD_SYMBOL_UPLOAD`). Gelen verinin boyutunu kontrol eder, loncanın arazi sahibi olup olmadığını doğrular (test sunucusu değilse) ve `CGuildMarkManager::instance().UploadSymbol` ile sembolü yükler, ardından `SaveSymbol` ile dosyaya kaydeder.
        *   **`GuildSymbolCRC(LPDESC d, const char* c_pData)`:** İstemciden gelen lonca sembolü CRC'sini sunucudaki ile karşılaştırır (`HEADER_CG_SYMBOL_CRC`). Farklıysa, sembol verisini istemciye (`HEADER_GC_SYMBOL_DATA`) gönderir.
        *   **`GuildMarkUpload(LPDESC d, const char* c_pData)`:** Lonca amblemi yükleme isteğini işler (`HEADER_CG_MARK_UPLOAD`). Loncanın varlığını ve seviyesini (minimum seviye kontrolü) doğrular. `CGuildMarkManager::instance().SaveMark` veya `DeleteMark` ile amblemi kaydeder/siler.
        *   **`GuildMarkIDXList(LPDESC d, const char* c_pData)`:** İstemciye sunucudaki tüm lonca amblemlerinin indeks listesini gönderir (`HEADER_GC_MARK_IDXLIST`).
        *   **`GuildMarkCRCList(LPDESC d, const char* c_pData)`:** İstemciden gelen CRC listesine göre farklı olan amblem bloklarını istemciye gönderir (`HEADER_GC_MARK_BLOCK`).
        *   **`Analyze(LPDESC d, BYTE bHeader, const char* c_pData)`:** `CInputProcessor::Analyze` fonksiyonunun override edilmiş halidir. Gelen paket başlığına (`bHeader`) göre ilgili `CInputLogin` metodunu çağırır. Pong, zaman senkronizasyonu (`Handshake`), istemci versiyonu (`Version`) gibi diğer temel paketleri de işler.
    *   **Yardımcı Fonksiyonlar:**
        *   **`_send_bonus_info(LPCHARACTER ch)`:** Oyuncuya aktif olan EXP, Yang, Eşya Düşürme gibi premium bonusların sohbet mesajı olarak gönderir.
        *   **`FN_is_battle_zone(LPCHARACTER ch)`:** Karakterin bulunduğu haritanın güvenli bölge olup olmadığını kontrol eder. Belirli başlangıç haritaları ve OX haritası güvenli kabul edilir.
        *   **`NewPlayerTable(TPlayerTable* table, ...)` (Eski versiyon):** Verilen parametrelere göre yeni bir karakter için `TPlayerTable` yapısını doldurur. Başlangıç statüleri, HP/SP, pozisyon gibi değerleri ayarlar. `china_event_server` tanımlıysa özel başlangıç seviyesi ve altını ayarlar.
        *   **`RaceToJob(unsigned race, unsigned* ret_job)`:** Verilen ırk (race) numarasına karşılık gelen sınıf (job) numarasını döndürür.
        *   **`NewPlayerTable2(TPlayerTable* table, ...)` (Güncel versiyon):** `NewPlayerTable`'a benzer şekilde, ancak ırk (race) parametresini alıp `RaceToJob` ile sınıfı belirleyerek `TPlayerTable` yapısını doldurur.
*   **Çalışma Prensibi:**
    İstemci giriş aşamasındayken (`PHASE_LOGIN`), sunucuya çeşitli istekler gönderir (login, karakter seçimi, oluşturma vb.). Bu istekler `CInputLogin::Analyze` fonksiyonu tarafından yakalanır ve ilgili işleyici metoda yönlendirilir. Bu metotlar, genellikle gelen veriyi doğrular, sunucu durumunu (yoğunluk, kapanma durumu) kontrol eder, gerekirse diğer oyun sistemleriyle (Lonca Yöneticisi, Karakter Yöneticisi) etkileşime girer ve çoğu durumda isteği DB process'ine ileterek asıl veritabanı işlemlerinin yapılmasını sağlar. DB process'inden gelen yanıtlar `input_db.cpp` tarafından işlenir ve istemcinin bir sonraki aşamaya geçmesi veya hata mesajı alması sağlanır. `Entergame` fonksiyonu, karakterin oyun dünyasına tam olarak dahil olduğu, çeşitli sistemlerin başlatıldığı ve oyuncuya gerekli bilgilerin gönderildiği kritik bir adımdır.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `config.h`, `utils.h`, `input.h` (ve `CInputProcessor`), `desc_client.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `cmd.h`, `buffer_manager.h`, `protocol.h`, `pvp.h`, `start_position.h`, `messenger_manager.h`, `guild_manager.h`, `party.h`, `dungeon.h`, `war_map.h`, `questmanager.h`, `building.h`, `wedding.h`, `affect.h`, `arena.h`, `OXEvent.h`, `priv_manager.h`, `block_country.h`, `log.h`, `horsename_manager.h`, `MarkManager.h`, (varsa) `MeleyLair.h`.

### `input_main.cpp`

*   **Amaç:** Oyuncunun oyuna girdikten sonra gerçekleştirdiği eylemlerle (hareket, saldırı, eşya kullanımı, sohbet, ticaret, pazar, lonca işlemleri vb.) ilgili istemci paketlerini işler. `CInputProcessor` sınıfından türeyen `CInputMain` ve `CInputDead` sınıfları aracılığıyla bu işlemleri yönetir.
*   **Temel İşlevler/İçerik:**
    *   **`CInputMain` Sınıfı:** Aktif oyun oturumundaki paketleri işler.
        *   **Sohbet ve Komut İşleme:**
            *   `Whisper(LPCHARACTER ch, const char* data, size_t uiBytes)`: Özel mesaj (fısıltı) paketlerini işler (`HEADER_CG_WHISPER`). Spam kontrolü, engelleme kontrolü ve P2P yönlendirmesi yapar.
            *   `Chat(LPCHARACTER ch, const char* data, size_t uiBytes)`: Genel, grup, lonca ve bağırma (shout) gibi farklı sohbet türlerini işler (`HEADER_CG_CHAT`). Komutları (`/komut`) `interpret_command` fonksiyonuna yönlendirir. Spam, yasaklı kelime ve metin etiketi (örn: renkli yazı, eşya linki) kontrolleri yapar.
        *   **Eşya Yönetimi:**
            *   `ItemUse(LPCHARACTER ch, const char* data)`: Eşya kullanma isteğini işler (`HEADER_CG_ITEM_USE`).
            *   `ItemToItem(LPCHARACTER ch, const char* pcData)`: Bir eşyayı başka bir eşya üzerinde kullanma isteğini işler (örneğin, efsun nesnesi) (`HEADER_CG_ITEM_USE_TO_ITEM`).
            *   `ItemDrop(LPCHARACTER ch, const char* data)` ve `ItemDrop2(LPCHARACTER ch, const char* data)`: Eşya veya altın/won bırakma isteklerini işler (`HEADER_CG_ITEM_DROP`, `HEADER_CG_ITEM_DROP2`).
            *   `ItemDestroy(LPCHARACTER ch, const char* data)`: Eşya yok etme isteğini işler (eğer `__NEW_DROP_DIALOG__` aktifse) (`HEADER_CG_ITEM_DESTROY`).
            *   `ItemMove(LPCHARACTER ch, const char* data)`: Envanter içinde eşya taşıma isteğini işler (`HEADER_CG_ITEM_MOVE`).
            *   `ItemPickup(LPCHARACTER ch, const char* data)`: Yerden eşya toplama isteğini işler (`HEADER_CG_ITEM_PICKUP`).
            *   `ItemGive(LPCHARACTER ch, const char* c_pData)`: Başka bir karaktere eşya verme isteğini işler (`HEADER_CG_ITEM_GIVE`).
        *   **Karakter Hareket ve Aksiyonları:**
            *   `Move(LPCHARACTER ch, const char* data)`: Karakter hareket paketlerini işler (`HEADER_CG_MOVE`). Hız ve kombo hilesi kontrolleri yapar.
            *   `Attack(LPCHARACTER ch, const BYTE header, const char* data)`: Normal saldırı (`HEADER_CG_ATTACK`) ve menzilli saldırı/beceri (`HEADER_CG_SHOOT`) isteklerini işler. Saldırı mesafesi ve beceri kullanım kontrolleri yapar.
            *   `UseSkill(LPCHARACTER ch, const char* pcData)`: Beceri kullanma isteğini işler (`HEADER_CG_USE_SKILL`).
            *   `Position(LPCHARACTER ch, const char* data)`: Karakterin duruş pozisyonunu (ayakta, oturuyor) değiştirme isteğini işler (`HEADER_CG_CHARACTER_POSITION`).
            *   `SyncPosition(LPCHARACTER ch, const char* c_pcData, size_t uiBytes)`: Diğer oyuncuların pozisyonlarını senkronize etme paketini işler (`HEADER_CG_SYNC_POSITION`). Hile kontrolü içerir.
            *   `OnClick(LPCHARACTER ch, const char* data)`: Bir NPC'ye veya başka bir karaktere tıklama eylemini işler (`HEADER_CG_ON_CLICK`).
            *   `Target(LPCHARACTER ch, const char* pcData)`: Hedef alma isteğini işler (`HEADER_CG_TARGET`).
            *   `Warp(LPCHARACTER ch, const char* pcData)`: Işınlanma tamamlandığında gelen paketi işler (`HEADER_CG_WARP`).
            *   `FlyTarget(LPCHARACTER ch, const char* pcData, BYTE bHeader)`: Uçarak hedefe gitme paketlerini işler (`HEADER_CG_ADD_FLY_TARGETING`, `HEADER_CG_FLY_TARGETING`).
        *   **Ticaret ve Pazar:**
            *   `Exchange(LPCHARACTER ch, const char* data)`: Oyuncular arası ticaret (takas) işlemlerini yönetir (başlatma, eşya/para ekleme/çıkarma, kabul etme, iptal etme) (`HEADER_CG_EXCHANGE`).
            *   `Shop(LPCHARACTER ch, const char* data, size_t uiBytes)`: NPC dükkanlarıyla etkileşimleri (alma, satma) yönetir (`HEADER_CG_SHOP`).
            *   `MyShop(LPCHARACTER ch, const char* c_pData, size_t uiBytes)`: Oyuncu pazarı kurma isteğini işler (`HEADER_CG_MYSHOP`).
            *   `PrivateShopBuild`, `PrivateShopClose`, `PrivateShopPanelOpen`, `PrivateShopPanelClose`, `PrivateShopStart`, `PrivateShopEnd`, `PrivateShopBuy`, `PrivateShopWithdraw`, `PrivateShopModify`, `PrivateShopItemPriceChange`, `PrivateShopItemMove`, `PrivateShopItemCheckin`, `PrivateShopItemCheckout`, `PrivateShopTitleChange`, `PrivateShopWarpRequest`, `PrivateShopSearchClose`, `PrivateShopSearch`, `PrivateShopSearchBuy`: Premium özel pazar sistemi (`__PREMIUM_PRIVATE_SHOP__`) ile ilgili tüm işlemleri yönetir.
        *   **Depo (Safebox) ve Nesne Market Deposu (Mall):**
            *   `SafeboxCheckin(LPCHARACTER ch, const char* c_pData)`: Depoya eşya koyma (`HEADER_CG_SAFEBOX_CHECKIN`).
            *   `SafeboxCheckout(LPCHARACTER ch, const char* c_pData, bool bMall)`: Depodan (`HEADER_CG_SAFEBOX_CHECKOUT`) veya Nesne Market deposundan (`HEADER_CG_MALL_CHECKOUT`) eşya alma.
            *   `SafeboxItemMove(LPCHARACTER ch, const char* data)`: Depo içinde eşya taşıma (`HEADER_CG_SAFEBOX_ITEM_MOVE`).
        *   **Grup (Party) İşlemleri:**
            *   `PartyInvite(LPCHARACTER ch, const char* c_pData)`: Gruba davet etme (`HEADER_CG_PARTY_INVITE`).
            *   `PartyInviteAnswer(LPCHARACTER ch, const char* c_pData)`: Grup davetine cevap verme (`HEADER_CG_PARTY_INVITE_ANSWER`).
            *   `PartyRemove(LPCHARACTER ch, const char* c_pData)`: Gruptan oyuncu çıkarma veya gruptan ayrılma (`HEADER_CG_PARTY_REMOVE`).
            *   `PartySetState(LPCHARACTER ch, const char* c_pData)`: Grup üyesinin rolünü (atakçı, tank vb.) ayarlama (`HEADER_CG_PARTY_SET_STATE`).
            *   `PartyUseSkill(LPCHARACTER ch, const char* c_pData)`: Grup liderinin grup becerisi kullanması (`HEADER_CG_PARTY_USE_SKILL`).
            *   `PartyParameter(LPCHARACTER ch, const char* c_pData)`: Grup eşya dağıtım modunu ayarlama (`HEADER_CG_PARTY_PARAMETER`).
        *   **Lonca (Guild) İşlemleri:**
            *   `AnswerMakeGuild(LPCHARACTER ch, const char* c_pData)`: Lonca kurma isteğine cevap (`HEADER_CG_ANSWER_MAKE_GUILD`).
            *   `Guild(LPCHARACTER ch, const char* data, size_t uiBytes)`: Lonca ile ilgili çeşitli alt işlemleri (para yatırma/çekme, üye ekleme/çıkarma, yetki/rütbe değiştirme, duyuru yapma, beceri kullanma vb.) yönetir (`HEADER_CG_GUILD`).
        *   **Görev (Quest) İşlemleri:**
            *   `ScriptButton(LPCHARACTER ch, const void* c_pData)`: Görev penceresindeki bir butona tıklama (`HEADER_CG_SCRIPT_BUTTON`).
            *   `ScriptAnswer(LPCHARACTER ch, const void* c_pData)`: Görev penceresindeki bir soruya cevap verme (`HEADER_CG_SCRIPT_ANSWER`).
            *   `ScriptSelectItem(LPCHARACTER ch, const void* c_pData)`: Görev için eşya seçme (`HEADER_CG_SCRIPT_SELECT_ITEM`).
            *   `QuestInputString(LPCHARACTER ch, const void* c_pData)`: Görev için metin girişi yapma (`HEADER_CG_QUEST_INPUT_STRING`).
            *   `QuestConfirm(LPCHARACTER ch, const void* c_pData)`: Başka bir oyuncudan gelen görev onay isteğine cevap verme (`HEADER_CG_QUEST_CONFIRM`).
        *   **Hızlı Erişim (Quickslot) Yönetimi:**
            *   `QuickslotAdd(LPCHARACTER ch, const char* data)`, `QuickslotDelete(LPCHARACTER ch, const char* data)`, `QuickslotSwap(LPCHARACTER ch, const char* data)`: Hızlı erişim slotlarını yönetir.
        *   **Arkadaş Listesi (Messenger) İşlemleri:**
            *   `Messenger(LPCHARACTER ch, const char* c_pData, size_t uiBytes)`: Arkadaş ekleme, çıkarma, engelleme gibi işlemleri yönetir (`HEADER_CG_MESSENGER`).
        *   **Balıkçılık:**
            *   `Fishing(LPCHARACTER ch, const char* c_pData)`: Balık tutma eylemini işler (`HEADER_CG_FISHING`).
        *   **Eşya Geliştirme (Refine):**
            *   `Refine(LPCHARACTER ch, const char* c_pData)`: Normal, parşömenli, sadece parayla (Kule demircisi) gibi farklı eşya geliştirme türlerini işler (`HEADER_CG_REFINE`).
            *   `RefineElement(LPCHARACTER ch, const char* c_pData)`: Elementli güçlendirme (`__REFINE_ELEMENT_SYSTEM__`) işlemlerini yönetir (`HEADER_CG_REFINE_ELEMENT`).
        *   **Ejderha Taşı Simyası (Dragon Soul):**
            *   `DragonSoul_RefineWindow_Close`, `DSManager::instance().DoRefineGrade`, `DSManager::instance().DoRefineStep`, `DSManager::instance().DoRefineStrength`, `DSManager::instance().DoChangeAttribute`: Ejderha taşı simyası ile ilgili (geliştirme, arındırma vb.) işlemleri yönetir (`HEADER_CG_DRAGON_SOUL_REFINE`).
        *   **Diğer Sistemler (Preprocessor ile aktifleşenler):**
            *   `LuckyBox`: Şans kutusu sistemi (`__LUCKY_BOX__`).
            *   `Acce`: Kostüm aksesuar sistemi (`__ACCE_COSTUME_SYSTEM__`).
            *   `ChangeLook`: Görünüm değiştirme sistemi (`__CHANGE_LOOK_SYSTEM__`).
            *   `TargetInfoLoad`: Hedef bilgi yükleme (`__SEND_TARGET_INFO__`).
            *   `SkillBookCombination`: Beceri kitabı birleştirme sistemi (`__SKILLBOOK_COMB_SYSTEM__`).
            *   `MailboxWrite`, `MailboxConfirm`, `MailboxProcess`: Posta kutusu sistemi (`__MAILBOX__`).
            *   `Aura`: Aura kostüm sistemi (`__AURA_COSTUME_SYSTEM__`).
            *   `LootFilter`: Ganimet filtresi sistemi (`__LOOT_FILTER_SYSTEM__`).
            *   `GrowthPet`, `GrowthPetHatching`, `GrowthPetLearnSkill`, `GrowthPetSkillUpgrade`, `GrowthPetFeedRequest`, `GrowthPetDeleteSkillRequest`, `GrowthPetNameChangeRequest`, `GrowthPetAttrDetermineRequest`, `GrowthPetAttrChangeRequest`, `GrowthPetReviveRequest`: Geliştirilebilir evcil hayvan sistemi (`__GROWTH_PET_SYSTEM__`).
            *   `Attr67Add`: 6. ve 7. efsun ekleme sistemi (`__ATTR_6TH_7TH__`).
            *   `GemShop`, `ScriptSelectItemEx`: Değerli taş dükkanı sistemi (`__GEM_SYSTEM__`).
            *   `CubeRenewalSend`: Küp yenileme sistemi (`__CUBE_RENEWAL__`).
            *   `InventoryExpansion`: Envanter genişletme sistemi (`__EXTEND_INVEN_SYSTEM__`).
            *   `MoveCostume`: Kostüm efsun aktarma sistemi (`__MOVE_COSTUME_ATTR__`).
            *   `MountUpGrade`: Binek geliştirme sistemi (`__RIDING_EXTENDED__`).
        *   **Genel Paket İşleyicisi:**
            *   `Analyze(LPDESC d, BYTE bHeader, const char* c_pData)`: Gelen paketin başlığına (header) göre ilgili `CInputMain` metodunu çağırır.
    *   **`CInputDead` Sınıfı:** Karakter öldüğünde gelen paketleri işler (genellikle sohbet ve zaman senkronizasyonu gibi temel işlevler).
        *   `Chat`, `Whisper`, `Hack`, `Pong`, `Handshake` metotlarını içerir.
        *   `Analyze(LPDESC d, BYTE bHeader, const char* c_pData)`: Ölü karakter için gelen paketin başlığına göre ilgili `CInputDead` metodunu çağırır.
    *   **Yardımcı Fonksiyonlar:**
        *   `SendBlockChatInfo`: Sohbet engelleme süresini oyuncuya bildirir.
        *   `SpamBlockCheck`: Spam kontrolü yapar ve gerekirse IP bazlı sohbet engellemesi uygular.
        *   `GetTextTag`, `GetTextTagInfo`, `ProcessTextTag`: Sohbet metinlerindeki özel etiketleri (renk, hyperlink) işler ve prizma taşı kontrolü yapar.
        *   `CheckComboHack`: Kombo saldırılarında hile kontrolü yapar.

*   **Çalışma Prensibi:**
    *   Oyuncu istemcisi sunucuya bir eylemle ilgili paket gönderdiğinde, `CInputMain::Analyze` (veya karakter ölü ise `CInputDead::Analyze`) fonksiyonu bu paketi alır.
    *   Paketin başlığına (header) göre, `Analyze` fonksiyonu ilgili işleyici metoda yönlendirir (örneğin, `HEADER_CG_MOVE` ise `Move` metodu çağrılır).
    *   İlgili metot, paketin içeriğini çözer, gerekli kontrolleri (izinler, hileler, koşullar vb.) yapar ve sunucu tarafında ilgili oyun mantığını çalıştırır (örneğin, karakteri hareket ettirir, eşyayı kullandırır, sohbet mesajını diğer oyunculara gönderir).
    *   Birçok fonksiyon, `CHARACTER`, `ITEM_MANAGER`, `SHOP_MANAGER`, `GUILD_MANAGER`, `QUEST_MANAGER` gibi yönetici sınıflar aracılığıyla oyun dünyasıyla etkileşime girer.
    *   Preprocessor direktifleri (`#if defined(...)`) ile birçok özellik opsiyonel olarak derlenir ve sadece tanımlıysa ilgili kod blokları aktif olur. Bu, farklı sunucu yapılandırmalarına olanak tanır.

---

### `input_p2p.cpp`

*   **Amaç:** Sunucular arası (Peer-to-Peer - P2P) iletişimi yönetir. Farklı kanallar veya aynı sunucu kümesindeki diğer bağlı sunuculardan gelen `HEADER_GG_*` önekli paketleri işleyen `CInputP2P` sınıfını içerir.
*   **Temel İşlevler/İçerik:**
    *   **`CInputP2P` Sınıfı:**
        *   **Bağlantı Yönetimi:** `Setup`, `Login`, `Logout` (Diğer sunucuların bağlanması, kimlik doğrulaması ve ayrılması).
        *   **Mesaj Yönlendirme:** `Relay` (Belirli bir oyuncuya paket yönlendirme, örn: fısıltı), `Notice`, `BigNotice`, `MonarchNotice`, `Shout`, `LocaleNotice` (Farklı kapsamlarda duyuru gönderme).
        *   **Lonca İşlemleri:** `Guild` (Lonca sohbeti, üye sayısı bonusu gibi sunucular arası lonca bilgilerini işler).
        *   **Arkadaş Listesi:** `MessengerAdd`, `MessengerRemove`, `MessengerBlockAdd`, `MessengerBlockRemove` (Sunucular arası arkadaş listesi ve engelleme listesi senkronizasyonu).
        *   **Karakter Konum/Işınlama:** `FindPosition`, `WarpCharacter`, `Transfer`, `MonarchTransfer` (Oyuncuları farklı sunucular arasında veya sunucu içinde ışınlama/konum bulma).
        *   **Etkinlik Yönetimi:** `XmasWarpSanta`, `XmasWarpSantaReply` (Noel etkinliği gibi sunucular arası etkinlik yönetimi).
        *   **Sunucu Yönetimi:** `Shutdown`, `LoginPing` (Sunucu kapatma, canlılık kontrolü), `BlockChat` (Oyuncunun sohbetini engelleme).
        *   **Özel Sistemler (Preprocessor ile):**
            *   `PrivateShopItemSearch`, `PrivateShopItemSearchResult`, `PrivateShopItemSearchUpdate`: Premium özel pazar sistemi (`__PREMIUM_PRIVATE_SHOP__`) için sunucular arası ürün arama, sonuç iletme ve durum güncelleme.
        *   **Diğer:** `GuildWarZoneMapIndex` (Lonca savaş alanı bilgisini senkronize etme), `ReloadCRCList`, `CheckClientVersion`, `PCBangUpdate`, `IamAwake` (Çeşitli yönetim ve kontrol işlemleri).
    *   **Yardımcı Fonksiyonlar:** `SendShout`, `SendLCNotice` (Mesajları uygun oyunculara dağıtan fonksiyonlar).
*   **Çalışma Prensibi:**
    *   Bir sunucu (peer), diğerine `HEADER_GG_*` ile başlayan bir paket gönderir.
    *   Paketi alan sunucudaki `CInputP2P::Analyze` metodu, paketin başlığına göre ilgili işleyici fonksiyonu çağırır.
    *   İşleyici fonksiyon, paketin içeriğine göre gerekli işlemi yapar. Bu işlem, `P2P_MANAGER` aracılığıyla diğer sunuculara bilgi yaymak, `CHARACTER_MANAGER` veya `GUILD_MANAGER` gibi yerel yöneticilerle etkileşime girmek veya doğrudan istemcilere (`DESC_MANAGER` aracılığıyla) bilgi göndermek olabilir. Örneğin, `Relay` paketi hedef oyuncuyu bulup ona yönlendirilirken, `Shout` paketi tüm uygun oyunculara gönderilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `config.h`, `desc_client.h`, `desc_manager.h`, `char.h`, `char_manager.h`, `p2p.h`, `guild.h`, `guild_manager.h`, `party.h`, `messenger_manager.h`, `empire_text_convert.h`, `unique_item.h`, `xmas_event.h`, `affect.h`, `castle.h`, `dev_log.h`, `locale_service.h`, `questmanager.h`, `pcbang.h`, `skill.h`, `threeway_war.h`, (varsa) `private_shop_manager.h`, `private_shop.h`, `buffer_manager.h`.

---

### `input_udp.cpp`

*   **Amaç:** UDP (User Datagram Protocol) üzerinden gelen ağ paketlerini işlemek üzere tasarlanmıştır. Genellikle TCP'ye göre daha hızlı ancak daha az güvenilir iletişim gerektiren durumlar için kullanılır.
*   **Temel İşlevler/İçerik:**
    *   **`CInputUDP` Sınıfı:** UDP paketlerini analiz eden ana sınıf.
    *   **`Handshake(LPDESC pDesc, const char* c_pData)` (Potansiyel/Eski):** TCP bağlantısı kurmuş bir istemcinin UDP adresini doğrulamak ve UDP iletişimine izin vermek için kullanılmış olabilir. Gelen paketteki `dwHandshake` değerini, istemcinin `LPDESC`'indeki değerle karşılaştırır.
    *   **`StateChecker(const char* c_pData)` (Potansiyel/Eski):** Sunucunun yoğunluk durumunu (Normal, Yoğun, Dolu) sorgulayan harici bir isteğe UDP üzerinden yanıt vermek için kullanılmış olabilir.
    *   **`Analyze(LPDESC pDesc, BYTE bHeader, const char* c_pData)`:** Gelen UDP paketinin başlığına göre ilgili fonksiyonu çağırması beklenir, ancak mevcut kodda içeriği yorum satırıdır.
*   **Mevcut Durum:** Bu dosyadaki UDP işleme mantığının büyük bir kısmı (özellikle `Process` ve `Analyze` içindeki `case` blokları) yorum satırı haline getirilmiştir. Bu durum, UDP tabanlı bu özelliklerin (Handshake, StateChecker) artık aktif olarak kullanılmadığını veya farklı bir mekanizmayla değiştirildiğini düşündürmektedir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `config.h`, `input.h`, `desc.h`, `desc_manager.h`, `item_manager.h`, `char_manager.h`, `protocol.h`.

--- 

### `input.h`

*   **Amaç:** Ağ paketlerini işleyen temel sınıf (`CInputProcessor`) ve ondan türeyen, farklı bağlantı aşamalarına (phases) özel sınıflar (`CInputHandshake`, `CInputLogin`, `CInputMain`, `CInputDead`, `CInputDB`, `CInputUDP`, `CInputP2P`, `CInputAuth`) için başlık dosyasını tanımlar. Girdi işleyici türlerini (`INPROC_*`) ve bazı yardımcı fonksiyon bildirimlerini (örn. `LoginFailure`, `GetHWIDStatus`) içerir.
*   **Temel İşlevler/İçerik:**
    *   **`INPROC_*` Enum:** Farklı girdi işleyici türlerini tanımlar.
    *   **`CInputProcessor` (Temel Sınıf):**
        *   `Process`: Gelen veri tamponunu işleyen ana sanal fonksiyon. Paketin başlığını okur, boyutunu belirler ve ilgili `Analyze` fonksiyonunu çağırır.
        *   `Analyze`: Belirli bir paketi işlemek için türetilmiş sınıflarda override edilmesi gereken saf sanal fonksiyon.
        *   `GetType`: İşleyici türünü döndüren saf sanal fonksiyon.
        *   `BindPacketInfo`: Kullanılacak paket tanımlama bilgisini (`CPacketInfo`) ayarlar.
        *   `Pong`, `Handshake`, `Version`: Temel paket işleyici metotları (uygulamaları `.cpp` dosyasındadır).
    *   **Türetilmiş Sınıflar (`CInputHandshake`, `CInputLogin` vb.):** Belirli bir bağlantı aşaması (phase) için `CInputProcessor`'dan türer. Kendi `Analyze` metodunu implemente ederek o aşamaya özgü paketleri işlerler. Başlık dosyasında, her sınıfın hangi paketleri işlemek için özel metotlara sahip olduğu (prototipler) listelenir.
    *   **Yardımcı Fonksiyonlar:** `LoginFailure` (istemciye giriş hatası bildirimi gönderir), `GetHWIDStatus` (Donanım ID'sini kontrol eder ve durumunu döndürür).

---

### `input.cpp`

*   **Amaç:** `input.h`'de bildirilen `CInputProcessor` temel sınıfının ve `CInputHandshake` sınıfının bazı metotlarını ve genel yardımcı fonksiyonları uygular.
*   **Temel İşlevler/İçerik:**
    *   **`CInputProcessor` Metotları:**
        *   `CInputProcessor::Process`: Gelen ham veriyi alır, paket başlıklarını (`bHeader`) okur, `CPacketInfo` kullanarak paket boyutunu belirler, `Analyze` sanal fonksiyonunu çağırarak paketin ilgili işleyici tarafından işlenmesini sağlar ve tamponu ilerletir. Paket sekansı kontrolü (`__SEND_SEQUENCE__` tanımlıysa) yapar.
        *   `CInputProcessor::Pong`: İstemciden gelen ping yanıtını (`HEADER_CG_PONG`) işler ve istemcinin hala bağlı olduğunu işaretler.
        *   `CInputProcessor::Handshake`: El sıkışma paketini (`HEADER_CG_HANDSHAKE`) işler, zaman senkronizasyonunu yapar ve başarılıysa istemciyi bir sonraki aşamaya geçirir (Auth veya Login). Şifreleme (`__IMPROVED_PACKET_ENCRYPTION__`) tanımlıysa anahtar anlaşması (`HEADER_CG_KEY_AGREEMENT`) adımını başlatır.
        *   `CInputProcessor::Version`: İstemci versiyon bilgisini (`HEADER_CG_CLIENT_VERSION`) alır ve kaydeder.
        *   `CInputProcessor::Phase`: Eski bir metin tabanlı faz değiştirme komutunu (`HEADER_CG_PHASE`) işler (muhtemelen artık kullanılmıyor).
    *   **`CInputHandshake` Metotları:**
        *   `CInputHandshake::Analyze`: Handshake (`PHASE_HANDSHAKE`) aşamasındayken gelen paketleri işler. Özellikle metin tabanlı komutları (`HEADER_CG_TEXT`) analiz eder (sunucu durumu sorgulama (`IS_SERVER_UP`), admin girişi (`SHOWMETHEMONEY`/`g_stAdminPagePassword`), kullanıcı sayısı (`USER_COUNT`), P2P kontrolü, yeniden yükleme komutları (`RELOAD`), olay bayrağı ayarlama (`EVENT`), sohbet engelleme (`BLOCK_CHAT`), ayrıcalık verme (`PRIV_EMPIRE`), blok istisnası yönetimi (`BLOCK_EXCEPTION`) vb.). Ayrıca Mark sunucusu girişi (`HEADER_CG_MARK_LOGIN`), kanal durumu sorgusu (`HEADER_CG_STATE_CHECKER`), Pong ve Handshake paketlerini de işler. Şifreleme aktifse anahtar anlaşması paketini (`HEADER_CG_KEY_AGREEMENT`) de burada işler.
    *   **Yardımcı Fonksiyonlar:**
        *   `IsAdminPage`, `IsEmptyAdminPage`, `ClearAdminPages`: Web arayüzü veya özel araçlar için yönetici erişimini IP bazında kontrol eden fonksiyonlar.
        *   `LoginFailure`: Giriş başarısız olduğunda istemciye `HEADER_GC_LOGIN_FAILURE` paketini gönderir.
        *   `GetHWIDStatus`: Veritabanından hesap ID'si ve donanım ID'si kullanarak durumu (değişmemiş, değişmiş, engellenmiş) ve karakter slotlarının geçerliliğini sorgular.
*   **Çalışma Prensibi:**
    *   Bir istemci bağlandığında genellikle `PHASE_HANDSHAKE` aşamasındadır ve `CInputHandshake` işleyicisi kullanılır.
    *   `CInputProcessor::Process`, ağdan gelen veriyi tampondan okur.
    *   `m_pPacketInfo` (genellikle `CPacketInfoCG`) kullanarak paketin başlığını ve boyutunu belirler.
    *   `LPDESC`'in o anki aşamasına (`GetInputProcessor()`) karşılık gelen işleyicinin (`CInputHandshake`, `CInputLogin`, `CInputMain` vb.) `Analyze` metodu çağrılır.
    *   `Analyze` metodu, gelen paketin başlığına göre uygun işlemi gerçekleştirir.
    *   `Handshake` işlemi başarıyla tamamlandığında, `LPDESC::SetPhase` ile istemcinin aşaması (ve dolayısıyla girdi işleyicisi) değiştirilir (genellikle `PHASE_LOGIN` veya `PHASE_AUTH`).
*   **Bağlantılı Dosyalar:** `stdafx.h`, `<sstream>`, `desc.h`, `desc_manager.h`, `char.h`, `buffer_manager.h`, `config.h`, `profiler.h`, `p2p.h`, `log.h`, `db.h`, `questmanager.h`, `login_sim.h`, `fishing.h`, `priv_manager.h`, `castle.h`, `dev_log.h`.

--- 

### `ip_ban.h`

*   **Amaç:** IP adresi engelleme işlevselliği için kullanılan harici fonksiyonların bildirimlerini içerir.
*   **Temel İşlevler/İçerik:**
    *   `LoadBanIP(const char* filename)`: Engellenmiş IP adreslerinin listesini bir dosyadan yüklemek için bildirilir.
    *   `IsBanIP(struct in_addr in)`: Verilen bir IP adresinin engellenmiş olup olmadığını kontrol etmek için bildirilir.
*   **Bağlantılı Dosyalar:** `ip_ban.cpp`.

---

### `ip_ban.cpp`

*   **Amaç:** `ip_ban.h`'de bildirilen IP engelleme fonksiyonlarını uygular. Engellenmiş IP adreslerini bir dosyadan yükler, saklar ve belirli bir IP'nin engelli olup olmadığını kontrol eder.
*   **Temel İşlevler/İçerik:**
    *   **`IP` Sınıfı:**
        *   IP adreslerini veya aralıklarını (başlangıç, bitiş, maske) `DWORD` olarak temsil eder.
        *   String veya `struct in_addr`'dan IP nesnesi oluşturabilir.
        *   `IsChildOf`: Bir IP'nin, tanımlanmış bir engelli IP aralığına dahil olup olmadığını maske kullanarak verimli bir şekilde kontrol eder.
        *   `hash`: IP'nin ilk oktetini, hızlı arama için kullanılan map'in anahtarı olarak döndürür.
    *   **Veri Saklama:** Engellenmiş IP aralıkları, ilk oktetlerine göre gruplandırılmış bir map (`mapBanIP <int, std::vector<IP>>`) içinde saklanır.
    *   **`LoadBanIP(const char* filename)`:**
        *   Belirtilen dosyayı açar ve satır satır okur.
        *   Her satırdan IP adresini veya başlangıç/bitiş IP'lerini ayrıştırır.
        *   Her IP/aralık için bir `IP` nesnesi oluşturur.
        *   Oluşturulan `IP` nesnesini, hash değerine (ilk oktet) göre `mapBanIP` haritasına ekler.
    *   **`IsBanIP(struct in_addr in)`:**
        *   Kontrol edilecek IP'nin hash değerini (ilk oktet) hesaplar.
        *   `mapBanIP` haritasında ilgili hash anahtarına sahip IP vektörünü arar.
        *   Eğer vektör bulunursa, vektördeki her bir `IP` aralığı için `IsChildOf` ile kontrol yapar.
        *   IP, herhangi bir engelli aralığa dahilse `true` döner.
*   **Çalışma Prensibi:** Sunucu başlangıcında `LoadBanIP` ile engelli IP listesi dosyadan okunarak bellekteki `mapBanIP` haritasına yüklenir. Yeni bir bağlantı geldiğinde veya kontrol gerektiğinde, `IsBanIP` fonksiyonu çağrılır. Bu fonksiyon, gelen IP'nin ilk oktetini kullanarak potansiyel olarak eşleşen küçük bir IP aralığı alt kümesini (map'teki vektörü) hızlıca bulur ve sadece bu alt küme üzerinde `IsChildOf` kontrolünü yaparak verimlilik sağlar.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `ip_ban.h`.

--- 

### `item_addon.h`

*   **Amaç:** Eşyalara ek bonuslar (addon) uygulamak için kullanılan `CItemAddonManager` sınıfının bildirimini içerir.
*   **Temel İşlevler/İçerik:**
    *   `CItemAddonManager` (Singleton): Eşyalara bonus ekleme işlevini yöneten sınıf.
    *   `ApplyAddonTo(int iAddonType, LPITEM pItem)`: Belirtilen eşyaya bonus eklemek için kullanılan metodun bildirimi.
*   **Bağlantılı Dosyalar:** `item_addon.cpp`.

---

### `item_addon.cpp`

*   **Amaç:** `item_addon.h`'de bildirilen `CItemAddonManager` sınıfının metotlarını uygular.
*   **Temel İşlevler/İçerik:**
    *   **`CItemAddonManager::ApplyAddonTo(int iAddonType, LPITEM pItem)`:**
        *   Belirtilen eşyaya (`pItem`) Gauss dağılımı kullanarak rastgele bir "Beceri Hasarı Bonusu" (`APPLY_SKILL_DAMAGE_BONUS`) hesaplar.
        *   Hesaplanan beceri bonusuna bağlı olarak bir "Normal Vuruş Hasarı Bonusu" (`APPLY_NORMAL_HIT_DAMAGE_BONUS`) hesaplar.
        *   Eşyadaki mevcut aynı türdeki bonusları kaldırır.
        *   Hesaplanan yeni bonusları eşyaya ekler.
        *   (Not: `iAddonType` parametresi mevcut implementasyonda kullanılmamaktadır.)
*   **Çalışma Prensibi:** Bu fonksiyon çağrıldığında, hedef eşya için belirlenen formüllere göre iki tür hasar bonusu rastgele üretilir ve eşyanın niteliklerine atanır. Bu, örneğin belirli eşyaların üretildiğinde veya dönüştürüldüğünde rastgele bonuslar almasını sağlamak için kullanılabilir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `constants.h`, `utils.h`, `item.h`, `item_addon.h`.

---

### `item_apply_random_table.h`

*   **Amaç:** (`__ITEM_APPLY_RANDOM__` tanımlıysa) Eşyalara uygulanacak rastgele temel bonusların konfigürasyonunu yönetmek için `CApplyRandomTable` sınıfını ve ilgili yapıları (`SApplyRandom`) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **`EApplyRandom` Enum:** Rastgele bonus değer yollarını (`APPLY_RANDOM_PATH1`, `APPLY_RANDOM_PATH2`) ve maksimum geliştirme seviyesini (`ITEM_REFINE_MAX_LEVEL`) tanımlar.
    *   **`SApplyRandom` Struct:** Bir bonus türünü (`EApplyTypes`) ve bu bonusun değerlerinin bulunduğu konfigürasyon grubunun adını (`ApplyValueGroupName`) bir arada tutar.
    *   **`CApplyRandomTable` Sınıfı:**
        *   `ReadApplyRandomTableFile`: Konfigürasyon dosyasını yüklemek için bildirilir.
        *   `GetApplyRandom`: Belirli bir indeks ve seviye için rastgele bir bonus türü, değeri ve kullanılan yolu almak için bildirilir.
        *   `GetApplyRandomValue`: Belirli bir indeks, seviye, yol ve bonus türü için spesifik değeri almak için bildirilir.
        *   `ApplyRandomVector`, `ApplyRandomGroupMap`: Bonus yapılarını ve gruplarını saklamak için kullanılan typedef'ler.
*   **Bağlantılı Dosyalar:** `item_apply_random_table.cpp`.

---

### `item_apply_random_table.cpp`

*   **Amaç:** (`__ITEM_APPLY_RANDOM__` tanımlıysa) `item_apply_random_table.h`'de bildirilen `CApplyRandomTable` sınıfının metotlarını uygular. Rastgele bonus konfigürasyonunu dosyadan okur ve sorgulara yanıt verir.
*   **Temel İşlevler/İçerik:**
    *   **`CApplyRandomTable::ReadApplyRandomTableFile`:**
        *   `CGroupTextParseTreeLoader` kullanarak belirtilen `.txt` dosyasını yükler.
        *   `ReadApplyRandomMapper` ve `ReadApplyRandomTypes` fonksiyonlarını çağırarak konfigürasyonu ayrıştırır.
        *   `applyrandomvalues` grubunu saklar.
    *   **`CApplyRandomTable::ReadApplyRandomMapper`:** Konfigürasyon dosyasındaki `applyrandommapper` grubunu okur ve rastgele bonus gruplarının isimlerini (`m_vecApplyRandomGroups`) saklar.
    *   **`CApplyRandomTable::ReadApplyRandomTypes`:** Konfigürasyon dosyasındaki `applyrandomtypes` grubunu okur. Her bir bonus grubu için olası bonus türlerini (`EApplyTypes`) ve bu türlerin değerlerinin hangi değer grubunda (`apply_value_group_name`) tanımlandığını `m_mapApplyRandomGroup` içinde saklar.
    *   **`CApplyRandomTable::GetApplyRandom`:**
        *   Verilen indekse göre ilgili bonus grubunu `m_mapApplyRandomGroup`'dan bulur.
        *   Bu gruptaki olası bonus türlerinden rastgele birini seçer (`iRandom`).
        *   Seçilen bonus türü için `GetApplyRandomPath` ile rastgele bir değer yolu (`uiPath`) belirler.
        *   `GetApplyRandomRowValue` ile belirlenen yol ve verilen seviye (`uiLevel`) için bonus değerini (`iApplyValue`) alır.
        *   Seçilen bonus türünü (`uiApplyType`), değerini ve yolunu döndürür.
    *   **`CApplyRandomTable::GetApplyRandomValue`:** Belirtilen indeks, seviye, yol ve bonus türü için `GetApplyRandomRowValue` kullanarak doğrudan bonus değerini döndürür.
    *   **`CApplyRandomTable::GetApplyRandomPath`:** Belirtilen değer grubu (`stApplyValueGroupName`) için konfigürasyonda tanımlı olan değer yollarından (satırlardan) rastgele birinin indeksini döndürür.
    *   **`CApplyRandomTable::GetApplyRandomRowValue`:** Belirtilen değer grubu, seviye (sütun) ve yol (satır) için konfigürasyon dosyasındaki kesişen hücredeki bonus değerini okur ve döndürür.
*   **Çalışma Prensibi:** Bu sistem, eşyalara rastgele temel bonuslar eklemek için esnek bir yapı sunar. Konfigürasyon dosyası (`.txt`), hangi eşya türlerinin hangi bonusları alabileceğini, bu bonusların hangi seviyelerde hangi değer aralıklarına (yollarına) sahip olabileceğini tanımlar. `GetApplyRandom` fonksiyonu, bu tanımlara göre rastgele bir seçim yaparak eşyaya eklenecek bonusu ve değerini belirler.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `group_text_parse_tree.h`, `item_apply_random_table.h`, `constants.h`.

--- 

### `item_manager_idrange.cpp`

*   **Amaç:** `ITEM_MANAGER` sınıfının benzersiz eşya ID'leri üretme ve yönetme işlevini uygular. Sunucuya veritabanı tarafından ayrılmış ID aralıklarını kullanarak yeni eşyalar için çakışmayan ID'ler sağlar.
*   **Temel İşlevler/Metotlar:**
    *   `ITEM_MANAGER::GetNewID()`: Yeni bir eşya için kullanılabilir bir sonraki ID'yi döndürür. Mevcut ID (`m_dwCurrentID`), sunucuya ayrılan ana aralığın (`m_ItemIDRange.dwMax`) sonuna ulaştığında, önceden alınmış yedek ID aralığına (`m_ItemIDSpareRange`) geçer. Aynı zamanda veritabanından yeni bir yedek aralık talep eder. Eğer yedek aralık da yoksa veya doluysa, kritik hata günlüğü kaydeder ve sunucuyu kapatır.
    *   `ITEM_MANAGER::SetMaxItemID(TItemIDRangeTable range)`: Veritabanından gelen ana ID aralığı bilgilerini (`dwMin`, `dwMax`, `dwUsableItemIDMin`) alır, `m_ItemIDRange` üyesine atar ve bir sonraki kullanılacak ID'yi (`m_dwCurrentID`) `dwUsableItemIDMin` olarak ayarlar.
    *   `ITEM_MANAGER::SetMaxSpareItemID(TItemIDRangeTable range)`: Veritabanından gelen yedek ID aralığı bilgilerini alır ve `m_ItemIDSpareRange` üyesine atar.
    *   `touch(const char* path)`: Belirtilen yolda bir dosya oluşturur veya son erişim zamanını günceller (Kritik ID hatası durumunda `.killscript` dosyası oluşturmak için kullanılır).
*   **Çalışma Prensibi:** Sunucu, DB sunucusundan kendine özel bir ana (`m_ItemIDRange`) ve bir yedek (`m_ItemIDSpareRange`) ID aralığı alır. Yeni eşya oluşturuldukça `GetNewID` çağrılır ve `m_dwCurrentID` kullanılır/artırılır. Ana aralık dolunca yedek aralığa geçilir ve DB'den yeni yedek istenir. Bu, farklı oyun sunucularının aynı ID'yi üretmesini engeller.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `desc_client.h`, `item_manager.h`.

---

### `item_manager_private_types.h`

*   **Amaç:** `ITEM_MANAGER` sınıfı tarafından dahili olarak kullanılan özel veri yapılarını tanımlar. Özellikle eşya düşürme (drop) mekanizmalarıyla ilgili tanımlamalar içerir.
*   **Temel İşlevler/İçerik:**
    *   **`CItemDropInfo` Sınıfı:**
        *   Bir eşyanın düşme olasılığıyla ilgili bilgileri tutar: Düşebileceği minimum seviye (`m_iLevelStart`), maksimum seviye (`m_iLevelEnd`), düşme oranı (binde bir olarak, `m_iPercent`), ve eşyanın VNUM'u (`m_dwVnum`).
        *   `operator<`: Seviye sonuna göre sıralama için karşılaştırma operatörü tanımlar.
    *   **`g_vec_pkCommonDropItem` Global Değişkeni:**
        *   `MOB_RANK_MAX_NUM` boyutunda bir `std::vector<CItemDropInfo>` dizisi. Yaratık rütbesine göre genel (common) eşya düşürme bilgilerini saklamak için kullanılır.
    *   **`SDropItem` Struct'ı:**
        *   Başka bir eşya düşürme bilgisi yapısı. Seviye aralığı (`iLvStart`, `iLvEnd`), düşme yüzdesi (`fPercent`), eşya adı (`szItemName`) ve düşecek adet (`iCount`) bilgilerini içerir. (Bu yapının spesifik kullanımı bu dosyadan anlaşılamamaktadır, muhtemelen başka dosyalarda veya özel düşürme tablolarında kullanılır.)
*   **Bağlantılı Dosyalar:** Bu dosya genellikle `item_manager.cpp` veya eşya düşürme mantığını içeren diğer `.cpp` dosyaları tarafından dahil edilir.

--- 

### `item_manager_read_tables.cpp`

*   **Amaç:** `ITEM_MANAGER` sınıfının eşya ile ilgili çeşitli konfigürasyon dosyalarını (genellikle `.txt` formatında) okuma ve bu verileri oyun içinde kullanılabilir hale getirme işlevlerini uygular. Bu, oyunun temel eşya veritabanı, düşürme tabloları ve diğer eşya bağlantılı sistemlerin (şans kutusu, set eşyaları vb.) yüklenmesini kapsar.
*   **Temel İşlevler/Metotlar:**
    *   **Düşürme (Drop) Tabloları Okuma:**
        *   `ReadCommonDropItemFile(filename)`: Yaratık rütbesine göre genel eşya düşürme bilgilerini (`common_drop_item.txt` gibi) okur ve `g_vec_pkCommonDropItem` global dizisini doldurur.
        *   `ReadSpecialDropItemFile(filename)`: Özel eşya gruplarını (`special_item_group.txt` gibi) okur. Bu gruplar, belirli bir "grup vnum" altında tanımlanır ve farklı türlerde olabilir (normal, yüzde bazlı, görev, özel, efsun grubu). Okunan veriler `m_map_pkSpecialItemGroup`, `m_map_pkQuestItemGroup`, `m_map_pkSpecialAttrGroup` gibi haritalarda saklanır.
        *   `ReadEtcDropItemFile(filename)`: Diğer çeşitli eşyaların genel düşme olasılıklarını (`etc_drop_item.txt` gibi) okur ve `m_map_dwEtcItemDropProb` haritasına kaydeder.
        *   `ReadMonsterDropItemGroup(filename)`: Belirli yaratıkların (`mob_vnum`) düşürebileceği eşya gruplarını (`mob_drop_item.txt` gibi) okur. Grup türüne göre (`kill`, `drop`, `limit`, `thiefgloves`) farklı gruplama mantıkları (`CMobItemGroup`, `CDropItemGroup`, `CLevelItemGroup`, `CBuyerThiefGlovesItemGroup`) kullanılır ve ilgili haritalarda (`m_map_pkMobItemGroup`, `m_map_pkDropItemGroup` vb.) saklanır.
        *   `ReadDropItemGroup(filename)`: Belirli bir yaratık için (`mob_vnum`) ve bir grup VNUM'u (`vnum`) ile tanımlanan genel düşürme gruplarını (`drop_item.txt` gibi) okur ve `m_map_pkDropItemGroup` haritasını günceller.
    *   **Eşya VNUM Maskeleme:**
        *   `ReadItemVnumMaskTable(filename)`: Bir VNUM'u başka bir VNUM ile değiştirmek için kullanılan bir tabloyu (`item_vnum_mask_table.txt` gibi) okur ve `m_map_new_to_ori` haritasında saklar.
    *   **Sisteme Özel Tablolar (Preprocessor ile Aktifleşenler):**
        *   `ReadApplyRandomTableFile(filename)` (`__ITEM_APPLY_RANDOM__`): Rastgele temel bonus konfigürasyonunu (`item_apply_random_table.txt`) okur.
        *   `ReadLuckyBoxFile(filename)` (`__LUCKY_BOX__`): Şans kutusu sisteminin konfigürasyonunu (`lucky_box.txt`) okur. Hangi kutu VNUM'unun hangi eşya grubuna karşılık geldiğini ve grupların içeriğini tanımlar.
        *   `LoadSetItemTable(filename)` (`__SET_ITEM__`): Set eşyalarının bonuslarını ve sete dahil eşyaların VNUM aralıklarını (`set_item.txt` gibi) okur.
    *   **Yardımcı ve Diğer Fonksiyonlar:**
        *   `ConvSpecialDropItemFile()`: `special_item_group.txt` dosyasını, VNUM'lar yerine eşya isimlerinin kullanıldığı bir formata dönüştürerek `special_item_group_vnum.txt` dosyasına yazar (muhtemelen hata ayıklama veya okunabilirlik için).
        *   `ReloadMobDropItemGroup(filename)` ve `ReloadSpecialItemGroup(filename)` (`__EXTENDED_RELOAD__`): Oyun çalışırken ilgili düşürme tablolarını yeniden yükleme imkanı sunar.
        *   `GetApplyRandom(...)`, `GetApplyRandomValue(...)` (`__ITEM_APPLY_RANDOM__`): Yüklenmiş rastgele temel bonus tablosundan bilgi alır.
        *   `GetLuckyBoxGroup()` (`__LUCKY_BOX__`): Yüklenmiş şans kutusu verilerine erişim sağlar.
        *   `GetItemSetValueMap()`, `GetItemSetItemMap()` (`__SET_ITEM__`): Yüklenmiş set eşya bilgilerine erişim sağlar.
*   **Genel Çalışma Prensibi:** Bu dosyadaki fonksiyonlar, genellikle sunucu başlatılırken veya belirli yeniden yükleme komutlarıyla çağrılır. `CTextFileLoader` veya standart C dosya G/Ç fonksiyonları kullanarak metin tabanlı konfigürasyon dosyalarını ayrıştırırlar. Okunan veriler (eşya VNUM'ları, olasılıklar, sayılar, isimler vb.) oyun içinde hızlı erişim ve kullanım için `ITEM_MANAGER` sınıfının çeşitli üye değişkenlerinde (genellikle haritalar veya vektörler) saklanır. Hata durumları genellikle `sys_err` ile loglanır ve ilgili fonksiyon `false` değeri döndürerek yükleme işleminin başarısız olduğunu belirtir.
*   **Bağlantılı Dosyalar:** `stdafx.h`, `utils.h`, `config.h`, `char.h`, `char_manager.h`, `item.h`, `item_manager.h`, `item_manager_private_types.h` (özellikle `CItemDropInfo` gibi yapılar için), `text_file_loader.h`, `group_text_parse_tree.h`, `locale_service.h`. Preprocessor direktiflerine bağlı olarak `item_apply_random_table.h`, `blend_item.h` gibi ek dosyalar da dahil edilebilir.

--- 

### `login_data.h`

*   **Amaç:** Oyuncunun giriş işlemi sırasında ve sonrasında geçici olarak saklanan verilerini tutmak için `CLoginData` sınıfını tanımlar. Bu veriler arasında güvenlik anahtarları, bağlantı bilgileri, giriş zamanı, IP adresi ve premium hizmet süreleri bulunur.
*   **Temel İşlevler/İçerik:**
    *   **`CLoginData` Sınıfı:**
        *   **Üye Değişkenler:**
            *   `m_dwKey`: Auth sunucusundan alınan veya oluşturulan bir anahtar.
            *   `m_adwClientKey[4]`: İstemci tarafından gönderilen güvenlik anahtarı.
            *   `m_dwConnectedPeerHandle`: Bağlı olan peer'in (genellikle DB veya auth sunucusu) handle'ı.
            *   `m_dwLogonTime`: Oyuncunun giriş yaptığı zaman (epoch time).
            *   `m_lRemainSecs`: Oturumun veya belirli bir sürenin kalan saniyesi (örn. ban süresi).
            *   `m_szIP[MAX_HOST_LENGTH + 1]`: Oyuncunun IP adresi.
            *   `m_bDeleted`: Bu `CLoginData` nesnesinin silinmek üzere işaretlenip işaretlenmediği.
            *   `m_stLogin`: Oyuncunun kullanıcı adı (login).
            *   `m_aiPremiumTimes[PREMIUM_MAX_NUM]`: Farklı premium hizmet türleri için kalan süreleri tutan bir dizi.
        *   **Metotlar (Setter/Getter):** Yukarıdaki üye değişkenlerin her biri için değer atama (`Set*`) ve değer okuma (`Get*`) metotları.
            *   `SetClientKey`, `GetClientKey`: İstemci anahtarını ayarlar ve alır.
            *   `SetKey`, `GetKey`: Oturum anahtarını ayarlar ve alır.
            *   `SetLogin`, `GetLogin`: Kullanıcı adını ayarlar ve alır.
            *   `SetConnectedPeerHandle`, `GetConnectedPeerHandle`: Bağlı peer handle'ını ayarlar ve alır.
            *   `SetLogonTime`, `GetLogonTime`: Giriş zamanını ayarlar ve alır.
            *   `SetIP`, `GetIP`: IP adresini ayarlar ve alır.
            *   `SetRemainSecs`, `GetRemainSecs`: Kalan saniyeyi ayarlar ve alır.
            *   `SetDeleted`, `IsDeleted`: Silinme durumunu ayarlar ve kontrol eder.
            *   `SetPremium`, `GetPremium`, `GetPremiumPtr`: Premium hizmet sürelerini ayarlar, belirli bir tür için alır veya tüm dizinin işaretçisini alır.
*   **Bağlantılı Dosyalar:** `login_data.cpp` (uygulama), `constants.h` (muhtemelen `PREMIUM_MAX_NUM` ve `MAX_HOST_LENGTH` için). Bu sınıf, genellikle `input_auth.cpp` veya `input_login.cpp` gibi giriş paketlerini işleyen dosyalarda kullanılır.

### `login_data.cpp`

*   **Amaç:** `login_data.h` dosyasında bildirilen `CLoginData` sınıfının metotlarını uygular. Sınıfın yapıcı metodunu ve tüm setter/getter fonksiyonlarının gövdelerini içerir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı Metot (`CLoginData::CLoginData()`):**
        *   Tüm üye değişkenleri varsayılan değerlerine başlatır:
            *   `m_dwKey` = 0
            *   `m_adwClientKey` dizisi 0'larla doldurulur.
            *   `m_dwConnectedPeerHandle` = 0
            *   `m_dwLogonTime` = 0
            *   `m_lRemainSecs` = 0
            *   `m_szIP` dizisi 0'larla doldurulur.
            *   `m_bDeleted` = false
            *   `m_aiPremiumTimes` dizisi 0'larla doldurulur.
            *   `m_stLogin` boş bir string olarak başlatılır (dolaylı olarak `std::string` yapıcısı ile).
    *   **Setter Metotları:**
        *   `SetClientKey`: `thecore_memcpy` kullanarak verilen `DWORD` dizisini `m_adwClientKey`'e kopyalar.
        *   `SetKey`, `SetConnectedPeerHandle`, `SetLogin`, `SetRemainSecs`, `SetDeleted`: İlgili üye değişkenlere doğrudan atama yapar. `SetRemainSecs` ayrıca bir log mesajı (`sys_log`) atar.
        *   `SetLogonTime`: `get_dword_time()` fonksiyonunu kullanarak mevcut zamanı alır ve `m_dwLogonTime`'a atar.
        *   `SetIP`: `strlcpy` kullanarak verilen IP string'ini `m_szIP`'ye kopyalar.
        *   `SetPremium`: `thecore_memcpy` kullanarak verilen `int` dizisini `m_aiPremiumTimes`'a kopyalar.
    *   **Getter Metotları:**
        *   İlgili üye değişkenlerin değerlerini veya işaretçilerini döndürürler. `GetPremium`, verilen türün `PREMIUM_MAX_NUM`'dan büyük olup olmadığını kontrol eder.
*   **Global Değişken (extern):**
    *   `g_stBlockDate` (std::string): Bu dosyada tanımlanmamış ancak extern olarak bildirilmiş bir global değişkendir. Muhtemelen hesap engelleme tarihleriyle ilgilidir, ancak `CLoginData` sınıfı içinde doğrudan kullanılmamaktadır.
*   **Bağlantılı Dosyalar:** `login_data.h` (tanım), `stdafx.h`, `constants.h`.

### `login_sim.h`

*   **Amaç:** Muhtemelen test veya özel senaryolar için sıralı bir şekilde birden fazla oyuncu adına giriş (login), karakter yükleme (load) ve çıkış (logout) işlemlerini simüle etmek amacıyla kullanılan `CLoginSim` sınıfını tanımlar. Veritabanı sunucusuna (`db_clientdesc`) bu işlemler için paketler gönderir.
*   **Temel İşlevler/İçerik:**
    *   **`CLoginSim` Sınıfı:**
        *   **Üye Değişkenler:**
            *   `auth` (`TPacketGDAuthLogin`): Auth sunucusuna gönderilecek giriş paketi verilerini tutar.
            *   `login` (`TPacketGDLoginByKey`): DB sunucusuna anahtar ile giriş için paket verilerini tutar.
            *   `load` (`TPlayerLoadPacket`): Karakter yükleme paket verilerini tutar.
            *   `logout` (`TLogoutPacket`): Çıkış paket verilerini tutar.
            *   `vecID` (`std::vector<DWORD>`): Simüle edilecek oyuncu ID'lerinin listesi.
            *   `vecIdx` (unsigned int): `vecID` içindeki mevcut işlem yapılan oyuncunun indeksi.
            *   `bCheck` (bool): Simülasyonun aktif olup olmadığını belirten bir bayrak.
        *   **Yapıcı Metot (`CLoginSim::CLoginSim()`):** Tüm paket yapılarını ve `vecIdx`, `bCheck` üyelerini sıfırlar/varsayılan değerlere ayarlar.
        *   **`AddPlayer(DWORD dwID)`:** Simüle edilecek oyuncu ID'sini `vecID` listesine ekler.
        *   **`IsCheck()`:** `bCheck` bayrağının durumunu döndürür.
        *   **`SendLogin()`:**
            *   `bCheck`'i `true` yapar.
            *   Eğer tüm oyuncular işlendiyse (`IsDone()` true ise) geri döner.
            *   Eğer ilk oyuncuysa (`vecIdx == 0`), `HEADER_GD_AUTH_LOGIN` paketi ile `auth` verisini gönderir.
            *   `vecID`'den sıradaki oyuncunun ID'sini `load.player_id`'ye atar, `vecIdx`'i artırır ve `HEADER_GD_LOGIN_BY_KEY` paketi ile `login` verisini gönderir.
        *   **`SendLoad()`:** `HEADER_GD_PLAYER_LOAD` paketi ile `load` verisini gönderir.
        *   **`SendLogout()`:** `HEADER_GD_LOGOUT` paketi ile `logout` verisini gönderir ve ardından bir sonraki oyuncu için `SendLogin()`'i çağırır.
        *   **`IsDone()`:** `vecIdx`'in `vecID` boyutuna ulaşıp ulaşmadığını kontrol ederek tüm oyuncuların işlenip işlenmediğini belirler.
*   **Kullanım Senaryosu:** Bu sınıf, belirli bir oyuncu listesi için otomatik olarak giriş yapma, karakter yükleme ve sonra çıkış yapıp bir sonraki oyuncuya geçme döngüsünü çalıştırmak için kullanılabilir. Örneğin, stres testi veya çoklu hesap yönetimi senaryolarında faydalı olabilir.
*   **Bağlantılı Dosyalar:** `desc_client.h` (`db_clientdesc` ve paket başlıkları/yapıları için).

### `lzo_manager.h`

*   **Amaç:** LZO (Lempel-Ziv-Oberhumer) veri sıkıştırma ve açma işlemlerini yönetmek için `LZOManager` singleton sınıfını tanımlar. `minilzo.h` başlık dosyasını kullanarak LZO algoritmasının implementasyonunu sarar.
*   **Temel İşlevler/İçerik:**
    *   **`LZOManager` Sınıfı (Singleton):**
        *   **Yapıcı/Yıkıcı:** `lzo_init()` ile LZO kütüphanesini başlatır ve sıkıştırma için gerekli çalışma belleğini (`m_workmem`) ayırır/serbest bırakır.
        *   **`Compress(const BYTE* src, size_t srcsize, BYTE* dest, lzo_uint* puiDestSize)`:** Verilen kaynak veriyi (`src`) LZO ile sıkıştırır ve sonucu `dest`'e yazar. Sıkıştırılmış boyutu `puiDestSize`'da döndürür.
        *   **`Decompress(const BYTE* src, size_t srcsize, BYTE* dest, lzo_uint* puiDestSize)`:** LZO ile sıkıştırılmış veriyi (`src`) açar ve sonucu `dest`'e yazar. Açılmış boyutu `puiDestSize`'da döndürür.
        *   **`GetMaxCompressedSize(size_t original)`:** Belirli bir orijinal boyut için LZO sıkıştırması sonrası oluşabilecek maksimum boyutu hesaplar. Bu, sıkıştırma için hedef tamponu ayırmak için kullanılır.
        *   **`GetWorkMemory()`:** Sıkıştırma/açma işlemleri için ayrılmış dahili çalışma belleği alanının işaretçisini döndürür.
    *   **Üyeler:**
        *   `m_workmem` (BYTE*): LZO işlemleri için gerekli olan çalışma belleği işaretçisi.
*   **Bağlantılı Dosyalar:** `lzo_manager.cpp` (uygulama), `minilzo.h` (LZO kütüphanesi), `singleton.h`. Bu sınıf, genellikle ağ iletişimi (paket sıkıştırma) veya veri saklama (dosya/veritabanı sıkıştırma) gibi yerlerde kullanılabilir.

### `lzo_manager.cpp`

*   **Amaç:** `lzo_manager.h`'de bildirilen `LZOManager` sınıfının metotlarını uygular. LZO kütüphanesi fonksiyonlarını çağırarak asıl sıkıştırma ve açma işlemlerini gerçekleştirir.
*   **Orta Seviye Implementasyon Detayları:**
    *   **Yapıcı (`LZOManager::LZOManager()`):**
        *   `lzo_init()` fonksiyonunu çağırarak LZO kütüphanesini başlatır. Başarısız olursa hata mesajı yazdırır ve programı sonlandırır (`abort()`).
        *   `LZO1X_MEM_COMPRESS` boyutunda bir bellek alanını `malloc` ile ayırır, `m_workmem` işaretçisine atar ve içini sıfırlar (`memset`).
    *   **Yıkıcı (`LZOManager::~LZOManager()`):**
        *   Yapıcıda ayrılan `m_workmem` bellek alanını `free` ile serbest bırakır.
    *   **`Compress(...)`:**
        *   `lzo1x_1_compress` fonksiyonunu çağırır. Bu fonksiyon, kaynak veriyi (`src`), boyutunu (`srcsize`), hedef tamponu (`dest`), sıkıştırılmış boyutu alacak işaretçiyi (`puiDestSize`) ve çalışma belleğini (`GetWorkMemory()`) parametre olarak alır.
        *   `lzo1x_1_compress` fonksiyonunun dönüş değerini kontrol eder. `LZO_E_OK` değilse `false`, başarılıysa `true` döndürür.
    *   **`Decompress(...)`:**
        *   `lzo1x_decompress_safe` fonksiyonunu çağırır. Bu, sıkıştırılmış verinin potansiyel olarak hatalı olması durumunda hedef tampon taşmasını önleyen güvenli bir açma fonksiyonudur. Parametreleri `Compress` ile benzerdir, ancak `puiDestSize` açılmış boyutu alır.
        *   `lzo1x_decompress_safe` fonksiyonunun dönüş değerini kontrol eder. `LZO_E_OK` değilse `false`, başarılıysa `true` döndürür.
    *   **`GetMaxCompressedSize(size_t original)`:**
        *   LZO kütüphanesinin dokümantasyonunda belirtilen formülü uygular: `original + (original / 16) + 64 + 3`. Bu, en kötü durumda sıkıştırılmış verinin orijinalinden biraz daha büyük olabileceği gerçeğini yansıtır.
*   **Çalışma Prensibi:** `LZOManager` oluşturulduğunda LZO kütüphanesi başlatılır ve gerekli bellek ayrılır. Sıkıştırma veya açma gerektiğinde, ilgili metot çağrılır ve bu metot doğrudan LZO kütüphanesinin (`minilzo`) ilgili fonksiyonunu çağırarak işlemi gerçekleştirir. `GetMaxCompressedSize`, sıkıştırma yapılacak verinin boyutuna göre güvenli bir hedef tampon boyutu belirlemek için kullanılır.
*   **Bağlantılı Dosyalar:** `lzo_manager.h`, `stdafx.h`.

### `Makefile` (`Source/srcServer/Source/game/src/Makefile`)

*   **Amaç:** `Source/srcServer/Source/game/src/` dizinindeki Metin2 oyun sunucusu (`game_dev` adlı çalıştırılabilir dosya) kaynak kodunu derlemek ve yönetmek için kullanılan bir `gmake` (GNU Make) dosyasıdır. Derleyici seçeneklerini, kütüphane ve başlık dosyası yollarını, derlenecek C/C++ kaynak dosyalarını ve çeşitli yardımcı hedefleri (temizleme, etiketleme, bağımlılık oluşturma) tanımlar.
*   **Temel İşlevler/İçerik:**
    *   **Derleyici ve Araç Tanımları:**
        *   `MAKE = gmake`: Kullanılacak `make` programı.
        *   `CC = clang++90`: Kullanılacak C++ derleyicisi (Clang sürüm 9.0).
    *   **Dizin Tanımları:**
        *   `INCDIR`: Derleme sırasında başlık dosyalarının (`.h`, `.hpp`) aranacağı dizinleri içerir. Örnekler:
            *   `/usr/local/include` (genel sistem başlıkları).
            *   `../../../External/include` (proje dışı harici kütüphane başlıkları).
            *   `../../../External/include/boost` (Boost kütüphanesi başlıkları).
            *   `../../../External/include/devil/IL` (DevIL kütüphanesi başlıkları).
            *   `/usr/local/include/mysql` (MySQL başlıkları, 32-bit platform için).
            *   `/usr/include` (OpenSSL başlıkları için).
            *   `../../liblua/include` (Proje içi Lua kütüphanesi başlıkları).
            *   `../../libserverkey` (Proje içi serverkey kütüphanesi başlıkları).
        *   `LIBDIR`: Derleme sırasında ve bağlantı (linking) aşamasında kütüphane dosyalarının (`.a`, `.so`) aranacağı dizinleri içerir. Örnekler:
            *   `/usr/local/lib` (genel sistem kütüphaneleri).
            *   `../../../External/library` (proje dışı harici kütüphaneler).
            *   FreeBSD sürümüne göre (`11.0` veya `12.0`) özel harici kütüphane yolları.
            *   `../../libthecore/lib`, `../../libpoly`, `../../libsql`, `../../libgame/lib`, `../../liblua/lib`, `../../libserverkey` (proje içi diğer modüllerin derlenmiş kütüphaneleri).
        *   `BINDIR = ..`: Derlenmiş nihai çalıştırılabilir dosyanın (`game_dev`) yerleştirileceği dizin (mevcut `src` dizininin bir üstü, yani `game` dizini).
        *   `OBJDIR = .obj`: Derleme sırasında oluşturulan ara nesne dosyalarının (`.o`) saklanacağı dizin (`src/.obj`). Bu dizin yoksa oluşturulur.
    *   **Platform ve Versiyon Tespiti (Shell Komutları ile):**
        *   `PLATFORM`: Çalıştırılan sistemin mimarisini (örn. 32-bit/64-bit) belirler.
        *   `GCC_VERSION`: GCC derleyicisinin sürümünü alır (Clang kullanılsa da bu değişken tanımlı).
        *   `BSD_VERSION`: FreeBSD işletim sisteminin sürümünü belirler. Bu, özellikle harici kütüphane yollarını seçmek için kullanılır.
    *   **Derleme Bayrakları (Compilation Flags):**
        *   `CFLAGS`: Hem C hem de C++ dosyaları için geçerli olan genel derleyici bayrakları.
            *   `-m32`: 32-bit mimari için çıktı üretir.
            *   `-w`: Çoğu uyarıyı bastırır (genellikle `-Wall` ile birlikte daha spesifik `-Wno-*` kullanılırken, burada sadece `-w` var).
            *   `-g`: Hata ayıklama (debugging) sembollerini ekler.
            *   `-Wall`: Önemli tüm uyarıları etkinleştirir.
            *   `-O2`: Optimizasyon seviyesi 2.
            *   `-pipe`: Derleme adımları arasında pipe kullanır (hızlandırabilir).
            *   `-fexceptions`: C++ istisna (exception) yönetimini etkinleştirir.
            *   `-fno-strict-aliasing`: Strict aliasing kurallarını esnetir (bazı eski kodlarla uyumluluk için).
            *   `-pthread`, `-D_THREAD_SAFE`: Çoklu iş parçacığı (multi-threading) desteği için.
            *   `-DNDEBUG`: `assert()` gibi hata ayıklama makrolarını devre dışı bırakır (genellikle sürüm (release) derlemeleri için).
            *   `-Wno-invalid-source-encoding`: Kaynak kodu kodlamasıyla ilgili belirli bir uyarıyı bastırır.
            *   `-fstack-protector-all`: Yığın taşması (stack smashing) korumalarını etkinleştirir.
        *   `CXXFLAGS += -std=c++2a`: C++ dosyaları için ek bayraklar. C++ standardını C++20 (veya o zamanki taslak sürümü) olarak ayarlar.
    *   **Bağlanacak Kütüphaneler (`LIBS`):**
        *   `-lm`: Matematik kütüphanesi.
        *   `-lmd`: MD5 kütüphanesi (FreeBSD'de).
        *   DevIL (Resim Kütüphanesi): `-lIL -lpng -ltiff -lmng -llcms -ljpeg`. Ayrıca `/usr/lib/liblzma.a` (LZMA sıkıştırma kütüphanesi) statik olarak bağlanır.
        *   CryptoPP (Kriptografi Kütüphanesi): `../../../External/library/libcryptopp.a` statik olarak bağlanır.
        *   MySQL: Platforma göre (`PLATFORM`) 64-bit ise `-lmysqlclient -lz -lzstd`, değilse (32-bit varsayımıyla) `/usr/local/lib/mysql/libmysqlclient.a /usr/lib/libz.a /usr/local/lib/libzstd.a` statik yolları kullanılır.
        *   OpenSSL: `-lssl -lcrypto`.
        *   Proje İçi Kütüphaneler: `-lthecore`, `-lpoly`, `-llua`, `-llualib`, `-lsql`, `-lgame`, `-lserverkey`.
    *   **Kaynak Dosya Listeleri:**
        *   `CFILE = minilzo.c`: Derlenecek tek C kaynak dosyası.
        *   `CPPFILE`: Derlenecek tüm C++ (`.cpp`) kaynak dosyalarının kapsamlı bir listesi (örn: `BattleArena.cpp`, `char.cpp`, `item.cpp`, `questlua.cpp`, `main.cpp` hariç diğer tüm cpp'ler).
        *   `COBJS = $(CFILE:%.c=$(OBJDIR)/%.o)`: C kaynak dosyalarından üretilecek nesne dosyalarının listesi.
        *   `CPPOBJS = $(CPPFILE:%.cpp=$(OBJDIR)/%.o)`: C++ kaynak dosyalarından üretilecek nesne dosyalarının listesi.
        *   `MAINOBJ = $(OBJDIR)/main.o`: `main.cpp`'den üretilecek nesne dosyası.
        *   `MAINCPP = main.cpp`: Ana program dosyasını belirtir.
    *   **Hedefler (Make Targets):**
        *   `TARGET = $(BINDIR)/game_dev`: Oluşturulacak nihai çalıştırılabilir dosyanın tam yolu.
        *   `default: $(TARGET)`: `make` komutu parametresiz çalıştırıldığında varsayılan olarak bu hedefi (yani `game_dev`'i derlemeyi) tetikler.
        *   Nesne Dosyası Derleme Kuralları:
            *   `$(OBJDIR)/minilzo.o: minilzo.c ...`: `minilzo.c`'yi derler.
            *   `$(OBJDIR)/version.o: version.cpp ...`: `version.cpp`'yi derler.
            *   `$(OBJDIR)/%.o: %.cpp ...`: Diğer tüm `.cpp` dosyalarını derlemek için genel bir kural.
        *   Bağlantı (Linking) Kuralı: `$(TARGET): $(CPPOBJS) $(COBJS) $(MAINOBJ) ...`: Tüm derlenmiş nesne dosyalarını (`.o`) ve `LIBS` ile belirtilen kütüphaneleri kullanarak `$(TARGET)` çalıştırılabilir dosyasını oluşturur.
        *   `clean`: `$(OBJDIR)` içindeki tüm nesne dosyalarını ve `$(BINDIR)` içindeki `game_dev` ve `conv` (başka bir hedef olabilir) dosyalarını siler.
        *   `tag`: `ctags *.cpp *.h *.c` komutunu çalıştırarak kod içinde gezinmeyi kolaylaştıran bir etiket dosyası oluşturur.
        *   `dep`: `makedepend` aracını kullanarak kaynak dosyalar arasındaki bağımlılıkları analiz eder ve `Depend` adlı bir dosyaya yazar. Bu dosya, `make`'in sadece değiştirilmiş ve bağımlı dosyaları yeniden derlemesini sağlar.
        *   `sinclude Depend`: Eğer `Depend` dosyası varsa, onu `Makefile`'a dahil eder.
        *   `limit_time`: `update_limit_time.py` adlı bir Python betiğini çalıştırır. Bu betiğin amacı `Makefile`'dan doğrudan anlaşılamaz, ancak muhtemelen sunucuyla ilgili bir zaman sınırlamasını günceller.
*   **Bağlantılı Dosyalar:** Bu `Makefile`, `Source/srcServer/Source/game/src/` dizinindeki tüm C ve C++ kaynak dosyalarıyla, ayrıca `INCDIR` ve `LIBDIR` değişkenlerinde belirtilen tüm harici ve proje içi kütüphane ve başlık dosyalarıyla doğrudan ilişkilidir.

</rewritten_file>