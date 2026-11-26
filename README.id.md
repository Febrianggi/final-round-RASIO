# Analisis Multilevel Prestasi Matematika Siswa

[English](README.md) · **Bahasa Indonesia**

Analisis multilevel (*linear mixed model*) dan dashboard HTML interaktif yang mengkaji faktor penentu nilai matematika pada 900 siswa yang bersarang di 30 sekolah, serta menjawab apakah intervensi sebaiknya menyasar siswa, sekolah, atau keduanya.

- **File dashboard interaktif:** [`index.html`](index.html)
- **Notebook analisis & data:** [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) · [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx)

## Pertanyaan penelitian dan jawabannya

1. **Berapa bagian variasi yang terletak antarsiswa dibanding antarsekolah?**
   Sekitar **8,2%** berada antarsekolah dan **91,8%** antarsiswa (`ICC = 0,082`, *design effect* `3,38`), sehingga model multilevel diperlukan.
2. **Apakah karakteristik siswa menjelaskan prestasi matematika?**
   Ya. `ses`, `prior_score`, `study_hours`, dan `gender` semuanya signifikan (`p < 0,001`) dan bersama-sama menjelaskan **98,6%** varians di dalam sekolah. `ses` adalah prediktor terkuat.
3. **Apakah karakteristik sekolah menjelaskan perbedaan antarsekolah?**
   Sebagian. `teacher_exp` merupakan prediktor positif yang konsisten di berbagai spesifikasi, sedangkan `facility` bernilai positif tetapi tidak konsisten signifikan.
4. **Apakah perbedaan antarsekolah masih tersisa setelah memperhitungkan kedua level?**
   Ya. Sekitar **62%** varians sekolah awal masih tersisa dan signifikan (`p < 0,001`).

## Temuan utama

| Besaran | M1 (Null) | M2 (Siswa) | M3 (Lengkap) | M4 (+ Random slope) |
|---|---:|---:|---:|---:|
| Varians sekolah (`tau00`) | 3,165 | 2,340 | 1,957 | 1,876 |
| Varians siswa (`sigma^2`) | 35,408 | 0,507 | 0,507 | 0,365 |
| AIC (ML) | 5808,4 | 2099,7 | 2096,3 | 1834,5 |
| BIC (ML) | 5822,8 | 2133,3 | 2139,5 | 1887,3 |

Efek tetap pada M3 (per 1 SD tiap prediktor):

| Prediktor | Koefisien | p-value |
|---|---:|---:|
| `ses` | 5,382 | < 0,001 |
| `prior_score` | 2,345 | < 0,001 |
| `gender` (laki-laki) | 0,422 | < 0,001 |
| `study_hours` | 0,332 | < 0,001 |
| `teacher_exp` | 0,702 | 0,010 |
| `facility` | 0,478 | 0,081 |

- **Kekokohan efek sekolah:** `teacher_exp` tetap signifikan di keempat spesifikasi (M3, M4, OLS *cluster-robust*, dan regresi dua tahap rata-rata sekolah) serta pada **29 dari 30** refit *leave-one-school-out*. `facility` hanya signifikan di M4 dan **2 dari 30** refit; dengan 30 sekolah, kuasa uji level 2 terbatas, jadi tidak signifikan tidak berarti efeknya nol.
- **Manfaat jam belajar berbeda antarsekolah (M4):** Menambahkan *random slope* sekolah untuk `study_hours` memperbaiki kecocokan model secara tajam (`LRT = 265,8, p < 0,001`). Rata-rata slope `0,310` per SD dengan simpangan baku antarsekolah `0,384`, menandakan tambahan jam belajar memberi dampak berbeda di tiap sekolah. *Random slope* untuk `ses` dan `prior_score` tidak didukung data.
- **Diagnostik:** Residual dan efek acak sekolah lolos uji normalitas Shapiro-Wilk, hubungannya linear, dan VIF terbesar `1,10`. M3 gagal pada homoskedastisitas antarsekolah (`Levene p < 0,001`), lalu teratasi di M4 (`Levene p = 0,348`).

### Rekomendasi

- Prioritaskan dukungan di tingkat siswa, terutama bagi siswa berlatar belakang SES rendah dan kemampuan awal lemah.
- Investasikan pada kualitas dan pengalaman guru sebagai faktor sekolah yang paling konsisten.
- Rancang program jam belajar secara spesifik per sekolah, bukan satu pola seragam.
- Teliti faktor sekolah tambahan karena 62% varians sekolah awal belum terjelaskan.

## Cara membaca dashboard

[`index.html`](index.html) adalah dashboard HTML satu berkas yang mandiri (font dan Chart.js tertanam; tanpa proses build). Jelajahi ke-12 tab pada bilah navigasi atas:

| Kelompok tab | Tab di layar | Hal yang bisa dilihat |
|---|---|---|
| **Konteks & data** | **Background**, **Overview**, **Models**, **Data & EDA** | Desain survei (`30 sekolah × 30 siswa`), tabel variabel dua level, spesifikasi M1–M4, dan distribusi skor antarsekolah. |
| **Pertanyaan inti** | **1 Variance**, **2 Student traits**, **3 School traits**, **4 Residual variance** | Dekomposisi ICC (`8,2%` antarsekolah), efek tetap terstandar, uji kekokohan `teacher_exp` vs `facility`, dan sisa varians sekolah. |
| **Perluasan & uji** | **Study-hours slope**, **Diagnostics**, **Conclusions** | Variasi slope `study_hours` per sekolah (M4), pemeriksaan normalitas/homoskedastisitas, dan grafik nilai aktual vs prediksi (`r = 0,994`). |
| **Alat interaktif** | **Simulator** | Geser slider `ses`, `prior_score`, `study_hours`, `facility`, `teacher_exp`, dan **Gender** untuk melihat prediksi `math_score` beserta kontribusi tiap variabel dibanding siswa rata-rata di sekolah rata-rata (efek tetap M3, efek acak sekolah = 0). |

## Data dan model

Survei mengambil 30 sekolah dan 30 siswa per sekolah (total 900 siswa) tanpa nilai hilang maupun duplikat. Karena siswa di sekolah yang sama berbagi guru dan fasilitas, regresi biasa membuat galat baku variabel level 2 terlalu kecil.

| Level | Variabel | Keterangan |
|---|---|---|
| 1, siswa | `math_score` (respons) | Nilai tes matematika |
| 1, siswa | `prior_score` | Kemampuan matematika sebelum pembelajaran |
| 1, siswa | `ses` | Status sosial ekonomi keluarga |
| 1, siswa | `study_hours` | Rata-rata jam belajar matematika per minggu |
| 1, siswa | `gender` | Laki-laki atau perempuan (`Laki-laki` / `Perempuan` di berkas Excel) |
| 2, sekolah | `facility` | Kualitas fasilitas belajar |
| 2, sekolah | `teacher_exp` | Lama pengalaman guru matematika (tahun) |

Prediktor level 1 distandardisasi (z-score) atas 900 siswa dan prediktor level 2 atas 30 sekolah sehingga koefisien dibaca sebagai efek per 1 SD. Keempat model (M1 Null, M2 Siswa, M3 Lengkap, M4 + *random slope* `study_hours`) diestimasi di Python dengan `statsmodels` `MixedLM` (REML untuk estimasi; ML untuk LRT, AIC, dan BIC), dilengkapi pemeriksaan OLS *cluster-robust*, regresi dua tahap, *leave-one-school-out*, dan model kontekstual eksploratif. Di [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx), nama kolom memakai titik (`study.hours`, `prior.score`, `teacher.exp`, `math.score`) lalu diubah notebook menjadi garis bawah.

## Keterbatasan

- Data observasional, sehingga hasil menunjukkan asosiasi dan bukan efek kausal.
- Jumlah sekolah hanya 30, membatasi kuasa uji level 2 dan membuat efek sekolah peka terhadap sekolah tertentu.
- Homoskedastisitas antarsekolah baru terpenuhi setelah *random slope* ditambahkan pada M4.
- Inferensi M4 bergantung pada korelasi intersep-slope sebesar `0,96` yang diestimasi dari 30 sekolah.
- Model kontekstual dengan rata-rata sekolah bersifat eksploratif.

## Cara menjalankan di lokal dan reproduksi

### 1. Membuka dashboard di lokal

Jalankan server HTTP lokal dari folder utama repositori:

```bash
python -m http.server 8000
```

Buka http://localhost:8000 di browser (hentikan dengan `Ctrl+C`). Karena `index.html` menyematkan Chart.js dan font langsung di dalam file, halaman ini juga bisa dibuka offline langsung dari disk di browser modern mana pun.

### 2. Mereproduksi notebook analisis

Seluruh alur Python berada di [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb):

- **Google Colab:** Buka [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) lalu jalankan semua sel. Saat sel data meminta unggahan file, pilih [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx).
- **Jupyter lokal:** Letakkan `FINAL_RASIO.ipynb` dan `Final Round Data.xlsx` di folder yang sama, pasang paket bila perlu, lalu jalankan semua sel dari atas ke bawah:

```bash
pip install pandas openpyxl statsmodels scipy matplotlib seaborn
```

Notebook berhasil dijalankan bila bagian 3 mencetak `ICC = 0.0821` dan bagian 9 menghasilkan tabel ringkasan model M1–M4 yang identik dengan dashboard.

## Isi repositori

| File | Isi |
|---|---|
| [`index.html`](index.html) | Dashboard interaktif mandiri |
| [`FINAL_RASIO.ipynb`](FINAL_RASIO.ipynb) | Notebook analisis Python lengkap dengan output |
| [`Final Round Data.xlsx`](Final%20Round%20Data.xlsx) | Dataset (900 siswa, 30 sekolah) |
| [`README.md`](README.md), [`README.id.md`](README.id.md) | Dokumentasi dalam bahasa Inggris dan Indonesia |
