# Modern Line Furniture — Lead Finder manual

Link: https://techhausinc1-wq.github.io/furniture-cockpit-demo/
Login: just a 4-digit PIN, no name or password. Vlad 1958, Yana 3072.

---

## Part 1 — how it works day to day

**Finding leads.** The "Find new leads" panel searches the web live for
real, currently-operating companies — reps, wholesalers, dealers, or
partner manufacturers — matching whatever category and region you pick.
Every result is cross-checked against your existing list so you never see
the same company twice.

**Region quick-picks.** Below the search fields, the chips (Northeast,
Southeast, Southwest, etc.) fill in the region field for you — they match
how furniture reps actually describe their territory, not just a single
state.

**A lead's detail card.** Click a company's name (not the little ↗ next to
it — that opens their actual website) to see everything on file for that
lead in one place: category, territory, why it's a fit, contact info,
status, who it's assigned to, and any notes.

**Emailing a lead as a cheat sheet.** From that detail card, "✉ Email this
card" opens Gmail with everything about that lead formatted into one
message — company, why to reach out, contact info, status — plus a spot
for your own note at the top. Good for handing a lead to someone else with
full context, not just a name.

**Assigning work.** "Send to…" on any lead assigns it to a teammate with a
note. "Team" in the header shows everyone with access and lets you add new
people — each gets an invite (their PIN + the link) you can copy or email.

**Selecting and exporting.** Check the boxes on the left of any leads you
want, then either "Export to Excel" (downloads a CSV) or "Export to Google
Sheet" (creates a real spreadsheet in whoever's Google account you pick,
auto-sized columns, no extra setup). Leave nothing checked and it exports
everything instead.

**Daily Scout** (🔍 button above the search panel) runs every category,
nationwide, in one click — meant for "just find me whatever's new today"
instead of picking a category and region manually. Whatever it finds gets
auto-selected, and it asks if you want to export just those to a Google
Sheet right away.

---

## Part 2 — this can all run by itself

Everything in Part 1 needs someone to open the app and click something.
There's a second version of Daily Scout that needs nobody at all: it runs
on a schedule, saves new leads the same way, and both fills in a running
Google Sheet and emails your team automatically — no browser, no login,
no manual export.

This is a different piece of infrastructure than the one-click export
button, because "nobody's watching" means there's no human available to
pick a Google account or click Send on an email. It uses two things
instead:

- **A Google service account** (not a person's login) to write to the
  spreadsheet — this doesn't expire the way a manual sign-in does.
- **Resend** (a transactional email API) to actually send the digest email
  — opening a Gmail draft and waiting for someone to click Send doesn't
  work when nobody's there to click it.

### Setting it up (~15 minutes total, one time)

**Google Sheets side** — full walkthrough in
`docs/DAILY-SCOUT-AUTOMATION-SETUP.md` in this repo. Short version: create
a service account in Google Cloud Console, download its key, create one
spreadsheet and share it with that service account's email, send me the
key and the spreadsheet ID.

**Email side** — sign up at resend.com (free tier is plenty for this
volume), verify a sending domain (or use their shared test domain to
start), create an API key. Send me the API key and which address you want
emails to come from. Then go to **Team** in the app and make sure whoever
should get the daily digest has an email address on their account — the
digest goes to everyone with one on file, nobody else.

Once both are set, tell me and I'll turn on the schedule. From that point
it runs with zero further action, indefinitely.

### What happens once it's live

Every day at 8am Central: searches all 5 categories nationwide, skips
anything already in your list, saves whatever's genuinely new the same way
a manual search would, adds those rows to the bottom of one running
"Daily Scout Log" spreadsheet (never overwrites previous days), and emails
a summary to your team — each new lead with its territory, why to reach
out, and contact info, so nobody has to open the app to know what showed
up.

You can also trigger this exact same run manually any time, without
waiting for the schedule — useful to test it once everything's
configured, or just to run it on demand:

```
curl -X POST https://igor-hq.techhausinc1.deno.net/mlf/scout/run-now \
  -H "X-Access-Code: modernline2026"
```

### Changing the schedule — daily vs. weekly

It's set to run every day by default. To switch it to weekly (say, every
Monday morning instead), the change is one line in
`~/code/igor-hq/worker/main.ts` — find:

```ts
Deno.cron("mlf-daily-scout", "0 13 * * *", async () => {
```

and change `"0 13 * * *"` (every day at 13:00 UTC = 8am Central) to
`"0 13 * * 1"` (every **Monday** at the same time — the trailing `1` picks
the day of week, 0=Sunday through 6=Saturday). Tell me if you want this
changed and I'll make the edit and redeploy — takes under a minute once
you know which cadence you want.
