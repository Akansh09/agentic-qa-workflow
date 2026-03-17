# Coverage Checklist

For **EACH requirement**, verify test cases exist for:

## Functional Dimensions

| Dimension | Check |
|-----------|-------|
| Happy path | Normal use case works |
| Empty/null inputs | Handles missing data |
| Invalid inputs | Wrong type, format rejected |
| Boundary values | Min, max, just over/under |
| Special characters | Unicode, emojis, escapes |
| Large data volumes | Performance under load |

## State Dimensions

| Dimension | Check |
|-----------|-------|
| First-time use | Clean state behavior |
| After previous action | Dirty state handling |
| Concurrent access | Multiple users/sessions |
| After error recovery | Graceful recovery |
| After system restart | Persistence verified |

## User Journey Dimensions

| Dimension | Check |
|-----------|-------|
| Complete happy path | End-to-end journey |
| Interrupted journey | User abandons mid-flow |
| Retry after failure | Recovery flow works |
| Back navigation | Undo/back behavior |
| Cross-feature interactions | Feature combinations |

## Platform Dimensions

| Dimension | Check |
|-----------|-------|
| Different browsers | Chrome, Firefox, Safari |
| Different OS versions | Windows, macOS, Linux |
| Different screen sizes | Desktop, tablet, mobile |
| Slow network | Timeout handling |

## Security Dimensions

| Dimension | Check |
|-----------|-------|
| Input sanitization | XSS, SQL injection, path traversal |
| Authentication | Session management, token expiry |
| Authorization | Privilege escalation, role boundaries |
| Data exposure | Error messages, API responses |

## Assertion Dimensions

| Action Type | Required Assertion |
|-------------|-------------------|
| API call | Assert on status code and response body |
| Database query | Assert on result value/count |
| UI interaction | Assert on visible state change |
| State transition | Assert on new state value |

> **Rule**: No "fire and forget" actions without verification!
