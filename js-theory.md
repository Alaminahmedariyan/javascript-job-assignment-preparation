# 📘 JavaScript + TypeScript Interview গাইড (বিস্তারিত ব্যাখ্যা)

> এই ফাইলে **বুঝে পড়ার** জন্য সব ব্যাখ্যা আছে। উপমা → ব্যাখ্যা → code।
> মুখে বলার ছোট উত্তরগুলো আলাদা ফাইলে আছে: `INTERVIEW_ANSWERS.md`

> **রঙের মানে:** 🔴 অবশ্যই পড়ো (প্রায় নিশ্চিত আসবে) | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে
> প্রতিটা প্রশ্নের heading এর পাশেই রঙ বসানো আছে, উপরে ফিরে দেখতে হবে না।

---

## 📑 সূচিপত্র

### Part A: JavaScript
| # | বিষয় | গুরুত্ব |
|---|---|---|
| 1 | Hoisting | 🔴 |
| 2 | Temporal Dead Zone (TDZ) | 🔴 |
| 3 | Execution Context | 🔴 |
| 4 | Call Stack, Task Queue, Microtask Queue | 🔴 |
| 5 | Event Loop: Promise কেন setTimeout এর আগে | 🔴 |
| 6 | Lexical Scope | 🔴 |
| 7 | Undeclared vs undefined | 🟡 |
| 8 | Closure ও memory leak | 🔴 |
| 9 | Function declaration vs expression | 🟡 |
| 10 | `this` | 🔴 |
| 11 | `bind`, `call`, `apply` | 🔴 |
| 12 | Currying | 🟡 |
| 13 | Function Composition | 🟡 |
| 14 | Prototype Chain | 🔴 |
| 15 | `Object.create()` vs class | 🟡 |
| 16 | Property Lookup | 🔴 |
| 17 | Shallow vs Deep Copy | 🔴 |
| 18 | `Object.freeze()` vs `Object.seal()` | 🟡 |
| 19 | Promise states ও chaining | 🔴 |
| 20 | async/await vs Promise | 🔴 |
| 21 | async function এ await না করলে | 🔴 |
| 22 | `Promise.all()` নিজে বানানো | 🟡 |
| 23 | `all`, `allSettled`, `race`, `any` | 🔴 |
| 24 | Concurrency limit | 🟡 |
| 25 | Callback hell | 🟡 |
| 26 | Memory leak এর কারণ | 🟡 |
| 27 | Garbage Collection | 🟡 |
| 28 | Debounce vs Throttle | 🔴 |
| 29 | Immutability | 🟡 |
| 30 | Pure Functions | 🟡 |
| 31 | Event Delegation | 🔴 |

### Part B: TypeScript
| # | বিষয় | গুরুত্ব |
|---|---|---|
| 32 | `type` vs `interface` | 🔴 |
| 33 | Union vs Intersection | 🔴 |
| 34 | Type Inference | 🟡 |
| 35 | Optional chaining ও Nullish coalescing | 🟡 |
| 36 | `any` vs `unknown` vs `never` | 🔴 |
| 37 | Structural Typing | 🟡 |
| 38 | `readonly` | 🟡 |
| 39 | Generics ও Constraints | 🔴 |
| 40 | `void` vs `never` | 🟡 |
| 41 | Utility Types | 🔴 |
| 42 | `keyof` ও `typeof` | 🟡 |
| 43 | Conditional Types (সংক্ষেপে) | ⚪ |
| 44 | Discriminated Union | 🔴 |
| 45 | API response safely type করা | 🔴 |
| 46 | TS কীভাবে runtime error কমায় | 🟡 |
| 47 | TS → JS compile ও runtime এ types | 🔴 |
| 48 | TypeScript এর trade-offs | 🟡 |

**চিহ্ন:** 🔴 অবশ্যই পড়ো | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে

---

# Part A: JavaScript

---

# 📦 PART 1: Core ও Execution (Hoisting, Scope, Closure, Event Loop)

> প্রশ্ন 1–8 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 1, 2, 3, 4, 5, 6, 8
> 🟡 **Good (মূল ধারণা):** 7

---

## 🔴 1. Hoisting

### 🧠 উপমা
পরীক্ষার আগে শিক্ষক roll sheet বানিয়ে নেন, কে কোথায় বসবে ঠিক করে ফেলেন। JS ও কোড চালানোর আগে সব variable/function এর নাম তালিকায় তুলে নেয়।

### 📖 ব্যাখ্যা
Hoisting মানে declaration গুলো scope এর উপরে "উঠে যায়" বলে মনে হওয়া। আসলে কোড নড়ে না, creation phase এ নামগুলো আগেই memory তে রাখা হয়।

```js
console.log(x);   // undefined (error না)
var x = 5;

console.log(y);   // ❌ ReferenceError
let y = 5;

sayHi();          // ✅ কাজ করে
function sayHi() { console.log('hi'); }

hello();          // ❌ TypeError: hello is not a function
var hello = function () {};
```

| | Hoist হয়? | আগে ব্যবহার করলে |
|---|---|---|
| `var` | ✅, মান `undefined` | `undefined` পাবে |
| `let`/`const` | ✅, কিন্তু TDZ তে | ❌ ReferenceError |
| function declaration | ✅ পুরোটা | ✅ কাজ করে |
| function expression | variable এর নিয়মে | ❌ TypeError (var হলে) |

---

## 🔴 2. Temporal Dead Zone (TDZ)

### 📖 ব্যাখ্যা
`let`/`const` এর scope শুরু থেকে declaration লাইন পর্যন্ত সময়টাকে TDZ বলে। এ সময় variable আছে কিন্তু ছোঁয়া নিষেধ।

```js
{
  console.log(z);  // ❌ ReferenceError (TDZ)
  let z = 10;      // এখান থেকে ব্যবহার করা যাবে
}
```

### কেন আছে?
`var` এর নীরব `undefined` bug লুকিয়ে রাখত। TDZ ভুলটা সাথে সাথে error দিয়ে ধরিয়ে দেয়।

**প্রমাণ যে `let` ও hoist হয়:**
```js
let a = 'বাইরের';
{
  console.log(a);  // ❌ ReferenceError (বাইরেরটা পায় না, ভেতরের a TDZ তে আড়াল করেছে)
  let a = 'ভেতরের';
}
```

---

## 🔴 3. Execution Context

### 📖 ব্যাখ্যা
Execution context হলো কোড চলার "পরিবেশ বা বাক্স"। এতে থাকে variable, `this`, আর outer scope এর reference।

**দুই ধরন:**
- **Global**: পুরো ফাইলের জন্য, একবার
- **Function**: প্রতিটা function call এ নতুন একটা

**দুই ধাপ:**
1. **Creation phase**: variable/function memory তে তোলা (hoisting এখানেই)
2. **Execution phase**: লাইন ধরে কোড চালানো

বাক্সগুলো **call stack** এ জমে। call হলে উপরে যোগ, শেষ হলে সরে যায়।

```js
function a() { b(); }
function b() { console.log('hi'); }
a();
// Stack: global → a() → b() → (শেষে একে একে বের হয়)
```

---

## 🔴 4. Call Stack, Task Queue, Microtask Queue

### 🧠 উপমা
ব্যাংকের কাউন্টার: কাউন্টারে যার কাজ চলছে = **call stack**। সাধারণ লাইন = **task queue**। VIP লাইন = **microtask queue** (আগে পুরো খালি হবে)।

### 📖 ব্যাখ্যা

| | কী থাকে | কখন চলে |
|---|---|---|
| **Call stack** | এখন চলা function | সবসময় |
| **Task (macrotask) queue** | `setTimeout`, `setInterval`, I/O, click event | stack ও microtask খালি হলে, একবারে **একটা** |
| **Microtask queue** | `.then/.catch/.finally`, `await` এর পরের অংশ, `queueMicrotask` | stack খালি হলেই, **সবগুলো** একসাথে |

**Event loop এর ধাপ:**
1. Stack এর কোড শেষ করো
2. Microtask queue **পুরো** খালি করো
3. Task queue থেকে **একটা** নাও
4. আবার ২ এ যাও

---

## 🔴 5. Event Loop: Promise কেন setTimeout এর আগে চলে

কারণ Promise callback যায় **microtask** এ, setTimeout যায় **task queue** তে, আর event loop সবসময় microtask আগে খালি করে।

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
```

**ধাপে ধাপে:** `1` sync, `setTimeout` task queue তে, `then` microtask এ, `4` sync। Stack খালি → microtask `3` → task `2`।

**কঠিন উদাহরণ:**
```js
setTimeout(() => console.log('timeout 1'), 0);
Promise.resolve()
  .then(() => {
    console.log('promise 1');
    setTimeout(() => console.log('timeout 2'), 0);
  })
  .then(() => console.log('promise 2'));
console.log('sync');
// sync → promise 1 → promise 2 → timeout 1 → timeout 2
```

**async/await এর উদাহরণ:**
```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
(async () => {
  console.log('D');   // await পর্যন্ত অংশ sync
  await null;
  console.log('E');   // এটা microtask
})();
console.log('F');
// A, D, F, C, E, B
```

> ⚠️ Microtask এ বারবার নতুন microtask যোগ করলে task queue starve হয়, UI আটকে যায়।

---

## 🔴 6. Lexical Scope

### 🧠 উপমা
রুশ পুতুল: ভেতরেরটা বাইরের সবগুলোকে দেখে, বাইরেরটা ভেতরেরটাকে দেখে না।

### 📖 ব্যাখ্যা
কোন function কোন variable দেখবে তা ঠিক হয় কোড **কোথায় লেখা হয়েছে** তার উপর, কোথা থেকে call হয়েছে তার উপর না।

```js
const country = 'Bangladesh';
function outer() {
  const city = 'Dhaka';
  function inner() {
    const area = 'Mirpur';
    console.log(area, city, country);  // ✅ তিনটাই পায়
  }
  inner();
}
```

**Scope chain:** নিজের scope → বাইরের scope → ... → global → না পেলে ReferenceError।

**প্রমাণ:**
```js
const name = 'global';
function showName() { console.log(name); }
function test() {
  const name = 'test';
  showName();        // "global" (লেখা হয়েছে global এ)
}
test();
```

**Block vs function scope:**
```js
if (true) {
  var a = 1;   // function scope, বাইরে দেখা যায়
  let b = 2;   // block scope
}
console.log(a); // 1
console.log(b); // ❌ ReferenceError
```

---

## 🟡 7. Undeclared vs undefined

```js
let a;
console.log(a);        // undefined → আছে, মান নেই
console.log(b);        // ❌ ReferenceError → declare ই হয়নি
console.log(typeof b); // "undefined" → typeof এ error দেয় না
```

---

## 🔴 8. Closure

### 🧠 উপমা
ছেলে গ্রাম ছেড়ে শহরে গেছে, কিন্তু **পকেটে গ্রামের ঘরের চাবি** আছে। Closure সেই চাবি।

### 📖 ব্যাখ্যা
Closure হলো এমন function যে তার জন্মস্থানের (lexical scope এর) variable মনে রাখে, outer function শেষ হয়ে গেলেও।

```js
function makeCounter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}
const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3

const c2 = makeCounter();
c2();      // 1 (আলাদা count)
```

**ব্যবহার:** private data, memoization, debounce, React hooks।

```js
function createBank() {
  let balance = 0;
  return {
    deposit: (n) => (balance += n),
    getBalance: () => balance,
  };
}
const acc = createBank();
acc.deposit(500);
acc.getBalance(); // 500
acc.balance;      // undefined (লুকানো)
```

### ক্লাসিক ফাঁদ
```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 100);
// 3, 3, 3 😱  (সবাই একই i শেয়ার করে)

for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 100);
// 0, 1, 2 ✅  (let প্রতি iteration এ নতুন i)
```

### Closure কখন memory leak করে?
Closure variable বাঁচিয়ে রাখে, তাই বড় ডেটা আটকে গেলে সমস্যা:
```js
function setup() {
  const huge = new Array(1_000_000).fill('x');
  return () => console.log(huge.length);  // huge কখনো মুছবে না
}
```
**সমাধান:** শুধু দরকারি অংশ ধরো (`const len = huge.length`), কাজ শেষে reference `null` করো, `removeEventListener` করো।

---

# 📦 PART 2: Functions ও `this`

> প্রশ্ন 9–13 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 10, 11
> 🟡 **Good (মূল ধারণা):** 9, 12, 13

---

## 🟡 9. Function Declaration vs Expression

```js
// Declaration
function add(a, b) { return a + b; }

// Expression
const add2 = function (a, b) { return a + b; };
const add3 = (a, b) => a + b;   // arrow expression
```

| | Declaration | Expression |
|---|---|---|
| Hoisting | ✅ পুরোটা, আগে call করা যায় | ❌ variable এর নিয়মে |
| নাম | বাধ্যতামূলক | optional (anonymous হতে পারে) |
| কখন তৈরি | creation phase এ | execution phase এ, লাইন এ পৌঁছালে |

---

## 🔴 10. `this`

### 🧠 সহজ নিয়ম
> **`this` = কে function টাকে call করছে।** (arrow function ছাড়া)

### Normal function
```js
const user = {
  name: 'Alamin',
  hi() { console.log(this.name); }
};
user.hi();            // 'Alamin' (user দিয়ে call হয়েছে)

const f = user.hi;
f();                  // undefined (কেউ call করেনি, this হারিয়ে গেছে)
                      // strict mode এ this = undefined, নইলে globalThis
```

| কীভাবে call | `this` কী |
|---|---|
| `obj.method()` | `obj` |
| `fn()` (সাধারণ) | `undefined` (strict) / `globalThis` |
| `new Fn()` | নতুন বানানো object |
| `fn.call(x)` / `apply` / `bind` | `x` (জোর করে সেট) |

### Arrow function
Arrow function এর **নিজের `this` নেই**। যেখানে লেখা হয়েছে সেখানকার `this` নেয় (lexical this)।

```js
const user = {
  name: 'Alamin',
  hi() {
    setTimeout(function () { console.log(this.name); }, 100);  // ❌ undefined
    setTimeout(() => console.log(this.name), 100);             // ✅ 'Alamin'
  }
};
```

**⚠️ Arrow কে object method বানিও না:**
```js
const obj = { name: 'A', hi: () => console.log(this.name) };  // ❌ this বাইরের scope এর
```

### Event handler
```js
button.addEventListener('click', function () {
  console.log(this);   // button element (event.currentTarget)
});
button.addEventListener('click', () => {
  console.log(this);   // বাইরের scope এর this (button না!)
});
```
Element দরকার হলে arrow তে `event.currentTarget` ব্যবহার করো।

---

## 🔴 11. `bind`, `call`, `apply`

`this` কে **জোর করে** ঠিক করার তিন উপায়।

```js
function greet(greeting, punct) {
  console.log(`${greeting}, ${this.name}${punct}`);
}
const user = { name: 'Alamin' };

greet.call(user, 'Hello', '!');      // সাথে সাথে চালায়, আর্গুমেন্ট আলাদা আলাদা
greet.apply(user, ['Hello', '!']);   // সাথে সাথে চালায়, আর্গুমেন্ট array তে
const bound = greet.bind(user, 'Hello');  // চালায় না, নতুন function ফেরত দেয়
bound('!');
```

| | চালায় কখন | আর্গুমেন্ট |
|---|---|---|
| `call` | সাথে সাথে | কমা দিয়ে |
| `apply` | সাথে সাথে | array |
| `bind` | পরে (নতুন function দেয়) | কমা দিয়ে (partial করা যায়) |

**ব্যবহার:**
- `bind`: `this` হারানো ঠিক করা (callback এ method পাঠালে), partial application
- `call`: method ধার করা: `Array.prototype.slice.call(arguments)`
- `apply`: `Math.max.apply(null, arr)` (এখন `Math.max(...arr)` ব্যবহার হয়)

---

## 🟡 12. Currying

একাধিক আর্গুমেন্টের function কে **একটা একটা আর্গুমেন্টের chain** এ ভাঙা।

```js
// সাধারণ
const add = (a, b, c) => a + b + c;
add(1, 2, 3);

// Curried
const curriedAdd = (a) => (b) => (c) => a + b + c;
curriedAdd(1)(2)(3);   // 6

// সুবিধা: আংশিক function বানানো (reuse)
const add10 = curriedAdd(10);
add10(5)(5);           // 20
```

**কেন কাজের:** reusable specialized function (`const withTax = multiply(1.15)`), composition এ সুবিধা, closure এর ব্যবহার।

**Generic curry:**
```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}
```

---

## 🟡 13. Function Composition

ছোট ছোট function জোড়া লাগিয়ে বড় কাজ। একটার output পরেরটার input।

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);  // ডান থেকে বামে
const pipe    = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);       // বাম থেকে ডানে

const trim  = (s) => s.trim();
const lower = (s) => s.toLowerCase();
const slug  = (s) => s.replace(/\s+/g, '-');

const makeSlug = pipe(trim, lower, slug);
makeSlug('  Hello World  ');  // 'hello-world'
```

---

# 📦 PART 3: Objects ও Prototype

> প্রশ্ন 14–18 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 14, 16, 17
> 🟡 **Good (মূল ধারণা):** 15, 18

---

## 🔴 14. Prototype Chain

### 🧠 উপমা
বাচ্চা কিছু না জানলে বাবাকে জিজ্ঞেস করে, বাবা না জানলে দাদাকে। এভাবে ওপরে ওঠে।

### 📖 ব্যাখ্যা
প্রতিটা object এর একটা লুকানো link আছে (`[[Prototype]]`, দেখা যায় `__proto__` বা `Object.getPrototypeOf()` দিয়ে) যা আরেকটা object কে নির্দেশ করে। সেই object এরও prototype আছে। এভাবে `null` পর্যন্ত চলে। এটাই **prototype chain**।

```js
const animal = { eats: true };
const dog = Object.create(animal);   // dog এর prototype = animal
dog.barks = true;

dog.barks;   // true (নিজের)
dog.eats;    // true (animal থেকে)
dog.flies;   // undefined (পুরো chain এ নেই)
```

Chain: `dog → animal → Object.prototype → null`

**Array এর উদাহরণ:** `[1,2].map(...)` এ `map` array তে নেই, আছে `Array.prototype` এ। Chain: `array → Array.prototype → Object.prototype → null`।

**Constructor function:**
```js
function Person(name) { this.name = name; }
Person.prototype.hi = function () { return 'Hi ' + this.name; };

const p = new Person('Alamin');
p.hi();   // method টা সব instance এ share হয়, memory বাঁচে
```

---

## 🟡 15. `Object.create()` vs Class Inheritance

আসলে **দুটোই prototype chain ই ব্যবহার করে**। `class` হলো prototype এর উপর সুন্দর syntax (syntactic sugar)।

```js
// Object.create: সরাসরি prototype সেট
const animal = { speak() { return 'sound'; } };
const dog = Object.create(animal);
dog.speak();

// class
class Animal {
  speak() { return 'sound'; }
}
class Dog extends Animal {
  constructor(name) { super(); this.name = name; }
}
```

| | `Object.create` | `class` |
|---|---|---|
| Syntax | কম, সরাসরি | পরিচিত OOP style |
| Constructor | নেই | আছে |
| `super`, `extends` | নেই | আছে |
| Private field (`#x`) | নেই | আছে |
| কখন | সহজ delegation, prototype ছাড়া object (`Object.create(null)`) | বড় প্রজেক্ট, OOP |

---

## 🔴 16. Property Lookup

`obj.x` লিখলে JS এই ক্রমে খোঁজে:
1. object এর **নিজের** property
2. না পেলে **prototype** এ
3. না পেলে prototype এর prototype এ ... `null` পর্যন্ত
4. কোথাও না পেলে `undefined`

```js
const a = { x: 1 };
const b = Object.create(a);
b.x;                    // 1 (a থেকে)
b.hasOwnProperty('x');  // false
Object.hasOwn(b, 'x');  // false (আধুনিক পদ্ধতি)
'x' in b;               // true (chain সহ চেক)
```

**Shadowing:** নিজের property থাকলে সেটাই আগে পায়।
**লেখার সময়:** `b.x = 5` লিখলে `b` এর **নিজের** নতুন property হয়, `a` বদলায় না।

---

## 🔴 17. Shallow Copy vs Deep Copy

### 🧠 উপমা
**Shallow:** বইয়ের শুধু সূচিপত্র ফটোকপি করলে, ভেতরের পাতা আসল বইয়েরই থাকে। **Deep:** পুরো বই ফটোকপি।

```js
const original = { name: 'A', address: { city: 'Dhaka' } };

// Shallow
const s1 = { ...original };
const s2 = Object.assign({}, original);

s1.name = 'B';             // original এ প্রভাব নেই ✅
s1.address.city = 'CTG';   // ❌ original.address.city ও বদলে গেল! (nested একই reference)

// Deep
const d1 = structuredClone(original);   // আধুনিক, সবচেয়ে ভালো
d1.address.city = 'Sylhet';             // original অপরিবর্তিত ✅
```

| উপায় | Deep? | সীমাবদ্ধতা |
|---|---|---|
| `{...obj}`, `Object.assign`, `slice()` | ❌ shallow | nested শেয়ার হয় |
| `JSON.parse(JSON.stringify(x))` | ✅ | `Date` string হয়, `undefined`/function হারায়, circular এ error |
| `structuredClone(x)` | ✅ | function copy করে না |
| `lodash.cloneDeep` | ✅ | library লাগে |

---

## 🟡 18. `Object.freeze()` vs `Object.seal()`

| | নতুন property যোগ | মুছা | মান বদলানো |
|---|---|---|---|
| normal | ✅ | ✅ | ✅ |
| `Object.preventExtensions` | ❌ | ✅ | ✅ |
| `Object.seal` | ❌ | ❌ | ✅ |
| `Object.freeze` | ❌ | ❌ | ❌ |

```js
const o = Object.freeze({ a: 1, nested: { b: 2 } });
o.a = 99;         // নীরবে ব্যর্থ (strict mode এ TypeError)
o.nested.b = 99;  // ✅ বদলে যায়! freeze shallow
```
**মনে রাখো:** দুটোই **shallow**। Deep freeze চাইলে recursive করতে হবে।

---

# 📦 PART 4: Asynchronous JavaScript (Promise, async/await)

> প্রশ্ন 19–25 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 19, 20, 21, 23
> 🟡 **Good (মূল ধারণা):** 22, 24, 25

---

## 🔴 19. Promise States ও Chaining

**Pending → Fulfilled / Rejected।** একবার settled হলে আর বদলায় না।

**প্রতিটা `.then()` নতুন Promise return করে**, তাই chain হয়।

```js
fetchUser(1)
  .then(user => fetchPosts(user.id))   // Promise return → অপেক্ষা করে
  .then(posts => posts.length)         // value return → পরেরটা পায়
  .catch(err => console.log(err))      // উপরের যেকোনো error
  .finally(() => console.log('শেষ'));  // সবসময় চলে
```

**নিয়ম:** value return → পরের then পায় | Promise return → অপেক্ষা | throw → কাছের catch | return ভুলে গেলে পরেরটা `undefined` পায়।

---

## 🔴 20. async/await vs Promise

`async/await` = Promise এর উপর syntactic sugar।

- `async` function **সবসময় Promise** return করে
- `await` function কে থামায় (program নয়), বাকি অংশ **microtask** এ রাখে
- Error: `try/catch`

```js
async function getData() {
  const user = await fetchUser(1);
  const posts = await fetchPosts(user.id);
  return posts.length;
}
```

**Performance ভুল:**
```js
// ❌ ৩ সেকেন্ড
const a = await getA(); const b = await getB(); const c = await getC();
// ✅ ১ সেকেন্ড
const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
```

---

## 🔴 21. async function এ `await` না করলে

```js
async function getName() { return 'Alamin'; }
const r = getName();
console.log(r);    // Promise { 'Alamin' } ❌
```

**সমস্যা:**
1. Promise object পাও, আসল value না
2. কাজ শেষ হওয়ার আগেই পরের কোড চলে (race condition)
3. Error হারিয়ে **unhandled rejection**, Node এ process crash করতে পারে
4. `forEach(async ...)` await এর জন্য অপেক্ষা করে না

```js
// ❌
items.forEach(async (i) => { await save(i); });
// ✅
await Promise.all(items.map(save));          // parallel
for (const i of items) await save(i);        // ক্রমানুসারে
```

ইচ্ছা করে না await করলে: `sendEmail(u).catch(logError);`

---

## 🟡 22. `Promise.all()` নিজে implement

```js
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;
    if (promises.length === 0) return resolve([]);

    promises.forEach((p, index) => {
      Promise.resolve(p)
        .then((value) => {
          results[index] = value;     // index ধরে → ক্রম ঠিক
          completed++;
          if (completed === promises.length) resolve(results);
        })
        .catch(reject);               // একটা fail → সব reject
    });
  });
}
```
**মূল পয়েন্ট:** `results[index]` (push না), counter, `Promise.resolve` দিয়ে wrap, খালি array হ্যান্ডেল।

---

## 🔴 23. `all`, `allSettled`, `race`, `any`

| Method | Resolve | Reject |
|---|---|---|
| `all` | সব সফল → value এর array | যেকোনো একটা fail |
| `allSettled` | সবসময়, প্রতিটার `{status, value/reason}` | কখনো না |
| `race` | প্রথম settle যদি সফল | প্রথম settle যদি fail |
| `any` | প্রথম **সফল** | সবাই fail (`AggregateError`) |

**ব্যবহার:** `all` = সব লাগবে | `allSettled` = bulk কাজের রিপোর্ট | `race` = timeout | `any` = একাধিক server থেকে যে আগে সফল হয়।

```js
const timeout = new Promise((_, rej) => setTimeout(() => rej(new Error('Timeout')), 5000));
const data = await Promise.race([fetch('/api'), timeout]);
```

---

## 🟡 24. Concurrency Limit সহ async কাজ

১০০০টা request একসাথে পাঠালে server/API ডুবে যাবে। তাই **একসাথে সর্বোচ্চ N টা** চালাতে হয়।

```js
async function runWithLimit(tasks, limit) {
  const results = [];
  let next = 0;

  async function worker() {
    while (next < tasks.length) {
      const i = next++;                 // নিজের কাজের index নিলাম
      results[i] = await tasks[i]();    // কাজ শেষ হলে পরেরটা নেবে
    }
  }

  // limit সংখ্যক worker একসাথে চালু
  await Promise.all(Array.from({ length: Math.min(limit, tasks.length) }, worker));
  return results;
}

// ব্যবহার: tasks = function এর array (Promise না!)
const tasks = urls.map((u) => () => fetch(u).then((r) => r.json()));
await runWithLimit(tasks, 5);   // একসাথে সর্বোচ্চ ৫টা
```
প্রস্তুত library: `p-limit`, `p-queue`।

---

## 🟡 25. Callback Hell ও আধুনিক সমাধান

Callback এর ভেতর callback এর ভেতর callback → ডানে সরতে থাকা "pyramid of doom", পড়া ও error handling কঠিন।

```js
// ❌ Callback hell
getUser(id, (err, user) => {
  getPosts(user.id, (err, posts) => {
    getComments(posts[0].id, (err, comments) => { /* ... */ });
  });
});

// ✅ async/await
try {
  const user = await getUser(id);
  const posts = await getPosts(user.id);
  const comments = await getComments(posts[0].id);
} catch (e) { /* এক জায়গায় সব error */ }
```

**সমাধান:** Promise chaining → async/await → named function এ ভাঙা → `util.promisify` (পুরনো callback API কে Promise বানাতে)।

---

# 📦 PART 5: Performance, Memory ও Advanced

> প্রশ্ন 26–31 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 28, 31
> 🟡 **Good (মূল ধারণা):** 26, 27, 29, 30

---

## 🟡 26. JavaScript এ Memory Leak এর কারণ

1. **Global variable** এ জমানো
2. **Unbounded cache** (সীমাহীন object/array)
3. **Event listener** যোগ করে remove না করা
4. **`setInterval`** clear না করা
5. **Closure** যা বড় ডেটা ধরে রাখে
6. **Detached DOM node** (DOM থেকে সরানো কিন্তু JS variable এ reference আছে)

**Debug:** Chrome DevTools → Memory → দুটো heap snapshot compare।
**প্রতিরোধ:** LRU cache, cleanup (`removeEventListener`, `clearInterval`), `WeakMap`/`WeakRef`।

---

## 🟡 27. Garbage Collection (Mark-and-Sweep)

### 📖 ব্যাখ্যা
Reference counting এর সমস্যা ছিল circular reference। এখন সব engine **mark-and-sweep** ব্যবহার করে:

1. **Roots** থেকে শুরু (global object, চলমান function এর variable)
2. Roots থেকে **পৌঁছানো যায়** এমন সব object কে **mark** করো
3. যা mark হয়নি সেগুলো **sweep** (মুছে memory ফেরত)

মূল ধারণা: **reachability**। কোনো roots থেকে পৌঁছানো না গেলে সে আবর্জনা, নিজেদের মধ্যে reference থাকলেও।

```js
let a = { name: 'x' };
a = null;   // আগের object এ আর কোনো reference নেই → GC মুছবে
```

**V8 এ generational GC:** নতুন object **young generation** এ (ঘন ঘন, দ্রুত পরিষ্কার), যারা টিকে যায় **old generation** এ (কম পরিষ্কার)।

---

## 🔴 28. Debounce vs Throttle

### 🧠 উপমা
- **Debounce:** লিফটের দরজা। কেউ ঢুকলে timer reset হয়। **সবাই থামার** পর বন্ধ হয়।
- **Throttle:** বাসের সময়সূচি। যতই যাত্রী আসুক, **প্রতি ১০ মিনিটে একটাই** বাস।

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);                       // আগের timer বাতিল
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, interval) {
  let lastTime = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

| | কখন চলে | ব্যবহার |
|---|---|---|
| **Debounce** | event **থামার** পর একবার | search box (user টাইপ থামালে API call), window resize শেষে, auto-save |
| **Throttle** | প্রতি নির্দিষ্ট সময়ে **সর্বোচ্চ একবার** | scroll, mousemove, button spam আটকানো |

Debounce এ `timer` আর throttle এ `lastTime` **closure** দিয়ে ধরা আছে।

---

## 🟡 29. Immutability

Immutable মানে ডেটা তৈরির পর **বদলানো হয় না**, বদলাতে হলে **নতুন কপি** বানাও।

```js
// ❌ Mutation
user.age = 26;
arr.push(4);

// ✅ Immutable
const newUser = { ...user, age: 26 };
const newArr = [...arr, 4];
const removed = arr.filter((x) => x !== 3);
```

**কেন জরুরি:**
- **Predictable**: কেউ লুকিয়ে ডেটা বদলে bug বানাতে পারে না
- **React/Redux**: state বদল চেনে reference তুলনা (`===`) দিয়ে। Mutate করলে reference একই থাকে, UI update হয় না
- **Debug সহজ**, undo/redo সম্ভব
- **Concurrency তে নিরাপদ**

Deep nested এর জন্য: `structuredClone`, **Immer** library।

---

## 🟡 30. Pure Functions

দুটো শর্ত:
1. **একই input → সবসময় একই output**
2. **Side effect নেই** (বাইরের কিছু বদলায় না: global, DB, console, DOM, input mutate)

```js
// ✅ Pure
const add = (a, b) => a + b;

// ❌ Impure
let total = 0;
const addToTotal = (n) => (total += n);        // বাইরের state বদলায়
const now = () => Date.now();                  // প্রতিবার আলাদা output
const push = (arr, x) => { arr.push(x); return arr; };  // input mutate
```

**সুবিধা:** test করা সহজ, predictable, cache করা যায় (memoization), parallel এ নিরাপদ।

---

## 🔴 31. Event Delegation

### 🧠 উপমা
১০০ জন কর্মচারীর জন্য আলাদা ১০০টা রিসেপশনিস্ট না রেখে **গেটে একজন** রাখা, সে-ই দেখে কে ঢুকলো।

### 📖 ব্যাখ্যা
প্রতিটা child এ listener না বসিয়ে **parent এ একটাই listener** বসানো। Event **bubble** করে উপরে আসে, parent এ `event.target` দেখে বোঝা যায় আসলে কোথায় click হয়েছে।

```js
// ❌ ১০০০টা li তে ১০০০টা listener
document.querySelectorAll('li').forEach((li) => li.addEventListener('click', handler));

// ✅ একটাই listener
document.querySelector('ul').addEventListener('click', (e) => {
  const li = e.target.closest('li');   // closest: ভেতরে icon/span এ click হলেও কাজ করে
  if (!li) return;
  console.log('clicked:', li.dataset.id);
});
```

**সুবিধা:**
- কম memory ও listener
- **পরে যোগ করা element এও** কাজ করে (dynamic list)
- cleanup সহজ

**সীমা:** `focus`, `blur` bubble করে না। `e.stopPropagation()` দিলে delegation ভাঙে।

---

# Part B: TypeScript

---

# 📦 PART 6: TypeScript Types

> প্রশ্ন 32–38 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 32, 33, 36
> 🟡 **Good (মূল ধারণা):** 34, 35, 37, 38

---

## 🔴 32. `type` vs `interface`

```ts
interface User { name: string; age: number; }
type UserT = { name: string; age: number };
```

| | `interface` | `type` |
|---|---|---|
| Object shape | ✅ | ✅ |
| Union (`A \| B`) | ❌ | ✅ |
| Primitive alias, tuple | ❌ | ✅ |
| Extend | `extends` | `&` (intersection) |
| **Declaration merging** | ✅ (একই নামে দুবার লিখলে মিশে যায়) | ❌ error |
| Mapped/conditional type | ❌ | ✅ |

```ts
interface A { x: number }
interface A { y: number }      // merge: A = { x, y }

type ID = string | number;     // শুধু type এ সম্ভব
type Point = [number, number]; // tuple
```
**নিয়ম:** object/class এর shape ও public API এর জন্য `interface`, union/tuple/utility এর জন্য `type`। টিমের একটা নিয়ম ঠিক করে সেটাই মানো।

---

## 🔴 33. Union vs Intersection

```ts
// Union: এটা অথবা ওটা
type ID = string | number;
let id: ID = 5;
id = 'abc';

// Intersection: এটা এবং ওটা (দুটোই)
type Person = { name: string };
type Employee = { salary: number };
type Staff = Person & Employee;       // { name, salary } দুটোই লাগবে
const s: Staff = { name: 'A', salary: 100 };
```

- **Union**: "একটা অপশন" (API response success | error)
- **Intersection**: "দুটো মিশিয়ে" (mixin, props extend করা)

Union এ শুধু **দুটোর common member** সরাসরি ব্যবহার করা যায়, বাকিদের জন্য narrowing (`typeof`, `in`, `instanceof`) লাগে।

---

## 🟡 34. Type Inference

TS নিজেই মান দেখে type বুঝে নেয়, সবসময় লিখতে হয় না।

```ts
let x = 5;              // number
const y = 'hi';         // 'hi' (literal type)
const arr = [1, 2, 3];  // number[]
function add(a: number, b: number) { return a + b; }  // return: number (inferred)
```
**ভালো অভ্যাস:** function এর **parameter** এ type লেখো, **return/local variable** এ inference এ ভরসা করো (public API তে return type লেখা ভালো)।

---

## 🟡 35. Optional Chaining (`?.`) ও Nullish Coalescing (`??`)

```ts
const city = user?.address?.city;        // user বা address null/undefined হলে error না, undefined
const fn = obj.method?.();               // method থাকলে call
const item = arr?.[0];

const port = config.port ?? 3000;        // শুধু null/undefined হলে ডিফল্ট
```

**`??` vs `||` (জনপ্রিয় প্রশ্ন):**
```ts
0 || 10      // 10  ❌ (0 falsy)
0 ?? 10      // 0   ✅ (0 valid মান)
'' || 'x'    // 'x'
'' ?? 'x'    // ''
```
`||` সব falsy (`0`, `''`, `false`) এ fallback দেয়, `??` শুধু `null`/`undefined` এ।

---

## 🔴 36. `any` vs `unknown` vs `never`

| | মানে | নিরাপদ? |
|---|---|---|
| `any` | type checking **বন্ধ**, যা খুশি করো | ❌ |
| `unknown` | "জানি না", ব্যবহারের আগে **যাচাই** করতেই হবে | ✅ |
| `never` | **কখনো ঘটে না** (কোনো মানই নেই) | — |

```ts
let a: any = 'hi';
a.foo.bar();               // কোনো error নেই, runtime এ crash 💥

let u: unknown = 'hi';
u.toUpperCase();           // ❌ error
if (typeof u === 'string') u.toUpperCase();   // ✅ narrowing এর পর

function fail(msg: string): never { throw new Error(msg); }   // কখনো return করে না
```
**নিয়ম:** `any` এড়াও, অজানা ডেটায় `unknown` ব্যবহার করো। `never` exhaustive check এ কাজে লাগে (প্রশ্ন ৪৪)।

---

## 🟡 37. Structural Typing

TS **নাম** দেখে না, **shape (গঠন)** দেখে। যার গঠন মিলে যায় সে compatible।

```ts
interface Point { x: number; y: number }
const p = { x: 1, y: 2, z: 3 };
const q: Point = p;     // ✅ z বাড়তি থাকলেও চলে (variable থেকে দিলে)
```
এটাকে **duck typing** ও বলে: "হাঁসের মতো হাঁটে, হাঁসের মতো ডাকে, তাহলে হাঁস।"

**বাড়তি property check:** সরাসরি object literal দিলে TS বাড়তি property ধরে:
```ts
const r: Point = { x: 1, y: 2, z: 3 };  // ❌ excess property error
```
(Java/C# এর মতো nominal typing এ নাম মিলতে হয়।)

---

## 🟡 38. `readonly`

```ts
interface Config { readonly apiUrl: string }
const c: Config = { apiUrl: 'x' };
c.apiUrl = 'y';                  // ❌ compile error

const arr: readonly number[] = [1, 2];
arr.push(3);                     // ❌

const o = { a: 1 } as const;     // সব readonly + literal type
type R = Readonly<{ a: number }>;
```
**⚠️ গুরুত্বপূর্ণ:** `readonly` শুধু **compile time এ** enforce হয়। JS এ compile হলে এর কোনো চিহ্ন থাকে না। Runtime এ সত্যিকারের সুরক্ষা চাইলে `Object.freeze`। আর `const` আর `readonly` আলাদা: `const` variable এ নতুন মান বসানো আটকায়, `readonly` property তে।

---

# 📦 PART 7: TypeScript Generics ও Advanced Types

> প্রশ্ন 39–44 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 39, 41, 44
> 🟡 **Good (মূল ধারণা):** 40, 42
> ⚪ **সময় থাকলে:** 43

---

## 🔴 39. Generics ও Constraints

### 🧠 উপমা
Generic হলো **টাইপের জন্য function parameter**। ফাঁকা বাক্স যার ভেতরে কী যাবে পরে ঠিক হবে।

```ts
function identity<T>(value: T): T { return value; }
identity<string>('hi');
identity(5);                 // T = number, TS নিজে বুঝে নেয় (inference)

function first<T>(arr: T[]): T | undefined { return arr[0]; }
const n = first([1, 2, 3]);  // number | undefined
```
`any` এর বদলে generic: input আর output এর **সম্পর্ক** ধরে রাখে।

### Constraints (`extends`)
```ts
function getLength<T extends { length: number }>(x: T) { return x.length; }
getLength('abc');    // ✅
getLength([1, 2]);   // ✅
getLength(5);        // ❌ number এ length নেই

function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
getProp({ name: 'A', age: 5 }, 'name');   // string
getProp({ name: 'A' }, 'x');              // ❌ 'x' key নেই
```

**Default generic:** `interface ApiRes<T = unknown> { data: T }`

---

## 🟡 40. `void` vs `never`

```ts
function log(): void { console.log('hi'); }     // return করে, কিন্তু কোনো দরকারি মান না
function fail(): never { throw new Error('x'); } // কখনো return ই করে না
function loop(): never { while (true) {} }
```
- `void`: function শেষ হয়, মান নেই
- `never`: function **শেষই হয় না** (throw বা অসীম loop)

---

## 🔴 41. Utility Types

```ts
interface User { id: number; name: string; email: string; age: number }

Partial<User>            // সব field optional (update এর জন্য)
Required<User>           // সব field বাধ্যতামূলক
Pick<User, 'id' | 'name'>   // শুধু এই দুটো রাখো
Omit<User, 'email'>         // email বাদে সব
Record<string, number>      // { [key: string]: number }
Readonly<User>              // সব readonly
```

| Utility | কাজ | ব্যবহার |
|---|---|---|
| `Partial<T>` | সব optional | PATCH/update payload |
| `Required<T>` | সব required | validated config |
| `Pick<T, K>` | নির্বাচিত field | API response এর অংশ |
| `Omit<T, K>` | কিছু বাদ | `Omit<User, 'password'>` |
| `Record<K, V>` | key-value map | lookup table |

```ts
type UserPreview = Pick<User, 'id' | 'name'>;
type SafeUser = Omit<User, 'password'>;
const roles: Record<'admin' | 'user', string[]> = { admin: ['all'], user: ['read'] };
```
আরো: `ReturnType<typeof fn>`, `Awaited<T>`, `NonNullable<T>`।

---

## 🟡 42. `keyof` ও `typeof` একসাথে

```ts
const config = { host: 'localhost', port: 3000 };

type Config = typeof config;           // { host: string; port: number }  (মান থেকে type)
type ConfigKey = keyof typeof config;  // 'host' | 'port'                  (type থেকে key)

function get(key: ConfigKey) { return config[key]; }
```
- `typeof x` (type position এ): variable থেকে type বের করে
- `keyof T`: type এর সব key এর union

**সুন্দর প্যাটার্ন:**
```ts
const ROLES = { admin: 'ADMIN', user: 'USER' } as const;
type Role = typeof ROLES[keyof typeof ROLES];   // 'ADMIN' | 'USER'
```

---

## ⚪ 43. Conditional Types (সংক্ষেপে)

```ts
type IsString<T> = T extends string ? true : false;
type A = IsString<'x'>;   // true
type B = IsString<5>;     // false

type ElementType<T> = T extends (infer U)[] ? U : T;   // infer: ভেতরের type বের করা
type E = ElementType<number[]>;                        // number
```
`ReturnType`, `Awaited`, `NonNullable` এসব এভাবেই বানানো। ইন্টারভিউতে মূল ধারণা বললেই যথেষ্ট: "type level এর if-else"।

---

## 🔴 44. Discriminated Union

### 📖 ব্যাখ্যা
Union এর প্রতিটা variant এ একটা **সাধারণ literal field** (tag) থাকে, যা দেখে TS বুঝে নেয় কোনটা।

```ts
type Result =
  | { status: 'success'; data: string[] }
  | { status: 'error'; message: string }
  | { status: 'loading' };

function handle(r: Result) {
  switch (r.status) {
    case 'success': return r.data.length;     // TS জানে এখানে data আছে
    case 'error':   return r.message;         // এখানে message আছে
    case 'loading': return 'wait';
    default: {
      const _exhaustive: never = r;           // নতুন status যোগ করে handle না করলে compile error!
      return _exhaustive;
    }
  }
}
```
**কোথায় কাজে লাগে:** API response (success/error), UI state (loading/success/error), Redux action, form state।
**বড় সুবিধা:** `never` দিয়ে **exhaustive checking**, নতুন variant যোগ করলে যেখানে handle করোনি সেখানে compile error।

---

# 📦 PART 8: TypeScript Real-world

> প্রশ্ন 45–48 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 45, 47
> 🟡 **Good (মূল ধারণা):** 46, 48

---

## 🔴 45. API Response Safely Type করা

**সমস্যা:** `const data = (await res.json()) as User` শুধু TS কে চুপ করায়। Backend ভুল ডেটা পাঠালে **runtime এ crash**, TS কিছু ধরবে না। Types runtime এ থাকে না!

**সঠিক উপায়: runtime validation (Zod)**
```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
});
type User = z.infer<typeof UserSchema>;     // schema থেকেই type, একটাই source of truth

async function getUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error('Request failed');
  const json: unknown = await res.json();   // any না, unknown
  return UserSchema.parse(json);            // ভুল হলে এখানেই error, ভেতরে ঢুকতে পারে না
}
```

**Backend এর অচেনা ডেটা (unknown data):**
1. `unknown` ব্যবহার করো, `any` না
2. Zod/Valibot দিয়ে validate
3. অথবা নিজের **type guard**:
```ts
function isUser(x: unknown): x is User {
  return typeof x === 'object' && x !== null && 'id' in x && 'name' in x;
}
```
4. Success/error এর জন্য discriminated union

---

## 🟡 46. TypeScript কীভাবে Runtime Error কমায়

- **null/undefined** ভুল ধরে (`strictNullChecks`): `Cannot read properties of undefined` এর অনেকটাই আগেই ধরা পড়ে
- ভুল type এ function call, ভুল property নাম (typo) ধরে
- **Refactor নিরাপদ**: নাম বদলালে সব জায়গায় error দেখায়
- Exhaustive check (discriminated union)
- IDE autocomplete ও docs

**সীমা:** TS শুধু compile time এ কাজ করে। External ডেটা (API, user input) এর জন্য runtime validation লাগে।

**`tsconfig.json` এ `"strict": true`** চালু রাখো, এটাই সবচেয়ে বড় সুরক্ষা।

---

## 🔴 47. TS → JS Compile ও Runtime এ Types

### কীভাবে compile হয়
```
.ts ফাইল → TypeScript Compiler (tsc) → .js ফাইল
```
1. **Parse**: কোড পড়ে AST বানায়
2. **Type check**: types যাচাই, error থাকলে report (তবু JS emit করতে পারে)
3. **Emit**: types **মুছে ফেলে** (type erasure), `tsconfig` এর `target` অনুযায়ী পুরনো JS syntax এ নামায়

### Runtime এ types কী হয়? **কিছুই না, মুছে যায়।**
```ts
interface User { name: string }
function hi(u: User): string { return u.name; }
```
↓ compile
```js
function hi(u) { return u.name; }   // interface, type annotation সব গায়েব
```

**ফলাফল:**
- `typeof`/`instanceof` দিয়ে interface চেক করা যায় না (`x instanceof User` ❌ যদি User interface হয়)
- Runtime validation এর জন্য Zod ইত্যাদি লাগে
- **ব্যতিক্রম:** `enum` আর `class` runtime এও থাকে (কারণ তাদের জন্য JS কোড তৈরি হয়)
- Node এ সরাসরি চালাতে `tsx`, `ts-node`, বা Node এর নতুন type-stripping, আর build এ `esbuild`/`swc`/`tsc`

---

## 🟡 48. TypeScript এর Trade-offs

| সুবিধা | অসুবিধা |
|---|---|
| bug আগেই ধরা, refactor নিরাপদ | শেখার খরচ (generics, advanced types) |
| ভালো autocomplete, living documentation | Build step ও config যোগ হয় |
| বড় টিমে কোড বোঝা সহজ | Compile time বাড়ে (বড় প্রজেক্টে) |
| API contract স্পষ্ট | জটিল type লিখতে সময় যায়, কখনো overengineering |
| | **Runtime নিরাপত্তা দেয় না**, external ডেটায় validation লাগে |
| | Third-party library তে type না থাকলে ঝামেলা (`@types/*`) |

**সারকথা:** ছোট স্ক্রিপ্টে overhead, কিন্তু বড়/দীর্ঘমেয়াদী/টিমের প্রজেক্টে লাভ অনেক বেশি।

---

## 🎯 শেষ মুহূর্তের টিপস

1. **মুখস্থ নয়, উপমা মনে রাখো।** উপমা মনে থাকলে নিজের ভাষায় বলতে পারবে।
2. **আগে ছোট উত্তর, তারপর "চাইলে আরো বিস্তারিত বলতে পারি।"**
3. **নিজের প্রজেক্টের অভিজ্ঞতা জুড়ে দাও** (Next.js, Prisma, Zod, Redis, queue)। এটাই সবচেয়ে বেশি impression ফেলে।
4. কিছু না জানলে সৎভাবে বলো, আর যতটুকু জানো সেটা থেকে যুক্তি দাও।
5. **Code লিখতে বললে** ধাপগুলো মুখে বলতে বলতে লেখো।

**সবচেয়ে বেশি আসা কোড-লেখার প্রশ্ন (নিজে একবার লিখে practice করো):**
- `debounce` / `throttle`
- `Promise.all` implement
- Closure counter
- `curry`
- Deep clone
- Concurrency limit pool
- Event loop output prediction

শুভকামনা! 🚀