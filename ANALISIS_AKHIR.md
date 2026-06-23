# GameHub Launcher - Analisis dan Rekomendasi Arsitektur

---

## 1. Executive Summary

Proyek GameHub Launcher memiliki tujuan yang jelas dan sesuai: launcher game cepat untuk lingkungan warnet diskless. Namun, terdapat **banyak fitur dan desain yang over-engineered** untuk MVP (Minimum Viable Product), yang dapat mengakibatkan:
- Keterlambatan waktu pengembangan
- Risiko performa (RAM > 100MB, startup > 1s)
- Kompleksitas maintenance yang tidak perlu

Rekomendasi utama: **Sederhanakan drastis untuk MVP 1.0**, fokus pada fitur inti saja, lalu tambahkan fitur secara bertahap di versi berikutnya.

---

## 2. Problem List

### 2.1 Inkonsistensi Antar Dokumen
- **01_PRODUCT_VISION.md** menyatakan V1.0 hanya: Library, Search, Favorites, Recently Played, Launch Game
- **02_PRODUCT_REQUIREMENTS.md** menambahkan: Trending, Collections, Recommendations di V1.0
- **06_FEATURE_ROADMAP.md** menyatakan V1.0 hanya core, V1.5 untuk Trending/Collections, V2.0 untuk Recommendations
- **09_API_SPECIFICATION.md** dan **10_ADMIN_PANEL_DESIGN.md** termasuk fitur yang belum dijadwalkan di MVP
- **03_SYSTEM_ARCHITECTURE.md** memiliki Clean Architecture 4 layer yang sangat kompleks untuk launcher sederhana

### 2.2 Over-Engineering
- Clean Architecture 4 layer (Presentation → Application → Domain → Infrastructure) untuk launcher sederhana
- 5 "Engine" (Catalog, Search, Launch, Analytics, Recommendation) yang tidak perlu untuk MVP
- Tags & screenshots tables di database yang belum diperlukan
- REST API layer di MVP (padahal semua bisa lokal)
- Admin Panel terpisah di MVP
- AnalyticsService & RecommendationService di V1.0

### 2.3 Risiko Performa
- Clean Architecture dengan banyak abstraksi dapat meningkatkan overhead RAM
- Multi-layer lazy loading bisa menambah startup time
- API layer dan jaringan dapat menambah latency di lingkungan diskless
- Multiple thumbnail sizes (128/256/512) dapat memakan cache space dan IO disk

### 2.4 Redundansi Fitur
- `statistics` table di 01_PRODUCT_VISION.md vs `launches` table di 04_DATABASE_DESIGN.md (duplikat)
- Hero Banner, Continue Playing, Trending, Recently Added, Recommended di Home (overkill untuk warnet)

---

## 3. Architecture Review

### 3.1 Masalah dengan Clean Architecture Saat Ini
Clean Architecture 4 layer (Presentation, Application, Domain, Infrastructure) **terlalu berat dan tidak sesuai** untuk use case launcher warnet sederhana. Alasan:
- Banyak boilerplate code
- Overhead abstraksi meningkatkan RAM usage
- Memperlambat startup time
- Tidak ada manfaat nyata untuk proyek skala kecil

### 3.2 Rekomendasi Arsitektur Simplifikasi
Gunakan **Modular Monolith Ringkas** dengan hanya 2-3 layer:

```
GameHubLauncher/
├── Core/              # Logika inti + data access
│   ├── Models/        # Game, Favorite, LaunchHistory
│   ├── Data/          # SQLite wrapper sederhana
│   └── Services/      # GameService, LaunchService (1 service = 1 file)
├── UI/                # Semua UI (Views, Components)
├── Assets/            # Covers, Icons
├── Cache/             # Thumbnail cache
└── Config/            # Settingan
```

Prinsip: **KISS (Keep It Simple, Stupid)** dan **YAGNI (You Aren't Gonna Need It)**

---

## 4. Database Review

### 4.1 Masalah Schema Saat Ini
- `screenshots`, `tags`, `game_tags` tables: tidak dibutuhkan di MVP
- `slug`, `publisher`, `description` di `games` table: opsional untuk V1.0
- Multiple index yang belum tentu diperlukan di MVP
- `statistics` table vs `launches` table: redundancy

### 4.2 Schema Simplifikasi untuk MVP
Hanya **3 tabel** yang dibutuhkan:

```sql
-- Tabel games (sederhana)
CREATE TABLE games (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    genre TEXT,
    cover_path TEXT,
    exe_path TEXT NOT NULL,
    date_added DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabel favorites
CREATE TABLE favorites (
    id INTEGER PRIMARY KEY,
    game_id INTEGER NOT NULL
);

-- Tabel launches (histori + trending bisa dihitung dari sini)
CREATE TABLE launches (
    id INTEGER PRIMARY KEY,
    game_id INTEGER NOT NULL,
    launch_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Indexes yang diperlukan
CREATE INDEX idx_game_name ON games(name);
CREATE INDEX idx_launch_game ON launches(game_id);
CREATE INDEX idx_launch_time ON launches(launch_time);
```

**Alasan:**
- Trending, Recently Played, dll bisa dihitung dari tabel `launches` tanpa perlu tabel terpisah
- Cukup 1 thumbnail size (misal 256px) untuk menghemat IO dan storage
- SQLite tetap cocok, tapi gunakan wrapper yang ringkas

---

## 5. UI/UX Review

### 5.1 Masalah UI Saat Ini
- **Home page terlalu kompleks**: Hero Banner + Continue Playing + Trending + Recently Added + Recommended = 5 section (overkill untuk warnet)
- **Terlalu banyak navigasi**: Home, Library, Favorites, Recently Played
- **Steam-like design**: bisa terlalu "wah" dan membutuhkan resource lebih banyak

### 5.2 Rekomendasi UI Simplifikasi
**Prinsip:** Search First + Click Less

```
┌───────────────────────────────────────────┐
│ 🔍 Search Game... [FOCUS OTOMATIS]        │
├───────────────────────────────────────────┤
│  ⭐ Favorites  [Tab 1]                    │
│  📚 All Games  [Tab 2] (default)          │
│  ⏰ Recently Played [Tab 3]               │
├───────────────────────────────────────────┤
│  [Grid Game Cards]                        │
│  [Grid Game Cards]                        │
│  [Grid Game Cards]                        │
└───────────────────────────────────────────┘
```

**Alasan:**
- User warnet hanya ingin: Cari Game → Klik → Main
- Search bar fokus otomatis saat startup
- Hanya 3 tab (tidak perlu Home page terpisah)
- Tidak ada banner/animasi berat

---

## 6. Performance Risk Analysis

### 6.1 Lingkungan Diskless CCBoot
Perhatikan batasan:
- **Disk IO**: Semua baca file dari server via jaringan → lambat!
- **RAM**: Target < 100MB idle
- **Startup time**: < 1 detik
- **Tidak boleh ada background service**

### 6.2 Risiko dan Mitigasi
| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| Clean Architecture overhead | RAM ↑, Startup ↑ | Gunakan arsitektur sederhana |
| Multi thumbnail sizes | IO ↑, Cache ↑ | Hanya 1 ukuran (256px) |
| API layer | Latency ↑, Kompleksitas ↑ | Hapus API di MVP |
| Admin Panel terpisah | Waktu dev ↑ | Integrasikan admin sederhana ke launcher (password protected) |
| Multiple services | RAM ↑ | Gabungkan service menjadi 1-2 file |

---

## 7. Feature Prioritization Correction

### MUST HAVE (MVP 1.0)
- [x] Search Game (real-time, lokal)
- [x] Launch Game (1 klik)
- [x] Library (semua game)
- [x] Favorites
- [x] Recently Played

### SHOULD HAVE (Versi 1.5)
- Trending Games (hitung dari launches lokal)
- Recently Added
- Genre Filter

### NICE TO HAVE (Versi 2.0+)
- Collections/Tags
- Screenshots
- Simple Recommendations (same genre)
- Basic Analytics

### SHOULD BE REMOVED dari MVP
- ❌ Hero Banner
- ❌ Home Page terpisah
- ❌ REST API Layer
- ❌ Admin Panel terpisah (ganti dengan modal password protected)
- ❌ Analytics Engine & Recommendation Engine
- ❌ Multi Branch features
- ❌ Cloud Sync
- ❌ Screenshots & Tags tables

---

## 8. Simplified MVP Proposal

### 8.1 Arsitektur Minimal
```
GameHubLauncher/
├── GameHub.exe          # Single executable (WPF or Qt)
├── Data/
│   └── launcher.db      # SQLite
├── Assets/
│   ├── covers/          # 256px WebP
│   └── icons/
├── Cache/               # Auto-generated thumbs
└── config.json          # Simple setting
```

### 8.2 Fitur Minimal
1. **Startup**: Search bar fokus otomatis
2. **Navigasi**: 3 Tab (Favorites, All Games, Recently Played)
3. **Search**: Real-time filter nama game
4. **Launch**: Double-click atau klik tombol Play
5. **Favorites**: Toggle bintang di card
6. **Admin**: Modal password-protected untuk Add/Edit/Delete Game

### 8.3 Database Minimal
Seperti yang diusulkan di Section 4.2 (hanya 3 tabel)

### 8.4 UI Minimal
- Dark theme, sederhana
- Grid 4-5 kolom
- Hanya hover effect ringan (tidak ada scale/animasi berat)
- Search bar selalu di atas

---

## 9. Engineering Recommendations

### 9.1 Tech Stack Pilihan
Pilih salah satu berdasarkan prioritas:

| Prioritas | Tech Stack | Alasan |
|-----------|------------|--------|
| 1 (Terbaik) | **Qt C++** | Paling ringkas, startup tercepat, RAM paling kecil, cross-platform (tapi fokus Windows) |
| 2 (Seimbang) | **WPF (.NET 8+)** | Native Windows, stabil, good performance, cukup ringkas |
| 3 (Alternatif) | **PySide6** | Cepat dikembangkan, tapi bundle size lebih besar |

**JANGAN GUNAKAN:** Electron, Tauri (WebView), atau apapun yang berbasis Chromium → terlalu berat!

### 9.2 Strategi Caching
- Pre-generate thumbnail 256px WebP saat admin menambah game
- Lazy load: Load gambar hanya saat terlihat di viewport
- Cache ke RAM untuk game yang sering ditampilkan

### 9.3 Strategi Startup
- **Load database sync (synchronous)** di awal (SQLite cepat)
- **Load gambar async** setelah UI tampil
- Fokus search bar otomatis saat pertama kali buka
- Hindari inisialisasi berat di startup

### 9.4 Strategi Launch Game
- Gunakan `CreateProcess()` (Windows native) atau `QProcess` (Qt)
- Tidak ada splash screen tambahan
- Verifikasi path exe sebelum launch (tampilkan error jika tidak ketemu)

---

## 10. Final Recommendation (SANGAT PENTING!)

### Langkah-langkah Implementasi yang Disarankan:

1. **HAPUS SEMUA DOKUMEN YANG OVER-ENGINEERED** (atau simpan sebagai referensi future):
   - Sederhanakan 03_SYSTEM_ARCHITECTURE.md menjadi arsitektur modular ringkas
   - Sederhanakan 04_DATABASE_DESIGN.md menjadi 3 tabel
   - Hapus API spec dari MVP
   - Sederhanakan Admin Panel menjadi modal sederhana di launcher

2. **MULAI DARI MVP SEDERHANA**:
   - Fokus pada 5 fitur inti: Search, Library, Launch, Favorites, Recently Played
   - Gunakan arsitektur 2-3 layer saja
   - Target: RAM < 50MB idle, startup < 0.5 detik

3. **TAMBAHKAN FITUR SECARA BERTAHAP**:
   - V1.5: Trending, Genre Filter
   - V2.0: Collections, Screenshots
   - V2.5: Admin Panel terpisah + API (jika benar-benar butuh)
   - V3.0: Multi-branch, cloud sync (jika ada permintaan)

4. **SELALU INGAT PRINSIP UTAMA**:
   > "Find game fast. Launch game instantly." — Everything else is secondary.

---

**Kesimpulan:** Proyek GameHub Launcher memiliki potensi besar, tapi perlu **disederhanakan drastis** untuk MVP agar cepat dirilis, stabil, dan sesuai dengan batasan lingkungan warnet diskless.
