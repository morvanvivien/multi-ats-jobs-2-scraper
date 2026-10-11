# How to Scrape Recruitee, Personio and Teamtailor Job Postings

> **Run it now, no code: [multi-ats-jobs-2-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-2-scraper)** &nbsp;|&nbsp; $1.20 per 1,000 postings &nbsp;|&nbsp; no login

Recruitee, Personio and Teamtailor are three of the applicant tracking systems (ATS) that companies use to run their career pages. Each one has its own public feed, but reading all three normally means building three separate scrapers. This guide shows how to get job postings from all three ATS platforms in one run — title, department, location, remote policy, salary when published — with a free Apify Actor, from the console or from code.

Searches this page answers: recruitee scraper, personio jobs scraper, teamtailor scraper, ATS jobs scraper.

## What you get from the Recruitee, Personio & Teamtailor scraper

- Job postings from company career pages hosted on any of the three platforms, with the same output fields whatever the platform.
- Department, location and remote policy for every posting.
- Salary range and currency, when the board publishes it (common on Recruitee's US boards).
- Published and updated dates, plus the full description as plain text when enabled.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Unique posting id (`platform:board:postingId`) and public link |
| `platform` | `recruitee`, `personio` or `teamtailor` |
| `company`, `title`, `department` | Hiring company, job title and department |
| `location`, `remote` | Where the job is based, and whether it is remote |
| `employmentType` | For example `fulltime_permanent` |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Published salary range, when shown |
| `publishedAt`, `updatedAt` | Posting dates |
| `description` | Plain text, only when `includeDescriptions` is on |

## Use cases

- **Monitor hiring on many sites with one input.** Company career pages on three different ATS platforms in the same run, with the same output fields.
- **Feed a job alert or an aggregator.** Dedupe on `id` to catch only the new postings.
- **Find companies that are hiring.** Employer, location, contract type and remote policy for each posting.
- **Benchmark salaries.** Published salary ranges and currency, when the employer publishes them.

## Input

| Field | What it does |
|---|---|
| `recruiteeBoards` | Subdomains, for example `grid` from grid.recruitee.com. |
| `personioBoards` | Subdomains, for example `holidu` from holidu.jobs.personio.com. |
| `teamtailorBoards` | Subdomains, for example `career` from career.teamtailor.com. |
| `maxItems` | The run stops once this many postings are saved, across all boards combined. |
| `includeDescriptions` | Adds the full job description as plain text. |

Leave every board list empty for a productive default run across all three platforms.

Example input:

```json
{
  "recruiteeBoards": [
    "grid"
  ],
  "maxItems": 100,
  "includeDescriptions": false
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/multi-ats-jobs-2-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the boards you want (or leave them empty for the default set) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/multi-ats-jobs-2-scraper').call({
  recruiteeBoards: ['grid'],
  maxItems: 100,
  includeDescriptions: false,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

The complete file is [`examples/run.js`](examples/run.js).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/multi-ats-jobs-2-scraper").call(run_input={
    "recruiteeBoards": ["grid"],
    "maxItems": 100,
    "includeDescriptions": False,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('company')} | {item.get('platform')} | {item.get('location')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$1.20 per 1,000 postings**, plus a tiny start fee of $0.00005 per run. You pay only for the postings saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/multi-ats-jobs-2-scraper) when this guide was written; the Store page is the reference.

## Why this one

- **Three ATS platforms, one actor.** Recruitee, Personio and Teamtailor in a single run instead of three separate tools.
- **Salary kept structured** (`salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`) whenever the board publishes it, not buried in free text.
- **No browser, no login.** Each platform's own public career-site feed is read directly: Recruitee's careers API, Personio's XML job feed, Teamtailor's JSON Feed.

## Limits and honest notes

- SmartRecruiters is not covered: its API's `robots.txt` disallows every automated visitor except LinkedIn's own crawler by name, so it is off-limits here.
- Personio has no dedicated remote flag; it is inferred from an office literally named "Remote". Teamtailor infers it from a posting with no office at all. Both stay `null` when the signal is not there, rather than guessing.
- A line that carries an e-mail address, generic or named, is dropped from the description: a handful of career pages sign off with a hiring inbox.
- Fields a board does not publish, such as salary on most Personio and Teamtailor postings, stay empty.
- The dataset contains job postings only. Recruiter and candidate personal data is never exported.
- This is an unofficial tool, not affiliated with Recruitee, Personio or Teamtailor. Check the terms of each board and the data protection rules that apply to your use.

## FAQ

**Do I need an account or a proxy?**
No. The actor reads each platform's own public career-site feed directly.

**How do I find a board's subdomain?**
Open a company's career page: the subdomain is the first part of the address, for example `grid` in `grid.recruitee.com`.

**Can I get the data in Excel or Google Sheets?**
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

**Something does not work?**
Open an issue in the Issues tab. Every public actor is test-run daily and the broken ones are repaired.

## Try it

**[Open multi-ats-jobs-2-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-2-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
