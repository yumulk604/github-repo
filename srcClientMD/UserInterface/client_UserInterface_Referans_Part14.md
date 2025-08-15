# UserInterface Referans Kılavuzu - Bölüm 14

Bu belge, Metin2 istemcisinin `UserInterface` modülünün belgelenmesine devam etmektedir.

## İçindekiler

*   [`PythonPrivateShopManager.cpp` (Özel Pazar Örnek Yöneticisi)](#pythonprivateshopmanagercpp-özel-pazar-örnek-yöneticisi)
*   [`PythonPrivateShopModule.cpp` (Özel Pazar Python Modülü)](#pythonprivateshopmodulecpp-özel-pazar-python-modülü)
*   [`PythonProfilerModule.cpp` (Performans Profilleyici Python Modülü)](#pythonprofilermodulecpp-performans-profilleyici-python-modülü)
*   [`PythonQuest.h` (Görev Sistemi Başlık Dosyası)](#pythonquesth-görev-sistemi-başlık-dosyası)
*   [`PythonQuest.cpp` (Görev Sistemi Uygulaması ve Python Modülü)](#pythonquestcpp-görev-sistemi-uygulaması-ve-python-modülü)
*   [`PythonRenderTargetModule.cpp` (Render Hedefi Python Modülü)](#pythonrendertargetmodulecpp-render-hedefi-python-modülü)
*   [`PythonrootlibManager.h` (Rootlib Yönetici Başlık Dosyası)](#pythonrootlibmanagerh-rootlib-yönetici-başlık-dosyası)
*   [`PythonrootlibManager.cpp` (Rootlib Yöneticisi ve Python Modülü)](#pythonrootlibmanagercpp-rootlib-yöneticisi-ve-python-modülü)
*   [`PythonSafeBox.h` (Depo Sistemi Başlık Dosyası)](#pythonsafeboxh-depo-sistemi-başlık-dosyası)
*   [`PythonSafeBox.cpp` (Depo Sistemi Uygulaması ve Python Modülü)](#pythonsafeboxcpp-depo-sistemi-uygulaması-ve-python-modülü)
*   [`PythonShop.h` (Dükkan ve Pazar Sistemi Başlık Dosyası)](#pythonshoph-dükkan-ve-pazar-sistemi-başlık-dosyası)
*   [`PythonShop.cpp` (Dükkan ve Pazar Sistemi Uygulaması ve Python Modülü)](#pythonshopcpp-dükkan-ve-pazar-sistemi-uygulaması-ve-python-modülü)

---

### `PythonPrivateShopManager.cpp` (Özel Pazar Örnek Yöneticisi)

**Dosyanın Amacı ve İşlevselliği**

`PythonPrivateShopManager.cpp`, `CPythonPrivateShop` sınıfının özel pazar (tezgah) örneklerinin (`TPrivateShopInstance`) yönetimiyle ilgili fonksiyonlarını içerir. Bu dosya, oyun dünyasında görünen pazar tezgahlarının oluşturulması, silinmesi, güncellenmesi, çizdirilmesi (render edilmesi) ve oyuncuyla etkileşimlerinin (seçim, mesafe kontrolü) yönetilmesinden sorumludur. Ayrıca pazar dekorasyon dosyalarının yüklenmesi de bu dosyada gerçekleştirilir.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **Pazar Örneği Yönetimi (`TPrivateShopInstance`)**:
    *   `CreatePrivateShopInstance()`: Yeni bir pazar tezgahı örneği oluşturur.
        *   Verilen sanal ID (`dwVirtualID`) ile daha önce bir örnek olup olmadığını kontrol eder, varsa eskisini siler.
        *   `std::make_unique<TPrivateShopInstance>()` ile yeni bir pazar örneği oluşturur.
        *   Global koordinatları yerel koordinatlara çevirir ve zemin yüksekliğini ayarlar.
        *   Pazarın sanal ID'sini ve sahibinin adını ayarlar.
        *   `pPrivateShopInstance->LoadRace()`: Pazar modelini (belirtilen `dwVirtualNumber` ile) ve ilişkili hareket/efekt verilerini yükler. Bu fonksiyon içinde:
            *   `CRaceManager`'dan ilgili ırk verisi alınır.
            *   Yeni bir `CGraphicThingInstance` oluşturulur.
            *   Model dosyası (`GetBaseModelFileName()`) ve hareket verileri (`RegisterMotionThing`) kaydedilir. Genellikle sadece "WAIT" animasyonu kullanılır.
            *   Irk verisinde tanımlı olan başlangıç efektleri (`NRaceData::ATTACHING_DATA_TYPE_EFFECT`) pazar modeline eklenir (`AttachEffectByName`).
            *   Model gösterilir, güncellenir ve deforme edilir.
        *   Başarılı olursa, oluşturulan pazar örneği `m_map_privateShopInstance` (sanal ID -> pazar örneği unique_ptr'ı) haritasına eklenir.
        *   `CPythonTextTail::Instance().RegisterPrivateShopTextTail()`: Pazar için metin kuyruğu (başlık) kaydeder.
    *   `GetPrivateShopInstance()`: Verilen sanal ID'ye sahip pazar örneğini döndürür.
    *   `GetPrivateShopInstanceVID()`: Pazar sahibinin adına göre pazarın sanal ID'sini bulur.
    *   `DeletePrivateShopInstance()`: Bir pazar örneğini siler. Metin kuyruğunu siler, arama sonuçlarından seçiliyse kaldırır ve grafik örneğini temizler.
    *   `Destroy()`: Oyundan çıkarken veya harita değiştirirken tüm pazar örneklerini ve ilgili verileri temizler.

*   **Pazar Seçimi ve Etkileşim**:
    *   `GetPickedInstanceVID()`: Fare ile üzerine gelinen (seçilen) pazarın sanal ID'sini alır.
    *   `GetPrivateShopPosition()`: Bir pazarın oyun dünyasındaki koordinatlarını döndürür.
    *   `SelectSearchPrivateShop()`: Pazar arama sonucundan seçilen bir pazar için:
        *   Minimap üzerinde bir işaretçi (`CPythonMiniMap::TYPE_TARGET`) ekler.
        *   Pazar modeline bir seçim efekti (`c_szEffectName`) ekler (`AttachSelectEffect`).
        *   Pazarı `m_map_selectedPrivateShop` haritasına ekler.
    *   `UnselectSearchPrivateShop()`: Pazar arama sonucu seçimini kaldırır, minimap işaretçisini ve seçim efektini siler.
    *   `ClearSelectedSearchPrivateShop()`: Tüm seçili pazar arama sonuçlarını temizler.

*   **Çizim (Render) ve Güncelleme Döngüsü**:
    *   `Render()`: Görünür olan tüm pazar örneklerini çizer (`pInstance->Render()`).
    *   `Deform()`: Görünür olan tüm pazar örneklerini deforme eder (`pInstance->Deform()`).
    *   `Update()`: Tüm pazar örnekleri için güncelleme mantığını çalıştırır:
        *   **Mesafe Kontrolü (`ENABLE_PRIVATE_SHOP_LIMITED_DISTANCE_RENDERING`)**: Eğer bu makro aktifse, pazarın oyuncuya olan mesafesi hesaplanır. Belirli bir `MAX_VIEW_DISTANCE` (veya `CPythonSystem`'den alınan değer) dışındaysa pazar gizlenir ve efektleri durdurulur; menzil içindeyse gösterilir.
        *   Görünür durumdaysa, pazarın grafik örneği (`pInstance->Update()`) ve bağlı efektleri (`pPrivateShopInstance->UpdateAttachingEffect()`) güncellenir.
        *   Fare ile bir pazarın üzerine gelinip gelinmediği kontrol edilir (`pInstance->Intersect()`). Eğer bir kesişme varsa, `SetSelectedInstance()` ile o pazar seçili olarak ayarlanır. Kesişme yoksa ve önceden seçili bir pazar varsa, seçim kaldırılır.

*   **Efekt Yönetimi (`CPythonPrivateShop::SPrivateShopInstance` içinde)**:
    *   `AttachEffectByName()`, `AttachEffectByID()`: Pazar modeline isim veya ID ile efekt ekler. Efektler `m_list_attachingEffect` listesinde tutulur.
    *   `DetachEffect()`: Belirli bir efekti kaldırır.
    *   `AttachSelectEffect()`, `DetachSelectEffect()`: Pazar arama sonuçlarında seçilen pazarlar için özel bir seçim efektini ekler/kaldırır.
    *   `ShowAllAttachingEffect()`, `HideAllAttachingEffect()`: Pazarla ilişkili tüm efektleri gösterir/gizler.
    *   `ClearAttachingEffect()`: Tüm bağlı efektleri temizler.
    *   `UpdateAttachingEffect()`: Bağlı efektlerin pozisyonlarını ve durumlarını günceller.

*   **Dekorasyon Yükleme**:
    *   `LoadDecoration()`: Belirtilen dosyadan (`locale/.../privateshop_deco.txt` gibi) pazar görünüm (`DECO_TYPE_APPEARANCE`) ve başlık (`DECO_TYPE_TITLE`) dekorasyonlarını yükler.
        *   Dosya `CEterPackManager` ile açılır.
        *   `CMemoryTextFileLoader` ile satır satır okunur.
        *   Her satır türüne göre ayrıştırılır ve ilgili dekorasyon verisi (`TAppearanceDeco` veya `TTitleDeco`) `m_vec_appearanceDeco` veya `m_vec_titleDeco` vektörlerine eklenir.
    *   `GetAppearanceDeco()`, `GetTitleDeco()`: Yüklenmiş dekorasyon bilgilerini indekse göre döndürür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyun dünyasında gördüğünüz tüm özel pazar tezgahlarının görsel yönlerini ve temel etkileşimlerini yönetir. Sunucudan bir pazar bilgisi geldiğinde (`GC_PRIVATE_SHOP_APPEAR`), `CreatePrivateShopInstance` çağrılarak o pazarın modeli yüklenir, pozisyonu ayarlanır ve başlığı gösterilir. Oyuncu etrafta dolaşırken, `Update` fonksiyonu sayesinde bu pazarların oyuncuya olan mesafesi kontrol edilir (eğer ilgili sistem aktifse), fare ile üzerlerine gelindiğinde seçili hale gelirler ve `Render` ile ekranda çizilirler. Pazar arama sonuçlarından birine tıklandığında, o pazara özel bir efekt eklenir ve minimap'te işaretlenir. Pazar kapandığında (`GC_PRIVATE_SHOP_DISAPPEAR`) ise `DeletePrivateShopInstance` ile ilgili pazar dünyadan kaldırılır.

**Linter Hataları ve Olası Etkileri**

Dosyada belirtilen `CMemoryTextFileLoader` ve `CTokenVector` ile ilgili "incomplete type" hataları, genellikle bu sınıfların tanımlandığı başlık dosyalarının (`EterPack/EterPackManager.h` veya benzeri bir dosya içinde olabilirler) `PythonPrivateShopManager.cpp` dosyasına dahil edilmemiş olmasından veya ileriye dönük bildirim (forward declaration) yapılıp tam tanımın eksik olmasından kaynaklanır. Bu hatalar düzeltilmezse, `LoadDecoration` fonksiyonu doğru şekilde derlenemez ve çalışmaz, bu da pazar dekorasyonlarının yüklenememesine neden olur.

---

### `PythonPrivateShopModule.cpp` (Özel Pazar Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonPrivateShopModule.cpp`, `CPythonPrivateShop` sınıfının C++ fonksiyonlarını Python betiklerinin kullanımına sunmak amacıyla `privateShop` adında bir Python modülü tanımlar. Bu modül, Python'un özel pazar (tezgah) sistemiyle ilgili verilere erişmesini, pazarla ilgili eylemleri tetiklemesini (örneğin pazar kurma) ve pazar bilgilerini sorgulamasını sağlar. Oyunun kullanıcı arayüzü (UI) katmanının pazar sistemiyle etkileşim kurabilmesi için bir köprü görevi görür.

**Ana Çalışma Prensipleri ve Python C API Kullanımı**

*   **Python Modülü (`privateShop`) Tanımlama**:
    *   `initPrivateShop()` fonksiyonu, Python C API'sini kullanarak `privateShop` adında yeni bir Python modülü oluşturur (`Py_InitModule("privateShop", s_methods)`).
    *   `s_methods` dizisi, Python'dan çağrılabilecek fonksiyonların isimlerini (Python'da kullanılacak isim), C++ tarafındaki sarmalayıcı (wrapper) fonksiyonlarını ve çağrı tipini (genellikle `METH_VARARGS`) belirtir.

*   **CPythonPrivateShop Fonksiyonlarını Python'a Aktarma**:
    *   Her bir Python'a açılacak C++ fonksiyonu için bir sarmalayıcı C fonksiyonu yazılır (örneğin, `privateShopBuild`, `privateShopAddItemStock`).
    *   Bu sarmalayıcı fonksiyonlar, `PyObject* poSelf` (modül örneği, genellikle kullanılmaz) ve `PyObject* poArgs` (Python'dan gelen argümanlar) parametrelerini alır.
    *   `PyTuple_GetString()`, `PyTuple_GetUnsignedLong()`, `PyTuple_GetInteger()`, `PyTuple_GetLongLong()`, `PyTuple_GetByte()` gibi Python C API fonksiyonları kullanılarak `poArgs` içinden argümanlar alınır ve C++ türlerine dönüştürülür. Başarısız olursa `Py_BuildException()` ile hata döndürülür.
    *   Argümanlar alındıktan sonra, ilgili `CPythonPrivateShop::Instance().<FonksiyonAdi>()` çağrılır.
    *   Sonuçlar, `Py_BuildValue()` kullanılarak Python nesnelerine dönüştürülür ve Python tarafına döndürülür (örneğin, `Py_BuildValue("i", c_pItemData->dwVnum)` bir integer döndürür, `Py_BuildNone()` ise Python'da `None` döndürür).

*   **Sabitlerin Python'a Eklenmesi**:
    *   `PyModule_AddIntConstant(poModule, "PYTHON_SABIT_ADI", CPP_SABIT_DEGERI)` kullanılarak, C++ tarafında tanımlı olan çeşitli sabitler (enum değerleri, makrolar) Python modülüne eklenir. Bu, Python tarafında bu değerlere sayısal karşılıkları yerine anlamlı isimlerle erişilmesini sağlar (örneğin, `privateShop.STATE_OPEN`).

**Python'a Aktarılan Önemli Fonksiyon Grupları ve Kullanım Amaçları**

*   **Pazar Kurma ve Stok Yönetimi**:
    *   `privateShopBuild(szName, dwPolyVnum, bTitleType, bPageCount)`: Yeni bir pazar kurar.
    *   `privateShopClearPrivateShopStock()`: Pazar kurma stoğunu temizler.
    *   `privateShopAddItemStock(itemWindowType, itemSlotIndex, displaySlotIndex, price, cheque)`: Pazar kurma stoğuna eşya ekler.
    *   `privateShopDeleteItemStock(itemWindowType, itemSlotIndex)`: Stoktan eşya siler.
    *   `privateShopGetStockItemPrice()`, `privateShopGetStockChequeItemPrice()`: Stoktaki bir eşyanın fiyatını alır.
    *   `privateShopGetTotalStockGold()`, `privateShopGetTotalStockCheque()`: Stoktaki tüm eşyaların toplam değerini alır.

*   **Pazar Bilgileri Sorgulama (Kendi Pazarı ve Görüntülenen Pazar)**:
    *   `privateShopGetLocation()`: Kendi pazarının konumunu ve kanalını alır.
    *   `privateShopGetMyTitle()`, `privateShopGetTitle()`: Kendi pazarının veya görüntülenen pazarın başlığını alır.
    *   `privateShopGetGold()`, `privateShopGetCheque()`: Kendi pazar kasasındaki altın/çek miktarını alır.
    *   `privateShopGetPremiumTime()`: Kendi pazarının premium süresini alır.
    *   `privateShopGetMyState()`, `privateShopGetState()`: Kendi pazarının veya görüntülenen pazarın durumunu (açık, kapalı, düzenleniyor) alır.
    *   `privateShopGetMyPageCount()`, `privateShopGetPageCount()`: Sayfa sayılarını alır.
    *   `privateShopGetTotalGold()`, `privateShopGetTotalCheque()`: Kendi açık pazarındaki toplam değeri alır.
    *   `privateShopGetActiveVID()`: Görüntülenen pazarın sahibinin VID'ini alır.

*   **Pazardaki Eşya Bilgilerini Sorgulama**:
    *   `privateShopGetItemVnum(pos)`, `privateShopGetItemCount(pos)`: Pazardaki (kendi veya başkasının) belirtilen pozisyondaki eşyanın VNUM'unu ve sayısını alır.
    *   `privateShopGetItemPrice(pos)`, `privateShopGetChequeItemPrice(pos)`: Eşyanın altın ve çek fiyatını alır.
    *   `privateShopGetItemMetinSocket(pos, metinSlotIndex)`: Eşyanın metin taşı bilgisini alır.
    *   `privateShopGetItemAttribute(pos, attrSlotIndex)`: Eşyanın efsun bilgisini alır.
    *   `ENABLE_REFINE_ELEMENT`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_SET_ITEM`, `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10` gibi makrolarla aktifleşen sistemlere özel eşya bilgilerini (element, görünüm, set bonusu, rastgele efsun, min/max değer) alan fonksiyonlar.

*   **Pazar Arama Sonuçları Yönetimi**:
    *   `privateShopClearSearchResult()`: Arama sonuçlarını temizler.
    *   `privateShopGetSearchResultMaxCount()`: Toplam arama sonucu sayısını alır.
    *   `privateShopGetSearchResultPage()`, `privateShopSetSearchResultPage()`: Arama sonuçlarının sayfa numarasını alır/ayarlar.
    *   `privateShopGetSearchResult(pos)`: Belirli bir pozisyondaki arama sonucunun temel bilgilerini (VNUM, sahip adı, sayı, fiyat, durum) alır.
    *   Arama sonucundaki bir eşyanın detaylarını (`GetSearchItemVnum`, `GetSearchItemMetinSocket`, `GetSearchItemAttribute` vb.) alan fonksiyonlar.
    *   `privateShopGetSeachItemVID(pos)`: Arama sonucundaki eşyanın ait olduğu pazarın VID'ini alır.

*   **Pazar Örneği (Tezgah Modeli) Bilgileri**:
    *   `privateShopIsMainPlayerPrivateShop()`: Şu an görüntülenen pazarın ana oyuncuya ait olup olmadığını döndürür.
    *   `privateShopGetName(virtualID)`: Belirli bir VID'ye sahip pazarın sahibinin adını alır.
    *   `privateShopGetProjectPosition(virtualID, height)`: Pazarın dünyadaki pozisyonunu ekran koordinatlarına yansıtır (başlık vb. UI elemanları için).
    *   `privateShopGetMainCharacterDistance(virtualID)`: Oyuncunun belirli bir pazara olan mesafesini hesaplar.
    *   `privateShopCreatePrivateShopSearchPos(ownerName)`, `privateShopDeletePrivateShopSearchPos(ownerName=None)`: Pazar arama sonucundan bir pazar seçildiğinde minimap'te ve dünyada işaretçi oluşturur/kaldırır.

*   **Pazar Dekorasyonları**:
    *   `privateShopGetAppearanceDecoMaxCount()`, `privateShopGetTitleDecoMaxCount()`: Mevcut görünüm ve başlık dekorasyonlarının sayısını döndürür.
    *   `privateShopGetAppearanceDeco(pos)`, `privateShopGetTitleDeco(pos)`: Belirli bir indeksteki dekorasyonun bilgilerini (isim, vnum/path, renk) döndürür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, Python ile yazılmış kullanıcı arayüzü (UI) betiklerinin (örneğin, `uiPrivateShop.py`, `uiPrivateShopBuilder.py`, `uiPrivateShopSearch.py` vb.) C++ tarafındaki özel pazar sistemiyle etkileşim kurmasını sağlar. Oyuncu bir pazar kurma arayüzünü açtığında, eşya eklediğinde, fiyat belirlediğinde; başka bir oyuncunun pazarını açıp eşyalarına baktığında; pazar arama yaptığında veya pazar dekorasyonlarını seçtiğinde, UI betikleri bu modül aracılığıyla C++ fonksiyonlarını çağırarak gerekli verileri alır, işlemleri tetikler ve sonuçları kullanıcıya gösterir. Örneğin, `uiPrivateShopBuilder.py` içindeki bir "Pazarı Kur" butonu, `privateShop.BuildPrivateShop()` fonksiyonunu çağırabilir.

---

### `PythonProfilerModule.cpp` (Performans Profilleyici Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonProfilerModule.cpp`, istemci tarafında performans profili oluşturma işlevlerini Python betiklerine açmak için `profiler` adında bir Python modülü tanımlar. Bu modül, `Profiler::Push` ve `Profiler::Pop` fonksiyonlarını çağırarak kod bloklarının yürütme sürelerinin ölçülmesine olanak tanır. Bu, geliştiricilerin performans darboğazlarını tespit etmelerine yardımcı olmak için tasarlanmıştır.

**Ana Çalışma Prensipleri**

*   **Python Modülü (`profiler`) Tanımlama**:
    *   `initProfiler()` fonksiyonu, `profiler` adlı bir Python modülü oluşturur.
    *   Bu modüle `Push` ve `Pop` olmak üzere iki fonksiyon kaydedilir.
*   **Sarmalayıcı Fonksiyonlar**:
    *   `profilerPush(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir isim (`szName`) argümanı alır.
        *   Ancak, mevcut implementasyonda bu isim `Profiler::Push(szName)` çağrısında **kullanılmaz**. Muhtemelen eksik veya basitleştirilmiş bir implementasyondur. Normalde bu isim, profil kaydında ilgili kod bloğunu tanımlamak için kullanılır.
    *   `profilerPop(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir isim (`szName`) argümanı alır.
        *   `profilerPush` gibi, bu isim de `Profiler::Pop(szName)` çağrısında **kullanılmaz**.
*   **EterLib Entegrasyonu**:
    *   Bu modül, `../EterLib/Profiler.h` başlık dosyasında tanımlanan `CProfiler` sınıfının fonksiyonlarını kullanmayı hedefler. Ancak, sarmalayıcı fonksiyonların içindeki `Profiler::Push` ve `Profiler::Pop` çağrıları yorum satırı haline getirilmiş veya eksik bırakılmışsa, modül gerçek bir profil oluşturma işlevi görmeyecektir. Mevcut kodda bu çağrılar doğrudan `Profiler` sınıfı üzerinden yapılmıyor, sadece `Py_BuildNone()` döndürülüyor. Bu haliyle fonksiyonlar Python'dan çağrılabilir ancak gerçek bir profil verisi toplamazlar.

**Dosyanın Oyundaki Kullanım Amacı**

Teorik olarak bu modül, Python betikleriyle yazılmış oyun mantığının veya UI işlemlerinin belirli bölümlerinin ne kadar sürede çalıştığını ölçmek için kullanılır. Geliştirme aşamasında, bir Python fonksiyonunun veya döngüsünün başına `profiler.Push("Profil_Ismi")` ve sonuna `profiler.Pop("Profil_Ismi")` eklenerek o aralığın ne kadar sürdüğü kaydedilebilir. Bu bilgiler daha sonra analiz edilerek optimizasyon yapılacak alanlar belirlenebilir.

Ancak, sağlanan kodda `profilerPush` ve `profilerPop` fonksiyonlarının gövdeleri boştur (sadece argümanı okuyup `Py_BuildNone()` döndürüyorlar). Bu, ya profilleme özelliğinin geçici olarak devre dışı bırakıldığını ya da bu modülün tam olarak entegre edilmediğini gösterir. Fonksiyonların `CProfiler::Instance().Push(szName);` ve `CProfiler::Instance().Pop();` gibi çağrılar içermesi beklenirdi.

---

### `PythonQuest.h` (Görev Sistemi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonQuest.h`, Metin2 istemcisindeki görev (quest) sisteminin C++ tarafındaki temel tanımlamalarını ve arayüzünü içerir. Bu başlık dosyası, aktif görev örneklerini yöneten `CPythonQuest` singleton sınıfını ve her bir görev örneğinin verilerini tutan `SQuestInstance` yapısını deklare eder. Ayrıca, `ENABLE_QUEST_RENEWAL` makrosu etkinleştirildiğinde görev türleri (`EQuestType`), görev dize türleri (`EQuestStringType`) ve görev arayüz görünümleri (`EQuestSkin`) gibi ek numaralandırmaları tanımlar.

**Ana Yapılar ve Tanımlamalar**

*   **`SQuestInstance` (Yapı)**:
    *   Aktif bir görevin tüm bilgilerini içerir.
    *   `dwIndex`: Görevin benzersiz kimliği (ID).
    *   `strIconFileName`: Görev penceresinde veya izleyicide gösterilecek ikonun dosya adı.
    *   `strTitle`: Görevin başlığı.
    *   `strClockName`: Zamanlayıcılı görevler için gösterilecek saat adı (örn: "Kalan Süre:").
    *   `strCounterName`: Sayaçlı görevler için gösterilecek sayaç adı (örn: "Kesilen Yaratık:").
    *   `iClockValue`: Saniye cinsinden görevin süresi (negatifse saymaz, sıfır veya pozitifse geri sayar).
    *   `iCounterValue`: Görev sayacının mevcut değeri.
    *   `iStartTime`: Görevin başladığı veya saat değerinin en son güncellendiği zaman (saniye cinsinden).
    *   **`ENABLE_QUEST_RENEWAL` ile eklenen alanlar**:
        *   `bType (BYTE)`: Görevin türünü belirtir (ana görev, yan görev, etkinlik vb.).
        *   `bIsConfirmed (bool)`: Muhtemelen görevin oyuncu tarafından bir şekilde onaylandığını veya belirli bir aşamada olduğunu belirtir (kullanımı tam olarak bağlama göre değişir).

*   **`TQuestInstanceContainer` (Typedef)**:
    *   `std::vector<SQuestInstance>` için bir takma addır. Aktif tüm görev örnekleri bu vektörde saklanır.

*   **Numaralandırmalar (`ENABLE_QUEST_RENEWAL` ile)**:
    *   `EQuestStringType`: Görev bilgilerinin nasıl gösterileceğini belirler (normal metin, saat, sayaç).
    *   `EQuestType`: Görevlerin kategorilendirilmesini sağlar (Ana, Alt, Seviye Atlama, Etkinlik, Koleksiyon, Sistem, Parşömen, Günlük, Gizli).
    *   `EQuestSkin`: Görev arayüzünün farklı görünümlerini tanımlar (penceresiz, normal, parşömen, sinematik vb.).

*   **`CPythonQuest` (Singleton Sınıfı)**:
    *   Görev sistemiyle ilgili tüm istemci tarafı işlemlerini yöneten merkezi sınıftır.
    *   Görev örneklerini kaydetme, silme, sorgulama ve güncelleme fonksiyonlarını sağlar.
    *   Python modülüne (`quest`) açılacak fonksiyonlar için temel mantığı içerir.
    *   **Temel Fonksiyonlar**:
        *   `Clear()`: Tüm aktif görevleri temizler.
        *   `RegisterQuestInstance()`: Tam bir `SQuestInstance` yapısını alır ve kaydeder.
        *   `DeleteQuestInstance()`: Belirli bir ID'ye sahip görevi siler.
        *   `IsQuest()`: Bir görevin aktif olup olmadığını kontrol eder.
        *   `MakeQuest()`: Yeni bir görev örneği oluşturur (sadece ID ile, diğer bilgiler sonradan set edilir).
        *   `SetQuestTitle()`, `SetQuestClockName()`, `SetQuestCounterName()`, `SetQuestClockValue()`, `SetQuestCounterValue()`, `SetQuestIconFileName()`: Belirli bir görevin detaylarını günceller.
        *   `SetQuestIsConfirmed()` (`ENABLE_QUEST_RENEWAL` ile): Görevin onay durumunu ayarlar.
        *   `GetQuestCount()`: Aktif görev sayısını döndürür.
        *   `GetQuestButtonNoticeCount()` (`ENABLE_QUEST_RENEWAL` ile): Belirli bir türdeki görevlerden kaç tanesinin bildirim olarak gösterileceğini döndürür.
        *   `GetQuestInstancePtr()`: Belirli bir indekse (ya vektör indeksi ya da görev ID'si, `ENABLE_QUEST_RENEWAL` makrosuna göre değişir) göre görev örneğine bir işaretçi döndürür.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, istemcinin sunucudan gelen görev bilgilerini alıp saklaması ve yönetmesi için gerekli altyapıyı tanımlar. Sunucu bir görev başlattığında, güncellediğinde veya bitirdiğinde, bu sınıftaki fonksiyonlar çağrılarak istemcideki görev listesi güncel tutulur. `SQuestInstance` yapısı, görevlerin tüm bilgilerini (başlık, ikon, süre, sayaç vb.) taşır. `CPythonQuest` sınıfı ise bu örnekleri bir koleksiyonda tutar ve Python tarafındaki görev arayüzünün (`uiQuest.py` veya ilgili UI dosyaları) bu bilgilere erişip kullanıcıya sunmasını sağlar. Örneğin, görev penceresinde listelenen görevler, zamanlayıcılar veya görev hedefleri bu yapıdaki verilerden okunur.

---

### `PythonQuest.cpp` (Görev Sistemi Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonQuest.cpp`, `CPythonQuest` sınıfının ve `quest` Python modülünün implementasyonunu içerir. Bu dosya, istemcideki aktif görevlerin yönetilmesi (ekleme, silme, güncelleme, sorgulama) ve bu görev bilgilerinin Python betiklerinin kullanımına sunulmasıyla ilgili mantığı gerçekleştirir. Oyuncunun arayüzde gördüğü görev listesi, görev detayları, zamanlayıcılar ve sayaçlar bu dosyadaki C++ ve Python fonksiyonları aracılığıyla yönetilir.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **`CPythonQuest` Sınıfı Implementasyonu**:
    *   **Görev Örneği Yönetimi**:
        *   `RegisterQuestInstance()`: Verilen `SQuestInstance`'ı kaydeder. Önce aynı `dwIndex` ile var olan bir görevi siler, sonra yenisini `m_QuestInstanceContainer` vektörüne ekler ve başlangıç zamanını (`iStartTime`) ayarlar.
        *   `DeleteQuestInstance()`: Belirtilen `dwIndex`'e sahip görevi `m_QuestInstanceContainer`'dan bulur ve siler (`std::find_if` ve `FQuestInstanceCompare` yapısı kullanılır).
        *   `IsQuest()`: Bir görevin `m_QuestInstanceContainer`'da olup olmadığını kontrol eder.
        *   `MakeQuest()`: Yeni bir görev oluşturur. Verilen `dwIndex` ile eski görevi siler, boş bir `SQuestInstance` oluşturur, `dwIndex` ve `iStartTime`'ı ayarlar. `ENABLE_QUEST_RENEWAL` aktifse, görev tipini (`bType`) ve onay durumunu (`bIsConfirmed`) da ayarlar.
        *   `SetQuest...()` fonksiyonları (örn: `SetQuestTitle`, `SetQuestClockValue`): `__GetQuestInstancePtr` ile ilgili görevi bulup alanını günceller. `SetQuestClockValue` ayrıca `iStartTime`'ı da günceller.
        *   `GetQuestCount()`: `m_QuestInstanceContainer.size()`'ı döndürür.
        *   `GetQuestButtonNoticeCount()` (`ENABLE_QUEST_RENEWAL` ile): `m_QuestInstanceContainer` içinde belirtilen tipe (`bQuestType`) uyan veya tüm görevleri sayar.
        *   `GetQuestInstancePtr()`: `ENABLE_QUEST_RENEWAL` makrosuna bağlı olarak ya doğrudan `dwArrayIndex` ile vektörden ya da `dwQuestIndex` ile arama yaparak (`std::find_if`) görev örneğine işaretçi döndürür.
        *   `__GetQuestInstancePtr()`: Her zaman `dwQuestIndex` ile arama yaparak görev örneğine işaretçi döndürür.
    *   **Başlatma ve Temizleme**:
        *   `Clear()`: `m_QuestInstanceContainer`'ı temizler.
        *   `CPythonQuest()` (Kurucu): `__Initialize()` çağırır (bu fonksiyon genellikle boştur veya debug amaçlı kod içerir).
        *   `~CPythonQuest()` (Yıkıcı): `Clear()` çağırır.

*   **`quest` Python Modülü Fonksiyonları**:
    *   `questGetQuestCount(PyObject* poSelf, PyObject* poArgs)`:
        *   `CPythonQuest::Instance().GetQuestCount()`'ı çağırır ve sonucu Python integer olarak döndürür.
    *   `questGetQuestData(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir `iIndex` (görev örneğinin vektördeki indeksi veya `ENABLE_QUEST_RENEWAL` aktifse görev ID'si) alır.
        *   `CPythonQuest::Instance().GetQuestInstancePtr()` ile görev örneğini alır.
        *   Görev ikonunu (`pQuestInstance->strIconFileName`) yükler. İkon dosyası yoksa varsayılan bir ikon (`icon/scroll_open.tga`) kullanılır. İkonun yolu "d:/ymir work/ui/game/quest/questicon/" olarak sabitlenmiştir.
        *   `ENABLE_QUEST_RENEWAL` makrosuna bağlı olarak farklı formatlarda (`ibsisi` veya `sisi`) görev bilgilerini (tip, onay durumu, başlık, ikon nesnesi, sayaç adı, sayaç değeri) Python'a döndürür.
    *   `questGetQuestIndex(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir `iIndex` alır, görev örneğini bulur ve görevin asıl `dwIndex`'ini (ID) Python integer olarak döndürür.
    *   `questGetQuestLastTime(PyObject* poSelf, PyObject* poArgs)`:
        *   Python'dan bir `iIndex` alır, görev örneğini bulur.
        *   Eğer `pQuestInstance->iClockValue` (görev süresi) pozitifse, kalan süreyi `(pQuestInstance->iStartTime + pQuestInstance->iClockValue) - int(CTimer::Instance().GetCurrentSecond())` formülüyle hesaplar.
        *   Saat adını (`pQuestInstance->strClockName`) ve hesaplanan kalan süreyi Python tuple (`si`) olarak döndürür.
    *   `questClear(PyObject* poSelf, PyObject* poArgs)`:
        *   `CPythonQuest::Instance().Clear()`'ı çağırır ve `Py_BuildNone()` döndürür.
    *   `questGetQuestCounterData(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_QUEST_RENEWAL` ile):
        *   Şu anki implementasyonda sadece `Py_BuildNone()` döndürür, yani bir işlevi yoktur.
    *   `questGetQuestButtonNoticeCount(PyObject* poSelf, PyObject* poArgs)` (`ENABLE_QUEST_RENEWAL` ile):
        *   Python'dan bir görev tipi (`bType`) alır.
        *   `CPythonQuest::Instance().GetQuestButtonNoticeCount(bType)`'ı çağırır ve sonucu Python integer olarak döndürür.

*   **Modül Başlatma (`initquest()`)**:
    *   `quest` adında Python modülünü oluşturur ve yukarıda bahsedilen fonksiyonları kaydeder.
    *   `PyModule_AddIntConstant` kullanarak çeşitli sabitleri (örn: `QUEST_MAX_NUM`, `ENABLE_QUEST_RENEWAL` ile `QUEST_TYPE_...` sabitleri) Python modülüne ekler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyuncunun aldığı ve aktif olan görevlerin istemci tarafında yönetilmesini sağlar. Sunucudan gelen görev komutları (yeni görev, görev güncellemesi, görev tamamlama vb.) `CPythonQuest` sınıfının fonksiyonlarını tetikler. Bu sınıf, görevleri bir listede tutar ve zamanlayıcıları, sayaçları günceller. Python tarafındaki kullanıcı arayüzü (`uiQuest.py` gibi) ise `quest` modülündeki fonksiyonları kullanarak bu görev bilgilerini çeker (örn: `quest.GetQuestCount()`, `quest.GetQuestData()`), görev listesini oluşturur, kalan süreleri gösterir ve görev ikonlarını yükler. Bu sayede oyuncu, görevlerini arayüz üzerinden takip edebilir. `ENABLE_QUEST_RENEWAL` aktif olduğunda, görevler türlerine göre sınıflandırılabilir ve arayüzde farklı şekillerde (örneğin, ana görevler, yan görevler için ayrı sekmeler) gösterilebilir. 

---

### `PythonRenderTargetModule.cpp` (Render Hedefi Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonRenderTargetModule.cpp`, oyun içinde belirli 3D modelleri veya sahneleri ayrı bir hedef dokuya (render target) çizdirmek ve bu dokuyu arayüzde (UI) göstermek için kullanılan Python modülünü tanımlar. Özellikle karakter önizleme, eşya önizleme gibi özelliklerde, modelin oyun dünyasından bağımsız bir şekilde, özel bir arka plan ve kamera açısıyla gösterilmesi gerektiğinde kullanılır. Bu dosya, `RENDER_TARGET` makrosu tanımlı olduğunda derlenir.

**Temel Fonksiyonlar ve Python Arayüzleri**

*   `initRenderTarget()`: `renderTarget` adlı Python modülünü ve bu modülün fonksiyonlarını Python yorumlayıcısına kaydeder.
*   `renderTargetSelectModel(index, modelIndex)`: Belirtilen `index`'e sahip render hedefinde gösterilecek olan modeli `modelIndex` ile seçer. Genellikle bir karakterin veya bir eşyanın farklı varyasyonlarını (örneğin zırh, silah) göstermek için kullanılır.
*   `renderTargetSetVisibility(index, isShow)`: Belirtilen `index`'e sahip render hedefinin görünürlüğünü ayarlar (`isShow` true ise görünür, false ise gizli).
*   `renderTargetSetBackground(index, szPathName)`: Belirtilen `index`'e sahip render hedefine bir arka plan resmi atar. `szPathName` ile belirtilen resim dosyası yüklenir ve render hedefinin arka planı olarak kullanılır.
*   `renderTargetSetZoom(index, bZoom)`: Belirtilen `index`'e sahip render hedefindeki modelin yakınlaştırılıp (zoom) yakınlaştırılamayacağını ayarlar.

**Orta Seviye İmpementasyon Detayları**

*   Modül, `CRenderTargetManager` singleton sınıfı üzerinden `CRenderTarget` nesnelerine erişir ve bu nesnelerin fonksiyonlarını çağırarak işlemleri gerçekleştirir.
*   Python'dan gelen argümanlar `PyTuple_GetByte`, `PyTuple_GetInteger`, `PyTuple_GetBoolean`, `PyTuple_GetString` gibi fonksiyonlarla C++ tarafında okunur.
*   `CreateBackground` fonksiyonu, `CPythonApplication::Instance().GetWidth()` ve `CPythonApplication::Instance().GetHeight()` çağrıları ile mevcut pencere boyutlarını alarak arka planı uygun şekilde oluşturur.

**Dosyanın Oyundaki Kullanım Amacı**

Bu modül, oyun arayüzünde 3D modellerin dinamik olarak gösterilmesi gereken durumlarda kullanılır. Örnekler:
*   Karakter oluşturma ekranında seçilen karakterin veya sınıfın önizlemesi.
*   Envanterde veya mağazada bir eşyanın üzerine gelindiğinde modelinin detaylı bir şekilde gösterilmesi.
*   Belirli özel yeteneklerin veya dönüşümlerin görsel bir sunumu.
*   Oyun içi mağazada satılan kostüm veya bineklerin önizlemesi.
Bu sayede oyuncular, eşyaları veya karakterleri oyun dünyasına girmeden veya onları kuşanmadan önce detaylı bir şekilde inceleyebilirler.

---

### `PythonrootlibManager.h` (Rootlib Yönetici Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonrootlibManager.h`, `ENABLE_CYTHON` makrosu tanımlı olduğunda kullanılan `rootlib` adlı bir Python modülünün başlatma fonksiyonunu (`initrootlibManager` veya `initrootlib` olarak tanımlanır) deklare eder. Bu başlık dosyası, `PythonrootlibManager.cpp` içinde implemente edilen ve çok sayıda alt Python modülünü (genellikle UI betikleri ve oyunla ilgili çeşitli yardımcı modüller) içeren `rootlib` modülünün C++ tarafından erişilebilir olmasını sağlar.

**Temel Yapısı**

*   `#if defined(ENABLE_CYTHON)`: Tüm içeriği bu önişlemci direktifi içine alır, yani sadece Cython kullanımı aktifse derlenir.
*   `#define initrootlibManager initrootlib`: `initrootlibManager` fonksiyon adını `initrootlib` olarak değiştirir. Bu, Python C API'sinin belirli bir isimlendirme kuralını takip etmesiyle ilgili olabilir (örneğin, `Py_InitModule` ile kullanılan isimle eşleşmesi).
*   Yorumlar içinde, `rootlib` modülünün sağladığı temel fonksiyonlar (`isExist`, `moduleImport`, `getList`) ve bu ana modül aracılığıyla erişilebilen çok sayıda alt modülün (örneğin, `colorInfo`, `consoleModule`, `game`, `ui`, `uiCharacter`, `uiInventory` vb.) listesi verilir.
*   `void initrootlibManager();`: `rootlib` modülünü başlatan fonksiyonun prototipini deklare eder.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, C++ ana programının, Cython ile derlenmiş ve `rootlib` adı altında toplanmış olan çeşitli Python modüllerini başlatabilmesi ve kullanabilmesi için gereklidir. `rootlib` modülü, oyunun Python ile yazılmış UI (kullanıcı arayüzü) kısımlarının ve diğer oyun mantığı betiklerinin C++ motoruyla entegrasyonunda merkezi bir rol oynar. Cython kullanıldığında, bu yapı Python betiklerinin C'ye dönüştürülerek daha performanslı çalışmasını sağlar ve C++ ile Python arasında daha verimli bir köprü oluşturur.

---

### `PythonrootlibManager.cpp` (Rootlib Yöneticisi ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonrootlibManager.cpp`, `ENABLE_CYTHON` makrosu tanımlı olduğunda, `rootlib` adında bir ana Python modülü oluşturur. Bu modül, istemcinin Python ile yazılmış çok sayıda alt modülünü (genellikle UI betikleri, oyun mantığı ve yardımcı fonksiyonlar içeren modüller) merkezi bir yerden yönetmek ve yüklemek için bir arayüz sağlar. Cython kullanıldığında, bu Python modülleri C koduna derlenir ve `rootlib` üzerinden C++ tarafına bağlanır, bu da performans artışı ve daha sıkı entegrasyon sağlar.

**Temel Fonksiyonlar ve Python Arayüzleri**

*   `rootlib_SMethodDef`: Her bir alt modülün adını ve başlatma fonksiyonunu tutan bir yapı.
*   `rootlib_init_methods[]`: `rootlib_SMethodDef` yapılarından oluşan bir dizi. Bu dizi, `rootlib` tarafından yönetilecek tüm alt modülleri ve bunların başlatma fonksiyonlarını listeler. Örneğin:
    *   `initcolorInfo`, `initconsoleModule`, `initconstInfo`
    *   `initgame`, `initinterfaceModule`, `initnetworkModule`
    *   Çok sayıda `initui*` fonksiyonu (örneğin, `initui`, `inituiCharacter`, `inituiInventory`, `inituiChat` vb.)
*   `rootlib_isExist(name)`: Verilen `name` (modül adı) ile bir alt modülün `rootlib_init_methods` listesinde var olup olmadığını kontrol eder.
*   `rootlib_moduleImport(name)`: Verilen `name` ile eşleşen alt modülün başlatma fonksiyonunu çağırır (`rootlib_init_methods[i].func()`), Python'un içe aktarılan modüller sözlüğünden (`PyImport_GetModuleDict()`) modül nesnesini alır ve bu nesneyi Python tarafına döndürür.
*   `rootlib_getList()`: `rootlib_init_methods` listesindeki tüm mevcut alt modül adlarının bir Python demetini (tuple) döndürür.
*   `initrootlibManager()` (veya `initrootlib`): `rootlib` ana modülünü Python'a kaydeder ve yukarıdaki `isExist`, `moduleImport`, `getList` fonksiyonlarını bu modülün metodları olarak tanımlar.

**Orta Seviye İmpementasyon Detayları**

*   Dosya, çok sayıda `PyMODINIT_FUNC init<ModuleName>();` şeklinde harici fonksiyon prototipi içerir. Bu fonksiyonlar, Cython tarafından üretilen ve her bir alt Python modülünü başlatan C fonksiyonlarıdır.
*   `Py_InitModule("rootlib", methods)`: Python C API'sini kullanarak `rootlib` adında yeni bir modül oluşturur ve `isExist`, `moduleImport`, `getList` C fonksiyonlarını bu modülün Python'dan çağrılabilir metodları olarak kaydeder.
*   `_stricmp` fonksiyonu, modül adlarını karşılaştırırken büyük/küçük harf duyarsız bir karşılaştırma yapar.
*   Hata durumlarında `PyErr_Occurred()`, `Py_BadArgument()`, `PyErr_SetString()` gibi Python C API fonksiyonları kullanılır.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, Cython aktif edildiğinde, istemcinin Python ile yazılmış (ve Cython ile C'ye derlenmiş) tüm UI ve oyun mantığı modüllerini tek bir `rootlib` Python modülü altında toplar. Bu, Python betiklerinin C++ motoru tarafından daha verimli bir şekilde yüklenmesini, yönetilmesini ve çağrılmasını sağlar. Oyuncunun gördüğü arayüzlerin (envanter, karakter ekranı, sohbet vb.) ve bu arayüzlerin C++ motoruyla etkileşiminin temelini oluşturur. `rootlib` modülü, Python tarafındaki kodun C++ tarafına "kaydedilmesi" ve oradan çağrılabilmesi için bir köprü görevi görür, bu da genellikle performans ve organizasyon açısından avantajlar sunar. 

---

### `PythonSafeBox.h` (Depo Sistemi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonSafeBox.h`, Metin2 istemcisindeki Depo (SafeBox) ve Nesne Market Deposu (Mall) sistemlerinin C++ tarafındaki veri yapısını ve temel fonksiyon arayüzünü tanımlar. `CPythonSafeBox` singleton sınıfını içerir, bu sınıf oyuncunun deposundaki ve nesne market deposundaki eşyaların ve paranın istemci tarafında tutulmasından ve yönetilmesinden sorumludur.

**Ana Yapılar ve Tanımlamalar**

*   **`CPythonSafeBox` (Singleton Sınıfı)**:
    *   Depo ve Nesne Market Deposu ile ilgili tüm istemci tarafı verilerini yönetir.
    *   `SAFEBOX_SLOT_X_COUNT`: Depo/Mall sayfasındaki sütun sayısı (genellikle 5).
    *   `SAFEBOX_SLOT_Y_COUNT`: Depo/Mall sayfasındaki satır sayısı (genellikle 9).
    *   `SAFEBOX_PAGE_SIZE`: Bir depo/mall sayfasındaki toplam slot sayısı (`X_COUNT * Y_COUNT`).
    *   `TItemInstanceVector`: `std::vector<TItemData>` için bir takma addır. Eşya verilerini (`TItemData` yapısı, muhtemelen `ItemData.h` veya benzeri bir dosyada tanımlıdır) tutar.
    *   **Depo (SafeBox) Fonksiyonları**:
        *   `OpenSafeBox(int iSize)`: Depoyu belirtilen sayıda sayfa ile açar (slot sayısını ayarlar ve eşya vektörünü yeniden boyutlandırır).
        *   `SetItemData(DWORD dwSlotIndex, const TItemData& rItemData)`: Belirtilen depo slotuna eşya verisi ekler/günceller.
        *   `DelItemData(DWORD dwSlotIndex)`: Belirtilen depo slotundaki eşya verisini siler.
        *   `SetMoney(DWORD dwMoney)`: Depodaki para miktarını ayarlar.
        *   `GetMoney()`: Depodaki para miktarını döndürür.
        *   `GetSlotItemID(DWORD dwSlotIndex, DWORD* pdwItemID)`: Belirtilen depo slotundaki eşyanın VNUM'unu alır.
        *   `GetCurrentSafeBoxSize()`: Mevcut depo slot sayısını (tüm sayfalar dahil) döndürür.
        *   `GetItemDataPtr(DWORD dwSlotIndex, TItemData** ppInstance)`: Belirtilen depo slotundaki eşyanın `TItemData` yapısına bir işaretçi döndürür.
    *   **Nesne Market Deposu (Mall) Fonksiyonları**:
        *   `OpenMall(int iSize)`: Nesne Market Deposunu belirtilen sayıda sayfa ile açar.
        *   `SetMallItemData(DWORD dwSlotIndex, const TItemData& rItemData)`: Belirtilen mall slotuna eşya verisi ekler/günceller.
        *   `DelMallItemData(DWORD dwSlotIndex)`: Belirtilen mall slotundaki eşya verisini siler.
        *   `GetMallItemDataPtr(DWORD dwSlotIndex, TItemData** ppInstance)`: Belirtilen mall slotundaki eşyanın `TItemData` yapısına bir işaretçi döndürür.
        *   `GetSlotMallItemID(DWORD dwSlotIndex, DWORD* pdwItemID)`: Belirtilen mall slotundaki eşyanın VNUM'unu alır.
        *   `GetMallSize()`: Mevcut mall slot sayısını döndürür.
    *   **Korunan Üyeler**:
        *   `m_ItemInstanceVector`: Normal depo eşyalarını saklayan vektör.
        *   `m_MallItemInstanceVector`: Nesne Market Deposu eşyalarını saklayan vektör.
        *   `m_dwMoney`: Depodaki para miktarı.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, istemcinin sunucudan aldığı depo ve nesne market deposu bilgilerini (eşyalar, para) saklaması ve yönetmesi için gerekli C++ sınıf yapısını tanımlar. Sunucu, oyuncu deposunu veya nesne market deposunu açtığında, eşya eklediğinde/çıkardığında veya para yatırdığında/çektiğinde, bu sınıftaki fonksiyonlar çağrılarak istemcideki veriler güncel tutulur. Bu bilgiler daha sonra Python tarafındaki UI betikleri (`uiSafebox.py`, `uiMall.py` vb.) tarafından okunarak oyuncuya arayüzde gösterilir.

---

### `PythonSafeBox.cpp` (Depo Sistemi Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonSafeBox.cpp`, `CPythonSafeBox` sınıfının implementasyonunu ve `safebox` adlı bir Python modülünün tanımlanmasını içerir. Bu dosya, istemci tarafındaki Depo (SafeBox) ve Nesne Market Deposu (Mall) verilerinin yönetilmesinden ve bu verilere Python betikleri aracılığıyla erişilmesini sağlayan fonksiyonların C++ tarafındaki mantığını gerçekleştirir.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **`CPythonSafeBox` Sınıfı Implementasyonu**:
    *   **Depo (SafeBox) Yönetimi**:
        *   `OpenSafeBox(int iSize)`: Depodaki para miktarını sıfırlar. `m_ItemInstanceVector`'ı temizler ve `SAFEBOX_SLOT_X_COUNT * iSize` boyutunda yeniden boyutlandırır. Tüm slotları `ZeroMemory` ile sıfırlar.
        *   `SetItemData(DWORD dwSlotIndex, const TItemData& rItemData)`: Verilen `dwSlotIndex` geçerliyse, `m_ItemInstanceVector`'daki o indekse `rItemData`'ı kopyalar.
        *   `DelItemData(DWORD dwSlotIndex)`: Verilen `dwSlotIndex` geçerliyse, `m_ItemInstanceVector`'daki o indeksteki `TItemData`'yı `ZeroMemory` ile sıfırlar.
        *   `SetMoney(DWORD dwMoney)`: `m_dwMoney`'i ayarlar.
        *   `GetMoney()`: `m_dwMoney`'i döndürür.
        *   `GetCurrentSafeBoxSize()`: `m_ItemInstanceVector.size()`'ı döndürür.
        *   `GetSlotItemID(DWORD dwSlotIndex, DWORD* pdwItemID)`: `dwSlotIndex` geçerliyse, `*pdwItemID`'ye o slottaki eşyanın `vnum`'unu atar.
        *   `GetItemDataPtr(DWORD dwSlotIndex, TItemData** ppInstance)`: `dwSlotIndex` geçerliyse, `*ppInstance`'ı `m_ItemInstanceVector`'daki ilgili `TItemData`'nın adresine ayarlar.
    *   **Nesne Market Deposu (Mall) Yönetimi**:
        *   `OpenMall(int iSize)`: `m_MallItemInstanceVector`'ı temizler ve `SAFEBOX_SLOT_X_COUNT * iSize` boyutunda yeniden boyutlandırır. Tüm slotları sıfırlar.
        *   `SetMallItemData(DWORD dwSlotIndex, const TItemData& rItemData)`: `dwSlotIndex` geçerliyse, `m_MallItemInstanceVector`'a eşya verisini kopyalar.
        *   `DelMallItemData(DWORD dwSlotIndex)`: `dwSlotIndex` geçerliyse, `m_MallItemInstanceVector`'daki ilgili eşya verisini sıfırlar.
        *   `GetMallItemDataPtr(DWORD dwSlotIndex, TItemData** ppInstance)`: `dwSlotIndex` geçerliyse, `*ppInstance`'ı `m_MallItemInstanceVector`'daki ilgili `TItemData`'nın adresine ayarlar.
        *   `GetSlotMallItemID(DWORD dwSlotIndex, DWORD* pdwItemID)`: `dwSlotIndex` geçerliyse, `*pdwItemID`'ye o slottaki eşyanın `vnum`'unu atar.
        *   `GetMallSize()`: `m_MallItemInstanceVector.size()`'ı döndürür.
    *   **Kurucu ve Yıkıcı**:
        *   `CPythonSafeBox()`: `m_dwMoney`'i 0 olarak başlatır.
        *   `~CPythonSafeBox()`: Boştur.

*   **`safebox` Python Modülü Fonksiyonları**:
    *   **Depo (SafeBox) ile İlgili Fonksiyonlar**:
        *   `safeboxGetCurrentSafeboxSize()`: `CPythonSafeBox::Instance().GetCurrentSafeBoxSize()` sonucunu döndürür.
        *   `safeboxGetItemID(ipos)`: Depodaki `ipos` slotundaki eşyanın VNUM'unu döndürür.
        *   `safeboxGetItemCount(ipos)`: Depodaki `ipos` slotundaki eşyanın sayısını döndürür.
        *   `safeboxGetItemFlags(ipos)`: Depodaki `ipos` slotundaki eşyanın bayraklarını (flags) döndürür.
        *   `safeboxGetItemMetinSocket(iSlotIndex, iSocketIndex)`: Depodaki `iSlotIndex` slotundaki eşyanın `iSocketIndex` numaralı taşını döndürür.
        *   `safeboxGetItemAttribute(iSlotIndex, iAttrSlotIndex)`: Depodaki `iSlotIndex` slotundaki eşyanın `iAttrSlotIndex` numaralı efsununu (tip, değer) döndürür.
        *   `safeboxGetMoney()`: Depodaki para miktarını döndürür.
        *   `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10`, `ENABLE_REFINE_ELEMENT`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_SET_ITEM` gibi makrolarla derlenen ek eşya bilgilerini (rastgele efsun, min/max değer, element, görünüm, set bonusu) döndüren fonksiyonlar.
    *   **Nesne Market Deposu (Mall) ile İlgili Fonksiyonlar**:
        *   `safeboxGetMallItemID(ipos)`: Mall deposundaki `ipos` slotundaki eşyanın VNUM'unu döndürür.
        *   `safeboxGetMallItemCount(ipos)`: Mall deposundaki `ipos` slotundaki eşyanın sayısını döndürür.
        *   `safeboxGetMallItemMetinSocket(iSlotIndex, iSocketIndex)`: Mall deposundaki `iSlotIndex` slotundaki eşyanın `iSocketIndex` numaralı taşını döndürür.
        *   `safeboxGetMallItemAttribute(iSlotIndex, iAttrSlotIndex)`: Mall deposundaki `iSlotIndex` slotundaki eşyanın `iAttrSlotIndex` numaralı efsununu döndürür.
        *   `safeboxGetMallSize()`: Mall deposunun toplam slot sayısını döndürür.
        *   Yukarıdaki makrolara bağlı olarak mall eşyaları için de ek bilgileri döndüren fonksiyonlar.
*   **Modül Başlatma (`initsafebox()`)**:
    *   `safebox` adında Python modülünü oluşturur ve yukarıda listelenen C++ fonksiyonlarını bu modülün Python'dan çağrılabilir metodları olarak kaydeder.
    *   `PyModule_AddIntConstant` kullanarak `SAFEBOX_SLOT_X_COUNT`, `SAFEBOX_SLOT_Y_COUNT`, `SAFEBOX_PAGE_SIZE` sabitlerini Python modülüne ekler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, oyuncunun deposu (hem normal depo hem de nesne market deposu) ile ilgili tüm verilerin istemci tarafında tutulmasını ve yönetilmesini sağlar. Sunucudan gelen depo bilgileri `CPythonSafeBox` sınıfı aracılığıyla işlenir ve saklanır. Python tarafındaki kullanıcı arayüzü betikleri (örneğin `uiSafebox.py`, `uiMall.py`) ise `safebox` Python modülünü kullanarak bu verilere erişir. Oyuncu depo arayüzünü açtığında, eşyaların listelenmesi, eşya üzerine gelindiğinde detaylarının (taşlar, efsunlar, görünüm vb.) gösterilmesi, depodaki paranın görüntülenmesi gibi işlemler bu dosyadaki fonksiyonlar aracılığıyla C++ ve Python katmanları arasında iletişim kurularak gerçekleştirilir. 


### `PythonShop.h` (Dükkan ve Pazar Sistemi Başlık Dosyası)

**Dosyanın Amacı ve İşlevselliği**

`PythonShop.h`, Metin2 istemcisindeki NPC dükkanları ve oyuncu tarafından kurulan özel pazar (tezgah) sistemleriyle ilgili C++ tarafındaki veri yapılarını ve temel fonksiyon arayüzünü tanımlar. `CPythonShop` singleton sınıfını içerir. Bu sınıf, açık olan bir dükkanın (NPC veya özel pazar) ürünlerini, sekmelerini, para birimlerini ve durumunu (açık/kapalı, özel pazar mı vb.) istemci tarafında yönetir. Ayrıca özel pazar kurmak için geçici olarak tutulan eşya stoğuyla ilgili işlemleri de içerir.

**Ana Yapılar ve Tanımlamalar**

*   **`EShopCoinType` (Enum)**:
    *   Dükkan sekmelerinde kullanılabilecek para birimi türlerini tanımlar:
        *   `SHOP_COIN_TYPE_GOLD`: Altın (varsayılan).
        *   `SHOP_COIN_TYPE_SECONDARY_COIN`: İkincil para birimi (örneğin Yang, Won, Çek gibi farklı birimler olabilir veya özel bir puan sistemi).
        *   `SHOP_COINT_TYPE_ITEM` (`ENABLE_SHOPEX_RENEWAL` ile): Eşya karşılığında satış.
        *   `SHOP_COINT_TYPE_EXP` (`ENABLE_SHOPEX_RENEWAL` ile): Tecrübe puanı karşılığında satış.

*   **`CPythonShop` (Singleton Sınıfı)**:
    *   **Temel Dükkan Durum ve Veri Yönetimi**:
        *   `Clear()`: Tüm dükkan verilerini temizler, durumu kapalıya ayarlar.
        *   `SetItemData(DWORD dwIndex, const TShopItemData& c_rShopItemData)`: Genel bir indeks (tüm sekmelerdeki slotları kapsayan) kullanarak dükkana eşya ekler/günceller.
        *   `GetItemData(DWORD dwIndex, const TShopItemData** c_ppItemData)`: Genel indeks ile dükkandaki bir eşyanın verisini alır.
        *   `SetItemData(BYTE tabIdx, DWORD dwSlotPos, const TShopItemData& c_rShopItemData)`: Belirli bir sekme (`tabIdx`) ve o sekmedeki slot pozisyonuna (`dwSlotPos`) eşya ekler/günceller.
        *   `GetItemData(BYTE tabIdx, DWORD dwSlotPos, const TShopItemData** c_ppItemData)`: Belirli bir sekme ve slottaki eşya verisini alır.
        *   `SetTabCount(BYTE bTabCount)`: Dükkandaki sekme sayısını ayarlar.
        *   `GetTabCount()`: Dükkandaki sekme sayısını döndürür.
        *   `SetTabCoinType(BYTE tabIdx, BYTE coinType)`: Belirli bir sekmenin kullanacağı para birimini ayarlar.
        *   `GetTabCoinType(BYTE tabIdx)`: Belirli bir sekmenin para birimini döndürür.
        *   `SetTabName(BYTE tabIdx, const char* name)`: Belirli bir sekmenin adını ayarlar.
        *   `GetTabName(BYTE tabIdx)`: Belirli bir sekmenin adını döndürür.
        *   `Open(BOOL isPrivateShop, BOOL isMainPrivateShop)`: Dükkanı açar ve özel pazar olup olmadığını, ana oyuncuya ait bir özel pazar olup olmadığını ayarlar.
        *   `Close()`: Dükkanı kapatır.
        *   `IsOpen()`: Dükkanın açık olup olmadığını döndürür.
        *   `IsPrivateShop()`: Mevcut dükkanın bir özel pazar olup olmadığını döndürür.
        *   `IsMainPlayerPrivateShop()`: Mevcut dükkanın ana oyuncuya ait bir özel pazar olup olmadığını döndürür.
    *   **Özel Pazar (Tezgah) Kurulumu ile İlgili Fonksiyonlar**:
        *   `SetNameDialogOpen(bool bOpen)`: Özel pazar adı girme diyaloğunun açık olup olmadığını ayarlar.
        *   `GetNameDialogOpen()`: Özel pazar adı girme diyaloğunun açık olup olmadığını döndürür.
        *   `ClearPrivateShopStock()`: Özel pazar kurmak için geçici olarak tutulan eşya stoğunu temizler.
        *   `AddPrivateShopItemStock(...)`: Envanterden bir eşyayı, belirtilen pozisyona ve fiyatla özel pazar stoğuna ekler (`ENABLE_CHEQUE_SYSTEM` ile çek fiyatı da eklenebilir).
        *   `DelPrivateShopItemStock(TItemPos ItemPos)`: Özel pazar stoğundan bir eşyayı kaldırır.
        *   `GetPrivateShopItemPrice(TItemPos ItemPos)`: Özel pazar stoğundaki bir eşyanın altın fiyatını alır.
        *   `GetPrivateShopItemPriceCheque(TItemPos ItemPos)` (`ENABLE_CHEQUE_SYSTEM` ile): Özel pazar stoğundaki bir eşyanın çek fiyatını alır.
        *   `BuildPrivateShop(const char* c_szName)`: Mevcut özel pazar stoğundaki eşyalarla ve verilen isimle bir özel pazar kurma isteğini sunucuya gönderir.
    *   **Korunan Üyeler**:
        *   `m_isShoping`: Dükkanın genel olarak açık olup olmadığını belirten bayrak.
        *   `m_isPrivateShop`: Açık olan dükkanın bir özel pazar olup olmadığını belirten bayrak.
        *   `m_isMainPlayerPrivateShop`: Açık olan özel pazarın ana oyuncuya ait olup olmadığını belirten bayrak.
        *   `m_isNameDialogOpen`: Özel pazar ismi girme penceresinin açık olup olmadığını tutar.
        *   `ShopTab` (Yapı): Her bir dükkan sekmesinin verilerini tutar:
            *   `coinType`: Sekmenin para birimi.
            *   `name`: Sekmenin adı.
            *   `items[SHOP_HOST_ITEM_MAX_NUM]`: Sekmedeki eşyaların (`TShopItemData`) dizisi.
        *   `m_bTabCount`: Aktif sekme sayısı.
        *   `m_aShoptabs[SHOP_TAB_COUNT_MAX]`: Dükkan sekmelerini tutan dizi.
        *   `TPrivateShopItemStock`: `std::map<TItemPos, TShopItemTable>` için bir takma ad. Özel pazar kurmak için geçici olarak seçilen eşyaların envanter pozisyonlarını ve satış bilgilerini (`TShopItemTable`) eşleştirir.
        *   `m_PrivateShopItemStock`: Özel pazar kurma stoğu.
    *   **`ENABLE_SHOPEX_RENEWAL` ile eklenenler**:
        *   `SetShopExLoading(const bool bShopExLoad)` ve `GetShopExLoading()`: Gelişmiş dükkan sistemi yükleme durumunu yönetir.
        *   `bShopExLoading`: Gelişmiş dükkanın yüklenip yüklenmediğini belirten özel bir bayrak.

**Dosyanın Oyundaki Kullanım Amacı**

Bu başlık dosyası, oyuncu bir NPC dükkanını açtığında veya bir oyuncu pazarına baktığında, istemcinin bu dükkanın içeriğini (eşyalar, fiyatlar, sekmeler) ve türünü yönetmesi için gerekli C++ sınıf yapısını tanımlar. Ayrıca, oyuncunun kendi özel pazarını kurarken envanterinden seçtiği eşyaları ve belirlediği fiyatları geçici olarak saklamak ve sunucuya göndermek için de kullanılır. Bu yapı, Python tarafındaki UI betiklerinin (`uiShop.py`, `uiPrivateShopBuilder.py` vb.) dükkan verilerine erişmesini ve kullanıcıya sunmasını sağlar.

---

### `PythonShop.cpp` (Dükkan ve Pazar Sistemi Uygulaması ve Python Modülü)

**Dosyanın Amacı ve İşlevselliği**

`PythonShop.cpp`, `CPythonShop` sınıfının implementasyonunu ve `shop` adlı bir Python modülünün tanımlanmasını içerir. Bu dosya, istemci tarafındaki NPC dükkanları ve oyuncu tarafından kurulan özel pazar (tezgah) sistemleriyle ilgili verilerin yönetilmesinden ve bu işlevlerin Python betikleri aracılığıyla erişilebilir olmasından sorumludur. Dükkan açma/kapama, eşya bilgilerini sorgulama, özel pazar kurma gibi işlemlerin C++ mantığını ve Python arayüzünü barındırır.

**Ana Çalışma Prensipleri ve Implementasyon Detayları**

*   **`CPythonShop` Sınıfı Implementasyonu**:
    *   **Dükkan Sekmesi ve Eşya Yönetimi**:
        *   `SetTabCoinType()`, `GetTabCoinType()`, `SetTabName()`, `GetTabName()`: Dükkan sekmelerinin para birimlerini ve isimlerini yönetir. `m_bTabCount` ile geçerli sekme sayısı kontrol edilir.
        *   `SetItemData(DWORD dwIndex, ...)` ve `GetItemData(DWORD dwIndex, ...)`: Genel bir slot indeksi (tüm sekmeleri kapsayan) kullanarak eşya verilerini ayarlar veya alır. Bu fonksiyonlar, indeksi sekme numarasına (`tabIdx = dwIndex / SHOP_HOST_ITEM_MAX_NUM`) ve sekme içindeki slot pozisyonuna (`dwSlotPos = dwIndex % SHOP_HOST_ITEM_MAX_NUM`) çevirerek diğer `SetItemData`/`GetItemData` varyasyonlarını çağırır.
        *   `SetItemData(BYTE tabIdx, DWORD dwSlotPos, ...)` ve `GetItemData(BYTE tabIdx, DWORD dwSlotPos, ...)`: Belirli bir sekme ve o sekmedeki slota göre eşya verilerini doğrudan `m_aShoptabs[tabIdx].items[dwSlotPos]` üzerinden yönetir. İndeks sınırları (`SHOP_TAB_COUNT_MAX`, `SHOP_HOST_ITEM_MAX_NUM`) kontrol edilir.
    *   **Özel Pazar Kurma Stoğu Yönetimi**:
        *   `ClearPrivateShopStock()`: `m_PrivateShopItemStock` map'ini temizler.
        *   `AddPrivateShopItemStock(...)`: Önce `DelPrivateShopItemStock` ile aynı pozisyondaki eski kaydı siler, sonra yeni `TShopItemTable` oluşturur (eşya VNUM ve sayısı başlangıçta 0'dır, sunucu onayı ile dolar), fiyatları ve gösterim pozisyonunu ayarlar ve `m_PrivateShopItemStock` map'ine ekler.
        *   `DelPrivateShopItemStock()`: Verilen `TItemPos`'a sahip eşyayı stoktan siler.
        *   `GetPrivateShopItemPrice()` (`ENABLE_CHEQUE_SYSTEM` ile `GetPrivateShopItemPriceCheque()`): Stoktaki bir eşyanın fiyatını döndürür.
    *   **Özel Pazar Kurma İşlemi**:
        *   `BuildPrivateShop(const char* c_szName)`: `m_PrivateShopItemStock`'taki eşyaları bir `std::vector<TShopItemTable>`'a kopyalar. Bu vektörü `ItemStockSortFunc` (eşyaları `display_pos`'a göre sıralar) kullanarak sıralar. Son olarak `CPythonNetworkStream::Instance().SendBuildPrivateShopPacket()` ile sunucuya pazar kurma isteği gönderir.
    *   **Dükkan Durum Yönetimi**:
        *   `Open()`: `m_isShoping`'i `TRUE` yapar, `m_isPrivateShop` ve `m_isMainPlayerPrivateShop` bayraklarını ayarlar.
        *   `Close()`: Yukarıdaki bayrakları `FALSE` yapar.
        *   `IsOpen()`, `IsPrivateShop()`, `IsMainPlayerPrivateShop()`: İlgili bayrakların durumunu döndürür.
    *   **Temizleme ve Başlatma**:\
        *   `Clear()`: Tüm durum bayraklarını sıfırlar/`FALSE` yapar, özel pazar stoğunu temizler, sekme sayısını 1'e ayarlar ve tüm sekmelerdeki eşya verilerini `memset` ile sıfırlar. `ENABLE_SHOPEX_RENEWAL` varsa `bShopExLoading`'i de `false` yapar.
        *   `CPythonShop()` (Kurucu): `Clear()`'ı çağırır.
*   **`shop` Python Modülü Fonksiyonları**:
    *   **Genel Dükkan İşlemleri**:
        *   `shopOpen(isPrivateShop, isMainPrivateShop)`: `CPythonShop::Instance().Open()`'ı çağırır.
        *   `shopClose()`: `CPythonShop::Instance().Close()`'ı çağırır.
        *   `shopIsOpen()`, `shopIsPrviateShop()`, `shopIsMainPlayerPrivateShop()`: İlgili C++ fonksiyonlarını çağırıp sonuçlarını Python'a döndürür.
    *   **Dükkan Eşya Bilgileri Sorgulama**:
        *   `shopGetItemID(nPos)`, `shopGetItemCount(iIndex)`, `shopGetItemPrice(iIndex)`: Belirtilen genel indeksteki eşyanın VNUM, sayı ve fiyatını döndürür.
        *   `shopGetItemMetinSocket(iIndex, iMetinSocketIndex)`, `shopGetItemAttribute(iIndex, iAttrSlotIndex)`: Eşyanın taş ve efsun bilgilerini döndürür.
        *   `ENABLE_CHEQUE_SYSTEM`, `ENABLE_APPLY_RANDOM`, `ENABLE_ITEM_VALUE10`, `ENABLE_REFINE_ELEMENT`, `ENABLE_CHANGE_LOOK_SYSTEM`, `ENABLE_SET_ITEM` gibi makrolara bağlı olarak ek eşya bilgilerini (çek fiyatı, rastgele efsun, min/max değer, element, görünüm, set bonusu) döndüren fonksiyonlar.
    *   **Dükkan Sekme Bilgileri**:
        *   `shopGetTabCount()`: `CPythonShop::instance().GetTabCount()` sonucunu döndürür.
        *   `shopGetTabName(bTabIdx)`: Belirtilen sekmenin adını döndürür.
        *   `shopGetTabCoinType(bTabIdx)`: Belirtilen sekmenin para birimi türünü döndürür.
    *   **Özel Pazar Kurma İşlemleri (Python Arayüzü)**:
        *   `shopClearPrivateShopStock()`: `CPythonShop::Instance().ClearPrivateShopStock()`'u çağırır.
        *   `shopAddPrivateShopItemStock(itemWindowType, itemSlotIndex, displaySlotIndex, price, priceCheque)`: Verilen bilgilerle `CPythonShop::Instance().AddPrivateShopItemStock()`'u çağırır.
        *   `shopDelPrivateShopItemStock(itemWindowType, itemSlotIndex)`: `CPythonShop::Instance().DelPrivateShopItemStock()`'u çağırır.
        *   `shopGetPrivateShopItemPrice(itemWindowType, itemSlotIndex)` (`ENABLE_CHEQUE_SYSTEM` ile `shopGetPrivateShopItemPriceCheque`): Stoktaki eşyanın fiyatını döndürür.
        *   `shopBuildPrivateShop(szName)`: `CPythonShop::Instance().BuildPrivateShop()`'u çağırır.
    *   **Özel Pazar İsim Diyaloğu**:
        *   `shopGetNameDialogOpen()`, `shopSetNameDialogOpen(bOpen)`: Özel pazar adı girme diyaloğunun durumunu alır/ayarlar.
*   **Modül Başlatma (`initshop()`)**:
    *   `shop` adında Python modülünü oluşturur ve yukarıda listelenen C++ fonksiyonlarını Python'dan çağrılabilir metodlar olarak kaydeder.
    *   `PyModule_AddIntConstant` kullanarak çeşitli sabitleri (örn: `SHOP_SLOT_COUNT`, `SHOP_COIN_TYPE_GOLD` vb.) Python modülüne ekler.

**Dosyanın Oyundaki Kullanım Amacı**

Bu dosya, hem NPC dükkanlarının hem de oyuncu tarafından kurulan özel pazarların istemci tarafındaki mantığını ve verilerini yönetir. Sunucudan gelen dükkan bilgileri (eşyalar, fiyatlar, sekmeler) `CPythonShop` sınıfı aracılığıyla işlenir ve saklanır. Oyuncu kendi özel pazarını kurmak istediğinde, envanterinden seçtiği eşyalar ve belirlediği fiyatlar yine bu sınıfın sağladığı geçici stokta tutulur ve \"Pazarı Kur\" komutuyla sunucuya gönderilir. Python tarafındaki kullanıcı arayüzü betikleri (`uiShop.py`, `uiPrivateShopBuilder.py` gibi) `shop` Python modülünü kullanarak bu verilere erişir, dükkan arayüzünü oluşturur, eşyaları listeler, fiyatları gösterir ve oyuncunun dükkanla etkileşimlerini (eşya satın alma, pazar kurma vb.) C++ tarafına iletir.

