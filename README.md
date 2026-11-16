# Damir Job Radar v4

https://damir-job-radar-v4.streamlit.app/

Automatic vacancy search for construction, infrastructure, WASH, procurement, project management and heritage roles, with explainable 0–100 fit scoring against Damir Hadžić's profile.

Three files. No API keys, no secrets, no database, no GitHub Actions, no paid services.

```text
app.py
requirements.txt
README.md
```

## What changed in v4 (repair)

| Problem | Cause | Repair |
|---|---|---|
| UNHCR: `HTTP 429` | Too many requests in a short time (one search per keyword, plus vacancy pages), or Workday's Cloudflare bot check | Small Workday sites are now read in one go (UNHCR: 1 request instead of 10+). Requests to the same site are paced. `Retry-After` is respected with back-off. A browser-like connection (`curl_cffi`) is used. A Cloudflare bot check is now named clearly instead of shown as a plain 429. |
| UNICEF: `no vacancies found on the listing page` | The listing page fills its results with scripts, so the downloaded page had no job links | The app now reads UNICEF's keyword search pages and the lighter mobile listing, accepts both page layouts, and merges results by job number. If it still fails, the message says what the page contained. |
| A failure stayed for 6 hours | Failed searches were cached like successful ones | Failed sources retry automatically after 20 minutes (or immediately with **Search now**). The last good results of a failing source stay visible. |

## What it does

- Searches the internet automatically when you open the app, and refreshes every 6 hours (**Search now** forces a fresh search).
- Sources: UNDP, IOM, WFP, UNHCR, UNICEF, Arbeitnow (Europe). Add more in the **Sources** tab (Workday, Oracle, Greenhouse, Lever, Ashby, PageUp, RSS).
- Reads the full vacancy text for the most relevant titles (up to 25 per site).
- Removes duplicates and expired vacancies.
- Scores every job and explains it: strengths, gaps, items to verify.
- Flags eligibility issues: national posts outside Bosnia and Herzegovina, nationality restrictions, licences, languages you do not list.
- Tabs: Best matches, All results, New, Closing soon, My tracker, Analyze vacancy, Import CSV, Sources, Help.
- Tracker (Saved / Applied / Interview / Rejected / Not interested), CSV and Markdown exports, and a CV-tailoring prompt for every vacancy.

## Update your existing repository

1. Open your repository on GitHub.
2. **Add file → Upload files**, drag in the three files, **Commit changes** (they replace the old ones).
3. Streamlit redeploys automatically. If it does not: app menu → **Reboot app**.

The new `requirements.txt` adds `curl_cffi`, so the first redeploy takes a few minutes longer.

## New deployment

1. Create a GitHub repository and upload the three files.
2. https://share.streamlit.io → **Create app** → repository, branch `main`, main file `app.py`.
3. **Advanced settings** → Python **3.12** → **Deploy**. No secrets needed.

## Run on your computer (Windows PowerShell)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
streamlit run app.py
```

## Reading the Sources tab

| Status | Meaning | What to do |
|---|---|---|
| OK / OK (cached) | Working | Nothing |
| Failed, showing last results | Temporary problem; older results still shown | Nothing. Automatic retry after 20 minutes |
| Failed: *rate limited* | The site asked for fewer requests | Wait; it retries automatically |
| Failed: *blocked by bot protection* | The site blocks requests from the hosting server | Run the app on your computer, or use that agency's email job alerts |
| Failed: *no vacancies read (page '…', 0 job links)* | The page layout changed | Use Analyze vacancy for that agency meanwhile |

The **Browser-like connection** line shows whether `curl_cffi` is installed.

## Scoring (100 points)

Title 20 · competencies 25 · organization/sector 15 · experience 15 · education 10 · location/eligibility 5 · language 5 · contract 5.
85+ Excellent · 75+ Strong · 65+ Good · 50+ Possible.

- **Disqualified**: unrelated role family or explicit nationality restriction (hidden from main tabs; still in "Download all").
- **Review required**: strong match with a possible blocker.
- **Verify**: unknowns, never counted as matched. French and Italian appear here until you set your level in `PROFILE` at the top of `app.py`.

## Limits

- These are the career sites' public website endpoints, not official APIs. They can change or block automated access without notice.
- Not searched automatically: LinkedIn, Indeed, UN Careers (Inspira), UNOPS. Paste those into **Analyze vacancy**.
- Streamlit Community Cloud keeps nothing between restarts: download your tracker and settings regularly.
- If `curl_cffi` ever fails to install, delete that line from `requirements.txt`. The app works without it (with a higher chance of bot blocks).
