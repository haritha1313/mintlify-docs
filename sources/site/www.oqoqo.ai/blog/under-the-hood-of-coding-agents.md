# Source: https://www.oqoqo.ai/blog/under-the-hood-of-coding-agents

[All posts](https://www.oqoqo.ai/blog/)

April 18, 2026·By Haritha Nair·7 min read

# Under the Hood of How Coding Agents Work: Codex vs Claude Code

![Under the Hood of How Coding Agents Work: Codex vs Claude Code](https://www.oqoqo.ai/blog/benchmarks.png)

For much of 2025, Cursor dominated the coding tool landscape while Claude Code maintained a devoted following. The release of Opus models shifted Claude Code into mainstream adoption, though infrastructure challenges emerged. The introduction of Codex has reversed these dynamics — Claude Code now serves the broader developer base while a growing group of specialists advocates strongly for Codex.

Drawing on experience benchmarking developer tasks across various coding agents, I became intrigued by Codex's superior performance on complex infrastructure work. I applied systematic analysis principles to understand the underlying mechanisms, comparing Codex (GPT 5.3, GPT 5.4) against Claude Code (Sonnet 4.6, Opus 4.6) across product-integration tasks.

## The Obvious Take

A clear usage pattern emerges: developers typically choose Codex for infrastructure and backend work requiring complex logic, while selecting Claude Code for frontend tasks and migrations where maintaining context proves critical. Codex's auto-compact mechanism preserves model quality across multiple cycles, whereas Claude Code's compact process initiates a new conversation.

However, the most revealing distinction appeared in the preparation phase before code generation.

## Before Anyone Writes a Line of Code

Codex invests substantially more time in exploration before execution. It researches, examines files, and locates relevant resources. Claude Code accelerates toward implementation much faster. This creates a tradeoff: Claude Code consumes fewer tokens on simpler tasks, but requires multiple corrections on complex ones.

**Search breadth metrics** reveal significant differences:

Codex explores 3 to 17 times more broadly than Claude Code, depending on task complexity.

One representative example illustrates their divergent strategies: when promoting a prompt version through an API, Claude Code contacted the live API directly, encountered failures (Method not allowed, schema errors), and corrected subsequent attempts. Codex first examined the runtime contract, reviewed documentation, created a standardized client library for the HTTP API, and tested idempotency before execution.

## They're Not Reading Docs

A critical discovery emerged from trace analysis: agents rarely consult documentation pages. Across 73 Qdrant trials, only 3 documentation lookups occurred. In 82 Langfuse trials, just 1 appeared.

**External lookup breakdown:**

Instead of consulting traditional documentation, agents primarily interact directly with running services and infer structure from responses. This observation carries implications for documentation strategy: agents discover patterns through live endpoint probing and OpenAPI specifications rather than reading written guides.

## When the Slow One Is Worth It

Codex's exploratory approach yields dividends on genuinely complex tasks requiring substantial background information. With richer context accumulated before commitment, it encounters fewer error-correction cycles. Conversely, Claude Code excels at speed when tasks are straightforward, but expensive repair loops materialize when assumptions prove incorrect.

The fundamental distinction centers on uncertainty management. Codex reduces uncertainty through advance exploration of the live system. Claude Code allows runtime errors to surface and drive corrections. While Claude Code eventually reaches correct solutions, it accumulates error costs along the way.

The central unresolved question: if agents disregard documentation and training data inevitably lags current releases, how should practitioners deliver relevant information to agents precisely when needed?

_Originally published at [harithanair.com](https://www.harithanair.com/essays/under-the-hood-of-coding-agents)._

[Back to all posts](https://www.oqoqo.ai/blog/)