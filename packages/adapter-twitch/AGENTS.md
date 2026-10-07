# AGENTS.md for `@chat-adapter/twitch`

Follow the repository-level `AGENTS.md` plus these package-specific rules.

## Scope

This package integrates Twitch channel chat and whispers through EventSub
webhooks and the Helix API. It runs as a Twitch "cloud chatbot": a dedicated
bot account, an app access token from the client credentials grant, and
one-time authorizations from the bot and each broadcaster. It uses the
EventSub webhook transport only, so it works on serverless platforms.

## Contracts

- Factory: `createTwitchAdapter`
- Adapter name: `twitch`
- Thread IDs: `twitch:{broadcasterUserId}` (chat room) and
  `twitch:whisper:{userId}` (whisper). A chat thread ID is also its channel ID.
- Lock scope: `channel`
- Handled EventSub types: `channel.chat.message` v1, `user.whisper.message` v1
- Required env vars: `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET`,
  `TWITCH_WEBHOOK_SECRET`, and `TWITCH_BOT_USERNAME` or `TWITCH_BOT_USER_ID`

## Tokens

- App access token (client credentials): EventSub subscriptions, Send Chat
  Message, Delete Chat Messages, Get Users. Cached in memory; a 401 clears it
  and retries once.
- User access token (`user:manage:whispers`): Send Whisper only. Static,
  provider function, or managed refresh persisted at `twitch:oauth:{clientId}`.

## Platform constraints

- Chat messages are plain text, one line, at most 500 characters.
- Whispers are at most 500 characters to first-time recipients and 10,000
  otherwise; the sender needs a verified phone number.
- Send Chat Message can return 200 with `is_sent: false`; surface
  `drop_reason` as an error.
- Chat events carry no timestamp; use `Twitch-Eventsub-Message-Timestamp`.
- Reject notifications older than 10 minutes. Chat SDK dedupes retries by
  message ID.
- No edits, reactions, typing indicators, uploads, or history API.

## Testing

Mock `fetch`; unit tests must not call Twitch. Add payloads to
`sample-messages.md` when adding an event shape.

```bash
pnpm --filter @chat-adapter/twitch test
pnpm --filter @chat-adapter/twitch typecheck
pnpm --filter @chat-adapter/twitch build
```

Never log access tokens, refresh tokens, client secrets, the webhook secret,
or webhook signatures.
