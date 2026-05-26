# magibu-llm-tools

> LLM inference sırasında modele yetenek kazandıran modüler tool & sistem koleksiyonu.

`magibu-tools`, bir dil modelinin tek başına iyi yapamadığı ya da hiç yapamadığı işleri (kesin matematik, güncel bilgiye erişim, OCR, alana özel bilgi getirimi vb.) güvenilir biçimde yapabilmesi için ona dışarıdan bağlanan araçların toplandığı açık kaynak bir repodur. Amaç; her biri net bir arayüze sahip, bağımsız test edilebilen ve kolayca yenisi eklenebilen bir **tool ekosistemi** kurmaktır.

### Bu repo aktif geliştirme aşamasındadır. Bazı modüller kullanıma hazır, bazıları ise yol haritasında. Katkıya açığız, aşağıdaki [Katkı Sağlama](#katkı-sağlama) bölümüne göz atın.

---

## Neden tool'lar?

Bir LLM dilsel akıl yürütmede güçlüdür, ama:

- bir kelimedeki harfleri saymak gibi token seviyesindeki işlerde yanılır,
- aritmetiği yaklaşık yapar, kesin sonuç garanti etmez,
- eğitim verisinden sonra olan hiçbir şeyi bilmez,
- görüntüden metin okuyamaz,
- alana özel (örn. tıbbi) sorularda atıf veremez ve halüsinasyon riski taşır.

Bu sınırları aşmanın yolu, modele "doğru aracı doğru zamanda çağırma" yeteneği vermektir. Model bir görevin kendi başına çözülemeyeceğini fark ettiğinde ilgili tool'u çağırır, deterministik/uzman bir bileşen işi yapar, sonuç modele geri döner.

---

## Sistemler

Her sistem bağımsız bir modüldür. Aşağıdaki tablo mevcut durumu özetler:

| Sistem | Açıklama | Durum |
|---|---|---|
| **String fonksiyonları** | Token seviyesi işlemler: bir kelimede kaç `r` var, karakter sayma, ters çevirme, substring arama vb. | ✅ Hazır |
| **Matematik fonksiyonları** | Hazır matematiksel işlemler: toplama, çarpma, karekök, logaritma, üs alma gibi deterministik hesaplamalar. | ✅ Hazır |
| **Python tool** | Model gerekli gördüğünde kendisi Python scripti yazıp izole bir ortamda çalıştırabilir; çıktıyı yorumlayıp cevaba katar. | ✅ Hazır |
| **Web search** | İnternet araması. Model güncel bilgiye ihtiyaç duyduğunda soru hakkında arama yapıp sonuçları cevaba dahil eder. | ✅ Hazır |
| **Translate** | Çeviri görevleri için daha küçük, özelleşmiş bir çeviri modelini çağırır. | 📋 Planlanıyor |
| **OCR** | OCR görevleri için özelleşmiş bir modeli (ör. `glm-0.8b`) çağırarak görüntüden metin çıkarır. | 📋 Planlanıyor |
| **Sağlık RAG** | Tıbbi sorularda özel tıp RAG sistemini çağırır; sistemdeki makaleleri kullanarak **atıflı** tıbbi cevaplar üretir. | 📋 Planlanıyor |
| **Uzman model yönlendirme** | Yukarıdakilere benzer şekilde, özelleşmiş görevler için ilgili uzman modele yönlendirip ondan cevap alır. | 📋 Planlanıyor |

> Durum etiketleri: ✅ Hazır · 🚧 Geliştiriliyor · 📋 Planlanıyor

---

## Mimari

Tüm sistemler ortak bir tool arayüzü etrafında toplanır. Genel akış:

```
Kullanıcı sorusu
      │
      ▼
   ┌─────────┐    tool gerekli mi?     ┌──────────────┐
   │   LLM   │ ──────────────────────► │ Tool Router  │
   └─────────┘                         └──────┬───────┘
      ▲                                       │
      │                                       ▼
      │            tool çıktısı          ┌──────────────┐
      └────────────────────────────────  │ İlgili Tool  │
                                         │ (math / OCR  │
                                         │  / RAG ...)  │
                                         └──────────────┘
```

Her tool'un ideal olarak şunları sağlaması beklenir:

- **Net bir şema** — girdi/çıktı tipleri ve çağrı isminin tanımı.
- **Bağımsızlık** — başka bir tool'a bağımlı olmadan tek başına çalışabilme.
- **Deterministik veya izole çalışma** — yan etkileri kontrol altında (özellikle Python ve web tool'ları için sandbox).
- **Testler** — beklenen davranışı doğrulayan birim testleri.

---

## Yol Haritası

Repo'nun hedeflediği gelişim sırası:

**Kısa vade**
- [ ] Tüm "Hazır" tool'lar için ortak şema/arayüz standardının netleştirilmesi
- [ ] Translate ve OCR uzman modellerinin entegrasyonu ve örnek çağrıları
- [ ] Sağlık RAG: makale indeksleme hattı + atıf formatının standardı

**Orta vade**
- [ ] Tool router'ın değerlendirme (eval) altyapısı: hangi soru hangi tool'u tetiklemeli
- [ ] Genel "uzman model yönlendirme" katmanı

**Uzun vade**
- [ ] Tool kullanım metriklerinin loglanması ve gözlemlenebilirlik
- [ ] Çoklu tool zincirleme (bir cevap için birden çok tool'un sıralı kullanımı)

---

## Başlangıç

> Not: Kurulum adımları geliştirme ilerledikçe netleşecektir. Aşağıdaki şablonu projenin gerçek yapısına göre güncelleyin.

```bash
# Repo'yu klonla
git clone https://github.com/magibu-ai/magibu-llm-tools.git
cd magibu-llm-tools

# Bağımlılıkları kur
# (paket yöneticinize göre düzenleyin)
```

---

## Katkı Sağlama

Katkılar çok değerli. Yeni bir tool eklemek genellikle en kolay başlangıç noktasıdır.

**Yeni bir tool eklerken:**

1. `tools/` altında tool'unuz için bir modül oluşturun.
2. Ortak tool arayüzünü uygulayın (girdi şeması, çıktı şeması, çağrı ismi/açıklaması).
3. En az birkaç birim testi yazın — özellikle uç durumlar için.
4. README'deki sistem tablosuna tool'unuzu ve durumunu ekleyin.
5. Kısa bir kullanım örneği ekleyin.

**Genel kurallar:**

- Önce bir issue açıp ne üzerinde çalışmak istediğinizi belirtin; böylece iş çakışmasını önleriz.
- Bir branch açın (`feat/tool-adi`), değişikliklerinizi yapın, PR gönderin.
- PR açıklamasında neyi neden değiştirdiğinizi net yazın.

**İlk katkı için iyi yerler:** Yukarıdaki yol haritasında `[ ]` ile işaretli, henüz başlanmamış maddeler ya da mevcut tool'lara test/doküman eklemek.

---

*Bu repo magibu LLM inference altyapısının bir parçası olarak geliştirilmektedir.*
