---
name: send-fax
description: >
  Send a fax with FaxFair to a United States or Canadian number. Use whenever
  the user wants to send a fax, quote a FaxFair price, or check a fax that
  FaxFair sent.
---

# Send a fax with FaxFair

Use this skill whenever the user wants to send a fax to a number in the United States or Canada, including US territories. FaxFair's MCP tools are `quote_fax`, `send_fax`, and `get_fax_status`. Call those tools by those names. Do not invent tools, arguments, prices, fax ids, or statuses.

The plugin talks to FaxFair's hosted server at `https://api.faxfair.com/mcp`. Cursor sends the prepaid API key as `Authorization: Bearer ${FAXFAIR_API_KEY}` from the plugin configuration. Never invent a key. Never ask the user to paste a key into chat. Never put a key in a tool argument, a file, a command, or a message. If a call fails because the key is missing or rejected, point the user to the FaxFair plugin's configuration (Customize, or Plugins → Configure) and to https://faxfair.com/docs/api/. If the user pastes a key anyway, do not repeat it or send it anywhere. Tell them to delete it from the chat, set `FAXFAIR_API_KEY` in the plugin configuration, and treat the pasted key as exposed.

## Prices

FaxFair charges its regular published prices:

- Standard: **$2.49** for a fax of up to 40 pages (249 cents, 2,490,000 USDC atomic units). USDC has 6 decimals.
- Priority: **$4.98** (498 cents, 4,980,000 USDC atomic units). A priority fax is sent before other agent faxes. A fax from the FaxFair website or app is still sent first.

Only `send_fax` charges. `quote_fax` and `get_fax_status` are free. The price is held when FaxFair accepts the fax and taken when it is delivered. A failed fax is not charged: the prepaid key gets the hold back.

The price to show the user is the price `quote_fax` returns for that fax. Do not substitute a remembered number for the quote. If the quote does not match the published prices above, show the quote and wait for the user to confirm that amount before sending.

## What you need before quoting

- A fax number in the United States or Canada. Premium-rate numbers such as 900 and 976 are rejected, and so are numbers outside those countries. The published docs use `+12125550100` as an example; the live `quote_fax` tool rejects that number (`invalid_number`, reason `special_service`). Do not use it.
- A page count from 1 to 40. Ask the user how many pages the PDF has if you do not know. Do not guess. More than 40 pages is `too_many_pages` on this API.
- For sending, an `https` URL to the PDF. `send_fax` takes `pdf_url` and does not accept a file upload. If the user only has a local file, ask for an `https` link. Do not upload the file yourself and do not call the HTTP API with the key.

`priority` is optional. Omit it, or pass `false`, for the standard price. Pass `true` only when the user wants the priority tier.

## quote_fax

Call `quote_fax` first, every time, before `send_fax`. It does not send the fax and does not charge.

Arguments:

| Argument | Required | Meaning |
| --- | --- | --- |
| `to` | yes | Fax number in the United States or Canada. |
| `pages` | yes | Integer page count, minimum 1. |
| `priority` | no | `true` to price the priority tier. The reply still lists both tiers. |
| `payer` | no | Wallet address, only if that wallet has credit from a failed USDC payment. Do not ask for a wallet. Do not pass `payer` for a prepaid API key. |

Show the user both tiers from the quote, and which tier would be charged. An `invalid_number` error means there is no price. Tell the user the tool's message and stop. Do not send.

## Confirm, then send_fax

`send_fax` is the only call that can charge the prepaid key. Do not call it until the user has confirmed this exact fax:

- the destination number
- the page count
- the `https` PDF URL
- standard or priority
- the price from `quote_fax`

Wait for a clear yes. A request to "send a fax" before the quote was shown is not confirmation.

Arguments:

| Argument | Required | Meaning |
| --- | --- | --- |
| `to` | yes | Same number you quoted. |
| `pdf_url` | yes | `https` link to the PDF. |
| `priority` | no | `true` for the priority price. Omit it for standard. It must match the tier the user confirmed. |
| `idempotency_key` | no | 8 to 64 letters, digits, hyphens, or underscores. A retry with the same key returns the same fax. Set one before the first `send_fax` and reuse it if that call times out. Do not start a second fax with a new key unless the user asks for another fax. |

Do not pass a payment signature, a card number, or the API key as an argument. The plugin header is how the prepaid key is sent.

If `send_fax` returns 402, or says the key is missing or the balance is too low, stop. Point the user to the plugin configuration and to https://faxfair.com/docs/api/ for a card top-up of at least $5. Do not try to pay with USDC from this skill, and do not ask the user to paste a key.

## get_fax_status

After `send_fax` succeeds, call `get_fax_status` with the id `send_fax` returned. The argument is `fax_id`. This call is free.

Status is `queued`, `sending`, `delivered`, or `failed`. The result also includes the page count, the price, and the last four digits of the number. The PDF is not in the result. FaxFair deletes the PDF from its servers once the fax is delivered or fails.

Report that status to the user. If it is still `queued` or `sending`, say it is not delivered yet and that a failed fax is not charged. Check again when the user asks. Do not poll in a tight loop.

Anyone with the fax id can read that status. Treat the id as a private link. Share it with the user who asked for the fax, and do not publish it.

## Errors

An error has a code and a message. Codes include `invalid_number`, `too_many_pages`, and `rate_limited`. Show the message. Do not retry a rejected number or an oversized PDF. On `rate_limited` or HTTP 429, wait for the time the server gives you, then try the same read or the same idempotent send once. Do not fire a new `send_fax` to get around the limit.
