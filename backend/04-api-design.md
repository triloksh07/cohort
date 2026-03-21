
# Designing APIs — Real Backend Structure

## 1. First Rule: Don’t Start Coding Immediately

Before writing routes or controllers, first design:

* What APIs?
* What data?
* What flow?
* What validation?
* What responses?
* What errors?

Most beginners skip this → project becomes spaghetti.

**Flow should look like this:**

```
Client → Route → DTO Validation → Controller → Service → Database → Response
```

This is **industry pattern**.

---

# 2. Why Mongoose if Express can talk to MongoDB directly?

Good question from class.

Express → MongoDB (possible)
But we use **Mongoose (ODM)** because:

* Schema validation
* Prevent wrong data types
* Middleware (pre save, etc.)
* Relationships
* Cleaner models
* Indexing
* Defaults
* Enum
* Timestamps
* Select fields
* Hooks

ODM = Object Document Mapper
ORM = Object Relational Mapper

Examples:

* Prisma
* Drizzle
* Sequelize
* TypeORM
* Mongoose

---

# 3. Basic Folder Structure (Simple MVC)

This is the minimum backend structure:

```
root/
    server.js
    .env
    .gitignore

    routes/
    controllers/
    models/
    utils/
    middleware/
    config/
```

But **better structure** (modular):

```
root/
    server.js

    modules/
        user/
            user.model.js
            user.controller.js
            user.service.js
            user.routes.js
            user.dto.js
            user.middleware.js

        cart/
        order/
        product/

    common/
        config/
        middleware/
        utils/
        errors/
```

This is **modular architecture** → scalable.

---

# 4. Separation of Concerns (Very Important Concept)

Each layer has one job:

| Layer      | Responsibility   |
| ---------- | ---------------- |
| Route      | Endpoint         |
| DTO        | Validation       |
| Controller | Handle req/res   |
| Service    | Business logic   |
| Model      | DB schema        |
| Middleware | Auth, validation |
| Utils      | Helpers          |
| Config     | DB, env          |

If controller talks directly to DB → bad design.

Correct flow:

```
Route → Controller → Service → Model → DB
```

---

# 5. Environment Variables

Never hardcode:

```
PORT
DB_URL
JWT_SECRET
EMAIL_PASS
```

Use `.env`

Example:

```js
const PORT = process.env.PORT || 5000;
```

Also:

```
NODE_ENV=development
NODE_ENV=production
```

---

# 6. Standard API Response Structure

Professional APIs never send random JSON.

Bad:

```json
{ "msg": "user created" }
```

Good:

```json
{
  "success": true,
  "message": "User created",
  "data": {...},
  "error": null
}
```

That’s why we create **ApiResponse class**.

Example idea:

```js
class ApiResponse {
    constructor(statusCode, message, data = null) {
        this.statusCode = statusCode;
        this.message = message;
        this.data = data;
        this.success = statusCode < 400;
    }
}
```

---

# 7. Error Handling System

This part is **very important**.

Custom error class:

```js
class ApiError extends Error {
    constructor(statusCode, message) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;

        Error.captureStackTrace(this, this.constructor);
    }

    static badRequest(msg) {
        return new ApiError(400, msg);
    }

    static unauthorized(msg) {
        return new ApiError(401, msg);
    }
}
```

Why?
Because **you don’t want raw errors leaking to users**.

---

# 8. DTO (Data Transfer Object) — Very Important Concept

This is industry practice.

DTO sits between:

```
Client → DTO → Controller → Service → DB
```

DTO job:

* Validate data
* Remove unwanted fields
* Transform data
* Prevent bad data from reaching DB

This is **Zero Trust Architecture**:

> Never trust client data.

---

# 9. Joi Validation Example Concept

Important options:

```
abortEarly: false
```

→ show all errors, not just first

```
stripUnknown: true
```

→ remove extra fields

Very important for security and performance.

Because if someone sends huge garbage data → CPU spike → server crash.

---

# 10. Database User Schema Example Design

Typical user schema in real apps:

```
name
email
password
role
isVerified
verificationToken
refreshToken
resetPasswordToken
resetPasswordExpires
createdAt
updatedAt
```

Mongoose example concepts:

```js
new mongoose.Schema({
    name: {
        type: String,
        trim: true,
        required: true,
    },
    role: {
        type: String,
        enum: ["customer", "seller", "admin"],
        default: "customer",
    },
    isVerified: {
        type: Boolean,
        default: false,
    },
    verificationToken: {
        type: String,
        select: false
    }
}, { timestamps: true })
```

Important:

```
select: false
```

→ field will not be returned from DB queries.

Used for:

* tokens
* password
* secrets

---

# 11. Token Generation

Reset password token:

```js
crypto.randomBytes()
```

Used for:

* email verification
* password reset
* refresh tokens
* API keys

---

# Final Big Picture (Very Important)

This is the **real backend architecture flow**:

```
Client Request
      ↓
Route
      ↓
DTO Validation (Joi/Zod)
      ↓
Controller
      ↓
Service (Business Logic)
      ↓
Model (Mongoose)
      ↓
Database
      ↓
Service
      ↓
Controller
      ↓
ApiResponse
      ↓
Client
```

If you understand this flow deeply →
you understand **backend architecture fundamentals**.

---

# What You Should Do Now

To actually learn this properly, you should build:

### Project Suggestions

Build backend only:

1. Auth system (register, login, logout)
2. Notes API
3. Blog API
4. E-commerce cart API
5. URL shortener API

Start with:

> **Auth + User + Notes API with DTO, ApiError, ApiResponse, modular structure**
