# UTS Data Acquisition & Management
## Akuisisi dan Integrasi Data Pariwisata Mancanegara di Indonesia

Mahasiswa : PUTU INTAN CAHYANTI PUTRI
NIM       : 2501010046
Kelas     : E
Program   : Informatika INSTIKI
Tanggal   : 2026-11-19 15:35:49 WIB

## Struktur Folder

- data/raw/
  - Tiga dataset CSV BPS
  - openmeteo_response.json
  - direktori_pariwisata.html
  - direktori_detail/ (HTML halaman detail jika tersedia)

- data/processed/
  - openmeteo_clean.csv
  - tautan_direktori_pariwisata.csv
  - direktori_pariwisata_clean.csv
  - bps_pintu_masuk_clean.csv
  - bps_paspor_clean.csv
  - bps_pengeluaran_clean.csv
  - pariwisata_terintegrasi.csv
  - bps_paspor_pengeluaran.csv

- documentation/
  - provenance.csv
  - data_ethics_checklist.xlsx
  - data_ethics_checklist.csv
  - metadata_gabungan.csv
  - storage_estimation.csv
  - request_log.csv
  - bukti_robots_direktori.txt

## Sumber Data

1. BPS:
   https://www.bps.go.id/id/statistics-table?subject=561

2. Open-Meteo:
   https://open-meteo.com/en/docs/historical-weather-api

3. Direktori Pariwisata Indonesia:
   https://www.direktoripariwisata.id/

## Metode Akuisisi

1. Download dataset CSV.
2. API menggunakan requests dan JSON.
3. Web scraping menggunakan requests dan BeautifulSoup.

## Catatan

Data mentah disimpan pada folder data/raw.
Data hasil pembersihan dan integrasi disimpan pada
folder data/processed.

Status lisensi dan ketentuan penggunaan Direktori Pariwisata
harus mengikuti hasil verifikasi dari website sumber.

File provenance digunakan untuk mencatat URL, tanggal akses,
metode, ukuran, dan hash MD5.
