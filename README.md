# SwiftUI Scroll & Animation Jank Diagnosis

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that diagnoses scroll and animation jank in SwiftUI macOS/iOS apps. It uses a frame-first workflow to isolate interaction windows, rank dropped frames, attribute each hitch to SwiftUI updates, CPU work, GPU/render, or RunLoop contention, and produce a prioritized fix plan.

This is a use-case skill that builds on [claude-instruments](https://github.com/jlreyes/claude-instruments), which provides the Instruments → DuckDB export pipeline, scripts, and schema reference.

## Install

### As a Claude Code skill

Install both the foundation and this skill:

```bash
claude install-skill jlreyes/claude-instruments
claude install-skill jlreyes/swiftui-scroll-animation
```

### Manual

```bash
git clone https://github.com/jlreyes/claude-instruments.git
cp -r claude-instruments/. your-project/.claude/skills/claude-instruments/

git clone https://github.com/jlreyes/swiftui-scroll-animation.git
cp -r swiftui-scroll-animation/. your-project/.claude/skills/swiftui-scroll-animation/
```

## Usage

### 1. Record and export a trace (via claude-instruments)

```bash
xcrun xctrace record --template 'SwiftUI' --time-limit 20s \
  --output ./traces/recording.trace \
  --attach YourApp --no-prompt

./scripts/export_to_duckdb.py traces/recording.trace traces/recording/analysis.duckdb
```

See [claude-instruments](https://github.com/jlreyes/claude-instruments) for full recording and export details.

### 2. Ask Claude to diagnose

```
Diagnose the scroll jank in traces/recording/analysis.duckdb.
The laggy scroll happens around 10-20s in the trace.
```

The skill walks through a structured workflow and produces a detailed report with root causes and a prioritized fix plan.

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
| `SKILL.md` | The skill prompt — frame-first jank diagnosis workflow |

Scripts, schemas, and the trace template live in [claude-instruments](https://github.com/jlreyes/claude-instruments).

## Requirements

- [claude-instruments](https://github.com/jlreyes/claude-instruments) skill (provides export scripts, schemas, and trace template)
- macOS with Xcode / Instruments
- [uv](https://github.com/astral-sh/uv) (for running the Python scripts)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

## License

MIT
