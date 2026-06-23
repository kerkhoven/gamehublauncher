# 🎯 TUJUAN UTAMA TUGAS

Kamu harus melakukan analisis menyeluruh terhadap seluruh dokumen dan menghasilkan:

## 1. Gap Analysis (WAJIB)
Identifikasi:

- bagian yang belum konsisten antar dokumen
- fitur yang redundant atau bertabrakan
- missing requirement penting untuk sistem launcher diskless
- potensi over-engineering
- risiko performa (RAM, startup time, IO disk)
- risiko arsitektur (tight coupling, spaghetti design)

---

## 2. Critical Review (WAJIB)

Berikan penilaian objektif:

- Apakah MVP terlalu kompleks?
- Apakah ada fitur yang seharusnya DIHAPUS?
- Apakah ada fitur yang seharusnya DIPINDAHKAN ke fase 2 atau 3?
- Apakah API layer terlalu dini untuk MVP?
- Apakah admin panel terlalu berat untuk awal?

---

## 3. Architecture Improvement Proposal

Berikan:

- versi arsitektur yang lebih optimal untuk MVP diskless launcher
- rekomendasi simplifikasi layer
- apakah Clean Architecture terlalu berat untuk use case ini
- rekomendasi alternatif (misalnya modular monolith ringan)

---

## 4. Database Optimization Review

Analisa:

- apakah schema terlalu kompleks untuk MVP?
- apakah ada tabel yang bisa digabung?
- apakah indexing sudah relevan untuk usage warnet?
- apakah SQLite cukup atau overkill?
- apakah perlu event-based logging atau cukup simple append log?

---

## 5. UI/UX Validation

Tinjau:

- apakah UI terlalu “Steam-like” (over-engineered)?
- apakah search-first sudah benar implementasinya?
- apakah navigation terlalu banyak?
- apakah ada friction dalam flow "click → play"?

---

## 6. Performance Risk Analysis (CRITICAL)

Untuk lingkungan warnet diskless:

- RAM constraint (<100MB target)
- startup speed (<1s target)
- disk IO bottleneck (CCBoot caching)
- asset loading strategy

Identifikasi semua potensi bottleneck.

---

## 7. Feature Prioritization Correction

Re-evaluate seluruh fitur dan klasifikasikan:

### MUST HAVE (MVP)
### SHOULD HAVE (Phase 2)
### NICE TO HAVE (Phase 3)
### SHOULD BE REMOVED

Jika ada fitur yang terlalu “enterprise-like”, wajib diturunkan prioritasnya.

---

## 8. Simplified MVP Proposal (WAJIB OUTPUT)

Buat ulang versi FINAL MVP yang:

- lebih ringan
- lebih realistis
- lebih cepat diimplementasi
- tidak over-engineered
- cocok untuk warnet diskless real-world

Harus mencakup:

- minimal architecture
- minimal database
- minimal UI flow
- minimal feature set

---

## 9. Engineering Recommendations

Berikan:

- rekomendasi bahasa/framework terbaik (WPF, Qt, Tauri, etc.)
- alasan berdasarkan diskless environment
- strategi caching terbaik
- strategi game metadata loading
- launch strategy paling cepat

---

## 10. Final Output Format

Output harus berupa:

1. Executive Summary
2. Problem List
3. Architecture Review
4. Database Review
5. UI/UX Review
6. Performance Review
7. Revised MVP Proposal
8. Final Recommendation (VERY IMPORTANT)

---

# ⚠️ CONSTRAINT

- Fokus utama: SPEED, SIMPLICITY, RELIABILITY
- Jangan bias ke enterprise architecture
- Jangan over-engineer
- Jangan menambahkan fitur baru tanpa alasan kuat
- Semua keputusan harus mempertimbangkan lingkungan warnet diskless (CCBoot)

---

# 🎯 CONTEXT

GameHub Launcher adalah:

"A very fast game launcher for internet cafes where users only care about finding and launching games as fast as possible."

Not a platform. Not an ecosystem. Just a launcher.

---

Mulai dengan membaca dan memahami semua 10 dokumen terlebih dahulu sebelum memberikan analisis.