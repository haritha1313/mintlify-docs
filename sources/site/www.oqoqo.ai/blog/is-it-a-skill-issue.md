# Source: https://www.oqoqo.ai/blog/is-it-a-skill-issue

[All posts](https://www.oqoqo.ai/blog/)

May 1, 2026·By Haritha Nair·11 min read

# Is it a skill issue?

![Is it a skill issue?](https://www.oqoqo.ai/blog/issues.png)

I've been spending a lot of time lately looking at how companies are publishing agent skills, what they contain, what they skip, and whether any of it actually works. The more I look, the more it feels like we're in the middle of a shift that most teams haven't fully realised.

The mechanism is simple enough. A skill is a directory with a SKILL.md file. Frontmatter tells coding agents when to invoke it, and the body tells it what to do. At startup, agents scan available skills and load about 100 tokens of metadata per skill, just enough to know each skill exists and when it might be relevant. The full content only loads when the task matches. The idea is you can have dozens of skills installed without paying the context cost of all of them upfront.

That progressive disclosure is the cleverest part of the design. What's less clever is how most skills are actually getting built.

## How individuals are using them

For someone using coding agents personally, a skill is a workflow layer. You have a procedure you run repeatedly (PR reviews, migrations, commit message formatting), and you're tired of re-explaining it every session. Write it once, put it in `~/.agents/skills/` or `~/.claude/skills/`, and it becomes reusable across all future tasks across a given project or all projects in your machine. Same goes for org level knowledge translated into skills shareable across teammates.

Where most individual skills fall apart is the description. "Use when working with our database" doesn't match much in practice, where "use when writing SQL queries, creating migrations, or debugging slow queries" matches the actual language someone uses when they hit those problems. The description is a classifier, and most people write it like a label, which is a different thing.

Vercel ran evals on this in January for Next.js framework knowledge specifically, and found their skill went uninvoked 56% of the time. A docs index embedded directly in AGENTS.md hit 100% pass rate, while the skill maxed at 79% even with explicit instructions telling the agent to use it, and performed at baseline when left to trigger naturally. In the majority of cases the skill was simply never loaded at all. The eval was scoped to one framework, but the failure mode is that descriptions that don't match how developers actually phrase their tasks don't get matched.

## How companies are publishing them

The open standard at agentskills.io was released by Anthropic 6 months ago, and is now adopted by Cursor, Copilot, Codex, Gemini CLI, and 20+ other tools. That created an incentive for companies to publish skills the same way they publish SDKs: ship a skill, get a curated integration in every tool that supports the standard.

Most of the developer-facing teams I talk to are not 100% bought in but are still investing into it. Documentation goes stale (anyone who's maintained a large docs site knows the half-life of each page). Skills are documentation with executable consequences. If a skill tells an agent to call a deprecated endpoint, it doesn't just confuse a user but it breaks every agent that loads it across every developer using a tool with that skill installed.

The most common response is a SKILL.md that ingests the full API or docs reference, inlines entire tutorials, and embeds the SDK changelog. One artifact to rule all the context. These end up at 500+ lines, consume context on every invocation, and are nearly impossible to keep current. The maintenance overhead has just shifted from docs to skills.

Going more specifically into companies that have been investing in skills, Qdrant's skills take a different shape, more like a solution architect than a documentation dump. Sections are named by symptom rather than by feature ("Don't know what's using memory," "Search was fast now it's slow," "Approximate search worse than exact"), and each bullet links to a specific docs page. Qdrant's own framing for this is "documentation answers how, skills answer when and why."

A second pattern, less opinionated but more common, is to use the skill as a docs router. The SKILL.md tells the agent how to fetch current information rather than storing the information itself. Supabase's main skill does this — instead of trying to capture the full Supabase surface in one file, it tells the agent to search Supabase docs via MCP first, fetch docs pages as markdown, and verify against the live source before implementing anything. Supabase has also invested into other patterns we'll discuss later as alternates. Langfuse takes a similar approach, with the top-level skill routing into references that carry the workflow judgment: a prompt-migration guide that warns Langfuse prompts only support simple `{{variable}}` substitution, an SDK-upgrade reference that tells the agent to fetch the latest migration guide before touching anything, an instrumentation guide that defines what good traces actually need.

The pattern I have the strongest evidence for is not "short skills" or "docs routers," but encoded judgment. The cleanest win in my runs was MongoDB on Codex, where skill treatment both improved pass rate and reduced exploration cost. Qdrant's published skills show the same design principle at the content level, but my Qdrant run didn't show much lift, partly because the exposed skills which were getting hit for my tasks were more often broader routers and SDK indexes rather than the sharper task-shaped custom skills. Docs-router skills aren't disqualified, Supabase and Langfuse show that a router can work when it carries product judgment in its own rules or routes into references that do. The weak version is a router with no opinion, because then the agent pays latency and context cost just to rediscover docs.

The heavy wrapper is the pattern I'm most skeptical of. It can work as a one-off knowledge dump, but structurally it fights the point of skills: it front-loads too much, goes stale like documentation, and makes the agent pay for detail before it knows which detail matters.

There's a quote from a Neon case study about a previous database provider that required "maintaining a large block of custom documentation just to get the agent to interact with it correctly, and it still didn't work reliably." Custom documentation isn't enough on its own. What works is opinionated judgment, the failure modes, the sequencing buried four pages into a tutorial, the edge case that causes 80% of the support tickets, and a clear rule for when "done" actually looks done. That's what the team's best developer advocate knows, and most of it never makes it into official docs in the first place.

## The agent reads less than you think

Underneath all of this is a separate problem that I find more interesting after watching how skills actually fire in benchmark runs. Publishing a skill isn't the same as having the agent use it.

The frontmatter description is doing more work than most teams realize. It's the only thing the agent reads at startup, and what the agent doesn't read, the agent doesn't use. A description like "provides client SDKs across multiple languages" is technically accurate and operationally useless, because none of those words are what a developer types when they're stuck on a hybrid retrieval setup or a private network migration. The skill never gets loaded, the agent reaches for a more generic one (or none at all), and the output looks plausible right up until it fails on the specific detail the narrow skill would have handled.

The same problem repeats with which skills end up exposed in the first place. Companies often have their sharpest workflow guidance in skills that don't make it into the agent's startup metadata, while the ones that do are the broader references, SDK indexes, deployment guides, generic getting-started skills. So the skill that fires isn't the skill that would have helped. What matters isn't whether the skills in your repo are good, it's whether the ones the agent loads are the ones it needs when developers run into trouble. Most companies don't have visibility into that yet.

## No free lunch

Progressive disclosure gets credited as the clever part of the design, but it doesn't actually make skills free. In benchmark runs I did across Qdrant, MongoDB, Weaviate, and CockroachDB, skill treatments mostly added input and cache tokens and agent time without proportional correctness lift. The worst offenders were broad docs-router and SDK-index skills, with input token overhead even crossing 800% in the most extreme case. The cost showed up almost entirely in extra context the model had to read, not in extra output.

The exception was MongoDB on Codex, where the more operational skills (connection pooling, query optimization, schema design) used roughly 27% fewer tokens and finished 16% faster than baseline. The pattern is closer to compression than coverage, a good skill gives the agent a better path early enough that it stops exploring sooner, instead of adding reading material that doesn't actually narrow the search.

## When agents are your primary user

Most company skills right now are being treated as a marketing artifact. Ship a skill, announce support, the skill goes into the repo, and nobody is sure if they are really being used and how.

That framing is going to age badly, and the data is starting to show why.

4% of all public GitHub commits are now authored by Claude Code, and climbing fast. For developer infrastructure companies, this means AI coding agents are becoming the primary interface through which developers integrate your product, not a supplement to the human developer workflow but a replacement for a meaningful chunk of it.

When Neon reached GA last year, 30% of databases on the platform were provisioned by AI agents rather than humans, and that number recently crossed 80%. Databricks paid roughly $1 billion to acquire Neon in May 2025, and the agent-native architecture was central to the rationale. Developers were using Neon mostly through an agent.

That trajectory is coming for every developer infrastructure company. The developer experience of a product used to mean: how good are your docs, how useful is your SDK, how fast does support respond. That was a human-to-product interface, where a developer hit friction, read something, figured it out. Now increasingly the entity integrating with your product isn't a developer reading docs but an agent acting on behalf of a developer, and the agent doesn't browse your docs site or notice the outdated code sample, it loads what's in its context and executes against that.

If your skill is the thing in its context, your skill is doing the work that your docs and SDK and onboarding used to do, all at once.

## Flying blind

Individual developers can iterate fast on their own skills. Companies have a much harder version of the same problem. They can't see how developers are phrasing tasks, can't A/B test descriptions at scale, can't watch which of their skills loads when. They're writing a classifier for a query distribution they can't observe directly. Anthropic's skills guide recommends paraphrase tests as part of evaluation — running the same task with the user's real phrasing instead of the product team's clean version but most public skill examples don't show evidence of that test being run, and the skill ends up evaluated in isolated conditions it was authored for.

Battery Ventures explains the stakes: winning in developer tools used to mean nailing the first five minutes of the install. If skills become the dominant adoption surface (and the Neon numbers suggest they already are for some products), winning shifts to whoever gets embedded in the agent context most correctly.

Skills aren't the only surface where this is playing out. Companies are also publishing MCP servers, agent-friendly CLIs, dedicated SDKs, browser extensions for agents — each is a place where the agent forms its picture of the product. Skills happen to be one of the cheapest to ship and one of the most direct in shaping what the agent does, which is part of why they're worth taking seriously now.

What's actually changed is who's using your product. It used to be developers reading docs. Now it's agents reading whatever's been loaded into their context. Most companies haven't started treating that as a thing they own, and the ones that do first will have a distribution advantage that's hard to close later.

_Originally published at [harithanair.com](https://www.harithanair.com/essays/is-it-a-skill-issue)._

[Back to all posts](https://www.oqoqo.ai/blog/)