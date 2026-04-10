A Google Apps Script that automatically archives newsletter emails from Gmail into structured Google Docs. Currently v1.3. The archived docs feed into a daily workflow using tools like NotebookLM, with plans drafted to build a dedicated front-end. 

The script runs daily on a timer and:
1. Searches Gmail for emails from a configurable list of newsletter domains
2. Extracts the content, strips HTML formatting, and pulls out article links
3. Writes everything into a dated Google Doc inside a "Newsletter Repository" folder in Google Drive
4. Labels processed emails `newsletter-archived` so they're skipped on the next run

Each email gets its own section in the doc with a heading, sender info, the plain text body, and a list of extracted links underneath for referncing later.

## Note: ensure you add your own domains by editing the `DOMAINS_TO_WATCH` array at the top of the script!!

## Setup

1. Go to [script.google.com](https://script.google.com) and create a new project
2. Paste the contents of `archiveNewsletters.gs` into the editor
3. Add the `DOMAINS_TO_WATCH` list to match the newsletters you subscribe to
4. Run `archiveNewsletters` once manually and authorise the required permissions (Gmail, Drive, Docs)
5. Set up a daily trigger:
   - Go to **Triggers** (clock icon in the left sidebar)
   - Click **Add Trigger**
   - Function: `archiveNewsletters`
   - Event source: **Time-driven**
   - Type: **Day timer**
   - Pick a time window (e.g. 6am - 7am)

## How it works

**Gmail query** — The script builds a search query that looks for emails from your watched domains received in the last 25 hours, excluding anything already labelled `newsletter-archived`. The 25-hour window (rather than 24) gives a small overlap to avoid missed emails.

**HTML to plain text** — Newsletter emails are heavily styled HTML. The script strips out `<style>` and `<script>` blocks entirely, converts block-level elements to line breaks, removes all remaining tags, and decodes common HTML entities.

**Link extraction** — Links are pulled from `<a>` tags in the original HTML. The script filters out tracking/redirect URLs from platforms like Beehiiv, Mailchimp, SendGrid, and ConvertKit, as well as unsubscribe links. Only clean destination URLs are kept.

**Deduplication** — Processed emails are labelled in Gmail, and the link extractor deduplicates by URL so each link only appears once per email.

## Cost

Free (🫡). The script runs entirely within Google Apps Script using built-in services (Gmail, Drive, Docs). No external APIs or paid tools required.

## Bugs fixed in v1.2

**CSS leaking into text** — `<style>` blocks were being stripped of their tags but leaving raw CSS (e.g. `@media` queries) in the plain text output. Fixed by removing style blocks before stripping tags.

**Tracking URLs in links** — Redirect URLs from email platforms (Beehiiv, Mailchimp, etc.) were being captured instead of actual article URLs. Fixed by adding a domain-based filter for known tracking services.
