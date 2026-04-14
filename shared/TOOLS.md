# TOOLS.md — Available Tools for All Agents

**This document lists the real, concrete tools available on the host machine.** Every agent should read this to know what they can actually execute via Bash.

---

## Shell Tools (available via Bash)

### Git

```bash
git --version   # git 2.53.0
```

All standard git operations: clone, branch, commit, push, pull, merge, rebase, log, diff, stash.

**Usage:** All engineering agents (CTO, Full-Stack Dev, DevOps, QA, Debugger).

**Conventions:** Follow the `git-workflow` skill for branching strategy and commit conventions.

---

### GitHub CLI (`gh`)

```bash
gh --version   # gh 2.89.0
```

Full GitHub interaction from the command line — no browser needed.

**Key commands:**

```bash
# Issues
gh issue list                          # List open issues
gh issue create --title "..." --body "..."  # Create issue
gh issue view 123                      # View issue details

# Pull Requests
gh pr list                             # List open PRs
gh pr create --title "..." --body "..."     # Create PR
gh pr view 123                         # View PR details
gh pr review 123 --approve             # Approve PR
gh pr review 123 --request-changes -b "..."  # Request changes
gh pr merge 123 --squash               # Merge PR
gh pr checks 123                       # Check CI status

# Code Review
gh pr diff 123                         # View PR diff
gh pr comment 123 --body "..."         # Comment on PR

# Repos
gh repo view                           # View current repo info
gh api repos/{owner}/{repo}/...        # Raw GitHub API calls
```

**Usage:** CTO (reviews, approvals), Full-Stack Dev (PRs, branches), DevOps (CI checks, releases).

**Important:** Requires authentication. Run `gh auth login` if not already authenticated.

---

### Vercel CLI (`vercel`)

```bash
vercel --version   # 51.1.0
```

Deploy, monitor, and manage Vercel projects.

**Key commands:**

```bash
# Deployments
vercel ls                              # List recent deployments
vercel inspect <url>                   # Inspect a deployment
vercel logs <url>                      # View deployment logs
vercel rollback                        # Rollback to previous deployment

# Projects
vercel project ls                      # List projects
vercel env ls                          # List environment variables

# Deploy
vercel --prod                          # Deploy to production
vercel                                 # Deploy to preview

# Domains
vercel domains ls                      # List domains
```

**Usage:** DevOps (deployments, monitoring, rollbacks), CTO (deploy oversight).

**Important:** Requires authentication. Run `vercel login` if not already authenticated.

---

### Node.js

```bash
node --version   # v22.22.2
npm --version
npx
```

Full Node.js runtime — can execute scripts, install packages, run dev servers.

**Usage:** Full-Stack Dev (development), DevOps (scripts, automation).

---

### curl

```bash
curl --version   # curl 8.7.1
```

HTTP requests to any API — the backbone of all Unipile API calls.

**Usage:** SDR, CSM, Content Strategist (LinkedIn via Unipile proxy), DevOps (API monitoring).

**Unipile proxy:** `$UNIPILE_BASE_URL` with `X-API-KEY: $UNIPILE_API_KEY` header. See `sonoria/` skills for detailed endpoint documentation.

---

## Environment Variables

Agents may need these environment variables. Check if they're set before using:

| Variable | Purpose | Used by |
|----------|---------|---------|
| `UNIPILE_BASE_URL` | Unipile proxy URL | SDR, CSM, Content Strategist |
| `UNIPILE_API_KEY` | Unipile authentication | SDR, CSM, Content Strategist |
| `UNIPILE_ACCOUNT_ID` | LinkedIn account in Unipile | SDR, CSM, Content Strategist |
| `GITHUB_TOKEN` | GitHub API authentication (if gh not authed) | CTO, Full-Stack Dev, DevOps |
| `VERCEL_TOKEN` | Vercel API authentication (if vercel not authed) | DevOps |

---

## What is NOT Available

- **MCP tools** — Vercel MCP, LinkedIn MCP, n8n MCP are configured at the human operator's Claude Code session level. Agents do NOT have access to them. Use the CLI tools above instead.
- **Browser** — Agents cannot open a browser or interact with web UIs. Use CLI and API calls.
- **Interactive prompts** — Agents cannot respond to interactive terminal prompts. Use `-y` flags or non-interactive modes.
- **sudo** — No root access. No commands requiring admin privileges.

---

## Quick Reference

| Need to... | Use |
|------------|-----|
| Create a PR | `gh pr create` |
| Review a PR | `gh pr diff` + `gh pr review` |
| Check CI status | `gh pr checks` |
| Deploy to Vercel | `vercel --prod` |
| Rollback a deploy | `vercel rollback` |
| View deploy logs | `vercel logs <url>` |
| Search LinkedIn | `curl` via `sonoria/linkedin-search` skill |
| Send LinkedIn DM | `curl` via `sonoria/linkedin-dm` skill (+ approval) |
| Read CRM data | `curl` via `sonoria/crm-pipeline` skill |
