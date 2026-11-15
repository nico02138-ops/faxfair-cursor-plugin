# FaxFair for Cursor

Send a fax to a United States or Canadian number from Cursor. This plugin is configuration and a skill. It points at FaxFair's already hosted public MCP server at `https://api.faxfair.com/mcp`. It does not include FaxFair's server, and it does not include an API key.

## What it does

The skill `send-fax` uses three tools on that server:

| Tool | What it does | Charge |
| --- | --- | --- |
| `quote_fax` | Prices one fax. Always call this first and show the price. | Free |
| `send_fax` | Sends an `https` PDF after you confirm the number, the document, and the price. | Charges the prepaid key |
| `get_fax_status` | Reads `queued`, `sending`, `delivered`, or `failed`. | Free |

A fax that fails is not charged. The prepaid key gets the hold back. Destinations are the United States and Canada, including US territories. Premium-rate numbers such as 900 and 976 are rejected.

## Prices

FaxFair charges its regular published prices. There is no monthly fee.

- **Standard:** $2.49 for a fax of up to 40 pages.
- **Priority:** $4.98. A priority fax is sent before other agent faxes. A fax from the FaxFair website or the iPhone app is still sent first.

The written price list is [https://faxfair.com/agents.txt](https://faxfair.com/agents.txt). Agent instructions are at [https://faxfair.com/docs/agents/](https://faxfair.com/docs/agents/).

## Setup

1. Buy a prepaid FaxFair API key by card. The smallest top-up is $5. The steps are on [https://faxfair.com/docs/api/](https://faxfair.com/docs/api/). The key is shown once.
2. In Cursor, open the FaxFair plugin's configuration and set `FAXFAIR_API_KEY`.

Cursor substitutes that value into the `Authorization: Bearer` header. The key stays in the plugin configuration. Do not paste it into chat, and do not commit it.

The API reference is [https://faxfair.com/docs/api/](https://faxfair.com/docs/api/). How an agent sends a fax is [https://faxfair.com/docs/agents/](https://faxfair.com/docs/agents/). The price file is [https://faxfair.com/agents.txt](https://faxfair.com/agents.txt).

## License

MIT. See [LICENSE](LICENSE).
