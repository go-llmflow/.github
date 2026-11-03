# go-llmflow

[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](https://github.com/go-llmflow/.github/blob/main/LICENSE)
[![Pure Go](https://img.shields.io/badge/pure%20Go-CGO%3D0-00ADD8?logo=go&logoColor=white)](https://github.com/go-llmflow/gateway)

**An OpenAI-compatible gateway, in pure Go, for GPUs that several institutions
paid for together.**

It sits in front of [vLLM](https://docs.vllm.ai) where LiteLLM would, with
OIDC single sign-on, and adds what a shared machine needs and no open-source
gateway provides:

- **Fair share with reservation and borrowing.** Each institution is
  guaranteed its share of a pool, at least in proportion to what it paid;
  capacity it does not use is lent to the others, and taken back through
  vLLM's priority scheduler when it returns. Shares can be set per pool, so
  a machine bought by a different group of institutions keeps its own
  apportionment.
- **Hard budgets that are never exceeded.** Before a request is admitted, a
  worst-case hold is placed in a PostgreSQL escrow, then settled against the
  actual usage.
- **Soft quotas that warn first.** Like the five-hour session and weekly
  limits of Anthropic's plans: a warning past 75 % and 95 % of each window
  (configurable), before the hard limit refuses.
- **Reporting for decision-makers.** Usage, cost and adoption per
  institution, team and model, exported as JSON, CSV, `sreport`-style text or
  PDF. Small cells are suppressed (k = 5) with secondary suppression across
  linked tables, so no one can be identified by subtracting one total from
  another.
- **External providers under the same guarantees.** Anthropic and Amazon
  Bedrock (Claude through InvokeModel, the other vendors through Converse),
  each opt-in, under the hard budgets.

## What is here

| Repo | Latest | |
|---|---|---|
| [`gateway`](https://github.com/go-llmflow/gateway) | v0.25.2 | The gateway itself: `llmflow-gateway serve`, `check-config`, `export`; the admin web UI (no JavaScript); the MCP gateway; Prometheus metrics. |
| [`aws`](https://github.com/go-llmflow/aws) | v0.1.2 | AWS SigV4 signing and event-stream decoding, standard library only. Checked against the AWS test-suite vectors, fuzzed, and measured faster than aws-sdk-go-v2 on every operation. |
| [`pii`](https://github.com/go-llmflow/pii) | v0.2.1 | PII recognisers in pure Go: checksum-validated IBAN, French NIR, payment cards, phone numbers, e-mail and IP addresses. |
| [`bench`](https://github.com/go-llmflow/bench) | v0.9.1 | The measuring tools: an interleaved, paired A/B overhead bench (`gwbench`), a recording fake vLLM, and end-to-end checks against a real vLLM. |
| [`docs`](https://github.com/go-llmflow/docs) | | The documentation site (Hugo + Hextra). |
| [`go-llmflow.github.io`](https://github.com/go-llmflow/go-llmflow.github.io) | | This organisation's landing page. |

## What it costs per request

Measured the same way for both (the end-to-end difference against a direct
call, interleaved rounds, the same instant upstream), llmflow v0.1.0 added
**117 µs** at p50, where LiteLLM 1.106.2 added **4.86 ms**. Measured with
`gwbench` from v0.6.0 to v0.24.0, the added p50 went from 57.4 µs to 62.7 µs
and has been flat since v0.17.1. Each figure, with its harness and machine
load, is in the
[gateway README](https://github.com/go-llmflow/gateway#readme).

## Links

- Landing: <https://go-llmflow.github.io/>
- Documentation: <https://go-llmflow.github.io/docs/latest/>
