## API Reference

All requests use Bearer auth:

```bash
-H "Authorization: Bearer $POKU_API_KEY"
```

Base URL:

```text
https://api.pokulabs.com
```

If `POKU_API_KEY` is missing, stop and ask the user to configure it before making any request.

---

### Calls

Use `/calls` for outbound voice calls, plus list/get operations for past calls.

#### Place a call

```bash
curl -X POST https://api.pokulabs.com/calls \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "You are a friendly voice assistant calling on behalf of Joe to make a dinner reservation for 2 people tomorrow at 7pm. If that time is unavailable, ask what nearby times are open. If no one answers, leave a short voicemail asking them to call back.",
    "to": "+14155559999",
    "transferTo": "+14155551234",
    "language": "en-US",
    "voice": "female"
  }'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | Yes | Instructions for the call. This defines the goal, context, branching behavior, and voicemail behavior. |
| `to` | string | Yes | Recipient phone number in E.164 format. |
| `from` | string | No | Caller ID to place the call from. Defaults to your reserved Poku number, or a shared Poku number if you do not have one. |
| `transferTo` | string | No | E.164 number to transfer the call to if the callee asks to speak with a human. |
| `language` | string | No | Language and dialect hint such as `en-US` or `en-GB`. Defaults to `en-US`. |
| `voice` | `"male"` \| `"female"` | No | Voice used for the call. Defaults to `"female"`. |

**Response:**

```json
{
  "response": "ABC restaurant confirmed your reservation for 2 people at 6 pm tomorrow.",
  "call": {
    "recordingUrl": "https://cdn.example.com/recordings/abc123.wav"
  },
  "interactionId": "f3f7870f-30f5-4bf2-83b0-a05bc140030c"
}
```

`response` is a natural-language summary of the call outcome. `call.recordingUrl` may be omitted when no recording is available.

---

### Messages

Use `/messages` for SMS, WhatsApp, Slack, message history, and sending through saved channels.

#### Send an SMS

```bash
curl -X POST https://api.pokulabs.com/messages/sms \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Your appointment is confirmed for Tuesday at 2pm.",
    "to": "+14155559999",
    "waitTime": 60,
    "followUpTime": 6000
  }'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | Yes | Message body. Maximum 1000 characters. |
| `to` | string | Yes | Recipient phone number in E.164 format. |
| `from` | string | No | Sender number in E.164 format. Defaults to your reserved Poku number, or a shared Poku number if you do not have one. |
| `waitTime` | number | No | Seconds to hold the HTTP request open while waiting for a reply. Must be between `1` and `600`. |
| `followUpTime` | number | No | Seconds to keep listening after `waitTime` closes. Replies in this window are delivered via webhook. Must be between `1` and `60000`. |

**Responses:**

When no `waitTime` is set:

```json
{
  "interactionId": "092f9eb4-3694-4718-9084-5c786afa87ec"
}
```

When a reply arrives during `waitTime`:

```json
{
  "response": "Human response: Hello there!"
}
```

When no reply arrives during `waitTime`:

```json
{
  "response": "Human did not respond, please continue where you left off."
}
```

#### Send a WhatsApp message

```bash
curl -X POST https://api.pokulabs.com/messages/whatsapp \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Hi! Just checking whether 3pm still works for you.",
    "to": "+14155559999"
  }'
```

WhatsApp has two documented modes:

1. Send from the Poku WhatsApp account using free-form text.
2. Send from your own WhatsApp number using a Meta-approved template.

**Send from Poku WhatsApp**

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | Yes | Free-form WhatsApp text sent from the Poku WhatsApp account. |
| `to` | string | Yes | Recipient phone number in E.164 format. |
| `waitTime` | number | No | Seconds to wait synchronously for a reply. |
| `followUpTime` | number | No | Seconds to continue listening after `waitTime`. |

**Send from your own WhatsApp number**

```json
{
  "template": {
    "id": "appointment_reminder",
    "variables": {
      "name": "Joe",
      "time": "3:00 PM"
    }
  },
  "to": "+14155559999",
  "from": "+14155551234"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `template` | object | Yes | Meta-approved WhatsApp template. |
| `template.id` | string | Yes | Template UUID from your Meta Business account. |
| `template.variables` | object | Yes | Key-value map of template variable names to string values. |
| `to` | string | Yes | Recipient phone number in E.164 format. |
| `from` | string | Yes | Your own WhatsApp-enabled number in E.164 format. |
| `waitTime` | number | No | Seconds to wait synchronously for a reply. |
| `followUpTime` | number | No | Seconds to continue listening after `waitTime`. |

#### Send a Slack message

```bash
curl -X POST https://api.pokulabs.com/messages/slack \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Can you review the latest build today?",
    "to": "U12345678"
  }'
```

Prerequisite: you must either join the Poku Slack workspace or add the Poku bot to your own workspace before using this endpoint.

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | Yes | Slack message body. |
| `to` | string | Yes | Slack user, member, or channel ID. |
| `from` | string | No | Slack workspace ID. If omitted, the message is sent from the Poku Slack workspace. |
| `waitTime` | number | No | Seconds to wait for a threaded reply in the HTTP response. |
| `followUpTime` | number | No | Seconds to continue listening after `waitTime`. |

#### Send through a saved channel

```bash
curl -X POST https://api.pokulabs.com/messages/CHANNEL_ID \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Following up on our last conversation.",
    "waitTime": 60
  }'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `message` | string | Conditionally | Message body. Provide either `message` or, for WhatsApp channels, `template`. |
| `template` | object | Conditionally | WhatsApp template payload for WhatsApp channels only. |
| `medium` | `"sms"` \| `"whatsapp"` \| `"slack"` | No | Override the medium when the saved channel supports it. |
| `to` | string | No | Override recipient. |
| `from` | string | No | Override sender. |
| `waitTime` | number | No | Seconds to hold the request open for a reply. |
| `followUpTime` | number | No | Seconds to keep listening after `waitTime`. |

---

### Numbers

Use `/numbers` to browse reservable numbers, reserve one, list your current numbers, release one, and inspect message history tied to a reserved number.

#### List available numbers

```bash
curl -G https://api.pokulabs.com/numbers/available \
  -H "Authorization: Bearer $POKU_API_KEY" \
  --data-urlencode "country=US" \
  --data-urlencode "areaCode=415"
```

| Field | Type | Required | Description |
|---|---|---|---|
| `country` | `"US"` \| `"GB"` | No | Country to search in. Defaults to `"US"`. |
| `areaCode` | number | No | Three-digit US area code. Only used for US numbers. |

**Response:**

```json
[
  {
    "phoneNumber": "+12184034360",
    "locality": "Hibbing",
    "region": "MN",
    "country": "US"
  },
  {
    "phoneNumber": "+12184008148",
    "locality": "Warren",
    "region": "MN",
    "country": "US"
  }
]
```

#### List your reserved numbers

```bash
curl https://api.pokulabs.com/numbers \
  -H "Authorization: Bearer $POKU_API_KEY"
```

Each reserved number includes fields such as:

- `id`
- `phoneNumber`
- `createdAt`
- `updatedAt`

#### Reserve a number

```bash
curl -X POST https://api.pokulabs.com/numbers/reserve \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+14155551234"
  }'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `phoneNumber` | string | Yes | E.164 number returned by `GET /numbers/available`. |

**Response:**

```json
{
  "id": "cab29b2d-a8be-43a6-82e1-7eb33b9fc844",
  "phoneNumber": "+14157046427",
  "createdAt": "2026-03-02T17:21:32.602Z",
  "updatedAt": "2026-03-02T17:21:32.602Z"
}
```

#### Release a number

```bash
curl -X DELETE https://api.pokulabs.com/numbers/RESERVED_NUMBER_ID \
  -H "Authorization: Bearer $POKU_API_KEY"
```

**Response:**

```json
{
  "id": "cab29b2d-a8be-43a6-82e1-7eb33b9fc844",
  "number": "+14157046427"
}
```

Releasing a number is a user-impacting action. Confirm before doing it.

---

### Webhooks

Use `/webhooks` to subscribe a URL to outbound events generated for your reserved numbers.

#### Supported event types

| Event | Description |
|---|---|
| `phone.sms.received` | Fired when an inbound SMS is received. |
| `phone.call.received` | Fired when an inbound phone call is received. |

#### List webhooks

```bash
curl https://api.pokulabs.com/webhooks \
  -H "Authorization: Bearer $POKU_API_KEY"
```

Example response:

```json
{
  "data": [
    {
      "id": "WEBHOOK_ID",
      "appUserId": "APP_USER_ID",
      "createdAt": "2026-04-21T18:12:00.000Z",
      "updatedAt": "2026-04-21T18:12:00.000Z",
      "name": "Inbound SMS webhook",
      "url": "https://example.com/poku-webhook",
      "headers": {
        "X-Source": "poku"
      },
      "eventTypes": ["phone.sms.received"],
      "interactionChannelId": null,
      "reservedPhoneNumberIds": [
        "cab29b2d-a8be-43a6-82e1-7eb33b9fc844"
      ],
      "isActive": true
    }
  ]
}
```

#### Create a webhook

```bash
curl -X POST https://api.pokulabs.com/webhooks \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Inbound SMS webhook",
    "url": "https://example.com/poku-webhook",
    "signingSecret": "supersecret123",
    "headers": {
      "X-Source": "poku"
    },
    "eventTypes": ["phone.sms.received"],
    "reservedPhoneNumberIds": ["cab29b2d-a8be-43a6-82e1-7eb33b9fc844"]
  }'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | No | Friendly name for the webhook. |
| `url` | string | Yes | Destination URL. Must be a valid URL. |
| `signingSecret` | string | No | Optional shared secret for signature verification. Must be 8 to 200 characters. |
| `headers` | object | No | Extra static headers to attach to deliveries. |
| `eventTypes` | string[] | Yes | One or more subscribed event types. Maximum 10. |
| `reservedPhoneNumberIds` | string[] | No | Limit deliveries to specific reserved number IDs. Maximum 50 IDs. |

#### Update a webhook

```bash
curl -X PATCH https://api.pokulabs.com/webhooks/WEBHOOK_ID \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "isActive": false,
    "eventTypes": ["phone.sms.received", "phone.call.received"]
  }'
```

Only the fields you send are updated. You can change:

- `name`
- `url`
- `signingSecret`
- `headers`
- `isActive`
- `eventTypes`
- `reservedPhoneNumberIds`

#### Delete a webhook

```bash
curl -X DELETE https://api.pokulabs.com/webhooks/WEBHOOK_ID \
  -H "Authorization: Bearer $POKU_API_KEY"
```

This returns `204 No Content`.
