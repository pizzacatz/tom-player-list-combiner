# Project Constitution – TOM Player List Merger

## 1. Mission
Provide tournament organizers (TOs) for Play! Pokémon events a zero‑dependency, single‑page browser tool that combines two TOM‑exported `players.xml` files into one merged list, while preserving renamed copies of the originals.

## 2. Core Principles
- **Local only** – no data leaves the user’s device; no server calls.
- **Zero runtime dependencies** – pure HTML, CSS, and vanilla JavaScript.
- **Self‑contained** – a single `.html` file that can run offline after first load.
- **Deterministic** – identical inputs always produce identical outputs.
- **Transparent** – every automated decision (merge conflicts, skipped entries) is shown to the user.
- **Resilient** – gracefully handles missing data, malformed input, and edge cases without crashing.

## 3. Functional Guarantees
### 3.1 Merge Logic
- Uniqueness is defined by the `userid` attribute (integer string, case‑sensitive comparison).
- For each unique `userid`, the winning record is the one with the **latest `<lastmodifieddate>`**.
- Tie‑breaker: prefer the record with the **oldest `<creationdate>`**.
- The winning record’s timestamps are adjusted:
  - `creationdate` ← **earliest** of the two records.
  - `lastmodifieddate` ← **latest** of the two records.
- Comparison is based purely on timestamps; name/birthdate are **not used** for the decision, but are shown in the conflict preview.

### 3.2 Data Integrity
- If a `<player>` element is missing any of the required children (`firstname`, `lastname`, `birthdate`, `creationdate`, `lastmodifieddate`), it is **skipped** and a warning is emitted.
- If the same `userid` appears **twice within one input file**, a warning is emitted, but processing continues (internally applying the same merge rules).
- Invalid XML input halts processing with a clear error.

### 3.3 Output Files
1. **`players.xml`** – merged list, sorted alphabetically by `firstname` (case‑insensitive), then `lastname`.
2. **`players_<storeA>.xml`** – exact copy of input file A, renamed.
3. **`players_<storeB>.xml`** – exact copy of input file B, renamed.
- Store names are sanitized: anything not `[A-Za-z0-9_-]` becomes a hyphen; consecutive hyphens collapse.
- If a store name field is left blank, the file’s `lastModified` date (ISO format `YYYY-MM-DD`) is used.

### 3.4 File Comments
- The merged output retains the original `<!-- Players database. -->` comment from **Input File 1**.
- Immediately following that line, a new comment is added:  
  `<!-- YYYY-MM-DD : Merged players from <Store A> and <Store B> -->`  
  using the current local date and the sanitised store labels.

## 4. User Experience Pillars
- **Single‑page, no multi‑step wizard** – all controls visible.
- **Drag‑and‑drop + file input** for XML files.
- **Conflict preview** – toggle that lists differences (name, birthdate) for identical userids.
- **Warnings panel** – non‑blocking messages for duplicates, skipped entries, identical store names.
- **Three separate download buttons** – user manually triggers each download to avoid browser pop‑up blocking.
- PWA‑ready – includes a manifest and a basic service worker for offline installation (optional but planned).

## 5. Technology Constraints
- No build tools, no bundlers, no npm.
- No external CDN references; all code inline.
- ES6+ allowed, but must work in modern browsers (Chrome, Edge, Firefox, Safari) from 2022+.
- XML parsing via `DOMParser` and serialization via `XMLSerializer`.
- All date comparisons performed after converting the `MM/DD/YYYY HH:MM:SS` format into a comparable value (e.g., ISO string or timestamp).
- BOM (byte order mark) is stripped before parsing.