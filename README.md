Overview

The tracker is a single HTML file hosted on GitHub Pages. All logic runs in your browser — there's no backend server. Data is pulled live from Google APIs on each sync and stored locally in your browser's localStorage.

Authentication

The tracker uses Google OAuth 2.0 with PKCE to connect to your Google account. On first load, you enter the password (shopee), which decrypts the hardcoded Google Client ID, Client Secret, and Anthropic API key from XOR-encoded strings in the HTML. This was done to avoid GitHub's secret scanning blocking plaintext credentials in a public repo.

Once authenticated, the access token is saved to localStorage. A refresh token handles silent re-authentication, so you don't need to reconnect Google on every visit. The aopt_unlocked flag means you also never have to re-enter the password after the first time on a given browser.

Data Sources

Main project list (S.rr)
The RR Projects list was originally pulled from the main Google Sheet (14egvvW…) on first sync. It now lives entirely in localStorage — syncs no longer refresh it from the sheet. You manage it manually via the Edit popup or the "+ New project" button.

Source sheet (synthesis input)
On every sync, the tracker reads the "RR Projects" tab of a second sheet (1C51vA2…), looked up by tab name (not GID, so tab reorganisation won't break it). It filters rows where:

Column D = "Noelle" (Team Lead)
Column E = one of: Mark, Seth, Akshit, Le Yu, Vidya, Cherie (Reg RR PIC)

From matching rows it extracts:

Column C → project name (for matching to the tracker)
Column L → Status Remarks
Column W → Weekly Update

Gmail
Searches the last 7 days of your connected Gmail for emails with subject Fwd: Work Emails as of. Up to 5 emails are fetched and their bodies extracted.

Google Calendar
Fetches events from 3 days ago to 7 days ahead from your primary calendar.

Project Matching

After fetching the source sheet, each source row is matched to a project in the tracker:

Exact name match → direct match (confidence score 1)
Confirmed via mapJudgements → direct match (confidence score 2, overrides fuzzy)
Word overlap ≥ 35% → uncertain match (shows Confirm/Reject buttons in the Sheet Match column)
Below threshold → unmapped (logged, needs manual action)
extraMappings → hardcoded system overrides that bypass the matching loop entirely. Currently:
DMS Features ← also pulls from DMS Requests (Adhoc)
L3 Headcount Reduction Initiatives ← pulls from L3 agent operational efficiency

Confirmed/rejected judgements are saved to localStorage permanently so you don't re-decide on every sync.

AI Synthesis

After matching, Claude (Haiku model) generates a 3-section Status Elaboration for every project. The synthesis priority for source material is:

Additional Context field (from the project popup) — overrides Column W if it was saved after the last sync
Column W (Weekly Update) — primary when no fresh manual context
Column L (Status Remarks) — fallback if W is empty
Gmail + Calendar — supplementary signals matched to the project by keyword overlap

Claude is instructed to output exactly three sections: Status, Next Steps, Risks — each with 1–3 bullet points, max 20 words per bullet.

Persistence

Everything is saved to localStorage under the key aopt:

The full project list (S.rr) and all manual edits (S.edits)
AI-generated elaborations (S.elab)
Auth tokens
Source row cache, match judgements, and extraMappings
Automation Pipeline and Team Scope tab data

A Google Drive backup (aopt-backup.json) is auto-saved at 5:30pm SGT daily and can be manually triggered via the ☁ Save button. The ↩ Restore button pulls it back.

Auto-Sync

The tracker schedules a background sync daily at 6:00am UTC. Manual sync is via the "Sync now" button. The sync log panel shows every step and can be dismissed.

Tabs
RR Projects — the main view with filters, search, AI elaboration, and task tracking
Automation Pipeline — editable table pulled from the main sheet on first load, then local
Team Scope — same pattern
PIC View — auto-generated breakdown of projects by person in charge
