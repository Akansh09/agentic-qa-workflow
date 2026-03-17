# Product Knowledge

This directory contains product documentation that agents read during test case generation.

## How It Works

Agents reference these files during **Phase 1-2 (RFC Analysis & Impact Assessment)** to:
- Understand existing product behavior
- Identify regression risks from the Feature Dependency Maps
- Map RFC changes against existing features
- Discover edge cases from documented error states

## Adding Your Product Docs

Add one `.md` file per feature area. The more detail you provide, the better the generated test cases.

### Recommended Structure Per File

```markdown
# Feature Name

## Overview
Brief description of what this feature does.

## User Flows
Step-by-step flows (happy path + alternatives).

## Validation Rules
Input constraints, rate limits, business rules.

## Error States
Every error condition and its user-facing message.

## Edge Cases
Known tricky scenarios, race conditions, boundary behaviors.

## API Endpoints
| Endpoint | Method | Purpose |
|----------|--------|---------|

## Feature Dependency Map
| Feature | Depends On | Depended By |
|---------|------------|-------------|

## State Diagram (if applicable)
State transitions with valid/invalid paths.
```

### Tips

- **Be specific about error messages** — agents use these to generate negative test cases
- **Include rate limits and thresholds** — these become boundary value test cases
- **Document dependencies** — agents use the dependency map for regression risk analysis
- **List API endpoints** — enables integration test case generation
- **Describe state machines** — agents apply the State Machine test pattern automatically

## Example Files

- `login-authentication.md` — Login flows (email/password, OAuth, SSO, MFA, magic link, sessions)
- Add more: `payments.md`, `user-management.md`, `notifications.md`, etc.
