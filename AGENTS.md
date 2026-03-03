# Emercit Connector

Python connector that fetches Emercit hydrometeorological data and stores it in MongoDB.

## Critical Rules

- Keep changes minimal and scoped to the requested behavior.
- Preserve Mongo document shapes used by `features` and `measurements` collections unless explicitly asked to migrate data.
- Preserve timezone handling (`Etc/GMT-3`) for measurement timestamps unless explicitly asked to change it.
- Do not add new abstractions or configuration unless requested.

## Commands

- `uv sync --frozen` - Install the locked project environment (`.venv`, `uv.lock`).
- `uv run main.py` - Manual local read/write smoke path.
- `uv run provider.py` - Bulk historical fetch and persistence.
- `uv run export.py` - CSV export from MongoDB.
- `uv run pylint *.py` - Lint check.

## Architecture

| Path | Purpose |
|------|---------|
| `connector.py` | HTTP client for Emercit endpoints (`overall.php`, `mgraph.php`). |
| `mongo.py` | MongoDB persistence and query methods. |
| `provider.py` | Orchestration for bulk ingestion across stations/date intervals. |
| `mappings.py` | Mapping from data type to connector query kwargs. |
| `export.py` | CSV export pipeline from stored measurements. |
| `main.py` | Small manual execution entrypoint. |

## Key Patterns

- `EmercitConnector.mgraph()` returns `(observations, period_from, period_to)` where `observations` is keyed by metric and timestamp.
- `EmercitMongo.save_measurements()` upserts by `(station_id, mode, time)` using bulk operations.
- `EmercitProvider.dump_all()` fetches station metadata first, then processes daily intervals concurrently.

## Data Stores

| Collection | Key Fields |
|------------|------------|
| `features` | `properties.id`, `properties.name`, `properties.data.*` |
| `measurements` | `station_id`, `mode`, `time`, metric fields (`d`, `z`, `t`, etc.) |
| `stations` | `id` (queried by `get_station`) |

