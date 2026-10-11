# Tiruppur Thozhil (திருப்பூர் தொழில்) - Supabase Backend Setup Guide

This guide explains how to connect your **Tiruppur Thozhil** website to your free **Supabase** backend.

---

## 1. Quick Directory Overview

```
tiruppur-thozhil/
├── index.html            # Complete website (Frontend + Supabase integration)
├── config.js             # Public Supabase credentials (URL & Anon Key)
├── supabase-schema.sql   # Complete PostgreSQL schema, triggers & RLS policies
├── .env.example          # Environment variables reference
└── README.md             # This setup guide
```

---

## 2. Step-by-Step Supabase Setup

### Step 2.1: Create a Free Supabase Project
1. Go to [supabase.com](https://supabase.com) and click **Start your project** (free tier).
2. Create an organization and click **New Project**.
3. Choose a name (e.g. `tiruppur-thozhil`), set a secure database password, and select region **India (Mumbai) / ap-south-1** for lowest latency.
4. Wait 1–2 minutes for the database to provision.

---

### Step 2.2: Run the Database Schema & RLS Policies
1. In your Supabase project dashboard, click on **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Copy the entire contents of [`supabase-schema.sql`](file:///home/sathish/.gemini/antigravity/scratch/tiruppur-thozhil/supabase-schema.sql) and paste it into the editor.
4. Click **Run** (or press `Ctrl+Enter`).
5. You will see a success message. This creates:
   - `profiles` table (Labourers and Garment Entrepreneurs)
   - `jobs` table (Live garment job vacancies)
   - `job_applications` table (Labourer job applications)
   - `handle_new_user()` trigger for automated profile generation on signup
   - Comprehensive **Row Level Security (RLS)** policies protecting all tables.

---

### Step 2.3: Configure Supabase Auth (Recommended for Local/Pilot Testing)
1. In your Supabase dashboard, go to **Authentication** -> **Providers** -> **Email**.
2. **Turn off "Confirm email"** if you want instant account activation during development and testing without having to click email confirmation links.
3. Click **Save**.

---

### Step 2.4: Configure Your Public Keys
1. In your Supabase dashboard, go to **Project Settings** (gear icon) -> **API**.
2. Copy the **Project URL** (e.g., `https://xyzproject.supabase.co`).
3. Copy the **Project API Key** under `anon` `public` (e.g., `eyJhbGciOi...`).
   > [!CAUTION]
   > NEVER use the `service_role` secret key in frontend files! Only use the `anon` public key.
4. Open [`config.js`](file:///home/sathish/.gemini/antigravity/scratch/tiruppur-thozhil/config.js) and paste the values:
   ```javascript
   window.SUPABASE_CONFIG = {
     url: "https://your-project-id.supabase.co",
     anonKey: "your-anon-key-here"
   };
   ```
*(Alternative: You can also click the "Enter Supabase Keys" button on the webpage to save them directly in your browser's local storage).*

---

## 3. Testing the Application Features

1. **Open `index.html` in your browser**.
2. **Test Entrepreneur (Garment Owner) Workflow**:
   - Scroll to **Account Access** or click **Register**.
   - Select role: **Garment Owner / Entrepreneur**.
   - Fill in name, mobile number, email, password (min 6 characters), and company name.
   - Click **Create Account**.
   - Notice the topbar now shows your name and **Entrepreneur** badge.
   - Scroll to **Post a Job**: notice your company name and phone are auto-filled!
   - Fill in the vacancy (e.g., Tailor, 3 workers needed, Daily salary) and click **Publish Job**.
   - Check **My Dashboard**: see your posted job and status toggle (Active/Closed).
3. **Test Labourer (Worker) Workflow**:
   - Log out by clicking **Logout** in the topbar.
   - Register a new account with role: **Worker / Labourer** (e.g., skill: Tailor).
   - In the **Find Jobs** feed, click **Apply Now** on the job posted by the entrepreneur.
   - The button immediately updates to `✓ Applied`.
   - In **Labourer Dashboard**, view your application status (`Pending Review`) and call action.
4. **Test Entrepreneur Applicant Management**:
   - Log out and log back in as the Entrepreneur.
   - Go to **Entrepreneur Dashboard** -> **Received Worker Applications**.
   - View the worker's name, phone, area, and skill.
   - Click **Call** or **WhatsApp** to reach out.
   - Update status to `Contacted` or `Reviewed`.
5. **Test Language Toggle**:
   - Click `தமிழ்` in the topbar: all elements, dashboards, statuses, and badges seamlessly switch to Tamil.

---

## 4. Security & Permissions Checklist

- [x] **No Plaintext Passwords**: Passwords are encrypted with bcrypt and stored exclusively in `auth.users` by Supabase Auth.
- [x] **Public Key Safety**: Only the public `anon` key is exposed, safe for frontend use.
- [x] **Database-Enforced Authorization**:
  - Only `owner` profiles can create jobs.
  - Only `worker` profiles can apply to jobs.
  - Job owners can only edit and delete their own jobs.
  - Applicants can only view their own applications.
  - Entrepreneurs can only view applications sent to their jobs.
