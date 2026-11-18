# Tugas 9 (Pekan 9) — Cloud Delivery & Deployment Model

**Materi terkait:** Karakteristik esensial cloud computing (NIST SP 800-145), Deployment Model (Public/Private/Hybrid/Community), Service Model (IaaS/PaaS/SaaS).

> Bagian 2 mata kuliah ini murni **analisis & desain di atas kertas** — tidak ada biaya cloud sama sekali, karena tugas berhenti di tahap rekomendasi, bukan implementasi nyata (implementasi baru mulai Tugas 11).

## Studi Kasus

Pemerintah ingin membangun **Sistem Data Kependudukan** (nama, NIK, alamat, data biometrik) yang:
- Sangat **rahasia** (data pribadi warga negara, diatur UU Perlindungan Data Pribadi).
- Diakses oleh **ribuan instansi** (dukcapil daerah, kepolisian, bank untuk verifikasi KYC) dengan pola akses yang **tidak rata** (lonjakan saat musim pemilu/pendaftaran sekolah).
- Butuh **skalabilitas** agar tidak downtime saat lonjakan akses, tapi **tidak boleh** datanya berada di infrastruktur yang tidak terkendali penuh oleh negara.

## Tugas Kelompok

1. **Rekomendasikan Deployment Model** (Public / Private / Hybrid / Community Cloud) yang paling tepat untuk sistem ini. Jelaskan alasannya berdasarkan **karakteristik NIST** (on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service) dikaitkan dengan kebutuhan kerahasiaan vs skalabilitas.
2. **Rekomendasikan Service Model** (IaaS / PaaS / SaaS) yang tepat untuk komponen-komponen berbeda dari sistem ini (boleh berbeda service model untuk komponen berbeda, mis. database inti vs portal publik untuk pengecekan status).
3. Jelaskan **minimal 2 risiko spesifik** dari pilihan kalian dan mitigasinya (mis. jika Hybrid Cloud dipilih: risiko sinkronisasi data antara private & public segment).
4. Bandingkan singkat dengan **satu alternatif yang kalian tolak** — kenapa alternatif itu kurang tepat (mis. kenapa Public Cloud murni ditolak, atau kenapa Private Cloud murni dianggap kurang skalabel).

## Struktur Submission

```
tugas-09-cloud-delivery-deployment/
├── README.md      # Analisis lengkap (isi dengan poin 1-4 di atas)
├── JURNAL.md
└── bukti/          # (opsional) diagram pendukung
```

## Rubrik Penilaian (Tugas 9)

| Komponen | Bobot | Kriteria |
|---|---|---|
| Ketepatan rekomendasi deployment model | 30% | Argumen berbasis karakteristik NIST, bukan sekadar "private cloud lebih aman" tanpa detail |
| Ketepatan rekomendasi service model | 25% | Pemetaan komponen sistem ke IaaS/PaaS/SaaS masuk akal |
| Analisis risiko & mitigasi | 25% | Risiko spesifik ke konteks data kependudukan (bukan risiko generik) |
| Perbandingan dengan alternatif yang ditolak | 20% | Argumen penolakan jelas dan berdasar |

## Batasan Penggunaan AI (Level 2)

Kebijakan **Level 2 (AI Assisted Idea Generation & Structuring)** berlaku — lihat [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Boleh bertanya ke AI gambaran umum karakteristik NIST/deployment model untuk brainstorming; **tidak boleh** meminta AI menuliskan rekomendasi/analisis akhir yang tinggal ditempel ke `README.md`. Catat pemakaian AI di "Log Penggunaan AI" pada `JURNAL.md`.

- Jawaban yang terasa seperti "template" (bisa dipakai untuk sistem cloud apa saja tanpa menyebut spesifik kependudukan/data pribadi/regulasi) akan dinilai rendah pada komponen ketepatan rekomendasi.
- `README.md` wajib membahas minimal satu skenario tandingan yang kalian buat sendiri (mis. "bagaimana jika instansi daerah tidak punya infrastruktur private cloud sendiri?") beserta jawabannya.


## Bagian 1 — Rekomendasi Deployment Model & Pemetaan Karakteristik NIST

### 1.1 Rekomendasi: Hybrid Cloud dengan Pemisahan Segmen Berbasis Klasifikasi Data

Kami merekomendasikan **Hybrid Cloud** untuk Sistem Data Kependudukan. Prinsip pemisahannya bukan "sebagian di private, sebagian di public" secara acak, melainkan berdasarkan klasifikasi data dan sifat beban kerja:

| Segmen | Lokasi | Isi | Alasan |
| :--- | :--- | :--- | :--- |
| **Private Segment** | Private cloud pemerintah / pusat data nasional (dikelola penuh oleh negara, di wilayah Indonesia) | Database inti (NIK, nama, alamat), **database biometrik** (sidik jari, iris, wajah), HSM/KMS untuk kunci enkripsi, IAM & audit log | Data pribadi bersifat spesifik (biometrik) menurut UU No. 27/2022 tentang PDP. Kendali fisik, kunci enkripsi, dan akses harus sepenuhnya di tangan negara. |
| **Public Segment** | Public cloud dengan region di Indonesia | WAF, API Gateway, portal verifikasi/cek status, cache hasil verifikasi minimal (berumur singkat), antrean permintaan terenkripsi | Lapisan ini hanya melewatkan permintaan dan hanya boleh menyimpan hasil verifikasi minimal (misalnya "cocok / tidak cocok"). NIK, data biometrik, dan data pribadi lain. |

Intinya, kerahasiaan dijaga dengan menempatkan data di private segment, sedangkan skalabilitas dicapai dengan menempatkan lapisan akses di public segment. Dua kebutuhan yang tampak bertentangan dalam studi kasus ini dipisahkan ke dua segmen yang berbeda.

> **Dasar regulasi:** UU No. 27 Tahun 2022 (PDP) mengategorikan data biometrik sebagai data pribadi yang bersifat spesifik dengan perlindungan lebih ketat. PP No. 71 Tahun 2019 (PSTE) pada pokoknya mewajibkan penyelenggara sistem elektronik lingkup publik mengelola dan memproses sistem serta data elektroniknya di wilayah Indonesia. Karena itu public segment kami batasi pada region Indonesia** dan tidak menyimpan data inti.

### 1.2 Batasan Data pada Public Segment

Permintaan verifikasi (seperti KYC dari bank) memuat NIK sebagai data pribadi. Artinya, public segment tetap bersentuhan dengan data pribadi karena dilalui oleh data tersebut, sehingga batasan desain berikut diterapkan:

| # | Batasan | Penerapan teknis |
| :--- | :--- | :--- |
| 1 | NIK tidak disimpan di cache, antrean persisten, maupun penyimpanan apa pun di Public Cloud. | NIK hanya ada di memori pemrosesan selama permintaan diteruskan. Kunci cache tidak memakai NIK mentah, melainkan HMAC-SHA256 dengan kunci rahasia (bukan hash polos, karena ruang NIK kecil dan mudah ditebak dengan brute force). Kunci HMAC dikelola dari KMS/HSM di private segment. |
| 2 | Cache hanya menyimpan hasil verifikasi minimal dengan masa berlaku singkat. | Isi cache: status ("cocok / tidak cocok") dan timestamp saja. TTL pendek (misalnya hitungan menit, nilai pastinya ditentukan kebijakan instansi), tanpa nama, alamat, atau biometrik. Cache bersifat opsional: manfaat utamanya untuk portal cek status yang pola aksesnya berulang. Untuk KYC bank yang tingkat pengulangannya rendah, cache boleh dimatikan. |
| 3 | Permintaan sensitif diteruskan lewat koneksi terenkripsi. | TLS 1.3 dari instansi ke gateway, lalu mTLS lewat VPN IPsec/dedicated link ke private segment. Untuk permintaan sensitif ditambahkan enkripsi tingkat field (payload NIK dienkripsi dengan kunci publik private segment), sehingga NIK tetap terlindungi sekalipun TLS diakhiri di WAF/gateway publik. |
| 4 | Data sensitif tidak boleh masuk log secara mentah. | Log di WAF, gateway, dan portal memakai masking atau redaction (misalnya `NIK=3578********0001` atau diganti ID transaksi). Request body tidak dicatat. Log lengkap yang memuat identitas hanya ada di Audit Log private segment. |
| 5 | Antrean permintaan tidak menyimpan NIK terbuka. | Pesan di antrean berisi payload terenkripsi (kunci dari private segment) dengan TTL pendek. Antrean tidak membuat arsip permanen. Permintaan yang kedaluwarsa dibuang dan instansi diminta mengirim ulang. |
| 6 | Tidak ada data biometrik yang melintas ke public segment. | Verifikasi biometrik dilakukan sepenuhnya di private segment. Public segment hanya menerima hasilnya. |

Dengan batasan ini, risiko pada public segment turun dari "menyimpan data pribadi" menjadi "dilalui data pribadi secara sementara dan terenkripsi". Paparan data pada public segment dibuat sekecil dan sesingkat mungkin.

### 1.3 Pemetaan ke 5 Karakteristik Esensial NIST (SP 800-145)

| Karakteristik NIST | Penerapan pada Hybrid Cloud Sistem Kependudukan | Hubungan Kerahasiaan vs Skalabilitas |
| :--- | :--- | :--- |
| **On demand self service** | Tim TI Dukcapil Pusat dapat menyediakan sendiri API key, kuota, dan environment untuk instansi baru (misalnya bank yang baru bekerja sama) lewat portal tanpa tiket manual ke vendor. Pembuatan kredensial tetap melewati approval workflow dan IAM di private segment. | Kemandirian provisioning dipertahankan, tetapi hak akses tidak otomatis diberikan. Self-service berlaku untuk sumber daya, bukan untuk izin akses data. |
| **Broad network access** | Ribuan instansi (dukcapil daerah, kepolisian, bank) mengakses lewat satu API Gateway publik berbasis HTTPS/TLS 1.3 dan mTLS, dari berbagai jaringan dan perangkat. | Hanya gateway di public segment yang terekspos internet. Database inti tidak punya alamat publik dan hanya dapat dijangkau lewat jalur privat (VPN IPsec / koneksi dedicated). |
| **Resource pooling** | Public segment memakai kumpulan sumber daya multi-tenant milik penyedia cloud untuk melayani beban gabungan ribuan instansi. Private segment memakai pool internal yang eksklusif untuk negara. | Multi-tenancy hanya diterima untuk lapisan yang tidak memuat data mentah. Data NIK dan biometrik tidak pernah berbagi hardware dengan tenant lain. |
| **Rapid elasticity** | Saat musim pemilu atau pendaftaran sekolah, API Gateway dan portal verifikasi di public segment di autoscale secara horizontal, lalu diturunkan setelah lonjakan reda. Antrean dan rate limiting melindungi database inti dari banjir permintaan. | Ini inti alasan memilih Hybrid. Private cloud murni harus dibeli untuk kapasitas puncak, yang idle sebagian besar tahun. Elastisitas hanya terjadi di lapisan depan, sehingga data tidak ikut "menyebar" saat skala naik. |
| **Measured service** | Pemakaian diukur per instansi: jumlah request API, volume verifikasi, dan latensi. Di public segment dasar penagihan pay-per-use. Di private segment metering dipakai untuk kuota, chargeback* antar-instansi, dan audit akses data pribadi. | Metering sekaligus menjadi alat kepatuhan. Setiap akses ke data warga tercatat (siapa, kapan, tujuan), yang berguna untuk pembuktian akuntabilitas sesuai UU PDP dan deteksi anomali. |

### 1.4 Narasi Analisis

Sistem kependudukan memiliki dua sifat yang saling bertentangan: data sangat sensitif dan pola akses sangat fluktuatif. **Private Cloud murni** memenuhi kendali tetapi gagal pada rapid elasticity, karena kapasitas harus dibangun untuk puncak pemilu yang hanya terjadi sesekali. Public Cloud murni memenuhi elastisitas tetapi menempatkan data biometrik pada infrastruktur resource pooling milik pihak ketiga, yang bertentangan dengan tuntutan kendali penuh negara dan kewajiban pemrosesan data di wilayah Indonesia.

**Hybrid Cloud** menyelesaikan konflik ini dengan memanfaatkan karakteristik NIST secara selektif. Resource pooling dan rapid elasticity dipakai penuh di lapisan akses yang tidak menyimpan data sensitif. Sementara itu di private segment, resource pooling dibatasi pada pool eksklusif milik negara, dan measured service diarahkan ke audit akses. Broad network access disediakan hanya lewat satu pintu terkontrol (API Gateway), sedangkan on demand self service dibatasi pada penyediaan sumber daya, bukan pemberian hak atas data. Dengan demikian, setiap karakteristik NIST tidak hanya terpenuhi, tetapi juga disesuaikan dengan tingkat sensitivitas data pada segmennya.

### 1.5 Diagram Deployment
%%{init: {"flowchart": {"nodeSpacing": 40, "rankSpacing": 55}}}%%
flowchart TB
    subgraph EXT["PENGGUNA EKSTERNAL - ZONA TIDAK TEPERCAYA"]
        DKCP["Dukcapil Daerah"]
        BANK["Bank<br/>Verifikasi KYC"]
        POLRI["Kepolisian"]
        WARGA["Warga<br/>Cek Status"]
    end

    subgraph PUB["PUBLIC SEGMENT - PUBLIC CLOUD REGION INDONESIA"]
        WAF["WAF dan DDoS Protection<br/>Log dimasking, tanpa request body"]
        PORTAL["Portal Verifikasi Publik<br/>Autoscaling"]
        GW["API Gateway<br/>Autentikasi mTLS / OAuth2<br/>Rate Limiting dan Autoscaling"]
        CACHE["Cache Hasil Verifikasi Minimal<br/>Tanpa NIK dan Biometrik<br/>Kunci HMAC - TTL singkat"]
        QUEUE["Antrean Permintaan<br/>Payload terenkripsi - TTL singkat"]
    end

    subgraph BATAS["BATAS SEGMEN - KONEKSI PRIVAT TERENKRIPSI"]
        VPN["VPN IPsec / Dedicated Link<br/>mTLS - Allowlist IP"]
    end

    subgraph PRV["PRIVATE SEGMENT - INFRASTRUKTUR PEMERINTAH"]
        IAM["IAM<br/>Otorisasi per Instansi"]
        INTAPI["API Internal Verifikasi"]
        AUDIT["Audit Log<br/>Akses Data Pribadi"]
        HSM["HSM dan KMS<br/>Kunci Enkripsi"]
        subgraph DATA["DATA TERLINDUNGI"]
            DBINTI[("Database Inti<br/>NIK, Nama, Alamat")]
            DBBIO[("Database Biometrik<br/>Terenkripsi")]
        end
        BACKUP[("Backup dan DR<br/>Terisolasi")]
    end

    EXT -->|HTTPS TLS 1.3| WAF
    WAF -->|Warga| PORTAL
    WAF -->|Instansi| GW
    PORTAL -->|Memanggil API| GW
    GW <-.->|Cek cache| CACHE
    GW -->|Cache miss| QUEUE
    QUEUE --> VPN
    VPN --> IAM
    IAM --> INTAPI
    INTAPI --> DATA
    INTAPI --> AUDIT
    HSM -.->|Kunci| DATA
    DATA -.->|Replikasi berkala| BACKUP

    classDef pub fill:#e3f2fd,stroke:#1565c0,color:#0d2b52
    classDef prv fill:#fce4ec,stroke:#ad1457,color:#4a0a24
    classDef lnk fill:#fff8e1,stroke:#f9a825,color:#4a3600
    classDef ext fill:#eef7ee,stroke:#2e7d32,color:#103d14
    class DKCP,BANK,POLRI,WARGA ext
    class WAF,PORTAL,GW,CACHE,QUEUE pub
    class IAM,INTAPI,AUDIT,HSM,DBINTI,DBBIO,BACKUP prv
    class VPN lnk

**Keterangan diagram:**
- **Biru (Public Segment):** komponen yang boleh diskalakan otomatis. Tidak menyimpan data inti, NIK, maupun biometrik. NIK hanya dilalui secara sementara dan terenkripsi, dan tidak pernah masuk log mentah.
- **Merah muda (Private Segment):** data NIK dan biometrik, kunci enkripsi, dan audit log. Tidak terekspos internet.
- **Kuning (Batas Segmen):** satu-satunya jalur penghubung, terenkripsi dan dibatasi allowlist.
- Cache di public segment hanya menyimpan hasil verifikasi minimal berumur pendek, tidak berisi NIK maupun data biometrik. Kunci cache berupa HMAC, bukan NIK mentah.

### 1.6 Penjelasan Alur Diagram

1. Pengguna eksternal (bank, kepolisian, Dukcapil daerah, warga) mengirim permintaan lewat internet dengan HTTPS (TLS 1.3).
2. WAF dan DDoS protection menyaring serangan jaringan. Warga dilayani Portal Verifikasi, sedangkan instansi langsung menuju API Gateway.
3. API Gateway melakukan autentikasi (mTLS/OAuth2) dan rate limiting. Portal juga memanggil gateway ini, sehingga hanya ada satu pintu menuju private segment.
4. Gateway mengecek cache. Jika ada hasil minimal yang masih berlaku, hasil langsung dikembalikan tanpa menyentuh private segment.
5. Jika tidak ada (cache miss), permintaan masuk antrean sebagai penyangga lonjakan, lalu diteruskan dengan laju terkendali lewat VPN IPsec atau Dedicated Link terenkripsi.
6. Di private segment, IAM memeriksa otorisasi instansi, lalu API Internal memproses permintaan.
7. API Internal mengakses database inti dan biometrik. Kunci enkripsi dikelola HSM/KMS.
8. API Internal menulis audit log. Hasil minimal kembali lewat jalur yang sama dan boleh disimpan di cache dengan TTL singkat.
9. Di luar alur permintaan: database direplikasi secara berkala ke backup/DR yang terisolasi.
