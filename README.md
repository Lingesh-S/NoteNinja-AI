# NoteNinja

**Your notes, reviewed.** NoteNinja compares the notes you took with the lecture you took them from. It shows what you missed and what you wrote down differently, and links every finding to the moment in the recording. Later phases add a drill on just those gaps. Everything runs on your own computer with free, open-source models.

> **Status: early development.** The concept and design are done, and Phase 1, the first working version, is next. See the [roadmap](docs/ROADMAP.md).

## Why

In a live lecture you can't write everything down. Checking your notes against the recording means rewatching the whole thing, so gaps stay hidden until the exam.

Most AI study tools solve this by writing the notes for you. A CSCW 2025 study ([Chen et al.](https://arxiv.org/abs/2509.03392)) found that fully automated AI notes led to the lowest test scores, even though participants preferred them. NoteNinja takes the other route: you keep writing your own notes, and it reviews them.

## What a review looks like

This is example output from the design, for a made-up lecture on TCP congestion control:

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

## How it works

1. The lecture transcript is split into short, overlapping time windows.
2. A local model lists the teaching points in each window, each with an exact quote.
3. A second step compares every point with your notes and labels it covered, partial, missed or conflict.
4. Plain code then checks the evidence:
   - every quote must appear in the transcript;
   - timestamps come from where the quote was found, never from the model;
   - cited note lines must exist.
5. You get a short, ranked review. Later phases add a drill on just your gaps.

NoteNinja isn't a chatbot and doesn't use retrieval. It checks every part of the lecture, and you can trace every finding back to its source. The details are in [ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Local, free and private

- **Open-weight models through [Ollama](https://ollama.com).** The default is `qwen3.5:4b`, a 3.3 to 4 GB download.
- **Nothing leaves your machine.** There are no paid APIs, no accounts, no API keys and no telemetry.
- **Target hardware:** a laptop with a 6 GB NVIDIA GPU (RTX 4050 class) and 16 GB of RAM. It also runs on the CPU, just more slowly.

## Roadmap

| Phase | What you get | Status |
|---|---|---|
| 1 | Review captions (`.vtt`, `.srt`) against Markdown or text notes from the command line; evaluation harness; open-model benchmark | Next |
| 2 | Audio and video with local transcription; Word and PDF notes, covering Word, Google Docs and OneNote exports | Planned |
| 3 | Local review UI: jump to each moment, accept suggested fixes | Planned |
| 4 | Handwritten notes and slides | Planned |
| 5 | Gap drills with spaced repetition and Anki export | Planned |
| 6 | Course dashboard and an MCP server | Planned |

Each phase ships with tests, linting, CI and a demo. [ROADMAP.md](docs/ROADMAP.md) lists the deliverables and done criteria for every phase.

## Tech stack

Python 3.14, uv, Ollama, Pydantic, Typer, RapidFuzz, pytest, Ruff, mypy (strict) and GitHub Actions. Later phases add faster-whisper, FastAPI with htmx, and FSRS. Versions and the reasons for each choice are in [CONCEPT.md](docs/CONCEPT.md#9-tech-stack).

## Getting started

Setup instructions will arrive with Phase 1. You'll need:

- [uv](https://docs.astral.sh/uv/), which also installs the right Python version;
- [Ollama](https://ollama.com), with the default model pulled: `ollama pull qwen3.5:4b`.

## Documentation

- [Concept](docs/CONCEPT.md): the problem, research, alternatives considered and decisions
- [Architecture](docs/ARCHITECTURE.md): components, data flow, interfaces and testing
- [Roadmap](docs/ROADMAP.md): phases, deliverables and done criteria

## License

[MIT](LICENSE). Made by Lingesh S ([@Lingesh-S](https://github.com/Lingesh-S)).
