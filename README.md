# Fluxer

<img width="1280" height="335" alt="Fluxer_app_logo svg" src="https://github.com/user-attachments/assets/02164b98-afa3-4db8-9bdd-3d078ddef7b9" />

## Sekilas Tentang

Fluxer merupakan aplikasi komunikasi real-time berbasis web yang dirancang sebagai platform komunikasi komunitas dan tim. Fluxer menyediakan konsep server, channel, pengguna, pesan, serta komunikasi suara yang memungkinkan beberapa pengguna berkomunikasi dalam satu platform.

Fluxer dikembangkan sebagai aplikasi open-source sehingga source code-nya dapat dipelajari, dikembangkan, dan dijalankan pada infrastruktur sendiri (*self-hosted*).

Secara konsep, Fluxer memiliki fungsi yang mirip dengan aplikasi komunikasi seperti Discord, tetapi memberikan lebih banyak kontrol kepada administrator terhadap aplikasi dan infrastrukturnya.

### Fitur Utama

Beberapa fungsi utama yang tersedia dalam Fluxer antara lain:

- Registrasi dan autentikasi pengguna.
- Pengelolaan profil pengguna.
- Pembuatan dan pengelolaan server.
- Pembuatan channel.
- Komunikasi pesan secara real-time.
- Pengiriman attachment/file.
- Voice channel.
- Pengelolaan anggota server.
- Role dan permission.
- Notifikasi.
- Pengelolaan komunitas.

Arsitektur aplikasi secara umum dapat digambarkan sebagai berikut:

```text
                         Client
                    Web Browser / App
                           |
                           v
                    +-------------+
                    |   Fluxer    |
                    | Application |
                    +------+------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
        Database                    Real-time
                                   Communication
             |                           |
             +-------------+-------------+
                           |
                           v
                       Storage
```

---

# Instalasi

## Prasyarat

Sebelum melakukan instalasi Fluxer, siapkan lingkungan Linux/server dan perangkat lunak yang diperlukan oleh versi Fluxer yang digunakan.

Perangkat lunak yang umum diperlukan antara lain:

- Linux
- Git
- Docker
- Docker Compose
- Database yang digunakan oleh Fluxer
- Node.js dan package manager apabila diperlukan oleh komponen frontend/backend
- Reverse proxy seperti Nginx untuk deployment production

Periksa instalasi Git:

```bash
git --version
```

Periksa Docker:

```bash
docker --version
```

Periksa Docker Compose:

```bash
docker compose version
```

Jika menggunakan Node.js:

```bash
node --version
npm --version
```

> **Catatan:** versi dependency harus disesuaikan dengan dokumentasi dan konfigurasi pada commit/release Fluxer yang digunakan. Jangan menggunakan versi dependency secara sembarang apabila repository menetapkan versi tertentu.

## Clone Repository

Clone repository Fluxer menggunakan Git:

```bash
git clone <URL-REPOSITORY-FLUXER>
```

Masuk ke direktori project:

```bash
cd fluxer
```

Periksa isi repository:

```bash
ls
```

## Instalasi Dependency

Jika project menyediakan file `package.json`, dependency dapat dipasang menggunakan package manager yang sesuai.

Dengan npm:

```bash
npm install
```

atau apabila repository menyediakan lockfile:

```bash
npm ci
```

Jika project menggunakan package manager lain, gunakan package manager yang ditentukan oleh repository.

## Konfigurasi Environment

Salin file environment contoh apabila tersedia:

```bash
cp .env.example .env
```

Kemudian edit:

```bash
nano .env
```

Contoh parameter yang dapat ditemukan pada aplikasi web:

```env
APP_URL=http://localhost
DATABASE_URL=...
REDIS_URL=...
```

Nilai aktual harus mengikuti `.env.example` dan dokumentasi Fluxer.

## Menjalankan dengan Docker

Apabila repository menyediakan `docker-compose.yml` atau `compose.yml`, jalankan:

```bash
docker compose up -d
```

Periksa status service:

```bash
docker compose ps
```

Periksa log:

```bash
docker compose logs
```

Untuk mengikuti log secara real-time:

```bash
docker compose logs -f
```

Setelah seluruh service berhasil berjalan, buka alamat aplikasi melalui browser.

Contoh:

```text
http://localhost
```

atau port yang ditentukan oleh konfigurasi Fluxer.

---

# Konfigurasi

Konfigurasi tambahan dapat dilakukan untuk meningkatkan fungsi, keamanan, dan performa aplikasi.

## Environment Variable

File `.env` digunakan untuk menyimpan konfigurasi yang bersifat spesifik terhadap deployment.

Contoh:

```env
APP_URL=https://fluxer.example.com
DATABASE_URL=...
REDIS_URL=...
```

Jangan memasukkan secret atau password ke dalam repository Git.

Tambahkan `.env` ke `.gitignore` jika belum tersedia:

```text
.env
.env.local
```

## Database

Fluxer memerlukan database untuk menyimpan data aplikasi seperti:

- akun pengguna;
- server;
- channel;
- pesan;
- role;
- permission;
- konfigurasi aplikasi; dan
- data lain yang diperlukan oleh aplikasi.

Database sebaiknya hanya dapat diakses oleh service yang memang membutuhkannya.

Pada deployment production, hindari mengekspos port database secara langsung ke internet.

## Reverse Proxy

Untuk deployment production, Fluxer dapat ditempatkan di belakang reverse proxy seperti Nginx.

Arsitektur:

```text
Internet
   |
   | HTTPS :443
   v
+-------+
| Nginx |
+---+---+
    |
    | HTTP / WebSocket
    v
+-----------+
|  Fluxer   |
|  Server   |
+-----+-----+
      |
      +--------> Database
      |
      +--------> Cache / Redis
      |
      +--------> File Storage
```

Reverse proxy dapat digunakan untuk:

- HTTPS/TLS.
- Domain.
- Routing request.
- WebSocket.
- Compression.
- Static file caching.
- Rate limiting.
- Menyembunyikan port internal aplikasi.

## HTTPS

Untuk server production, gunakan HTTPS.

Contoh domain:

```text
https://fluxer.example.com
```

Sertifikat TLS dapat diperoleh menggunakan Let's Encrypt atau certificate authority lainnya.

## Batas Upload

Jika instance Fluxer digunakan untuk mengirim attachment, batas ukuran file perlu disesuaikan dengan kemampuan server.

Perhatikan bahwa batas upload dapat ditentukan oleh lebih dari satu komponen:

```text
Browser
   |
   v
Nginx / Reverse Proxy
   |
   v
Fluxer
   |
   v
Storage
```

Pastikan seluruh komponen menggunakan batas yang konsisten.

## Cache

Jika Fluxer menggunakan Redis atau mekanisme cache lainnya, pastikan service cache berjalan dan dapat diakses oleh aplikasi.

Cache dapat membantu mengurangi beban database dan meningkatkan respons aplikasi.

## Plugin dan Integrasi

Fungsi tambahan dapat diberikan melalui integrasi yang didukung oleh Fluxer atau komponen eksternal.

Contoh integrasi yang dapat dipertimbangkan:

- OAuth/social login.
- Bot.
- Webhook.
- Integrasi layanan eksternal.
- Markdown.
- Integrasi penyimpanan file.
- Monitoring.
- Logging.

> Tidak semua integrasi di atas merupakan plugin bawaan Fluxer. Implementasi harus mengikuti fitur yang benar-benar tersedia pada versi Fluxer yang digunakan.

---

# Maintenance

Maintenance diperlukan agar instance Fluxer tetap stabil, aman, dan dapat digunakan dalam jangka panjang.

## Backup Database

Database harus di-backup secara berkala.

Contoh jadwal menggunakan `cron`:

```cron
0 2 * * 0 /path/to/backup-fluxer.sh
```

Jadwal tersebut menjalankan backup setiap hari Minggu pukul 02.00.

Contoh struktur backup:

```text
backup/
├── database/
│   ├── fluxer-2026-10-04.sql
│   ├── fluxer-2026-10-11.sql
│   └── ...
└── files/
    └── ...
```

Backup sebaiknya disimpan pada lokasi yang berbeda dari server utama.

## Backup File

Selain database, file attachment atau data persistent lainnya juga perlu di-backup.

Contoh:

```bash
tar -czf fluxer-files-$(date +%F).tar.gz /path/to/fluxer-data
```

## Update Aplikasi

Sebelum melakukan update:

1. Backup database.
2. Backup file persistent.
3. Periksa release atau perubahan dependency.
4. Perbarui source code.
5. Jalankan migration jika diperlukan.
6. Restart service.
7. Periksa log.

Contoh:

```bash
git pull
docker compose pull
docker compose up -d
```

## Pemeriksaan Service

Periksa container:

```bash
docker compose ps
```

Periksa log:

```bash
docker compose logs --tail=100
```

Periksa penggunaan resource:

```bash
docker stats
```

## Membersihkan Resource Docker

Resource Docker yang tidak digunakan dapat diperiksa dengan:

```bash
docker system df
```

Resource yang tidak diperlukan dapat dibersihkan dengan hati-hati:

```bash
docker system prune
```

> Jangan menjalankan perintah pembersihan secara sembarangan pada production karena dapat menghapus resource yang masih diperlukan.

## Maintenance Berkala

Contoh jadwal maintenance:

| Aktivitas | Frekuensi |
|---|---|
| Backup database | Harian |
| Backup file | Harian |
| Pemeriksaan log | Harian |
| Pemeriksaan disk | Mingguan |
| Update aplikasi | Sesuai release |
| Pengujian restore backup | Bulanan |
| Audit keamanan | Berkala |

---

# Otomatisasi

Proses instalasi dan maintenance dapat dibuat otomatis menggunakan shell script.

Contoh struktur:

```text
fluxer/
├── setup.sh
├── backup.sh
├── maintenance.sh
└── README.md
```

## Setup Script

Contoh `setup.sh`:

```bash
#!/bin/bash

set -e

echo "=== Fluxer Setup ==="

if ! command -v docker >/dev/null 2>&1; then
    echo "Docker belum terinstall."
    exit 1
fi

if ! docker compose version >/dev/null 2>&1; then
    echo "Docker Compose belum tersedia."
    exit 1
fi

echo "Pulling images..."
docker compose pull

echo "Starting services..."
docker compose up -d

echo "Checking services..."
docker compose ps

echo "Fluxer setup completed."
```

Jadikan executable:

```bash
chmod +x setup.sh
```

Jalankan:

```bash
./setup.sh
```

## Backup Script

Contoh `backup.sh`:

```bash
#!/bin/bash

set -e

BACKUP_DIR="$HOME/fluxer-backup"
DATE=$(date +%Y-%m-%d_%H-%M-%S)

mkdir -p "$BACKUP_DIR"

echo "Starting backup: $DATE"

# Tambahkan perintah backup database
# sesuai database yang digunakan oleh deployment Fluxer.

echo "Backup completed."
```

Jadikan executable:

```bash
chmod +x backup.sh
```

## Maintenance Script

Contoh `maintenance.sh`:

```bash
#!/bin/bash

set -e

echo "Starting Fluxer maintenance..."

docker compose pull
docker compose up -d
docker compose ps

echo "Maintenance completed."
```

Jadikan executable:

```bash
chmod +x maintenance.sh
```

## Cron

Script maintenance dapat dijalankan otomatis menggunakan cron.

Edit crontab:

```bash
crontab -e
```

Contoh:

```cron
# Backup setiap hari pukul 02.00
0 2 * * * /path/to/backup.sh

# Maintenance setiap Minggu pukul 03.00
0 3 * * 0 /path/to/maintenance.sh
```

---

# Cara Pemakaian

## Membuka Aplikasi

Setelah Fluxer berhasil dijalankan, buka browser dan akses alamat instance.

Contoh:

```text
http://localhost
```

Untuk deployment production:

```text
https://fluxer.example.com
```

## Registrasi

Pengguna dapat membuat akun melalui halaman registrasi.

Contoh data dummy:

```text
Username : Kemas
Email    : kemas@example.com
Password : ********
```

Setelah registrasi, pengguna dapat melakukan login.

## Membuat Server

Setelah login, pengguna dapat membuat server baru.

Contoh:

```text
Nama Server:
IPB Computer Science
```

Server dapat digunakan untuk mengelompokkan pengguna berdasarkan komunitas atau tujuan tertentu.

## Membuat Channel

Contoh struktur server:

```text
IPB Computer Science
│
├── TEXT
│   ├── general
│   ├── announcements
│   ├── programming
│   ├── operating-systems
│   └── networking
│
└── VOICE
    ├── General
    └── Study Room
```

## Komunikasi Teks

Pengguna dapat mengirim pesan pada channel.

Contoh:

```text
# programming

Kemas:
Ada yang sudah mengerjakan tugas Operating System?

Andi:
Sudah. Bagian Peterson Algorithm sudah selesai.

Siti:
Aku masih mengerjakan bagian semaphore.
```

## Attachment

Pengguna dapat mengirim file apabila fitur attachment tersedia dan ukuran file memenuhi batas yang dikonfigurasi.

Contoh:

```text
Kemas mengirim:
Tugas_OS.pdf
```

## Voice Channel

Pengguna dapat masuk ke voice channel untuk berkomunikasi menggunakan suara.

Contoh:

```text
Study Room

Members:
- Kemas
- Andi
- Siti
```

## Role dan Permission

Administrator dapat mengatur role pengguna.

Contoh:

```text
Administrator
├── Manage Server
├── Manage Channels
├── Manage Roles
└── Manage Members

Moderator
├── Manage Messages
└── Manage Members

Member
├── Send Messages
└── Join Voice
```

Role memungkinkan administrator membatasi operasi yang dapat dilakukan pengguna.

## Contoh Data Demonstrasi

Gunakan data yang realistis ketika melakukan demonstrasi.

```text
Server:
IPB Computer Science

Members:
- Kemas
- Andi
- Budi
- Siti

Text Channels:
# general
# programming
# operating-systems
# networking

Voice Channels:
Study Room
General
```

## Screenshot

Screenshot aplikasi dapat ditempatkan pada direktori:

```text
docs/
└── screenshots/
    ├── login.png
    ├── register.png
    ├── server.png
    ├── chat.png
    ├── members.png
    └── voice.png
```

Contoh penggunaan:

```markdown
### Halaman Login

![Halaman Login](docs/screenshots/login.png)

### Server

![Halaman Server](docs/screenshots/server.png)

### Chat

![Channel Chat](docs/screenshots/chat.png)

### Voice Channel

![Voice Channel](docs/screenshots/voice.png)
```

Screenshot sebaiknya menunjukkan aplikasi yang sudah berisi data dummy sehingga fitur dapat terlihat dengan jelas.

---

# Pembahasan

## Kelebihan

### 1. Open Source

Fluxer dikembangkan sebagai proyek open-source sehingga source code dapat dipelajari dan dikembangkan.

Hal ini memberikan kesempatan bagi developer untuk memahami bagaimana aplikasi komunikasi real-time dibangun.

### 2. Self-hosted

Aplikasi dapat dijalankan pada server sendiri.

Keuntungan dari pendekatan ini adalah administrator memiliki kontrol lebih besar terhadap:

- server;
- database;
- konfigurasi;
- storage;
- network;
- backup; dan
- deployment.

### 3. Komunikasi Real-time

Aplikasi komunikasi membutuhkan kemampuan untuk mengirim dan menerima data secara cepat.

Fluxer menyediakan komunikasi real-time sehingga pengguna dapat bertukar pesan tanpa harus melakukan refresh halaman secara manual.

### 4. Server dan Channel

Penggunaan server dan channel membuat komunikasi dapat dikelompokkan berdasarkan komunitas, organisasi, mata kuliah, proyek, atau topik tertentu.

### 5. Role dan Permission

Role dan permission memberikan mekanisme untuk membedakan hak akses pengguna.

Contohnya administrator memiliki hak yang lebih tinggi dibandingkan member biasa.

### 6. Dapat Dikembangkan

Karena source code tersedia, developer dapat mempelajari dan memodifikasi aplikasi sesuai kebutuhan.

---

## Kekurangan

### 1. Deployment Lebih Kompleks

Self-hosting membuat administrator harus memahami berbagai komponen seperti:

- Linux;
- Docker;
- database;
- networking;
- reverse proxy;
- HTTPS;
- storage; dan
- monitoring.

Hal ini membuat instalasi lebih kompleks dibandingkan menggunakan layanan hosted.

### 2. Maintenance Menjadi Tanggung Jawab Administrator

Administrator harus melakukan:

- backup;
- update;
- monitoring;
- security patch;
- troubleshooting; dan
- pengelolaan server.

### 3. Membutuhkan Resource

Aplikasi komunikasi real-time membutuhkan resource server, terutama ketika jumlah pengguna meningkat.

Resource yang perlu diperhatikan antara lain:

- CPU;
- RAM;
- storage;
- bandwidth; dan
- database storage.

### 4. Skalabilitas

Ketika jumlah pengguna meningkat, konfigurasi sederhana mungkin tidak lagi mencukupi.

Administrator dapat membutuhkan:

- load balancer;
- beberapa application server;
- database optimization;
- caching;
- object storage; dan
- monitoring yang lebih baik.

---

# Perbandingan dengan Aplikasi Sejenis

Fluxer dapat dibandingkan dengan beberapa platform komunikasi seperti Discord dan Mattermost.

| Fitur | Fluxer | Discord | Mattermost |
|---|---|---|---|
| Open source | Ya | Tidak | Ya |
| Self-hosted | Ya | Tidak | Ya |
| Text chat | Ya | Ya | Ya |
| Voice communication | Ya | Ya | Ya |
| Server / workspace | Ya | Ya | Ya |
| Channel | Ya | Ya | Ya |
| Role / permission | Ya | Ya | Ya |
| Custom deployment | Ya | Terbatas | Ya |
| Kontrol server | Tinggi | Rendah | Tinggi |
| Cocok untuk komunitas | Ya | Sangat cocok | Ya |
| Cocok untuk organisasi | Ya | Ya | Sangat cocok |

### Fluxer vs Discord

Discord merupakan platform komunikasi yang sangat populer dan memiliki ekosistem besar. Pengguna dapat langsung membuat akun dan menggunakan layanan tanpa harus mengelola server sendiri.

Fluxer memiliki pendekatan yang lebih menarik untuk pengguna yang menginginkan kontrol terhadap infrastruktur dan source code karena dapat digunakan sebagai platform self-hosted.

### Fluxer vs Mattermost

Mattermost lebih berfokus pada komunikasi organisasi dan kolaborasi tim. Fluxer memiliki pendekatan yang lebih dekat dengan platform komunitas dan komunikasi real-time seperti Discord.

Keduanya memiliki keuntungan dari sisi self-hosting dan kontrol terhadap infrastruktur.

---

# Kesimpulan

Fluxer merupakan aplikasi komunikasi real-time yang dapat digunakan untuk membangun platform komunikasi komunitas maupun tim.

Fitur server, channel, pesan, voice, role, dan permission membuat Fluxer dapat digunakan untuk berbagai kebutuhan komunikasi.

Dari sisi pembelajaran, Fluxer juga menarik karena mencakup berbagai aspek ilmu komputer, seperti:

- Web application.
- Client-server architecture.
- Database.
- Networking.
- Real-time communication.
- Authentication.
- Authorization.
- WebSocket.
- File storage.
- Deployment.
- Containerization.
- Server administration.

Penggunaan Fluxer secara self-hosted juga memberikan kesempatan untuk mempelajari proses deployment aplikasi web secara langsung, mulai dari konfigurasi server hingga maintenance dan backup.

---

# Referensi

1. Fluxer GitHub Repository  
   https://github.com/fluxerapp/fluxer

2. Fluxer Website  
   https://fluxer.app/

3. Docker Documentation  
   https://docs.docker.com/

4. Docker Compose Documentation  
   https://docs.docker.com/compose/

5. Nginx Documentation  
   https://nginx.org/en/docs/

6. Git Documentation  
   https://git-scm.com/doc

7. Discord  
   https://discord.com/

8. Mattermost  
   https://mattermost.com/

---

## Lisensi

Lisensi aplikasi mengikuti lisensi yang tercantum pada repository Fluxer.

Periksa file `LICENSE` pada repository untuk mengetahui ketentuan penggunaan, modifikasi, dan distribusi source code.
