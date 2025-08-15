# VSFiles Klasörü Referans Kılavuzu

Bu belge, Metin2 istemci kaynak kodu (`@srcClient`) içindeki `VSFiles` klasörünün yapısını ve amacını açıklamaktadır.

## Genel Amaç

`VSFiles` klasörü, Metin2 istemcisini oluşturan çeşitli kütüphanelerin ve modüllerin Microsoft Visual Studio için proje ve çözüm dosyalarını içerir. Bu dosyalar, projenin Visual Studio ortamında açılması, derlenmesi, hata ayıklanması ve yönetilmesi için gereklidir.

Her bir alt klasör, genellikle belirli bir kütüphane veya modülün proje dosyalarını barındırır. Bu proje dosyaları (`.vcxproj` veya `.vcproj` uzantılı) ve çözüm dosyaları (`.sln` uzantılı), kaynak kod dosyalarını, derleyici ayarlarını, kütüphane bağımlılıklarını ve diğer derleme süreçlerini tanımlar.

## Klasör Yapısı ve İçerikleri

`VSFiles` klasörü aşağıdaki alt klasörleri içermektedir:

*   **`UserInterface/`**: Kullanıcı arayüzü (UI) ile ilgili modülün Visual Studio proje dosyalarını içerir. Bu, oyun içi pencereler, butonlar, metinler ve diğer UI elemanlarının C++ ve Python entegrasyonuyla ilgili olabilir.
*   **`TerrainLib/`**: Arazi (terrain) oluşturma ve işleme kütüphanesinin proje dosyalarını içerir. Oyun dünyasındaki haritaların ve zeminlerin yönetimiyle ilgilidir.
*   **`SphereLib/`**: Küresel veya hacimsel hesaplamalarla ilgili olabilecek bir kütüphanenin proje dosyalarını barındırır. (Detaylı işlevi proje dosyaları incelenerek anlaşılabilir.)
*   **`SpeedTreeLib/`**: SpeedTree kütüphanesi entegrasyonu için proje dosyalarını içerir. Bu kütüphane, oyun dünyasındaki ağaçların ve bitki örtüsünün gerçekçi bir şekilde oluşturulması ve render edilmesi için kullanılır.
*   **`ScriptLib/`**: Genel betik (script) işleme veya yönetimiyle ilgili bir kütüphanenin proje dosyalarını içerir.
*   **`MilesLib/`**: Miles Ses Sistemi (Miles Sound System) kütüphanesi entegrasyonu için proje dosyalarını içerir. Bu, oyun içi ses efektleri ve müziklerin yönetimi için kullanılır.
*   **`GameLib/`**: Temel oyun mekanikleri, karakter yönetimi, eşya yönetimi gibi çekirdek oyun işlevlerini içeren kütüphanenin proje dosyalarını barındırır.
*   **`EterPythonLib/`**: Python betik dilinin C++ ile entegrasyonunu sağlayan `EterPython` kütüphanesinin proje dosyalarını içerir. Python modüllerinin C++ fonksiyonlarına erişimi bu kütüphane üzerinden sağlanır.
*   **`EterPack/`**: Oyun varlıklarını içeren paketlenmiş dosyaların (`.epk`, `.eix`) yönetimi için kullanılan `EterPack` kütüphanesinin proje dosyalarını içerir.
*   **`EterLocale/`**: Yerelleştirme (dil, bölge ayarları) ile ilgili `EterLocale` kütüphanesinin proje dosyalarını içerir.
*   **`EterLib/`**: Çeşitli yardımcı fonksiyonları, veri yapılarını ve temel sistemleri içeren genel amaçlı `Eter` kütüphanesinin proje dosyalarını barındırır.
*   **`EterImageLib/`**: Resim dosyalarını yükleme, işleme ve yönetme (örneğin `.dds`, `.tga` formatları) işlevlerini sağlayan `EterImage` kütüphanesinin proje dosyalarını içerir.
*   **`EterGrnLib/`**: Granny 3D (`.gr2`) animasyon ve model formatını işleyen `EterGrn` kütüphanesinin proje dosyalarını içerir. Karakter, canavar ve diğer 3D modellerin animasyonları ve geometrileri bu kütüphane ile yönetilir.
*   **`EterBase/`**: `Eter` kütüphane ailesinin temelini oluşturan veya diğer temel işlevleri içeren bir kütüphanenin proje dosyalarını barındırır.
*   **`EffectLib/`**: Oyun içi görsel efektlerin ( büyü efektleri, patlamalar vb.) yönetimi ve render edilmesiyle ilgili kütüphanenin proje dosyalarını içerir.
*   **`CWebBrowser/`**: Muhtemelen Chromium Embedded Framework (CEF) veya benzeri bir gömülü web tarayıcısı entegrasyonu için proje dosyalarını içerir. Bu, oyun içinde web tabanlı içeriklerin (örneğin, item shop, duyurular) gösterilmesi için kullanılabilir.

**Not:** Bu açıklamalar, klasör isimlerinden ve Metin2 istemci mimarisi hakkındaki genel bilgilerden yola çıkılarak yapılmıştır. Her bir alt klasörün kesin içeriği ve barındırdığı proje dosyalarının detayları, ilgili `.sln` ve `.vcxproj`/`.vcproj` dosyaları incelenerek daha ayrıntılı bir şekilde belgelenebilir.

---

## Çözüm Dosyası (`M2Client.sln`)

`VSFiles` klasörünün bir üst dizininde (genellikle `@srcClient` kök dizininde) bulunan `M2Client.sln` dosyası, tüm Metin2 istemci projesini bir araya getiren ana Visual Studio Çözüm Dosyasıdır. Bu dosya, Visual Studio'da projeyi açmak, yönetmek ve derlemek için kullanılır.

### Temel Özellikleri:

*   **Proje Listesi**: Çözüm, `VSFiles` klasörü altında bulunan tüm bireysel kütüphane ve modül projelerini (`.vcxproj` dosyaları) listeler. Bunlar arasında `UserInterface`, `GameLib`, `EterLib`, `EffectLib` gibi ana bileşenler bulunur. `M2Client.sln` dosyasında tanımlanan projeler şunlardır: `CWebBrowser`, `EffectLib`, `EterBase`, `EterGrnLib`, `EterImageLib`, `EterLib`, `EterLocale`, `EterPack`, `EterPythonLib`, `GameLib`, `MilesLib`, `PRTerrainLib` (TerrainLib olarak da bilinir), `ScriptLib`, `SpeedTreeLib`, `SphereLib`, ve `UserInterface`.
*   **Yapılandırma Yönetimi**: `M2Client.sln` dosyası, projenin farklı senaryolar için nasıl derleneceğini tanımlayan çeşitli yapılandırmaları ve platformları yönetir:
    *   **Yapılandırmalar (Configurations):**
        *   `Debug`: Geliştirme ve hata ayıklama için kullanılır. Optimizasyonlar genellikle kapalıdır ve hata ayıklama sembolleri eklenir.
        *   `Release`: Son kullanıcıya dağıtılacak optimize edilmiş sürüm için kullanılır.
        *   `Distribute`: Genellikle `Release` yapılandırmasına benzer, ancak bazen farklı dağıtım ayarları veya ön işlemci tanımları içerebilir.
    *   **Platformlar (Platforms):**
        *   `Win32`: 32-bit Windows platformu için derleme yapar.
        *   `x64`: 64-bit Windows platformu için derleme yapar.
*   **Proje Bağımlılıkları**: Çözüm dosyası, projeler arasındaki derleme bağımlılıklarını (bir projenin derlenmeden önce hangi diğer projelerin derlenmesi gerektiğini) dolaylı olarak yönetir. Visual Studio, bu bağımlılıkları projelerin birbirlerine referans verme şekline göre belirler.
*   **Visual Studio Entegrasyonu**: Bu dosya, projenin Visual Studio 17 (Visual Studio 2022) ile uyumlu olduğunu ve en az Visual Studio 2010 gerektirdiğini belirtir.

### Derleme Sürecinde Dikkat Edilmesi Gerekenler:

1.  **Doğru Çözüm Dosyasını Açmak**: Metin2 istemcisini derlemek veya üzerinde çalışmak için her zaman `M2Client.sln` dosyası Visual Studio ile açılmalıdır.
2.  **Yapılandırma ve Platform Seçimi**: Derleme yapmadan önce Visual Studio araç çubuğundan uygun "Solution Configuration" (örneğin, `Release`) ve "Solution Platform" (örneğin, `Win32`) seçilmelidir.
3.  **Proje Bağımlılıklarının Çözülmesi**: Visual Studio, projeler arasındaki bağımlılıklara göre derleme sırasını otomatik olarak belirleyecektir. Bir proje, bağımlı olduğu başka bir proje henüz derlenmediyse hata verebilir. "Build Solution" (Çözümü Derle) seçeneği genellikle tüm bağımlılıkları doğru sırada derler.
4.  **Platform Uyumluluğu**: `M2Client.sln` dosyasındaki bazı `x64` çözüm yapılandırmalarının, bazı projeler için `Win32` proje yapılandırmalarını işaret ettiğine dikkat edilmelidir. Tam bir `x64` derleme hedefleniyorsa, tüm ilgili projelerin `x64` için doğru şekilde yapılandırıldığından emin olunması gerekebilir. Bu, proje dosyalarının (`.vcxproj`) detaylı incelenmesini gerektirebilir.

Bu `.sln` dosyası, geliştiricilerin tüm istemci kod tabanını tutarlı bir şekilde derleyip yönetebilmeleri için merkezi bir rol oynar. 

---

### `CWebBrowser.vcxproj` (Gömülü Web Tarayıcı Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisi içinde web tabanlı içeriklerin gösterilmesi için kullanıldığı düşünülen `CWebBrowser` statik kütüphanesini (`.lib`) derlemek için kullanılır. Kaynak dosyasının `.c` uzantılı olması (`CWebBrowser.c`) ve C dil standardı ayarları, bu kütüphanenin C ile yazılmış olabileceğini veya C derleyicisi ile derlenen C++ kodu olabileceğini düşündürmektedir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{2E953487-E73A-4C43-A9B6-174AB7B9A7E2}`
*   **Hedeflenen Windows SDK**: `10.0` (Dosyada `WindowsTargetPlatformVersion` olarak belirtilmiş)
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (CWebBrowser.lib üretir)

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` ve `x64` platformları için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/CWebBrowserD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MDd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB` (ve diğer varsayılanlar).
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: Bu blokta doğrudan boş, ancak `PropertyGroup` içinde mutlak bir `IncludePath` (`D:\Développement\METIN2 GF Like\srcClient\External\include`) tanımlanmış. Bu, dış bağımlılıkların başlık dosyaları için kullanılır.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **C Dil Standardı (`LanguageStandard_C`)**: `Default`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/CWebBrowser/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MD)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `PrecompiledHeader`: `NotUsing`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/CWebBrowser/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MD)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `PrecompiledHeader`: Tanımlı değil (muhtemelen `NotUsing`).

**4. Debug | x64 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama (64-bit).
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/CWebBrowser/Debug/`)
*   **Ara Dizin (`IntDir`)**: `$(Configuration)\`
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: Belirtilmemiş (muhtemelen `/MDd` varsayılanı veya `.props`'tan miras).
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`. **Dikkat:** `x64` platformu için `_WIN64` yerine `WIN32` tanımlanmış olması, platforma özgü kodlarda sorun yaratabilir.
*   **Temel Çalışma Zamanı Kontrolleri (`BasicRuntimeChecks`)**: `EnableFastChecks` (/RTC1)
*   **Ek Dahil Etme Dizinleri**: Belirtilmemiş. Eğer harici kütüphane başlıkları gerekiyorsa, bunların nasıl çözüldüğü incelenmelidir.

**Release|x64 ve Distribute|x64 Yapılandırmaları:**

Bu yapılandırmalar için `ItemDefinitionGroup` blokları dosyada eksik görünüyor. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel ayarlardan miras alınmaktadır. Tam bir `x64` derlemesi için bu yapılandırmaların detaylı incelenmesi ve gerekirse eksik ayarların tanımlanması önemlidir.

**ARM Platformları:**

Proje, ARM platformları için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar, genellikle `Win32` yapılandırmalarına benzer prensipleri izler.

**Kaynak Dosyalar:**

*   `..\..\Source\CWebBrowser\CWebBrowser.c`
*   `..\..\Source\CWebBrowser\CWebBrowser.h`

**Bağımlılıklar ve Önemli Notlar:**

*   Bu proje, `StaticLibrary` olarak derlenir ve çıktı olarak `CWebBrowser.lib` (veya yapılandırmaya göre `CWebBrowserD.lib`) üretir.
*   `Debug|Win32` yapılandırmasındaki `IncludePath` içinde mutlak bir yol (`D:\Développement\...`) bulunmaktadır. Bu, projenin farklı geliştirici ortamlarında derlenmesini zorlaştırabilir ve göreceli yollar veya ortam değişkenleri ile değiştirilmesi önerilir. Diğer yapılandırmalarda (Release, Distribute, x64) harici kütüphane başlık yollarının nasıl tanımlandığı açıkça belirtilmemiştir; bu durum, derleme sırasında bu yapılandırmaların harici bağımlılıkları bulmada sorun yaşamasına neden olabilir.
*   `x64` yapılandırmalarında `_WIN64` yerine `WIN32` ön işlemci tanımının kullanılması, 64-bit'e özgü kod bloklarının doğru derlenmemesine yol açabilir.
*   Projenin Chromium Embedded Framework (CEF) gibi bir dış web tarayıcı kütüphanesine dayanması muhtemeldir. Bu durumda, ilgili CEF kütüphane dosyalarının (`.lib`, `.dll`, kaynak dosyaları) projeye doğru şekilde dahil edilmesi ve bağlanması kritik öneme sahiptir. Mevcut `.vcxproj` dosyasında bu harici kütüphanelere olan bağlantılar (linker dependencies) açıkça belirtilmemiştir. Bu, ya başka `.props` dosyalarında yönetiliyor ya da `CWebBrowser.c` içinde dinamik yükleme (LoadLibrary) gibi yöntemler kullanılıyor olabilir.

--- 

### `EffectLib.vcxproj` (Oyun Efektleri Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisindeki görsel efektlerin (örneğin büyü efektleri, patlamalar, parçacık sistemleri) yönetimi ve işlenmesiyle ilgili `EffectLib` statik kütüphanesini (`.lib`) derlemek için kullanılır.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{7F1EC9EC-35DA-4332-A339-B68E3C95976F}`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EffectLib.lib üretir)

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` ve `x64` platformları için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client\` (Genellikle `srcClient/Client/EffectLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MDd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`, `WINDOWS_IGNORE_PACKING_MISMATCH`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\...` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Taşınabilirlik için bu yolların göreceli olması tercih edilir.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (genel ayar), ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EffectLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MD)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (genel ayar), ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EffectLib/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MD)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`, `WINDOWS_IGNORE_PACKING_MISMATCH`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `Use` (`StdAfx.cpp` için `Create` olarak özel ayarlanmış).
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)

**4. Debug | x64 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama (64-bit).
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/EffectLib/Debug/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MDd)
*   **Optimizasyon (`Optimization`)**: Tanımlı değil (muhtemelen `Disabled` varsayılanı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN64`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`, `WINDOWS_IGNORE_PACKING_MISMATCH`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `NotUsing` (genel ayar). `StdAfx.cpp` için bu yapılandırmada özel bir `PrecompiledHeader` ayarı (`Create` veya `Use`) görünmemektedir, bu durum ön derlenmiş başlıkların x64 Debug'da nasıl ele alındığının incelenmesini gerektirebilir.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi).

**Release|x64 ve Distribute|x64 Yapılandırmaları:**

Bu yapılandırmalar için `ItemDefinitionGroup` blokları dosyada tam olarak listelenmemiştir. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel ayarlardan miras alınmaktadır.

**ARM Platformları:**

Proje, ARM platformları için de yapılandırma tanımları içerir ve genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir.

**Kaynak Dosyalar:**

Proje, `../../Source/EffectLib/` dizini altında efekt sistemiyle ilgili çok sayıda C++ kaynak ve başlık dosyası içerir:
*   `EffectData.h/.cpp`
*   `EffectElementBase.h/.cpp`
*   `EffectElementBaseInstance.h/.cpp`
*   `EffectInstance.h/.cpp`
*   `EffectManager.h/.cpp`
*   `EffectMesh.h/.cpp`
*   `EffectMeshInstance.h/.cpp`
*   `EffectUpdateDecorator.h/.cpp`
*   `EmitterProperty.h/.cpp`
*   `FrameController.h/.cpp`
*   `ParticleInstance.h/.cpp`
*   `ParticleProperty.h/.cpp`
*   `ParticleSystemData.h/.cpp`
*   `ParticleSystemInstance.h/.cpp`
*   `SimpleLightData.h/.cpp`
*   `SimpleLightInstance.h/.cpp`
*   `StdAfx.h/.cpp` (Ön derlenmiş başlıklar için kullanılır)
*   `Type.h/.cpp`

**Bağımlılıklar ve Önemli Notlar:**

*   `EffectLib`, statik bir kütüphane (`.lib`) olarak derlenir.
*   Diğer birçok kütüphane gibi, `GameLib` ve `UserInterface` gibi ana modüller tarafından kullanılır.
*   Proje, `../../External/include` altındaki harici başlık dosyalarına bağımlıdır.
*   `WINDOWS_IGNORE_PACKING_MISMATCH` ön işlemci tanımının kullanılması, harici kütüphanelerle veya farklı derleyici ayarlarıyla oluşturulmuş bileşenlerle olası yapı paketleme (structure packing) uyumsuzluklarını gidermeyi amaçlar.
*   Ön derlenmiş başlıklar (`StdAfx.h`) çoğu yapılandırmada aktif olarak kullanılmaktadır. Ancak, `Debug|x64` için `StdAfx.cpp`'nin `PrecompiledHeader` ayarının eksik olması dikkat çekicidir ve bu yapılandırmada derleme süresini etkileyebilir veya PCH kullanımında tutarsızlığa yol açabilir.

--- 

### `EterBase.vcxproj` (Temel Eter Kütüphanesi Projesi)

Bu proje dosyası, Eter kütüphane ailesinin temelini oluşturan ve çeşitli alt seviye yardımcı programları, veri yapılarını ve fonksiyonları içeren `EterBase` statik kütüphanesini (`.lib`) derlemek için kullanılır. Bu kütüphane, istemcinin diğer birçok modülü için temel işlevsellik sağlar.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{D8C71073-CDC3-4AB1-A84A-5829F28BFF56}`
*   **Kök Ad Alanı (RootNamespace)**: `EterBase`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterBase.lib üretir)

**`EterBase.vcxproj.user` Dosyası:**

Bu dosya boştur ve kullanıcıya özel ayar içermez.

**`EterBase.vcxproj.filters` Dosyası:**

Bu dosya, proje içindeki kaynak ve başlık dosyalarını sanal klasörler (filtreler) altında düzenler. Başlıca filtreler şunlardır:
*   **Code**: Kütüphanenin temel kaynak dosyalarını içerir (`cipher.cpp`, `FileBase.cpp`, `Debug.cpp`, `StdAfx.cpp` vb.).
*   **Poly**: Polinomlarla ilgili olabilecek bir alt modülün kaynak dosyalarını içerir (`Poly/Base.cpp`, `Poly/Poly.cpp` vb.).

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` ve `x64` platformları için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\\..\\..\\Client\\` (Genellikle `srcClient/Client/EterBaseD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include;../../source/EterBase/Poly`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\include` ve `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Taşınabilirlik için bu yolların göreceli olması tercih edilir.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Çoğu dosya için `NotUsing`.
    *   `StdAfx.cpp`: `PrecompiledHeader` ayarı `Create` olarak belirtilmiştir.
    *   `Poly/` altındaki `.cpp` dosyaları (`Base.cpp`, `Poly.cpp`, `Symbol.cpp`, `SymTable.cpp`): `PrecompiledHeader` ayarı `NotUsing` olarak belirtilmiştir.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false` (Genellikle Debug için `true` olur, bu ayar derleme sürelerini etkileyebilir).
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterBase/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Çoğu dosya için `NotUsing`.
    *   `StdAfx.cpp`: `PrecompiledHeader` ayarı `Create`.
    *   `Poly/*` dosyaları: `NotUsing`.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false` (MaxSpeed optimizasyonunda genellikle `true` olur)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterBase/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel olarak `Use` (`StdAfx.h` kullanılır).
    *   `StdAfx.cpp`: `PrecompiledHeader` ayarı `Create`.
    *   `Poly/*` dosyaları: `NotUsing`.
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**4. Debug | x64 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama (64-bit).
*   **Çıktı Dizini (`OutDir`)**: Belirtilmemiş (muhtemelen `$(ProjectDir)$(Configuration)\`).
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: Belirtilmemiş.
*   **Optimizasyon (`Optimization`)**: Belirtilmemiş.
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `ItemDefinitionGroup` içinde eksik. **Dikkat:** `x64` platformu için `_WIN64` tanımlanmalıdır. Mevcut durumda `WIN32` tanımını miras alabilir, bu da hatalara yol açar.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include;../../source/EterBase/Poly`.
*   **Temel Çalışma Zamanı Kontrolleri (`BasicRuntimeChecks`)**: `EnableFastChecks` (/RTC1).
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için özel bir ayar yok. Bu, PCH kullanımında sorunlara yol açabilir.

**5. Release | x64 ve Distribute | x64 Yapılandırmaları:**

Bu yapılandırmalar için `ItemDefinitionGroup` blokları proje dosyasında (`EterBase.vcxproj`) **bulunmamaktadır**. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel Visual Studio ayarlarından miras alınmaktadır. Tam bir `x64` derlemesi için bu yapılandırmaların detaylı incelenmesi ve gerekli ayarların projeye özel olarak tanımlanması kritik öneme sahiptir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir.

**Kaynak Dosyalar:**

Proje, `../../Source/EterBase/` dizini altında çeşitli C++ kaynak (`.cpp`, `.cc`) ve başlık (`.h`) dosyalarını içerir:
*   **Temel Yardımcı Programlar**: `cipher.cpp/h` (şifreleme), `CPostIt.cpp/h`, `CRC32.cpp/h`, `Debug.cpp/h`, `error.cpp/h`, `FileBase.cpp/h`, `FileDir.cpp/h`, `FileLoader.cpp/h`, `Filename.h`, `grid.cc/h`, `lzo.cpp/h` (sıkıştırma), `MappedFile.cpp/h`, `Random.cpp/h`, `StdAfx.cpp/h` (ön derlenmiş başlık), `Stl.cpp/h` (STL ile ilgili yardımcılar), `tea.cpp/h` (şifreleme algoritması), `TempFile.cpp/h`, `Timer.cpp/h`, `Utils.cpp/h`, `vk.h` (sanal tuş kodları), `obfuscate.h`.
*   **Poly Alt Modülü**: `Poly/Base.cpp/h`, `Poly/Poly.cpp/h`, `Poly/Symbol.cpp/h`, `Poly/SymTable.cpp/h`.
*   **Diğer**: `ServiceDefs.h`, `Singleton.h`.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   `StdAfx.cpp` dosyası `Debug|Win32`, `Release|Win32`, `Distribute|Win32` ve tüm ARM yapılandırmalarında PCH oluşturmak (`Create`) üzere ayarlanmıştır.
*   `Debug|Win32` ve `Release|Win32` yapılandırmalarında, genel PCH ayarı `NotUsing` iken, `StdAfx.cpp` PCH oluşturur. Bu, diğer dosyaların PCH'yi kullanmayacağı anlamına gelir (özel olarak `Use` olarak ayarlanmadıkça). Bu durum biraz alışılmadıktır.
*   `Distribute|Win32` yapılandırmasında genel PCH ayarı `Use` şeklindedir, bu daha standart bir yaklaşımdır.
*   `Poly/` altındaki kaynak dosyalar, PCH kullanmayacak şekilde (`NotUsing`) açıkça ayarlanmıştır.
*   **`Debug|x64` yapılandırmasında `StdAfx.cpp` için PCH ayarı eksiktir.** Bu, x64 Debug derlemelerinde PCH'nin doğru kullanılmamasına veya hiç kullanılmamasına neden olabilir.
*   `Release|x64` ve `Distribute|x64` için `ItemDefinitionGroup` olmadığından PCH davranışları belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterBase`, statik bir kütüphane (`.lib`) olarak derlenir ve istemcinin diğer birçok modülü tarafından kullanılır.
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\\Développement\\...`) içermektedir.
*   **x64 Yapılandırma Eksiklikleri**:
    *   `Debug|x64` için `ItemDefinitionGroup` içinde `_WIN64` ön işlemci tanımı, çalışma zamanı kütüphanesi ve `StdAfx.cpp` için PCH ayarları gibi kritik tanımlamalar eksiktir.
    *   `Release|x64` ve `Distribute|x64` yapılandırmaları için `ItemDefinitionGroup` blokları hiç yoktur. Bu durum, bu platformlarda kararlı ve optimize edilmiş derlemeler almayı zorlaştırır.
*   **Platform Ön İşlemci Tanımları**: `Debug|x64` ve ARM yapılandırmalarında `WIN32` tanımının kullanılması, platforma özgü kodlarda sorunlara yol açabilir. `x64` için `_WIN64`, ARM için `_ARM_` gibi daha spesifik makrolar kullanılmalıdır.
*   **PCH Stratejisi**: Win32 yapılandırmalarında PCH kullanımı tutarsız görünüyor. x64 Debug için PCH yapılandırması eksik.
*   `Debug|Win32` yapılandırmasında `MinimalRebuild` ayarının `false` olması, hata ayıklama sırasında artımlı derleme sürelerini uzatabilir.
*   `Release|Win32` yapılandırmasında `IntrinsicFunctions` ayarının `false` olması, `MaxSpeed` optimizasyon hedefiyle çelişebilir (genellikle `/Oi` - `true` - kullanılır).
*   Proje genelinde C++ dil standardı olarak `stdcpplatest` kullanılması modern bir yaklaşımdır.

--- 

### `EterGrnLib.vcxproj` (Granny 3D Kütüphanesi Entegrasyon Projesi)

Bu proje dosyası, Metin2 istemcisinde 3D modellerin ve animasyonların yönetimi için kullanılan Granny 3D (`.gr2` formatı) kütüphanesinin entegrasyonunu sağlayan `EterGrnLib` statik kütüphanesini (`.lib`) derlemek için kullanılır. Karakterler, canavarlar, binekler ve diğer birçok 3D varlık bu kütüphane aracılığıyla işlenir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{935F7B65-3574-41C4-B4F9-1C2EC950463A}`
*   **Kök Ad Alanı (RootNamespace)**: `EterGrnLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (EterGrnLib.lib üretir)

**`EterGrnLib.vcxproj.user` Dosyası:**

Bu dosya boştur ve kullanıcıya özel ayar içermez.

**`EterGrnLib.vcxproj.filters` Dosyası:**

Bu dosya, proje içindeki kaynak ve başlık dosyalarını "Source Files" ve "Header Files" filtreleri altında düzenler. Kütüphaneyi oluşturan temel bileşenler şunlardır:
*   `LODController.cpp/h`: Detay Seviyesi (Level of Detail) yönetimi.
*   `Material.cpp/h`: Materyal ve doku yönetimi.
*   `Mesh.cpp/h`: Model geometrisi (vertex, polygon) yönetimi.
*   `Model.cpp/h`: Genel 3D model yapısı.
*   `ModelInstance.cpp/h`: Bir modelin oyun dünyasındaki örneği (instance).
*   `ModelInstanceCollisionDetection.cpp`: Model örnekleri için çarpışma tespiti.
*   `ModelInstanceModel.cpp`: Model örneğinin model verileriyle ilişkisi.
*   `ModelInstanceMotion.cpp`: Model örneğinin animasyon yönetimi.
*   `ModelInstanceRender.cpp`: Model örneğinin render edilmesi.
*   `ModelInstanceUpdate.cpp`: Model örneğinin güncellenmesi.
*   `Motion.cpp/h`: Animasyon verileri.
*   `StdAfx.cpp/h`: Ön derlenmiş başlıklar.
*   `Thing.cpp/h`: Genel bir 3D nesne.
*   `ThingInstance.cpp/h`: Bir `Thing` nesnesinin oyun dünyasındaki örneği.
*   `Util.cpp/h`: Yardımcı fonksiyonlar.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\\..\\..\\Client\\` (Genellikle `srcClient/Client/EterGrnLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\include` ve `D:\\Développement\\METIN2 GF Like\\srcClient\\External\\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır. Taşınabilirlik için bu yolların göreceli olması tercih edilir.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `NotUsing`, ancak `StdAfx.cpp` için `Create` olarak özel ayarlanmış.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterGrnLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MinSpace` (/O1) (Boyut için optimizasyon).
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
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\\` (Örn: `VSFiles/EterGrnLib/Distribute/`)
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

Proje dosyasında (`EterGrnLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **bulunmamaktadır**. Ayarlar muhtemelen varsayılanlardan, `.props` dosyalarından veya diğer genel Visual Studio ayarlarından miras alınmaktadır. Özellikle ön işlemci tanımları (`_WIN64`), çalışma zamanı kütüphaneleri ve PCH ayarları gibi kritik konfigürasyonlar için bu platformlarda derleme yapılıyorsa detaylı inceleme ve gerekirse eksik ayarların tanımlanması önemlidir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. Bu platformlara özel ayarlar genellikle `Win32` yapılandırmalarına benzer prensipleri izler. Ön derlenmiş başlıklar (`StdAfx.cpp` için `Create` ve genel `Use`) ARM için aktif görünmektedir.

**Kaynak Dosyalar:**

Proje, `../../Source/EterGrnLib/` dizini altında Granny 3D entegrasyonuyla ilgili C++ kaynak (`.cpp`) ve başlık (`.h`) dosyalarını içerir (yukarıda `EterGrnLib.vcxproj.filters` bölümünde listelenmiştir).

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   `StdAfx.cpp` dosyası `Debug|Win32`, `Release|Win32`, `Distribute|Win32` ve tüm ARM yapılandırmalarında PCH oluşturmak (`Create`) üzere ayarlanmıştır.
*   `Debug|Win32` ve `Release|Win32` yapılandırmalarında, genel PCH ayarı `NotUsing` iken, `StdAfx.cpp` PCH oluşturur. Bu, diğer dosyaların PCH'yi kullanmayacağı anlamına gelir (özel olarak `Use` olarak ayarlanmadıkça).
*   `Distribute|Win32` yapılandırmasında genel PCH ayarı `Use` şeklindedir.
*   x64 platformları için PCH ayarları `ItemDefinitionGroup` eksikliği nedeniyle belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `EterGrnLib`, statik bir kütüphane (`.lib`) olarak derlenir ve özellikle grafik motoru (`GameLib` veya ilgili grafik kütüphaneleri) tarafından kullanılır.
*   Proje, `../../External/include` altındaki harici başlık dosyalarına bağımlıdır. Bu genellikle Granny 3D SDK'sının başlık dosyalarını içerir.
*   `Debug|Win32` yapılandırması, taşınabilirliği olumsuz etkileyebilecek mutlak yollar (`D:\\Développement\\...`) içermektedir.
*   **x64 Yapılandırma Eksiklikleri**: Proje dosyasında `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması en önemli eksikliktir. Bu, `_WIN64` gibi platforma özgü ön işlemci tanımlarının, doğru çalışma zamanı kütüphanelerinin ve PCH ayarlarının eksik olabileceği anlamına gelir. Tam bir 64-bit derleme hedefleniyorsa bu yapılandırmaların dikkatlice tanımlanması gerekir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması, platforma özgü kodlarda sorunlara yol açabilir. `_ARM_` gibi daha spesifik makrolar kullanılmalıdır.
*   **Optimizasyon Ayarı**: `Release|Win32` yapılandırmasında `MinSpace` (/O1) optimizasyonunun seçilmiş olması, hız yerine dosya boyutunu önceliklendirdiğini gösterir. `Distribute|Win32` ise hızı (`Speed` /Ot) önceliklendirir.

--- 

**Not:** `VSFiles` klasörünün dokümantasyonu `client_VSFiles_Referans_Part2.md` dosyasında devam etmektedir.

*   **Bölüm 2**: [client_VSFiles_Referans_Part2.md](client_VSFiles_Referans_Part2.md)
    *   `EterImageLib.vcxproj`: Görüntü İşleme Kütüphanesi Projesi
    *   `EterLib.vcxproj`: Genel Eter Kütüphanesi Projesi
    *   `EterLocale.vcxproj`: Yerelleştirme Kütüphanesi Projesi
    *   `EterPack.vcxproj`: Paket Dosya Sistemi Kütüphanesi Projesi
    *   `EterPythonLib.vcxproj`: Python Entegrasyon Kütüphanesi Projesi
*   **Bölüm 3**: [client_VSFiles_Referans_Part3.md](client_VSFiles_Referans_Part3.md)
    *   `GameLib.vcxproj`: Oyun Mantığı Kütüphanesi Projesi
    *   `MilesLib.vcxproj`: Miles Ses Sistemi Kütüphanesi Projesi
    *   `ScriptLib.vcxproj`: Betik Yönetim Kütüphanesi Projesi
    *   `SpeedTreeLib.vcxproj`: SpeedTree Kütüphanesi Projesi
*   **Bölüm 4**: [client_VSFiles_Referans_Part4.md](client_VSFiles_Referans_Part4.md)
    *   `SphereLib.vcxproj`: Küre ve Hacim Kütüphanesi Projesi
    *   `PRTerrainLib.vcxproj` (TerrainLib): Arazi Kütüphanesi Projesi
    *   `UserInterface.vcxproj`: Ana İstemci Uygulaması Projesi