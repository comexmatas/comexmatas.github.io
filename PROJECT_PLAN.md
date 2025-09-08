## Project Plan: Competitor Price Capture (Single-File App)

### Goals
- **Single-file app**: All logic in `index.html` (no backend), using `data/products.xlsx` as input.
- **Search**: Filter by `brand_name`, `product_name`, or `product_category`.
- **Capture**: Enter competitor price for selected product(s).
- **Save**: Persist captured prices back to an XLSX file (client-only workflow).

### Constraints & Saving Strategy
- The app runs as a static site and cannot write to the server repository automatically.
- "Sync" behavior:
  - First try to save directly to the local repository file `data/products.xlsx` via the File System Access API (user grants access via a picker the first time).
  - If not supported or permission denied, fall back to downloading the updated workbook so the user can replace the file manually.
- Supported best on Chromium browsers. Non-Chromium will use the download fallback.

### Data Model Assumptions
- Input XLSX has a single primary sheet (`Products`).
- App adds/uses a volatile column in memory: `competitor_price` (captured by the user).

### Tech Choices
- **XLSX parsing/generation**: SheetJS (`xlsx`) via a browser CDN bundle.
- **UI**: Plain HTML/CSS + vanilla JS inside `index.html`.
- **Storage**: In-memory state + optional `localStorage` for unsaved draft entries.

### User Flows
1. **Load data**
   - Default: fetch `data/products.xlsx` (GET). And show the first 100 rows.
2. **Search/filter**
   - Single search box that matches substring across `product_id`,`brand_name`, `product_name`, `product_category_name_level_1`.
3. **Capture price**
   - Each result row has an input for `competitor_price`. Edits are saved to the original file `data/products.xlsx` when the user hits the `sync` button.
   - On first sync, the app will ask the user to select the local `data/products.xlsx` file to grant write access; subsequent syncs reuse that handle.
   - If the browser or user does not allow direct file write, the app provides a download of the updated workbook to be placed at `data/products.xlsx` manually.
   - Safety: validation (numeric, non-negative), highlight errors, prevent save with invalid cells.

### Architecture (Single `index.html`)
- Sections:
  - `<header>`: search, file open button, clear filters.
  - `<main>`: results table with paging; per-row price inputs; dirty-state indicators.
  - `<footer>`: status bar (rows loaded, dirty count), actions (Sync, and fallback Download if needed).
- JS modules inside `<script type="module">`:
  - `loadWorkbook()` – fetches default XLSX or opens via file picker; parses to JSON rows.
  - `renderTable(rows)` – renders current page; binds events.
  - `applySearch(query)` – filters rows.
  - `updatePrice(rowId, price)` – updates state and marks dirty.
  - `syncToLocalFile()` – writes the updated workbook to `data/products.xlsx` using File System Access API if available.
  - `downloadUpdatedWorkbook()` – fallback to generate and download XLSX when direct write is unavailable.
  - `persistDraft()` / `restoreDraft()` – optional local draft handling.

### Validation & UX
- Numeric-only price inputs; disallow negative values; only allow numbers
- Visual cues for dirty rows and unsaved changes count.
- Disable sync if there are validation errors; provide inline error messages.
- Keyboard-friendly: typing in price auto-commits on blur/Enter; arrow keys navigate rows (stretch goal).

### Performance
- Load and parse on a Web Worker if the file is large (stretch goal). Start simple in main thread.
- Paginate results to 50 rows/page to avoid DOM bloat.

### Milestones & Acceptance Criteria
- **M1: Data load & render**
  - Can fetch and parse `data/products.xlsx`.
  - First 50 rows render in a table with pagination controls.
- **M2: Search**
  - Typing in the search box filters by brand/product/category across all rows.
  - Clear button resets to full dataset and page 1.
- **M3: Edit capture**
  - User can input `competitor_price` per row; edits persist during navigation.
  - Dirty-state indicator shows count of edited rows.
- **M4: Sync saving**
  - Sync writes updated workbook to local `data/products.xlsx` via File System Access API.
  - Fallback: provides a valid XLSX download if direct write isn’t available.

### Risks & Mitigations
- **Browser support for direct file writes**: Use File System Access API when available; otherwise fallback to download.
- **Large XLSX files**: paginate, consider worker offloading later.
- **Header name drift**: implement a header mapping UI if headers aren’t exact (later).

### Implementation Checklist (Short)
- Wire SheetJS; load/parse `data/products.xlsx`.
- Render table with pagination; add search.
- Price input per row; track dirty rows.
- Implement Sync via File System Access API with download fallback.
