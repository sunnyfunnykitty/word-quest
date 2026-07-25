# Word Quest — handoff for the assignments.json (Cowork) agent

This file briefs the agent that generates `assignments.json` from Sunny's schoolwork.
It documents exactly how the game consumes that file today, so the data pipeline and the
game stay in sync. The game is a single self-contained `index.html` (no build step, no
accounts, no data collection); `assignments.json` is the ONLY file it reads.

## The story so far (2026-07-25)

Sunny designed the game into a kawaii Toca-Boca-style city ("Word Quest City"):

- A pastel town (light pink / mint / light yellow) with four challenge shops — Bakery,
  Pet Café, Flower Shop, School. Tapping a shop starts a level of **3 questions** drawn
  from `assignments.json`.
- Right answers earn coins (5 solo, 2 after the "Show me 💡" help). Finishing a level
  opens a treasure chest: bonus coins (8 + 4 × stars) plus a surprise present (any
  catalog item priced ≤ 45) that waits at the Post Office.
- Coins buy pets and furniture at the Kawaii Market; owned things live in a "drop bar"
  and can be placed in her house. Furniture is interactive (sit, lamp/evening mode,
  radio melody, bathtub, fridge snacks, TV). 17 hand-drawn full-body avatars; all art is
  inline SVG, sounds are Web-Audio synthesized — no external assets, by design.
- Progress persists in `localStorage` under the key `wordquest_city_v1` (device-only).

## How the game loads tasks

On boot it does `fetch("assignments.json", {cache:"no-store"})`. If the fetch fails or
the JSON has no non-empty `tasks` array, it silently falls back to a small embedded
sample. Expected shape:

```json
{
  "generated_at": "2026-07-25",
  "for": "Sunny",
  "tasks": [
    {
      "id": "t001",
      "type": "choose_best_word",
      "target_skill": "shades_of_meaning",
      "difficulty": 2,
      "prompt": "The castle was very old. Which word paints the strongest picture?",
      "options": ["old", "ancient", "used", "late"],
      "answer": "ancient",
      "teach": "'Ancient' means very, very old — a much stronger picture than plain 'old'."
    }
  ]
}
```

Top-level fields other than `tasks` are ignored by the game (safe to keep for
bookkeeping).

## Task contract (what the game actually does with each field)

| Field          | Required | Behaviour in the game                                                        |
|----------------|----------|------------------------------------------------------------------------------|
| `id`           | yes      | Not displayed; keep unique for traceability.                                  |
| `type`         | yes      | Stable contract — new types may be ADDED, existing ones never renamed.        |
| `target_skill` | no       | Shown as a small chip, `_` replaced by spaces.                                |
| `difficulty`   | no       | Currently **unused** by the game (fine to keep sending).                      |
| `prompt`       | yes      | Shown as the question text.                                                   |
| `options`      | see below| If present and non-empty → rendered as tap buttons.                           |
| `answer`       | yes*     | Grading (see below). *Tasks without an `answer` are excluded from levels.     |
| `teach`        | no       | Shown after answering (and after "Show me"). Keep it kind and one sentence.   |

### Rendering / grading rules — IMPORTANT for generation

1. **Any task with a non-empty `options` array** becomes multiple-choice buttons.
   Grading is **exact string equality** between the tapped option and `answer` — the
   `answer` string must appear in `options` character-for-character (case, spacing,
   punctuation, apostrophe style ' vs ').
2. **Any task without `options`** becomes a typed-answer box. Grading normalizes both
   sides: lowercase, strip `. , ! ? ; : ' '`, collapse whitespace. So capitalisation and
   end punctuation are forgiven, but WORDS must match exactly — keep expected answers
   short and unambiguous (one clearly correct sentence/word).
3. `type: "design_me"` tasks are **filtered out** of gameplay (they were design
   placeholders). Don't send them in real assignments.
4. Unknown new `type` values don't crash the game: with `options` they render as
   multiple-choice, without as typed input. Still, coordinate before inventing types so
   the game can add bespoke fun for them (e.g. `match_pairs` is a wished-for future type
   and has NO special UI yet).

### Level mechanics that affect content design

- A level pulls **3 tasks** (fewer if the pool is smaller). The whole task list is
  shuffled and consumed without repeats until exhausted, then reshuffled — so ~9-15
  varied tasks make a good daily file.
- A wrong tap/typo lets her retry; after a wrong attempt a gentle "Show me 💡" button
  reveals answer + teach for reduced coins. Nothing is ever punishing.
- Every 3-task level always ends in a treasure chest, so even short files feel rewarding.

## Repo / publish model (unchanged)

- Sunny designs from the iPad on her branch (currently `claude/word-quest-design-c3ey7d`);
  **`main` is protected** and only Dad reviews/merges. The public play link
  (https://sunnyfunnykitty.github.io/word-quest/) serves **`main`** — merging is the
  publish step, done by Dad from the Mac.
- Code lives in git; personal data (schoolwork photos, progress) stays local on the Mac.
  `.gitignore` blocks `data/`, `inbox/`, `corpus/`, `*.sqlite`, photos — keep it that way.
  `assignments.json` in the repo should only ever contain content that is fine being
  public (it currently holds neutral sample tasks; real generated ones should stay
  local/private per the family rule).

## Code map (for quick orientation)

Everything is in `index.html`:

- `EMBEDDED` — fallback tasks · `boot()` — loads assignments.json
- `SHOPS`/`SPECIALS` + `shopfrontArt()` — the town · `startLevel()`/`showTask()` — quiz flow
- `CATALOG` + `ART` — buyable items and their SVG drawings · `KIDS` + `kidArt()` — avatars
- `SFX`/`tone()` — synthesized sounds · `state` + `save()`/`load()` — localStorage save
- House: `renderRoom()`, `interact()` (tap-to-play), long-press = pick up

Keep talking to Sunny the way CLAUDE.md describes: warm, simple, emoji-friendly — she is
the designer and the boss. 🎀
