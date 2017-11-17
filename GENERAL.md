# Testing

## Terminology
**Factory** - A function which simplifies the construction of some test dependency (arguments, objects, etc.).  
**Fixture** - Any form of static test data.  
**Test-Driven Development (TDD)** - A programming discipline which emphasises writing tests as a part of code design, as easy of testing is a good indicator of code quality.  
**Behavior-Driven Development (BDD)** - A approach to writing tests which focuses on specification and user-facing behavior.  
**Unit/System Under Test (SUT)/Test Subject** — The simplest piece of self-contained functionality. It could be as simple as a method, or as complex as a class, but should be isolated sufficiently from collaborators.  
**Test Case** - One atomic test (usually implemented as a function) which runs some code and makes assertions about it.  
**Test Double[\*](http://xunitpatterns.com/Test%20Double.html)** - A generic term for any kind of pretend object used in place of a real object for testing purposes. Specific examples given below:

* **Fake** — A test double that actually has a working implementation, but usually takes some shortcut which makes it not suitable for production (an [in-memory database](https://martinfowler.com/bliki/InMemoryTestDatabase.html) is a good example, as is [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store)).
* **Dummy** - A test double passed around but never actually used in the code path the test is exercising. Usually they are just used to fill parameter lists.
* **Stub** - A test double which provides canned answers to calls made during the test.
* **Spy** - A Stub that also records some information based on how it was called (how many times and with what parameters).
* **Mock[\*](https://martinfowler.com/articles/mocksArentStubs.html)** - A spy with pre-programmed expectations.

## Best-Practices

#### Keep irrelevant data and setup out of tests
Use factories to make SUT construction easy.

**Benefits**
1. No copy-and-paste.
1. Tests which are easier to read.
1. Easy to fix entire suite when the SUT's signature changes.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
import subject from '../get-pr-status-payload';

it("sets state to 'success' when no failures", () => {
  const { state: actual } = subject({
    thresholdFailures: [],
    label: '' // Why do we have to pass this in? We don't care about label.
  });

  expect(actual).toEqual('success');
});

it("sets state to 'failure' when there are failures", () => {
  const { state: actual } = subject({
    thresholdFailures: [{ message: 'file3 is too big' }]
    label: ''
  });

  expect(actual).toEqual('failure');
});

it('sets context to label', () => {
  const { context: actual } = subject({
    thresholdFailures: [], // Why do we have to pass this in?
    label: 'bundle sizes'
  });

  expect(actual).toEqual('bundle sizes');
});
```

**Good**

```js
import getPrStatusPayload from '../get-pr-status-payload';

const optsFac = (opts = {}) => ({
  thresholdFailures: [],
  label: '',
  // If the SUT adds any required options (e.g. delay) we can add it here, and
  //   fix all our tests!
  ...opts
});

const subject = R.pipe(optsFac, getPrStatusPayload);

it("sets state to 'success' when no failures", () => {
  const { state: actual } = subject({ thresholdFailures: [] });

  expect(actual).toEqual('success');
});

it("sets state to 'failure' when there are failures", () => {
  const { state: actual } = subject({
    thresholdFailures: [{ message: 'file3 is too big' }]
  });

  expect(actual).toEqual('failure');
});

it('sets context to label', () => {
  const { context: actual } = subject({ label: 'bundle sizes' });

  expect(actual).toEqual('bundle sizes');
});
```
</details>

#### Factories only provide *required* data[\*](https://robots.thoughtbot.com/factories-should-be-the-bare-minimum)
Factories should only provide minimal amount of data required to staisfy the SUT's interface.

**Benefits**
1. No confusing test results caused by optional data added by factory.
1. No unnecessary work performed by factory.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
import Person from '../Person';

const subject = (opts = {}) => ({
  name: 'Bob',
  favoriteNumbers: [2, 4, 7], // Optional required!
  ...opts
});

it('defaults favoriteNumbers to empty list', () => {
  const person = subject();

  expect(person.favoriteNumbers.length).toBe(0); // Fails! Length is 3!
});
```

**Good**

```js
import Person from '../Person';

const subject = (opts = {}) => ({
  name: 'Bob',
  ...opts
});

it('defaults favoriteNumbers to empty list', () => {
  const person = subject();
  
  expect(person.favoriteNumbers.length).toBe(0); // Fails! Length is 3!
});
```
</details>

#### Apply BDD Principles
Test external-facing behavior, not implementation. Don’t test private APIs.

![Unit testing cheatsheet](public/unit-testing-cheatsheet.png)

**Benefits**
1. Allows you to refactor SUT internals without breaking tests.
1. Tests what's really important (user-facing behavior). Tests SUT internals as a side-effect.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
it('tick increases count to 1 after calling tick', function() {
  const subject = Counter();

  subject.tick();

  // Tests two things! That count is incremented when tick is called, and that
  //   count is defaulted to 0. In the context of this test, the latter is an
  //   implementation detail.
  assert.equal(subject.count, 1);
});
```

**Good**

```js
it('increases count by 1 after calling tick', function() {
  const subject = Counter();
  const originalCount = subject.count;

  subject.tick();

  // Only tests one thing.
  assert.equal(subject.count, expectedCount + 1);
});
```
</details>

#### Each test should only verify one behavior

Verifying one behavior != making one assertion, although this is usually the case.

**Benefits**
1. Eliminates test redundancy.
1. Makes it easier to remove/modify a behavior from the SUT, as it should require deleting/modifying one corresponding test.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
it('tick increases count to 1 after calling tick', function() {
  const subject = Counter();

  subject.tick();

  // Tests two things! That count is incremented when tick is called, and that
  //   count is defaulted to 0.
  assert.equal(subject.count, 1);
});
```

**Good**

```js
it('defaults count to 0', function() {
  const subject = Counter();

  subject.tick();

  assert.equal(subject.count, 0);
});

it('increases count by 1 after calling tick', function() {
  const subject = Counter();
  const originalCount = counter.count;

  subject.tick();

  assert.equal(subject.count, expectedCount + 1);
});
```
</details>

#### Avoid fixtures
Fixtures are inflexible and necessitate a lot of redundancy. If the shape of your data changes, you have to change it in **every single fixture**. Use factories instead. Create all data/objects your test needs inside the test.

#### Prefer BDD syntax
Many [testing frameworks](https://mochajs.org/#interfaces) offer an xUnit-style syntax (`suite`/`test`) and a BDD-style syntax (`describe`/`it`). The latter helps to nudge the developer in the direction of testing in terms of specifications and external behaviors rather than implementation details and read a little more naturally.

**Good**

```js
describe('thing', () => {
  it('does the thing');
});
```

**Less good**

```js
suite('thing', () => {
  test('does the thing');
});
```

#### Keep your test cases flat

1. Only use `describe` blocks to broadly categorize tests (e.g. `describe('performance')`, `describe('integration')`) never to group tests by conditions (e.g. `describe('with 1000 elements')`, `describe('when request fails')`).
1. Keep nesting to two levels deep.
1. Don't add top-level describe block. It adds unnecessary nesting. It should be clear what you are testing by the test file name.

**Benefits**
1. Easier to read test cases as all conditions are listed in the description.
1. Discourages nested `beforeEach` logic which makes test cases exponentially more difficult to reason about.
1. Reduces indentation.

**Costs**
1. Test descriptions are longer. (Big deal.)
1. More duplication in a test case since you can't do setup in a `beforeEach`. (Some duplication in tests is fine and ancillary boilerplate can be extracted into a helper method/factory.)

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
describe('requestMaker', () => {
  describe('valid token given', () => {
    it('uses authorization header along with passed headers', () => {
      // ...
    });
  });

  describe('expired token given', () => {
    it('generates a new token', () => {
      // ...
    });
  });
});
```

**Good**

```js
// request-maker-test.js
it('uses authorization header along with passed headers when valid token given', () => {
  // ...
});

it('generates a new token when expired token given', () => {
  // ...
});
```
</details>

#### Use common set of variable names
You will find yourself setting many variables which do the same thing in tests. Why try to come up with creative names for them, when you can just use one from a pre-defined set? Examples: `subject`, `expected`, `result`, `actual`, etc.

**Benefits**
1. Reduces cognitive load because naming things is hard.
1. Allows you to rename your subject without changing a bunch of variable names (or, even worse, forgetting to change them, leaving them around to confuse future readers).

#### Omit `it` and `should` in test description

These are implied, and removing them keeps descriptions short.

<details>
  <summary>Code Example:</summary>
  </br>

**Bad**

```js
it('should call handler with event object', () => {
  // ...  
});
```

**Good**

```js
it('calls handler with event object', () => {
  // ...
});
```
</details>

#### Keep all data local to test cases

Don't rely on [shared state](https://robots.thoughtbot.com/lets-not) set up in a `before`/`beforeEach` blocks. This makes your tests easier to [reason about](https://robots.thoughtbot.com/mystery-guest) and minimizes the possibility of flaky tests.

> If you're tempted to share state for performance reasons, don't do so until you have actually identified a problematic test.[\*](http://wiki.c2.com/?PrematureOptimization) Also realize that sharing state between tests eliminates the possibility to parallelize your tests, so you may end up with slower overall test run times by introducing shared state.

an instance variable defined at the top of a long context block, or nested up multiple context blocks

Mystery Guest - "The test reader is not able to see the cause and effect between fixture and verification logic because part of it is done outside the Test Method."

#### Organize tests into [stages](https://robots.thoughtbot.com/four-phase-test)
1. Setup/Arrange
1. Exercise/Act
1. Verify/Assert
1. (Teardown)*

\* Requiring teardown in a test is a code-smell and is usally the result of stubbing a global method which should be passed in as a dependency.

<details>
  <summary>Example (Using React + Enzyme):</summary>

```jsx
it('renders stat and label of currently selected datum', () => {
  // Setup/Arrange
  const data = [
    { x: 'Cats', y: 2 },
    { x: 'Dogs', y: 17 },
  ];

  // Exercise/Act
  const root = mount(<DoughnutChart data={data} />);
  const subject = root.find('VictoryPie').find(HighlightableSlice).at(1);
  subject.simulate('mouseover');

  // Verify/Assert
  const stat = root.find('.highlight-stat').text();
  const label = root.find('.highlight-label').text();
  expect(stat).toBe('17');
  expect(label).toBe('Dogs');
});
```
</details>

## Benefits of following these principles
* Better code (no reliance on global state)!
* Determinism
* Atomicity (no order-dependence)
* Parallelization potential
* Focus
* Robustness
* Readability
* Reasonableness
* Less cognitive load

Resources:
https://martinfowler.com/articles/mocksArentStubs.html
https://robots.thoughtbot.com/four-phase-test
https://robots.thoughtbot.com/lets-not
https://robots.thoughtbot.com/factories-should-be-the-bare-minimum
https://robots.thoughtbot.com/mystery-guest
https://www.youtube.com/watch?v=R9FOchgTtLM
http://xunitpatterns.com/Test%20Double.html

# Error Handling
**Store error messages in external file (don't be afraid to be verbose). Allow string interpolation via sprintf.**

# Writing Better Functions[\*](https://www.youtube.com/watch?v=T8J0j2xJFgQ)
1. Gather Input
1. Perform Work
1. Deliver Results
1. Handle Errors

# Variable/Function Naming
* Good names
  1. Reveal the intention (http://wiki.c2.com/?IntentionRevealingNames, http://wiki.c2.com/?IdentifiersRevealIntent, http://wiki.c2.com/?IntentionRevealingSelector)
  1. Hide the implementation
  1. Give a clear hint as to the expected return type
* Name a method after *what* it does, not *how* it does it or *why* (i.e. name a method after its reason for existence, or its "essence")
* Examples:

Good identifier: `sortVectorIntoAlphaOrder(vector & v)`

Bad identifier: `collateCustomerNames(vector & v)`
  - To test a name: Imagine a second very different implementation. Will the name work for both implementations?
	- Booleans
	- Constants
  "name services after what they do, and postpone the why until the point of usage."

# Exceptions
Stop throwing them. Stop using them for control flow.[\*](https://www.joelonsoftware.com/2003/10/13/13/)

# Git

## Branching Strategy
[TODO: Add note for app code]
[TODO: Add note for library code]

## Merging Strategies
[TODO: Always rebase? Always create merge commit?]

**Branch Name Format**
In shared repos, it's helpful to prepend your branch names with your name, so you know which branch belongs to which dev (e.g. `dla/my-cool-feature`).

## Commit Message Format
* Limit subject line to 50 characters.
* Separate subject from body with blank line.
* Capitalize subject line.
* Do not end subject line with a period.
* Use imperative mood in subject line.
* Hard-wrap body at 72 characters.
* Use body to explain *what* and *why*, not *how*
Reference any fixed/related issues at the end of the body, following a blank line. Example (Refs/Fixes #532).
* Prepend subject with a pre-defined tag. (This is especially nice for generating change logs.)[\*](https://eslint.org/docs/developer-guide/contributing/pull-requests#step-2-make-your-changes)
  * Fix - Bug fix.
  * Update - Backwards-compatible enhancement.
  * New - New feature.
  * Breaking - Backwards-incompatible enhancement or feature.
  * Docs - Change to documentation only.
  * Build - Change to build process only.
  * Upgrade - Dependency upgrade.
  * Chore - Refactoring, adding tests, etc. (anything that isn’t user-facing).

Example:
```
Fix: Improve chart rendering performance.

Our charting dashboard was starting to lag with more than 200 data
points. This changeset fixes that and adds performance tests to prevent
regressions.

Fixes #926.
```

# Workflow

1. Write broken test
1. Write code to fix test
1. Refactor
1. Rename
1. Make commit
1. Squash commits
