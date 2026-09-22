# AI in a Consignment Store

I build the software for a family-run consignment gallery. It has one shared mailbox, one owner who reads it, and no IT department. This is a note about where a language model actually earned its place in that business, how I keep it from costing anything, and the one rule that made it usable by someone who is not an engineer.

## Problem one: the inbox was unreadable

Everything arrives in the same mailbox. People offering a sofa. People looking for a sofa. Vendors pitching. Utility bills, SaaS receipts, Instagram notifications. The first version of triage only knew about consignment, so everything else piled up under a bucket called "Review," and Review stopped meaning anything.

The fix was a small taxonomy, and the important bucket in it is the one that absorbs automated-but-legitimate noise, so that *review* shrinks back to its real meaning: a human whose intent is unclear. A small model labels each message once, and the owner opens a mailbox that is already sorted by what he has to do about it.

The first run on real mail found something nobody had counted: 27% of what came into the "consign" address was buyers, not sellers — people writing to say *I'm looking for a mid-century credenza*. That demand had been sitting in the inbox unlabeled for years. It now has its own bucket and its own routing.

## Problem two: retyping

Consignment starts as an email with photos. Someone sends four pictures of a dining set and a paragraph about it. Downstream, the owner reads that and types an item list into the agreement by hand.

Now, when he accepts a consignor, a model reads the message and the photos and proposes the item list. It never proposes a price; pricing stays human. The proposals appear as suggestions with a "Use" action, and nothing is filled in until he clicks.

Every click is recorded. A row he took verbatim, a row he took and edited, and a row he typed from scratch are three different signals, and the edited ones are the valuable part: they are the store's own house style, corrected by the person who owns it. So far about three quarters of new item rows start from a suggestion, and most of those are accepted unchanged.

## Problem three: paying for it

The whole AI layer costs about two dollars a month. That is not because the volume is tiny — a retail store's shared mailbox gets plenty of mail. It is because every model call is gated on something, and the gate is usually a person.

**Choose the model by measurement.** Before picking a model for triage, I ran the small one and the large one against the same real inbox. They agreed 95% of the time; the disagreements were all on the genuinely ambiguous axis, where I would have hesitated too. So the small model shipped.

**Only classify what someone will look at.** Mail is labeled as the owner reaches it, once, and the label is kept. Nothing in the archive is ever sent to a model, because nobody is going to read the archive.

**The expensive call waits for a decision.** Reading photos is the costly step, and it would be easy to run it on every consignor email as it arrives. I don't, because the owner accepts a bit under half of the inquiries, and the rest will never become an agreement. So it runs on *accept* — a human decision that already happened, and the first moment the photos are worth a model's attention.

**Pay for latency only when someone is waiting.** Work with slack in it can take the cheap, slow path. Work a person is standing at the screen for gets the fast path, whatever it costs. I got this wrong once in the frugal direction: an early version deferred suggestions during exactly the hours the owner was triaging, and he opened agreements that afternoon to find them empty. The saving was real and the cost was his time. At this volume the difference is pennies, and pennies are the right price for not making him wait.

Each suggestion records what produced it, so the bill reads as a list of moments someone was waiting rather than a number to argue about.

## The rule underneath all of it

Early on, the agreement's price field had a placeholder — "$2000–$3000" — meant as a format hint. The owner read it as an entered value. That one moment set the design constraint for everything above.

Model output must look like a suggestion, never like data. In practice:

- A format hint lives *outside* the field, as a label. Never as ghost text that imitates a value.
- A suggested value is explicitly labeled, visibly system-sourced, and needs a deliberate action to accept. He always knows he is accepting a suggestion, never discovering something already "in."
- Anything derived from a stale snapshot carries a freshness label, so it reads as advisory.

This is why the item suggestions never auto-fill, why triage is a label on the message rather than a rewrite of it, and why the model never touches price. The business problem was never "we need AI." It was that one person reads every email and types every agreement, and each of these takes a step of that off his plate without ever making a decision for him.
