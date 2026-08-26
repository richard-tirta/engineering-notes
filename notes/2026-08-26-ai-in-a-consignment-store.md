# AI in a Consignment Store

I build the software for a family-run consignment gallery. It has one shared mailbox, one owner who reads it, and no IT department. This is a note about where a language model actually earned its place in that business, how I keep it from costing anything, and the one rule that made it usable by someone who is not an engineer.

## Problem one: the inbox was unreadable

Everything arrives in the same mailbox. People offering a sofa. People looking for a sofa. Vendors pitching. Utility bills, SaaS receipts, Instagram notifications. The first version of triage only knew about consignment, so everything else piled up under a bucket called "Review," and Review stopped meaning anything.

The fix was a taxonomy with six buckets: consignor, wishlist, vendor, review, etc, spam. The important one is *etc* — it absorbs automated-but-legitimate noise so that *review* shrinks to its real meaning: a human whose intent is unclear. A small model classifies each message once, the result is cached, and the owner opens a mailbox that is already sorted by what he has to do about it.

The first run on real mail found something nobody had counted: 27% of what came into the "consign" address was buyers, not sellers — people writing to say *I'm looking for a mid-century credenza*. That demand had been sitting in the inbox unlabeled for years. It now has its own bucket and its own routing.

## Problem two: retyping

Consignment starts as an email with photos. Someone sends four pictures of a dining set and a paragraph about it. Downstream, the owner reads that and types an item list into the agreement by hand.

Now, when he accepts a consignor, a vision-capable model reads the email body and the attached photos and proposes the item list — description, brand if visible, category, condition, and a confidence score. It never proposes a price; pricing stays human. The proposals appear in the agreement panel as chips with a "Use" action. Nothing is filled in until he clicks.

Every click is recorded. A row he took verbatim, a row he took and edited, and a row he typed from scratch are three different signals, and the edited ones are the valuable part: they are the store's own house style, corrected by the person who owns it. So far about 77% of new item rows start from a chip, and 70% of those are accepted unchanged. The corrections are being collected to feed back into the prompt once there are enough of them to matter.

## Problem three: paying for it

The whole AI layer costs about two dollars a month. That is not because the volume is tiny — a retail store's shared mailbox gets plenty of mail. It is because every model call is gated on something, and the gate is usually a person.

**Choosing the model by measurement.** Before picking a model for triage, I ran the small one and the large one against the same real inbox. They agreed 95% of the time on the buckets; the disagreements were all on the genuinely ambiguous axis, where I would have hesitated too. So the small model shipped.

**Only classify what someone will look at.** The inbox loads one week at a time, each message is classified once, and the label is cached. Nothing in the archive is ever sent to a model, because nobody is going to read the archive. That alone keeps triage on the free tier of a hosted gateway almost all the time — the free tier is a per-minute rate cap, not a spend cap, and normal use fits under it.

**The expensive call waits for a decision.** The vision pass is the costly one, and it would be easy to run it on every consignor email as it arrives. I do not, because the owner accepts a bit under half of the inquiries, and the rest will never become an agreement. So it runs on *accept* — a human decision that already happened, and the first moment the photos are worth a model's attention. Accept is also the only moment the app holds a live token for the mailbox, so that is when the photos are copied to private storage; every later step reads the stored copies, never the mailbox.

**Three lanes, one rule.** After accept, the extraction can run in one of three places, and the difference is whether a human is waiting. A *background* lane starts right after the accept action returns, off its critical path. An *urgent* lane runs on Generate Agreement if the suggestion is not ready — the owner is on screen, so on a throttle it fails over to the paid API immediately. A *nightly cron* around two in the morning re-runs anything deferred or failed from the stored photos, free tier only, with a retry budget, and sweeps photos past a sixty-day TTL. The rule across all three: pay for latency only when someone is blocked on it; everything with slack rides the free tier, later.

**The revision.** The first background lane obeyed that rule too strictly — on a throttle it deferred to the nightly drain, since the chips were not needed until the agreement was generated. But the gateway throttles reliably during the hours the owner is actually triaging, so daytime accepts deferred every time, and when he opened the agreement that afternoon the chips were not there. The saving was real and the cost was his time. The background lane now pays on throttle too; the nightly drain is a safety net, not the main path. At this volume the change is pennies.

Nearly all of the two dollars is that failover. Each row records which model served it, so the bill is a list of moments someone was waiting rather than a number to argue about. I have deliberately not added prompt caching to the vision call: caching pays when a prefix is reused within minutes, and these calls fire once per accept, days apart.

## The rule underneath all of it

Early on, the agreement's price field had a placeholder — "$2000–$3000" — meant as a format hint. The owner read it as an entered value. That one moment set the design constraint for everything above.

Model output must look like a suggestion, never like data. In practice:

- A format hint lives *outside* the field, as a label. Never as ghost text that imitates a value.
- A suggested value is explicitly labeled, visibly system-sourced, and needs a deliberate action to accept. He always knows he is accepting a suggestion, never discovering something already "in."
- Anything derived from a stale snapshot carries a freshness label, so it reads as advisory.

This is why the item chips never auto-fill, why triage is a label on the message rather than a rewrite of it, and why the model never touches price. The business problem was never "we need AI." It was that one person reads every email and types every agreement, and each of these takes a step of that off his plate without ever making a decision for him.
