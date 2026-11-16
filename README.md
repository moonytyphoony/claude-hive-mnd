# Claude Hive MNO

**Multi-Agent Network Orchestrator for Claude Code**

An independent, open-source Claude Code plugin and browser-based knowledge graph for **parallel research**, **context optimization**, and **portable community personality profiles**.

> **Beta / experimental:** This is a community project, not an official Anthropic product. Features are at different stages of maturity. In particular, measured end-to-end token savings, live Claude Code multi-agent performance, and cloud-provider integration have not yet been independently validated.

## Why Claude Hive?

AI coding assistants can waste context rereading information, and researching several sources can be slow. Claude Hive brings together a visual knowledge graph, source importers, optional context-condensing hooks, and a bounded multi-agent research workflow. The goal is to help developers **reuse relevant knowledge** and **choose the right amount of work for each task**.

## Features at a glance

| Component | What it does | Status |
|---|---|---|
| **Hive Graph** | Visualize, search, edit and link notes in a browser | Implemented |
| **Hive ECO** | Opt-in log condensation and targeted context retrieval | Implemented; live savings unverified |
| **Hive MNO** | Run up to 8 parallel Claude CLI source-analysis processes | Prototype; requires local Claude Code |
| **Research importers** | Add wiki pages, public forum threads, local text and YouTube transcripts | Implemented with source restrictions |
| **Community personalities** | Export/import shareable profile JSON with tone and preferences | Implemented; not auto-applied to Claude Code |
| **Cloud adapters** | Manual save/load integrations for supported cloud services | Requires provider setup and live validation |
| **Hive Market** | Separate future website for sharing profiles | Planned, not included |

## Quick start

**Requirements:** A modern browser. Node.js for CLI tools and tests. An authenticated Claude Code installation for `--analyze` multi-agent runs.

```bash
# From the repository root:
npm test

# Run the static web interface:
python -m http.server 8080
# Then open http://localhost:8080
```

### Research multiple sources in parallel

Edit `sources.example.json` to specify supported sources. **YouTube video processing currently requires a local transcript file** (`.txt` or `.vtt`); Hive does not yet interpret video frames or audio.

```bash
node scripts/hive-agents.mjs sources.example.json hive.json --agents 3 --analyze
```

The `--analyze` option invokes the local `claude` CLI for each source, with tool use disabled for those processes. Omit `--analyze` to import sources without calling Claude. Research sources are untrusted: verify findings before relying on them.

### Create and share a personality

```bash
node scripts/hive-profile.mjs create profile.example.json my-profile.json
node scripts/hive-profile.mjs inspect my-profile.json
```

The dashboard also has **Share Profile** and **Profile** controls. Exported profiles exclude conversation history, private notes, local file paths and credentials. Inspect the public URLs and preferences before publishing. Imported profiles are saved locally but **do not automatically change Claude Code behavior**.

## What is not yet supported?

- No central Hive Market website or user accounts.
- No automatic learning of model weights; “learning” means extracting and storing knowledge for retrieval.
- No automatic viewing/listening to YouTube videos.
- No proven adaptive agent scheduler that optimizes live task costs.
- No guaranteed end-to-end token reductions or quality improvements.
- No automatic synchronization or merging across cloud providers.

## Open source and contributions

Issues, bug reports, reproducible benchmarks, documentation improvements, and pull requests are welcome. Please avoid committing API keys, private conversations, or proprietary datasets. Test locally before submitting changes.

**License:** See the repository's `LICENSE` file for applicable terms. Verify third-party notices and dependencies before redistribution.

---

## Detailed documentation

## 1. Publish it as a website (no install for visitors)

1. Create a new **public repository** named `claude-hive` on GitHub. Upload all files from this project **including** `.github` and `.claude-plugin` (keep the directory layout).
2. In repository **Settings → Pages**, set **Source: GitHub Actions**.
3. Push to the `main` branch if not already there. The included `.github/workflows/deploy.yml` deploys the static site.
4. Visit `https://YOUR_USERNAME.github.io/claude-hive/` after deployment. Share this URL with testers.

The site itself needs no backend, accounts or build step. For local development, serve it over HTTP (`python -m http.server 8080` in the repository) instead of opening it with `file://`, because JavaScript modules require a proper origin. For cloud OAuth, use HTTPS, except for approved localhost testing.

## 2. Import Claude history

Use Claude's **Export data** option in its account settings (the UI may change), extract `conversations.json` from the downloaded archive, then click **Import Claude data** in Hive. Or import `.md`/`.txt` notes. The example visualization is labeled `SAMPLE` and is replaced on your first actual import or new note.

Supported Claude conversation format resembles:

```json
[
  {
    "name": "An idea discussion",
    "chat_messages": [
      { "sender": "human", "text": "Could we build a hive?" },
      { "sender": "assistant", "text": "Yes, with connected ideas." }
    ]
  }
]
```

This project does **not** import `.zip` directly; unzip the Claude export first. Export contents may contain sensitive information. Review before uploading any backup to a cloud account. Markdown is treated as plain text, not executed HTML.

## 3. Connect cloud storage

Open **Cloud storage** inside the app. Each provider has `Connect`, `Load`, `Save`, `Disconnect`. No cloud upload occurs merely by connecting. Save/load operations ask before overwriting existing data, and backup files are *plaintext JSON*: providers protect them at rest according to their normal policies; this project does not add end-to-end encryption. Your OAuth **client ID** (public, not secret) can be saved in the browser; access tokens and MEGA passwords are **not persisted** by this project. Tokens expire or are lost on refresh and you may need to reconnect.

### Google Drive

1. Open [Google Cloud Console](https://console.cloud.google.com/), create or select a project, and enable the **Google Drive API**.
2. Configure an OAuth consent screen and create an OAuth **Web application** client ID.
3. Add your site's exact origin to **Authorized JavaScript origins**, e.g. `https://YOUR_USERNAME.github.io`. (Google uses a popup through its official Google Identity Services library, not a redirect URI for this flow.)
4. Paste the public client ID into Hive and click **Connect**. The app asks for the `drive.appdata` scope, which accesses only its private app-data folder; that folder is not visible in the normal Drive UI.
5. Click **Save** or **Load** when you want. You may need to add your own Google account as a test user until the consent screen is published or verified.

### OneDrive Personal

1. Open [Microsoft Entra admin center](https://entra.microsoft.com/) → **App registrations** → new app. Register support for **personal Microsoft accounts**.
2. Add a **Single-page application (SPA)** redirect URL matching the *exact site address including path and slash*, e.g. `https://YOUR_USERNAME.github.io/claude-hive/`.
3. Configure delegated Microsoft Graph `Files.ReadWrite.AppFolder` permission. This sample uses the `consumers` endpoint, **personal OneDrive only** (not OneDrive for Business).
4. Paste the **Application (client) ID** into Hive, click **Connect**; it uses OAuth authorization code + **PKCE**. Files live under the app's special `approot` folder.

### Dropbox

1. Use [Dropbox App Console](https://www.dropbox.com/developers/apps) to create a **Scoped access / App folder** app.
2. Enable `files.content.read` and `files.content.write` scopes.
3. Add the exact site URL as an OAuth 2 redirect URI, e.g. `https://YOUR_USERNAME.github.io/claude-hive/`.
4. Paste the app **key** (not app secret) into Hive and click **Connect**. It uses OAuth **PKCE** and saves `/claude-hive.json` inside its app folder.

### MEGA

1. Click **Connect** and enter your MEGA email/password and optional 2FA code **only on your own trusted Hive website**.
2. The pinned, **unofficial** [MEGAJS SDK](https://mega.js.org/) connects to MEGA directly from your browser; no Hive backend receives your password. The app clears password/2FA form fields after the attempt and does not save the credentials or session to local storage.
3. Your backup is in `Cloud Drive / Claude Hive / claude-hive.json`. MEGAJS handles MEGA's client-side file encryption. This connection's browser compatibility and account auth are subject to the third-party SDK and MEGA's API.

**Security:** This is a transparent starter project, not security-audited software. Review dependencies and host it yourself before using sensitive data. It loads the official Google Identity Services SDK, and dynamically loads pinned MEGAJS from `unpkg.com` on request; public Google Fonts are also used. There is no analytics script. Never place OAuth **client secrets**, MEGA passwords, refresh tokens or user data in your GitHub repository. OAuth client IDs are public identifiers, not secrets. On a shared computer, local browser storage can be accessed by other users of that browser profile.

## 4. Claude Code plugin (optional)

The `skills/hive/SKILL.md` and `.claude-plugin/plugin.json` files are recognized by Claude Code's plugin structure. Use this *optional* component when working in Claude Code to organize project ideas; its instructions generate an importable JSON knowledge bundle. It does **not** install itself into Claude.ai or provide direct cloud account access within Claude Code.

For transcripts you explicitly choose to export, you can optionally run:

```sh
node scripts/claude-to-hive.mjs /path/to/your/claude-code-jsonl-folder ./my-hive.json
```

Import the resulting JSON file into the web app. The converter never runs automatically.

## 5. Testing and contributing

Requires Node.js 20+ only to run the tests / optional converter (the hosted site requires no Node installation).

```sh
npm test
```

Good next contributions: direct ZIP import, encrypted portable backups, user-controlled cloud conflict merging, OneDrive for Business support, OAuth integration tests with test accounts, a more advanced similarity model, graph performance optimization, and accessibility testing.

## Eligibility for Claude for Open Source

The separate [Claude for Open Source program](https://claude.com/contact-sales/claude-for-oss) provides six months of Claude Max 20x to eligible **established** open-source contributors. Creating this repository does **not** automatically qualify you. The program lists criteria such as 100 merged PRs into others' repositories in the previous year, 20 unique external contributors to your repository, or significant package reach. You can apply explaining genuine contributions and impact; do not misrepresent project adoption.

## License

MIT. Community contributions welcome.

## Claude Code: connected knowledge sources (v0.3)

This repository is a **Claude Code plugin** with `hive` and `hive-sources` skills. Source ingestion is opt-in and produces an importable `hive.json` file. It is retrieval/context, **not fine-tuning or model training**.

```bash
node scripts/knowledge-sources.mjs add wiki https://en.wikipedia.org/wiki/Knowledge_graph hive.json
node scripts/knowledge-sources.mjs add web https://www.britannica.com/science/encyclopedia hive.json
node scripts/knowledge-sources.mjs add text ./notes.md hive.json
node scripts/knowledge-sources.mjs add image ./diagram.png hive.json
npm test
```

Import `hive.json` in the browser UI. Remote sources are restricted to public encyclopedia hosts, with HTTPS-only requests, no redirects, response limits and timeouts. Image files currently contribute metadata only; **no image understanding or OCR is implemented**. HTML extraction is basic and may include navigation text. Future milestones: user-approved site connectors, chunk-level retrieval with citations, image captioning with consent, deduplication, and cloud synchronization.

## 6. Claude Hive ECO: token-efficient Claude Code context (v0.4)

**Goal:** keep more of Claude's context available for reasoning and relevant evidence, rather than spending it on repetitive successful test/build output or whole libraries of unrelated documents. This version adds a real Claude Code **PostToolUse hook** and a local, budgeted knowledge retriever. The optimized output is *lossy*; turn ECO off when full evidence matters.

**No performance promises:** ECO cannot change model weights, output-token pricing or Claude's internal reasoning. It measures only characters/4 as a **rough token proxy**, not actual Claude usage or subscription savings. The hook shortens eligible *successful Bash result bodies*; overall savings depend on your workflow. It has not been compared against Ponytail or Caveman with controlled Claude sessions.

### Enable in a Claude Code working project

The plugin automatically registers `hooks/hooks.json` when Claude Code loads it. Requires local Node.js 20+. Hooks are **inactive by default**, and do not write files or alter Claude's result unless you opt in in that project:

```bash
# Install or try the plugin from this directory with Claude Code:
claude --plugin-dir /absolute/path/to/claude-hive

# In a separate shell, from the WORKING PROJECT (not the plugin directory):
node /absolute/path/to/claude-hive/scripts/hive-eco.mjs on balanced
# or: on max (smaller summaries, higher chance of hiding details)
node /absolute/path/to/claude-hive/scripts/hive-eco.mjs status
node /absolute/path/to/claude-hive/scripts/hive-eco.mjs stats
node /absolute/path/to/claude-hive/scripts/hive-eco.mjs off
```

**How it works:** the hook only attempts to shorten large, successful `Bash` outputs from allowlisted test/build/lint commands (such as `npm test`, `pytest`, `go test`, `cargo test`). It **does not** rewrite commands or grant permissions. It skips output containing errors, exceptions, failure markers, stderr diagnostics, interrupted/image results, short outputs, unrelated tools and shell pipelines. A condensed output says explicitly that information was omitted. Re-run with ECO off if a full listing is needed. Claude Code's `PostToolUse` only fires for successful tool executions; failed calls remain untouched.

The included `.gitignore` ignores local `.claude/hive-eco.json` config and `.claude/hive-eco-usage.jsonl` **numeric-only** usage events *when the working project is this repository*. If enabling ECO in another project, add those two paths to that project’s `.gitignore`; no command or tool content is collected in these usage events. The hook itself receives Claude Code's tool result over stdin and modifies what Claude sees, but does not send data to a service. See [Claude Code hook docs](https://code.claude.com/docs/en/hooks) for the `updatedToolOutput` mechanism.

### Retrieve focused passages with provenance

After explicitly importing selected websites or text files with `knowledge-sources.mjs`, request only the relevant parts. No embeddings, external model call or dependency installation is needed:

```bash
node scripts/hive-eco.mjs pack \
  --input ./hive.json \
  --query "knowledge graph citations and relationships" \
  --budget 1100 \
  --out ./focused-context.md
```

This uses locally ranked, de-duplicated excerpts and includes the source URL (when known) and approximate line ranges. The budget is **approximate** at four characters per token, not a model-specific tokenizer. Sources are untrusted data; review before relying on them. For full context or precise quotes, open the underlying source; retrieval is not a substitute for verification.

### Validate and benchmark

```bash
npm test
npm run bench:eco
```

The deterministic benchmark illustrates **reduction in example log bodies and knowledge packs only**. It does not measure billed Claude tokens or answer quality. To compare fairly against other extensions, run identical tasks with each enabled/disabled, use Claude's actual usage records and evaluate correctness, retries and total cost per successful task.

## v0.5 — Adaptive Context Intelligence + YouTube encyclopedia sources

Hive can now index **user-supplied YouTube transcripts** as timestamped reference notes and retrieve task-relevant excerpts from the same Hive file as Wikipedia, encyclopedias and documents. No model weights are changed; this is retrieval, not training.

```bash
# Supply a transcript you own or are permitted to use (.txt or .vtt):
node scripts/hive-intelligence.mjs youtube 'https://www.youtube.com/watch?v=VIDEO_ID' captions.vtt hive.json 'Video title'

# Retrieve relevant encyclopedia + video passages within a 3,000-character budget:
node scripts/hive-intelligence.mjs context hive.json 'Explain knowledge graphs' --budget 3000
```

**Adaptive selection:** lexical query matching, weighted titles, ranked transcript/document lines, per-source caps, source URLs and a hard character budget. This minimizes irrelevant context but **does not claim** better answers or verified token savings without live end-to-end benchmarks. The token estimate uses a rough 4-characters/token approximation.

**YouTube limitations:** the tool does not fetch video captions automatically or analyze video frames. Export/provide transcript text first. Use only content you have permission to process. Citations are source URLs and optional transcript timestamps, not proof of source accuracy. The tool does not crawl or bypass access controls.

## Forum links (v0.6)

Paste a public HTTPS development forum thread URL directly into the source importer:

```bash
node scripts/knowledge-sources.mjs add forum "https://discuss.example.org/t/my-thread" hive.json
```

Supports public HTML forum threads (including many Discourse, Stack Overflow, GitHub Discussions, Reddit, DEV and community sites) without per-site credentials. It extracts readable page text, retains the original link and timestamp, and makes it searchable alongside encyclopedia and YouTube transcript sources. Some sites block automated requests or require sign-in/JavaScript and cannot be imported through this basic HTML mode. It does **not** bypass logins, paywalls or access controls. No automatic crawling.

Safety: only user-selected HTTPS links; checks DNS for private addresses; blocks redirects; enforces time and content-size limits. This is a starter importer rather than a hardened network sandbox; do not use on hostile URLs without additional network-level egress controls.

## v0.7 Parallel research agents

Create `sources.json` from `sources.example.json`, replacing placeholders with real URLs and local transcript paths. Then run:

```bash
node scripts/hive-agents.mjs sources.json hive.json --agents 3 --analyze
```

The `--analyze` option runs **up to three separate Claude CLI processes concurrently**, each with tools disabled, to summarize and classify independently imported source content. It requires Claude Code CLI installed and authenticated and **can use more tokens/cost** than sequential retrieval. Omit `--analyze` for concurrent ingestion without Claude calls. `--agents` accepts 1–8. Sources are added in manifest order in a single write, even if completion order differs. Errors are reported per source. No background monitoring or automatic link crawling.

YouTube analysis uses **user-provided .txt/.vtt transcripts**, not video frames or audio. Image ingestion remains metadata-only. These agents don't fine-tune Claude; they build searchable knowledge nodes. Forum pages may block scraping, and sources must be public or explicitly provided. Imported text is untrusted; analysis runs with CLI tools disabled. Never include private credentials in source files.

## Community personality profiles (v0.8)

Hive users can export a **portable, public personality profile** using the **Share Profile** button in the web dashboard, or the CLI:

```bash
node scripts/hive-profile.mjs create profile.example.json my-profile.json
node scripts/hive-profile.mjs inspect my-profile.json
node scripts/hive-profile.mjs instructions my-profile.json
```

Share the generated JSON via GitHub repositories, releases, or other public hosting. Other users can import the file with **Profile** in the dashboard, review its name, tone, rules and source count, then choose whether to store it locally.

**Privacy:** Profile export uses an allowlist; it excludes conversations, private Hive nodes, local file paths, OAuth tokens, passwords and cloud credentials. Public HTTPS source URLs are optional and should be reviewed before sharing. The web importer stores community profiles but does not automatically apply their instructions to Claude Code. CLI `instructions` prints a profile as lower-trust, user-selected guidance; you decide whether to include it in a session. No public account directory, upload server, moderation, ratings, automatic publishing or automatic profile activation is included yet.

**Security:** Community profiles are untrusted data, not system instructions. Do not execute code from downloaded profiles. Profiles may contain misleading or unsafe instructions; review before use. Cloud sync of profiles is not automatic.
