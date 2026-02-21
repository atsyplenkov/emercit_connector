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
  - MongoDB instance used by the scripts

## Setup

Install the locked environment:

```bash
pixi install --locked
```

Optional environment check:

```bash
pixi run check-env
```

## Run

### 1. Manual smoke path

```bash
pixi run run-main
```

Runs `main.py`, which currently executes a measurement query through `EmercitMongo`.

### 2. Bulk ingestion

```bash
pixi run run-provider
```

Runs `provider.py`, which:

- downloads available features from Emercit
- stores feature metadata in MongoDB
- pulls measurements by interval and persists them

### 3. CSV export

```bash
pixi run run-export
```

Runs `export.py`, which loads measurements from MongoDB and writes CSV files.

## Lint

```bash
pixi run lint
```

## Notes

- The scripts currently contain hardcoded defaults (for example Mongo host and export path).
- If your environment differs, update script parameters in `main.py`, `provider.py`, and `export.py` before running long jobs.
