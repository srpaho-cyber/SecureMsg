# unknow — Private Messaging App

> *Private messaging — no one knows who you are*

A minimal, WhatsApp-style messaging app built with pure HTML + Supabase. No frameworks, no build tools. Deploy for free on GitHub Pages.

---

## Features

- **Username + Password login** — no email required from the user
- **1-week persistent session** — stay logged in automatically
- **Unique display name** — chosen once after registration; how others find you
- **Search users** — find anyone by their display name
- **Real-time messaging** — messages appear instantly via Supabase Realtime
- **Pagination** — loads 30 messages at a time with a "Load older" button
- **WhatsApp-style UI** — sent/received bubbles, timestamps, date dividers
- **Mobile responsive** — full-screen sidebar + chat panel on small screens
- **SQL with proper indexing** — fast queries even at scale

---

## Project Structure

```
unknow/
├── index.html   ← entire frontend (one file, zero dependencies)
└── setup.sql    ← run once in Supabase SQL Editor
```

---

## Setup Guide

### Step 1 — Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign up (free)
2. Click **New Project**, give it a name, set a database password
3. Wait ~2 minutes for it to provision

### Step 2 — Run the SQL

1. In your Supabase dashboard → **SQL Editor** → **New Query**
2. Open `setup.sql`, copy everything, paste it in, click **Run**
3. You should see "Success" with no errors

### Step 3 — Set session duration to 1 week

1. Supabase dashboard → **Authentication** → **Settings**
2. Scroll to **JWT expiry** → set to `604800` (7 days in seconds)
3. Set **Refresh token expiry** to `604800` as well
4. Click **Save**

### Step 4 — Get your API keys

1. Supabase dashboard → **Project Settings** → **API**
2. Copy:
   - **Project URL** (looks like `https://xxxx.supabase.co`)
   - **anon / public** key

### Step 5 — Configure `index.html`

Open `index.html` and find these two lines near the top of the `<script>` section:

```js
const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_ANON_KEY';
```

Replace with your actual values from Step 4.

### Step 6 — Deploy to GitHub Pages (free hosting)

1. Create a new repository on [github.com](https://github.com)
2. Upload `index.html` (you don't need `setup.sql` in the repo — it's just a one-time tool)
3. Go to repo **Settings** → **Pages**
4. Under **Source**, select **Deploy from a branch**
5. Choose `main` branch, `/ (root)` folder → click **Save**
6. Wait ~1 minute, your app will be live at:
   ```
   https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/
   ```

---

## How It Works

### Registration flow
1. User picks a **username** (3–20 chars, used to log in)
2. Account is created with an internal email (`username@unknow.app`)
3. After first login, user picks a **display name** (public, used for search)
4. Display name is permanent and unique across all users

### Login flow
1. User enters username + password
2. App looks up the internal email from the `profiles` table
3. Signs in via Supabase Auth
4. Session is stored in `localStorage` and auto-refreshed for 7 days

### Messaging flow
1. Search for a user by display name
2. Click their name — a conversation row is created in the DB (or the existing one is opened)
3. Messages are stored in the `messages` table with a `conversation_id`
4. Supabase Realtime pushes new messages instantly to both participants

---

## Database Schema

### `profiles`
| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid | References `auth.users` |
| `username` | text | Login handle, unique |
| `email` | text | Internal, derived from username |
| `display_name` | text | Public search name, unique |
| `created_at` | timestamptz | |

### `conversations`
| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid | Primary key |
| `user1_id` | uuid | References `profiles` |
| `user2_id` | uuid | References `profiles` |
| `last_message` | text | Preview shown in sidebar |
| `last_message_at` | timestamptz | Used for ordering |
| `created_at` | timestamptz | |

### `messages`
| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid | Primary key |
| `conversation_id` | uuid | References `conversations` |
| `sender_id` | uuid | References `profiles` |
| `content` | text | 1–5000 characters |
| `created_at` | timestamptz | Pagination key |

### Indexes
```sql
idx_messages_conv_created   -- fast pagination per conversation
idx_conversations_user1     -- conversation list for user1
idx_conversations_user2     -- conversation list for user2
idx_profiles_display_name   -- search by display name
idx_profiles_username       -- lookup at login
```

### Row Level Security
- **Profiles** — anyone can read (for search); only owner can write
- **Conversations** — only the two participants can read/write
- **Messages** — only participants of the parent conversation

---

## Customization

### Change the app name
Search for `unknow` in `index.html` and replace with your preferred name.

### Change colors
Edit the CSS variables at the top of `<style>`:
```css
:root {
  --accent: #7c6af7;      /* primary purple */
  --sent-bg: #3d2fa8;     /* sent bubble color */
  --bg: #0a0a0f;          /* page background */
}
```

### Change message page size
```js
const PAGE_SIZE = 30;  // load N messages at a time
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML + CSS + JS (single file) |
| Auth | Supabase Auth |
| Database | Supabase (PostgreSQL) |
| Realtime | Supabase Realtime (WebSocket) |
| Hosting | GitHub Pages |
| Cost | **$0** |

---

## License

MIT — do whatever you want with it.
