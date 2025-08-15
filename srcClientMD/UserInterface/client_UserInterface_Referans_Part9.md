# UserInterface Referans Kılavuzu - Bölüm 9

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonGuild.h` (Python Lonca Sistemi Başlık Dosyası)](#pythonguildh-python-lonca-sistemi-başlık-dosyası)
*   [`PythonGuild.cpp` (Python Lonca Sistemi Implementasyonu ve `guild` Modülü)](#pythonguildcpp-python-lonca-sistemi-implementasyonu-ve-guild-modülü)
*   [`PythonIllustratedManager.h` (Resimli Kılavuz/Model Görüntüleyici Yöneticisi Başlık Dosyası)](#pythonillustratedmanagerh-resimli-kılavuzmodel-görüntüleyici-yöneticisi-başlık-dosyası)
*   [`PythonIllustratedManager.cpp` (Resimli Kılavuz/Model Görüntüleyici Yöneticisi Implementasyonu)](#pythonillustratedmanagercpp-resimli-kılavuzmodel-görüntüleyici-yöneticisi-implementasyonu)
*   [`PythonIME.h` (Python IME Entegrasyon Başlık Dosyası)](#pythonimeh-python-ime-entegrasyon-başlık-dosyası)
*   [`PythonIME.cpp` (Python IME Entegrasyon Implementasyonu)](#pythonimecpp-python-ime-entegrasyon-implementasyonu)
*   [`PythonIMEModule.cpp` (Python `ime` Modülü)](#pythonimemodulecpp-python-ime-modülü)

---

### `PythonGuild.h` (Python Lonca Sistemi Başlık Dosyası)

Bu başlık dosyası, `CPythonGuild` adlı bir singleton C++ sınıfını tanımlar. Bu sınıf, istemci tarafındaki lonca (guild) ile ilgili tüm verileri ve temel işlevleri yönetmekten sorumludur. Oyuncunun kendi loncasının detayları, üye listesi, rütbeler ve yetkileri, lonca becerileri, lonca panosu yorumları, diğer loncalarla olan savaş durumları gibi pek çok bilgiyi içerir ve yönetir.

#### Dosyanın Genel Amaçları

*   **`CPythonGuild` Sınıf Bildirimi**: Lonca ile ilgili tüm istemci tarafı verilerini ve işlemlerini merkezi olarak yöneten singleton sınıfını deklare eder.
*   **Veri Yapıları Tanımlama**: Lonca bilgilerini, rütbelerini, üyelerini, pano yorumlarını ve becerilerini temsil etmek için çeşitli `struct`'lar tanımlar:
    *   `SGulidInfo` (typedef `TGuildInfo`): Temel lonca bilgilerini (ID, isim, lider, seviye, EXP, üye sayısı, para, arazi durumu) tutar.
    *   `SGuildGradeData` (typedef `TGuildGradeData`): Lonca içindeki bir rütbenin (grade) ismini ve yetki bayraklarını tanımlar.
    *   `SGuildMemberData` (typedef `TGuildMemberData`): Bir lonca üyesinin detaylarını (PID, isim, rütbe, meslek, seviye, genel bayrak, bağış miktarı) saklar.
    *   `SGuildBoardCommentData` (typedef `TGuildBoardCommentData`): Lonca panosundaki bir yorumun ID'sini, yazarını ve içeriğini tutar.
    *   `SGuildSkillData` (typedef `TGuildSkillData`): Loncanın sahip olduğu beceri puanlarını, her bir becerinin seviyesini ve lonca puanlarını (ejderha gücü) içerir.
*   **Typedef ve Enum Tanımları**:
    *   `TGradeDataMap`, `TGuildMemberDataVector`, `TGuildBoardCommentDataVector`, `TGuildNameMap`: Yukarıdaki yapılarla ilişkili standart STL map ve vectorlerini tanımlar.
    *   `GUILD_SKILL_MAX_NUM`: Kullanılabilir maksimum lonca beceri sayısını belirtir.
    *   `ENEMY_GUILD_SLOT_MAX_COUNT`: Aynı anda savaş halinde olunabilecek maksimum düşman lonca sayısını tanımlar.
*   **Genel Fonksiyon Prototipleri (Public)**: `CPythonGuild` sınıfının dışa açık arayüzünü tanımlar. Bu fonksiyonlar şunları içerir:
    *   Veri ayarlama (`Set*` fonksiyonları): Lonca parası, EXP, rütbe detayları, üye bilgileri, yorumlar, lonca isimleri gibi verileri güncellemek için.
    *   Veri sorgulama (`Get*` fonksiyonları): Mevcut lonca bilgilerini, rütbe detaylarını, üye listesini, yorumları, beceri seviyelerini, lonca savaş durumlarını vb. almak için.
    *   Durum kontrolü (`Is*` fonksiyonları): Lonca sisteminin aktif olup olmadığı, ana oyuncunun belirli bir PID olup olmadığı, lonca savaşı yapılıp yapılmadığı gibi durumları kontrol etmek için.
    *   Eylemler: `EnableGuild`, `Destroy`, `ClearComment`, `RegisterMember`, `RemoveMember`, `StartGuildWar`, `EndGuildWar`.
*   **Koşullu Derleme Blokları**: `ENABLE_LOADING_PERFORMANCE` gibi makrolarla belirli özelliklerin (örneğin, lonca bina listesi) derlenip derlenmeyeceğini kontrol eder.
*   **Dahili Yönetim Fonksiyonları (Protected)**: `__CalculateLevelAverage` (üyelerin ortalama seviyesini hesaplar), `__SortMember` (üyeleri rütbeye göre sıralar), `__IsGradeData` (rütbe geçerliliğini kontrol eder), `__Initialize` (tüm verileri sıfırlar) gibi sınıf içi yardımcı fonksiyonlar.
*   **Üye Değişkenler**: `m_GuildInfo`, `m_GradeDataMap`, `m_GuildMemberDataVector` gibi yukarıda tanımlanan veri yapılarını ve `m_adwEnemyGuildID`, `m_bGuildEnable` gibi durum değişkenlerini tutar.

#### C++ ve Python Arayüzü ile İlişkisi (Dolaylı)

`CPythonGuild` sınıfı, doğrudan Python'a sunulmasa da, Python'da oluşturulan `guild` modülünün C++ tarafındaki temelini oluşturur. `PythonGuild.cpp` dosyasındaki Python sarmalayıcı fonksiyonları, `CPythonGuild::Instance()` üzerinden bu sınıfın public metotlarını çağırarak lonca verilerini ve işlevlerini Python UI scriptlerinin erişimine sunar. Sunucudan gelen lonca ile ilgili ağ paketleri (`CPythonNetworkStream` aracılığıyla) yine bu sınıfın `Set*` ve diğer ilgili metotlarını çağırarak istemcideki lonca durumunu günceller.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Lonca Arayüzü Yönetimi**: Oyuncunun lonca penceresindeki tüm sekmeler (Bilgi, Üyeler, Rütbeler, Beceriler, Pano, Savaşlar) için gerekli verileri sağlar.
*   **Lonca Bilgilerinin Gösterimi**: Lonca adı, seviyesi, lideri, üye sayısı, parası, duyuruları vb. bilgilerin UI'da gösterilmesi.
*   **Üye Yönetimi**: Üye listesinin görüntülenmesi, üyelerin rütbelerinin, seviyelerinin ve online durumlarının takibi.
*   **Rütbe ve Yetki Yönetimi**: Lonca içindeki rütbelerin ve bu rütbelere atanmış yetkilerin (üye ekleme, atma, duyuru yapma, beceri kullanma) UI'da gösterilmesi ve yetkiye bağlı işlemlerin kontrolü.
*   **Lonca Becerileri**: Loncanın sahip olduğu becerilerin ve seviyelerinin görüntülenmesi.
*   **Lonca Savaşları**: Mevcut lonca savaşlarının ve düşman loncaların listelenmesi.

Bu başlık dosyası, Metin2 istemcisinin lonca sistemine ait verilerin ve temel mantığının organize edildiği merkezi bir yapıyı tanımlar.

---

### `PythonGuild.cpp` (Python Lonca Sistemi Implementasyonu ve `guild` Modülü)

Bu dosya iki ana bölümden oluşur: `CPythonGuild` singleton C++ sınıfının tam implementasyonu ve bu sınıfın işlevlerini Python betiklerine sunan `guild` adlı Python modülü.

#### `CPythonGuild` Sınıfı Implementasyon Detayları

Bu bölüm, `PythonGuild.h` dosyasında deklare edilen `CPythonGuild` sınıfının metotlarının gövdelerini içerir. Temel sorumlulukları arasında lonca verilerinin (genel bilgiler, üyeler, rütbeler, beceriler, savaşlar, pano yorumları vb.) saklanması, güncellenmesi ve yönetilmesi bulunur.

*   **Yapıcı, Yıkıcı ve Başlatma (`__Initialize`, `Destroy`)**:
    *   Sınıf oluşturulduğunda (`CPythonGuild()`) veya `Destroy()` çağrıldığında, `__Initialize()` metodu aracılığıyla tüm lonca verileri (lonca bilgisi, rütbe haritası, üye vektörü, yorum vektörü, beceri verileri, düşman lonca ID'leri dizisi, istatistiksel özetler ve `m_bGuildEnable` bayrağı) varsayılan durumlarına sıfırlanır veya temizlenir.
    *   `Destroy()` ayrıca global `g_GuildSkillSlotToIndexMap` haritasını da temizler.

*   **Veri Ayarlama (`Set*` Fonksiyonları)**:
    *   `EnableGuild()`: `m_bGuildEnable` bayrağını `TRUE` yaparak lonca sisteminin aktif olduğunu belirtir.
    *   `SetGuildMoney()`, `SetGuildEXP()`: Loncanın parasını ve tecrübe/seviye bilgilerini `m_GuildInfo` içinde günceller.
    *   `SetGradeData()`: Yeni bir rütbe bilgisini (`TGuildGradeData`) `m_GradeDataMap`'e ekler veya var olanı günceller.
    *   `SetGradeName()`, `SetGradeAuthority()`: Belirli bir rütbenin adını veya yetki bayrağını, rütbe numarasını bularak `m_GradeDataMap` içinde günceller.
    *   `ClearComment()`: `m_GuildBoardCommentVector`'ü temizleyerek tüm pano yorumlarını siler.
    *   `RegisterComment()`: Yeni bir yorumu (`TGuildBoardCommentData`) alır ve boş değilse `m_GuildBoardCommentVector`'e ekler.
    *   `RegisterMember()`: Gelen `TGuildMemberData`'yı işler. Eğer üye PID'si zaten `m_GuildMemberDataVector` içinde varsa, o üyenin genel bayrağını, rütbesini, seviyesini ve bağışını günceller. Yoksa, yeni üyeyi vektöre ekler. Her iki durumda da `__CalculateLevelAverage()` (ortalama seviye ve toplam bağışı yeniden hesaplamak için) ve `__SortMember()` (üyeleri rütbeye göre sıralamak için) çağrılır.
    *   `ChangeGuildMemberGrade()`, `ChangeGuildMemberGeneralFlag()`: Belirli bir üyenin (PID ile bulunur) rütbesini veya genel bayrağını günceller.
    *   `RemoveMember()`: Verilen PID'ye sahip üyeyi `m_GuildMemberDataVector`'den bulur (`std::find_if` ve özel arama functor'ı `CPythonGuild_FFindGuildMemberByPID` ile) ve siler.
    *   `RegisterGuildName()`: Bir lonca ID'si ve ismini `m_GuildNameMap`'e kaydeder. Bu genellikle oyuncunun kendi loncası dışındaki loncaların isimlerini saklamak için kullanılır.

*   **Veri Alma (`Get*`) ve Durum Kontrolü (`Is*`) Fonksiyonları**:
    *   Bu fonksiyonlar genellikle ilgili `m_` üye değişkenlerinden veya `m_GuildInfo` gibi saklanan yapılardan doğrudan veri okur ve döndürür.
    *   `GetMemberDataPtr()`, `GetMemberDataPtrByPID()`, `GetMemberDataPtrByName()`: Üyeleri sırasıyla indeks, PID veya isme göre arar. PID ve isme göre arama için `std::find_if` ve özel arama functor'ları (`CPythonGuild_FFindGuildMemberByPID`, `CPythonGuild_FFindGuildMemberByName`) kullanılır. Başarılı olursa üye verisine bir işaretçi döndürür.
    *   `IsMainPlayer()`: Verilen PID'nin o anki ana oyuncuya (`IAbstractPlayer::GetSingleton().GetName()`) ait olup olmadığını kontrol eder.
    *   `StartGuildWar()`, `EndGuildWar()`: `m_adwEnemyGuildID` dizisinde boş bir slot bularak veya mevcut bir ID'yi sıfırlayarak savaş durumunu yönetir.
    *   `IsDoingGuildWar()`: `m_adwEnemyGuildID` dizisinde sıfırdan farklı bir ID olup olmadığını kontrol ederek loncanın savaşta olup olmadığını belirler.

*   **Korunan (Protected) Yardımcı Fonksiyonlar**:
    *   `__CalculateLevelAverage()`: `m_GuildMemberDataVector` içindeki tüm üyelerin seviyelerini toplayarak `m_dwMemberLevelSummary`'yi ve bağışlarını toplayarak `m_dwMemberExperienceSummary`'yi hesaplar. Ardından üye sayısına bölerek `m_dwMemberLevelAverage`'ı bulur.
    *   `__SortMember()`: `m_GuildMemberDataVector`'ü, üyelerin rütbe (`byGrade`) değerlerine göre artan sırada sıralamak için `std::sort` ve özel karşılaştırma functor'ı `CPythonGuild_SLessMemberGrade`'i kullanır.
    *   `__IsGradeData()`: Verilen bir rütbe numarasının `m_GradeDataMap` içinde tanımlı olup olmadığını kontrol eder.

*   **Global Değişken (`g_GuildSkillSlotToIndexMap`)**: Bu `std::map<DWORD, DWORD>`, UI'da kullanılan lonca beceri slot indekslerini, `m_GuildSkillData.bySkillLevel` dizisindeki gerçek beceri indeksiyle eşleştirmek için kullanılır. Bu eşleme `guildSetSkillIndex` Python fonksiyonuyla doldurulur.

#### `guild` Python Modülü

Bu bölüm, `CPythonGuild` sınıfının işlevselliğini Python betiklerine açan `guild` adlı bir Python modülünü tanımlar ve başlatır.

*   **`initguild()` Fonksiyonu**:
    *   Python yorumlayıcısı tarafından `guild` modülü ilk kez yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyon adlarını (örn. "GetGuildID", "GetMemberData") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örn. `guildGetGuildID`, `guildGetMemberData`) tanımlar. Bu dizi oldukça kapsamlıdır ve lonca ile ilgili hemen hemen her veriye erişim sağlar.
    *   `PyObject* poModule = Py_InitModule("guild", s_methods);` ile `guild` modülü oluşturulur.
    *   `PyModule_AddIntConstant` kullanılarak lonca yetki sabitleri (`GUILD_AUTH_ADD_MEMBER`, `GUILD_AUTH_REMOVE_MEMBER`, `GUILD_AUTH_NOTICE`, `GUILD_AUTH_SKILL`) ve `ENEMY_GUILD_SLOT_MAX_COUNT` sabiti Python modülüne (`guild.AUTH_ADD_MEMBER` vb.) aktarılır.

*   **Python Fonksiyon Sarmalayıcıları (`guild*` önekli fonksiyonlar)**:
    Bu fonksiyonlar, `CPythonGuild::Instance()` üzerinden C++ metotlarını çağırır ve sonuçları Python veri türlerine dönüştürür.
    *   **Temel Bilgiler**: `guildIsGuildEnable`, `guildGetGuildID`, `guildHasGuildLand`, `guildGetGuildName` (mevcut lonca veya ID ile başka lonca), `guildGetGuildMasterName`, `guildGetEnemyGuildName` (indekse göre), `guildGetGuildMoney`.
    *   **Pano Yorumları**: `guildGetGuildBoardCommentCount`, `guildGetGuildBoardCommentData` (indekse göre yorum ID, isim, metin döndürür).
    *   **Lonca Seviyesi ve Üyeler**: `guildGetGuildLevel`, `guildGetGuildExperience` (mevcut ve sonraki seviyeye kalan EXP), `guildGetGuildMemberCount` (mevcut/maksimum), `guildGetGuildMemberLevelSummary`, `guildGetGuildMemberLevelAverage`, `guildGetGuildExperienceSummary`.
    *   **Lonca Becerileri**: `guildGetGuildSkillPoint`, `guildGetDragonPowerPoint` (lonca puanı/maksimum puan), `guildGetSkillLevel` (slot indeksinden `g_GuildSkillSlotToIndexMap` ile gerçek indekse ulaşıp beceri seviyesini alır), `guildSetSkillIndex` (slot-gerçek indeks eşlemesini `g_GuildSkillSlotToIndexMap`'e ekler), `guildGetSkillIndex` (slottan gerçek indeksi alır). `guildGetGuildSkillLevel` fonksiyonu eski veya kullanılmayan bir fonksiyon olarak `assert(FALSE)` içerir.
    *   **Rütbeler**: `guildGetGradeData` (rütbe numarasına göre isim ve yetki bayrağı), `guildGetGradeName` (sadece isim).
    *   **Üye Detayları**: `guildGetMemberCount` (toplam kayıtlı üye sayısı), `guildGetMemberData` (indekse göre üyenin PID, isim, rütbe, meslek, seviye, bağış, genel bayrak bilgilerini tuple olarak döndürür), `guildMemberIndexToPID` (indeksten PID alır), `guildIsMember` (indeksteki üyenin geçerli olup olmadığı), `guildIsMemberByName` (isme göre üye olup olmadığı).
    *   **Yetki Kontrolü**: `guildMainPlayerHasAuthority` (ana oyuncunun belirli bir lonca yetkisine sahip olup olmadığını kontrol eder).
    *   **Lonca Markası (Amblemi)**: `guildGuildIDToMarkID` (`CGuildMarkManager` kullanarak lonca ID'sinden marka ID'sini alır), `guildGetMarkImageFilenameByMarkID` (marka ID'sinden marka resim dosyasının paket içindeki yolunu alır), `guildGetMarkIndexByMarkID` (marka ID'sinden resim dosyası içindeki marka indeksini alır).
    *   **Yönetim**: `guildDestroy` (`CPythonGuild::Instance().Destroy()` çağırır ve `g_GuildSkillSlotToIndexMap`'i temizler).
    *   **Lonca Binaları** (`ENABLE_LOADING_PERFORMANCE` ile): `guildGetGuildBuildingList` (Python nesnesi olarak lonca bina listesini döndürür).

#### Oyunda Kullanım Amacı ve Senaryoları

`CPythonGuild` sınıfı ve onun Python'a açılan `guild` modülü, oyun istemcisinin lonca arayüzünün (UI) bel kemiğidir.

*   **Lonca Penceresi**: Oyuncunun lonca penceresindeki tüm bilgiler (lonca seviyesi, üyeler, rütbeler, beceriler, duyurular, savaşlar vb.) bu modül aracılığıyla `CPythonGuild`'dan çekilerek görüntülenir.
*   **Veri Güncelleme**: Sunucudan gelen lonca ile ilgili paketler (yeni üye, rütbe değişikliği, beceri kullanımı, savaş durumu vb.) `CPythonGuild` içindeki verileri günceller. UI, bu güncel verilere `guild` modülü fonksiyonlarıyla erişir.
*   **Yetki Kontrolleri**: Oyuncunun lonca içinde bir eylem yapmaya (örneğin, üye atma, duyuru yazma) yetkisi olup olmadığı `guild.MainPlayerHasAuthority()` gibi fonksiyonlarla kontrol edilir.
*   **Lonca Amblemleri**: Diğer karakterlerin veya lonca binalarının üzerinde gösterilen lonca amblemleri, lonca ID'sinden yola çıkarak `CGuildMarkManager` ve bu modüldeki ilgili fonksiyonlar (`guildGuildIDToMarkID` vb.) aracılığıyla yüklenir ve çizilir.
*   **Lonca Becerileri**: Lonca beceri penceresi, `guild.GetSkillLevel()` ve `guild.GetGuildSkillPoint()` gibi fonksiyonlarla doldurulur.

Bu dosya, istemcinin lonca sistemiyle ilgili karmaşık veri yapısını ve mantığını etkili bir şekilde yönetir ve Python UI scriptlerinin bu bilgilere kolayca erişmesini sağlar.

---

### `PythonIllustratedManager.h` (Resimli Kılavuz/Model Görüntüleyici Yöneticisi Başlık Dosyası)

Bu başlık dosyası, `CPythonIllustratedManager` adlı bir singleton C++ sınıfını tanımlar. Bu sınıf, oyun içinde genellikle "Resimli Kılavuz", "Canavar Ansiklopedisi" veya benzeri bir arayüzde 3D modellerin (karakterler, canavarlar, eşyalar vb.) özel bir sahnede görüntülenmesi, yönetilmesi ve etkileşimde bulunulması (döndürme, yakınlaştırma, ekipman değiştirme) için gerekli altyapıyı sağlar.

#### Dosyanın Genel Amaçları

*   **`CPythonIllustratedManager` Sınıf Bildirimi**: 3D model görüntüleme işlevlerini merkezi olarak yöneten singleton sınıfını deklare eder.
*   **Veri Yapıları ve Üye Değişkenler**: Görüntülenecek model, arka plan resmi, önbelleğe alınmış modellerin bir haritası ve kamera/model pozisyonlandırması için çeşitli float değişkenleri (isimlendirilmemiş `m_fUnk*` serisi) ve görünürlük bayrağı (`m_bShow`) tanımlar.
    *   `m_pModelInstance`: Aktif olarak görüntülenen `CInstanceBase` (3D model) işaretçisi.
    *   `m_pBackgroundImage`: Görüntüleyici için `CGraphicImageInstance` türünde arka plan resmi.
    *   `m_mapModelInstance`: `std::map<DWORD, CInstanceBase *>` (VNUM -> Model İşaretçisi), daha önce yüklenmiş modelleri tekrar kullanmak için saklar.
    *   `m_fUnk08`, `m_fUnk12`, `m_fUnk16`, `m_fUnk20`, `m_fUnk24`: Modelin yüksekliği, kamera uzaklığı/yakınlaştırma ofseti, yakınlaştırma adım değeri, kamera dikey kaydırma ofseti ve dikey kaydırma adım değeri gibi parametreler.
    *   `m_bShow`: Görüntüleyicinin render edilip edilmeyeceğini belirten bayrak.
*   **Temel Fonksiyon Prototipleri (Public)**:
    *   **Kurulum ve Temizleme**: `__Initialize`, `Destroy`, `ClearInstances`.
    *   **Görsel Hazırlık**: `CreateBackground`.
    *   **Model Yönetimi**: `GetInstancePtr`, `CreateModelInstance`, `SelectModel`.
    *   **Render İşlemleri**: `RenderBackground`, `DeformModel`, `RenderModel`.
    *   **Güncelleme ve Etkileşim**: `UpdateModel`, `ChangeMotion`, `ModelViewReset`, `ModelRotation`, `ModelZoom`, `ModelUpDown`.
    *   **Ekipman Yönetimi**: `SetHair`, `SetArmor`, `SetWeapon`.
    *   **Görünürlük**: `SetShow`.

#### C++ ve Python Arayüzü ile İlişkisi

Bu sınıf, C++ tarafında bir model görüntüleme sistemi oluşturur. Fonksiyonları, genellikle bu sınıfı kullanan C++ tabanlı UI pencereleri veya bu sınıfın işlevlerini Python'a sarmalayan (wrapper) başka bir Python modülü aracılığıyla çağrılır. `PythonIllustratedManager` ismi, Python entegrasyonu için tasarlandığını ima eder, ancak bu başlık dosyası doğrudan bir Python modülü oluşturmaz.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Canavar Ansiklopedisi (Bestiary)**: Oyuncuların karşılaştıkları canavarların 3D modellerini detaylı olarak inceleyebilmeleri için.
*   **Karakter/Ekipman Görüntüleme**: Envanterde, karakter bilgi penceresinde veya mağazada eşyaların (zırh, silah) veya karakterin genel görünümünün önizlemesi için.
*   **NPC Tanıtımı**: Önemli NPC'lerin modellerini özel bir arayüzde sergilemek için.
*   **Yardım ve Kılavuzlar**: Oyun içi rehberlerde belirli varlıkların görsel temsillerini sunmak için.

Bu sınıf, oyunun ana render döngüsünden ayrı, kontrollü bir ortamda 3D modellerin sergilenmesi için güçlü bir araç seti sunar. Kendi render hedefini (`CPythonApplication::RENDER_TARGET_ILLUSTRATION`) ve özel kamera kontrollerini kullanarak esnek bir görüntüleme deneyimi sağlar.

---

### `PythonIllustratedManager.cpp` (Resimli Kılavuz/Model Görüntüleyici Yöneticisi Implementasyonu)

Bu dosya, `PythonIllustratedManager.h` dosyasında bildirilen `CPythonIllustratedManager` sınıfının tüm metotlarının C++ implementasyonlarını içerir. Bu sınıf, oyun içi bir model görüntüleyici (örneğin, canavar ansiklopedisi, karakter önizleme) için gerekli tüm işlevleri sağlar.

#### Dosyanın Genel Amaçları

*   **Model Görüntüleme Mantığının Uygulanması**: Modellerin yüklenmesi, seçilmesi, özel bir sahnede (render hedefi) çizilmesi, kamera ile etkileşim (döndürme, yakınlaştırma, kaydırma) ve model üzerinde ekipman (saç, zırh, silah) değişikliklerinin yapılması gibi işlevleri gerçekleştirir.
*   **Kaynak Yönetimi**: Arka plan resmi ve yüklenmiş 3D model örneklerinin (`CInstanceBase`) yönetimi ve temizlenmesi.
*   **Render Döngüsü Entegrasyonu**: Modellerin deformasyonu, güncellenmesi ve çizilmesi için ayrı metotlar sunar; bu metotlar genellikle ana oyun döngüsü veya UI güncelleme döngüsü tarafından uygun zamanlarda çağrılır.

#### C++ Implementasyon Detayları

*   **Başlatma ve Yok Etme (`CPythonIllustratedManager`, `~CPythonIllustratedManager`, `__Initialize`, `Destroy`)**:
    *   Yapıcı metot, üye değişkenleri (işaretçiler NULL, float'lar 0.0f, `m_bShow` false) başlatır ve `m_mapModelInstance` haritasını temizler.
    *   Yıkıcı metot, `Destroy()` çağırır.
    *   `__Initialize()`: Temel sıfırlama işlemlerini yapar.
    *   `Destroy()`: `__Initialize()`'ı çağırır, ayrıca `m_pBackgroundImage` işaretçisini siler (eğer oluşturulmuşsa) ve `m_mapModelInstance` içindeki tüm modelleri temizler (ancak modellerin kendisini silmez, sadece haritayı temizler; modellerin silinmesi `CInstanceBase::Delete` ile yapılmalıdır).

*   **Görsel ve Model Kurulumu**:
    *   `ClearInstances()`: Aktif modeli (`m_pModelInstance`) NULL yapar ve `m_mapModelInstance` haritasını tamamen boşaltır.
    *   `CreateBackground(DWORD dwWidth, DWORD dwHeight)`: Eğer daha önce oluşturulmamışsa, "d:/ymir work/ui/game/monster_card/model_view_bg.sub" yolundan bir arka plan resmi (`CGraphicImageInstance`) yükler ve belirtilen `dwWidth`, `dwHeight` boyutlarına göre ölçekler.
    *   `GetInstancePtr(DWORD dwVnum)`: `m_mapModelInstance` haritasından verilen VNUM'a sahip bir model işaretçisi arar.
    *   `CreateModelInstance(DWORD dwVnum)`: Verilen VNUM için yeni bir `CInstanceBase` (model) oluşturur. Modelin türünü (`CActorInstance::TYPE_PC` veya `CActorInstance::TYPE_OBJECT`) VNUM'un `CRaceData::RACE_MAX_NUM`'dan küçük olup olmamasına göre belirler. Oluşturulan model `m_mapModelInstance` haritasına eklenir. Eğer model zaten varsa bir işlem yapmaz.
    *   `SelectModel(DWORD dwVnum)`: Görüntülenecek modeli VNUM ile seçer. Eğer model yoksa `CreateModelInstance` ile oluşturur. Seçilen modelin animasyonunu bekleme (`CRaceMotionData::NAME_WAIT`) durumuna getirir, LOD ayarlarını yapar, pozisyonunu ve rotasyonunu sıfırlar. Modelin yüksekliğini (`CActorInstance::GetHeight()`) alarak `m_fModelHeight` (kodda `m_fUnk08`) değişkenine atar. Bu yüksekliğe bağlı olarak kamera uzaklık/yakınlaştırma ofsetini (`m_fCamDistanceOffset` - kodda `m_fUnk12`) ve dikey kaydırma ofsetini (`m_fCamVerticalOffset` - kodda `m_fUnk20`) sıfırlar. Yakınlaştırma adımını (`m_fCamZoomStep` - kodda `m_fUnk16`) ve dikey kaydırma adımını (`m_fCamPanStep` - kodda `m_fUnk24`) modelin yüksekliğine orantılı olarak hesaplar. Son olarak, `CCameraManager::ILLUSTRATED_CAMERA` adlı özel kamerayı ayarlar ve hedef yüksekliğini modelin yarısı olarak belirler.

*   **Kamera ve Model Etkileşimleri (`ModelViewReset`, `ModelRotation`, `ModelZoom`, `ModelUpDown`)**:
    *   `ModelViewReset()`: Kamera uzaklık ve dikey ofsetlerini sıfırlar, modelin rotasyonunu ve dünya pozisyonunu sıfırlar.
    *   `ModelRotation(float fRot)`: Modelin Y ekseni etrafındaki dönüşünü ayarlar.
    *   `ModelZoom(bool bZoom)`: `m_fCamDistanceOffset` değerini `m_fCamZoomStep` kullanarak artırır veya azaltır. Yakınlaştırma ve uzaklaştırma belirli sınırlar (`m_fModelHeight`'a bağlı) içinde kalacak şekilde kısıtlanır.
    *   `ModelUpDown(bool bUp)`: `m_fCamVerticalOffset` değerini `m_fCamPanStep` kullanarak artırır veya azaltır. Dikey kaydırma da modelin yüksekliğine bağlı belirli sınırlar içinde tutulur.

*   **Render İşlemleri (`RenderBackground`, `DeformModel`, `RenderModel`)**:
    *   Bu fonksiyonlar sadece `m_bShow` true ise çalışır.
    *   `RenderBackground()`: `CPythonApplication::RENDER_TARGET_ILLUSTRATION` adlı özel bir render hedefine geçer, hedefi temizler ve `m_pBackgroundImage`'ı çizer.
    *   `DeformModel()`: Seçili modelin (`m_pModelInstance`) iskelet animasyonunu günceller (`Deform()`).
    *   `RenderModel()`: Yine `RENDER_TARGET_ILLUSTRATION` hedefine çalışır. Derinlik tamponunu temizler. Mevcut grafik ayarlarını (FOV, en-boy oranı, yakın/uzak kırpma düzlemleri) ve sis durumunu saklar, ardından sisi kapatır. `ILLUSTRATED_CAMERA`'yı aktif eder. Kamera pozisyonunu (`v3Eye`) `-(m_fModelHeight * 8.9f + m_fCamDistanceOffset)` Z değeriyle, kamera hedef pozisyonunu (`v3Target`) `m_fCamVerticalOffset` Y değeriyle (modelin kendi koordinat sisteminde) ayarlar. Özel bir perspektif (FOV 10.0f) ayarlar ve modeli (`m_pModelInstance->Render()`) çizer. Son olarak, saklanan grafik ayarlarını ve sis durumunu geri yükler.

*   **Model Güncelleme ve Ekipman Değişiklikleri (`UpdateModel`, `ChangeMotion`, `SetHair`, `SetArmor`, `SetWeapon`)**:
    *   `UpdateModel()`: Seçili modelin dünya dönüşümlerini (`Transform()`) ve rotasyon işlemlerini (`RotationProcess()`) günceller.
    *   `ChangeMotion()`: Modelin bir sonraki geçerli animasyonuna (`NextValidMotion()`) geçiş yapar.
    *   `SetHair()`, `SetArmor()`, `SetWeapon()`: Modelin ilgili parçalarını (`CInstanceBase` metotları aracılığıyla) ayarlar ve animasyonunu yeniler (`Refresh()`). `SetArmor`, zırhın türüne göre speküler parlaklığı ayarlamaya çalışır ve mevcut saç/silah varsa bunları tekrar uygular.

*   **Görünürlük (`SetShow(bool bShow)`)**: `m_bShow` bayrağını ayarlayarak tüm görüntüleyiciyi aktif veya pasif hale getirir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu sınıf, istemcideki "Resimli Kılavuz", "Karakter Önizleme", "Canavar Kartları" gibi arayüzlerin arka planındaki 3D model görüntüleme motorudur.

*   **Model Önizleme**: Oyuncular, bir canavarın, NPC'nin veya giyilebilir bir eşyanın 3D modelini bu arayüz üzerinden detaylıca inceleyebilir.
*   **İnteraktif Kontroller**: Kullanıcı, modeli döndürebilir, yakınlaştırıp uzaklaştırabilir ve yukarı/aşağı kaydırarak farklı açılardan görebilir.
*   **Ekipman Deneme**: Özellikle karakter modelleri için saç, zırh, silah gibi ekipmanların model üzerinde nasıl durduğu gösterilebilir.
*   **İzole Render Ortamı**: Görüntüleme işlemi, oyunun ana sahnesinden bağımsız, özel bir render hedefi ve kamera ile yapılır, bu da temiz ve odaklanmış bir sunum sağlar.

Bu sınıfın metotları, genellikle bu görüntüleyiciyi içeren UI penceresinin Python veya C++ kodları tarafından, kullanıcı etkileşimlerine (buton tıklamaları, fare hareketleri vb.) yanıt olarak çağrılır.

---

### `PythonIME.h` (Python IME Entegrasyon Başlık Dosyası)

Bu başlık dosyası, `CPythonIME` adlı bir C++ singleton sınıfını tanımlar. Bu sınıf, oyun istemcisinin İşletim Sistemi'nin Giriş Metodu Düzenleyicisi (Input Method Editor - IME) ile etkileşimini yönetmekten sorumludur. IME, özellikle Asya dilleri gibi klavyede doğrudan temsil edilemeyen karakterlerin girilmesi için kullanılır. `CPythonIME`, IME olaylarını işler, IME pencerelerinin (aday listesi, kompozisyon penceresi vb.) konumlandırılmasına yardımcı olur ve metin giriş alanlarıyla IME arasındaki koordinasyonu sağlar. Sınıf, `CSingleton<CPythonIME>`'den miras alarak singleton deseni uygular.

#### Dosyanın Genel Amaçları

*   **`CPythonIME` Sınıf Bildirimi**: IME etkileşimlerini yöneten `CSingleton` C++ sınıfını deklare eder.
*   **Miras Aldığı Sınıflar**:
    *   `IIMEEventSink`: IME olaylarını almak için bir arayüz. `CPythonIME` bu arayüzdeki sanal fonksiyonları (virtual functions) implemente ederek IME olaylarına tepki verir.
    *   `CIME`: EterLib içinde tanımlı olan temel IME işlevselliğini sağlayan sınıf. `CPythonIME`, bu sınıftan miras alarak mevcut IME altyapısını kullanır ve genişletir.
    *   `CSingleton<CPythonIME>`: Bu sınıfın oyun içinde tek bir örneğinin olmasını garanti eder.
*   **Temel Fonksiyonlar**:
    *   `Create(HWND hWnd)`: IME'yi belirtilen pencereye (handle) bağlayarak başlatır.
    *   Metin Düzenleme Fonksiyonları: `MoveLeft()`, `MoveRight()`, `MoveHome()`, `MoveEnd()`, `SetCursorPosition(int iPosition)`, `Delete()`. Bu fonksiyonlar, IME'nin aktif olduğu metin giriş alanındaki imleç pozisyonunu ve içeriği değiştirmek için kullanılır.
*   **Korunan Sanal Olay İşleyici Fonksiyonları (Overrides)**:
    *   `OnTab()`, `OnReturn()`, `OnEscape()`: Tab, Enter ve Escape tuşlarına basıldığında tetiklenir.
    *   `OnWM_CHAR(WPARAM wParam, LPARAM lParam)`: Standart karakter girişi olaylarını işler.
    *   `OnUpdate()`: IME durumunun güncellenmesi gerektiğinde çağrılır.
    *   `OnChangeCodePage()`: IME için kod sayfası değiştiğinde tetiklenir.
    *   `OnOpenCandidateList()`, `OnCloseCandidateList()`: Aday karakter listesi penceresi açıldığında veya kapandığında çağrılır.
    *   `OnOpenReadingWnd()`, `OnCloseReadingWnd()`: Okuma/Kompozisyon penceresi açıldığında veya kapandığında çağrılır.

#### C++ ve Python Arayüzü ile İlişkisi

`CPythonIME` sınıfı, "Python" önekine sahip olsa da doğrudan Python'a bir modül olarak sunulmaz. Bunun yerine, C++ tabanlı UI bileşenleri (örneğin, `CEditLine` veya IME destekli özel edit kontrolleri) bu singleton sınıfı kullanarak IME işlevselliğini yönetir. Python tabanlı UI scriptleri IME ile etkileşimde bulunuyorsa (örneğin, IME'yi programatik olarak açıp kapatmak, giriş metnini yönetmek), bu genellikle `CPythonIME`'yi kullanan C++ UI nesnelerinin Python'a sarmalanmış (wrapped) arayüzleri üzerinden dolaylı olarak gerçekleşir. Temel amaç, IME olaylarını ve yönetimini `AbstractApplication` üzerinden merkezi bir yapıya yönlendirmektir.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Çok Dilli Metin Girişi**: Oyuncuların Çince, Japonca, Korece gibi IME gerektiren dillerde karakter girişi yapmalarını sağlar.
*   **Sohbet ve İsimlendirme**: Oyun içi sohbet, karakter adı oluşturma, lonca adı belirleme gibi metin girişi gerektiren her yerde IME desteği sunar.
*   **Metin Alanları ile Entegrasyon**: Oyundaki tüm metin giriş kutularının işletim sistemi IME'si ile düzgün çalışmasını sağlar. Kompozisyon dizelerinin (henüz tamamlanmamış karakterler) ve aday listelerinin doğru yerde görüntülenmesine yardımcı olur.
*   **Olay Yönlendirme**: IME'den gelen çeşitli olayları (`OnTab`, `OnReturn` vb.) yakalar ve bunları `IAbstractApplication` singleton'ı aracılığıyla oyunun daha üst seviye mantığına iletir.

Bu başlık dosyası, istemcinin farklı dillerdeki kullanıcılar için erişilebilir olmasını sağlayan temel bir bileşen olan IME entegrasyonunun C++ tarafındaki arayüzünü ve olay işleme mekanizmalarını tanımlar.

---

### `PythonIME.cpp` (Python IME Entegrasyon Implementasyonu)

Bu dosya, `PythonIME.h` başlık dosyasında bildirilen `CPythonIME` singleton C++ sınıfının metotlarının tam implementasyonunu içerir. Temel olarak, `CIME` sınıfından miras aldığı işlevleri kullanarak ve IME olaylarını `IAbstractApplication`'a yönlendirerek çalışır.

#### Dosyanın Genel Amaçları

*   `CPythonIME` sınıfının yapıcı ve yıkıcı metotlarını tanımlamak.
*   Başlık dosyasında deklare edilen metin düzenleme ve IME olay işleyici fonksiyonlarının gövdelerini sağlamak.
*   IME olaylarını `CIME` taban sınıfından alıp, oyunun ana uygulama katmanına (`IAbstractApplication`) iletmek.

#### C++ Implementasyon Detayları

*   **Yapıcı (`CPythonIME::CPythonIME()`)**:
    *   `CIME()` taban sınıfının yapıcısını çağırır.
    *   `ms_pEvent = this;` satırı ile kendisini `CIME` sınıfının olay dinleyicisi (event sink) olarak ayarlar. Bu sayede `CIME` içinde oluşan IME olayları, `CPythonIME`'nin override ettiği sanal fonksiyonlara yönlendirilir.

*   **Yıkıcı (`CPythonIME::~CPythonIME()`)**:
    *   `Tracen("PythonIME Clear");` ile bir izleme mesajı yazar. Temizlik işlemleri genellikle taban sınıfın yıkıcısında veya özel `Destroy` metotlarında yapılır.

*   **`Create(HWND hWnd)`**:
    *   `Initialize(hWnd);` çağrısı yaparak `CIME` taban sınıfının başlatma rutinini çalıştırır ve IME'yi belirtilen pencereye bağlar.

*   **Metin Düzenleme Fonksiyonları**:
    *   `MoveLeft()`: `DecCurPos();` (imleci sola kaydır - `CIME` fonksiyonu).
    *   `MoveRight()`: `IncCurPos();` (imleci sağa kaydır - `CIME` fonksiyonu).
    *   `MoveHome()`: `ms_curpos = 0;` (imleci başa al).
    *   `MoveEnd()`: `ms_curpos = ms_lastpos;` (imleci sona al).
    *   `SetCursorPosition(int iPosition)`: `SetCurPos(iPosition);` (imleci belirtilen pozisyona ayarla - `CIME` fonksiyonu).
    *   `Delete()`: `DelCurPos();` (imlecin bulunduğu pozisyondaki karakteri sil - `CIME` fonksiyonu).
    Bu fonksiyonlar, `CIME` sınıfının temel imleç ve metin manipülasyon fonksiyonları için bir sarmalayıcı (wrapper) görevi görür.

*   **IME Olay İşleyicileri ve `IAbstractApplication`'a Yönlendirme**:
    *   `OnUpdate()`: `IAbstractApplication::GetSingleton().RunIMEUpdate();`
    *   `OnTab()`: `IAbstractApplication::GetSingleton().RunIMETabEvent();`
    *   `OnReturn()`: `IAbstractApplication::GetSingleton().RunIMEReturnEvent();`
    *   `OnEscape()`: Bu fonksiyonun içeriği yorum satırı haline getirilmiştir (`//IAbstractApplication::GetSingleton().RunIMEEscapeEvent();`). Yani Escape tuşu basıldığında `AbstractApplication`'a bir olay gönderilmez.
    *   `OnChangeCodePage()`: `IAbstractApplication::GetSingleton().RunIMEChangeCodePage();`
    *   `OnOpenCandidateList()`: `IAbstractApplication::GetSingleton().RunIMEOpenCandidateListEvent();`
    *   `OnCloseCandidateList()`: `IAbstractApplication::GetSingleton().RunIMECloseCandidateListEvent();`
    *   `OnOpenReadingWnd()`: `IAbstractApplication::GetSingleton().RunIMEOpenReadingWndEvent();`
    *   `OnCloseReadingWnd()`: `IAbstractApplication::GetSingleton().RunIMECloseReadingWndEvent();`
    Bu fonksiyonların tamamı, ilgili IME olayını aldıklarında `IAbstractApplication` singleton'ı üzerinden oyunun ana uygulama katmanındaki ilgili bir fonksiyonu çağırır. Bu, IME olaylarının merkezi bir yerden yönetilmesini sağlar.

*   **`OnWM_CHAR(WPARAM wParam, LPARAM lParam)`**:
    *   Gelen karakteri (`wParam`) alır.
    *   `VK_RETURN`, `VK_TAB`, `VK_ESCAPE` gibi özel tuşları kontrol eder:
        *   `VK_RETURN`: `OnReturn()` çağrılır.
        *   `VK_TAB`: Eğer `ms_bCaptureInput` (girişi yakala bayrağı, muhtemelen `CIME`'den miras alınmış) `true` ise `OnTab()` çağrılır.
        *   `VK_ESCAPE`: Eğer `ms_bCaptureInput` `true` ise `OnEscape()` çağrılır.
    *   Eğer bu özel tuşlardan biri değilse `false` döner, bu da karakterin başka bir şekilde (muhtemelen `CIME` taban sınıfı tarafından) işlenebileceği anlamına gelir.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosyadaki implementasyon, `CPythonIME` sınıfının `CIME`'den gelen düşük seviyeli IME olaylarını nasıl yakaladığını ve bunları daha yüksek seviyeli bir işleme için `IAbstractApplication`'a nasıl aktardığını gösterir.

*   **Olay Köprüsü**: `CPythonIME`, işletim sistemi seviyesindeki IME olayları ile oyunun kendi olay döngüsü ve UI yönetimi arasında bir köprü görevi görür.
*   **Merkezi Olay Yönetimi**: Tüm IME olaylarının `IAbstractApplication` üzerinden geçirilmesi, bu olaylara verilecek tepkilerin (örneğin, UI güncellemesi, Python scriptlerine bildirim) merkezi bir noktada kontrol edilmesini sağlar.
*   **Metin Giriş Alanlarının Kontrolü**: Sohbet kutuları, isim giriş alanları gibi UI elemanları, metin girişi için `CPythonIME`'nin sağladığı temel düzenleme fonksiyonlarını (imleç hareketleri, silme) ve olaylarını kullanır.

`CPythonIME.cpp`, `UserInterface` modülünün işletim sistemi IME altyapısıyla etkileşim kurmasını ve çok dilli metin girişini desteklemesini sağlayan kritik bir bileşenin C++ implementasyonudur.

---

### `PythonIMEModule.cpp` (Python `ime` Modülü)

Bu dosya, `CPythonIME` C++ singleton sınıfının işlevselliğini Python betiklerine açan `ime` adlı bir Python modülünü tanımlar ve başlatır. Bu sayede Python tabanlı UI scriptleri, IME'yi (Input Method Editor) kontrol edebilir, metin girişini yönetebilir ve IME ile ilgili bilgileri alabilir.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma**: `ime` adında bir Python modülü oluşturur.
*   **C++ Fonksiyonlarını Python'a Sarmalama**: `CPythonIME` sınıfının public metotlarını çağıran ve sonuçları Python veri türlerine dönüştüren C fonksiyonları (Python sarmalayıcıları) tanımlar.
*   **Modül Başlatma**: Python yorumlayıcısı tarafından `ime` modülü yüklendiğinde çağrılacak olan `initime()` fonksiyonunu sağlar.

#### C++ ve Python Arayüzü

Bu dosya, `CPythonIME` C++ sınıfı ile Python scriptleri arasında doğrudan bir köprü görevi görür.

*   **`initime()` Fonksiyonu**:
    *   Python modülü yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyonların isimlerini (örneğin, "Enable", "SetText", "GetText") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını (örneğin, `imeEnable`, `imeSetText`, `imeGetText`) tanımlar.
    *   `Py_InitModule("ime", s_methods);` komutu ile `ime` modülünü Python'a kaydeder.

*   **Python Fonksiyon Sarmalayıcıları (`ime*` önekli fonksiyonlar)**:
    Bu fonksiyonlar, Python'dan çağrıldığında `CPythonIME::Instance()` üzerinden ilgili C++ metodunu çağırır, Python'dan gelen argümanları C++ türlerine dönüştürür ve C++ metotlarından dönen sonuçları Python veri türlerine çevirir.

    *   **IME Kontrolü**:
        *   `imeEnable()`: `CPythonIME::Instance().Initialize(CPythonApplication::Instance().GetWindowHandle())` çağırır. IME'yi başlatır ve oyun penceresine bağlar.
        *   `imeDisable()`: `CPythonIME::Instance().Uninitialize()` çağırır. IME'yi devre dışı bırakır.
        *   `imeEnableCaptureInput()`: `CPythonIME::Instance().EnableCaptureInput()` çağırır. IME'nin klavye girişini yakalamasını aktif eder.
        *   `imeDisableCaptureInput()`: `CPythonIME::Instance().DisableCaptureInput()` çağırır. IME'nin klavye girişini yakalamasını pasifize eder.
        *   `imeEnableIME()`: `CPythonIME::Instance().EnableIME()` çağırır. İşletim sistemi seviyesinde IME'yi etkinleştirir.
        *   `imeDisableIME()`: `CPythonIME::Instance().DisableIME()` çağırır. İşletim sistemi seviyesinde IME'yi devre dışı bırakır.

    *   **Metin ve Uzunluk Ayarları**:
        *   `imeSetMax(int iMax)`: `CPythonIME::Instance().SetMax(iMax)` çağırır. Giriş yapılabilecek maksimum karakter sayısını (genel) ayarlar.
        *   `imeSetUserMax(int iMax)`: `CPythonIME::Instance().SetUserMax(iMax)` çağırır. Kullanıcı tarafından belirlenen maksimum karakter sayısını ayarlar (muhtemelen farklı bir kontrol için).
        *   `imeSetText(char* szText)`: `CPythonIME::Instance().SetText(szText, strlen(szText))` çağırır. IME'nin mevcut metnini ayarlar.
        *   `imeGetText(int bCodePage = 0)`: `CPythonIME::Instance().GetText(strText, bCodePage ? true : false)` çağırır. Mevcut metni alır. `bCodePage` true ise kod sayfasına göre çevrim yapılabilir.

    *   **IME Bilgileri Alma**:
        *   `imeGetCodePage()`: `CPythonIME::Instance().GetCodePage()` çağırır. Aktif kod sayfasını döndürür.
        *   `imeGetCandidateCount()`: `CPythonIME::Instance().GetCandidatePageCount()` çağırır. Aday listesindeki sayfa sayısını (veya toplam aday sayısını, isimlendirme biraz kafa karıştırıcı olabilir) döndürür.
        *   `imeGetCandidate(int index)`: `CPythonIME::Instance().GetCandidate(index, strText)` çağırır. Belirtilen indeksteki aday karakteri/dizeyi ve uzunluğunu döndürür.
        *   `imeGetCandidateSelection()`: `CPythonIME::Instance().GetCandidateSelection()` çağırır. Aday listesinde o anda seçili olan adayın indeksini döndürür.
        *   `imeGetReading()`: `CPythonIME::Instance().GetReading(strText)` çağırır. Okuma/Kompozisyon dizesini (henüz tamamlanmamış giriş) alır.
        *   `imeGetReadingError()`: `CPythonIME::Instance().GetReadingError()` çağırır. Okuma dizesiyle ilgili bir hata kodu varsa onu döndürür.
        *   `imeGetInputMode()`: `CPythonIME::Instance().GetInputMode()` çağırır. Mevcut giriş modunu (örn: Alfanümerik, Hiragana, Katakana vb.) döndürür.

    *   **Giriş Modu Ayarları**:
        *   `imeSetInputMode(int mode)`: `CPythonIME::Instance().SetInputMode(mode)` çağırır. Giriş modunu ayarlar.
        *   `imeSetNumberMode()`: `CPythonIME::Instance().SetNumberMode()` çağırır. IME'yi sayı giriş moduna geçirir.
        *   `imeSetStringMode()`: `CPythonIME::Instance().SetStringMode()` çağırır. IME'yi standart karakter/dize giriş moduna geçirir.

    *   **Özel Tuş Yönetimi**:
        *   `imeAddExceptKey(int key)`: `CPythonIME::Instance().AddExceptKey(key)` çağırır. Belirli bir tuşun IME tarafından işlenmeyip doğrudan oyuna iletilmesini sağlar.
        *   `imeClearExceptKey()`: `CPythonIME::Instance().ClearExceptKey()` çağırır. Tüm istisna tuş tanımlarını temizler.

    *   **İmleç Kontrolü (CPythonIME üzerinden dolaylı)**:
        *   `imeMoveLeft()`: `CPythonIME::Instance().MoveLeft()`
        *   `imeMoveRight()`: `CPythonIME::Instance().MoveRight()`
        *   `imeMoveHome()`: `CPythonIME::Instance().MoveHome()`
        *   `imeMoveEnd()`: `CPythonIME::Instance().MoveEnd()`
        *   `imeSetCursorPosition(int iPosition)`: `CPythonIME::Instance().SetCursorPosition(iPosition)`
        *   `imeDelete()`: `CPythonIME::Instance().Delete()`

    *   **Pano ve Yapıştırma İşlemleri**:
        *   `imePasteTextFromClipBoard()`: `CPythonIME::Instance().PasteTextFromClipBoard()` çağırır. Panodan metin yapıştırır.
        *   `imeEnablePaste(int iFlag)`: `CPythonIME::Instance().EnablePaste(iFlag ? true : false)` çağırır. Metin yapıştırma özelliğini aktif/pasif eder.
        *   `imePasteString(char* szText)`: `CPythonIME::Instance().PasteString(szText)` çağırır. Verilen metni doğrudan yapıştırır (sanki kullanıcı yazmış gibi).
        *   `imePasteBackspace()`: `CPythonIME::Instance().WMChar(NULL, WM_CHAR, 0x08, NULL)` çağırır. Backspace tuşuna basılmasını simüle eder.
        *   `imePasteReturn()`: `CPythonIME::Instance().WMChar(NULL, WM_CHAR, 0x0D, NULL)` çağırır. Enter tuşuna basılmasını simüle eder.

#### Oyunda Kullanım Amacı ve Senaryoları

Python'daki `ime` modülü, oyunun UI scriptlerinin (genellikle Python ile yazılır) metin giriş kutuları ve IME yönetimiyle doğrudan etkileşim kurmasını sağlar.

*   **Metin Giriş Kutuları (EditLine, ChatLineEdit vb.)**: Python tabanlı UI elemanları, kullanıcı metin girdiğinde veya IME ile etkileşimde bulunduğunda bu modül fonksiyonlarını çağırarak `CPythonIME`'yi kontrol eder. Örneğin, bir giriş alanına tıklanıldığında `ime.EnableCaptureInput()` çağrılabilir, maksimum karakter sayısı `ime.SetMax()` ile ayarlanabilir veya mevcut metin `ime.GetText()` ile okunabilir.
*   **Programatik IME Kontrolü**: Belirli UI durumlarında IME'nin davranışını değiştirmek için kullanılabilir. Örneğin, bir şifre alanında IME'yi otomatik olarak İngilizce moda geçirmek veya sayısal bir alanda `ime.SetNumberMode()` çağırmak.
*   **Özel Klavye Kısayolları**: `ime.AddExceptKey()` ile bazı tuşların IME tarafından değil, doğrudan oyunun kısayol mantığı tarafından işlenmesi sağlanabilir.
*   **Pano İşlemleri**: Python scriptleri üzerinden panodan metin yapıştırma veya programatik olarak metin "yazdırma" (örneğin, bir makro için) işlemleri için `ime.PasteTextFromClipBoard()` veya `ime.PasteString()` kullanılabilir.

Bu modül, `CPythonIME` sınıfının sunduğu düşük seviyeli IME kontrolünü, Python scriptlerinin daha kolay kullanabileceği bir arayüze dönüştürür ve oyunun UI'sının çok dilli metin girişini esnek bir şekilde yönetmesine olanak tanır.

---

### `PythonItem.h` (Python Eşya Yönetimi Başlık Dosyası)

Bu başlık dosyası, `CPythonItem` adlı bir C++ singleton sınıfını tanımlar. Bu sınıf, istemci tarafındaki tüm eşya (item) yönetimiyle ilgili temel işlevlerden sorumludur. Özellikle yere düşen eşyaların (ground items) oluşturulması, güncellenmesi, render edilmesi, silinmesi, etkileşimleri (toplama, üzerine gelme) ve eşyalarla ilişkili seslerin yönetimi bu sınıfın ana görevlerindendir. Ayrıca, Python'a sunulacak `item` modülü için C++ tarafındaki altyapıyı sağlar.

#### Dosyanın Genel Amaçları

*   **`CPythonItem` Sınıf Bildirimi**: Eşya yönetimiyle ilgili tüm istemci tarafı operasyonlarını merkezi olarak yöneten `CSingleton<CPythonItem>` sınıfını deklare eder.
*   **`SGroundItemInstance` Yapısı**: Yere düşmüş bir eşyanın örneğini temsil eden iç içe geçmiş bir yapı (`struct`) tanımlar. Bu yapı şunları içerir:
    *   `dwVirtualNumber`: Eşyanın sanal numarası (VNUM).
    *   `v3EndPosition`: Eşyanın yerdeki son konumu.
    *   `v3RotationAxis`, `qEnd`: Eşyanın düşme animasyonu ve yerdeki son rotasyonu için kullanılan değişkenler.
    *   `v3Center`: Eşya modelinin merkez noktası.
    *   `ThingInstance`: Eşyanın 3D modelini temsil eden `CGraphicThingInstance`.
    *   `dwStartTime`, `dwEndTime`: Düşme animasyonunun başlangıç ve bitiş zamanları.
    *   `eDropSoundType`: Düşerken çalınacak sesin türü.
    *   `bAnimEnded`: Düşme animasyonunun bitip bitmediğini belirten bayrak.
    *   `dwEffectInstanceIndex`: Eşyaya bağlı efektin (örneğin, parlama) ID'si.
    *   `stOwnership`: Eşyanın sahiplik bilgisi (kimin düşürdüğü veya kime ait olduğu).
    *   `tDestroyTime` (`ENABLE_DROP_DESTROY_TIME` ile): Eşyanın kaybolma zamanı.
    *   `ms_astDropSoundFileName`: Farklı eşya türleri için düşme sesi dosyalarının statik bir dizisi.
    *   Metotlar: `Update()` (animasyonu günceller), `Clear()` (örneği temizler), `__PlayDropSound()` (düşme sesini çalar).
*   **Typedef ve Sabitler**:
    *   `TGroundItemInstanceMap`: `std::map<DWORD, TGroundItemInstance*>` (Sanal ID -> Yerdeki Eşya Örneği İşaretçisi).
    *   `INVALID_ID`, `VNUM_MONEY`: Geçersiz ID ve para VNUM'u gibi sabitler.
    *   `USESOUND_NUM`, `DROPSOUND_NUM`: Kullanım ve düşme ses türlerinin maksimum sayısı için enum'lar ve ilgili ses türleri (örn: `USESOUND_WEAPON`, `DROPSOUND_ARMOR`).
*   **Temel Fonksiyon Prototipleri (Public)**:
    *   **Kurulum ve Temizleme**: `Destroy()`, `Create()`.
    *   **Ses Yönetimi**: `PlayUseSound()`, `PlayDropSound()`, `PlayUsePotionSound()`, `SetUseSoundFileName()`, `SetDropSoundFileName()`.
    *   **Bilgi Alma**: `GetInfo()` (hata ayıklama bilgisi), `GetOwnership()`, `GetGroundItemPosition()`, `GetPickedItemID()`, `GetVirtualNumberOfGroundItem()`, `GetItemNameByVnum` (`ENABLE_MOVE_COSTUME_ATTR` ile).
    *   **Eşya Yönetimi (Yerdeki)**: `DeleteAllItems()`, `Render()`, `Update()`, `CreateItem()`, `DeleteItem()`, `SetOwnership()`.
    *   `SetDestroyTime` (`ENABLE_DROP_DESTROY_TIME` ile).
    *   **Yakındaki Eşyaları Bulma**: `GetCloseItem()`, `GetAllCloseItems()`, `GetCloseMoney()`.
    *   **"NoGradeName" Veri Yönetimi**: `BuildNoGradeNameData()`, `GetNoGradeNameDataCount()`, `GetNoGradeNameDataPtr()` (muhtemelen belirli türdeki eşyaları özel bir liste için filtreler).
    *   **Change Look Sistemi ile İlgili Fonksiyonlar** (`ENABLE_CHANGE_LOOK_SYSTEM` ile): `CanAddChangeLookItem()`, `CanAddChangeLookFreeItem()`, `IsChangeLookClearScrollItem()`.
    *   **Loot Filter Sistemi ile İlgili Fonksiyonlar** (`ENABLE_LOOTING_SYSTEM` ile): `IsLootFilteredItem()`, `InsertLootFilteredItem()`, `EraseLootFilteredItem()`, `ClearLootFilteredItems()`.
    *   `CanPickGroundItem` (`ENABLE_CLOSE_ITEM_TEXT_TAIL_COLOR` ile).
*   **Korunan (Protected) Yardımcı Fonksiyonlar**:
    *   `__Pick()`: Fare ile bir eşyanın seçilip seçilmediğini kontrol eder.
    *   `__GetUseSoundType()`, `__GetDropSoundType()`: Bir `CItemData`'ya göre uygun kullanım veya düşme sesi türünü belirler.
*   **Üye Değişkenler**:
    *   `m_GroundItemInstanceMap`: Aktif yerdeki eşya örneklerini tutan harita.
    *   `m_GroundItemInstancePool`: `SGroundItemInstance` nesneleri için bir dinamik bellek havuzu.
    *   `m_dwDropItemEffectID`: Yere düşen eşyalar için varsayılan efekt ID'si.
    *   `m_dwPickedItemID`: Fare ile üzerine gelinen veya seçilen eşyanın ID'si.
    *   `m_astUseSoundFileName`: Farklı eşya türleri için kullanım sesi dosyalarının dizisi.
    *   `m_NoGradeNameItemData`: "NoGradeName" (muhtemelen seviyesiz isimler) için `CItemData` işaretçilerinin bir vektörü.
    *   `m_LootFilteredItemsSet` (`ENABLE_LOOTING_SYSTEM` ile): Filtrelenmiş eşyaların ID'lerini tutan küme.

#### C++ ve Python Arayüzü ile İlişkisi

`CPythonItem` sınıfı, Python'a doğrudan sunulmaz ancak `PythonItemModule.cpp` dosyasında tanımlanan `item` adlı Python modülünün C++ tarafındaki temelini oluşturur. Python modülündeki fonksiyonlar (örneğin, `item.Pick()`, `item.GetItemNameByVnum()`) `CPythonItem::Instance()` üzerinden bu sınıfın public metotlarını çağırarak eşya verilerine ve işlevlerine erişir.

#### Oyunda Kullanım Amacı ve Senaryoları

*   **Yere Düşen Eşyaların Görselleştirilmesi**: Canavarlar öldüğünde veya oyuncular eşya düşürdüğünde, bu sınıf aracılığıyla eşyaların 3D modelleri, animasyonları (düşme efekti) ve üzerlerindeki isim etiketleri (`CPythonTextTail` ile entegre) yönetilir.
*   **Eşya Toplama Etkileşimi**: Oyuncu bir eşyanın yakınına geldiğinde veya üzerine tıkladığında, bu sınıf seçilen eşyanın ID'sini belirler ve toplama isteğinin sunucuya gönderilmesine yardımcı olur.
*   **Eşya Ses Efektleri**: Eşyalar kullanıldığında (örn: iksir içme, silah kuşanma) veya yere düştüğünde uygun ses efektlerinin çalınmasını sağlar.
*   **Eşya Bilgilerinin Sunulması**: UI scriptlerinin, eşya VNUM'undan yola çıkarak eşya adı, tipi, alt tipi gibi bilgilere erişmesi için gerekli fonksiyonları (dolaylı olarak Python modülü üzerinden) sağlar.
*   **Sistem Entegrasyonları**: Grafik ayarları (`ENABLE_GRAPHIC_ON_OFF`), eşya düşürme yenilemeleri (`ENABLE_ITEM_DROP_RENEWAL`), eşya yok olma süresi (`ENABLE_DROP_DESTROY_TIME`), görünüm değiştirme (`ENABLE_CHANGE_LOOK_SYSTEM`), yağma filtresi (`ENABLE_LOOTING_SYSTEM`) gibi çeşitli oyun sistemleriyle entegre çalışır.

Bu başlık dosyası, istemcinin eşya sistemiyle ilgili görsel ve işitsel tüm temel etkileşimleri yöneten merkezi bir yapıyı tanımlar.

---

### `PythonItem.cpp` (Python Eşya Yönetimi Implementasyonu)

Bu dosya, `PythonItem.h`'de bildirilen `CPythonItem` singleton sınıfının ve onun iç içe yapısı `CPythonItem::TGroundItemInstance`'ın tüm metotlarının C++ implementasyonlarını içerir. Yerdeki eşyaların yaşam döngüsü (oluşturma, animasyon, güncelleme, render, silme), ses yönetimi ve çeşitli eşya etkileşimleri bu dosyada kodlanmıştır.

#### Dosyanın Genel Amaçları

*   **Yerdeki Eşya Yönetimi**: Yere düşen eşyaların (`TGroundItemInstance`) oluşturulması, saklanması, güncellenmesi, çizilmesi ve silinmesi.
*   **Düşme Animasyonu**: Eşyaların yere düşerkenki fiziksel hareketlerini ve rotasyonlarını simüle eden animasyon mantığı.
*   **Ses Efektleri**: Eşya türüne göre kullanım ve düşme seslerini çalma.
*   **Etkileşim (Picking)**: Fare ile yerdeki bir eşyanın üzerine gelindiğinde veya tıklandığında hangi eşyanın hedeflendiğini belirleme.
*   **Veri Yönetimi**: Eşya sahipliği, yok olma süresi gibi bilgilerin yönetimi.
*   **Diğer Sistemlerle Entegrasyon**: `CPythonBackground` (zemin yüksekliği), `CEffectManager` (eşya efektleri), `CPythonTextTail` (eşya isim etiketleri), `CItemManager` (eşya temel verileri), `CSoundManager` (ses çalma) gibi diğer modüllerle etkileşim.

#### `CPythonItem::TGroundItemInstance` Implementasyon Detayları

*   **`Clear()`**: Eşya örneğini temizler. Sahiplik bilgisini sıfırlar, `ThingInstance`'ı temizler ve bağlı efekt örneğini `CEffectManager` üzerinden yok eder.
*   **`__PlayDropSound(DWORD eItemType, const D3DXVECTOR3& c_rv3Pos)`**: Verilen eşya türüne ve pozisyona göre uygun düşme sesini `CSoundManager` aracılığıyla 3D olarak çalar. `ms_astDropSoundFileName` statik dizisini kullanır.
*   **`Update()`**: Eşyanın düşme animasyonunu yönetir.
    *   Eğer animasyon bitmişse (`bAnimEnded` true ise) bir şey yapmaz.
    *   Animasyon süresi (`dwEndTime`) dolmuşsa, eşyayı son pozisyonuna ve rotasyonuna (`qEnd`) ayarlar. Modelin merkez noktasına (`v3Center`) göre pozisyon ayarlaması yapar ve `__PlayDropSound` ile düşme sesini çalar. `bAnimEnded`'ı `true` yapar.
    *   Animasyon devam ediyorsa:
        *   Geçen süreye (`CTimer::Instance().GetCurrentMillisecond() - dwStartTime`) ve toplam süreye (`dwEndTime - dwStartTime`) göre bir ilerleme oranı (`rate`) hesaplar.
        *   Pozisyonu interpolasyon ve bir parabolik düşme eğrisi ( `100 - 100 * rate * (3 * rate - 2)` ) kullanarak günceller.
        *   Rotasyonu, `v3RotationAxis` etrafında, kalan süreye ve ilerleme oranına bağlı bir açıyla (`etime * 0.03f * (-1 + rate * (3 * rate - 2))`) `qEnd`'e ekleyerek günceller (Quaternion Slerp veya benzeri bir mantık).
        *   Modelin merkezine göre pozisyon ayarlaması yapar.
    *   Son olarak `ThingInstance.Transform()` ve `ThingInstance.Deform()` çağırır. Animasyonun devam edip etmediğini döndürür.

#### `CPythonItem` Sınıfı Implementasyon Detayları

*   **Yapıcı (`CPythonItem()`)**:
    *   Bellek havuzuna (`m_GroundItemInstancePool`) isim atar ("CDynamicPool<TGroundItemInstance>").
    *   `m_dwPickedItemID`'yi `INVALID_ID` olarak başlatır.

*   **Yıkıcı (`CPythonItem::~CPythonItem()`)**:
    *   `m_GroundItemInstanceMap`'in boş olduğunu varsayar (`assert`). Normalde `Destroy()` içinde temizlenir.

*   **`Create()`**: `CEffectManager` üzerinden "d:/ymir work/effect/etc/dropitem/dropitem.mse" efektini `m_dwDropItemEffectID`'ye kaydeder.
*   **`Destroy()`**: `DeleteAllItems()` çağırarak tüm yerdeki eşyaları ve `m_GroundItemInstancePool`'u temizler.

*   **Ses Yönetimi**:
    *   `SetUseSoundFileName()`, `SetDropSoundFileName()`: İlgili ses türü için dosya adını `m_astUseSoundFileName` veya `SGroundItemInstance::ms_astDropSoundFileName` dizilerinde saklar.
    *   `PlayUseSound(DWORD dwItemID)`, `PlayDropSound(DWORD dwItemID)`: Verilen eşya ID'si için `CItemManager`'dan `CItemData` alır, `__GetUseSoundType` veya `__GetDropSoundType` ile ses türünü belirler ve ilgili 2D sesi `CSoundManager` ile çalar.
    *   `PlayUsePotionSound()`: Doğrudan iksir kullanım sesini çalar.
    *   `__GetUseSoundType()`, `__GetDropSoundType()`: `CItemData`'nın türüne (örn: `ITEM_TYPE_WEAPON`, `ITEM_TYPE_ARMOR`) ve alt türüne göre uygun ses enum değerini döndürür.

*   **Eşya Oluşturma (`CreateItem`)**:
    *   `CItemManager`'dan `dwVirtualNumber` ile `CItemData` alır.
    *   `m_GroundItemInstancePool`'dan yeni bir `TGroundItemInstance` alır.
    *   Eğer `bDrop` true ise (yere düşme animasyonuyla):
        *   Zemin yüksekliğini `CPythonBackground::Instance().GetHeight()` ile alır.
        *   Bazı silah türleri için (`WEAPON_SWORD`, `WEAPON_ARROW`) yere saplanma (`bStabGround`) durumu ayarlanabilir (ancak kodda `bStabGround = false;` olarak sabitlenmiş).
        *   `bAnimEnded`'ı `false` yapar.
    *   Eğer `bDrop` false ise (direkt yerde belirme): `bAnimEnded`'ı `true` yapar.
    *   `CEffectManager` ile `m_dwDropItemEffectID` kullanarak bir efekt oluşturur ve ID'sini saklar.
    *   `__GetDropSoundType` ile düşme sesi türünü belirler.
    *   Zemin normalini `CPythonBackground::Instance().GetNormal()` ile alır.
    *   `ThingInstance`'ı ayarlar: model ekler, pozisyonu ayarlar.
    *   Eğer `bDrop` true ise:
        *   Eşyanın sınırlayıcı kutusunu (`GetBoundBox`) alarak merkezini (`v3Center`) hesaplar.
        *   Eşyanın boyutlarına göre rastgele bir son rotasyon (`qEnd`) ve bir rotasyon ekseni (`v3RotationAxis`) belirler. Zemin normalini de dikkate alarak rotasyonu ayarlar.
        *   Animasyon için `dwStartTime` ve `dwEndTime` (300 milisaniye sonra) ayarlar.
    *   `ENABLE_GRAPHIC_ON_OFF` tanımlıysa, sistem ayarlarına göre eşya modelini ve efektini gizleyip/gösterir.
    *   Oluşturulan `pGroundItemInstance`'ı `m_GroundItemInstanceMap`'e ekler.
    *   `CPythonTextTail::Instance().RegisterItemTextTail()` ile eşya için isim etiketi oluşturur. `ENABLE_ITEM_DROP_RENEWAL` tanımlıysa, eşya adı polimorf veya beceri kitabı gibi özel durumlara göre farklı oluşturulabilir ve efsun varlığına göre etiket farklı olabilir.

*   **Eşya Silme**:
    *   `DeleteItem(DWORD dwVirtualID)`: Verilen ID'deki eşyayı `m_GroundItemInstanceMap`'ten bulur, `Clear()` ile temizler, `m_GroundItemInstancePool`'a geri verir ve haritadan siler. `CPythonTextTail::Instance().DeleteItemTextTail()` ile isim etiketini de siler. `ENABLE_LOOTING_SYSTEM` tanımlıysa filtreden de çıkarır.
    *   `DeleteAllItems()`: `m_GroundItemInstanceMap`'teki tüm eşyaları siler.

*   **Güncelleme ve Render**:
    *   `Update(const POINT& c_rkPtMouse)`: `m_GroundItemInstanceMap`'teki tüm eşyaların `Update()` metodunu çağırır (animasyonlarını günceller). `ENABLE_DROP_DESTROY_TIME` tanımlıysa, periyodik olarak isim etiketlerindeki yok olma süresini günceller. Son olarak `__Pick(c_rkPtMouse)` ile fare altındaki eşyayı belirler ve `m_dwPickedItemID`'ye atar.
    *   `Render()`: `m_GroundItemInstanceMap`'teki tüm eşyaların `ThingInstance`'larını render eder. `ENABLE_GRAPHIC_ON_OFF` tanımlıysa, sistem ayarlarına göre eşya modelini ve efektini gizleyip/gösterir.

*   **Diğer Fonksiyonlar**:
    *   `SetOwnership()`, `GetOwnership()`: Eşya sahiplik bilgisini ayarlar ve alır, ayrıca isim etiketindeki sahiplik bilgisini günceller.
    *   `SetDestroyTime()` (`ENABLE_DROP_DESTROY_TIME` ile): Eşyanın yok olma süresini ayarlar.
    *   `GetCloseMoney()`, `GetCloseItem()`, `GetAllCloseItems()`: Belirli bir pozisyona yakın olan paraları veya diğer eşyaları (sahiplik ve yağma filtresi kontrolüyle) bulur.
    *   `GetGroundItemPosition()`: Yerdeki bir eşyanın pozisyonunu döndürür.
    *   `__Pick()`: Fare ışınıyla kesişen ilk `ThingInstance`'ı veya `CPythonTextTail`'i bulur ve ID'sini döndürür.
    *   `GetPickedItemID()`: `m_dwPickedItemID`'yi döndürür.
    *   `GetVirtualNumberOfGroundItem()`: Verilen ID'deki yerdeki eşyanın VNUM'unu döndürür.
    *   `BuildNoGradeNameData()`, `GetNoGradeNameDataCount()`, `GetNoGradeNameDataPtr()`: Belirli bir türdeki eşyaları `m_NoGradeNameItemData` vektörüne yükler (muhtemelen UI'da özel bir liste için, kod yorum satırı halinde).
    *   `CanAddChangeLookItem()`, `CanAddChangeLookFreeItem()`, `IsChangeLookClearScrollItem()` (`ENABLE_CHANGE_LOOK_SYSTEM` ile): Görünüm değiştirme sistemi için eşya uygunluk kontrolleri yapar.
    *   `IsLootFilteredItem()`, `InsertLootFilteredItem()`, `EraseLootFilteredItem()`, `ClearLootFilteredItems()` (`ENABLE_LOOTING_SYSTEM` ile): Yağma filtresiyle ilgili işlemleri gerçekleştirir.
    *   `CanPickGroundItem()` (`ENABLE_CLOSE_ITEM_TEXT_TAIL_COLOR` ile): Oyuncunun bir eşyayı toplayıp toplayamayacağını mesafe kontrolü ile belirler.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dosya, `CPythonItem` sınıfının kalbidir ve yerdeki eşyaların tüm görsel ve temel etkileşim mantığını barındırır.

*   **Yerdeki Eşyaların Canlı Tutulması**: Düşme animasyonları, pozisyon güncellemeleri ve render işlemleri bu dosyadaki kodlarla yönetilir.
*   **Oyuncu Etkileşimi**: Oyuncunun fare ile bir eşyayı seçmesi (`__Pick`), eşyaya yakın olup olmadığını kontrol etmesi (`GetCloseItem`) gibi işlemler burada gerçekleşir.
*   **Ses ve Efekt Entegrasyonu**: Eşyaların düşerken veya kullanılırken çıkardığı sesler ve görsel efektler bu sınıftaki fonksiyonlar aracılığıyla tetiklenir.
*   **Sistem Özellikleri**: Koşullu derleme blokları (`#ifdef`) sayesinde, oyundaki farklı sistemlerin (yağma filtresi, eşya yok olma süresi, grafik seçenekleri vb.) eşya yönetimiyle nasıl entegre olduğu görülür.

---

### `PythonItemModule.cpp` (Python `item` Modülü)

Bu dosya, `CPythonItem` C++ sınıfının ve `CItemManager` (GameLib içinde yer alan, eşya prototiplerini yöneten sınıf) sınıfının çeşitli işlevlerini Python betiklerine sunan `item` adlı bir Python modülünü tanımlar ve başlatır. Bu sayede Python tabanlı UI scriptleri, eşya verilerine erişebilir, eşya özelliklerini sorgulayabilir, yerdeki eşyalarla etkileşimde bulunabilir ve eşya seslerini yönetebilir.

#### Dosyanın Genel Amaçları

*   **Python Modülü Oluşturma**: `item` adında bir Python modülü oluşturur.
*   **C++ Fonksiyonlarını Python'a Sarmalama**: `CPythonItem` ve `CItemManager` sınıflarının public metotlarını çağıran ve sonuçları Python veri türlerine dönüştüren C fonksiyonları (Python sarmalayıcıları) tanımlar.
*   **Sabitlerin Python'a Aktarılması**: Eşya türleri, alt türleri, bayraklar, anti-bayraklar, kullanım limit türleri, efsun türleri gibi birçok sabiti Python modülüne `PyModule_AddIntConstant` ile ekler.
*   **Modül Başlatma**: Python yorumlayıcısı tarafından `item` modülü yüklendiğinde çağrılacak olan `initItem()` fonksiyonunu sağlar.

#### C++ ve Python Arayüzü

Bu dosya, C++ eşya yönetim sistemi ile Python scriptleri arasında bir köprü görevi görür.

*   **`initItem()` Fonksiyonu**:
    *   Python modülü yüklendiğinde çağrılır.
    *   `static PyMethodDef s_methods[]` dizisi, Python'a sunulacak tüm fonksiyonların isimlerini (örneğin, "SelectItem", "GetItemName", "Pick") ve bunlara karşılık gelen C fonksiyon sarmalayıcılarını tanımlar.
    *   `Py_InitModule("item", s_methods);` komutu ile `item` modülünü Python'a kaydeder.
    *   Çok sayıda eşya ile ilgili sabiti (örn: `ITEM_TYPE_WEAPON`, `WEAPON_SWORD`, `APPLY_MAX_HP`, `ITEM_ANTIFLAG_FEMALE`) modüle ekler.

*   **Python Fonksiyon Sarmalayıcıları (`item*` önekli fonksiyonlar)**:
    Bu fonksiyonlar, Python'dan çağrıldığında genellikle `CItemManager::Instance()` veya `CPythonItem::Instance()` üzerinden ilgili C++ metodunu çağırır, Python'dan gelen argümanları C++ türlerine dönüştürür ve C++ metotlarından dönen sonuçları Python veri türlerine çevirir.

    *   **Eşya Prototip Verileri (Genellikle `CItemManager` üzerinden)**:
        *   `itemSelectItem(int iIndex)`: Verilen VNUM'daki eşyayı `CItemManager`'da aktif eşya olarak seçer.
        *   `itemGetItemName()`, `itemGetItemNameByVnum(int iIndex)`: Seçili veya verilen VNUM'daki eşyanın adını döndürür.
        *   `itemGetItemDescription()`, `itemGetItemSummary()`: Eşyanın açıklamasını ve özetini döndürür.
        *   `itemGetIconImage()`: Eşyanın ikon resminin handle'ını (işaretçi olarak) döndürür.
        *   `itemGetIconImageFileName()`: Eşyanın ikon resim dosyasının adını döndürür.
        *   `itemGetItemSize()`: Eşyanın envanterdeki boyutunu (genişlik, yükseklik) döndürür.
        *   `itemGetItemType()`, `itemGetItemSubType()`: Eşyanın ana ve alt türünü döndürür.
        *   `itemGetIBuyItemPrice()`, `itemGetISellItemPrice()`: Alış ve satış fiyatını döndürür.
        *   `itemIsAntiFlag(int iFlag)`, `itemIsFlag(int iFlag)`, `itemIsWearableFlag(int iFlag)`: Belirli bayrakların (anti-flag, flag, giyilebilirlik flag'ı) kurulu olup olmadığını kontrol eder.
        *   `itemGetLimit(int iValueIndex)`: Eşyanın kullanım limitlerinden birini (tür, değer) döndürür.
        *   `itemGetAffect(int iValueIndex)`: Eşyanın verdiği efsunlardan (apply/bonus) birini (tür, değer) döndürür. Çiftel silahlar için saldırı hızı düzeltmesi içerir.
        *   `itemGetValue(int iValueIndex)`: Eşyanın `Value0`..`Value5` gibi genel amaçlı değerlerinden birini döndürür.
        *   `itemGetSocket(int iValueIndex)`: Eşyanın belirli bir soketindeki değeri (genellikle taş VNUM'u) döndürür.
        *   `itemGetIconInstance()`: Eşya ikonu için yeni bir `CGraphicImageInstance` oluşturur ve handle'ını döndürür.
        *   `itemDeleteIconInstance(handle)`: Oluşturulan ikon instance'ını siler.
        *   `itemIsEquipmentVID(int iItemVID)`: Verilen VNUM'daki eşyanın bir ekipman olup olmadığını kontrol eder.
        *   `itemGetUseType(int iItemVID)`: Eşyanın kullanım türüyle ilgili bir string döndürür.
        *   `itemIsRefineScroll()`, `itemIsDetachScroll()`, `itemIsKey()`, `itemIsMetin()`: Belirli eşya türlerini (geliştirme parşömeni, taş sökme, anahtar, metin taşı) kontrol eder.
        *   `itemCanAddToQuickSlotItem(int iItemIndex)`: Eşyanın hızlı slota eklenip eklenemeyeceğini kontrol eder.
        *   `itemGetRefinedVnum()`: Seçili eşyanın bir sonraki basımındaki VNUM'unu döndürür.
        *   `itemLoadItemTable(char* szFileName)`: Verilen dosyadan eşya prototip tablosunu (`item_proto`) yükler.
        *   Dragon Soul (Ejderha Taşı Simyası) ile ilgili çeşitli fonksiyonlar (`itemGetDSSetWeight`, `itemIsItemUsedForDragonSoul` vb.).
        *   Görünüm Değiştirme Sistemi (`ENABLE_CHANGE_LOOK_SYSTEM`) ile ilgili fonksiyonlar.
        *   Evlilik eşyalarını kontrol eden `itemIsWeddingItem()`.
        *   Diğer özel sistemlerle (Premium Pazar, Ruh Sistemi, Evcil Hayvan, 6./7. Efsun vb.) ilgili çeşitli kontrol ve bilgi alma fonksiyonları.

    *   **Yerdeki Eşya Yönetimi (`CPythonItem` üzerinden)**:
        *   `itemSetUseSoundFileName()`, `itemSetDropSoundFileName()`: Kullanım ve düşme sesi dosyalarını ayarlar.
        *   `itemRender()`: Yerdeki tüm eşyaları render eder.
        *   `itemUpdate()`: Yerdeki eşyaların durumunu ve fare etkileşimlerini günceller.
        *   `itemCreateItem(virtualID, virtualNumber, x, y, z, bDrop)`: Yere bir eşya örneği oluşturur.
        *   `itemDeleteItem(virtualID)`: Yerdeki bir eşyayı siler.
        *   `itemPick()`: Fare ile üzerine gelinen yerdeki eşyanın ID'sini döndürür.

#### Oyunda Kullanım Amacı ve Senaryoları

Python'daki `item` modülü, oyunun UI scriptlerinin (genellikle Python ile yazılır) eşyalarla ilgili hemen hemen her türlü bilgiye erişmesini ve temel işlemleri yapmasını sağlar.

*   **Envanter ve Ekipman Yönetimi**: Envanterdeki veya karakterin üzerindeki eşyaların bilgilerini (isim, ikon, özellikler) göstermek için kullanılır.
*   **Eşya Açıklama Pencereleri (Tooltip)**: Fare bir eşyanın üzerine geldiğinde açılan bilgi penceresindeki tüm veriler bu modül aracılığıyla çekilir.
*   **NPC Dükkanları ve Ticaret**: Eşyaların fiyatlarını, türlerini ve diğer özelliklerini kontrol etmek için kullanılır.
*   **Geliştirme (Refine) ve Üretim Arayüzleri**: Eşyaların geliştirme bilgilerini, gerekli malzemeleri ve sonuçlarını göstermek için kullanılır.
*   **Yere Düşen Eşyalarla Etkileşim**: Python scriptleri, `item.Pick()` ile oyuncunun hangi eşyayı toplamaya çalıştığını anlar.
*   **Hızlı Slotlar ve Yetenekler**: Kullanılabilir eşyaların hızlı slota eklenip eklenemeyeceğini kontrol eder.
*   **Özel Sistem Arayüzleri**: Ejderha Taşı Simyası, Görünüm Değiştirme gibi özel sistemlerin arayüzleri, eşya uygunluklarını ve bilgilerini bu modül üzerinden sorgular.

Bu modül, `CItemManager`'ın statik eşya veritabanı ile `CPythonItem`'ın dinamik yerdeki eşya yönetimi arasında bir arayüz görevi görerek Python scriptlerine güçlü bir eşya yönetim kabiliyeti sunar.

---