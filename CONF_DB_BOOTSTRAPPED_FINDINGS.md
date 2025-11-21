# CONF_DB_BOOTSTRAPPED - Complete Instance Analysis

## Summary

This document provides a comprehensive analysis of all instances of `CONF_DB_BOOTSTRAPPED` in the home-generative-agent repository.

## Total Instances Found: 4

### Instance 1: Constant Definition
**File:** `custom_components/home_generative_agent/const.py`  
**Line:** 16  
**Type:** Constant Definition

```python
CONF_DB_BOOTSTRAPPED = "db_bootstrapped"
```

**Purpose:** Defines the configuration key used to track whether the PostgreSQL database (used for vector store + checkpointer) has been bootstrapped/initialized.

**Context:** This constant is part of the PostgreSQL configuration section, located alongside other database-related constants like `CONF_DB_URI`.

---

### Instance 2: Import Statement
**File:** `custom_components/home_generative_agent/__init__.py`  
**Line:** 44  
**Type:** Import

```python
from .const import (
    ...
    CONF_DB_BOOTSTRAPPED,
    ...
)
```

**Purpose:** Imports the constant from the const module for use in the initialization module.

**Context:** Part of a large import block that brings in all configuration constants needed for the integration setup.

---

### Instance 3: Bootstrap Check
**File:** `custom_components/home_generative_agent/__init__.py`  
**Line:** 160  
**Type:** Usage - Conditional Check

```python
if entry.data.get(CONF_DB_BOOTSTRAPPED):
    return
```

**Purpose:** Checks if the database has already been bootstrapped to avoid re-running the setup process.

**Context:** Located in the `_bootstrap_db_once()` async function. This function ensures that database initialization (store.setup() and checkpointer.setup()) only runs on the first initialization.

**Function Context:**
```python
async def _bootstrap_db_once(
    hass: HomeAssistant,
    entry: ConfigEntry,
    store: AsyncPostgresStore,
    checkpointer: AsyncPostgresSaver,
) -> None:
    if entry.data.get(CONF_DB_BOOTSTRAPPED):
        return

    # First time only
    await store.setup()
    await checkpointer.setup()

    # Persist the flag so it survives restarts
    hass.config_entries.async_update_entry(
        entry, data={**entry.data, CONF_DB_BOOTSTRAPPED: True}
    )
```

---

### Instance 4: Flag Persistence
**File:** `custom_components/home_generative_agent/__init__.py`  
**Line:** 169  
**Type:** Usage - Setting Value

```python
hass.config_entries.async_update_entry(
    entry, data={**entry.data, CONF_DB_BOOTSTRAPPED: True}
)
```

**Purpose:** Persists the bootstrap flag to the config entry data after successful database initialization, ensuring the flag survives Home Assistant restarts.

**Context:** Located in the `_bootstrap_db_once()` function after the database setup operations complete successfully.

---

## Architectural Overview

### Purpose of CONF_DB_BOOTSTRAPPED

The `CONF_DB_BOOTSTRAPPED` constant is part of a one-time initialization pattern for the PostgreSQL database that stores:
- Long-term memory using vector store (AsyncPostgresStore)
- Short-term thread-based memory using checkpointer (AsyncPostgresSaver)

### Workflow

1. **First Installation/Setup:**
   - `CONF_DB_BOOTSTRAPPED` is not set in entry.data
   - `_bootstrap_db_once()` runs store.setup() and checkpointer.setup()
   - Flag is set to `True` in config entry data
   
2. **Subsequent Restarts:**
   - `CONF_DB_BOOTSTRAPPED` is `True` in entry.data
   - `_bootstrap_db_once()` returns early without re-initializing
   - Database tables/schemas are preserved

### Related Components

- **AsyncPostgresStore:** LangGraph store for long-term memory with vector similarity search
- **AsyncPostgresSaver:** LangGraph checkpointer for conversation state persistence
- **PostgreSQL with pgvector:** External add-on that provides the database backend

### Configuration Entry Data

The flag is stored in the Home Assistant config entry's `data` dictionary, which persists across restarts in the `.storage/core.config_entries` file.

---

## Additional Notes

- The database URI is configured via `CONF_DB_URI` (default: `postgresql://ha_user:ha_password@localhost:5432/ha_db?sslmode=disable`)
- Database bootstrapping is called from `async_setup_entry()` at line 461
- The pattern ensures idempotent database initialization
- No migration or version tracking is associated with this flag (migrations are handled separately, see `migrate_person_gallery()`)

---

## Search Command Used

```bash
grep -r "CONF_DB_BOOTSTRAPPED" . --include="*.py" --include="*.yaml" --include="*.yml" --include="*.json" --include="*.md"
```

## Files Containing CONF_DB_BOOTSTRAPPED

1. `custom_components/home_generative_agent/const.py` (1 instance)
2. `custom_components/home_generative_agent/__init__.py` (3 instances)

**Total Files:** 2  
**Total Instances:** 4
