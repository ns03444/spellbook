# Phase 0: Vendor Excalidraw and deploy it live

Spellbook starts as a **vendored copy of [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw)** (MIT) inside this repo, then run via **Docker** so a live whiteboard is reachable under your domain. Later phases add accounts, storage, collab policy, AI, and teams on top.

This doc is the working plan for Phase 0 only. See [README](./README.md) for the broader research and phase table.

---

## Goals

- Put Excalidraw **source** into `ns03444/spellbook` (not a separate GitHub Fork repo).
- Keep the **MIT license and copyright notices** intact.
- Run the app with the **official Excalidraw Docker path** (build image from the vendored tree, or use `excalidraw/excalidraw` as a reference while you wire your own build).
- Deploy somewhere public so you can open a whiteboard URL and draw.
- Leave a clear baseline for Phase 1 (auth, cloud scenes, etc.).

## Non-goals (Phase 0)

- npm embed of `@excalidraw/excalidraw` as the product path (useful later for a custom shell; **not** Phase 0).
- Creating only a GitHub **Fork** of excalidraw under your account without putting code in this repo.
- Obsessing over which host (any VPS or container host that can run Docker is fine).
- Live collaboration / share rooms (needs a Socket.IO relay such as [excalidraw-room](https://github.com/excalidraw/excalidraw-room); treat as **out of scope or later**).
- Accounts, cloud sync, billing, AI, teams, SSO.
- Deep editor forks or rebranding beyond a light Spellbook identity if you choose.

---

## Prerequisites

- GitHub access to push `ns03444/spellbook` (main).
- Docker installed locally (and on the host you deploy to).
- Basic familiarity with clone / rsync / `git subtree`.
- A host that can run a container and expose HTTP(S) (VPS, Fly, Railway, ECS, k8s, etc.). Host choice is intentionally light for Phase 0.
- Optional: domain + TLS terminator (Caddy, nginx, Traefik, or host-managed HTTPS).

Upstream: https://github.com/excalidraw/excalidraw (MIT).

---

## How to get the code into this repo

GitHub's **Fork** button creates a *separate* repo under your account. Phase 0 wants the Excalidraw tree **inside spellbook**.

### Option A: rsync / copy (simple, recommended for first land)

1. Clone upstream somewhere outside this repo:

```bash
git clone --depth 1 https://github.com/excalidraw/excalidraw.git /tmp/excalidraw
```

2. Copy into a clear path in spellbook (example layout):

```bash
mkdir -p vendor/excalidraw
rsync -a --delete \
  --exclude .git \
  /tmp/excalidraw/ vendor/excalidraw/
```

3. Commit the vendored tree. Prefer keeping upstream `LICENSE` / copyright files visible (see MIT section below).

### Option B: git subtree (keeps upstream history linkable)

```bash
git remote add excalidraw https://github.com/excalidraw/excalidraw.git
git fetch excalidraw
git subtree add --prefix=vendor/excalidraw excalidraw master --squash
```

Later updates:

```bash
git fetch excalidraw
git subtree pull --prefix=vendor/excalidraw excalidraw master --squash
```

### Not Phase 0

| Approach | Why not for Phase 0 |
| --- | --- |
| GitHub Fork button only | Separate repo; spellbook still has no app code |
| `npm i @excalidraw/excalidraw` | Embeds the React editor without vendoring the app; different product shape |

Pick A or B; both satisfy "code lives in spellbook." Document which you used in the commit message.

---

## MIT notice

Excalidraw is **MIT**. You may fork, modify, and ship commercially, but you must **retain copyright and permission notices**.

Practical checklist:

- Keep upstream `LICENSE` (or equivalent) in the vendored tree.
- Do not strip copyright headers from files you modify unless you know the project policy; when in doubt, leave them.
- In the spellbook root README (or a short `NOTICE`), state that the whiteboard is based on Excalidraw and point at the MIT license under `vendor/excalidraw` (or whatever path you chose).

---

## Step-by-step

### 1. Vendor the code

- Use Option A or B above into e.g. `vendor/excalidraw/`.
- Confirm the monorepo layout is present (`packages/`, `excalidraw-app/` or current upstream structure).
- Commit on `main` with a message that names the upstream commit/tag you vendored.

### 2. Keep license files

- Verify `LICENSE` (and any `NOTICE`) survived the copy.
- Add a one-line attribution in the root README if not already there.

### 3. Run with Docker (local first)

Official pattern (from Excalidraw docs / Docker Hub `excalidraw/excalidraw`):

```bash
# From the vendored tree (or after docker build -t spellbook-excalidraw .)
docker build -t spellbook-excalidraw .
docker run --rm -dit --name spellbook-excalidraw -p 5000:80 spellbook-excalidraw
```

Then open `http://localhost:5000`. You should get the static whiteboard client (no analytics in the official image).

Notes:

- The published image `excalidraw/excalidraw:latest` is a useful reference: `docker pull excalidraw/excalidraw` then map `-p 5000:80`.
- Phase 0 success is **your** image built from the vendored tree (or an explicitly documented interim use of the official image while the build is wired). Prefer building from vendor so this repo owns the runnable artifact.
- Standalone drawing works without a backend. Collaboration does not.

### 4. Deploy live (host choice light)

- Push the same Docker image to any registry your host uses, or build on the host from this repo.
- Run the container with a published port (or behind a reverse proxy).
- Point a hostname at it; terminate TLS at the proxy or platform.
- Examples of acceptable hosts: any small VPS with Docker, or a managed container platform. Do not block Phase 0 on picking the "perfect" cloud.

Minimal mental model:

```text
Browser --> HTTPS (proxy / platform) --> container :80 (Excalidraw static app)
```

### 5. Optional: collab relay (out of scope / later)

Live rooms need a separate Socket.IO service (e.g. [excalidraw-room](https://github.com/excalidraw/excalidraw-room)) and client config pointing at your relay URL, plus HTTPS for Web Crypto. Official self-host docs note that the stock Docker client does **not** include sharing/collab out of the box.

**Phase 0:** skip collab. Document as a Phase 1+ follow-up if you want multiplayer before custom Spellbook backends.

---

## Success criteria

Phase 0 is done when all of the following are true:

1. Excalidraw source (or the agreed subset) lives under this repo (e.g. `vendor/excalidraw/`), not only as a separate GitHub Fork.
2. MIT / copyright notices are retained and attributed.
3. `docker build` + `docker run` (or equivalent compose) starts the whiteboard from that tree.
4. A public URL loads the editor; you can draw and export/save locally as the stock app allows.
5. README Phase 0 one-liner matches this plan and links here.
6. Collab relay is either absent or clearly marked optional / later (not required to call Phase 0 done).

---

## Checklist

- [ ] Clone or subtree upstream into `vendor/excalidraw` (or chosen path)
- [ ] Confirm `LICENSE` / copyright notices present
- [ ] Root README attributes Excalidraw + MIT; Phase 0 links to this file
- [ ] `docker build` succeeds from vendored tree
- [ ] Local `docker run -p 5000:80 ...` loads the app
- [ ] Image (or build-from-git) deployed on a host; public HTTPS URL works
- [ ] Smoke test: draw, undo, export PNG/SVG or `.excalidraw` as supported
- [ ] Note in README or this file: collab relay deferred
- [ ] Tag or commit message records upstream revision vendored

---

## After Phase 0

Phase 1 (from root README): auth, scene CRUD + cloud storage, share links, basic live collab, org invites. Use the vendored app as the canvas, then decide whether to keep the full `excalidraw-app` shell or migrate toward a Spellbook shell that still embeds / builds on the same packages.
