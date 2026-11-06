% Vibe-o-Phone
% Rob <robert@rtward.com>
%![](static/qrcode.png)<br/>Talk: [${TALK_URL}](${TALK_URL})<br/>Repo: [${REPO_URL}](${REPO_URL})

# Intro

# First-Party Agent Systems

- Built by the company that makes the agent
- Usually only drive **their** agent
- The baseline every third-party tool is measured against

::: notes
- "First-party" here means the coordinator ships from the same company that makes the model / agent: Anthropic, OpenAI, Cursor (Anysphere), GitHub (Microsoft), Google.
- Five to cover: Claude Code, OpenAI Codex, Cursor 3, GitHub Agent HQ, Google Antigravity.
- The three questions I'm asking of every system in this talk:
  1. Can I drive it from my phone?
  2. Can I drive agents running on *another* machine (a desktop at home, a dev box, a VPS, or the vendor's cloud)?
  3. Can I run several agents at once without them stepping on each other?
- Spoiler: the big three (Claude Code, Codex, Cursor) now say "yes" to all three. That happened fast. In early 2026 almost none of them did, which is why the whole third-party ecosystem (Happy, Conductor, Vibe Kanban, etc.) sprang up.
- The common trade-off: first-party tools are polished and well integrated, but they lock you into one vendor's agent and models. GitHub is the interesting exception: it's a first-party *platform* that hosts other vendors' agents.
- Everything in this section was checked against vendor docs or launch coverage in early November 2026. This space changes monthly; double-check before quoting version numbers.
:::

## Claude Code (Anthropic)

- 📱 **Phone:** Remote Control via the Claude app
- 🖥️ **Other machines:** SSH, cloud, and self-hosted sessions
- 🤖 **Many agents:** parallel sessions, subagents, Agent Teams

::: notes
**Surfaces:** terminal CLI, IDE extensions, the Code tab in Claude Desktop (macOS, Windows, Linux beta), claude.ai/code on the web, and the Claude iOS/Android app.

**Phone: Remote Control**

- Launched Feb 24–25, 2026 as a research preview (Max first), now on Pro, Max, Team and Enterprise. Not available with API keys; you must be logged in with a claude.ai subscription. Team/Enterprise owners have to turn it on in admin settings.
- How it works: run `claude remote-control` (server mode) or `claude --remote-control` / `--rc`, or type `/remote-control` (`/rc`) in a running session. It prints a URL and, if you press space, a QR code. Scan it and the same session shows up in the Claude mobile app or at claude.ai/code.
- The session keeps running **on your machine**: your filesystem, MCP servers, tools and config are all still there. Conversation, subagent progress and workflows stay in sync across terminal, browser and phone; you can type from any of them.
- You can attach photos and files from your phone, and pick the model from the device.
- Server mode can host many sessions at once: default capacity is 32 (`--capacity`). `--spawn worktree` gives each new session its own git worktree; `same-dir` shares the folder; `session` is single-session mode. Press `w` to toggle.
- Desktop app has a setting to connect every new session to Remote Control automatically.
- Optional "Trusted Devices" requires each device to be verified before it can view or steer sessions.
- Limits: the local `claude` process must keep running. If the laptop sleeps it reconnects when it wakes. In server mode, a network outage over ~10 minutes ends the process. Stopped sessions can be resumed for about 4 hours with `--continue` / `--session-id`. Some commands (`/plugin`, `/resume`) are terminal-only.
- Caveat: a competitor (Grass) reports an open bug with approving MCP tool calls from mobile and points out the relay is TLS, not end-to-end encrypted. I haven't verified the bug, so treat it as "check before you rely on it."
- Separately, **Dispatch** lets you send a task from your phone that the Desktop app picks up and runs.

**Other machines**

- The Desktop app can run a session locally, **over SSH** on a remote box, in WSL, or in Anthropic's cloud. SSH sessions get the terminal, file viewer and code review; the file pane is view-only.
- **Cloud sessions** (Claude Code on the web) run on Anthropic infrastructure and keep going with your laptop closed. Team/Enterprise can deploy **self-hosted environments**, cloud sessions that run inside your own network.
- You can also start sessions from Slack by mentioning @Claude.
- Low-tech option the docs recommend: run `claude` in tmux on a remote box and turn on Remote Control.

**Many agents**

- Desktop: run several sessions in parallel, split view or separate windows, each optionally in its own git worktree (stored under `.claude/worktrees/`). Sessions can check on and message each other.
- **Subagents** work inside one session and report back to the parent.
- **Agent Teams** (experimental since Feb 2026, needs v2.1.32+ and a flag): a lead session spawns teammate sessions in separate worktrees. Unlike subagents, teammates can message each other directly and you can talk to any one of them.
- Scheduled tasks run recurring work, and cloud scheduled tasks need none of your machines online.

**Cons:** Claude models only, and closed source. That's the main reason third-party coordinators exist on top of it.
:::

## Rob's Review

::: notes

 - Works well for claude obviously
 - Great for reviewing plans
 - Notifications and multi-session support work well
 - Annoying to have to run remote control on each session

:::

## OpenAI Codex (ChatGPT desktop app)

- 📱 **Phone:** ChatGPT app pairs with your computer
- 🖥️ **Other machines:** SSH hosts, other desktops, cloud tasks
- 🤖 **Many agents:** parallel threads, each in its own worktree

::: notes
**History:** the Codex desktop app launched for macOS on Feb 2, 2026, billed as a "command center" for running several agents in parallel. On July 9, 2026 OpenAI folded it into the ChatGPT desktop app, so "the Codex app" now means the Codex area inside ChatGPT desktop. The Codex **CLI** is separate and open source (Apache-2.0, over 120k GitHub stars), which is why so many third-party tools wrap it.

**Phone: remote connections**

- The host is the ChatGPT desktop app on **macOS or Windows**. The phone is the ChatGPT app on iOS or Android, where it appears under "Codex" (older builds call it "Remote").
- Setup must start on the desktop (Settings → Connections → Control this Mac/PC), which shows a QR code you scan. You can't set it up from the Codex CLI or IDE extension. Workspace admins may need to enable it.
- From the phone you can start new chats in the host's projects, continue existing ones, answer questions, steer, approve commands, review diffs, test results, terminal output and screenshots, and get notified when a task finishes or needs you. You can switch between several paired hosts.
- Everything runs on the host: repo files, shell, MCP servers, skills, credentials, Computer Use, and browser setup. Traffic goes through a secure relay, so the host isn't exposed to the internet.
- The host must be awake, online and signed in. There's a "Keep this Mac awake" setting while plugged in. A Mac laptop stays reachable with the lid closed only if it's on power with an external display.

**Other machines**

- **SSH hosts:** the desktop app reads `~/.ssh/config`, starts a remote Codex app server over SSH, and runs chats against the remote filesystem and shell. Codex must be installed and logged in on the remote box. Your phone still talks to the desktop host, which talks to the SSH box, so it's two hops.
- **Control other devices:** one desktop can drive another desktop, which is useful for an always-on Mac mini.
- **Cloud tasks:** a task can run in OpenAI's cloud and keep going with the laptop closed. Reviewers note the local↔cloud switch is manual.
- Newer "Local computer access with Work Cloud": OpenAI's cloud coordinates the task and uses your computer for steps that need local files. If the computer is offline, the task continues in a cloud container without local access.

**Many agents**

- Projects contain threads, and each thread can get its own git worktree automatically. Run as many in parallel as your plan allows.
- Merging worktrees back is manual, and there's no built-in editor, so small fixes mean jumping to your IDE.

**Cons:** OpenAI models only. Windows support for the app came later than Mac.
:::

## Rob's Review

::: notes

 - I was not able to maintain remote connections at all either on the phone or another computer

:::

## GitHub Agent HQ

- 📱 **Phone:** Mission Control in GitHub Mobile
- 🖥️ **Other machines:** GitHub's cloud, not your boxes
- 🤖 **Many agents:** Copilot, Claude *and* Codex on one issue

::: notes
**What it is:** GitHub's control plane for coding agents. Announced at GitHub Universe (Oct 2025); Claude and OpenAI Codex arrived as public-preview agents on **Feb 4, 2026** for Copilot **Pro+ and Enterprise**. GitHub said other tiers would follow, so check current availability.

**Why it's interesting for this talk:** it's the only first-party option that is **multi-vendor**. GitHub isn't selling a model, it's selling the place where agents do work.

**Phone**

- **Mission Control** is the dashboard for starting, watching and reviewing agent sessions. It's on github.com, in VS Code, in the Copilot CLI, and in **GitHub Mobile**.
- From the phone you can assign an issue to an agent, watch progress, and review the draft PR it opens. It's the most "manager-like" phone experience of the five: you're delegating and reviewing, not babysitting a terminal.

**Other machines**

- Agents run in GitHub-hosted, Actions-powered environments, not on your laptop or dev box. Great for "keep working while my laptop is closed," but you can't point it at your own machine.
- VS Code (Feb 2026 multi-agent update) can run Claude and Codex locally *or* as cloud agents, and the Claude integration uses Anthropic's official agent harness.
- I rated this "Partial" in the spreadsheet: remote, but only GitHub's remote.

**Many agents**

- Assign one issue to Copilot, Claude, Codex, or all three, and compare the PRs side by side. That's a neat demo idea.
- Mention `@Claude` or `@Codex` in PR review comments to send them back for changes.
- Repo admins have to enable Claude and Codex per repository.
- Enterprise governance: audit logs, policies on which agents and models are allowed, agents restricted to designated branches, and a Copilot metrics dashboard for agent activity.

**Cons:** async, PR-shaped workflow only. It's not interactive pair-programming, and it needs a paid Copilot tier.
:::

## Rob's Review

::: notes

Actually works pretty well if you're living in GitHub already
Cloud workspaces are the best implementation from any major provider
Works with non-copilot agents _kinda_

:::

## Google Antigravity 2.0

- 📱 **Phone:** no official app
- 🖥️ **Other machines:** not documented
- 🤖 **Many agents:** Manager Surface for parallel agents

::: notes
**History:** Antigravity launched in late 2025 as Google's agent-first IDE. **Antigravity 2.0** was announced at Google I/O in May 2026 as a standalone desktop app for orchestrating several agents in parallel, with scheduled background tasks and subagent workflows.

**Many agents (its strength)**

- The **Manager Surface** lets you spawn, orchestrate and monitor several agents running in parallel and asynchronously. Google was arguably first to make "agent manager" the main view rather than the editor.
- Also new in 2.0: a Go-based Antigravity CLI, a public SDK, Managed Agents in the Gemini API, built-in integrations with AI Studio, Firebase and Android, and voice control with a wake word ("Hey Antigravity").

**Phone**

- No official mobile app for Antigravity itself. The nearest Google option is the **AI Studio** mobile app, which is for prototyping, not steering agents. A third-party "Mobile IDE for Antigravity" exists but isn't from Google.
- Workaround: several third-party coordinators now support Antigravity as a provider (T3 Code, Emdash, PhoneCMD, Synara), so you can drive it from a phone through them.

**Other machines**

- I found nothing on SSH or remote-machine support in what I reviewed. Managed Agents in the Gemini API point toward cloud execution, but that's an API, not a coordinator feature.
- **Jules**, Google's earlier async GitHub agent (cloud-run, PR-based), still exists, but its changelog reportedly hasn't moved since March 2026. Mention it only in passing.

**Cons:** the weakest of the five on my three criteria. Strong on multi-agent, weak on mobile and remote.
:::

## Rob's Review

::: notes

Doesn't really have remote options like the others aside from SSH

:::

## Cursor 3 (Agents Window)

- � **Phone:** native iOS app (beta), web on Android
- �️ **Other machines:** local, SSH, or cloud VMs
- 🤖 **Many agents:** parallel agents, each in a worktree or VM

::: notes
**History:** Cursor 3 shipped in April 2026 and reorganized the product around an **Agents Window**, one place for every agent you have running, whether it was started from desktop, web, phone, Slack, GitHub or Linear.

**Phone**

- Native **iOS / iPadOS app**, public beta since June 29, 2026, on all paid plans. No Android app announced; Android users install the web app as a PWA.
- From the phone you can launch cloud agents, watch them run, review diffs plus the screenshots and videos agents record of their work, merge PRs, and talk to agents by voice.
- At launch the iOS app drove **cloud** agents only. A third-party write-up says an Oct 6, 2026 update added steering agents running on your own computer. I couldn't confirm that in Cursor's changelog, so say it's reported, not confirmed.
- Grass's comparison describes Cursor mobile as "directing, not controlling": PR review and steering, with no terminal, file browser, or tool-call approvals.

**Other machines**

- The Agents Window runs agents **locally, over SSH, or on cloud VMs**.
- Cloud agents get an isolated VM. For front-end work they launch headless Chromium to test their own changes and record demos.
- **Self-hosted cloud agents** for Enterprise since March 25, 2026, so the VMs can live in your own infrastructure.

**Many agents**

- Each agent gets its own git worktree (local) or VM (cloud). Sources commonly cite up to 8 parallel agents.
- Design Mode previews UI changes before you accept them.
- Agents can be kicked off from Slack, GitHub issues/PRs, or Linear.

**Pricing:** free Hobby tier caps agent requests. Cloud agents start at Pro ($20/mo), with Pro+ ($60) and Ultra ($200).

**Cons:** closed-source VS Code fork, no self-hosting except Enterprise, and it's Cursor's own agent (though you choose the model). Unlike Claude Code and Codex, it's model-agnostic, which counts in its favor.
:::

## Rob's Review

::: notes

Haven't actually used cursor

:::

# Third-Party Coordinators

- Sit **on top of** the vendors' agents
- Mix Claude, Codex, Cursor and more in one place
- Mostly open source, mostly free

::: notes
- "Third-party" here means the coordinator comes from someone other than the agent vendor. It drives the CLIs you already have installed and logged in (Claude Code, Codex, Cursor CLI, OpenCode…), so it runs on your existing subscriptions.
- Why they exist: until early 2026 none of the vendors offered phone control, remote machines *and* parallel agents. These tools filled that gap. The vendors have mostly caught up, so the pitch now is **vendor-neutral**: one UI across every agent, and often self-hostable.
- How I picked these six: my research sheet has 32 third-party tools. I kept only the ones with real development activity between Oct 6 and Nov 6, 2026 (27 made the cut), then picked six that show different approaches:
  - **Conductor**: polished, closed-source Mac app, and a warning about what happens when a startup gets acquired.
  - **Emdash**: desktop app with the most agents supported, plus SSH.
  - **Xum (Coder)**: its own agent loop, with SSH, Docker and dev container runtimes.
  - **T3 Code**: the full package, with phone, remote machines and parallel agents.
  - **HAPI**: self-hosted phone remote that wraps the official CLIs.
  - **Omnigent**: a "meta-harness" that mixes agents in one session and adds policies.
- Dropped as inactive: Claude Squad (last release Aug 20), Ciao, Grass, PhoneCMD, and Augment Intent/Cosmos.
- Same three questions as before: phone, other machines, many agents.
:::

## Conductor

- ⚠️ **Joining Meta:** winding down as of Nov 4
- 📱 **Phone:** iOS app, remote control of your Mac
- 🤖 **Many agents:** one git worktree per agent

::: notes
**What it is:** a closed-source macOS app from Melty Labs for running Claude Code, Codex, Cursor and OpenCode agents in parallel. It was one of the tools that started the category, and T3 Code's README names it as an inspiration.

**Big news (changelog 0.93.0, Nov 4, 2026): Conductor is joining Meta and the product is being wound down.**

- **Conductor Cloud** goes read-only **Nov 13** and shuts down **Nov 20**. Right-click a cloud workspace and choose "Transfer to local" to keep it.
- The **local app** gets new models and critical bug fixes **until early 2027**, then it's effectively done.
- Export everything from Settings → Account → Export data.
- Use it as the cautionary tale of the talk: closed source plus a startup means your workflow can disappear in two weeks. Compare that with Vibe Kanban, which kept going as a community project after Bloop shut down because it was open source.

**Phone**

- **Conductor for iOS** (0.90.0, Oct 2, 2026): chat with agents and merge PRs from your phone. At launch it was aimed at **cloud** workspaces.
- **Remote control** (0.92.0, Oct 23, 2026, experimental): turn it on in Settings → Experimental, then scan a QR code with the iOS app. From the phone you can browse and create workspaces, read chats, send messages, pick models and stop agents, but only for the Mac's **local** workspaces.
- It stays connected while the window is minimized, and keeps the Mac awake while the phone is connected and the Mac is plugged in.
- No Android app.

**Other machines**

- **Conductor Cloud** (0.78.0, Jul 30, 2026) ran workspaces in hosted sandboxes and was reported to need the $50/mo Pro plan. It's the part being shut down first.
- I found no SSH support for your own machines.

**Many agents**

- Each agent gets its own git worktree. One view shows progress, diffs and PRs, with merge stacks and automerge.

**Cons:** macOS only and closed source, and now it's being sunset. Hard to recommend starting with it today.
:::

## Rob's Review

::: notes

Actually worked pretty well, aside from the fact that Meta bought it to kill it.

:::

## Emdash

- 📱 **Phone:** none
- 🖥️ **Other machines:** SSH/SFTP remote projects
- 🤖 **Many agents:** worktree per task, 30+ agent CLIs

::: notes
**What it is:** an open-source (Apache-2.0) desktop app from General Action (YC W26) for running coding agents in parallel. Electron, so it runs on macOS, Windows and Linux (x64 and ARM64). Started Aug 2025, about 6.2k GitHub stars, v1.2.11 shipped Nov 6, 2026 with 290 commits in the past month.

**Phone**

- No mobile client and no web UI. It's the "desk" option in this lineup.
- Emdash 2.0 is announced as a cloud-first product for teams, but there's only an early-access waitlist and no details.

**Other machines**

- **Remote projects** over SSH/SFTP: connect to your own boxes and run the same parallel workflow on remote codebases.
- Supports SSH agent, key and password auth, including OpenSSH certificates. Credentials go in your OS keychain.
- tmux sessions keep long-running agents alive across reconnects, locally or remotely.

**Many agents**

- Every task gets its own git worktree and branch, and a task can create several worktrees, each with its own agent.
- The broadest agent support I found: the docs list **34 CLI providers**, including Claude Code, Codex, Cursor, OpenCode, Amp, Devin, Qwen Code, Droid and GitHub Copilot. It detects installed CLIs automatically.
- It installs lifecycle hooks into agent configs to track status, notifications and resumable sessions.
- Pulls tickets from Linear, GitHub, Jira, GitLab, Asana, Notion, Monday.com and others straight into an agent. Review diffs, open PRs, check CI and merge from the app.

**Privacy:** local-first. State lives in a local SQLite database and no code or chats go to Emdash servers. Telemetry is optional (`TELEMETRY_ENABLED=false`).

**Pricing:** free.

**Cons:** no phone story at all, so it only answers two of my three questions.
:::

## Rob's Review

::: notes

Works really well for multi-machine setups, though no phone app.

:::

## T3 Code

- 📱 **Phone:** native iOS & Android apps plus web
- 🖥️ **Other machines:** T3 Connect relay, LAN/Tailscale, SSH
- 🤖 **Many agents:** threads across agents *and* machines

::: notes
**What it is:** an open-source (MIT) "agent harness control surface" from Ping Labs (Theo Browne). Repo created Feb 8, 2026 and already about **49k GitHub stars**, the second-biggest tool in this space after Orca. Very active: 4,800+ commits in the past month, nightly builds.

**Their pitch:** "What are you selling me? Nothing." It runs on the subscriptions you already have for Claude Code, Codex, Cursor, Grok Build, OpenCode, Google Antigravity and fx. Install and log in to at least one provider CLI first. It's one of the few ways to drive Antigravity from a phone.

**Setup:** `curl -fsSL https://t3.codes/install.sh | sh`, then `t3`. `t3 service install` keeps it running in the background on macOS and Linux. Desktop builds come through Homebrew, winget, scoop, AUR and Nix.

**Phone**

- Native **iOS and Android apps** plus a hosted web app at app.t3.codes.
- Pair by scanning a QR code or link (`t3 pair`), or sign in to T3 Connect and pick the machine.

**Other machines** (all three routes can be set on one machine; it uses the fastest one that answers)

- **T3 Connect:** a relay, so no port forwarding. `t3 up` on the host. Free accounts can link **3 machines**. A paid T3 Chat plan raises that to 10 (Pro) or 100 (Premier).
- **LAN / Tailscale:** `t3 up --lan` and a one-time pairing code. Once paired through T3 Connect, it learns the LAN and Tailscale addresses on its own and switches to them when you're home.
- **SSH:** the desktop app can install and start a T3 server on a remote Linux or Apple Silicon box.
- **Load balancing:** optionally spreads new threads across several connected machines.

**Many agents**

- Many threads across providers, branches/worktrees and machines in one UI.
- Outside agents (Claude Code, Codex, ChatGPT) can connect to it over MCP.

**Cons:** the maintainers say "very very early, expect bugs", and they're mostly not accepting contributions. The host has to stay on, there's no cloud execution, and the background service doesn't run on Windows.
:::

## Rob's Review

::: notes

 - Probably the most "full featured" system
 - Includes cloud service for connecting

:::

## HAPI

- 📱 **Phone:** web/PWA, native apps, even Telegram
- 🖥️ **Other machines:** self-hosted hub, runners on any box
- 🤖 **Many agents:** many sessions, agents can spawn peers

::: notes
**What it is:** a self-hosted remote control for the **official** agent CLIs: Claude Code, Codex, Cursor Agent, Grok Build, OpenCode, Kimi, Copilot, Antigravity, Pi and DeepSeek. Open source (AGPL-3.0), by tiann. Started Dec 2025, about 5.4k stars, v0.34.3 on Nov 5, 2026.

**The name:** "哈皮" (*hā pí*) is a Chinese transliteration of **Happy**, the best-known mobile client. HAPI is pitched as the local-first alternative:

- Happy runs a central cloud server that stores your end-to-end encrypted data.
- HAPI has each user run **their own hub**, a single binary with SQLite.

**Setup:** `npx @twsxtd/hapi hub --relay` starts the hub with an end-to-end encrypted relay (WireGuard + TLS). Then run `hapi` (or `hapi claude`, `hapi codex`) to start a session. The hub prints a URL and QR codes.

**Phone**

- **Web/PWA (free):** the most complete client. It has a remote terminal, file browser and git diffs, approvals, a voice assistant and push notifications.
- **Native iOS/Android apps (paid):** chat, approvals, new sessions, files, dictation and native push. No terminal or realtime voice yet.
- Also works as a **Telegram Mini App**.
- "Seamless handoff": start in your terminal, take it to your phone, come back, with no session restart. Codex sessions can be used from terminal and phone at the same time (needs Codex 0.154.0+).

**Other machines**

- Architecture: CLI/Runner machines ↔ hub ↔ clients. The hub can run on your desktop or a VPS.
- Run `hapi runner start` on any machine and you can **start new sessions on it from your phone**.
- Reach it through the public relay, Cloudflare Tunnel, Tailscale, or your own HTTPS reverse proxy.

**Many agents**

- Many sessions across machines and agent types in one list.
- Agents get MCP **peer tools** (`list_peers`, `inspect_peer`, `ping_peer`, `spawn_peer`) to find, message and start other sessions, which is a lightweight multi-agent setup.
- There's no worktree management or task board, so I'd call it partial on isolation.

**Cons:** you run and secure the hub yourself, the native apps cost money, and the host machines have to stay on.
:::

## Rob's Review

::: notes

I had trouble keeping this one working.
The connection seemed to drop a lot, but I'm not sure if I was doing something wrong.

:::

## Omnigent

- 📱 **Phone:** mobile web UI plus iOS/Android apps
- 🖥️ **Other machines:** your hosts or cloud sandboxes
- 🤖 **Many agents:** mix vendors in one session, with policies

::: notes
**What it is:** an open-source (Apache-2.0) **"meta-harness"**, an orchestration layer over Claude Code, Codex, Cursor, OpenCode, Goose, Antigravity, Copilot, Devin, Kiro, Qwen Code, Kimi Code, Grok Build, Pi and custom agents defined in YAML. Self-described **alpha**. Started Jun 2026, about 11k stars, v0.21.0 on Nov 5, 2026. Installs from PyPI, and there's a macOS desktop app.

**Credentials:** API keys, Claude Pro/Max or ChatGPT **subscriptions** (through the official CLIs), any OpenAI- or Anthropic-compatible gateway (OpenRouter, LiteLLM, Ollama, vLLM), or Databricks.

**Phone**

- Deploy the server somewhere reachable and the web UI is built for phones: chat, subagents, terminals and files, all synced with your laptop.
- The website also links native iOS and Android apps.
- On your home network you can just open `http://<lan-ip>:6767` on your phone.

**Other machines**

- Server options: `docker compose up` on a VPS or home server, one-click Render or Railway, or Fly.io, Modal, Cloudflare and others. Reach a laptop-hosted server with a Cloudflare quick tunnel or Tailscale.
- `omnigent host https://your-host` registers a machine as a place sessions can run.
- **Cloud sandboxes** per session (Modal, Daytona, E2B, Kubernetes and about ten more), so no laptop has to stay online.

**Many agents (its strength)**

- Mix harnesses **in one session**: have one agent review another's work, or split a task by strengths.
- Example agents ship with it:
  - **Polly** is a tech-lead orchestrator. She hands work to Claude Code, Codex or Pi in parallel worktrees, then sends each diff to a reviewer **from a different vendor** than the one that wrote it.
  - **Debby** asks Claude and GPT the same question and shows the answers side by side, and `/debate` makes them critique each other.
- **Policies:** pause for approval before shell commands, cap tool calls, set hard spend limits. They stack at server, agent and session level.
- **Team features:** multi-user accounts with invite links or OIDC, share a live session, co-drive someone's running session, or fork it.

**Cons:** alpha, so expect churn. More moving parts than the others (a server, hosts, policies). It's most worth it for teams or if you want cross-vendor review.
:::

## Rob's Review

::: notes

I'm actually trying this one out right now, so TBD.

:::

## Orca

- 📱 **Phone:** iOS app plus Android APK (beta)
- 🖥️ **Other machines:** SSH hosts, Orca servers, cloud VMs
- 🤖 **Many agents:** worktree per agent, plus an orchestrator

::: notes
**What it is:** a free, open-source (MIT) desktop app from Stably AI (YC) for running Claude Code, Codex and other coding agents in parallel, each in its own worktree. Runs on macOS, Windows and Linux. Repo created Mar 17, 2026 and now about **117k GitHub stars**, by far the biggest third-party coordinator. It's also the busiest: about 6,800 commits in the past month and releases almost daily (v1.4.251 on Nov 6, 2026).

**Agents:** the site claims 27 supported agents, including Claude Code, Codex, OpenCode, Grok, Gemini, Cursor, Copilot and Amp, plus "any other CLI agent". Run each one as the raw terminal CLI or in Orca's chat UI, and switch between the two without losing context. It uses your own agent subscriptions, and you can switch between several Claude or Codex accounts.

**Phone (Orca Mobile, beta)**

- iOS from the App Store; Android as an **APK download**, not Google Play.
- Pair once: desktop gives a one-time code, enter it on the phone. **Orca Relay** (needs sign-in) is the default route; a LAN address is an optional faster route; Tailscale works too.
- From the phone:
  - Chat with agents, or use the raw terminal with a live keystroke mode.
  - Answer prompts and approve permissions.
  - Stage, commit and read diffs, leave PR comments, resolve merge conflicts.
  - Push notifications when an agent finishes.
  - Create new workspaces (from GitHub, Linear, GitLab or a branch) and choose which host builds them.
- Limits: it's deliberately a remote control, not an editor. With LAN pairing, closing the desktop app drops the connection. The docs don't say whether the relay is end-to-end encrypted.

**Other machines** (four ways to run)

- **Local:** everything on your laptop, which stops when the laptop sleeps.
- **SSH hosts:** add a host in Settings → SSH (it needs git). Orca installs its own server there on first connect. Agents keep running on the host and your laptop reattaches when it reconnects.
- **Remote Orca Server:** pair two machines over Tailscale, or run `orca serve` on headless Linux. The server owns the worktrees, terminals and agents, so work continues while your laptop sleeps and the phone can reconnect.
- **Cloud VM per worktree** (experimental): an `orca.yaml` recipe boots an environment for each worktree on Vercel Sandbox, Fly, Modal, local Docker or an SSH host. You bring and pay for the cloud account; Orca doesn't sell hosting.

**Many agents**

- Parallel worktrees, split panes (agents, terminals, browser and diffs side by side), and GitHub and Linear tasks built in.
- **Orchestration** (experimental): give a big task to one coordinator agent. It splits the work into tasks, starts worker agents (Claude, Codex, Cursor, OpenCode, Antigravity…) in child worktrees, even on other hosts, and collects their results through an inbox.
- The **Orca CLI** lets agents do anything you can: create worktrees, start workers, drive the browser, schedule automations.
- Extras: line-by-line review notes on diffs sent back to the agent, failing CI checks sent back for fixes, Design Mode (click an element in the built-in browser to give its HTML and CSS to the agent), and an iOS/Android emulator the agent can test with.

**Cons:** the UI is dense and there's a lot to learn. The mobile app, cloud VMs and orchestration are all beta or experimental. Shipping this fast means things change week to week.
:::

## Rob's Review

::: notes

This is on the list for me to try next week

:::

# DIY: The Raw Terminal

- No coordinator app, just SSH and tmux
- Works with **any** agent CLI
- The baseline every tool here is competing with

::: notes
- Every tool in this talk is, underneath, a nicer version of this: a long-running process on a machine you control, a way to reach it from anywhere, and a way to keep several agents apart.
- You can build that yourself from four boring, free, battle-tested pieces:
  - **Tailscale** to reach the machine.
  - **SSH or mosh** to connect.
  - **tmux** to keep sessions alive.
  - **git worktrees** to keep agents out of each other's way.
- Why bother when the tools exist?
  - Nothing new to trust: no relay server, no new app with access to your code.
  - Works with every agent on day one, including ones no coordinator supports yet.
  - Nothing to be acquired or shut down. Compare Conductor.
  - It's what the Claude Code docs themselves suggest for remote boxes: run `claude` in tmux, then turn on Remote Control.
- The cost is ergonomics, especially on a phone. That's the trade-off to land in this section.
:::

## SSH + VPN + tmux

- 📱 **Phone:** any SSH app (Blink, Termius, Termux)
- 🖥️ **Other machines:** anything on your tailnet
- 🤖 **Many agents:** a tmux window and a worktree per agent

::: notes
**Setup, start to finish**

1. **Tailscale** on the host and on your phone. It's a WireGuard mesh, so the host gets a stable name (MagicDNS, e.g. `devbox`) and nothing is exposed to the public internet. The personal plan is free.
2. **SSH** into the host.
   - On Linux, `tailscale up --ssh` lets Tailscale handle auth, so you don't manage keys.
   - On a Mac, turn on Remote Login and use normal SSH keys.
3. **tmux** so the agents outlive your connection:

   ```bash
   tmux new -A -s agents     # attach if it exists, create if not
   ```

   Run one agent per tmux window. `Ctrl-b d` detaches and the agent keeps working; `tmux attach -t agents` picks it back up from your laptop or phone.
4. **git worktrees** so parallel agents don't edit the same checkout:

   ```bash
   git worktree add ../myrepo-auth -b feature/auth
   cd ../myrepo-auth && claude
   ```

   Merge the branch when it's done; `git worktree remove` cleans up.

**Phone clients**

- **iOS:** Blink Shell (built-in mosh, good keyboard handling) or Termius.
- **Android:** Termux (full Linux userland, `pkg install mosh openssh`) or Termius.
- Add `set -g mouse on` to `~/.tmux.conf` so you can tap between panes and scroll with your finger.

**Other machines**

- Anything that can run tmux and join your tailnet: a desktop at home, a Mac mini, a VPS, a cloud dev box. Since it's all just SSH, hopping from one box to another is free.

**Many agents**

- As many as you have tmux windows and worktrees for. There's no dashboard: you cycle through windows (`Ctrl-b w` shows a list) to see who's waiting on you.
- If you want a little more structure without leaving the terminal, Claude Squad was the TUI for exactly this (tmux + worktree per agent), but it hasn't had a release since Aug 2026.
:::

## Making It Livable

- 🔔 **Notifications:** agent hooks + ntfy
- 📶 **Flaky networks:** mosh instead of SSH
- ☕ **Stay awake:** caffeinate, or an always-on box

::: notes
The raw setup works, but three things make it miserable on a phone. Each has a small fix.

**1. You don't know when an agent needs you.** Push notifications with **ntfy** (free, open source, iOS/Android apps): subscribe to a topic on your phone, then have the agent `curl` it.

Claude Code, in `~/.claude/settings.json` (the `Notification` hook fires when Claude needs input, `Stop` fires when it finishes):

```json
{
  "hooks": {
    "Notification": [{ "hooks": [{ "type": "command",
      "command": "curl -s -d 'Claude needs you' ntfy.sh/<your-topic>" }] }],
    "Stop": [{ "hooks": [{ "type": "command",
      "command": "curl -s -d 'Claude finished' ntfy.sh/<your-topic>" }] }]
  }
}
```

Codex, in `~/.codex/config.toml` (runs when a turn completes):

```toml
notify = ["bash", "-lc", "curl -s -d 'Codex finished' ntfy.sh/<your-topic>"]
```

Public ntfy.sh topics are readable by anyone who guesses the name, so use a long random topic or self-host ntfy.

**2. Phone connections drop constantly.** Switching Wi-Fi to cellular, or locking the phone, kills an SSH session. **mosh** runs over UDP and survives roaming and sleep, and it echoes your typing locally so it feels fast on bad connections.

- Install `mosh` on the host. Over Tailscale the UDP ports (60000–61000) just work.
- mosh has no scrollback of its own, but tmux handles that.
- Even without mosh, tmux means a dropped connection costs you nothing: just reattach.

**3. The host goes to sleep.**

- macOS: `caffeinate -is` stops idle sleep while plugged in. A laptop with the lid closed still sleeps unless it's on power with an external display.
- The real fix is an always-on box (Mac mini, home server, or a small VPS), so your laptop is just another client.

**Cons of the whole DIY approach**

- Typing and reading on a phone terminal is hard: no diff viewer, no tap-to-approve, and long agent output is painful to scroll.
- You build and maintain it yourself: tmux config, hooks, worktree cleanup, security updates.
- No overview: nothing tells you at a glance which of six agents is stuck.
- That's exactly the gap the tools in this talk sell into. DIY is the floor: if a coordinator isn't clearly better than this for how you work, you don't need it.
:::

## Rob's Review

::: notes

I mean, it works...

:::

# The End

---

Rob <robert@rtward.com>

![](static/qrcode.png)

Talk: [${TALK_URL}](${TALK_URL})

Repo: [${REPO_URL}](${REPO_URL})
