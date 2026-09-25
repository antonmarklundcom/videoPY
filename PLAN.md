# Content system: master plan (draft v1, 2026-09-25)

Status (2026-09-25): **§1 confirmed with changes.**
- **Building now:** the research studio in `content-engine` (competitor research,
  YT digests + saved lessons, ideas, titles, on-camera scripts with a
  teleprompter, Higgsfield shot lists). See `content-engine/PLAN.md` §1.27–36
  and phases O5–O8, S5, S6, S8, S10–S12, S9.
- **Deferred:** everything in this repo (voice, captions, render, clipping).
  Anton records on camera for residency and real estate for now; the
  Paraguayan voice is chosen the week of 2026-10-05, then track V starts.
- **Separate build later:** children's stories for the Paraguayan market (no
  competitor research needed, so not part of content-engine).

This file covers the whole content system across four repos. Each repo keeps its
own detailed `PLAN.md`; this one decides what goes where and in what order.

---

## §0. The goal in one paragraph

Research a topic, learn from other people's YouTube videos without watching them,
write a script in the right language for the audience (English for expats,
castellano paraguayo or Jopará for Paraguayans), and turn that script into a
finished or nearly finished video with a Paraguayan voice, captions and b-roll.
Also cut your own recordings into shorts. Everything runs **on your own PC**,
paid only through API credits you already have, with a human approval step
before anything is published.

---

## §1. Decisions (proposed, reply "yes" or correct by number)

| # | Decision | Recommendation | Why |
|---|---|---|---|
| 1 | Where it runs | **Local PC first.** No Node.js hosting on Hostinger for this. | You are the only user. Local means no deploys, API keys stay on your PC, and YouTube captions are fetched from your home IP (YouTube blocks datacenter IPs, which was the biggest open risk in `yt` and `content-engine`). |
| 2 | Repo roles | **content-engine** = research, learn from YT, ideas, scripts (Node, browser UI on `localhost`). **videoPY** = voice, captions, video render, clipping (Python CLI). **aiinsights** = AI lessons from IG (stays separate, online). **contenido** = website only. **yt** = archive (content-engine already contains it). | One place to think, one place to produce. The best open-source audio/video tools (Whisper, auto-editor, Chatterbox, MoviePy) are Python; content-engine is already Node. |
| 3 | The link between them | A single **`script.json`** file per video (§3). content-engine writes it, videoPY reads it. | Two programs that only share a file can't break each other, and you can hand-edit the script between steps. |
| 4 | Database | content-engine runs locally with **`DB_DRIVER=pg` against a Neon free database** (or local Postgres). | Solves the open `db.transaction()` bug in `content-engine/docs/decisions-needed.md` without code changes: the `pg` driver supports transactions. Neon free keeps the data safe if the PC dies. |
| 5 | AI provider for research/scripts/digests | **Gemini** (your existing credits; already wired in content-engine, with Google Search grounding). Claude/Codex via your subscription for hand-written, interactive work. | Zero new cost. |
| 6 | Language per audience | Residency and real estate → **English first** (plus Spanish later; the buyers are mostly foreigners). Local business marketing → **castellano paraguayo** with light Jopará. Children's stories → **castellano paraguayo + Guaraní words** (Jopará). Pure Guaraní → later, only if demand shows. | Matches who actually watches each topic. |
| 7 | Voice | Test 3 voices in phase V2, pick by ear (§5). No model training. | Cloning from a clean 1–3 min sample is enough; training costs time and money for little gain. |
| 8 | Automation level | "Script → rendered draft video" is automatic; **you approve before upload**. No auto-posting. | YouTube demonetises "inauthentic / mass-produced" content; a human check and a real intro (idea 5 in §9) protect the channel. |
| 9 | Kids channel | Mark every video "Made for Kids". Build it **after** the residency pipeline works. | Legal requirement (COPPA); same pipeline, different template. |
| 10 | Build models | Opus for foundation phases, Sonnet for the rest. Never Fable for build work. | Standing cost rule. |

---

## §2. What each piece does

```
 YouTube videos ─┐                                   ┌─> Higgsfield (b-roll, images)
 IG / web links ─┤                                   │
                 v                                   v
         content-engine (localhost)          videoPY (Python CLI)
         ─────────────────────────           ────────────────────────────
         • YT digests: key points,           • voice: script -> narration.mp3
           lessons, saved notes                (Paraguayan / English voices)
         • research with sources             • pronunciation dictionary
         • ideas + social captions           • captions: word timestamps
         • long-form YT scripts   ─script.json─> • render: explainer 16:9,
         • fact sheet per topic                 short 9:16, kids story
                                             • clip: your own recordings ->
                                               silence cut, highlights, shorts
                                                     │
                                                     v
                                        draft.mp4 + captions.srt + thumbnail ideas
                                                     │
                                              you review -> upload

 aiinsights (online, Telegram bot): AI tools/lessons you see on IG -> summary +
 "how to start" steps -> weekly "implement one thing" reminder.
```

### What already exists (as of 2026-09-25)

| Repo | Built | Missing |
|---|---|---|
| content-engine | Brands (Residency Guide, Propia, Contenido + 8 more), idea + caption generation with Gemini and Search grounding, YT ingest (video, playlist, channel), captions, one AI analysis per video stored forever, topics, marks, search, listen mode, clip inbox (share from phone), spend cap. 191 unit tests + 34 DB tests. | **Never run against a real database or real Gemini key.** No long-form script writer. No fallback for YT videos without captions. No Paraguayan style guide. Phases O5, O6, S5–S9 not started. |
| videoPY | Nothing (empty repo). | Everything in §6 track V. |
| aiinsights | Telegram capture, Claude summary, dashboard, "implemented" flag. | Deployed? (check). The weekly nudge (idea 10 in §9). |
| contenido | Agency landing page. `PLAN-app-contenido.md` (the voice/clip idea this plan absorbs). | Real media. |
| yt | Superseded by content-engine. | Archive it. |

---

## §3. The `script.json` contract (written in V1, never changed silently)

```jsonc
{
  "id": "residency-2026-10-cedula-steps",
  "brand": "residency-guide",
  "format": "explainer_16x9",          // explainer_16x9 | short_9x16 | kids_story | listing_short
  "language": "en",                     // en | es-PY | jopara
  "voice": "narrator_en_1",             // key in videoPY voices.yaml
  "title_options": ["...", "...", "..."],
  "hook": "First 10 seconds, spoken.",
  "sections": [
    {
      "narration": "Spoken text for this section.",
      "on_screen_text": "Short overlay text",
      "visual": { "kind": "broll", "file": "broll/asuncion-skyline.mp4" },
      // kind: broll | image | own_footage | higgsfield_prompt (still to generate)
      "higgsfield_prompt": "optional prompt if no file yet"
    }
  ],
  "cta": "Spoken call to action.",
  "sources": [{ "claim": "...", "url": "...", "checked": "2026-09-25" }],
  "pronounce": { "Ypacaraí": "Ipacaraí" }   // per-video overrides
}
```

A video project is a folder: `projects/<id>/script.json`, `broll/`, `out/`.

---

## §4. Learning from YouTube (already mostly built; small additions)

Goal: read, not watch. Listen only when something is unclear.

1. **Captions first (free).** content-engine already pulls a video's existing
   captions and runs one Gemini analysis: summary, key points, and topics,
   stored forever and searchable. Running locally fixes the "YouTube blocks
   server IPs" risk.
2. **New: fallback when a video has no captions.** Send the YouTube URL straight
   to Gemini (it accepts public YouTube links) with low media resolution,
   audio-focused prompt. Capped per video by the existing spend cap, and only
   on click, never automatic.
3. **New: "Save lesson".** One click turns a key point into a saved lesson
   (text, video link, timestamp, brand/topic tag). Export all lessons for a
   topic as Markdown, so they can feed a script.
4. **Listen mode** already exists for the "listen instead of read" case.
5. **Not now:** deep learning features, flashcards, spaced repetition.

Cost: a 20-minute video's captions are about 4–5k tokens. On Gemini Flash that
is a fraction of a cent per digest; a few videos a day stays around $1–3/month,
covered by your credits. The no-captions fallback costs more (it processes audio),
which is why it is click-only.

---

## §5. Voice options that fit Paraguay

| Option | Fit for Paraguay | Cost | Commercial use | Notes |
|---|---|---|---|---|
| **Clone a real Paraguayan narrator** (hire a *locutor/locutora*, 2–3 min clean recording, written consent) with ElevenLabs Instant Voice Clone | Best: the accent comes from the recording | ~$5–22/month plan + one-time narrator fee | Yes (paid plan) | The quality benchmark. Consent contract: AI use, commercial, platforms, duration, right to revoke. |
| **Azure AI Speech, `es-PY` stock voices** | Real Paraguayan-locale voices (a male and a female) | Free tier ~0.5M characters/month, then pay per use | Yes | No cloning without Microsoft approval. Good, cheap default for castellano paraguayo. Test how natural they sound. |
| **Gemini TTS** | Accent is steered by prompt ("habla con acento paraguayo"), so results vary | Your existing Gemini credits | Yes | Worth testing because it costs nothing extra. |
| **Higgsfield voice** (create/clone) | Depends on the sample | Included in your subscription | Per Higgsfield terms | Already connected. |
| **Chatterbox Multilingual** (open source, MIT) | Clones from ~10 s sample; Spanish supported | Free locally (NVIDIA GPU recommended) or pennies on Replicate | Yes (MIT) | The open-source choice. CPU-only is slow. |
| English (residency/real estate) | Your own voice cloned, or any ElevenLabs/Azure English voice | as above | Yes | An expat voice that lived the process builds trust. |

**Avoid for commercial videos** (non-commercial licences on the model weights):
F5-TTS (note: `contenido/PLAN-app-contenido.md` calls it "commercially clean" — it
isn't), XTTS-v2, Fish Speech/OpenAudio, Meta MMS-TTS (has Guaraní, but CC-BY-NC).

**Making it sound Paraguayan is half voice, half text:**
- A style guide per language mode (castellano paraguayo, Jopará, English) that
  the script writer follows: voseo, local words (*che*, *nde*, *luego*, *un poco*
  softeners), when to drop a Guaraní word and when not to.
- A pronunciation dictionary (`pronounce.yaml`) applied before TTS:
  place names and Guaraní words respelled so the voice says them right.
  Starts small, grows every time you hear a mistake.

**Voice gate (phase V2):** the same 45-second script rendered with 3–4 options,
played side by side. You pick. Nothing downstream is built until one passes.

---

## §6. Build phases

Phases are one session each (≤ 90 min), one PR each. Human gates are marked ✋.

### Track C — content-engine (Node, local)

| Phase | Model | What | Depends on |
|---|---|---|---|
| ✋ C0 | you | Install Node LTS, Git; create Neon free DB; put `DATABASE_URL`, `DB_DRIVER=pg`, `GEMINI_API_KEY` in `.env`; `npm install && npm run db:migrate && npm run db:seed && npm run dev`. ~20 min. | — |
| O5 | Opus | Existing: Gemini test double + live smoke (unchanged). | C0 |
| O6′ | Opus | **Changed:** local-first hardening. Keep lease, owner gate, clip reaper, notes fix, `posted` status, dict split. Replace Vercel cron with a local `npm run poll` + a Windows Task Scheduler recipe. Drop S7 (caption probe route): local runs fetch from your home IP. | O5 |
| S5, S6, S8 | Sonnet | Existing: design system, ideas workflow, docs (parallel). | O6′ |
| O7 | Opus | **New: Script studio foundation.** `scripts` table; generate a long-form YT script from a brief + saved lessons + fact sheet, with Search grounding and a source list; write `projects/<id>/script.json` per §3. Language style guides (en, es-PY, Jopará) as editable files. | O6′ |
| S10 | Sonnet | **New: Learn-from-YT upgrades.** "Save lesson" button, lessons list per topic, Markdown export, click-only Gemini fallback for videos with no captions. | O6′ |
| S11 | Sonnet | **New: Script studio UI.** Brief form, section editor, title options, "export to videoPY" button. | O7 |
| S9′ | Sonnet | Link pass (existing, now after S10/S11 too). | all above |

### Track V — videoPY (Python, local)

| Phase | Model | What | Depends on |
|---|---|---|---|
| ✋ V0 | you | Install Python 3.12 and ffmpeg (one `winget` line each). Optional: NVIDIA GPU for local Whisper/Chatterbox. | — |
| V1 | Opus | **Foundation.** CLI (`videopy voice|captions|render|clip <project>`), project folder layout, `script.json` loader + validator (§3), `voices.yaml`, `pronounce.yaml`, provider interfaces, sample project, tests. | V0 |
| V2 | Sonnet | **Voice adapters + voice gate.** Gemini TTS, Azure es-PY, ElevenLabs, Chatterbox (local or Replicate), Higgsfield. `videopy voice-test` renders one script with every configured voice into a comparison folder. | V1 |
| ✋ V2-gate | you | Listen, pick the voices per language. Record/hire a narrator if none passes. | V2 |
| V3 | Opus | **Captions + render.** faster-whisper word timestamps; templates: explainer 16:9 (b-roll + overlay text + burned or soft captions + music bed with ducking), short 9:16 (word-pop captions). Output `out/draft.mp4` + `captions.srt`. | V2-gate |
| V4 | Sonnet | **Clip your own videos.** Silence/filler cut (auto-editor), transcript → Gemini picks 3–8 highlight segments → ffmpeg cut → 9:16 reframe → captions. Optional: re-voice with the chosen voice. | V3 |
| V5 | Sonnet | **Kids story + listing templates.** Story: illustrations + slow pan/zoom + narration + page-turn. Listing: property photos + price/features overlays + voice (from a Propia listing export). | V3 |

**Order:** C0 + V0 (you, same afternoon) → O5 → O6′ → {S5, S6, S8, O7, S10} and V1 in
parallel → V2 → ✋ gate → S11, V3 → V4, V5 → S9′.

### Open-source video tools (used inside videoPY, not separate apps)

| Job | Tool | Licence |
|---|---|---|
| Cut, join, overlays, audio mix | ffmpeg + MoviePy | LGPL/GPL, MIT |
| Word timestamps / captions | faster-whisper (WhisperX if needed) | MIT / BSD |
| Remove silences, jump cuts | auto-editor | Unlicense |
| Pick highlights from a long video | transcript + Gemini (the idea behind FunClip/ClipsAI) | own code |
| Fancy animated templates (later, optional) | Remotion (free up to 3 people) or HeyGen's open-source HyperFrames | see each |
| Manual editing when you want it | DaVinci Resolve (free) or CapCut; videoPY exports `captions.srt` and the narration track | — |

Not building: a full timeline editor. Hard, and free ones already exist.

---

## §7. Cost

### Minimum to start: **$0/month extra**

| Item | Minimum | Realistic |
|---|---|---|
| Hosting | $0 (local) | $0 |
| Database | $0 (Neon free) | $0 |
| Research, digests, scripts | Gemini credits you have | ~$2–10/month after credits |
| Your hands-on AI work | Claude / Codex subscription you have | same |
| Voice | $0 (Azure free tier, Gemini TTS, Chatterbox) | $5–22/month ElevenLabs + one-time narrator fee |
| B-roll / images | Higgsfield subscription you have | same |
| Render, captions, clipping | $0 (open source on your PC) | $0 |
| YouTube Data API | $0 (free daily quota) | $0 |
| aiinsights online | $0 (Vercel hobby + Neon free) | $0 |

### Build effort

About 11 build phases. At the method's usual $8–15 per phase that is roughly
$90–150 of Opus/Sonnet usage (or the equivalent in subscription limits), plus
about 1–2 hours of your time for C0, V0 and the voice gate.

---

## §8. Human-inputs checklist

| # | Input | Needed by |
|---|---|---|
| 1 | Answers to §1 | before any prompt file is written |
| 2 | Neon free project + `DATABASE_URL` | C0 |
| 3 | `GEMINI_API_KEY` from the project that holds your credits (billing enabled for Search grounding) | C0 |
| 4 | YouTube Data API key (free, Google Cloud console) | C0 |
| 5 | Azure Speech key (free tier) and/or ElevenLabs key | V2 |
| 6 | 2–3 min clean voice recording(s) + signed consent | V2 |
| 7 | 5–10 Higgsfield b-roll clips + any own footage for the first residency video | V3 |

---

## §9. Ten ideas: more content in less time

1. **One pillar, many pieces.** Each long YouTube video automatically becomes
   5–8 shorts (V4), IG/FB captions (content-engine already), a blog post for the
   Residency Guide / Propia sites, and a WhatsApp/newsletter blurb.
2. **Fact sheet per topic.** One file per topic (residency costs, requirements,
   processing times) with source URL and "last checked" date. Scripts pull
   facts from it; when a rule changes you update one line and the system lists
   which published videos now need a correction.
3. **Question mining.** Pull comments from the top Paraguay residency and real
   estate videos (YouTube Data API) and expat Facebook groups; cluster the
   questions; each cluster is a video topic with proven demand.
4. **Hook bank.** When a digest finds a strong opening line or title pattern
   from a successful video, "Save lesson" tags it as a hook. The script writer
   draws from your own bank instead of generic hooks.
5. **10 seconds of real you.** Record one on-camera intro and outro per month
   and reuse them; the AI does the middle. Builds trust and keeps the channel
   clearly "authentic" for YouTube's monetisation rules.
6. **Batch day.** Once a month: generate 8 scripts in one run (Gemini Batch API
   is half price), approve them in one sitting, render overnight.
7. **B-roll library.** Every Higgsfield clip and own drone/property shot goes
   into `library/` with tags; the renderer reuses before generating. Saves
   Higgsfield credits and makes the channel look consistent.
8. **Listing videos from Propia.** A property listing (photos + price + features)
   becomes a 30-second short with a Paraguayan voice, automatically (V5).
   A content stream that writes itself from data you already have.
9. **Title + thumbnail triples.** Every script ships 3 titles and 3 thumbnail
   concepts; use YouTube's built-in thumbnail "Test & Compare" and let viewers pick.
10. **"Implement one thing" Monday.** aiinsights sends a Telegram message every
    Monday with 3 saved-but-not-implemented AI lessons; reply with the one you
    will do this week, and it asks on Friday whether you did. Fixes the "saved
    it, never used it" problem the app was built for.

Bonus: **dub the best videos.** An English residency video that performs well
gets a Spanish and Portuguese (Brazilian buyers) version with the same b-roll
and a new voice track. One script, three audiences.

---

## §10. Backlog (not now)

- Pure Guaraní voice.
- Auto-posting / scheduling to YouTube, Meta, TikTok.
- Talking-avatar videos (HeyGen or similar): only if faceless underperforms.
- Selling the clip generator to clients (the original `app.contenido.com.py` idea).
- Deep learning features on YT digests (flashcards, spaced repetition).
- A desktop wrapper so content-engine and videoPY open as one app.
