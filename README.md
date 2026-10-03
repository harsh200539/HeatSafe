# HeatSafe

Compare fictional routes using temperature observations and available cooling spaces.

## Working features
- Editable domain registers with strict server validation and bounded record search.
- Server-computed analysis, visible explanations and saved result snapshots.
- Human review notes, CSV record exports and complete workspace JSON exports.
- Private Sites access; pseudonymous fictional data only.
- Revision-checked writes and SHA-256-linked mutation history.
- Recomputed recommendations, idempotent reservations, confirmation/cancellation and capacity reconciliation.
- Departure releases confirmed occupancy exactly once.

## Domain method
The Rothfusz regression estimates apparent temperature for the entered warm-weather observations (restricted here to 27–43°C and 13–85% humidity). Simple paths are enumerated on a bounded graph. Route score is degree-minutes above your baseline plus weighted travel minutes. Centres must be open at arrival, accessible when requested, and have capacity. This custom route score is not a physiological risk model.

## Scope
All forecasts and routes are fictional, with no live weather or navigation. Apparent temperature has model assumptions and does not account for individual vulnerability, sun exposure, workload or acclimatization. The planner does not prescribe safe exposure times.

## Architecture
React, TypeScript, Vinext, Cloudflare Worker, Zod validation, Sites-managed D1 SQLite. A bounded versioned workspace document is stored in one D1 row. Prepared compare-and-swap updates atomically save linked entity changes, reservations and audit events. Another writer using the old revision receives 409. A broken audit chain blocks mutations. Private platform access is the authorization boundary; do not make this public without adding application authentication and role permissions.

The SHA-256 chain detects modifications to retained events; it is not independently notarized and a database administrator can rewrite the whole chain. No files, live connectors, outbound messages, payments or real identities are included. Real operations need organization roles, verified evidence, backups and domain validation.

## Data limits
100 records per type;100 saved reports;100 reservations;300 mutation events;1.8MB document. Heat routing:10 locations,30 links and15,000 explored labels. Food dispatch:8 active lots,8 recipients,100-state beam and3 jobs. For scale, normalize entities and move audit events into append-only indexed tables. The current primary-key lookup needs no extra index.

## API
GET /api/state:state,revision,initial calculation and audit verification. POST /api/state:strictly validated action with expected revision. Actions:analyze,snapshot,add,edit,review; reservations add reserve/settle. Writes require same-origin Origin and x-workbench-request:1. Prices are integer paise. Client-supplied recommendations and reserved quantities are never trusted.

## Run and verify
Install with node scripts/install-ci.mjs. Generate schema using npm run db:generate. Apply Drizzle migrations to local D1 before preview. The Sites managed preview runtime supports this checkout. Build with npm run build.

Run pnpm exec tsc --noEmit.
Run node --experimental-strip-types --test tests/domain.test.mjs.
After building, run node --experimental-strip-types --test tests/api.test.mjs.

The API suite runs the built Worker with a real local D1 database. It covers validation, creation/editing, immutable snapshots, review notes, simultaneous writes, hash integrity and rendered HTML. Reservation projects additionally cover simultaneous claims, replay, cancellation, confirmation and release/fulfillment. Browser click-through and WebMCP validation are unavailable in this environment; WebMCP read-workspace support is feature detected.

## Deployment
Sites owns the Worker runtime and D1 resources. Logical binding DB and source identity live in .openai/hosting.json. Generated migrations are saved and applied by Sites on publish. Source is pushed before deploying the matching build.

## Primary background sources
- [NWS Rothfusz technical attachment](https://www.weather.gov/media/ffc/ta_htindx.PDF)
