---
name: poku
description: "Places outbound phone calls, sends outbound SMS texts, and reserves dedicated phone numbers on the user's behalf using the Poku API. Use when the user wants to call or text a restaurant, a contact, a business, doctor's office, or any phone number. Also use when the user wants to reserve a dedicated phone number for their agent."
---

# Poku — Outbound Calls, SMS & Number Reservation

## Variables

- `POKU_API_KEY` *(required)* — Poku API key. If not set, inform the user to configure it before proceeding.
- `POKU_TRANSFER_NUMBER` *(optional)* — A phone number in E.164 format to transfer calls to if the agent cannot answer a question. If not set, ask the user if they have a transfer number they'd like to use; if they don't, skip it.

## How to Execute API Calls

Use an http client such as curl in a shell environment, or fetch in a browser environment. Example:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}' \
  https://api.pokulabs.com/...
```

Never display the full curl command with a resolved API key in user-facing output. Mask the key if showing commands for debugging: `Bearer ***`.

---

## Identify Which Flow to Use

| The user wants to... | Read this file |
|---|---|
| Call someone | `references/CALL.md` |
| Text someone | `references/SMS.md` |
| Reserve a phone number | `references/NUMBER.md` |

Read only the file(s) that match the user's request. For error codes and API parameters, see `references/API.md`.
