# Pengelolaan Persediaan dan Pemantauan Penjualan Bulanan pada Toko Pertanian Multikanal

Bagian **Data Science** dari proyek capstone Studi Independen Asah by Dicoding. Repositori ini berisi proses pembersihan data (data wrangling) penjualan dan stok sebuah toko pertanian yang menjual lewat toko fisik, Shopee, Tokopedia, dan TikTok Shop. Hasilnya adalah tabel bersih yang dipakai untuk analisis, peramalan penjualan, dan sistem pemantauan stok.

> **Semua data dalam repositori ini adalah data dummy.** Nama produk, harga, nomor pesanan, dan transaksi dibuat untuk keperluan pembelajaran. Data ini bukan data transaksi nyata dari toko mana pun, dan tidak berisi data pribadi.

## Tujuan

- Membersihkan rekap penjualan offline, laporan penjualan marketplace, catatan stok masuk dan keluar, pemecahan kemasan, perakitan paket, dan hasil stock opname.
- Menyatukan semuanya menjadi tabel yang konsisten (satu SKU, satu penulisan, satu aturan tanggal) sehingga dapat dipakai tim lain.
- Mencatat setiap keputusan dan setiap baris yang diperbaiki atau ditolak, supaya hasil dapat ditelusuri.

## Data

- **Periode:** 1 November 2023 sampai 31 Oktober 2026 (36 bulan).
- **Kanal:** Offline (sejak Nov 2023), Shopee dan Tokopedia (sejak Feb 2025), TikTok Shop (sejak Jun 2025).
- **Produk:** 190 SKU, terdiri dari 186 SKU yang dijual dan 4 induk karung 50 kg yang dicatat dalam kg.
- **Masalah data mentah** yang ditangani di notebook: format tanggal campuran, angka tertulis sebagai teks, salah ketik nama dan kode produk, SKU kosong, baris ganda, qty kosong atau negatif, subtotal tidak cocok, komisi kosong, nomor nota kosong, dan lain-lain.

## Struktur Folder

```
suburonline-data-science/
├── data/
│   ├── raw/          # data mentah, tidak diubah
│   ├── interim/      # hasil antara (dibuat ulang oleh notebook, tidak di-commit)
│   └── processed/    # tabel bersih untuk dipakai tim
├── notebooks/
│   └── 01_data_wrangling.ipynb
├── requirements.txt
└── README.md
```

## Cara Menjalankan

1. Pasang Python 3.10 atau lebih baru.
2. Pasang pustaka:
   ```
   pip install -r requirements.txt
   ```
3. Buka `notebooks/01_data_wrangling.ipynb`, pilih **Restart kernel**, lalu **Run all**.
4. Hasil tersimpan di `data/interim/` dan `data/processed/`. Notebook membuat kedua folder itu bila belum ada.

Notebook diuji dengan pandas 2.2.3 dan 3.0.5.

## Isi Notebook

| Bagian | Isi |
|---|---|
| Gathering | Membaca semua berkas mentah dan memeriksa bentuknya |
| Assessing | Tinjauan umum tiap tabel, lalu pengecekan masalah yang ditemukan |
| Keputusan | Tabel keputusan pembersihan (K1 sampai K16) beserta alasannya |
| Cleaning | Memperbaiki tanggal, angka, SKU, duplikat, dan nilai kosong; baris yang tidak bisa diperbaiki dengan aman ditolak dan dilaporkan |
| Pemetaan SKU | Mencocokkan teks produk ke SKU master, tanpa membuat SKU baru |
| Buku besar stok | Menyusun pergerakan stok dari stok awal, penjualan, restok, pemecahan, perakitan, dan penyesuaian opname |
| Validasi | Neraca baris (masuk = bersih + ditolak + dibuang), kunci SKU, kolom inti tidak kosong, rumus netto dan laba |
| Penyimpanan | Menulis tabel ke `data/processed/` |

## Tabel Hasil (`data/processed/`)

| Tabel | Isi |
|---|---|
| `products` | 190 SKU beserta kategori, induk, faktor konversi, harga, dan supplier utama |
| `channels` | Empat kanal penjualan |
| `suppliers` | Supplier (dummy) dan lead time asumsi |
| `bundle_components` | Komposisi paket |
| `product_price_history` | Harga jual dan harga beli per SKU per bulan |
| `transactions` | Satu baris per nota atau pesanan |
| `transaction_items` | Satu baris per barang terjual (59.946 baris) |
| `online_item_financials` | Rincian keuangan per barang untuk penjualan online (bruto, netto, komisi, laba) |
| `stock_movements` | Buku besar stok: semua pergerakan, qty positif masuk dan negatif keluar |
| `stock_opname` | Hasil hitung fisik beserta selisih dengan stok sistem |
| `marketplace_metrics` | Impresi, klik, pesanan, dan rating bulanan produk di marketplace |
| `product_mapping` | Teks produk dari berkas sumber dan SKU hasil pemetaannya |
| `penjualan_ambigu` | Penjualan yang ditolak karena nama produk dipakai beberapa ukuran |
| `laporan_ditolak` | Baris yang ditolak beserta berkas, baris, dan alasannya |
| `duplikat_per_berkas` | Jumlah duplikat yang dibuang per berkas |
| `data_status` | Batas data lengkap per kanal (`data_complete_until`) |

Catatan untuk pemakai data:

- Hanya baris bersih yang masuk ke tabel utama. Baris yang tidak valid dibuang atau ditolak dan dicatat di `laporan_ditolak`.
- Tanggal penjualan online adalah tanggal pesanan. Pengurangan stok untuk penjualan online dicatat pada akhir bulan laporan, sedangkan tanggal pesanan asli ada di kolom `order_date`.
- Baris dengan `order_no_generated = true` adalah nota offline yang nomornya kosong. Jumlah transaksi offline tidak boleh dihitung dari baris ini.
- Penjualan yang namanya dipakai beberapa ukuran dan tidak bisa dibedakan lewat harga tidak dimasukkan ke `transaction_items`. Dampaknya sekitar 0,36% omzet offline. Rinciannya ada di `penjualan_ambigu`.
- Nilai `lead_time_hari` pada `suppliers` dan sebagian harga adalah asumsi tim.

## Keterbatasan

- Pembersihan memakai aturan yang hanya bergantung pada isi berkas itu sendiri, sehingga aturan yang sama dapat dipakai saat admin meng-import berkas baru.
- Beberapa SKU masih bernilai stok negatif sesudah penyesuaian opname, karena barang keluar setelah opname terakhir lebih banyak daripada yang tercatat masuk. Hal ini dianalisis di tahap EDA, bukan diperbaiki di tahap wrangling.
- Rekap offline tidak mencatat kejadian di luar nota (misalnya barang hilang), sehingga selalu ada selisih kecil antara stok sistem dan stok fisik.

## Rencana Selanjutnya

- `02_eda`: eksplorasi penjualan per kanal, kategori, dan musim.
- `03_modeling`: peramalan penjualan bulanan per SKU dengan Random Forest Regressor, dibandingkan dengan baseline rata-rata bergerak 3 bulan.
- `04_evaluation`: evaluasi dengan pembagian data berdasarkan waktu dan metrik WAPE.

## Tim

Proyek capstone ini dikerjakan oleh tiga learning path: Data Science (repositori ini), Full-Stack Developer, dan Gen AI Engineer.