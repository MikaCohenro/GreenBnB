# Shared reservations backend (Supabase)

The site works out of the box using per-browser `localStorage`. To make reservations
**shared across everyone and every device** (so Hadar can see who booked each room from
her own phone), connect a free Supabase project. Takes ~3 minutes.

## 1. Create a project
- Go to https://supabase.com → sign in → **New project** (free tier is fine).
- Pick any name/region, set a database password (you won't need it for the site).

## 2. Create the table
Open **SQL Editor** in the Supabase dashboard, paste this, and click **Run**:

```sql
create table reservations (
  id          bigint generated always as identity primary key,
  room        text not null,            -- 'suite' | 'topfloor' | 'kidsroom'
  name        text not null,            -- who the booking is under
  guests      int  not null default 1,
  check_in    date not null,
  check_out   date not null,
  created_at  timestamptz not null default now()
);

-- Allow the public anon key to read/insert/delete (no login on the site).
alter table reservations enable row level security;

create policy "public read"   on reservations for select using (true);
create policy "public insert" on reservations for insert with check (true);
create policy "public delete" on reservations for delete using (true);
```

## 3. Get your keys
- **Project Settings → API**
- Copy **Project URL** (e.g. `https://abcdxyz.supabase.co`)
- Copy the **anon public** key (a long JWT). This key is safe to ship in the page —
  it only allows what the policies above permit.

## 4. Paste them into the site
In `index.html`, near the top of the main `<script>`, replace:

```js
var SUPABASE_URL = 'YOUR_SUPABASE_URL';
var SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

with your real values, commit, and push. Done — reservations are now shared.

## Notes / tradeoffs
- The policies above are **fully public**: anyone with the page can add or cancel any
  reservation. That's fine for a family joke site. If you ever want it locked down,
  add Supabase Auth and tighten the policies.
- "Your reservations" (the cancellable list in the card) still tracks *which* bookings
  this browser created via `localStorage`, so each person can cancel their own. The
  **Host view** shows *everyone's* bookings across all rooms.
- If the keys are left as placeholders, the site silently falls back to per-browser
  `localStorage` — nothing breaks.
