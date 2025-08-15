# UserInterface Referans Kılavuzu - Bölüm 4

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir. Bölüm 3'te ele alınan konuların ardından, bu bölümde kalan C++ sınıfları, Python modülleri ve diğer ilgili bileşenler incelenecektir.

## İçindekiler

* [`InstanceBaseEvent.cpp` (CInstanceBase Sınıfının Parçası)](#instancebaseeventcpp-cinstancebase-sınıfının-parçası)
* [`InstanceBaseMotion.cpp` (CInstanceBase Sınıfının Parçası)](#instancebasemotioncpp-cinstancebase-sınıfının-parçası)
* [`InstanceBaseMovement.cpp` (CInstanceBase Sınıfının Parçası)](#instancebasemovementcpp-cinstancebase-sınıfının-parçası)
* [`InstanceBaseTransform.cpp` (CInstanceBase Sınıfının Parçası)](#instancebasetransformcpp-cinstancebase-sınıfının-parçası)
* [`InsultChecker.h` ve `InsultChecker.cpp`](#insultcheckerh-ve-insultcheckercpp)
* [`Locale_inc.h`, `Locale.h` ve `Locale.cpp` (Yerelleştirme Servisleri)](#locale_inch-localeh-ve-localecpp-yerelleştirme-servisleri)
* [`MarkImage.h` ve `MarkImage.cpp` (Lonca Sembolü Yönetimi)](#markimageh-ve-markimagecpp-lonca-sembolü-yönetimi)
* [`MarkManager.h` ve `MarkManager.cpp` (Lonca Mark ve Sembol Yöneticisi)](#markmanagerh-ve-markmanagercpp-lonca-mark-ve-sembol-yöneticisi)

---

### `InstanceBaseEvent.cpp` (CInstanceBase Sınıfının Parçası)

#### Dosyanın Genel Amacı

`InstanceBaseEvent.cpp` dosyası, `CInstanceBase` sınıfının olay işleme mekanizmasıyla ilgili temel fonksiyonlarını içerir. `CInstanceBase` nesneleri, `CActorInstance` (grafiksel varlık) aracılığıyla olayları alabilir ve işleyebilir. Bu dosyadaki fonksiyonlar, bu olay işleyicilere erişim ve bunların ayarlanması için arayüz sağlar. `InstanceBase.h` başlık dosyasında `CInstanceBase` sınıf tanımının bir parçası olarak deklare edilirler.

#### C++ Implementasyon Detayları (`InstanceBaseEvent.cpp`)

*   **`CActorInstance::IEventHandler& CInstanceBase::GetEventHandlerRef()`**:
    *   İlişkili `m_GraphicThingInstance` (bir `CActorInstance`) nesnesinin olay işleyicisine (`IEventHandler`) bir referans döndürür. Bu, olay işleyicinin doğrudan manipülasyonuna izin verir.
*   **`CActorInstance::IEventHandler* CInstanceBase::GetEventHandlerPtr()`**:
    *   İlişkili `m_GraphicThingInstance` nesnesinin olay işleyicisine bir işaretçi döndürür. İşleyici mevcut değilse null olabilir.
*   **`void CInstanceBase::SetEventHandler(CActorInstance::IEventHandler* pkEventHandler)`**:
    *   `m_GraphicThingInstance` için yeni bir olay işleyici ayarlar. Bu, `CInstanceBase`'in farklı olaylara nasıl tepki vereceğini dinamik olarak değiştirmek için kullanılabilir.

#### Oyunda Kullanım Amacı ve Senaryoları

`CInstanceBase` için olay işleme, oyun dünyasındaki çeşitli etkileşimler ve durum değişiklikleri için kritik öneme sahiptir:

*   **Animasyon Olayları**: Belirli bir animasyonun belirli bir karesinde bir ses çalmak veya bir efekt tetiklemek gibi olaylar bu mekanizma ile yönetilebilir.
*   **Girdi İşleme**: Karakterin kullanıcı girdilerine (örneğin, tıklama) tepki vermesi dolaylı olarak olay işleyiciler aracılığıyla gerçekleşebilir.
*   **Yapay Zeka (AI) Tetikleyicileri**: NPC'lerin veya canavarların belirli olaylara (örneğin, oyuncunun menzile girmesi) tepki vermesi için olay işleyiciler kullanılabilir.

Bu fonksiyonlar, `CInstanceBase`'in grafiksel temsili olan `CActorInstance` ile olay tabanlı iletişimi sağlar.

### `InstanceBaseMotion.cpp` (CInstanceBase Sınıfının Parçası)

#### Dosyanın Genel Amacı

`InstanceBaseMotion.cpp` dosyası, `CInstanceBase` sınıfının karakter animasyonları (motion) ve özel eylemleriyle ilgili fonksiyonlarını barındırır. Bu, genel hareketlerin (yürüme, koşma hariç) yanı sıra balıkçılık, duygular (emotions) gibi özel durum animasyonlarını yönetmeyi içerir. Fonksiyonlar `InstanceBase.h` başlık dosyasında `CInstanceBase` sınıf tanımının bir parçası olarak deklare edilirler.

#### C++ Implementasyon Detayları (`InstanceBaseMotion.cpp`)

*   **Animasyon Modu ve Kontrolü**:
    *   `void CInstanceBase::SetMotionMode(int iMotionMode)`: Karakterin hareket modunu (örneğin, normal, savaş) ayarlar.
    *   `int CInstanceBase::GetMotionMode(DWORD dwMotionIndex)`: Mevcut hareket modunu döndürür.
    *   `void CInstanceBase::SetLoopMotion(WORD wMotion, float fBlendTime = 0.1f, float fSpeedRatio)`: Belirtilen animasyonu döngüsel olarak oynatmaya başlar. `fBlendTime` ile önceki animasyondan yumuşak geçiş sağlanır, `fSpeedRatio` ile animasyon hızı ayarlanır.
    *   `void CInstanceBase::PushOnceMotion(WORD wMotion, float fBlendTime, float fSpeedRatio)`: Belirtilen animasyonu bir kez oynatır ve ardından genellikle bekleme (WAIT) animasyonuna döner. Animasyon kuyruğuna eklenir.
    *   `void CInstanceBase::PushLoopMotion(WORD wMotion, float fBlendTime, float fSpeedRatio)`: Belirtilen animasyonu döngüsel olarak oynatmak üzere animasyon kuyruğuna ekler.
    *   `void CInstanceBase::ResetLocalTime()`: Animasyonun yerel zamanını sıfırlar.
    *   `void CInstanceBase::SetEndStopMotion()`: Mevcut animasyon bittiğinde karakterin durmasını sağlar.
    *   `BOOL CInstanceBase::isLock()`: Karakterin bir animasyon tarafından kilitli olup olmadığını (yani başka bir animasyona geçip geçemeyeceğini) kontrol eder.

*   **Balıkçılık (Fishing) Mekanikleri**:
    *   `void CInstanceBase::StartFishing(float frot)`: Balıkçılık eylemini başlatır. Karakteri belirtilen yöne döndürür, oltanın düşeceği pozisyonu hesaplar ve balıkçılık animasyonlarını (FISHING_THROW, FISHING_WAIT) oynatır.
    *   `void CInstanceBase::StopFishing()`: Balıkçılığı durdurur, ilgili animasyonu oynatır ve bekleme durumuna geçer.
    *   `void CInstanceBase::ReactFishing()`: Balık tutma tepkisi animasyonunu oynatır.
    *   `void CInstanceBase::CatchSuccess()`: Başarılı balık tutma animasyonunu oynatır.
    *   `void CInstanceBase::CatchFail()`: Başarısız balık tutma animasyonunu oynatır.
    *   `BOOL CInstanceBase::GetFishingRot(int* pirot)`: Karakterin balık tutabileceği uygun bir yön (suya doğru) olup olmadığını kontrol eder ve varsa açıyı döndürür.

*   **Duygu (Emotion) ve Özel Eylemler**:
    *   `void CInstanceBase::__EnableChangingTCPState()`, `void CInstanceBase::__DisableChangingTCPState()`: TCP (Target Character Position) durumunun değiştirilip değiştirilemeyeceğini kontrol eden bayrakları ayarlar. Bu, bazı animasyonlar sırasında karakterin pozisyonunun ağ üzerinden senkronize edilmesini engellemek veya izin vermek için kullanılır.
    *   `void CInstanceBase::ActDualEmotion(CInstanceBase& rkDstInst, WORD wMotionNumber1, WORD wMotionNumber2)`: İki karakter arasında senkronize bir duygu (örneğin, dans, selamlaşma) eylemi başlatır. Hedef karakterin pozisyonunu ayarlar ve her iki karakter için de ilgili animasyonları tetikler. Ana oyuncuysa, duygu sürecini başlatır.
    *   `void CInstanceBase::ActEmotion(DWORD dwMotionNumber)`: Tek bir karakter için belirtilen duygu animasyonunu oynatır.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu fonksiyonlar, karakterlerin canlı ve etkileşimli görünmesini sağlar:

*   **Genel Animasyonlar**: Karakterlerin bekleme, yürüme, koşma dışındaki tüm animasyonları (saldırı, büyü yapma, özel yetenekler) bu fonksiyonlarla yönetilir.
*   **Balıkçılık**: Oyuncuların balık tutma mini oyununu oynaması için gerekli tüm animasyon ve durum değişiklikleri bu fonksiyonlar aracılığıyla kontrol edilir.
*   **Sosyal Etkileşimler**: Oyuncuların birbirleriyle veya NPC'lerle çeşitli duyguları (alkışlama, gülme, üzülme vb.) ifade etmeleri sağlanır. Çiftli duygular, iki karakterin senkronize hareket etmesini gerektiren daha karmaşık sosyal etkileşimler sunar.
*   **Özel Durumlar**: Bazı görevler veya oyun içi olaylar, karakterlerin belirli animasyonları oynamasını gerektirebilir.

Bu dosyadaki kodlar, `CInstanceBase`'in görsel davranışlarının ve oyuncu komutlarına verdiği tepkilerin önemli bir bölümünü oluşturur.

### `InstanceBaseMovement.cpp` (CInstanceBase Sınıfının Parçası)

#### Dosyanın Genel Amacı

`InstanceBaseMovement.cpp` dosyası, `CInstanceBase` sınıfının karakterlerin oyun dünyasındaki temel fiziksel hareketlerini yöneten fonksiyonlarını içerir. Bu, yürüme, koşma, durma, hedef pozisyona hareket etme, hız ayarları ve hareketle ilgili durum kontrollerini kapsar. Fonksiyonlar `InstanceBase.h` başlık dosyasında `CInstanceBase` sınıf tanımının bir parçası olarak deklare edilirler.

#### C++ Implementasyon Detayları (`InstanceBaseMovement.cpp`)

*   **Hız Ayarları**:
    *   `void CInstanceBase::SetAttackSpeed(UINT uAtkSpd)`: Karakterin saldırı hızını ayarlar. Bu değer `m_GraphicThingInstance` (grafiksel temsil) ve `m_kHorse` (eğer varsa at) için kullanılır. Değer genellikle 100 tabanlıdır (örneğin, 120, %20 daha hızlı demektir).
    *   `void CInstanceBase::SetMoveSpeed(UINT uMovSpd)`: Karakterin hareket hızını ayarlar. Saldırı hızı gibi, bu da grafiksel temsil ve at için kullanılır.
    *   `void CInstanceBase::SetRotationSpeed(float fRotSpd)`: Karakterin maksimum dönüş hızını ayarlar.

*   **Temel Hareket Kontrolleri**:
    *   `void CInstanceBase::NEW_Stop()`: Karakterin mevcut hareketini durdurur. Eğer senkronizasyon durumunda değilse, kilitli değilse veya bir yetenek kullanmıyorsa ve bekliyorsa yürüme eylemini sonlandırır.
    *   `void CInstanceBase::NEW_SyncPixelPosition(long& nPPosX, long& nPPosY)`: Karakterin pozisyonunu sunucudan gelen verilerle senkronize etmek için kullanılır (`TEMP_Push` ile grafik nesnesine iletilir).
    *   `bool CInstanceBase::NEW_CanMoveToDestPixelPosition(const TPixelPosition& c_rkPPosDst)`: Karakterin belirtilen hedef piksel pozisyonuna hareket edip edemeyeceğini kontrol eder (mevcut pozisyondan farklı olup olmadığına bakar).
    *   `float CInstanceBase_GetDegreeFromPosition(float x, float y)`: Verilen x, y vektöründen bir açı (derece cinsinden) hesaplar.
    *   `float CInstanceBase::NEW_GetAdvancingRotationFromDirPixelPosition(const TPixelPosition& c_rkPPosDir)`: Bir yön vektöründen ilerleme açısını hesaplar.
    *   `float CInstanceBase::NEW_GetAdvancingRotationFromDestPixelPosition(const TPixelPosition& c_rkPPosDst)`: Mevcut pozisyondan hedef pozisyona doğru olan ilerleme açısını hesaplar.
    *   `float CInstanceBase::NEW_GetAdvancingRotationFromPixelPosition(const TPixelPosition& c_rkPPosSrc, const TPixelPosition& c_rkPPosDst)`: Kaynak ve hedef pozisyonlarına göre ilerleme açısını hesaplar.
    *   `void CInstanceBase::NEW_SetAdvancingRotationFromDirPixelPosition(const TPixelPosition& c_rkPPosDir)`: Yön vektörüne göre ilerleme açısını ayarlar ve dönüş yönünü belirler.
    *   `void CInstanceBase::NEW_SetAdvancingRotationFromPixelPosition(const TPixelPosition& c_rkPPosSrc, const TPixelPosition& c_rkPPosDst)`: Kaynak ve hedef pozisyonlarına göre ilerleme açısını ayarlar.
    *   `bool CInstanceBase::NEW_SetAdvancingRotationFromDestPixelPosition(const TPixelPosition& c_rkPPosDst)`: Hedef pozisyona göre ilerleme açısını ayarlar. Eğer hareket mümkün değilse `false` döner.
    *   `void CInstanceBase::SetAdvancingRotation(float fRotation)`: Karakterin ilerleyeceği hedef açıyı ayarlar ve bu açıya göre dönüş hızını optimize eder.
    *   `void CInstanceBase::StartWalking()`: Karakterin yürüme/koşma animasyonunu başlatır ve ilgili buff etkilerini (örneğin, Gyeonggong, Kwaesok) aktive eder.
    *   `void CInstanceBase::EndWalking(float fBlendingTime = 0.1f)`: Karakterin yürüme/koşma eylemini sonlandırır, durma animasyonuna geçer ve ilgili buff etkilerini deaktive eder. `fBlendingTime` ile yumuşak geçiş sağlanır.
    *   `void CInstanceBase::EndWalkingWithoutBlending()`: Yürüme/koşmayı anında, yumuşak geçiş olmadan sonlandırır.

*   **Durum Kontrolleri (Movement ile ilgili)**:
    *   `BOOL CInstanceBase::IsWaiting()`: Karakterin bekleme durumunda olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsWalking()`: Karakterin yürüyor/koşuyor olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsPushing()`: Karakterin itiliyor olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsAttacked()`: Karakterin saldırı almış ve tepki veriyor olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsKnockDown()`: Karakterin yere düşmüş (knock-down) olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsAttacking()`: Karakterin saldırı yapıyor olup olmadığını kontrol eder.
    *   `BOOL CInstanceBase::IsActingTargetEmotion()`: Karakterin hedefe yönelik bir duygu eylemi gerçekleştirip gerçekleştirmediğini kontrol eder.
    *   `BOOL CInstanceBase::IsActingEmotion()`: Karakterin bir duygu eylemi gerçekleştirip gerçekleştirmediğini kontrol eder.

*   **Hedefe Yönelik Hareket**:
    *   `BOOL CInstanceBase::IsGoing()`: Karakterin bir hedefe doğru aktif olarak hareket edip etmediğini (`m_isGoing` bayrağı) kontrol eder.
    *   `void CInstanceBase::NEW_MoveToDestInstanceDirection(CInstanceBase& rkInstDst)`: Başka bir `CInstanceBase` örneğinin (hedef) bulunduğu yöne doğru hareket başlatır.
    *   `bool CInstanceBase::NEW_MoveToDestPixelPositionDirection(const TPixelPosition& c_rkPPosDst)`: Belirtilen piksel pozisyonuna doğru hareket başlatır. Önce o yöne döner, sonra ilerler.
    *   `bool CInstanceBase::NEW_Goto(const TPixelPosition& c_rkPPosDst, float fDstRot)`: Belirtilen hedef pozisyona ve hedef açıya doğru hareket başlatır. Eğer karakter zaten hareket ediyorsa veya kilitliyse işlem yapmaz. `m_isGoing` bayrağını `TRUE` yapar ve yürüme animasyonunu başlatır.
    *   `void CInstanceBase::NEW_MoveToDirection(float fDirRot)`: Belirtilen bir yöne doğru (sabit bir mesafe kadar) hareket başlatır. `m_isGoing` bayrağını `FALSE` yapar çünkü belirli bir hedefe değil, bir yöne doğru anlık bir harekettir.
    *   `void CInstanceBase::EndGoing()`: `NEW_Goto` ile başlatılan hedefe yönelik hareketi sonlandırır (`m_isGoing = FALSE;`) ve yürüme animasyonunu durdurur.

*   **Hareket Modları**:
    *   `void CInstanceBase::SetRunMode()`: Karakterin hareket modunu "koşma" olarak ayarlar.
    *   `void CInstanceBase::SetWalkMode()`: Karakterin hareket modunu "yürüme" olarak ayarlar.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu fonksiyonlar, karakterlerin oyun dünyasında gezinmesinin temelini oluşturur:

*   **Oyuncu Kontrolü**: Oyuncu fare ile tıkladığında veya klavye ile hareket komutları verdiğinde, bu fonksiyonlar çağrılarak karakterin ilgili pozisyona veya yöne hareket etmesi sağlanır.
*   **NPC ve Canavar Hareketi**: Yapay zeka tarafından kontrol edilen karakterler de (NPC'ler, canavarlar) devriye gezerken, oyuncuyu takip ederken veya bir hedefe kaçarken bu hareket fonksiyonlarını kullanır.
*   **Hız Değişiklikleri**: Binekler, yetenekler veya eşyalar karakterlerin saldırı ve hareket hızlarını değiştirebilir; bu değişiklikler `SetAttackSpeed` ve `SetMoveSpeed` ile uygulanır.
*   **Durum Tespiti**: Oyun mantığının diğer kısımları (örneğin, yetenek kullanımı, etkileşimler), bir karakterin hareket durumu hakkında bilgi almak için `IsWaiting`, `IsWalking` gibi fonksiyonları kullanır.
*   **Senkronizasyon**: `NEW_SyncPixelPosition` gibi fonksiyonlar, istemci ve sunucu arasında karakter pozisyonlarının tutarlı kalmasına yardımcı olur.

Bu dosya, `UserInterface` modülünün `GameLib` ve grafik motoru ile etkileşimde bulunarak karakterlerin akıcı ve doğru bir şekilde hareket etmesini sağlayan kritik bir bileşendir.

### `InstanceBaseTransform.cpp` (CInstanceBase Sınıfının Parçası)

#### Dosyanın Genel Amacı

`InstanceBaseTransform.cpp` dosyası, `CInstanceBase` sınıfının karakterlerin oyun dünyasındaki pozisyonlarını, rotasyonlarını (dönüş açıları) ve yönlerini yöneten fonksiyonlarını içerir. Bu fonksiyonlar, karakterin konumunu ayarlamak, baktığı yönü değiştirmek ve bu bilgileri sorgulamak için kullanılır. Fonksiyonlar `InstanceBase.h` başlık dosyasında `CInstanceBase` sınıf tanımının bir parçası olarak deklare edilirler.

#### C++ Implementasyon Detayları (`InstanceBaseTransform.cpp`)

*   **Pozisyon Yönetimi**:
    *   `void CInstanceBase::SCRIPT_SetPixelPosition(float fx, float fy)`: Python betiklerinden çağrılmak üzere tasarlanmıştır. Verilen 2D koordinatlar (fx, fy) için zemin yüksekliğini (`__GetBackgroundHeight`) bularak karakterin 3D pozisyonunu ayarlar.
    *   `void CInstanceBase::NEW_SetPixelPosition(const TPixelPosition& c_rPixelPosition)`: Karakterin mevcut piksel pozisyonunu doğrudan ayarlar (`m_GraphicThingInstance.SetCurPixelPosition`).
    *   `void CInstanceBase::NEW_GetPixelPosition(TPixelPosition* pPixelPosition)`: Karakterin mevcut piksel pozisyonunu `pPixelPosition` işaretçisi aracılığıyla döndürür.

*   **Rotasyon (Dönüş) Yönetimi**:
    *   `void CInstanceBase::SetRotation(float fRotation)`: Karakterin Y ekseni etrafındaki dönüş açısını (yaw) anında ayarlar.
    *   `void CInstanceBase::BlendRotation(float fRotation, float fBlendTime)`: Karakterin mevcut açısından hedef `fRotation` açısına `fBlendTime` süresi boyunca yumuşak bir geçişle dönmesini sağlar.
    *   `float CInstanceBase::GetRotation()`: Karakterin mevcut dönüş açısını döndürür.
    *   `float CInstanceBase::GetAdvancingRotation()`: Karakterin hareket ederken ilerlediği hedef açıyı (eğer varsa) döndürür.

*   **Bakış (LookAt) Fonksiyonları**:
    *   `void CInstanceBase::NEW_LookAtFlyTarget()`: Karakterin, varsa uçan bir hedefe (fly target) bakmasını sağlar.
    *   `void CInstanceBase::NEW_LookAtDestPixelPosition(const TPixelPosition& c_rkPPosDst)`: Karakterin belirtilen bir hedef piksel pozisyonuna (X, -Y koordinatlarını kullanarak) bakmasını sağlar.
    *   `void CInstanceBase::NEW_LookAtDestInstance(CInstanceBase& rkInstDst)`: Karakterin başka bir `CInstanceBase` örneğine (hedef karakter/nesne) bakmasını sağlar.

*   **Yön (Direction) Yönetimi**:
    *   `void CInstanceBase::SetDirection(int dir)`: Karakterin yönünü 8 ana yönden (DIR_NORTH, DIR_NORTHEAST vb. gibi `DIR_MAX_NUM` ile tanımlanan) birine göre ayarlar. Hem anlık rotasyonu hem de ilerleme rotasyonunu ayarlar.
    *   `void CInstanceBase::BlendDirection(int dir, float blendTime)`: Karakterin belirtilen yöne doğru yumuşak bir geçişle dönmesini sağlar.
    *   `float CInstanceBase::GetDegreeFromDirection(int dir)`: 8 ana yönden birine karşılık gelen açıyı (derece cinsinden) döndüren statik bir yardımcı fonksiyondur. Örneğin, `DIR_NORTH` 0.0 derece, `DIR_NORTHEAST` 45.0 derece vb.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu dönüşüm fonksiyonları, karakterlerin oyun dünyasındaki varlığını ve etkileşimini tanımlamada temel rol oynar:

*   **Karakter Oluşturma ve Işınlama**: Bir karakter oyuna girdiğinde veya bir yerden bir yere ışınlandığında, başlangıç pozisyonu ve yönü bu fonksiyonlarla ayarlanır (`SCRIPT_SetPixelPosition`, `SetDirection`).
*   **Hareket ve Navigasyon**: Karakterler bir hedefe doğru hareket ederken (`InstanceBaseMovement.cpp` içindeki fonksiyonlar), sürekli olarak rotasyonları güncellenir (`SetAdvancingRotation`, `BlendRotation`). Hedefe ulaştıklarında veya bir eylem gerçekleştireceklerinde belirli bir yöne bakmaları gerekebilir (`NEW_LookAtDestPixelPosition`, `NEW_LookAtDestInstance`).
*   **Yetenek Kullanımı ve Etkileşimler**: Bazı yetenekler veya etkileşimler, karakterin belirli bir yöne veya hedefe bakmasını gerektirir.
*   **Kamera Kontrolü**: Oyuncu kamerasının karakteri takip etmesi, karakterin pozisyon ve rotasyon bilgilerine dayanır.
*   **Betik Kontrolü**: `SCRIPT_SetPixelPosition` gibi fonksiyonlar, Python betiklerinin karakterlerin pozisyonlarını oyun mantığına göre dinamik olarak değiştirmesine olanak tanır.

Bu dosyadaki fonksiyonlar, `CInstanceBase`'in temel uzamsal özelliklerini yöneterek oyun dünyasında tutarlı ve doğru bir şekilde konumlanmasını ve yönlenmesini sağlar.

### `InsultChecker.h` ve `InsultChecker.cpp`

#### Sınıfın Genel Amacı

`CInsultChecker` sınıfı, oyun içinde kullanılan metinlerde (örneğin, sohbet mesajları, karakter isimleri, lonca isimleri) küfür, argo veya istenmeyen kelimelerin ("insult") bulunup bulunmadığını kontrol etmek ve bunları filtrelemek için tasarlanmış bir yardımcı singleton sınıfıdır. Temel amacı, oyun içi iletişimin belirli bir düzeyde tutulmasına yardımcı olmaktır.

#### Başlık Dosyası Tanımları (`InsultChecker.h`)

`InsultChecker.h` dosyası, `CInsultChecker` sınıfının yapısını ve arayüzünü tanımlar. (Bu dosya doğrudan sağlanmamış olsa da, `InsultChecker.cpp` içeriğinden çıkarımlar yapılabilir):

```cpp
#pragma once // Genellikle başlık dosyalarında bulunur

#include <string>
#include <list>
// Muhtemelen #include "../EterLocale/Locale.h" veya benzeri bir başlık (LocaleService_*)

class CInsultChecker
{
public:
    static CInsultChecker& GetSingleton();

    CInsultChecker();
    virtual ~CInsultChecker();

    void Clear();
    void AppendInsult(const std::string& c_rstInsult);
    bool IsInsultIn(const char* c_szLine, UINT uLineLen);
    void FilterInsult(char* szLine, UINT uLineLen);

protected:
    bool __IsInsult(const char* c_szWord);
    bool __GetInsultLength(const char* c_szWord, UINT* puInsultLen);

private:
    std::list<std::string> m_kList_stInsult; // Yasaklı kelimelerin listesi
};
```

*   **`static CInsultChecker& GetSingleton()`**: Singleton deseni için global erişim noktası.
*   **`CInsultChecker()` ve `~CInsultChecker()`**: Kurucu ve yıkıcı metotlar.
*   **`void Clear()`**: Mevcut tüm yasaklı kelime listesini temizler.
*   **`void AppendInsult(const std::string& c_rstInsult)`**: Yasaklı kelime listesine yeni bir kelime ekler.
*   **`bool IsInsultIn(const char* c_szLine, UINT uLineLen)`**: Verilen bir metin satırında herhangi bir yasaklı kelimenin olup olmadığını kontrol eder.
*   **`void FilterInsult(char* szLine, UINT uLineLen)`**: Verilen bir metin satırındaki yasaklı kelimeleri belirli bir karakterle (örneğin, `*`) değiştirir.
*   **`bool __IsInsult(const char* c_szWord)`**: Yardımcı fonksiyon, bir kelimenin doğrudan yasaklı olup olmadığını kontrol eder.
*   **`bool __GetInsultLength(const char* c_szWord, UINT* puInsultLen)`**: Yardımcı fonksiyon, bir kelime yasaklıysa uzunluğunu döndürür.
*   **`std::list<std::string> m_kList_stInsult`**: Yasaklı kelimeleri saklayan özel üye değişken.

#### C++ Implementasyon Detayları (`InsultChecker.cpp`)

*   **`GetSingleton()`**: Standart singleton implementasyonu, statik bir `CInsultChecker` örneği döndürür.
*   **`Clear()`**: `m_kList_stInsult` listesini temizler.
*   **`AppendInsult(const std::string& c_rstInsult)`**: Gelen kelime boş değilse `m_kList_stInsult` listesine ekler.
*   **`__GetInsultLength(const char* c_szWord, UINT* puInsultLen)`**:
    *   `m_kList_stInsult` içindeki her bir yasaklı kelime (`rstInsult`) ile verilen `c_szWord`'ü karşılaştırır.
    *   Karşılaştırma `LocaleService_StringCompareCI` (büyük/küçük harf duyarsız karşılaştırma) fonksiyonu ile yapılır ve yasaklı kelimenin uzunluğu kadar karakter karşılaştırılır.
    *   Eşleşme bulunursa, `*puInsultLen` yasaklı kelimenin uzunluğuna ayarlanır ve `true` döndürülür.
*   **`__IsInsult(const char* c_szWord)`**: `__GetInsultLength` fonksiyonunu çağırarak bir kelimenin yasaklı olup olmadığını kontrol eder.
*   **`FilterInsult(char* szLine, UINT uLineLen)`**:
    *   Verilen `szLine` üzerinde karakter karakter ilerler.
    *   Her pozisyonda, o pozisyondan başlayan alt dizgenin `__GetInsultLength` ile yasaklı olup olmadığını kontrol eder.
    *   Eğer yasaklı bir kelime bulunursa, `memset` ile o kelimenin geçtiği bölüm `INSULT_FILTER_CHAR` (varsayılan `*`) ile doldurulur ve `uPos` yasaklı kelimenin uzunluğu kadar artırılır.
    *   Eğer yasaklı değilse, karakterin çok baytlı olup olmadığına (`LocaleService_IsLeadByte`) göre `uPos` 1 veya 2 artırılır.
*   **`IsInsultIn(const char* c_szLine, UINT uLineLen)`**:
    *   Verilen `c_szLine` üzerinde karakter karakter ilerler.
    *   Her pozisyonda, `__IsInsult` ile o pozisyondan başlayan alt dizgenin yasaklı olup olmadığını kontrol eder.
    *   Herhangi bir yasaklı kelime bulunursa hemen `true` döndürür.
    *   Çok baytlı karakterleri doğru şekilde işler.

#### Oyunda Kullanım Amacı ve Senaryoları

`CInsultChecker`, oyun istemcisinin çeşitli yerlerinde metin tabanlı girdileri ve gösterimleri denetlemek için kullanılır:

1.  **Yasaklı Kelime Listesinin Yüklenmesi**: Oyun başlangıcında veya belirli bir konfigürasyon dosyasından `AppendInsult` aracılığıyla yasaklı kelimeler listesi `CInsultChecker`'a yüklenir. Bu liste genellikle sunucu tarafından sağlanır veya istemci tarafında önceden tanımlanır.
2.  **Sohbet Filtreleme**: Oyuncuların sohbet penceresine yazdığı mesajlar gönderilmeden veya görüntülenmeden önce `FilterInsult` ile filtrelenir. Yasaklı kelimeler `***` gibi karakterlerle değiştirilir.
3.  **İsim Denetimi**: Oyuncular karakter, evcil hayvan veya lonca oluştururken verdikleri isimler `IsInsultIn` ile kontrol edilir. Eğer yasaklı bir kelime içeriyorsa, isim reddedilebilir.
4.  **Diğer Metin Alanları**: Oyun içindeki diğer kullanıcı tarafından oluşturulan metinler (örneğin, özel dükkan isimleri, lonca duyuruları) da benzer şekilde denetlenebilir.

Bu sınıf, oyun ortamının daha düzenli ve kullanıcı dostu kalmasına yardımcı olur. `LocaleService` fonksiyonlarını kullanması, farklı dil ve karakter setlerini destekleyebileceğini gösterir.

### `Locale_inc.h`, `Locale.h` ve `Locale.cpp` (Yerelleştirme Servisleri)

Bu üç dosya, Metin2 istemcisinin yerelleştirme (localization - L10n) ve bölgeselleştirme (regionalization) altyapısının temelini oluşturur. Birlikte çalışarak oyunun farklı dil, bölge ve servis sağlayıcılarına göre özelliklerini, metinlerini, yapılandırmalarını ve hatta bazı oyun içi verilerini yönetirler.

#### Dosyaların Genel Amaçları

*   **`Locale_inc.h`**: Bu dosya, bir dizi `#define` önişlemci direktifinden oluşur. Bu direktifler, genellikle belirli bir özelliği veya sistemi etkinleştirmek için kullanılır. Örnek kategoriler ve makrolar:
    *   **Para Birimi Sistemleri**: `ENABLE_CHEQUE_SYSTEM`, `ENABLE_GEM_SYSTEM`
    *   **Ejderha Taşı Simyası**: `ENABLE_DRAGON_SOUL_SYSTEM`, `ENABLE_DS_GRADE_MYTH`
    *   **Kostüm Sistemleri**: `ENABLE_COSTUME_SYSTEM`, `ENABLE_WEAPON_COSTUME_SYSTEM`
    *   **Envanter ve Ekipman**: `ENABLE_EXTEND_INVEN_SYSTEM`, `ENABLE_QUIVER_SYSTEM`, `ENABLE_NEW_EQUIPMENT_SYSTEM`
    *   **Dükkan ve Ticaret**: `ENABLE_SHOPEX_RENEWAL`, `ENABLE_PREMIUM_PRIVATE_SHOP`
    *   **Karakter Özellikleri**: `ENABLE_WOLFMAN_CHARACTER`, `ENABLE_CONQUEROR_LEVEL`
    *   **Yetenek Sistemleri**: `ENABLE_678TH_SKILL`
    *   **Lonca ve Parti**: `ENABLE_GUILD_LEADER_GRADE_NAME`
    *   **Oyun Sistemleri**: `ENABLE_QUEST_RENEWAL`, `ENABLE_CUBE_RENEWAL`, `ENABLE_SEND_TARGET_INFO`
    *   **Harita ve Zindanlar**: `ENABLE_SHIP_DEFENSE` (Hidra Zindanı)
    *   **Etkinlikler**: `ENABLE_EVENT_BANNER_FLAG`
    *   **Pet Sistemleri**: `ENABLE_GROWTH_PET_SYSTEM`
    *   **Eşya Özellikleri**: `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_ATTR_6TH_7TH`
    *   **Arayüz (UI) Özellikleri**: `ENABLE_DETAILS_UI`, `WJ_SHOW_MOB_INFO`, `ENABLE_KEYCHANGE_SYSTEM`
    *   **Grafik ve Çevre**: `ENABLE_FOV_OPTION`, `ENABLE_SHADOW_RENDER_QUALITY_OPTION`
    *   **Uygulama ve Ağ Ayarları**: `DISABLE_INDEX_FILE`, `ENABLE_DISCORD_RPC`, `CEF_BROWSER`

Bu makrolar, oyunun farklı sürümlerinin (örneğin, farklı yayıncılar veya bölgeler için) farklı özellik setlerine sahip olmasını sağlar. Kodun diğer kısımları `#ifdef ENABLE_XYZ_SYSTEM ... #endif` gibi bloklarla bu makroların varlığını kontrol ederek ilgili kod bölümlerini derler veya derlemez.

#### `Locale.h` API Fonksiyonları (Önemli Olanlar)

`Locale.h` içindeki `LocaleService_` fonksiyonları şunları içerir:

*   **Bölge Kontrol Fonksiyonları**:
    *   `bool LocaleService_IsYMIR()`: Mevcut bölgenin YMIR (genellikle Kore veya geliştirici varsayılanı) olup olmadığını kontrol eder.
    *   `bool LocaleService_IsJAPAN()`, `LocaleService_IsENGLISH()`, `LocaleService_IsEUROPE()` vb.: Belirli bir bölgeye veya dil sürümüne ait olup olmadığını kontrol eder.
*   **Temel Yerel Bilgileri Alma**:
    *   `unsigned LocaleService_GetCodePage()`: Mevcut yerel ayarın kullandığı Windows kod sayfasını döndürür (örneğin, 1252 Batı Avrupa, 949 Korece).
    *   `const char* LocaleService_GetName()`: Servis sağlayıcısının veya genel bölgenin adını döndürür (örneğin, "YMIR", "EUROPE").
    *   `const char* LocaleService_GetLocaleName()`: Spesifik yerel ayar adını döndürür (örneğin, "ymir", "de", "en"). Bu genellikle `locale/` altındaki klasör adıyla eşleşir.
    *   `const char* LocaleService_GetLocalePath()`: Yerel ayar dosyalarının bulunduğu tam yolu döndürür (örneğin, "locale/de").
    *   `const char* LocaleService_GetSecurityKey()`: Mevcut yerel ayar için kullanılan güvenlik anahtarını (genellikle ağ paketleri veya istemci-sunucu iletişimi için) döndürür.
*   **Metin İşleme**:
    *   `BOOL LocaleService_IsLeadByte(const char chByte)`: Verilen bir karakterin çok baytlı bir karakterin başlangıç baytı olup olmadığını kontrol eder (yerel ayara göre).
    *   `int LocaleService_StringCompareCI(LPCSTR szStringLeft, LPCSTR szStringRight, size_t sizeLength)`: İki stringi büyük/küçük harf duyarsız olarak karşılaştırır (yerel ayara göre).
*   **Yerel Ayar Yönetimi**:
    *   `void LocaleService_ForceSetLocale(const char* name, const char* localePath)`: Yerel ayarı programatik olarak belirli bir ada ve yola zorlar (genellikle test veya özel durumlar için).
    *   `BYTE LocaleService_GetLocaID()`: Mevcut yerel ayar adına göre `CPythonApplication` içinde tanımlı bir `LOCALE_` ID'si döndürür.
    *   `bool LocaleService_SaveLoca(int iCodePage, const char* szLocale)`: Seçilen yerel ayarı ve kod sayfasını `loca.cfg` dosyasına kaydeder.
    *   `const char* LocaleService_GetLoca()`: `MULTI_LOCALE_NAME` değerini döndürür, bu da genellikle kullanıcı tarafından seçilen veya yapılandırılan yerel ayardır.
    *   `void LocaleService_LoadConfig(const char* fileName)`: Belirtilen yapılandırma dosyasından (genellikle "loca.cfg" veya "locale.cfg") yerel ayar bilgilerini (kod sayfası, isim) yükler.
    *   `bool LocaleService_LoadGlobal(HINSTANCE hInstance)`: Eğer `LOCALE_SERVICE_GLOBAL` tanımlıysa, istemcide paketlenmiş birden fazla yerel ayar arasından kullanıcının seçim yapması için bir iletişim kutusu gösterir ve seçilen ayarları yükler.
*   **Oyuna Özel Veri Fonksiyonları**:
    *   `unsigned LocaleService_GetLastExp(int level)`: Belirli bir lonca seviyesi için gereken toplam deneyim puanını döndürür. Bu değerler `LocaleService_IsCHEONMA()` (Çin sunucuları için özel bir yerel ayar) durumuna göre farklılık gösterebilir.
    *   `int LocaleService_GetSkillPower(unsigned level)`: Belirli bir yetenek seviyesi için yetenek gücünü döndürür. Bu da `LocaleService_IsCHEONMA()` durumuna göre değişebilir.
*   **CHEONMA Özel Fonksiyonları**:
    *   `void LocaleService_SetCHEONMA(bool isEnable)`: CHEONMA modunu aktif veya pasif yapar.
    *   `bool LocaleService_IsCHEONMA()`: CHEONMA modunun aktif olup olmadığını kontrol eder (aslında `LocaleService_IsYMIR()`'ı çağırır).

#### `Locale.cpp` Implementasyon Detayları

*   **Global Değişkenler**:
    *   `LSS_YMIR`, `LSS_JAPAN` vb.: Sabit stringler olarak tanımlanmış bölge isimleri.
    *   `IS_CHEONMA`: CHEONMA modunun aktif olup olmadığını tutan boolean bayrak.
    *   `__SECURITY_KEY_STRING__`: Kullanılacak güvenlik anahtarını saklayan string.
    *   `MULTI_LOCALE_SERVICE`, `MULTI_LOCALE_PATH`, `MULTI_LOCALE_NAME`, `MULTI_LOCALE_CODE`, `MULTI_LOCALE_REPORT_PORT`: Aktif yerel ayar bilgilerini tutan global değişkenler. Bunlar `LocaleService_LoadConfig` veya `LocaleService_LoadGlobal` ile doldurulur.
*   **`LocaleService_LoadConfig()`**:
    *   Önce "loca.cfg" dosyasının varlığını kontrol eder. Varsa bu dosyayı, yoksa argüman olarak verilen `fileName`'i kullanır.
    *   Dosyayı okuyarak kod sayfasını ve yerel ayar adını `MULTI_LOCALE_CODE` ve `MULTI_LOCALE_NAME` değişkenlerine yükler. `MULTI_LOCALE_PATH` da buna göre ayarlanır.
*   **`LocaleService_GetLastExp()` ve `LocaleService_GetSkillPower()`**:
    *   İçlerinde statik diziler halinde (biri "CHEONMA" diğeri "INTERNATIONAL" için) deneyim ve yetenek gücü tabloları barındırır. `LocaleService_IsCHEONMA()` (pratikte `LocaleService_IsYMIR()`) fonksiyonunun sonucuna göre uygun tablodan değeri döndürür.
*   **Bölge Belirleme (`LocaleService_GetName`, `LocaleService_GetCodePage` vb.)**:
    *   Bu fonksiyonların davranışları, `Locale_inc.h` dosyasında `LOCALE_SERVICE_EUROPE`, `LOCALE_SERVICE_JAPAN` gibi hangi makronun tanımlandığına bağlıdır.
    *   Eğer `_LSS_USE_LOCALE_CFG` tanımlıysa (örneğin `LOCALE_SERVICE_EUROPE` için), bu fonksiyonlar `MULTI_LOCALE_*` global değişkenlerinden değerleri okur (yani `loca.cfg`'den gelen değerler).
    *   Eğer `_LSS_USE_LOCALE_CFG` tanımlı değilse ama spesifik bir bölge makrosu (örneğin `LOCALE_SERVICE_JAPAN`) tanımlıysa, o bölge için önceden belirlenmiş sabit değerleri (`_LSS_SERVICE_NAME`, `_LSS_SERVICE_CODEPAGE` vb.) döndürürler.
*   **`LocaleService_GetLocaID()`**: Mevcut `MULTI_LOCALE_NAME` değerini `CPythonApplication`'daki `LOCALE_` sabitleriyle karşılaştırarak bir ID döndürür. Bu, genellikle Python tarafında dil dosyalarını veya arayüz elemanlarını seçmek için kullanılır.
*   **`LocaleService_SaveLoca()`**: Kullanıcının oyun içinden dil seçimi yapması durumunda, seçilen kod sayfasını ve yerel ayar adını "loca.cfg" dosyasına yazar. `SetDefaultCodePage` ile sistemin varsayılan kod sayfasını da ayarlar.
*   **`LocaleService_LoadGlobal()`**:
    *   Sadece `LOCALE_SERVICE_GLOBAL` makrosu tanımlıysa çalışır.
    *   `gs_stLocaleData` adlı bir statik yapılar dizisi üzerinden tanımlı yerel ayarları tarar.
    *   Her bir yerel ayar için `locale/[locale_name]/item_proto` dosyasının varlığını `CEterPackManager` ile kontrol eder.
    *   Eğer birden fazla geçerli yerel ayar bulunursa, Windows API'sini (`DialogBox`, `ListBox_AddString` vb.) kullanarak kullanıcıya bir seçim penceresi (`IDD_SELECT_LOCALE` kaynak ID'li) gösterir.
    *   Kullanıcının seçimine veya tek bir geçerli yerel ayar varsa ona göre `MULTI_LOCALE_*` değişkenlerini ve `__SECURITY_KEY_STRING__`'i ayarlar.
*   **`LocaleService_IsLeadByte()` ve `LocaleService_StringCompareCI()`**:
    *   `LOCALE_SERVICE_WE_JAPAN` tanımlıysa, Shift-JIS karakter seti için özel `ShiftJIS_IsLeadByte` ve `ShiftJIS_StringCompareCI` fonksiyonlarını kullanır. (Bu fonksiyonların implementasyonları `../EterLocale/Japanese.h` içinde olabilir.)
    *   Diğer durumlarda, genel bir çok baytlı karakter kontrolü (`((unsigned char)chByte) & 0x80)`) ve standart `strnicmp` fonksiyonunu kullanır.

#### Oyunda Kullanım Amacı ve Senaryoları

Bu yerelleştirme servisleri, oyunun farklı coğrafi bölgelerdeki oyunculara ve farklı dil konuşan topluluklara hitap edebilmesi için hayati öneme sahiptir:

1.  **Başlangıçta Yerel Ayarın Belirlenmesi**: Oyun başladığında `LocaleService_LoadConfig` ve/veya `LocaleService_LoadGlobal` çağrılarak istemcinin hangi yerel ayar için çalışacağı belirlenir. Bu, doğru dil paketlerinin, yazı tiplerinin, arayüz düzenlerinin ve bölgesel özelliklerin yüklenmesini sağlar.
2.  **Özelliklerin Şartlı Etkinleştirilmesi**: `Locale_inc.h` içindeki makrolar sayesinde, oyunun belirli bir sürümünde (örneğin, Avrupa sunucuları için) bazı özellikler aktifken (örneğin, belirli bir kostüm sistemi), başka bir sürümde (örneğin, Japonya sunucuları için) bu özellikler devre dışı olabilir. Bu, farklı yayıncıların veya bölgelerin gereksinimlerine göre esneklik sağlar.
3.  **Dil ve Metin Gösterimi**: `LocaleService_GetCodePage` doğru kod sayfasını belirleyerek metinlerin doğru görüntülenmesini sağlar. `CPythonApplication::SetDefaultFont` gibi fonksiyonlar, yerel ayara uygun yazı tipini yüklemek için bu servislerden gelen bilgiyi kullanabilir.
4.  **Ağ İletişimi ve Güvenlik**: `LocaleService_GetSecurityKey` tarafından döndürülen anahtar, istemci ile sunucu arasındaki iletişimin şifrelenmesinde veya doğrulanmasında kullanılabilir, böylece farklı servis sağlayıcıları kendi güvenlik mekanizmalarını uygulayabilir.
5.  **Oyun İçi Verilerin Farklılaştırılması**: Lonca seviye atlama deneyimleri veya yetenek güçleri gibi bazı oyun içi dengeler, `LocaleService_GetLastExp` ve `LocaleService_GetSkillPower` aracılığıyla yerel ayara göre farklılık gösterebilir. Bu, farklı oyuncu kitlelerine veya pazar beklentilerine göre ince ayar yapılmasına olanak tanır.
6.  **Kullanıcı Arayüzü Uyarlaması**: `LocaleService_GetLocaID` veya `LocaleService_GetLocaleName` gibi fonksiyonlardan gelen bilgi, Python UI betiklerinin doğru dil dosyalarını (`.py` veya `.sub` dosyaları içinde string tabloları) yüklemesini ve arayüzü yerel dile göre göstermesini sağlar.

Özetle, bu üç dosya, Metin2 istemcisinin küresel bir oyuncu kitlesine hizmet verebilmesi için gerekli olan yerelleştirme ve yapılandırma esnekliğini sağlayan temel bileşenlerdir.

### `MarkImage.h` ve `MarkImage.cpp` (Lonca Sembolü Yönetimi)

Bu dosyalar, Metin2 istemcisinde lonca sembollerinin (guild marks) oluşturulması, saklanması, yüklenmesi ve ağ üzerinden verimli bir şekilde senkronize edilmesi için gerekli olan `CGuildMarkImage` sınıfını ve ilgili yardımcı yapıları (`SGuildMark`, `SGuildMarkBlock`) tanımlar ve uygular. Temel olarak, tüm lonca sembollerini içeren büyük bir resim dosyasını (`mark.tga`) yönetir, bu resmi daha küçük, sıkıştırılabilir bloklara ayırır ve bu blokların güncellenmesini sağlar.

#### Dosyaların Genel Amaçları

*   **`MarkImage.h`**: `CGuildMarkImage` sınıfının, bireysel lonca sembollerini temsil eden `SGuildMark` yapısının ve sıkıştırılmış sembol gruplarını (blokları) temsil eden `SGuildMarkBlock` yapısının tanımlarını içerir. Bu başlık dosyası, lonca sembolü resminin boyutları, blok yapısı ve bireysel sembollerin boyutları gibi sabitleri de tanımlar.
*   **`MarkImage.cpp`**: `CGuildMarkImage` sınıfının ve yardımcı yapıların metotlarının implementasyonlarını içerir. Görüntü işleme için DevIL (Developer's Image Library) kütüphanesini, sıkıştırma için LZO (Lempel-Ziv-Oberhumer) kütüphanesini ve veri bütünlüğü için CRC32 algoritmalarını kullanır.

#### Temel Yapılar ve Sınıflar (`MarkImage.h`)

1.  **`Pixel`**:
    *   `typedef unsigned long Pixel;`
    *   Genellikle 32-bit bir renk değerini (örneğin, BGRA formatında) temsil eder.

2.  **`struct SGuildMark`**: Tek bir lonca sembolünü tanımlar.
    *   **Sabitler**:
        *   `WIDTH = 16`: Sembol genişliği (piksel).
        *   `HEIGHT = 12`: Sembol yüksekliği (piksel).
        *   `SIZE = WIDTH * HEIGHT`: Toplam piksel sayısı (192).
    *   **Üyeler**:
        *   `Pixel m_apxBuf[SIZE]`: Sembolün ham piksel verilerini tutan dizi.
    *   **Metotlar**:
        *   `void Clear()`: Sembolü varsayılan bir duruma (genellikle tamamen siyah veya şeffaf) sıfırlar. Koddaki implementasyon 0xFF000000 değerini kullanır.
        *   `bool IsEmpty()`: Sembolün boş olup olmadığını (tüm pikselleri 0x00000000 ise) kontrol eder.

3.  **`struct SGuildMarkBlock`**: Lonca sembollerinin bir bloğunu tanımlar. Bu bloklar sıkıştırılarak saklanır ve transfer edilir.
    *   **Sabitler**:
        *   `MARK_PER_BLOCK_WIDTH = 4`: Bir bloktaki yatay sembol sayısı.
        *   `MARK_PER_BLOCK_HEIGHT = 4`: Bir bloktaki dikey sembol sayısı.
        *   `WIDTH = SGuildMark::WIDTH * MARK_PER_BLOCK_WIDTH` (64 piksel).
        *   `HEIGHT = SGuildMark::HEIGHT * MARK_PER_BLOCK_HEIGHT` (48 piksel).
        *   `SIZE = WIDTH * HEIGHT`: Bir bloktaki toplam piksel sayısı (3072).
        *   `MAX_COMP_SIZE`: Sıkıştırılmış blok verisi için maksimum tampon boyutu.
    *   **Üyeler**:
        *   `Pixel m_apxBuf[SIZE]`: Bloğun ham piksel verilerini tutan dizi (genellikle sadece geçici olarak sıkıştırma/açma sırasında kullanılır).
        *   `BYTE m_abCompBuf[MAX_COMP_SIZE]`: Bloğun LZO ile sıkıştırılmış verilerini tutan tampon.
        *   `size_t m_sizeCompBuf`: Sıkıştırılmış verinin gerçek boyutu.
        *   `DWORD m_crc`: Bloğun ham piksel verilerinin CRC32 özeti.
    *   **Metotlar**:
        *   `DWORD GetCRC() const`: Bloğun CRC değerini döndürür.
        *   `void CopyFrom(const BYTE* pbCompBuf, DWORD dwCompSize, DWORD crc)`: Sıkıştırılmış veriyi ve CRC'yi kopyalar.
        *   `void Compress(const Pixel* pxBuf)`: Verilen ham piksel verisini LZO1X-999 algoritması ile sıkıştırır, `m_abCompBuf` ve `m_sizeCompBuf`'u günceller ve `m_crc`'yi hesaplar.

4.  **`class CGuildMarkImage`**: Tüm lonca sembollerini içeren ana resmi (genellikle "mark.tga") yönetir.
    *   **Sabitler**:
        *   `WIDTH = 512`, `HEIGHT = 512`: Ana resmin boyutları.
        *   `BLOCK_ROW_COUNT`, `BLOCK_COL_COUNT`, `BLOCK_TOTAL_COUNT`: Ana resimdeki toplam blok sayısı.
        *   `MARK_ROW_COUNT`, `MARK_COL_COUNT`, `MARK_TOTAL_COUNT`: Ana resimdeki toplam bireysel sembol slotu sayısı (1280).
        *   `INVALID_MARK_POSITION = 0xffffffff`: Geçersiz bir sembol pozisyonunu belirtir.
    *   **Üyeler**:
        *   `SGuildMarkBlock m_aakBlock[BLOCK_ROW_COUNT][BLOCK_COL_COUNT]`: Ana resmi oluşturan sıkıştırılmış blokların 2D dizisi.
        *   `Pixel m_apxImage[WIDTH * HEIGHT * sizeof(Pixel)]`: Ana resmin ham piksel verilerini tutan dizi (kullanımı sınırlı görünüyor, DevIL doğrudan kendi tamponlarını kullanır).
        *   `ILuint m_uImg`: DevIL kütüphanesi tarafından kullanılan resim tanıtıcısı (handle).
    *   **Ana Metotlar (Detaylar `MarkImage.cpp`'de)**:
        *   `Create()`, `Destroy()`: DevIL resim nesnesini oluşturur ve siler.
        *   `Build(const char* c_szFileName)`: Belirtilen dosyaya varsayılan (boş) bir TGA resmi oluşturur.
        *   `Save(const char* c_szFileName)`: Mevcut resmi TGA formatında dosyaya kaydeder.
        *   `Load(const char* c_szFileName)`: TGA dosyasından resmi yükler. Eğer dosya bozuksa veya yoksa `Build()` ile yenisini oluşturur. Başarıyla yüklendikten sonra `BuildAllBlocks()` çağrılır.
        *   `PutData(UINT x, UINT y, UINT width, UINT height, void* data)`: Resmin belirli bir bölgesine piksel verisi yazar.
        *   `GetData(UINT x, UINT y, UINT width, UINT height, void* data)`: Resmin belirli bir bölgesinden piksel verisi okur.
        *   `SaveMark(DWORD posMark, BYTE* pbMarkImage)`: (Sunucu tarafı) Belirli bir pozisyondaki (`posMark`) lonca sembolünü günceller ve ilgili bloğu yeniden sıkıştırır.
        *   `DeleteMark(DWORD posMark)`: (Sunucu tarafı) Belirli bir pozisyondaki sembolü siler (boş veri ile üzerine yazar).
        *   `SaveBlockFromCompressedData(DWORD posBlock, const BYTE* pbComp, DWORD dwCompSize)`: (İstemci tarafı) Sunucudan gelen sıkıştırılmış bir sembol bloğunu alır, LZO ile açar, CRC'sini doğrular, resme yerleştirir ve `m_aakBlock`'u günceller.
        *   `GetEmptyPosition()`: Resimde boş bir sembol slotu arar.
        *   `GetBlockCRCList(DWORD* crcList)`: Tüm blokların CRC değerlerini bir listeye yazar.
        *   `GetDiffBlocks(const DWORD* crcList, std::map<BYTE, const SGuildMarkBlock*>& mapDiffBlocks)`: Verilen bir CRC listesi (genellikle istemciden gelir) ile kendi blok CRC'lerini karşılaştırır ve farklı olan blokları (sunucuda güncellenmiş olanlar) bir haritaya ekler.
        *   `void BuildAllBlocks()`: Ana resimdeki tüm blokları `GetData` ile okur, `SGuildMarkBlock::Compress` ile sıkıştırır ve `m_aakBlock` dizisini doldurur.

#### `MarkImage.cpp` Implementasyon Detayları

*   **DevIL Entegrasyonu**: Resim oluşturma, yükleme (TGA), kaydetme (TGA), piksel formatı dönüştürme (BGRA) ve ham piksel verilerine erişim için DevIL fonksiyonları (`ilGenImages`, `ilDeleteImages`, `ilBindImage`, `ilLoad`, `ilSave`, `ilTexImage`, `ilConvertImage`, `ilSetPixels`, `ilCopyPixels`, `ilEnable(IL_ORIGIN_SET)`, `ilOriginFunc(IL_ORIGIN_UPPER_LEFT)`) yoğun bir şekilde kullanılır. `IL_ORIGIN_UPPER_LEFT` ayarı, koordinat sisteminin sol üst köşe olmasını sağlar.
*   **LZO Sıkıştırması**:
    *   `SGuildMarkBlock::Compress`: `lzo1x_999_compress` fonksiyonu ile blok piksel verilerini sıkıştırır.
    *   `CGuildMarkImage::SaveBlockFromCompressedData`: `lzo1x_decompress_safe` fonksiyonu ile sıkıştırılmış blok verilerini açar.
    *   Sıkıştırma ve açma işlemleri için `LZOManager::Instance().GetWorkMemory()` (veya benzeri bir singleton) aracılığıyla geçici bir çalışma belleği alanı kullanılır.
*   **CRC32 Hesaplaması**:
    *   `SGuildMarkBlock::Compress` içinde sıkıştırılmadan önceki ham piksel verilerinin CRC32'si `GetCRC32()` fonksiyonu (muhtemelen `crc32.h` içinde tanımlı) ile hesaplanır ve `m_crc` üyesinde saklanır.
    *   `CGuildMarkImage::SaveBlockFromCompressedData` içinde, açılan verinin CRC'si yeniden hesaplanıp gelen CRC ile karşılaştırılmasa da, `SGuildMarkBlock::CopyFrom` metodu gelen CRC'yi doğrudan saklar.
*   **Blok ve Mark Yönetimi**:
    *   `SaveMark`: Sunucu tarafında, bir sembol güncellendiğinde, `PutData` ile ana resme yazılır. Ardından, bu sembolü içeren blok `GetData` ile okunur ve `SGuildMarkBlock::Compress` ile yeniden sıkıştırılarak `m_aakBlock` güncellenir.
    *   `GetDiffBlocks`: Bu fonksiyon, istemci ve sunucu arasındaki lonca sembollerini senkronize etmek için kritik bir rol oynar. İstemci, sahip olduğu blokların CRC listesini sunucuya gönderir. Sunucu, bu listeyi kendi blok CRC'leriyle karşılaştırır ve sadece farklı (yani güncellenmiş) olan blokları istemciye geri gönderir. Bu, tüm sembol resmini göndermek yerine sadece değişikliklerin transfer edilmesini sağlayarak bant genişliğinden tasarruf sağlar.
*   **Hata Yönetimi ve Loglama**: `sys_err` ve `sys_log` makroları (derleme zamanı tanımlarına göre `TraceError` veya benzeri fonksiyonlara yönlendirilir) hata durumlarını ve önemli adımları loglamak için kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

1.  **Lonca Sembolü Yükleme/Oluşturma**:
    *   İstemci başladığında, `mark.tga` dosyası `CGuildMarkImage::Load` ile yüklenir. Eğer dosya yoksa veya bozuksa, boş bir `mark.tga` oluşturulur.
    *   `BuildAllBlocks` çağrılarak tüm bloklar sıkıştırılır ve CRC'leri hesaplanır.
2.  **Lonca Sembolü Senkronizasyonu (İstemci-Sunucu)**:
    *   İstemci, sunucuya bağlanırken veya belirli aralıklarla, `GetBlockCRCList` ile elde ettiği CRC listesini sunucuya gönderir.
    *   Sunucu (`CGuildMarkImage`'in bir örneğini kullanarak), `GetDiffBlocks` ile istemcinin CRC listesini kendi güncel CRC listesiyle karşılaştırır.
    *   Sunucu, farklı olan blokların sıkıştırılmış verilerini (`SGuildMarkBlock::m_abCompBuf` ve `m_sizeCompBuf`) ve CRC'lerini istemciye gönderir.
    *   İstemci, aldığı her farklı blok için `SaveBlockFromCompressedData` fonksiyonunu çağırarak yerel `mark.tga` resmini ve `m_aakBlock` yapısını günceller.
3.  **Lonca Sembolü Gösterimi**:
    *   Oyun içinde bir lonca sembolü gösterileceği zaman (örneğin, karakterlerin üzerinde, lonca penceresinde), ilgili sembolün koordinatları `mark.tga` dosyasından (veya hafızadaki `CGuildMarkImage`'nin DevIL ile yönettiği ham resimden) okunarak ekrana çizilir. İstemci tarafı genellikle doğrudan sıkıştırılmış bloklarla değil, DevIL aracılığıyla erişilen ve açılmış piksel verileriyle çalışır.
4.  **Yeni Lonca Sembolü Ekleme/Değiştirme (Sunucu Tarafı)**:
    *   Bir oyuncu lonca sembolünü değiştirdiğinde veya yeni bir sembol yüklendiğinde, sunucu `SaveMark` fonksiyonunu kullanarak ilgili sembolü `mark.tga` dosyasına (veya hafızadaki kopyasına) işler ve etkilenen bloğu günceller. Bu değişiklik bir sonraki senkronizasyonda istemcilere yansıtılır.

Bu sistem, çok sayıda küçük resim (lonca sembolleri) yerine tek bir büyük resim dosyası kullanarak dosya yönetimi ve I/O işlemlerini basitleştirir. Blok tabanlı sıkıştırma ve CRC kontrolü ile de ağ üzerinden verimli ve güvenilir bir senkronizasyon mekanizması sunar.

### `MarkManager.h` ve `MarkManager.cpp` (Lonca Mark ve Sembol Yöneticisi)

`CGuildMarkManager`, `CGuildMarkImage` sınıfını kullanarak birden fazla lonca "mark" resim dosyasını (örneğin, `mark/guild_0.tga`, `mark/guild_1.tga`) yöneten, lonca kimliklerini bu resimlerdeki belirli sembol pozisyonlarına eşleyen ve bu eşlemeleri bir indeks dosyasında saklayan bir singleton sınıfıdır. Ayrıca, "mark" sisteminden ayrı olarak, lonca kimlikleriyle ilişkilendirilmiş daha genel "sembol" verilerini (ham bayt dizileri ve CRC'leri) de yönetir.

#### Dosyaların Genel Amaçları

*   **`MarkManager.h`**: `CGuildMarkManager` sınıfının arayüzünü tanımlar. Bu, birden fazla `CGuildMarkImage` örneğini yönetme, lonca ID'lerini global mark ID'lerine eşleme, bu eşlemeleri yükleme/kaydetme ve istemci-sunucu senaryolarında mark bloklarının transferini koordine etme yeteneklerini içerir. Ayrıca `TGuildSymbol` adlı bir yapıyı ve bununla ilgili fonksiyonları tanımlayarak ikinci bir sembol yönetim sistemi sunar.
*   **`MarkManager.cpp`**: `CGuildMarkManager` sınıfının metotlarının implementasyonlarını sağlar. Dosya I/O işlemleri (indeks ve sembol dosyaları için), `CGuildMarkImage` nesnelerinin yaşam döngüsü yönetimi ve mark/sembol verilerinin mantıksal organizasyonunu içerir.

#### Temel Kavramlar ve Yapılar

*   **Mark Sistemi**:
    *   **Global Mark ID**: Her lonca sembolüne atanan benzersiz bir kimliktir. Bu ID, sembolün hangi `CGuildMarkImage` dosyasında (`imgIdx = markID / CGuildMarkImage::MARK_TOTAL_COUNT`) ve o dosyanın içinde hangi sırada (`markPosInImage = markID % CGuildMarkImage::MARK_TOTAL_COUNT`) olduğunu belirtir.
    *   **Resim Dosyaları**: `MAX_IMAGE_COUNT` (varsayılan 5) adet `CGuildMarkImage` nesnesi tarafından yönetilen TGA dosyaları (örneğin, `mark/prefix_0.tga`). Her biri çok sayıda 16x12 piksellik lonca markı içerir.
    *   **İndeks Dosyası**: `mark/[prefix]_index` formatında bir metin dosyasıdır. Her satırda bir `guildID` ve ona karşılık gelen global `markID` bulunur. Bu, hangi loncanın hangi sembolü kullandığını takip eder.
*   **Sembol Sistemi (`TGuildSymbol`)**:
    *   `struct TGuildSymbol { DWORD crc; std::vector<BYTE> raw; };`
    *   Bu, `CGuildMarkImage`'ın 16x12 piksellik marklarından farklı, daha genel amaçlı bir sembol türüdür. Her sembol, bir CRC değeri ve ham bayt verisi içerir. `m_mapSymbol` içinde lonca ID'leri ile eşleştirilirler. Bu sembollerin ne için kullanıldığı (örneğin, daha büyük/farklı formatta lonca amblemleri veya başka bir amaçla) koddaki kullanıma bağlıdır. `UploadSymbol` fonksiyonu, bu sembollerin ağ üzerinden alınabildiğini gösterir.

#### Sınıf Üyeleri ve Fonksiyonları (`CGuildMarkManager`)

*   **Singleton Erişimi**: `CGuildMarkManager::Instance()` ile erişilir.
*   **Ana Üye Değişkenler**:
    *   `std::map<DWORD, CGuildMarkImage*> m_mapIdx_Image`: Resim indeksi -> `CGuildMarkImage` işaretçisi.
    *   `std::map<DWORD, DWORD> m_mapGID_MarkID`: Lonca ID -> Global Mark ID.
    *   `std::set<DWORD> m_setFreeMarkID`: Kullanılmayan global Mark ID'lerinin kümesi.
    *   `std::string m_pathPrefix`: Mark dosyaları için önek (örn: "guild").
    *   `std::map<DWORD, TGuildSymbol> m_mapSymbol`: Lonca ID -> `TGuildSymbol`.

*   **Mark Yönetimi Fonksiyonları**:
    *   `void SetMarkPathPrefix(const char* prefix)`: Mark dosyaları için kullanılacak öneki ayarlar.
    *   `bool LoadMarkIndex()`: `mark/[prefix]_index` dosyasını okur, `m_mapGID_MarkID`'yi doldurur ve `m_setFreeMarkID`'yi günceller. Ardından `LoadMarkImages()` çağırır.
    *   `bool SaveMarkIndex()`: `m_mapGID_MarkID` içeriğini indeks dosyasına yazar.
    *   `void LoadMarkImages()`: `m_mapGID_MarkID`'de listelenen ve kullanılan tüm `CGuildMarkImage` örneklerini (`mark_X.tga` dosyalarını) `__GetImage()` aracılığıyla yükler.
    *   `void SaveMarkImage(DWORD imgIdx)`: Belirli bir indeksteki `CGuildMarkImage`'ı (`mark_[imgIdx].tga`) diske kaydeder.
    *   `bool GetMarkImageFilename(DWORD imgIdx, std::string& path) const`: Verilen resim indeksi için tam dosya yolunu oluşturur.
    *   `DWORD GetMarkID(DWORD guildID)`: Bir loncanın global mark ID'sini döndürür.
    *   `CGuildMarkImage* __GetImage(DWORD imgIdx)` (Özel): Verilen indeksteki `CGuildMarkImage` nesnesini döndürür. Eğer yüklenmemişse, dosyadan yükler ve `m_mapIdx_Image`'e ekler.
    *   `DWORD __AllocMarkID(DWORD guildID)` (Özel): Boş bir mark ID bulur, loncaya atar, `m_setFreeMarkID`'den çıkarır ve ilgili `CGuildMarkImage`'ın yüklenmesini sağlar.

*   **Sunucu Tarafı Mark Fonksiyonları**:
    *   `DWORD SaveMark(DWORD guildID, BYTE* pbMarkImage)`:
        1.  Lonca için mevcut bir mark ID yoksa `__AllocMarkID` ile yeni bir tane alır.
        2.  Global mark ID'den resim indeksi (`imgIdx`) ve o resim içindeki pozisyonu (`markPosInImage`) hesaplar.
        3.  `__GetImage(imgIdx)->SaveMark(markPosInImage, pbMarkImage)` ile mark verisini ilgili `CGuildMarkImage`'a kaydeder.
        4.  `SaveMarkImage(imgIdx)` ile değişiklik yapılan TGA dosyasını diske kaydeder.
        5.  `SaveMarkIndex()` ile indeks dosyasını günceller.
    *   `void DeleteMark(DWORD guildID)`:
        1.  Loncanın mark ID'sini bulur.
        2.  İlgili `CGuildMarkImage` üzerinde `DeleteMark()` çağırır.
        3.  Mark ID'yi `m_mapGID_MarkID`'den siler ve `m_setFreeMarkID`'ye geri ekler.
        4.  İlgili TGA dosyasını ve indeks dosyasını kaydeder.
    *   `void GetDiffBlocks(DWORD imgIdx, const DWORD* crcList, std::map<BYTE, const SGuildMarkBlock*>& mapDiffBlocks)`: Belirli bir resim dosyası (`imgIdx`) için, istemciden gelen `crcList` ile sunucudaki blok CRC'lerini karşılaştırarak farklı olan blokları `mapDiffBlocks`'a doldurur. Bu, `CGuildMarkImage::GetDiffBlocks`'a delege edilir.
    *   `void CopyMarkIdx(char* pcBuf) const`: Tüm (lonca ID, mark ID) çiftlerini bir tampona kopyalar (muhtemelen istemciye göndermek için).

*   **İstemci Tarafı Mark Fonksiyonları**:
    *   `bool SaveBlockFromCompressedData(DWORD imgIdx, DWORD idBlock, const BYTE* pbBlock, DWORD dwSize)`: Sunucudan gelen sıkıştırılmış bir mark bloğunu alır ve ilgili `CGuildMarkImage`'a (`__GetImage(imgIdx)->SaveBlockFromCompressedData()`) kaydedilmek üzere iletir.
    *   `bool GetBlockCRCList(DWORD imgIdx, DWORD* crcList)`: Belirli bir resim dosyası (`imgIdx`) için tüm blokların CRC listesini `CGuildMarkImage::GetBlockCRCList` aracılığıyla alır.

*   **Sembol Yönetimi Fonksiyonları (`TGuildSymbol` için)**:
    *   `const TGuildSymbol* GetGuildSymbol(DWORD GID)`: Belirli bir loncanın sembolünü döndürür.
    *   `bool LoadSymbol(const char* filename)`: İkili bir sembol dosyasından (`filename`) tüm lonca sembollerini (`m_mapSymbol`) yükler. Dosya formatı: `[sembol_sayısı (DWORD)][loncaID (DWORD)][veri_boyutu (DWORD)][veri (BYTE[])]...` şeklindedir. Yükleme sırasında CRC hesaplanır.
    *   `void SaveSymbol(const char* filename)`: `m_mapSymbol` içeriğini aynı ikili formatta dosyaya kaydeder.
    *   `void UploadSymbol(DWORD guildID, int iSize, const BYTE* pbyData)`: Belirli bir lonca için sembol verisini alır, `m_mapSymbol`'a ekler/günceller ve CRC'sini hesaplar.

#### Implementasyon Detayları (`MarkManager.cpp`)

*   **Başlatma**: Kurucu metot, `mark` klasörünün var olduğundan emin olur ve `m_setFreeMarkID` kümesini olası tüm boş mark ID'leriyle doldurur.
*   **Dosya İşlemleri**: Mark indeks dosyası (`%s_index`) metin tabanlı olarak okunup yazılırken, sembol dosyası (`symbol.bin` gibi) ikili formatta işlenir.
*   **Dinamik Yükleme**: `CGuildMarkImage` nesneleri (`mark_X.tga` dosyaları) sadece ihtiyaç duyulduklarında (`__GetImage` içinde) yüklenir. `LoadMarkImages` fonksiyonu, `LoadMarkIndex` sonrası gerekli tüm resimlerin yüklenmesini tetikler.
*   **ID Yönetimi**: `__AllocMarkID`, `m_setFreeMarkID` kümesini kullanarak verimli bir şekilde boş bir global mark ID'si bulur.
*   **Hata ve Loglama**: `sys_err` ve `sys_log` makroları (derleyiciye bağlı olarak `TraceError` vb. ile tanımlanır) işlemler sırasında hata ayıklama ve bilgi mesajları için kullanılır.

#### Oyunda Kullanım Amacı ve Senaryoları

1.  **Sunucu Başlangıcı**:
    *   `SetMarkPathPrefix` ile önek ayarlanır.
    *   `LoadMarkIndex` çağrılarak hangi loncanın hangi markı kullandığı bilgisi yüklenir.
    *   `LoadMarkImages` ile ilgili tüm `mark_X.tga` dosyaları belleğe (yani `CGuildMarkImage` nesnelerine) yüklenir.
    *   `LoadSymbol` ile `symbol.bin` (veya benzeri) dosyadan `TGuildSymbol` verileri yüklenir.
2.  **Yeni Lonca Sembolü Yükleme/Değiştirme (Sunucu Tarafı)**:
    *   Bir oyuncu loncası için 16x12'lik bir mark yüklediğinde, sunucu `SaveMark` fonksiyonunu çağırır. Bu fonksiyon uygun bir boş global mark ID bulur (veya mevcut olanı kullanır), mark verisini ilgili `CGuildMarkImage` nesnesine ve ardından diske (`mark_X.tga`) kaydeder. Son olarak `mark_index` dosyasını günceller.
    *   Eğer loncalar `TGuildSymbol` sistemini kullanıyorsa, `UploadSymbol` ile gelen veri `m_mapSymbol`'da saklanır ve periyodik olarak `SaveSymbol` ile diske yazılır.
3.  **Lonca Sembolü Silme (Sunucu Tarafı)**:
    *   `DeleteMark` çağrılır, ilgili `CGuildMarkImage` üzerinde mark silinir, TGA ve indeks dosyaları güncellenir, global mark ID tekrar boşa çıkar.
4.  **İstemci Senkronizasyonu (Mark Sistemi)**:
    *   İstemci, sunucuya bağlandığında veya periyodik olarak, her bir `mark_X.tga` dosyası için (0'dan `MAX_IMAGE_COUNT-1`'e kadar) `GetBlockCRCList` (istemci tarafında) ile kendi blok CRC listesini oluşturur.
    *   Bu CRC listelerini sunucuya gönderir.
    *   Sunucu, her bir `imgIdx` için `GetDiffBlocks` kullanarak istemcinin CRC listesiyle kendi listesini karşılaştırır ve sadece farklı olan (güncellenmiş) blokları sıkıştırılmış veri olarak istemciye gönderir.
    *   İstemci, aldığı bu blokları `SaveBlockFromCompressedData` ile yerel `CGuildMarkImage` nesnesine işler, bu da yerel `mark_X.tga` resmini günceller.
5.  **İstemci Sembol Alımı (`TGuildSymbol`)**:
    *   İstemci, bir loncanın `TGuildSymbol`'unu göstermesi gerektiğinde, sunucudan bu sembolü talep edebilir. Sunucu `GetGuildSymbol` ile bu veriyi bulup istemciye gönderebilir (bu transfer mekanizması bu dosyalarda belirtilmemiştir, ancak `UploadSymbol` varlığına bakarak benzer bir indirme mekanizması olduğu varsayılabilir).

`CGuildMarkManager`, birden fazla büyük resim dosyasını ve bu dosyalardaki binlerce küçük sembolü verimli bir şekilde yöneterek, lonca sembollerinin oyun içinde gösterilmesini ve güncellenmesini sağlar. CRC tabanlı fark senkronizasyonu, ağ trafiğini minimize eder. Ayrı `TGuildSymbol` sistemi ise farklı ihtiyaçlar için ek bir sembol yönetim katmanı sunar.

---

*(Bu bölümdeki içerikler tamamlanmıştır. Devamı için [UserInterface Referans Kılavuzu - Bölüm 5](./client_UserInterface_Referans_Part5.md) ve [UserInterface Referans Kılavuzu - Bölüm 6](./client_UserInterface_Referans_Part6.md) dosyalarına bakınız.)*