---
name: bug-batch-decomposition
description: "Structured decomposition of numbered bug batches into well-clustered, appropriately-routed subtasks. Use this skill whenever you receive an issue that contains a numbered list of bugs to fix (e.g. '22 bugs', 'bug list', 'batch of defects'), when an issue has more than 3 bugs attached as a group, when asked to decompose a batch or sprint of issues, or when you are about to create subtasks from a set of bugs. Always invoke before creating any child issues from a multi-bug parent — do not begin creating subtasks until you have read the full bug inventory and posted a decomposition plan."
---

# Bug Batch Decomposition

You have received a batch of bugs to decompose into subtasks. The goal is a set of focused, well-scoped child issues that any agent can pick up without re-reading the entire batch — not a 1:1 mapping of bugs to subtasks, but a thoughtful grouping that surfaces shared root causes and assigns work to the right developer.

Decomposition done poorly means redundant PRs, re-opened bugs, and missed groupings. Done well, it cuts execution time in half.

## Step 1 — Read the full batch before acting

Read every bug in the list before you create a single subtask. Resist the urge to create an issue for the first bug you understand — that always leads to missed clusters.

As you read each bug, note:
- Which component or layer it lives in (UI, backend logic, API, auth, database)
- Whether it is a symptom that shares a root cause with another bug
- Whether it can be reproduced and scoped from the description alone, or whether a live investigation is needed first

Keep a running mental (or scratch) inventory:

```
Bug N — component: <X>  root-cause hint: <Y>  reproducible: yes/no/unclear
```

## Step 2 — Cluster by shared root cause or co-located component

Group bugs that share a root cause or that touch the same function/component. A fix for one bug in a cluster will almost always require reading and potentially touching the same code as the other bugs in the cluster — routing them separately wastes developer context.

**Cluster signals (any one is sufficient to group):**
- Same component name appears in multiple bug descriptions
- Same user action triggers multiple different bugs (e.g. "clicking Save also breaks X and Y")
- Multiple bugs describe the same wrong value appearing in different places (likely one data-flow error)
- Multiple layout/alignment bugs that affect the same view or widget
- Multiple "missing" items (missing buttons, missing fields) that are likely all handled by the same rendering function

**Do NOT over-cluster:** only group bugs you can actually fix with a single coherent PR. If two bugs are in the same component but clearly require different investigations, keep them separate.

## Step 3 — Score each cluster and route to the right developer

For each cluster, assign a developer type based on the dominant work involved:

| Work type | Route to |
|-----------|----------|
| UI layout, alignment, missing visual elements, CSS/styling, label text | **Codex** (Full-Stack Developer — Codex) |
| Frontend behaviour, state management, form validation, client-side logic | **Claude** developer (Full-Stack Developer — Claude) |
| Backend logic, data processing, business rules, API responses | **Backend Specialist** |
| API integration, external service calls, token handling, auth flows | **Integration Specialist** |
| Cross-cutting or uncertain routing with both frontend + backend touchpoints | **Claude** developer as primary; note the cross-cutting nature in the subtask |

If a cluster contains mixed work types, use the type of the most complex bug in the cluster.

## Step 4 — Flag investigation-first items

If a bug cannot be scoped without live reproduction — the description is ambiguous, the trigger condition is unclear, or the fix range is wide enough that you could easily spend 3× the estimated time going in the wrong direction — do **not** create a Fix subtask. Create an **Investigate** subtask instead:

- Title: `Investigate: <short description of what is unknown>`
- Description: what specifically needs to be determined before a fix can be scoped (reproduction steps, affected data paths, error logs to examine)
- Assignee: the developer best suited to the reproduction environment
- No code changes expected from this subtask; the output is a comment that scopes the Fix subtask to follow

Create the Fix subtask as a child of the Investigate subtask (or block Fix on Investigate), not as a separate parallel item.

## Step 5 — Post a decomposition comment on the parent before creating child issues

Before you create any child issue, post a comment on the parent issue that lists your proposed subtask plan. This gives the board a chance to redirect groupings that look wrong from the outside.

Format:

```
## Decomposition Plan

| # | Title | Cluster | Developer | Priority |
|---|-------|---------|-----------|----------|
| 1 | Fix: <cluster title> (bugs N, N) | <component> | <developer type> | <priority> |
| 2 | Investigate: <ambiguous bug> | <component> | <developer type> | <priority> |
...

Proceeding to create these subtasks unless redirected.
```

This comment is not optional — it creates an audit trail and catches routing mistakes before they become misdirected PRs.

## Step 6 — Write subtask titles that name the cluster, not just one bug

Subtask titles should describe the cluster as a unit. A reader should understand the scope from the title alone without having to read the description.

**Good:** `Fix: Missing action buttons on line items, assumptions, and risk rows (bugs 1–3)`
**Bad:** `Fix bug 1: missing add button on line items`

**Good:** `Fix: Auth token not refreshed on API 401 responses (bug 7)`
**Bad:** `Fix the auth bug`

Include the bug numbers in parentheses at the end of the title so the parent batch traceability is preserved.

## Step 7 — Create child issues

For each cluster:

```
POST /api/companies/{companyId}/issues
{
  "title": "<cluster title>",
  "description": "<which bugs are included, what shared root cause you observed, acceptance criteria>",
  "parentId": "<parent issue id>",
  "assigneeAgentId": "<developer agent id>",
  "priority": "<inherit from parent unless a specific bug is lower severity>",
  "goalId": "<parent goal id if set>"
}
```

Reference each bug number explicitly in the description body. Include acceptance criteria so the developer knows when the cluster is done without re-reading the batch.

## Step 8 — Post a completion comment

After all subtasks are created, post a final comment on the parent listing each child issue with a link:

```
## Subtasks Created

- [TEC-XXXX](/TEC/issues/TEC-XXXX) — Fix: <cluster title>
- [TEC-YYYY](/TEC/issues/TEC-YYYY) — Investigate: <ambiguous bug>
...

All bugs accounted for. Parent remains in_progress until children are done.
```

## Priority and Severity Inheritance

Unless the bug proposal specifies otherwise, inherit the parent issue's priority on every subtask. If one specific bug is described as a minor cosmetic defect within a high-priority batch, you may set that subtask to `low` — but call this out explicitly in the decomposition comment.

## Sizing Heuristics

- A cluster of 1–4 simple UI bugs: one subtask, route to Codex, estimated 1–2 heartbeats.
- A single backend logic bug: one subtask, route to Backend Specialist or Claude developer, estimated 1–3 heartbeats.
- An ambiguous bug with no reproduction path: Investigate subtask first, Fix subtask created after investigation resolves.
- Cross-cutting cluster (frontend + backend): one subtask routed to Claude developer with a note that cross-layer changes are expected.

Do not create subtasks for more than 8–10 bugs at once. If the batch is larger, decompose in waves and note in the parent comment which wave is covered by the current subtasks.

---

*TEC Custom Skill — maintained by the Deltek Technical Services Engineering team.*
