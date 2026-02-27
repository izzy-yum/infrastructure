# Phase 1: Authentication Setup Checklist
## Twilio SMS & Supabase OTP Configuration

**Estimated Time:** 20-30 minutes
**Cost:** $0 (using free tiers and trial credits)

---

## 📱 Part A: Twilio Setup (10 minutes)

### Step 1: Create Twilio Account

- [ ] Go to https://www.twilio.com/try-twilio
- [ ] Click **"Start for free"** or **"Sign up"**
- [ ] Fill in registration form:
  - [ ] First name
  - [ ] Last name
  - [ ] Email address
  - [ ] Password (strong password recommended)
- [ ] Click **"Sign Up"**
- [ ] Check your email and verify your email address
- [ ] Complete phone verification (Twilio will call or text you)
- [ ] Answer the onboarding questions (select your use case)

### Step 2: Get a Trial Phone Number

- [ ] After login, you'll see the Twilio Console
- [ ] Look for "Get a Twilio phone number" prompt
- [ ] Click **"Get a trial number"**
- [ ] Review the suggested number (should be a US number like +1 555-123-4567)
- [ ] Click **"Choose this number"** (or click "Search for a different number" if you want to pick)
- [ ] Confirm the number is now in your account

### Step 3: Collect Twilio Credentials

**Go to:** https://console.twilio.com/

- [ ] Find the **Account Info** section on the dashboard
- [ ] Copy and save the following (you'll need these later):

```
Account SID: AC________________________________
(Starts with "AC" followed by 32 characters)

Auth Token: ________________________________
(Click "Show" to reveal, then copy)

Twilio Phone Number: +1__________
(Format: +15551234567)
```

- [ ] Save these credentials in a secure note/password manager
- [ ] **IMPORTANT:** Keep Auth Token secret - treat it like a password

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

- [ ] Click **"Save"** if you made changes

### Step 3: Configure Site URLs

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/url-configuration
- [ ] Set **Site URL** to: `https://izzy-yum.com`
- [ ] Under **Redirect URLs**, verify or add these URLs:
  - [ ] `https://izzy-yum.com`
  - [ ] `https://izzy-yum.com/verify-email`
  - [ ] `http://localhost:9001` (for local development)
- [ ] Click **"Save"**

### Step 4: Verify Email Settings

- [ ] Go back to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/providers
- [ ] Confirm **Email** provider shows as "Enabled" (green toggle)
- [ ] Confirm **Email OTP** is checked
- [ ] ✅ Email OTP is now configured

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
- [ ] Wait for "Settings saved successfully" confirmation

### Step 2: Enable Phone Provider

- [ ] Go to https://supabase.com/dashboard/project/oogpnrrsmpklafdrvvkg/auth/providers
- [ ] Scroll to the **"Phone"** section
- [ ] Toggle **ON** (should turn green): "Enable Phone provider"
- [ ] Under "Phone OTP" settings:
  - [ ] Check ✅ **"Enable Phone OTP"**
- [ ] Click **"Save"** at the bottom

### Step 3: Configure Phone Settings (Optional)

- [ ] Still in the Phone section, review these settings:
  - [ ] **OTP Expiry**: Default is 60 seconds (can change to 60-300 seconds)
  - [ ] **OTP Length**: Default is 6 digits (recommended)
- [ ] Leave defaults or adjust as needed
- [ ] Click **"Save"** if you made changes

### Step 4: Verify Phone Settings

- [ ] Confirm **Phone** provider shows as "Enabled" (green toggle)
- [ ] Confirm **Phone OTP** is checked
- [ ] ✅ Phone OTP is now configured

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
