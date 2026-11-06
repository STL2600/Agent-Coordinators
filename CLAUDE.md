# Vibe-o-Phone: talk on coding agent coordinators

Context handoff from a previous session (Nov 6, 2026). Read this before editing `talk.md`.

## The talk

Rob is giving a talk comparing **coding agent coordinators**, the tools that sit above agents like Claude Code and Codex. Every tool is judged on three features:

1. **Phone**: can you steer agents from a phone?
2. **Other machines**: can you drive agents running on another box (SSH, a daemon on a server, desktop-to-desktop, or vendor cloud)?
3. **Many agents**: can several agents run in parallel with isolation (worktrees, containers, VMs)?

Ratings used everywhere: **Yes / Partial / No / Unclear** (on slides: ✅ ◐ ❌ ❓).

## Repo and build

- `talk.md` is the whole deck. GitHub Actions (`.github/workflows/publish.yml`) runs pandoc 3.9 to build reveal.js 5.2.1 (`--slide-level 2`) and publishes to GitHub Pages on every push.
- Theme: the custom Blade Runner-inspired "Neon Noir" stylesheet `static/theme/blade-runner.css`, layered on reveal's `black` theme with `--css`. Section (`#`) slides get the neon-sign treatment; slides whose id starts with `robs-review` get the iris graphic. Screenshot slides assume `![](…){height=560px}` on its own line.
- `# Heading` makes a section title slide; `## Heading` makes a content slide.
- Speaker notes go in a `::: notes` … `:::` fenced div under each slide.
- `${TALK_URL}` and `${REPO_URL}` are filled in by `envsubst` in CI. Leave them alone.
- `static/qrcode.png` is generated in CI.
- Local check: `pandoc --slide-level 2 --standalone -t revealjs -V theme=black -V transition=fade --css static/theme/blade-runner.css -s talk.md -o preview.html`. Write the output to the repo root so `static/` resolves. Local pandoc is 3.13, whose template expects reveal 6 file paths, so don't pass `revealjs-url` pointing at 5.2.1 locally or the plugins 404 and the page renders blank.

### Style Rob asked for

On-screen content stays **high level**: about three short bullets per slide, using the 📱 / 🖥️ / 🤖 icons for the three features. **Speaker notes should be detailed**: dates, how setup works, limits, pricing, and the downsides.

## Status

- `# Intro`: empty, not written yet.
- `# First-Party Agent Systems`: **done**. Slides:
  - Framing slide
  - Claude Code
  - OpenAI Codex (inside the ChatGPT desktop app)
  - Cursor 3
  - GitHub Agent HQ
  - Google Antigravity 2.0
  - Scorecard table
- `# Third-Party Coordinators`: **drafted** (Nov 6). An intro slide, then Conductor, Emdash, Xum, T3 Code, HAPI and Omnigent, each followed by a notes-only "Rob's Review" slide with `_TODO_` for Rob to fill in. Rob's own review notes on the first-party slides are his. Don't rewrite them.
- `# The End`: template closing slide.

Possible additions:

- An "honorable mentions" slide for other vendor platforms (Amp, Warp + Oz, Factory Droid, Devin, Augment Intent/Cosmos).
- A section on mobile-only clients.
- A "your machine vs vendor cloud" framing slide.

## Research data

- `research/coding_agent_coordinators.xlsx` is the main spreadsheet. It has three tabs: Comparison (38 tools), Summary (COUNTIF formulas), and Legend & Sources.
- `research/coding_agent_coordinators.csv` is the Comparison tab as CSV, for easy reading.
- `research/coding_agent_coordinators_active.xlsx` (+ `.csv`) holds only the third-party tools with development activity Oct 6 – Nov 6, 2026 (27 active, 5 excluded). Tabs: Active (the original columns plus last activity date and evidence), Excluded, Summary, Method. Conductor is still shipping but winding down: it announced it's joining Meta on Nov 4, Cloud shuts down Nov 20, and local gets fixes until early 2027. Mux is now **Xum** (coder/xum).
- Each row has: category, license, agents supported, the three ratings with detail text, whether work continues while the laptop sleeps, platforms, pricing, status notes, and source URLs.
- Edit the xlsx directly if you need to. It was generated with openpyxl, and the Summary tab's formulas reference `Comparison!E5:I42`, so extend those ranges if you add rows.

### Tools with Yes on all three (as of Nov 6, 2026)

- Claude Code
- OpenAI Codex
- Cursor 3
- amux
- T3 Code
- Orca
- Omnigent (alpha)
- Vicoa

### Groupings for the third-party section

- **Desktop/worktree coordinators:**
  - Conductor (macOS, closed; cloud and mobile app need the $50/mo Pro plan)
  - Nimbalyst (formerly Crystal; native iOS app)
  - Vibe Kanban (community-maintained since the Bloop shutdown on Apr 10, 2026)
  - Claude Squad (TUI)
  - Emdash (SSH)
  - Paradev (**formerly Superset**; Elastic License 2.0; remote access is a paid beta)
  - Mux (Coder)
  - Sculptor (containers)
  - Agent Orchestrator
  - Jean
  - Synara (fork of T3 Code)
  - Agentastic (beta)
  - Verdent (early access)
- **Full stack with phone, remote and multi-agent:**
  - Orca (MIT, about 66k stars, SSH worktrees, iOS plus Android APK)
  - T3 Code (MIT, Theo/Ping Labs; T3 Connect relay, LAN/Tailscale pairing, desktop-managed SSH, load-balances across machines)
  - Paseo (Apache-2.0, daemon-first)
  - amux (self-host or amux Cloud from $39/mo)
  - Omnigent
  - Vicoa
- **Mobile clients:**
  - Happy (MIT)
  - HAPI (also works over Telegram)
  - Tactic Remote
  - Ciao (P2P, launched Sep 30, 2026)
  - Grass (cloud VM)
  - swe-mux (PWA)
  - PhoneCMD (Android only)
  - Omnara (pivoted to a voice-first platform)
  - AgentWire (voice over tmux)
- **DIY baseline:** SSH + Tailscale + tmux.

### Key fact: two meanings of "remote"

1. **Your machine, steered remotely.** Your local tools, MCP servers and credentials are available, but the host must stay awake.
2. **Vendor cloud.** Work survives a closed laptop, but your local environment isn't there.

### Caveats

- Many details come from **vendor-written roundups** (amux, Nimbalyst, Augment, Grass, Verdent) and may be biased against competitors.
- Unverified claims that are flagged in the notes:
  - Grass reports a bug with approving MCP tool calls in Claude Code Remote Control from mobile (GitHub #69371).
  - A third party says Cursor's iOS app gained control of local agents on Oct 6, 2026.
  - Antigravity's support for other machines is undocumented.
- This space changes monthly. Re-check anything time-sensitive before the talk.

### Most useful sources

- https://openalternative.co/blog/best-ai-coding-agent-orchestrators (updated Nov 4, 2026; best single overview)
- https://codeongrass.com/blog/best-app-to-control-coding-agents-from-mobile/ (mobile clients, Oct 2026; written by a vendor)
- https://amux.io/guides/best-phone-apps-for-coding-agents-2026/ (written by a vendor)
- https://code.claude.com/docs/en/remote-control and https://code.claude.com/docs/en/desktop
- https://learn.chatgpt.com/docs/remote-connections
- https://github.com/pingdotgg/t3code/blob/main/docs/user/remote-access.md
- https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/
- https://cursor.com/blog/ios-mobile-app
