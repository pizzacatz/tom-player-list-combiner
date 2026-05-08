# Validation & Unit Test Cases

All unit tests should be runnable in a browser console or a tiny built‑in test runner. Each case states input, expected result, and relevant edge conditions.

## 1. Merge Algorithm

### 1.1 Basic latest‑modified wins
- **Input:**  
  File A: userid="1", lastmodified="04/30/2026 22:00:00"  
  File B: userid="1", lastmodified="05/01/2026 00:00:00"  
- **Expected:** Winning record from File B, combined creationdate = earliest of the two, lastmodified = "05/01/2026 00:00:00".

### 1.2 Tie in lastmodified → oldest creationdate wins
- **Input:**  
  Both records have same lastmodified="04/30/2026 20:00:00".  
  File A creation="04/30/2026 19:00:00", File B creation="04/30/2026 19:30:00".  
- **Expected:** File A wins (older creation). Combined creationdate = earliest ("04/30/2026 19:00:00"), lastmodified stays same.

### 1.3 Timestamp adjustment correctness
- **Input:**  
  File A: creation="04/01/2026 10:00:00", modified="04/10/2026 10:00:00"  
  File B: creation="04/05/2026 10:00:00", modified="04/20/2026 10:00:00"  
- **Expected:** Winner is File B. Final record: creation = "04/01/2026 10:00:00", lastmodified = "04/20/2026 10:00:00".

### 1.4 Identical timestamps (extremely rare)
- **Input:** Both records identical in creation and modification.  
- **Expected:** Winner is arbitrary but deterministic (e.g., first file processed). Combined fields unchanged.

### 1.5 Duplicate userid within one file
- **Input:** File A contains two players with userid="5", different modified dates.  
- **Expected:** Warning emitted. Merge treats them as internal candidates and picks the newest modified among all three (two from A, one from B). Final merged record follows same rules.

## 2. Player Validation

### 2.1 Missing required field → skip
- **Input:** `<player userid="1"><firstname>John</firstname>...` but `<lastname>` is empty/missing.  
- **Expected:** Player skipped, warning logged. Not included in merged output.

### 2.2 Missing userid attribute
- **Input:** `<player><firstname>...</firstname>...` without `userid`.  
- **Expected:** Skipped, warning.

### 2.3 Malformed date string
- **Input:** `lastmodifieddate="invalid date"`.  
- **Expected:** How to handle? Current spec says “compare timestamps” – we can parse dates using `Date.parse` with `/` replaced or by splitting. If parsing fails, treat the timestamp as the lowest possible value (or skip the player). **Decision needed:** probably skip the player if any date is unparseable, with warning. Add test case.

## 3. File Renaming & Sanitization

### 3.1 Basic sanitization
- **Input:** Store name = "Downtown League!" → Sanitized: "Downtown-League".  
- **Test:** Check that `!` becomes `-`, no double `--`.

### 3.2 Consecutive special characters
- **Input:** "My Super###Store" → "My-Super-Store". (No collapsing of repeated `#` separately? We treat all as replace then collapse consecutive hyphens.)  
  Actually: replace each non-allowed with `-`, then collapse consecutive `-`. So "###" becomes "---" → collapsed to "-". Expected: "My-Super-Store".

### 3.3 Leading/trailing hyphens
- **Input:** " -Hello World- " → after sanitization: "-Hello-World-", then trim leading/trailing hyphens → "Hello-World".

### 3.4 Empty store name → file date fallback
- **Input:** Store name empty, file A has `lastModified` timestamp 2026-05-01T12:00:00.000Z → fallback name "2026-05-01".  
- **Test:** verify that file’s lastModified property is read and formatted correctly (UTC date, `YYYY-MM-DD`). Ensure it’s not broken by timezone.

### 3.5 Identical store names
- **Input:** Both store names = "Mall". After sanitization, both remain "Mall".  
- **Expected:** Warning shown, auto‑append `_1` to Store 1, `_2` to Store 2. Resulting filenames: `players_Mall_1.xml` and `players_Mall_2.xml`.

## 4. XML Output and Comments

### 4.1 Preserve original comment
- **Input:** File 1 has `<!-- Players database. -->` comment.  
- **Expected:** Merged `players.xml` starts with `<?xml ...>`, then `<!-- Players database. -->`, then the new date‑and‑store comment.

### 4.2 New comment format
- **Test:** For store names "StoreA" and "StoreB" and current date 2026-05-08, the comment should be:  
  `<!-- 2026-05-08 : Merged players from StoreA and StoreB -->`.  
  (No extra spaces or punctuation inside store names.)

### 4.3 Escaping in output
- **Input:** Player name contains `&` or `<`.  
- **Expected:** XML serializer escapes correctly; no manual checks needed beyond using `XMLSerializer`.

## 5. BOM Handling
- **Input:** File starts with `\uFEFF<?xml ...`.  
- **Expected:** BOM is stripped before parsing; parsing succeeds normally.

## 6. Sort Order
- **Test:** Create unsorted player entries with names `["zara", "adam", "Adam", "Bob"]`. After merge, order should be `adam`, `Adam`, `Bob`, `zara` (case‑insensitive alphabetical). Names with identical first name are then sorted by last name.

## 7. Conflict Preview Table
- **Test:** Two records with userid="99": File1 name "John Doe", File2 name "Johnathan Doe". Merge keeps File2 because of newer date. Conflict table must show:
  - userid=99
  - Field: full name (or first/last separately)
  - Values from both files
  - Kept value
- **Test:** No conflict if only dates differ but names/birthdate identical.

## 8. Download Functionality
- **Test:** After merge, clicking “Download Combined” triggers a download of `players.xml` blob. Verify file name and content.  
- **Test:** Store file downloads preserve original raw content exactly (byte‑for‑byte identical, except maybe BOM removed? We’ll keep original raw text as‑is including any BOM for the renamed copies, because it’s a direct copy). Ensure renamed files match input files.

## 9. Edge Cases

### 9.1 One file empty (no players)
- **Input:** File 2 is a valid XML with `<players></players>` but no `<player>` elements.  
- **Expected:** Warnings (maybe none), merge works, output contains only players from File 1. Renamed copy of File 2 is still provided.

### 9.2 Both files empty
- **Input:** Both have no valid players.  
- **Expected:** Abort with message “No valid players found in either file.”

### 9.3 Non‑XML file
- **Input:** File is a text file with “Hello World”.  
- **Expected:** Show parsing error, no downloads.

### 9.4 Very large file (performance)
- **Test:** Create a synthetic file with 10,000 players. Parsing and merge should complete in under a few seconds; UI remains responsive.