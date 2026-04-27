# Real Estate APIs — Zillow, Redfin, Airbnb

| Endpoint | Returns |
|---|---|
| `/scrape/zillow/listing` | Search results by area + filters |
| `/scrape/zillow/property` | Single home (history, agent, schools, taxes) |
| `/scrape/redfin/listing` | Redfin search results |
| `/scrape/redfin/property` | Single Redfin home |
| `/scrape/airbnb/listing` | Search results |
| `/scrape/airbnb/property` | Single Airbnb listing |

All synchronous `GET`.

## Zillow Listing

Filter params use **bracketed** keys (`price[min]`, `beds[max]`).

```python
import requests

def zillow_search(keyword, listing_type="forSale", **filters):
    r = requests.get(
        "https://api.hasdata.com/scrape/zillow/listing",
        headers={"x-api-key": API_KEY},
        params={"keyword": keyword, "type": listing_type, **filters},
        timeout=300,
    )
    return r.json()

zillow_search("Brooklyn, NY", price={"min": 800000, "max": 2000000})
zillow_search("33321", "sold", daysOnZillow="6m")  # recent comps
```

`requests` + `axios` serialize nested dicts as `price[min]=…&price[max]=…` automatically. With raw `URLSearchParams`, build the bracketed keys yourself.

| Param | Notes |
|---|---|
| `keyword` | **Required.** Area string ("New York, NY", zip, neighborhood). |
| `type` | **Required.** `forSale`, `forRent`, `sold`. |
| `price[min/max]`, `beds[min/max]`, `baths[min/max]`, `sqft[min/max]` | Range filters. |
| `daysOnZillow` | `24h`, `7d`, `14d`, `30d`, `90d`, `6m`, `12m`. |
| `page` | Pagination. |

Response: `requestMetadata`, `searchInformation`, **`properties`** (the listings array — not `listings`), `pagination`.

## Zillow Property

```python
requests.get(
    "https://api.hasdata.com/scrape/zillow/property",
    headers={"x-api-key": API_KEY},
    params={"url": url, "extractAgentEmails": "true"},
    timeout=300,
)
```

Takes a full Zillow URL (not zpid). Returns address, lot/sqft/beds/baths, price + tax history, schools, agent block, photos. Agent emails are best-effort.

## Redfin

```python
# Listing
params = {"keyword": "33321", "type": "forSale", "page": 1}
# Property
params = {"url": "https://www.redfin.com/FL/Tamarac/9...html"}
```

Same bracketed `price[min]`, `beds[min]`, etc. as Zillow. Zip codes work best for `keyword`.

## Airbnb

```python
def airbnb_search(location, check_in, check_out, **kwargs):
    return requests.get(
        "https://api.hasdata.com/scrape/airbnb/listing",
        headers={"x-api-key": API_KEY},
        params={"location": location, "checkIn": check_in, "checkOut": check_out, **kwargs},
        timeout=300,
    ).json()
```

| Param | Notes |
|---|---|
| `location` | **Required.** Free-form. |
| `checkIn` | **Required.** `YYYY-MM-DD`. |
| `checkOut`, `adults`, `children`, `infants`, `pets` | Optional. |
| `nextPageToken` | Pagination cursor. |

### Token pagination

```python
def airbnb_all(location, check_in, check_out):
    out, token = [], None
    while True:
        page = airbnb_search(location, check_in, check_out,
                             **({"nextPageToken": token} if token else {}))
        out.extend(page.get("listings", []))
        token = page.get("nextPageToken")
        if not token:
            return out
```

## Patterns

### Sold comps for ROI

```python
sold = zillow_search(zip_code, "sold", daysOnZillow="6m").get("properties", [])
ppsf = [(l["price"] / l["livingArea"]) for l in sold if l.get("livingArea")]
```

### STR yield estimate

```python
rentals = airbnb_search(area, ci, co).get("listings", [])           # Airbnb → "listings"
homes   = zillow_search(area, "forSale").get("properties", [])      # Zillow → "properties"
night   = sum(r.get("price", 0) for r in rentals) / max(len(rentals), 1)
price   = sum(h.get("price", 0) for h in homes)   / max(len(homes), 1)
yield_  = (night * 365 * 0.65) / price if price else None  # 65% occupancy
```

## Gotchas

- **Bracketed query keys** — work with `requests`/`axios`, not raw `URLSearchParams`.
- **`type=sold` + `daysOnZillow` = comps recipe.** Without `daysOnZillow`, history is unbounded.
- **Airbnb requires `checkIn`** and uses **token** pagination — store `nextPageToken`, not page numbers.
- **Property endpoints take URLs**, not IDs.
- **Agent emails are best-effort.**
