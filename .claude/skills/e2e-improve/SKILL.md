---
name: e2e-improve
description: "Analyze bugs found during testing, identify gaps in test case coverage, propose new test cases, and recommend improvements to the test generation process."
---

# E2E Agent Improvement from Bug Feedback

Analyze bugs found during manual testing to identify coverage gaps and improve the test generation process.

## What This Does

1. **Fetch and categorize bugs** from the JIRA ticket/filter
2. **Extract bug details** including Jam video recordings (console logs, network errors, user events)
3. **Map bugs against existing test cases** to find coverage gaps
4. **Propose new test cases** for uncovered bug patterns
5. **Recommend improvements** to agent docs (patterns, checklists, phase templates)
6. **Optionally push new TCs** to TMS

## Input Types

| Input Type | Example | How It Works |
|------------|---------|--------------|
| **Parent ticket + Feature ticket** | `TE-6144 TE-1585` | Fetch child bugs from parent, map against TCs for feature |
| **JQL filter + Feature ticket** | `"parent = TE-6144 AND type = Bug" TE-1585` | Use JQL to fetch bugs |

## MCP Tools Used

- **Atlassian MCP**: Fetch JIRA bugs, comments, linked issues
- **Jam MCP**: Fetch video details, console logs, network errors for bugs with recordings
- **Slack MCP**: Read related threads for additional context

## Process

### Step 1: Fetch All Bugs
- Parse input to identify parent ticket and feature ticket
- Fetch all child bugs from parent ticket
- For each bug, extract: summary, description, priority, status, reproduction steps

### Step 2: Extract Jam Video Details (if available)
- For bugs with Jam recordings, use Jam MCP tools:
  - `mcp__Jam__getDetails` — Bug overview
  - `mcp__Jam__getScreenshots` — Visual evidence
  - `mcp__Jam__getVideoTranscript` — User actions
  - `mcp__Jam__getNetworkRequests` — Failed API calls
  - `mcp__Jam__getUserEvents` — Click/input sequence

### Step 3: Map Bugs to Existing Test Cases
- Cross-reference each bug against test cases generated for the feature
- Identify which bugs WOULD have been caught by existing TCs
- Identify which bugs represent GAPS in coverage

### Step 4: Gap Analysis & New Test Cases
For each gap, propose new test cases:
- Root cause category (edge case, race condition, integration, etc.)
- Priority based on bug severity
- Test steps to reproduce
- Expected vs actual behavior

### Step 5: Recommend Process Improvements
- Which test patterns should have caught these bugs?
- What should be added to the coverage checklist?
- What new edge case categories emerged?

**Present findings and wait for user approval before pushing new TCs.**

## Input

$ARGUMENTS
