# 🛒 E-Commerce Müştəri Seqmentasiyası — RFM + Klasterləşdirmə

## 📦 Data haqqında

**Dataset:** [E-Commerce Data — Kaggle (carrie1)](https://www.kaggle.com/datasets/carrie1/ecommerce-data)

Bu dataset Böyük Britaniyada fəaliyyət göstərən bir onlayn mağazanın **2010–2011-ci illər ərzindəki** əməliyyat məlumatlarını əhatə edir. Dataset aşağıdakı sütunlardan ibarətdir:

| Sütun | Açıqlama |
|---|---|
| InvoiceNo | Sifariş nömrəsi |
| StockCode | Məhsul kodu |
| Description | Məhsul adı |
| Quantity | Sifariş miqdarı |
| InvoiceDate | Sifariş tarixi |
| UnitPrice | Vahid qiyməti |
| CustomerID | Müştəri ID-si |
| Country | Ölkə |

---

## 🎯 Məqsəd

Müştəriləri davranışlarına görə qruplara ayırmaq — **kim ən dəyərli müştəridir, kim itirilmək üzrədir?** Bu sualı cavablandırmaq üçün **RFM analizi** və **klasterləşdirmə alqoritmləri** tətbiq edilmişdir.

---

## 🪜 Addımlar

### 1. Data Yüklənməsi və İlkin Baxış
- `df.info()`, `df.describe()`, `df.shape` ilə ümumi struktura baxıldı
- Null dəyərlərin sayı və faizi hesablandı
- `CustomerID` və `Description`-da boş sətrlər silindi

### 2. Feature Engineering — RFM Hesablanması
Hər müştəri üçün 3 metrik hesablandı:

| Metrik | Açıqlama |
|---|---|
| **Recency** | Son alış-verişdən neçə gün keçib (az = yaxşı) |
| **Frequency** | Neçə dəfə sifariş verib |
| **Monetary** | Ümumi xərclədiyi məbləğ |

### 3. EDA (Kəşfedici Məlumat Analizi)

**📊 Boxplot — log-dan əvvəl**
Bütün 3 metrikdə güclü outlier-lər aşkar edildi. Recency, Frequency və Monetary dəyərləri çox sağa yayılmış (right-skewed) paylanma göstərirdi.

**📊 Histogram — log-dan əvvəl**
Paylanma normala yaxın deyildi — əksər müştərilər aşağı Frequency və Monetary dəyərlərinə sahib idi, az sayda müştəri çox yüksək dəyərlər göstərirdi.

**🔄 log1p Transformasiyası**
`np.log1p()` tətbiq edildi — skewness azaldıldı, outlier-lərin təsiri zəiflədildi.

**📊 Boxplot — log-dan sonra**
Transformasiyadan sonra dəyərlər daha balanslaşdı, paylanma normala yaxınlaşdı.

**📊 Scatter plot — Frequency vs Monetary**
Frequency artdıqca Monetary da artır — müsbət korrelyasiya müşahidə edildi. Tez-tez alış-veriş edən müştərilər daha çox xərcləyir.

**📊 Korrelyasiya Heatmap**
Frequency və Monetary arasında güclü müsbət korrelyasiya var. Recency ilə digər metriklərin korrelyasiyası zəifdir.

**📊 Pairplot**
Hər iki metrik cütü arasındakı əlaqə vizuallaşdırıldı. Frequency-Monetary cütü ən aydın klaster strukturunu göstərdi.

### 4. Scaling
`StandardScaler` ilə data standartlaşdırıldı — bütün feature-lar eyni miqyasa gətirildi ki, məsafə əsaslı alqoritmlər düzgün işləsin.

### 5. KMeans Klasterləşdirməsi

**📊 Elbow Metodu**
K=1-dən K=10-a qədər inertia dəyərləri hesablandı. K=4-də "dirsək" (elbow) aydın görünür — optimal klaster sayı **4** seçildi.

**📊 Klaster Sayı — Countplot**
Hər klasterdəki müştəri sayı göstərildi. Klasterlər arasında müştəri sayı nisbətən balanslaşdırılmış paylanma göstərdi.

**📊 Klaster Paylanması — Pie Chart**
Hər klasterin ümumi müştəri bazasındakı faiz payı göstərildi.

**📊 RFM Metrik Müqayisəsi — Line Plot**
Hər klasterin ortalama Recency, Frequency və Monetary dəyərləri müqayisə edildi. Klaster 3-ün Monetary dəyəri digərlərindən əhəmiyyətli dərəcədə yüksəkdir.

**Seqment Adlandırması:**

| Klaster | Seqment | Xüsusiyyət |
|---|---|---|
| 3 | 🏆 VIP | Ən yüksək xərcləmə, tez-tez alış-veriş |
| 2 | 💚 Loyal | Yüksək Frequency, sabit müştəri |
| 1 | 🌱 Potential | Orta dəyərlər, inkişaf potensialı var |
| 0 | ⚠️ At Risk | Aşağı aktivlik, itirilmə riski var |

**📊 Seqment Countplot**
Seqment adları ilə hər qrupdakı müştəri sayı göstərildi. Ən böyük qrup **Potential** (~1600), ən kiçik qrup isə **VIP** (~740) müştərilərdən ibarətdir.

**📊 Scatter Plot — Centroid-lərlə**
Klasterlər və centroid-lər (mərkəz nöqtələr) vizuallaşdırıldı. Hər klasterin mərkəzi aydın görünür.

### 6. İyerarxik Klasterləşdirmə

**📊 Dendrogram**
Ward linkage metodu ilə dendrogram quruldu. 4 əsas branch aydın görünür — bu da K=4 seçimini təsdiqləyir.

`AgglomerativeClustering` ilə eyni data üzərində 4 klaster quruldu.

### 7. DBSCAN
`eps=0.5`, `min_samples=5` parametrləri ilə DBSCAN tətbiq edildi.

---

## 📈 Score Müqayisəsi

| Alqoritm | Silhouette Score |
|---|---|
| **KMeans** | ~0.35+ (ən yüksək) |
| **İyerarxik** | orta |
| **DBSCAN** | ~0.206 (ən aşağı) |

### Niyə belə oldu?

**KMeans ən yüksək score-u göstərdi** çünki:
- KMeans dəyirmi, kompakt klasterlər yaradır
- Silhouette score da məhz bu tip klasterləri yüksək qiymətləndirir
- RFM datası KMeans üçün əlverişli strukturdadır

**İyerarxik KMeans-dən bir qədər aşağıdır** çünki:
- Dendrogram vasitəsilə struktur görünür, amma silhouette-i optimize etmir
- Birləşmə məntiqi fərqli işləyir

**DBSCAN ən aşağı score-u göstərdi** çünki:
- DBSCAN noise nöqtələrini (-1) ayrı klaster kimi qeyd edir — bu silhouette hesablamasını aşağı salır
- RFM kimi sıx və davamlı paylanmalı data DBSCAN üçün ideal deyil
- DBSCAN daha çox qeyri-müntəzəm formalı klasterlər üçün güclüdür

---

## 💡 Biznes İnsightları

- **VIP müştərilər** (~740 nəfər) ən az sayda olsa da ən yüksək gəliri gətirir — loyallıq proqramları ilə saxlanılmalıdır
- **At Risk müştərilər** (~860 nəfər) — xüsusi kampaniyalar, endirim təklifləri ilə geri qaytarılmalıdır
- **Potential müştərilər** (~1600 nəfər) — ən böyük qrupdur, düzgün marketinqlə Loyal-a çevrilə bilər
- **Loyal müştərilər** (~1150 nəfər) — sabit baza, VIP-ə keçid üçün əlavə təşviq oluna bilər

---

## 🛠️ İstifadə olunan texnologiyalar

`Python` `Pandas` `NumPy` `Scikit-learn` `Matplotlib` `Seaborn` `SciPy`
