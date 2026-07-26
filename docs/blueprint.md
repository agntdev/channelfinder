# Channel Link Bot — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets the owner register a single channel (name + link). Users searching the channel name receive a reply with a button linking to the channel, which auto-deletes after 5 minutes. Owner controls registration and gets update confirmations.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- bot owner
- Telegram users

## Success criteria

- Owner receives confirmation when channel is registered/updated
- Users receive auto-deleted channel link messages after 5 minutes
- Bot validates channel links are valid Telegram URLs

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: owner, command: /start) — Show owner controls (only visible to owner)
- **/register** (command, actor: owner, command: /register) — Register/update channel (requires name and link arguments)
- **/unregister** (command, actor: owner, command: /unregister) — Remove stored channel
- **Channel search** (command, actor: user, command: /text) — Any user can send text to search for registered channel

## Flows

### Owner registration
_Trigger:_ /register

1. Owner sends /register with name and link
2. Bot validates link format
3. Bot stores channel info
4. Bot sends confirmation in same chat

_Data touched:_ owner record

### Public search
_Trigger:_ text message

1. User sends text message
2. Bot checks for channel name match (case-insensitive)
3. If match: send channel link with button
4. Schedule message deletion after 5 minutes

_Data touched:_ search query, search result message

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **owner record** _(retention: persistent)_ — Stores owner Telegram ID, registered channel name, link, and last update timestamp
  - fields: telegram_id, channel_name, channel_link, last_updated
- **search query** _(retention: none)_ — User-submitted text for channel name matching
  - fields: query_text, matched
- **search result message** _(retention: session)_ — Bot's reply containing channel link and deletion schedule
  - fields: message_id, scheduled_delete_time

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /start
- /register
- /unregister

## Notifications

- Owner receives registration/update confirmation in same chat
- Owner receives error messages for invalid links

## Permissions & privacy

- Only owner can modify channel registration
- Bot only deletes its own messages after 5 minutes

## Edge cases

- No channel registered when user searches
- Invalid link format provided in /register
- Case-insensitive matching edge cases
- Message deletion fails due to Telegram limits

## Required tests

- End-to-end test of owner registration flow with valid/invalid links
- Test public search with matching/non-matching queries
- Verify message auto-deletion after 5 minutes

## Assumptions

- Owner is determined by /register command issuer
- Match uses case-insensitive substring with whitespace tolerance
- Only bot messages are deleted, not user messages
