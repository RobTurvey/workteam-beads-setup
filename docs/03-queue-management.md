# 03 — Queue Management (mem-next, mem-active, mem-ops)

Queue separation keeps delivery flow visible and predictable.

## Commands

- `make mem-ready` — all ready issues from Beads
- `make mem-next` — strict claimable delivery queue
- `make mem-active` — active delivery work + epics
- `make mem-ops` — operational queue

## Usage Pattern

1. `make mem-next` — pick next delivery item
2. `make mem-active` — review active delivery context
3. `make mem-ops` — triage operational load separately

## Why This Matters

Without queue split:

- Operational issues can bury feature delivery
- Agents pick inconsistent priorities
- Epic context is harder to track

With queue split:

- Delivery remains visible and prioritized
- Active work is easy to monitor
- Ops work still tracked without disrupting execution

## Suggested Team Ritual

At daily kickoff:

```bash
make mem-next
make mem-active
make mem-ops
```

At handoff:

```bash
make mem-handoff
```

## Common Anti-Patterns

- Picking from `mem-ops` for product work
- Closing issues before merge/push
- Working unclaimed items
- Ignoring blocked status/escalation flow
