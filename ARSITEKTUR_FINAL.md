# GameHub Launcher - Final Architecture Design

> Arsitektur Seimbang: Sederhana untuk MVP, Extensible untuk Masa Depan
>
> Prinsip: **"Simple, but not simpler"**

---

## 1. Executive Summary

Setelah mempertimbangkan feedback, saya mengusulkan arsitektur **Modular Monolith Ringkas dengan Prinsip Clean Architecture yang Disesuaikan** (tidak 4 layer penuh, tapi tetap terorganisir dengan baik).

Alasan:
- Tidak over-engineering (tanpa CQRS, Mediator, Domain Event)
- Tetap maintainable dan extensible
- Cocok untuk lingkungan warnet diskless (performa tinggi)
- Mudah dikembangkan fitur baru nanti

---

## 2. Final Tech Stack

### Core Technology
| Komponen | Pilihan | Alasan |
|----------|---------|--------|
| **UI Framework** | **WPF (.NET 8+)** | Native Windows, stabil, good performance, tooling mature, cukup ringkas |
| **Database** | **SQLite** | File-based, zero-config, cepat untuk read-heavy workload |
| **Language** | **C#** | Mudah maintain, banyak library, dukungan WPF sangat bagus |
| **Image Format** | **WebP (256px & 512px)** | Small file size, fast decode |

### Why Not Qt C++?
- WPF lebih cepat dikembangkan dan lebih mudah dicari developer-nya di Indonesia
- Performa cukup bagus untuk launcher (bisa < 50MB idle, < 1s startup)
- Tetap native, tidak menggunakan WebView

---

## 3. Final Architecture: Clean Architecture "Ringkas"

Kita gunakan **3 layer saja** (bukan 4 layer penuh), tanpa abstraksi yang tidak perlu:

```
┌─────────────────────────────────────────────┐
│           Presentation Layer (UI)            │
│  (Views, ViewModels, Controls, Resources)   │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│         Application Layer (Services)         │
│  (GameService, LaunchService, SearchService)│
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│        Infrastructure Layer (Data/IO)        │
│  (Database, FileSystem, Cache, Settings)    │
└─────────────────────────────────────────────┘
```

**Aturan Penting:**
- Presentation hanya depend ke Application
- Application hanya depend ke Infrastructure (atau abstraksi sederhana)
- Tidak ada Domain Layer terpisah (model digabung ke Application/Data)
- **Tidak ada CQRS/Mediator/Domain Event** — keep it simple!

---

## 4. Final Folder Structure

```
GameHubLauncher/
├── src/
│   ├── GameHub.Launcher/              # Startup Project (WPF EXE)
│   │   ├── App.xaml / App.xaml.cs
│   │   ├── AssemblyInfo.cs
│   │   └── appsettings.json
│   │
│   ├── GameHub.Presentation/          # Presentation Layer
│   │   ├── Views/
│   │   │   ├── MainWindow.xaml
│   │   │   ├── MainWindowViewModel.cs
│   │   │   ├── GameCardControl.xaml
│   │   │   └── AdminDialog.xaml
│   │   ├── ViewModels/
│   │   │   ├── GameCardViewModel.cs
│   │   │   └── AdminViewModel.cs
│   │   └── Resources/
│   │       ├── Styles.xaml
│   │       └── Icons.xaml
│   │
│   ├── GameHub.Application/           # Application Layer (Business Logic)
│   │   ├── Models/
│   │   │   ├── Game.cs
│   │   │   ├── Favorite.cs
│   │   │   └── LaunchHistory.cs
│   │   ├── Services/
│   │   │   ├── IGameService.cs        # Simple interface (optional but good)
│   │   │   ├── GameService.cs
│   │   │   ├── ILaunchService.cs
│   │   │   ├── LaunchService.cs
│   │   │   └── SearchService.cs       # No interface, simple static class OK
│   │   └── DTOs/                      # Simple records for data transfer
│   │       ├── GameDto.cs
│   │       └── LaunchStatsDto.cs
│   │
│   └── GameHub.Infrastructure/        # Infrastructure Layer (Data/IO)
│       ├── Data/
│       │   ├── DatabaseContext.cs     # Simple SQLite wrapper (EF Core or Dapper)
│       │   └── DatabaseInitializer.cs
│       ├── Cache/
│       │   ├── ImageCache.cs
│       │   └── ThumbnailGenerator.cs
│       ├── IO/
│       │   ├── GameScanner.cs
│       │   └── FileHelper.cs
│       └── Settings/
│           ├── AppSettings.cs
│           └── SettingsManager.cs
│
├── assets/
│   ├── covers/                        # Game cover images (512px WebP)
│   └── icons/                         # App icons
│
├── docs/                              # Documentation
│
└── tests/
    └── GameHub.Tests/                 # Unit tests
```

### Alasan Struktur Ini:
- **Terorganisir dengan baik** (setiap layer jelas tanggung jawabnya)
- **Tidak overkill** (tanpa folder yang tidak perlu)
- **Extensible** (mudah menambah fitur baru tanpa merusak yang ada)
- **Testable** (dependensi jelas, mudah di-mock jika perlu)

---

## 5. Final Database Schema (Balanced)

Kita gunakan **5 tabel** — cukup untuk MVP dan extensible untuk fitur berikutnya:

```sql
-- Tabel Games: Semua informasi game
CREATE TABLE games (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    slug TEXT UNIQUE,               -- Untuk URL-friendly nanti (optional)
    genre TEXT,
    developer TEXT,
    publisher TEXT,
    description TEXT,
    cover_path TEXT,                -- Path ke file cover (512px WebP)
    exe_path TEXT,                  -- Absolute path (backward compatibility)
    relative_exe_path TEXT NOT NULL,-- Relative path dari base path (CCBoot-optimized!)
    install_path TEXT,              -- Absolute install path
    relative_install_path TEXT,     -- Relative install path
    size_bytes INTEGER,             -- Ukuran game dalam bytes
    status TEXT DEFAULT 'active',   -- active / inactive / needs_repair
    health_status TEXT DEFAULT 'unknown', -- unknown / ok / warning / error
    last_health_check DATETIME,     -- Terakhir kali health check dijalankan
    fail_count INTEGER DEFAULT 0,   -- Jumlah gagal launch
    date_added DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_played DATETIME
);

-- Tabel Favorites: Game favorit user
CREATE TABLE favorites (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    game_id INTEGER NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE
);

-- Tabel Launches: Histori launch game
CREATE TABLE launches (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    game_id INTEGER NOT NULL,
    launch_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    duration_seconds INTEGER,      -- Durasi bermain (null jika tidak diketahui)
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE
);

-- Tabel Tags: Tag untuk game (extensible, tapi tidak dipakai di MVP)
CREATE TABLE tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);

-- Tabel GameTags: Relasi many-to-many game & tag
CREATE TABLE game_tags (
    game_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (game_id, tag_id),
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);

-- Indexes untuk performa
CREATE INDEX idx_games_name ON games(name);
CREATE INDEX idx_games_status ON games(status);
CREATE INDEX idx_favorites_game_id ON favorites(game_id);
CREATE INDEX idx_launches_game_id ON launches(game_id);
CREATE INDEX idx_launches_launch_time ON launches(launch_time);
```

### Alasan Schema Ini:
- **Cukup untuk MVP**: Semua fitur inti bisa berjalan
- **Extensible**: Tags sudah disiapkan untuk v1.5
- **Tidak redundan**: `launches` bisa untuk Recently Played dan Trending
- **Relasi jelas**: ON DELETE CASCADE untuk integritas data

---

## 6. Final Roadmap (Realistic & Balanced)

### Version 1.0 (MVP - Production Ready!)
**Target Rilis:** 4-6 minggu  
**Fokus:** Core Features + Stabilitas + Performa

| Fitur | Status | Keterangan |
|-------|--------|------------|
| Search Game Real-time | ✅ | Filter nama game secara lokal |
| Library Grid View | ✅ | Tampilkan semua game dengan cover |
| Launch Game (1-klik) | ✅ | Double-click / klik Play |
| Favorites | ✅ | Toggle bintang di game card |
| Recently Played | ✅ | Tampilkan 10 game terakhir |
| Admin Modal (Password) | ✅ | Add/Edit/Delete game (tanpa panel terpisah) |
| Basic Error Handling | ✅ | Tampilkan pesan error yang jelas |
| Performance Optimized | ✅ | Target: RAM < 50MB idle, startup < 1s |

### Version 1.5 (Enhanced Discovery)
**Target Rilis:** 3-4 minggu setelah v1.0  
**Fokus:** Discovery + Analytics Lokal

| Fitur | Keterangan |
|-------|------------|
| Trending Games | Hitung dari tabel launches (7 hari terakhir) |
| Genre Filter | Filter by genre dari games table |
| Recently Added | Tampilkan game baru di 30 hari terakhir |
| Basic Local Analytics | Lihat total launch per game |
| Thumbnail 256px | Cache untuk grid view (lebih cepat) |
| Backup/Restore DB | Backup database manual |

### Version 2.0 (Advanced Features)
**Target Rilis:** 4-6 minggu setelah v1.5  
**Fokus:** Admin Panel + API (Optional)

| Fitur | Keterangan |
|-------|------------|
| Tags System | Klasifikasi game dengan tags |
| Admin Panel Terpisah | Aplikasi khusus untuk operator |
| REST API (Optional) | Untuk sync multi-PC (offline-first tetap utama) |
| Collections | Grouping game sesuai keinginan |
| Simple Recommendations | Rekomendasi game dengan genre sama |
| Auto-scan Games | Scan folder game otomatis |

---

## 7. Final UI Design (Seimbang)

### Layout Utama (Sederhana tapi Menarik)
```
┌─────────────────────────────────────────────────────┐
│  [ICON] GameHub Launcher          ⚙️ Admin 🔍 Search │
├─────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────┐  │
│  │  🔍 Cari game... (FOCUS OTOMATIS)             │  │
│  └───────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│  [⭐ Favorites]  [📚 All Games (ACTIVE)]  [⏰ Recent]│
├─────────────────────────────────────────────────────┤
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ [COVER]  │ │ [COVER]  │ │ [COVER]  │ │        │ │
│  │ Valorant │ │ PB       │ │ Dota 2   │ │  ...   │ │
│  │ FPS      │ │ FPS      │ │ MOBA     │ │        │ │
│  │ [★] [PLAY]│ │ [★] [PLAY]│ │ [★] [PLAY]│ │        │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘ │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ [COVER]  │ │ [COVER]  │ │ [COVER]  │ │        │ │
│  │ ...      │ │ ...      │ │ ...      │ │  ...   │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘ │
└─────────────────────────────────────────────────────┘
```

### Prinsip UI:
- **Search First**: Search bar selalu di atas, fokus otomatis
- **3 Tab Saja**: Favorites, All Games, Recently Played
- **Dark Theme**: Modern dan nyaman di warnet
- **Hover Ringkas**: Tidak ada animasi berat, cuma highlight sederhana
- **No Banner**: Tidak ada hero banner yang memakan resource

---

## 8. Performance Strategy (Diskless-Optimized)

### 8.1 Startup Optimization
1. **Load DB Synchronous**: SQLite sangat cepat, load dulu sebelum UI tampil
2. **Lazy Load Images**: Load cover hanya saat terlihat di viewport
3. **Cache Thumbnail**: Generate 256px thumbnail dan simpan di cache
4. **No Heavy Init**: Tidak ada background service, tidak ada network call di startup

### 8.2 RAM Optimization
1. **Reuse Image Controls**: Virtualize list/grid (WPF VirtualizingStackPanel)
2. **Dispose with Care**: Release gambar yang tidak terlihat
3. **Target**: < 50MB idle, < 100MB saat banyak game

### 8.3 Disk IO & Path Management Optimization (Khusus CCBoot!)
Ini adalah **bagian paling penting** untuk lingkungan diskless:

1. **Relative Path + Base Path**: Jangan simpan absolute path di DB! Gunakan relative path + base path yang dapat dikonfigurasi.
2. **Multiple Base Paths**: Dukung sampai 3 base path untuk fleksibilitas.
3. **Health Check Otomatis**: Validasi executable sebelum launch (file exists, size > 0, readable).
4. **Cache 3 Tingkat**: RAM → Local Disk (RAM disk di client) → Server.
5. **Pre-Generate Thumbnail**: Operator harus generate semua WebP thumbnail di server.
6. **Repair Mode**: Fitur repair library untuk bulk-edit path atau auto-detect game.

Untuk detail lengkap, lihat **CCBOOT_OPTIMIZATION_GUIDE.md**!

---

## 9. CCBoot-Specific Features (Sudah Termasuk di MVP!)

| Fitur | Deskripsi | Prioritas |
|-------|-----------|-----------|
| Relative Path Support | Gunakan relative path + configurable base path | P0 |
| Multi-Step Validation | Validasi file sebelum launch | P0 |
| Graceful Error Handling | Pesan error jelas dan actionable | P0 |
| Repair Mode di Admin | Scan, auto-repair, bulk edit path | P0 |
| 3-Level Cache Strategy | RAM → Local Disk → Server | P0 |
| Health Status Column | Tracking status kesehatan game | P1 |
| Fail Count Counter | Jumlah gagal launch per game | P1 |
| Simple Logging | Rotasi log harian, max 7 hari | P1 |

---

## 10. Development Guidelines (Sederhana tapi Jelas)

### 10.1 Do's
✅ Gunakan WPF dengan .NET 8+  
✅ Gunakan EF Core (atau Dapper) untuk SQLite — simple dan cepat  
✅ Gunakan VirtualizingStackPanel untuk game grid  
✅ Lazy load gambar dengan async/await  
✅ Tulis unit test untuk service layer  
✅ Gunakan dependency injection sederhana (Microsoft.Extensions.DependencyInjection)  
✅ Selalu validasi path executable sebelum launch!  
✅ Gunakan relative path + base path untuk semua game!

### 10.2 Don'ts
❌ Jangan gunakan CQRS/Mediator pattern — overkill  
❌ Jangan gunakan Event Sourcing/Domain Events — tidak perlu  
❌ Jangan buat microservices — monolith dulu!  
❌ Jangan pakai Electron/Tauri/WebView — too heavy  
❌ Jangan buat abstraksi yang tidak perlu (misal IRepository untuk setiap table)  
❌ Jangan over-validate — keep it simple!  
❌ Jangan simpan absolute path di database!

---

## 11. Kesimpulan Final

Arsitektur ini **seimbang** antara:
1. **Kesederhanaan MVP**: 3 layer, 5 tabel, fitur inti jelas
2. **Performa Diskless**: Optimized untuk CCBoot, RAM rendah, startup cepat
3. **Extensibility**: Mudah menambah fitur v1.5/v2.0 tanpa rewrite besar-besaran
4. **Maintainability**: Struktur jelas, mudah dipahami developer baru
5. **CCBoot-Ready**: Semua fitur khusus lingkungan diskless sudah termasuk!

Ini adalah desain yang **saya benar-benar akan gunakan** sebagai Lead Architect untuk proyek production GameHub Launcher!
