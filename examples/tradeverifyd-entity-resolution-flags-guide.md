---
generated: '2026-07-21'
method: searched
source: https://github.com/tradeverifyd/docs_examples/blob/main/api_examples/Entity%20Resolution%20Flags%20Retrieval-Best%20Practices%20Use%20Pattern.md
---

# **Entity Resolution and Flags Retrieval – Best Practice Use Pattern**

## **Introduction**

This document outlines the recommended use pattern for retrieving flags associated with entities via Tradeverifyd’s APIs. It addresses current limitations in the `GET /v1/flags` endpoint when using **entity name** and **jurisdiction** parameters, and provides guidance for ensuring accurate and consistent results.

---

## **Background**

The `GET /v1/flags` endpoint supports retrieving flags associated with an entity. Flags represent compliance issues, risk indicators, or other important alerts.

The endpoint accepts either:

* `entity_id`, **or**

* Both `entity_name` and `entity_jurisdiction`.

However, using `entity_name` and `entity_jurisdiction` currently relies on legacy fuzzy matching logic. This may occasionally result in **missing or unexpected flag results**, particularly when querying well-known entities subject to UFLPA or other sanctions/regulations.

---

## **Recommended Use Pattern**

To ensure reliable and accurate retrieval of flags, we recommend the following workflow:

1. **Entity Resolution**  
    Call the `GET /v1/resolve/entity` endpoint with parameters such as `name`, `address`, `jurisdiction`, or `entity_id`.  
    → This resolves to the top-scoring, validated entity record.

2. **Retrieve Entity ID**  
    From the resolution response, capture the `entity_id` of the matched entity.

3. **Get Flags by Entity ID**  
    Pass the `entity_id` into the `GET /v1/flags` endpoint to retrieve all associated flags.  
    → This guarantees alignment with the exact entity record, avoiding fuzzy-match errors.

---

## **Example Workflow**

### **Step 1 – Resolve Entity**

**Request:**

```shell
curl -X GET "https://api.tradeverifyd.com/v1/resolve/entity?name=Example%20Corp&jurisdiction=US" \
  -H "ocp-apim-subscription-key: <your_api_key>"

```

**Sample Response (Single Entity search result):**

```json
[
  {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Example Corp",
    "jurisdiction": "US",
    "addresses": [
      "123 Main Street, New York, NY"
    ],
    "flagged": false,
    "description": "Sample description",
    "primary_role": "Manufacturer",
    "url": "https://example.com",
    "extended_data": {},
    "auto_research": false,
    "monitor": false,
    "callback_url": "https://webhook.example.com",
    "references": {
      "flags": { "total": 0 },
      "relationships": {
        "total": 0,
        "from_relationships": { "count": 0 },
        "to_relationships": { "count": 0 }
      },
      "risk_events": { "total": 0 }
    },
    "external_ids": [
      {
        "id": "US123456",
        "type": "LEI",
        "source": "GLEIF"
      }
    ]
  }
]

```

### **Step 2 – Retrieve Flags by Entity ID**

**Request:**

```shell
curl -X GET "https://api.tradeverifyd.com/v1/flags?entity_id=123e4567-e89b-12d3-a456-426614174000" \
  -H "ocp-apim-subscription-key: <your_api_key>"

```

**Sample Response (Flags list):**

```json
[
  {
    "_id": "abc123",
    "entity_id": "123e4567-e89b-12d3-a456-426614174000",
    "created_at": "2025-01-01T00:00:00.000Z",
    "time_start": "2024-12-01T00:00:00.000Z",
    "time_end": "2025-12-01T00:00:00.000Z",
    "type": "UFLPA",
    "description": "Entity is listed as restricted under UFLPA",
    "sources": [
      {
        "id": "src-1",
        "url": "https://source.example.com/uflpa",
        "title": "UFLPA Entity List",
        "full_text": "Entity confirmed as restricted",
        "signature": {
          "property1": "hash1",
          "property2": "hash2"
        }
      }
    ],
    "status": {
      "state": "suspected",
      "confidence": "high",
      "sources": [
        {
          "id": "src-1",
          "url": "https://source.example.com/uflpa",
          "title": "UFLPA Entity List"
        }
      ]
    },
    "entity_name": "Example Corp",
    "entity_jurisdiction": "US",
    "flag_type": "sanction",
    "flag_subtype": "uflpa",
    "depth": null
  }
]
```

---

## **Frequently Asked Questions (FAQ)**

**Q: Why should I not rely on `entity_name` and `entity_jurisdiction` for flag retrieval?**  
 A: These parameters use older fuzzy matching logic, which can return incomplete or unexpected results. Using `entity_id` ensures precision.

**Q: Will this workflow change in the future?**  
 A: Yes. As part of our API enhancements, we are improving the `GET /v1/flags` endpoint to better align with entity resolution. In the meantime, using entity resolution followed by `entity_id`\-based flag retrieval is the recommended approach.

**Q: Does this approach support recursive flag retrieval?**  
 A: Yes. You may still use the `recursive=true` and `depth` parameters with `entity_id` to retrieve flags from upstream related entities.

---

## **Closing Note**

This workflow ensures that customers obtain **complete and accurate flag data** tied to the correct entity. Tradeverifyd is actively enhancing the flags service so that name/jurisdiction matching will be as reliable as entity resolution in the future.

