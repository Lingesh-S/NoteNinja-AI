# NoteNinja roadmap

**Date:** 2026-10-29 · See [CONCEPT.md](CONCEPT.md) for what is being built and [ARCHITECTURE.md](ARCHITECTURE.md) for how.

## How phases work

NoteNinja is built in phases. Each phase is a small, working step on top of the last.

1. **Plan.** A short implementation plan for the phase goes in `docs/plans/` and is reviewed before any code is written. Dependency versions are checked again at this point, because they move quickly.
2. **Build.** Work happens on a branch named after the phase (for example `phase-1-mvp`), in small commits.
3. **Check.** Tests, Ruff, mypy (strict) and CI pass on Ubuntu and Windows.
4. **Demo and review.** The phase ends with a working demo and a pull request. The next phase starts only after review.

Every phase must leave the project releasable: no half-built features on the main branch, and the README always describes what actually works.

## Status

| Phase | Goal | Status |
|---|---|---|
| 0 | Concept, architecture and roadmap | Done |
| 1 | MVP: review captions against Markdown or text notes | Next |
| 2 | Audio, video and typed note formats | Planned |
| 3 | Review UI | Planned |
| 4 | Handwriting and slides | Planned |
| 5 | Gap drills | Planned |
| 6 | Course view and MCP server | Planned |

---

## Phase 0: Foundation

**Goal:** agree on what to build and how, before writing code.

**Deliverables:**
- [CONCEPT.md](CONCEPT.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- this roadmap
- a rewritten [README](../README.md)

**Done criteria:**
- [x] Concept approved (2026-10-29)
- [x] Architecture and roadmap approved (2026-10-29)

---

## Phase 1: MVP

**Goal:** the smallest end-to-end version. One command turns lecture captions and Markdown notes into a verified review, fully offline.

**Scope:**
- **Project setup:** a uv project on Python 3.14 with the `src/` layout, Ruff, mypy (strict), pytest, pre-commit, `.env.example` and `.gitattributes`.
- **Lecture readers:** `.vtt`, `.srt` and timestamped `.txt`.
- **Note readers:** `.md` and `.txt`, which also covers Notion's and Google Docs' Markdown exports.
- **Pipeline:** windowing, extraction, quote location and duplicate removal, comparison, verdict checks and ranking.
- **Model client interface,** with Ollama and fake implementations, and a content-hash cache.
- **CLI:** `review`, `doctor` and `cache clear`.
- **Reports:** Markdown and JSON.
- **Eval harness v0** and a benchmark of four open models.
- **CI** on Ubuntu and Windows, plus Dependabot.

**Deliverables:**
- The `noteninja` package and CLI.
- `examples/tcp-congestion/`: a sample lecture, notes, and the review NoteNinja generated for them.
- `evals/datasets/v0/`: at least 3 synthetic lectures, each with at least 2 note variants and their labels.
- `evals/RESULTS.md`: the benchmark table.
- An updated README with setup steps, real sample output and the benchmark summary.

**Done criteria:**
- [ ] On a clean machine, `uv sync --locked` then `uv run noteninja review examples/tcp-congestion/lecture.vtt examples/tcp-congestion/notes.md` writes a review. This also works with the network off once the model is pulled.
- [ ] `uv run noteninja doctor` detects a missing Ollama server or model and prints the fix.
- [ ] Tests prove that every finding in the main list passed the evidence checks.
- [ ] Tests cover readers, windowing, the verifier, ranking, the cache, reports, the CLI and the Ollama adapter, with at least 85% coverage of `src/noteninja`.
- [ ] `ruff check`, `ruff format --check` and `mypy` (strict) pass.
- [ ] CI passes on Ubuntu and Windows.
- [ ] The benchmark of `qwen3.5:4b`, `ministral-3:3b`, `phi4-mini` and `granite4:3b` (3 seeds each) is recorded, and the default model is chosen by the rule in [ARCHITECTURE.md](ARCHITECTURE.md#11-evaluation-harness).
- [ ] The default model meets the first targets (precision ≥ 0.85, omission recall ≥ 0.75), or the gap is documented with next steps.
- [ ] Reviewing a 60-minute lecture from captions takes 15 minutes or less on the RTX 4050 laptop, or the measured time is documented.

**Demo:** run the sample review with the network off, then walk through the report and the benchmark table.

**Not in this phase:** audio or video, other note formats, the UI, drills.

---

## Phase 2: Audio, video and typed note formats

**Goal:** review against a recording instead of captions, and accept notes from Word, Google Docs and OneNote.

**Scope:**
- **Local transcription:** faster-whisper with word timestamps, on the GPU when available with a CPU (int8) fallback. It ships as an optional `audio` extra.
- **`noteninja transcribe`** writes a `.vtt`, and `review` accepts `.mp3`, `.wav`, `.m4a` and `.mp4` directly. Transcripts are cached.
- **Course vocabulary hints** to help with technical terms.
- **`.docx` notes** (python-docx), covering Word, Google Docs and OneNote's Word export.
- **Typed `.pdf` notes** (pypdf), covering OneNote's PDF export and other apps.

**Deliverables:**
- Media and document readers.
- Synthetic test audio, generated from the synthetic lectures with an open-source text-to-speech tool.
- Sample exports from Word, Google Docs and OneNote, made from the synthetic notes.
- Windows GPU setup notes.

**Done criteria:**
- [ ] `noteninja review lecture.mp3 notes.docx` works end to end on the laptop, using the GPU.
- [ ] The CPU fallback works and is covered by a test.
- [ ] The word error rate on the synthetic audio is measured and recorded.
- [ ] `.docx` and `.pdf` notes keep their headings and lists, and each app's export sample has a test.
- [ ] The core install still works without the `audio` extra, and CI checks both.
- [ ] Tests, Ruff, mypy and CI pass. Transcription tests that need a real model run locally only, and CI uses a stub.

**Demo:** review OneNote-exported Word notes against an `.mp3` of a synthetic lecture.

---

## Phase 3: Review UI

**Goal:** make reviewing fast: see a finding, hear the moment, fix the notes.

**Scope:**
- **`noteninja ui`**, a local web app (FastAPI, Jinja2, htmx served locally) bound to localhost.
- **Review page:** findings next to an audio or video player that jumps to each timestamp.
- **Accept or dismiss** each finding. Accepted fixes are written to Markdown notes after a timestamped backup, and dismissals are saved as future eval cases.

**Deliverables:**
- The UI module.
- Browser tests with Playwright.
- Screenshots in the README.

**Done criteria:**
- [ ] Clicking a finding plays the recording from that timestamp.
- [ ] Accepting a fix changes the notes file, and a backup exists. Undo restores it.
- [ ] The app works offline, with no external CDN or fonts.
- [ ] The main flows work with the keyboard alone.
- [ ] Playwright tests for the main flows pass in CI.
- [ ] Tests, Ruff, mypy and CI pass.

**Demo:** open a review, click a finding, hear the moment, accept the fix, and watch the notes update.

---

## Phase 4: Handwriting and slides

**Goal:** support paper notes and use slides as a second source.

**Scope:**
- **Handwritten notes:** photos (`.jpg`, `.png`, `.heic`), scanned PDFs, and OneNote ink exported to PDF. A local vision model reads them into note lines with a confidence for each line.
- **Confirmation step:** unsure words are highlighted in the UI for you to confirm before the review runs.
- **Slides (PDF) as a second source,** so findings can cite a slide number.

**Deliverables:**
- Handwriting and slide readers.
- Handwritten sample pages written for this repo.
- A vision model benchmark comparing `qwen3.5:4b`, `ministral-3:3b` and `gemma4:e2b`.

**Done criteria:**
- [ ] The character error rate on the sample pages is measured for each model, and the default is chosen from those results.
- [ ] Low-confidence words always reach the confirmation step. A test checks this.
- [ ] Findings can point to a slide number as well as a timestamp.
- [ ] Tests, Ruff, mypy and CI pass.

**Demo:** photograph a handwritten page, confirm the transcription, and review it against the lecture and slides.

---

## Phase 5: Gap drills

**Goal:** turn findings into practice that sticks.

**Scope:**
- **`noteninja drill`:** questions generated only from findings, with each answer checked against the lecture quote.
- **Scheduling:** FSRS spaced repetition across all lectures, stored in SQLite.
- **Anki export** (`.apkg`).

**Done criteria:**
- [ ] Every generated question can be answered from its cited quote. The eval harness checks a sample.
- [ ] Due drills are correct across several lectures and days. Tests use a fake clock.
- [ ] An exported deck imports into Anki.
- [ ] Tests, Ruff, mypy and CI pass.

**Demo:** run a drill session on the points missed in the last two sample lectures.

---

## Phase 6: Course view and MCP server

**Goal:** see patterns across a whole course, and let other tools use NoteNinja's results.

**Scope:**
- **Course folders** holding several lectures.
- **Weak-topics view** in the UI.
- **MCP server** (official Python SDK, stdio) with tools for listing reviews, weak topics and due drills.
- **Obsidian-friendly output:** callouts and links.

**Done criteria:**
- [ ] The weak-topics view is correct on a sample course of at least 4 lectures.
- [ ] The MCP tools are tested with the SDK's in-process client and work from a desktop MCP client.
- [ ] Tests, Ruff, mypy and CI pass.

**Demo:** ask an MCP client "what are my weak topics this week?" and open the matching findings.

---

## Later ideas (not scheduled)

- **Fine-tuning:** a QLoRA fine-tune of a small open model (up to about 3B parameters on a 6 GB GPU) on synthetic review data, if evals show the model is the bottleneck.
- **Other runtimes:** support for other open-model runtimes, such as llama.cpp or LM Studio servers.
- **Course-wide search** with local embeddings.
- **Very long notes:** splitting notes that don't fit the context budget.
