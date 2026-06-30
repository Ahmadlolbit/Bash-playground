# ijsi — JetBrains shared-index manager for PHP API forks

`ijsi` builds [JetBrains **shared indexes**](https://www.jetbrains.com/help/phpstorm/shared-indexes.html)
for a folder of forked PHP/Laravel API repositories and publishes them to a
MinIO (S3-compatible) bucket. When a developer opens one of those projects in
PhpStorm, the IDE downloads the prebuilt index instead of analysing the project
— and its `vendor/` dependencies — from scratch.

It's a single Bash script: [`src/ijsi`](src/ijsi). Run it with no arguments for
a terminal UI, or `ijsi help` for the CLI.

---

## The idea

You keep one folder of forks (default `~/Main`), one repo per API project. On
demand, `ijsi refresh` walks that folder and, **for each repo**:

1. **`git fetch` + fast-forward** the default branch (`main`/`master`),
2. **`composer install`** so `vendor/` matches the committed `composer.lock`,
3. **generates a shared index** with the PhpStorm CLI, and
4. **uploads** it to your MinIO bucket (`mc mirror`, incremental).

Each repo also gets an `intellij.yaml` pointing PhpStorm at the bucket. That
file is delivered to the repo **as a pull request** ("intellij.yaml fix") that
you review and merge, so the whole team inherits the shared-index config.

```
~/Main/
├── api-billing/     ── fetch+ff main → composer install → index → upload
├── api-identity/    ─┘
└── api-orders/
                         indexes ─► MinIO bucket ─► every dev's PhpStorm
```

### Why this makes `vendor/` stop being re-analysed

PhpStorm treats every `vendor/*` package as a **library root** and indexes it
even though it's "excluded" content. Shared indexes are matched to a developer's
files **by the hash of each file's contents**, not by path — so a `vendor` file
is only covered when its bytes are identical to the bytes present when the index
was built.

By indexing **after `composer install`**, the published index covers `vendor/`.
A teammate gets that coverage as long as they `composer install` from the **same
`composer.lock`** on a compatible PHP version/platform — the normal case.

> **Caveat (a JetBrains constraint, not a bug):** coverage is by content hash, so
> a teammate whose `vendor/` differs (different PHP version, platform-specific
> packages, an out-of-date `composer install`) will have those differing files
> indexed locally. Keep everyone on the same committed `composer.lock` for the
> best hit rate. JetBrains also auto-downloads its own prebuilt indexes for
> *popular versions* of common packages (Laravel, Symfony, Doctrine, PHPUnit, …)
> from `composer.json`, independently of `ijsi`.

### When it re-indexes

Re-indexing is **on demand** (`ijsi refresh`); there is no timer. A repo is
re-indexed only when its **fingerprint** changes. The fingerprint is a hash of:

- the committed tree of `origin/<branch>` **excluding `intellij.yaml`**,
- `composer.lock` + `composer.json` (so **new packages always re-index**),
- the PhpStorm build, and
- the published base-url.

`intellij.yaml` is deliberately excluded, so **merging the PRs `ijsi` opens
never triggers a re-index loop**. `ijsi refresh --force` ignores the fingerprint
and re-indexes everything.

---

## Requirements

| Tool | Why | Config var |
|------|-----|------------|
| `ij-shared-indexes-tool-cli` | generates the indexes (JetBrains) | `CLI` |
| PhpStorm | provides the indexer engine (`--ij`) | `PHPSTORM_IDE` |
| `composer` | installs `vendor/` before indexing | `COMPOSER` |
| `mc` (MinIO Client) | uploads indexes to the bucket | `MC` |
| `gh` (GitHub CLI) | opens the `intellij.yaml` PRs | — |
| `git` | fetch / fast-forward / worktree PR | — |
| `whiptail` or `dialog` | optional terminal UI | — |

---

## Quick start

```bash
# 1. Put your forked API repos in one folder (default ~/Main), then configure:
ijsi config edit          # set REPOS_DIR and S3_BUCKET

# 2. Store MinIO credentials (kept in mc, never in ijsi's config):
ijsi s3 login             # endpoint + access key + secret key

# 3. Create the bucket and grant anonymous (public) download:
ijsi s3 setup

# 4. Index everything and upload:
ijsi refresh              # → review & merge each "intellij.yaml fix" PR

# Health check whenever something looks off:
ijsi check
```

Then, in PhpStorm, opening each project will offer to download the shared
indexes from the bucket (once `intellij.yaml` is merged).

---

## CLI

```
ijsi                      Open the terminal GUI
ijsi refresh [opts]       Update + index changed repos, then upload
                            --force, -f      re-index everything
                            --only a,b,...   only repos matching these names
                            --no-upload      index locally, skip the upload
ijsi s3 [sub]             MinIO: status | login | setup | sync | test
ijsi config [sub]         Config: show | path | init | edit
ijsi list                 List discovered repos (name, branch@HEAD, path)
ijsi logs                 Show the newest run log
ijsi reset-state [name]   Forget fingerprints (all, or one repo)
ijsi prune-logs [keep]    Delete old logs, keep newest <keep> (default 100)
ijsi clear-cache          Remove the whole cache directory
ijsi check | doctor       Detected tools, PhpStorm, repos, MinIO readiness
ijsi version
ijsi help
```

---

## Configuration

Config lives at `${XDG_CONFIG_HOME:-~/.config}/ijsi/config` (a sourced Bash
file, mode `0600`). Any `IJSI_<VAR>` environment variable overrides it.
**S3 credentials are never written here** — they live in `mc`'s own store
(`~/.mc/config.json`).

| Variable | Default | Meaning |
|----------|---------|---------|
| `REPOS_DIR` | `~/Main` | Folder of forked API repos (one per subfolder) |
| `PHPSTORM_IDE` | `auto` | PhpStorm install dir, or `auto` (Toolbox detection) |
| `DEFAULT_BRANCH` | *(auto)* | Branch to track; blank = `origin/HEAD`, then `main`/`master` |
| `INTELLIJ_YAML_VCS` | `pr` | `pr` (open a PR) · `leave` (write only, you commit) · `off` (don't touch git) |
| `CLI` | `ij-shared-indexes-tool-cli` | Indexer CLI command/path |
| `COMPOSER` | `composer` | Composer command/path |
| `MC` | `mc` | MinIO Client command/path |
| `CACHE_DIR` | `~/.cache/ij-shared-indexes` | Generated indexes, state, logs |
| `S3_ALIAS` | `ijsi` | `mc` alias holding the credentials |
| `S3_ENDPOINT` | — | MinIO API endpoint URL |
| `S3_BUCKET` | — | Target bucket |
| `S3_PREFIX` | — | Optional key prefix inside the bucket |
| `PUBLIC_BASE_URL` | *(derived)* | Override the public download URL (CDN/proxy) |

---

## The `intellij.yaml` PR flow

`intellij.yaml` must be committed so the team inherits the shared-index config,
but `ijsi` never commits to your default branch directly. When the generated
`intellij.yaml` differs from the branch, `ijsi`:

1. builds a stable branch **`ijsi/intellij-yaml`** off `origin/<branch>` in a
   throwaway git worktree (your repo's own checkout is never disturbed),
2. force-pushes it, and
3. opens (or updates) a PR titled **"intellij.yaml fix"** via `gh`.

It's idempotent: nothing happens when the branch already carries the current
`intellij.yaml`, and an existing open PR is reused rather than duplicated. Set
`INTELLIJ_YAML_VCS=leave` to skip the PR and just write the file (you commit it),
or `off` to leave git untouched.

---

## Notes & limitations

- **PHP/PhpStorm only.** Multi-IDE/language detection, the local test server,
  the `boost` benchmark, and the systemd units from older versions were removed
  in v4 — this tool does one thing.
- **Same IDE build on both ends.** The machine that builds an index and the
  machines that consume it must run the same PhpStorm build (the OS may differ).
- **Forks are kept clean.** A repo with uncommitted local changes (other than
  the regenerated `intellij.yaml`, which `ijsi` discards before pulling) or a
  diverged branch is skipped with a clear message rather than force-updated.

---

A native Qt/QML + Kirigami port of this tool is specified in
[`NativeApp.md`](NativeApp.md).
