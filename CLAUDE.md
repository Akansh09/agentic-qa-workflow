# Agentic QA Workflow

RFC-driven test case generation framework powered by Claude Code agents. Analyzes requirements from JIRA, Confluence, GitHub PRs, or plain RFC text to produce comprehensive, critique-validated test cases.

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| **E2E Test Cases** | `/e2e-testcases` | Full RFC → Test Case pipeline (Phases 1-5) |
| **E2E Improve** | `/e2e-improve` | Analyze bugs to find test coverage gaps |

## Agents (Phase Execution)

Each phase runs as a specialized agent with focused tools and instructions:

| Agent | Phase | What It Does | Key Tools |
|-------|-------|-------------|-----------|
| `rfc-analysis` | 1-2 | Parse RFC, analyze impact & regression risks | Atlassian MCP, GitHub CLI, WebFetch |
| `test-case-proposal` | 3 | Generate test cases with priority hierarchy | Read (reference docs) |
| `critique-validation` | 4 | 5-persona parallel critique, coverage scoring | Read (reference docs) |
| `tms-integration` | 5 | Push approved test cases to TMS via API | Bash (curl), Atlassian MCP |

### Agent Flow

```
/e2e-testcases <input>
    │
    ├─► Agent: rfc-analysis ──► RFC Summary + Impact Analysis
    │   STOP → User confirms
    │
    ├─► Agent: test-case-proposal ──► Test Cases (P0/P1/P2, [A]/[M])
    │   STOP → User reviews
    │
    ├─► Agent: critique-validation ──► 5 Reviews + Coverage Score
    │   STOP → Must reach ≥85% coverage
    │
    └─► Agent: tms-integration ──► Push to TMS + JIRA linking
        STOP → Workflow complete
```

## Key Constraints

- **ONE phase at a time** — complete fully before proceeding
- **STOP and WAIT** after each agent returns — user must confirm
- **Coverage gate** — Phase 4 must reach ≥85% overall, 100% requirement coverage
- **Pass context forward** — each agent receives accumulated output from prior phases

## Input Types

| Input | Example | How |
|-------|---------|-----|
| JIRA Ticket | `JIRA-5935` | Fetch via Atlassian MCP |
| Confluence Doc | `https://site.atlassian.net/wiki/...` | Fetch via Atlassian MCP |
| GitHub PR | `https://github.com/org/repo/pull/123` | Fetch via `gh` CLI |
| RFC Text | Pasted content | Use directly |
| Combined | JIRA + Confluence + PR | Merge all sources |

## Reference Docs (`.claude/docs/`)

| Document | Purpose |
|----------|---------|
| [Product Knowledge](.claude/docs/product/) | Product feature docs — add one file per feature area |
| [Coverage Checklist](.claude/docs/coverage-checklist.md) | Dimensions to verify for each requirement |
| [Test Patterns](.claude/docs/test-patterns.md) | Reusable patterns: CRUD, State Machine, Permission, Async |

## Integrations

- **Atlassian MCP** — JIRA issues, Confluence pages, remote links
- **GitHub CLI** — PR details (title, body, files, commits)
- **Slack MCP** — Thread reading for context
- **Jam MCP** — Bug video analysis (console logs, network errors)
