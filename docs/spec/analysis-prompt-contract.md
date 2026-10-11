# Analysis prompt contract

Status: resolved in the wayfinder ticket "Grilling: humour voice and analysis prompt contract". Terms are defined in `CONTEXT.md`; the decision is recorded in ADR 0005. The bake-off (separate ticket) tests this contract; it does not change it.

## Call shape

One synchronous, structured-output vision call per Sighting from the BullMQ worker, through the provider-agnostic adapter. Input: the resized photo (image first), the Note if any, Neighbourhood and street name if known, the Taxonomy with definitions, the two Tone shortlists, and the rules below. Raw lat/lng is not sent. Every Analysis row stores `model`, `prompt_version`, `taxonomy_version` and the raw output.

## Output fields (in schema order)

| Field | Type | Notes |
|---|---|---|
| `observations` | string, English, 1-2 sentences | The model describes the scene before judging it. Stored for audit, never shown publicly. |
| `category` | enum, Taxonomy | Includes `other` and `unclassifiable`. |
| `severity` | enum `nuisance` / `hazard` / `dangerous` | Honest, not comedic. |
| `ridiculousness` | enum `mild` / `notable` / `legendary` | The comedic axis, independent of Severity. |
| `caption_el`, `caption_en` | string | One factual sentence, at most ~12 words, no humour. |
| `joke_tone_el`, `joke_tone_en` | enum, Tone list | Must come from that language's shortlist. |
| `joke_el`, `joke_en` | string | One or two sentences, at most ~25 words, written independently in each language, not translated. |
| `confidence` | enum `low` / `medium` / `high` | |
| `note_conflict` | boolean | True when the Note contradicts what the photo shows. |
| `unclassifiable_reason` | nullable enum | `no_problem_visible`, `image_unusable`, `person_dominant`, `refused`, `invalid_output`, `other`. Set only when `category` is `unclassifiable`. |

Length limits live in the prompt; the provider schemas cannot enforce them. App code (Zod) re-validates every field, compares enums case-insensitively, and on a length failure retries once and then routes to `needs-review`. It never truncates mid-sentence.

## Voice

- No fixed voice. Tone varies per Sighting and per language.
- Shortlist draw: 3 Tones per language, random, language-eligible, excluding the last 5 used in that language, and always containing at least one `gentle` Tone.
- The model picks the best fit per language; the Greek and English Jokes may use different Tones and different jokes.
- Greek is modern standard (demotic) Greek: no Greeklish, no katharevousa for effect.

### Tone list v1

| Tone | Languages | Gentle |
|---|---|---|
| `deadpan-reporter` | el, en | yes |
| `mock-bureaucrat` | el, en | yes |
| `greek-tragedy` | el, en | yes |
| `noir-detective` | el, en | yes |
| `exasperated-local` | el, en | no |
| `nature-documentary` | el, en | no |
| `sports-commentator` | el, en | no |
| `travel-brochure` | el, en | no |
| `kafeneio-uncle` | el | no |
| `dry-british-understatement` | en | yes |

The gentle flag on the last two was not discussed; it is proposed here (understatement is gentle, the kafeneio uncle is not) and can be changed with a `prompt_version` bump.

## Guardrails

1. Never joke about people in the photo. If a person is the focus, the Joke targets the problem, not them.
2. Never name or mock identifiable private businesses, brands or license plates.
3. No jokes at the expense of disability, homelessness, poverty or accessibility users. Inaccessibility itself is fair game, since the target is the infrastructure.
4. The target is always the object or the system, never an individual. "The Municipality" generically is fine; named officials or parties are not.
5. No profanity.
6. For `dangerous` Severity the Joke must use a `gentle` Tone: wry, not mocking an injury risk.
7. Place references in a Joke are allowed only when factual (for example "on Ermou"); no stereotyping of a Neighbourhood.

## The Note

Context and a hint, never quoted verbatim in the Joke (it is already displayed beside it). The photo wins for Category. If the Note contradicts the photo the model sets `note_conflict: true` and the Sighting goes to `needs-review` rather than publishing a confident-looking wrong Category.

## Outcomes and Status

| Result | Status |
|---|---|
| Real Category, no `note_conflict`, valid output | `published` |
| `category: unclassifiable` | `needs-review` |
| `note_conflict: true` | `needs-review` |
| Provider refusal, safety-filter blank, invalid JSON after one retry, length-validation failure after one retry | `needs-review`, `category: unclassifiable`, reason `refused` or `invalid_output` (or `image_unusable`), raw failure stored |
| Transient error (timeout, 5xx, rate limit) | BullMQ retry with backoff; `failed` after the final retry |

Whether `confidence: low` alone holds a Sighting back is left to the owner-screen design (UI prototype ticket); it is stored either way.

## Versioning

One `prompt_version` string (for example `2026-10-a`) covers the system prompt, the Tone list and the output schema. The prompt text lives in the repo as a versioned file, not in the database. Any change to prompt text, Tone list or schema bumps it.

## Left to other tickets

- The regression photo set for prompt changes: folded into the blind bake-off ticket.
- The final model pick and the bake-off's rating protocol: bake-off.
- How Tones, `confidence` and the review queue appear in the UI: UI prototype.
