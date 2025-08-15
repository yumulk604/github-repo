`liblua`, Lua script dilini Metin2 sunucusuna entegre etmek, C++ ve Lua arasında iletişim kurmak ve sunucu işlevlerini Lua scriptlerine açmak için kullanılan temel kütüphanedir.

## Klasör Yapısı ve Ana Bileşenler

*   **`include/`:** Lua C API'sinin standart başlık dosyalarını içerir. Bu dosyalar, C++ kodunun Lua ile etkileşim kurmasını sağlar.
    *   **`lua.h`:** Temel Lua C API fonksiyonları, türleri ve sabitleri (yığın yönetimi, fonksiyon çağırma vb.).
    *   **`lauxlib.h`:** Lua C API'si için yardımcı (auxiliary) fonksiyonlar ve makrolar (dosya yükleme, hata raporlama, üst düzey işlemler).
    *   **`lualib.h`:** Lua standart kütüphanelerini (base, table, string vb.) C'den açmak için fonksiyonlar (örn: `luaL_openlibs`).
*   **`src/`:** **Lua yorumlayıcısının kendi kaynak kodunu** içerir (`lapi.c`, `lvm.c`, `lparser.c`, `lgc.c` vb.). Bu, Lua'nın harici bir kütüphane olarak değil, doğrudan projenin bir parçası olarak derlendiğini gösterir.
*   **Not (Detaylı Analiz Kapsamı):** `src/` klasörü, Lua yorumlayıcısının tam kaynak kodunu içerir. Bu kodu derinlemesine analiz etmek, Lua'nın iç çalışma mekanizmalarını anlamayı sağlasa da, standart Metin2 geliştirme görevleri (yeni görevler yazmak, C++ fonksiyonlarını Lua'ya bağlamak vb.) için genellikle gerekli değildir. Bu tür görevler için Lua dilinin kendisine, Lua C API'sine (`include/` altındaki başlıklar ve Lua resmi belgeleri) ve **diğer modüllerde (örn. `libgame`, `game`) bulunan Metin2'ye özgü Lua binding'lerine** odaklanmak daha verimlidir. Bu `liblua` kütüphanesi, temel olarak oyunun Lua scriptlerini çalıştırabilmesi için gerekli olan Lua yorumlayıcısını derleyip sağlamaktan sorumludur.
*   **`lib/`:** Büyük olasılıkla `src/` klasöründeki Lua kaynak kodunun derlenmesiyle oluşturulan statik Lua kütüphanesini (`liblua.a` veya `liblua.lib`) içerir.
*   **`Makefile`, `liblua.vcxproj`, `config`:** Kütüphanenin (yani Lua yorumlayıcısının) farklı platformlarda nasıl derleneceğini tanımlayan dosyalar.
*   **`image.png`:** Muhtemelen kütüphaneyle ilgisiz, yanlışlıkla eklenmiş bir resim dosyası. 