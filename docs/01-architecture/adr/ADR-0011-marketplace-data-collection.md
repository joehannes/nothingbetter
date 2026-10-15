# ADR-0011 · Collecting data from marketplaces (Uber Eats, PedidosYa, DoorDash)

- Status: **Proposed — owner decision required** (2026-10-15). Default until decided: **Option A**.
- Deciders: Founder + counsel

## Context
The founder proposes collecting foods, prices and WhatsApp numbers from Uber Eats / PedidosYa in the background (not for ordering) to bootstrap the food graph. Findings ([research §3.2](../../00-product/02-research-and-partner-landscape.md#32-marketplace-data-uber-eats-pedidosya-doordash--findings)):

- **Uber US terms** prohibit using "any content available through the Services, whether or not you are logged in, for any commercial purpose or for developing software… without Uber's prior written permission".
- **PedidosYa DR terms** prohibit copying, reusing or extracting all or part of the portal's content and cite copyright/industrial-property law.
- **Bot protection:** both serve challenges/denials to automated clients (observed 2026-10-15).
- **US law:** public-page scraping is generally not a CFAA violation (*hiQ*); *Meta v. Bright Data* (2024) held Meta's then-terms didn't bind logged-out scrapers — but Uber's terms now expressly reach logged-out commercial use. Remaining exposure: breach of contract (enforceability uncertain), tortious interference, trespass to chattels, copyright in photos/descriptions, IP blocking, cease-and-desist. DR law (copyright, industrial property, Ley 53-07 on high-tech crimes, Ley 172-13 on personal data) needs local counsel.
- **Data quality:** Uber Eats doesn't publish restaurant phone numbers; app prices run above in-store prices; marketplace menus change without notice.
- **WhatsApp:** harvested numbers don't meet WhatsApp's opt-in requirement.

## Options

| Option | Description | Risk | Value |
|---|---|---|---|
| **A. No marketplace collection** (default) | Food graph from open data (Overture, OSM), Google Places at request time, an AI "Neighborhood Scout" reading restaurants' own websites, WhatsApp menus, owner-authorized Google Business Profile menus, POS, licensed nutrition data ([AI/12](../../03-ai/12-restaurant-discovery-and-data-sourcing.md)) | Low | Measured: Overture lists 12,553 food places in a Miami-Dade box (88 % with phone) and 2,286 in Gran Santo Domingo (85 % with phone); menus are the gap, covered by website research + kitchen confirmation |
| **B. Human market research only** | Staff browse marketplaces to *identify* restaurants worth contacting; no automated collection; no copying of menus/prices/photos into our systems; outreach via the restaurant's own public channels with opt-in capture | Low–medium (Uber terms bar commercial use of content; counsel to confirm that noting a restaurant's existence for outreach is acceptable) | Faster prospect list |
| **C. Automated logged-out collection of facts** (names, cuisines, item names, price levels) at low rate, honoring robots.txt, **stopping on any challenge/denial**, no photos/descriptions, never shown to users as our menu, deleted after onboarding | Medium–high (contract/tort claims; in practice blocked by bot protection) | Limited: prices unreliable, no phone numbers |
| **D. Circumventing bot protection, logged-in scraping, or harvesting phone numbers for WhatsApp** | — | **Rejected** | — |

## Decision (proposed)
Adopt **A**, allow **B** with counsel sign-off. **C** only with written counsel approval and as a disabled-by-default adapter (`MarketplaceResearchAdapter`, feature flag `marketplace_research`, hard guardrails above). **D** is out of scope for this codebase.

## Consequences
- Supply-side work (outreach, onboarding visits, WhatsApp menu capture) is on the critical path for Phase 1.
- Ingestion pipeline treats any marketplace-derived record as `provenance=marketplace_research`, `trust=low`, never orderable until the kitchen confirms its menu.
