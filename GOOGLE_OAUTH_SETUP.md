# Google OAuth Setup Guide

## What's Been Implemented ✅

The code for Google OAuth is now in place:
1. ✅ `handleGoogleSignIn()` function added to `auth.js`
2. ✅ Google Sign-In button added to `login.html`
3. ✅ Google Sign-Up button added to `register-student.html`
4. ✅ Company registration page kept as email/password only (as requested)

## Required: Supabase Configuration

> [!IMPORTANT]
> **Manual Setup Required**
> 
> Google OAuth will NOT work until you complete these steps in your Supabase dashboard and Google Cloud Console.

---

## Step 1: Create Google OAuth Credentials

### 1.1 Go to Google Cloud Console

1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Sign in with your Google account

### 1.2 Create or Select Project

1. Click the project dropdown at the top
2. Either:
   - Create a new project: Click "NEW PROJECT"
     - Name it "Intern Academy" or similar
     - Click "CREATE"
   - Or select an existing project

### 1.3 Enable Google+ API

1. In the left sidebar, go to **APIs & Services** → **Library**
2. Search for "Google+ API"
3. Click on it and press **ENABLE**
4. (If already enabled, you'll see "Manage" instead)

### 1.4 Create OAuth Credentials

1. Go to **APIs & Services** → **Credentials**
2. Click **+ CREATE CREDENTIALS** → **OAuth client ID**
3. If prompted to configure consent screen:
   - Click **CONFIGURE CONSENT SCREEN**
   - Select **External** (unless you have Google Workspace)
   - Click **CREATE**
   - Fill in required fields:
     - **App name**: Intern Academy
     - **User support email**: Your email
     - **Developer contact email**: Your email
   - Click **SAVE AND CONTINUE**
   - Skip "Scopes" → Click **SAVE AND CONTINUE**
   - Skip "Test users" → Click **SAVE AND CONTINUE**
   - Click **BACK TO DASHBOARD**

4. Now create OAuth Client ID:
   - Go back to **Credentials**
   - Click **+ CREATE CREDENTIALS** → **OAuth client ID**
   - Application type: **Web application**
   - Name: "Intern Academy Web"
   
5. **IMPORTANT**: Add Authorized Redirect URIs

---

## Step 2: Get Supabase Callback URL

### 2.1 Find Your Supabase Project Reference

1. Go to your Supabase Dashboard: https://supabase.com/dashboard
2. Select your project
3. Go to **Settings** (gear icon) → **API**
4. Find your **Project URL**
   - It will look like: `https://abcdefghijklmnop.supabase.co`
   - The part before `.supabase.co` is your project reference ID

### 2.2 Construct Callback URL

Your callback URL will be:
```
https://YOUR-PROJECT-REF.supabase.co/auth/v1/callback
```

For example, if your project URL is `https://abcdefgh.supabase.co`, then your callback URL is:
```
https://abcdefgh.supabase.co/auth/v1/callback
```

---

## Step 3: Complete Google OAuth Setup

### 3.1 Add Redirect URI to Google

1. Back in Google Cloud Console → **Credentials**
2. Under "Authorized redirect URIs", click **+ ADD URI**
3. Paste your Supabase callback URL:
   ```
   https://YOUR-PROJECT-REF.supabase.co/auth/v1/callback
   ```
4. Click **CREATE**

### 3.2 Copy Credentials

After creating, you'll see:
- **Client ID** (looks like: `123456789-abc123.apps.googleusercontent.com`)
- **Client Secret** (looks like: `GOCSPX-abc123xyz`)

**Copy both of these** - you'll need them in the next step!

---

## Step 4: Configure Supabase

### 4.1 Enable Google Provider

1. Go to Supabase Dashboard → Your Project
2. Navigate to **Authentication** → **Providers**
3. Find **Google** in the list
4. Toggle it to **Enabled**

### 4.2 Enter Google Credentials

1. Paste **Client ID** from Google Cloud Console
2. Paste **Client Secret** from Google Cloud Console
3. Click **Save**

### 4.3 Configure Redirect URLs (Optional but Recommended)

1. Go to **Authentication** → **URL Configuration**
2. Set **Site URL** to your production domain
   - For local testing: `http://localhost:5500` (or your local port)
   - For production: `https://yoursite.com`
3. Add **Redirect URLs** (optional):
   - `http://localhost:5500/dashboard.html` (for local testing)
   - `https://yoursite.com/dashboard.html` (for production)
4. Click **Save**

---

## Step 5: Test Google Sign-In

### 5.1 Test on Login Page

1. Open your website (locally or hosted)
2. Go to the login page
3. Click **"Sign in with Google"**
4. You should be redirected to Google's sign-in page
5. Select your Google account
6. Grant permissions
7. You should be redirected back to your dashboard

### 5.2 Test on Student Registration

1. Go to the student registration page
2. Click **"Continue with Google"**
3. Follow the same flow
4. Should redirect to dashboard

### 5.3 Verify User Created

1. Go to Supabase Dashboard → **Authentication** → **Users**
2. You should see your new user
3. Check that:
   - Email is from Google account
   - Provider is "google"
   - User metadata contains name from Google

---

## Important Notes

### User Data Handling

When users sign in with Google:
- ✅ Their Google email is automatically used
- ✅ Their Google name goes to `user_metadata.full_name`
- ✅ Email is verified by default (Google handles this)
- ⚠️ They still need to complete their profile (phone, college, etc.)

### Next Steps After Google Sign-In

You may want to:
1. **Redirect new Google users to complete profile page**
   - Check if `student_registrations` record exists
   - If not, show a profile completion form
   - Save phone, college, graduation year, etc.

2. **Or create a minimal profile automatically**
   - Create `student_registrations` record on first Google sign-in
   - Use name/email from Google
   - Leave other fields empty for user to fill later

### Company Accounts

As requested:
- ❌ **Google sign-in is NOT enabled for companies**
- ✅ Companies must use email/password registration
- ✅ This ensures professional email addresses are used

---

## Troubleshooting

### Error: "Redirect URI mismatch"

**Cause**: The redirect URI in Google Cloud doesn't match Supabase callback URL.

**Fix**:
1. Double-check your Supabase project URL
2. Ensure the callback URL in Google Cloud is EXACTLY:
   ```
   https://YOUR-PROJECT-REF.supabase.co/auth/v1/callback
   ```
3. No trailing slashes, must use `https://`

### Error: "Access blocked: This app's request is invalid"

**Cause**: OAuth consent screen not configured properly.

**Fix**:
1. Go to Google Cloud → **OAuth consent screen**
2. Ensure app status is "In production" or add your email as a test user
3. Fill in all required fields

### Google Sign-In button doesn't work

**Cause**: Provider not enabled in Supabase or credentials incorrect.

**Fix**:
1. Check Supabase → Authentication → Providers → Google is enabled
2. Verify Client ID and Secret are correct (no extra spaces)
3. Check browser console for errors

### Users created but not in student_registrations table

**Cause**: Google OAuth creates user in `auth.users`, but doesn't automatically create profile record.

**Fix**: You need to add code to create profile on first Google sign-in (see below).

---

## Optional: Auto-Create Profile for Google Users

Add this to your dashboard.html to handle new Google users:

```javascript
async function loadDashboard() {
    try {
        const session = await protectPage();
        if (!session) return;

        currentUser = session.user;

        // Check if profile exists
        const { data: profile, error } = await supabase
            .from('student_registrations')
            .select('*')
            .eq('user_id', currentUser.id)
            .single();

        if (!profile) {
            // New Google user - create minimal profile
            const { error: insertError } = await supabase
                .from('student_registrations')
                .insert({
                    user_id: currentUser.id,
                    full_name: currentUser.user_metadata.full_name || currentUser.email,
                    email: currentUser.email,
                    // Other fields can be null for now
                });

            if (insertError) {
                console.error('Error creating profile:', insertError);
            } else {
                // Optionally show a "Complete your profile" banner
                alert('Welcome! Please complete your profile to apply for internships.');
            }
        }

        // Continue with normal dashboard loading...
    } catch (error) {
        console.error('Dashboard error:', error);
    }
}
```

---

## Summary

✅ **What's Done**:
- Google OAuth code implemented in frontend
- Sign-in buttons added to login and student registration pages
- Companies kept as email/password only

⏳ **What You Need to Do**:
1. Create Google OAuth credentials in Google Cloud Console
2. Enable Google provider in Supabase
3. Add credentials to Supabase
4. Test the sign-in flow
5. (Optional) Add profile auto-creation for Google users

Once configured, students will be able to sign up and sign in with just one click! 🎉
