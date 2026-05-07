# 📦 SSCC API

**Free REST API for SSCC validation, generation and range expansion. Built on Cloudflare Workers.**

🔗 **Base URL**: `https://sscc.birth1mark.workers.dev`

🔗 **Live app**: [birth1mark.github.io/sscc-check](https://birth1mark.github.io/sscc-check/)

🔗 **Guide**: [birth1mark.github.io/sscc-check/guide.html](https://birth1mark.github.io/sscc-check/sscc-api-guide.html)

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-orange)
![Free](https://img.shields.io/badge/free-100%25-brightgreen)

---

## Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/validate?sscc=<18 digits>` | Validate an 18-digit SSCC |
| `GET` | `/generate?body=<17 digits>` | Generate check digit for a 17-digit body |
| `GET` | `/range?from=<SSCC>&to=<SSCC>` | Generate all SSCCs between two codes |
| `GET` | `/prefix?sscc=<18 digits>` | Get GS1 country info from SSCC prefix |
| `POST` | `/extract` | Extract and validate all SSCCs from file content |
| `GET` | `/health` | API status and endpoint list |

---

## Examples

### Validate an SSCC

```bash
curl "https://sscc.birth1mark.workers.dev/validate?sscc=356012345600000016"
```

```json
{
  "input": "356012345600000016",
  "sscc": "356012345600000016",
  "formatted": "(00)356012345600000016",
  "valid": true,
  "checkDigit": { "provided": 6, "expected": 6 },
  "gs1": {
    "extensionDigit": 3,
    "prefix": 560,
    "country": "Portugal",
    "countryCode": "PT",
    "flag": "🇵🇹"
  }
}
```

### Generate check digit

```bash
curl "https://sscc.birth1mark.workers.dev/generate?body=35601234560000001"
```

```json
{
  "body": "35601234560000001",
  "checkDigit": 6,
  "sscc": "356012345600000016",
  "formatted": "(00)356012345600000016"
}
```

### Range expansion

```bash
curl "https://sscc.birth1mark.workers.dev/range?from=35601234560000001&to=35601234560000005"
```

```json
{
  "from": "35601234560000001",
  "to": "35601234560000005",
  "count": 5,
  "items": [
    { "sscc": "356012345600000016", "formatted": "(00)356012345600000016", "checkDigit": 6 },
    ...
  ]
}
```

### Extract SSCCs from file

```bash
curl -X POST "https://sscc.birth1mark.workers.dev/extract" \
  -H "Content-Type: text/plain" \
  --data-binary @your-file.edi
```

```json
{
  "format": "edifact",
  "count": 3,
  "summary": { "valid": 2, "invalid": 1, "generated": 0 },
  "items": [...]
}
```

---

## JavaScript

```javascript
// Validate
const res = await fetch('https://sscc.birth1mark.workers.dev/validate?sscc=356012345600000016');
const data = await res.json();
console.log(data.valid); // true

// Generate
const res2 = await fetch('https://sscc.birth1mark.workers.dev/generate?body=35601234560000001');
const data2 = await res2.json();
console.log(data2.sscc); // 356012345600000016

// Extract from file
const content = await file.text();
const res3 = await fetch('https://sscc.birth1mark.workers.dev/extract', {
    method: 'POST',
    body: content,
});
const data3 = await res3.json();
console.log(data3.items);
```

## Python

```python
import requests

# Validate
r = requests.get('https://sscc.birth1mark.workers.dev/validate', params={'sscc': '356012345600000016'})
print(r.json()['valid'])  # True

# Generate
r = requests.get('https://sscc.birth1mark.workers.dev/generate', params={'body': '35601234560000001'})
print(r.json()['sscc'])  # 356012345600000016

# Extract from file
with open('desadv.edi', 'r') as f:
    content = f.read()
r = requests.post('https://sscc.birth1mark.workers.dev/extract', data=content)
print(r.json()['count'])
```

---

## Notes

- **CORS**: All endpoints return `Access-Control-Allow-Origin: *` — safe to call from any browser
- **Rate limit**: 100,000 requests/day (Cloudflare Workers free tier)
- **Max range**: 500 codes per `/range` request
- **No auth required**: fully open and free
- **No data stored**: stateless — nothing is logged or retained

---

## Tech Stack

- **[Cloudflare Workers](https://workers.cloudflare.com/)** — edge runtime, zero cold starts
- **GS1 General Specifications 2025** — check digit algorithm and prefix table
- Vanilla JS — no dependencies

---

## License

MIT — free to use, modify and distribute.
