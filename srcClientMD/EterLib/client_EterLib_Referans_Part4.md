# EterLib Referans Kılavuzu - Bölüm 4

Bu dosya, `EterLib` referans kılavuzunun devamıdır.

---

## Dosya Bazlı Detaylandırma (Devamı)

### `LensFlare.h` ve `LensFlare.cpp` (`CLensFlare` ve `CFlare` Sınıfları)

*   **Kaynak Notu:** Kod içindeki yorumlar, bu sınıfların orijinal olarak IDV, Inc. (Interactive Data Visualization) tarafından geliştirilmiş olabileceğini belirtmektedir.
*   **Amaç:** `CLensFlare` sınıfı (`CScreen`'den türetilmiştir), 3D dünyadaki bir ışık kaynağının (genellikle güneş) ekrandaki projeksiyonuna bağlı olarak gerçekçi bir mercek parlaması (lens flare) efekti oluşturur ve yönetir. Bu efekt, ışık kaynağının görünürlüğüne (örneğin, sahnedeki nesneler tarafından engellenip engellenmediğine) ve ekrandaki konumuna göre dinamik olarak değişir.

*   **`LensFlare.h` - Tanımlar ve Bildirimler:**
    *   **`CFlare` Sınıfı:** Tek bir lens flare efektini oluşturan bireysel parlama parçalarını (örneğin, halkalar, parıltılar) yönetir.
        *   `SFlarePiece`: Bir parlama parçasını temsil eder. Bir resim (`CGraphicImageInstance`), ışık kaynağına göre göreceli konumu (`m_fPosition`), boyutu (`m_fWidth`) ve rengi (`m_pColor`) içerir.
        *   `Init(std::string strPath)`: Belirtilen yoldan parlama parçalarının dokularını ve yapılandırmasını (konum, boyut, renk) yükler.
        *   `Draw(...)`: Tüm parlama parçalarını, genel parlaklık ölçeği ve ekrandaki konuma göre çizer.
    *   **`CLensFlare` Sınıfı:**
        *   **Kalıtım:** `public CScreen`.
        *   Kurucu (`CLensFlare()`), Yıkıcı (`~CLensFlare()`).
        *   `Initialize(std::string strPath)`: `CFlare` nesnesini başlatır.
        *   `SetMainFlare(std::string strSunFile, float fSunSize)`: Ana ışık kaynağı parlamasının (genellikle güneş diski) dokusunu ve boyutunu ayarlar.
        *   `Compute(const D3DXVECTOR3& c_rv3LightDirection)`: Verilen ışık yönüne göre ışık kaynağının ekrandaki pozisyonunu hesaplar ve görünürlüğünü belirler.
        *   `DrawBeforeFlare()`, `DrawAfterFlare()`, `DrawFlare()`: Efektin farklı aşamalarını çizer (örneğin, ekranı aydınlatma, ana parlama, ikincil parlamalar).
        *   `SetFlareLocation(double dX, double dY)`: Işık kaynağının ekrandaki projeksiyon koordinatlarını ayarlar.
        *   `SetVisible(bool bState)`, `IsVisible()`: Efektin genel görünürlüğünü kontrol eder.
        *   `SetBrightnesses(float fBeforeBright, float fAfterBright)`: Efektin farklı aşamaları için parlaklık değerlerini ayarlar.
        *   `ReadControlPixels()`, `AdjustBrightness()`: Işık kaynağının engellenip engellenmediğini (occlusion) kontrol etmek ve parlaklığı buna göre ayarlamak için derinlik tamponu okuma yöntemleri kullanır.
        *   `CharacterizeFlare(...)`: Efektin genel özelliklerini (aktif mi, ana parlama gösterilsin mi, maksimum parlaklık, renk) ayarlar.
    *   **Önemli Üyeler:**
        *   `m_cFlare`: İkincil parlama parçalarını yöneten `CFlare` nesnesi.
        *   `m_SunFlareImageInstance`: Ana ışık kaynağı (güneş) parlaması için resim örneği.
        *   `m_afFlarePos`, `m_afFlareWinPos`: Parlamanın ekrandaki göreceli ve pencere koordinatları.
        *   `m_fBeforeBright`, `m_fAfterBright`: Efekt parlaklıkları.
        *   `m_bFlareVisible`, `m_bDrawFlare`: Görünürlük ve çizim bayrakları.
        *   `m_pControlPixels`, `m_pTestPixels`: Derinlik testi (occlusion) için kullanılan piksel tamponları.
        *   `m_bEnabled`, `m_bShowMainFlare`, `m_fMaxBrightness`: Efektin genel kontrol bayrakları ve parlaklık sınırı.

*   **`LensFlare.cpp` - Uygulama Detayları:**
    *   **Başlatma (`Initialize`, `SetMainFlare`, `CFlare::Init`):**
        *   `Initialize`, `m_cFlare.Init`'i çağırır.
        *   `SetMainFlare`, güneş diski için `CGraphicImageInstance` oluşturur.
        *   `CFlare::Init`, `g_strFiles`, `g_fPosition`, `g_fWidth`, `g_afColors` gibi statik dizilerde tanımlanan varsayılan parlama parçası yapılandırmasını (doku dosyaları, konumlar, boyutlar, renkler) yükler.
    *   **Konum ve Görünürlük Hesaplama (`Compute`):**
        *   Verilen ışık yönünü ve kamera hedefini kullanarak ışık kaynağının dünya koordinatlarını tahmin eder.
        *   `ProjectPosition` ile bu dünya koordinatlarını ekran koordinatlarına (`fX`, `fY`) dönüştürür ve `SetFlareLocation` ile saklar.
        *   Işık kaynağı vektörü ile kamera yön vektörü arasındaki açıyı (`acosf(fDotProduct)`) hesaplayarak ışık kaynağının kameranın önünde olup olmadığını belirler ve `SetVisible` ile görünürlük bayrağını ayarlar.
        *   Işık kaynağının ekrandaki konumuna göre (merkeze yakınlık) temel parlaklık değerlerini (`fBeforeBright`, `fAfterBright`) hesaplar ve `SetBrightnesses` ile ayarlar.
    *   **Occlusion Testi ve Parlaklık Ayarı (`ReadDepthPixels`, `ReadControlPixels`, `AdjustBrightness`, `ClampBrightness`):**
        *   `ReadDepthPixels`: Derinlik tamponundan (`D3DRS_ZBUFFER`) belirtilen bir alandaki (genellikle ışık kaynağının ekrandaki pozisyonu etrafında `c_nDepthTestDimension`x`c_nDepthTestDimension` boyutunda) derinlik değerlerini okur ve `m_pTestPixels` tamponuna yazar. Bu işlem `GetRenderTargetData` ile yapılır.
        *   `ReadControlPixels`: Eğer `m_pControlPixels` boşsa (genellikle ilk karede), `ReadDepthPixels`'i çağırarak referans derinlik değerlerini doldurur.
        *   `AdjustBrightness`: `ReadDepthPixels` ile güncel derinlik değerlerini okur. `m_pTestPixels` ile `m_pControlPixels`'i karşılaştırır. Eğer test piksellerinin büyük bir kısmı kontrol piksellerinden daha yakınsa (yani arada engel varsa), parlama parlaklığını (`m_fAfterBright`) azaltır. Eğer arada engel yoksa parlaklığı artırır (maksimum `m_fMaxBrightness`'a kadar).
        *   `ClampBrightness`: Parlaklık değerlerinin [0.0, 1.0] aralığında kalmasını sağlar.
    *   **Çizim (`DrawBeforeFlare`, `DrawFlare`, `CFlare::Draw`):**
        *   `DrawBeforeFlare`: Ana güneş parlamasını (`m_SunFlareImageInstance`) ekrandaki `m_afFlarePos` konumuna, `m_fSunSize` boyutunda çizer. Gerekli render durumlarını (Z-test kapalı, alpha blend açık vb.) ayarlar.
        *   `DrawFlare`: `m_cFlare.Draw` metodunu çağırır.
        *   `CFlare::Draw`: Efekt görünürse ve `m_fAfterBright` sıfırdan büyükse çalışır. Tüm parlama parçaları (`m_vFlares`) üzerinde döngüye girer. Her parça için:
            1.  Konumunu hesaplar (ışık kaynağı ile ekran merkezi arasındaki çizgi üzerinde, `pPiece->m_fPosition` değerine göre interpolasyon yaparak).
            2.  Boyutunu hesaplar.
            3.  Rengini ayarlar (genel `m_afColor` ve parça rengi `pPiece->m_pColor` ile karıştırılır, `fBrightScale` yani `m_fAfterBright` ile ölçeklenir).
            4.  Parçanın resmini (`pPiece->m_imageInstance`) hesaplanan konum, boyut ve renkle ekrana çizer.

*   **Kullanım Alanı:**
    *   Oyun dünyasına parlak bir ışık kaynağı (genellikle güneş) eklendiğinde daha fazla gerçekçilik ve atmosfer katmak için kullanılır.
    *   Efekt, genellikle render döngüsünün belirli aşamalarında çağrılır:
        1.  `Compute`: Her frame'de, kamera güncellendikten sonra ışık kaynağının pozisyonunu ve görünürlüğünü hesaplamak için.
        2.  `DrawBeforeFlare`: Ana parlama diskinin çizilmesi (genellikle sahne çizildikten sonra, occlusion testi yapılmadan önce).
        3.  `ReadControlPixels` / `AdjustBrightness`: Derinlik tamponu okunarak occlusion testi yapılır ve parlaklık ayarlanır (genellikle sahne çizildikten sonra).
        4.  `DrawFlare`: Occlusion testine göre ayarlanmış parlaklıkla ikincil parlama parçalarının çizilmesi (genellikle UI'dan önce veya sonra).

### `JpegFile.h` ve `JpegFile.cpp` (JPEG Dosya İşleme Yardımcıları)

*   **Bağımlılık:** Bu dosyalar, JPEG işlemleri için harici `libjpeg-turbo` kütüphanesine bağımlıdır (`#include <libjpeg-turbo/jpeglib.h>`).
*   **Amaç:** `libjpeg-turbo` kütüphanesini kullanarak JPEG formatındaki resim dosyalarını veya bellek içi JPEG verilerini yüklemek (decode) ve ham RGB piksel verilerini JPEG formatına dönüştürerek dosyaya veya belleğe kaydetmek (encode) için bir dizi C fonksiyonu arayüzü sunar.

*   **`JpegFile.h` - Fonksiyon Bildirimleri:**
    *   `jpeg_save(unsigned char* data_d, int width, int height, int quality, const char* filename)`: Verilen ham RGB piksel verisini (`data_d`, 3 bileşenli - RGBRGB...), belirtilen kaliteyle (`quality`, 0-100) bir JPEG dosyasına (`filename`) kaydeder.
    *   `jpeg_save_to_file(unsigned char* data_d, int width, int height, int quality, FILE* fi)`: Ham RGB verisini, önceden açılmış bir dosya işaretçisine (`fi`) JPEG olarak yazar.
    *   `jpeg_save_to_mem(unsigned char* data_d, int width, int height, int quality, unsigned char* dest, int destsize)`: Ham RGB verisini, belirtilen bellek tamponuna (`dest`, `destsize` boyutunda) JPEG olarak yazar. Yazılan gerçek boyutu döndürür.
    *   `jpeg_load(const char* filename, unsigned char** dest, int* width, int* height)`: Belirtilen JPEG dosyasını (`filename`) yükler. Çıktı olarak ayrılan bellek tamponunun işaretçisini (`*dest`), resmin genişliğini (`*width`) ve yüksekliğini (`*height`) döndürür. Yüklenen veri genellikle 32-bit RGBA formatındadır.
    *   `jpeg_load_from_mem(unsigned char* _data, int size_d, unsigned char* dest, int width, int height)`: Bellekteki JPEG verisini (`_data`, `size_d` boyutunda) okur ve önceden ayrılmış hedef bellek tamponuna (`dest`, beklenen `width` ve `height` ile) RGB olarak yazar.

*   **`JpegFile.cpp` - Uygulama Detayları:**
    *   **`libjpeg-turbo` Entegrasyonu:** Fonksiyonlar, `libjpeg-turbo` kütüphanesinin standart yapılarını (`jpeg_compress_struct`, `jpeg_decompress_struct`, `jpeg_error_mgr`) ve fonksiyonlarını (`jpeg_create_compress`, `jpeg_stdio_dest`, `jpeg_mem_dest`, `jpeg_set_defaults`, `jpeg_set_quality`, `jpeg_start_compress`, `jpeg_write_scanlines`, `jpeg_finish_compress`, `jpeg_destroy_compress`, `jpeg_create_decompress`, `jpeg_stdio_src`, `jpeg_mem_src`, `jpeg_read_header`, `jpeg_start_decompress`, `jpeg_read_scanlines`, `jpeg_finish_decompress`, `jpeg_destroy_decompress` vb.) kullanır.
    *   **Hedef Yönetimi (Kaydetme):**
        *   Dosyaya kaydetme işlemleri (`jpeg_save`, `jpeg_save_to_file`), `libjpeg-turbo`'nun standart dosya hedef yöneticisini (`jpeg_stdio_dest`) veya özel tamponlama yapan `file_init_destination`, `file_empty_output_buffer`, `file_term_destination` callback fonksiyonlarını kullanır.
        *   Belleğe kaydetme (`jpeg_save_to_mem`), özel `mem_init_destination`, `mem_empty_output_buffer`, `mem_term_destination` callback fonksiyonlarını kullanarak veriyi doğrudan hedef bellek tamponuna yazar.
    *   **Kaynak Yönetimi (Yükleme):**
        *   Dosyadan yükleme (`jpeg_load`), standart dosya kaynak yöneticisini (`jpeg_stdio_src`) kullanır.
        *   Bellekten yükleme (`jpeg_load_from_mem`), özel `mem_init_source`, `mem_fill_input_buffer`, `mem_skip_input_data`, `mem_term_source` callback fonksiyonlarını kullanarak veriyi doğrudan kaynak bellek tamponundan okur.
    *   **Renk Formatı:** Kaydetme fonksiyonları RGB (3 bileşenli) girdi bekler (`cinfo.in_color_space = JCS_RGB`). Yükleme fonksiyonu (`jpeg_load`) ise çıktı olarak genellikle RGBA (4 bileşenli) formatında bir tampon ayırır ve döndürür (kod içinde `dest[y * width + x].a = 255;` satırı bunu gösterir). `jpeg_load_from_mem` ise RGB (3 bileşenli) çıktı yazar.

*   **Kullanım Alanı:**
    *   JPEG formatındaki resimlerin oyun içine yüklenmesi (örneğin, UI elemanları, yükleme ekranları, bazı dokular) için kullanılır. `jpeg_load` veya `jpeg_load_from_mem` ile ham piksel verisi elde edilir ve bu veri daha sonra bir `CGraphicImage` veya Direct3D dokusu oluşturmak için kullanılabilir.
    *   Oyun içinden ekran görüntüsü alma veya render edilmiş bir sahneyi/doku'yu JPEG olarak kaydetme işlevleri için `jpeg_save`, `jpeg_save_to_file` veya `jpeg_save_to_mem` fonksiyonları kullanılabilir.

### `Input.h` ve `Input.cpp` (DirectInput Klavye Yönetimi)

*   **Bağımlılık:** DirectInput 8 (`dinput.h`, `dinput8.lib`).
*   **Amaç:** Bu dosyalar, Microsoft DirectInput 8 API'sini kullanarak klavye girişini yönetmek için temel sınıfları (`CInputDevice`, `CInputKeyboard`) tanımlar ve uygular. Klavye durumunu okumak ve tuş basma/bırakma olaylarını işlemek için bir altyapı sağlar.

*   **`Input.h` - Tanımlar ve Bildirimler:**
    *   **`CInputDevice` Sınıfı:** DirectInput 8 nesnesinin (`ms_lpDI`) oluşturulması ve yönetilmesi için temel bir sınıf.
        *   `CreateDevice(HWND hWnd)`: DirectInput 8 arayüzünü (`ms_lpDI`) oluşturur (eğer zaten oluşturulmamışsa).
        *   `ms_lpDI` (static `LPDIRECTINPUT8`): Paylaşılan DirectInput 8 nesnesi.
    *   **`CInputKeyboard` Sınıfı:** Klavye cihazını yöneten soyut sınıf.
        *   **Kalıtım:** `public CInputDevice`.
        *   `InitializeKeyboard(HWND hWnd)`: Klavye cihazını (`ms_lpKeyboard`) oluşturur, veri formatını (`c_dfDIKeyboard`) ve işbirliği seviyesini (`SetCooperativeLevel`) ayarlar ve cihazı ele geçirir (`Acquire`).
        *   `UpdateKeyboard()`: Klavye durumunu (`ms_diks`) okur ve `ms_bPressedKey` dizisini güncelleyerek `OnKeyDown`/`OnKeyUp` olaylarını tetikler.
        *   `ResetKeyboard()`: Klavye durum dizilerini sıfırlar.
        *   `IsPressed(int iIndex)`: Belirtilen tuşun (`DIK_...` sabiti) basılı olup olmadığını kontrol eder.
        *   `KeyDown(int iIndex)`, `KeyUp(int iIndex)`: Belirtilen tuşun durumunu günceller ve ilgili `OnKey...` sanal metodunu çağırır.
        *   **Soyut Sanal Metotlar:**
            *   `virtual void OnKeyDown(int iIndex) = 0`: Tuşa basıldığında türetilmiş sınıf tarafından implemente edilmelidir.
            *   `virtual void OnKeyUp(int iIndex) = 0`: Tuş bırakıldığında türetilmiş sınıf tarafından implemente edilmelidir.
        *   **Statik Üyeler:**
            *   `ms_lpKeyboard` (`LPDIRECTINPUTDEVICE8`): Paylaşılan DirectInput klavye cihazı.
            *   `ms_bPressedKey[256]`: Her tuşun basılı olup olmadığını tutan boolean dizi.
            *   `ms_diks[256]`: DirectInput'tan okunan ham klavye durum verisi.

*   **`Input.cpp` - Uygulama Detayları:**
    *   **DirectInput Başlatma (`CInputDevice::CreateDevice`):** `DirectInput8Create` ile ana DirectInput nesnesini oluşturur. Bu nesne statiktir ve `CInputKeyboard` örnekleri tarafından paylaşılır.
    *   **Klavye Cihazı Başlatma (`CInputKeyboard::InitializeKeyboard`):**
        *   `ms_lpDI->CreateDevice(GUID_SysKeyboard, ...)` ile sistem klavye cihazını oluşturur.
        *   `SetDataFormat(&c_dfDIKeyboard)` ile standart klavye veri formatını ayarlar.
        *   `SetCooperativeLevel` ile uygulamanın klavyeyi nasıl kullanacağını belirler (genellikle `DISCL_FOREGROUND | DISCL_NONEXCLUSIVE`, yani uygulama ön plandayken özel olmayan erişim).
        *   `Acquire()` ile klavye girdisini almaya başlar.
    *   **Klavye Durumu Güncelleme (`CInputKeyboard::UpdateKeyboard`):**
        *   `ms_lpKeyboard->GetDeviceState` ile anlık klavye durumunu `ms_diks` dizisine okur.
        *   Eğer cihaz kaybedilmişse (`FAILED(hr)`), tekrar `Acquire()` yapmayı dener.
        *   `ms_diks` dizisindeki her tuşu kontrol eder:
            *   Eğer tuş basılıysa (`ms_diks[i] & 0x80`) ve daha önce basılı değilse (`!IsPressed(i)`), `KeyDown(i)` çağrılır.
            *   Eğer tuş basılı değilse ve daha önce basılıysa (`IsPressed(i)`), `KeyUp(i)` çağrılır.
    *   **Olay Yönetimi (`KeyDown`, `KeyUp`):** İlgili tuşun `ms_bPressedKey` dizisindeki durumunu günceller ve türetilmiş sınıfın implemente ettiği `OnKeyDown` veya `OnKeyUp` sanal metodunu çağırarak olayı bildirir.

*   **Kullanım Alanı:**
    *   `CInputKeyboard` sınıfı, klavye girdilerini almak için temel bir arayüz sağlar.
    *   Genellikle oyunun ana giriş yönetim sistemi (örneğin, `CPythonPlayer` veya benzeri bir sınıf) `CInputKeyboard`'dan miras alır.
    *   Türetilmiş sınıf, `InitializeKeyboard`'ı çağırarak klavyeyi başlatır ve her frame'de `UpdateKeyboard`'ı çağırarak klavye durumunu günceller.
    *   `OnKeyDown` ve `OnKeyUp` metotlarını override ederek, hangi tuşa basıldığında veya bırakıldığında hangi oyun içi aksiyonun (hareket, yetenek kullanımı, UI etkileşimi vb.) tetikleneceğini tanımlar.

### `lineintersect_utils.h` ve `lineintersect_utils.cpp` (3D Doğru Parçası Kesişim Yardımcıları)

*   **Kaynak ve Telif Hakkı:** Bu dosyalar orijinal olarak Graham Rhodes tarafından yazılmış ve "Game Programming Gems II" kitabının "Fast, Robust Intersection of 3D Line Segments" bölümünde yer alan koda dayanmaktadır. Telif hakkı Graham Rhodes, 2001'e aittir.
*   **Amaç:** Bu dosyalar, 3D uzayda iki doğru parçasının (veya sonsuz doğruların) kesişimini veya birbirine en yakın noktalarını bulan bir dizi yardımcı fonksiyon sunar. Çeşitli özel durumları (dejenere doğru parçaları, paralel doğrular) ele alır.

*   **`lineintersect_utils.h` - Fonksiyon Bildirimleri:**
    *   `IntersectLineSegments(const D3DXVECTOR3& A1, const D3DXVECTOR3& A2, const D3DXVECTOR3& B1, const D3DXVECTOR3& B2, D3DXVECTOR3& OutA, D3DXVECTOR3& OutB)`: İki doğru parçasının (A1-A2 ve B1-B2) birbirine en yakın noktalarını (`OutA` ve `OutB`) bulur. Bu, `D3DXVECTOR3` kullanan daha modern bir arayüzdür ve varsayılan olarak sonlu doğru parçalarıyla çalışır.
    *   `IntersectLineSegments(const float A1x, ..., bool infinite_lines, float epsilon, float& PointOnSegAx, ..., bool& true_intersection)`: Daha eski, float tabanlı ve çok sayıda çıktı parametresi olan bir arayüz. İki doğru parçasının birbirine en yakın noktalarını (`PointOnSegA`, `PointOnSegB`), bu noktaların ortalamasını (`NearestPointX/Y/Z`), aralarındaki vektörü (`NearestVectorX/Y/Z`) ve gerçekten kesişip kesişmediklerini (`true_intersection`) hesaplar. `infinite_lines` parametresiyle sonsuz doğrular olarak değerlendirilip değerlendirilmeyeceklerini kontrol eder.
    *   `IntersectLineSegments(const float A1x, ..., bool infinite_lines, float epsilon, float& PointOnSegAx, ..., float& PointOnSegBz)`: Önceki fonksiyonun sadece en yakın noktaları döndüren daha sade bir versiyonu.
    *   `FindNearestPointOnLineSegment(const float A1x, ..., bool infinite_line, float epsilon_squared, float& NearestPointX, ..., float& parameter)`: Bir doğru parçası (veya sonsuz doğru) üzerinde, verilen bir noktaya (Bx, By, Bz) en yakın olan noktayı bulur. Çıktı olarak en yakın noktayı ve bu noktanın doğru parçası üzerindeki parametrik konumunu (0 ile 1 arasında) döndürür.
    *   `FindNearestPointOfParallelLineSegments(...)`: Paralel olduğu bilinen iki doğru parçasının birbirine en yakın noktalarını bulur.
    *   `AdjustNearestPoints(...)`: Sonsuz doğrular için bulunan en yakın noktaların parametrelerini (`s` ve `t`), sonlu doğru parçalarına uyacak şekilde ayarlar.

*   **`lineintersect_utils.cpp` - Uygulama Detayları:**
    *   **Matematiksel Temel:** Fonksiyonlar, Graham Rhodes'un makalesinde açıklanan geometrik ve vektörel cebir prensiplerine dayanır. İki doğru arasındaki en kısa mesafeyi bulmak için parametrik doğru denklemlerini ve vektör projeksiyonlarını kullanır.
    *   **Ana Kesişim Mantığı (`IntersectLineSegments`):**
        1.  **Dejenere Kontrolü:** Önce her iki doğru parçasının da dejenere olup olmadığını (yani uzunluklarının bir epsilon değerinden küçük olup olmadığını) kontrol eder. Eğer biri dejenere ise, diğer doğru parçası üzerindeki o noktaya en yakın noktayı bulur (`FindNearestPointOnLineSegment` çağrılır).
        2.  **Paralellik Kontrolü:** İki doğru parçasının yön vektörleri (`La`, `Lb`) arasındaki determinantı (`DetL`) hesaplar. Eğer `DetL` epsilon değerine yakınsa, doğrular paralel kabul edilir ve `FindNearestPointOfParallelLineSegments` çağrılır.
        3.  **Genel Durum (Kesişen veya Aykırı Doğrular):**
            *   İki doğru üzerindeki en yakın noktaların parametreleri olan `s` (A doğrusu için) ve `t` (B doğrusu için) hesaplanır. Bu hesaplama, doğrusal denklem sisteminin çözülmesini içerir.
            *   Eğer `infinite_lines` `true` ise veya hesaplanan `s` ve `t` parametreleri [0, 1] aralığındaysa, en yakın noktalar doğrudan bu parametreler kullanılarak bulunur.
            *   Eğer `infinite_lines` `false` ise ve `s` veya `t` (ya da her ikisi) [0, 1] aralığının dışındaysa, en yakın noktaları doğru parçalarının uç noktalarına veya diğer doğru parçası üzerindeki en yakın noktaya göre ayarlamak için `AdjustNearestPoints` çağrılır.
    *   **`FindNearestPointOnLineSegment`:** Verilen noktanın doğru parçasına olan vektörünü, doğru parçasının yön vektörü üzerine projekte ederek parametreyi bulur. Eğer sonlu bir doğru parçasıysa, parametreyi [0, 1] aralığına sınırlar.
    *   **`FindNearestPointOfParallelLineSegments`:** Paralel doğrular durumunda, bir doğrunun uç noktalarının diğer doğruya olan izdüşümlerini veya segmentlerin birbiriyle örtüşme durumunu analiz ederek en yakın noktaları belirler. Genellikle bir segment üzerindeki noktanın diğer segmente olan mesafesini minimize etmeye çalışır.
    *   **`AdjustNearestPoints`:** `s` ve `t` parametrelerinin [0,1] aralığının dışında olduğu durumları ele alır. Örneğin, eğer `s < 0` ise A segmentindeki en yakın nokta A1 olur; eğer `s > 1` ise A2 olur. Bu durumda, bu sabitlenmiş nokta için diğer segmentteki en yakın nokta yeniden hesaplanır ve gerekirse o da sabitlenir.
    *   **Epsilon Değerleri:** Fonksiyonlar, kayan nokta karşılaştırmalarında ve dejenere/paralel durumların tespitinde küçük bir `epsilon` (veya `epsilon_squared`) tolerans değeri kullanır.
    *   **Makrolar:** `FMAX`, `FMIN`, `FABS`, `OUT_OF_RANGE` gibi yardımcı makrolar kullanılır.
    *   **D3DXVECTOR3 Kullanımı:** Modern `IntersectLineSegments` fonksiyonu ve iç yardımcı fonksiyonlar (`FindNearestPointOnLineSegment`, `FindNearestPointOfParallelLineSegments`, `AdjustNearestPoints`) `D3DXVECTOR3` yapılarını ve ilgili D3DX matematik fonksiyonlarını (`D3DXVec3LengthSq`, `D3DXVec3Dot`, operatörler) yoğun bir şekilde kullanır.

*   **Kullanım Alanı:**
    *   Bu fonksiyonlar, genellikle 3D oyunlarda ve simülasyonlarda çarpışma tespiti (collision detection) ve yanıtı için kullanılır.
    *   Örneğin, bir karakterin veya merminin bir engelle (başka bir karakter, duvar vb.) kesişip kesişmediğini veya ne kadar yaklaştığını belirlemek için kullanılabilir.
    *   Yapay zeka (AI) için görüş hattı (line-of-sight) kontrollerinde veya yol bulma algoritmalarında yardımcı olabilir.
    *   3D modelleme veya düzenleme araçlarında nesnelerin birbirine göre konumlandırılması veya hizalanması için kullanılabilir.
    *   İki hareketli nesnenin gelecekteki çarpışma noktalarını tahmin etmede temel bir bileşen olabilir.

### `IME.h` ve `IME.cpp` (`CIME` Sınıfı ve TSF Entegrasyonu)

*   **Amaç:** Bu dosyalar, Windows Input Method Editor (IME) ve Text Services Framework (TSF) ile etkileşim kurarak oyun içinde karmaşık metin girişlerini (özellikle Asya dilleri için) yöneten `CIME` sınıfını tanımlar ve uygular. Klavye düzeni değişikliklerini, IME durumunu (açık/kapalı, İngilizce/yerel mod), kompozisyon (composition) dizesini, aday listesini (candidate list) ve okuma penceresini (reading window) ele alır.

*   **Temel Yapılar ve Sınıflar:**
    *   **`IIMEEventSink` Arayüzü (`IME.h`):** `CIME` sınıfının olaylarını (WM_CHAR, metin güncellemesi, kod sayfası değişikliği, aday listesi/okuma penceresi açma/kapama) işlemek için türetilmiş sınıfların implemente etmesi gereken bir arayüz tanımlar.
    *   **`CTsfUiLessMode` Sınıfı (`IME.cpp` içinde):** Text Services Framework (TSF) kullanarak UI'sız modda IME olaylarını yönetmek için bir yardımcı sınıftır. Özellikle Vista ve sonrası işletim sistemlerinde IMM (Input Method Manager) API'lerinin bazı uyumluluk sorunlarını aşmak için kullanılır.
        *   **`CUIElementSink` (iç sınıf):** `ITfUIElementSink` (okuma ve aday penceresi olayları için), `ITfInputProcessorProfileActivationSink` (klavye düzeni/dil değişikliği olayları için) ve `ITfCompartmentEventSink` (IME açık/kapalı durumu değişikliği olayları için) arayüzlerini implemente eden bir TSF olay dinleyicisi (sink).
    *   **`CDisableCicero` Sınıfı (`IME.cpp` içinde):** Gerekirse Cicero (TSF'nin eski adı) metin servislerini belirli bir pencere için devre dışı bırakmaya çalışan bir yardımcı sınıftır.

*   **`CIME` Sınıfı Temel Özellikleri:**
    *   **Statik Üyeler:** IME durumu, klavye düzeni, aktif pencere, TSF durumu, aday listesi verileri, okuma penceresi verileri, gösterge metni gibi birçok global IME bilgisini tutar. `ms_pEvent` ile `IIMEEventSink` olaylarını iletir.
    *   **Başlatma ve Sonlandırma:**
        *   `Initialize(HWND hWnd)`: Ana pencereyi (`ms_hWnd`) alır, işletim sistemi sürümünü kontrol eder, `imm32.dll`'den gerekli IMM fonksiyonlarını yükler, TSF sink'lerini (`CTsfUiLessMode::SetupSinks`) kurar ve mevcut giriş dilini/durumunu kontrol eder.
        *   `Uninitialize()`: TSF sink'lerini serbest bırakır, pencere ile IME ilişkisini keser ve yüklenen DLL'leri serbest bırakır.
    *   **IME Kontrolü:**
        *   `EnableIME(bool bEnable)` / `DisableIME()`: Belirtilen pencere için IME'yi aktif eder veya devre dışı bırakır (`ImmAssociateContext`).
        *   `EnableCaptureInput()` / `DisableCaptureInput()`: `CIME`'nin klavye ve IME mesajlarını işleyip işlemeyeceğini kontrol eden bir bayrak (`ms_bCaptureInput`).
        *   `SetInputMode(DWORD dwMode)` / `GetInputMode()`: IME'nin dönüşüm modunu (örn: tam/yarım genişlik, yerel/alfanümerik) ayarlar/alır.
        *   `SetNumberMode()` / `SetStringMode()`: Yalnızca sayı girişine izin veren özel bir mod.
    *   **Metin Yönetimi:**
        *   `SetText(const char* c_szText, int len)`: Mevcut metin içeriğini ayarlar.
        *   `GetText(std::string& rstrText, bool addCodePage)`: Nihai metni (kompozisyon öncesi + kompozisyon + kompozisyon sonrası) bir string olarak alır. Gerekirse kod sayfası bilgisini (`@CPAGE`) ekler.
        *   `PasteTextFromClipBoard()`: Panodan metin yapıştırır, zararlı olabilecek etiketleri filtreler.
        *   `PasteString(const char* str)`: Verilen string'i mevcut metne ekler.
        *   `FinalizeString(bool bSend)`: Mevcut kompozisyon dizesini sonlandırır.
        *   `IncCurPos()`, `DecCurPos()`, `SetCurPos(int offset)`, `DelCurPos()`: Metin içindeki imleç pozisyonunu yönetir (metin etiketlerini dikkate alarak).
    *   **Olay İşleme (WM Mesajları):** `WMInputLanguage`, `WMStartComposition`, `WMComposition`, `WMEndComposition`, `WMNotify`, `WMChar` gibi Windows mesajlarını işleyerek IME durumunu günceller ve ilgili olayları tetikler.
    *   **Kompozisyon Dizesi (`m_wszComposition`, `ms_compLen`):** Kullanıcının girdiği ancak henüz sonlandırmadığı karakterleri tutar. `CompositionProcess`, `CompositionProcessBuilding` ile güncellenir.
    *   **Sonuç Dizesi (`m_wText`, `ms_curpos`, `ms_lastpos`):** IME tarafından sonlandırılmış ve metin alanına eklenmiş karakterleri tutar. `ResultProcess` ile güncellenir.
    *   **Aday Listesi (`ms_wszCandidate`, `ms_dwCandidateCount`, vb.):** IME'nin sunduğu giriş adaylarını yönetir. `CandidateProcess` (IMM için) veya `CTsfUiLessMode::MakeCandidateStrings` (TSF için) ile güncellenir.
    *   **Okuma Penceresi/Bilgisi (`ms_wstrReading`, `ms_bHorizontalReading`, vb.):** Özellikle bazı CJK (Çince, Japonca, Korece) IME'lerde görünen telaffuz veya yardımcı karakter bilgilerini yönetir. `ReadingProcess` (IMM için) veya `CTsfUiLessMode::MakeReadingInformationString` (TSF için) ile güncellenir.
    *   **Dil ve Kod Sayfası Yönetimi:** `GetCodePageFromLang`, `CheckInputLocale`, `ChangeInputLanguage`, `ms_uInputCodePage`, `ms_uOutputCodePage`. Aktif klavye düzenine göre giriş ve çıkış kod sayfalarını belirler ve yönetir.
    *   **IME ID ve Yetenekleri (`GetImeId`, `SetupImeApi`):** Aktif IME'nin türünü ve sürümünü belirleyerek bazı özel davranışları (örneğin, okuma penceresi API'sinin varlığı) yönetir.

*   **`CTsfUiLessMode` Detayları:**
    *   TSF arayüzlerini (`ITfThreadMgrEx`, `ITfUIElementSink`, `ITfInputProcessorProfileActivationSink`, `ITfCompartmentEventSink`) kullanarak IME olaylarını dinler.
    *   `OnActivated`: Klavye düzeni veya IME profili değiştiğinde tetiklenir.
    *   `OnChange`: IME'nin açık/kapalı durumu veya dönüşüm modu değiştiğinde tetiklenir.
    *   `BeginUIElement`, `UpdateUIElement`, `EndUIElement`: Okuma penceresi veya aday listesi gibi UI elemanları gösterildiğinde, güncellendiğinde veya gizlendiğinde tetiklenir. Bu fonksiyonlar içinde `MakeReadingInformationString` ve `MakeCandidateStrings` çağrılarak `CIME`'nin ilgili statik üyeleri güncellenir.

*   **Ana Çalışma Prensibi:**
    1.  `CIME::Initialize` ile sistem başlatılır. Pencereye bir IME context'i atanır veya TSF sink'leri kurulur.
    2.  Kullanıcı klavye düzenini değiştirdiğinde (`WM_INPUTLANGCHANGE` veya TSF `OnActivated`) `ChangeInputLanguage` çağrılır, kod sayfaları ve IME'ye özgü ayarlar güncellenir.
    3.  Kullanıcı metin girerken:
        *   Eğer bir IME aktifse, işletim sistemi `WM_IME_STARTCOMPOSITION`, `WM_IME_COMPOSITION`, `WM_IME_ENDCOMPOSITION` ve `WM_IME_NOTIFY` (aday listesi, okuma penceresi değişiklikleri için) mesajlarını gönderir. `CIME`, bu mesajları işleyerek kompozisyon dizesini, aday listesini vb. günceller ve `IIMEEventSink` aracılığıyla olayları bildirir.
        *   TSF modunda ise benzer olaylar `CTsfUiLessMode` içindeki sink'ler tarafından yakalanır ve `CIME`'nin statik üyeleri güncellenir.
        *   Kullanıcı bir karakteri sonlandırdığında (örneğin, Enter'a basarak veya aday listesinden seçerek), GCS_RESULTSTR ile gelen metin `ResultProcess` ile `m_wText`'e eklenir.
        *   Normal karakter girişleri (`WM_CHAR`) `OnChar` ile işlenir ve doğrudan `m_wText`'e eklenir.
    4.  Metin alanının güncellenmesi gerektiğinde (örneğin, render için), `GetText` çağrılarak son birleştirilmiş metin alınır.

*   **Kullanım Alanı:**
    *   Oyun içindeki tüm metin giriş alanları (sohbet, isim girme, arama kutuları vb.) için temel IME desteğini sağlar.
    *   Kullanıcının işletim sistemindeki IME'yi kullanarak kendi dilinde (özellikle Çince, Japonca, Korece gibi dillerde) sorunsuz bir şekilde metin girebilmesini mümkün kılar.
    *   Aday listesi, okuma penceresi gibi IME'ye özgü UI elemanlarının bilgilerini alarak oyunun kendi arayüzünde özel bir şekilde gösterilmesine olanak tanır (veya IME'nin varsayılan UI'sını kullanır).
    *   Kod sayfası dönüşümlerini ve metin etiketlerini (`TextTag.h` ile entegre) yöneterek farklı dillerdeki metinlerin doğru görüntülenmesine yardımcı olur.

### `GrpVertexShader.h` ve `GrpVertexShader.cpp` (`CVertexShader` Sınıfı)

*   **Amaç:** Bu dosyalar, Direct3D 8 için vertex shader (köşe gölgelendirici) programlarını diskten yüklemek, derlemek ve yönetmek üzere `CVertexShader` sınıfını tanımlar ve uygular. Vertex shader'lar, 3D modellerin köşe (vertex) verilerini (pozisyon, normal, renk, doku koordinatları vb.) işleyerek ve dönüştürerek özel görsel efektler, animasyonlar veya aydınlatma hesaplamaları yapmak için kullanılır.

*   **`GrpVertexShader.h` - Tanımlar ve Bildirimler:**
    *   **`CVertexShader` Sınıfı:**
        *   **Kalıtım:** `public CGraphicBase`. (Bu, `CGraphicBase`'in statik üyelerine, özellikle Direct3D cihazına (`ms_lpd3dDevice`) erişim sağlar.)
        *   Kurucu (`CVertexShader()`), Yıkıcı (`virtual ~CVertexShader()`).
        *   `Destroy()`: Oluşturulan vertex shader Direct3D kaynağını siler.
        *   `CreateFromDiskFile(const char* c_szFileName, const DWORD* c_pdwVertexDecl)`: Belirtilen dosyadan (genellikle `.vsh` veya benzeri assembly formatında) bir vertex shader programını derler ve oluşturur. `c_pdwVertexDecl`, bu shader'ın beklediği vertex yapısının (vertex declaration) tanımıdır.
        *   `Set()`: Bu vertex shader'ı Direct3D render pipeline'ı için aktif hale getirir (`STATEMANAGER` aracılığıyla).
    *   **Korumalı Metot:**
        *   `Initialize()`: Üyeleri (özellikle `m_handle`) başlangıç durumuna getirir.
    *   **Korumalı Üye:**
        *   `m_handle`: Oluşturulan Direct3D vertex shader'ının tanıtıcısı (`DWORD`).

*   **`GrpVertexShader.cpp` - Uygulama Detayları:**
    *   **Başlatma ve Yok Etme:**
        *   Kurucu `Initialize()`'ı çağırır, bu da `m_handle`'ı 0 yapar.
        *   Yıkıcı `Destroy()`'ı çağırır.
        *   `Destroy()`, eğer `m_handle` geçerliyse ve Direct3D cihazı (`ms_lpd3dDevice`) mevcutsa, `ms_lpd3dDevice->DeleteVertexShader(m_handle)` çağırarak Direct3D vertex shader kaynağını serbest bırakır ve `m_handle`'ı 0 yapar.
    *   **Oluşturma (`CreateFromDiskFile`):**
        1.  Önce varsa mevcut shader'ı `Destroy()` ile siler.
        2.  `D3DXAssembleShaderFromFile` fonksiyonunu kullanarak verilen dosyadaki (`c_szFileName`) vertex shader assembly kodunu derler. Derlenmiş shader kodu `lpd3dxShaderBuffer` içine, olası derleme hataları ise `lpd3dxErrorBuffer` içine yazılır.
        3.  Derleme başarısız olursa (`FAILED`) `false` döner.
        4.  `CDirect3DXBuffer` (muhtemelen `lpd3dxShaderBuffer` ve `lpd3dxErrorBuffer` için bir RAII sarmalayıcı) ile D3DX buffer'larının yönetimi kolaylaştırılır.
        5.  `ms_lpd3dDevice->CreateVertexShader` fonksiyonunu çağırarak derlenmiş shader kodundan (`(DWORD*)shaderBuffer.GetPointer()`) ve verilen vertex bildirimiyle (`c_pdwVertexDecl`) bir Direct3D vertex shader nesnesi oluşturur. Oluşturulan shader'ın tanıtıcısı `m_handle` üyesinde saklanır.
        6.  Vertex shader oluşturma işlemi başarısız olursa `false` döner.
        7.  Başarılı olursa `true` döner.
    *   **Aktif Etme (`Set`):**
        *   `STATEMANAGER.SetVertexShader(m_handle)` çağrısını yaparak bu shader'ı render pipeline'ı için aktif vertex shader olarak ayarlar.

*   **Kullanım Alanı:**
    *   `CVertexShader` sınıfı, özel vertex tabanlı efektler ve işlemler uygulamak için kullanılır.
    *   Genellikle bir materyalin veya özel bir render geçişinin parçası olarak, belirli 3D nesnelerin (örneğin, karakterler, arazi, özel efekt geometrileri) nasıl işleneceğini özelleştirmek amacıyla kullanılır.
    *   Kullanım senaryoları şunları içerebilir:
        *   **Özel Animasyonlar:** Kemik tabanlı animasyon (skinning), vertex morphing veya prosedürel animasyonlar.
        *   **Gelişmiş Aydınlatma:** Vertex başına (per-vertex) aydınlatma hesaplamaları, normal haritalama (normal mapping) için gerekli teğet uzayı (tangent space) hesaplamaları.
        *   **Geometrik Deformasyonlar:** Rüzgarla dalgalanan bitkiler, su yüzeyi dalgalanmaları gibi efektler.
        *   **Parçacık Sistemleri:** Parçacıkların hareketini ve davranışını kontrol etmek.
    *   Bir vertex shader dosyası (`.vsh`) oluşturulduktan sonra, bu dosya `CreateFromDiskFile` ile yüklenir ve bir `CVertexShader` nesnesi oluşturulur. Render sırasında, ilgili nesneler çizilmeden önce `CVertexShader::Set()` metodu çağrılarak bu özel vertex shader aktif hale getirilir. Ardından geometrinin çizimi yapılır ve gerekirse varsayılan vertex işleme (fixed-function pipeline veya başka bir shader) geri dönülür.

### `GrpVertexBufferStatic.h` ve `GrpVertexBufferStatic.cpp` (`CStaticVertexBuffer` Sınıfı)

*   **Amaç:** Bu dosyalar, statik (içeriği genellikle bir kez yüklenip nadiren veya hiç değişmeyen) vertex verilerini tutmak için kullanılan `CStaticVertexBuffer` sınıfını tanımlar ve uygular. `CGraphicVertexBuffer` temel sınıfından miras alır.

*   **`GrpVertexBufferStatic.h` - Tanımlar ve Bildirimler:**
    *   **`CStaticVertexBuffer` Sınıfı:**
        *   **Kalıtım:** `public CGraphicVertexBuffer`.
        *   Kurucu (`CStaticVertexBuffer()`), Yıkıcı (`virtual ~CStaticVertexBuffer()`).
        *   `Create(int vtxCount, DWORD fvf, bool isManaged = true)`: Belirtilen sayıda köşe (`vtxCount`) ve FVF (Flexible Vertex Format) kodu (`fvf`) ile bir statik vertex buffer oluşturur. `isManaged` parametresi teorik olarak buffer'ın yönetilip yönetilmeyeceğini kontrol edebilir, ancak mevcut implementasyonda her zaman `D3DPOOL_MANAGED` kullanılır.

*   **`GrpVertexBufferStatic.cpp` - Uygulama Detayları:**
    *   **`Create` Metodu:**
        *   Temel sınıf olan `CGraphicVertexBuffer::Create` metodunu çağırır.
        *   Bu çağrıda, `Usage` parametresini `D3DUSAGE_WRITEONLY` (buffer'a sadece yazılacak, okunmayacak) ve `Pool` parametresini `D3DPOOL_MANAGED` olarak sabit bir şekilde ayarlar.
        *   `D3DPOOL_MANAGED`, Direct3D'nin buffer'ı sistem belleğinde tutmasını ve gerektiğinde otomatik olarak video belleğine kopyalamasını sağlar. Bu, cihaz kaybolması (lost device) durumlarında buffer içeriğinin otomatik olarak geri yüklenmesine yardımcı olur.
        *   `isManaged` parametresi fonksiyona geçirilmesine rağmen, `Create` içindeki implementasyonda bu parametre kullanılmaz ve her zaman `D3DPOOL_MANAGED` havuzu tercih edilir.
    *   **Kurucu ve Yıkıcı:** Boştur, temel işlevler `CGraphicVertexBuffer` tarafından sağlanır.

*   **Kullanım Alanı:**
    *   `CStaticVertexBuffer`, oyun dünyasındaki statik geometrilerin (örneğin, binalar, sabit çevre nesneleri, değişmeyen UI elemanları) köşe verilerini depolamak için kullanılır.
    *   Veriler genellikle bir kez yüklenir (örneğin, bir model dosyasından okunarak `Lock` ve `Unlock` ile buffer'a yazılır) ve render döngüsü boyunca değişmez.
    *   `D3DPOOL_MANAGED` kullanımı, bu tür statik veriler için genellikle iyi bir tercihtir çünkü cihaz kaybolması gibi durumları daha kolay yönetir, ancak sık güncellenen veriler için performanslı olmayabilir (bu tür veriler için `CDynamicVertexBuffer` daha uygundur).
    *   Render sırasında, `CStaticVertexBuffer::SetStreamSource` (temel sınıftan miras alınır) çağrılarak bu buffer aktif vertex kaynağı olarak ayarlanır ve ardından çizim komutları verilir.

### `GrpVertexBufferDynamic.h` ve `GrpVertexBufferDynamic.cpp` (`CDynamicVertexBuffer` Sınıfı)

*   **Amaç:** Bu dosyalar, dinamik (içeriği her frame veya sık sık değişebilen) vertex verilerini tutmak için kullanılan `CDynamicVertexBuffer` sınıfını tanımlar ve uygular. `CGraphicVertexBuffer` temel sınıfından miras alır.

*   **`GrpVertexBufferDynamic.h` - Tanımlar ve Bildirimler:**
    *   **`CDynamicVertexBuffer` Sınıfı:**
        *   **Kalıtım:** `public CGraphicVertexBuffer`.
        *   Kurucu (`CDynamicVertexBuffer()`), Yıkıcı (`virtual ~CDynamicVertexBuffer()`).
        *   `Create(int vtxCount, int fvf)`: Belirtilen sayıda köşe (`vtxCount`) ve FVF (Flexible Vertex Format) kodu (`fvf`) ile bir dinamik vertex buffer oluşturur. Eğer mevcut bir buffer varsa ve istenen boyut/FVF ile uyumluysa, yeni buffer oluşturmak yerine mevcut olanı kullanabilir.
    *   **Korumalı Üyeler:**
        *   `m_vtxCount`: Buffer'daki köşe sayısı.
        *   `m_fvf`: Buffer'ın FVF kodu.

*   **`GrpVertexBufferDynamic.cpp` - Uygulama Detayları:**
    *   **`Create` Metodu:**
        1.  Eğer zaten bir Direct3D vertex buffer (`m_lpd3dVB`) varsa:
            *   Mevcut buffer'ın FVF'si (`m_fvf`) istenen FVF (`fvf`) ile aynı mı diye kontrol eder.
            *   Eğer FVF'ler aynıysa, mevcut buffer'ın kapasitesi (`m_vtxCount`) istenen köşe sayısından (`vtxCount`) büyük veya eşit mi diye kontrol eder.
            *   Eğer her iki koşul da sağlanıyorsa (yani mevcut buffer yeniden kullanılabiliyorsa), `true` döndürerek yeni buffer oluşturma işlemini atlar.
        2.  Eğer yeniden kullanılamıyorsa veya ilk oluşturma ise, `m_vtxCount` ve `m_fvf` üyelerini günceller.
        3.  Temel sınıf olan `CGraphicVertexBuffer::Create` metodunu çağırır.
        4.  Bu çağrıda, `Usage` parametresini `D3DUSAGE_DYNAMIC` ve `Pool` parametresini `D3DPOOL_SYSTEMMEM` olarak sabit bir şekilde ayarlar.
            *   `D3DUSAGE_DYNAMIC`: Buffer'ın içeriğinin sık sık güncelleneceğini belirtir ve genellikle `Lock` sırasında `D3DLOCK_DISCARD` bayrağı ile birlikte kullanılır.
            *   `D3DPOOL_SYSTEMMEM`: Buffer'ın sistem belleğinde (RAM) tutulacağını belirtir. Dinamik buffer'lar için bu genellikle daha performanslıdır çünkü GPU'ya veri transferi optimize edilebilir (örneğin, AGP veya PCI Express üzerinden doğrudan erişim).
    *   **Kurucu:** `m_vtxCount` ve `m_fvf` üyelerini 0 olarak başlatır.
    *   **Yıkıcı:** Boştur, temel temizlik `CGraphicVertexBuffer` tarafından yapılır.

*   **Kullanım Alanı:**
    *   `CDynamicVertexBuffer`, içeriği sık sık güncellenen vertex verileri için idealdir. Örneğin:
        *   Parçacık sistemleri (her frame değişen parçacık pozisyonları).
        *   Arayüz (UI) elemanları veya 2D çizimler için kullanılan geçici geometriler (örneğin, `CGraphicBase::SetPDTStream` tarafından kullanılan buffer'lar).
        *   Prosedürel olarak üretilen veya her frame deforme olan geometriler.
        *   Debug çizimleri için anlık oluşturulan geometriler.
    *   Veriler genellikle her kullanım öncesinde veya her frame'de `Lock` (genellikle `D3DLOCK_DISCARD` ile) edilerek güncellenir ve ardından `Unlock` edilir.
    *   `D3DPOOL_SYSTEMMEM` ve `D3DUSAGE_DYNAMIC` kombinasyonu, bu tür sık güncellemeler için Direct3D tarafından önerilen bir yaklaşımdır.

### `GrpVertexBuffer.h` ve `GrpVertexBuffer.cpp` (`CGraphicVertexBuffer` Sınıfı)

*   **Amaç:** Bu dosyalar, Direct3D vertex buffer (`LPDIRECT3DVERTEXBUFFER8`) nesneleri için temel bir sarmalayıcı (wrapper) sınıf olan `CGraphicVertexBuffer`'ı tanımlar ve uygular. Bu sınıf, vertex buffer'ların oluşturulması, yok edilmesi, kilitlenmesi (veriye erişim için), kilidin açılması ve render pipeline'ına akış kaynağı olarak ayarlanması gibi temel işlemleri yönetir. `CStaticVertexBuffer` ve `CDynamicVertexBuffer` sınıfları için temel sınıf görevi görür.

*   **`GrpVertexBuffer.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicVertexBuffer` Sınıfı:**
        *   **Kalıtım:** `public CGraphicBase`. (Bu, `CGraphicBase`'in statik üyelerine, özellikle Direct3D cihazına (`ms_lpd3dDevice`) erişim sağlar.)
        *   Kurucu (`CGraphicVertexBuffer()`), Yıkıcı (`virtual ~CGraphicVertexBuffer()`).
        *   `Destroy()`: Vertex buffer'ı ve ilişkili Direct3D kaynağını yok eder.
        *   `virtual bool Create(int vtxCount, DWORD fvf, DWORD usage, D3DPOOL d3dPool)`: Belirtilen parametrelerle yeni bir Direct3D vertex buffer oluşturur. Bu, türetilmiş sınıflar tarafından çağrılır.
        *   **Cihaz Nesneleri Yönetimi:**
            *   `bool CreateDeviceObjects()`: Direct3D vertex buffer nesnesini (`m_lpd3dVB`) oluşturur. `Create` metodu tarafından dahili olarak çağrılır.
            *   `void DestroyDeviceObjects()`: Direct3D vertex buffer nesnesini serbest bırakır.
        *   `bool Copy(int bufSize, const void* srcVertices)`: Verilen kaynak veriyi (`srcVertices`, `bufSize` boyutunda) vertex buffer'ın içine kopyalar. Bu işlem için buffer'ı kilitler ve sonra kilidini açar.
        *   **Kilitleme/Kilit Açma Metotları:**
            *   `bool LockRange(unsigned count, void** pretVertices) const`: Buffer'ın başlangıcından itibaren belirtilen `count` sayıda vertex'lik bir bölümünü kilitler. Kilitlenmiş belleğe işaretçiyi `pretVertices` ile döndürür.
            *   `bool Lock(void** pretVertices) const`: Tüm vertex buffer'ı kilitler (const versiyon).
            *   `bool Unlock() const`: Kilitlenmiş buffer'ın kilidini açar (const versiyon).
            *   `bool LockDynamic(void** pretVertices)`: (Muhtemelen dinamik buffer'lar için özel bir kilitleme, `Lock(0,0, &ptr, 0)` çağırır)
            *   `virtual bool Lock(void** pretVertices)`: Tüm vertex buffer'ı kilitler (non-const versiyon).
            *   `bool Unlock()`: Kilitlenmiş buffer'ın kilidini açar (non-const versiyon).
        *   `void SetStream(int stride, int layer = 0) const`: Bu vertex buffer'ı, belirtilen `stride` (bir vertex'in bayt cinsinden boyutu) ile belirtilen stream katmanına (`layer`) akış kaynağı olarak ayarlar (`STATEMANAGER` aracılığıyla).
        *   **Bilgi Alma Metotları:**
            *   `int GetVertexCount() const`: Buffer'daki toplam vertex sayısını döndürür.
            *   `int GetVertexStride() const`: Bir vertex'in FVF'ye göre bayt cinsinden boyutunu döndürür (`D3DXGetFVFVertexSize`).
            *   `DWORD GetFlexibleVertexFormat() const`: Buffer'ın FVF (Flexible Vertex Format) kodunu döndürür.
            *   `inline LPDIRECT3DVERTEXBUFFER8 GetD3DVertexBuffer() const`: Dahili Direct3D vertex buffer nesnesine işaretçi döndürür.
            *   `inline DWORD GetBufferSize() const`: Buffer'ın toplam boyutunu (bayt cinsinden) döndürür.
            *   `bool IsEmpty() const`: Buffer'ın oluşturulup oluşturulmadığını (yani `m_lpd3dVB`'nin geçerli olup olmadığını) kontrol eder.
    *   **Korumalı Metot:**
        *   `Initialize()`: Üye değişkenleri başlangıç durumuna getirir.
    *   **Korumalı Üye:**
        *   `m_lpd3dVB`: Asıl Direct3D vertex buffer nesnesine işaretçi.
        *   `m_dwBufferSize`: Buffer'ın toplam boyutu (bayt).
        *   `m_dwFVF`: Vertex formatı (FVF).
        *   `m_dwUsage`: Buffer kullanım amacı (örn: `D3DUSAGE_WRITEONLY`, `D3DUSAGE_DYNAMIC`).
        *   `m_d3dPool`: Buffer'ın bulunduğu bellek havuzu (örn: `D3DPOOL_MANAGED`, `D3DPOOL_SYSTEMMEM`).
        *   `m_vtxCount`: Vertex sayısı.
        *   `m_dwLockFlag`: `Lock` işlemi sırasında kullanılacak bayrak (genellikle `0` veya `D3DLOCK_READONLY`).

*   **`GrpVertexBuffer.cpp` - Uygulama Detayları:**
    *   **`Create` Metodu:**
        *   Mevcut bir buffer varsa `Destroy()` ile onu siler.
        *   `m_vtxCount`, `m_dwFVF`, `m_dwUsage`, `m_d3dPool` üyelerini saklar.
        *   `D3DXGetFVFVertexSize(fvf)` ile bir vertex'in boyutunu hesaplar ve `m_dwBufferSize`'ı buna göre ayarlar.
        *   `m_dwLockFlag`'ı `usage` parametresine göre belirler: Eğer `D3DUSAGE_WRITEONLY` veya `D3DUSAGE_DYNAMIC` ise `0` (tam erişim), aksi takdirde `D3DLOCK_READONLY` (sadece okuma) olarak ayarlar.
        *   Son olarak `CreateDeviceObjects()`'u çağırarak asıl Direct3D buffer nesnesini oluşturur.
    *   **`CreateDeviceObjects` Metodu:** `ms_lpd3dDevice->CreateVertexBuffer` çağırarak Direct3D vertex buffer nesnesini oluşturur ve `m_lpd3dVB`'ye atar.
    *   **`DestroyDeviceObjects` Metodu:** `safe_release(m_lpd3dVB)` ile Direct3D kaynağını serbest bırakır.
    *   **`Lock` / `Unlock` Metotları:** `m_lpd3dVB->Lock()` ve `m_lpd3dVB->Unlock()` fonksiyonlarını çağırarak buffer belleğine erişim sağlar/keser. `Lock` metotları, `m_dwLockFlag`'ı veya belirli durumlarda `0`'ı kullanır.
    *   **`Copy` Metodu:** Buffer'ı `Lock` eder, `memcpy` ile veriyi kopyalar ve `Unlock` eder.
    *   **`SetStream` Metodu:** `STATEMANAGER.SetStreamSource` çağırarak bu buffer'ı Direct3D cihazına bir vertex stream kaynağı olarak tanıtır.
    *   **Bilgi Alma Metotları:** İlgili üye değişkenlerin değerlerini döndürür.
    *   **Kurucu/Yıkıcı/Initialize/Destroy:** Temel başlatma ve temizleme işlemlerini yönetir.

*   **Kullanım Alanı:**
    *   `CGraphicVertexBuffer`, 3D geometrilerin köşe verilerini GPU'ya aktarmak ve render sırasında kullanmak için temel bir arayüz sağlar.
    *   Doğrudan örneklenmek yerine, genellikle `CStaticVertexBuffer` (değişmeyen veriler için) veya `CDynamicVertexBuffer` (sık güncellenen veriler için) aracılığıyla kullanılır.
    *   Bu türetilmiş sınıflar, `Create` metodunu kendi özel `Usage` ve `Pool` parametreleriyle çağırarak uygun türde bir vertex buffer oluşturur.
    *   Buffer'a veri yazmak için `Lock` ve `Unlock` kullanılır. Veri yazıldıktan sonra, `SetStream` ile render için hazırlanır ve ardından Direct3D çizim komutları (`DrawPrimitive` vb.) kullanılır. 

### `GrpSubImage.h` ve `GrpSubImage.cpp` (`CGraphicSubImage` Sınıfı)

*   **Amaç:** Bu dosyalar, `CGraphicSubImage` sınıfını tanımlar ve uygular. `CGraphicSubImage`, bir ana `CGraphicImage` nesnesinin (genellikle daha büyük bir resim veya sprite sheet) belirli bir dikdörtgen alanını (alt resim) temsil eder ve bu alt alanı ayrı bir resim gibi kullanmayı sağlar. Temel olarak, bir "referans" veya "pencere" görevi görerek ana resmin bir bölümüne işaret eder.

*   **`GrpSubImage.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicSubImage` Sınıfı:**
        *   **Kalıtım:** `public CGraphicImage`. (Bu, `CGraphicSubImage`'ın da bir `CGraphicImage` gibi davranabileceği ve `CResourceManager` tarafından yönetilebileceği anlamına gelir.)
        *   **`TRef`:** `CRef<CGraphicImage>` için bir typedef. `m_roImage` üyesinin türünü belirtir.
        *   `static TType Type()`: Bu kaynak sınıfının türünü ("CGraphicSubImage") döndürür.
        *   `static char m_SearchPath[256]`: Alt resim tanım dosyalarının (.sui uzantılı olabilir) veya bu dosyalar içinde referans verilen ana resim dosyalarının aranacağı varsayılan bir arama yolu. `SetSearchPath` ile değiştirilebilir.
        *   Kurucu (`CGraphicSubImage(const char* c_szFileName)`), Yıkıcı (`virtual ~CGraphicSubImage()`).
        *   `bool CreateDeviceObjects()`: Ana resmin (`m_roImage`) doku işaretçisini kendi `m_imageTexture`'sine (temel sınıf `CGraphicImage`'dan miras alınan) atar. Esasen, alt resim kendi Direct3D doku nesnesini oluşturmaz, ana resmin dokusunu paylaşır.
        *   `bool SetImageFileName(const char* c_szFileName)`: Verilen dosya adıyla bir `CGraphicImage` kaynağını `CResourceManager` üzerinden yükler ve `SetImagePointer` ile bu alt resmin ana resmi olarak ayarlar.
        *   `void SetRectPosition(int left, int top, int right, int bottom)`: Alt resmin ana resim içindeki koordinatlarını (dikdörtgenini) ayarlar.
        *   `void SetRectReference(const RECT& c_rRect)`: `SetRectPosition`'a benzer şekilde `RECT` yapısı alarak koordinatları ayarlar.
        *   `static void SetSearchPath(const char* c_szFileName)`: `m_SearchPath` statik üyesini günceller.
    *   **Korumalı Metotlar:**
        *   `void SetImagePointer(CGraphicImage* pImage)`: Verilen `CGraphicImage` işaretçisini `m_roImage` olarak ayarlar ve `CreateDeviceObjects()`'u çağırarak doku bağlantısını kurar.
        *   `OnLoad(int iSize, const void* c_pvBuf)`: Alt resim tanım dosyasının (.sui veya benzeri bir metin dosyası) içeriğini ayrıştırır. Bu dosya, ana resmin dosya adını ve alt resmin koordinatlarını içerir.
        *   `OnClear()`: Ana resim referansını (`m_roImage`) `NULL` yapar ve `m_rect`'i sıfırlar.
        *   `OnIsEmpty() const`: Ana resmin geçerli ve yüklü olup olmadığını kontrol eder.
        *   `OnIsType(TType type)`: Kaynak tipini kontrol eder (hem `CGraphicSubImage` hem de `CGraphicImage` tiplerine yanıt verir).
    *   **Korumalı Üye:**
        *   `CGraphicImage::TRef m_roImage`: Alt resmin ait olduğu ana `CGraphicImage` nesnesine bir referans sayımlı işaretçi (akıllı işaretçi).

*   **`GrpSubImage.cpp` - Uygulama Detayları:**
    *   **`m_SearchPath`:** Varsayılan olarak "D:/Ymir Work/UI/" şeklinde bir değere sahiptir.
    *   **Kurucu/Yıkıcı:** Kurucu, temel sınıf `CGraphicImage`'ın kurucusunu çağırır. Yıkıcı, `m_roImage` referansını `NULL` yapar (akıllı işaretçi sayesinde referans sayısı azalır).
    *   **`CreateDeviceObjects`:** `m_roImage` geçerliyse, onun `GetTexturePointer()` metodundan aldığı `LPDIRECT3DTEXTURE8` işaretçisini kendi `m_imageTexture.CreateFromTexturePointer()` metoduna vererek doku paylaşımını gerçekleştirir. Alt resmin kendi `m_lpd3dTexture`'ı (temel sınıftan) ana resmin dokusunu gösterir.
    *   **`SetImageFileName`:** `CResourceManager::Instance().GetResourcePointer` ile ana resmi yükler/alır, tipini kontrol eder ve `SetImagePointer` ile atar.
    *   **`OnLoad` Metodu (Tanım Dosyası Ayrıştırma):**
        1.  Giriş olarak aldığı `c_pvBuf` (dosya içeriği) bir metin dosyası olarak kabul edilir.
        2.  `CMemoryTextFileLoader` kullanarak bu metin dosyasını satır satır işler.
        3.  Her satırın "anahtar değer" formatında olduğunu varsayar ve bir `std::map<std::string, std::string>` içine yükler (tüm anahtar ve değerler küçük harfe çevrilir).
        4.  Haritadan "title", "version", "image", "left", "top", "right", "bottom" anahtarlarını arar.
        5.  "title" değeri "subimage" olmalıdır.
        6.  "version" değeri "2.0" ise, ana resmin dosya yolu (`c_rstImage`) alt resim dosyasının bulunduğu dizine göre göreceli olarak kabul edilir. Aksi takdirde, `m_SearchPath` ile birleştirilerek mutlak bir yol oluşturulur.
        7.  Hesaplanan ana resim dosya adıyla `SetImageFileName` çağrılır.
        8.  "left", "top", "right", "bottom" değerleri `atoi` ile integer'a çevrilerek `SetRectPosition` ile alt resmin koordinatları ayarlanır.
    *   **`OnClear`:** `m_roImage`'ı `NULL` yapar ve `m_rect`'i sıfırlar. Bu, `CGraphicImage`'ın temel `OnClear`'ını çağırmaz, yani `m_imageTexture` doğrudan temizlenmez (çünkü o ana resme aittir).
    *   **`OnIsEmpty`:** `m_roImage` null değilse ve `m_roImage` boş değilse `false` (yani dolu) döner.
    *   **`OnIsType`:** Hem kendi tipini (`CGraphicSubImage::Type()`) hem de temel sınıfının tipini (`CGraphicImage::OnIsType(type)`) kontrol eder.

*   **Kullanım Alanı:**
    *   `CGraphicSubImage`, genellikle kullanıcı arayüzü (UI) elemanlarında veya 2D sprite animasyonlarında kullanılır.
    *   Büyük bir resim dosyası (texture atlas veya sprite sheet) içinde birden fazla küçük resim barındırıldığında, her bir küçük resmi ayrı bir `CGraphicSubImage` olarak tanımlayarak yönetmeyi kolaylaştırır.
    *   Örneğin, bir `.tga` dosyası birçok UI ikonunu içerebilir. Her ikon için bir `.sui` (veya benzeri bir tanım dosyası) oluşturulur. Bu `.sui` dosyası, ana `.tga` dosyasının adını ve ikonun o `.tga` içindeki koordinatlarını belirtir.
    *   Oyun, `.sui` dosyasını `CResourceManager` aracılığıyla bir `CGraphicSubImage` olarak yükler. Bu `CGraphicSubImage` nesnesi, render edilirken aslında ana `.tga`'nın dokusunu kullanır ancak sadece belirtilen `m_rect` alanı içindeki pikselleri çizer.
    *   Bu yöntem, doku sayısını azaltarak ve doku değiştirme (texture swapping) maliyetini düşürerek performansı artırabilir.

### `GrpShadowTexture.h` ve `GrpShadowTexture.cpp` (`CGraphicShadowTexture` Sınıfı)

*   **Amaç:** Bu dosyalar, `CGraphicShadowTexture` sınıfını tanımlar ve uygular. Bu sınıf, gölge haritalama (shadow mapping) tekniği için bir render hedefi dokusu (render target texture) oluşturmak ve yönetmek amacıyla kullanılır. Sahneyi ışığın bakış açısından bu dokuya render ederek bir derinlik haritası (gölge haritası) oluşturur. Bu gölge haritası daha sonra ana render geçişinde sahnedeki nesnelerin gölgeli mi yoksa aydınlık mı olduğunu belirlemek için kullanılır.

*   **`GrpShadowTexture.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicShadowTexture` Sınıfı:**
        *   **Kalıtım:** `public CGraphicTexture`. (Temel doku özelliklerini ve Direct3D cihazına erişimi miras alır.)
        *   Kurucu (`CGraphicShadowTexture()`), Yıkıcı (`virtual ~CGraphicShadowTexture()`).
        *   `Destroy()`: Gölge dokusuyla ilişkili tüm Direct3D kaynaklarını (doku, yüzeyler) serbest bırakır.
        *   `Create(int width, int height)`: Belirtilen genişlik ve yükseklikte bir gölge dokusu (`m_lpd3dShadowTexture`), bu dokuya ait bir render hedefi yüzeyi (`m_lpd3dShadowSurface`) ve bir derinlik/stencil yüzeyi (`m_lpd3dDepthSurface`) oluşturur.
        *   `Begin()`: Gölge haritası render geçişini başlatır. Eski render hedefini, derinlik tamponunu ve viewport'u kaydeder. Ardından gölge dokusunun yüzeyini ve derinlik yüzeyini aktif render hedefi olarak ayarlar. Viewport'u gölge dokusunun boyutlarına göre ayarlar, sahneyi başlatır (`BeginScene`), hedef dokuyu ve derinlik tamponunu temizler. Ayrıca gölge render'ı için uygun render durumlarını (`CULLMODE`, `ZFUNC`, `ALPHABLEND`, `TEXTUREFACTOR` vb.) ayarlar.
        *   `End()`: Gölge haritası render geçişini sonlandırır. Sahneyi bitirir (`EndScene`), `Begin()` içinde kaydedilmiş olan orijinal render hedefini, derinlik tamponunu ve viewport'u geri yükler. Kaydedilmiş render durumlarını da geri yükler.
        *   `Set(int stage = 0) const`: Oluşturulan gölge dokusunu (`m_lpd3dShadowTexture`) belirtilen doku aşamasına (`stage`) ayarlar. Bu, ana render geçişinde gölge haritasını okumak için kullanılır.
        *   `GetLightVPMatrixReference() const`: Işığın view-projection matrisine (`m_d3dLightVPMatrix`) bir referans döndürür. Bu matris, `Begin()` içinde mevcut kamera view ve projection matrisleri çarpılarak hesaplanır ve gölge haritası render'ı sırasında kullanılır.
        *   `GetD3DTexture() const`: Ham Direct3D gölge dokusu nesnesine (`m_lpd3dShadowTexture`) bir işaretçi döndürür.
    *   **Korumalı Metot:**
        *   `Initialize()`: Tüm üye işaretçilerini `NULL` olarak ve diğer üyeleri varsayılan değerlerine ayarlar.
    *   **Korumalı Üyeler:**
        *   `m_d3dLightVPMatrix`: Işığın view-projection matrisi.
        *   `m_d3dOldViewport`: `Begin()` çağrılmadan önceki orijinal viewport.
        *   `m_lpd3dShadowTexture`: Asıl gölge haritasını tutan Direct3D dokusu.
        *   `m_lpd3dShadowSurface`: `m_lpd3dShadowTexture`'a ait render hedefi yüzeyi.
        *   `m_lpd3dDepthSurface`: Gölge haritası render'ı için kullanılan derinlik/stencil yüzeyi.
        *   `m_lpd3dOldBackBufferSurface`: `Begin()` çağrılmadan önceki orijinal renk tamponu yüzeyi.
        *   `m_lpd3dOldDepthBufferSurface`: `Begin()` çağrılmadan önceki orijinal derinlik/stencil yüzeyi.

*   **`GrpShadowTexture.cpp` - Uygulama Detayları:**
    *   **`Create` Metodu:**
        *   Mevcut kaynakları `Destroy()` ile temizler.
        *   `ms_lpd3dDevice->CreateTexture` ile `D3DUSAGE_RENDERTARGET` ve `D3DFMT_A8R8G8B8` formatında bir doku oluşturur (`m_lpd3dShadowTexture`).
        *   Bu dokunun 0. seviye yüzeyini `GetSurfaceLevel` ile alır (`m_lpd3dShadowSurface`).
        *   `ms_lpd3dDevice->CreateDepthStencilSurface` ile `D3DFMT_D16` formatında bir derinlik yüzeyi oluşturur (`m_lpd3dDepthSurface`).
    *   **`Begin` Metodu:**
        *   `m_d3dLightVPMatrix`'i, o anki global `ms_matView` ve `ms_matProj` (genellikle ana kameranın matrisleri) çarpımıyla hesaplar. Bu, gölge haritasının ışığın bakış açısından render edileceği anlamına gelir (ışık = kamera gibi davranır).
        *   Mevcut render hedefini, derinlik tamponunu ve viewport'u saklar.
        *   `ms_lpd3dDevice->SetRenderTarget` ile `m_lpd3dShadowSurface` ve `m_lpd3dDepthSurface`'i aktif hale getirir.
        *   Viewport'u gölge dokusunun boyutlarına ayarlar.
        *   `ms_lpd3dDevice->Clear` ile hedef dokuyu siyaha (0x00000000), derinlik tamponunu 1.0f'a temizler.
        *   `STATEMANAGER` aracılığıyla çeşitli render durumlarını (culling yok, Z-test açık, alpha blend açık, doku filtreleme point vb.) gölge render'ı için özel olarak ayarlar ve eski durumları kaydeder.
    *   **`End` Metodu:**
        *   `ms_lpd3dDevice->EndScene()` çağırır.
        *   `SetRenderTarget` ve `SetViewport` ile orijinal durumları geri yükler.
        *   Saklanan yüzeyleri (`m_lpd3dOldBackBufferSurface`, `m_lpd3dOldDepthBufferSurface`) serbest bırakır.
        *   `STATEMANAGER` aracılığıyla `Begin`'de değiştirilen tüm render durumlarını orijinal hallerine geri yükler.
    *   **`Set` Metodu:** `STATEMANAGER.SetTexture` çağırarak `m_lpd3dShadowTexture`'ı belirtilen doku aşamasına bağlar.
    *   **Kurucu/Yıkıcı/Initialize/Destroy:** Kaynakların doğru bir şekilde başlatılmasını ve serbest bırakılmasını sağlar.

*   **Kullanım Alanı:**
    *   `CGraphicShadowTexture`, dinamik gölgeler oluşturmak için yaygın bir teknik olan gölge haritalama (shadow mapping) implementasyonunun bir parçasıdır.
    *   Render döngüsünde genellikle şu adımlar izlenir:
        1.  Bir `CGraphicShadowTexture` nesnesi oluşturulur (`Create`).
        2.  Ana render döngüsünün başında, gölge üretecek nesneler için:
            a.  `CGraphicShadowTexture::Begin()` çağrılır. Bu, render hedefini gölge dokusuna yönlendirir ve ışığın bakış açısından bir projeksiyon ayarlar.
            b.  Sahnedeki gölge üretecek nesneler (genellikle sadece derinlik bilgileri veya basitleştirilmiş bir formda) bu gölge dokusuna render edilir. Bu işlem sırasında, nesnelerin ışığa olan mesafeleri (derinlikleri) dokuya yazılır.
            c.  `CGraphicShadowTexture::End()` çağrılarak ana render hedefine geri dönülür.
        3.  Ana sahne render geçişinde:
            a.  `CGraphicShadowTexture::Set()` çağrılarak oluşturulan gölge haritası bir doku olarak aktif edilir.
            b.  Sahnedeki her bir piksel render edilirken, o pikselin dünya koordinatları ışığın view-projection matrisi (`GetLightVPMatrixReference()`) kullanılarak ışığın bakış açısındaki koordinatlara dönüştürülür.
            c.  Bu dönüştürülmüş koordinatlar, gölge haritası dokusundan bir değer okumak için kullanılır. Okunan değer (ışığa olan kayıtlı mesafe) ile pikselin ışığa olan gerçek mesafesi karşılaştırılır.
            d.  Eğer pikselin ışığa olan gerçek mesafesi, gölge haritasındaki değerden büyükse, piksel gölgede kalmış demektir ve daha koyu render edilir. Aksi takdirde aydınlıktır.
    *   Bu sınıf, gölge oluşturma işleminin "ışığın bakış açısından render etme" kısmını yönetir.

### `GrpRenderTargetTexture.h` ve `GrpRenderTargetTexture.cpp` (`CGraphicRenderTargetTexture` Sınıfı)

*   **Koşullu Derleme:** Bu sınıfla ilgili kodlar, yalnızca `RENDER_TARGET` makrosu tanımlı olduğunda derlenir.
*   **Amaç:** `CGraphicRenderTargetTexture` sınıfı, Direct3D'de bir dokuya render yapma (render-to-texture) yeteneği sağlar. Bu, 3D sahnenin veya nesnelerin normal ekran yerine özel bir dokuya çizilmesini mümkün kılar. Bu doku daha sonra başka bir nesne üzerinde kaplama olarak veya bir UI elemanı olarak kullanılabilir.

*   **`GrpRenderTargetTexture.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicRenderTargetTexture` Sınıfı:**
        *   **Kalıtım:** `public CGraphicTexture`. (Temel doku özelliklerini ve `CGraphicBase` üzerinden Direct3D cihazına erişimi miras alır.)
        *   Kurucu (`CGraphicRenderTargetTexture()`), Yıkıcı (`virtual ~CGraphicRenderTargetTexture()`).
        *   `Create(int width, int height, D3DFORMAT texFormat, D3DFORMAT depthFormat)`: Belirtilen boyutlarda ve formatlarda bir render hedefi dokusu ve bir derinlik/stencil yüzeyi oluşturur.
        *   `CreateTextures()`: Kayıtlı boyut ve formatlarla doku ve derinlik yüzeyini (yeniden) oluşturur. Genellikle cihaz kaybolduktan sonra kaynakları geri yüklemek için kullanılır.
        *   `CreateRenderTexture(int width, int height, D3DFORMAT format)`: Belirtilen boyut ve formatta asıl render hedefi dokusunu (`m_lpd3dRenderTexture`) ve buna bağlı yüzeyi (`m_lpd3dRenderTargetSurface`) oluşturur.
        *   `CreateRenderDepthStencil(int width, int height, D3DFORMAT format)`: Render hedefi için bir derinlik/stencil yüzeyi (`m_lpd3dDepthSurface`) oluşturur.
        *   `SetRenderTarget()`: Bu nesnenin render hedefi yüzeyini (`m_lpd3dRenderTargetSurface`) ve derinlik yüzeyini (`m_lpd3dDepthSurface`) Direct3D cihazının aktif render hedefi olarak ayarlar. Orijinal render hedefi ve derinlik yüzeyini saklar.
        *   `ResetRenderTarget()`: `SetRenderTarget` ile saklanmış olan orijinal render hedefini ve derinlik yüzeyini geri yükler.
        *   `SetRenderingRect(RECT* rect)`: Render hedefi dokusunun ekranda çizileceği hedef dikdörtgeni ayarlar.
        *   `GetRenderingRect()`: Ayarlanmış hedef çizim dikdörtgenini döndürür.
        *   `LPDIRECT3DTEXTURE8 GetRenderTargetTexture() const`: Oluşturulan Direct3D render hedefi dokusuna (`m_lpd3dRenderTexture`) bir işaretçi döndürür.
        *   `ReleaseTextures()`: Oluşturulmuş tüm Direct3D doku ve yüzey kaynaklarını serbest bırakır.
        *   `static void Clear()`: Aktif render hedefini (bu sınıfın veya ana ekranın) ve derinlik tamponunu varsayılan bir renkle (siyah, alfa=0) temizler.
        *   `Render(RECT* pClipRect = NULL) const` (`ENABLE_CLIP_MASK` tanımlıysa) veya `Render() const`: Bu render hedefi dokusunu, `m_renderRect` ile belirlenen hedef dikdörtgene 2D olarak çizer. Eğer `ENABLE_CLIP_MASK` tanımlıysa, `pClipRect` ile kırpma yapabilir.
    *   **Korumalı Metot:**
        *   `Reset()`: `Destroy()` (temel sınıftan) ve `ReleaseTextures()` çağırarak tüm kaynakları temizler ve format bilgilerini sıfırlar.
    *   **Korumalı Üyeler:**
        *   `m_lpd3dRenderTexture`: Asıl render işleminin yapıldığı Direct3D dokusu.
        *   `m_lpd3dRenderTargetSurface`: `m_lpd3dRenderTexture`'a ait render hedefi yüzeyi.
        *   `m_lpd3dDepthSurface`: Bu render hedefi için kullanılan derinlik/stencil yüzeyi.
        *   `m_lpd3dOriginalRenderTarget`: `SetRenderTarget` çağrılmadan önceki orijinal renk tamponu yüzeyi.
        *   `m_lpd3dOldDepthBufferSurface`: `SetRenderTarget` çağrılmadan önceki orijinal derinlik/stencil yüzeyi.
        *   `m_d3dFormat`: Render hedefi dokusunun piksel formatı (örn: `D3DFMT_A8R8G8B8`).
        *   `m_depthStencilFormat`: Derinlik/stencil yüzeyinin formatı (örn: `D3DFMT_D16`).
        *   `m_renderRect`: Render hedefi dokusunun ekrana çizileceği hedef dikdörtgen.

*   **`GrpRenderTargetTexture.cpp` - Uygulama Detayları:**
    *   **Başlatma ve Kaynak Yönetimi:**
        *   Kurucu, `Initialize()` (temel sınıftan) çağırır ve `m_renderRect`'i sıfırlar.
        *   `Create`, `Reset` ile önceki kaynakları temizler, boyut ve formatları saklar, ardından `CreateRenderTexture` ve `CreateRenderDepthStencil`'ı çağırır.
        *   `CreateRenderTexture`, `D3DUSAGE_RENDERTARGET` ve `D3DPOOL_DEFAULT` bayraklarıyla `ms_lpd3dDevice->CreateTexture` kullanarak dokuyu oluşturur ve `GetSurfaceLevel` ile yüzeyini alır.
        *   `CreateRenderDepthStencil`, `ms_lpd3dDevice->CreateDepthStencilSurface` ile derinlik yüzeyini oluşturur.
        *   `ReleaseTextures` ve `Reset` metotları `safe_release` kullanarak tüm Direct3D kaynaklarını düzgün bir şekilde serbest bırakır.
    *   **Render Hedefi Ayarlama:**
        *   `SetRenderTarget`, `ms_lpd3dDevice->GetRenderTarget` ve `GetDepthStencilSurface` ile mevcut hedefleri saklar, ardından `ms_lpd3dDevice->SetRenderTarget` ile bu nesnenin yüzeylerini aktif hale getirir.
        *   `ResetRenderTarget`, saklanmış orijinal hedefleri `SetRenderTarget` ile geri yükler ve saklanan yüzey işaretçilerini serbest bırakır.
    *   **Temizleme (`Clear` - statik metot):** `ms_lpd3dDevice->Clear` fonksiyonunu çağırarak aktif render hedefini (bu bir `CGraphicRenderTargetTexture` olabilir veya olmayabilir) ve Z-buffer'ı siyaha temizler.
    *   **Render (`Render` metodu):**
        1.  Render edilecek doku yüzeyinin (`m_lpd3dRenderTargetSurface`) özelliklerini (`GetDesc`) alır.
        2.  `m_renderRect` ile belirtilen hedef çizim alanına göre köşe (`TPDTVertex`) pozisyonlarını ve doku UV koordinatlarını hesaplar. UV koordinatları, dokunun tamamını (`0.0` ile `1.0` arası) hedef dikdörtgene eşler.
        3.  `ENABLE_CLIP_MASK` tanımlıysa ve `pClipRect` verilmişse, çizilecek dörtgeni ve UV koordinatlarını bu kırpma dikdörtgenine göre ayarlar.
        4.  Hesaplanan köşe noktalarını `SetPDTStream` ile bir vertex buffer'a yükler.
        5.  `SetDefaultIndexBuffer(DEFAULT_IB_FILL_RECT)` ile standart bir dörtgen çizim indeksi ayarlar.
        6.  `STATEMANAGER` aracılığıyla render hedefi dokusunu (`GetRenderTargetTexture()`) 0. doku aşamasına ayarlar, 1. doku aşamasını temizler ve varsayılan bir vertex shader (`D3DFVF_XYZ | D3DFVF_TEX1 | D3DFVF_DIFFUSE`) ayarlar.
        7.  `STATEMANAGER.DrawIndexedPrimitive` ile dörtgeni çizer.

*   **Kullanım Alanı:**
    *   Oyun içi 3D nesnelerin veya sahnelerin bir kullanıcı arayüzü (UI) elemanı üzerinde (örneğin, karakter önizleme penceresi, eşya bilgi penceresindeki 3D model) gösterilmesi için kullanılır. Sahne önce bu `CGraphicRenderTargetTexture`'a render edilir, sonra bu doku UI üzerine 2D bir resim olarak çizilir (`Render` metodu ile).
    *   Ayna yüzeyleri, güvenlik kameraları gibi efektler oluşturmak için kullanılabilir (sahnenin farklı bir açıdan bir dokuya render edilmesi).
    *   Post-processing efektleri uygulamadan önce sahnenin tamamını bir dokuya render etmek için bir ara adım olarak kullanılabilir.
    *   `CRenderTarget` sınıfı, genellikle bu `CGraphicRenderTargetTexture` sınıfını kullanarak kendi render-to-texture işlevselliğini oluşturur.

### `GrpMarkInstance.h` ve `GrpMarkInstance.cpp` (`CGraphicMarkInstance` Sınıfı)

*   **Amaç:** `CGraphicMarkInstance` sınıfı, bir ana resim dosyasından (genellikle bir sprite sheet veya atlas) belirli bir indekse karşılık gelen küçük bir "işaret" (mark) resmini (sabit boyutlarda: `MARK_WIDTH`, `MARK_HEIGHT`) ekranda 2D olarak çizmek için kullanılır. Bu işaretler genellikle lonca sembolleri, ikonlar veya benzeri küçük UI elemanları için kullanılır. Sınıf, nesne havuzlaması (`CDynamicPool`) ile yönetilir.

*   **`GrpMarkInstance.h` - Tanımlar ve Bildirimler:**
    *   **`CGraphicMarkInstance` Sınıfı:**
        *   `static DWORD Type()`: Sınıf için bir tip tanımlayıcısı döndürür (CRC32 tabanlı).
        *   `BOOL IsType(DWORD dwType)`: Verilen tipin bu sınıfın tipiyle eşleşip eşleşmediğini kontrol eder.
        *   `SetImageFileName(const char* c_szFileName)`, `GetImageFileName()`: İşaretlerin alınacağı ana resim dosyasının adını ayarlar/alır.
        *   Kurucu (`CGraphicMarkInstance()`), Yıkıcı (`virtual ~CGraphicMarkInstance()`).
        *   `Destroy()`: Kaynakları temizler ve örneği başlangıç durumuna sıfırlar.
        *   `Render(RECT* pClipRect = NULL)` (`ENABLE_CLIP_MASK` tanımlıysa) veya `Render()`: İşareti ekrana çizer. `OnRender` sanal metodunu çağırır.
        *   **Özellik Ayarlama Metotları:**
            *   `SetDepth(float fDepth)`: Çizim derinliğini ayarlar (ancak mevcut implementasyonda Z pozisyonu hep 0.0f olarak kullanılır).
            *   `SetDiffuseColor(float fr, float fg, float fb, float fa)`: İşaretin çizileceği rengi (ve alfayı) ayarlar.
            *   `SetPosition(float fx, float fy)`: İşaretin ekran üzerindeki sol üst köşe pozisyonunu ayarlar.
            *   `SetIndex(UINT uIndex)`: Ana resim içindeki hangi işaretin (indeks) çizileceğini belirler.
            *   `SetScale(float fScale)`: İşaretin çizim ölçeğini ayarlar.
        *   `Load()`: `m_stImageFileName` ile belirtilen ana resim dosyasını `CResourceManager` kullanarak yükler ve `SetImagePointer` ile ayarlar.
        *   `IsEmpty() const`: Ana resmin yüklenip yüklenmediğini kontrol eder.
        *   `GetWidth()`, `GetHeight()`: İşaretin sabit boyutlarını (`MARK_WIDTH`, `MARK_HEIGHT`) döndürür (ölçekleme hariç).
        *   **Ana Resim ve Doku Erişimi:**
            *   `GetTexturePointer()`: Ana resmin `CGraphicTexture` işaretçisini döndürür.
            *   `GetTextureReference()`: Ana resmin `CGraphicTexture` referansını döndürür.
            *   `GetGraphicImagePointer()`: Ana `CGraphicImage` işaretçisini döndürür.
        *   `bool operator == (const CGraphicMarkInstance& rhs) const`: İki örneğin aynı ana resmi kullanıp kullanmadığını karşılaştırır.
    *   **Enum:** `MARK_WIDTH = 16`, `MARK_HEIGHT = 12` sabitlerini tanımlar.
    *   **Korumalı Sanal Metotlar:**
        *   `OnRender(RECT* pClipRect)` (`ENABLE_CLIP_MASK` ile) veya `OnRender()`: Asıl çizim mantığını içerir.
        *   `OnSetImagePointer()`: Ana resim işaretçisi ayarlandıktan sonra çağrılır (boş implementasyon).
        *   `OnIsType(DWORD dwType)`: Tip kontrolü için sanal metot.
    *   **Korumalı Metot:**
        *   `Initialize()`: Üyeleri varsayılan değerlere sıfırlar.
        *   `SetImagePointer(CGraphicImage* pImage)`: Verilen `CGraphicImage`'ı ana resim olarak ayarlar.
    *   **Korumalı Üyeler:**
        *   `m_DiffuseColor`: Çizim rengi (`D3DXCOLOR`).
        *   `m_v2Position`: Ekran pozisyonu (`D3DXVECTOR2`).
        *   `m_uIndex`: Ana resimdeki işaret indeksi.
        *   `m_fScale`: Çizim ölçeği.
        *   `m_fDepth`: Çizim derinliği (kullanılmıyor gibi).
        *   `m_roImage`: Ana `CGraphicImage` nesnesine referans sayımlı işaretçi.
        *   `m_stImageFileName`: Ana resim dosyasının adı.
    *   **Statik Havuz Yönetimi:**
        *   `CreateSystem(UINT uCapacity)`, `DestroySystem()`: Nesne havuzunu (`ms_kPool`) oluşturur/yok eder.
        *   `New()`, `Delete(CGraphicMarkInstance* pkImgInst)`: Havuzdan nesne alır/verir.
        *   `ms_kPool`: `CDynamicPool<CGraphicMarkInstance>` tipinde statik nesne havuzu.

*   **`GrpMarkInstance.cpp` - Uygulama Detayları:**
    *   **Havuz Yönetimi:** `CreateSystem`, `DestroySystem`, `New`, `Delete` statik metotları, `ms_kPool` aracılığıyla `CGraphicMarkInstance` nesnelerinin verimli bir şekilde oluşturulup silinmesini yönetir.
    *   **`OnRender` Metodu:**
        1.  Eğer örnek boşsa (`IsEmpty()`) veya ana resim (`m_roImage`) geçerli değilse erken çıkar.
        2.  Ana resmin genişliğine (`pImage->GetWidth()`) ve `MARK_WIDTH` sabitine göre sütun sayısını (`uColCount`) hesaplar.
        3.  `m_uIndex` (işaret indeksi) kullanarak bu işaretin ana resimdeki satır (`uRow`) ve sütununu (`uCol`) belirler.
        4.  Bu satır/sütun ve `MARK_WIDTH`/`MARK_HEIGHT` sabitlerini kullanarak işaretin ana resim dokusu üzerindeki kaynak dikdörtgenini (`kRect`) ve buna karşılık gelen UV koordinatlarını (`su`, `sv`, `eu`, `ev`) hesaplar.
        5.  `m_fScale` kullanarak işaretin ekranda çizileceği nihai genişlik (`fRenderWidth`) ve yüksekliği (`fRenderHeight`) hesaplar.
        6.  `m_v2Position` ve hesaplanan `fRenderWidth`/`fRenderHeight`'a göre çizilecek dörtgenin köşe pozisyonlarını (`sx`, `sy`, `ex`, `ey`) belirler. (`-0.5f` düzeltmesi, piksel merkezleriyle hizalama için yaygın bir tekniktir.)
        7.  Eğer `ENABLE_CLIP_MASK` tanımlıysa ve `pClipRect` verilmişse, çizilecek dörtgeni ve UV koordinatlarını bu kırpma dikdörtgenine göre ayarlar (kısmi çizimler için).
        8.  Hesaplanan pozisyonlar, UV koordinatları ve `m_DiffuseColor` ile 4 adet `TPDTVertex` oluşturur.
        9.  Bu vertexleri `CGraphicBase::SetPDTStream` ile bir vertex buffer'a yükler.
        10. `CGraphicBase::SetDefaultIndexBuffer(CGraphicBase::DEFAULT_IB_FILL_RECT)` ile standart bir dörtgen çizim indeksi ayarlar.
        11. `STATEMANAGER` aracılığıyla ana resmin dokusunu 0. doku aşamasına ayarlar, 1. doku aşamasını temizler ve varsayılan bir vertex shader (`D3DFVF_XYZ | D3DFVF_DIFFUSE | D3DFVF_TEX1`) ayarlar.
        12. `STATEMANAGER.DrawIndexedPrimitive` ile dörtgeni (işareti) çizer.
    *   **`Load` Metodu:** `m_stImageFileName` kullanarak `CResourceManager`'dan ana resmi alır ve `SetImagePointer` ile atar. Resim bulunamazsa veya tipi yanlışsa hata verir.
    *   **`GetWidth` / `GetHeight`:** Ölçeklemeden bağımsız olarak sabit `MARK_WIDTH` (16) ve `MARK_HEIGHT` (12) değerlerini döndürür. Bu, ana resmin gerçek boyutlarından bağımsızdır.
    *   **Diğer Metotlar:** Genellikle ilgili üye değişkenleri ayarlar veya durum kontrolü yapar.

*   **Kullanım Alanı:**
    *   Genellikle lonca sembolleri (guild marks) gibi küçük, standart boyutlu ikonları veya işaretleri göstermek için kullanılır.
    *   Bir ana resim dosyası (sprite sheet), birden fazla lonca işaretini veya ikonu ızgara düzeninde içerebilir.
    *   `CGraphicMarkInstance` oluşturulurken, `SetImageFileName` ile bu ana resim dosyası belirtilir.
    *   `SetIndex` ile hangi belirli işaretin (sprite sheet içindeki konuma göre) çizileceği seçilir.
    *   `SetPosition`, `SetScale`, `SetDiffuseColor` ile işaretin ekrandaki konumu, boyutu ve rengi ayarlanır.
    *   `Render` metodu çağrılarak işaret ekrana çizilir.
    *   Nesne havuzlaması, çok sayıda işaretin (örneğin, birçok oyuncunun lonca işareti) sık sık oluşturulup silindiği senaryolarda performansı artırmaya yardımcı olur.