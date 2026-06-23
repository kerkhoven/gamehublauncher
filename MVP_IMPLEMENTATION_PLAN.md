# GameHub Launcher - MVP 1.0 Implementation Plan

> Berdasarkan ARSITEKTUR_FINAL.md dan CCBOOT_OPTIMIZATION_GUIDE.md
>
> Hanya Fitur MVP 1.0!

---

## Pendahuluan
- **Total Sprint**: 9
- **Estimasi Total Waktu**: ~5-7 minggu (dengan 1 sprint = 3-4 hari kerja)
- **Fokus**: Stabilitas, Performa, Core Features CCBoot-Ready
- **No Scope Creep!**

---

## Critical Path
```
Sprint 0 → Sprint 1 → Sprint 2 → Sprint 3 → Sprint 4 → Sprint 5 → Sprint 6 → Sprint 7 → Sprint 8 → Sprint 9 (Release)
   ↓          ↓         ↓         ↓         ↓         ↓         ↓         ↓         ↓         ↓
  Setup     DB+Infra  Models    Search    Launch    Fav+Rec   Admin     Repair    Polish    Testing
```

## Dependency Diagram Sederhana
```
[Project Setup]
    ↓
[Database & Infrastructure]
    ↓
[Core Models & Services] ←───────────────────────────────────┐
    ↓                                                      │
[Search & Library]  ←───────────────────────────────────────┤
    ↓                                                      │
[Game Launch System]  ←────────────────────────────────────┤
    ↓                                                      │
[Favorites & Recently Played]  ←───────────────────────────┤
    ↓                                                      │
[Admin Modal]  ←───────────────────────────────────────────┤
    ↓                                                      │
[Repair Library & Health Check]  ←─────────────────────────┘
    ↓
[UI Polish & Performance]
    ↓
[Testing & MVP Release]
```

---

## Sprint 0 - Project Foundation & Solution Setup

**Tujuan**: Membuat struktur project dan konfigurasi dasar sesuai arsitektur final.

**Deliverables**:
- Solution file dengan 4 project (Launcher, Presentation, Application, Infrastructure)
- Konfigurasi .NET 8 WPF
- Basic folder structure sesuai ARSITEKTUR_FINAL.md
- Dependency Injection setup
- Git repository initialized dengan .gitignore

**Tasks**:
1. **Create WPF Solution & Projects**
   - Deskripsi: Buat solution dengan nama `GameHubLauncher` dan 4 project:
     - `GameHub.Launcher` (WPF EXE, startup project)
     - `GameHub.Presentation` (WPF User Controls, Views, ViewModels)
     - `GameHub.Application` (Models, Services, DTOs)
     - `GameHub.Infrastructure` (Database, Cache, IO, Settings)
   - Dependencies: Tidak ada
   - Kompleksitas: S
   - Acceptance Criteria:
     - Solution bisa di-build tanpa error
     - Semua project reference sesuai (Presentation → Application → Infrastructure)
     - Folder structure sesuai ARSITEKTUR_FINAL.md

2. **Setup .NET 8 & WPF Configuration**
   - Deskripsi: Konfigurasi target framework .NET 8.0 Windows, enable nullable, dan basic WPF settings
   - Dependencies: Task 1
   - Kompleksitas: S
   - Acceptance Criteria:
     - Target framework: net8.0-windows
     - Nullable reference types enabled
     - WPF windows bisa berjalan dan menampilkan window kosong

3. **Setup Dependency Injection (Microsoft.Extensions.DependencyInjection)**
   - Deskripsi: Setup DI container di App.xaml.cs, register service sederhana untuk testing
   - Dependencies: Task 1 & 2
   - Kompleksitas: S
   - Acceptance Criteria:
     - DI container bisa resolve service sederhana
     - Tidak ada error saat startup

4. **Initialize Git Repository & .gitignore**
   - Deskripsi: Init git, buat .gitignore standard untuk .NET/WPF
   - Dependencies: Tidak ada
   - Kompleksitas: S
   - Acceptance Criteria:
     - Git initialized
     - .gitignore ada dan sesuai (tidak track bin/, obj/, user files, dll.)

**Estimasi**: 1 hari (S)

---

## Sprint 1 - Database & Infrastructure Layer

**Tujuan**: Membuat database SQLite, schema final, dan Infrastructure layer dasar.

**Deliverables**:
- SQLite database schema final (5 tabel)
- EF Core DbContext
- Settings manager (appsettings.json)
- Simple file helper
- Cache service dasar

**Tasks**:
1. **Design & Create SQLite Database Schema (EF Core)**
   - Deskripsi: Buat entity classes untuk Game, Favorite, Launch, Tag, GameTag sesuai schema final di ARSITEKTUR_FINAL.md. Buat DbContext dengan semua DbSet dan konfigurasi.
   - Dependencies: Sprint 0 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua entity punya property sesuai schema
     - Relasi foreign key benar (ON DELETE CASCADE)
     - Index sudah dibuat untuk kolom yang sering di-query
     - Migration initial bisa di-generate dan dijalankan tanpa error

2. **Setup EF Core SQLite & Database Initialization**
   - Deskripsi: Install Microsoft.EntityFrameworkCore.Sqlite, buat DatabaseInitializer untuk create DB dan seed sample data (opsional 2-3 game untuk testing)
   - Dependencies: Task 1
   - Kompleksitas: S
   - Acceptance Criteria:
     - Database launcher.db dibuat otomatis di folder Database saat startup
     - Sample data bisa dimasukkan
     - EF Core bisa query dan insert data tanpa error

3. **Create Settings Manager (appsettings.json)**
   - Deskripsi: Buat AppSettings class, SettingsManager untuk membaca dan menulis appsettings.json. Simpan:
     - BasePaths (list<string>)
     - CacheFolderPath
     - DatabasePath
     - AdminPasswordHash (hash sederhana, jangan plain text!)
   - Dependencies: Sprint 0 selesai
   - Kompleksitas: S
   - Acceptance Criteria:
     - appsettings.json dibuat otomatis jika tidak ada
     - Baca dan tulis settings berfungsi
     - Admin password disimpan sebagai hash (gunakan BCrypt atau SHA256 sederhana)

4. **Create Simple File Helper & Cache Service Dasar**
   - Deskripsi: Buat FileHelper untuk path manipulation dan file exists check. Buat ImageCacheService dasar untuk mengelola folder cache.
   - Dependencies: Task 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - FileHelper bisa menggabungkan base path + relative path
     - FileHelper bisa check file exists
     - Cache folder dibuat otomatis di %LOCALAPPDATA%\GameHub\Cache
     - ImageCacheService bisa membersihkan file lama (opsional tapi bagus)

**Estimasi**: 2-3 hari (S-M)

---

## Sprint 2 - Core Models & Application Layer Services

**Tujuan**: Membuat core models, DTOs, dan service dasar di Application layer.

**Deliverables**:
- Core DTOs (GameDto, LaunchStatsDto)
- GameService (CRUD game sederhana)
- Basic validation logic

**Tasks**:
1. **Create Core DTOs**
   - Deskripsi: Buat DTOs sederhana: GameDto, FavoriteDto, LaunchDto. Jangan expose entity langsung ke Presentation layer.
   - Dependencies: Sprint 1 selesai
   - Kompleksitas: S
   - Acceptance Criteria:
     - DTOs punya property yang cukup untuk UI
     - Tidak ada circular dependency
     - Bisa di-map ke/ dari entity (manual mapping cukup, tidak perlu AutoMapper untuk MVP)

2. **Create GameService (CRUD & Query)**
   - Deskripsi: Buat IGameService dan GameService dengan method:
     - GetAllGamesAsync()
     - GetGameByIdAsync(int id)
     - AddGameAsync(GameDto game)
     - UpdateGameAsync(GameDto game)
     - DeleteGameAsync(int id)
     - GetActiveGamesAsync()
   - Dependencies: Task 1 & Sprint 1
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua method berfungsi dengan benar
     - Menggunakan DbContext dari Infrastructure
     - Handle exception sederhana (log, tapi tidak crash)
     - Bisa di-test dengan sample data

3. **Create Path Resolution Logic (CCBoot-Ready)**
   - Deskripsi: Buat helper di GameService untuk resolve full path dari relative_exe_path + base paths. Coba base path satu per satu sampai menemukan file yang valid.
   - Dependencies: Task 2 & Sprint 1 Task 3 & 4
   - Kompleksitas: M
   - Acceptance Criteria:
     - Bisa resolve path dengan multiple base paths
     - Return path valid pertama yang ditemukan
     - Return null jika semua base path gagal
     - Test dengan berbagai skenario path

**Estimasi**: 2-3 hari (M)

---

## Sprint 3 - Search & Library Grid UI

**Tujuan**: Membuat search feature dan library grid UI dengan virtualization.

**Deliverables**:
- SearchService
- MainWindow UI
- GameCard UserControl
- Library grid dengan VirtualizingStackPanel
- Real-time search

**Tasks**:
1. **Create SearchService**
   - Deskripsi: Buat SearchService dengan method SearchGamesAsync(string query, List<GameDto> allGames). Filter berdasarkan nama game (case-insensitive, contains).
   - Dependencies: Sprint 2 selesai
   - Kompleksitas: S
   - Acceptance Criteria:
     - Search real-time bekerja (filter cepat)
     - Case-insensitive
     - Bisa mencari sebagian nama (e.g., "val" → "Valorant")
     - Performance: < 50ms untuk 200 game

2. **Create MainWindow Layout**
   - Deskripsi: Buat MainWindow.xaml sesuai UI final:
     - Top bar: Icon + Title + Admin Button + Search Box
     - Tab Control: Favorites, All Games, Recently Played
     - Game grid area dengan VirtualizingStackPanel
   - Dependencies: Sprint 0 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Layout sesuai desain
     - Search box fokus otomatis saat startup
     - Tab control berfungsi berganti tab
     - Tidak ada layout error di berbagai resolusi

3. **Create GameCard UserControl**
   - Deskripsi: Buat GameCard.xaml UserControl dengan:
     - Cover image placeholder
     - Game name
     - Genre
     - Favorite button (★)
     - Play button
   - Dependencies: Task 2
   - Kompleksitas: S
   - Acceptance Criteria:
     - Tampilan sesuai desain dark theme
     - Hover effect sederhana
     - Semua elemen ada dan terlihat

4. **Bind Library Grid to All Games**
   - Deskripsi: Buat MainWindowViewModel, inject GameService, load active games, bind ke grid. Gunakan ObservableCollection dan VirtualizingStackPanel di ItemsPanelTemplate.
   - Dependencies: Task 1, 2, 3 & Sprint 2
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua game aktif ditampilkan di grid "All Games"
     - Virtualization berfungsi (hanya render item yang terlihat)
     - UI responsif saat scrolling 200+ game
     - GameCard ditampilkan dengan benar

5. **Implement Real-time Search**
   - Deskripsi: Bind search box ke property di ViewModel, trigger filter setiap kali teks berubah (gunakan debounce sederhana 200ms untuk performance).
   - Dependencies: Task 1 & 4
   - Kompleksitas: M
   - Acceptance Criteria:
     - Search bekerja real-time
     - Debounce berfungsi (tidak filter setiap huruf langsung)
     - Grid update cepat saat search
     - Clear search menampilkan semua game kembali

**Estimasi**: 3-4 hari (M)

---

## Sprint 4 - Game Launch System

**Tujuan**: Membuat fitur launch game dengan validasi ketat (CCBoot-Ready!).

**Deliverables**:
- LaunchService dengan multi-step validation
- Game launch functionality
- Graceful error handling
- Launch logging

**Tasks**:
1. **Create LaunchService with Multi-Step Validation**
   - Deskripsi: Buat ILaunchService dan LaunchService dengan method ValidateAndLaunchGameAsync(GameDto game). Lakukan validasi:
     1. Resolve path (dari Sprint 2)
     2. Check file exists
     3. Check file size > 0
     4. Check file extension valid (.exe/.bat/.lnk)
     5. Check file readable
   - Dependencies: Sprint 2 & 3 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua validasi berjalan
     - Return validation result dengan pesan error jelas
     - Tidak crash jika validasi gagal

2. **Implement Game Launch (Process.Start)**
   - Deskripsi: Jika validasi lolos, jalankan executable dengan Process.Start(). Gunakan UseShellExecute=true untuk .lnk.
   - Dependencies: Task 1
   - Kompleksitas: S
   - Acceptance Criteria:
     - Game bisa di-launch dengan double-click atau Play button
     - Tidak ada error saat launch
     - Launcher tetap berjalan di background (opsional, tapi bagus)

3. **Create Launch History Recording**
   - Deskripsi: Tambahkan method di LaunchService untuk mencatat launch ke tabel launches. Simpan game_id dan launch_time.
   - Dependencies: Task 1 & Sprint 1
   - Kompleksitas: S
   - Acceptance Criteria:
     - Setiap launch dicatat di database
     - LaunchTime disimpan dengan benar
     - Tidak crash jika pencatatan gagal (log saja)

4. **Implement Graceful Error Handling UI**
   - Deskripsi: Buat ErrorDialog UserControl atau MessageBox khusus untuk menampilkan error launch dengan pesan jelas dan tips untuk operator.
   - Dependencies: Task 1 & 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Error ditampilkan dengan UI yang bagus
     - Pesan error jelas dan actionable
     - Tips untuk operator ditampilkan
     - Tidak ada crash launcher

**Estimasi**: 2-3 hari (M)

---

## Sprint 5 - Favorites & Recently Played

**Tujuan**: Membuat fitur favorites dan recently played.

**Deliverables**:
- FavoriteService
- Recently Played logic
- Favorites tab UI
- Recently Played tab UI
- Update last_played di games table

**Tasks**:
1. **Create FavoriteService**
   - Deskripsi: Buat IFavoriteService dan FavoriteService dengan method:
     - GetFavoritesAsync()
     - AddFavoriteAsync(int gameId)
     - RemoveFavoriteAsync(int gameId)
     - IsFavoriteAsync(int gameId)
   - Dependencies: Sprint 1 & 2 selesai
   - Kompleksitas: S
   - Acceptance Criteria:
     - Semua method berfungsi
     - Data disimpan ke tabel favorites
     - Handle duplicate entry (tidak crash)

2. **Bind Favorites Tab**
   - Deskripsi: Load favorit di ViewModel, bind ke tab Favorites. Gunakan GameCard yang sama.
   - Dependencies: Task 1 & Sprint 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Tab Favorites menampilkan game favorit
     - Toggle bintang di GameCard berfungsi menambah/hapus favorit
     - UI update real-time saat favorit berubah

3. **Create Recently Played Logic**
   - Deskripsi: Tambahkan method di LaunchService atau buat baru: GetRecentlyPlayedAsync(int count = 10). Ambil dari tabel launches, group by game_id, order by launch_time desc, take 10. Juga update last_played di games table setiap launch.
   - Dependencies: Sprint 4 Task 3 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Recently played diurutkan dengan benar
     - last_played di games table diupdate setiap launch
     - Hanya 10 game terakhir yang ditampilkan

4. **Bind Recently Played Tab**
   - Deskripsi: Load recently played di ViewModel, bind ke tab Recently Played.
   - Dependencies: Task 3 & Sprint 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Tab Recently Played menampilkan 10 game terakhir
     - Urutan benar (terbaru di atas)
     - Update real-time setelah launch game baru

**Estimasi**: 2 hari (S-M)

---

## Sprint 6 - Admin Modal (Password Protected)

**Tujuan**: Membuat admin modal untuk Add/Edit/Delete game, password protected.

**Deliverables**:
- AdminLoginDialog (password input)
- AdminModalWindow
- Game list di admin
- Add/Edit/Delete game forms
- Base path configuration

**Tasks**:
1. **Create Admin Login Dialog**
   - Deskripsi: Buat AdminLoginDialog.xaml dengan password box. Verifikasi password dengan hash di SettingsManager.
   - Dependencies: Sprint 1 Task 3 selesai
   - Kompleksitas: S
   - Acceptance Criteria:
     - Password di-hash saat verifikasi
     - Tidak bisa login dengan password salah
     - Login berhasil membuka AdminModal

2. **Create Admin Modal Window Layout**
   - Deskripsi: Buat AdminModalWindow.xaml dengan:
     - Tabs: Games, Settings, (Health Monitor nanti Sprint 7)
     - Games tab: List game, Add/Edit/Delete buttons
     - Settings tab: Base path configuration
   - Dependencies: Task 1 & Sprint 3
   - Kompleksitas: M
   - Acceptance Criteria:
     - Layout sesuai desain
     - Tab control berfungsi
     - Admin modal hanya bisa dibuka lewat login

3. **Implement Games CRUD di Admin**
   - Deskripsi: Buat AdminViewModel, inject GameService. Implement Add, Edit, Delete game.
     - Add/Edit form: Name, Genre, Relative EXE Path, Relative Install Path, Cover Path, Status
   - Dependencies: Task 2 & Sprint 2
   - Kompleksitas: M
   - Acceptance Criteria:
     - List game menampilkan semua game
     - Add game berfungsi dan menyimpan ke DB
     - Edit game berfungsi
     - Delete game berfungsi (dengan konfirmasi)
     - Status game bisa diubah (active/inactive)

4. **Implement Base Path Configuration**
   - Deskripsi: Di Settings tab admin, buat UI untuk menambah/edit/menghapus base paths. Simpan ke appsettings.json.
   - Dependencies: Task 2 & Sprint 1 Task 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Base paths ditampilkan di UI
     - Bisa menambah base path baru
     - Bisa menghapus base path
     - Perubahan disimpan ke appsettings.json

5. **Add Admin Button & Shortcut**
   - Deskripsi: Tambahkan Admin button di MainWindow top bar. Tambahkan shortcut Ctrl+Shift+A untuk membuka admin login (opsional tapi bagus).
   - Dependencies: Task 1 & Sprint 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Admin button terlihat di MainWindow
     - Klik button membuka login dialog
     - Shortcut Ctrl+Shift+A berfungsi (opsional tapi recommended)

**Estimasi**: 3-4 hari (M)

---

## Sprint 7 - Repair Library & Health Check

**Tujuan**: Membuat fitur repair library dan health check (CCBoot-Critical!).

**Deliverables**:
- HealthCheckService
- Repair Mode di Admin
- Health Monitor tab di Admin
- Auto health check for top games at startup

**Tasks**:
1. **Create HealthCheckService**
   - Deskripsi: Buat IHealthCheckService dan HealthCheckService dengan method:
     - CheckGameHealthAsync(GameDto game) → HealthStatus (Ok/Warning/Error) + message
     - CheckAllGamesAsync() → List<GameHealthResult>
   - Dependencies: Sprint 2 & 4 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - CheckGameHealth melakukan validasi sama seperti launch
     - Update health_status dan last_health_check di games table
     - Return pesan error jelas jika ada masalah
     - CheckAllGames bekerja untuk semua game

2. **Add Health Monitor Tab to Admin**
   - Deskripsi: Tambahkan tab Health Monitor di AdminModal. Tampilkan:
     - Ringkasan: Total OK, Warning, Error
     - List semua game dengan status health
     - Last health check time
     - Fail count
   - Dependencies: Task 1 & Sprint 6
   - Kompleksitas: M
   - Acceptance Criteria:
     - Ringkasan ditampilkan dengan benar
     - List game dengan status health
     - Warna berbeda untuk setiap status (hijau OK, kuning Warning, merah Error)
     - Bisa refresh health check manual

3. **Implement Repair Mode Features**
   - Deskripsi: Di Health Monitor tab, tambahkan tombol:
     - Scan All: Jalankan health check untuk semua game
     - Auto-Repair: Coba cari game di base paths dan update relative path jika ditemukan
     - Bulk Edit Base Path: Ubah base path untuk semua game (atau update base paths setting)
   - Dependencies: Task 1 & 2
   - Kompleksitas: L
   - Acceptance Criteria:
     - Scan All berfungsi dan update status semua game
     - Auto-Repair berhasil menemukan game yang path berubah
     - Bulk Edit Base Path berfungsi
     - Semua perubahan disimpan ke database

4. **Implement Quick Health Check at Startup**
   - Deskripsi: Saat startup launcher, jalankan quick health check untuk:
     - 10 game favorit teratas
     - 10 recently played
   - Jika ada yang error, tampilkan notifikasi ringkas di UI (tidak mengganggu user).
   - Dependencies: Task 1 & Sprint 3, 5
   - Kompleksitas: M
   - Acceptance Criteria:
     - Quick check berjalan di background (async, tidak block startup)
     - Notifikasi muncul jika ada game error
     - User tetap bisa menggunakan launcher tanpa terganggu

**Estimasi**: 3-4 hari (M-L)

---

## Sprint 8 - UI Polish & Performance Optimization

**Tujuan**: Polishing UI, dark theme, lazy load cover, cache strategy, dan performance tuning.

**Deliverables**:
- Dark theme final
- Lazy load cover images
- 3-level cache (RAM → Local Disk → Server)
- Performance tuning (RAM < 50MB idle, startup < 1s)
- Hover effects dan micro-interactions ringkas

**Tasks**:
1. **Apply Dark Theme Final**
   - Deskripsi: Buat ResourceDictionary dengan warna final (background #111111, surface #1A1A1A, accent #3B82F6, dll.). Terapkan ke semua kontrol.
   - Dependencies: Sprint 3 selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua UI menggunakan dark theme
     - Warna sesuai desain
     - Kontras bagus, teks mudah dibaca

2. **Implement Lazy Load Cover Images**
   - Deskripsi: Di GameCard, gunakan async/await untuk load cover. Gunakan placeholder saat loading. Hanya load cover jika GameCard terlihat di viewport.
   - Dependencies: Sprint 3 & 1 Task 4
   - Kompleksitas: M
   - Acceptance Criteria:
     - Cover dimuat secara async (tidak block UI)
     - Placeholder ditampilkan saat loading
     - Hanya load cover yang terlihat
     - Tidak ada lag saat scrolling banyak game

3. **Implement 3-Level Cache Strategy**
   - Deskripsi: Update ImageCacheService untuk:
     1. RAM Cache: Simpan BitmapImage di MemoryCache untuk cover yang sedang ditampilkan
     2. Local Disk Cache: Simpan thumbnail 256px di %LOCALAPPDATA%\GameHub\Cache
     3. Server Source: Baca dari server jika tidak ada di cache
   - Dependencies: Task 2 & Sprint 1 Task 4
   - Kompleksitas: L
   - Acceptance Criteria:
     - Cache berfungsi (tidak download cover berulang kali)
     - Local disk cache dibuat dan diakses cepat
     - RAM cache di-evict jika tidak digunakan
     - Max cache size 100MB (auto delete file lama)

4. **Performance Tuning**
   - Deskripsi: Optimize:
     - Startup time: Load DB sync, lalu load cover async
     - RAM usage: Dispose BitmapImage dengan benar, virtualization optimal
     - Search performance: Pastikan < 50ms
   - Dependencies: Semua sprint sebelumnya selesai
   - Kompleksitas: L
   - Acceptance Criteria:
     - Startup time < 1 detik
     - RAM idle < 50MB
     - Search < 50ms untuk 200 game
     - Scroll lancar tanpa lag
     - Tidak ada memory leak

5. **Add Simple Hover Effects & Micro-interactions**
   - Deskripsi: Tambahkan hover effect ringkas:
     - GameCard sedikit scale (1.02x) saat hover
     - Play button highlight saat hover
     - Warna bintang berubah saat toggle favorite
   - Dependencies: Sprint 3
   - Kompleksitas: S
   - Acceptance Criteria:
     - Efek smooth, tidak lag
     - Tidak ada efek berat (blur, particle, dll.)
     - Semua interaksi terasa responsif

**Estimasi**: 3-5 hari (L)

---

## Sprint 9 - Testing, Bug Fixes & MVP Release

**Tujuan**: Testing menyeluruh, bug fixes, final check, dan release MVP.

**Deliverables**:
- Tested MVP build
- Bug fixes semua critical issues
- Final checklist selesai
- Release-ready executable

**Tasks**:
1. **Unit Testing Core Services**
   - Deskripsi: Buat unit test untuk GameService, SearchService, LaunchService, FavoriteService, HealthCheckService.
   - Dependencies: Semua sprint sebelumnya selesai
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua service punya unit test
     - Coverage > 70% untuk logic penting
     - Semua test passing

2. **Manual Testing & Bug Bash**
   - Deskripsi: Testing manual menyeluruh:
     - Semua fitur MVP berfungsi
     - CCBoot-specific features (relative path, repair, health check)
     - Performance (RAM, startup, search)
     - UI di berbagai resolusi
     - Error scenarios (path salah, file corrupt, dll.)
   - Dependencies: Task 1
   - Kompleksitas: L
   - Acceptance Criteria:
     - Semua critical bug ditemukan dan dicatat
     - Tidak ada crash dalam penggunaan normal
     - Semua fitur MVP berjalan sesuai desain

3. **Fix All Critical & Major Bugs**
   - Deskripsi: Perbaiki semua bug critical dan major yang ditemukan saat testing.
   - Dependencies: Task 2
   - Kompleksitas: Tergantung bug, tapi estimasi M-L
   - Acceptance Criteria:
     - Tidak ada critical bug tersisa
     - Tidak ada major bug yang mengganggu penggunaan
     - Minor bug boleh ditunda ke v1.5

4. **Final CCBoot Deployment Checklist**
   - Deskripsi: Jalankan semua checklist di CCBOOT_OPTIMIZATION_GUIDE.md:
     - Launcher di folder read-only server
     - Database di folder writeable
     - Covers pre-generated WebP
     - Base path dikonfigurasi
     - Semua game di-scan
     - Shortcut di desktop
     - Auto-start billing (opsional)
     - Lockdown mode (opsional)
     - Test 5 client bersama
     - Test 10 game berbeda
   - Dependencies: Task 3
   - Kompleksitas: M
   - Acceptance Criteria:
     - Semua checklist item selesai dan dicentang
     - Test deployment di environment simulasi CCBoot berhasil

5. **Build Release & Package**
   - Deskripsi: Build release mode (x64, Release), package ke folder release dengan:
     - GameHub.exe
     - Semua DLL dependencies
     - Assets folder (contoh cover, icon)
     - Database kosong atau sample
     - appsettings.json default
     - README sederhana untuk operator
   - Dependencies: Task 4
   - Kompleksitas: S
   - Acceptance Criteria:
     - Build release tanpa error
     - Semua file dependencies ada
     - Executable bisa berjalan di PC Windows 10/11 tanpa install .NET 8 (bisa gunakan Self-Contained jika perlu, tapi operator mungkin lebih suka Framework-dependent)

**Estimasi**: 3-5 hari (M-L)

---

## Risiko Implementasi & Mitigasi

| Risiko | Dampak | Probabilitas | Mitigasi |
|--------|--------|--------------|----------|
| EF Core performance kurang bagus | Startup lambat | M | Gunakan NoTracking untuk query read-only, optimize index, bisa gunakan Dapper jika benar-benar perlu |
| Virtualization tidak berjalan dengan benar | RAM tinggi, lag | M | Test dengan 200+ game sejak awal, pastikan ItemsPanelTemplate benar |
| Path resolution logic gagal di CCBoot | Game tidak bisa di-launch | H | Test banyak skenario path sejak Sprint 2, buat unit test untuk path resolver |
| Cache memory leak | RAM terus bertambah | M | Gunakan using pattern untuk BitmapImage, evict cache secara berkala, test dengan long-running session |
| Time management sprint tidak sesuai | Telat rilis | M | Prioritaskan fitur P0 terlebih dahulu, bisa tunda fitur minor (shortcut, dll.) ke v1.5 jika perlu |

---

## Urutan Pengerjaan Paling Aman (Priority-Based)
1. **P0**: Semua task di Sprint 0-4 (Core: Setup, DB, Models, Search, Launch)
2. **P0**: Sprint 7 Task 1-3 (Health Check & Repair, CCBoot-Critical!)
3. **P0**: Sprint 5 (Favorites & Recently Played)
4. **P0**: Sprint 6 (Admin Modal)
5. **P1**: Sprint 7 Task 4 (Quick check startup)
6. **P1**: Sprint 8 (UI Polish & Performance)
7. **P1**: Sprint 9 (Testing & Release)

---

## Kesimpulan
Implementation plan ini **terstruktur, jelas, dan bisa dieksekusi langkah demi langkah**. Setiap task memiliki dependencies, acceptance criteria, dan estimasi yang jelas. Fokus selalu pada **MVP 1.0, CCBoot-readiness, dan stabilitas**.
