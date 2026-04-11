# Chapter 12 — Error Handling & Best Practices

> Error handling separates coders from production-ready developers. Interviewers love this topic because it shows how you build resilient, maintainable apps.

---

## 📘 Summary / One-liner
Error handling in JavaScript covers synchronous `try/catch/finally`, Promise-based `.catch`, async/await `try/catch`, global handlers, and best practices (throw `Error` objects, don't swallow errors, use logging/monitoring). Knowing the differences between sync and async error flow is interview gold.

---

## 📚 Theory (Textbook-style)

### 🔹 Types of Errors in JavaScript
- **ReferenceError** — accessing a variable that doesn’t exist.
  ```js
  console.log(x); // ReferenceError: x is not defined
  ```
- **TypeError** — invalid operation on a type (e.g., calling something that's not a function).
  ```js
  null.f(); // TypeError: Cannot read property 'f' of null
  ```
- **SyntaxError** — invalid code / parse error.
  ```js
  eval("if ( ) {"); // SyntaxError: Unexpected token ')'
  ```
- **RangeError** — invalid numeric range or length.
  ```js
  new Array(-1); // RangeError: Invalid array length
  ```
- **Custom Errors** — created using `throw` (you can throw *anything* but prefer `Error`).

### 🔹 `try / catch / finally` (sync)
```js
try {
  console.log("Start");
  throw new Error("Something broke");
} catch (e) {
  console.log("Caught:", e.message);
} finally {
  console.log("Always runs");
}
```
**Output**
```
Start
Caught: Something broke
Always runs
```

- `finally` always executes (use it for cleanup).
- `catch` receives the thrown value (object/primitive).
- Best practice: throw `Error` objects to preserve stack trace.

### 🔹 Throwing custom errors
Prefer `Error` (or subclass) to keep stack info:
```js
class BadRequestError extends Error {
  constructor(message) {
    super(message);
    this.name = 'BadRequestError';
  }
}

function divide(a, b) {
  if (b === 0) throw new BadRequestError("Division by zero");
  return a / b;
}
```

### 🔹 Promises error handling
- Use `.catch()` to capture rejections.
```js
Promise.reject("fail")
  .then(res => console.log(res))
  .catch(err => console.log("Caught:", err));
```
- `.finally()` runs regardless of resolved/rejected—but it does **not** consume the error.

### 🔹 async/await error handling
- `throw` inside an `async` function results in a rejected Promise.
- Use `try/catch` **inside** the `async` function to handle it.
```js
async function fetchData() {
  try {
    const result = await Promise.reject("network");
  } catch (err) {
    console.log("Caught:", err);
  }
}
fetchData();
```

### 🔹 Global handlers (last-resort)
**Browser**
```js
window.onerror = (msg, url, line, col, err) => {
  console.log("Global Error:", msg, err && err.stack);
};
window.addEventListener('unhandledrejection', ev => {
  console.log('Unhandled Rejection (browser):', ev.reason);
});
```

**Node.js**
```js
process.on("uncaughtException", err => {
  console.error("Uncaught Exception:", err);
  // In many cases, you should shutdown gracefully after logging.
});

process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  // Prefer handling rejections where they happen rather than relying on this.
});
```

---

## 🧠 Key Notes (quick interview-ready bullets)
- `try/catch` **only** handles synchronous exceptions in the same call stack.
- Async errors must be handled with `.catch()` or inside `async` functions using `try/catch`.
- `finally` runs after `try/catch`, **always**, and does not swallow errors.
- Prefer throwing `Error` objects for stack traces and consistent debugging.
- Use global handlers only as a last-resort / crash-logging mechanism; they are not a substitute for proper local handling.
- Avoid swallowing errors silently (`catch(e) {}`) — always at least log or rethrow.
- For long-lived Node services, prefer process-level monitoring, graceful shutdown, and structured logging/alerting.

**Interview-line (short):**  
> “Try/catch works synchronously; promises need `.catch`; inside `async` functions, `throw` returns a rejected Promise — handle where you can, use global handlers only for last-resort logging and graceful shutdown.”

---

## 🧪 Practice — step-by-step (try before you open answers)

> Try to predict outputs and explain why. Then open the `<details>`.

---

### 1) Q1 — `typeof` of thrown primitive
```js
try {
  throw "Oops!";
} catch (e) {
  console.log(typeof e);
}
```
<details>
<summary>✅ Answer & explanation</summary>

**Output**
```
string
```

**Why:** You can throw any value in JS (primitives included). Here we threw a string, so `typeof e === "string"`.

**Interview-line:** “JS allows throwing any value, but best practice is to throw `Error` instances for stack traces and readable types.”
</details>

---

### 2) Q2 — async throw handling
```js
async function test() {
  throw new Error("fail");
}
test().catch(e => console.log("Caught:", e.message));
```
<details>
<summary>✅ Answer & explanation</summary>

**Output**
```
Caught: fail
```

**Why:** `throw` inside an `async` function rejects the returned Promise. `.catch()` handles the rejection.

**Interview-line:** “Throwing inside `async` is equivalent to returning a rejected promise; consume it with `.catch()` or `try/catch` inside `async`.”
</details>

---

### 3) Q3 — `finally` runs first
```js
Promise.reject("bad")
  .finally(() => console.log("cleanup"))
  .catch(err => console.log("err:", err));
```
<details>
<summary>✅ Answer & explanation</summary>

**Output**
```
cleanup
err: bad
```

**Why:** `.finally()` executes before the promise chain continues to `.catch()`. It does not consume the rejection — the error is passed down to `.catch()`.

**Interview-line:** “Use `finally` for cleanup; it runs before downstream handlers and does not suppress errors.”
</details>

---

### 4) Q4 — async throw outside try/catch (classic trap)
```js
try {
  setTimeout(() => { throw new Error("Async boom"); }, 0);
} catch (e) {
  console.log("Caught:", e.message);
}
```
<details>
<summary>✅ Answer & explanation</summary>

**Output**
```
Uncaught Error: Async boom
```
(or platform-specific uncaught error logging)

**Why:** The `throw` inside `setTimeout` is executed in a different tick (call stack), so the outer `try/catch` cannot catch it. To catch, handle errors inside the callback or use Promisified patterns.

**Fixes:**
- Wrap callback logic in `try/catch` inside the callback.
- Use Promises (e.g., `new Promise((res, rej) => setTimeout(() => rej(new Error('boom')),0)).catch(...)`).

**Interview-line:** “try/catch only applies to the current call stack. Async callbacks need their own handling.”
</details>

---

### 5) Q5 — throwing non-Error and reading stack
```js
try {
  throw { code: 400, msg: "Bad" };
} catch (e) {
  console.log(e.code, e.msg, typeof e, e.stack);
}
```
<details>
<summary>✅ Answer & explanation</summary>

**Typical Output**
```
400 Bad object undefined
```

**Why:** Thrown object is not an Error, so `e.stack` is usually `undefined`. Another reason to throw `Error` is stack availability.

**Interview-line:** “Throwing plain objects loses stack info — prefer `new Error()` or subclassing it.”
</details>

---

## ⚡ Rapid-fire — just predict outputs (answers below)

- **R1**
  ```js
  try {
    throw 42;
  } catch (e) {
    console.log(e + 1);
  }
  ```
  **Output:** `43`

- **R2**
  ```js
  Promise.resolve().then(() => {
    throw "bad";
  }).catch(e => console.log(e));
  ```
  **Output:** `bad`

- **R3**
  ```js
  (async () => {
    try {
      return await Promise.reject("fail");
    } catch (e) {
      return "recovered";
    }
  })().then(console.log);
  ```
  **Output:** `recovered`

- **R4**
  ```js
  try {
    JSON.parse("{ bad json }");
  } catch (e) {
    console.log(e.name);
  }
  ```
  **Output:** `SyntaxError`

- **R5 (Node specific)**
  ```js
  process.on("unhandledRejection", err => console.log("Oops:", err));
  Promise.reject("lost");
  ```
  **Output:** `Oops: lost` (note: timing may vary—Node prints this handler message if registered before rejection)

---

## 🏢 Top MNC / FAANG-style interview questions (with crisp interview lines)

1. **Q:** Can `try/catch` catch asynchronous errors?  
   **Crisp answer:** No — `try/catch` only catches synchronous exceptions in the same call stack. Async errors must be handled with `.catch()` (Promises) or `try/catch` inside `async` functions.

2. **Q:** Why prefer throwing `Error` objects over strings/objects?  
   **Crisp answer:** `Error` instances provide a message, type and stack trace which vastly improves debugging and observability.

3. **Q:** What’s the purpose of `.finally()` on a Promise?  
   **Crisp answer:** Run cleanup code after promise resolution or rejection. It executes before downstream handlers and does not remove the original error/result.

4. **Q:** How would you structure error handling in a production Node app?  
   **Crisp answer:** Handle errors locally where they occur, use structured logging, convert errors to meaningful responses at API boundaries, use process-level handlers for last-resort logging, and implement graceful shutdown and alerts.

5. **Q:** How do you avoid swallowing errors in async code?  
   **Crisp answer:** Always return or rethrow after logging if you cannot handle an error; avoid empty `catch` blocks and prefer `.catch()` or `try/catch` in `async` functions with clear handling.

6. **Q:** When to use `uncaughtException` / `unhandledRejection` handlers?  
   **Crisp answer:** Only as a last-resort for logging/alerting and to attempt a graceful shutdown — not for normal error flow. Prefer explicit handling at source.

---

## ✅ Best Practices Cheatsheet (for interview recall)
- Use `const`/`let` properly; errors caused by accidental `var` leaks are common in legacy code.  
- Throw `Error` objects (or subclasses).  
- Handle rejections — every Promise should be handled or returned.  
- Log with context (request id, user id).  
- Don’t catch and ignore errors. If you catch and can’t handle, rethrow.  
- Add tracing / monitoring (Sentry, Datadog) for production errors.  
- For user-facing errors, map internal errors to safe messages and proper status codes (avoid leaking internals).  
- Test error paths — unit test unusual branches and failure cases.

---

## 📎 Full Answer Key (Practice Recap)
- **Q1** → `string`  
- **Q2** → `Caught: fail`  
- **Q3** → `cleanup` then `err: bad`  
- **Q4** → `Uncaught Error: Async boom` (uncaught)  
- **Q5** → prints `400 Bad object undefined` (no stack)

- **Rapid-fire Answers**
  - R1 → `43`
  - R2 → `bad`
  - R3 → `recovered`
  - R4 → `SyntaxError`
  - R5 → `Oops: lost` (if handler registered before rejection)

---

## ✍️ Quick "What to say in an interview" script
- “I handle errors at the closest possible layer and always convert them into meaningful responses at boundaries. For async code I prefer `async/await` with `try/catch` or Promise chains with `.catch()`; for critical runtime failures I have global logging and graceful shutdown to avoid data corruption.”
- “I throw `Error` objects or custom subclasses so we preserve stack traces and can filter by type in monitoring tools.”
- “Global handlers (`process.on`) are for last-resort logging — not for regular error handling.”

---