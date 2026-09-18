# ETBZ Access Setup Checklist

## A. Atlassian — required
Preferred: connect the workspace's native Atlassian/Rovo integration.

Target:
- site `https://dyai2026.atlassian.net`
- Confluence space `ETBZ`
- Jira project `ETBZ`

Required effective capabilities:
- read Confluence pages/search;
- create/update pages/comments when explicitly authorized;
- read Jira issues/JQL;
- create/edit/comment/transition Jira issues when explicitly authorized.

The GPT must read after write.

## B. GitHub — required
ETBZ currently spans multiple candidate repositories rather than one proven monorepo. Connect the GitHub account/org with least privilege for read/review. Before each code-bearing Jira slice, resolve the exact owning repository/repositories and starting SHA. Writes/PR/merge follow explicit project authorization and evidence gates. See `knowledge_upload/16_REPOSITORY_DOMAIN_MAP.md`.

## C. ETBZ Runtime Control Plane — required for VPS/Hermes
Deploy an HTTPS API implementing `actions/openapi.etbz-control-plane.yaml`.
- The API runs server-side with controlled access to the VPS/internal FuFirE/Etsy/Hermes surfaces.
- GPT receives a separate bearer credential for the control plane, stored only in Builder Action auth.
- Backend secrets never transit through GPT responses.

## D. Etsy
Store Etsy OAuth credentials server-side, not in GPT Knowledge. Expose only sanitized connection state and allowed order operations through the control plane. Current OAuth/shop binding is unverified.

## E. FuFirE
Keep `X-API-Key` server-side. Expose contract/status/smoke operations through the control plane. Do not put FuFirE keys into GPT Actions as query/body parameters.

## F. Telegram Human Gate
GPT may create an approval request and read approval state. It must not mint an approval itself. Telegram bot credentials remain server-side.

## G. Hermes
GPT submits bounded jobs. Hermes does not receive unrestricted secrets unless required by a server-side execution context and approved policy. GPT receives sanitized result/evidence candidates.
