# Bartoline Procedure Knowledge Tests

A bilingual (English/Polish) knowledge-test app for site procedures. One app,
any number of procedures — each procedure is just a data file, so new ones
can be added without touching the app itself.

## Folder structure

```
/                       (publish this whole folder on the web server)
  index.html            <- the app (never needs editing to add a procedure)
  README.md             <- this file
  procedures/
    manifest.json       <- list of available procedures (shows in the dropdown)
    wp09z.json           <- WP09Z: General Safety & Emergency Equipment
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

- Candidate picks a procedure, enters their name and date
- Bilingual EN/PL toggle switches every question, label, and the certificate
- Scores out of 100%, pass mark is per-procedure (set in that procedure's JSON)
- Full review of right/wrong answers after submitting
- On a pass: a printable certificate with the candidate's name, date, score,
  and the procedure's reference/title, styled to match the Bartoline WP09Z
  document branding
- "Retake test" resets answers; "Choose a different procedure" returns to
  the start to pick another one

## What it doesn't do (by design, for now)

There's no central record of who's taken a test or what they scored — each
pass produces a printable/PDF certificate, and that's the only record. If a
shared log of results is ever wanted, that's a bigger step (some form of
server-side storage) and isn't part of this static version.
