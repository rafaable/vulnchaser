## Daftar Isi

1. [Ringkasan Textbook](#ringkasan-textbook)
2. [Step by Step & Link](#step-by-step--link)
3. [Template Peta Serangan](#template-peta-serangan)


## Ringkasan Textbook

Fondasi Metodologi Penetration Testing & Penulisan Laporan Teknis

Coba bayangkan begini: kamu baru saja belajar cara mengeksploitasi SQL Injection, berhasil dapat akses root di satu mesin lab, rasanya puas banget. Terus apa? Kalau kamu seorang pentester profesional, "terus apa"-nya justru bagian yang paling sering diremehkan pemula — yaitu menjelaskan ke klien apa yang baru saja kamu lakukan, kenapa itu berbahaya, dan bagaimana cara memperbaikinya. Ironisnya, di dunia kerja nyata, kemampuan menjelaskan ini sering lebih menentukan karier seorang pentester dibanding kemampuan eksploitasinya sendiri.

Makanya bab ini sengaja diletakkan di paling depan, bukan di akhir. Sebelum kita ngoprek exploit, port scanning, atau bikin reverse shell, kita perlu ngerti dulu: apa sebenarnya yang kita kerjakan, batasannya di mana, dan gimana cara menyampaikan hasilnya supaya benar-benar berguna. Tanpa fondasi ini, semua skill teknis yang kamu pelajari di bab-bab selanjutnya cuma jadi "iseng hacking" yang nggak jelas arahnya — atau lebih parah, bisa berujung masalah hukum.

## 1.1 Penetration Testing Itu Sebenarnya Apa, Sih?

Banyak orang awam — dan jujur saja, beberapa yang sudah kerja di IT juga — sering menyamakan tiga istilah ini: *penetration testing*, *vulnerability assessment*, dan *red teaming*. Padahal ketiganya beda tujuan, beda cakupan, dan beda output yang diharapkan klien.

**Vulnerability Assessment (VA)** itu ibarat medical check-up tahunan. Tujuannya luas: cari sebanyak mungkin potensi masalah kesehatan (baca: kerentanan) di seluruh sistem, biasanya pakai scanner otomatis, lalu kasih daftar hasilnya. VA nggak berusaha membuktikan bahwa kerentanan itu benar-benar bisa dieksploitasi — cukup identifikasi dan laporkan.

**Penetration Testing** lebih mirip simulasi perampokan yang disetujui pemilik toko. Kamu nggak cuma bilang "pintu belakang ini kelihatannya rapuh", tapi kamu benar-benar coba dobrak pintunya, buktikan bisa masuk, lalu tunjukkan apa yang bisa kamu ambil kalau kamu benar-benar penjahat. Fokusnya pada pembuktian dampak nyata (*proof of exploitability*), bukan sekadar daftar kemungkinan.

**Red Teaming** naik satu level lagi. Kalau pentest biasanya fokus ke satu sistem atau aplikasi dengan waktu terbatas, red team mensimulasikan serangan penuh gaya musuh sungguhan (*adversary emulation*) — termasuk menghindari deteksi tim keamanan internal (blue team), memakai kombinasi teknik teknis dan rekayasa sosial, dan biasanya berlangsung lebih lama tanpa pemberitahuan detail ke tim defense.

Kenapa perbedaan ini penting? Karena kalau kamu ditugaskan melakukan penetration testing tapi malah menjalankan mentalitas VA (cuma scan lalu tempel hasil scanner ke laporan tanpa validasi), klien akan kecewa — mereka bayar mahal untuk pembuktian nyata, bukan output mentah dari Nessus.

## 1.2 Batasan yang Tidak Boleh Dilanggar: Etika dan Hukum

Ini bagian yang sering bikin bosan tapi justru paling krusial. Serius, lebih krusial dari teknik hacking apapun yang akan kamu pelajari.

Melakukan aktivitas yang secara teknis identik — port scanning, mencoba login brute-force, mengeksploitasi kerentanan — bisa jadi hal yang legal dan dihargai, atau bisa jadi tindak pidana serius, tergantung satu hal: **apakah kamu punya izin eksplisit untuk melakukannya.**

Beberapa konsep kunci yang wajib kamu pahami sebelum menyentuh target apapun (bahkan target lab sekalipun, kalau itu bukan milikmu):

- **Rules of Engagement (RoE)** — dokumen yang menentukan apa yang boleh dan tidak boleh dilakukan selama engagement. Misalnya: apakah social engineering diizinkan? Apakah DoS testing termasuk scope? Jam berapa saja testing boleh dilakukan?
- **Scope** — daftar aset yang secara eksplisit disetujui untuk diuji. Menyerang IP atau domain di luar scope, sekalipun ditemukan "nyambung" ke target utama, adalah pelanggaran serius.
- **Get-out-of-jail letter** — semacam surat resmi dari klien yang menyatakan bahwa aktivitas testing dilakukan atas izin mereka, biasanya dibawa pentester sebagai bukti kalau ada pihak lain (misal hosting provider atau penegak hukum) yang mempertanyakan aktivitas mencurigakan.
- **Kontrak & NDA** — mengatur kerahasiaan data yang ditemukan, tanggung jawab hukum kalau terjadi kerusakan tidak disengaja, dan durasi retensi data hasil testing.

Satu hal yang perlu ditekankan: di seluruh buku ini, semua praktik hands-on dilakukan di lingkungan lab milik sendiri yang sengaja dibuat rentan (*vulnerable-by-design*) dan terisolasi dari internet. Teknik yang sama, kalau dipraktikkan ke sistem orang lain tanpa izin, bisa membawamu ke ranah pidana — terlepas dari niat baikmu.

## 1.3 Siklus Hidup Sebuah Penetration Test

Kalau kita rangkum, hampir semua metodologi pentest — apapun standarnya — mengikuti pola besar yang sama. Ini bukan urutan kaku yang harus 100% linear (kadang kamu akan bolak-balik antar tahap), tapi lebih sebagai peta mental supaya kamu nggak "nyasar" saat kerja:

1. **Pre-engagement** — negosiasi scope, RoE, jadwal, dan ekspektasi dengan klien.
2. **Reconnaissance** — mengumpulkan informasi tentang target, baik secara pasif (tanpa menyentuh sistem target langsung) maupun aktif.
3. **Scanning & Enumeration** — memetakan permukaan serangan secara lebih detail: port terbuka, service yang berjalan, versi software.
4. **Vulnerability Analysis** — mengidentifikasi celah keamanan potensial dari hasil enumerasi.
5. **Exploitation** — membuktikan kerentanan tersebut benar-benar bisa dimanfaatkan.
6. **Post-Exploitation** — setelah dapat akses, apa yang bisa dicapai lebih lanjut? Privilege escalation? Lateral movement? Data apa yang bisa diakses?
7. **Reporting** — menyampaikan seluruh temuan dengan cara yang bisa ditindaklanjuti oleh klien.

Perhatikan, tahap terakhir bukan "bonus" atau formalitas administratif. Kalau laporan yang kamu buat nggak jelas, nggak actionable, atau salah sasaran audiens, maka lima tahap sebelumnya — sekeren apapun teknik yang kamu pakai — jadi nggak ada gunanya buat klien.

## 1.4 Standar Metodologi yang Dipakai Industri

Kamu nggak perlu menciptakan metodologi dari nol. Ada beberapa kerangka kerja yang sudah teruji dan dipakai luas, dan memahami minimal satu di antaranya akan sangat membantumu berbicara "bahasa yang sama" dengan pentester lain di industri.

**PTES (Penetration Testing Execution Standard)** memberikan kerangka tujuh fase yang cukup mirip dengan siklus di atas, plus panduan teknis detail untuk masing-masing fase. Cocok dijadikan referensi struktur kerja.

**OWASP Testing Guide** lebih fokus ke aplikasi web, dan jadi rujukan hampir universal untuk siapa saja yang bekerja di ranah web application security. Kita akan banyak merujuk ke sini mulai Bab 5 dan seterusnya.

**NIST SP 800-115 ("Technical Guide to Information Security Testing and Assessment")** memberikan perspektif yang lebih formal dan sering dipakai di lingkungan pemerintahan atau korporat yang butuh kepatuhan terhadap standar tertentu.

Kamu tidak harus menghafal semua ini kata per kata. Yang penting: kalau suatu saat klien atau atasanmu bilang "kita pakai pendekatan PTES", kamu tahu harus buka dokumen apa dan cari bagian mana.

## 1.5 Note-Taking: Kebiasaan yang Membedakan Amatir dan Profesional

Ini pengalaman yang hampir semua pentester senior pernah alami: menemukan kerentanan kritis jam 2 pagi, saking excited-nya langsung lanjut eksploitasi tanpa dicatat, dan besok paginya lupa detail command yang dipakai atau parameter mana persisnya yang rentan. Alhasil harus mengulang semuanya dari awal.

Note-taking yang baik itu bukan soal rapi-rapian estetika, tapi soal **rekonstruksi**. Idealnya, siapapun (termasuk kamu sendiri enam bulan kemudian) bisa membaca catatanmu dan mengulang persis langkah yang sama untuk memverifikasi temuan tersebut.

Beberapa pendekatan yang bisa kamu pakai:

- **Metode Cornell Notes** — membagi halaman jadi area catatan utama, area kata kunci/pertanyaan, dan ringkasan di bagian bawah. Aslinya metode belajar mahasiswa, tapi cocok banget diadaptasi untuk mendokumentasikan proses investigasi teknis.
- **Mind-mapping serangan** — berguna terutama saat target kompleks (misal jaringan Active Directory dengan banyak host), supaya kamu bisa melihat gambaran besar relasi antar temuan.

Untuk toolingnya, kamu nggak perlu berlangganan software mahal. **Obsidian**, **CherryTree**, dan **Joplin** semuanya gratis dan berbasis local-first (data tersimpan di komputer kamu sendiri, bukan cloud pihak ketiga — penting untuk kerahasiaan data klien). Ada juga **Sysreptor**, tool open-source yang dirancang khusus untuk workflow pelaporan pentest dari awal sampai akhir.

## 1.6 Anatomi Laporan yang Benar-Benar Dibaca Klien

Sekarang bagian yang tadi saya bilang paling menentukan karier: bagaimana menulis laporan yang benar-benar berguna.

Fakta yang agak menyakitkan tapi perlu kamu terima: sebagian besar orang yang membaca laporanmu **tidak akan membaca semuanya**. Direktur atau manajemen akan baca satu-dua paragraf paling atas, lalu langsung ke kesimpulan. Tim teknis mungkin skip ke bagian temuan detail dan langsung cari command untuk reproduce. Karena itu, struktur laporan perlu disusun berdasarkan siapa yang membacanya, bukan berdasarkan urutan kronologis kamu bekerja.

**Executive Summary** ditulis untuk audiens non-teknis — biasanya manajemen atau pengambil keputusan. Hindari jargon. Fokus pada dampak bisnis: "Kami menemukan celah yang memungkinkan penyerang mengakses data pelanggan tanpa autentikasi", bukan "Ditemukan IDOR pada endpoint /api/users/{id}".

**Technical Summary** menjembatani antara ringkasan eksekutif dan detail teknis — cocok untuk manajer IT yang paham teknologi tapi tidak perlu detail exploit-level.

**Technical Findings** adalah bagian utama untuk tim teknis: deskripsi kerentanan, bukti (screenshot, command, response), tingkat keparahan, dan rekomendasi perbaikan spesifik.

Nah, soal tingkat keparahan ini, jangan asal bilang "kritis" atau "tinggi" berdasarkan feeling. Gunakan **CVSS (Common Vulnerability Scoring System)** versi 3.1, yang menghitung skor berdasarkan beberapa faktor terukur: seberapa mudah diakses (attack vector), seberapa kompleks eksploitasinya (attack complexity), level hak akses yang dibutuhkan penyerang, dan seberapa besar dampaknya terhadap kerahasiaan, integritas, dan ketersediaan data. Dengan begini, prioritas remediasi jadi objektif, bukan berdasarkan siapa yang paling galak protes ke manajemen.

## 1.7 Menulis Rekomendasi yang Benar-Benar Bisa Dikerjakan

Ada satu kesalahan klasik pentester pemula: menulis rekomendasi generik seperti "gunakan input validation yang lebih baik" atau "terapkan security best practice". Klien yang membaca ini biasanya cuma bisa garuk-garuk kepala — best practice yang mana? Divalidasi di mana persisnya?

Rekomendasi yang baik itu spesifik dan bisa langsung dieksekusi tim developer atau sysadmin. Alih-alih "perbaiki validasi input", tulis sesuatu seperti: "Terapkan parameterized query pada endpoint `/login` untuk mencegah SQL Injection; hindari string concatenation langsung pada query SQL seperti yang ditemukan di parameter `username`."

Semakin konkret rekomendasimu, semakin besar kemungkinan benar-benar diperbaiki — dan pada akhirnya itulah tujuan sesungguhnya dari seluruh pekerjaan pentest ini: bukan sekadar membuktikan kamu bisa masuk, tapi membantu organisasi jadi lebih aman.

## Rangkuman Bab

Sebelum lanjut ke bab berikutnya, ada baiknya kamu pastikan beberapa hal ini sudah tertanam:

- Kamu paham beda VA, pentest, dan red team — dan bisa menjelaskan ke orang awam dalam satu kalimat masing-masing.
- Kamu ngerti kenapa RoE dan scope itu bukan formalitas, tapi pagar pengaman hukum buatmu sendiri.
- Kamu punya gambaran siklus hidup pentest dari awal sampai akhir, dan tahu kalau reporting itu bukan tahap "buang-buang waktu".
- Kamu sudah pilih minimal satu tool note-taking untuk dipakai sepanjang buku ini.

## Checkpoint Mandiri

Coba jawab pertanyaan-pertanyaan ini tanpa membuka ulang materi di atas. Kalau ada yang masih terasa ragu, itu sinyal untuk baca ulang bagian terkait sebelum lanjut ke Bab 2:

1. Bisakah kamu menjelaskan bedanya *scope-creep* (pelanggaran) dengan *legitimate scope expansion* (perluasan yang sah)?
2. Kalau kamu menemukan sistem "menarik" yang ternyata di luar scope yang disepakati, apa yang seharusnya kamu lakukan?
3. Bisakah kamu menulis satu contoh temuan lengkap — mulai dari deskripsi kerentanan, bukti, skor CVSS, sampai rekomendasi — dari nol?

## Tantangan Mastery

Tanpa melihat contoh dari sumber manapun, buat template laporan pentest sendiri yang terdiri dari lima bagian: scope, ringkasan eksekutif, metodologi yang dipakai, temuan lengkap dengan skor CVSS, dan lampiran bukti. Template ini akan kamu pakai secara konsisten untuk semua sesi praktikum di buku ini — jadi luangkan waktu untuk membuatnya benar-benar nyaman dipakai berulang.

## Sumber Bacaan Lanjutan

- Penetration Testing Execution Standard: http://www.pentest-standard.org/
- OWASP Web Security Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- NIST SP 800-115, *Technical Guide to Information Security Testing and Assessment*
- FIRST CVSS v3.1 Calculator: https://www.first.org/cvss/calculator/3.1
- Arsip laporan bug bounty publik di HackerOne Hacktivity, sebagai referensi standar penulisan temuan di industri nyata

## Step by Step & Link

Setup Dokumentasi & Baseline Recon

Oke, teori sudah cukup. Sekarang saatnya kotor-kotoran tangan sedikit.

Tujuan praktikum di bab ini sebenarnya sederhana, tapi jangan salah — ini fondasi yang akan kamu pakai berulang-ulang sampai bab terakhir buku ini: (1) membangun *cyber range* pribadi di laptopmu sendiri, dan (2) membiasakan diri mendokumentasikan setiap langkah sejak awal, bukan setelah "nanti kalau sempat" (yang biasanya artinya nggak pernah).

Kalau kamu langsung loncat ke bab-bab eksploitasi tanpa membiasakan diri dengan alur di bab ini, saya jamin nanti di tengah jalan kamu bakal kebingungan sendiri: "Tadi command yang berhasil itu yang mana ya?"

## 1.1 Yang Perlu Kamu Siapkan

Sebelum mulai, pastikan tiga hal ini sudah terpasang di laptopmu:

- **VirtualBox** dengan VM **Kali Linux** yang sudah jalan (kalau belum, unduh image resmi dari kali.org — pilih versi VirtualBox pre-built supaya tidak perlu instalasi manual dari ISO).
- **Docker** terpasang di Kali VM (atau di host, tergantung arsitektur lab kamu — panduan di bagian 1.4 di bawah pakai asumsi Docker jalan di dalam Kali).
- Koneksi jaringan **host-only** antara Kali VM dan container Docker yang akan kita jalankan — supaya lab kamu tetap terisolasi dari internet.

Kalau bagian jaringan host-only ini masih bikin bingung, tenang, semua langkah teknisnya sudah saya siapkan di paket resource yang menyertai bab ini (lihat bagian akhir bab ini).

## 1.2 Kenapa Kita Mulai dari Dokumentasi, Bukan Langsung Hacking?

Mungkin agak aneh — bab praktikum pertama malah bahas soal catat-mencatat, bukan langsung nge-scan atau nge-exploit. Tapi coba pikir lagi: kalau kebiasaan dokumentasi nggak dibangun sejak sesi pertama, kebiasaan itu nggak akan muncul tiba-tiba pas kamu sampai di bab Active Directory yang jauh lebih kompleks nanti.

Jadi, sebelum menyentuh target apapun, install dulu salah satu tool note-taking yang sudah dibahas di textbook Bab 1 — **Obsidian** atau **CherryTree**, keduanya gratis dan bisa jalan langsung di Kali.

Setelah terpasang, buat struktur folder seperti ini untuk setiap "engagement" (dalam konteks buku ini, satu folder per bab praktikum):

```
Lab-Notes/
└── Bab-01-Baseline-Recon/
    ├── 00-scope-dan-tujuan.md
    ├── 01-recon-notes.md
    ├── 02-screenshot/
    └── 03-laporan-final.md
```

Struktur ini kelihatan sepele, tapi percayalah, konsistensi struktur folder seperti ini yang bakal menyelamatkanmu waktu suatu saat harus balik lagi ke catatan lama entah kenapa.

## 1.3 Menjalankan Target Pertama: OWASP Juice Shop

**OWASP Juice Shop** adalah aplikasi web yang sengaja dibuat penuh kerentanan untuk keperluan latihan — dipelihara komunitas OWASP dan dipakai luas di seluruh dunia untuk belajar web security. Kita pakai ini sebagai "target pemanasan" karena instalasinya gampang (satu baris Docker) dan aman dijadikan sasaran latihan.

Dari dalam Kali VM, jalankan:

```bash
docker run --rm -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
```

Setelah container jalan, buka browser di dalam Kali VM dan akses `http://localhost:3000`. Kalau muncul halaman toko online dengan judul "OWASP Juice Shop", berarti target sudah siap.

> Catatan: kalau kamu menjalankan Docker di *host* (bukan di dalam Kali VM), sesuaikan cara akses network-nya — pastikan Kali VM tetap bisa menjangkau container tersebut melalui jaringan host-only yang sudah kamu konfigurasi, bukan lewat internet.

## 1.4 Baseline Scan: Langkah Pertama yang (Hampir) Selalu Sama

Setiap kali kamu berhadapan dengan target baru — mau itu satu container sederhana atau jaringan korporat yang kompleks — hampir selalu langkah pertamanya sama: cari tahu dulu apa yang sebenarnya "hidup" dan terbuka di target itu.

Dari Kali, jalankan scan sederhana ke target:

```bash
nmap -sV -p 3000 <IP-target-atau-localhost>
```

Perhatikan outputnya. Kamu akan lihat informasi seperti port terbuka, service yang jalan di port tersebut, dan kalau beruntung, versi frameworknya juga (misalnya versi Express.js atau Node.js yang dipakai Juice Shop). Informasi sekecil ini sebenarnya sudah jadi "temuan" pertamamu — versi software yang ter-expose bisa jadi petunjuk kerentanan yang sudah dikenal publik (nanti kita bahas lebih dalam soal ini di Bab 4 saat masuk vulnerability scanning).

## 1.5 Menuliskan Temuan Pertamamu

Sekarang, bagian yang sering dilewatkan pemula: pindahkan hasil di atas ke dalam template laporan yang sudah kamu buat di Tantangan Mastery Bab 1 (textbook). Kalau kamu belum sempat membuat template sendiri, pakai dulu `template_laporan_pentest.md` yang sudah disediakan di paket resource bab ini sebagai starting point — tapi usahakan tetap modifikasi supaya jadi gaya kamu sendiri.

Isi minimal yang wajib ada di laporan baseline ini:

- **Scope**: satu container Juice Shop lokal, jaringan host-only, hanya untuk keperluan latihan.
- **Tools**: Nmap versi berapa, dijalankan dari Kali versi berapa.
- **Temuan awal**: port dan service yang terdeteksi, beserta versi software (jika ada).
- **Screenshot**: minimal satu tangkapan layar hasil scan dan satu tangkapan layar halaman Juice Shop yang berhasil diakses.

## Checkpoint Praktikum

Sebelum lanjut ke Bab 2, pastikan kamu sudah bisa mencentang semua ini:

- [ ] Kali Linux VM jalan normal dan bisa mengakses container Docker.
- [ ] Juice Shop berhasil diakses lewat browser dari dalam Kali.
- [ ] Kamu sudah menjalankan minimal satu scan Nmap dan memahami setiap kolom pada outputnya.
- [ ] Laporan baseline pertamamu sudah tersimpan rapi di folder `Bab-01-Baseline-Recon/03-laporan-final.md`.

Kalau salah satu poin di atas masih bolong, jangan buru-buru ke bab berikutnya. Kebiasaan yang dibangun (atau tidak dibangun) di sini akan kebawa terus sampai akhir buku.

## Sumber Daya Pendukung

Semua file teknis yang kamu butuhkan untuk praktikum bab ini — `docker-compose.yml`, script otomasi baseline scan, dan template laporan — sudah disiapkan dalam satu paket terpisah yang menyertai modul ini.


## Video Tutorial

https://youtu.be/SJBOK0tmA-I

## Template Laporan Pentest

Laporan Baseline Recon — Bab 1

## 1. Scope & Tujuan
- **Target**: 3000
- **Jaringan**: Host-only / internal, terisolasi dari internet
- **Tujuan**: Latihan baseline reconnaissance & pembiasaan dokumentasi
- **Tanggal**: 31 Agustus 2026
- **Dilakukan oleh**: Pujo

## 2. Ringkasan Eksekutif
"Dilakukan pemindaian awal terhadap satu aplikasi latihan untuk memetakan layanan yang berjalan. Ditemukan satu layanan web aktif dengan versi framework yang teridentifikasi."

## 3. Metodologi & Tools
| Tool | Versi | Fungsi |
|---|---|---|
| Nmap | (isi versi, cek dengan `nmap -V`) | Port & service scanning |
| Kali Linux | (isi versi) | Sistem operasi attacker |

## 4. Temuan

### Temuan #1 — (judul singkat, mis. "Port Web Terbuka Tanpa Pembatasan")
- **Deskripsi**: 
- **Bukti**: (lampirkan screenshot / potongan output scan)
- **Skor CVSS**: (isi setelah mempelajari CVSS lebih lanjut di Bab 4; untuk bab ini cukup catat versi service yang terdeteksi)
- **Rekomendasi**: 

## 5. Lampiran
- [ ] Screenshot hasil `nmap`
- [ ] Screenshot halaman target berhasil diakses
- [ ] File log scan mentah (`01-nmap-scan-*.txt`)
