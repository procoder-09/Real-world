Excellent 👍
**Union & Intersection Types** are **core TypeScript features** that you will use **daily** in **React, APIs, forms, auth, Redux, and backend contracts**.

I’ll explain this **exactly like your previous topics**:
👉 **simple → real-world analogy → practical examples → use cases → best practices → pitfalls → summary**

---

# 🔀 Union & Intersection Types in TypeScript

(Simple, Practical & Real-World)

---

## 1️⃣ Why Do We Need Union & Intersection?

JavaScript is flexible, but **too flexible** 😅
TypeScript adds **controlled flexibility**.

Union & Intersection help when:

* Data can be **one of many types**
* Data must satisfy **multiple types at once**

---

# PART 1️⃣ — UNION TYPES (`|`)

---

## 2️⃣ What is a Union Type? (Plain English)

> **A union type means a value can be ONE of several types.**

### Syntax

```ts
type Result = string | number;
```

👉 `Result` can be **string OR number**

---

## 🧠 Real-World Analogy

### 🎫 Ticket Example

A movie ticket can be:

* Online ticket
* Physical ticket

You only need **one**, not both.

---

## 3️⃣ Simple Union Example

```ts
let id: number | string;

id = 101;
id = "A102";
// id = true ❌
```

---

## 4️⃣ Union with Functions

```ts
function printId(id: number | string) {
  console.log(id);
}
```

✔ Accepts flexible input
✔ Still type-safe

---

## 5️⃣ Narrowing Union Types (VERY IMPORTANT 🔥)

TypeScript needs help to know **which type is currently used**.

---

### 🔍 `typeof` narrowing

```ts
function format(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}
```

---

### 🔍 `in` operator

```ts
type Admin = { role: "admin"; permissions: string[] };
type User = { role: "user"; email: string };

type Person = Admin | User;

function getDetails(person: Person) {
  if ("permissions" in person) {
    console.log(person.permissions);
  } else {
    console.log(person.email);
  }
}
```

---

## 6️⃣ Real-World Use Cases of Union Types

---

### 🧾 Use Case 1: API Status

```ts
type Status = "loading" | "success" | "error";

let apiStatus: Status;
```

Used in:

* React loading states
* UI feedback

---

### 📦 Use Case 2: API Responses

```ts
type ApiResponse =
  | { success: true; data: User[] }
  | { success: false; error: string };
```

✔ Forces proper error handling
✔ Prevents runtime bugs

---

### 🔐 Use Case 3: Auth Roles

```ts
type Role = "admin" | "user" | "guest";
```

---

## 7️⃣ Common Mistakes with Union ❌

❌ Forgetting to narrow types
❌ Assuming all properties exist
❌ Overusing `any` instead

---

---

# PART 2️⃣ — INTERSECTION TYPES (`&`)

---

## 8️⃣ What is an Intersection Type? (Plain English)

> **Intersection means combining multiple types into one.**
> The value must satisfy **ALL types**.

---

## 🧠 Real-World Analogy

### 🧑‍💼 Employee Example

An employee is:

* A Person
* AND a Worker

They must have **both identities**.

---

## 9️⃣ Simple Intersection Example

```ts
type Person = {
  name: string;
};

type Employee = {
  employeeId: number;
};

type Staff = Person & Employee;

const staff: Staff = {
  name: "Ramya",
  employeeId: 101
};
```

---

## 10️⃣ Intersection with Interfaces (Common in React)

```ts
interface ButtonProps {
  label: string;
}

interface DisabledProps {
  disabled: boolean;
}

type ButtonConfig = ButtonProps & DisabledProps;
```

---

## 11️⃣ Real-World Use Cases of Intersection Types

---

### ⚛️ Use Case 1: React Component Props

```ts
type BaseProps = {
  id: string;
};

type StyleProps = {
  className?: string;
};

type Props = BaseProps & StyleProps;
```

---

### 🌐 Use Case 2: API Request Payloads

```ts
type Pagination = {
  page: number;
};

type Filter = {
  search: string;
};

type RequestParams = Pagination & Filter;
```

---

### 🔐 Use Case 3: Auth User Object

```ts
type User = {
  id: number;
  email: string;
};

type Token = {
  token: string;
};

type AuthUser = User & Token;
```

---

## 12️⃣ Intersection vs Extends

```ts
interface Admin extends User {
  permissions: string[];
}
```

vs

```ts
type Admin = User & {
  permissions: string[];
};
```

✔ Both valid
✔ `&` works with unions
✔ `extends` is interface-only

---

## 13️⃣ Union vs Intersection (IMPORTANT 🔥)

| Feature | Union (`|`) | Intersection (`&`) |
|------|-----------|----------------|
| Meaning | One of many | Combine all |
| Flexibility | High | Strict |
| Use when | Options | Requirements |
| Example | `string \| number` | `User & Token` |

---

## 14️⃣ Advanced Pattern: Union + Intersection Together 🔥

```ts
type Admin = {
  role: "admin";
  permissions: string[];
};

type User = {
  role: "user";
  email: string;
};

type Base = {
  id: number;
};

type Account = Base & (Admin | User);
```

✔ Very common in real apps
✔ Used in auth systems

---

## 15️⃣ Best Practices ✅

✔ Use union for states & variants
✔ Use intersection for combining props
✔ Always narrow unions
✔ Prefer literal unions over enums
✔ Keep types small & composable

---

## 16️⃣ Common Pitfalls ❌

❌ Accessing properties without narrowing
❌ Over-complicated types
❌ Mixing union & intersection blindly

---

## 🎯 Interview-Ready Definitions

**Union Type**

> Allows a variable to be one of multiple types.

**Intersection Type**

> Combines multiple types into one.

---

## 🧠 Mental Model (Remember This)

* **Union (`|`)** → *OR*
* **Intersection (`&`)** → *AND*

---

## 📌 Summary

✔ Union = flexibility
✔ Intersection = composition
✔ Both improve type safety
✔ Used heavily in React, APIs, auth

---

### 🔜 Recommended Next (Perfect for You)

Since you’re mastering **advanced frontend + backend**:

* **Discriminated Unions (deep dive)**
* **Generics with real projects**
* **Utility Types (`Partial`, `Pick`, `Omit`)**
* **TypeScript for API contracts**

Just tell me which one 🚀
