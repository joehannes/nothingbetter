# WF/05 · Location, GPS & Addresses

> Status: Draft v1 · Principle (from the earlier investigation): **detected position → suggested delivery location → user confirmation**. GPS ≠ delivery address.

## 1. Sources of location

| Platform | Primary | Fallback |
|---|---|---|
| iOS / Android (Tauri) | `tauri-plugin-geolocation` (requires `NSLocationWhenInUseUsageDescription` on iOS; Android location permissions auto-added) | Last saved place, IP geolocation |
| Desktop (Tauri) | WebView Geolocation API (where supported; macOS needs location usage string) | IP geolocation → "Is this right?" |
| Web | Browser Geolocation API (HTTPS, user prompt) | IP geolocation |
| Any | Saved places (Home, Work, Mom's) | Manual search |

Ask for location **in context** ("To find kitchens near you, can I use your location?") — never on cold start before value is shown.

## 2. Flow

```text
1. Locate (GPS/IP) → lat/lng ± accuracy
2. Reverse-geocode → candidate addresses + nearby saved places
3. Suggest: "Deliver to Home (120 m away)?" or "Deliver here: 1450 Brickell Ave?"
4. Confirm/adjust: map pin drag, unit/apartment, buzzer, floor, gate code, drop-off notes, contactless
5. Validate: geocode → rooftop precision, service-area polygon check (PostGIS), deliverability flags (gated community, hotel, campus)
6. Save (optional) as named place; set as default for this household/time slot
```

## 3. Data

```ts
type Place = {
  id: string; label: "Home" | "Work" | string;
  address: { line1: string; line2?: string; city: string; region: string; postalCode: string; country: "US" | "CA" };
  location: { lat: number; lng: number; precision: "rooftop" | "interpolated" | "approximate" };
  dropoff: { instructions?: string; buzzer?: string; gateCode?: string; contactless?: boolean; handoff: "door" | "lobby" | "curb" | "meet" };
  timezone: string; serviceAreaIds: string[];
};
```

Gate/buzzer codes are encrypted and shared only with the assigned courier during an active order.

## 4. Service areas & time zones

- Service areas are PostGIS polygons per metro with enabled modes (single, synchronized, concierge) and fee tables.
- Every place and merchant location carries an IANA time zone; all "ready by" and promise windows computed in local time and stored in UTC.

## 5. Geo services

| Need | Service |
|---|---|
| Autocomplete & geocoding | Mapbox Search/Geocoding (Google Places optional; mind caching & display terms) |
| Routes & ETAs | Mapbox Directions; Matrix API for multi-stop planning |
| Map display | MapLibre GL with Mapbox/MapTiler tiles |
| Sunset/sunrise (iftar, Shabbat) | Astronomical calculation library (no API needed) |
| Weather | Weather API (e.g., Open-Meteo or a commercial provider) for comfort/cooling suggestions |

## 6. Live tracking

- Courier positions from provider webhooks or courier app; throttled to 1 update/10 s to clients.
- Customer sees the courier only during active delivery; positions purged after 30 days.

## 7. Privacy

- Precise GPS used only to suggest an address; not stored unless saved as a place.
- No background location for eaters.
- Location history not used for profiling beyond saved places and order addresses.
