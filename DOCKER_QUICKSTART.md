# Docker Quickstart Guide

## Prerequisites

- Docker and Docker Compose installed
- `.env` file with `BETTER_AUTH_SECRET` (already configured)

## Quick Start

1. Start the services:

```bash
docker compose up -d
```

2. Check the logs:

```bash
docker compose logs -f paperclip
```

3. Access Paperclip:

Open your browser to: **http://localhost:3100**

## Useful Commands

```bash
# Stop services
docker compose down

# Stop and remove volumes (fresh start)
docker compose down -v

# Rebuild after code changes
docker compose up -d --build

# View logs
docker compose logs -f

# View only Paperclip logs
docker compose logs -f paperclip

# View only PostgreSQL logs
docker compose logs -f postgres

# Check service status
docker compose ps
```

## Configuration

The docker-compose.yml includes:

- **PostgreSQL 17** - Database on port 5432
- **Paperclip** - Application on port 3100
- **Persistent volumes** - Data survives container restarts

### Optional: Add API Keys

To enable local adapters (Claude, Codex), add to your `.env` file:

```env
OPENAI_API_KEY=your-openai-key
ANTHROPIC_API_KEY=your-anthropic-key
```

Then restart:

```bash
docker compose down
docker compose up -d
```

## Troubleshooting

### Port already in use

If port 3100 or 5432 is already in use, modify the ports in `docker-compose.yml`:

```yaml
ports:
  - "3200:3100" # Change host port (left side)
```

### Database connection issues

Wait for PostgreSQL to be healthy:

```bash
docker compose logs postgres
```

Look for: `database system is ready to accept connections`

### Fresh start

Remove all data and start over:

```bash
docker compose down -v
docker compose up -d
```
