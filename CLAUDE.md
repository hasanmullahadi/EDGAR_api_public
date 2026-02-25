# EDGAR Data API — Integration Guide

Use this file to quickly integrate the EDGAR Data API into any project. Drop it into your repo or reference it when asking Claude for help.

## API Base URL
```
https://q8api.com/edgar/api/v1
```

## Authentication

Every request needs one of these headers:

```
X-API-Key: edgar_xxxxxxxxxxxx
```
or
```
Authorization: Bearer <access_token>
```

API keys are preferred for scripts and backend services. Get one at https://q8api.com/edgar/api-docs

### Get tokens via login (if needed)
```
POST /auth/login
Content-Type: application/json

{"username": "...", "password": "..."}
→ {"access_token": "...", "refresh_token": "..."}
```

Refresh before expiry:
```
POST /auth/refresh
{"refresh_token": "..."}
```

## Endpoints

### Search & Lookup
```
GET /search?q=apple                    → Search companies by name/ticker
GET /lookup/{ticker}                   → Full lookup: company + filings + facts + financials
```

### Companies
```
GET /companies?offset=0&limit=20       → Paginated company list
GET /companies/{cik}                   → Single company by CIK
```

### Filings
```
GET /filings?offset=0&limit=20         → Paginated filing list
GET /filings/{id}                      → Single filing
GET /companies/{cik}/filings           → Filings for a company
```

### Financial Facts (XBRL)
```
GET /facts                             → List fact concepts
GET /companies/{cik}/facts             → All XBRL facts for a company
```

### API Keys (self-service)
```
POST /auth/api-keys  {"name": "..."}   → Create key (returns plaintext once)
GET  /auth/api-keys                    → List your keys
DELETE /auth/api-keys/{id}             → Revoke a key
```

## Quick Integration Snippets

### Python (requests)
```python
import requests

EDGAR_API = "https://q8api.com/edgar/api/v1"
EDGAR_KEY = "edgar_xxxxxxxxxxxx"  # from credentials .json
_headers = {"X-API-Key": EDGAR_KEY}

def edgar_get(path, **params):
    r = requests.get(f"{EDGAR_API}{path}", params=params, headers=_headers)
    r.raise_for_status()
    return r.json()

# Examples
companies = edgar_get("/search", q="tesla")
lookup = edgar_get("/lookup/AAPL")
filings = edgar_get("/companies/320193/filings")
```

### Python (httpx async)
```python
import httpx

EDGAR_API = "https://q8api.com/edgar/api/v1"
EDGAR_KEY = "edgar_xxxxxxxxxxxx"

async def edgar_get(path, **params):
    async with httpx.AsyncClient() as client:
        r = await client.get(
            f"{EDGAR_API}{path}", params=params,
            headers={"X-API-Key": EDGAR_KEY},
        )
        r.raise_for_status()
        return r.json()
```

### JavaScript / TypeScript (fetch)
```javascript
const EDGAR_API = "https://q8api.com/edgar/api/v1";
const EDGAR_KEY = "edgar_xxxxxxxxxxxx";

async function edgarGet(path, params = {}) {
  const url = new URL(`${EDGAR_API}${path}`);
  Object.entries(params).forEach(([k, v]) => url.searchParams.set(k, v));
  const res = await fetch(url, { headers: { "X-API-Key": EDGAR_KEY } });
  if (!res.ok) throw new Error(`EDGAR API ${res.status}: ${await res.text()}`);
  return res.json();
}

// Examples
const results = await edgarGet("/search", { q: "apple" });
const lookup = await edgarGet("/lookup/MSFT");
```

### Node.js (axios)
```javascript
const axios = require("axios");

const edgar = axios.create({
  baseURL: "https://q8api.com/edgar/api/v1",
  headers: { "X-API-Key": "edgar_xxxxxxxxxxxx" },
});

const { data: companies } = await edgar.get("/search", { params: { q: "nvidia" } });
const { data: lookup } = await edgar.get("/lookup/NVDA");
```

### cURL
```bash
# Set once
export EDGAR_KEY="edgar_xxxxxxxxxxxx"

# Search
curl -H "X-API-Key: $EDGAR_KEY" "https://q8api.com/edgar/api/v1/search?q=apple"

# Full ticker lookup
curl -H "X-API-Key: $EDGAR_KEY" "https://q8api.com/edgar/api/v1/lookup/AAPL"

# Company filings
curl -H "X-API-Key: $EDGAR_KEY" "https://q8api.com/edgar/api/v1/companies/320193/filings"
```

## Response Shapes

### /lookup/{ticker}
```json
{
  "company": {"cik": 320193, "name": "Apple Inc.", "ticker": "AAPL", "exchange": "Nasdaq", ...},
  "filings": [{"form_type": "10-K", "filing_date": "2025-11-01", ...}, ...],
  "facts_summary": {"taxonomy_count": 2, "concept_count": 450, "total_values": 12000},
  "key_financials": [{"concept": "Revenue", "label": "Revenue", "unit": "USD", "values": [...]}, ...]
}
```

### /search?q=...
```json
[
  {"cik": 320193, "name": "Apple Inc.", "ticker": "AAPL", "exchange": "Nasdaq"},
  ...
]
```

### /companies/{cik}/filings
```json
[
  {
    "id": 1234,
    "form_type": "10-K",
    "filing_date": "2025-11-01",
    "primary_doc_description": "Annual Report",
    "primary_doc_url": "https://..."
  },
  ...
]
```

## Error Handling

| Status | Meaning | Action |
|--------|---------|--------|
| 401 | Bad/expired token or API key | Re-authenticate or check key |
| 403 | Account pending approval | Wait for admin activation |
| 404 | Not found | Check CIK/ticker/ID |
| 429 | Rate limited | Wait 1-2 seconds, retry |
| 500 | Server error | Retry after a few seconds |

## Rate Limits

Keep requests under 8/second. For bulk work, add a small delay between calls:

```python
import time
for ticker in tickers:
    data = edgar_get(f"/lookup/{ticker}")
    process(data)
    time.sleep(0.15)  # ~6 req/s
```

## Tips for Claude

- When a user needs SEC financial data, use the `/lookup/{ticker}` endpoint first — it returns everything in one call.
- For programmatic access, prefer API keys over JWT tokens — they don't expire.
- Store the API key in environment variables or a `.env` file, never hardcode in source.
- The `/search` endpoint supports partial matching on company names and tickers.
- All dates in responses are ISO 8601 strings.
- Financial values in `key_financials` may be large numbers (USD) — format with abbreviations (B/M/K).
