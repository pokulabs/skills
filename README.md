# Poku Skills

A collection of Agent Skills for using Poku to place outbound phone calls, send outbound SMS messages, and reserve dedicated phone numbers on a user's behalf.

## Installing

These skills work with agents that support the Agent Skills format, including Cursor, Claude Code, OpenCode, OpenAI Codex, and Pi.

### Cursor

Install from the Cursor Marketplace or add manually via **Settings > Rules > Add Rule > Remote Rule (Github)** with:

```text
pokulabs/skills
```

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```bash
npx skills add pokulabs/skills
```

### Clone / Copy

Clone this repo and copy the skill folders into the appropriate directory for your agent:

| Agent | Skill Directory | Docs |
|-------|-----------------|------|
| Claude Code | `~/.claude/skills/` | [docs](https://code.claude.com/docs/en/skills) |
| Cursor | `~/.cursor/skills/` | [docs](https://cursor.com/docs/context/skills) |
| OpenCode | `~/.config/opencode/skills/` | [docs](https://opencode.ai/docs/skills/) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://developers.openai.com/codex/skills/) |
| Pi | `~/.pi/agent/skills/` | [docs](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#skills) |

## Skills

Skills are contextual and auto-loaded based on the user's request. When a request matches a skill's triggers, the agent loads the relevant instructions and reference docs.

| Skill | Useful for |
|-------|------------|
| `poku` | Placing outbound phone calls, sending outbound SMS messages, and reserving dedicated phone numbers through the Poku API |

## What The `poku` Skill Covers

The `poku` skill helps an agent:

- Place outbound phone calls to businesses, contacts, restaurants, offices, and other phone numbers
- Send outbound SMS messages
- Reserve dedicated phone numbers for the agent
- Normalize numbers into E.164 format before making requests
- Confirm user intent before sending texts, placing calls, or reserving numbers
- Follow country support constraints for calling and SMS
- Handle common API errors safely

## Configuration

The `poku` skill uses these environment variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `POKU_API_KEY` | Yes | API key used to authenticate requests to the Poku API |
| `POKU_TRANSFER_NUMBER` | No | E.164 phone number to transfer active calls to if the agent cannot answer a question |

If `POKU_API_KEY` is not configured, the agent should stop and ask the user to set it before proceeding.

## Reference Docs

The skill includes task-specific reference files:

| File | Purpose |
|------|---------|
| `poku/references/CALL.md` | Outbound call flow, confirmation requirements, templates, and supported countries |
| `poku/references/SMS.md` | SMS drafting and sending flow, templates, and supported countries |
| `poku/references/NUMBER.md` | Number search and reservation flow |
| `poku/references/API.md` | Endpoint reference, request parameters, and error handling |

## API Coverage

The current skill covers these Poku API actions:

| Action | Method | Endpoint |
|--------|--------|----------|
| Place a call | `POST` | `https://api.pokulabs.com/phone/call` |
| Send an SMS | `POST` | `https://api.pokulabs.com/phone/sms` |
| List available reserved numbers | `GET` | `https://api.pokulabs.com/reserved-numbers/available` |
| Reserve a phone number | `POST` | `https://api.pokulabs.com/reserved-numbers/reserve` |

## Safety Notes

The `poku` skill is designed to require explicit confirmation before taking user-impacting actions such as:

- placing a phone call
- sending a text
- reserving a phone number

It also includes guidance to avoid deceptive, abusive, or sensitive-information-seeking call behavior.

## Resources

- [Poku Skills Repository](https://github.com/pokulabs/skills)
- [Agent Skills Overview](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
