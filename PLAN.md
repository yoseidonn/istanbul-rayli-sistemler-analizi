# 🔥 İBB Raylı Sistemler Veri Analizi — Proje Planı

**Ders:** Veri Bilimi Dönem Projesi  
**Tema:** Akıllı Şehir & Toplu Taşıma  
**Teslim Tarihi:** 3 Temmuz 2026, 12:30  
**Çalışma Şekli:** İki kişilik grup

---

## Veri Kaynakları

| # | Veri Seti | Format | Link |
|---|-----------|--------|------|
| A | Raylı Sistemler İstasyon Bazlı Yolcu ve Yolculuk Sayıları (2021-2025) | 5× CSV (`;` delimiter) | [İBB Portal](https://data.ibb.gov.tr/dataset/rayli-sistemler-istasyon-bazli-yolcu-ve-yolculuk-sayilari) |
| B | Raylı Sistemlere Ait İstasyon Bilgileri (178 istasyon) | 1× XLSX | [İBB Portal](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-istasyon-bilgileri) |
| C | Raylı Sistemlere Ait Hat Uzunluk Bilgileri | 1× XLSX | [İBB Portal](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-hat-uzunluk-bilgileri) |
| D | Raylı Sistemlere Ait Aktarma Bilgileri | 1× XLSX | [İBB Portal](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-aktarma-bilgileri) |

**Lisans:** İstanbul Büyükşehir Belediyesi Açık Veri Lisansı  
**Kategori:** Mobilite

---

## Gerçek Veri Şemaları

### Dataset A: Günlük Yolcu Verisi (ANA)
```
CSV, ; delimiter, ~135K satır/yıl, toplam ~680K satır
─────────────────────────────────────────────────────────
transaction_year  │ 2025
transaction_month │ 5
transaction_day   │ 26
line              │ TF2-TELEFERIK EYUP PIYERLOTI (uzun format)
station_name      │ Pierloti
station_number    │ PLT-EYU
town              │ EyüpSultan
longitude         │ 28.933...
latitude          │ 41.053...
passage_cnt       │ 1386  (geçiş/tap-in sayısı)
passanger_cnt     │ 604   (tekil yolcu sayısı)
─────────────────────────────────────────────────────────
22 hat: M1-M9 Metro, T1-T5 Tramvay, F1/F4 Füniküler,
TF1-TF2 Teleferik, TCDD/Marmaray, İETT Nostaljik
```

### Dataset B: İstasyon Envanteri (178 istasyon)
```
XLSX
─────────────────────────────────────────────
Hat Adı                  │ M1
İstasyon Adı             │ YENİKAPI
İlçe Adı                 │ FATİH
İstasyon Büyüklüğü       │ 2500 (m²)
Yürüyen Merdiven Sayısı  │ 8
Asansör Sayısı           │ 2
─────────────────────────────────────────────
```

### Dataset C: Hat Uzunlukları
```
XLSX — Hat başına metre cinsinden uzunluk
```

### Dataset D: Aktarma Noktaları
```
XLSX — Hangi istasyonda hangi hatta aktarma yapılabiliyor
```

---

## Birleştirme (Merge) Stratejisi

```
                            ┌──────────────────────┐
                            │  Dataset B: İstasyon │
                            │  Envanteri (178 satır)│
                            │  • büyüklük          │
                            │  • yürüyen merdiven   │
                            │  • asansör            │
                            └──────┬───────────────┘
                                   │ JOIN: line_code + station_name
                                   │ (Temizlenmiş isim eşleştirmesi)
                                   ▼
┌──────────────────────┐    ┌──────────────────────────┐
│  Dataset A: Yolcu    │    │  Dataset A+B: Günlük     │
│  2021-2025 (680K)    │───▶│  yolcu + istasyon        │
│  • günlük granularite│    │  altyapı özellikleri     │
│  • passage_cnt       │    └──────────┬───────────────┘
│  • passanger_cnt     │               │
│  • lat/long          │               │ JOIN: line_code
└──────────────────────┘               ▼
                            ┌──────────────────────────┐
┌──────────────────────┐    │  Dataset A+B+C:          │
│  Dataset C: Hat      │───▶│  + hat uzunluğu          │
│  Uzunlukları         │    │  (km başına yolcu hesabı)│
└──────────────────────┘    └──────────┬───────────────┘
                                       │
┌──────────────────────┐               │ JOIN: line + station
│  Dataset D: Aktarma  │───────────────┘
│  Bilgileri           │    ▶ Dataset A+B+C+D: FULL
└──────────────────────┘    • is_transfer (binary)
                            • transfer_count
```

**Merge Zorlukları:**
- Hat isimleri A'da `M1-YENIKAPI-HAVALIMANI`, B'de `M1` formatında → regex ile kısa kod çıkarma
- İstasyon isimleri büyük/küçük harf ve Türkçe karakter normalizasyonu
- Gerekirse fuzzy matching (difflib) uyumsuz isimler için

---

## 5 Araştırma Sorusu

| # | Soru | Veri Kaynağı | Yöntem |
|---|---|---|---|
| **RQ1** | İstanbul raylı sistemlerinde **mevsimsellik ve haftanın günü etkisi** nedir? Hangi hat tipleri (Metro/Tramvay/Füniküler) farklı zamansal profiller gösterir? | A (günlük) | Time series decomposition, heatmap (ay×gün), lineplot |
| **RQ2** | Bir istasyonun **fiziksel özellikleri** (büyüklük, yürüyen merdiven, asansör) günlük yolcu trafiğini ne ölçüde açıklar? Altyapı → talep tahmini yapılabilir mi? | A+B | Linear Regression, correlation heatmap, residual analysis |
| **RQ3** | İstasyonlar altyapı, lokasyon ve yolcu profiline göre nasıl **kümelenir**? Aktarma istasyonları ayrı bir küme oluşturur mu? | A+B+C+D | K-Means (Elbow + Silhouette), PCA, scatter + bar plot |
| **RQ4** | Yolcu yoğunluğu İstanbul coğrafyasında nasıl **mekansal** dağılıyor? Hangi ilçeler/metro aksları baskın? | A (lat/long) | Folium/Scatter geo-plot, ilçe bazlı choropleth |
| **RQ5** | **Hat verimliliği:** Km başına düşen yolcu sayısı hangi hatlarda en yüksek? En "verimsiz" hat hangisi? Yatırım önceliklendirmesi için ne söyler? | A+B+C | Barplot (yolcu/km), grouped bar (hat tipi bazında) |

---

## Notebook Yapısı (8 Bölüm)

```
§1  GİRİŞ & PROBLEM TANIMI
    - 5 araştırma sorusu, bağlam (İstanbul ulaşımı)
    - Veri kaynakları tablosu (4 dataset, lisans, link)

§2  VERİ YÜKLEME
    - 5 CSV (2021-25) pd.read_csv(sep=';')
    - 3 XLSX pd.read_excel()
    - pd.concat([df21, df22, ...]) → ~680K satır

§3  VERİ TEMİZLEME & BİRLEŞTİRME
    - Eksik veri: isnull().sum(), dropna/fillna stratejisi
    - Tip dönüşümü: date sütunları → datetime
    - Aykırı değer: passage_cnt için IQR, boxplot
    - Hat ismi normalizasyonu: regex extract kısa kod
      "M1-YENIKAPI-HAVALIMANI" → "M1"
    - İstasyon ismi eşleştirme: .str.upper().str.strip()
      + fuzzy matching (difflib) uyumsuzlar için
    - Merge A+B → merge C → merge D
    - Yeni türetilmiş sütunlar:
      • day_of_week, month, is_weekend, is_holiday
      • passage_per_m2, passanger_per_escalator
      • is_transfer (binary)
      • line_efficiency (passage_cnt / line_length_km)

§4  KEŞİFSEL VERİ ANALİZİ (EDA)
    - .info(), .describe(), missingno matrix
    - Viz ①: Lineplot — 5 yıllık aylık trend (tüm hatlar)
    - Viz ②: Heatmap — Gün×Ay ortalama yolcu (mevsimsel)
    - Viz ③: Barplot — Hat bazında yıllık toplam (top-10)
    - Viz ④: Boxplot — Hat tipine göre günlük dağılım
    - Viz ⑤: Scatter geo — İstasyon konum×yolcu büyüklüğü
    - Viz ⑥: Correlation heatmap — sayısal değişkenler
    ▶ RQ1 cevabı (mevsimsellik + gün etkisi)

§5  MODELLEME #1: REGRESYON
    - Hedef: günlük passage_cnt
    - Features: büyüklük, merdiven, asansör, is_transfer,
      day_of_week, month, line_type
    - LinearRegression, train/test split
    - R², MAE, RMSE, residual plot, feature importance
    ▶ RQ2 cevabı (altyapı → talep tahmini)

§6  MODELLEME #2: KÜMELEME
    - Features: ortalama passage, büyüklük, merdiven,
      asansör, is_transfer, ilçe (one-hot)
    - StandardScaler, PCA (2 comp görselleştirme)
    - Elbow + Silhouette → optimal k
    - K-Means fit, küme profillemesi (grup istatistikleri)
    - Viz: PCA scatter (renk=küme), barplot (küme profili)
    ▶ RQ3 cevabı (istasyon tipolojisi)

§7  COĞRAFİ & VERİMLİLİK ANALİZİ
    - Folium interaktif harita: circle marker büyüklüğü
      yolcu ile orantılı, renk = hat tipi
    - Barplot: yolcu/km (line_efficiency)
    - İlçe bazlı toplam yolcu (choropleth ops.)
    ▶ RQ4 + RQ5 cevapları

§8  SONUÇLAR, SINIRLAMALAR, ÖĞRENİLENLER
    - 5 sorunun net cevapları
    - Merge zorlukları (isim eşleştirme sorunları)
    - Veri kalitesi notları (passage vs passanger farkı)
    - İstanbul ulaşım planlaması için çıkarımlar
    - Gelecek çalışma: GTFS + saatlik veri entegrasyonu
```

---

## Teslimat Paketi

| Dosya | İçerik |
|---|---|
| `proje.ipynb` | Tüm kod + markdown açıklamaları, yorum satırlarıyla |
| `rapor.pdf` | 3-5 sayfa: problem, veri seti, yöntem, bulgular, sınırlamalar, öğrenilenler |
| `prompt_gunlugu.md` | Kronolojik 10-15 önemli prompt, hangi adımda kullanıldığı |
| `README.md` | Ekip bilgileri, çalıştırma talimatları, kütüphaneler, veri kaynağı & lisans |
| GitHub repo | Tümünü içeren tek repo → link olarak teslim |

---

## Karşılanan Zorunlu Bileşenler

| Bileşen | Karşılama |
|---|---|
| Veri yükleme ve temizleme | 4 dataset, encoding, eksik veri, aykırı değer, tip dönüşümü, fuzzy matching merge |
| EDA + ≥4 görselleştirme | 6 viz: lineplot, heatmap, barplot, boxplot, scatter geo, correlation heatmap |
| ≥3 araştırma sorusu | 5 soru: mevsimsellik, altyapı-talep, kümeleme, mekansal, verimlilik |
| Basit modelleme | LinearRegression + K-Means (2 model) |
| Tema bağlamında yorum | Her soru İstanbul ulaşımı bağlamında cevaplanacak |

---

## Kütüphaneler

```
pandas, numpy, matplotlib, seaborn, scikit-learn
(LinearRegression, KMeans, StandardScaler, PCA, silhouette_score),
folium (harita), difflib (fuzzy matching), openpyxl (XLSX okuma)
```

---

## Neden Zengin Bir Proje?

1. **4 ayrı veri setinin birleştirilmesi** — şema uyuşmazlığı, isim normalizasyonu, fuzzy matching gibi gerçek dünya problemleri
2. **Günlük granularite** — mevsimsellik ve haftanın günü analizi mümkün
3. **Çok boyutluluk:** Zamansal (5 yıl) × Mekansal (GPS) × Altyapısal × Ağsal (aktarma)
4. **İki farklı model:** Regresyon (tahmin) + Kümeleme (tipoloji)
5. **Coğrafi görselleştirme:** Folium interaktif harita
6. **Somut politika çıkarımları:** Hangi hatta yatırım yapılmalı? Hangi istasyon tipi neye hizmet ediyor?
