Perfect! Let’s break down **Error Handling Patterns in Node.js + Express** in a **detailed, real-world way**, including **best practices, examples, and common pitfalls**.

---

# ⚠️ Node.js + Express — Error Handling Patterns

---

## 1️⃣ Why Error Handling Matters

* Prevents server crashes
* Returns **consistent API responses**
* Helps **debugging and monitoring**
* Improves **user experience**

> Without proper error handling, a single unhandled error can **crash the entire Express app**.

---

## 2️⃣ Types of Errors in Express

1. **Synchronous errors** → thrown directly in route
2. **Asynchronous errors** → promise rejections, async/await
3. **Application errors** → validation failure, unauthorized, not found
4. **System errors** → DB connection, file system failure

---

## 3️⃣ Basic Error Handling

### Synchronous Example

```js
app.get("/sync-error", (req, res) => {
  throw new Error("Something went wrong!");
});
```

* Express automatically catches **sync errors** if there is an **error-handling middleware**.

---

## 4️⃣ Error-Handling Middleware

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    status: "error",
    message: err.message,
  });
});
```

* Middleware signature: `(err, req, res, next)`
* Catches all errors passed via `next(err)` or thrown synchronously

---

## 5️⃣ Async/Await Error Handling

### Problem

```js
app.get("/async-error", async (req, res) => {
  const data = await someAsyncFunction(); // throws error
  res.json(data);
});
```

> This **won’t be caught automatically** by Express.

### Solution 1: Try/Catch

```js
app.get("/async-error", async (req, res) => {
  try {
    const data = await someAsyncFunction();
    res.json(data);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});
```

### Solution 2: Async Wrapper (Recommended)

```js
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get("/async-error", asyncHandler(async (req, res) => {
  const data = await someAsyncFunction();
  res.json(data);
}));
```

> Keeps code clean and centralized error handling works.

---

## 6️⃣ Centralized App Error Class

* Create custom error classes for **HTTP status codes**:

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

---

## 7️⃣ Throwing Custom Errors

```js
const AppError = require("./AppError");

app.get("/notfound", (req, res, next) => {
  return next(new AppError("Resource not found", 404));
});
```

* The error-handling middleware will handle **AppError** objects

```js
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || "error";
  res.status(err.statusCode).json({
    status: err.status,
    message: err.message,
  });
});
```

---

## 8️⃣ Validation & Client Errors

* **400 Bad Request** → invalid input
* **401 Unauthorized** → missing or invalid auth
* **403 Forbidden** → access denied
* **404 Not Found** → resource missing

> Always use structured JSON response for consistency:

```json
{
  "status": "fail",
  "message": "Invalid email format"
}
```

---

## 9️⃣ Best Practices ✅

1. Use **centralized error-handling middleware**
2. Use **custom error classes** for clarity
3. Always return **consistent JSON structure**
4. Handle **async errors** with wrapper or try/catch
5. Avoid sending **stack traces to clients** in production
6. Log errors for monitoring (e.g., Winston, Bunyan, or Sentry)
7. Separate **client vs server errors** (4xx vs 5xx)

---

## 🔟 Common Mistakes ❌

* Not catching async errors → unhandled promise rejections
* Returning inconsistent error responses
* Throwing plain strings instead of `Error` objects
* Exposing sensitive information (stack trace, DB info)
* Duplicating error-handling logic in each route

---

## 1️⃣1️⃣ Real-World Example: Express Error Handling Flow

```js
// routes/user.js
app.get("/user/:id", asyncHandler(async (req, res, next) => {
    const user = await User.findById(req.params.id);
    if (!user) throw new AppError("User not found", 404);
    res.json(user);
}));

// Global error handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(err.statusCode || 500).json({
        status: err.status || "error",
        message: err.message || "Internal server error"
    });
});
```

* Works for **sync + async + custom errors**
* Centralized, maintainable, clean

---

## 1️⃣2️⃣ Interview-Ready Explanation 🎯

> **In Express, error handling is done using middleware. Synchronous errors are caught automatically, while async errors require try/catch or a wrapper. Custom error classes and centralized middleware ensure consistent responses, logging, and clean code structure. Proper error handling differentiates client errors (4xx) from server errors (5xx) and prevents app crashes.**

---

## 1️⃣3️⃣ Summary 🧠

✔ Centralized error-handling middleware
✔ Async errors → try/catch or wrapper
✔ Custom error classes for clarity
✔ Consistent JSON response structure
✔ Client errors = 4xx, Server errors = 5xx
✔ Log errors, avoid exposing sensitive info
✔ Keep routes clean, DRY error handling

---

If you want, I can make a **mini Express project template** with:

* Async routes
* JWT auth
* Centralized error handling
* Rate limiting + logging

This will be a **production-ready starter template**.

Do you want me to do that next?
