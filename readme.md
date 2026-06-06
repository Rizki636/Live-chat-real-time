# 🎮 Live Chat Overlay — Panduan Setup Lengkap

Sistem overlay ini mengubah komentar YouTube Live Chat menjadi karakter pixel art yang hidup di bagian bawah layar OBS.

---

## 📐 Arsitektur Sistem

```
YouTube Live Chat
       ↓ (YouTube Data API v3)
Google Apps Script (pollingLoop tiap 1 menit)
       ↓ (simpan ke Sheet)
Google Spreadsheet (buffer komentar)
       ↓ (Web App endpoint / doGet)
overlay.html (GitHub Pages)
       ↓
OBS Browser Source (1920×1080, transparent)
```

---

## 🔑 LANGKAH 1 — Dapatkan YouTube Data API Key

1. Buka [Google Cloud Console](https://console.cloud.google.com/)
2. Buat project baru (atau gunakan yang sudah ada)
3. Masuk ke **APIs & Services → Library**
4. Cari **"YouTube Data API v3"** → Enable
5. Buka **APIs & Services → Credentials**
6. Klik **Create Credentials → API Key**
7. Salin API Key tersebut

> ⚠️ Disarankan membatasi API Key hanya untuk YouTube Data API v3

---

## 📋 LANGKAH 2 — Setup Google Spreadsheet & Apps Script

1. Buka [Google Sheets](https://sheets.google.com/) → buat spreadsheet baru
2. Beri nama spreadsheet, misal: **"LiveChat Overlay"**
3. Buka menu **Extensions → Apps Script**
4. Hapus kode default, paste seluruh isi file `Code.gs`
5. Edit bagian konfigurasi di baris atas:
   ```javascript
   const YT_API_KEY   = 'ISI_API_KEY_ANDA_DI_SINI';
   const LIVE_VIDEO_ID = 'ISI_VIDEO_ID_YOUTUBE_DI_SINI';
   ```

   > Video ID ada di URL YouTube:  
   > `youtube.com/watch?v=`**`VIDEO_ID_INI`**

6. Klik **Save** (💾)

---

## 🚀 LANGKAH 3 — Deploy Apps Script sebagai Web App

1. Di Apps Script, klik **Deploy → New deployment**
2. Klik ikon ⚙️ di sebelah "Select type" → pilih **Web App**
3. Isi konfigurasi:
   - **Description**: Live Chat Overlay v1
   - **Execute as**: Me (your Gmail account)
   - **Who has access**: Anyone
4. Klik **Deploy**
5. Izinkan permissions yang diminta (akses ke YouTube, Spreadsheet)
6. **Salin URL Web App** yang muncul (formatnya:  
   `https://script.google.com/macros/s/XXXX.../exec`)

---

## ⚙️ LANGKAH 4 — Setup Trigger Polling

1. Masih di Apps Script editor
2. Klik **Run → resetToken** (bersihkan data lama)
3. Klik **Run → setupTrigger**
4. Izinkan permissions jika diminta
5. Trigger akan otomatis dibuat: `pollingLoop` tiap 1 menit

> **Catatan**: Apps Script minimum trigger = 1 menit.  
> `pollingLoop` melakukan polling internal setiap 5 detik dalam 1 menit tersebut,  
> sehingga komentar baru muncul dalam ~5-10 detik.

---

## 🌐 LANGKAH 5 — Upload Overlay ke GitHub Pages

1. Buka [github.com](https://github.com) → Login / buat akun
2. Buat repository baru, misal: `livestream-overlay`
   - Visibility: **Public**
3. Upload file `overlay.html` ke repository
4. Buka **Settings → Pages**
5. Source: **Deploy from branch** → pilih `main` / `master`
6. Save → tunggu 1-2 menit
7. GitHub Pages URL akan muncul:  
   `https://USERNAME.github.io/livestream-overlay/overlay.html`

---

## 🔧 LANGKAH 6 — Konfigurasi overlay.html

Sebelum upload ke GitHub, edit file `overlay.html`:

Temukan bagian CONFIG di dalam `<script>`:

```javascript
const CONFIG = {
  // ← Ganti dengan URL Web App Apps Script dari Langkah 3
  APPS_SCRIPT_URL: 'https://script.google.com/macros/s/XXXX/exec',

  POLL_INTERVAL: 2000,          // Poll tiap 2 detik (ms)
  QUEUE_PROCESS_INTERVAL: 1200, // Tampilkan 1 komentar/1.2 detik
  BUBBLE_DURATION: 5000,        // Bubble tampil 5 detik
  INACTIVITY_MINUTES: 5,        // Karakter hilang setelah 5 menit tidak aktif
  MAX_CHARS: 20,                // Maks 20 karakter di layar
  WALK_SPEED: 0.8,              // Kecepatan jalan karakter
  ...
};
```

Sesuaikan nilai-nilai tersebut dengan kebutuhan stream Anda.

---

## 📺 LANGKAH 7 — Tambahkan ke OBS

1. Buka OBS Studio
2. Di panel **Sources**, klik **+** → pilih **Browser**
3. Isi konfigurasi:
   - **URL**: `https://USERNAME.github.io/livestream-overlay/overlay.html`
   - **Width**: `1920`
   - **Height**: `1080`
   - ✅ **Shutdown source when not visible**
   - ✅ **Refresh browser when scene becomes active**
4. Klik **OK**
5. Posisikan source di **paling atas** layer stack (agar karakter tampil di atas elemen lain)
6. Pastikan background OBS scene/game **bisa terlihat** di belakang overlay

> 💡 Overlay sudah `background: transparent` sehingga hanya karakter & ground yang tampil.

---

## 🎬 Sebelum Mulai Stream

Lakukan langkah ini setiap kali memulai stream baru:

1. Jalankan `resetToken()` di Apps Script (agar tidak mengambil komentar stream lama)
2. Update `LIVE_VIDEO_ID` di `Code.gs` dengan Video ID stream yang baru
3. Re-deploy Apps Script jika ada perubahan kode:
   **Deploy → Manage deployments → Edit → New version → Deploy**
4. Refresh browser source di OBS

---

## 🐛 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| Karakter tidak muncul | Pastikan URL Apps Script benar di CONFIG |
| Komentar tidak terdeteksi | Cek apakah stream benar-benar LIVE, bukan premieres |
| API Error 403 | Cek API Key dan pastikan YouTube Data API v3 sudah di-enable |
| Overlay tidak transparan di OBS | Pastikan tidak ada `background-color` pada `body` |
| Terlalu banyak karakter | Kurangi `MAX_CHARS` di CONFIG |
| Komentar terlalu cepat | Naikkan `QUEUE_PROCESS_INTERVAL` (misal 2000 = 2 detik) |

---

## 🧪 Mode Demo (Testing tanpa Apps Script)

Jika `APPS_SCRIPT_URL` masih berisi `'YOUR_APPS_SCRIPT_URL_HERE'`,  
overlay otomatis masuk **Demo Mode** dan menampilkan komentar contoh.

Gunakan ini untuk melihat preview tampilan sebelum setup selesai.

---

## 💸 Biaya

| Layanan | Biaya |
|---------|-------|
| GitHub Pages | **Gratis** |
| Google Apps Script | **Gratis** (6 menit/hari execution time) |
| Google Spreadsheet | **Gratis** |
| YouTube Data API v3 | **Gratis** (10.000 quota/hari, cukup untuk ~1-2 jam stream) |

> ⚠️ **Quota YouTube API**: setiap request Live Chat Messages = 5 units.  
> Dengan polling 5 detik → ~720 request/jam → ~3.600 units/jam.  
> Untuk 2 jam stream = ~7.200 units. Aman dalam limit gratis 10.000/hari.

---

## 📁 Struktur File

```
📂 Project
├── overlay.html          ← Upload ke GitHub Pages
├── apps-script/
│   └── Code.gs          ← Paste ke Google Apps Script
└── PANDUAN_SETUP.md     ← Panduan ini
```

---

Selamat streaming! 🎮✨
