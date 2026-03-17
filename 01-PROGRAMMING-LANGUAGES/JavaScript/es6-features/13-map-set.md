# Map & Set

> Quay lại [Mục lục](./README.md) | Trước: [Iterators & Generators](./12-iterators-generators.md)

## 📌 Tổng quan

ES6 giới thiệu 4 collections mới:
- **Map**: Key-value pairs với key là **bất kỳ kiểu nào**
- **Set**: Tập hợp giá trị **không trùng lặp**
- **WeakMap**: Map với keys là **objects only**, cho phép garbage collection
- **WeakSet**: Set với values là **objects only**, cho phép garbage collection

---

## 1. Map

### 1.1. Tạo và sử dụng Map

```javascript
// Tạo Map rỗng
const map = new Map();

// Set key-value pairs
map.set("name", "John");
map.set("age", 25);
map.set(true, "boolean key");
map.set(42, "number key");
map.set(null, "null key");

// Object as key (chỉ Map làm được!)
const objKey = { id: 1 };
map.set(objKey, "object value");

// Function as key
const fnKey = () => {};
map.set(fnKey, "function value");

// Get values
console.log(map.get("name"));    // "John"
console.log(map.get(objKey));    // "object value"
console.log(map.get("unknown")); // undefined

// Check existence
console.log(map.has("name")); // true
console.log(map.has("xyz"));  // false

// Size
console.log(map.size); // 6

// Delete
map.delete("age");     // true (deleted)
map.delete("xyz");     // false (not found)

// Clear all
map.clear();
console.log(map.size); // 0
```

### 1.2. Initialize với entries

```javascript
const map = new Map([
  ["key1", "value1"],
  ["key2", "value2"],
  ["key3", "value3"],
]);

// Từ Object
const obj = { a: 1, b: 2, c: 3 };
const fromObj = new Map(Object.entries(obj));

// Ngược lại: Map → Object
const backToObj = Object.fromEntries(fromObj);
```

### 1.3. Iteration

```javascript
const map = new Map([
  ["name", "John"],
  ["age", 25],
  ["city", "NYC"],
]);

// for...of (default: entries)
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// forEach
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

// Keys, Values, Entries
console.log([...map.keys()]);    // ["name", "age", "city"]
console.log([...map.values()]);  // ["John", 25, "NYC"]
console.log([...map.entries()]); // [["name", "John"], ["age", 25], ...]

// ⚠️ Map giữ nguyên insertion order
```

### 1.4. Chaining

```javascript
const map = new Map()
  .set("a", 1)
  .set("b", 2)
  .set("c", 3);  // set() returns the Map → chainable
```

---

## 2. Map vs Object

```
┌────────────────────┬──────────────────────┬──────────────────────┐
│ Feature            │ Map                  │ Object               │
├────────────────────┼──────────────────────┼──────────────────────┤
│ Key types          │ Any (obj, fn, null)  │ String / Symbol only │
│ Key order          │ Insertion order      │ Complex rules¹       │
│ Size               │ map.size             │ Object.keys().length │
│ Iteration          │ Directly iterable    │ Need Object.keys()   │
│ Performance        │ Better for frequent  │ Better for static    │
│                    │ add/delete           │ property access      │
│ Default keys       │ None                 │ Has prototype keys   │
│ Serialization      │ No native JSON       │ JSON.stringify ✅    │
│ Destructuring      │ ❌                   │ ✅ { a, b } = obj   │
│ Syntax sugar       │ map.get(k)           │ obj.key or obj[key]  │
└────────────────────┴──────────────────────┴──────────────────────┘

¹ Object key order: integers first (sorted), then strings (insertion),
  then symbols (insertion)
```

```javascript
// Object CAN'T use object as key
const obj = {};
const key1 = { id: 1 };
const key2 = { id: 2 };

obj[key1] = "first";
obj[key2] = "second";  // ⚠️ key1 và key2 cùng convert thành "[object Object]"

console.log(obj[key1]); // "second" ← Bị ghi đè!
console.log(Object.keys(obj)); // ["[object Object]"]

// Map CAN use any key
const map = new Map();
map.set(key1, "first");
map.set(key2, "second"); // ✅ key1 và key2 là hai entries khác nhau

console.log(map.get(key1)); // "first"
console.log(map.get(key2)); // "second"
```

### Khi nào dùng Map vs Object?

```javascript
// ✅ Dùng Map khi:
// - Keys không phải string (object keys, dynamic keys)
// - Cần duy trì insertion order
// - Thường xuyên add/delete entries
// - Cần biết size nhanh
// - Cần iterate

// ✅ Dùng Object khi:
// - Keys là string, cấu trúc cố định
// - Cần JSON serialization
// - Cần destructuring
// - Cần property access sugar (obj.prop)
// - Working với APIs trả về plain objects
```

---

## 3. WeakMap

**WeakMap** giống Map nhưng:
- Keys **PHẢI** là objects (không phải primitives)
- Keys là **weak references** → cho phép garbage collection
- **Không iterable** (không có size, keys(), values(), forEach)

```javascript
const weakMap = new WeakMap();

let obj = { name: "John" };
weakMap.set(obj, "metadata");

console.log(weakMap.get(obj)); // "metadata"
console.log(weakMap.has(obj)); // true

// Khi obj bị garbage collected, entry trong WeakMap cũng tự động bị xóa
obj = null;
// Entry { name: "John" } → "metadata" sẽ bị GC thu hồi

// ❌ Không thể
weakMap.set("string", "value");  // TypeError: Invalid value used as weak map key
weakMap.size;                     // undefined
weakMap.keys();                   // TypeError
[...weakMap];                     // TypeError
```

### Use Cases

```javascript
// 1. Private data
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age });
  }
  getName() {
    return privateData.get(this).name;
  }
  getAge() {
    return privateData.get(this).age;
  }
}

const john = new Person("John", 25);
john.getName(); // "John"
// privateData không accessible từ bên ngoài
// Khi john bị GC, private data cũng bị GC

// 2. Caching / Memoization
const cache = new WeakMap();

function computeExpensive(obj) {
  if (cache.has(obj)) {
    return cache.get(obj); // Cache hit
  }
  const result = /* expensive computation */ obj.data.length * 100;
  cache.set(obj, result);
  return result;
}
// Khi obj bị GC, cache entry cũng tự động xóa → no memory leak

// 3. DOM element metadata
const elementData = new WeakMap();

function trackElement(element) {
  elementData.set(element, {
    clicks: 0,
    lastInteraction: null,
  });
}

// Khi DOM element bị remove, data cũng tự động GC
```

---

## 4. Set

### 4.1. Tạo và sử dụng Set

```javascript
// Tạo Set
const set = new Set();

// Add values (duplicates ignored)
set.add(1);
set.add(2);
set.add(2);      // Duplicate — bị bỏ qua
set.add("hello");
set.add(true);

console.log(set.size); // 4 (không phải 5)

// Check existence
console.log(set.has(1));     // true
console.log(set.has("bye")); // false

// Delete
set.delete(2);       // true
set.delete("xyz");   // false

// Clear
set.clear();

// Initialize với values
const set2 = new Set([1, 2, 3, 3, 4, 4, 5]);
console.log([...set2]); // [1, 2, 3, 4, 5]
```

### 4.2. Remove Duplicates (use case phổ biến nhất)

```javascript
// Array → unique
const arr = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4];
const unique = [...new Set(arr)];
// [1, 2, 3, 4]

// String unique characters
const uniqueChars = [...new Set("abracadabra")].join("");
// "abrcd"

// Remove duplicate objects (by reference)
// ⚠️ Set dùng SameValueZero comparison (giống ===, nhưng NaN === NaN)
const a = { id: 1 };
const b = { id: 1 };
const set = new Set([a, b]);
console.log(set.size); // 2 (a !== b, dù cùng nội dung)

const set2 = new Set([a, a]);
console.log(set2.size); // 1 (cùng reference)

// Unique objects by property
function uniqueBy(arr, key) {
  const seen = new Set();
  return arr.filter(item => {
    const val = item[key];
    if (seen.has(val)) return false;
    seen.add(val);
    return true;
  });
}
```

### 4.3. Iteration

```javascript
const set = new Set(["a", "b", "c"]);

for (const value of set) {
  console.log(value);
}

set.forEach(value => console.log(value));

// Convert to array
const arr = [...set];
const arr2 = Array.from(set);
```

---

## 5. Set Operations

```javascript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// Union (A ∪ B) — tất cả phần tử
const union = new Set([...setA, ...setB]);
// {1, 2, 3, 4, 5, 6}

// Intersection (A ∩ B) — phần tử chung
const intersection = new Set([...setA].filter(x => setB.has(x)));
// {3, 4}

// Difference (A - B) — chỉ trong A
const difference = new Set([...setA].filter(x => !setB.has(x)));
// {1, 2}

// Symmetric Difference (A △ B) — không ở cả hai
const symDiff = new Set(
  [...setA].filter(x => !setB.has(x)).concat(
    [...setB].filter(x => !setA.has(x))
  )
);
// {1, 2, 5, 6}

// Subset check (A ⊆ B)
const isSubset = [...setA].every(x => setB.has(x)); // false

// Superset check (A ⊇ B)
const isSuperset = [...setB].every(x => setA.has(x)); // false

// Disjoint check (A ∩ B = ∅)
const isDisjoint = ![...setA].some(x => setB.has(x)); // false

// ✅ ES2025 sẽ có built-in methods:
// setA.union(setB)
// setA.intersection(setB)  
// setA.difference(setB)
// setA.symmetricDifference(setB)
// setA.isSubsetOf(setB)
```

---

## 6. WeakSet

```javascript
// WeakSet — chỉ objects, garbage collected, không iterable
const weakSet = new WeakSet();

let obj = { name: "John" };
weakSet.add(obj);
weakSet.has(obj); // true

obj = null;
// obj bị GC → entry trong WeakSet cũng bị xóa

// ❌ Không thể
weakSet.add("string"); // TypeError
weakSet.size;           // undefined
[...weakSet];           // TypeError
```

### Use Cases

```javascript
// 1. Track visited/processed objects (avoid infinite loops)
const visited = new WeakSet();

function deepClone(obj) {
  if (visited.has(obj)) return obj; // Circular reference protection
  visited.add(obj);
  
  const clone = Array.isArray(obj) ? [] : {};
  for (const [key, value] of Object.entries(obj)) {
    clone[key] = typeof value === "object" && value !== null
      ? deepClone(value)
      : value;
  }
  return clone;
}

// 2. Instance validation
const validInstances = new WeakSet();

class SecureToken {
  constructor(value) {
    this.value = value;
    validInstances.add(this);
  }

  static isValid(token) {
    return validInstances.has(token);
  }
}

const token = new SecureToken("abc");
SecureToken.isValid(token);      // true
SecureToken.isValid({ value: "abc" }); // false (fake object)

// 3. Brand checking
const branded = new WeakSet();

class SafeClass {
  constructor() {
    branded.add(this);
  }

  doSomething() {
    if (!branded.has(this)) {
      throw new Error("Invalid instance");
    }
    // ... safe operation
  }
}
```

---

## 7. Comparison Table

```
┌────────────────┬──────────┬──────────┬──────────┬──────────┐
│ Feature        │ Map      │ WeakMap  │ Set      │ WeakSet  │
├────────────────┼──────────┼──────────┼──────────┼──────────┤
│ Key/Value type │ Any      │ Obj keys │ Any      │ Obj only │
│ Iterable       │ ✅       │ ❌       │ ✅       │ ❌       │
│ .size          │ ✅       │ ❌       │ ✅       │ ❌       │
│ GC-friendly    │ ❌       │ ✅       │ ❌       │ ✅       │
│ forEach        │ ✅       │ ❌       │ ✅       │ ❌       │
│ clear()        │ ✅       │ ❌       │ ✅       │ ❌       │
│ JSON serialize │ Manual   │ ❌       │ Manual   │ ❌       │
└────────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

> Tiếp theo: [for...of Loop →](./14-for-of-loop.md)
