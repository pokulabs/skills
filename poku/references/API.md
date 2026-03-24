# Poku API Reference

## Endpoints

| Action | Method | URL |
|---|---|---|
| Place a call | `POST` | `https://api.pokulabs.com/phone/call` |
| Send an SMS | `POST` | `https://api.pokulabs.com/phone/sms` |
| List available numbers | `GET` | `https://api.pokulabs.com/reserved-numbers/available` |
| Reserve a number | `POST` | `https://api.pokulabs.com/reserved-numbers/reserve` |

## Authentication

All requests require a Bearer token in the `Authorization` header:

```
Authorization: Bearer $POKU_API_KEY
```

If `POKU_API_KEY` is not set, inform the user to configure it before proceeding.

---

## POST /phone/call

### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `message` | string | Yes | Context on the goal and task for the voice agent to accomplish. See `references/CALL.md` for templates. |
| `to` | string | Yes | Destination phone number in E.164 format (e.g. `+15551234567`). |
| `transferNumber` | string | No | Phone number to transfer the call to if the agent cannot answer a question. Must be valid E.164 format. Validate before placing the call. |

### Example Requests

```bash
# Without transfer number
jq -n --arg msg "<goal and context summary for voice agent>" --arg to "+15551234567" \
  '{"message": $msg, "to": $to}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/call

# With transfer number
jq -n --arg msg "<goal and context summary for voice agent>" --arg to "+15551234567" --arg transfer "$POKU_TRANSFER_NUMBER" \
  '{"message": $msg, "to": $to, "transferNumber": $transfer}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/call
```

### Error Responses

| Error string | Meaning | What to do |
|---|---|---|
| `"human did not respond"` | Call connected but no one answered or engaged | Report to user and stop |
| `"invalid to number"` | `to` field is malformed or unroutable | Report to user; re-check E.164 formatting |
| `"timeout"` | Call exceeded 5-minute limit | Report to user; do not retry automatically |

---

## POST /phone/sms

### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `message` | string | Yes | The text message body to send. |
| `to` | string | Yes | Destination phone number in E.164 format (e.g. `+15551234567`). |

### Example Request

Run using `bash_tool`:

```bash
jq -n --arg msg "<message body>" --arg to "+15551234567" \
  '{"message": $msg, "to": $to}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/sms
```

### Error Responses

| Error string | Meaning | What to do |
|---|---|---|
| `"invalid to number"` | `to` field is malformed or unroutable | Report to user; re-check E.164 formatting |

---

## GET /reserved-numbers/available

Lists phone numbers available to reserve.

### Query Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `country` | string | No | Filter by country. Accepted values: `"US"`, `"GB"`. Omit to search all. |
| `areaCode` | number | No | Filter by area code (e.g. `415`). US only. |
| `limit` | number | No | Maximum results to return. Use `1` for auto-select, `10` for browse. |

### Example Request

```bash
curl -s -G \
  -H "Authorization: Bearer $POKU_API_KEY" \
  --data-urlencode "country=US" \
  --data-urlencode "areaCode=916" \
  --data-urlencode "limit=10" \
  https://api.pokulabs.com/reserved-numbers/available
```

### Response

Returns an array of available phone number objects. `locality` may be `null` for some numbers.

```json
[
  {
    "phoneNumber": "+18142307424",
    "locality": "Warren",
    "region": "PA",
    "country": "US"
  },
  {
    "phoneNumber": "+18392266992",
    "locality": null,
    "region": "US",
    "country": "US"
  }
]
```

When displaying options to the user, show `phoneNumber` and include `locality` and `region` where available (e.g. "Warren, PA"). Use only `phoneNumber` when passing to the reserve endpoint.

### Error Responses

| Error string | Meaning | What to do |
|---|---|---|
| `"invalid country"` | `country` value is not `"US"` or `"GB"` | Report to user; correct the value |

---

## POST /reserved-numbers/reserve

Reserves a specific phone number for the user's account. This is a one-time, non-reversible action.

### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phoneNumber` | string | Yes | The phone number to reserve, in E.164 format. Must be a number returned by `GET /reserved-numbers/available`. |

### Example Request

```bash
jq -n --arg phoneNumber "+15551234567" \
  '{"phoneNumber": $phoneNumber}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/reserved-numbers/reserve
```
