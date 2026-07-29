# cursor-api-key-check-limit

Bash CLI that validates a Cursor API key against `GET /v1/me` and prints all available key metadata.

## Requirements

- `bash`
- `curl`
- `python3` (optional — cleaner formatting)

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
```

On `429`, also prints `retry_after` when `Retry-After` / reset headers are present.

Service-account keys omit user fields (`user_id`, `user_email`, `user_name`).

## Exit codes

| Code | Meaning                 |
| ---- | ----------------------- |
| `0`  | Valid key               |
| `1`  | Missing / invalid key   |
| `2`  | Rate limited (`429`)    |
| `3`  | Network / request error |

```bash
if ./check-key; then
  echo "ok"
else
  echo "failed ($?)"
fi
```

## How it works

Uses the Cloud Agents [API Key Info](https://cursor.com/docs/cloud-agent/api/endpoints#api-key-info) endpoint:

```bash
curl https://api.cursor.com/v1/me -u "$API_KEY:"
```

`200` ⇒ valid key.
