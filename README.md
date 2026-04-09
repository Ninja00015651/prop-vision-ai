# PropVision.AI

**AI-Powered Real Estate Visualization & Intelligence Platform for Uzbekistan**

> A B2B platform that transforms flat 2D property listings into immersive, data-rich experiences — interactive maps, AI natural language search, comfort analytics, 3D property models, and video walkthroughs, built for the Tashkent real estate market.

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Nginx  (ports 50080 / 50443)             │
├─────────────────────────────┬────────────────────────────────┤
│  React 18 + TypeScript      │  FastAPI  (Python 3.11)        │
│  Mapbox GL JS v3            │  PostgreSQL 16 + PostGIS 3.4   │
│  Three.js / R3F             │  Redis 7                       │
│  Recharts + Framer Motion   │  OpenAI GPT-4o-mini            │
│  port 53000                 │  Meshy AI (image-to-3D)        │
│                             │  Luma AI / Pollinations (video)│
│                             │  port 58000                    │
└─────────────────────────────┴────────────────────────────────┘
```

## Features

| Feature | Description |
|---------|-------------|
| **Interactive Map** | Dark-themed Mapbox GL JS map with 3D buildings, property price markers, and comfort heatmap overlay |
| **AI Search** | Natural language search in English, Russian, and Uzbek via GPT-4o-mini with a 200+ term RAG glossary |
| **Comfort Analytics** | 7-axis livability scores (transport, shopping, education, green space, safety, healthcare, entertainment) from OSM + Google Places POI data |
| **3D Property Viewer** | Photo-to-3D model pipeline via Meshy AI, rendered in-browser with Three.js / React Three Fiber |
| **Video Walkthroughs** | AI-generated video tours from property images via Luma AI (Ray-2) or Pollinations |
| **Uybor.uz Sync** | Daily background sync of live listings from the Uybor.uz marketplace (04:00 UTC cron) |
| **Authentication** | JWT-based auth with role-aware API key management |
| **Admin Panel** | Dashboard for managing users, API keys, and monitoring platform usage |
| **Embeddable Widget** | 2-line JavaScript embed (`public/widget.js`) for partner platforms |

## Quick Start

### Prerequisites

- Docker and Docker Compose
- API keys (see [Environment Variables](#environment-variables))

### 1. Configure environment

```bash
cp backend/.env.example backend/.env
# Fill in your API keys in backend/.env
```

### 2. Start services

```bash
docker-compose up -d --build
```

Check container health:

```bash
docker-compose ps
```

### 3. Initialize database and seed data

```bash
docker exec -it propvision-api bash scripts/setup_data.sh
```

This single script handles:
1. **Migrations** — applies all Alembic schema migrations
2. **POI seed** — fetches real-world points of interest (metro, parks, hospitals, schools) from OpenStreetMap for the Tashkent metro area
3. **Marketplace sync** — pulls the latest listings from Uybor.uz

### Service URLs

| Service | URL |
|---------|-----|
| Frontend | http://localhost:53000 |
| Nginx proxy | http://localhost:50080 |
| API health | http://localhost:50080/health |
| API docs (Swagger) | http://localhost:58000/api/v1/docs |
| API docs (ReDoc) | http://localhost:58000/api/v1/redoc |
| PostgreSQL | localhost:55433 |
| Redis | localhost:56379 |

## Project Structure

```
prop-vision-ai/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── routes/          # properties, search, comfort, reconstruction,
│   │   │   │                    #   analytics, video, auth, admin
│   │   │   ├── dependencies.py  # JWT auth + rate limiting
│   │   │   └── middleware.py    # Request logging
│   │   ├── models/              # SQLAlchemy ORM models
│   │   ├── schemas/             # Pydantic v2 request/response schemas
│   │   ├── services/
│   │   │   ├── ai_search_service.py
│   │   │   ├── auth_service.py
│   │   │   ├── comfort_service.py
│   │   │   ├── poi_fetcher.py
│   │   │   ├── property_service.py
│   │   │   ├── rag_context.py
│   │   │   ├── reconstruction_service.py
│   │   │   ├── uybor_scraper.py
│   │   │   └── video_walkthrough_service.py
│   │   ├── data/                # RAG glossary (200+ Uzbek real estate terms)
│   │   ├── tasks/               # APScheduler background tasks (Uybor daily sync)
│   │   ├── config.py            # Pydantic BaseSettings (all env vars)
│   │   ├── database.py          # Async SQLAlchemy engine + PostGIS init
│   │   └── main.py              # FastAPI app entry point
│   ├── migrations/              # Alembic migrations
│   ├── scripts/                 # setup_data.sh, seed helpers
│   ├── tests/                   # Pytest suite
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Admin/           # Admin dashboard
│   │   │   ├── Auth/            # Login / auth flows
│   │   │   ├── Comfort/         # ComfortRadar (Recharts)
│   │   │   ├── Dashboard/       # Analytics dashboard
│   │   │   ├── Map/             # MapView (Mapbox GL JS)
│   │   │   ├── Nav/             # Navigation
│   │   │   ├── Property/        # PropertyPanel, ThreeViewer
│   │   │   └── Search/          # AISearchBar, SearchResults
│   │   ├── contexts/            # React context providers
│   │   ├── hooks/               # React Query hooks
│   │   ├── types/               # TypeScript definitions
│   │   └── config.ts
│   ├── public/
│   │   └── widget.js            # Embeddable partner widget
│   ├── Dockerfile
│   └── package.json
├── infra/
│   └── nginx/                   # Reverse proxy config + SSL
├── docs/                        # Extended documentation
├── .github/workflows/ci.yml     # GitHub Actions CI
├── docker-compose.yml
├── openapi.yaml                 # OpenAPI 3.0 spec
└── README.md
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | PostgreSQL asyncpg connection string |
| `REDIS_URL` | Yes | Redis connection URL |
| `OPENAI_API_KEY` | Yes | GPT-4o-mini for AI search |
| `MAPBOX_ACCESS_TOKEN` | Yes | Map rendering (frontend + backend) |
| `GOOGLE_PLACES_API_KEY` | Yes | POI data for comfort scores |
| `MESHY_API_KEY` | Yes | Image-to-3D model generation (meshy.ai) |
| `LUMA_API_KEY` | No | Video walkthrough generation (lumalabs.ai) |
| `POLLINATIONS_API_KEY` | No | Alternative video generation |
| `AWS_ACCESS_KEY_ID` | Yes | S3-compatible storage for 3D models |
| `AWS_SECRET_ACCESS_KEY` | Yes | S3 credentials |
| `S3_BUCKET_NAME` | Yes | Bucket for GLB model uploads |
| `JWT_SECRET_KEY` | Yes | JWT signing key (min 32 chars) |
| `API_SECRET_KEY` | Yes | General signing/hashing secret |
| `CORS_ORIGINS` | Yes | Comma-separated allowed origins |
| `UYBOR_SYNC_ENABLED` | No | Enable/disable daily Uybor sync (default: true) |
| `RATE_LIMIT_PER_MINUTE` | No | API requests per key per minute (default: 100) |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, TypeScript 5, Vite |
| Map | Mapbox GL JS v3, react-map-gl |
| 3D Viewer | Three.js, @react-three/fiber |
| Charts | Recharts |
| Animation | Framer Motion |
| Backend | FastAPI, Python 3.11 |
| ORM | SQLAlchemy 2.0 (async), GeoAlchemy2 |
| Database | PostgreSQL 16 + PostGIS 3.4 |
| Cache | Redis 7 |
| AI Search | OpenAI GPT-4o-mini (structured output + RAG) |
| 3D Reconstruction | Meshy AI (image-to-3D) |
| Video Generation | Luma AI Ray-2, Pollinations (fallback) |
| POI Data | OpenStreetMap Overpass API, Google Places API |
| Marketplace | Uybor.uz API (daily sync) |
| Auth | JWT (HS256) |
| Deployment | Docker Compose, Nginx, AWS Lightsail |
| CI/CD | GitHub Actions |

## Testing

```bash
# Backend tests
docker-compose exec api pytest tests/ -v

# Frontend type checking
docker-compose exec frontend npx tsc --noEmit

# Frontend linting
docker-compose exec frontend npx eslint src/ --ext .ts,.tsx
```

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/ARCHITECTURE.md) | System diagrams, database schema |
| [API Documentation](docs/API_DOCUMENTATION.md) | Full endpoint reference |
| [Integration Guide](docs/INTEGRATION_GUIDE.md) | Partner widget & API integration |
| [Project Rationale](docs/PROJECT_RATIONALE.md) | Problem statement, market context |
| [Scaling Roadmap](docs/SCALING_ROADMAP.md) | Lightsail → EKS growth path |
| [Risk Register](docs/RISK_REGISTER.md) | Known risks and mitigations |
| [Setup Guide](SETUP_GUIDE.md) | Detailed setup walkthrough |
| [Admin Setup](ADMIN_SETUP.md) | Admin user and panel setup |

## License

Proprietary — © 2026 PropVision.AI. All rights reserved.
