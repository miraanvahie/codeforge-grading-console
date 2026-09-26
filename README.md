# Grading Console – BITS Digital CodeForge V1.0

**Live app:** (https://miraanvahie.github.io/codeforge-grading-console/)

A browser-based console for turning a marks spreadsheet into final letter grades. It started as the buggy prototype supplied for CodeForge; this repo contains the debugged version and a redesigned version built on top of it.

![Grading console](docs/screenshot.png)

## What's in this repo

| Path | What it is |
|---|---|
| `index.html` | The redesigned console (Stage 2). This is the deployed app. |
| `stage1-bugfix/index.html` | The original prototype with only the bugs fixed (Stage 1), kept separate so the fixes are easy to review |
| `original/` | The file as supplied, for comparison |
| `BUG_FIX_LOG.md` | All 21 bugs: how each was reproduced, the root cause, the fix and how it was tested |
| `test-data/` | Spreadsheets used for testing, including edge cases |

Everything runs in the browser. There is no server and no student data leaves the instructor's computer.

## Stage 1: debugging

21 bugs were found and fixed. The ones that could produce wrong grades are the most important:

- Students above the A maximum or below the E minimum were silently dropped from the exported file.
- Marks with decimals (79.5) fell between two grades and disappeared from the export.
- Rows with missing IDs were exported as `undefined`, and duplicate IDs were exported twice.
- Min and Max statistics were swapped, and text in the marks column produced a median of 3033.5.
- Files using the header wording from the challenge brief loaded with no data.

The full log with reproduction steps is in [BUG_FIX_LOG.md](BUG_FIX_LOG.md).

## Stage 2: enhancements

Each change starts from a problem an instructor actually has while grading.

**1. Cutoffs that cannot be wrong.** The original asked for 16 values (a min and a max for every grade) and then told you when they did not line up. Grades must touch, so the maximum of each grade is always one below the start of the grade above it. The console now asks only where each grade starts; the ends are filled in automatically, A always ends at 100 and E always starts at 0. Gaps and overlaps are impossible by design, and a value outside the allowed range is highlighted with a message that says which numbers are allowed and why.

**2. A distribution chart you can grade on.** One bar per mark, coloured by the grade it currently gets, with a marker at each cutoff. Drag a marker (or focus it and use the arrow keys) and the bars, counts and student list update as you move. This shows the thing instructors look for when setting cutoffs: natural gaps in the distribution, and clusters sitting right under a boundary. An optional fitted normal curve is available for comparison.

**3. Students near a cutoff.** A dedicated view lists every student who is 1, 2 or 3 marks short of the next grade. Each row has a one-click action to move that cutoff down to include the student, with a tooltip saying how many students it would move. Every change can be undone (button, toast, or Ctrl+Z).

**4. An import that explains itself.** The file is checked row by row. Rows that cannot be graded (blank or non-numeric marks, marks outside 0–100, missing IDs or courses, duplicates) are listed with their Excel row number so the instructor can fix the source file. Header names are matched flexibly, drag and drop works, and a blank template can be downloaded. Where marks have decimals, the instructor chooses the rounding rule (nearest or always up) instead of the app deciding silently.

**5. Review before download.** "Finalize" opens a summary first: the grade distribution, which cutoffs differ from the defaults, how many rows were excluded, how many marks were rounded, and the instructor name (required). Downloads are CSV (compatible with the original format) or Excel with a second sheet summarising cutoffs and counts. Files are named after the course and date.

**6. Safer everyday use.** Cutoffs are saved per course in the browser, so reopening a course restores them. The course list marks courses already downloaded, the page warns before closing with changes that were not downloaded, and loading a new file asks before discarding unsaved work.

**Also:** grade band table shows count, share and change against the default ranges; student list with search, sort and grade filter; sample data button so anyone can try the app without a file; responsive layout down to phone width; keyboard access and screen reader labels throughout; reduced motion respected.

## Try it quickly

1. Open the app and click **Try with sample data**.
2. Pick a course. Drag the **A** marker on the chart and watch the counts change.
3. Open **Near a cutoff** and use one of the **Lower … to …** buttons, then **Undo**.
4. Click **Review and download**.

To test error handling, upload `test-data/edge_cases.xlsx`.

## Deploying

The app is a single HTML file, so any static host works.

**Netlify Drop (fastest, no account needed to start):** go to https://app.netlify.com/drop and drag the project folder onto the page. You get a public URL straight away. Sign up to keep it permanently.

**GitHub Pages:**
1. Create a new public repository and upload the contents of this folder (keep `index.html` at the top level).
2. In the repository, open Settings → Pages.
3. Under "Build and deployment", choose "Deploy from a branch", branch `main`, folder `/ (root)`, and save.
4. After about a minute the site is live at `https://<your-username>.github.io/<repo-name>/`.

**Vercel:** import the GitHub repository at https://vercel.com/new, keep the framework preset as "Other", and deploy.

After deploying, open the URL in a private window and run through "Try it quickly" once.

## Technical notes

- Plain HTML, CSS and JavaScript, no build step. [SheetJS](https://sheetjs.com) 0.18.5 (pinned) reads and writes Excel files.
- The chart is SVG generated from the data, so it stays sharp at any size and each marker is a focusable slider.
- Marks are rounded once on import; grading uses whole numbers only, which matches the grade bands.
- Saved cutoffs and the instructor name use `localStorage`. If storage is unavailable (private mode) the app still works, it just does not remember between visits.
