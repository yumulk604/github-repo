# EterBase Referans Kılavuzu - Bölüm 2

Bu belge, `@srcClient/Source/EterBase/` klasöründe bulunan `EterBase` modülünün kalan bileşenlerinin detaylandırılmasına devam etmektedir.

---

## Dosya Bazlı Detaylandırma (Devam)

### `obfuscate.h`

*   **Amaç:** Bu başlık dosyası, C++14 için derleme zamanında string literallerini karartmak (obfuscate) amacıyla kullanılan, Adam Yaxley tarafından geliştirilmiş harici bir kütüphaneyi içerir. Temel amacı, ikili (binary) dosya içerisinde string'lerin düz metin olarak görünmesini engelleyerek basit tersine mühendislik çabalarını zorlaştırmaktır.
*   **Genel Çalışma Prensibi:**
    *   String literalleri `AY_OBFUSCATE("gizlenecek_string")` veya `AY_OBFUSCATE_KEY("gizlenecek_string", ozel_anahtar)` makroları kullanılarak şifrelenir.
    *   Şifreleme, derleme zamanında (`constexpr` kullanılarak) basit bir XOR tabanlı şifreleme algoritması ile yapılır.
    *   Her şifrelenmiş string için bir anahtar kullanılır. Varsayılan anahtar, karartmanın yapıldığı satır numarasından (`__LINE__` veya Visual Studio için özel `AY_LINE`) türetilir. `ay::generate_key` fonksiyonu, MurmurHash3 benzeri bir algoritmayla bu seed'den 64-bitlik bir anahtar üretir.
    *   Karartılmış string, çalışma zamanında `ay::obfuscated_data` sınıfı tarafından yönetilir.
    *   Bu sınıf, string'e erişildiğinde (örneğin, `char*` türüne dönüştürüldüğünde) otomatik olarak deşifreleme yapar ve deşifrelenmiş string'e bir işaretçi döndürür.
    *   Kullanım sonrası string'in bellekte tekrar şifrelenmesi (`encrypt()`) veya yıkıcıda (`~obfuscated_data`) sıfırlanması gibi özellikler sunar.
*   **Ana Bileşenler:**
    *   **`ay::generate_key(seed)` (constexpr):** Verilen bir `seed`'den 64-bitlik bir şifreleme anahtarı üretir.
    *   **`ay::cipher(data, size, key)` (constexpr):** Verilen veriyi/string'i XOR algoritması ve anahtar ile şifreler veya deşifreler.
    *   **`ay::obfuscator<N, KEY>` Sınıfı (template, constexpr):** Derleme zamanında bir string alır, verilen anahtarla şifreler ve şifrelenmiş veriyi saklar.
    *   **`ay::obfuscated_data<N, KEY>` Sınıfı (template):** Çalışma zamanında şifrelenmiş veriyi tutar. Otomatik (implicit `operator char*()`) veya manuel (`decrypt()`, `encrypt()`) olarak deşifreleme/şifreleme yetenekleri sunar. Yıkıcısında veriyi bellekten temizler.
    *   **`AY_OBFUSCATE(data)` Makrosu:** String'i varsayılan anahtarla karartmak için kullanılır.
    *   **`AY_OBFUSCATE_KEY(data, key)` Makrosu:** String'i kullanıcı tanımlı bir anahtarla karartmak için kullanılır.
    *   Makrolar, bir lambda fonksiyonu içinde statik bir `ay::obfuscated_data` nesnesi oluşturarak global ömürlü ve tekil örnekler sağlar.
*   **Kullanım Alanı:**
    *   İstemci kodunda gömülü olan ve ikili dosyada kolayca bulunması istenmeyen hassas string literallerini (örneğin, bazı özel anahtarlar, geliştirici mesajları, belirli konfigürasyon stringleri, anti-hile ile ilgili olabilecek metinler) gizlemek için kullanılır.
    *   Bu, programın ikili analizini yapan birinin string tablosuna bakarak kolayca bilgi edinmesini bir nebze zorlaştırır.
    *   **Not:** Kullanılan XOR şifrelemesi ve anahtar türetme yöntemi (satır numarasına dayalı) kriptografik olarak çok güçlü değildir, ancak string'lerin düz metin olarak görünmesini engellemek için basit ve etkili bir yöntemdir.
    *   **Bağımlılıklar:** Sadece C++14 standart başlıklarını kullanır, harici bir `.cpp` dosyası veya kütüphane gerektirmez.

### `Random.h` ve `Random.cpp`

*   **Amaç:** Bu dosyalar, oyun içinde çeşitli rastgele sayı üretme ihtiyaçları için fonksiyonlar sunar. Standart C kütüphanesindeki `rand()` fonksiyonuna alternatif olarak, belirli istatistiksel özelliklere sahip bir rastgele sayı üreteci (Lehmer RNG) ve belirli aralıklarda sayı üretimi için yardımcı fonksiyonlar sağlar.
*   **Fonksiyonlar (`Random.h` bildirimleri, `Random.cpp` implementasyonları):**
    *   **`void srandom(unsigned long seed)`:**
        *   Rastgele sayı üretecinin tohum (seed) değerini ayarlar. Bu, üretecin başlangıç durumunu belirler.
        *   Aynı `seed` değeriyle başlatılan üreteç, her zaman aynı rastgele sayı dizisini üretecektir.
        *   Global statik `randseed` değişkenini günceller.
    *   **`unsigned long random()`:**
        *   Park ve Miller tarafından önerilen bir Lehmer rastgele sayı üreteci (MINSTD varyantı: `X_n+1 = (16807 * X_n) mod (2^31 - 1)`) kullanır.
        *   `[0, 2^31 - 1]` aralığında uniform dağılıma sahip işaretsiz bir uzun tamsayı döndürür.
        *   Her çağrıda dahili `randseed` değerini günceller.
    *   **`float frandom(float flLow, float flHigh)`:**
        *   `random()` fonksiyonunu kullanarak `[0, 1)` aralığında bir ondalıklı sayı üretir.
        *   Bu sayıyı ölçekleyerek `[flLow, flHigh)` aralığında (yani `flLow` dahil, `flHigh` hariç) rastgele bir `float` değer döndürür.
    *   **`long random_range(long from, long to)`:**
        *   `random()` fonksiyonunu kullanarak `[from, to]` aralığında (her iki sınır da dahil) rastgele bir `long` tamsayı döndürür.
        *   `from <= to` olmalıdır (bir `assert` ile kontrol edilir).
*   **Kullanım Alanı:**
    *   **Genel Rastgelelik:** Oyunun herhangi bir yerinde rastgele bir olayın sonucunu belirlemek, rastgele seçimler yapmak veya rastgele değerler atamak için kullanılır.
    *   **Yapay Zeka:** NPC veya canavar davranışlarında rastgelelik katmak (hareket, saldırı seçimi, bekleme süreleri).
    *   **Oyun Mekanikleri:** Hasar hesaplamalarına varyasyon eklemek, kritik vuruş olasılıkları, eşya düşürme (loot drop) oranları, görevlerin rastgele bileşenleri.
    *   **Görsel Efektler ve Animasyonlar:** Parçacıkların başlangıç hızları, yönleri, ömürleri; animasyonlarda küçük rastgele varyasyonlar.
    *   **Prosedürel Üretim:** Harita üzerinde nesnelerin (ağaçlar, kayalar vb.) rastgele yerleşimi gibi durumlarda kullanılabilir.
    *   Oyun başlangıcında genellikle `srandom(time(NULL))` gibi bir çağrı ile tohum ayarlanarak her oyun oturumunda farklı bir rastgelelik deneyimi sağlanır.

### `ServiceDefs.h`

*   **Amaç:** Bu başlık dosyası, istemcinin çeşitli servisleri ve özellikleri için derleme zamanı yapılandırma makrolarını (genellikle özellik bayrakları - feature flags) ve EterPack paket sistemi ile ilgili temel sabitleri (örneğin, anahtar setleri, dosya uzantıları, şifreleme anahtarları) tanımlar.
*   **İçerik:**
    *   **Özellik Bayrakları (Feature Flags - Makrolar):**
        *   `__IMPROVED_PACKET_ENCRYPTION__`: Tanımlıysa, geliştirilmiş ağ paketi şifrelemesini etkinleştirir (muhtemelen `cipher.h` içindeki `Cipher` sınıfını devreye sokar).
        *   `__GM_THROUGH_EVERYTHING__`: Tanımlıysa, Game Master (GM) karakterlerinin oyun dünyasındaki nesnelerin ve engellerin içinden geçebilmesini sağlar.
        *   `__SEND_SEQUENCE__`: (Yorum satırı içinde) Paket gönderiminde sıra numarası kontrolünü etkinleştirmek için kullanılmış olabilir.
        *   `__DOUBLE_RECV_BUFFER__`: Tanımlıysa, ağdan veri alımı için çift arabellek (double buffer) kullanımını etkinleştirir.
        *   `__VERTEX_BUFFER_PERFORMANCE__`: Tanımlıysa, vertex buffer ile ilgili bir performans optimizasyonunu etkinleştirir.
    *   **`EEterPackKeySet` Enum'u:**
        *   EterPack (`.epk`/`.eix`) dosyaları ve diğer önemli veri paketleri (eşya/mob prototipleri) için kullanılan farklı anahtar setlerini veya işlem türlerini tanımlayan sabitler içerir:
            *   `COMPRESS_EIX`, `COMPRESS_EPK`: Genel `.eix` ve `.epk` dosyaları için.
            *   `COMPRESS_ITEM`, `COMPRESS_MOB`: Eşya ve yaratık prototipleri için.
            *   Ayrıca anahtar yapısıyla ilgili boyut sabitleri (`COMPRESS_BRICK_SIZE`, `COMPRESS_BIRCK_LEN`, `COMPRESS_KEY_SIZE`) içerir. (`COMPRESS_BIRCK_LEN` isminde muhtemel bir yazım hatası var: "BRICK" olmalı).
    *   **Global Sabitler (Dosya Uzantıları ve Şifreleme Anahtarları):**
        *   `s_strEIX (".idx")`, `s_strEPK (".data")`: `.eix` ve `.epk` dosyaları için standart uzantılar.
        *   `s_strEterPackSecurityKey[]`, `s_strEterPackKey[]`, `s_strItemProtoKey[]`, `s_strMobProtoKey[]`: Bu `std::string` dizileri, sırasıyla genel EterPack dosyaları, eşya prototipleri ve mob prototipleri için kullanılan şifreleme/deşifreleme anahtarlarının parçalarını içerir. Her bir anahtar parçası, `AY_OBFUSCATE` makrosu (bkz. `obfuscate.h`) kullanılarak derleme zamanında karartılmıştır. Bu parçalar, çalışma zamanında birleştirilerek asıl anahtarları oluşturur.
*   **Kullanım Alanı:**
    *   **Derleme Zamanı Yapılandırması:** Özellik bayrakları, istemcinin hangi özelliklerle derleneceğini belirler. Bu, farklı istemci yapıları (örneğin, geliştirme vs. yayın) oluşturmak veya özellikleri modüler olarak etkinleştirmek/devre dışı bırakmak için kullanılır.
    *   **Varlık ve Veri Güvenliği:** Enum ve karartılmış anahtar string'leri, oyunun varlık paketlerinin (.epk) ve önemli veri dosyalarının (item_proto, mob_proto) şifrelenmesi ve çalışma zamanında güvenli bir şekilde açılması için temel altyapıyı sağlar. Anahtarların karartılması, bu anahtarların istemci belleğinden veya ikili dosyasından kolayca elde edilmesini zorlaştırmaya yöneliktir.
    *   **Paket Sistemi Yönetimi:** Tanımlanan sabitler, EterPack sisteminin dosyaları nasıl işlemesi gerektiğini (hangi anahtarları kullanacağı, hangi uzantıları bekleyeceği vb.) belirlemesine yardımcı olur.

### `StdAfx.h` ve `StdAfx.cpp`

*   **Amaç:** Bu dosyalar, `EterBase` projesi için ön derlenmiş başlık (precompiled header - PCH) mekanizmasını oluşturur. Sık kullanılan ve nadiren değiştirilen sistem başlıklarını (Windows API, STL) ve proje içi temel başlıkları (`vk.h`, `filename.h`, `ServiceDefs.h` vb.) içerir.
*   **Temel Özellikler/İçerik (`StdAfx.h`):**
    *   **Windows ve CRT Ayarları:** `WIN32_LEAN_AND_MEAN`, `_CRT_SECURE_NO_WARNINGS` gibi makrolarla derleme ayarları yapılır.
    *   **Uyarı Yönetimi:** Çeşitli `#pragma warning(disable: ...)` direktifleri ile belirli derleyici uyarıları (özellikle STL ve CRT kaynaklı) bastırılır.
    *   **Temel Sistem Başlıkları:** `<windows.h>`, `<assert.h>`, `<stdio.h>`, `<mmsystem.h>`, `<imagehlp.h>`, `<time.h>`.
    *   **STL Başlıkları:** `<algorithm>`, `<string>`, `<vector>`, `<deque>`, `<list>`, `<map>`.
    *   **Uyumluluk Makroları:** Eski CRT fonksiyon isimlerini (`stricmp`, `atoi` vb.) yeni versiyonlarla (`_stricmp`, `_atoi64` vb.) eşleştiren makrolar (VS2005 ve sonrası için).
    *   **Armadillo Nanomite Makroları:** `NANOBEGIN`, `NANOEND` makroları, potansiyel bir yazılım koruma mekanizmasıyla ilgili assembly kodları içerir.
    *   **Proje İçi `EterBase` Başlıkları:** `obfuscate.h`, `vk.h`, `filename.h`, `ServiceDefs.h`.
    *   **Harici Bağımlılık:** `../UserInterface/Locale_inc.h` başlığını içerir. Bu, `EterBase`'in `UserInterface` modülüne bir bağımlılığı olduğunu gösterir (yerelleştirme için).
*   **`StdAfx.cpp`:**
    *   Genellikle sadece `#include "StdAfx.h"` satırını içerir.
    *   Bu dosyanın derlenmesi, Visual Studio'nun `StdAfx.h` içinde listelenen tüm başlıkları işlemesini ve bir `.pch` dosyası oluşturmasını tetikler.
*   **Kullanım Alanı:** Doğrudan oyun mantığında bir rolü yoktur. Temel amacı, Visual Studio'da ön derlenmiş başlıkları kullanarak `EterBase` kütüphanesinin ve ona bağımlı diğer projelerin derleme sürelerini önemli ölçüde kısaltmaktır. `.pch` dosyası, sık kullanılan başlıkların her `.cpp` dosyasında tekrar tekrar işlenmesini engeller.

### `TempFile.h` ve `TempFile.cpp`

*   **Amaç:** `CTempFile` sınıfı, `CFileBase`'den türeyerek, geçici bir dosyanın oluşturulmasını ve otomatik olarak temizlenmesini yönetir. RAII (Resource Acquisition Is Initialization) prensibini kullanarak, nesne oluşturulduğunda geçici bir dosya yaratır ve nesne yok edildiğinde (kapsam dışına çıktığında) bu dosyayı otomatik olarak siler.
*   **Temel Özellikler/İçerik (`TempFile.h`):**
    *   **Kalıtım:** `CFileBase`
    *   **Yapıcı (`CTempFile(const char* c_pszPrefix = NULL)`):** İsteğe bağlı bir dosya adı öneki alır.
    *   **Yıkıcı (`virtual ~CTempFile()`):** Oluşturulan geçici dosyayı siler.
    *   **Korumalı Üye (`m_szFileName`):** Oluşturulan geçici dosyanın tam yolunu saklar.
*   **Implementasyon Detayları (`TempFile.cpp`):
    *   **Yapıcı:**
        1.  `EterBase/Utils.h`'daki `CreateTempFileName()` fonksiyonunu kullanarak sistemden benzersiz bir geçici dosya adı alır.
        2.  Bu adı `m_szFileName`'e kaydeder.
        3.  `CFileBase::Create()` metodunu bu dosya adı ve yazma modu (`FILEMODE_WRITE`) ile çağırarak dosyayı oluşturur ve açar.
    *   **Yıkıcı:**
        1.  `CFileBase::Destroy()` ile dosya tanıtıcısını kapatır.
        2.  Windows API `DeleteFile(m_szFileName)` ile dosyayı diskten siler.
*   **Kullanım Alanı:**
    *   Programın çalışması sırasında geçici depolama alanına ihtiyaç duyulduğunda kullanılır.
    *   Örneğin, büyük veri setlerinin işlenmesi sırasında ara adımları kaydetmek, harici bir araçla veri alışverişi yapmak veya indirilen/işlenen dosyaların geçici kopyalarını tutmak için kullanılabilir.
    *   RAII yaklaşımı sayesinde, geçici dosyanın oluşturulduktan sonra manuel olarak silinmesinin unutulması riskini ortadan kaldırır ve kaynak sızıntılarını önler.

### `vk.h`

*   **Amaç:** Bu başlık dosyası, 0-9 rakamları ve A-Z harfleri için standart Windows sanal klavye kodlarını (virtual key codes) tanımlayan `#define` direktiflerini içerir.
*   **İçerik:**
    *   Rakamlar için `VK_0` (0x30) ile `VK_9` (0x39) arası tanımlar.
    *   Harfler için `VK_A` (0x41) ile `VK_Z` (0x5A) arası tanımlar.
    *   Tanımlar, olası yeniden tanımlama hatalarını önlemek için `#ifndef ... #endif` blokları içine alınmıştır (ancak bu kodlar genellikle `<windows.h>` tarafından zaten tanımlanır).
*   **Kullanım Alanı:**
    *   İstemci kodunda klavye girdilerini işlerken kullanılır. Windows mesaj döngüsünde (`WM_KEYDOWN`, `WM_KEYUP` vb.) gelen tuş kodları, bu `VK_*` sabitleriyle karşılaştırılarak hangi tuşa basıldığını belirlemek ve ilgili eylemi (hareket, yetenek kullanımı, arayüz açma vb.) tetiklemek için kullanılır.


</rewritten_file> 