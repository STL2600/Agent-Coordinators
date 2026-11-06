% Vibe-o-Phone
% Rob <robert@rtward.com>
%![](static/qrcode.png)<br/>Talk: [${TALK_URL}](${TALK_URL})<br/>Repo: [${REPO_URL}](${REPO_URL})

# Intro

# First-Party Agent Systems

## The vendors' own cockpits

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

## Scorecard

| | 📱 Phone | 🖥️ Other machines | 🤖 Many agents |
|---|:-:|:-:|:-:|
| Claude Code | ✅ | ✅ | ✅ |
| Codex | ✅ | ✅ | ✅ |
| Cursor 3 | ✅ | ✅ | ✅ |
| GitHub Agent HQ | ✅ | ◐ | ✅ |
| Antigravity | ❌ | ❓ | ✅ |

::: notes
- ✅ yes · ◐ partial · ❌ no · ❓ undocumented
- The big story: in under a year the three main agent vendors all shipped phone control, remote execution and parallel agents. Claude Code Remote Control (Feb), the Codex app (Feb) then its merge into ChatGPT (Jul), and Cursor 3 (Apr) plus its iOS app (Jun).
- Two different meanings of "remote" hide in this table:
  - **Your machine, steered remotely** (Claude Remote Control, Codex host pairing, SSH sessions). You keep your local tools, MCP servers and credentials, but the machine has to stay awake.
  - **Vendor cloud** (Claude cloud sessions, Codex cloud tasks, Cursor cloud agents, GitHub Agent HQ). It survives a closed laptop, but your local environment isn't there.
- What first-party tools *don't* give you:
  - Mixing vendors. Each one drives only its own agent (GitHub is the exception).
  - Open source or self-hosting (except enterprise tiers).
  - One view across Claude + Codex + Cursor at once.
- That gap is exactly what the third-party coordinators fill, which leads into the next section.
:::

# The End

---

Rob <robert@rtward.com>

![](static/qrcode.png)

Talk: [${TALK_URL}](${TALK_URL})

Repo: [${REPO_URL}](${REPO_URL})
