# GlowEr Beauty Studio Bot — Bot specification

**Archetype:** booking

A Telegram bot for GlowEr beauty studio enabling clients to browse services, view portfolios with photo reviews, book appointments, and receive post-appointment review prompts. Studio admins can manage services, portfolio items, and respond to reviews while receiving booking notifications.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Clients of GlowEr beauty studio
- Studio staff/admins

## Success criteria

- Users can book appointments and receive confirmation/status updates
- Admins can manage services/portfolio and respond to reviews
- Post-appointment review prompts trigger with photo support
- Data persistence survives restarts for all entities

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with service browse, portfolio, reviews, and booking options
- **View Services** (button, actor: user, callback: services:list) — Browse available beauty services with durations and example photos
  - inputs: service category filter
  - outputs: service list with details
- **View Portfolio** (button, actor: user, callback: portfolio:gallery) — Browse portfolio items by category with image captions
  - inputs: category selection
  - outputs: image gallery with metadata
- **Leave a Review** (button, actor: user, callback: reviews:submit) — Submit photo reviews with ratings and optional staff tags
  - inputs: rating, text, photos
  - outputs: review confirmation with admin reply thread
- **/my_bookings** (command, actor: user, command: /my_bookings) — View and manage active bookings with cancellation/reschedule options

## Flows

### Service Booking
_Trigger:_ services:details

1. Select service
2. Choose optional staff member
3. Pick date from calendar
4. Select time slot
5. Confirm booking with summary

_Data touched:_ Booking, Service

### Post-Appointment Review
_Trigger:_ booking:completed

1. Send 1-hour post-appointment prompt
2. Collect rating/text/photos
3. Notify admins of new review

_Data touched:_ Review

### Admin Review Management
_Trigger:_ reviews:new

1. Send admin notification
2. Allow reply to review
3. Display reply in user thread

_Data touched:_ Review

### Booking Management
_Trigger:_ /my_bookings

1. List active bookings
2. Offer cancellation/reschedule options
3. Update booking status

_Data touched:_ Booking

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Service** _(retention: persistent)_ — Beauty service offering with pricing and duration
  - fields: name, description, duration, price, portfolio_examples
- **PortfolioItem** _(retention: persistent)_ — Studio work examples with photos and tags
  - fields: title, description, photos, tags
- **Review** _(retention: persistent)_ — Client feedback with photos and admin replies
  - fields: rating, text, photos, staff_tag, admin_replies
- **Booking** _(retention: persistent)_ — Appointment reservation with status tracking
  - fields: user_id, service_id, staff_id, datetime, status, contact_notes
- **Admin** _(retention: persistent)_ — Studio account with management permissions
  - fields: telegram_id, permissions

## Integrations

- **Telegram** (required) — Bot API messaging for notifications and interactions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/update/delete services and portfolio items
- Manage admin accounts and permissions
- View bookings and mark as completed
- Reply to reviews with public messages

## Notifications

- Booking status updates to users
- New booking/admin review alerts to studio admins
- Post-appointment review prompts

## Permissions & privacy

- Admin access restricted to configured Telegram accounts
- User data stored securely with GDPR-compliant retention
- Photo reviews anonymized if requested

## Edge cases

- User cancels within 2-hour window
- Missing photos in portfolio reviews
- Timezone mismatches during booking
- Concurrent booking edits by multiple admins

## Required tests

- End-to-end booking flow with confirmation/cancellation
- Admin review management workflow
- Post-appointment review prompt delivery
- Data persistence across bot restarts

## Assumptions

- Studio timezone defaults to owner's locale
- Business hours default to 09:00-19:00 Mon-Sat
- Cancellation window is 2 hours pre-appointment
