# Tugas Hands-On 3 — Kasus 3: Transaksi E-Commerce

**Kasus yang dipilih:** Kasus 3 — `case3_ecommerce_transactions` (10.000 transaksi, 14 kolom, 2022–2024).
**Target model:** klasifikasi `status_retur` (Diretur ±22,5% vs Tidak Diretur).

## Isi folder
```
tugas-hands-on-3/
├── README.md
├── notebook.ipynb            # dijalankan dari atas ke bawah, output sudah tersimpan
├── data/case3_ecommerce_transactions.csv   # dataset asli (ekspor xlsx, isi tidak diubah)
├── output/
│   ├── data_bersih.csv       # hasil pembersihan
│   └── kamus_data.csv        # kolom, tipe, n_missing, n_unik (+ keterangan)
└── laporan_kualitas.csv      # tabel sebelum vs sesudah
```

## Cara menjalankan
```bash
pip install pandas numpy scipy scikit-learn matplotlib jupyter
cd tugas-hands-on-3
jupyter notebook notebook.ipynb     # lalu Kernel > Restart & Run All
```
Jalankan dari folder `tugas-hands-on-3/` karena path bersifat relatif. `SEED = 42` untuk semua proses acak.

## Ringkasan pembersihan (urutan: duplikat → missing → outlier → tipe data)
| Item | Sebelum | Sesudah | Catatan |
|---|---|---|---|
| Baris | 10.000 | 10.000 | |
| Duplikat (`transaction_id`) | 0 | 0 | tidak ditemukan |
| Missing (sel) | 2.177 | 1.792 | kategorikal diisi `Tidak Diketahui`; numerik/tanggal/`user_id` sengaja dibiarkan kosong dan diimputasi setelah split (tanpa leakage). Sisa termasuk 486 sel kolom turunan `total_nilai` |
| Outlier IQR `harga_idr` per kategori | 0 | 0 | tidak ada; harga berdistribusi hampir seragam |

Perbaikan lain: label `sentimen_ulasan` yang salah ketik/kapital (`Neg@tif`, `NETRAL`, dll., 11 nilai) dinormalkan menjadi 3 label; tipe data disesuaikan (`kuantitas` → Int64, kolom kategorikal → `category`).

## Tiga temuan
1. **Retur ditentukan pengalaman pembeli.** Rating ≤ 2 → 50,7% diretur (rating > 4: 5,0%); rating ≤ 2 hanya 24,8% transaksi tetapi 55,9% dari semua retur. *Aksi:* tindak lanjut CS otomatis untuk rating rendah / ulasan negatif sebelum retur diajukan.
2. **Kategori, metode bayar, kota, dan diskon tidak memengaruhi retur** (semua ±20–25%, chi-kuadrat p > 0,78). *Aksi:* diskon aman dari sisi retur; fokus pada kualitas pasca-pembelian, bukan pembatasan per segmen.
3. **Omzet stagnan (±Rp20 miliar/tahun, +1,1% lalu −2,0%) dan segmen "Berisiko hilang" memegang 19,8% omzet** (479 pelanggan, ±480 hari tidak belanja). 20% pelanggan teratas = 41,4% omzet. *Aksi:* kampanye win-back + loyalitas untuk Champions/Setia.

## Persiapan model
Split 80/20 dengan `stratify=y` (`random_state=42`) → `ColumnTransformer` (numerik: median imputer + indicator + StandardScaler; kategorikal: konstanta + OneHotEncoder) yang di-`fit` hanya pada train. Baseline regresi logistik: ROC-AUC 0,79 dengan semua fitur, 0,55 tanpa `rating_pembeli` & `sentimen_ulasan`. Kedua fitur itu baru tersedia setelah barang diterima, jadi untuk prediksi saat checkout skenario kedua yang realistis.

## Catatan
Data tampak sintetis (harga seragam, `nama_produk` tidak konsisten per `product_id`), sehingga temuan 2–3 bersifat deskriptif, bukan klaim kausal.
