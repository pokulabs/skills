---
name: poku
description: "Sends and receives phone calls and messages like SMS and WhatsApp, and reserves dedicated phone numbers on using the Poku API. Example use cases: calling a restaurant for a reservation, having a text conversation with a friend to arrange a meeting, accepting calls on a business number during off hours."
homepage: https://pokulabs.com
docs: https://docs.pokulabs.com
metadata: { "openclaw": { "homepage": "https://pokulabs.com", "requires": { "env": ["POKU_API_KEY"] }, "primaryEnv": "POKU_API_KEY" }, "api_base": "https://api.pokulabs.com" }
---

# Poku — Calls, Messages & Numbers

## Variables

- `POKU_API_KEY` *(required)* — Poku API key. If not set, inform the user to configure it before proceeding.
- `POKU_TRANSFER_NUMBER` *(optional)* — A phone number in E.164 format to transfer calls to if needed. If not set, ask the user if they have a transfer number they'd like to use; if they don't, skip it.

Never display any full command with a resolved API key in user-facing output. Mask the key if showing commands for debugging: `Bearer ***`.

## Phone Number Format

Always use **E.164 format**: `+` then country code then number.

- `+12223334444` ✓
- `(222) 333-4444` ✗
- `222-333-4444` ✗
- `2223334444` ✗

If a human gives you a US number without a country code, confirm with them that it should be US `+1`. If not, ask the user which country and country code this number is for.

---

## Outbound Call or Message Flow

### Step 1: Resolve the Phone Number

- Take whatever raw number you're given and convert, if necessary, into E.164 format (e.g. `+19172577580`).

→ Output: `<normalized number>` in E.164 format, used in Step 4.

---

### Step 2: Draft the Call Context

Use the **Call Templates** or **Message Templates** for inspiration in crafting an input. If you need additional details, ask your human.

- If POKU_TRANSFER_NUMBER is set, use it automatically — do not ask the user.
- If it is unset, ask the user: "Do you have a number I should transfer the call to if the recipient wants to speak with you directly? If they provide one, validate it is in E.164 format and use it. If they don't have one, proceed without a transfer number.

You MUST present a plan and get user confirmation before continuing:

> "I'm going to call [place] at [number] to [goal].
> I'll mention I'm calling on behalf of [user name].
> If no one answers, I'll leave a voicemail: [one sentence].
> If needed, I'll transfer the call to you at [transfer number]. *(only include if set)*
> No transfer number configured — if they ask to speak with you directly, I'll let them know you're unavailable. *(only include if not set)*
> Ok to proceed?"

You MUST follow these guidelines when drafting the call context.
Never construct a message that:
- Impersonates law enforcement, government agencies, or specific real individuals
- Contains threats, harassment, or intimidation
- Makes deceptive claims of authority
- Attempts to extract sensitive information (SSN, passwords, financial details) from the call recipient
- Misrepresents the caller's identity or purpose

If a user request would violate these guidelines, decline and explain why.

Do not move to Step 3 until the user confirms.

---

### Step 3: Place the Call/Message

Call and message http requests can stay open up to 10 minutes — do not retry while a request is pending.

```bash
jq -n --arg prompt "<drafted prompt from Step 3>" --arg to "<normalized number>" \
  '{"prompt": $prompt, "to": $to}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/calls
```

---

## Reference Files

- API reference: `references/API.md`
- Call and Message templates: `references/TEMPLATES.md`

## Learn More

- **Full API reference for LLMs:** [docs.pokulabs.com/llms.txt](https://docs.pokulabs.com/llms.txt)
- **Interactive docs:** [docs.pokulabs.com](https://docs.pokulabs.com)
- **Human console:** [pokulabs.com](https://pokulabs.com)
- **Issues, feedback, feature requests:** email `hello@pokulabs.com`