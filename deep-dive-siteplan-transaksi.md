# Deep Dive: Siteplan Interaktif & Alur Transaksi — sdpcore.com

**Tanggal:** 5 Oktober 2026 (~00:31–00:35 WITA)
**Metode:** lanjutan audit read-only, sesi SUPERADMIN masih aktif (tanpa login ulang). Tidak ada form yang disubmit; tombol Hapus/Batalkan/Simpan tidak diklik.

---

## A. Arsitektur Siteplan Interaktif

Halaman yang dibedah: `/admin/siteplan/siteplan-penjualan`, dibandingkan dengan `siteplan-proyek`.

### A.1 Teknik render: SVG inline raksasa, bukan canvas
- Setiap tab lokasi me-render **satu SVG inline raksasa** di dalam halaman, contoh: `<svg id="svg-image-1" width="400mm" height="500mm" viewBox="0 0 40000 50000">`.
- Satu halaman memuat **4 SVG sekaligus** (satu per lokasi: BSK / MCS 5 / VILLA / BARUGA) — total **~1,56 juta karakter HTML** per halaman.
- Setiap kavling adalah elemen SVG: `<a class="detail-button" href="javascript:void(0);" data-id="{id}" data-url="https://sdpcore.com/admin/siteplan/siteplan-penjualan/{id}">` yang membungkus `<g>` berisi `<polygon style="fill:{warna_status}">` + `<text>` label kode kavling (mis. "C-7").
- Warna status di-**inject server-side** sebagai inline `style="fill:..."` pada tiap polygon (contoh: C-1 `fill:#ffffff`).
- Pan/zoom: drag dengan cursor grab + CSS transform via JS; tombol **"Reset Siteplan"** me-reset transform — murni client-side, tanpa request server.
- Artefak dev: komentar HTML `<!-- resources/views/admin/bank/index.blade.php -->` di awal body — nama view Blade yang tidak sesuai dengan halaman siteplan (indikasi copy-paste template).

### A.2 Sumber data peta
- Markup SVG di-inline langsung ke HTML oleh server — kemungkinan dibaca dari folder yang dikonfigurasi di **Pengaturan Profil → "Folder SVG"**. Tiap lokasi punya SVG sendiri (`svg-image-1` s.d. `svg-image-4`).

### A.3 Pemetaan warna → status jual (siteplan-penjualan)
Terverifikasi visual dan cocok dengan `/admin/pengaturan/list-penjualan`:

| Status | Warna |
|---|---|
| Booking | hijau |
| Ready | putih `#ffffff` |
| Booking Fee | kuning `#ffff80` |
| On Proses Bank | oranye `#ff7300` |
| SP3K | biru muda `#6ab5ff` |
| Akad | magenta `#fb00ff` |
| Serah Terima | merah marun `#800040` |
| Pembelian Cash | tosca `#00ffd5` |

- `siteplan-proyek` memakai legenda berbeda (progres konstruksi: Belum Mulai / <100% / Selesai / Serah Terima) — struktur DOM dan pola endpoint-nya **identik**, hanya overlay status yang berbeda. Pola yang sama dipakai ulang untuk `siteplan-unit-ready`, `siteplan-listrik` (Terpasang/Belum Terpasang), `siteplan-air`, `st-bphtb-ssp`, `st-balik-nama`.

### A.4 Klik titik kavling → modal "Detail Data Kavling"
Data diambil dari atribut `data-url` (pola `data-url` + modal sangat mengindikasikan AJAX GET, namun lalu lintas network tidak dapat di-capture — lihat §C). Isi modal 5 tab, semua readonly:

1. **Data Unit Rumah** — Perumahan, Kode Kavling, Panjang Kanan/Kiri, Lebar Depan/Belakang, Luas Tanah/Bangunan, Harga Jual, Daya Listrik, Keterangan, No. Sertifikat.
2. **Data Customer** — Nama Lengkap, No. KTP, No. Telp/WA, Tempat Lahir, Tanggal Lahir, Jenis Kelamin, Alamat KTP, Alamat Domisili, NPWP, Jenis Pembelian.
3. **Tagihan & Pembayaran** — tabel rincian tagihan + tabel pemasukan + total.
4. **Foto Unit** — gambar.
5. **Listrik & Air** — No. Rekening + foto (kosong pada unit yang diklik).

Tombol modal: Cetak Data, Close, Keluar.

### A.5 Export: server-side, bukan client-side
- **Cetak Denah PDF** → `GET /admin/siteplan/siteplan-penjualan/cetak/pdf/{id_lokasi}`
- **Download Denah JPG** → `GET /admin/siteplan/siteplan-penjualan/cetak/jpg/{id_lokasi}`
- Pola sama di siteplan-proyek: `/admin/siteplan/siteplan-proyek/cetak/{pdf|jpg}/{id_lokasi}`.

### A.6 Tab lokasi
Bootstrap pills (`data-toggle="pill"`) yang show/hide 4 pane SVG yang **sudah di-render server-side sekaligus** — murni client-side, tanpa reload halaman atau XHR saat ganti tab.

---

## B. Pipeline Transaksi

Sumber: `/admin/transaksi/*` + `/admin/pengaturan/list-penjualan` (kolom: No, Status Progres, Warna, Urutan, Keterangan).

### B.1 Diagram alur

```
Ready (unit tersedia, putih)
  → Booking Fee (kuning) — customer booking + bayar UTJ/booking fee
      modul: Pengajuan Hold, Customer, Pembayaran
  → Wawancara — /admin/transaksi/wawancara
      penjadwalan wawancara KPR (kolom: Customer, Lokasi Rumah, Tgl Wawancara, Bank KPR, Catatan)
  → ACC Bank — /admin/transaksi/akad... (lihat catatan)
      /admin/transaksi/acc-bank: plafon ACC, Tgl SP3K, Tgl Expired, Sisa Hari
      → status "On Proses Bank" (oranye)
  → SP3K — Pencairan Kredit Bank (biru muda)
  → Akad — /admin/transaksi/akad: PENJADWALAN batch
      (kolom: Jadwal Akad, Total Akad, Keterangan)
  → Serah Terima (marun) — /admin/customer/serah-terima-kunci

Jalur alternatif:
  → Pembelian Cash / Cash keras (tosca) — modul PPJB
      (Perjanjian Pengikatan Jual Beli; kolom: Tanggal PPJB, No. PPJB, Customer, Lokasi Rumah)
  → User Cancel / Batal (putih) — modul pembelian-cancel
      tombol "Batalkan" per baris; baris customer yang kavlingnya sudah
      berpindah pemilik tombolnya disabled ("Kavling sudah milik orang lain")

Operasi samping:
  • Pindah Unit — memindahkan customer dari kavling lama ke kavling baru
      (Biaya Administrasi + pilih Rekening + Metode Pembayaran + upload Bukti Pembayaran)
  • Ganti Nama — mengalihkan booking dari customer lama ke customer baru
      (input data lengkap customer baru + Biaya Ganti Nama + bukti bayar)
```

### B.2 Field form "Tambah Data Wawancara" (modal, tidak disubmit)
Hari (readonly, otomatis) · Tanggal · Pilih Customer (Select2) · NIK (auto-readonly) · Alamat KTP (auto) · Lokasi Rumah (auto) · Tipe Bangunan (auto) · Luas Tanah (auto) · Luas Bangunan (auto) · Marketing (auto) · Pilih Bank KPR · Catatan Wawancara (editor Summernote). Tombol: Batal / Simpan.

Pola penting: pilih customer → field identitas & unit **terisi otomatis** (relasi customer → kavling → marketing).

### B.3 Field form "Tambah Jadwal" Akad (modal, tidak disubmit)
Hanya **Tanggal** dan **Keterangan** — artinya modul Akad mengelola **jadwal akad (batch)**, bukan record akad per customer.

### B.4 Daftar 8 status progres + warna (dari list-penjualan)

| No | Status | Warna | Urutan | Keterangan |
|---|---|---|---|---|
| 1 | Ready | `#ffffff` (putih) | 1 | unit tersedia |
| 2 | Booking Fee | `#ffff80` (kuning) | 2 | — |
| 3 | On Proses Bank | `#ff7300` (oranye) | 3 | — |
| 4 | SP3K (Pencairan Kredit Bank) | `#6ab5ff` (biru muda) | 4 | — |
| 5 | Akad (Akad internal) | `#fb00ff` (magenta) | 5 | — |
| 6 | Serah Terima | `#800040` (marun) | 6 | — |
| 7 | User Cancel (Batal) | `#ffffff` (putih) | 7 | — |
| 8 | Pembelian Cash (Cash keras) | `#00ffd5` (tosca) | 8 | — |

Catatan: legenda siteplan-penjualan menampilkan **"Booking" (hijau)** sebagai tambahan — tidak ada di daftar 8 ini; kemungkinan status awal sebelum Booking Fee.

---

## C. Yang Tidak Bisa Diverifikasi

1. URL AJAX persis milik DataTables server-side (skrip inline terpotong batas baca tool).
2. Apakah klik kavling memakai AJAX GET ke `data-url` atau navigasi biasa — polanya sangat mengindikasikan AJAX GET, tetapi capture network tidak tersedia (DevTools tidak bisa dibuka).
3. Isi file SVG mentah di folder server.
4. Endpoint POST untuk simpan form (tidak diuji, sesuai batasan read-only).
