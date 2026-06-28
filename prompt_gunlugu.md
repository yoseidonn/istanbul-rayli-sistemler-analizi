# Prompt Günlüğü — İstanbul Raylı Sistemler Yolcu Analizi

**Proje:** Veri Bilimi Dönem Projesi  
**Tema:** Akıllı Şehir & Toplu Taşıma  
**Tarih:** 26-28 Haziran 2026  

Bu dosya, proje boyunca Claude Code'a yöneltilen 12 kritik prompt'u kronolojik olarak listeler. Her prompt'un hangi aşamada kullanıldığı ve ne işe yaradığı açıklanmıştır.

---

## Aşama 1: Proje Planlama ve Veri Keşfi

### Prompt 1 — Proje konsepti belirleme
> İBB Açık Veri Portalı'ndaki ulaşım veri setlerini kullanarak bir veri bilimi projesi yapmak istiyorum. Raylı sistemler yolcu verisi, istasyon envanteri, hat uzunlukları ve aktarma bilgileri var. Bu 4 veri setiyle hangi araştırma sorularını cevaplayabilirim? Bir proje planı oluştur.

**Aşama:** Proje planlama  
**Çıktı:** 5 araştırma sorusu (mevsimsellik, altyapı-talep ilişkisi, kümeleme, mekansal dağılım, hat verimliliği), PLAN.md taslağı, notebook bölüm yapısı (§1-§8).

---

### Prompt 2 — Veri setlerini indirme
> https://data.ibb.gov.tr/group/ulasim-hizmetleri?q=hat%2Fyolcu adresindeki raylı sistem veri setlerini bul ve indir. 2021-2025 arası tüm CSv'leri, istasyon bilgileri, hat uzunlukları ve aktarma bilgilerini data/ klasörüne kaydet.

**Aşama:** Veri toplama  
**Çıktı:** 5 yıllık CSV (toplam ~210 MB) + 3 XLSX dosyası data/ dizinine indirildi.

---

## Aşama 2: Veri Yükleme ve Temizleme

### Prompt 3 — Farklı formatlardaki CSv'leri yükleme
> 5 yılın CSV'si farklı delimiter, encoding ve sütun formatında: 2021/2024 virgül delimiter + tırnaklı + BOM, 2022 virgül + Latin-1 encoding, 2023/2025 noktalı virgül. Her yılı ayrı ayrı okuyup tek bir DataFrame'de birleştirir misin? 2021 ve 2024'te terminal_number sütunu var, onu gruplayıp istasyon bazına indirge.

**Aşama:** §2 Veri yükleme  
**Çıktı:** `load_terminal_based()`, `load_2022()`, `load_2023()`, `load_2025()` fonksiyonları; `standardize_columns()` ile sütun isimleri normalize edildi; 544 bin satır tek DataFrame'de birleştirildi.

---

### Prompt 4 — Koordinat düzeltme ve aykırı değer temizliği
> Bazı koordinatlar Türkçe locale yüzünden bozuk: 28.933 yerine 289.333 veya 289.343.888.889 olarak kaydedilmiş. Bu değerleri düzelten bir fonksiyon yaz. Ayrıca passage_cnt için %1-%99 persentil aralığında aykırı değer temizliği yap, passage/passanger oranı 10'dan büyük olanları filtrele.

**Aşama:** §3.2-3.3 Veri temizleme  
**Çıktı:** `fix_coordinate()` fonksiyonu (çoklu nokta, eksik değer, ValueError handling ile); persentil tabanlı filtreleme ve oran kontrolü.

---

### Prompt 5 — Hat ve istasyon ismi normalizasyonu + fuzzy merge
> Dataset A'da hat isimleri "M1-YENIKAPI-HAVALIMANI", Dataset B'de "M1" formatında. Regex ile kısa hat kodu çıkar. İstasyon isimlerini de normalize edip (büyük harf, Türkçe karakter temizliği) merge et. Eşleşmeyenler için difflib fuzzy matching kullan. Sonra hat uzunlukları ve aktarma bilgilerini de merge edip `is_transfer` binary flag'i oluştur.

**Aşama:** §3.4-3.5 Merge  
**Çıktı:** `extract_line_code()` (M1A, TF2, TCDD, TUNEL özel durumlarıyla), `normalize_name()`, fuzzy matching ile ~%48 eşleşme oranı, 4 veri seti birleştirildi, 44 aktarma istasyonu tespit edildi.

---

## Aşama 3: Keşifsel Veri Analizi (EDA)

### Prompt 6 — Eksik veri ve temel istatistikler
> Birleştirilmiş veri setinde eksik veri analizi yap. missingno matrix ile görselleştir, .describe() ile temel istatistikleri çıkar. Hangi sütunlarda ne kadar eksik var?

**Aşama:** §4.0-4.1 EDA  
**Çıktı:** missingno matrix, eksik veri raporu, sayısal sütunlar için describe() özeti.

---

### Prompt 7 — Mevsimsellik ve zamansal analiz (RQ1)
> 5 araştırma sorusundan ilki mevsimsellik ve haftanın günü etkisi. Aylık trend lineplot'u hat tipine göre renklendir, gün×ay heatmap'i yap, haftaiçi/haftasonu boxplot karşılaştırması ekle. Aykırı değerleri gösterme. Bulguları RQ1 etiketiyle yazdır.

**Aşama:** §4.2-4.5 EDA  
**Çıktı:** 4 görselleştirme (lineplot, heatmap, barplot, boxplot); RQ1 cevabı: haftasonu %24 düşüş, yaz aylarında azalma, Metro hatları lider.

---

### Prompt 8 — Korelasyon ve altyapı-yolcu ilişkisi
> Sayısal değişkenler arası korelasyon heatmap'i çiz, istasyon büyüklüğü vs yolcu scatter plot'unu aktarma renklendirmesiyle yap. RQ2 için gözlem notunu ekle.

**Aşama:** §4.6 EDA  
**Çıktı:** Korelasyon heatmap (9 değişken), büyüklük-yolcu scatter plot (aktarma kırmızı).

---

## Aşama 4: Modelleme

### Prompt 9 — Linear Regression (RQ2)
> İstasyon altyapı özellikleri (büyüklük, yürüyen merdiven, asansör, aktarma durumu, haftasonu, ay, hat uzunluğu) ile günlük passage_cnt arasında Linear Regression kur. %80/%20 split, StandardScaler, R²/MAE/RMSE metriklerini hesapla. Katsayı analizi ve rezidü plot'u ekle.

**Aşama:** §5.1 Regresyon  
**Çıktı:** R²=0.137, MAE=5.980, RMSE=8.197; en güçlü belirleyici: is_transfer (+2.741), is_weekend (-1.554). Altyapı tek başına varyansın %13.7'sini açıklıyor.

---

### Prompt 10 — K-Means Kümeleme (RQ3)
> İstasyonları altyapı ve yolcu profiline göre kümele. İstasyon bazlı aggregation yap, log_avg_passage ekle, StandardScaler ile ölçeklendir. K=2-9 için Elbow ve Silhouette analizi yap, optimal K'yı seç. PCA ile 2B görselleştir, küme profillerini yazdır (istasyon sayısı, ort. geçiş, büyüklük, aktarma oranı).

**Aşama:** §6.1-6.3 Kümeleme  
**Çıktı:** Optimal K=6 (Silhouette=0.372), 6 küme profili: 2 istasyonlu mega hub'dan (83.212 m²) 47 istasyonlu küçük duraklara (1.196 m²). Aktarma istasyonları Küme 0 ve 3'te yoğunlaşıyor.

---

## Aşama 5: Coğrafi Analiz ve Görselleştirme

### Prompt 11 — İnteraktif harita ve geo scatter (RQ4)
> İstasyon konumlarını matplotlib scatter ile görselleştir (büyüklük ∝ yolcu, renk = hat tipi, ★ = aktarma). AYRICA Folium kütüphanesiyle interaktif bir harita oluştur: CircleMarker büyüklüğü yolcu ile orantılı, renk hat tipine göre, popup'ta istasyon adı ve günlük geçiş görünsün. Aktarma istasyonları kalın kenarlı olsun. HTML olarak kaydet.

**Aşama:** §7.1 Coğrafi analiz  
**Çıktı:** `geo_stations.png` (statik scatter), `folium_map.html` (338 istasyon, interaktif, renkli popup'lı).

---

### Prompt 12 — Hat verimliliği analizi (RQ5)
> Hat bazında toplam yolcu, istasyon sayısı ve km başına yolcu metriklerini hesapla. En verimli 15 hattı ve hat tipi bazında ortalama verimliliği barplot ile göster. RQ5 cevabını yazdır.

**Aşama:** §7.2 Hat verimliliği  
**Çıktı:** Füniküler ve tramvay hatları km başına en yüksek yoğunluğa sahip. Metro hatları toplamda lider ama km başına verimlilikte orta sırada. `line_efficiency.png`.

---

## Aşama 6: Sonuçlar ve Teslimat

### Prompt 13 — Sonuçları yorumlama ve rapor taslağı
> 5 araştırma sorusunun cevaplarını İstanbul ulaşım planlaması bağlamında yorumla. Hangi bulgular politika önerisine dönüşebilir? Sınırlamaları ve öğrenilenleri de ekle. 3-5 sayfalık bir PDF rapor için içerik üret.

**Aşama:** §8 ve rapor.pdf  
**Çıktı:** §8 sonuç bölümü (5 RQ cevabı, 5 sınırlama, 4 öğrenilen), rapor.pdf (problem, veri seti, yöntem, bulgular, sınırlamalar, öğrenilenler).

---

## Kullanılan Teknolojiler

| Araç | Kullanım Amacı |
|------|---------------|
| Claude Code (browser-agent) | İBB portalından veri setlerini keşfetme ve indirme |
| Claude Code (verify) | Notebook'u çalıştırarak hata ayıklama ve doğrulama |
| Claude Code (code-review) | Kod kalitesi ve bug tespiti (8 sorun bulundu, düzeltildi) |
| Claude Code (simplify) | Kod tekrarını azaltma, fonksiyon birleştirme |
| pandas, numpy | Veri yükleme, temizleme, transformasyon |
| matplotlib, seaborn | 11 görselleştirme |
| scikit-learn | LinearRegression, KMeans, PCA, StandardScaler |
| folium | İnteraktif coğrafi harita |
| missingno | Eksik veri matrisi |
| difflib | Fuzzy string matching (istasyon ismi eşleştirme) |
| fpdf2 | PDF rapor üretimi |
