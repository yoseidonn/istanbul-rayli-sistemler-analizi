# İstanbul Raylı Sistemler Yolcu Analizi (2021-2025)

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**Veri Bilimi Dönem Projesi** — İstanbul Büyükşehir Belediyesi Açık Veri Portalı'ndaki raylı sistem verilerinin çok boyutlu analizi.

> **Tema:** Akıllı Şehir & Toplu Taşıma  
> **Teslim Tarihi:** 3 Temmuz 2026  
> **Çalışma Şekli:** İki kişilik proje

### Ekip

| Öğrenci | Numara |
|---------|--------|
| Yusuf Kiriş | 1306240051 |
| Burhan Seyfi | 1306200019

---

## Araştırma Soruları

| # | Soru | Yöntem |
|---|------|--------|
| RQ1 | Mevsimsellik ve haftanın günü etkisi nedir? | Time series, heatmap, lineplot |
| RQ2 | İstasyon altyapısı yolcu trafiğini ne ölçüde açıklar? | Linear Regression |
| RQ3 | İstasyonlar nasıl kümelenir? | K-Means + PCA + Silhouette |
| RQ4 | Yolcu yoğunluğu mekansal olarak nasıl dağılıyor? | Geo scatter, koordinat analizi |
| RQ5 | Km başına yolcu verimliliği hangi hatlarda en yüksek? | Hat verimlilik analizi |

---

## Veri Kaynakları

Tüm veriler [İBB Açık Veri Portalı](https://data.ibb.gov.tr) — **Mobilite** kategorisinden alınmıştır.

| Dataset | İçerik | Format | Lisans |
|---------|--------|--------|--------|
| A | Raylı Sistemler İstasyon Bazlı Yolcu ve Yolculuk Sayıları (2021-2025) | 5× CSV | İBB Açık Veri Lisansı |
| B | Raylı Sistemlere Ait İstasyon Bilgileri (178 istasyon) | 1× XLSX | İBB Açık Veri Lisansı |
| C | Raylı Sistemlere Ait Hat Uzunluk Bilgileri | 1× XLSX | İBB Açık Veri Lisansı |
| D | Raylı Sistemlere Ait Aktarma Bilgileri | 1× XLSX | İBB Açık Veri Lisansı |

**Veri Seti Bağlantıları:**
- [Yolcu Sayıları](https://data.ibb.gov.tr/dataset/rayli-sistemler-istasyon-bazli-yolcu-ve-yolculuk-sayilari)
- [İstasyon Bilgileri](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-istasyon-bilgileri)
- [Hat Uzunlukları](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-hat-uzunluk-bilgileri)
- [Aktarma Bilgileri](https://data.ibb.gov.tr/dataset/rayli-sistemlere-ait-aktarma-bilgileri)

---

## Proje Yapısı

```
verib/
├── proje.ipynb              # Ana notebook (8 bölüm, 25 kod hücresi)
├── rapor.pdf → outputs/     # 4 sayfa PDF rapor
├── prompt_gunlugu.md        # 13 prompt'luk kronolojik günlük
├── README.md                # Bu dosya
├── PLAN.md                  # Orijinal proje planı
├── .gitignore               # data/ ve .venv/ hariç
├── data/                    # Ham veri setleri (~210 MB, git-ignore)
│   ├── 2021_yolcu.csv
│   ├── 2022_yolcu.csv
│   ├── 2023_yolcu.csv
│   ├── 2024_yolcu.csv
│   ├── 2025_yolcu.csv
│   ├── istasyon_bilgileri.xlsx
│   ├── hat_uzunluklari.xlsx
│   └── aktarma_bilgileri.xlsx
└── outputs/                 # Notebook tarafından üretilen tüm çıktılar
    ├── viz1_monthly_trend.png
    ├── viz2_heatmap.png
    ├── viz3_barplot.png
    ├── viz4_boxplot.png
    ├── viz5_corr_scatter.png
    ├── missingno_matrix.png
    ├── outlier_analysis.png
    ├── regression_results.png
    ├── optimal_k.png
    ├── cluster_results.png
    ├── geo_stations.png
    ├── line_efficiency.png
    ├── folium_map.html       # İnteraktif harita
    └── rapor.pdf
```

---

## Notebook Yapısı

| Bölüm | İçerik |
|-------|--------|
| **§1** | Giriş & Problem Tanımı — 5 araştırma sorusu, veri kaynakları |
| **§2** | Veri Yükleme — 5 CSV + 3 XLSX, farklı format/encoding handling |
| **§3** | Veri Temizleme & Birleştirme — eksik veri, aykırı değer, fuzzy merge |
| **§4** | Keşifsel Veri Analizi (EDA) — 6 görselleştirme, RQ1 yanıtı |
| **§5** | Modelleme #1: Regresyon — Linear Regression, RQ2 yanıtı |
| **§6** | Modelleme #2: Kümeleme — K-Means, PCA, Elbow/Silhouette, RQ3 yanıtı |
| **§7** | Coğrafi Analiz & Hat Verimliliği — Geo scatter, RQ4+RQ5 yanıtı |
| **§8** | Sonuçlar, Sınırlamalar ve Öğrenilenler |

---

## Gereksinimler

### Kütüphaneler

```bash
pip install --break-system-packages \
  pandas numpy matplotlib seaborn scikit-learn \
  jupyter folium openpyxl
```

Veya sistem paketleri olarak (Debian/Ubuntu):

```bash
sudo apt install python3-pandas python3-numpy python3-matplotlib \
  python3-seaborn python3-sklearn python3-jupyter python3-folium
```

### Minimum Sürümler

| Kütüphane | Versiyon |
|-----------|----------|
| Python | ≥ 3.8 |
| pandas | ≥ 1.3 |
| numpy | ≥ 1.21 |
| matplotlib | ≥ 3.5 |
| seaborn | ≥ 0.11 |
| scikit-learn | ≥ 1.0 |
| jupyter | ≥ 6.0 |
| openpyxl | ≥ 3.0 |
| folium | ≥ 0.12 |

---

## Çalıştırma

### 1. Veri Setlerini İndirin

Veri setlerini [İBB Açık Veri Portalı](https://data.ibb.gov.tr/group/ulasim-hizmetleri?q=hat%2Fyolcu)'ndan indirip `data/` dizinine yerleştirin.

### 2. Notebook'u Başlatın

```bash
cd verib/
jupyter notebook proje.ipynb
```

### 3. Tüm Hücreleri Çalıştırın

Kernel → Restart & Run All (veya Cell → Run All)

### 4. Komut Satırından Çalıştırma

```bash
jupyter nbconvert --to notebook --execute --inplace proje.ipynb
```

---

## Modelleme Özeti

### Regresyon (RQ2)
- **Hedef:** Günlük `passage_cnt`
- **Features:** İstasyon büyüklüğü, yürüyen merdiven, asansör, aktarma durumu, haftasonu, ay, hat uzunluğu
- **Model:** Linear Regression (StandardScaler ile)
- **Metrikler:** R², MAE, RMSE

### Kümeleme (RQ3)
- **Features:** log(ortalama geçiş), büyüklük, merdiven, asansör, aktarma
- **Model:** K-Means (StandardScaler + PCA ile)
- **Optimal K:** Elbow + Silhouette analizi ile belirlenir

---

## Lisans

Bu proje [MIT Lisansı](LICENSE) altında lisanslanmıştır.  
Projede kullanılan veriler **İstanbul Büyükşehir Belediyesi Açık Veri Lisansı** altındadır.  
Detaylar için: [İBB Açık Veri Lisansı](https://data.ibb.gov.tr)
