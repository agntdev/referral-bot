# Referral Bot — Bot specification

**Archetype:** custom

**Voice:** warm and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram referral bot where users invite friends via a unique start parameter link and both earn credits as rewards.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- End users who invite friends via Telegram.

## Success criteria

- Users can generate a unique referral link via /start or a button.
- Users can share the link with friends.
- When a friend joins via the link, both users receive a credit.
- Users can view their current credit balance.
- Referral relationships are persisted (who referred whom).

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and generate a unique referral link.
- **My credits** (button, actor: user, callback: credits:show) — Show the user's current credit balance.
  - outputs: credit_balance
- **Referral link** (button, actor: user, callback: referral:show) — Show the user's unique referral link/code.
  - outputs: referral_link

## Flows

### User referral flow
_Trigger:_ User shares referral link

1. User starts the bot via /start or a button.
2. Bot generates a unique referral link (e.g., t.me/bot?start=ref123).
3. User shares the link with friends.
4. Friend joins via the link.
5. Bot records the referral relationship and credits both users.

_Data touched:_ user, referral_relationship, credit_balance

### Credit display flow
_Trigger:_ User clicks 'My credits' button

1. User clicks 'My credits' button.
2. Bot retrieves the user's credit balance.
3. Bot displays the balance to the user.

_Data touched:_ credit_balance

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user** _(retention: persistent)_ — A Telegram user who can refer friends and earn credits.
  - fields: telegram_id, credit_balance
- **referral_relationship** _(retention: persistent)_ — Records which user referred which friend.
  - fields: referrer_id, referred_id
- **credit_balance** _(retention: persistent)_ — The number of credits a user has earned.
  - fields: user_id, balance

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Notifications

- Users receive credit-earned notifications in Telegram when a friend joins via their referral link.

## Permissions & privacy

- The bot stores user Telegram IDs and credit balances in a simple key-value store (SQLite or in-memory).
- Referral relationships are stored to track who referred whom.
- No external payment processing or sensitive data is handled.

## Edge cases

- What happens if a user tries to refer themselves?
- What happens if a user tries to refer a friend who already joined?
- What happens if the referral link expires or is invalidated?
- What happens if the bot restarts and loses in-memory data?
- What happens if a user's credit balance is negative or invalid?

## Required tests

- Dialog-level acceptance test: User starts the bot, generates a referral link, shares it, and both users receive credits.
- Dialog-level acceptance test: User clicks 'My credits' button and sees their balance.
- Dialog-level acceptance test: User tries to refer themselves and receives an error.
- Dialog-level acceptance test: User tries to refer a friend who already joined and receives an error.

## Assumptions

- Credits are stored in a simple key-value store (SQLite or in-memory); no external database needed for MVP.
- Referral link is a Telegram start parameter (e.g., t.me/bot?start=ref123); no custom domain or tracking service required.
- One-time credit per referral; no recurring or volume-based rewards.
- No referral expiration or expiry logic.
- No admin panel; only user-facing referral flow.
