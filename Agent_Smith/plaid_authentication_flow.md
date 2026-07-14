# Understanding the Plaid Authentication Flow

Plaid's auth model is fundamentally different from Whoop's or Google's. There's no single redirect that carries an authorization code — instead there's a **link_token → public_token → access_token** handshake split across your server and a client-side SDK. This doc walks through each hop with representative JSON packets.

## The three tokens

| Token | Created by | Lives where | Lifespan |
|---|---|---|---|
| `link_token` | Your server | Passed to frontend | ~4 hours, single use to open Link |
| `public_token` | Plaid Link (browser) | Passed from frontend to your server | 30 minutes, single use |
| `access_token` | Plaid, via exchange | Your server only, never sent to frontend | Does not expire; static until revoked or item breaks |

The `access_token` is the sensitive, long-lived credential — equivalent to Whoop's `access_token`/`refresh_token` pair, except there's no rotation. It should never reach the browser.

## Step 1 — Server creates a link_token

Your Flask server calls `POST /link/token/create` with your `client_id`/`secret`. This is analogous to Whoop's `get_authorization_url()`, but instead of building a redirect URL, it returns a token the frontend hands to the Link SDK.

**Request (your server → Plaid):**
```json
{
  "client_id": "5f2c...",
  "secret": "b91a...",
  "client_name": "neo4j-flask-app",
  "language": "en",
  "country_codes": ["US"],
  "user": { "client_user_id": "brendan-local-user" },
  "products": ["transactions"],
  "webhook": "https://yourapp.com/plaid/webhook"
}
```

**Response (Plaid → your server):**
```json
{
  "link_token": "link-sandbox-3c1e...",
  "expiration": "2026-07-14T18:30:00Z",
  "request_id": "m8MDnv9j4z"
}
```

Your server passes just the `link_token` value down to the page that hosts the Link SDK.

## Step 2 — Frontend opens Link with the token

This part never touches your Flask routes — it's the Plaid Link JS SDK running in the browser. The SDK opens a hosted UI (bank picker, credential entry, MFA) and on success calls your `onSuccess` handler with a `public_token`.

**What Link hands your JS (browser-side only, not a network packet to your server yet):**
```json
{
  "public_token": "public-sandbox-9e21...",
  "metadata": {
    "institution": { "name": "Chase", "institution_id": "ins_56" },
    "accounts": [
      { "id": "acct_1", "name": "Plaid Checking", "mask": "0000", "type": "depository" }
    ],
    "link_session_id": "3d6a1c7e-..."
  }
}
```

Your JS then POSTs the `public_token` to your server — this is the request your `exchange_public_token` route receives.

## Step 3 — Server exchanges public_token for access_token

This is the closest analog to Whoop's `flow.fetch_token(...)` call, but it's triggered by an AJAX call from your own frontend rather than by a browser redirect carrying a `code` param.

**Request (your server → Plaid):**
```json
{
  "client_id": "5f2c...",
  "secret": "b91a...",
  "public_token": "public-sandbox-9e21..."
}
```

**Response (Plaid → your server):**
```json
{
  "access_token": "access-sandbox-7a44...",
  "item_id": "M5eVxNMj3nS...",
  "request_id": "Aim3b7IjcHb"
}
```

The `item_id` identifies this specific bank connection. Store `access_token` keyed by `item_id`, not as a single flat `plaid` credential — a user can link more than one bank, and each link produces its own `item_id`/`access_token` pair. This is the biggest structural difference from Whoop's credentials file, where there's just one token set per service.

## Step 4 — Using the access_token

Every subsequent Plaid API call sends the `access_token` in the request body (not as a bearer header, unlike Whoop):

**Request — fetch accounts:**
```json
{
  "client_id": "5f2c...",
  "secret": "b91a...",
  "access_token": "access-sandbox-7a44..."
}
```

**Response:**
```json
{
  "accounts": [
    {
      "account_id": "vokyE5Rn6oc...",
      "name": "Plaid Checking",
      "mask": "0000",
      "type": "depository",
      "subtype": "checking",
      "balances": { "available": 100.0, "current": 110.0, "iso_currency_code": "USD" }
    }
  ],
  "item": { "item_id": "M5eVxNMj3nS...", "institution_id": "ins_56" },
  "request_id": "Wo6Ug"
}
```

## No refresh tokens — webhooks instead

Whoop's access token expires on a clock, so you refresh proactively. Plaid's `access_token` doesn't expire on a schedule at all — it stays valid until the user revokes access or the connection breaks (bank changed their password, MFA reset, credentials rotated on the bank's end). Plaid tells you about that reactively, via a webhook POST to the URL you registered in Step 1.

**Example webhook — item needs re-auth:**
```json
{
  "webhook_type": "ITEM",
  "webhook_code": "ITEM_LOGIN_REQUIRED",
  "item_id": "M5eVxNMj3nS...",
  "error": {
    "error_code": "ITEM_LOGIN_REQUIRED",
    "error_message": "the user needs to re-enter their legacy credentials"
  }
}
```

**Example webhook — new transactions available:**
```json
{
  "webhook_type": "TRANSACTIONS",
  "webhook_code": "SYNC_UPDATES_AVAILABLE",
  "item_id": "M5eVxNMj3nS...",
  "initial_update_complete": true,
  "historical_update_complete": true
}
```

When you see `ITEM_LOGIN_REQUIRED`, the fix isn't a token refresh call — it's re-opening Plaid Link in **update mode**, which requires generating a new `link_token` that includes the existing `access_token`:

```json
{
  "client_id": "5f2c...",
  "secret": "b91a...",
  "user": { "client_user_id": "brendan-local-user" },
  "access_token": "access-sandbox-7a44..."
}
```

This produces a fresh `link_token` that opens Link directly into "please re-enter your credentials for this bank" mode, rather than a full new bank search.

## OAuth institutions (Chase, some others)

A subset of banks route through their own OAuth login inside Link, which requires bouncing the browser out to the bank and back. This needs a `redirect_uri` registered in both your `link_token/create` call and the Plaid dashboard exactly. Practically, this just means:

1. `link_token/create` includes `"redirect_uri": "https://yourapp.com/plaid/oauth2callback"`.
2. The bank's OAuth step redirects the browser to that URL with `?oauth_state_id=...` appended.
3. Your `oauth2callback` route doesn't exchange anything — it just needs to get the browser back to the page hosting the Link SDK, with that query string intact, so Link can resume the same session using the state id.

## Security note: verifying webhooks

Anyone who finds your webhook URL can POST fake JSON to it — Plaid signs real webhook bodies with a JWT in the `Plaid-Verification` request header, and you're expected to verify that signature (fetching Plaid's current signing key, checking the body hash) before trusting a payload. This is a gap in the stub I wrote and worth closing before relying on webhooks for anything that changes account state.

## Summary of what's structurally different from Whoop

- Token exchange is client-initiated (AJAX from your own frontend), not a redirect carrying a `code`.
- `access_token` doesn't expire and isn't rotated — no refresh helper needed.
- One user can have many `item_id`/`access_token` pairs; credentials need to be keyed accordingly.
- Staleness is reported reactively via webhooks, not discovered by checking an `expires_at` field.
- Recovery from a broken item is "re-open Link in update mode," not "call the token refresh endpoint."