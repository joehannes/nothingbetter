# knowledge/

Versioned knowledge, rulesets and templates used by the AI platform. Everything here is **data**, validated in CI against JSON Schemas in `packages/knowledge-kit/schemas/` (to be created in milestone M1).

| Folder | Contents | Spec |
|---|---|---|
| `rulesets/situations/` | Situation → dial presets, faculties, constraints, weights | [AI/10](../docs/03-ai/10-prompts-rulesets-and-templates.md) |
| `rulesets/occasions/` | Occasion playbooks (courses, extras, tone) | [AI/10](../docs/03-ai/10-prompts-rulesets-and-templates.md) |
| `rulesets/dials/` | Dial signals and deltas | [AI/04](../docs/03-ai/04-conversation-engine-and-adaptive-dials.md) |
| `cultures/` | Cultural packs (self-declared heritage only) | [AI/06](../docs/03-ai/06-knowledge-base-culture-health-seasons.md) |
| `natural-medicine/` | Ingredient/herb entries with traditional views, evidence, interactions | [AI/06](../docs/03-ai/06-knowledge-base-culture-health-seasons.md) |
| `calendar/` | Occasion date rules (computed yearly into `occasion_calendar`) | [AI/06](../docs/03-ai/06-knowledge-base-culture-health-seasons.md) |
| `templates/prompts/` | Host, faculty and ops-agent prompts (front matter + body) | [AI/10](../docs/03-ai/10-prompts-rulesets-and-templates.md) |
| `templates/merchant/` | WhatsApp/SMS templates (mirrored to provider registries) | [WF/02](../docs/02-workflows/02-availability-confirmation-whatsapp.md) |
| `templates/questions/` | Question bank for information-gain questioning | [AI/04](../docs/03-ai/04-conversation-engine-and-adaptive-dials.md) |

**Status: seed drafts.** Health, natural-medicine and cultural entries must be reviewed by the appropriate advisor before production use (`review:health`, `review:culture` PR labels).
