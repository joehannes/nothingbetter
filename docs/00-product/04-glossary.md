# 04 · Glossary

| Term | Meaning |
|---|---|
| **Avatar** | A profile of a person the user orders for (self, partner, child, parent, friend). Holds hard constraints, preferences, culture, health opt-ins. |
| **Household / Circle** | A group of avatars that often eat together (family, flatmates, office team). |
| **Seat** | An avatar's place at a specific meal; basket items are assigned to seats. |
| **Table view** | The signature UI showing a round table with seats and each person's dish. |
| **Session** | One conversation from opening the app to ordering or leaving. |
| **Situation Brief** | Structured snapshot of the moment: time, place, calendar event, weather, season, present avatars, mood, budget, urgency. Input to all AI faculties. |
| **Faculty** | A specialist AI skill (Nutrition, Natural Medicine, Culture, Occasion, Social, Inspiration, Logistics, Safety…) that returns structured advice. |
| **Council** | The set of faculties consulted for a session; results merged by the Synthesizer. |
| **Advice Card** | A faculty's structured output: hard constraints, boosts, cautions, ideas, evidence level, confidence. |
| **Orchestrator / Host** | The conversational agent the user talks to; owns tone, questions, tool calls and presentation. |
| **Dials** | Live parameters for Urgency, Inspiration, Directness, Scenic (plus secondary: Health emphasis, Budget sensitivity). |
| **Lane** | UI mode derived from dials: **Express**, **Guided**, **Scenic**. |
| **Ruleset** | Versioned YAML rules mapping situations to dial presets, faculty activation, constraints and weights. |
| **Food graph** | Our normalized database of restaurants, menus, canonical dishes, ingredients, nutrition, semantic tags. |
| **Canonical dish** | A normalized dish concept (e.g., *pad thai*) that many menu items map to. |
| **Meal set** | A proposed combination of items (possibly from several restaurants) covering all seats. |
| **Leg** | The part of an order belonging to one merchant (restaurant or shop). |
| **Availability check** | Asking a merchant whether the leg can be prepared by a time; soft hold. |
| **Firm confirmation** | Commitment sent after payment authorization; the merchant starts cooking at the scheduled time. |
| **Restaurant liaison** | The automated agent that talks to merchants over WhatsApp, SMS, voice, POS. |
| **DaaS** | Delivery-as-a-service (DoorDash Drive, Uber Direct, via Nash). |
| **Synchronized legs** | Several DaaS deliveries timed to arrive together. |
| **Concierge run** | One courier from our own/partner pool doing multiple pickups and shop stops. |
| **Shop extras** | Items bought at shops along the route (drinks, dessert, flowers, essentials). |
| **Detour budget** | Max minutes a shop stop may add to the route. |
| **Thermal class** | hot / cold / frozen / ambient — drives pickup order. |
| **Hard constraint** | Allergy, diet rule (e.g., halal, vegan), medical restriction — never violated. |
| **Soft preference** | Learned taste with confidence and decay. |
| **Situational context** | "Tonight only" wishes, never persisted as preferences. |
| **Evidence level** | A–D grading attached to health/natural-medicine statements. |
| **MoR** | Merchant of record — NothingBetter sells the order to the customer. |
| **86'd** | Restaurant slang for an item that is sold out. |
