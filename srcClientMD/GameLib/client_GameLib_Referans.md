# GameLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `GameLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır.

## İçindekiler

<!-- TOC -->
<!-- /TOC -->

## Dosya Bazlı Detaylandırma 

### `ActorInstance.h` ve `ActorInstance.cpp` (`CActorInstance` Sınıfı)

**Amaç:**

`CActorInstance` sınıfı, Metin2 oyun dünyasındaki tüm dinamik varlıkların (oyuncular, NPC'ler, canavarlar, binekler, bazı etkileşimli objeler vb.) temel temsilidir. `CGraphicThingInstance` sınıfından türeyerek grafiksel gösterim yeteneklerini miras alır ve bunun üzerine oyun mantığına özgü işlevler ekler. Bu sınıf, bir aktörün görünümünü, animasyonlarını, hareketlerini, durumlarını, savaş yeteneklerini, çarpışmalarını ve diğer oyun içi etkileşimlerini yönetir.

**Kalıtım:**

*   `public IActorInstance` (Muhtemelen `CGraphicThingInstance` veya benzeri bir temel grafik sınıfından gelen bir arayüz)
*   `public IFlyTargetableObject` (Uçan nesneler/yetenekler için hedeflenebilir olma arayüzü)

**Temel Bileşenler ve İşlevler:**

1.  **Grafiksel Temsil ve Özelleştirme:**
    *   **Model ve Parçalar:** Irk (`CRaceData`), saç, şekil gibi temel görünümleri yönetir. Silah, zırh gibi farklı vücut parçalarına (`CRaceData::PART_MAIN`, `CRaceData::PART_WEAPON` vb.) eşyalar ekleyebilir (`AttachWeapon`, `ChangePart`).
    *   **SpeedTree Entegrasyonu:** `CSpeedTreeWrapper` kullanarak bir aktörü SpeedTree nesnesi olarak temsil edebilir (`__CreateTree`, `__DestroyTree`). Bu genellikle sabit, büyük objeler için kullanılır.
    *   **Render Modları:** `ERenderMode` (NORMAL, BLEND, ADD, MODULATE) enum'u ile farklı render efektleri uygulayabilir. Alpha değeri (`SetAlphaValue`, `BlendAlphaValue`) ve malzeme rengi (`SetMaterialColor`) ayarlanabilir.
    *   **Efektler:** Kemiklere bağlı çeşitli görsel efektler (`TAttachingEffect`) ekleyebilir ve yönetebilir (`AttachEffectByName`, `AttachEffectByID`, `DettachEffect`). Efektlerin ömrü (`EFFECT_LIFE_NORMAL`, `EFFECT_LIFE_INFINITE`, `EFFECT_LIFE_WITH_MOTION`) olabilir.
    *   **Silah İzleri (Weapon Traces):** Saldırı animasyonları sırasında silahların bıraktığı görsel izleri (`CWeaponTrace`) yönetir.

2.  **Durum Yönetimi (State Machine):**
    *   Aktörün içinde bulunabileceği çok sayıda durumu (flag) yönetir: `m_isWalking`, `m_isMain` (ana oyuncu karakteri mi), `m_isSleep`, `m_isParalysis` (felç), `m_isFaint` (baygın), `m_isRealDead` (gerçekten ölü), `m_isStun` (sersemlemiş), `m_isHiding` (gizlenmiş), `m_isResistFallen` (düşmeye dirençli) vb.
    *   Bu durumlar, aktörün hareketlerini, animasyonlarını ve diğer yeteneklerini kısıtlayabilir veya değiştirebilir.
    *   `SetParalysis`, `SetFaint`, `SetSleep`, `Die`, `Revive` gibi metotlarla bu durumlar değiştirilir.

3.  **Hareket ve Konumlandırma:**
    *   **Konum:** Dünya koordinatlarında (`m_x`, `m_y`, `m_z`) ve piksel pozisyonlarında (`m_kPPosCur`, `m_kPPosSrc`, `m_kPPosDst`) konumunu saklar.
    *   **Hareket Vektörü:** `m_v3Movement` ile anlık hareketi biriktirir.
    *   **Dönüş (Rotation):** `m_fcurRotation` (mevcut), `m_fAdvancingRotation` (ilerleme yönü), `m_rotEnd` (hedef dönüş) gibi değişkenlerle dönüşü yönetir. `LookAt` metotlarıyla belirli bir yöne veya hedefe bakabilir.
    *   **Hız:** Hareket hızı (`m_fMovSpd`) ve saldırı hızı (`m_fAtkSpd`) ayarlanabilir.
    *   **Binek (Mount):** Bir ata (`m_pkHorse`) binebilir. Binek üzerindeyken konum ve bazı hareketler binek tarafından kontrol edilir.
    *   **Fizik:** `CPhysicsObject` üyesi ile basit fizik güncellemeleri (yerçekimi, itme vb.) alır.
    *   **Hareket Komutları:** `Move()` (koşma/yürüme), `Stop()`, `SetBlendingPosition` (belirli bir konuma yumuşak geçiş).

4.  **Animasyon Sistemi (Motion System):**
    *   **Hareket Modları:** `CRaceMotionData::MODE_GENERAL`, `MODE_TWOHAND_SWORD`, `MODE_BOW`, `MODE_HORSE` gibi farklı duruş ve silah modlarına göre animasyonları belirler.
    *   **Hareket Kuyruğu (`TMotionDeque`):** `PushOnceMotion`, `PushLoopMotion`, `InterceptOnceMotion` gibi metotlarla animasyonları bir kuyruğa alarak sıralı oynatır.
    *   **Hareket Bilgileri:** `CRaceMotionData` ve `CRaceData` üzerinden mevcut ırka ve hareket moduna uygun animasyon verilerini alır.
    *   **Animasyon Parametreleri:** Animasyonların başlama zamanı, karışım süresi (blend time), süresi, hızı (`fSpeedRatio`) ve tekrar sayısı (`iLoopCount`) ayarlanabilir.
    *   **Hareket Olayları (`TMotionEventInstance`):** Animasyonların belirli zamanlarında tetiklenen olayları (ses, efekt, saldırı, warp vb.) işler (`MotionEventProcess`).

5.  **Savaş ve Yetenek Sistemi:**
    *   **Saldırı Komutları:** `InputNormalAttackCommand`, `InputComboAttackCommand`, `NormalAttack`, `ComboAttack`.
    *   **Kombo Sistemi:** `m_wcurComboType`, `m_dwcurComboIndex` ile kombo saldırılarını takip eder. `__RunNextCombo` ile bir sonraki komboya geçer.
    *   **Yetenek Kullanımı:** `OnUseSkill` (olay işleyici üzerinden) ve hareket sistemiyle entegre yetenek animasyonları.
    *   **Hasar Verme ve Alma:** `AttackingProcess`, `__ProcessDataAttackSuccess`, `__OnHit`.
    *   **Saldırı Menzili:** `m_fReachScale` ile saldırı menzili ayarlanır.
    *   **Hedefleme:** `CFlyTarget` ile uçan yetenekler veya menzilli saldırılar için hedef belirleyebilir (`AddFlyTarget`, `SetFlyTarget`).
    *   **Savunma Durumları:** `IsParalysis`, `IsFaint`, `IsStun` gibi durumlar savaş yeteneğini etkiler.

6.  **Çarpışma Tespiti (Collision Detection):**
    *   **Çarpışma Noktaları (`TCollisionPointInstance`):** Aktörün vücuduna (body) ve savunma alanlarına (defending) bağlı çarpışma küreleri (`CDynamicSphereInstanceVector`) tanımlanır.
    *   **Güncelleme:** `UpdatePointInstance` ile bu kürelerin pozisyonları güncellenir.
    *   **Testler:** `TestActorCollision` (diğer aktörlerle), `__TestObjectCollision` (statik nesnelerle), `TestCollisionWithDynamicSphere`.
    *   **Hareket Ayarlaması:** Çarpışma durumunda hareketi düzenler (`AdjustDynamicCollisionMovement`, `__AdjustCollisionMovement`, `BlockMovement`).
    *   **Sıçrama Alanı (Splash Area):** Alan etkili saldırılar için `SSplashArea` yapısını kullanır ve bu alandaki hedefleri `HittedInstanceMap` ile takip eder.

7.  **Olay Yönetimi (`IEventHandler`):**
    *   Aktörde meydana gelen önemli olayları (pozisyon senkronizasyonu, hareket başlangıcı/bitişi, saldırı, efekt değişimi vb.) dışarıya bildirmek için bir olay işleyici (`m_pkEventHandler`) kullanır.
    *   `__OnSyncing`, `__OnMoving`, `__OnAttack` gibi iç metotlar aracılığıyla bu olaylar tetiklenir.

8.  **Aktör Tipleri (`EType`):**
    *   `TYPE_PC` (Oyuncu Karakteri), `TYPE_NPC`, `TYPE_ENEMY`, `TYPE_STONE` (Metin Taşı), `TYPE_WARP`, `TYPE_DOOR`, `TYPE_BUILDING`, `TYPE_HORSE`, `TYPE_PET` gibi farklı aktör türlerini destekler.
    *   Bu türler, aktörün davranışlarını, etkileşimlerini ve bazı özelliklerini belirler.

9.  **Yardımcı İşlevler:**
    *   **Kimlik:** `m_dwSelfVID` (kendi sanal ID'si), `m_dwOwnerVID` (sahibinin ID'si).
    *   **Durum Kontrolleri:** `IsPC()`, `IsNPC()`, `IsMoving()`, `IsDead()`, `isAttacking()` gibi çok sayıda `Is...()` metodu ile aktörün mevcut durumu sorgulanabilir.
    *   **Başlatma ve Yok Etme:** `__Initialize()` ile üye değişkenleri sıfırlar, `Destroy()` ile kaynakları serbest bırakır.

**Önemli Üye Değişkenler (Özet):**

*   **Grafiksel:** `m_pkCurRaceData`, `m_pkCurRaceMotionData` (animasyon ve model verileri), `m_AttachingEffectList`, `m_WeaponTraceVector`, `m_pkTree`.
*   **Konum/Hareket:** `m_x, m_y, m_z`, `m_kPPosCur`, `m_fcurRotation`, `m_fMovSpd`, `m_fAtkSpd`, `m_pkHorse`, `m_PhysicsObject`.
*   **Animasyon:** `m_MotionDeque`, `m_kCurMotNode`, `m_wcurMotionMode`.
*   **Durum/Savaş:** `m_eActorType`, `m_eRace`, `m_isRealDead`, `m_isStun`, `m_isParalysis`, `m_fInvisibleTime`, `m_dwcurComboIndex`, `m_kSplashArea`, `m_HitDataMap`.
*   **Çarpışma:** `m_BodyPointInstanceList`, `m_DefendingPointInstanceList`.
*   **Olay/Hedef:** `m_pkEventHandler`, `m_kFlyTarget`, `m_pFlyEventHandler`.
*   **Render:** `m_iRenderMode`, `m_fAlphaValue`, `m_AddColor`.

**`ActorInstance.cpp` Dosyasındaki Ana İşlem Grupları:**

*   **`INSTANCEBASE_Deform()` ve `INSTANCEBASE_Transform()`:** Temel güncelleme döngüsünün parçaları. `Deform` genellikle iskelet/vertex deformasyonlarını, `Transform` ise pozisyon/rotasyon güncellemelerini ve buna bağlı diğer işlemleri (çarpışma, boundingsphere vb.) yapar.
*   **`OnUpdate()`:** `CGraphicThingInstance::OnUpdate()` çağrısını yapar, ekli instanceleri ve alfa blendini günceller.
*   **Durum Ayarlama Metotları:** `SetParalysis`, `SetFaint`, `SetSleep`, `SetAttackSpeed`, `SetMoveSpeed` vb.
*   **Durum Sorgulama Metotları:** `IsPC`, `IsNPC`, `IsMoving`, `IsDead`, `IsAttacked` vb.
*   **Hareket ve Animasyon Yönetimi:** `Move`, `Stop`, `SetLoopMotion`, `InterceptOnceMotion`, `MotionProcess`, `ReservingMotionProcess`, `CurrentMotionProcess`.
*   **Savaş Mantığı:** `AttackingProcess`, `NormalAttack`, `ComboAttack`, `Die`, `Revive`, `__ProcessDataAttackSuccess`.
*   **Çarpışma Yönetimi:** `UpdatePointInstance`, `TestActorCollision`, `AdjustDynamicCollisionMovement`.
*   **Pozisyon ve Rotasyon Yönetimi:** `SetCurPixelPosition`, `SetRotation`, `BlendRotation`, `AccumulationMovement`, `TransformProcess`.
*   **Efekt ve Ek Parça Yönetimi:** `AttachWeapon`, `AttachEffectByID`, `UpdateAttachingInstances`.
*   **Başlatma ve Temizleme:** `__Initialize`, `Destroy`.
*   **Olay İşleme:** `MotionEventProcess`, `__OnSyncing`, `__OnAttack` vb. (genellikle `m_pkEventHandler` üzerinden çağrılır).

`CActorInstance`, istemci tarafındaki oyun mantığının büyük bir kısmını barındıran, karmaşık ve merkezi bir sınıftır. Diğer birçok sistem (UI, ağ, efektler) bu sınıfla doğrudan veya dolaylı olarak etkileşimdedir.

#### `ActorInstanceAttach.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının, karakterlerin ve diğer aktörlerin üzerine çeşitli görsel öğelerin (silahlar, zırhlar, aksesuarlar, kostümler) ve görsel efektlerin nasıl eklendiğini, yönetildiğini ve güncellendiğini ele alır. Oyuncunun karakterinin görünümünü kişiselleştirmesi, eşya giyip çıkarması ve bu eşyaların veya durumların yarattığı görsel efektlerin gösterilmesi doğrudan bu modülün sorumluluğundadır.

**Temel İşlevler ve Önemli Metotlar:**

*   **Silah Yönetimi:**
    *   `AttachWeapon(DWORD dwItemIndex, ...)` ve `AttachWeapon(DWORD dwParentPartIndex, DWORD dwPartIndex, CItemData* pItemData)`: Bir aktöre, verilen eşya ID'si (`dwItemIndex`) veya `CItemData` pointer'ı aracılığıyla silah takar. Silahın hangi ele (`CRaceData::PART_WEAPON` veya `CRaceData::PART_WEAPON_LEFT`) takılacağını belirler.
    *   `__IsLeftHandWeapon(DWORD type)`, `__IsRightHandWeapon(DWORD type)`: Verilen silah türünün (örneğin, `CItemData::WEAPON_DAGGER`) sol ele mi, sağ ele mi ait olduğunu kontrol eder.
    *   `__IsWeaponTrace(DWORD weaponType)`: Belirli bir silah türünün (yay, yelpaze, çan hariç) saldırı sırasında görsel bir iz (`CWeaponTrace`) bırakıp bırakmayacağını belirler.
    *   `__DestroyWeaponTrace()`: Mevcut tüm silah izlerini temizler (genellikle yeni bir silah takıldığında çağrılır).
    *   `SetWeaponTraceTexture(const char* szTextureName)`, `UseTextureWeaponTrace()`, `UseAlphaWeaponTrace()`: Silah izlerinin dokusunu ve render modunu (doku tabanlı veya alfa tabanlı) ayarlar.
*   **Efekt Yönetimi:**
    *   `AttachEffectByID(DWORD dwParentPartIndex, const char* c_pszBoneName, DWORD dwEffectID, ...)`: Belirli bir efekt ID'sini kullanarak, aktörün bir parçasına (`dwParentPartIndex`) ve isteğe bağlı olarak o parçanın bir kemiğine (`c_pszBoneName`) görsel efekt ekler. Efektin pozisyonu ve ölçeği ayarlanabilir.
    *   `AttachEffectByName(DWORD dwParentPartIndex, const char* c_pszBoneName, const char* c_pszEffectName)`: Efekt dosya adını kullanarak efekt ekler.
    *   `AttachSmokeEffect(DWORD eSmoke)`: Irk verilerinde tanımlanmış özel duman efektlerini (örneğin, seviye atlama efekti) ekler.
    *   `DettachEffect(DWORD dwEID)`: Belirli bir efekt ID'sine sahip efekti aktörden kaldırır.
    *   `UpdateAttachingInstances()`: Aktöre bağlı tüm efektlerin konumlarını (eğer bir kemiğe bağlıysa) ve yaşam sürelerini günceller. Ömrü dolan veya bağlı olduğu kaynak yok olan efektleri temizler.
    *   `ShowAllAttachingEffect()`, `HideAllAttachingEffect()`, `__ClearAttachingEffect()`: Tüm bağlı efektleri gösterir, gizler veya temizler.
    *   `SetActiveAllAttachingEffect()`, `SetDeactiveAllAttachingEffect()`: Efektlerin güncellenip güncellenmeyeceğini toplu olarak ayarlar (grafik optimizasyonları için olabilir).
*   **Parça ve Model Yönetimi:**
    *   `RefreshActorInstance()`: Aktörün tüm görünümünü (ana model, takılı parçalar, bunlara bağlı çarpışma verileri ve efektler) yeniden yükler ve ayarlar. Genellikle karakterin ırkı değiştiğinde veya önemli bir görünüm güncellemesi gerektiğinde çağrılır.
    *   `ChangePart(DWORD dwPartIndex, DWORD dwItemIndex)`: Aktörün belirli bir parçasında (`dwPartIndex`) takılı olan eşyanın ID'sini (`dwItemIndex`) günceller. Bu, sadece ID'yi saklar, asıl model değişimi `RefreshActorInstance` veya ilgili `Attach` metotlarıyla olur.
    *   `GetAttachingBoneName(DWORD dwPartIndex, const char** c_pszBoneName)`: Belirli bir parça türü için kemik adını alır (örn: silahın takılacağı kemik).
*   **Kostüm Sistemleri Desteği (Koşullu Derleme):**
    *   `ENABLE_ACCE_COSTUME_SYSTEM`, `ENABLE_WEAPON_COSTUME_SYSTEM`, `ENABLE_QUIVER_SYSTEM` gibi C++ makroları ile kontrol edilen bölümlerde, aksesuar (`AttachAcce`), silah kostümü ve ok çantası gibi ek kozmetik veya işlevsel eşyaların takılmasını ve yönetilmesini sağlar.
*   **Diğer:**
    *   `USE_WEAPON_SPECULAR`: Silahların yansıma (specular) özelliğini kullanıp kullanmayacağını belirleyen global bir boolean değişkendir.
    *   `Vietnam_ConvertWeaponVnum(DWORD vnum)`: Vietnam istemcisine özel eski silah ID'lerini yeni sisteme dönüştüren bir fonksiyondur.

#### `ActorInstanceBattle.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının savaşla ilgili tüm mekaniklerini içerir. Karakterlerin ve yaratıkların birbirleriyle nasıl dövüştüğünü, hasar alıp verdiğini, özel durumların (sersemleme, yere düşme vb.) savaşı nasıl etkilediğini ve saldırı/savunma aksiyonlarının hangi koşullarda gerçekleştirilebileceğini tanımlar. Oyunun temel dövüş akışı bu modüldeki fonksiyonlar tarafından yönetilir.

**Temel İşlevler ve Önemli Metotlar:**

*   **Aksiyon İzinleri (Can... metotları):**
    *   `CanAct()`: Aktörün genel olarak bir eylemde bulunup bulunamayacağını (ölü, sersemlemiş, felçli, baygın, uyuyor veya yere düşmüş değilse) kontrol eder.
    *   `CanMove()`: `CanAct()` koşuluna ek olarak, aktörün hareketinin kilitli olup olmadığını kontrol eder.
    *   `CanAttack()`: `CanAct()` koşuluna ek olarak, hareket hızı sıfır değilse ve bir yetenek kullanımı iptal edilemiyorsa saldırı yapılıp yapılamayacağını belirler.
    *   `CanUseSkill()`: `CanAct()` koşuluna ek olarak, mevcut yetenek kullanımı iptal edilebilir değilse veya özel bazı animasyonlarda (örn: balık tutma) değilse yetenek kullanılıp kullanılamayacağını kontrol eder.
    *   `CanFishing()`: Balık tutma eyleminin yapılıp yapılamayacağını kontrol eder.
*   **Saldırı Komutları ve Zamanlaması:**
    *   `InputNormalAttackCommand(float fDirRot)`: Oyuncudan gelen normal saldırı komutunu alır ve eğer mümkünse (`__CanInputNormalAttackCommand`) saldırıyı başlatır.
    *   `InputComboAttackCommand(float fDirRot)`: Kombo saldırı komutunu alır. Eğer kombo sırasında doğru zamanda (`m_pkCurRaceMotionData` içindeki zamanlamalara göre) girdi yapılmışsa bir sonraki komboyu (`__RunNextCombo`) tetikler veya girdiyi önceden alır (`m_isPreInput`).
    *   `NormalAttack(float fDirRot, float fBlendTime)`: Verilen yöne doğru normal saldırı animasyonunu başlatır.
    *   `ComboAttack(DWORD dwMotionIndex, float fDirRot, float fBlendTime)`: Belirli bir kombo hareketini verilen yöne doğru başlatır.
    *   `ComboProcess()`: Her güncellemede çağrılarak devam eden komboları, zamanlamalarını ve önceden alınmış girdileri işler.
    *   `__RunNextCombo()`, `__OnEndCombo()`, `__ClearCombo()`: Kombo saldırı zincirinin ilerlemesini, bitişini ve sıfırlanmasını yönetir.
*   **Saldırı Durumları ve Kontrolleri:**
    *   `isAttacking()`: Aktörün o anda herhangi bir saldırı (normal, kombo, splash) yapıp yapmadığını döndürür.
    *   `isValidAttacking()`: Mevcut saldırı animasyonunun, hasar verme zaman aralığında olup olmadığını kontrol eder.
    *   `isNormalAttacking()`, `isComboAttacking()`, `IsSplashAttacking()`: Aktörün belirli bir saldırı türünü yapıp yapmadığını kontrol eder.
    *   `GetAttackingElapsedTime()`: Mevcut saldırı animasyonunun ne kadar süredir devam ettiğini döndürür.
*   **Hasar İşleme ve Efektler:**
    *   `__ProcessDataAttackSuccess(const NRaceData::TAttackData& c_rAttackData, CActorInstance& rVictim, ...)`: Bir saldırı başarılı olduğunda çağrılır. Kurbana `NRaceData::TAttackData` içinde tanımlanan etkileri (sersemletme süresi, geri itme kuvveti, görünmezlik süresi) uygular. Vuruş efektlerini (`m_dwBattleHitEffectID`) oluşturur ve kurbana özel animasyonları (`__HitStone`, `__HitGood`, `__HitGreate`) tetikler.
    *   `__ProcessMotionEventAttackSuccess(...)`, `__ProcessMotionAttackSuccess(...)`: Animasyon olaylarından veya doğrudan animasyon verilerinden gelen saldırı başarı bilgilerini işleyerek `__ProcessDataAttackSuccess`'ı çağırır.
    *   `__HitStone(CActorInstance& rVictim)`: Kurban bir metin taşı veya kapı ise uygulanacak özel vuruş tepkilerini (genellikle sadece titreme) yönetir.
    *   `__HitGood(CActorInstance& rVictim)`: Kurban hafif bir darbe aldığında (veya düşmeye dirençliyse) uygulanacak hasar animasyonunu (öne veya arkaya doğru hafif sendeleme) oynatır.
    *   `__HitGreate(CActorInstance& rVictim)`: Kurban ağır bir darbe aldığında yere düşme (`CRaceMotionData::NAME_DAMAGE_FLYING`) ve ardından kalkma (`CRaceMotionData::NAME_STAND_UP`) animasyonlarını tetikler.
    *   `OnShootDamage()`: Uzaktan bir saldırı ile hasar alındığında çağrılır, genellikle sersemlemişse ölür, değilse hasar animasyonu oynatır.
*   **Geri İtme ve Fiziksel Tepkiler:**
    *   `__CanPushDestActor(CActorInstance& rkActorDst)`: Bir hedefin geri itilip itilemeyeceğini kontrol eder (bina, kapı, metin taşı, NPC veya çok büyük yaratıklar genellikle itilemez).
    *   `__PushCircle(CActorInstance& rVictim)`: Kurbanı, saldırganın konumundan çembersel bir şekilde dışa doğru iter.
    *   `__PushDirect(CActorInstance& rVictim)`: Kurbanı, saldırganın baktığı yöne doğru iter.
    *   `__SetFallingDirection(float fx, float fy)`: Kurbanın fizik nesnesine (`m_PhysicsObject`) düşme yönünü ayarlar.
    *   `__Shake(DWORD dwDuration)`: Aktörün belirtilen süre boyunca titremesini sağlar. `ShakeProcess()` ile her frame güncellenir.
*   **Durum ve Yetenek Kısıtlamaları:**
    *   `SetBattleHitEffect(DWORD dwID)`, `SetBattleAttachEffect(DWORD dwID)`: Savaş sırasında kullanılacak varsayılan vuruş ve kurbana takılacak efektlerin ID'lerini ayarlar.
    *   `IsActTargetEmotion()`, `IsActEmotion()`: Karakterin hedefe yönelik bir duygu ifadesi mi yoksa genel bir duygu ifadesi mi yaptığını kontrol eder (bu sırada genellikle saldırı yapılamaz).
*   **Diğer Savaşla İlgili İşlevler:**
    *   `SetBlendingPosition(...)`, `ResetBlendingPosition()`, `GetBlendingPosition(...)`: Karakterin pozisyonunu yumuşak bir geçişle (blending) değiştirmek için fizik nesnesini kullanır.
    *   `__isInvisible()`: Karakterin görünmezlik süresinin devam edip etmediğini kontrol eder.

Bu iki dosya, `CActorInstance`'ın oyun dünyasında nasıl göründüğünü, nasıl donatıldığını ve diğer aktörlerle nasıl savaştığını belirleyen temel mantığı içerir. Karakterin ekipmanını değiştirmek, yeni bir efekt eklemek veya bir saldırının sonucunu değiştirmek gibi işlemler bu dosyalardaki fonksiyonlar üzerinden gerçekleştirilir.

#### `ActorInstanceBlend.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının görsel şeffaflık (alfa) efektlerini yönetir. Özellikle bir karakterin veya nesnenin zaman içinde yavaşça görünür hale gelmesi (fade-in) veya yavaşça kaybolması (fade-out) gibi yumuşak geçişli şeffaflık animasyonlarını gerçekleştirmek için kullanılır. Bu, örneğin bir karakterin gizlenme yeteneğinden çıkarken veya öldükten sonra yavaşça kaybolurken görülebilir.

**Temel İşlevler ve Önemli Metotlar:**

*   **Alfa Değeri Ayarlama ve Alma:**
    *   `SetAlphaValue(float fAlpha)`: Aktörün anlık alfa (şeffaflık) değerini ayarlar (0.0: tamamen şeffaf, 1.0: tamamen opak).
    *   `GetAlphaValue()`: Aktörün mevcut alfa değerini döndürür.
*   **Blend Render Modu:**
    *   `SetBlendRenderMode()`: Aktörün render modunu `RENDER_MODE_BLEND` olarak ayarlar. Bu, alfa değerinin render sırasında dikkate alınmasını sağlar.
*   **Alfa Geçişi (Blending) Yönetimi:**
    *   `BlendAlphaValue(float fDstAlpha, float fDuration)`: Aktörün alfa değerini, mevcut değerinden `fDstAlpha` hedef değerine `fDuration` saniye içinde yumuşak bir şekilde değiştirmek için bir geçiş başlatır. Dahili olarak `__BlendAlpha_Apply` çağrılır.
    *   `__BlendAlpha_Initialize()`: Alfa geçişi için kullanılan `m_kBlendAlpha` (tür: `SBlendAlpha`) yapısındaki değişkenleri (başlangıç zamanı, süresi, başlangıç/hedef alfa değeri, eski render modu vb.) sıfırlar.
    *   `__BlendAlpha_Apply(float fDstAlpha, float fDuration)`: Alfa geçişini başlatır. Mevcut alfa değerini, başlangıç zamanını, süresini, hedef alfa değerini ve o anki render modunu `m_kBlendAlpha` yapısında saklar. `m_kBlendAlpha.m_isBlending` bayrağını `true` yapar.
    *   `__BlendAlpha_Update()`: Her frame çağrılarak aktif bir alfa geçişi varsa günceller. Geçen süreye (`__BlendAlpha_GetElapsedTime`) göre mevcut alfa değerini doğrusal olarak hesaplar ve `SetAlphaValue` ile uygular. Geçiş sırasında render modunu `RENDER_MODE_BLEND` yapar.
    *   `__BlendAlpha_UpdateComplete()`: Alfa geçişi tamamlandığında çağrılır. Hedef alfa değeri 1.0'den küçükse render modunu `RENDER_MODE_BLEND` olarak bırakır, aksi halde geçiş öncesindeki render moduna (`m_kBlendAlpha.m_iOldRenderMode`) geri döner. `m_kBlendAlpha.m_isBlending` bayrağını `false` yapar.
    *   `__BlendAlpha_GetElapsedTime()`: Alfa geçişinin başlangıcından itibaren ne kadar süre geçtiğini hesaplar.

**`SBlendAlpha` Yapısı:**

Bu özel yapı, bir alfa geçişinin durumunu ve parametrelerini tutar:
*   `m_isBlending` (bool): Aktif bir alfa geçişi olup olmadığını belirtir.
*   `m_fBaseTime` (float): Geçişin başladığı zaman (genellikle `GetLocalTime()` ile alınır).
*   `m_fDuration` (float): Geçişin toplam süresi.
*   `m_fBaseAlpha` (float): Geçiş başladığındaki alfa değeri.
*   `m_fDstAlpha` (float): Geçişin sonunda ulaşılacak hedef alfa değeri.
*   `m_iOldRenderMode` (DWORD): Geçiş başlamadan önceki render modu.

#### `ActorInstanceCollisionDetection.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının çarpışma tespiti ve tepkileriyle ilgili tüm mantığı içerir. Oyun dünyasındaki aktörlerin (oyuncular, canavarlar vb.) birbirleriyle, statik çevre nesneleriyle (binalar, kayalar vb.) ve saldırıların etki alanlarıyla nasıl fiziksel olarak etkileşime girdiğini yönetir. Bu, hem karakterlerin birbirinin içinden geçmesini engellemek hem de saldırıların hedefe isabet edip etmediğini belirlemek için kritik öneme sahiptir.

**Temel İşlevler ve Önemli Metotlar:**

*   **Çarpışma Kürelerinin (Collision Spheres) Yönetimi:**
    *   `UpdatePointInstance()` ve `UpdatePointInstance(TCollisionPointInstance* pPointInstance)`: Aktörün vücuduna veya ekli parçalarına (örn: silah) tanımlanmış olan çarpışma noktalarının (`TCollisionPointInstance`) dünyadaki pozisyonlarını günceller. Eğer çarpışma noktası bir kemiğe bağlıysa (`isAttached == true`), o kemiğin güncel pozisyonunu kullanarak kürenin pozisyonunu hesaplar.
    *   `UpdateAdvancingPointInstance()`: Özellikle hareket eden aktörlerin bir sonraki frame'deki potansiyel pozisyonlarına göre vücut çarpışma kürelerini günceller. Bu, hareket sırasında oluşabilecek çarpışmaları öngörmek için kullanılır.
    *   `CreateCollisionInstancePiece(DWORD dwAttachingModelIndex, const NRaceData::TAttachingData* c_pAttachingData, TCollisionPointInstance* pPointInstance)`: Bir model parçası (ana model veya ekli eşya) ve ona ait bir `NRaceData::TAttachingData` (çarpışma verisi tanımı) kullanarak yeni bir `TCollisionPointInstance` (çarpışma kümesi) oluşturur. Bu küreler, `NRaceData` içinde tanımlanan yarıçap ve göreceli pozisyonlara göre oluşturulur.
*   **Çarpışma Testleri:**
    *   `CheckCollisionDetection(const CDynamicSphereInstanceVector* c_pAttackingSphereVector, D3DXVECTOR3* pv3Position)`: Verilen bir saldıran küre seti (`c_pAttackingSphereVector`, genellikle bir saldırı animasyonundan gelir) ile bu aktörün savunma küreleri (`m_DefendingPointInstanceList`) arasında çarpışma olup olmadığını kontrol eder. Çarpışma varsa, yaklaşık vuruş pozisyonunu `pv3Position`'a yazar.
    *   `TestActorCollision(CActorInstance& rVictim)`: Bu aktör ile başka bir aktör (`rVictim`) arasında genel bir çarpışma olup olmadığını test eder. Genellikle karakterlerin birbirinin içinden geçmesini engellemek için kullanılır. Saldırı durumunda savunma kürelerini, diğer durumlarda vücut kürelerini kullanır.
    *   `TestPhysicsBlendingCollision(CActorInstance& rVictim)`: Fiziksel pozisyon yumuşatma (blending) sırasında başka bir aktörle çarpışma olup olmadığını test eder.
    *   `__TestObjectCollision(const CGraphicObjectInstance* c_pObjectInstance)`: Aktörün hareket halindeyken statik bir çevre nesnesiyle (`CGraphicObjectInstance`) çarpışıp çarpışmadığını test eder.
    *   `AvoidObject(const CGraphicObjectInstance& c_rkBGObj)`: Statik bir nesneyle çarpışma tespit edilirse, aktörün hareketini bu nesneden kaçınacak şekilde ayarlar (`__AdjustCollisionMovement` çağrılır).
    *   `IsBlockObject(const CGraphicObjectInstance& c_rkBGObj)`: Aktörün statik bir nesne tarafından engellenip engellenmediğini kontrol eder.
    *   `TestCollisionWithDynamicSphere(const CDynamicSphereInstance& dsi)`: Verilen tek bir dinamik küre ile bu aktörün vücut küreleri arasında çarpışma olup olmadığını test eder.
*   **Saldırı İşleme (Çarpışma Bazlı):**
    *   `__SplashAttackProcess(CActorInstance& rVictim)`: Alan etkili bir saldırının (`m_kSplashArea`) kurbanla (`rVictim`) çarpışıp çarpışmadığını kontrol eder. Çarpışma varsa ve kurban daha önce vurulmamışsa ve vuruş limiti aşılmamışsa, `__ProcessDataAttackSuccess` çağrılarak hasar ve etkiler uygulanır.
    *   `__NormalAttackProcess(CActorInstance& rVictim)`: Normal veya kombo saldırı sırasında, saldırganın aktif vuruş küreleri (animasyonun belirli zamanlarında aktifleşen `NRaceData::THitData` ve `mapHitPosition` ile tanımlanır) ile kurbanın savunma küreleri arasında çarpışma olup olmadığını test eder (`DetectCollisionDynamicZCylinderVSDynamicZCylinder`). Çarpışma varsa, `__ProcessDataAttackSuccess` çağrılır.
    *   `AttackingProcess(CActorInstance& rVictim)`: Bir kurbana saldırırken hem alan etkili saldırıları hem de normal/kombo saldırılarını kontrol eder.
*   **Çarpışma Atlatma:**
    *   `EnableSkipCollision()`, `DisableSkipCollision()`, `CanSkipCollision()`: Aktörün geçici olarak tüm çarpışmaları yok saymasını sağlayan bir bayrağı (`m_canSkipCollision`) yönetir. Bu, özel durumlar veya komutlar için kullanılabilir.
*   **Yardımcı Fonksiyonlar:**
    *   `__InitializeCollisionData()`: Çarpışmayla ilgili bayrakları (örn: `m_canSkipCollision`) başlangıç durumuna getirir.
    *   `BlockMovement()`: Aktörün mevcut hareketini durdurur (genellikle bir engelle karşılaşıldığında).
    *   `IsInSafeZone(CActorInstance& ptr)`: Verilen aktörün güvenli bir bölgede (harita attribute'larına göre belirlenir) olup olmadığını kontrol eder. (Bu fonksiyon `UserInterface/PythonBackground.h` bağımlılığına sahiptir.)

**Önemli Yapılar ve Listeler:**

*   `TCollisionPointInstance`: Bir kemiğe bağlı olabilen, birden fazla çarpışma küresi (`CDynamicSphereInstanceVector`) içeren bir çarpışma noktası/bölgesi.
*   `m_BodyPointInstanceList`: Aktörün genel vücut çarpışmaları için kullanılan `TCollisionPointInstance` listesi.
*   `m_DefendingPointInstanceList`: Aktörün saldırılara karşı savunma alanlarını tanımlayan `TCollisionPointInstance` listesi.
*   `m_kSplashArea` (`SSplashArea`): Alan etkili saldırıların aktif kürelerini, vurduğu hedefleri ve diğer parametrelerini tutar.
*   `m_HitDataMap` (`THitDataMap`): Normal/kombo saldırılarda, her bir vuruş dilimi (`NRaceData::THitData`) için hangi hedeflerin vurulduğunu takip eder, böylece aynı saldırı animasyonuyla bir hedefin birden fazla kez hasar alması engellenir. 

#### `ActorInstanceData.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının temel veri yönetimiyle ilgili işlevlerini içerir. Bir aktörün oyun dünyasındaki kimliğini (ID), temel görünümünü (ırk, saç, şekil), üzerine giydiği eşyaların bilgilerini ve özel harita özellikleriyle (attribute) etkileşimini yönetir. Karakterin sunucudan gelen verilerle oluşturulması ve güncellenmesi, görünümünün değiştirilmesi gibi temel işlemler bu modül aracılığıyla yapılır.

**Temel İşlevler ve Önemli Metotlar:**

*   **Kimlik Yönetimi:**
    *   `GetVirtualID()`: Aktörün oyun içindeki benzersiz sanal kimliğini (`m_dwSelfVID`) döndürür.
    *   `SetVirtualID(DWORD dwVID)`: Aktörün sanal kimliğini ayarlar.
*   **Irk (Race) Yönetimi:**
    *   `GetRace()`: Aktörün mevcut ırk ID'sini (`m_eRace`) döndürür.
    *   `SetRace(DWORD eRace)`: Aktörün ırkını ayarlar. Bu işlem sırasında:
        *   `CRaceManager` aracılığıyla ilgili `CRaceData` yüklenir (`m_pkCurRaceData`).
        *   Eğer ırk verisi bir `CAttributeData` içeriyorsa (`pRaceData->GetAttributeDataPtr()`), bu özellik verisiyle bir `CAttributeInstance` oluşturulur (`__CreateAttributeInstance`).
        *   Takılı olan tüm parçaların ID'leri sıfırlanır (`m_adwPartItemID`).
        *   Mevcut tüm ekli efektler temizlenir (`__ClearAttachingEffect`).
        *   `CGraphicThingInstance` temizlenir ve aktör PC ise `CRaceData::PART_MAX_NUM` kadar, değilse 1 model/instance slotu rezerve edilir.
        *   Irka ait tüm animasyonlar (`CRaceData::TMotionVectorMap`) `CGraphicThingInstance`'a kaydedilir (`RegisterMotionThing`).
*   **Görünüm Yönetimi (Saç, Şekil, Materyal):**
    *   `SetHair(DWORD eHair)`: Aktörün saç modelini ve dokularını ayarlar. `CRaceData` içinden ilgili saç tanımını (`CRaceData::SHair`) bulur, modelini (`pkHair->m_stModelFileName`) ve ciltlerini (`pkHair->m_kVct_kSkin`) yükler.
    *   `SetShape(DWORD eShape, float fSpecular)`: Aktörün ana vücut modelini (şeklini) ve ciltlerini ayarlar. `CRaceData` içinden ilgili şekil tanımını (`CRaceData::SShape`) bulur. Eğer şekil tanımında model dosyası belirtilmişse onu, yoksa ırkın temel modelini kullanır. LOD (Level of Detail) modellerini de yükler. Ciltleri (`pkShape->m_kVct_kSkin`) ve isteğe bağlı olarak yansıma (specular) özelliklerini ayarlar. Eğer ırk bir ağaç ise (`pRaceData->IsTree()`), model yerine SpeedTree nesnesi oluşturur (`__CreateTree`). Şekle bağlı temel efektleri de ekler.
    *   `ChangeMaterial(const char* c_szFileName)`: Aktörün mevcut şeklinin ilk cilt parçasının materyalini (dokusunu) verilen dosya adıyla değiştirir. Genellikle lonca sembolleri gibi dinamik doku değişiklikleri için kullanılır.
    *   `SetSpecularInfo(BOOL bEnable, int iPart, float fAlpha)` ve `SetSpecularInfoForce(...)`: Belirli bir parçanın yansıma (specular) efektini açıp kapatır ve gücünü ayarlar.
*   **Özellik (Attribute) Yönetimi:**
    *   `UpdateAttribute()`: Eğer aktör bir `CAttributeInstance`'a (`m_pAttributeInstance`) sahipse ve çarpışma güncellemesi gerekiyorsa (`m_bNeedUpdateCollision`), bu özellik nesnesinin çarpışma verilerini (`UpdateCollisionData`) ve yükseklik haritası verilerini (`UpdateHeightInstance`) günceller.
    *   `__CreateAttributeInstance(CAttributeData* pData)`: Verilen bir `CAttributeData` (genellikle bir harita nesnesinin veya ırkın özellikleri) ile yeni bir `CAttributeInstance` oluşturur ve saklar.
*   **Parça Yönetimi:**
    *   `GetPartItemID(DWORD dwPartIndex)`: Belirli bir vücut parçasında (`dwPartIndex`) takılı olan eşyanın ID'sini döndürür.
    *   `m_adwPartItemID[CRaceData::PART_MAX_NUM]`: Aktörün farklı vücut parçalarına takılı olan eşyaların ID'lerini tutan bir dizidir.

#### `ActorInstanceEvent.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının oyun içindeki çeşitli önemli anlarda (durum değişiklikleri, eylemler, etkileşimler) dış sistemlere (genellikle Python katmanına veya üst seviye oyun mantığına) bilgi göndermesini sağlayan olay (event) mekanizmasını içerir. Bir karakterin harekete başlaması, durması, saldırması, bir yetenek kullanması veya bir başkasına vurması gibi durumlar, bu modül aracılığıyla ilgili olay işleyicilere (event handler) iletilir. Bu, oyun mantığının C++ çekirdeği ile daha üst seviye script veya arayüz mantığı arasında bir köprü görevi görür.

**Temel İşlevler ve Önemli Metotlar (`__On...` serisi):**

Bu metotların çoğu, `CActorInstance` içinde belirli bir durum veya eylem gerçekleştikten sonra çağrılır ve `m_pkEventHandler` (bir `IEventHandler` arayüzü implementasyonu) üzerinden ilgili olayı tetikler. Her olayda genellikle aktörün mevcut konumu (`kPPosSelf`) ve ilerleme yönü (`fAdvRotSelf`) gibi temel durum bilgileri `IEventHandler::SState` yapısıyla iletilir.

*   **Pozisyon ve Durum Senkronizasyonu:**
    *   `__OnSyncing()`: Aktörün pozisyonu ve yönü senkronize edildiğinde tetiklenir. `IEventHandler::OnSyncing` çağrılır.
    *   `__OnWaiting()`: Aktör bekleme durumuna geçtiğinde tetiklenir. `IEventHandler::OnWaiting` çağrılır.
    *   `__OnMoving()`: Aktör hareket etmeye başladığında (hedef pozisyona doğru) tetiklenir. Hedef pozisyon çok uzaksa, bildirilen pozisyon 1000 birimle sınırlandırılır. `IEventHandler::OnMoving` çağrılır.
    *   `__OnMove()`: Aktör hareket halindeyken (muhtemelen her frame veya belirli aralıklarla) tetiklenir. `IEventHandler::OnMove` çağrılır.
    *   `__OnStop()`: Aktör durduğunda tetiklenir. `IEventHandler::OnStop` çağrılır.
    *   `__OnWarp()`: Aktör ışınlandığında tetiklenir. `IEventHandler::OnWarp` çağrılır.
*   **Savaş ve Yetenek Olayları:**
    *   `__OnAttack(WORD wMotionIndex)`: Aktör bir saldırı başlattığında tetiklenir. Saldırı hareketinin indeksi (`wMotionIndex`) ile birlikte `IEventHandler::OnAttack` çağrılır.
    *   `__OnUseSkill(UINT uMotSkill, UINT uLoopCount, bool isMovingSkill)`: Aktör bir yetenek kullandığında tetiklenir. Yetenek ID'si (`uMotSkill`), tekrar sayısı (`uLoopCount`) ve hareketli bir yetenek olup olmadığı bilgisiyle `IEventHandler::OnUseSkill` çağrılır.
    *   `__OnHit(UINT uSkill, CActorInstance& rkActorVictim, BOOL isSendPacket)`: Aktör başka bir aktöre (`rkActorVictim`) bir yetenekle (`uSkill`) vurduğunda tetiklenir. Sunucuya paket gönderilip gönderilmeyeceği (`isSendPacket`) bilgisi de iletilir. `IEventHandler::OnHit` çağrılır.
*   **Efekt (Affect) Olayları:**
    *   `__OnClearAffects()`: Aktör üzerindeki tüm özel durum etkileri (affect'ler) temizlendiğinde tetiklenir. `IEventHandler::OnClearAffects` çağrılır.
    *   `__OnSetAffect(UINT uAffect)`: Aktöre yeni bir özel durum etkisi eklendiğinde tetiklenir. `IEventHandler::OnSetAffect` çağrılır.
    *   `__OnResetAffect(UINT uAffect)`: Aktör üzerinden bir özel durum etkisi kaldırıldığında tetiklenir. `IEventHandler::OnResetAffect` çağrılır.
*   **Olay İşleyici (Event Handler) Yönetimi:**
    *   `SetEventHandler(IEventHandler* pkEventHandler)`: `CActorInstance` için olayları alacak olan `IEventHandler` nesnesini ayarlar.
    *   `__GetEventHandlerRef()` ve `__GetEventHandlerPtr()`: Ayarlanmış olan `IEventHandler` nesnesine referans veya pointer döndürür. Eğer bir handler ayarlanmamışsa `assert` ile hata verir.

**`IEventHandler` Arayüzü ve `CEmptyEventHandler`:**

*   **`CActorInstance::IEventHandler`**: `CActorInstance`'tan olayları almak isteyen sınıfların implemente etmesi gereken soyut bir arayüzdür. `OnSyncing`, `OnWaiting`, `OnAttack`, `OnHit` gibi yukarıda listelenen tüm olaylar için sanal (virtual) metotlar tanımlar.
*   **`IEventHandler::SState`**: Olaylarla birlikte gönderilen temel durum bilgilerini (pozisyon, yön) içeren bir yapıdır.
*   **`CActorInstance::IEventHandler::GetEmptyPtr()`**: Hiçbir işlem yapmayan varsayılan bir `IEventHandler` implementasyonu (`CEmptyEventHandler`) döndürür. Bu, bir `CActorInstance`'ın olay işleyicisi `NULL` olmadığından emin olmak için kullanılır ve olayların işlenmediği durumlarda programın çökmesini engeller.

Bu iki dosya, `CActorInstance`'ın temel verilerinin nasıl yapılandırıldığını, yüklendiğini ve oyun içindeki dinamik olayların nasıl dış sistemlere iletilerek oyun mantığının ve arayüzünün güncellenmesini sağladığını gösterir. 

#### `ActorInstanceFly.cpp`

**Dosyanın Amacı ve Genel Bakış:**

`ActorInstanceFly.cpp` dosyası, `CActorInstance` sınıfının "uçan hedef" (fly target) mekanikleriyle ilgili fonksiyonlarını içerir. Bu, bir aktörün belirli bir nesneyi veya konumu hedef olarak belirlemesi, bu hedefe doğru yönelmesi, hedefin konumunu ve mesafesini sorgulaması gibi yetenekleri kapsar. Genellikle okçuluk gibi menzilli saldırılarda veya belirli bir hedefe kilitlenmeyi gerektiren yeteneklerde kullanılır.

**Oyundaki Kullanım Amacı:**

Bu dosyadaki fonksiyonlar, karakterlerin veya NPC'lerin (Non-Player Character) menzilli saldırılar için bir hedef seçmesini ve bu hedefi takip etmesini sağlar. Örneğin:

*   Bir okçu karakter, düşmanı hedef aldığında, bu mekanizma sayesinde okların hedefe doğru gitmesi sağlanır.
*   Bazı büyüler veya yetenekler, kullanıldığında otomatik olarak bir düşmana kilitlenir; bu kilitlenme `FlyTarget` ile yönetilebilir.
*   Karakterin, uçan bir merminin (örneğin bir ok) varış noktasını belirlemesine yardımcı olur.

**Önemli Fonksiyonlar ve Mekanizmalar:**

*   **Hedef Yönetimi:**
    *   `CFlyTarget` Sınıfı: Uçan hedefin kendisini (bir `CActorInstance` pointer'ı olabilir) veya konumunu saklar.
    *   `m_kFlyTarget`: Mevcut aktif uçan hedef.
    *   `m_kBackupFlyTarget`: Yedek uçan hedef.
    *   `m_kQue_kFlyTarget`: Hedef kuyruğu; mevcut hedef işlenirken yeni hedefler bu kuyruğa eklenebilir.
    *   `SetFlyTarget(const CFlyTarget& cr_FlyTarget)`: Aktif uçan hedefi ayarlar.
    *   `AddFlyTarget(const CFlyTarget& cr_FlyTarget)`: Yeni bir hedef ekler. Eğer aktif bir hedef varsa kuyruğa atar, yoksa doğrudan aktif hedef yapar.
    *   `ClearFlyTarget()`: Tüm uçan hedef bilgilerini temizler.
    *   `IsFlyTargetObject()`: Mevcut hedefin bir nesne olup olmadığını kontrol eder.
*   **Hedef Bilgisi Alma:**
    *   `OnGetFlyTargetPosition()`: Hedefin 3D dünyadaki pozisyonunu hesaplar. Eğer hedef bir nesne ise, o nesnenin merkezini veya ayarlanmış bir "fly adjust" pozisyonunu alır. Binek üzerindeyse yükseklik ayarı da yapar.
    *   `__GetFlyTargetPosition()`: `m_kFlyTarget` içindeki ham hedef pozisyonunu döndürür.
    *   `GetFlyTargetDistance()`: Aktör ile hedef arasındaki (yatay) mesafeyi hesaplar.
*   **Hedefe Yönelme:**
    *   `LookAtFlyTarget()`: Aktörün, mevcut uçan hedefin bulunduğu yöne doğru dönmesini sağlar.
*   **Hedef Değiştirme Kontrolü:**
    *   `CanChangeTarget()`: Aktörün mevcut durumda hedefini değiştirip değiştiremeyeceğini kontrol eder (örneğin, belirli bir animasyon oynatılıyorsa hedef değiştirilemeyebilir). Bu fonksiyon `__IsNeedFlyTargetMotion()` sonucuna bağlıdır.
*   **Olay İşleyici (Event Handler):**
    *   `SetFlyEventHandler(IFlyEventHandler* pHandler)`: Uçuşla ilgili olayları (örneğin hedefe varıldığında) işleyecek bir handler ayarlar.
    *   `ClearFlyEventHandler()`: Ayarlanmış handler'ı temizler.

**Çalışma Prensibi:**

1.  Oyuncu veya yapay zeka, bir hedef seçtiğinde (örneğin, bir düşmana tıklayarak veya bir yetenek kullanarak), bu hedef `SetFlyTarget` veya `AddFlyTarget` aracılığıyla `CActorInstance` için ayarlanır.
2.  Hedeflenen bir nesne ise (`IsFlyTargetObject()` doğru ise), o nesnenin güncel pozisyonu `OnGetFlyTargetPosition` ile takip edilir.
3.  Karakter, menzilli bir saldırı yapacağı zaman `LookAtFlyTarget` fonksiyonu ile hedefe doğru dönebilir.
4.  Atılan mermiler veya yönlendirilen büyüler, `OnGetFlyTargetPosition` tarafından döndürülen konumu kullanarak hedefe doğru ilerler.
5.  `GetFlyTargetDistance` ile hedefe olan mesafe kontrol edilerek menzil dışı durumlar yönetilebilir.
6.  Belirli hareketler veya yetenekler sırasında hedefin değişmesini engellemek için `CanChangeTarget` kontrolü yapılabilir.

Bu mekanizma, özellikle menzilli dövüş ve hedef tabanlı yeteneklerin doğru çalışması için kritik öneme sahiptir.

## `IActorInstance` Arayüzü

### `ActorInstanceInterface.h`

**Dosyanın Amacı ve Genel Bakış:**

`ActorInstanceInterface.h` dosyası, `IActorInstance` adında bir arayüz (interface) sınıfı tanımlar. Bu arayüz, `CGraphicThingInstance` sınıfından türetilmiştir ve oyun dünyasındaki tüm aktör benzeri nesneler (karakterler, canavarlar, NPC'ler vb.) için ortak bir temel kontrat sunar. Soyut bir sınıf olduğu için doğrudan örneklenemez; bunun yerine, `CActorInstance` gibi somut aktör sınıfları bu arayüzü uygular.

**Oyundaki Kullanım Amacı:**

`IActorInstance` arayüzü, oyun motorunun farklı türdeki aktörlerle polimorfik bir şekilde çalışabilmesini sağlar. Bu sayede, bir fonksiyonun veya sistemin, nesnenin spesifik tipini (oyuncu, canavar A, canavar B) bilmeden genel bir "aktör" olarak onunla etkileşim kurmasına olanak tanır. Örneğin, bir çarpışma sistemi, `IActorInstance` pointer'ları üzerinden farklı aktörlerin çarpışmalarını test edebilir.

**Temel Özellikler ve Fonksiyonlar:**

*   **Kalıtım:** `CGraphicThingInstance` sınıfından public olarak kalıtım alır. Bu, `IActorInstance`'ın aynı zamanda bir grafik nesnesi örneği olduğunu ve grafik motoru tarafından yönetilebileceğini gösterir.
*   **`ID` Enum:**
    *   `enum { ID = ACTOR_OBJECT };`
    *   Bu enum, bu arayüzü uygulayan nesnelerin tipini `ACTOR_OBJECT` olarak tanımlar. `GetType()` fonksiyonu bu ID'yi döndürerek nesne tipinin sorgulanmasını sağlar.
*   **Saf Sanal Fonksiyonlar (Pure Virtual Functions):**
    *   `virtual bool TestCollisionWithDynamicSphere(const CDynamicSphereInstance& dsi) = 0;`
        *   Bu fonksiyon, aktörün verilen bir dinamik küre (`CDynamicSphereInstance`) ile çarpışıp çarpışmadığını test etmek için somut alt sınıflar tarafından implemente edilmelidir. Dinamik küreler genellikle mermiler veya kısa süreli etki alanları için kullanılır.
    *   `virtual DWORD GetVirtualID() = 0;`
        *   Bu fonksiyon, aktörün benzersiz bir sanal kimliğini (Virtual ID) döndürmek için somut alt sınıflar tarafından implemente edilmelidir. Bu ID, aktörleri birbirinden ayırmak ve network üzerinden referanslamak için kullanılır.
*   **Yapıcı ve Yıkıcı Fonksiyonlar:**
    *   `IActorInstance() {}`
    *   `virtual ~IActorInstance() {}`
    *   Temel yapıcı ve sanal yıkıcı fonksiyonlar, arayüzün doğru şekilde kullanılmasını ve türetilmiş sınıfların kaynaklarının düzgünce serbest bırakılmasını sağlar.

**Çalışma Prensibi:**

1.  `CActorInstance` gibi somut sınıflar, `IActorInstance` arayüzünü public olarak miras alır.
2.  Bu somut sınıflar, `IActorInstance` içinde tanımlanmış olan tüm saf sanal fonksiyonları (`TestCollisionWithDynamicSphere`, `GetVirtualID`) kendi özel mantıklarına göre implemente ederler.
3.  Oyunun diğer bölümleri (örneğin, hedefleme sistemleri, yetenek yöneticileri, UI elemanları), aktörlerle doğrudan `CActorInstance` üzerinden değil, `IActorInstance*` (pointer) veya `IActorInstance&` (referans) üzerinden etkileşim kurabilir.
4.  Bu sayede, bu sistemlerin belirli bir aktör tipine bağımlılığı azalır ve koda esneklik katılır. Örneğin, bir `std::vector<IActorInstance*>` listesi, hem oyuncuları hem de farklı türdeki canavarları bir arada tutabilir ve hepsi üzerinde ortak işlemler (örneğin, `GetVirtualID()` çağrısı) yapılabilir.

`IActorInstance` arayüzü, Metin2 istemcisindeki aktör yönetim sisteminin temel taşlarından biridir ve nesne yönelimli programlamanın polimorfizm ilkesini etkin bir şekilde kullanır.

#### `ActorInstanceMotion.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının animasyon (motion) sisteminin temel mantığını içerir. Karakterlerin ve diğer varlıkların hangi animasyonu oynatacağını, animasyonlar arası geçişlerin nasıl yapılacağını, animasyonların ne kadar süreceğini, tekrar edip etmeyeceğini ve hangi hızda oynatılacağını yönetir. Aynı zamanda, animasyon kuyruğu (`m_MotionDeque`) sayesinde animasyonların sıralı bir şekilde veya birbirini keserek (intercept) oynatılmasını sağlar. Ata binme durumundaki animasyon senkronizasyonları ve çeşitli animasyon modları (bekleme, yürüme, koşma, saldırı, yetenek kullanımı, sosyal hareketler vb.) da bu dosyadaki fonksiyonlarla kontrol edilir.

**Temel İşlevler ve Önemli Mekanizmalar:**

*   **Animasyon İşleme Döngüsü:**
    *   `MotionProcess(BOOL isPC)`: Her frame'de çağrılan ana animasyon güncelleme fonksiyonudur. Sırasıyla `__MotionEventProcess` (animasyon olaylarını ve seslerini işler), `CurrentMotionProcess` (mevcut animasyonun durumunu yönetir) ve `ReservingMotionProcess` (kuyruktaki bir sonraki animasyonu başlatır) fonksiyonlarını çağırır.
    *   `HORSE_MotionProcess(BOOL isPC)`: Eğer karakter ata binmişse, atın animasyon döngüsünü yönetmek için çağrılır.
    *   `__MotionEventProcess(BOOL isPC)`: Mevcut animasyon karesi için tanımlanmış animasyon olaylarını (`MotionEventProcess()`) ve ses olaylarını (`SoundEventProcess()`) tetikler. Saldırı animasyonları için kare ilerlemesini saldırı süresine göre ayarlar.
*   **Animasyon Kuyruğu ve Durumu (`m_MotionDeque`, `m_kCurMotNode`):**
    *   `m_MotionDeque` (`TMotionDeque`): Oynatılmayı bekleyen animasyonların bir listesidir. Her bir `TReservingMotionNode` animasyonun türünü, anahtarını, başlama zamanını, karışım süresini (blend time), hızını ve süresini tutar.
    *   `m_kCurMotNode` (`SCurrentMotionNode`): O anda oynatılan animasyonun detaylarını (tür, anahtar, başlangıç/bitiş zamanı, mevcut kare, toplam kare, yetenek ID'si, hız oranı, tekrar sayısı vb.) tutar.
    *   `ReservingMotionProcess()`: `m_MotionDeque` boş değilse ve kuyruktaki ilk animasyonun başlama zamanı gelmişse, o animasyonu `__SetMotion` ile başlatır ve kuyruktan çıkarır. Ölüm, bayılma gibi durumları kontrol ederek animasyon başlatmayı engelleyebilir.
    *   `CurrentMotionProcess()`: Mevcut animasyonun (`m_kCurMotNode`) durumunu günceller. Döngüsel animasyonların tekrarını, tek seferlik animasyonların bitiminde varsayılan `WAIT` animasyonuna geçişi yönetir. Ayrıca, uçan hedef (`FlyTarget`) kuyruğunu işleyerek döngüsel animasyonlar sırasında hedef değiştirmeyi sağlar.
*   **Animasyon Ayarlama ve Değiştirme:**
    *   `SetMotionMode(int iMotionMode)`: Aktörün animasyon modunu (`m_wcurMotionMode`) ayarlar (örneğin, `CRaceMotionData::MODE_GENERAL`, `MODE_TWOHAND_SWORD`). Eğer karakter bir dönüşüm (poly) durumundaysa, modu `MODE_GENERAL` olarak zorlar.
    *   `SetMotionLoopCount(int iCount)`: Mevcut animasyonun tekrar sayısını (`m_kCurMotNode.iLoopCount`) ayarlar.
    *   `PushMotion(EMotionPushType iMotionType, DWORD dwMotionKey, float fBlendTime, float fSpeedRatio)`: Verilen animasyonu (`dwMotionKey`) ve parametrelerini `m_MotionDeque` kuyruğuna ekler.
    *   `InterceptOnceMotion(DWORD dwMotion, ...)` ve `InterceptLoopMotion(DWORD dwMotion, ...)`: Mevcut animasyonları temizleyerek (`__ClearMotion`) hemen yeni bir tek seferlik veya döngüsel animasyon başlatır. Aslında `InterceptMotion`'ı çağırırlar.
    *   `SetLoopMotion(DWORD dwMotion, ...)`: Mevcut animasyonları temizleyerek hemen yeni bir döngüsel animasyon başlatır (doğrudan `__SetMotion` çağırır).
    *   `InterceptMotion(EMotionPushType iMotionType, WORD wMotion, ...)`: Belirli bir ırk animasyonunu (`wMotion`) ve modunu (`m_wcurMotionMode`) kullanarak `MOTION_KEY` alır, mevcut animasyonları temizler ve `__SetMotion` ile yeni animasyonu başlatır. Eğer animasyon uçan hedef gerektiriyorsa (`__IsNeedFlyTargetMotion`) ilgili olayı (`m_pFlyEventHandler->OnSetFlyTarget()`) tetikler.
    *   `__SetMotion(const SSetMotionData& c_rkSetMotData, DWORD dwRandMotKey = 0)`: Animasyonları ayarlayan çekirdek fonksiyondur.
        *   Gerekirse rastgele bir animasyon varyasyonu seçer (`GetRandomMotionKey`).
        *   Çeşitli durum kontrolleri yapar (ölü mü, yetenek kullanıyor mu, ayakta mı, hareket ediyor mu, gizleniyor mu vb.) ve bu durumlara göre `__OnStop()`, `__OnMove()`, `__ShowEvent()` gibi olayları tetikler.
        *   Balık tutma efektini temizler.
        *   Eğer ata binilmişse (`m_pkHorse`), hem karakterin hem de atın animasyonunu ayarlar ve senkronize eder.
        *   `CGraphicThingInstance::SetMotion` veya `ChangeMotion` ile grafik katmanında animasyonu ayarlar.
        *   Silah izlerini gizler/gösterir (`__HideWeaponTrace`, `__ShowWeaponTrace`).
        *   `__BindMotionData` ile yeni animasyonun verilerini (`m_pkCurRaceMotionData`) yükler.
        *   Tekrar sayısını ve saldırı/kombo durumlarını günceller.
    *   `__ClearMotion()`: Animasyon kuyruğunu ve mevcut animasyon bilgisini (`m_kCurMotNode`) temizler, silah izlerini gizler.
*   **Animasyon Bilgisi ve Kontrolleri:**
    *   `__GetCurrentMotionIndex()`, `__GetCurrentMotionKey()`: Mevcut oynatılan animasyonun indeksini ve tam anahtarını döndürür.
    *   `GetMotionDuration(DWORD dwMotionKey)`: Verilen animasyon anahtarının süresini (saniye cinsinden) döndürür.
    *   `GetLastMotionTime(float fBlendTime)`: Animasyon kuyruğundaki son animasyonun veya mevcut animasyonun (eğer kuyruk boşsa) ne zaman biteceğini (veya yeni bir animasyonun ne zaman başlayabileceğini) hesaplar.
    *   `GetRandomMotionKey(MOTION_KEY dwMotionKey)`: Bir ana animasyon için tanımlanmışsa, olasılıklara göre rastgele bir alt animasyon (varyasyon) seçer.
    *   `IsUsingSkill()`: Aktörün bir yetenek (`SKILL` veya `SPECIAL`) animasyonu oynatıp oynatmadığını kontrol eder.
    *   `IsFishing()`: Aktörün balık tutma (`FISHING_WAIT`, `FISHING_REACT`) animasyonlarından birini oynatıp oynatmadığını kontrol eder.
    *   `CanCancelSkill()`: Mevcut yetenek animasyonunun, animasyon verisinde (`m_pkCurRaceMotionData`) tanımlandığı üzere iptal edilebilir olup olmadığını kontrol eder.
    *   `isLock()`: Aktörün, mevcut animasyonu nedeniyle başka bir eylem yapamayacak şekilde "kilitli" olup olmadığını kontrol eder. Saldırı, balık tutma, birçok sosyal/duygu animasyonu ve iptal edilemeyen yetenekler sırasında `TRUE` döner.
    *   `__GetMotionType()`: Mevcut animasyonun genel tipini (`CRaceMotionData::TYPE_NONE`, `TYPE_WAIT`, `TYPE_MOVE`, `TYPE_ATTACK` vb.) döndürür.
    *   `__BindMotionData(DWORD dwMotionKey)`: Verilen animasyon anahtarı için `CRaceData`'dan ilgili `CRaceMotionData` pointer'ını (`m_pkCurRaceMotionData`) yükler.
    *   `__GetLoopCount()`: Mevcut animasyon verisinden (`m_pkCurRaceMotionData`) varsayılan tekrar sayısını alır.
    *   `__CanAttack()`, `__CanNextComboAttack()`, `__IsComboAttacking()`: Mevcut animasyonun saldırı yapmaya uygun olup olmadığını, bir sonraki komboya geçilip geçilemeyeceğini ve kombo saldırısı durumunda olup olmadığını kontrol eder.
    *   `__IsNeedFlyTargetMotion()`, `__HasMotionFlyEvent()`: Mevcut animasyonun uçan bir hedef gerektirip gerektirmediğini veya uçuşla ilgili bir olay içerip içermediğini kontrol eder.
    *   `__IsWaitMotion()`, `__IsMoveMotion()`, `__IsAttackMotion()`, `__IsDamageMotion()`, vb.: Mevcut animasyonun belirli bir kategoride olup olmadığını kontrol eden yardımcı fonksiyonlar.


#### `ActorInstanceMotionEvent.cpp` Alt Modülü

**Oyundaki Kullanım Amacı:**

Bu bölüm, `CActorInstance` sınıfının, oynatılan animasyonların (`CRaceMotionData`) belirli karelerinde tanımlanmış olayları (motion events) nasıl işlediğini yönetir. Animasyonlar sadece görsel hareketlerden ibaret değildir; aynı zamanda seslerin çalınması, efektlerin oluşturulması, saldırıların gerçekleşmesi, karakterin ışınlanması veya gizlenip görünür hale gelmesi gibi çeşitli oyun mekaniklerini tetikleyebilirler. Bu dosya, bu tür olayların doğru zamanda ve doğru şekilde gerçekleşmesini sağlar.

**Temel İşlevler ve Önemli Mekanizmalar:**

*   **Ana Olay İşleme Akışı:**
    *   `MotionEventProcess()`: Bu fonksiyon, mevcut animasyon verisi (`m_pkCurRaceMotionData`) içinde tanımlanmış tüm "motion event" tanımlarını döngüye alır.
    *   `MotionEventProcess(DWORD dwcurFrame, int iIndex, const CRaceMotionData::TMotionEventData* c_pData)`: Her bir animasyon olayı için çağrılır. Eğer olayın tanımlandığı kare (`c_pData->dwFrame`), animasyonun o anki karesine (`dwcurFrame`) eşitse, olayın tipine (`c_pData->iType`) göre ilgili `ProcessMotionEvent...` fonksiyonunu çağırır.
    *   `SoundEventProcess(BOOL bCheckFrequency)`: Mevcut animasyon karesiyle ilişkili ses olaylarını `CSoundManager` aracılığıyla işler. `bCheckFrequency` ile sesin ne sıklıkta çalınabileceği kontrol edilebilir.
*   **Animasyon Olayı Türüne Göre İşleme Fonksiyonları:**
    *   **Efektler:**
        *   `ProcessMotionEventEffectEvent(const CRaceMotionData::TMotionEffectEventData* c_pData)`: `MOTION_EVENT_TYPE_EFFECT` türündeki olayları işler.
            *   `isIndependent`: Efekt, aktörden bağımsız olarak dünya pozisyonunda oluşturulur.
            *   `isAttaching`: Efekt, aktörün belirli bir kemiğine (`strAttachingBoneName`) veya genel olarak aktöre eklenir.
            *   `isFollowing`: Eğer `isAttaching` ise, efekt kemiği/aktörü takip eder. Takip etmiyorsa, kemiğin o anki pozisyonunda tek seferlik oluşturulur.
            *   Efektler `CEffectManager` aracılığıyla oluşturulur ve yönetilir.
        *   `ProcessMotionEventEffectToTargetEvent(const CRaceMotionData::TMotionEffectToTargetEventData* c_pData)`: `MOTION_EVENT_TYPE_EFFECT_TO_TARGET` türündeki olayları işler.
            *   `isFishingEffect`: Balık tutma animasyonları için özeldir; efekti `m_v3FishingPosition` (oltacının ucunun olduğu varsayılan pozisyon) üzerinde oluşturur.
            *   Diğer durumlarda, aktörün mevcut uçan hedefini (`m_kFlyTarget`) kullanır. Efekt, hedefin pozisyonuna göre (isteğe bağlı `v3EffectPosition` ofsetiyle) veya eğer `isFollowing` ise doğrudan hedefe eklenerek oluşturulur.
    *   **Ekran Sallanması:**
        *   `CGameEventManager::Instance().ProcessEventScreenWaving(...)`: `MOTION_EVENT_TYPE_SCREEN_WAVING` türündeki olayları `CGameEventManager`'a yönlendirerek ekran sallanma efekti oluşturur.
    *   **Özel Saldırılar (Alan Etkili):**
        *   `ProcessMotionEventSpecialAttacking(int iMotionEventIndex, const CRaceMotionData::TMotionAttackingEventData* c_pData)`: `MOTION_EVENT_TYPE_SPECIAL_ATTACKING` türündeki olayları işler. Genellikle alan etkili saldırılar için kullanılır.
            *   Aktörün `m_kSplashArea` yapısını, saldırının çarpışma küreleri (`CollisionData`), süresi, yetenek ID'si gibi bilgilerle doldurur. Bu `m_kSplashArea`, daha sonra `ActorInstanceCollisionDetection.cpp` içinde hedeflere hasar vermek için kullanılır.
    *   **Sesler:**
        *   `ProcessMotionEventSound(const CRaceMotionData::TMotionSoundEventData* c_pData)`: `MOTION_EVENT_TYPE_SOUND` türündeki olayları işler. `CSoundManager::Instance().PlaySound3D` ile 3 boyutlu ses çalar.
    *   **Uçan Nesneler (Mermiler):**
        *   `ProcessMotionEventFly(const CRaceMotionData::TMotionFlyEventData* c_pData)`: `MOTION_EVENT_TYPE_FLY` türündeki olayları işler.
            *   Eğer geçerli bir uçan hedef (`m_kFlyTarget`) varsa, `CFlyingManager` aracılığıyla bir `CFlyingInstance` (mermi, ok vb.) oluşturur.
            *   Merminin başlangıç pozisyonu (`v3FlyPosition`), aktörün pozisyonuna ve isteğe bağlı olarak bir kemiğe (`strAttachingBoneName`) göre ayarlanır.
            *   Oluşturulan mermiye sahip (`owner`), yetenek indeksi ve olay işleyici (`m_pFlyEventHandler`) atanır.
            *   `ENABLE_QUIVER_SYSTEM` tanımlıysa ve karakter uygun bir ırktaysa, ok çantası efektini kullanabilir.
    *   **Karakter Görünürlüğü:**
        *   `__ShowEvent()` (`MOTION_EVENT_TYPE_CHARACTER_SHOW`): Karakteri görünür yapar (`m_isHiding = FALSE`), render modunu normale döndürür ve alfa değerini 1.0 yapar.
        *   `__HideEvent()` (`MOTION_EVENT_TYPE_CHARACTER_HIDE`): Karakteri gizler (`m_isHiding = TRUE`), render modunu blend yapar ve alfa değerini 0.0 yapar.
    *   **Işınlanma (Warp):**
        *   `ProcessMotionEventWarp(const CRaceMotionData::TMotionEventData* c_pData)`: `MOTION_EVENT_TYPE_WARP` olaylarını işler (eğer `WORLD_EDITOR` tanımlı değilse).
            *   Aktörü, mevcut uçan hedefine (`m_kFlyTarget`) doğru, hedeften belirli bir mesafe (`sc_fDistanceFromTarget`) kalacak şekilde ışınlar.
            *   Işınlanma öncesinde hedef noktanın `IBackground::IsBlock` ile geçilebilir olup olmadığını kontrol eder.
            *   Aktörün hedefe bakmasını (`LookAt`) sağlar ve `__OnWarp()` olayını tetikler.
    *   **Göreli Hareket:**
        *   `ProcessMotionEventRelativeMoveOn(const CRaceMotionData::TMotionRelativeMoveOn* c_pData)`: `MOTION_EVENT_TYPE_RELATIVE_MOVE_ON` olayını işler. Hedefe doğru göreceli hareket modunu (`m_bIsRelativeMoveMode = true`) aktif eder ve hareket çarpımını (`m_fRelativeMoveMul`) hedefe olan mesafeye ve taban hıza göre hesaplar. Bu, karakterin bir hedefe doğru yavaşlayarak veya hızlanarak ilerlemesini sağlayabilir.
        *   `ProcessMotionEventRelativeMoveOff(...)`: `MOTION_EVENT_TYPE_RELATIVE_MOVE_OFF` olayını işleyerek göreceli hareket modunu kapatır (`m_bIsRelativeMoveMode = false`).

Bu iki dosya, `CActorInstance`'ın animasyonlarının nasıl oynatıldığını, sıralandığını ve bu animasyonlar sırasında hangi oyun içi olayların (efektler, sesler, saldırılar, özel hareketler vb.) tetiklendiğini yöneten kritik bir altyapı sunar.

### `ActorInstancePosition.cpp`

**Oyundaki Kullanım Amacı:**
Bu dosya, `CActorInstance` sınıfının bir parçası olarak, oyun dünyasındaki aktörlerin (karakterler, canavarlar, NPC'ler vb.) pozisyonlarını ve hareket hedeflerini yönetmekten sorumludur. Aktörlerin nerede durduğunu, nereye hareket ettiğini ve saldırı gibi eylemler için hangi pozisyonları hedeflediğini belirler. Bu, oyun mantığının ve grafiksel gösterimin temelini oluşturur.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**
`ActorInstancePosition.cpp` içindeki fonksiyonlar, `CActorInstance` nesnelerinin konumla ilgili verilerini ayarlar ve bu verilere erişim sağlar. Temel çalışma prensipleri şunlardır:

*   **Pozisyon Türleri:**
    *   **Mevcut Pozisyon (`m_kPPosCur`):** Aktörün anlık dünya koordinatlarını (x, y, z) ifade eder. `SetPixelPosition`, `SCRIPT_SetPixelPosition` gibi fonksiyonlarla ayarlanır ve `GetPixelPosition`, `SCRIPT_GetPixelPosition`, `GetXYZ` gibi fonksiyonlarla okunur.
    *   **Hedef Pozisyon (`m_kPPosDst`):** Aktörün hareket etmeyi amaçladığı hedef dünya koordinatlarıdır. Özellikle hareket komutları (örneğin, sunucudan gelen) işlenirken kullanılır. `SetDestinationPixelPosition` ve `SetDestinationTime` ile ayarlanır.
    *   **Saldırı Pozisyonu (`m_kPPosAtk`):** Aktörün bir saldırı gerçekleştireceği hedef pozisyonu belirtir. Genellikle saldırı animasyonları ve mantığıyla birlikte kullanılır. `SetAttackPixelPosition` ile ayarlanır.
    *   **Kaynak/Başlangıç Pozisyonu (`m_kPPosSrc`):** Bir hareketin başladığı veya önceki önemli bir pozisyonu temsil edebilir. `SetSourcePixelPosition` ile ayarlanır.
    *   **Son Piksel Pozisyonu (`m_kPPosLast`):** Aktörün bir önceki güncellemedeki pozisyonunu saklar; genellikle enterpolasyon veya hız hesaplamaları için kullanılabilir.
*   **Koordinat Sistemleri:**
    *   Genellikle oyun içi mantık için dünya koordinatları (float) ve grafiksel gösterim/piksel bazlı işlemler için piksel koordinatları (float, ancak genellikle tam sayıya yakın değerlerle çalışır) arasında dönüşümler ve yönetim söz konusudur. `TPixelPosition` sıkça kullanılır.
*   **Pozisyon Ayarlama ve Alma:**
    *   `SetPixelPosition(float x, float y, float z)`: Aktörün mevcut dünya pozisyonunu doğrudan ayarlar.
    *   `SCRIPT_SetPixelPosition(float x, float y)`: Python betiklerinden çağrılmak üzere aktörün x ve y pozisyonunu ayarlar (z genellikle arazi yüksekliğinden alınır).
    *   `GetPixelPosition()`: Mevcut piksel pozisyonunu `TPixelPosition` olarak döndürür.
    *   `SetDestinationPixelPosition(const TPixelPosition& c_rkPPosDst)`: Hareket hedefini ayarlar.
    *   `GetDestinationPixelPosition()`: Ayarlanmış hareket hedefini döndürür.
    *   `SetAttackPixelPosition(const TPixelPosition& c_rkPPosAtk)`: Saldırı hedef pozisyonunu ayarlar.
*   **Hareket Süresi:**
    *   `SetDestinationTime(float fTime)`: Hedef pozisyona ne kadar sürede ulaşılacağını belirler. Bu bilgi, hareketin hızını ve yumuşaklığını ayarlamak için kullanılır.
*   **Durum ve Bilgi Fonksiyonları:**
    *   `IsGoing()`: Aktörün bir hedefe doğru hareket edip etmediğini kontrol eder.
    *   `RefreshActorInstance()`: Muhtemelen aktörün pozisyonuna bağlı bazı iç durumları günceller.

Bu dosya, aktörün oyun dünyasındaki fiziksel varlığının temelini oluşturur ve diğer modüllerin (hareket, render, çarpışma tespiti vb.) doğru çalışması için kritik öneme sahiptir.

---

### `ActorInstanceRender.cpp`

**Oyundaki Kullanım Amacı:**
Bu dosya, `CActorInstance` sınıfının bir parçası olarak, aktörlerin (karakterler, canavarlar, NPC'ler vb.) oyun dünyasında nasıl görüneceğini, yani ekrana nasıl çizileceğini (render edileceğini) yönetir. Bu, aktörün 3D modelinin, dokularının, materyal özelliklerinin, gölgelerinin ve diğer görsel efektlerinin doğru bir şekilde işlenmesini kapsar.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**
`ActorInstanceRender.cpp`, aktörlerin görselleştirilmesiyle ilgili çeşitli fonksiyonları ve mantığı içerir. Temel çalışma prensipleri şunlardır:

*   **Ana Çizim Fonksiyonları:**
    *   `Render()`: Aktörün ana çizim işlemidir. Bu fonksiyon, aktörün modelini, varsa eklenmiş efektleri ve diğer standart görsel bileşenlerini ekrana çizer.
    *   `BlendRender()`: Genellikle yarı saydam (alpha blending) veya özel efekt gerektiren aktörlerin (örneğin, hayaletler, bazı büyüler) çizimi için kullanılır. Bu, diğer nesnelerin arkasından doğru şekilde görünebilmelerini sağlar.
    *   `RenderToShadowMap()`: Aktörün gölgesinin oluşturulacağı gölge haritasına (shadow map) çizimini gerçekleştirir. Bu, dinamik gölgelerin oluşturulması için kritik bir adımdır.
    *   `RenderCollision()`: Hata ayıklama (debug) amacıyla aktörün çarpışma geometrisini (örneğin, bounding box, sphere) ekrana çizer. Bu, geliştiricilerin çarpışma sorunlarını görsel olarak teşhis etmesine yardımcı olur.
*   **Çizim Modları ve Nitelikleri:**
    *   `SetRenderMode(int iRenderMode)`: Aktörün nasıl çizileceğini belirleyen render modunu ayarlar (örneğin, `RENDER_MODE_NORMAL`, `RENDER_MODE_BLEND`).
    *   `SetAddRenderAttribute(int iAttribute)`: Ek çizim nitelikleri ekler (örneğin, belirli bir ışıklandırma efekti).
    *   `ClearAddRenderAttribute()`: Eklenmiş tüm çizim niteliklerini temizler.
*   **Materyal ve Renk Yönetimi:**
    *   `SetMaterialColor(DWORD dwColor)`: Aktörün materyalinin ana rengini ayarlar.
    *   `SetMaterialAlpha(float fAlpha)`: Aktörün materyalinin alfa (saydamlık) değerini ayarlar.
    *   `RestoreMaterial()`: Materyal özelliklerini varsayılan değerlerine döndürür.
*   **Deformasyon ve Animasyon:**
    *   `Deform()`: İskelet animasyonları için gereklidir. Vertex'leri (modelin köşe noktalarını) kemiklerin hareketine göre deforme ederek animasyonlu bir görünüm oluşturur. Genellikle her frame'de çağrılır.
*   **Ekli Efektler:**
    *   `UpdateAttachingEffect()`: Aktöre bağlı olan parçacık efektleri veya diğer görsel efektlerin güncellenmesini ve çizimini yönetir.
*   **Görünürlük ve Çizim Kontrolü:**
    *   `Show()` / `Hide()`: Aktörün görünür olup olmayacağını kontrol eder. `IsShow()` ile durum kontrol edilir.
    *   `EnableBlendRender()` / `DisableBlendRender()`: `BlendRender` modunu aktif hale getirir veya devre dışı bırakır.
*   **Hata Ayıklama (Debug) Çizimleri:**
    *   `RenderTrace()`: Aktörün hareket izini (trace) çizebilir (debug amaçlı).
    *   `RenderAttachingCollision()`: Aktöre bağlı çarpışma şekillerini çizer.
*   **Çeşitli Yardımcı Fonksiyonlar:**
    *   `UpdateTransform(D3DXMATRIX* pMatrix)`: Aktörün transformasyon matrisini (pozisyon, rotasyon, ölçek) günceller.
    *   `GetBoundBox(D3DXVECTOR3* vtMin, D3DXVECTOR3* vtMax)`: Aktörün sınırlayıcı kutusunun (bounding box) minimum ve maksimum noktalarını alır.

Bu dosya, aktörlerin oyun ekranında canlı ve doğru bir şekilde temsil edilmesini sağlar, bu da oyunun görsel kalitesi ve atmosferi için hayati öneme sahiptir.

### `ActorInstanceRotation.cpp`

**Oyundaki Kullanım Amacı:**
Bu dosya, `CActorInstance` sınıfının bir parçası olarak, oyun dünyasındaki aktörlerin (karakterler, canavarlar, NPC'ler vb.) yönelimlerini (rotasyonlarını) yönetmekten sorumludur. Aktörlerin hangi yöne baktığını, hedeflere nasıl döndüğünü ve bu dönüşlerin anlık mı yoksa yumuşak bir geçişle mi olacağını belirler.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**
`ActorInstanceRotation.cpp` içindeki fonksiyonlar, `CActorInstance` nesnelerinin Y ekseni etrafındaki dönüşünü (yaw) ve potansiyel olarak X/Y eksenlerindeki eğimlerini kontrol eder. Temel çalışma prensipleri şunlardır:

*   **Temel Rotasyon Ayarları:**
    *   `SetRotation(float fRot)`: Aktörün yönünü anında belirtilen açıya (`fRot`) ayarlar. Devam eden yumuşak dönüşleri iptal eder.
    *   `SetXYRotation(float fRotX, float fRotY)`: Aktörün X ve Y eksenlerindeki eğimini ayarlar. Bu genellikle standart karakter hareketlerinden ziyade özel durumlar veya efektler için kullanılır.
*   **Yumuşak Dönüş (Blending):**
    *   `BlendRotation(float fRot, float fBlendTime)`: Aktörün mevcut yönünden hedef yöne (`fRot`) doğru, belirtilen süre (`fBlendTime`) boyunca yumuşak bir geçiş yapmasını sağlar. Bu, `m_rotBegin` (başlangıç açısı), `m_rotEnd` (hedef açı), `m_rotBlendTime` (geçiş süresi), `m_rotBeginTime` (başlangıç zamanı) ve `m_rotEndTime` (bitiş zamanı) üyelerini kullanarak enterpolasyon yapar.
*   **İlerleme Rotasyonu:**
    *   `SetAdvancingRotation(float fRot)`: Genellikle hareket yönüyle ilişkili olan `m_fAdvancingRotation` değerini ayarlar. `RotationProcess` içinde mevcut hesaplanan rotasyonla güncellenir.
*   **Rotasyon Süreci:**
    *   `RotationProcess()`: Her oyun döngüsünde çağrılır. Aktif bir yumuşak dönüş varsa, `GetInterpolatedRotation` fonksiyonu aracılığıyla mevcut rotasyonu (`m_fcurRotation`) günceller. Ardından, `m_rotX` ve `m_rotY` değerlerine bağlı olarak `CGraphicObjectInstance::SetRotation()` çağrılarak grafik nesnesinin nihai 3D yönü ayarlanır.
*   **Hedefe Bakma (`LookAt`) Fonksiyonları:**
    *   Çeşitli `LookAt` overload'ları bulunur: Belirli bir açıya, belirli (x,y) koordinatlarına veya başka bir `CActorInstance`'a doğru bakmayı sağlar. Genellikle `BlendRotation` kullanarak 0.3 saniyelik yumuşak bir dönüş gerçekleştirirler.
    *   `LookWith(CActorInstance* pInstance)`: Başka bir aktörün hedeflediği yöne doğru bakar.
*   **Durum ve Bilgi Alma:**
    *   `GetRotation()`: Anlık rotasyonu döndürür.
    *   `GetTargetRotation()`: Yumuşak dönüşün hedef rotasyonunu döndürür.
    *   `GetRotatingTime()`: Yumuşak dönüşün bitiş zamanını döndürür.
    *   `GetAdvancingRotation()`: İlerleme ile ilişkili rotasyonu döndürür.
*   **At Entegrasyonu:**
    *   Eğer aktör bir ata binmişse (`m_pkHorse` geçerliyse), birçok rotasyon fonksiyonu (örneğin `SetRotation`, `BlendRotation`, `SetAdvancingRotation`) atın da ilgili rotasyon fonksiyonunu çağırır.

Bu dosya, aktörlerin çevreleriyle etkileşimde bulunurken veya hareket ederken doğal ve tepkisel görünmelerini sağlayan önemli bir bileşendir.

---

### `ActorInstanceSync.cpp`

**Oyundaki Kullanım Amacı:**
Bu dosya, `CActorInstance` sınıfının bir parçası olarak, aktörlerin pozisyonlarının senkronize edilmesi veya dış etkilerle (örneğin, bir yetenek tarafından itilme) zorla değiştirilmesi gibi durumları yönetir. Ayrıca, bir aktörün oyuncu kontrolü dışında olup olmadığını belirleyen senkronizasyon durum kontrollerini içerir.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**
`ActorInstanceSync.cpp` içindeki fonksiyonlar, aktörlerin özellikle sunucu komutları veya özel oyun mekanikleri sonucu pozisyonlarının ani veya kontrollü değişimini ve bu sırada yapılabilecek durum kontrollerini ele alır.

*   **İtme Mekanizması (`__Push`):**
    *   `__Push(int x, int y)`: Aktörü belirtilen dünya koordinatlarına (x, -y) doğru "itmeye" çalışır.
    *   **Düşme Direnci:** Eğer aktör `IsResistFallen()` ile düşmeye karşı dirençliyse, itme işlemi gerçekleşmez.
    *   **Çarpışma Kontrolü:** Kaynak pozisyondan hedef pozisyona doğru 100 adımda bir yol izlenir. Her adımda `IPhysicsWorld::isPhysicalCollision()` kullanılarak fiziksel bir çarpışma olup olmadığı kontrol edilir. Eğer herhangi bir adımda çarpışma tespit edilirse, aktörün pozisyon harmanlaması sıfırlanır (`ResetBlendingPosition()`) ve itme iptal edilir. Bu, aktörlerin engellerin (duvarlar, diğer nesneler vb.) içinden geçmesini engeller.
    *   **Pozisyon Ayarlama:** Eğer yol boyunca çarpışma olmazsa, aktörün hedef pozisyonu `SetBlendingPosition()` ile ayarlanır, bu da genellikle yumuşak bir pozisyon geçişi başlatır.
    *   **Animasyon Tetikleme:** Eğer aktör bir yetenek kullanmıyorsa ve itme mesafesi belirli bir eşiği (150 birim) aşıyorsa, aktörün `CRaceMotionData::NAME_DAMAGE_FLYING` (hasar alıp uçma) animasyonunu oynaması ve ardından `CRaceMotionData::NAME_STAND_UP` (ayağa kalkma) animasyonunu kuyruğa alması sağlanır. Bu, güçlü itmelerde görsel bir etki yaratır.
    *   `TEMP_Push(int x, int y)`: `__Push` fonksiyonunu çağıran bir sarmalayıcıdır.
*   **Senkronizasyon Durumu (`__IsSyncing`, `IsPushing`):**
    *   `__IsSyncing()`: Aktörün normal oyuncu kontrolü dışında bir durumda olup olmadığını belirler. Şu durumlarda `true` döner:
        *   `IsDead()`: Aktör ölü.
        *   `IsStun()`: Aktör sersemlemiş.
        *   `IsPushing()`: Aktör itiliyor.
    *   `IsPushing()`: Aktörün aktif olarak bir itme kuvveti etkisi altında olup olmadığını kontrol eder. Aktör düşmeye karşı dirençli değilse ve fizik nesnesi (`m_PhysicsObject`) bir pozisyon harmanlaması yapıyorsa (`isBlending()`) `true` döner.

Bu dosya, özellikle ağ senkronizasyonu ve karaktere dışarıdan etki eden kuvvetlerin (knockback, itme vb.) oyun mantığına ve görselliğine doğru bir şekilde yansıtılması için önemlidir. Çarpışma kontrolü, bu tür hareketlerin oyun dünyasının kuralları içinde kalmasını sağlar.

### `ActorInstanceWeaponTrace.cpp`

**Oyundaki Kullanım Amacı:**
Bu dosya, `CActorInstance` sınıfının bir parçası olarak, aktörlerin kullandığı silahların hareketleri sırasında oluşturduğu görsel izleri (genellikle "kılıç izi" veya "efekt izi" olarak bilinir) yönetmekten sorumludur. Bu izler, özellikle saldırı ve yetenek animasyonları sırasında silahın havada çizdiği yolu vurgulayarak dövüşe daha dinamik ve görsel olarak çekici bir hava katar.

**Orta Seviye Öz Bilgi / Temel Çalışma Prensibi:**
`ActorInstanceWeaponTrace.cpp`, `CActorInstance`'a bağlı `CWeaponTrace` nesnelerinin bir koleksiyonunu (`m_WeaponTraceVector`) yönetir. Her bir `CWeaponTrace` nesnesi, silahın belirli bir noktasından çıkan ve zamanla güncellenen bir görsel izi temsil eder.

*   **İzlerin Yaşam Döngüsü ve Yönetimi:**
    *   **Oluşturma:** Silah izleri genellikle bir saldırı animasyonu başladığında veya belirli bir yetenek kullanıldığında `CActorInstance`'a eklenir (bu dosyanın dışında, muhtemelen `SetMotion` veya benzeri bir animasyon ayarlama fonksiyonunda `CWeaponTrace` nesneleri oluşturulup `m_WeaponTraceVector`'a eklenir).
    *   **Güncelleme (`TraceProcess()`):**
        *   Her oyun döngüsünde (frame) çağrılır.
        *   Vektördeki her bir `CWeaponTrace` nesnesi için:
            *   `SetPosition(m_x, m_y, m_z)`: İzin pozisyonunu, aktörün mevcut dünya pozisyonuna günceller.
            *   `SetRotation(m_fcurRotation)`: İzin rotasyonunu, aktörün mevcut yönüne göre ayarlar.
            *   `Update(__GetReachScale())`: `CWeaponTrace` nesnesinin kendi iç mantığını (örneğin, izin şekli, uzunluğu, solma efekti) günceller. `__GetReachScale()` muhtemelen aktörün saldırı menzili veya boyutuna bağlı bir ölçekleme faktörü sağlar.
    *   **Çizim (`RenderTrace()`):**
        *   Oyunun çizim (render) aşamasında çağrılır.
        *   `m_WeaponTraceVector` içindeki her bir `CWeaponTrace` nesnesinin `Render()` metodunu çağırarak izin ekrana çizilmesini sağlar.
    *   **Gösterme/Gizleme:**
        *   `__ShowWeaponTrace()`: Tüm aktif silah izlerini görünür yapar (her `CWeaponTrace` için `TurnOn()` çağrılır). Genellikle bir saldırı animasyonunun aktif fazında kullanılır.
        *   `__HideWeaponTrace()`: Tüm aktif silah izlerini gizler (her `CWeaponTrace` için `TurnOff()` çağrılır). Saldırı bittiğinde veya izlerin görünmemesi gerektiğinde kullanılır.
    *   **Yok Etme (`__DestroyWeaponTrace()`):**
        *   Aktöre bağlı tüm `CWeaponTrace` nesnelerini bellekten siler (`CWeaponTrace::Delete`) ve `m_WeaponTraceVector`'ı temizler. Bu genellikle aktör yok olduğunda, silah değiştirdiğinde veya artık izlere ihtiyaç kalmadığında çağrılır.

Bu mekanizma, silahlı saldırıların daha etkileyici görünmesini sağlar ve oyuncuya hangi yönde ve ne kadar bir alanda saldırı yapıldığına dair görsel bir ipucu verir. Her bir iz nesnesinin (`CWeaponTrace`) kendi davranışı ve görünümü olabilir, bu da farklı silahlar veya yetenekler için çeşitli iz efektleri oluşturulmasına olanak tanır.

