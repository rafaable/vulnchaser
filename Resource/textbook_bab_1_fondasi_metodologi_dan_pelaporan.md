# TEXTBOOK CYBERSECURITY BAB 1

## PROGRAM SARJANA

# Fondasi Metodologi Penetration Testing dan Pelaporan Teknis

**Dari kewenangan, metodologi, dan bukti hingga laporan yang dapat ditindaklanjuti**

**Hatma Suryotrisongko**

*Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

## Isi Bab

Bab ini bergerak dari kewenangan menuju metodologi, bukti, penilaian, pelaporan, dan verifikasi perbaikan.

* 1.1 Hakikat Penetration Testing
* 1.2 Batas Etika, Hukum, dan Kewenangan
* 1.3 Siklus Hidup Penetration Test
* 1.4 Memilih dan Menyesuaikan Metodologi
* 1.5 Catatan Teknis sebagai Sistem Kendali Mutu
* 1.6 Arsitektur Laporan untuk Pembaca yang Berbeda
* 1.7 Menulis Ringkasan Eksekutif
* 1.8 Menilai Keparahan dan Risiko
* 1.9 Anatomi Temuan Teknis yang TUNTAS
* 1.10 Contoh Temuan Lengkap pada Laboratorium
* 1.11 Prioritas Remediasi dan Retest
* 1.12 Kendali Mutu Laporan
* 1.13 Praktik Terbimbing

> *Catatan akademik. Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium vulnerable-by-design. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.*

## Gagasan Utama

Nilai sebuah penetration test tidak berhenti ketika penguji berhasil memperoleh akses. Nilai tersebut baru terwujud ketika bukti teknis diterjemahkan menjadi keputusan risiko, tindakan perbaikan, dan verifikasi yang dapat dipertanggungjawabkan.

## Pendahuluan

Seorang mahasiswa dapat menguasai pemindaian port, menemukan kesalahan konfigurasi, atau membuktikan kelemahan kontrol akses di laboratorium. Namun, kemampuan teknis tersebut belum cukup untuk menghasilkan pekerjaan penetration testing yang profesional. Organisasi tidak membeli demonstrasi kecakapan eksploitasi; organisasi membutuhkan jawaban yang dapat dipercaya atas empat pertanyaan: apa yang diuji, apa yang terbukti, mengapa hal itu penting, dan apa yang harus dilakukan selanjutnya.

Atas dasar itu, bab ini ditempatkan pada awal buku. Sebelum mempelajari teknik serangan secara lebih mendalam, mahasiswa perlu memahami tujuan pengujian, batas kewenangan, cara menjaga catatan dan bukti, serta cara menulis laporan untuk pembaca yang berbeda. Susunan ini sejalan dengan orientasi PEN-200 yang menempatkan pelaporan sebagai kompetensi inti dan menuntut dokumentasi yang cukup rinci agar proses teknis dapat diulangi oleh pembaca yang kompeten \[1\], \[2\]. Bab ini tidak menyalin struktur atau kalimat dari materi tersebut; seluruh uraian, contoh, kerangka bantu, dan latihan disusun secara mandiri untuk konteks pendidikan sarjana di Indonesia.

Penetration testing dalam buku ini selalu berarti pengujian berizin, terkendali, dan berorientasi perbaikan. Semua contoh teknis ditujukan untuk sistem milik sendiri, lingkungan laboratorium yang sengaja dibuat rentan, atau sistem yang secara tertulis telah dimasukkan ke dalam ruang lingkup pengujian.

### Capaian Pembelajaran Bab

Setelah menyelesaikan bab ini, mahasiswa mampu:

1. membedakan vulnerability assessment, penetration testing, red teaming, dan bug bounty berdasarkan tujuan, kewenangan, cara kerja, serta keluarannya;
2. menjelaskan fungsi scope, rules of engagement, otorisasi tertulis, penghentian darurat, dan perlindungan data selama pengujian;
3. memetakan aktivitas pengujian ke fase metodologi PTES, NIST SP 800-115, dan OWASP WSTG;
4. membuat catatan teknis dan paket bukti yang akurat, aman, serta dapat direproduksi;
5. menulis ringkasan eksekutif dan temuan teknis yang sesuai dengan kebutuhan pembacanya;
6. menggunakan CVSS v4.0 secara transparan tanpa menyamakan skor keparahan teknis dengan risiko bisnis; dan
7. menilai mutu laporan penetration testing dengan daftar periksa dan rubrik yang terukur.

### Peta Konsep

| Pertanyaan Profesional | Konsep yang Menjawab | Artefak Utama | 
| ----- | ----- | ----- | 
| **Apakah kegiatan ini sah dan terkendali?** | Otorisasi, scope, rules of engagement, jalur eskalasi | Surat otorisasi, daftar aset, matriks pengujian | 
| **Bagaimana pengujian dilakukan secara sistematis?** | Metodologi dan penyesuaian berbasis risiko | Rencana uji, log aktivitas, cakupan pengujian | 
| **Apa yang benar-benar terbukti?** | Catatan, bukti, reproduktibilitas, validasi manual | Perintah, keluaran, tangkapan layar, artefak terverifikasi | 
| **Seberapa penting temuan tersebut?** | Dampak, kemungkinan, CVSS, konteks bisnis | Skor dan vektor, narasi risiko, prioritas remediasi | 
| **Apa yang harus diperbaiki?** | Analisis akar masalah dan rekomendasi | Rencana remediasi, pemilik tindakan, target waktu | 
| **Bagaimana memastikan perbaikan berhasil?** | Retest dan penutupan | Status temuan, bukti retest, risiko residual | 

## 1.1 Hakikat Penetration Testing

NIST mendefinisikan penetration testing sebagai pengujian yang memverifikasi sejauh mana sistem, perangkat, atau proses mampu menahan upaya aktif untuk mengompromikan keamanannya \[3\]. Kata *aktif* dan *memverifikasi* membedakan penetration testing dari kegiatan yang sekadar menginventarisasi kemungkinan kelemahan.

Penetration testing adalah proses pengumpulan bukti melalui simulasi serangan yang telah disetujui. Penguji tidak hanya mencari kelemahan, tetapi juga menguji hipotesis serangan: apakah prasyaratnya terpenuhi, apakah kontrol pencegahan bekerja, apakah dampak dapat dicapai, dan apakah jejak serangan terdeteksi. Eksploitasi bukan tujuan akhir; eksploitasi adalah salah satu cara untuk memperoleh bukti yang proporsional terhadap tujuan pengujian.

### 1.1.1 Perbedaan Layanan Keamanan Ofensif

| Kegiatan | Pertanyaan Utama | Kedalaman Pembuktian | Cakupan dan Waktu | Keluaran Dominan | 
| ----- | ----- | ----- | ----- | ----- | 
| **Vulnerability assessment** | Kelemahan apa yang mungkin ada? | Identifikasi dan validasi terbatas | Luas, biasanya periodik | Inventaris kerentanan dan prioritas perbaikan | 
| **Penetration testing** | Jalur serangan apa yang benar-benar dapat dibuktikan dalam batas yang disepakati? | Eksploitasi terkontrol dan bukti dampak | Terdefinisi, berbatas waktu | Laporan eksekutif, temuan teknis, bukti, rekomendasi | 
| **Red teaming** | Apakah organisasi dapat mencegah, mendeteksi, dan merespons perilaku lawan tertentu? | Rantai serangan dan penghindaran deteksi sesuai tujuan | Lebih panjang, berbasis sasaran | Narasi operasi, pencapaian objective, evaluasi kontrol manusia-proses-teknologi | 
| **Bug bounty** | Dapatkah peneliti eksternal menemukan kelemahan dalam kebijakan program? | Bergantung pada bukti tiap laporan | Berkelanjutan, berbasis kebijakan | Laporan per kerentanan dan keputusan triase | 

Perbedaan tersebut menentukan ekspektasi. Menyerahkan keluaran pemindai otomatis sebagai laporan penetration test adalah kegagalan metodologis karena pemindai dapat menghasilkan *false positive*, melewatkan kelemahan logika bisnis, atau tidak membuktikan dampak. Sebaliknya, melakukan *lateral movement* ketika kontrak hanya mengizinkan validasi konfigurasi juga merupakan kegagalan, walaupun secara teknis berhasil.

### 1.1.2 Tiga Nilai yang Harus Dihasilkan

Penetration test yang baik menghasilkan tiga jenis nilai sekaligus:

1. **nilai teknis**, berupa bukti bahwa kontrol dapat atau tidak dapat ditembus dalam kondisi tertentu;
2. **nilai risiko**, berupa penjelasan mengenai konsekuensi terhadap aset, proses bisnis, dan pihak yang berkepentingan; serta
3. **nilai perbaikan**, berupa tindakan yang spesifik, berurutan, dan dapat diuji ulang.

Apabila salah satu nilai hilang, laporan menjadi timpang. Bukti tanpa konteks menimbulkan kepanikan atau salah prioritas. Konteks tanpa bukti berubah menjadi opini. Rekomendasi tanpa akar masalah berisiko hanya menutup gejala.

## 1.2 Batas Etika, Hukum, dan Kewenangan

Perintah teknis yang sama dapat menjadi kegiatan profesional atau tindakan melawan hukum bergantung pada kewenangan, target, waktu, dan cara pelaksanaannya. Karena itu, persetujuan lisan atau anggapan bahwa suatu aset "masih milik organisasi yang sama" tidak memadai untuk memulai pengujian.

Di Indonesia, penguji perlu memahami bahwa rezim hukum terkait informasi elektronik mengalami perubahan, antara lain melalui Undang-Undang Nomor 1 Tahun 2024 sebagai perubahan kedua atas Undang-Undang Informasi dan Transaksi Elektronik \[12\]. Penanganan data yang muncul selama pengujian juga harus memperhatikan Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi \[13\]. Uraian ini adalah panduan kehati-hatian akademik, bukan nasihat hukum. Untuk engagement nyata, organisasi perlu meminta penelaahan bagian hukum atau penasihat yang berwenang terhadap kontrak, kewenangan pemilik aset, pemrosesan data, dan kewajiban sektoral.

### 1.2.1 Uji Empat Lapis Kewenangan

Sebelum menyentuh target, terapkan uji empat lapis berikut:

1. **Pihak yang memberi izin berwenang.** Pemesan pengujian harus memiliki kewenangan atas sistem, data, dan penyedia pihak ketiga yang terlibat.
2. **Aset tercantum secara eksplisit.** Domain, alamat IP, aplikasi, API, akun, lokasi fisik, dan lingkungan cloud dituliskan dengan jelas; asumsi tidak menggantikan daftar aset.
3. **Teknik dan intensitas diizinkan.** Pemindaian, eksploitasi, *password spraying*, *phishing*, *persistence*, atau pengujian ketersediaan memiliki profil risiko berbeda dan harus diputuskan secara terpisah.
4. **Waktu dan kondisi operasi sesuai.** Jendela pengujian, batas laju, jam terlarang, periode kritis bisnis, dan prosedur penghentian harus diketahui seluruh pihak terkait.

Jika salah satu lapis tidak terpenuhi, penguji harus berhenti dan meminta klarifikasi tertulis.

### 1.2.2 Dokumen Pre-engagement yang Esensial

**Scope**  
Scope mendefinisikan objek yang boleh diuji dan objek yang dikecualikan. Scope yang baik tidak berhenti pada daftar alamat IP. Ia juga menjelaskan jenis pengujian, akun yang disediakan, environment, data yang boleh disentuh, dan ketergantungan pihak ketiga.

**Rules of Engagement**  
Rules of Engagement (RoE) menerjemahkan scope menjadi aturan operasional. Unsur minimum RoE meliputi:
* tujuan dan kriteria keberhasilan pengujian;
* aset in-scope dan out-of-scope;
* teknik yang diizinkan, dibatasi, dan dilarang;
* waktu pengujian dan batas laju aktivitas;
* sumber alamat IP penguji;
* penanganan data sensitif dan bukti;
* jalur komunikasi, pelaporan temuan kritis, serta prosedur eskalasi;
* kondisi stop-work, misalnya gangguan layanan, indikasi insiden nyata, atau keterlibatan aset pihak ketiga; dan
* prosedur pembersihan artefak serta retensi data.

**Otorisasi Tertulis**  
Surat otorisasi—kadang secara informal disebut *get-out-of-jail letter*—menunjukkan bahwa aktivitas tertentu dilakukan atas penugasan pihak berwenang. Dokumen ini bukan kekebalan hukum. Ia tidak memperluas scope, tidak menghapus kewajiban terhadap penyedia cloud atau operator jaringan, dan tidak membenarkan tindakan yang dilarang kontrak.

**NDA dan Ketentuan Data**  
Non-disclosure agreement serta klausul perlindungan data menetapkan siapa yang boleh menerima hasil, media penyimpanan, enkripsi, lokasi pemrosesan, periode retensi, dan cara pemusnahan. Bukti penetration test sering memuat kredensial, token, data pribadi, konfigurasi internal, atau jalur serangan; kebocoran laporan dapat menciptakan risiko baru.

### 1.2.3 Scope Creep dan Perluasan Scope yang Sah

Scope creep terjadi ketika penguji secara sepihak memperluas aktivitas melampaui persetujuan. Contohnya adalah memindai subdomain yang ditemukan saat reconnaissance tetapi tidak tercantum dalam daftar target, atau menguji tenant cloud pihak ketiga tanpa konfirmasi.

Perluasan scope yang sah memiliki jejak keputusan: penemuan dicatat, risiko dijelaskan, pihak berwenang menyetujui perubahan, RoE diperbarui, dan tim terkait diberi tahu. Sampai perubahan berlaku, aset tersebut tetap *out-of-scope*.

> **Aturan berhenti yang sederhana:** Temukan, dokumentasikan seminimal mungkin, jangan eksploitasi, lalu eskalasikan.

## 1.3 Siklus Hidup Penetration Test

Metodologi menjaga agar penguji tidak bergantung pada intuisi semata. NIST SP 800-115 mengelompokkan assessment ke dalam perencanaan, pelaksanaan, dan pascapelaksanaan \[4\]. PTES merinci tujuh bagian, yaitu pre-engagement, intelligence gathering, threat modeling, vulnerability analysis, exploitation, post-exploitation, dan reporting \[5\]. OWASP WSTG menyediakan kerangka pengujian khusus aplikasi web dan web service \[6\].

Dalam praktik, fase tersebut bersifat iteratif. Hasil eksploitasi dapat membuka kebutuhan enumeration baru; temuan kritis dapat memicu komunikasi sebelum laporan akhir; dan retest mengulangi sebagian fase pada scope yang lebih sempit.

### 1.3.1 Fase Kerja Terpadu

| Fase Terpadu | Tujuan | Contoh Aktivitas | Bukti / Keluaran | 
| ----- | ----- | ----- | ----- | 
| **1. Pre-engagement** | Menetapkan kewenangan dan tujuan | Scope, RoE, jadwal, kontak, data handling | Rencana uji yang disetujui | 
| **2. Reconnaissance** | Memahami permukaan serangan | Pengumpulan informasi pasif dan aktif yang diizinkan | Peta aset dan asumsi awal | 
| **3. Enumeration dan threat modeling** | Mengubah data menjadi hipotesis serangan | Identifikasi layanan, trust, peran, alur data, target value | Daftar hipotesis dan prioritas uji | 
| **4. Vulnerability analysis** | Menilai kelemahan potensial | Korelasi versi, konfigurasi, logika, validasi manual | Kandidat temuan dengan confidence | 
| **5. Exploitation terkontrol** | Membuktikan exploitability dan dampak minimum | Uji hipotesis dengan tindakan paling aman | Bukti keberhasilan / kegagalan | 
| **6. Post-exploitation terukur** | Menilai konsekuensi yang diizinkan | Privilege escalation atau akses data secara minimal | Jalur serangan dan blast radius | 
| **7. Reporting dan komunikasi** | Mengubah bukti menjadi keputusan | Temuan, prioritas, rekomendasi, briefing | Laporan dan readout | 
| **8. Retest dan penutupan** | Memastikan perbaikan efektif | Verifikasi fix, kontrol kompensasi, pembersihan | Status closed/partial/open dan risiko residual | 

### 1.3.2 Prinsip Bukti Minimum yang Memadai

Penguji tidak perlu mengambil seluruh basis data untuk membuktikan bahwa akses tidak sah memungkinkan. Bukti harus cukup untuk mendukung klaim, tetapi tidak berlebihan sehingga menambah dampak. Prinsip ini menuntut pertimbangan profesional:

1. gunakan jumlah rekaman minimum;
2. samarkan data pribadi atau rahasia;
3. hindari perubahan permanen;
4. jangan mempertahankan akses setelah tujuan tercapai;
5. catat batas pembuktian dengan jujur; dan
6. minta persetujuan sebelum tindakan yang berisiko mengganggu layanan.

### 1.3.3 Komunikasi Temuan Kritis

Pelaporan bukan hanya dokumen akhir. Temuan yang menunjukkan kompromi aktif, risiko keselamatan, kebocoran kredensial utama, atau jalur serangan berdampak sangat tinggi harus mengikuti mekanisme eskalasi RoE. Pesan awal sebaiknya memuat fakta minimum: aset, waktu, apa yang terbukti, dampak potensial, tindakan pengamanan segera, dan apakah aktivitas penguji telah dihentikan.

## 1.4 Memilih dan Menyesuaikan Metodologi

Tidak ada satu metodologi yang cocok untuk semua engagement. Metodologi harus disesuaikan dengan tujuan, jenis aset, tingkat kematangan organisasi, kewajiban sektor, dan waktu yang tersedia.

| Rujukan | Kekuatan | Penggunaan yang Tepat | Batas yang Perlu Dipahami | 
| ----- | ----- | ----- | ----- | 
| **NIST SP 800-115** | Tata kelola assessment, perencanaan, teknik, analisis, mitigasi | Pemerintah, organisasi formal, program assessment | Dokumen 2008; teknik spesifik perlu diperkaya sumber mutakhir | 
| **PTES** | Alur penetration testing end-to-end dan pemisahan laporan eksekutif-teknis | Penugasan jaringan / infrastruktur umum | Bukan kontrak; harus *tailor* ke risiko dan lingkungan | 
| **OWASP WSTG** | Katalog dan kerangka pengujian aplikasi web / web service | Web, API, dan alur aplikasi | Tidak menggantikan pengujian infrastruktur atau tata kelola engagement | 
| **PEN-200** | Pembelajaran hands-on, disiplin dokumentasi, dan reproduksibilitas | Pengembangan kompetensi penguji pemula-menengah | Materi kursus bukan standar kontraktual organisasi | 

Cara menyesuaikan metodologi dapat diringkas menjadi lima keputusan:

1. tentukan outcome bisnis atau keamanan yang ingin dinilai;
2. petakan aset dan jalur serangan yang relevan;
3. pilih teknik yang menghasilkan bukti dengan risiko operasional terendah;
4. definisikan bukti keberhasilan dan batas pembuktian; dan
5. rencanakan cara setiap bukti akan masuk ke laporan.

Metodologi yang baik dapat ditelusuri dari tujuan hingga bukti. Jika suatu aktivitas tidak mendukung tujuan, tidak menghasilkan bukti yang relevan, atau tidak diizinkan, aktivitas tersebut tidak perlu dilakukan.

## 1.5 Catatan Teknis sebagai Sistem Kendali Mutu

Catatan bukan sekadar pengingat pribadi. Catatan adalah sumber rekonstruksi, pengendalian scope, penghubung antara bukti dan klaim, serta bahan audit apabila terjadi gangguan atau sengketa.

OffSec menekankan pencatatan selama pekerjaan masih segar dan dokumentasi langkah, perintah, keluaran, serta tangkapan layar yang memungkinkan pembaca teknis mengulangi serangan \[2\]. Prinsip yang sama berlaku dalam pekerjaan profesional, tetapi harus ditambah dengan pengamanan data, pemisahan fakta dari hipotesis, dan jejak waktu.

### 1.5.1 Kerangka JEJAK untuk Catatan dan Bukti

Gunakan lima kriteria JEJAK berikut:

1. **Jelas** – setiap entri menyebut tujuan, target, dan hasil; tangkapan layar tidak berdiri tanpa keterangan.
2. **Eksak** – perintah, parameter, versi alat, akun/peran, serta prasyarat dicatat sebagaimana digunakan.
3. **Jejak waktu** – waktu dan zona waktu konsisten; urutan aktivitas dapat direkonstruksi.
4. **Aman** – rahasia disamarkan, berkas dienkripsi, akses dibatasi, dan retensi mengikuti RoE.
5. **Konsisten** – identitas aset, ID temuan, format nama berkas, dan istilah digunakan secara seragam.

### 1.5.2 Struktur Entri Harian

Satu entri yang baik dapat menggunakan format berikut:

* **Waktu:** `[2026-08-31 09:42:18 WIB]`
* **Tujuan:** Memvalidasi kontrol akses profil pada endpoint aktif
* **Prasyarat:** Akun mahasiswa biasa (`student-a`), sesi aktif `10.10.10.25`
* **Permintaan:** `GET /api/profiles/1003` (mengganti ID milik sendiri `1002` menjadi `1003`)
* **Hasil:** Server mengembalikan HTTP 200 dan profil pengguna lain
* **Bukti:** `F-001_req.txt`, `F-001_resp_redacted.txt`, `F-001_01.png`
* **Hipotesis / Catatan:** Otorisasi hanya dilakukan pada antarmuka, bukan pada backend; tidak ada rate limit yang relevan
* **Tindak lanjut:** Uji dua peran lain dan konfirmasi

Pisahkan observasi dari interpretasi. "Server mengembalikan HTTP 200 dan data pengguna lain" adalah observasi. "Kontrol akses tidak ada pada backend" masih merupakan interpretasi sampai dikonfirmasi melalui pengujian yang memadai atau penelaahan kode.

### 1.5.3 Bukti yang Layak Masuk Laporan

Bukti sebaiknya menjawab tiga hal: konteks awal, tindakan penguji, dan kondisi hasil. Untuk temuan akses tidak sah, misalnya, bukti perlu menunjukkan identitas atau peran sesi, permintaan yang relevan, serta respons yang membuktikan objek lain dapat diakses.

Hindari dua ekstrem. Tangkapan layar yang dipotong terlalu sempit kehilangan konteks; tangkapan layar seluruh desktop dapat membocorkan data yang tidak relevan. Berikan nomor gambar, waktu, aset, dan keterangan singkat. Samarkan token, kata sandi, data pribadi, dan informasi di luar kebutuhan pembuktian.

### 1.5.4 Integritas dan Pengelolaan Berkas

Untuk engagement yang sensitif, simpan hash berkas bukti, gunakan penyimpanan terenkripsi, dan batasi akses berdasarkan kebutuhan. Contoh konvensi nama:

```
ENG-2026-014/
├── 01-admin/
├── 02-notes/
├── 03-evidence/F-001/
├── 04-tools-output/
├── 05-report/
└── 06-retest/
```

Jangan menyimpan data klien ke layanan sinkronisasi pribadi tanpa persetujuan. Alat pencatat *local-first* dapat membantu, tetapi status *local-first* tidak otomatis memenuhi kebijakan organisasi. Konfigurasi, backup, plugin, dan lokasi penyimpanan tetap perlu ditelaah.

## 1.6 Arsitektur Laporan untuk Pembaca yang Berbeda

Laporan penetration testing adalah produk komunikasi multi-audiens. Manajemen memerlukan gambaran risiko dan keputusan. Pemilik sistem memerlukan prioritas dan pemilik tindakan. Engineer memerlukan langkah reproduksi serta rekomendasi teknis. Auditor atau tim risiko memerlukan scope, metodologi, keterbatasan, dan jejak perubahan.

OWASP WSTG menyarankan struktur yang memberi tim teknis informasi untuk memahami, mereplikasi, dan memperbaiki temuan; informasi sensitif harus disamarkan \[6\]. PTES juga membedakan pelaporan eksekutif dan teknis \[5\].

### 1.6.1 Susunan Laporan yang Disarankan

1. **Halaman judul dan klasifikasi.** Nama engagement, organisasi, versi, tanggal, penulis/reviewer, dan klasifikasi kerahasiaan.
2. **Kontrol dokumen.** Riwayat versi, daftar penerima, dan batas distribusi.
3. **Ringkasan eksekutif.** Postur risiko, pola utama, konsekuensi bisnis, dan prioritas tindakan.
4. **Scope dan keterbatasan.** Aset, periode, akun, pengecualian, perubahan scope, serta hal yang tidak diuji.
5. **Metodologi.** Pendekatan, tingkat pengetahuan awal, teknik, validasi, dan kriteria penilaian.
6. **Ringkasan temuan.** Daftar ID, judul, aset, tingkat keparahan, dan status.
7. **Temuan teknis rinci.** Akar masalah, bukti, reproduksi, dampak, skor, dan remediasi.
8. **Rantai serangan.** Jika beberapa kelemahan membentuk satu konsekuensi yang lebih besar, hubungan antartemuan dijelaskan.
9. **Rencana prioritas.** Pengelompokan tindakan segera, jangka pendek, dan perbaikan struktural.
10. **Lampiran.** Definisi tingkat risiko, daftar alat, port/service, bukti tambahan, serta keterbatasan teknis.

### 1.6.2 Prinsip "Satu Fakta, Beberapa Kedalaman"

Fakta inti tidak boleh berubah antarbagian. Yang berubah adalah kedalaman dan bahasa.

| Lapisan | Contoh Penyampaian Fakta yang Sama | 
| ----- | ----- | 
| **Eksekutif** | Pengguna biasa dapat mengakses data profil pengguna lain, sehingga kerahasiaan data pelanggan tidak terjaga. | 
| **Manajerial** | Kontrol otorisasi objek tidak diterapkan secara konsisten pada endpoint API; prioritas perbaikan adalah seluruh endpoint yang menggunakan pengenal objek. | 
| **Teknis** | Permintaan `GET /api/profiles/{id}` mengembalikan HTTP 200 untuk ID yang bukan milik sesi aktif; server tidak memverifikasi kepemilikan objek. | 

Konsistensi ini mencegah situasi ketika ringkasan eksekutif menyatakan "kompromi penuh", sedangkan bukti hanya menunjukkan akses ke satu objek uji.

## 1.7 Menulis Ringkasan Eksekutif

Ringkasan eksekutif bukan kumpulan nama kerentanan dan bukan salinan tabel temuan. Ia harus membantu pengambil keputusan memahami kondisi, penyebab sistemik, potensi konsekuensi, dan urutan tindakan.

### 1.7.1 Empat Pertanyaan yang Harus Terjawab

1. Apa yang diuji dan dengan tingkat keyakinan apa?
2. Apa pola risiko paling penting yang terbukti?
3. Apa konsekuensi terhadap layanan, data, kepatuhan, atau reputasi?
4. Apa tiga tindakan prioritas dan mengapa urutannya demikian?

> **Contoh Lemah:**  
> Pengujian menemukan satu critical, dua high, dan tiga medium. Organisasi disarankan menerapkan best practice keamanan.  
> *(Kalimat tersebut tidak menjelaskan aset, konsekuensi, akar masalah, ataupun keputusan yang perlu diambil.)*

> **Contoh yang Diperbaiki:**  
> Pengujian terhadap portal layanan mahasiswa dan API pendukung membuktikan bahwa akun berhak akses rendah dapat membaca data profil milik pengguna lain. Kelemahan ini berasal dari pemeriksaan otorisasi yang tidak konsisten pada sisi server dan ditemukan pada dua dari enam endpoint objek yang diuji. Belum ditemukan bukti perubahan data atau pengambilalihan akun dalam scope pengujian. Organisasi perlu segera membatasi endpoint terdampak, kemudian menerapkan kontrol otorisasi terpusat dan pengujian regresi untuk seluruh endpoint serupa.

Versi kedua menyatakan apa yang terbukti dan tidak terbukti, sehingga pembaca tidak diarahkan pada kepastian palsu.

### 1.7.2 Bahasa yang Konstruktif

Gunakan bahasa berbasis bukti. Hindari mempermalukan tim, menebak motivasi, atau menggunakan kata absolut tanpa dukungan. "Kontrol tidak ditemukan pada dua endpoint yang diuji" lebih bertanggung jawab daripada "developer sama sekali tidak memahami keamanan". Laporan yang konstruktif meningkatkan peluang remediasi karena pembaca dapat berfokus pada sistem, bukan pada pembelaan diri.

## 1.8 Menilai Keparahan dan Risiko

Severity dan risk berhubungan, tetapi tidak identik. Severity menggambarkan karakteristik teknis kelemahan. Risk mempertimbangkan kemungkinan, dampak terhadap konteks organisasi, nilai aset, ancaman, kontrol yang ada, dan toleransi risiko.

CVSS adalah kerangka terbuka untuk mengomunikasikan karakteristik dan keparahan kerentanan. CVSS v4.0 terdiri atas kelompok Base, Threat, Environmental, dan Supplemental \[7\]. FIRST meminta publikasi skor disertai vector string agar pembaca dapat memahami dasar perhitungannya \[7\].

### 1.8.1 Kelompok Metrik CVSS v4.0

| Kelompok | Pertanyaan | Contoh Konteks | 
| ----- | ----- | ----- | 
| **Base** | Bagaimana sifat intrinsik kerentanan? | Jalur serangan, kompleksitas, kebutuhan hak, interaksi pengguna, dampak teknis | 
| **Threat** | Bagaimana keadaan ancaman saat ini? | Ketersediaan exploit atau bukti eksploitasi aktif | 
| **Environmental** | Bagaimana kerentanan berlaku pada lingkungan organisasi? | Kritikalitas aset, kontrol kompensasi, dampak lokal | 
| **Supplemental** | Informasi tambahan apa yang membantu keputusan? | Otomatisasi, urgensi penyedia, keselamatan, pemulihan | 

Panduan implementasi konsumen CVSS menekankan bahwa Base score adalah baseline umum; Threat dan Environmental metrics membantu menghasilkan penilaian yang lebih sesuai lingkungan \[8\]. Dengan demikian, skor dasar tidak boleh menjadi satu-satunya dasar SLA remediasi.

### 1.8.2 Praktik Pelaporan Skor

Untuk setiap temuan, cantumkan:

1. versi CVSS;
2. skor dan label kualitatif;
3. vector string lengkap;
4. asumsi penting yang memengaruhi metrik;
5. konteks risiko bisnis yang terpisah; dan
6. penjelasan apabila organisasi menggunakan matriks internal tambahan.

Contoh format:

* **CVSS v4.0:** 8.7 (High)
* **Base Vector:** `CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N`
* **Asumsi:** Eksploitasi memerlukan akun valid berhak akses rendah
* **Risiko Bisnis:** Kebocoran data profil lintas pengguna; dampak dipengaruhi jumlah objek dan jenis data

Vector contoh harus dihitung ulang dengan kalkulator resmi untuk kasus nyata \[9\]. Angka tidak menggantikan argumentasi. Dua kerentanan dengan skor sama dapat memiliki prioritas berbeda karena satu berada pada layanan publik yang kritis dan yang lain berada pada sistem laboratorium terisolasi.

### 1.8.3 Ketidakpastian dan Confidence

Tambahkan confidence apabila bukti tidak lengkap:

* **Tinggi** – temuan direproduksi secara konsisten dan dampak minimum telah dibuktikan;
* **Sedang** – kelemahan terkonfirmasi, tetapi sebagian dampak disimpulkan karena batas scope;
* **Rendah** – indikator ada, namun reproduksi terhambat kondisi teknis atau waktu.

Confidence tidak sama dengan severity. Temuan berpotensi kritis dengan confidence rendah harus dijelaskan sebagai hipotesis yang memerlukan validasi, bukan dipaksakan menjadi kepastian.

## 1.9 Anatomi Temuan Teknis yang TUNTAS

Kerangka TUNTAS membantu mahasiswa mengingat enam unsur temuan yang dapat ditindaklanjuti.

1. **T – Tanda pengenal dan judul.** Gunakan ID stabil dan judul yang menyebut kelemahan serta konsekuensinya.
2. **U – Unit atau aset terdampak.** Cantumkan host, endpoint, komponen, peran, environment, dan versi yang relevan.
3. **N – Narasi akar masalah dan prasyarat.** Jelaskan mengapa kontrol gagal dan kondisi yang diperlukan penyerang.
4. **T – Tahapan reproduksi dan bukti.** Susun langkah minimum yang aman, bernomor, serta dapat diulang.
5. **A – Akibat dan penilaian.** Bedakan dampak teknis, konsekuensi bisnis, severity, risk, dan confidence.
6. **S – Solusi dan strategi verifikasi.** Berikan perbaikan akar masalah, mitigasi sementara, serta kriteria retest.

### 1.9.1 Judul Berorientasi Sebab dan Dampak

Judul "IDOR" mungkin dipahami engineer keamanan, tetapi kurang informatif bagi pembaca lain. Judul "Kontrol otorisasi objek yang tidak memadai memungkinkan akses profil lintas pengguna" menyatakan akar masalah dan dampak secara langsung.

### 1.9.2 Akar Masalah, Bukan Hanya Gejala

Perintah atau payload menjelaskan cara memicu gejala. Akar masalah menjelaskan kontrol yang hilang atau gagal. Pada kasus akses objek, mengubah angka ID hanyalah teknik pemicu; akar masalahnya adalah keputusan otorisasi tidak dilakukan pada server untuk setiap objek.

### 1.9.3 Langkah Reproduksi yang Dapat Dipercaya

Langkah reproduksi harus:
1. menyebut environment dan prasyarat;
2. menggunakan identitas uji atau data yang telah disamarkan;
3. menjelaskan tindakan, bukan hanya menampilkan screenshot;
4. menyertakan hasil yang diharapkan dan hasil aktual;
5. menghindari langkah destruktif yang tidak diperlukan; dan
6. merujuk bukti dengan ID yang konsisten.

### 1.9.4 Rekomendasi yang Bisa Dikerjakan

Hindari frasa "tingkatkan keamanan", "gunakan best practice", atau "validasi input dengan benar" tanpa lokasi dan mekanisme. Rekomendasi yang baik menyebut kontrol, titik penerapan, cakupan, urutan, dan cara verifikasi.

Contoh:
> Terapkan pemeriksaan otorisasi berbasis subjek-aksi-objek pada middleware server untuk setiap endpoint profil. Jangan mengandalkan penyembunyian tombol pada antarmuka. Tambahkan pengujian negatif yang memastikan akun A menerima HTTP 403 ketika meminta objek milik akun B. Tinjau seluruh endpoint yang menggunakan pengenal objek langsung karena akar masalah berpotensi sistemik.

## 1.10 Contoh Temuan Lengkap pada Laboratorium

Contoh berikut menggunakan aplikasi fiktif *Portal Akademik Latih* pada jaringan laboratorium terisolasi. Nilai dan nama pengguna telah disamarkan. Contoh ini menunjukkan bentuk penulisan, bukan target untuk diserang.

---

### F-001: Kontrol Otorisasi Objek Tidak Memadai Memungkinkan Akses Profil Lintas Pengguna

* **Tingkat keparahan:** High
* **Confidence:** Tinggi
* **Aset:** `https://lab-app.local/api/profiles/{studentId}`
* **Peran penguji:** Mahasiswa biasa
* **Akar masalah:** Server menerima pengenal objek dari klien tanpa memverifikasi bahwa objek tersebut boleh dibaca oleh subjek yang sedang login.

#### Deskripsi
Endpoint profil mengembalikan data berdasarkan `studentId`. Penguji yang login sebagai pengguna A dapat mengganti pengenal objek menjadi milik pengguna B. Server mengembalikan HTTP 200 beserta profil pengguna B. Antarmuka aplikasi memang tidak menampilkan tautan ke profil pengguna lain, tetapi pembatasan pada antarmuka bukan kontrol otorisasi.

#### Prasyarat
1. Penyerang memiliki akun pengguna biasa yang valid.
2. Penyerang mengetahui atau dapat menebak pengenal objek pengguna lain.

#### Langkah Reproduksi
1. Login dengan akun laboratorium `student-a`.
2. Buka profil sendiri dan tangkap permintaan `GET /api/profiles/1002`.
3. Ubah pengenal menjadi `1003` tanpa mengubah cookie sesi.
4. Kirim ulang permintaan.
5. Amati bahwa server merespons HTTP 200 dan mengembalikan profil milik pengguna lain.

#### Bukti Ringkas
```http
GET /api/profiles/1003 HTTP/1.1
Host: lab-app.local
Cookie: session=[REDACTED]

HTTP/1.1 200 OK
Content-Type: application/json

{"studentId":"1003","name":"[REDACTED]","email":"[REDACTED]"}
```

#### Dampak
Penyerang dengan akun biasa dapat membaca data profil pengguna lain. Dalam environment produksi, dampak bergantung pada jenis data yang tersedia, jumlah objek yang dapat dienumerasi, kemampuan deteksi, dan kewajiban perlindungan data. Dalam pengujian ini tidak dilakukan pengambilan massal dan tidak ditemukan bukti perubahan data.

#### Penilaian
CVSS v4.0 contoh: **8.7 (High)**, vector `CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N`. Asumsi utama adalah kebutuhan akun berhak akses rendah dan dampak pada kerahasiaan sistem rentan. Pengguna harus menghitung ulang vector berdasarkan kondisi engagement nyata dengan kalkulator resmi \[9\].

#### Rekomendasi
* **Tindakan segera:** Batasi endpoint terdampak atau tambahkan pemeriksaan kepemilikan objek pada server. Pantau akses berurutan terhadap banyak `studentId`.
* **Perbaikan akar masalah:** Terapkan fungsi otorisasi terpusat yang menerima subjek, aksi, dan objek. Gunakan *deny-by-default* dan panggil pemeriksaan tersebut pada setiap endpoint objek.
* **Pencegahan berulang:** Tambahkan pengujian unit/integrasi negatif untuk akses lintas pengguna, lakukan code review pada pola endpoint serupa, dan masukkan pengujian otorisasi ke pipeline pengembangan.

#### Kriteria Retest
1. Permintaan akun A terhadap objek milik akun B menghasilkan HTTP 403 atau respons aman yang setara.
2. Endpoint lain dengan pola pengenal objek telah diuji.
3. Log mencatat penolakan tanpa merekam token atau data pribadi secara berlebihan.
4. Tidak ada jalur alternatif—misalnya endpoint ekspor—yang melewati kontrol baru.

---

## 1.11 Prioritas Remediasi dan Retest

Rekomendasi perlu disusun dalam horizon waktu agar organisasi dapat mengurangi risiko sebelum perbaikan struktural selesai.

| Horizon | Tujuan | Contoh Tindakan | 
| ----- | ----- | ----- | 
| **Segera** | Mengurangi eksposur aktif | Menonaktifkan fitur, membatasi akses, rotasi rahasia, aturan WAF sementara | 
| **Jangka pendek** | Memperbaiki kelemahan spesifik | Patch, perubahan konfigurasi, pemeriksaan otorisasi, hardening | 
| **Struktural** | Memperbaiki kelas kelemahan berulang | Library kontrol terpusat, secure coding standard, pengujian regresi, review arsitektur | 

Retest bukan sekadar mencoba payload yang sama. Penguji harus menilai apakah akar masalah telah diperbaiki, apakah kontrol baru berlaku pada seluruh variasi relevan, apakah muncul jalur bypass, dan apakah risiko residual telah dinyatakan. Status yang berguna meliputi:

* `Closed` – akar masalah diperbaiki dan kriteria verifikasi terpenuhi;
* `Partially remediated` – sebagian jalur diperbaiki, tetapi cakupan atau kontrol belum lengkap;
* `Mitigated` – risiko dikurangi dengan kontrol kompensasi, sementara akar masalah masih ada;
* `Open` – temuan masih dapat direproduksi; dan
* `Not retested` – verifikasi tidak dilakukan atau terhalang keterbatasan yang dicatat.

## 1.12 Kendali Mutu Laporan

Laporan profesional sebaiknya melewati *technical review* dan *editorial review*. Reviewer teknis memeriksa bukti, reproduksi, akar masalah, skor, serta keamanan rekomendasi. Reviewer editorial memeriksa konsistensi istilah, keterbacaan, nada, format, dan kebocoran informasi sensitif.

### 1.12.1 Daftar Periksa Sebelum Laporan Dikirim

**Kewenangan dan Scope**
1. Semua aset dan aktivitas sesuai scope serta RoE.
2. Perubahan scope memiliki persetujuan tertulis.
3. Temuan pada aset pihak ketiga ditangani sesuai jalur eskalasi.

**Bukti**
4. Setiap klaim penting memiliki bukti yang relevan.
5. Langkah reproduksi diuji ulang dari catatan bersih.
6. Token, kata sandi, data pribadi, dan informasi tidak relevan telah disamarkan.
7. Waktu, zona waktu, dan identitas aset konsisten.

**Penilaian**
8. CVSS mencantumkan versi, skor, vector, dan asumsi.
9. Severity tidak disebut sebagai risiko bisnis tanpa konteks.
10. Confidence dan keterbatasan dinyatakan dengan jujur.

**Rekomendasi**
11. Solusi menargetkan akar masalah, bukan hanya payload contoh.
12. Ada tindakan segera, perbaikan permanen, dan kriteria retest bila relevan.
13. Rekomendasi tidak menciptakan gangguan atau kelemahan baru.

**Komunikasi**
14. Ringkasan eksekutif dapat dipahami tanpa membaca detail teknis.
15. Judul temuan menyatakan kelemahan dan dampak.
16. Fakta konsisten di ringkasan, tabel, detail, dan lampiran.
17. Laporan dienkripsi dan hanya dikirim kepada penerima yang berwenang.

### 1.12.2 Kesalahan yang Sering Terjadi

1. **Menempel keluaran scanner tanpa validasi.** Perbaikan: validasi manual, nyatakan confidence, dan bedakan indikator dari temuan.
2. **Screenshot tanpa konteks.** Perbaikan: tambahkan identitas aset, tindakan, hasil, dan keterangan.
3. **Melebih-lebihkan dampak.** Perbaikan: pisahkan apa yang terbukti, apa yang disimpulkan, dan apa yang tidak diuji.
4. **Skor tanpa vector.** Perbaikan: gunakan versi dan vector lengkap serta catat asumsi.
5. **Rekomendasi generik.** Perbaikan: sebutkan kontrol, lokasi penerapan, cakupan, prioritas, dan cara verifikasi.
6. **Mengabaikan keterbatasan.** Perbaikan: dokumentasikan waktu, akun, environment, gangguan, dan area yang tidak tercakup.
7. **Membocorkan rahasia di laporan.** Perbaikan: redaksi, enkripsi, pembatasan distribusi, dan retensi minimum.

## 1.13 Praktik Terbimbing: Dari Bukti Menjadi Temuan

Gunakan OWASP Juice Shop, WebGoat, atau aplikasi vulnerable-by-design lain pada laboratorium lokal yang terisolasi. Jangan menghubungkan laboratorium rentan langsung ke internet dan jangan menguji target publik.

### Skenario
Pilih satu kelemahan yang dapat dibuktikan tanpa tindakan destruktif. Tujuan latihan bukan mendapatkan sebanyak mungkin temuan, melainkan menghasilkan satu temuan yang memenuhi kerangka TUNTAS.

### Langkah Kerja
1. Tuliskan scope mini: aplikasi, alamat, akun, waktu, teknik yang diizinkan, dan stop condition.
2. Buat log aktivitas menggunakan format JEJAK.
3. Reproduksi kelemahan dua kali dari kondisi awal yang jelas.
4. Simpan bukti minimum dan samarkan rahasia.
5. Identifikasi akar masalah dan bedakan dari teknik pemicu.
6. Hitung CVSS v4.0 serta tulis asumsi metrik.
7. Tulis temuan TUNTAS maksimal empat halaman.
8. Tukarkan laporan dengan rekan dan lakukan peer review tanpa membagikan kredensial.
9. Revisi laporan berdasarkan komentar yang dapat diverifikasi.

### Rubrik Penilaian Praktik

| Dimensi | Bobot | Indikator Kinerja Unggul | 
| ----- | ----- | ----- | 
| **Kewenangan dan scope** | 15% | Batas target, teknik, waktu, dan stop condition eksplisit | 
| **Catatan dan bukti** | 20% | JEJAK lengkap; bukti cukup, konsisten, aman, dan dapat ditelusuri | 
| **Reproduksibilitas** | 20% | Pembaca dapat mengulangi langkah tanpa menebak prasyarat | 
| **Analisis akar masalah dan dampak** | 20% | Sebab, gejala, dampak terbukti, dan batas kesimpulan dibedakan | 
| **Penilaian severity / risk** | 10% | CVSS v4.0 transparan; konteks bisnis dan asumsi dijelaskan | 
| **Remediasi dan retest** | 15% | Solusi spesifik, berlapis, dan memiliki kriteria verifikasi | 

## Rangkuman Bab

1. Penetration testing adalah pengujian aktif yang berizin untuk menghasilkan bukti mengenai ketahanan kontrol, bukan perlombaan memperoleh akses.
2. Scope, RoE, otorisasi tertulis, jalur eskalasi, dan stop condition adalah kontrol keselamatan serta kewenangan.
3. Metodologi menghubungkan tujuan, aktivitas, bukti, dan laporan; pelaksanaannya dapat iteratif.
4. Catatan yang memenuhi JEJAK membuat pekerjaan jelas, eksak, bertanda waktu, aman, dan konsisten.
5. Laporan harus melayani pembaca eksekutif dan teknis tanpa mengubah fakta inti.
6. CVSS v4.0 mengomunikasikan severity; prioritas organisasi tetap memerlukan konteks ancaman, lingkungan, nilai aset, dan risiko bisnis.
7. Temuan TUNTAS memuat identitas, aset, akar masalah, reproduksi, akibat, solusi, dan retest.
8. Rekomendasi terbaik memperbaiki akar masalah dan mencegah kelas kelemahan yang sama muncul kembali.

## Uji Pemahaman

### Pilihan Ganda

1. Unsur yang paling mendasar agar aktivitas penetration testing sah adalah:
   * A. Penggunaan Kali Linux
   * **B. Izin eksplisit dari pihak yang berwenang**
   * C. Penggunaan VPN
   * D. Laporan dengan CVSS

2. Perbedaan utama vulnerability assessment dan penetration testing adalah:
   * A. Ukuran jaringan
   * B. Jenis sistem operasi
   * **C. Tingkat pembuktian exploitability dan dampak**
   * D. Merek alat pemindai

3. Ketika menemukan subdomain menarik yang tidak tercantum dalam scope, tindakan pertama adalah:
   * A. Mengeksploitasi secara hati-hati
   * B. Memindai seluruh port
   * **C. Mendokumentasikan secara minimal dan meminta perluasan tertulis**
   * D. Mengabaikan seluruh engagement

4. Pernyataan yang merupakan observasi adalah:
   * A. Developer lalai
   * **B. Server mengembalikan HTTP 200 untuk objek pengguna lain**
   * C. Seluruh database pasti bocor
   * D. Otorisasi sama sekali tidak pernah dirancang

5. Fungsi utama vector string CVSS adalah:
   * A. Mengenkripsi laporan
   * **B. Menunjukkan dasar metrik skor**
   * C. Mengganti ringkasan eksekutif
   * D. Menentukan biaya pentest

6. CVSS Base score paling tepat dipahami sebagai:
   * A. Keputusan risiko final organisasi
   * **B. Baseline keparahan teknis**
   * C. Bukti eksploitasi aktif
   * D. SLA remediasi universal

7. Bukti minimum yang memadai berarti:
   * A. Mengambil semua data agar dampak jelas
   * B. Tidak menyimpan bukti
   * **C. Mengumpulkan bukti secukupnya untuk klaim tanpa menambah dampak**
   * D. Hanya menggunakan screenshot

8. Bagian laporan yang terutama ditujukan kepada manajemen adalah:
   * A. Raw scanner output
   * **B. Ringkasan eksekutif**
   * C. Daftar perintah
   * D. Hash artefak

9. Rekomendasi "terapkan best practice keamanan" lemah karena:
   * A. Terlalu teknis
   * **B. Tidak menyebut kontrol, lokasi, dan cara verifikasi**
   * C. Terlalu panjang
   * D. Selalu salah

10. Status retest "mitigated" menunjukkan:
    * A. Akar masalah pasti hilang
    * B. Pengujian belum pernah dilakukan
    * **C. Risiko dikurangi oleh kontrol kompensasi walau akar masalah mungkin masih ada**
    * D. Temuan adalah false positive

11. Confidence rendah pada temuan berpotensi kritis seharusnya:
    * A. Disembunyikan
    * B. Dinaikkan menjadi pasti
    * **C. Dinyatakan sebagai keterbatasan dan divalidasi lebih lanjut**
    * D. Diberi skor nol

12. Technical review terutama memeriksa:
    * A. Warna sampul
    * **B. Bukti, reproduksi, akar masalah, skor, dan keamanan rekomendasi**
    * C. Harga proyek
    * D. Biodata pembaca

### Kunci Jawaban Pilihan Ganda
1. B | 2. C | 3. C | 4. B | 5. B | 6. B | 7. C | 8. B | 9. B | 10. C | 11. C | 12. B

## Pertanyaan Esai

1. Jelaskan perbedaan antara scope creep dan perluasan scope yang sah dengan satu contoh untuk lingkungan cloud.
2. Mengapa severity teknis tidak boleh langsung disamakan dengan risiko bisnis? Gunakan contoh sistem akademik dan sistem laboratorium.
3. Tulis ulang rekomendasi "perbaiki validasi input" menjadi rekomendasi yang spesifik, berlapis, dan dapat diuji ulang.
4. Buat format entri JEJAK untuk satu aktivitas enumeration pada laboratorium lokal.
5. Jelaskan bagaimana satu fakta teknis harus disampaikan secara berbeda kepada direksi dan engineer tanpa mengubah maknanya.

## Glosarium

* **Akar masalah (root cause):** Kondisi mendasar yang memungkinkan kelemahan terjadi, berbeda dari payload atau gejala.
* **Bukti:** Artefak yang mendukung klaim, misalnya permintaan-respons, keluaran perintah, konfigurasi, atau tangkapan layar.
* **Confidence:** Tingkat keyakinan terhadap validitas dan cakupan kesimpulan.
* **Exploitability:** Kemudahan dan prasyarat untuk memanfaatkan suatu kelemahan.
* **Finding:** Temuan terstruktur yang menggabungkan fakta, bukti, dampak, penilaian, dan remediasi.
* **Out-of-scope:** Aset atau aktivitas yang tidak diizinkan dalam engagement.
* **Penetration testing:** Pengujian aktif berizin untuk memverifikasi ketahanan kontrol terhadap upaya kompromi.
* **Retest:** Pengujian ulang terarah untuk memverifikasi perbaikan dan risiko residual.
* **Rules of Engagement:** Aturan operasional yang menetapkan cara, waktu, batas, komunikasi, dan penghentian pengujian.
* **Scope:** Batas aset, environment, akun, teknik, dan tujuan yang disetujui.
* **Severity:** Ukuran karakteristik teknis dan dampak relatif suatu kerentanan.
* **Threat modeling:** Proses mengidentifikasi aset bernilai, pelaku ancaman, jalur serangan, dan kontrol relevan.
* **Vulnerability assessment:** Proses menemukan dan memprioritaskan kemungkinan kelemahan, biasanya dengan cakupan luas dan pembuktian terbatas.

## Referensi

\[1\] OffSec, "PEN-200: Penetration Testing with Kali Linux - Syllabus," 2026. `https://manage.offsec.com/app/uploads/2026/03/PEN-200_Syllabus.pdf`

\[2\] OffSec, "OSCP+ Reporting Requirements," diperbarui 6 Agustus 2026. `https://help.offsec.com/hc/en-us/articles/360046787731-OSCP-Reporting-Requirements`

\[3\] NIST, "penetration testing - Glossary." `https://csrc.nist.gov/glossary/term/penetration_testing`

\[4\] K. Scarfone, M. Souppaya, A. Cody, dan A. Orebaugh, "Technical Guide to Information Security Testing and Assessment," NIST SP 800-115, 2008. `https://doi.org/10.6028/NIST.SP.800-115`

\[5\] Penetration Testing Execution Standard, "The Penetration Testing Execution Standard" dan "Reporting." `https://www.pentest-standard.org/`

\[6\] OWASP Foundation, "Web Security Testing Guide Reporting Structure," edisi daring terbaru. `https://owasp.org/www-project-web-security-testing-guide/latest/5-Reporting/01-Reporting_Structure`

\[7\] FIRST, "Common Vulnerability Scoring System version 4.0: Specification Document," versi dokumen 1.2. `https://www.first.org/cvss/v4.0/specification-document`

\[8\] FIRST, "CVSS v4.0 Consumer Implementation Guide," versi dokumen 1.0, 2026. `https://www.first.org/cvss/v4.0/implementation-guide`

\[9\] FIRST, "CVSS v4.0 Calculator." `https://www.first.org/cvss/calculator/4.0`

\[10\] OWASP Foundation, "Web Security Testing Guide." `https://owasp.org/www-project-web-security-testing-guide/`

\[11\] OffSec, "Authoritative References List: OSCP+ - Domain 5: Documenting Findings," 2025. `https://help.offsec.com/hc/en-us/articles/37192004980628-Authoritative-References-List-OSCP`

\[12\] Republik Indonesia, Undang-Undang Nomor 1 Tahun 2024 tentang Perubahan Kedua atas Undang-Undang Nomor 11 Tahun 2008 tentang Informasi dan Transaksi Elektronik. `https://peraturan.bpk.go.id/Details/274494/`

\[13\] Republik Indonesia, Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi. `https://peraturan.bpk.go.id/Details/229798/uu-no-27-tahun-2022`

\[14\] Institut Teknologi Sepuluh Nopember, "Panduan Identitas Visual." `https://www.its.ac.id/tentang/panduan-identitas-visual/`

*Seluruh sumber daring diakses pada 31 Agustus 2026. Versi standar, pedoman, dan peraturan perlu diverifikasi kembali sebelum digunakan dalam engagement profesional.*