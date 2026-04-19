# Authentication Flow Documentation
## Email & Phone OTP Verification

> [!WARNING]
> **SUPERSEDED — 2026-04-19.** This document describes the Twilio/SMS-era authentication flow that has been retired. The current design is in [`ACCOUNT_AND_PAIRING.md`](ACCOUNT_AND_PAIRING.md) — email-verified, passwordless (Apple / Google / email magic link), with QR-based device pairing for iOS and Android.
>
> Preserved for history. **Do not implement from this document.**

**Last Updated:** 2026-02-27
**Status:** Superseded by `ACCOUNT_AND_PAIRING.md` (2026-04-19). Original status: "Planned for Phase 2 Implementation."

---

## Overview

This document describes the complete authentication flow for Izzy Yum, including email and phone verification using 6-digit OTP (One-Time Password) codes.

**User Experience:**
- Single registration form with email, password, and phone
- After submission, two 6-digit code fields appear
- User enters code from email, then code from SMS
- Both verified → user is logged in and redirected

**Technologies:**
- **Supabase Auth**: Manages users, sessions, and OTP generation
- **Supabase Email**: Sends email codes (FREE)
- **Twilio SMS**: Sends phone codes via Supabase ($0.05 per SMS)

---

## Complete Registration Flow

### Phase 1: Initial Registration

```
┌─────────────────────────────────────────────────────────────┐
│                  Step 1: User Enters Info                    │
└─────────────────────────────────────────────────────────────┘

User fills out form on https://izzy-yum.com/register:
├─ Email: user@example.com
├─ Password: ********
└─ Phone: (415) 555-1234

User clicks "Create Account" button
```

```
┌─────────────────────────────────────────────────────────────┐
│              Step 2: App Validates & Formats                 │
└─────────────────────────────────────────────────────────────┘

Web App (Client-side):
├─ Validates email format: user@example.com ✓
├─ Validates password length: >= 6 chars ✓
├─ Validates phone format: (415) 555-1234 ✓
├─ Formats phone to E.164: +14155551234
└─ Proceeds to send request
```

---

### Phase 2: Email Verification

```
┌─────────────────────────────────────────────────────────────┐
│           Step 3: Request Email OTP from Supabase            │
└─────────────────────────────────────────────────────────────┘

Web App → Supabase API

Request:
  POST https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/otp

  Headers:
    apikey: sb_publishable_QtrddDowXS9MFvM5JGr5qg_VGG1IqhS
    Content-Type: application/json

  Body:
    {
      "email": "user@example.com",
      "create_user": true,
      "data": {
        "password": "hashed_password",
        "phone": "+14155551234"
      }
    }
```

```
┌─────────────────────────────────────────────────────────────┐
│         Step 4: Supabase Generates & Stores Email OTP        │
└─────────────────────────────────────────────────────────────┘

Supabase Backend Processing:
├─ Generates random 6-digit code: "123456"
├─ Stores in auth.otp_codes table:
│  └─ {
│      id: uuid,
│      user_id: uuid (or null if new user),
│      email: "user@example.com",
│      token_hash: hash("123456"),
│      type: "email",
│      expires_at: NOW() + 60 minutes,
│      created_at: NOW()
│    }
├─ Creates user record in auth.users (if new):
│  └─ {
│      id: uuid,
│      email: "user@example.com",
│      email_confirmed_at: null,  -- Not confirmed yet
│      phone: "+14155551234",
│      phone_confirmed_at: null,
│      encrypted_password: bcrypt_hash
│    }
└─ Prepares to send email
```

```
┌─────────────────────────────────────────────────────────────┐
│              Step 5: Supabase Sends Email OTP                │
└─────────────────────────────────────────────────────────────┘

Supabase Email Service → SMTP Server → User's Email

Email Details:
  From: noreply@oogpnrrsmpklafdrvvkg.supabase.co
  To: user@example.com
  Subject: Your Izzy Yum verification code

  Body:
    Hi there,

    Your verification code for Izzy Yum is:

    123456

    This code will expire in 60 minutes.

    If you didn't request this code, please ignore this email.

    Thanks,
    The Izzy Yum Team

Delivery time: Usually < 30 seconds
```

```
┌─────────────────────────────────────────────────────────────┐
│         Step 6: Supabase Returns Success to Web App          │
└─────────────────────────────────────────────────────────────┘

Supabase → Web App

Response:
  Status: 200 OK
  Body:
    {
      "message_id": "uuid-here"
    }

Note: Supabase does NOT return the actual code (security)
```

```
┌─────────────────────────────────────────────────────────────┐
│        Step 7: Web App Shows Verification Form               │
└─────────────────────────────────────────────────────────────┘

Web App UI Updates:
├─ Hides registration form (email/password/phone fields)
├─ Shows verification panel:
│  ├─ Email code: [______] (6-digit input)
│  ├─ Phone code: [______] (6-digit input)
│  └─ "Verify" button
├─ Shows message: "Check your email for verification code"
└─ Starts waiting for user input
```

```
┌─────────────────────────────────────────────────────────────┐
│           Step 8: User Checks Email & Enters Code            │
└─────────────────────────────────────────────────────────────┘

User Actions:
├─ Opens email client (Gmail, iPhone Mail, etc.)
├─ Finds email from Supabase (usually in inbox, sometimes spam)
├─ Reads code: 123456
├─ Returns to https://izzy-yum.com/register tab
├─ Clicks email code field
├─ Types: 1-2-3-4-5-6
└─ Code field now shows: 123456

Typical time: 30-60 seconds
```

```
┌─────────────────────────────────────────────────────────────┐
│        Step 9: App Verifies Email Code with Supabase         │
└─────────────────────────────────────────────────────────────┘

Web App → Supabase API

Request:
  POST https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/verify

  Headers:
    apikey: sb_publishable_QtrddDowXS9MFvM5JGr5qg_VGG1IqhS
    Content-Type: application/json

  Body:
    {
      "type": "email",
      "email": "user@example.com",
      "token": "123456"
    }
```

```
┌─────────────────────────────────────────────────────────────┐
│         Step 10: Supabase Validates Email Code               │
└─────────────────────────────────────────────────────────────┘

Supabase Backend:
├─ Queries auth.otp_codes table:
│  └─ WHERE email = "user@example.com"
│     AND type = "email"
│     AND expires_at > NOW()
│     AND used_at IS NULL
│
├─ Finds matching code record
├─ Compares hashes: hash("123456") === stored_token_hash
│
├─ If match found and valid:
│  ├─ Updates auth.users table:
│  │  └─ SET email_confirmed_at = NOW()
│  ├─ Updates auth.otp_codes table:
│  │  └─ SET used_at = NOW()
│  ├─ Creates session token (JWT)
│  └─ Returns success with session
│
└─ If invalid:
   └─ Returns error: "Invalid or expired OTP"
```

```
┌─────────────────────────────────────────────────────────────┐
│      Step 11: Web App Receives Email Verification Success    │
└─────────────────────────────────────────────────────────────┘

Supabase → Web App

Response:
  Status: 200 OK
  Body:
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "bearer",
      "expires_in": 3600,
      "refresh_token": "refresh_token_here",
      "user": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "email": "user@example.com",
        "email_confirmed_at": "2026-02-27T10:00:00.000Z",
        "phone": "+14155551234",
        "phone_confirmed_at": null,
        "created_at": "2026-02-27T10:00:00.000Z"
      }
    }

Web App Updates:
├─ Stores session token in browser
├─ Email code field shows: ✓ Verified
├─ Enables phone code field
└─ Shows message: "Email verified! Now verify phone..."
```

---

### Phase 3: Phone Verification

```
┌─────────────────────────────────────────────────────────────┐
│          Step 12: Request Phone OTP from Supabase            │
└─────────────────────────────────────────────────────────────┘

Web App → Supabase API

Request:
  POST https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/otp

  Headers:
    apikey: sb_publishable_QtrddDowXS9MFvM5JGr5qg_VGG1IqhS
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
    Content-Type: application/json

  Body:
    {
      "phone": "+14155551234",
      "create_user": false
    }

Note: User is already authenticated from email verification
```

```
┌─────────────────────────────────────────────────────────────┐
│        Step 13: Supabase Generates & Stores Phone OTP        │
└─────────────────────────────────────────────────────────────┘

Supabase Backend:
├─ Generates random 6-digit code: "654321"
├─ Stores in auth.otp_codes table:
│  └─ {
│      id: uuid,
│      user_id: "550e8400-e29b-41d4-a716-446655440000",
│      phone: "+14155551234",
│      token_hash: hash("654321"),
│      type: "sms",
│      expires_at: NOW() + 5 minutes,
│      created_at: NOW()
│    }
└─ Prepares to call Twilio
```

```
┌─────────────────────────────────────────────────────────────┐
│          Step 14: Supabase Calls Twilio API                  │
└─────────────────────────────────────────────────────────────┘

Supabase → Twilio API

Request:
  POST https://api.twilio.com/2010-04-01/Accounts/AC.../Messages.json

  Headers:
    Authorization: Basic [Base64(AccountSID:AuthToken)]
    Content-Type: application/x-www-form-urlencoded

  Body:
    From=+15551234567          // Your Twilio number
    To=+14155551234            // User's phone
    Body=Your Izzy Yum verification code is: 654321

Supabase uses the Twilio credentials you configured in settings
```

```
┌─────────────────────────────────────────────────────────────┐
│            Step 15: Twilio Processes & Sends SMS             │
└─────────────────────────────────────────────────────────────┘

Twilio Backend:
├─ Validates phone number format
├─ Checks account balance/credits
├─ Queues message for delivery
├─ Routes to appropriate mobile carrier (AT&T, Verizon, T-Mobile, etc.)
├─ Delivers SMS to carrier gateway
└─ Returns delivery status to Supabase

Twilio → Mobile Carrier → User's Phone

SMS Delivered (typically < 5 seconds)
```

```
┌─────────────────────────────────────────────────────────────┐
│        Step 16: Twilio Returns Status to Supabase            │
└─────────────────────────────────────────────────────────────┐

Twilio → Supabase

Response:
  Status: 201 Created
  Body:
    {
      "sid": "SM...",
      "status": "queued",
      "to": "+14155551234",
      "from": "+15551234567",
      "body": "Your Izzy Yum verification code is: 654321",
      "price": null,
      "price_unit": "USD"
    }

Delivery statuses:
- "queued" → Message accepted, being sent
- "sent" → Delivered to carrier
- "delivered" → Confirmed received by phone
- "failed" → Delivery failed (bad number, etc.)
```

```
┌─────────────────────────────────────────────────────────────┐
│         Step 17: Supabase Returns Success to Web App         │
└─────────────────────────────────────────────────────────────┘

Supabase → Web App

Response:
  Status: 200 OK
  Body:
    {
      "message_id": "uuid-here"
    }

Note: Does NOT return the actual code (security)
```

```
┌─────────────────────────────────────────────────────────────┐
│          Step 18: User Receives SMS on Phone                 │
└─────────────────────────────────────────────────────────────┘

User's Phone:
├─ Phone buzzes/beeps
├─ SMS notification appears
├─ User unlocks phone
└─ Reads message:

    From: +15551234567

    Your Izzy Yum verification code is: 654321

    (If trial account:)
    Sent from your Twilio trial account

Typical delivery time: 3-10 seconds
```

```
┌─────────────────────────────────────────────────────────────┐
│              Step 19: User Enters Phone Code                 │
└─────────────────────────────────────────────────────────────┘

User Actions:
├─ Looks at phone SMS: 654321
├─ Returns to web browser (izzy-yum.com/register)
├─ Clicks phone code field
├─ Types: 6-5-4-3-2-1
├─ Phone code field now shows: 654321
└─ Both fields filled (email ✓, phone ✓)

User clicks "Verify" button
```

```
┌─────────────────────────────────────────────────────────────┐
│        Step 20: App Verifies Phone Code with Supabase        │
└─────────────────────────────────────────────────────────────┘

Web App → Supabase API

Request:
  POST https://oogpnrrsmpklafdrvvkg.supabase.co/auth/v1/verify

  Headers:
    apikey: sb_publishable_QtrddDowXS9MFvM5JGr5qg_VGG1IqhS
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
    Content-Type: application/json

  Body:
    {
      "type": "sms",
      "phone": "+14155551234",
      "token": "654321"
    }
```

```
┌─────────────────────────────────────────────────────────────┐
│         Step 21: Supabase Validates Phone Code               │
└─────────────────────────────────────────────────────────────┘

Supabase Backend:
├─ Queries auth.otp_codes table:
│  └─ WHERE phone = "+14155551234"
│     AND type = "sms"
│     AND expires_at > NOW()
│     AND used_at IS NULL
│
├─ Finds matching code record
├─ Compares hashes: hash("654321") === stored_token_hash
│
├─ If match found and valid:
│  ├─ Updates auth.users table:
│  │  └─ SET phone_confirmed_at = NOW()
│  │     WHERE id = user_id
│  ├─ Updates auth.otp_codes table:
│  │  └─ SET used_at = NOW()
│  ├─ Updates session with phone verification
│  └─ Returns success
│
└─ If invalid:
   └─ Returns error: "Invalid or expired OTP"
```

```
┌─────────────────────────────────────────────────────────────┐
│     Step 22: Web App Receives Phone Verification Success     │
└─────────────────────────────────────────────────────────────┘

Supabase → Web App

Response:
  Status: 200 OK
  Body:
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "email": "user@example.com",
        "email_confirmed_at": "2026-02-27T10:00:00.000Z",
        "phone": "+14155551234",
        "phone_confirmed_at": "2026-02-27T10:05:00.000Z",
        "user_metadata": {
          "phone": "+14155551234"
        }
      }
    }

Web App Updates:
├─ Email code field: ✓ Verified
├─ Phone code field: ✓ Verified
├─ Shows success message: "Account verified!"
└─ User is now fully authenticated
```

---

### Phase 4: Completion

```
┌─────────────────────────────────────────────────────────────┐
│            Step 23: Redirect to Application                  │
└─────────────────────────────────────────────────────────────┘

Web App:
├─ Session stored in browser (httpOnly cookie)
├─ User marked as authenticated
├─ Redirects to: https://izzy-yum.com/
└─ User can now browse recipes, create shopping lists, etc.

Total time from registration to completion: ~2-3 minutes
```

---

## Database State Changes

### Before Verification
```sql
auth.users:
  id: 550e8400-e29b-41d4-a716-446655440000
  email: "user@example.com"
  email_confirmed_at: NULL                    ← Not verified
  phone: "+14155551234"
  phone_confirmed_at: NULL                    ← Not verified
  encrypted_password: "$2a$10$..."
  created_at: "2026-02-27T10:00:00Z"

auth.otp_codes (2 records):
  [1] {
    email: "user@example.com",
    token_hash: hash("123456"),
    type: "email",
    expires_at: "2026-02-27T11:00:00Z",
    used_at: NULL                             ← Not used yet
  }
  [2] {
    phone: "+14155551234",
    token_hash: hash("654321"),
    type: "sms",
    expires_at: "2026-02-27T10:05:00Z",
    used_at: NULL                             ← Not used yet
  }
```

### After Email Verification
```sql
auth.users:
  email_confirmed_at: "2026-02-27T10:01:00Z"  ← ✓ Verified
  phone_confirmed_at: NULL                     ← Still pending

auth.otp_codes:
  [1] {
    type: "email",
    used_at: "2026-02-27T10:01:00Z"           ← ✓ Used
  }
  [2] {
    type: "sms",
    used_at: NULL                              ← Still valid
  }
```

### After Phone Verification (Complete)
```sql
auth.users:
  email_confirmed_at: "2026-02-27T10:01:00Z"  ← ✓ Verified
  phone_confirmed_at: "2026-02-27T10:05:00Z"  ← ✓ Verified

auth.otp_codes:
  [1] {
    type: "email",
    used_at: "2026-02-27T10:01:00Z"           ← ✓ Used
  }
  [2] {
    type: "sms",
    used_at: "2026-02-27T10:05:00Z"           ← ✓ Used
  }

User is now fully verified and authenticated!
```

---

## API Call Summary

### 1. Send Email OTP
```typescript
const { data, error } = await supabase.auth.signInWithOtp({
  email: 'user@example.com',
  options: {
    shouldCreateUser: true,
    data: {
      phone: '+14155551234'
    }
  }
})
```

### 2. Verify Email OTP
```typescript
const { data, error } = await supabase.auth.verifyOtp({
  email: 'user@example.com',
  token: '123456',
  type: 'email'
})
```

### 3. Send Phone OTP
```typescript
const { data, error } = await supabase.auth.signInWithOtp({
  phone: '+14155551234',
  options: {
    shouldCreateUser: false
  }
})
```

### 4. Verify Phone OTP
```typescript
const { data, error } = await supabase.auth.verifyOtp({
  phone: '+14155551234',
  token: '654321',
  type: 'sms'
})
```

---

## Security Features

### Code Expiration
- **Email codes**: Expire after 60 minutes
- **Phone codes**: Expire after 5 minutes (shorter for security)
- Expired codes automatically rejected by Supabase

### One-Time Use
- Each code can only be used once
- After verification, `used_at` timestamp is set
- Prevents replay attacks

### Rate Limiting
- Supabase limits OTP requests:
  - **Email**: Max 5 requests per hour per email
  - **Phone**: Max 5 requests per hour per phone
- Prevents spam and abuse
- Returns error if limit exceeded

### Secure Storage
- Codes stored as hashes (not plaintext)
- Only Supabase can verify codes
- Codes never returned to client

### Session Management
- After verification, JWT session token created
- Token expires after 1 hour (configurable)
- Refresh token allows re-authentication
- Stored as httpOnly cookie (prevents XSS attacks)

---

## Error Handling

### Common Errors & User Messages

| Error | Cause | User Message |
|-------|-------|--------------|
| `Invalid OTP` | Wrong code entered | "Invalid verification code. Please try again." |
| `Expired OTP` | Code older than expiry time | "Code expired. Click 'Resend code' to get a new one." |
| `Rate limit exceeded` | Too many requests | "Too many attempts. Please try again in 1 hour." |
| `Invalid phone number` | Bad format or non-existent | "Invalid phone number. Please check and try again." |
| `SMS delivery failed` | Twilio couldn't deliver | "Couldn't send SMS. Please verify your phone number." |
| `User already exists` | Email already registered | "Email already registered. Try logging in instead." |

### Retry Logic

**Email Code:**
- User can request new code after 60 seconds
- Old code remains valid until expiry
- Shows "Resend email code" button after 60 seconds

**Phone Code:**
- User can request new code after 30 seconds
- Old code remains valid until expiry
- Shows "Resend SMS code" button after 30 seconds

---

## Network Latency

### Expected Response Times

| Step | Typical Time | Max Time |
|------|--------------|----------|
| Request email OTP | 200-500ms | 2 seconds |
| Email delivery | 10-30 seconds | 2 minutes |
| Verify email OTP | 100-300ms | 1 second |
| Request phone OTP | 200-500ms | 2 seconds |
| SMS delivery | 3-10 seconds | 30 seconds |
| Verify phone OTP | 100-300ms | 1 second |
| **Total flow** | **2-3 minutes** | **5 minutes** |

### User Wait Times

**Email verification:**
- Code arrives: ~30 seconds
- User reads and enters: ~30 seconds
- **Total: ~1 minute**

**Phone verification:**
- Code arrives: ~5-10 seconds
- User reads and enters: ~20 seconds
- **Total: ~30 seconds**

**Complete registration: ~2-3 minutes**

---

## Data Flow Diagram

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│   USER   │         │  WEB APP │         │ SUPABASE │         │ EMAIL/SMS│
└─────┬────┘         └─────┬────┘         └─────┬────┘         └─────┬────┘
      │                    │                    │                    │
      │ 1. Enter info      │                    │                    │
      ├───────────────────>│                    │                    │
      │                    │ 2. Request email   │                    │
      │                    │       OTP          │                    │
      │                    ├───────────────────>│                    │
      │                    │                    │ 3. Generate code   │
      │                    │                    │    Store: 123456   │
      │                    │                    │ 4. Send email      │
      │                    │                    ├───────────────────>│
      │                    │ 5. Success         │                    │
      │                    │<───────────────────┤                    │
      │ 6. Show verify UI  │                    │                    │
      │<───────────────────┤                    │                    │
      │                    │                    │ 7. Email arrives   │
      │                    │                    │   Code: 123456     │
      │<───────────────────┼────────────────────┼────────────────────┤
      │ 8. Enter: 123456   │                    │                    │
      ├───────────────────>│ 9. Verify email    │                    │
      │                    ├───────────────────>│ 10. Check code     │
      │                    │                    │     Valid ✓        │
      │                    │11. Email verified  │                    │
      │                    │   + session token  │                    │
      │12. Email ✓         │<───────────────────┤                    │
      │<───────────────────┤                    │                    │
      │                    │                    │                    │
      │13. (auto) Request  │                    │                    │
      │    phone OTP       │                    │                    │
      │                    ├───────────────────>│ 14. Generate code  │
      │                    │                    │     Store: 654321  │
      │                    │                    │ 15. Call Twilio    │
      │                    │                    ├───────────────────>│
      │                    │                    │                    │ 16. Send SMS
      │                    │                    │                    │ ┌────────┐
      │                    │                    │                    │ │ Twilio │
      │                    │                    │                    │ │ Routes │
      │                    │                    │                    │ │   to   │
      │                    │                    │                    │ │Carrier │
      │                    │                    │                    │ └────┬───┘
      │                    │                    │ 17. SMS queued     │      │
      │                    │18. Success         │<───────────────────┤      │
      │                    │<───────────────────┤                    │      │
      │19. "Code sent"     │                    │                    │      │
      │<───────────────────┤                    │                    │      │
      │                    │                    │                    │<─────┘
      │20. SMS arrives     │                    │                    │   18. Deliver
      │    Code: 654321    │                    │                    │
      │<───────────────────┼────────────────────┼────────────────────┤
      │21. Enter: 654321   │                    │                    │
      ├───────────────────>│ 22. Verify phone   │                    │
      │                    ├───────────────────>│ 23. Check code     │
      │                    │                    │     Valid ✓        │
      │                    │24. Phone verified  │                    │
      │                    │   + updated token  │                    │
      │25. Both verified ✓ │<───────────────────┤                    │
      │    Redirect home   │                    │                    │
      │<───────────────────┤                    │                    │
      │                    │                    │                    │
      ▼                    ▼                    ▼                    ▼
```

---

## Session & Token Management

### Session Token (JWT)

**After email verification:**
```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "email_confirmed_at": "2026-02-27T10:01:00Z",
  "phone": "+14155551234",
  "phone_confirmed_at": null,
  "role": "authenticated",
  "iat": 1709028060,
  "exp": 1709031660
}
```

**After phone verification:**
```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "email_confirmed_at": "2026-02-27T10:01:00Z",
  "phone": "+14155551234",
  "phone_confirmed_at": "2026-02-27T10:05:00Z",  ← Updated
  "role": "authenticated",
  "iat": 1709028300,
  "exp": 1709031900
}
```

### Session Storage

**Browser (Client):**
```
localStorage:
  supabase.auth.token: {
    access_token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    refresh_token: "refresh_token_here",
    expires_at: 1709031900,
    user: { ... }
  }

Cookies (httpOnly):
  sb-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  sb-refresh-token: refresh_token_here
```

---

## Cost Tracking

### Email OTP
```
Request: FREE
Email sent: FREE
Verification: FREE
Total: $0
```

### Phone OTP
```
Request: FREE
Twilio API call: $0.05
SMS delivery: Included in $0.05
Verification: FREE
Total: $0.05 per registration
```

### Example Monthly Costs

| Signups/Month | Email Cost | Phone Cost | Total |
|---------------|------------|------------|-------|
| 10 | $0 | $0.50 | $0.50 |
| 50 | $0 | $2.50 | $2.50 |
| 100 | $0 | $5.00 | $5.00 |
| 500 | $0 | $25.00 | $25.00 |
| 1,000 | $0 | $50.00 | $50.00 |

**Trial credit:** $15.50 = ~310 free registrations

---

## Implementation Notes for Phase 2

### Code Changes Required

**Files to modify:**
1. `src/app/register/page.tsx` - Main registration component
2. `src/lib/supabase.ts` - Add OTP helper functions (optional)

**Changes:**
1. Replace `signUp()` with `signInWithOtp()` for email
2. Add phone OTP request after email verification
3. Replace hardcoded "1234" checks with `verifyOtp()` calls
4. Add error handling for all OTP operations
5. Add "Resend code" functionality
6. Format phone numbers to E.164 before sending

### Testing Strategy

**Local testing:**
1. Test with your verified phone number
2. Verify email codes arrive
3. Verify SMS codes arrive
4. Test invalid codes (should reject)
5. Test expired codes (wait 60+ min for email, 5+ min for phone)

**Production testing:**
1. Deploy to Vercel
2. Test full flow on https://izzy-yum.com
3. Verify Supabase logs show successful OTP sends
4. Check Twilio logs show successful SMS delivery

---

## Monitoring & Logs

### Supabase Logs
https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/logs/auth-logs

**What to watch:**
- OTP requests (should see email + phone requests)
- Verification attempts (successful and failed)
- Rate limit violations
- Error messages

### Twilio Logs
https://console.twilio.com/us1/monitor/logs/sms

**What to watch:**
- SMS delivery status (queued → sent → delivered)
- Failed deliveries (bad numbers, carrier issues)
- Cost per message
- Credit balance

---

*This flow will be implemented in Phase 2 code integration.*
