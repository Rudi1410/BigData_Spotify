# BigData_Spotify 🎧

Proyek Big Data untuk menganalisis dan mengelompokkan lagu berdasarkan pola performa pada **Spotify Daily Charts**. Proyek ini mencakup proses ETL (Extract, Transform, Load) berskala besar menggunakan PySpark, eksplorasi data (EDA), dan clustering dengan algoritma K-Means untuk menemukan segmentasi/pola tersembunyi dari lagu-lagu yang masuk chart.

## 📌 Ringkasan Proyek

Dataset yang digunakan adalah [`gonzalopezgil/spotify-charts-daily-updated`](https://www.kaggle.com/datasets/gonzalopezgil/spotify-charts-daily-updated) dari Kaggle — berisi data chart harian Spotify dari berbagai negara sejak tahun 2017, dengan total lebih dari 1 juta baris data dan beberapa file terkait (chart, songs, artist, dll) yang saling terhubung melalui `track_uri` / `artist_uris`.

Tujuan utama proyek:
1. Membersihkan dan menggabungkan data chart harian berskala besar (volume, variety) menggunakan **Apache Spark**.
2. Melakukan eksplorasi data untuk memahami karakteristik chart, artis, label, dan tren waktu.
3. Membangun fitur (feature engineering) di level *track × country* untuk merepresentasikan performa sebuah lagu di chart.
4. Melakukan **clustering (K-Means)** untuk mengelompokkan lagu/track berdasarkan pola performanya di chart.

## 📂 Struktur Notebook

| Notebook | Deskripsi |
|---|---|
| [`EDA.ipynb`](./EDA.ipynb) | Eksplorasi data awal pada dataset Spotify Charts Daily (sampling, statistik deskriptif, analisis univariate/bivariate/multivariate, analisis time series, dan insight akhir). |
| [`ETL.ipynb`](./ETL.ipynb) | Pipeline ETL menggunakan PySpark: load data dari Google Drive, data cleaning, penanganan missing value, record linkage antar tabel, feature engineering, agregasi, encoding, scaling, hingga export hasil akhir ke Parquet. |
| [`Clustering.ipynb`](./Clustering.ipynb) | Pemodelan **K-Means Clustering** dengan PySpark MLlib: pencarian K optimal (Elbow Method & Silhouette Score), training model final, visualisasi cluster dengan PCA 2D, dan profiling rata-rata fitur per cluster. |

## 🔍 EDA — Exploratory Data Analysis (`EDA.ipynb`)

Tahapan yang dilakukan:
0. Instalasi & import library (`kagglehub`, `missingno`, `pandas`, `seaborn`, dll)
1. Load dataset dari Kaggle (sampling karena ukuran data ±42 juta baris)
2. Data overview (tipe data, statistik deskriptif, memory usage)
3. Data cleaning (missing values, duplikat, konversi tipe data)
4. Univariate analysis (distribusi numerik & kategorikal)
5. Bivariate analysis (top artist, top track, top label, scatter plot)
6. Multivariate analysis (correlation heatmap, tren artist per tahun, perbandingan volume seluruh file)
7. Time series analysis (ekstraksi fitur waktu, tren bulanan, tren artist per kuartal)
8. Insight & kesimpulan EDA

**Insight utama:**
- Dataset bersifat *big data* — ratusan ribu hingga puluhan juta baris, tersebar di 9 file berbeda yang saling terhubung.
- Sebagian besar kolom awalnya bertipe `string` dan perlu dikonversi (terutama kolom tanggal).
- Sejumlah kecil *major label* mendominasi chart, sementara distribusi artis bersifat *long-tail*.
- Volume data chart meningkat dari tahun ke tahun seiring pertumbuhan pengguna Spotify.

## ⚙️ ETL Pipeline (`ETL.ipynb`)

Dibangun menggunakan **PySpark** untuk menangani data berskala besar (`chart_song_daily.parquet`, `songs.parquet`). Tahapan utama:

1. **Load** — load data chart harian dari Google Drive ke Spark DataFrame.
2. **Data Cleaning**
   - Cek nilai null & duplikat
   - Cek distribusi kolom numerik & deteksi outlier (IQR method)
   - Penanganan missing value dengan strategi *coalesce* antara kolom asli dan kolom estimasi (`estimated_*`)
3. **Record Linkage** — memetakan variasi `uri` (`all_uris`) ke `canonical_uri` menggunakan tabel `songs.parquet`, agar lagu yang sama tidak dihitung sebagai entitas berbeda.
4. **Identifier Mapping** — menyatukan ID lagu (`uri`) yang sudah dikanonikalisasi ke seluruh dataset chart.
5. **Feature Engineering** — membuat fitur baru seperti `rank_change`, flag dari `entry_status`, dll pada level baris harian.
6. **Agregasi** — agregasi dari level harian ke level *track × country*, menghasilkan fitur seperti `total_streams`, `avg_daily_streams`, `num_days_observed`, `total_days_on_chart`, `max_consecutive_days`, dsb.
7. **Encoding** — *frequency encoding* untuk kombinasi artis (`artist_combo_frequency`) dan label (`label_catalog_size`, `label_avg_streams_loo`).
8. **Feature Selection** — membuang kolom yang sudah terwakili oleh fitur lain (kolom mentah artis, label, tanggal, dll).
9. **Standarisasi** — log transform untuk fitur yang *highly skewed*, lalu `StandardScaler` untuk seluruh fitur numerik.
10. **Handling Missing Value akhir** — imputasi nilai NaN sisa dengan median (`SimpleImputer`).
11. **Load (Export)** — menyimpan hasil akhir (`csd_model.parquet`) sebagai dataset siap untuk pemodelan (≈ 663.000 baris × 24 fitur).

## 🧩 Clustering (`Clustering.ipynb`)

Menggunakan **K-Means** dari PySpark MLlib pada dataset hasil ETL (`csd_final.parquet`):

1. Menyusun fitur menjadi vektor menggunakan `VectorAssembler` (mengeluarkan kolom identifier seperti `uri`, `country`, `artist_combo`).
2. Mencari jumlah cluster (K) optimal dengan menguji **K = 2 hingga 10**, dievaluasi menggunakan:
   - **Elbow Method** (Inertia / SSE)
   - **Silhouette Score**
3. Melatih model K-Means final dengan **K optimal = 8**.
4. Memvisualisasikan hasil cluster dalam ruang 2D menggunakan **PCA**, lengkap dengan titik centroid tiap cluster.
5. Membuat **profil rata-rata fitur per cluster** untuk interpretasi karakteristik masing-masing kelompok lagu.

## 🛠️ Tech Stack

- **Big Data Processing:** Apache Spark (PySpark, Spark MLlib)
- **Data Manipulation:** Pandas, NumPy
- **Visualisasi:** Matplotlib, Seaborn, missingno
- **Machine Learning:** Scikit-learn (StandardScaler, SimpleImputer, PCA), Spark MLlib (KMeans, ClusteringEvaluator)
- **Penyimpanan Data:** Parquet (PyArrow / fastparquet)
- **Sumber Dataset:** Kaggle (`kagglehub`)
- **Environment:** Google Colab + Google Drive

## 🚀 Cara Menjalankan

Seluruh notebook dirancang untuk dijalankan di **Google Colab** dengan Google Drive sebagai penyimpanan data.

1. Buka notebook melalui badge "Open In Colab" di masing-masing file, atau upload manual ke Colab.
2. Pastikan Google Drive sudah dimount dan struktur folder berikut tersedia di Drive:
   ```
   /MyDrive/spotify/
   ├── chart_song_daily.parquet
   ├── songs.parquet
   ├── csd_model.parquet      (output ETL)
   └── csd_final.parquet      (input Clustering)
   ```
3. Jalankan notebook secara berurutan:
   1. `EDA.ipynb` — eksplorasi awal dataset mentah dari Kaggle.
   2. `ETL.ipynb` — proses transformasi data hingga menghasilkan dataset siap model.
   3. `Clustering.ipynb` — pemodelan clustering pada dataset hasil ETL.

## 📁 Alur Data

```
Kaggle Dataset (CSV, ±42 juta baris)
        │
        ▼
   EDA.ipynb  ──────────► insight & pemahaman data
        │
        ▼
chart_song_daily.parquet + songs.parquet
        │
        ▼
   ETL.ipynb  ──────────► cleaning, record linkage, feature engineering,
        │                  agregasi, encoding, scaling
        ▼
   csd_model.parquet / csd_final.parquet
        │
        ▼
 Clustering.ipynb ──────► K-Means, evaluasi K, visualisasi PCA, profil cluster
```

## 👤 Author

Repository: [Rudi1410/BigData_Spotify](https://github.com/Rudi1410/BigData_Spotify)