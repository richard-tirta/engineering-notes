# React Components Are Not Domain Objects

I understand why backend engineers like class-based React.

If you spend most of your time in object-oriented systems, this feels natural:

```tsx
class Dashboard extends React.Component {
  state = {
    processedData: null,
  };

  componentDidMount() {}
  componentDidUpdate() {}
  componentWillUnmount() {}

  render() {
    return <Chart data={this.state.processedData} />;
  }
}
```

There is an object. It owns state. It has methods. It has a lifecycle.

That is a perfectly reasonable model in many backend domains.

But I don't think it is a good reason to use class-based React.

The issue isn't that object-oriented programming is bad. The issue is that **a persistent mutable object is not a particularly good model for a React render**.

## The dangerous word is `this`

A class component encourages us to think of the component as one object living through time:

```text
Component instance
│
├── this.props
├── this.state
├── this.processedData
├── this.cache
├── this.handlers
│
├── componentDidMount()
├── componentDidUpdate()
└── componentWillUnmount()
```

That persistence can be convenient.

It can also make ownership surprisingly hard to reason about.

Anything reachable through that instance can potentially remain reachable for the lifetime of the component.

For ordinary UI data, that may not matter much.

For data-heavy applications, it can matter enormously.

## I learned this with a 20+ GB browser process

I once worked on a visualization application where the incoming payload could already be roughly 20–50 MB.

The problem was not simply the size of the payload. The data was being recalculated as it traveled down the component tree.

Conceptually, the application was doing something like:

```text
Large payload
    │
    ▼
Parent component
    ├── original data
    └── derived data A
             │
             ▼
Child component
    ├── props / retained references
    └── derived data B
             │
             ▼
Another component
    ├── filtered/grouped data
    └── another representation
             │
             ▼
Plotly
    └── plot-ready traces and structures
```

Each level could create another set of arrays and objects.

The result was not a tidy 50 MB payload making its way to a chart.

It was an expanding object graph.

Under the wrong conditions the browser process could grow beyond **20 GB of RAM**.

That experience permanently changed how I think about frontend data ownership.

## Garbage collection doesn't care that data is "old"

JavaScript garbage collection fundamentally cares about reachability.

If an old object can still be reached, it cannot be collected yet.

So this:

```js
this.processedData = transform(this.props.data);
```

doesn't imply that the previous `processedData` immediately disappears.

If something else still references it — a component instance, callback, cache, subscription, pending asynchronous operation, or visualization library — that object graph remains alive.

Now combine that with repeated transformations:

```js
const filtered = data.filter(...);
const mapped = filtered.map(...);
const grouped = groupData(mapped);
const traces = buildPlotlyTraces(grouped);
```

For a small dataset, nobody cares.

For tens of megabytes of source data, every intermediate representation is a real allocation.

If several generations overlap, memory can explode very quickly.

## Functional React changes the default mental model

Functional React encourages a different model:

```tsx
const Chart = ({ data }) => {
  const processedData = transform(data);

  return <Plot data={processedData} />;
};
```

The component itself is not a persistent mutable application object.

It is much closer to:

```text
UI = render(state, props)
```

React owns the persistent component identity and hook state. The component function is invoked to calculate a render snapshot.

```text
React-owned state
      │
      ├── render → snapshot A
      ├── render → snapshot B
      └── render → snapshot C
```

When an obsolete snapshot and its temporary objects are no longer reachable, they can become eligible for garbage collection.

This does **not** mean functional components cannot leak memory.

They absolutely can.

A closure can retain a giant object. An effect can retain a subscription. A ref can retain almost anything. A cache can grow forever.

The advantage is not "functions don't leak."

The advantage is that functional React makes this architecture natural:

> Inputs come in → calculate the current representation → render it → let obsolete work die.

That model fits React much better than treating every component as a long-lived mutable domain object.

## Hooks organize code around concerns, not the lifetime of an object

Class React also tends to split one concern across lifecycle methods:

```tsx
componentDidMount() {
  subscribe(this.props.roomId);
}

componentDidUpdate(prevProps) {
  if (prevProps.roomId !== this.props.roomId) {
    unsubscribe(prevProps.roomId);
    subscribe(this.props.roomId);
  }
}

componentWillUnmount() {
  unsubscribe(this.props.roomId);
}
```

But the actual requirement is simpler:

> Keep this subscription synchronized with `roomId`.

Hooks let us express that directly:

```tsx
useEffect(() => {
  subscribe(roomId);

  return () => {
    unsubscribe(roomId);
  };
}, [roomId]);
```

Instead of asking:

> What point in my object's lifecycle am I in?

we can ask:

> What external behavior needs to be synchronized with this render?

That is a much better fit for React's rendering model.

## Functional React does not mean OOP is forbidden

This is where the argument can easily become silly.

Moving away from class components does not mean abandoning:

- objects
- encapsulation
- interfaces
- modules
- dependency boundaries
- domain models
- object-oriented design where it actually fits

There may be perfectly good reasons to write:

```ts
class PricingEngine {
  calculate() {}
}
```

The narrower point is:

> **A React component does not need to become a long-lived mutable domain object merely because the rest of the system uses OOP.**

React already owns component identity, state scheduling, rendering, and lifecycle.

Wrapping those concepts in another persistent object model can create competing ideas about who owns state and how long data should live.

## `useMemo` doesn't magically fix memory either

A common reaction to expensive data processing is:

```tsx
const processedData = useMemo(
  () => transform(data),
  [data]
);
```

Sometimes that's exactly right.

But `useMemo` is not a free memory optimization.

From a memory perspective, memoization means something close to:

> Keep this value reachable because I may want it again.

For a tiny derived object, who cares.

For a gigantic visualization structure, that is an architectural decision.

Memoization trades computation for retention.

Sometimes recomputing a small result and allowing the previous one to die is better than deliberately retaining a huge intermediate structure.

## Fix the data architecture, not just the component syntax

The real solution to the 20 GB problem was not simply:

```text
class → function
```

The deeper problem was repeated derivation and unclear ownership of large datasets throughout the component tree.

For large visualization workloads, I would much rather see:

```text
Large API payload
        │
        ▼
Data transformation boundary
        │
        ▼
Plot-ready model
        │
        ▼
React presentation components
        │
        ▼
Plotly
```

than:

```text
payload
  ↓ transform
component
  ↓ transform
component
  ↓ filter
component
  ↓ map
component
  ↓ transform
Plotly
```

Compute expensive representations deliberately.

Know who owns them.

Know who references them.

Know when they become obsolete.

## So why not class-based React?

Not because classes are old.

Not because functional programming defeated object-oriented programming.

Not because `function` is somehow more fashionable than `class`.

The reason is architectural:

> **Modern React is based around render snapshots, explicit synchronization, composable stateful behavior, and work that may be recreated or discarded. Functional components map naturally onto that execution model. Persistent mutable component instances do not.**

Classes remain useful tools.

OOP remains useful.

But choosing class-based React because "backend engineers prefer object-oriented programming" is choosing an architecture based on familiarity rather than fit.

The frontend deserves a model appropriate to the system it is running.

In modern React, that model is overwhelmingly the function.
