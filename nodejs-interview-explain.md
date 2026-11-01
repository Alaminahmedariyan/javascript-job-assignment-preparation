# 📗 Node.js Interview গাইড (বিস্তারিত ব্যাখ্যা)

> এই ফাইলে **বুঝে পড়ার** জন্য ব্যাখ্যা: উপমা → ব্যাখ্যা → code।
> মুখে বলার ছোট উত্তর আলাদা ফাইলে: `nodejs-interview-answers.md`

> **রঙের মানে:** 🔴 অবশ্যই পড়ো (প্রায় নিশ্চিত আসবে) | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে
> প্রতিটা প্রশ্নের heading এর পাশেই রঙ বসানো আছে।

---

## 📑 সূচিপত্র

| # | বিষয় | গুরুত্ব | Part |
|---|---|---|---|
| 1 | Event loop architecture | 🔴 | 1 |
| 2 | nextTick vs setImmediate vs setTimeout | 🔴 | 1 |
| 3 | Async operation ভেতরে কীভাবে চলে | 🔴 | 1 |
| 4 | libuv এর ভূমিকা | 🟡 | 1 |
| 5 | Streams ও তাদের ধরন | 🔴 | 2 |
| 6 | বড় ফাইল আপলোড | 🔴 | 2 |
| 7 | Backpressure | 🟡 | 2 |
| 8 | Clustering | 🟡 | 3 |
| 9 | Worker threads vs Cluster | 🟡 | 3 |
| 10 | Non-blocking I/O | 🔴 | 3 |
| 11 | CommonJS vs ES Modules | 🔴 | 4 |
| 12 | Memory leak ও debug | 🔴 | 4 |
| 13 | Slow app optimize করা | 🔴 | 4 |
| 14 | Event Emitter | 🟡 | 4 |
| 15 | `npm install` ভেতরে কী ঘটে | 🟡 | 4 |

## শুরুর আগে ৫টা শব্দ

- **Thread**: কাজ করার "কর্মী"। Single thread = একজনই কর্মী।
- **Blocking**: একটা কাজ শেষ না হওয়া পর্যন্ত বাকি সব থেমে থাকা।
- **Non-blocking**: কাজ শুরু করে অপেক্ষা না করে পরের কাজে চলে যাওয়া।
- **Callback**: "কাজ শেষ হলে এই function চালিয়ো" বলে রাখা function।
- **I/O**: ফাইল, database, network, মানে বাইরের জগতের সাথে কথা বলা। এগুলো ধীর কারণ উত্তরের জন্য অপেক্ষা করতে হয়।

---

# 📦 PART 1: Event Loop ও Async এর ভেতরের কাজ

> প্রশ্ন 1–4 | ⏱️ আনুমানিক সময়: ~৩৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 1, 2, 3
> 🟡 **Good (মূল ধারণা):** 4

---

## 🔴 1. Event Loop Architecture

### 🧠 উপমা
রেস্টুরেন্টে **একজনই ওয়েটার**। অর্ডার নিয়ে রান্নাঘরে দিয়ে আসে, রান্না হওয়া পর্যন্ত দাঁড়িয়ে থাকে না, পরের টেবিলে যায়। খাবার রেডি হলে ঘণ্টা বাজে, তখন পরিবেশন করে। এই "অনবরত ঘুরে দেখা" টাই event loop।

### 📖 ব্যাখ্যা
Node এ JavaScript কোড **একটাই thread এ** চলে। ধীর কাজ (file, DB) Node নিজে করে না, OS বা thread pool কে দিয়ে দেয়। কাজ শেষ হলে callback queue তে জমা পড়ে। Event loop দেখে: "call stack খালি? queue তে কিছু আছে? থাকলে চালাও।"

**৬টা phase (এই ক্রমে ঘোরে):**

| # | Phase | কী করে |
|---|---|---|
| 1 | **Timers** | সময় হয়ে যাওয়া `setTimeout`/`setInterval` চালায় |
| 2 | **Pending callbacks** | আগের round এ পিছিয়ে যাওয়া কিছু system callback |
| 3 | **Idle, Prepare** | শুধু Node এর ভেতরের কাজ |
| 4 | **Poll** | নতুন I/O event আনে ও I/O callback চালায়, সবচেয়ে গুরুত্বপূর্ণ |
| 5 | **Check** | `setImmediate` এর callback চালায় |
| 6 | **Close callbacks** | `socket.on('close')` টাইপের cleanup |

প্রতিটা phase এর মাঝখানে আগে **`process.nextTick` queue**, তারপর **Promise microtask queue** খালি করা হয়।

```js
// ❌ খারাপ: এই loop চলার সময় সব request আটকে থাকবে
app.get('/heavy', (req, res) => {
  let sum = 0;
  for (let i = 0; i < 1e10; i++) sum += i;
  res.send(String(sum));
});
```
**সবচেয়ে বড় নিয়ম:** Event loop কে block করা যাবে না। ওয়েটার একজনই, সে আটকালে সবার সার্ভিস বন্ধ।

---

## 🔴 2. `process.nextTick()` vs `setImmediate()` vs `setTimeout()`

### 🧠 উপমা
অফিসে তিনজন তোমার কাছে এলো: **nextTick** = বস বললেন "এখনই, যা করছো শেষ করেই এটা"। **setTimeout** = "পরের মিটিং এ দেখো"। **setImmediate** = "এই round এর I/O সেরে তারপর এটা"।

### 📖 ব্যাখ্যা

| | কখন চলে | কোথায় |
|---|---|---|
| `process.nextTick()` | বর্তমান operation শেষ হওয়ার **সাথে সাথে**, event loop এগোনোর আগেই | nextTick queue (সবার আগে) |
| `Promise.then` | nextTick এর পর | microtask queue |
| `setTimeout(fn, 0)` | Timers phase এ, কমপক্ষে ১ms পর | Timers phase |
| `setImmediate(fn)` | Poll phase শেষে | Check phase |

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
console.log('sync');
// sync → nextTick → promise → timeout → immediate
```
- Main file এ `timeout` আর `immediate` এর ক্রম **নিশ্চিত না**।
- **I/O callback এর ভেতরে** `setImmediate` সবসময় আগে চলে।
- `nextTick` বারবার recursive ভাবে চালালে event loop **starve** হয় (I/O আটকে যায়)।

---

## 🔴 3. Node এ Async Operation ভেতরে কীভাবে চলে

### 🧠 উপমা
ম্যানেজার (main thread) নিজে সব করে না। সহজ কাজ অফিসের ফোন সিস্টেম (OS) সামলায়, কঠিন কাজ ৪ জনের সহকারী টিম (thread pool) করে। শেষ হলে চিরকুট রাখে (callback queue)।

### 📖 ধাপে ধাপে
```js
fs.readFile('data.txt', (err, data) => console.log('ফাইল পড়া শেষ'));
console.log('এটা আগে print হবে');
```
1. **V8** কোড call stack এ চালায়।
2. `fs.readFile` এ এসে Node কাজটা **libuv** কে দেয় এবং অপেক্ষা না করে পরের লাইনে যায়।
3. libuv দুইভাবে সামলায়:
   - **Network I/O**: OS এর native async (`epoll` Linux, `kqueue` macOS, `IOCP` Windows), আলাদা thread লাগে না।
   - **File system, DNS, crypto, zlib**: libuv এর **thread pool** (default **৪টা** thread)।
4. কাজ শেষে callback **event queue** তে যায়।
5. Call stack খালি হলে event loop সেটা চালায়।

```
JS কোড → V8 (call stack) → libuv → { OS (network) | Thread Pool (file, crypto, dns) }
                                          ↓
                                    Callback Queue → Event Loop → call stack
```

---

## 🟡 4. libuv এর ভূমিকা

### 🧠 উপমা
গাড়ির **চালক = V8** (JS বোঝে), **ইঞ্জিন = libuv** (আসলে গাড়ি চালায়)।

### 📖 ব্যাখ্যা
V8 শুধু JS চালায়। File পড়া, network, timer এর ক্ষমতা দেয় **libuv**, C এ লেখা cross-platform library।

**libuv যা করে:**
- **Event loop** implement করে
- **Thread pool** চালায় (`UV_THREADPOOL_SIZE` দিয়ে বাড়ানো যায়, সর্বোচ্চ ১০২৪)
- Async file system, TCP/UDP, DNS, timers, child process, signal
- Windows/Linux/macOS এর আলাদা OS API কে একটা interface এ আনে

```bash
UV_THREADPOOL_SIZE=8 node app.js   # crypto/file কাজ বেশি হলে
```
**মনে রাখো:** V8 = JS চালায়। libuv = async কাজ ও event loop চালায়।

---

# 📦 PART 2: Streams ও ফাইল হ্যান্ডলিং

> প্রশ্ন 5–7 | ⏱️ আনুমানিক সময়: ~৩৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 5, 6
> 🟡 **Good (মূল ধারণা):** 7

---

## 🔴 5. Streams ও ধরন

### 🧠 উপমা
পুকুর থেকে পুরো পানি বিশাল ট্যাংকে তুলে ব্যবহার (RAM ভরে যায) বনাম **পাইপ দিয়ে একটু একটু করে** আসা। YouTube ও তাই: পুরো ভিডিও download এর অপেক্ষা না করে একটু একটু আসে আর চলে।

### 📖 ব্যাখ্যা
২ GB ফাইলে `fs.readFile` দিলে পুরোটা RAM এ ওঠে। ১০ জন করলে ২০ GB, server crash। Stream ডেটাকে **ছোট chunk** এ ভেঙে একটা একটা করে প্রসেস করে।

**ফায়দা:** কম memory, দ্রুত শুরু, pipeline বানানো যায়।

| ধরন | কাজ | উদাহরণ |
|---|---|---|
| **Readable** | ডেটা পড়া | `fs.createReadStream`, HTTP request |
| **Writable** | ডেটা লেখা | `fs.createWriteStream`, HTTP response |
| **Duplex** | পড়া ও লেখা দুটোই | TCP socket |
| **Transform** | Duplex, ডেটা বদলে দেয় | `zlib.createGzip()` |

```js
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

await pipeline(
  fs.createReadStream('big.txt'),      // Readable
  zlib.createGzip(),                   // Transform
  fs.createWriteStream('big.txt.gz')   // Writable
);
```
ফাইল টুকরো টুকরো আসছে → compress হচ্ছে → লেখা হচ্ছে। ২ GB হলেও RAM এ মাত্র কয়েক KB।

---

## 🔴 6. বড় ফাইল আপলোড Efficiently Handle করা

### 🧠 উপমা
১০০০ বইয়ের কার্টন তোমার ছোট ঘরে ঢুকিয়ে গুদামে পাঠানো বোকামি। **ট্রাক সরাসরি গুদামে** মাল নামাবে (presigned URL)।

### 📖 মূল নীতি: ফাইল কখনো পুরোটা memory তে বাফার করা যাবে না।

1. **Streaming upload**: `busboy` বা `multer` (disk storage)
2. **Presigned URL (সবচেয়ে scalable ✅)**: client সরাসরি S3/R2 তে আপলোড করে, server এ চাপ শূন্য
3. **Chunked / resumable upload**: ছোট ছোট অংশে, fail করলে শুধু সেই অংশ আবার (`tus` protocol)
4. **Limits**: max size, MIME type, timeout
5. **Validation**: আসল content type, দরকারে virus scan
6. **ভারী প্রসেসিং background এ**: BullMQ queue

```js
// Presigned URL (AWS SDK v3)
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
const s3 = new S3Client({ region: 'ap-southeast-1' });

app.post('/upload-url', async (req, res) => {
  const cmd = new PutObjectCommand({ Bucket: 'my-bucket', Key: `uploads/${Date.now()}-${req.body.name}` });
  const url = await getSignedUrl(s3, cmd, { expiresIn: 300 }); // ৫ মিনিট
  res.json({ url });   // client এই URL এ সরাসরি PUT করবে
});
```
```js
// Busboy দিয়ে streaming
const busboy = require('busboy');
const { pipeline } = require('stream/promises');

app.post('/upload', (req, res) => {
  const bb = busboy({ headers: req.headers, limits: { fileSize: 500 * 1024 * 1024 } });
  bb.on('file', (name, file, info) => {
    pipeline(file, fs.createWriteStream(`/uploads/${info.filename}`)).catch(console.error);
  });
  bb.on('close', () => res.send('আপলোড শেষ'));
  req.pipe(bb);
});
```

---

## 🟡 7. Backpressure

### 🧠 উপমা
বড় পাইপ থেকে ছোট বালতিতে পানি ঢালা। বালতি ভরে গেলে **কল সাময়িক বন্ধ**, খালি হলে আবার চালু। এই "থামো, সামলাতে পারছি না" সংকেতই backpressure।

### 📖 ব্যাখ্যা
Readable এর গতি Writable এর গতির চেয়ে বেশি হলে internal buffer ভরে memory বাড়ে, শেষে crash।

- Writable এর buffer সীমা `highWaterMark` (default ১৬ KB)
- `writable.write(chunk)` **`false`** দিলে buffer full, **থামো**
- Buffer খালি হলে **`'drain'`** event আসে, আবার চালাও

**সমাধান ১: `pipe()` / `pipeline()`** (সবচেয়ে সহজ, নিজেই handle করে)
```js
await pipeline(readable, transform, writable);
```
**সমাধান ২: Manual**
```js
readable.on('data', (chunk) => {
  if (!writable.write(chunk)) {
    readable.pause();
    writable.once('drain', () => readable.resume());
  }
});
```
`pipeline()` ভালো কারণ error হলে সব stream destroy করে, `pipe()` তা করে না (leak ঝুঁকি)।

---

# 📦 PART 3: Scaling ও Concurrency

> প্রশ্ন 8–10 | ⏱️ আনুমানিক সময়: ~৩০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 10
> 🟡 **Good (মূল ধারণা):** 8, 9

---

## 🟡 8. Clustering

### 🧠 উপমা
দোকানে ৮টা কাউন্টার ফাঁকা কিন্তু কর্মী একজন। ৮ জন নিয়োগ দাও, **ম্যানেজার** দরজায় কাস্টমার ভাগ করুক। ম্যানেজার = primary process, কর্মী = workers।

### 📖 ব্যাখ্যা
Node default এ **একটা CPU core** ব্যবহার করে। `cluster` module একই app এর **একাধিক process** চালায়, সবাই একই port share করে। Primary connection ভাগ করে (round-robin), worker মরলে নতুন চালু করা যায়।

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) cluster.fork();
  cluster.on('exit', () => cluster.fork());   // auto-restart
} else {
  require('./server');
}
```
**কখন:** অনেক concurrent HTTP request, সব core ব্যবহার, fault tolerance।
**সতর্কতা:** worker দের memory আলাদা, in-memory session/cache share হয় না → **Redis**। Production এ সাধারণত **PM2** (`pm2 start app.js -i max`) বা Docker/Kubernetes।

---

## 🟡 9. Worker Threads vs Cluster

### 🧠 উপমা
**Cluster** = একই ব্র্যান্ডের আলাদা আলাদা দোকান (নিজস্ব ভবন)। **Worker thread** = একই দোকানের ভেতরে অতিরিক্ত কর্মী।

| | **Cluster** | **Worker Threads** |
|---|---|---|
| কী | আলাদা **process** | একই process এ **thread** |
| Memory | সম্পূর্ণ আলাদা | আলাদা V8, `SharedArrayBuffer` দিয়ে share সম্ভব |
| যোগাযোগ | IPC | `postMessage` / shared memory |
| খরচ | বেশি | কম |
| উদ্দেশ্য | **অনেক request** সামলানো | **ভারী CPU কাজ** |

**সহজ নিয়ম:** অনেক user → Cluster | একটা ভারী হিসাব main thread আটকাচ্ছে → Worker Threads।

```js
// main.js
const { Worker } = require('worker_threads');
app.get('/heavy', (req, res) => {
  const w = new Worker('./heavy.js', { workerData: { n: 1e9 } });
  w.on('message', (r) => res.json({ r }));
  w.on('error', (e) => res.status(500).send(e.message));
});
// heavy.js
const { parentPort, workerData } = require('worker_threads');
let s = 0; for (let i = 0; i < workerData.n; i++) s += i;
parentPort.postMessage(s);
```
দুটো একসাথেও চলে: cluster দিয়ে সব core, প্রতিটার ভেতরে ভারী কাজে worker thread।

---

## 🔴 10. Non-blocking I/O

### 🧠 উপমা
ডাক্তার রক্তের নমুনা ল্যাবে পাঠিয়ে **পরের রোগী দেখেন**, রিপোর্ট এলে ল্যাব ডাকে। Blocking হলে রিপোর্ট না আসা পর্যন্ত কেউ ঢুকতে পারে না।

### 📖 ব্যাখ্যা
Non-blocking = I/O শুরু করে **অপেক্ষা না করে** পরের কোডে যাওয়া। তিনটা জিনিসের সমন্বয়: **Event loop** + **libuv** (OS/thread pool) + **callback/Promise/async-await**।

```js
// ❌ Blocking
const data = fs.readFileSync('big.txt');

// ✅ Non-blocking
fs.readFile('big.txt', (err, data) => {});
console.log('এটা আগে print হবে');

// ✅ আধুনিক
const data = await fs.promises.readFile('big.txt');
```
১০০ user এর DB query, প্রতিটা ৫০০ms: blocking এ ৫০ সেকেন্ড, non-blocking এ প্রায় ৫০০ms।
**মনে রাখো:** Non-blocking শুধু **I/O** তে। ভারী **CPU কাজ** তবু main thread আটকায়, তখন worker threads।

---

# 📦 PART 4: Modules, Memory, Performance ও Tools

> প্রশ্ন 11–15 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** 11, 12, 13
> 🟡 **Good (মূল ধারণা):** 14, 15

---

## 🔴 11. CommonJS vs ES Modules

```js
// CommonJS
const express = require('express');
module.exports = { hello };
// ESM
import express from 'express';
export const hello = () => {};
```

| বিষয় | CommonJS | ES Modules |
|---|---|---|
| Syntax | `require` / `module.exports` | `import` / `export` |
| Loading | **Synchronous** | **Asynchronous** |
| বিশ্লেষণ | Runtime এ | **Static** (parse এ), tree-shaking সম্ভব |
| Top-level `await` | ❌ | ✅ |
| `__dirname` | ✅ | ❌ (`import.meta.url`) |
| Binding | Value এর **copy** | **Live binding** |
| চালু করতে | default | `.mjs` বা `"type": "module"` |

```js
// ESM এ __dirname
import { fileURLToPath } from 'url';
import path from 'path';
const __dirname = path.dirname(fileURLToPath(import.meta.url));
```
**Interop:** ESM থেকে CJS import সহজ ✅, CJS থেকে ESM `require` ❌ (dynamic `import()` লাগে)। নতুন প্রজেক্টে ESM standard (TypeScript, Next.js)।

---

## 🔴 12. Memory Leak ও Debug

### 🧠 উপমা
রোজ নতুন জিনিস কিনছো কিন্তু পুরনো ফেলছো না, ঘর ভরে যাচ্ছে।

### 📖 ব্যাখ্যা
Garbage collector অব্যবহৃত object মুছে দেয়। কিন্তু **reference ধরে রাখা** থাকলে মোছা যায় না → memory বাড়ে → `heap out of memory` crash।

**সাধারণ কারণ:**
```js
const cache = {};                                  // ১. সীমাহীন cache
app.get('/u/:id', async (req, res) => { cache[req.params.id] = await getUser(req.params.id); });

app.get('/', () => { process.on('SIGINT', handler); });   // ২. listener জমছে

const t = setInterval(work, 1000);                 // ৩. clearInterval ভুলে যাওয়া
```
৪. বড় ডেটা ধরে রাখা closure, ৫. বন্ধ না করা DB connection/stream।

**Debug (ধাপে ধাপে):**
1. `process.memoryUsage().heapUsed` সময়ের সাথে শুধু বাড়ছে? → leak
2. `node --inspect app.js` → Chrome `chrome://inspect` → Memory → **Snapshot 1** → load দাও → **Snapshot 2** → **compare**
3. Tools: Clinic.js (`clinic heapprofiler`), `heapdump`, `memlab`
4. `MaxListenersExceededWarning` = listener leak সন্দেহ
5. Production এ APM (Datadog, New Relic, Sentry)

**প্রতিরোধ:** LRU/TTL cache (`lru-cache`), সবসময় cleanup, `WeakMap`/`WeakRef`, load test।

---

## 🔴 13. Slow Node App Optimize করা

**ধাপ ০: আগে মাপো 🔍** (আন্দাজে না): `node --prof`, Clinic.js, 0x flamegraph, APM, load test (`autocannon`, `k6`)।

1. **Event loop block** সরাও: sync কাজ, বড় `JSON.parse`, ভারী loop → worker threads
2. **Database (প্রায়ই ৮০% সমস্যা)**: index, **N+1** ঠিক করা, connection pooling, দরকারি column select, pagination
3. **Caching**: Redis
4. **Parallel**: `Promise.all`
5. **Stream** করে বড় ডেটা
6. **Network**: gzip/brotli, keep-alive, CDN
7. **Scaling**: cluster/PM2, load balancer
8. **Background job**: BullMQ
9. ছোট জিনিস: `pino` logger, অপ্রয়োজনীয় middleware বাদ, `NODE_ENV=production`

```js
// ❌ N+1: ১০১টা query
const posts = await prisma.post.findMany();
for (const p of posts) p.author = await prisma.user.findUnique({ where: { id: p.authorId } });
// ✅ একটাই query
const posts = await prisma.post.findMany({ include: { author: true } });

// ❌ ক্রমান্বয়ে ৩s   ✅ একসাথে ১s
const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
```

---

## 🟡 14. Event Emitters

### 🧠 উপমা
YouTube চ্যানেল (emitter) ভিডিও দিলে subscriber (listener) সবাই notification পায়। চ্যানেল জানে না কে কী করছে। এটাই **publish-subscribe**।

| Method | কাজ |
|---|---|
| `on(event, fn)` | বারবার শোনো |
| `once(event, fn)` | একবার শোনো |
| `emit(event, data)` | ঘোষণা করো |
| `off(event, fn)` | listener সরাও |

```js
const EventEmitter = require('events');
class OrderService extends EventEmitter {}
const orders = new OrderService();

orders.on('orderPlaced', (o) => console.log(`📧 Email: #${o.id}`));
orders.on('orderPlaced', (o) => console.log(`📦 Stock: #${o.id}`));
orders.emit('orderPlaced', { id: 101 });
```
**সুবিধা: decoupling।** নতুন কাজ যোগ করতে শুধু আরেকটা listener, মূল কোড ছোঁয়া লাগে না।
**বাস্তব ব্যবহার:** order হলে email/inventory, logging, real-time। Node এর streams, HTTP server, `process` সবই EventEmitter।

**ভুল এড়াও ⚠️:**
1. `error` event এ listener না থাকলে emit করলে **process crash** → সবসময় `.on('error', ...)`
2. Listener গুলো **synchronously** চলে
3. ১০টার বেশি listener এ warning (সম্ভাব্য leak)

---

## 🟡 15. `npm install` ভেতরে কী ঘটে

### 🧠 উপমা
বাজারের **তালিকা** (`package.json`) আর **আগের বারের রসিদ** (`package-lock.json`) নিয়ে যাও। আগে ফ্রিজ (cache) দেখো, না থাকলে বাজার (registry) থেকে আনো, জিনিস ঠিক কিনা মিলাও (integrity), তারপর সাজাও (`node_modules`)।

**ধাপে ধাপে:**
1. **Config পড়া**: `package.json`, `.npmrc`, lock file
2. **Dependency tree (Arborist)**: lock থাকলে সুনির্দিষ্ট version, না থাকলে range (`^4.17.0`) থেকে; transitive dependency সহ পুরো গাছ
3. **Registry থেকে metadata** (`registry.npmjs.org`)
4. **Cache চেক** (`~/.npm/_cacache`), না থাকলে tarball download
5. **Integrity যাচাই** (SHA-512 hash)
6. **`node_modules` এ বসানো**, duplicate কমাতে **hoist**
7. **Lifecycle scripts**: `preinstall` → `install` → `postinstall` (native module compile `node-gyp`)। ⚠️ অচেনা package ঝুঁকিপূর্ণ, scripts এ যেকোনো কোড চলতে পারে
8. **`package-lock.json` আপডেট**
9. Summary (`npm audit` সহ)

| | `npm install` | `npm ci` |
|---|---|---|
| Lock file | দরকারে আপডেট | **কখনো বদলায় না** |
| `node_modules` | থাকলে রেখে দেয় | আগে **মুছে** নতুন |
| গতি | ধীর | দ্রুত |
| কোথায় | local dev | **CI/CD, production build** |

---

## 🎯 ৫ মিনিটের Revision

| বিষয় | এক লাইনে |
|---|---|
| Event loop | একজন ওয়েটার, ৬ phase, block করা যাবে না |
| nextTick/setImmediate/setTimeout | nextTick → Promise → Timers → Check |
| Async internals | V8 → libuv → OS/thread pool → queue → event loop |
| libuv | V8 JS চালায়, libuv async ও event loop |
| Streams | Readable, Writable, Duplex, Transform; chunk এ কাজ |
| বড় upload | RAM এ বাফার নয়; streaming, presigned URL, chunked |
| Backpressure | `write()` false → থামো, `drain` এ চালু; `pipeline()` |
| Cluster | একাধিক process, সব core |
| Cluster vs Worker | process (request) vs thread (CPU কাজ) |
| Non-blocking | event loop + libuv + callbacks |
| CJS vs ESM | require vs import; ESM এ top-level await |
| Memory leak | reference ধরে রাখা; heap snapshot compare |
| Optimize | আগে profile; DB, cache, parallel, queue |
| EventEmitter | pub-sub; `error` listener ভুলো না |
| npm install | config → tree → registry/cache → integrity → extract → scripts → lock |

শুভকামনা! 🚀
