# Signals In Angular — Is RxJS Doomed?

Using Signals in Angular can simplify the coding experience. Reactivity with Signals could lead to better change detection.

The Angular team is finally releasing the first comments and prototypes regarding bringing Signals, a new reactive primitive, to the core framework.

A lighthouse emitting a Signal. AI-generated image.

What You Should Know: TL;DR:

- To some extent, Signals is powerful like RxJS but with a simpler syntax.
- If implemented, it might change the way change detection work. Possibly, removing zone.js.
- Signals are not going to replace RxJS any time soon

# What Are Signals?

> A Signal is a source of value
> Alex Rickabaugh

Signals have been popularized recently by [Solid.js](https://www.solidjs.com/) which [describes](https://www.solidjs.com/tutorial/introduction_signals) them as follows:

*Signals are the cornerstone of reactivity in Solid. They contain values that change over time; when you change a signal’s value, it automatically updates anything that uses it.*

If you are familiar with RxJS, this might remind you of [BehaviorSubject](https://rxjs.dev/api/index/class/BehaviorSubject) or, more generally, multicasted values. However, unlike BehaviorSubject, Signals would not require the Observers to subscribe to be notified of changes.

With Signals, Subscriptions get created and destroyed automatically under the hood. More or less what happens using the async pipe in RxJS. However, unlike Observables, Signals don’t require a Subscription to be used outside the template.

Signals go to great lengths to hide some of the complexities of RxJS.

As said above, Signals contain values that change over time. So, let’s have a look at that.

## Signals Example

The following example starts from [Introduction/Signals](https://www.solidjs.com/tutorial/introduction_signals) and shows how the `count` value changes over time and gets reflected in the DOM.

We will then see how it could be implemented in Angular.

Let’s start by creating a Signal.

1. Create a Signal, by importing `createSignal` from `solid-js`.
2. Pass an argument to `createSignal`. This is the initial value.
3. `createSignal`returns an array with two functions, a getter and a setter.
4. *By* [*destructuring*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)*, we can name these functions whatever we like. In this case, we name the getter* `*count*` *and the setter* `*setCount*`*.*

```
const [count, setCount] = createSignal(0);
```

It is interesting to notice the similarities with React!

The syntax reminds a hook and it works pretty much in the same way.

The similarities don’t end here. Solid’s general approach consists in wrapping reactive computations in functions and it uses JSX.

Here is how it plays out:

```
import { createSignal} from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);

  return <div>Count: {count()}</div>;
}
```

If you are familiar with React, you might notice that we are calling `*count()*` as a function in the template*.* This makes a lot of sense because *the first returned value is a getter (a function returning the current value) and not the value itself.* Therefore, it has to be evaluated by the JSX compiler.

By using JavaScript `setInterval` we create a periodically incrementing counter that is reflected in the UI.

```
import { createSignal } from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);
  setInterval(() => setCount(count() + 1), 1000);

  return <div>Count: {count()}</div>;
}
```

So far, it is not a big deal, but let’s expand the example to see how anything that uses Signals gets updated automatically.

We add the following line in Counter:

```
const doubledCount = () => 2 * count();
```

We can then use `doubleCount` in the template as follows:

```
return <div>Count: {count()}, Double: {doubledCount()}</div>;
```

You will notice how `doubleCount()` is automatically updated and reflected in the DOM.

This is pretty cool because it simplifies the developer experience making the code more reactive.

# Signals In Angular — Example

Let’s see an example of how it could look in Angular.

Be aware that if you want to use Signals in Angular, you should use [Angular v16.0.0-next.0](https://github.com/angular/angular/releases/tag/16.0.0-next.0) which has just been released.

Needless to say, it is still a prototype and you may not want to use it in production:)

Another way to try it out is by using [StackBlitz](https://stackblitz.com/edit/angular-ednkcj?file=src%2Fmain.ts) and importing the signal folder.

Here is what Signals in Angular could look like:

```JS
...

@Component({
  selector: 'signals-not-noise',
  standalone: true,
  template: `
    <div>Count: {{ count() }}</div>
    <div>Double: {{ doubledCount() }}</div>
  `
})
export class App {
  count = signal(0);
  doubledCount = computed(() => this.count() * 2);

  constructor() {
    setInterval(() => this.count.set(this.count() + 1), 1000);
  }
}
```

Since `doubleCount` uses `count` that is a Signal, `doubleCount` gets updated every time `count` changes.

In other words, `doubleCount` depends on, or is computed from, `count` and this dependency is reflected automatically whenever the latter changes.

Thanks to `computed` we can take the value of other Signals and build on them. Moreover, it is memorized, so it doesn’t need to run the computation every time you need the value, but only when it changes.

This leads to fine-grained reactivity that could support better change detection.

## Function Calls In Angular Templates

Are we really going to use function calls on Angular templates!?

Isn’t that bad?

**Short answer:** It can be bad.

**Longer answer**: [It is not so bad if the evaluation is cheap](https://www.youtube.com/live/ov0rSrZT4lk?feature=share&t=2225) and Signals are going to be cheap.

Not calling functions in templates is a general rule that prevents something that can be a problem rather than something that always is a problem.

If the function you call in the template executes expensive operations, then you might have problems. The change detection system has to re-evaluate the function at any change, and expensive operations can pile up.

You can read more in the following post by 

[Enea Jahollari](https://medium.com/u/eb2c15cb2c38?source=post_page-----5b5dac574306--------------------------------)

.

It’s ok to use function calls in Angular templates!

“You should never use function calls on Angular templates!” — That’s what you will see all over the internet! And I’m…

itnext.io

## What Is Wrong With RxJS?

Using RxJS we would need to do something like this:

```js
import { Component } from '@angular/core';
import { BehaviorSubject, map, take } from 'rxjs';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  count$ = new BehaviorSubject(0);
  doubleCount$ = this.count$.pipe(map((value) => value * 2));

  constructor() {
    setInterval(() => {
      let currentCount = 0;
      this.count$.pipe(take(1)).subscribe((x) => (currentCount = x));
      this.count$.next(currentCount + 1);
    }, 1000);
  }
}
```

Which also implies using the[ async pipe](https://medium.com/better-programming/go-reactive-with-angular-async-pipe-b290988f4000) in the template. Some waterfall effects include:

- Having to manage a null value until the async request completes and returns another value
- Using the async pipe several times creates separate Subscriptions and separate HTTP requests. You may want to use [shareReplay](https://www.learnrxjs.io/learn-rxjs/operators/multicasting/sharereplay) to avoid multiple side effects then.

As shown by 

[Maxime ROBERT](https://medium.com/u/27d6942c6fa4?source=post_page-----5b5dac574306--------------------------------)

 in his comment (which I suggest you read), you could simplify the RxJS part as follows:



```
constructor() {
    setInterval(() => {
      this.count$.next(this.count$.value + 1);
    }, 1000);
}
```

## Signals and RxJS

It might seem Signals is going to take over and replace RxJS.

However, Signals don’t handle race conditions very well, so RxJS is going to stay and support asynchronous operations.

It is also possible to turn SIgnals into Observables and vice versa.

You can watch an explanation by Joshua Morony.

<iframe src="https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fwww.youtube.com%2Fembed%2F4FkFmn0LmLI%3Ffeature%3Doembed&amp;display_name=YouTube&amp;url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3D4FkFmn0LmLI&amp;image=https%3A%2F%2Fi.ytimg.com%2Fvi%2F4FkFmn0LmLI%2Fhqdefault.jpg&amp;key=a19fcc184b9711e1b4764040d3dc5c07&amp;type=text%2Fhtml&amp;schema=youtube" allowfullscreen="" frameborder="0" height="480" width="854" title="Angular is about to get its most IMPORTANT change in a long time..." class="fc n id dh bf" scrolling="auto" style="box-sizing: inherit; top: 0px; width: 680px; height: 382.188px; left: 0px;"></iframe>

[Angular is about to get its most IMPORTANT change in a long time](https://www.youtube.com/watch?v=4FkFmn0LmLI&ab_channel=JoshuaMorony)

Using Signals in Angular might facilitate learning Angular, at least in the beginning.

# New Change Detection

This could be a big change for Angular.

Thanks to Signals, we can see a future without zone.js.

Signals allow checking changes at a more granular level rather than checking the whole component tree for changes. In turn, this could be used to improve Angular change detection.

At the moment, different frameworks check what changes in the DOM tree at different levels, for example at

- App tree level — Angular
- Component tree level — React
- Individual elements — Solid

It seems like Angular aims at becoming more granular, and in doing so, they plan to keep Signals interoperable with zone.js.

According to Alex Rickabaugh (the same person behind [The Little-known Story Behind Angular Standalone Components](https://medium.com/javascript-in-plain-english/the-little-known-story-behind-angular-standalone-components-2392abc4fcfa) and Technical Lead in the Angular Core Team), you wouldn’t need `ngOnChange` in a Signal world.