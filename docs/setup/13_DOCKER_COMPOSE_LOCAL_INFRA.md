# Docker Compose Local Infrastructure

Goal: one command starts local dependencies.

## 1. Create `compose.yaml`

Initial services:

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: buildora
      POSTGRES_USER: buildora
      POSTGRES_PASSWORD: local-development-only
    ports:
      - "5432:5432"
    volumes:
      - buildora_postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U buildora -d buildora"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - buildora_redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  buildora_postgres:
  buildora_redis:
```

Pin exact image versions later in release-oriented infrastructure.

## 2. Start

```bash
docker compose up -d
```

## 3. Check

```bash
docker compose ps
docker compose logs postgres
docker compose logs redis
```

## 4. Test DB

```bash
docker compose exec postgres psql -U buildora -d buildora -c "SELECT NOW();"
```

## 5. Test Redis

```bash
docker compose exec redis redis-cli ping
```

## 6. Stop

```bash
docker compose down
```

Data remains because named volumes remain.

## 7. Full local reset

Destructive:

```bash
docker compose down -v
docker compose up -d
```

Then rerun migrations/seeds.

## 8. Never bake secrets

Keep `.env` out of Docker images and Git.

## 9. Temporal

For a solo macOS setup, use the Temporal CLI separately at first rather than putting a full Temporal stack into Compose.

This keeps Compose simple.
