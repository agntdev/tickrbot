# Crypto Watch & Alerts Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto prices with customizable alerts. Users manage watchlists of tickers, set price threshold or percent-change alerts with cooldowns, configure quiet hours, and receive optional morning summaries. The owner gains visibility into usage stats and alert activity while maintaining user data privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto watchers
- crypto price monitoring enthusiasts
- Telegram bot users seeking financial tracking

## Success criteria

- users receive accurate price alerts per configured rules
- owners view aggregated usage statistics
- watchlist management works without data leakage between users

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Onboarding menu with privacy explanation and quick tips
  - inputs: none
  - outputs: welcome message with common coin buttons
- **Manage Watchlist** (button, actor: user, callback: watchlist:manage) — Add/remove coins from watchlist
  - inputs: ticker symbol, display name
  - outputs: updated watchlist confirmation
- **Add Alert** (button, actor: user, callback: alert:create) — Configure price threshold or percent-change alert
  - inputs: ticker, alert type, value, window
  - outputs: alert confirmation with cooldown details
- **/price** (command, actor: user, command: /price) — Request on-demand price for specific ticker or full watchlist
  - inputs: ticker symbol, all
  - outputs: current price with 24h% change
- **Configure Settings** (button, actor: user, callback: settings:quiet_hours) — Set quiet hours and morning summary preferences
  - inputs: start/end time, summary time
  - outputs: updated configuration confirmation

## Flows

### Onboarding
_Trigger:_ /start

1. Display privacy policy and quick tips
2. Show pre-seeded coin buttons
3. Prompt for initial watchlist selection

_Data touched:_ user profile

### Alert Creation
_Trigger:_ Add Alert button

1. Select coin from watchlist
2. Choose alert type (threshold/percent)
3. Enter value/window parameters
4. Confirm alert creation

_Data touched:_ alert rule

### Price Check
_Trigger:_ /price command

1. Parse ticker parameter
2. Fetch current price
3. Display price with 24h% change

_Data touched:_ user profile, watchlist item

### Alert Delivery
_Trigger:_ Price update event

1. Check all active alert rules
2. Calculate price changes
3. Send alert if threshold met and cooldown expired

_Data touched:_ alert rule, user profile

### Quiet Hours Handling
_Trigger:_ Scheduled check

1. Check current time against user's quiet hours
2. Queue/delay alerts as configured

_Data touched:_ user profile, alert rule

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User preferences and configuration
  - fields: chat_id, timezone, quiet_hours_start, quiet_hours_end, summary_time
- **watchlist_item** _(retention: persistent)_ — Tracked cryptocurrency ticker
  - fields: symbol, display_name, alert_rules
- **alert_rule** _(retention: persistent)_ — Price alert configuration
  - fields: type, direction, target_price, percent_value, window, enabled, last_alerted_at
- **owner_stats** _(retention: persistent)_ — Aggregated usage metrics
  - fields: total_users, alert_counts, recent_failures

## Integrations

- **Telegram** (required) — Bot API messaging
- **Price Feed API** (required) — Cryptocurrency price data
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View aggregated usage statistics
- Request recent alert logs
- Access top 10 alert rules and tickers

## Notifications

- Telegram direct message alerts
- Owner channel summaries
- Error notifications for failed price fetches

## Permissions & privacy

- All user data encrypted at rest
- No cross-user data sharing
- Alert rules stored with user-specific access

## Edge cases

- Unknown ticker symbol handling
- Price feed retries on failure
- Quiet hours alert queuing
- Cooldown expiration calculations

## Required tests

- End-to-end alert flow from rule creation to delivery
- Quiet hours boundary testing
- Watchlist management with 10+ coins

## Assumptions

- Price feed has 99.9% uptime
- Users understand crypto ticker symbols
- Cooldown prevents alert spamming
