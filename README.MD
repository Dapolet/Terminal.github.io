# WebTerminal

A self-contained, single-file browser terminal. No build step, no dependencies, no server — open `index.html` in any modern browser and you get a working Unix-style shell with a simulated file system, editors, a Python interpreter, networking, and multi-terminal tiling. Everything (files, config, theme, font) persists in the browser's `localStorage`.

## Highlights

- **One file, zero deps** — all app code lives in one `<script>` IIFE. Works fully offline except for the network commands and Python.
- **Real shell behavior** — shell operators (`&&`, `;`, `|`, `>`, `>>`), pipes with stdin consumers (`cat`, `grep`, `xargs`), aliases, env vars, and shell options (`set -e`/`-u`/`-x`).
- **Simulated file system** — directories, files, permissions-free; lives in `localStorage` and survives reloads.
- **Real editors** — `nano` and `vim` with a shared editing engine.
- **Real Python** — CPython via Pyodide (lazy-loaded from a CDN, only when you run `python`); the terminal FS is mirrored at `/mnt/webterm`.
- **Tiling terminal host** — spawn multiple terminals in iframe panels (`+ terminal`), all sharing one FS, theme, and font with live cross-terminal sync.
- **Real ZIP** — `zip`/`unzip` use genuine STORE-method archives (CRC32) so they interoperate with real tools when exported.

## Quick start

```sh
# just open it
start index.html
```

Or serve it:

```sh
python -m http.server 8000   # then visit http://localhost:8000
```

## Getting around

| Task | Command |
|---|---|
| List files | `ls`, `ls -la` |
| Navigate | `cd`, `cd ..`, `cd ~`, `pwd` |
| Files & dirs | `touch`, `mkdir`, `cat`, `mv`, `cp`, `rm -rf`, `find`, `tree` |
| Edit files | `nano file`, `vim file` (or `edit file` — respects `$EDITOR`) |
| Config | edit `~/.bashrc`, or use `export` / `alias` (both persist there) |
| Python | `python` (REPL), `python -c "code"`, `python file.py` |
| Network | `curl`, `wget`, `ping`, `ip`, `nslookup`, `dig`, `open <url>` |
| Archives | `zip -r out.zip dir`, `unzip out.zip`, `unzip -l out.zip` |
| Download | `download file`, `download dir`, `download --fs`, `download history` |
| Shell | `time`, `watch -n 2`, `xargs`, `seq`, `yes`, `sleep`, `sudo` (with a sense of humor) |
| Appearance | `theme green`, `font 16`, `matrix on` |
| System | `reboot` (soft restart, keeps `~/pers` + `~/.bashrc`), `reset` (full wipe), `neofetch`, `uname` |
| Help | `help`, `help <command>` |

Type `help` in the terminal for the full command list, and use **Tab** to autocomplete commands and paths. **Ctrl+L** clears the screen, **↑/↓** walks history.

## How it works

- **Shell** — commands are registered as `commands.<name> = { help, usage, fn(args, stdin) }`. A command line is tokenized (quote-aware), split on top-level operators, and each segment runs with the previous pipe segment's captured output as `stdin`. Only synchronous `print()` output is capturable by pipes/redirects.
- **File system** — a tree of `{ dirs, files }` nodes where files are strings. Paths are arrays; `""` is the root marker. The whole tree is JSON-serialized to `localStorage` (`webterm_fs_v5`) on every mutation.
- **Config** — `~/.bashrc` inside the FS holds `export NAME=value` and `alias name=value` lines, applied at boot. It survives `reboot`; `reset` restores the default template.
- **Sync** — every terminal writes its changes to `localStorage` and broadcasts them via `postMessage` to sibling panels and the host page, so all open terminals stay in sync instantly.
- **Python** — Pyodide is loaded lazily on first use and mirrors the terminal FS at `/mnt/webterm`, so files you create in Python show up in the terminal and vice versa.

## Browser support

Any modern evergreen browser (Chrome, Firefox, Safari, Edge). `TextEncoder`, `fetch`, `AbortController`, and `Blob` downloads are used where available; everything except network/Python works offline.

## Project structure

```
index.html          the entire application (HTML + CSS + JS in one file)
README.md           this file
```

## Development notes

- No build tools, no package manager, no test runner in-repo.
- The regression harnesses are Node scripts with hand-rolled DOM stubs that live outside the repo in `%TEMP%\opencode\` (`harness.js`–`harness4.js`, plus `probe_*.js`). They extract the `<script>` block from `index.html` and `eval` it against stubbed DOM. Run them with `node`; keep them green when changing the app.
- Follow the existing style when editing: legacy `var`, one IIFE, no comments unless asked. See `AGENTS.md` for the full editing conventions.
