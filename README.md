# The MyE28 Community

This GH org contains the **source of truth** for MyE28 — the forums (www.mye28.com) and
wiki (wiki.mye28.com) for E28-chassis BMW enthusiasts worldwide.

<img width="1171" height="279" alt="image" src="https://github.com/user-attachments/assets/c1f28e21-31af-4813-aea4-0e8f0646602f" />


## The AI-agent administration model

MyE28 is a proof-of-concept for a new way to run a volunteer community forum:
**any admin — even a non-technical one — can maintain the infrastructure by
delegating to an AI coding agent.** The human sets the goal; the agent reads the
docs, proposes a plan, and executes it; the human approves or redirects.

This isn't an accident — the repo is *designed* that way. Here's how:

### 1. The repo is the brain

Every piece of knowledge needed to operate this system lives in Git, in plain
English: configuration decisions, architecture, runbooks, the location of every
credential, the conventions for making changes, and the current list of open
work. Nothing critical lives only in an admin's head or an undocumented config
file. If a new admin (human or AI) can read this repo, they can operate the
system.

### 2. `AGENTS.md` is an AI onboarding document

[`AGENTS.md`](AGENTS.md) is the single entry point for any agent — it tells the
agent what the system is, where to find the live system's state, what credentials
exist and where they're stored, what the working conventions are, and what the
current open work is. An AI coding agent pointed at this repo can orient itself
in a single read of that file and the linked docs, then safely ask clarifying
questions and propose changes.

### 3. The Constitution constrains the agent safely

[`CONSTITUTION.md`](CONSTITUTION.md) defines the Five Tenets the system must
uphold — with *Continuity* first. These rules are written explicitly enough that
an AI can follow them: no irreversible unilateral actions, recovery path before
any destructive step, always verify before acting. A non-technical admin can
hand a task to an AI and trust the system itself limits what the agent can safely
do.

### 4. 1Password is the trust anchor

Secrets never live in this repo. They live in a **community-owned 1Password
Teams vault** (`mye28.1password.com`). Docs reference credentials by vault name
and item name — the agent knows *where* to tell the human to retrieve them, but
the value never touches the repo or the agent's context. The vault's Service
Account enables non-interactive automation (e.g. the backup restore runbook). The
vault is also what makes **continuity** possible: rebuild the VM, rotate an admin,
hand the project to a successor — the credentials remain accessible to the
community, not locked to any one person.

The [break-glass bundle](ops/breakglass/README.md) takes this further: critical
recovery secrets are Shamir-split 2-of-3 across backup admins, so no single
person is a point of failure and no single person can act alone.

1Password is not incidental to this design — it is the linchpin. Without a
community-owned vault with a service account, the AI-maintainable model collapses
back to "only the person who knows the passwords can do anything."

### 5. The `ops/` scripts are the hands

Backup, rebuild-from-scratch, and break-glass recovery are fully scripted in
[`ops/`](ops/). An agent can execute them; a non-technical admin can follow them
step-by-step; a new maintainer can read them and trust they're complete. The
rebuild scripts have been proven end-to-end on a real DigitalOcean droplet.

### What a non-technical admin can do today

1. Open a GitHub issue describing a problem or goal.
2. Attach an AI coding agent (GitHub Copilot Tasks / Claude Code) and point it
   at this repo.
3. The agent reads `AGENTS.md`, consults the relevant docs, proposes a plan.
4. The admin reviews and approves — no terminal required.
5. The agent executes, commits any doc updates, and closes the issue.

---

## This pattern is reusable

The approach here — a Constitution, an `AGENTS.md` onboarding file, structured
infra docs, a community secrets vault, and scripted ops — is not specific to
MyE28. Any volunteer-run phpBB forum (or MediaWiki, or both) could adopt it.
The infrastructure stack (one VPS, Cloudflare for edge TLS and storage, restic
backups) is deliberately generic and inexpensive (~$6–12/mo at today's prices,
excluding legacy Azure).

If you run a similar community and want to adopt this model, the pieces are:

- **Fork this repo** and replace MyE28-specific content with your community's.
- **`AGENTS.md` and `CONSTITUTION.md`** are the hardest-won pieces — adapt them,
  don't skip them.
- **A community-owned 1Password Teams vault** (not a personal vault) with a
  Service Account. This is non-negotiable for the model to work safely across
  admin changes.
- The **`ops/`** scripts work against any similarly structured phpBB + MariaDB +
  Apache host.

