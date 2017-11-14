Disclaimer: This article is in no way meant to belittle the react-css-modules project, or its author. Gajus is a smart dude and does a lot of great work for free for the JavaScript community to enjoy! ❤️ This is merely an evaluation of this particular library.

As with many popular libraries, I'm sure "react-css-modules" had a valid use-case at the time it was created, but at the present, it's drawbacks far outweigh its benefits. This article is meant to be a warning against picking it up without thinking about what you get from it.

tl;dr: Use "css-loader" over "react-css-modules"/"babel-plugin-react-css-modules" because the latter relies on side-effects, adds cognitive overhead (too much 🦄 ), causes React errors in your tests, requires complex webpack config, requires an additional dependency, is slower than css-loader, and doesn’t work with webpack/babel `import` aliases.

## "PROs"[*](https://github.com/gajus/react-css-modules#whats-the-problem)
**Throws error when trying to use an `undefined` class name.**

Only actual pro IMHO.

---

**You don't have to use the styles object whenever constructing a className.**

Why is explicitness a bad thing?

---

**Mixing CSS Modules and global CSS classes is easy.**

https://gist.github.com/danny-andrews/19c09efbb71b8b452b2eea9bedee65d2

> Yeah, the second example is *slightly* more involved, but if you're using the fantastic (and tiny) [classnames](https://github.com/JedWatson/classnames) lib, you can simplify the "css-modules" example to:

https://gist.github.com/danny-andrews/a91b287169cd6bb4e985939f33266750

---

**Don't have to use `camelCase` CSS class names.**

Okay, sure, but the same is true for "css-loader." It has a [`camelCase`](https://github.com/webpack-contrib/css-loader#camelcase) option which converts kabab-case’ed class names to `camelCase`.

## CONs

**Relies on side-effects**

https://gist.github.com/danny-andrews/383d6e4a3d27575605ffd5721046e595

> Where does `"myClass"` come from? Why am I not using the `./styles.scss` import?
---

**Adds magic and cognitive overhead.**

What’s the difference between `className` and `styleName`? Why are there both?

---

**Causes [React errors](https://github.com/gajus/react-css-modules/issues?utf8=%E2%9C%93&q=unknown%20prop%20stylename%20) about unrecognized property name (`styleName`) for native DOM elements.**

---

**Requires pretty convoluted webpack config.**

https://gist.github.com/danny-andrews/bdb0b950e6f4a1c898f3b47a227edc6b#file-webpack-config-js

---

**Requires an additional dependency.**

You're already using "css-loader" if you are importing css in your app, which already works for css modules out of the box. Why add another dependency?

---

**Slower than using css-loader directly.**

https://github.com/gajus/babel-plugin-react-css-modules#performance

---

**Doesn’t work with webpack aliases (or [babel-plugin-module-resolver](https://github.com/tleunen/babel-plugin-module-resolver)) with [no plans to support](https://github.com/gajus/babel-plugin-react-css-modules/issues/46#issuecomment-307552410).**

---

**~~Generates random number for style map which causes changes to dist files even when there were no code changes.~~**

Fixed in [v2.8.0](https://github.com/gajus/babel-plugin-react-css-modules/commit/ab2fe0e0f1f7771a71af1acd5b36454f6b68b669).
