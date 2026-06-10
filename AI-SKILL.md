# WCP AI Release — AI Skill

> **Version 1.0.0** | For consumption by any AI engine releasing a WCP artefact.
> Source of truth: [github.com/penrithbeacon/wcp-ai-release](https://github.com/penrithbeacon/wcp-ai-release)

---

## 0. Universal Pipeline Rules

These rules apply to every release pipeline, regardless of artefact type.
They override any conflicting guidance in specialist sub-skills.

### 0.1 Explicit consent before every public push

> The developer is **personally responsible** for everything published under their
> identity. **Never push to a public registry or repository without explicit,
> per-action confirmation** — even in "live" mode. Each push is a separate consent
> action.

### 0.2 Mandatory completion summary

> **Every release pipeline, whether dry-run or live, MUST end with a written
> results summary delivered to the developer.** The developer cannot verify
> outcomes they did not personally observe. The AI must not assume the developer
> inferred the outcome from watching commands scroll past.

The summary must:
- State the artefact name and version released (or dry-run simulated)
- State the mode (Dry Run / Live) and stage (Beta / Release)
- List every public action taken with its result (✅ success / ⚠ warning / ❌ failure)
- Provide direct URLs to everything now publicly accessible (live mode)
- State clearly if anything was skipped or failed
- End with: **"The release pipeline is now complete. You (the developer) are responsible
  for everything now publicly visible."** (live mode) or **"Nothing was published
  publicly — run a live release when ready."** (dry-run mode)

See Sections 9/10 of the specialist release skills for the per-artefact result table format.

---

## 1. What This Repository Is

This is the **entry point** for releasing any WCP artefact — taking it from a
verified local build to a published, documented, production-ready component.

It does not perform the release itself. It orients and routes to the appropriate
specialist release skill based on what you have built.

**Prerequisites:** You must arrive here from a completed build pipeline:
- Widget: from [wcp-ai-build-widget](https://github.com/penrithbeacon/wcp-ai-build-widget)
  Section 6 mandatory handoff
- Agent: from [wcp-ai-build-agent-mac](https://github.com/penrithbeacon/wcp-ai-build-agent-mac)
  (or equivalent platform skill) Section 7 mandatory handoff

If you have not completed a build pipeline, go there first. Do not run the release
pipeline on an artefact that has not passed its build verification steps.

---

## 2. Skill: Determine What to Release

### ⚠️ Mandatory rule — explicit consent required before every public push

> The developer is **personally responsible** for everything published under their
> Docker Hub account and GitHub identity. The AI must **never** push to a public
> registry or public GitHub repository without the developer's explicit, per-action
> confirmation — even when running in "live" mode.

**Live mode** means the pipeline *can* push; it does not mean the AI *should* push
without asking. Before every command that would make something publicly visible
(docker buildx push, git push with a tag, gh release create, Docker Hub description
update), the AI must pause and ask:

> _"I'm about to push X to Y. This will be publicly visible. Do you want me to proceed?"_

The developer must answer yes before the command runs. This applies even if the
developer previously said "live mode" — that sets the capability, not the blanket
authorisation. **Do not batch multiple public pushes into a single question.**
Each push is a separate action requiring separate confirmation.

A dry-run release is always safe to run without asking, as it never touches public
registries.

---

### Step 1 — Establish release stage and mode

Before routing to any specialist skill, ask the developer two questions:

**Question A — Release stage:**

> _"Is this a beta release (first public release, for broader testing) or a
> production release (stable, ready for general use)?"_

| Stage | Meaning | Docker Hub tags | GitHub repo |
|-------|---------|----------------|-------------|
| **Beta** | First public release; graduating from alpha kiosk to beta kiosk | `{version}-beta`, `beta` | Public on penrithbeacon |
| **Release** | Production release; graduating from beta kiosk to release kiosk | `{version}`, `latest` | Public on penrithbeacon |

**Question B — Release mode:**

> _"Would you like to run a dry run first (validates everything but does not push
> publicly), or go straight to a live release?"_

| Mode | Meaning |
|------|---------|
| **Dry run** | Executes every step — builds, audits, generates all documents — but does **not** push to public GitHub or Docker Hub. Proves release-readiness without publishing. Reports exactly what a live release would do. |
| **Live** | Full execution. All public pushes are made. After a live release, the artefact is publicly available. |

Record both answers. They flow through to the specialist release skill and govern
whether publication steps are executed or reported-only.

**Recommended flow for a first release:**
1. Run a **dry-run beta release** — validate everything is in order
2. Fix any issues found
3. Run a **live beta release** — publish to Docker Hub as beta, launch from beta kiosk
4. Test in beta; iterate as needed
5. Run a **dry-run release** — validate production readiness
6. Run a **live release** — publish as stable

### Step 2 — Route to the specialist skill

| Developer has built | Route to |
|--------------------|----------|
| A widget only | [wcp-ai-release-widget AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-release-widget/blob/main/AI-SKILL.md) |
| A companion agent only | [wcp-ai-release-agent AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-release-agent/blob/main/AI-SKILL.md) |
| A widget + companion agent together | Run **both** release pipelines. Run wcp-ai-release-agent first (agent installer must be built before it can be bundled into the widget image). Then run wcp-ai-release-widget with the agent installer already in `src/installers/`. |
| A shipped application (.WCPA → native app) | wcp-ai-release-ship — *coming soon. See [WCP-AI-SHIP.md](https://github.com/HarrisonOfTheNorth/claude-dashboard/blob/working/WCP-AI-SHIP.md) for the concept.* |

**Before routing, confirm** the developer has these items ready:
- Credentials file path (GitHub PAT + Docker Hub token)
- Artefact name, GitHub path, and (for widgets) Docker Hub path
- Confirmed: alpha kiosk QA is complete and the widget is ready to graduate

**Announce the routing decision** — state the stage, mode, and artefact type before
proceeding so the developer can correct you if anything is wrong.

---

## 3. Widget + Agent Pair Release Order

When releasing a companion widget + agent together, the order matters:

```
1. wcp-ai-release-agent
   → builds .pkg installer
   → publishes to GitHub Releases
   → copies .pkg to widget's src/installers/

2. wcp-ai-release-widget
   → widget image is built with the .pkg already inside
   → GET /widget/agent/installer serves the correct installer
   → publishes image to Docker Hub

3. Companion pair audit
   → GET /widget/agent/installer returns 200 (not 503)
   → GET /widget/api/agent/status correctly detects the agent
   → agent's /agent/wcp companion_widget field matches widget name
```

---

## 4. WCP Ecosystem Context

| Artefact | Published to | Version format |
|----------|-------------|----------------|
| Widget | Docker Hub — `<org>/wcp-widget-<name>:<version>` | Semantic (e.g. `1.0.0`) |
| Agent | GitHub Releases — `.pkg` attached to release tag | Semantic (e.g. `v1.0.0`) |
| Shipped app | GitHub Releases — `.app`/platform binary | Semantic (e.g. `v1.0.0`) |

---

## 5. Cross-References

| Repository | Purpose | AI Skill |
|-----------|---------|---------|
| [wcp-ai-release](https://github.com/penrithbeacon/wcp-ai-release) | Entry point — this repo | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-release/blob/main/AI-SKILL.md) |
| [wcp-ai-release-widget](https://github.com/penrithbeacon/wcp-ai-release-widget) | Release a WCP widget | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-release-widget/blob/main/AI-SKILL.md) |
| [wcp-ai-release-agent](https://github.com/penrithbeacon/wcp-ai-release-agent) | Release a WCP agent | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-release-agent/blob/main/AI-SKILL.md) |
| [wcp-ai-build](https://github.com/penrithbeacon/wcp-ai-build) | Build entry point | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build/blob/main/AI-SKILL.md) |
| [wcp-ai-release-widget](https://github.com/penrithbeacon/wcp-ai-release-widget) `standards/` | Widget build spec, audit checklist, release workflow, index format | Migrated from retired wcp-ai-automation |
