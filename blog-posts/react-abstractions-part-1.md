# React Abstractions: Part 1 ("What" and "How")

## Background
If you're a part of the web-dev Twittersphere, then you've probably heard of the component enhancer vs. render prop "drama" (😛) created by @mjackson's tweet:
https://twitter.com/mjackson/status/885910701520207872.

> If you are unfamiliar with the concept of render props and/or haven't read his article on the subject, I would highly recommend doing so [here](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) before continuing this article.

This tweet sparked a lot of productive debate on the relative merits of render props vs component enhancers. However, by the end of my exploration down this rabbit hole, I felt very much like Alice at the end of the Hatter's tea party: stimulated but ultimately just confused.

## Purpose
This set of articles is an attempt to organize my thoughts on the matter and hopefully help others along their journey through the wonderful world of React 🐰. Part 1 will be concerned with the "What" and "How" of these abstractions, and part 2 will be concerned with the "When" and "Why."

## Definitions (a.k.a. "What")
- [Render (prop/callback)](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) - A function prop that a component uses to know what to render. Example: https://gist.github.com/danny-andrews/e214e127c9610b54dea735b932edf396
	- [Function as Child Component (FaCC)](https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9) - Really just a special case of the render prop where it is passed via `children`. Example:
https://gist.github.com/danny-andrews/a0af4993396a27a5eadc1234c6087ec4
	- [Component Injection](http://americanexpress.io/faccs-are-an-antipattern/#component-injection---a-better-solution): Another special case of the render prop pattern where instead of inlining the render function, you pass a previously-defined component instead. Example: https://gist.github.com/danny-andrews/d8bc8640ab125d2b5857070d6e472c47
- [Component Enhancer](https://reactjs.org/docs/higher-order-components.html) - A function which takes a component as an argument and returns a new component, which wraps the old one and endows it with some new functionality. Commonly, although incorrectly, known as a "higher-order component" (HoC). Example: https://gist.github.com/danny-andrews/340660e5f76b962f7698927d182b080a
	- [Inheritence Inversion](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#5247) - A special case of the HoC pattern whereby the returned (enhanced) component extends the wrapped component. Example: https://gist.github.com/danny-andrews/899a6ca064e659941c031f6276f5c02e
- Higher-order component (HoC) - Any component which takes a render prop. So in our examples, the `WindowWidth` component would be a HoC.

> The term "higher-order component," as it's currently used, is inaccurate because the thing it refers to is not itself a component, but a function. That's why I call them "component enhancers" instead.

> Higher-order **function** (HoF): A **function** that takes a function as an argument and/or returns a function as a result. ([ref](http://wiki.c2.com/?HigherOrderFunction))
> Higher order **component** (HoC): A **component** that takes a component as an argument (i.e. prop).
> Component enhancer: A **function** that takes a component as an argument and returns a component as a result.

## How?
https://github.com/facebook/react/pull/10741#discussion_r139836136
https://github.com/facebook/react/pull/10741#discussion_r139894711

## Unanswered Questions (Hopefully to be Answered in Part 2!)
1. How do HoCs and render props compare as far as performance is concerned?
1. Should we be concerned about inlining render callbacks from a garbage collection standpoint?
1. Should we be concerned about inlining render callbacks?
1. Can HoCs be composed as elegantly as component decorators? Should they be?
1. Are HoCs easier or more difficult to test than component decorators?

References:
https://reactjs.org/docs/higher-order-components.html
https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce
https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9
https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce
http://americanexpress.io/faccs-are-an-antipattern/
https://twitter.com/pspeter3/status/913423110862508032
https://codepen.io/danny-andrews/pen/LzyRvO
