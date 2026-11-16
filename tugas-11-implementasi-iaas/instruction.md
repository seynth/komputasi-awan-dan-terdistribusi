# 📘 Instruction — Tugas 11: Implementasi IaaS (Simulasi VM Kustom)

**Kelompok:** [Isi Nama Kelompok]  
**Target Selesai:** [Isi Tanggal, misal: H-2 sebelum deadline]  
**Fokus Utama:** Mensimulasikan provisioning server IaaS menggunakan Docker, membangun script otomasi Bash yang modular dan berfungsi nyata, serta menganalisis perbedaan mendasar antara Container dan Virtual Machine (VM).
---

## 👥 Pembagian Peran & Spesifikasi Tugas

| Nama Anggota | 💻 Tugas Coding / Scripting (Wajib) | 🧪 Testing & Validasi | 📝 Analisis & Dokumentasi (README) |
| :--- | :--- | :--- | :--- |
| **Anggota 1** | `Dockerfile`: Base OS penuh, instalasi paket, setup user/direktori, `COPY`, `ENTRYPOINT`. | Memastikan image build sukses dan semua tools (`cron`, `curl`, dll) terinstal & bisa dipanggil. | Analisis karakteristik IaaS (Shared Responsibility Model). |
| **Anggota 2** | `provision.sh` (Modul Inti): Menulis logika utama skenario (misal: perintah `rsync` backup atau `curl` health check) beserta error handling (`if/else`). | Menjalankan modul inti secara manual di dalam container untuk memastikan logika bekerja sebelum dijadwalkan. | Penjelasan alur kerja skrip inti + Bukti screenshot hasil eksekusi. |
| **Anggota 3** | `provision.sh` (Modul Scheduler): Menulis konfigurasi `crontab`, setup log rotation untuk log otomasi, dan perintah agar container tidak mati (`cron -f` atau `tail`). | Memastikan cronjob benar-benar terpanggil secara otomatis sesuai jadwal (misal: setiap 1 menit) tanpa trigger manual. | Analisis mendalam: Perbedaan Docker Container vs VM Sungguhan (isolasi kernel, PID 1). |
| **Anggota 4** | `provision.sh` (Modul Reporting): Menulis fungsi yang mencatat ringkasan eksekusi (misal: "Backup Gagal: Disk Penuh" atau "Health Check: 3x gagal berturut-turut") ke file `report.txt`. | Memicu kondisi gagal (misal: matikan service target), jalankan container, dan buktikan file `report.txt` terisi dengan alert yang benar. | Estimasi Biaya VM Cloud (AWS EC2) + Lead Integrasi `README` & `JURNAL`. |

---

## 📋 Spesifikasi Detail Tugas Masing-Masing

### 👤 Anggota 1: Infrastruktur Dasar (`Dockerfile`)
- **Tugas Coding:** Buat `Dockerfile` dari base image OS penuh (contoh: `FROM ubuntu:22.04`, **DILARANG** pakai `-slim` atau `-alpine`). Lakukan `apt-get update && apt-get install -y cron curl rsync` (sesuaikan skenario). Buat direktori kerja (`WORKDIR`) dan `COPY provision.sh`.
- **Tugas Testing:** Build image (`docker build -t foodgo-custom-vm .`). Jalankan `docker run --rm foodgo-custom-vm which cron` untuk memvalidasi instalasi.
- **Tugas Dokumentasi:** Jelaskan di README mengapa ini adalah **IaaS** (jelaskan Shared Responsibility Model: user mengelola OS, runtime, aplikasi; provider hanya menyediakan virtualisasi hardware).

### 👤 Anggota 2: Logika Inti Otomasi (`provision.sh` - Modul Eksekusi)
- **Tugas Coding:** Tulis fungsi Bash utama. Pilih **satu** skenario bersama tim:
  1. *Backup:* Script `rsync` atau `cp` yang menyalin folder `/data` ke `/backup` dengan timestamp.
  2. *Health Check:* Script `curl` yang mengecek port tertentu, lalu menyimpan status (200 OK atau Error).
  - **Wajib:** Tambahkan error handling sederhana (misal: `if [ ! -d "/data" ]; then echo "Folder tidak ada" >> /var/log/automation.log; fi`).
- **Tugas Testing:** Eksekusi fungsi ini secara manual di dalam container (`bash provision.sh`). Pastikan file output/log terbentuk.
- **Tugas Dokumentasi:** Jelaskan alur kerja skrip inti di README dan sertakan screenshot bukti file hasil otomasi.

### 👤 Anggota 3: Penjadwalan & Manajemen Proses (`provision.sh` - Modul Scheduler)
- **Tugas Coding:** Lengkapi `provision.sh` agar berjalan otomatis. Ini termasuk: membuat file cron (`echo "* * * * * root /provision.sh" > /etc/cron.d/myjob`), mengatur hak akses (`chmod 0644`), dan yang paling krusial: **menjaga container tetap hidup** di `Dockerfile` (misal: `CMD service cron start && tail -f /var/log/automation.log`).
- **Tugas Testing:** Jalankan container, tunggu 2-3 menit, dan verifikasi bahwa cronjob memicu skrip secara otomatis berulang kali.
- **Tugas Dokumentasi:** Tulis di README analisis: **Perbedaan Docker Container vs VM Sungguhan**. Jelaskan konsep isolasi kernel, overhead, dan mengapa menjalankan `cron` di Docker lebih rumit daripada di VM biasa (masalah PID 1).

### 👤 Anggota 4: Modul Pelaporan, Validasi & Costing
- **Tugas Coding:** Tambahkan modul **Reporting/Alerting** di `provision.sh`. Contoh: Jika modul health check (Anggota 2) mendeteksi error, modul Anda akan menambahkan entri ke file `/var/log/alert_report.txt` dengan format: `[ALERT] $(date): Service Down`. Atau, buat skrip validasi `check_status.sh` yang membaca log dan mencetak ringkasan "SUKSES" atau "GAGAL".
- **Tugas Testing:** Sengaja buat skenario gagal (misal: hapus folder tujuan backup, atau stop service yang di-check). Jalankan container dan buktikan bahwa modul reporting Anda berhasil menangkap error tersebut di `alert_report.txt`.
- **Tugas Dokumentasi:** 
  1. Lakukan riset harga: Jika ini di-deploy di AWS EC2 (misal: `t3.micro`), berapa biaya per bulan jika melewati Free Tier? (Sertakan asumsi harga per jam x 720 jam).
  2. Menjadi *Lead Integrator*: Gabungkan semua file, pastikan `docker build` dan `docker run` berjalan mulus dari repo akhir, dan susun `JURNAL.md`.

