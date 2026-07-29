# cursor-api-key-check-limit

Bash CLI that validates a Cursor API key against `GET /v1/me`, then sends a short hello to a couple of models via no-repo Cloud Agents.

## Requirements

- `bash`
- `curl`
- `python3`

## Usage

```bash
./check-key                         # uses $CURSOR_API_KEY
./check-key "crsr_xxxxxxxx"         # key as argument
```

Key source (first match wins): positional argument, then `$CURSOR_API_KEY`.

## Output

```text
valid:        yes
http_status:  200
latency_s:    0.412
api_key_name: Production API Key
created_at:   2026-04-13T18:30:00.000Z
user_id:      42
user_email:   developer@example.com
user_name:    Alex Rivera

--- model hello ---
models:       composer-2, claude-4.6-sonnet-thinking

model:        Composer 2 (composer-2)
agent_id:     bc-…
run_id:       run-…
status:       finished
duration_s:   12.350
reply:        Hello! How can I help you today?

model:        Claude 4.6 Sonnet (Thinking) (claude-4.6-sonnet-thinking)
agent_id:     bc-…
run_id:       run-…
status:       finished
duration_s:   18.102
reply:        Hello from Claude.
```

Model hello uses `GET /v1/models`, then creates temporary no-repo agents (`POST /v1/agents`) with a one-line hello prompt, polls until finished, and deletes each agent.

Service-account keys omit user fields (`user_id`, `user_email`, `user_name`).

## Exit codes

| Code | Meaning                 |
| ---- | ----------------------- |
| `0`  | Valid key               |
| `1`  | Missing / invalid key   |
| `2`  | Rate limited (`429`)    |
| `3`  | Network / request error |

Model hello results (finished, usage limit, etc.) are status-only and do not change the exit code when the key itself is valid.

```bash
if ./check-key; then
  echo "ok"
else
  echo "failed ($?)"
fi
```

## How it works

1. [API Key Info](https://cursor.com/docs/cloud-agent/api/endpoints#api-key-info) — `GET /v1/me`
2. [List Models](https://cursor.com/docs/cloud-agent/api/endpoints#list-models) — `GET /v1/models`
3. [Create An Agent](https://cursor.com/docs/cloud-agent/api/endpoints#create-an-agent) — no-repo hello for ~2 models
4. Poll run status, print reply, delete agent

`200` on `/v1/me` ⇒ valid key. Model hello may take a couple of minutes and uses Cloud Agent quota.
