# GameHub Launcher
## Product Vision (Final & Konsisten)

Version: 2.0 (Final)
Target Platform: Windows 10 / Windows 11
Target Environment: Warnet Diskless (CCBoot)

---

## 1. Core Principles

### Prinsip Utama (TIDAK BERUBAH!)
> **"Find game fast. Launch game instantly."**

Launcher ini **bukan** platform, bukan social network, bukan store. Hanya launcher game untuk warnet.

### Secondary Principles
- Fast First (kecepatan adalah raja)
- Offline First (tidak butuh internet)
- Simple & Reliable (tidak ada fitur yang tidak perlu)
- Low Resource Usage (RAM < 50MB idle)
- Diskless-Optimized (CCBoot-friendly)

---

## 2. Product Goals

### Primary Goals (MVP v1.0)
1. User dapat mencari game dalam < 1 detik
2. User dapat menjalankan game dengan 1 klik (double-click)
3. Launcher berjalan dengan RAM < 50MB saat idle
4. Launcher startup dalam < 1 detik
5. Launcher tetap responsif di PC spesifikasi rendah

### Secondary Goals (v1.5+)
- Meningkatkan discoverability game dengan Trending & Genre Filter
- Menyediakan analytics lokal untuk operator
- Mempermudah manajemen game dengan admin panel terpisah

---

## 3. Version Roadmap (Final & Konsisten)

### Version 1.0 (MVP - Production Ready!)
**Fitur Wajib:**
- ✅ Library (semua game dalam grid view dengan cover)
- ✅ Real-time Search Game (filter nama game)
- ✅ Launch Game (double-click atau tombol Play)
- ✅ Favorites (toggle bintang di game card)
- ✅ Recently Played (10 game terakhir)
- ✅ Admin Modal (password-protected: Add/Edit/Delete Game)
- ✅ Basic Error Handling
- ✅ Performance Optimized (RAM < 50MB, startup < 1s)

**Tidak Ada di MVP:**
- ❌ Hero Banner / Home Page terpisah
- ❌ REST API Layer
- ❌ Admin Panel terpisah
- ❌ Recommendation Engine
- ❌ Analytics Engine
- ❌ Tags / Collections
- ❌ Trending Games
- ❌ Multi-branch / Cloud Sync

### Version 1.5 (Enhanced Discovery)
- Trending Games (7 hari terakhir)
- Genre Filter
- Recently Added Games (30 hari terakhir)
- Basic Local Analytics
- Thumbnail 256px (cache untuk grid view)
- Backup/Restore Database

### Version 2.0 (Advanced Features)
- Tags System
- Admin Panel Terpisah
- REST API (Optional - untuk sync multi-PC)
- Collections
- Simple Recommendations (same genre)
- Auto-scan Games Folder

---

## 4. UI/UX Guidelines (Final)

### Design Philosophy
- Search First (search bar selalu di atas, fokus otomatis saat startup)
- Click Less (max 2 klik untuk main game)
- Minimal Distraction (tidak ada banner/animasi berat)
- Dark Theme (nyaman di warnet)

### Main Layout
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
│  [Grid Game Cards - Virtualized]                    │
│  [Grid Game Cards - Virtualized]                    │
└─────────────────────────────────────────────────────┘
```

### Tab Navigation (Hanya 3!)
1. **⭐ Favorites**: Game favorit user
2. **📚 All Games**: Semua game aktif (default)
3. **⏰ Recently Played**: 10 game terakhir dimainkan

---

## 5. Database Schema (Final - v1.0 Compatible)

### Core Tables (Digunakan di MVP)
1. **games**: Semua informasi game
2. **favorites**: Game favorit
3. **launches**: Histori launch game (untuk Recently Played & Trending nanti)

### Extended Tables (Disiapkan untuk v1.5+)
4. **tags**: Tag untuk game
5. **game_tags**: Relasi many-to-many game & tag

Lihat **ARSITEKTUR_FINAL.md** untuk schema lengkap.

---

## 6. Technology Stack (Final)

| Komponen | Pilihan |
|----------|---------|
| UI Framework | WPF (.NET 8+) |
| Database | SQLite |
| Language | C# |
| Image Format | WebP (256px & 512px) |
| ORM | EF Core (atau Dapper) |

**Tidak Dipakai:**
- Electron / Tauri / WebView (too heavy)
- CQRS / Mediator / Domain Events (overkill)
- Microservices (monolith dulu!)

---

## 7. Success Metrics

1. Search success: < 1 detik
2. Launch success: > 99%
3. Startup time: < 1 detik
4. RAM usage: < 50MB idle
5. No crashes di warnet 24/7
