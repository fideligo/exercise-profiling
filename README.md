# Reflection - Milestone 1: Profiling & Performance Analysis

### 1. Observasi Proses Data Seeding
Selama pengerjaan Milestone 1, ditemukan bahwa proses seeding data ke database mengalami kendala performa yang signifikan. Terjadi duplikasi eksekusi yang mengakibatkan jumlah data membengkak menjadi **40.000 mahasiswa** dan **61.174 record** pada tabel `student_courses`.

**Analisis Masalah:**
Proses seeding sangat lambat karena penggunaan `repository.save()` di dalam iterasi loop (N+1 writes). Hal ini menciptakan bottleneck pada I/O disk karena setiap record memicu transaksi database individual.

### 2. Hasil Eksekusi JMeter
Berikut adalah hasil pengujian pada ketiga endpoint dengan beban data ~61k record:

| Nama Sampler | Endpoint | Status | Keterangan |
| --- | --- | --- | --- |
| `all-student-request` | `/all-student-data` | Success | Paling lambat karena payload JSON sangat besar. |
| `all-student-name` | `/all-student-name` | Success | Cukup lambat, memproses list String nama mahasiswa. |
| `highest-gpa` | `/highest-gpa` | Success | Cepat secara I/O, namun memicu lonjakan CPU untuk sorting. |

#### Screenshot Hasil JMeter:
![JMeter Test Results](./doc/images/test-results.png)

### 3. Analisis Profiling (Hot Spots)
Berdasarkan pengamatan menggunakan Profiling Tool (VisualVM/JProfiler):
- **CPU Spikes:** Lonjakan CPU terlihat jelas saat melakukan pencarian mahasiswa dengan IPK tertinggi di antara puluhan ribu data.
- **Memory/Heap Usage:** Penggunaan memori meningkat tajam saat aplikasi mencoba memuat seluruh daftar mahasiswa ke dalam RAM untuk dikirim sebagai response JSON.

### 4. Kesimpulan
Profiling membantu mengidentifikasi bahwa pendekatan imperatif sederhana (seperti looping manual) tidak efisien untuk dataset besar. Diperlukan optimasi seperti Batch Processing atau penggunaan query database yang lebih spesifik untuk meningkatkan skalabilitas aplikasi.

### 5. Bukti Eksekusi JMeter melalui Command Line (CLI)
Sesuai dengan instruksi modul, berikut adalah hasil pengujian performa yang dijalankan melalui terminal untuk memverifikasi statistik waktu respon secara mendetail:

#### a. Log Eksekusi: All Student Request
Menunjukkan durasi eksekusi untuk penarikan data lengkap (61k+ records).
![All Student Request Log](./doc/images/all-student-request-log.png)

#### b. Log Eksekusi: All Student Name
Menunjukkan statistik waktu respon untuk endpoint yang hanya mengembalikan daftar nama.
![All Student Name Log](./doc/images/all-student-name-log.png)

#### c. Log Eksekusi: Highest GPA
Menunjukkan hasil eksekusi terminal untuk pencarian data tunggal dengan IPK tertinggi.
![Highest GPA Log](./doc/images/highest-gpa-log.png)

---
*Dokumentasi ini disusun sebagai bagian dari pemenuhan kriteria penilaian Milestone 1 - Profiling.*