# Jurnal Proses — Tugas 9

## Diskusi deployment model
- **Opsi yang dipertimbangkan:** Public Cloud, Private Cloud, Hybrid Cloud, dan Community Cloud.
- **Public Cloud murni** paling cepat dicoret. Elastisitasnya memang paling bagus untuk lonjakan pemilu dan PPDB, tetapi data biometrik dan NIK akan berada di infrastruktur resource pooling milik pihak ketiga. Ini bertentangan dengan syarat studi kasus bahwa data harus berada di infrastruktur yang dikendalikan penuh negara, dan dengan kewajiban pemrosesan data di wilayah Indonesia (PP 71/2019).
- **Private Cloud murni** sempat menjadi kandidat kuat karena paling aman dari sisi kendali. Masalahnya ada di rapid elasticity: kapasitas harus dibangun untuk puncak pemilu yang hanya terjadi sesekali, sehingga sebagian besar server menganggur hampir sepanjang tahun.
- **Community Cloud** dibahas karena penggunanya sesama instansi pemerintah. Opsi ini ditolak karena bank swasta juga menjadi pengguna utama (KYC), dan model ini tetap tidak menyelesaikan masalah lonjakan.
- **Keputusan: Hybrid Cloud.** Titik baliknya adalah ketika kami sadar bahwa kerahasiaan dan skalabilitas tidak harus dipenuhi oleh komponen yang sama. Data disimpan di private segment, sedangkan lapisan akses yang perlu diskalakan ditempatkan di public segment.
- **Perdebatan yang muncul:** apakah public segment benar-benar "tidak menyentuh data pribadi". Kami menyimpulkan bahwa NIK tetap melintas di public segment saat permintaan KYC. Karena itu kami menambahkan batasan desain: NIK tidak disimpan, kunci cache memakai HMAC (bukan hash biasa, karena ruang NIK kecil), log dimasking, dan payload dienkripsi di tingkat field.

## Diskusi service model per komponen
- **Opsi yang dipertimbangkan:** IaaS, PaaS, dan SaaS untuk empat kelompok komponen, yaitu database inti & biometrik, API Gateway beserta antrean dan cache, portal verifikasi & API internal, serta monitoring. Pembagian tugas hanya meminta minimal 3 komponen. Monitoring kami pisahkan karena data yang disentuhnya berbeda (hanya metrik dan ID transaksi), sehingga service model-nya juga bisa berbeda.
- **Database inti & biometrik** memakai IaaS di private segment. Managed database (DBaaS) sempat dibahas karena backup dan patching ditangani penyedia, tetapi itu berarti penyedia memegang akses admin ke mesin yang menyimpan NIK dan biometrik. Dengan IaaS, tim negara sendiri yang memegang hardening OS, enkripsi partisi penyimpanan, firewall internal, dan kunci di HSM/KMS.
- **API Gateway, antrean, dan cache** memakai PaaS di public segment. Opsi IaaS (menyewa VM lalu memasang gateway dan message broker sendiri) ditolak karena autoscaling saat pemilu dan PPDB harus diatur sendiri, dan patching OS di lapisan yang paling terekspos internet jadi tanggung jawab kami. Lapisan ini tidak menyimpan data mentah, jadi menyerahkan pengelolaan servernya ke penyedia tidak mengurangi kendali negara atas data.
- **Portal verifikasi dan API internal** sama-sama memakai PaaS berbasis container terkelola, walaupun portal ada di public segment dan API internal di private segment. Sempat dipertanyakan apakah PaaS cocok di private segment. Kami memutuskan boleh, asalkan platform container-nya berjalan di infrastruktur negara dan dikelola tim pemerintah. Tim pengembang cukup mengurus kode aplikasi, validasi skema permintaan, dan masking log.
- **Monitoring dan tiket insiden** adalah satu-satunya komponen yang boleh memakai SaaS, dengan syarat layanan itu tidak pernah menerima NIK, nama, atau biometrik. Yang dikirim hanya metrik performa (uptime, latensi) dan ID transaksi anonim. Jika penyedia tidak bisa menjamin syarat ini, monitoring dijalankan self-hosted.
- **Keputusan: semakin dekat ke data mentah, semakin rendah abstraksinya.** Makin tinggi abstraksi service model, makin banyak tanggung jawab yang pindah ke penyedia (Shared Responsibility Model). Karena itu IaaS dipakai di database yang menyimpan NIK dan biometrik, PaaS di lapisan stateless yang harus autoscale saat lonjakan, dan SaaS hanya untuk alat yang tidak menyentuh data pribadi.
- **Perdebatan yang muncul:** apakah layanan PaaS di public segment aman dari sisi residensi data. Managed gateway, antrean, dan cache praktis karena autoscaling dan rate limiting sudah tersedia, tetapi beberapa layanan terkelola punya fitur bawaan yang bisa membawa data ke luar Indonesia, misalnya replikasi lintas region dan log yang dikirim ke control plane global. Kami tetap memilih PaaS, lalu membatasinya dengan policy-as-code (larangan membuat sumber daya di luar region Indonesia) dan enkripsi memakai kunci dari HSM negara. Pembahasan lengkapnya menjadi Risiko 3 di Bagian 3.

## Log Penggunaan AI (Level 2)

> Wajib diisi sesuai kebijakan Level 2 di [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Tulis "Tidak memakai AI" pada baris pertama jika memang tidak dipakai. Hanya untuk brainstorming ide/outline — bukan untuk kode/analisis/teks akhir.

| Tanggal | Tool AI | Prompt yang diberikan | Ringkasan saran/ide AI | Bagaimana diolah jadi tulisan/kode sendiri |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |
