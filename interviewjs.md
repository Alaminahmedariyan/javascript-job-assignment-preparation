# 🎤 Interview এ যেভাবে বলবে (JavaScript + TypeScript)

> এই ফাইলে শুধু **মুখে বলার উত্তর**। ব্যাখ্যা ও code এর জন্য `README.md` দেখো।
> প্রতিটা উত্তর ৩০-৬০ সেকেন্ডে বলার মতো। নিজের ভাষায় একটু বদলে বলবে, মুখস্থ শোনালে ধরা পড়ে।

> **রঙের মানে:** 🔴 অবশ্যই পড়ো (প্রায় নিশ্চিত আসবে) | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে
> প্রতিটা প্রশ্নের heading এর পাশেই রঙ বসানো আছে, উপরে ফিরে দেখতে হবে না।

---

# Part A: JavaScript

---

# 📦 PART 1: Core ও Execution (Hoisting, Scope, Closure, Event Loop)

> প্রশ্ন 1–8 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 1, 2, 3, 4, 5, 6, 8
> 🟡 **Good (মূল ধারণা):** 7

---

### 🔴 1. Hoisting
> "Hoisting হলো JS engine কোড চালানোর আগে creation phase এ declaration গুলো memory তে তুলে রাখা। `var` hoist হয়ে `undefined` মান পায়, তাই আগে ব্যবহার করলে error হয় না। `let` আর `const` ও hoist হয়, কিন্তু declaration লাইন পর্যন্ত Temporal Dead Zone এ থাকে, তখন access করলে ReferenceError আসে। Function declaration পুরোটাই hoist হয়, তাই আগে call করা যায়। কিন্তু function expression শুধু variable এর নিয়ম মানে, `var` দিয়ে লিখলে আগে call করলে TypeError আসে।"

### 🔴 2. Temporal Dead Zone
> "TDZ হলো scope এর শুরু থেকে `let` বা `const` declaration এর লাইন পর্যন্ত সময়, যখন variable টা আছে কিন্তু access করা যায় না, করলে ReferenceError আসে। এটা আছে কারণ `var` এর নীরব `undefined` আচরণ bug লুকিয়ে রাখত। TDZ ভুলটা সাথে সাথে error দিয়ে ধরিয়ে দেয়। প্রমাণ হিসেবে বলা যায়, ভেতরের block এ `let a` থাকলে বাইরের `a` ও পাওয়া যায় না, কারণ `let` hoist হয়ে বাইরেরটাকে আড়াল করে।"

### 🔴 3. Execution Context
> "Execution context হলো এমন একটা পরিবেশ যেখানে কোড চলে। এতে variable, `this` আর outer scope এর reference থাকে। দুই ধরনের আছে: Global, যেটা একবার তৈরি হয়, আর Function, যেটা প্রতিটা function call এ নতুন তৈরি হয়। প্রতিটার দুটো ধাপ: creation phase, যেখানে variable আর function memory তে বসে, আর execution phase, যেখানে লাইন ধরে কোড চলে। Context গুলো call stack এ একটার উপর একটা জমে, function শেষ হলে সরে যায়।"

### 🔴 4. Call Stack, Task Queue, Microtask Queue
> "Call stack এ যেখানে কোড execute হয়। Async callback গুলো দুই ধরনের queue তে জমা হয়। Macrotask বা task queue এ থাকে setTimeout, setInterval, I/O আর UI event এর callback। Microtask queue এ থাকে Promise এর then, catch, finally আর await এর পরের অংশ। Call stack খালি হলে event loop আগে microtask queue পুরোটা খালি করে, তারপর task queue থেকে একটা task নেয়, আবার microtask খালি করে, এভাবে চলতে থাকে।"

### 🔴 5. Promise কেন setTimeout এর আগে চলে
> "কারণ Promise এর callback microtask queue তে যায়, আর setTimeout এর callback task queue তে যায়। Event loop সবসময় একটা task শেষ হওয়ার পর microtask queue পুরোটা খালি করে, তারপর পরের task নেয়। তাই `setTimeout(0)` আগে লিখলেও Promise এর then আগে চলে। একটা সতর্কতা হলো, microtask এ বারবার নতুন microtask যোগ হতে থাকলে task queue starve হয়ে UI আটকে যেতে পারে।"

### 🔴 6. Lexical Scope
> "Lexical scoping মানে একটা function কোন variable access করতে পারবে সেটা নির্ভর করে কোড কোথায় লেখা হয়েছে তার উপর, কোথা থেকে call হয়েছে তার উপর নয়। Inner function প্রথমে নিজের scope, তারপর outer scope, এভাবে global পর্যন্ত scope chain ধরে variable খোঁজে। না পেলে ReferenceError দেয়। `var` function-scoped আর `let`, `const` block-scoped।"

### 🟡 7. Undeclared vs undefined
> "`undefined` মানে variable declare করা আছে কিন্তু কোনো মান দেওয়া হয়নি। Undeclared মানে variable ই তৈরি হয়নি, তাকে access করলে ReferenceError আসে। একটা মজার বিষয়, undeclared variable এ `typeof` ব্যবহার করলে error হয় না, `'undefined'` string ফেরত আসে।"

### 🔴 8. Closure ও Memory Leak
> "Closure হলো এমন একটা function যে তার lexical scope এর variable গুলো মনে রাখে, এমনকি outer function শেষ হয়ে গেলেও। যেমন counter factory তে inner function টা `count` ধরে রাখে। এটা দিয়ে private data, memoization, debounce বানানো যায়। Closure memory leak করতে পারে যখন সেটা বড় object এর reference ধরে রাখে যা আর দরকার নেই, যেমন remove না করা event listener। সমাধান হলো শুধু দরকারি অংশ ধরা আর কাজ শেষে reference null করা বা listener সরানো। আর ক্লাসিক ফাঁদ হলো `var` দিয়ে loop এ setTimeout, যেখানে সবগুলো একই `i` শেয়ার করে, `let` দিয়ে সেটা ঠিক হয় কারণ `let` প্রতি iteration এ নতুন binding বানায়।"

# 📦 PART 2: Functions ও `this`

> প্রশ্ন 9–13 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 10, 11
> 🟡 **Good (মূল ধারণা):** 9, 12, 13

---

### 🟡 9. Function Declaration vs Expression
> "Function declaration `function foo() {}` আকারে লেখা হয় এবং পুরোটা hoist হয়, তাই আগে call করা যায়। Function expression হলো function কে variable এ রাখা, যেমন `const foo = function() {}` বা arrow function। এটা hoist হয় না, লাইনে পৌঁছালে তবেই তৈরি হয়, তাই আগে call করলে error আসে। Expression এ নাম optional, declaration এ বাধ্যতামূলক।"

### 🔴 10. `this`
> "`this` নির্ভর করে function কে কীভাবে call করা হয়েছে তার উপর। Method হিসেবে call করলে `this` হলো সেই object। সাধারণ function call এ strict mode এ `undefined`, নইলে global object। `new` দিয়ে call করলে নতুন তৈরি object। Arrow function এর নিজের `this` নেই, সে যেখানে লেখা হয়েছে সেখানকার `this` নেয়, তাই callback বা setTimeout এর ভেতরে `this` ধরে রাখতে arrow ভালো, কিন্তু object এর method হিসেবে arrow ব্যবহার করা ঠিক না। Event handler এ সাধারণ function হলে `this` হলো সেই element, arrow হলে বাইরের scope এর `this`।"

### 🔴 11. `bind`, `call`, `apply`
> "তিনটাই `this` কে জোর করে সেট করার জন্য। `call` function সাথে সাথে চালায়, আর্গুমেন্ট কমা দিয়ে আলাদা আলাদা দিতে হয়। `apply` ও সাথে সাথে চালায়, কিন্তু আর্গুমেন্ট array আকারে নেয়। `bind` function চালায় না, `this` বাঁধা একটা নতুন function ফেরত দেয় যেটা পরে call করা যায়। `bind` সাধারণত callback এ method পাঠালে `this` হারানো ঠিক করতে ব্যবহার করি, আর partial application এও কাজে লাগে।"

### 🟡 12. Currying
> "Currying হলো একাধিক আর্গুমেন্টের function কে এমনভাবে ভাঙা যাতে প্রতিটা call একটা করে আর্গুমেন্ট নেয়, যেমন `add(1)(2)(3)`। এটা closure এর উপর নির্ভর করে। সুবিধা হলো আংশিক আর্গুমেন্ট দিয়ে reusable specialized function বানানো যায়, যেমন `const add10 = add(10)`। আর function composition এও সুবিধা হয়।"

### 🟡 13. Function Composition
> "Function composition হলো ছোট ছোট function জোড়া লাগিয়ে বড় কাজ করা, যেখানে একটার output পরেরটার input হয়। `compose` ডান থেকে বামে চালায়, `pipe` বাম থেকে ডানে। এতে কোড ছোট, পুনঃব্যবহারযোগ্য আর test করা সহজ হয়, বিশেষ করে pure function এর সাথে।"

# 📦 PART 3: Objects ও Prototype

> প্রশ্ন 14–18 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 14, 16, 17
> 🟡 **Good (মূল ধারণা):** 15, 18

---

### 🔴 14. Prototype Chain
> "প্রতিটা JavaScript object এর একটা লুকানো link থাকে যা তার prototype object কে নির্দেশ করে। সেই prototype এরও prototype থাকে, এভাবে `null` পর্যন্ত যায়। একে বলে prototype chain। কোনো property object এ না পেলে JS chain ধরে উপরে খোঁজে। যেমন array তে `map` method নেই, আছে `Array.prototype` এ। এর সুবিধা হলো method গুলো সব instance এ share হয়, তাই memory বাঁচে।"

### 🟡 15. `Object.create()` vs Class
> "দুটোই ভেতরে prototype chain ই ব্যবহার করে। `class` হলো prototype এর উপর syntactic sugar, যেখানে constructor, `extends`, `super` আর private field পাওয়া যায়, তাই বড় প্রজেক্টে পরিচিত OOP style এ ভালো। `Object.create(proto)` সরাসরি একটা object বানায় যার prototype আমি ঠিক করে দিই, সহজ delegation এর জন্য বা prototype ছাড়া object (`Object.create(null)`) দরকার হলে কাজে লাগে।"

### 🔴 16. Property Lookup
> "`obj.x` লিখলে JS আগে object এর নিজের property তে খোঁজে। না পেলে তার prototype এ, তারপর prototype এর prototype এ, `null` পর্যন্ত। কোথাও না পেলে `undefined`। নিজের property থাকলে সেটা prototype এরটাকে shadow করে। লেখার সময় সবসময় নিজের object এ লেখা হয়, prototype বদলায় না। নিজের property চেক করতে `Object.hasOwn` আর chain সহ চেক করতে `in` ব্যবহার করি।"

### 🔴 17. Shallow vs Deep Copy
> "Shallow copy শুধু উপরের স্তর কপি করে, nested object গুলো আগের মতোই একই reference শেয়ার করে। Spread `{...obj}` আর `Object.assign` shallow। তাই nested property বদলালে মূল object ও বদলে যায়। Deep copy পুরো গাছ নতুন করে বানায়। আধুনিক সবচেয়ে ভালো উপায় `structuredClone`। `JSON.parse(JSON.stringify())` ও চলে কিন্তু Date, undefined, function এ সমস্যা করে আর circular reference এ error দেয়। বড় প্রজেক্টে lodash এর `cloneDeep`।"

### 🟡 18. `Object.freeze()` vs `Object.seal()`
> "দুটোতেই নতুন property যোগ করা আর মুছে ফেলা যায় না। পার্থক্য হলো `seal` করা object এ আগের property এর মান বদলানো যায়, কিন্তু `freeze` করলে মান বদলানোও যায় না। দুটোই shallow, মানে nested object এর ভেতরের মান তখনো বদলানো যায়। Deep freeze চাইলে recursive ভাবে করতে হয়।"

# 📦 PART 4: Asynchronous JavaScript (Promise, async/await)

> প্রশ্ন 19–25 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 19, 20, 21, 23
> 🟡 **Good (মূল ধারণা):** 22, 24, 25

---

### 🔴 19. Promise States ও Chaining
> "Promise এর তিনটা state: pending, fulfilled আর rejected। একবার settled হলে আর বদলায় না। প্রতিটা `.then()` একটা নতুন Promise return করে, তাই chaining সম্ভব। Handler এ value return করলে পরের then সেটা পায়, Promise return করলে সেটা resolve হওয়া পর্যন্ত অপেক্ষা করে, আর error throw করলে সোজা কাছের catch এ যায়। `finally` সফল বা ব্যর্থ দুই ক্ষেত্রেই চলে। একটা সাধারণ ভুল হলো then এর ভেতরে return ভুলে যাওয়া।"

### 🔴 20. async/await vs Promise
> "async/await হলো Promise এর উপর syntactic sugar, ভেতরে একই Promise আর microtask ব্যবহার হয়। async function সবসময় Promise return করে। `await` function execution কে থামিয়ে দেয় আর বাকি অংশ microtask হিসেবে schedule করে, কিন্তু main thread block করে না। এতে কোড synchronous এর মতো পড়া যায় আর error handling try/catch দিয়ে হয়। তবে independent কাজ পরপর await না করে `Promise.all` দিয়ে parallel চালানো উচিত, নইলে অকারণে ধীর হয়।"

### 🔴 21. async function এ await না করলে
> "await না করলে async function চলে ঠিকই, কিন্তু আমি একটা pending Promise পাই, আসল value না। এতে তিনটা সমস্যা: ভুল ফলাফল পাওয়া, কাজ শেষ হওয়ার আগেই পরের কোড চলে যাওয়া, আর error হারিয়ে unhandled rejection হওয়া, যেটা Node এ process crash করতে পারে। একটা বিখ্যাত ফাঁদ হলো `forEach` এ async callback, যেটা await এর জন্য অপেক্ষা করে না। সেখানে `for...of` বা `Promise.all` ব্যবহার করতে হয়। ইচ্ছা করে না await করলে অন্তত `.catch` লাগাই।"

### 🟡 22. `Promise.all()` নিজে বানানো
> "আমি একটা নতুন Promise return করি। ভেতরে প্রতিটা input কে `Promise.resolve` দিয়ে wrap করে `.then` ধরি, যাতে সাধারণ value ও চলে। সফল হলে `results[index]` তে value রাখি, push না, যাতে output এর ক্রম input এর মতো থাকে, আর একটা counter বাড়াই। Counter input এর সমান হলে resolve করি। কোনো একটা reject করলে সাথে সাথে পুরো Promise কে reject করি। খালি array এর জন্য সাথে সাথে খালি array দিয়ে resolve করি।"

### 🔴 23. `all`, `allSettled`, `race`, `any`
> "চারটাই একাধিক Promise একসাথে চালায়, ফলাফলের নিয়ম আলাদা। `all` সব সফল হলে সব value এর array দেয়, একটা fail করলেই reject করে। `allSettled` কখনো reject করে না, সবার শেষে প্রতিটার status আর value বা reason দেয়, bulk কাজের রিপোর্টে কাজে লাগে। `race` যেটা সবার আগে settle হয় তার ফলাফল দেয়, সফল বা ব্যর্থ যাই হোক, timeout বানাতে ব্যবহার হয়। `any` প্রথম সফলটা দেয়, সবাই fail করলে তবেই AggregateError দিয়ে reject করে, একাধিক server থেকে যে আগে সাড়া দেয় তার ডেটা নিতে ভালো।"

### 🟡 24. Concurrency Limit সহ async কাজ
> "সব request একসাথে পাঠালে server বা API ডুবে যায়, তাই একসাথে সর্বোচ্চ N টা চালাতে হয়। আমি N সংখ্যক worker function বানাই, প্রতিটা একটা shared index থেকে পরের task নিয়ে await করে, শেষ হলে আবার পরেরটা নেয়। সবগুলো worker কে `Promise.all` দিয়ে চালাই। ফলাফল index ধরে রাখি যাতে ক্রম ঠিক থাকে। বাস্তবে আমি `p-limit` বা `p-queue` এর মতো library ও ব্যবহার করি। Task গুলো Promise না, Promise return করা function হতে হয়, নইলে তৈরির সময়ই চালু হয়ে যায়।"

### 🟡 25. Callback Hell
> "Callback hell হলো callback এর ভেতরে callback এর ভেতরে callback, যাতে কোড ডানে সরতে থাকে, পড়া কঠিন হয় আর error handling ছড়িয়ে যায়। আধুনিক সমাধান হলো Promise chaining, আরও ভালো async/await যেখানে কোড সোজা লাইনে পড়া যায় আর এক জায়গায় try/catch দিয়ে সব error ধরা যায়। পুরনো callback API কে `util.promisify` দিয়ে Promise বানানো যায়, আর কোড ছোট named function এ ভাঙলেও পড়তে সুবিধা হয়।"

# 📦 PART 5: Performance, Memory ও Advanced

> প্রশ্ন 26–31 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 28, 31
> 🟡 **Good (মূল ধারণা):** 26, 27, 29, 30

---

### 🟡 26. JavaScript এ Memory Leak
> "Memory leak হলো যখন আর দরকার নেই এমন memory মুক্ত হয় না কারণ কোথাও reference ধরে রাখা আছে। সাধারণ কারণ: global variable এ জমানো, সীমাহীন cache, remove না করা event listener, clear না করা setInterval, বড় ডেটা ধরে রাখা closure, আর DOM থেকে সরানো কিন্তু variable এ ধরা detached node। Debug করতে Chrome DevTools এ দুটো heap snapshot নিয়ে compare করি। প্রতিরোধে LRU cache, সবসময় cleanup, আর দরকারে WeakMap বা WeakRef ব্যবহার করি।"

### 🟡 27. Garbage Collection
> "JavaScript এ garbage collection হয় mark-and-sweep algorithm এ। Engine roots থেকে শুরু করে, যেমন global object আর চলমান function এর variable, যা যা পৌঁছানো যায় সেগুলো mark করে। যেগুলো mark হয়নি সেগুলো আর কোথাও থেকে পৌঁছানো যায় না, তাই sweep করে মুছে memory ফেরত দেয়। মূল ধারণা reachability। এ কারণে circular reference ও সমস্যা না, কারণ roots থেকে না পৌঁছালে সবই আবর্জনা। V8 generational GC ব্যবহার করে: নতুন object young generation এ ঘন ঘন পরিষ্কার হয়, যারা টিকে যায় old generation এ যায়।"

### 🔴 28. Debounce vs Throttle
> "দুটোই function কত ঘন ঘন চলবে তা নিয়ন্ত্রণ করে, দুটোই closure দিয়ে বানানো। Debounce অপেক্ষা করে, প্রতিবার event এলে timer reset হয়, আর event থামার নির্দিষ্ট সময় পর একবার চলে। যেমন search box এ user টাইপ থামালে তবেই API call। Throttle নির্দিষ্ট সময়ের মধ্যে সর্বোচ্চ একবার চলতে দেয়, যতই event আসুক। যেমন scroll বা mousemove handler। সংক্ষেপে, debounce 'থামার পর', throttle 'প্রতি এত সময়ে একবার'।"

### 🟡 29. Immutability
> "Immutability মানে ডেটা তৈরির পর বদলানো হয় না, বদলাতে হলে নতুন কপি বানানো হয়, যেমন spread বা `filter` দিয়ে। এটা জরুরি কারণ কোড predictable হয়, কেউ লুকিয়ে ডেটা বদলে bug বানাতে পারে না, debug আর undo/redo সহজ হয়। React এ বিশেষ গুরুত্বপূর্ণ কারণ state বদল চেনা হয় reference তুলনা দিয়ে, mutate করলে reference একই থাকে আর UI update হয় না। Deep nested ডেটার জন্য Immer ব্যবহার করা যায়।"

### 🟡 30. Pure Functions
> "Pure function এর দুটো শর্ত: একই input এ সবসময় একই output, আর কোনো side effect নেই, মানে বাইরের কিছু বদলায় না, যেমন global variable, DB, DOM বা input argument mutate। উদাহরণ `add(a, b)`। `Date.now()` বা বাইরের variable বদলানো function impure। সুবিধা হলো test করা সহজ, predictable, cache করা যায় আর parallel এ নিরাপদ।"

### 🔴 31. Event Delegation
> "Event delegation হলো প্রতিটা child element এ আলাদা listener না বসিয়ে তাদের parent এ একটাই listener বসানো। Event bubble করে parent এ আসে, আর `event.target` বা `closest()` দিয়ে বুঝে নিই কোথায় click হয়েছে। সুবিধা: কম memory আর কম listener, আর পরে dynamically যোগ হওয়া element এও কাজ করে, নতুন করে listener বসাতে হয় না। সীমা: `focus` আর `blur` এর মতো যে event bubble করে না তাতে কাজ করে না, আর কেউ `stopPropagation` দিলে ভেঙে যায়।"

---

# Part B: TypeScript

---

# 📦 PART 6: TypeScript Types

> প্রশ্ন 32–38 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 32, 33, 36
> 🟡 **Good (মূল ধারণা):** 34, 35, 37, 38

---

### 🔴 32. `type` vs `interface`
> "দুটোই object এর shape বর্ণনা করতে পারে। Interface এর বিশেষত্ব declaration merging, একই নামে দুবার লিখলে মিশে যায়, আর `extends` দিয়ে সহজে বাড়ানো যায়। `type` বেশি নমনীয়: union, tuple, primitive alias, mapped আর conditional type শুধু `type` দিয়েই সম্ভব। আমার নিয়ম: object আর class এর shape ও public API এর জন্য interface, union বা complex type এর জন্য type। টিমে একটা convention ঠিক করে সেটা মেনে চলা সবচেয়ে গুরুত্বপূর্ণ।"

### 🔴 33. Union vs Intersection
> "Union `A | B` মানে মান A হতে পারে অথবা B, যেমন `string | number`, বা API response এ success অথবা error। Union এ সরাসরি শুধু দুটোর common member ব্যবহার করা যায়, বাকির জন্য narrowing লাগে। Intersection `A & B` মানে মান A এবং B দুটোর সব বৈশিষ্ট্যই থাকতে হবে, যেমন Person আর Employee মিলিয়ে Staff। এটা mixin বা props extend করতে কাজে লাগে।"

### 🟡 34. Type Inference
> "Type inference মানে TypeScript মান দেখে নিজেই type বুঝে নেয়, আমাকে সবসময় লিখতে হয় না। যেমন `let x = 5` এ `number`, আর function এর return type ও ভেতরের কোড থেকে বুঝে নেয়। আমি সাধারণত function এর parameter এ type লিখি আর local variable বা return এ inference এর উপর ভরসা করি, তবে public API তে return type স্পষ্ট লিখি।"

### 🟡 35. Optional Chaining ও Nullish Coalescing
> "Optional chaining `?.` দিয়ে nested property নিরাপদে access করা যায়, মাঝে কোনো মান `null` বা `undefined` হলে error না দিয়ে পুরো expression `undefined` হয়। Nullish coalescing `??` শুধু `null` বা `undefined` হলে ডিফল্ট মান দেয়। পার্থক্য `||` থেকে: `||` সব falsy মান, যেমন `0`, খালি string, `false` এ ডিফল্টে চলে যায়, যা ভুল হতে পারে। `port ?? 3000` এ port যদি `0` হয় সেটা বৈধ মান হিসেবে থাকে।"

### 🔴 36. `any` vs `unknown` vs `never`
> "`any` মানে type checking পুরো বন্ধ, যা খুশি করা যায়, তাই runtime এ crash হতে পারে, এড়িয়ে চলা উচিত। `unknown` মানে ধরনটা জানি না, কিন্তু ব্যবহারের আগে `typeof` বা type guard দিয়ে যাচাই করতে বাধ্য, তাই এটা নিরাপদ বিকল্প। আমি API বা user input এর অজানা ডেটায় `unknown` ব্যবহার করি। `never` মানে এমন মান যা কখনো ঘটে না, যেমন সবসময় error throw করা function এর return type। এটা exhaustive check এও কাজে লাগে, যেখানে সব case handle না করলে compile error আসে।"

### 🟡 37. Structural Typing
> "TypeScript এ type এর compatibility ঠিক হয় তার গঠন বা shape দেখে, নাম দেখে না। দুটো আলাদা নামের type এর গঠন একই হলে তারা compatible। একে duck typing ও বলা যায়। Java বা C# এর মতো nominal typing এ নাম মিলতে হয়। একটা বিশেষ বিষয়: সরাসরি object literal দিলে TypeScript বাড়তি property ধরে excess property error দেয়, কিন্তু variable থেকে দিলে বাড়তি property থাকলেও চলে।"

### 🟡 38. `readonly`
> "`readonly` property বা array কে compile time এ পরিবর্তন থেকে রক্ষা করে, বদলাতে গেলে compile error আসে। `Readonly<T>` পুরো type কে readonly বানায়, আর `as const` দিয়ে literal আর readonly দুটোই পাওয়া যায়। তবে এটা শুধু compile time এর সুরক্ষা, JavaScript এ compile হলে এর কোনো চিহ্ন থাকে না। Runtime এও আটকাতে চাইলে `Object.freeze` লাগে। আর `const` variable এ নতুন মান বসানো আটকায়, `readonly` আটকায় property বদলানো।"

# 📦 PART 7: TypeScript Generics ও Advanced Types

> প্রশ্ন 39–44 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 39, 41, 44
> 🟡 **Good (মূল ধারণা):** 40, 42
> ⚪ **সময় থাকলে:** 43

---

### 🔴 39. Generics ও Constraints
> "Generic হলো type এর জন্য parameter, যাতে একটা function বা type বিভিন্ন type এর সাথে কাজ করে কিন্তু type safety হারায় না। `any` এ input আর output এর সম্পর্ক হারিয়ে যায়, generic এ সেটা ধরে রাখা যায়। যেমন `function first<T>(arr: T[]): T`। Constraint দিয়ে সীমা বেঁধে দেওয়া যায়, যেমন `T extends { length: number }`, তখন শুধু length আছে এমন type ই চলবে। `K extends keyof T` দিয়ে object এর বৈধ key ই শুধু নেওয়া যায়। আর default generic দিয়ে ডিফল্ট type দেওয়া যায়, যেমন `ApiResponse<T = unknown>`। বেশিরভাগ ক্ষেত্রে TypeScript argument থেকেই T infer করে নেয়।"

### 🟡 40. `void` vs `never`
> "`void` হলো এমন function এর return type যেটা শেষ হয় কিন্তু দরকারি কোনো মান দেয় না, যেমন console.log করা function। `never` হলো এমন function যেটা কখনো স্বাভাবিকভাবে শেষই হয় না, হয় সবসময় error throw করে নয়তো অসীম loop। সংক্ষেপে, void মানে 'ফেরত দেয় কিন্তু মান নেই', never মানে 'কখনো ফেরতই আসে না'।"

### 🔴 41. Utility Types
> "Utility types হলো TypeScript এর built-in generic helper, যা বিদ্যমান type থেকে নতুন type বানায়। `Partial<T>` সব field optional করে, update payload এ কাজে লাগে। `Required<T>` সব বাধ্যতামূলক করে। `Pick<T, K>` শুধু নির্বাচিত field রাখে, `Omit<T, K>` কিছু field বাদ দেয়, যেমন `Omit<User, 'password'>` দিয়ে নিরাপদ user type। `Record<K, V>` key-value map এর type বানায়। এছাড়া `Readonly`, `ReturnType`, `Awaited` ও খুব কাজের।"

### 🟡 42. `keyof` ও `typeof` একসাথে
> "`typeof` যখন type position এ ব্যবহার করি তখন একটা variable বা মান থেকে তার type বের করে। `keyof` কোনো type এর সব key এর union দেয়। দুটো একসাথে `keyof typeof obj` লিখলে একটা object এর key গুলোর union পাই। যেমন config object থেকে `'host' | 'port'`। `as const` এর সাথে মিলিয়ে `typeof ROLES[keyof typeof ROLES]` দিয়ে object এর মান থেকে union type বানানো যায়, তাই একটাই source of truth থাকে।"

### ⚪ 43. Conditional Types
> "Conditional type হলো type level এর if-else, `T extends U ? X : Y`। T যদি U এর সাথে মেলে তাহলে X, নইলে Y। `infer` কীওয়ার্ড দিয়ে শর্তের ভেতর থেকে কোনো type বের করা যায়, যেমন array এর element type। `ReturnType`, `Awaited`, `NonNullable` এই ধারণাতেই বানানো। আমি সাধারণত library বা reusable utility type লেখার সময় এটা ব্যবহার করি, সাধারণ অ্যাপ কোডে কম লাগে।"

### 🔴 44. Discriminated Union
> "Discriminated union হলো এমন union যেখানে প্রতিটা variant এ একটা সাধারণ literal field থাকে, যেমন `status: 'success' | 'error' | 'loading'`। সেই field চেক করলে TypeScript নিজেই narrow করে বুঝে নেয় কোন variant এ আছি, তাই `success` এ `data` আর `error` এ `message` নিরাপদে পাওয়া যায়। বড় সুবিধা exhaustive checking: switch এর default এ `never` টাইপে assign করলে নতুন variant যোগ করে handle না করলে compile error আসে। এটা API response, UI state আর Redux action এ খুব কাজে লাগে।"

# 📦 PART 8: TypeScript Real-world

> প্রশ্ন 45–48 | ⏱️ আনুমানিক সময়: ~২৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 45, 47
> 🟡 **Good (মূল ধারণা):** 46, 48

---

### 🔴 45. API Response নিরাপদে Type করা ও অজানা ডেটা সামলানো
> "শুধু `as User` দিয়ে cast করলে TypeScript চুপ করে যায় কিন্তু backend ভুল ডেটা পাঠালে runtime এ crash হবে, কারণ type runtime এ থাকে না। তাই আমি response কে আগে `unknown` ধরি, তারপর Zod এর মতো schema দিয়ে validate করি। Zod এর সুবিধা হলো `z.infer` দিয়ে একই schema থেকে type পাওয়া যায়, তাই runtime validation আর type এর একটাই source of truth থাকে। ভুল ডেটা এলে সেখানেই error ধরা পড়ে, অ্যাপের ভেতরে ঢুকতে পারে না। আর success বা error response কে discriminated union দিয়ে মডেল করি।"

### 🟡 46. TypeScript কীভাবে Runtime Error কমায়
> "TypeScript compile time এ অনেক bug ধরে ফেলে: null বা undefined এর ভুল ব্যবহার (`strictNullChecks` চালু থাকলে), ভুল type এ function call, property নামের typo। Refactor ও নিরাপদ হয়, কারণ নাম বদলালে সব জায়গায় error দেখায়। তবে সীমা আছে: এটা শুধু compile time এ কাজ করে, তাই API বা user input এর মতো external ডেটার জন্য runtime validation লাগে। আর আমি সবসময় `strict: true` চালু রাখি।"

### 🔴 47. TS → JS Compile ও Runtime এ Types
> "TypeScript compiler `tsc` প্রথমে কোড parse করে, type check করে, তারপর types মুছে ফেলে (type erasure) এবং `tsconfig` এর target অনুযায়ী সাধারণ JavaScript বের করে। তাই runtime এ types এর কোনো অস্তিত্ব নেই। এর ফলে interface দিয়ে `instanceof` বা `typeof` চেক করা যায় না, আর runtime validation এর জন্য Zod এর মতো কিছু লাগে। ব্যতিক্রম `enum` আর `class`, কারণ এদের জন্য সত্যিকারের JavaScript কোড তৈরি হয়। বাস্তবে build এ `tsc`, `esbuild` বা `swc` আর development এ `tsx` ব্যবহার হয়।"

### 🟡 48. TypeScript এর Trade-offs
> "সুবিধা: bug আগেই ধরা পড়ে, refactor নিরাপদ, autocomplete আর living documentation পাওয়া যায়, আর বড় টিমে কোড বোঝা সহজ হয়। অসুবিধা: শেখার খরচ আছে, build step ও config যোগ হয়, বড় প্রজেক্টে compile সময় বাড়ে, আর জটিল type লিখতে গিয়ে overengineering হতে পারে। সবচেয়ে বড় বিষয়, এটা runtime নিরাপত্তা দেয় না, external ডেটায় validation লাগে। আমার মতে ছোট স্ক্রিপ্টে overhead, কিন্তু বড় বা দীর্ঘমেয়াদী প্রজেক্টে লাভ অনেক বেশি, আমার Next.js আর Prisma প্রজেক্টগুলোতে এটাই দেখেছি।"

---

# 💡 কিছু বাড়তি বলার লাইন

**উত্তর শেষে বলতে পারো:**
> "চাইলে এর একটা বাস্তব উদাহরণ দিয়ে দেখাতে পারি।"

**কিছু না জানলে:**
> "এটা আমি সরাসরি ব্যবহার করিনি, তবে যতটুকু জানি... আর আমার ধারণা এভাবে কাজ করে..."

**নিজের অভিজ্ঞতা জুড়তে (সবচেয়ে বেশি impression):**
- Closure/debounce: "আমি search input এ debounce ব্যবহার করেছি যাতে প্রতি keystroke এ API call না যায়।"
- Promise.all: "Dashboard এর ডেটা আনতে independent query গুলো `Promise.all` দিয়ে parallel চালিয়েছি।"
- TypeScript + Zod: "আমার প্রজেক্টে API input validate করতে Zod ব্যবহার করি, আর Prisma থেকে type পাই।"
- Utility types: "`Omit<User, 'password'>` দিয়ে response type বানাই।"
- Memory/queue: "ভারী কাজ background job এ পাঠিয়েছি যাতে request ধীর না হয়।"

শুভকামনা! 🚀