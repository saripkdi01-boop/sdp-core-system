# Laporan Audit Teknis — sdpcore.com (Read-Only)

**Tanggal audit:** 5 Oktober 2026, ~00:21–01:30 WITA
**Metode:** eksplorasi browser live dengan akun `master` (role SUPERADMIN). Seluruh eksplorasi read-only: tidak ada form yang disubmit, tidak ada data yang diubah/dihapus.

---

## 1. Ringkasan

- **Nama:** PT. Swarna Dwipa Property — aplikasi backoffice/ERP internal untuk developer perumahan (Bahasa Indonesia).
- **Skala data:** 4 lokasi perumahan, total **1.635 kavling/unit** — BUMI SAMARKAND (891), MADINAH CITY SQUARE V (259), VILLA MADINAH LAND (186), BARUGA REGENCY (299).
- **Tidak ada situs publik/landing page:** `https://sdpcore.com/` me-redirect (302) ke halaman login.
- **Arsitektur:** aplikasi multi-page server-rendered (bukan SPA). Backend Laravel, frontend template AdminLTE 3.

---

## 2. Halaman Publik

| Route | Keterangan |
|---|---|
| `GET /` | Redirect 302 → `/admin/login`. Tidak ada landing page. |
| `/admin/login` | Form login (username + password + token CSRF). Sukses → `/admin/dashboard`. |
| `/panduan-aplikasi` | Halaman dokumentasi mandiri (tanpa layout admin), 21 panduan per peran: Admin, Marketing, Proyek, Gudang, Keuangan, Legal, KPR, Penjualan. Butuh login. |
| `/unit-ready` | Di luar prefix `/admin` tetapi me-render layout admin penuh (sidebar + navbar sama) — inkonsistensi routing. |

---

## 3. Katalog Modul Admin (19 grup navigasi)

### 3.1 Beranda & Dashboard
- **Beranda** (`/admin/beranda`) — halaman sambutan "Halo, master" + area quick-action (kosong).
- **Dashboard** (`/admin/dashboard`) — kartu statistik: Total Unit 1635/1635, Booking 1, Wawancara 0, Akad 0. Statistik Penjualan per Lokasi (READY 1627, HOLD 6, BF 1; KPR 1), Statistik Unit Ready, Statistik Status Progres, Statistik Penggunaan Bank (kosong), Statistik Penjualan Marketing (24 marketing, 1 berpenjualan), Statistik Penjualan Admin Pemberkasan (8 staf, 1 berpenjualan). Tautan Detail → `/admin/dashboard/lokasi-penjualan/{1..4}` (tabel server-side 891/259/186/299 baris).

### 3.2 Siteplan
- 7 peta siteplan interaktif per lokasi (tab BSK / MCS 5 / VILLA / BARUGA, ~890 titik kavling) dengan tombol **Cetak Denah PDF**, **Download Denah JPG**, **Reset Siteplan**.
- Varian: `siteplan-penjualan` (status jual: Booking / Ready / Booking Fee / On Proses Bank / SP3K / Akad / Serah Terima / Pembelian Cash), `siteplan-proyek` (progres: Belum Mulai / <100% / Selesai / Serah Terima), `siteplan-unit-ready` (Belum Mulai / Belum Ready), `siteplan-listrik` (Terpasang / Belum Terpasang).
- Tiga subvarian tercantum di navigasi dengan pola template sama (`siteplan-air`, `st-bphtb-ssp`, `st-balik-nama`) — tidak dibuka satu per satu.

### 3.3 Unit Ready — `/unit-ready`
Tabel 1.635 kavling (filter perumahan + status), kolom Status Ready / Keterangan, tombol Edit per baris.

### 3.4 Pengajuan Hold — `/admin/pengajuan-hold`
7 pengajuan berstatus Pending; aksi Edit / Lampiran / Verifikasi / Hapus.

### 3.5 Pembayaran — `/admin/pembayaran`
Data pembayaran per customer: rincian tagihan (Harga Rumah, Biaya Surat, Biaya Peningkatan Mutu, Booking Fee), kolom Tagihan / Sudah Bayar / Sisa Bayar, tautan Rekap Pembayaran.

### 3.6 Transaksi — `/admin/transaksi/*`
Submodul: wawancara, acc-bank, akad, pindah-unit, pembelian-cancel (2 baris — 1 baris data uji "aaaaa", 1 baris customer dengan tombol Batalkan disabled), ganti-nama, ppjb. Mayoritas tabel masih kosong; tiap halaman punya tombol tambah.

### 3.7 Customer — `/admin/customer/*`
- customer: 1 data (kolom Tanggal, Nama Nasabah, Marketing, Perumahan, Status Progres; aksi Edit / Upload File / Cetak Form Subsidi).
- prospek, upload-file (form pilih customer + tabel berkas), arsip-customer, aduan-customer, serah-terima-kunci.

### 3.8 Marketing — `/admin/marketing/*`
- marketing: 24 data (Kode / Nama / Alamat / Rekening / Status Aktif).
- admin-pemberkasan: 8 data.

### 3.9 Operasional Proyek (OP)
- **OP Bangunan:** proyek-bangunan (kosong), jenis-pekerjaan-bangunan (4 baris per perumahan + tautan Cek).
- **OP Jalan:** proyek-jalan (kosong), jalan (kosong), jenis-pekerjaan-jalan (1 baris "Pek. Finishing Jalan" 5%).
- **OP Saluran:** proyek-saluran (kosong), saluran (kosong), jenis-pekerjaan-saluran (1 baris "Pek. Finishing Saluran" 10%).

### 3.10 Legal — `/admin/legal/*`
listrik-air (kosong), pengajuan-berkas (1 baris: checklist dokumen IPH / SHGB / SSP / BPHTB / SIKUMBANG / DAFTAR SIKASEP / FOTO SIKASEP / TRILOGI — semua ✗), bphtb-ssp (kosong), balik-nama (kosong).

### 3.11 Keuangan — `/admin/keuangan/*`
pemasukan (1 baris: Rp1.000.000 Booking Fee via Bank BRI), pengeluaran (kosong), hutang (kosong), piutang (3 baris Rp0 "Belum Lunas"), kategori-transaksi (20 kategori Pemasukan/Pengeluaran), mutasi-saldo (kosong), laporan-arus-kas (filter tahun/bulan/rekening + Export PDF/Excel, kosong).

### 3.12 Pembelian & Barang
- `/admin/pembelian/input-po` (kosong), `/admin/pembelian/barang-masuk` (kosong).
- `/admin/barang-keluar` (kosong).

### 3.13 Master Data — `/admin/master/*`
perusahaan (5 PT), lokasi-kavling (4), kavling (1.635, server-side + Cetak PDF/Excel per lokasi), barang (kosong), supplier (kosong), satuan (3: UNIT/SET/PCS), bank-transaksi (3 rekening), bank-kpr (21 bank), notaris (1).

### 3.14 Pengaturan — `/admin/pengaturan/*`
- pengaturan-profil (form profil perusahaan).
- pengaturan-media (5 aset: logo, favicon, background).
- pengaturan-pengguna: 7 user (keuangan, gudang, legal, kpr, dev, proyek, master/SUPERADMIN).
- hak-akses: matriks izin per user (Lihat, Beranda, Tambah, Edit, Hapus).
- role-user: matriks izin per role.
- konten: 19 item konten CMS untuk website publik (Navbar, Slider, dsb.) — tetapi tidak tersaji di mana pun karena root hanya redirect ke login.
- list-penjualan: 8 status progres + warna.
- log-aktivitas: 206 entri login/logout, filter tanggal, server-side.

### 3.15 Panduan Aplikasi
Tautan ke `/panduan-aplikasi`.

---

## 4. Endpoint / Route yang Teramati

> Catatan: tooling browser tidak menyediakan capture network (DevTools tidak dapat dibuka), dan URL AJAX DataTables berada di skrip inline yang terpotong batas baca. Daftar ini hanya yang terverifikasi dari navigasi aktual dan sumber halaman.

**Auth & sesi**
- `GET /` → 302 ke `/admin/login`
- `GET /admin/login` — form login (username/password + CSRF `_token`)
- `POST` (AJAX) login dari form `/admin/login` — path persis tidak tercakup sumber terbaca; sukses → redirect `/admin/dashboard`
- `GET /refresh-csrf` — JSON `{token}`; dipanggil via fetch sebelum logout (pola serupa saat login) untuk menyegarkan token CSRF
- `POST /admin/logout` — action form logout (dengan `_token` baru)

**Navigasi halaman (full page load, bukan SPA)** — semua di bawah pola berikut:
- `/admin/beranda`, `/admin/dashboard`, `/admin/dashboard/lokasi-penjualan/{id}`
- `/admin/siteplan/*` (penjualan, proyek, unit-ready, listrik, air, bphtb-ssp, balik-nama)
- `/unit-ready`
- `/admin/pengajuan-hold`
- `/admin/pembayaran`
- `/admin/transaksi/*` (wawancara, acc-bank, akad, pindah-unit, pembelian-cancel, ganti-nama, ppjb)
- `/admin/customer/*` (customer, prospek, upload-file, arsip-customer, aduan-customer, serah-terima-kunci)
- `/admin/marketing/*` (marketing, admin-pemberkasan)
- `/admin/op-bangunan/*`, `/admin/op-jalan/*`, `/admin/op-saluran/*`
- `/admin/legal/*` (listrik-air, pengajuan-berkas, bphtb-ssp, balik-nama)
- `/admin/keuangan/*` (pemasukan, pengeluaran, hutang, piutang, kategori-transaksi, mutasi-saldo, laporan-arus-kas)
- `/admin/pembelian/*` (input-po, barang-masuk), `/admin/barang-keluar`
- `/admin/master/*` (perusahaan, lokasi-kavling, kavling, barang, supplier, satuan, bank-transaksi, bank-kpr, notaris)
- `/admin/pengaturan/*` (pengaturan-profil, pengaturan-media, pengaturan-pengguna, hak-akses, role-user, konten, list-penjualan, log-aktivitas)
- `/panduan-aplikasi`

**Aksi spesifik yang teramati**
- `GET /admin/customer/customer/cetak` — modal cetak data (target `_blank`, Excel/PDF)
- `GET /admin/customer/customer/tempo` — halaman Customer Tempo
- `GET /admin/customer/customer/{id}/edit` — form edit customer (via atribut `data-url`)
- `GET /admin/customer/customer/{id}/subsidi-cetak` — cetak form subsidi
- `GET /admin/customer/upload-file?id_customer={id}` — halaman upload berkas customer
- `GET /admin/master/kavling/cetak-pdf/__ID__` dan `.../cetak-excel/__ID__` — ekspor kavling per lokasi
- `GET /admin/master/kavling/{id}/edit` dan `GET /admin/master/kavling/{id}` — edit kavling dan lihat foto

**XHR DataTables server-side** (terpicu saat halaman dimuat — terlihat status "Loading..." lalu data terisi; URL ajax persisnya tidak tercakup): `/unit-ready`, `/admin/master/kavling`, `/admin/pengaturan/log-aktivitas`, `/admin/dashboard/lokasi-penjualan/{id}`.

**Mekanisme auth**
- Session berbasis cookie (khas Laravel); sesi bertahan di semua navigasi.
- Proteksi CSRF: meta `csrf-token` + field `_token` tersembunyi di form + endpoint `GET /refresh-csrf`; AJAX memakai header `X-CSRF-TOKEN`.
- Tidak ditemukan bearer token / JWT di lalu lintas yang teramati.
- Kontrol akses berbasis role: 7 role (Admin, Keuangan, Gudang, Legal, KPR, Proyek + SUPERADMIN) dengan matriks izin per menu (Lihat, Beranda, Tambah, Edit, Hapus).

---

## 5. Tech Stack

- **Backend:** Laravel (pola Blade, CSRF, route helper) — server-rendered multi-page app.
- **Frontend:** AdminLTE 3; jQuery + jQuery UI; Bootstrap 4; DataTables (+ Buttons, jszip, pdfmake); Select2; SweetAlert2; Toastr; Chart.js; Summernote; daterangepicker; Tempusdominus; overlayScrollbars; fontawesome-iconpicker.
- **Aset:** statis di `/templates/…`, media di `/config_media/…`, audio notifikasi `/audio/notification.ogg`.
- Tidak ada framework JS modern (Next.js/React/Vue) atau build-id yang terdeteksi.

---

## 6. Temuan & Rekomendasi

1. **Inkonsistensi zona waktu** di Log Aktivitas: teks aktivitas memakai WIB (mis. "login … jam 23:22") sedangkan kolom Tanggal/Jam memakai UTC ("04 Oktober 2026" / "16:22") — padahal pengguna di WITA. Normalisasi ke satu zona waktu.
2. **Bug JS** di `/admin/pengaturan/role-user`: pager menampilkan "Showing 0 to 0 of 0 entries (filtered from NaN total entries)".
3. **Data uji tersisa** di database: baris "aaaaa" di Pembelian Cancel dan Hutang; placeholder "contoh…"; 1 customer dengan NIK/telepon yang tampak fiktif.
4. **Pengungkapan path server:** field "Folder SVG" di Pengaturan Profil berisi `/home/sidd4282/public_html/property.aplikasikavling.com` — membocorkan username hosting dan domain lama; sebaiknya disamarkan/dihapus.
5. `/unit-ready` berada di luar prefix `/admin` namun me-render layout admin penuh — inkonsistensi routing/prefix.
6. Modul Konten mengelola konten website publik (Navbar/Slider/Produk/Siteplan/Kontak), tetapi root `/` hanya redirect ke login — frontend publik tampaknya nonaktif/dipindah; konten CMS tidak tersaji di mana pun yang teramati.
7. **Kredensial superadmin sangat lemah** (kredensial bawaan umum `master`/`admin`) — disarankan diganti segera setelah audit selesai.
8. Tidak ditemukan halaman 404/rusak: seluruh 60+ tautan navigasi berhasil dimuat.
9. **Catatan privasi:** selama katalogisasi terlihat data pribadi (nama/NIK/telepon customer, nomor rekening bank) — sengaja tidak direproduksi di laporan ini; hanya nama kolom/fitur yang dicatat.

---

## 7. Batasan Audit

- **Kode server (PHP/Laravel) tidak dapat diambil dari sebuah website** — yang bisa diobservasi hanya aset frontend (HTML/CSS/JS), struktur route, dan permukaan API. Untuk audit kode backend diperlukan akses ke repository atau server.
- URL AJAX DataTables tidak teridentifikasi karena capture network tidak tersedia pada tooling browser yang dipakai; daftar endpoint di §4 mencakup route navigasi dan aksi yang terverifikasi dari sumber halaman.
- Tiga subvarian siteplan (`siteplan-air`, `st-bphtb-ssp`, `st-balik-nama`) tercantum di navigasi dengan pola template yang sama tetapi tidak dibuka satu per satu.
