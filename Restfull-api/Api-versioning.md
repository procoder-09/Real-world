Perfect! Let’s dive into **API Versioning in RESTful API design**, a key concept for building scalable, maintainable SaaS or web applications. I’ll explain **clearly**, with **real-world analogies**, **examples in Node.js + Express**, **best practices**, and a **summary**.

---

# 🔵 What is API Versioning?

> **API Versioning** is the practice of **managing changes and updates in your API** by creating separate versions so that old clients don’t break.

* Allows adding new features, changing responses, or fixing bugs without breaking existing clients.
* Typically, **version numbers** (v1, v2) are used in URLs, headers, or query parameters.

---

# 🏠 Real-World Analogy

Think of **software updates**:

| Concept    | Analogy                                    |
| ---------- | ------------------------------------------ |
| v1 API     | Old version of a mobile app                |
| v2 API     | New version of the app with features       |
| Client     | User using app                             |
| Versioning | Ensures old users don’t lose functionality |

> Users on old version still work, while new features are available for others.

---

# 🔄 Versioning Strategies

## 1️⃣ URI / Path Versioning (Most common)

```text
GET /api/v1/products
GET /api/v2/products
```

* Clear, easy to implement
* Supports multiple versions simultaneously

### Express Example:

```javascript
const express = require("express");
const app = express();

const v1Routes = require("./routes/v1/productRoutes");
const v2Routes = require("./routes/v2/productRoutes");

app.use("/api/v1/products", v1Routes);
app.use("/api/v2/products", v2Routes);

app.listen(3000, () => console.log("Server running on 3000"));
```

---

## 2️⃣ Query Parameter Versioning

```text
GET /api/products?version=1
GET /api/products?version=2
```

* Easy to implement
* Less visible than URI versioning
* Can be **parsed in middleware**

```javascript
app.use("/api/products", (req, res, next) => {
    const version = req.query.version || "1";
    if (version === "1") require("./routes/v1/productRoutes")(req, res, next);
    else require("./routes/v2/productRoutes")(req, res, next);
});
```

---

## 3️⃣ Header Versioning

```text
GET /api/products
Headers: Accept-Version: v1
```

* Clean URL
* Requires **middleware** to read headers
* Common in **enterprise APIs**

```javascript
app.use("/api/products", (req, res, next) => {
    const version = req.headers["accept-version"] || "v1";
    if (version === "v1") require("./routes/v1/productRoutes")(req, res, next);
    else require("./routes/v2/productRoutes")(req, res, next);
});
```

---

## 4️⃣ Content Negotiation (Media Type Versioning)

```text
Accept: application/vnd.myapp.v1+json
Accept: application/vnd.myapp.v2+json
```

* Less common in small apps
* More **RESTful**, supports advanced client negotiation

---

# 🧩 Real-World SaaS Dashboard Use Cases

1. **Product API changes**

   * v1: `price` as number
   * v2: `price` as object with `value` and `currency`

2. **User management changes**

   * v1: basic profile
   * v2: profile with social links

3. **Analytics API**

   * v1: basic stats
   * v2: enhanced metrics + filters

---

# ⚡ Best Practices

1. **Start with v1** → don’t version too early
2. **URI versioning is the simplest & most visible**
3. Keep **old versions active** until clients migrate
4. **Document all versions** in Swagger/OpenAPI
5. Avoid breaking changes → create new version instead
6. Use **semantic versioning** if needed (`v1.0`, `v1.1`)
7. Consider **deprecation strategy** → announce when v1 will be removed

---

# 🧾 Summary

* **API Versioning** allows safe evolution of REST APIs
* Common strategies:

  * **URI / Path versioning** → `/api/v1/products`
  * **Query parameters** → `?version=1`
  * **Header versioning** → `Accept-Version: v1`
  * **Content negotiation** → media type versioning
* Best practices: start with v1, avoid breaking changes, document versions, plan deprecation
* Essential for **large SaaS apps and public APIs**

---

If you want, I can create a **full Product Management API example** showing **versioned API endpoints (v1 & v2)**, **Swagger docs for each version**, and **best practices for backward compatibility**, which will be **portfolio-ready and real-world practical**.

Do you want me to do that next?
