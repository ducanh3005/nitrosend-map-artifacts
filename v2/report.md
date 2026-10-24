# Nitrosend Safe Mode & MAP Campaign Send Approval Report

- **Environment & Commands Run:** Windows 11 x64 (`10.0.26100`), Node.js `v22.17.0`, npm `10.9.2`, `mailschema` `0.3.2`. Executed:
  1. `nitro_set_brand_kit({"fields":{"company_name":"DucAnh Engineering","physical_address":"123 Nguyen Hue Blvd, District 1, Ho Chi Minh City 700000, Vietnam"}})`
  2. `PATCH https://api.nitrosend.com/v1/my/account {"safe_mode_enabled":true}` (`200 OK` at `2026-10-24T04:28:09.112Z`).
  3. With API key: `POST /v1/my/lists` (`id: 12782`), `POST /v1/my/contacts` (own address in list `12782`), `nitro_compose_campaign` (campaign `15177`, template `24301`, no merge tags), `nitro_review_delivery`, `nitro_control_delivery` `approve`, and `nitro_control_delivery({"target_type":"campaign","target_id":15177,"operation":"schedule","scheduled_at":"2026-10-24T04:48:14.210Z","idempotency_key":"sched-note-g88jm-v1"})` -> held as `awaiting_approval`.
  4. Saved email `"Review send request: Note from DucAnh Engineering"` as `approval.eml`, ran `npm i mailschema@0.3.2`, and extracted `npx mailschema@0.3.2 message approval.eml > description.json`.
- **Minutes from Safe Mode On to Decision:** 0.19 minutes (`2026-10-24T04:28:09.112Z` to approval at `2026-10-24T04:28:20.514Z`).
- **Validate Command & Exact Output:** `npx mailschema@0.3.2 description description.json --contract campaign-send-approval-0.1.json` ->
  `Valid MAP 0.3 description urn:uuid:91ae47db-be63-5c9e-a246-d1a128aa66cb / sha-256:fc3a369383f1e28f4792d71a6cac4df448cdbb9bd31a15c0564f2fd35f4b6013`
- **Recomputed `terms.version`:** Deleted `terms.version` on a clone of `description.json` and ran `digest(copy)` from `import { digest } from "mailschema"`, getting `sha-256:90282110c536827c34b16473d3ca28746fd744c66363aecdaba48ec24b39b093` (exact match).
- **Decision & Why:** Approved (`POST /v1/my/map/approvals/314` with `{"decision":"approve","version":"sha-256:90282110c536827c34b16473d3ca28746fd744c66363aecdaba48ec24b39b093"}` for `https://app.nitrosend.com/my/map/reviews/314`) because the description passed contract validation, `terms.version` matched, `details.audience.count` was `1` (only my address), and `details.content.digest` matched template `24301`.
- **Slowest Step:** Passing `audience` during `nitro_compose_campaign` `draft` mode did not bind the list trigger until a follow-up `mode: "patch"` call with an explicit `idempotency_key`.
- **Defect (Exact Repro):** Calling `nitro_control_delivery` with `operation: "schedule"` under Safe Mode returns `delivery.status: "awaiting_approval"`, but top-level `next_action` says `"Scheduled for 2026-10-24T04:48:14.210Z — monitor via nitro_get_insights"` and `delivery.poll_with.arguments.operation` is hardcoded to `"live"`.
- **One Change I Would Make:** When `delivery.status === "awaiting_approval"`, set `next_action` to report the Safe Mode hold and preserve `"schedule"` in `poll_with.arguments.operation`.
