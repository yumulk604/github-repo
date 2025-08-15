# VSFiles Klasörü Referans Kılavuzu - Bölüm 4

Bu belge, Metin2 istemci kaynak kodu (`@srcClient`) içindeki `VSFiles` klasörünün yapısını ve amacını açıklamaya devam etmektedir.

---

### `SphereLib.vcxproj` (Küre ve Hacim Kütüphanesi Projesi)

Bu proje dosyası, muhtemelen 3D uzayda küresel hacimlerle ilgili hesaplamalar, çarpışma detection (tespiti) veya culling (ayıklama) gibi geometrik işlemler için kullanılan `SphereLib` statik kütüphanesini (`.lib`) derlemek amacıyla kullanılır. `frustum.h`, `sphere.h`, `vector.h` gibi başlık dosyaları, bu kütüphanenin temel geometrik şekiller ve vektör matematiği üzerine kurulu olduğunu düşündürmektedir. `spherepack.h/cpp` dosyaları, küre paketleme algoritmaları veya birden fazla kürenin yönetimiyle ilgili olabilir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{B1F24BAE-61E8-41E9-B2BB-A6905E5D64FD}`
*   **Kök Ad Alanı (RootNamespace)**: `SphereLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (SphereLib.lib üretir)

**`SphereLib.vcxproj.user` Dosyası:**

Sağlanan bilgilere göre bu dosya boştur (`<PropertyGroup />`) ve kullanıcıya özel bir yapılandırma içermemektedir.

**`SphereLib.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını filtreler altında düzenler:
*   **Header Files**:
    *   `frustum.h`: Görüş alanı (frustum) tanımları ve işlemleri. Genellikle culling için kullanılır.
    *   `pool.h`: Bellek havuzu yönetimi için olabilir, sık oluşturulup yok edilen geometrik nesneler için performansı artırır.
    *   `sphere.h`: Tek bir küre geometrisi, özellikleri ve işlemleri.
    *   `spherepack.h`: Birden fazla kürenin paketlenmesi, gruplanması veya yönetimi ile ilgili olabilir.
    *   `StdAfx.h`: Ön derlenmiş başlıklar.
    *   `vector.h`: 3D vektör işlemleri için temel matematik fonksiyonları.
*   **Source Files**:
    *   `frustum.cpp`: `frustum.h` içindeki fonksiyonların implementasyonu.
    *   `sphere.cpp`: `sphere.h` içindeki fonksiyonların implementasyonu.
    *   `spherepack.cpp`: `spherepack.h` içindeki fonksiyonların implementasyonu.
    *   `StdAfx.cpp`: Ön derlenmiş başlık oluşturma dosyası.

Tüm kaynak dosyalar `../../Source/SphereLib/` dizininden referans alınmıştır.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları yine eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client` (Genellikle `srcClient/Client/SphereLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, ancak genel proje ayarı `NotUsing`. Bu, PCH'nin etkin bir şekilde kullanılmadığı anlamına gelir.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **Kayan Nokta Modeli (`FloatingPointModel`)**: `Strict` (/fp:strict). Bu, kayan nokta hesaplamalarında daha kesin ve standartlara uygun sonuçlar verir ancak performansı biraz düşürebilir.
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/SphereLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, genel ayar `NotUsing`. PCH etkin kullanılmıyor olabilir.
*   **Kayan Nokta Modeli (`FloatingPointModel`)**: `Strict` (/fp:strict).
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için son kullanıcı sürümü.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/SphereLib/Distribute/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: Genel ayar `Use`, `StdAfx.cpp` için `Create`. Bu yapılandırmada PCH doğru şekilde kullanılıyor.
*   **Kayan Nokta Modeli (`FloatingPointModel`)**: `Strict` (/fp:strict).
*   **Önemli Derleyici Ayarları**:
    *   `IntrinsicFunctions`: `true` (/Oi)
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**

Proje dosyasında (`SphereLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **tamamen eksiktir**. Bu platformlar için yalnızca genel `PlatformToolset` (`v143`) tanımlanmıştır. `x64` derlemeleri için hayati ayarların (örneğin `_WIN64` ön işlemci tanımı, doğru çalışma zamanı kütüphanesi, PCH ayarları, kayan nokta modeli vb.) eksik olması beklenir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. PCH kullanımı (`StdAfx.cpp` için `Create` ve genel `Use`) ve `FloatingPointModel` (`Strict`) ayarları ARM yapılandırmalarında da mevcuttur.
*   ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır.

**Kaynak Dosyalar:**

Proje, `../../Source/SphereLib/` dizini altında, yukarıda `SphereLib.vcxproj.filters` bölümünde detaylandırılan ve küresel/vektörsel hesaplamalar yapan C++ kaynak (`.cpp`) ve başlık (`.h`) dosyalarını içerir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   PCH kullanımı, diğer birçok kütüphanede gözlemlenen paterni takip eder:
    *   **`Debug|Win32` ve `Release|Win32`**: `StdAfx.cpp` için `Create`, genel ayar `NotUsing` (PCH etkisiz).
    *   **`Distribute|Win32` ve tüm `ARM` yapılandırmaları**: Genel ayar `Use`, `StdAfx.cpp` için `Create` (PCH doğru kullanılıyor).
*   **x64 platformları**: `ItemDefinitionGroup` eksikliği nedeniyle PCH ayarları belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `SphereLib`, statik bir kütüphane (`.lib`) olarak derlenir.
*   Kütüphane, temel geometrik ve vektör işlemleri sağladığı için muhtemelen `GameLib`, `EffectLib` veya grafik motoruyla ilgili diğer kütüphaneler tarafından kullanılır (örneğin, çarpışma tespiti, görüş alanı kontrolü, culling vb. için).
*   `Debug|Win32` yapılandırmasındaki mutlak yollar (`D:\Développement\...`) projenin taşınabilirliğini kısıtlar.
*   **Kritik x64 Yapılandırma Eksiklikleri**: `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması, bu platformda derleme yapılmasını son derece sorunlu hale getirir.
*   **PCH Kullanımında Tutarsızlık**: `Debug|Win32` ve `Release|Win32` yapılandırmalarındaki PCH ayarları etkisizdir.
*   **Kayan Nokta Modeli**: Tüm yapılandırmalarda `/fp:strict` kullanılması, potansiyel performans maliyetine rağmen kayan nokta hesaplamalarında tutarlılık ve doğruluğa öncelik verildiğini gösterir. Bu, özellikle geometrik hesaplamaların hassas olması gereken durumlarda önemlidir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması platforma özgü kodlarda sorun yaratabilir.

---

### `PRTerrainLib.vcxproj` (Arazi Kütüphanesi Projesi)

Bu proje dosyası, Metin2 istemcisindeki oyun dünyası arazilerinin oluşturulması, yüklenmesi, işlenmesi ve yönetilmesi için kullanılan `PRTerrainLib` (genellikle `TerrainLib` olarak da anılır) statik kütüphanesini (`.lib`) derlemek amacıyla kullanılır. Bu kütüphane, arazi yükseklik haritalarını, doku katmanlarını (splatting), su yüzeylerini ve araziyle ilgili diğer görsel ve çarpışma verilerini yönetir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{A1ED61AC-6324-43B1-BC9B-548208D625CF}`
*   **Proje Adı (ProjectName)**: `PRTerrainLib` (Klasör adı `TerrainLib`, Kök Ad Alanı `TerrainLib` olmasına rağmen proje adı `PRTerrainLib` olarak belirtilmiş)
*   **Kök Ad Alanı (RootNamespace)**: `TerrainLib`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `StaticLibrary` (PRTerrainLib.lib üretir)

**`TerrainLib.vcxproj.user` Dosyası:**

Sağlanan bilgilere göre bu dosya boştur (`<PropertyGroup />`) ve kullanıcıya özel bir yapılandırma içermemektedir.

**`TerrainLib.vcxproj.filters` Dosyası:**

Proje içindeki kaynak ve başlık dosyalarını filtreler altında düzenler:
*   **Header Files**:
    *   `StdAfx.h`: Ön derlenmiş başlıklar.
    *   `Terrain.h`: Ana arazi sınıfı ve arazi yönetimi fonksiyonları.
    *   `TerrainType.h`: Arazi ile ilgili çeşitli tür tanımları (örneğin, arazi fırçaları, doku katmanları).
    *   `TextureSet.h`: Arazi doku setlerinin yönetimi.
*   **Source Files**:
    *   `StdAfx.cpp`: Ön derlenmiş başlık oluşturma dosyası.
    *   `Terrain.cpp`: `Terrain.h` içindeki fonksiyonların implementasyonu.
    *   `TextureSet.cpp`: `TextureSet.h` içindeki fonksiyonların implementasyonu.

Tüm kaynak dosyalar `../../Source/PRTerrainLib/` dizininden referans alınmıştır.

**Yapılandırmaya Göre Önemli Ayarlar:**

Aşağıda `Win32` platformu için `Debug`, `Release` ve `Distribute` yapılandırmalarındaki kilit ayarlar özetlenmiştir. `x64` platformu için proje dosyasında detaylı `ItemDefinitionGroup` blokları yine eksiktir.

**1. Debug | Win32 Yapılandırması:**

*   **Amaç**: Geliştirme ve hata ayıklama.
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client` (Genellikle `srcClient/Client/PRTerrainLibD.lib` gibi bir yola işaret eder)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Optimizasyon (`Optimization`)**: `Disabled`
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`.
    *   *Not: Proje dosyasında `Debug|Win32` için `PropertyGroup` içinde `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak `IncludePath` ve `LibraryPath` tanımları bulunmaktadır.*
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, ancak genel proje ayarı `NotUsing`. Bu, PCH'nin etkin bir şekilde kullanılmadığı anlamına gelir.
*   **Hata Ayıklama Bilgisi (`DebugInformationFormat`)**: `ProgramDatabase` (/Zi)
*   **C++ Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **En Az Yeniden Derleme (`MinimalRebuild`)**: `false`.
*   **Çoklu İşlemci Derlemesi (`MultiProcessorCompilation`)**: `true`

**2. Release | Win32 Yapılandırması:**

*   **Amaç**: Performans için optimize edilmiş sürüm.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/TerrainLib/Release/`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_LIB`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, genel ayar `NotUsing`. PCH etkin kullanılmıyor olabilir.
*   **Önemli Derleyici Ayarları**:
    *   `FunctionLevelLinking`: `true` (/Gy)
    *   `StringPooling`: `true` (/GF)
    *   `WholeProgramOptimization`: `false`
    *   `IntrinsicFunctions`: `false`
    *   `InlineFunctionExpansion`: `OnlyExplicitInline` (/Ob1)
    *   `LanguageStandard`: `stdcpplatest`
    *   `MultiProcessorCompilation`: `true`

**3. Distribute | Win32 Yapılandırması:**

*   **Amaç**: Dağıtım için son kullanıcı sürümü.
*   **Çıktı Dizini (`OutDir`)**: `$(ProjectDir)$(Configuration)\` (Örn: `VSFiles/TerrainLib/Distribute/`)
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

Proje dosyasında (`TerrainLib.vcxproj`) `Debug|x64`, `Release|x64` ve `Distribute|x64` platformları için özel `ItemDefinitionGroup` blokları **tamamen eksiktir**. Bu platformlar için yalnızca genel `PlatformToolset` (`v143`) tanımlanmıştır. `x64` derlemeleri için hayati ayarların (örneğin `_WIN64` ön işlemci tanımı, doğru çalışma zamanı kütüphanesi, PCH ayarları vb.) eksik olması beklenir.

**ARM Platformları:**

Proje, ARM platformları (`Debug|ARM`, `Release|ARM`, `Distribute|ARM`) için de yapılandırma tanımları içerir. PCH kullanımı (`StdAfx.cpp` için `Create` ve genel `Use`) ARM yapılandırmalarında da mevcuttur.
*   ARM yapılandırmalarında da `WIN32` ön işlemci tanımı kullanılmaktadır.

**Kaynak Dosyalar:**

Proje, `../../Source/PRTerrainLib/` dizini altında, yukarıda `TerrainLib.vcxproj.filters` bölümünde detaylandırılan ve arazi yönetimini sağlayan C++ kaynak (`.cpp`) ve başlık (`.h`) dosyalarını içerir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**

*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   PCH kullanımı, diğer birçok kütüphanede gözlemlenen paterni takip eder:
    *   **`Debug|Win32` ve `Release|Win32`**: `StdAfx.cpp` için `Create`, genel ayar `NotUsing` (PCH etkisiz).
    *   **`Distribute|Win32` ve tüm `ARM` yapılandırmaları**: Genel ayar `Use`, `StdAfx.cpp` için `Create` (PCH doğru kullanılıyor).
*   **x64 platformları**: `ItemDefinitionGroup` eksikliği nedeniyle PCH ayarları belirsizdir.

**Bağımlılıklar ve Önemli Notlar:**

*   `PRTerrainLib` (TerrainLib), statik bir kütüphane (`.lib`) olarak derlenir.
*   Bu kütüphane, oyun dünyasının temelini oluşturan araziyi yönettiği için `GameLib` ve grafik motoruyla ilgili diğer kütüphaneler (`EterLib`, `EterGrnLib` vb.) tarafından yoğun bir şekilde kullanılır.
*   `Debug|Win32` yapılandırmasındaki mutlak yollar (`D:\Développement\...`) projenin taşınabilirliğini kısıtlar.
*   **Kritik x64 Yapılandırma Eksiklikleri**: `x64` platformları için özel `ItemDefinitionGroup` bloklarının olmaması, bu platformda derleme yapılmasını son derece sorunlu hale getirir.
*   **PCH Kullanımında Tutarsızlık**: `Debug|Win32` ve `Release|Win32` yapılandırmalarındaki PCH ayarları etkisizdir.
*   **Platform Ön İşlemci Tanımları**: ARM yapılandırmalarında `WIN32` tanımının kullanılması platforma özgü kodlarda sorun yaratabilir.
*   `Release|Win32` yapılandırmasında `WholeProgramOptimization` seçeneği `false` olarak ayarlanmıştır.

---

### `UserInterface.vcxproj` (Ana İstemci Uygulaması Projesi)

Bu proje dosyası, Metin2 oyun istemcisinin ana yürütülebilir dosyasını (`Metin2Debug.exe`, `Metin2Release.exe`, `Metin2Distribute.exe`) derlemek için kullanılır. Diğer tüm statik kütüphanelerin (`.lib`) bir araya getirildiği, Python betik motorunun başlatıldığı, oyunun ana döngüsünün yönetildiği, kullanıcı arayüzü etkileşimlerinin işlendiği ve ağ iletişiminin kurulduğu merkezi projedir.

**Genel Proje Bilgileri:**

*   **Proje GUID'i**: `{E0DC3917-08C3-4F15-A7E7-6CB395EC83F0}`
*   **Kök Ad Alanı (RootNamespace)**: `UserInterface`
*   **Hedeflenen Windows SDK**: `10.0`
*   **Visual Studio Araç Takımı (PlatformToolset)**: `v143` (Visual Studio 2022)
*   **Karakter Seti (CharacterSet)**: `MultiByte`
*   **Çıktı Türü (ConfigurationType)**: `Application` (.exe üretir)

**`UserInterface.vcxproj.user` Dosyası:**

*   Bu dosya, `Debug|Win32` yapılandırması için kullanıcıya özel hata ayıklama ayarları içerir.
*   **`DebuggerFlavor`**: `WindowsLocalDebugger` - Yerel Windows hata ayıklayıcısının kullanılacağını belirtir.
*   **`LocalDebuggerWorkingDirectory`**: `..\..\..\..\Client` - Hata ayıklama sırasında çalışma dizininin `srcClient/Client/` (proje dosyasından dört seviye yukarıdaki `Client` klasörü) olarak ayarlandığını gösterir. Bu, istemcinin çalışması için gerekli olan paketler, DLL'ler ve diğer kaynak dosyalarının bu dizinde bulunmasını beklediği anlamına gelir.

**`UserInterface.vcxproj.filters` Dosyası:**

Bu dosya, projenin çok sayıda kaynak (`.cpp`) ve başlık (`.h`) dosyasını mantıksal klasörler altında gruplandırır. Başlıca filtreler şunlardır:

*   **Abstract**: Soyut temel sınıflar.
*   **Game**: Oyun mantığı, karakter yönetimi, efektler, olay yönetimi, eşyalar, görevler gibi çekirdek oyun mekanikleriyle ilgili Python sarmalayıcıları ve C++ implementasyonları.
*   **GuildMark**: Lonca sembollerinin indirilmesi, yüklenmesi ve yönetimi.
*   **InstanceBase**: Oyun dünyasındaki tüm varlıkların (karakterler, NPC'ler, canavarlar, eşyalar vb.) temel sınıfı ve bununla ilişkili alt sistemler (savaş, efekt, hareket, dönüşüm).
*   **Interface**: Kullanıcı arayüzü elemanları (metin kuyrukları, mağaza, güvenli kutu, mini harita, mesajlaşma, lonca arayüzü, takas) için Python sarmalayıcıları.
*   **Locale**: Yerelleştirme ve dil desteği.
*   **Network**: Ağ iletişimi, hesap bağlantısı, hakaret filtresi, sunucu durumu kontrolü ve Python ağ akışı yönetimi. `PythonNetworkStreamPhase*` dosyaları oyunun farklı ağ aşamalarını (giriş, seçme, yükleme, oyun içi) yönetir.
*   **Pack**: Paketlenmiş dosya sistemine erişim için Python sarmalayıcısı.
*   **Security**: İstemci güvenliğiyle ilgili mekanizmalar (dosya kontrolü, MD5).
*   **System**: Ana uygulama döngüsü, Python entegrasyonu (uygulama, IME, sistem konfigürasyonu), konfigürasyon yönetimi, HWID yönetimi, yükleme ekranı ve diğer sistem seviyesi işlevler.
*   **Browser**: Gömülü Chromium web tarayıcısı (CEF - Chromium Embedded Framework) ile ilgili sınıflar.
*   **Resource Files**: Projenin kaynak dosyaları (`.rc`), ikonlar (`.ico`) ve fare imleçleri (`.cur`).

Dosya, `../../Source/UserInterface/` altındaki kaynaklara referans verir ve `minIni.c` gibi bazı harici (ancak proje içinde yer alan) dosyaları da içerir.

**Proje Bağımlılıkları (`ProjectReference`):**

`UserInterface.vcxproj`, çözümdeki diğer tüm statik kütüphanelerine bağımlıdır:
`CWebBrowser`, `EffectLib`, `EterBase`, `EterGrnLib`, `EterImageLib`, `EterLib`, `EterPack`, `EterPythonLib`, `GameLib`, `MilesLib`, `ScriptLib`, `SpeedTreeLib`, `SphereLib`, `TerrainLib (PRTerrainLib)`.
Bu, `UserInterface` projesi derlenmeden önce bu kütüphanelerin derlenmesi gerektiği ve linkleme aşamasında bu kütüphanelerin çıktılarının (`.lib` dosyaları) kullanılacağı anlamına gelir.

**Yapılandırmaya Göre Önemli Ayarlar:**

**1. Debug | Win32 Yapılandırması:**
*   **Çıktı Dosyası (`TargetName`)**: `Metin2$(Configuration)` (örn: `Metin2Debug.exe`)
*   **Çıktı Dizini (`OutDir`)**: `..\..\..\Client` (Proje dosyasından üç seviye yukarıdaki `Client` klasörü, yani `srcClient/Client/Metin2Debug.exe`)
*   **Ek Dahil Etme Dizinleri (`AdditionalIncludeDirectories`)**: `../../External/include`
*   **Kütüphane Dizinleri (`AdditionalLibraryDirectories`)**: `../../External/library/Visual_Studio`, `../../External/library`
    *   *Not: `PropertyGroup` içinde yine `D:\Développement\METIN2 GF Like\srcClient\External\include` ve `D:\Développement\METIN2 GF Like\srcClient\External\library` gibi mutlak yollar mevcut.*
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreadedDebug` (/MTd)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `_DEBUG`, `_WINDOWS`, `USE_LOD`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `NotUsing` (genel ayar). Bu, PCH'nin bu yapılandırmada kullanılmadığını gösterir. Bu durum, `StdAfx.cpp` dosyasının diğer birçok kütüphanede `Create` olarak ayarlanmasıyla çelişir ve muhtemelen bir yapılandırma hatasıdır veya PCH kasıtlı olarak devre dışı bırakılmıştır.
*   **Dil Standardı (`LanguageStandard`)**: `stdcpplatest`
*   **Linker Ayarları**:
    *   `UACExecutionLevel`: `RequireAdministrator` (Uygulamanın yönetici hakları gerektirdiğini belirtir).
    *   `SubSystem`: `Windows`
    *   `LargeAddressAware`: `true` (Uygulamanın 2GB'den fazla sanal adres alanı kullanabileceğini belirtir).
    *   `RandomizedBaseAddress`: `false` (ASLR'yi devre dışı bırakır, genellikle hata ayıklama için).
    *   `ImageHasSafeExceptionHandlers`: `false` (Güvenli özel durum işleyicileri tablosu oluşturulmaz).
    *   `Manifest`: `mt.exe -manifest Metin2Debug.exe.manifest -outputresource:Metin2Debug.exe;1` (Harici bir manifest dosyasını yürütülebilir dosyaya gömer).

**2. Release | Win32 Yapılandırması:**
*   **Çıktı Dosyası (`TargetName`)**: `Metin2$(Configuration)` (örn: `Metin2Release.exe`)
*   **Çıktı Dizini (`OutDir`)**: `$(SolutionDir)Binary\` (Çözüm dizini altındaki `Binary` klasörü, örn: `srcClient/Binary/Metin2Release.exe`)
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `MaxSpeed` (/O2)
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_WINDOWS`, `USE_LOD`, `DUNGEON_WORK`, `_CRT_SECURE_NO_WARNINGS`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `NotUsing` (PCH kullanılmıyor).
*   **Tüm Derleyici Çıktıları (`AssemblerOutput`)**: `All` (/FAcs). Bu, derleyicinin assembly kodu, makine kodu ve kaynak kodu içeren listeleme dosyaları üretmesini sağlar.
*   **Linker Ayarları**:
    *   `LinkIncremental`: `false`
    *   `IgnoreAllDefaultLibraries`: `true` (Tüm varsayılan kütüphaneleri yoksayar, gerekli tüm kütüphaneler `AdditionalDependencies` ile belirtilmelidir). Bu riskli bir ayardır ve eksik sembol hatalarına yol açabilir.
    *   `GenerateMapFile`: `true` (Linker bir map dosyası oluşturur).
    *   `WholeProgramOptimization`: `true` (/GL derleyici bayrağı ile etkinleştirilir, linkleme sırasında `/LTCG` kullanılır).

**3. Distribute | Win32 Yapılandırması:**
*   **Çıktı Dosyası (`TargetName`)**: `Metin2$(Configuration)` (örn: `Metin2Distribute.exe`)
*   **Çıktı Dizini (`OutDir`)**: `$(SolutionDir)Binary\`
*   **Çalışma Zamanı Kütüphanesi (`RuntimeLibrary`)**: `MultiThreaded` (/MT)
*   **Optimizasyon (`Optimization`)**: `FavorSizeOrSpeed` (`Speed` /Ot olarak ayarlı).
*   **Ön İşlemci Tanımları (`PreprocessorDefinitions`)**: `WIN32`, `NDEBUG`, `_WINDOWS`, `USE_LOD`, `_DISTRIBUTE`, `_CRT_SECURE_NO_WARNINGS`, `WINDOWS_IGNORE_PACKING_MISMATCH`.
*   **Ön Derlenmiş Başlık (`PrecompiledHeader`)**: `StdAfx.cpp` için `Create`, genel ayar `Use` (`StdAfx.h` kullanılır). PCH bu yapılandırmada doğru kullanılıyor.
*   **Linker Ayarları**:
    *   `LinkTimeCodeGeneration`: `UseLinkTimeCodeGeneration` (/LTCG)
    *   `MapExports`: `true`

**x64 Yapılandırmaları (Debug, Release, Distribute):**
*   **`Debug|x64`**: Yalnızca `PlatformToolset`, `OutDir` (`$(SolutionDir)Binary\x64\`) ve `TargetName` ayarlanmış. `ItemDefinitionGroup` içinde `AdditionalIncludeDirectories` `../../External/include` olarak belirtilmiş ancak diğer kritik derleyici ve linker ayarları eksik.
*   **`Release|x64` ve `Distribute|x64`**: Yalnızca `PlatformToolset` ayarlanmış. `ItemDefinitionGroup` blokları **tamamen eksiktir**. Bu, `x64` platformları için derlemenin ciddi şekilde eksik yapılandırıldığı anlamına gelir.

**ARM Platformları:**
*   ARM platformları için de yapılandırmalar mevcuttur.
*   **PCH Kullanımı**: `StdAfx.cpp` için `Create`, genel ayar `Use`. PCH doğru kullanılıyor.
*   **Ön İşlemci Tanımları**: `WIN32` tanımı ARM için de kullanılıyor.
*   **Linker Ayarları**: `Debug|ARM` için `GenerateDebugInformation` `false` olarak ayarlanmış, bu hata ayıklamayı zorlaştırabilir. `libcef_dll_wrapper.lib` ve `libcef.lib` gibi CEF'e özgü kütüphaneler ARM bağımlılıklarına eklenmiş.

**Kaynak Dosyalar ve Kaynaklar:**
*   Proje, `../../Source/UserInterface/` altında çok sayıda C++ kaynak (`.cpp`) ve başlık (`.h`) dosyası içerir.
*   Ayrıca çeşitli fare imleçleri (`.cur`), uygulama ikonları (`.ico`) ve bir manifest dosyası (`metin2client.exe.manifest`) içerir. `UserInterface.rc` kaynak betik dosyası, bu ikonları ve diğer Windows kaynaklarını yönetir.

**Ön Derlenmiş Başlık (PCH) Kullanımı:**
*   `StdAfx.h` ön derlenmiş başlık dosyası olarak hedeflenmiştir.
*   **`Debug|Win32` ve `Release|Win32`**: `StdAfx.cpp` için `PrecompiledHeader` ayarı `NotUsing` olarak belirtilmiş. Ancak `StdAfx.cpp` dosyasının kendisinde `/Yc` (Create) bayrağı `Release|Win32` ve `Debug|ARM` için var, `Debug|Win32` ve `Release|ARM` için `NotUsing` (veya eksik). Bu oldukça kafa karıştırıcı ve tutarsız bir PCH yapılandırmasıdır. `Distribute|Win32` ve `Distribute|ARM` için ise `Use` (/Yu) ve `Create` (/Yc) doğru bir şekilde ayarlanmış görünüyor.
*   **x64 platformları**: `ItemDefinitionGroup` eksikliği nedeniyle PCH ayarları belirsizdir.

**Önemli Notlar ve Potansiyel Sorunlar:**
*   **Ana Yürütülebilir Proje**: Bu, oyunun çalıştırılabilir (`.exe`) dosyasını üreten son projedir.
*   **Mutlak Yollar**: `Debug|Win32` yapılandırmasındaki `IncludePath` ve `LibraryPath` için mutlak yollar (`D:\Développement\...`) taşınabilirliği engeller.
*   **Kritik x64 Yapılandırma Eksiklikleri**: `x64` platformları (özellikle Release ve Distribute) için `ItemDefinitionGroup` bloklarının neredeyse tamamen eksik olması, bu platformlarda stabil ve optimize edilmiş derlemeler yapılmasını imkansız hale getirir.
*   **PCH Tutarsızlığı**: Özellikle `Debug|Win32` ve `Release|Win32` yapılandırmalarında PCH ayarları karmaşık ve muhtemelen hatalıdır. `StdAfx.cpp` dosyasının proje ayarlarında `NotUsing` olarak işaretlenmesi, ancak bazı yapılandırmalarda `Create` bayrağına sahip olması çelişkilidir.
*   **ARM için WIN32 Tanımı**: ARM platformlarında `WIN32` ön işlemci tanımının kullanılması sorunlara yol açabilir.
*   **`IgnoreAllDefaultLibraries` (Release|Win32)**: Bu ayar tehlikelidir. Gerekli tüm sistem kütüphanelerinin (`kernel32.lib`, `user32.lib` vb.) `AdditionalDependencies` içinde manuel olarak listelenmesini gerektirir. Eğer bir tanesi bile unutulursa, linkleme hataları oluşur.
*   **UAC İzni**: Uygulamanın yönetici izni istemesi (`RequireAdministrator`), bazı kullanıcı ortamlarında sorun yaratabilir veya gereksiz olabilir.
*   **CEF Entegrasyonu**: Proje, gömülü web tarayıcı için CEF (Chromium Embedded Framework) kullanmaktadır. Bu, `CWebBrowser.lib` bağımlılığı ve ARM yapılandırmasındaki `libcef.lib` gibi link ayarlarına yansır.
*   **Çıktı Dizini Farklılıkları**: `Debug` yapılandırmısı çıktıyı `srcClient/Client` altına, `Release` ve `Distribute` ise `srcClient/Binary` altına yazar. Bu tutarlı bir yaklaşımdır.
*   **Dil Standardı**: Birçok yapılandırmada C++'ın en son standartlarını kullanma eğilimi (`stdcpplatest`) moderndir.
*   **`ImageHasSafeExceptionHandlers` (false)**: Bu ayar, özellikle 64-bit sistemlerde ve SEH (Structured Exception Handling) kullanan kodlarda potansiyel güvenlik riskleri oluşturabilir.

Bu proje, istemcinin kalbidir ve diğer tüm kütüphanelerin işlevlerini bir araya getirerek son kullanıcıya sunulan uygulamayı oluşturur. Yapılandırmasındaki, özellikle x64 ve PCH ile ilgili tutarsızlıklar ve eksiklikler, projenin genel sağlığı ve bakımı için önemli sorun teşkil etmektedir. 