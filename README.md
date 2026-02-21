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

- `pixi` installed (project uses Pixi as the only environment manager)
- Network access to:
  - Emercit API (`http://emercit.com/map`)
  - Local MongoDB (`127.0.0.1:27017`)

## Setup

Install the locked environment:

```bash
pixi install --locked
```

Start local MongoDB with Docker:

```bash
docker run -d --name emercit-mongo -p 27017:27017 mongo:7
```

Optional environment check:

```bash
pixi run check-env
```

## Run

### 1. Bulk ingestion (Emercit -> MongoDB)

```bash
pixi run run-provider
```

Runs `provider.py`, which:

- downloads available features from Emercit
- stores feature metadata in MongoDB
- pulls measurements by interval and persists them

### 2. CSV export (MongoDB -> CSV)

```bash
pixi run run-export
```

Runs `export.py`, which loads measurements from MongoDB and writes CSV files.

### 3. Manual smoke path (optional)

```bash
pixi run run-main
```

Runs `main.py`, which executes a MongoDB-backed measurement read.

## Lint

```bash
pixi run lint
```

## Notes

- Mongo host defaults now point to local loopback (`127.0.0.1`).
- `export.py` currently contains a hardcoded CSV output path (`F:/export/...`); adjust it for your OS/filesystem if needed.
