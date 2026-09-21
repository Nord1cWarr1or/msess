# msess

**English** | **[Русский](README.ru.md)**

msess is an fzf-based session picker for MiMoCode. It lists all sessions from
all project directories — the built-in `/sessions` TUI view shows only sessions
of the current directory — searches titles and full conversation transcripts,
and opens the picked session right in the current terminal.

## Quick start

```bash
msess
```

The picker opens over the terminal: type to filter, `Enter` to open a session.

## Installation

```bash
cp msess ~/.local/bin/msess && chmod +x ~/.local/bin/msess
```

After editing the script, repeat the copy (or use a symlink: `ln -sf`).

## Usage

| Argument | Action |
|----------|--------|
| *(none)* | open the session picker |
| `-h`, `--help` | print usage and exit |

### Keys

| Key | Action |
|-----|--------|
| `Enter` | open the session (`exec mimo -s <id>`) in the folder set by `Ctrl-F` |
| `Ctrl-R` | rename the session |
| `Ctrl-D` | delete marked sessions (or the current one); `Enter` confirms, `n` cancels |
| `Tab` | mark a session for batch deletion |
| `Ctrl-T` | mode: titles ⇄ full-text content search |
| `Ctrl-F` | open-in folder: **current** ⇄ **session's saved folder** |
| `Esc` / `Ctrl-C` | quit |

In "session folder" mode the session opens in the directory it was last opened
from (if that directory is gone — in the current one). Opening a session in a
new place records the path: `session.directory` is updated to the actual folder.

All actions — rename, delete, and both toggles — run without leaving the list:
the picker is a single fzf process, the list refreshes in place, and the cursor
and search query always survive. The chosen modes (`Ctrl-T`, `Ctrl-F`) persist
between runs. `Tab` marks reset when the list reloads.

In content-search mode (`fts>`) the query goes to a full-text index (FTS5)
that matches **what was discussed** in the sessions, with hits highlighted.
The preview on the right: the **top** half shows the start of the session
(first messages; short sessions shown in full), a divider line marks the
middle, and below it the latest messages and search matches grow downward
toward the bottom edge.

## Configuration

The database path is resolved in a deliberate order: `$MIMOCODE_DB` → the
default path (honors `$MIMOCODE_HOME`) → `mimo db path`.

| Variable | Purpose | Default |
|----------|---------|---------|
| `MIMOCODE_DB` | path to `mimocode.db` | auto-detected |
| `MIMOCODE_HOME` | MiMoCode home directory; shifts the default DB path | `~/.local/share/mimocode` |
| `MSESS_STATE` | state file for the two picker modes | `~/.local/state/msess/state` |

The state file is rewritten atomically on every toggle, including toggles
issued from inside fzf.

## Data and storage

- Reads: SQLite `~/.local/share/mimocode/mimocode.db` — the `session` table
  for the list (`mimo session list` CLI is the fallback when the database is
  unavailable); the `history_fts` full-text index for content search, filtered
  to `user_text`/`assistant_text`; `message`/`part` rows for the preview.
- Writes: exactly two — `UPDATE session SET title=?` (rename) and
  `UPDATE session SET directory=?` (recording the folder of the last open).
- Never: direct `DELETE FROM` — deletion goes only through the official
  `mimo session delete`.

## Requirements

- bash
- fzf 0.36+ (enforced at startup)
- python3 3.10+
- MiMoCode CLI (`mimo`)

Verified on: fzf 0.74.3, Python 3.14, MiMoCode 0.1.13.

## License

Distributed under the GNU GPL v3.0. See `LICENSE` for more information.
