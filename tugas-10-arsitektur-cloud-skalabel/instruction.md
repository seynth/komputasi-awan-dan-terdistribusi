# 📘 Instruction — Tugas 10: Arsitektur Cloud yang Skalabel

**Kelompok:** [Isi Nama Kelompok]  
**Target Selesai:** [Isi Tanggal, misal: H-2 sebelum deadline]  
**Fokus Utama:** Membuktikan distribusi beban (Load Balancing) dan membandingkan efektivitas Horizontal vs Vertical Scaling melalui simulasi Docker Compose, Load Testing, dan analisis berbasis data eksperimen.


## 👥 Pembagian Peran & Spesifikasi Tugas

| Nama Anggota | 💻 Tugas Coding / Konfigurasi (Wajib) | 🧪 Testing & Validasi | 📝 Analisis & Dokumentasi (README) |
| :--- | :--- | :--- | :--- |
| **Anggota 1** | `app/app.py`: Implementasi pembacaan `INSTANCE_ID` dari env var, dan penambahan *artificial delay* kecil untuk mensimulasikan beban kerja. | **Eksperimen Vertical Scaling**: Jalankan 1 instance dengan limit resource besar (`deploy.resources`), lakukan load test, catat metrik (sukses/gagal/latensi). | Analisis mengapa Vertical Scaling rentan terhadap *thundering herd* dan *single point of failure* berdasarkan data tes. |
| **Anggota 2** | `nginx/nginx.conf`: Membuat blok `upstream` yang benar, mengonfigurasi `proxy_pass`, dan menambahkan parameter health check dasar (misal: `max_fails=3`). | **Eksperimen Horizontal Dasar**: Jalankan 3 instance, eksekusi load test, kumpulkan dan validasi angka distribusi request per instance. | Bukti distribusi beban (tabel angka konkret) & penjelasan algoritma Nginx (Round Robin) yang menyebabkan distribusi tersebut. |
| **Anggota 3** | `load_test.py`: Implementasi request konkuren (menggunakan `concurrent.futures` atau `asyncio`), parsing response, dan logika penghitungan frekuensi per `INSTANCE_ID`. | **Eksperimen Scale-Up**: Modifikasi compose & nginx untuk menambah `app4`, restart, jalankan load test ulang, bandingkan distribusi dengan hasil 3 instance. | Analisis perbandingan data Horizontal (3 vs 4 instance) dan bukti skalabilitas linear tanpa mengganti hardware. |
| **Anggota 4** | `docker-compose.yml`: Struktur multi-instance, definisi `deploy.resources` (untuk tes Anggota 1), dan implementasi blok `healthcheck` pada service app. | **Eksperimen Resiliensi (Wajib)**: Jalankan load test, lalu **matikan paksa 1 container `app`** di tengah proses. Catat jumlah request gagal (502) vs yang berhasil di-redirect. | Analisis keterbatasan simulasi manual vs *Auto-Scaling* cloud asli (Metric Collector, ASG, Health Check) + Lead Integrasi. |

---

## 📋 Spesifikasi Detail Tugas Masing-Masing

### 👤 Anggota 1: Aplikasi & Vertical Scaling
- **Tugas Coding:** Buka `app/app.py`. Lengkapi TODO agar aplikasi membaca environment variable `INSTANCE_ID` (misal: "app1") dan mengembalikannya dalam response. Tambahkan `time.sleep(0.05)` untuk mensimulasikan beban pemrosesan yang realistis.
- **Tugas Testing:** Ubah `docker-compose.yml` sementara untuk menjalankan hanya 1 instance dengan resource besar (misal: `cpus: '2.0', memory: 1G`). Jalankan `load_test.py` (misal: 300 request). Catat berapa request yang sukses, timeout, atau error.
- **Tugas Dokumentasi:** Tulis di README analisis mengapa Vertical Scaling (meski resource besar) tetap gagal menangani fenomena *thundering herd* (1 juta pengguna) dibandingkan Horizontal Scaling.

### 👤 Anggota 2: Load Balancer & Distribusi Beban Awal
- **Tugas Coding:** Buka `nginx/nginx.conf`. Lengkapi TODO dengan membuat blok `upstream` yang berisi `server app1:5000;`, `server app2:5000;`, `server app3:5000;`. Pastikan `proxy_pass` mengarah ke upstream ini.
- **Tugas Testing:** Jalankan `docker compose up --build` dengan 3 instance. Jalankan `load_test.py`. Ambil screenshot terminal yang menunjukkan output pembagian request (misal: app1: 100x, app2: 100x, app3: 100x).
- **Tugas Dokumentasi:** Tulis di README bukti distribusi beban dalam bentuk tabel/angka, dan jelaskan secara singkat bagaimana Nginx mendistribusikan beban tersebut.

### 👤 Anggota 3: Load Testing Script & Scale-Up
- **Tugas Coding:** Buka `load_test.py`. Lengkapi TODO untuk mengirim N request secara konkuren. Tambahkan logika untuk mengekstrak `INSTANCE_ID` dari response dan mencetak rekapitulasi: "app1: X request, app2: Y request, app3: Z request".
- **Tugas Testing:** Edit `docker-compose.yml` dan `nginx.conf` untuk menambahkan `app4`. Restart stack. Jalankan `load_test.py` lagi. Bandingkan hasilnya: apakah beban sekarang terbagi lebih rata ke 4 instance?
- **Tugas Dokumentasi:** Tulis di README perbandingan data antara 3 instance vs 4 instance. Jelaskan keunggulan Horizontal Scaling dalam menambah kapasitas secara linear.

### 👤 Anggota 4: Infrastruktur Docker, Resiliensi & Auto-Scaling
- **Tugas Coding:** Pastikan `docker-compose.yml` rapi dan produksi-ready. Tambahkan blok `healthcheck` pada service `app` (misal: `test: ["CMD", "curl", "-f", "http://localhost:5000/"]`) agar Nginx bisa mendeteksi instance yang mati.
- **Tugas Testing (Wajib Jurnal):** Jalankan load test (misal: 500 request). Di tengah proses, jalankan `docker stop [nama_container_app]`. Catat di `JURNAL.md`: berapa request yang gagal (502 Bad Gateway) sesaat, dan apakah sisa request berhasil dialihkan ke instance yang hidup berkat healthcheck?
- **Tugas Dokumentasi:** 
  1. Tulis di README analisis **keterbatasan simulasi ini**. Jelaskan bahwa di cloud asli, penambahan instance dilakukan **otomatis** oleh *Metric Collector* (misal: CloudWatch), *Auto Scaling Group*, dan *Health Check*, bukan manual edit file.
  2. Menjadi *Lead Integrator*: Gabungkan semua bagian ke `README.md`, pastikan format konsisten, dan susun `JURNAL.md`.
