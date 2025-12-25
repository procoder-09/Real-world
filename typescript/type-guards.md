Perfect 👍
**Type Guards** are a **core TypeScript feature** that allows you to **narrow types at runtime**, making your code safer and preventing runtime errors.
I’ll explain it **step by step**, with **real-world examples**, **use cases**, **best practices**, **common mistakes**, and **summary**—just like the previous topics.

---

# 🔵 TypeScript: Type Guards

---

## 1️⃣ What are Type Guards?

> **Type Guards = Runtime checks that tell TypeScript the exact type of a variable**

This allows TypeScript to **narrow down union or any types** safely.

---

### Example

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}
```

✅ Here, `typeof id === "string"` is a **type guard**

* TypeScript knows inside `if` → `id` is a string
* Else → `id` is a number

---

## 2️⃣ Why Type Guards Exist

* TypeScript can’t always infer types at runtime
* Helps **avoid runtime errors**
* Essential when working with **unions, any, unknown**
* Improves **code readability**

---

## 3️⃣ Basic Type Guards

### 🔹 `typeof`

Works for **primitive types** (`string`, `number`, `boolean`, `symbol`, `undefined`)

```ts
function example(x: string | number) {
  if (typeof x === "number") {
    console.log(x + 10); // number
  } else {
    console.log(x.toUpperCase()); // string
  }
}
```

---

### 🔹 `instanceof`

Works for **classes / objects**

```ts
class Dog { bark() {} }
class Cat { meow() {} }

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

---

### 🔹 `in` Operator

Checks **if property exists** in object

```ts
interface Bird { fly: () => void }
interface Fish { swim: () => void }

function move(animal: Bird | Fish) {
  if ("fly" in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

---

## 4️⃣ Custom Type Guards

You can create your **own type guard functions**

```ts
interface Admin { role: "admin" }
interface User { role: "user" }

function isAdmin(user: Admin | User): user is Admin {
  return user.role === "admin";
}

const u: Admin | User = { role: "admin" };

if (isAdmin(u)) {
  console.log("Admin access granted"); // TypeScript knows u is Admin
}
```

> Key syntax: `param is Type`
> Tells TS: *Inside this block, the type is narrowed to Type*

---

## 5️⃣ Real-World Use Cases

### 🔹 API Response Handling

```ts
interface Success { data: string }
interface Error { message: string }

function handle(response: Success | Error) {
  if ("data" in response) {
    console.log(response.data);
  } else {
    console.error(response.message);
  }
}
```

---

### 🔹 React Props

```ts
type ButtonProps = { type: "submit" } | { type: "reset" };

function Button(props: ButtonProps) {
  if (props.type === "submit") {
    console.log("Submit button");
  } else {
    console.log("Reset button");
  }
}
```

---

### 🔹 Complex Nested Objects

```ts
interface Circle { radius: number }
interface Rectangle { width: number; height: number }

function area(shape: Circle | Rectangle) {
  if ("radius" in shape) {
    return Math.PI * shape.radius ** 2;
  } else {
    return shape.width * shape.height;
  }
}
```

---

## 6️⃣ Best Practices

### ✅ Use Built-in Guards First

* `typeof` for primitives
* `instanceof` for classes
* `in` for object properties

### ✅ Use Custom Guards for Complex Types

* Clean, reusable checks
* Improves readability

### ✅ Narrow Types Early

* Helps avoid `as` casting
* Prevents runtime errors

---

## 7️⃣ Common Mistakes

❌ Using `any` and ignoring type checks
❌ Casting with `as` without checks
❌ Overcomplicating guards
❌ Not narrowing union types properly

---

## 8️⃣ Interview-Level Explanation

> Type Guards are runtime checks that **narrow the type of a variable**, allowing TypeScript to understand which type is being handled, preventing runtime errors and improving type safety.

---

## 📌 Summary

✔ Type Guards narrow types at runtime
✔ Built-in: `typeof`, `instanceof`, `in`
✔ Custom: `param is Type`
✔ Essential for union types, APIs, and complex objects
✔ Helps avoid `any` and unsafe casting

---

If you want next, I can explain:
**“Generics in TypeScript with advanced examples”** — very important for **React, Redux, and backend APIs**.

Do you want me to go next into **Generics**?
