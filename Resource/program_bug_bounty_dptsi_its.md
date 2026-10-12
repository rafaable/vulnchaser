# Program Bug Bounty DPTSI ITS

## Tentang Program

Program Bug Bounty DPTSI ITS adalah program pelaporan dan pengujian kerentanan keamanan siber secara bertanggung jawab pada website/aplikasi yang berada dalam ruang lingkup yang telah disetujui DPTSI ITS. Program ini bertujuan membantu DPTSI ITS menemukan, memvalidasi, dan memperbaiki kerentanan sebelum disalahgunakan oleh pihak yang tidak bertanggung jawab.

Peserta yang menemukan kerentanan, mematuhi seluruh aturan program, dan temuan tersebut dinyatakan valid setelah proses verifikasi dapat memperoleh **sertifikat apresiasi**.

---

## Ketentuan Utama

| Ketentuan | Ringkasan |
| :--- | :--- |
| **Izin sebelum pengujian** | Wajib registrasi, mengunggah NDA, dan memperoleh persetujuan tertulis DPTSI ITS sebelum melakukan aktivitas pengujian. |
| **Ruang lingkup** | Aset kandidat: `*.its.ac.id` dan `*.its.id`. Aset yang benar-benar boleh diuji adalah aset yang dikelola/diotorisasi DPTSI ITS dan secara eksplisit tercantum atau dikonfirmasi dalam izin/tiket. |
| **Batas eksploitasi** | Gunakan eksploitasi minimum yang diperlukan untuk membuktikan kerentanan. Jangan mengambil, mengubah, menghapus, atau menyebarluaskan data. |
| **Stabilitas layanan** | Dilarang melakukan DoS/DDoS, stress test, atau pengujian lain yang berpotensi menurunkan ketersediaan layanan. |
| **Kerahasiaan** | Detail kerentanan tidak boleh dipublikasikan tanpa persetujuan tertulis DPTSI ITS. |
| **Pelaporan** | Semua temuan dan komunikasi resmi dilakukan melalui ServiceDesk ITS. |
| **Keputusan validasi** | Penetapan valid/tidak valid, duplikat, *out-of-scope*, tingkat keparahan, dan kelayakan sertifikat ditentukan oleh Tim Keamanan Informasi/CSIRT ITS berdasarkan bukti dan ketentuan program. |

---

## Pendaftaran dan Otorisasi Pengujian

Sebelum melakukan pemindaian, eksploitasi, validasi teknis, atau aktivitas lain yang dapat dikategorikan sebagai pengujian keamanan, peserta wajib menyelesaikan proses berikut:

1. Buka ServiceDesk ITS di [servicedesk.its.ac.id](https://servicedesk.its.ac.id) lalu register atau login.
2. Buat tiket baru dengan Tipe **"Permintaan/Keluhan"**.
3. Pilih kategori **"Direktorat Pengembangan Teknologi Sistem Informasi (DPTSI) - Lapor Celah Keamanan Aplikasi (Bug Bounty)"**.
4. Gunakan subjek: `"Bug Bounty - [domain/aset target]"`.
5. Jelaskan tujuan, aset yang akan diuji, metode yang direncanakan, akun pengujian, perkiraan waktu pengujian, serta sumber IP publik yang akan digunakan.
6. Unggah NDA sesuai ketentuan peserta.
7. Tunggu verifikasi dan persetujuan tertulis DPTSI ITS. Jangan memulai pengujian sebelum status izin dinyatakan disetujui.

### Ketentuan NDA

* **WNI:** Melampirkan NDA yang telah ditandatangani dan dibubuhi e-Meterai sesuai ketentuan DPTSI ITS.
* **WNA:** NDA dapat ditandatangani secara elektronik/digital menggunakan platform penandatanganan elektronik yang dapat diverifikasi, misalnya DocuSign, Adobe Acrobat Sign, atau layanan sejenis yang diterima DPTSI ITS.
* *Template NDA WNI:* [Download NDA WNI](#)
* *Template NDA WNA:* [Download NDA WNA](#)

> **Catatan:** Persetujuan DPTSI ITS adalah izin terbatas. Jika pada saat pengujian peserta menemukan aset yang tidak tercantum dalam izin, aset milik pihak ketiga, atau kondisi yang meragukan, peserta wajib menghentikan pengujian pada aset tersebut dan meminta konfirmasi melalui ServiceDesk ITS.

---

## Ruang Lingkup Pengujian

Ruang lingkup kandidat program mencakup domain dan subdomain berikut:
* `*.its.ac.id` : Hanya aset yang dikelola/diotorisasi DPTSI ITS dan dikonfirmasi pada tiket izin.
* `*.its.id` : Hanya aset yang dikelola/diotorisasi DPTSI ITS dan dikonfirmasi pada tiket izin.

### Aset di Luar Ruang Lingkup (Out-of-Scope)
Aset berikut tidak otomatis menjadi *in-scope* walaupun menggunakan nama/domain terkait ITS:
* Layanan yang dikelola pihak ketiga.
* Layanan fakultas/unit yang tidak berada di bawah pengelolaan atau otorisasi DPTSI ITS.
* Infrastruktur vendor/SaaS.
* Akun individu.
* Aset yang secara eksplisit dikecualikan dalam tiket izin.

### Metode Pengujian yang Diperbolehkan:
* Reconnaissance pasif dan verifikasi teknis berisiko rendah terhadap aset yang telah disetujui.
* Pengujian manual terhadap kontrol autentikasi, otorisasi, *session management*, *input validation*, dan konfigurasi keamanan.
* Pemindaian otomatis dengan intensitas wajar apabila tidak mengganggu layanan dan tidak bertentangan dengan pembatasan pada tiket izin.
* Pengujian terhadap kerentanan seperti *broken access control* / IDOR, *injection*, XSS, SSRF, *file upload*, *path traversal*, *insecure deserialization*, *authentication bypass*, dan kerentanan lain sepanjang tetap mematuhi batas eksploitasi dan ruang lingkup.

---

## Rules of Engagement

1. **Gunakan akun dan data milik sendiri:** Jangan mengakses atau berinteraksi dengan akun pengguna lain tanpa izin eksplisit. Jangan menggunakan data pribadi pengguna sebagai bahan pengujian.
2. **Minimalkan dampak:** Gunakan metode paling ringan yang cukup untuk membuktikan kerentanan. Segera hentikan pengujian jika muncul error berulang, degradasi kinerja, risiko kehilangan data, atau tanda-tanda gangguan layanan.
3. **Jangan melakukan eskalasi yang tidak diperlukan:** Setelah dampak kerentanan terbukti, jangan memperluas akses, melakukan *lateral movement*, *persistence*, memasang *backdoor*, atau mengakses sistem lain.
4. **Jangan mengambil atau mengubah data:** Dilarang mengekstrak database, menyalin massal file, menghapus/mengubah data, mengubah konfigurasi, atau menjalankan aksi destruktif.
5. **Lindungi data sensitif:** Jika secara tidak sengaja melihat kredensial, token, data pribadi, atau informasi rahasia, hentikan eksploitasi pada titik tersebut, jangan menyebarluaskan, dan laporkan secara minimal dengan bukti yang disensor bila memungkinkan.
6. **Jaga kerahasiaan:** Jangan mempublikasikan detail kerentanan, PoC, *screenshot*, video, atau status perbaikan tanpa persetujuan tertulis DPTSI ITS.
7. **Gunakan jalur komunikasi resmi:** Semua komunikasi, permintaan klarifikasi, bukti, dan laporan final disampaikan melalui ServiceDesk ITS, kecuali DPTSI ITS secara tertulis memberikan kanal lain.

Sepanjang peserta bertindak dengan itikad baik dan sepenuhnya mematuhi izin tertulis, ruang lingkup, metode, serta ketentuan program ini, DPTSI ITS akan memperlakukan aktivitas tersebut sebagai pengujian keamanan yang diotorisasi untuk ruang lingkup yang disetujui. 

Apabila peserta tidak yakin apakah suatu aktivitas diperbolehkan, peserta wajib meminta klarifikasi sebelum melanjutkan pengujian.

---

## Aktivitas yang Dilarang

Untuk menjaga keamanan pengguna dan ketersediaan layanan publik, aktivitas berikut **dilarang** kecuali terdapat izin tertulis yang sangat spesifik dari DPTSI ITS:

* Denial of Service (DoS/DDoS), *stress test*, *load test*, *resource exhaustion*, atau teknik yang sengaja membebani layanan.
* *Social engineering*, *phishing*, *spear phishing*, *smishing*, *vishing*, *pretexting*, atau manipulasi pengguna/staf ITS.
* Pengujian fisik, *tailgating*, akses ruang/perangkat tanpa izin, atau manipulasi perangkat keras.
* *Brute force*, *password spraying*, *credential stuffing*, *MFA fatigue*, atau percobaan login berulang dalam volume tinggi yang dapat mengakibatkan gangguan layanan.
* Pengiriman spam, *mass email*, atau penggunaan sistem ITS untuk mengirim konten yang tidak diminta.
* Pemasangan malware, *ransomware*, *cryptominer*, *web shell* permanen, *backdoor*, *persistence*, atau payload destruktif.
* Eksfiltrasi data, *dump database*, *download* massal, perubahan/penghapusan data, atau modifikasi konfigurasi produksi.
* Pemindaian otomatis berintensitas tinggi yang dapat mengganggu layanan atau melampaui pembatasan pada izin.
* Pengujian terhadap layanan pihak ketiga/vendor yang tidak secara eksplisit termasuk dalam izin tertulis.

---

## Temuan Non-Prioritas / Informational

Temuan berikut tidak diprioritaskan atau dapat dikategorikan sebagai *informational* / *out-of-scope* apabila tidak menunjukkan dampak keamanan yang nyata:

* *Self-XSS* tanpa skenario serangan terhadap pengguna lain.
* *Clickjacking* pada halaman yang tidak memicu tindakan sensitif atau tidak menunjukkan dampak keamanan yang dapat dieksploitasi.
* Rekomendasi SSL/TLS *best practices* atau *security headers* yang bersifat *hardening* tanpa bukti dampak eksploitasi.
* *Banner/version disclosure*, *verbose error*, atau informasi teknologi yang tidak dapat digunakan untuk menimbulkan dampak keamanan bermakna.
* *Open redirect*, CSRF, CORS, atau isu konfigurasi lain tanpa skenario eksploitasi yang relevan dan dapat direproduksi.

> DPTSI ITS dapat tetap menerima laporan *informational* sebagai masukan perbaikan, namun laporan tersebut tidak otomatis memenuhi syarat untuk sertifikat apresiasi.

---

## Format dan Alur Pelaporan

### Setelah Izin Diterbitkan
1. Lakukan pengujian hanya terhadap aset dan metode yang disetujui.
2. Catat waktu pengujian, domain/URL, akun uji, *request/response* penting, serta bukti yang diperlukan.
3. Jika menemukan indikasi dampak kritikal atau gangguan layanan, hentikan eksploitasi dan segera beritahukan melalui ServiceDesk ITS.

### Informasi Minimal dalam Laporan
* Nama lengkap peserta (digunakan untuk administrasi dan sertifikat).
* Nomor tiket Bug Bounty (ServiceDesk ITS).
* Domain/aset/endpoint yang terdampak.
* Judul kerentanan dan kategori teknis (mis. CWE/OWASP bila diketahui).
* Ringkasan kerentanan dalam 2–5 kalimat.
* Tingkat keparahan yang diusulkan dan skor/vector CVSS bila tersedia.
* Prasyarat pengujian, termasuk akun/role yang digunakan.
* Langkah reproduksi yang jelas dan berurutan.
* PoC minimal: *request/response*, *screenshot*, atau rekaman video yang relevan.
* Dampak keamanan yang dapat dibuktikan, bukan hanya kemungkinan teoritis.
* Rekomendasi mitigasi/perbaikan apabila peserta memilikinya.
* Informasi apakah data sensitif terlihat secara tidak sengaja dan tindakan yang telah dilakukan untuk meminimalkan/meniadakan penyimpanan data tersebut.

### Proses Pengiriman
1. Unduh dan lengkapi Form Laporan Bug Bounty: **[Template Laporan Bug Bounty](#)**
2. Masuk ke ServiceDesk ITS dan buka tiket Bug Bounty yang telah memperoleh izin.
3. Unggah form laporan dan dokumentasi tambahan yang diperlukan. Hindari mengunggah *dump data*, kredensial aktif, atau data pribadi dalam jumlah besar.
4. Periksa kembali kelengkapan laporan, kemudian kirim pembaruan pada tiket.
5. Pantau ServiceDesk ITS dan email yang terdaftar untuk pertanyaan tambahan atau hasil validasi.

---

## Validasi, Duplikat, dan Sertifikat Apresiasi

Tim Keamanan Informasi/CSIRT ITS akan menganalisis dan memvalidasi laporan. Dalam proses triage, laporan dapat dikategorikan sebagai:
* `valid`
* `perlu informasi tambahan`
* `duplicate/known issue`
* `informational`
* `out-of-scope`
* `bukti tidak cukup`
* `tidak valid`

Keputusan akhir mengenai status laporan, tingkat keparahan, hubungan dengan laporan sebelumnya, serta kelayakan sertifikat merupakan kewenangan DPTSI ITS berdasarkan hasil validasi teknis dan kepatuhan terhadap ketentuan program.

---

## Kerahasiaan, Responsible Disclosure, dan Perlindungan Data

* Peserta wajib menjaga kerahasiaan detail teknis temuan sejak pengujian dimulai sampai DPTSI ITS memberikan persetujuan tertulis untuk publikasi.
* Jangan mengunggah PoC, *screenshot*, video, kode eksploitasi, atau detail endpoint rentan ke media sosial, repositori publik, forum, atau platform pihak ketiga tanpa persetujuan.
* Jika laporan mengandung data pribadi atau informasi sensitif, minimalkan data yang dicantumkan dan lakukan *masking/redaction* bila memungkinkan.
* Data identitas peserta yang disampaikan melalui ServiceDesk ITS digunakan untuk proses verifikasi, komunikasi program, validasi laporan, dan penerbitan sertifikat sesuai kebijakan DPTSI ITS dan ketentuan perlindungan data yang berlaku.

---

## Penghentian Izin dan Diskualifikasi

DPTSI ITS dapat meminta peserta menghentikan pengujian atau mencabut izin apabila diperlukan untuk menjaga keamanan, stabilitas, kepatuhan, atau kepentingan operasional. Pelanggaran serius terhadap aturan program dapat menyebabkan laporan tidak diproses sebagai Bug Bounty, diskualifikasi dari sertifikat, pemblokiran aktivitas pengujian, dan/atau tindak lanjut sesuai kebijakan serta peraturan yang berlaku.

---

## FAQ (Pertanyaan yang Sering Diajukan)

* **Apakah saya boleh menguji sebelum mendapat balasan ServiceDesk?**
  *Tidak. Pengujian hanya boleh dimulai setelah memperoleh persetujuan/otorisasi tertulis DPTSI ITS.*
* **Apakah semua subdomain `*.its.ac.id` dan `*.its.id` boleh diuji?**
  *Boleh, selama masih dalam ruang lingkup yang sudah ditentukan dan diotorisasi.*
* **Bolehkah menggunakan scanner otomatis?**
  *Boleh secara terbatas jika tidak mengganggu layanan dan sesuai izin. Dilarang melakukan pemindaian agresif/high-volume atau yang berpotensi menyebabkan gangguan.*
* **Bagaimana jika saya tidak sengaja melihat data pribadi atau kredensial?**
  *Hentikan eksploitasi pada titik tersebut, jangan menyalin/menyebarluaskan data, minimalkan bukti, lakukan masking jika memungkinkan, dan segera laporkan melalui ServiceDesk ITS.*
* **Bolehkah saya mempublikasikan temuan setelah melapor?**
  *Tidak tanpa persetujuan tertulis dari DPTSI ITS. Responsible disclosure harus dikoordinasikan dengan DPTSI ITS.*
* **Bagaimana jika laporan saya dianggap duplikat atau tidak valid?**
  *DPTSI ITS akan menyampaikan status melalui ServiceDesk. Penetapan akhir didasarkan pada bukti teknis, kronologi laporan, ruang lingkup, dan ketentuan program.*