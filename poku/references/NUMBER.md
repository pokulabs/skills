# Number Reservation Flow — Steps I → II → III

## Step I: Clarify Intent

Determine which path the user wants:

- **"Get me any number"** → go to Step II, Path A (auto-select)
- **"Show me what's available"** or a preference is mentioned (area code, country) → go to Step II, Path B (browse first)

---

## Step II: Find a Number

**Path A — Auto-select (user wants any number):**

Fetch one available number and proceed directly to Step III:

```bash
curl -s -G \
  -H "Authorization: Bearer $POKU_API_KEY" \
  --data-urlencode "limit=1" \
  https://api.pokulabs.com/reserved-numbers/available
```

Use the first number returned. If the response is empty, tell the user no numbers are currently available and stop.

---

**Path B — Browse first (user has a preference or wants to choose):**

Ask for any preferences not already stated (area code, country: US or GB).

Then fetch up to 10 options, omitting any params the user didn't specify:

```bash
curl -s -G \
  -H "Authorization: Bearer $POKU_API_KEY" \
  --data-urlencode "country=<US|GB>" \
  --data-urlencode "areaCode=<area code>" \
  --data-urlencode "limit=10" \
  https://api.pokulabs.com/reserved-numbers/available
```

Display results as a numbered list and ask the user to pick one. If the response is empty, tell the user no numbers match and offer to broaden the search.

---

## Step III: Confirm and Reserve

Before reserving, confirm with the user:

> "I'm going to reserve [number] for you. This is a one-time action. Ok to proceed?"

Do not reserve until the user says yes.

```bash
jq -n --arg phoneNumber "<selected number>" \
  '{"phoneNumber": $phoneNumber}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/reserved-numbers/reserve
```

On success, display the reserved number clearly to the user. For error codes, see `references/API.md`.
