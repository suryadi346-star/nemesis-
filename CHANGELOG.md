# Changelog

Semua perubahan signifikan pada project ini didokumentasikan di file ini.
Format mengikuti [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [2.0.1] — 2026-04-30

### Added

- **Anomaly Analysis Layer** — fitur utama rilis ini
  - `GET /api/anomaly/top` — top risky packages nasional, sortir by `risk_score DESC`
    - Filter: `ownerType`, `severity`, `mencurigakan`, `pemborosan`, `priorityOnly`, `limit`
  - `GET /api/anomaly/owners/summary` — agregat anomali per owner (mencurigakan count, pemborosan count, avg/max risk_score) + top packages
  - `GET /api/anomaly/severity` — distribusi severity nasional (low/med/high/absurd)
  - `GET /api/anomaly/methods` — breakdown metode pengadaan dengan anomali count, berguna deteksi dominasi penunjukan langsung
- `src/backend/repositories/anomaly-repository.js` — query layer ESM dengan prepared statement cache via WeakMap
- `src/backend/repositories/anomaly-repository.test.js` — 22 integration tests (node --test, in-memory SQLite), semua pass
- `src/backend/anomaly-engine.js` — risk scoring engine
- `src/frontend/assets/js/anomaly-widget.js` — widget frontend untuk visualisasi anomali
- `docs/anomaly-api.md` — dokumentasi lengkap 4 endpoint anomali beserta query params dan contoh response
- `.github/ISSUE_TEMPLATE/bug_report.md` — template laporan bug
- `.github/ISSUE_TEMPLATE/data_anomaly.md` — template laporan anomali data pengadaan

### Changed

- `src/backend/app.js` — tambah 4 route anomali, import `anomaly-repository.js`
- Security middleware stack: Helmet (CSP), HPP, rate limiting (100 req/15min per IP) sudah aktif di semua route `/api`

### Security

- Semua input query divalidasi via whitelist sebelum masuk ke prepared statement
- Limit di-cap: `getTopRiskyPackages` max 200, `getOwnerAnomalySummary` max 100

---

## [2.0.0] — 2026 (upstream)

### Added

- Migrasi ke single-worker architecture (`worker.js`) — satu proses menangani Express + Vite dev server
- Vite 6 sebagai bundler frontend (replace setup lama)
- Preact sebagai framework frontend
- MapLibre GL untuk visualisasi peta interaktif
- Morgan dengan rotating file stream — access log dan error log terpisah, rotasi harian, kompresi gzip
- Helmet, HPP, express-rate-limit sebagai security middleware
- Responsive layout untuk semua ukuran layar termasuk mobile

### Changed

- Seluruh codebase migrasi dari CJS ke ESM (`"type": "module"`)
- Arsitektur backend: flat files → `repositories/` + `services/` separation
- Frontend dipindahkan ke `src/frontend/` (dari `frontend/` root)

### Fixed

- `NODE_ENV` restriction compatibility dengan Vite
- Cross-platform stability untuk log handling
- Gzip handling pada rotating log stream
- SQL bug pada beberapa query dashboard
- Mobile responsive layout

---

## [1.x] — sebelumnya

Versi awal — dashboard pengadaan publik berbasis Express + SQLite tanpa bundler modern.
Riwayat lengkap tersedia via `git log`.
