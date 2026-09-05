# trips — the plan

## The verdict, in one paragraph

The fun half of the dream is real; the hard half isn't what I thought. **"Google
Maps but better"** is genuinely buildable by one person, on open-source, with no
Google lock-in and near-zero per-use cost. **"Buy tickets from the terminal"** is
the wall: no rail seller offers a booking API to an individual, and scripting a
checkout breaks terms of service and hits payment/accreditation moats I can't
clear solo. **"Carry it on my phone"** is the easy part — a Telegram bot gives me
lock-screen push and zero-install access today. So the shippable product is *plan
and compare in the terminal, then one tap to a ready-to-pay checkout, with the
ticket landing in my pocket* — not literal unattended terminal payment.

## What's actually hard

| Piece | Feasible solo? | Why |
|---|---|---|
| Routing & maps | **Yes** | Open-source engines match Google for drive/walk/transit; the gap is *data*, not software |
| Buying tickets | **No, not end-to-end** | No individual-accessible booking API; scripting checkout breaks ToS + PCI + UK rail licensing |
| On my phone | **Yes** | Telegram bot = free push, no install, no public server needed |

The order of difficulty is the opposite of the order in the provocation. Detail
lives in the three deep-dives:

- **[01-routing-and-maps.md](01-routing-and-maps.md)** — the engine + data stack
- **[02-ticketing.md](02-ticketing.md)** — why buying is gated, and the realistic path
- **[03-on-my-phone.md](03-on-my-phone.md)** — the bot-on-a-spine delivery model

## Architecture: one core, many faces

Everything hangs off a single idea, the same one the email and slack tools use: a
**headless core** that every surface is a thin adapter over.

```
        ┌──────────────── trips core (the planner) ─────────────────┐
        │  routing (Valhalla) · transit (OTP2) · geocode (Pelias)   │
        │  fare lookup (open data) · checkout deep-links · ranking   │
        └───────────────────────────┬───────────────────────────────┘
                                     │  one small HTTP/CLI surface
        ┌──────────────┬─────────────┼─────────────┬─────────────────┐
     terminal      Telegram bot   Claude Code     PWA (later,     own-account
      (me)        (phone + push)  (automations)   visual maps)    buy (opt-in)
```

Pick the spine first; the frontends are cheap and swappable once it exists.

## The three core interfaces

```
# 1. plan — itineraries from open routing/transit data
planTrip(origin, dest, modes=[DRIVE,WALK,TRANSIT], departAt) -> Itinerary[]

# 2. price — fares from operator open-data lookups (no purchase)
price(itinerary) -> Fare[]

# 3. checkout — an affiliate-tagged deep-link into the operator's own payment flow
checkoutUrl(itinerary) -> URL        # the honest "buy" for v1
```

A fourth, `buy(itinerary, --confirm)`, browser-automates *my own* account checkout.
It's optional, personal-use, and gated behind an explicit confirm-before-pay step,
because it lives in a terms-of-service grey area (see 02).

## Derisking at a glance

| Dependency | Risk | What saves it |
|---|---|---|
| Live transit data (GTFS-RT) | High | Start in cities that publish it; elsewhere show *scheduled* times, labelled |
| Long-distance multimodal (Rome2Rio-style) | High | Out of scope for v1; link out for cross-border legs |
| Buying a ticket programmatically | High | Reframe to deep-link checkout; never become the seller |
| Self-hosting routing engines (ops) | Medium | One small VM, one metro at a time; managed tiles as a fallback |
| Phone delivery | Low | Telegram long-polling needs no public server |

## What I'd build first (MVP)

1. Stand up Valhalla on one OSM extract + OTP2 on one city's GTFS. Wrap both in a
   `planTrip` facade behind a tiny HTTP API.
2. A terminal command: `trips plan "A -> B" --date ...` returning ranked options.
3. A Telegram bot over the same API for phone access + push.
4. `trips checkout <id>` that `open`s an affiliate deep-link to the operator.

That's a useful tool end to end, with zero ToS or payment risk. Maps UI and the
opt-in personal-account `buy` come after.

## Open decisions (my guesses — easy to change)

- **Personal vs product.** I've assumed *trips* is for my own travel. If it's ever
  meant to sell tickets to others, the whole ticketing story changes (incorporation +
  a reseller contract). I built for personal use; say the word to flip it.
- **Region first.** I'd start with one city/region where transit data is good rather
  than going wide-and-shallow.
- **"Better" means UX + multimodal + privacy**, not live-traffic accuracy — Google
  still wins on live traffic and POI depth, and I'm conceding those for v1.
