# GameHub Launcher

## Product Requirement Document (PRD)

Version: 1.0
Target Platform: Windows 10 / Windows 11
Target Environment: Warnet Diskless (CCBoot)

---

# 1. Overview

GameHub Launcher adalah aplikasi launcher game modern yang dirancang khusus untuk lingkungan warnet diskless.

Fokus utama launcher adalah:

* Menemukan game dengan cepat
* Menjalankan game dalam 1 klik
* Tampilan modern dan menarik
* Penggunaan RAM rendah
* Startup sangat cepat
* Ramah terhadap lingkungan diskless
* Tidak menggantikan software billing

Launcher tidak berfungsi sebagai:

* Billing client
* Store game
* Browser web
* Social platform
* Chat platform
* Auto updater kompleks

---

# 2. Product Goals

## Primary Goals

* User dapat menemukan game dalam < 3 detik
* User dapat menjalankan game dalam 1 klik
* Launcher berjalan dengan RAM rendah
* Launcher tetap responsif pada PC spesifikasi rendah

## Secondary Goals

* Meningkatkan discoverability game
* Menampilkan game populer
* Menampilkan game terbaru
* Menampilkan rekomendasi game

---

# 3. Design Philosophy

Inspirasi:

* Steam Library
* Xbox App
* Riot Client
* Playnite

Prinsip:

* Fast First
* Search First
* Click Less
* Resource Efficient

---

# 4. Main Navigation

## Home

Menampilkan:

* Hero Banner
* Continue Playing
* Trending Games
* Recently Added
* Recommended Games

## Library

Menampilkan:

* Semua game
* Filter kategori
* Search
* Sort

## Favorites

Menampilkan game favorit user.

## Recently Played

Menampilkan histori game terakhir dimainkan.

---

# 5. Home Page Layout

```text
┌─────────────────────────────────────┐
│             HERO BANNER             │
└─────────────────────────────────────┘

Continue Playing
[Valorant] [Point Blank] [Roblox]

Trending Today
[Game] [Game] [Game]

Recently Added
[Game] [Game] [Game]

Recommended For You
[Game] [Game] [Game]
```

---

# 6. Library Layout

```text
┌─────────────────────────────────────┐
│ Search Game...                      │
└─────────────────────────────────────┘

Genre:
[FPS]
[MOBA]
[Racing]
[RPG]
[Horror]

Games Grid:

┌─────┐ ┌─────┐ ┌─────┐
│Game │ │Game │ │Game │
└─────┘ └─────┘ └─────┘
```

---

# 7. Game Detail Layout

```text
┌─────────────────────────────┐
│         GAME COVER          │
└─────────────────────────────┘

Game Name

Genre
Developer
Size

Description

[ PLAY ]
```

---

# 8. Core Features

## Smart Search

Search harus real-time.

Contoh:

Input:

valo

Output:

Valorant

Latency target:

< 50 ms

---

## Quick Launch

Double click game:

langsung menjalankan executable.

Tanpa splash screen tambahan.

---

## Hover Preview

Saat cursor berada di atas game:

Tampilkan:

* Cover
* Genre
* Size
* Launch Button

---

## Recently Played

Menyimpan:

* Last Played
* Play Count

Menampilkan game yang sering dimainkan.

---

## Favorites

User dapat:

* Menambahkan favorit
* Menghapus favorit

Disimpan lokal.

---

## Trending System

Menghitung:

launch_count

per hari.

Menampilkan:

Top 10 game paling sering dimainkan.

---

## Recommended Games

Menggunakan rule sederhana:

Jika user sering memainkan FPS:

Rekomendasikan FPS lain.

Tidak menggunakan AI online.

---

## Recently Added

Menampilkan game yang baru ditambahkan operator.

---

# 9. User Flow

## Launch Game

```text
Open Launcher

↓

Search Game

↓

Click Game

↓

Play
```

## Browse Game

```text
Open Launcher

↓

Library

↓

Select Genre

↓

Game Detail

↓

Play
```

---

# 10. Database Structure

SQLite

## games

```sql
id
name
genre
developer
description
cover_image
exe_path
install_path
size
status
date_added
```

## play_history

```sql
id
game_id
launch_time
duration
```

## favorites

```sql
id
game_id
```

## statistics

```sql
game_id
launch_count
last_launch
```

---

# 11. Folder Structure

```text
GameHubLauncher/

├── assets/
│   ├── covers/
│   ├── banners/
│   └── icons/
│
├── database/
│   └── launcher.db
│
├── games/
│
├── logs/
│
├── config/
│   └── settings.json
│
├── ui/
│
├── core/
│
├── cache/
│
└── GameHub.exe
```

---

# 12. Resource Targets

## Startup Time

Target:

< 1 second

## RAM Usage

Idle:

50 MB - 100 MB

Maximum:

150 MB

## CPU Usage

Idle:

< 1%

## Disk Usage

Cache:

< 100 MB

---

# 13. Diskless Optimization

## Avoid

* Web browser engine
* Electron
* Chromium embedded
* Background services

## Prefer

* Native UI
* Local assets
* SQLite
* Lazy loading

## Cover Images

Gunakan:

* WebP
* Thumbnail cache

Maksimal:

512x768

---

# 14. Recommended Technology Stack

## Option Comparison

### Electron

Pros:

* Cepat dikembangkan

Cons:

* RAM besar
* Startup lambat

Status:
Not Recommended

---

### PySide6

Pros:

* Cepat dikembangkan
* UI modern

Cons:

* Bundle besar

Status:
Good

---

### WPF

Pros:

* Native
* Stabil
* Cepat

Cons:

* Windows only

Status:
Very Good

---

### WinUI 3

Pros:

* Modern
* Native

Cons:

* Masih berkembang

Status:
Good

---

### Qt C++

Pros:

* Sangat cepat
* Sangat ringan

Cons:

* Development lebih kompleks

Status:
Best Performance

---

# 15. Visual Design

Theme:

Dark Modern

Colors:

Background:
#111111

Surface:
#1A1A1A

Border:
#2A2A2A

Accent:
#3B82F6

Text:
#FFFFFF

Secondary Text:
#AAAAAA

---

# 16. Animations

Allowed:

* Fade
* Opacity Transition
* Small Scale Hover

Not Allowed:

* Heavy Blur
* Complex Particle Effects
* Video Background
* Continuous Animations

---

# 17. Version Roadmap

## Version 1.0

* Library
* Search
* Favorites
* Recently Played
* Launch Game

## Version 1.5

* Trending Games
* Recently Added
* Statistics

## Version 2.0

* Recommendation Engine
* Operator Dashboard
* Game Health Status

## Version 3.0

* Cloud Sync
* Multi Branch Statistics
* Cross-Warnet Analytics

---

# 18. Success Metrics

Target:

* Search success < 3 seconds
* Launch success > 99%
* Startup < 1 second
* RAM < 100 MB idle

---

# Final Principle

Launcher exists for one purpose:

"Find game fast. Launch game instantly."

Everything else is secondary.
