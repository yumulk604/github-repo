# Metin2 İstemci (srcClient) Kaynak Kodu Referans Kılavuzu

Bu kılavuz, Metin2 istemci kaynak kodunun (`srcClient`) yapısını, önemli klasörlerini ve temel bileşenlerini anlamak için bir referans noktası sağlamayı amaçlamaktadır. İstemcinin kullanıcı arayüzü, oyun mantığı, grafik işleme, ağ iletişimi ve diğer temel sistemleri burada ana hatlarıyla belgelenecektir.

## İçindekiler

- [Proje Yapılandırma Dosyaları](#proje-yapılandırma-dosyaları)
  - [`M2Client.sln` (Visual Studio Solution Dosyası)](#m2clientsln-visual-studio-solution-dosyası)
  - [`VSFiles/` Klasörü (Visual Studio Proje Dosyaları)](#vsfiles-klasörü-visual-studio-proje-dosyaları)
- [Genel Bakış ve Ana Bileşenler](#genel-bakış-ve-ana-bileşenler)
- [Harici Bağımlılıklar (`External/`)](#harici-bağımlılıklar-external)
- [Derlenmiş Çıktılar, Kütüphaneler ve Yapılandırmalar](#derlenmiş-çıktılar-kütüphaneler-ve-yapılandırmalar)
  - [`Binary/` Klasörü (Release Sürümü)](#binary-klasörü-release-sürümü)
  - [`Debug/` Klasörü (Debug Sürümü Kütüphaneleri)](#debug-klasörü-debug-sürümü-kütüphaneleri)
- [Yardımcı Kaynaklar (`Resources/`)](#yardımcı-kaynaklar-resources)
- [İstemci Kaynak Kodu (`Source/`)](#istemci-kaynak-kodu-source)

## Proje Yapılandırma Dosyaları

Bu bölümde, projenin genel yapısını ve derleme süreçlerini tanımlayan ana yapılandırma dosyaları açıklanmaktadır.

### `M2Client.sln` (Visual Studio Solution Dosyası)

`@srcClient` kök dizininde bulunan `M2Client.sln` dosyası, Microsoft Visual Studio için ana çözüm dosyasıdır. Bu dosyanın temel işlevleri şunlardır:

*   **Proje Konteyneri:** Metin2 istemcisini oluşturan tüm alt projeleri (örneğin, `EterLib`, `GameLib`, `UserInterface`, `EffectLib` gibi kütüphaneler ve modüller) bir araya getirir.
*   **Proje Yolları:** Her bir alt projenin `.vcxproj` (Visual Studio C++ Proje Dosyası) dosyasının konumunu belirtir (genellikle `VSFiles/` altındaki ilgili proje klasöründe).
*   **Yapılandırma Yönetimi:** İstemcinin farklı derleme yapılandırmalarını (örneğin, `Debug`, `Release`, `Distribute`) ve platformlarını (`Win32`, `x64`) tanımlar. Bu, projenin farklı amaçlar ve ortamlar için nasıl derleneceğini yönetir.
*   **Derleme Yönetimi:** Visual Studio'nun tüm projeyi doğru bağımlılıklar ve sıralama ile derlemesini sağlar.

Kısacası, `M2Client.sln` dosyası, Metin2 istemci projesinin Visual Studio IDE'si (Entegre Geliştirme Ortamı) içinde açılması, yönetilmesi ve derlenmesi için merkezi bir rol oynar.

### `VSFiles/` Klasörü (Visual Studio Proje Dosyaları)

`@srcClient/VSFiles/` klasörü, `M2Client.sln` çözüm dosyasında listelenen her bir alt proje (modül veya kütüphane) için Microsoft Visual Studio'ya özgü proje dosyalarını içerir. Bu klasördeki her bir alt klasör (örneğin, `VSFiles/EterLib/`, `VSFiles/GameLib/`) genellikle şunları barındırır:

*   **`.vcxproj` Dosyası:** Her modülün C++ proje dosyasıdır. Bu dosya, modüle ait kaynak dosyalarını, derleyici ve bağlayıcı ayarlarını, bağımlılıkları ve diğer proje yapılandırma bilgilerini tanımlar.
*   **`.vcxproj.filters` Dosyası:** Kaynak dosyalarının Visual Studio Çözüm Gezgini'nde nasıl organize edileceğini (sanal klasörler) belirler.

Bu dosyalar, `Source/` klasöründeki gerçek kaynak kod dosyalarıyla birlikte çalışarak her bir modülün Visual Studio tarafından nasıl derleneceğini ve yönetileceğini tanımlar. `M2Client.sln` dosyası, bu `VSFiles/` altındaki `.vcxproj` dosyalarına başvurarak tüm istemci çözümünü oluşturur.

## Genel Bakış ve Ana Bileşenler

Bu bölüm, `@srcClientMD` dokümantasyon projesi kapsamında, Metin2 istemcisinin (`@srcClient/Source/` altındaki) ana modüllerine ve bileşenlerine karşılık gelen dokümantasyon klasörlerini ve bu klasörler içinde bulunan Markdown formatındaki referans dosyalarını listeler. Bu yapı, projenin genel dokümantasyon haritasını sunar.

*   **`CWebBrowser/`**:
    *   [`client_CWebBrowser_Referans.md`](./CWebBrowser/client_CWebBrowser_Referans.md)
*   **`EffectLib/`**:
    *   [`client_EffectLib_Referans.md`](./EffectLib/client_EffectLib_Referans.md)
    *   [`client_EffectLib_Referans_Part2.md`](./EffectLib/client_EffectLib_Referans_Part2.md)
    *   [`client_EffectLib_Referans_Part3.md`](./EffectLib/client_EffectLib_Referans_Part3.md)
*   **`EterBase/`**:
    *   [`client_EterBase_Referans.md`](./EterBase/client_EterBase_Referans.md)
    *   [`client_EterBase_Referans_Part2.md`](./EterBase/client_EterBase_Referans_Part2.md)
*   **`EterGrnLib/`**:
    *   [`client_EterGrnLib_Referans.md`](./EterGrnLib/client_EterGrnLib_Referans.md)
*   **`EterImageLib/`**:
    *   [`client_EterImageLib_Referans.md`](./EterImageLib/client_EterImageLib_Referans.md)
*   **`EterLib/`**:
    *   [`client_EterLib_Referans.md`](./EterLib/client_EterLib_Referans.md)
    *   [`client_EterLib_Referans_Part2.md`](./EterLib/client_EterLib_Referans_Part2.md)
    *   [`client_EterLib_Referans_Part3.md`](./EterLib/client_EterLib_Referans_Part3.md)
    *   [`client_EterLib_Referans_Part4.md`](./EterLib/client_EterLib_Referans_Part4.md)
    *   [`client_EterLib_Referans_Part5.md`](./EterLib/client_EterLib_Referans_Part5.md)
*   **`EterLocale/`**:
    *   [`client_EterLocale_Referans.md`](./EterLocale/client_EterLocale_Referans.md)
*   **`EterPack/`**:
    *   [`client_EterPack_Referans.md`](./EterPack/client_EterPack_Referans.md)
*   **`EterPythonLib/`**:
    *   [`client_EterPythonLib_Referans.md`](./EterPythonLib/client_EterPythonLib_Referans.md)
*   **`GameLib/`**:
    *   [`client_GameLib_Referans.md`](./GameLib/client_GameLib_Referans.md)
    *   [`client_GameLib_Referans_Part2.md`](./GameLib/client_GameLib_Referans_Part2.md)
    *   [`client_GameLib_Referans_Part3.md`](./GameLib/client_GameLib_Referans_Part3.md)
    *   [`client_GameLib_Referans_Part4.md`](./GameLib/client_GameLib_Referans_Part4.md)
    *   [`client_GameLib_Referans_Part5.md`](./GameLib/client_GameLib_Referans_Part5.md)
*   **`MilesLib/`**:
    *   [`client_MilesLib_Referans.md`](./MilesLib/client_MilesLib_Referans.md)
*   **`PRTerrainLib/`**:
    *   [`client_PRTerrainLib_Referans.md`](./PRTerrainLib/client_PRTerrainLib_Referans.md)
*   **`ScriptLib/`**:
    *   [`client_ScriptLib_Referans.md`](./ScriptLib/client_ScriptLib_Referans.md)
*   **`SpeedTreeLib/`**:
    *   [`client_SpeedTreeLib_Referans.md`](./SpeedTreeLib/client_SpeedTreeLib_Referans.md)
*   **`SphereLib/`**:
    *   [`client_SphereLib_Referans.md`](./SphereLib/client_SphereLib_Referans.md)
*   **`UserInterface/`**:
    *   [`client_UserInterface_Referans.md`](./UserInterface/client_UserInterface_Referans.md)
    *   [`client_UserInterface_Referans_Part2.md`](./UserInterface/client_UserInterface_Referans_Part2.md)
    *   [`client_UserInterface_Referans_Part3.md`](./UserInterface/client_UserInterface_Referans_Part3.md)
    *   [`client_UserInterface_Referans_Part4.md`](./UserInterface/client_UserInterface_Referans_Part4.md)
    *   [`client_UserInterface_Referans_Part5.md`](./UserInterface/client_UserInterface_Referans_Part5.md)
    *   [`client_UserInterface_Referans_Part6.md`](./UserInterface/client_UserInterface_Referans_Part6.md)
    *   [`client_UserInterface_Referans_Part7.md`](./UserInterface/client_UserInterface_Referans_Part7.md)
    *   [`client_UserInterface_Referans_Part8.md`](./UserInterface/client_UserInterface_Referans_Part8.md)
    *   [`client_UserInterface_Referans_Part9.md`](./UserInterface/client_UserInterface_Referans_Part9.md)
    *   [`client_UserInterface_Referans_Part10.md`](./UserInterface/client_UserInterface_Referans_Part10.md)
    *   [`client_UserInterface_Referans_Part11.md`](./UserInterface/client_UserInterface_Referans_Part11.md)
    *   [`client_UserInterface_Referans_Part12.md`](./UserInterface/client_UserInterface_Referans_Part12.md)
    *   [`client_UserInterface_Referans_Part13.md`](./UserInterface/client_UserInterface_Referans_Part13.md)
    *   [`client_UserInterface_Referans_Part14.md`](./UserInterface/client_UserInterface_Referans_Part14.md)
    *   [`client_UserInterface_Referans_Part15.md`](./UserInterface/client_UserInterface_Referans_Part15.md)
    *   [`client_UserInterface_Referans_Part16.md`](./UserInterface/client_UserInterface_Referans_Part16.md)
*   **`VSFiles/`**:
    *   [`client_VSFiles_Referans.md`](./VSFiles/client_VSFiles_Referans.md)
    *   [`client_VSFiles_Referans_Part2.md`](./VSFiles/client_VSFiles_Referans_Part2.md)
    *   [`client_VSFiles_Referans_Part3.md`](./VSFiles/client_VSFiles_Referans_Part3.md)
    *   [`client_VSFiles_Referans_Part4.md`](./VSFiles/client_VSFiles_Referans_Part4.md)

## Harici Bağımlılıklar (`External/`)

`@srcClient/External/` klasörü, Metin2 istemci projesinin derlenmesi ve çalışması için gereken üçüncü parti kütüphaneleri ve diğer harici bağımlılıkları içerir. Bu, projenin kendi kaynak kodunu dış bileşenlerden ayrı tutarak düzenli bir yapı sağlar.

*   **`include/`**: Bu alt klasör, harici kütüphanelerin başlık dosyalarını (`.h`, `.hpp`, `.inl`) barındırır. Bu dosyalar, harici kütüphanelerin fonksiyon, sınıf ve veri yapısı bildirimlerini içerir. Ana proje kodu, bu kütüphanelerin arayüzlerine erişmek için bu dosyaları kullanır. Klasörün yapısı ve önemli içerikleri şunlardır:
    *   **Alt Klasörler (Kütüphane Bazlı):**
        *   `d3d9/`: DirectX 9 ile ilgili ek başlık dosyalarını içerebilir.
        *   `Python-2.7/`: Python 2.7 C API başlık dosyaları.
        *   `minini_12b/`: MinINI (INI dosyası okuyucu) kütüphanesi başlıkları.
        *   `minilzo/`, `lzo/`: MiniLZO ve LZO veri sıkıştırma kütüphanesi başlıkları.
        *   `libjpeg-turbo/`: libjpeg-turbo (JPEG işleme) kütüphanesi başlıkları.
        *   `il/`: DevIL (Developer's Image Library) başlık dosyaları.
        *   `cryptopp/`: Crypto++ (kriptografi kütüphanesi) başlıkları.
        *   `cef/`: Chromium Embedded Framework başlık dosyaları.
    *   **Doğrudan Klasör İçindeki Önemli Başlık Dosyaları/Grupları:**
        *   **DirectX Başlıkları:** `d3d9.h`, `d3d9types.h`, `d3dx9.h`, `d3dx9anim.h`, `d3dx9core.h`, `d3dx9effect.h`, `d3dx9math.h`, `d3dx9math.inl`, `d3dx9mesh.h`, `d3dx9shader.h`, `d3dx9shape.h`, `d3dx9tex.h`, `d3dx9xof.h`. Ayrıca DirectX 8 ile ilgili `d3d8.h`, `d3dx8.h` serisi başlıklar da bulunur.
        *   `SpeedTreeRT.h`: SpeedTree (gerçek zamanlı bitki örtüsü) kütüphanesi.
        *   `MSS.H`: Miles Sound System (ses motoru).
        *   `granny.h`, `granny2_spu_samplemodel.h`: Granny 3D (animasyon sistemi).
        *   `discord_rpc.h`: Discord Rich Presence entegrasyonu.
        *   `qedit.h`: Microsoft DirectShow ile ilgili (video/ses işleme).
        *   `radtypes.h`: Muhtemelen RAD Game Tools ile ilişkili temel tür tanımları.
*   **`library/`**: Bu alt klasör, `include/` klasöründe başlık dosyalarıyla arayüzleri tanımlanan harici kütüphanelerin önceden derlenmiş gerçek kodlarını içerir. Bu dosyalar genellikle statik kütüphane (`.lib`) formatındadır. Ana istemci projesi derlenirken, bağlayıcı (linker) bu `.lib` dosyalarını kullanarak kaynak koddaki harici fonksiyon çağrılarını ve sınıf kullanımlarını çözümler. Önemli içerikler şunlardır:
    *   **Derlenmiş Kütüphaneler (`.lib`):**
        *   `include/` altında başlıkları bulunan birçok kütüphanenin (örneğin, CEF, Crypto++, DirectX (D3D8, D3D9, D3DX, DirectInput, DirectSound), DevIL, Discord RPC, Granny 3D, libjpeg-turbo, LZO, Miles Sound System, Python, SpeedTreeRT) derlenmiş `.lib` dosyaları burada yer alır.
        *   Bazı kütüphaneler için hem **Release** (örneğin, `cryptlib.lib`) hem de **Debug** (örneğin, `cryptlibd.lib`, genellikle 'd' sonekine sahip) sürümleri bulunabilir. Debug sürümleri, hata ayıklama bilgileri içerir.
        *   DirectX'in daha yeni sürümlerine (D3D10, D3D11, DXGI) ait `.lib` dosyaları da görülebilir.
        *   `XInput.lib` (Xbox kontrolcü desteği) ve `X3DAudio.lib` (3D ses) gibi ek DirectX bileşenleri de bulunabilir.
    *   **Hata Ayıklama Sembolleri (`.pdb`):**
        *   Bazı `.lib` dosyalarına eşlik eden `.pdb` (Program Database) dosyaları, ilgili kütüphanenin hata ayıklama sembollerini içerir. Bu semboller, derlenmiş kodun orijinal kaynak koda geri eşlenmesini sağlayarak geliştiricilerin programı bir hata ayıklayıcıda (debugger) adım adım izlemesine, değişkenleri incelemesine ve hataları daha kolay bulmasına olanak tanır.

Bu klasördeki bileşenler, istemcinin temel işlevselliğinin ötesinde, örneğin grafik, ses, ağ veya diğer özel görevler için ek yetenekler sağlamak amacıyla kullanılır.

## Derlenmiş Çıktılar, Kütüphaneler ve Yapılandırmalar

Bu bölümde, istemci projesinin derlenmesi sonucu ortaya çıkan dosyalar ve farklı derleme yapılandırmalarıyla (örneğin "Debug" ve "Release") ilişkili klasörler açıklanmaktadır.

### `Binary/` Klasörü (Release Sürümü)

`@srcClient/Binary/` klasörü, Metin2 istemci projesinin genellikle son kullanıcıya dağıtılmak üzere optimize edilmiş "Release" sürümünün derlenmiş çıktılarını içerir.

*   **`Metin2Release.exe`**: Bu, derlenmiş ana istemci uygulamasıdır. Oyunun çalıştırılabilir "Release" sürümünü temsil eder.
*   **`Metin2Release.bsc`**: Microsoft Visual Studio tarafından oluşturulan bir gözatma bilgi dosyasıdır. Kaynak kod sembollerinin hızlı navigasyonu için geliştirme sırasında kullanılır ve genellikle son kullanıcıya dağıtılmaz.
*   **`Client.lnk`**: `Metin2Release.exe` dosyasına işaret eden bir Windows kısayol dosyasıdır, istemcinin kolayca başlatılmasına olanak tanır.

Bu klasör, kaynak kodun son kullanıcı için optimize edilmiş, derlenmiş ve çalışmaya hazır halini barındırır.

### `Debug/` Klasörü (Debug Sürümü Kütüphaneleri)

`@srcClient/Debug/` klasörü, istemciyi oluşturan çeşitli kütüphane modüllerinin "Debug" (Hata Ayıklama) yapılandırmasıyla derlenmiş sürümlerini ve bunlarla ilişkili hata ayıklama sembollerini içerir. Bu dosyalar, geliştirme sırasında hataları tespit etmek ve çözmek için kullanılır.

*   **`.lib` Dosyaları (Statik Kütüphaneler):**
    *   Bunlar, projenin `EterLib`, `GameLib`, `EterGrnLib`, `EffectLib` gibi çeşitli modüllerinin derlenmiş kodunu içeren statik kütüphane dosyalarıdır.
    *   Bu kütüphaneler, ana çalıştırılabilir dosya oluşturulurken birbirine bağlanır. "Debug" klasöründe bulunmaları, hata ayıklama bilgileriyle birlikte derlendiklerini gösterir.
*   **`.pdb` Dosyaları (Program Veritabanı Dosyaları):**
    *   Her `.lib` dosyasına karşılık gelen bir `.pdb` dosyası bulunur (örneğin, `GameLib.lib` ve `GameLib.pdb`).
    *   `.pdb` dosyaları, hata ayıklama sembollerini içerir. Bu semboller, derlenmiş kodun orijinal kaynak koda geri eşlenmesini sağlayarak geliştiricilerin programı bir hata ayıklayıcıda (debugger) adım adım izlemesine, değişkenleri incelemesine ve hataları daha kolay bulmasına olanak tanır.

"Debug" klasöründeki dosyalar genellikle "Release" sürümündeki karşılıklarından daha büyüktür ve performansları biraz daha düşük olabilir, çünkü ek hata ayıklama bilgileri içerirler. Geliştirme tamamlandığında ve ürün dağıtıma hazır olduğunda, genellikle `Binary/` klasöründeki gibi optimize edilmiş "Release" sürümleri kullanılır.

## Yardımcı Kaynaklar (`Resources/`)

`@srcClient/Resources/` klasörü, istemcinin veya entegre edilmiş harici kütüphanelerin çalışması için ihtiyaç duyduğu kod olmayan varlıkları, yapılandırma dosyalarını veya diğer yardımcı kaynakları içerir. Bu dosyalar genellikle programın çalışma zamanında eriştiği destekleyici materyallerdir.

Belirlenen alt klasörler şunlardır:

*   **`DiscordRPC/`**: Discord Rich Presence özelliği için gerekli ek kaynakları (örneğin, ikonlar, yapılandırmalar) içerebilir.
*   **`Cythonize/`**: Python betiklerinin Cython aracıyla C modüllerine dönüştürülmesiyle ilgili dosyaları veya Cython'ın çalışması için gereken kaynakları barındırabilir. Bu, genellikle Python performansını artırmak veya C/C++ ile entegrasyonu kolaylaştırmak için yapılır.
*   **`CEF/`**: Chromium Embedded Framework (CEF) için gerekli kaynak dosyalarını (örneğin, yerelleştirme paketleri `.pak`, yazı tipleri, JavaScript uzantıları vb.) içerir.

Bu klasördeki bileşenler, istemcinin temel işlevselliğinin ötesinde, örneğin grafik, ses, ağ veya diğer özel görevler için ek yetenekler sağlamak amacıyla kullanılır.