# 04 · Data Model

> Status: Draft v1 · PostgreSQL 17 + PostGIS + pgvector · One schema per module · Drizzle is the source of truth once scaffolded; this doc is the design.

Conventions: `id text primary key` (prefixed ULID), `created_at/updated_at timestamptz`, soft delete via `deleted_at` where users can restore, money as `amount_minor bigint` + `currency char(3)`. Sensitive columns marked 🔒 are encrypted at the application layer (envelope encryption, per-tenant data key in KMS).

## 1. Entity map

```text
identity ── users ─┬─ devices, auth_identities, consents
                   └─ households ── household_members ── avatars
people   ── avatars ─┬─ avatar_constraints (hard)      ─ avatar_health 🔒 (opt-in)
                     ├─ avatar_culture (self-declared) ─ avatar_preferences (soft, learned)
                     └─ memories (episodic, embeddings)  ─ important_dates
conversation ── sessions ── messages, situation_briefs, dial_states, council_runs, proposals ── proposal_sets ── proposal_items
catalog ── merchants (restaurant|shop) ── merchant_locations (PostGIS) ── merchant_channels
        └─ menus ── menu_versions ── menu_sections ── menu_items ── item_modifiers
        └─ canonical_dishes ── dish_enrichment, dish_ingredients ── ingredients
knowledge ── kn_entities ── kn_relations ── kn_sources ; occasion_calendar ; season_produce
order ── baskets ── basket_items (seat=avatar) ; orders ── order_legs ── leg_items
      └─ availability_checks ── channel_messages
payments ── payment_intents ── ledger_entries ; issuing_cards ; transfers ; refunds
dispatch ── dispatches ── dispatch_stops ── tracking_points ; provider_quotes
feedback ── dish_feedback (per seat) ; session_feedback
ai ── ai_runs (telemetry) ; prompt_versions ; ruleset_versions ; eval_results
ops ── audit_log ; outbox_events ; incidents
```

## 2. People & memory

```sql
create table people.households (
  id text primary key,                -- hh_
  owner_user_id text not null references identity.users(id),
  name text not null,                 -- "Home", "Office team"
  default_place_id text,
  created_at timestamptz not null default now()
);

create table people.avatars (
  id text primary key,                -- av_
  household_id text references people.households(id),
  owner_user_id text not null,        -- who controls this profile
  linked_user_id text,                -- set when the person claims their own avatar
  display_name text not null,         -- "Mia", "Mom"
  relationship text not null,         -- self|partner|child|parent|grandparent|friend|coworker|other
  age_band text,                      -- infant|child|teen|adult|older_adult (no DOB needed)
  is_minor boolean not null default false,
  languages text[] default '{}',
  spice_tolerance smallint,           -- 0..5
  adventurousness smallint,           -- 0..100 (learned + stated)
  portion_size text,                  -- small|regular|large
  avatar_style jsonb,                 -- illustration config (no photos of minors)
  created_at timestamptz not null default now(),
  deleted_at timestamptz
);

-- Hard constraints: never learned, only user-confirmed.
create table people.avatar_constraints (
  id text primary key,                -- con_
  avatar_id text not null references people.avatars(id),
  kind text not null,                 -- allergy|intolerance|diet|religious|medical|texture|never
  code text not null,                 -- e.g. allergen:sesame, diet:vegan, religious:halal, medical:low_sodium
  severity text,                      -- for allergies: anaphylaxis|severe|mild
  notes text,
  source text not null,               -- user_stated|imported|caregiver
  confirmed_at timestamptz not null,
  unique (avatar_id, code)
);

-- Sensitive, opt-in health context (consumer health data laws apply).
create table people.avatar_health (
  avatar_id text primary key references people.avatars(id),
  conditions_enc bytea,               -- 🔒 e.g. hypertension, diabetes_t2, pregnancy, ibs
  goals_enc bytea,                    -- 🔒 e.g. muscle_gain, weight_mgmt, recovery
  medications_enc bytea,              -- 🔒 optional, only for interaction checks
  natural_medicine_lens boolean not null default false,
  consent_id text not null references identity.consents(id),
  updated_at timestamptz not null default now()
);

-- Self-declared culture; never inferred.
create table people.avatar_culture (
  avatar_id text not null references people.avatars(id),
  heritage_code text not null,        -- kn entity: culture:cuban, culture:punjabi …
  relation text not null,             -- grew_up_with|family_heritage|lives_by|curious_about
  observances text[] default '{}',    -- ramadan, lent, shabbat, navratri, kosher_style …
  primary key (avatar_id, heritage_code)
);

-- Soft preferences learned with confidence and decay.
create table people.avatar_preferences (
  id text primary key,                -- pref_
  avatar_id text not null references people.avatars(id),
  dimension text not null,            -- cuisine|dish|ingredient|flavor|texture|restaurant|price|temperature
  key text not null,                  -- e.g. cuisine:thai, flavor:umami, dish:pad_see_ew
  polarity smallint not null,         -- -2 (avoid) .. +2 (love)
  confidence real not null,           -- 0..1
  evidence_count int not null default 1,
  context_scope text,                 -- null=always | lunch|weeknight|celebration|sick …
  source text not null,               -- stated|order|rating|conversation|inferred
  last_evidence_at timestamptz not null,
  half_life_days int not null default 180,
  unique (avatar_id, dimension, key, coalesce(context_scope,''))
);

create table people.memories (
  id text primary key,                -- mem_
  avatar_id text,                     -- null = household-level memory
  household_id text,
  kind text not null,                 -- episode|ritual|story|feedback|date
  summary text not null,              -- "Loved the Peruvian tart on their 5th anniversary"
  details jsonb,
  embedding vector(1024),
  provenance jsonb not null,          -- {session_id, message_id, order_id}
  sensitivity text not null default 'normal', -- normal|sensitive
  visible boolean not null default true,
  created_at timestamptz not null default now(),
  expires_at timestamptz
);
create index on people.memories using hnsw (embedding vector_cosine_ops);
```

## 3. Conversation

```sql
create table conversation.sessions (
  id text primary key,                -- ses_
  user_id text not null,
  household_id text,
  channel text not null,              -- app|web|voice|whatsapp
  locale text not null,               -- en-US, es-US, fr-CA
  place_id text,                      -- delivery target
  status text not null,               -- active|ordered|abandoned
  started_at timestamptz not null default now(),
  ended_at timestamptz
);

create table conversation.messages (
  id text primary key,                -- msg_
  session_id text not null references conversation.sessions(id),
  role text not null,                 -- user|assistant|system_event
  modality text not null,             -- text|voice|ui_action
  content jsonb not null,             -- text, transcript, chip selection, card action
  ui_directives jsonb,                -- chips/cards rendered with this message
  created_at timestamptz not null default now()
);

create table conversation.situation_briefs (
  session_id text primary key,
  brief jsonb not null,               -- see contracts SituationBrief
  version int not null,
  updated_at timestamptz not null
);

create table conversation.dial_states (
  session_id text not null,
  at timestamptz not null,
  urgency smallint, inspiration smallint, directness smallint, scenic smallint,
  health_emphasis smallint, budget_sensitivity smallint,
  lane text not null,                 -- express|guided|scenic
  reasons jsonb not null,             -- signals that moved the dials
  user_override boolean not null default false,
  primary key (session_id, at)
);

create table conversation.council_runs (
  id text primary key,
  session_id text not null,
  faculties text[] not null,
  advice jsonb not null,              -- AdviceCard[]
  synthesis jsonb not null,           -- merged constraints/weights
  ai_run_ids text[] not null,
  created_at timestamptz not null default now()
);

create table conversation.proposals (
  id text primary key,                -- prp_
  session_id text not null,
  council_run_id text,
  sets jsonb not null,                -- MealSet[] (items per seat, merchants, totals, ETA, rationale)
  chosen_set_index int,
  created_at timestamptz not null default now()
);
```

## 4. Catalog

```sql
create table catalog.merchants (
  id text primary key,                -- rst_ (restaurant) / shp_ (shop)
  kind text not null,                 -- restaurant|shop|florist|pharmacy|grocery|bakery
  name text not null,
  legal_name text,
  status text not null,               -- prospect|onboarding|active|paused|offboarded
  partner_tier text not null,         -- partner|listed (listed = no settlement agreement)
  cuisines text[] default '{}',       -- kn codes
  diet_certifications text[] default '{}', -- halal_certified, kosher_certified, gluten_free_kitchen …
  default_prep_minutes int,
  payment_methods text[] not null,    -- connect_settlement|virtual_card_phone|virtual_card_online|courier_card
  languages text[] default '{en}',
  created_at timestamptz not null default now()
);

create table catalog.merchant_locations (
  id text primary key,
  merchant_id text not null references catalog.merchants(id),
  address jsonb not null,
  geom geography(Point, 4326) not null,
  timezone text not null,             -- IANA
  hours jsonb not null,               -- weekly schedule + exceptions
  pickup_instructions text,
  service_area_ids text[] not null
);
create index on catalog.merchant_locations using gist (geom);

create table catalog.merchant_channels (
  id text primary key,
  merchant_id text not null,
  channel text not null,              -- whatsapp|sms|voice|pos_square|pos_toast|pos_clover|partner_app|email
  address text not null,              -- E.164 phone, POS location id, email
  priority smallint not null,         -- escalation order
  opt_in_at timestamptz,              -- required for whatsapp/sms/voice automation
  opt_in_evidence jsonb,
  quiet_hours jsonb,
  active boolean not null default true
);

create table catalog.menu_items (
  id text primary key,                -- itm_
  menu_version_id text not null,
  section text,
  name text not null,
  description text,
  price_minor bigint not null, currency char(3) not null,
  canonical_dish_id text,             -- mapped dish concept
  labels text[] default '{}',         -- restaurant-declared: vegan, gf, spicy, halal …
  allergens_declared text[] default '{}', -- restaurant-verified only
  allergens_estimated jsonb,          -- {sesame: 0.6, …} estimated, never shown as "free of"
  nutrition_estimate jsonb,           -- kcal, protein_g, fat_g, sat_fat_g, carbs_g, fiber_g, sodium_mg + confidence
  thermal_class text,                 -- hot|cold|frozen|ambient
  travel_score smallint,              -- 0..100 how well it survives delivery
  available boolean not null default true,
  embedding vector(1024)
);

create table catalog.canonical_dishes (
  id text primary key,                -- dish_
  name text not null,
  cuisine_codes text[] not null,
  description text,
  typical_ingredients text[],
  flavor_profile jsonb,               -- sweet, sour, salty, bitter, umami, spicy, rich, fresh (0..1)
  texture_profile jsonb,
  tcm_nature text,                    -- warming|cooling|neutral (traditional lens)
  ayurveda jsonb,                     -- rasa, virya, dosha effects (traditional lens)
  occasion_tags text[],
  comfort_tags text[],
  embedding vector(1024)
);
```

## 5. Orders, payments, dispatch

```sql
create table "order".orders (
  id text primary key,                -- ord_
  user_id text not null, session_id text, household_id text,
  place_id text not null,
  status text not null,               -- see WF/01 state machine
  scheduled_for timestamptz,          -- null = ASAP
  dispatch_mode text not null,        -- single|synchronized_legs|concierge_run
  subtotal_minor bigint not null, fees_minor bigint not null, tax_minor bigint not null,
  tip_minor bigint not null default 0, extras_cap_minor bigint not null default 0,
  total_authorized_minor bigint, total_captured_minor bigint,
  currency char(3) not null,
  promise_window tstzrange,           -- promised arrival window
  created_at timestamptz not null default now()
);

create table "order".order_legs (
  id text primary key,                -- leg_
  order_id text not null references "order".orders(id),
  merchant_id text not null,
  kind text not null,                 -- restaurant|shop_extra
  status text not null,               -- pending_check|available|modified|unavailable|confirmed|paid|preparing|ready|picked_up|cancelled
  ready_by timestamptz,
  merchant_reference text,            -- their order/ticket number
  payment_method text not null,       -- connect_settlement|virtual_card|courier_card
  amount_minor bigint not null, currency char(3) not null,
  thermal_class text
);

create table "order".availability_checks (
  id text primary key,                -- avc_
  leg_id text not null,
  requested_ready_by timestamptz not null,
  status text not null,               -- sent|answered|timeout|escalated|cancelled
  result jsonb,                       -- per-item availability, substitutions, ready_at, price changes
  hold_expires_at timestamptz,
  channel_attempts jsonb not null     -- [{channel, sent_at, message_id, status}]
);

create table liaison.channel_messages (
  id text primary key,
  merchant_id text not null, leg_id text,
  channel text not null, direction text not null,   -- outbound|inbound
  provider_message_id text unique,
  template_name text, template_lang text,
  body jsonb not null, status text,                  -- queued|sent|delivered|read|failed
  parsed jsonb,                                      -- AI/deterministic parse result
  created_at timestamptz not null default now()
);

create table payments.ledger_entries (                -- double-entry
  id text primary key, txn_id text not null,
  account text not null,              -- customer_receivable|merchant_payable:rst_x|courier_payable|delivery_cost|revenue_service_fee|tax_payable|stripe_fees|refunds
  direction text not null,            -- debit|credit
  amount_minor bigint not null, currency char(3) not null,
  order_id text, leg_id text, created_at timestamptz not null default now()
);

create table payments.issuing_cards (
  id text primary key, stripe_card_id text unique not null,
  purpose text not null,              -- merchant_single_use|courier_run
  order_id text, leg_id text, dispatch_id text,
  spending_limit_minor bigint not null, allowed_categories text[] not null,
  status text not null, expires_at timestamptz not null
);

create table dispatch.dispatches (
  id text primary key,                -- dsp_
  order_id text not null, mode text not null,
  provider text not null,             -- nash|doordash_drive|uber_direct|own_fleet
  provider_ref text, status text not null,
  quote_minor bigint, actual_minor bigint, currency char(3),
  tracking_url text
);

create table dispatch.dispatch_stops (
  id text primary key, dispatch_id text not null,
  seq smallint not null, kind text not null,   -- pickup|shop|dropoff
  leg_id text, eta timestamptz, arrived_at timestamptz, completed_at timestamptz,
  geom geography(Point,4326) not null
);
```

## 6. Knowledge

```sql
create table knowledge.kn_entities (
  code text primary key,              -- culture:cuban, dish:ropa_vieja, ingredient:ginger, occasion:iftar, concept:tcm_warming
  type text not null,                 -- culture|cuisine|dish|ingredient|nutrient|occasion|season|religion_rule|tcm_concept|ayurveda_concept|condition|etiquette|region
  labels jsonb not null,              -- {en:"…", es:"…", fr:"…"}
  attributes jsonb not null,
  locale_scope text[],                -- US-FL, CA-ON, …
  evidence_level char(1),             -- A|B|C|D for health claims
  review jsonb not null,              -- {reviewer_role, reviewed_at, next_review}
  version int not null
);

create table knowledge.kn_relations (
  subject text not null, predicate text not null, object text not null,
  weight real, qualifiers jsonb,      -- e.g. {season:"winter", evidence:"B"}
  source_id text, primary key (subject, predicate, object)
);

create table knowledge.kn_sources (
  id text primary key, title text, url text, license text, accessed_at date
);

create table knowledge.occasion_calendar (
  occasion_code text not null, region text not null,
  starts_at timestamptz not null, ends_at timestamptz not null,
  daily_timing jsonb,                 -- e.g. iftar = local sunset; computed per place
  primary key (occasion_code, region, starts_at)
);
```

## 7. AI telemetry

```sql
create table ai.ai_runs (
  id text primary key, session_id text, purpose text not null,  -- orchestrator|faculty:nutrition|menu_extract|reply_parse …
  model text not null, effort text, prompt_version text not null, ruleset_version text,
  input_tokens int, output_tokens int, cache_read_tokens int, cache_write_tokens int,
  latency_ms int, stop_reason text, refusal_category text, fallback_model text,
  cost_micro_usd bigint, created_at timestamptz not null default now()
);
```

## 8. Retention

| Data | Default retention |
|---|---|
| Raw voice audio | Not stored (streamed); opt-in debug clips ≤ 7 days |
| Transcripts & messages | 18 months, user-deletable |
| Memories | Until deleted; episodic memories auto-summarized after 12 months |
| Health data | Until deleted or consent withdrawn (immediate purge) |
| Courier GPS traces | 30 days, then aggregated |
| Ledger & orders | 7 years (tax/accounting) |
| Merchant messages | 24 months |
