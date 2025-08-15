# Metin2 Sunucu (srcServer) Kaynak Kodu Referans Kılavuzu

Bu kılavuz, Metin2 sunucu kaynak kodunun (`srcServer`) yapısını, önemli klasörlerini ve temel bileşenlerini anlamak için bir referans noktası sağlamayı amaçlamaktadır.

---

## Genel Klasör Yapısı

### `External/`

*   **Amaç:** Bu klasör, sunucu kaynak kodunun derlenmesi ve çalışması için **gereken harici kütüphaneleri ve bağımlılıkları** barındırır. Metin2'nin çekirdek kodunun kendisi olmayan ama çalışması için ihtiyaç duyduğu üçüncü parti yazılımların veya paylaşılan bileşenlerin dosyaları burada bulunur.
*   **Tipik İçerik:**
    *   `include/`: Harici kütüphanelerin **başlık dosyalarını (.h, .hpp)** içerir. Bu dosyalar, ana sunucu kodunun (`Source/` altındaki) bu kütüphanelerin fonksiyonlarını ve yapılarını nasıl kullanacağını tanımlar.
    *   `lib/` (Opsiyonel): Önceden derlenmiş kütüphane dosyalarını (.lib, .a, .so) içerebilir.
    *   `bin/` (Opsiyonel): Harici araçlar veya dinamik bağlantı kütüphanelerini (.dll) içerebilir.
    *   **Örnek Kütüphaneler:** Bu projede `include` altında `fmt` (gelişmiş metin formatlama) ve `nlohmann` (JSON işleme) gibi kütüphanelerin başlık dosyaları bulunmaktadır.
*   **Önem:** Sunucunun başarıyla derlenebilmesi için bu klasördeki dosyalar kritik öneme sahiptir. Eksik veya yanlış yapılandırılmış harici bağımlılıklar derleme hatalarına yol açar. 

---

### `Source/`

*   **Amaç:** Metin2 sunucusunun **asıl kaynak kodunu** içerir. Oyun mantığı, veritabanı iletişimi, ağ yönetimi ve diğer çekirdek işlevler bu klasördeki C++ dosyalarında (.cpp, .h) ve alt klasörlerinde tanımlanır. `External` klasöründeki kütüphaneler burada kullanılır.
*   **Alt Klasör Yapısı (Önerilen İnceleme Sırası):**
    1.  **Temel Kütüphaneler (`lib*`):** Diğer modüllerin temelini oluşturan, genellikle yeniden kullanılabilir işlevsellikler içerir.
        *   `libthecore/`: Sunucunun çekirdek fonksiyonlarını (event loop, socket yönetimi, buffer işlemleri, zamanlama, loglama vb.) barındırır. ([`libthecore_Referans.md`](libthecore_Referans.md))
        *   `liblua/`: Lua script motoru entegrasyonu. ([`liblua_Referans.md`](liblua_Referans.md))
        *   `libsql/`: MySQL veritabanı işlemleri için sarmalayıcı fonksiyonlar içerir. ([`libsql_Referans.md`](libsql_Referans.md))
        *   `libpoly/`: Şekil ve polinom hesaplamaları için fonksiyonlar içerir (Muhtemelen kullanılmıyor). ([`libpoly_Referans.md`](libpoly_Referans.md))
        *   `libgame/`: Oyunun temel mekaniklerini, karakter yönetimini, eşya sistemini, görevleri (quest) ve diğer oyun mantığını içerir. ([`libgame_Referans.md`](libgame_Referans.md))
        *   `libserverkey/`: Sunucu anahtarı doğrulaması ve ilgili kriptografik işlemleri yönetir. ([`libserverkey_Referans.md`](libserverkey_Referans.md))
    2.  **Ortak Kod (`common/`):** Sunucu ve istemci tarafından paylaşılan ortak kodları (tanımlamalar, sabitler, veri yapıları) içerir. ([`common_Referans.md`](common_Referans.md))
    3.  **Ana Uygulama Modülleri:** Sunucunun ana çalıştırılabilir bileşenleri.
        *   `db/`: Veritabanı sunucusunun kodunu içerir (`libsql` ve `libthecore`'u kullanır). ([`db_Referans.md`](Source/db/db_Referans.md))
        *   `game/`: Oyun sunucusunun ana kodunu içerir (en kapsamlı bölüm, `libgame`'i kullanır).
            *   [`game/src/`](#gamesrc) - Oyun sunucusunun ana kaynak kodları. (Detaylar için aşağıdaki özel referans dosyalarına bakınız.)
                *   [Oyun Sunucusu - Çekirdek (`game_Core_Referans.md`)](Source/game/game_Core_Referans.md)
                *   [Oyun Sunucusu - Özellikler (`game_Features_Referans.md`)](Source/game/game_Features_Referans.md)
                    *   [Oyun Sunucusu - Özellikler Bölüm 2 (`game_Features_Referans_Part2.md`)](Source/game/game_Features_Referans_Part2.md)
                *   [Oyun Sunucusu - Görev Sistemi (`game_Quest_Referans.md`)](Source/game/game_Quest_Referans.md)
                *   [Oyun Sunucusu - Yardımcı Bileşenler (`game_Utils_Referans.md`)](Source/game/game_Utils_Referans.md)
                    *   [Oyun Sunucusu - Yardımcı Bileşenler Bölüm 2 (`game_Utils_Referans_Part2.md`)](Source/game/game_Utils_Referans_Part2.md)
*   **Not:** Bu klasördeki `.sln` (Visual Studio Solution) dosyası, projenin Visual Studio ortamında nasıl derleneceğini tanımlar. 

*   [`libsql`](libsql_Referans.md): MySQL veritabanı işlemleri için sarmalayıcı fonksiyonlar içerir.
*   [`libthecore`](libthecore_Referans.md): Sunucunun çekirdek fonksiyonlarını (event loop, socket yönetimi, buffer işlemleri, zamanlama, loglama vb.) barındırır.
*   [`libpoly`](libpoly_Referans.md): Şekil ve polinom hesaplamaları için fonksiyonlar içerir (Muhtemelen kullanılmıyor).
*   [`libgame`](libgame_Referans.md): Oyunun temel mekaniklerini, karakter yönetimini, eşya sistemini, görevleri (quest) ve diğer oyun mantığını içerir.
*   [`libserverkey`](libserverkey_Referans.md): Sunucu anahtarı doğrulaması ve ilgili kriptografik işlemleri yönetir.
*   `common`: Sunucu ve istemci tarafından paylaşılan ortak kodları içerir.
*   `game`: Oyun sunucusunun ana kodunu içerir (`libgame`'i kullanır).
*   [`db`](Source/db/db_Referans.md): Veritabanı sunucusunun kodunu içerir (`libsql` ve `libthecore`'u kullanır). 

## `game/src/` Klasörünün Belgelenmesi

Oyun sunucusunun ana mantığını içeren `game/src` klasöründeki C++ kaynak dosyaları (.cpp) ve başlık dosyaları (.h) aşağıda kategorize edilmiş ve belgelenmiştir:

*   **Çekirdek Mekanikleri:** Oyunun temel sistemleri ve işleyişi.
    *   [`game_Core_Referans.md`](Source/game/game_Core_Referans.md)
    *   [`game_Core_Referans_Part2.md`](Source/game/game_Core_Referans_Part2.md)
*   **Oyun Özellikleri:** Belirli oyun sistemleri ve özellikleri.
    *   [`game_Features_Referans.md`](Source/game/game_Features_Referans.md)
    *   [`game_Features_Referans_Part2.md`](Source/game/game_Features_Referans_Part2.md)
    *   [`game_Features_Referans_Part3.md`](Source/game/game_Features_Referans_Part3.md)
*   **Görev (Quest) Sistemi:** Görev mantığı ve Lua arayüzleri.
    *   [`game_Quest_Referans.md`](Source/game/game_Quest_Referans.md)
*   **Yardımcı Bileşenler ve Araçlar:** Genel yardımcı sınıflar, fonksiyonlar ve sistemler.
    *   [`game_Utils_Referans.md`](Source/game/game_Utils_Referans.md)
    *   [`game_Utils_Referans_Part2.md`](Source/game/game_Utils_Referans_Part2.md)
    *   [`game_Utils_Referans_Part3.md`](Source/game/game_Utils_Referans_Part3.md)

---

## Yardımcı Araçlar ve Betikler

Bu bölümde, sunucu kaynak kodunun doğrudan bir parçası olmayan ancak geliştirme, yerelleştirme veya veri yönetimi gibi süreçlerde kullanılan yardımcı araçlar ve betikler belgelenmektedir.

### `Source/srcServer/Source/game/src/merge_locale_string.py`

*   **Amaç:** İki ayrı yerelleştirme metin dosyasını (genellikle bir ana dil dosyası, örneğin Korece, ve bir hedef dil dosyası, örneğin İngilizce) alıp, bunları sunucunun `locale_string.txt` dosyasında kullandığı formata uygun şekilde birleştirmek için kullanılan bir Python betiğidir.
*   **Temel İşlevler/İçerik:**
    *   **Argümanlar:** Komut satırından iki dosya adı alır: `han_file_name` (ana dil dosyası) ve `locale_file_name` (hedef/çeviri dil dosyası).
    *   **`ReadLocaleLines(fileName)` Fonksiyonu:**
        *   Verilen dosya adını açar ve tüm satırları okur.
        *   Her satırın sonundaki `\r\n` veya `\n` gibi yeni satır karakterlerini temizler.
        *   Temizlenmiş satırları bir liste olarak döndürür.
    *   **Ana İşleyiş:**
        *   Komut satırı argüman sayısını kontrol eder (program adı + 2 dosya adı olmalı).
        *   `ReadLocaleLines` fonksiyonunu kullanarak her iki giriş dosyasının içeriğini ayrı listelere (`srcList`, `dstList`) okur.
        *   `zip(srcList, dstList)` ile her iki listedeki karşılıklı satırları çiftler halinde işler.
        *   Her çift için çıktı listesine (`outList`) şu formatta üç satır ekler:
            1.  `"ana_dil_satiri";`
            2.  `"hedef_dil_satiri";`
            3.  (Boş bir satır)
        *   `outList` içeriğini `locale_string_out.txt` adlı yeni bir dosyaya, her eleman arasına yeni satır karakteri (`\n`) ekleyerek yazar.
*   **Kullanım Örneği (Komut Satırı):**
    ```bash
    python merge_locale_string.py korece_metinler.txt ingilizce_metinler.txt
    ```
*   **Çıktı:** `locale_string_out.txt` dosyası, sunucunun okuyabileceği birleşik yerelleştirme metinlerini içerir.
*   **Bağlantılı Dosyalar/Süreçler:** Bu betik doğrudan oyun sunucusu tarafından çalıştırılmaz, ancak sunucunun kullandığı `locale_string.txt` dosyasını oluşturmak/güncellemek için geliştirme aşamasında kullanılır. 