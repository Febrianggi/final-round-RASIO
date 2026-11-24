# Analisis Multilevel Prestasi Matematika Siswa

[English](README.md) · **Bahasa Indonesia**

Dashboard interaktif dan ringkasan analisis multilevel (linear mixed model) untuk babak final olimpiade statistika RASIO. Kasusnya menanyakan apa yang memengaruhi nilai matematika siswa ketika siswa bersarang di dalam sekolah, dan apakah intervensi sebaiknya menyasar siswa, sekolah, atau keduanya.

## Kasus

Sebuah lembaga melihat bahwa prestasi matematika sangat berbeda antarsekolah. Survei memilih 30 sekolah secara acak dan 30 siswa dari setiap sekolah, sehingga ada 900 siswa dalam hierarki dua level. Siswa di sekolah yang sama berbagi fasilitas, guru, dan lingkungan akademik, jadi nilai mereka tidak saling bebas. Regresi biasa akan membuat galat baku variabel tingkat sekolah terlalu kecil.

Dataset disediakan oleh panitia lomba dan tidak disertakan di repositori ini.

| Level | Variabel | Keterangan |
|-------|----------|------------|
| 1, siswa | math_score (respons) | Nilai tes matematika |
| 1, siswa | prior_score | Kemampuan matematika sebelum pembelajaran |
| 1, siswa | ses | Status sosial ekonomi keluarga |
| 1, siswa | study_hours | Rata-rata jam belajar matematika per minggu |
| 1, siswa | gender | Laki-laki atau perempuan |
| 2, sekolah | facility | Kualitas fasilitas belajar |
| 2, sekolah | teacher_exp | Lama pengalaman guru matematika (tahun) |

Tidak ada nilai hilang maupun baris duplikat. Prediktor level 1 distandardisasi (z-score) atas 900 siswa dan prediktor level 2 atas 30 sekolah, sehingga koefisien dibaca sebagai efek per 1 SD.

## Pertanyaan penelitian dan jawabannya

1. Berapa bagian variasi yang terletak antarsiswa dan berapa antarsekolah?
   Sekitar 8,2% antarsekolah dan 91,8% antarsiswa (ICC = 0,082). Design effect 3,38, jadi model multilevel diperlukan.
2. Apakah karakteristik siswa dapat menjelaskan prestasi?
   Ya. SES, kemampuan awal, jam belajar, dan gender semuanya signifikan, dan bersama-sama menjelaskan 98,6% varians di dalam sekolah. SES adalah prediktor terkuat.
3. Apakah karakteristik sekolah dapat menjelaskan perbedaan antarsekolah?
   Sebagian. Pengalaman guru adalah prediktor positif yang robust. Kualitas fasilitas bernilai positif tetapi tidak konsisten signifikan.
4. Setelah memperhitungkan kedua level, apakah perbedaan antarsekolah masih tersisa?
   Ya. Sekitar 62% varians sekolah awal masih tersisa, dan sisa ini signifikan.

## Model

Semua model adalah linear mixed model dengan random intercept sekolah, diestimasi di Python dengan `MixedLM` dari statsmodels. REML dipakai untuk estimasi, sedangkan ML untuk uji rasio likelihood, AIC, dan BIC.

| Model | Spesifikasi | Tujuan |
|-------|-------------|--------|
| M1 Null | Intersep + random intercept sekolah | Dekomposisi varians dan ICC |
| M2 Siswa | M1 + prior_score, ses, study_hours, gender | Pertanyaan 2 |
| M3 Lengkap | M2 + facility, teacher_exp | Pertanyaan 3 dan 4 |
| M4 Perluasan | M3 + random slope study_hours per sekolah | Heterogenitas efek jam belajar |

Pemeriksaan pendukung: OLS dan OLS dengan galat baku cluster-robust, regresi dua tahap pada rata-rata sekolah, refit M3 dengan menghapus satu sekolah bergantian (leave-one-school-out), penyaringan random slope untuk ses dan prior_score, serta model kontekstual eksploratif dengan rata-rata sekolah.

## Hasil utama

| Besaran | M1 | M2 | M3 | M4 |
|---------|----|----|----|----|
| Varians sekolah (tau00) | 3,165 | 2,340 | 1,957 | 1,876 |
| Varians siswa (sigma kuadrat) | 35,408 | 0,507 | 0,507 | 0,365 |
| AIC (ML) | 5808,4 | 2099,7 | 2096,3 | 1834,5 |
| BIC (ML) | 5822,8 | 2133,3 | 2139,5 | 1887,3 |

Efek tetap pada M3, per 1 SD:

| Prediktor | Koefisien | p |
|-----------|-----------|---|
| ses | 5,382 | < 0,001 |
| prior_score | 2,345 | < 0,001 |
| gender (laki-laki) | 0,422 | < 0,001 |
| study_hours | 0,332 | < 0,001 |
| teacher_exp | 0,702 | 0,010 |
| facility | 0,478 | 0,081 |

Kekokohan efek sekolah:

- teacher_exp signifikan di keempat spesifikasi (M3, M4, OLS cluster-robust, dua tahap) dan di 29 dari 30 refit leave-one-school-out.
- facility hanya signifikan di M4 dan di 2 dari 30 refit leave-one-school-out, jadi buktinya belum konsisten. Dengan hanya 30 sekolah, kuasa uji di level 2 terbatas, dan "tidak signifikan" tidak berarti "tidak ada efek".

Random slope untuk study_hours jelas diperlukan (LRT = 265,8, p < 0,001). Rata-rata slope 0,310 per SD dengan SD antarsekolah 0,384, jadi manfaat tambahan jam belajar sangat berbeda antarsekolah. Random slope untuk ses dan prior_score tidak didukung data.

## Diagnostik

Residual dan efek acak sekolah lolos uji normalitas (Shapiro-Wilk), hubungannya tampak linear, dan VIF terbesar 1,10. M3 gagal pada uji homoskedastisitas antarsekolah (Levene p < 0,001), sejalan dengan efek jam belajar yang berbeda antarsekolah. M4 mengatasinya (Levene p = 0,348).

## Rekomendasi

- Prioritaskan dukungan di tingkat siswa, terutama bagi siswa dengan SES rendah dan kemampuan awal yang lemah.
- Investasikan pada kualitas dan pengalaman guru, faktor sekolah yang paling robust.
- Rancang program jam belajar secara spesifik per sekolah, bukan satu program untuk semua.
- Cari faktor sekolah lain, karena sebagian besar varians antarsekolah belum terjelaskan.

## Keterbatasan

- Data observasional, sehingga hasilnya berupa asosiasi, bukan efek kausal.
- Hanya 30 sekolah, sehingga kuasa uji terbatas dan efek tingkat sekolah peka terhadap sekolah tertentu.
- Homoskedastisitas antarsekolah baru terpenuhi setelah random slope ditambahkan di M4.
- Inferensi M4 bergantung pada korelasi intersep-slope sebesar 0,96 yang diestimasi dari 30 sekolah.
- Model kontekstual dengan rata-rata sekolah bersifat eksploratif.

## Dashboard

Dashboard berupa satu file HTML mandiri tanpa server atau proses build. Buka di browser modern mana pun. Font dan Chart.js sudah tertanam di dalam file. Isinya berupa tab untuk latar belakang, ringkasan, setiap model, eksplorasi data, setiap pertanyaan penelitian, slope jam belajar, diagnostik, kesimpulan, dan simulator yang memprediksi nilai dari efek tetap M3.

## Data

Data disediakan oleh panitia lomba. Periksa ketentuan panitia sebelum membagikan data mentah, dan perhatikan bahwa file dashboard memuat nilai per siswa yang dibutuhkan untuk grafiknya.
