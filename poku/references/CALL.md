# Call Flow — Steps 1 → 2 → 3 → 4

## Step 1: Resolve the Phone Number

- **Raw number** (e.g. `917-257-7580`) — if no country code is included, do not assume one. Ask the user: "What country is this number in?" Then prepend the correct dial code. Result: `+19172577580`
- **Contact name** — ask the user for the number directly; do not guess
- **Business name only** — use the search tool to find the number, then confirm with the user

Once the country is known, verify it is supported for voice calling:

| Country | Dial code |
|---|---|
| United States & Canada | +1 |
| Australia | +61 |
| Brazil | +55 |
| France | +33 |
| Germany | +49 |
| Israel | +972 |
| Italy | +39 |
| Luxembourg | +352 |
| Norway | +47 |
| Portugal | +351 |
| Spain | +34 |
| United Kingdom | +44 |

If the country is not in this list, stop and tell the user:
> "Voice calling isn't supported for that country yet. See supported countries here: https://docs.pokulabs.com/reference/supported-countries."
Do not proceed to Step 2 if the country is unsupported.

→ Output: `<normalized number>` in E.164 format, used in Step 4.

---

## Step 2: Gather Details and Confirm Intent

Use the **Call Templates** section below to identify what details are missing, then ask the user for only those details. If no template matches, ask for: the goal and any names or reference numbers needed.

```bash
echo "$POKU_TRANSFER_NUMBER"
```
- If it returns a non-empty value, use it automatically — do not ask the user.
- If it is empty or unset, ask the user: "Do you have a number I should transfer the call to if the recipient wants to speak with you directly?" If they provide one, validate it is in E.164 format and use it. If they don't have one, proceed without a transfer number.

You MUST present a plan and get user confirmation before continuing:

> "I'm going to call [place] at [number] to [goal].
> I'll mention I'm calling on behalf of [user name].
> If no one answers, I'll leave a voicemail: [one sentence].
> If needed, I'll transfer the call to you at [transfer number]. *(only include if set)*
> No transfer number configured — if they ask to speak with you directly, I'll let them know you're unavailable. *(only include if not set)*
> Ok to proceed?"

Do not move to Step 3 until the user says yes.

---

## Step 3: Draft the Call Context

Use a matching template below — fill in every placeholder with a real value. Never leave a placeholder blank.

If no template matches, draft a message with:
1. **Identity** — who the agent is and who they're calling on behalf of
2. **Goal** — the specific objective with branching logic for likely responses
3. **Voicemail script** — exact words to leave if no one answers

You MUST follow these guidelines when drafting the call context.
Never construct a message that:
- Impersonates law enforcement, government agencies, or specific real individuals
- Contains threats, harassment, or intimidation
- Makes deceptive claims of authority
- Attempts to extract sensitive information (SSN, passwords, financial details) from the call recipient
- Misrepresents the caller's identity or purpose

If a user request would violate these guidelines, decline and explain why.

---

## Step 4: Place the Call

Calls can stay open up to 5 minutes — do not retry while a request is pending.

```bash
# Without transfer number
jq -n --arg msg "<drafted message from Step 3>" --arg to "<normalized number>" \
  '{"message": $msg, "to": $to}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/call

# With transfer number
jq -n --arg msg "<drafted message from Step 3>" --arg to "<normalized number>" --arg transfer "$POKU_TRANSFER_NUMBER" \
  '{"message": $msg, "to": $to, "transferNumber": $transfer}' | \
curl -s -X POST \
  -H "Authorization: Bearer $POKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  https://api.pokulabs.com/phone/call
```

Never display the full curl command with the resolved API key in user-facing output. If you need to show the command for debugging, mask the key: `Bearer ***`.

For error codes, see `references/API.md`.

---

# Call Templates

Use the closest matching template and adapt it — an exact match is not required. For example, use the Medical or Dental Appointment template for any routine specialist visit.

If no template fits, use **General / Other** as a starting point.

**Placeholder rules:** All placeholders appear in `[brackets]`. Replace every placeholder with a real value. Never leave a placeholder unfilled.

---

### Restaurant Reservation

```
You are a friendly voice assistant calling on behalf of [user name] to make a dinner reservation.

Make a dinner reservation for [party size] people on [day] at [time], under the name [user name].
- If that time is available, confirm the reservation and ask if a note is needed for a special occasion.
- If [time] is unavailable, ask what times are open and accept an alternative within one hour of the original if reasonable. Confirm the new time back clearly before ending the call.
- If no one answers, leave this voicemail: "Hi, this is a message on behalf of [user name] — I'm hoping to make a dinner reservation for [party size] on [day] at [time]. Please call back to confirm. Thank you."
```

---

### Medical or Dental Appointment

```
You are a friendly voice assistant calling on behalf of [user name] to schedule an appointment.

Schedule a [appointment type — e.g. routine dental cleaning, annual physical, follow-up visit] for [user name].
- Preferred times are [preferred times], but any weekday works if those aren't available.
- Ask for the earliest available slot and confirm the date and time before ending the call.
- If no one answers, leave this voicemail: "Hi, this is a message on behalf of [user name] — I'm calling to schedule a [appointment type]. Please call back at your earliest convenience. Thank you."
```

---

### Get a Quote from a Business

```
You are a friendly voice assistant calling on behalf of [user name] to get a price quote.

Get a quote from [business name] for [service description — e.g. replacing a water heater, painting a two-bedroom apartment, weekly lawn maintenance].
- Briefly describe the job: [relevant details — e.g. size, location, urgency, existing conditions].
- Ask what information they need to provide an estimate and answer any clarifying questions as best you can.
- If they can give a ballpark or firm quote over the phone, note it clearly and ask what the next step would be.
- If they need to send someone out first, ask for the earliest available time. Preferred times are [preferred times]. Accept an alternative if needed and confirm the appointment clearly before ending the call.
- If no one answers, leave this voicemail: "Hi, this is a message on behalf of [user name] — I'm hoping to get a quote for [service description]. Please call back at your convenience. Thank you."
```

---

### Follow-Up or Check-In Call

```
You are a friendly voice assistant calling on behalf of [user name] to follow up on a previous matter.

Follow up with [business or contact name] about [topic — e.g. a pending repair estimate, an open support ticket, an order status].
- Reference any relevant context: [reference number, date of previous contact, or other identifying details].
- Ask for a status update and note any new information clearly.
- If there is a next step or expected timeline, confirm it before ending the call.
- If no one answers, leave this voicemail: "Hi, this is a message on behalf of [user name] — I'm following up about [topic]. Please call back when you have a moment. Thank you."
```

---

### General / Other

```
You are a friendly voice assistant calling on behalf of [user name].

[Describe the specific objective in one or two sentences.]
- [Branch 1: what to do if the primary goal succeeds]
- [Branch 2: what to do if the primary goal is not possible — ask for alternatives or next steps]
- [Branch 3: any other likely scenario worth handling]
- If no one answers, leave this voicemail: "[Exact voicemail script — one or two sentences, natural language]"
```
