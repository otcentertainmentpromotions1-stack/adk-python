## Project Overview

The Agent Development Kit (ADK) is an open-source, code-first Python toolkit for building, evaluating, and deploying sophisticated AI agents.

### Key Components

- **Agent**: Blueprint defining identity, instructions, and tools.
- **Runner**: Stateless execution engine that orchestrates agent execution.
- **Tool**: Functions/capabilities agents can call.
- **Session**: Conversation state management.
- **Memory**: Long-term recall across sessions.
- **Workflow** (ADK 2.0): Graph-based orchestration of complex, multi-step agent interactions.
- **BaseNode** (ADK 2.0): Contract for all nodes, supporting output streaming and human-in-the-loop steps.
- **Context** (ADK 2.0): Holds execution state and telemetry context mapped 1:1 to nodes.

For details on how the Runner works and the invocation lifecycle, please refer to the `adk-architecture` skill and the referenced documentation therein.

## ADK Knowledge, Architecture, and Style

Skills related to ADK development are in `.agents/skills/`.

## Project Architecture

For detailed architecture patterns, component descriptions, and core interfaces, please refer to the **`adk-architecture`** skill at `.agents/skills/adk-architecture/SKILL.md`.

## Development Setup

The project uses `uv` for package management and Python 3.11+. Please refer to
the **`adk-setup`** skill at `.agents/skills/adk-setup/SKILL.md` for detailed
instructions.

## Running the app in the Base44 sandbox

`docker compose -f docker-compose.base44.yml up -d` runs the ADK dev UI on host
port 3000 (`adk web` inside the container on 8000) against the sample agents in
`contributing/samples/core`.

- The image (`Dockerfile.base44`) contains only `uv`; the checkout is bind
  mounted and dependencies are installed on container start with
  `uv pip install --system -e '.[eval]'`. The `eval` extra is required: the
  evaluation modules imported by the CLI and by `adk eval` need `rouge-score`,
  which is not a core dependency. A core-only install cannot import
  `google.adk.evaluation.response_evaluator`.
- `adk web` rejects requests whose `Origin` does not match the request host
  (403 "origin not allowed"). Behind the preview proxy the browser origin is
  `https://3000-$BASE44_PUBLIC_HOST_SUFFIX` while the server sees a different
  Host, so compose passes that origin via `--allow_origins`. Without it the dev
  UI's module scripts (fetched in CORS mode, which sends `Origin`) return 403 and
  the page stays blank.
- Verify: `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/dev-ui/`
  returns 200, `/list-apps` lists the samples, and the UI renders the app
  selector. Sending a chat message needs `GOOGLE_API_KEY`.
- Selecting an app also calls `/dev/apps/{app}/builder`, which only accepts
  loopback clients (403 "This /dev endpoint only accepts requests from the
  machine running the server"). It backs the Agent Builder YAML editor, so that
  editor does not work through the preview proxy. Setting
  `ADK_ALLOW_REMOTE_AGENT_BUILDER=1` would enable it, at the cost of allowing any
  client that can reach the server to read and write agent files; leave it unset
  unless that trade-off is wanted.
- Known cosmetic issue, unrelated to this setup: the shipped dev UI bundle
  requests `prism-dark.css`/`prism-light.css`, which are absent from
  `src/google/adk/cli/browser`, so the console shows one 404 for code
  highlighting.
- Run tests inside the container, e.g.
  `docker compose -f docker-compose.base44.yml exec -T web python -m pytest tests/unittests/evaluation/test_final_response_match_v1.py -q`.
