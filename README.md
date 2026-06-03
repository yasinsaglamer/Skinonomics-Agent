#  Skinonomics Agent

> **CS2 skin pazarını bir FMCG tedarik zinciri gibi modelleyen, uçtan uca Agentic BI & Veri Mühendisliği platformu.**

Steam Market üzerindeki sanal eşyaları (CS2 skinleri) hızlı tüketim ürünleri gibi ele alır; arz-talep dengesizliklerini, pazar likiditesini ve stok tükenme (stock-out) risklerini istatistiksel modellerle tespit eder ve **kendi kendine anomali sinyalleri üreten** otonom bir karar katmanı sunar.

---

##  Proje Vizyonu

Geleneksel BI dashboard'ları veriyi *gösterir*. Skinonomics Agent veriyi **yorumlar ve uyarır**. Sistem, ham Steam verisini alıp şu soruları otomatik yanıtlar:

- Hangi ürünüm **tükenmek üzere** ve fiyatı spekülatif mi artıyor? *(Stock-out / Arz Şoku)*
- Hangi üründe **organik talep patlaması** var? *(Demand Surge — büyüme fırsatı)*
- Hangi ürün **ölü stok**? *(Düşük likidite — sermaye sıkışması)*

Bu, sanal eşya ekonomisini klasik perakende (FMCG) metrikleriyle birleştiren bir köprüdür.

---

##  Mimari: Medallion Architecture (Bronze → Silver → Gold)

```
Steam API → [Data Factory Trigger] → Bronze (Raw Delta)
                                          ↓
                                   Silver (Cleaned / Flattened)
                                          ↓
                                   Gold (İstatistik + Agentic Flags)
                                          ↓
                                   Power BI (DirectLake)
```

| Katman | Sorumluluk | FMCG Karşılığı |
|--------|------------|----------------|
| **Bronze** | Ham JSON'u dokunmadan, append mantığıyla arşivle | Tedarikçiden mal kabul |
| **Silver** | Nested veriyi düzleştir, temizle, tipleri standardize et | Depo / kalite kontrol |
| **Gold** | Z-Score, IQR, SMA/EMA + likidite skorları hesapla | Karar merkezi |
| **Agentic** | Kurallara göre insan-okunur uyarı (flag) üret | Otomatik denetçi |
| **Serving** | DirectLake ile anlık güncellenen dashboard | Vitrin |

---

##  İstatistiksel Modeller & FMCG Karşılıkları

| Model | Ne Yapar | FMCG Anlamı |
|-------|----------|-------------|
| **Rolling Z-Score** | Fiyat/hacmin geçmiş normale göre sapması | Beklenmedik talep/arz sapması |
| **IQR Outlier (Tukey)** | Robust fiyat fırlaması tespiti | Spekülatif fiyat anomalisi |
| **SMA-7 / EMA-3** | Trend ve momentum | Shelf turnover / talep ivmesi |
| **Liquidity Score** | Spread + hacim birleşimi | Rafta bulunabilirlik |

###  Agentic Tetikleyiciler (Triggers)

| Kural | Koşul | Sinyal |
|-------|-------|--------|
| **RULE 1 — Stock-out** | Volume Z < -1.5 **VE** Price Z > 1.5 |  Arz Şoku |
| **RULE 2 — Demand Surge** | 3 gün üst üste hacim > SMA-7 × 1.2 |  Talep Patlaması |
| **RULE 3 — Dead Inventory** | 7 gün boyunca hacim < 100 |  Ölü Stok |

---

##  Teknoloji Yığını

**Platform:** Microsoft Fabric (OneLake, Data Factory, Synapse Data Engineering)
**İşleme:** PySpark, Python (Requests, Pandas)
**Depolama:** Delta Lake (Lakehouse)
**Görselleştirme:** Power BI (DirectLake)
**İstatistik:** Time-Series Analysis, Anomaly Detection (Z-Score, IQR)

---

##  Repo Yapısı

```
skinonomics-agent/
├── docs/              
├── notebooks/         
├── src/               
│   ├── ingestion/         
│   ├── transformations/   
│   ├── statistics/        
│   └── agentic/           
├── config/            
├── pipelines/         
├── powerbi/           
└── tests/             
```

---

##  Dashboard 

| Sayfa | Hedef Kitle | İçerik |
|-------|-------------|--------|
| **Executive Summary** | C-Level / İş Birimleri | Portföy sağlık skoru, aktif alarmlar, jargon yok |
| **Market Analyst View** | Analistler | Fiyat/hacim trendleri, slicer'lı detay |
| **Statistical Detail** | Veri Mühendisleri | Z-Score serileri, IQR sınırları, doğrulama |

---

##  Proje Durumu

>  **Geliştirme aşamasında.** Mimari ve kod katmanları kurgulanıyor; canlı dağıtım (deployment) öncesi yerel validasyon devam ediyor.

- [x] Mimari tasarım & repo yapısı
- [x ] Bronze ingestion (API client)
- [ ] Silver transformation
- [ ] Gold statistics + Agentic rules
- [ ] Power BI DirectLake dashboard
- [ ] Canlı dağıtım (Fabric deployment)

---

##  Geliştirici

**Yasin Sağlamer** — Data & Business Intelligence Analyst
İstatistik (B.Sc.) öğrencisi 

 yasinsaglamer9@gmail.com · [LinkedIn](https://linkedin.com/in/yasinsaglamer) · [GitHub](https://github.com/yasinsaglamer)

---

*Bu proje eğitim ve portföy amaçlıdır. Steam ve CS2, Valve Corporation'ın ticari markalarıdır proje Valve ile bağlantılı değildir.*
