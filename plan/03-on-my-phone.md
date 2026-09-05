# On my phone

## Recommendation: a Telegram bot on the shared API spine

The trick is to keep one **self-hosted HTTP API** wrapping the trips core, and make
every phone option a thin, swappable frontend over it. With that spine in place, a
**Telegram bot** is the lowest-friction way to carry trips in my pocket *today*:

- **Nothing to install** — Telegram is already on the phone.
- **Free push** to the lock screen via Telegram's own infrastructure — no APNs, no
  Apple Developer account ($99/yr), no PWA install ritual.
- **No public server needed.** With long-polling the home box dials out to Telegram;
  there's no inbound port, no port-forwarding, no TLS cert to manage.
- **Trivial auth** — allowlist my own Telegram user ID.
- **Claude Code drives the same API**, or can even post through the bot for
  proactive nudges ("your train's delayed").

## Why not the others

| Option | Verdict |
|---|---|
| **PWA + API** | Best *visual* UI, but iOS push needs manual Add-to-Home-Screen + permission and breaks on reinstall. Add it *later* for maps, not as the entry point. |
| **iOS Shortcuts** | Great for one-shot actions via Siri/Back Tap, but can't *receive* push and has no real UI. A nice optional glue layer. |
| **Tailscale + SSH** | Lowest build effort (runs the CLI unchanged) but highest day-to-day friction — typing CLI on a phone. It's "remote into my desk," not "in my pocket." Keep as a power-user escape hatch. |
| **Termux** | Android-only; **doesn't exist on iOS**. Not an option. |
| **Native / Expo app** | Highest ceiling (native maps, APNs) but highest cost + $99/yr. Overkill until a polished visual app is the point. |

## The one real weakness

Chat is a poor surface for the **visual route comparison and maps** that are the
product's actual promise. So:

- Scope Telegram to **commands + notifications** (next train, delays, "buy it").
- Design the shared API so a **PWA map view** can be added with no backend change,
  and have the bot deep-link into it for visual results.

## Derisking

- **Home-server uptime** (the bot dies silently if the box reboots or the ISP drops
  — exactly when travelling) → run it supervised (`launchd`/Docker `restart=always`)
  with a heartbeat; consider a small always-on Pi/mini-PC or a cheap VPS rather than
  the laptop.
- **A money-spending endpoint on a phone** → keep `buy` behind an explicit confirm
  step and a separate scoped secret; never expose buy on the open internet
  (long-polling already avoids any public endpoint).

## Interfaces

```
# the shared spine
POST /route {from,to,modes,prefs} -> ranked options
POST /book  {option_id} -> {needs_confirm, order};  POST /book/confirm {order_id}

# Telegram bot (long-poll loop on the home server, no inbound port)
while true: getUpdates(offset, timeout=50) -> if from.id == ME: dispatch -> call API -> reply

# push: server event (delay, "ticket bought") -> sendMessage(ME, text, inline_keyboard)
```

Full comparison and citations: [research/mobile-delivery.md](research/mobile-delivery.md).
