# Product Requirements Document (PRD - Final & Konsisten)

## Product Name
GameHub Launcher

## Purpose
Launcher game modern yang **cepat, ringkas, dan stabil** untuk lingkungan warnet diskless (CCBoot).

## Objectives

### Primary (MVP v1.0)
- User dapat mencari game dengan cepat (< 1 detik)
- User dapat menjalankan game dengan 1 klik (double-click)
- Launcher berjalan dengan RAM < 50MB idle
- Launcher startup dalam < 1 detik
- Launcher stabil dan jarang crash di warnet 24/7

### Secondary (v1.5+)
- Meningkatkan discoverability game
- Menyediakan analytics lokal untuk operator
- Mempermudah manajemen game

### Non Goals (Sampai Kapan Pun!)
- ❌ Billing System
- ❌ Auto Updater Game
- ❌ Game Store
- ❌ Social Platform
- ❌ Chat
- ❌ Browser Built-in

---

## Functional Requirements

### v1.0 (MVP)
| ID | Fitur | Deskripsi |
|----|-------|-----------|
| FR-101 | Search Game | Real-time filter nama game secara lokal |
| FR-102 | Library View | Tampilkan semua game dalam grid dengan cover |
| FR-103 | Launch Game | Double-click atau klik tombol Play → langsung jalan |
| FR-104 | Favorites | Toggle bintang di game card untuk favorit |
| FR-105 | Recently Played | Tampilkan 10 game terakhir dimainkan |
| FR-106 | Admin Modal | Password-protected untuk Add/Edit/Delete game |

### v1.5 (Enhanced)
| ID | Fitur | Deskripsi |
|----|-------|-----------|
| FR-151 | Trending Games | Top 10 game terbanyak diluncurkan 7 hari terakhir |
| FR-152 | Genre Filter | Filter game berdasarkan genre |
| FR-153 | Recently Added | Game baru di 30 hari terakhir |
| FR-154 | Basic Analytics | Lihat total launch per game |
| FR-155 | Backup/Restore | Backup dan restore database |

---

## Non-Functional Requirements

| Kategori | Target |
|----------|--------|
| Startup Time | < 1 detik |
| RAM Usage (Idle) | < 50 MB |
| RAM Usage (Max) | < 100 MB |
| CPU Usage (Idle) | < 1% |
| Search Latency | < 50 ms |
| Platform | Windows 10 1809+, Windows 11 |
| Availability | Offline First (100% offline) |

---

## Tech Stack (Final)
Lihat **01_PRODUCT_VISION.md** atau **ARSITEKTUR_FINAL.md** untuk detail lengkap.
