# UserInterface Referans Kılavuzu - Bölüm 13

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonPlayerInput.cpp` (Oyuncu Girdi İşleme - Genel)](#pythonplayerinputcpp-oyuncu-girdi-işleme---genel)
*   [`PythonPlayerInputKeyboard.cpp` (Oyuncu Klavye Girdi İşleme)](#pythonplayerinputkeyboardcpp-oyuncu-klavye-girdi-işleme)
*   [`PythonPlayerInputMouse.cpp` (Oyuncu Fare Girdi İşleme)](#pythonplayerinputmousecpp-oyuncu-fare-girdi-işleme)
*   [`PythonPlayerModule.cpp` (Oyuncu Python Modülü)](#pythonplayermodulecpp-oyuncu-python-modülü)
*   [`PythonPlayerSkill.cpp` (Oyuncu Yetenek Yönetimi)](#pythonplayerskillcpp-oyuncu-yetenek-yönetimi)
*   [`PythonPrivateShop.h` & `PythonPrivateShop.cpp` (Özel Pazar Sistemi)](#pythonprivateshoph--pythonprivateshopcpp-özel-pazar-sistemi)

---

### `PythonPlayerInput.cpp` (Oyuncu Girdi İşleme - Genel)

Bu C++ kaynak dosyası, `CPythonPlayer` sınıfının oyuncu karakterinin genel girdilerini ve bu girdilere bağlı temel eylemlerini yöneten fonksiyonlarını içerir. Oyuncunun dünyayla etkileşiminin çekirdek mantığını oluşturur.

#### Dosyanın Oyundaki Kullanım Amaçları

Bu dosya, oyuncunun oyun dünyasıyla gerçekleştirdiği temel ve en sık kullanılan etkileşimlerin C++ mantığını içerir. Oyuncunun doğrudan kontrolü dışındaki bazı otomatik veya rezerve edilmiş eylemleri de yönetir:

*   **Temel Etkileşimler**: Yere tıklayarak hareket etme, karakterlere (NPC, diğer oyuncular, canavarlar) tıklayarak hedef alma veya etkileşim menülerini açma, yerdeki eşyalara tıklayarak toplama gibi oyuncunun yaptığı en temel eylemler bu dosyada işlenir.
*   **Otomatik Eylemler**: Yakındaki paraları veya tüm eşyaları toplama gibi oyuncuya kolaylık sağlayan otomatik eylemlerin mantığını barındırır.
*   **Hedef Yönetimi**: Oyuncunun bir hedef seçmesi, mevcut hedefini değiştirmesi veya temizlemesi gibi durumlar burada yönetilir. Bu, özellikle savaş ve yetenek kullanımı için kritik öneme sahiptir.
*   **Eylem Rezervasyonu**: Oyuncu bir hedefe ulaşmadan bir eylem (örneğin saldırı veya eşya toplama) başlattığında, karakter hedefe doğru hareket ederken bu eylemin "saklanması" ve hedefe ulaşıldığında gerçekleştirilmesi mekaniği bu dosyada bulunur. Bu, oyun akıcılığını artırır.
*   **Durum Kontrolleri**: Karakterin hareket edip edemeyeceği, saldırı yapıp yapamayacağı gibi temel durum kontrolleri, oyuncu girdilerine verilecek tepkileri belirlemede kullanılır.
*   **Balıkçılık Mekanikleri**: Balık tutma eyleminin başlatılması ve iptal edilmesi gibi özel oyun mekanikleriyle ilgili girdi yönetimi de burada yer alır.

Kısacası, `PythonPlayerInput.cpp`, oyuncu tarafından yapılan genel girdilerin (fare tıklamaları ve bazı genel eylem komutları) nasıl yorumlanacağını ve oyun dünyasında ne tür sonuçlar doğuracağını belirleyen çekirdek bir dosyadır.

#### Dosyanın Temel İşlevleri

*   **Eşya Toplama**:
    *   `PickCloseMoney()`: Yakındaki paraları toplar.
    *   `PickCloseItem()`: Yakındaki tek bir eşyayı toplar.
    *   `PickAllCloseItems()`: Yakındaki tüm eşyaları toplar.
    *   `__GetPickableDistance()`: Eşya toplama mesafesini belirler (at üzerindeyken daha uzak).
*   **Hedef Yönetimi**:
    *   `__IsTarget()`, `__IsSameTargetVID()`, `__GetTargetVID()`, `GetTargetVID()`: Mevcut bir hedefin olup olmadığını ve kim olduğunu kontrol eder.
    *   `__SetTargetVID()`: Programatik olarak hedef belirler.
    *   `__ClearTarget()`: Mevcut hedefi temizler ve sunucuya bilgi gönderir.
    *   `SetTarget(DWORD dwVID, BOOL bForceChange)`: Yeni bir hedef belirler. Oyuncunun hedef değiştirebilir durumda olup olmadığını kontrol eder. Hedef değiştiğinde eski hedefe `OnUntargeted`, yeni hedefe `OnTargeted` olaylarını tetikler ve sunucuya yeni hedef bilgisini gönderir.
    *   `__ChangeTargetToPickedInstance()`: Tıklanan (seçilen) karakteri hedefe alır.
    *   `__GetTargetActorPtr()`: Hedefteki karakterin `CInstanceBase` işaretçisini döndürür.
    *   `__GetSkillTargetInstancePtr()`, `__GetDeadTargetInstancePtr()`, `__GetAliveTargetInstancePtr()`: Yeteneklerin hedefini (canlı veya ölü olmasına göre) döndürür.
    *   `SelectNearTarget()` (`ENABLE_TAB_NEXT_TARGET` ile): Yakındaki bir sonraki uygun hedefi seçer (genellikle TAB tuşu ile).
*   **Karakter Etkileşimleri**:
    *   `OpenCharacterMenu(DWORD dwVictimActorID)`: Tıklanan oyuncu veya bina için karakter menüsünü (Python tarafında `SetPCTargetBoard`) açar.
    *   `__OnClickActor(CInstanceBase& rkInstMain, DWORD dwPickedActorID, bool isAuto)`: Bir karaktere tıklandığında çağrılır. Eğer rezerve bir yetenek kullanımı varsa ve hedef farklıysa veya karakter hızla hareket ediyorsa (`__CheckDashAffect`) tıklamayı rezerve eder. Yoksa, tıklanan karakter NPC ise `__SendClickActorPacket` ile sunucuya bilgi gönderir, saldırılabilir bir hedefse ve mesafede değilse `__ReserveClickActor` ile eylemi rezerve eder.
    *   `__OnPressActor(CInstanceBase& rkInstMain, DWORD dwPickedActorID, bool isAuto)`: Bir karaktere fare tuşu basılı tutulduğunda çağrılır. Hedefi değiştirir, saldırı koşullarını kontrol eder (`__CanAttack`, `__CanShot`), mesafeyi kontrol eder ve gerekirse saldırıyı başlatır (`NEW_AttackToDestInstanceDirection`). Otomatik saldırı için hedefi ayarlar.
    *   `__OnClickItem(CInstanceBase& rkInstMain, DWORD dwItemID)`: Yerdeki bir eşyaya tıklandığında (bu fonksiyon genelde boştur, asıl işlev `__OnPressItem` veya `SendClickItemPacket` ile yapılır).
    *   `__OnPressItem(CInstanceBase& rkInstMain, DWORD dwPickedItemID)`: Yerdeki bir eşyaya fare tuşu basılı tutulduğunda çağrılır. Eşyanın pozisyonunu alır, mesafeyi kontrol eder, spam engellemek için kısa bir bekleme uygular ve `SendClickItemPacket` ile toplama isteği gönderir.
    *   `__OnClickGround(CInstanceBase& rkInstMain, const TPixelPosition& c_rkPPosPickedGround)`: Yere tıklandığında çağrılır. Hareket edilebilir mesafeyi kontrol eder ve karaktere o noktaya hareket emri verir (`NEW_MoveToDestPixelPositionDirection`), ardından tıklama efektini gösterir (`__ShowPickedEffect`).
    *   `__OnPressGround(CInstanceBase& rkInstMain, const TPixelPosition& c_rkPPosPickedGround)`: Yere fare tuşu basılı tutulduğunda çağrılır. Balık tutuyorsa iptal eder, hareket mesafesini kontrol eder ve hareket emri verir.
    *   `__OnPressScreen(CInstanceBase& rkInstMain)`: Boş bir ekrana (herhangi bir nesneye tıklanmadığında) fare tuşu basılı tutulduğunda çağrılır ve karakteri fare yönüne hareket ettirir.
*   **Hareket ve Durum Kontrolleri**:
    *   `SetMovableGroundDistance(float fDistance)`: Yere tıklayarak hareket edilebilecek minimum mesafeyi ayarlar.
    *   `__IsMovableGroundDistance()`: Verilen pozisyonun hareket için uygun mesafede olup olmadığını kontrol eder.
    *   `NEW_MoveToDirection(float fDirRot)`: Karakteri belirtilen yöne (kamera açısı hesaba katılarak) hareket ettirir.
    *   `NEW_Stop()`: Karakterin hareketini durdurur ve yön tuşu durumlarını sıfırlar.
    *   `__CanMove()`, `__CanAttack()`, `__CanShot()`, `__CanChangeTarget()`: Karakterin hareket etme, saldırma, ok atma veya hedef değiştirme gibi eylemleri gerçekleştirebilir durumda olup olmadığını kontrol eder (efektler, durumlar, özel yetenekler vb. dikkate alınır).
*   **Balıkçılık**:
    *   `NEW_CancelFishing()`: Balık tutmayı iptal eder.
    *   `NEW_Fishing()`: Balık tutma eylemini başlatır veya devam ettirir/iptal eder.
*   **Saldırı**:
    *   `NEW_Attack()`: Genel saldırı fonksiyonu. Yayın varsa ve hedef varsa hedefe saldırır, yoksa yön tuşlarına göre veya direkt olarak saldırır. At üzerinde ve silahsızken saldırmayı engeller.
*   **Kamera**:
    *   `NEW_SetAutoCameraRotationSpeed(float fRotSpd)`: Otomatik kamera dönüş hızını ayarlar.
    *   `NEW_ResetCameraRotation()`: Kamera sürüklemesini bitirir ve imleci normale döndürür.
*   **Yardımcı Fonksiyonlar**:
    *   `GetDegreeFromPosition()`: Ekran koordinatlarından açı hesaplar.
    *   `NEW_GetMouseDirRotation()`, `NEW_GetMultiKeyDirRotation()`: Fare veya klavye girdilerine göre hareket yönünü hesaplar.
*   **Rezerve Edilmiş Eylemler (Action Reservation)**:
    *   Oyuncunun bir hedefe tıklaması ama hedefin anında ulaşılabilecek mesafede olmaması durumunda, karakter hedefe doğru hareket ederken yapılacak eylemi (tıklama, yetenek kullanma) "rezerve" eder.
    *   `__ClearReservedAction()`: Rezerve edilmiş eylemi temizler.
    *   `__ReserveClickItem(DWORD dwItemID)`: Eşya tıklama eylemini rezerve eder.
    *   `__ReserveClickActor(DWORD dwActorID)`: Karaktere tıklama eylemini rezerve eder.
    *   `__ReserveClickGround(const TPixelPosition& c_rkPPosPickedGround)`: Yere tıklama eylemini rezerve eder.
    *   `__IsReservedUseSkill(DWORD dwSkillSlotIndex)`: Belirli bir yeteneğin rezerve edilip edilmediğini kontrol eder.
    *   `__ReserveUseSkill(DWORD dwActorID, DWORD dwSkillSlotIndex, DWORD dwRange)`: Yetenek kullanma eylemini rezerve eder.
    *   `__ReserveProcess_ClickActor()`: Rezerve edilmiş karakter tıklama eylemini işler (hedefe yaklaşma, saldırı veya NPC etkileşimi).
*   **Otomatik Saldırı**:
    *   `__ClearAutoAttackTargetActorID()`: Otomatik saldırı hedefini temizler.
    *   `__SetAutoAttackTargetActorID(DWORD dwVID)`: Otomatik saldırı hedefini ayarlar.

Bu dosya, oyuncunun oyun dünyasıyla olan temel ve sık kullanılan etkileşimlerinin mantığını yöneterek akıcı bir oyun deneyimi sunar.

---

### `PythonPlayerInputKeyboard.cpp` (Oyuncu Klavye Girdi İşleme)

Bu C++ kaynak dosyası, `CPythonPlayer` sınıfının klavye girdilerini işleyen fonksiyonlarını içerir. Karakterin klavye aracılığıyla yönlendirilmesi, saldırması ve çeşitli kısayol tuşlarının kullanımından sorumludur.

#### Dosyanın Oyundaki Kullanım Amaçları

Bu dosya, oyuncunun klavye aracılığıyla oyunla etkileşim kurmasının temelini oluşturur. Oyun deneyimini kişiselleştirme ve hızlandırma açısından kritik rol oynar:

*   **Karakter Hareketi**: Yön tuşları (W, A, S, D veya ok tuşları) kullanılarak karakterin oyun dünyasında hareket ettirilmesi bu dosyadaki fonksiyonlarla sağlanır.
*   **Temel Saldırı**: Atanmış bir klavye tuşu (genellikle Boşluk tuşu) ile temel saldırı eyleminin gerçekleştirilmesi burada yönetilir.
*   **Kısayol Tuşları (Tuş Atama Sistemi)**: Oyunun en önemli kullanım kolaylıklarından biri olan tuş atama sisteminin kalbidir. Oyuncular, bu sistem sayesinde:
    *   Envanter, statü penceresi, yetenekler, görevler gibi çeşitli kullanıcı arayüzü pencerelerini tek bir tuşla açabilirler.
    *   Hızlı erişim slotlarına yerleştirdikleri iksirleri, becerileri veya diğer eşyaları klavye kısayollarıyla anında kullanabilirler.
    *   Duygu ifadelerini (dans etme, selam verme vb.) kolayca tetikleyebilirler.
    *   Kamera kontrolü, sonraki hedefi seçme, ata binme gibi sık kullanılan eylemleri kendilerine özel tuşlara atayabilirler.
*   **Sistem Komutları**: Sol Alt tuşu gibi değiştirici tuşlarla birlikte kullanılan kısayollar (örn: hızlı erişim sayfasını değiştirme, yerdeki isimleri gösterme/gizleme) da burada işlenir.

Bu dosya, klavyeyi oyun içinde birincil kontrol aracı olarak kullanan oyuncular için akıcı ve verimli bir oyun deneyimi sunar. Özellikle Tuş Atama Sistemi sayesinde, her oyuncu kendi oyun stiline uygun bir kontrol şeması oluşturabilir.

#### Dosyanın Temel İşlevleri

*   **Hareket Tuşları**:
    *   `NEW_SetSingleDIKKeyState(int eDIKKey, bool isPress)`: Tek bir yön tuşuna (`DIK_UP`, `DIK_DOWN`, `DIK_LEFT`, `DIK_RIGHT`) basılma veya bırakılma durumunu işler.
    *   `NEW_SetSingleDirKeyState(int eDirKey, bool isPress)`: Yön tuşu durumlarını (`m_isUp`, `m_isDown`, `m_isLeft`, `m_isRight`) günceller ve `NEW_SetMultiDirKeyState` fonksiyonunu çağırır.
    *   `NEW_SetMultiDirKeyState(bool isLeft, bool isRight, bool isUp, bool isDown)`: Birden fazla yön tuşunun kombinasyonuna göre karakterin hareket etmesini veya durmasını sağlar. `__CanMove()` kontrolü ile hareket edebilirliği denetler ve `NEW_MoveToDirection` ile hareketi başlatır.
    *   `GetDegreeFromDirection(int iUD, int iLR)`: Basılan yön tuşlarına göre (örn: yukarı ve sol) hareket açısını derece cinsinden hesaplar.
*   **Saldırı Tuşu**:
    *   `SetAttackKeyState(bool isPress)`: Atanmış saldırı tuşuna basılma durumunu (`m_isAtkKey`) ayarlar. Eğer oyuncu balık tutma modundaysa ve tuşa basılırsa `NEW_Fishing()` çağrılır.
*   **Tuş Atama Sistemi (`ENABLE_KEYCHANGE_SYSTEM` tanımlıysa)**:
    *   `OnKeyDown(int iKey)`: Bir tuşa basıldığında çağrılır.
        *   Eğer tuş ayar penceresi (`m_isOpenKeySettingWindow`) açıksa işlem yapmaz.
        *   `DIK_LMENU` (Sol Alt) tuşuna basıldığında hızlı erişim sayfasını değiştirir ve isimleri gösterme fonksiyonunu (`ShowName`) çağırır.
        *   Ctrl, Alt, Shift gibi değiştirici tuşlarla birlikte basılan tuşları algılar ve `m_keySettingMap` içinde tanımlı eylemi gerçekleştirir.
        *   Tanımlı eylemler arasında:
            *   Hareket tuşları (`KEY_MOVE_UP_1`, `KEY_MOVE_DOWN_1` vb.)
            *   Saldırı (`KEY_ATTACK`)
            *   Otomatik koşu (`KEY_AUTO_RUN`)
            *   Ata binme/besleme komutları (`KEY_RIDEMYHORS`, `KEY_FEEDMYHORS`, `KEY_RIDEHORS`, `KEY_BYEMYHORS`)
            *   Kamera kontrolleri (döndürme, yakınlaştırma, eğme)
            *   Eşya toplama (`KEY_ROOTING_1`, `KEY_ROOTING_2`)
            *   Duygu ifadeleri (`KEY_EMOTION1` - `KEY_EMOTION9`)
            *   Sonraki hedefi seçme (`KEY_NEXT_TARGET`) (`ENABLE_TAB_NEXT_TARGET` ile)
            *   Hızlı erişim slotlarını kullanma (`KEY_SLOT_1` - `KEY_SLOT_8`)
            *   Hızlı erişim sayfasını değiştirme (`KEY_SLOT_CHANGE_1` - `KEY_SLOT_CHANGE_4`)
            *   Çeşitli UI pencerelerini açma (Statü, Yetenek, Görev, Envanter, Simya, Minimap, Sohbet Kaydı, Lonca, Messenger, Yardım, Aksiyonlar/Duygular, Pet, Özel Envanter)
            *   Minimap büyütme/küçültme
            *   Ekran görüntüsü alma (`KEY_SCREENSHOT`)
            *   İsimleri gösterme/gizleme (`KEY_SHOW_NAME`)
    *   `OnKeyUp(int iKey)`: Bir tuş bırakıldığında çağrılır.
        *   `DIK_LMENU` (Sol Alt) bırakıldığında hızlı erişim sayfasını geri alır ve isimleri gizleme fonksiyonunu (`HideName`) çağırır.
        *   Basılı tutulan hareket, saldırı veya kamera kontrol tuşları bırakıldığında ilgili eylemi durdurur (örneğin, hareketi durdurur, kamera dönüşünü durdurur).
    *   `SetEmoticon(BYTE byIndex)`: Belirtilen duygu ifadesini oynatır ve sunucuya bilgisini gönderir.

Bu dosya, oyuncunun klavye kullanarak karakterini kontrol etmesini ve oyun içi fonksiyonlara hızla erişmesini sağlar, özellikle tuş atama sistemi sayesinde kişiselleştirilmiş bir kontrol şeması sunar.

---

### `PythonPlayerInputMouse.cpp` (Oyuncu Fare Girdi İşleme)

Bu C++ kaynak dosyası, `CPythonPlayer` sınıfının fare girdilerini işleyen fonksiyonlarını içerir. Oyuncunun fare tıklamaları ve hareketleriyle karakterini yönlendirmesi, dünyayla etkileşim kurması ve kamerayı kontrol etmesinden sorumludur.

#### Dosyanın Oyundaki Kullanım Amaçları

Bu dosya, oyuncunun fareyi kullanarak oyun dünyasıyla etkileşim kurmasının ve oyun kamerasını yönetmesinin tüm mantığını içerir. Sezgisel ve dinamik bir oyun deneyimi için kritik öneme sahiptir:

*   **Fare ile Karakter Hareketi**: Oyuncuların fare sol tuşuna basılı tutarak karakterlerini istedikleri yöne doğru hareket ettirmeleri bu dosya sayesinde mümkün olur. Yere tıklayarak belirli bir noktaya gitme eylemi de burada yönetilir.
*   **Fare ile Kamera Kontrolü**: Genellikle fare orta tuşuna basılı tutarak veya belirli bir tuş kombinasyonuyla kamerayı döndürme, eğme ve oyun dünyasına farklı açılardan bakma imkanı sunar. Bu, çevreyi keşfetmek ve taktiksel avantaj sağlamak için önemlidir.
*   **"Akıllı" Tıklama Sistemi**: Oyunun en çok kullanılan fare etkileşimi olan sol veya sağ tıklamaların (oyuncunun ayarlarına göre) ne anlama geldiğini bağlama göre yorumlar:
    *   Bir **canavara veya düşman oyuncuya** tıklamak saldırı başlatır veya hedef almayı sağlar.
    *   Bir **NPC\'ye** tıklamak konuşma penceresini açar veya etkileşim seçenekleri sunar.
    *   **Yerdeki bir eşyaya** tıklamak onu toplamaya çalışır.
    *   **Boş bir zemine** tıklamak karakteri o noktaya hareket ettirir.
    *   **Özel Pazar tezgahlarına** tıklamak pazar içeriğini görüntüler.
*   **Fare Tuşu Fonksiyon Atamaları**: Oyuncuların fare tuşlarına farklı işlevler (hareket, saldırı, yetenek kullanma, kamera kontrolü) atamasına olanak tanır, böylece kişisel oyun tarzlarına uygun bir kontrol şeması oluşturabilirler. Örneğin, bazı oyuncular sağ fare tuşunu yetenek kullanmak için ayarlayabilir.
*   **Rezerve Edilmiş Eylemlerin Yönetimi**: Bir hedefe (eşya, karakter) tıklanıp hemen ulaşılamadığında, karakter hedefe doğru hareket ederken yapılacak eylemin (toplama, saldırma) fare girdileriyle koordine edilmesi ve hedefe varıldığında otomatik olarak gerçekleştirilmesi bu dosya tarafından yönetilir. Bu, oyunun akıcılığını artırır.

Kısacası, `PythonPlayerInputMouse.cpp`, fareyi oyun içinde ana etkileşim aracı olarak kullanan oyuncular için zengin ve bağlama duyarlı bir kontrol deneyimi sağlar.

#### Dosyanın Temel İşlevleri

*   **Fare ile Hareket**:
    *   `NEW_SetMouseMoveState(int eMBS)`: Fare tuşuna basılı tutulduğunda (`MBS_PRESS`) karakteri fare imlecinin gösterdiği yöne doğru hareket ettirir (`NEW_MoveToMouseScreenDirection`). Fare tuşu bırakıldığında (`MBS_CLICK`) karakteri durdurur (`NEW_Stop`).
    *   `NEW_MoveToMouseScreenDirection()`: Fare imlecinin ekran üzerindeki pozisyonunu alarak karakterin o yöne doğru hareket etmesi için gerekli açıyı hesaplar ve `NEW_MoveToDirection` fonksiyonunu çağırır.
*   **Fare ile Kamera Kontrolü**:
    *   `NEW_SetMouseCameraState(int eMBS)`: Genellikle fare orta tuşu veya belirli bir tuş kombinasyonuyla kamera moduna geçildiğinde çağrılır.
        *   `MBS_PRESS`: Fare pozisyonunu alarak kamera sürüklemesini (`CCamera::BeginDrag`) başlatır. İmleci kamera döndürme ikonuna değiştirir.
        *   `MBS_CLICK`: Kamera sürüklemesini (`CCamera::EndDrag`) bitirir. Eğer kamera sürüklenmediyse (sadece tıklandıysa) ve bir karaktere tıklanmışsa, o karakterin menüsünü (`OpenCharacterMenu`) açar. İmleci normale döndürür.
    *   `SetQuickCameraMode(bool isEnable)`: Hızlı kamera modunu etkinleştirir veya devre dışı bırakır (devre dışı bırakıldığında `NEW_SetMouseCameraState(MBS_CLICK)` çağrılır).
    *   `NEW_SetMouseMiddleButtonState(int eMBState)`: Fare orta tuşu durumunu doğrudan `NEW_SetMouseCameraState` fonksiyonuna yönlendirir.
*   **"Akıllı" Fare Tıklamaları (Smart Click)**:
    *   `NEW_SetMouseSmartState(int eMBS, bool isAuto)`: Sol veya sağ fare tıklamalarını (duruma göre) "akıllı" bir şekilde işler. Karakterin özel durumlarını (pazar açık, duygu ifadesi oynatılıyor, sersemlemiş) kontrol eder.
        *   `MBS_PRESS`: `__OnPressSmart` fonksiyonunu çağırır.
        *   `MBS_CLICK`: `__OnClickSmart` fonksiyonunu çağırır.
    *   `__OnPressSmart(CInstanceBase& rkInstMain, bool isAuto)`: Fare tuşuna basılı tutulduğunda, tıklanan hedefin ne olduğuna (eşya, karakter, özel pazar (`ENABLE_PREMIUM_PRIVATE_SHOP` ile), zemin veya boş ekran) karar verir ve ilgili `__OnPressItem`, `__OnPressActor`, `__OnPressPrivateShop`, `__OnPressGround` veya `__OnPressScreen` fonksiyonunu çağırır.
    *   `__OnClickSmart(CInstanceBase& rkInstMain, bool isAuto)`: Fare tuşuna tıklandığında (basılıp bırakıldığında), tıklanan hedefe göre ilgili `__OnClickItem`, `__OnClickActor`, `__OnClickPrivateShop` veya `__OnClickGround` fonksiyonunu çağırır. Eğer hiçbir şeye tıklanmadıysa karakteri durdurur.
*   **Fare Tuş Fonksiyon Atamaları**:
    *   `NEW_SetMouseFunc(int eMBT, int eMBF)`: Belirli bir fare tuşuna (`eMBT` - örn: `MBT_LEFT`, `MBT_RIGHT`) belirli bir fonksiyon (`eMBF` - örn: `MBF_MOVE`, `MBF_SMART`, `MBF_CAMERA`, `MBF_ATTACK`, `MBF_SKILL`) atar.
    *   `NEW_GetMouseFunc(int eMBT)`: Belirli bir fare tuşuna atanmış fonksiyonu döndürür.
    *   `NEW_SetMouseState(int eMBT, int eMBS)`: Belirli bir fare tuşuna (`eMBT`) basılma/bırakılma (`eMBS`) durumuna göre atanmış olan fare fonksiyonunu (`eMBF`) çalıştırır.
        *   `MBF_MOVE`: Hareket (`NEW_SetMouseMoveState`).
        *   `MBF_SMART`: Akıllı tıklama (`NEW_SetMouseSmartState`), Ctrl basılıysa `NEW_Attack()`.
        *   `MBF_CAMERA`: Kamera kontrolü (`NEW_SetMouseCameraState`).
        *   `MBF_AUTO`: Otomatik akıllı tıklama.
        *   `MBF_ATTACK`: Direkt saldırı (`NEW_Attack()`).
        *   `MBF_SKILL`: Ctrl basılıysa kamera, değilse mevcut yeteneği kullanma (`__UseCurrentSkill`).
*   **Efektler**:
    *   `__ShowPickedEffect(const TPixelPosition& c_rkPPosPickedGround)`: Yere tıklandığında bir efekt (`EFFECT_PICK`) gösterir.
*   **Rezerve Edilmiş Eylemlerin Güncellenmesi**:
    *   `NEW_RefreshMouseWalkingDirection()`: Oyunun her frame'inde çağrılan bu fonksiyon, daha önce fare tıklamasıyla rezerve edilmiş bir eylem varsa (örneğin, bir eşyaya tıklanmış ama karakter henüz mesafede değilse) bu eylemi kontrol eder.
        *   `MODE_CLICK_PRIVATE_SHOP`: Özel pazara yaklaşır ve yeterince yakınsa pazar açma isteği gönderir.
        *   `MODE_CLICK_ITEM`: Eşyaya yaklaşır ve yeterince yakınsa toplama isteği gönderir.
        *   `MODE_CLICK_ACTOR`: `__ReserveProcess_ClickActor` çağrılarak karaktere yaklaşma/etkileşim/saldırı süreci devam ettirilir.
        *   `MODE_CLICK_POSITION`: Yere tıklanmış ve bir gecikme süresi dolmuşsa, o pozisyona hareket eder.
        *   `MODE_USE_SKILL`: Hedefe yaklaşır ve yetenek menziline girince hedefe alır ve yeteneği kullanır.
        *   Ayrıca, fare tuşu basılı tutularak hareket ediliyorsa (`m_isSmtMov`, `m_isDirMov`) veya klavye yön tuşları basılıysa (`m_isDirKey`) ya da saldırı tuşu basılıysa (`m_isAtkKey`) ilgili eylemleri günceller/devam ettirir.
*   `__IsRightButtonSkillMode()`: Sağ fare tuşuna yetenek kullanma fonksiyonunun atanıp atanmadığını kontrol eder.

Bu dosya, oyuncunun fareyi kullanarak oyun dünyasıyla sezgisel ve çeşitli şekillerde etkileşim kurmasını sağlar, farklı fare tuşu davranışlarını yöneterek esnek bir kontrol imkanı sunar.

---

### `PythonPlayerModule.cpp` (Oyuncu Python Modülü)

Bu C++ kaynak dosyası, istemcideki ana oyuncu karakterini yöneten `CPythonPlayer` sınıfının C++ fonksiyonlarını Python betiklerinin kullanımına sunmak amacıyla `player` adında bir Python modülü tanımlar. Bu modül, Python'un oyuncu verilerine erişmesini, oyuncu eylemlerini tetiklemesini ve oyunun çeşitli temel mekaniklerini kontrol etmesini sağlar.

#### Dosyanın Oyundaki Kullanım Amaçları

Bu modül, oyunun kullanıcı arayüzünü (UI) ve çeşitli oyun içi sistemleri çalıştıran Python betikleri için temel bir araçtır. Oyuncunun neredeyse tüm yönlerini Python seviyesinden kontrol etmeye ve sorgulamaya olanak tanır:

*   **Kullanıcı Arayüzü Entegrasyonu**: Oyuncunun canı, manası, seviyesi, envanterindeki eşyalar, yetenekleri gibi bilgilerin UI üzerinde gösterilmesi bu modül aracılığıyla sağlanır. Örneğin, envanter penceresi (`uiInventory.py`) oyuncunun eşyalarını listelemek için `player.GetItemIndex()`, `player.GetItemCount()` gibi fonksiyonları kullanır.
*   **Oyuncu Eylemlerinin Tetiklenmesi**: Python betikleri, kullanıcı etkileşimlerine (örn: bir butona tıklama) veya oyun içi olaylara yanıt olarak oyuncunun saldırmasını (`player.NEW_Attack()`), hareket etmesini, bir yetenek kullanmasını (`player.ClickSkillSlot()`), bir eşyayı kullanmasını veya NPC ile konuşmasını (`player.OpenCharacterMenu()`) bu modül üzerinden tetikler.
*   **Oyun Mekaniklerine Erişim**: PK modu değiştirme, partiye katılma/ayrılma, ticaret başlatma gibi temel oyun mekanikleri Python arayüzleri tarafından bu modül kullanılarak yönetilir.
*   **Kısayol ve Otomasyon**: Oyuncunun klavye kısayolları (örn: hızlı erişim slotları `player.RequestUseLocalQuickSlot()`) veya bazı otomatik eylemler (örn: otomatik pot kullanma betikleri) `player` modülünün fonksiyonlarını çağırarak çalışır.
*   **Sistemlerin Python'a Açılması**: Oyuna sonradan eklenen veya karmaşık yapıdaki sistemler (Simya, Kostüm, Evcil Hayvan vb.) genellikle C++ tarafında yönetilir ve bu modül aracılığıyla Python betiklerinin bu sistemlerle etkileşime girmesi için gerekli arayüzler sağlanır. Örneğin, bir simya penceresi (`uiDragonSoul.py`) simya taşlarını yönetmek için `player.SendDragonSoulRefine()` gibi fonksiyonları çağırır.
*   **Durum ve Veri Sorgulama**: Python betikleri, oyun mantığını yürütmek veya UI'ı dinamik olarak güncellemek için oyuncunun mevcut durumunu (örn: `player.IsInSafeArea()`, `player.IsMountingHorse()`), hedefini (`player.GetTargetVID()`) veya belirli bir eşyanın özelliklerini (`player.GetItemAttribute()`) sürekli olarak bu modül üzerinden sorgular.

Kısacası, `player` modülü, C++ çekirdeğinde yönetilen oyuncu karakterinin tüm karmaşıklığını soyutlayarak Python betiklerinin oyuncuyla ilgili işlemleri kolayca ve etkili bir şekilde gerçekleştirmesine olanak tanıyan merkezi bir bileşendir.

#### Dosyanın Genel Amaçları ve Yapısı

*   **Python Modülü (`player`) Tanımlama**: `initPlayer()` fonksiyonu aracılığıyla `player` adında bir Python modülü oluşturulur ve `CPythonPlayer` sınıfının çeşitli işlevlerini Python'dan çağrılabilir hale getiren sarmalayıcı (wrapper) C fonksiyonları bu modüle kaydedilir.
*   **CPythonPlayer İşlevlerini Python'a Aktarma**: Oyuncunun temel özelliklerinden (isim, sınıf, seviye, statüler, envanter, para, yetenekler) karmaşık sistemlere (PK modu, parti, hızlı erişim, simya, kostüm sistemleri, evcil hayvanlar vb.) kadar geniş bir yelpazedeki işlevsellik Python'a açılır.
*   **Veri ve Durum Erişimi**: Python betikleri, oyuncunun anlık konumunu, hedefini, HP/SP gibi durumlarını, eşya bilgilerini (vnum, sayı, soketler, efsunlar), yetenek bilgilerini (seviye, bekleme süresi) ve daha birçok veriyi bu modül aracılığıyla sorgulayabilir.
*   **Eylem Tetikleme**: Python, oyuncunun hareket etmesini, saldırmasını, eşya toplamasını/kullanmasını, yetenek kullanmasını, duygu ifadesi yapmasını, çeşitli UI pencerelerini açıp kapatmasını ve ağ mesajları göndermesini sağlayabilir.
*   **Sabitlerin Tanımlanması**: Oyun içinde kullanılan çok sayıda sabit değer (örneğin, statü ID'leri, envanter türleri, eşya bayrakları, yetenek dereceleri, PK modları, duygu ID'leri, çeşitli sistemlere özgü sabitler) `PyModule_AddIntConstant` kullanılarak `player` modülüne eklenir. Bu, Python tarafında bu değerlere sayısal karşılıkları yerine anlamlı isimlerle erişilmesini sağlar.
*   **Sistem Entegrasyonu**: `#if defined(ENABLE_...)` önişlemci direktifleri kullanılarak, derleme zamanında aktif edilen çeşitli oyun sistemlerine (örneğin, Kemer Envanteri, Simya, Kostüm, Aura, Evcil Hayvan, Eşya Görünümü Değiştirme, Tuş Atama vb.) ait Python arayüzleri modüle dahil edilir veya çıkarılır.

#### Önemli Python'a Aktarılan Fonksiyon Grupları ve Kullanım Amaçları

*   **Temel Oyuncu Bilgileri ve Durumları**:
    *   `playerGetName()`, `playerGetRace()`, `playerGetJob()`, `playerGetLevel()`, `playerGetConquerorLevel()`: Oyuncunun temel kimlik ve seviye bilgilerini döndürür.
    *   `playerGetStatus(POINT_HP)`, `playerGetEXP()`, `playerGetElk()`: Oyuncunun HP, EXP, Yang gibi anlık statülerini alır.
    *   `playerSetStatus(POINT_HP, value)`: Oyuncu statülerini (nadiren doğrudan, genellikle sunucu tarafından güncellenir) ayarlar.
    *   `playerGetPlayTime()`: Oyuncunun oynama süresini alır.
    *   `playerGetMainCharacterIndex()`, `playerGetMainCharacterPosition()`: Ana karakterin ID'sini ve pozisyonunu alır.
    *   `playerIsInSafeArea()`, `playerIsMountingHorse()`: Oyuncunun güvenli bölgede olup olmadığını veya ata binip binmediğini kontrol eder.
*   **Hedefleme ve Etkileşim**:
    *   `playerSetTarget(vid)`, `playerClearTarget()`: Hedef belirler veya temizler.
    *   `playerGetTargetVID()`: Mevcut hedefin ID'sini alır.
    *   `playerCanAttackInstance(vid)`, `playerIsPVPInstance(vid)`, `playerIsSameEmpire(vid)`: Hedefle ilgili saldırı ve ilişki durumlarını kontrol eder.
    *   `playerGetCharacterDistance(vid)`: Hedefle olan mesafeyi ölçer.
    *   `playerOpenCharacterMenu(vid)`: Hedefin karakter menüsünü açar.
    *   `playerPickCloseItem()`, `playerPickAllCloseItems()`: Yakındaki eşyaları toplar.
    *   `playerSendClickItemPacket(item_vid)`: Yerdeki bir eşyaya tıklama paketini gönderir.
*   **Hareket ve Kontrol**:
    *   `playerSetAttackKeyState(isPressed)`, `playerSetSingleDIKKeyState(key, isPressed)`: Saldırı ve hareket tuşlarının durumunu ayarlar.
    *   `playerEndKeyWalkingImmediately()`: Klavye ile hareketi anında durdurur.
    *   `playerComboAttack()`: Kombo saldırısı yapar.
    *   `playerSetMouseState(button, state)`, `playerSetMouseFunc(button, function)`: Fare tuşlarının davranışlarını ve atanmış işlevlerini yönetir.
*   **Eşya Yönetimi (Envanter, Ekipman)**:
    *   `playerGetItemIndex(window_type, slot_index)`, `playerGetItemCount(window_type, slot_index)`: Belirtilen slottaki eşyanın VNUM'unu ve sayısını alır.
    *   `playerGetItemFlags()`, `playerGetItemMetinSocket()`, `playerGetItemAttribute()`, `playerGetItemApplyRandom()`, `playerGetItemRefineElement()`, `playerGetItemSetValue()`: Eşyanın bayraklarını, taşlarını, efsunlarını, rastgele efsunlarını, elementini ve set değerini alır.
    *   `playerGetItemLink(window_type, slot_index)`: Eşyanın sohbet penceresine linklenebilir metnini oluşturur.
    *   `playerMoveItem(src_pos, dst_pos)`: Eşyayı bir yerden bir yere taşır (sunucu onayı gerekir).
    *   `playerIsEquipmentSlot(slot_index)`, `playerIsCostumeSlot(slot_index)`, `playerIsBeltInventorySlot(slot_index)`: Slotun türünü kontrol eder.
    *   `playerCanRefine()`, `playerCanAttachMetin()`, `playerCanDetach()`: Eşya geliştirme, taş ekleme/çıkarma işlemlerinin yapılabilirliğini kontrol eder.
*   **Yetenek Yönetimi**:
    *   `playerSetSkill(slot_index, skill_vnum)`, `playerGetSkillIndex(slot_index)`: Yetenek slotlarına yetenek atar veya bilgisini alır.
    *   `playerGetSkillGrade(slot_index)`, `playerGetSkillLevel(slot_index)`: Yeteneğin derecesini ve seviyesini alır.
    *   `playerIsSkillCoolTime(slot_index)`, `playerGetSkillCoolTime(slot_index)`: Yetenek bekleme sürelerini kontrol eder.
    *   `playerClickSkillSlot(skill_slot_index)`: Belirli bir yetenek slotunu kullanır.
*   **Hızlı Erişim Slotları**:
    *   `playerGetQuickPage()`, `playerSetQuickPage(page_index)`: Hızlı erişim sayfasını alır veya ayarlar.
    *   `playerGetLocalQuickSlot(slot_index)`, `playerGetGlobalQuickSlot(slot_index)`: Yerel ve global hızlı erişim slotlarındaki bilgileri alır.
    *   `playerRequestAddLocalQuickSlot()`, `playerRequestDeleteGlobalQuickSlot()`: Hızlı erişim slotlarına ekleme/çıkarma/taşıma istekleri gönderir.
    *   `playerRequestUseLocalQuickSlot(slot_index)`: Yerel hızlı erişim slotunu kullanır.
*   **Parti Sistemi**:
    *   `playerIsPartyMember(vid)`, `playerIsPartyLeader(vid)`: Bir karakterin parti üyesi veya lideri olup olmadığını kontrol eder.
    *   `playerGetPartyMemberHPPercentage(pid)`, `playerGetPartyMemberAffects(pid)`: Parti üyesinin HP yüzdesini ve aktif efektlerini alır.
    *   `playerExitParty()`: Partiden ayrılır.
*   **PK Modu**:
    *   `playerGetPKMode()`: Mevcut PK modunu alır.
*   **Duygu İfadeleri**:
    *   `playerActEmotion(emotion_index)`: Belirtilen duygu ifadesini oynatır.
    *   `playerRegisterEmotionIcon(index, filename)`, `playerGetEmotionIconImage(index)`: Duygu ifadeleri için ikonları kaydeder ve alır.
*   **Çeşitli Sistemler için Arayüzler**:
    *   **Simya (Dragon Soul)**: `playerSendDragonSoulRefine()`
    *   **Kemer Envanteri (`ENABLE_NEW_EQUIPMENT_SYSTEM`)**: `playerIsEquippingBelt()`, `playerIsAvailableBeltInventoryCell()`
    *   **Kostüm ve Acce (`ENABLE_COSTUME_SYSTEM`, `ENABLE_ACCE_COSTUME_SYSTEM`)**: `playerIsCostumeSlot()`, `playerSetAcceRefineWindowOpen()`
    *   **Aura Sistemi (`ENABLE_AURA_COSTUME_SYSTEM`)**: `playerGetAuraItemID()`, `playerGetAuraItemAttribute()`, `playerIsAuraRefineWindowOpen()`
    *   **Evcil Hayvan Sistemi (`ENABLE_GROWTH_PET_SYSTEM`)**: `playerGetPetItem()`, `playerGetActivePetItemId()`, `playerGetPetSkill()`, `playerSetOpenPetHatchingWindow()`
    *   **Mühürleme/Mühür Kaldırma (`ENABLE_SOUL_BIND_SYSTEM`)**: `playerCanSealItem()`, `playerGetItemSealDate()`, `playerGetItemUnSealLeftTime()`
    *   **Eşya Görünümü Değiştirme (`ENABLE_CHANGE_LOOK_SYSTEM`)**: `playerGetChangeLookVnum()`, `playerSetChangeLookWindowOpen()`, `playerCanChangeLookClearItem()`
    *   **Tuş Atama Sistemi (`ENABLE_KEYCHANGE_SYSTEM`)**: `playerOpenKeyChangeWindow()`, `playerKeySetting()`, `playerOnKeyDown()`
    *   **Güvenli Kasa, Nesne Market**: `playerIsOpenSafeBox()`, `playerSetOpenMall()`
    *   **Diğerleri**: Küp Yenileme, Set Eşya Efektleri, Balıkçılık Yenileme, Kostüm Efsun Aktarımı gibi sistemler için çeşitli fonksiyonlar.
*   **Sabit Değerler**:
    *   `POINT_...` (statü türleri), `SLOT_TYPE_...` (slot türleri), `INVENTORY`, `EQUIPMENT`, `PK_MODE_...`, `EMOTION_...` gibi çok sayıda sabit, Python tarafında kod okunabilirliğini artırmak için modüle eklenir.

#### Linter Hataları ve Olası Etkileri

Dosyada belirtilen linter hataları (özellikle `PyDict_Next` argüman türü ve `CPythonSkillPet` sınıfının eksik tanımı), derleme sırasında sorunlara yol açabilir. `PyDict_Next` ile ilgili hata, Python C API'sinin farklı versiyonları arasındaki `Py_ssize_t` tanımından kaynaklanıyor olabilir. `CPythonSkillPet` ile ilgili hata ise genellikle ilgili başlık dosyasının (`CPythonSkillPet.h`) `PythonPlayerModule.cpp` içine dahil edilmemesinden veya eksik bir tanım içermesinden kaynaklanır. Bu hatalar düzeltilmezse, ilgili fonksiyonlar (`playerSendDragonSoulRefine`, `playerGetPetSkillByIndex`) Python'a doğru şekilde aktarılamayabilir veya çalışma zamanı hatalarına neden olabilir.

Bu modül, `CPythonPlayer` sınıfının yeteneklerini Python dünyasına taşıyarak oyunun UI ve betik katmanının oyuncu karakteriyle derinlemesine ve esnek bir şekilde etkileşim kurmasına olanak tanır. 

### `PythonPlayerSkill.cpp` (Oyuncu Yetenek Yönetimi)

**Dosyanın Amacı ve İşlevselliği**

`PythonPlayerSkill.cpp`, oyuncunun yetenek (skill) kullanımıyla ilgili istemci tarafı mantığını yönetir. Bu dosya, oyuncunun yetenekleri kullanma, yeteneklerin bekleme sürelerini (cooldown) yönetme, yetenek kullanım koşullarını kontrol etme (örneğin, yeterli mana, doğru silah türü, hedef gereksinimi vb.) ve yeteneklerin etkilerini (affect) uygulama gibi temel işlevleri içerir. Aynı zamanda Python ile etkileşim kurarak oyun arayüzünde yetenek bilgilerinin gösterilmesini ve güncellenmesini sağlar.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **Yetenek Kullanımı (`__UseSkill`)**:
    *   Oyuncunun yetenek kullanma isteği bu fonksiyon üzerinden işlenir.
    *   İlk olarak, oyuncunun yetenek kullanıp kullanamayacağı kontrol edilir (`__CanUseSkill`). Bu kontrol, oyuncunun özel bir durumda olup olmadığını (örneğin, özel dükkan açık mı, gözlemci modunda mı) inceler.
    *   Belirtilen yetenek slotunun geçerliliği kontrol edilir.
    *   Balıkçılık veya kombo gibi özel yetenekler için ayrı bir kontrol (`__CheckSpecialSkill`) yapılır.
    *   `CPythonSkill` singleton'ından yetenek verileri alınır.
    *   Eğer yetenek bir "toggle" (aç/kapa) yeteneğiyse ve aktifse, kapatma paketi gönderilir.
    *   Yetenek kullanım koşulları (`__CheckSkillUsable`) detaylı bir şekilde kontrol edilir:
        *   At üzerinde olup olmama ve yeteneğin at yeteneği olup olmaması.
        *   Güvenli bölgede olup olmama (saldırı yetenekleri için).
        *   Yetenek için gerekli eşyaların (örneğin, boş şişe, zehir şişesi) envanterde olup olmaması.
        *   Balık tutma modunda olup olmama.
        *   Yeterli seviyede olup olmama.
        *   Doğru silah türünün kuşanılmış olup olmaması.
        *   Yeterli ok olup olmaması (yay gerektiren yetenekler için).
        *   Bekleme süresinin (cooldown) dolmuş olması.
        *   Yeterli HP veya SP (mana) olup olmaması.
    *   Ana karakter (`CInstanceBase`) ve hedef karakter (`CInstanceBase`) örnekleri alınır.
    *   Yetenek bir hedef gerektiriyorsa (`IsNeedTarget`), hedef seçimi ve doğrulaması yapılır:
        *   Ceset gerektiren (`IsNeedCorpse`) veya canlı hedef gerektiren yetenekler için uygun hedef aranır.
        *   Hedef yoksa, fare ile seçilen hedef atanmaya çalışılır.
        *   Hedefin dost/düşman durumu, güvenli bölgede olup olmadığı ve menzil kontrolü yapılır.
        *   Gerekirse yetenek kullanımı rezerve edilir (`__ReserveUseSkill`) veya hedefe dönülür.
    *   Alan etkili yetenekler (`IsFanRange`, `IsCircleRange`) için menzil içindeki hedefler belirlenir ve sunucuya ek hedef bilgileri gönderilir.
    *   Yetenek animasyonu (`NEW_UseSkill`) oynatılır.
    *   Sunucuya yetenek kullanım paketi gönderilir (`__SendUseSkill`).

*   **Yetenek Etkileri (`SetAffect`, `ResetAffect`)**:
    *   Oyuncuya bir yetenek etkisi (buff/debuff) geldiğinde veya kaldırıldığında çağrılır.
    *   Python arayüz fonksiyonları (`m_ppyGameWindow->SetAffect`, `m_ppyGameWindow->ResetAffect`) aracılığıyla UI güncellenir.
    *   "Toggle" yetenekler için, etki aktifleştiğinde veya kalktığında ilgili yetenek slotu arayüzde aktif/deaktif edilir (`__ActivateSkillSlot`, `__DeactivateSkillSlot`).

*   **Bekleme Süresi Yönetimi (`__RunCoolTime`)**:
    *   Bir yetenek kullanıldığında bu fonksiyon çağrılır.
    *   Yetenek verilerinden temel bekleme süresi alınır.
    *   Karakterin sahip olduğu bekleme süresi azaltma etkileri (örneğin, 6. ve 7. efsunlar, taşlar) hesaba katılarak nihai bekleme süresi belirlenir.
    *   `ENABLE_ATTR_6TH_7TH` makrosu ile 6. ve 7. efsunlardan gelen bekleme süresi azaltma bonusları (belirli yetenekler için %10 veya %20 ihtimalle) eklenir.
    *   `POINT_CASTING_SPEED` (büyü hızı) değeri de bekleme süresini etkiler.
    *   Hesaplanan bekleme süresi arayüze (`m_ppyGameWindow->RunUseSkillEvent`) gönderilir.

*   **Koşul Kontrolleri**:
    *   `__CheckSkillUsable`: Yetenek kullanılıp kullanılamayacağını bir dizi alt kontrol fonksiyonuyla denetler.
    *   `__CheckShortArrow`: Yeterli ok olup olmadığını kontrol eder.
    *   `__CheckShortMana`: Yeterli SP (mana) olup olmadığını kontrol eder.
    *   `__CheckShortLife`: Yeterli HP olup olmadığını kontrol eder.
    *   `__CheckRestSkillCoolTime`: Yeteneğin bekleme süresinde olup olmadığını kontrol eder.
    *   `__CheckDashAffect`: Oyuncunun "dash" (atılma) etkisine sahip olup olmadığını kontrol eder.

*   **Lonca Yetenekleri (`UseGuildSkill`)**:
    *   Lonca yeteneklerinin kullanımı için özel bir fonksiyondur.
    *   Benzer şekilde bekleme süresi ve kullanım koşulları (örneğin, sadece lonca savaşında kullanılabilme) kontrol edilir.
    *   Sunucuya lonca yeteneği kullanım paketi gönderilir.

*   **Diğer Fonksiyonlar**:
    *   `ClearAffects`: Tüm aktif etkileri temizler.
    *   `FindSkillSlotIndexBySkillIndex`: Yetenek ID'sine göre slot indeksini bulur.
    *   `ClickSkillSlot`: Bir yetenek slotuna tıklandığında çağrılır, yeteneği kullanır veya aktif yetenek olarak ayarlar.
    *   `ChangeCurrentSkillNumberOnly`: Sadece aktif yetenek numarasını değiştirir (sağ tık yetenek modu için).
    *   `SetComboSkillFlag`: Kombo yeteneğinin durumunu (aktif/deaktif) ayarlar.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyuncunun oyun içinde yeteneklerini kullanabilmesi için gerekli tüm mantığı barındırır. Bir oyuncu klavyeden bir yetenek tuşuna bastığında veya fare ile yetenek çubuğundaki bir ikona tıkladığında, bu dosyadaki fonksiyonlar devreye girer. Yeteneğin kullanılabilir olup olmadığını (mana, bekleme süresi, hedef, özel koşullar vb.) kontrol eder, gerekirse hedefleme mekanizmalarını çalıştırır, karakterin yetenek animasyonunu başlatır, sunucuya yeteneğin kullanıldığı bilgisini gönderir ve arayüzde bekleme sürelerini, aktif etkileri günceller. Kısacası, oyuncunun yetenek sistemiyle etkileşiminin kalbidir.

**Önemli Fonksiyonlar**

*   `CPythonPlayer::__UseSkill(DWORD dwSlotIndex)`: Belirtilen slottaki yeteneği kullanmaya çalışır.
*   `CPythonPlayer::__CheckSkillUsable(DWORD dwSlotIndex)`: Bir yeteneğin kullanılabilir olup olmadığını kontrol eder.
*   `CPythonPlayer::__RunCoolTime(DWORD dwSkillSlotIndex)`: Yeteneğin bekleme süresini başlatır ve hesaplar.
*   `CPythonPlayer::SetAffect(UINT uAffect)`: Bir yetenek etkisini oyuncuya ekler.
*   `CPythonPlayer::ResetAffect(UINT uAffect)`: Bir yetenek etkisini oyuncudan kaldırır.
*   `CPythonPlayer::ClickSkillSlot(DWORD dwSlotIndex)`: Yetenek slotuna tıklama olayını işler.
*   `CPythonPlayer::UseGuildSkill(DWORD dwSkillSlotIndex)`: Lonca yeteneği kullanımını yönetir.

**UI Etkileşimleri**

*   `PyCallClassMemberFunc(m_ppyGameWindow, "ClearAffects", ...)`: Arayüzdeki tüm etki ikonlarını temizler.
*   `PyCallClassMemberFunc(m_ppyGameWindow, "SetAffect", ...)`: Arayüze bir etki ikonu ekler.
*   `PyCallClassMemberFunc(m_ppyGameWindow, "ResetAffect", ...)`: Arayüzden bir etki ikonunu kaldırır.
*   `PyCallClassMemberFunc(m_ppyGameWindow, "OnCannotUseSkill", ...)`: Yetenek kullanılamadığında arayüze bildirim gönderir (örneğin, "Yeterli mana yok", "Bekleme süresi dolmadı").
*   `PyCallClassMemberFunc(m_ppyGameWindow, "ChangeCurrentSkill", ...)`: Seçili yeteneği arayüzde günceller.
*   `PyCallClassMemberFunc(m_ppyGameWindow, "RunUseSkillEvent", ...)`: Yetenek kullanıldığında arayüze olay gönderir (bekleme süresi başlatma vb.).
*   `PyCallClassMemberFunc(m_ppyGameWindow, "OnCannotShotError", ...)`: Ok atılamadığında arayüze hata mesajı gönderir. 


### `PythonPrivateShop.h` & `PythonPrivateShop.cpp` (Özel Pazar Sistemi)

#### `PythonPrivateShop.h`

**Dosyanın Amacı ve İşlevselliği**

`PythonPrivateShop.h`, Metin2 istemcisindeki özel oyuncu pazarları (tezgahları) sisteminin C++ tarafındaki temel tanımlamalarını ve arayüzünü içerir. Bu başlık dosyası, `CPythonPrivateShop` singleton sınıfını, özel pazar örneklerini temsil eden `TPrivateShopInstance` yapısını, pazar eşyalarını, dekorasyonlarını ve arama sonuçlarını tanımlayan çeşitli veri yapılarını (struct) ve numaralandırmaları (enum) deklare eder. Özel pazar sisteminin istemci tarafındaki mantığının temelini oluşturur.

**Ana Çalışma Prensipleri ve Yapılar**

*   **`CPythonPrivateShop` (Singleton Sınıfı)**:
    *   Tüm özel pazar işlemlerini yöneten merkezi sınıftır.
    *   Hem oyuncunun kendi açtığı pazarın hem de diğer oyuncuların pazarlarının verilerini tutar.
    *   Pazar kurma, eşya ekleme/çıkarma/fiyat değiştirme, pazar arama, pazar bilgilerini alma gibi ana fonksiyonları sağlar.
    *   Dünyadaki pazar tezgahlarının görsel örneklerini (`TPrivateShopInstance`) yönetir.

*   **`TPrivateShopInstance` (Yapı)**:
    *   Oyun dünyasında görünen her bir özel pazar tezgahını temsil eder.
    *   Pazarın Sanal ID'si (`m_dwVID`), sahibinin adı (`m_strName`), grafiksel örneği (`m_pGraphicThingInstance`) ve pazarla ilişkili efektleri (`m_list_attachingEffect`) içerir.
    *   Pazar modelinin yüklenmesi, efektlerin eklenip çıkarılması ve güncellenmesi gibi görsel işlemleri yönetir.

*   **Numaralandırmalar (Enums)**:
    *   `EPrivateShopState`: Bir pazarın durumunu belirtir (örneğin, `STATE_CLOSED`, `STATE_OPEN`, `STATE_MODIFY`).
    *   `EPrivateShopSearchState`: Pazar arama sonuçlarındaki bir eşyanın durumunu belirtir.
    *   `EMode`: Pazarla etkileşim modunu tanımlar (bakma, ticaret yapma).
    *   `EFilterTypes`: Pazar arama filtrelerinin türlerini listeler.
    *   `EShopSearch`: Pazar arama sonuçları ve seçilen eşya limitlerini tanımlar.
    *   `EConfig`: Pazar görüntüleme mesafesi gibi yapılandırma sabitlerini içerir.
    *   `EAttachEffect`: Pazar tezgahlarına eklenebilecek efektlerin ömür türlerini belirtir.
    *   `EDeco`: Pazar dekorasyon türlerini (görünüm, başlık) ve bunların tokenlarını tanımlar.

*   **Veri Yapıları (Structs)**:
    *   `TAttachingEffect`: Pazar tezgahına bağlı bir efektin detaylarını tutar.
    *   `SAppearanceDeco`: Pazar görünüm dekorasyonunun adını ve model VNUM'unu içerir.
    *   `STitleDeco`: Pazar başlık dekorasyonunun adını, dosya yolunu ve metin rengini içerir.
    *   `TPrivateShopItemStock`: `TItemPos` (eşya konumu) ve `TPrivateShopItem` (satılan eşya detayları - fiyat, pozisyon) arasında bir eşleme (map) tutar, oyuncunun kendi pazarını kurarken kullandığı eşya stoğunu temsil eder.
    *   `m_aPrivateShopItem`, `m_aMyPrivateShopItem`: Diğer oyuncuların pazarlarındaki ve oyuncunun kendi pazarındaki eşyaların verilerini (`TPrivateShopItemData`) tutan dizilerdir.
    *   Pazar sahibinin bilgileri (`m_llGold`, `m_dwCheque`, `m_strMyTitle`, `m_bMyState` vb.).
    *   Görüntülenen aktif pazarın bilgileri (`m_strTitle`, `m_bState`, `m_dwActiveVID` vb.).
    *   Pazar arama sonuçları (`m_vec_searchItem` içinde `TPrivateShopSearchData`).
    *   Pazar dekorasyonları (`m_vec_appearanceDeco`, `m_vec_titleDeco`).

**Önemli Fonksiyon Deklarasyonları (CPythonPrivateShop içinde)**

*   `CreatePrivateShopInstance()`, `GetPrivateShopInstance()`, `DeletePrivateShopInstance()`: Dünyadaki pazar tezgahı örneklerini oluşturur, alır ve siler.
*   `Render()`, `Deform()`, `Update()`, `Destroy()`: Pazar tezgahı örneklerinin çizimi, deformasyonu, güncellenmesi ve yok edilmesiyle ilgili fonksiyonlardır.
*   `BuildPrivateShop()`: Yeni bir özel pazar kurma isteğini sunucuya gönderir.
*   `ClearPrivateShopStock()`, `AddPrivateShopItemStock()`, `DelPrivateShopItemStock()`, `GetPrivateShopItemPrice()`: Oyuncunun kendi pazarını kurarken kullanacağı eşya stoğunu yönetir.
*   `SetItemData()`, `GetItemData()`, `RemoveItemData()`, `ChangeItemPrice()`, `MoveItem()`: Pazardaki (hem kendi hem başkasının) eşyaların verilerini ayarlar, alır, siler, fiyatını değiştirir veya pozisyonunu değiştirir.
*   Oyuncunun kendi pazarının durumunu, altınını, çeki, başlığını, sayfa sayısını ayarlayan ve alan fonksiyonlar (örn: `SetMyState()`, `GetGold()`, `SetMyTitle()`).
*   Görüntülenen (aktif) pazarın bilgilerini ayarlayan ve alan fonksiyonlar (örn: `SetTitle()`, `GetState()`, `SetActiveVID()`).
*   `ClearSearchResult()`, `SortSearchResult()`, `SetSearchItemData()`, `GetSearchItemData()`: Pazar arama sonuçlarını yönetir.
*   `LoadDecoration()`: Pazar dekorasyon dosyalarını yükler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, oyuncuların özel pazar (tezgah) sistemini kullanabilmesi için gerekli olan istemci tarafı mantığını içerir. Bir oyuncu:
*   Yeni bir pazar kurmak istediğinde, satacağı eşyaları ve fiyatlarını seçer. Bu bilgiler `m_map_privateShopItemStock` içinde tutulur ve `BuildPrivateShop` ile sunucuya gönderilir.
*   Kendi açık pazarını yönetirken (eşya çıkarma, fiyat güncelleme) veya başka bir oyuncunun pazarını görüntülerken, eşya bilgileri `m_aMyPrivateShopItem` veya `m_aPrivateShopItem` üzerinden işlenir.
*   Pazar arama özelliğini kullandığında, sunucudan gelen sonuçlar `m_vec_searchItem` içinde depolanır, sıralanır ve arayüzde gösterilmek üzere hazırlanır.
*   Pazar kasasındaki altın/çek miktarını veya pazarındaki eşyaların toplam değerini görmek istediğinde ilgili fonksiyonlar çağrılır.

Bu dosya, pazar sistemiyle ilgili verilerin yerel olarak saklanması, sunucuyla iletişim kurulması ve arayüzün ihtiyaç duyduğu bilgilerin sağlanması açısından kritik bir rol oynar.

**Önemli Fonksiyonlar**

*   `CPythonPrivateShop::BuildPrivateShop(const char* c_szName, DWORD dwPolyVnum, BYTE bTitleType, BYTE bPageCount)`: Oyuncunun pazarını kurma isteğini başlatır.
*   `CPythonPrivateShop::AddPrivateShopItemStock(TItemPos ItemPos, WORD wDisplayPos, long long llPrice, DWORD dwCheque)`: Pazar kurma stoğuna eşya ekler.
*   `CPythonPrivateShop::DelPrivateShopItemStock(TItemPos ItemPos)`: Pazar kurma stoğundan eşya siler.
*   `CPythonPrivateShop::SetItemData(const TPrivateShopItemData& c_rShopItemData, bool bIsMainPlayerPrivateShop)`: (Kendi veya başkasının) pazarındaki bir eşyanın bilgilerini ayarlar.
*   `CPythonPrivateShop::GetItemData(WORD wPos, const TPrivateShopItemData** c_ppItemData, bool bIsMainPlayerPrivateShop)`: Pazardaki bir eşyanın bilgilerini alır.
*   `CPythonPrivateShop::ClearSearchResult()`: Pazar arama sonuçlarını temizler.
*   `CPythonPrivateShop::SetSearchItemData(TPrivateShopSearchData& rSearchItem)`: Pazar arama sonuçlarına yeni bir eşya ekler.
*   `CPythonPrivateShop::SortSearchResult()`: Pazar arama sonuçlarını sıralar.

**UI Etkileşimleri**

Bu dosyadaki fonksiyonlar doğrudan Python'a expose edilmese de (bu genellikle ayrı bir `PythonPrivateShopModule.cpp` gibi bir dosyada yapılır), `CPythonPrivateShop` singleton'ı, Python tarafındaki UI betiklerinin (`uiPrivateShop.py`, `uiPrivateShopBuilder.py`, `uiPrivateShopSearch.py` gibi) çağıracağı C++ fonksiyonları için temel veri yönetimi ve işlem mantığını sağlar. Örneğin, Python'daki bir pazar arayüzü, bir eşyayı satışa eklemek istediğinde, dolaylı olarak `AddPrivateShopItemStock` fonksiyonunu tetikleyecek bir ağ isteği gönderilmesine veya C++ çağrısı yapılmasına yol açar. Benzer şekilde, pazar eşyalarının listelenmesi için `GetItemData` gibi fonksiyonların döndürdüğü veriler kullanılır. 