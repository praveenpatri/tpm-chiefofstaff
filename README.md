# TPM-CoS: Chief of Staff for TPMs

One persistent proxy agent per program. A portfolio view across all of them. Slack access for your stakeholders.

You are the manager. Each program gets a TPM agent whose identity, knowledge, decisions, risks, reporting templates, and schedule live in plain markdown and YAML files on a local drive or a cloud drive. The agent spins up the sub-agents it needs (status, dependency, risk, meeting, comms, decision log) based on that context, and acts as your proxy, including inside Slack channels.

## How it works

```
You (manager)
 └─ Chief of Staff (portfolio level)
     ├─ Program agent: Atlas          ├─ Program agent: Orion
     │   └─ sub-agents on demand      │   └─ sub-agents on demand
     └─ Portfolio view (reads every program folder)
```

1. `cos-setup-program` interviews you, then writes the program's files.
2. `cos-program-agent` runs the program from those files, on a schedule or on demand, and answers in Slack.
3. `cos-portfolio` rolls every program into one view for you.
4. `cos-chief-of-staff` and `cos-second-brain` sit above and beside them: briefing, triage, and long-term memory.

## Chief of Staff and Second Brain

Three ideas run through the whole design:

- **Chief of Staff (acts).** Portfolio-level agent that briefs you daily, triages what comes in, delegates to program agents, tracks follow-ups, and runs the weekly review. Skill: `cos-chief-of-staff`.
- **Second Brain (remembers).** A `brain/` folder that outlives any one program: fast capture, stakeholder profiles, lessons learned, playbooks, weekly reviews. A librarian agent organizes it and every skill consults it. Skill: `cos-second-brain`.

- **Mentor (grows you).** A private career agent that logs your work across the portfolio, writes a weekly summary, and coaches you toward your target role with evidence, not flattery. Skill: `cos-mentor`.

Program folders are program memory. The brain is your memory. The chief of staff reads both.

## Program folder layout (PARA)

Every program agent gets one folder, organized with PARA (details in `docs/program-layout.md`):

```
<program>/
  0-agent/       how the agent works: identity, manual, schedule, slack.yaml, sources, pending, reporting, log
  1-projects/    what we deliver: knowledge, plans, specs, reports, customer-facing facts
  2-areas/       ongoing: risks, decisions, meetings/
  3-resources/   reference, plus linked-files.md (register of link-only files)
  4-archive/     inactive items, never deleted, with an archive log
```

`3-resources/linked-files.md` tracks files that cannot live in storage (Google Docs, Figma, Jira boards, dashboards) with owner, access, last-verified date, status, and a short summary. The portfolio view flags stale and broken links.

## Quick start

1. Install the plugin in Claude.
2. Copy `tpm-cos.config.yaml`, set your storage root.
3. Say: "Set up a new program agent." Give it a name you will remember (for example, Sherpa for Atlas) and answer the intake questions.
4. Review the generated files and approve. When asked, say yes to connect the agent to Slack (one-time app setup in `slack/setup.md`). The schedule then goes live.
5. Run the indexer for the portfolio view:
   `python portfolio/indexer/index.py <programs-root> -o portfolio.json`

Try it on the fictional programs in `examples/programs/`.

## How to use it

### One time
1. Install the plugin and copy `tpm-cos.config.yaml` to your working folder (for example `~/TPM-CoS/`). Set your name, time zone, and storage.
2. Say **"Set up Mentor"** and answer the career-goal questions. Mentor stays private to you.

### Create a program agent
Say **"Set up a new program agent"**. The agent asks for a name you will remember (for example, Sherpa for Atlas), then your goal, stakeholders, milestones, dependencies, sources, and reporting audiences, three or four questions at a time. It then asks whether to connect Slack (yes, or not now). You review the generated files and approve, and its schedule goes live. Each program gets its own folder with the PARA layout.

### Everyday phrases
| You say | What happens |
|---|---|
| "What needs my attention?" | Chief of Staff gives a brief of at most 5 items across all programs |
| "Status of Atlas" / "Ask Sherpa about risks" | The program agent answers from its files, with sources |
| "Run the dependency sweep for Atlas" | Pulls Jira data, flags silent, slipping, or blocked dependencies |
| "Prep me for the Atlas sync" | Context, open decisions, and suggested asks |
| "Capture: Sam says capacity may slip" | Saved to the second brain inbox |
| "Have we seen this before?" | Searches lessons and playbooks, with citations |
| "Draft the exec update for Atlas" | A draft in `0-agent/log/drafts/`, nothing sent until you approve |
| "Write my Mentor summary" | Weekly summary and coaching based on your work log and goals |
| "Connect Sherpa to Slack" | Walks you through Slack setup for that program |

### In Slack (after `slack/setup.md`)
- `@mention` the app in a program channel to ask a question. It answers only from approved program files and cites them. Questions it cannot support go to you.
- `/tpm status`, `/tpm risks`, `/tpm drafts` (approvers approve or reject drafts), `/tpm new` (create a program), `/tpm portfolio` (your attention list).

### What runs on its own (after you install the scheduler)
| When | What |
|---|---|
| Daily, early | Portfolio index refresh |
| Weekdays 08:30 | Daily brief |
| Mon, Wed, Fri | Risk review, dependency sweep, weekly status drafts (per program schedule) |
| Fri afternoon | Weekly review, brain sync, Mentor summary, portfolio digest DM to you |
| Before a meeting | Meeting prep draft |
| A milestone slips over 7 days | Escalation draft |
Everything it produces is a draft until you approve it.

### What you edit by hand
`0-agent/schedule.yaml` (when jobs run), `0-agent/slack.yaml` (channel and audience), `3-resources/linked-files.md` (link-only files), `1-projects/customer-facing.md` (what customers may see), and `mentor/profile.md` (your goals). Everything else the agents maintain.

### See the portfolio
Run the indexer (`docs/getting-started.md`, step 8) and open `portfolio/dashboard/index.html`.

## Design principles

- Files are the memory. Human-readable, versionable, forkable.
- Draft, then approve for anything going to execs or customers.
- Never guess. No source means "I'll check with the TPM", logged in 0-agent/pending.md.
- Read-only connectors by default.
- Storage is an adapter, so local or cloud is a config line.

## What is tested
`python -m pytest` runs 50 automated tests: storage adapters (against fakes of Drive, OneDrive, and S3, plus real folders for sync), connectors and analyzers, the scheduler, Slack core logic, and the indexer. Not tested against live services: Slack, Google Drive, OneDrive, S3, the Atlassian and Calendar MCP servers, and headless `claude -p` runs. Each has a first-run checklist in its README.

## Docs
`docs/getting-started.md`, `docs/architecture.md`, `docs/program-layout.md`, `slack/README.md`, `storage/README.md`, `connectors/README.md`, `scheduler/README.md`.

## Roadmap

- [x] v0.1 Structure, templates, skills, indexer
- [x] v0.2 Chief of Staff and Second Brain layers, portfolio dashboard (`portfolio/dashboard/index.html`)
- [x] v0.2 Named agents, Mentor career agent
- [x] v0.3 Slack app (Socket Mode): mentions, `/tpm` commands, approval buttons, optional connect during setup
- [x] v0.3 PARA program layout and linked-files register
- [x] v0.4 Cloud storage adapters (Google Drive, OneDrive, S3) with mirror sync
- [x] v0.5 Connectors: Jira, Confluence, Calendar (MCP, snapshots, analyzers) and a scheduler for the cadence

## License

MIT
