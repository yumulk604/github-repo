# EterPythonLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `EterPythonLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. `EterPythonLib`, C++ ile yazılmış istemci çekirdeği ile Python betikleme dili arasında bir köprü görevi görür. Genellikle kullanıcı arayüzü (UI) yönetimi, olay işleme ve oyun mantığının bazı kısımlarının Python üzerinden kontrol edilmesi için kullanılır.

---

## Dosya Bazlı Detaylandırma 

### `PythonGraphic.h` ve `PythonGraphic.cpp` (`CPythonGraphic` Sınıfı)

*   **Genel Amaç:** `CPythonGraphic` sınıfı, bir singleton olarak, Python betiklerinin temel 2D ve bazı 3D grafik işlemlerini gerçekleştirmesi için bir arayüz sağlar. `CScreen` sınıfından miras alarak Direct3D cihazına ve temel ekran yönetimi fonksiyonlarına erişir. Python'dan çağrılabilen fonksiyonlar aracılığıyla renk ayarları, çizim işlemleri (dikdörtgen, çizgi, imaj), render durumu yönetimi, ekran görüntüsü alma ve gamma ayarları gibi işlevler sunar.

*   **`PythonGraphic.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıflar:** `CScreen` (temel ekran ve Direct3D cihaz yönetimi), `CSingleton<CPythonGraphic>` (singleton deseni için).
    *   **`SState` Yapısı (iç yapı):** Grafik durumlarını (özellikle `D3DXMATRIX matView` ve `D3DXMATRIX matProj`) kaydetmek ve geri yüklemek için `m_stateStack` içinde kullanılır.
    *   **Üye Değişkenler:**
        *   `m_lightColor`, `m_darkColor` (DWORD): Genellikle buton gibi UI elemanlarının kenar çizgileri için açık ve koyu renkleri tutar.
        *   `m_stateStack` (`std::stack<TState>`): Grafik durumlarını (projeksiyon ve görünüm matrisleri) itip çekmek için bir yığın.
        *   `m_SaveWorldMatrix` (`D3DXMATRIX`): Dünya matrisini kaydetmek için (kullanımı tam aktif görünmüyor).
        *   `m_CullingManager` (`CCullingManager`): Görüş alanı dışındaki nesnelerin çizilmesini engellemek için (bu sınıfta doğrudan aktif kullanımı az).
        *   `m_backupViewport` (`D3DVIEWPORT8`): Viewport ayarlarını yedeklemek için.
        *   `m_fOrthoDepth` (float): 2D ortografik projeksiyon için Z derinliği.
    *   **Metot Bildirimleri:**
        *   `PushState()` / `PopState()`: Mevcut projeksiyon ve görünüm matrislerini yığına iter veya yığından çeker.
        *   `GetD3D()`: Direct3D8 arayüzüne bir işaretçi döndürür.
        *   `SetInterfaceRenderState()`: 2D UI çizimi için uygun render durumlarını (alfa blending, ışıklandırma kapalı, ortografik projeksiyon vb.) ayarlar.
        *   `SetGameRenderState()`: 3D oyun dünyası çizimi için uygun render durumlarını (ışıklandırma açık, mipmapping vb.) ayarlar.
        *   `SetCursorPosition(int x, int y)`: Fare imlecinin ekran üzerindeki pozisyonunu ayarlar.
        *   `SetOmniLight()`: Sahneye genel bir aydınlatma (iki ışık kaynağı: bir spot, bir nokta) ekler (genellikle test veya basit sahneler için).
        *   `SetViewport()`, `RestoreViewport()`: Çizim yapılacak ekran alanını (viewport) ayarlar ve geri yükler.
        *   `GenerateColor(float r, float g, float b, float a)`: Verilen RGBA değerlerinden bir DWORD renk değeri oluşturur.
        *   `RenderDownButton()`, `RenderUpButton()`: Basit 2D buton görünümü çizer.
        *   `RenderImage()`, `RenderAlphaImage()`: Bir `CGraphicImageInstance`'ı belirtilen koordinatlarda çizer. `RenderAlphaImage` kenarlarda farklı alfa değerleri ile çizim yapar.
        *   `RenderCoolTimeBox()`, `RenderCoolTimeBoxInverse()`: Genellikle yetenek veya eşya bekleme sürelerini göstermek için kullanılan dairesel bir 'dolum' efekti çizer.
        *   `SaveJPEG()`, `SaveScreenShot()`: Ekran görüntüsünü JPEG formatında kaydeder. `SaveScreenShot`, JPEG'e ek olarak özel bir EXIF etiketi (`YMIR_METIN2`) ekler.
        *   `GetAvailableMemory()`: Kullanılabilir video belleği miktarını döndürür.
        *   `SetGamma()`: Ekranın gamma değerini ayarlar.

*   **`PythonGraphic.cpp` - Uygulama Detayları:**
    *   **Render Durumu Yönetimi:**
        *   `SetInterfaceRenderState()`: Projeksiyon, görünüm ve dünya matrislerini kimlik matrisine ayarlar. Alfa karıştırmayı etkinleştirir (`D3DBLEND_SRCALPHA`, `D3DBLEND_INVSRCALPHA`). 2D ortografik projeksiyonu `SetOrtho2D` ile ayarlar. Işıklandırmayı kapatır.
        *   `SetGameRenderState()`: Tekstür filtrelemeyi lineer yapar. Alfa karıştırmayı devre dışı bırakır ve ışıklandırmayı açar.
    *   **Ekran Görüntüsü (`SaveScreenShot`):**
        *   `GetBackBuffer()` ile arka tamponu alır.
        *   Desteklenen yüzey formatlarını (R8G8B8, A8R8G8B8, X8R8G8B8, R5G6B5 vb.) kontrol eder.
        *   Arka tamponu kilitleyerek (`LockRect`) piksel verilerine erişir.
        *   Farklı piksel formatlarından (örn: A8R8G8B8, R5G6B5) standart bir RGB24 formatına dönüştürme işlemi yapar.
        *   Dönüştürülmüş piksel verilerini `jpeg_save` (muhtemelen `JpegFile.h/cpp` içinden) fonksiyonu ile JPEG olarak kaydeder.
        *   Eğer `g_isScreenShotKey` global değişkeni `true` ise (ekran görüntüsü bir tuş ile alındıysa), kaydedilen JPEG dosyasına `GenScreenShotTag` ve `GetCRC32` kullanarak dosya adı ve içerik CRC'sinden oluşan özel bir EXIF kullanıcı yorumu etiketi ekler. Bu, dosyanın kaynağını ve bütünlüğünü doğrulamak için kullanılabilir.
    *   **`RenderCoolTimeBox()` / `RenderCoolTimeBoxInverse()`:**
        *   Belirli bir merkez ve yarıçapta, geçen süreye (`fTime`) bağlı olarak bir daire dilimini veya tersini üçgen fanı (triangle fan) kullanarak çizer. Bu, UI'da bekleme süresi göstergeleri için kullanılır.
        *   `ENABLE_GROWTH_PET_SYSTEM` makrosu tanımlıysa, `RenderCoolTimeBox` için farklı bir varsayılan renk ve alfa değeri kullanılır.
    *   **Diğer Çizim Fonksiyonları:**
        *   `RenderImage`: `CGraphicImageInstance`'dan alınan dokuyu (texture) kullanarak basit bir dörtgen çizer.
        *   `RenderAlphaImage`: Köşeleri için farklı alfa değerleri tanımlanabilen bir resim çizer.
        *   `RenderDownButton`, `RenderUpButton`: Basitçe bir iç kutu ve etrafına `m_lightColor` ve `m_darkColor` ile çizgiler çizerek buton efekti verir.
    *   **`SetGamma()`:** `SetGammaRamp` Direct3D fonksiyonunu kullanarak ekranın gamma ayarını değiştirir.

*   **Kullanım Amacı ve Python Entegrasyonu:**
    *   Bu sınıfın metotları, muhtemelen `PythonGraphicModule.cpp` (veya benzeri bir modül dosyası) içinde Python'a açılan fonksiyonlar aracılığıyla betiklerden çağrılır.
    *   Python tabanlı UI sisteminin (örneğin, pencere çizimi, butonlar, resimler, metinler) temel grafiksel ihtiyaçlarını karşılar.
    *   Oyun içi bazı özel efektlerin veya HUD elemanlarının Python tarafından çizilmesine olanak tanır.

*   **Bağımlılıklar:**
    *   `EterLib`: `StateManager`, `JpegFile`, `GrpTextInstance`, `GrpMarkInstance`, `GrpImageInstance`, `GrpExpandedImageInstance`.
    *   `EterGrnLib`: `ThingInstance` (doğrudan kullanılmasa da başlıkta include edilmiş).
    *   Direct3D 8 API. 

### `PythonGraphicModule.cpp` ve `PythonGraphicImageModule.cpp` (Python `grp` ve `grpImage` Modülleri)

*   **Genel Amaç:** Bu iki dosya, Python betiklerinin istemcinin grafik yeteneklerini kullanabilmesi için C++ fonksiyonlarını Python modülleri olarak sarmalar. `PythonGraphicModule.cpp`, genel grafik işlemleri için `grp` modülünü oluştururken; `PythonGraphicImageModule.cpp`, özellikle 2D resimlerin (image) yönetimi için `grpImage` modülünü oluşturur.

*   **`PythonGraphicImageModule.cpp` (`grpImage` Modülü):**
    *   **Temel İşlev:** Python'dan 2D resim dosyalarını yüklemek, bu resimlerden örnekler (`CGraphicImageInstance` ve `CGraphicExpandedImageInstance`) oluşturmak, bu örneklerin özelliklerini (konum, köken, döndürme, ölçek, renk, render alanı) ayarlamak ve onları ekranda çizdirmek için fonksiyonlar sunar.
    *   **Yardımcı Fonksiyonlar:**
        *   `PyTuple_GetImageInstance()`: Python tuple'dan bir `CGraphicImageInstance` işaretçisi alır.
        *   `PyTuple_GetExpandedImageInstance()`: Python tuple'dan bir `CGraphicExpandedImageInstance` işaretçisi alır ve tip kontrolü yapar.
    *   **Python'a Açılan Fonksiyonlar (`grpImage.` ile başlayanlar):**
        *   `Generate(filename)`: Verilen dosya adından bir `CGraphicImageInstance` oluşturur ve handle'ını (işaretçi) döndürür.
        *   `GenerateExpanded(filename)`: Verilen dosya adından bir `CGraphicExpandedImageInstance` oluşturur ve handle'ını döndürür.
        *   `GenerateFromHandle(handle)`: Mevcut bir `CGraphicImage` (resource handle) kullanarak yeni bir `CGraphicImageInstance` oluşturur.
        *   `Delete(imageInstanceHandle)`: Bir `CGraphicImageInstance`'ı siler.
        *   `DeleteExpanded(expandedImageInstanceHandle)`: Bir `CGraphicExpandedImageInstance`'ı siler.
        *   `SetFileName(imageInstanceHandle, filename)`: Mevcut bir `CGraphicImageInstance`'ın kaynak resmini değiştirir.
        *   `Render(imageInstanceHandle)`: Resim örneğini mevcut konumunda çizer (genellikle `CPythonGraphic::RenderImage` çağrılır, ancak bu modülde `pImageInstance->Render()` doğrudan çağrılır).
        *   `SetPosition(imageInstanceHandle, x, y)`: Resim örneğinin ekran üzerindeki konumunu ayarlar.
        *   `SetOrigin(expandedImageInstanceHandle, x, y)`: `CGraphicExpandedImageInstance`'ın döndürme ve ölçekleme için merkez noktasını (orijin) ayarlar.
        *   `SetRotation(expandedImageInstanceHandle, degrees)`: `CGraphicExpandedImageInstance`'ı belirtilen derece kadar döndürür.
        *   `SetScale(expandedImageInstanceHandle, x_scale, y_scale)`: `CGraphicExpandedImageInstance`'ı belirtilen oranlarda ölçekler.
        *   `SetRenderingRect(expandedImageInstanceHandle, left, top, right, bottom)`: `CGraphicExpandedImageInstance`'ın kaynak resimden hangi kısmının çizileceğini belirleyen bir dikdörtgen ayarlar (sprite sheet kullanımı için).
        *   `SetDiffuseColor(imageInstanceHandle, r, g, b, a)`: Resim örneğinin genel rengini ve alfa değerini ayarlar.
        *   `GetWidth(imageInstanceHandle)`: Resim örneğinin genişliğini döndürür.
        *   `GetHeight(imageInstanceHandle)`: Resim örneğinin yüksekliğini döndürür.
    *   **Modül Başlatma (`initgrpImage()`):** Yukarıdaki fonksiyonları içeren `PyMethodDef` dizisini kullanarak `grpImage` modülünü Python'a kaydeder.

*   **`PythonGraphicModule.cpp` (`grp` Modülü):**
    *   **Temel İşlev:** `CPythonGraphic` sınıfının birçok fonksiyonunu ve diğer grafiksel yardımcı sınıfların (örn: `CTextBar`, `CCameraManager`) işlevlerini Python'a açar. Bu, Python'dan genel 2D/3D çizim, render durumu yönetimi, kamera kontrolü, metin işleme, ekran görüntüsü alma gibi çok çeşitli grafiksel işlemleri yapma imkanı sunar.
    *   **Python'a Açılan Fonksiyonlar (`grp.` ile başlayanlar - bazı önemli örnekler):**
        *   **`CTextBar` Yönetimi:**
            *   `CreateTextBar(width, height)`, `CreateBigTextBar(width, height, fontSize)`: Belirtilen boyutlarda ve isteğe bağlı font büyüklüğünde `CTextBar` nesneleri oluşturur.
            *   `DestroyTextBar(textBarHandle)`: Bir `CTextBar` nesnesini siler.
            *   `RenderTextBar(textBarHandle, x, y)`: `CTextBar`'ı belirtilen konumda çizer.
            *   `TextBarSetTextColor(textBarHandle, r, g, b)`: Metin rengini ayarlar.
            *   `TextBarGetTextExtent(textBarHandle, text)`: Verilen metnin kaplayacağı boyutu döndürür.
            *   `TextBarTextOut(textBarHandle, x, y, text)`: `CTextBar` içine metin yazar.
            *   `ClearTextBar(textBarHandle)`: `CTextBar` içeriğini temizler.
            *   `SetTextBarClipRect(textBarHandle, sx, sy, ex, ey)`: `CTextBar` için bir kırpma (clipping) alanı tanımlar.
        *   **Genel Grafik ve Render İşlemleri (`CPythonGraphic` sarmalayıcıları):**
            *   `InitScreenEffect()`, `ClearDepthBuffer()`
            *   `PushState()`, `PopState()`: Grafik durumlarını (projeksiyon, görünüm matrisleri) yönetir.
            *   `PushMatrix()`, `PopMatrix()`: Mevcut dünya matrisini yığına iter/çeker.
            *   `Translate(x, y, z)`, `Rotate(angle, x, y, z)`, `Identity()`: Dünya matrisini manipüle eder.
            *   `SetPerspective()`, `SetOrtho2d()`, `SetOrtho3d()`: Projeksiyon matrislerini ayarlar.
            *   `GenerateColor(r, g, b, a)`, `SetColor(color_long)`, `SetDiffuseColor(r, g, b, a)`, `SetClearColor(r, g, b)`: Çizim ve temizleme renklerini ayarlar.
            *   `GetCursorPosition3d()`, `SetCursorPosition(x, y)`.
            *   **2D Çizim:** `RenderLine()`, `RenderBox()`, `RenderRoundBox()`, `RenderBar()`, `RenderGradationBar()`, `RenderDownButton()`, `RenderUpButton()`.
            *   **3D Çizim:** `RenderCube()`, `RenderBar3d()`, `RenderBox3d()`.
            *   `GetAvailableMemory()`: Kullanılabilir video belleğini döndürür.
            *   `SaveScreenShot()`: Ekran görüntüsünü kullanıcının "Belgelerim/METIN2" klasörüne zaman damgalı bir dosya adıyla kaydeder.
            *   `SaveScreenShotToPath(basePath)`: Ekran görüntüsünü belirtilen yola zaman damgalı olarak kaydeder.
            *   `SetGamma(gamma_value)`.
            *   `SetInterfaceRenderState()`, `SetGameRenderState()`.
            *   `SetViewport(fx, fy, fWidth, fHeight)`, `RestoreViewport()`.
            *   `SetOmniLight()`.
        *   **Kamera İşlemleri (`CCameraManager` sarmalayıcıları):**
            *   `SetAroundCamera(distance, pitch, roll, lookAtZ)`
            *   `SetPositionCamera(posX, posY, posZ, distance, pitch, roll)`
            *   `SetEyeCamera(eyeX, eyeY, eyeZ, centerX, centerY, centerZ, upX, upY, upZ)`
            *   `GetCameraPosition()`: Kameranın mevcut pozisyonunu döndürür (eye position).
            *   `GetTargetPosition()`: Kameranın baktığı hedef noktayı döndürür.
        *   **Diğer:**
            *   `Culling()`: `CCullingManager::Instance().Process()` çağırır.
            *   `SetAlpha(alpha_value)`: (Yorum satırı halinde, muhtemelen eski veya tamamlanmamış bir fonksiyon).
    *   **Modül Başlatma (`initgrp()`):** Yukarıdaki fonksiyonları içeren `PyMethodDef` dizisini kullanarak `grp` modülünü Python'a kaydeder.
    *   **Not (Linter Hataları):** `grpSaveScreenShot` fonksiyonunda `SHGetSpecialFolderPath` ve `CreateDirectory` çağrıları `char*` (ANSI) argümanlar alırken, bu fonksiyonların Unicode ortamında `wchar_t*` (LPWSTR/LPCWSTR) beklemesi muhtemeldir. Bu durum, eğer proje tam Unicode desteğiyle derleniyorsa, çalışma zamanı hatalarına veya beklenmedik davranışlara yol açabilir. Uygun string dönüşümleri (örn: `MultiByteToWideChar`) veya `TCHAR` ve ilgili makroların (örn: `_T()`, `CreateDirectory` yerine `CreateDirectoryAuto`) kullanılması gerekebilir.

*   **Kullanım Amacı:**
    *   Bu modüller, Metin2'nin kullanıcı arayüzünün (pencereler, butonlar, envanter slotları, metin gösterimi vb.) büyük ölçüde Python ile yazılmasına ve yönetilmesine olanak tanır.
    *   Python betiklerinin, C++ tarafındaki karmaşık grafik işlemlerine kolayca erişmesini sağlar.
    *   Oyun içi özel efektlerin, HUD elemanlarının ve diğer görsel bileşenlerin dinamik olarak Python üzerinden kontrol edilmesini mümkün kılar.

*   **Bağımlılıklar:**
    *   Python C API.
    *   `EterLib`: `CPythonGraphic`, `CResourceManager`, `CGraphicImage`, `CGraphicImageInstance`, `CGraphicExpandedImageInstance`, `CTextBar`, `CCameraManager`, `CCullingManager` gibi birçok sınıf.
    *   Windows API (örn: `SHGetSpecialFolderPath`, `CreateDirectory`, `time`). 

### `PythonGraphicTextModule.cpp` ve `PythonGraphicThingModule.cpp` (Python `grpText` ve `grpThing` Modülleri)

*   **Genel Amaç:** Bu dosyalar, `EterLib` ve `EterGrnLib` kütüphanelerindeki metin (`CGraphicTextInstance`) ve basit 3D nesne (`CGraphicThingInstance`) işleme yeteneklerini Python betiklerine açar. `PythonGraphicTextModule.cpp`, `grpText` modülünü; `PythonGraphicThingModule.cpp` ise `grpThing` modülünü oluşturur.

*   **`PythonGraphicTextModule.cpp` (`grpText` Modülü):**
    *   **Temel İşlev:** Python'dan `CGraphicTextInstance` nesneleri oluşturmak, yönetmek ve çizdirmek için fonksiyonlar sunar. Bu, kullanıcı arayüzündeki metinlerin (etiketler, buton yazıları, bilgi mesajları vb.) Python ile dinamik olarak kontrol edilmesini sağlar.
    *   **Yardımcı Fonksiyon:**
        *   `PyTuple_GetTextInstance()`: Python tuple'ından bir `CGraphicTextInstance` işaretçisi alır.
    *   **Python'a Açılan Fonksiyonlar (`grpText.` ile başlayanlar):**
        *   `Generate()`: Yeni bir `CGraphicTextInstance` oluşturur ve handle'ını döndürür.
        *   `Destroy(textInstanceHandle)`: Bir `CGraphicTextInstance`'ı siler.
        *   `GetSize(textInstanceHandle)`: Metnin mevcut piksel cinsinden genişliğini ve yüksekliğini döndürür `(width, height)`.
        *   `SetPosition(textInstanceHandle, x, y)`: Metnin ekran üzerindeki konumunu ayarlar.
        *   `SetText(textInstanceHandle, text_string)`: Metin örneğinin göstereceği string'i ayarlar.
        *   `SetSecret(textInstanceHandle, boolean_value)`: Metnin gizli modda (örn: şifre alanları için yıldız `*` olarak) gösterilip gösterilmeyeceğini ayarlar.
        *   `SetOutline(textInstanceHandle, boolean_value)`: Metne dış çizgi (outline) eklenip eklenmeyeceğini ayarlar.
        *   `GetText(textInstanceHandle)`: Metin örneğinin mevcut string değerini döndürür.
        *   `SetFontName(textInstanceHandle, font_name_string)`: Kullanılacak yazı tipini ayarlar (verilen `font_name_string`'e `.fnt` uzantısı eklenerek `CResourceManager`'dan yüklenir).
        *   `SetFontColor(textInstanceHandle, color_long)` veya `SetFontColor(textInstanceHandle, r, g, b)`: Metin rengini ayarlar (DWORD veya float RGB).
        *   `SetOutlineColor(textInstanceHandle, color_long)` veya `SetOutlineColor(textInstanceHandle, r, g, b)`: Dış çizgi rengini ayarlar.
        *   `Render(textInstanceHandle)`: Metin örneğini çizer.
        *   `Update(textInstanceHandle)`: Metin örneğini günceller (içsel durumlar için, örneğin imleç yanıp sönmesi).
        *   `ShowCursor(textInstanceHandle)`, `HideCursor(textInstanceHandle)`: Metin içindeki düzenleme imlecini gösterir/gizler.
        *   `SetHorizontalAlign(textInstanceHandle, align_enum)`, `SetVerticalAlign(textInstanceHandle, align_enum)`: Metnin yatay ve dikey hizalamasını ayarlar.
        *   `SetMax(textInstanceHandle, max_chars)`: Metin örneğinin kabul edeceği maksimum karakter sayısını ayarlar.
        *   `GetSplitingTextLineCount(text_string, line_limitation_width)`: Verilen metni, belirtilen piksel genişliğine göre kaç satıra bölüneceğini hesaplar (`|` karakteri manuel satır sonu olarak kabul edilir).
        *   `GetSplitingTextLine(text_string, line_limitation_width, line_number)`: Verilen metni, belirtilen genişliğe göre böler ve istenen satırı döndürür.
        *   `PixelPositionToCharacterPosition(textInstanceHandle, pixel_position_x)`: Metin içinde verilen yatay piksel konumuna karşılık gelen karakter indeksini döndürür (metin düzenleme için).
    *   **Modül Başlatma (`initgrpText()`):** Fonksiyonları `grpText` modülü olarak Python'a kaydeder.

*   **`PythonGraphicThingModule.cpp` (`grpThing` Modülü):**
    *   **Temel İşlev:** Python'dan basit 3D nesneleri (`CGraphicThingInstance`) yüklemek, konumlandırmak, döndürmek, ölçeklemek, güncellemek ve çizdirmek için fonksiyonlar sunar. Bunlar genellikle UI'da veya oyun dünyasında basit dekoratif 3D objeler, efekt parçacıkları veya karakterlere takılan küçük modeller olabilir.
    *   **Yardımcı Fonksiyon:**
        *   `PyTuple_GetThingInstance()`: Python tuple'ından bir `CGraphicThingInstance` işaretçisi alır.
    *   **Python'a Açılan Fonksiyonlar (`grpThing.` ile başlayanlar):**
        *   `Generate(filename)`: Verilen dosya adından (genellikle `.gr2` uzantılı bir model) bir `CGraphicThingInstance` oluşturur, modeli yükler ve handle'ını döndürür. Oluşturulurken varsayılan olarak bir model ve bir model örneği için yer ayırır.
        *   `Delete(thingInstanceHandle)`: Bir `CGraphicThingInstance`'ı siler.
        *   `SetFileName(thingInstanceHandle, filename)`: Mevcut bir `CGraphicThingInstance`'ın modelini değiştirir. Öncekini temizler ve yenisini yükler.
        *   `Render(thingInstanceHandle)`: 3D nesne örneğini çizer.
        *   `Update(thingInstanceHandle)`: Nesne örneğini günceller. Bu, `Deform()` metodunu da çağırarak animasyonlu modellerin iskelet/vertex deformasyonlarını hesaplar.
        *   `SetPosition(thingInstanceHandle, x, y, z)`: Nesnenin 3D uzaydaki konumunu ayarlar.
        *   `SetRotation(thingInstanceHandle, degrees_y_axis)`: Nesneyi Y ekseni etrafında belirtilen derece kadar döndürür.
        *   `SetScale(thingInstanceHandle, x_scale, y_scale, z_scale)`: Nesneyi belirtilen eksenlerde ölçekler.
    *   **Modül Başlatma (`initgrpThing()`):** Fonksiyonları `grpThing` modülü olarak Python'a kaydeder.

*   **Kullanım Amacı:**
    *   `grpText` modülü, oyunun kullanıcı arayüzündeki tüm metin tabanlı gösterimler için kritik öneme sahiptir. Python UI scriptleri, bu modül aracılığıyla metinleri kolayca oluşturur, günceller ve biçimlendirir.
    *   `grpThing` modülü, UI'da veya oyun dünyasında daha basit 3D grafik öğelerinin Python ile kontrol edilmesini sağlar. Örneğin, karakter seçim ekranındaki basit bir dönen silah modeli veya bir yetenek kullanıldığında çıkan geçici bir görsel efekt bu modülle yönetilebilir.

*   **Bağımlılıklar:**
    *   Python C API.
    *   `EterLib`: `CGraphicTextInstance`, `CGraphicText`, `CResourceManager`.
    *   `EterGrnLib`: `CGraphicThingInstance`, `CGraphicThing` (3D model ve animasyon için).
    *   Standart C++ kütüphaneleri (örn: `string` `PythonGraphicTextModule.cpp` içinde). 

### `PythonGridSlotWindow.h` ve `PythonGridSlotWindow.cpp` (`UI::CGridSlotWindow` Sınıfı)

*   **Genel Amaç:** `CGridSlotWindow` sınıfı, `CSlotWindow` sınıfından miras alarak, envanterler, depolar veya ticaret pencereleri gibi 2D ızgara (grid) düzeninde slotların gösterilmesi ve yönetilmesi için özelleşmiş bir pencere türü sunar. Birden fazla slot kaplayabilen eşyaların (örneğin, 2x2 veya 1x3 boyutunda bir eşya) bu ızgara üzerinde doğru bir şekilde yerleştirilmesini, seçilmesini ve etkileşimde bulunulmasını yönetir.

*   **`PythonGridSlotWindow.h` - Temel Tanımlar ve Bildirimler:**
    *   **Miras Aldığı Sınıf:** `UI::CSlotWindow`.
    *   **`Type()` (static DWORD):** Sınıf için benzersiz bir tip kimliği döndürür (RTTI benzeri bir mekanizma için).
    *   **Yapıcı/Yıkıcı:** Standart.
    *   **Temel Metot Bildirimleri:**
        *   `Destroy()`: Pencereyi ve içindeki slotları temizler.
        *   `ArrangeGridSlot(startIndex, xCount, yCount, slotSizeX, slotSizeY, tempSizeX, tempSizeY)`: Belirtilen sayıda satır (`yCount`) ve sütun (`xCount`) ile slotları bir ızgara şeklinde düzenler. Slotların boyutu ve aralarındaki boşluklar da ayarlanabilir.
    *   **Korumalı (Protected) Üye Değişkenler:**
        *   `m_dwxCount`, `m_dwyCount` (DWORD): Izgaradaki sütun ve satır sayısı.
        *   `m_SlotVector` (`std::vector<TSlot*>`): Izgaradaki tüm slotlara hızlı erişim için bir vektör. Slotlar bu vektörde `x + y * m_dwxCount` indeksiyle saklanır.
    *   **Korumalı (Protected) Metot Bildirimleri (İç Mantık):**
        *   `__Initialize()`: Üye değişkenleri başlangıç değerlerine ayarlar.
        *   `GetPickedSlotPointer(TSlot** ppSlot)`: Fare imlecinin altındaki (birden fazla slot kaplayan eşyalar için merkez) slotu bulur.
        *   `GetPickedSlotList(itemWidth, itemHeight, std::list<TSlot*>* pSlotPointerList)`: Fare imlecinin altında, belirtilen boyuttaki bir eşyanın kaplayacağı tüm slotları bir liste olarak döndürür.
        *   `GetGridSlotPointer(gridX, gridY, TSlot** ppSlot)`: Izgara koordinatlarına (x, y) göre bir slot işaretçisi döndürür.
        *   `GetSlotPointerByNumber(slotNumber, TSlot** ppSlot)`: Global slot numarasına göre bir slot işaretçisi döndürür.
        *   `GetPickedGridSlotPosition(localMouseX, localMouseY, int* pGridX, int* pGridY)`: Fare imlecinin pencere içindeki lokal koordinatlarına karşılık gelen ızgara pozisyonunu (x, y) bulur.
        *   `CheckMoving(realSlotNumberOfMovingItem, movingItemIndex, const std::list<TSlot*>& targetSlots)`: Bir eşyanın `targetSlots` listesindeki slotlara taşınıp taşınamayacağını kontrol eder (slotlar boş mu, aynı eşya mı var vb.).
        *   `OnIsType(dwType)`: Verilen tipin bu sınıf tipiyle eşleşip eşleşmediğini kontrol eder.
        *   `OnRefreshSlot()`: Izgaradaki tüm slotların `dwCenterSlotNumber` ve `dwRealCenterSlotNumber` gibi içsel bilgilerini, üzerlerinde bulunan eşyaların boyutlarına göre günceller. Bu, bir eşya eklendiğinde veya kaldırıldığında çağrılır.
        *   `OnRenderPickingSlot()`: Fare ile bir eşya sürüklenirken, eşyanın bırakılabileceği potansiyel slot alanını görsel olarak (genellikle yarı saydam bir dörtgen ile) vurgular. `CheckMoving` sonucuna göre vurgu rengi değişebilir (örneğin, geçerliyse beyaz, geçersizse kırmızı).

*   **`PythonGridSlotWindow.cpp` - Uygulama Detayları:**
    *   **`ArrangeGridSlot()`:** Önce mevcut slotları `Destroy()` ile temizler. Ardından `m_dwxCount` ve `m_dwyCount` ayarlar. Belirtilen sayıda slotu `AppendSlot` (üst sınıftan gelir) ile oluşturur ve her oluşturulan slotun işaretçisini `m_SlotVector` içine doğru indekse yerleştirir. Son olarak pencerenin genel boyutunu hesaplayıp ayarlar.
    *   **`OnRefreshSlot()`:** İki aşamalı çalışır:
        1.  Önce tüm slotların merkez slot numaralarını kendi slot numaralarına eşitler.
        2.  Sonra ızgarayı tekrar tarar. Eğer bir slot üzerinde bir eşya varsa (`pSlot->isItem`), bu eşyanın kapladığı tüm alt slotların (`pSubSlot`) `dwCenterSlotNumber` ve `dwItemIndex` gibi bilgilerini ana slotun bilgileriyle günceller.
    *   **`GetPickedSlotList()`:** Fare pozisyonuna göre, belirli bir boyuttaki (`iWidth`, `iHeight` - genellikle sürüklenen eşyanın boyutu) bir öğenin merkezleneceği potansiyel ızgara konumlarını hesaplar. Bu hesaplama, öğenin yarısının hangi slota denk geldiğini ve kenarlardan taşma olup olmadığını dikkate alır. Sonuçta, öğenin kaplayacağı tüm slotları `pSlotPointerList`'e ekler.
    *   **`GetPickedSlotPointer()`:** Eğer fare ile bir eşya sürükleniyorsa, `GetPickedSlotList` ile potansiyel slotları alır ve bu slotlar arasından en uygun olanını (genellikle en küçük slot numarasına sahip veya üzerinde eşya olmayan) merkez slot olarak seçer.
    *   **`OnRenderPickingSlot()`:**
        *   Eğer `CWindowManager` üzerinden bir eşya sürükleniyorsa çalışır.
        *   `GetPickedSlotList` ile potansiyel slotları alır.
        *   Eğer "kullanım modu" (`m_isUseMode`) aktifse ve fare tek bir slotun üzerindeyse, o slotun ait olduğu (birden fazla slot kaplayan) eşyanın tamamını vurgular. Kullanılabilirliğe göre (`m_isUsableItem`) veya eşya türüne göre farklı renkler kullanabilir.
        *   Aksi halde, `CheckMoving` ile sürüklenen eşyanın bu potansiyel slotlara bırakılıp bırakılamayacağını kontrol eder. Sonuca göre (geçerli/geçersiz) farklı bir renkle, eşyanın kaplayacağı tüm alanı birleşik bir dörtgen olarak çizer.
        *   `ENABLE_CLIP_MASK` makrosu tanımlıysa, çizim alanı `m_rMaskRect` ile sınırlandırılır.
    *   **`CheckMoving()`:** Bir eşyanın hedef slotlara taşınmasının geçerli olup olmadığını kontrol eder. Eğer hedef slotlarda başka bir eşya varsa ve bu eşya, sürüklenen eşya ile aynı değilse `false` döner.
    *   **Diğer Metotlar:** `GetGridSlotPointer`, `GetSlotPointerByNumber`, `GetPickedGridSlotPosition` gibi fonksiyonlar, 2D ızgara yapısı üzerinden slotlara ve pozisyonlara erişim için temel yardımcı fonksiyonlardır.

*   **Kullanım Amacı ve Python Entegrasyonu:**
    *   Bu C++ sınıfı, Python tarafında oluşturulan envanter, karakter ekipman penceresi, hızlı slot barı, dükkan, depo gibi ızgara tabanlı kullanıcı arayüzü elemanlarının temel mantığını ve çizimini yönetir.
    *   Python UI scriptleri, bu sınıftan türetilmiş (veya doğrudan kullanılan) nesneler aracılığıyla slotlara eşya ekler, kaldırır, slotların durumunu sorgular ve kullanıcı etkileşimlerini (tıklama, sürükleme) C++ tarafına iletir.
    *   Özellikle `OnRenderPickingSlot` ve `OnRefreshSlot` gibi olay tabanlı metotlar, Python tarafından tetiklenen eylemlere (eşya sürükleme, envanter güncelleme) görsel geri bildirim sağlar.

*   **Bağımlılıklar:**
    *   `PythonSlotWindow.h` (üst sınıf).
    *   `EterBase/CRC32.h` (Type() için).
    *   `CPythonGraphic` (çizim işlemleri için).
    *   Standart C++ kütüphaneleri (`vector`, `list`, `algorithm`). 

### `PythonSlotWindow.h` ve `PythonSlotWindow.cpp` (`UI::CSlotWindow` Sınıfı)

*   **Genel Amaç:** `CSlotWindow`, kullanıcı arayüzündeki (UI) slot tabanlı pencerelerin (örneğin envanter, beceri çubuğu, karakter ekipman penceresi) temel sınıfıdır. Slotlara eşya, beceri veya diğer tıklanabilir öğelerin yerleştirilmesini, bu öğelerin görsel olarak temsil edilmesini (ikon, sayı, bekleme süresi, efektler) ve kullanıcı etkileşimlerini (tıklama, sürükleme, üzerine gelme) yönetir. Bu sınıf, Python'dan çağrılan fonksiyonlarla dinamik olarak yapılandırılır ve kontrol edilir.

*   **`PythonSlotWindow.h` - Temel Tanımlar ve Bildirimler:**
    *   **Enum'lar:**
        *   `ITEM_WIDTH`, `ITEM_HEIGHT`: Slot içindeki bir eşya ikonunun varsayılan piksel boyutu (genellikle 32x32).
        *   `SLOT_NUMBER_NONE`: Geçerli bir slot numarasının olmadığını belirtmek için kullanılır.
        *   `ESlotStyle`: Slot penceresinin etkileşim stilini belirler (`SLOT_STYLE_NONE`, `SLOT_STYLE_PICK_UP` - sürükle bırak için, `SLOT_STYLE_SELECT` - seçme modu için).
        *   `ESlotState`: Bir slotun durumunu bit flag'leri ile tanımlar (`SLOT_STATE_LOCK` - kilitli, `SLOT_STATE_CANT_USE` - kullanılamaz, `SLOT_STATE_DISABLE` - devre dışı, `SLOT_STATE_ALWAYS_RENDER_COVER` - kapak butonu her zaman çizilsin, `SLOT_STATE_HIGHLIGHT_GREEN` - yeşil vurgu).
            *   Koşullu derleme ile eklenen durumlar: `WJ_ENABLE_TRADABLE_ICON` için `SLOT_STATE_CANT_MOUSE_EVENT` (fare etkileşimi kapalı), `SLOT_STATE_UNUSABLE_ON_TOP_WND` (en üstteki pencerede kullanılamaz).
    *   **İç Sınıflar:**
        *   `CSlotButton`: Slot üzerine yerleştirilebilen genel amaçlı bir buton (örn: artı '+' butonu).
        *   `CCoverButton`: Slotun tamamını kaplayan, sol ve sağ tıklama davranışları özelleştirilebilen bir buton.
        *   `CCoolTimeFinishEffect`: Bir beceri veya eşyanın bekleme süresi dolduğunda oynatılan animasyonlu bir efekt (`CAniImageBox` türevi).
        *   `CHighLightImage` (`ENABLE_HIGH_LIGHT_IMAGE` ile): Bir slotu vurgulamak için kullanılan, alfa ve döndürme animasyonlarına sahip bir resim (`CExpandedImageBox` türevi).
    *   **`TSlot` Yapısı:** Her bir slotun tüm bilgilerini tutar:
        *   Durum (`dwState`), slot numaraları (`dwSlotNumber`, `dwCenterSlotNumber`, `dwRealSlotNumber`, `dwRealCenterSlotNumber`), eşya indeksi (`dwItemIndex`), eşya olup olmadığı (`isItem`).
        *   Bekleme süresi (`fCoolTime`, `fStartCoolTime`). `ENABLE_GROWTH_PET_SYSTEM` ile `bIsInverseCoolTime`.
        *   Aktiflik durumu (`bActive`).
        *   Pozisyon (`ixPosition`, `iyPosition`), hücre boyutu (`ixCellSize`, `iyCellSize`).
        *   Yerleştirilen eşyanın kapladığı slot sayısı (`byxPlacedItemSize`, `byyPlacedItemSize`).
        *   Görsel işaretçiler: `CGraphicImageInstance* pInstance` (eşya ikonu), `CNumberLine* pNumberLine` (eşya sayısı), `CCoverButton* pCoverButton`, `CSlotButton* pSlotButton`, `CImageBox* pSignImage` (gereksinim işareti), `CAniImageBox* pFinishCoolTimeEffect`.
        *   Koşullu derleme ile eklenen işaretçiler: `WJ_ENABLE_PICKUP_ITEM_EFFECT` için `pSlotEffect` (slot aktifleşme efekti), `ENABLE_SLOT_COVER_IMAGE_SYSTEM` için `pCoverImage` (slot kapak resmi), `ENABLE_HIGH_LIGHT_IMAGE` için `pHighLightImage` (vurgu resmi).
    *   **`SStoreCoolDown` Yapısı:** Bekleme sürelerini geçici olarak saklamak için kullanılır (örneğin, pencere kapatılıp açıldığında).
    *   **Üye Değişkenler:**
        *   `m_dwSlotType`, `m_dwSlotStyle`: Slot penceresinin tipi ve stili.
        *   `m_dwSelectedSlotIndexList`: Seçili slotların indekslerini tutan liste.
        *   `m_SlotList`: Penceredeki tüm `TSlot` yapılarını içeren liste.
        *   `m_dwOverInSlotNumber`, `m_dwToolTipSlotNumber`: Fare ile üzerine gelinen ve tooltip gösterilen slotların numaraları.
        *   `m_CoolDownStore`: Saklanan bekleme süreleri için bir map.
        *   `m_isUseMode`, `m_isUsableItem`: Kullanım modu ve kullanılabilir eşya durumu.
        *   İşaretçiler: `m_pBaseImageInstance` (slot arka planı), `m_pToggleSlotImage` (kullanımı belirsiz), `m_pSlotActiveEffect` (aktif slot efekti).
        *   `m_ReserveDestroyEffectDeque`: Çerçeve sonunda silinecek bekleme süresi bitiş efektlerini tutar.

*   **`PythonSlotWindow.cpp` - Uygulama Detayları:**
    *   **Slot Yönetimi:**
        *   `AppendSlot()`: Yeni bir `TSlot` oluşturur ve `m_SlotList`'e ekler.
        *   `SetSlot()`: Bir slota eşya (ikon, boyut) atar. Varsa mevcut ikon güncellenir, yoksa yenisi oluşturulur.
        *   `SetSlotCount()`, `SetSlotCountNew()`: Slot üzerindeki eşya sayısını gösteren `CNumberLine` oluşturur/günceller.
        *   `SetSlotCoolTime()`: Slot için bekleme süresi (cooldown) ayarlar. Kalan süreye göre bir `RenderCoolTimeBox` çizer.
        *   `ClearSlot()`: Bir slotun içeriğini (eşya ikonu, sayı, efektler vb.) temizler ve başlangıç durumuna getirir.
        *   `ActivateSlot()`, `DeactivateSlot()`: Bir slotu aktif veya pasif hale getirir. Aktif slotlar için `m_pSlotActiveEffect` (veya `WJ_ENABLE_PICKUP_ITEM_EFFECT` ile özel efektler) gösterilir.
        *   Slot durumlarını değiştiren fonksiyonlar (`LockSlot`, `DisableSlot`, `SetCantUseSlot` vb.) `TSlot::dwState` üyesindeki bitleri ayarlar veya temizler.
    *   **Görsel Oluşturma ve Yönetim:**
        *   `__Create...` ve `__Destroy...` fonksiyonları (örn: `__CreateBaseImage`, `__CreateSlotEnableEffect`): Slotlarla ilişkili çeşitli görsel efektleri ve temel resimleri oluşturur veya siler.
        *   `SetCoverButton()`, `AppendSlotButton()`, `AppendRequirementSignImage()`: Slotlara çeşitli buton ve işaret resimleri ekler.
        *   Koşullu derleme ile eklenen `AppendHighLightImage()`, `SetSlotCoverImage()` gibi fonksiyonlar, yeni görsel özellikleri yönetir.
    *   **Olay İşleme:**
        *   Fare olayları (`OnMouseLeftButtonDown`, `OnMouseRightButtonDown`, `OnMouseOver` vb.) yakalanır.
        *   `GetPickedSlotPointer()` ile fare altındaki slot belirlenir.
        *   Belirlenen slota ve olayın türüne göre Python tarafındaki ilgili handler fonksiyonlar (`m_poHandler` üzerinden `PyCallClassMemberFunc` ile) çağrılır (örn: `OnSelectEmptySlot`, `OnSelectItemSlot`, `OnOverInItem`).
        *   `OnRenderPickingSlot()`: Eğer `SLOT_STYLE_PICK_UP` aktifse ve bir eşya sürükleniyorsa, fare altındaki geçerli slotu vurgular.
        *   `OnRenderSelectedSlot()`: Eğer `SLOT_STYLE_SELECT` aktifse, `m_dwSelectedSlotIndexList` içindeki seçili slotları vurgular.
    *   **Çizim (`OnRender()`):**
        *   Önce slotların arka plan resmini (`RenderSlotBaseImage`) çizer.
        *   Slot stiline göre (`OnRenderPickingSlot` veya `OnRenderSelectedSlot`) özel vurgulamaları çizer.
        *   Tüm çocuk pencereleri (`m_pChildList`) çizer.
        *   Her bir slotu (`m_SlotList` üzerinden) tarayarak:
            *   Eşya ikonunu (`rSlot.pInstance`).
            *   `ENABLE_SLOT_COVER_IMAGE_SYSTEM` ile slot kapak resmini.
            *   Slotun durumuna göre (devre dışı, kilitli vb.) ek bir kaplama rengi.
            *   Bekleme süresi göstergesini (`RenderCoolTimeBox`).
            *   Kapak butonunu (`rSlot.pCoverButton`).
            *   Eşya sayısını (`rSlot.pNumberLine`).
            *   Bekleme süresi bitiş efektini (`rSlot.pFinishCoolTimeEffect`).
            *   Aktif slot efektini (`m_pSlotActiveEffect` veya `WJ_ENABLE_PICKUP_ITEM_EFFECT` ile `rSlot.pSlotEffect`).
            *   `ENABLE_HIGH_LIGHT_IMAGE` ile vurgu resmini çizer.
        *   Son olarak kilitli slotları (`RenderLockedSlot`) çizer.
    *   **Diğer:**
        *   `RefreshSlot()`: Slotların durumunu günceller ve fare üzerine gelme olaylarını yeniden tetikler.
        *   `OnRefreshSlot()`: Genellikle alt sınıflar (`CGridSlotWindow` gibi) tarafından override edilerek slotların iç düzenini günceller.

*   **Kullanım Amacı ve Python Entegrasyonu:**
    *   Python tarafında `ui.SlotWindow` veya benzeri bir isimle oluşturulan UI elemanlarının C++ temelini oluşturur.
    *   Python'dan gelen komutlarla (eşya ekle, slotu kilitle, bekleme süresi ata vb.) slotların içeriği ve davranışı dinamik olarak değiştirilir.
    *   Kullanıcı etkileşimleri (tıklama, sürükleme) C++ tarafında algılanır ve Python'da tanımlanmış geri çağırım (callback) fonksiyonlarına olay bilgisi iletilir.
    *   Envanter, dükkan, depo, beceri çubuğu, karakter durumu penceresi gibi oyunun temel arayüz bileşenlerinin oluşturulmasında kullanılır.

*   **Bağımlılıklar:**
    *   `PythonWindow.h` (üst sınıf).
    *   `EterBase/CRC32.h`, `EterBase/Filename.h`.
    *   `CPythonGraphic`, `CWindowManager`, `CGraphicImageInstance`, `CNumberLine`, `CImageBox`, `CAniImageBox`, `CButton`, `CTimer`.
    *   Python C API. 

### `PythonWindow.h` ve `PythonWindow.cpp` (Temel `UI::CWindow` Sınıfı ve Türevleri)

*   **Genel Amaç:** `UI::CWindow` sınıfı, Metin2 kullanıcı arayüzü (UI) sisteminin en temel yapı taşıdır. Ekrandaki tüm görsel elemanlar (pencereler, butonlar, resimler, metin alanları, slotlar vb.) doğrudan veya dolaylı olarak bu sınıftan miras alır. Bir pencerenin pozisyonunu, boyutunu, görünürlüğünü, hiyerarşik yapısını (ebeveyn-çocuk ilişkisi) ve temel olay işleme mekanizmalarını yönetir. Python tarafındaki UI scriptlerinin C++ ile etkileşimde bulunabilmesi için birincil arayüzü sağlar.

*   **`PythonWindow.h` - Temel Tanımlar ve Bildirimler (`UI::CWindow`):**
    *   **Enum'lar:**
        *   `EHorizontalAlign`, `EVerticalAlign`: Pencerenin ebeveynine göre yatay ve dikey hizalanma türlerini tanımlar (sol, orta, sağ; üst, orta, alt).
        *   `EFlags`: Pencerenin davranışını belirleyen bit flag'leri (örn: `FLAG_MOVABLE`, `FLAG_LIMIT`, `FLAG_DRAGABLE`, `FLAG_FLOAT`, `FLAG_NOT_PICK` - fare ile seçilemez, `FLAG_RTL` - sağdan sola düzen).
    *   **Temel Üyeler:**
        *   `m_x`, `m_y`: Pencerenin ebeveynine göre yerel koordinatları.
        *   `m_lWidth`, `m_lHeight`: Pencerenin genişliği ve yüksekliği.
        *   `m_rect` (RECT): Pencerenin ekran üzerindeki mutlak (global) koordinatlarını tutan dikdörtgen.
        *   `m_bShow`: Pencerenin görünür olup olmadığını belirten boolean.
        *   `m_dwFlag`: `EFlags` enum'undaki bayrakları tutan DWORD.
        *   `m_poHandler` (PyObject*): Bu C++ pencere nesnesiyle ilişkili Python UI nesnesine (event handler) işaretçi.
        *   `m_pParent` (CWindow*): Ebeveyn pencereye işaretçi.
        *   `m_pChildList` (`std::list<CWindow*>`): Çocuk pencerelerin listesi.
        *   `m_isUpdatingChildren`, `m_pReserveChildList`: Çocuk listesi güncellenirken (iteration sırasında) doğrudan silme/ekleme yapmamak için kullanılan geçici liste.
    *   **Hiyerarşi ve Konumlandırma Metotları:**
        *   `AddChild()`, `DeleteChild()`, `SetTop()`, `GetParent()`, `GetRoot()`, `IsChild()`.
        *   `SetSize()`, `SetPosition()`, `SetHorizontalAlign()`, `SetVerticalAlign()`, `UpdateRect()` (yerel koordinatları ve hizalamayı kullanarak `m_rect`'i günceller).
        *   `GetLocalPosition()`, `GetMouseLocalPosition()`.
    *   **Olay İşleme Sanal (Virtual) Metotları:**
        *   Genel: `OnUpdate()`, `OnRender()`, `OnChangePosition()`, `OnSetFocus()`, `OnKillFocus()`, `OnTop()`.
        *   Fare: `OnMouseDrag()`, `OnMouseOverIn()`, `OnMouseOverOut()`, `OnMouseLeftButtonDown()`, `OnMouseLeftButtonUp()`, `OnMouseLeftButtonDoubleClick()`, (ve sağ/orta butonlar için benzerleri), `RunMouseWheelEvent()`.
        *   Klavye/IME: `RunIMETabEvent()`, `RunIMEReturnEvent()`, `RunIMEKeyDownEvent()`, `RunKeyDownEvent()`, `RunKeyUpEvent()`, `RunPressEscapeKeyEvent()`.
        *   Bu metotlar genellikle Python'daki `m_poHandler` üzerindeki karşılık gelen fonksiyonları `PyCallClassMemberFunc` ile çağırır.
    *   **Diğer Metotlar:** `Show()`, `Hide()`, `IsShow()`, `IsIn()` (fare pozisyonunun pencere içinde olup olmadığını kontrol eder), `PickWindow()` (belirli bir koordinattaki en üstteki çocuk pencereyi bulur).
    *   **Koşullu Derleme (`ENABLE_CLIP_MASK`):**
        *   `SetClippingMaskRect()`, `SetClippingMaskWindow()`: Pencereye ve çocuklarına bir kırpma (clipping) alanı uygular.

*   **`PythonWindow.cpp` - Uygulama Detayları (`UI::CWindow`):**
    *   **Yapıcı/Yok Edici:** Temel başlatmaları yapar.
    *   **Olay Yayılımı:** Klavye ve IME olayları genellikle hiyerarşide yukarıdan aşağıya (ebeveynden çocuğa, en son eklenenden ilk eklenene doğru) `Run...Event()` metotları ile yayılır. Bir pencere olayı işlerse (`TRUE` dönerse) yayılım durur.
    *   **`UpdateRect()`:** Bu metot, pencerenin `m_rect`'ini (global ekran koordinatları) hesaplar. Ebeveyni varsa, ebeveynin `m_rect`'ine, kendi yerel `m_x`, `m_y` koordinatlarına ve `m_HorizontalAlign`, `m_VerticalAlign` ayarlarına göre konumunu belirler. `_USE_CPP_RTL_FLIP` makrosu tanımlıysa, sağdan-sola (RTL) diller için yatay hizalama mantığı değişir.
    *   **`PickWindow()`:** Verilen (x,y) koordinatlarında, görünür ve fare ile seçilebilir (`FLAG_NOT_PICK` olmayan) en üstteki (çocuk listesinde en sonda olan) pencereyi bulur. Özyinelemeli olarak çocukları kontrol eder.
    *   Çoğu `On...` olay metodu, `m_poHandler` aracılığıyla Python tarafındaki aynı isimli bir metodu çağırarak olayı Python'a iletir. Örneğin, `OnMouseLeftButtonDown()` C++ tarafında çağrıldığında, Python'daki `self.OnMouseLeftButtonDown()` fonksiyonunu tetikler.

*   **Aynı Dosyalarda Tanımlanmış Diğer UI Sınıfları (`CWindow` Türevleri):**
    *   **`CLayer`:** Sadece diğer pencereleri gruplamak için kullanılan, kendisi çizilmeyen bir katman.
    *   **`CRenderTarget` (`RENDER_TARGET` ile):** Bir render target dokusunu (oyun dünyasının bir görüntüsü vb.) UI elemanı olarak çizmek için kullanılır. `CRenderTargetManager` ile etkileşir.
    *   **`CBox`:** Basit, içi dolu tek renk bir dikdörtgen çizer (`CPythonGraphic::RenderBox2d`).
    *   **`CBar`:** `CBox`'a benzer, içi dolu bir dikdörtgen çizer (`CPythonGraphic::RenderBar2d`). `ENABLE_CLIP_MASK` ile kırpma desteği vardır.
    *   **`CLine`:** İki nokta arasında düz bir çizgi çizer (`CPythonGraphic::RenderLine2d`).
    *   **`CBar3D`:** Ortası bir renkle, kenarları ise farklı (genellikle daha açık/koyu) renklerle çizilen, 3D efekti verilmiş bir bar/dikdörtgen çizer. `ENABLE_CLIP_MASK` ile kırpma desteği vardır.
    *   **`CTextLine`:** Tek satırlık veya çok satırlı metinleri göstermek için `CGraphicTextInstance` kullanır. Font adı, rengi, hizalama, gizli mod, dış çizgi gibi özellikleri ayarlanabilir. `ENABLE_CLIP_MASK` ile kırpma desteği vardır.
    *   **`CNumberLine`:** Sayıları, her bir rakam veya özel karakter için ayrı resim dosyaları kullanarak gösterir (örn: "1.sub", "colon.sub"). Genellikle UI'da skor, para miktarı gibi sayısal değerleri göstermek için kullanılır. `ENABLE_CLIP_MASK` ile kırpma desteği vardır.
    *   **`CImageBox`:** Basit bir 2D resmi (`CGraphicImageInstance` kullanarak) gösterir. Resim yükleme, renk ayarı ve basit bekleme süresi (cooldown) gösterimi gibi özelliklere sahiptir. `ENABLE_CLIP_MASK` ve `ENABLE_IMAGE_SCALE` ile ek özellikler kazanır.
    *   **`CMarkBox`:** Genellikle guild logoları gibi küçük, indekslenmiş resimleri (sprite sheet içinden) göstermek için `CGraphicMarkInstance` kullanır. Ölçekleme ve renk ayarı yapılabilir. `ENABLE_CLIP_MASK` ile kırpma desteği vardır.
    *   **`CExpandedImageBox`:** `CImageBox`'tan türer, `CGraphicExpandedImageInstance` kullanarak resimlere ek olarak döndürme (rotation), orijin (origin) ayarlama ve render alanını (rendering rect) belirleme gibi gelişmiş transformasyon yetenekleri sunar. `ENABLE_CLIP_MASK` ve `ENABLE_GROWTH_PET_SYSTEM` / `ENABLE_IMAGE_CLIP_RECT` ile ek özellikler kazanır.
    *   **`CAniImageBox`:** Bir dizi resim karesinden oluşan bir animasyon oynatır (`CGraphicExpandedImageInstance` vektörü kullanır). Kareler arası gecikme, render modu ve ölçekleme ayarlanabilir. `ENABLE_CLIP_MASK` ve `ENABLE_IMAGE_SCALE` / `ENABLE_GROWTH_PET_SYSTEM` ile ek özellikler kazanır.
    *   **`CButton`:** Temel tıklanabilir buton sınıfı. "Up" (normal), "Over" (fare üzerinde), "Down" (basılı) ve "Disable" (devre dışı) durumları için ayrı `CGraphicImageInstance`'lar kullanır. Flash efekti, etkinleştirme/devre dışı bırakma gibi özelliklere sahiptir. `ENABLE_CLIP_MASK` ve `ENABLE_GROWTH_PET_SYSTEM` (ölçekleme için) ile ek özellikler kazanır.
    *   **`CRadioButton`:** `CButton`'dan türer, sadece bir kez basılabilen (tekrar basıldığında durum değiştirmeyen) radyo butonu davranışı sergiler.
    *   **`CToggleButton`:** `CButton`'dan türer, her basıldığında "down" ve "up" durumları arasında geçiş yapan toggle butonu davranışı sergiler.
    *   **`CDragButton`:** `CButton`'dan türer, basılı tutulup sürüklenebilen bir buton oluşturur. Hareket alanı sınırlandırılabilir (`SetRestrictMovementArea`).

*   **Kullanım Amacı ve Python Entegrasyonu:**
    *   `CWindow` ve türevleri, Python UI scriptlerinin (`ui.py`, `uitooltip.py` vb.) `window.Create` gibi fonksiyonlarla oluşturduğu tüm UI elemanlarının C++ arka planını oluşturur.
    *   Python'da bir UI elemanının `self.windowName = ui.Window()` gibi bir ifadeyle oluşturulması, arka planda bir `CWindow` (veya türevi) nesnesinin `m_poHandler` olarak bu Python nesnesini tutacak şekilde yaratılmasına neden olur.
    *   Pozisyon, boyut, resim, metin gibi özellikler Python'dan ayarlanır ve bu ayarlar C++ nesnesine yansıtılır.
    *   Kullanıcı etkileşimleri (fare, klavye) C++ tarafında `CWindowManager` tarafından algılanır, ilgili `CWindow` nesnesine yönlendirilir ve oradan da ilişkili Python nesnesinin olay işleyici (`event handler`) fonksiyonlarına aktarılır.

*   **Bağımlılıklar:**
    *   Python C API.
    *   `EterBase/CRC32.h`, `EterBase/Filename.h`, `EterBase/Utils.h`.
    *   `UserInterface/Locale_inc.h`.
    *   `CPythonGraphic`, `CGraphicImageInstance`, `CGraphicExpandedImageInstance`, `CGraphicTextInstance`, `CGraphicMarkInstance`, `CResourceManager` (`EterLib`).
    *   `CSlotWindow` (bazı metodlarda tip kontrolü için).
    *   `CPythonWindowManager` (genel pencere yönetimi için).
    *   `CRenderTargetManager` (`RENDER_TARGET` tanımlıysa).
    *   Standart C++ kütüphaneleri (`list`, `string`, `algorithm`, `functional`). 

    ### `PythonWindowManager.h` ve `PythonWindowManager.cpp` (`UI::CWindowManager` Sınıfı)

*   **Genel Amaç:** `CWindowManager`, bir singleton (`CSingleton<CWindowManager>`) olarak, Metin2 kullanıcı arayüzü (UI) sistemindeki tüm pencerelerin (`CWindow` ve türevleri) oluşturulması, yönetilmesi, katmanlandırılması ve olaylarının (fare, klavye, IME) işlenmesinden sorumlu merkezi bir sınıftır. Python betiklerinden gelen pencere oluşturma isteklerini karşılar, olayları uygun pencerelere yönlendirir ve genel UI durumunu (aktif pencere, kilitli pencere, sürüklenen öğe vb.) takip eder.

*   **`PythonWindowManager.h` - Temel Tanımlar ve Bildirimler:**
    *   **Typedef'ler:**
        *   `TLayerContainer`: `std::map<std::string, CWindow*>` - Katman adlarını (`"UI"`, `"GAME"` vb.) ilgili katman penceresine eşler.
        *   `TWindowContainer`: `std::list<CWindow*>` - Pencere listeleri için kullanılır.
        *   `TKeyCaptureWindowMap`: `std::map<int, CWindow*>` - Belirli bir sanal tuş kodunu yakalayan pencereyi eşler.
    *   **Enum (`WT_...` - Window Types):**
        *   Python'dan `RegisterTypeWindow` ile farklı `CWindow` türevlerini (örn: `WT_SLOT` için `CSlotWindow`, `WT_BUTTON` için `CButton`) oluşturmak için kullanılan pencere tipi tanımlayıcıları.
    *   **Temel Üye Değişkenler:**
        *   `m_lWidth`, `m_lHeight`: Ekranın piksel cinsinden genişliği ve yüksekliği.
        *   `m_iVres`, `m_iHres`: Çözünürlük (render hedefi boyutu).
        *   `m_lMouseX`, `m_lMouseY`: Fare imlecinin normalize edilmiş (ekran koordinatlarına dönüştürülmüş) pozisyonu.
        *   `m_pActiveWindow`: Şu anda aktif (odaklanmış) olan pencere.
        *   `m_pPointWindow`: Fare imlecinin üzerinde bulunduğu pencere.
        *   `m_pLockWindow`: Diğer tüm UI etkileşimlerini engelleyen kilitli pencere (örn: kesit sahneleri, önemli uyarılar).
        *   `m_pLeftCaptureWindow`, `m_pRightCaptureWindow`, `m_pMiddleCaptureWindow`: Fare butonlarına basıldığında olayı yakalayan pencereler (sürükleme işlemleri için).
        *   `m_pRootWindow`: Tüm UI pencerelerinin en üstteki ebeveyni olan kök pencere.
        *   `m_LayerWindowList`, `m_LayerWindowMap`: UI katmanlarını (`CLayer` nesneleri) tutar. Varsayılan katmanlar: `"GAME"`, `"UI_BOTTOM"`, `"UI"`, `"TOP_MOST"`, `"CURTAIN"`.
        *   **Sürükleme (Attaching) ile İlgili Üyeler:**
            *   `m_bAttachingFlag`: Bir öğenin (ikon) fare ile sürüklenip sürüklenmediğini belirtir.
            *   `m_dwAttachingType`, `m_dwAttachingIndex`, `m_dwAttachingSlotNumber`, `m_dwAttachingRealSlotNumber`: Sürüklenen öğenin türü, indeksi ve slot bilgileri.
            *   `m_byAttachingIconWidth`, `m_byAttachingIconHeight`: Sürüklenen ikonun boyutu.
            *   `m_poMouseHandler`: Python tarafında fare olaylarını ve özellikle sürükleme bırakma işlemlerini yöneten Python nesnesine işaretçi.
        *   `m_ReserveDeleteWindowList`: Çerçeve sonunda güvenli bir şekilde silinecek pencerelerin listesi.
        *   `m_PickAlwaysWindowList`: Her zaman fare etkileşimine açık olması gereken pencerelerin listesi (kilitli pencere durumunda bile).
        *   `ENABLE_MOUSE_WHEEL_TOP_WINDOW` ile `m_pTopWindow`: Fare tekerleği olaylarını alacak en üstteki pencere.
    *   **Temel Metot Bildirimleri:**
        *   Pencere Kayıt (`RegisterWindow`, `RegisterTypeWindow`, `RegisterSlotWindow` vb.): Python'dan gelen istekle belirtilen tipte bir pencere oluşturur ve ilgili katmana ekler.
        *   Olay Yönlendirme (`RunMouseMove`, `RunMouseLeftButtonDown`, `RunKeyDown`, `RunIMETabEvent` vb.): Gelen ham giriş olaylarını işler, ilgili pencereyi bulur ve olayı o pencerenin uygun metoduna iletir.
        *   Durum Yönetimi (`ActivateWindow`, `LockWindow`, `AttachIcon`, `DeattachIcon`, `SetTop`).
        *   `Update()`, `Render()`: Kök pencere üzerinden tüm UI hiyerarşisini günceller ve çizer.

*   **`PythonWindowManager.cpp` - Uygulama Detayları:**
    *   **Yapıcı (`CWindowManager()`):**
        *   Kök pencereyi (`m_pRootWindow`) oluşturur.
        *   Katmanları (`"GAME"`, `"UI"`, `"TOP_MOST"` vb.) oluşturur, `m_LayerWindowMap` ve `m_LayerWindowList`'e ekler ve kök pencereye çocuk olarak bağlar.
        *   Boş bir Python tuple'ı (`gs_poEmptyTuple`) önceden oluşturur (performans için).
    *   **Pencere Oluşturma ve Kayıt:**
        *   `__NewWindow(PyObject* po, DWORD dwWndType)`: Verilen `dwWndType`'a göre `CWindow`'dan türemiş uygun sınıfın (örn: `CSlotWindow`, `CButton`) bir örneğini `new` ile oluşturur ve Python nesnesi `po`'yu bu C++ penceresine bağlar.
        *   `Register...` fonksiyonları, `__NewWindow`'u (veya doğrudan `new CWindow` vs.) çağırarak pencereyi oluşturur ve belirtilen katmanın çocuğu olarak ekler.
        *   `__WINDOW_LEAK_CHECK__` makrosu tanımlıysa, oluşturulan pencereleri `gs_kSet_pkWnd` setine ekleyerek sızıntı takibi yapar.
    *   **Olay İşleme Mantığı:**
        *   **Fare Olayları:**
            *   `SetMousePosition()`: Gelen ham fare koordinatlarını ekran çözünürlüğüne göre normalize eder.
            *   `__PickWindow(long x, long y)`: Verilen koordinatlardaki en üstteki, görünür ve tıklanabilir pencereyi bulur. Kilitli pencere varsa sadece onun altındakileri arar. `m_PickAlwaysWindowList`'teki pencereleri öncelikli kontrol eder. Katmanları tersten (en üstten en alta) tarar.
            *   `RunMouseMove()`: Sürüklenen pencere varsa hareket ettirir (`FLAG_MOVABLE` veya `FLAG_DRAGABLE`). `m_pPointWindow`'u günceller ve `OnMouseOverIn`/`OnMouseOverOut` olaylarını tetikler.
            *   `RunMouseLeftButtonDown()`: `SetTopUIWindow()` ile tıklanan UI katmanındaki pencereyi en üste taşır. Tıklanan pencereyi yakalar (`m_pLeftCaptureWindow`) ve `OnMouseLeftButtonDown` olayını tetikler.
            *   `RunMouseLeftButtonUp()`: Yakalanan pencere veya fare altındaki pencere için `OnMouseLeftButtonUp` olayını tetikler. Sürükleme işlemini sonlandırır.
            *   Benzer mantık sağ ve orta fare butonları için de geçerlidir.
        *   **IME ve Klavye Olayları:**
            *   `RunIME...` fonksiyonları: Aktif veya kilitli pencereye IME olaylarını (güncelleme, tab, return, karakter girişi vb.) iletir.
            *   `RunKeyDown()`: Önce kilitli veya aktif pencereye olayı iletir. Eğer onlar işlemezse, kök pencere üzerinden tüm hiyerarşide olayı (`RunKeyDownEvent`) yayar. Olayı yakalayan pencere `m_KeyCaptureWindowMap`'e eklenir.
            *   `RunKeyUp()`: `m_KeyCaptureWindowMap`'ten ilgili tuşu yakalayan pencereyi bulur ve ona `OnKeyUp` olayını iletir.
    *   **Durum Yönetimi:**
        *   `AttachIcon()`/`DeattachIcon()`: Fareye bir ikon (eşya, yetenek) takma/çıkarma işlemlerini yönetir. `m_poMouseHandler` (Python tarafı) ile iletişim kurar.
        *   `LockWindow()`/`UnlockWindow()`: Bir pencereyi kilitleyerek diğer tüm UI etkileşimlerini geçici olarak engeller veya kilidi kaldırır. Kilitli pencereler bir yığında tutularak önceki aktif pencereye dönülebilir.
        *   `ActivateWindow()`/`DeactivateWindow()`: Bir pencereyi aktif (odaklanmış) yapar veya deaktif eder. Aktif pencereler de bir yığında tutularak önceki aktif pencereye dönülebilir.
        *   `SetTop()`: Bir pencereyi kendi ebeveyni içindeki çocuk listesinin en sonuna taşıyarak en üste çizilmesini sağlar.
        *   `SetTopUIWindow()`: Genellikle fare tıklamasıyla çağrılır, fare imlecinin altındaki "UI" katmanındaki pencereyi en üste taşır.
    *   **Güncelleme ve Çizim:**
        *   `Update()`: Önce `__ClearReserveDeleteWindowList()` ile silinmek üzere işaretlenmiş pencereleri gerçekten siler. Sonra `m_pRootWindow->Update()` çağırarak tüm UI ağacının güncellenmesini tetikler.
        *   `Render()`: `m_pRootWindow->Render()` çağırarak tüm UI ağacının çizilmesini tetikler.
    *   **Bellek Yönetimi:**
        *   `DestroyWindow()`: Bir pencereyi silmek için işaretler (`m_ReserveDeleteWindowList`'e ekler). Gerçek silme işlemi `__ClearReserveDeleteWindowList` içinde bir sonraki `Update`'te yapılır. Bu, bir pencerenin kendi olay döngüsü içinde silinmeye çalışılmasından kaynaklanabilecek sorunları önler.
        *   `NotifyDestroyWindow()`: Bir pencere silinmeden önce çağrılır ve bu pencereyi tutan çeşitli işaretçilerin (`m_pActiveWindow`, `m_pLockWindow` vb.) temizlenmesini sağlar.

*   **Kullanım Amacı ve Python Entegrasyonu:**
    *   Python UI scriptleri (`ui.py`, `interfaceModule.py` vb.) `wndMgr` veya benzeri bir global referans aracılığıyla `CWindowManager`'a erişir.
    *   Python'dan `wndMgr.RegisterWindow("isim", "layer")` gibi çağrılarla C++ tarafında `CWindow` nesneleri oluşturulur ve bu nesneler Python'daki UI sınıfı örnekleriyle ilişkilendirilir.
    *   Fare, klavye ve IME olayları Windows mesaj döngüsünden (genellikle `MSApplication` veya benzeri bir sınıf üzerinden) `CWindowManager`'a gelir. `CWindowManager` bu olayları işler, uygun `CWindow` nesnesini belirler ve bu C++ nesnesi de ilişkili Python nesnesinin ilgili olay işleyici metodunu (`OnMouseLeftButtonDown`, `OnKeyDown` vb.) çağırır.
    *   Genel UI davranışları (pencere kilitleme, ikon sürükleme) C++ tarafında yönetilirken, bu eylemlerin tetiklediği oyun mantığı veya daha detaylı UI tepkileri Python tarafında işlenir.

*   **Bağımlılıklar:**
    *   Python C API.
    *   `PythonWindow.h` (ve dolayısıyla `CWindow` ve tüm türevleri).
    *   `PythonSlotWindow.h`, `PythonGridSlotWindow.h`.
    *   `EterBase/Singleton.h`, `StdAfx.h`.
    *   Windows API (`timeGetTime`).
    *   Standart C++ kütüphaneleri (`map`, `list`, `set`, `string`, `algorithm`).

*   **Önemli Notlar:**
    *   `g_bShowOverInWindowName` global değişkeni `TRUE` ise, fare ile üzerine gelinen pencerenin adı konsola yazdırılır (debug amaçlı).
    *   `ENABLE_MOUSE_WHEEL_TOP_WINDOW` makrosu, fare tekerleği olaylarının doğrudan fare altındaki pencere yerine, özel olarak belirlenmiş bir `m_pTopWindow`'a gitmesini sağlar. Bu, genellikle kaydırma çubukları olan pencerelerde veya tam ekran arayüzlerde kullanılır.
    *   `__WINDOW_LEAK_CHECK__` makrosu, pencere sızıntılarını tespit etmek için geliştirme sırasında kullanılır.
    *   Çözünürlük (`m_iHres`, `m_iVres`) ve ekran boyutu (`m_lWidth`, `m_lHeight`) ayrı ayrı yönetilir; fare koordinatları bu iki değer arasında dönüştürülür.

### `PythonWindowManagerModule.cpp` (Python `wndMgr` Modülü)

*   **Genel Amaç:** Bu C++ dosyası, `UI::CWindowManager` ve `UI::CWindow` (ve `CTextLine`, `CSlotWindow`, `CButton` gibi türevleri) sınıflarının fonksiyonlarını Python betiklerinin kullanabilmesi için bir Python modülü (`wndMgr`) oluşturur. Python UI scriptleri, bu modül aracılığıyla C++ tarafındaki UI elemanlarını oluşturabilir, özelliklerini değiştirebilir (pozisyon, boyut, resim, metin vb.), olayları yönetebilir ve genel UI davranışlarını kontrol edebilir.

*   **Temel Yapı ve İşlevler:**
    *   **`PyTuple_GetWindow(PyObject* poArgs, int pos, UI::CWindow** ppRetWindow)` (Yardımcı Fonksiyon):** Python tuple'ından belirtilen pozisyondaki bir integer değeri (C++ pencere nesnesinin işaretçisi olarak saklanan handle) alıp `UI::CWindow*` işaretçisine dönüştürür. Bu, Python'dan C++'a pencere referanslarını güvenli bir şekilde geçirmek için kullanılır.
    *   **Python'a Açılan Fonksiyonlar (`wndMgr.` ile başlayanlar):** Bu dosya, çok sayıda C++ fonksiyonunu Python'a açar. Her bir Python fonksiyonu genellikle:
        1.  Python argümanlarını (`poArgs`) ayrıştırır (örn: pencere handle'ı, string'ler, integer'lar, float'lar).
        2.  `PyTuple_GetWindow` kullanarak C++ pencere nesnesine bir işaretçi elde eder.
        3.  Gerekirse tip kontrolü yapar (örn: `pWindow->IsType(UI::CSlotWindow::Type())`).
        4.  İlgili C++ pencere nesnesinin metodunu çağırır.
        5.  Sonucu (gerekiyorsa) Python nesnesine dönüştürerek (`Py_BuildValue`) geri döndürür veya `Py_BuildNone()` ile bir şey döndürmez.
    *   **Kapsanan Fonksiyon Kategorileri:**
        *   **WindowManager Global İşlemleri:**
            *   `SetMouseHandler()`: Fare olaylarını işleyecek Python nesnesini ayarlar.
            *   `SetScreenSize()`, `GetScreenWidth()`, `GetScreenHeight()`, `GetAspect()`: Ekran boyutu ve en-boy oranı yönetimi.
            *   `AttachIcon()`, `DeattachIcon()`, `SetAttachingFlag()`: Fare ile ikon/eşya sürükleme işlemleri.
            *   `OnceIgnoreMouseLeftButtonUpEvent()`: Bir sonraki sol fare bırakma olayını bir kereliğine yok sayar.
        *   **Pencere Kayıt ve Yok Etme:**
            *   `Register()`: Genel bir `CWindow` kaydeder.
            *   `RegisterSlotWindow()`, `RegisterGridSlotWindow()`, `RegisterTextLine()`, `RegisterImageBox()`, `RegisterButton()` vb.: Belirli `CWindow` türevlerini kaydeder.
            *   `Destroy()`: Bir pencereyi ve çocuklarını siler.
        *   **Temel Pencere Özellikleri (Okuma/Yazma):**
            *   `SetName()`, `GetName()`.
            *   `Show()`, `Hide()`, `IsShow()`.
            *   `SetParent()`, `SetPickAlways()`.
            *   `IsFocus()`, `SetFocus()`, `KillFocus()`.
            *   `Lock()`, `Unlock()`.
            *   `SetWindowSize()`, `GetWindowWidth()`, `GetWindowHeight()`.
            *   `SetWindowPosition()`, `GetWindowLocalPosition()`, `GetWindowGlobalPosition()`, `GetWindowRect()`.
            *   `SetWindowHorizontalAlign()`, `SetWindowVerticalAlign()`.
            *   `AddFlag()` (örn: "movable", "limit", "rtl"), `SetLimitBias()`.
            *   `UpdateRect()`.
            *   `GetChildCount()`.
        *   **Fare ve Durum Bilgileri:**
            *   `IsPickedWindow()`, `IsIn()` (bir pencerenin fare altında olup olmadığı).
            *   `GetMouseLocalPosition()`, `GetMousePosition()`.
            *   `IsDragging()`.
        *   **SlotWindow ve GridSlotWindow İşlemleri:**
            *   `AppendSlot()`, `ArrangeSlot()` (GridSlotWindow için).
            *   `ClearSlot()`, `ClearAllSlot()`, `HasSlot()`.
            *   `SetSlot()` (ikon, boyut, renk ile), `SetSlotCount()`, `SetSlotCountNew()`.
            *   `SetSlotCoolTime()`, `StoreSlotCoolTime()`, `RestoreSlotCoolTime()`, `SetSlotCoolTimeInverse()` (`ENABLE_GROWTH_PET_SYSTEM` ile).
            *   `ActivateSlot()`, `DeactivateSlot()`, `EnableSlot()`, `DisableSlot()`.
            *   `SetSlotType()`, `SetSlotStyle()`.
            *   `SetSlotBaseImage()`, `ShowSlotBaseImage()`, `HideSlotBaseImage()`.
            *   Kapak Butonları: `SetCoverButton()`, `EnableCoverButton()`, `DisableCoverButton()`, `IsDisableCoverButton()`, `SetAlwaysRenderCoverButton()`.
            *   Slot Butonları ve İşaretleri: `AppendSlotButton()`, `ShowSlotButton()`, `HideAllSlotButton()`, `AppendRequirementSignImage()`, `ShowRequirementSign()`, `HideRequirementSign()`.
            *   `RefreshSlot()`, `SetUseMode()`, `SetUsableItem()`.
            *   Slot Seçimi: `SelectSlot()`, `ClearSelected()`, `GetSelectedSlotCount()`, `GetSelectedSlotNumber()`, `IsSelectedSlot()`.
            *   `LockSlot()`, `UnlockSlot()`.
            *   Koşullu derleme ile eklenenler: `SetSlotCoverImage()` (`ENABLE_SLOT_COVER_IMAGE_SYSTEM`), `SetSlotDiffuseColor()` (`WJ_ENABLE_PICKUP_ITEM_EFFECT`), `SetCantMouseEventSlot()` (`WJ_ENABLE_TRADABLE_ICON`), `SetSlotHighlightedGreeen()`.
        *   **TextLine İşlemleri:**
            *   `SetMax()`, `SetHorizontalAlign()`, `SetVerticalAlign()`.
            *   `SetSecret()`, `SetOutline()`, `SetOutlineColor()`.
            *   `SetText()`, `GetText()`, `GetTextSize()`.
            *   `SetFontName()`, `SetFontColor()`.
            *   `ShowCursor()`, `HideCursor()`, `GetCursorPosition()`.
            *   `GetTextLineCount()`, `SetLineHeight()`.
        *   **ImageBox, ExpandedImageBox, AniImageBox İşlemleri:**
            *   `LoadImage()`.
            *   `SetDiffuseColor()`.
            *   `SetScale()`, `SetOrigin()`, `SetRotation()` (Expanded/Ani için).
            *   `SetRenderingRect()`, `SetRenderingMode()` (Expanded/Ani için).
            *   `SetDelay()` (Ani için), `AppendImage()` (Ani için), `ResetFrame()` (Ani için).
            *   `SetCoolTimeImageBox()`, `SetStartCoolTimeImageBox()`.
        *   **Button, RadioButton, ToggleButton, DragButton İşlemleri:**
            *   `SetUpVisual()`, `SetOverVisual()`, `SetDownVisual()`, `SetDisableVisual()`.
            *   `Enable()`, `Disable()`, `IsDisable()`.
            *   `Down()`, `SetUp()`, `IsDown()`.
            *   `Flash()`, `EnableFlash()`, `DisableFlash()`.
            *   `SetRestrictMovementArea()` (DragButton için).
        *   **NumberLine İşlemleri:** `SetNumber()`, `SetPath()`.
        *   **MarkBox İşlemleri:** `MarkBox_SetImageFilename()`, `MarkBox_SetIndex()`, `MarkBox_SetScale()`.
        *   **Diğer/Debug:** `GetHyperlink()`, `SetOutlineFlag()`.
        *   Koşullu Derleme ile Eklenenler: `SetClippingMaskRect()` (`ENABLE_CLIP_MASK`), `SetWheelTopWindow()` (`ENABLE_MOUSE_WHEEL_TOP_WINDOW`).
    *   **Modül Başlatma (`initwndMgr()`):**
        *   `PyMethodDef s_methods[]` dizisi, Python fonksiyon adlarını C++ fonksiyon işaretçilerine eşler.
        *   `Py_InitModule("wndMgr", s_methods)` ile `wndMgr` modülü Python'a kaydedilir.
        *   Çeşitli UI ile ilgili sabitler (hizalama türleri, slot stilleri, renk türleri vb.) `PyModule_AddIntConstant` ile modüle eklenir.

*   **Kullanım Amacı:**
    *   Bu dosya, C++ UI sisteminin Python'dan tamamen kontrol edilebilir hale gelmesini sağlayan ana köprüdür.
    *   Python UI geliştiricileri, `import wndMgr` yaparak bu modüldeki fonksiyonları kullanarak dinamik ve karmaşık arayüzler oluşturabilirler.
    *   Oyunun tüm kullanıcı arayüzü (envanter, karakter ekranı, görevler, dükkanlar, sistem seçenekleri vb.) büyük ölçüde bu modül aracılığıyla Python tarafından yönetilir.

*   **Bağımlılıklar:**
    *   Python C API (`Python.h`).
    *   `StdAfx.h` (proje genel başlıkları).
    *   `PythonWindow.h` (ve dolayısıyla `CWindow` ve tüm UI elemanı sınıfları).
    *   `PythonSlotWindow.h`, `PythonGridSlotWindow.h`.

*   **Önemli Notlar ve Potansiyel Sorunlar:**
    *   Fonksiyonların çoğunda hata kontrolü ve tip güvenliği için `PyTuple_Get...` fonksiyonlarının dönüş değerleri kontrol edilir ve `Py_BuildException()` ile Python tarafına istisna fırlatılabilir.
    *   Bazı fonksiyonlarda (`wndMgrSetSlot` gibi) Python tuple'ından renk gibi karmaşık verilerin okunması için detaylı ayrıştırma yapılır.
    *   Çok sayıda `#if defined(...)` koşullu derleme bloğu, farklı özellik setlerine (örn: `ENABLE_SLOT_COVER_IMAGE_SYSTEM`, `WJ_ENABLE_PICKUP_ITEM_EFFECT`, `RENDER_TARGET`) göre modülün içeriğini değiştirir. Bu, farklı sunucu yapılandırmaları veya istemci sürümleri için esneklik sağlar ancak kodun okunabilirliğini ve bakımını zorlaştırabilir.
    *   `PyTuple_GetWindow` fonksiyonu, pencere handle'larını (işaretçilerini) integer olarak alıp C++ pointer'ına dönüştürür. Bu, C++ nesne ömrü yönetimi açısından dikkatli olmayı gerektirir; Python tarafında referansı tutulan bir pencere C++ tarafında silinirse geçersiz işaretçi sorunları yaşanabilir (genellikle `CWindowManager::DestroyWindow` ve `m_ReserveDeleteWindowList` ile bu tür sorunlar önlenmeye çalışılır).

### `StdAfx.h` ve `StdAfx.cpp` (Ön Derlenmiş Başlıklar)

*   **Amaç:** Bu dosyalar, Microsoft Visual C++ projelerinde yaygın olarak kullanılan ön derlenmiş başlık (precompiled header - PCH) mekanizmasının bir parçasıdır. Temel amaçları, `EterPythonLib` kütüphanesi içinde sıkça kullanılan ve nadiren değiştirilen standart başlıkları, diğer temel kütüphane başlıklarını (`EterLib/StdAfx.h`, `ScriptLib/StdAfx.h`), yerelleştirme tanımlarını (`Locale_inc.h`) ve bu kütüphanedeki ana C++ sınıflarının (`CPythonGraphic`, `CWindowManager`) başlıklarını tek bir yerde toplayarak projenin genel derleme süresini önemli ölçüde kısaltmaktır.

*   **`StdAfx.h` İçeriği:**
    *   `#pragma once`: Bu başlık dosyasının her derleme birimi (.cpp dosyası) için yalnızca bir kez dahil edilmesini sağlar.
    *   `#define _CRT_SECURE_NO_WARNINGS`: Microsoft C Çalışma Zamanı Kütüphanesi (CRT) içindeki bazı fonksiyonların (güvenlik açısından riskli kabul edilen eski versiyonlar, örn: `strcpy` yerine `strcpy_s` kullanımı önerilir) kullanımı sırasında derleyicinin verdiği uyarıları bastırır.
    *   `#include "../UserInterface/Locale_inc.h"`: İstemcinin yerelleştirme (dil, bölge ayarları) ile ilgili tanımlamalarını ve makrolarını içerir.
    *   `#include "../EterLib/StdAfx.h"`: `EterLib` kütüphanesinin kendi ön derlenmiş başlığını dahil eder. Bu, `EterLib`'deki temel sınıflara ve fonksiyonlara erişim sağlar.
    *   `#include "../ScriptLib/StdAfx.h"`: `ScriptLib` kütüphanesinin ön derlenmiş başlığını dahil eder. Bu, betiklerle ilgili temel işlevlere erişim sağlar.
    *   `#include "PythonGraphic.h"`: `CPythonGraphic` sınıfının başlık dosyasını içerir.
    *   `#include "PythonWindowManager.h"`: `CWindowManager` sınıfının başlık dosyasını içerir.
    *   **Python Modül Başlatma Fonksiyon Bildirimleri:**
        *   `void initgrp();`: `PythonGraphicModule.cpp` içindeki `grp` Python modülünü başlatan fonksiyon.
        *   `void initgrpImage();`: `PythonGraphicImageModule.cpp` içindeki `grpImage` Python modülünü başlatan fonksiyon.
        *   `void initgrpText();`: `PythonGraphicTextModule.cpp` içindeki `grpText` Python modülünü başlatan fonksiyon.
        *   `void initgrpThing();`: `PythonGraphicThingModule.cpp` içindeki `grpThing` Python modülünü başlatan fonksiyon.
        *   `void initscriptWindow();`: Muhtemelen `PythonScriptWindowModule.cpp` (veya benzeri bir dosya) içindeki Python pencere betikleme ile ilgili bir modülü başlatan fonksiyon (bu dosya henüz analiz edilmedi).
        *   `void initwndMgr();`: `PythonWindowManagerModule.cpp` içindeki `wndMgr` Python modülünü başlatan fonksiyon.

*   **`StdAfx.cpp` İçeriği:**
    *   Bu dosya tipik olarak sadece `#include "stdafx.h"` satırını içerir. Visual C++ derleyicisi, bu .cpp dosyasını kullanarak `StdAfx.h` içinde listelenen tüm başlıkları önceden derler ve `.pch` uzantılı bir dosyada saklar. Diğer .cpp dosyaları derlenirken, bu ön derlenmiş başlık doğrudan kullanılarak derleme süreci hızlandırılır.

*   **Kullanım Amacı:**
    *   `EterPythonLib` içindeki tüm `.cpp` dosyaları muhtemelen ilk satırlarında `#include "StdAfx.h"` ifadesini içerir.
    *   Bu sayede, her bir `.cpp` dosyasının `StdAfx.h` içinde listelenen başlıkları tekrar tekrar işlemesi yerine, önceden derlenmiş olan `.pch` dosyası kullanılır.
    *   Özellikle büyük projelerde derleme sürelerinde önemli bir azalma sağlar.
    *   Python modüllerini başlatan `init...()` fonksiyonlarının bildirimleri, bu modüllerin ana uygulama veya kütüphane tarafından doğru bir şekilde çağrılabilmesini sağlar.