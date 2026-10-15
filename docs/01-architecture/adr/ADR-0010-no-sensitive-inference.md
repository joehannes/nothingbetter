# ADR-0010 · Sensitive attributes are self-declared, never inferred

- Status: Accepted (2026-10-15)

## Context
Cultural background, religion and health strongly shape good food advice, but inferring them from names, voices, faces, neighborhoods or orders is error-prone, stereotyping, and regulated as sensitive data.

## Decision
- Culture/heritage, observances, health conditions are only set by the user (or caregiver for a child) with explicit consent.
- Regional/cultural knowledge may shape **what is available and popular nearby** (supply-side) but never assumptions about an individual.
- Learned taste signals (e.g., "orders Korean often") stay as taste preferences, never converted to identity labels.
- The assistant may *ask* gently ("Any cuisine that feels like home?") and accept "prefer not to say".

## Consequences
- + Trust, legal safety, less stereotyping.
- − Less personalization for users who skip these questions; mitigated by taste learning.
