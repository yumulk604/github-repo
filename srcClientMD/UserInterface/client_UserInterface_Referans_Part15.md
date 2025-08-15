# UserInterface Referans Kılavuzu - Bölüm 15

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonSkill.h` (Yetenek Sistemi Başlık Dosyası)](#pythonskillh-yetenek-sistemi-başlık-dosyası)
*   [`PythonSkill.cpp` (Yetenek Sistemi Uygulaması ve Python Modülü)](#pythonskillcpp-yetenek-sistemi-uygulaması-ve-python-modülü)
*   [`PythonSoundManagerModule.cpp` (Ses Yöneticisi Python Modülü)](#pythonsoundmanagermodulecpp-ses-yöneticisi-python-modülü)
*   [`PythonSwitchbot.h` (Efsun Botu Sistemi Başlık Dosyası)](#pythonswitchboth-efsun-botu-sistemi-başlık-dosyası)
*   [`PythonSwitchbot.cpp` (Efsun Botu Sistemi Uygulaması ve Python Modülü)](#pythonswitchbotcpp-efsun-botu-sistemi-uygulaması-ve-python-modülü)
*   [`PythonSystem.h` (Sistem Ayarları Başlık Dosyası)](#pythonsystemh-sistem-ayarları-başlık-dosyası)
*   [`PythonSystem.cpp` (Sistem Ayarları Uygulaması)](#pythonsystemcpp-sistem-ayarları-uygulaması)
*   [`PythonSystemModule.cpp` (Sistem Ayarları Python Modülü)](#pythonsystemmodulecpp-sistem-ayarları-python-modülü)
*   [`PythonTextTail.h` (Metin Kuyruğu Başlık Dosyası)](#pythontexttailh-metin-kuyruğu-başlık-dosyası)
*   [`PythonTextTail.cpp` (Metin Kuyruğu Uygulaması)](#pythontexttailcpp-metin-kuyruğu-uygulaması)
*   [`PythonTextTailModule.cpp` (Metin Kuyruğu Python Modülü)](#pythontexttailmodulecpp-metin-kuyruğu-python-modülü)
*   [`PythonWhisper.h` (Fısıltı Sistemi Yenileme Başlık Dosyası)](#pythonwhisperh-fısıltı-sistemi-yenileme-başlık-dosyası)
*   [`PythonWhisper.cpp` (Fısıltı Sistemi Yenileme Uygulaması ve Python Modülü)](#pythonwhispercpp-fısıltı-sistemi-yenileme-uygulaması-ve-python-modülü)
*   [`resource.h` (Kaynak Tanımlama Başlık Dosyası)](#resourceh-kaynak-tanımlama-başlık-dosyası)
*   [`ServerStateChecker.h` (Sunucu Durum Denetleyicisi Başlık Dosyası)](#serverstatecheckerh-sunucu-durum-denetleyicisi-başlık-dosyası)
*   [`ServerStateChecker.cpp` (Sunucu Durum Denetleyicisi Uygulaması)](#serverstatecheckercpp-sunucu-durum-denetleyicisi-uygulaması)
*   [`ServerStateCheckerModule.cpp` (Sunucu Durum Denetleyicisi Python Modülü)](#serverstatecheckermodulecpp-sunucu-durum-denetleyicisi-python-modülü)
*   [`StdAfx.h` (Standart Uygulama Çerçevesi Başlık Dosyası)](#stdafxh-standart-uygulama-çerçevesi-başlık-dosyası)
*   [`StdAfx.cpp` (Standart Uygulama Çerçevesi Kaynak Dosyası)](#stdafxcpp-standart-uygulama-çerçevesi-kaynak-dosyası)

---

### `PythonSkill.h` (Yetenek Sistemi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonSkill.h`, Metin2 istemcisindeki karakter yetenekleriyle (skill) ilgili C++ tarafındaki temel veri yapılarını, sabitlerini ve `CPythonSkill` (ve `ENABLE_GROWTH_PET_SYSTEM` aktifse `CPythonSkillPet`) singleton sınıfının arayüzünü tanımlar. Bu dosya, yetenek verilerinin yüklenmesi, saklanması ve Python'a açılması için altyapı sağlar. Yeteneklerin türleri, etkileri, gereksinimleri, ikonları, animasyonları ve diğer tüm özellikleri bu başlık dosyasında tanımlanan yapılar aracılığıyla yönetilir.

**Ana Yapılar ve Tanımlamalar**

*   **`CPythonSkill::SKILL_TYPE` (Enum)**:
    *   Temel yetenek türlerini tanımlar: `SKILL_TYPE_NONE`, `SKILL_TYPE_ACTIVE`, `SKILL_TYPE_SUPPORT`, `SKILL_TYPE_GUILD`, `SKILL_TYPE_HORSE`.

*   **`CPythonSkill::ESkillIndexes` (Enum)**:
    *   Oyundaki tüm yetenekler için sabit VNUM (Virtual Number - Sanal Numara) değerlerini listeler. Örneğin, `SKILL_SAMYEON` (Üç Yönlü Kesme), `SKILL_GEOMKYUNG` (Kılıç Çevirme), `SKILL_LEADERSHIP` (Liderlik), `SKILL_POLYMORPH` (Dönüşüm) gibi. Lonca yetenekleri ve at yetenekleri için de aralıklar belirtilmiştir.

*   **`CPythonSkill::ESkillTableTokenType` ve `CPythonSkill::ESkillDescTokenType` (Enum'lar)**:
    *   `skilltable.txt` ve `skilldesc.txt` dosyalarından veri okurken kullanılacak sütun (token) indekslerini tanımlar. Bu, dosya formatındaki değişikliklere karşı bir nebze esneklik sağlar ve kod okunurluğunu artırır.

*   **Yetenek Özellikleri (`SKILL_ATTRIBUTE_*`) (Enum benzeri sabitler)**:
    *   Bir yeteneğin sahip olabileceği çeşitli özellikleri bit bayrakları (bit flags) olarak tanımlar. Örneğin:
        *   `SKILL_ATTRIBUTE_NEED_TARGET`: Hedef gerektirir.
        *   `SKILL_ATTRIBUTE_TOGGLE`: Açılıp kapatılabilir (toggle) yetenek.
        *   `SKILL_ATTRIBUTE_MELEE_ATTACK`: Yakın dövüş saldırısı.
        *   `SKILL_ATTRIBUTE_PASSIVE`: Pasif yetenek.
        *   `SKILL_ATTRIBUTE_HORSE_SKILL`: At üzerinde kullanılabilen yetenek.
        *   `SKILL_ATTRIBUTE_TIME_INCREASE_SKILL`: Etki süresi seviyeyle artan yetenek.

*   **Gerekli Silah Türleri (`SKILL_NEED_WEAPON_*`) (Enum benzeri sabitler)**:
    *   Bir yeteneğin kullanılabilmesi için hangi silah türlerinin kuşanılmış olması gerektiğini bit bayrakları olarak tanımlar (örneğin, `SKILL_NEED_WEAPON_SWORD`, `SKILL_NEED_WEAPON_BOW`).

*   **`CPythonSkill::SSkillData` (Yapı)**:
    *   Bir yeteneğin tüm verilerini ve bu verileri işleyen fonksiyonları içeren ana yapıdır.
    *   **Temel Bilgiler**: `byType` (tür), `dwSkillIndex` (VNUM), `byMaxLevel` (maksimum seviye), `byLevelUpPoint` (seviye başına artış puanı), `byLevelLimit` (kullanım için seviye sınırı), `bNoMotion` (animasyonsuz mu?).
    *   **İsim, İkon, Açıklama**: `strName` (isim), `strIconFileName` (ikon dosya adı), `strDescription` (açıklama), `pImage` (yüklenmiş ikon `CGraphicImage` nesnesi).
    *   **Formüller**:
        *   `strCoolTimeFormula`: Bekleme süresi formülü.
        *   `strNeedSPFormula`: Gerekli SP formülü.
        *   `strContinuationSPFormula`: Sürekli SP tüketimi formülü (toggle yetenekler için).
        *   `strDuration`: Etki süresi formülü.
        *   `strTargetCountFormula`: Etkilenen hedef sayısı formülü.
        *   `strMotionLoopCountFormula`: Animasyon döngü sayısı formülü.
    *   **Özellikler ve Gereksinimler**:
        *   `dwSkillAttribute`: Yetenek özellik bayrakları.
        *   `dwNeedWeapon`: Gerekli silah türü bayrakları.
        *   `dwTargetRange`: Yetenek menzili.
        *   `isRequirement`: Başka bir yeteneğe gereksinim duyup duymadığı.
        *   `strRequireSkillName`, `byRequireSkillLevel`: Gerekli yeteneğin adı ve seviyesi.
        *   `ConditionDataVector`: Kullanım koşulu açıklamaları (string vektörü).
        *   `RequireStatDataVector`: Gerekli stat (güç, zeka vb.) ve seviyeleri (`TRequireStatData` vektörü).
    *   **Etki Verileri (`TAffectData`, `TAffectDataNew`)**:
        *   `AffectDataVector`: Yeteneğin etkilerini (açıklama, min/max formül) tutan `TAffectData` vektörü.
        *   `AffectDataNewVector`: Yeni sistem için etki verilerini tutan `TAffectDataNew` vektörü (etki türü, etki formülü).
    *   **Animasyon ve Derece Bilgileri**:
        *   `wMotionIndex`: Temel animasyon indeksi.
        *   `wMotionIndexForMe`: Kendine uygulanan animasyon indeksi.
        *   `GradeData[SKILL_EFFECT_COUNT]`: Yetenek derecelerine (Normal, Master, Grandmaster, Perfect) göre isim, ikon ve animasyon indeksi (`TGradeData` dizisi).
    *   **Statik Üyeler**:
        *   `ms_StatusNameMap`, `ms_NewMinStatusNameMap`, `ms_NewMaxStatusNameMap`: Formüllerde kullanılan stat isimlerini (`"LV"`, `"STR"`, `"INT"`, `"MinATK"` vb.) `POINT` enum değerleriyle eşleştiren map'ler.
        *   `ms_dwTimeIncreaseSkillNumber`: Süresi seviyeyle artan özel bir yeteneğin VNUM'unu tutar.
    *   **Fonksiyonlar**: `GetTargetRange()`, `IsToggleSkill()`, `GetSkillCoolTime()`, `GetNeedSP()`, `GetAffectDescription()` gibi yetenek verilerini işleyen ve döndüren çok sayıda üye fonksiyon.

*   **`CPythonSkill::SSkillData::TGradeData` (Yapı)**:
    *   Bir yeteneğin belirli bir derecesi (M, G, P) için isim (`strName`), ikon (`pImage`) ve animasyon indeksi (`wMotionIndex`) bilgilerini tutar.

*   **`CPythonSkill` (Singleton Sınıfı)**:
    *   Tüm yetenek verilerini yönetir (`m_SkillDataMap`).
    *   `RegisterSkillTable()`, `RegisterSkillDesc()`: `skilltable.txt` ve `skilldesc.txt` dosyalarından yetenek verilerini okuyup `m_SkillDataMap`'e yükler.
    *   `GetSkillData()`: VNUM ile yetenek verisini alır.
    *   `GetSkillDataByName()`: İsim ile yetenek verisini alır.
    *   `SetPathName()`: Yetenek dosyalarının bulunduğu yolu ayarlar.
    *   Çeşitli yardımcı map'ler (`m_SkillTypeIndexMap`, `m_SkillAttributeIndexMap` vb.) stringleri enum/sabit değerlere çevirmek için kullanılır.

*   **`CPythonSkillPet` (Singleton Sınıfı - `ENABLE_GROWTH_PET_SYSTEM` ile)**:
    *   Evcil hayvan yetenekleri için benzer bir yapı sunar.
    *   `CPythonSkillPet::SSkillDataPet` yapısı evcil hayvan yetenek verilerini tutar (VNUM, isim, ikon, tür, açıklama, bekleme süresi).
    *   `RegisterSkillPet()`: Evcil hayvan yetenek verilerini yükler.
    *   `GetSkillData()`: VNUM ile evcil hayvan yetenek verisini alır.
    *   `SetSkillbySlot()`: Evcil hayvanın yetenek slotlarına yetenek atar.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, istemcinin oyun içindeki tüm karakter ve evcil hayvan yeteneklerinin tanımlarını, özelliklerini ve davranışlarını bilmesini sağlar. Oyun başladığında, bu sınıflar aracılığıyla ilgili metin dosyalarından (`skilldesc.txt`, `skilltable.txt`, `petskill.txt`) tüm yetenek verileri yüklenir. Oyuncu yetenek arayüzünü açtığında, bir yeteneğin üzerine geldiğinde, bir yeteneği kullandığında veya yetenekle ilgili herhangi bir bilgiye ihtiyaç duyulduğunda, bu başlık dosyasında tanımlanan yapılar ve fonksiyonlar kullanılır. Python tarafındaki UI (`uiSkill.py`, `uiToolTip.py` vb.) ve oyun mantığı, `CPythonSkill` (ve `CPythonSkillPet`) sınıfına erişerek yetenek bilgilerini alır ve oyuncuya sunar veya oyun mekaniklerini çalıştırır.

---

### `PythonSkill.cpp` (Yetenek Sistemi Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonSkill.cpp`, `CPythonSkill` sınıfının (ve `ENABLE_GROWTH_PET_SYSTEM` aktifse `CPythonSkillPet` sınıfının) implementasyonunu ve `skill` adlı bir Python modülünün tanımlanmasını içerir. Bu dosya, `skilldesc.txt` ve `skilltable.txt` (ve evcil hayvanlar için `petskill.txt`) dosyalarından yetenek verilerinin okunup işlenmesi, saklanması ve bu verilere hem C++ içinden hem de Python betikleri aracılığıyla erişilmesi için gerekli mantığı barındırır. Yeteneklerin isimleri, açıklamaları, ikonları, türleri, etkileri, bekleme süreleri, SP tüketimleri, animasyonları gibi tüm detayları bu dosya üzerinden yönetilir.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **`CPythonSkill` Sınıfı Implementasyonu**:
    *   **Yetenek Veri Yükleme (`RegisterSkillDesc`, `RegisterSkillTable`)**:
        *   `RegisterSkillDesc()`: `skilldesc.txt` dosyasını okur. Her satırdan yeteneğin temel bilgilerini (VNUM, sınıf/tür, isimler, açıklama, kullanım koşulları, özellik bayrakları, gerekli silah türleri, ikon adı, animasyon indeksi, etki açıklamaları ve formülleri, seviye sınırı, maksimum seviye) alır.
            *   İkonlar: Yetenek türüne ve işine (`job`) göre doğru yoldan (`g_strImagePath` + sınıf yolu + ikon adı) yüklenir. `__RegisterGradeIconImage` ve `__RegisterNormalIconImage` yardımcı fonksiyonları kullanılır.
            *   Özellikler ve Gerekli Silahlar: `"|"` ile ayrılmış stringler okunur ve `m_SkillAttributeIndexMap`, `m_SkillNeedWeaponIndexMap` kullanılarak DWORD bayraklarına çevrilir.
            *   Etkiler (`AffectDataVector`): Birden fazla etki tanımlanabilir, her biri için açıklama ve min/max formülleri okunur.
        *   `RegisterSkillTable()`: `skilltable.txt` dosyasını okur. Bu dosya, `skilldesc.txt`'de tanımlanan yeteneklere ek bilgiler (SP maliyeti, bekleme süresi, etki süresi formülleri, hedef menzili, maksimum seviye, seviye sınırı, asıl etki değeri formülü (`PointPoly`)) ekler.
            *   `OVERWRITE_SKILLPROTO_POLY` makrosu aktifse ve yetenek bazı koşulları sağlıyorsa (`atk`, `mwep`, `number` içeriyorsa), `PointPoly` formülü `min` ve `max` değerler için ayrı formüllere (`strAffectMinFormula`, `strAffectMaxFormula`) dönüştürülür. Bu, genellikle saldırı yeteneklerinin hasar aralıklarını belirlemek için kullanılır.
    *   **Yetenek Verilerine Erişim**:
        *   `GetSkillData(DWORD dwSkillIndex, TSkillData** ppSkillData)`: Verilen VNUM'a sahip yeteneğin `TSkillData` yapısına işaretçi döndürür.
        *   `GetSkillDataByName(const char* c_szName, TSkillData** ppSkillData)`: Verilen isme sahip yeteneği bulur.
    *   **Kurucu (`CPythonSkill()`)**:
        *   `m_SkillTypeIndexMap`, `m_SkillAttributeIndexMap`, `m_SkillNeedWeaponIndexMap`, `m_SkillWeaponTypeIndexMap`, `SSkillData::ms_StatusNameMap` gibi çeşitli map'leri yetenek özelliklerini, türlerini, silah gereksinimlerini ve formüllerde kullanılacak stat isimlerini (`"LV"`, `"STR"`, `"MinATK"` vb.) ilgili enum/sabit değerlerle doldurur.
    *   **Yıkıcı (`~CPythonSkill()`)**: Kaynakları serbest bırakır (genellikle `m_SkillDataMap.clear()` yeterlidir).

*   **`CPythonSkill::SSkillData` Yapısı Fonksiyonları**:
    *   `IsMeleeSkill()`, `IsToggleSkill()`, `CanUseWeaponType()` vb.: Yeteneğin özellik bayraklarını (`dwSkillAttribute`, `dwNeedWeapon`) kontrol ederek `TRUE`/`FALSE` döndürür.
    *   `GetTargetRange()`: Yeteneğin menzilini döndürür (yakın dövüşse sabit bir değer, değilse `dwTargetRange`).
    *   `ProcessFormula(CPoly* pPoly, float fSkillLevel, int iMinMaxType)`: Verilen bir `CPoly` nesnesi (formül) ve yetenek seviyesi ile formülü hesaplar. Formüldeki değişkenleri (`"LV"`, `"STR"`, `"SkillPoint"` vb.) `GetState()` veya `fSkillLevel` ile oyuncunun/yeteneğin mevcut değerleriyle değiştirir ve sonucu döndürür. `iMinMaxType` (VALUE_TYPE_MIN/MAX) ile min/max değerler için farklı stat map'leri kullanılabilir.
    *   `GetAffectDescription(DWORD dwIndex, float fSkillLevel)`: Belirli bir etki için (formülleri kullanarak) hesaplanmış değerleri içeren açıklama metnini döndürür.
    *   `GetSkillCoolTime()`, `GetNeedSP()`, `GetContinuationSP()`, `GetDuration()`, `GetTargetCount()`, `GetMotionLoopCount()`: İlgili formülleri (`strCoolTimeFormula` vb.) `ProcessFormula` ile hesaplayıp sonucu döndürür.
    *   `GetSkillMotionIndex(int iGrade)`: Yeteneğin derecesine göre animasyon indeksini döndürür.
    *   `GetMaxLevel()`, `GetLevelUpPoint()`: Yeteneğin maksimum seviyesini ve seviye başına artış puanını döndürür.

*   **`CPythonSkillPet` Sınıfı Implementasyonu (`ENABLE_GROWTH_PET_SYSTEM` ile)**:
    *   `RegisterSkillPet()`: `petskill.txt` dosyasını okuyarak evcil hayvan yeteneklerinin VNUM, isim, ikon adı, tür, açıklama ve bekleme süresi gibi bilgilerini `m_SkillDataPetMap`'e yükler.
    *   `GetSkillData()`: VNUM ile evcil hayvan yetenek verisini alır.
    *   `SetSkillbySlot()`, `GetSkillIndex()`: Evcil hayvanın yetenek slotlarındaki yetenekleri yönetir.

*   **`skill` Python Modülü Fonksiyonları**:
    *   `skillRegisterSkillDesc(szFileName)`, `skillRegisterSkillTable(szFileName)`: C++ tarafındaki `RegisterSkillDesc` ve `RegisterSkillTable` fonksiyonlarını çağırır.
    *   `skillGetSkillName(iSkillIndex, iSkillGrade=-1)`: Yeteneğin (veya belirli bir derecesinin) adını döndürür.
    *   `skillGetSkillDescription(iSkillIndex)`: Yeteneğin açıklamasını döndürür.
    *   `skillGetSkillType(iSkillIndex)`: Yeteneğin türünü döndürür.
    *   `skillGetSkillCoolTime(iSkillIndex, fSkillPoint)`, `skillGetSkillNeedSP(iSkillIndex, fSkillPoint)` vb.: `SSkillData` üye fonksiyonlarını çağırarak ilgili değerleri Python'a döndürür.
    *   `skillGetSkillAffectDescription(iSkillIndex, iAffectIndex, fSkillPoint)`: Belirli bir yetenek etkisinin hesaplanmış açıklamasını döndürür.
    *   `skillGetIconImage(iSkillIndex)`, `skillGetIconImageNew(iSkillIndex, iGradeIndex)`: Yetenek ikonunun (`CGraphicImage*`) adresini (handle) döndürür.
    *   `skillGetIconInstance(iSkillIndex)`, `skillGetIconInstanceNew(iSkillIndex, iGradeIndex)`: Yeni bir `CGraphicImageInstance` oluşturup yetenek ikonunu atar ve bu nesnenin adresini döndürür.
    *   `skillDeleteIconInstance(iHandle)`: Verilen handle'daki `CGraphicImageInstance`'ı siler.
    *   `skillIsToggleSkill(iSkillIndex)`, `skillCanUseSkill(iSkillIndex)` vb.: Yeteneğin çeşitli özelliklerini sorgulayan fonksiyonlar.
    *   `skillCanLevelUpSkill(iSkillIndex, iSkillLevel)`: Yeteneğin seviye atlatılıp atlatılamayacağını kontrol eder (maksimum seviye, stat/diğer yetenek gereksinimleri).
    *   `ENABLE_GROWTH_PET_SYSTEM` ile evcil hayvan yetenekleri için benzer Python fonksiyonları (`skillSetPetSkillSlot`, `skillGetPetSkillIconImage`, `skillGetPetSkillInfo` vb.) eklenir.

*   **Modül Başlatma (`initskill()`)**:
    *   `skill` adında Python modülünü oluşturur ve yukarıda bahsedilen C++ fonksiyonlarını sarmalayan Python çağrılabilir metodları kaydeder.
    *   `PyModule_AddIntConstant` kullanarak `SKILL_TYPE_*`, `SKILL_GRADE_COUNT` gibi sabitleri Python modülüne ekler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, istemcinin tüm yetenek sistemi mantığının merkezidir. Oyun başladığında, `skilldesc.txt` ve `skilltable.txt` dosyaları okunarak tüm yeteneklerin özellikleri, formülleri, ikonları ve diğer detayları `CPythonSkill` sınıfı içinde saklanır. Python tarafındaki UI betikleri (`uiSkill.py`, `QuickSlot.py`, `GameWindow.py` gibi), `skill` modülü aracılığıyla bu bilgilere erişir. Örneğin:
*   Yetenek penceresinde (`uiSkill.py`) yeteneklerin listelenmesi, ikonlarının gösterilmesi, üzerine gelindiğinde detaylı bilgi (`ToolTip`) görüntülenmesi.
*   Bir yetenek kullanıldığında (`CPythonPlayer::UseSkill()`), gerekli SP'nin, bekleme süresinin, animasyonun bu dosyadan alınan verilere göre belirlenmesi.
*   Yetenek seviyesi atlatılırken gereksinimlerin kontrol edilmesi.
*   Pasif yeteneklerin etkilerinin veya aktif destek yeteneklerinin sürelerinin ve etkilerinin hesaplanması.
Kısacası, oyuncunun yeteneklerle ilgili gördüğü ve etkileşimde bulunduğu her şey, bu dosyadaki C++ ve Python katmanları arasındaki veri alışverişi ve mantık işlemleriyle yönetilir. Linter hataları (örneğin, `ENABLE_ATTR_6TH_7TH` ile ilgili eksik parantez veya `IBackground` ile ilgili eksik tip tanımı) derleme sorunlarına ve dolayısıyla bu işlevlerin hatalı çalışmasına veya hiç çalışmamasına neden olabilir. 

---

### `PythonSoundManagerModule.cpp` (Ses Yöneticisi Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonSoundManagerModule.cpp`, Metin2 istemcisindeki ses yönetimi işlevlerini Python betiklerine açan bir CPython modülü (`snd`) tanımlar. Bu dosya, `CSoundManager` singleton sınıfının fonksiyonlarını sarmalayarak Python üzerinden 2D ve 3D ses efektlerinin çalınmasını, müziklerin çalınmasını, ses seviyelerinin ayarlanmasını ve çeşitli ses kontrol işlemlerinin yapılmasını mümkün kılar.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **Python Modülü (`snd`) Tanımlaması**:
    *   `initsnd()` fonksiyonu, `snd` adında bir Python modülü oluşturur ve bu modüle C++ fonksiyonlarını bağlayan Python metodlarını (`PyMethodDef s_methods[]`) kaydeder.

*   **Ses Çalma Fonksiyonları**:
    *   `sndPlaySound(fileName)`: Belirtilen 2D ses dosyasını çalar. `CSoundManager::Instance().PlaySound2D()` fonksiyonunu çağırır.
    *   `sndPlaySound3D(x, y, z, fileName)`: Belirtilen 3D pozisyonda (x, y, z koordinatları) ses dosyasını çalar. `CSoundManager::Instance().PlaySound3D()` fonksiyonunu çağırır.
    *   `sndPlayMusic(fileName)`: Belirtilen müzik dosyasını çalar. `CSoundManager::Instance().PlayMusic()` fonksiyonunu çağırır.
    *   `sndFadeInMusic(fileName)`: Belirtilen müzik dosyasını yavaşça artan ses seviyesiyle (fade in) çalmaya başlar. `CSoundManager::Instance().FadeInMusic()` fonksiyonunu çağırır.
    *   `sndFadeOutMusic(fileName)`: Belirtilen müzik dosyasının sesini yavaşça azaltarak (fade out) durdurur. `CSoundManager::Instance().FadeOutMusic()` fonksiyonunu çağırır.
    *   `sndFadeOutAllMusic()`: Çalmakta olan tüm müzikleri yavaşça azaltarak durdurur. `CSoundManager::Instance().FadeOutAllMusic()` fonksiyonunu çağırır.
    *   `sndFadeLimitOutMusic(fileName, limitVolume)`: Belirtilen müziğin sesini belirli bir seviyeye (`limitVolume`) kadar yavaşça azaltır. `CSoundManager::Instance().FadeLimitOutMusic()` fonksiyonunu çağırır.
    *   `sndStopAllSound()`: Tüm 3D ses efektlerini durdurur. `CSoundManager::Instance().StopAllSound3D()` fonksiyonunu çağırır.

*   **Ses Seviyesi Ayarlama Fonksiyonları**:
    *   `sndSetMusicVolume(volume)` (veya `sndSetMusicVolumef`): Müzik ses seviyesini ayarlar (0.0 ile 1.0 arasında float değer). `CSoundManager::Instance().SetMusicVolume()` fonksiyonunu çağırır.
    *   `sndSetSoundVolumef(volume)`: Genel ses efektleri için ses seviyesi oranını ayarlar (float değer). `CSoundManager::Instance().SetSoundVolumeRatio()` fonksiyonunu çağırır.
    *   `sndSetSoundVolume(volume)`: Genel ses efektleri için ses seviyesi derecesini (genellikle 0-5 arası integer) ayarlar. `CSoundManager::Instance().SetSoundVolumeGrade()` fonksiyonunu çağırır.
    *   `sndSetSoundScale(scale)`: Ses efektlerinin genel ölçeğini ayarlar. `CSoundManager::Instance().SetSoundScale()` fonksiyonunu çağırır.
    *   `sndSetAmbienceSoundScale(scale)`: Ortam seslerinin ölçeğini ayarlar. `CSoundManager::Instance().SetAmbienceSoundScale()` fonksiyonunu çağırır.

*   **Fonksiyon Argümanları ve Hata Yönetimi**:
    *   Tüm Python'a açılan fonksiyonlar `PyObject* poSelf, PyObject* poArgs` argümanlarını alır.
    *   `PyTuple_GetString()`, `PyTuple_GetFloat()`, `PyTuple_GetInteger()` gibi fonksiyonlar Python'dan gelen argümanları C++ türlerine dönüştürmek için kullanılır.
    *   Argüman parse etme veya tip dönüşümü başarısız olursa `Py_BuildException()` ile Python tarafına hata fırlatılır.
    *   Başarılı işlemlerde genellikle `Py_BuildNone()` döndürülür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, oyun içindeki Python betiklerinin (özellikle kullanıcı arayüzü betikleri veya görev betikleri) ses ve müzik çalmasını, durdurmasını ve ses seviyelerini kontrol etmesini sağlar. Örneğin:
*   Bir butona tıklandığında arayüz sesi çalmak (`uiAffectShower.py` içindeki gibi).
*   Belirli bir haritaya girildiğinde veya bir ara sahne başladığında özel bir müzik çalmak (`introLoading.py`, `background.py` gibi).
*   Oyun içi ayarlardan müzik ve ses efektlerinin seviyelerini değiştirmek (`OptionDialog.py` gibi).
*   Belirli oyun olaylarında (örneğin, bir eşya düşünce, bir yetenek kullanılınca) ses efektleri tetiklemek.
Bu sayede oyunun ses atmosferi dinamik bir şekilde yönetilebilir ve oyuncu etkileşimlerine sesli geri bildirimler sağlanabilir.

---

### `PythonSwitchbot.h` (Efsun Botu Sistemi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonSwitchbot.h` başlık dosyası, `ENABLE_SWITCHBOT` makrosu aktif olduğunda derlenen Efsun Botu (Switchbot) sisteminin C++ tarafındaki veri yapılarını ve `CPythonSwitchbot` singleton sınıfının arayüzünü tanımlar. Bu dosya, efsun botu arayüzü için gerekli olan efsun alternatifleri, aktif slotlar, tamamlanmış slotlar ve efsunlanacak eşyalar gibi verilerin istemci tarafında tutulmasını ve yönetilmesini sağlar.

**Ana Yapılar ve Tanımlamalar**

*   **`TSwitchbotAttributeAlternativeTable` (Yapı)**:
    *   Bir efsun botu slotundaki tek bir efsun alternatifi için istenen efsunları tutar.
    *   `attributes[MAX_NORM_ATTR_NUM]`: `TPlayerItemAttribute` tipinde bir dizi. Her eleman bir efsun türünü (`wType`) ve istenen değerini (`sValue`) belirtir.
    *   `IsConfigured()`: Bu alternatifte en az bir efsun yapılandırılmışsa `true` döner.

*   **`TSwitchbotTable` (Yapı)**:
    *   Efsun botu sisteminin tüm durumunu ve yapılandırmasını tutan ana yapıdır.
    *   `player_id`: Oyuncunun kimliği.
    *   `active[SWITCHBOT_SLOT_COUNT]`: Hangi efsun botu slotlarının aktif olduğunu belirten boolean dizi.
    *   `finished[SWITCHBOT_SLOT_COUNT]`: Hangi slotlardaki efsunlama işleminin tamamlandığını belirten boolean dizi.
    *   `items[SWITCHBOT_SLOT_COUNT]`: Her slotta bulunan eşyanın VNUM'unu tutan dizi.
    *   `alternatives[SWITCHBOT_SLOT_COUNT][SWITCHBOT_ALTERNATIVE_COUNT]`: Her slot için birden fazla efsun alternatifi (`TSwitchbotAttributeAlternativeTable`) tutan 2 boyutlu dizi. Oyuncu bir eşya için birden fazla kabul edilebilir efsun kombinasyonu tanımlayabilir.

*   **`TSwitchbottAttributeTable` (Yapı)**:
    *   Belirli bir eşya türü (attribute set) için geçerli olan bir efsunun türünü (`apply_num`) ve alabileceği maksimum değeri (`max_value`) tutar. Bu, arayüzde gösterilecek efsun listesini ve maksimum değerlerini belirlemek için kullanılır.

*   **`CPythonSwitchbot` (Singleton Sınıfı)**:
    *   **Üye Değişkenler**:
        *   `m_SwitchbotTable`: Efsun botunun mevcut durumunu (`TSwitchbotTable` türünde) saklar.
        *   `m_map_AttributesBySet`: Eşya türlerine (attribute set) göre geçerli efsunları ve maksimum değerlerini (`std::map<uint8_t, std::map<uint8_t, long>>`) saklar. İlk anahtar attribute set, ikinci anahtar efsun türü (`apply_num`), değer ise maksimum efsun değeridir.
    *   **Temel Fonksiyonlar**:
        *   `Initialize()`: `m_SwitchbotTable`'ı sıfırlar ve `m_map_AttributesBySet`'i temizler.
        *   `Update(const TSwitchbotTable& table)`: Sunucudan gelen yeni efsun botu verileriyle `m_SwitchbotTable`'ı günceller.
    *   **Efsun Alternatifi Yönetimi**:
        *   `SetAttribute(slot, alternative, attrIdx, attrType, attrValue)`: Belirli bir slot ve alternatifteki bir efsunu ayarlar.
        *   `GetAttribute(slot, alternative, attrIdx)`: Belirli bir slot ve alternatifteki bir efsunu döndürür.
        *   `GetAlternatives(slot, std::vector<TSwitchbotAttributeAlternativeTable>& attributes)`: Bir slot için tanımlanmış tüm efsun alternatiflerini vektöre doldurur.
    *   **Eşya ve Efsun Seti Bilgileri**:
        *   `GetAttibuteSet(slot)`: Belirli bir slottaki eşyanın türüne göre hangi efsun setine (örneğin, `ATTRIBUTE_SET_WEAPON`, `ATTRIBUTE_SET_BODY`) ait olduğunu döndürür. `CItemManager` ve `CPythonPlayer` kullanarak eşya bilgilerini alır.
    *   **Durum Kontrolleri**:
        *   `IsActive(slot)`: Belirli bir slotun aktif olup olmadığını döndürür.
        *   `IsFinished(slot)`: Belirli bir slottaki efsunlama işleminin bitip bitmediğini döndürür.
    *   **Slot Yönetimi**:
        *   `ClearSlot(slot)`: Belirli bir slotu temizler (aktif durumunu, eşyasını ve alternatiflerini sıfırlar).
    *   **Efsun Haritası Yönetimi (Geçerli Efsunlar İçin)**:
        *   `ClearAttributeMap()`: `m_map_AttributesBySet`'i temizler.
        *   `AddAttributeToMap(const TSwitchbottAttributeTable& table)`: Geçerli bir efsunu ve maksimum değerini `m_map_AttributesBySet`'e ekler.
        *   `GetAttributesBySet(iAttributeSet, std::vector<TPlayerItemAttribute>& vec_attributes)`: Belirli bir efsun seti için geçerli tüm efsunları (tür ve maksimum değer) bir vektöre doldurur.
        *   `GetAttributeMaxValue(iAttributeSet, attrType)`: Belirli bir efsun seti ve efsun türü için maksimum efsun değerini döndürür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, istemcinin Efsun Botu sistemiyle ilgili tüm verileri ve durumları yönetmesi için gerekli C++ yapılarını ve sınıf arayüzünü sağlar. Oyuncu efsun botu arayüzünü açtığında (`uiSwitchbot.py`), arayüz bu sınıftaki fonksiyonları çağırarak efsunlanacak eşyaları, tanımlanmış efsun alternatiflerini, slotların aktif/tamamlanmış durumlarını ve bir eşya türü için seçilebilecek geçerli efsunları listeler. Oyuncu efsunları yapılandırdığında veya efsunlamayı başlattığında/durdurduğunda, bu bilgiler `CPythonSwitchbot` sınıfı üzerinden güncellenir ve sunucuya gönderilmek üzere hazırlanır (`PythonSwitchbot.cpp` içindeki Python metodları aracılığıyla).

---

### `PythonSwitchbot.cpp` (Efsun Botu Sistemi Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonSwitchbot.cpp` dosyası, `ENABLE_SWITCHBOT` makrosu tanımlı olduğunda derlenir ve `CPythonSwitchbot` sınıfının implementasyonunu ve `switchbot` adlı bir Python modülünün tanımlanmasını içerir. Bu dosya, efsun botu sisteminin istemci tarafındaki mantığını yönetir: sunucudan gelen verilerin işlenmesi, oyuncunun arayüz üzerinden yaptığı değişikliklerin saklanması, efsun botu işlemlerinin başlatılması/durdurulması için sunucuya paket gönderilmesi ve Python arayüzünün (`uiSwitchbot.py`) ihtiyaç duyduğu verilerin sağlanması.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **`CPythonSwitchbot` Sınıfı Implementasyonu**:
    *   **`Initialize()`**: Kurucuda ve yıkıcıda çağrılır. `m_SwitchbotTable`'ı varsayılan değerlere ayarlar ve `m_map_AttributesBySet` haritasını temizler.
    *   **`Update(const TSwitchbotTable& table)`**: Sunucudan `TSwitchbotTable` yapısı geldiğinde çağrılır. Mevcut `m_SwitchbotTable`'ı gelen veriyle günceller (aktif durumlar, bitmiş durumlar, alternatifler, eşyalar).
    *   **`SetAttribute(slot, alternative, attrIdx, attrType, attrValue)`**: Oyuncunun arayüzden belirlediği bir efsun alternatifini (`TSwitchbotAttributeAlternativeTable` içindeki belirli bir efsunu) `m_SwitchbotTable`'a kaydeder.
    *   **`GetAttribute(slot, alternative, attrIdx)`**: Belirli bir slot, alternatif ve efsun indeksindeki `TPlayerItemAttribute` verisini döndürür.
    *   **`GetAlternatives(slot, std::vector<TSwitchbotAttributeAlternativeTable>& vec_alternatives)`**: Belirli bir slottaki tüm efsun alternatiflerini (`TSwitchbotAttributeAlternativeTable`) bir vektöre kopyalar.
    *   **`GetAttibuteSet(uint8_t slot)`**: Verilen slottaki eşyanın türüne göre (örneğin, silah, zırh, kolye) hangi "efsun setine" ait olduğunu belirler. Bu, `CItemManager` ve `CPythonPlayer` kullanılarak eşya VNUM'u üzerinden eşya tipi ve alt tipi sorgulanarak yapılır. Dönen değere göre (`ATTRIBUTE_SET_WEAPON`, `ATTRIBUTE_SET_BODY` vb.) o eşyaya hangi efsunların gelebileceği belirlenir.
    *   **`IsActive(slot)` / `IsFinished(slot)`**: `m_SwitchbotTable` üzerinden slotun aktif veya bitmiş olup olmadığını döndürür.
    *   **`ClearSlot(slot)`**: Bir slotu temizler; aktif durumunu `false` yapar, eşya VNUM'unu sıfırlar ve o slota ait tüm alternatifleri sıfırlar.
    *   **`ClearAttributeMap()`**: Efsun seti-efsun eşleşmelerini tutan `m_map_AttributesBySet` haritasını temizler.
    *   **`AddAttributeToMap(const TSwitchbottAttributeTable& table)`**: Sunucudan veya bir yapılandırma dosyasından okunan, bir efsun setine ait efsun türünü (`apply_num`) ve maksimum değerini (`max_value`) `m_map_AttributesBySet` haritasına ekler.
    *   **`GetAttributesBySet(int iAttributeSet, std::vector<TPlayerItemAttribute>& vec_attributes)`**: Belirli bir efsun seti için `m_map_AttributesBySet` haritasından geçerli tüm efsunları (tür ve maksimum değerleriyle) bir vektöre doldurur. Bu, arayüzde "Efsun Seç" listesini doldurmak için kullanılır.
    *   **`GetAttributeMaxValue(int iAttributeSet, uint16_t attrType)`**: Belirli bir efsun seti ve efsun türü için maksimum efsun değerini `m_map_AttributesBySet`'ten okur.

*   **`switchbot` Python Modülü Fonksiyonları**:
    *   `switchbotIsActive(slot)`, `switchbotIsFinished(slot)`: C++ tarafındaki `IsActive` ve `IsFinished` fonksiyonlarını çağırır.
    *   `switchbotGetAttribute(slot, alternative, attrIdx)`: C++ `GetAttribute` fonksiyonunu çağırarak efsun türü ile değerini Python tuple `(type, value)` olarak döndürür.
    *   `switchbotSetAttribute(slot, alternative, attrIdx, type, value)`: C++ `SetAttribute` fonksiyonunu çağırarak Python'dan gelen efsun bilgilerini kaydeder.
    *   `switchbotClearSlot(slot)`: C++ `ClearSlot` fonksiyonunu çağırır.
    *   `switchbotGetAttributesForSet(slot)`: Önce `GetAttibuteSet` ile slottaki eşyanın efsun setini bulur, sonra `GetAttributesBySet` ile o sete ait tüm geçerli efsunları alır ve bunları Python listesi olarak `[(type1, maxValue1), (type2, maxValue2), ...]` formatında döndürür. Efsunlar türlerine göre sıralanır.
    *   `switchbotGetAttributeMaxValue(slot, attrType)`: Önce `GetAttibuteSet` ile efsun setini bulur, sonra `GetAttributeMaxValue` ile belirtilen efsun türünün o setteki maksimum değerini döndürür.
    *   `switchbotStart(slot)`: C++ `GetAlternatives` ile o slottaki tüm efsun alternatiflerini alır ve `CPythonNetworkStream::Instance().SendSwitchbotStartPacket()` fonksiyonunu çağırarak efsunlama işlemini başlatmak için sunucuya bir paket gönderir.
    *   `switchbotStop(slot)`: `CPythonNetworkStream::Instance().SendSwitchbotStopPacket()` fonksiyonunu çağırarak belirtilen slottaki efsunlama işlemini durdurmak için sunucuya paket gönderir.

*   **Modül Başlatma (`initSwitchbot()`)**:
    *   `switchbot` adında Python modülünü oluşturur ve yukarıda listelenen C++ fonksiyonlarını sarmalayan Python metodlarını kaydeder.
    *   `PyModule_AddIntConstant` ile `SLOT_COUNT` ve `ALTERNATIVE_COUNT` gibi sabitleri Python modülüne ekler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, `ENABLE_SWITCHBOT` sistemi aktif olduğunda, oyuncunun oyun içindeki efsun botu arayüzü (`uiSwitchbot.py`) ile etkileşim kurmasını ve efsunlama işlemlerini yönetmesini sağlar.
*   Arayüz, `switchbot` modülü fonksiyonlarını kullanarak slotlardaki eşyaları, kayıtlı efsun alternatiflerini, hangi slotların aktif/bitmiş olduğunu gösterir.
*   Oyuncu yeni bir efsun alternatifi eklediğinde veya mevcut birini değiştirdiğinde, `switchbot.SetAttribute` çağrılır.
*   Oyuncu bir eşya için efsun seçmek istediğinde, `switchbot.GetAttributesForSet` çağrılarak o eşya türü için geçerli efsunlar listelenir ve `switchbot.GetAttributeMaxValue` ile bu efsunların maksimum değerleri gösterilir.
*   Oyuncu "Başlat" butonuna tıkladığında, `switchbot.Start` çağrılarak sunucuya efsunlama işlemini başlatma isteği gönderilir.
*   "Durdur" butonuna tıklandığında ise `switchbot.Stop` çağrılır.
Sunucudan gelen efsun botu durumu güncellemeleri (`CPythonNetworkStream` üzerinden) `CPythonSwitchbot::Instance().Update()` ile işlenir ve arayüz bu güncel bilgilere göre kendini yeniler. 

---

### `PythonSystem.h` (Sistem Ayarları Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonSystem.h`, Metin2 istemcisinin temel sistem ayarlarını yöneten `CPythonSystem` singleton sınıfının başlık dosyasıdır. Bu dosya, grafik, ses, arayüz ve oyun içi çeşitli seçeneklerle ilgili yapıları (`TConfig`, `TResolution`, `TWindowStatus`), sabitleri (`EWindow`, `FREQUENCY_MAX_NUM`, `RESOLUTION_MAX_NUM`) ve `CPythonSystem` sınıfının arayüzünü tanımlar. İstemcinin yapılandırma dosyalarından (`metin2.cfg`, `interface.cfg`, karaktere özel `config` dosyaları) ayarları yüklemesi, kaydetmesi, uygulaması ve bu ayarlara C++ ve Python katmanlarından erişilmesi için gerekli altyapıyı sağlar.

**Temel Yapılar ve Fonksiyonlar**

*   **`struct SConfig`**: İstemcinin temel yapılandırma ayarlarını tutan ana yapıdır. İçerisinde grafik ayarları (çözünürlük, renk derinliği, frekans, yazılımsal imleç, nesne gizleme, görüş mesafesi, gölge seviyesi/kalitesi/hedefi, FOV), ses ayarları (müzik ve efekt ses seviyeleri), gamma, kayıtlı ID, pencere modu, DDS doku sıkıştırma, ses kartı yokluğu, varsayılan IME kullanımı, yazılımsal tiling, sohbet görünürlüğü, isimlerin her zaman gösterilmesi, hasar gösterimi, satış yazıları, sis modu, gece/kar modu, kar dokusu modu, mob seviyesi/AI bayrağı gösterimi, efekt/özel pazar/düşen eşya seviyesi, evcil hayvan/NPC isim durumu, özel pazar görüş mesafesi ve kamera modu gibi birçok ayar bulunur. Bu ayarların çoğu `#if defined(...)` direktifleriyle belirli sistemlerin aktif olup olmamasına göre derlenir.
*   **`struct SResolution`**: Ekran çözünürlüklerini (genişlik, yükseklik, BPP) ve desteklenen yenileme hızlarını (`frequency`) tutar.
*   **`struct SWindowStatus`**: Oyun içi pencerelerin (envanter, statü vb.) görünürlük, küçültülme durumu ve pozisyon bilgilerini saklar.
*   **`enum EWindow`**: `SWindowStatus` yapısında kullanılacak pencere indekslerini tanımlar (örneğin, `WINDOW_INVENTORY`).
*   **`CPythonSystem::Instance()`**: `CPythonSystem` singleton örneğine erişim sağlar.
*   **`LoadConfig()`, `SaveConfig()`**: `metin2.cfg` dosyasından ayarları yükler ve kaydeder.
*   **`LoadInterfaceStatus()`, `SaveInterfaceStatus()`**: `interface.cfg` dosyasından pencere durumlarını yükler ve kaydeder.
*   **`LoadCharConfig()`, `SaveCharConfig()`**: Karaktere özel yapılandırma ayarlarını (`UserData/config/<KarakterAdı>`) yükler ve kaydeder (eğer `CHAR_CONFIG` aktifse).
*   **`ApplyConfig()`**: Mevcut ayarları oyuna uygular (örneğin, gamma, imleç modu).
*   **`GetDisplaySettings()`**: Kullanılabilir ekran çözünürlüklerini ve frekanslarını listeler.
*   **Getter/Setter Fonksiyonları**: `SConfig` yapısındaki çeşitli ayarlara erişmek ve bunları değiştirmek için çok sayıda fonksiyon bulunur (örneğin, `GetWidth()`, `GetHeight()`, `SetMusicVolume()`, `GetShadowLevel()`, `IsWindowed()` vb.).
*   **`SetInterfaceHandler()`, `DestroyInterfaceHandler()`**: Python tarafındaki arayüz yöneticisi nesnesini (`PyObject*`) ayarlar veya kaldırır. Bu, C++ ve Python UI arasındaki iletişimi sağlar.

**Dosyanın Oyundaki Kullanım Amacı**

`PythonSystem.h`, oyuncunun oyun ayarları menüsünden yaptığı tüm değişikliklerin (grafik, ses, oyun seçenekleri vb.) C++ tarafında nasıl saklanacağını ve yönetileceğini tanımlar. Oyuncu bir ayarı değiştirdiğinde, bu değişiklikler `CPythonSystem` aracılığıyla `SConfig` yapısına yazılır, `metin2.cfg`'ye kaydedilir ve `ApplyConfig()` ile oyuna yansıtılır. Oyun başladığında bu ayarlar tekrar yüklenir. Pencere pozisyonları, kayıtlı kullanıcı ID'si gibi veriler de bu sistem üzerinden yönetilir ve Python arayüz betiklerinin bu bilgilere erişmesi sağlanır.

---

### `PythonSystem.cpp` (Sistem Ayarları Uygulaması)

**Dosyanın Amacı ve İşlevselliği**

`PythonSystem.cpp`, `PythonSystem.h` dosyasında tanımlanan `CPythonSystem` sınıfının fonksiyonlarını uygular. Bu dosya, istemcinin yapılandırma ayarlarının (`metin2.cfg`, `interface.cfg` ve karaktere özel konfigürasyon dosyaları) okunması, yazılması, varsayılan değerlerinin ayarlanması, mevcut ayarların oyun içinde aktif hale getirilmesi ve bu ayarlara erişim sağlanması gibi temel işlemleri gerçekleştirir. Grafik ayarları (çözünürlük, yenileme hızı, BPP), ses ayarları (müzik, efekt ses seviyeleri), oyun içi arayüz seçenekleri (sohbet, isim gösterme, hasar gösterimi vb.) ve diğer birçok sistem parametresi bu sınıf tarafından yönetilir.

**Ana Çalışma Prensipleri ve Önemli Fonksiyonlar**

*   **Yapılandırma Yönetimi (`TConfig` struct):**
    *   **`SetDefaultConfig()`**: `m_Config` üyesini varsayılan değerlerle doldurur (örneğin, 1024x768 çözünürlük, %100 müzik sesi). Birçok ayar, derleme zamanı makrolarıyla (`#if defined(...)`) farklı yerelleştirmeler veya özellikler için özelleştirilir.
    *   **`LoadConfig()`**: "metin2.cfg" dosyasını açar, satır satır okur ve komut-değer çiftlerini ayrıştırarak `m_Config` yapısını doldurur. Eskiden kalma uyumluluk için müzik ses seviyesi gibi bazı ayarların farklı formatlarını da işleyebilir. Pencere modunda ise çözünürlüğün ekran sınırlarını aşmamasını sağlar.
    *   **`SaveConfig()`**: `m_Config` yapısındaki mevcut ayarları "metin2.cfg" dosyasına belirli bir formatta yazar. Yalnızca varsayılan değerinden farklı olan bazı ayarlar dosyaya yazılır.
    *   **`ApplyConfig()`**: `m_OldConfig` (önceki ayarlar) ile `m_Config` (yeni ayarlar) arasındaki farkları kontrol eder ve değişen ayarları uygular. Örneğin, gamma değeri değiştiyse `CPythonGraphic::Instance().SetGamma()` çağrılır, yazılımsal imleç ayarı değiştiyse `CPythonApplication::Instance().SetCursorMode()` çağrılır. Sonrasında `ChangeSystem()` çağrılarak ses ayarları güncellenir.
    *   **`ChangeSystem()`**: Özellikle ses ayarlarını (`CSoundManager::Instance().SetMusicVolume()`, `CSoundManager::Instance().SetSoundVolumeGrade()`) günceller.
    *   **`GetConfig()`, `SetConfig()`**: Mevcut yapılandırma ayarlarına işaretçi döndürür veya yeni bir yapılandırma ayarları.

*   **Arayüz Durum Yönetimi (`TWindowStatus` struct):**
    *   **`LoadInterfaceStatus()`**: "interface.cfg" dosyasını okuyarak `m_WindowStatus` dizisini (oyun içi pencerelerin pozisyonu, görünürlüğü vb.) doldurur.
    *   **`SaveInterfaceStatus()`**: Python tarafındaki `m_poInterfaceHandler` üzerinden `OnSaveInterfaceStatus` fonksiyonunu çağırarak pencere durumlarını alır ve "interface.cfg" dosyasına yazar.
    *   **`SaveWindowStatus()`**: Belirli bir pencerenin durumunu (görünürlük, küçültülme, pozisyon, yükseklik) `m_WindowStatus` dizisinde günceller.
    *   **`GetWindowStatusReference()`**: Belirli bir pencerenin durum bilgisine referans döndürür.

*   **Ekran Çözünürlüğü ve Frekans Yönetimi (`TResolution` struct):**
    *   **`GetDisplaySettings()`**: Direct3D8 arayüzünü kullanarak sistemde mevcut olan ve oyunun desteklediği (minimum 800x600, 16 veya 32 BPP) tüm ekran çözünürlüklerini ve bu çözünürlükler için desteklenen yenileme hızlarını tesp_it eder. Bu bilgileri `m_ResolutionList` dizisinde saklar ve `m_ResolutionCount`'u günceller. Aynı çözünürlük ve BPP için birden fazla frekans varsa bunları da listeler.
    *   **`GetResolutionCount()`, `GetFrequencyCount()`**: Toplam çözünürlük sayısını veya belirli bir çözünürlük için frekans sayısını döndürür.
    *   **`GetResolution()`, `GetFrequency()`**: Belirli bir indeksteki çözünürlük veya frekans bilgisini döndürür.
    *   **`GetResolutionIndex()`, `GetFrequencyIndex()`**: Verilen genişlik, yükseklik, BPP veya frekans değerlerine karşılık gelen indeksi bulur.

*   **Karaktere Özel Ayarlar (`CHAR_CONFIG` makrosu aktifse):**
    *   **`CreateCharConfigPath()`**: `.\UserData\config\` yolunu oluşturur.
    *   **`LoadCharConfig()`**: Aktif karakterin ismine göre (örneğin, `.\UserData\config\KarakterAdi`) yapılandırma dosyasını okur.
    *   **`SaveCharConfig()`**: Aktif karakterin yapılandırma dosyasını yazar.
    *   Not: Linter, `GetFileAttributes` ve `CreateDirectory` fonksiyonlarının `const char*` yerine `LPCWSTR` (geniş karakter dizisi) beklediği konusunda hatalar göstermektedir. Bu, Unicode uyumluluğu ile ilgili bir durumdur ve modern Windows API'lerinde bu fonksiyonların `A` (ANSI) ve `W` (Wide) versiyonları bulunur. Kodda `A` versiyonlarının kullanılması veya uygun dönüşümlerin yapılması gerekebilir.

*   **Diğer Fonksiyonlar:**
    *   `SetInterfaceHandler()`, `DestroyInterfaceHandler()`: Python UI ile etkileşim için arayüz işleyiciyi ayarlar/kaldırır.
    *   Getter/Setter fonksiyonları: `GetWidth()`, `GetHeight()`, `GetBPP()`, `GetFrequency()`, `IsNoSoundCard()`, `IsSoftwareCursor()`, `GetMusicVolume()`, `SetMusicVolume()`, `GetShadowLevel()`, `SetShadowLevel()`, `IsSaveID()`, `GetSaveID()`, `SetSaveID()` gibi `CPythonSystem::TConfig` yapısındaki birçok alana erişim ve değişiklik imkanı sunar.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyunun "Sistem Seçenekleri" veya "Grafik Ayarları" gibi menülerinden yapılan tüm kullanıcı tercihlerinin kalıcı olarak saklanmasını ve oyun her başladığında bu tercihlerin yüklenerek uygulanmasını sağlar. Çözünürlük, ses seviyeleri, gölge detayları, isimlerin gösterilip gösterilmeyeceği gibi birçok ayar bu mekanizma ile yönetilir. Oyuncu ayarları değiştirdiğinde, bu değişiklikler önce `CPythonSystem` sınıfının belleğindeki `TConfig` yapısına yansıtılır, ardından `SaveConfig()` ile `metin2.cfg` dosyasına kaydedilir ve `ApplyConfig()` ile oyunun aktif durumuna etki eder. Oyun içi pencerelerin (envanter, sohbet vb.) pozisyonları ve durumları da benzer şekilde `interface.cfg` dosyasına kaydedilir ve yüklenir.

---

### `PythonSystemModule.cpp` (Sistem Ayarları Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonSystemModule.cpp`, `CPythonSystem` sınıfının C++ tarafında yönettiği sistem ayarlarına Python betikleri üzerinden erişilmesini ve bu ayarların Python'dan değiştirilebilmesini sağlayan bir Python modülü (`systemSetting`) tanımlar. Bu modül, `CPythonSystem` sınıfındaki fonksiyonları Python'a açarak (expose ederek) kullanıcı arayüzü (UI) gibi Python tabanlı sistemlerin oyunun temel yapılandırma verilerine ulaşmasına olanak tanır. Örneğin, oyun ayarları penceresi (genellikle Python ile yazılır) bu modül aracılığıyla mevcut çözünürlükleri listeleyebilir, ses seviyelerini okuyabilir/ayarlayabilir veya gölge seviyesini değiştirebilir.

**Temel Yapılar ve Fonksiyonlar**

*   **Python Metot Tanımları (`PyMethodDef s_methods[]`)**: Python'dan çağrılabilecek fonksiyonların bir listesini ve bu fonksiyonlara karşılık gelen C++ sarmalayıcı (wrapper) fonksiyonlarını içerir. Her bir girdi, Python'daki fonksiyon adını, C++ fonksiyon işaretçisini, argüman tipini (`METH_VARARGS` vb.) ve bir açıklama metnini (burada `NULL`) belirtir.
*   **Modül Başlatma Fonksiyonu (`initsystem()` veya `initsystemSetting()`):**
    *   `Py_InitModule("systemSetting", s_methods)`: "systemSetting" adında yeni bir Python modülü oluşturur ve `s_methods` dizisindeki fonksiyonları bu modüle kaydeder.
    *   `PyModule_AddIntConstant()`: Modüle, `CPythonSystem::EWindow` enum'undaki pencere türleri için tamsayı sabitleri ekler (örneğin, `systemSetting.WINDOW_STATUS`).

*   **C++ Sarmalayıcı (Wrapper) Fonksiyonları**:
    *   Her bir Python fonksiyonu için (örneğin, `systemGetWidth`, `systemSetMusicVolume`), C++ tarafındaki ilgili `CPythonSystem` metodunu çağıran ve Python veri türleri (`PyObject*`) ile C++ veri türleri arasında dönüşüm yapan bir sarmalayıcı fonksiyon bulunur.
    *   **Argüman Alma**: `PyTuple_GetInteger()`, `PyTuple_GetFloat()`, `PyTuple_GetString()`, `PyTuple_GetObject()` gibi fonksiyonlar Python tuple argümanlarından C++ türlerine veri çıkarır.
    *   **Değer Döndürme**: `Py_BuildValue()` fonksiyonu C++ türlerinden Python nesnelerine (tamsayı, float, string, tuple vb.) dönüşüm yaparak sonuçları Python'a döndürür. `Py_BuildNone()` ise Python'da `None` döndürür.
    *   **Örnekler:**
        *   `systemGetWidth()`: `CPythonSystem::Instance().GetWidth()` çağırır ve sonucu Python tamsayısı olarak döndürür.
        *   `systemSetMusicVolume(PyObject* poSelf, PyObject* poArgs)`: `poArgs`'tan bir float (ses seviyesi) alır, `CPythonSystem::Instance().SetMusicVolume()` ile ayarlar ve `None` döndürür.
        *   `systemGetConfig()`: `CPythonSystem::Instance().GetConfig()` ile ayarları alır ve çözünürlük indeksi, frekans indeksi, yazılımsal imleç durumu gibi birçok ayarı bir Python tuple'ı içinde döndürür.
        *   `systemSetConfig()`: Python'dan bir tuple içinde yapılandırma parametrelerini alır, `CPythonSystem::TConfig` yapısını günceller ve `CPythonSystem::Instance().SetConfig()` ile ayarlar.
        *   `systemSaveWindowStatus()`: Pencere indeksi ve durum bilgilerini Python'dan alır ve C++ tarafında kaydeder.
        *   `systemGetWindowStatus()`: Belirli bir pencerenin durumunu C++'tan okur ve Python tuple'ı olarak döndürür.
        *   Çok sayıda `#if defined(...)` bloğu, `ENABLE_SHADOW_RENDER_QUALITY_OPTION`, `ENABLE_FOG_FIX`, `ENABLE_ENVIRONMENT_EFFECT_OPTION`, `WJ_SHOW_MOB_INFO`, `ENABLE_GRAPHIC_ON_OFF`, `ENABLE_FOV_OPTION`, `ENABLE_PREMIUM_PRIVATE_SHOP`, `CHAR_CONFIG`, `ENABLE_OPTIMIZATION`, `ENABLE_SAVE_CAMERA_MODE` gibi farklı sistemlerin veya özelliklerin varlığına bağlı olarak ilgili Python fonksiyonlarını ekler veya çıkarır.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, oyunun Python ile yazılmış kullanıcı arayüzü betiklerinin (örneğin, sistem seçenekleri menüsü `uiSystemOption.py`) C++ motoru tarafından yönetilen temel sistem ayarlarına erişmesini ve bunları değiştirmesini sağlar. Örneğin, oyuncu oyun içindeki ayarlar menüsünden grafik çözünürlüğünü değiştirdiğinde, Python UI betiği `systemSetting.SetConfig()` gibi bir fonksiyonu çağırarak bu değişikliği C++ tarafına iletir. `CPythonSystem` bu değişikliği alır, uygular ve `metin2.cfg` dosyasına kaydeder. Benzer şekilde, mevcut ayarları görüntülemek için `systemSetting.GetConfig()` veya `systemSetting.GetMusicVolume()` gibi fonksiyonlar kullanılır. Bu, C++ çekirdeği ile Python UI katmanı arasında önemli bir köprü görevi görür. 

---

### `PythonTextTail.h` (Metin Kuyruğu Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonTextTail.h`, Metin2 istemcisindeki oyun içi varlıkların (karakterler, eşyalar, özel dükkanlar vb.) üzerinde görünen metin "kuyrukları" (text tails) için C++ tarafındaki temel veri yapılarını, sabitlerini ve `CPythonTextTail` singleton sınıfının arayüzünü tanımlar. Bu başlık dosyası, metin kuyruklarının oluşturulması, güncellenmesi, çizdirilmesi ve yönetilmesi için gerekli olan `STextTail` yapısını ve ilgili fonksiyon prototiplerini içerir.

**Temel Yapılar ve Fonksiyonlar**

*   **`STextTail` Yapısı**: Bir metin kuyruğunun tüm bilgilerini tutar:
    *   `pTextInstance`: Asıl metin (`CGraphicTextInstance`).
    *   `pOwnerTextInstance`: Eşya sahibi metni (varsa).
    *   `pDestroyTimeTextInstance`: Eşya yok olma süresi metni (varsa, `ENABLE_DROP_DESTROY_TIME` ile).
    *   `pMarkInstance`: Lonca sembolü (`CGraphicMarkInstance`).
    *   `pGuildNameTextInstance`: Lonca adı (`CGraphicTextInstance`).
    *   `pTitleTextInstance`: Karakter unvanı (`CGraphicTextInstance`).
    *   `pLevelTextInstance`: Karakter seviyesi (`CGraphicTextInstance`).
    *   `pAIFlagTextInstance`: Yaratık yapay zeka bayrağı (örneğin agresif, `WJ_SHOW_MOB_INFO` ile).
    *   `pLanguageInstance`: Oyuncu dil bayrağı (`CGraphicImageInstance`, `DISABLE_FLAG_COUNTRY` değilse).
    *   `pOwner`: Metin kuyruğunun sahibi olan grafik nesnesi (`CGraphicObjectInstance`).
    *   `dwVirtualID`: Sahip varlığın sanal ID'si.
    *   `x`, `y`, `z`: Metin kuyruğunun 2D ekran koordinatları ve derinlik.
    *   `fDistanceFromPlayer`: Oyuncuya olan uzaklığı.
    *   `Color`: Metin rengi (`D3DXCOLOR`).
    *   `bNameFlag`: Sohbet kuyruğu için isim gösterme bayrağı.
    *   `xStart`, `yStart`, `xEnd`, `yEnd`: Eşya metin kuyruğu için arka plan kutusunun sınırları.
    *   `LivingTime`: Sohbet kuyruğunun ekranda kalma süresi.
    *   `fHeight`: Metnin sahip nesne üzerindeki yükseklik ofseti.
    *   `bIsPC`: Sahibin oyuncu olup olmadığı (`WJ_SHOW_MOB_INFO` ile).
*   **`TTextTailMap`**: Sanal ID'den `TTextTail` işaretçisine eşleme (map).
*   **`TTextTailList`**: `TTextTail` işaretçilerinden oluşan liste.
*   **`CPythonTextTail` Sınıfı Fonksiyonları (Özet)**:
    *   `Initialize()`, `Destroy()`, `Clear()`: Başlatma, yok etme ve temizleme.
    *   `UpdateAllTextTail()`, `UpdateShowingTextTail()`: Tüm veya sadece görünürdeki kuyrukları güncelleme.
    *   `Render()`: Görünürdeki kuyrukları çizdirme.
    *   `ArrangeTextTail()`: Eşya metin kuyruklarının üst üste binmesini engellemek için düzenleme.
    *   `RegisterCharacterTextTail()`, `RegisterItemTextTail()`, `RegisterChatTail()`, `RegisterInfoTail()`: Farklı türde metin kuyrukları kaydetme.
    *   `RegisterPrivateShopTextTail()`: Özel pazar tezgahı için metin kuyruğu kaydetme (`ENABLE_PREMIUM_PRIVATE_SHOP` ile).
    *   `DeleteCharacterTextTail()`, `DeleteItemTextTail()`, `DeletePrivateShopTextTail()`: Metin kuyruklarını silme.
    *   `AttachTitle()`, `DetachTitle()`, `AttachLevel()`, `DetachLevel()`: Karakterlere unvan ve seviye metinleri ekleme/kaldırma.
    *   `SetItemTextTailOwner()`: Yerdeki eşyalara sahip bilgisi ekleme.
    *   `SetItemTextTailDestroyTime()`: Yerdeki eşyalara yok olma süresi ekleme (`ENABLE_DROP_DESTROY_TIME` ile).
    *   `Pick()`: Fare ile bir eşya metin kuyruğunun seçilip seçilmediğini kontrol etme.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, oyun dünyasındaki karakterlerin isimleri, seviyeleri, unvanları, lonca isimleri; yaratıkların isimleri ve seviyeleri; yerdeki eşyaların isimleri, sahipleri ve yok olma süreleri; oyuncuların sohbet mesajları gibi çeşitli metin bilgilerinin ekranda, ilgili varlıkların üzerinde doğru şekilde konumlandırılması, güncellenmesi ve çizdirilmesi için gerekli olan sınıf ve veri yapısı tanımlamalarını içerir. `CPythonTextTail` sınıfı, bu metin kuyruklarının merkezi yönetimini üstlenir.

---

### `PythonTextTail.cpp` (Metin Kuyruğu Uygulaması)

**Dosyanın Amacı ve İşlevselliği**

`PythonTextTail.cpp`, `CPythonTextTail` sınıfının fonksiyonlarının gerçeklemesini içerir. Bu dosya, oyun içi varlıkların (karakterler, canavarlar, eşyalar, özel dükkanlar vb.) üzerinde görünen metinlerin (isim, seviye, lonca, unvan, sohbet mesajları, eşya bilgileri vb.) yönetimi, güncellenmesi, konumlandırılması ve çizdirilmesi (render edilmesi) işlemlerini gerçekleştirir.

**Ana Çalışma Prensipleri ve Önemli Fonksiyonlar**

*   **Metin Kuyruğu Yönetimi**:
    *   `RegisterTextTail()`: Genel bir metin kuyruğu oluşturur ve havuza (`m_TextTailPool`) ekler. Temel bilgileri (sahip nesne, metin, yükseklik, renk) ayarlar.
    *   `DeleteTextTail()`: Bir metin kuyruğunu ve ona bağlı grafik örneklerini (metin, lonca işareti vb.) siler ve havuzdan serbest bırakır.
    *   `RegisterCharacterTextTail()`: Karakterler (oyuncular, NPC'ler) için isim, lonca adı/amblemi, unvan, seviye gibi bilgileri içeren metin kuyruklarını kaydeder.
        *   Lonca bilgileri `CGuildMarkManager` ve `CPythonGuild` üzerinden alınır.
        *   `ENABLE_GUILD_LEADER_GRADE_NAME` ile lonca lideri rütbe adı eklenebilir.
        *   `WJ_SHOW_MOB_INFO` ile canavarlar için agresiflik bayrağı (`*`) ve PC/NPC ayrımı için `bIsPC` bayrağı ayarlanır.
        *   `DISABLE_FLAG_COUNTRY` tanımlı değilse oyuncu dil bayrağı gösterilir.
    *   `RegisterItemTextTail()`: Yerdeki eşyalar için isimlerini içeren metin kuyruklarını kaydeder.
        *   `ENABLE_ITEM_DROP_RENEWAL` ile özel eşyalar için farklı renk (`c_TextTail_SpecialItem_Color`) kullanılabilir.
    *   `RegisterChatTail()`, `RegisterInfoTail()`: Karakterlerin sohbet ve bilgi mesajları için geçici metin kuyrukları oluşturur. Belirli bir süre (`gs_TextTail_LivingTime`) sonra kaybolurlar.
    *   `RegisterPrivateShopTextTail()`: Özel pazar tezgahları için isimlerini içeren metin kuyruklarını kaydeder (`ENABLE_PREMIUM_PRIVATE_SHOP` ile).
*   **Güncelleme ve Konumlandırma**:
    *   `UpdateAllTextTail()`: Oyuncunun pozisyonuna göre tüm kayıtlı metin kuyruklarının oyuncuya olan mesafesini günceller (`UpdateDistance`).
    *   `UpdateShowingTextTail()`: Sadece ekranda gösterilmekte olan metin kuyruklarının pozisyonlarını günceller (`UpdateTextTail`).
    *   `UpdateTextTail()`: Bir metin kuyruğunun sahibinin 3D pozisyonunu 2D ekran koordinatlarına yansıtır. Uzaklığa göre Z-derinliğini ayarlar (uzaktaki kuyruklar daha "arkada" görünür).
    *   `ArrangeTextTail()`: Özellikle yerdeki eşya isimlerinin üst üste binmesini engellemek için dikey konumlarını ayarlar. `isIn()` fonksiyonu ile çakışma kontrolü yapar. Karakter metin kuyruklarında ise lonca amblemi, lonca adı, unvan, seviye ve isim gibi bileşenlerin birbirine göre pozisyonlarını düzenler. Dil (Avrupa, Arapça vb.) ve yerel ayarlar (`LocaleService_IsEUROPE`, `GetDefaultCodePage`) dikkate alınarak düzenlemeler yapılır.
*   **Çizdirme (Render)**:
    *   `Render()`: Ekranda gösterilmesi gereken karakter, eşya ve sohbet metin kuyruklarını çizer.
        *   Karakterler için: İsim, lonca amblemi/adı, unvan, seviye, yapay zeka bayrağı (`WJ_SHOW_MOB_INFO`), dil bayrağı (`DISABLE_FLAG_COUNTRY` değilse) render edilir. `CPythonSystem` üzerinden mob seviyesi/AI bayrağı gösterim ayarları kontrol edilir.
        *   Eşyalar için: `RenderTextTailBox()` ile bir arka plan kutusu ve ardından eşya adı, sahip adı (`pOwnerTextInstance`), yok olma süresi (`pDestroyTimeTextInstance` ve `ENABLE_DROP_DESTROY_TIME`) render edilir.
        *   `ENABLE_CLOSE_ITEM_TEXT_TAIL_COLOR` ile yakındaki ve alınabilir eşyaların kutu rengi değişebilir.
        *   Sohbet kuyrukları için: `RenderTextTailName()` ile sadece metin render edilir, eğer sahibi görünür durumdaysa.
        *   Özel pazar isimleri (`ENABLE_PREMIUM_PRIVATE_SHOP` ile) render edilir.
*   **Görünürlük ve Etkileşim**:
    *   `ShowAllTextTail()`, `HideAllTextTail()`: Tüm metin kuyruklarını gösterme/gizleme listelerine ekler/çıkarır. Belirli bir mesafe (`3500.0f` veya `CPythonSystem::GetPrivateShopViewDistance()`) içindekiler gösterilir.
    *   `ShowCharacterTextTail()`, `ShowItemTextTail()`, `ShowPrivateShopTextTail()`: Belirli bir ID'ye sahip metin kuyruğunu görünür listesine ekler. Eğer zaten listedeyse veya sahibi görünmüyorsa işlem yapmaz. NPC isimlerinin gösterilip gösterilmeyeceği `ENABLE_GRAPHIC_ON_OFF` ve `CPythonSystem::instance().IsNpcNameStatus()` ile kontrol edilebilir. Yerdeki eşya isimlerinin gösterilip gösterilmeyeceği `ENABLE_GRAPHIC_ON_OFF` ve `CPythonSystem::instance().GetDropItemLevel()` ile kontrol edilebilir.
    *   `Pick()`: Verilen fare koordinatları ile hangi eşya metin kuyruğunun üzerine tıklandığını belirler.
    *   `SelectItemName()`: Seçilen eşya metin kuyruğunun rengini değiştirir.
*   **Ek Özellikler**:
    *   `SetCharacterTextTailColor()`: Bir karakterin metin kuyruğu rengini değiştirir.
    *   `SetItemTextTailOwner()`: Bir eşya metin kuyruğuna sahip adını ekler (örneğin, "OyuncuAdı'nın"). `ENABLE_ITEM_DROP_RENEWAL` ile sahip olan oyuncu için farklı renk kullanılabilir.
    *   `SetItemTextTailDestroyTime()`: Yerdeki bir eşyanın yok olma süresini gösteren bir metin ekler (`ENABLE_DROP_DESTROY_TIME` ile).
    *   `AttachTitle()`, `DetachTitle()`: Karakter metin kuyruğuna unvan ekler/kaldırır. `bPKTitleEnable` bayrağına bağlıdır.
    *   `AttachLevel()`, `DetachLevel()`: Karakter metin kuyruğuna seviye bilgisi ekler/kaldırır. `bPKTitleEnable` bayrağına bağlıdır.
    *   `AttachConquerorLevel()`: Fetih seviyesi ekler (`ENABLE_CONQUEROR_LEVEL` ile).
    *   `EnablePKTitle()`: Unvan ve seviye gösterimini genel olarak aktif eder/kapatır.
*   **Başlatma ve Temizleme**:
    *   `Initialize()`: Kullanılacak varsayılan fontu (`ms_pFont`) yükler.
    *   `Destroy()`, `Clear()`: Tüm metin kuyruklarını ve kullanılan kaynakları (özellikle `m_TextTailPool`) temizler.

**Sabitler ve Global Değişkenler**

*   `c_TextTail_Player_Color`, `c_TextTail_Monster_Color`, `c_TextTail_Item_Color`, `c_TextTail_SpecialItem_Color`, `c_TextTail_Chat_Color`, `c_TextTail_Info_Color`, `c_TextTail_Guild_Name_Color`: Farklı metin türleri için varsayılan renkler.
*   `c_TextTail_Name_Position`, `c_fxMarkPosition`, `c_fyGuildNamePosition`, `c_fyMarkPosition`: Metin bileşenlerinin göreceli konumları için sabitler.
*   `bPKTitleEnable`: Unvanların gösterilip gösterilmeyeceğini belirleyen global bayrak.
*   `gs_TextTail_LivingTime`: Sohbet kuyruklarının varsayılan yaşam süresi.
*   `ms_pFont`: Metin kuyrukları için kullanılan global font nesnesi.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyundaki tüm dinamik metin gösterimlerinin (oyuncu/NPC isimleri, canavar isimleri, eşya isimleri, sohbet balonları, lonca adları, unvanlar, seviyeler vb.) 3D dünyada doğru bir şekilde konumlandırılmasını, güncellenmesini ve oyuncuya sunulmasını sağlar. Oyuncunun etrafındaki varlıklara ait bilgilerin ve iletişimlerin görsel olarak takip edilebilmesi için kritik bir rol oynar. Çeşitli sistem (`ENABLE_...` makroları) entegrasyonları ile oyunun farklı özelliklerine uyum sağlar.

---

### `PythonTextTailModule.cpp` (Metin Kuyruğu Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonTextTailModule.cpp`, `CPythonTextTail` C++ sınıfının fonksiyonlarını Python betiklerine açan bir Python modülü tanımlar. Bu sayede, kullanıcı arayüzü (UI) gibi Python tarafında çalışan sistemler, C++ tarafındaki metin kuyruğu yönetim sistemine erişebilir ve onu kontrol edebilir.

**Python'a Açılan Fonksiyonlar**

*   **`textTailClear()`**: `CPythonTextTail::Instance().Clear()` fonksiyonunu çağırarak tüm metin kuyruklarını temizler.
*   **`textTailUpdateAllTextTail()`**: `CPythonTextTail::Instance().UpdateAllTextTail()` fonksiyonunu çağırarak tüm metin kuyruklarının güncellenmesini tetikler.
*   **`textTailUpdateShowingTextTail()`**: `CPythonTextTail::Instance().UpdateShowingTextTail()` fonksiyonunu çağırarak sadece görünürdeki metin kuyruklarının güncellenmesini tetikler.
*   **`textTailRender()`**: `CPythonTextTail::Instance().Render()` fonksiyonunu çağırarak görünürdeki metin kuyruklarını çizer.
*   **`textTailRegisterCharacterTextTail(iGuildID, [szGradeName,] iVirtualID)`**: Bir karakter için metin kuyruğu kaydeder. `ENABLE_GUILD_LEADER_GRADE_NAME` tanımlıysa lonca lideri rütbe adını da alır.
*   **`textTailGetPosition(VirtualID)`**: Belirli bir sanal ID'ye sahip varlığın metin kuyruğunun 2D ekran pozisyonunu (x, y, z) döndürür. Eğer kuyruk yoksa, ana karakterin pozisyonunu yansıtarak bir pozisyon döndürür.
*   **`textTailIsChat(VirtualID)`**: Belirli bir sanal ID için aktif bir sohbet kuyruğu olup olmadığını kontrol eder.
*   **`textTailRegisterChatTail(VirtualID, szText)`**: Belirli bir sanal ID'li karakter için sohbet kuyruğu kaydeder.
*   **`textTailRegisterInfoTail(VirtualID, szText)`**: Belirli bir sanal ID'li karakter için bilgi kuyruğu (genellikle sistem mesajları) kaydeder.
*   **`textTailAttachTitle(iVirtualID, szName, fr, fg, fb)`**: Belirli bir karaktere belirtilen renklerde bir unvan metni ekler.
*   **`textTailShowCharacterTextTail(VirtualID)`**: Belirli bir karakterin metin kuyruğunu görünür hale getirir.
*   **`textTailShowItemTextTail(VirtualID)`**: Belirli bir eşyanın metin kuyruğunu görünür hale getirir.
*   **`textTailArrangeTextTail()`**: Metin kuyruklarını (özellikle eşya isimlerini) düzenleyerek çakışmaları önler.
*   **`textTailHideAllTextTail()`**: Tüm metin kuyruklarını gizler.
*   **`textTailShowAllTextTail()`**: Mesafe kontrolüne göre uygun metin kuyruklarını gösterir.
*   **`textTailPick(ix, iy)`**: Verilen fare koordinatları (ix, iy) ile hangi eşya metin kuyruğunun seçildiğini döndürür (eşya ID'si veya -1).
*   **`textTailSelectItemName(iVirtualID)`**: Belirli bir eşya metin kuyruğunun rengini değiştirerek seçildiğini belirtir.
*   **`textTailEnablePKTitle(iFlag)`**: Unvan ve seviye gösterimini genel olarak aktif eder (1) veya kapatır (0).

**Modül Başlatma**

*   **`initTextTail()`**: Python modülünü (`textTail`) ve yukarıda listelenen C fonksiyonlarını Python metotları olarak kaydeder. Bu fonksiyon, istemci başladığında `CPythonLauncher` tarafından çağrılır.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, Python ile yazılmış UI betiklerinin (örneğin, karakter isimlerinin, sohbet balonlarının, eşya isimlerinin gösterilmesi ve güncellenmesi) C++ tarafındaki metin kuyruğu sistemini yönetmesine olanak tanır. Oyuncunun gördüğü ve etkileşimde bulunduğu hemen hemen tüm oyun içi metinlerin dinamik olarak oluşturulması, pozisyonlanması ve ekranda sergilenmesi bu modül aracılığıyla C++ ve Python katmanları arasındaki köprü sayesinde mümkün olur. Örneğin, bir oyuncu konuştuğunda, UI (`chat.py` gibi) bu modül aracılığıyla `textTailRegisterChatTail` fonksiyonunu çağırarak sohbet balonunu oluşturur.

---

### `PythonWhisper.h` (Fısıltı Sistemi Yenileme Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonWhisper.h` başlık dosyası, `ENABLE_WHISPER_RENEWAL` makrosu aktif olduğunda derlenen "Fısıltı Yenileme" sisteminin C++ tarafındaki `CPythonWhisper` singleton sınıfının arayüzünü tanımlar. Bu sistem, oyuncuların kimlere fısıltı gönderdiğini (ME) ve kimlerden fısıltı beklediğini (TARGET) takip etmek için kullanılır. Bu, genellikle oyuncu bir başkasına fısıltı göndermeye çalıştığında, karşı tarafın fısıltıları kabul edip etmediğini veya oyuncunun fısıltı ayarlarını yönetmek için bir arayüzde kullanılır.

**Ana Yapılar ve Tanımlamalar**

*   **`CPythonWhisper` (Singleton Sınıfı)**:
    *   **`enum { TARGET, ME }`**: İki tür fısıltı listesini ayırt etmek için kullanılır. `TARGET`, oyuncunun fısıltı beklediği kişilerin listesini; `ME`, oyuncunun fısıltı gönderdiği kişilerin listesini ifade eder.
    *   **`std::list<std::string> b_writing[ME+1]`**: İki adet string listesi tutan bir dizi. `b_writing[TARGET]` hedef oyuncuların isimlerini, `b_writing[ME]` ise oyuncunun fısıltı gönderdiği kişilerin isimlerini saklar.
    *   **`CPythonWhisper()` (Kurucu)**: `Clear()` fonksiyonunu çağırarak listeleri temizler.
    *   **`~CPythonWhisper()` (Yıkıcı)**: `Clear()` fonksiyonunu çağırarak listeleri temizler.
    *   **`Clear()`**: Her iki fısıltı listesini (`b_writing[TARGET]` ve `b_writing[ME]`) temizler.
    *   **`AddName(std::string name, BYTE i)`**: Verilen ismi, belirtilen listeye (`i` parametresi `TARGET` veya `ME` olabilir) ekler. Eğer isim listede zaten varsa tekrar eklemez.
    *   **`DeleteName(std::string name, BYTE i)`**: Verilen ismi, belirtilen listeden siler. Eğer isim listede yoksa bir işlem yapmaz.
    *   **`CheckName(std::string name, BYTE i)`**: Verilen ismin, belirtilen listede olup olmadığını kontrol eder ve boolean bir değer döndürür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, `ENABLE_WHISPER_RENEWAL` sistemi aktif olduğunda, fısıltı etkileşimlerini daha kontrollü bir şekilde yönetmek için C++ tarafında bir altyapı sunar. Özellikle, bir oyuncunun kimlere fısıltı gönderdiğini ve kimlerden fısıltı almaya açık olduğunu takip ederek, istemci tarafında fısıltı arayüzlerinin (`uiWhisper.py` gibi) bu bilgilere göre davranmasını sağlar. Örneğin, bir oyuncu fısıltı penceresini açtığında, bu sınıftaki `CheckName` fonksiyonu ile karşı tarafın fısıltıları kabul edip etmediği kontrol edilebilir veya oyuncunun daha önce kimlere fısıltı gönderdiği listelenebilir.

---

### `PythonWhisper.cpp` (Fısıltı Sistemi Yenileme Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonWhisper.cpp` dosyası, `ENABLE_WHISPER_RENEWAL` makrosu tanımlı olduğunda derlenir. `CPythonWhisper` sınıfının C++ implementasyonunu (bu dosya özelinde `PythonWhisper.h` içinde inline olarak tanımlanmış) ve `whisper` adlı bir Python modülünün tanımlanmasını içerir. Bu Python modülü, yenilenmiş fısıltı sisteminin işlevlerini Python betiklerine açar. Bu sayede, Python arayüzü (`uiWhisper.py` gibi) oyuncuların fısıltı gönderme/alma durumlarını yönetebilir ve sunucuyla iletişim kurabilir.

**Ana Çalışma Prensipleri ve Python Modülü Fonksiyonları**

*   **`CPythonWhisper` Sınıfı Kullanımı**:
    *   Python'a açılan fonksiyonlar, `CPythonWhisper::Instance()` aracılığıyla singleton nesnesine erişir ve `AddName`, `DeleteName`, `CheckName` gibi metodlarını kullanır.

*   **Python Modülü (`whisper`) Fonksiyonları**:
    *   **`AddName(Name)`**:
        *   Argüman olarak bir oyuncu adı (`Name`) alır.
        *   `CPythonNetworkStream::Instance().SendWhisperPacket(Name, "|?whisper_renewal>|")` fonksiyonunu çağırarak sunucuya, belirtilen oyuncuya fısıltı gönderme isteği (veya fısıltı almaya başlama durumu) ile ilgili özel bir paket gönderir. `|?whisper_renewal>|` özel bir komut stringidir.
        *   `CPythonWhisper::Instance().AddName(Name, CPythonWhisper::ME)` çağırarak, bu oyuncunun `Name` adlı kişiye fısıltı gönderdiğini kendi listesine (`ME` listesine) kaydeder.
    *   **`DeleteName(Name)`**:
        *   Argüman olarak bir oyuncu adı (`Name`) alır.
        *   `CPythonNetworkStream::Instance().SendWhisperPacket(Name, "|?whisper_renewal<|")` fonksiyonunu çağırarak sunucuya, belirtilen oyuncuya fısıltı göndermeyi durdurma isteği (veya fısıltı almayı durdurma durumu) ile ilgili özel bir paket gönderir. `|?whisper_renewal<|` özel bir komut stringidir.
        *   `CPythonWhisper::Instance().DeleteName(Name, CPythonWhisper::ME)` çağırarak, bu oyuncunun artık `Name` adlı kişiye fısıltı göndermediğini kendi listesinden (`ME` listesinden) siler.
    *   **`CheckName(Name)`**:
        *   Argüman olarak bir oyuncu adı (`Name`) alır.
        *   `CPythonWhisper::Instance().CheckName(Name, CPythonWhisper::TARGET)` fonksiyonunu çağırarak, `Name` adlı oyuncunun, fısıltı almayı kabul edenler listesinde (yani `TARGET` listesinde) olup olmadığını kontrol eder. Sonucu (1 veya 0) Python'a döndürür. Bu, genellikle "Bu oyuncu sizden fısıltı alıyor mu?" bilgisini kontrol etmek için kullanılır.
    *   **`IsSended(Name)`**:
        *   Argüman olarak bir oyuncu adı (`Name`) alır.
        *   `CPythonWhisper::Instance().CheckName(Name, CPythonWhisper::ME)` fonksiyonunu çağırarak, oyuncunun kendisinin `Name` adlı kişiye fısıltı gönderip göndermediğini (yani `ME` listesinde olup olmadığını) kontrol eder. Sonucu (1 veya 0) Python'a döndürür. Bu, "Bu oyuncuya zaten bir fısıltı isteği gönderdiniz mi?" bilgisini kontrol etmek için kullanılır.

*   **Modül Başlatma (`initWhisper()`)**:
    *   `whisper` adında Python modülünü oluşturur.
    *   Yukarıda listelenen C++ fonksiyonlarını sarmalayan Python metodlarını (`Add`, `Remove`, `CheckName`, `IsSended`) bu modüle kaydeder.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, `ENABLE_WHISPER_RENEWAL` sistemi aktif olduğunda, oyuncuların fısıltı etkileşimlerini daha gelişmiş bir şekilde yönetmelerini sağlar. Kullanıcı arayüzü (`uiWhisper.py`), bu `whisper` Python modülü aracılığıyla:
*   Bir oyuncuya fısıltı göndermeye başlamak için `whisper.Add(targetName)` çağırabilir. Bu, hem sunucuya durumu bildirir hem de yerel listeyi günceller.
*   Bir oyuncuya fısıltı göndermeyi durdurmak için `whisper.Remove(targetName)` çağırabilir.
*   Hedef oyuncunun fısıltı almaya açık olup olmadığını `whisper.CheckName(targetName)` ile sorgulayabilir.
*   Kendisinin hedef oyuncuya zaten bir "fısıltı açık" isteği gönderip göndermediğini `whisper.IsSended(targetName)` ile sorgulayabilir.
Bu sistem, oyuncular arası iletişimi ve fısıltı ayarlarının yönetimini kolaylaştırmayı amaçlar. Sunucudan gelen `|?whisper_renewal>|` ve `|?whisper_renewal<|` paketlerine göre de `CPythonWhisper::Instance()`'ın `TARGET` listesi güncellenir (bu kısım `PythonNetworkStreamModule.cpp` veya benzeri bir yerde işlenir).

---

### `resource.h` (Kaynak Tanımlama Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`resource.h`, Microsoft Visual C++ tarafından otomatik olarak üretilen ve kullanılan bir başlık dosyasıdır. `UserInterface.rc` (Kaynak Komut Dosyası) içinde tanımlanan kaynaklar için sembolik sabitler (ID'ler) içerir. Bu kaynaklar genellikle diyaloglar, ikonlar, imleçler, string tabloları ve diğer kullanıcı arayüzü elemanlarıdır. Programın C++ kodu, bu ID'leri kullanarak ilgili kaynaklara erişir.

**Ana İçerik Türleri**

*   **String Tablosu ID'leri (`IDS_...`)**:
    *   `IDS_APP_NAME`: Uygulama adı.
    *   `IDS_POSSESSIVE_MORPHENE`: İyelik eki (örneğin, "'s" veya yerelleştirilmiş karşılığı).
    *   `IDS_WARN_BAD_DRIVER`: Kötü sürücü uyarısı.
    *   `IDS_WARN_NO_TNL`: Donanımsal T&L (Transform and Lighting) desteği olmadığı uyarısı.
    *   `IDS_ERR_CANNOT_READ_FILE`: Dosya okunamadı hatası.
    *   `IDS_ERR_NOT_LATEST_FILE`: Dosya güncel değil hatası.
    *   `IDS_ERR_MUST_LAUNCH_FROM_PATCHER`: Oyunun patcher üzerinden başlatılması gerektiği hatası.
    Bu stringler `CStringTable` veya `ApplicationStringTable_GetString()` gibi fonksiyonlarla kod içinden okunarak kullanıcıya gösterilir.

*   **İkon ID'leri (`IDI_...`)**:
    *   `IDI_METIN2`: Uygulamanın ana ikonu.

*   **Diyalog ID'leri (`IDD_...`)**:
    *   `IDD_COLOR_DIALOG`: Renk seçimi için bir diyalog şablonu.
    *   `IDD_SELECT_LOCALE`: Bölge/dil seçimi için bir diyalog şablonu.

*   **İmleç ID'leri (`IDC_CURSOR_...`)**:
    *   `IDC_CURSOR_NORMAL`: Normal fare imleci.
    *   `IDC_CURSOR_ATTACK`: Saldırı imleci.
    *   `IDC_CURSOR_CHAIR`: Sandalyeye oturma imleci.
    *   `IDC_CURSOR_DOOR`: Kapı açma imleci.
    *   `IDC_CURSOR_NO`: İzin verilmeyen işlem imleci.
    *   `IDC_CURSOR_PICK`, `IDC_CURSOR_PICKUP`: Eşya toplama imleci.
    *   `IDC_CURSOR_TALK`: NPC ile konuşma imleci.
    *   `IDC_CURSOR_BUY`, `IDC_CURSOR_SELL`: Alış/Satış imleci.
    *   `IDC_CURSOR_CAMERA_ROTATE`: Kamera döndürme imleci.
    *   `IDC_CURSOR_HSIZE`, `IDC_CURSOR_VSIZE`, `IDC_CURSOR_HVSIZE`: Yeniden boyutlandırma imleçleri.
    *   `IDC_CURSOR_FISH`: Balık tutma imleci.
    Bu ID'ler `CPythonApplication::SetCursor()` gibi fonksiyonlarla fare imlecini değiştirmek için kullanılır.

*   **Vertex Shader Kaynak ID'leri (`IDR_VERTEXSHADER_...`)**:
    *   `IDR_VERTEXSHADER_LEAF_DYNAMIC_LIGHTING`, `IDR_VERTEXSHADER_LEAF_STATIC_LIGHTING` vb.: SpeedTree gibi bitki örtüsü render sistemleri için kullanılan vertex shader programlarının kaynak ID'leri.

*   **Diyalog Kontrol ID'leri (`IDC_...`)**:
    *   `IDC_MATERIAL_AMBIENT_SLIDER_RED` vb.: `IDD_COLOR_DIALOG` içindeki kaydırma çubukları, metin kutuları gibi kontrollerin ID'leri.
    *   `IDC_START`, `IDC_EXIT`: Genellikle bir başlangıç veya yerel seçim diyalogundaki butonlar.
    *   `IDC_LOCALE_LIST`: `IDD_SELECT_LOCALE` diyalogundaki yerel listesi kontrolü.

*   **Otomatik Oluşturulan Değerler**:
    *   `_APS_NEXT_RESOURCE_VALUE`, `_APS_NEXT_COMMAND_VALUE`, `_APS_NEXT_CONTROL_VALUE`, `_APS_NEXT_SYMED_VALUE`: Visual Studio kaynak düzenleyicisinin yeni kaynaklar için bir sonraki varsayılan değeri takip etmesine yardımcı olan semboller.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya doğrudan oyun mantığını etkilemese de, `UserInterface.dll` (veya projenin derlenmiş hali) içindeki gömülü kaynaklara C++ kodu üzerinden erişmek için bir köprü görevi görür. Oyunun kullanıcıya gösterdiği metinlerin (hatalar, uyarılar), kullandığı ikonların, fare imleçlerinin ve bazı özel efektler için shader'ların programatik olarak yüklenmesini ve kullanılmasını sağlar. Kaynak düzenleyici (örneğin Visual Studio'daki) bu dosyayı `UserInterface.rc` ile senkronize tutar. Yeni bir kaynak eklendiğinde veya bir ID değiştirildiğinde bu dosya güncellenir.

---

### `ServerStateChecker.h` (Sunucu Durum Denetleyicisi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`ServerStateChecker.h`, Metin2 istemcisinin sunucu listesindeki kanalların (channel) durumlarını (online, offline, yoğunluk vb.) periyodik olarak kontrol etmesini sağlayan `CServerStateChecker` singleton sınıfının arayüzünü tanımlar. Bu sınıf, genellikle sunucu seçme ekranında veya karakter seçme ekranında, oyuncuya kanalların güncel durumlarını göstermek için kullanılır.

**Ana Yapılar ve Tanımlamalar**

*   **`struct SChannel` (TChannel olarak typedef edilir)**:
    *   Bir sunucu kanalının bilgilerini tutar.
    *   `uServerIndex`: Kanalın Python tarafında kullanılan benzersiz indeksi/kimliği.
    *   `c_szAddr`: Kanalın IP adresi (string).
    *   `uPort`: Kanalın port numarası.

*   **`CPythonServerStateChecker` (Singleton Sınıfı)**:
    *   **Üye Değişkenler**:
        *   `m_poWnd`: Python tarafındaki bir UI pencere nesnesine (genellikle sunucu listesi veya karakter seçme ekranı) işaretçi (`PyObject*`). Sunucu durumları alındığında bu pencerenin ilgili fonksiyonları çağrılır.
        *   `m_lstChannel`: Durumu kontrol edilecek kanalların bir listesi (`std::list<TChannel>`).
        *   `m_kStream`: Sunucularla iletişim kurmak için kullanılan bir `CNetworkStream` nesnesi.
    *   **Temel Fonksiyonlar**:
        *   `CServerStateChecker()`: Kurucu, `Initialize()` fonksiyonunu çağırır.
        *   `~CServerStateChecker()`: Yıkıcı, `Initialize()` fonksiyonunu çağırır ve `m_poWnd`'yi `NULL` yapar.
        *   `Create(PyObject* poWnd)`: Python UI pencere nesnesini (`m_poWnd`) ayarlar.
        *   `AddChannel(UINT uServerIndex, const char* c_szAddr, UINT uPort)`: Kontrol edilecekler listesine yeni bir kanal ekler.
        *   `Request()`: Kayıtlı kanalların durumunu sorgulamak için ilk kanala bağlanır ve bir istek paketi gönderir.
        *   `Update()`: Ağ akışını işler (`m_kStream.Process()`), sunucudan gelen kanal durumu cevaplarını alır ve `m_poWnd` üzerinden Python UI'yi bilgilendirir.
        *   `Initialize()`: Kanal listesini (`m_lstChannel`) temizler ve ağ bağlantısını (`m_kStream.Disconnect()`) keser.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, istemcinin sunucu kanallarının durumunu aktif olarak sorgulaması için gerekli C++ sınıfının temel yapısını ve fonksiyonlarını tanımlar. Oyuncu sunucu listesi ekranına geldiğinde, bu sınıf aracılığıyla listelenen tüm kanalların IP ve port bilgileri kaydedilir. Ardından `Request()` çağrılarak durum sorgusu başlatılır. `Update()` fonksiyonu periyodik olarak (genellikle oyun döngüsü içinde) çağrılarak sunucudan gelen cevaplar işlenir ve Python tarafındaki arayüze (`m_poWnd`) kanalların çevrimiçi olup olmadığı veya yoğunluk durumu gibi bilgiler iletilir. Bu, oyuncuya güncel ve doğru sunucu durumu bilgisi sunulmasını sağlar.

---

### `ServerStateChecker.cpp` (Sunucu Durum Denetleyicisi Uygulaması)

**Dosyanın Amacı ve İşlevselliği**

`ServerStateChecker.cpp`, `CServerStateChecker` sınıfının fonksiyonlarının implementasyonunu içerir. Bu dosya, sunucu kanallarının durumlarını sorgulamak için ağ iletişimi kurma, istek gönderme, gelen cevapları işleme ve sonuçları Python arayüzüne bildirme mantığını yönetir.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **Paket Yapıları (Dosya içinde tanımlı)**:
    *   `ServerStateChecker_Header`: Paket başlığı türü (`unsigned char`).
    *   `ServerStateChecker_Key`, `ServerStateChecker_Index`: Kullanılmıyor gibi görünen tür tanımları.
    *   `ServerStateChecker_State`: Kanal durumunu belirten tür (`unsigned char`).

*   **`CServerStateChecker::Create(PyObject* poWnd)`**:
    *   Python tarafından gönderilen UI pencere nesnesini (`PyObject*`) `m_poWnd` üyesine atar. Bu pencere, sunucu durumları hakkında bilgilendirilecek olan Python nesnesidir.

*   **`CServerStateChecker::AddChannel(UINT uServerIndex, const char* c_szAddr, UINT uPort)`**:
    *   Yeni bir `TChannel` yapısı oluşturur ve verilen sunucu indeksi, IP adresi ve port numarası ile doldurur.
    *   Bu `TChannel` yapısını `m_lstChannel` listesine ekler.

*   **`CServerStateChecker::Request()`**:
    *   Eğer `m_lstChannel` boşsa (kontrol edilecek kanal yoksa) işlem yapmadan çıkar.
    *   Listenin ilk kanalına (`m_lstChannel.begin()`) `m_kStream.Connect()` ile bağlanmaya çalışır.
        *   Bağlantı başarısız olursa, `m_lstChannel` içindeki tüm kanallar için `m_poWnd` üzerindeki "NotifyChannelState" Python fonksiyonunu çağırarak durumlarını 0 (çevrimdışı) olarak bildirir ve fonksiyondan çıkar.
    *   Bağlantı başarılı olursa:
        *   `m_kStream.ClearRecvBuffer()` ile alım tamponunu temizler.
        *   Gönderme ve alma tampon boyutlarını ayarlar (`SetSendBufferSize`, `SetRecvBufferSize`).
        *   `HEADER_CG_STATE_CHECKER` başlığını (`BYTE bHeader`) içeren bir paket gönderir.
            *   Paket gönderimi başarısız olursa, tüm kanallar için durumu 0 olarak bildirir, `Initialize()` çağırır ve çıkar.

*   **`CServerStateChecker::Update()`**:
    *   `m_kStream.Process()` ile ağ olaylarını işler (gelen veriyi alır, bağlantı durumunu kontrol eder).
    *   Gelen veriden ilk olarak bir `BYTE bHeader` okumaya çalışır. Okuma başarısızsa çıkar.
    *   Okunan başlığın `HEADER_GC_RESPOND_CHANNELSTATUS` olup olmadığını kontrol eder. Değilse çıkar.
    *   Ardından bir `int nSize` (gelen kanal durumu sayısını) okumaya çalışır. Okuma başarısızsa çıkar.
    *   `nSize` kadar döngüye girer:
        *   Her döngüde bir `TChannelStatus` yapısı (genellikle `nPort` ve `bStatus` içerir, `Packet.h` içinde tanımlıdır) okumaya çalışır. Okuma başarısızsa çıkar.
        *   Okunan `channelStatus.nPort` değerini `m_lstChannel` listesindeki kanalların portlarıyla karşılaştırır.
        *   Eşleşen kanalı bulduğunda, `PyCallClassMemberFunc` ile `m_poWnd` üzerindeki "NotifyChannelState" Python fonksiyonunu çağırır. Bu çağrı, Python'a kanalın indeksini (`it->uServerIndex`) ve sunucudan gelen durumu (`channelStatus.bStatus`) iletir.
    *   Tüm cevaplar işlendikten sonra `Initialize()` çağrılarak mevcut bağlantı kesilir ve kanal listesi temizlenir (bir sonraki `Request` için hazırlanır).

*   **`CServerStateChecker::Initialize()`**:
    *   `m_lstChannel` listesini temizler.
    *   `m_kStream.Disconnect()` ile mevcut ağ bağlantısını sonlandırır.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, istemcinin sunucu seçme ekranında gösterilen kanalların (örneğin CH1, CH2) durumlarını (Yoğun, Normal, Kapalı gibi) periyodik olarak sorgulamasını sağlar.
1.  Python tarafındaki sunucu seçme arayüzü (`uiSelectServer.py` veya benzeri) başlatıldığında, `ServerStateCheckerCreate` ile bir Python pencere nesnesi kaydedilir.
2.  Arayüz, gösterilecek her bir kanal için `ServerStateCheckerAddChannel` ile kanal bilgilerini (indeks, IP, port) C++ tarafına ekler.
3.  `ServerStateCheckerRequest` çağrılarak C++ tarafının sunucuya bağlanıp durum sorgusu yapması tetiklenir.
4.  Oyun döngüsünde düzenli olarak `ServerStateCheckerUpdate` çağrılır. Bu fonksiyon, sunucudan gelen cevapları alır ve her bir kanalın durumunu `NotifyChannelState` aracılığıyla Python arayüzüne bildirir.
5.  Python arayüzü, bu bilgileri alarak ekrandaki kanal listesini günceller (örneğin, kanalın yanındaki durumu gösteren metni veya rengi değiştirir).
Bu mekanizma, oyuncuların sunucuya bağlanmadan önce kanalların güncel durumlarını görmelerini sağlar.

---

### `ServerStateCheckerModule.cpp` (Sunucu Durum Denetleyicisi Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`ServerStateCheckerModule.cpp`, `CServerStateChecker` C++ sınıfının fonksiyonlarını Python betiklerine açan bir Python modülü (`ServerStateChecker`) tanımlar. Bu modül, Python tarafındaki kullanıcı arayüzünün (genellikle sunucu seçme ekranı) C++ tarafındaki sunucu durum denetleme mekanizmasıyla etkileşim kurmasını sağlar.

**Python'a Açılan Fonksiyonlar**

*   **`ServerStateCheckerCreate(poWnd)`**:
    *   Argüman olarak bir Python pencere nesnesi (`poWnd`) alır.
    *   `CServerStateChecker::Instance().Create(poWnd)` fonksiyonunu çağırarak bu Python nesnesini C++ tarafına kaydeder. Bu nesne, sunucu durumları güncellendiğinde C++ tarafından bilgilendirilecek olan Python UI elemanıdır.

*   **`ServerStateCheckerUpdate()`**:
    *   `CServerStateChecker::Instance().Update()` fonksiyonunu çağırarak C++ tarafındaki sunucu durumu denetleyicisinin ağ olaylarını işlemesini ve gelen cevapları işlemesini tetikler. Bu fonksiyon genellikle oyun döngüsünde periyodik olarak çağrılır.

*   **`ServerStateCheckerRequest()`**:
    *   `CServerStateChecker::Instance().Request()` fonksiyonunu çağırarak C++ tarafının sunuculara durum sorgusu göndermesini başlatır.

*   **`ServerStateCheckerAddChannel(serverIndex, addr, port)`**:
    *   Argüman olarak sunucu indeksi (`serverIndex`), IP adresi (`addr`) ve port numarasını (`port`) alır.
    *   `CServerStateChecker::Instance().AddChannel()` fonksiyonunu çağırarak bu kanal bilgilerini C++ tarafındaki kontrol listesine ekler.

*   **`ServerStateCheckerInitialize()`**:
    *   `CServerStateChecker::Instance().Initialize()` fonksiyonunu çağırarak C++ tarafındaki durumu sıfırlar (kanal listesini temizler, bağlantıyı keser).

**Modül Başlatma (`initServerStateChecker()`)**

*   `s_methods` adlı bir `PyMethodDef` dizisi tanımlar. Bu dizi, Python'da kullanılacak fonksiyon adlarını (`"Create"`, `"Update"` vb.) ve bunlara karşılık gelen C++ sarmalayıcı fonksiyonlarını (`ServerStateCheckerCreate`, `ServerStateCheckerUpdate` vb.) eşleştirir.
*   `Py_InitModule("ServerStateChecker", s_methods)` fonksiyonunu çağırarak `ServerStateChecker` adında bir Python modülü oluşturur ve tanımlanan metodları bu modüle kaydeder.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, Python ile yazılmış sunucu seçme arayüzünün (`uiSelectServer.py` veya benzeri), oyun sunucularının kanallarının (CH1, CH2 vb.) durumlarını (online, offline, yoğunluk) öğrenmesini ve oyuncuya göstermesini sağlar.
1.  Arayüz, `ServerStateChecker.Create(self)` gibi bir çağrı ile kendi pencere nesnesini C++'a bildirir.
2.  Listelenen her sunucu kanalı için `ServerStateChecker.AddChannel(index, ip, port)` çağrılarak C++ tarafına bilgi verilir.
3.  Periyodik olarak (veya bir "Yenile" butonuna basıldığında) `ServerStateChecker.Request()` çağrılarak durum sorgulaması başlatılır.
4.  Oyunun ana güncelleme döngüsünde `ServerStateChecker.Update()` çağrılarak C++ tarafının sunucudan gelen cevapları işlemesi sağlanır.
5.  C++ tarafı cevapları aldığında, `Create` ile kaydedilmiş Python pencere nesnesinin `NotifyChannelState(serverIndex, status)` metodunu çağırır.
6.  Python arayüzü bu metot içinde gelen `serverIndex` ve `status` bilgilerine göre ekrandaki sunucu listesini günceller.
Bu sayede oyuncular, sunucuya bağlanmadan önce hangi kanalların aktif olduğunu ve yoğunluk durumlarını görebilirler.

---

### `StdAfx.h` (Standart Uygulama Çerçevesi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`StdAfx.h`, Visual C++ projelerinde sıkça kullanılan "Precompiled Headers" (Önceden Derlenmiş Başlıklar) mekanizmasının bir parçasıdır. Temel amacı, projenin `UserInterface` modülü için sıkça kullanılan ve nadiren değişen başlık dosyalarını tek bir yerde toplayarak derleme sürelerini kısaltmaktır. Bu dosya, `UserInterface` modülünün ihtiyaç duyduğu diğer kütüphanelerin (`EterLib`, `EterPythonLib`, `GameLib` vb.) `StdAfx.h` dosyalarını, Windows API başlıklarını, DirectX başlıklarını, projeye özgü temel tanımlamaları, sabitleri ve Python modüllerinin başlatma fonksiyonlarının bildirimlerini içerir.

**Ana İçerik Türleri**

*   **Pragma Direktifleri**:
    *   `#pragma once`: Bu başlık dosyasının derleme sırasında yalnızca bir kez dahil edilmesini sağlar.
    *   `#pragma warning(disable:...)`: Belirli derleyici uyarılarını (örneğin, 4702 - ulaşılamayan kod) devre dışı bırakır. Bu, bazen kaçınılmaz olan veya bilinçli olarak göz ardı edilen uyarıların derleme çıktısını kirletmesini engellemek için kullanılır.
    *   `#define _USE_32BIT_TIME_T`: Visual Studio 2005 (MSC_VER >= 1400) ve sonrası için `time_t` türünün 32-bit olarak kullanılmasını zorlar. Bu, eski sistemlerle uyumluluk veya belirli bir veri yapısı beklentisi için olabilir.

*   **Dahil Edilen Diğer `StdAfx.h` Dosyaları**:
    *   `../EterLib/StdAfx.h`
    *   `../EterPythonLib/StdAfx.h`
    *   `../GameLib/StdAfx.h`
    *   `../ScriptLib/StdAfx.h`
    *   `../MilesLib/StdAfx.h` (Ses kütüphanesi)
    *   `../EffectLib/StdAfx.h`
    *   `../PRTerrainLib/StdAfx.h` (Arazi render kütüphanesi)
    *   `../SpeedTreeLib/StdAfx.h` (Ağaç ve bitki örtüsü render kütüphanesi)
    Bu sayede, `UserInterface` modülü bu kütüphanelerin temel tanımlarına ve yapılarına erişebilir.

*   **DirectX ve Windows Başlıkları**:
    *   `#ifndef __D3DRM_H__ #define __D3DRM_H__ #endif`: `d3drm.h` başlığının (Direct3D Retained Mode, eski bir DirectX API'si) dahil edilmesini engellemek veya çift dahil edilmesini önlemek için bir kontrol. Genellikle bu, projenin bu başlığa ihtiyacı olmadığını veya kendi minimal tanımını kullandığını gösterir.
    *   `<dshow.h>`: DirectShow (video ve ses oynatma için).
    *   `<qedit.h>`: DirectShow Editing Services.

*   **Projeye Özgü Başlıklar ve Tanımlamalar**:
    *   `Locale.h`: Yerelleştirme ile ilgili tanımları içerir.
    *   `GameType.h`: Oyunla ilgili temel tür tanımlarını içerir.
    *   `extern DWORD __DEFAULT_CODE_PAGE__;`: Varsayılan karakter kod sayfasını tutan global bir değişkenin bildirimi.
    *   `#define APP_NAME "Metin 2"`: Uygulama adını tanımlar.

*   **Global Sabitler (Enum)**:
    *   `POINT_MAX_NUM`: Maksimum "point" (stat, özellik) sayısını tanımlar. `ENABLE_ATTR_6TH_7TH` makrosuna göre değeri değişir.
    *   `CHARACTER_NAME_MAX_LEN`: Maksimum karakter adı uzunluğu.
    *   `PLAYER_NAME_MAX_LEN`: Oyuncu adı için maksimum uzunluk (genellikle daha kısa bir gösterim).
    *   `CHARACTER_FOLDER_MAX_LEN`: Karakter yapılandırma dosyalarının saklandığı klasör adı için maksimum uzunluk.
    *   `GROWTH_EVO_NAME_LENGTH`: `ENABLE_GROWTH_PET_SYSTEM` aktifse, evcil hayvan evrim isimlerinin maksimum uzunluğu.

*   **Python Modülü Başlatma Fonksiyonlarının Bildirimleri (`void init...()`)**:
    *   `UserInterface` modülünün C++ tarafında tanımlanan ve Python'a açılan çeşitli alt sistemlerinin (`initapp`, `initsystem`, `initchr`, `initnet`, `initskill` vb.) başlatma fonksiyonlarının bildirimlerini içerir. Bu fonksiyonlar, istemci başladığında `CPythonLauncher` tarafından çağrılarak ilgili Python modüllerini Python yorumlayıcısına kaydeder.
    *   Birçok `init` fonksiyonu `#if defined(...)` direktifleriyle belirli sistemlerin (örneğin, `ENABLE_MAILBOX`, `ENABLE_ACCE_COSTUME_SYSTEM`, `RENDER_TARGET`) derleme zamanında aktif olup olmamasına göre dahil edilir veya edilmez.
    *   `ENABLE_CYTHON` makrosu tanımlıysa, `initsystemSetting` ve `initrootlibManager` gibi bazı modüllerin farklı başlatma fonksiyonları çağrılabilir.

*   **Uygulama String Tablosu Fonksiyon Bildirimleri**:
    *   `ApplicationStringTable_GetString(DWORD dwID)`: Verilen ID'ye karşılık gelen string'i (genellikle `resource.h` içinde tanımlı) döndürür.
    *   `ApplicationStringTable_GetString(DWORD dwID, LPCSTR szKey)`: Aynı işlevi yapar ancak bir anahtar (key) ile daha spesifik bir string alabilir (kullanımı nadirdir).
    *   `ApplicationStringTable_GetStringz(...)`: `const char*` olarak string döndüren versiyonları.
    *   `ApplicationSetErrorString(const char* szErrorString)`: Uygulama genelinde bir hata mesajı ayarlamak için kullanılır.

**Dosyanın Oyundaki Kullanım Amacı**

`StdAfx.h`, `UserInterface.dll` (veya projenin derlenmiş hali) için merkezi bir başlık dosyası görevi görür.
1.  **Derleme Süresini Azaltma**: Sık kullanılan ve nadiren değişen başlıkları önceden derleyerek genel derleme süresini önemli ölçüde azaltır.
2.  **Bağımlılık Yönetimi**: `UserInterface` modülünün ihtiyaç duyduğu tüm temel harici ve dahili kütüphanelerin başlıklarını, genel sabitleri ve Python modül başlatıcılarını tek bir yerden yönetir.
3.  **Yapılandırma ve Koşullu Derleme**: `#if defined(...)` makroları aracılığıyla farklı oyun sistemlerinin veya özelliklerinin derlemeye dahil edilip edilmeyeceğini kontrol eder, bu da projenin farklı versiyonlarının (örneğin, farklı yerelleştirmeler veya özellik setleri ile) yönetilmesini kolaylaştırır.

Bu dosyadaki değişiklikler (özellikle yeni başlıkların eklenmesi), genellikle tüm projenin yeniden derlenmesini gerektirir çünkü önceden derlenmiş başlık (`.pch` dosyası) geçersiz hale gelir.

---

### `StdAfx.cpp` (Standart Uygulama Çerçevesi Kaynak Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`StdAfx.cpp`, Visual C++ projelerinde "Precompiled Headers" (Önceden Derlenmiş Başlıklar) mekanizmasını desteklemek için kullanılan çok basit bir kaynak dosyasıdır. Temel ve tek amacı, `StdAfx.h` başlık dosyasını dahil etmektir.

**Ana İçerik**

*   `#include "stdafx.h"`: Bu satır, `StdAfx.h` dosyasını ve dolayısıyla `StdAfx.h` içinde referans verilen tüm diğer başlık dosyalarını içerir.

**Derleme Sürecindeki Rolü**

1.  Derleyici, `StdAfx.cpp` dosyasını işlerken, `#include "stdafx.h"` direktifi aracılığıyla `StdAfx.h` içindeki tüm başlıkları okur.
2.  Eğer proje "Precompiled Headers Kullan" (/Yu) seçeneği ile yapılandırılmışsa, derleyici bu başlıkların derlenmiş bir halini bir `.pch` (Precompiled Header) dosyasına yazar. Bu işlem genellikle projenin ilk derlemesinde veya `StdAfx.h` (ya da onun içerdiği bir başlık) değiştiğinde yapılır.
3.  Projedeki diğer `.cpp` dosyaları derlenirken, eğer onlar da "Precompiled Headers Kullan" (/Yc veya /YX) seçeneğiyle yapılandırılmışsa ve ilk satırlarında `#include "stdafx.h"` içeriyorlarsa, derleyici bu başlıkları tekrar ayrıştırmak yerine önceden derlenmiş `.pch` dosyasındaki bilgileri kullanır. Bu, özellikle büyük projelerde derleme sürelerini önemli ölçüde kısaltır.
4.  `StdAfx.cpp` dosyasından derlenen `stdafx.obj` dosyası, genellikle `.pch` dosyasını oluşturmak için gereken bilgileri ve sembolleri içerir.

**Dosyanın Oyundaki Kullanım Amacı**

`StdAfx.cpp` dosyasının doğrudan oyunun çalışma zamanı davranışlarına bir etkisi yoktur. Tamamen derleme sürecini optimize etmek ve geliştirmeyi hızlandırmak için var olan bir araçtır. `UserInterface` modülünün diğer tüm `.cpp` dosyaları, `#include "stdafx.h"` satırını en başa ekleyerek önceden derlenmiş başlıkların avantajlarından faydalanır.