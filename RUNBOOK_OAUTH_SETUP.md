# OAuth Setup Runbook

**Scope:** Apple Sign in with Apple (web flow). Google OAuth is deferred — when it lands under [web#50](https://github.com/izzy-yum/web/issues/50), append its section here.

**Audience:** whoever is on-call when Apple's client_secret JWT expires, or when credentials need to be rotated after a compromise / team change.

**Related:**
- Design: [`ACCOUNT_AND_PAIRING.md` §3.5](./ACCOUNT_AND_PAIRING.md#35-sign-in-method-flows)
- Issues: [web#38](https://github.com/izzy-yum/web/issues/38) (Apple), [web#50](https://github.com/izzy-yum/web/issues/50) (Google, deferred)
- Tooling: `web/scripts/generate-apple-client-secret.js`

---

## 1. Apple — what's configured

| Thing | Value | Where it lives |
|---|---|---|
| Primary App ID | `com.izzyyum.app` | Apple Developer → Identifiers |
| Services ID (OAuth client) | `com.izzyyum.app.web-signin` | Apple Developer → Identifiers |
| Sign in with Apple Key | Key ID `RY499TP287`, Team ID `VWFHD2N44D` | Apple Developer → Keys; `.p8` private key stored in 1Password |
| Supabase provider | Enabled on project `oogpnrrsmpklafdrvvkg` | Supabase Dashboard → Auth → Providers → Apple |
| Local dev config | `[auth.external.apple]` in `web/supabase/config.toml` (env-var secret) | Repo |
| Client secret generator | `web/scripts/generate-apple-client-secret.js` | Repo |

The Services ID's configured Return URLs are:

- `https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/callback` (Supabase cloud)
- `https://izzy-yum.com/auth/callback` (production web)

Domains and Subdomains registered with Apple:

- `oogpnrrsmpklafdrvvkg.supabase.co`
- `izzy-yum.com`

> **Note on domain verification:** Apple offers a `.well-known/apple-developer-domain-association.txt` file for claiming domain ownership. We **skipped** this for `izzy-yum.com` because the primary OAuth callback goes to `oogpnrrsmpklafdrvvkg.supabase.co`, which Apple pre-verifies via its partnership with Supabase. Only needed if we ever wire the direct `izzy-yum.com/auth/callback` path.

---

## 2. Client secret rotation (every ~180 days)

Apple's OAuth client secret is a JWT signed with the Sign-in-with-Apple key (ES256). Apple caps its lifetime at ~6 months, so we rotate on a fixed cadence.

### 2.1 When to rotate

- **Scheduled:** ~150 days after generation (gives a 30-day buffer before hard expiry).
- **On compromise:** immediately, if the `.p8` key file or the generated JWT leaked.
- **On team change:** if the Apple Developer account owner changes.

### 2.2 How to rotate

Prereqs:
- The `.p8` key file at a known path (stored in 1Password by default).
- Node.js 16+ (the generator uses `crypto.sign` with `dsaEncoding: 'ieee-p1363'`).

1. From `web/`, run the generator with the four required env vars:

   ```bash
   APPLE_TEAM_ID=VWFHD2N44D \
   APPLE_KEY_ID=RY499TP287 \
   APPLE_CLIENT_ID=com.izzyyum.app.web-signin \
   APPLE_P8_PATH=/path/to/AuthKey_RY499TP287.p8 \
   node scripts/generate-apple-client-secret.js | pbcopy
   ```

   The JWT is written to stdout. The expiry timestamp is written to stderr — record it somewhere reliable (calendar reminder, runbook log at the bottom of this file).

2. In the Supabase Dashboard (project `oogpnrrsmpklafdrvvkg`):
   - Authentication → Providers → Apple → **Secret Key (for OAuth)** — replace with the new JWT.
   - Leave Client IDs (`com.izzyyum.app.web-signin`) unchanged.
   - Save.

3. For local dev, export the env var before `supabase start` (or set it in `.env.local` and source it):

   ```bash
   export SUPABASE_AUTH_EXTERNAL_APPLE_SECRET="$(... generator command ...)"
   supabase stop && supabase start
   ```

4. Verify with a round-trip on `/auth/test` — sign in → check a new row appears in `auth.users` (or the existing row updates `last_sign_in_at`).

### 2.3 Rotation log

Append one line per rotation: `YYYY-MM-DD — operator — expiry YYYY-MM-DD — note`.

- 2026-04-19 — Edwin — expiry 2026-10-17 — initial setup for #38.

---

## 3. Full setup (for disaster recovery / fresh environment)

If every piece of Apple configuration had to be recreated from scratch — e.g., Apple Developer account moved, `.p8` lost and no backup — follow these steps. In normal operation you never need this section; rotation above is enough.

### 3.1 App ID

1. Apple Developer → Identifiers → `+` → **App IDs** → **App**.
2. Bundle ID: `com.izzyyum.app` (Explicit).
3. Capabilities: check **Sign in with Apple** (configure as primary if prompted) + **Push Notifications**.
4. Register.

### 3.2 Services ID

1. Apple Developer → Identifiers → `+` → **Services IDs**.
2. Description: `Izzy Yum Web — Sign in with Apple`
3. Identifier: `com.izzyyum.app.web-signin`
4. After registering, open it → check Sign in with Apple → **Configure**:
   - Primary App ID: `com.izzyyum.app`
   - Domains: `oogpnrrsmpklafdrvvkg.supabase.co`, `izzy-yum.com`
   - Return URLs: `https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/callback`, `https://izzy-yum.com/auth/callback`
5. Save.

### 3.3 Sign in with Apple Key

1. Apple Developer → Keys → `+`.
2. Name: `Izzy Yum — Sign in with Apple`
3. Check **Sign in with Apple** → Configure → Primary App ID `com.izzyyum.app` → Save.
4. Continue → Register.
5. **Download the `.p8` immediately** — Apple only allows one download. Store in 1Password as a secure file.
6. Record Key ID (10 chars) and Team ID (10 chars; top-right of Apple Developer portal next to your name).

### 3.4 Supabase provider

1. Dashboard → project `oogpnrrsmpklafdrvvkg` → Authentication → Providers → Apple → Enable.
2. Client IDs: `com.izzyyum.app.web-signin`
3. Secret Key (for OAuth): the JWT from `generate-apple-client-secret.js` (see §2.2).
4. Allow users without email: **OFF** (design invariant: email is the account anchor — see `ACCOUNT_AND_PAIRING.md` §3.4).
5. Save.

### 3.5 Repo config

Already in place:
- `web/supabase/config.toml` has `[auth.external.apple]` enabled with the Services ID and env-var-backed secret.
- `web/scripts/generate-apple-client-secret.js` produces the JWT.
- `web/src/app/auth/test/page.tsx` is the smoke test (deleted once web#41 signup UI ships).

---

## 4. Google (deferred)

Tracked in [web#50](https://github.com/izzy-yum/web/issues/50). When that lands, add a section here mirroring §1 + §3 for Google.
