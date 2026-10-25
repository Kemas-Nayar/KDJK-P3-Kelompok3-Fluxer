# Fluxer

<img width="1280" height="335" alt="Fluxer_app_logo svg" src="https://github.com/user-attachments/assets/02164b98-afa3-4db8-9bdd-3d078ddef7b9" />


## Sekilas Tentang

Fluxer adalah aplikasi chat komunitas open source (lisensi AGPLv3) yang dapat di-hosting sendiri, dirancang sebagai alternatif Discord. Fitur utamanya meliputi server komunitas, kanal teks, pesan langsung, upload berkas, pencarian pesan, serta suara/video.

Pengaturan self-hosting resminya berjalan di atas Docker Compose dengan banyak layanan (sekitar 25 container menurut salah satu sumber), antara lain PostgreSQL, Valkey, NATS, Meilisearch, SeaweedFS, LiveKit, API, gateway, worker, dan Caddy sebagai proxy.

**Lingkungan**

| Komponen | Nilai |
|---|---|
| Server | VM VirtualBox (host Linux Mint), Ubuntu Server 24.04 LTS, jaringan NAT |
| Spesifikasi | 4 vCPU, 8 GB RAM, 80 GB disk |
| Akses publik | Cloudflare Tunnel (HTTPS dikelola Cloudflare) |
| Domain | `chat.kdjk-fluxer.web.id` (domain dibeli di Dewabiz, DNS dikelola Cloudflare) |
| Image | `ghcr.io/fluxerapp/fluxer-api:v1` |


## Instalasi

Prasyarat:
- Server Ubuntu Server 24.04 LTS (VPS atau VM) dengan RAM minimal 8 GB (lebih besar lebih aman).
- Domain/subdomain. Jalur VPS dengan IP publik: A record mengarah ke IP VPS, port 80 dan 443 terbuka. Jalur Cloudflare Tunnel: lihat bagian "Akses Publik via Cloudflare Tunnel".
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

Atur domain di `.env` sesuai jalur akses. Untuk VPS dengan IP publik, ikuti komentar bawaan `.env` (Caddy mengurus HTTPS sendiri lewat port 80/443). Untuk VM tanpa IP publik, gunakan Cloudflare Tunnel (bagian berikut), yang nilai `.env`-nya berbeda.

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

> Jangan unggah `.env` ke GitHub. Tambahkan `.env` ke `.gitignore`.

### Akses Publik via Cloudflare Tunnel

VM berada di belakang NAT tanpa IP publik, sehingga akses dari internet dibuat lewat Cloudflare Tunnel: `cloudflared` di dalam VM membuat koneksi keluar ke Cloudflare, dan Cloudflare meneruskan permintaan ke Caddy lokal serta menyediakan HTTPS. Tidak ada port router atau port forwarding VirtualBox yang perlu dibuka untuk jalur ini.

Prasyarat: akun Cloudflare (gratis), domain yang nameserver-nya diarahkan ke Cloudflare, dan `cloudflared` terpasang di VM (ikuti panduan resmi Cloudflare untuk Ubuntu).

**1. Tambahkan domain ke Cloudflare.** Di dashboard Cloudflare pilih *Add a site* dengan paket Free, lalu ganti nameserver domain di panel registrar (Dewabiz) ke dua nameserver yang diberikan. Tunggu status situs menjadi *Active*.

**2. Buat tunnel dan rute DNS.**

```
cloudflared tunnel login
cloudflared tunnel create fluxer
cloudflared tunnel route dns fluxer chat.kdjk-fluxer.web.id
```

Jika `route dns` gagal dengan error 1003 (record sudah ada), hapus record `chat` yang lama di *DNS → Records* pada dashboard Cloudflare, lalu ulangi.

**3. Buat `~/.cloudflared/config.yml`** (ganti `<ID-TUNNEL>` dengan ID dari langkah 2; berkas `.json` kredensial bersifat rahasia):

```yaml
tunnel: <ID-TUNNEL>
credentials-file: /home/fluxer/.cloudflared/<ID-TUNNEL>.json
ingress:
  - hostname: chat.kdjk-fluxer.web.id
    service: http://localhost:19080
  - service: http_status:404
```

**4. Sesuaikan `.env` Fluxer.** Alamat publik yang dilihat browser dibuat HTTPS, sedangkan Caddy tetap melayani HTTP biasa di port 19080 (variabel `FLUXER_HTTP_PORT` dan `FLUXER_HTTPS_PORT` tidak diubah):

```
FLUXER_DOMAIN=chat.kdjk-fluxer.web.id
FLUXER_PUBLIC_SCHEME=https
FLUXER_PUBLIC_PORT=443
FLUXER_PUBLIC_ORIGIN=https://chat.kdjk-fluxer.web.id
FLUXER_EDGE_SITE_ADDRESS=http://chat.kdjk-fluxer.web.id
FLUXER_HTTP_PORT=19080
FLUXER_HTTPS_PORT=127.0.0.1:443
```

**5. Terapkan dan jalankan tunnel.**

```
cd /opt/fluxer
sudo docker compose config --quiet
sudo docker compose up -d
sudo docker compose ps
cloudflared tunnel run fluxer      # biarkan berjalan (mis. di tmux)
```

**6. Verifikasi.**

```
curl -s -H 'Host: chat.kdjk-fluxer.web.id' http://127.0.0.1:19080/_health   # OK (langsung ke Caddy)
curl -s https://chat.kdjk-fluxer.web.id/_health                              # OK (lewat tunnel)
```

Lalu buka `https://chat.kdjk-fluxer.web.id` dari perangkat di jaringan lain (mis. HP dengan data seluler).

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

1. Halaman Setup dan Registrasi
   ![Halaman registrasi](deploy/01-registrasi.png)
   Pada instance baru, pengguna menyelesaikan setup dengan setup token lalu mendaftarkan akun pertama.

2. Halaman Utama
   ![Halaman utama](deploy/02-halaman-utama.png)
   Setelah login, pengguna melihat daftar server, kanal, dan pesan langsung.

3. Membuat Komunitas dan Kanal
   ![Membuat server](deploy/03-komunitas.png)
   Dibuat komunitas **KDJK Kelompok 3** dengan kanal `#general`, `#pengumuman`, dan `#dokumentasi`. 

4. Mengirim Pesan dan Lampiran
   ![Pesan](deploy/04-pesan.png)
   Beberapa akun demo saling bertukar pesan dan mengunggah lampiran pada kanal.

5. Pengaturan Pengguna
   ![Pengaturan](deploy/05-pengaturan.png)
   Pengguna dapat mengubah profil dan tampilan.

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
