# Smart Waste Management Simulator (SWMS)

Smart Waste Management Simulator (SWMS) is a full-stack planning and decision-support platform for municipal solid-waste management. It combines structured habitation data, configurable municipal parameters, GIS layers, long-term waste simulations, scenario analysis, and operational reporting in a single web application.

The system is designed to help municipalities explore waste-generation trends, evaluate infrastructure and collection strategies, compare disruption scenarios, and make better-informed planning decisions.

## Key Capabilities

### Municipal Planning

- Municipality-scoped habitation and ward management
- Centralized records for demographic, infrastructure, industrial, environmental, economic, terrain, and cultural parameters
- Baseline parameter records automatically created for new habitations
- Municipality-level data isolation enforced by the API

### Data Import and Validation

- CSV and XLSX template generation
- Authenticated template downloads
- Asynchronous upload validation using background workers
- Row-level validation issues and correction notes
- Type coercion and schema validation with Zod
- Valid-row insertion and upsert into the appropriate parameter tables

### GIS and Mapping

- GeoJSON and Shapefile upload support
- PostGIS geometry storage with SRID 4326
- GIS layer processing and status tracking
- Map overlays for habitations and uploaded layers

### Simulation and Analysis

- Twenty-year waste-management projections
- Historical reconstruction, current base-year data, and future projections
- Flood, heavy-monsoon, and road-blockage stress scenarios
- Sensitivity analysis for population, growth, and industrial-waste changes
- Budget estimates and operational projections
- Route optimization using a greedy nearest-neighbor TSP/VRP heuristic
- CSV simulation report downloads

### User and Administration

- JWT-based authentication
- Access and refresh token support
- Role-based access control
- Admin user activation and management
- Password reset tokens and reset workflows
- Live account-status refresh
- Simulation-aware Ask SWMS assistant

## Technology Stack

| Layer | Technologies |
| --- | --- |
| Frontend | React, Vite, TypeScript, Leaflet, Lucide React |
| Backend | Node.js, Express, TypeScript |
| Database | PostgreSQL with PostGIS |
| Queue and workers | Redis and BullMQ |
| Validation | Zod |
| File processing | CSV Parse, ExcelJS, Multer, Shapefile |
| Authentication | JWT and bcryptjs |
| Security and middleware | Helmet, CORS, Morgan, dotenv |
| Testing | Jest, ts-jest, Supertest |

## AI-Assisted Development

This project was developed with assistance from **Antigravity**, an AI-assisted coding and development platform. AI assistance was used to support implementation, documentation, problem-solving, and development workflow improvements.

All generated code and recommendations should be reviewed, tested, and validated by the project maintainers before use in production environments.

## Architecture Overview

SWMS follows a client-server architecture:

```text
┌────────────────────┐
│ React + Vite UI    │
│ Dashboard, GIS,    │
│ uploads, reports   │
└─────────┬──────────┘
          │ HTTP / JSON
          ▼
┌────────────────────┐       ┌────────────────────┐
│ Express API        │──────▶│ PostgreSQL         │
│ Auth, simulation,  │       │ + PostGIS          │
│ uploads, GIS       │       │ relational + GIS   │
└─────────┬──────────┘       └────────────────────┘
          │
          ▼
┌────────────────────┐       ┌────────────────────┐
│ BullMQ workers     │◀──────▶│ Redis              │
│ Validation and GIS │       │ job queue          │
│ processing         │       └────────────────────┘
└────────────────────┘
```

## Requirements

Before running the application locally, install:

- **Node.js 20 or newer**
- **npm**
- **PostgreSQL** with the **PostGIS** extension
- **Redis**
- **Git**

Verify the Node.js and npm installations:

```bash
node --version
npm --version
```

Verify PostgreSQL and Redis are available through their respective command-line tools or services.

## Run Locally

### 1. Clone the repository

```bash
git clone https://github.com/yukthasmayya8/swms-agriInnovators.git
cd swms-agriInnovators
```

### 2. Install dependencies

```bash
npm install
```

### 3. Create the environment file

Create a `.env` file in the repository root:

```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/swms
REDIS_URL=redis://localhost:6379
JWT_ACCESS_SECRET=replace-with-a-long-random-secret
JWT_REFRESH_SECRET=replace-with-a-different-long-random-secret
PORT=4000
```

Use strong, unique secrets for any environment beyond local development. Do not commit `.env` to version control.

### 4. Prepare the database

Make sure PostgreSQL is running, the `swms` database exists, and PostGIS is enabled. Then run the migrations:

```bash
npm run migrate
```

Seed the database with deterministic development data:

```bash
npm run seed
```

The seed process creates development users, municipalities, wards, parameter records, and GIS/upload examples. Seed credentials are intended only for local development.

### 5. Start the application

Run the API, background worker, and frontend in separate terminals from the repository root.

**Terminal 1 — API:**

```bash
npm run dev:api
```

**Terminal 2 — background worker:**

```bash
npm run dev:worker
```

**Terminal 3 — frontend:**

```bash
npm run dev
```

The API is available at:

```text
http://localhost:4000
```

Vite normally serves the frontend at:

```text
http://localhost:5173
```

Open the frontend URL printed in the terminal if Vite selects a different port.

> When the API is launched through `src/server.ts`, validation and GIS workers may also be started by the server. Running `npm run dev:worker` separately remains supported for local development and queue isolation.

## Available Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Starts the Vite frontend development server. |
| `npm run dev:api` | Starts the API in TypeScript watch mode. |
| `npm run dev:worker` | Starts the background validation and GIS worker in watch mode. |
| `npm run migrate` | Applies database migrations. |
| `npm run seed` | Loads deterministic development data. |
| `npm run build` | Builds the frontend for production. |
| `npm run build:api` | Compiles the API for production. |
| `npm run preview` | Previews the production frontend build. |
| `npm run start:api` | Starts the API entry point. |
| `npm test` | Runs the Jest test suite. |
| `npm run smoke` | Runs the PowerShell smoke-test workflow. |
| `npm run smoke:request` | Runs the request-based smoke test. |

## Production Checks

Run the following commands before deployment:

```bash
npm run build:api
npm run build
npm test -- --runInBand
```

Integration tests use `TEST_DATABASE_URL` when it is provided. Otherwise, they use a test database derived from `DATABASE_URL`. Apply the migrations to the database used by the tests before running the suite.

## Main API Flows

Protected routes require an access token in the following format:

```http
Authorization: Bearer <access-token>
```

| Flow | Endpoint |
| --- | --- |
| Login and token refresh | `POST /api/auth/login`, `POST /api/auth/refresh` |
| Password reset | `POST /api/auth/forgot-password`, `POST /api/auth/reset-password` |
| Current account | `GET /api/auth/me` |
| Habitations | `GET/POST /api/habitations`, `GET/PATCH/DELETE /api/habitations/:id` |
| Parameters | `GET/PUT /api/habitations/:id/parameters/:category` |
| Simulation | `GET /api/habitations/:id/simulation` |
| Stress scenarios | `?scenario=normal`, `flood`, `heavy_monsoon`, or `road_blockage` |
| Sensitivity analysis | `GET /api/habitations/:id/simulation/sensitivity` |
| Budget estimates | `GET /api/habitations/:id/simulation/budget` |
| SWMS assistant | `POST /api/habitations/:id/simulation/ask` |
| Uploads | `GET /api/uploads`, `POST /api/uploads`, `GET /api/uploads/:id/issues` |
| Validation correction | `PATCH /api/uploads/:batchId/issues/:issueId` |
| GIS layers | `POST/GET /api/habitations/:id/map-layers` |
| Administration | `GET/POST /api/users`, `PATCH /api/users/:id` |

## CSV/XLSX Import Workflow

1. Open **Data Upload** and select a ward and parameter category.
2. Download the authenticated category template.
3. Keep the `habitation_id` column.
4. Replace `PASTE-WARD-UUID-HERE` with the selected ward UUID.
5. Enter values using the exact column headers provided by the template.
6. Upload the completed CSV or XLSX file.
7. Review asynchronous validation results and row-level issues.
8. Add correction notes where required.
9. Valid records are inserted or upserted into the corresponding parameter table.

All rows in one upload must use the selected ward UUID. This prevents an edited file from modifying records belonging to another ward.

## Algorithms and Planning Models

### Population Projection

For year `t`, the simulation uses compound population growth:

```text
P(t) = P(0) × (1 + r)^t
```

The stored percentage growth rate is converted to a decimal before calculation. Historical population is reconstructed using the inverse growth factor.

### Waste Generation

Residential waste is calculated using effective population, floating population, and a per-capita waste-generation benchmark. Industrial waste is then added using the stored industrial parameters.

### Scenario Analysis

Stress scenarios apply explicit collection-disruption multipliers to baseline coverage. The simulator generates year-by-year waste, collection, treatment, disposal, and vehicle projections for each scenario.

### Sensitivity Analysis

The server reruns the simulation using controlled changes to population, population growth, and industrial-waste inputs. It compares peak daily waste, total projected waste, and landfill requirements against the baseline.

### Route Optimization

The reusable route utility applies a greedy nearest-neighbor TSP/VRP heuristic with Haversine distance, vehicle-capacity checks, depot returns, total distance, estimated time, and utilization calculations.

### Validation and Asynchronous Processing

CSV/XLSX files are parsed, coerced into typed values, validated with Zod schemas, and processed by BullMQ/Redis workers. Database updates use transactions and PostgreSQL `ON CONFLICT` upserts.

### GIS Processing

Uploaded GeoJSON and Shapefile geometries are converted to GeoJSON and stored in PostGIS with SRID 4326. The frontend displays stored geometry and layer status using Leaflet.

## Security Considerations

- Keep `.env` files and secrets out of version control.
- Use long, random JWT access and refresh secrets.
- Do not use seeded development credentials in production.
- Review authentication and role-based access rules before deployment.
- Preserve municipality-level predicates on every protected data query.
- Validate uploaded files by type, size, schema, and ownership.
- Use HTTPS and secure deployment configuration in production.
- Restrict PostgreSQL and Redis network access to trusted services.
- Review firewalls, CORS, and allowed origins before exposing the API publicly.

## Repository Structure

```text
swms-agriInnovators/
├── src/
│   ├── server.ts          # Express API entry point
│   ├── worker.ts          # Background worker entry point
│   ├── run.ts             # API runtime entry point
│   └── ...                # Frontend, API, simulation, GIS, and shared modules
├── scripts/               # Database, smoke-test, and utility scripts
├── migrations/            # PostgreSQL database migrations
├── public/                # Static frontend assets
├── package.json           # Dependencies and npm scripts
├── tsconfig.json          # TypeScript configuration
├── tsconfig.build.json    # Production API build configuration
├── .env.example           # Environment variable template, if present
└── README.md              # Project documentation
```

## Development Status

SWMS is an actively developed project and should be treated as a planning and demonstration system unless it has been reviewed and hardened for a production deployment.

Before production use, perform application security testing, load testing, database backup planning, operational monitoring, and validation with real municipal requirements.

## License

No formal open-source license has been specified for this repository. Contact the repository owner before redistributing, modifying, or using the project beyond personal, academic, or demonstration purposes.
