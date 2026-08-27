# Fifteen Years to My First Test

I have wanted automated tests for most of my career. I am writing this at the point where I finally have them, end to end, and am about to try writing them first. It took a long time, and the reasons are not flattering, so here they are.

## Why it kept losing

I came up through marketing and agency work. In that world speed is the product. A campaign site has a launch date and a shelf life, and the thing that gets you the next project is shipping the last one on time and making it look good. Nobody has ever won a pitch with a coverage report.

That shaped how I spent time for years, long after I had left agencies. My days went to active feature development and prototyping — the work that is visible, that moves a roadmap, that someone is waiting on. Testing was always the thing I would do after. After never came, because there was always another feature. Which meant severity was handled reactively: a bug shipped, a user hit it, I fixed it. I was good at that. Being good at it is part of why I never had to change.

I want to be honest that this was not a tooling problem or a management problem. I could have started any year. I did not, because the incentive I actually felt was always the next thing on screen.

## Declaring it

When I started at a large company, I made a decision on the way in: this time I test. Not as a team mandate — as a rule for myself, before anyone asked.

It was not easy, and the first obstacle was the codebase. It was legacy class-based React, and class components with data cached on the instance are miserable to unit test — you end up testing lifecycle plumbing instead of behavior. I was already refactoring it to functional components for memory reasons; that refactor became the opening. Every component that became a function got a unit test as it landed. Not before — I was not there yet — but alongside, which was more than I had ever done.

## The wall

Unit tests were real progress, but the dashboards' actual failures were integration failures: the data path, the host platform, the rendering. That needs end-to-end tests, and I spent days trying to make them possible and could not.

The dashboards ran as widgets inside an iframe, and the iframe was the security boundary — data access was gated to it. From a test runner outside that frame, every request hit CORS, and getting the runner *inside* it would have meant access I did not have and could not get on any timeline. I tried the workarounds. None of them held. E2E went on the shelf, and I went back to shipping features with unit tests and a feeling that I had, again, stopped short.

## The way through was a side door

What eventually made E2E possible was not a testing tool. It was a debugging feature I had built for a different reason.

To diagnose problems in the field, I had added a way to capture the data a dashboard was actually rendering — the real payload, as it arrived, saved as a fixture I could load and replay. It existed so I could reproduce a user's broken chart without their session. But a fixture that reproduces a broken chart also reproduces a working one, and a test runner that loads fixtures never needs to cross the iframe boundary at all. No live data, no CORS, no access request. The security gate was never in the way; I had just been aiming at the wrong side of it.

That is the part I would tell someone earlier in this: the thing that unblocks testing is often not testing infrastructure. It is whatever makes your application's real inputs reproducible. Build that for debugging, and testing falls out of it.

## Where I am

Today the dashboards have full end-to-end coverage running on captured fixtures, on top of the unit tests from the refactor. AI wrote most of the test code, and I am not going to pretend otherwise. What I did was decide what to test: the scenarios, the fixtures that mattered, the failures I had actually seen in the field. That split turned out to be the right one for me — the part I had avoided for fifteen years was the typing, and the part that needs an engineer who knows the system is the choosing. Having the first one cost almost nothing is what finally turned "after" into "now." The next step is the one I have never taken: writing the test before the feature.

I am not worried about being good at it. The skill was never the problem; time was, and the time it costs is now small enough that there is no excuse left. I spent fifteen years being good at the opposite, and it worked right up until it was the thing holding me back. Testing does not ride in the back seat anymore.
