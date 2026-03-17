# Product Knowledge: Login & Authentication

This document describes the Login & Authentication feature for test case design context. Use this during RFC analysis (Phases 1-2) to understand existing product behavior and identify regression risks.

---

## Feature Overview

The application supports multiple authentication methods. Users can sign in through various flows depending on their account type and organization configuration.

---

## Authentication Methods

### 1. Email & Password Login

**Flow:**
1. User navigates to `/login`
2. Enters registered email address
3. Enters password
4. Clicks "Sign In" button
5. Redirected to `/dashboard` on success

**Validation Rules:**
- Email: Must be valid format, case-insensitive
- Password: Minimum 8 characters, requires uppercase + number + special character
- Rate limiting: 5 failed attempts → 15-minute lockout
- Session: JWT token stored in httpOnly cookie, 24-hour expiry

**Error States:**
- Invalid email format → inline validation error
- Wrong password → "Invalid credentials" (generic, no email enumeration)
- Account locked → "Account temporarily locked. Try again in X minutes."
- Unverified email → "Please verify your email first" with resend link
- Disabled account → "Account has been deactivated. Contact support."

---

### 2. Social Login (OAuth 2.0)

**Supported Providers:**
- Google (OpenID Connect)
- GitHub (OAuth 2.0)
- Microsoft (Azure AD)

**Flow:**
1. User clicks provider button on `/login`
2. Redirected to provider's consent screen
3. User authorizes the application
4. Callback to `/auth/callback/{provider}`
5. Account created (first-time) or matched (returning user) by email
6. Redirected to `/dashboard`

**Edge Cases:**
- Email mismatch: Social email differs from registered email → prompt to link accounts
- Provider unavailable: Show fallback to email/password login
- Revoked access: User revokes OAuth app → next login requires re-authorization
- Multiple providers: Same email linked to Google AND GitHub → either works

---

### 3. SSO / SAML Login

**Flow:**
1. User enters corporate email on `/login`
2. System detects SSO-enabled domain
3. Redirected to organization's Identity Provider (IdP)
4. User authenticates via IdP (Okta, Azure AD, OneLogin, etc.)
5. SAML assertion sent back to `/auth/saml/callback`
6. User provisioned (JIT) or matched to existing account
7. Redirected to `/dashboard`

**Configuration:**
- SSO enforced at org level → email/password login disabled for SSO users
- Just-In-Time (JIT) provisioning: Auto-create accounts on first SSO login
- Domain verification required before SSO activation
- IdP-initiated vs SP-initiated login supported

**Edge Cases:**
- SSO misconfiguration → graceful error page with admin contact info
- IdP timeout → "Authentication provider did not respond. Try again."
- User removed from IdP → next login fails, account remains in "suspended" state

---

### 4. Multi-Factor Authentication (MFA / 2FA)

**Supported Methods:**
- TOTP (Time-based One-Time Password) via authenticator apps (Google Authenticator, Authy)
- SMS verification code
- Email verification code
- Hardware security keys (WebAuthn/FIDO2)

**Flow (after primary authentication):**
1. User completes email/password or social login
2. MFA challenge screen appears
3. User enters TOTP code / SMS code / email code OR touches security key
4. Verified → redirected to `/dashboard`

**Setup Flow:**
1. User navigates to `/settings/security`
2. Clicks "Enable MFA"
3. Scans QR code with authenticator app
4. Enters verification code to confirm setup
5. Receives backup/recovery codes (one-time use)

**Edge Cases:**
- Wrong TOTP code → "Invalid code. Please try again." (3 attempts before lockout)
- Expired TOTP code → code rotates every 30 seconds
- Lost authenticator → use recovery codes
- All recovery codes used → must contact support with identity verification
- SMS not received → "Resend code" button (rate limited: 1 per 60 seconds)
- Hardware key not recognized → fallback to TOTP/SMS

---

### 5. Magic Link Login (Passwordless)

**Flow:**
1. User enters email on `/login/magic-link`
2. System sends email with unique, time-limited login link
3. User clicks link in email
4. Link validated → user authenticated → redirected to `/dashboard`

**Constraints:**
- Link expires after 15 minutes
- Single-use: link invalidated after first click
- Rate limited: max 3 magic links per hour per email
- Link contains signed JWT with user identity + expiry + nonce

**Edge Cases:**
- Expired link → "This link has expired. Request a new one."
- Already-used link → "This link has already been used."
- Multiple links requested → only the latest link is valid (previous ones invalidated)
- Link opened in different browser/device → works (not device-bound)

---

### 6. Remember Me & Session Management

**Session Behavior:**
- Default session: 24-hour JWT expiry
- "Remember Me" checked: 30-day refresh token + 24-hour access token
- Refresh token rotation on each use
- Concurrent sessions: maximum 5 active sessions per user

**Session Management UI (`/settings/sessions`):**
- View all active sessions (device, browser, IP, last active)
- Revoke individual sessions
- "Sign out all devices" button
- Session activity notifications (optional)

**Edge Cases:**
- Expired access token + valid refresh token → silent refresh
- Expired refresh token → redirect to login
- 6th concurrent session → oldest session automatically revoked
- Password change → all sessions except current revoked
- Account lockout → all sessions immediately revoked

---

## Feature Dependency Map

| Feature | Depends On | Depended By |
|---------|------------|-------------|
| Email/Password Login | User Registration, Email Verification | Dashboard, All authenticated features |
| Social Login | OAuth Provider Availability | Account Linking, User Profile |
| SSO/SAML | Org Configuration, IdP Setup | JIT Provisioning, Domain Verification |
| MFA | Primary Authentication | Recovery Codes, Security Settings |
| Magic Link | Email Service | Session Creation |
| Session Management | Authentication (any method) | All authenticated API calls |
| Password Reset | Email Service, User Registration | Email/Password Login |

---

## API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/auth/login` | POST | Email/password login |
| `/api/auth/logout` | POST | End session |
| `/api/auth/refresh` | POST | Refresh access token |
| `/api/auth/oauth/{provider}` | GET | Initiate OAuth flow |
| `/api/auth/oauth/callback` | GET | OAuth callback |
| `/api/auth/saml/login` | POST | Initiate SAML flow |
| `/api/auth/saml/callback` | POST | SAML assertion callback |
| `/api/auth/mfa/setup` | POST | Begin MFA setup |
| `/api/auth/mfa/verify` | POST | Verify MFA code |
| `/api/auth/magic-link` | POST | Request magic link |
| `/api/auth/magic-link/verify` | GET | Validate magic link |
| `/api/auth/password/reset` | POST | Request password reset |
| `/api/auth/password/reset/confirm` | POST | Confirm password reset |
| `/api/auth/sessions` | GET | List active sessions |
| `/api/auth/sessions/{id}` | DELETE | Revoke specific session |
| `/api/auth/sessions/all` | DELETE | Revoke all sessions |

---

## User States

```
Unverified ──[verify email]──► Active ──[disable]──► Disabled
                                  │
                                  ├──[lock (failed attempts)]──► Locked ──[timeout/unlock]──► Active
                                  │
                                  ├──[SSO suspend]──► Suspended ──[IdP re-enable]──► Active
                                  │
                                  └──[delete account]──► Deleted (soft, 30-day grace period)
```

## Password Reset Flow

1. User clicks "Forgot Password" on `/login`
2. Enters registered email
3. System sends reset link (expires in 1 hour, single-use)
4. User clicks link → `/reset-password?token=xxx`
5. Enters new password (must differ from last 5 passwords)
6. Password updated → all sessions revoked → redirect to `/login`

**Edge Cases:**
- Email not found → still show "Reset link sent" (prevent email enumeration)
- Expired token → "This reset link has expired"
- Password reuse → "Cannot reuse recent passwords"
- Reset during active session → current session preserved, others revoked
