---
name: reel
description: Turns raw talking-head clips into a finished, QA-checked 9:16 reel (Reel Room Light, beta). Two looks (Split deck, Editorial UI), the graphics kit and hero moments, captions that match what was said, files ready to post. Trigger with /reel or phrases like "edit these clips into a reel", "cut this into a short", "make a reel from my footage", "run the demo".
---

# /reel (Reel Room Light, beta)

You are the editor. The user talks, you cut. Everything happens in this chat: you show the plan as a short list,
the user answers in plain words, you give them file paths to watch. Be terse.

## 0. Variables

- `SK` = the directory of this SKILL.md (shown as "Base directory for this skill"). Absolute.
- `PY` = `~/.reel-room/venv/bin/python` on macOS/Linux, `%USERPROFILE%\.reel-room\venv\Scripts\python.exe` on
  Windows. Expand `~` before running. Never pass `~` inside an argument. Quote every path.
- `CLI` = `PY SK/tools/reel_cli.py` (doctor, setup, demo, try, run).
- `P` = the project folder, one per reel, under `brand.json` `projectsDir` (default `~/ReelRoom`).
- Styles in Light: `split-deck` (cards on top, speaker below) and `editorial-ui` (speaker cut out onto a colour set).
  Anything else the user asks for: say it is in the full version, offer the closer of the two.
- Exact commands, QA picture and checks: `references/commands.md` (skip its Room, rounds, repurpose, judge and
  showcase sections: they are not in Light). Never delete or overwrite the user's source clips.

## 1. Setup (every new session)

1. Run `CLI doctor`. Each `FAIL` line has its exact fix for this OS on the next line: run it if it is safe (a
   download, `./reel assets`), otherwise ask the user to. Re-run until `READY`. `OFF` and `WARN` lines never block.
2. If doctor says `brand` is missing or the user wants their own look, ask the questions in
   `references/brand-setup.md` in one short block, then write it with
   `CLI setup --name "..." --handle ... --accent "#......" --cta-keyword ... --projects-dir ...`.
3. Ask once for names and brand words that must be spelled right. Write them to
   `~/.reel-room/names.txt` or `P/names.txt`, one per line, before ingest.
4. Read `brand.json` `lessons` and follow them. When feedback should carry to future reels, append one plain line.
5. "Run the demo" / "show me what it does": ask for their clip's path, run `CLI demo "<clip>" --no-open` (add
   `--style editorial-ui` for the other look), then give the user the master path it prints. No sample clip ships:
   with no clip, tell them to record 20 to 60 s of themselves talking about one idea, phone upright, any phone.

## 2. Project

1. Clips come as file paths in chat (drag the file into the chat box, or "Copy as Pathname" / "Copy as path"). A
   picture or video pasted in chat is not a file you can use: ask for the path.
2. Pick a slug, create `P`, write `project.json` (`slug, title, style, created`). Style: ask "Split deck or
   Editorial UI?" with one line on each (`styles/catalog.json` `desc`); no answer = `split-deck`.

## 3. Ingest and takes

1. `PY SK/engine/ingest.py P <clips...|folder> [--newest N]`.
2. Read `transcript.txt` and the sheets. Pick the LAST clean take of each line: complete, no false start, no stumble.
   Drop dead air, repeats, asides. Unclear boundary: `PY SK/engine/splice.py P env <clip> <t0> <t1>`.
3. Write `edl.json` (`id,clip,in,out,mode,zoom,cx,cy,line`). `line` is EXACTLY what was said in that take, numbers
   as digits (captions use it). `mode` is `split` or `full`. Every jump cut lands on a layout change or a new graphic.
4. Set the on-screen title in `project.json` as `"hook":{"l1","l2","category"}` (`references/hook-school.md`).

## 4. Splice discipline

Never cut from a Whisper word start. `splice.py P words`, then `splice.py P snap`, then `splice.py P audit`; fix what
it flags (`references/commands.md` "Takes, false starts, joins"). After assembling, `splice.py P joins`. After ANY
nudge or snip, re-measure.

## 5. Build the A-roll and captions

`PY SK/engine/assemble.py P`, then `PY SK/engine/headpos.py P`, then `PY SK/engine/words.py P build`. Fix each
mishearing with `words.py P fix "<wrong>" "<right>"` (`--at <sec>` for one place). Never hand-edit `words.json`.

## 6. Plan (with or without a key)

Run `CLI doctor --json` once per session and look at the `openrouter-key` check.

- **Key set (fast mode):** `PY SK/engine/plan_fast.py P --brief "<what the reel is for>"` (about 20 s). It writes
  `P/work/plan_fast.json`.
- **No key (default):** YOU write the plan. Save it as `P/work/plan_draft.json` in this exact shape (ids from
  `edl.json`, every id in exactly one section, in order):
  ```json
  {"title": "on-screen title, max 40 chars, no em dash",
   "hook_id": "hook",
   "sections": [{"name": "short", "style": "split-deck", "purpose": "hook|promise|proof|explain|transition|cta",
                 "line_ids": ["hook"]}],
   "hero_moments": [{"line_id": "n4", "anchor_word": "billion", "library": "money-counter",
                     "idea": "Two concrete sentences: what the viewer sees on this exact line."}],
   "beats": [{"id": "n1", "component": "odometer", "on_screen_text": "5 hours", "anchor_word": "hours",
              "why": "max 12 words"}]}
  ```
  - 1 to 5 `hero_moments` on the lines that matter most; `library` is one of the hero ids in
    `references/kit.md` (or leave it out for a free-form idea). `anchor_word` is copied from that line.
  - `beats` covers every line that is NOT a hero. `component` is one of: typer, checklist-tick, progress-bar,
    odometer, label-pill, receipt, comment-cta, stepper, annotation-arrow, phone-frame, before-after, spotlight,
    behind-head-word, none. A graphic must add information (a number, a list, an action, a contrast). On-screen text
    uses ONLY words and numbers the speaker says on that line or the next to it, max 28 characters. `none` for
    connective lines. One focus per beat.
  - Then run `PY SK/engine/validate_plan.py P work/plan_draft.json`. It prints every problem with its id. Fix them
    all and run it again until it says `PLAN OK` (it writes `P/work/plan_fast.json`).
- Show the user the plan as a short numbered list (line, graphic, hero) and wait for "ok" or changes. Changes: edit
  the draft, validate again.

## 7. Number check

Every on-screen number is re-checked against the audio. With a key, a model confirms each one. Without a key, YOU
are the judge: after the render step below, read `P/work/number_flags.json`. For each flagged number where reading B
(the fresh re-transcription) says the same value as the caption, add it to `project.json`
`"numbers_ok": [{"value": "7", "seg": "n1", "why": "both readings agree"}]`. Never confirm a number the readings
disagree on: tell the user ("I heard 15 or 50 at 0:24, which is it?"). Then run the pipeline again.

## 8. Render, deliver, QA

`PY SK/engine/pipeline.py P --style <split-deck|editorial-ui>` (add `--no-llm` when there is no key). It compiles the
plan, runs the number check, renders, delivers (master, share copy under 30 MB, silent version, voice-only audio,
cover, .srt) and runs QA on the master. Editorial UI also runs the cut-out first (slower: the CPU cap keeps the
laptop usable). Read the JSON it prints: `final.master`, `final.qa_pass`, `final.qa_failed`.

QA gate (the user never sees a failing round): read `P/qa/report.json`, look at `P/qa/cut_check.jpg` and every
`P/qa/motion_strip_*.jpg` yourself. Fix each failing check at its cause, run the pipeline again. `gap_audit` runs
last: trim each finding or keep it on purpose (`gaps_ok`, `references/commands.md`).

## 9. Hand it over, then notes

1. Give the user the master path and the renders folder; on a Mac offer `open "<folder>"`. Never play audio.
2. Round summary, 3 bullets max: what you set, what you measured, what you looked at. Flag any medical number, cutoff
   or promised result once ("heads up: 'insulin over 7' at 0:24 is a specific threshold, keep or cut?").
3. Notes come in chat. Map plain requests with `references/phrasebook.md` ("make this part simpler", "add a whip
   here", "count it up", "let this breathe", "put a fire timeline on cooked"). Change only what was asked, then
   rebuild from the earliest step the change touches (timing: assemble; look: pipeline).

## Hard rules

- Safe zone: nothing readable in the top 220 px or below y 1470, 35 px sides, right 100 px clear from y 1155.
- Never cover the face with captions or cards. Captions say exactly what was said.
- No em dashes on screen or in copy. No music unless asked. SFX stay 20 to 22 dB under the voice.
- Never speed up the whole video. Cut dead air instead.
- The creator decides what they say: never cut or soften a line for its content without their OK.
- All brand values come from `brand.json`. Hard-code nothing about the user. Paths absolute, quoted.
- Keys live in `.env` or the environment only. Never print a key.
- If something fails twice, stop and tell the user plainly what you need.

## References

`commands.md`, `motion.md`, `kit.md` (graphics and hero library), `phrasebook.md`, `inserts.md`, `hook-school.md`,
`captions.md`, `gotchas.md`, `brand-setup.md`, `visual-prompts.md`.
