# UTS Data Acquisition & Management — Geografi, Administrasi Wilayah, dan Iklim Kecamatan Kintamani, Kabupaten Bangli

Ujian Tengah Semester **Data Acquisition & Management** (Semester Ganjil 2026)
Program Studi Informatika dan Teknologi — INSTIKI
Dosen: Dr. I Gede Totok Suryawan, S.Kom., M.T.

| | |
|---|---|
| **Nama** | Putu Intan Cahyanti Putri |
| **NIM** | 2501010046 |
| **Notebook pipeline (Colab)** |https://colab.research.google.com/drive/1yugo7k8jEzIkQtDuuAFZPKUs0xC_8-Yz#scrollTo=Tr_LzUDlzKbe |
| **Repositori** | https://github.com/intancahyanti341-hue/UTS_Putu_Intan_Cahyanti_Putri.git |

---

## 1. Topik

**Geografi, administrasi wilayah, dan iklim Kecamatan Kintamani, Kabupaten Bangli, Bali.**

Pertanyaan yang ingin dijawab dengan data gabungan: bagaimana posisi Kabupaten Bangli dalam kode wilayah administrasi Indonesia, desa apa saja yang berada di Kecamatan Kintamani, dan seperti apa kondisi cuaca (suhu dan curah hujan) di Kintamani pada periode 1–14 Januari 2024?

---

## 2. Tiga Sumber Data

| # | Cara akuisisi | Sumber | Isi | Lisensi | Berkas mentah |
|---|---|---|---|---|---|
| 1 | **Unduh berkas dataset terbuka** | [ISI: URL sumber dataset] | Kode wilayah kabupaten/kota Indonesia, 514 baris × 3 kolom | Public Domain / Open Data (menurut catatan checklist) | `data/raw/dataset_wilayah_indonesia_raw.csv` |
| 2 | **API** | [Open-Meteo Historical Weather API](https://open-meteo.com/) (`archive-api.open-meteo.com`), tanpa API key | Cuaca harian historis Kintamani: suhu maks, suhu min, curah hujan (1–14 Januari 2024) | CC BY 4.0 (atribusi Open-Meteo.com) | `data/raw/api_cuaca_kintamani_raw.json` |
| 3 | **Web scraping halaman statis** | [Wikipedia ID — Kintamani, Bangli](https://id.wikipedia.org/wiki/Kintamani,_Bangli) | Daftar desa di Kecamatan Kintamani (48 desa) | CC BY-SA 4.0 | `data/raw/scraping_desa_kintamani_raw.csv` |

Tidak ada data dari Booking.com, Agoda, Traveloka, Airbnb, maupun platform komersial lain.

### Provenance dan integritas (MD5)

Berkas mentah di `data/raw/` disimpan **apa adanya** dan tidak diedit. Fungsi `hitung_md5()` di notebook menghitung checksum MD5 tiap berkas.

| Berkas | Ukuran (byte) | MD5 |
|---|---:|---|
| `dataset_wilayah_indonesia_raw.csv` | 14.890 | [ISI: hasil `hitung_md5()`] |
| `api_cuaca_kintamani_raw.json` | [ISI] | [ISI] |
| `scraping_desa_kintamani_raw.csv` | [ISI] | [ISI] |

---

## 3. Struktur Repositori

```
UTS_Putu_Intan_Cahyanti_Putri/
├── notebook/
│   └── UTS_DAM_Kintamani.ipynb                 # pipeline Soal 1–5
├── README.md
└── data/
    ├── raw/                                    # data mentah, tidak diubah
    │   ├── dataset_wilayah_indonesia_raw.csv
    │   ├── api_cuaca_kintamani_raw.json
    │   └── scraping_desa_kintamani_raw.csv
    └── hasil_penggabungan_kintamani.csv        # hasil gabungan ketiga sumber
```

---

## 4. Ringkasan Pengerjaan per Soal

### Persiapan — Direktori dan etika akses web

Sel pertama membuat folder `data/raw/` dan `notebook/`, lalu menyiapkan konfigurasi akses yang sopan:

- **User-Agent transparan:** `INSTIKI-DataAcquisition-UTS/1.0 (https://instiki.ac.id; mailto:2501010046@std.instiki.ac.id)`.
- **Session dengan `Retry`:** `total=3`, `backoff_factor=2`, untuk status 429, 500, 502, 503, 504 (hanya metode GET).
- **Pemeriksaan `robots.txt`** lewat `urllib.robotparser` sebelum scraping.
- **Fungsi `hitung_md5()`** untuk checksum berkas.

### Soal 1 — Pembacaan berkas, identifikasi data, estimasi storage

| Aspek | Hasil |
|---|---|
| Jenis data | **Terstruktur** (CSV tabular, skema kolom tetap) |
| Jumlah baris × kolom | 514 × 3 |
| Kolom | `kode_kabupaten` (int64), `kode_provinsi` (int64), `nama_kabupaten` (object) |
| Satuan | Kabupaten/kota (satu baris per kabupaten/kota; 514 nama unik) |
| Wilayah | Seluruh Indonesia |
| Ukuran di disk | 14.890 byte (±14,54 KB) |
| Pemakaian RAM | 43.292 byte (±42,28 KB) |
| Nilai hilang | 0 pada semua kolom |
| PII | Tidak ditemukan (pemindaian nama kolom: `nik`, `nama_orang`, `email`, `telepon`, `alamat_pribadi`, `gender`) |

**Estimasi storage 1 tahun**

Asumsi: tiap bulan menyimpan satu salinan berkas dengan ukuran sama, ditambah cadangan 30% untuk backup dan metadata.

- Berkas CSV: 14.890 B × 12 = 178.680 B, lalu × 1,3 = **232.284 B (±226,8 KB, ±0,22 MB)**

### Soal 2 — Akuisisi lewat API (Open-Meteo)

- **Endpoint:** `https://archive-api.open-meteo.com/v1/archive`
- **Lokasi:** Kecamatan Kintamani, latitude `-8.2417`, longitude `115.3667`
- **Periode:** 1–14 Januari 2024 (14 hari)
- **Variabel harian:** `temperature_2m_max`, `temperature_2m_min`, `precipitation_sum`
- **Zona waktu:** `Asia/Makassar` (WITA)
- **Penanganan request:** `timeout=15` detik, jeda 2 detik sebelum request, `raise_for_status()` untuk status non-200, dan retry otomatis pada kegagalan sementara. Tidak ada API key.
- **Keluaran mentah:** `data/raw/api_cuaca_kintamani_raw.json`

### Soal 3 dan 4 — Web scraping

- **Target:** halaman statis Wikipedia ID, [Kintamani, Bangli](https://id.wikipedia.org/wiki/Kintamani,_Bangli).
- **Etika:** `robots.txt` diperiksa terlebih dahulu (hasil: diizinkan), jeda 3 detik, `timeout=15`, dan User-Agent berisi email kampus. Hanya satu halaman yang diambil.
- **Ekstraksi:** semua tabel dibaca dengan `pd.read_html`. Tabel infobox (memuat "km²", "kepadatan", atau kolom ≤ 2) dilewati, lalu dipilih tabel yang memuat nama desa (`batur` dan `songan`). Teks dibersihkan dari awalan "Desa", entri yang mengandung angka dibuang, dan duplikat dihapus.
- **Cadangan:** jika hasil ekstraksi kurang dari 10 desa, notebook memakai daftar desa resmi Kecamatan Kintamani yang ditulis manual di kode.
- **Hasil:** **48 desa**, dengan kolom `nama_desa`, `kecamatan` (Kintamani), dan `kunci_kabupaten` (KABUPATEN BANGLI). Disimpan ke `data/raw/scraping_desa_kintamani_raw.csv`.

### Soal 5 — Penggabungan data

Ketiga sumber digabung dengan kunci `kunci_kabupaten` = `KABUPATEN BANGLI`, dalam dua tahap:

| Merge | Kiri | Kanan | Kunci | how | `validate` |
|---|---|---|---|---|---|
| 1 | Data wilayah Kabupaten Bangli (1 baris, hasil filter dari 514 baris) | Ringkasan cuaca Kintamani (1 baris agregasi) | `kunci_kabupaten` | inner | `one_to_one` |
| 2 | Hasil merge 1 (1 baris) | Daftar desa Kintamani (48 baris) | `kunci_kabupaten` | inner | `one_to_many` |

Ringkasan cuaca dihitung dari 14 hari data API:

| Indikator | Nilai |
|---|---:|
| Periode observasi | 2024-01-01 s.d. 2024-01-14 |
| Rata-rata suhu maksimum | 24,4 °C |
| Rata-rata suhu minimum | 17,59 °C |
| Total curah hujan | 41,9 mm |

Satu Kabupaten Bangli memuat banyak desa, sehingga relasi tahap kedua adalah `one_to_many` dan nilai wilayah serta cuaca berulang pada setiap baris desa.

**Bukti tidak ada duplikasi**

| Pemeriksaan | Hasil |
|---|---:|
| Baris data wilayah Bangli (assert harus tepat 1) | 1 |
| Baris daftar desa | 48 |
| Baris hasil akhir | 48 |
| Kolom hasil akhir | 10 |
| Baris duplikat penuh di hasil | 0 |

Keluaran: `data/hasil_penggabungan_kintamani.csv`. Notebook juga mencetak metadata tiap kolom (tipe data dan contoh nilai):

| Kolom | Tipe | Contoh |
|---|---|---|
| `kode_kabupaten` | int64 | 5106 |
| `kode_provinsi` | int64 | 51 |
| `nama_kabupaten` | object | KABUPATEN BANGLI |
| `kunci_kabupaten` | object | KABUPATEN BANGLI |
| `periode_observasi_cuaca` | object | 2024-01-01 s.d. 2024-01-14 |
| `rata_suhu_maks_c` | float64 | 24.4 |
| `rata_suhu_min_c` | float64 | 17.59 |
| `total_curah_hujan_mm` | float64 | 41.9 |
| `nama_desa` | object | Desa Abang Songan |
| `kecamatan` | object | Kintamani |

---

## 5. Data Ethics Checklist

**1. Kepatuhan sumber dan lisensi**
- [x] Tidak mengambil data dari situs akomodasi/booking komersial.
- [x] Memakai dataset terbuka asli wilayah administratif Indonesia (514 baris, lebih dari 100).
- [x] API publik resmi Open-Meteo, lisensi CC BY 4.0.
- [x] Web scraping dari Wikipedia Indonesia, lisensi CC BY-SA 4.0.

**2. Etika akses web**
- [x] `robots.txt` diperiksa sebelum request HTTP.
- [x] User-Agent berisi nama proyek, tautan institusi, dan email kampus.
- [x] Rate limiting: jeda 2–3 detik antar-request.
- [x] `timeout=15` detik dan exponential backoff retry.

**3. Privasi dan perlindungan data pribadi**
- [x] Seluruh dataset hanya memuat kode/nama wilayah, nama desa, dan parameter cuaca. Tidak ada data pribadi individu.

**4. Integritas, provenance, dan penyimpanan**
- [x] Data mentah disimpan utuh di `data/raw/`.
- [x] Penggabungan memakai `validate='one_to_one'` dan `validate='one_to_many'`.
- [x] Checksum MD5 dihasilkan oleh kode (`hitung_md5()`).

---

## 6. Keamanan dan Etika

- Tidak ada API key atau token di notebook. Open-Meteo tidak memerlukan key.
- Setiap request memakai **timeout**, **User-Agent berisi email kampus**, dan **jeda** antar-request.
- Data mentah tidak dimodifikasi. Seluruh pengolahan dilakukan pada salinan DataFrame di memori.

## 7. Keterbatasan

- Data cuaca hanya mencakup **14 hari** (1–14 Januari 2024) di satu titik koordinat, sehingga belum mewakili iklim Kintamani sepanjang tahun. Koordinat adalah titik perkiraan kecamatan.
- Dataset wilayah hanya memuat kode dan nama kabupaten/kota, tanpa data tingkat kecamatan atau desa.
- Daftar desa dari Wikipedia mengikuti data sumber saat diambil. Jika tabel tidak terbaca, notebook memakai daftar cadangan manual (lihat Soal 3 dan 4).
- Kunci gabung berada di tingkat kabupaten, sehingga nilai cuaca Kintamani dan data wilayah Bangli berulang pada setiap desa dan bukan nilai khusus per desa.

## 8. Atribusi

- Dataset wilayah: [ISI: nama penyedia dataset dan URL].
- Data cuaca: [Open-Meteo.com](https://open-meteo.com/), CC BY 4.0.
- Daftar desa: [Wikipedia ID](https://id.wikipedia.org/), CC BY-SA 4.0. Karena share-alike, turunan data dari Wikipedia di repositori ini mengikuti lisensi yang sama.
