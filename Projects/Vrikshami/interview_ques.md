# Vrikshami — Complete Interview Preparation Guide

> Read this file end-to-end before any interview.  
> Every answer is written in first-person so you can say it directly.

---

## PART 1 — Project Introduction & Elevator Pitch

### Q: Tell me about your project Vrikshami.

**Short version (30 sec):**
"Vrikshami is a web platform I built that lets users adopt trees through verified NGOs, make donations, and receive a digital certificate of appreciation. It also has a homegrown plant marketplace where users can buy and sell plants, a referral system with badge rewards, and an AI-powered chatbot for plant-related queries."

**Detailed version (1.5 min):**
"Vrikshami is a full-stack Node.js + Express application focused on environmental sustainability. The core feature is tree adoption — users browse NGOs, select how many trees to adopt, pay via Razorpay, and the system automatically generates a personalized PDF Certificate of Appreciation using pdfmake. Beyond adoption, I built a plant marketplace where logged-in users can list homegrown plants with photos (uploaded to Cloudinary), and other users can browse and buy them. There's also a referral system: users get a unique link, and whenever someone signs up through it, the referrer earns a 'Referral Bonus' badge. Finally, I integrated a Groq AI chatbot scoped only to plant and environment topics."

---

## PART 2 — Tech Stack Questions (Why did you choose this?)

### Q: Why did you use Node.js and Express.js?

"I chose Node.js because its non-blocking, event-driven I/O model is well-suited for an application with multiple async operations like database calls, API calls to Razorpay, Cloudinary, and Groq, and file I/O for certificate generation. Express.js is the most popular and minimal framework on top of Node.js that gives me full control over routing and middleware without forcing any structure. Since I was handling everything in a single app.js with custom route logic, Express was the right fit."

### Q: Why MongoDB instead of MySQL or PostgreSQL?

"I chose MongoDB because the data in Vrikshami has variable structure. For example, NGOs each have a different number of stats (some have 3, some have 5). MongoDB's document model handles this naturally without needing extra tables or null columns. Also, I needed to store arrays inside documents — like a user's adoptions array or an NGO's adoptionIds array — which is very natural in MongoDB. The flexible schema let me iterate quickly during development. If I were building a system requiring complex joins or transactions (like a banking system), I would use PostgreSQL, but for this use case MongoDB was appropriate."

### Q: Why EJS instead of React or Vue?

"I chose EJS because Vrikshami is a server-rendered application where most pages are relatively static HTML with a few dynamic values injected (NGO names, user data, etc.). EJS let me render complete HTML on the server and send it to the browser, which is simpler, faster for initial page load, and SEO-friendly. A React SPA would have been overkill here since most interactions don't require a rich client-side component tree. I used jQuery and AJAX for the parts that needed interactivity (like the Razorpay payment flow) without needing a full SPA framework."

### Q: Why Razorpay and not Stripe or PayU?

"Razorpay is the most widely used payment gateway in India and is specifically designed for INR transactions. It natively supports UPI, which is the most popular payment method in India. Stripe's Indian support is more limited, and Razorpay has better documentation, a clean Node.js SDK, and a good test mode. The checkout.js library gives a polished, ready-made payment popup so I didn't need to build a custom payment form."

### Q: Why pdfmake for certificate generation?

"pdfmake is a pure JavaScript library that runs on Node.js and generates PDFs programmatically using a JSON-based document definition. I didn't want to use a headless browser like Puppeteer just to generate a certificate — that's too heavy. pdfmake lets me define the entire PDF layout in JavaScript: fonts, colors, columns, alignment, background images. It generates a readable stream that I buffer and write to disk. It's lightweight, server-side, and keeps the certificate fully automated without any user intervention."

### Q: Why bcrypt for passwords?

"bcrypt is the industry standard for password hashing. It uses a salt to prevent rainbow table attacks, and the work factor (rounds) makes brute-force attacks computationally expensive. I used Mongoose's pre-save middleware to automatically hash the password before it ever touches the database, so the plain password is never stored anywhere. I used `bcrypt.compare()` during login to verify against the stored hash."

### Q: Why express-session for authentication instead of JWT?

"For this application, session-based auth was simpler and more appropriate. The user logs in, the server stores their `user_id` in the session object, and every subsequent request checks `req.session.user_id`. JWTs are better for stateless APIs where you need to authenticate across multiple servers without a shared session store. Since Vrikshami runs on a single server with server-rendered pages, express-session was the straightforward, correct choice."

### Q: Why Cloudinary for image uploads?

"Plant images need to be served from a CDN so that image loading is fast for users. Storing images directly on the server's filesystem would cause problems if the server restarts or if I scale horizontally. Cloudinary handles image storage, CDN delivery, and even transformations like resizing and format conversion (I configured multer-storage-cloudinary to convert all uploads to JPEG). The `multer-storage-cloudinary` package makes the integration very clean — the uploaded file path from `req.file.path` is automatically a Cloudinary CDN URL."

### Q: Why uuid for user IDs?

"MongoDB already generates `_id` (ObjectId) for documents, but I used UUIDs as a separate `user_id` field because I needed a user-facing referral identifier that is URL-safe and doesn't expose internal ObjectIds. UUID v4 generates a cryptographically random 36-character string like `550e8400-e29b-41d4-a716-446655440000`. I use this `user_id` in the referral link like `/?ref=<user_id>` so that users can share it without exposing database internals."

### Q: Why Groq SDK and the mixtral-8x7b-32768 model?

"Groq provides extremely fast LLM inference — it's significantly faster than OpenAI for the same model size. mixtral-8x7b-32768 is a powerful open-source model with a 32k context window. I wanted the chatbot to be fast and responsive. I also added a system prompt to scope it strictly to plant adoption, plants, environment, and conservation topics. And I added rate limiting (20 requests/minute per IP) using express-rate-limit to protect the API from abuse since Groq API calls have usage costs."

### Q: Why express-rate-limit?

"The `/chat` route calls the Groq API which has usage-based billing. Without rate limiting, a single user could spam the chatbot and generate large API costs. I applied rate limiting only to `/chat` (not the entire app) to keep it targeted. The limit is 20 requests per minute per IP, which is reasonable for real users but prevents abuse."

### Q: Why ejs-mate?

"EJS alone doesn't support layout inheritance — every EJS file has to include all its own HTML boilerplate. ejs-mate adds the concept of layouts and partials (like `extends boilerplate.ejs`), which lets all pages share a common base HTML structure. This eliminated repetition across all 13+ EJS templates and made it easy to add a consistent navbar and footer."

---

## PART 3 — Architecture & Design Questions

### Q: Explain your MVC architecture.

"I followed MVC where:
- **Models** are my Mongoose schemas — User, Ngo, BuySell, Seller, and Adoption — each in the `/models` folder. They define the data structure and handle database operations. The User model also has a bcrypt pre-save hook making it responsible for password security logic.
- **Views** are EJS templates in `/views`. They receive data from the controller as template variables and render HTML. There are two sub-folders: `/views/adoption` for NGO-related pages and `/views/listing` for user/marketplace pages. All use a shared `boilerplate.ejs` layout via ejs-mate.
- **Controllers** are Express route handlers in `app.js`. Each route function receives the HTTP request, interacts with models (database queries), processes business logic, and either renders an EJS view or sends a JSON response (for AJAX routes)."

### Q: Why is everything in one app.js instead of separate route files?

"Honestly, this was a trade-off I made during development to move quickly. Ideally, I should have separated routes into files like `routes/ngo.js`, `routes/user.js`, `routes/payment.js` and used Express Router. The single-file approach works for a project of this scale but would be difficult to maintain as it grows. In a production codebase I would refactor this immediately."

### Q: How does request flow work in your app?

"A request comes in from the browser. Express middleware runs first (session, body-parser, static files). Then the matching route handler executes. The handler calls Mongoose models to read or write MongoDB. For server-rendered pages, it calls `res.render('viewname', { data })` which ejs-mate compiles into HTML. For AJAX endpoints (Razorpay, certificate, chatbot), it calls `res.json({ ... })`. The response goes back to the browser."

### Q: How do you protect routes that require login?

"I check `req.session.user_id` at the beginning of protected routes. If it's not set (user is not logged in), I redirect to `/login`. For example, `/adoption`, `/addPlant`, `/profile`, and `/sell` all have this check. This is a simple but effective session-based guard. In a larger application, I'd extract this into a reusable middleware function called `isLoggedIn` rather than duplicating the check in every route."

---

## PART 4 — Feature Deep Dive Questions

### Q: Walk me through the complete Razorpay payment flow.

"There are 5 steps:

1. **Frontend**: On the NGO detail page, the user opens a Bootstrap modal, selects number of trees, and jQuery calculates `price × quantity = total` live. On clicking 'Proceed to Payment', a jQuery AJAX POST goes to `/razorpay/create-order` with the total amount in rupees.

2. **Backend - Create Order**: The Express route validates the amount, multiplies by 100 to convert to paise (Razorpay's smallest unit), then calls `razorpayInstance.orders.create()` using the Razorpay Node SDK. The SDK returns an `orderId` which we send back as JSON.

3. **Frontend - Checkout**: With the `orderId`, we initialize Razorpay Checkout.js and call `razorpay.open()`. The Razorpay-hosted popup handles the actual payment via UPI, card, etc.

4. **Success Handler**: On successful payment, Razorpay calls our `handler` function with the `razorpay_payment_id`. We then call `/generate-certificate` via AJAX to generate the PDF certificate.

5. **Completion**: After getting the certificate URL, we show a success message with a download link inside the modal. The user clicks 'Done', which POSTs adoption details (userId, paymentId, amount, date) to `/ngo/:id`, which saves everything to MongoDB and redirects to `/adoption`."

### Q: How does the certificate generation work?

"It's fully automated on the backend using a separate module `certificate-generator.js`. When called with `(userName, ngoName)`:
1. A `docDefinition` object is created that defines the entire PDF layout — page size (LETTER), margins, a background image with 0.2 opacity, and all text elements with styling (colors, fonts, sizes, alignment).
2. A pdfmake printer instance is created using custom Roboto TTF fonts from `/public/fonts`.
3. `printer.createPdfKitDocument(docDefinition)` generates a readable PDF stream.
4. We collect stream chunks into an array, and on the 'end' event we concatenate them into a Buffer using `Buffer.concat()`.
5. The buffer is written to `/public/certificates/certificate-<Date.now()>.pdf` using `fs.writeFileSync()`.
6. We return the public URL `/certificates/certificate-<timestamp>.pdf`.

The timestamp in the filename ensures uniqueness."

### Q: How does the referral system work?

"Users visit `/referal` where their `user_id` is rendered from `req.session.user_id`. jQuery constructs a shareable link like `http://localhost:3000/?ref=<userId>`. When a new user clicks that link, the homepage route reads `req.query.ref` and stores it in `req.session.referral`. When the new user submits the signup form, the POST `/signup` handler creates their account, then checks if `req.session.referral` is set. If it is, it finds the referrer by their `user_id`, pushes 'Referral Bonus' into their `badges` array, saves the referrer document, and clears `req.session.referral`."

### Q: What is a limitation of your referral system?

"The referral code is stored in the session. If the user opens the referral link in a different browser, a different device, or after the session expires, the referral won't be tracked. A better approach would be to pass the `referralCode` as a hidden input field in the signup form and send it in the POST body, so it's explicitly attached to the signup request regardless of session state. Even better, store the referral code in the new user's document so it's always auditable."

### Q: How does the plant marketplace work?

"A seller (any logged-in user) goes to `/addPlant`, fills in plant name, description, price, category, care level, height, lifespan, quantity, address, and contact. They also upload a plant image. Multer with CloudinaryStorage intercepts the file upload — it streams the image directly to Cloudinary and returns a CDN URL. A new BuySell document is created with all details including the Cloudinary URL as `plantImg` and the seller's `user_id`.

On the buyer side, `/sell` displays all listings. A buyer visits `/plantDescription/:id` for details and posts to `/buy/:id`. The buy handler reduces the plant's quantity in MongoDB. If quantity reaches 0, the listing is deleted. A Seller record tracks all buyers. The plant's `_id` is pushed into the buyer's `plants` array in the User document."

### Q: How does the AI chatbot work?

"When the user sends a message, the frontend POSTs to `/chat`. The route is protected by rate limiting. The handler sends the message to the Groq API using the `groq-sdk`, passing a system prompt that restricts the model to only answer questions about plant adoption, plants, environment, and conservation. The model used is `mixtral-8x7b-32768`. The response text is extracted and sent back as JSON. The frontend displays it in the chat widget."

---

## PART 5 — MongoDB & Database Questions

### Q: How did you design your MongoDB schemas?

"I designed schemas around the main entities: User, NGO, Adoption, BuySell (plant listing), and Seller. The key design decisions were:

- **References vs Embedding**: I used references (ObjectIds) for relationships that grow over time, like a user's `adoptions` array and `plants` array. These are populated when needed using Mongoose's `.populate()`. For small, fixed sub-documents like NGO stats (label/value pairs) or plant specifications, I used embedded sub-documents.
- **Dual ID system on User**: User has MongoDB's `_id` (ObjectId) for database operations and a custom `user_id` (UUID string) for application-level referencing like referral links and cross-referencing plant sellers.
- **NGO tracks adoptions**: The NGO document has an `adoptionIds` array of Adoption ObjectIds. This lets me quickly count adoptions per NGO and display stats without a separate query."

### Q: What is Mongoose populate() and why do you use it?

"`.populate()` replaces an ObjectId reference in a document with the actual referenced document from its collection. For example, when I load a user's profile, I call `User.findOne({ user_id }).populate('adoptions').populate('plants')`. Without populate, `user.adoptions` would be an array of ObjectIds. With populate, it's an array of full Adoption objects, so I can access `adoption.adoptionPrice`, `adoption.adoptionDate` directly in my EJS template or route handler."

### Q: Why not use SQL for this project?

"MongoDB was appropriate here for a few reasons:
- The NGO stats field is an array of `{label, value}` objects — different NGOs have different numbers of stats. In SQL this would require a separate `ngo_stats` table with foreign keys. In MongoDB it's just an embedded array.
- The badges field on User is a flexible string array that might grow.
- I was building a prototype quickly, so schema flexibility helped.

Where SQL would be better: if I needed complex reporting queries (JOIN adoptions with users with NGOs), enforced foreign key constraints, or ACID transactions (like ensuring a payment deduction and record creation happen atomically). MongoDB 4+ supports multi-document transactions, but it's not used here."

### Q: How did you handle password security?

"Passwords are hashed using bcrypt before being stored. I added a Mongoose pre-save hook on the User schema:
```javascript
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});
```
The `isModified` check ensures we don't re-hash an already hashed password on subsequent saves. During login, I use `bcrypt.compare(enteredPassword, storedHash)` which handles the salt automatically."

### Q: How are indexes used in your schemas?

"Unique constraints act as unique indexes in MongoDB. I have `unique: true` on `User.username`, `User.email`, `User.user_id`, `Ngo.ngoId`, and `Seller.seller_id`. These prevent duplicate registrations and duplicate NGOs, and also make queries on those fields faster."

### Q: What would you improve about your database design?

"A few things:
1. The Adoption model is referenced in app.js but I never created a separate `adoptionschema.js` file — it's created inline. I'd clean that up.
2. I'd add a proper `Payment` collection to track payment states (pending, completed, failed) separately from the Adoption record — right now I'm trusting the client-side success handler which isn't verified server-side.
3. For the referral system, I'd store the `referredBy` field directly on the new User document instead of relying on session state.
4. I'd add timestamps to all schemas to track creation and update times."

---

## PART 6 — Frontend Questions

### Q: How did you handle frontend validation?

"I did both client-side and server-side validation:

**Client-side (EJS + jQuery):** In the donation modal, before calling Razorpay I check if `userName` and `userEmail` are non-empty. If validation fails, I add Bootstrap's `is-invalid` class to the input and prevent the AJAX call. For the `addPlant` form, I validate in the Express route handler using explicit if-checks.

**Server-side (Express):** For `/addPlant`, I validate that required fields exist, that price is a positive number, and that contact is exactly 10 digits using a regex. For `/generate-certificate`, I check that both `userName` and `ngoName` are provided. For `/razorpay/create-order`, I check that amount is present."

### Q: What is EJS and how does it work?

"EJS (Embedded JavaScript) is a templating engine that lets you embed JavaScript inside HTML using `<% %>` tags. `<%= %>` outputs an escaped value, `<% %>` executes code without output, and `<%- %>` outputs unescaped HTML. When Express calls `res.render('view', { data })`, the EJS engine processes the template file, executes the embedded JavaScript with the passed data as local variables, and returns a complete HTML string to the browser."

### Q: How did you use jQuery in the project?

"I used jQuery mainly for:
1. **Live total calculation** in the donation modal: listening to the `change` event on the tree quantity input and calculating `price × quantity` dynamically.
2. **AJAX calls**: `$.post('/razorpay/create-order', {...}, callback)` and `$.post('/generate-certificate', {...}, callback)` to call backend APIs without page reload.
3. **DOM manipulation**: showing/hiding success messages, updating the modal content after payment succeeds, adding the certificate download link dynamically.
4. **Referral link generation**: constructing the referral URL from the userId rendered in the page and setting it in a text input."

### Q: How does Bootstrap help in your project?

"Bootstrap provides:
- **Responsive grid system**: the layout adjusts for mobile and desktop automatically using `col-md-4` etc.
- **Modals**: the donation modal on the NGO detail page is a Bootstrap modal component. I trigger it with `$('#donationModal').modal('show')`.
- **Form styling**: `form-control`, `is-invalid`, `btn` classes give consistent form styling across all pages.
- **Navbar**: responsive collapsible navbar for mobile.
- **Cards and badges**: NGO cards on the listing page and user badges on the profile page use Bootstrap card and badge components."

### Q: How did you handle image uploads on the frontend?

"The addPlant form has a file input (`<input type='file' name='plantImage'>`). On form submission, the browser encodes the file as `multipart/form-data`. On the backend, Multer middleware intercepts the request, reads the file buffer, and the CloudinaryStorage adapter streams it directly to Cloudinary. After upload, `req.file.path` contains the Cloudinary CDN URL, which I store in MongoDB as the `plantImg` field."

### Q: How does the Razorpay popup work on the frontend?

"Razorpay provides a JavaScript file `checkout.js` loaded from their CDN. Once I have an `orderId` from my backend, I create a `Razorpay(options)` object with keys like the orderId, amount, currency, business name, user's prefilled name/email, and a `handler` callback function. Calling `razorpay.open()` displays the Razorpay-hosted payment popup. All payment processing happens inside Razorpay's secure iframe — my app never touches the card details. On successful payment, Razorpay calls the `handler` with the `razorpay_payment_id`."

---

## PART 7 — Security Questions

### Q: Are there any security vulnerabilities in your project?

"Yes — I know of several and can discuss them honestly:

1. **API keys in code**: The Razorpay `key_id` and `key_secret` are hardcoded in `app.js`. These should be in `.env` and loaded with `dotenv`. The Cloudinary and Groq keys are correctly in `.env`, but Razorpay's are exposed in source code.

2. **Client-side Razorpay key exposed**: The Razorpay `key_id` in the frontend JavaScript is visible to the user. This is technically acceptable for the publishable key, but the `key_secret` must never appear in frontend code — which it doesn't.

3. **No payment signature verification**: After Razorpay payment succeeds, I trust the client-side `payment_id` without verifying the Razorpay signature on the server. Razorpay provides a signature (`razorpay_signature`) that should be verified using HMAC-SHA256 before saving the adoption record. This is a real security gap.

4. **No CSRF protection**: Forms don't have CSRF tokens, making them vulnerable to cross-site request forgery attacks.

5. **No input sanitization for XSS**: User-provided strings (plantName, userName, etc.) are stored and rendered in EJS using `<%= %>` which HTML-escapes output (safe), but I haven't used a dedicated sanitization library."

### Q: How would you improve the payment security?

"I'd add Razorpay signature verification on the server side. After the client sends `razorpay_payment_id`, `razorpay_order_id`, and `razorpay_signature`, I'd verify them like this:
```javascript
const crypto = require('crypto');
const body = razorpay_order_id + '|' + razorpay_payment_id;
const expectedSignature = crypto
  .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET)
  .update(body)
  .digest('hex');
if (expectedSignature === razorpay_signature) { /* proceed */ }
```
This ensures the payment response came from Razorpay and wasn't tampered with."

### Q: How does session management work?

"express-session stores session data server-side and gives the browser a session cookie (a random session ID). When a user logs in, I set `req.session.user_id = user.user_id`. On every subsequent request, express-session reads the cookie, looks up the session on the server, and attaches the data to `req.session`. I check `req.session.user_id` for protected routes. `cookie: { secure: false }` means the cookie works over HTTP. In production this should be `true` and the app should run over HTTPS."

---

## PART 8 — Node.js & Express Internals

### Q: What is middleware in Express.js?

"Middleware is a function with signature `(req, res, next)` that executes in sequence for every incoming request. I use several middleware: `express.static` serves static files, `express.urlencoded` parses form data, `express.json` parses JSON bodies, `express-session` attaches session to `req`, `express-rate-limit` blocks excessive requests. Each middleware calls `next()` to pass control to the next one. If it doesn't call `next()` and doesn't send a response, the request hangs."

### Q: What is async/await and how do you use it?

"async/await is syntactic sugar over Promises. An `async` function always returns a Promise. Inside it, `await` pauses execution until the awaited Promise resolves. I use it throughout app.js for all database operations: `await Ngo.find()`, `await User.findOne(...)`, `await newAdoption.save()`. This makes async code read like synchronous code and avoids callback hell. I wrap everything in `try/catch` to handle errors — if any awaited operation throws, the catch block handles it gracefully."

### Q: What is the event loop in Node.js?

"Node.js is single-threaded but handles concurrency via the event loop. When you call an async operation like a database query, Node.js offloads it to libuv (the underlying library) and registers a callback. The event loop continues processing other requests. When the database responds, the callback is queued and executed on the next iteration of the event loop. This means one Node.js process can handle thousands of concurrent connections because it never blocks waiting for I/O."

### Q: What is method-override and why do you use it?

"HTML forms only support GET and POST methods. For RESTful routes requiring PUT or DELETE, `method-override` reads a `_method` query parameter (like `?_method=DELETE`) and sets `req.method` accordingly. This lets EJS forms simulate PUT and DELETE methods even though the browser only sends a POST."

---

## PART 9 — Scalability & Improvement Questions

### Q: How would you make this production-ready?

"Several improvements:
1. **Separate route files** using Express Router — user routes, NGO routes, payment routes, etc.
2. **Move to MongoDB Atlas** instead of `localhost:27017` for a managed, replicated cloud database.
3. **Add Razorpay webhook** for server-side payment verification instead of trusting the client-side handler.
4. **Environment variables** for all secrets including the Razorpay keys.
5. **HTTPS** with `cookie: { secure: true }`.
6. **Session store**: currently sessions are in-memory and lost on server restart. Use `connect-mongo` to persist sessions in MongoDB.
7. **Error pages**: currently some routes return plain `res.status(500).send('Error')`. I'd render proper error pages.
8. **Input validation library**: use `joi` or `express-validator` instead of manual if-checks.
9. **Authentication middleware**: extract the login check into a reusable `isAuthenticated` middleware."

### Q: What was the most challenging part of building this?

"The most challenging part was the async coordination in the payment flow. After Razorpay payment succeeds, I needed to: (1) generate the certificate via AJAX, (2) show the success message with the certificate link, (3) then on 'Done' save the adoption to the database. Each step depends on the previous one, and if certificate generation fails after payment succeeds, the user paid but got no certificate. I handled this by nesting the AJAX callbacks carefully and showing clear error messages at each step. Ideally, I'd use a queue or retry mechanism for the certificate generation."

### Q: What is a race condition and could it happen in your plant marketplace?

"A race condition is when two concurrent operations produce an incorrect result because they both read a shared value, decide based on it, and then both write. In my plant marketplace, if two buyers simultaneously buy the last item, both could read `quantity = 1`, both see it's available, and both complete the purchase — resulting in `quantity = -1`. I use `plants.quantity -= quantity; await plants.save()` which has this vulnerability. The correct fix is to use MongoDB's atomic operators like `findOneAndUpdate({ _id, quantity: { $gte: 1 } }, { $inc: { quantity: -1 } })` which makes the decrement atomic."

---

## PART 10 — Quick-Fire Questions

**Q: What is Mongoose?**  
"Mongoose is an ODM (Object Document Mapper) for MongoDB and Node.js. It provides schema definitions, data validation, type casting, query building, and middleware hooks (like pre-save) on top of MongoDB's native driver."

**Q: What is a Mongoose schema vs a model?**  
"A schema defines the structure, types, validations, and middleware for documents. A model is a class created from a schema that you use to interact with a MongoDB collection — query, create, update, delete documents."

**Q: What is `.populate()` in Mongoose?**  
"It replaces an ObjectId reference field with the actual referenced document, fetched from its collection in a second database query."

**Q: What is the difference between `res.render()` and `res.json()`?**  
"`res.render('view', data)` compiles an EJS template with the passed data and sends HTML back. `res.json(data)` sends a JSON-encoded response and sets `Content-Type: application/json`. I use render for page loads and json for AJAX API responses."

**Q: What is paise?**  
"Paise is the smallest unit of INR — 1 rupee = 100 paise. Razorpay and most payment gateways require amounts in the smallest currency unit to avoid floating point issues. So ₹250 becomes 25000 paise: `amount * 100`."

**Q: What is a pre-save hook in Mongoose?**  
"A middleware function that runs before a document is saved to the database. In User schema, I use it to hash the password automatically before every save."

**Q: What does `{ recursive: true }` do in `fs.mkdirSync()`?**  
"It creates the directory and all parent directories that don't exist yet, without throwing an error if the directory already exists. Without `recursive: true`, it would throw if the parent folder didn't exist."

**Q: Why do you use `Date.now()` in the certificate filename?**  
"It generates a Unix timestamp in milliseconds that is unique to that moment. Using `Date.now()` guarantees the filename `certificate-1739965485324.pdf` won't collide with other certificates unless two are generated in the exact same millisecond (extremely unlikely in this app)."

**Q: What is the purpose of `method-override`?**  
"HTML forms can only send GET and POST. method-override intercepts a POST with `?_method=DELETE` or `?_method=PUT` and changes `req.method` so REST routes work correctly."

**Q: How does dotenv work?**  
"`require('dotenv').config()` reads the `.env` file in the project root and loads each key-value pair as a `process.env` variable. This keeps secrets out of source code."

---

## PART 11 — Behavioral Questions About the Project

**Q: What would you build next if you had more time?**  
"I'd add: (1) Razorpay signature verification for proper payment security, (2) a dashboard showing adoption statistics per NGO, (3) email notifications after adoption using Nodemailer, (4) persistent sessions with connect-mongo, (5) admin panel to manage NGOs."

**Q: If 10,000 users signed up, what would break first?**  
"In-memory sessions would be the first problem — they're lost on restart and eat memory. I'd move to MongoDB-backed sessions with `connect-mongo`. The certificate `/public/certificates` folder would also grow indefinitely — I'd move generated PDFs to Cloudinary or S3 with a lifecycle policy."

**Q: How did you test this application?**  
"I tested manually using Razorpay test mode with test cards (like `4111 1111 1111 1111`). For the database, I used MongoDB Compass to verify documents. In a production setting, I'd write unit tests with Jest for the certificate generator module and integration tests for the routes using Supertest."
