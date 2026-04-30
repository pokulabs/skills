# Poku Skills

Enable your AI agent to send/receive phone calls and messages (SMS and WhatsApp) on its own phone numbers.

## Installing

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```bash
npx skills add pokulabs/skills
```

Alternatively, manually copy `poku/` into your agent's `skill/` directory.

## Use cases

- Reserve dedicated phone numbers
- Place outbound calls
- Send SMS and WhatsApp messages
- Receive inbound calls and messages

## Configuration

The `poku` skill uses these environment variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `POKU_API_KEY` | Yes | API key used to authenticate requests to the Poku API |
| `POKU_TRANSFER_NUMBER` | No | E.164 phone number to transfer active calls to if the agent cannot answer a question |

If `POKU_API_KEY` is not configured, the agent should stop and ask the user to set it before proceeding.

## Resources

- **Poku skills repo** [https://github.com/pokulabs/skills](https://github.com/pokulabs/skills)
- **Full API reference for LLMs:** [docs.pokulabs.com/llms.txt](https://docs.pokulabs.com/llms.txt)
- **Interactive docs:** [docs.pokulabs.com](https://docs.pokulabs.com)
- **Human console:** [pokulabs.com](https://pokulabs.com)
- **Issues, feedback, feature requests:** email `hello@pokulabs.com`