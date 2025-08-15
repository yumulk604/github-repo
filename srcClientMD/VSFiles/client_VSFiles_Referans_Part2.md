# VSFiles Klasörü Referans Kılavuzu - Bölüm 2

Bu belge, Metin2 istemci kaynak kodu (`@srcClient`) içindeki `VSFiles` klasörünün yapısını ve amacını açıklamaya devam etmektedir.

---

### `EterImageLib.vcxproj` (Resim İşleme Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisinde kullanılan çeşitli resim formatlarını (örneğin `.tga`, `.dds` gibi sıkıştırılmış dokular) yüklemek, işlemek ve yönetmek için kullanılan `EterImageLib` statik kütüphanesini (`.lib`) derlemek amacıyla kullanılır. Bu kütüphane, oyun içi dokuların, arayüz elemanlarının ve diğer görsel varlıkların işlenmesinde kritik bir rol oynar.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{3AA4BFA3-DF0E-42B9-A82C-E1BE16139AF3}`
*   **Kök Ad Alanı (RootNamespace)**: `EterImageLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterImageLib.lib üretir)

**`EterImageLib.vcxproj.user` Dosyası:**

Bu dosya boştur ve kullanıcıya özel ayar içermez.

**`EterImageLib.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını "Source Files" ve "Header Files" filtreleri altında düzenler. Kütüphaneyi oluşturan temel bileşenler şunlardır:
*   `DXTCImage.cpp/h`: DirectX Doku Sıkıştırma (DXTC/S3TC) formatındaki resimlerin işlenmesi.
*   `Image.cpp/h`: Genel bir resim sınıfı ve temel resim işleme fonksiyonları.
*   `StdAfx.cpp/h`: Ön derlenmiş başlıklar.
*   `TGAImage.cpp/h`: Targa (.tga) resim formatının yüklenmesi ve işlenmesi.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/EterImageLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Taşınabilirlik için bu yolların göreceli olması tercih edilir.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterImageLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `true` (/LTCG) - Bu ayar `EterGrnLib` ve `EterBase`'den farklı olarak burada `true`.
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterImageLib/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `Use` (`StdAfx.h` kullanılır), `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`EterImageLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **bulunmamaktadır**. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel Visual Studio ayarlarından miras alınmaktadır. Bu platformlar için derleme yapılıyorsa, `_WIN64` ön işlemci tanımı, doğru çalışma zamanı kütüphanesi ve PCH ayarları gibi kritik konfigürasyonların kontrol edilmesi ve gerekirse tanımlanması önemlidir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir. ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır.

**Kaynak Dosyalar:**

Proje, `../../Source/EterImageLib/` dizini altında resim işleme ile ilgili C++ kaynak (`.cpp`) ve başlık (`.h`) dosyalarını içerir:
*   `DXTCImage.cpp/h`
*   `Image.cpp/h`
*   `StdAfx.cpp/h`
*   `TGAImage.cpp/h`

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   `StdAfx.cpp` dosyası `Debug|Win32`, `Release|Win32`, `Distribute|Win32` ve tüm ARM yapılandırmalarında PCH oluşturmak (`Create`) üzere ayarlanmıştır.
*   `Debug|Win32` ve `Release|Win32` yapılandırmalarında, genel PCH ayarı `NotUsing` iken, `StdAfx.cpp` PCH oluşturur.
*   `Distribute|Win32` ve tüm ARM yapılandırmalarında genel PCH ayarı `Use` şeklindedir.
*   x64 platformları için PCH ayarları `ItemDefinitionGroup` eksikliği nedeniyle belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterImageLib`, statik bir kütüphane (`.lib`) olarak derlenir ve grafik motoru, UI sistemi gibi istemcinin görsel öğelerle çalışan kısımları tarafından kullanılır.
*   Proje, `../../External/include` altındaki harici başlık dosyalarına (örneğin, `libjpeg`, `libpng`, `DirectX SDK` başlıkları gibi) bağımlı olabilir. `DXTCImage` için DirectX SDK'sına bağımlılık olması muhtemeldir.
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\Développement\...`) içermektedir.
*   **x64 Yapılandırma Eksiklikleri**: Proje dosyasında `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması, bu platform için derleme ayarlarının varsayılanlara veya paylaşılan `.props` dosyalarına dayandığı anlamına gelir. Bu durum, `_WIN64` tanımının eksikliği, yanlış çalışma zamanı kütüphanesi seçimi veya PCH sorunları gibi potansiyel problemlere yol açabilir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması, platforma özgü kodlarda sorunlara yol açabilir. `_ARM_` gibi daha spesifik makrolar kullanılmalıdır.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneğinin (`/LTCG`) `true` olarak ayarlanması, bağlantı zamanında optimizasyon yaparak daha performanslı bir kod üretebilir ancak derleme süresini artırabilir. Bu, diğer Eter kütüphanelerinin Release yapılandırmalarından farklı bir ayardır.

---

### `EterLib.vcxproj` (Genel Amaçlı Eter Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisinin temel sistemlerinin ve yardımcı fonksiyonlarının büyük bir bölümünü içeren kapsamlı `EterLib` statik kütüphanesini (`.lib`) derlemek için kullanılır. `EterLib`, grafiksel kullanıcı arayüzü (GUI) elemanları, 2D/3D grafik işleme, ağ iletişimi, kaynak yönetimi, metin işleme, font yönetimi, giriş/çıkış (I/O) işlemleri ve daha birçok temel işlevi barındırır. Neredeyse diğer tüm istemci modülleri için birincil bir bağımlılıktır.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{634437B1-4B3B-48C5-9220-619FB4D8F99B}`
*   **Kök Ad Alanı (RootNamespace)**: `EterLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterLib.lib üretir)

**`EterLib.vcxproj.user` Dosyası:**

Bu dosya boştur ve kullanıcıya özel ayar içermez.

**`EterLib.vcxproj.filters` Dosyası:**

Proje içindeki çok sayıda kaynak ve başlık dosyasını çeşitli filtreler altında düzenler. Başlıca filtreler ve içerdiği bazı önemli bileşenler şunlardır:
*   **Culling**: Görüş alanı dışındaki nesnelerin çizilmemesi (`CullingManager.cpp/h`).
*   **Font**: Eski font sistemi (`GrpFontTexture.cpp/h`, `GrpText.cpp/h`, `GrpTextInstance.cpp/h`).
*   **Font_New**: Yeni font sistemi (`BlockTexture.cpp/h`, `DibBar.cpp/h`, `TextBar.cpp/h`).
*   **Graphic**: Temel grafik işlemleri, cihaz yönetimi, matematik fonksiyonları, ekran yönetimi (`GrpBase.cpp/h`, `GrpDevice.cpp/h`, `GrpMath.cpp/h`, `GrpScreen.cpp/h`, `StateManager.cpp/h`).
*   **Image**: 2D resimler, dokular ve örnekleri (`GrpImage.cpp/h`, `GrpImageInstance.cpp/h`, `GrpTexture.cpp/h`, `GrpSubImage.cpp/h`, `GrpRenderTargetTexture.cpp/h`).
*   **IME**: Giriş Metodu Düzenleyicisi (Input Method Editor) yönetimi (`IME.cpp/h`, `Dimm.h`, `msctf.h`).
*   **Network**: Ağ adresi, datagram, akış ve cihaz yönetimi (`NetAddress.cpp/h`, `NetDatagram.cpp/h`, `NetStream.cpp/h`, `NetDevice.cpp/h`).
*   **Resource**: Kaynak yükleme ve yönetimi (`Resource.cpp/h`, `ResourceManager.cpp/h`, `TargaResource.cpp/h`).
*   **Text**: Metin dosyası yükleme (`TextFileLoader.cpp/h`, `GroupTextParseTree.cpp/h`).
*   **Source Files / Header Files**: Yukarıdaki kategorilere tam olarak girmeyen diğer birçok temel sınıf ve yardımcı program (örn: `Camera.cpp/h`, `CollisionData.cpp/h`, `Input.cpp/h`, `MSApplication.cpp/h`, `MSWindow.cpp/h`, `Thread.cpp/h`, `Util.cpp/h`, `SkyBox.cpp/h`, `RenderTarget.cpp/h`, `RenderTargetManager.cpp/h` vb.).

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/EterLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false` (Diğer bazı kütüphanelerde Release'de `true` olabiliyordu).
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterLib/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `Use` (`StdAfx.h` kullanılır), `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`EterLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **bulunmamaktadır**. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel Visual Studio ayarlarından miras alınmaktadır. Bu platformlar için derleme yapılıyorsa, `_WIN64` ön işlemci tanımı, doğru çalışma zamanı kütüphanesi ve PCH ayarları gibi kritik konfigürasyonların kontrol edilmesi ve gerekirse tanımlanması önemlidir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir. ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır.

**Kaynak Dosyalar:**

Proje, `../../Source/EterLib/` dizini altında yukarıda `EterLib.vcxproj.filters` bölümünde özetlenen çok sayıda C++ kaynak (`.cpp`) ve başlık (`.h`) dosyasını içerir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   `StdAfx.cpp` dosyası `Debug|Win32`, `Release|Win32`, `Distribute|Win32` ve tüm ARM yapılandırmalarında PCH oluşturmak (`Create`) üzere ayarlanmıştır.
*   `Debug|Win32` ve `Release|Win32` yapılandırmalarında, genel PCH ayarı `NotUsing` iken, `StdAfx.cpp` PCH oluşturur.
*   `Distribute|Win32` ve tüm ARM yapılandırmalarında genel PCH ayarı `Use` şeklindedir.
*   x64 platformları için PCH ayarları `ItemDefinitionGroup` eksikliği nedeniyle belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterLib`, statik bir kütüphane (`.lib`) olarak derlenir ve neredeyse tüm diğer istemci kütüphaneleri ve ana uygulama için bir temel oluşturur.
*   **Proje Referansı**: `EterLib.vcxproj` dosyası, `EterLocale.vcxproj` projesine bir referans (`<ProjectReference>`) içerir. Bu, `EterLib`'in `EterLocale` kütüphanesine bağımlı olduğu ve derleme sırasında bu bağımlılığın göz önünde bulundurulacağı anlamına gelir. Ancak, `<LinkLibraryDependencies>` ve `<UseLibraryDependencyInputs>` ayarlarının `false` olması, bu bağımlılığın nasıl yönetildiği konusunda daha detaylı bir inceleme gerektirebilir (genellikle bu ayarlar `true` olur).
*   Proje, `../../External/include` altındaki harici başlık dosyalarına (örneğin, `DirectX SDK`, `JPEG kütüphanesi`, `Python` vb.) bağımlıdır.
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\Développement\...`) içermektedir.
*   **x64 Yapılandırma Eksiklikleri**: Önceki kütüphanelerde olduğu gibi, `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması önemli bir eksikliktir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması platforma özgü kodlarda sorun yaratabilir.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneği `false` olarak ayarlanmıştır. Bu, bazı diğer kütüphanelerin Release yapılandırmalarından farklıdır ve performans üzerinde etkisi olabilir.

---

### `EterLocale.vcxproj` (Yerelleştirme ve Karakter Kodlama Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisindeki metinlerin farklı dillere ve karakter setlerine uyumlu bir şekilde işlenmesi, dönüştürülmesi ve görüntülenmesi için gerekli olan `EterLocale` statik kütüphanesini (`.lib`) derlemek amacıyla kullanılır. Bu kütüphane, çeşitli kod sayfaları (code page) arasında dönüşüm yapma, özel karakter setlerini (Arapça, Japonca, Vietnamca gibi) destekleme ve genel string işleme yardımcı programları sunar. `EterLib` başta olmak üzere metinlerle çalışan birçok modül için önemli bir bağımlılıktır.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{08E76C01-D25C-4684-911C-876A33F27CE1}`
*   **Kök Ad Alanı (RootNamespace)**: `EterLocale`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterLocale.lib üretir)

**`EterLocale.vcxproj.user` Dosyası:**

Bu dosya boştur ve kullanıcıya özel ayar içermez.

**`EterLocale.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını çeşitli filtreler altında düzenler:
*   **Core**: Temel yerelleştirme ayarları ve ön derlenmiş başlıklar (`StdAfx.cpp/h`, `CodePageId.h`).
*   **StringCodec**: Karakter kodlaması ve çözme işlemleri (`StringCodec.cpp/h`, `StringCodec_Vietnamese.cpp/h`).
*   **StringUtility**: Dile özgü string yardımcı fonksiyonları (`Arabic.cpp/h`, `Japanese.cpp/h`).

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/EterLocaleD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterLocale/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterLocale/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `Use` (`StdAfx.h` kullanılır), `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`EterLocale.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **bulunmamaktadır**. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel Visual Studio ayarlarından miras alınmaktadır. Bu platformlar için derleme yapılıyorsa, `_WIN64` ön işlemci tanımı, doğru çalışma zamanı kütüphanesi ve PCH ayarları gibi kritik konfigürasyonların kontrol edilmesi ve gerekirse tanımlanması önemlidir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir. ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır.

**Kaynak Dosyalar:**

Proje, `../../Source/EterLocale/` dizini altında yerelleştirme ve karakter kodlama ile ilgili C++ kaynak (`.cpp`) ve başlık (`.h`) dosyalarını içerir:
*   `Arabic.cpp/h`
*   `CodePageId.h`
*   `Japanese.cpp/h`
*   `StdAfx.cpp/h`
*   `StringCodec.cpp/h`
*   `StringCodec_Vietnamese.cpp/h`

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   `StdAfx.cpp` dosyası `Debug|Win32`, `Release|Win32`, `Distribute|Win32` ve tüm ARM yapılandırmalarında PCH oluşturmak (`Create`) üzere ayarlanmıştır.
*   `Debug|Win32` ve `Release|Win32` yapılandırmalarında, genel PCH ayarı `NotUsing` iken, `StdAfx.cpp` PCH oluşturur.
*   `Distribute|Win32` ve tüm ARM yapılandırmalarında genel PCH ayarı `Use` şeklindedir.
*   x64 platformları için PCH ayarları `ItemDefinitionGroup` eksikliği nedeniyle belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterLocale`, statik bir kütüphane (`.lib`) olarak derlenir.
*   `EterLib` projesi, `EterLocale.vcxproj` projesine doğrudan bir referans içerir, bu da `EterLib`'in bu kütüphaneye bağımlı olduğunu gösterir.
*   Proje, `../../External/include` altındaki harici başlık dosyalarına bağımlı olabilir (özellikle string işleme veya Unicode kütüphaneleri).
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\Développement\...`) içermektedir.
*   **x64 Yapılandırma Eksiklikleri**: Diğer kütüphanelerde olduğu gibi, `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması önemli bir eksikliktir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması platforma özgü kodlarda sorun yaratabilir.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneği `false` olarak ayarlanmıştır.

---

### `EterPack.vcxproj` (Paket Dosya Sistemi Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisinin kullandığı paketlenmiş dosya formatlarını (genellikle `.epk` ve `.eix` uzantılı dosyalar) yönetmek, okumak ve bu dosyalara erişmek için kullanılan `EterPack` statik kütüphanesini (`.lib`) derlemek amacıyla kullanılır. Bu kütüphane, oyun varlıklarının (modeller, dokular, sesler vb.) verimli bir şekilde saklanması ve yüklenmesinde merkezi bir rol oynar. Ayrıca şifreleme ve dosya bütünlüğü için mekanizmalar içerebilir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{EEAB8CB2-0891-4579-905E-B37A5F04F1D1}`
*   **Kök Ad Alanı (RootNamespace)**: `EterPack`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterPack.lib üretir)

**`EterPack.vcxproj.user` Dosyası:**

Sağlanan içeriğe göre bu dosya boştur ve kullanıcıya özel proje ayarı içermez (`<PropertyGroup />`).

**`EterPack.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını "Source Files" ve "Header Files" filtreleri altında düzenler. Kütüphaneyi oluşturan temel bileşenler şunlardır:
*   `EterPack.cpp/h`: Paket dosyalarının temel yönetimi.
*   `EterPackCursor.cpp/h`: Paket içindeki dosyalara erişim için imleç benzeri yapılar.
*   `EterPackManager.cpp/h`: Paketlerin yüklenmesi, kaydedilmesi ve genel yönetimi.
*   `EterPackPolicy_CSHybridCrypt.cpp/h`: Belirli bir hibrit şifreleme politikası implementasyonu.
*   `md5.c/h`: MD5 karma algoritması (dosya bütünlüğü veya diğer amaçlar için kullanılabilir).
*   `StdAfx.cpp/h`: Ön derlenmiş başlıklar (bu projede aktif olarak kullanılmıyor gibi görünüyor).
*   `FoxFS.h`: Potansiyel bir dosya sistemi soyutlaması veya arayüzü.
*   `Inline.h`: Satır içi (inline) fonksiyonlar için yardımcı başlık.
*   `obfuscate.h`: Kod gizleme teknikleri için başlık.

Tüm kaynak dosyalar `../../Source/EterPack/` dizininden referans alınmıştır.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/EterPackD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Taşınabilirlik için bu yolların göreceli olması tercih edilir.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (Hem genel ayar hem de `StdAfx.cpp` için).
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterPack/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (Hem genel ayar hem de `StdAfx.cpp` için).
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EterPack/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (Hem genel ayar hem de `StdAfx.cpp` için).
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`EterPack.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **bulunmamaktadır**. Bu platformlar için yalnızca genel `PlatformToolset` (`v143`) tanımlanmıştır. Bu, `x64` derlemeleri için ayarların Visual Studio varsayılanlarından, paylaşılan `.props` dosyalarından veya diğer genel ayarlardan miras alınacağı anlamına gelir. Bu durum, `_WIN64` ön işlemci tanımının eksikliği, yanlış çalışma zamanı kütüphanesi seçimi, PCH ayarlarının olmaması veya yanlış olması gibi potansiyel derleme sorunlarına veya beklenmedik davranışlara yol açabilir. Bu platformlar için derleme yapılacaksa, bu ayarların dikkatlice incelenmesi ve gerekirse proje dosyasına eklenmesi kritik öneme sahiptir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel `ItemDefinitionGroup` ayarları genellikle `Win32` yapılandırmalarına benzerdir:
*   Ön derlenmiş başlıklar (`StdAfx.cpp` için `NotUsing` ve genel `NotUsing`) ARM için de geçerlidir, yani PCH kullanılmaz.
*   ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır. Bu, platforma özgü kodlarda sorunlara yol açabilir; `_ARM_` gibi daha spesifik makrolar tercih edilmelidir.
*   Diğer ayarlar (optimizasyon, çalışma zamanı kütüphanesi vb.) `Win32` karşılıklarını yansıtır.

**Kaynak Dosyalar:**

Proje, `../../Source/EterPack/` dizini altında yukarıda `EterPack.vcxproj.filters` bölümünde listelenen C (`md5.c`) ve C++ kaynak (`.cpp`) ile başlık (`.h`) dosyalarını içerir. `md5.c` dosyası, C++ derleyicisi tarafından derlenecektir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

`EterPack.vcxproj` dosyasındaki tüm `Win32` ve `ARM` yapılandırmaları (`Debug`, `Release`, `Distribute`) için hem genel `ItemDefinitionGroup` içinde hem de `StdAfx.cpp` dosyasına özel olarak `<PrecompiledHeader>` ayarı `NotUsing` olarak belirtilmiştir. Bu, bu projede **ön derlenmiş başlıkların aktif olarak kullanılmadığı** anlamına gelir. Bu, diğer birçok Eter kütüphanesinden farklı bir yaklaşımdır. x64 yapılandırmaları için PCH durumu, `ItemDefinitionGroup` eksikliği nedeniyle belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterPack`, statik bir kütüphane (`.lib`) olarak derlenir ve istemcinin paketlenmiş dosyalara erişen tüm bölümleri için temel bir bağımlılıktır.
*   Proje, `../../External/include` altındaki harici başlık dosyalarına (örneğin, sıkıştırma kütüphaneleri, şifreleme kütüphaneleri için başlıklar) bağımlı olabilir.
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\Développement\...`) içermektedir.
*   **Kritik x64 Yapılandırma Eksiklikleri**: `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması, bu platformlarda kararlı ve doğru derlemeler için ciddi bir risk oluşturur. Ayarların varsayılana bırakılması, 32-bit ve 64-bit derlemeler arasında tutarsızlıklara ve hatalara yol açabilir.
*   **PCH Kullanılmıyor**: Ön derlenmiş başlıkların kullanılmaması, özellikle büyük projelerde derleme sürelerini artırabilir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması, platforma özgü koşullu derleme bloklarında hatalara neden olabilir.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneği `false` olarak ayarlanmıştır.

---

---

### `EterPythonLib.vcxproj` (Python Entegrasyon Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisinin C++ tabanlı sistemleri ile Python betikleme dili arasında bir köprü görevi gören `EterPythonLib` statik kütüphanesini (`.lib`) derlemek için kullanılır. Bu kütüphane, özellikle kullanıcı arayüzü (UI) elemanlarının ve bazı grafik fonksiyonlarının Python üzerinden kontrol edilmesini ve yönetilmesini sağlar. Oyun içi arayüzlerin büyük bir kısmı Python ile yazıldığı için bu kütüphane kritik bir öneme sahiptir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{396764A4-9226-4F33-930F-E577A3BF51D4}`
*   **Kök Ad Alanı (RootNamespace)**: `EterPythonLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterPythonLib.lib üretir)

**`EterPythonLib.vcxproj.user` Dosyası:**

Bu dosya genellikle kullanıcıya özel ayarları içerir, ancak sağlanan bilgilere göre boştur (`<PropertyGroup />`).

**`EterPythonLib.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını Python entegrasyonunun mantıksal bölümlerine göre filtreler altında düzenler. Başlıca filtreler şunlardır:
*   **Interface**: Python'a açılan temel arayüz sınıfları ve yöneticileri.
    *   `PythonGridSlotWindow.cpp/h`: Oyun içi envanter veya yetenek çubukları gibi ızgara tabanlı slotlu pencerelerin Python entegrasyonu.
    *   `PythonSlotWindow.cpp/h`: Daha genel slotlu pencerelerin Python entegrasyonu.
    *   `PythonWindow.cpp/h`: Temel pencere davranışlarının (boyut, pozisyon, görünürlük vb.) Python'dan yönetilmesi için altyapı.
    *   `PythonWindowManager.cpp/h`: İstemcideki tüm Python tabanlı pencerelerin yönetimi, oluşturulması ve yok edilmesinden sorumlu C++ sınıfı.
    *   `PythonWindowManagerModule.cpp`: `PythonWindowManager` sınıfının fonksiyonlarını Python betiklerine doğrudan sunan modül.
*   **Graphic Files**: Grafiksel işlemlerin Python'dan çağrılabilmesi için oluşturulmuş modüller.
    *   `PythonGraphic.cpp/h`: Temel grafik ayarları veya çizim fonksiyonları için Python arayüzü.
    *   `PythonGraphicImageModule.cpp`: Resim (image) nesnelerinin yüklenmesi, yönetilmesi ve çizdirilmesi gibi işlemlerin Python'a açıldığı modül.
    *   `PythonGraphicModule.cpp`: Genel grafik işlemlerini (renk ayarları, çizgi çizme vb.) Python'a sunan modül.
    *   `PythonGraphicTextModule.cpp`: Oyun içi metinlerin oluşturulması, font ayarları ve ekrana çizdirilmesi gibi işlemlerin Python'a açıldığı modül.
    *   `PythonGraphicThingModule.cpp`: Oyun dünyasındaki "şeylerin" (muhtemelen efektler, basit 3D objeler veya arayüz elemanları) grafiksel temsillerinin Python'dan yönetilmesini sağlayan modül.
*   `StdAfx.cpp/h`: Ön derlenmiş başlıklar için standart dosyalar.

Tüm kaynak dosyalar `../../Source/EterPythonLib/` dizininden referans alınmıştır.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\\..\\..\\Client\\` (Genellikle `srcClient/Client/EterPythonLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`. Bu dizin, Python'un başlık dosyalarını (örneğin `Python.h`) içermelidir. Genellikle bu başlıklar `External/include/pythonX.Y` gibi bir alt dizinde bulunur.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\include` ve `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Bu, projenin taşınabilirliğini olumsuz etkiler.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create` olarak ayarlanmış, ancak genel proje ayarı `NotUsing`. Bu durum, PCH'nin etkin bir şekilde kullanılmadığı anlamına gelebilir; `StdAfx.cpp` PCH'yi oluşturur ama diğer kaynak dosyalar bunu kullanmaz.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterPythonLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, genel ayar `NotUsing`. PCH etkin kullanılmıyor olabilir.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false` (Diğer bazı Eter kütüphanelerinde `true` olabiliyordu).
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için son kullanıcı sürümü.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterPythonLib/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `Use` (`StdAfx.h` kullanılır), `StdAfx.cpp` için `Create`. Bu yapılandırmada PCH doğru şekilde kullanılıyor.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`EterPythonLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **tamamen eksiktir**. Bu platformlar için yalnızca genel `PlatformToolset` (`v143`) tanımlanmıştır. Bu, `x64` derlemeleri için hayati önem taşıyan ayarların (örneğin `_WIN64` ön işlemci tanımı, Python'un 64-bit kütüphane ve başlık dosyalarının yolları, doğru çalışma zamanı kütüphanesi, PCH ayarları vb.) eksik veya yanlış olacağı anlamına gelir. Bu platformlarda başarılı bir derleme için bu ayarların proje dosyasına manuel olarak eklenmesi ve yapılandırılması zorunludur.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel `ItemDefinitionGroup` ayarları genellikle `Win32 Distribute` yapılandırmasına benzer şekilde PCH kullanımını benimser (genel ayar `Use`, `StdAfx.cpp` için `Create`).
*   Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM yapılandırmalarında aktif ve doğru şekilde kullanılıyor gibi görünmektedir.
*   Ancak, ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır. Bu, platforma özgü kod bloklarında sorunlara yol açabilir; ideal olarak `_ARM_` gibi daha spesifik bir makro kullanılmalıdır.
*   Diğer derleyici ve bağlayıcı ayarları genellikle `Win32` karşılıklarını yansıtır.

**Kaynak Dosyalar:**

Proje, `../../Source/EterPythonLib/` dizini altında, yukarıda `EterPythonLib.vcxproj.filters` bölümünde detaylandırılan ve C++ ile Python arasında köprü kuran çok sayıda `.cpp` ve `.h` dosyasını içerir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   PCH kullanımı yapılandırmalara göre farklılık göstermektedir:
    *   **`Debug|Win32` ve `Release|Win32`**: Bu yapılandırmalarda `StdAfx.cpp` dosyası `<PrecompiledHeader>Create</PrecompiledHeader>` ayarına sahipken, projenin genel PCH ayarı `<PrecompiledHeader>NotUsing</PrecompiledHeader>` şeklindedir. Bu tutarsızlık, PCH dosyasının oluşturulmasına rağmen diğer kaynak dosyalar tarafından kullanılmadığı anlamına gelir, dolayısıyla PCH bu yapılandırmalarda etkili bir şekilde devre dışı kalmış olur.
    *   **`Distribute|Win32` ve tüm `ARM` yapılandırmaları**: Bu yapılandırmalarda genel PCH ayarı `<PrecompiledHeader>Use</PrecompiledHeader>` (`StdAfx.h` dosyasını kullanır) ve `StdAfx.cpp` için `<PrecompiledHeader>Create</PrecompiledHeader>` ayarı mevcuttur. Bu, PCH'nin bu yapılandırmalarda beklendiği gibi doğru bir şekilde kullanıldığını gösterir.
*   **x64 platformları**: `ItemDefinitionGroup` bloklarının eksikliği nedeniyle PCH ayarları belirsizdir ve muhtemelen PCH kullanılmayacaktır veya hatalı yapılandırılacaktır.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterPythonLib`, statik bir kütüphane (`.lib`) olarak derlenir.
*   Bu kütüphane, temel işlevi gereği **Python C API'sine güçlü bir şekilde bağımlıdır.** Proje ayarlarında `../../External/include` genel bir dahil etme dizini olarak belirtilmiştir. Python başlık dosyalarının (`Python.h` ve ilişkili dosyalar) ve derlenmiş Python kütüphanelerinin (`.lib` dosyaları) bu `External` dizini altında doğru bir şekilde yapılandırılmış olması (örneğin, `External/include/pythonX.Y`, `External/lib/pythonX.Y`) gerekmektedir.
*   Kütüphane, Python'a açtığı fonksiyonlar aracılığıyla `EterLib` (grafik, pencereleme temelleri için) ve dolaylı olarak `UserInterface` (UI mantığı için) gibi diğer istemci kütüphaneleriyle sıkı bir etkileşim içindedir.
*   `Debug|Win32` yapılandırmasındaki mutlak yollar (`D:\\Développement\\...`) projenin farklı geliştirme ortamlarına taşınabilirliğini kısıtlar.
*   **Kritik x64 Yapılandırma Eksiklikleri**: `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması, bu platformda derleme yapılmasını fiilen imkansız hale getirir veya çok sorunlu kılar. Özellikle Python'un 64-bit sürümüne ait doğru kütüphane ve başlık yollarının belirtilmesi şarttır.
*   **PCH Kullanımında Tutarsızlık**: `Debug|Win32` ve `Release|Win32` yapılandırmalarındaki PCH ayarları, PCH kullanımını etkisiz kılacak şekilde yapılandırılmıştır. Bu, derleme sürelerini olumsuz etkileyebilir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması, platforma özgü kodların yanlış derlenmesine veya hatalı çalışmasına neden olabilir.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneği (`/LTCG`) `false` olarak ayarlanmıştır. Bu, bazı diğer Eter kütüphanelerinin Release yapılandırmalarındaki `true` ayarından farklıdır.

---