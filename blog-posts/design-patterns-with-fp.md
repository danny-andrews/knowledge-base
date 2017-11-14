# OOP Design Patterns - Inheritance + FP = ???

Over the last decade, the software industry has slowly been moving away from classical-inheritance in favor of more compositional patterns (mixins in Ruby, traits in Scala, etc.) and its begs the question: what happens to the OOP design patterns we've all come to know and love (or hate 😬) in a post-inheritance world? Additionally, what happens to these patterns when we have the full power of functional programming (first-class functions, partial application, monads, etc.) at our disposal?

These are the questions this article attempts to answer. So let's take another look at how the classical OOP design patterns patterns manifest themselves (if at all) in a functional programming language with support for objects. Although several superior languages fit this bill (OCaml, Lisp, etc.) my code examples are going to be in JavaScript, because of its approachable syntax and large user-base.

> In my examples, I'll be using the object creation technique simliar to the one outlined [here](https://medium.com/javascript-scene/javascript-factory-functions-with-es6-4d224591a8b1). Reason: no need for `new` or `this` and no confusing `prototype` behavior. Just functions which return objects.

> For an explanation of patterns, and their classical OOP solutions, head [here](https://github.com/kamranahmedse/design-patterns-for-humans). Or to the [source](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8).

## Conditions
\- Class-based inheritance  
\+ First-class functions  
\+ Partial function application (currying)  
\+ Monads

## Terms
* Lambda - A function used as data (usually synonymous with anonymous function).
* Client - The consumer of the pattern artifact.
* Object Composition - "The act of combining component pieces to form a new object." [\*](https://medium.com/javascript-scene/the-open-minded-explorer-s-guide-to-object-composition-88fe68961bed).

### 🚫 ~~Builder~~

⛓ Purpose: Break complex construction into multiple steps. Prevents constructor telescoping:

```js
const Burger = (size, cheese = true, tomato = false, lettuce = true) => ({
  /* ... */
});

const myBurger = Burger('big', true, true); // What do these params mean?
```

💡 OOP Solution: Break out construction process into steps (e.g. `addCheese`, `addTomato`).  
🔥 OOP + FP Solution: At first blush, it would seem this pattern seems contrary to FP principles. It's analogous to building a value via assignment rather than using function composition. Also, having many params for a constructor in the first place is a code smell, and we shouldn't accept patterns which cover up code smells (aka, Febreze Patterns). Possible alternatives:
1. Use keyword arguments.

<details>
  <summary>Code Example</summary>

```js
const Burger = ({ size, cheese = true, tomato = false, lettuce = true }) => ({
  /* ... */
});

const cheeseburgerWithTomato = Burger({ size: 'big', tomato: true });
```
</details>
<br/>

2. Use currying.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const Burger = R.curry(
  (size, cheese, tomato, lettuce) => ({
    /* ... */
  })
);

const BigBurger = Burger('big');
const BigCheeseBurger = BigBurger(true);
const bigCheeseBurgerWithTomato = BigCheeseBurger(true, false);
```
</details>
<br/>

3. Consider breaking your object apart as it probably has too many concerns.

### 🚫 ~~Singleton~~

⛓ Purpose: Share a global instance. Ensure there is at most one instance of a given object (e.g. a database).  
💡 OOP Solution: Global variables. 😵 (Specifically, a static field on a class which holds the singleton instance.) Alternatively, use some heavyweight dependency injection framework.  
🔥 OOP + FP Solution: Use the [`Reader`](https://monet.github.io/monet.js/#reader) monad.

<details>
  <summary>Code Example</summary>

```js
import {Reader} from 'monet';
import R from 'ramda';

const updateProfile = (id, attributes) => Reader(
    ({ db }) => db.update('profile', id, attributes)
  );

const getUser = id => Reader(({ db }) => db.get('user', id));

const main = () => getUser(5)
  .chain(user => updateProfile(user.profileId, { catchprase: 'Phrasing!' }))
  .map(R.prop('catchprase'))
  .map(console.log);

const MyDB = () => {
  const data = {
    user: {
      5: { name: 'Sterling Archer', profileId: 34 }
    },
    profile: {
      34: { catchprase: 'Bazinga!' }
    }
  };

  const get = (type, id) => data[type][id];
  const update = (type, id, attributes) => data[type][id] = {
    ...data[type][id],
    ...attributes
  };

  return { get, update };
}

main().run({
  db: MyDB({ username: 'fj3hro3', password: 'secret' })
});

// Ouptut:
// Phrasing!
```
</details>

### 👻 Visitor

⛓ Purpose: Enable adding further operations to objects without having to modify them.  
💡 OOP Solution: Create objects which perform some operation on the subject.  
🔥 OOP + FP Solution: Create functions which perform a `map` operation with pattern-matching on the "visited" type.

<details>
  <summary>Code Example</summary>

```js
const BookVisitee = ({ title, author }) => ({
  title,
  author,
  constructor: BookVisitee
});
const SoftwareVisitee = ({ title, name, url }) => ({
  title,
  name,
  url,
  constructor: SoftwareVisitee
});

const fancyVisitor = visitee => {
  // This would be handled via pattern matching in a statically-typed
  //   functional language.
  switch(visitee.constructor) {
  case BookVisitee:
    return `${visitee.title}...!*@*! written !*! by !@! ${visitee.author}`;
  case SoftwareVisitee:
    return `${visitee.title}...!!! made !*! by !@@! ${visitee.name}...www website !**! at http://${visitee.url}`;
  }
};

const plainVisitor = visitee => {
  switch(visitee.constructor) {
  case BookVisitee:
    return `${visitee.title}. written by ${visitee.author}`;
  case SoftwareVisitee:
    return `${visitee.title}. made by ${visitee.name}. website at ${visitee.url}`;
  }
};

const book = BookVisitee({
  title: 'Design Patterns',
  author: 'Gamma, Helm, Johnson, and Vlissides'
});
const software = SoftwareVisitee({
  title: 'Zend Studio',
  name: 'Zend Technologies',
  url: 'www.zend.com'
});

console.log('Plain description of book: ', plainVisitor(book));
console.log('Plain description of software: ', plainVisitor(software));
console.log('Fancy description of book: ', fancyVisitor(book));
console.log('Fancy description of software: ', fancyVisitor(software));

// Plain description of book:  Design Patterns. written by Gamma, Helm, Johnson, and Vlissides
// Plain description of software:  Zend Studio. made by Zend Technologies. website at www.zend.com
// Fancy description of book:  Design Patterns...!*@*! written !*! by !@! Gamma, Helm, Johnson, and Vlissides
// Fancy description of software:  Zend Studio...!!! made !*! by !@@! Zend Technologies...www website !**! at http://www.zend.com
```
</details>

### 👻 Command
⛓ Purpose: Encapsulate actions in objects. Decouple client from receiver.  
💡 OOP Solution: Create a `Command` object with an `execute` method. This can encapsulate the action.  
🔥 OOP + FP Solution: Use lambdas. Since functions are first-class citizens, there's no need to create an object to encapsulate an action.

<details>
  <summary>Code Example</summary>

```js
// Receiver
const Bulb = () => ({
  turnOn: () => console.log('Bulb has been lit'),
  turnOff: () => console.log('Darkness!'),
});

// Invoker
const RemoteControl = () => ({
  submit: command => command()
});

const bulb = Bulb();
const remote = RemoteControl();
remote.submit(bulb.turnOn);
remote.submit(bulb.turnOff);

// Output:
// Bulb has been lit!
// Darkness!
```
</details>

### 👻 Iterator
⛓ Purpose: Provide a way to access the elements of an object without exposing its underlying representation.  
💡 OOP Solution: Define an interface which iterable classes must implement.  
🔥 OOP + FP Solution: Prefer higher order functions (`forEach`, `map`, `reduce`) for iterating over objects.

<details>
  <summary>Code Example</summary>

```js
const RadioStation = frequency => ({ frequency });

const StationList = () => {
  let stations = [];
  let counter = 0;

  const addStation = station => { stations.push(station) };
  const removeStation = station => {
    stations = stations.filter(item => item !== station)
  };

  return {
    addStation,
    removeStation,
    forEach: (...args) => stations.forEach(...args)
  };
};

stationList = StationList();

const station89 = RadioStation(89);
stationList.addStation(station89);
stationList.addStation(RadioStation(101));
stationList.addStation(RadioStation(102));
stationList.addStation(RadioStation(103.2));
stationList.forEach(station => console.log(station.frequency));

// Output:
// 89
// 101
// 102
// 103.2

stationList.removeStation(station89);
stationList.forEach(station => console.log(station.frequency));

// Output:
// 101
// 102
// 103.2
```
</details>

### 👻 Strategy

⛓ Purpose: Enable an algorithm's behavior to be selected at runtime.  
💡 OOP Solution: Use inheritence/polymorphism.  
🔥 OOP + FP Solution: Use lambdas. Strategies are just objects with one method, so just pass the function directly instead of creating an unnecessary object.

<details>
  <summary>Code Example</summary>

```js
const derpSortStrategy = list => {
  console.log('Sorting using derp sort');
  return list;
};

const lolSortStrategy = list => {
  console.log('Sorting using lol sort');
  return [1, 0, 1];
};

const Sorter = sortStrategy => ({
  sort: list => sortStrategy(list)
});

const dataset = [1, 5, 4, 3, 2, 8];

derpSorter = Sorter(derpSortStrategy);
derpSorter.sort(dataset);

lolSorter = Sorter(lolSortStrategy);
lolSorter.sort(dataset);

// Output:
// Sorting using derp sort
// Sorting using lol sort
```
</details>

### 🎉 Factory

I combined the Abstract Factory and Factory Method patterns, since we are disallowing inheritance, and therefore, abstract classes.️

⛓ Purpose: Simplify object construction. Generate an instance for the client without exposing its instantiation logic.  
💡 OOP Solution: Build an abstract class with an abstract `create` method, and have child classes implement it.  
🔥 OOP + FP Solution: Firstly, all constructors are just factory functions, so in many cases, this pattern is eliminated. However, in times where you need polymorphism based on context, a simple factory-returning-function works quite well.

<details>
  <summary>Code Example</summary>

```js
const SoundPlayerFactory = hasAudioApi => hasAudioApi
  ? AudioContextSoundPlayer
  : AudioTagSoundPlayer;

// This piece of code shouldn't be concerned with what player to fallback to if
//   AudioContext isn't supported: it just wants a sound player!
const SoundPlayer = SoundPlayerFactory(detectFeature('audio-api'));
const soundPlayer = SoundPlayer('mysound.mp3');
```
</details>

### 🎉 Composite

⛓ Purpose: Allow clients to treat the individual objects in a uniform manner. We have containers and objects inside them which may also be containers. Create a recursive data structure (tree).  
💡 OOP Solution: Use polymorphism. Give parents and children a uniform interface.  
🔥 OOP + FP Solution: Same. Simplified with duck-typing.

<details>
  <summary>Code Example</summary>

```js
const File = name => ({
  ls: (indent = '') => console.log(indent + name)
});

const Directory = name => {
  let includedFiles = [];
  const ls = (indent = '') => {
    console.log(indent + name);
    includedFiles.forEach(includedFile => includedFile.ls(indent + '  '));
  }
  const add = file => { includedFiles = [...includedFiles, file] };

  return { ls, add };
};

const music = Directory("MUSIC");
const scorpions = Directory("SCORPIONS");
const dio = Directory("DIO");
const track1 = File("Don't wary, be happy.mp3");
const track2 = File("track2.m3u");
const track3 = File("Wind of change.mp3");
const track4 = File("Big city night.mp3");
const track5 = File("Rainbow in the dark.mp3");
music.add(track1);
music.add(scorpions);
music.add(track2);
scorpions.add(track3);
scorpions.add(track4);
scorpions.add(dio);
dio.add(track5);
music.ls();

// Output:
// MUSIC
//    Don't wary, be happy.mp3
//    SCORPIONS
//       Wind of change.mp3
//       Big city night.mp3
//       DIO
//          Rainbow in the dark.mp3
//    track2.m3u
```
</details>

### 🎉 Flyweight

⛓ Purpose: Minimize memory usage or computational expenses by sharing as much as possible with similar objects.  
💡 OOP Solution: Manually amatorize data in another object.  
🔥 OOP + FP Solution: Use function memoization.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

// Anything that will be cached is flyweight.
// Types of tea here will be flyweights.
const GreenTea = strength => ({ name: 'Green Tea', strength });
const EarlGreyTea = strength => ({ name: 'Earl Grey Tea', strength });

// Two patterns in one example!? Awwwwwww shiiiiiii boiiiiiii!
const TeaFactory = type => {
  switch(type){
  case 'green':
    return GreenTea;
  case 'earl-grey':
    return EarlGreyTea;
  }
};

const TeaMaker = () => {
  let availableTea = {};

  const make = R.memoize(type =>
    R.tap(tea => console.log(`Making ${tea.name}!`))(
      TeaFactory(type)(3)
    )
  );

  // Just for comparison.
  const makeWithoutMemoization = type => {
    if(!availableTea[type]) {
      const tea = TeaFactory(type)();
      console.log(`Making ${tea.name}!`);
      availableTea[type] = tea;
    }

    return availableTea[type];
  };

  return { make };
};

const TeaShop = teaMaker => {
  let orders = {};

  const takeOrder = (teaType, tableNum) => {
    orders = {
      [tableNum]: teaMaker.make(teaType),
      ...orders 
    };
  };
  const serve = () => {
    R.toPairs(orders)
      .forEach(([tableNum, tea]) =>
        console.log(`Serving tea: ${tea.name} to table #${tableNum}`)
      );
  }

  return { takeOrder, serve };
};

const teaMaker = TeaMaker();
const shop = TeaShop(teaMaker);

shop.takeOrder('green', 1);
shop.takeOrder('earl-grey', 2);
shop.takeOrder('earl-grey', 4);
shop.takeOrder('green', 24);

shop.serve();

// Output:
// Making Green Tea!
// Making Earl Grey Tea!
// Serving tea: Green Tea to table #1
// Serving tea: Earl Grey Tea to table #2
// Serving tea: Earl Grey Tea to table #4
// Serving tea: Green Tea to table #24
```
</details>

### 🎉 Template

⛓ Purpose: Define a skeleton of how a certain algorithm could be performed, but defers the implementation of those steps to the client.  
💡 OOP Solution: Use inheritence/polymorphism. Abstract base class defines the skeleton method, and calls abstract functions which are implemented by child classes.  
🔥 OOP + FP Solution: Use higher order functions. Pass the functions directly to the builder object.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const compact = list => R.reject(item => !item, list);

const compactAndJoin = (separator, list) =>
  R.pipe(compact, R.join(separator))(list);

const TemplateBookViewer = ({ processTitle, processAuthor = R.identity }) => ({
  showBookTitleInfo: book => {
    const processedTitle = processTitle(book.title());
    const processedAuthor = processAuthor(book.author());

    return compactAndJoin(' by ', [processedTitle, processedAuthor]);
  }
});

const EnthusiasticBookViewer = () => TemplateBookViewer({
  processTitle: R.replace(/ /g, '!!!'),
  processAuthor: R.replace(/ /g, '!!!')
});

const HyphenBookViewer = () => TemplateBookViewer({
  processTitle: R.replace(/ /g, '-')
});

const Book = ({ title, author = '' }) => ({
  title: () => title,
  author: () => author
});

const book = Book({ title: 'PHP for Cats' });
const book2 = Book({
  title: 'How to Disapoint People and Lose Friends',
  author: 'Every Millinial, Ever'
});

const enthusiasticBookViewer = EnthusiasticBookViewer();
const hyphenBookViewer = HyphenBookViewer();

console.log(enthusiasticBookViewer.showBookTitleInfo(book));
console.log(hyphenBookViewer.showBookTitleInfo(book));
console.log(hyphenBookViewer.showBookTitleInfo(book2));

// Output:
// PHP!!!for!!!Cats
// PHP-for-Cats
// How-to-Disapoint-People-and-Lose-Friends by Every Millinial, Ever
```
</details>

### 🎉 Observer (Pub/Sub)

⛓ Purpose: Define a dependency between objects so that whenever an object changes its state, all its dependents are notified.  
💡 OOP Solution: Create an object which maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.  
🔥 OOP + FP Solution: Same, except functions are passed directly.

<details>
  <summary>Code Example</summary>

```js
const JobPost = title => ({ title });

const JobSeeker = name => ({
  onJobPosted: job => console.log(`Hi ${name}. New job posted: ${job.title}`)
});

// With lambdas we can build generic Publisher!
const Publisher = () => {
  let handlers = [];
  const notify = (...args) => handlers.forEach(handler => handler(...args));
  const attach = handler => {
    handlers = [...handlers, handler]
  };

  return { notify, attach };
};

const johnDoe = JobSeeker('John Doe');
const janeDoe = JobSeeker('Jane Doe');
const publisher = Publisher();
publisher.attach(johnDoe.onJobPosted);
publisher.attach(janeDoe.onJobPosted);
publisher.notify(JobPost('Software Engineer'));

// Output:
// Hi John Doe! New job posted: Software Engineer
// Hi Jane Doe! New job posted: Software Engineer
```
</details>

### 🎉 Chain of Responsibility
⛓ Purpose: Build a response chain. Hide from the client the process of finding a suitable handler.   
💡 OOP Solution: Create an object which links a list of handlers together and handles the process of finding a suitable one in response to a request.  
🔥 OOP + FP Solution: Monadic composition.

<details>
  <summary>Code Example</summary>

```js
const {Either} = require('monet');

const Account = ({balance, name}) => {
  let successor;

  const canPay = amount => balance >= amount;
  const pay = amountToPay => {
    if(canPay(amountToPay)) {
      console.log(`Paid ${amountToPay} using ${name}`);
      return Either.Left();
    } else {
      console.log(`Cannot pay with ${name}. Proceeding...`);
      return Either.Right(amountToPay);
    }
  }

  return { pay };
};

const Bank = balance => Account({ balance, name: 'Bank' });
const PayPal = balance => Account({ balance, name: 'PayPal' });
const Bitcoin = balance => Account({ balance, name: 'Bitcoin' });

const bank = Bank(100);
const paypal = PayPal(200);
const bitcoin = Bitcoin(300);

const pay = amount => bank.pay(amount)
  .chain(paypal.pay)
  .chain(bitcoin.pay)
  .chain(() => console.log('Insufficient amount!'))

pay(259);

// Output:
// Cannot pay with Bank. Proceeding...
// Cannot pay with PayPal. Proceeding...
// Paid 259 using Bitcoin

pay(400);

// Output:
// Cannot pay with Bank. Proceeding...
// Cannot pay with PayPal. Proceeding...
// Cannot pay with Bitcoin. Proceeding...
// Insufficient amount!
```
</details>

### ✅ Adapter

⛓ Purpose: Adapt an object to a different interface.  
💡 OOP Solution: Wrap incompatable object in an adapter object to make it compatible.  
🔥 OOP + FP Solution: Same. Simplified with duck-typing, first-class functions, and function composition.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const AfricanLion = () => ({
  roar: () => console.log('rarr')
});
const AsianLion = () => ({
  roar: () => console.log('rawr')
});
const WildDog = () => ({
  bark: () => console.log('woof')
});

// Wow! That was easy! Great moves, keep it up, proud of you!
const DogToLionAdapter = dog => ({
  roar: dog.bark
});

const Hunter = () => ({ hunt: lion => lion.roar() });

// A new constructor!
const HuntableWildDog = R.pipe(WildDog, DogToLionAdapter);

// Now I can create a huntable wild dog in one go!
const wildDog = HuntableWildDog();
const hunter = Hunter();
hunter.hunt(wildDog);

// Output: woof
```
</details>

### ✅ Bridge

⛓ Purpose: Decouple an abstraction from its implementation so that the two can vary independently. Allow us to have variations of an object, but not create a new subtytpe for each variation.  
💡 OOP Solution: Have one object use another in order to provide variations.  
🔥 OOP + FP Solution: Same. This pattern is already compositional in nature. It's about eliminating hierarchy. Simplified with duck-typing.

<details>
  <summary>Code Example</summary>

```js
const About = theme => ({
  getContent: () => `About page in striking ${theme.color()}`
});

const Careers = theme => ({
  getContent: () => `Careers page in striking ${theme.color()}`
});

const DarkTheme = () => ({
  color: () => 'Dark Black'
});
const AquaTheme = () => ({
  color: () => 'Light Blue'
});

const darkTheme = DarkTheme();
const aquaTheme = AquaTheme();
const about = About(darkTheme);
const careers = Careers(aquaTheme);

console.log(about.getContent());    
console.log(careers.getContent());

// Output:
// About page in striking Dark Black
// Careers page in striking Light Blue
```
</details>

### ✅ Decorator

⛓ Purpose: Dynamically modify an object's functionailty. Can also provide new functionality.
💡 OOP Solution: Create a new object which takes the original object and provides new functionality.  
🔥 OOP + FP Solution: Same. Simplified with duck-typing and function composition.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const SimpleCoffee = () => ({
  cost: () => 10,
  description: () => 'Simple coffee'
});

const SugarCoffeeDecorator = coffee => ({
  cost: coffee.cost,
  description: () => coffee.description() + ', sugar'
});

const MilkCoffeeDecorator = coffee => ({
  cost: () => coffee.cost() + 2,
  description: () => coffee.description() + ', milk',
  sour: () => console.log('Left me out too long, bro')
});

const simpleCoffee = SimpleCoffee();
console.log(simpleCoffee.cost());
console.log(simpleCoffee.description());

const sweetLatte = R.pipe(SugarCoffeeDecorator, MilkCoffeeDecorator)(simpleCoffee);
console.log(sweetLatte.cost());
console.log(sweetLatte.description());
sweetLatte.sour();

// Output:
// 10
// Simple coffee
// 12
// Simple coffee, sugar, milk
// Left me out too long, bro

```
</details>

### ✅ Facade

⛓ Purpose: Provide a high-level interface using a low-level object.  
💡 OOP Solution: Create an object which wraps the low-level one, providing a high-level interface to it.  
🔥 OOP + FP Solution: Same. Function composition cleans it up, though.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const Computer = () => ({
  getElectricShock: () => console.log("Ouch!"),
  makeSound: () => console.log("Beep beep!"),
  showLoadingScreen: () => console.log("Loading..."),
  bam: () => console.log("Ready to be used!"),
  closeEverything: () => console.log("Bup bup bup buzzzz!"),
  sooth: () => console.log("Zzzzz..."),
  pullCurrent: () => console.log("Haaah!")
});

const ComputerFacade = computer => ({
  turnOn: () => {
    computer.getElectricShock();
    computer.makeSound();
    computer.showLoadingScreen();
    computer.bam();
  },
  turnOff: () => {
    computer.closeEverything();
    computer.pullCurrent();
    computer.sooth();
  }
});

// Look ma, a new constructor!
const SimpleComputer = R.pipe(Computer, ComputerFacade);

const computer = SimpleComputer();
computer.turnOn();

// Output:
// Ouch!
// Beep beep!
// Loading...
// Ready to be used!

computer.turnOff();

// Output:
// Bup bup buzzz!
// Haah!
// Zzzzz...
```
</details>

### ✅ Proxy

⛓ Purpose: Have an object represent the functionality of another object.  
💡 OOP Solution: Create an object which wraps another delegating all method calls to it and optionally modifying behavior in some way.  
🔥 OOP + FP Solution: Same. Simplified with duck-typing and first-class functions.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const Door = (color = 'red') => ({
  open: () => console.log('Opening lab door'),
  close: () => console.log('Closing lab door'),
  color: () => color
});

const SecureDoor = color => {
  const door = Door(color);
  const authenticate = password => password === '$ecr3t';
  const open = password => authenticate(password)
    ? door.open()
    : console.log('Uh oh! No can do.');
  const close = door.close;

  return { open, close };
};

door = SecureDoor('blue');
door.open('invalid');

door.open('$ecr3t');
door.close();

// Output:
// Uh oh! No can do.
// Opening lab door
// Closing lab door
```

Since ES6, JavaScript provides the `Proxy` object, which allows us to simplify the `SecurityDoor` constructor to this:

```js
const SecureDoor = color => {
  const door = Door(color);
  const authenticate = password => password === '$ecr3t';
  const open = password => authenticate(password)
    ? door.open()
    : console.log('Uh oh! No can do.');

  return new Proxy(door, {open});
}
```

Notice we didn't need to explicitly delegate the close method to the underlying door instance. This is done for us automatically by the `Proxy`. Pretty cool!
</details>

### ✅ Mediator

⛓ Purpose: Reduce coupling between the objects communicating with each other.  
💡 OOP Solution: Add a third party object (called a mediator) to control the interaction between two objects (called colleagues).  
🔥 OOP + FP Solution: Same.

<details>
  <summary>Code Example</summary>

```js
import R from 'ramda';

const ChatRoomMediator = () => ({
  showMessage: R.curry((username, message) => console.log(`Jan 2, 10:58 [${username}]: ${message}`))
});

const User = R.curry(
  (chatMediator, { name }) => ({
    send: chatMediator.showMessage(name)
  })
);

const mediator = ChatRoomMediator();
const MediatedUser = User(mediator);

const john = MediatedUser({ name: 'John Doe' });
const jane = MediatedUser({ name: 'Jane Doe' });
john.send('Hi there!');
jane.send('Hey!');

// Output:
// Jan 2, 10:58 [John Doe]: Hi there!
// Jan 2, 10:58 [Jane Doe]: Hey!
```
</details>

### 🤔 Momento

⛓ Purpose: Provide the ability to restore an object to its previous state.  
💡 OOP Solution: Create an object which handles storing and restoring the state.  
🔥 OOP + FP Solution: Same.

<details>
  <summary>Code Example</summary>

```js
const EditorMemento = content => ({content});

const Editor = () => {
  let content = '';
  const type = words => {
    content = [content, words].join(' ');
  }
  const save = () => EditorMemento(content);
  const restore = momento => {
    content = momento.content
  };

  return { type, save, restore, content: () => content };
}

editor = Editor();
editor.type('This is the first sentence.');
editor.type('This is second.');
saved = editor.save();
editor.type('And this is third.');
console.log(editor.content());
editor.restore(saved);
console.log(editor.content());

// Output:
// This is the first sentence. This is second. And this is third.
// This is the first sentence. This is second.
```
</details>

### 🤔 State (FSM)

⛓ Purpose: Finite-state machine. Change the behavior of an object when the state changes.  
💡 OOP Solution: Implement a state machine by implementing each individual state as a derived class of the state pattern interface, and implementing state transitions by invoking methods defined by the pattern's superclass.  
🔥 OOP + FP Solution: Same.

<details>
  <summary>Code Example</summary>

```js
const UpperCase = () => ({
  write: words => console.log(words.toUpperCase())
});
const LowerCase = () => ({
  write: words => console.log(words.toLowerCase())
});
const Default = () => ({
  write: words => console.log(words)
});

const TextEditor = initialState => {
  let state = initialState;

  const setState = newState => {
    state = newState
  };
  const type = words => state.write(words);

  return { setState, type };
};

const editor = TextEditor(Default());
editor.type('First line');
editor.setState(UpperCase());
editor.type('Second line');
editor.type('Third line');
editor.setState(LowerCase());
editor.type('Fourth line');
editor.type('Fifth line');

// Output:
// First line
// SECOND LINE
// THIRD LINE
// fourth line
// fifth line
```
</details>

### 🤔 Prototype

⛓ Purpose: Create an object based on an existing object through cloning. Can reduce initializtion cost and memory allocation.  
💡 OOP Solution: Some sort of cloning mechanism.  
🔥 OOP + FP Solution: 🤷 Not sure FP techniques help much here.

## Summary
Of the 21 patterns we looked at:

🚫 2 (Builder, Singleton) were found to be antithetical to FP.  
👻 4 (Command, Strategy, Visitor, Iterator) become "invisible" (all are realized using lambdas).  
🎉 6 (Flyweight, Factory, Template, Composite, Observer, Chain of Responsibility) were drastically simplified due to: function memoization and lambdas, first class types, and duck-typing.  
✅ 6 (Adapter, Bridge, Decorator, Facade, Proxy, Mediator) remain, albeit simplified, since they were already compositional in nature.  
🤔 3 (Momento, Prototype, State) remain virtually unchanged.

> Note on compositional patterns:

> * Proxy and Decorator - Both have the same interface as their wrapped types, but the proxy creates an instance under the hood, whereas the decorator takes an instance in the constructor.
> * Adapter and Facade - Both have a different interface than what they wrap, but the adapter derives from an existing interface, whereas the facade creates a new interface.
> * Bridge and Adapter - Both point at an existing type. The bridge will allow you to pair the implementation at runtime, whereas the adapter usually won't. With the bridge, you're not adapting to some legacy or third-party code, you're the designer of all the code but you need to be able to swap out different implementations.

> https://stackoverflow.com/questions/350404/how-do-the-proxy-decorator-adapter-and-bridge-patterns-differ

It seems the terms "Builder," "Command," "Singleton," and "Iterator" can be retired and the terms "Flyweight," "Visitor," and "State" replaced with "Memoization," "Map," and "Finite State Machine."

## Conclusion
Turns out a large portion of OOP design patterns, are already compositional in nature, so they translate very well to a functional style! Who knew!? Makes perfect sense, though, when you learn that the phrase "favor object composition over class inheritance" originated in the seminal book on the subject.

So, anytime you hear people saying functional programming eliminates the GoF design patterns, you can tell them they are, respectfully, full of shit. :) The majority of patterns presented are still just as much relevant today as they were in 1994, they are just realized in different (often simpler) ways.

## Takeaways
* It's better to think of design patterns in terms of what problem they are trying to solve rather than how they are implemented.
* Even patterns which **are** [missing language features](http://wiki.c2.com/?AreDesignPatternsMissingLanguageFeatures) (Command, Strategy, Visitor, Iterator) are value from a nomenclature standpoint.

### References
https://github.com/kamranahmedse/design-patterns-for-humans  
http://www.jot.fm/issues/issue_2008_09/article2.pdf  
https://www.youtube.com/watch?v=ZlPfH6wNDpo  
http://www.grahamlea.com/2014/07/lambda-design-patterns-java-8/  
http://www.norvig.com/design-patterns/design-patterns.pdf  
https://medium.com/javascript-scene/the-open-minded-explorer-s-guide-to-object-composition-88fe68961bed
http://blog.ezyang.com/2010/05/design-patterns-in-haskel/
http://www.cs.ox.ac.uk/jeremy.gibbons/publications/hodgp.pdf
https://www.voxxed.com/2016/05/gang-four-patterns-functional-light-part-1/
