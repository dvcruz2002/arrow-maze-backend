# Release Guide

## Prerequisites

- Node.js 22+
- npm 10+
- PostgreSQL 16 (local or Docker)
- GitHub repository with Actions enabled

## Environment variables

Copy `.env.example` to `.env`. Fill in all required values. Never commit `.env`.

```
DATABASE_URL=postgresql://user:pass@host:5432/arrow_maze
JWT_SECRET=<strong-random-secret>
PORT=3000
```

## Database setup

Apply migrations in order before first run:

```bash
psql $DATABASE_URL -f src/infrastructure/database/migrations/001_create_users.sql
psql $DATABASE_URL -f src/infrastructure/database/migrations/002_create_leaderboard.sql
psql $DATABASE_URL -f src/infrastructure/database/migrations/003_create_player_progress.sql
```

## Local run

```bash
npm install
npm run dev
```

## Verify before release

```bash
npm run verify   # lint + typecheck + test:coverage + build
```

All checks must pass before opening a release PR.

## Docker

```bash
cp .env.example .env
# fill in .env
docker compose up --build
```

## Production deployment

1. Confirm `npm run verify` passes on the release branch.
2. Apply any new migrations on the production database.
3. Build the image: `docker build -t arrow-maze-backend .`
4. Set production environment variables (never in source).
5. Run the image: `docker run -p 3000:3000 --env-file .env arrow-maze-backend`

## Versioning

Follows semver. Tag releases as `v<major>.<minor>.<patch>` on the `main` branch.

```bash
git tag v1.0.0
git push origin v1.0.0
```

## CI/CD

Every PR runs: install → lint → typecheck → test:coverage → build via GitHub Actions (`.github/workflows/`).
