# React

## Component Design
Follow the Low of Demeter: keep component props flat. Don't make your components reach deep into nested objects to get the data it needs to render. This couples your view code with your underlying data structure. Components should have clear contracts, requiring only the minimal amount of information it needs to do its job.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**
This component has way to much knowledge about how we store user objects!

```js
// UserBadge.jsx
import React from 'react';

export default ({ user }) => (
  <div>
    <img src={user.avatar.imagePathLarge} alt="User image" />
    <span>Hello {user.firstName}!</span>
  </div>
);
```

**Good**

```js
// UserBadge.jsx
import React from 'react';

export default ({ avatarImagePath, displayName }) => (
  <div>
    <img src={avatarImagePath} alt="User image" />
    <span>Hello {displayName}!</span>
  </div>
);
```
</details>

## Terms
HOC - Higher-order component
Component decorator
PFC - Pure functional component

## Testing

**General Tips**
- Prefer shallow rendering - This speeds up your tests and prevents you from writing redundant tests

**Testing Components**

- Use factories (See [Testing section](GENERAL.md#Testing) for general rationale)
- Only test public API (rendered elements, action handlers)
- Assert collaborator components are given proper props
- [TODO: Add examples]

**Testing Higher-Order-Components (HOCs)**

[TODO]

**Testing Containers**

[TODO]

## Sharing Code/State
**[Higher-Order-Component (HOC)](https://reactjs.org/docs/higher-order-components.html)**

[TODO]

**[Render prop pattern](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)**

[TODO]

- [Function as Child Component](https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9)

  [TODO]

- [Function as Prop Component](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)

  [TODO]

- [Component Injection](http://americanexpress.io/faccs-are-an-antipattern/)

  [TODO]

** Gotchas **

- If you use the methods outlined below to avoid inlining callbacks in `render`, you may end up with stale closure variables. Thanks to @pspeter3 for [pointing this out](https://twitter.com/pspeter3/status/913423110862508032)!

  [Pen demonstrating issue](https://codepen.io/danny-andrews/pen/VMzbwW)

SICK AWESOME DOPE COMPARISON SHOWING WHICH USAGE PATTERNS CAUSE EXTRA RENDERS: https://codepen.io/danny-andrews/pen/LzyRvO.

[TODO: Add example pen]

## Performance
**Optimize component rendering**

By default, React components (class [and functional](https://github.com/facebook/react/issues/5677#issuecomment-241190513)) always re-render, even if the props passed in are identical to those previously given. Sometimes this ends up being faster, since you don't have the [overhead](https://github.com/facebook/react/issues/5677#issuecomment-165457596) of comparing props on every render, but for components on hot code paths (or high in the tree) this can negatively impact performance [TODO: Add link to real-world benchmark demonstrating this]. Luckily, there are workarounds.

- For class methods, inherit from [`PureComponent`](https://reactjs.org/docs/react-api.html#reactpurecomponent) - This defines a `shouldComponentUpdate` hook which shallowly compares props and state (so don't be mutating those!)
- For PFCs, you can either use [recompose#pure](https://github.com/acdlite/recompose/blob/master/docs/API.md#pure) or [this little snippet](https://github.com/facebook/react/issues/5677#issuecomment-280295107):

  ```js
  function pure(func) {
    class PureComponentWrap extends React.PureComponent {
      render() {
        return func(this.props, this.context)
      }
    }
    return PureComponentWrap
  }
  ```

**Avoid using `bind` or inlining function definitions in `render` <sup>[*](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)</sup>**

Both of these patterns result in a new function being created each time a component. This results in
1. Extraneous garbage collection invocations which can slow performance [TODO: Add link to real-world benchmark demonstrating this]
2. Extra re-renders in components which define `shouldComponentUpdate`.

[TODO: Add Good and Bad examples]

[TODO: Add section describing workarounds for both class and functional components]

## Organization
- Directory structure

## + Redux
- Separate components from containers

Crisp Reading:
- http://mxstbr.blog/2017/02/react-children-deepdive/
- https://cdb.reacttraining.com/react-inline-functions-and-performance-bdff784f5578#9ae9

# CSS
- Creating consistent spacing: https://medium.com/fed-or-dead/handling-spacing-in-a-ui-component-library-70f3b22ec89
- Creating consistent headings: https://medium.com/fed-or-dead/handling-headings-in-a-ui-component-library-2587de93c890

# Npm

## Modern Packaging
http://2ality.com/2017/04/setting-up-multi-platform-packages.html#main-native-features-cjs
https://github.com/rollup/rollup/wiki/pkg.module

## package.json schema
* Always write three digits (e.g. don’t do `1.0`)
* Prefer `~` and `^` to any other types of ranges
* If you need a more complex range, prefer the `-` syntax
* If you need an X-Range, use `x` as the character, not `X`

# JavaScript

## Object Creation Pattern

Douglas Crockford recommends a pattern for creating objects in a functional way. The basic idea is to have a factory function which acts as a constructor. Now new. No class. No inheritance. But easy composition/mixin potential. This pattern allows for true private members. I dub this pattern "Crock-Pot" patterns, because you but good things in, and get something better out! And you freeze it for later! Example:

```js
const Reader = value => {
  const run = config => value(config);
  const map = fn => Reader(config => fn(run(config)));
  const chain = fn => Reader(config => fn(run(config)).run(config));

  return Object.freeze({ map, chain });
}
```
