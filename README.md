# cursor-api-key-check

Bash CLI that checks whether a Cursor API key is valid. Prints only `true` or `false`.

## Requirements

- `bash`
- `curl`

## Usage

```bash
./check-key                    # uses $CURSOR_API_KEY
./check-key "crsr_xxxxxxxx"    # pass key as argument
```

## Output

| Result  | Meaning                                  |
| ------- | ---------------------------------------- |
| `true`  | Key is valid (`GET /v1/me` returned 200) |
| `false` | Key missing, invalid, or request failed  |

Exit code: `0` when valid, `1` otherwise.

## How it works

Calls the Cursor Cloud Agents [API Key Info](https://cursor.com/docs/cloud-agent/api/endpoints#api-key-info) endpoint:

```bash
curl https://api.cursor.com/v1/me -u "$API_KEY:"
```

A `200` response means the key is valid.
# cursor-api-key-check-limit
