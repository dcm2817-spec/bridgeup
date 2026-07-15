# BridgeUp

Mobile-first mentor-matching flow: homepage → mentor list → mentor profile → message form.
Plain HTML/CSS/JS, no framework, no build step. Deploys to Vercel as-is.

## 1. Set up Supabase

1. Create a project at supabase.com if you don't already have one for this.
2. Go to **SQL Editor** and run everything in `schema.sql`. This creates the
   `mentors` and `messages` tables, turns on Row Level Security, and adds
   policies that match the "no login" design:
   - Anyone can **read** mentors.
   - Anyone can **insert** into messages.
   - Nobody (via the public anon key) can **read** messages — only you, from
     the Supabase dashboard. That's deliberate: without it, any visitor could
     read every student's message to every mentor.
3. The bottom of `schema.sql` seeds 4 sample mentors so you can test the flow
   immediately. Delete that block once you're adding real mentors.

## 2. Connect the app to your project

`config.js` already has your project URL and **publishable key** filled in.
That key is meant to be public in client code — it's not a secret, RLS is
what actually controls access.

Your Supabase project also has a **secret key** (`sb_secret_...`). That one
bypasses RLS entirely and must never go in `config.js`, any other client
file, or a public git repo. Keep it private — you'd only use it in
server-side code (e.g. an Edge Function), never in the browser.

## 3. Run it locally

No build step. Any static server works, e.g.:

```
npx serve .
```

(Opening `index.html` directly with `file://` will break the module imports —
serve it over http.)

## 4. Deploy

Push this folder to a GitHub repo and import it in Vercel with **no framework
preset** (Other / Static). No environment variables needed since the keys
live in `config.js`.

## Files

- `index.html` — homepage, single "Find Mentor" CTA
- `mentors.html` — mentor grid, fetched from Supabase
- `mentor.html` — single mentor profile, reads `?id=`
- `message.html` — message form, inserts into `messages`, reads `?id=`
- `styles.css` — shared design tokens and layout
- `config.js` — your Supabase URL + anon key
- `schema.sql` — table definitions, RLS policies, sample seed data

## Known limitations (read before you show this to real students)

- **No mentor notification.** A message lands in the `messages` table, but no
  email or alert goes to the mentor. You either need to check the Supabase
  table manually or add a notification (e.g. a Supabase Edge Function that
  emails the mentor on insert) before this is usable in practice.
- **No spam protection.** Since there's no login and the insert policy is
  wide open, anyone can script-submit the message form repeatedly. Fine for
  an MVP with a small trusted user base; add rate limiting or a honeypot
  field before pointing broad traffic at it.
- **No edit/remove path for mentors.** Adding or updating mentors is a manual
  SQL/table-editor job for now, matching "no extra features" — but that
  means it doesn't scale past a handful of mentors without you doing data
  entry by hand.
