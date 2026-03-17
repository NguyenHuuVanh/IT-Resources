# Enhanced Object Literals

> Quay lại [Mục lục](./README.md) | Trước: [Default Parameters](./06-default-parameters.md)

## 📌 Tổng quan

ES6 cải tiến cú pháp object literal (cách tạo object) với 3 tính năng chính:
1. **Property Shorthand** — rút gọn khi tên biến trùng tên property
2. **Method Shorthand** — rút gọn method definition
3. **Computed Property Names** — tên property động

---

## 1. Property Shorthand

Khi tên biến **trùng** với tên property, có thể viết tắt.

```javascript
// ES5 — phải lặp lại tên
var name = "John";
var age = 25;
var city = "NYC";

var person = {
  name: name,
  age: age,
  city: city,
};

// ES6 — Property shorthand
const name = "John";
const age = 25;
const city = "NYC";

const person = { name, age, city };
// Tương đương: { name: "John", age: 25, city: "NYC" }

// Mixed: shorthand + regular
const email = "john@example.com";
const user = {
  name: "John",   // Regular property
  age,             // Shorthand
  email,           // Shorthand
  role: "admin",   // Regular property
};
```

### Practical Use Cases

```javascript
// ✅ Function return
function getUser(id) {
  const name = "John";
  const email = "j@x.com";
  return { id, name, email }; // Shorthand
}

// ✅ Module exports
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;
module.exports = { add, subtract };

// ✅ React state
const [count, setCount] = useState(0);
const [name, setName] = useState("");
// { count, setCount, name, setName }

// ✅ Destructure → transform → re-construct
function transformUser(user) {
  const { name, email } = user;
  const upperName = name.toUpperCase();
  return { name: upperName, email }; // Partial shorthand
}
```

---

## 2. Method Shorthand

```javascript
// ES5 — method là property + function expression
var calculator = {
  add: function(a, b) {
    return a + b;
  },
  subtract: function(a, b) {
    return a - b;
  },
};

// ES6 — Method shorthand (bỏ `: function`)
const calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  },
};

// Async methods
const api = {
  async fetchData(url) {
    const response = await fetch(url);
    return response.json();
  },
  async postData(url, data) {
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });
    return response.json();
  },
};

// Generator methods
const collection = {
  items: [1, 2, 3],
  *[Symbol.iterator]() {
    yield* this.items;
  },
  *range(start, end) {
    for (let i = start; i <= end; i++) {
      yield i;
    }
  },
};

// Getter và Setter
const person = {
  _name: "",
  get name() {
    return this._name.toUpperCase();
  },
  set name(value) {
    this._name = value.trim();
  },
};

person.name = "  John  ";
console.log(person.name); // "JOHN"
```

### ⚠️ Method Shorthand vs Arrow Function trong Object

```javascript
const obj = {
  value: 42,
  
  // ✅ Method shorthand — this = obj
  method() {
    return this.value; // 42
  },
  
  // ❌ Arrow function — this = enclosing scope (KHÔNG phải obj)
  arrowMethod: () => {
    return this.value; // undefined
  },
  
  // ✅ Regular function — this = obj (nhưng dài hơn)
  regularMethod: function() {
    return this.value; // 42
  },
};
```

---

## 3. Computed Property Names

Cho phép sử dụng **biểu thức** làm tên property (wrapped trong `[]`).

```javascript
// ES5 — phải gán sau khi tạo object
var prop = "name";
var obj = {};
obj[prop] = "John";

// ES6 — Computed property names
const prop = "name";
const obj = {
  [prop]: "John",
};
// { name: "John" }
```

### 3.1. Dynamic Keys

```javascript
const prefix = "user";
const user = {
  [`${prefix}Name`]: "John",
  [`${prefix}Age`]: 25,
  [`${prefix}Email`]: "john@example.com",
};
// { userName: "John", userAge: 25, userEmail: "john@example.com" }

// Index-based keys
const items = {};
for (let i = 0; i < 3; i++) {
  items[`item${i}`] = `Value ${i}`;
}
// { item0: "Value 0", item1: "Value 1", item2: "Value 2" }
```

### 3.2. Symbol as Key

```javascript
const id = Symbol("id");
const secret = Symbol("secret");

const user = {
  [id]: 12345,
  [secret]: "classified",
  name: "John",
};

console.log(user[id]);     // 12345
console.log(user[secret]); // "classified"

// Symbol keys không xuất hiện trong Object.keys()
console.log(Object.keys(user)); // ["name"]
```

### 3.3. Enum-like Pattern

```javascript
const STATUS = {
  PENDING: "pending",
  ACTIVE: "active",
  INACTIVE: "inactive",
};

const statusColors = {
  [STATUS.PENDING]: "yellow",
  [STATUS.ACTIVE]: "green",
  [STATUS.INACTIVE]: "gray",
};

console.log(statusColors[STATUS.ACTIVE]); // "green"
```

---

## 4. Practical Examples

### 4.1. Redux Action Creator

```javascript
const createAction = (type) => (payload) => ({ type, payload });

const addTodo = createAction("ADD_TODO");
addTodo({ text: "Learn ES6" });
// { type: "ADD_TODO", payload: { text: "Learn ES6" } }

// Reducer with computed keys
const createReducer = (handlers, defaultState) => {
  return (state = defaultState, action) => {
    if (handlers[action.type]) {
      return handlers[action.type](state, action);
    }
    return state;
  };
};
```

### 4.2. Dynamic Object Creation

```javascript
function createObject(keys, values) {
  return keys.reduce(
    (obj, key, i) => ({
      ...obj,
      [key]: values[i],
    }),
    {}
  );
}

createObject(["a", "b", "c"], [1, 2, 3]);
// { a: 1, b: 2, c: 3 }

// Form data to object
function formToObject(formData) {
  return [...formData.entries()].reduce(
    (obj, [key, value]) => ({ ...obj, [key]: value }),
    {}
  );
}
```

### 4.3. State Management

```javascript
const setState = (state, key, value) => ({
  ...state,
  [key]: value,
});

let state = { count: 0, name: "John" };
state = setState(state, "count", 1);    // { count: 1, name: "John" }
state = setState(state, "name", "Jane"); // { count: 1, name: "Jane" }

// Multi-key update
const setMultiple = (state, updates) => ({ ...state, ...updates });
state = setMultiple(state, { count: 5, loading: true });
```

### 4.4. API Response Mapper

```javascript
const fieldMap = {
  first_name: "firstName",
  last_name: "lastName",
  email_address: "email",
};

function mapFields(data, mapping) {
  return Object.entries(data).reduce(
    (result, [key, value]) => ({
      ...result,
      [mapping[key] || key]: value,
    }),
    {}
  );
}

mapFields(
  { first_name: "John", last_name: "Doe", email_address: "j@x.com" },
  fieldMap
);
// { firstName: "John", lastName: "Doe", email: "j@x.com" }
```

---

## 5. Kết hợp tất cả tính năng

```javascript
// Complete example — tất cả enhanced object literal features
const createStore = (initialState = {}) => {
  let state = { ...initialState };
  const listeners = [];

  return {
    // Method shorthand
    getState() {
      return { ...state };
    },

    // Computed + method shorthand
    setState(key, value) {
      state = { ...state, [key]: value }; // Computed property
      listeners.forEach(fn => fn(state));
    },

    // Method shorthand
    subscribe(listener) {
      listeners.push(listener);
      return () => {
        const index = listeners.indexOf(listener);
        if (index > -1) listeners.splice(index, 1);
      };
    },
  };
};

const store = createStore({ count: 0, theme: "dark" });
store.subscribe(state => console.log("Updated:", state));
store.setState("count", 1);
// Updated: { count: 1, theme: "dark" }
```

---

> Tiếp theo: [Classes →](./08-classes.md)
