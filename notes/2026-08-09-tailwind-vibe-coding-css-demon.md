# Tailwind, Vibe Coding, and the CSS Demon

Tailwind itself isn’t necessarily the disaster.

**Tailwind + vibe coding is where you can summon the CSS demon.**

AI is *very* happy to solve visual problems locally.

“move this a little right” → `ml-2`  
“still wrong” → `ml-3`  
“but desktop broke” → `lg:ml-1`  
“tablet?” → `md:ml-[11px]`  
“dark mode?” → another utility  
“this one page is special” → conditional `className`  
“perfect!” → **SHIP IT ☠️**

Six months later:

```tsx
<div
  className={cn(
    "flex relative w-full px-4 md:px-6 lg:px-[22px]",
    isOpen && "ml-2 md:ml-0",
    compact && "px-2 gap-[7px]",
    "hover:bg-slate-800/40",
    className
  )}
>
```

And now nobody can explain the styling system because there may not actually be one.

What you have instead is a geological record of prompts.

Every individual change might have been reasonable at the time. The user wanted something moved. The AI moved it. A breakpoint broke, so another utility fixed the breakpoint. A special case appeared, so another conditional got added.

The problem is that nobody stopped to ask:

**Why did this element need the margin in the first place?**

Maybe the child isn’t wrong.

Maybe the parent is wrong.

Maybe the spacing should come from `gap`.

Maybe the container width is wrong.

Maybe the component boundary is wrong.

Maybe there is supposed to be a design token here instead of `11px`.

There’s another failure mode too: specificity becomes harder to reason about.

Tailwind doesn’t remove the cascade. It just makes it easier to forget that the cascade still exists.

Once utilities are coming from the component, props, conditional `cn()` calls, responsive variants, and caller-supplied `className`, debugging “why is this rule winning?” can become less obvious than looking at a stylesheet.

```tsx
className={cn(
  "text-sm text-slate-700",
  active && "text-blue-600",
  disabled && "text-slate-400",
  className
)}
```

Now add `className="text-red-500"` from the caller. Which one wins?

The answer isn’t necessarily just “the last string I see.” The generated CSS, variants, arbitrary selectors, class-merging behavior, and other layers can all affect the result.

And eventually somebody reaches for:

```tsx
"!text-red-500"
```

At that point the CSS demon has evolved.

The problem isn’t that Tailwind somehow invented specificity. The problem is that utility-heavy code can make the underlying CSS behavior easier to ignore until something stops behaving the way you expect.

That is the part AI doesn’t naturally force you to think about. Its default incentive is to satisfy the immediate request. If the screenshot now looks right, the task appears solved.

And Tailwind makes that kind of local patch incredibly cheap.

That doesn’t mean Tailwind is bad. A well-structured Tailwind codebase can be disciplined, consistent, and easy to maintain. Utility classes can work very well when they sit on top of a real component system, sensible design tokens, and engineers who understand the CSS underneath them.

But Tailwind also makes it easy to keep stacking local answers until architecture disappears underneath the utilities.

AI-assisted coding amplifies that tendency.

This is why I don’t think AI makes CSS knowledge less important.

I think it makes **CSS reasoning more important**.

Tailwind abstracts syntax. It does not abstract CSS behavior.

You still need to understand the cascade, specificity, inheritance, layout, source order, and what the browser is actually doing.

The valuable skill is no longer just knowing how to produce the visual result. AI can often do that very quickly.

The valuable skill is recognizing when the generated solution is fighting the browser, fighting the component hierarchy, or quietly turning a design system into a collection of exceptions.

Sometimes the right answer is not another utility class.

Sometimes the parent is wrong.
