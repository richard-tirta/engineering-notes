# One Percent Is a City

*Written in 2026 about work from 2023–2024. Filed under the year it happened.*

I spent a stretch of my career on the small-business side of a platform with roughly three billion users — the tools a shop owner in Jakarta or Lagos or São Paulo uses to run ads and manage a page. Most of what I learned there about inclusion had nothing to do with assistive technology. It was about language, and about a number.

## The number

In most scaling conversations, one percent is a rounding error. A feature that one percent of users touch is a candidate for deletion. A bug that hits one percent ships.

On a platform that size, one percent is about the population of Beijing.

That reframing changed how I read every decision. "Only" the users on a particular locale, "only" the ones on a right-to-left script, "only" the ones whose currency has no minor unit — each of those "only"s is a major city, full of small-business owners whose livelihood runs through the product. You do not get to ignore a city because it is a small fraction of a country.

## Internationalization is the easy half

Engineers tend to hear "internationalization" and think of the mechanics: externalize the strings, handle plurals, format dates and numbers per locale, make the layout survive German compound nouns and Arabic direction. That work is real, and it is the part engineering can do alone.

The hard half is what sits under it — localization and regionalization. Not "is this string translated" but "does this make sense to a person here." A payment flow that assumes a credit card. An ad objective named after a metaphor that does not exist in Vietnamese. A help article that references a holiday sale season the region does not have. Those are not string bugs. They are product decisions that were made once, in one culture, and then shipped everywhere.

## A language is not a country

The most common wrong assumption is the one baked into the word "locale" itself: that a country has a language, so a user's country tells you what to show them.

Most people picture the United States as an English-speaking market. Tens of millions of people there run their lives in Spanish, Chinese, Vietnamese, Tagalog. A shop owner in Texas may want the ad tools in Spanish and the invoices in English, because that is who reads each one. It runs the other way too: plenty of people in Indonesia would rather read a product interface in English than in Bahasa, because they learned the product's vocabulary in English, or because the translation reads like a translation. Serving them Indonesian because their IP address is in Jakarta is not localization. It is a guess wearing localization's clothes.

Language is a preference the user holds, not a property of the border they are inside. The product has to ask, remember, and let them change their mind — per person, sometimes per surface.

## Translation is expensive, and AI did not change that

The instinct now is that machine translation makes this cheap. It makes the first draft cheap. It does not make the *process* cheap, and the process is where the cost lives.

Every string goes out to dozens of languages. If a product manager changes one word after the handoff — tightens a label, renames a feature — that is not one edit. It is that edit multiplied by every language, each re-reviewed, each re-shipped. Late changes are the single most expensive thing in a localization pipeline, and they are invisible to the person making them because they see one string.

So the discipline that matters is a **content lock**: copy is final before it goes to translation, and "final" means something. Nobody gets to tweak a label the week before launch without understanding that they are reopening thirty deliverables.

## Why humans still review

The other thing AI did not remove is human review, and the reason is context.

A machine can translate "campaign" and "reach" and "conversion" into any language. It cannot know that in this product, "reach" is a specific metric with a specific definition, not the ordinary word, and that the translation has to match the one already used in the analytics dashboard, the billing page, and the help center, or the shop owner will think they are three different things. Product jargon is a controlled vocabulary, and consistency across surfaces is what makes it learnable. A native-speaking reviewer who knows the product catches that. A model, per string, does not.

The same reviewer catches the things that are not wrong but are off — the formal register where the market expects casual, the example business name that means something unfortunate locally, the metaphor that landed in English and lands nowhere else.

Tone is the subtlest of these, and it does not survive translation at all. A blunt call to action — *Buy now. Start today.* — is written to read as confident. Whether it actually reads as confident, or as pushy, or as slightly rude, depends on where it lands. Latin American markets were where this came up for us: the same button copy that tested fine in one market needed a different register in another, and nothing about the words themselves would tell you that. Only someone from there would. A perfectly accurate translation of the wrong tone is still the wrong tone.

## Inclusion, at scale

I came into this thinking of inclusion as an accessibility problem — a door that opens for the person with a screen reader. It is that. It is also a door that opens in the right language, with the right examples, for a shop owner who will never file a bug report, because from where they sit the product is simply not for them.

At the scale I was working, that shop owner was one of thirty million. One percent. A city.
