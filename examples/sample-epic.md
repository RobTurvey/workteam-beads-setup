# Sample Epic Lifecycle (Reference)

## Epic

- ID: `td-demo`
- Title: `pilot: validate multi-agent workflow`
- Status: `open` → `in_progress` → `closed`

## Child Tasks

- `td-demo.1` — Setup baseline templates (assignee: copilot)
- `td-demo.2` — Implement docs and queue commands (assignee: peer)
- `td-demo.3` — Review + finalize (assignee: copilot)

## Execution Trace

1. Create epic and child tasks
2. Assign tasks to agents
3. Each agent claims task before work
4. Commits include issue IDs
5. Peer review posted using checklist template
6. Issues closed only after merge/push
7. Session ends with `make mem-handoff`

## Expected Outcome

- Workflow executed end-to-end without ambiguity
- Audit trail preserved in Beads
- Team can repeat process consistently on new laptops
