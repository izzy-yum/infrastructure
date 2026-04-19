# Authentication Quick Reference
## Email & Phone OTP - Visual Flow

> [!WARNING]
> **SUPERSEDED — 2026-04-19.** This document describes the Twilio/SMS-era authentication flow that has been retired. The current design is in [`ACCOUNT_AND_PAIRING.md`](ACCOUNT_AND_PAIRING.md) — email-verified, passwordless (Apple / Google / email magic link), with QR-based device pairing for iOS and Android.
>
> Preserved for history. **Do not implement from this document.**

---

## 🎯 User Journey (2-3 minutes)

```
┌─────────────────────────────────────────────────┐
│  1. USER OPENS REGISTRATION PAGE                │
│     https://izzy-yum.com/register               │
└─────────────────────────────────────────────────┘

Form Fields:
  ┌─────────────────────────────────┐
  │ Email:    [                   ] │
  │ Password: [                   ] │
  │ Phone:    [                   ] │
  │                                 │
  │        [ Create Account ]       │
  └─────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────┐
│  2. USER FILLS FORM & CLICKS "CREATE ACCOUNT"   │
└─────────────────────────────────────────────────┘

  Email:    user@example.com
  Password: ••••••••
  Phone:    (415) 555-1234

  [ Create Account ] ← Click
```

```
┌─────────────────────────────────────────────────┐
│  3. APP SENDS EMAIL CODE                        │
└─────────────────────────────────────────────────┘

  Web App → Supabase: "Send email OTP"

  Supabase generates: 123456
  Supabase → Email: Sends code

  ⏱️  30 seconds later...

  📧 User's Inbox:
  ┌─────────────────────────────────┐
  │ From: Supabase                  │
  │ Subject: Your verification code │
  │                                 │
  │ Your code is: 123456            │
  └─────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────┐
│  4. VERIFICATION FORM APPEARS                   │
└─────────────────────────────────────────────────┘

  ✉️  Check your email for verification code

  ┌─────────────────────────────────┐
  │ Email Code:  [______]           │ ← User enters: 123456
  │ Phone Code:  [______] (disabled)│
  │                                 │
  │          [ Verify ]             │
  └─────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────┐
│  5. USER ENTERS EMAIL CODE                      │
└─────────────────────────────────────────────────┘

  User types: 1-2-3-4-5-6

  ┌─────────────────────────────────┐
  │ Email Code:  [123456]           │
  │ Phone Code:  [______] (disabled)│
  │                                 │
  │          [ Verify ]             │ ← Click
  └─────────────────────────────────┘

  Web App → Supabase: "Verify email OTP: 123456"
  Supabase checks code → Valid ✓
  Supabase → Web App: "Email verified + session token"
```

```
┌─────────────────────────────────────────────────┐
│  6. EMAIL VERIFIED - PHONE CODE SENT            │
└─────────────────────────────────────────────────┘

  ✅ Email verified!
  📱 Sending code to your phone...

  ┌─────────────────────────────────┐
  │ Email Code:  [123456] ✓         │
  │ Phone Code:  [______]           │ ← Now enabled
  │                                 │
  │          [ Verify ]             │
  └─────────────────────────────────┘

  Behind the scenes:
  Web App → Supabase: "Send phone OTP"
  Supabase generates: 654321
  Supabase → Twilio: "Send SMS"
  Twilio → Carrier → User's phone
```

```
┌─────────────────────────────────────────────────┐
│  7. USER RECEIVES SMS                           │
└─────────────────────────────────────────────────┘

  ⏱️  5-10 seconds later...

  📱 User's Phone Buzzes:
  ┌─────────────────────────────────┐
  │ Message from +15551234567       │
  │                                 │
  │ Your Izzy Yum verification      │
  │ code is: 654321                 │
  │                                 │
  │ (Sent from your Twilio trial   │
  │  account)                       │
  └─────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────┐
│  8. USER ENTERS PHONE CODE                      │
└─────────────────────────────────────────────────┘

  User types: 6-5-4-3-2-1

  ┌─────────────────────────────────┐
  │ Email Code:  [123456] ✓         │
  │ Phone Code:  [654321]           │
  │                                 │
  │          [ Verify ]             │ ← Click
  └─────────────────────────────────┘

  Web App → Supabase: "Verify phone OTP: 654321"
  Supabase checks code → Valid ✓
  Supabase → Web App: "Phone verified"
```

```
┌─────────────────────────────────────────────────┐
│  9. COMPLETE - REDIRECT TO HOME                 │
└─────────────────────────────────────────────────┘

  ✅ Account verified!

  ┌─────────────────────────────────┐
  │ Email Code:  [123456] ✓         │
  │ Phone Code:  [654321] ✓         │
  │                                 │
  │   🎉 Success! Redirecting...    │
  └─────────────────────────────────┘

  User is logged in
  Redirects to: https://izzy-yum.com/
  Can now browse recipes, create shopping lists, etc.
```

---

## 📊 Technical Flow Summary

### Email Verification

| # | Action | From | To | Data |
|---|--------|------|-----|------|
| 1 | Request OTP | Web App | Supabase | `{email, create_user: true}` |
| 2 | Generate code | Supabase | - | Code: `123456` (stored) |
| 3 | Send email | Supabase | Email Server | Email with code |
| 4 | Deliver email | Email Server | User Inbox | Email arrives (~30s) |
| 5 | Enter code | User | Web App | User types: `123456` |
| 6 | Verify code | Web App | Supabase | `{type: email, token: "123456"}` |
| 7 | Validate | Supabase | - | Check hash, expiry, usage |
| 8 | Return session | Supabase | Web App | `{access_token, user}` |

**Total time:** ~1-2 minutes

---

### Phone Verification

| # | Action | From | To | Data |
|---|--------|------|-----|------|
| 1 | Request OTP | Web App | Supabase | `{phone: "+14155551234"}` |
| 2 | Generate code | Supabase | - | Code: `654321` (stored) |
| 3 | Call Twilio | Supabase | Twilio | SMS request with code |
| 4 | Send SMS | Twilio | Carrier | Route to mobile network |
| 5 | Deliver SMS | Carrier | User Phone | SMS arrives (~5-10s) |
| 6 | Enter code | User | Web App | User types: `654321` |
| 7 | Verify code | Web App | Supabase | `{type: sms, token: "654321"}` |
| 8 | Validate | Supabase | - | Check hash, expiry, usage |
| 9 | Return success | Supabase | Web App | `{user}` with phone verified |

**Total time:** ~30-60 seconds

---

## 🔐 Security Model

### Code Generation
```
Supabase generates codes using:
- Cryptographically secure random number generator
- 6 digits = 1,000,000 possible combinations
- Short expiry times limit brute force attacks
```

### Storage
```
Database stores:
- Hash of code (not plaintext)
- Email or phone associated with code
- Expiry timestamp
- Usage timestamp

Example:
  token_hash: sha256("123456") = "8d969eef6..."
  NOT stored: "123456"
```

### Validation
```
When user enters code:
1. Supabase hashes entered code
2. Compares hash to stored hash
3. Checks expiry time
4. Checks if already used
5. Only then marks as verified
```

### Rate Limiting
```
Supabase automatically limits:
- Max 5 OTP requests per hour per email
- Max 5 OTP requests per hour per phone
- Prevents spam and abuse
```

---

## 🔄 Alternative Flows

### User Never Receives Email

```
User: "I didn't get the email"

Option 1: Resend Code (after 60 seconds)
  Web App: Shows "Resend email code" button
  User clicks → Sends new OTP
  New code: 789012
  Old code: 123456 (still valid until expiry)

Option 2: Check Spam
  User checks spam/junk folder
  Finds original email with: 123456
  Enters code → Works fine
```

### User Never Receives SMS

```
User: "I didn't get the text"

Common causes:
- Phone number format wrong
- Phone not verified in Twilio (trial mode)
- Carrier blocked message
- Phone turned off

Solution:
  Web App: Shows "Resend SMS code" button (after 30 seconds)
  User clicks → Sends new OTP via Twilio
  Check Twilio logs for delivery status
```

### User Enters Wrong Code

```
User enters: 999999 (incorrect)

Web App → Supabase: Verify "999999"
Supabase: Code doesn't match
Supabase → Web App: Error "Invalid OTP"
Web App → User: "Invalid code. Please try again."

User can re-enter correct code (codes don't get "locked out")
```

### Code Expires

```
Email code expires: After 60 minutes
Phone code expires: After 5 minutes

User enters expired code:

Supabase checks: expires_at < NOW()
Returns: "Expired OTP"
Web App shows: "Code expired. Click 'Resend' to get a new one."

User clicks "Resend" → Gets fresh code
```

---

## 📱 Phone Number Format Handling

### What User Enters
```
Common formats users might type:
- (415) 555-1234
- 415-555-1234
- 415.555.1234
- 4155551234
- +1 415 555 1234
```

### What App Converts To (E.164)
```
All formats → +14155551234

Conversion logic:
1. Remove all non-digits: 4155551234
2. Add country code: +1 (if US)
3. Result: +14155551234

This is what gets sent to Supabase/Twilio
```

### International Numbers
```
User enters: +44 20 7946 0958 (UK)
App formats: +442079460958

User enters: +81 3-1234-5678 (Japan)
App formats: +81312345678
```

---

## 🧪 Testing Scenarios

### Happy Path (Everything Works)
```
✓ User enters valid email
✓ Email code arrives in 30s
✓ User enters correct email code
✓ Email verified
✓ Phone code sent
✓ SMS arrives in 5s
✓ User enters correct phone code
✓ Phone verified
✓ User redirected home
✓ Total time: ~2 minutes
```

### Edge Cases

**1. Email in Spam Folder**
```
- Email arrives but user doesn't see it
- User waits 2-3 minutes
- User clicks "Resend code"
- Checks spam folder
- Finds BOTH emails
- Either code works (both valid)
```

**2. SMS Delayed**
```
- Carrier delay (rare but happens)
- User waits 30 seconds
- No SMS arrives
- User clicks "Resend SMS"
- New code sent: 111222
- Original code arrives: 654321
- Either code works (both valid)
```

**3. User Typo in Code**
```
Actual code: 123456
User enters: 123465 (typo in last digit)

Result: "Invalid code"
User re-enters: 123456
Result: Success ✓
```

**4. User Closes Browser**
```
During verification process:
- User closes browser tab
- Codes are still valid (stored in Supabase)
- User returns to site
- Must start registration again
- New codes will be generated
```

---

## 💰 Cost per Registration

### Breakdown
```
Email OTP:
  Supabase email service: $0
  Email delivery: $0
  Verification: $0
  Subtotal: $0

Phone OTP:
  Supabase request: $0
  Twilio SMS: $0.05
  SMS delivery: $0 (included)
  Verification: $0
  Subtotal: $0.05

Total per registration: $0.05
```

### Trial Credit Usage
```
Twilio trial credit: $15.50
Cost per SMS: $0.05
Free registrations: 310

After 310 registrations:
- Need to add credit to Twilio
- Minimum: $20 (gives 400 more SMS)
```

---

## 🚨 Common Issues & Solutions

### Issue: Email Not Arriving

**Check:**
- [ ] Spam/junk folder
- [ ] Email address correct
- [ ] Supabase email provider enabled

**Solution:**
```
1. Wait 2 minutes (sometimes delayed)
2. Check spam folder
3. Click "Resend email code"
4. Check Supabase logs for send confirmation
```

---

### Issue: SMS Not Arriving

**Check:**
- [ ] Phone number format: +14155551234
- [ ] Phone verified in Twilio (trial mode)
- [ ] Phone is mobile (not landline)
- [ ] Carrier not blocking messages

**Solution:**
```
Trial Mode:
1. Verify your phone number in Twilio console
2. Use only verified numbers for testing

Production Mode:
1. Add $20 credit to Twilio
2. SMS will work for all US/Canada numbers
```

---

### Issue: "Invalid OTP" Error

**Causes:**
- Typo in code entry
- Code expired
- Code already used
- Wrong code for wrong field (email code in phone field)

**Solution:**
```
1. Check code was entered correctly
2. Check code hasn't expired
3. Click "Resend code" if needed
4. Try new code
```

---

### Issue: "Rate Limit Exceeded"

**Cause:**
- Too many OTP requests in 1 hour (>5)
- Prevents spam

**Solution:**
```
1. Wait 1 hour
2. Try again
3. Contact support if legitimate user
```

---

## 📊 Timeline Comparison

### Current (Hardcoded "1234")
```
User action                    Time        Total
─────────────────────────────────────────────────
Enter email/password/phone     30s         30s
Click "Create Account"         1s          31s
Enter "1234" for email         2s          33s
Enter "1234" for phone         2s          35s
Click "Verify"                 1s          36s
Redirect home                  1s          37s

Total: ~37 seconds
```

### After Implementation (Real OTP)
```
User action                    Time        Total
─────────────────────────────────────────────────
Enter email/password/phone     30s         30s
Click "Create Account"         1s          31s
Wait for email                 30s         61s
Check email, copy code         20s         81s
Enter email code (123456)      5s          86s
Click verify → phone sent      2s          88s
Receive SMS                    10s         98s
Enter phone code (654321)      5s          103s
Click "Verify"                 1s          104s
Redirect home                  1s          105s

Total: ~2 minutes
```

**Trade-off:** +90 seconds total time, but REAL security

---

## 🎨 UI States

### State 1: Registration Form
```
┌─────────────────────────────────────────┐
│  Create your account                    │
│                                         │
│  Email:    [user@example.com        ]  │
│  Password: [••••••••                ]  │
│  Phone:    [(415) 555-1234          ]  │
│                                         │
│  [ Create Account ]                     │
│                                         │
│  Already have an account? Log in        │
└─────────────────────────────────────────┘
```

### State 2: Verification Form (Waiting for Email)
```
┌─────────────────────────────────────────┐
│  Verify your account                    │
│                                         │
│  ✉️  Check your email                   │
│  We sent a code to:                     │
│  user@example.com                       │
│                                         │
│  Email Code:  [______]                  │
│                                         │
│  📱 Phone verification pending          │
│  Phone Code:  [______] (disabled)       │
│                                         │
│  [ Verify ]                             │
│                                         │
│  Didn't receive it? Resend code (60s)   │
└─────────────────────────────────────────┘
```

### State 3: Email Verified, Waiting for Phone
```
┌─────────────────────────────────────────┐
│  Verify your account                    │
│                                         │
│  ✅ Email verified!                     │
│  Email Code:  [123456] ✓                │
│                                         │
│  📱 Check your phone                    │
│  We sent a code to:                     │
│  (415) 555-1234                         │
│                                         │
│  Phone Code:  [______]                  │
│                                         │
│  [ Verify ]                             │
│                                         │
│  Didn't receive it? Resend code (30s)   │
└─────────────────────────────────────────┘
```

### State 4: Both Verified
```
┌─────────────────────────────────────────┐
│  Verify your account                    │
│                                         │
│  ✅ Email verified!                     │
│  Email Code:  [123456] ✓                │
│                                         │
│  ✅ Phone verified!                     │
│  Phone Code:  [654321] ✓                │
│                                         │
│  🎉 Account created successfully!       │
│                                         │
│  Redirecting to home...                 │
└─────────────────────────────────────────┘

(Redirects after 2 seconds)
```

---

## 🔑 Key Differences from Current Implementation

### Current (Hardcoded "1234")

```typescript
// In handleVerification:
if (emailCode !== '1234' || phoneCode !== '1234') {
  setError('Invalid verification codes. Enter 1234 for both codes.')
  return
}
// No actual verification, just string comparison
```

**Problems:**
- ❌ No security (anyone can use "1234")
- ❌ No email actually sent
- ❌ No SMS actually sent
- ❌ Email/phone not verified in database

---

### After Phase 2 (Real OTP)

```typescript
// Step 1: Send email OTP
const { data: emailData } = await supabase.auth.signInWithOtp({
  email: email,
  options: { shouldCreateUser: true }
})

// Step 2: Verify email code
const { data: emailVerify } = await supabase.auth.verifyOtp({
  email: email,
  token: emailCode,
  type: 'email'
})

// Step 3: Send phone OTP
const { data: phoneData } = await supabase.auth.signInWithOtp({
  phone: phone
})

// Step 4: Verify phone code
const { data: phoneVerify } = await supabase.auth.verifyOtp({
  phone: phone,
  token: phoneCode,
  type: 'sms'
})
```

**Benefits:**
- ✅ Real security (random 6-digit codes)
- ✅ Email actually sent and verified
- ✅ SMS actually sent and verified
- ✅ Email/phone confirmed in database
- ✅ Proper session management
- ✅ Same UI/UX for user

---

## 📈 Scalability

### Current Limits (Free Tiers)

**Supabase Free Tier:**
- Email OTP: Unlimited (no cost)
- Phone OTP: Unlimited requests (Twilio charges apply)
- Database storage: 500MB
- No monthly OTP limits

**Twilio Trial:**
- SMS: $15.50 credit (~310 messages)
- Only to verified numbers
- Includes trial message

**Twilio Paid (after trial):**
- SMS: $0.05 each, pay-as-you-go
- No monthly minimum
- No verification requirement
- No trial message

### Expected Usage

**Alpha (10 users):**
- Email OTP: 10 × $0 = $0
- Phone OTP: 10 × $0.05 = $0.50
- Total: $0.50 (covered by trial)

**Beta (100 users):**
- Email OTP: 100 × $0 = $0
- Phone OTP: 100 × $0.05 = $5
- Total: $5/month

**Production (1,000 signups/month):**
- Email OTP: 1,000 × $0 = $0
- Phone OTP: 1,000 × $0.05 = $50
- Total: $50/month

---

## 🎯 Success Criteria

### User Experience Metrics
- [ ] Registration completion rate: >80%
- [ ] Email code entry time: <60 seconds
- [ ] Phone code entry time: <30 seconds
- [ ] Total verification time: <3 minutes
- [ ] Error rate: <5%

### Technical Metrics
- [ ] Email delivery rate: >95%
- [ ] SMS delivery rate: >98%
- [ ] Code validation success: >90%
- [ ] API response time: <500ms
- [ ] Zero security vulnerabilities

---

## 📝 Phase 2 Implementation Checklist

When ready to implement:

- [ ] Update `src/app/register/page.tsx`
  - [ ] Replace hardcoded signup with `signInWithOtp()`
  - [ ] Add email OTP verification
  - [ ] Add phone OTP request
  - [ ] Add phone OTP verification
  - [ ] Remove hardcoded "1234" checks
  - [ ] Add error handling for all scenarios
  - [ ] Add "Resend code" buttons
  - [ ] Add phone number formatting (E.164)

- [ ] Test locally
  - [ ] Email OTP flow works
  - [ ] Phone OTP flow works (with verified number)
  - [ ] Error handling works
  - [ ] Invalid codes rejected
  - [ ] Resend functionality works

- [ ] Deploy to production
  - [ ] Build succeeds
  - [ ] Test on https://izzy-yum.com
  - [ ] Register with real email/phone
  - [ ] Verify complete flow works
  - [ ] Check Twilio logs for SMS delivery

- [ ] Update documentation
  - [ ] Update Issue #23 with completion
  - [ ] Document any issues encountered
  - [ ] Add troubleshooting notes

---

*Ready for Phase 2 implementation when Twilio and Supabase setup is complete.*
