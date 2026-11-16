# 📘 Instruction — Tugas 9: Cloud Delivery & Deployment Model

**Kelompok:** [Isi Nama Kelompok]  
**Target Selesai:** [Isi Tanggal, misal: H-2 sebelum deadline]  
**Fokus Utama:** Merancang arsitektur Sistem Data Kependudukan yang aman, skalabel, dan sesuai regulasi melalui analisis mendalam dan visualisasi desain.

## 👥 Pembagian Peran & Spesifikasi Tugas

| Nama Anggota | 🧠 Analisis (README) | 🎨 Artefak Visual / Teknis | 📝 Dokumentasi & Output |
| :--- | :--- | :--- | :--- |
| **Anggota 1** | **Deployment Model & Karakteristik NIST**<br>Rekomendasi model + pemetaan ke 5 karakteristik NIST. | **Mermaid Deployment Diagram**<br>(Pemisahan segmen Public vs Private). | README Bagian 1 & Bagian Jurnal Pribadi. |
| **Anggota 2** | **Service Model per Komponen**<br>Pemecahan sistem jadi 3 komponen + rekomendasi IaaS/PaaS/SaaS. | **Mermaid Sequence Diagram**<br>(Alur data dari instansi eksternal ke DB inti). | README Bagian 2 & Bagian Jurnal Pribadi. |
| **Anggota 3** | **Analisis Risiko & Mitigasi**<br>2 risiko spesifik (bukan generik) + solusi mitigasi teknis. | **Tabel Matriks Risiko**<br>(Markdown table: Risiko, Dampak, Mitigasi). | README Bagian 3 & Bagian Jurnal Pribadi. |
| **Anggota 4** | **Alternatif Ditolak & Skenario Tandingan**<br>Perbandingan dengan 1 model yang ditolak + 1 skenario tandingan wajib. | **Tabel Perbandingan** + **Mermaid Flowchart**<br>(Logika fallback skenario tandingan). | README Bagian 4 & 5, Lead Integrasi `JURNAL.md`. |

---

## 📋 Spesifikasi Detail Tugas Masing-Masing

### 👤 Anggota 1: Arsitektur Deployment & NIST
- **Tugas Analisis:** Rekomendasikan Deployment Model (Saran: *Hybrid Cloud*). Jelaskan secara spesifik bagaimana model ini memenuhi 5 karakteristik NIST (*On-demand self-service, Broad network access, Resource pooling, Rapid elasticity, Measured service*) dalam konteks data kependudukan yang rahasia.
- **Tugas Visual:** Buat **Mermaid Deployment Diagram** yang memvisualisasikan pemisahan fisik/logical antara *Private Segment* (misal: Database Biometrik di on-premise/private cloud) dan *Public Segment* (misal: API Gateway untuk akses eksternal).
- **Output:** Paragraf analisis NIST + 1 Diagram Deployment.

### 👤 Anggota 2: Service Model & Komponen Sistem
- **Tugas Analisis:** Pecah sistem menjadi minimal 3 komponen (contoh: 1. Database Inti Kependudukan, 2. Portal Verifikasi Publik, 3. API Gateway untuk Bank/Kepolisian). Tentukan Service Model (IaaS/PaaS/SaaS) untuk masing-masing dan berikan argumen kenapa pilihan itu paling efisien dan aman.
- **Tugas Visual:** Buat **Mermaid Sequence Diagram** yang menunjukkan alur permintaan dari "Bank (KYC)" → "API Gateway" → "Private DB", termasuk di titik mana autentikasi/validasi terjadi.
- **Output:** Paragraf pemetaan komponen + 1 Diagram Sequence.

### 👤 Anggota 3: Manajemen Risiko & Mitigasi
- **Tugas Analisis:** Identifikasi **minimal 2 risiko spesifik** (DILARANG risiko generik seperti "server mati" atau "diretas"). Fokus pada konteks data, misal: *Risiko inkonsistensi data saat replikasi antara segmen private dan public*, atau *Risiko kebocoran data melalui celah API pihak ketiga*. Berikan mitigasi teknis yang konkret.
- **Tugas Visual:** Buat **Tabel Matriks Risiko** yang rapi menggunakan Markdown di README (Kolom: No, Risiko Spesifik, Dampak Bisnis, Mitigasi Teknis).
- **Output:** Paragraf mitigasi + 1 Tabel Matriks Risiko.

### 👤 Anggota 4: Skenario Tandingan, Alternatif Ditolak & Integrasi
- **Tugas Analisis:** 
  1. Bandingkan dengan **1 alternatif yang ditolak** (misal: kenapa *Public Cloud Murni* ditolak karena UU PDP, atau kenapa *Private Cloud Murni* gagal saat lonjakan pemilu).
  2. **Wajib:** Buat 1 skenario tandingan (misal: *"Bagaimana jika Kantor Dukcapil di daerah terpencil tidak memiliki jaringan stabil untuk mengakses Hybrid Cloud?"*) dan berikan solusi arsitektural (misal: *Edge Caching* atau *Offline-First sync*).
- **Tugas Visual:** Buat **Tabel Perbandingan Singkat** (Model Dipilih vs Model Ditolak) DAN **Mermaid Flowchart** yang menggambarkan logika fallback saat skenario tandingan terjadi.
- **Tugas Tambahan:** Menjadi *Lead Integrator* yang menggabungkan semua bagian ke `README.md` utama, memastikan format konsisten, dan menyusun kerangka `JURNAL.md`.
- **Output:** Paragraf perbandingan & skenario tandingan + 1 Tabel + 1 Flowchart + Integrasi Final.

