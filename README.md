<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Truth Social API

[![SocQ Truth Social API](https://img.shields.io/badge/API-SocQ%20Truth%20Social%20API-F64C31)](https://socq.ai/apis/truth-social?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/truth-social/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/truth-social-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/truth-social-api/actions/workflows/check.yml)

3 endpoints for public Truth Social data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Truth Social APIs](https://socq.ai/apis/truth-social?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api) · [Documentation](https://docs.socq.ai/api-manual/truth-social/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Truth Social Post API](#truth-social-post-api) | `truth-social/post` | Resolve one known Truth Social post URL into normalized content, author context, media, timestamps, and available visible engagement fields. | 0.19 credits ($0.00095) per call | 2026-08-17 |
| [Truth Social Profile API](#truth-social-profile-api) | `truth-social/profile` | Look up one known public Truth Social username and receive a normalized profile record with identity, biography, profile media, account flags, and visible profile counts when available. | 0.19 credits ($0.00095) per call | 2026-08-17 |
| [Truth Social User Posts API](#truth-social-user-posts-api) | `truth-social/user-posts` | Collect one source page of public Truth Social posts by username or numeric account ID, returning normalized content, authors, media, timestamps, and visible engagement fields. | 0.19 credits ($0.00095) per call | 2026-08-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/truth-social-api.git
cd truth-social-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs truth-social/post

# Python 3.8+ (standard library only)
python3 examples/python/socq.py truth-social/post payloads/post.json

# cURL + jq
bash examples/curl/socq.sh truth-social/post '{"url":"https://truthsocial.com/@realDonaldTrump/117091934703272898"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Truth Social Post API

Resolve one known Truth Social post URL into normalized content, author context, media, timestamps, and available visible engagement fields.

- **Request:** `POST https://api.socq.ai/v1/truth-social/post`
- **publicId:** `truth-social/post`
- **Price:** 0.19 credits ($0.00095) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/post.json`](payloads/post.json)

Fetch normalized posts data from Truth Social.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/truth-social/post" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://truthsocial.com/@realDonaldTrump/117091934703272898"
}'
```

Full docs: [https://docs.socq.ai/api-manual/truth-social/post](https://docs.socq.ai/api-manual/truth-social/post) · Landing page: [socq.ai/apis/truth-social/post](https://socq.ai/apis/truth-social/post?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api)

### Truth Social Profile API

Look up one known public Truth Social username and receive a normalized profile record with identity, biography, profile media, account flags, and visible profile counts when available.

- **Request:** `POST https://api.socq.ai/v1/truth-social/profile`
- **publicId:** `truth-social/profile`
- **Price:** 0.19 credits ($0.00095) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/profile.json`](payloads/profile.json)

Fetch normalized profiles data from Truth Social.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `username` | string | Yes | A public account username. |

```bash
curl -X POST "https://api.socq.ai/v1/truth-social/profile" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "realDonaldTrump"
}'
```

Full docs: [https://docs.socq.ai/api-manual/truth-social/profile](https://docs.socq.ai/api-manual/truth-social/profile) · Landing page: [socq.ai/apis/truth-social/profile](https://socq.ai/apis/truth-social/profile?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api)

### Truth Social User Posts API

Collect one source page of public Truth Social posts by username or numeric account ID, returning normalized content, authors, media, timestamps, and visible engagement fields.

- **Request:** `POST https://api.socq.ai/v1/truth-social/user-posts`
- **publicId:** `truth-social/user-posts`
- **Price:** 0.19 credits ($0.00095) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/user-posts.json`](payloads/user-posts.json)

Fetch normalized user posts data from Truth Social.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `trim` | boolean | No | - |
| `user_id` | string | One of* | A public account identifier. |
| `username` | string | One of* | A public account username. |

*Required rule: send at least one of `user_id`, `username`.

```bash
curl -X POST "https://api.socq.ai/v1/truth-social/user-posts" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "realDonaldTrump"
}'
```

Full docs: [https://docs.socq.ai/api-manual/truth-social/user-posts](https://docs.socq.ai/api-manual/truth-social/user-posts) · Landing page: [socq.ai/apis/truth-social/user-posts](https://socq.ai/apis/truth-social/user-posts?utm_source=github&utm_medium=readme&utm_campaign=socq-truth-social-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Truth Social data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
