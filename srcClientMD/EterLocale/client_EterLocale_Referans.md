# EterLocale Referans Kılavuzu

Bu belge, Metin2 istemcisinin `EterLocale` klasöründeki bileşenleri ve dosyaları detaylandırmaktadır. `EterLocale`, genellikle yerelleştirme, dil desteği, karakter kodlaması ve bölgeye özgü yapılandırmalarla ilgili işlevleri içerir.

---

## Dosya Bazlı Detaylandırma 

### `Arabic.h` ve `Arabic.cpp`

*   **Amaç:** Bu dosyalar, Arapça metinlerin doğru bir şekilde işlenmesi ve karakterlerin Unicode standartlarına uygun olarak doğru bağlamsal formlarda (yalın, başlangıç, orta, son) gösterilmesi için gerekli fonksiyonları içerir. Ayrıca, bazı özel sembollerin Arapça metin akışına uygun olarak dönüştürülmesini de ele alır.

*   **Temel Fonksiyonlar ve Mantık:**
    *   **`Arabic_IsInSpace(wchar_t code)`**: Verilen karakterin boşluk veya tab karakteri olup olmadığını kontrol eder.
    *   **`Arabic_IsInSymbol(wchar_t code)`**: Verilen karakterin genel bir sembol aralığında olup olmadığını kontrol eder.
    *   **`Arabic_IsInPresentation(wchar_t code)`**: Verilen karakterin Arapça sunum formu (şekillendirilmiş karakter) olup olmadığını kontrol eder (Unicode aralıkları 0xFB50-0xFDFF ve 0xFE70-0xFEFF).
    *   **`Arabic_HasPresentation(wchar_t* codes, int last)`**: Verilen bir string'in sonunda (boşlukları atlayarak) zaten Arapça sunum formu içerip içermediğini kontrol eder.
    *   **`Arabic_GetComposition(wchar_t cur, wchar_t next, ARABIC_FORM_TYPE pos)`**: Lam-Elif ( Örneğin ل + ا = لا ) gibi özel Arapça ligatürlerini (birleşik harf formları) oluşturur. Verilen mevcut (`cur`) ve sonraki (`next`) harfe ve istenen forma (`pos`) göre doğru ligatür karakterini döndürür.
    *   **`Arabic_GetMap(wchar_t code, ARABIC_FORM_TYPE pos)`**: Arapça bir karakterin (`code`) bağlama göre alması gereken sunum formunu (yalın, başlangıç, orta, son) döndürür. Bu, her bir temel Arapça harfi için tanımlanmış statik haritalama dizileri üzerinden yapılır.
    *   **`Arabic_IsInMap(wchar_t code)`**: Verilen karakterin `Arabic_GetMap` içinde tanımlı bir temel Arapça harfi olup olmadığını kontrol eder (Unicode 0x0621-0x064A aralığı).
    *   **`Arabic_IsInComposing(wchar_t code)`**: Verilen karakterin bir hareke (örn: fatha, damma) veya birleştirici olmayan başka bir işaret olup olmadığını kontrol eder.
    *   **`Arabic_IsNext(wchar_t code)`**: Verilen karakterin Tatweel (U+0640, Arapça uzatma karakteri) olup olmadığını kontrol eder.
    *   **`Arabic_IsComb1(wchar_t code)`, `Arabic_IsComb2(wchar_t code)`**: `Arabic_MakeShape` içinde Lam-Elif ligatürlerini tespit etmek için yardımcı fonksiyonlardır (`IsComb1` Lam'ı, `IsComb2` ise Elif varyantlarını kontrol eder).
    *   **`Arabic_MakeShape(wchar_t* src, size_t srcLen, wchar_t* dst, size_t dstLen)`**: Bu, çekirdek fonksiyondur. Temel Arapça karakterler içeren bir kaynak string'i (`src`) alır ve karakterlerin komşularına göre doğru sunum formlarını (yalın, başlangıç, orta, son ve ligatürler) bularak hedef string'e (`dst`) yazar.
        *   Her bir Arapça karakter için, bir önceki ve bir sonraki temel (hareke olmayan) Arapça karaktere bakar.
        *   Bu komşulara göre karakterin hangi formda (yalın, başlangıç, orta, son) olması gerektiğini belirler.
        *   Lam-Elif kombinasyonlarını `Arabic_GetComposition` ile özel olarak ele alır.
        *   Diğer karakterler için `Arabic_GetMap` kullanarak doğru sunum formunu seçer.
        *   Arapça olmayan karakterler doğrudan kopyalanır.
    *   **`Arabic_ConvSymbol(wchar_t c)`**: Parantez `()` gibi bazı çiftli sembolleri, sağdan sola yazım düzenine uygun olarak ters çevirir (örn: `(` -> `)`).
    *   **`Arabic_ConvEnglishModeSymbol(wchar_t code)`**: Başlık dosyasında (`Arabic.h`) bildirimi olmasına rağmen, sağlanan `Arabic.cpp` içeriğinde implementasyonu bulunmamaktadır. Muhtemelen Arapça metin içinde İngilizce modundayken bazı sembollerin özel dönüşümü için tasarlanmıştır.

*   **`ARABIC_CODE` Enum'u**: Temel Arapça karakterlerin Unicode aralığını tanımlar (0x0621 - 0x064A).
*   **`ARABIC_FORM_TYPE` Enum'u**: Bir karakterin alabileceği bağlamsal formları tanımlar: `DEBUG_CODE`, `ISOLATED` (yalın), `INITIAL` (başlangıç), `MEDIAL` (orta), `FINAL` (son).

*   **Kullanım Amacı:**
    *   Metin2 istemcisinde Arapça metinlerin doğru şekilde render edilebilmesi için temel bir altyapı sağlar. Kullanıcıdan alınan veya oyun dosyalarından okunan temel Arapça karakter dizilerini, ekranda düzgün ve okunabilir bir şekilde gösterilebilecek sunum formlarına dönüştürür.
    *   Özellikle `EterLib` içindeki `CGraphicTextInstance` gibi metin işleme sınıfları, `WM_CHAR` veya IME (Input Method Editor) üzerinden gelen Arapça metinleri işlerken bu fonksiyonları kullanabilir. 

### `CodePageId.h`

*   **Amaç:** Bu başlık dosyası, Windows işletim sistemlerinde kullanılan çeşitli karakter kod sayfaları (code page) için standart sayısal tanımlayıcıları, okunabilir makro isimleriyle eşleştirir. Bu sayede, kod içinde farklı dil ve karakter setlerini temsil eden kod sayfalarına atıfta bulunurken sayısal değerler yerine `CP_TURKISH`, `CP_JAPANESE` gibi anlaşılır isimler kullanılabilir.

*   **İçerik:**
    *   Tayca (CP_874)
    *   Japonca (CP_932 - Shift-JIS)
    *   Basitleştirilmiş Çince (CP_936)
    *   Korece (Hangeul - CP_949)
    *   Geleneksel Çince (CP_950)
    *   Orta ve Doğu Avrupa (CP_1250)
    *   Kiril (CP_1251)
    *   Latin (CP_1252)
    *   Yunanca (CP_1253)
    *   Türkçe (CP_1254)
    *   İbranice (CP_1255)
    *   Arapça (CP_1256)
    *   Baltık (CP_1257)
    *   Vietnamca (CP_1258)
    *   UTF-8 (CP_65001)

*   **Kullanım Amacı:**
    *   İstemcinin farklı dillerdeki metinleri doğru bir şekilde işlemesi, yorumlaması ve dönüştürmesi gereken yerlerde (örneğin, dosya okuma/yazma, ağ iletişimi, IME entegrasyonu) kullanılır.
    *   `EterLib` veya diğer kütüphanelerdeki metin işleme fonksiyonları, hangi kod sayfasının kullanılacağını belirlemek için bu sabitlere başvurabilir.

### `Japanese.h` ve `Japanese.cpp`

*   **Amaç:** Bu dosyalar, Japonca metinlerin Shift-JIS (CP932) karakter kodlamasıyla doğru bir şekilde işlenmesi için yardımcı fonksiyonlar sağlar. Shift-JIS, Japonca karakterleri temsil etmek için kullanılan, değişken uzunlukta bayt dizilerinden oluşan bir kodlama standardıdır.

*   **Temel Fonksiyonlar ve Mantık:**
    *   **`ShiftJIS_IsLeadByte(const char chByte)`**: Verilen bir baytın Shift-JIS standardında çok baytlı bir karakterin öncü (ilk) baytı olup olmadığını kontrol eder. Öncu baytlar belirli aralıklardadır (0x81-0x9F veya 0xE0-0xFC).
    *   **`ShiftJIS_IsTrailByte(const char chByte)`**: Verilen bir baytın Shift-JIS standardında çok baytlı bir karakterin takip eden (ikinci) baytı olup olmadığını kontrol eder. Takip eden baytlar da belirli aralıklardadır (0x40-0x7E veya 0x80-0xFC).
    *   **`ShiftJIS_StringCompareCI(LPCSTR szStringLeft, LPCSTR szStringRight, size_t sizeLength)`**: İki Shift-JIS kodlanmış string'i, belirtilen uzunluk kadar, büyük/küçük harf duyarsız bir şekilde karşılaştırır.
        *   Karşılaştırma sırasında her bir karakterin öncü bayt olup olmadığını kontrol eder.
        *   Eğer öncü bayt ise, bir sonraki baytı da alarak tam bir Shift-JIS karakteri (2 bayt) oluşturur.
        *   Tek baytlı karakterler (genellikle ASCII) için `tolower()` fonksiyonu ile küçük harfe çevirerek karşılaştırma yapar. Çok baytlı Japonca karakterler için doğrudan ikili karşılaştırma yapılır (Japonca karakterlerin büyük/küçük harf dönüşümü bu fonksiyonda ele alınmaz).
        *   String sonu veya belirtilen uzunluğa ulaşıldığında özel durumları ele alır.

*   **Kullanım Amacı:**
    *   İstemcinin Japonca yerelleştirmesinde, kullanıcı girdilerini, oyun içi metinleri veya yapılandırma dosyalarını Shift-JIS formatında doğru bir şekilde işlemesi gerektiğinde bu fonksiyonlar kullanılır.
    *   Örneğin, Japonca kullanıcı adlarının veya sohbet mesajlarının büyük/küçük harf duyarsız karşılaştırılması veya sıralanması gibi senaryolarda `ShiftJIS_StringCompareCI` önemli bir rol oynar.
    *   Metin ayrıştırma veya işleme sırasında bir karakterin tam olarak okunabilmesi için `ShiftJIS_IsLeadByte` ve `ShiftJIS_IsTrailByte` fonksiyonları, bir baytın tek başına mı yoksa bir çiftin parçası mı olduğunu belirlemede kullanılır. 

### `StdAfx.h` ve `StdAfx.cpp`

*   **Amaç:** Bu dosyalar, Microsoft Visual C++ projelerinde yaygın olarak kullanılan ön derlenmiş başlık (precompiled header) mekanizmasının bir parçasıdır. Amaçları, sık kullanılan ve nadiren değişen standart başlık dosyalarını ve projeye özgü bazı temel başlıkları tek bir yerde toplayarak derleme sürelerini kısaltmaktır.

*   **`StdAfx.h` İçeriği:**
    *   `#pragma once`: Bu başlık dosyasının derleme birimi başına yalnızca bir kez dahil edilmesini sağlar.
    *   `#define WIN32_LEAN_AND_MEAN`: Windows başlıklarından (`windows.h`) daha az kullanılan API'lerin hariç tutulmasını sağlayarak derleme boyutunu ve süresini azaltır.
    *   `#include <windows.h>`: Temel Windows API fonksiyonları ve veri türleri için gereklidir.
    *   `#include <assert.h>`: `assert()` makrosunu kullanarak çalışma zamanı kontrolleri (iddialar) eklemek için kullanılır.
    *   `#include "CodePageId.h"`: Proje içindeki kod sayfası tanımlayıcılarını (`CP_TURKISH` vb.) bu kütüphanenin tümüne dahil eder.
    *   `#include "../UserInterface/Locale_inc.h"`: Bir üst dizindeki `UserInterface` klasöründe bulunan ve muhtemelen yerelleştirmeyle ilgili genel tanımlamalar veya ayarlar içeren `Locale_inc.h` dosyasını dahil eder. Bu, `EterLocale` kütüphanesinin `UserInterface` kütüphanesiyle bir bağımlılığı veya ortak yapılandırması olduğunu gösterir.

*   **`StdAfx.cpp` İçeriği:**
    *   Genellikle sadece `#include "stdafx.h"` satırını içerir. Bu dosyanın temel amacı, ön derlenmiş başlık (`.pch`) dosyasının oluşturulmasını tetiklemektir. Derleyici bu `.cpp` dosyasını işlerken, `StdAfx.h` içinde listelenen tüm başlıkları derler ve bir `.pch` dosyasına kaydeder. Daha sonra projedeki diğer `.cpp` dosyaları derlenirken, eğer onlar da ilk satırda `StdAfx.h`'ı içeriyorsa, derleyici bu önceden derlenmiş bilgiyi kullanarak zamandan tasarruf eder.

*   **Kullanım Amacı:**
    *   `EterLocale` kütüphanesindeki tüm kaynak dosyalarının ortak olarak ihtiyaç duyduğu temel Windows API'lerini, hata ayıklama araçlarını ve projeye özgü yerelleştirme tanımlarını (kod sayfaları, `Locale_inc.h`) tek bir yerden sağlayarak kod tekrarını azaltır.
    *   Ön derlenmiş başlıklar sayesinde, `EterLocale` kütüphanesinin genel derleme süresini iyileştirir. 

### `StringCodec_Vietnamese.h` ve `StringCodec_Vietnamese.cpp`

*   **Amaç:** Bu dosyalar, Vietnamca metinlerin Windows-1258 (CP1258) karakter kodlaması ile Unicode (UTF-16 veya `wchar_t`) arasında çift yönlü dönüşümünü (decode/encode) gerçekleştirmek için gerekli fonksiyonları ve veri tablolarını sağlar. Vietnamca, Latin alfabesini temel almakla birlikte, harflere eklenen çok sayıda ton işareti (aksan) nedeniyle özel bir işleme gerektirir.

*   **Temel Veri Yapıları ve Fonksiyonlar:**
    *   **`cp1258_to_unicode[256]` (statik `wchar_t` dizisi):** Windows-1258 kod sayfasındaki her bir 8-bit karakterin karşılık geldiği Unicode (`wchar_t`) değerini içeren bir arama tablosudur. Bu tablo, CP1258'den Unicode'a doğrudan dönüşüm için kullanılır. Tabloda, standart ASCII karakterlerinin yanı sıra Vietnamcaya özgü harfler (örneğin, `Ă`, `Â`, `Đ`, `Ê`, `Ô`, `Ơ`, `Ư`) ve bazı Unicode ton işaretleri (0x0300, 0x0301, 0x0309, 0x0303, 0x0323) bulunur.
    *   **`cp1258_composed_table[][5]` (statik `wchar_t` dizisi):** Bu tablo, Vietnamca temel harfler ile beş farklı ton işaretinin birleşimi sonucu oluşan önceden birleştirilmiş (precomposed) Unicode karakterlerini içerir. `ComposeTone` fonksiyonu bu tabloyu kullanarak bir temel harf ve bir ton işaretinden doğru birleşik Unicode karakterini bulur.
        *   Satırlar: Temel harfleri temsil eder (A, a, Ă, ă, Â, â, E, e, Ê, ê, I, i, O, o, Ô, ô, Ơ, ơ, U, u, Ư, ư, Y, y).
        *   Sütunlar: Beş ana tonu temsil eder (keskin, düşük, kanca, tilde, nokta altı).
    *   **`IsTone(wchar_t tone)` (statik bool fonksiyonu):** Verilen bir Unicode karakterinin Vietnamca ton işaretlerinden biri olup olmadığını kontrol eder (0x0300, 0x0301, 0x0309, 0x0303, 0x0323).
    *   **`ComposeTone(wchar_t prev, wchar_t tone)` (statik `wchar_t` fonksiyonu):** Bir önceki karakter (`prev` - genellikle bir temel harf) ile bir ton işaretini (`tone`) birleştirerek önceden birleştirilmiş (precomposed) tek bir Unicode karakteri oluşturur. `cp1258_composed_table` kullanılır.
    *   **`EL_String_Decode_Vietnamese(const char* multi, int multiLen, wchar_t* wide, int wideLen)` (int fonksiyonu):**
        *   Windows-1258 kodlamasındaki çok baytlı bir string'i (`multi`) Unicode (`wchar_t`) string'ine (`wide`) dönüştürür (decode eder).
        *   CP1258 string'ini karakter karakter işler.
        *   Her bir CP1258 karakterini `cp1258_to_unicode` tablosunu kullanarak Unicode'a çevirir.
        *   Eğer bir sonraki CP1258 karakteri bir ton işaretine karşılık geliyorsa (`IsTone` ile kontrol edilir), bu ton işaretini bir önceki temel harfle `ComposeTone` kullanarak birleştirir.
        *   Ton olmayan karakterler veya birleştirme sonucu oluşan karakterler hedef `wide` tamponuna yazılır.
        *   Yazılan `wchar_t` sayısını döndürür.
    *   **`DecomposeLetter(wchar_t input, char* letter)` (statik bool fonksiyonu):** Verilen bir Unicode Vietnamca karakterini (tonlu veya tonsuz) temel CP1258 harfine ayrıştırır. Örneğin, `L'Á'` (0x00c1) veya `L'À'` (0x00c0) karakterlerini CP1258'deki `'A'` harfine dönüştürür. Başarılı olursa `true` döner ve `*letter`'a CP1258 karakterini yazar.
    *   **`DecomposeTone(wchar_t input, char* tone)` (statik bool fonksiyonu):** Verilen bir Unicode Vietnamca karakterinden (eğer tonluysa) CP1258'deki karşılık gelen ton işaretini ayrıştırır. Örneğin, `L'Á'` (0x00c1) karakterinden keskin ton işaretine karşılık gelen CP1258 karakterini (`(char)0xec`) çıkarır. Başarılı olursa `true` döner ve `*tone`'a CP1258 ton karakterini yazar.
    *   **`EL_String_Encode_Vietnamese(const wchar_t* wide, int wideLen, char* multi, int multiLen)` (int fonksiyonu):**
        *   Unicode (`wchar_t`) bir string'i (`wide`) Windows-1258 kodlamasındaki çok baytlı bir string'e (`multi`) dönüştürür (encode eder).
        *   Unicode string'ini karakter karakter işler.
        *   Her bir Unicode karakteri için önce `DecomposeLetter` ile temel CP1258 harfini alır ve `multi` tamponuna yazar.
        *   Ardından `DecomposeTone` ile (eğer varsa) CP1258 ton işaretini alır ve `multi` tamponuna yazar.
        *   Bu, Unicode'daki önceden birleştirilmiş tonlu harfleri, CP1258'de bir temel harf ve ardından gelen bir ton işareti (iki bayt) olarak ayırır.
        *   Yazılan `char` sayısını döndürür.

*   **Kullanım Amacı:**
    *   İstemcinin Vietnamca yerelleştirmesi için temel karakter kodlama dönüşümlerini sağlar.
    *   Oyun dosyalarından okunan veya ağ üzerinden gelen CP1258 formatındaki Vietnamca metinlerin, istemci içinde Unicode olarak işlenebilmesi için `EL_String_Decode_Vietnamese` kullanılır.
    *   Kullanıcı tarafından girilen veya istemci tarafından oluşturulan Unicode Vietnamca metinlerin, CP1258 formatında (örneğin, log dosyalarına yazmak veya eski sistemlerle uyumluluk için) kaydedilmesi gerektiğinde `EL_String_Encode_Vietnamese` kullanılır.
    *   Bu kodek, Vietnamcanın karmaşık ton yapısını ve Windows-1258 kodlamasının özelliklerini dikkate alarak doğru metin dönüşümünü hedefler. 

### `StringCodec.h` ve `StringCodec.cpp`

*   **Amaç:** Bu dosyalar, Unicode (geniş karakter - `wchar_t`) string'ler ile çok baytlı (multibyte) string'ler arasında dönüşüm yapmak için kullanılan standart Windows API fonksiyonları olan `WideCharToMultiByte` ve `MultiByteToWideChar` için bir sarmalayıcı (wrapper) katmanı sunar. Bu sarmalayıcıların temel amacı, belirli kod sayfaları (bu durumda Vietnamca - `CP_1258`) için özel dönüşüm mantığı ekleyebilmektir.

*   **Sarmalanan Fonksiyonlar ve Özel Davranış:**
    *   **`Ymir_WideCharToMultiByte(UINT CodePage, ..., LPBOOL lpUsedDefaultChar)`**:
        *   Standart `WideCharToMultiByte` fonksiyonu ile aynı parametreleri alır.
        *   Eğer `CodePage` parametresi `CP_1258` (Vietnamca) ise, dönüşüm için `StringCodec_Vietnamese.h` içinde tanımlı olan `EL_String_Encode_Vietnamese` fonksiyonunu çağırır.
        *   Diğer tüm `CodePage` değerleri için doğrudan Windows API'si olan `WideCharToMultiByte` fonksiyonunu çağırır.
    *   **`Ymir_MultiByteToWideChar(UINT CodePage, ..., int cchWideChar)`**:
        *   Standart `MultiByteToWideChar` fonksiyonu ile aynı parametreleri alır.
        *   Eğer `CodePage` parametresi `CP_1258` (Vietnamca) ise, dönüşüm için `StringCodec_Vietnamese.h` içinde tanımlı olan `EL_String_Decode_Vietnamese` fonksiyonunu çağırır.
        *   Diğer tüm `CodePage` değerleri için doğrudan Windows API'si olan `MultiByteToWideChar` fonksiyonunu çağırır.

*   **Kullanım Amacı:**
    *   İstemci genelinde karakter kodlama dönüşümleri için merkezi bir nokta oluşturur.
    *   Windows'un standart dönüşüm fonksiyonlarını kullanırken, Vietnamca gibi özel işlem gerektiren diller için (`CP_1258`) kendi özel kodeklerini (`StringCodec_Vietnamese.cpp` içindeki fonksiyonlar) şeffaf bir şekilde devreye sokar.
    *   Bu yaklaşım, kodun farklı yerlerinde `if (CodePage == CP_1258)` gibi kontroller yapma ihtiyacını ortadan kaldırır ve yerelleştirme mantığını daha düzenli hale getirir.
    *   Gelecekte başka diller için de özel kodekler eklenmesi gerekirse, bu sarmalayıcı fonksiyonlar kolayca genişletilebilir. 