# Sems3
# Bab 2 (Praktikum) — OSINT Pasif

Sebelum mulai, ada satu hal yang perlu diluruskan dulu: OSINT itu unik dibanding sesi praktikum lain di buku ini, karena "targetnya" nggak selalu bisa kita jalankan sendiri di Docker. Kenapa? Ya karena inti dari OSINT adalah menggali informasi yang sudah publik di internet nyata — dan itu nggak bisa disimulasikan sepenuhnya di lingkungan lab tertutup.

Makanya, praktikum bab ini kita bagi jadi dua bagian: **Bagian A** untuk latihan tools OSINT sungguhan terhadap sumber-sumber publik yang memang dirancang untuk pencarian bebas (jadi sepenuhnya legal dan aman), dan **Bagian B** untuk latihan hands-on menggali kebocoran informasi dari kode sumber/halaman web, memakai "perusahaan fiktif" yang sudah saya siapkan supaya kamu bisa praktik tanpa khawatir menyentuh data organisasi nyata.

## Bagian A — Latihan Tools OSINT Terhadap Sumber Publik

Ini bagian yang murni read-only, hanya bertanya ke database publik pihak ketiga (bukan menyerang server siapapun secara langsung), jadi aman dilakukan.

### A.1 Pilih Target yang Sah untuk Diriset

Pilih salah satu dari opsi berikut sebagai "target" latihanmu:
- Domain milikmu sendiri (kalau punya).
- Domain contoh yang secara terbuka mendorong riset publik/security research (misalnya organisasi yang mempublikasikan program bug bounty dengan lingkup riset OSINT yang jelas).

Jangan pernah pakai domain organisasi acak yang kamu pilih sembarangan tanpa tahu kebijakan mereka soal riset keamanan.

### A.2 WHOIS dan DNS Dasar

Dari Kali VM:

```bash
whois <domain-pilihanmu>
dig <domain-pilihanmu> ANY
dig <domain-pilihanmu> MX
```

Catat: siapa registrar-nya, kapan domain terdaftar, dan mail server apa yang dipakai (dari record MX) — ini nanti berguna untuk menebak pola email organisasi.

### A.3 Certificate Transparency (crt.sh)

Buka `https://crt.sh/?q=%25.<domain-pilihanmu>` di browser (tanda `%25.` adalah wildcard subdomain). Catat semua subdomain unik yang muncul di hasil.

### A.4 theHarvester

```bash
theHarvester -d <domain-pilihanmu> -b all -l 200
```

Tool ini akan mencoba mengumpulkan email dan subdomain dari berbagai sumber publik sekaligus. Simpan hasilnya:

```bash
theHarvester -d <domain-pilihanmu> -b all -l 200 -f ~/Lab-Notes/Bab-02-OSINT/theharvester-hasil
```

### A.5 Shodan (Opsional, Butuh Akun Gratis)

Daftar akun gratis di shodan.io, lalu coba pencarian sederhana seperti `org:"Nama Organisasi"` untuk melihat perangkat/server apa saja milik organisasi tersebut yang terindeks Shodan.

### A.6 Susun Peta OSINT

Pindahkan semua temuan Bagian A ke `template_osint_mindmap.md` yang sudah disediakan di paket resource. Isi minimal: daftar subdomain, teknologi yang terdeteksi (kalau ada), dan pola format email organisasi (kalau berhasil ditemukan dari theHarvester).

## Bagian B — Hands-On: Menggali Kebocoran dari Kode Sumber & Halaman Web

Nah, ini bagian yang bisa kita simulasikan penuh secara lokal. Saya sudah siapkan sebuah "perusahaan fiktif" bernama **PT Contoh Nusantara** dalam bentuk website statis sederhana, lengkap dengan beberapa "jebakan" OSINT yang sengaja ditanam untuk latihan — mirip seperti yang sering ditemukan di dunia nyata (komentar HTML yang harusnya dihapus, file backup yang lupa dihapus, dan seterusnya).

### B.1 Jalankan Target

```bash
docker compose up -d
```

Akses `http://localhost:8080` dari browser di Kali. Kamu akan melihat halaman depan perusahaan fiktif ini.

### B.2 Mulai Menggali

Beberapa hal yang perlu kamu coba (tanpa saya kasih tahu semua jawabannya — coba temukan sendiri dulu):

1. Lihat **page source** (Ctrl+U di browser) halaman utama. Baca semua komentar HTML dengan teliti.
2. Cek file `robots.txt` (`http://localhost:8080/robots.txt`) — file ini justru sering membocorkan path yang "ingin disembunyikan" dari mesin pencari, padahal manusia tetap bisa mengaksesnya langsung.
3. Cek `sitemap.xml` untuk melihat daftar halaman yang mereka daftarkan sendiri ke mesin pencari.
4. Kalau kamu menemukan path yang menarik dari `robots.txt`, coba akses langsung.
5. Perhatikan meta tag `generator` di HTML — informasi kecil ini bisa memberi petunjuk soal CMS/framework yang dipakai (dan versi berapa, kalau beruntung).

### B.3 Dokumentasikan Temuanmu

Untuk setiap "kebocoran" yang kamu temukan, catat di `01-osint-notes.md`:
- Di mana kamu menemukannya (URL/file spesifik).
- Informasi apa yang bocor.
- Kalau ini organisasi nyata, dampak apa yang mungkin terjadi (misalnya: pola email yang bocor bisa dipakai untuk phishing terarah di Bab 8 nanti).

## Checkpoint Praktikum

- [ ] Berhasil menjalankan WHOIS, dig, dan theHarvester terhadap target sah pilihanmu (Bagian A).
- [ ] Menemukan minimal 3 subdomain via crt.sh.
- [ ] Menemukan minimal 2 "kebocoran informasi" tersembunyi di website PT Contoh Nusantara (Bagian B).
- [ ] Peta OSINT (`template_osint_mindmap.md`) sudah terisi lengkap untuk kedua bagian.

## Sumber Daya Pendukung

Semua file teknis untuk praktikum ini — `docker-compose.yml` beserta konten website PT Contoh Nusantara, konfigurasi Vagrant untuk provisioning tools OSINT di Kali, script automasi, dan template mind-map — ada di paket resource yang menyertai modul ini. Cek `README.md` di dalamnya untuk instruksi lengkap.

## Vide Tutorial (step-by-step walkthrough)

https://youtu.be/L5Z1sGrUmu8


# Resource Praktikum — Bab 2: OSINT Pasif

Paket ini berisi semua file teknis untuk Modul Praktikum Bab 2, terbagi dua bagian sesuai modul:
**Bagian A** (riset terhadap sumber publik sungguhan) dan **Bagian B** (hands-on lokal dengan website fiktif "PT Contoh Nusantara").

## Isi Paket

| File/Folder | Fungsi |
|---|---|
| `www/` | Konten website fiktif PT Contoh Nusantara (target Bagian B) |
| `nginx-default.conf` | Konfigurasi nginx untuk menyajikan website tersebut |
| `docker-compose.yml` | Menjalankan target PT Contoh Nusantara di port 8080 |
| `Vagrantfile` | Provisioning VM Kali dengan tools OSINT |
| `provision.sh` | Script instalasi whois, dig, theHarvester, exiftool, Docker |
| `osint_recon.sh` | Automasi sebagian langkah OSINT Bagian A (WHOIS, DNS, theHarvester) |
| `template_osint_mindmap.md` | Template dokumentasi hasil riset OSINT |

## Setup Cepat (Jika Kali VM Sudah Ada dari Bab 1)

```bash
chmod +x provision.sh
sudo ./provision.sh
```

Lalu jalankan target Bagian B:

```bash
docker compose up -d
```

Akses `http://localhost:8080` dari browser.

## Setup dari Nol (Vagrant + VirtualBox)

```bash
vagrant up
vagrant ssh
cd /vagrant
docker compose up -d
```

## Menjalankan Bagian A (Riset Domain Sah)

```bash
chmod +x osint_recon.sh
./osint_recon.sh <domain-target-yang-sah>
```

**Ingat**: hanya jalankan ini terhadap domain yang memang sah kamu riset. Jangan pernah pakai script ini terhadap domain organisasi sembarangan tanpa izin — meski semua langkah di dalamnya bersifat pasif/read-only terhadap sumber pihak ketiga, tetap gunakan penilaian etis yang baik.

## Menjalankan Bagian B (Hands-On PT Contoh Nusantara)

Setelah container jalan, coba akses:
- `http://localhost:8080/` — halaman utama
- `http://localhost:8080/robots.txt`
- `http://localhost:8080/sitemap.xml`
- Lalu telusuri sendiri path-path yang menarik dari `robots.txt`

Jangan buka jawaban di bawah ini sebelum mencoba sendiri dulu ya:

<details>
<summary>Spoiler: daftar "kebocoran" yang sengaja ditanam (klik untuk buka)</summary>

- Komentar HTML di `index.html` berisi kredensial staging lama dan kontak staf IT.
- Meta tag `generator` membocorkan nama & versi CMS fiktif.
- `robots.txt` membocorkan path `/internal/` dan file `notes-lama.txt.bak`.
- `/internal/employees.html` berisi direktori staf lengkap dengan email.
- `notes-lama.txt.bak` berisi catatan internal termasuk nama VPN dan kontak eskalasi.
- Halaman karir membocorkan pola format email organisasi secara eksplisit.

</details>

## Menyalin Template

```bash
cp template_osint_mindmap.md ~/Lab-Notes/Bab-02-OSINT/02-peta-osint.md
```

## Catatan Keamanan & Etika Lab

- Website "PT Contoh Nusantara" adalah data fiktif 100% — nama, email, dan detail lain tidak merujuk ke entitas nyata manapun.
- Jangan expose port 8080 ke internet.
- Untuk Bagian A, selalu pastikan target yang kamu pilih memang sah untuk diriset secara OSINT.
