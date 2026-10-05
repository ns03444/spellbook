# spellbook

Research notes and plan for building a premium Excalidraw-like whiteboard.

Sources checked live (Oct 2026): [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw), [plus.excalidraw.com](https://plus.excalidraw.com/excalidraw-for-teams), [plus AI](https://plus.excalidraw.com/ai), [plus security](https://plus.excalidraw.com/security-and-compliance), [tldraw](https://github.com/tldraw/tldraw), [tldraw license key](https://tldraw.dev/sdk-features/license-key).

---

## 1. Free Excalidraw (OSS)

| Item | Detail |
| --- | --- |
| License | **MIT** (fork, modify, sell OK; keep copyright notice) |
| Repo | Monorepo: `packages/excalidraw` + `excalidraw-app` |
| `@excalidraw/excalidraw` | Embeddable React editor. No backend. Canvas, tools, libraries, i18n, PNG/SVG/clipboard, `.excalidraw` JSON |
| `excalidraw-app` | excalidraw.com shell: PWA, live collab, E2E encryption, local autosave, readonly share links |
| Self-host | Official Docker serves static app. Standalone whiteboard works out of the box |
| Collab self-host | Needs separate Socket.IO relay ([excalidraw-room](https://github.com/excalidraw/excalidraw-room) or compatible) + storage backend (Firebase or HTTP, e.g. community storage servers). HTTPS required for Web Crypto |
| Collab architecture | Client encrypts with AES-GCM. Room key stays in URL fragment (never sent to server). Relay broadcasts ciphertext. Fractional indexing reconciles concurrent edits |

**You get:** editor + optional collab/share if you wire servers.

**You must build yourself for a "premium" product:** accounts, cloud sync, orgs/teams, RBAC, comments, presentations mode, voice/screenshare, PDF/PPTX export, server-side libraries, AI, admin, audit, SSO, billing UI (product side only listed below).

---

## 2. Excalidraw+ product features (no pricing)

From [plus comparison](https://plus.excalidraw.com/excalidraw-for-teams), [AI](https://plus.excalidraw.com/ai), [security](https://plus.excalidraw.com/security-and-compliance):

**Workspace / teams**
- User accounts, cloud-saved scenes, auto sync, dashboard
- Workspace teams, user management, collections
- Personal library saved on server (free: browser-only). Workspace libraries: marked **soon**

**Collaborate**
- Invite by link, view-only access (also on free)
- Comments
- Voice hangout and screensharing
- Live real-time presentations; presentations as slides

**Create / share**
- Generative AI (Plus: extended vs free capped)
- PDF and PPTX export
- Embeddable / readonly links (also on free for basic share)

**AI** ([/ai](https://plus.excalidraw.com/ai))
- Text-to-diagram, Mermaid → native shapes (Mermaid: no AI quota)
- Free editor: capped (~10 AI requests/day per their page)
- Plus: higher Excalidraw-key quota, API + MCP server
- BYOK (OpenAI, Anthropic, Gemini, Grok, OpenRouter); admin model/permission controls
- Read vs write keys; personal vs workspace key scopes

**Privacy / security**
- Plus: AES-256 at rest, TLS in transit; app-level encryption for tokens/keys
- Free shared rooms: E2E encryption; local-first storage
- RBAC, admin controls (sharing, AI), SOC 2 Type I & II (claimed on security page)
- Daily DB backups, 14-day PITR (claimed)

Roadmap items (not shipped as of check): workspace libraries, library search, SSO, self-host Enterprise, domain capture - treat as unverified until live.

---

## 3. Excalidraw vs tldraw (for your own premium whiteboard)

| | Excalidraw | tldraw |
| --- | --- | --- |
| License | MIT (true OSS) | Custom / source-available. Dev free; **production needs license key** (trial / commercial / hobby+watermark) |
| React | `@excalidraw/excalidraw` component | First-class SDK (`tldraw`), Editor API |
| Collab | App-layer Socket.IO + E2E; not in npm package by default | `@tldraw/sync` (+ sync-core server); self-hostable |
| Extensibility | Customizable UI/props; less of a full canvas engine | Custom shapes, tools, bindings, UI, side effects; starter kits (some MIT) |
| Ecosystem | Huge familiarity, Obsidian/Notion embeds, libraries | Strong SDK docs, agents/workflow kits, DOM canvas embeds |
| Commercial model | Free core + Plus SaaS | SDK license is the product |

**Rule of thumb:** Excalidraw for MIT + familiar sketch UX + lowest legal friction. tldraw for product-canvas SDK power if you accept commercial licensing.

---

## 4. Plan: build Spellbook (premium Excalidraw-like)

### Recommendation

**Phase 0: fork Excalidraw into this repo and deploy it live.** Later phases add Spellbook product layers on top (accounts, storage, collab policy, AI, teams).

- MIT keeps legal friction low vs tldraw production licensing.
- Alternative: **tldraw** if custom shapes/tools are the product (budget for commercial license).
- npm `@excalidraw/excalidraw` remains an option if you only need the editor component (not a full app fork).

### Getting Excalidraw into this repo

- **Preferred (code inside spellbook):** Clone [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw), copy the tree you need (often the whole monorepo or selected packages) into this repo. Or use `git subtree` / add a remote and pull from that URL.
- **GitHub Fork:** Creates a separate fork under your account. You can rename or mirror into spellbook, but a fork of excalidraw is not the same as putting the code inside this repo.
- **npm `@excalidraw/excalidraw`:** Embeds the editor without vendoring source. Not a full fork of the app.

### Architecture phases

| Phase | Scope |
| --- | --- |
| **0 - Live fork** | Fork Excalidraw into the spellbook repo and deploy it live |
| **1 - MVP** | Auth (email/OAuth), scene CRUD + cloud storage, share links (view/edit), basic live collab (WS relay), org with invites |
| **2 - Team** | Collections, roles (owner/editor/viewer), comments, server libraries, audit log |
| **3 - Premium parity** | Presentations, PDF/PPTX export, voice/screenshare (WebRTC or vendor), AI text→diagram + BYOK + optional MCP |
| **4 - Hardening** | E2E or at-rest encryption choice, SSO, backups, SOC2-oriented controls, rate limits |

**Suggested stack:** React + Vite or Next.js; Postgres (+ object storage for images); Redis; Socket.IO or PartyKit/CF Durable Objects for rooms; Auth.js/Clerk/etc.; LLM gateway for AI.

### Feature backlog ↔ free / Plus gaps

| Feature | Free OSS | Plus | Spellbook |
| --- | --- | --- | --- |
| Editor canvas | Yes (pkg / app) | Yes | Fork app (Phase 0); pkg optional |
| Live collab | App + self-host relay | Yes | Build relay + presence |
| Cloud sync / accounts | No | Yes | **MVP** |
| Teams / RBAC / collections | No | Yes | **MVP → Team** |
| Comments | No | Yes | Team |
| Presentations / live present | No | Yes | Later |
| Voice / screenshare | No | Yes | Later (or integrate) |
| PDF / PPTX export | No | Yes | Later |
| Server libraries | Browser / public | Server personal; workspace soon | Team |
| AI + MCP / BYOK | Limited free | Extended | Later |
| SSO / Enterprise self-host | No | Roadmap | Phase 4 |

### Non-goals (initially)

Pixel-perfect clone of Plus UI. Competing on brand libraries marketplace. Shipping billing before MVP collab + cloud scenes work.

---

## Status

Research README only. Implementation not started.
