# Keycloak 7-Module Demo — Import & Presentation Guide

## What's in this file
`college-demo-realm.json` pre-builds these modules automatically on import:

| # | Module | What's pre-configured |
|---|--------|------------------------|
| 1 | Realm | `college-demo` realm, events logging on |
| 2 | Client | `student-portal` (OpenID Connect, public client) |
| 3 | Users | `alice` (admin) and `bob` (student), password `Pass@1234` |
| 4 | Roles | `admin` and `student` realm roles, assigned to the users |
| 5 | Strong password policy + MFA | min 8 chars, uppercase, digit, special char; TOTP required on first login |
| 6 | Brute-force protection | locks after 3 failed attempts |
| 7 | Events | login/logout/failure logging turned on |

---

## Step 1 — Start Keycloak
1. In Docker Desktop, click **▶ Play** on your `ecstatic_pascal` container (or `docker start ecstatic_pascal`).
2. Wait ~30–60 seconds, then open **http://localhost:8080**.
3. Click **Administration Console** and log in with your admin credentials.

## Step 2 — Import the realm
1. Top-left realm dropdown → **Create realm**.
2. Click **Browse** and select `college-demo-realm.json`.
3. Click **Create**.
4. Confirm you now see the `college-demo` realm with:
   - **Clients** → `student-portal`
   - **Realm roles** → `admin`, `student`
   - **Users** → `alice`, `bob`

If import fails on a field, your Keycloak version's export schema differs slightly — tell me the exact error and I'll adjust the JSON.

## Step 3 — Verify the security settings landed
- **Realm settings → Security defenses → Brute force detection** → should show enabled, max failures 3.
- **Authentication → Policies → Password policy** → should list length/uppercase/digit/special rules.
- **Authentication → Policies → OTP policy** → TOTP, 6 digits, 30-second period.
- **Realm settings → Events** → Save events should be on.

## Step 4 — Live demo in front of sir
1. Open a private/incognito window → `http://localhost:8080/realms/college-demo/account`
2. Log in as `alice` / `Pass@1234`.
3. Because of the `CONFIGURE_TOTP` required action, Keycloak will **show a QR code** — scan it with Google Authenticator or Microsoft Authenticator, enter the code, and finish login. This proves module 5 (MFA) live.
4. Open another private window and deliberately type the wrong password for `bob` **3 times**. On the 3rd attempt Keycloak locks the account. This proves module 6.
5. Go back to the admin console → **Users → bob** — he'll show as temporarily locked.
6. Open **Events** in the left menu → show the `LOGIN_ERROR` entries and the lockout. This proves module 7.
7. Walk through **Clients → student-portal**, **Realm roles**, and **Users → alice → Role mapping** to close out modules 1–4.

## Explaining to sir
"This is a real Keycloak identity server, not a simulation. I imported a pre-built configuration covering realms, clients, users, roles, a strong password policy, multi-factor authentication with an authenticator app, brute-force account lockout, and an audit log of every login event."

## If something doesn't match your Keycloak version
Realm export JSON format can shift slightly between Keycloak versions (your container's `keycloak/keycloak` tag). If any section is rejected on import, send me the error text and I'll fix the JSON to match your version.
