# SwiftUI Perf Debugging

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that diagnoses scroll and animation jank in SwiftUI macOS/iOS apps. It uses Instruments `.trace` exports loaded into DuckDB to do **frame-by-frame** root cause analysis — not just "SwiftUI was slow."

## What it does

Given an Instruments trace, this skill will:

1. **Export** the trace to DuckDB (via `export_to_duckdb.py`)
2. **Isolate** interaction windows (scroll, animation) using signposts or heuristic clustering
3. **Rank** every dropped frame by severity (missed frames, hitch duration)
4. **Deep dive** the worst frames — attributing each to SwiftUI updates, RunLoop, Core Animation, GPU, or CPU work
5. **Cascade analysis** — detect when one bad frame causes subsequent frames to miss deadlines
6. **Cluster** root causes and estimate fix payoff
7. **Output** a prioritized fix plan with concrete validation steps

## Install

### As a Claude Code skill

```bash
claude install-skill freeze-rey/swiftui-perf-debugging
```

### Manual

Clone and copy the skill files into your project's `.claude/skills/` directory:

```bash
git clone https://github.com/freeze-rey/swiftui-perf-debugging.git
cp -r swiftui-perf-debugging/.  your-project/.claude/skills/swiftui-perf-debugging/
```

## Usage

### 1. Record a trace

```bash
xcrun xctrace record --template 'SwiftUI' --time-limit 20s \
  --output ./traces/recording.trace \
  --attach YourApp --no-prompt
```

Or use the included `PerfDebugging.tracetemplate` in Instruments.

### 2. Export to DuckDB

```bash
./scripts/export_to_duckdb.py traces/recording.trace traces/recording/analysis.duckdb
```

Requires [uv](https://github.com/astral-sh/uv) (dependencies are managed inline via PEP 723).

### 3. Ask Claude to analyze

```
Diagnose the scroll jank in traces/recording/analysis.duckdb.
The laggy scroll happens around 10-20s in the trace.
```

The skill will walk through a structured workflow and produce a detailed report with root causes and a prioritized fix plan.

## Example output

The skill produces a structured Markdown report:

- **Scope** — interaction type, mode (hitch vs smoothness), frame budget
- **Dropped frame inventory** — severity buckets, worst frames table
- **Frame deep dives** — per-frame attribution with evidence
- **Root cause clusters** — deduplicated causes ranked by total impact
- **Fix plan** — prioritized fixes with expected payoff and validation steps

## What's included

| File | Description |
|------|-------------|
| `SKILL.md` | The skill prompt (frame-first jank diagnosis workflow) |
| `SCHEMAS.md` | Full DuckDB schema reference for all exported tables |
| `scripts/export_to_duckdb.py` | Exports Instruments `.trace` to DuckDB + Parquet |
| `scripts/prepare_analysis.py` | Creates analysis views, frame summaries, cascade analysis |
| `PerfDebugging.tracetemplate` | Instruments template for recording traces |

## Requirements

- macOS with Xcode / Instruments
- [uv](https://github.com/astral-sh/uv) (for running the Python scripts)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

## License

MIT
