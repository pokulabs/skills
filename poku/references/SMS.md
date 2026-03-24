# SMS Flow — Steps A → B → C

## Step A: Resolve the Phone Number

- **Raw number** (e.g. `917-257-7580`) — if no country code is included, do not assume one. Ask the user: "What country is this number in?" Then prepend the correct dial code. Result: `+19172577580`
- **Contact name** — ask the user for the number directly; do not guess
- **Business name only** — use the search tool to find the number, then confirm with the user

Once the country is known, verify it is supported for SMS:

| Country | Dial code |
|---|---|
| United States & Canada | +1 |
| Australia | +61 |
| Mexico | +52 |
| Germany | +49 |
| Netherlands | +31 |
| Portugal | +351 |
| Spain | +34 |
| United Kingdom | +44 |

If the country is not in this list, stop and tell the user:
> "SMS isn't supported for that country yet. See supported countries here: https://docs.pokulabs.com/reference/supported-countries."
Do not proceed to Step B if the country is unsupported.

→ Output: `<normalized number>` in E.164 format, used in Step C.

---

## Step B: Draft and Confirm

Use the **SMS Templates** section below to draft the message. Show it clearly:

> **Here's the draft:**
> "[message body]"
>
> Sending to [number] — good to go?

Do not send until the user confirms.

---

## Step C: Send the Text

```bash
jq -n --arg msg "<message body>" --arg to "<normalized number>" \
  '{"message": $msg, "to": $to}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/sms
```

Never display the full curl command with the resolved API key in user-facing output. If you need to show the command for debugging, mask the key: `Bearer ***`.

Once sent, confirm: "Done — your text was sent to [number]."

For error codes, see `references/API.md`.

---

# SMS Templates

Use the closest matching template and adapt it — an exact match is not required.

If no template fits, use **General / Other** as a starting point.

**Placeholder rules:** All placeholders appear in `[brackets]`. Replace every placeholder with a real value. Never leave a placeholder unfilled.

---

### Scheduling

```
Hi, I'm reaching out on behalf of [user name] to coordinate [activity] for you both. Let me know a few times that you're available [timeframe]!
```

---

### Follow-Up After a Call

```
Hi, this is [user name] following up on our recent conversation about [topic]. [One sentence summary of next step or ask.] Feel free to reply here or call back at your convenience.
```

---

### General / Other

```
Hi, this is a message on behalf of [user name]. [State the purpose in one or two plain sentences.] [Optional: include a call to action — reply, call back, confirm, etc.] Thank you.
```
