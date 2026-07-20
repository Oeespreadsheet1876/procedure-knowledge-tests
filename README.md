# Bartoline Procedure Knowledge Tests

A bilingual (English/Polish) knowledge-test app for site procedures. One app,
any number of procedures — each procedure is just a data file, so new ones
can be added without touching the app itself.

## Folder structure

```
/                       (publish this whole folder on the web server)
  index.html            <- the test-taking app
  records.html          <- the training record matrix (candidates × procedures)
  styles.css            <- shared visual styling for both pages
  results-store.js      <- shared folder connection logic (File System Access API)
  matrix-builder.js     <- shared logic that turns result files into the matrix
  README.md             <- this file
  candidates/
    list.json           <- editable roster of candidate names (shows in the dropdown)
  procedures/
    manifest.json       <- list of available procedures (shows in the dropdown)
    wp09z.json           <- WP09Z: General Safety & Emergency Equipment
  results/
    (result files land here automatically once the folder is connected)
```

## Hosting requirements

This is a fully static site — **no database, no server-side application,
no PHP/ASP/Node needed.** It just needs to be served over `http://` or
`https://` by any basic web server (IIS, Apache, Nginx, etc.) — a folder on
the intranet is enough. This is the one hard requirement: the app loads its
question data with `fetch()`, which browsers block if the file is opened
directly (`file://...`) rather than through a server. If someone opens it
the wrong way, the app shows a message explaining this rather than failing
silently.

**For IT:** publish this folder as-is (e.g. as a subsite/virtual directory).
No build step, no dependencies, no server config beyond normal static file
serving.

## Adding a new procedure

No changes to `index.html` are needed. Two steps:

**1. Create `procedures/<your-id>.json`** following this shape:

```json
{
  "id": "wp09l",
  "docRef": "WP09L",
  "issue": "2",
  "title": {
    "en": "Tub Mix Filling — Karmelle Line",
    "pl": "Napełnianie mieszanki w kadziach — linia Karmelle"
  },
  "passMark": 80,
  "questions": [
    {
      "en": { "q": "Question text in English?", "o": ["Option A", "Option B", "Option C", "Option D"] },
      "pl": { "q": "Treść pytania po polsku?",   "o": ["Opcja A", "Opcja B", "Opcja C", "Opcja D"] },
      "correct": 1
    }
  ]
}
```

Notes:
- `correct` is the zero-based index into the `o` (options) array — so `1` means the *second* option is correct. Keep the option order identical between `en` and `pl` for the same question.
- `passMark` is a whole number percentage (e.g. `80` for 80%).
- Any number of questions is fine — the test always scores out of 100%.
- Both `en` and `pl` are required for every question (this is a bilingual tool).

**2. Add one entry to `procedures/manifest.json`:**

```json
{
  "procedures": [
    { "id": "wp09z", "file": "wp09z.json", "docRef": "WP09Z", "title": {"en": "General Safety & Emergency Equipment", "pl": "Bezpieczeństwo ogólne i wyposażenie awaryjne"} },
    { "id": "wp09l", "file": "wp09l.json", "docRef": "WP09L", "title": {"en": "Tub Mix Filling — Karmelle Line", "pl": "Napełnianie mieszanki w kadziach — linia Karmelle"} }
  ]
}
```

Refresh the page — the new procedure appears in the dropdown immediately.
No app code is touched, so there's no risk of breaking the existing tests
when adding new ones.

## What it does

- Candidate picks their name (from a shared roster) and a procedure, enters the date
- Bilingual EN/PL toggle switches every question, label, and the certificate
- Scores out of 100%, pass mark is per-procedure (set in that procedure's JSON)
- Full review of right/wrong answers after submitting
- Result is saved automatically to the shared `results` folder (or offered as a download if that's not possible on that machine)
- On a pass: a printable certificate with the candidate's name, date, score,
  and the procedure's reference/title, styled to match the Bartoline WP09Z
  document branding
- `records.html` shows a live training matrix (candidates × procedures) built from every saved result, exportable to CSV
- "Retake test" clears answers and starts over; "Choose a different procedure" returns to the start to pick another one

## Shared results & the training record

Results now save centrally, and there's a live training matrix — no manual step, no database, and still no backend server code. Here's how:

**Connecting the results folder (once per browser/computer).** The first time
someone uses `index.html` or `records.html`, they click "Connect results
folder" and pick the shared `results` folder (e.g. a mapped network drive).
The browser remembers this choice, so it's normally a one-off per machine,
not per test. This uses the **File System Access API**, which needs a
**Chromium-based browser — Chrome or Edge** (not Firefox or Safari). Edge is
the default on Windows, so this should be fine for most machines; it's worth
just confirming with IT which browser is standard on the shopfloor PCs.

**Taking a test.** Once connected, finishing a test writes a small JSON
result file straight into the `results` folder automatically — no download,
no manual save. If a computer *isn't* connected (or is on a browser that
doesn't support this), the app automatically falls back to offering the
result as a normal download, with a note to save it into the shared folder
by hand. Either way, nothing is lost — it only degrades to a manual step
where it strictly has to.

**Viewing the training record.** Open `records.html`, connect to the same
results folder, and it builds a live matrix: every candidate down the side,
every procedure across the top, and each cell showing their most recent
pass/fail, score, and date (hovering a cell shows how many times they've
attempted it, if more than once). "Refresh" re-reads the folder; "Export
CSV" downloads the whole matrix for Excel/Teams/reporting.

**Candidates.** `candidates/list.json` is a simple editable list, exactly
like `procedures/manifest.json` — add or rename people there and the
dropdown (and the matrix) picks it up. If someone isn't on the list yet,
there's an "Other / not listed" option so a test never has to be blocked
on the roster being up to date.

```json
{ "candidates": ["Jan Kowalski", "Anna Nowak"] }
```

## What it doesn't do (by design, for now)

There's no server-side database — the "database" is simply the folder of
result files, read live by the browser. That's deliberately simple and
matches how the OEE dashboard already works. The trade-off: everyone
viewing the training record needs Chrome or Edge and access to that shared
folder. If a proper database/reporting layer is ever wanted instead, that's
a bigger step involving IT running an actual backend service — this
version avoids that entirely.
