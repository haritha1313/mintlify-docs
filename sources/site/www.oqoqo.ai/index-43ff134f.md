# Source: https://www.oqoqo.ai/?experiment=agent-evals

# The experimentation platform for the agentic era

Run agentic workflows in production-like environments. 
Define the experiment once, repeat it at scale, measure what changes.

[Join waitlist](https://www.oqoqo.ai/waitlist/)

Lab › Define experiment Agent Browser

Create a new task

Open five product pages in parallel browser tabs, extract the price from each, and return them as a JSON array in page order.

\+ Advanced

Add task

\+ Add rubric

Select an existing task

TaskRubricLibraryEnv

✓parallel-tab-pool––✓navigate-spa-history––✓session-cookie-refresh––✓scrape-paginated-table–––✓retry-flaky-fetch–––

Live demo · click anywhere to drive

## What do you want to test first?

[**Test agent-facing interfaces**\\ \\ Improve the token, latency, and path efficiency of your MCP servers, skills, CLIs, SDKs, and APIs before agents use them.](https://www.oqoqo.ai/?experiment=agent-interfaces#start)

[**Run scalable agent evals**\\ \\ Turn real workflows into repeatable evals across agents, models, and treatments in production-like environments.](https://www.oqoqo.ai/?experiment=agent-evals#start)

[**Gate releases in CI/CD**\\ \\ Block pull requests that break agent workflows, not just unit tests.](https://www.oqoqo.ai/?experiment=ci-cd#start)

[**Catch model and dependency drift**\\ \\ Rerun the same workflows on a schedule as models, dependencies, and your product change.](https://www.oqoqo.ai/?experiment=drift#start)

[**Build a benchmark for your product**\\ \\ Turn your user journeys into a repeatable scorecard for how agents perform on your product.](https://www.oqoqo.ai/?experiment=benchmark#start)

[**Optimize model spend**\\ \\ Compare your internal workflows across models to find where cheaper models can replace expensive ones.](https://www.oqoqo.ai/?experiment=model-spend#start)

[**Power your agent loop**\\ \\ Add a testing harness with proper rubrics and validators, so each iteration is evaluated consistently before feeding the next loop.](https://www.oqoqo.ai/?experiment=agent-loop#start)

## Agents are becoming software’s primary user.Build systems that work for them.

## Evaluate agentic workflows the way they actually run

GUICLI

Lab›Define experiment

Agent Browser

workflow: refactor-authtreatment: skill

Agents

Opus · High, Sonnet · High

Treatments

Baseline, + skill

Library

⬡ 12 repos

Environment

cloud

Trials

3

0 runsLaunch

❯\_ oqo·~/refactor-authzsh

### Define any experiment

Choose the workflow, treatment, agents, and configuration. Oqoqo turns it into a repeatable experiment.

### Run it in a production-like environment

### Capture the full trajectory

### Measure the effect

### Feed the next loop

## Evaluate agentic workflows the way they actually run

Lab›Define experiment

Agent Browser

workflow: refactor-authtreatment: skill

Agents

Opus · High, Sonnet · High

Treatments

Baseline, + skill

Library

⬡ 12 repos

Environment

cloud

Trials

3

0 runsLaunch

❯\_ oqo·~/refactor-authzsh

### Define any experiment

Choose the workflow, treatment, agents, and configuration. Oqoqo turns it into a repeatable experiment.

Experiments›refactor-auth›Run

Agent Browser

env: production-likeisolatedreproducible

sandbox-01 · trial 1running

sandbox-02 · trial 2running

sandbox-03 · trial 3running

trials0 / 36

❯\_ oqo·~/refactor-authzsh

### Run it in a production-like environment

Execute trials in clean, isolated sandboxes so agents interact with the same setup every time.

Experiments›Baseline vs Skill›refactor-auth

Agent Browser

refactor-authFail

TracesOutputEvals

#1

Instruction

Refactor auth so require() stays deterministic across larger graphs.

#2

Assistant

I’ll inspect the module cache before touching the resolver.

#3

Tool · bash

✓ read auth.ts · ✓ grep usages

#4

Tool · fs.edit

✗ edit auth.ts — TypeError: cannot read ‘config’

❯\_ oqo·~/refactor-authzsh

### Capture the full trajectory

Record tool calls, commands, tokens, outputs, and the exact step where the agent got stuck.

Experiments›refactor-auth›Compare

Agent Browser

Metrics

Steps

0

Tokens

0k

Cost

$0.00

Duration

0m 00s

Tool calls

0

Tool calls13

bash30.8%

browser.click30.8%

fs.read23.1%

search15.3%

❯\_ oqo·~/refactor-authzsh

### Measure the effect

Compare success, latency, token use, and frictions across runs. Ask questions grounded in the trace when you need the why.

Automations›refactor-auth›Loop

Agent Browser

improve → agent-facing interface

\+ workflow updated

iteration 2Re-run →

on: pull\_request…

schedule: nightly…

❯\_ oqo·~/refactor-authzsh

### Feed the next loop

Use the results to improve the agent-facing interface or workflow, then rerun automatically, on a schedule, or in CI/CD.

## One experiment, end to endand back again

Evals

✓Returns requested shape

✓Preserves order

✕Reports cache state

Library

acme/checkout-flows

acme/web-platform

data/pricing-fixtures

Agents

✓Opus 4.8High

✓Sonnet 4.6

✓GPT-5.5

Treatments

✓Baseline · raw

✓All official skills

✓Official MCP

Trials

3parallel

Environment

✓local

✓cloud

Input

Instructions

refactor-auth

parallel-tab-pool

navigate-spa-history

iteration 2 →

Sandbox

idle

Output

Steps218

Tokens8.4M

Cost$5.25

Tool calls23

traces.jsonl

auth.ts+2−1

+cache.set(key, mod)

−return require(path)

+return require(resolve(path))

## One experiment, end to endand back again

01

### Input

Everything, held constant

Prompt

task · instruction

Evals

rubric + validator

Library

repos + data

Environment

local + cloud

Product catalog

what you're testing

02

### Sandbox

Isolated, reproducible runs

Agents

models · effort

Treatments

raw · skills · MCP

Reasoning hook

capture the thinking

Trials

parallel runs

03

### Output

Everything the run produced

Traces

full trajectory

Metrics

tokens · calls · steps · time

answer.md

final answer

Diff

files changed

Frictions

where it struggled

Evals outcome

pass / fail

04

### Analyze

Find the fix

Chat with our agent

grounded in the trace

Or take it anywhere

export · API

## Frequently askedquestions

Is there a free trial?

Yes. You get up to 100 runs for free.

How long does a first experiment take to set up?

A few minutes. Pick one of our templates or define your own experiment, then configure the environment however you like.

Why can't I just use Codex or Claude Code to test my product?

You can, and you should. Oqoqo turns that useful dogfood step into repeatable testing across controlled tasks, agents, versions, environments, and product changes.

How is this different from running my own tests in a sandbox?

A sandbox just runs the agent. Oqoqo runs the whole experiment around it: production-like environments with real repos, data, and files, controlled variables across agents, models, and versions, and full traces, metrics, diffs of what changed, and evals on every run. You get repeatable, comparable results at scale instead of a one-off script you have to build and maintain.

What agents can I run?

We start with Claude Code and Codex, and are expanding coverage to Cursor, GitHub Copilot, Antigravity, and OpenCode. You can run different models and effort levels per agent.

What surfaces can I cover?

SDKs, APIs, CLIs, MCP servers, skills, and more. Anything an agent would touch can be tested and evaluated.

Can I set rubrics to perform evaluation?

Yes. You define a basic rubric and our system rewrites it to follow best practices — an out-of-the-box capability that delivers high-grade eval results at scale.

Can I see the agent traces behind a result?

Yes. Each run records files read, commands issued, tool calls made, errors hit, and the point where the agent recovered or stopped. Traces live in the workspace for every published run. A reasoning hook also captures the “why” behind each agent decision throughout the run.

## Run your first agent experiment

Start with a template or define your own. Test agentic workflows in production-like environments, measure the effect, and feed your next iteration.

[Join waitlist](https://www.oqoqo.ai/waitlist/)