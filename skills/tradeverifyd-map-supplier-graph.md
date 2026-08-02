---
name: Map a multi-tier supplier graph and its flagged relationships
description: >-
  Resolve an entity, retrieve its multi-tier supplier relationship graph,
  and isolate the flagged (risk-bearing) relationships.
api: openapi/tradeverifyd-openapi-original.json
operations:
  - resolve_entity_resolve_entity_get
  - get_entity_graph_entities__entity_id__graph_get
  - get_entity_graph_flagged_relationships_entities__entity_id__graph_flagged_relationships_get
  - get_relationship_types_detailed_relationship_types_get
generated: '2026-07-21'
method: generated
---

# Map a multi-tier supplier graph

Auth: `ocp-apim-subscription-key` header on every call. Base URL:
`https://api.tradeverifyd.com`.

1. **Resolve the entity.** If you hold an external identifier, call
   `resolve_entity_resolve_entity_get` (`GET /v1/resolve/entity`) to map it
   to a Tradeverifyd `entity_id`; otherwise search first.
2. **Fetch relationship vocabulary once.** Call
   `get_relationship_types_detailed_relationship_types_get`
   (`GET /v1/relationship-types`) and cache it — the graph edges reference
   these type ids.
3. **Retrieve the graph.** Call
   `get_entity_graph_entities__entity_id__graph_get`
   (`GET /v1/entities/{entity_id}/graph`) with `depth` to control how many
   supplier tiers to traverse and `direction` for upstream/downstream;
   `flagged_only=true` trims to risk-bearing paths.
4. **Isolate flagged relationships.** Call
   `get_entity_graph_flagged_relationships_entities__entity_id__graph_flagged_relationships_get`
   (`GET /v1/entities/{entity_id}/graph/flagged_relationships`) to list only
   edges carrying risk flags — chokepoints and sanctioned/flagged links.

Deep graphs are expensive: start with a small `depth` and increase only when
needed. All operations are idempotent GETs.
