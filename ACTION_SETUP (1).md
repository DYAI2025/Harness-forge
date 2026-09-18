# ETBZ Runtime Action Setup

1. Deploy a real HTTPS service implementing `openapi.etbz-control-plane.yaml`.
2. Replace `https://YOUR_ETBZ_CONTROL_PLANE_HOST` with the deployed public HTTPS base URL.
3. Import the OpenAPI schema into the Custom GPT Actions configuration.
4. Configure Bearer authentication in the Builder UI using a dedicated control-plane token.
5. The control plane, not the GPT, owns internal SSH, FuFirE API key, Etsy OAuth, Telegram and Hermes execution credentials.
6. Enforce server-side policy: no arbitrary shell endpoint, no raw secret-returning endpoint, no self-approval, no unaudited paid/production operation.
7. Log every accepted mutation job with requester, approval, exact target state, command/action set, result, tests and evidence reference.

The OpenAPI file is a **contract/design artifact** until the API exists. Uploading the schema alone does not create the backend.
