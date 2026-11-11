# devcontainer-go-sqlite

Dev container Go yang siap dipakai di PC mana pun. Buka folder, "Reopen in Container", langsung ngoding — tidak perlu install Go, sqlite3, atau tool pendukung lain di mesin host.

## Kenapa repo ini ada

Saya sering pindah distro, dan tiap distro punya cara sendiri dalam menyediakan Go:

- **Debian/Ubuntu**: `apt install golang-go` sudah cukup, dan `GOTOOLCHAIN=auto` bisa menarik toolchain yang lebih baru sesuai `go.mod` saat dibutuhkan.
- **Fedora**: Go-nya harus disetel manual supaya versi toolchain mengikuti kode yang sedang dikerjakan, tidak otomatis seperti di Debian.
- **Distro immutable** (mis. Silverblue, NixOS, openSUSE Aeon): package sistem read-only, jadi menambah runtime di host terasa mengganggu.

Daripada mengulang setup di setiap mesin, lingkungannya dipindah ke dalam container. Host cukup punya Docker; sisanya sudah dibekukan di image.

Awalnya image ini mencakup PHP, Java, Go, dan Rust sekaligus, tapi ukurannya jadi terlalu besar. Scope-nya sekarang dipersempit ke Go + SQLite saja.

## Isi image

| Komponen | Keterangan |
| --- | --- |
| Base | `debian:13` (trixie) |
| Go | `golang-go` dari apt + `GOTOOLCHAIN=auto` |
| SQLite | `sqlite3` CLI |
| Node.js | Node 24 lewat `nvm` (`~/.nvm`), plus global package: `npm@latest`, `command-code`, `serve`, `concurrently` |
| Docker CLI | `docker-ce-cli`, `buildx`, `compose` (daemon pakai socket host) |
| opencode | v2, terpasang di `~/.opencode/bin` |
| Claude Code | Installer resmi, launcher di `~/.local/bin` |
| Command Code | CLI `command-code`, settings dari `.devcontainer/commandcode-settings.json` |
| User | `agungdh` (UID/GID 1000), sudo tanpa password |
| Workspace | `/workspaces/<nama-folder-repo>` |

## Cara pakai

Prasyarat di host:

- Docker (daemon berjalan normal)
- VS Code + ekstensi **Dev Containers**

Langkahnya:

1. Clone repo yang mau dikerjakan, lalu buka foldernya di VS Code.
2. Command Palette → **Dev Containers: Reopen in Container** (atau "Clone Repository in Container Volume").
3. Image `ghcr.io/agungdh/devcontainer-go-sqlite:latest` di-pull otomatis, container dibuat dengan user `agungdh`.

Tidak perlu build image secara lokal. Image dibangun oleh GitHub Actions setiap ada perubahan di `.devcontainer/` dan di-push ke GHCR (`latest` untuk branch `main`, plus tag per commit SHA).

## Networking

Devcontainer berjalan dengan `--network=host`, dan container yang dijalankan dari dalamnya adalah **sibling** (memakai daemon host lewat `/var/run/docker.sock`), bukan child.

Karena itu:

- `docker run -p 8080:8080 ...` dari dalam devcontainer bisa diakses lewat `localhost:8080` dari dalam devcontainer juga.
- Service yang listen di host (mis. Postgres di `localhost:5432`) ikut terlihat dari dalam container.

Tanpa host networking, port yang di-publish sibling container hanya menempel di loopback host dan tidak bisa dijangkau lewat `localhost` dari devcontainer.

## Struktur repo

```
.devcontainer/
  Dockerfile                    # definisi image
  devcontainer.json             # konfigurasi container, mount, ekstensi
  commandcode-settings.json     # di-copy ke ~/.commandcode/settings.json
  .gitignore_global             # di-mount sebagai core.excludesFile
.github/workflows/
  devcontainer.yml              # build & push image ke GHCR
```

`.gitignore_global` mengabaikan folder editor/agent (`.claude/`, `.vscode/`, `.idea`, `.zed`, `.commandcode/`, `.DS_Store`) supaya tidak ikut ter-commit di repo mana pun.

`postCreateCommand` menyiapkan git (`safe.directory` + `core.excludesFile`) dan menyalin `commandcode-settings.json` ke `~/.commandcode/settings.json`, jadi mesin baru tidak perlu konfigurasi manual.

## Mengubah environment

1. Edit `.devcontainer/Dockerfile` atau `devcontainer.json`.
2. Commit dan push ke `main`; CI akan membangun image baru.
3. **Dev Containers: Rebuild Container** untuk memakai image terbaru.

Untuk uji coba cepat tanpa menunggu CI, build lokal juga bisa: `docker build -t ghcr.io/agungdh/devcontainer-go-sqlite:latest .devcontainer`, lalu sesuaikan `image` di `devcontainer.json` bila perlu.
