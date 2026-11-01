# 🎤 Express.js, Auth ও Security, Scenario: Interview এ যেভাবে বলবে

> শুধু **মুখে বলার উত্তর**। ব্যাখ্যা ও code এর জন্য `express-interview-explain.md` দেখো।
> প্রতিটা ৩০-৬০ সেকেন্ডের। নিজের ভাষায় একটু বদলে বলবে, মুখস্থ শোনালে ধরা পড়ে।

> **রঙের মানে:** 🔴 অবশ্যই পড়ো | 🟡 মূল ধারণা জানলেই চলবে | ⚪ সময় থাকলে
> **নম্বর:** `E` = Express, `A` = Auth ও Security, `S` = Scenario

---

# 📦 PART 1: Express Core

> প্রশ্ন E1–E4 | 🔴 Must: E1, E2, E4 | 🟡 Good: E3

---

### 🔴 E1. Request-Response Lifecycle
> "Client request পাঠালে Node এর http server সেটা ধরে Express app কে দেয়। তারপর request middleware stack এর উপর দিয়ে যে ক্রমে লেখা হয়েছে সেই ক্রমে যায়, যেমন `express.json()`, logger, auth। প্রতিটা middleware হয় response দিয়ে চক্র শেষ করে, নয়তো `next()` দিয়ে পরেরটায় পাঠায়। Route match হলে সেই route এর handler চলে আর `res.json()` দিয়ে response যায়। মাঝে কোথাও error হলে সোজা error-handling middleware এ যায়, আর কোনো route না মিললে শেষের 404 handler চলে। তাই middleware এর ক্রম খুব গুরুত্বপূর্ণ।"

### 🔴 E2. Middleware কী
> "Middleware হলো এমন function যেটা `req`, `res` আর `next` পায়। এটা request এর আর response এর মাঝখানে চেকপয়েন্টের মতো কাজ করে। তিনটা কাজের একটা করতে পারে: `req` বা `res` পরিবর্তন করা, যেমন `req.user` সেট করা, অথবা response দিয়ে চক্র শেষ করা, অথবা `next()` দিয়ে পরের middleware এ পাঠানো। `next()` বা response কোনোটাই না করলে request ঝুলে থাকে। Logging, authentication, validation, CORS, rate limiting, body parsing সবই middleware দিয়ে হয়।"

### 🟡 E3. Application-level, Router-level, Error-handling Middleware
> "Application-level middleware `app.use()` দিয়ে বসে আর পুরো app এ প্রযোজ্য হয়, যেমন `express.json()` বা logger। Router-level middleware `router.use()` দিয়ে বসে আর শুধু সেই router এর route গুলোয় কাজ করে, যেমন `/admin` router এ admin check। Error-handling middleware এর চারটা parameter থাকে, `err, req, res, next`, আর সেই চার parameter ই Express কে বলে এটা error handler। এটা সবসময় সব route আর middleware এর শেষে রাখতে হয়।"

### 🔴 E4. `next()` ভেতরে কীভাবে কাজ করে
> "Express ভেতরে প্রতিটা middleware আর route কে একটা layer হিসেবে একটা stack এ রাখে। `next` হলো একটা closure যেটা একটা index মনে রাখে। `next()` call হলে index বাড়িয়ে পরের matching layer, মানে path আর method মেলে এমনটা, খুঁজে চালায়। `next(err)` call করলে error mode চালু হয়: সাধারণ middleware গুলো skip হয়ে সোজা চার-parameter এর error handler এ যায়। `next('route')` বর্তমান route এর বাকি handler skip করে পরের route এ যায়। Sync কোডে throw করলেও Express ধরে `next(err)` এর মতো কাজ করে, আর Express 5 এ async function এর rejection ও নিজে থেকেই error handler এ যায়।"

---

# 📦 PART 2: Structure, API Design, Error ও Validation

> প্রশ্ন E5–E9 | 🔴 Must: E5, E6, E7, E8 | 🟡 Good: E9

---

### 🔴 E5. Scalable Project Structure
> "আমি layered architecture ব্যবহার করি: Route থেকে Controller, তারপর Service, তারপর Repository। Route শুধু URL কে controller এর সাথে জোড়ে, controller পাতলা থাকে আর শুধু request আর response সামলায়, business logic থাকে service এ, আর database query থাকে repository তে। Folder সাজাই feature-based, যেমন `users/`, `orders/`, প্রতিটার ভেতরে নিজস্ব route, controller, service আর validation। `app.js` আর `server.js` আলাদা রাখি যাতে test এ app আলাদাভাবে নেওয়া যায়। Config environment থেকে আসে, কোনো secret hardcode করি না। সাথে centralized error handling, logger আর background job এর জন্য আলাদা folder থাকে।"

### 🔴 E6. REST API Best Practices
> "URL এ noun ব্যবহার করি, plural, যেমন `/users`, আর কাজের ধরন HTTP method দিয়ে বোঝাই: GET, POST, PUT, PATCH, DELETE। সঠিক status code দিই, যেমন create এ 201, validation ভুলে 400, লগইন না থাকলে 401, অনুমতি না থাকলে 403, না পেলে 404, duplicate এ 409। API তে versioning রাখি, `/api/v1`, আর list এ pagination, filtering, sorting দিই, বড় ডেটায় cursor pagination। Response ফরম্যাট সবখানে একই রাখি, success আর error দুটোর জন্যই। Validation, authentication, rate limiting আর HTTPS সবসময় থাকে, আর Swagger বা OpenAPI দিয়ে documentation করি। ভেতরের তথ্য যেমন stack trace কখনো response এ দিই না।"

### 🔴 E7. Centralized Error Handling
> "প্রতিটা route এ আলাদা try/catch না লিখে আমি একটাই জায়গায় সব error সামলাই। প্রথমে একটা custom `AppError` class বানাই যেটায় status code আর একটা `isOperational` flag থাকে, মানে প্রত্যাশিত error। Async handler এ একটা wrapper ব্যবহার করি যাতে rejected Promise `next(err)` এ যায়, Express 5 এ এটা নিজেই হয়। তারপর সব route এর শেষে একটা 404 handler আর একটা চার-parameter এর central error middleware রাখি। সেই middleware error log করে আর সবসময় একই ফরম্যাটে response দেয়। Production এ অপ্রত্যাশিত error এর ক্ষেত্রে সাধারণ message দিই আর stack trace লুকিয়ে রাখি। Prisma বা Zod এর error কেও এখানেই 409 বা 400 এ রূপান্তর করি।"

### 🔴 E8. Request Data Validation
> "Client এর ডেটা কখনো বিশ্বাস করা যায় না, তাই backend validation বাধ্যতামূলক। আমি Zod ব্যবহার করি কারণ TypeScript এর সাথে ভালো মেলে। একটা reusable `validate(schema)` middleware বানাই যেটা request এর body, params আর query তিনটাই যাচাই করে। ভুল হলে 400 দিয়ে বিস্তারিত error পাঠায়, আর সফল হলে পরিষ্কার ডেটা `req.body` এ বসায়। এতে অচেনা field বাদ পড়ে, যা mass assignment আটকায়, যেমন কেউ `role: 'admin'` পাঠাতে পারে না। Query parameter সব string আসে, তাই `z.coerce` দিয়ে সংখ্যায় রূপান্তর করি। Validation controller এর আগেই middleware এ হয়।"

### 🟡 E9. Route Modularization
> "প্রতিটা feature এর route আলাদা ফাইলে `express.Router()` দিয়ে লিখি, তারপর main app এ `app.use('/api/v1/users', userRoutes)` দিয়ে mount করি। Router এর ভেতরে router-level middleware বসানো যায়, যেমন পুরো `/admin` এ authentication। এতে ফাইল ছোট থাকে, maintenance সহজ হয়, versioning সহজ হয়, আর টিমের বিভিন্নজন আলাদা module এ কাজ করতে পারে।"

---

# 📦 PART 3: Auth Basics ও Express Security

> প্রশ্ন E10–E15 | 🔴 Must: E10, E11, E12, E13 | 🟡 Good: E14, E15

---

### 🔴 E10. Authentication vs Authorization
> "Authentication হলো তুমি কে তা প্রমাণ করা, যেমন password, OTP বা token দিয়ে। Authorization হলো তুমি কী করতে পারো তা ঠিক করা। অফিস বিল্ডিং এর উদাহরণে, গেটে আইডি কার্ড দেখানো authentication, আর সেই কার্ড দিয়ে কোন কোন রুমে ঢুকতে পারি সেটা authorization। Authentication আগে হয়, তারপর authorization। ব্যর্থ হলে authentication এ 401 Unauthorized, আর authorization এ 403 Forbidden দিই।"

### 🔴 E11. RBAC
> "RBAC এ user এর role অনুযায়ী অ্যাক্সেস ঠিক হয়, যেমন admin, editor, user, আর প্রতিটা role এর সাথে permission জোড়া থাকে। Express এ আমি `authorize('admin', 'editor')` ধরনের একটা middleware বানাই যেটা authentication এর পরে চলে আর `req.user.role` মিলিয়ে দেখে, না মিললে 403 দেয়। বড় সিস্টেমে role আর permission আলাদা table এ রাখি, many-to-many সম্পর্কে, যাতে code না বদলিয়ে role বদলানো যায়। একটা জরুরি জিনিস হলো ownership চেক, শুধু role দেখলে হয় না, যেমন user শুধু নিজের post ই edit করতে পারবে। আর JWT তে role রাখলে role বদলালে token পুরনো থাকে, তাই sensitive কাজে DB থেকে আবার যাচাই করি।"

### 🔴 E12. Express API Secure করা
> "আমি স্তরে স্তরে নিরাপত্তা দিই। HTTPS বাধ্যতামূলক, Helmet দিয়ে security header, CORS শুধু বিশ্বস্ত origin এর জন্য, আর rate limiting। সব input Zod দিয়ে validate করি আর ORM বা parameterized query ব্যবহার করি injection আটকাতে। Password bcrypt বা argon2 দিয়ে hash করি। Token ছোট আয়ুর রাখি, আর cookie তে `httpOnly`, `secure`, `sameSite` flag দিই। Body size limit রাখি, error এ stack trace লুকাই, secret environment variable বা secret manager এ রাখি, আর `npm audit` দিয়ে dependency নজরে রাখি। সাথে logging আর monitoring।"

### 🔴 E13. CORS ও সাধারণ সমস্যা
> "Origin হলো protocol, domain আর port এর সমষ্টি। ব্রাউজারের same-origin policy এক origin এর JS কে অন্য origin এর response পড়তে দেয় না। CORS হলো server এর বলে দেওয়া, কোন origin গুলোকে অনুমতি আছে। এটা শুধু ব্রাউজারে প্রযোজ্য আর server কে নিরাপদ করে না। Non-simple request এর আগে ব্রাউজার একটা OPTIONS preflight request পাঠায়। সাধারণ সমস্যাগুলো হলো: `credentials: true` এর সাথে origin `*` দিলে কাজ করে না, সুনির্দিষ্ট origin দিতে হয়। Client এ `credentials: 'include'` না থাকা। `cors()` middleware route এর পরে বসানো। `Authorization` header `allowedHeaders` এ না থাকা। আর preflight request auth middleware এ আটকে 401 পাওয়া। Debug করতে Network tab এ OPTIONS request দেখি।"

### 🟡 E14. Cookies ও Sessions
> "Cookie হলো ব্রাউজারে রাখা ছোট ডেটা, যেটা ব্রাউজার প্রতিটা request এ নিজে থেকেই পাঠায়। Session এ আসল ডেটা server এ থাকে, আর cookie তে শুধু একটা session ID থাকে। Login হলে server session বানিয়ে ডেটা store এ রাখে আর `Set-Cookie` এ ID পাঠায়, পরের request এ সেই ID দিয়ে user চেনে। Production এ default MemoryStore ব্যবহার করা যায় না, Redis এর মতো shared store লাগে। Cookie তে `httpOnly` দিলে JS পড়তে পারে না, `secure` দিলে শুধু HTTPS এ যায়, আর `sameSite` CSRF কমায়। লগইনের পর session regenerate করি যাতে session fixation না হয়।"

### 🟡 E15. Stateless vs Stateful Authentication
> "Stateful এ, যেমন session, state থাকে server এ, প্রতিটা request এ store এ lookup হয়। এতে logout বা revoke সহজ, কিন্তু scale করতে shared store লাগে। Stateless এ, যেমন JWT, সব তথ্য token এই থাকে, server শুধু signature যাচাই করে, তাই যেকোনো instance যাচাই করতে পারে আর scale সহজ। কিন্তু token revoke করা কঠিন, expire পর্যন্ত valid থাকে। সাধারণ web app এ session সহজ ও নিরাপদ, আর mobile বা microservice এ JWT সুবিধাজনক। বাস্তবে আমি hybrid ব্যবহার করি: ছোট আয়ুর JWT access token আর server এ সংরক্ষিত refresh token।"

---

# 📦 PART 4: JWT ও Token

> প্রশ্ন A1–A5 | 🔴 Must: A1, A2, A3, A5 | 🟡 Good: A4

---

### 🔴 A1. JWT কীভাবে কাজ করে
> "JWT এর তিনটা অংশ: header, payload আর signature, যা ডট দিয়ে জোড়া আর base64url এ encoded। Header এ algorithm থাকে, payload এ claim যেমন user id, role আর expiry, আর signature তৈরি হয় header ও payload কে secret key দিয়ে sign করে। Login সফল হলে server token বানিয়ে দেয়, client প্রতিটা request এ `Authorization: Bearer` header এ পাঠায়, আর server শুধু signature ও expiry যাচাই করে, কোনো DB lookup ছাড়াই user চেনে। কেউ payload বদলালে signature মিলবে না। তবে payload encrypted না, শুধু encoded, যে কেউ পড়তে পারে, তাই সংবেদনশীল তথ্য রাখি না। আর verify করার সময় `algorithms` স্পষ্ট দিই।"

### 🔴 A2. JWT এর অসুবিধা
> "সবচেয়ে বড় অসুবিধা হলো revoke করা কঠিন, একবার ইস্যু হলে expiry পর্যন্ত valid, logout করলেও। Token চুরি হলে চোর সেই সময় পর্যন্ত ব্যবহার করতে পারে। Payload পড়া যায় আর size বড়। Role বদলালে পুরনো token এ পুরনো role থাকে। Storage এ সমস্যা: localStorage এ XSS ঝুঁকি, আর cookie তে CSRF ঝুঁকি। Secret leak হলে যে কেউ নকল token বানাতে পারে, আর ভুল configuration, যেমন `alg: none` বা দুর্বল secret, ঝুঁকি বাড়ায়। সমাধান হিসেবে access token ছোট আয়ুর রাখি, refresh token rotation ব্যবহার করি, দরকারে Redis এ blocklist রাখি, আর শক্ত secret বা asymmetric key ব্যবহার করি।"

### 🔴 A3. Refresh Token Flow
> "Login এ server দুটো token দেয়: ছোট আয়ুর access token, ধরো ১৫ মিনিট, আর বড় আয়ুর refresh token, ধরো ৭ দিন। Refresh token যায় httpOnly cookie তে, আর DB তে hash করে রাখা হয়। Access token expire হলে API 401 দেয়, client তখন `/auth/refresh` call করে। Server refresh token যাচাই করে DB তে আছে কিনা দেখে, তারপর নতুন access token আর নতুন refresh token দেয় আর পুরনোটা বাতিল করে, একে বলে rotation। যদি বাতিল হওয়া refresh token আবার আসে, মানে চুরি হয়েছে, তখন ওই user এর সব session বাতিল করি, একে reuse detection বলে। Logout এ refresh token DB থেকে মুছে cookie clear করি।"

### 🟡 A4. Access Token vs Refresh Token
> "Access token দিয়ে API অ্যাক্সেস হয়, তাই এর আয়ু ছোট, ৫ থেকে ১৫ মিনিট, প্রতিটা request এ যায়, আর সাধারণত server এ সংরক্ষণ করা হয় না। Refresh token শুধু নতুন access token আনার জন্য, আয়ু বড়, শুধু refresh endpoint এ যায়, httpOnly cookie তে থাকে, আর server এ hash করে রাখা হয় যাতে revoke করা যায়। দুটো রাখার কারণ নিরাপত্তা আর সুবিধা: ছোট আয়ুর token চুরি হলে ক্ষতি সীমিত, আর user কে বারবার লগইন করতে হয় না।"

### 🔴 A5. Token নিরাপদে রাখা
> "localStorage এ token রাখা ঝুঁকিপূর্ণ কারণ XSS হলে যেকোনো JS সেটা পড়তে পারে। আমার পছন্দের কৌশল হলো access token JS memory তে রাখা, আর refresh token `httpOnly`, `Secure`, `SameSite` cookie তে রাখা, যেটা JS পড়তে পারে না। Page reload হলে refresh endpoint থেকে নতুন access token আনি। Server এ refresh token hash করে রাখি যাতে DB leak হলেও কাজে না লাগে। সবসময় HTTPS, XSS কমাতে CSP আর input sanitize। Mobile এ iOS Keychain বা Android Keystore ব্যবহার করি। আর token কখনো log এ বা URL এ রাখি না।"

---

# 📦 PART 5: Password, Attack Protection ও Security Tools

> প্রশ্ন A6–A13 | 🔴 Must: A6, A9, A12 | 🟡 Good: A7, A8, A10, A11, A13

---

### 🔴 A6. bcrypt দিয়ে Password Hashing
> "Password কখনো plain text এ রাখা যাবে না, hash করে রাখতে হয়। Hashing এক-মুখী, ফেরত আনা যায় না, আর encryption দুই-মুখী। SHA256 এর মতো দ্রুত hash এ আক্রমণকারী সেকেন্ডে বিলিয়ন guess করতে পারে, তাই bcrypt ব্যবহার করি যেটা ইচ্ছা করে ধীর এবং cost factor দিয়ে ধীরতা বাড়ানো যায়। আমি `bcrypt.hash(password, 12)` দিয়ে hash করি, cost সাধারণত ১০ থেকে ১২। Login এ `bcrypt.compare` দিয়ে মিলাই। Bcrypt নিজেই salt বানিয়ে hash এর সাথে রাখে। ভুল হলে কোনটা ভুল তা না বলে generic message দিই। আরো আধুনিক বিকল্প argon2id।"

### 🟡 A7. Salting
> "Salt হলো প্রতিটা password এর জন্য আলাদা র‍্যান্ডম ডেটা যা hash করার আগে যোগ হয়। Salt না থাকলে দুজনের একই password এর hash ও একই হতো, আর আক্রমণকারী আগে থেকে বানানো rainbow table দিয়ে সহজে ভাঙতে পারত। Salt থাকলে একই password এরও আলাদা hash হয়, তাই rainbow table অকেজো হয়ে যায় আর একজনের password ভাঙলে অন্যদেরটা বোঝা যায় না। Salt গোপন নয়, hash এর সাথেই থাকে, কাজ শুধু ইউনিক করা। Bcrypt এটা নিজেই করে। এর উপরে pepper নামে একটা গোপন value ও env তে রাখা যায়।"

### 🟡 A8. Brute Force আটকানো
> "কয়েকটা স্তরে আটকাই। Login route এ কড়া rate limit, যেমন ১৫ মিনিটে ৫ বার, IP আর account দুই ভিত্তিতে। বারবার ভুল হলে account সাময়িক lock বা progressive delay। কয়েকবার ভুলের পর CAPTCHA। 2FA চালু করা। Bcrypt বা argon2 এর মতো ধীর hash যাতে offline cracking কঠিন হয়। ভুল হলে generic message, যেমন 'Invalid credentials', যাতে কোন email আছে তা বোঝা না যায়। সাথে সন্দেহজনক IP এর monitoring আর alert। বহু server এ চললে rate limit এর count Redis এ রাখি।"

### 🔴 A9. CSRF, XSS, SQL/NoSQL Injection
> "XSS এ আক্রমণকারী ওয়েবসাইটে ক্ষতিকর JavaScript ঢুকিয়ে দেয় যা অন্য user এর ব্রাউজারে চলে। প্রতিরোধ: output escape করা, দরকারে DOMPurify দিয়ে sanitize, CSP header, আর cookie `httpOnly`। CSRF এ user লগইন করা থাকলে অন্য ক্ষতিকর সাইট তার ব্রাউজার দিয়ে গোপনে request পাঠায়, আর ব্রাউজার নিজেই cookie জুড়ে দেয়। প্রতিরোধ: `SameSite` cookie, CSRF token, আর state বদলানো কাজে GET না ব্যবহার করা। SQL injection এ input এ SQL কোড ঢুকিয়ে query বদলে ফেলা হয়, প্রতিরোধ parameterized query বা Prisma এর মতো ORM, আর raw query তে unsafe variant এড়ানো। NoSQL injection এ MongoDB তে `{ $ne: null }` এর মতো object পাঠিয়ে password ছাড়াই লগইন করা যায়। প্রতিরোধ: Zod দিয়ে type যাচাই, `express-mongo-sanitize`। সবার মূল নিয়ম: user input কে বিশ্বাস না করা, validate, sanitize আর parameterize করা।"

### 🟡 A10. Environment Variable Secure করা
> "প্রথম নিয়ম, `.env` কখনো git এ যাবে না। ভুলে চলে গেলে history থেকে মুছলেও যথেষ্ট না, secret rotate করতে হয়। Production এ AWS Secrets Manager, Vault বা Doppler এর মতো secret manager অথবা platform এর encrypted environment variable ব্যবহার করি। Dev, staging আর production এ আলাদা secret রাখি। DB user এ least privilege দিই, secret নিয়মিত rotate করি, আর log বা error response এ কখনো ছাপাই না। Startup এ Zod দিয়ে env validate করি, না থাকলে app চালুই হয় না। Docker image এ secret ঢুকিয়ে build করি না।"

### 🟡 A11. Helmet.js
> "Helmet হলো একটা middleware যেটা security সম্পর্কিত HTTP response header সেট করে। যেমন Content-Security-Policy যেটা কোন source থেকে script লোড হবে সীমিত করে আর XSS কমায়। Strict-Transport-Security ব্রাউজারকে সবসময় HTTPS এ যেতে বাধ্য করে। `X-Content-Type-Options: nosniff` MIME sniffing বন্ধ করে, frame protection clickjacking আটকায়, আর `X-Powered-By` সরিয়ে framework এর তথ্য লুকায়। এক লাইনে `app.use(helmet())` দিয়ে অনেক সাধারণ আক্রমণের বিরুদ্ধে প্রতিরক্ষা পাওয়া যায়। তবে এটা একমাত্র সমাধান না, defense-in-depth এর একটা স্তর, আর CSP নিজের প্রজেক্ট অনুযায়ী কনফিগার করতে হয়।"

### 🔴 A12. Rate Limiting ও Throttling
> "Rate limiting হলো নির্দিষ্ট সময়ে একজন কতগুলো request করতে পারবে তার সীমা, সীমা পার হলে request reject হয় আর 429 Too Many Requests আর `Retry-After` header যায়। Throttling হলো request বাদ না দিয়ে ধীর করা বা queue তে রাখা। বহুল ব্যবহৃত algorithm হলো token bucket, যেখানে bucket এ token জমে আর প্রতি request এ একটা খরচ হয়, তাই burst সহ্য করে। এছাড়া fixed window, sliding window আর leaky bucket আছে। Express এ `express-rate-limit` ব্যবহার করি। বহু instance এ চললে Redis store লাগে, নইলে প্রতি server এ আলাদা count হয়। Proxy এর পেছনে `trust proxy` সেট করতে হয়, নইলে সবার IP এক দেখায়। Login এর মতো route এ কড়া limit আর সাধারণ API তে নরম রাখি।"

### 🟡 A13. API Key Authentication
> "এটা সাধারণত server-to-server বা third-party developer দের জন্য। Key তৈরি করি `crypto.randomBytes` দিয়ে, user কে একবারই দেখাই, আর DB তে SHA-256 hash করে রাখি, plain না। একটা prefix রাখি শনাক্ত করার জন্য। Client `x-api-key` header এ পাঠায়, middleware hash করে DB তে খোঁজে। প্রতিটা key এর scope বা permission, আলাদা rate limit, expiry আর revoke বা rotate করার ব্যবস্থা রাখি। সবসময় HTTPS, আর URL এ key রাখা যাবে না কারণ log এ চলে যায়। JWT আর API key এর পার্থক্য: API key দীর্ঘস্থায়ী আর application কে শনাক্ত করে, JWT ছোট আয়ুর আর user কে শনাক্ত করে।"

---

# 📦 PART 6: Scenario-Based (১ম অংশ)

> প্রশ্ন S1–S5 | 🔴 Must: S1, S5 | 🟡 Good: S2, S3, S4

> 💡 **Scenario এর কাঠামো:** আগে মাপো, তারপর কারণ, তারপর সমাধান, শেষে প্রতিরোধ।

---

### 🔴 S1. Heavy Traffic এ API Slow
> "আমি আগে আন্দাজে না গিয়ে মাপি। APM আর logs দেখে কোন endpoint ধীর বের করি, আর CPU, memory, event loop lag, DB connection, p95 response time দেখি। তারপর bottleneck খুঁজি: event loop block করা sync কাজ, ধীর database query, ধীর third-party API, memory leak, বা শুধু একটা instance এ সব চলা। সমাধানে Redis cache, DB index আর N+1 ঠিক করা, ভারী কাজ BullMQ queue তে পাঠানো, আর load balancer এর পেছনে horizontal scaling। সাথে rate limiting, compression, CDN আর pagination। প্রতিরোধে load test CI তে রাখা, alert আর capacity planning।"

### 🟡 S2. Deploy এর পর Random Logout
> "আমি সম্ভাব্য কারণ ধরে ধরে দেখব। সবচেয়ে সাধারণ কারণ JWT বা session secret বদলে যাওয়া, বা বিভিন্ন instance এ আলাদা থাকা। এরপর in-memory session store, যেটা restart এ সব হারায় আর একাধিক instance এ শেয়ার হয় না, তখন Redis লাগে। Cookie কনফিগারেশন বদলানো, যেমন domain, secure, sameSite। Proxy বা HTTPS এর পেছনে `trust proxy` না থাকলে secure cookie সেটই হয় না। Server দের সময় না মেলা, বা refresh token rotation এ একাধিক tab এর race condition। এমনকি CDN `Set-Cookie` সহ response cache করতে পারে। খুঁজতে logs এ 401 এর কারণ আর deploy এর আগে-পরের config diff দেখব। প্রতিরোধে shared session store আর পরিকল্পিত secret rotation।"

### 🟡 S3. বড় ফাইল আপলোডে Crash
> "কারণ সম্ভবত ফাইল পুরোটা memory তে বাফার হচ্ছে, যেমন multer এর memoryStorage, আর RAM শেষ হয়ে crash। সমাধান: streaming upload, মানে busboy বা multer এর disk storage। সবচেয়ে ভালো presigned URL দিয়ে client কে সরাসরি S3 তে আপলোড করানো। খুব বড় ফাইলে chunked বা resumable upload। সাথে file size limit, MIME validation, আর Nginx এর `client_max_body_size` ও timeout ঠিক করা। ভারী প্রসেসিং background queue তে পাঠাই।"

### 🟡 S4. Memory ক্রমাগত বাড়ছে
> "আগে নিশ্চিত হই এটা সত্যিই leak কিনা: memory graph শুধু উপরে উঠছে নাকি GC এ নামছে। তারপর `--inspect` দিয়ে আলাদা সময়ে দুটো heap snapshot নিয়ে compare করি কোন object জমছে। Clinic.js heapprofiler বা memlab ও ব্যবহার করি। সাধারণ কারণ: সীমাহীন cache, remove না করা listener, clear না করা timer, closure এ বড় ডেটা, বন্ধ না করা connection। সমাধান LRU cache, cleanup আর stream ব্যবহার। সাময়িক ব্যবস্থা হিসেবে memory limit এ auto-restart রাখা যায়, কিন্তু সেটা মূল সমাধান না।"

### 🔴 S5. লক্ষ User এর জন্য Scalable Authentication
> "আমি hybrid design করব। Access token হবে ছোট আয়ুর stateless JWT, তাই যেকোনো instance DB lookup ছাড়াই যাচাই করতে পারবে। Refresh token থাকবে DB বা Redis এ hash করে, rotation আর reuse detection সহ। Auth service হবে stateless, load balancer এর পেছনে auto-scale। Microservice থাকলে asymmetric key আর JWKS ব্যবহার করব, যাতে অন্য service শুধু public key দিয়ে যাচাই করে আর key rotation সহজ হয়। Password hashing CPU-ভারী, তাই লগইন spike এ সেটা bottleneck হতে পারে, আলাদা auth service আর rate limit দিয়ে সামলাব। Email এ unique index, read replica আর connection pooling। Google বা GitHub এর মতো OAuth দিয়ে password এর চাপ কমাব, MFA রাখব, আর verification email queue তে পাঠাব। Trade-off হলো JWT এ scale সহজ কিন্তু revoke কঠিন, তাই short-lived token আর server-side refresh token।"

---

# 📦 PART 7: Scenario-Based (২য় অংশ)

> প্রশ্ন S6–S10 | 🔴 Must: S6, S9, S10 | 🟡 Good: S7, S8

---

### 🔴 S6. DB Query ৫-১০ সেকেন্ড
> "আগে slow query log বা `pg_stat_statements` থেকে কোন query ধীর বের করি, তারপর `EXPLAIN ANALYZE` দিয়ে plan দেখি। `Seq Scan` মানে পুরো table scan হচ্ছে, index দরকার। তারপর optimize করি: WHERE, JOIN, ORDER BY এর column এ index, N+1 ঠিক করা, `SELECT *` এর বদলে শুধু দরকারি column, আর বড় offset এর বদলে cursor pagination। Connection pooling, বারবার লাগা ডেটা Redis এ cache, আর ভারী aggregate এর জন্য materialized view। পড়ার লোড বেশি হলে read replica। Long transaction এড়াই। Index দেওয়ার পর `EXPLAIN` এ `Index Scan` আসছে কিনা যাচাই করি, মেপে নিশ্চিত হই।"

### 🟡 S7. Public API এর Rate Limiting
> "আগে ঠিক করব কী দিয়ে চিনব: বেনামী হলে IP, আর API key বা user ID থাকলে সেটা। Tier অনুযায়ী আলাদা limit রাখব, যেমন free আর pro। Algorithm হিসেবে token bucket, যেটা burst সহ্য করে। বহু instance এ চলবে বলে count রাখব Redis এ, atomic operation দিয়ে। Endpoint ভেদে আলাদা limit, যেমন login কড়া আর সাধারণ GET নরম। সীমা পার হলে 429 আর `Retry-After` আর `RateLimit-*` header দেব। Proxy এর পেছনে `trust proxy` সেট করব। বড় স্কেলে app এ আসার আগেই Cloudflare, Nginx বা API gateway স্তরে আটকাব। সাথে monitoring আর documentation এ limit জানিয়ে রাখব।"

### 🟡 S8. Local এ চলে, Production এ Fail
> "আগে logs দেখি, সেটাই সবচেয়ে দ্রুত। তারপর সাধারণ কারণগুলো checklist ধরে দেখি: environment variable নেই বা ভুল, Node বা package version আলাদা। Linux এর case-sensitive filesystem, যেমন `./User` আর `./user` আলাদা। `devDependencies` production এ install না হওয়া, বা build step বাদ পড়া। Container এ `localhost` এ listen করলে বাইরে থেকে পাওয়া যায় না, `0.0.0.0` এ listen করতে হয়। Database এর firewall, SSL বা connection string। CORS, HTTPS আর cookie flag, আর memory limit এ OOM kill। Reproduce করতে Docker দিয়ে local এ production এর মতো পরিবেশ বানাই। প্রতিরোধে Docker, env validation আর staging environment।"

### 🔴 S9. Duplicate Transaction আটকানো
> "মূল সমাধান idempotency key। Client প্রতিটা payment এর ইচ্ছার জন্য একটা ইউনিক key পাঠায়, server সেই key DB তে unique constraint সহ রাখে। একই key আবার এলে নতুন করে না চালিয়ে আগের ফলাফলই ফেরত দেয়। দ্বিতীয় স্তর হলো DB constraint, যেমন booking এ `unique(roomId, date)`, যাতে race condition এও DB নিজে দ্বিতীয়টা আটকায়। তৃতীয় স্তর transaction আর row lock, `SELECT ... FOR UPDATE`, বা version column দিয়ে optimistic locking। এছাড়া payment কে state machine দিয়ে চালাই, `pending` থেকে `paid`, আর webhook handler কে idempotent রাখি, কারণ gateway একই event দুবার পাঠাতে পারে, তাই event id সংরক্ষণ করি। UI তে button disable করাও লাগে, কিন্তু সেটা যথেষ্ট না, backend সুরক্ষাই আসল। সাথে gateway এর নিজস্ব idempotency key আর নিয়মিত reconciliation।"

### 🔴 S10. ধীর ও অনির্ভরযোগ্য Third-Party API
> "মূল নীতি হলো বাইরের সিস্টেমের ব্যর্থতা যেন আমার সিস্টেম না ডোবায়। প্রথমে সবসময় timeout সেট করি, নইলে request ঝুলে থেকে resource শেষ করে। দ্বিতীয়ত retry করি exponential backoff আর jitter দিয়ে, কিন্তু শুধু idempotent কাজে। তৃতীয়ত circuit breaker, যেমন `opossum`, বারবার fail করলে কিছুক্ষণ call বন্ধ রেখে fail fast করে, যাতে তারা ডুবলে আমি না ডুবি। এরপর cache আর fallback দিয়ে graceful degradation, ডেটা না পেলে পুরনো বা ডিফল্ট ডেটা দেখাই। সম্ভব হলে কাজটা background queue তে পাঠাই, user কে সাথে সাথে response দিই আর ফলাফল webhook বা polling এ জানাই। আলাদা connection pool দিয়ে bulkhead, তাদের rate limit মানা, একটা adapter layer, আর latency ও error rate এর monitoring আর alert।"

---

# 💡 বাড়তি টিপস

- **আগে ছোট উত্তর, তারপর:** "চাইলে আরো বিস্তারিত বলতে পারি।"
- **Scenario প্রশ্নে এই ক্রম:** মাপো → কারণ খোঁজো → সমাধান → প্রতিরোধ। জোরে বলে চিন্তা করলে interviewer তোমার চিন্তার পদ্ধতি দেখতে পায়।
- **নিজের অভিজ্ঞতা জুড়ো (সবচেয়ে বেশি impression):**
  - "আমার প্রজেক্টে Prisma তে N+1 ঠিক করতে `include` ব্যবহার করেছি।"
  - "Auth এর জন্য access আর refresh token আলাদা রেখেছি, refresh token httpOnly cookie তে।"
  - "Input validation এ Zod, আর রোল অনুযায়ী middleware দিয়ে access control।"
- **না জানলে:** "এটা সরাসরি ব্যবহার করিনি, তবে যতটুকু জানি..."

শুভকামনা! 🚀
