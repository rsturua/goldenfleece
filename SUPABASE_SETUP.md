# 🚀 Supabase Setup Guide for GoldenFleece

## Problem Identified

The current Supabase configuration has invalid or incomplete credentials:
- **URL**: `https://qyhxgswnrkkwfmukvmie.supabase.co` - Cannot be reached (DNS error)
- **ANON_KEY**: Appears truncated/invalid (should be ~250+ characters)

## ✅ Solution: Set Up a New Supabase Project

Follow these steps to get authentication working:

---

## Step 1: Create a Supabase Project

1. Go to https://app.supabase.com
2. Click **"New Project"**
3. Fill in:
   - **Name**: GoldenFleece (or any name)
   - **Database Password**: Create a strong password (save it!)
   - **Region**: Choose closest to you
4. Click **"Create new project"**
5. Wait 2-3 minutes for the project to be ready

---

## Step 2: Get Your API Credentials

1. In your Supabase dashboard, click **Settings** (⚙️ icon in sidebar)
2. Click **API** in the settings menu
3. You'll see:
   - **Project URL**: `https://xxxxxxxxxxxxx.supabase.co`
   - **anon/public key**: A very long string starting with `eyJ...`

4. Copy these values!

---

## Step 3: Update Your .env.local File

Replace the values in `.env.local` with your actual Supabase credentials.

The anon key should be 200+ characters long (it's a JWT token).

---

## Step 4: Set Up the Database Tables

Run the SQL queries in the Supabase SQL Editor to create the profiles table and triggers.

---

## Step 5: Test Your Setup

1. Restart your dev server
2. Go to http://localhost:3000/signup
3. Create a test account
4. Should redirect to dashboard!

---

See full instructions in the project documentation.
