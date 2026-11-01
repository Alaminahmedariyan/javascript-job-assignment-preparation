# 🎤 Node.js: Interview এ যেভাবে বলবে

> শুধু **মুখে বলার উত্তর**। ব্যাখ্যা ও code এর জন্য `nodejs-interview-explain.md` দেখো।
> প্রতিটা ৩০-৬০ সেকেন্ডের। নিজের ভাষায় একটু বদলে বলবে, মুখস্থ শোনালে ধরা পড়ে।

> **রঙের মানে:** 🔴 অবশ্যই পড়ো | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে

---

# 📦 PART 1: Event Loop ও Async এর ভেতরের কাজ

> প্রশ্ন 1–4 | 🔴 Must: 1, 2, 3 | 🟡 Good: 4

---

### 🔴 1. Event Loop Architecture
> "Node.js single-threaded, কিন্তু event loop এর কারণে non-blocking। Event loop একটা অসীম loop যেটা ৬টা phase এ ঘোরে: timers, pending callbacks, idle/prepare, poll, check, আর close callbacks। I/O এর মতো ধীর কাজ Node libuv এর মাধ্যমে OS বা thread pool এ পাঠিয়ে দেয়, কাজ শেষ হলে callback queue তে আসে, আর call stack খালি থাকলে event loop সেটা চালায়। প্রতিটা phase এর মাঝে nextTick আর Promise microtask গুলো আগে চলে। তাই আমি সবসময় event loop কে block না করার চেষ্টা করি, ভারী CPU কাজ worker thread এ পাঠাই।"

### 🔴 2. nextTick vs setImmediate vs setTimeout
> "তিনটাই কাজ পরে চালানোর জন্য, কিন্তু কখন চলবে আলাদা। `process.nextTick` চলে বর্তমান operation শেষ হওয়ার সাথে সাথে, event loop এর কোনো phase এ যাওয়ার আগেই, এর priority সবচেয়ে বেশি। `setTimeout` চলে Timers phase এ, আর `setImmediate` চলে Check phase এ, অর্থাৎ Poll এর ঠিক পরে। I/O callback এর ভেতরে `setImmediate` সবসময় `setTimeout(0)` এর আগে চলে। আর nextTick বেশি recursive ভাবে ব্যবহার করলে event loop starve হয়, তাই সাবধানে ব্যবহার করি।"

### 🔴 3. Async Operation ভেতরে কীভাবে চলে
> "Node এ কোনো async কাজ, যেমন file read বা DB call, এলে V8 সেটা নিজে করে না, libuv এর কাছে delegate করে দেয়। Network I/O libuv সরাসরি OS এর native async mechanism দিয়ে handle করে, যেমন Linux এ epoll। আর file system, DNS, crypto এর মতো কাজ তার thread pool এ চালায়, যেটার default size ৪। কাজ শেষ হলে callback event queue তে যায়, আর event loop call stack খালি পেলে সেটা চালিয়ে দেয়। এভাবে main thread কখনো আটকে থাকে না।"

### 🟡 4. libuv এর ভূমিকা
> "libuv হলো C এ লেখা একটা cross-platform library যেটা Node এর asynchronous I/O এর ভিত্তি। এটা event loop implement করে, thread pool চালায়, আর file system, networking, DNS, timer এর মতো কাজ handle করে। V8 শুধু JavaScript execute করে, কিন্তু I/O এর আসল কাজ হয় libuv এর মাধ্যমে। আর Windows, Linux, macOS এর আলাদা আলাদা OS API কে এটা একটা unified interface এ আনে, তাই Node cross-platform।"

---

# 📦 PART 2: Streams ও ফাইল হ্যান্ডলিং

> প্রশ্ন 5–7 | 🔴 Must: 5, 6 | 🟡 Good: 7

---

### 🔴 5. Streams ও ধরন
> "Stream হলো ডেটাকে পুরোটা memory তে না এনে ছোট ছোট chunk এ পড়া বা লেখার পদ্ধতি। এতে memory কম লাগে আর বড় ফাইল efficiently handle করা যায়। Node এ চার ধরনের stream আছে: Readable যেখান থেকে ডেটা পড়ি, Writable যেখানে লিখি, Duplex যেটা দুটোই পারে যেমন TCP socket, আর Transform যেটা ডেটা বদলে দেয় যেমন gzip compression। আমি সাধারণত `pipeline()` দিয়ে এগুলো জোড়া দিই কারণ এটা error handling আর cleanup ঠিকমতো করে।"

### 🔴 6. বড় ফাইল আপলোড
> "বড় ফাইলের ক্ষেত্রে মূল নিয়ম হলো পুরো ফাইল memory তে লোড না করা। আমি streaming ব্যবহার করি, যেমন busboy দিয়ে chunk আসামাত্র disk বা cloud এ পাঠিয়ে দিই। সবচেয়ে scalable উপায় হলো presigned URL, যেখানে client সরাসরি S3 এ আপলোড করে, আমার server এর উপর load পড়ে না। খুব বড় ফাইলের জন্য chunked বা resumable upload ব্যবহার করি যাতে fail হলে শুধু ওই অংশ আবার পাঠাতে হয়। সাথে file size limit, MIME type validation রাখি, আর ভারী প্রসেসিং BullMQ এর মতো background queue তে করি।"

### 🟡 7. Backpressure
> "Backpressure হলো যখন data producer এর গতি consumer এর গতির চেয়ে বেশি হয়, ফলে consumer এর buffer ভরে memory বাড়তে থাকে। Node এ `write()` যদি `false` return করে তার মানে buffer ভরে গেছে, তখন readable কে pause করে `drain` event এর জন্য অপেক্ষা করতে হয়। তবে আমি manual এ না গিয়ে `stream.pipeline()` ব্যবহার করি, কারণ এটা backpressure নিজে handle করে এবং error হলে সব stream properly cleanup করে।"

---

# 📦 PART 3: Scaling ও Concurrency

> প্রশ্ন 8–10 | 🔴 Must: 10 | 🟡 Good: 8, 9

---

### 🟡 8. Clustering
> "Node default এ একটা core ব্যবহার করে। Cluster module দিয়ে আমি একটা primary process থেকে CPU core সংখ্যা অনুযায়ী worker process fork করি, যারা একই port share করে এবং primary লোড ভাগ করে দেয়। এটা তখন কাজে লাগে যখন HTTP server এ অনেক request আসে এবং সব core ব্যবহার করতে চাই, বা একটা worker crash করলেও যেন service চলতে থাকে। তবে worker দের memory আলাদা, তাই session বা cache এর জন্য Redis এর মতো external store লাগে। Production এ আমি সাধারণত PM2 বা container orchestration ব্যবহার করি।"

### 🟡 9. Worker Threads vs Cluster
> "দুটোর উদ্দেশ্য আলাদা। Cluster আলাদা process বানায়, প্রতিটার memory আলাদা, আর এটা মূলত অনেক incoming request সামলাতে ও সব CPU core কাজে লাগাতে ব্যবহার হয়। Worker threads একই process এর ভেতরে হালকা thread বানায়, যারা SharedArrayBuffer দিয়ে memory share করতে পারে, আর এটা ভারী CPU-bound কাজ, যেমন image processing বা encryption, main thread থেকে সরিয়ে নিতে ব্যবহার হয়। I/O-heavy scaling এর জন্য cluster, আর CPU-heavy কাজের জন্য worker threads।"

### 🔴 10. Non-blocking I/O
> "Node এ I/O non-blocking কারণ যখন আমি file read বা DB call করি, Node কাজটা libuv এর মাধ্যমে OS বা thread pool এ পাঠিয়ে দিয়ে অপেক্ষা না করে পরের কোড চালাতে থাকে। কাজ শেষ হলে callback event queue তে আসে আর event loop সেটা চালায়। এর ফলে একটা single thread ই হাজার হাজার concurrent connection সামলাতে পারে। তবে এটা শুধু I/O-bound কাজের জন্য, CPU-bound কাজ main thread block করে, সেগুলোর জন্য আমি worker threads ব্যবহার করি।"

---

# 📦 PART 4: Modules, Memory, Performance ও Tools

> প্রশ্ন 11–15 | 🔴 Must: 11, 12, 13 | 🟡 Good: 14, 15

---

### 🔴 11. CommonJS vs ES Modules
> "CommonJS হলো Node এর original module system, `require` আর `module.exports` দিয়ে কাজ করে, synchronous ভাবে লোড হয়। ES Modules হলো official JavaScript standard, `import`/`export` ব্যবহার করে, asynchronous ভাবে লোড হয় এবং static analysis সম্ভব বলে tree-shaking করা যায়। ESM এ top-level await আছে আর live binding আছে, কিন্তু `__dirname` নেই। ESM থেকে CJS import করা যায়, উল্টোটা সরাসরি যায় না। আমি নতুন প্রজেক্টে ESM ব্যবহার করি, কারণ TypeScript আর Next.js এর মতো আধুনিক tooling সেটাকেই follow করে।"

### 🔴 12. Memory Leak ও Debug
> "Memory leak হলো যখন অব্যবহৃত memory garbage collector মুক্ত করতে পারে না কারণ কোথাও সেটার reference ধরে রাখা আছে, ফলে heap ধীরে ধীরে বেড়ে app crash করে। সাধারণ কারণ: unbounded cache বা global array, remove না করা event listener, clear না করা timer, আর closure এ বড় ডেটা ধরে রাখা। Debug করতে আমি আগে `process.memoryUsage()` দিয়ে heap এর trend দেখি, তারপর `--inspect` দিয়ে Chrome DevTools এ আলাদা সময়ে দুটো heap snapshot নিয়ে compare করি কোন object জমছে। প্রতিরোধে আমি LRU cache ব্যবহার করি আর resource সবসময় cleanup করি।"

### 🔴 13. Slow App Optimize
> "আমি সবসময় আগে profiling করি, আন্দাজে optimize করি না। Clinic.js বা APM দিয়ে আসল bottleneck বের করি। সাধারণত সমস্যা থাকে তিন জায়গায়: event loop block করা sync কাজ, ধীর database query, আর caching এর অভাব। Database এ আমি index, N+1 ঠিক করা, connection pooling আর pagination দেখি। বারবার লাগে এমন ডেটা Redis এ cache করি, independent async কাজ `Promise.all` দিয়ে parallel চালাই, ভারী কাজ BullMQ এর মতো queue তে পাঠাই, আর CPU-heavy কাজ worker thread এ। শেষে scaling এর জন্য cluster বা load balancer ব্যবহার করি।"

### 🟡 14. Event Emitters
> "EventEmitter হলো Node এর publish-subscribe pattern এর implementation। একটা object event emit করে, আর যারা সেই event এ listen করছে তারা react করে। মূল method হলো on, once, emit আর off। এর বড় সুবিধা হলো decoupling, যেমন অর্ডার সার্ভিস শুধু 'orderPlaced' emit করে, আর email, inventory, analytics আলাদা listener হিসেবে কাজ করে। Node এর streams, HTTP server, process এসবই EventEmitter এর উপর তৈরি। আর মনে রাখতে হয় যে `error` event এ listener না থাকলে process crash করে, আর listener গুলো synchronously চলে।"

### 🟡 15. `npm install` ভেতরে কী ঘটে
> "`npm install` চালালে প্রথমে npm `package.json`, `.npmrc` আর lock file পড়ে। তারপর Arborist দিয়ে পুরো dependency tree তৈরি করে, lock file থাকলে সুনির্দিষ্ট version ধরে, না থাকলে semver range থেকে। তারপর registry থেকে metadata আনে, local cache চেক করে, না থাকলে tarball download করে। Download করা ফাইলের SHA-512 integrity hash যাচাই করে, তারপর `node_modules` এ extract করে, duplicate কমাতে dependency গুলো hoist করে। এরপর preinstall, install, postinstall এর মতো lifecycle script চালায় আর শেষে `package-lock.json` আপডেট করে। CI/CD তে আমি `npm ci` ব্যবহার করি কারণ এটা lock file অনুযায়ী clean আর reproducible install দেয়।"

---

# 💡 বাড়তি টিপস

- **আগে ছোট উত্তর, তারপর:** "চাইলে আরো বিস্তারিত বলতে পারি।"
- **নিজের অভিজ্ঞতা জুড়ো:** "আমার প্রজেক্টে ভারী কাজ BullMQ queue তে পাঠিয়েছি", "Prisma তে N+1 ঠিক করতে `include` ব্যবহার করেছি।"
- **না জানলে:** "এটা সরাসরি ব্যবহার করিনি, তবে যতটুকু জানি..."

শুভকামনা! 🚀
