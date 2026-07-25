# 🎒 Handoff brief — Word Quest "Progress Backpack" (paste into Sunny's game session)

You are Sunny's coding partner on word-quest. Build the two features below with her —
she is the designer, so present each as a fun choice she shapes, per the repo's
CLAUDE.md coaching style. Written in grown-up language so nothing needs second-guessing.

## Context (read first)

- The game runs on **GitHub Pages, played in Safari on her iPad** — everything must stay
  a single static `index.html`: no server, no accounts, no external services.
- The game already loads `assignments.json` (fixed tasks with ids like `v001`) and keeps
  coins/purchases in `localStorage`.
- The tasks file is regenerated periodically by the Mac side ("Dad's helper"), which
  wants to know how she got on so the next pack fits her better.

## Feature 1 — the game remembers (client-side scheduler)

1. Log every answer to `localStorage` (key e.g. `wordquest_attempts_v1`) as
   `{task_id, correct, helped, tries, at}` — `helped` = she used "Show me".
   Append-only; survives reloads (same pattern as the existing save).
2. Use the log when picking level questions:
   - a task missed last time comes back **sooner** (next level or two);
   - a task won twice cold **rests** (stop serving; it can return much later);
   - tasks won only with "Show me" come back for a **cold retry**;
   - unseen tasks fill the rest, shuffled as now.
3. Keep all of this invisible and kind — no "you got this wrong before" messaging ever.
   The game just quietly feels *right*. (Frame to her as: "the game remembers which
   words are your rivals and sends them back for a rematch!")

## Feature 2 — "Send progress to HQ" (results export)

1. A small, non-shiny button tucked away (e.g. bottom of the avatar screen):
   **"📮 Send progress to HQ"**.
2. On tap, build this exact JSON from the attempts log (see the brain-side contract —
   field names must match precisely):

```json
{ "format": "wordquest-results-v1", "exported_at": "2026-07-25T18:00",
  "batch": "<the batch field from assignments.json>",
  "attempts": [ { "task_id": "v001", "correct": true, "helped": false,
                  "tries": 1, "at": "2026-07-25T17:55" } ] }
```

3. Share it via `navigator.share({ files: [new File([json], "gameresults_<date>.json",
   {type:"application/json"})] })` — on iPad this opens the share sheet (AirDrop/Messages
   to Dad). Fallback if share isn't available: show the JSON in a textarea with a
   "Copy" button.
4. Contains ONLY task ids and results — no names, no free text. Never auto-sends;
   only on her tap.

## Definition of done

- Plays exactly as before on the Pages link from her iPad (test on `dev` first).
- Attempts log grows and persists; missed tasks visibly return sooner.
- Export produces valid JSON matching the schema; share sheet opens on iPad.
- No new files, no libraries, no accounts, no personal info.
- Merge to `main` only when she's happy and it's tested.
