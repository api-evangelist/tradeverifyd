---
generated: '2026-07-21'
method: searched
source: https://github.com/tradeverifyd/docs_examples/blob/main/api_examples/Entities%20by%20Flag%20Type-Best%20Practice%20Use%20Pattern.md
---

# **Using Flag Types & Entities by Flag Type \- Best Practice Use Pattern**

This guide shows how to:

1. Call **GET /v1/flags/types** to retrieve all flag types and subtypes.

2. Select a flag type by its `id`.

3. Call **GET /v1/entities/by-flag-type/{flag\_type\_id}** to list entities carrying that flag.

It also covers pagination, filtering, and caching.

---

## **Base Info**

* **Base URL:** `https://api.tradeverifyd.com`

* **Auth:** API key header `ocp-apim-subscription-key: <YOUR_KEY>`

* **Content-Type:** `application/json` (responses)

---

## **Step 1 — Get Flag Types**

Use this endpoint to fetch all known flag types/subtypes. Store the results locally to drive UI pickers and avoid repeated calls.

**Endpoint**  
 `GET /v1/flags/types`

**Response (trimmed example)**

```json
[
  {
    "id": "64fb1b2c...",
    "flag_type_description": "Sanctions designation",
    "flag_type": "Sanctions",
    "flag_subtype": "OFAC SDN",
    "flag_polarity": -1
  },
  {
    "id": "64fb1b2d...",
    "flag_type_description": "PEP exposure",
    "flag_type": "PEP",
    "flag_subtype": "Domestic",
    "flag_polarity": -1
  }
]
```

**Best Practices**

* **Cache** the list for 12–24 hours; invalidate on user action (e.g., refresh button) or deploys.

* **Search/Filter client-side** by `flag_type`, `flag_subtype`, or description to help users find what they need.

* **Display polarity** if relevant:  
  * `-1` → negative/high-risk indicator  
  * `0` → neutral/informational  
  * `1` → positive flag type

**cURL**

```shell
curl -s \
  -H "ocp-apim-subscription-key: $API_KEY" \
  https://api.tradeverifyd.com/v1/flags/types
```

---

## **Step 2 — Let Users Choose a Flag Type**

From the cached list, allow selection by:

* **Flag Type,**

* **Subtype**, or

* **Description**.

Map the selection back to the **`id`** field. That `id` is required by the next endpoint.

✅ Tip: Also persist the human-friendly `flag_type` \+ `flag_subtype` text so you can label results pages and audit logs.

### **API‑Only Usage (No UI)**

If you are integrating purely at the API layer and not building a user interface, you do not need to provide a picker or cache for human selection. Instead, identify the relevant `flag_type_id` from `/v1/flags/types` programmatically and pass that **id** directly into the `/v1/entities/by-flag-type/{flag_type_id}` call.

---

## **Step 3 — Get Entities by Flag Type**

**Endpoint**  
 `GET /v1/entities/by-flag-type/{flag_type_id}`

**Query Params**

* `page` (int, default 1\) — 1‑based page index

* `limit` (int, default 50\) — items per page

**Example cURL**

```shell
FLAG_TYPE_ID=64fb1b2c...
curl -s \
  -H "ocp-apim-subscription-key: $API_KEY" \
  "https://api.tradeverifyd.com/v1/entities/by-flag-type/$FLAG_TYPE_ID?page=1&limit=50"
```

---

## **Response Shape & Display**

The endpoint returns a **paginated list of entities** that carry the flag type, including:

* **Entity details**: name, jurisdiction, type, status

* **Flag information**

* **Pagination metadata** (e.g., `page`, `limit`, `total_pages`, `total_entities`)

**UI Recommendations**

* Show a **summary header**: selected flag type/subtype, total entities, last refreshed time.

* Surface **flag details** inline (e.g., subtype, sources) and link to the entity profile.

