---
name: Screen a supplier for supply chain risk
description: >-
  Search for a supply chain entity, retrieve its profile and verification
  details, and list the risk events and flags affecting it.
api: openapi/tradeverifyd-openapi-original.json
operations:
  - search_entities_search_entities_get
  - get_entity_entities__entity_id__get
  - get_entity_verification_details_entities__entity_id__verification_get
  - get_entity_risk_events_entities__entity_id__risk_events_get
generated: '2026-07-21'
method: generated
---

# Screen a supplier for supply chain risk

Every request must send the API key header `ocp-apim-subscription-key: <key>`
(see `authentication/tradeverifyd-authentication.yml`). Base URL:
`https://api.tradeverifyd.com`.

1. **Find the entity.** Call `search_entities_search_entities_get`
   (`GET /v1/search/entities`) with `name` and optionally `jurisdiction`
   (ISO country), `address`, or `hs_code`. Results are paginated with
   `page`/`page_size`. Pick the best match and keep its `entity_id`.
2. **Load the profile.** Call `get_entity_entities__entity_id__get`
   (`GET /v1/entities/{entity_id}`), optionally `include_flags=true` to get
   risk flags inline.
3. **Get verification details.** Call
   `get_entity_verification_details_entities__entity_id__verification_get`
   (`GET /v1/entities/{entity_id}/verification`) for the supplier assessment
   (Tradeverifyd Score, verification flags).
4. **List risk events.** Call
   `get_entity_risk_events_entities__entity_id__risk_events_get`
   (`GET /v1/entities/{entity_id}/risk_events`), filtering with `event_types`
   when only certain categories matter.

Error handling: `404` = unknown entity id; `422` is a FastAPI validation
error whose body lists the offending parameters (see
`errors/tradeverifyd-problem-types.yml`). All operations are read-only GETs
and safe to retry.
