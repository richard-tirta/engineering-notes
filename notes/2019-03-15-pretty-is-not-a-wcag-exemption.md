# Pretty Is Not a WCAG Exemption

*Written in 2026 about work from 2018–2019. Filed under the year it happened.*

For a stretch of my time as a frontend engineer at a fast-fashion retailer, my job was accessibility, because the company had been sued under the ADA and the website had to reach WCAG AA. Not "improve." Reach.

I want to write down what that was actually like, because most accessibility writing is about the standard, and the standard was never the hard part.

## The hard part was the room

Fast fashion sells on looks. The brand lived on stylized display fonts, saturated colors, autoplaying video with sound, and a mega menu that unfolded on hover into a full-width wall of categories. Every one of those was a deliberate design choice, made by people who were good at their jobs. And every one of them failed.

The fonts and colors failed contrast — constantly. A thin display face in brand pink on white does not reach 4.5:1, and it does not get closer by being on-brand. The mega menu could not be navigated with a keyboard, and to a screen reader it was either invisible or a hundred links with no structure. The video autoplayed with audio, which is disorienting for sighted users and actively hostile to anyone using a screen reader, who now has two voices talking at once.

None of this was carelessness. The designers did not understand accessibility because nobody had ever asked them to, and from where they sat I was the person showing up to make the site uglier.

## What I learned to say

"This fails 1.4.3" is true and changes nobody's mind. A success criterion number is a citation, not a reason. Two things did the work instead.

**Show the user, not the rule.** A designer who has never used a screen reader has no model of what a hover-only mega menu is like when you cannot hover. Demonstrating beats describing: tab through the header, hear the menu read out, try to reach a category page. Try to read the pink-on-white promo with the brightness turned down. Once someone has seen it, the conversation stops being about compliance and becomes about a customer who cannot buy a dress.

**Separate what is possible now from what needs design work.** Some of this is just engineering. Focus states, keyboard handling, skip links, ARIA on the menu, a pause control and captions on the video — I can do those without asking anyone. Some of it genuinely requires a design decision: if the brand color cannot carry body text at 4.5:1, then either the color changes, or the text gets larger (WCAG relaxes to 3:1 for large text — the standard has more room in it than people assume), or that color is reserved for things that are not text. Those are design choices, and they belong to design.

Putting those in two columns was the whole unlock. The first column stopped being a fight. The second column became a collaboration, because designers were being asked to design, with a constraint, rather than being handed a list of violations.

## What accessibility is for

The lawsuit is why the work got funded. It is not why the work matters.

The word "accessibility" makes people picture a blind user with a screen reader, and then quietly decide that is a small audience. It is not that audience, and it is not small.

The contrast rule is for the customer who is colorblind — about one in twelve men — and for the customer in her sixties whose eyes no longer separate light pink from white. It is for everyone reading on a phone in sunlight. The stylized display font is a wall for someone with dyslexia, who is already working harder than you to read plain type. A mega menu that unfolds into a hundred links at once is a problem for keyboard users, and also for someone with ADHD or an anxiety disorder for whom a wall of options is not a menu but a demand. Autoplay with sound is hostile to screen reader users, and to anyone with sensory sensitivity, and to the person shopping quietly next to a sleeping kid.

Most of these people do not think of themselves as disabled. They just find the site tiring and go somewhere else. And nearly everyone joins one of these groups eventually — eyesight, dexterity, and attention all change with age. Accessibility is not a feature for a minority. It is the store deciding that its door opens for the people it will actually have as customers, including the ones its own staff will become.

A retail site is a store. If a customer cannot get from the homepage to checkout, the store has a door that does not open for them. That is not a compliance problem. That is a store deciding who it will serve.

"Pretty" and "accessible" were never actually in conflict. The conflict was between *this specific pretty* and the customers it excluded, and a good designer, once they can see those customers, will find a different pretty. The compromise we reached was not a less pretty site. It was a different pretty, with a door that opened.

If you are the engineer in this seat: do not lead with the criterion number. Lead with the person. Then hand design the parts that are theirs.
