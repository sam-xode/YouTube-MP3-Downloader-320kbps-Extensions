# YouTube-MP3-Downloader-320kbps-Extensions
Chrome extension for DJs to download high-quality songs without the hassle.

<div align="center">

# 🎧 YouTube MP3 Downloader — Glass

**Chrome extension + local backend untuk mengunduh audio YouTube (single video, banyak video, atau playlist penuh) ke MP3 320kbps CBR — dengan tampilan liquid glass modern.**

![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-informational)
![Chrome Extension](https://img.shields.io/badge/Chrome-Manifest%20V3-yellow?logo=googlechrome&logoColor=white)
![Python](https://img.shields.io/badge/backend-Python%203.8%2B-3776AB?logo=python&logoColor=white)
![Powered by yt--dlp](https://img.shields.io/badge/powered%20by-yt--dlp-red)
![License](https://img.shields.io/badge/license-MIT-green)

</div>

---

## Daftar Isi

- [Ringkasan](#ringkasan)
- [Fitur](#fitur)
- [Arsitektur](#arsitektur)
- [Prasyarat](#prasyarat)
- [Instalasi](#instalasi)
  - [1. Jalankan backend](#1-jalankan-backend)
  - [2. Load extension di Chrome](#2-load-extension-di-chrome)
- [Setup Otomatis (Zero-Config)](#setup-otomatis-zero-config)
- [Cara Pakai](#cara-pakai)
- [Konfigurasi](#konfigurasi)
- [Auto-Start saat Boot/Login](#auto-start-saat-bootlogin)
- [Struktur Proyek](#struktur-proyek)
- [Referensi API Backend](#referensi-api-backend)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Keamanan & Privasi](#keamanan--privasi)
- [Disclaimer](#disclaimer)
- [Kontribusi](#kontribusi)
- [Lisensi](#lisensi)

---

## Ringkasan

**YouTube MP3 Downloader — Glass** adalah tool unduh audio YouTube yang terdiri
dari dua bagian yang bekerja sama:

| Bagian | Peran |
|---|---|
| 🧩 **Chrome Extension** (`extension/`) | Antarmuka yang kamu pakai — popup dengan tema *liquid glass*, tempat menempel link dan melihat progres unduhan secara real-time. |
| ⚙️ **Backend lokal** (`backend/`) | Server Python kecil yang berjalan di `127.0.0.1:8766`, menjalankan proses berat: `yt-dlp` untuk ekstraksi audio dan `FFmpeg` untuk encode ke MP3 320kbps CBR. |

Kedua bagian ini berkomunikasi lewat HTTP di localhost — **tidak ada data yang
dikirim ke server pihak ketiga mana pun.** Semua proses unduh, konversi, dan
penyimpanan file terjadi 100% di komputermu sendiri.

## Fitur

- 🎵 **Single** — unduh satu video sekaligus (link `watch` yang membawa
  parameter `list=` dari Mix/Up Next otomatis tetap diperlakukan sebagai video
  tunggal, bukan playlist).
- 📚 **Multi** — tempel banyak link sekaligus (dipisah baris/koma/spasi),
  daftar link bisa diedit, dihapus per-item, dan tetap tersimpan walau
  extension ditutup.
- 📀 **Playlist** — unduh seluruh playlist ke sub-folder otomatis sesuai judul
  playlist, lengkap dengan laporan video yang gagal/di-skip.
- 🗂️ **Library** — lihat semua musik yang sudah tersimpan, lengkap jumlah lagu
  per folder dan ukuran file.
- ⚡ **Auto-setup Python & FFmpeg** — laptop baru tanpa Python/FFmpeg sama
  sekali pun bisa langsung jalan, keduanya otomatis terpasang **tanpa perlu
  admin/sudo** ([lihat detail](#setup-otomatis-zero-config)).
- 🚀 **Auto-start saat boot/login** — backend otomatis menyala di background
  setiap laptop hidup, tidak perlu dijalankan manual tiap kali.
- 🔁 **Progres tidak pernah hilang** — tutup popup kapan saja, unduhan tetap
  jalan di background; buka lagi popup-nya dan progres lanjut ditampilkan.
- 🚫 **Deteksi duplikat otomatis** — video/playlist yang sudah pernah diunduh
  tidak akan diunduh ulang, termasuk lintas beberapa **Folder Bank Musik**
  yang hanya dicek isinya (tidak pernah ditulis).
- ⏱️ **Filter durasi minimum** — lewati otomatis YouTube Shorts/cuplikan
  pendek (default: video < 30 detik dilewati, bisa diatur).
- 🔔 **Notifikasi in-page** — notifikasi melayang muncul di halaman web yang
  sedang dibuka begitu unduhan selesai, tanpa perlu buka popup.
- 🍪 **Dukungan `cookies.txt`** — untuk video yang butuh login/dibatasi umur.
- 🎨 **UI liquid glass** — tema kaca modern ala iOS, ringan dan responsif.

## Arsitektur

```
┌─────────────────────────┐        HTTP (localhost)        ┌──────────────────────────┐
│   Chrome Extension       │ ──────────────────────────────▶│   Backend Server          │
│   (popup.html/css/js)    │ ◀──────────────────────────────│   Flask · 127.0.0.1:8766  │
│                          │      job status / progress      │                          │
│  • Kirim URL             │                                 │  • yt-dlp (ekstraksi)     │
│  • Tampilkan progres     │                                 │  • FFmpeg (encode MP3)    │
│  • Kelola pengaturan     │                                 │  • Deteksi duplikat       │
└─────────────────────────┘                                 │  • Kelola job & queue     │
                                                              └──────────────────────────┘
                                                                          │
                                                                          ▼
                                                              📁 Folder musik di komputermu
```

Browser extension **tidak bisa** menjalankan proses seperti FFmpeg/yt-dlp atau
menulis file langsung ke sistem — ini batasan keamanan bawaan semua browser,
bukan cuma Chrome. Karena itu logika unduh dijalankan oleh backend Python
lokal, sementara extension hanya bertugas sebagai antarmuka.

## Prasyarat

> [!NOTE]
> Kamu **tidak perlu** menyiapkan apa pun di bawah ini secara manual — lihat
> [Setup Otomatis](#setup-otomatis-zero-config). Bagian ini murni informasi
> tentang apa yang dibutuhkan di baliknya.

| Kebutuhan | Keterangan |
|---|---|
| **Python 3.8+** | Menjalankan backend. Auto-terpasang kalau belum ada. |
| **FFmpeg** (dengan `libmp3lame`) | Encode audio ke MP3 320kbps. Auto-terpasang kalau belum ada. |
| **Google Chrome** / browser berbasis Chromium | Untuk memuat extension (Manifest V3). |
| Koneksi internet | Untuk mengunduh video, dan sekali saja di percobaan pertama untuk auto-setup. |

## Instalasi

### 1. Jalankan backend

```bash
# Clone atau download repo ini, lalu masuk ke folder backend/
cd backend
```

**Windows:**
```bat
start.bat
```

**macOS / Linux:**
```bash
chmod +x start.sh
./start.sh
```

Biarkan jendela terminal ini tetap terbuka — ini server lokalmu. Tunggu sampai
muncul log:

```
Listening on: http://127.0.0.1:8766
```

### 2. Load extension di Chrome

1. Buka `chrome://extensions`
2. Aktifkan **Developer mode** (kanan atas)
3. Klik **Load unpacked**
4. Pilih folder `extension/`
5. Ikon extension akan muncul di toolbar Chrome — siap dipakai 🎉

## Setup Otomatis (Zero-Config)

Skrip `start.bat` / `start.sh` di atas akan **otomatis memasang Python dan
FFmpeg** kalau belum ada di komputer — cocok untuk laptop baru yang benar-benar
kosong, **tanpa perlu hak admin/sudo sama sekali**:

<table>
<tr><th>Platform</th><th>Python</th><th>FFmpeg</th></tr>
<tr>
<td><b>Windows</b></td>
<td>

`winget` (kalau tersedia), atau installer resmi python.org secara **silent
per-user** (`InstallAllUsers=0`) — tidak memicu prompt UAC.

</td>
<td>

Build portable dari [BtbN/FFmpeg-Builds](https://github.com/BtbN/FFmpeg-Builds),
diekstrak ke `backend\ffmpeg\` lewat PowerShell `Expand-Archive`. Tidak
diinstall ke sistem, tidak menyentuh PATH.

</td>
</tr>
<tr>
<td><b>macOS</b></td>
<td>

Homebrew kalau sudah ada (`brew install python3`). Kalau tidak, build portable
dari [python-build-standalone](https://github.com/astral-sh/python-build-standalone)
ke `backend/python-portable/` — tanpa sudo/password.

</td>
<td>

Homebrew kalau sudah ada (`brew install ffmpeg`). Kalau tidak, static build
dari [evermeet.cx](https://evermeet.cx/ffmpeg/) ke `backend/ffmpeg/`.

</td>
</tr>
</table>

Proses ini hanya berjalan **sekali** (di percobaan pertama) dan butuh koneksi
internet. Kalau gagal (mis. tidak ada internet atau diblokir firewall
kantor/kampus), pesan error yang jelas akan muncul di terminal (atau
`backend/server.log` kalau lewat auto-start) beserta link untuk pasang manual.

## Cara Pakai

Klik ikon extension di toolbar, lalu pilih salah satu tab:

| Tab | Fungsi |
|---|---|
| **Single** | Tempel satu link video (atau klik **Tab aktif** kalau sedang membuka halaman video YouTube), klik Download. |
| **Multi** | Tempel banyak link sekaligus (satu per baris/koma/spasi) — bisa campur video & playlist, otomatis terdeteksi. |
| **Playlist** | Tempel link playlist — semua videonya diunduh ke sub-folder sesuai judul playlist. |
| **Library** | Lihat semua musik yang sudah tersimpan di folder aktif. |
| **Setelan** | Ganti folder penyimpanan, atur folder bank musik, filter durasi minimum, dan cek status backend/FFmpeg. |

Progres, log, dan status berhasil/gagal tiap video ditampilkan secara
real-time di dalam popup.

## Konfigurasi

Semua pengaturan ada di tab **Setelan** pada extension:

- **Folder Penyimpanan** — lokasi MP3 disimpan. Default:
  `~/Music/YouTube MP3 Downloads`. Bisa diganti kapan saja lewat dialog folder
  native.
- **Folder Bank Musik** — folder tambahan yang *hanya dicek isinya* (tidak
  pernah ditulis), berguna kalau koleksi musikmu tersebar di beberapa
  folder/drive. Video/playlist yang sudah ada di salah satu folder ini tidak
  akan diunduh ulang.
- **Filter Durasi Minimum** — video dengan durasi di bawah nilai ini (default
  30 detik) tidak diunduh; judul & link-nya tetap ditampilkan untuk dicek
  manual. Berguna untuk melewati YouTube Shorts.
- **`cookies.txt`** *(opsional, manual)* — untuk mengunduh video yang butuh
  login atau dibatasi umur, taruh file `cookies.txt` format Netscape (hasil
  export ekstensi seperti "Get cookies.txt") di dalam folder `backend/`.
  Backend otomatis mendeteksi dan memakainya.

## Auto-Start saat Boot/Login

Supaya backend otomatis menyala tiap kali laptop hidup (tidak perlu
double-klik `start.bat`/`start.sh` manual tiap saat):

**Windows:**
```bat
cd backend
install_autostart.bat
```

**macOS:**
```bash
cd backend
chmod +x install_autostart_mac.sh
./install_autostart_mac.sh
```

Backend akan berjalan diam-diam di background (tanpa jendela terminal
muncul). Cek statusnya lewat dot hijau di tab **Setelan**, atau lihat log
lengkap di `backend/server.log`.

Untuk menonaktifkan: jalankan `uninstall_autostart.bat` (Windows) atau
`./uninstall_autostart_mac.sh` (macOS).

## Struktur Proyek

```
.
├── backend/
│   ├── server.py                  # REST API lokal (Flask)
│   ├── downloader_core.py         # Logika inti: yt-dlp + FFmpeg + deteksi duplikat
│   ├── requirements.txt
│   ├── _bootstrap_windows.bat     # Auto-install Python & FFmpeg (Windows)
│   ├── _bootstrap_mac.sh          # Auto-install Python & FFmpeg (macOS)
│   ├── start.bat / start.sh       # Entry point utama
│   ├── start_silent.bat / start_silent_mac.sh   # Dipakai oleh auto-start
│   ├── install_autostart.bat / .sh
│   ├── uninstall_autostart.bat / .sh
│   ├── run_hidden.vbs             # Peluncur tanpa jendela CMD (Windows)
│   ├── com.samxode.ytmp3downloader.backend.plist.template  # LaunchAgent (macOS)
│   └── config.json                # Dibuat otomatis (pengaturan tersimpan)
└── extension/
    ├── manifest.json              # Manifest V3
    ├── popup.html / popup.css / popup.js
    ├── background.js
    └── icons/
```

## Referensi API Backend

Backend menyediakan REST API sederhana di `http://127.0.0.1:8766` (dipakai
oleh extension, tapi bisa juga dipanggil langsung untuk keperluan lain):

<details>
<summary><b>Klik untuk lihat daftar endpoint</b></summary>

| Method | Endpoint | Keterangan |
|---|---|---|
| `GET` | `/api/health` | Status backend, FFmpeg, dan versi kode. |
| `GET`/`POST` | `/api/settings` | Baca/ubah folder penyimpanan & filter durasi. |
| `POST` | `/api/browse-folder` | Buka dialog folder native OS. |
| `GET` | `/api/bank-folders` | Daftar folder bank musik. |
| `POST` | `/api/bank-folders/add` | Tambah folder bank musik. |
| `POST` | `/api/bank-folders/remove` | Hapus folder bank musik. |
| `POST` | `/api/bank-folders/update` | Ganti path folder bank musik. |
| `POST` | `/api/download/single` | Mulai job unduh satu URL. |
| `POST` | `/api/download/multiple` | Mulai job unduh banyak URL. |
| `POST` | `/api/download/playlist` | Mulai job unduh playlist. |
| `GET` | `/api/job/<job_id>` | Status & log job (polling). |
| `POST` | `/api/job/<job_id>/cancel` | Batalkan job yang sedang berjalan. |
| `GET` | `/api/jobs` | Daftar 20 job terakhir. |
| `GET` | `/api/library` | Daftar musik yang sudah tersimpan. |
| `POST` | `/api/open-folder` | Buka folder penyimpanan di file explorer. |

</details>

## Troubleshooting

<details>
<summary><b>Download gagal dengan error "ffprobe and ffmpeg not found"</b></summary>

Pastikan kamu memakai versi backend terbaru — versi lama punya bug di mana
FFmpeg terdeteksi OK oleh health-check tapi tidak diberi tahu ke `yt-dlp`
secara eksplisit. Restart backend (`start.bat`/`start.sh`) setelah update.
</details>

<details>
<summary><b>Backend menyala tapi extension bilang "Backend tidak aktif"</b></summary>

- Pastikan jendela terminal `start.bat`/`start.sh` masih terbuka (atau
  auto-start sudah terpasang).
- Cek apakah port `8766` dipakai proses lain — restart backend.
- Klik tombol refresh (🔄) di pojok kanan atas popup extension.
</details>

<details>
<summary><b>Auto-start sudah dipasang tapi backend tetap tidak pernah menyala (macOS)</b></summary>

`launchd` menjalankan LaunchAgent dengan PATH minimal yang tidak menyertakan
lokasi Homebrew — versi terbaru skrip ini sudah menambahkan PATH tersebut
secara eksplisit. Cek `backend/server.log` untuk detail error, dan pastikan
kamu pakai skrip auto-start versi terbaru.
</details>

<details>
<summary><b>Tab Single tiba-tiba mengunduh seluruh playlist/Mix</b></summary>

Ini bug lama yang sudah diperbaiki: link `watch` YouTube yang disalin saat
autoplay/Mix/Up Next sering membawa parameter `&list=...` walau kamu cuma mau
video itu saja. Tab Single/Multi sekarang selalu memperlakukan link `watch`
sebagai video tunggal, mengabaikan `list=` yang cuma "nebeng".
</details>

<details>
<summary><b>Video gagal diunduh: privat/dibatasi umur/region-locked</b></summary>

Ini pembatasan dari YouTube sendiri. Untuk video yang butuh login, taruh
`cookies.txt` di folder `backend/` (lihat [Konfigurasi](#konfigurasi)).
</details>

## FAQ

**Apakah data saya dikirim ke server manapun?**
Tidak. Semua proses (ekstraksi, konversi, penyimpanan) terjadi 100% lokal di
komputermu. Backend hanya menerima request dari extension di `127.0.0.1`.

**Kenapa butuh backend Python terpisah, bukan langsung di extension?**
Karena kebijakan keamanan browser (semua browser, bukan cuma Chrome) melarang
extension menjalankan proses eksternal seperti FFmpeg atau menulis file
langsung ke sistem.

**Apakah bisa dipakai di Firefox/Edge/Brave?**
Extension ini dibuat untuk Manifest V3 (Chrome/Chromium). Untuk browser
berbasis Chromium lain (Edge, Brave, Opera) kemungkinan besar bisa dipakai
langsung lewat "Load unpacked" yang serupa. Firefox butuh penyesuaian manifest.

**Bagaimana kualitas audionya?**
MP3 320kbps CBR (constant bitrate) — bitrate tertinggi standar MP3, diambil
dari sumber audio terbaik yang tersedia di video.

## Keamanan & Privasi

- Backend hanya listen di `127.0.0.1` (localhost) — **tidak bisa diakses**
  dari perangkat/jaringan lain.
- Tidak ada telemetry, analytics, atau pengumpulan data apa pun.
- Tidak ada API key atau kredensial pihak ketiga yang dibutuhkan.
- Kode sepenuhnya terbuka untuk diperiksa — tidak ada proses tersembunyi.

## Disclaimer

Menghormati hak cipta serta [Persyaratan Layanan
YouTube](https://www.youtube.com/t/terms) sepenuhnya menjadi tanggung jawab
pengguna. Jangan gunakan tool ini untuk mendistribusikan ulang konten
berhak cipta tanpa izin dari pemiliknya.



## Lisensi

Didistribusikan di bawah [Lisensi MIT](LICENSE). Bebas dipakai, dimodifikasi,
dan didistribusikan ulang selama mencantumkan atribusi.

---

<div align="center">

Dibuat dengan ❤️ oleh **&lt;SamXode/&gt;**

</div>
