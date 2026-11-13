# WhatsApp webhook signature check for n8n

An n8n workflow that checks Meta's `X-Hub-Signature-256` header on incoming WhatsApp Cloud API webhooks. Invalid requests get a 401. Valid ones continue to your own logic.

Without this check, anyone who knows your webhook URL can send fake messages to your workflow.

## What it does

1. **GET webhook:** Answers Meta's verification challenge, but only if `hub.verify_token` matches your token.
2. **POST webhook:** Receives messages with the raw body enabled.
3. **Check signature:** Calculates an HMAC SHA256 of the raw body with your app secret and compares it to the header using a constant-time comparison.
4. **Valid:** Replies 200 right away, then continues to the "Your logic here" node.
5. **Invalid:** Replies 401 and stops.

## Setup

1. Import `whatsapp-signature-check.json` into n8n.
2. Set two environment variables:
   - `WHATSAPP_APP_SECRET`: your Meta app secret
   - `WHATSAPP_VERIFY_TOKEN`: the verify token you enter in the Meta webhook setup
3. Self-hosted n8n: allow the `crypto` module in the Code node with `NODE_FUNCTION_ALLOW_BUILTIN=crypto`. Env access in nodes must not be blocked (`N8N_BLOCK_ENV_ACCESS_IN_NODE=false`).
4. Activate the workflow and use the production URL of the webhook as the callback URL in Meta.
5. Add your own nodes after **Your logic here**.

## Notes

- The hash must use the raw body. A re-serialized JSON body gives a different hash.
- The header looks like `sha256=<hex>`. The workflow compares the full string.
- Meta expects a fast 200 response, so the workflow replies before your logic runs.
- Test with a real Meta app before you rely on it in production.

## License

MIT
