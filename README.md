# Emercit Connector

Python project for collecting hydrometeorological measurements from the Emercit API, storing them in MongoDB, and exporting station time-series to CSV.

## Scope

This repository provides:

- API client logic for station metadata and measurements (`connector.py`)
- MongoDB persistence/query layer (`mongo.py`)
- Bulk historical ingestion orchestration (`provider.py`)
- CSV export pipeline (`export.py`)
- Measurement mode mapping helpers (`mappings.py`)

## Prerequisites

- `uv` installed (project uses uv as the environment manager)
- Network access to:
  - Emercit API (`http://emercit.com/map`)
  - Local MongoDB (`127.0.0.1:27017`)

## Setup

Install the locked environment:

```bash
uv sync --frozen
```

Start local MongoDB with Apptainer:

```bash
apptainer instance stop emercit-mongo 2>/dev/null || true
apptainer instance start --writable-tmpfs docker://mongo:7 emercit-mongo
apptainer exec instance://emercit-mongo sh -lc "mkdir -p /tmp/mongo-db && mongod --bind_ip_all --dbpath /tmp/mongo-db --fork --logpath /tmp/mongod.log"
```

Optional environment check:

```bash
uv run python -c "import platform,sys; print(platform.platform()); print(sys.version)"
```

Verify MongoDB is reachable from the host:

```bash
uv run python -c "from pymongo import MongoClient; c=MongoClient('127.0.0.1',27017,serverSelectionTimeoutMS=1500); print(c.admin.command('ping'))"
```

## Run

### 1. Bulk ingestion (Emercit -> MongoDB)

```bash
uv run provider.py
```

Runs `provider.py`, which:

- downloads available features from Emercit
- stores feature metadata in MongoDB
- pulls measurements by interval and persists them

Monitor ingestion progress (updates are expected to dominate inserts):

```bash
watch -n 2 'apptainer exec instance://emercit-mongo mongosh --quiet --eval "s=db.serverStatus(); printjson({insert:s.opcounters.insert, update:s.opcounters.update, query:s.opcounters.query})"'
```

### 2. CSV export (MongoDB -> CSV)

```bash
uv run export.py
```

Runs `export.py`, which loads measurements from MongoDB and writes CSV files.

### 3. Manual smoke path (optional)

```bash
uv run main.py
```

Runs `main.py`, which executes a MongoDB-backed measurement read.

## Lint

```bash
uv run pylint *.py
```

## Notes

- Mongo host defaults now point to local loopback (`127.0.0.1`).
- `provider.py` uses `8` worker threads for bulk ingestion.
- `export.py` currently contains a hardcoded CSV output path (`F:/export/...`); adjust it for your OS/filesystem if needed.

## Stop

Stop ingestion:

```bash
pkill -f "provider.py"
```

Stop MongoDB instance:

```bash
apptainer exec instance://emercit-mongo mongosh --quiet --eval "db.adminCommand({ shutdown: 1 })" || true
apptainer instance stop emercit-mongo
```
