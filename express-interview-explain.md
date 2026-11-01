# 📘 Express.js, Auth ও Security, Scenario: Interview গাইড (বিস্তারিত ব্যাখ্যা)

> এই ফাইলে **বুঝে পড়ার** জন্য ব্যাখ্যা: উপমা → ব্যাখ্যা → code।
> মুখে বলার ছোট উত্তর আলাদা ফাইলে: `express-interview-answers.md`

> **রঙের মানে:** 🔴 অবশ্যই পড়ো (প্রায় নিশ্চিত আসবে) | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে
> প্রতিটা প্রশ্নের heading এর পাশেই রঙ বসানো আছে।
> **নম্বর:** `E` = Express, `A` = Auth ও Security, `S` = Scenario

---

## 📑 সূচিপত্র

| # | বিষয় | গুরুত্ব | Part |
|---|---|---|---|
| E1 | Request-response lifecycle | 🔴 | 1 |
| E2 | Middleware কী | 🔴 | 1 |
| E3 | Application / Router / Error middleware | 🟡 | 1 |
| E4 | `next()` ভেতরে কীভাবে কাজ করে | 🔴 | 1 |
| E5 | Scalable project structure | 🔴 | 2 |
| E6 | REST API best practices | 🔴 | 2 |
| E7 | Centralized error handling | 🔴 | 2 |
| E8 | Request data validation | 🔴 | 2 |
| E9 | Route modularization | 🟡 | 2 |
| E10 | Authentication vs Authorization | 🔴 | 3 |
| E11 | RBAC | 🔴 | 3 |
| E12 | Express API secure করা | 🔴 | 3 |
| E13 | CORS ও সাধারণ সমস্যা | 🔴 | 3 |
| E14 | Cookies ও Sessions | 🟡 | 3 |
| E15 | Stateless vs Stateful auth | 🟡 | 3 |
| A1 | JWT কীভাবে কাজ করে | 🔴 | 4 |
| A2 | JWT এর অসুবিধা | 🔴 | 4 |
| A3 | Refresh token flow | 🔴 | 4 |
| A4 | Access token vs Refresh token | 🟡 | 4 |
| A5 | Token নিরাপদে রাখা | 🔴 | 4 |
| A6 | bcrypt দিয়ে password hashing | 🔴 | 5 |
| A7 | Salting | 🟡 | 5 |
| A8 | Brute force আটকানো | 🟡 | 5 |
| A9 | CSRF, XSS, Injection | 🔴 | 5 |
| A10 | Environment variable secure করা | 🟡 | 5 |
| A11 | Helmet.js | 🟡 | 5 |
| A12 | Rate limiting ও throttling | 🔴 | 5 |
| A13 | API key authentication | 🟡 | 5 |
| S1 | Heavy traffic এ API slow | 🔴 | 6 |
| S2 | Deploy এর পর random logout | 🟡 | 6 |
| S3 | বড় ফাইল আপলোডে crash | 🟡 | 6 |
| S4 | Memory ক্রমাগত বাড়ছে | 🟡 | 6 |
| S5 | লক্ষ user এর scalable auth | 🔴 | 6 |
| S6 | DB query ৫-১০ সেকেন্ড | 🔴 | 7 |
| S7 | Public API rate limiting | 🟡 | 7 |
| S8 | Local এ চলে, production এ fail | 🟡 | 7 |
| S9 | Duplicate transaction আটকানো | 🔴 | 7 |
| S10 | ধীর ও অনির্ভরযোগ্য third-party API | 🔴 | 7 |

---

# 📦 PART 1: Express Core

> প্রশ্ন E1–E4 | ⏱️ আনুমানিক সময়: ~৩৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** E1, E2, E4
> 🟡 **Good (মূল ধারণা):** E3

---

## 🔴 E1. Request-Response Lifecycle

### 🧠 উপমা
এয়ারপোর্টে যাত্রী: **গেট** (server) → **ব্যাগ স্ক্যান** (middleware) → **চেক-ইন** (আরেক middleware) → **নির্দিষ্ট বিমান** (route handler) → বিমান ছাড়ে (response)। কোনো ধাপে সমস্যা হলে **অফিসে** পাঠানো হয় (error handler)।

### 📖 ধাপে ধাপে
1. Client request পাঠায় (`GET /api/users`)
2. Node এর `http` server request ধরে, Express app কে দেয়
3. Express request কে **middleware stack** এর উপর দিয়ে **ক্রমানুসারে** পাঠায় (`express.json()`, logger, auth...)
4. প্রতিটা middleware হয় response দিয়ে শেষ করে, নয়তো `next()` call করে পরেরটায় পাঠায়
5. **Route match** হলে সেই route এর handler চলে
6. Handler `res.json()` / `res.send()` দিয়ে response পাঠায় → lifecycle শেষ
7. কোথাও error হলে (`next(err)` বা throw) সোজা **error-handling middleware** এ যায়
8. কোনো route match না হলে শেষের **404 handler** চলে

```js
const app = express();
app.use(express.json());                  // ১. body parse
app.use((req, res, next) => {             // ২. logger
  console.log(req.method, req.url); next();
});
app.get('/users', authMiddleware, (req, res) => {   // ৩. route (auth → handler)
  res.json({ users: [] });
});
app.use((req, res) => res.status(404).json({ message: 'Not found' }));   // ৪. 404
app.use((err, req, res, next) => {                                       // ৫. error handler
  res.status(err.status || 500).json({ message: err.message });
});
```
**মূল কথা:** ক্রম (order) খুব গুরুত্বপূর্ণ। যেটা আগে লেখা, সেটা আগে চলে।

---

## 🔴 E2. Middleware কী

### 🧠 উপমা
Request ও response এর মাঝের **চেকপয়েন্ট**। প্রতিটা চেকপয়েন্টে পরীক্ষা, দরকারে বদলানো, তারপর পাস বা আটকে দেওয়া।

### 📖 ব্যাখ্যা
Middleware হলো এমন function যেটা `(req, res, next)` পায়। এটা তিনটার একটা করতে পারে:
1. `req`/`res` **পরিবর্তন** করতে পারে (যেমন `req.user = ...`)
2. Response দিয়ে চক্র **শেষ** করতে পারে (`res.status(401).json(...)`)
3. **`next()`** দিয়ে পরের middleware এ পাঠাতে পারে

> ⚠️ `next()` ও response কোনোটাই না করলে request **ঝুলে থাকে** (hang)।

```js
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();                                  // না দিলে request আটকে থাকবে
};
const auth = (req, res, next) => {
  if (!req.headers.authorization) return res.status(401).json({ message: 'Unauthorized' });
  req.user = verify(req.headers.authorization);
  next();
};
app.use(logger);
app.get('/profile', auth, (req, res) => res.json(req.user));
```
**সাধারণ ব্যবহার:** logging, auth, body parsing, CORS, validation, rate limiting, error handling।

---

## 🟡 E3. Application-level, Router-level, Error-handling Middleware

| ধরন | কোথায় বসে | কাজ |
|---|---|---|
| **Application-level** | `app.use()`, `app.get()` | পুরো app এ প্রযোজ্য (logger, `express.json()`, cors) |
| **Router-level** | `router.use()` | শুধু সেই router এর route গুলোয় (যেমন `/admin` এর সবটায় auth) |
| **Error-handling** | `(err, req, res, next)` | **৪টা** parameter, error ধরে |
| Built-in / Third-party | | `express.json()`, `helmet`, `cors` |

```js
// Application-level
app.use(express.json());

// Router-level
const adminRouter = express.Router();
adminRouter.use(requireAdmin);              // শুধু /admin এর জন্য
adminRouter.get('/stats', getStats);
app.use('/admin', adminRouter);

// Error-handling: ৪টা parameter বাধ্যতামূলক, নইলে Express চিনবে না
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ message: err.message });
});
```
**নিয়ম:** error middleware সবার **শেষে** রাখতে হয়।

---

## 🔴 E4. `next()` ভেতরে কীভাবে কাজ করে

### 🧠 উপমা
Relay race এর **ব্যাটন**। এক দৌড়বিদ পরের জনকে ব্যাটন দেয় (`next()`)। কেউ ব্যাটন না দিলে দৌড় সেখানেই থেমে যায়।

### 📖 ব্যাখ্যা
Express ভেতরে প্রতিটা middleware/route কে একটা **Layer** হিসেবে একটা **array (stack)** এ রাখে। `next` হলো একটা closure যেটা একটা **index** মনে রাখে। `next()` call হলে index বাড়িয়ে পরের **matching** layer (path ও method মেলে এমন) খোঁজে ও চালায়।

**সহজ সংস্করণ:**
```js
function createApp() {
  const stack = [];
  const app = { use: (fn) => stack.push(fn) };

  app.handle = (req, res) => {
    let i = 0;
    function next(err) {
      const layer = stack[i++];
      if (!layer) return res.end(err ? 'Error' : 'Not found');

      const isErrorHandler = layer.length === 4;       // ৪ parameter = error handler
      try {
        if (err && isErrorHandler) layer(err, req, res, next);
        else if (!err && !isErrorHandler) layer(req, res, next);
        else next(err);                                 // মেলেনি, পরেরটায় যাও
      } catch (e) { next(e); }
    }
    next();
  };
  return app;
}
```
**`next()` এর বিভিন্ন রূপ:**
| Call | কী হয় |
|---|---|
| `next()` | পরের middleware |
| `next(err)` | **error mode**: সাধারণ middleware গুলো **skip**, সোজা error handler এ |
| `next('route')` | এই route এর বাকি handler skip, পরের route এ |
| `next('router')` | পুরো router থেকে বেরিয়ে যাও |

`throw` করলেও Express (sync কোডে) ধরে `next(err)` এর মতো কাজ করে। **Express 5** এ async function এর rejected Promise ও নিজে থেকেই error handler এ যায়। Express 4 এ `try/catch` বা wrapper লাগে।

---

# 📦 PART 2: Structure, API Design, Error ও Validation

> প্রশ্ন E5–E9 | ⏱️ আনুমানিক সময়: ~৫০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** E5, E6, E7, E8
> 🟡 **Good (মূল ধারণা):** E9

---

## 🔴 E5. Scalable Express Project Structure

### 🧠 উপমা
রেস্টুরেন্টে **ওয়েটার** (route) অর্ডার নেয়, **শেফ** (service) রান্না করে, **স্টোররুম** (repository) থেকে মাল আনে। সবাই একই কাজ করলে বিশৃঙ্খলা।

### 📖 Layered architecture
```
src/
├── app.js                 # Express app (middleware, routes), test এর জন্য আলাদা
├── server.js              # শুধু listen, graceful shutdown
├── config/                # env, db, redis config
├── modules/               # feature-based (সবচেয়ে scalable)
│   ├── users/
│   │   ├── user.routes.js       # শুধু URL → controller
│   │   ├── user.controller.js   # req/res, কোনো business logic নেই
│   │   ├── user.service.js      # business logic
│   │   ├── user.repository.js   # DB query (Prisma)
│   │   └── user.validation.js   # Zod schema
│   └── orders/ ...
├── middlewares/           # auth, error, validate, rateLimit
├── utils/                 # AppError, asyncHandler, logger
└── jobs/                  # background job (BullMQ)
```
**মূল নীতি:**
- **Route → Controller → Service → Repository** (প্রতিটার একটাই দায়িত্ব)
- Controller পাতলা, **Service এ business logic**
- `app.js` আর `server.js` আলাদা (test এ `supertest` দিয়ে app নেওয়া যায়)
- Config environment থেকে, **কোনো hardcode secret না**
- Feature-based folder (`users/`, `orders/`), type-based (`controllers/`, `models/`) এর চেয়ে বড় প্রজেক্টে ভালো
- Logger (`pino`), centralized error handling, graceful shutdown

```js
// user.controller.js: পাতলা
const getUser = async (req, res) => {
  const user = await userService.getById(req.params.id);
  res.json({ data: user });
};
// user.service.js: business logic
const getById = async (id) => {
  const user = await userRepo.findById(id);
  if (!user) throw new AppError('User not found', 404);
  return user;
};
```

---

## 🔴 E6. REST API Best Practices

1. **Noun (বিশেষ্য), plural**: `/users`, `/users/42/orders` (❌ `/getUsers`)
2. **HTTP method সঠিক:** `GET` পড়া, `POST` তৈরি, `PUT` পুরো বদল, `PATCH` আংশিক, `DELETE` মোছা
3. **সঠিক status code:**

| Code | মানে |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content (delete) |
| 400 | Bad request / validation |
| 401 | Unauthenticated (লগইন নেই) |
| 403 | Forbidden (অনুমতি নেই) |
| 404 | Not found |
| 409 | Conflict (duplicate) |
| 422 | Unprocessable |
| 429 | Too many requests |
| 500 | Server error |

4. **Versioning:** `/api/v1/users`
5. **Pagination, filtering, sorting:** `/users?page=2&limit=20&sort=-createdAt&role=admin` (বড় ডেটায় cursor pagination)
6. **Consistent response format:**
```json
{ "success": true, "data": {...}, "meta": { "page": 1, "total": 50 } }
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [...] } }
```
7. **Idempotency:** `GET`, `PUT`, `DELETE` idempotent; payment এ `POST` এর জন্য idempotency key
8. **Validation, auth, rate limiting, HTTPS** সবসময়
9. **Stateless**: প্রতিটা request এ দরকারি সব তথ্য থাকবে
10. **Documentation:** OpenAPI/Swagger
11. ভেতরের তথ্য (stack trace, DB error) leak করো না

---

## 🔴 E7. Centralized Error Handling

### 🧠 ধারণা
প্রতিটা route এ আলাদা `try/catch` + `res.status(500)` লেখার বদলে **একটাই জায়গা** যেখানে সব error আসে ও একই ফরম্যাটে response যায়।

**ধাপ ১: নিজের Error class**
```js
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;     // প্রত্যাশিত error (ভুল input, not found)
    Error.captureStackTrace(this, this.constructor);
  }
}
```
**ধাপ ২: async wrapper** (Express 4 এর জন্য; Express 5 এ লাগে না)
```js
const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await userService.getById(req.params.id);   // throw করলেই handler এ যাবে
  res.json({ data: user });
}));
```
**ধাপ ৩: 404 ও central handler (সবার শেষে)**
```js
app.use((req, res, next) => next(new AppError(`Route ${req.originalUrl} not found`, 404)));

app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  if (status >= 500) logger.error(err);                      // শুধু অপ্রত্যাশিত error log
  res.status(status).json({
    success: false,
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.isOperational ? err.message : 'Something went wrong',   // ভেতরের তথ্য লুকাও
      ...(process.env.NODE_ENV !== 'production' && { stack: err.stack }),
    },
  });
});
```
**বাড়তি:** Prisma error (`P2002` = unique violation) → 409, Zod error → 400 এ রূপান্তর। `unhandledRejection` ও `uncaughtException` এ log করে graceful shutdown।

---

## 🔴 E8. Request Data Validation

**কেন:** Client কে বিশ্বাস করা যায় না। ভুল/ক্ষতিকর ডেটা DB তে ঢুকলে bug ও security সমস্যা।

**Libraries:** **Zod** (TypeScript এর সাথে সেরা), Joi, express-validator।

```js
const { z } = require('zod');

const createUserSchema = z.object({
  body: z.object({
    name: z.string().min(2).max(50),
    email: z.string().email(),
    password: z.string().min(8),
    age: z.number().int().min(18).optional(),
  }),
  params: z.object({}).optional(),
  query: z.object({}).optional(),
});

// reusable middleware
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse({ body: req.body, params: req.params, query: req.query });
  if (!result.success) {
    return res.status(400).json({
      success: false,
      error: { code: 'VALIDATION_ERROR', details: result.error.issues },
    });
  }
  req.body = result.data.body;     // পরিষ্কার ও type-safe ডেটা, অচেনা field বাদ
  next();
};

app.post('/users', validate(createUserSchema), createUser);
```
**ভালো অভ্যাস:**
- `body`, `params`, `query` তিনটাই validate
- **Whitelist**: অচেনা field বাদ (mass assignment আটকায়: কেউ `role: 'admin'` পাঠাতে পারবে না)
- Validation **controller এর আগে**, middleware এ
- Type coercion সাবধানে (query সব string আসে, `z.coerce.number()`)
- Frontend validation আলাদা, backend validation **বাধ্যতামূলক**

---

## 🟡 E9. Route Modularization

প্রতিটা feature এর route আলাদা ফাইলে `express.Router()` দিয়ে, তারপর main app এ mount।

```js
// modules/users/user.routes.js
const router = require('express').Router();
router.use(authenticate);                       // এই router এর সবার জন্য
router.get('/', validate(listSchema), ctrl.list);
router.get('/:id', ctrl.getOne);
router.post('/', authorize('admin'), validate(createSchema), ctrl.create);
module.exports = router;

// routes/index.js
const v1 = require('express').Router();
v1.use('/users', require('../modules/users/user.routes'));
v1.use('/orders', require('../modules/orders/order.routes'));

// app.js
app.use('/api/v1', v1);
```
**সুবিধা:** ছোট ফাইল, সহজ maintenance, router-level middleware, versioning সহজ, টিমে ভাগ করে কাজ।

---

# 📦 PART 3: Auth Basics ও Express Security

> প্রশ্ন E10–E15 | ⏱️ আনুমানিক সময়: ~৫০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** E10, E11, E12, E13
> 🟡 **Good (মূল ধারণা):** E14, E15

---

## 🔴 E10. Authentication vs Authorization

### 🧠 উপমা
অফিস বিল্ডিং এ ঢোকার সময়: **Authentication** = গেটে আইডি কার্ড দেখিয়ে প্রমাণ করা "আমি কে"। **Authorization** = কার্ড দিয়ে কোন কোন রুমে ঢুকতে পারি।

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| প্রশ্ন | তুমি **কে**? | তুমি **কী করতে পারো**? |
| কখন | আগে | পরে |
| উদাহরণ | password, OTP, fingerprint, JWT যাচাই | admin হলে delete করতে পারে |
| ব্যর্থ হলে | **401 Unauthorized** | **403 Forbidden** |

```js
const authenticate = (req, res, next) => {          // তুমি কে?
  const user = verifyToken(req.headers.authorization);
  if (!user) return res.status(401).json({ message: 'Login required' });
  req.user = user; next();
};
const authorize = (...roles) => (req, res, next) => {   // তুমি পারো কি?
  if (!roles.includes(req.user.role)) return res.status(403).json({ message: 'Forbidden' });
  next();
};
app.delete('/users/:id', authenticate, authorize('admin'), deleteUser);
```

---

## 🔴 E11. Role-Based Access Control (RBAC)

User এর **role** (admin, editor, user) অনুযায়ী অ্যাক্সেস। Role এর সাথে **permission** জুড়ে দেওয়া থাকে।

**সহজ রূপ:**
```js
const authorize = (...allowed) => (req, res, next) => {
  if (!allowed.includes(req.user.role)) throw new AppError('Forbidden', 403);
  next();
};
router.post('/posts', authenticate, authorize('admin', 'editor'), createPost);
```
**Permission-based (বড় সিস্টেম):**
```js
const permissions = {
  admin:  ['user:read', 'user:write', 'user:delete', 'post:write'],
  editor: ['post:read', 'post:write'],
  user:   ['post:read'],
};
const can = (perm) => (req, res, next) =>
  permissions[req.user.role]?.includes(perm) ? next() : next(new AppError('Forbidden', 403));
router.delete('/users/:id', authenticate, can('user:delete'), remove);
```
**DB design:** `users` ↔ `roles` ↔ `permissions` (many-to-many), যাতে code না বদলিয়ে role বদলানো যায়।

**⚠️ Ownership চেক ভুলো না (খুব বড় security bug):** Role দেখলেই হয় না। "user নিজের post ই edit করতে পারবে"।
```js
const post = await prisma.post.findUnique({ where: { id } });
if (post.authorId !== req.user.id && req.user.role !== 'admin') throw new AppError('Forbidden', 403);
```
JWT তে role রাখলে role বদলালে token পুরনো থাকে, তাই sensitive কাজে DB থেকে আবার যাচাই।

---

## 🔴 E12. Express API Secure করা

**Checklist:**
1. **HTTPS** সবসময় (production এ `trust proxy` সেট করো)
2. **Helmet** (security headers)
3. **CORS** শুধু বিশ্বস্ত origin
4. **Rate limiting** (brute force, DoS আটকায়)
5. **Input validation** (Zod/Joi) ও whitelist
6. **Injection থেকে সুরক্ষা**: parameterized query / ORM (Prisma), `express-mongo-sanitize`
7. **Authentication**: শক্ত JWT/session, short expiry
8. **Password**: bcrypt/argon2, কখনো plain text না
9. **Cookie flags:** `httpOnly`, `secure`, `sameSite`
10. **Body size limit:** `express.json({ limit: '10kb' })`
11. **Error এ stack trace leak না করা**
12. **Secrets env তে**, git এ না
13. **`npm audit`**, dependency update
14. **Logging ও monitoring**
15. `app.disable('x-powered-by')`
16. **Least privilege** (DB user, role)

```js
app.set('trust proxy', 1);
app.use(helmet());
app.use(cors({ origin: ['https://myapp.com'], credentials: true }));
app.use(express.json({ limit: '10kb' }));
app.use('/api', rateLimit({ windowMs: 15 * 60 * 1000, limit: 100 }));
```

---

## 🔴 E13. CORS ও সাধারণ সমস্যা

### 🧠 ব্যাখ্যা
**Origin** = protocol + domain + port। ব্রাউজারের **Same-Origin Policy** এক origin এর JS কে অন্য origin এ request এর response পড়তে দেয় না। **CORS** হলো server এর বলা "এই origin গুলোকে আমি অনুমতি দিলাম"। এটা **শুধু ব্রাউজারে** প্রযোজ্য (Postman/curl এ না) এবং server কে নিরাপদ করে না।

**Preflight:** non-simple request (যেমন `Authorization` header, `PUT`/`DELETE`, JSON body) এর আগে ব্রাউজার একটা `OPTIONS` request পাঠিয়ে জিজ্ঞেস করে "অনুমতি আছে?"।

```js
const cors = require('cors');
app.use(cors({
  origin: ['https://myapp.com', 'http://localhost:3000'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,          // cookie পাঠাতে চাইলে
}));
```
**সাধারণ সমস্যা:**
| সমস্যা | সমাধান |
|---|---|
| `credentials: true` এর সাথে `origin: '*'` কাজ করে না | নির্দিষ্ট origin দাও |
| Client এ `credentials: 'include'` / `withCredentials` নেই | দুই দিকেই দাও |
| `cors()` route এর **পরে** বসানো | সবার **আগে** বসাও |
| `Authorization` header `allowedHeaders` এ নেই | যোগ করো |
| Preflight (`OPTIONS`) auth middleware এ আটকে 401 | cors আগে, OPTIONS এ auth নয় |
| Error এ response এও CORS header নেই | error handler এর আগেই cors |
| Proxy/Nginx আলাদা header বসায় (duplicate) | একজায়গায় handle করো |
| Cookie এ `sameSite=None; Secure` লাগে cross-site এ | HTTPS ও সঠিক flag |

**Debug টিপস:** ব্রাউজারের Network tab এ `OPTIONS` request ও response header দেখো।

---

## 🟡 E14. Cookies ও Sessions

### 🧠 উপমা
**Cookie** = ব্রাউজারে রাখা ছোট টোকেন। **Session** = server এর খাতায় তোমার তথ্য, cookie তে শুধু খাতার **নম্বর (session ID)**।

**কীভাবে কাজ করে:**
1. Login সফল → server session বানায়, ডেটা **server এ** রাখে (memory/Redis/DB)
2. Response এ `Set-Cookie: sid=abc123` (শুধু ID)
3. ব্রাউজার পরের প্রতিটা request এ **নিজে থেকেই** cookie পাঠায়
4. Server ID দিয়ে session খুঁজে user চেনে

```js
const session = require('express-session');
const RedisStore = require('connect-redis').default;

app.use(session({
  store: new RedisStore({ client: redisClient }),   // ❌ default MemoryStore production এ না
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 1000 * 60 * 60 * 24 },
}));
```
**Cookie flags:**
- `httpOnly`: JS পড়তে পারে না (XSS থেকে সুরক্ষা)
- `secure`: শুধু HTTPS এ
- `sameSite`: cross-site এ পাঠানো সীমিত (CSRF সুরক্ষা)
- `maxAge`/`expires`

Login এর পর `req.session.regenerate()` (session fixation আটকায়)।

---

## 🟡 E15. Stateless vs Stateful Authentication

| | **Stateful (Session)** | **Stateless (JWT)** |
|---|---|---|
| State কোথায় | **Server এ** (session store) | **Token এ** (client এ) |
| প্রতি request | store এ lookup | শুধু signature যাচাই |
| Scaling | shared store (Redis) লাগে | সহজ, যেকোনো server যাচাই করতে পারে |
| **Logout / revoke** | ✅ সহজ (session মুছে দাও) | ❌ কঠিন (expire পর্যন্ত valid) |
| ডেটা সাইজ | cookie ছোট | token বড় |
| Mobile/API | ততটা সুবিধাজনক না | ✅ ভালো |
| Control | বেশি | কম |

**সারকথা:** সাধারণ web app এ session সহজ ও নিরাপদ। Mobile/microservice/API এ JWT সুবিধাজনক। বাস্তবে অনেকে **hybrid**: short-lived JWT + server-stored refresh token।

---

# 📦 PART 4: JWT ও Token

> প্রশ্ন A1–A5 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** A1, A2, A3, A5
> 🟡 **Good (মূল ধারণা):** A4

---

## 🔴 A1. JWT কীভাবে কাজ করে

### 🧠 উপমা
**সিল করা ছাড়পত্র।** তুমি ছাড়পত্র দেখাও, গেটের লোক শুধু সিল (signature) যাচাই করে ঢুকতে দেয়, প্রতিবার অফিসে ফোন করে জানতে হয় না।

### 📖 গঠন: `header.payload.signature` (তিনটা অংশ, base64url)
```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiI0MiIsInJvbGUiOiJ1c2VyIiwiZXhwIjoxNzAwfQ . SflKxwRJ...
   Header                    Payload (claims)                                  Signature
```
- **Header**: algorithm (`HS256`/`RS256`)
- **Payload**: claims, যেমন `sub` (user id), `role`, `iat`, `exp`
- **Signature**: `HMAC(header + payload, secret)`. কেউ payload বদলালে signature মিলবে না

> ⚠️ Payload **encrypted না, শুধু encoded**। যে কেউ পড়তে পারে। তাই password বা সংবেদনশীল তথ্য রাখবে না।

**Flow:**
1. Login (email + password) → server যাচাই করে → JWT বানায়
2. Client token সংরক্ষণ করে
3. প্রতি request এ `Authorization: Bearer <token>`
4. Server signature ও `exp` যাচাই করে → বৈধ হলে user চেনে (**DB lookup ছাড়াই**)

```js
const jwt = require('jsonwebtoken');

// তৈরি
const token = jwt.sign({ sub: user.id, role: user.role }, process.env.JWT_SECRET, { expiresIn: '15m' });

// যাচাই middleware
const authenticate = (req, res, next) => {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) return res.status(401).json({ message: 'No token' });
  try {
    req.user = jwt.verify(header.split(' ')[1], process.env.JWT_SECRET, { algorithms: ['HS256'] });
    next();
  } catch {
    res.status(401).json({ message: 'Invalid or expired token' });
  }
};
```
`algorithms` স্পষ্ট দেওয়া জরুরি (algorithm confusion attack আটকায়)।

---

## 🔴 A2. JWT এর অসুবিধা

1. **Revoke করা কঠিন**: একবার ইস্যু হলে `exp` পর্যন্ত valid, logout করলেও। (সমাধান: short expiry, blocklist)
2. **Token চুরি হলে** চোর expire পর্যন্ত ব্যবহার করতে পারে
3. **Payload পড়া যায়** (encrypted না)
4. **সাইজ বড়**, প্রতি request এ পাঠাতে হয়
5. **Stale ডেটা**: role বদলালে পুরনো token এ পুরনো role থাকে
6. **Storage সমস্যা**: `localStorage` এ XSS ঝুঁকি, cookie তে CSRF ঝুঁকি
7. **Secret leak হলে** সবাইকে নকল token বানানো যায়
8. **ভুল configuration**: `alg: none`, দুর্বল secret, algorithm confusion
9. Sensitive অ্যাকশনে (logout everywhere, ban) real-time control নেই

**সমাধানের কৌশল:** access token **ছোট আয়ু (৫-১৫ মিনিট)**, **refresh token rotation**, দরকারে Redis এ **blocklist** (`jti`), শক্ত secret/asymmetric key, ও কম claim।

---

## 🔴 A3. Refresh Token Flow

### 🧠 উপমা
**Access token** = দিনের ভিজিটর পাস (ছোট সময়, হারালে ক্ষতি কম)। **Refresh token** = স্থায়ী কর্মচারী কার্ড, যেটা দেখিয়ে রোজ নতুন ভিজিটর পাস নাও।

### 📖 Flow
1. **Login** → server দেয় **Access token** (১৫ মিনিট) + **Refresh token** (৭-৩০ দিন)
2. Refresh token যায় **`httpOnly` cookie** তে; DB তে সংরক্ষিত হয় **hash করে**
3. Client access token দিয়ে API call করে
4. Access token **expire** → API `401` দেয়
5. Client `POST /auth/refresh` call করে (cookie নিজে যায়)
6. Server refresh token **যাচাই করে + DB তে আছে কিনা দেখে**
7. **Rotation:** নতুন access + **নতুন refresh token** দেয়, **পুরনোটা বাতিল**
8. **Reuse detection:** বাতিল refresh token আবার এলে মানে চুরি → ওই user এর সব session বাতিল
9. **Logout** → refresh token DB থেকে মুছে, cookie clear

```js
app.post('/auth/refresh', async (req, res) => {
  const token = req.cookies.refreshToken;
  if (!token) return res.sendStatus(401);

  const payload = jwt.verify(token, process.env.REFRESH_SECRET);
  const stored = await db.refreshToken.findUnique({ where: { tokenHash: sha256(token) } });

  if (!stored || stored.revoked) {
    await db.refreshToken.updateMany({ where: { userId: payload.sub }, data: { revoked: true } }); // reuse → সব বাতিল
    return res.sendStatus(401);
  }
  await db.refreshToken.update({ where: { id: stored.id }, data: { revoked: true } });   // পুরনো বাতিল

  const accessToken = jwt.sign({ sub: payload.sub }, process.env.JWT_SECRET, { expiresIn: '15m' });
  const newRefresh = jwt.sign({ sub: payload.sub }, process.env.REFRESH_SECRET, { expiresIn: '7d' });
  await db.refreshToken.create({ data: { userId: payload.sub, tokenHash: sha256(newRefresh) } });

  res.cookie('refreshToken', newRefresh, { httpOnly: true, secure: true, sameSite: 'strict', maxAge: 7 * 864e5 });
  res.json({ accessToken });
});
```
**Frontend টিপস:** অনেক tab একসাথে refresh করলে race condition হয়, তাই একটা request এ lock/queue রাখো।

---

## 🟡 A4. Access Token vs Refresh Token

| | **Access Token** | **Refresh Token** |
|---|---|---|
| কাজ | API অ্যাক্সেস | নতুন access token আনা |
| আয়ু | **ছোট** (৫-১৫ মিনিট) | **বড়** (৭-৩০ দিন) |
| কোথায় যায় | প্রতিটা API request এ | শুধু `/refresh` এ |
| কোথায় রাখে | memory (JS variable) | `httpOnly` cookie |
| Server এ সংরক্ষণ | সাধারণত না (stateless) | **হ্যাঁ** (hash করে, revoke এর জন্য) |
| চুরি হলে ক্ষতি | কম সময়ের | বেশি, তাই বেশি সুরক্ষা |

কেন দুটো: নিরাপত্তা (ছোট আয়ুর token চুরি হলে ক্ষতি সীমিত) ও সুবিধা (user কে বারবার লগইন করতে হয় না)।

---

## 🔴 A5. Token নিরাপদে রাখা

| জায়গা | ঝুঁকি |
|---|---|
| `localStorage` / `sessionStorage` | ❌ **XSS** এ যেকোনো JS পড়তে পারে |
| `httpOnly` cookie | ✅ JS পড়তে পারে না। কিন্তু **CSRF** ঝুঁকি (`sameSite` দিয়ে কমে) |
| JS memory (variable) | ✅ XSS এ কিছুটা নিরাপদ, কিন্তু refresh করলে হারায় |

**সেরা প্রচলিত কৌশল:**
- **Access token → memory** তে (React state/variable)
- **Refresh token → `httpOnly; Secure; SameSite` cookie**
- Page reload এ refresh endpoint দিয়ে নতুন access token আনো
- **Server এ refresh token hash করে** রাখো (DB leak হলেও কাজে লাগবে না)
- সবসময় **HTTPS**
- XSS কমাতে **CSP**, input sanitize
- **Mobile:** iOS Keychain / Android Keystore (AsyncStorage না)
- Token log এ ছাপাবে না, URL এ রাখবে না

---

# 📦 PART 5: Password, Attack Protection ও Security Tools

> প্রশ্ন A6–A13 | ⏱️ আনুমানিক সময়: ~৫৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** A6, A9, A12
> 🟡 **Good (মূল ধারণা):** A7, A8, A10, A11, A13

---

## 🔴 A6. bcrypt দিয়ে Password Hashing

**Hashing ≠ Encryption।** Encryption দুই-মুখী (চাবি দিয়ে ফেরত আনা যায়), hashing **এক-মুখী** (ফেরত আনা যায় না)। Password কখনো plain text এ বা reversible ভাবে রাখা যাবে না।

**কেন bcrypt (SHA256/MD5 না):** SHA256 খুব **দ্রুত**, আক্রমণকারী সেকেন্ডে বিলিয়ন guess করতে পারে। bcrypt ইচ্ছা করে **ধীর** এবং **cost factor** দিয়ে ধীরতা বাড়ানো যায় (hardware শক্তিশালী হলে cost বাড়াও)।

```js
const bcrypt = require('bcrypt');

// Register
const hash = await bcrypt.hash(password, 12);     // 12 = cost (2^12 বার কাজ), salt নিজেই বানায়
await db.user.create({ data: { email, password: hash } });

// Login
const user = await db.user.findUnique({ where: { email } });
const ok = user && await bcrypt.compare(password, user.password);
if (!ok) return res.status(401).json({ message: 'Invalid credentials' });   // কোনটা ভুল তা বলো না
```
Hash দেখতে: `$2b$12$N9qo8uLOickgx2ZMRZoMye...` → `2b` = version, `12` = cost, তারপর **salt + hash** একসাথে।

**মনে রাখো:**
- Cost সাধারণত **10-12**
- bcrypt এর **৭২ বাইটের** সীমা আছে
- এটা CPU-ভারী, তাই এটা event loop block করে না (libuv thread pool এ চলে), কিন্তু লক্ষ লক্ষ login এ thread pool সীমা দেখা দিতে পারে
- আরো আধুনিক বিকল্প: **argon2id** (OWASP এর সুপারিশ)

---

## 🟡 A7. Salting

**Salt** = প্রতিটা password এ যোগ করা **র‍্যান্ডম, ইউনিক ডেটা**, hash করার আগে।

**সমস্যা (salt ছাড়া):** দুই user এর password একই (`123456`) হলে hash ও একই। আক্রমণকারী আগে থেকে বানানো **rainbow table** (সাধারণ password এর hash তালিকা) দিয়ে সহজে ভাঙতে পারে।

**Salt থাকলে:**
```
user1: hash('123456' + saltA) → X
user2: hash('123456' + saltB) → Y     // একই password, আলাদা hash!
```
- Rainbow table অকেজো (প্রতিটার জন্য আলাদা table লাগবে)
- একজনের password ভাঙলে অন্যদেরটা বোঝা যায় না
- Salt **গোপন নয়**, hash এর সাথেই থাকে। কাজ হলো ইউনিক করা
- **bcrypt নিজেই salt বানায় ও hash এ রাখে**, আলাদা করে সামলাতে হয় না

**Pepper** = সব password এর জন্য একটা গোপন value (env তে), বাড়তি স্তর।

---

## 🟡 A8. Brute Force আটকানো

Brute force = অসংখ্য password অনুমান করে লগইন চেষ্টা।

1. **Rate limit** লগইন route এ (IP + account দুই ভিত্তিতে)
2. **Account lockout / progressive delay** (৫ বার ভুলে ১৫ মিনিট লক, বা প্রতিবার দেরি বাড়ানো)
3. **CAPTCHA** (কয়েকবার ভুলের পর)
4. **2FA / MFA**
5. **ধীর hash** (bcrypt/argon2) যাতে offline cracking কঠিন হয়
6. **Generic error message**: "Invalid credentials" (কোন email আছে তা বোঝা যায় না)
7. শক্ত password নীতি, leaked password check (HIBP)
8. Monitoring ও alert, সন্দেহজনক IP block

```js
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 5,                                   // ১৫ মিনিটে ৫ বার
  skipSuccessfulRequests: true,
  message: { message: 'Too many attempts, try later' },
});
app.post('/auth/login', loginLimiter, login);
```
বহু server এ চললে Redis store (`rate-limit-redis`) লাগে।

---

## 🔴 A9. CSRF, XSS, SQL/NoSQL Injection

### 🛡️ XSS (Cross-Site Scripting)
**আক্রমণ:** আক্রমণকারী ওয়েবসাইটে ক্ষতিকর JavaScript ঢুকিয়ে দেয় (যেমন comment এ `<script>steal(document.cookie)</script>`), যা অন্য user এর ব্রাউজারে চলে।
**প্রতিরোধ:**
- Output **escape** করা (React ডিফল্টে করে; `dangerouslySetInnerHTML` এড়াও)
- User HTML দরকার হলে **sanitize** (`DOMPurify`)
- **CSP** header (Helmet)
- Token/session cookie `httpOnly`
- Input validation

### 🛡️ CSRF (Cross-Site Request Forgery)
**আক্রমণ:** তুমি bank এ লগইন করা আছো। অন্য একটা ক্ষতিকর সাইট তোমার ব্রাউজার দিয়ে bank এ গোপনে request পাঠায়, ব্রাউজার **নিজে থেকেই cookie** জুড়ে দেয়, bank ভাবে তুমিই পাঠিয়েছো।
**প্রতিরোধ:**
- **`SameSite=Lax/Strict`** cookie (সবচেয়ে সহজ ও কার্যকর)
- **CSRF token** (প্রতি form এ ইউনিক token)
- State-changing কাজে `GET` না, `POST/PUT/DELETE`
- Origin/Referer header যাচাই
- `Authorization: Bearer` header ব্যবহার করলে (cookie না) CSRF প্রায় প্রযোজ্য না

### 🛡️ SQL Injection
**আক্রমণ:** Input এ SQL কোড ঢুকিয়ে query বদলে ফেলা।
```js
// ❌ ভয়ংকর
db.query(`SELECT * FROM users WHERE email = '${email}' AND password = '${pw}'`);
// email = ' OR '1'='1' --   → সবাই লগইন করতে পারবে!

// ✅ Parameterized query
db.query('SELECT * FROM users WHERE email = $1', [email]);
// ✅ ORM (Prisma): স্বয়ংক্রিয় সুরক্ষা
prisma.user.findUnique({ where: { email } });
// ⚠️ Prisma raw: $queryRaw (tagged template, নিরাপদ) vs $queryRawUnsafe (ঝুঁকিপূর্ণ)
```
### 🛡️ NoSQL (MongoDB) Injection
**আক্রমণ:** Input এ object পাঠানো।
```js
// ❌ কেউ body তে { "email": "a@b.com", "password": { "$ne": null } } পাঠালে পাসওয়ার্ড ছাড়াই লগইন
User.findOne({ email: req.body.email, password: req.body.password });
```
**প্রতিরোধ:** Zod দিয়ে type যাচাই (`password` অবশ্যই string), `express-mongo-sanitize` (`$` ও `.` বাদ), আর password সরাসরি query তে না রেখে hash `compare`।

**সবার জন্য সাধারণ নিয়ম:** কখনো user input কে বিশ্বাস করো না: validate, sanitize, parameterize।

---

## 🟡 A10. Production এ Environment Variable Secure করা

1. **`.env` কখনো git এ না** (`.gitignore`), ভুলে দিয়ে ফেললে secret **rotate** করো (history থেকে মুছলেও যথেষ্ট না)
2. **Secret Manager:** AWS Secrets Manager, HashiCorp Vault, Doppler, GCP Secret Manager
3. **Platform এর env** (Vercel, Railway, Docker/K8s secrets), ফাইল বানিয়ে server এ রাখা না
4. **Environment আলাদা:** dev/staging/prod এ আলাদা secret
5. **Least privilege:** DB user এর শুধু দরকারি অনুমতি
6. **Secret rotation** নিয়মিত
7. **Log এ ছাপাবে না**, error response এ দেবে না
8. **Startup এ validate** (না থাকলে app চালু হবে না):
```js
const env = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
}).parse(process.env);
```
9. CI/CD এর secret গুলো encrypted variable এ
10. Docker image এ secret ঢুকিয়ে build করো না (`ENV` বা `COPY .env` ❌)
11. Access log ও audit

---

## 🟡 A11. Helmet.js ও গুরুত্ব

Helmet হলো middleware যা **security-related HTTP response header** সেট করে, ব্রাউজারকে নিরাপদ আচরণ করতে বলে।

```js
const helmet = require('helmet');
app.use(helmet());
```
**মূল header গুলো:**
| Header | কাজ |
|---|---|
| `Content-Security-Policy` | কোন source থেকে script/image লোড হবে সীমিত করে (**XSS** কমায়) |
| `Strict-Transport-Security` (HSTS) | ব্রাউজারকে সবসময় HTTPS ব্যবহার করতে বাধ্য করে |
| `X-Content-Type-Options: nosniff` | MIME sniffing বন্ধ |
| `X-Frame-Options` / frame-ancestors | **Clickjacking** আটকায় (iframe এ ঢোকানো) |
| `Referrer-Policy` | কতটুকু referrer তথ্য যাবে |
| `X-Powered-By` সরায় | কোন framework তা লুকায় |

**গুরুত্ব:** এক লাইনে অনেক সাধারণ আক্রমণ (XSS, clickjacking, sniffing) এর বিরুদ্ধে প্রতিরক্ষা। তবে এটা **একমাত্র সমাধান নয়**, defense-in-depth এর একটা স্তর। CSP কে নিজের প্রজেক্ট অনুযায়ী কনফিগার করতে হয়।

---

## 🔴 A12. Rate Limiting ও API Throttling

### 🧠 উপমা
Rate limit = দোকানের দরজায় প্রহরী: "প্রতি ঘণ্টায় একজন সর্বোচ্চ ১০০ বার ঢুকতে পারবে।" Throttling = ভিড় বেশি হলে লোক ঢোকানোর **গতি কমানো** (বাদ না দিয়ে ধীর করা)।

| | Rate Limiting | Throttling |
|---|---|---|
| কাজ | সীমা পার হলে **reject** (429) | request **ধীর করে/queue** এ রাখে |
| উদ্দেশ্য | অপব্যবহার, brute force, DoS আটকানো | লোড মসৃণ রাখা |

**Algorithms:**
- **Fixed Window**: প্রতি নির্দিষ্ট সময়ে N টা। সহজ, কিন্তু window এর সীমানায় burst হতে পারে
- **Sliding Window**: আরো সঠিক, চলমান সময়ের উপর
- **Token Bucket**: bucket এ token জমে, প্রতি request এ একটা খরচ। **burst** সহ্য করে, বহুল ব্যবহৃত
- **Leaky Bucket**: স্থির গতিতে প্রক্রিয়া

```js
const rateLimit = require('express-rate-limit');
const { RedisStore } = require('rate-limit-redis');

app.use('/api', rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 100,
  standardHeaders: true,                         // RateLimit-* header
  store: new RedisStore({ sendCommand: (...args) => redis.call(...args) }),  // বহু server এ shared
  keyGenerator: (req) => req.user?.id || req.ip,
}));
```
**গুরুত্বপূর্ণ বিষয়:**
- Response: **`429 Too Many Requests`** + **`Retry-After`** header
- **বহু instance** এ চললে **Redis** store (নইলে প্রতি server এ আলাদা count)
- Proxy এর পেছনে `app.set('trust proxy', 1)` নইলে সবার IP এক দেখাবে
- আলাদা limit: login কড়া, সাধারণ API নরম
- Key: IP, user ID, API key (পরিস্থিতি অনুযায়ী)
- বড় স্কেলে gateway/CDN স্তরে (Nginx, Cloudflare, API Gateway)

---

## 🟡 A13. API Key Authentication

সাধারণত **server-to-server** বা third-party developer দের জন্য।

**ডিজাইন:**
1. **Key তৈরি:** শক্ত র‍্যান্ডম (`crypto.randomBytes(32)`)
2. **একবারই দেখাও** user কে (Stripe এর মতো)
3. **DB তে hash করে রাখো** (SHA-256 যথেষ্ট, কারণ key নিজেই উচ্চ-entropy)। Plain রাখবে না
4. একটা **prefix** (`sk_live_ab12...`) রাখো যাতে শনাক্ত করা যায়
5. Client পাঠায়: `x-api-key: <key>` header
6. Middleware hash করে DB/cache এ খোঁজে
7. **Scopes/permissions**, per-key **rate limit**, expiry, **revoke/rotate**
8. সবসময় **HTTPS**, URL এ রাখবে না (log এ চলে যায়)
9. Usage log

```js
const crypto = require('crypto');

// তৈরি
const rawKey = 'sk_' + crypto.randomBytes(32).toString('hex');
const keyHash = crypto.createHash('sha256').update(rawKey).digest('hex');
await db.apiKey.create({ data: { userId, keyHash, prefix: rawKey.slice(0, 8), scopes: ['read'] } });

// যাচাই middleware
const apiKeyAuth = async (req, res, next) => {
  const key = req.header('x-api-key');
  if (!key) return res.status(401).json({ message: 'API key required' });
  const hash = crypto.createHash('sha256').update(key).digest('hex');
  const record = await db.apiKey.findUnique({ where: { keyHash: hash } });
  if (!record || record.revoked) return res.status(401).json({ message: 'Invalid API key' });
  req.apiKey = record; next();
};
```
**JWT vs API key:** API key দীর্ঘস্থায়ী, **application** শনাক্ত করে। JWT ছোট আয়ুর, **user** শনাক্ত করে।

---

# 📦 PART 6: Scenario-Based (১ম অংশ)

> প্রশ্ন S1–S5 | ⏱️ আনুমানিক সময়: ~৪০ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** S1, S5
> 🟡 **Good (মূল ধারণা):** S2, S3, S4

> 💡 **Scenario প্রশ্নের উত্তরের কৌশল:** (১) আগে **মাপো/বোঝো** (আন্দাজে না), (২) **কারণ খোঁজো**, (৩) **সমাধান**, (৪) **প্রতিরোধ**। এই ক্রমে বললে interviewer খুশি হয়।

---

## 🔴 S1. Heavy Traffic এ API হঠাৎ Slow

**ধাপ ১: মাপো ও বোঝো**
- Monitoring/APM (Datadog, New Relic, Grafana), logs, **কোন endpoint** ধীর?
- Metrics: CPU, memory, **event loop lag**, DB connection, response time (p95/p99), error rate
- `clinic doctor`, flamegraph; load test (`k6`, `autocannon`) দিয়ে reproduce

**ধাপ ২: সম্ভাব্য কারণ (কোথায় bottleneck?)**
| জায়গা | লক্ষণ | সমাধান |
|---|---|---|
| **Event loop block** | CPU 100%, event loop lag | sync কাজ সরাও, worker thread |
| **Database** (সবচেয়ে সাধারণ) | DB CPU/connection উচ্চ, slow query | index, N+1, pooling, cache |
| **Third-party API** | বাইরের call ধীর | timeout, cache, circuit breaker |
| **Memory/GC** | memory বাড়ছে, GC pause | leak খোঁজো |
| **Single instance** | একটা server এ সব | cluster/horizontal scale |
| **Connection pool শেষ** | request queue হচ্ছে | pool সাইজ ঠিক করো |

**ধাপ ৩: Fix**
- **Cache** (Redis) বারবার লাগা ডেটা
- **DB:** index, query optimize, read replica
- **Horizontal scaling:** load balancer + একাধিক instance, auto-scaling
- ভারী কাজ **queue** তে (BullMQ), সাথে সাথে response
- **Rate limiting**, compression, CDN, pagination
- **Graceful degradation**

**ধাপ ৪: প্রতিরোধ:** load test CI তে, alert, capacity planning, dashboard।

---

## 🟡 S2. Deploy এর পর User দের Random Logout

**সম্ভাব্য কারণ (গুরুত্ব অনুযায়ী):**
1. **JWT/session secret বদলে গেছে** বা বিভিন্ন instance এ আলাদা (সব পুরনো token invalid)
2. **In-memory session store**: restart এ সব session হারায়; বহু instance এ এক server এর session অন্যটা চেনে না → **Redis** লাগে
3. **Cookie configuration বদলেছে:** `domain`, `secure`, `sameSite`, `path`
4. **Proxy/HTTPS এর পেছনে `trust proxy` নেই** → `secure` cookie সেট হয় না
5. **Load balancer** sticky session ছাড়া (in-memory session থাকলে)
6. **CORS/credentials** ভুল (cookie যাচ্ছে না)
7. **Token expiry/clock skew** (server দের সময় মেলে না)
8. **Refresh token rotation race:** একাধিক tab একসাথে refresh করে, একটা invalid হয়ে যায়
9. **Environment variable ভুল** (staging এর secret production এ)
10. CDN/cache `Set-Cookie` সহ response cache করছে
11. Domain/subdomain বদল, HTTP↔HTTPS mismatch

**কীভাবে খুঁজবে:** logs এ 401 এর কারণ, কোন instance এ হচ্ছে, browser এ cookie/Network tab দেখা, deploy এর আগে-পরের config diff।
**প্রতিরোধ:** shared session store (Redis), secret rotation পরিকল্পিতভাবে (পুরনো key ও কিছুদিন গ্রহণ), rolling deploy, staging এ test।

---

## 🟡 S3. বড় ফাইল আপলোডে API Crash

**কারণ:** ফাইল পুরোটা memory তে (`multer` memoryStorage, `req.body` buffer), RAM শেষ → crash বা OOM।

**সমাধান:**
1. **Streaming** (`busboy`/`multer` diskStorage), RAM এ বাফার নয়
2. **Presigned URL**: client সরাসরি S3 তে আপলোড, server এ চাপ নেই
3. **Chunked/resumable upload** (`tus`)
4. **Limits:** max file size, `express.json({ limit })`
5. **Proxy সেটিংস:** Nginx এর `client_max_body_size`, timeout বাড়ানো
6. **Validation:** MIME, extension, content
7. ভারী প্রসেসিং (resize, scan) **background queue** তে
8. Temp ফাইল cleanup

বিস্তারিত code: `nodejs-interview-explain.md` এর প্রশ্ন ৬।

---

## 🟡 S4. Server Memory ক্রমাগত বাড়ছে

**তদন্ত (ধাপে ধাপে):**
1. **নিশ্চিত হও:** monitoring এ memory graph করাতের মতো (GC এ নামে) নাকি **শুধু উপরে উঠছে** (leak)?
2. `process.memoryUsage()` log, কোন endpoint এর পর বাড়ছে
3. **Heap snapshot** (`--inspect` + Chrome DevTools) আলাদা সময়ে দুটো নিয়ে **compare**
4. **Clinic.js heapprofiler**, `memlab`
5. Load test দিয়ে reproduce

**সাধারণ কারণ:** সীমাহীন cache/global array, remove না করা event listener, clear না করা timer/interval, closure এ বড় ডেটা, বন্ধ না করা DB connection/stream, request এর ডেটা global এ জমানো।

**সমাধান:** LRU/TTL cache, cleanup, stream ব্যবহার, `WeakMap`।
**সাময়িক ব্যবস্থা (মূল সমাধান না):** `--max-old-space-size` বাড়ানো, PM2/K8s এ memory limit এ auto-restart, rolling restart।
বিস্তারিত: `nodejs-interview-explain.md` এর প্রশ্ন ১২।

---

## 🔴 S5. লক্ষ লক্ষ User এর জন্য Scalable Authentication Design

**স্থাপত্য:**
1. **Stateless access token** (JWT, ১৫ মিনিট) → server এ lookup নেই, যেকোনো instance যাচাই করতে পারে
2. **Refresh token** DB/Redis এ (hash করে), **rotation + reuse detection**
3. **Redis**: session/blocklist/rate limit এর জন্য দ্রুত store
4. **Horizontal scaling:** auth service stateless, load balancer এর পেছনে auto-scale
5. **Asymmetric keys (RS256/EdDSA) + JWKS:** auth service সই করে, অন্য সার্ভিস শুধু public key দিয়ে যাচাই করে (secret share লাগে না), key rotation সহজ
6. **Password hashing CPU-ভারী:** argon2/bcrypt লগইন spike এ CPU bottleneck হতে পারে → আলাদা auth service/worker, cost ঠিক রাখা, queue/rate limit
7. **DB:** email এ **unique index**, read replica, connection pooling (PgBouncer/Neon pooler); পরে sharding
8. **Rate limiting + brute force সুরক্ষা** (Redis ভিত্তিক)
9. **MFA, OAuth/SSO (Google, GitHub)**, social login দিয়ে password এর চাপ কমানো
10. **Async কাজ queue তে:** verification email, notification
11. **আলাদা Auth microservice** বা managed (Auth0, Cognito, Better Auth)
12. **Monitoring, audit log**, anomaly detection
13. **Security:** HTTPS, httpOnly cookie, CSRF সুরক্ষা, token revocation

**Trade-off বলো:** JWT এ scale সহজ কিন্তু revoke কঠিন। তাই short-lived JWT + server-side refresh token এর hybrid।

---

# 📦 PART 7: Scenario-Based (২য় অংশ)

> প্রশ্ন S6–S10 | ⏱️ আনুমানিক সময়: ~৪৫ মিনিট
>
> 🔴 **Must (গভীরে পড়ো):** S6, S9, S10
> 🟡 **Good (মূল ধারণা):** S7, S8

---

## 🔴 S6. DB Query ৫-১০ সেকেন্ড লাগছে

**ধাপ ১: কোন query ধীর খোঁজো**
- **Slow query log** (Postgres `pg_stat_statements`), APM
- **`EXPLAIN ANALYZE`**: query plan দেখো। `Seq Scan` (পুরো table scan) মানে index দরকার

**ধাপ ২: Optimization**
1. **Index** যোগ করো (`WHERE`, `JOIN`, `ORDER BY` এর column এ), composite index এর ক্রম খেয়াল রাখো
2. **N+1 ঠিক করা** (join/`include`/batch)
3. `SELECT *` এড়াও, **শুধু দরকারি column**
4. **Pagination**, `OFFSET` বড় হলে ধীর, **cursor pagination** ব্যবহার করো
5. **Connection pooling** (PgBouncer/Neon pooler)
6. **Caching** (Redis) বারবার লাগা ও কম বদলানো ডেটা
7. **Query rewrite:** subquery এর বদলে join, অপ্রয়োজনীয় `DISTINCT`/`ORDER BY` বাদ
8. **Denormalization / materialized view** ভারী aggregate এর জন্য
9. **Read replica** পড়ার লোড সরাতে
10. **Long transaction/lock** এড়াও
11. **Partitioning/archive** পুরনো বিশাল ডেটা
12. DB hardware/config (সবশেষে)

```js
// ❌ N+1
const posts = await prisma.post.findMany();
for (const p of posts) p.author = await prisma.user.findUnique({ where: { id: p.authorId } });

// ✅ একটাই query, দরকারি field
const posts = await prisma.post.findMany({
  select: { id: true, title: true, author: { select: { name: true } } },
  take: 20,
  cursor: lastId ? { id: lastId } : undefined,   // cursor pagination
  orderBy: { id: 'asc' },
});
```
```prisma
model Post {
  id       Int  @id
  authorId Int
  @@index([authorId])        // foreign key এ index
}
```
**পরিমাপ করে যাচাই করো:** index দেওয়ার পর `EXPLAIN` এ `Index Scan` আসছে কিনা।

---

## 🟡 S7. Public API এর জন্য Rate Limiting

1. **কী দিয়ে চিনবে:** IP (বেনামী), **API key** (প্রতি client), user ID (লগইন করা)
2. **Tiered limit:** free 100/ঘণ্টা, pro 10,000/ঘণ্টা
3. **Algorithm:** token bucket (burst সহ্য করে + গড় সীমা)
4. **Shared store: Redis** (বহু instance এ একই count)। Atomic ops (`INCR` + `EXPIRE` বা Lua script)
5. **Endpoint ভেদে আলাদা:** login/search কড়া, সহজ GET নরম
6. **Response:** `429` + `Retry-After` + `RateLimit-Limit/Remaining/Reset` header
7. **Proxy এর পেছনে** `trust proxy`, নইলে সব user এক IP
8. **Gateway/CDN স্তরে:** Cloudflare, Nginx, API Gateway (app এ আসার আগেই আটকানো)
9. **Quota** (দৈনিক/মাসিক) আলাদা, burst limit আলাদা
10. **Abuse detection:** বারবার ভঙ্গ করলে temporary block
11. Monitoring ও dashboard, ডক্যুমেন্টেশনে limit জানাও

---

## 🟡 S8. Local এ চলে, Production এ Fail

**Checklist (সম্ভাবনা অনুযায়ী):**
1. **Environment variable** নেই বা ভুল (সবচেয়ে সাধারণ)
2. **Node/package version** আলাদা, `npm ci` ও lock file
3. **Case-sensitive filesystem:** Windows/Mac এ `./User` ঠিক, Linux এ `./user` না হলে ❌
4. **`devDependencies`** production এ install হয়নি, কিন্তু runtime এ লাগছে
5. **Build step** বাদ (TypeScript compile হয়নি)
6. **Port/host binding:** `0.0.0.0` এ listen না করা (container এ `localhost` এ listen করলে বাইরে থেকে পাওয়া যায় না), `process.env.PORT`
7. **Database:** connection string, **firewall/IP allowlist**, **SSL** দরকার, connection limit
8. **CORS, HTTPS, cookie** (`secure`, `sameSite`, `trust proxy`)
9. **File path/permission**, write access, ephemeral disk
10. **Memory/CPU limit** (container এ OOM kill)
11. **Timezone/locale** পার্থক্য
12. **Reverse proxy** (Nginx) এর body size/timeout/header

**কীভাবে debug:**
- **Logs দেখো** (সবার আগে), structured logging
- Health check endpoint
- **Docker দিয়ে local এ production এর মতো পরিবেশ** বানিয়ে reproduce
- Staging environment
- Error tracking (Sentry)

**প্রতিরোধ:** Docker, env validation (Zod), CI/CD এ build+test, staging, `.env.example`।

---

## 🔴 S9. Booking/Payment এ Duplicate Transaction আটকানো

### 🧠 সমস্যা
User দুইবার "Pay" ক্লিক করল, নেটওয়ার্ক retry হলো, বা webhook দুইবার এলো → **দুইবার টাকা কাটা**, বা **একই সিট দুজনের**।

### সমাধান ১: **Idempotency Key** (সবচেয়ে গুরুত্বপূর্ণ)
Client প্রতি **ইচ্ছা (intent)** এর জন্য একটা ইউনিক key (UUID) পাঠায়। Server ওই key দিয়ে ফলাফল সংরক্ষণ করে; একই key আবার এলে **আগের ফলাফলই** ফেরত দেয়, নতুন করে চালায় না।

```js
app.post('/payments', async (req, res) => {
  const key = req.header('Idempotency-Key');
  if (!key) return res.status(400).json({ message: 'Idempotency-Key required' });

  // key unique constraint দিয়ে "প্রথম হওয়া" নিশ্চিত
  try {
    await db.idempotency.create({ data: { key, status: 'processing' } });
  } catch (e) {
    if (e.code === 'P2002') {                              // duplicate key
      const saved = await db.idempotency.findUnique({ where: { key } });
      return res.status(saved.statusCode ?? 409).json(saved.response ?? { message: 'In progress' });
    }
    throw e;
  }
  const result = await chargeCustomer(req.body);
  await db.idempotency.update({ where: { key }, data: { status: 'done', statusCode: 201, response: result } });
  res.status(201).json(result);
});
```
### সমাধান ২: **DB Constraint** (শেষ সুরক্ষা)
```prisma
model Booking {
  id     Int @id
  roomId Int
  date   DateTime
  @@unique([roomId, date])     // একই রুম একই তারিখে দুবার বুক হবে না
}
```
Race condition এ DB নিজেই দ্বিতীয়টা আটকায় (`P2002` → 409)।

### সমাধান ৩: **Transaction + Lock**
```js
await prisma.$transaction(async (tx) => {
  const [seat] = await tx.$queryRaw`SELECT * FROM "Seat" WHERE id = ${id} FOR UPDATE`;  // row lock
  if (seat.booked) throw new AppError('Already booked', 409);
  await tx.seat.update({ where: { id }, data: { booked: true } });
  await tx.payment.create({ data: { seatId: id, amount } });
});
```
বিকল্প: **Optimistic locking** (`version` column, update এর সময় version মেলে কিনা চেক)।

### বাকি কৌশল
- **State machine:** `pending → paid → confirmed`, অবৈধ ট্রানজিশন বন্ধ
- **Webhook idempotent:** payment gateway (Stripe/bKash) একই event দুবার পাঠাতে পারে। `event.id` সংরক্ষণ করে ডুপ্লিকেট এড়াও
- **UI:** button disable, debounce (সাহায্য করে, কিন্তু **শুধু এটা যথেষ্ট না**, backend সুরক্ষা লাগবেই)
- **Redis distributed lock** (`SET NX`) সংক্ষিপ্ত সময়ের জন্য
- Payment gateway এর নিজস্ব idempotency key ব্যবহার করো
- **Reconciliation job** (gateway র সাথে নিয়মিত মেলানো), audit log

---

## 🔴 S10. ধীর ও অনির্ভরযোগ্য Third-Party API এর চারপাশে Design

### 🧠 মূল নীতি
**বাইরের সিস্টেমের ব্যর্থতা যেন তোমার সিস্টেম না ডোবায়।**

1. **Timeout সবসময় সেট করো** (নইলে request ঝুলে থেকে connection/thread শেষ করে দেয়)
```js
const res = await fetch(url, { signal: AbortSignal.timeout(3000) });   // ৩ সেকেন্ড
```
2. **Retry + Exponential Backoff + Jitter**: ব্যর্থ হলে আবার, কিন্তু বাড়তে থাকা বিরতিতে (১s, ২s, ৪s) + র‍্যান্ডম কম-বেশি (সবাই একসাথে retry না করতে)। **শুধু idempotent কাজে** retry (payment এ সাবধান)
3. **Circuit Breaker** (`opossum` library): বারবার fail করলে কিছুক্ষণ **call বন্ধ** (fail fast), পরে একটু একটু করে চালু করে দেখো। তৃতীয় পক্ষ ডুবলে তুমি না ডোবো।
```js
const CircuitBreaker = require('opossum');
const breaker = new CircuitBreaker(callThirdParty, { timeout: 3000, errorThresholdPercentage: 50, resetTimeout: 30000 });
breaker.fallback(() => cachedOrDefaultData);
const data = await breaker.fire(params);
```
4. **Cache:** একই ডেটা বারবার না আনা (Redis), পুরনো (stale) ডেটাও fallback হিসেবে
5. **Fallback / Graceful degradation:** ডেটা না পেলে ডিফল্ট দেখাও, ফিচার আংশিক বন্ধ রাখো, পুরো সিস্টেম নয়
6. **Async/Queue:** user কে সাথে সাথে response, third-party কাজ **background job** (BullMQ) এ, retry সহ, ফলাফল **webhook/polling** এ
7. **Bulkhead:** third-party এর জন্য আলাদা connection pool/concurrency limit, যাতে বাকি সিস্টেম প্রভাবিত না হয়
8. **Rate limit মেনে চলা** (তাদের সীমা পার না করা)
9. **Adapter/wrapper layer**: তৃতীয় পক্ষের কোড এক জায়গায়, বদলানো ও mock সহজ
10. **Monitoring, alert, SLA ট্র্যাক**, লগে latency ও error rate
11. **Test এ mock/stub**; contract test
12. বিকল্প provider (failover) যদি সম্ভব

**উত্তরের কাঠামো:** "Timeout → Retry with backoff → Circuit breaker → Cache/fallback → Async via queue → Monitoring।"

---

## 🎯 ৫ মিনিটের Revision

| বিষয় | এক লাইনে |
|---|---|
| Lifecycle | request → middleware stack (ক্রমানুসারে) → route → response; error → error handler |
| Middleware | `(req,res,next)`; `next()` বা response, নইলে ঝুলে যায় |
| `next()` | stack এর index closure; `next(err)` error handler এ লাফ দেয় |
| Structure | route → controller → service → repository; feature-based folder |
| REST | noun, সঠিক method ও status, versioning, pagination, consistent format |
| Error handling | AppError + asyncHandler + একটাই error middleware (শেষে) |
| Validation | Zod middleware, body/params/query, whitelist |
| AuthN vs AuthZ | তুমি কে (401) vs কী পারো (403) |
| RBAC | role/permission middleware + **ownership চেক** |
| CORS | ব্রাউজার-ভিত্তিক; `credentials` এর সাথে `*` চলে না; preflight |
| Session vs JWT | server-state (revoke সহজ) vs token-state (scale সহজ) |
| JWT | header.payload.signature; encoded, encrypted না |
| JWT সমস্যা | revoke কঠিন; short expiry + refresh rotation |
| Token storage | access = memory; refresh = httpOnly cookie |
| bcrypt | ধীর, salt সহ, cost 10-12 |
| Brute force | rate limit + lockout + CAPTCHA + 2FA |
| XSS/CSRF/Injection | escape+CSP / SameSite+token / parameterized query |
| Rate limit | token bucket; Redis; 429 + Retry-After |
| Scenario কৌশল | মাপো → কারণ → সমাধান → প্রতিরোধ |
| Duplicate payment | idempotency key + DB unique + transaction/lock |
| Third-party | timeout → retry → circuit breaker → cache/fallback → queue |

শুভকামনা! 🚀
