# FinGuard

**Winner: Best Capital One Hack (Capital Finance Track), Bitcamp 2025**

FinGuard is a Chrome extension plus web dashboard that watches out for you while you shop online. As you browse, it checks every site you land on for phishing, malware, and scam signals, and warns you in plain English before you type in a card number. At checkout it tells you which of your credit cards earns the most on that purchase, and the dashboard ties it all together with spending analytics, fraud monitoring, and an AI assistant that answers questions about your own transactions.

Built by a team of four at [Bitcamp 2025](https://bitca.mp/), the University of Maryland's hackathon.

## What it does

- **Real-time scam and phishing warnings.** The extension's background worker watches every page load and asks the FinGuard server whether the site is risky. The server runs the URL through IPQS (IP Quality Score) reputation checks for phishing, malware, spamming, and suspicious signals, and uses Gemini to classify gambling sites. If anything comes back positive, Gemini writes a short, non-technical "nudge" that slides into the page as a popup: no jargon, just why you should be careful.
- **Best-card recommendations.** Tell the extension what you're buying (or let Gemini categorize the merchant for you) and it compares reward rates and active offers across your linked cards to pick the one that earns the most, with one-click autofill of the card details at checkout.
- **Fraud monitoring on the dashboard.** The backend screens transactions against known phishing domains, suspicious URL patterns, and high-risk geographies, and supports flagging, approval flows, and per-card security settings.
- **Spending analytics.** Category breakdowns, merchant analysis, spending trends, budget tracking, and reward-optimization views, powered by real transaction data pulled through Plaid and the Capital One Nessie API.
- **An AI financial assistant.** A chatbot on the analytics screen that receives your recent transactions, cards, top categories, and top merchants as context, so it can answer questions like "where did most of my money go this month?" or "does anything here look fraudulent?"

## How it's wired together

The repo has four moving parts:

```
Chrome Extension (React + Vite, Manifest V3)
      │  page URLs, card/category requests
      ▼
Extension Server (Express, :3000) ──► Nudge engine ──► IPQS + Gemini 1.5 Flash
      │                                                (scam check + nudge text)
      ▼
MongoDB Atlas  (card catalog, reward rates, offers, transactions)
      ▲
      │  populated by cardServer.py (Selenium + BeautifulSoup scraper)

Dashboard (React + Tailwind)
      │
      ▼
Backend API (Express, :5000) ──► Plaid (bank linking) ─► Supabase (accounts, txns)
                             ──► Capital One Nessie API (banking sandbox data)
                             ──► Gemini 2.0 Flash (chatbot + AI insights)
                             ──► Phishing protection service (rule-based screening)
```

| Directory | What it is |
|---|---|
| `Chrome Extension/` | Manifest V3 extension. React 19 popup (card picker, autofill, Gemini merchant categorization), background service worker that requests nudges on every page load, content script that renders the warning popup in-page. |
| `Server/` | Express server the extension talks to. Best-card lookup against MongoDB, card details for autofill, transaction logging, and the `/generateNudge` endpoint. |
| `Nudge/` | The safety-nudge engine (JS and Python versions). IPQS URL reputation plus Gemini 1.5 Flash for gambling classification and for writing the user-facing warning. |
| `Backend/` | Express API for the dashboard. Plaid integration, Supabase persistence, Nessie API routes, Gemini-powered chatbot and insights, card/transaction security routes. |
| `Dashboard/` | React web app: analytics, card management, Plaid linking, payments, rewards, and security screens. |
| `cardServer.py` | Python scraper (Selenium + BeautifulSoup) that pulls credit-card reward rates and offers from issuer sites into MongoDB Atlas. |

## Tech stack

- **Frontend:** React 19, Tailwind CSS, Chart.js, Framer Motion; Vite for the extension build, Chrome Manifest V3
- **Backend:** Node.js + Express (two services), Python for the scraper and the alternate nudge implementation
- **AI:** Google Gemini (1.5 Flash for site classification and nudge generation, 2.0 Flash for the financial chatbot, spending insights, and merchant categorization)
- **Data:** MongoDB Atlas (card catalog, rewards, offers, transactions), Supabase/Postgres (users, Plaid items, accounts, synced transactions)
- **External APIs:** IPQS (URL reputation), Plaid (bank account linking), Capital One Nessie API (banking sandbox)

## Running it locally

You'll need Node 18+, Python 3.11+ (only for the scraper), and API keys for Gemini and IPQS, plus optionally Plaid and Nessie.

**1. Extension server** (powers the extension: nudges + card recommendations)

```bash
cd Server
npm install
# copy .env.example to .env and fill in your Gemini and IPQS keys
node server.js        # runs on :3000
```

**2. Chrome extension**

```bash
cd "Chrome Extension"
npm install
npm run build
```

Then open `chrome://extensions`, enable Developer mode, click "Load unpacked," and select the `Chrome Extension/dist` folder.

**3. Dashboard backend**

```bash
cd Backend
npm install
npm run dev           # runs on :5000
```

Environment variables it reads: `GEMINI_API_KEY`, `PLAID_CLIENT_ID`, `PLAID_SECRET`, `PLAID_ENV`, `NESSIE_API_KEY`, `MONGODB_URI`, `MONGODB_DB`.

**4. Dashboard**

```bash
cd Dashboard
npm install
npm start
```

Set `REACT_APP_API_URL`, `REACT_APP_SUPABASE_URL`, `REACT_APP_SUPABASE_ANON_KEY`, and `REACT_APP_PLAID_ENV` in `Dashboard/.env`.

**5. (Optional) Card rewards scraper**

```bash
pip install pymongo selenium beautifulsoup4
python cardServer.py
```

Requires ChromeDriver and a MongoDB Atlas connection string.

Note: this is a hackathon build. Some demo data, test card numbers, and shortcuts are baked in, and the two servers expect to run on localhost.

## Team

FinGuard was built at Bitcamp 2025 by:

- [Ishaan Narang](https://github.com/Ishaannarang22)
- [Mantavya](https://github.com/mantavya0807)
- [Manchi0](https://github.com/Manchi0)
- [Shreyansh](https://github.com/Nailfighter)

Original repository: [mantavya0807/FinGuard](https://github.com/mantavya0807/FinGuard)
