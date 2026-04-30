# Contributing to Nemesis

Nemesis adalah platform analisis pengadaan publik berbasis SQLite + Express + Preact. Panduan ini berlaku untuk semua kontributor.

---

## Stack

| Layer | Teknologi |
|---|---|
| Runtime | Node.js ≥ 22 LTS |
| Backend | Express 5, better-sqlite3 |
| Frontend | Preact, MapLibre GL |
| Bundler | Vite 6 |
| Module system | ESM (`"type": "module"`) — tidak ada CJS |
| Test runner | `node --test` (built-in, tanpa Jest/Vitest) |
| Linter | ESLint 9 |
| Formatter | Prettier |

---

## Setup

```bash
git clone https://github.com/suryadiarsyil-ops/nemesis
cd nemesis
npm install
cp .env.example .env   # sesuaikan path jika perlu
npm run dev
```

> **Termux / Android:** `npm install` butuh compiler untuk `better-sqlite3`.
> Jalankan `pkg install python make cmake` terlebih dahulu, lalu patch
> `~/.cache/node-gyp/<version>/include/node/common.gypi` — ganti
> `<(android_ndk_path)/sources/android/cpufeatures` ke `$PREFIX/include`.

---

## Struktur Direktori

```
nemesis/
├── src/
│   ├── backend/
│   │   ├── repositories/       # Query layer (better-sqlite3, sync)
│   │   ├── services/           # Business logic
│   │   ├── app.js              # Express app factory
│   │   ├── server.js           # HTTP server
│   │   ├── db.js               # Database connection
│   │   ├── config.js           # Environment config
│   │   ├── anomaly-engine.js   # Risk scoring engine
│   │   └── seed.js             # Database seeder
│   └── frontend/
│       ├── assets/
│       ├── App.jsx
│       ├── main.jsx
│       └── index.html
├── docs/                       # API documentation
├── scripts/                    # DB utilities (export, import, reset)
├── seed/geo/                   # GeoJSON fixtures
├── data/                       # SQLite database (gitignored)
├── worker.js                   # Entry point (morgan + vite + express)
└── .github/ISSUE_TEMPLATE/     # Bug report + data anomaly templates
```

---

## Konvensi Kode

### ESM Only

Semua file `.js` wajib ESM. Tidak ada `require()` atau `module.exports`.

```js
// ✅ benar
import { openDatabase } from './db.js';
export function getTopRiskyPackages(db, query) { ... }

// ❌ salah
const db = require('./db');
module.exports = { getTopRiskyPackages };
```

Ekstensi `.js` wajib disertakan di setiap import path.

### Repository Pattern

- Semua query SQL ditulis di `src/backend/repositories/`
- `db` selalu di-pass sebagai parameter, tidak ada singleton global
- Gunakan `better-sqlite3` sync API — tidak ada `async/await` di query layer
- Prepared statements di-cache per `db` instance via `WeakMap`

```js
const stmtCache = new WeakMap();

function getStmts(db) {
  if (stmtCache.has(db)) return stmtCache.get(db);
  const stmts = { myQuery: db.prepare(`SELECT ...`) };
  stmtCache.set(db, stmts);
  return stmts;
}
```

### Validasi Input

Selalu whitelist nilai yang valid sebelum masuk ke query:

```js
const VALID_OWNER_TYPES = ["kabkota", "provinsi", "central", "other"];
const ownerType = VALID_OWNER_TYPES.includes(query.ownerType) ? query.ownerType : "";
```

### Response Shape

Konsisten di semua endpoint:

```js
// List endpoint
{ data: [...], meta: { returned, filters } }

// Single owner endpoint
{ summary: {...}, topPackages: [...] }

// Distribution endpoint
{ data: [...] }
```

---

## Testing

Test runner: `node --test` (Node.js built-in, tidak perlu install apapun).

```bash
# Jalankan satu file test
node --test src/backend/repositories/anomaly-repository.test.js

# Jalankan semua test (recursive)
node --test src/**/*.test.js
```

### Menulis Test

- Gunakan in-memory SQLite (`new Database(":memory:")`) — tidak perlu file fixture
- Seed data minimal yang cukup untuk cover semua branch
- Struktur: `describe` → `test` → `assert`

```js
import { test, describe, before } from "node:test";
import assert from "node:assert/strict";
import Database from "better-sqlite3";
import { getTopRiskyPackages } from "./anomaly-repository.js";

let db;
before(() => { db = buildTestDb(); });

describe("getTopRiskyPackages", () => {
  test("sorted by risk_score DESC", () => {
    const { data } = getTopRiskyPackages(db, { limit: 10 });
    assert.ok(data.length > 0);
  });
});
```

---

## Git Workflow

```bash
# Branch dari main
git checkout -b feat/nama-fitur

# Commit convention (Conventional Commits)
git commit -m "feat: tambah endpoint /api/anomaly/top"
git commit -m "fix: filter pemborosan tidak bekerja saat limit kecil"
git commit -m "docs: update anomaly-api.md dengan contoh response"
git commit -m "test: tambah edge case ownerType invalid"

# Push dan buka PR ke main
git push origin feat/nama-fitur
```

### Prefix Commit

| Prefix | Kapan dipakai |
|---|---|
| `feat` | Fitur baru |
| `fix` | Bug fix |
| `docs` | Perubahan dokumentasi |
| `test` | Tambah atau perbaiki test |
| `refactor` | Refactor tanpa perubahan behavior |
| `chore` | Dependency update, config, tooling |

---

## Melaporkan Masalah

Gunakan template yang tersedia di `.github/ISSUE_TEMPLATE/`:

- **bug_report.md** — untuk bug umum (crash, wrong output, dll)
- **data_anomaly.md** — untuk anomali data pengadaan yang mencurigakan

---

## API Documentation

Semua endpoint baru wajib didokumentasikan di `docs/`. Lihat `docs/anomaly-api.md` sebagai referensi format.

---

## Environment Variables

| Variable | Default | Keterangan |
|---|---|---|
| `PORT` | `3000` | Port HTTP server |
| `CORS_ORIGIN` | `*` | Allowed origins (comma-separated) |
| `SQLITE_PATH` | `data/dashboard.sqlite` | Path ke database |
| `DATA_DIR` | `data/` | Direktori data |
| `AUDIT_DATASET_DIR` | `dataset/` | Direktori dataset audit |
| `AUDIT_DATASET_YEAR` | `2026` | Tahun dataset aktif |
| `GEO_ROOT_PATH` | `seed/geo/` | Root GeoJSON |
| `GEOJSON_PATH` | `seed/geo/03-districts` | Path GeoJSON kecamatan |
| `PROVINCE_GEOJSON_PATH` | `seed/geo/02-provinces/province-only` | Path GeoJSON provinsi |
| `NODE_ENV` | `development` | `development` atau `production` |
