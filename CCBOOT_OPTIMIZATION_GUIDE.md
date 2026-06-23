# GameHub Launcher - CCBoot Diskless Optimization Guide

> Panduan Khusus untuk Lingkungan Warnet Diskless dengan CCBoot
>
> Fokus: Stabilitas, Keandalan, dan Performa di Lingkungan Jaringan

---

## 1. Pendahuluan

Lingkungan **CCBoot Diskless** memiliki karakteristik unik yang berbeda dengan PC biasa:
- Semua file (termasuk game) disimpan di **server CCBoot**
- Client boot dari jaringan (PXE)
- Path game bisa berubah konfigurasi server berubah
- Disk IO melibatkan jaringan (lebih lambat dibanding local SSD)
- Semua client "share" file yang sama dari server

Dokumen ini menjelaskan bagaimana mengoptimalkan GameHub Launcher untuk lingkungan ini.

---

## 2. Risiko Utama & Solusi

### 2.1 Risiko: Path Game Berubah atau Hilang

#### Masalah
- Operator sering memindahkan folder game atau mengubah struktur drive di server
- Path yang disimpan di database bisa menjadi invalid
- Launcher gagal menjalankan game karena path tidak ditemukan

#### Solusi
##### A. Relative Path + Base Path Configuration
Jangan simpan **absolute path** di database! Gunakan **relative path** + **base path** yang dapat dikonfigurasi.

**Contoh:**
```
Base Path (di settings): D:\Games\
Relative Path di DB: Valorant\VALORANT.exe
Full Path = Base Path + Relative Path → D:\Games\Valorant\VALORANT.exe
```

**Keuntungan:**
- Jika operator memindahkan semua game ke `E:\Games\`, cukup ubah **1 setting** saja
- Tidak perlu mengedit setiap game di database

**Implementasi di Database:**
```sql
ALTER TABLE games ADD COLUMN relative_exe_path TEXT;
ALTER TABLE games ADD COLUMN relative_install_path TEXT;
-- Jangan hapus kolom lama untuk backward compatibility!
```

##### B. Fallback Path & Multiple Base Paths
Dukung **multiple base paths** untuk skenario yang lebih kompleks:
```
Base Path 1: D:\Games\
Base Path 2: E:\SteamLibrary\
Base Path 3: F:\SinglePlayerGames\
```

Launcher akan mencoba base path satu per satu sampai menemukan executable yang valid.

##### C. Auto-Detect Game Folder
Saat startup, launcher bisa **scan folder game** secara otomatis dan mencocokkan dengan game di database berdasarkan:
- Nama executable
- Ukuran file
- Hash sederhana (opsional)

---

### 2.2 Risiko: Executable Invalid/Corrupt Sebelum Launch

#### Masalah
- File exe bisa corrupt di server
- Izin akses bisa salah
- File bisa di-quarantine oleh antivirus

#### Solusi
##### A. Validasi Multi-Step Sebelum Launch
Lakukan validasi berikut **sebelum** mencoba menjalankan game:

| Langkah | Validasi | Aksi Jika Gagal |
|---------|----------|-----------------|
| 1 | File exists? | Tampilkan error "Game tidak ditemukan" + suggest repair |
| 2 | File size > 0? | Tampilkan error "File game corrupt" |
| 3 | File extension .exe/.bat/.lnk? | Tampilkan error "File bukan executable" |
| 4 | Can read file? | Tampilkan error "Tidak ada izin akses" |
| 5 | (Optional) Quick hash check | Tampilkan error "File corrupt" |

##### B. Graceful Error Handling
Jangan biarkan launcher crash! Tampilkan pesan error yang **jelas dan actionable** untuk user dan operator:
```
❌ Gagal menjalankan Valorant

Alasan: File executable tidak ditemukan di D:\Games\Valorant\VALORANT.exe

💡 Tips untuk Operator:
1. Periksa apakah game masih ada di folder tersebut
2. Periksa base path configuration di Settings
3. Jalankan Repair dari Admin Menu
```

---

### 2.3 Risiko: Tidak Ada Mekanisme Repair/Update Path Otomatis

#### Masalah
- Jika banyak path berubah, operator harus edit satu per satu
- Memakan waktu dan rawan kesalahan

#### Solusi
##### A. Repair Mode di Admin Modal
Tambahkan fitur **"Repair Library"** di admin modal:

1. **Quick Scan**: Cek semua game dan tandai yang path-nya invalid
2. **Auto-Repair**: Coba cari game di base paths yang tersedia
3. **Bulk Edit**: Ubah base path untuk semua game sekaligus
4. **Verify All**: Validasi semua executable di library

**UI Repair Mode:**
```
┌─────────────────────────────────────┐
│     🔧 Repair Game Library          │
├─────────────────────────────────────┤
│ Status: 142 OK, 8 Invalid            │
├─────────────────────────────────────┤
│ [📋 Scan All]  [🔧 Auto-Repair]     │
│ [📝 Bulk Edit Base Path]            │
├─────────────────────────────────────┤
│ Invalid Games:                      │
│ ❌ Valorant - Path not found        │
│ ❌ Point Blank - File corrupt       │
└─────────────────────────────────────┘
```

##### B. Auto-Repair Saat Startup (Opsional)
Launcher bisa melakukan **quick check** saat startup:
- Cek 10 game teratas (favorites + recently played)
- Jika ada yang invalid, tampilkan notifikasi ke operator
- Tawarkan untuk menjalankan Repair Mode

---

### 2.4 Risiko: Cache Cover Image di Client Diskless

#### Masalah
- Di environment diskless, **setiap client harus download cover dari server**
- Jika banyak client booting bersama, bisa overload jaringan
- Loading cover lambat karena IO jaringan

#### Solusi
##### A. Cache Hierarchy 3 Tingkat
Gunakan strategi cache berikut untuk optimal:

```
1. RAM Cache (Level 1 - Tercepat)
   └─ Cache cover game yang sedang ditampilkan di UI
   └─ Auto evict jika tidak terlihat selama 1 menit

2. Local Disk Cache (Level 2 - Cepat)
   └─ Simpan thumbnail di folder %LOCALAPPDATA%\GameHub\Cache\
   └─ Karena diskless, ini sebenarnya RAM disk di client (cepat!)
   └─ Max size: 100MB (auto delete file lama jika penuh)

3. Server Source (Level 3 - Asli)
   └─ Folder Cover di server CCBoot (read-only)
```

##### B. Pre-Generate Thumbnail di Server
Operator harus **pre-generate semua thumbnail** di server:
- Ukuran: 256px (untuk grid) dan 512px (untuk detail)
- Format: WebP (kecil dan cepat)
- Simpan di folder yang sama dengan game, atau folder centralized

**Struktur Folder di Server:**
```
D:\GameHub\
├── Covers\
│   ├── 256px\
│   │   ├── valorant.webp
│   │   └── pointblank.webp
│   └── 512px\
│       ├── valorant.webp
│       └── pointblank.webp
└── Database\
    └── launcher.db
```

##### C. Lazy Loading dengan Placeholder
- Gunakan placeholder abu-abu saat cover sedang dimuat
- Load cover **hanya** saat game card terlihat di viewport
- Gunakan WPF VirtualizingStackPanel untuk virtualisasi

---

### 2.5 Risiko: Tidak Ada Monitoring Kesehatan Library Game

#### Masalah
- Operator tidak tahu game mana yang corrupt
- Tidak ada laporan tentang game yang sering gagal di-launch
- Susah troubleshooting di warnet yang ramai

#### Solusi
##### A. Health Monitor Panel di Admin
Tambahkan tab **"Health Monitor"** di admin modal:

| Kolom | Keterangan |
|-------|------------|
| Game | Nama game |
| Status | ✅ OK / ⚠️ Warning / ❌ Error |
| Last Launch | Terakhir kali berhasil diluncurkan |
| Launch Count | Jumlah berhasil launch |
| Fail Count | Jumlah gagal launch |
| Last Error | Pesan error terakhir |

**Fitur Health Monitor:**
- Auto-scan library setiap 24 jam
- Kirim notifikasi jika ada game dengan Fail Rate > 10%
- Export laporan ke CSV untuk analisis

##### B. Local Logging dengan Rotation
Simpan log di client (dan opsional di server):
- Folder log: `%LOCALAPPDATA%\GameHub\Logs\`
- Rotasi log harian (max 7 hari)
- Log level: Error, Warning, Info
- Format: Simple text (mudah dibaca)

**Contoh Log:**
```
2026-06-23 14:32:15 [INFO] Starting GameHub Launcher v1.0.0
2026-06-23 14:32:16 [INFO] Loaded 145 games from database
2026-06-23 14:32:45 [ERROR] Failed to launch Valorant: File not found
2026-06-23 14:33:10 [INFO] Launched Point Blank successfully
```

---

## 3. Rekomendasi Tambahan untuk Lingkungan Warnet

### 3.1 Quick Launch dari Desktop Shortcut
Buat **shortcut launcher** di desktop semua client:
- Target: `D:\GameHub\GameHub.exe`
- Start in: `D:\GameHub\`
- Icon: Gunakan icon keren untuk menarik perhatian

Opsional: Tambahkan **argument** untuk launch game langsung:
```
GameHub.exe --launch "Valorant"
```

### 3.2 Auto-Start dengan Billing Software
Integrasikan dengan billing software warnet (misal: Penerbit Billing, etc.):
- Launcher otomatis terbuka saat user login ke billing
- Launcher otomatis tertutup saat user logout

### 3.3 Lockdown Mode untuk User
Di client warnet, **non-aktifkan fitur admin** kecuali dengan password khusus:
- Admin button hanya terlihat jika menekan kombinasi tombol rahasia (misal: Ctrl+Shift+A)
- Tidak ada shortcut ke folder settings
- Tidak ada akses ke file system dari launcher

### 3.4 Hotkey Support
Tambahkan **global hotkey** untuk quick access:
- `F12`: Buka GameHub Launcher
- `Ctrl+F`: Fokus ke search bar
- `Ctrl+1/2/3`: Pindah tab Favorites/All/Recent

### 3.5 Offline-First dengan Lazy Sync (Untuk Masa Depan)
Jika nanti ada API untuk multi-branch:
- Semua operasi tetap lokal terlebih dahulu
- Sync ke server hanya saat ada koneksi
- Conflict resolution: Last Write Wins (sederhana dan efektif)

---

## 4. Checklist Deployment di CCBoot

Sebelum rilis ke warnet, pastikan semua ini sudah dicek:

| Item | Status |
|------|--------|
| Launcher di-install di folder read-only di server | ⬜ |
| Database launcher di folder writeable (misal: D:\GameHub\Database\) | ⬜ |
| Covers di folder read-only dengan pre-generated WebP | ⬜ |
| Base path sudah dikonfigurasi dengan benar | ⬜ |
| Semua game di-scan dan divalidasi | ⬜ |
| Shortcut launcher di desktop semua client | ⬜ |
| Auto-start dengan billing software sudah di-setup | ⬜ |
| Lockdown mode sudah diaktifkan | ⬜ |
| Test launch 5 client bersama-sama (cek jaringan) | ⬜ |
| Test launch 10 game yang berbeda (cek stability) | ⬜ |

---

## 5. Kesimpulan

Lingkungan CCBoot diskless memerlukan **perhatian khusus** pada:
1. **Path management**: Gunakan relative path + base path
2. **Validasi ketat**: Cek semua file sebelum launch
3. **Cache strategy**: 3 tingkat cache untuk kecepatan
4. **Health monitoring**: Pantau kesehatan library secara terus-menerus
5. **Lockdown**: Jangan izinkan user mengubah setting penting

Dengan panduan ini, GameHub Launcher akan **stabil, cepat, dan mudah di-maintain** di lingkungan warnet diskless!
