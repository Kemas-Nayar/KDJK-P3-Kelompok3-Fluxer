# Fluxer

<img width="1280" height="335" alt="Fluxer_app_logo svg" src="https://github.com/user-attachments/assets/02164b98-afa3-4db8-9bdd-3d078ddef7b9" />


## Sekilas Tentang

Fluxer adalah aplikasi chat komunitas open source (lisensi AGPLv3) yang dapat di-hosting sendiri, dirancang sebagai alternatif Discord. Fitur utamanya meliputi server komunitas, kanal teks, pesan langsung, upload berkas, pencarian pesan, serta suara/video.

Pengaturan self-hosting resminya berjalan di atas Docker Compose dengan banyak layanan (sekitar 25 container menurut salah satu sumber), antara lain PostgreSQL, Valkey, NATS, Meilisearch, SeaweedFS, LiveKit, API, gateway, worker, dan Caddy sebagai proxy.

## Instalasi

Prasyarat:
- VPS Ubuntu Server 24.04 LTS dengan RAM minimal 8 GB (lebih besar lebih aman) dan IP publik.
- Domain/subdomain dengan A record yang mengarah ke IP VPS.
- Port 80 dan 443 terbuka di firewall.
- Docker Engine dan plugin Docker Compose (ikuti panduan resmi: https://docs.docker.com/engine/install/ubuntu/).

Verifikasi Docker:

```
sudo systemctl enable --now docker
docker --version
docker compose version
sudo docker run --rm hello-world
```

Unduh berkas deployment resmi:

```
sudo mkdir -p /opt/fluxer
sudo chown "$USER":"$(id -gn)" /opt/fluxer
cd /opt/fluxer

BASE="https://raw.githubusercontent.com/fluxerapp/fluxer/main/deploy/self-hosting"
curl -fLO "$BASE/docker-compose.yml"
curl -fLO "$BASE/Caddyfile"
curl -fsSL "$BASE/.env.example" -o .env
chmod 600 .env
```

Atur domain di `.env`. Nama variabel dan nilai untuk domain publik dengan HTTPS dicocokkan dengan komentar di `.env.example` versi yang diunduh. <img width="1172" height="983" alt="image" src="https://github.com/user-attachments/assets/4bbe9d8e-1e47-4057-a903-fb0fa168c5ee" />

Ganti semua nilai `CHANGE_ME` dengan secret acak:

```
grep -n 'CHANGE_ME' .env
openssl rand -hex 32     # ulangi untuk tiap variabel, lalu isi ke .env
```

Pada percobaan ini tersisa tiga: `POSTGRES_PASSWORD`, `MEILI_MASTER_KEY`, dan `LIVEKIT_API_SECRET`.

Buat setup token agar instance baru dapat didaftarkan:

```
TOKEN=$(openssl rand -hex 32)
sed -i '/^#FLUXER_SETUP_TOKEN=/d; /^FLUXER_SETUP_TOKEN=/d' .env
printf '\nFLUXER_SETUP_TOKEN=%s\n' "$TOKEN" >> .env
```

Validasi dan jalankan:

```
sudo docker compose config --quiet
sudo docker compose pull
sudo docker compose up -d
sudo docker compose ps
```

Kemudian kunjungi `https://<domain-anda>`, masukkan setup token, dan buat akun.

### Kendala yang ditemui (uji awal di VM lokal)

| Gejala | Penyebab | Solusi |
|---|---|---|
| Web tidak terbuka dari host, padahal `curl` di dalam VM berstatus 200 | VirtualBox mode NAT: host tidak bisa menjangkau IP VM | Tambahkan port forwarding (host 19080 ke VM 19080) |
| Akses lewat IP/`127.0.0.1` menghasilkan halaman kosong | Caddy memilih respons berdasarkan hostname | Akses lewat nama domain yang dikonfigurasi |
| Daftar akun: "Verification required. Try again." | Respons asli API: `SETUP_TOKEN_REQUIRED`, instance belum di-setup | Isi `FLUXER_SETUP_TOKEN` di `.env`, restart `api` |
| Console: `crypto.subtle is undefined` / widget ALTCHA tidak muncul di Android | ALTCHA butuh Web Crypto yang hanya tersedia di *secure context* (HTTPS atau localhost) | Gunakan HTTPS (domain publik di VPS) |


## Konfigurasi (opsional)

Batas memori bawaan Fluxer ditujukan untuk host 16 GB. Pada RAM 8 GB, nilai berikut dipakai sebagai titik awal (tambahkan di bagian bawah `.env`, lalu pantau dengan `docker stats --no-stream`):

```
FLUXER_POSTGRES_MEMORY_LIMIT=1gb
FLUXER_MEILISEARCH_MEMORY_LIMIT=768mb
FLUXER_MEILISEARCH_MAX_INDEXING_MEMORY=128mb
FLUXER_MEILISEARCH_MAX_INDEXING_THREADS=1
FLUXER_SEAWEEDFS_MEMORY_LIMIT=768mb
FLUXER_API_MEMORY_LIMIT=1280mb
FLUXER_API_NODE_HEAP_MB=768
FLUXER_WORKER_MEMORY_LIMIT=1280mb
FLUXER_WORKER_NODE_HEAP_MB=768
FLUXER_GATEWAY_MEMORY_LIMIT=512mb
FLUXER_LIVEKIT_MEMORY_LIMIT=384mb
```

Daftar variabel lengkap ada di `.env.example`. Nilai ini eksperimen, bukan konfigurasi resmi yang dijamin stabil.


## Cara Pemakaian

Hasil pengujian (isi sesuai kenyataan):

| Pengujian | Hasil |
|---|---|
| Semua layanan berjalan | <img width="1915" height="703" alt="image" src="https://github.com/user-attachments/assets/ae5e5983-ae88-4504-a325-7192e0b3e091" /> |
| Web dapat dibuka dari internet | <img width="1280" height="1498" alt="Screenshot_2026-10-25-13-14-47-340_com android chrome-edit" src="https://github.com/user-attachments/assets/701f75d5-9f95-4b79-9e27-524e464078e0" /> | 
| Setup dan pembuatan akun | <img width="1280" height="2772" alt="Screenshot_2026-10-25-13-16-25-168_com android chrome" src="https://github.com/user-attachments/assets/b0a934a9-99cb-48de-9325-8e8dae561ed8" /> |
| Kirim dan terima pesan | <img width="1600" height="873" alt="image" src="https://github.com/user-attachments/assets/41aaacc7-7b88-4723-b157-f10c830533ed" /> |
| Upload lampiran | <img width="956" height="409" alt="image" src="https://github.com/user-attachments/assets/6068c091-2608-4f18-ba46-65f1fc034596" /> |


## Pembahasan

### Kelebihan Fluxer

1. **Open source dan self-hosted**: kode terbuka (AGPLv3) dan data tetap di server sendiri.
2. **Pengalaman mirip Discord**: server, kanal, pesan langsung, upload, dan pencarian sehingga mudah dipakai pengguna baru.
3. **Deployment resmi dikemas dalam Docker Compose**: instalasi dapat diulang dengan beberapa perintah.
4. **ALTCHA bawaan**: perlindungan pendaftaran tanpa layanan CAPTCHA eksternal.

### Kekurangan Fluxer

1. **Berat sumber daya**: banyak layanan sekaligus. Pada RAM 8 GB perlu tuning manual karena batas bawaan ditujukan untuk 16 GB.
2. **Konfigurasi rumit**: banyak secret wajib, routing bergantung hostname, dan perlu setup token.
3. **Pesan error kurang jelas**: "Verification required" muncul padahal penyebab aslinya `SETUP_TOKEN_REQUIRED`.
4. **Masih berkembang**: pengembang menyebut membuat self-hosting andal sebagai prioritas, sehingga kemungkinan masih ada kekurangan.
5. **Butuh HTTPS**: ALTCHA tidak berjalan di HTTP non-localhost, sehingga tidak cocok untuk akses HTTP biasa dari perangkat lain.

### Perbandingan Fluxer dengan Aplikasi Sejenis

*(Poin pada bagian ini berdasarkan telaah dokumentasi dan artikel, bukan pengujian langsung. **[ISI]** verifikasi ulang sebelum dikumpulkan.)*

#### 1. Fluxer vs Discord

- **Kelebihan Fluxer:**
  * Open source dan dapat di-hosting sendiri, data di server sendiri.
  * Tidak bergantung pada satu penyedia layanan.
- **Kelebihan Discord:**
  * Komunitas dan ekosistem (bot, integrasi) jauh lebih besar.
  * Matang dan stabil, tidak perlu mengelola server sendiri.

#### 2. Fluxer vs Revolt

- **Kelebihan Fluxer:**
  * Berusaha menyamai pengalaman Discord selengkap mungkin. <img width="1920" height="1040" alt="image" src="https://github.com/user-attachments/assets/422c7624-2a58-4c7c-a038-a0faec47b78d" />
- **Kelebihan Revolt:**
  * Dianggap lebih stabil saat ini oleh salah satu ulasan, dan sama-sama dapat di-hosting sendiri.

#### 3. Fluxer vs Matrix/Element

- **Kelebihan Fluxer:**
  * Antarmuka dan alur penggunaan yang familiar bagi pengguna Discord.
- **Kelebihan Matrix/Element:**
  * Protokol terbuka dengan federasi antar server dan banyak pilihan aplikasi klien.
  * Mendukung enkripsi end-to-end.

### Kesimpulan

*(Draf, sesuaikan setelah pengujian selesai.)* Fluxer cocok bagi komunitas yang ingin alternatif Discord dengan kontrol data penuh dan pengalaman penggunaan yang familiar. Namun, kebutuhan sumber daya yang besar, konfigurasi yang rumit, dan statusnya yang masih berkembang membuatnya kurang ideal untuk server kecil berspesifikasi rendah. Aplikasi seperti Discord (kemudahan dan ekosistem), Revolt (stabilitas), atau Matrix/Element (federasi dan enkripsi) bisa lebih tepat sesuai kebutuhan.


## Referensi

- Fluxer, repositori dan berkas deployment: https://github.com/fluxerapp/fluxer (`deploy/self-hosting`)
- Fluxer, situs dan dokumentasi self-hosting: https://fluxer.app/ dan https://docs.fluxer.app/
- Blog pengembang Fluxer, "How I built Fluxer, a Discord-like chat app": https://canary.fluxer.app/blog/how-i-built-fluxer-a-discord-like-chat-app
- Ulasan perbandingan dengan alternatif Discord: https://www.sinologic.net/?p=21651
- Docker Engine di Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- ALTCHA: https://altcha.org
- Revolt dan Matrix/Element: https://github.com/stoatchat/self-hosted & https://matrix.org/docs/chat_basics/matrix-for-im/
