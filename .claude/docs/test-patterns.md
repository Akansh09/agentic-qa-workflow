# Test Patterns Guide

Reusable test design patterns for comprehensive coverage.

## Pattern 1: CRUD Pattern

For any entity (user, project, resource):

| Test | Description |
|------|-------------|
| TC-1 | Create entity with valid data |
| TC-2 | Create entity with invalid data (each field) |
| TC-3 | Read entity after creation |
| TC-4 | Read non-existent entity (404 handling) |
| TC-5 | Update entity with valid data |
| TC-6 | Update entity with invalid data |
| TC-7 | Delete entity |
| TC-8 | Delete non-existent entity |
| TC-9 | Use deleted entity (should fail gracefully) |

## Pattern 2: State Machine Pattern

For entities with states (Draft → Active → Archived):

| Test | Description |
|------|-------------|
| TC-1 | Valid state transition (Draft → Active) |
| TC-2 | Invalid state transition (Archived → Draft) |
| TC-3 | Action in each state (edit in Draft, view in Active) |
| TC-4 | Boundary state (just entered, about to leave) |

## Pattern 3: Reference Pattern

For features that reference other entities:

| Test | Description |
|------|-------------|
| TC-1 | Use valid reference |
| TC-2 | Use non-existent reference |
| TC-3 | Use deleted reference (orphan handling) |
| TC-4 | Circular reference (A→B→A) |
| TC-5 | Deeply nested reference (A→B→C→D) |

## Pattern 4: Permission Pattern

For features with access control:

| Test | Description |
|------|-------------|
| TC-1 | Owner can perform action |
| TC-2 | Admin can perform action |
| TC-3 | Regular user cannot perform restricted action |
| TC-4 | Guest/anonymous cannot access |
| TC-5 | Cross-org access denied |

## Pattern 5: Async/Sync Pattern

For async operations (jobs, webhooks, notifications):

| Test | Description |
|------|-------------|
| TC-1 | Successful async completion |
| TC-2 | Async timeout handling |
| TC-3 | Async failure handling |
| TC-4 | Cancel async operation |
| TC-5 | Retry failed async operation |

---

## Pattern Application Example

**Feature**: Login Authentication

| Pattern | Applied Tests |
|---------|---------------|
| CRUD-1 | Create account with valid credentials |
| CRUD-2 | Create account with invalid email format |
| STATE-1 | Active → Locked (after failed attempts) |
| STATE-2 | Locked → Active (after timeout) |
| PERM-1 | Admin can reset any user's password |
| PERM-4 | Anonymous cannot access dashboard |
| ASYNC-1 | Magic link email delivered successfully |
| ASYNC-2 | Magic link expires after timeout |
