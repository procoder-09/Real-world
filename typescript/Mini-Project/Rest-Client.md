Here’s a **complete mini-project: Type-Safe REST Client in TypeScript for React**, explained **step by step**, with **real-world use case**, **best practices**, **generic types**, **Enums/Literal types**, and **error handling**.

This is an **advanced mini-project** and will help you **understand generics, enums, and literal types in a real-world React + TS app**.

---

# 🚀 Mini Project: Type-Safe REST Client

---

## 📌 Project Overview

**Goal:**
Create a **REST client** that:

* Fetches data from an API
* Is **fully type-safe** using TypeScript
* Uses **generics**, **enums**, and **literal types**
* Handles **loading, error, and success states**

**API Used:** [JSONPlaceholder](https://jsonplaceholder.typicode.com/)

* Endpoint example: `https://jsonplaceholder.typicode.com/posts`

---

## 🏗 Project Structure

```
type-safe-rest-client/
│
├── src/
│   ├── api/
│   │    └── client.ts
│   ├── types/
│   │    └── index.ts
│   ├── components/
│   │    └── PostList.tsx
│   ├── App.tsx
│   └── main.tsx
├── package.json
├── tsconfig.json
└── index.html
```

---

## 1️⃣ Define Types (Literal Types + Enums)

**src/types/index.ts**

```ts
// Literal types for status
export type FetchStatus = "idle" | "loading" | "success" | "error";

// API response types
export interface Post {
  userId: number;
  id: number;
  title: string;
  body: string;
}

// Generic API response wrapper
export interface ApiResponse<T> {
  data: T;
  error?: string;
}
```

---

## 2️⃣ Create Type-Safe REST Client (Generics)

**src/api/client.ts**

```ts
import { ApiResponse } from "../types";

// Generic fetch function
export async function fetchApi<T>(url: string): Promise<ApiResponse<T>> {
  try {
    const res = await fetch(url);

    if (!res.ok) {
      return { data: null as any, error: `Error: ${res.status}` };
    }

    const data: T = await res.json();
    return { data };
  } catch (err: any) {
    return { data: null as any, error: err.message || "Unknown Error" };
  }
}
```

✅ Key points:

* Generic `<T>` ensures type safety
* Returns `ApiResponse<T>`
* Proper error handling

---

## 3️⃣ Create React Component

**src/components/PostList.tsx**

```tsx
import React, { useEffect, useState } from "react";
import { fetchApi } from "../api/client";
import { Post, FetchStatus } from "../types";

const PostList: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [status, setStatus] = useState<FetchStatus>("idle");
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadPosts = async () => {
      setStatus("loading");
      const response = await fetchApi<Post[]>(
        "https://jsonplaceholder.typicode.com/posts"
      );

      if (response.error) {
        setError(response.error);
        setStatus("error");
      } else {
        setPosts(response.data);
        setStatus("success");
      }
    };

    loadPosts();
  }, []);

  if (status === "loading") return <p>Loading...</p>;
  if (status === "error") return <p>Error: {error}</p>;

  return (
    <div>
      <h2>Posts</h2>
      {posts.map((post) => (
        <div key={post.id} style={{ marginBottom: "20px" }}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
};

export default PostList;
```

✅ Notes:

* Uses **FetchStatus literal type** for state
* **Generics** ensure `fetchApi<Post[]>` returns correct type
* Strongly typed `posts`

---

## 4️⃣ App Component

**src/App.tsx**

```tsx
import React from "react";
import PostList from "./components/PostList";

const App: React.FC = () => {
  return (
    <div style={{ padding: "20px" }}>
      <h1>Type-Safe REST Client Demo</h1>
      <PostList />
    </div>
  );
};

export default App;
```

---

## 5️⃣ Key Features

✅ **Type-Safety**

* `fetchApi<T>` ensures correct type returned
* Avoids `any`
* Autocomplete and compile-time checks

✅ **Enums / Literal Types**

* Status state (`"idle" | "loading" | "success" | "error"`)
* Enums can be used for **more complex app states**

✅ **Error Handling**

* Catches network errors
* Displays meaningful messages

✅ **Reusability**

* `fetchApi<T>` can fetch **any endpoint**
* Works for **Posts, Users, Comments**, etc.

---

## 6️⃣ Optional Improvement: Enum for Status

```ts
export enum Status {
  Idle = "IDLE",
  Loading = "LOADING",
  Success = "SUCCESS",
  Error = "ERROR",
}
```

* Replace literal type with `Status` enum
* Makes code **more maintainable** in large apps

---

## 7️⃣ Common Mistakes ❌

* Using `any` instead of generics
* Not handling fetch errors
* Not typing state in React
* Forgetting `key` in `map()`

---

## 8️⃣ Best Practices ✅

* Use **generic API functions**
* Use **literal types / enums** for status
* Always type **React state & props**
* Centralize API calls in a **single client**
* Handle errors gracefully

---

## 🎯 Interview Questions

**Q:** Why use generics in REST clients?
✅ To ensure type safety across endpoints.

**Q:** Difference between literal type & enum for status?
✅ Literal: compile-time only, lightweight.
✅ Enum: runtime value, reusable.

**Q:** How to make fetchApi reusable for multiple endpoints?
✅ Accept generic `<T>` and endpoint URL.

---

## 🧾 Final Summary

* **Generics** → reusable, type-safe API functions
* **Literal types** → restrict states like `"loading" | "success"`
* **Enums** → reusable constants for statuses
* **React integration** → type-safe state and props
* Can scale to **complex REST apps**

---

If you want next, I can make a **version with pagination + infinite scroll + type-safe API**, combining your **previous infinite scroll project + REST client**, fully typed in TypeScript.

Do you want me to do that?
