# Source: https://www.oqoqo.ai/blog/how-do-coding-agents-navigate-your-product

[All posts](https://www.oqoqo.ai/blog/)

April 8, 2026·10 min read

# How Do Coding Agents Navigate Your Product?

![How Do Coding Agents Navigate Your Product?](https://www.oqoqo.ai/blog/Traces.png)

When a coding agent receives a task involving your library, what does it actually do? Does it read your documentation? Does it check what's installed? Or does it just start writing code from memory?

We analyzed 112 benchmark trials across two leading coding agents — Claude Code and Codex — to trace exactly how they discover and use a product's API. The benchmark tasks were built around a well-known vector database library, covering everything from basic table operations to hybrid search and complex codebase modifications. The results challenge common assumptions about how AI agents interact with developer tools.

## The Big Finding

**Coding agents never read your documentation.**

Across 112 trials, neither Claude Code nor Codex performed a single web search or fetched a single documentation page. Zero. Every piece of API knowledge came from three sources: what the model already knew from training, what it could discover by reading files in the workspace, and what it could extract by interrogating the installed library at runtime.

This has immediate implications for anyone building developer tools. If you're optimizing your docs site for AI agents, you may be solving the wrong problem. The agents aren't visiting your docs — they're running `help()` and `inspect.getsource()` on the installed package.

A quick primer on these two mechanisms: **`help()`** is Python's built-in command that prints a function's signature and its docstring — the short description the library author wrote in the source code. Think of it as reading the tooltip when you hover over a function in an IDE. **`inspect.getsource()`** goes deeper: it prints the actual source code of a class or function as installed on disk. The agent isn't reading documentation — it's reading the implementation itself.

## Summary of Results

Here's what we found across the two agents:

**On workflow:** Codex explores the workspace before doing anything else (74.5% of trials). Claude jumps to API probing or starts writing code immediately (49.2% of trials). Neither agent starts by reading documentation.

**On knowledge sourcing:** Claude prefers `help()` (59.6% of trials). Codex prefers `inspect.getsource()` (87.3%). Both ignore external docs entirely.

**On technology match:** When the task instruction didn't name the target library explicitly, agents defaulted to familiar alternatives 6.2% of the time — using SQLite instead of the target library for a database merge task. Every one of those trials scored near zero.

**On recovery:** When code fails, Claude is more likely to investigate why (26.7% of failures trigger new API discovery). Codex does more investigation upfront, so it has less to learn from failure (13.9%).

## Methodology

### The Benchmark

We used a vector database benchmark pack containing 35 tasks ranging from simple standalone scripts (create a table, run a hybrid search) to complex codebase modifications (modify an existing multi-service application). Each task was run as a **baseline trial** — the agent received only a natural-language instruction and a workspace, with no special guidance or skill packages.

### The Agents

- **Claude Code** (claude-sonnet-4-6, v2.1.90) — Anthropic's CLI coding agent
- **Codex** — OpenAI's coding agent

Both ran in permissionless mode with access to standard tools (file read/write, bash, web search, etc.). Each task was run across multiple independent runs to measure consistency.

### The Analysis

We parsed the complete audit logs from each trial — every tool call, every bash command, every thinking block. Each agent action was classified into one of six phases:

- **EXPLORE\_ENV** — checking the workspace structure, dependencies, task files
- **EXPLORE\_CODE** — reading existing source code in the project
- **API\_DISCOVER** — probing the library at runtime via `help()`, `inspect.getsource()`, `dir()`, or exploratory code snippets
- **BUILD** — writing or editing solution code
- **TEST** — running the solution
- **RECOVER** — any build or discovery action that occurs after a test failure

We tracked the sequence of phases per trial, the knowledge sources used, whether the agent chose the correct technology, and how these behaviors correlated with benchmark scores.

**Total dataset:** 112 baseline trials (57 Claude Code, 55 Codex) across 8 independent runs.

## Detailed Findings

### 1\. The Agent's First Move

The first substantive action an agent takes reveals its strategy. We classified the first non-thinking action across all trials:

Codex is methodical — nearly three quarters of the time, its first action is something like `ls /app` or `pip show` . It wants to understand the workspace before touching anything.

Claude is more impulsive. 40% of the time it goes straight to probing the API (running `help(table.Table.search)`) without checking what's even installed. And 8.8% of the time, it starts writing code immediately — no exploration of any kind.

That 8.8% matters. Every one of those "write first" trials was a merge workflow task, and Claude used SQLite instead of the target vector database in every single one. It never checked `requirements.txt`, never ran `pip list`, never noticed the intended library was installed. It just started writing.

### 2\. The Full Workflow Sequence

Looking beyond the first action, we tracked what percentage of trials perform each action type at each step in the sequence:

**Claude's typical sequence:**

Claude builds early and iterates. By step 2, over a quarter of trials are already writing code. The cycle is: try something, test it, fix it, test again.

**Codex's typical sequence:**

Codex investigates for three full steps before testing anything. It checks the environment, probes the API, reads existing code — then builds. The building phase comes later but with more context.

### 3\. Knowledge Sourcing: Two Opposite Strategies

The most striking finding is how differently the two agents discover API knowledge:

**Claude reads the manual.** It calls `help(table.Table.search)` to get function signatures and docstrings — the equivalent of hovering over a function in an IDE. It does this an average of 1.11 times per trial.

**Codex reads the source code.** It calls `inspect.getsource(HybridQueryBuilder)` to read the actual implementation. It does this an average of 3.27 times per trial — nearly three times as often as Claude uses `help()`.

This means Codex's understanding of an API comes from reading how the library actually works, not from what the library authors chose to document. If your docstrings are incomplete but your code is clean, Codex will figure it out. If your docstrings are excellent but your code is tangled, Claude will have the better experience.

Neither agent ever searched the web or fetched external documentation.

### 4\. When Agents Choose the Wrong Technology

In 7 out of 112 trials (6.2%), the agent used a completely different technology than intended — SQLite instead of the target library. All 7 were on task\_04, a merge workflow task.

The task instruction read: _"Repair one document's chunk range with a single merge workflow instead of manual delete-and-reinsert steps."_ It specified creating a table named `task_04_chunk_replace` with a composite key. It never mentioned the target library by name.

The results:

Claude used SQLite every single time. Its thinking block from one trial reads: _"Let me create a solution that implements a merge workflow for chunk range replacement in a SQLite database. I'll use SQLite since no external database service endpoints are available."_

It never checked `requirements.txt` (which lists the target library). It never ran `pip list`. It saw "local table" and "merge workflow" and reached for the most familiar tool.

Codex used SQLite twice but discovered the target library in the other three runs — because it explored the workspace first and found the dependency.

The score impact was total: average score with the target library was **3.27/5**, with SQLite it was **0.14/5**. The solutions were logically correct (the merge logic worked) but used the wrong library entirely.

### 5\. What Predicts Success

We measured which agent behaviors correlated with higher benchmark scores:

Technology match dominates. After that, using runtime API probing — either `help()` or `inspect.getsource()` — is the strongest predictor. Simply exploring before building has a modest effect; what matters more is _how_ the agent explores.

The codebase complexity gap is stark: standalone tasks average 3.02/5 while tasks requiring modification of existing code average just 0.75/5. Both agents struggle dramatically when they need to understand and modify an existing application.

### 6\. Recovery Behavior

58.9% of all trials encountered at least one runtime error. The agents handle failure differently:

Codex fails more often (it experiments more during its investigation phase) but has less to learn from failure because it already probed the API thoroughly. Claude fails less often but is more likely to change strategy after a failure — 26.7% of Claude's failures lead to new API discovery it hadn't done before.

This maps to their overall strategies: Claude is reactive (build, fail, learn, retry), Codex is proactive (investigate, investigate, investigate, build).

## Implications for Developer Tools

These findings suggest several practical takeaways:

**Your docstrings matter more than your docs site.** Agents discover your API through `help()` and `inspect.getsource()`, not through your documentation website. Clean function signatures, comprehensive docstrings, and readable source code are the primary interface between your library and AI agents.

**Name your library in task descriptions.** When a task instruction is technology-agnostic, agents default to the most familiar tool. If you're building integrations, prompts, or task specifications that rely on a specific library, mention it explicitly.

**Existing code is a learning resource.** For codebase tasks, agents learn API patterns by reading existing usage in the project. Well-structured example usage in your codebase may teach agents more effectively than external documentation.

**Expect two different discovery patterns.** Some agents will read your docstrings; others will read your source code. Both paths need to work. If your implementation details contradict your documentation, agents will find the discrepancy.

_Analysis based on 112 baseline benchmark trials from the oqoqo simulation framework, using a vector database benchmark pack (35 tasks, 8 independent runs, 2 agents)._

[Back to all posts](https://www.oqoqo.ai/blog/)