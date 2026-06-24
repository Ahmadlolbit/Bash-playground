# Project Ideas — Fun Stuff to Build

A grab-bag of things that are genuinely fun to make. Tuned to your style: small,
sharp tools with nice UX (TUIs, daemons, sensible config). Bash first, then a
pile of non-bash projects worth doing in other languages.

Each idea has a **difficulty** (🟢 weekend / 🟡 a few evenings / 🔴 a real project)
and a **why it's fun** note.

---

## Bash / Shell

### 🟢 `dotsync` — dotfiles manager with profiles
Symlink-based dotfile manager that supports machine profiles (laptop/desktop/server),
templated configs (inject hostname/secrets), and a `doctor` that reports drift.
**Fun:** the diffing + dry-run UX, and getting symlink edge cases exactly right.

### 🟢 `prj` — fuzzy project jumper + session restorer
`cd` into any project from a fzf list, auto-attach a tmux session with your usual
window layout per-project (editor, server, logs). Remembers last-opened.
**Fun:** instant payoff, you'll use it 50×/day.

### 🟡 `bkup` — incremental snapshot backup with a TUI
Wrap `rsync --link-dest` (or btrfs/zfs snapshots) into a friendly tool: scheduled
snapshots, retention policy (keep 7 daily / 4 weekly / 12 monthly), a whiptail
browser to restore a single file from any snapshot.
**Fun:** hardlink-based dedup feels like magic; the restore UI is satisfying.

### 🟡 `clip` — terminal clipboard history daemon
Background daemon watches the clipboard, stores history (with dedup + pinning),
fzf picker to re-paste. Works over Wayland (`wl-paste --watch`) and X11.
**Fun:** a daemon + picker combo, very much your wheelhouse after ijsi.

### 🟡 `watchdog` — generic file-watch task runner
`watchdog 'src/**/*.go' -- go build ./...` with debouncing, a status line, and
desktop notifications on failure. Like `entr` but with config files and a TUI log.
**Fun:** the debounce/coalesce logic is a nice little puzzle.

### 🟡 `menu` — build TUI menus from a YAML/JSON spec
A reusable whiptail/dialog/gum front-end: feed it a YAML tree of commands, get a
navigable menu with checklists, gauges, and form inputs. Basically extract the GUI
engine from ijsi into a standalone library others can use.
**Fun:** turning your one-off TUI code into a generic tool is genuinely useful.

### 🔴 `lazypkg` — unified package-manager TUI for Arch
One TUI over pacman + AUR helper: search, preview PKGBUILDs, batch install/remove,
orphan cleanup, mirror ranking, news feed before upgrades.
**Fun:** you're on Arch — this scratches a real itch and the UX surface is huge.

### 🔴 `ssh-mux` — SSH connection manager + jump-host orchestrator
Parse `~/.ssh/config`, present hosts in a TUI, manage ControlMaster sockets, show
which connections are live, one-key port-forward setups.
**Fun:** ControlMaster socket management is a fun systems puzzle.

---

## TUIs in a Real Language (Go / Rust)

Bash TUIs hit a wall fast. These are the same energy with a proper widget toolkit
(**Go: Bubble Tea / Lip Gloss**, **Rust: ratatui**).

### 🟡 A better `htop`-style monitor — but focused
Pick one thing and do it beautifully: GPU monitor, per-process network usage,
or disk I/O heatmap. Scoped > comprehensive.
**Fun:** real-time sparklines and smooth redraws are addictive.

### 🟡 Git "porcelain" TUI for one specific workflow
Not another lazygit — a TUI for *your* exact flow (e.g. interactive staging by hunk
+ conventional-commit builder + push). Opinionated and fast.
**Fun:** you'll feel every saved keystroke.

### 🔴 A terminal music player / podcast client
mpris integration, library browser, queue, scrobbling. ratatui + symphonia (Rust).
**Fun:** audio + UI + state management all at once.

---

## Web / Full-Stack

### 🟢 A personal dashboard ("start page")
Self-hosted homepage: weather, calendar, RSS, server status, quick links, bookmarks.
SvelteKit or plain HTML+HTMX.
**Fun:** instant gratification, infinitely customizable, you see it every day.

### 🟡 A self-hosted URL shortener + link analytics
Short links, QR codes, click stats, expiry. Go backend + SQLite + a tiny frontend.
**Fun:** clean small-scope full-stack; great for learning a new web framework.

### 🟡 A "read it later" / bookmark archiver
Save URLs, fetch + render reader-mode text, full-text search, tags. Like a tiny
self-hosted Pocket/Wallabag.
**Fun:** the content-extraction + search indexing is meaty and rewarding.

### 🔴 A live collaborative tool (cursor presence, shared doc)
WebSockets + CRDT (Yjs / Automerge). A shared whiteboard or markdown editor.
**Fun:** real-time sync is genuinely hard and very satisfying when it clicks.

---

## Systems / Lower-Level

### 🟡 A tiny load balancer / reverse proxy
Round-robin + health checks + sticky sessions, config-file driven. Go is perfect.
**Fun:** you learn how nginx actually works by rebuilding a slice of it.

### 🟡 A key-value store with a write-ahead log
In-memory hashmap + append-only log + compaction + a simple wire protocol.
Follow "Build Your Own Redis" energy.
**Fun:** WAL + compaction is the core of every real database; very illuminating.

### 🔴 A toy container runtime
Use Linux namespaces + cgroups + chroot to isolate a process. ~300 lines of Go/C.
**Fun:** demystifies Docker completely; "containers are just Linux features".

### 🔴 A network packet sniffer / protocol analyzer
Raw sockets / libpcap, decode Ethernet→IP→TCP, pretty-print. A mini tcpdump.
**Fun:** seeing bytes-on-the-wire turn into structured packets is a rush.

---

## Languages / Parsers / Interpreters

### 🟡 A calculator / expression language
Lexer → parser → evaluator. Add variables, functions, units (e.g. `3 GB / 2 hours`).
**Fun:** the classic "first compiler" project; clicks open a whole skill.

### 🔴 A small scripting language
Tree-walking interpreter from *Crafting Interpreters* (free book). Then add a
bytecode VM for round two.
**Fun:** one of the most rewarding "aha" projects in all of programming.

### 🟡 A regex engine
NFA construction (Thompson) + simulation. Match without backtracking.
**Fun:** elegant theory → working code; small but deep.

---

## Games / Graphics / Creative

### 🟢 A terminal game
Snake/Tetris/2048 in a TUI, or a roguelike with procedural dungeons.
**Fun:** game loops are a different kind of programming; very playful.

### 🟡 A particle system / physics sandbox
Falling-sand game (sand, water, fire interactions) in a canvas or with raylib.
**Fun:** emergent behavior from simple cellular-automata rules is mesmerizing.

### 🟡 A ray tracer
"Ray Tracing in One Weekend" (free). Output a PNG with spheres, reflections, shadows.
**Fun:** you write ~500 lines and produce a genuinely beautiful image.

### 🔴 A Game Boy / CHIP-8 emulator
Start with **CHIP-8** (a weekend), graduate to Game Boy. Emulate CPU, memory, display.
**Fun:** running real ROMs on code you wrote is an unmatched feeling.

---

## AI / Data (non-bash, but pairs well with CLI tools)

### 🟢 A CLI that summarizes / Q&As your files
Point the Claude API at a folder, ask questions, get cited answers. A tiny RAG tool.
**Fun:** immediately useful for your own notes/docs.

### 🟡 A "second brain" semantic search over your notes
Embed your markdown/notes, vector search, ask questions. Local embeddings + SQLite-vss.
**Fun:** search that understands meaning, not keywords, on your own data.

### 🟡 A commit-message / PR-description generator
Reads the diff, drafts a conventional commit or PR body. Hook it into git.
**Fun:** automates the most tedious part of your day; fits your tooling instinct.

---

## Picking one

If you want the **fastest dopamine**: `prj` (project jumper) or the personal dashboard.
If you want to **level up**: a small interpreter (Crafting Interpreters) or CHIP-8 emulator.
If you want **more of what you already do well**: extract `menu` (the TUI engine) out
of ijsi, or build `clip` (clipboard daemon) — both are natural next steps from your
existing bash tooling.
