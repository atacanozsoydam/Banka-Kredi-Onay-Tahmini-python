# Banka Kredi Onay Tahmini (Python Veri Analizi ve Veri Bilimi Projesi)

Bu çalışmada 4.269 banka müşterisinin kredi başvuru verilerini (`loan_approval_dataset.csv`) inceleyerek, kredi onay ve red kararlarında hangi değişkenlerin etkili olduğunu analiz ettim ve yeni başvuruları tahmin edebilen bir sınıflandırma modeli kurdum.

## Kullanılan Kütüphaneler
* **Veri Analizi:** `pandas`, `numpy`
* **Görselleştirme:** `matplotlib`, `seaborn`
* **İstatistiksel Test:** `scipy.stats`
* **Makine Öğrenmesi:** `scikit-learn`

## Veri Seti Değişkenleri

| Değişken Adı | Türkçe Karşılığı / Açıklama |
| :--- | :--- |
| `loan_id` | Başvuru kimlik numarası |
| `no_of_dependents` | Bakmakla yükümlü olunan kişi sayısı |
| `education` | Eğitim durumu (`Graduate` / `Not Graduate`) |
| `self_employed` | Kendi işinde çalışma durumu (`Yes` / `No`) |
| `income_annum` | Yıllık gelir |
| `loan_amount` | Talep edilen kredi tutarı |
| `loan_term` | Kredi vadesi (Yıl) |
| `cibil_score` | CIBIL Kredi Notu (300 - 900 arası) |
| `residential_assets_value` | Konut varlığı değeri |
| `commercial_assets_value` | Ticari varlık değeri |
| `luxury_assets_value` | Lüks varlık (araç, mücevher vb.) değeri |
| `bank_asset_value` | Banka nakit / mevduat varlığı değeri |
| `loan_status` | **Hedef Değişken:** Kredi onay durumu (`Approved` / `Rejected`) |

## Yapılan İşlemler

### 1. Veri Ön İşleme ve Temizlik
* Sütun isimlerinin ve metin tipindeki sütunların (`education`, `self_employed`, `loan_status`) başında bulunan gereksiz boşluklar `.str.strip()` ile temizlendi.
* `df.describe()` kontrolünde konut varlığı (`residential_assets_value`) sütununda hatalı olarak `-100.000` girilmiş 28 kayıt tespit edildi ve bu değerler `0` olarak düzeltildi.

### 2. Analiz ve Görselleştirme Bulguları
* Müşterinin 4 farklı varlık sütunu toplanarak `total_assets` (Toplam Varlık), ayrıca analiz aşamasında incelemek için `loan_to_income` (Kredi/Gelir Oranı) ve `asset_coverage` (Teminat Oranı) sütunları üretildi.
* **Gelir ve Varlık:** Onaylanan ve reddedilen müşterilerin ortalama yıllık gelirleri (~5.03M vs ~5.11M) ve toplam varlıkları (~32.38M vs ~32.83M) birbirine oldukça yakın çıktı. Korelasyon matrisinde gelir ile kredi tutarı arasında `0.93` korelasyon olduğu görüldü (geliri yüksek olan müşteriler paralel olarak daha yüksek kredi talep ediyor).
* **Eğitim ve Çalışma Şekli:** Üniversite mezunu olma veya kendi işinde çalışma durumlarına göre onay oranları (~%62) belirgin bir farklılık göstermedi.
* **CIBIL Kredi Notu ve Vade:** Onaylanan müşterilerin ortalama CIBIL notu `703.46`, reddedilenlerin ise `429.47` bulundu. Bu farkın tesadüf olmadığı Bağımsız İki Örneklem T-Testi (`T = 78.96`, `p < 0.001`) ile doğrulandı.
* Resmi CIBIL kredi notu aralıklarına göre yapılan segmentasyonda, 550 puan ve üzeri gruplarda onay oranı ~%99.5 iken `300-549` (Yüksek Risk) grubunda %89.64 red oranı görüldü. Yüksek riskli grupta onay alan %10.36'lık kesimin tamamının 2 ve 4 yıllık kısa vadeli başvurular olduğu, 6 yıl ve üzeri vadelerde bu gruba onay verilmediği gözlemlendi.

### 3. Modelleme Sonuçları (Random Forest)
* Veri seti %80 eğitim (3.415 satır) ve %20 test (854 satır) olarak ayrıldı.
* Aşırı öğrenmeyi (overfitting) önlemek için `GridSearchCV` (5 katlı çapraz doğrulama) ile ağaç derinliği sınırlandırıldı ve `max_depth=5`, `n_estimators=100` parametreleri seçildi.
* **Eğitim (Train) Doğruluğu:** %98.48
* **Test (Sınav) Doğruluğu:** %97.54 (854 başvuruda 833 doğru tahmin)
* **Eğitim - Test Farkı:** %0.94
* Modelin kararlarında en etkili değişkenler `cibil_score` (yaklaşık %82) ve `loan_term` (yaklaşık %10) oldu.

## Dosyalar
* `banka_kredi_onay_tahmini.ipynb`: Analiz, görselleştirme ve modelleme kodlarının tamamı.
* `loan_approval_dataset.csv`: Projede kullanılan veri seti.
