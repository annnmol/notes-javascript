# Chapter 17 — JavaScript Design Patterns – Part 3: The Factory Pattern

> The **Factory Pattern** is about **object creation without exposing the creation logic**.  
> Instead of calling `new` everywhere, the client calls a factory function/class → factory decides what to return.  

👉 **Why interviewers ask:**  
Tests whether you understand **OOP, abstraction, and clean code practices**.  

---

## 📘 Theory

### 🔹 What is the Factory Pattern?
- A **function or class** that creates objects.  
- Hides the construction logic from the client.  
- The client just asks: “Give me an object of type X,” and the factory decides what to return.  

---

### 🔹 Code Example 1 – Simple Factory
```js
class Dog {
  speak() { return "Woof"; }
}
class Cat {
  speak() { return "Meow"; }
}

function AnimalFactory(type) {
  if (type === "dog") return new Dog();
  if (type === "cat") return new Cat();
}

const a1 = AnimalFactory("dog");
console.log(a1.speak()); // Woof

const a2 = AnimalFactory("cat");
console.log(a2.speak()); // Meow
```

✅ Each call returns a **new object of the requested type**.  

---

### 🔹 Code Example 2 – Configurable Factory
```js
function CarFactory(model) {
  return {
    model,
    drive: () => console.log(model + " is driving")
  };
}

const car1 = CarFactory("Tesla");
const car2 = CarFactory("BMW");

car1.drive(); // Tesla is driving
car2.drive(); // BMW is driving
```

✅ Factory encapsulates object creation and allows variations.  

---

### 🔹 Real Interview Use Cases
- **UI components** → Button factory (PrimaryButton / SecondaryButton).  
- **Database connectors** → MySQL vs MongoDB clients.  
- **Payment gateways** → Stripe vs PayPal factory.  

---

## 🧠 Key Notes
- Factory centralizes **object creation logic**.  
- Promotes **abstraction**: clients don’t need to know *how* objects are created.  
- Useful when you need to create multiple object types dynamically.  
- Helps with **loose coupling** → client code depends only on the factory interface.  
- Overuse can reduce clarity → use when creation logic is complex or repeated.  

**Interview line:**  
👉 “Factory Pattern abstracts the `new` keyword and centralizes creation logic, making code easier to maintain and extend.”  

---

## 🎯 Practice (interactive)

### Q1 — Factory return objects
```js
function ShapeFactory(type) {
  return {
    type,
    draw() { return `Drawing a ${type}`; }
  };
}

const s1 = ShapeFactory("circle");
const s2 = ShapeFactory("square");
console.log(s1.draw());
console.log(s2.draw());
```

<details>
<summary>✅ Answer</summary>

Output:
```
Drawing a circle
Drawing a square
```

- Each call returns a **new object** with its own `type`.  
- **Interview line:** “Factories generate new instances each time you call them — unlike Singletons.”  
</details>

---

### Q2 — Conceptual
**Q:** When would you prefer Factory over directly using `new`?  

<details>
<summary>✅ Answer</summary>

- When creation logic is **complex or conditional**.  
- When you want a **single API** to produce multiple types.  
- When the client **shouldn’t know construction details**.  

**Examples:**  
- Button factory → returns `PrimaryButton`, `SecondaryButton`, etc.  
- Payment service factory → returns Stripe or PayPal client.  

**Interview line:**  
👉 “I’d use a Factory when I need to encapsulate creation logic, keep client code clean, and support multiple variations.”  
</details>

---

## ⚡ Rapid-fire (just outputs)

- **R1**
  ```js
  function Logger(type) {
    return { log: msg => console.log(`[${type}] ${msg}`) };
  }
  const l1 = Logger("info");
  const l2 = Logger("error");
  l1.log("ok");
  l2.log("fail");
  ```
  **Output:**
  ```
  [info] ok
  [error] fail
  ```

- **R2**
  ```js
  class A {}
  class B {}
  function Factory(flag) {
    return flag ? new A() : new B();
  }
  console.log(Factory(true) instanceof A);
  console.log(Factory(false) instanceof B);
  ```
  **Output:**  
  ```
  true
  true
  ```

- **R3**
  ```js
  function UserFactory(role) {
    if (role === "admin") return { role, canDelete:true };
    return { role, canDelete:false };
  }
  console.log(UserFactory("admin"));
  console.log(UserFactory("user"));
  ```
  **Output:**
  ```
  { role: "admin", canDelete: true }
  { role: "user",  canDelete: false }
  ```

- **R4**
  ```js
  const Service = (() => {
    let instance;
    function create() { return { id: Math.random() }; }
    return { get: () => instance || (instance = create()) };
  })();

  console.log(Service.get() === Service.get());
  ```
  **Output:** `true`

- **R5**
  ```js
  function Animal(name) { this.name = name; }
  const f = (type) => new Animal(type);
  console.log(f("dog") === f("dog"));
  ```
  **Output:** `false` (new object each call)

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** What is the Factory Pattern?  
- **Answer:** It abstracts object creation logic; the client just asks for an object, and the factory decides which concrete instance to return.  

**Q2.** How is a Factory different from `new` keyword usage?  
- **Answer:** `new` requires the client to know the exact class. Factory hides creation details and can decide which type to instantiate at runtime.  

**Q3.** Compare Factory vs Singleton.  
- **Answer:** Factory produces new instances on demand; Singleton ensures only one instance exists. Factories promote flexibility, Singletons enforce uniqueness.  

**Q4.** Real-world use cases of Factory?  
- **Answer:** UI components (Button, Modal factories), database connectors (MySQL vs Mongo), payment gateways (Stripe vs PayPal).  

**Q5.** What are pros & cons of Factory Pattern?  
- **Pros:** Encapsulation, reusability, easy to swap implementations.  
- **Cons:** Can add unnecessary complexity if object creation is simple.  

**Q6.** How does Factory improve testability?  
- **Answer:** By abstracting creation, you can swap real implementations with mocks/stubs during testing.  

**Q7.** How is the Module Pattern related to the Factory Pattern?  
- **Answer:** Both encapsulate code. Modules expose a static singleton by default, while Factories dynamically create new instances.  

**Q8.** Can a Factory return different classes?  
- **Answer:** Yes. That’s the main advantage — it can decide at runtime which subclass/object to return based on conditions.  

**Q9.** How would you implement a Factory with ES6 classes?  
- **Answer:** Create a static `create` method that returns instances of different subclasses:  
  ```js
  class AnimalFactory {
    static create(type) {
      if (type === "dog") return new Dog();
      if (type === "cat") return new Cat();
    }
  }
  ```  

**Q10.** What’s the difference between Factory Pattern and Abstract Factory?  
- **Answer:** Factory creates objects of one family. Abstract Factory provides an interface for creating **families of related objects** without specifying exact classes.  

---

## 📎 Answer Key (Practice Recap)

- **Q1:** `Drawing a circle`, `Drawing a square`  
- **Q2:** Use Factory when creation logic is complex/hidden; client just calls factory, not `new`.  

- **Rapid-fire:**  
  - R1 → `[info] ok` / `[error] fail`  
  - R2 → `true`, `true`  
  - R3 → `{ role: "admin", canDelete: true }`, `{ role: "user", canDelete: false }`  
  - R4 → `true`  
  - R5 → `false`  

---

## ✍️ Quick Interview Script
- “Factory Pattern hides object creation logic and provides a single entry point for different object types.”  
- “It’s perfect for creating variations of objects like services, connectors, or UI components.”  
- “Overuse adds complexity — I use factories when object creation involves conditions, not for trivial `new` calls.”  

---