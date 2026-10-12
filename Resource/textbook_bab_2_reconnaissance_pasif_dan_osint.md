# TEXTBOOK CYBERSECURITY BAB 2

## PROGRAM SARJANA

# Reconnaissance Pasif dan Open-Source Intelligence

**Dari sumber terbuka menuju hipotesis permukaan serangan yang terverifikasi**

**Hatma Suryotrisongko**

*Materi latihan hanya untuk lingkungan milik sendiri atau yang memiliki izin tertulis.*

## Isi Bab

Bab ini bergerak dari klasifikasi aktivitas menuju pengumpulan sumber terbuka, korelasi bukti, kerangka SARING, handoff tahap aktif, dan pengurangan paparan dari sisi defender.

* 2.1 Information Gathering dalam Siklus Penetration Testing
* 2.2 Batas antara Pasif dan Aktif
* 2.3 Menetapkan Kebutuhan Informasi
* 2.4 Anatomi Domain dan Data Registrasi
* 2.5 Search-Engine Reconnaissance
* 2.6 Certificate Transparency dan Passive DNS
* 2.7 Indeks Pemindaian Internet
* 2.8 Repositori Kode dan Jejak Pengembangan
* 2.9 Informasi Organisasi dan Identitas
* 2.10 Automasi OSINT dan Batas Penggunaan AI
* 2.11 Kerangka SARING
* 2.12 Model Data dan Paket Bukti
* 2.13 Studi Kasus Terbimbing
* 2.14 Handoff ke Tahap Aktif
* 2.15 Perspektif Defender
* 2.16 Kendali Mutu
* 2.17 Praktik Terbimbing

> *Catatan akademik. Contoh pada bab ini menggunakan aplikasi fiktif atau laboratorium vulnerable-by-design. Pembaca dilarang mempraktikkan teknik pada sistem publik tanpa otorisasi eksplisit.*

## Gagasan Utama

Reconnaissance pasif bukan kegiatan mengumpulkan sebanyak mungkin data. Kegiatan ini merupakan proses terarah untuk membangun hipotesis tentang permukaan serangan dari sumber terbuka, tanpa berinteraksi langsung dengan sistem target, sambil menjaga kewenangan, privasi, asal-usul bukti, dan ketepatan kesimpulan.

## Pendahuluan

Penguji pemula sering tergoda untuk membuka terminal dan segera menjalankan pemindaian. Kebiasaan tersebut terlihat produktif karena menghasilkan banyak keluaran, tetapi belum tentu efisien, aman, atau sesuai dengan *rules of engagement*. Sebelum satu paket pun dikirim langsung ke sistem target, banyak pertanyaan dasar dapat dijawab melalui informasi yang telah dipublikasikan, diindeks, diarsipkan, atau dikumpulkan oleh pihak ketiga. Proses inilah yang menjadi inti reconnaissance pasif dan *open-source intelligence* (OSINT).

Dalam konteks penetration testing, OSINT bukan sinonim dari pencarian acak di Internet. OSINT adalah proses sistematis untuk menentukan kebutuhan informasi, memilih sumber yang sah, mengumpulkan data secara proporsional, menilai kualitasnya, menghubungkan fakta lintas sumber, dan mengubahnya menjadi hipotesis pengujian. Hasil yang baik bukan tumpukan nama domain atau tangkapan layar, melainkan peta permukaan serangan yang memiliki sumber, waktu pengamatan, tingkat keyakinan, serta hubungan yang dapat dijelaskan.

OffSec menempatkan information gathering sebagai bagian penting dari siklus penetration testing. Capaian modul PEN-200 mencakup pembedaan pengumpulan informasi aktif dan pasif, pemahaman OSINT, serta pengumpulan informasi pasif terkait web server dan Domain Name System (DNS) [1], [2]. Bab ini memperluas capaian tersebut dengan tata kelola bukti, Registration Data Access Protocol (RDAP), Certificate Transparency (CT), indeks pemindaian Internet, repositori kode, verifikasi lintas sumber, dan penggunaan kecerdasan artifisial secara bertanggung jawab.

Semua contoh menggunakan domain dokumentasi seperti `example.com`, `example.org`, atau data fiktif. Praktik terhadap organisasi nyata hanya boleh dilakukan apabila target, teknik, sumber, dan cara pengolahan datanya telah diizinkan secara tertulis.

### Capaian Pembelajaran Bab

Setelah menyelesaikan bab ini, mahasiswa mampu:

1. menjelaskan peran information gathering dalam siklus penetration testing;
2. membedakan reconnaissance pasif, interaksi langsung berintensitas rendah, dan reconnaissance aktif berdasarkan jalur observasinya;
3. memetakan informasi organisasi, identitas, domain, DNS, sertifikat, aset Internet, dokumen, serta repositori kode dari sumber terbuka;
4. menggunakan operator mesin pencari, RDAP, Certificate Transparency, passive DNS, Shodan, Censys, dan alat agregasi OSINT secara terarah;
5. mencatat provenance, timestamp, confidence, dan keterbatasan setiap artefak;
6. membangun hipotesis permukaan serangan tanpa mengubah dugaan menjadi fakta;
7. menerapkan prinsip minimisasi data, perlindungan data pribadi, dan kewenangan; serta
8. menyusun laporan reconnaissance pasif yang dapat digunakan untuk merencanakan tahap aktif secara aman.

### Peta Konsep

| Pertanyaan Analitis | Sumber yang Relevan | Artefak yang Dihasilkan | 
| ----- | ----- | ----- | 
| **Organisasi apa yang sedang dinilai?** | Situs korporat, dokumen publik, pengumuman, profil organisasi | Daftar nama, merek, unit, lokasi, dan relasi bisnis | 
| **Ruang nama apa yang mungkin dimiliki?** | RDAP, CT log, passive DNS, mesin pencari | Domain, subdomain, nameserver, mail exchanger, riwayat resolusi | 
| **Aset apa yang pernah terlihat di Internet?** | Indeks pemindaian pihak ketiga | IP, port, banner, sertifikat, waktu observasi | 
| **Teknologi apa yang mungkin digunakan? Informasi apa yang sensitif?** | Dokumen, lowongan, repositori, data historis, indeks web. Dokumen terindeks, metadata, commit history, secret scanner | Kandidat platform, versi, cloud, SaaS, dan dependensi. Bukti paparan dengan redaksi dan prioritas respons | 
| **Seberapa dapat dipercaya kesimpulan?** | Korelasi lintas sumber dan data primer | Confidence, timestamp, contradiction log, dan kebutuhan validasi | 

## 2.1 Information Gathering dalam Siklus Penetration Testing

Information gathering adalah proses membangun pemahaman awal mengenai target dan lingkungan operasinya. Dalam NIST SP 800-115, teknik penemuan dan pengujian dipilih sesuai tujuan assessment, risiko operasi, dan tingkat akses yang tersedia [3]. Dalam PEN-200, information gathering menghubungkan tahap persiapan dengan enumeration, analisis kerentanan, dan penyusunan jalur serangan [1].

Reconnaissance pasif memiliki empat fungsi utama:

1. **Mengurangi ketidakpastian.** Penguji memperoleh kosakata awal tentang domain, unit organisasi, teknologi, dan pihak ketiga.
2. **Menajamkan scope.** Aset yang ditemukan dibandingkan dengan daftar *in-scope* dan *out-of-scope*; penemuan tidak otomatis memperluas izin.
3. **Membentuk hipotesis.** Data publik diubah menjadi pertanyaan yang kelak dapat divalidasi pada tahap aktif.
4. **Mengurangi aktivitas yang tidak perlu.** Tahap aktif dapat diarahkan kepada aset dan asumsi yang paling relevan, dengan laju serta metode yang disepakati.

MITRE ATT&CK memodelkan reconnaissance sebagai rangkaian perilaku yang dapat dilakukan lawan sebelum memperoleh akses, antara lain mencari situs terbuka, situs milik korban, basis data teknis, passive DNS, data registrasi, sertifikat digital, dan basis data hasil pemindaian Internet [4], [5]. Bagi penguji, model ini berguna untuk dua sudut pandang sekaligus: menemukan paparan secara berizin dan membantu organisasi memahami informasi apa yang juga tersedia bagi lawan.

### 2.1.1 Dari Data ke Hipotesis

Data mentah belum merupakan temuan. Nama `vpn-old.example.org` pada CT log hanya membuktikan bahwa nama tersebut pernah tercantum dalam sertifikat yang dicatat pada waktu tertentu. Data itu tidak membuktikan bahwa host masih aktif, dimiliki target, rentan, atau berada dalam scope.

Urutan penalaran yang benar adalah:
1. **observasi** – nama muncul dalam sumber tertentu;
2. **interpretasi** – nama mungkin berkaitan dengan layanan akses jarak jauh;
3. **hipotesis** – layanan tersebut mungkin masih menjadi bagian dari permukaan serangan;
4. **kebutuhan validasi** – kepemilikan, status, dan konfigurasi perlu diperiksa pada tahap aktif yang diizinkan; dan
5. **keputusan** – validasi dilakukan, ditunda, atau dihentikan berdasarkan scope dan risiko.

> **Aturan epistemik:** Tuliskan apa yang diketahui, apa yang diduga, dan apa yang belum diketahui sebagai tiga kategori yang berbeda.

## 2.2 Batas antara Pasif dan Aktif

Definisi yang paling berguna tidak didasarkan pada nama alat, melainkan pada jalur interaksi. Reconnaissance bersifat pasif relatif terhadap target apabila penguji memperoleh data dari sumber pihak ketiga atau koleksi yang telah tersedia tanpa mengirim permintaan langsung ke infrastruktur target.

### 2.2.1 Matriks Klasifikasi Aktivitas

| Aktivitas | Jalur Observasi | Kategori Operasional | Catatan | 
| ----- | ----- | ----- | ----- | 
| **Mencari `site:example.org` di mesin pencari** | Penguji $\rightarrow$ mesin pencari | Pasif terhadap target | Penyedia pencarian tetap melihat kueri | 
| **Membaca data domain melalui ICANN Lookup/RDAP** | Penguji $\rightarrow$ layanan registrasi | Pasif terhadap target | Data dapat disamarkan atau dibatasi | 
| **Menelusuri CT log melalui layanan pencarian CT** | Penguji $\rightarrow$ repositori CT | Pasif terhadap target | Membuktikan pencatatan sertifikat, bukan host aktif | 
| **Mencari host pada Shodan atau Censys** | Penguji $\rightarrow$ indeks pemindaian pihak ketiga | Pasif terhadap target | Data historis dan identifikasi layanan dapat keliru | 
| **Menelusuri repositori publik melalui antarmuka platform** | Penguji $\rightarrow$ platform repositori | Pasif terhadap target | Tetap tunduk pada kebijakan platform dan scope | 
| **Menanyakan resolver passive DNS** | Penguji $\rightarrow$ basis data passive DNS | Pasif terhadap target | Berbeda dari kueri langsung ke *authoritative* DNS target | 
| **Menjalankan `curl -I https://target`** | Penguji $\rightarrow$ web server target | Aktif berintensitas rendah | Permintaan dapat tercatat di log target | 
| **Membuka halaman target dengan browser** | Penguji $\rightarrow$ web server/CDN | Aktif berintensitas rendah | Browser dapat mengambil banyak resource tambahan | 
| **Melakukan TLS handshake langsung** | Penguji $\rightarrow$ endpoint target | Aktif berintensitas rendah | Koneksi terlihat oleh target atau penyedia jaringan | 
| **Menanyakan authoritative DNS target** | Penguji $\rightarrow$ nameserver target | Aktif berintensitas rendah | Berbeda dari membaca koleksi passive DNS | 
| **Menjalankan Nmap atau enumerasi layanan** | Penguji $\rightarrow$ host target | Aktif | Memerlukan izin teknik, target, waktu, dan laju | 
| **Menghubungi karyawan untuk meminta informasi** | Penguji $\rightarrow$ manusia | Interaksi aktif | Social engineering memerlukan izin terpisah | 

Klasifikasi tersebut memperbaiki anggapan populer bahwa `curl`, pembacaan header HTTP, atau inspeksi sertifikat langsung selalu pasif. Aktivitas itu mungkin tidak agresif, tetapi tetap menyentuh target. Karena itu, ia harus dicatat dan dilakukan di bawah aturan tahap aktif.

### 2.2.2 Pasif Tidak Berarti Anonim

Mesin pencari, layanan CT, platform kode, dan basis data aset dapat menyimpan kueri, alamat IP, akun, atau riwayat penggunaan. Penggunaan akun personal juga dapat menghubungkan kegiatan dengan identitas penguji. Oleh sebab itu, rencana engagement perlu menetapkan akun kerja, jaringan keluar, API key, retensi keluaran, dan kepatuhan terhadap *terms of service*.

Pasif juga tidak berarti bebas risiko hukum atau etika. Pengumpulan data pribadi, penggunaan breach data, pemetaan individu, atau penggabungan beberapa sumber dapat meningkatkan sensitivitas informasi. Di Indonesia, pemrosesan data perlu memperhatikan Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi [18]. Aktivitas teknis tetap harus ditempatkan dalam konteks kewenangan dan ketentuan hukum yang berlaku, termasuk perubahan rezim Informasi dan Transaksi Elektronik melalui Undang-Undang Nomor 1 Tahun 2024 [19]. Uraian ini merupakan panduan akademik, bukan nasihat hukum.

## 2.3 Menetapkan Kebutuhan Informasi

Reconnaissance yang tidak dibatasi akan berubah menjadi koleksi data tanpa akhir. Sebelum membuka alat, penguji perlu menulis *intelligence requirements*, yaitu pertanyaan yang apabila terjawab akan memengaruhi rencana pengujian.

### 2.3.1 Pertanyaan Pengarah

1. Apa nama hukum, merek, dan domain utama organisasi?
2. Aset mana yang dinyatakan *in-scope* dan bagaimana penulisan namanya?
3. Apakah ada unit, anak perusahaan, kampus, tenant, atau penyedia pihak ketiga yang harus dibedakan?
4. Informasi apa yang diperlukan untuk memilih teknik tahap aktif?
5. Data apa yang tidak boleh dikumpulkan, disalin, atau disimpan?
6. Kapan data dianggap terlalu lama untuk mendukung keputusan?
7. Bukti apa yang diperlukan untuk mengangkat kandidat aset menjadi temuan?

Setiap kebutuhan informasi sebaiknya memiliki pemilik keputusan. Sebagai contoh, daftar subdomain bukan tujuan tersendiri. Daftar itu digunakan oleh koordinator pengujian untuk memeriksa scope, oleh penguji teknis untuk menyusun daftar validasi, dan oleh pemilik aset untuk mengidentifikasi *shadow IT*.

### 2.3.2 Batas Koleksi

Definisikan batas berdasarkan lima dimensi: entitas, waktu, jenis data, sumber, dan volume. Contoh batas yang dapat dimasukkan ke RoE adalah: "mengumpulkan nama host dan metadata sertifikat untuk domain in-scope dari sumber pihak ketiga; tidak mengumpulkan isi komunikasi, credential aktif, atau data pribadi nonpublik; menyimpan hasil selama 30 hari dalam repositori engagement."

## 2.4 Anatomi Domain dan Data Registrasi

Nama domain membentuk ruang identitas teknis yang menghubungkan pengguna, aplikasi, email, dan layanan jaringan. Penguji perlu memahami relasi antara registrant, registrar, registry, nameserver, DNS record, subdomain, dan sertifikat.

### 2.4.1 Domain, Subdomain, dan Zona DNS

Pada `portal.example.org`, `.org` adalah top-level domain, `example` adalah label yang terdaftar, dan `portal` adalah subdomain. Akan tetapi, batas administratif DNS tidak selalu sama dengan batas organisasi. Sebuah subdomain dapat dikelola vendor, menunjuk ke platform cloud, atau didelegasikan kepada unit yang berbeda. Kepemilikan nama tidak selalu identik dengan kepemilikan server.

Record DNS memberikan jenis petunjuk yang berbeda:

| Record | Makna Umum | Hipotesis yang Mungkin | Batas Interpretasi | 
| ----- | ----- | ----- | ----- | 
| **A / AAAA** | Nama menuju alamat IPv4/IPv6 | Kandidat host atau layanan | CDN dan shared hosting dapat mengaburkan origin | 
| **CNAME** | Alias menuju nama lain | Dependensi SaaS/CDN atau delegasi layanan | Tidak membuktikan konfigurasi takeover | 
| **MX** | Penerima email domain | Penyedia email dan alur surat elektronik | Prioritas MX bukan indikator keamanan | 
| **NS** | Nameserver otoritatif | Penyedia DNS dan batas delegasi | Tidak membuktikan seluruh domain dikelola pihak sama | 
| **TXT** | Teks kebijakan/verifikasi | SPF, verifikasi SaaS, metadata domain | Token verifikasi belum tentu aktif | 
| **SOA** | Otoritas zona dan parameter | Zona serta pola administrasi | Serial tidak selalu dapat diartikan sebagai tanggal | 

Dalam bab ini, record tersebut diperoleh dari passive DNS atau sumber pihak ketiga. Penggunaan `dig` langsung kepada nameserver target dibahas sebagai aktivitas aktif berintensitas rendah pada bab berikutnya.

### 2.4.2 RDAP Menggantikan WHOIS untuk gTLD

WHOIS telah lama digunakan untuk melihat data registrasi domain. Namun, untuk generic top-level domain, ICANN menetapkan RDAP sebagai sumber definitif pengganti layanan WHOIS sejak 28 Januari 2025 [7]. RDAP memberikan respons terstruktur, discovery layanan otoritatif, dukungan internasionalisasi, akses aman, dan kemungkinan akses yang dibedakan.

Data yang mungkin tersedia mencakup registrar, status domain, tanggal pendaftaran atau perubahan, nameserver, dan kontak yang tidak disamarkan. Ketiadaan nama pemilik bukan kegagalan alat; kebijakan privasi dan *differentiated access* memang dapat membatasi data.

Contoh alur aman untuk latihan adalah menggunakan antarmuka ICANN Lookup terhadap domain dokumentasi, kemudian mencatat:

* **Objek:** `example.com`
* **Sumber:** ICANN Lookup / RDAP
* **Waktu akses:** `2026-08-31T10:15:00Z`
* **Fakta:** Registrar, status, nameserver, tanggal yang ditampilkan
* **Batas:** Tidak diketahui pengelola operasional host dan arsitektur layanan

Jangan menyimpulkan usia infrastruktur dari tanggal registrasi domain. Domain lama dapat menunjuk ke sistem baru; domain baru dapat menjadi aset sah, defensive registration, atau bagian dari kampanye penyalahgunaan. Penilaian memerlukan konteks tambahan.

## 2.5 Search-Engine Reconnaissance

Mesin pencari mengindeks halaman, dokumen, dan URL yang ditemukan crawler. Google mendokumentasikan `site:` untuk membatasi hasil kepada domain atau awalan URL tertentu dan `filetype:` untuk membatasi jenis file [8], [9]. OWASP WSTG memasukkan search-engine discovery sebagai teknik untuk menemukan kebocoran informasi dan membedakan pencarian langsung pada indeks dengan pencarian tidak langsung pada forum atau situs pihak ketiga [6].

### 2.5.1 Operator Dasar

Gunakan kueri sebagai alat untuk menjawab pertanyaan, bukan sebagai mantra. Contoh berikut memakai domain dokumentasi:

* `site:example.org`
* `site:example.org filetype:pdf`
* `site:example.org filetype:xlsx`
* `site:example.org inurl:login`
* `site:example.org "confidential"`
* `site:example.org -www`

Makna operator dapat berubah dan indeks tidak lengkap. `site:` tidak menghasilkan inventaris seluruh URL; hasil dipengaruhi sistem indeks, canonicalization, kebijakan pencarian, lokasi, personalisasi, dan waktu crawl. Karena itu, hasil pencarian adalah sampel observasi, bukan *ground truth*.

### 2.5.2 Dokumen dan Metadata

Dokumen publik dapat mengungkap pola penamaan, unit kerja, perangkat lunak, vendor, alamat surel, nama pembuat, atau jalur file. Namun, penguji harus membedakan informasi yang memang dimaksudkan untuk publik dari informasi yang terekspos tanpa sengaja.

Analisis dokumen mengikuti prinsip minimum yang memadai:

1. catat URL hasil pencarian dan waktu observasi;
2. unduh hanya jika diizinkan dan memang diperlukan;
3. hitung hash file apabila file menjadi bukti;
4. ekstrak metadata yang relevan tanpa memperluas koleksi;
5. samarkan data pribadi atau rahasia pada laporan; dan
6. jangan mencoba kredensial atau token yang mungkin ditemukan.

File `robots.txt` bukan mekanisme kontrol akses. Google menjelaskan bahwa file tersebut mengatur perilaku crawler dan bukan cara untuk menyembunyikan halaman dari hasil pencarian [20]. Demikian pula, keberadaan path pada indeks tidak berarti penguji boleh membukanya apabila interaksi langsung belum diizinkan.

## 2.6 Certificate Transparency dan Passive DNS

Certificate Transparency menyediakan log publik, *append-only*, dan dapat diaudit mengenai sertifikat TLS yang diterbitkan atau diamati [10]. Ekosistem ini dibuat untuk memungkinkan deteksi sertifikat yang salah terbit, tetapi data nama pada sertifikat juga berguna untuk pemetaan aset.

### 2.6.1 Membaca CT secara Benar

Pencarian nama domain pada CT log atau antarmuka seperti `crt.sh` dapat menghasilkan common name, Subject Alternative Name (SAN), issuer, serial, dan waktu pencatatan. Gunakan data tersebut untuk menemukan kandidat nama host, pola environment, dan riwayat penamaan.

Namun, CT tidak membuktikan hal-hal berikut:
1. host masih aktif;
2. DNS masih mengarah ke alamat tertentu;
3. sertifikat masih digunakan oleh endpoint;
4. nama merupakan aset yang dikelola langsung oleh organisasi;
5. wildcard mewakili daftar subdomain aktual; atau
6. endpoint memiliki kerentanan.

Wildcard seperti `*.example.org` hanya menyatakan ruang nama yang dapat dicakup sertifikat. Ia tidak mengungkap semua label di bawah domain. Nama internal yang pernah masuk CT juga harus diperlakukan sebagai informasi sensitif walaupun sudah berada di log publik.

### 2.6.2 Passive DNS

Passive DNS adalah koleksi historis respons DNS yang diamati oleh penyedia atau sensor. MITRE membedakannya dari kueri langsung ke nameserver target [5]. Data passive DNS dapat membantu menjawab pertanyaan "nama ini pernah mengarah ke alamat apa?" atau "alamat ini pernah berkaitan dengan nama apa?".

Setiap record harus dicatat bersama *first seen*, *last seen*, sumber, dan tipe record. Hubungan historis tidak membuktikan kepemilikan saat ini. IP cloud dapat digunakan ulang; shared hosting dapat menghubungkan banyak domain yang tidak berkaitan; dan alamat origin lama dapat sudah berpindah tangan.

### 2.6.3 Korelasi CT dan Passive DNS

Korelasi yang sehat menggunakan CT untuk menghasilkan kandidat nama, kemudian passive DNS untuk mencari hubungan historis tanpa menyentuh target. Hasilnya masuk ke candidate asset register, bukan langsung ke daftar target aktif.

| Kandidat | Sumber CT | Passive DNS | Status Analitis | Tindakan Berikutnya | 
| ----- | ----- | ----- | ----- | ----- | 
| `portal.example.org` | SAN, diamati 2026-07 | Resolusi historis tersedia | Confidence sedang | Konfirmasi scope dan validasi aktif | 
| `vpn-old.example.org` | Sertifikat kedaluwarsa 2021 | Last seen 2022 | Kandidat historis | Tanyakan pemilik aset; jangan probe | 
| `*.dev.example.org` | Wildcard | Tidak ada nama spesifik | Ruang nama, bukan host | Jangan mengarang daftar subdomain | 

## 2.7 Indeks Pemindaian Internet

Shodan dan Censys mengumpulkan data dari pemindaian Internet yang dilakukan oleh layanan mereka. Penguji kemudian menelusuri indeks tersebut tanpa memindai target secara langsung. MITRE mengelompokkan perilaku ini sebagai pencarian *scan databases* [5].

### 2.7.1 Shodan

Shodan menyediakan filter dengan bentuk `filter:value` [11]. Contoh defensif untuk aset milik sendiri dapat memakai `hostname:`, `org:`, `net:`, `port:`, atau `ssl:`. Kueri harus dimulai dari identifier yang telah diverifikasi, bukan sekadar nama organisasi yang mudah ambigu.

* `hostname:example.org`
* `org:"Example Organization"`
* `net:192.0.2.0/24`
* `ssl.cert.subject.cn:"example.org"`

Rentang `192.0.2.0/24` adalah TEST-NET-1 untuk dokumentasi; jangan menggantinya dengan rentang pihak lain tanpa izin. Hasil Shodan memuat waktu pengumpulan dan banner yang dilihat crawler. Informasi kerentanan yang diinferensikan dari versi tidak selalu terverifikasi; Shodan sendiri membedakan indikasi *verified* dan *unverified* [11].

### 2.7.2 Censys

Censys menyediakan pencarian terhadap host, web property, dan sertifikat dengan full-text maupun field-value query [12]. Model data serta sintaks dapat berubah; query lama tidak boleh diasumsikan berlaku selamanya. Censys juga menghubungkan data sertifikat dari CT log dan hasil observasi Internet.

Data Shodan dan Censys sebaiknya dibandingkan karena cakupan, waktu pemindaian, protokol, dan metode klasifikasinya berbeda. Ketidaksesuaian bukan bukti bahwa salah satu layanan "salah"; ia adalah sinyal bahwa data memiliki waktu dan vantage point yang berbeda.

### 2.7.3 Aturan Interpretasi

1. Selalu catat `observed_at`, bukan hanya waktu pengambilan dari platform.
2. Pisahkan banner mentah dari label yang dibuat platform.
3. Anggap versi sebagai indikasi sampai tervalidasi; backport patch dapat membuat banner menyesatkan.
4. Jangan menganggap IP cloud atau CDN sebagai milik eksklusif target.
5. Jangan mencoba login, mengunduh data, atau mengeksploitasi layanan dari hasil indeks.
6. Gunakan hasil sebagai dasar konfirmasi scope dan perencanaan tahap aktif.

## 2.8 Repositori Kode dan Jejak Pengembangan

Repositori publik dapat memuat source code, issue, pull request, workflow CI/CD, artefak rilis, dokumentasi, dan histori commit. Informasi tersebut dapat mengungkap nama layanan, pola environment, dependensi, endpoint, atau secret yang terekspos.

### 2.8.1 Pencarian Berbasis Identitas yang Terverifikasi

Mulailah dari organisasi atau akun yang dapat dihubungkan dengan target melalui situs resmi, bukan sekadar kesamaan nama. Setelah itu, cari domain email, nama paket, endpoint, atau pola konfigurasi yang relevan. Setiap hubungan diberi status:
1. **terverifikasi** – ada bukti resmi atau dua sumber independen;
2. **kemungkinan** – indikator konsisten tetapi belum cukup;
3. **tidak terverifikasi** – hanya kesamaan nama atau dugaan; atau
4. **ditolak** – bukti menunjukkan entitas berbeda.

### 2.8.2 Secret dalam Histori Git

Menghapus file dari commit terbaru tidak otomatis menghilangkannya dari histori. GitHub menyarankan segera mencabut atau merotasi secret yang terekspos; pembersihan histori dilakukan setelah risiko akses ditangani [13]. Gitleaks dan TruffleHog dapat mendeteksi pola credential pada repositori dan histori [14], [15].

Dalam engagement, penemuan kandidat secret harus memicu prosedur eskalasi, bukan uji coba credential. Catat lokasi secara minimum, samarkan nilainya, hubungi pemilik melalui jalur RoE, dan minta organisasi memvalidasi status secret. Jangan menyalin seluruh repository apabila pencarian platform atau clone terbatas sudah cukup.

Contoh catatan yang aman:

* **Artefak:** Commit `7f3...` pada repositori terverifikasi milik target
* **Indikator:** Pola token penyedia cloud; nilai disamarkan
* **Confidence:** Tinggi untuk paparan, belum diketahui apakah token aktif
* **Tindakan:** Eskalasi segera; jangan melakukan autentikasi
* **Rekomendasi:** Revoke/rotate, telusuri log penggunaan, bersihkan histori, aktifkan push protection

### 2.8.3 Metadata Pengembangan

Informasi seperti nama branch, file pipeline, badge, package manifest, dan issue dapat membantu pemetaan teknologi. Akan tetapi, penggunaan suatu dependensi tidak membuktikan bahwa versi produksi identik dengan versi repository. Repository dapat tidak aktif, berupa fork, *proof of concept*, atau tidak pernah dideploy.

## 2.9 Informasi Organisasi dan Identitas

Situs resmi, laporan tahunan, pengadaan, lowongan, publikasi, konferensi, dan profil profesional dapat mengungkap unit, lokasi, teknologi, vendor, serta pola alamat surel. Informasi ini bermanfaat untuk memahami konteks bisnis, tetapi mudah berubah menjadi pengawasan individu yang tidak proporsional.

### 2.9.1 Minimisasi Data Pribadi

Kumpulkan hanya atribut yang mendukung tujuan yang disetujui. Jika tujuan adalah memetakan pola alamat surel, tidak perlu menyimpan tanggal lahir, nomor telepon personal, keluarga, atau aktivitas pribadi. Gunakan fungsi/peran pada laporan apabila identitas individu tidak diperlukan.

Hindari praktik berikut:
1. mengumpulkan data keluarga atau lokasi personal;
2. menyimpan foto identitas tanpa kebutuhan;
3. mengakses akun dengan identitas palsu;
4. melakukan *pretexting* tanpa izin social engineering;
5. membeli data hasil pelanggaran atau memakai sumber ilegal;
6. menggabungkan data sehingga menghasilkan profil yang melampaui tujuan engagement; dan
7. memasukkan data pribadi mentah ke layanan AI publik.

*Berkeley Protocol* menekankan metodologi, verifikasi, preservasi, dan profesionalisme dalam penggunaan informasi digital terbuka [16]. Walaupun konteks protokol tersebut lebih luas daripada penetration testing, prinsip provenance dan verifikasinya relevan untuk disiplin OSINT.

## 2.10 Automasi OSINT dan Batas Penggunaan AI

Alat seperti `theHarvester` dapat menggabungkan sumber publik dan menormalisasi hostname, alamat surel, URL, ASN, atau data lain [14]. Automasi menghemat waktu, tetapi tidak menghapus kewajiban untuk memahami sumber, lisensi, biaya API, rate limit, dan jenis interaksi yang dilakukan modul.

### 2.10.1 Passive Mode Harus Diverifikasi

Label "passive" pada alat tidak selalu menjamin seluruh modul hanya menggunakan pihak ketiga. Beberapa alat dapat mengaktifkan DNS resolution, screenshot, virtual-host check, atau takeover check. `theHarvester`, misalnya, memisahkan pengumpulan sumber publik dari sejumlah aksi opsional yang dapat menambah interaksi [14]. Sebelum menjalankan alat:

1. baca dokumentasi dan versi alat;
2. identifikasi setiap provider atau modul;
3. nonaktifkan modul yang menyentuh target;
4. uji perilaku dalam laboratorium dan amati traffic jaringan;
5. simpan konfigurasi serta daftar provider; dan
6. catat error, rate limit, serta coverage yang hilang.

### 2.10.2 Peran LLM

Large language model (LLM) dapat membantu mengelompokkan nama aset, merangkum dokumen, menormalisasi format, atau menghasilkan pertanyaan verifikasi. Namun, LLM tidak boleh dijadikan sumber fakta. NIST Generative AI Profile memasukkan *confabulation* dan tantangan validitas informasi sebagai risiko yang perlu dikelola [17].

Gunakan LLM di dalam pagar berikut:
1. masukkan hanya data yang diizinkan dan telah disamarkan;
2. minta model mengutip record sumber untuk setiap klaim;
3. larang model mengisi atribut yang tidak tersedia;
4. verifikasi nama, waktu, dan relasi terhadap sumber asli;
5. simpan prompt, versi model, dan output apabila digunakan dalam analisis; dan
6. beri label jelas pada inferensi yang dibantu AI.

> **Prinsip kendali:** AI boleh membantu menyusun dan menantang hipotesis; keputusan fakta, scope, confidence, serta eskalasi tetap berada pada analis manusia.

## 2.11 Kerangka SARING untuk Reconnaissance Pasif

Bab ini memperkenalkan kerangka SARING sebagai alur kerja yang dapat diulang.

### 2.11.1 S – Scope dan Sasaran Keputusan
Tentukan entitas, domain, jenis data, rentang waktu, dan pertanyaan yang hendak dijawab. Hubungkan setiap kebutuhan informasi dengan keputusan berikutnya. Aset di luar scope hanya dicatat seminimal mungkin dan dieskalasikan.

### 2.11.2 A – Akuisisi Nonintrusif
Pilih sumber pihak ketiga yang sesuai. Gunakan akun engagement, rate limit, dan teknik yang disetujui. Jangan mengunduh lebih banyak data daripada yang dibutuhkan.

### 2.11.3 R – Rekam Provenance
Untuk setiap artefak, simpan URL atau identifier sumber, query, waktu akses, waktu observasi sumber, hash bila relevan, akun atau alat, serta kondisi akses. Tangkapan layar tanpa URL dan waktu memiliki nilai verifikasi yang lemah.

### 2.11.4 I – Integrasikan Lintas Sumber
Hubungkan entitas dengan alasan eksplisit. Sebuah domain dapat terkait organisasi karena tercantum pada situs resmi, memiliki data registrasi konsisten, dan digunakan pada repository terverifikasi. Jangan membuat *edge* hanya karena kemiripan nama.

### 2.11.5 N – Nilai Confidence dan Kontradiksi
Gunakan skala yang konsisten:

| Confidence | Kriteria Minimum | Contoh | 
| ----- | ----- | ----- | 
| **Tinggi** | Sumber primer atau dua sumber independen yang konsisten; data mutakhir | Domain tercantum pada situs resmi dan RDAP konsisten | 
| **Sedang** | Satu sumber teknis terpercaya atau beberapa indikator tidak langsung | Nama host pada CT dan passive DNS historis | 
| **Rendah** | Satu sumber lama, label otomatis, atau kesamaan nama | Label teknologi dari banner berusia dua tahun | 
| **Tidak dapat dinilai** | Sumber tidak tersedia, waktu tidak diketahui, atau data saling bertentangan | Screenshot tanpa provenance | 

*Contradiction log* penting karena sumber dapat tidak sinkron. Jangan memilih bukti yang hanya mendukung dugaan awal.

### 2.11.6 G – Gunakan sesuai Kewenangan
Ubah hasil menjadi candidate asset register, hipotesis, risiko informasi, dan rencana validasi. Jangan melakukan autentikasi, exploit, atau komunikasi kepada individu tanpa izin tambahan. Lindungi keluaran OSINT seperti laporan penetration testing karena gabungan data publik dapat menghasilkan informasi sensitif baru.

## 2.12 Model Data dan Paket Bukti

Spreadsheet sederhana sudah memadai apabila kolomnya disiplin. Model minimum terdiri dari:

| Kolom | Isi | 
| ----- | ----- | 
| `entity_id` | ID stabil, misalnya `DOM-002` atau `HOST-017` | 
| `entity_type` | domain, hostname, IP, certificate, repository, person-role, document | 
| `value` | Nilai yang telah dinormalisasi | 
| `source` | URL, platform, API, atau dataset | 
| `query` | Query yang digunakan | 
| `accessed_at` | Waktu penguji mengakses sumber | 
| `observed_at` | Waktu data diamati oleh sumber, jika tersedia | 
| `relationship` | Alasan hubungan dengan entitas lain | 
| `confidence` | Tinggi, sedang, rendah, atau tidak dapat dinilai | 
| `scope_status` | In-scope, out-of-scope, unknown, pending approval | 
| `sensitivity` | Publik, internal, rahasia, data pribadi | 
| `next_action` | Konfirmasi, eskalasi, validasi aktif, atau tutup | 

### 2.12.1 Normalisasi

Ubah domain menjadi huruf kecil, hapus titik terminal yang tidak diperlukan, dan pertahankan Internationalized Domain Name dalam bentuk Unicode serta Punycode jika relevan. Untuk IP, jangan menghilangkan konteks prefix. Untuk sertifikat, gunakan fingerprint agar record tidak hanya bergantung pada common name.

### 2.12.2 Deduplication Bukan Penghapusan Provenance

Beberapa sumber dapat menghasilkan entitas yang sama. Gabungkan entitasnya, tetapi pertahankan semua record sumber. Jumlah sumber dan kesesuaian waktu dapat meningkatkan confidence; menghapus duplikasi mentah dapat menghilangkan bukti korelasi.

### 2.12.3 Snapshot dan Perubahan

Reconnaissance adalah pengamatan pada waktu tertentu. Simpan snapshot awal dan bandingkan pada retest atau engagement berikutnya. Perubahan sertifikat, DNS, repository, dan service exposure dapat menunjukkan perbaikan, migrasi, atau shadow IT baru.

## 2.13 Studi Kasus Terbimbing: Universitas Fiktif

Sebuah universitas fiktif, Universitas Nusa Digital (UND), menugaskan pengujian terhadap `und.example`. RoE mengizinkan reconnaissance pasif melalui mesin pencari, RDAP, CT, passive DNS, indeks Shodan/Censys, dan repositori publik. Interaksi langsung dengan host, pengumpulan data pribadi nonpublik, serta penggunaan credential dilarang pada fase ini.

### 2.13.1 Kebutuhan Informasi

Tim menetapkan tiga pertanyaan:
1. ruang nama dan aset Internet apa yang kemungkinan berhubungan dengan domain utama;
2. teknologi dan penyedia eksternal apa yang mungkin memengaruhi rencana aktif; dan
3. apakah ada informasi sensitif yang perlu segera dieskalasikan.

### 2.13.2 Dataset Fiktif

| ID | Observasi | Sumber | Waktu Sumber | Confidence Awal | 
| ----- | ----- | ----- | ----- | ----- | 
| `CT-01` | `sso.und.example`, sertifikat masih berlaku | CT log | 2026-08-15 | Sedang | 
| `CT-02` | `vpn-old.und.example`, sertifikat berakhir 2022 | CT log | 2022-03-10 | Rendah untuk status kini | 
| `PDNS-01` | `sso.und.example` pernah menuju `192.0.2.44` | Passive DNS | Last seen 2026-08-20 | Sedang | 
| `IDX-01` | `192.0.2.44` menampilkan produk reverse proxy | Indeks pihak ketiga | Observed 2026-08-18 | Sedang | 
| `REP-01` | Repository organisasi mencantumkan `sso.und.example` | Platform kode | Commit 2026-07-11 | Tinggi untuk relasi | 
| `REP-02` | Pola secret ditemukan pada histori; nilai disamarkan | Secret scanner | Commit 2025-12-02 | Tinggi untuk paparan, status secret unknown | 

Alamat IP menggunakan TEST-NET untuk dokumentasi. Tim tidak mencoba membuka host atau menggunakan secret.

### 2.13.3 Korelasi

`CT-01`, `PDNS-01`, `IDX-01`, dan `REP-01` membentuk hipotesis bahwa `sso.und.example` adalah aset relevan. Confidence hubungan meningkat karena repository terverifikasi memakai nama yang sama. Namun, status in-scope tetap harus dikonfirmasi. `CT-02` hanya dicatat sebagai aset historis; tidak ada alasan untuk memindainya.

`REP-02` diperlakukan sebagai *potential exposed secret*. Tim mengirim eskalasi sesuai RoE dengan commit identifier, jenis pola, redaksi nilai, dan rekomendasi revoke/rotate. Tim tidak melakukan validasi melalui login karena tindakan tersebut akan melampaui fase dan teknik yang diizinkan.

### 2.13.4 Candidate Asset Register

| Entitas | Relasi | Confidence | Scope | Rencana | 
| ----- | ----- | ----- | ----- | ----- | 
| `sso.und.example` | CT + passive DNS + repository | Tinggi | Pending approval | Konfirmasi pemilik; validasi aktif terjadwal | 
| `192.0.2.44` | Resolusi historis dan indeks | Sedang | Unknown | Konfirmasi kepemilikan dan cloud tenancy | 
| `vpn-old.example.org` | CT historis | Rendah | Unknown | Tanyakan status; tidak ada probing | 
| Commit dengan kandidat secret | Repository terverifikasi | Tinggi untuk paparan | In-scope repository | Eskalasi, revoke/rotate, audit log | 

### 2.13.5 Contoh Ringkasan Reconnaissance

Tim mengidentifikasi satu kandidat layanan single sign-on yang didukung oleh empat sumber independen serta satu nama layanan historis yang belum dapat dipastikan masih digunakan. Sebuah kandidat secret juga ditemukan pada histori repository terverifikasi. Nilai secret tidak diuji dan telah disamarkan. Organisasi disarankan segera memvalidasi serta merotasi credential tersebut, kemudian mengonfirmasi kepemilikan `sso.und.example` dan alamat terkait sebelum tahap aktif. Data indeks pihak ketiga mencerminkan observasi pada waktu tertentu dan belum membuktikan konfigurasi saat ini.

## 2.14 Dari Reconnaissance ke Tahap Aktif

Handoff yang baik tidak menyerahkan ribuan baris data. Ia menyediakan prioritas, asumsi, batas, dan acceptance criteria.

### 2.14.1 Isi Handoff Minimum

1. daftar kandidat aset beserta scope status;
2. sumber dan confidence setiap relasi;
3. data historis yang tidak boleh dianggap aktual;
4. indikasi pihak ketiga, CDN, atau shared infrastructure;
5. dugaan teknologi yang perlu fingerprinting aktif;
6. temuan sensitif yang sudah dieskalasikan;
7. teknik aktif yang diusulkan, rate limit, dan jendela waktu; serta
8. pertanyaan yang belum terjawab.

### 2.14.2 Stop Conditions

Hentikan atau eskalasikan apabila:
1. aset tampak dimiliki pihak ketiga yang tidak tercakup;
2. ditemukan data pribadi atau rahasia dalam jumlah besar;
3. alat hendak mengaktifkan probing langsung;
4. sumber mensyaratkan tindakan yang melanggar kebijakan platform;
5. credential, private key, atau token mungkin masih aktif;
6. hubungan aset terlalu lemah untuk membenarkan tahap aktif; atau
7. permintaan pengumpulan melampaui tujuan engagement.

## 2.15 Perspektif Defender: Mengurangi Paparan OSINT

Reconnaissance pasif juga dapat digunakan untuk External Attack Surface Management. Defender dapat menjalankan proses SARING secara berkala terhadap domain dan organisasi sendiri untuk menemukan aset terlupakan, sertifikat tidak dikenal, DNS historis, repository publik, atau dokumen yang terindeks.

### 2.15.1 Prioritas Perbaikan

1. **Kendalikan inventaris.** Hubungkan domain, sertifikat, IP, cloud account, dan repository dengan pemilik layanan.
2. **Pantau CT dan DNS.** Buat alert untuk penerbitan sertifikat atau perubahan record yang tidak dikenal.
3. **Kurangi banner dan metadata.** Tampilkan informasi layanan yang diperlukan saja; hapus komentar, path, dan metadata sensitif dari artefak publik.
4. **Kelola secret.** Terapkan secret scanning, push protection, vault, rotasi, dan prosedur respons.
5. **Kelola dokumen publik.** Gunakan classification, metadata scrubbing, review sebelum publikasi, dan kontrol indexing yang benar.
6. **Tata kelola akun dan repository.** Verifikasi organisasi resmi, nonaktifkan aset tidak terpakai, serta dokumentasikan fork dan archive.
7. **Uji dari luar.** Bandingkan inventaris internal dengan apa yang dilihat mesin pencari, CT, passive DNS, dan scan databases.

*Security by obscurity* tidak menggantikan kontrol akses. Tujuan mengurangi paparan adalah menghilangkan informasi yang tidak perlu, menurunkan kebingungan inventaris, dan mempercepat deteksi—bukan menganggap sistem aman hanya karena sulit ditemukan.

## 2.16 Kendali Mutu Reconnaissance Pasif

Sebelum hasil diserahkan, lakukan pemeriksaan berikut.

### 2.16.1 Validitas
1. Apakah setiap klaim menunjuk kepada record sumber?
2. Apakah waktu observasi dibedakan dari waktu akses?
3. Apakah data historis diberi label?
4. Apakah relasi organisasi didukung bukti, bukan kesamaan nama?
5. Apakah hasil otomatis telah ditinjau manusia?

### 2.16.2 Kepatuhan
1. Apakah sumber dan teknik berada dalam RoE?
2. Apakah alat benar-benar tidak menyentuh target?
3. Apakah data pribadi diminimalkan dan dilindungi?
4. Apakah terms of service dan batas API dipatuhi?
5. Apakah credential atau secret tidak diuji?

### 2.16.3 Kegunaan
1. Apakah candidate asset register memiliki pemilik keputusan?
2. Apakah confidence dan contradiction terlihat?
3. Apakah handoff menjelaskan validasi aktif yang diperlukan?
4. Apakah keluaran dapat direproduksi dengan query dan timestamp?
5. Apakah pembaca dapat membedakan fakta, inferensi, dan pertanyaan terbuka?

## 2.17 Praktik Terbimbing

### 2.17.1 Tujuan

Mahasiswa menyusun profil reconnaissance pasif dari dataset fiktif tanpa mengirim traffic ke target. Praktik dapat menggunakan record yang disediakan dosen atau domain milik mahasiswa sendiri dengan izin eksplisit.

### 2.17.2 Tugas

1. tulis tiga intelligence requirements;
2. klasifikasikan sepuluh aktivitas sebagai pasif, aktif berintensitas rendah, aktif, atau interaksi manusia;
3. analisis dataset RDAP, CT, passive DNS, indeks Internet, dan repository yang telah disediakan;
4. bangun entity register dengan provenance, waktu, confidence, scope status, dan sensitivity;
5. buat minimal lima hipotesis serta kebutuhan validasinya;
6. identifikasi satu kontradiksi atau keterbatasan data;
7. tulis handoff satu halaman untuk tahap aktif; dan
8. berikan tiga rekomendasi defensif.

### 2.17.3 Rubrik Penilaian

| Komponen | Bobot | Kriteria Unggul | 
| ----- | ----- | ----- | 
| **Kewenangan dan klasifikasi** | 20% | Semua aktivitas diklasifikasikan berdasarkan jalur interaksi; batas scope jelas | 
| **Provenance dan waktu** | 20% | Setiap record memiliki sumber, query, `accessed_at`, dan `observed_at` bila tersedia | 
| **Korelasi dan confidence** | 25% | Relasi memiliki alasan; kontradiksi dan data historis ditangani benar | 
| **Hipotesis dan handoff** | 20% | Hipotesis dapat diuji, prioritas jelas, dan tidak mengubah dugaan menjadi fakta | 
| **Privasi dan komunikasi** | 15% | Data diminimalkan, rahasia disamarkan, bahasa formal dan dapat ditindaklanjuti | 

### 2.17.4 Larangan Praktik

Mahasiswa tidak boleh melakukan port scanning, membuka halaman target tanpa izin, login, mencoba token, menghubungi pegawai, mengunduh breach data, atau melakukan social engineering. Jika alat mengaktifkan resolusi DNS, screenshot, takeover check, atau probing langsung, modul tersebut harus dinonaktifkan.

## Rangkuman Bab

1. Reconnaissance pasif membangun hipotesis permukaan serangan dari sumber pihak ketiga tanpa interaksi langsung dengan sistem target.
2. Klasifikasi ditentukan oleh jalur observasi. Browser, curl, TLS handshake, dan kueri langsung ke authoritative DNS merupakan aktivitas aktif berintensitas rendah.
3. RDAP telah menjadi pengganti definitif WHOIS untuk data registrasi gTLD; data kontak dapat tetap dibatasi.
4. Mesin pencari, CT, passive DNS, scan databases, dokumen, serta repository memiliki coverage, waktu, dan bias yang berbeda.
5. CT membuktikan pencatatan sertifikat, bukan keberadaan atau kerentanan host saat ini.
6. Shodan dan Censys menyediakan observasi pihak ketiga; banner, label, dan indikasi kerentanan tetap memerlukan validasi.
7. Kandidat secret tidak boleh dicoba. Nilainya disamarkan, kemudian revoke/rotate dan pemeriksaan log diprioritaskan.
8. Kerangka SARING menjaga agar proses dimulai dari scope, menggunakan akuisisi nonintrusif, merekam provenance, mengintegrasikan sumber, menilai confidence, dan berakhir pada penggunaan yang berwenang.
9. LLM dapat membantu organisasi data, tetapi bukan sumber fakta dan tidak boleh menerima data sensitif tanpa kontrol.
10. Keluaran utama adalah candidate asset register, hipotesis, serta handoff tahap aktif—bukan koleksi data sebanyak-banyaknya.

## Uji Pemahaman

### Pilihan Ganda

 1. Aktivitas yang paling tepat disebut pasif terhadap target adalah:
    * A. Membuka halaman login target
    * B. Menjalankan `curl -I`
    * **C. Menelusuri CT log melalui layanan pihak ketiga**
    * D. Menanyakan authoritative DNS target

 2. Munculnya `vpn-old.example.org` pada CT log membuktikan bahwa ...
    * A. Host sedang aktif
    * B. Host rentan
    * **C. Nama pernah tercantum pada sertifikat yang dicatat**
    * D. Host pasti dimiliki target saat ini

 3. Perbedaan utama passive DNS dan kueri DNS langsung adalah:
    * A. Passive DNS selalu lebih mutakhir
    * **B. Passive DNS membaca koleksi observasi pihak ketiga**
    * C. Kueri langsung tidak dapat dicatat
    * D. Passive DNS membuktikan kepemilikan IP

 4. Fungsi confidence adalah:
    * A. Menggantikan bukti sumber
    * **B. Mengukur tingkat dukungan terhadap suatu klaim**
    * C. Menentukan bahwa aset boleh dieksploitasi
    * D. Menghapus kebutuhan timestamp

 5. Tindakan pertama ketika menemukan kandidat secret aktif adalah ...
    * A. Mencoba login untuk memastikan
    * B. Mempublikasikan tangkapan layar
    * **C. Menyamarkan nilai dan melakukan eskalasi untuk revoke/rotate**
    * D. Menghapus repository target

 6. RDAP penting karena ...
    * A. Menjamin semua identitas registrant terlihat
    * **B. Menggantikan WHOIS sebagai sumber definitif gTLD dan memberi data terstruktur**
    * C. Memindai seluruh port domain
    * D. Mengungkap password registrar

 7. Keterbatasan utama hasil `site:` adalah:
    * A. Hanya bekerja pada IP
    * **B. Hasil merupakan sampel indeks, bukan inventaris lengkap**
    * C. Selalu mengirim traffic ke target
    * D. Tidak dapat menemukan dokumen

 8. Data Shodan berusia satu tahun sebaiknya diperlakukan sebagai ...
    * A. Bukti konfigurasi produksi saat ini
    * **B. Indikasi historis yang memerlukan validasi**
    * C. Izin untuk melakukan exploitation
    * D. Data tanpa provenance

 9. Dalam SARING, huruf R menekankan:
    * A. Rate scanning
    * **B. Rekam provenance**
    * C. Reverse shell
    * D. Remediasi otomatis

10. Mengapa output LLM tidak boleh menjadi sumber fakta OSINT?
    * A. LLM tidak dapat memproses teks
    * **B. LLM dapat menghasilkan informasi yang tidak didukung sumber**
    * C. LLM selalu melakukan port scan
    * D. LLM tidak dapat membuat tabel

11. Aset yang ditemukan tetapi tidak tercantum dalam scope seharusnya ...
    * A. Langsung diuji karena sudah publik
    * **B. Dicatat minimal dan dimintakan konfirmasi tertulis**
    * C. Diabaikan tanpa catatan
    * D. Diserahkan kepada pihak ketiga

12. Tujuan utama handoff reconnaissance adalah:
    * A. Menyerahkan semua raw data tanpa prioritas
    * **B. Memberi aset kandidat, confidence, batas, dan rencana validasi**
    * C. Membuktikan seluruh aset rentan
    * D. Menggantikan tahap aktif

### Kunci Jawaban Pilihan Ganda

1. C | 2. C | 3. B | 4. B | 5. C | 6. B | 7. B | 8. B | 9. B | 10. B | 11. B | 12. B

## Pertanyaan Esai

1. Jelaskan mengapa "tidak memindai" belum cukup untuk menyatakan suatu aktivitas pasif. Gunakan tiga contoh.
2. Rancang intelligence requirements untuk reconnaissance pasif aplikasi penerimaan mahasiswa baru.
3. Jelaskan cara mengorelasikan CT, passive DNS, repository, dan scan database tanpa melakukan *confirmation bias*.
4. Susun prosedur respons ketika secret ditemukan pada histori repository publik.
5. Jelaskan bagaimana tim defender dapat menggunakan SARING untuk *continuous external attack-surface monitoring*.

## Glosarium

* **Active reconnaissance:** Pengumpulan informasi dengan interaksi langsung kepada sistem, jaringan, layanan, atau manusia target.
* **Attack surface:** Keseluruhan aset, jalur, identitas, layanan, dan hubungan yang dapat menjadi titik interaksi atau serangan.
* **Certificate Transparency (CT):** Ekosistem log publik dan dapat diaudit untuk pencatatan sertifikat TLS.
* **Confidence:** Tingkat dukungan bukti terhadap suatu klaim atau relasi.
* **Contradiction log:** Catatan sistematis tentang data yang tidak konsisten atau menentang hipotesis.
* **Entity:** Objek analisis seperti domain, IP, sertifikat, repository, dokumen, organisasi, atau peran.
* **Intelligence requirement:** Pertanyaan informasi yang jawabannya memengaruhi keputusan.
* **Observed_at:** Waktu ketika sumber mengamati data.
* **Open-Source Intelligence (OSINT):** Intelligence yang dihasilkan melalui pengumpulan, evaluasi, korelasi, dan analisis informasi yang tersedia secara terbuka.
* **Passive DNS:** Basis data historis respons DNS yang dikumpulkan dari sensor atau observasi pihak ketiga.
* **Passive reconnaissance:** Pengumpulan informasi dari sumber pihak ketiga tanpa interaksi langsung dengan sistem target.
* **Provenance:** Asal-usul artefak, termasuk sumber, query, waktu, alat, dan kondisi pengambilan.
* **RDAP:** Protokol akses data registrasi terstruktur yang menggantikan WHOIS untuk gTLD.
* **Scope status:** Status apakah entitas *in-scope*, *out-of-scope*, belum diketahui, atau menunggu persetujuan.
* **Secret:** Informasi autentikasi atau otorisasi seperti password, API key, token, atau private key.

## Referensi

[1] OffSec. "OSCP+ Body of Knowledge - Information Gathering." 2026. `https://help.offsec.com/hc/en-us/articles/38543335188756-OSCP-Body-of-knowledge`

[2] OffSec. "PEN-200: Penetration Testing with Kali Linux - Course Syllabus." 2026. `https://manage.offsec.com/app/uploads/2026/03/PEN-200_Syllabus.pdf`

[3] National Institute of Standards and Technology. "Technical Guide to Information Security Testing and Assessment." NIST SP 800-115. 2008. `https://csrc.nist.gov/pubs/sp/800/115/final`

[4] MITRE ATT&CK. "Search Open Websites/Domains, T1593." `https://attack.mitre.org/techniques/T1593/`

[5] MITRE ATT&CK. "Search Open Technical Databases, T1596." (termasuk passive DNS, WHOIS, digital certificates, dan scan databases). `https://attack.mitre.org/techniques/T1596/`

[6] OWASP Foundation. "Conduct Search Engine Discovery Reconnaissance for Information Leakage." Web Security Testing Guide. `https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage`

[7] Internet Corporation for Assigned Names and Numbers. "ICANN Update: Launching RDAP; Sunsetting WHOIS." 27 Januari 2025. `https://www.icann.org/en/announcements/details/icann-update-launching-rdap-sunsetting-whois-27-01-2025-en`

[8] Google Search Central. "How to Use the site: Search Operator." `https://developers.google.com/search/docs/monitor-debug/search-operators/all-search-site`

[9] Google Search Central. "File Types Indexable by Google." `https://developers.google.com/search/docs/crawling-indexing/indexable-file-types`

[10] Laurie, B., Messeri, E., dan Stradling, R. "Certificate Transparency Version 2.0." RFC 9162. 2021. `https://www.rfc-editor.org/info/rfc9162/`

[11] Shodan. "Search Query Fundamentals" dan "Understanding Shodan Vulnerability Assessment." `https://help.shodan.io/the-basics/search-query-fundamentals`

[12] Censys. "Censys Query Language." `https://docs.censys.com/docs/censys-query-language`

[13] GitHub Docs. "Removing Sensitive Data from a Repository" dan "Secret Scanning." `https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository`

[14] theHarvester Project. "theHarvester." `https://github.com/laramies/theHarvester`

[15] Gitleaks Project. "Find Secrets with Gitleaks." `https://github.com/gitleaks/gitleaks`

[16] Office of the United Nations High Commissioner for Human Rights dan Human Rights Center, UC Berkeley. "Berkeley Protocol on Digital Open Source Investigations." 2022. `https://www.ohchr.org/en/publications/policy-and-methodological-publications/berkeley-protocol-digital-open-source`

[17] National Institute of Standards and Technology. "Artificial Intelligence Risk Management Framework: Generative Artificial Intelligence Profile." NIST AI 600-1. 2024. `https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence`

[18] Republik Indonesia. Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi. `https://peraturan.bpk.go.id/Details/229798/uu-no-27-tahun-2022`

[19] Republik Indonesia. Undang-Undang Nomor 1 Tahun 2024 tentang Perubahan Kedua atas Undang-Undang Nomor 11 Tahun 2008 tentang Informasi dan Transaksi Elektronik. `https://peraturan.bpk.go.id/Details/274494/`

[20] Google Search Central. "Introduction to robots.txt." `https://developers.google.com/search/docs/crawling-indexing/robots/intro`

*Seluruh sumber daring diperiksa pada 31 Agustus 2026. Materi PEN-200 digunakan sebagai rujukan cakupan kompetensi; narasi, kerangka SARING, contoh, dataset fiktif, latihan, dan struktur penyajian bab ini disusun secara mandiri.*