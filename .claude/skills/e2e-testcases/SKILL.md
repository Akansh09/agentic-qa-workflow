---
name: e2e-testcases
description: "RFC-driven test case generation through 5 phases using specialized agents: RFC Analysis, Impact Analysis, Test Case Proposal, Multi-Reviewer Critique, and TMS Integration."
---

# E2E Test Case Generation & Review

Generate comprehensive, critique-validated test cases from RFC/JIRA/Confluence/PR input.

## Workflow: Spawn Agents Sequentially

Execute phases by spawning specialized agents. **STOP after each phase** — present results and wait for user confirmation before spawning the next agent.

### Phase 1-2: RFC & Impact Analysis
**Agent**: `rfc-analysis`
**Input**: The user's RFC source (JIRA ticket, Confluence URL, GitHub PR, or text)
**Returns**: RFC summary + impact analysis + risk matrix

→ Present results to user. Wait for confirmation. Store the output.

### Phase 3: Test Case Proposal
**Agent**: `test-case-proposal`
**Input**: RFC summary + impact analysis from Phase 1-2 (pass as prompt context)
**Returns**: Classified test cases with priority, requirement mapping, coverage summary

→ Present test cases to user. Wait for review/modifications.

### Phase 4: Critique & Validation
**Agent**: `critique-validation`
**Input**: RFC summary + impact analysis + proposed test cases (pass all as prompt context)
**Returns**: 5 reviewer reports, gap analysis, final coverage score, PASS/FAIL verdict

→ Present critique results. If FAIL (coverage < 85%), iterate. Wait for user approval.

### Phase 5: TMS Integration
**Agent**: `tms-integration`
**Input**: Final approved test cases + TMS config + JIRA ticket ID (if any)
**Returns**: Creation results, linked tickets, summary

→ Present TMS results. Workflow complete.

## Key Rules

1. **STOP between every phase** — never auto-proceed
2. **Pass accumulated context** — each agent needs the output of previous phases
3. **User can modify** — between phases, user may add/remove/change test cases
4. **Coverage gate** — Phase 4 must reach ≥85% before Phase 5
5. **Phase 5 is optional** — user can skip TMS integration

## Input

$ARGUMENTS
