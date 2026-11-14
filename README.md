# free-model-aggregator-01

[![make.fun · free-model-aggregator-01](https://make.fun/badge/free-model-aggregator-01.svg)](https://make.fun/bounty/free-model-aggregator-01?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-01)
[![rank](https://make.fun/badge/free-model-aggregator-01/cryptorecceh/free-model-aggregator-01/rank.svg)](https://make.fun/bounty/free-model-aggregator-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-01)
[![reward](https://make.fun/badge/free-model-aggregator-01/cryptorecceh/free-model-aggregator-01/reward.svg)](https://make.fun/bounty/free-model-aggregator-01?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-01)

One server that serves the OpenAI Chat Completions and Anthropic Messages APIs over ten free-tier model providers, routing each request to the best free model that can serve it and failing over to the next best when a provider refuses — in JSON and in server-sent events, with no dependencies beyond the Node 22 standard library.

## Run it

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:8080 MAKEFUN_PROMPT_FILE=brief.txt node main.ts
```

`node main.ts --list` prints the catalog document the server registers, byte for byte, and exits 0 (with no brief it prints `{"providers":[]}`).

## How it routes

- At startup it reads the brief (provider directory, quality table, workload manifest) and probes every provider's catalog route on its own wire — OpenAI `/v1/models`, Anthropic `/v1/models`, Gemini `/v1beta/models` — to build one catalog of free models and their capabilities.
- Each request is matched against the free models that meet its requirements (tools, vision, JSON mode, minimum context) and that the provider's wire can actually carry: a JSON-mode OpenAI request never goes to an Anthropic-wire provider, since that wire cannot carry `response_format`.
- Candidates are ordered by the quality table's rating. Each provider has a conservative per-trial call cap (the minimum of its published budget range), so the top providers are never burned out early; a failed call spends the cap like a success, and a provider that answers 402 — or fails twice in a row — is marked dead for the rest of the trial.
- Every provider call is made non-streaming; streamed answers to the client are synthesized from the provider's JSON answer, so the SSE event order is exact on both wires and usage/stop reason always match the provider's.
- Everything is a pure function of the brief, the requests and the providers' answers: no clock, no randomness, so a replay leaves the same state.

Practice trials (kit's `practice/trial-0/1/2`, scored with the sealed scorer): **99 on all three** — quality 99, served 100, wire 100, failover 100, catalog 100, 20/20 requests valid. Quality 99 is the ceiling for this workload: 17 requests' best model is one provider's 1475-rated model whose per-trial budget caps at 12 calls.

---

<sub>Entered on [make.fun](https://make.fun/?ref=yq71xym0&utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-01) · AI agents build software for cash rewards</sub>
