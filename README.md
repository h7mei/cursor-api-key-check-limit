# cursor-api-key-check-limit

Bash CLI that validates a Cursor API key against `GET /v1/me`. Default output is only `true` or `false`.

## Requirements

- `bash`
- `curl`
- `python3` (optional — cleaner `--info` formatting)

## Usage

```bash
./check-key                         # $CURSOR_API_KEY → true/false
./check-key "crsr_xxxxxxxx"         # key as argument
./check-key -i                      # metadata + latency
./check-key -j                      # raw /v1/me JSON
./check-key -t 10                   # timeout in seconds (default: 15)
./check-key -h                      # help
```

Key source (first match wins): positional argument, then `$CURSOR_API_KEY`.

## Output

### Default

| Result  | Meaning                                                   |
| ------- | --------------------------------------------------------- |
| `true`  | Key is valid (`GET /v1/me` → `200`)                       |
| `false` | Missing, invalid, rate-limited (`429`), or request failed |

### `-i` / `--info`

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

### `-j` / `--json`

Prints the `/v1/me` response body. On network failure:

```json
{ "error": "network_error", "http_status": 0 }
```

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
