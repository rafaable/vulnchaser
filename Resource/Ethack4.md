# Bab 4 — Vulnerability Scanning & Analisis Risiko

Ada satu momen yang bikin saya sadar betapa pentingnya bab ini: pernah dengar cerita pentester yang menjalankan scanner, dapat 200 halaman hasil PDF penuh warna merah menyala, lalu... langsung tempel semuanya ke laporan klien tanpa dibaca satu-satu? Klien yang menerima laporan itu panik, minta emergency meeting, padahal separuh dari 200 temuan itu adalah false positive atau severity yang di-generate scanner tanpa mempertimbangkan konteks nyata sistem mereka.

Nah, di sinilah bedanya orang yang "bisa jalanin scanner" dengan orang yang benar-benar paham vulnerability scanning sebagai sebuah disiplin. Scanner cuma alat. Yang bikin hasilnya berguna atau justru bikin geger tanpa alasan itu kamu, si operatornya.

## 4.1 Bagaimana Sebenarnya Cara Kerja Vulnerability Scanner?

Di balik antarmuka yang kelihatan "ajaib" — tinggal masukin target, klik scan, tunggu hasil — ada logika yang sebenarnya cukup straightforward.

Mayoritas scanner bekerja berdasarkan **signature/plugin matching**. Setiap plugin (kadang disebut NVT — Network Vulnerability Test — di dunia OpenVAS) berisi logika untuk mendeteksi satu kerentanan spesifik: entah dengan mengecek versi service yang terdeteksi lalu mencocokkannya ke database CVE, atau dengan benar-benar mengirim probe khusus untuk melihat bagaimana target merespons. Semakin lengkap dan update database plugin ini, semakin bagus cakupan deteksinya — makanya scanner komersial biasanya rutin banget update signature-nya, kadang harian.

Ini juga menjelaskan kenapa scanner **tidak bisa** mendeteksi kerentanan yang benar-benar baru (zero-day) atau kerentanan logika bisnis yang sifatnya unik untuk aplikasi tertentu — scanner cuma sepintar database plugin yang dimilikinya.

## 4.2 Unauthenticated vs Authenticated Scan

Ini pembeda penting yang sering diremehkan pemula.

**Unauthenticated scan** dijalankan tanpa kredensial apapun — scanner berinteraksi dengan target persis seperti attacker luar yang belum punya akses apapun. Cakupannya terbatas pada apa yang bisa "dilihat" dari luar: port terbuka, banner service, response HTTP.

**Authenticated scan** dijalankan dengan kredensial valid (biasanya level admin/root) ke sistem target. Dengan akses ini, scanner bisa mengecek versi patch yang benar-benar terpasang di level OS, konfigurasi lokal, software terinstall, bahkan registry (untuk Windows) — jauh lebih akurat dan komprehensif dibanding cuma mengintip dari luar.

Kenapa ini penting dipahami betul: kalau klien minta "scan komprehensif" tapi kamu cuma jalankan unauthenticated scan, kamu bisa saja melewatkan puluhan kerentanan kritis yang sebenarnya ada tapi tidak terlihat dari luar. Sebaliknya, authenticated scan butuh kredensial yang valid — yang berarti butuh koordinasi lebih dengan klien, dan juga berarti scope-nya harus jelas: kredensial siapa yang dipakai, level akses apa.

## 4.3 Instalasi dan Penggunaan Scanner Open-Source

Kamu nggak perlu keluar uang untuk mulai belajar. **OpenVAS/Greenbone Community Edition** adalah scanner open-source dengan database NVT yang terus di-update komunitas, sementara **Nessus Essentials** dari Tenable gratis untuk keperluan non-komersial (dengan batasan jumlah IP yang bisa di-scan).

Setelah instalasi (kita akan lakukan ini di modul praktikum), alur kerja umumnya:
1. Definisikan target (IP atau range).
2. Pilih scan policy/config — apakah full scan, scan cepat, atau scan spesifik ke satu kategori kerentanan.
3. (Opsional) masukkan kredensial untuk authenticated scan.
4. Jalankan scan, tunggu selesai (bisa dari hitungan menit sampai jam, tergantung ukuran target dan kedalaman scan).
5. Review hasil.

## 4.4 Membaca Hasil: Severity, False Positive, dan CVSS

Begitu hasil scan keluar, godaan terbesar adalah langsung percaya 100% ke semua yang ditampilkan. Jangan. Tiga hal yang perlu kamu pahami sebelum menyimpulkan apapun:

**Severity rating** yang ditampilkan scanner biasanya berdasarkan skor CVSS bawaan si plugin — tapi skor ini dihitung dalam vakum, tanpa tahu konteks nyata sistemmu. Sebuah kerentanan "critical" di database CVE bisa jadi kurang relevan kalau, misalnya, service yang terpengaruh cuma bisa diakses dari jaringan internal yang sudah sangat terisolasi.

**False positive** terjadi ketika scanner "salah tuduh" — mendeteksi kerentanan yang sebenarnya tidak ada, biasanya karena pencocokan versi yang kurang presisi (misal software sudah di-patch tapi banner version string-nya belum diupdate vendor). Ini alasan utama kenapa validasi manual itu wajib, bukan opsional.

**False negative**, kebalikannya, adalah kerentanan yang benar-benar ada tapi luput dari deteksi scanner — entah karena belum ada plugin untuk itu, atau karena kondisi eksploitasinya terlalu spesifik untuk dideteksi otomatis.

Soal **CVSS v3.1** sendiri, skor ini dihitung dari beberapa metrik yang saling berkombinasi:
- **Attack Vector (AV)** — seberapa jauh attacker perlu berada dari target (network, adjacent, local, physical).
- **Attack Complexity (AC)** — seberapa rumit kondisi yang dibutuhkan agar eksploitasi berhasil.
- **Privileges Required (PR)** dan **User Interaction (UI)** — apakah butuh akses awal tertentu, atau butuh korban melakukan sesuatu.
- **Impact** terhadap **Confidentiality, Integrity, Availability** — seberapa besar dampak ke tiga pilar keamanan informasi ini.

Kombinasi metrik-metrik ini menghasilkan skor 0.0–10.0. Memahami *breakdown*-nya (bukan cuma angka akhirnya) penting karena dua kerentanan bisa punya skor akhir yang mirip tapi profil risiko yang benar-benar berbeda — satu butuh akses fisik ke mesin, satu lagi bisa dieksekusi remote tanpa autentikasi sama sekali.

## 4.5 Vulnerability Scanning Terarah dengan Nmap NSE

Selain scanner dedicated, Nmap sendiri (yang sudah kita kenal di Bab 3) punya kumpulan script khusus vulnerability detection lewat kategori `vuln`:

```bash
nmap --script vuln <target>
```

Ini bukan pengganti scanner full-fledged seperti OpenVAS, tapi cocok untuk pengecekan cepat dan terarah — terutama kalau kamu sudah punya dugaan spesifik dari hasil enumerasi Bab 3 dan cuma ingin konfirmasi cepat tanpa menjalankan scan penuh yang memakan waktu lama.

## 4.6 Dari Data Mentah ke Risk Register

Hasil scan mentah — apalagi kalau jumlahnya puluhan atau ratusan temuan — nggak langsung berguna buat klien. Di sinilah kamu perlu menyusun **risk register**: daftar temuan yang sudah diprioritaskan berdasarkan kombinasi **likelihood** (seberapa mungkin kerentanan ini dieksploitasi di dunia nyata) dan **impact** (seberapa besar kerugian kalau benar-benar dieksploitasi).

Kenapa nggak cukup pakai skor CVSS mentah saja untuk prioritas? Karena CVSS itu context-free — dia nggak tahu bahwa server tertentu, misalnya, adalah server testing yang isinya cuma data dummy, atau sebaliknya, server yang "cuma" punya skor CVSS medium tapi ternyata menyimpan database pelanggan paling sensitif di seluruh organisasi. Risk register yang baik mempertimbangkan konteks bisnis ini, bukan cuma angka dari scanner.

## 4.7 Validasi Manual: Kenapa Ini Bukan Opsional

Saya ulangi lagi karena ini penting banget: **jangan pernah menyerahkan hasil scanner mentah ke klien tanpa validasi manual.**

Validasi manual nggak harus berarti kamu benar-benar eksploitasi setiap temuan (itu ranah Bab-bab selanjutnya di buku ini). Minimal, validasi berarti: cek ulang apakah versi yang dilaporkan scanner memang akurat, cek apakah kerentanan tersebut memang applicable ke konfigurasi spesifik target (bukan cuma "versi X biasanya rentan Y" secara generik), dan kalau memungkinkan, cari tahu apakah sudah ada mitigasi lain (seperti WAF atau patch virtual) yang membuat kerentanan tersebut sebenarnya sudah tidak eksploitable meski secara versi masih "rentan".

## 4.8 Menghubungkan CVE ke Exploit Publik

Salah satu skill yang akan sangat kamu andalkan di Bab 10 dan 13 nanti: begitu kamu tahu ID CVE spesifik dari sebuah temuan, kamu bisa langsung cek apakah sudah ada exploit publik yang terdokumentasi untuknya. **NVD (National Vulnerability Database)** dari NIST adalah sumber resmi untuk detail teknis dan skor CVSS resmi tiap CVE, sementara **Exploit-DB** sering jadi tempat pertama exploit proof-of-concept dipublikasikan begitu ditemukan.

Kebiasaan menghubungkan CVE ke exploit publik ini bukan cuma berguna untuk fase eksploitasi — ini juga jadi bagian penting analisis risiko: kerentanan dengan exploit publik yang sudah tersedia luas jelas punya *likelihood* eksploitasi jauh lebih tinggi dibanding kerentanan yang secara teori ada tapi belum ada satupun PoC publik yang terdokumentasi.

## Rangkuman Bab

- Scanner bekerja berdasarkan plugin/signature — sepintar database yang dimilikinya, tidak bisa mendeteksi zero-day atau logika bisnis unik.
- Authenticated scan jauh lebih komprehensif dibanding unauthenticated, tapi butuh koordinasi kredensial dengan klien.
- CVSS bukan cuma satu angka — pahami breakdown metriknya untuk analisis yang lebih tajam.
- Risk register mempertimbangkan konteks bisnis, bukan cuma angka mentah dari scanner.
- Validasi manual adalah kewajiban, bukan langkah opsional.
- Menghubungkan CVE ke exploit publik membantu penilaian likelihood sekaligus mempersiapkan fase eksploitasi.

## Checkpoint Mandiri

1. Bisakah kamu menjelaskan kenapa dua kerentanan dengan skor CVSS akhir yang sama bisa punya profil risiko yang sangat berbeda?
2. Kapan authenticated scan menjadi krusial dibanding cukup dengan unauthenticated scan saja?
3. Bisakah kamu menghitung skor CVSS v3.1 sebuah kerentanan sederhana secara manual, hanya dari deskripsi tekstualnya (tanpa kalkulator otomatis)?

## Tantangan Mastery

Jalankan authenticated scan terhadap target lab kamu, lalu susun laporan prioritas remediasi **1 halaman** yang ditujukan untuk "manajemen non-teknis" — tanpa satupun istilah seperti CVSS, CVE, atau nama tool yang kamu pakai. Fokus sepenuhnya pada dampak bisnis dan urutan tindakan yang perlu diambil.

## Sumber Bacaan Lanjutan

- NVD (National Vulnerability Database): https://nvd.nist.gov/
- FIRST CVSS v3.1 Calculator & Spesifikasi Lengkap: https://www.first.org/cvss/calculator/3.1
- Dokumentasi resmi Greenbone/OpenVAS: https://greenbone.github.io/docs/
- Nmap NSE Vulnerability Scripts: https://nmap.org/nsedoc/categories/vuln.html

Isi paketnya:

- **`Bab_04_Ringkasan_Textbook_Vulnerability_Scanning.md`** — cara kerja scanner (plugin/NVT matching), unauthenticated vs authenticated scan, membaca severity/false positive/CVSS breakdown secara kritis, Nmap NSE `--script vuln`, menyusun risk register berdasarkan likelihood×impact (bukan cuma skor mentah), pentingnya validasi manual, dan menghubungkan CVE ke exploit publik.
- **`Bab_04_Modul_Praktikum_Vulnerability_Scanning.md`** — praktikum lengkap: deploy DVWA, deploy OpenVAS/Greenbone, jalankan unauthenticated scan lalu authenticated scan ke Metasploitable2 (pakai kredensial `msfadmin`), bandingkan dengan Nmap NSE, susun risk register.
- **`resources/`**:
    - `docker-compose.yml` — DVWA + OpenVAS/Greenbone Community Edition (image `immauss/openvas`, sudah saya beri catatan jujur soal butuh internet untuk sinkronisasi feed NVT pertama kali).
    - `Vagrantfile` + `provision.sh` — Kali attacker dengan RAM lebih besar (OpenVAS cukup berat).
    - `check_openvas_ready.sh` — bantu cek status inisialisasi & temukan kredensial default dari log.
    - `nmap_vuln_scan.sh` — automasi NSE vuln scan.
    - `template_risk_register.md` — template lengkap dengan kolom likelihood/impact/prioritas.
    - `README.md` — panduan step-by-step plus troubleshooting (terutama soal waktu inisialisasi OpenVAS yang lama).

Catatan penting yang saya sertakan di modul: bab ini **melanjutkan Metasploitable2 dari Bab 3**, jadi pastikan itu masih jalan sebelum mulai.


# Resource Praktikum — Bab 4: Vulnerability Scanning

## Isi Paket

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Menjalankan DVWA + OpenVAS/Greenbone Community Edition |
| `Vagrantfile` | Provisioning VM Kali attacker (RAM lebih besar, OpenVAS cukup berat) |
| `provision.sh` | Instalasi Nmap dan Docker di Kali VM |
| `check_openvas_ready.sh` | Cek status inisialisasi OpenVAS & bantu temukan kredensial default |
| `nmap_vuln_scan.sh` | Automasi Nmap NSE `--script vuln` |
| `template_risk_register.md` | Template dokumentasi risk register |

## Prasyarat

- **Metasploitable2 dari Bab 3 harus masih berjalan** — bab ini melanjutkan target yang sama.
- RAM laptop disarankan minimal 8GB total, karena OpenVAS cukup memakan resource.

## Langkah 1 — Setup Kali Attacker (Jika Belum Ada dari Bab Sebelumnya)

```bash
vagrant up
vagrant ssh
```

Kalau sudah punya Kali VM dari bab sebelumnya, cukup jalankan `sudo ./provision.sh` di dalamnya.

## Langkah 2 — Jalankan DVWA & OpenVAS

Jalankan command di bawah dalam folder `resources`

```bash
docker compose up -d
```

**Penting**: inisialisasi pertama OpenVAS (mengunduh feed NVT) bisa memakan waktu **4 jam**, tergantung kecepatan internet. Pantau progresnya:

```bash
chmod +x check_openvas_ready.sh
./check_openvas_ready.sh
```

Ulangi script ini beberapa kali sampai muncul indikasi web service (GSA) siap.

Untuk memantau progress berjalannya dari program dapat dilakukan dengan menggunakan script `monitor_openvas_progress.sh`

```bash
chmod +x monitor_openvas_progress.sh
./monitor_openvas_progress.sh
```

Biarkan `monitor_openvas_progress.sh` berjalan secara terus menerus dan pantau progress meningkat maupun bertambahnya angka-angka pada output script.

Jika berhasil akan terdapat tampilan menyerupai berikut:

```text
 ./monitor_openvas_progress.sh
[*] Starting OpenVAS Live Metrics Monitor...
==========================================================
[*] Vol Data: 29463 MB | NVTs: 95256 files | SCAP: 1088 MB | CERT: 173 MB | GSAD: ACTIVE
==========================================================
[🎉] OpenVAS Web Interface is Live (GSAD Active)!
==========================================================
[+] Web Portal: http://127.0.0.1:9392
```

## Langkah 3 — Setup DVWA

Buka `http://localhost:8081`, klik tombol **"Create / Reset Database"** di halaman awal. Login dengan `admin` / `password`.

## Langkah 4 — Akses Greenbone Web Interface

Buka `https://localhost:9392` (perhatikan HTTPS). Browser akan memperingatkan soal sertifikat self-signed — ini normal untuk lab lokal, lanjutkan saja (biasanya ada opsi "Advanced" → "Proceed anyway").

**Kalau tidak bisa**, coba gunakan `http://localhost:9392` saja.

Gunakan kredensial yang ditemukan lewat `check_openvas_ready.sh` di atas (kalau ada).

## Langkah 5 — Jalankan Scan (Manual via Web Interface)

Ikuti panduan lengkap di modul praktikum (`Bab_04_Modul_Praktikum_Vulnerability_Scanning.md`) untuk:
1. Membuat target scan (Metasploitable2 dan DVWA).
2. Menjalankan unauthenticated scan.
3. Menambahkan kredensial (`msfadmin`/`msfadmin`) dan menjalankan authenticated scan.

## Langkah 6 — Nmap NSE sebagai Pembanding

```bash
chmod +x nmap_vuln_scan.sh
./nmap_vuln_scan.sh <IP-Metasploitable2>
```

## Langkah 7 — Susun Risk Register

```bash
cp template_risk_register.md ~/Lab-Notes/Bab-04-Vuln-Scan/risk-register.md
```

## Troubleshooting Umum

- **OpenVAS lama sekali initialize / sepertinya stuck**: ini wajar untuk percobaan pertama, feed NVT ukurannya cukup besar. Jangan restart container di tengah proses, tunggu saja.
- **Tidak bisa akses https://localhost:9392**: pastikan container `openvas-scanner` statusnya `Up` (`docker ps`), dan cek log untuk error.
- **Scan sangat lambat**: wajar, terutama di laptop dengan resource terbatas. Kalau perlu, kurangi scope target atau gunakan scan config yang lebih ringan dari "Full and fast".

## Catatan Keamanan Lab

- Seperti bab-bab sebelumnya, seluruh scan hanya dijalankan terhadap target di jaringan lab/host-only.
- Setelah selesai, matikan container untuk menghemat resource:
  ```bash
  docker compose down
  ```

  # Risk Register — Bab 4

## Info Umum
- **Target yang di-scan**: (Metasploitable2 / DVWA)
- **Tools yang dipakai**: (OpenVAS/Greenbone, Nmap NSE)
- **Tanggal scan**:

## Daftar Temuan (Urutkan dari Prioritas Tertinggi)

| # | Nama Temuan | CVE (jika ada) | CVSS Scanner | Likelihood (1-5) | Impact (1-5) | Prioritas Akhir | Sumber Deteksi | Rekomendasi |
|---|---|---|---|---|---|---|---|---|
| 1 | | | | | | | | |
| 2 | | | | | | | | |
| 3 | | | | | | | | |
| 4 | | | | | | | | |
| 5 | | | | | | | | |

**Catatan penilaian Likelihood & Impact:**
- Likelihood mempertimbangkan: apakah ada exploit publik? Seberapa mudah dieksploitasi? Apakah service ini terekspos ke jaringan yang lebih luas?
- Impact mempertimbangkan: data/akses apa yang bisa diperoleh attacker? Apakah ini jalur menuju sistem lain yang lebih kritis?
- Prioritas akhir TIDAK harus sama dengan skor CVSS mentah — jelaskan alasan penyesuaian jika ada di kolom catatan.

## Perbandingan Hasil OpenVAS vs Nmap NSE
> Catat: temuan mana yang HANYA muncul di OpenVAS, mana yang HANYA muncul di Nmap NSE, dan mana yang muncul di keduanya. Menurutmu kenapa ada perbedaan ini?

## Validasi Manual yang Sudah Dilakukan
> Untuk minimal 1 temuan teratas, jelaskan langkah validasi manual yang kamu lakukan (bukan sekadar percaya skor scanner mentah-mentah).

# Bab 4 (Praktikum) — Vulnerability Scanning

Di bab praktikum ini kita akan pasang "mesin scanner" sungguhan, arahkan ke target-target yang sudah kita kenal dari bab-bab sebelumnya, dan yang paling penting: belajar membaca hasilnya dengan kepala dingin, bukan langsung panik atau langsung percaya buta.

## 4.1 Target yang Dipakai

- **Metasploitable2** — lanjutan dari Bab 3, pastikan VM/container ini masih jalan.
- **DVWA (Damn Vulnerable Web Application)** — target baru untuk bab ini, akan kita deploy lewat Docker.

## 4.2 Deploy DVWA

```bash
docker compose up -d dvwa
```

Akses `http://localhost:8081` dari browser, lalu klik tombol setup database di halaman awal (DVWA butuh inisialisasi database sekali di awal — ikuti saja tombol yang muncul di layar, cukup klik "Create / Reset Database").

Login default DVWA: `admin` / `password`.

## 4.3 Deploy OpenVAS/Greenbone Community Edition

Scanner ini agak "berat" dibanding target-target sebelumnya (butuh waktu inisialisasi database NVT yang cukup lama di jalankan pertama kali), jadi bersabar ya:

```bash
docker compose up -d openvas
```

Proses inisialisasi pertama kali bisa memakan waktu **15–30 menit** (mengunduh dan sinkronisasi feed NVT). Cek statusnya dengan:

```bash
docker logs -f openvas-scanner
```

Setelah muncul indikasi siap (biasanya pesan terkait GSA/web service sudah berjalan), akses web interface di `https://localhost:9392` (perhatikan **https**, bukan http, dan browser akan komplain soal sertifikat self-signed — itu wajar untuk lab lokal, klik "lanjutkan" atau "advanced → proceed").

Login menggunakan kredensial yang tercantum di `README.md` paket resource (di-generate otomatis saat container pertama kali start — cek juga log container kalau kredensial berubah).

## 4.4 Menjalankan Unauthenticated Scan ke Metasploitable2

1. Di web interface Greenbone, masuk ke menu **Configuration → Targets**, buat target baru dengan IP Metasploitable2.
2. Masuk ke **Scans → Tasks**, buat task baru, pilih target yang barusan dibuat, gunakan scan config **"Full and fast"**.
3. Jalankan task, tunggu sampai selesai (bisa 10-20 menit tergantung resource laptop).
4. Setelah selesai, buka hasilnya di **Scans → Results**.

## 4.5 Menjalankan Authenticated Scan

Kali ini kita masukkan kredensial supaya scan bisa "masuk lebih dalam":

1. Masuk ke **Configuration → Credentials**, buat credential baru dengan username `msfadmin` dan password `msfadmin` (kredensial default Metasploitable2).
2. Edit target Metasploitable2 yang sudah dibuat sebelumnya, tambahkan credential SSH tersebut.
3. Jalankan ulang scan (bisa buat task baru atau clone task sebelumnya).
4. Bandingkan jumlah dan kedalaman temuan dibanding scan unauthenticated tadi — kamu akan lihat perbedaannya cukup signifikan.

## 4.6 Vulnerability Scan Terarah dengan Nmap NSE

Sebagai pembanding cepat, dari Kali jalankan:

```bash
nmap --script vuln <IP-Metasploitable2> -oN ~/Lab-Notes/Bab-04-Vuln-Scan/nmap-vuln-scan.txt
```

Bandingkan hasilnya dengan temuan OpenVAS — apakah ada yang saling melengkapi? Ada yang cuma terdeteksi salah satu?

## 4.7 Scan DVWA

Ulangi proses target + task di Greenbone, kali ini arahkan ke DVWA (`http://localhost:8081`). Perhatikan: scanner web umumnya perlu tahu ini target web application (pilih scan config yang sesuai, atau tetap gunakan "Full and fast" untuk latihan awal).

## 4.8 Menyusun Risk Register

Sekarang bagian terpenting: pindahkan seluruh temuan (dari kedua target, gabungan OpenVAS + Nmap NSE) ke `template_risk_register.md`. Untuk setiap temuan, isi kolom likelihood dan impact berdasarkan penilaianmu sendiri — bukan cuma nyalin skor CVSS mentah.

Minimal harus ada **5 temuan** yang terurut prioritas di risk register akhirmu.

## Checkpoint Praktikum

- [ ] DVWA berhasil di-setup dan bisa diakses.
- [ ] OpenVAS/Greenbone berhasil dijalankan dan web interface bisa diakses.
- [ ] Unauthenticated scan terhadap Metasploitable2 selesai dan hasilnya sudah dibaca.
- [ ] Authenticated scan terhadap Metasploitable2 selesai, dan kamu sudah membandingkan hasilnya dengan scan unauthenticated.
- [ ] Nmap `--script vuln` sudah dijalankan sebagai pembanding.
- [ ] Risk register (`template_risk_register.md`) sudah terisi minimal 5 temuan terprioritas.

## Sumber Daya Pendukung

Semua file teknis — `docker-compose.yml` (OpenVAS/Greenbone + DVWA), provisioning Vagrant untuk Kali, dan template risk register — ada di paket resource yang menyertai modul ini. Cek `README.md` untuk kredensial default dan troubleshooting umum (terutama soal waktu inisialisasi OpenVAS yang cukup lama).


## YouTube Tutorial

https://youtu.be/lqCSdrRq9bg


