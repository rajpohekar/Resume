# Vrikshami — Project Architecture

---

## 1. High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CLIENT (Browser)                               │
│                                                                         │
│   EJS Templates  +  Bootstrap CSS  +  jQuery  +  Razorpay Checkout.js  │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │  HTTP Requests (GET / POST)
                            │  AJAX calls via jQuery $.post / $.get
                            ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     SERVER  (Node.js + Express.js)                      │
│                                                                         │
│   app.js  ── single-file entry point with all routes (controllers)      │
│                                                                         │
│   Middleware Stack:                                                      │
│   ┌──────────────┐  ┌──────────────────┐  ┌───────────────────────┐    │
│   │express-static│  │express-session   │  │express-rate-limit     │    │
│   │/public folder│  │(auth + referral) │  │(for /chat route)      │    │
│   └──────────────┘  └──────────────────┘  └───────────────────────┘    │
│   ┌──────────────┐  ┌──────────────────┐  ┌───────────────────────┐    │
│   │body-parser   │  │method-override   │  │multer (file uploads)  │    │
│   │(urlencoded + │  │(_method param)   │  │→ CloudinaryStorage    │    │
│   │ json)        │  └──────────────────┘  └───────────────────────┘    │
│   └──────────────┘                                                      │
│                                                                         │
│   View Engine:  ejs-mate  (layouts + partials in /views/layouts)        │
└──────────┬───────────────────────────────────┬──────────────────────────┘
           │ mongoose.connect()                │ External API calls
           ▼                                   ▼
┌──────────────────────┐       ┌───────────────────────────────────────────┐
│   MongoDB (local)    │       │           External Services                │
│   db: vrikshami      │       │                                           │
│                      │       │  ┌──────────────────────────────────────┐ │
│  Collections:        │       │  │  Razorpay API                        │ │
│  ┌────────────────┐  │       │  │  razorpayInstance.orders.create()    │ │
│  │ users          │  │       │  │  Checkout.js (frontend popup)        │ │
│  ├────────────────┤  │       │  └──────────────────────────────────────┘ │
│  │ ngos           │  │       │  ┌──────────────────────────────────────┐ │
│  ├────────────────┤  │       │  │  Cloudinary                          │ │
│  │ adoptions      │  │       │  │  multer-storage-cloudinary           │ │
│  ├────────────────┤  │       │  │  (plant image uploads → CDN)         │ │
│  │ buysells       │  │       │  └──────────────────────────────────────┘ │
│  ├────────────────┤  │       │  ┌──────────────────────────────────────┐ │
│  │ sellers        │  │       │  │  Groq AI  (mixtral-8x7b-32768)       │ │
│  └────────────────┘  │       │  │  /chat endpoint  (AI chatbot)        │ │
└──────────────────────┘       │  └──────────────────────────────────────┘ │
                               │  ┌──────────────────────────────────────┐ │
                               │  │  pdfmake  (local Node library)       │ │
                               │  │  certificate-generator.js            │ │
                               │  │  → /public/certificates/*.pdf        │ │
                               │  └──────────────────────────────────────┘ │
                               └───────────────────────────────────────────┘
```

---

## 2. MVC Architecture Breakdown

```
┌────────────────────────────────────────────────────────────────────────┐
│                         MVC in Vrikshami                               │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  VIEW  (V)  — /views/                                            │  │
│  │                                                                  │  │
│  │  /views/layouts/boilerplate.ejs  ← base HTML layout (ejs-mate)  │  │
│  │  /views/adoption/                                                │  │
│  │    index.ejs        → Homepage (NGO list, stats, plant preview)  │  │
│  │    adoption.ejs     → NGO browsing page                          │  │
│  │    ngo-detail.ejs   → NGO detail + Razorpay donation modal       │  │
│  │    about.ejs        → About page                                 │  │
│  │    article.ejs      → Articles / blog                            │  │
│  │  /views/listing/                                                 │  │
│  │    signup.ejs       → Registration form                          │  │
│  │    login.ejs        → Login form                                 │  │
│  │    sell.ejs         → Plant marketplace listing                  │  │
│  │    addPlant.ejs     → Add new plant form                         │  │
│  │    plantDescription.ejs → Single plant detail                    │  │
│  │    profile.ejs      → User profile + transactions + badges       │  │
│  │    referal.ejs      → Referral link generator                    │  │
│  │    confirmation.ejs → Purchase confirmation page                 │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  CONTROLLER  (C)  — Express Routes in app.js                     │  │
│  │                                                                  │  │
│  │  GET  /                 → Homepage (NGO list + stats)            │  │
│  │  GET  /adoption         → NGO browse (auth required)             │  │
│  │  GET  /ngo/:id          → NGO detail page (auth required)        │  │
│  │  POST /ngo/:id          → Save adoption after payment            │  │
│  │  GET  /signup           → Signup form                            │  │
│  │  POST /signup           → Create user + referral badge logic     │  │
│  │  GET  /login            → Login form                             │  │
│  │  POST /login            → Authenticate + set session             │  │
│  │  GET  /sell             → Marketplace listing (auth required)    │  │
│  │  GET  /addPlant         → Add plant form (auth required)         │  │
│  │  POST /addPlant         → Upload plant + Cloudinary              │  │
│  │  GET  /plantDescription/:id → Single plant page                  │  │
│  │  POST /buy/:id          → Buy plant, update stock                │  │
│  │  GET  /confirmation     → Purchase confirmation                  │  │
│  │  GET  /profile          → User profile + transaction history     │  │
│  │  GET  /referal          → Referral page (shows user's ref link)  │  │
│  │  GET  /article          → Articles page                          │  │
│  │  GET  /about            → About page                             │  │
│  │  POST /razorpay/create-order → Create Razorpay order (AJAX)     │  │
│  │  POST /generate-certificate  → Generate PDF cert (AJAX)         │  │
│  │  POST /chat             → Groq AI chatbot (rate-limited AJAX)    │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  MODEL  (M)  — /models/ + /certificate-generator.js             │  │
│  │                                                                  │  │
│  │  user.js          → User schema (username, email, password,      │  │
│  │                      user_id, badges[], adoptions[], plants[])   │  │
│  │                      pre-save hook: bcrypt password hashing      │  │
│  │  ngoschema.js     → NGO schema (ngoId, name, logo, description,  │  │
│  │                      adoptionPrice, adoptionIds[], stats[],      │  │
│  │                      trustLink)                                  │  │
│  │  buy-sell.js      → Plant listing schema (plantName, price,      │  │
│  │                      plantImg[Cloudinary URL], specs, quantity,  │  │
│  │                      seller{seller_id, address, contact},        │  │
│  │                      user_id)                                    │  │
│  │  seller.js        → Seller transaction schema                    │  │
│  │                      (seller_id, buyers[{user_id,amount,qty}])   │  │
│  │  adoptionschema   → (referenced inline / via Adoption model)     │  │
│  │                      (adoptionId, transactionId, adoptionDate,   │  │
│  │                       adoptionPrice, ngoId)                      │  │
│  │                                                                  │  │
│  │  certificate-generator.js  → Service module (not a DB model)     │  │
│  │     generatePdfCertificate(userName, ngoName)                    │  │
│  │     → pdfmake → Buffer → /public/certificates/cert-<ts>.pdf     │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Complete User Flow Diagrams

### 3A. Tree Adoption + Payment + Certificate Flow

```
User Opens NGO Detail Page (/ngo/:id)
          │
          ▼
  Clicks "Donate & Adopt" button
          │
          ▼
  Bootstrap Modal opens
  ┌─────────────────────────────────┐
  │  Select no. of trees            │
  │  jQuery updateTotal() fires     │  ← price × qty = total (live)
  │  Enter Name & Email             │
  │  Click "Proceed to Payment"     │
  └────────────────┬────────────────┘
                   │ Frontend validation (name + email check)
                   │
                   ▼
  $.post('/razorpay/create-order', { amount })
                   │
                   ▼
  ┌─────────────────────────────────────────┐
  │  Express Route: POST /razorpay/create-order │
  │  1. Validate amount exists              │
  │  2. amount * 100  (₹ → paise)           │
  │  3. razorpayInstance.orders.create()    │
  │  4. res.json({ orderId })               │
  └─────────────────────┬───────────────────┘
                        │ orderId returned
                        ▼
  Razorpay Checkout popup opens (Checkout.js)
  User pays via UPI / Card / NetBanking
                        │
                        ▼  (on payment success)
  handler(response) fires  →  razorpay_payment_id received
                        │
                        ▼
  $.post('/generate-certificate', { userName, ngoName })
                        │
                        ▼
  ┌───────────────────────────────────────────────────┐
  │  Express Route: POST /generate-certificate        │
  │  1. Validate userName + ngoName                   │
  │  2. certificateGenerator.generatePdfCertificate() │
  │     a. Build pdfmake docDefinition (styles, text) │
  │     b. printer.createPdfKitDocument()             │
  │     c. Stream → Buffer → writeFileSync()          │
  │     d. Save → /public/certificates/cert-<ts>.pdf  │
  │     e. Return  /certificates/cert-<ts>.pdf URL    │
  │  3. res.json({ certificateUrl })                  │
  └──────────────────────┬────────────────────────────┘
                         │ certificateUrl returned
                         ▼
  Frontend: Show success message in modal
            Show "Download Certificate" link
            User clicks "Done"
                         │
                         ▼
  $.post('/ngo/:id', { userName, userEmail, paymentId, totalAmount, date })
                         │
                         ▼
  ┌──────────────────────────────────────────────────────┐
  │  Express Route: POST /ngo/:id                        │
  │  1. Find NGO by ID                                   │
  │  2. Create new Adoption document (UUID, paymentId,   │
  │     date, price, ngoId)                              │
  │  3. Push adoption._id → ngo.adoptionIds → ngo.save() │
  │  4. Push adoption._id → user.adoptions → user.save() │
  │  5. res.redirect('/adoption')                        │
  └──────────────────────────────────────────────────────┘
```

### 3B. Plant Marketplace Flow

```
Seller (Logged-in User)                    Buyer (Logged-in User)
        │                                           │
        ▼                                           ▼
  GET /addPlant                             GET /sell
        │                                    (shows all BuySell listings)
  POST /addPlant                                    │
  multer → Cloudinary upload                 GET /plantDescription/:id
  BuySell.create({...})                             │
        │                                    POST /buy/:id
        │                                    1. Find plant by ID
        │                                    2. Find/create Seller record
        │                                    3. Push buyer info to seller.buyers
        │                                    4. plant.quantity -= qty
        │                                    5. Delete if qty <= 0
        │                                    6. user.plants.push(plant._id)
        │                                    7. redirect → /confirmation
        ▼                                           ▼
  Plant visible on /sell              GET /confirmation?seller_name&contact&...
```

### 3C. Referral System Flow

```
Existing User (Referrer)               New User (Referee)
       │                                       │
  GET /referal                          Clicks referral link:
  (shows userId in page)                /?ref=<user_id>
  jQuery generates link:                       │
  /?ref=<userId>                        GET /   (homepage)
       │                                req.session.referral = <user_id>
  Shares via WhatsApp/Email                    │
                                        GET /signup  →  fills form
                                               │
                                        POST /signup
                                        1. User.create({username, email, password, user_id})
                                        2. Check req.session.referral
                                        3. User.findOne({ user_id: referralCode })
                                        4. referrer.badges.push("Referral Bonus")
                                        5. referrer.save()
                                        6. req.session.referral = null
                                        7. redirect → /login
```

### 3D. AI Chatbot Flow

```
User types in chat widget (frontend)
        │
        ▼
POST /chat  { message: "..." }
  ├── rate limiter: 20 req/min per IP
  │
  ▼
Groq SDK → mixtral-8x7b-32768
  System prompt: "Answer ONLY about plant adoption, 
                  plants, environment, conservation"
        │
        ▼
res.json({ reply })  →  shown in chat widget
```

---

## 4. MongoDB Schema Relationships

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Schema Relationships                          │
│                                                                      │
│  User ──────────────────────────────────────────────────────────┐   │
│  { _id, username, email, password(hashed), user_id(uuid),       │   │
│    badges[], adoptions: [ObjectId→Adoption],                    │   │
│    plants: [ObjectId→BuySell], timestamps }                     │   │
│         │                          │                            │   │
│         │ .populate('adoptions')   │ .populate('plants')        │   │
│         ▼                          ▼                            │   │
│  Adoption                    BuySell (Plant Listing)            │   │
│  { _id, adoptionId(uuid),    { _id, plantName, plantPrice,      │   │
│    transactionId(rzp),         plantImg(Cloudinary URL),        │   │
│    adoptionDate, adoptionPrice,  plantSpecifications{category,  │   │
│    ngoId(ObjectId→Ngo) }        careLevel, height, lifeSpan},   │   │
│         │                        quantity, seller{seller_id,    │   │
│         │                        address, contact}, user_id }   │   │
│         │                                │                      │   │
│         ▼                                ▼                      │   │
│  Ngo                           Seller                           │   │
│  { _id, ngoId, name, logo,     { _id, seller_id,               │   │
│    description, adoptionPrice,   buyers[{user_id, amount,      │   │
│    adoptionIds: [ObjectId],      quantity}] }                   │   │
│    stats[], trustLink }                                         │   │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 5. File/Folder Structure

```
Vrikshami/
├── app.js                        ← Entry point: all routes + middleware
├── certificate-generator.js      ← pdfmake service module
├── package.json
├── .env                          ← Cloudinary + Groq keys
│
├── models/
│   ├── user.js                   ← User schema (bcrypt pre-save hook)
│   ├── ngoschema.js              ← NGO schema
│   ├── buy-sell.js               ← Plant marketplace schema
│   └── seller.js                 ← Seller transaction tracking
│
├── data/
│   └── ngo-data.js               ← Seed data for NGOs
│
├── init/                         ← DB seeding scripts
│
├── views/
│   ├── layouts/
│   │   └── boilerplate.ejs       ← Base HTML layout (ejs-mate)
│   ├── includes/                 ← Partials (navbar, footer)
│   ├── adoption/
│   │   ├── index.ejs             ← Homepage
│   │   ├── adoption.ejs          ← NGO list
│   │   ├── ngo-detail.ejs        ← NGO detail + payment modal
│   │   ├── about.ejs
│   │   └── article.ejs
│   └── listing/
│       ├── signup.ejs
│       ├── login.ejs
│       ├── sell.ejs              ← Marketplace
│       ├── addPlant.ejs
│       ├── plantDescription.ejs
│       ├── profile.ejs
│       ├── referal.ejs
│       └── confirmation.ejs
│
└── public/
    ├── css/                      ← Page-specific CSS files
    ├── js/
    │   └── articles.js           ← Static articles data
    ├── images/                   ← NGO logos, homepage assets
    ├── fonts/                    ← Roboto TTF (used by pdfmake)
    └── certificates/             ← Generated PDF certificates
        └── certificate-<ts>.pdf
```

---

## 6. Complete Tech Stack Summary

| Layer | Technology | Purpose |
|---|---|---|
| Runtime | Node.js | JavaScript server-side runtime |
| Framework | Express.js | HTTP routing and middleware |
| Templating | EJS + ejs-mate | Server-side HTML rendering with layouts |
| Database | MongoDB (local) | NoSQL document storage |
| ODM | Mongoose | Schema, validation, relationships |
| Auth | express-session | Session-based login state |
| Password | bcrypt | Secure password hashing (salt rounds=10) |
| Payment | Razorpay Node SDK + Checkout.js | Order creation + frontend payment popup |
| PDF | pdfmake | Dynamic certificate generation |
| Image Upload | Multer + multer-storage-cloudinary | Plant photo upload to Cloudinary CDN |
| AI Chatbot | Groq SDK (mixtral-8x7b-32768) | Fast LLM for plant/env Q&A |
| ID Generation | uuid (v4) | Unique IDs for users, adoptions, sellers |
| Rate Limiting | express-rate-limit | Protect /chat from abuse |
| Frontend UI | Bootstrap 5 | Responsive layout, modals |
| Frontend JS | jQuery | AJAX calls, DOM manipulation |
| Env Config | dotenv | Load secrets from .env |
