# EDGAR Data API

Public REST API for SEC EDGAR data — company filings, XBRL financial facts, and full-text search.

**Base URL:** `https://q8api.com/edgar/api/v1`

## Getting Started

### 1. Create an Account

Register at [q8api.com/edgar/register](https://q8api.com/edgar/register). Your account will be activated by an admin.

### 2. Get Your API Key

Once approved, log in at [q8api.com/edgar/api-docs](https://q8api.com/edgar/api-docs) and generate an API key. Download your credentials `.json` file for easy reference.

### 3. Make API Calls

```bash
# Using API key (recommended for scripts)
curl -H "X-API-Key: edgar_xxxxxxxxxxxx" \
  "https://q8api.com/edgar/api/v1/companies"

# Using Bearer token (from login)
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://q8api.com/edgar/api/v1/companies"
```

## Authentication

### Login (get tokens)

```bash
curl -X POST https://q8api.com/edgar/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "your_username", "password": "your_password"}'
```

Response:
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

Access tokens expire after 30 minutes. Use the refresh token to get a new pair:

```bash
curl -X POST https://q8api.com/edgar/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refresh_token": "eyJ..."}'
```

### API Keys (recommended)

API keys don't expire and are simpler to use. Create one via the API or the web dashboard:

```bash
curl -X POST https://q8api.com/edgar/api/v1/auth/api-keys \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-app"}'
```

Use the key via the `X-API-Key` header:
```bash
curl -H "X-API-Key: edgar_xxxxxxxxxxxx" \
  "https://q8api.com/edgar/api/v1/search?q=apple"
```

## API Endpoints

### Companies

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/companies` | List companies (paginated) |
| `GET` | `/companies/{cik}` | Get company by CIK number |
| `GET` | `/search?q=apple` | Search by company name or ticker |

### Filings

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/filings` | List filings (paginated, filterable) |
| `GET` | `/filings/{id}` | Get filing details |
| `GET` | `/companies/{cik}/filings` | List filings for a specific company |

### Financial Facts (XBRL)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/facts` | List available fact concepts |
| `GET` | `/companies/{cik}/facts` | Get all XBRL facts for a company |

### Lookup

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/lookup/{ticker}` | Full company lookup — returns company info, recent filings, facts summary, and key financials |

## Examples

### Python

```python
import requests

API = "https://q8api.com/edgar/api/v1"
headers = {"X-API-Key": "edgar_xxxxxxxxxxxx"}

# Search for a company
resp = requests.get(f"{API}/search", params={"q": "tesla"}, headers=headers)
companies = resp.json()
print(companies)

# Get company filings
cik = companies[0]["cik"]
resp = requests.get(f"{API}/companies/{cik}/filings", headers=headers)
filings = resp.json()

# Full ticker lookup (company + filings + financials)
resp = requests.get(f"{API}/lookup/AAPL", headers=headers)
data = resp.json()
print(f"Company: {data['company']['name']}")
print(f"Recent filings: {len(data['filings'])}")
print(f"XBRL concepts: {data['facts_summary']['concept_count']}")
```

### JavaScript (fetch)

```javascript
const API = "https://q8api.com/edgar/api/v1";
const headers = { "X-API-Key": "edgar_xxxxxxxxxxxx" };

// Search
const resp = await fetch(`${API}/search?q=microsoft`, { headers });
const companies = await resp.json();

// Lookup
const lookup = await fetch(`${API}/lookup/MSFT`, { headers });
const data = await lookup.json();
console.log(data.company.name, data.filings.length, "filings");
```

### cURL

```bash
# Search companies
curl -H "X-API-Key: edgar_xxxxxxxxxxxx" \
  "https://q8api.com/edgar/api/v1/search?q=apple"

# Get Apple filings
curl -H "X-API-Key: edgar_xxxxxxxxxxxx" \
  "https://q8api.com/edgar/api/v1/lookup/AAPL"

# List all companies (paginated)
curl -H "X-API-Key: edgar_xxxxxxxxxxxx" \
  "https://q8api.com/edgar/api/v1/companies?offset=0&limit=20"
```

## Response Format

All responses are JSON. Dates use ISO 8601 format. Example company response:

```json
{
  "cik": 320193,
  "name": "Apple Inc.",
  "ticker": "AAPL",
  "exchange": "Nasdaq",
  "sic_code": "3571",
  "entity_type": "operating",
  "state_of_incorporation": "CA",
  "fiscal_year_end": "0930"
}
```

## Rate Limits

The API is rate-limited. If you receive a `429` response, wait and retry. For bulk operations, keep requests under 8 per second.

## Errors

| Status | Meaning |
|--------|---------|
| `401` | Not authenticated — check your token or API key |
| `403` | Account pending approval or insufficient permissions |
| `404` | Resource not found |
| `409` | Conflict (e.g., duplicate username on register) |
| `429` | Rate limited — slow down |
| `500` | Server error — try again later |

## Interactive Tools

- **API Documentation & Credentials**: [q8api.com/edgar/api-docs](https://q8api.com/edgar/api-docs)
- **Data Lookup (visual)**: [q8api.com/edgar/test](https://q8api.com/edgar/test)
- **Swagger UI**: [q8api.com/edgar/docs](https://q8api.com/edgar/docs)

## Support

For questions or issues, open an issue on this repository.

---

Built by [Kuwait Data Systems](https://kds.software)
