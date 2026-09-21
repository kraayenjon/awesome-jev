<p align="center">
  <img src="https://raw.githubusercontent.com/kraayenjon/awesome-jev/main/assets/cover.jpeg" alt="Awesome Jev — typed decisions for software" width="100%">
</p>

<h1 align="center">Awesome Jev</h1>

<p align="center">
  <a href="https://awesome.re"><img src="https://awesome.re/badge.svg" alt="Awesome"></a>
  <a href="https://madewithjev.com"><img src="https://img.shields.io/badge/directory-madewithjev.com-0d9488" alt="madewithjev.com"></a>
  <a href="https://x.com/kraayenjon"><img src="https://img.shields.io/badge/follow-@kraayenjon-000000?logo=x" alt="Follow on X"></a>
</p>

<p align="center">
  <b>A curated list of Jev use cases, projects, SDKs, tools, and learning resources.</b><br>
  <a href="https://typesafe.ai/blog/introducing-system-one-models-and-jev">Jev</a> is the first System One model from <a href="https://typesafe.ai">TypeSafe AI</a> — an AI model that returns typed decisions (Choice, Score, Noul) with calibrated probabilities instead of generated text.
</p>

> **Looking for real-world Jev use cases with numbers?** [madewithjev.com](https://madewithjev.com) is a directory of what people are building with Jev — every build with the cost, latency, and source the author reported. [Submit yours →](https://madewithjev.com/submit)

Jev launched in early access on **September 15, 2026**. This list is unofficial and not affiliated with TypeSafe AI. Pull requests are welcome — the ecosystem is days old and growing fast.

## Contents

- [What is Jev?](#what-is-jev)
- [Jev vs LLM](#jev-vs-llm)
- [Pricing, limits, and access](#pricing-limits-and-access)
- [Quick start](#quick-start)
- [Official resources](#official-resources)
- [Community](#community)
- [Featured builds with real numbers](#featured-builds-with-real-numbers)
- [SDKs and clients](#sdks-and-clients)
- [Applications](#applications)
  - [Browser and computer-use agents](#browser-and-computer-use-agents)
  - [Search, retrieval, and data](#search-retrieval-and-data)
  - [Developer tools and code review](#developer-tools-and-code-review)
  - [Model routing](#model-routing)
  - [Business and vertical apps](#business-and-vertical-apps)
  - [Robotics and hardware](#robotics-and-hardware)
- [Demos and games](#demos-and-games)
- [Agent tools and MCP servers](#agent-tools-and-mcp-servers)
- [Use cases by industry](#use-cases-by-industry)
- [Patterns](#patterns)
- [Cookbooks](#cookbooks)
- [Benchmarks and evaluations](#benchmarks-and-evaluations)
- [Research and open models](#research-and-open-models)
- [Articles and coverage](#articles-and-coverage)
- [Discussions](#discussions)
- [FAQ](#faq)
- [Related lists](#related-lists)
- [Contribute](#contribute)

## What is Jev?

Large language models generate text. Jev does not. It evaluates typed *questions* against a *state* and returns values your code can branch on, sort by, and route with — plus calibrated probabilities and confidence. TypeSafe AI calls this model class a **System One model**: fast, structured decisions that software can use directly, trained with **RLCD (Reinforcement Learning for Calibrated Decisions)**.

```
text or JSON state + typed questions → constrained answers + probabilities → your code
```

Jev exposes three question types. Questions in one request run in parallel against the same state.

| Question | Goal | Returns |
|---|---|---|
| [Choice](https://docs.typesafe.ai/primitives/choice) | Pick one option from a list | `choice`, `probabilities`, `confidence` |
| [Score](https://docs.typesafe.ai/primitives/score) | Rate the state on a rubric | `score`, `probabilities`, `confidence` |
| [Noul](https://docs.typesafe.ai/primitives/noul) | Is this statement true? | `noul` (0–1) |

Use it to classify, route, score, detect, rank, extract, verify, and gate automation — anywhere you would otherwise write a brittle regex or pay an LLM to return JSON you then have to parse. Questions describe judgments; your code owns composition, thresholds, and side effects.

## Jev vs LLM

From TypeSafe's [launch post](https://typesafe.ai/blog/introducing-system-one-models-and-jev):

| | Existing LLMs | System One + Jev |
|---|---|---|
| Optimized with | RLHF / RLVR | RLCD (Reinforcement Learning for Calibrated Decisions) |
| Optimizes for | Human preference; verifiable rewards | Calibrated decisions with honest probabilities |
| Output | Strings that need parsing and validation | Type-safe structured values, defined in advance |
| Sampling | Sequential, token by token | Parallel, all outputs in a single query |
| Cost | $0.20–$10 / MTok input, output ~5x more | $0.042 / MTok input, output free |
| Speed (vendor-reported) | 3–329 s end-to-end for frontier models | 70–500 ms end-to-end |
| Confidence | Tends to be overconfident if asked | Calibrated confidence on every answer |
| Best at | Chat, writing, code, open-ended reasoning | Decisions inside software: classify, route, score, verify |

Jev is not a replacement for an LLM. When you need free-form text, pair them: let Jev route, retrieve, verify, or guard the call, then let the LLM write inside the boundaries your code enforces.

## Pricing, limits, and access

Snapshot reviewed September 18, 2026. Check [Models](https://docs.typesafe.ai/models) for current values — limits can change dynamically.

| Item | Current detail |
|---|---|
| Model alias | `jev-latest` (current version: `jev-1.13.0`) |
| Endpoint | `POST https://api.typesafe.ai/v1/systemone` |
| Price | $0.042 / 1M input tokens; output tokens free |
| Listed limits | 250,000 tokens/second, 1,200 requests/minute |
| Choice cardinality | Up to 255 options per Choice question |
| Modalities | Text only — no images, audio, or video |
| Direct access | Early access via waitlist at [typesafe.ai](https://typesafe.ai) |
| No-waitlist access | [Vercel AI Gateway](https://vercel.com/ai-gateway/models/jev) (`typesafe-ai/jev`) and [Cloudflare Workers AI](https://developers.cloudflare.com/ai/models/typesafe/jev/) (`typesafe/jev`) |

## Quick start

Get an API key from the [TypeSafe console](https://console.typesafe.ai/settings/keys), then:

**Python** — `pip install typesafe-sdk`

```python
from typesafe_sdk import Choice, Noul, Score, TypeSafeClient

state = {"ticket": "I was charged twice and need the duplicate refunded today."}

with TypeSafeClient() as client:  # reads TYPESAFE_API_KEY from the environment
    response = client.system_one(
        state=state,
        questions={
            "intent": Choice(
                instructions="What is the customer's main request?",
                criteria={
                    "refund": "The customer wants money returned.",
                    "technical_help": "The customer needs a bug or integration fixed.",
                    "information": "The customer is asking for information only.",
                    "other": "None of the other options clearly fits.",
                },
            ),
            "is_urgent": Noul(instructions="Does the ticket explicitly communicate time pressure?"),
            "frustration": Score(
                instructions="How frustrated does the customer appear?",
                criteria=["Calm and neutral", "Concerned but civil", "Very angry or using strong language"],
            ),
        },
    )

print(response.answers["intent"].choice)        # "refund"
print(response.answers["is_urgent"].noul)       # 0.0–1.0
print(response.answers["frustration"].score)    # probability-weighted rubric position
```

**JavaScript / TypeScript** — `npm install @typesafe-ai/sdk`

```ts
import { choice, noul, score, TypeSafeClient } from "@typesafe-ai/sdk";

const client = new TypeSafeClient();
const result = await client.systemOne({
  state: { ticket: "I was charged twice and need the duplicate refunded today." },
  questions: {
    intent: choice("What is the customer's main request?", {
      refund: "The customer wants money returned.",
      technical_help: "The customer needs a bug or integration fixed.",
      information: "The customer is asking for information only.",
      other: "None of the other options clearly fits.",
    }),
    isUrgent: noul("Does the ticket explicitly communicate time pressure?"),
  },
});
```

On Vercel AI Gateway, use `experimental_evaluate` from the AI SDK with the model id `typesafe-ai/jev`. See the [official quick start](https://docs.typesafe.ai/introduction/quickstart) for details.

## Official resources

- [TypeSafe AI](https://typesafe.ai) - Company homepage, waitlist, and product overview.
- [Introducing System One Models and Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) - Launch post by founder Diogo Almeida: architecture, RLCD, pricing, Doom and Wikiracing demos, FAQ.
- [Documentation](https://docs.typesafe.ai/introduction) - Introduction, primitives, patterns, API, and SDKs. Start with the [quick start](https://docs.typesafe.ai/introduction/quickstart).
- [Playground](https://console.typesafe.ai/playground) - Paste a state, add questions, see typed answers in the browser.
- [API keys](https://console.typesafe.ai/settings/keys) - Dashboard for TypeSafe API keys (`TYPESAFE_API_KEY`).
- [HTTP API reference](https://docs.typesafe.ai/api) - `POST https://api.typesafe.ai/v1/systemone`.
- [Models, prices, and limits](https://docs.typesafe.ai/models) - Aliases, versions, and rate limits.
- [Workflow evals](https://evals.typesafe.ai) - Published eval methodology and per-model results on automation workflows.
- [GitHub org](https://github.com/typesafe-ai) - Official open-source repositories.
- [Agent skill](https://docs.typesafe.ai/agent-skill) - Drop-in skill for Claude Code, Codex, and other coding agents ([typesafe-ai/skills](https://github.com/typesafe-ai/skills)).
- [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13) - Known failure modes of the current public model.
- [Manifesto](https://typesafe.ai/manifesto) - The case for machine-native intelligence built for software, not conversation.
- [The Bitterest Lesson](https://typesafe.ai/blog/bitterest-lesson) - Why optimizing the wrong task can dominate gains from scale.
- [AI: too good to be true, too bad to be useful](https://typesafe.ai/blog/ai-too-good-to-be-true-too-bad-to-be-useful-typesafe-ai) - Against preference-optimized chat models for automation.
- [Jev on Vercel AI Gateway](https://vercel.com/ai-gateway/models/jev) - Hosted `typesafe-ai/jev` for the AI SDK's `experimental_evaluate`, no TypeSafe waitlist required.
- [Jev on Cloudflare Workers AI](https://developers.cloudflare.com/ai/models/typesafe/jev/) - `typesafe/jev` via `env.AI.run`, with worked support-routing and risk-escalation examples.

## Community

- [Discord](https://discord.gg/typesafe) - Official TypeSafe server; builder demos live in Show and Tell.
- [X @typesafeai](https://x.com/typesafeai) - Product and research updates.
- [X @CompleteSkeptic](https://x.com/CompleteSkeptic) - Founder Diogo Almeida.
- [LinkedIn](https://www.linkedin.com/company/typesafe-ai/) - Company announcements and hiring.

## Featured builds with real numbers

Production-shaped uses with the cost and latency their authors reported. Each links to a full breakdown on [madewithjev.com](https://madewithjev.com), the Jev use-case directory that maintains this list.

| Build | What it does | Reported numbers | Source |
|---|---|---|---|
| [Jev plays Doom](https://madewithjev.com/builds/jev-plays-doom) | Game loop asking Jev what to do ~10 times a second | ~10 queries/s, ~$7/hour | [X](https://x.com/CompleteSkeptic/status/2099925687465570372) |
| [jev-ultrafast](https://madewithjev.com/builds/jev-ultrafast) | Browser Use's agent with the next-action decision moved to Jev | ~2.9k stars | [GitHub](https://github.com/browser-use/jev-ultrafast) |
| [Flight search with Browser Use](https://madewithjev.com/builds/browser-use-flights) | Booking flow driven end to end | ~7 s, ~$0.004 | [X](https://x.com/gregpr07/status/2100411066966749359) |
| [Stagehand on a remote browser](https://madewithjev.com/builds/stagehand-remote-browser) | Browser tasks at a tenth of a cent each | ~$0.001/task | [X](https://x.com/kylejeong/status/2100622054945095934) |
| [jev-trader](https://madewithjev.com/builds/jev-trader) | Buy/sell decided inside a 300 ms Monad block, on Kuru's order book | 300 ms decision window | [GitHub](https://github.com/jarrodwatts/jev-trader) |
| [Triage across 1,500 emails](https://madewithjev.com/builds/inbox-triage-1500-emails) | A full inbox sorted in one pass | ~1,500 emails | [X](https://x.com/ryanvogel/status/2100042788851101842) |
| [Every's editorial vibe check](https://madewithjev.com/builds/every-editorial-judgments) | 37 documents, 21 questions each; 6 of 7 planted defects caught | 1,709 judgments, <$0.01, 0.35 s median | [Every](https://every.to/also-true-for-humans/mini-vibe-check-typesafe-s-jev-judged-everything-i-ve-written-in-0-7-seconds) |
| [1kpapers](https://madewithjev.com/builds/1kpapers) | A corpus classified by topic and published as a site | 1,018 papers | [Site](https://www.1kpapers.com/) |
| [Jev plays chess](https://madewithjev.com/builds/jev-plays-chess) | Legal moves as a Choice, compared with reasoning models | illegal moves impossible by construction | [dev.to](https://dev.to/maximsaplin/typesafe-jev-played-chess-and-landed-next-to-reasoning-models-28ga) |
| [3,282 posts, eight questions each](https://madewithjev.com/builds/x-post-analysis) | Ian Nuttall's X back catalogue scored for what travels | 4.25M tokens, $0.1282, 8 m 34 s | [X](https://x.com/iannuttall/status/2100668908227162567) |
| [Post scoring with SuperX](https://madewithjev.com/builds/superx-post-scoring) | 61 questions about a draft before it ships | ~1 s, $0.0004/draft | [X](https://x.com/robj3d3/status/2100722975645598191) |
| [724 competitor ads, broken down](https://madewithjev.com/builds/competitor-ad-teardown) | Hook, format, offer, CTA per ad across 37 brands | ~40 s, ~$0.09 | [X](https://x.com/TheMattBerman/status/2100654891756589230) |
| [typesafe-computer-use](https://madewithjev.com/builds/typesafe-computer-use) | macOS computer use, one typed decision per step | ~$0.0002/step | [GitHub](https://github.com/awlevin/typesafe-computer-use) |
| [jev-drone](https://madewithjev.com/builds/jev-drone) | Tactical judgment loop flying on hardware | control at 2.5 Hz | [GitHub](https://github.com/RomanSlack/jev-drone) |
| [Wikiracing](https://madewithjev.com/builds/wikiracing) | Pick one link out of thousands until you arrive | 255-option Choice ceiling | [TypeSafe](https://typesafe.ai/blog/introducing-system-one-models-and-jev) |

All figures are as reported by each author, not measured by this list. → [Browse the full directory at madewithjev.com](https://madewithjev.com)

## SDKs and clients

Official first, then community clients. Community packages are not affiliated with TypeSafe.

**Official**

- [Python SDK](https://github.com/typesafe-ai/typesafe-sdk-python) - `pip install typesafe-sdk`. [Docs](https://docs.typesafe.ai/sdk/python).
- [JavaScript / TypeScript SDK](https://github.com/typesafe-ai/typesafe-sdk-js) - `npm install @typesafe-ai/sdk`. [Docs](https://docs.typesafe.ai/sdk/javascript).
- [System One adapter (Python)](https://github.com/typesafe-ai/system-one-adapter-python) - Drop-in `TypeSafeClient` replacement backed by LLM APIs, to compare Jev against chat models on the same questions. `pip install system-one-adapter`.
- [Vercel AI SDK provider](https://ai-sdk.dev/providers/ai-sdk-providers/typesafe-ai) - `@ai-sdk/typesafe-ai` with `experimental_evaluate`; use `typeSafeAi.evaluationModel('jev-latest')` or the Gateway id `typesafe-ai/jev`.

**Community, by language**

- Go: [jev-go](https://github.com/Gaurav-Gosain/jev-go) - `go get github.com/Gaurav-Gosain/jev-go`. Also [Stumble/jev-go](https://github.com/Stumble/jev-go) - dependency-free, works against TypeSafe direct and Vercel AI Gateway, with an interactive CLI and an installable agent skill.
- Elixir: [typesafe_sdk](https://github.com/nshkrdotcom/typesafe_sdk) - Hex package for `system_one` and model listing. Also [Jev (OTP)](https://github.com/dannote/jev) - Jev as a peer GenServer; answers arrive as messages you pattern-match, with network-free tests.
- Ruby: [typesafe-sdk](https://github.com/joshmn/typesafe-sdk) - Ruby 3.1+, retries, thread-safe pooled HTTP. Also [RubyLLM TypeSafe](https://github.com/kieranklaassen/ruby_llm-typesafe) - TypeSafe provider for RubyLLM 2. And [typesafe-ai-rails](https://github.com/GenieRobot/typesafe-ai-rails) - Rails integration with usage telemetry and opt-in confidence policies.
- Rust: [typesafe-ai-rs](https://github.com/gilljon/typesafe-ai-rs) - async and blocking client. Also [Twister915/typesafe-ai](https://github.com/Twister915/typesafe-ai) - observable retries; [typesafe-rs](https://github.com/AbdelStark/typesafe-rs) - latency-focused transport; [s1-rs](https://github.com/AbdelStark/s1-rs) - derive layer for Choice / Score / Noul with confidence gates and network-free tests.
- PHP / Laravel: [typesafe-sdk-php](https://github.com/Butochnikov/typesafe-sdk-php) - typed DTOs and promises. Plus [laravel-typesafe-jev](https://github.com/Butochnikov/laravel-typesafe-jev) - Laravel 12/13 config, facade, scoped DI, and a recording fake.
- Python: [jevclient](https://github.com/AboveColin/jevclient) - async client (`pip install jevclient`), separate from the official SDK.
- Swift: [swift-typesafe](https://github.com/ainame/swift-typesafe) - Swift 6.4 client aligned with the Python SDK 0.6.0 API, including Linux.
- Scala / ZIO: [zio-typesafe-ai](https://github.com/jamesward/zio-typesafe-ai) - ZIO client with a small DSL for noul / choice / score.
- .NET: [typesafe-dotnet-sdk](https://github.com/saibimajdi/typesafe-dotnet-sdk) - typed questions and confidence-scored answers.
- TypeScript: [Advocaat](https://github.com/pithings/advocaat) - small client with tagged helpers for chances, choices, and scores.
- Cloud: [typesafe-on-neon](https://github.com/andrelandgraf/typesafe-on-neon) - Neon Function proxy for the Neon AI Gateway.

## Applications

Open-source projects that put Jev in a real loop. Grouped by what Jev decides.

### Browser and computer-use agents

- [Jev Ultrafast](https://github.com/browser-use/jev-ultrafast) - Browser agent from [Browser Use](https://github.com/browser-use). Jev picks an operation and a DOM element in one request; a small LLM writes text only for `TYPE_TEXT`. Zürich → London on Google Flights in ~7 s. Library, local inspector, and measurements included.
- [Jev for Chrome](https://github.com/chy4pro/jev-for-chrome) - Unofficial Chrome extension (Manifest V3) port of Jev Ultrafast: Jev picks the operation and DOM element in one request, a small text model writes typed values, and it runs in the user's own tabs through OpenRouter, TypeSafe or Cloudflare; includes a 17-task headless-Chromium suite with recorded traces.
- [jev-ego](https://github.com/romaluev/jev-ego) - Browser agent on ego lite: one TypeSafe request picks operation + indexed element; agent-facing observe/act/suggest/step CLI.
- [jev-browser](https://github.com/Ying-Kai-Liao/jev-browser) - An LLM plans the outcome, Jev decides each click/type on a Playwright snapshot (~300 ms/call). Ships as a library, CLI, and MCP server.
- [Jev Browser (Vlad Terin)](https://github.com/vlad-terin/jev-browser) - Agent skill + runtime: Codex plans, Jev selects elements, a runner acts and verifies each step.
- [typesafe-computer-use](https://github.com/awlevin/typesafe-computer-use) - macOS computer-use loop: OCR the screen, Jev classifies the next action, then click. About $0.0002/step.
- [Mobile Jev](https://github.com/droidrun/mobile-jev) - Android agent on Mobilerun: Jev decides each tap. Opens Uber, SFO → Golden Gate, payment screen in ~21 s / 9 actions. No ADB.
- [Unclutter](https://github.com/kitze/unclutter) - Chrome / Firefox extension: Jev classifies nonessential page elements; local template rules hide them on later visits.
- [TypeSafe AdBlock](https://github.com/realZachi/typesafe-adblock) - Chrome extension: Jev judges whether a DOM element is an ad and removes it. BYOK, no backend; a demo, not a real ad blocker.
- [jev-skip](https://github.com/valentynkit/jev-skip) - Browser extension that reads a YouTube video's caption track and paints a per-segment sponsor probability on the seek bar before the intro ends, with no crowdsourced database.

→ [More agents and browsers on madewithjev.com](https://madewithjev.com/categories/agents-and-browsers)

### Search, retrieval, and data

- [Every](https://github.com/sufianetaouil/every) - Semantic code-search CLI: a yes/no question against every function, ranked by Noul probability.
- [blink](https://github.com/ellipsis-dev/blink) - Codebase search: an ensemble of walkers asks Jev which file answers a natural-language query.
- [Jev Search](https://github.com/superagents-lab/jev-search) - Web search app using Choice and Noul judgments to select sources, time ranges, and query candidates, then rank results retrieved through Search1API. Live demo: [jev.s1.dev](https://jev.s1.dev).
- [neo4jev](https://github.com/jexp/neo4jev) - Neo4j graph navigation: at each node Jev chooses which relationship to follow, with beam search over log-probabilities.
- [jev-bfs](https://github.com/komikat/jev-bfs) - Finds link paths between Wikipedia articles; Jev ranks each page's outgoing links while Python controls the search.
- [hono-jev-router](https://github.com/yusukebe/hono-jev-router) - Experimental Hono router: Jev matches an incoming request to a plain-language route description.
- [sqlite3-jev](https://github.com/mattn/sqlite3-jev) - SQLite C extension: `jev_noul` / `jev_choice` / `jev_score` as SQL functions via libcurl.
- [jev-curate](https://github.com/AkashPriyadarshii/jev-curate) - High-throughput synthetic dataset sifter in Rust: Noul checks on JSONL and Parquet rows, streaming clean and rejected rows to disk.
- [1kpapers](https://www.1kpapers.com/) - 1,018 papers classified by topic and published as a browsable site.
- [jev.nvim](https://github.com/valentynkit/jev.nvim) - Neovim plugin that splits the current buffer into functions with Treesitter, asks Jev a plain-language question against each one, and lists the answers in the quickfix window ranked by probability.

→ [More research and data builds on madewithjev.com](https://madewithjev.com/categories/research-and-data)

### Developer tools and code review

- [Jev Review](https://github.com/devagrawal09/jev-review) - Staged code-review workflow and local dashboard driven by focused Jev calls.
- [Foreman](https://github.com/thruwire/foreman) - Software-factory loop: Codex implements; Jev independently judges completeness, tests, and whether a human is needed.
- [Clean Code Judge](https://github.com/frostney/clean-code-review) - Scores every PR file on 31 boolean Clean Code smells plus function size and nesting, then hands verdicts to a writing model for prose.
- [OpenWork](https://github.com/different-ai/openwork) - Wires Jev into its eval testkit as a verification judge so agent-produced work is gated by typed verdicts.
- [jev-shell-history](https://github.com/mrnugget/jev-shell-history) - Fish-style zsh autosuggestions: Jev ranks recent history as you type.
- [jev-secret-detection](https://github.com/teyhouse/jev-secret-detection) - Secret-in-diff detector with repeatable Jev verdicts.
- [commit-miner](https://github.com/devanshbatham/commit-miner) - Rust CLI that classifies commit diffs: bug fixes, security/CWEs, and change types. HTML/CSV reports.
- [Jev Logs](https://github.com/reachjalil/jevlogs) - OpenTelemetry log triage: Jev scores diagnostic value and priority before an expensive LLM looks at the archive.
- [typeful-triage](https://github.com/cephalization/jev-triage) - Multiplayer issue-triage dashboard: fixed typed questions per issue (kind, severity, urgency, duplicate, next step), with human corrections shown back to the model on later runs.
- [jev-resilience](https://github.com/Vicente-MD/jev-resilience) - Spring WebFlux starter: a semantic circuit breaker that uses Jev to catch silent HTTP 200 failures.
- [tripwire](https://github.com/noelzappy/tripwire) - AI SDK middleware and OpenAI-compatible proxy: seven Jev checks on every LLM response in ~100 ms, confidence-gated.
- [ProgressGate](https://github.com/AshutoshVJTI/progressgate) - Detects semantic stagnation in agent loops: Jev judges the trajectory; code returns CONTINUE / WARN / REPLAN / HALT.
- [jev-harness](https://github.com/AntonioCoppe/jev-harness) - Production layer around Jev: policy, confidence gate, shadow mode, recipes, and an eval CLI.
- [jev-tree](https://github.com/reachjalil/jev-tree) - Recursive Choice over a taxonomy so catalogs larger than Jev's 255-option cap still fit.
- [Notra](https://github.com/usenotra/notra) - Marketing analytics: its `NOTRA_JEV_CLASSIFIERS` flag routes brand-visibility classifiers off an LLM and onto Jev boolean decisions at a 0.5 threshold, targeting 300 ms p50.
- [jev-eval-agent](https://github.com/vinilana/jev-eval-agent) - Public eval harness for early Jev tests.
- [Supercov](https://github.com/supercorp-ai/supercov) - Code quality and test coverage for coding agents: Jev scores each source file so the agent knows what to fix first.
- [jev-commit](https://github.com/valentynkit/jev-commit) - Pre-commit hook where one Jev call judges whether the commit message matches the staged diff, checks for debug leftovers and unmentioned work, and blocks the commit only when it detects a credential.

### Model routing

- [jev-router](https://github.com/gargpratyush/jev-router) - Per-turn routing for Claude Code and Codex: simple work to the fast tier, hard work to the strong tier. `npm i -g jev-router`.
- [jev-codex-router](https://github.com/0xNatoshi/jev-codex-router) - Per-turn Codex routing: Jev picks model, thinking depth, and speed mode.
- [jev-router (prismhq)](https://github.com/prismhq/jev-router) - Open-source LiteLLM-based router where a Jev decision picks which model serves each request.
- [pi-jev-router](https://github.com/mejiasd3v/pi-jev-router) - Automatic per-request model routing for the Pi coding agent through Jev decisions on Vercel AI Gateway.
- [jcm-router](https://github.com/adarshmishra07/jcm-router) - Local proxy that picks the Claude model and reasoning effort per message while leaving the cached main chat untouched.
- [jev-agent-skill-router](https://github.com/GodsBoy/jev-agent-skill-router) - Routes agent skill selection through typed, confidence-aware decisions so weak matches are declined instead of guessed.
- [Jevonian](https://github.com/xinyao27/jevonian) - Local OpenAI, Anthropic, and Responses-compatible proxy where one Jev call answers both the model route and the thinking level for the virtual model jevonian/auto, from session state, quota health, candidate capabilities, and cache-switch penalties; code filters candidates by wire protocol, context window, effort floor, and spent quota windows before Jev is asked, and a pinned model ID or explicit jevonian route skips Jev entirely.

### Business and vertical apps

- [typesafe-jev CV screener](https://github.com/gtaras7/typesafe-jev) - Screens a folder of CVs against an editable policy; re-scoring candidates is free when the policy changes.
- [Jev email intent workflow](https://github.com/GiesN/typesafe-jev-workflow) - Async LangGraph workflow: a typed Choice (`invoice` or `general`) routes each inbound email to the matching handler.
- [HA-Jev](https://github.com/AboveColin/HA-Jev) - Home Assistant integration: typed questions about entity state become sensors and automation actions, with usage, cost, and daily-budget entities.
- [Jev Trader](https://github.com/jarrodwatts/jev-trader) - One buy/sell decision per Monad block on Kuru's MON-USDC book. Live demo: [jev-trader.vercel.app](https://jev-trader.vercel.app/).
- [Human Compiler](https://github.com/asfarsadewa/human-compiler) - Paste corporate prose; Jev scores passive-aggression, urgency, and information density, then code emits rustc-style diagnostics. Live: [human-compiler.asfarlab.fun](https://human-compiler.asfarlab.fun).
- [JEVMETER](https://github.com/ChetasLua/jevmeter) - Live Jev meter on any video: every sentence scored, rendered as a 16:9 edit.
- [jev-audio-beeper](https://github.com/santos-sanz/jev-audio-beeper) - Low-latency audio insult detector: Jev decides, ffmpeg beeps in ~466 ms without rewriting the rest of the track.
- [Jev Moderation Bot](https://github.com/brainstormity/Jev-Moderation-Bot) - Discord bot scoring incoming messages for phishing, spam, and social engineering, with a four-stage escalation ladder.
- [citation-verifier](https://github.com/MarissaFamularo/citation-verifier) - Checks whether each cited paper actually supports the sentence citing it: Claude locates the quote, Jev scores the support, a human makes the final call.
- [LegalForecast-MTD](https://github.com/johnhughes3/LegalForecastBench) - Benchmark that asks Jev to predict federal motion-to-dismiss rulings, scored with claim-defendant micro-Brier metrics.
- [Smart home assistant demo](https://docs.typesafe.ai/demos/smart-home) - Official interactive demo of speculative fan-out: many questions in one call, code keeps the relevant answers, LLM only for splits and chit-chat.
- [SmartMoney-Cub](https://github.com/myc0576/Smartmoney-Cub) - Experimental, read-only trading journal that passes filings, event wires and central-bank statements to Jev for typed Choice, Noul and Score answers on evidence and policy stance, keeping a human in the promote/reject loop and never placing an order.
- [Jev Web Analyzer](https://github.com/replynodes/jev-web-analyzer) - Product evaluation: evaluates a public SaaS landing page as clean Markdown with ten bounded Jev `Choice` questions about first-visit understanding, leaving validation, policy, and presentation in application code.

### Robotics and hardware

- [Jev Drone](https://github.com/RomanSlack/jev-drone) - MuJoCo quadrotor: control and safety stay in code; Jev handles slower tactical judgments at 2.5 Hz.
- [jev-askable-arm](https://github.com/TarunTomar122/jev-askable-arm) - Zero-shot English goals on a simulated Franka arm; Jev chains hardcoded primitives.
- [robo-harness](https://github.com/grmkris/robo-harness) - SO-101 arm workbench: a Jev decision runner picks bounded joint steps from typed candidate actions under a spend budget.

→ [More robotics and devices on madewithjev.com](https://madewithjev.com/categories/robotics-and-devices)

## Demos and games

Toys, live sites, and realtime agents. Most shipped in the first days after launch.

- [Yes / No](https://yesno.coderai.dev) - Free no-signup Noul demo. Ask a question, get yes / no / maybe, with web search when needed.
- [Jev Tetris](https://jev-omega.vercel.app) - Jev picks rotation and column from holes, stack height, and bumpiness.
- [Jev Pac-Man](https://jev-pacman.ephraimduncan.com) - Maze as JSON; Jev picks the turn at each junction in realtime.
- [typesafe-mario](https://github.com/fhshaik/typesafe-mario) - Super Mario Bros. from structured emulator state.
- [jev-doom-agent](https://github.com/lukaske/jev-doom-agent) - Browser-native Doom with Chocolate Doom WASM, spatial state, and live decision telemetry.
- [jev-gomoku](https://github.com/mizchi/jev-gomoku) - MoonBit client plus Jev-vs-Jev gomoku.
- [jev-t-rex-runner](https://github.com/joshlarsen/jev-t-rex-runner) - The Chrome dinosaur game, played by Jev.
- [snake-jev](https://github.com/siroccomask/snake-jev) - Snake: hundreds of typed direction decisions per run. Also [typesafe-snake](https://github.com/sorrycc/typesafe-snake).
- [Jev Plays StarCraft](https://github.com/phyous/tsai-sc) - Structured-state harness for the original StarCraft shareware campaign, with verified run and probability traces.
- [Jev × Civilization II](https://github.com/phyous/tsai-civ2) - Original Civ II in a browser; Jev chooses empire, city, research, and unit actions. Experimental; no verified win yet.
- [Jev Guard](https://guard-jev.vercel.app) - Comment-moderation playground.
- [Hollow Creek](https://hollow-creek-sigma.vercel.app) - Village NPCs that *judge* you each tick instead of chatting.
- [Jev mood demo](https://jev-demo.vercel.app) - Talk nicely or nastily over time; structured state tracks mood.
- [Jev Room](https://jev-room.moe136231.chatgpt.site) - One sentence → six room settings. Jev chooses, the app renders.
- [TypeSafe Typewriter](https://typesafe-demo.val.run/) - Live Val Town demo: 16 typed judgments update as you type.
- [got-jev](https://github.com/phureewat29/got-jev) - Game of Thrones roleplay: a story model writes the scene; Jev answers where Jon Snow is, how much danger, and what should play under it.
- [Little Airways](https://github.com/lbotinelly/jev-little-airways) - Toy archipelago air-traffic control: divert / emergency / who lands first, ~150 ms.
- [jev-plays-pokemon-red](https://github.com/valentynkit/jev-plays-pokemon-red) - Pokemon Red on PyBoy where code owns the route and the arithmetic, Jev picks only at branches, and every battle turn logs a faint prediction scored by Brier against what the RAM says.

→ [More games and real-time builds on madewithjev.com](https://madewithjev.com/categories/games-and-real-time)

## Agent tools and MCP servers

Tools that expose Jev to coding agents and MCP clients.

- [TypeSafe agent skill](https://github.com/typesafe-ai/skills) - Official skill: primitives, patterns, and how to structure evaluations. Claude Code: `claude plugin marketplace add typesafe-ai/skills` then `claude plugin install typesafe@typesafe-ai`. Other agents: `npx skills add typesafe-ai/skills --skill typesafe-ai`.
- [eve](https://github.com/vercel/eve) - Vercel's agent framework. Experimental `autoModel` defaults to Gateway `typesafe-ai/jev` to pick a language model from an allowlist.
- [AI CLI](https://github.com/vercel-labs/ai-cli) - Vercel Labs CLI that can run Jev as the evaluation model for its `evaluate` command.
- [jev-mcp (jkudish)](https://github.com/jkudish/jev-mcp) - Node MCP wrapping three cookbook patterns: `jev_verify` (citation check), `jev_screen` (prompt-injection guardrails), `jev_find` (semantic ranking without embeddings). `npx -y github:jkudish/jev-mcp`.
- [jev-mcp (blakestone-x)](https://github.com/blakestone-x/jev-mcp) - Python MCP server: classify, score, check, match, and screen tools.
- [Jev Review MCP](https://github.com/NiazMorshed2007/jev-review) - Local-first MCP: Claude Code, Codex, Cursor, and OpenCode get structured quality review from Jev while they write.
- [typesafe-mcp](https://github.com/itsmostafa/typesafe-mcp) - Go CLI and single-binary MCP for Claude Desktop, Claude Code, and Codex.
- [Jevbridge](https://github.com/gamesonrblx/Jevbridge) - ACP/MCP adapter: typed Jev decisions and computer use beside Codex, Claude, Grok, and OpenCode.
- [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) - Claude Code plugin and npm library: Jev scores tool calls and drops stale ones instead of summarizing context.
- [SkillRanker](https://github.com/Dicklesworthstone/skillranker) - Rust CLI: Jev ranks which agent skill fits the next step from live session context, with Claude Code hooks.
- [pi-typesafe](https://github.com/DevMortimer/pi-typesafe) - Pi extension: one consented, key-managed TypeSafe client, batched `typesafe_evaluate`, offline-testable transport.
- [pi-jev](https://github.com/y0usaf/pi-jev) - Pi extension with a shadow-mode tool-call gate, output judge, and typed `jev_ask`.
- [pi-warden](https://github.com/DevMortimer/pi-warden) - Pi guardrails on pi-typesafe: held tool results instead of a dialog; write checks against a project rules file.
- [pi-jev-auto-mode](https://github.com/jomatsu/pi-jev-auto-mode) - Pi auto mode: Jev semantically approves `bash` / `write` / `edit`, and fails closed when it cannot decide.
- [Bicameral](https://github.com/AbdelStark/bicameral) - Pi coding harness: LLM writes, Jev supplies typed reflexes for policy, loop detection, and review. Explicitly not a sandbox.
- [ask-jev-skill](https://github.com/shantanugoel/ask-jev-skill) - Hermes skill: ask Jev whenever the agent needs a bounded decision.
- [jev-system-architect](https://github.com/samtay32/jev-system-architect) - Skill that hunts for brittle semantic logic and turns it into Choice / Score / Noul boundaries.
- [augustus](https://github.com/24601/Augustus) - Design-judgment skill: maps Choice/Score/Noul onto classical methods with a composition algebra, question-design diagnosis, and falsifying validation gates.
- [jev-judgment](https://github.com/HyunjunJeon/jev-judgment) - Agent skill that sends closed coding-agent judgments to Jev so verdicts stay typed, cheap, and comparable across runs.
- [pi-typesafe-jev](https://github.com/legacybridge-tech/pi-typesafe-jev) - Exposes System One judgments as five Pi tools; code and users keep control of thresholds, weights, and actions.
- [limpet](https://github.com/noplan-inc/limpet) - Stop hook that keeps an agent from finishing too early by judging plain-language completion rules with Jev.
- [jev-guard](https://github.com/leepokai/jev-guard) - Prompt-injection and dangerous-action guard for Claude Code, Codex, Pi, and ACP agents.
- [dsh-auto-mode](https://git.allen-software.com/allenh1/dsh-auto-mode) - DeepSeek Harness permission preset whose end-prompt step has Jev answer the open questions an agent leaves in its final message.
- [jev-belay](https://github.com/valentynkit/jev-belay) - Claude Code Stop hook that reads the transcript for evidence of a completed task and, only when files changed with no passing check since, spends one four-question Jev call before allowing the agent to stop, failing open on every error path.

## Use cases by industry

Decision shapes that recur across domains. Each is a small decision system: a state object, atomic questions, and a code-owned review branch. Expanded from TypeSafe's [use-case map](https://docs.typesafe.ai/concepts/use-case-map) and [workflow evals](https://evals.typesafe.ai/).

| Domain | Example Jev workflow |
|---|---|
| Customer support | Classify intent, detect urgency and refund intent, score frustration; route with ordinary code and escalate low-confidence tickets. |
| Security operations | Join an alert with asset context and authorizations; ask whether the activity is unauthorized, then let a deterministic playbook close, queue, notify, or contain. |
| Finance and payments | Match invoices against POs and contracts; Jev flags duplicate/fraud/wrong-vendor signals while code owns totals, dates, and execution. |
| Insurance | Run a claims rubric as independent Nouls — coverage, exclusions, fraud indicators — and map the middle band to human review. |
| Legal and compliance | Find missing clauses, prohibited claims, and policy violations in contracts, filings, and marketing material. |
| Recruiting | Evaluate job-related evidence, match candidates to roles, route applications, escalate uncertain cases. |
| Sales and lead gen | Score ICP fit, buyer relevance, pain points, and purchase intent before routing leads. |
| E-commerce | Normalize listings, extract product attributes, detect counterfeit or prohibited-listing signals, route exceptions. |
| Moderation and trust & safety | Apply org-specific criteria to toxicity, spam, fraud, and personal-data exposure, with an explicit uncertain outcome. |
| Advertising | Check brand safety, audience suitability, regulatory claims, and ad-to-landing-page alignment. |
| Gaming | Moderate chat, score engagement or frustration, detect abuse and churn signals, route player support. |
| Financial crime | Evaluate transaction narratives and KYC material; match entities and prioritize investigator queues. |
| Scientific discovery | Screen papers, label themes in qualitative research, check manuscript citations, link entities to evidence. |
| Risk and forecasting | Turn incident reports and transaction descriptions into probabilistic features for a supervised model. |
| Knowledge graphs | Classify entity types and relationships, detect contradictions, support probabilistic traversal. |

Recurring architectures worth stealing:

- **Support inbox triage** - Fan out intent, urgency, severity, and frustration questions in one call; act on the confident answers, route the rest.
- **RAG passage filtering** - Score relevance, contradiction, and injection risk per passage before the answering model sees it.
- **LLM guardrails** - Screen prompts, replies, and tool calls with hazard Nouls and a harm Score; policy passes, reviews, or blocks.
- **Confidence-gated actions** - Lower thresholds for reversible read-only actions, higher ones for risky operations, humans for the rest.
- **Model routing** - Let a fast typed decision choose between deterministic code, a cheap LLM, a frontier LLM, or a person.
- **Structured extraction cascades** - A small model extracts candidate fields; Jev verifies each value; only failures escalate to a reasoning model.
- **Composite scoring** - Score independent dimensions, then combine with weights you own in code — leads, candidates, vendors, risk.
- **Corpus map-reduce** - Ask the same questions of every document: 1,018 papers, 3,282 posts, 724 ads, 1,500 emails. Read the aggregate, not the documents.
- **Real-time control** - When the deadline is a frame, a block, or a tick, code generates legal actions and Jev picks one.

## Patterns

Architectural recipes from the official docs.

- [Speculative fan-out](https://docs.typesafe.ai/patterns/fan-out) - Ask many questions, including ones that may not apply; filter in code.
- [Confidence-gated routing](https://docs.typesafe.ai/patterns/confidence-routing) - The answer is *what*; confidence is *whether to act*.
- [Composite scoring](https://docs.typesafe.ai/patterns/composite-scoring) - Atomic scores, weights you own in code.
- [Intent routing](https://docs.typesafe.ai/patterns/intent-routing) - Classify, then hand off to logic, a specialist LLM, or a human.

See also: [How to build with System One](https://docs.typesafe.ai/concepts/how-to-build-with-system-one), the [use-case map](https://docs.typesafe.ai/concepts/use-case-map), and [confidence](https://docs.typesafe.ai/confidence).

## Cookbooks

Official, copy-pasteable workflows. Full index: [console cookbooks](https://console.typesafe.ai/docs/cookbooks) and the [docs index](https://docs.typesafe.ai/llms.txt).

- [Parallel questions](https://docs.typesafe.ai/cookbooks/parallel_questions) - Batch many questions over one state; one call instead of N.
- [Line-by-line search](https://docs.typesafe.ai/cookbooks/semantic_find) - Score hundreds of line ids against a query with Choice + a Noul "does an answer exist?" check.
- [Re-ranking](https://docs.typesafe.ai/cookbooks/rerank_typesafe) - BM25 shortlist, then one TypeSafe question per query–candidate pair.
- [Guardrails for LLMs](https://docs.typesafe.ai/cookbooks/llm_guardrails) - Screen messages in and out of an LLM; threshold probabilities in code.
- [Double-checking citations](https://docs.typesafe.ai/cookbooks/citation_check) - Whether a quote's context supports the claim; confidence gates human review.
- [Classifying RAG passages](https://docs.typesafe.ai/cookbooks/classifying_rag_passages) - Keep, flag, or drop retrieved passages before the answering model.
- [Function calling](https://docs.typesafe.ai/cookbooks/function_calling) - Map natural-language requests onto ordinary typed functions with closed-set arguments.
- [Skill suggestion](https://docs.typesafe.ai/cookbooks/skill_suggestion) - Rank an agent skill catalog, then read only the top few.
- [Hierarchical classification](https://docs.typesafe.ai/cookbooks/hierarchical_classification) - Beam search over deep taxonomies with Choice probabilities.
- [SDE cascade](https://docs.typesafe.ai/cookbooks/sde_cascade) - Two-stage structured-data-extraction cascade (mini → verify → reasoning).
- [Date extraction](https://docs.typesafe.ai/cookbooks/date_extraction_cookbook) - Ask for named date parts, resolve and validate in code.
- [Pre-parsed value extraction](https://docs.typesafe.ai/cookbooks/pre_parsed_value_extraction_cookbook) - Regex candidates, then Jev selects the requested span.
- [Knowledge graph entity alignment](https://docs.typesafe.ai/cookbooks/entity_alignment) - Score merge / leave unlinked / send to a curator.
- [Autoresearch feature discovery](https://docs.typesafe.ai/cookbooks/autoresearch_feature_discovery) - Propose TypeSafe questions as numeric features for a supervised model.
- [Classification using confidence](https://docs.typesafe.ai/cookbooks/classification_using_confidence) - Report a fine label only when confidence is high; otherwise climb the hierarchy.
- [Structure recovery](https://docs.typesafe.ai/cookbooks/autoformat) - Reconstruct Markdown from de-formatted plain text.
- [Self-consistency: nouls](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook) / [choices](https://docs.typesafe.ai/cookbooks/consistency_choice_cookbook) - Route uncertain probabilities to review without hiding the raw values.
- [Jev Cookbook](https://github.com/nexibeo/jev-cookbook) - Community cookbook: 15 runnable Node recipes that pass tickets, table rows, documents, invoices and emails as state, ask Choice, Noul and Score questions in one call, and keep thresholds, review bands and actions in code.

## Benchmarks and evaluations

Official numbers are vendor-reported; these community efforts measure for themselves.

- [Workflow evals](https://evals.typesafe.ai) - Official: four automation workflows, accuracy/cost/time per case, Jev vs frontier models.
- [typesafe-ai-benchmark](https://github.com/iammrduncan/typesafe-ai-benchmark) - Jev vs Qwen 3.8 27B on Cerebras for the same System One questions.
- [Jev Rerank Bench](https://github.com/anessbelbati/jev-rerank-bench) - Reranking comparison with raw provider responses, scoring code, and uncertainty intervals.
- [Jev Spam Eval](https://github.com/bitnovus/jev-spam-eval) - Zero-shot spam study vs trained TF-IDF baselines, with post-hoc-tuning caveats.
- [Jev Phishing Bench](https://github.com/anisselbd/jev-phishing-bench) - 2,000 emails: Jev vs Claude Haiku 4.5 on click-or-not, with calibration, latency, and cost. Haiku wins accuracy here.
- [jev-agent-failure-benchmark](https://github.com/TokenTrim/jev-agent-failure-benchmark) - Who&When Pro (injected agent failures): Jev vs a strong LLM on who / which step / error category.
- [jev-sec-bench](https://github.com/Gaurav-Gosain/jev-sec-bench) - Blind prompt-injection and vulnerable-code detection benches on public corpora.
- [Jev DSPy Lab](https://github.com/jmanhype/jev-dspy-lab) - DSPy companion that records and replays TypeSafe calls while measuring calibration, selective risk, abstention, latency, and cost.
- [jevcal](https://github.com/abhixhek/jevcal) - CLI that fits a per-question confidence threshold to a target accuracy on your own labeled data, and fails CI when a Jev update breaks locked thresholds.
- [ASSAY-001](https://github.com/jourdanlabs/assay-001) - Independent pre-registered check of Jev calibration and type safety on Banking77 / CLINC150. Split verdict, full logs. [Write-up](https://donttrustme.ai/assay-001.html).
- [Jev search rerank eval](https://github.com/zhuyansen/jev-search-rerank-eval) - 9,831 labelled pairs: Jev rerank vs BM25 / bge-m3. Fusion wins; Jev alone does not beat embeddings.
- [Smoking-history extraction benchmark](https://github.com/vclic/smoking-extraction-benchmark) - 1,000 synthetic notes: Jev vs OpenAI structured outputs on accuracy, cost, and latency.
- [Jev Playground](https://github.com/hegargarcia/jev-playground) - Benchmarks Jev against Luna, Haiku, and Gemini at choosing validated legal moves in explicit-state games.
- [jev-research-eval](https://github.com/jgridifier/jev-research-eval) - Reproducible eval harness plus field note for Jev Ultrafast research-browser tasks.

## Research and open models

Independent work inspired by Jev's interface. These are not TypeSafe models.

- [jevlike](https://github.com/vinnylarouge/jevlike) - Train a small one-pass scorer mapping context + N text options to a probability per option. Doom / chess vision demos and a Wikispeedia example. Explicitly not a reproduction of TypeSafe's architecture or RLCD.
- [openjev](https://github.com/TheoLeeCJ/openjev) - Can we run something Jev-like on a home RTX 3090? Reads option logits instead of generating text. Also [zhihz/openjev](https://github.com/zhihz/openjev) - an independent local preview answering bilingual probability questions.
- [PocketJev](https://github.com/NullPo-jp/PocketJev) - On-device iPhone visual decisions with MLX + Qwen3-VL option logits. Camera + 3-choice, no text generation, ~1 s, no photo saved.
- [jev-visual](https://github.com/hr98w/jev-visual) - Educational Jev-like visual inference on Apple Silicon: shared multimodal context, candidate scoring, sorting-factory / Breakout / gesture demos.
- [jevmlx](https://github.com/bnsd55/jevmlx) - Jev-style parallel constrained decisions for any MLX model on Apple Silicon: schema-valid JSON in one forward pass.
- [JEVfire](https://github.com/kikoncuo/jevfire) - Jev-inspired parallel decisions for CUDA LLMs via vLLM, with a browser Mario demo (~71 ms/action locally).
- [decider](https://github.com/Mapika/decider) - Qwen3.5-2B fine-tune that emits typed decisions with calibrated probabilities in one pass.
- [Parallel Constrained Decoding (Qwen2.5-1B-RLCD)](https://huggingface.co/spaces/drinkmoonshine/parallel-constrained-decoding) - Hugging Face space exploring open-source RLCD-style parallel constrained decoding.
- [eve-rlcd](https://github.com/anthony-maio/eve-rlcd) - Jev-inspired 0.6B decision model trained with reinforcement learning from right/wrong feedback only (reward: outcome minus stated probability); answers parallel Choice, Score and Noul questions over one state without generating text, with an RLVR ablation and [open weights](https://huggingface.co/anthonym21/qwen3-0.6b-rlcd-decision). Explicitly not a reproduction of TypeSafe's method.

## Articles and coverage

- [TypeSafe AI debuts model for machines that plays Doom](https://www.theregister.com/ai-and-ml/2026/09/16/typesafe-ai-debuts-model-for-machines-that-plays-doom/5296711) - The Register's launch coverage.
- [Mini-Vibe Check: TypeSafe's Jev Judged Everything I've Written in 0.7 Seconds](https://every.to/also-true-for-humans/mini-vibe-check-typesafe-s-jev-judged-everything-i-ve-written-in-0-7-seconds) - Every's Mike Taylor runs Jev over his writing corpus: 1,709 judgments for under a cent.
- [Building a harness with Jev](https://www.langchain.com/blog/building-a-harness-with-jev) - LangChain on model routing and gating dangerous tool calls behind a typed decision.
- [Jev, from a developer's angle](https://flaviocopes.com/jev/) - Flavio Copes on triage, RAG filtering, citation checks, and confidence gates.
- [Jev, Sorted](https://pearpages.com/blog/2026/09/16/jev-sorted-what-typesafes-system-one-model-actually-is-and-what-is-still-just-a-claim) - What the launch claims survive a reading of the primary sources, and what is still vendor-reported.
- [TypeSafe Jev played chess (and landed next to reasoning models)](https://dev.to/maximsaplin/typesafe-jev-played-chess-and-landed-next-to-reasoning-models-28ga) - Maxim Saplin constrains chess to legal-move Choices.
- [Jev: one judge call, or twelve dimension scores?](https://agentjournal.dev/blog/llm-judge-vs-feature-extraction/) - Independent measurement on three classification tasks, with token costs and false-positive rates.
- [Testing Jev on public and private data: classifier or filter?](https://amankumar.ai/blogs/jev-measured) - 16,000 calls vs gpt-5.4-mini and gpt-5.6-luna; where it wins, where it breaks, and a threshold procedure.
- [Jev vs Mistral and Gemini for event validation](https://nearhere.events/blog/typesafe-jev-mistral-gemini-event-validation) - Head-to-head at validating local event listings.
- [TypeSafeのJevを正しく驚く、それってLLMでできませんか？](https://zenn.dev/nwn/articles/824026c76116e0) - (Japanese) Reproduces the JSON-vs-logit shortcut on Gemma and compares Jev with LLMs on the public Mario harness.
- [jev 同士に五目並べで対戦させた](https://zenn.dev/mizchi/articles/jev-plays-gomoku) - (Japanese) Jev vs Jev gomoku with source and timing logs.
- [Jev on AI Wiki](https://aiwiki.ai/wiki/jev) - Community-maintained reference page.

## Discussions

- [Introducing System One Models and Jev](https://news.ycombinator.com/item?id=49717558) - The 1,800-point Hacker News launch thread; the sceptical reading of the benchmarks lives here.
- [Launch thread by Diogo Almeida](https://x.com/CompleteSkeptic/status/2099925682726002904) - TypeSafe's founder argues RLCD-trained decision models are a shorter path to economic value than chat models.
- [TypeSafe AI releases Jev (r/singularity)](https://reddit.com/r/singularity/comments/1whop6b/typesafe_ai_releases_ai_model_called_jev_rather/) - Reddit frames Jev as a low-hallucination, low-cost decision model for software rather than chat.
- [Testing Jev for Pi extensions (r/PiCodingAgent)](https://reddit.com/r/PiCodingAgent/comments/1whsav6/anyone_else_testing_out_typesafe_ais_new_system/) - Builders using Jev as an agent tool-use safety layer.
- [Jev "playing" Minecraft (r/accelerate)](https://reddit.com/r/accelerate/comments/1whk9oy/new_typesafe_ai_jev_model_playing_minecraft_wip/) - Work-in-progress demo, including fleeing zombies at night.
- [Model router built with Jev](https://x.com/ephraimduncan/status/2100454070536351824) - Jev decides which model should serve a request before it is forwarded.
- [MLP on Qwen 4B mimicking Jev](https://x.com/justALEXWORTEGA/status/2100341039986798930) - A small MLP on top of Qwen 4B reproduces Jev-like decision behaviour.
- [Running a local TypeSafe Jev](https://x.com/wmoto_ai/status/2100454049359577516) - (Japanese) Local Jev-style decision model attempt.
- [Jev as an AI agent safety monitor](https://x.com/isNickMa/status/2100566407524344225) - Checking each agent action first reportedly catches most attacks with almost no false blocks.
- [Rethinking security engineering with Jev](https://x.com/Kostastsale/status/2100362415187833048) - Argues purely engineering decisions in security work belong to Jev rather than a chat model.
- [Ask Jev anything, it will judge](https://x.com/waynesutton/status/2100487878992388279) - Public Convex-backed demo inviting one million judged questions.
- [First Jev use case in a Mac app](https://x.com/malekoo/status/2100439840575684910) - A shipped Mac app routes setup and troubleshooting questions to Jev when no language model is loaded.
- [Jev 中文解读](https://x.com/dotey/status/2100109937237987823) - (Chinese) The System One category explained as a calibrated, typed decision layer for code.
- [Launch roundup](https://x.com/VaibhavSisinty/status/2100619641827836222) - Browser, papers, email, trading, and games in one thread.

## FAQ

### What is Jev?

Jev is an AI model from TypeSafe AI, launched in early access on September 15, 2026. It is the first "System One" model: instead of generating text, it evaluates typed questions (Choice, Score, Noul) against a state and returns structured answers with calibrated probabilities, in 70–500 ms (vendor-reported).

### What is TypeSafe AI?

TypeSafe AI is a San Francisco AI lab founded by Diogo Almeida, previously at OpenAI, where he worked on instruction-following methods. The company raised $40M and works on "machine-native" intelligence: models built for software to consume, not for people to chat with.

### Is Jev an LLM?

No. It reads natural language but never generates text. The answer space is defined in advance by your questions, so outputs are type-safe by construction and cannot hallucinate a value outside the space you gave it. See [Jev vs LLM](#jev-vs-llm).

### What is RLCD?

Reinforcement Learning for Calibrated Decisions — TypeSafe's training method for System One models. Where RLHF optimizes for responses humans prefer, RLCD optimizes for decisions with epistemically honest probabilities: higher confidence should mean higher accuracy.

### How much does Jev cost?

$0.042 per million input tokens; output tokens are free. A typical typed question costs a tiny fraction of a cent, which is why the featured builds above report numbers like 1,709 judgments for under a cent.

### How do I get access to the Jev API?

Three ways: join the early-access waitlist at [typesafe.ai](https://typesafe.ai), use [Vercel AI Gateway](https://vercel.com/ai-gateway/models/jev) (model id `typesafe-ai/jev`, no waitlist), or use [Cloudflare Workers AI](https://developers.cloudflare.com/ai/models/typesafe/jev/) (`typesafe/jev`).

### What are Jev's limits?

Choice questions cap at 255 options. Text only — no images, audio, or video. Listed rate limits are 250,000 tokens/second and 1,200 requests/minute, and TypeSafe says they can change dynamically. Known failure modes of the current model are documented in [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13).

### What is a System One model?

TypeSafe's name for a model class built for fast, structured decisions inside software — as opposed to chat models that generate text for humans. Named after the fast, intuitive "System 1" mode of thinking. Jev is the first public one.

## Related lists

- [awesome-jev (AnotiaWang)](https://github.com/AnotiaWang/awesome-jev) - Community list of Jev applications, libraries, and resources. English and 简体中文.
- [awesome-jev (yibie)](https://github.com/yibie/awesome-jev) - Jev projects and discussions organized by application domain.
- [awesome-jev (cobanov)](https://github.com/cobanov/awesome-jev) - Curated, source-backed list of Jev projects, sorted by decision domain.
- [awesome-jev-by-typesafe (Anil-matcha)](https://github.com/Anil-matcha/awesome-jev-by-typesafe) - Evidence-backed use cases, patterns, and starter code.
- [awesome-jev-typesafe (valentynkit)](https://github.com/valentynkit/awesome-jev-typesafe) - CC0, awesome-lint clean, sorted by what you would install, with a short know-before-you-build section on the limits.
- [typesafe-ai on PyPI](https://pypi.org/project/typesafe-ai/) - Community redirect shim; the real package is `typesafe-sdk`. Registered to block slopsquatting.

## Contribute

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: open a pull request that adds a project with a link and a one-line description. It should be useful, interesting, and actually built on Jev (or clearly inspired by its interface). Mark experimental or dry-run-only paths (trading, home automation) explicitly.

Built something with Jev? Also submit it to [madewithjev.com/submit](https://madewithjev.com/submit) to get it in the directory with your reported numbers.

## License

[CC0 1.0](LICENSE) — this list is dedicated to the public domain.

---

<p align="center">
  Maintained by <a href="https://x.com/kraayenjon">@kraayenjon</a> as part of <a href="https://madewithjev.com">madewithjev.com</a> — a directory of what people are building with Jev. Not affiliated with TypeSafe AI.
</p>
