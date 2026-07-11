# Source: https://www.oqoqo.ai/blog/agents-need-affordance-too

[All posts](https://www.oqoqo.ai/blog/)

June 2, 2026·By Haritha Nair·13 min read

# Designing Headless Products: Agents Need Affordance Too

![Designing Headless Products: Agents Need Affordance Too](https://www.oqoqo.ai/blog/issues.png)

Almost everyone has, at some point, been made to feel stupid by an object. There is the office door with a handle on both sides, which you pull, and which does not move, until you spot the PUSH sign you walked straight past. There is the public bathroom faucet that fires water the moment you reach for a paper towel and then ignores your hands when they are actually under it. There is the elevator button that gives no sign of whether your press is registered, so you stand there pressing it a fourth and fifth time like a rat that has stopped trusting the lever. None of these things is broken in any way a test case could catch. The faucet runs, the door swings, the elevator is always coming. What failed is harder to evaluate: each object made a hidden promise about how it should be used and then turned out to mean something else.

The discipline that studies this gap has a word for the promise an object makes through its shape, and that word has organized about a century of design thinking: [affordance](https://www.nngroup.com/articles/affordances/). A handle proposes that you pull it, a flat plate proposes that you push, a knob asks to be turned, and a dirt path worn across a campus lawn tells you, against every sign telling you otherwise, where people have actually decided to walk. Most of the craft comes down to keeping that promise, which means arranging things so the action an object invites is the same action it rewards. We watch someone yank uselessly at a door and assume the problem is the person, but the problem is almost always the door, which has been lying about itself to everyone who walked toward it.

Agents are the most literal users of affordances I have ever seen. A person who pulls a push-door reads the sign, recovers, and forgets it ever happened, but an agent has no such recovery. When a surface promises one thing through its shape and delivers another, the agent keeps acting on the promise, attempt after attempt, until something finally tells it the shape was a lie. Hand your API to one and watch where it walks into the glass.

## Nostalgia

The pattern that caught my attention initially looked obvious: agents kept reaching for old, deprecated endpoints even when the SDK was on the latest version with the current endpoint right there and working. The easy explanation is that the model ingested stale examples as part of its training data and reached for them out of habit, but the reasoning trace didn't support that. The agents were being practical, and the deprecated endpoint was often the most practical thing in the room they could use.

To see why, look at what an agent has to figure out before it can write a single line against an unfamiliar service:

```
Is the service alive? What base URL / version?
Which client class is the live one in this installed version?
What's the operation actually called? search? query? find?
What auth / tenant / database scope does the call need?
What request shape gets accepted?
What does a real response look like when it comes back?
```

When the surface answers these quickly, the agent reads the answers and writes its code. When it doesn't, the agent starts knocking on doors, and the doors it tries first are the common, high-probability ones it has seen a thousand times:

```
GET /
GET /health
GET /api/v1/heartbeat
client.search(...)
client.help(...)
```

Here is where it turns into a design problem instead of a memory problem. The agent isn't seeking out deprecated routes, and it can't know in advance which guesses are current and which are stale. It guesses common shapes, and some of them happen to land on retired endpoints. What decides where it goes next is the reply, not the age of the route, and this is where old surfaces win:

```
GET /api/v2/collections -> 404 Not Found
                           (tells the agent nothing; abandoned)

GET /api/v1/heartbeat   -> 410 Gone
                           {"error": "The v1 API is deprecated.
                                      Please use /v3 apis"}
                           (confirms the product, confirms an API
                            namespace, names the active replacement)
```

A 404 is a blank wall. The [410](https://developer.mozilla.org/en/docs/Web/HTTP/Status/410) is a signpost that gives the agent three things in one response: right product, real namespace, and a direction to walk. So the intuition inverts. We treat a deprecation warning as something to clear away and delete, but to an agent it is often the best-shaped affordance the product has, the one door in the building with a clear sign on it. The agent keeps ending up at your old endpoints because your old endpoints are the ones still keeping their promises.

The deprecated endpoint is the most visible version of the problem. The same broken affordance shows up in subtler forms, and those are more instructive, because nothing announces that anything went wrong.

Take the humble identifier. One database returns a collection ID as a string over REST, and as a uuid.UUID object from its Python SDK. Both are reasonable things to call an id. But an agent that fetches the ID both ways and compares them gets this:

```
rest_id = "9f1c..."              # str, from the REST endpoint
sdk_id  = collection.id          # uuid.UUID, from the SDK

print(rest_id)          # 9f1c8e0a-...
print(sdk_id)           # 9f1c8e0a-...   <- looks identical
print(rest_id == sdk_id)   # False       <- no error, no warning
```

The agent did everything right; the two surfaces used the same word, id, to mean two incompatible types.

Names mislead the same way as types. An agent uses find() because every example it has ever seen has called the search operation something like 'find', the obvious name for the thing that searches. The installed client disagrees:

```
client.find(...)        # AttributeError: no attribute 'find'
client.run_query(...)   # works
```

In both of these the agent understood the task completely. It got stranded on the gap between what the surface looked like it should accept and what it actually accepted. That gap is an affordance failure: the invitation and the reward came apart.

There is a known interface principle underneath this: [recognition is easier than recall](https://www.nngroup.com/articles/recognition-and-recall/). A dropdown beats an empty text box, and a menu with pictures beats a waiter reciting specials. A schema with an enum, a typed field, or an example value gives the agent something to recognize, while a bare surface with no introspection and no examples forces recall. And the only thing an agent can recall is the most common version it has seen, reliably the oldest, most-documented name, which is reliably the deprecated one.

## Confusion

Recall fails one way when the surface is bare. It fails another, more expensive way when the surface offers too many doors that look alike. Give an agent run\_query and run\_query\_batch and run\_query\_groups side by side, all of them plausible, all of them named to suggest they do roughly what the task needs.

This gets worse the moment you move from a single SDK to a bag of tools, which is exactly what the Model Context Protocol hands an agent. An MCP server exposes its operations as a flat list of named tools with short descriptions, and the agent chooses among them by matching the task against those names and descriptions. When a server offers search\_files, find\_file, get\_file, and read\_document, the agent has no way to know from the outside that three of them overlap and one is the one you meant. It picks by surface resemblance, calls the wrong neighbor, reads a response that is plausible but not what the task wanted, and corrects, and every wrong turn is a full round trip. I have watched agents burn most of a task budget not on hard reasoning but on disambiguating four tools that should have been one, or that should have carried names and descriptions distinct enough to choose between on the first try.

This is the same affordance problem in a different form. A row of near-identical names invites the same confident wrong choice from everyone, and the resemblance costs the agent another round trip every time it guesses wrong.

## Promise

The single most avoidable failure I saw is this: docs tell you what to call and almost never show you what comes back, in the real language, from the version you actually have installed. So the agent calls the right method and writes code against a response it imagined:

```
result = client.run_query(...)

# the agent assumes a dict:
for hit in result["matches"]:       # TypeError: not subscriptable
    ...

# what actually came back:
# QueryResult(matches=[Record(id=UUID(...), score=0.82, ...)])
```

None of this is a reasoning failure. The agent guessed a shape because no real shape was available to analyze, and guessed in the most reasonable direction available. The fix is almost insultingly cheap: show one real return value. Not a prose description, the actual object, real types, current version:

```
>>> client.run_query(collection="docs", query=[...], limit=2)
QueryResult(
    matches=[
        Record(id=UUID('9f1c...'), score=0.82, payload={'title': '...'}),
        Record(id=UUID('1abe...'), score=0.79, payload={'title': '...'}),
    ]
)
```

One concrete example collapses a dozen guesses into a single act of recognition. "Inspect a real return value before you code against it" gets passed around as a debugging tip, but it is really a confession that the docs should have inspected it for you and showed the result. For something that learns by pattern-matching, the example is the contract.

The lazy version of this argument is "write more docs," and the evidence points somewhere more specific. In most cases the docs existed. The heartbeat endpoint was documented, search was documented, and what was missing was alignment between the docs and the thing actually running. It fails in a few recurring, fixable ways: the docs don't make the version boundary obvious, so the agent can't tell which documented surface applies to this install; they show a class or helper that isn't in the pinned version; they show what to call but never one real object coming back; they point at "v2" in the abstract without linking the exact replacement call and its response.

## Designing products that mean what they say

None of this requires deleting your old endpoints. The goal is to make the supported path the most legible thing in the room, and the usual advice gets you partway: make the shape match the behavior, so a field that needs a UUID isn't called id and typed as a string; show one real output per important call; keep the surface small enough to read. All true, all worth doing. But the more interesting moves are the ones that stop treating the agent as a passive reader of your design and start treating it as the instrument that measures it.

Treat your error messages as your most-read documentation, because they are. The agent encounters them at the exact moment it is deciding what to do next, which is the one moment your prose docs will never reach. An error that only reports what went wrong leaves the agent to guess its way out; an error that names the working call ends the guessing in one step:

```
# unhelpful - sends the agent hunting:
AttributeError: 'VectorClient' object has no attribute 'find'

# helpful - ends the hunt:
AttributeError: 'find' was removed in 1.10. Use 'run_query'.
  client.run_query(collection=..., query=...)
```

The deprecated route that returns a 410 naming its replacement is doing this by accident. Your supported surface should do it on purpose, and not only on the deprecation path. The "unknown field" error, the wrong-scope error, the malformed-filter error: each is a fork where the agent either recovers in one step or starts circling, and the difference is whether you spent a sentence telling it where to go.

Let the agent name things in its own language. A surface that returns id=UUID('9f1c8e0a-...') hands the agent a token it can only copy, not reason about; a surface that returns a human-legible handle alongside it gives the agent something it can recognize later in the conversation without a lookup. The same logic argues against making the agent assemble low-level HTTP verbs and paths by hand. Give it one named operation that says what it does, run\_query rather than a generic call it has to configure correctly from scratch, and you have moved the work from recall back to recognition.

Watch how much you return, because every token in the response has to sit somewhere. An agent does not stream a four-thousand-row result past a cursor and forget it; the whole thing lands in the same working memory it needs to hold the task, the earlier steps, and the next decision. Claude Code [caps tool responses at 25,000 tokens by default](https://www.anthropic.com/engineering/writing-tools-for-agents) for this reason, and the teams who build good agent tools ship pagination, filtering, and default limits as ordinary parameters rather than things you bolt on after a tool starts returning too much. The format matters for the same underlying reason: a model predicts each token against the patterns it was trained on, so a response shaped like the JSON and Markdown it has seen a billion times reads more reliably than a compact custom encoding you invented to save bytes.

Then read the transcripts, because the agent records exactly where your surface misled it. Point one at a dozen realistic tasks, let it work, and read back its reasoning at every step where it stalled, retried, or guessed. Each of those moments is a place where the shape promised one thing and the behavior delivered another, logged in detail by the most literal user you will ever have, and the log costs you nothing to collect. Anthropic has pushed this further, [feeding the failed transcripts back to Claude and having it rewrite the tool descriptions that tripped it up](https://www.anthropic.com/engineering/writing-tools-for-agents), then re-running the tasks to see whether the edits held. This is the loop we are building Oqoqo to run at scale: point coding agents at real APIs, SDKs, skills, and MCP servers, and measure where they fail, recover, or give up, so the audit that one careful reading of a transcript gives you can be run across a whole surface, version over version. Whichever way you collect it, the point is the same. You read the agent's confusion as a measurement of your design rather than a shortcoming of the agent.

And still, keep the surface small enough to read, because forty plausible-looking entry points, or four tools whose names all promise the same thing, is a hallway of identical unmarked doors. The kindest thing you can build is one obvious way in.

## Inevitable

The faucet runs, the door swings, the elevator is always coming. The work is the same as it has always been: close the gap between what the surface invites and what it rewards. The gift of these tireless, literal, unembarrassable users is that they show you exactly where that gap is, over and over.

_Originally published at [harithanair.com](https://www.harithanair.com/essays/agents-need-affordance-too)._

[Back to all posts](https://www.oqoqo.ai/blog/)