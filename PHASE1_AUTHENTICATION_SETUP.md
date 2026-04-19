# Phase 1: Authentication Setup Checklist
## Twilio SMS & Supabase OTP Configuration

> [!WARNING]
> **SUPERSEDED — 2026-04-19.** This checklist was for the Twilio/SMS-era auth setup, which has been retired. The current design is in [`ACCOUNT_AND_PAIRING.md`](ACCOUNT_AND_PAIRING.md) — email-verified, passwordless (Apple / Google / email magic link), with QR-based device pairing. A new setup checklist will be written once the new design is implementation-ready.
>
> Preserved for history. **Do not follow the steps below.**

**Estimated Time:** 20-30 minutes
**Cost:** $0 (using free tiers and trial credits)

---

## 📱 Part A: Twilio Setup (10 minutes)

### Step 1: Create Twilio Account

- [X] Go to https://www.twilio.com/try-twilio
- [X] Click **"Start for free"** or **"Sign up"**
- [X] Fill in registration form:
  - [ ] First name
  - [ ] Last name
  - [ ] Email address
  - [ ] Password (strong password recommended)
- [X] Click **"Sign Up"**
- [X] Check your email and verify your email address
- [X] Complete phone verification (Twilio will call or text you)
- [X] Answer the onboarding questions (select your use case)

### Step 2: Get a Trial Phone Number

- [X] After login, you'll see the Twilio Console
- [X] Look for "Get a Twilio phone number" prompt
- [X] Click **"Get a trial number"**
- [X] Review the suggested number (should be a US number like +1 555-123-4567)
- [X] Click **"Choose this number"** (or click "Search for a different number" if you want to pick)
- [X] Confirm the number is now in your account

### Step 3: Collect Twilio Credentials

**Go to:** https://console.twilio.com/

- [X] Find the **Account Info** section on the dashboard
- [X] Copy and save the following (you'll need these later):

```
Account SID: AC________________________________
(Starts with "AC" followed by 32 characters)

Auth Token: ________________________________
(Click "Show" to reveal, then copy)

Twilio Phone Number: +1__________
(Format: +15551234567)
```

- [X] Save these credentials in a secure note/password manager
- [X] **IMPORTANT:** Keep Auth Token secret - treat it like a password

### Step 4: Verify Your Test Phone Number

**Why:** Trial accounts can only send SMS to verified numbers.

- [ ] Go to https://console.twilio.com/us1/develop/phone-numbers/manage/verified
- [ ] Click **"Add a new number"** (red + button)
- [ ] Enter YOUR personal phone number (for testing)
  - [ ] Use format: +1XXXXXXXXXX (include country code)
  - [ ] Example: +14155551234
- [ ] Click **"Send verification code"**
- [ ] Enter the code you receive via SMS
- [ ] Click **"Verify"**
- [ ] ✅ Your number should now show as "Verified"

**Optional:** Verify additional test numbers if you want others to test:
- [ ] Repeat above steps for additional numbers

### Step 5: Understand Trial Limitations

**Review these important trial account details:**

- [ ] ⚠️ Trial accounts can **only send SMS to verified numbers**
- [ ] ⚠️ SMS will include: "Sent from your Twilio trial account"
- [ ] ✅ You have **$15.50 in free credit** (~310 SMS messages)
- [ ] ✅ To remove limitations: Add $20+ credit to your account
- [ ] 💡 You can upgrade later - trial is fine for testing

---

## 📧 Part B: Supabase Email OTP Setup (5 minutes)

### Step 1: Enable Email Provider

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/providers
- [ ] Scroll to the **"Email"** section
- [ ] Toggle **ON** (should turn green): "Enable Email provider"
- [ ] Under "Email OTP" settings:
  - [ ] Check ✅ **"Enable Email OTP"**
- [ ] Click **"Save"** at the bottom

### Step 2: Configure Email Template (Optional but Recommended)

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/templates
- [ ] Click on **"Magic Link"** template (or "Confirm Signup" template)
- [ ] Review the default template
- [ ] **Optional:** Customize the email template:

**Suggested template:**
```
Subject: Your Izzy Yum verification code

Hi there,

Your verification code for Izzy Yum is:

{{ .Token }}

This code will expire in 60 minutes.

If you didn't request this code, please ignore this email.

Thanks,
The Izzy Yum Team
```

- [X] Click **"Save"** if you made changes

### Step 3: Configure Site URLs

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/url-configuration
- [ ] Set **Site URL** to: `https://izzy-yum.com`
- [ ] Under **Redirect URLs**, verify or add these URLs:
  - [ ] `https://izzy-yum.com`
  - [ ] `https://izzy-yum.com/verify-email`
  - [ ] `http://localhost:9001` (for local development)
- [X] Click **"Save"**

### Step 4: Verify Email Settings

- [ ] Go back to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/providers
- [ ] Confirm **Email** provider shows as "Enabled" (green toggle)
- [ ] Confirm **Email OTP** is checked
- [X] ✅ Email OTP is now configured

---

## 📱 Part C: Supabase Phone OTP Setup (5 minutes)

### Step 1: Add Twilio Credentials to Supabase

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/settings/auth
- [ ] Scroll down to **"Phone Auth"** section
- [ ] Under "SMS Provider", select: **Twilio**
- [ ] Fill in the credentials from Part A, Step 3:

  - [ ] **Twilio Account SID**: Paste your Account SID (starts with AC)
  - [ ] **Twilio Auth Token**: Paste your Auth Token
  - [ ] **Twilio Messaging Service SID**: Leave blank
  - [ ] **Twilio Verify Service SID**: Leave blank

- [ ] Click **"Save"**
- [X] Wait for "Settings saved successfully" confirmation

### Step 2: Enable Phone Provider

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/providers
- [ ] Scroll to the **"Phone"** section
- [ ] Toggle **ON** (should turn green): "Enable Phone provider"
- [ ] Under "Phone OTP" settings:
  - [ ] Check ✅ **"Enable Phone OTP"**
- [X] Click **"Save"** at the bottom

### Step 3: Configure Phone Settings (Optional)

- [ ] Still in the Phone section, review these settings:
  - [ ] **OTP Expiry**: Default is 60 seconds (can change to 60-300 seconds)
  - [ ] **OTP Length**: Default is 6 digits (recommended)
- [ ] Leave defaults or adjust as needed
- [X] Click **"Save"** if you made changes

### Step 4: Verify Phone Settings

- [X] Confirm **Phone** provider shows as "Enabled" (green toggle)
- [X] Confirm **Phone OTP** is checked
- [X] ✅ Phone OTP is now configured

---

## ✅ Final Verification Checklist

Before proceeding to Phase 2, verify all items are complete:

### Twilio Account:
- [ ] ✅ Twilio account created and verified
- [ ] ✅ Trial phone number obtained (+1XXXXXXXXXX format)
- [ ] ✅ Your test phone number verified in Twilio
- [ ] ✅ Account SID copied and saved
- [ ] ✅ Auth Token copied and saved
- [ ] ✅ Twilio phone number copied and saved
- [ ] ✅ Free credit balance shows ~$15.50

### Supabase Email OTP:
- [ ] ✅ Email provider enabled (green toggle)
- [ ] ✅ Email OTP enabled (checkbox checked)
- [ ] ✅ Site URL set to https://izzy-yum.com
- [ ] ✅ Redirect URLs configured
- [ ] ✅ Email template customized (optional)

### Supabase Phone OTP:
- [ ] ✅ Twilio credentials added to Supabase settings
- [ ] ✅ Phone provider enabled (green toggle)
- [ ] ✅ Phone OTP enabled (checkbox checked)
- [ ] ✅ SMS provider set to "Twilio"

---

## 🧪 Quick Test (Optional)

Want to verify everything works before Phase 2 code integration?

### Test Email OTP (via Supabase Dashboard):

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/users
- [ ] Click **"Add user"** → **"Create new user"**
- [ ] Enter a test email (your email)
- [ ] Check **"Auto Confirm Email"** for now
- [ ] Create the user
- [ ] Check if you received an email from Supabase

### Test Phone SMS (Optional):

**Note:** This requires API calls or code testing, which will happen in Phase 2.
You can skip this test for now.

---

## 📝 Information for Phase 2

When Phase 1 is complete, provide this information:

```
✅ Phase 1 Complete

Twilio Phone Number Format: +1__________
(Example: +15551234567)

Any issues encountered: None / [describe any problems]
```

---

## 🚨 Troubleshooting

### Issue: Can't find Twilio credentials
**Solution:** Go to https://console.twilio.com/ and look for "Account Info" section

### Issue: Twilio SMS not sending
**Solution:** Verify the recipient's phone number is verified at https://console.twilio.com/us1/develop/phone-numbers/manage/verified

### Issue: Supabase email not arriving
**Solution:**
1. Check spam/junk folder
2. Verify Site URL is correct in Supabase settings
3. Check Supabase logs: https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/logs/edge-logs

### Issue: Phone number format error
**Solution:** Always use E.164 format: +[country code][number]
- ✅ Correct: +14155551234
- ❌ Wrong: (415) 555-1234
- ❌ Wrong: 415-555-1234

---

## 💰 Cost Summary

| Service | Setup Cost | Trial/Free Credit | Ongoing Cost |
|---------|-----------|-------------------|--------------|
| **Twilio Account** | $0 | $15.50 credit | $0.05 per SMS |
| **Supabase Email OTP** | $0 | Unlimited (free tier) | $0 |
| **Supabase Phone OTP** | $0 | Uses Twilio credit | $0 (uses Twilio) |
| **Total Setup** | **$0** | **$15.50 credit** | **~$0.05/SMS** |

**Notes:**
- Email OTP is completely free on Supabase free tier
- Phone SMS costs $0.05 each (charged to Twilio)
- Trial credit covers ~310 SMS messages
- Add $20+ to Twilio to remove trial limitations

---

## ⏭️ Next: Phase 2 (Code Integration)

Once all checkboxes above are complete:

1. ✅ Mark Phase 1 as complete
2. 🔄 Code integration will update the registration flow
3. 🚀 Deploy updated code to production
4. 🧪 Test with real OTP codes
5. ✅ Remove hardcoded "1234" verification

**Ready?** Report back when Phase 1 is complete!

---

*Document created: 2026-02-27*
*Project: Izzy Yum - Gluten-Free Recipe System*

---

## 🚨 Update: Phone Verification Temporarily Disabled

**Date:** March 1, 2026

**Reason:** Twilio requires campaign registration and approval for A2P 10DLC messaging, which can take up to 3 weeks.

**Current Implementation:**
- ✅ Email OTP verification: **Active** (6-digit code)
- ❌ Phone OTP verification: **Disabled** (temporarily)
- ✅ Phone number collection: **Active** (saved for future use)

**Registration Flow:**
1. User enters email, password, and phone number
2. Email OTP sent (6-digit code via email)
3. User verifies email
4. Phone number saved to user metadata (unverified)
5. Registration complete → user logged in

**To Re-enable Phone Verification:**
1. Complete Twilio A2P 10DLC campaign registration
2. Wait for approval (up to 3 weeks)
3. Uncomment phone OTP code in `src/app/register/page.tsx`
4. Redeploy to production

**Impact:**
- Users can register with email verification only
- Phone numbers are collected and stored for future shopping list features
- No SMS costs during this period ($0 instead of $0.05 per registration)

