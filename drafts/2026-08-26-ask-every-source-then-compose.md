# Ask Every Source, Then Compose

The previous version of the assistant made the user pick an agent before asking a question.

There were three: one over raw operational data through an MCP service, a knowledge RAG over documents, and a RAG over dashboard content. Each was a separate subagent in a Chainlit prototype, and the first thing you did was choose which one to talk to. That is a reasonable way to build a demo. It is a strange thing to ask of a person, because the person does not know which source holds their answer — that is why they are asking.

The two RAGs were built by colleagues; I consumed them. My job was to replace the prototype with a Next.js product UI, and the part worth writing down is the orchestration layer I put in front of those three sources: one agent, one question, one answer.

## Why a router alone is the wrong shape

The obvious design is a classifier: a small model decides which backend a question belongs to, and that backend answers. The trouble is that most real questions belong to more than one. "How is yield trending on line four?" is a data question. It is also a knowledge question (what counts as yield, what changed last quarter) and a dashboard question (there is already a chart for this). Routing to one source gives a correct, thin answer. The user wanted the whole thing.

## Classify as a hint, ask everyone

So the design does both. Haiku classifies the question — fast, cheap, easy for most questions — and that classification decides which source should *lead*. But it is not a gate. Every question also fans out to all three sources in parallel, and each is asked a narrower question than the user's: *can you answer this, and with what?* A RAG that has nothing relevant says so, cheaply. The MCP service either returns rows or it does not.

Then composition. The answer is assembled in a fixed order, because a fixed order is what makes it read as one voice instead of three pasted together:

1. **Lead with knowledge.** The document answer frames the question in the domain's own terms.
2. **Support with data.** If the MCP service returned rows, render them as a chart, not prose. Numbers in a paragraph are where trust goes to die.
3. **Point to the dashboard.** If the dashboard RAG found an existing view, link it — "see this live here." The assistant should hand off to the tool that already exists, not compete with it.
4. **Finish with citations.** Every claim from a document points back to its source.

The language model's job in this design is narrow: turn three structured results into connective prose. It is not the source of truth for any of them.

## Grounding, measured

The composition step is where a model drifts. Given a knowledge snippet and a table, it will happily invent a trend the table does not show. The fix was grounding: the composer is constrained to the returned material, and we measured whether it stayed inside the lines.

We built a set of fifty golden queries — real questions with a known correct routing and a reference answer. Haiku generated the candidate set; humans corrected it. Then every candidate for the composition and routing roles ran the same fifty: Haiku, Sonnet, and Gemini Flash.

The point of the exercise was not to find the best model. It was to find the cheapest one that matched the best one on this specific task. That is a number you can only get by running it — arguing about model tiers from benchmarks tells you nothing about *your* fifty questions.

Do not make the user route. Choosing an agent is the system's job, and the best version of that job is not choosing at all. Multi-source answers are a composition problem, not a routing problem: ask every source what it can contribute, fix the order the answer is assembled in, keep the model on a short leash between the pieces, and measure the leash with a golden set you actually own.
