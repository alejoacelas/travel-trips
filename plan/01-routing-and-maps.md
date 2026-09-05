# Routing & maps

## Recommendation

Build the whole map/routing layer on open-source and touch Google nowhere:

| Job | Use | Why not Google |
|---|---|---|
| Drive / walk / bike routing | **Valhalla** (OSRM if you need raw speed) | per-request fees, no caching allowed |
| Transit / multimodal | **OpenTripPlanner 2** on GTFS | Google transit is fine but locks you in |
| Basemap & rendering | **MapLibre GL** + **Protomaps** PMTiles | Google forbids showing its data on a non-Google map |
| Place search / geocoding | **Pelias** or **Photon** | Google Places caching + display ToS |

All free software on OpenStreetMap data; cost is one small VM (~$20–80/mo) and the
ops to keep it fed.

## Why not just use Google

Two clauses in Google's terms forbid the exact product I want:

- You may **not cache or store** results (only a `place_id` indefinitely, lat/lng
  for 30 days) — so I can't pre-compute or recombine routes.
- Anything displayed **must sit on a Google basemap** — which kills "a better,
  custom map."

On top of that, the universal $200/mo credit ended in March 2025, replaced by thin
per-SKU free caps. Google is the wrong foundation for "Maps but better" both
legally and economically. Use it, if ever, only for a text-only POI lookup that
never gets cached or mapped.

## The real wall: data, not engines

The routing math is solved. What's scarce:

- **Live transit (GTFS-RT).** Real-time arrivals are the thing that beats Google,
  but the feeds are per-agency, fragmented, and often absent. Where there's no
  RT feed, I fall back to *scheduled* times — still useful, just labelled honestly.
- **Long-distance, cross-border door-to-door** (the Rome2Rio domain — flights +
  rail + bus + ferry). No open engine does this, and the two APIs that do
  (Rome2Rio, Citymapper) have both closed to solo developers. **Out of scope for
  v1**; link out for those legs.

## Derisking

- **Transit data** → start only in cities with good GTFS(-RT); use Transitland /
  the Mobility Database to pick them. Don't promise live timing where the data
  isn't live.
- **Ops burden** → containerize; one metro extract at a time; OTP2 for transit and
  Valhalla for roads (Valhalla's own transit support still has open bugs). If
  self-hosting tiles is too much at first, start on Mapbox/MapTiler free tiers and
  swap to Protomaps later.
- **No live traffic** in OSM → present drive ETAs as free-flow estimates, labelled;
  a paid traffic source is a later, optional add.

## Interfaces

```
# Valhalla (road)
POST /route { locations:[{lat,lon}...], costing:'auto'|'pedestrian'|'bicycle' }
  -> { trip:{ legs:[{ shape, summary:{time,length}, maneuvers }] } }

# OTP2 (transit) — GraphQL
plan(from,to,date,time,transportModes:[WALK,TRANSIT])
  -> { itineraries{ duration legs{ mode route{shortName} legGeometry realTime } } }

# Your normalization facade (the bit you own)
planTrip(origin, dest, modes, departAt) -> Itinerary[]
  # fans out to Valhalla + OTP2, merges, ranks
```

Full options table, costs, and citations: [research/routing-and-maps.md](research/routing-and-maps.md).
