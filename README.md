# Audit Teknis sdpcore.com

Hasil audit read-only terhadap https://sdpcore.com (PT. Swarna Dwipa Property — aplikasi ERP/backoffice internal developer perumahan, Laravel + AdminLTE 3).

## Isi repo

- `laporan-audit-sdpcore.md` — laporan audit umum: ringkasan situs, halaman publik, katalog 19 grup modul admin, daftar endpoint/route yang teramati, mekanisme auth, tech stack, temuan & rekomendasi, batasan audit.
- `deep-dive-siteplan-transaksi.md` — galian mendalam dua modul: arsitektur siteplan interaktif (teknik render SVG, sumber data peta, pemetaan warna status, modal detail kavling, export PDF/JPG server-side, tab lokasi) dan pipeline transaksi lengkap (diagram alur, field tiap form, 8 status progres + warna, operasi pindah unit/ganti nama/cancel).

## Metode

Eksplorasi browser live dengan akun superadmin (diberikan pemilik untuk audit ini), seluruhnya read-only: tidak ada form yang disubmit, tidak ada data yang diubah/dihapus.

## Batasan

- Kode server (PHP/Laravel) tidak dapat diambil dari sebuah website — yang dipetakan adalah aset frontend, struktur route, dan permukaan API.
- URL AJAX DataTables tidak teridentifikasi (capture network tidak tersedia pada tooling yang dipakai).
- Data pribadi yang terlihat selama audit (nama/NIK/telepon customer, nomor rekening) sengaja tidak direproduksi di dokumen ini.

## Catatan

Dokumen di repo ini adalah hasil analisis orisinal. Kode, aset, dan data milik PT. Swarna Dwipa Property tidak disertakan.
