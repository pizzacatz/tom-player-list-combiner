# Implementation Task List

## Phase 0: Setup
- [ ] Create a single `index.html` file containing all HTML, CSS, and JavaScript.
- [ ] Add minimal PWA files if desired: `manifest.json` and `sw.js` (can be inline via blob URL).

## Phase 1: UI Skeleton
- [ ] Build the DOM structure:
  - Two drop zones (or `<input type="file">`) with labels “Store 1 File” and “Store 2 File”.
  - Text inputs for “Store 1 Name” and “Store 2 Name”.
  - “Merge & Prepare Downloads” action button.
  - Status/warning area (initially hidden).
  - Conflict toggle and table area (hidden until conflicts exist).
  - Three download buttons (initially disabled).
- [ ] Style with simple, mobile‑friendly CSS.

## Phase 2: File Handling & Parsing
- [ ] Implement file reading using `FileReader.readAsText`.
- [ ] Strip BOM if present (U+FEFF at position 0).
- [ ] Attempt `DOMParser.parseFromString` — on error, show error and stop.
- [ ] Extract all `<player>` elements.
- [ ] Validate each player: must contain non‑empty `firstname`, `lastname`, `birthdate`, `creationdate`, `lastmodifieddate`; otherwise mark as skipped.
- [ ] Collect warnings:
  - Duplicate `userid` within a single file.
  - Skipped players.
- [ ] Store parsed data in a structured array of objects.

## Phase 3: Merge Algorithm
- [ ] Build a `Map<userid, {players: [playerObj, ...], winningIndex: number}>` from both datasets.
- [ ] For each map entry:
  - Sort the player candidates by `lastmodifieddate` descending, then `creationdate` ascending.
  - Pick the first candidate as winner.
  - Compute merged `creationdate` = earliest among candidates; `lastmodifieddate` = latest.
- [ ] Generate the sorted combined list: sort by `firstname` lowercase, then `lastname` lowercase.

## Phase 4: Conflict Detection
- [ ] For each userid that appears in both files, compare `firstname`, `lastname`, and `birthdate`.
  - If any differ, log a conflict with both values and mark which record was kept.
- [ ] If no differences exist, treat as duplicate without conflict (still merge timestamps).

## Phase 5: File Renaming Logic
- [ ] Implement store name sanitization:
  - Replace `[^A-Za-z0-9_-]` with `-`.
  - Collapse consecutive `-`.
  - Remove leading/trailing `-`.
- [ ] If name is empty after trimming, use `YYYY-MM-DD` from file’s `lastModified` date.
- [ ] If both sanitized store names are identical, warn user and auto‑append `_1` to Store 1 and `_2` to Store 2.

## Phase 6: XML Generation
- [ ] Reconstruct `players.xml` string:
  - `<?xml version="1.0" encoding="UTF-8"?>`
  - Original comment from File 1.
  - New merge comment with date and store names.
  - `<players>` root.
  - Sorted player elements (using string templates, properly escaped).
- [ ] Create renamed copies: take raw text of each original file, no modifications.
- [ ] Create Blob objects for each file with `type: "application/xml"`.

## Phase 7: Downloads
- [ ] Attach blob URLs to three download links, set `download` attribute with proper filenames.
- [ ] Enable buttons only after successful merge.
- [ ] Provide visual feedback on click (optional).

## Phase 8: Conflict Preview UI
- [ ] When conflicts exist, populate a table with columns: `userid`, `Field`, `Store 1 value`, `Store 2 value`, `Kept value`.
- [ ] Show/hide via toggle button; count shown in summary.

## Phase 9: Warnings & Messages
- [ ] Render warnings as a bulleted list in the warning area.
- [ ] If both files contain valid players but skipped entries or duplicates, still allow merge but show warning.
- [ ] If no valid players found in either file, abort with clear message.

## Phase 10: PWA (Optional but Recommended)
- [ ] Add `<link rel="manifest" href="...">` or inline manifest.
- [ ] Register a minimal service worker that caches the page for offline use (if desired).
- [ ] Ensure the page passes Lighthouse PWA audit.

## Phase 11: Unit Testing
- [ ] Write unit tests for:
  - Merge decision logic (winning record, tie‑breaker, timestamp adjustment).
  - Store name sanitization and fallback.
  - Player validation (missing fields, duplicate userid detection).
  - XML comment generation.
  - BOM stripping.
  - Sorting.
- [ ] Use a simple inline test harness (e.g., a hidden `<div>` that runs tests on load in a debug mode) or a separate test file. See `validation.md` for test cases.