# NoteNinja: concept

**Status:** proposal, waiting for approval · **Date:** 2026-10-29 · **Replaces:** the "chat with your lecture notes" idea in the old README

> **Your notes, reviewed.** NoteNinja compares the notes you took with the lecture you took them from. It shows what you missed and what you wrote down differently, and links every finding to the moment in the recording. Then it quizzes you on just those gaps.

You keep writing your own notes. NoteNinja is the second pass: a reviewer, not a ghostwriter.

---

## 1. Candidates considered

Research on AI note-taking tools (October 2026) produced three directions. Each one has a different user and problem.

| | Concept | For | Problem | State of the space | Verdict |
|---|---|---|---|---|---|
| **A** | **Second pass**: review your own notes against the lecture | Students and self-learners who take their own notes | Notes have gaps and errors that nobody catches until the exam | Many tools write notes *for* you. None found that reviews *your* notes against the source | **Recommended** |
| B | **Vault gardener**: code-review-style upkeep of a Markdown vault (duplicates, contradictions, stale notes) | Obsidian and PKM power users | Notes rot: duplicates, contradictions, outdated facts | Already covered piece by piece by Obsidian plugins (Contradiction Finder, Prior Art, Dualyze Notes, Burnish) | Not chosen: little room to stand out |
| C | **Decision ledger**: track decisions across meeting notes and flag reversals | Small teams, student project groups | "What did we decide, and why?" gets lost | Crowded (Fabric, Sentra, Decision Log, Decisions.ai), and hard to demo or evaluate without real team data | Not chosen |

**Why A:**
- It's the least-served need: no mainstream product does it.
- It answers the main criticism of AI study tools, that they do the thinking for you.
- Its quality can be measured with precision and recall on labelled data. That makes it testable and a strong portfolio project.

Also ruled out early:
- **AI flashcards from notes:** RemNote, Anki with FSRS, Knowt.
- **"Explain it back" tutors:** ReviseMate, Feynman AI, Teachback.
- **Sharing your notes with other AI apps over MCP:** Unabyss, Reflect's MCP server.

NoteNinja keeps the strongest idea from flashcards and tutors, retrieval practice, but uses it only on the gaps it finds.

## 2. The problem

In a live lecture you can't write everything down. While you write one point you miss the next. You mishear numbers, names and terms. A recording often exists, but checking your notes against it means rewatching 60 to 90 minutes. Almost nobody does that, so the gaps stay hidden until a problem set or exam exposes them.

Today's AI tools respond by taking the notes for you. Turbo AI, Knowt, Mindgrasp, StudyFetch and Coconote (bought by Quizlet in February 2026) turn a recording into notes, flashcards and quizzes. That's convenient, but it skips the work that makes taking notes useful for learning.

A CSCW 2025 study of AI-supported note-taking (Chen et al., 30 participants) compared three levels of AI help. Fully automated AI notes gave the **lowest** post-test scores, even though participants preferred them. Moderate help gave the highest.

So the need is this: **keep writing your own notes, but find out quickly and reliably what you missed and what you got wrong.**

## 3. Who it's for

**Primary:** university students who take their own notes, typed or handwritten, in lectures that are recorded or have captions or slides. It matters most in dense technical courses (CS, engineering, medicine), where a missed definition or a wrong number costs marks.

**Also:**
- Self-learners following recorded courses and talks.
- Professionals in recorded training or certification courses.

**Typical user:** a second-year CS student who types notes in Markdown (Obsidian or VS Code) during lectures. The university's learning platform posts recordings with auto-captions a day later. Before exams, the student skims their notes but never rewatches whole lectures.

**Not for:** people who want AI to write their notes, team meeting minutes, or live help during the lecture.

## 4. What's new

### 4.1 Compared with existing tools

| Tool (Oct 2026) | Writes notes for you | Chat or Q&A over sources | Links your notes to the recording | Reviews **your** notes against the source |
|---|---|---|---|---|
| NotebookLM | Study guides, overviews | Yes | No | No |
| Turbo AI, Knowt, Mindgrasp, StudyFetch, Coconote | Yes | Some | No | No |
| Otter.ai | Transcript, summary | Yes | No | No (you compare by hand) |
| Genio Notes, Notability | AI outline or summary | Some | Yes | No |
| Notion AI, Mem, Reflect, Obsidian Copilot | On request | Yes | No | No |
| **NoteNinja** | **No, by design** | **No** | **Through timestamps** | **Yes** |

This is based on public product pages and reviews (see [Sources](#sources)). I found no product that offers the last column.

### 4.2 Compared with a plain RAG chatbot

A RAG chatbot answers questions you already know to ask. It fetches the few chunks that best match your question and writes an answer. NoteNinja answers a question you can't ask: *what did I miss?* That needs a different design.

| | Plain RAG chatbot | NoteNinja |
|---|---|---|
| Starts from | Your question | Your notes plus the source, no question needed |
| Coverage | The top few matching chunks | Every part of the source is checked |
| Output | Free-text answer | Typed findings (`missed`, `partial`, `conflict`) with timestamp, quote and note line |
| Trust | Citations at best | Code checks every quote and timestamp before showing it |
| Quality | Hard to measure | Precision and recall on labelled test sets |
| Storage | Vector database | None needed for the MVP, since a lecture fits in the model's context |

## 5. How it works

1. **You give it two files:** the lecture (captions or a transcript in the MVP, audio, video and slides later) and your notes (Markdown or plain text).
2. **It maps the lecture.** The transcript is split into short, overlapping time windows (about 4 minutes each, overlapping by 30 seconds; both adjustable). A model lists the *teaching points* in each window: every fact, definition, rule, step or warning the lecturer states. Each point comes with a short exact quote, its key terms and an importance level (high, medium or low).
3. **It checks your notes.** A second step compares every teaching point with your notes, gives it one of four labels and points to the note lines involved:

   | Label | Meaning |
   |---|---|
   | `covered` | Your notes capture the point |
   | `partial` | Your notes mention it but miss a key detail |
   | `missed` | Your notes don't have it |
   | `conflict` | Your notes say something different from the lecture |
4. **It verifies before it shows anything.** Code, not the model, checks that:
   - every quote really appears in the transcript, and takes the timestamp from where the quote was found (it never trusts a timestamp the model wrote);
   - every cited note line exists and says what the model claims;
   - a "missed" point's key terms really don't appear in your notes. If they do, the finding is moved to "worth a check".

   Findings that fail these checks are dropped and counted.
5. **You get a review.** It's a short ranked list, conflicts first and then important missed points. Each finding shows what was said, when it was said, what your notes say, and a suggested one-line fix. In the MVP you copy fixes in yourself; from Phase 3 the review UI can apply the ones you accept. From Phase 5, a short *gap drill* follows each review, with questions on only what you missed.

Extra material in your notes (from a textbook, say) is fine. It's only flagged if it contradicts the lecture.

Example output, for a made-up lecture on TCP congestion control:

```text
CONFLICT · high importance · 23:41-23:58
  Lecture:    "on a timeout, cwnd drops back to one segment and we go into slow start again"
  Your notes: line 42: "timeout -> cwnd halves"
  Suggested:  line 42: "timeout -> cwnd = 1 MSS, back to slow start (halving is for 3 dup ACKs)"

MISSED · high importance · 31:05-31:40
  Lecture:    "the receiver window caps all of this, the sender can never exceed rwnd"
  Your notes: nothing in "Flow control" (lines 50-58)
  Suggested:  after line 55: "sender rate <= min(cwnd, rwnd)"
```

Teaching points are cached per lecture. After you fix your notes, a new review only reruns the cheaper comparison step.

## 6. Features: MVP vs later

| Phase | Scope | Demo |
|---|---|---|
| **1: MVP** | <ul><li>CLI `noteninja review lecture.vtt notes.md`</li><li>Inputs: `.vtt` or `.srt` captions, or a timestamped transcript, plus Markdown or text notes</li><li>Teaching-point extraction, comparison with notes, evidence checks</li><li>Cache of model results keyed by content hash</li><li>Ranked Markdown and JSON report</li><li>Claude, Ollama and fake (test) providers</li><li>Evaluation harness v0 on synthetic lectures</li></ul> | One command on the bundled sample lecture and notes gives a review with verified findings |
| 2: Audio and video | Local transcription with word timestamps (faster-whisper on the GPU if there is one, CPU otherwise); transcript cache; course vocabulary hints | Review notes against an `.mp3` or `.mp4` |
| 3: Review UI | Local web app with findings next to a player that jumps to each timestamp; accept or dismiss suggestions; accepted fixes are written into the notes file, with a backup | Click a finding, hear the moment, accept the fix |
| 4: More inputs | Slides (PDF) as a second source; photos of handwritten notes read by a vision model, with unsure words flagged for you to confirm | Review handwritten notes against the lecture and slides |
| 5: Gap drills | Questions built only from findings; spaced-repetition scheduling (FSRS) across all lectures of a course; Anki export | A daily drill on the points you keep missing |
| 6: Course view and integrations | Course dashboard (weak topics across lectures); an MCP server so assistants such as Claude Desktop can ask "what are my weak topics this week?"; Obsidian-friendly output | Ask an MCP client about your weak topics |

Each phase ships with tests, linting, CI and a working demo, and ends with a stop for review. After approval, `docs/ROADMAP.md` will list deliverables and done criteria for each phase.

**Out of scope:** live help during the lecture, recording lectures, shared team workspaces, a hosted multi-user service, and mobile apps.

## 7. AI tech it builds on, and why

| Capability | How NoteNinja uses it | Why it's possible now |
|---|---|---|
| Structured outputs | Every model call returns a schema-checked object (Pydantic models), so there's no parsing of free text | Claude (`output_config.format`, `messages.parse`) and Ollama (`format` set to a JSON schema) both enforce schemas |
| Long context | Your whole notes plus a transcript window (or a whole lecture, as a baseline) fit in one call | Current Claude models take 1M tokens; Qwen3.5 on Ollama takes 256K |
| Local speech recognition with word timestamps | Precise anchors for every finding; audio never leaves your machine (Phase 2) | faster-whisper (CTranslate2) runs on a 6 GB laptop GPU or on the CPU |
| Small local models | A fully offline mode with `qwen3.5:4b` (3.3 to 4 GB download) | 4B-parameter models now fit a 6 GB GPU with room left for context |
| Vision | Reading handwritten notes and slides (Phase 4) | All current Claude models and Qwen3.5 accept images |
| Spaced repetition (FSRS) | Scheduling gap drills (Phase 5) | FSRS is the standard scheduler in Anki and RemNote, and has a maintained Python library |
| MCP | Sharing reviews and weak topics with other assistants (Phase 6) | The current spec (2026-07-28) has no session handshake, so servers are simpler to run; the official Python SDK is at 2.x |

**Deliberately not used:**
- **Vector database or RAG retrieval in the MVP.** The task needs every part of the lecture checked, not the top few matches. Embeddings may come back later for search across a course.
- **An autonomous agent.** The steps are known in advance, so a fixed pipeline is cheaper, more predictable and easier to test.
- **Claude's built-in citations.** They can't be used together with structured outputs (the API returns a 400), so NoteNinja verifies quotes itself. That also works for local models.
- **Fine-tuning.** It isn't needed for the MVP. The eval harness will show whether it's worth doing later.

## 8. Architecture

```mermaid
flowchart TD
    subgraph IN["Inputs"]
        CAP["Captions or transcript<br/>.vtt / .srt / .txt"]
        MED["Audio or video<br/>(Phase 2)"]
        SLD["Slides PDF<br/>(Phase 4)"]
        NOTES["Your notes<br/>.md / .txt"]
    end

    MED --> ASR["Local transcription<br/>faster-whisper"]
    ASR --> TL
    CAP --> TL["Source timeline<br/>segments with start and end times"]
    SLD -.-> TL
    NOTES --> NL["Note lines<br/>with line numbers"]

    TL --> WIN["Windowing<br/>overlapping time windows"]
    WIN --> EXT["Step 1: Extract teaching points<br/>LLM, structured output"]
    EXT --> CMP["Step 2: Compare with notes<br/>LLM, structured output"]
    NL --> CMP
    CMP --> VER["Step 3: Verify evidence<br/>plain code, no LLM"]
    TL --> VER
    NL --> VER
    VER --> RANK["Step 4: Rank and remove duplicates"]
    RANK --> OUT["Review report<br/>Markdown + JSON"]
    OUT --> UI["Review UI<br/>(Phase 3)"]
    OUT --> DRILL["Gap drills + FSRS<br/>(Phase 5)"]
    OUT --> MCPS["MCP server<br/>(Phase 6)"]

    CACHE[("Cache<br/>keyed by content hash")]
    EXT -.- CACHE
    CMP -.- CACHE

    subgraph PORT["LLM port: one interface, three adapters"]
        AC["Claude API<br/>anthropic SDK"]
        OC["Ollama<br/>local model"]
        FC["Fake client<br/>for tests"]
    end
    EXT -.-> PORT
    CMP -.-> PORT
```

How quality is measured (the eval harness):

```mermaid
flowchart LR
    GOLD["Synthetic lecture<br/>+ complete reference notes"] --> DEG["Remove points and inject errors<br/>(each change is labelled)"]
    DEG --> RUN["Run the review pipeline<br/>per provider, several runs"]
    RUN --> SCORE["Compare findings with labels"]
    SCORE --> MET["Precision, recall, evidence pass rate,<br/>cost and time per lecture"]
```

Key design decisions:

- **The model writes quotes; code writes timestamps.** A timestamp always comes from where the quote matched in the transcript, so it can't be made up.
- **Two cached stages.** Teaching points are cached by a hash of the transcript, and comparison results by a hash of the points and notes. Reviewing edited notes again is cheap.
- **One LLM port, three adapters.** The pipeline depends on an `LLMClient` interface with Claude, Ollama and fake implementations. It never imports an SDK directly, and tests never touch the network.
- **Precision before recall in what's shown.** A wrong "you missed this" costs trust. Uncertain findings go into a collapsed "worth a check" section instead of the main list.
- **Plain files.** Inputs and outputs are Markdown, JSON and caption files. There's no database until spaced repetition needs one (Phase 5, SQLite).
- **Config from the environment.** `pydantic-settings` reads `NOTENINJA_*` variables and an optional `.env` file. API keys never go in the repo, and `.env.example` documents the settings.
- **Privacy:**
  - Audio is always transcribed locally.
  - With the Ollama provider, nothing leaves your machine.
  - With the Claude provider, transcript text and notes are sent to the Anthropic API, and the CLI says so on first use.
  - There's no telemetry.

## 9. Tech stack

Versions were checked on 2026-10-29 against PyPI, the official docs and GitHub releases. Exact versions will be locked in `uv.lock`.

| Area | Choice | Version | Why |
|---|---|---|---|
| Language | Python | 3.14 (3.14.10) | Strong ML and audio ecosystem. 3.14 gets bug fixes until 2030. 3.15 is out, but onnxruntime (which faster-whisper needs) has no 3.15 wheels yet |
| Environment and packaging | uv | 0.13.6 | One tool for the Python install, virtual environment and lockfile; `uv sync` gives the same setup everywhere |
| Lint and format | Ruff | 0.17.3 | Fast; replaces flake8, isort and black |
| Type checking | mypy (strict) | 2.4.1 | Mature. Astral's `ty` (0.0.94) is still before 1.0 |
| Tests | pytest, pytest-cov, Hypothesis | 9.2.0, 7.1.0, 6.170.13 | Hypothesis for property tests of the parsers and the evidence checker |
| Data models | Pydantic | 2.14.1 | The same schemas serve model outputs, reports and tests |
| Config | pydantic-settings | 2.16.2 | Typed settings from env and `.env` |
| CLI | Typer, Rich | 0.27.3, 15.0.0 | Typed commands and readable terminal output |
| Cloud model | Anthropic Python SDK, default model `claude-opus-5-5` | 2.3.0 | Structured outputs, 1M context, vision. The model is set in config |
| Local model | Ollama server and `ollama` Python client, model `qwen3.5:4b` | 0.40.13, 0.6.3 | JSON-schema outputs; fits a 6 GB GPU |
| Captions | webvtt-py | 0.6.0 | Reads WebVTT, SRT and SBV |
| Quote matching | RapidFuzz | 3.14.6 | Fast fuzzy matching for the evidence checks |
| Transcription (Phase 2) | faster-whisper on CTranslate2 | 1.2.1 on 4.8.4 | Word timestamps on CUDA or CPU, MIT license. Last release Oct 2025, still active (commits in Oct 2026) |
| Slides (Phase 4) | pypdf | 6.21.4 | BSD license (avoids the AGPL-licensed PyMuPDF) |
| Review UI (Phase 3) | FastAPI, Uvicorn, Jinja2, htmx | 0.162.0, 0.54.0, 3.1.6, 2.0.11 | Small server-rendered app with almost no hand-written JavaScript |
| Spaced repetition (Phase 5) | fsrs (py-fsrs) | 6.3.2 | Reference FSRS implementation |
| MCP server (Phase 6) | `mcp` (official Python SDK) | 2.3.0 | Supports spec 2026-07-28 |
| CI | GitHub Actions with `actions/checkout`, `astral-sh/setup-uv` | v7.0.1, v10.3.3 | Ubuntu and Windows matrix |
| Git hooks | pre-commit with ruff-pre-commit | 4.7.0, v0.17.3 | The same checks run locally and in CI |

All runtime dependencies use MIT, BSD or Apache-2.0 licenses, which fit the project's MIT license.

## 10. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| False "you missed this" findings | Users stop trusting it | Evidence checks, the key-term absence check, precision-first ranking, a collapsed "worth a check" section; precision tracked in evals |
| Shorthand and paraphrase in notes ("cwnd ½ on loss") | Points that are covered get reported as missed | A `partial` label; the model sees the whole notes; dismissed findings are kept as future test cases |
| Transcription errors on technical terms or accents (Phase 2) | Wrong quotes, so wrong findings | Prefer official captions when they exist; vocabulary hints; flag low-confidence words |
| CUDA setup on Windows is fiddly (Phase 2) | GPU transcription fails | CPU (int8) fallback always works; setup documented and tested in CI on Windows |
| Recording consent and copyright | Legal or ethical trouble for users | Only process sources you have the right to use; local processing; no upload or sharing features; stated in the README. Sample data is original or public domain |
| Cloud cost | Expensive to use or to evaluate | Caching, prompt caching, Batch API for eval runs, cost per lecture reported; free local mode |
| A 4B local model is too weak for the comparison step | Local mode gives poor results | Measured in evals; optional hybrid mode (local extraction, cloud comparison) |
| Outputs vary between runs (current Claude models don't accept a custom `temperature`) | Flaky tests, noisy evals | Tests use the fake client; evals run several times and report the mean and spread |
| Scope creep from many input types | The MVP slips | Text-only MVP; one new input type per phase; a review stop after every phase |
| Fast-moving SDKs | Breakage | `uv.lock`, Dependabot, CI on every PR; adapters keep SDK changes in one place |
| The learning benefit of this exact design is unproven | Overclaiming | Claim only what's measured (detection quality, time saved); cite related research honestly |

## 11. Open questions

1. **Default cloud model.** It follows current guidance: `claude-opus-5-5`. A rough estimate for a 60-minute lecture, to be measured in Phase 1:

   | Model | First review | Review after editing notes |
   |---|---|---|
   | Opus 5.5 | about $1 to $1.50 | about $0.20 |
   | Sonnet 5.5 | about $0.60 | about $0.10 |
   | Haiku 5.5 | a few cents | under a cent |
   | Local | $0 | $0 |

   Keep Opus as the default, or choose another?
2. **Local or cloud by default?** Audio stays local either way, but the transcript and notes go to Anthropic when the Claude provider is used. Should the first run default to local (private, weaker) or cloud (stronger, needs an API key)?
3. **Note format.** The MVP reads Markdown and plain text. Is that how you and your target users take notes, or do OneNote, Notion or paper matter more?
4. **Real data.** Do you have lecture recordings or captions you're allowed to use for a private test set? If not, evals use synthetic lectures only.
5. **Phase order.** The review UI (Phase 3) comes before slides and handwriting (Phase 4). Swap them?
6. **Naming and license.** Keep the repo name `NoteNinja-AI`, with `noteninja` as the CLI and package name? Keep the MIT license?

## 12. How success is measured

| Area | Metric | First target | How it's measured |
|---|---|---|---|
| Finding gaps | Recall of seeded omissions of important points | ≥ 0.80 | Eval harness |
| Trust | Precision of shown `missed` and `conflict` findings | ≥ 0.85 | Eval harness |
| Catching errors | Recall of seeded contradictions | ≥ 0.70 | Eval harness |
| Grounding | Share of raw model findings that pass the evidence checks | ≥ 0.95 (shown findings pass 100%, since failures are dropped) | Checker counters in every run |
| Time for the user | Time to review notes for a 60-minute lecture | Under 10 minutes, compared with 60+ minutes of rewatching | Timed trial on real lectures |
| Cost | Cloud cost per 60-minute lecture | Reported per model; ≤ $1.50 on the default | Token usage logged in eval runs |
| Local mode | The same quality metrics with `qwen3.5:4b` on an RTX 4050 (6 GB) | Reported, with the gap to cloud documented | Eval harness |
| Engineering | CI passing on Ubuntu and Windows, mypy strict clean, at least 85% test coverage of the core package, one-command demo | Every phase | CI |

These targets are first guesses. Phase 1 sets the baseline, and the roadmap may adjust the targets based on what it shows.

## Sources

Research done in October 2026. Many product details come from reviews and secondary sources, so specifics may have changed.

**Learning research**
- Chen et al., *More AI Assistance Reduces Cognitive Engagement: Examining the AI Assistance Dilemma in AI-Supported Note-Taking*, CSCW 2025: <https://arxiv.org/abs/2509.03392>

**Note-taking and study tools**
- AI lecture note tools for students: <https://www.meetjamie.ai/blog/best-ai-note-takers-for-students>
- Turbo AI (formerly TurboLearn): <https://scribbl.co/post/turbo-ai-notetaker-alternative>
- Coconote and the Quizlet acquisition: <https://makerstack.co/reviews/coconote-review/>
- Genio Notes: <https://learningtech.virginia.edu/tools/genio>, <https://dres.illinois.edu/accommodations/ams/assitive-technology/note-taking-software/>
- Notability, GoodNotes, OneNote Copilot: <https://www.youngju.dev/blog/culture/2026-05-16-ai-digital-ink-handwritten-notes-2026-apple-pencil-math-notes-goodnotes-6-notability-concepts-onenote-copilot-nebo-supernote-remarkable-deep-dive.en>
- NotebookLM 2026 updates: <https://nokiapoweruser.com/google-notebooklm-upgrade-agentic-features-ai-ultra/>
- Notion AI agents: <https://eesel.ai/blog/notion-ai-review>
- Mem: <https://blog.saner.ai/mem-ai-reviews>
- Reflect: <https://yespress.io/products/reflect.md>
- Obsidian AI plugins: <https://unabyss.com/blog/best-ai-plugins-for-obsidian>
- Vault maintenance plugins:
  - <https://community.obsidian.md/plugins/contradiction-finder>
  - <https://community.obsidian.md/plugins/prior-art>
  - <https://forum.obsidian.md/t/dualyze-notes-find-similar-notes-and-create-safe-merge-drafts/115158>
  - <https://community.obsidian.md/plugins/burnish>
- Decision logs:
  - <https://fabric.so/solutions/decision-log>
  - <https://www.sentra.app/articles/ai-meeting-memory>
  - <https://www.upmeet.ai/en/blog/meeting-ai-is-starting-to-track-decision-drift>
- Flashcards and explain-back apps: <https://www.remnote.com/feature/spaced-repetition>, <https://teachback-web.pages.dev/>

**Technology**
- Claude models and pricing: <https://platform.claude.com/docs/en/about-claude/models/overview>
- Claude structured outputs: <https://platform.claude.com/docs/en/build-with-claude/structured-outputs>
- Claude citations (incompatible with structured outputs): <https://platform.claude.com/docs/en/build-with-claude/citations>
- Ollama structured outputs: <https://docs.ollama.com/capabilities/structured-outputs>
- Qwen3.5 on Ollama: <https://ollama.com/library/qwen3.5>
- Small models for 6 GB of VRAM: <https://localaimaster.com/vram/best-ollama-models-6gb-vram>
- Local speech-to-text comparison: <https://openwhispr.com/blog/parakeet-vs-whisper-vs-nemotron>
- faster-whisper: <https://github.com/SYSTRAN/faster-whisper>
- MCP versioning: <https://modelcontextprotocol.io/docs/learn/versioning>
- MCP Python SDK 2.3.0: <https://github.com/modelcontextprotocol/python-sdk/releases/tag/v2.3.0>
- Python release status: <https://www.python.org/downloads/>
- uv installation: <https://docs.astral.sh/uv/getting-started/installation/>
- Package versions: PyPI JSON API (`https://pypi.org/pypi/<package>/json`), checked 2026-10-29
