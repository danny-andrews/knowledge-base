```jsx
<WrappedComponent {...this.props}/>
// is equivalent to
React.createElement(WrappedComponent, this.props, null)
```

– https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#5247

---

**Regular expression to match a string **not** containing a character sequence: `^((?!WORD_1).)*$`**

– https://stackoverflow.com/questions/406230/regular-expression-to-match-a-line-that-doesnt-contain-a-word

---

**When to use quotes in YAML (why is this so complicated?)**

In general, you don't need quotes. However, you should use them...
* to force a string, e.g. if your key or value is 10 but you want it to return a String and not a Fixnum, write '10' or "10".
* if your value starts with special characters: (`:`, `{`, `}`, `[`, `]`, ,`,`, `&`, `*`, `#`, `|`, `-`, `>`, `=`, `!`, `%`, or `@`)
* if your value ends with special characters: (`:` or `#`)
* "`Yes`" and "`No`" should be enclosed in quotes (single or double) or else they will be interpreted as `TrueClass` and `FalseClass` values

> Single quotes let you put almost any character in your string, and won't try to parse escape codes. `'\n'` would be returned as the string `\n`.

> Double quotes parse escape codes. `"\n"` would be returned as a line feed character.

> Seems to me that the best approach would be to not use quotes unless you have to, and then to use single quotes unless you specifically want to process escape codes. **Yeah, but then I have to consult this list every time!** 😡😡😡

---

**ASI Edge-Cases**

> Function return statement

```js
function a() {
  return
  "Nope, I am sorry!"
}

a() // undefined
```

> Line starting with parenthesis

```js
const b = 1
(function() { console.log('Old school module pattern!') })()

// Uncaught TypeError: 1 is not a function
```

> Line starting with angle brackets

```js
const squared = []
[1, 2].forEach(i => c.push(i * 2))

// Uncaught TypeError: Cannot read property 'map' of undefined
```

– https://maurobringolf.ch/2017/06/automatic-semicolon-insertion-edge-cases-in-javascript/

---

You can use cherry-pick a range of commits!

---

You can rename branches! `git branch -m NEW_BRANCH_NAME`

---

How to deal with conflicts in yarn.lock files:
* git checkout BASE_BRANCH -- yarn.lock
* yarn install

```

You can conditionally add elements/properties to Arrays/Objects:

```js
const cond = false;
const arr = [
  ...(cond ? ['a'] : []),
  'b',
];
```

```js
const cond = false;
const obj = {
  ...(cond ? {a: 1} : {}),
  b: 2,
};
```

---

You can omit properties with pure ES6
```js
const myObject = {
  a: 1,
  b: 2,
  c: 3
};
const { a, ...noA } = myObject;
console.log(noA); // => { b: 2, c: 3 };
```

---

Difference between TODO and FIXME comments
TODO: We should go back and improve this
FIXME: This hack MUST be fixed before going to production

---

JSON is NOT a subset of Javascript!
JSON allows the /u2028 and /u2029 in string literals, Javascript does not

– http://timelessrepo.com/json-isnt-a-javascript-subset

---

When generating scoped class names, using 6 digits is a good rule of thumb. this will give you 0.001% certainty of zero collisions as long as you don’t have more than 1,172 class names.

---

Functional core, imperative shell - A way to design systems which isolates side-effectful code from pure functional code. Impure functions should call pure functions, but not the other way around.
Functional core: handles decisions (lots of paths, little/no dependencies)
Imperative shell: handles dependencies (lots of dependencies, few paths)

---

Stateless function components cannot use pureRender (they are re-rendered whenever their parent changes, regardless of whether or not their props have changed.

Workaround:
```jsx
const pure = func =>
  class PureComponentWrap extends React.PureComponent {
    render() {
      return func(this.props, this.context);
    }
  }

const MyComponent = ({msg}) =>
  <span>{msg}</span>

const MyComponentPure = pure(MyComponent);
```

---

You can simulate objects in Elm!

```elm
import Html exposing (text)

point = \x y -> { multiply = \multiplier -> (x * multiplier, y * multiplier) }

showAge { age, name } = name ++ " is " ++ (toString age) ++ " years old"

z = { age = 21, name = "Shady" }

f = { id = \x -> x }

myPoint = point 1 3
(_, y) = myPoint.multiply 4

main = y |> toString |> text
```
