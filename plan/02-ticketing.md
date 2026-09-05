# Ticketing — the hard half

## Verdict: you can't buy a ticket end-to-end from a terminal as a solo dev

Both researchers, including an adversarial red-team, landed on the same answer with
high confidence: **refuted**. "Integrate with Trainline and buy tickets" is not
achievable the way it sounds. Four independent walls:

1. **No booking API for individuals.** Trainline's only real API (Partner Solutions
   Global API) is enterprise, contact-sales, ~12-week integration, registered
   business required. Its only individual-open channel is the *affiliate* program —
   deep-links and commission, not transactions. Every aggregator that *can* sell
   (Distribusion, Omio, Rail Europe, SilverRail, Amadeus, Lyko) is a signed B2B
   reseller contract. The free self-serve APIs (SNCF/Navitia, DB Marketplace, UK
   Darwin, Renfe) are **information and price lookup only** — none can purchase.
2. **Terms of service.** Trainline §12.2 explicitly prohibits automated agents,
   scripts, scraping, and building a competing service.
3. **Payment.** Taking the card yourself puts you in PCI-DSS scope with
   chargeback/fraud liability. The only escape is letting someone else be
   merchant-of-record — i.e. not you.
4. **UK rail's moat.** Selling National Rail tickets needs RDG/RSP accreditation
   (~£1,000/day, re-accredited every ≤3 years) and an approved Ticket Issuing
   System. This *is* Trainline's moat; it's categorically out of reach.

## The realistic path

Split the dream in two. Planning is fully doable; buying is a sanctioned hand-off.

```
trips plan "Paris -> Lyon" --date 2026-07-10   # open-data itineraries (free, legit)
trips price <id>                               # fares via operator lookup
trips checkout <id>                            # open() an affiliate deep-link to
                                               # the operator's own payment page
```

- **Phases 0–1** (plan → deep-link to checkout) are 100% legitimate, contract-free,
  and monetizable: register as a Trainline affiliate via Partnerize (no fee, ~3%
  commission). The user finishes payment in Trainline's PCI-compliant flow and the
  mobile ticket lands in Trainline's app — which satisfies "carry it on my phone."
- Honest caveat: an affiliate deep-link prefills a *search/journey*, not a fully
  populated cart. So it's **"one command to the right checkout,"** not unattended
  purchase.

## The optional, grey-area buy

```
trips buy <id> --account me --confirm   # browser-automate MY OWN checkout
```

The *only* way an individual truly buys from a terminal is to drive your own
account's web checkout with browser automation (Playwright / claude-in-chrome).
It's real but:

- **Brittle** — Cloudflare/bot-detection, CAPTCHA, 3-D Secure step-ups.
- **ToS grey** — personal use mitigates but doesn't license it.
- **Fulfilment may defeat it** — many EU/UK tickets are mobile-app-locked (Aztec/QR
  only inside the operator app), so a terminal-bought ticket can be unusable
  outside that app.

Keep it optional, personal-use, and **gated behind an explicit confirm-before-pay
prompt** (paying is the irreversible step). Before committing to it, spend one day
spiking thetrainline.com / sncf-connect.com to check bot-detection survivability
and whether a print-at-home PDF (PDF417) fulfilment option exists for the routes I
care about.

## If "trips" ever becomes a product

Selling to others flips everything: incorporate, sign a Distribusion/Omio reseller
contract (prototype free against the Distribusion OSDM sandbox first), and inherit
an accredited partner's merchant-of-record + PCI scope rather than building your
own. That's a different project — flagged, not planned here.

Full evidence and citations: [research/ticketing.md](research/ticketing.md) and the
adversarial [research/redteam-ticketing.md](research/redteam-ticketing.md).
