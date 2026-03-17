---
name: e2e-testcases
description: "RFC-driven test case generation through 5 phases using specialized agents: RFC Analysis, Impact Analysis, Test Case Proposal, Multi-Reviewer Critique, and TMS Integration."
---

# E2E Test Case Generation & Review

Generate comprehensive, critique-validated test cases from RFC/JIRA/Confluence/PR input.

## Workflow: Spawn Agents Sequentially

Execute phases by spawning specialized agents. **STOP after each phase** — present results and wait for user confirmation before spawning the next agent.

**CRITICAL**: For each phase, you MUST:
1. **Read the agent definition file** from `.claude/agents/<agent-name>.md`
2. **Use the agent definition's full instructions as the agent prompt** — include ALL sections (input parsing, steps, output format)
3. **Append the accumulated context** from prior phases to the prompt
4. Do NOT paraphrase or summarize the agent instructions — pass them verbatim so the agent follows the exact output format

### Phase 1-2: RFC & Impact Analysis
**Agent definition**: Read `.claude/agents/rfc-analysis.md` and use its FULL contents as the agent prompt
**Append to prompt**: The user's RFC source (JIRA ticket, Confluence URL, GitHub PR, or text)
**Returns**: RFC summary (title, what, why, boundaries, components) + impact analysis (DIRECT IMPACT, INDIRECT IMPACT, REGRESSION RISK sections) + risk matrix

→ Present results to user. Wait for confirmation. Store the output.

### Phase 3: Test Case Proposal
**Agent definition**: Read `.claude/agents/test-case-proposal.md` and use its FULL contents as the agent prompt
**Append to prompt**: RFC summary + impact analysis output from Phase 1-2
**Returns**: Classified test cases with priority, requirement mapping, coverage summary

→ Present test cases to user. Wait for review/modifications.

### Phase 4: Critique & Validation
**Agent definition**: Read `.claude/agents/critique-validation.md` and use its FULL contents as the agent prompt
**Append to prompt**: RFC summary + impact analysis + proposed test cases from prior phases
**Returns**: 5 reviewer reports, gap analysis, final coverage score, PASS/FAIL verdict

→ Present critique results. If FAIL (coverage < 85%), iterate. Wait for user approval.

### Phase 5: TMS Integration
**Agent definition**: Read `.claude/agents/tms-integration.md` and use its FULL contents as the agent prompt
**Append to prompt**: Final approved test cases + TMS config + JIRA ticket ID (if any)
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
