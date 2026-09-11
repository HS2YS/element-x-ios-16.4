# Build error inventory

Raw compiler output of iOS 16.4 simulator builds. It is the authoritative list of what breaks; `COMPAT-MATRIX.md` is the curated view of it.

- One file per build: `build-errors-<upstream tag or develop@sha>.txt`.
- Content: only `error:` lines (file:line:col: message), sorted and deduplicated, per scheme (ElementX, NSE, ShareExtension).
- After each file is added: reconcile `COMPAT-MATRIX.md` (new rows, counts, statuses) and note the total in `SYNC.md`.
- Compare two builds: `diff <(cut -d: -f4- a.txt | sort -u) <(cut -d: -f4- b.txt | sort -u)`.
