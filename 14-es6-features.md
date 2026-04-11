# Chapter 14 — ES6+ Features (Interview Essentials)

> ES6+ gave us modern JS syntax: `let/const`, arrow functions, promises, classes, modules.  
> Later versions (ES7, ES8, ES9, ES2020+) added async/await, object methods, optional chaining, BigInt, etc.  
> 👉 Almost every interview will include at least 2–3 questions from these features.

---

## 📖 Key Areas

### 🔹 1. `let` & `const` vs `var`
- `var`: function-scoped, hoisted as `undefined`, redeclarable.  
- `let`: block-scoped, hoisted but in **Temporal Dead Zone (TDZ)** until initialized, not redeclarable in same scope.  
- `const`: same as `let` but cannot be reassigned.  

```js
{
  var a = 1;
  let b = 2;
  const c = 3;
}
console.log(a); // 1
// console.log(b); // ReferenceError
```

---

### 🔹 2. Arrow Functions
- Shorter syntax.  
- No own `this`, `arguments`, or `new.target`.  
- Great for callbacks; avoid in object methods/constructors.  

```js
const add = (x, y) => x + y;
```

---

### 🔹 3. Template Literals
```js
const name = "Anmol";
console.log(`Hello, ${name}!`);
```

---

### 🔹 4. Default + Rest + Spread
```js
function greet(msg = "Hi") { console.log(msg); }
greet(); // Hi

function sum(...nums) { return nums.reduce((a,b)=>a+b,0); }
console.log(sum(1,2,3)); // 6

const arr = [1,2];
console.log([...arr,3]); // [1,2,3]
```

---

### 🔹 5. Destructuring
```js
const [a,b] = [1,2];
const {x,y} = {x:10,y:20};
console.log(a,b,x,y); // 1 2 10 20
```

---

### 🔹 6. Modules (import/export)
```js
// utils.js
export const add = (a,b) => a+b;

// main.js
import { add } from "./utils.js";
console.log(add(2,3)); // 5
```

---

### 🔹 7. Classes
Syntactic sugar over prototypes.

```js
class Person {
  constructor(name){ this.name = name; }
  greet(){ console.log("Hello " + this.name); }
}

const p = new Person("Anmol");
p.greet(); // Hello Anmol
```

---

### 🔹 8. Promises & async/await
- **ES6** introduced Promises.  
- **ES8** added async/await (syntactic sugar over Promises).  

---

### 🔹 9. Enhanced Object Literals
```js
let x = 1;
let obj = {
  x,               // shorthand for x: x
  greet() { },     // method shorthand
  ["key"+x]: 42    // computed property name
};
```

---

### 🔹 10. Iterators & Generators
```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
const it = gen();
console.log(it.next().value); // 1
console.log(it.next().value); // 2
```

---

### 🔹 11. ES2020+ Goodies
- **Optional chaining:** `obj?.prop?.subprop`  
- **Nullish coalescing (`??`):** returns right side only if left side is `null` or `undefined`.  
- **BigInt:** supports arbitrarily large integers.  
- **Promise.allSettled:** waits for all promises to finish (resolved or rejected).  

---

## 🧪 Practice (Your Turn)

**Q1**  
Why does this throw?
```js
let x = 1;
{
  console.log(x);
  let x = 2;
}
```

**Q2**  
What is the output?
```js
const obj = {a:1, b:2};
const clone = {...obj, b:99, c:3};
console.log(clone);
```

**Q3**  
What logs?
```js
function test(...args) {
  console.log(args[0], args.length);
}
test(10,20,30);
```

**Q4**  
What does this generator print?
```js
function* g() {
  yield "a";
  yield "b";
}
const it = g();
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
```

**Q5**  
What will this log?
```js
const user = {profile:{name:"Alex"}};
console.log(user.profile?.name);
console.log(user.address?.city);
```

---

## 📝 Your Answers
- Q1: reference error  
- Q2: `{a:1,b:99,c:3}`  
- Q3: `10, 3`  
- Q4: `a`, `b`, `undefined (not sure)`  
- Q5: `Alex`, `undefined`  

---

## ✅ Corrections & Deeper Reasoning

### Q1 — `let` + TDZ
```js
let x = 1;
{
  console.log(x);
  let x = 2;
}
```
**Output**
```
ReferenceError: Cannot access 'x' before initialization
```
- Block `let x` shadows outer `x`.  
- In the **Temporal Dead Zone** until initialized, so `console.log(x)` throws.  
- ✅ Your answer (`ReferenceError`) was correct.

---

### Q2 — Spread override
```js
const obj = {a:1, b:2};
const clone = {...obj, b:99, c:3};
console.log(clone);
```
**Output**
```
{ a: 1, b: 99, c: 3 }
```
- Later properties override earlier ones.  
- ✅ Your answer was correct.

---

### Q3 — Rest params
```js
function test(...args) {
  console.log(args[0], args.length);
}
test(10,20,30);
```
**Output**
```
10 3
```
- First arg = `10`, total args = `3`.  
- You wrote `10, 3` (with a comma) but console prints `10 3`.  
- ✔ Concept was correct.

---

### Q4 — Generator iteration
```js
function* g() {
  yield "a";
  yield "b";
}
const it = g();
console.log(it.next().value);
console.log(it.next().value);
console.log(it.next().value);
```
**Output**
```
a
b
undefined
```
- After second yield, generator is done → `value: undefined`.  
- ✅ Your answer was correct.

---

### Q5 — Optional chaining
```js
const user = {profile:{name:"Alex"}};
console.log(user.profile?.name);
console.log(user.address?.city);
```
**Output**
```
Alex
undefined
```
- `user.profile?.name` works.  
- `user.address` is `undefined`, but `?.city` safely returns `undefined`.  
- ✅ Your answer was correct.

---

## ⚡ Rapid-fire (just outputs)

- **R6**
  ```js
  const nums = [1,2,3];
  console.log(Math.max(...nums));
  ```
  **Output:** `3`

- **R7**
  ```js
  const [a, , b=5] = [10];
  console.log(a, b);
  ```
  **Output:** `10 5`

- **R8**
  ```js
  class A { x = 1; }
  class B extends A {
    constructor() { super(); this.y = 2; }
  }
  console.log(new B());
  ```
  **Output:** `B { x:1, y:2 }`

- **R9**
  ```js
  const greet = name => ({ msg: `Hi ${name}` });
  console.log(greet("Sam"));
  ```
  **Output:** `{ msg: "Hi Sam" }`

- **R10**
  ```js
  console.log(null ?? "fallback");
  console.log(0 ?? 5);
  ```
  **Output:**  
  ```
  fallback
  0
  ```

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** Difference between `var`, `let`, `const`?  
- **Answer:** `var` → function-scoped, hoisted as `undefined`, redeclarable.  
  `let` → block-scoped, TDZ, not redeclarable.  
  `const` → same as let but immutable binding (not reassignable).  

**Q2.** Why prefer `let/const` over `var`?  
- **Answer:** Prevents hoisting bugs, ensures block scoping, improves readability and predictability.

**Q3.** What’s the difference between arrow functions and normal functions?  
- **Answer:** Arrow functions have no `this`, `arguments`, `super`, or `new.target`. They inherit `this` from the enclosing scope, making them ideal for callbacks.

**Q4.** Use case for template literals?  
- **Answer:** Cleaner string concatenation and multiline strings. Example: building HTML templates or dynamic SQL queries.

**Q5.** How do rest and spread differ?  
- **Answer:** `...rest` collects multiple args into an array; `...spread` expands arrays/objects. Rest gathers, spread expands.

**Q6.** What problem do classes solve in JS?  
- **Answer:** They provide a cleaner, more familiar syntax for prototype-based OOP, making inheritance and object creation more intuitive.

**Q7.** What are iterators & generators?  
- **Answer:** Iterators define a sequence interface with `next()`. Generators (`function*`) produce iterators, enabling lazy evaluation and pausable functions.

**Q8.** What is optional chaining and when to use it?  
- **Answer:** `obj?.prop?.sub` safely returns `undefined` if any level is `null/undefined`. Useful to avoid runtime crashes in deeply nested data.

**Q9.** Difference between `??` and `||`?  
- **Answer:** `||` treats falsy values (`0`, `""`, `false`) as fallback, while `??` only falls back on `null` or `undefined`.

**Q10.** How do ES6+ features improve performance?  
- **Answer:** 
  - `let/const` → fewer memory leaks.  
  - Arrow functions → concise + lexical `this`.  
  - Modules → better bundling, tree-shaking.  
  - Async/await → cleaner async flow.  
  - Destructuring/spread → more expressive, less boilerplate.

---

## 📎 Answer Key (Practice Qs Recap)

- **Q1** → `ReferenceError` (TDZ)  
- **Q2** → `{ a:1, b:99, c:3 }`  
- **Q3** → `10 3`  
- **Q4** → `a, b, undefined`  
- **Q5** → `Alex`, `undefined`  

- **Rapid-fire:**  
  - R6 → `3`  
  - R7 → `10 5`  
  - R8 → `B { x:1, y:2 }`  
  - R9 → `{ msg: "Hi Sam" }`  
  - R10 → `fallback`, `0`

---

## ✍️ Quick Interview Script
- “ES6+ features modernized JS with block scoping (`let/const`), concise arrow functions, and modules that enable tree-shaking.”  
- “Async/await built on top of promises simplified async code.”  
- “Optional chaining and nullish coalescing make code safer, while features like destructuring, spread/rest, and classes improve readability and maintainability.”  

---
