# Classes

> Quay lại [Mục lục](./README.md) | Trước: [Enhanced Object Literals](./07-enhanced-object-literals.md)

## 📌 Tổng quan

ES6 Classes là **syntactic sugar** (đường cú pháp) trên hệ thống **prototypal inheritance** sẵn có của JavaScript. Chúng cung cấp cú pháp quen thuộc hơn cho OOP mà không thay đổi cơ chế bên dưới.

> **Lưu ý**: JavaScript KHÔNG phải ngôn ngữ class-based (như Java/C++). Classes trong JS vẫn hoạt động dựa trên prototype chain.

---

## 1. Class Declaration vs Constructor Function

### 1.1. ES5 — Constructor Function

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

Person.prototype.getAge = function() {
  return this.age;
};

// Static method
Person.create = function(name, age) {
  return new Person(name, age);
};

const john = new Person("John", 25);
john.greet(); // "Hello, I'm John"
```

### 1.2. ES6 — Class

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }

  getAge() {
    return this.age;
  }

  static create(name, age) {
    return new Person(name, age);
  }
}

const john = new Person("John", 25);
john.greet(); // "Hello, I'm John"
```

### 1.3. Class vs Function — Khác biệt quan trọng

```javascript
// 1. Class KHÔNG được hoisted (khác function declaration)
const p = new Person("John"); // ❌ ReferenceError
class Person {}

// 2. Class body luôn chạy trong strict mode
class StrictClass {
  method() {
    // Tự động strict mode
    undeclared = 5; // ❌ ReferenceError (strict mode)
  }
}

// 3. Class methods KHÔNG enumerable
class MyClass {
  method() {}
}
console.log(Object.keys(MyClass.prototype)); // [] (empty)

// Function.prototype methods LÀ enumerable
function MyFunc() {}
MyFunc.prototype.method = function() {};
console.log(Object.keys(MyFunc.prototype)); // ["method"]

// 4. Class PHẢI dùng new
class Foo {}
Foo(); // ❌ TypeError: Class constructor cannot be invoked without 'new'
```

---

## 2. Class Expression

```javascript
// Named class expression
const Person = class PersonClass {
  constructor(name) {
    this.name = name;
  }
  
  identify() {
    // PersonClass chỉ accessible bên trong class body
    return PersonClass.name; // "PersonClass"
  }
};

// Anonymous class expression
const Animal = class {
  constructor(name) {
    this.name = name;
  }
};

// Immediately invoked class (hiếm dùng)
const instance = new (class {
  constructor() {
    this.value = 42;
  }
  getValue() {
    return this.value;
  }
})();
console.log(instance.getValue()); // 42

// Class as argument
function createSingleton(ClassDef) {
  return new ClassDef();
}
const single = createSingleton(class {
  greet() { return "Hello!"; }
});
```

---

## 3. Constructor

```javascript
class User {
  // Constructor — gọi khi dùng new User()
  constructor(name, email) {
    // this = instance mới được tạo
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
    
    // Có thể return object khác (hiếm dùng)
    // Nếu return primitive → bị bỏ qua
    // Nếu return object → object đó được dùng thay instance
  }
}

// Nếu không khai báo constructor, một default constructor tự động được tạo:
// constructor() {} — cho class thường
// constructor(...args) { super(...args); } — cho class kế thừa
```

---

## 4. Getters và Setters

```javascript
class Circle {
  constructor(radius) {
    this._radius = radius; // Convention: _ cho "private"
  }

  // Getter — truy cập như property
  get radius() {
    return this._radius;
  }

  // Setter — validation khi gán giá trị
  set radius(value) {
    if (typeof value !== "number") {
      throw new TypeError("Radius must be a number");
    }
    if (value <= 0) {
      throw new RangeError("Radius must be positive");
    }
    this._radius = value;
  }

  // Computed getter (read-only)
  get area() {
    return Math.PI * this._radius ** 2;
  }

  get circumference() {
    return 2 * Math.PI * this._radius;
  }

  get diameter() {
    return this._radius * 2;
  }

  set diameter(d) {
    this._radius = d / 2;
  }
}

const circle = new Circle(5);
console.log(circle.radius);       // 5 (getter)
console.log(circle.area);         // 78.54... (computed)
console.log(circle.diameter);     // 10

circle.radius = 10;    // setter (validation OK)
circle.diameter = 6;   // setter → radius = 3
circle.radius = -5;    // ❌ RangeError!
circle.radius = "abc"; // ❌ TypeError!
```

---

## 5. Static Methods và Properties

Static members thuộc về **class**, không thuộc về instance.

```javascript
class MathUtils {
  // Static property (ES2022)
  static PI = 3.14159;
  static E = 2.71828;

  // Static method
  static add(a, b) {
    return a + b;
  }

  static multiply(a, b) {
    return a * b;
  }

  // Factory method
  static random(min = 0, max = 1) {
    return Math.random() * (max - min) + min;
  }

  // Static method gọi static method khác
  static circleArea(radius) {
    return MathUtils.PI * radius ** 2;
    // hoặc: return this.PI * radius ** 2; (this = class trong static)
  }
}

// Gọi trực tiếp trên class
console.log(MathUtils.PI);           // 3.14159
console.log(MathUtils.add(2, 3));    // 5
console.log(MathUtils.random(1, 10));

// ❌ Không thể gọi từ instance
const utils = new MathUtils();
utils.add(2, 3); // ❌ TypeError: utils.add is not a function
```

### Use Cases cho Static

```javascript
class User {
  static #nextId = 1;

  constructor(name) {
    this.id = User.#nextId++;
    this.name = name;
  }

  // Factory methods
  static fromJSON(json) {
    const data = JSON.parse(json);
    return new User(data.name);
  }

  static fromObject({ name }) {
    return new User(name);
  }

  // Utility
  static isValidName(name) {
    return typeof name === "string" && name.length >= 2;
  }
}

const user1 = new User("John");           // id: 1
const user2 = User.fromJSON('{"name":"Jane"}'); // id: 2
const user3 = User.fromObject({ name: "Bob" }); // id: 3
```

---

## 6. Inheritance (extends / super)

### 6.1. Cơ bản

```javascript
// Parent class
class Animal {
  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  speak() {
    return `${this.name} says ${this.sound}`;
  }

  eat(food) {
    return `${this.name} is eating ${food}`;
  }
}

// Child class
class Dog extends Animal {
  constructor(name, breed) {
    // ⚠️ PHẢI gọi super() TRƯỚC khi dùng this
    super(name, "Woof"); // Gọi constructor của parent
    this.breed = breed;
  }

  // Override method
  speak() {
    return `${this.name} barks: ${this.sound}!`;
  }

  // Method mới
  fetch(item) {
    return `${this.name} fetches the ${item}`;
  }
}

class Cat extends Animal {
  constructor(name) {
    super(name, "Meow");
    this.indoor = true;
  }

  // Override + gọi parent method
  speak() {
    const parentResult = super.speak(); // Gọi Animal.speak()
    return `${parentResult} (softly)`;
  }
}
```

### 6.2. `super` keyword

```javascript
class Parent {
  constructor(x) {
    this.x = x;
  }
  method() {
    return "parent";
  }
}

class Child extends Parent {
  constructor(x, y) {
    // ⚠️ Trong constructor: super() PHẢI gọi trước this
    super(x);    // Gọi Parent.constructor(x)
    this.y = y;  // OK — sau super()
  }

  method() {
    // Trong method: super.method() gọi parent method
    const parentResult = super.method();
    return `${parentResult} + child`;
  }
}

// Quy tắc super:
// 1. Trong constructor: super() gọi parent constructor — BẮT BUỘC
// 2. Trong method: super.methodName() gọi parent method — TÙY CHỌN
// 3. super() PHẢI gọi TRƯỚC khi dùng this trong constructor
```

### 6.3. instanceof và Prototype Chain

```javascript
const dog = new Dog("Buddy", "Golden");

// instanceof checks
dog instanceof Dog;    // true
dog instanceof Animal; // true
dog instanceof Object; // true

// Prototype chain
// dog → Dog.prototype → Animal.prototype → Object.prototype → null

// Check methods
dog.speak();  // "Buddy barks: Woof!" (Dog.speak)
dog.eat("bone"); // "Buddy is eating bone" (inherited from Animal)
dog.fetch("ball"); // "Buddy fetches the ball" (Dog.fetch)
```

---

## 7. Private Fields và Methods (ES2022)

### 7.1. Private Fields (#)

```javascript
class BankAccount {
  // Private fields (bắt đầu bằng #)
  #balance = 0;
  #pin;
  #transactions = [];

  constructor(initialBalance, pin) {
    this.#balance = initialBalance;
    this.#pin = pin;
    this.owner = ""; // Public field
  }

  // Private method
  #validatePin(inputPin) {
    return this.#pin === inputPin;
  }

  #recordTransaction(type, amount) {
    this.#transactions.push({
      type,
      amount,
      date: new Date(),
      balance: this.#balance,
    });
  }

  // Public methods
  deposit(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.#balance += amount;
    this.#recordTransaction("deposit", amount);
  }

  withdraw(amount, pin) {
    if (!this.#validatePin(pin)) throw new Error("Invalid PIN");
    if (amount > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= amount;
    this.#recordTransaction("withdrawal", amount);
    return amount;
  }

  // Public getter
  get balance() {
    return this.#balance;
  }

  getStatement(pin) {
    if (!this.#validatePin(pin)) throw new Error("Invalid PIN");
    return [...this.#transactions]; // Return copy
  }
}

const account = new BankAccount(1000, "1234");
account.deposit(500);
console.log(account.balance); // 1500

// ❌ Cannot access private fields
account.#balance;      // SyntaxError
account.#pin;          // SyntaxError
account.#validatePin;  // SyntaxError
```

### 7.2. Static Private

```javascript
class Config {
  static #instance = null;
  #settings = {};

  constructor() {
    if (Config.#instance) {
      throw new Error("Use Config.getInstance()");
    }
  }

  static getInstance() {
    if (!Config.#instance) {
      Config.#instance = new Config();
    }
    return Config.#instance;
  }

  set(key, value) {
    this.#settings[key] = value;
  }

  get(key) {
    return this.#settings[key];
  }
}
```

---

## 8. Mixins Pattern (thay thế multiple inheritance)

JavaScript không hỗ trợ multiple inheritance. Dùng **mixins** để tái sử dụng code.

```javascript
// Mixin functions
const Serializable = (Base) => class extends Base {
  serialize() {
    return JSON.stringify(this);
  }
  
  static deserialize(json) {
    return Object.assign(new this(), JSON.parse(json));
  }
};

const Validatable = (Base) => class extends Base {
  validate() {
    for (const [key, value] of Object.entries(this)) {
      if (value === null || value === undefined) {
        throw new Error(`${key} is required`);
      }
    }
    return true;
  }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date();
    this.updatedAt = new Date();
  }

  touch() {
    this.updatedAt = new Date();
  }
};

// Áp dụng nhiều mixins
class User extends Timestamped(Validatable(Serializable(class {}))) {
  constructor(name, email) {
    super();
    this.name = name;
    this.email = email;
  }
}

const user = new User("John", "j@x.com");
user.validate();           // true
user.serialize();          // JSON string
console.log(user.createdAt); // Date
user.touch();               // Update timestamp
```

---

## 9. Practical Patterns

### 9.1. Builder Pattern

```javascript
class QueryBuilder {
  #table = "";
  #conditions = [];
  #orderBy = null;
  #limit = null;

  from(table) {
    this.#table = table;
    return this; // Fluent interface
  }

  where(condition) {
    this.#conditions.push(condition);
    return this;
  }

  order(field, dir = "ASC") {
    this.#orderBy = `${field} ${dir}`;
    return this;
  }

  take(n) {
    this.#limit = n;
    return this;
  }

  build() {
    let query = `SELECT * FROM ${this.#table}`;
    if (this.#conditions.length) {
      query += ` WHERE ${this.#conditions.join(" AND ")}`;
    }
    if (this.#orderBy) query += ` ORDER BY ${this.#orderBy}`;
    if (this.#limit) query += ` LIMIT ${this.#limit}`;
    return query;
  }
}

const query = new QueryBuilder()
  .from("users")
  .where("age > 18")
  .where("status = 'active'")
  .order("name")
  .take(10)
  .build();
// "SELECT * FROM users WHERE age > 18 AND status = 'active' ORDER BY name ASC LIMIT 10"
```

### 9.2. Observer Pattern

```javascript
class EventEmitter {
  #listeners = new Map();

  on(event, callback) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, []);
    }
    this.#listeners.get(event).push(callback);
    return this;
  }

  off(event, callback) {
    const listeners = this.#listeners.get(event);
    if (listeners) {
      this.#listeners.set(event, listeners.filter(cb => cb !== callback));
    }
    return this;
  }

  emit(event, ...args) {
    const listeners = this.#listeners.get(event) || [];
    listeners.forEach(cb => cb(...args));
    return this;
  }
}

// Usage
class UserService extends EventEmitter {
  createUser(data) {
    const user = { id: Date.now(), ...data };
    this.emit("userCreated", user);
    return user;
  }
}

const service = new UserService();
service.on("userCreated", user => console.log("New user:", user));
service.createUser({ name: "John" });
```

---

## 10. Câu hỏi phỏng vấn

### Q1: Class trong JS có phải "real class" không?
**A**: Không. Class trong JS là syntactic sugar cho prototypal inheritance. Bên dưới vẫn dùng prototype chain.

### Q2: `super()` là gì? Khi nào bắt buộc?
**A**: `super()` gọi constructor của parent class. Bắt buộc trong constructor của child class trước khi dùng `this`. Trong methods, `super.method()` gọi parent method.

### Q3: `static` dùng để làm gì?
**A**: Định nghĩa methods/properties thuộc về **class** (không phải instance). Thường dùng cho utility methods, factory methods, và constants.

---

> Tiếp theo: [Modules →](./09-modules.md)
