# news-digest-agent

A conversational news-digest prototype that lets a reader choose sources and delivery preferences, summarizes scraped content, and sends email digests.

## Overview

A static React interface talks to a FastAPI backend. OpenAI parses setup messages and generates summaries, Firecrawl retrieves source content, and Resend sends email. User preferences are stored in a local JSON file.

## Features

- Conversational source addition/removal with URL confirmation.
- Delivery-time and timezone preferences.
- Source-list review and unsubscribe handling.
- AI-assisted source scraping/summarization and digest composition.
- Immediate test delivery and a scheduled-delivery endpoint.

## Architecture

The frontend posts email and messages to `/api/process`. The backend reads/writes `users.json`, parses intent with OpenAI, and updates the corresponding preferences. Digest routes scrape the configured sources, compose HTML, and call Resend. The cron handler compares the current time in each user's timezone with their configured delivery minute.

## Tech stack

Python, FastAPI, Pydantic, Uvicorn, OpenAI, Firecrawl, Resend, pytz, and browser React loaded by the static HTML page. There is no frontend bundling pipeline.

## Project structure

- `api/app.py` — API, intent parsing, preferences, scraping, and delivery.
- `index.html` — static conversational frontend.
- `requirements.txt` — backend dependency versions.
- `vercel.json` — static/Python routing and cron configuration.
- `DEPLOY_INSTRUCTIONS.md` — additional deployment notes.

## Run locally

Create and activate a Python virtual environment:

```bash
git clone https://github.com/anishkganesh/news-automation-agent.git
cd news-automation-agent
python -m venv .venv
pip install -r requirements.txt
```

Create a root `.env`:

```dotenv
OPENAI_API_KEY=your-openai-key
FIRECRAWL_API_KEY=your-firecrawl-key
RESEND_API_KEY=your-resend-key
```

Start the backend from the repository root:

```bash
python api/app.py
```

It listens on port 8000. In another terminal, serve the root page:

```bash
python -m http.server 8001
```

Open `http://localhost:8001`. The page detects `localhost` and calls `http://localhost:8000/api`; use this hostname for the documented local setup.

## Configuration and data

`users.json` is created in the current working directory and contains email, source, timezone, and delivery settings. It is not a database and has no concurrent-write coordination.

OpenAI calls request `gpt-3.5-turbo` in this snapshot. Confirm model/provider compatibility with the committed client versions before use. Resend currently uses its onboarding/test sender; public delivery needs an appropriately configured sender/domain and provider permissions.

The Vercel cron expression is `* * * * *` — **every minute**, not hourly. The endpoint sends only when a user's local hour/minute matches the preference. There is no durable delivery ledger or deduplication, so repeated requests in that minute can send duplicates.

## Usage

Enter an email, add source URLs or names, confirm the URL, and set time/timezone preferences. Review sources or unsubscribe through the conversational interface. Test-digest actions send actual email; use an address you control.

API routes are `POST /api/process`, `GET /api/test-digest/{email}`, and `GET /api/cron/send-digests`. The backend root returns an API welcome response.

## Validation

No automated tests are configured; `npm test` is only the default failing placeholder. Verify preference updates, URL confirmation, timezone matching, provider errors, and one controlled delivery. This documentation review did not send email or run paid API calls.

## Deployment

`vercel.json` maps API requests to the Python app and other requests to the HTML page. Configure service keys and a scheduler allowed by the chosen hosting plan. Local `users.json` persistence is unsuitable as shared durable storage in a serverless deployment; production requires an external store and delivery coordination.

## Limitations

- There is no complete authentication boundary for preference updates or delivery routes.
- Public delivery/cron endpoints need access controls before exposure.
- Filesystem persistence and exact-minute scheduling are fragile in multi-instance environments.
- Scraping can fail, and generated summaries can be incorrect.
- Email failures are logged inside the sender function, so a route response does not guarantee inbox delivery.

## Attribution and license

`package.json` contains an ISC label, but no standalone license file is included. Provider services and dependencies retain their own terms.
