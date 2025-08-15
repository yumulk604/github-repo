# PRTerrainLib Referans Kılavuzu

Bu belge, Metin2 istemcisinin `PRTerrainLib` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. 

## Dosya Bazlı Detaylandırma

### `StdAfx.h` ve `StdAfx.cpp` (Ön Derlenmiş Başlıklar)

*   **Amaç:** Bu dosyalar, `PRTerrainLib` kütüphanesi için ön derlenmiş başlık (precompiled header - PCH) mekanizmasını oluşturur. Temel amaçları, sık kullanılan ve nadiren değişen başlık dosyalarını tek bir yerde toplayarak kütüphanenin genel derleme süresini kısaltmaktır.

*   **`StdAfx.h` İçeriği:**
    *   Standart başlık koruması (`#ifndef AFX_STDAFX_H__...`).
    *   `WIN32_LEAN_AND_MEAN` makrosu ile Windows başlıklarından daha az kullanılan kısımların çıkarılması.
    *   `#pragma warning(disable:4702)`: Ulaşılamayan kod (unreachable code) için derleyici uyarısını devre dışı bırakır.
    *   **Dahil Edilen Başlıklar:**
        *   `../EterLib/StdAfx.h`: Temel `EterLib` kütüphanesinin ön derlenmiş başlığını dahil eder.
        *   `../EterGrnLib/StdAfx.h`: Grafik nesneleriyle ilgili `EterGrnLib` kütüphanesinin ön derlenmiş başlığını dahil eder.
        *   `../ScriptLib/StdAfx.h`: Betikleme ile ilgili `ScriptLib` kütüphanesinin ön derlenmiş başlığını dahil eder.
        *   `../UserInterface/Locale_inc.h`: Kullanıcı arayüzü ve yerelleştirme ile ilgili tanımlamaları içerir.
    *   **Hızlı Float-Integer Dönüşümü İçin Değişkenler:**
        *   `extern float PR_FCNV;`
        *   `extern long PR_ICNV;`
        *   Bu değişkenler, muhtemelen FPU (Floating Point Unit) komutlarını kullanarak float ve integer türleri arasında hızlı dönüşüm yapmak için kullanılan bir teknikle ilgilidir. Genellikle performansa duyarlı grafik ve fizik hesaplamalarında kullanılırlar.

*   **`StdAfx.cpp` İçeriği:**
    *   `#include "stdafx.h"`: Bu satır, Visual C++ derleyicisinin `Stdafx.h` içinde listelenen tüm başlıkları kullanarak ön derlenmiş başlık dosyasını (`.pch`) oluşturmasını tetikler.
    *   **Hızlı Float-Integer Dönüşümü İçin Değişken Tanımları:**
        *   `float	PR_FCNV;`
        *   `long	PR_ICNV;`
        *   `StdAfx.h`'de `extern` olarak bildirilen global değişkenlerin tanımlandığı yerdir.

*   **Kullanım Amacı:**
    *   `PRTerrainLib` kütüphanesi içindeki diğer tüm `.cpp` dosyaları, derleme sürecini hızlandırmak için ilk satırlarında `#include "StdAfx.h"` ifadesini içermelidir.
    *   Temel sistem başlıklarına, diğer bağımlı kütüphanelerin başlıklarına ve sık kullanılan tanımlamalara merkezi bir erişim noktası sağlar.
    *   Hızlı float-integer dönüşümü için gerekli global değişkenleri tanımlar.

*   **Bağımlılıklar:**
    *   Windows Platform SDK.
    *   `EterLib`
    *   `EterGrnLib`
    *   `ScriptLib`
    *   `UserInterface` (özellikle `Locale_inc.h`) 

### `Terrain.h` ve `Terrain.cpp` (`CTerrainImpl` Sınıfı)

*   **Amaç:** `CTerrainImpl` sınıfı, oyun dünyasının temel arazi yapısını yönetmek için çekirdek işlevselliği sağlar. Bu sınıf, yükseklik haritaları (height maps), öznitelik haritaları (attribute maps - yürüme, su, blokaj gibi), su seviyeleri, gölge haritaları (shadow maps) ve doku kaplama (texture splatting) için alfa haritaları gibi çeşitli arazi verilerini yüklemekten, saklamaktan ve bu verilere erişim sağlamaktan sorumludur.

*   **`Terrain.h` - Temel Tanımlar ve Bildirimler:**
    *   **Enum Sabitleri:**
        *   `XSIZE`, `YSIZE`: Arazinin ana boyutları (örneğin, 128x128 hücre).
        *   `HEIGHTMAP_XSIZE`, `HEIGHTMAP_YSIZE`: Yükseklik haritasının boyutları (genellikle `XSIZE+1`).
        *   `HEIGHTMAP_RAW_XSIZE`, `HEIGHTMAP_RAW_YSIZE`: Ham yükseklik verisi için boyutlar (genellikle sınırlarda ek pikseller içerir).
        *   `ATTRMAP_XSIZE`, `ATTRMAP_YSIZE`: Öznitelik haritasının boyutları (genellikle `XSIZE*2`).
        *   `TILEMAP_XSIZE`, `TILEMAP_YSIZE`: Doku karıştırma (splatting) için kullanılan tile haritasının boyutları.
        *   `WATERMAP_XSIZE`, `WATERMAP_YSIZE`: Su haritasının boyutları.
        *   `SHADOWMAP_XSIZE`, `SHADOWMAP_YSIZE`: Gölge haritasının boyutları.
        *   `PATCH_XSIZE`, `PATCH_YSIZE`, `PATCH_XCOUNT`, `PATCH_YCOUNT`: Araziyi daha küçük parçalara (patch) bölmek için kullanılan boyutlar ve sayılar.
        *   `CELLSCALE`: Bir arazi hücresinin oyun içi birim cinsinden ölçeği (örn: 200 birim = 2 metre).
        *   `MAX_ATTRIBUTE_NUM`: Maksimum öznitelik sayısı.
        *   `MAX_WATER_NUM`: Maksimum su katmanı sayısı.
        *   **Öznitelik Bayrakları:** `ATTRIBUTE_BLOCK`, `ATTRIBUTE_WATER`, `ATTRIBUTE_BANPK` (muhtemelen "yasaklı PK" bölgesi).
    *   **Statik Üyeler (Doku Setleri):**
        *   `ms_pTextureSet` (CTextureSet*): Ana arazi doku setine işaretçi.
        *   `SetTextureSet(CTextureSet*)`, `GetTextureSet()`: Ana doku setini ayarlamak ve almak için statik metotlar.
        *   `ENABLE_ENVIRONMENT_EFFECT_OPTION` ile koşullu derlenen `ms_pSnowTextureSet`, `SetSnowTextureSet`, `GetSnowTextureSet`: Kar efekti için ayrı bir doku seti yönetimi.
    *   **Temel Üye Fonksiyonlar:**
        *   Yapıcı (`CTerrainImpl()`) ve Yıkıcı (`~CTerrainImpl()`).
        *   `GetTerrainSplatPatch()`: Doku kaplama bilgilerini içeren `TTerrainSplatPatch` yapısını döndürür.
        *   `GetNumTextures()`, `GetTexture()`: Ana doku setindeki doku sayısını ve belirli bir dokuyu alır.
        *   `LoadWaterMap()`: Su haritası dosyasını yükler.
        *   `GetShadowTexture()`: Gölge haritası için Direct3D dokusunu döndürür.
        *   `GetShadowMapColor()`: Belirli bir dünya koordinatındaki gölge haritası rengini alır.
        *   `GetHeightMapValue()`: Belirli bir koordinattaki ham yükseklik değerini alır (inline).
    *   **Korunan (Protected) Üye Değişkenler (Arazi Verileri):**
        *   `m_lpAlphaTexture[MAXTERRAINTEXTURES]`: Doku kaplama için alfa dokularına işaretçiler.
        *   `m_awRawHeightMap[]`: Ham yükseklik verilerini tutan dizi (WORD).
        *   `m_abyTileMap[]`: Tile haritası verilerini tutan dizi (BYTE - doku indeksleri).
        *   `m_abyAttrMap[]`: Öznitelik haritası verilerini tutan dizi (BYTE - bayraklar).
        *   `m_abyWaterMap[]`: Su varlığını belirten harita verileri (BYTE).
        *   `m_acNormalMap[]`: Normal haritası verileri (CHAR - x,y,z bileşenleri).
        *   `m_HeightMapHeader` (TGA_HEADER): Yükseklik haritası TGA dosyası için başlık bilgisi (kullanımı tam net değil, ham veri yükleniyor gibi).
        *   `m_wTileMapVersion`: Tile haritasının sürüm numarası.
        *   `m_lViewRadius`: Görüş yarıçapı.
        *   `m_fHeightScale`: Yükseklik verileri için ölçek faktörü.
        *   `m_TerrainSplatPatch` (TTerrainSplatPatch): Doku kaplama yaması bilgileri.
        *   `m_byNumWater`: Aktif su katmanı sayısı.
        *   `m_lWaterHeight[]`: Her bir su katmanının yükseklik seviyeleri.
        *   `m_lpShadowTexture`: Gölge haritası için Direct3D dokusu.
        *   `m_awShadowMap[]`: Ham gölge haritası verileri (WORD - R5G5B5 formatında renk).
        *   `m_lSplatTilesX`, `m_lSplatTilesY`: Kaplama tile'larının sayısı.
    *   **Korunan (Protected) Üye Fonksiyonlar (Yükleme ve Başlatma):**
        *   `Initialize()`: Sınıf üyelerini başlangıç değerlerine ayarlar.
        *   `Clear()`: Ayrılan kaynakları (özellikle alfa dokularını) serbest bırakır ve `Initialize` çağırır.
        *   `LoadTextures()`: (Bu dosyada implementasyonu yok, muhtemelen türetilmiş sınıflarda).
        *   `LoadHeightMap()`: Yükseklik haritası dosyasını yükler.
        *   `RAW_LoadTileMap()`: Ham tile haritası dosyasını yükler.
        *   `LoadAttrMap()`: Öznitelik haritası dosyasını yükler.

*   **`Terrain.cpp` - Uygulama Detayları:**
    *   **Doku Seti Yönetimi:**
        *   `SetTextureSet()` ve `GetTextureSet()` (ve kar versiyonları `SetSnowTextureSet`, `GetSnowTextureSet`), `CTerrainImpl` sınıfının global olarak kullanacağı doku setlerini yönetir. Eğer `NULL` bir doku seti atanırsa, statik boş bir doku seti (`s_EmptyTextureSet`) kullanılır.
    *   **Başlatma ve Temizleme:**
        *   `Initialize()`: Tüm arazi verilerini (su yükseklikleri, gölge haritası, alfa dokuları vb.) varsayılan/boş değerlere sıfırlar.
        *   `Clear()`: Varsa yüklenmiş alfa dokularını `Release()` eder ve ardından `Initialize()` çağırarak tüm durumu sıfırlar.
    *   **Harita Yükleme Fonksiyonları (`EterPackManager` Kullanımı):**
        *   `LoadHeightMap(const char* c_szFileName)`:
            *   `CEterPackManager` aracılığıyla belirtilen dosyadan ham yükseklik verilerini (`WORD` dizisi) okur ve `m_awRawHeightMap` dizisine kopyalar. Dosya boyutu `HEIGHTMAP_RAW_XSIZE * HEIGHTMAP_RAW_YSIZE * sizeof(WORD)` olmalıdır.
        *   `LoadAttrMap(const char* c_szFileName)`:
            *   Belirtilen öznitelik haritası dosyasını yükler.
            *   Dosyanın başında bir `SAttrMapHeader` (magic, width, height) bekler.
            *   Magic numarasını (`2634`), genişlik ve yüksekliği (`ATTRMAP_XSIZE`, `ATTRMAP_YSIZE`) kontrol eder.
            *   Başlık bilgisinden sonra gelen veriyi `m_abyAttrMap` dizisine kopyalar.
        *   `RAW_LoadTileMap(const char* c_szFileName)`:
            *   `CEterPackManager` aracılığıyla ham tile haritası verilerini (`BYTE` dizisi) okur ve `m_abyTileMap` dizisine kopyalar. Dosya boyutu `TILEMAP_RAW_XSIZE * TILEMAP_RAW_YSIZE` olmalıdır.
        *   `LoadWaterMap(const char* c_szFileName)` (ve `LoadWaterMapFile` yardımcı fonksiyonu):
            *   Belirtilen su haritası dosyasını yükler.
            *   Dosyanın başında bir `SWaterMapHeader` (magic, width, height, layerCount) bekler.
            *   Magic numarasını (`5426`), genişlik ve yüksekliği (`WATERMAP_XSIZE`, `WATERMAP_YSIZE`) kontrol eder.
            *   `m_byNumWater` (su katmanı sayısı) bu başlıktan okunur.
            *   Başlıktan sonra gelen su varlık haritasını (`m_abyWaterMap`) ve ardından her bir su katmanının yüksekliklerini (`m_lWaterHeight`) okur.
            *   Eski formatta su yüksekliklerinin `WORD` olarak saklandığı bir durumu da kontrol eder ve gerekirse `long` tipine dönüştürür.
    *   `GetShadowMapColor(float fx, float fy)`:
        *   Verilen dünya koordinatlarını (`fx`, `fy`) gölge haritası koordinatlarına dönüştürür.
        *   `m_awShadowMap` dizisinden ilgili `WORD` değerini okur.
        *   Bu `WORD` değerinin R5G5B5 formatında olduğunu varsayarak R, G, B bileşenlerini çıkarır ve 0xAARRGGBB formatında bir `DWORD` renk değeri olarak döndürür. (Not: Kodda G bileşeni hem G hem de B için kullanılmış gibi görünüyor: `(g << 16) | (g << 8) | r`. Bu bir hata olabilir veya istenen bir davranış olabilir.)

*   **Kullanım Amacı:**
    *   `CTerrainImpl`, genellikle istemcinin ana harita yönetim sınıfı (örneğin, `CMapOutdoor`) tarafından bir üye değişken olarak tutulur veya ondan miras alınır.
    *   Harita dosyaları yüklendiğinde, bu sınıfın yükleme fonksiyonları çağrılarak arazi verileri doldurulur.
    *   Oyun sırasında, render motoru araziyi çizmek için bu sınıftan yükseklik, doku ve gölge bilgilerini alır.
    *   Fizik ve çarpışma tespiti için öznitelik haritası ve yükseklik verileri kullanılır.
    *   Su efektleri için su haritası ve yükseklik bilgileri kullanılır.

*   **Bağımlılıklar:**
    *   `Stdafx.h` (ve dolayısıyla `EterLib`, `EterGrnLib`, `ScriptLib`, `UserInterface` başlıkları).
    *   `../EterPack/EterPackManager.h`: Dosya yükleme işlemleri için.
    *   `../EterImageLib/TGAImage.h`: `TGA_HEADER` için (kullanımı tam açık olmasa da).
    *   `TextureSet.h`: Arazi doku setlerini yönetmek için.
    *   `TerrainType.h`: `TTerrainSplatPatch` gibi araziye özgü veri yapıları için.
    *   `math.h`: Matematiksel fonksiyonlar için. 

### `TerrainType.h` (Arazi Veri Tipleri ve Sabitleri)

*   **Amaç:** `TerrainType.h` dosyası, `PRTerrainLib` kütüphanesi genelinde kullanılan temel arazi veri yapılarını, sabitlerini ve bazı yardımcı makroları tanımlar. Bu tanımlamalar, arazi yamaları, doku kaplama (splatting), vertex buffer'lar, materyaller ve genel arazi ayarları gibi çeşitli bileşenler için standart bir yapı sunar.

*   **Temel Sabitler:**
    *   `TERRAIN_PATCHSIZE` (16): Bir arazi "patch"inin (yamasının) bir kenarındaki hücre sayısı.
    *   `TERRAIN_SIZE` (128): Arazinin toplam hücre cinsinden bir kenar boyutu.
    *   `TERRAIN_PATCHCOUNT` (`TERRAIN_SIZE / TERRAIN_PATCHSIZE`): Arazinin bir kenarındaki patch sayısı (128/16 = 8).
    *   `MAXTERRAINTEXTURES` (256): Kullanılabilecek maksimum arazi dokusu sayısı.

*   **Ana Veri Yapıları (Struct'lar):**
    *   **`TTerainSplat`:** Tek bir splat (doku kaplama katmanı) dokusunun durumunu tutar.
        *   `Active` (long): Splat'in aktif olup olmadığı.
        *   `NeedsUpdate` (long): Splat dokusunun güncellenmesi gerekip gerekmediği.
        *   `pd3dTexture` (LPDIRECT3DTEXTURE8): Direct3D doku nesnesine işaretçi.
    *   **`TTerrainSplatPatch`:** Arazi genelindeki tüm splatting bilgilerini yönetir.
        *   `TileCount[MAXTERRAINTEXTURES]` (DWORD): Her bir doku katmanının toplam tile (karosu) sayısı.
        *   `PatchTileCount[TERRAIN_PATCHCOUNT * TERRAIN_PATCHCOUNT][MAXTERRAINTEXTURES]` (DWORD): Her bir arazi patch'i için her bir doku katmanının tile sayısı.
        *   `Splats[MAXTERRAINTEXTURES]` (TTerainSplat): Tüm splat dokularının bir dizisi.
        *   `m_bNeedsUpdate` (bool): Genel splatting bilgisinin güncellenmesi gerekip gerekmediği.
    *   **`TERRAIN_VBUFFER`:** Arazi parçalarının render edilmesi için kullanılan bir vertex buffer'ı temsil eder.
        *   `used` (char): Bu buffer'ın kullanılıp kullanılmadığı.
        *   `mat` (short): Materyal indeksi.
        *   `vb` (CGraphicVertexBuffer): Vertex buffer nesnesi.
        *   `ib` (CGraphicIndexBuffer): Index buffer nesnesi.
        *   `VertexSize` (long): Vertex yapısının boyutu.
        *   `NumIndices` (short): Index sayısı.
        *   `minx, maxx, miny, maxy, minz, maxz` (float): Buffer'daki geometrinin sınırlayıcı kutusu (AABB).
    *   **`PR_MATERIAL`:** Bir materyalin özelliklerini tanımlar.
        *   `name[19]` (char): Materyal adı.
        *   `ambi_r, ambi_g, ambi_b, ambi_a` (float): Ambient (ortam) renk bileşenleri.
        *   `diff_r, diff_g, diff_b, diff_a` (float): Diffuse (dağınık) renk bileşenleri.
        *   `spec_r, spec_g, spec_b, spec_a` (float): Specular (yansıma) renk bileşenleri.
        *   `spec_power` (float): Specular gücü (parlaklık).
    *   **`TTerrainGlobals`:** Arazi için genel ayarları tutar.
        *   `PageUVLength` (float): Doku UV koordinatlarının sayfa uzunluğu.
        *   `SquaresPerTexture` (long): Bir dokunun kapladığı yükseklik haritası kare sayısı.
        *   `SplatTilesX`, `SplatTilesY` (long): Harita üzerindeki splat doku tile'larının X ve Y eksenlerindeki sayısı.
        *   `DisableWrapping` (long): Doku sarma (wrapping) özelliğinin kapalı olup olmadığı.
        *   `DisableShadow` (long): Gölgelerin kapalı olup olmadığı.
        *   `ShadowMode` (long): Gölge modu.
        *   `OutsideVisible` (long): Harita dışındaki alanların görünüp görünmediği.
        *   `SunLocation` (D3DXVECTOR3): Güneşin konumu.

*   **Hızlı Float-Integer Dönüşüm Makroları:**
    *   `PR_FLOAT_TO_INTASM`: FPU'nun `fistp` komutunu kullanarak `PR_FCNV` (float) global değişkenini `PR_ICNV` (long) global değişkenine dönüştüren assembly kodunu içerir.
    *   `PR_FLOAT_TO_FIXED(inreal, outint)`: Bir float değeri sabit noktalı (fixed-point) bir tam sayıya dönüştürür (`* 65536.0f`).
    *   `PR_FLOAT_TO_INT(inreal, outint)`: Bir float değeri keserek (truncate) tam sayıya dönüştürür. Sonucun orijinal float'tan büyük olması durumunda 1 çıkararak düzeltme yapar.
    *   `PR_FLOAT_ADD_TO_INT(inreal, outint)`: Bir float değeri tam sayıya dönüştürüp mevcut bir tam sayıya ekler.
    *   Bu makrolar, `StdAfx.h` ve `StdAfx.cpp`'de tanımlanan `PR_FCNV` ve `PR_ICNV` global değişkenlerini kullanır.

*   **Kullanım Amacı:**
    *   `CTerrainImpl` ve diğer arazi ile ilgili sınıflar için temel veri yapısı tanımlamalarını sağlar.
    *   Arazi geometrisinin, dokularının ve materyallerinin nasıl organize edileceğini ve yönetileceğini belirler.
    *   Performans açısından kritik olabilecek float'tan integer'a dönüşümler için optimize edilmiş makrolar sunar.

*   **Bağımlılıklar:**
    *   `../Eterlib/GrpVertexBuffer.h`: `CGraphicVertexBuffer` için.
    *   `../Eterlib/GrpIndexBuffer.h`: `CGraphicIndexBuffer` için.
    *   DirectX (D3DXVECTOR3, LPDIRECT3DTEXTURE8). 

### `TextureSet.h` ve `TextureSet.cpp` (`CTextureSet` Sınıfı)

*   **Amaç:** `CTextureSet` sınıfı, bir arazi için kullanılacak doku koleksiyonlarını (setlerini) yönetir. Bu, doku dosyalarının tanımlandığı bir yapılandırma dosyasını yüklemeyi, her bir doku için UV ölçekleme/kaydırma gibi özellikleri ayarlamayı, dokuları çalışma zamanında ekleyip çıkarmayı ve bu doku seti yapılandırmasını tekrar dosyaya kaydetmeyi içerir.

*   **`TextureSet.h` - Temel Tanımlar ve Bildirimler:**
    *   **`STerrainTexture` (typedef `TTerrainTexture`):** Tek bir arazi dokusunun özelliklerini tanımlayan yapı.
        *   `stFilename` (std::string): Doku dosyasının adı.
        *   `pd3dTexture` (LPDIRECT3DTEXTURE8): Direct3D doku nesnesine işaretçi.
        *   `ImageInstance` (CGraphicImageInstance): Doku görüntüsünün bir örneği.
        *   `UScale`, `VScale` (float): Doku UV koordinatları için ölçekleme faktörleri.
        *   `UOffset`, `VOffset` (float): Doku UV koordinatları için kaydırma (offset) miktarları.
        *   `bSplat` (bool): Bu dokunun splatting için kullanılıp kullanılmayacağı.
        *   `Begin`, `End` (unsigned short): Genellikle yükseklik haritası değerlerine göre dokunun uygulanacağı aralığı belirtir (0-65535).
        *   `m_matTransform` (D3DXMATRIX): Doku koordinat dönüşüm matrisi (ölçekleme ve kaydırmayı içerir).
    *   **`CTextureSet` Sınıfı:**
        *   `typedef std::vector<TTerrainTexture> TTextureVector;`
        *   **Yapıcı/Yıkıcı:** `CTextureSet()`, `~CTextureSet()`.
        *   **Temel Metotlar:**
            *   `Initialize()`: Üye değişkenleri başlangıç durumuna getirir.
            *   `Clear()`: Yüklenmiş tüm dokuları ve kaynakları temizler.
            *   `Create()`: Hata dokusunu (`m_ErrorTexture`) oluşturur ve genellikle ilk (0 indeksli) boş bir "silgi" dokusu ekler.
        *   **Dosya İşlemleri:**
            *   `Load(const char* c_pszFileName, float fTerrainTexCoordBase)`: Belirtilen `.set` (veya benzeri) dosyasından doku seti bilgilerini yükler.
            *   `Save(const char* c_pszFileName)`: Mevcut doku seti bilgilerini belirtilen dosyaya kaydeder.
        *   **Doku Yönetimi:**
            *   `GetTextureCount()`: Toplam doku sayısını döndürür.
            *   `GetTexture(unsigned long ulIndex)`: Belirli bir indeksteki `TTerrainTexture` referansını döndürür. İndeks geçersizse `m_ErrorTexture` döndürür.
            *   `SetTexture(...)`: Belirli bir indeksteki dokunun özelliklerini ayarlar (dosya adı, ölçek, offset vb.).
            *   `AddTexture(...)`: Doku setine yeni bir doku ekler. Dosya adı tekrarını ve maksimum doku sayısını kontrol eder.
            *   `RemoveTexture(unsigned long ulIndex)`: Belirli bir indeksteki dokuyu setten çıkarır.
            *   `Reload(float fTerrainTexCoordBase)`: Setteki tüm dokuları yeniden yükler.
        *   `GetFileName()`: Yüklenmiş olan doku seti dosyasının adını döndürür.
    *   **Korunan (Protected) Üyeler:**
        *   `AddEmptyTexture()`: `m_Textures` vektörüne boş bir `TTerrainTexture` ekler.
        *   `m_Textures` (TTextureVector): Doku nesnelerini tutan vektör.
        *   `m_ErrorTexture` (TTerrainTexture): Hata durumunda veya geçersiz indekslerde kullanılacak varsayılan doku.
        *   `m_stFileName` (std::string): Yüklenen doku seti dosyasının adı.

*   **`TextureSet.cpp` - Uygulama Detayları:**
    *   **`Create()`:**
        *   `CResourceManager` kullanarak bir hata dokusu (örn: "d:/ymir work/special/error.tga") yükler ve `m_ErrorTexture`'a atar.
        *   `AddEmptyTexture()` ile ilk (indeks 0) dokuyu ekler. Bu genellikle "silgi" veya varsayılan doku olarak işlev görür.
    *   **`Load()`:**
        *   `LoadMultipleTextData` (muhtemelen `EterBase`'den gelen bir yardımcı) ile metin tabanlı doku seti dosyasını ayrıştırır.
        *   Dosyada "textureset" ve "texturecount" anahtar kelimelerini arar.
        *   "texturecount" değerine göre `m_Textures` vektörünü yeniden boyutlandırır.
        *   "texture001", "texture002" gibi başlıklar altında her bir dokunun özelliklerini (dosya adı, UV ölçek/offset, splat durumu, yükseklik aralığı) okur.
        *   Okunan her doku için `SetTexture()` metodunu çağırır.
        *   `fTerrainTexCoordBase` parametresi, doku koordinatlarının genel bir ölçek faktörüyle çarpılmasında kullanılır.
    *   **`SetTexture()`:**
        *   Verilen indeksteki `TTerrainTexture` nesnesini alır.
        *   `CResourceManager` aracılığıyla doku dosyasını yükler ve `ImageInstance`'a atar.
        *   `pd3dTexture` üyesini `ImageInstance`'dan alır.
        *   UV ölçek, offset ve diğer özellikleri ayarlar.
        *   `D3DXMatrixScaling` kullanarak `fTerrainTexCoordBase`, `UScale`, `VScale` ile bir ölçekleme matrisi oluşturur ve `UOffset`, `VOffset` ile kaydırma yapar. Sonuç `m_matTransform` matrisinde saklanır. UV koordinatlarının Y ekseninin ters çevrildiğine dikkat edin (`-fTerrainTexCoordBase * tex.VScale`, `-tex.VOffset`).
    *   **`AddTexture()`:**
        *   Maksimum doku sayısını (256) kontrol eder.
        *   Aynı dosya adına sahip bir dokunun zaten var olup olmadığını kontrol eder.
        *   Yeni bir doku için yer ayırır, `AddEmptyTexture()` çağırır ve ardından `SetTexture()` ile özelliklerini ayarlar.
    *   **`Save()`:**
        *   Doku seti bilgilerini metin formatında dosyaya yazar.
        *   "TextureSet" başlığı, "TextureCount" ve ardından her bir doku için "Start TextureXXX" ... "End TextureXXX" blokları içinde özelliklerini (dosya adı, ölçek, offset vb.) yazar.
    *   **`Reload()`:**
        *   Mevcut tüm dokuları (indeks 0 hariç) `CResourceManager::GetResourcePointer()` ile yeniden yükler ve `m_matTransform` matrislerini günceller.
    *   **`Clear()`:** `m_ErrorTexture`'ı ve `m_Textures` vektöründeki tüm dokuları `Destroy()` eder/temizler.

*   **Kullanım Amacı:**
    *   `CTerrainImpl` sınıfı tarafından, araziye uygulanacak farklı doku katmanlarını (çimen, kaya, toprak vb.) ve bu dokuların nasıl döşeneceğini (ölçekleme, kaydırma) tanımlamak için kullanılır.
    *   Harita editörleri veya araçları, bu sınıf aracılığıyla doku setleri oluşturabilir ve düzenleyebilir.
    *   Oyun yükleme sırasında, `CTerrainImpl::SetTextureSet()` ile bir `CTextureSet` örneği araziye atanır ve arazi render sistemi bu doku bilgilerini kullanır.

*   **Bağımlılıklar:**
    *   `Stdafx.h`
    *   `../EterLib/GrpImageInstance.h`: `CGraphicImageInstance` için.
    *   `../EterLib/ResourceManager.h` (dolaylı olarak `GrpImageInstance.h` veya `StdAfx.h` üzerinden): Doku kaynaklarını yüklemek için.
    *   `../EterBase/鮒밑嶺滲LoadMultipleTextData.h` (veya benzeri bir token parser, `LoadMultipleTextData` fonksiyonu için).
    *   Standart C++ kütüphaneleri (`vector`, `string`, `cstdio`).
    *   DirectX (LPDIRECT3DTEXTURE8, D3DXMATRIX). 