# agent-observer

Agent for the GOSIM 2026 Agentic Observer Hackathon (`survey26`), speaking
`participant-agent-protocol-v4`: one JSON message per line on stdin, one
`decision_response` per line on stdout, logs on stderr.

The strategy is a port of the official `rust-pro` example: a deterministic
planner picks pointing, fiber assignments, duration and program together;
instrument-fault reporting is driven by the quality/band ratio; an LLM
(Kimi Coding Plan by default) advises at each night start and confirms paid
fault reports. See `src/main.rs` for the decision loop.

## Usage

```sh
cargo build --release
./target/release/agent-observer run      # the v4 agent (stdin/stdout JSONL)
```

Without `OPENAI_API_KEY` (or `KIMI_API_KEY`, or a `.env` file — see
`.env.example`) the agent runs rules-only, no model calls.
`OBSERVER_MODEL_DISABLED=1` forces rules-only even with a key.

## Local evaluation

Using the official runner (engine identical to the platform's):

```sh
python3 tmp/gosim-observer-examples/runner/run_local.py \
  --card L1 --agent "./target/release/agent-observer run" --agent-cwd .
```

Cards L1/L2 are normal mode; L3/L4 add hard-mode `data_loss` and
`pointing_offset` events. Score breakdown lands in the `--out` directory
(`score_report.json`, `messages.jsonl`, `agent.log`, ...).

## Submission

`observer.project.json` declares the build (`cargo build --release --locked`)
and run (`./target/release/agent-observer run`) steps. Pack with:

```sh
python3 pack_agent.py --out agent-observer.zip
```

`.env` is never packed; set keys on the platform's Participate page.

## Layout

```
src/main.rs       protocol loop, pacing, fault reporting, model stages
src/planner.rs    pointing + fiber + duration + program search; learning from results
src/skymath.rs    sidereal time, alt/az, gnomonic projection, fiber grid, Moon
src/advisor.rs    LLM stages: night plan, fault review, paid-report confirmation
src/llm_client.rs OpenAI-compatible chat client on background threads
```

Strategy constants can be overridden with `PRO_<NAME>` environment variables
(e.g. `PRO_FIXED_LEVEL=0` pins the planner's search level).
