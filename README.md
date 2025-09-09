<!-- README.md -->

# 🌊 Shaka — Surf Spots Platform (Frontend • API • Database)

**Shaka** is a full-stack, TypeScript-first platform for exploring surf spots. It includes:

* A cross-platform **Expo + React Native app** (iOS • Android • Web),
* A **NestJS API** that enriches surf spot data,
* A reusable **Prisma/MySQL data layer** published as an npm package.

The repo is organized as **git submodules**:

```
shaka/
├─ shaka_frontend/   # Expo + React Native app (iOS/Android/Web)
├─ shaka_backend/    # NestJS API (Prisma, Swagger, security, rate-limiting)
└─ shaka_database/   # Prisma schema + client + seed tooling (Packaged as "shakadb")
```

---

## ✨ Highlights

* **Cross-platform app** from one codebase (Expo 53, RN 0.79, React 19)
* **Native & Web maps** (react-native-maps on iOS/Android, Leaflet on Web)
* **Clean API** (NestJS 11, Prisma, Swagger docs, Pino logs, rate-limit)
* **Reusable DB package** (`shakadb`) with schema, seed helpers, and typed client
* **Type-safe everywhere** (TS strict), **robust tests**, and **CI pipelines**

---

## 🧩 Packages

### `shaka_frontend/` — Expo + React Native

* **Screens**: Home, Favorites, Detail, All Spots Map
* **State**: Redux Toolkit (`favorites` slice)
* **Maps**: `react-native-maps` (native) / `react-leaflet` (web)
* **DX**: Jest + RTL, Husky, ESLint Flat, Prettier (Tailwind plugin), Commitlint
* **Web export**: Expo “export” job in CI produces `/dist`

> Note: On web builds, the project renders Leaflet. The RN Maps plugin is disabled on web to avoid bundling native JSX in Node.

---

### `shaka_backend/` — NestJS API (Prisma/MySQL)

* **Endpoints**:

  * `GET /surfspot/all` — list enriched spots
  * `GET /surfspot/:id` — single enriched spot
  * `POST /surfspot` — validated create
  * `GET /healthz` — health check
* **Security**: Helmet, strict CORS, global rate-limit (memory/Redis)
* **Docs**: Swagger at `/docs` with typed DTOs
* **Logging**: `nestjs-pino` + request IDs
* **Testing**: Jest + Supertest (runs against real MySQL in CI)

---

### `shaka_database/` — Prisma package (`shakadb`)

* **Entities**: `SurfSpot`, `Photo → Thumbnail`, `Influencer`, `Traveller`, `SurfBreakType` + pivot tables
* **Usage** (any service):

  ```ts
  import prisma from "shakadb/lib/client";
  const spots = await prisma.surfSpot.findMany();
  ```
* **Tooling**: Seed helpers, purge utilities, ESM-friendly tests, Docker compose for local MySQL

---

## 🚀 Quick Start

### 1) Clone with submodules

```bash
git clone --recurse-submodules <this-repo-url>
# or, if already cloned:
git submodule update --init --recursive
```

### 2) Backing services (local MySQL option)

Use the DB package’s docker compose for a local MySQL:

```bash
cd shaka_database
docker compose up --build
# Seeds once and exits ("Done")
```

> `DATABASE_URL` for local compose is prewired internally as
> `mysql://root:shaka_root_pw@mysql:3306/shaka`.

### 3) API (NestJS)

```bash
cd ../shaka_backend
cp .env.sample .env          # set DATABASE_URL, FRONT_API_BASE_URL, etc.
npm ci
npm run start:dev            # http://localhost:3000, Swagger at /docs
```

### 4) Frontend (Expo)

```bash
cd ../shaka_frontend
cp .env.sample .env          # set EXPO_PUBLIC_API_BASE_URL to your API URL
npm ci
npm start                    # press a/i/w for Android/iOS/Web
```

---

## 🔌 Configuration (summary)

**Frontend**

* `EXPO_PUBLIC_API_BASE_URL` – API base URL
* (Android native maps) `GOOGLE_MAPS_API_KEY`

**Backend**

* `DATABASE_URL` – MySQL DSN
* `FRONT_API_BASE_URL` – CORS allowlist (comma-sep)
* `RATE_LIMIT_*`, `RATE_LIMIT_STORE`, `REDIS_URL` (optional)
* `LOG_LEVEL`

**Database package**

* `DATABASE_URL` for local usage or CI generation

---

## 🧪 Testing & CI

* **Frontend**: `jest-expo`, @testing-library/react-native, robust mocks (safe-area, status bar, maps, HTTP).
* **Backend**: Jest unit + E2E (Supertest) against a disposable MySQL instance.
* **Database**: ESM-friendly Jest with schema push and seed in global setup.

**CI (GitHub Actions)**

* Frontend: Lint → TS check → Jest (coverage) → **Expo web export artifact**
* Backend: Install → Prisma generate → Build → Unit/E2E → Lint
* Database: MySQL service → Prisma generate → Build → Tests → Lint

Artifacts:

* `web-dist` from the frontend export
* Coverage uploads on test jobs

---

## 🗺️ Data & Flow (high level)

1. **Frontend** calls:

   * `GET /surfspot/all` to list,
   * `GET /surfspot/:id` to view details and map location.
2. **Backend** uses Prisma against **ShakaDB**, enriches results (photo URLs, break types, influencers).
3. **Frontend** renders:

   * **Web**: Leaflet maps with OSM tiles,
   * **Native**: `react-native-maps` (+ Google Maps key on Android if configured).

---

## 📁 Useful NPM scripts (per package)

**Frontend**

* `npm run lint` • `npm test` • `npm run export`

**Backend**

* `npm test` • `npm run test:e2e` • `npm run test:cov` • `npm run start:dev`

**Database**

* `npm run prisma:db:push|seed|migrate:*|studio`
* `npm run test` • `npm run build` • `npm run db:cleanup`

---

## 🧭 Roadmap / Ideas

* Auth & user profiles (favorites per user)
* Offline caching & optimistic UI for the app
* Admin UI for data curation (photos, break types, influencers)
* Observability: tracing, metrics, dashboards
* CDN for photos/thumbnails

---

## License

MIT (or your preferred license). Add a `LICENSE` file if needed.
