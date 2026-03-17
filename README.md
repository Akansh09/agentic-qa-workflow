# Agentic QA Workflow

Generate comprehensive, critique-validated test cases from PRDs/RFCs using Claude Code agents.

Feed it a JIRA ticket, Confluence doc, GitHub PR, or plain text requirements — it produces prioritized test cases through a 5-phase pipeline with multi-reviewer critique.

## How It Works

```mermaid
flowchart TD
    A["🎯 /e2e-testcases &lt;PRD input&gt;"] --> B

    subgraph Phase1["Phase 1-2: RFC Analysis"]
        B["🤖 rfc-analysis agent"]
        B --> B1["Parse JIRA / Confluence / PR / RFC text"]
        B1 --> B2["Read product docs for regression context"]
        B2 --> B3["📄 RFC Summary + Impact Matrix + Risk Assessment"]
    end

    B3 --> C1{"👤 User Review\n& Confirmation"}

    C1 --> D

    subgraph Phase3["Phase 3: Test Case Proposal"]
        D["🤖 test-case-proposal agent"]
        D --> D1["Classify scenario: CRUD / Journey / Integration"]
        D1 --> D2["Apply test patterns & coverage checklist"]
        D2 --> D3["📋 Test Cases: P0/P1/P2 × [A]uto / [M]anual"]
    end

    D3 --> E1{"👤 User Review\n& Modifications"}

    E1 --> F

    subgraph Phase4["Phase 4: Critique & Validation"]
        F["🤖 critique-validation agent"]
        F --> G1["R1: Test Architect"]
        F --> G2["R2: Security Specialist"]
        F --> G3["R3: Business Analyst"]
        F --> G4["R4: Integration Engineer"]
        F --> G5["R5: Devil's Advocate"]
        G1 & G2 & G3 & G4 & G5 --> H["📊 Coverage Score ≥85%?"]
    end

    H -->|"❌ Below 85%"| I["Iterate: Add missing tests"] --> F
    H -->|"✅ Pass"| J1{"👤 User Approval"}

    J1 --> K

    subgraph Phase5["Phase 5: TMS Integration"]
        K["🤖 tms-integration agent"]
        K --> K1["Create folder in TMS"]
        K1 --> K2["Push test cases via API"]
        K2 --> K3["Link JIRA tickets"]
        K3 --> K4["✅ Done — Test cases live in TMS"]
    end

    style Phase1 fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style Phase3 fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style Phase4 fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style Phase5 fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style A fill:#e94560,stroke:#e94560,color:#fff
    style C1 fill:#0f3460,stroke:#533483,color:#e0e0e0
    style E1 fill:#0f3460,stroke:#533483,color:#e0e0e0
    style J1 fill:#0f3460,stroke:#533483,color:#e0e0e0
    style H fill:#533483,stroke:#533483,color:#e0e0e0
    style K4 fill:#2eb086,stroke:#2eb086,color:#fff
```

### Agent Pipeline (Text Version)

```
/e2e-testcases <PRD input>
       │
       ├─► Agent: rfc-analysis         → Parses requirements, maps impact & regression risks
       │   ⏸ User reviews
       │
       ├─► Agent: test-case-proposal   → Generates test cases (P0/P1/P2, positive/negative/edge)
       │   ⏸ User reviews
       │
       ├─► Agent: critique-validation  → 5 reviewer personas critique in parallel, scores coverage
       │   ⏸ Must reach ≥85% coverage
       │
       └─► Agent: tms-integration      → Pushes approved test cases to your TMS via API
```

## Input Sources

```mermaid
flowchart LR
    subgraph Sources["Input Sources"]
        J["🎫 JIRA Ticket"]
        C["📄 Confluence Doc"]
        P["🔀 GitHub PR"]
        T["📝 Plain Text RFC"]
    end

    J --> M["🔗 Merge & Enrich"]
    C --> M
    P --> M
    T --> M

    M --> |"Auto-discovers\nlinked PRs"| A["🤖 rfc-analysis agent"]

    style Sources fill:#1a1a2e,stroke:#16213e,color:#e0e0e0
    style M fill:#533483,stroke:#533483,color:#e0e0e0
    style A fill:#e94560,stroke:#e94560,color:#fff
```

| Input | Example |
|-------|---------|
| JIRA Ticket | `/e2e-testcases JIRA-5935` |
| Confluence Doc | `/e2e-testcases https://site.atlassian.net/wiki/.../pages/123/Title` |
| GitHub PR | `/e2e-testcases https://github.com/org/repo/pull/42` |
| Plain RFC | `/e2e-testcases Add OAuth2 login with Google and GitHub providers...` |
| Combined | `/e2e-testcases JIRA-5935 https://site.atlassian.net/wiki/.../pages/123` |

## Project Structure

```mermaid
flowchart TD
    subgraph Root["📁 agentic-qa-workflow"]
        CM["📄 CLAUDE.md — Auto-loaded context"]

        subgraph Claude["📁 .claude/"]
            subgraph Agents["📁 agents/"]
                A1["rfc-analysis.md"]
                A2["test-case-proposal.md"]
                A3["critique-validation.md"]
                A4["tms-integration.md"]
            end

            subgraph Docs["📁 docs/"]
                subgraph Product["📁 product/"]
                    P1["login-authentication.md"]
                    P2["+ your-feature.md"]
                end
                D1["coverage-checklist.md"]
                D2["test-patterns.md"]
            end

            subgraph Skills["📁 skills/"]
                S1["e2e-testcases/SKILL.md"]
                S2["e2e-improve/SKILL.md"]
            end

            ST["settings.local.json"]
        end
    end

    S1 -.->|orchestrates| A1
    A1 -.->|reads| P1
    A3 -.->|reads| D1
    A3 -.->|reads| D2

    style Root fill:#0d1117,stroke:#30363d,color:#e0e0e0
    style Claude fill:#161b22,stroke:#30363d,color:#e0e0e0
    style Agents fill:#1a1a2e,stroke:#e94560,color:#e0e0e0
    style Docs fill:#1a1a2e,stroke:#2eb086,color:#e0e0e0
    style Product fill:#1f2937,stroke:#2eb086,color:#e0e0e0
    style Skills fill:#1a1a2e,stroke:#533483,color:#e0e0e0
```

## Quick Start

### Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- (Optional) [Atlassian MCP](https://github.com/anthropics/mcp-atlassian-server) for JIRA/Confluence
- (Optional) [GitHub CLI](https://cli.github.com/) for PR context

### Usage

```bash
# Clone and open in Claude Code
git clone https://github.com/<your-username>/agentic-qa-workflow.git
cd agentic-qa-workflow
claude

# Run the test case generation pipeline
/e2e-testcases <your input>
```

## The 5 Phases

### Phase 1-2: RFC Analysis & Impact Assessment (`rfc-analysis` agent)

Parses requirements from any input source. Fetches JIRA details, Confluence pages, linked PRs. Reads product docs to identify regression risks. Produces a structured RFC summary + impact matrix.

### Phase 3: Test Case Proposal (`test-case-proposal` agent)

Classifies the scenario (CRUD, User Journey, Integration, Configuration). Generates test cases across all priorities:
- **P0 Critical**: Happy path, negative, user journeys
- **P1 High**: Edge cases, boundary values
- **P2 Medium**: Regression, cross-feature

Each test case marked `[A] Automatable` or `[M] Manual-Only`.

### Phase 4: Critique & Validation (`critique-validation` agent)

```mermaid
flowchart LR
    TC["📋 Proposed\nTest Cases"] --> R1 & R2 & R3 & R4 & R5

    R1["🏗️ R1: Test Architect\nDesign & patterns"]
    R2["🔒 R2: Security\nVulnerabilities & edges"]
    R3["💼 R3: Business\nJourneys & acceptance"]
    R4["⚡ R4: Integration\nAPIs & concurrency"]
    R5["😈 R5: Devil's Advocate\nChallenge everything"]

    R1 & R2 & R3 & R4 & R5 --> Score["📊 Coverage\nScore"]

    Score -->|"≥85%"| Pass["✅ Approved"]
    Score -->|"<85%"| Fail["🔄 Iterate"]
    Fail --> TC

    style R5 fill:#e94560,stroke:#e94560,color:#fff
    style Pass fill:#2eb086,stroke:#2eb086,color:#fff
    style Fail fill:#e94560,stroke:#e94560,color:#fff
    style Score fill:#533483,stroke:#533483,color:#e0e0e0
```

Calculates coverage score. Must reach **≥85% overall** and **100% requirement coverage** to proceed.

### Phase 5: TMS Integration (`tms-integration` agent)

Pushes approved test cases to your Test Management System (TestRail, Zephyr, qTest, Xray, or custom API). Links JIRA tickets for traceability.

## Customization

### Add Your Product Knowledge

Add your product docs as `.md` files in `.claude/docs/product/`. One file per feature area (e.g., `login.md`, `payments.md`, `notifications.md`). See `.claude/docs/product/README.md` for the recommended structure. The more detail you provide about features, flows, edge cases, and API endpoints, the better the test cases.

### Modify Coverage Standards

Edit `.claude/agents/critique-validation.md` to adjust:
- Coverage dimension weights
- Minimum thresholds (default: 85% overall)
- Reviewer personas and focus areas

### Add More Test Patterns

Extend `.claude/docs/test-patterns.md` with domain-specific patterns (e.g., payment flows, file upload, real-time collaboration).

## Claude Code Features Used

| Feature | How It's Used |
|---------|---------------|
| **CLAUDE.md** | Auto-loaded project context every conversation |
| **Skills** (`.claude/skills/`) | `/e2e-testcases` and `/e2e-improve` slash commands |
| **Agents** (`.claude/agents/`) | Specialized subprocesses for each phase |
| **Hooks** (`settings.local.json`) | Phase-gate reminder on stop |
| **MCP Servers** | Atlassian (JIRA/Confluence), Slack, Jam |

## License

MIT
