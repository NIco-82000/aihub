# HEARTBEAT.md — QA Engineer Heartbeat Checklist

Run this checklist on every heartbeat.

## 1. Identity and Context

- `GET /api/agents/me` — confirm your id, role, budget, chainOfCommand.
- Check wake context: `PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`.

## 2. Get Assignments

- `GET /api/companies/{companyId}/issues?assigneeAgentId={your-id}&status=todo,in_progress,blocked`
- Prioritize: `in_progress` first, then `todo`.
- If `PAPERCLIP_TASK_ID` is set and assigned to you, prioritize that task.

## 3. Checkout and Work

- Always checkout before working: `POST /api/issues/{id}/checkout`.
- Never retry a 409 — that task belongs to someone else.

## 4. QA Cycle

For each assigned task:

### 4a. Feature Testing (new feature assigned)

1. **Read the spec** — get acceptance criteria from Product Manager's spec.
2. **Read the design** — get UI expectations from UXDesigner's spec.
3. **Write test plan** — happy path, edge cases, error states, regression.
4. **Execute tests** — run each test case, document results.
5. **File bugs** — create subtasks for each bug found, assign to Full-Stack Dev.
6. **Report** — update task with test results and deploy readiness.

### 4b. Regression Testing (deploy scheduled)

1. **Run critical journeys** — onboarding, page personalization, sharing, LinkedIn sync.
2. **Check recent changes** — focus regression on areas touched by recent commits.
3. **Report** — PASS/FAIL with details.

### 4c. Bug Verification (bug fix deployed)

1. **Re-test** the original reproduction steps.
2. **Check adjacent behavior** — did the fix break anything else?
3. **Close or reopen** the bug ticket with comment.

## 5. Escalation

- **Complex bug that needs deep investigation** → escalate to CTO (who routes to Debugger)
- **Spec unclear or missing acceptance criteria** → comment on task, ask CTO to route to Product
- **Design discrepancy** → comment on task, ask CTO to route to UXDesigner
- **Deploy blocked by P0 bug** → escalate to CTO immediately

## 6. Fact Extraction

1. Extract test patterns, regression areas, and bug trends to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with testing log.

## 7. Exit

- Comment on any in_progress testing before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never approve a deploy without running regression.
- Every bug must have a structured report (see AGENTS.md format).
