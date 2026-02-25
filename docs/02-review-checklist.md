# 02 — Peer Review Checklist (Team Standard)

Use this checklist exactly for all peer reviews between agents.

## Review Protocol

1. Confirm issue scope and acceptance criteria
2. Inspect changed files and commit messages
3. Run relevant checks (tests/lint/build where applicable)
4. Validate workflow discipline (claim, commit message issue ID, close timing)
5. Record findings with severity
6. Publish verdict (`PASS`, `NITS`, `FAIL`)
7. If disputed, escalate to team lead for decision

## Required Review Comment Template

```text
Review: <PASS|NITS|FAIL>
Issue: <id>
Scope:
- <what was reviewed>

Checks:
- <check 1>
- <check 2>

Findings:
- [high|med|low] <finding>

Recommendation:
- <merge now / fix before merge / blocked pending decision>

Disputes:
- <none | summary + escalation issue>
```

## Verdict Rules

- **PASS** — No blocking issues; acceptable quality for merge
- **NITS** — Non-blocking improvements requested
- **FAIL** — Blocking issue(s) must be resolved before merge

## Severity Guidance

- **high** — correctness/security/data loss/workflow integrity risk
- **med** — significant quality/maintainability risk
- **low** — style/clarity/minor improvement

## PASS Criteria

All must be true:

- Implementation matches issue scope
- No known blocking defects introduced
- Workflow protocol followed
- Commit message includes issue ID
- Appropriate tests/checks performed or justified
