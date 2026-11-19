# Jurnal Proses — Tugas 9

## Diskusi deployment model
- **Opsi yang dipertimbangkan:** Public Cloud, Private Cloud, Hybrid Cloud, dan Community Cloud.
- **Public Cloud murni** paling cepat dicoret. Elastisitasnya memang paling bagus untuk lonjakan pemilu dan PPDB, tetapi data biometrik dan NIK akan berada di infrastruktur resource pooling milik pihak ketiga. Ini bertentangan dengan syarat studi kasus bahwa data harus berada di infrastruktur yang dikendalikan penuh negara, dan dengan kewajiban pemrosesan data di wilayah Indonesia (PP 71/2019).
- **Private Cloud murni** sempat menjadi kandidat kuat karena paling aman dari sisi kendali. Masalahnya ada di rapid elasticity: kapasitas harus dibangun untuk puncak pemilu yang hanya terjadi sesekali, sehingga sebagian besar server menganggur hampir sepanjang tahun.
- **Community Cloud** dibahas karena penggunanya sesama instansi pemerintah. Opsi ini ditolak karena bank swasta juga menjadi pengguna utama (KYC), dan model ini tetap tidak menyelesaikan masalah lonjakan.
- **Keputusan: Hybrid Cloud.** Titik baliknya adalah ketika kami sadar bahwa kerahasiaan dan skalabilitas tidak harus dipenuhi oleh komponen yang sama. Data disimpan di private segment, sedangkan lapisan akses yang perlu diskalakan ditempatkan di public segment.
- **Perdebatan yang muncul:** apakah public segment benar-benar "tidak menyentuh data pribadi". Kami menyimpulkan bahwa NIK tetap melintas di public segment saat permintaan KYC. Karena itu kami menambahkan batasan desain: NIK tidak disimpan, kunci cache memakai HMAC (bukan hash biasa, karena ruang NIK kecil), log dimasking, dan payload dienkripsi di tingkat field.

## Diskusi service model per komponen
- ...

## Log Penggunaan AI (Level 2)

> Wajib diisi sesuai kebijakan Level 2 di [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Tulis "Tidak memakai AI" pada baris pertama jika memang tidak dipakai. Hanya untuk brainstorming ide/outline — bukan untuk kode/analisis/teks akhir.

| Tanggal | Tool AI | Prompt yang diberikan | Ringkasan saran/ide AI | Bagaimana diolah jadi tulisan/kode sendiri |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |
