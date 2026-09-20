# Trip planning decisions

## Core decisions

### Retain an honest plan

- [Keep implementation proposals separate from working software](#keep-implementation-proposals-separate-from-working-software).
- [Treat checkout links as the initial ticketing boundary](#treat-checkout-links-as-the-initial-ticketing-boundary).
- [Start with bounded geography and label data limitations](#start-with-bounded-geography-and-label-data-limitations).

## Details

### Keep implementation proposals separate from working software

[README.md](README.md) explicitly identifies this as a plan, with no routing or ticketing implementation. The [overview](plan/00-overview.md) proposes a shared core with thin terminal and phone clients. Do not describe that architecture as a deployed service.

### Treat checkout links as the initial ticketing boundary

The plan proposes route comparison and links into a seller’s checkout; unattended ticket sales are not established. Personal-account purchase automation is only an optional, separately confirmed later idea. Recheck dated provider and access assumptions before implementing it.

### Start with bounded geography and label data limitations

The proposed first version chooses one region with usable transit feeds and labels scheduled times when live data is absent. Long-distance multimodal routing is outside that initial scope. These choices address data and operating complexity; keep them explicit if the project resumes.
