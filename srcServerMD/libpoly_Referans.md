# libpoly Referans Kılavuzu

Bu dosya, `srcServer/Source/libpoly` kütüphanesinin amacını, içerdiği temel bileşenleri ve fonksiyonları açıklamaktadır.

`libpoly`, muhtemelen sembolik hesaplama, polinom işlemleri veya benzeri matematiksel görevler için tasarlanmış bir kütüphanedir.

## Header Dosyaları

### `Base.h`

*   **Amaç:** Kütüphanedeki diğer matematiksel nesne sınıfları için temel sınıf (`CBase`) ve tür tanımlayıcı sabitleri (`MID_*`) tanımlar.
*   **İçerik:**
    *   **`MID_*` Sabitleri:** Nesne türlerini (Bilinmeyen, Sayı, Değişken, Sembol ve Sayı alt türleri: Tamsayı, Karekök, Kesir) tanımlamak için kullanılır.
    *   **`CBase` Sınıfı:**
        *   `id`: Nesnenin türünü (`MID_*`) tutar.
        *   `isSymbol()`, `isVar()`, `isNumber()`: Tür kontrolü için yardımcı metodlar.
        *   Sanal Yıkıcı (`virtual ~CBase()`): Bu sınıfın kalıtım için temel olarak tasarlandığını gösterir.

### `Constants.h`

*   **Amaç:** Matematiksel ifadeleri ayrıştırma ve değerlendirme süreçlerinde kullanılan çeşitli sabitleri (operatörler, fonksiyonlar, token türleri) tanımlar.
*   **Tanımlanan Sabit Türleri:**
    *   Temel Tokenler/Durumlar (`NONE`, `ROOT`, `NUM`, `ID`, `EOS`).
    *   Aritmetik Operatörler (`MUL`, `PLU`, `POW`, `MIN`, `DIV`, `MOD`).
    *   Parantezler (`OPEN`, `CLOSE`).
    *   Trigonometrik Fonksiyonlar (`COS`, `SIN`, `TAN`, `CSC`, `SEC`, `COT`).
    *   Logaritmik/Üstel Fonksiyonlar (`EXP`, `LOG`, `LN`, `LOG10`).
    *   Diğer Matematiksel Fonksiyonlar (`ABS`, `MINF`, `MAXF`, `IRAND`, `FRAND`, `FLOOR`, `SIGN`).
    *   Özel Sabitler (`PI`).
    *   Yapılandırma (`MAXSTACK` - Muhtemelen ifade değerlendirme yığını boyutu).
*   **İma Ettiği İşlevsellik:** Kütüphanenin metin tabanlı matematiksel ifadeleri işleyebildiğini gösterir.

### `Poly.h`

*   **Amaç:** Matematiksel ifadeleri metin olarak alıp ayrıştıran, değişken değerlerini yöneten ve ifadeyi değerlendirerek sonucunu hesaplayan merkezi `CPoly` sınıfını tanımlar.
*   **`CPoly` Sınıfı Özellikleri:**
    *   **Public Arayüz:**
        *   `Analyze()`: İfadeyi ayrıştırır.
        *   `Eval()`: Ayrıştırılmış ifadeyi değerlendirir.
        *   `SetStr()`: İşlenecek ifade metnini ayarlar.
        *   `SetVar()`, `GetVar()`: Değişken değerlerini ayarlama ve okuma.
        *   `Clear()`: İç durumu sıfırlama.
    *   **Dahili Mekanizma (Protected):**
        *   Tipik bir recursive descent parser yapısını kullanır (`lexan`, `expr`, `term`, `factor`, `expo`, `match`, `emit`, `error`).
        *   Sembol (değişken) yönetimi için fonksiyonlar (`insert`, `find`).
        *   Rastgele sayı üretimi (`my_irandom`, `my_frandom`).
    *   **Veri Depolama (Protected):**
        *   Ayrıştırılmış token'lar (`tokenBase`), sayısal değerler (`numBase`) ve sembol tablosu (`lSymbol`, `SymbolIndex`) için vektörler kullanır.
        *   Ayrıştırıcının durumunu (`iLookAhead`, `ErrorOccur` vb.) takip eder.
*   **Bağımlılıklar:** `SymTable.h`, `<string>`, `<vector>`, `<list>` (List'in yorumda belirtilmesi ilginç).

### `Symbol.h`

*   **Amaç:** İfade içindeki sembolleri (özellikle operatörler ve parantezler) temsil etmek için kullanılan `CSymbol` sınıfını ve ilgili sabitleri tanımlar.
*   **İçerik:**
    *   **`ST_*` Sabitleri:** Sembollerin (operatörler, parantezler) türünü veya önceliğini tanımlar (`ST_PLUS`, `ST_MULTIPLY` vb.). Değerler işlem önceliği ile ilişkili olabilir.
    *   **`SY_*` Sabitleri:** Operatör ve parantezlerin karakter karşılıklarını (`+`, `*`, `(` vb.) tanımlar.
    *   **`CSymbol` Sınıfı (`CBase`'den türemiştir):**
        *   `iType`: Sembolün türünü (`ST_*`) tutar.
        *   `issymbol()`: Bir karakterin sembol olup olmadığını kontrol eder (statik metod).
        *   `SetType()`, `GetType()`: Sembol türünü yönetir.
        *   `Equal()`, `Less()`: Sembolleri karşılaştırır (muhtemelen işlem önceliği için).
*   **Kullanım Alanı:** `CPoly` sınıfındaki `lSymbol` vektörü, bu türdeki nesneleri tutarak ifade içindeki değişkenleri ve değerlerini saklar.

### `SymTable.h`

*   **Amaç:** Sembol tablosundaki tek bir girdiyi (genellikle bir değişkeni) temsil eden `CSymTable` sınıfını tanımlar.
*   **`CSymTable` Sınıfı:**
    *   **Üyeler:**
        *   `dVal`: Değişkenin sayısal değeri (`double`).
        *   `token`: Değişkenin token türü (örn: `ID`).
        *   `strlex`: Değişkenin adı (lexem, `string`).
    *   **Metodlar:** Kurucu (`CSymTable`) ve sanal yıkıcı (`~CSymTable`).
*   **Kullanım Alanı:** `CPoly` sınıfındaki `lSymbol` vektörü, bu türdeki nesneleri tutarak ifade içindeki değişkenleri ve değerlerini saklar.

## Kaynak Kod Dosyaları

### `Base.cc`

*   **Amaç:** `Base.h`'de bildirilen `CBase` sınıfının temel metodlarını implemente eder.
*   **İçerik:**
    *   Kurucu (`CBase()`): `id`'yi 0 (`MID_UNKNOWN`) olarak başlatır.
    *   Yıkıcı (`~CBase()`): Boştur.
    *   Tür Kontrol Metodları (`isNumber`, `isVar`, `isSymbol`): `id` üyesi ile ilgili `MID_*` sabitini bitwise AND (`&`) işlemi kullanarak kontrol eder.

### `main.cc`

*   **Amaç:** `libpoly` kütüphanesini test etmek veya komut satırından basit bir hesaplayıcı olarak kullanmak için bir `main` giriş noktası sağlar.
*   **İşlev:**
    1.  Rastgele sayı üretecini başlatır.
    2.  Komut satırından iki argüman alır: işlenecek matematiksel ifade ve 'b' değişkeninin değeri.
    3.  `CPoly` sınıfını kullanarak ifadeyi analiz eder.
    4.  'b' ve 'k' (sabit 20) değişkenlerini ayarlar.
    5.  İfadeyi değerlendirir ve sonucu ekrana yazdırır.
*   **Not:** Bu dosya, kütüphanenin çekirdek işlevselliğinin bir parçası değildir, daha çok bir test/yardımcı programdır.

### `Poly.cc`

*   **Amaç:** `CPoly` sınıfının ana işlevselliğini, yani matematiksel ifadelerin ayrıştırılmasını (parsing), tokenizasyonunu (lexing) ve değerlendirilmesini (evaluation) implemente eder.
*   **Ana Bileşenler ve İşleyiş:**
    *   **Değerlendirme (`Eval`):**
        *   Yığın tabanlı bir hesap makinesi (stack machine) mantığıyla çalışır.
        *   `Analyze` tarafından oluşturulan `tokenBase` (işlemler/operan türleri) ve `numBase` (sayısal değerler) vektörlerini işler.
        *   Operatörleri ve fonksiyonları (`Constants.h`'dekiler) `cmath` kütüphanesi ve temel aritmetik kullanarak uygular.
        *   Sıfıra bölme gibi temel hataları kontrol eder.
    *   **Analiz/Ayrıştırma (`Analyze`, `expr`, `term`, `factor`, `expo`):**
        *   Recursive descent parser tekniğini kullanır.
        *   İşlem önceliğini (üs alma > çarpma/bölme > toplama/çıkarma) ve parantezleri yönetir.
        *   Fonksiyon çağrılarını (örn: `sin(x)`, `log(a, b)`) tanır.
        *   Sonuç olarak `Eval` fonksiyonunun işleyebileceği bir ara gösterimi (`tokenBase`, `numBase`, `SymbolIndex`) oluşturur.
    *   **Tokenizasyon (`lexan`):**
        *   İfade metnini okuyarak sayılar, değişkenler/fonksiyon adları ve operatörler/parantezler gibi token'ları tanır.
        *   Tanımlayıcıları (`isalpha` ile başlayan) sembol tablosunda (`lSymbol`) arar veya ekler (`find`, `insert`).
    *   **Sembol Tablosu Yönetimi (`find`, `insert`, `init`, `SetVar`, `GetVar`):**
        *   `lSymbol`: `CSymTable` nesnelerini (değişken adı, değeri, token türü) tutan ana vektör.
        *   `SymbolIndex`: `lSymbol`'daki nesnelere işaret eden ve değişken adına göre sıralı tutulan bir index vektörü.
        *   `find`: `SymbolIndex` üzerinde binary search yaparak değişken arar.
        *   `insert`: Yeni değişkeni `lSymbol`'a ekler ve `SymbolIndex`'i sıralı tutacak şekilde günceller.
        *   `init`: Ön tanımlı fonksiyonları (`sin`, `cos`, `log`, `min`, `max` vb.) ve sabitleri (`pi`, `e`) sembol tablosuna ekler.
    *   **Yardımcı Fonksiyonlar:** `Clear` (kaynak temizleme), `error` (hata işaretleme), `my_irandom`, `my_frandom` (rastgele sayılar).
*   **Yorum Satırları:** Aktif olmayan sembolik türev alma (`Diff`) ve MFC bağımlı olabilecek `AddSymTable` fonksiyonu kodlarını içerir.
*   **Bağımlılıklar:** `Poly.h`, `Constants.h`, `cmath`, `cctype`, `cstdlib`.

### `Symbol.cc`

*   **Amaç:** `Symbol.h`'de bildirilen `CSymbol` sınıfının metodlarını implemente eder.
*   **İçerik:**
    *   Kurucu (`CSymbol()`): `id`'yi `MID_SYMBOL`, `iType`'ı `ST_UNKNOWN` olarak başlatır.
    *   Yıkıcı (`~CSymbol()`): Boştur.
    *   Öncelik Karşılaştırma (`Equal`, `Less`): İki operatörün işlem önceliğini `iType / 10` değerlerini karşılaştırarak belirler. (`ST_*` sabitlerinin onlar basamağı öncelik seviyesini gösterir).
    *   Get/Set Metodları (`GetType`, `SetType`): `iType` üyesini yönetir.
    *   Statik Metod (`issymbol`): Verilen karakteri (`SY_*`) ilgili sembol türüne/önceliğine (`ST_*`) eşler. (Not: `SY_DIVIDE` için `ST_DIVIDE` yerine `SY_DIVIDE` döndürmesi bir hata olabilir).

### `SymTable.cc`

*   **Amaç:** `SymTable.h`'de bildirilen `CSymTable` sınıfının basit kurucu ve yıkıcı metodlarını implemente eder.
*   **İçerik:**
    *   Kurucu (`CSymTable()`): Üyeleri (`token`, `strlex`) argümanlarla ve `dVal`'i 0.0 olarak başlatır.
    *   Yıkıcı (`~CSymTable()`): Boştur.

## Diğer Dosyalar ve Klasörler

*   **`Makefile`:** Linux/FreeBSD ortamında `make` komutu ile `libpoly.a` statik kütüphanesini derlemek için kullanılan betik.
*   **`libpoly.vcxproj`:** Windows ortamında Visual Studio ile `libpoly.lib` statik kütüphanesini derlemek için kullanılan proje dosyası.
*   **`Depend`:** `Makefile` tarafından kullanılan, kaynak dosyaların bağımlılıklarını içeren dosya.
*   **`Base.o`, `Poly.o`, `Symbol.o`, `SymTable.o`:** Derlenmiş nesne kod dosyaları.
*   **`libpoly.a`:** Derlenmiş statik kütüphane arşivi (Linux/FreeBSD).
*   **`win32/` Klasörü:** Windows platformuna özgü derleme çıktılarını (`.obj`, `.lib`) veya ayarları içerebilir. 