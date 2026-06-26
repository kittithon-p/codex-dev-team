---
name: nayoo-service-map
description: Use when wiring or debugging ANY cross-service call, BFF gateway, .env/PORT config, or DocumentDB connection in the NaYoo Go backend. The canonical map of service ports, GO_SYSTEM_*_URL discovery, DocumentDB connection options, and go-common-response error-code zones. Trigger on "connection refused", new gateway route, env-var wiring, or "which port is service X".
---

# NaYoo backend service map

Canonical reference for how the Go services connect. Verify a specific value against the service's `env/.env.example` if unsure.

## Ports (local dev; from each service's `PORT` env)
| Service | Port |
|---|---|
| BFF go-process-nayoo-service | 8080 |
| BFF go-process-backoffice-service | 8088 |
| go-system-authentication-service | 8081 |
| go-system-integration-service | 8082 |
| go-system-product-service | 8083 |
| go-system-listing-service | 8084 |
| go-system-media-service | 8085 |
| go-system-notification-service | 8086 |
| go-system-report-service | 8087 |
| go-worker-product-service | 8089 |
| go-worker-listing-service | 8090 |
| metrics/health (EVERY service) | 9090 |

Frontends: console 3000 / backoffice 3001 / nayoo 3002 (Makefile) — but package.json defaults differ (nayoo 3000, console 3001, backoffice 3030). Verify the launch path.

**EXPOSE gotcha:** every Dockerfile hardcodes `EXPOSE 8080` + `HEALTHCHECK :8080`, but the app binds the `PORT` env (8081-8090). `PORT`/`GO_SYSTEM_*_URL` is the source of truth, not `EXPOSE`.

## Service discovery
Services call each other via `GO_SYSTEM_<NAME>_URL` env vars; `restclient` builds the name as `fmt.Sprintf("GO_SYSTEM_%s_URL", upper)`. Local = `http://localhost:<port>`, no API key server-to-server.

## DocumentDB connection (mandatory options)
The DB is **AWS DocumentDB, not MongoDB**. Every connection string needs ALL of: `tls=true` (CA `global-bundle.pem` via `MONGO_TLS_CA_FILE`), `replicaSet=rs0`, `readPreference=secondaryPreferred`, `retryWrites=false`, `directConnection=true`. Non-prod host `43.209.100.87:27017`. auth/listing/report open a second DB (`MONGO_BO_DB` = `bo-develop`). Use `go-common-db` `TracedDatabase`, never raw collections. No `$facet`/`$graphLookup`/`$sortByCount` (see the `documentdb-constraints` skill).

## Response envelope + error codes (go-common-response)
Emitted code = `{ZONE}-{SERVICE}-{HTTP_STATUS}` (e.g. `C-LST-404`). Zones: B=BFF, C=Core, X=Integration, W=Worker. Prefixes: `B-NYB`/`B-BOB` (BFFs), `C-AUT`/`C-LST`/`C-MED`/`C-SRC`/`C-NOT`/`C-RPT`/`C-PRD` (core), `X-INT`, `W-RSS`/`W-PRD`/`W-LST`; tracking uses `D-WRK`. At BFF/proxy boundaries use `CodeForStatus(prefix, status)` to match the forwarded downstream status. Always go through each repo's `util.ResponseSuccess`/`ResponseError` wrappers.

This mirrors the auto-memory entries [[backend-service-port-map]], [[documentdb-connection-invariants]], [[go-common-response]] — kept here as an on-demand skill so it surfaces during backend work.
