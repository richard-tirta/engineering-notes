# The 20 GB Browser Tab

The bug report was not "the dashboard is slow." It was "the dashboard crashed my machine."

Users were loading a little over five million data points into a set of analytics dashboards I owned, hosted as widgets inside a larger internal platform. At that volume the browser process climbed past 20 GB of RAM. It did not degrade. It took the whole workstation down with it.

The first thing everyone said, including me, was some version of "that's too much data for a browser." Which is true, and also not the problem.

## The data was in the browser on purpose

Those dashboards held the full dataset client-side by design. Sort, filter, split-by — every analytical interaction recomputed locally, so it felt instant. Pushing that back to the server would have meant a round-trip and a full recalculation on every control change. The team had chosen interaction latency over memory headroom, and that choice was the product.

So "paginate it" or "downsample it server-side" would have fixed the crash by destroying the thing the design was protecting. The job was narrower and harder: buy the headroom back without reversing the tradeoff.

## Profile first, theorize later

I had a theory. The codebase was legacy class-based React, and I had a strong hunch about what the class pattern was doing with large datasets. I wrote that up separately — [React Components Are Not Domain Objects](./2026-08-10-react-classes-oop-and-memory.md) — because the mechanism deserves its own entry.

But I did not act on the theory. I opened Chrome DevTools and took heap snapshots of a 20 GB page, which is its own small adventure, and the profile said something I would not have guessed.

There were two problems, with two different owners.

**One I did not own.** The host platform exposed a context API to widgets like mine, and it was recording every change made on a dashboard — an append-only history that grew for as long as the session lived. Unbounded, and invisible from inside my code. This was the larger structural term.

**One I did.** My own layer had real overhead: derived datasets cached on component instances where they stayed reachable long after they were stale, and a backend response carrying fields the UI never rendered but the browser was holding anyway.

## What I got wrong

I assumed the whole problem was mine. Not out of humility — out of habit. When your dashboard crashes, you look at your dashboard. If I had gone straight to the refactor I had already decided on, I would have shipped a real improvement, watched the page still blow past its limits, and concluded that the browser simply could not do this.

The profile crossed a team boundary, and I had to follow it there.

## Two fixes, in parallel

For the part that was not mine, I took the heap snapshots to the team that owned the platform layer. Evidence, not a complaint. They fixed the unbounded history.

For the part that was, I did two things at once: refactored the class components to functions with hooks, which made the data flow explicit and got the cached intermediates off long-lived instances; and trimmed the response payload to what the UI actually consumed. Aggregation stayed client-side. Only the dead weight left.

## Result, stated honestly

With both fixes in, page memory dropped by up to 90%, and the ceiling moved from crashing at five million points to handling over a hundred million.

I want to be precise about that number: it was measured after *both* changes. Part of the win was another team's fix that I diagnosed. I have seen people tell this kind of story as a solo optimization, and it is weaker that way — the seam is findable, and the actual skill on display was profiling instead of guessing, working out which problem was mine, and bringing evidence across the boundary rather than waiting on it.

The GPU rendering path has its own limit; it surfaces around eight dashboards open at once. This raised the ceiling substantially. It did not remove it. Naming the next constraint is part of the fix.

## The lesson I keep

When something you own is on fire, the profile does not care about your org chart. Follow it, even when it leads into code you cannot change, and do your own half while you wait.
