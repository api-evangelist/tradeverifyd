---
name: Create a shipment, issue verifiable credentials, and submit to US CBP
description: >-
  The customs flow the provider's own Postman collection walks through -
  create a shipment, upload supporting documents, issue W3C Verifiable
  Credentials, and submit a verifiable presentation to US Customs (CBP).
api: openapi/tradeverifyd-openapi-original.json
operations:
  - create_shipment_shipments_post
  - upload_shipment_file_shipment__shipment_id__upload_post
  - issue_vc_credentials_issue_post
  - get_shipment_verifiable_credentials_credentials_list_shipment__shipment_id__get
  - submit_verifiable_presentation_us_customs_submit_presentation_post
  - get_cbp_requests_for_shipment_shipments__shipment_id__cbps_get
generated: '2026-07-21'
method: generated
---

# Shipment → Verifiable Credentials → CBP submission

Auth: `ocp-apim-subscription-key` header. Base URL:
`https://api.tradeverifyd.com`. The provider's public Postman collection
(`postman/tradeverifyd-shipments-cbp-postman-collection.json`) mirrors this
flow with environment variables `api-root-v1` and `api-key`.

1. **Create the shipment.** `create_shipment_shipments_post`
   (`POST /v1/shipments`) with `shipment_name`, `status` (e.g. "Created"),
   and `organizational_units`. Keep the returned `shipment_id`.
2. **Upload supporting documents.**
   `upload_shipment_file_shipment__shipment_id__upload_post`
   (`POST /v1/shipment/{shipment_id}/upload`) — multipart file upload.
3. **Issue credentials.** `issue_vc_credentials_issue_post`
   (`POST /v1/credentials/issue`) issues a W3C Verifiable Credential against
   a registered schema (list schemas via `/v1/credentials/schemas`).
4. **Collect the shipment's credentials.**
   `get_shipment_verifiable_credentials_credentials_list_shipment__shipment_id__get`
   (`GET /v1/credentials/list/shipment/{shipment_id}`) — note the VC ids to
   present.
5. **Submit to CBP.**
   `submit_verifiable_presentation_us_customs_submit_presentation_post`
   (`POST /v1/us_customs/submit_presentation`) with the verifiable
   presentation for the shipment.
6. **Track CBP requests.**
   `get_cbp_requests_for_shipment_shipments__shipment_id__cbps_get`
   (`GET /v1/shipments/{shipment_id}/cbps`) to monitor CBP's responses.

The POST operations are not idempotent (no Idempotency-Key contract — see
`conventions/tradeverifyd-conventions.yml`): on a timeout, list before
retrying to avoid duplicate shipments or credentials.
