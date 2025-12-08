# JavaScript Array & Object Methods - Hướng dẫn toàn diện

## 📌 Mục lục

### Array Methods

1. [Mutating Methods](#1-mutating-methods-thay-đổi-array-gốc)
2. [Non-Mutating Methods](#2-non-mutating-methods-không-thay-đổi-array-gốc)
3. [Iteration Methods](#3-iteration-methods)
4. [Search Methods](#4-search-methods)
5. [Transform Methods](#5-transform-methods)
6. [Static Methods](#6-array-static-methods)

### Object Methods

7. [Object Static Methods](#7-object-static-methods)
8. [Object Instance Methods](#8-object-instance-methods)
9. [Property Descriptors](#9-property-descriptors)

### Advanced

10. [Chaining Methods](#10-chaining-methods)
11. [Performance Tips](#11-performance-tips)
12. [Common Patterns](#12-common-patterns)

---

# ARRAY METHODS

## 1. Mutating Methods (Thay đổi Array gốc)

### ⚠️ Lưu ý quan trọng

Các methods này **THAY ĐỔI** array gốc. Cẩn thận khi dùng với React/Vue state!

---

### 📌 push() - Thêm vào cuối

```javascript
const arr = [1, 2, 3];
const newLength = arr.push(4, 5);

console.log(arr); // [1, 2, 3, 4, 5]
console.log(newLength); // 5 (trả về length mới)
```

| Đặc điểm        | Giá trị    |
| --------------- | ---------- |
| Thay đổi gốc    | ✅ Có      |
| Return          | Độ dài mới |
| Time Complexity | O(1)       |

**Khi nào dùng:** Thêm phần tử vào cuối array.

---

### 📌 pop() - Xóa từ cuối

```javascript
const arr = [1, 2, 3];
const removed = arr.pop();

console.log(arr); // [1, 2]
console.log(removed); // 3 (phần tử bị xóa)
```

| Đặc điểm        | Giá trị        |
| --------------- | -------------- |
| Thay đổi gốc    | ✅ Có          |
| Return          | Phần tử bị xóa |
| Time Complexity | O(1)           |

**Khi nào dùng:** Implement Stack (LIFO).

---

### 📌 unshift() - Thêm vào đầu

```javascript
const arr = [2, 3, 4];
const newLength = arr.unshift(0, 1);

console.log(arr); // [0, 1, 2, 3, 4]
console.log(newLength); // 5
```

| Đặc điểm        | Giá trị                           |
| --------------- | --------------------------------- |
| Thay đổi gốc    | ✅ Có                             |
| Return          | Độ dài mới                        |
| Time Complexity | O(n) - phải shift tất cả elements |

**⚠️ Chậm với array lớn!**

---

### 📌 shift() - Xóa từ đầu

```javascript
const arr = [1, 2, 3];
const removed = arr.shift();

console.log(arr); // [2, 3]
console.log(removed); // 1
```

| Đặc điểm        | Giá trị        |
| --------------- | -------------- |
| Thay đổi gốc    | ✅ Có          |
| Return          | Phần tử bị xóa |
| Time Complexity | O(n)           |

**Khi nào dùng:** Implement Queue (FIFO).

---

### 📌 splice() - Thêm/Xóa/Thay thế

```javascript
const arr = [1, 2, 3, 4, 5];

// splice(start, deleteCount, ...items)

// Xóa 2 phần tử từ index 1
arr.splice(1, 2);
console.log(arr); // [1, 4, 5]

// Thêm phần tử tại index 1
arr.splice(1, 0, "a", "b");
console.log(arr); // [1, 'a', 'b', 4, 5]

// Thay thế 1 phần tử tại index 2
arr.splice(2, 1, "NEW");
console.log(arr); // [1, 'a', 'NEW', 4, 5]
```

| Đặc điểm        | Giá trị                  |
| --------------- | ------------------------ |
| Thay đổi gốc    | ✅ Có                    |
| Return          | Array các phần tử bị xóa |
| Time Complexity | O(n)                     |

**Khi nào dùng:** Cần thêm/xóa/thay thế tại vị trí cụ thể.

---

### 📌 sort() - Sắp xếp

```javascript
// ⚠️ Mặc định sort theo STRING!
const nums = [10, 2, 30, 1];
nums.sort();
console.log(nums); // [1, 10, 2, 30] - SAI!

// ✅ Sort số đúng cách
nums.sort((a, b) => a - b); // Tăng dần
console.log(nums); // [1, 2, 10, 30]

nums.sort((a, b) => b - a); // Giảm dần
console.log(nums); // [30, 10, 2, 1]

// Sort objects
const users = [
  { name: "John", age: 30 },
  { name: "Jane", age: 25 },
  { name: "Bob", age: 35 },
];

users.sort((a, b) => a.age - b.age);
// [{ name: 'Jane', age: 25 }, { name: 'John', age: 30 }, { name: 'Bob', age: 35 }]

// Sort strings (có dấu tiếng Việt)
const names = ["Ánh", "An", "Bình", "Ân"];
names.sort((a, b) => a.localeCompare(b, "vi"));
// ['An', 'Ân', 'Ánh', 'Bình']
```

| Đặc điểm        | Giá trị                        |
| --------------- | ------------------------------ |
| Thay đổi gốc    | ✅ Có                          |
| Return          | Array đã sort (cùng reference) |
| Time Complexity | O(n log n)                     |

---

### 📌 reverse() - Đảo ngược

```javascript
const arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr); // [5, 4, 3, 2, 1]
```

| Đặc điểm        | Giá trị                       |
| --------------- | ----------------------------- |
| Thay đổi gốc    | ✅ Có                         |
| Return          | Array đã đảo (cùng reference) |
| Time Complexity | O(n)                          |

---

### 📌 fill() - Điền giá trị

```javascript
const arr = [1, 2, 3, 4, 5];

// fill(value, start, end)
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

arr.fill(7, 1, 3);
console.log(arr); // [0, 7, 7, 0, 0]

// Tạo array với giá trị mặc định
const zeros = new Array(5).fill(0);
console.log(zeros); // [0, 0, 0, 0, 0]

// ⚠️ Cẩn thận với objects!
const matrix = new Array(3).fill([]); // SAI - cùng reference
matrix[0].push(1);
console.log(matrix); // [[1], [1], [1]] - Tất cả đều bị ảnh hưởng!

// ✅ Đúng cách
const matrix2 = Array.from({ length: 3 }, () => []);
matrix2[0].push(1);
console.log(matrix2); // [[1], [], []]
```

---

### 📌 copyWithin() - Copy trong array

```javascript
const arr = [1, 2, 3, 4, 5];

// copyWithin(target, start, end)
arr.copyWithin(0, 3); // Copy từ index 3 đến cuối, paste tại index 0
console.log(arr); // [4, 5, 3, 4, 5]

const arr2 = [1, 2, 3, 4, 5];
arr2.copyWithin(1, 3, 4); // Copy index 3-4, paste tại index 1
console.log(arr2); // [1, 4, 3, 4, 5]
```

---

## 2. Non-Mutating Methods (Không thay đổi Array gốc)

### 📌 slice() - Cắt array

```javascript
const arr = [1, 2, 3, 4, 5];

// slice(start, end) - end không bao gồm
const sliced = arr.slice(1, 4);
console.log(sliced); // [2, 3, 4]
console.log(arr); // [1, 2, 3, 4, 5] - không đổi

// Negative index
arr.slice(-2); // [4, 5] - 2 phần tử cuối
arr.slice(-3, -1); // [3, 4]

// Clone array
const clone = arr.slice();
// hoặc: const clone = [...arr];
```

| Đặc điểm        | Giá trị   |
| --------------- | --------- |
| Thay đổi gốc    | ❌ Không  |
| Return          | Array mới |
| Time Complexity | O(n)      |

**Khi nào dùng:** Lấy một phần của array, clone array.

---

### 📌 concat() - Nối arrays

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const arr3 = [5, 6];

const combined = arr1.concat(arr2, arr3);
console.log(combined); // [1, 2, 3, 4, 5, 6]
console.log(arr1); // [1, 2] - không đổi

// Có thể concat với values
const result = arr1.concat(3, 4, [5, 6]);
console.log(result); // [1, 2, 3, 4, 5, 6]

// Modern: Spread operator
const combined2 = [...arr1, ...arr2, ...arr3];
```

---

### 📌 join() - Chuyển thành string

```javascript
const arr = ["Hello", "World", "!"];

console.log(arr.join()); // "Hello,World,!" (default: comma)
console.log(arr.join(" ")); // "Hello World !"
console.log(arr.join("-")); // "Hello-World-!"
console.log(arr.join("")); // "HelloWorld!"

// Useful patterns
const path = ["users", "john", "profile"].join("/");
// "users/john/profile"

const csv = [
  ["Name", "Age"],
  ["John", 30],
  ["Jane", 25],
]
  .map((row) => row.join(","))
  .join("\n");
// "Name,Age\nJohn,30\nJane,25"
```

---

### 📌 flat() - Làm phẳng array

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat()); // [1, 2, 3, 4, [5, 6]] - depth 1
console.log(nested.flat(2)); // [1, 2, 3, 4, 5, 6] - depth 2
console.log(nested.flat(Infinity)); // Flatten hoàn toàn

// Loại bỏ empty slots
const sparse = [1, , 3, , 5];
console.log(sparse.flat()); // [1, 3, 5]
```

---

### 📌 flatMap() - Map + Flat

```javascript
const arr = [1, 2, 3];

// Tương đương arr.map(...).flat()
const result = arr.flatMap((x) => [x, x * 2]);
console.log(result); // [1, 2, 2, 4, 3, 6]

// Useful: Tách words
const sentences = ["Hello World", "How are you"];
const words = sentences.flatMap((s) => s.split(" "));
console.log(words); // ["Hello", "World", "How", "are", "you"]

// Filter + Map trong 1 bước
const nums = [1, 2, 3, 4, 5];
const doubled = nums.flatMap((n) => (n > 2 ? [n * 2] : []));
console.log(doubled); // [6, 8, 10]
```

---

### 📌 toSorted(), toReversed(), toSpliced() (ES2023)

```javascript
// Non-mutating versions của sort, reverse, splice
const arr = [3, 1, 2];

const sorted = arr.toSorted((a, b) => a - b);
console.log(sorted); // [1, 2, 3]
console.log(arr); // [3, 1, 2] - không đổi!

const reversed = arr.toReversed();
console.log(reversed); // [2, 1, 3]
console.log(arr); // [3, 1, 2] - không đổi!

const spliced = arr.toSpliced(1, 1, "new");
console.log(spliced); // [3, 'new', 2]
console.log(arr); // [3, 1, 2] - không đổi!
```

---

### 📌 with() (ES2023) - Thay thế tại index

```javascript
const arr = [1, 2, 3, 4, 5];

const newArr = arr.with(2, "three");
console.log(newArr); // [1, 2, 'three', 4, 5]
console.log(arr); // [1, 2, 3, 4, 5] - không đổi!

// Negative index
const newArr2 = arr.with(-1, "last");
console.log(newArr2); // [1, 2, 3, 4, 'last']
```

---

## 3. Iteration Methods

### 📌 forEach() - Duyệt qua từng phần tử

```javascript
const arr = [1, 2, 3];

arr.forEach((value, index, array) => {
  console.log(`Index ${index}: ${value}`);
});
// Index 0: 1
// Index 1: 2
// Index 2: 3

// ⚠️ Không thể break/return sớm
// ⚠️ Không return giá trị (luôn undefined)
// ⚠️ Không nên dùng với async/await
```

| Đặc điểm       | Giá trị      |
| -------------- | ------------ |
| Return         | undefined    |
| Break/Continue | ❌ Không thể |
| Async support  | ❌ Không tốt |

**Khi nào dùng:** Side effects đơn giản (log, DOM manipulation).
**Khi KHÔNG dùng:** Cần return value, cần break sớm, async operations.

---

### 📌 map() - Biến đổi từng phần tử

```javascript
const nums = [1, 2, 3, 4, 5];

const doubled = nums.map((n) => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Với index
const indexed = nums.map((n, i) => `${i}: ${n}`);
console.log(indexed); // ['0: 1', '1: 2', '2: 3', '3: 4', '4: 5']

// Transform objects
const users = [
  { firstName: "John", lastName: "Doe" },
  { firstName: "Jane", lastName: "Smith" },
];

const fullNames = users.map((u) => `${u.firstName} ${u.lastName}`);
console.log(fullNames); // ['John Doe', 'Jane Smith']

// Extract property
const firstNames = users.map((u) => u.firstName);
console.log(firstNames); // ['John', 'Jane']
```

| Đặc điểm        | Giá trị               |
| --------------- | --------------------- |
| Thay đổi gốc    | ❌ Không              |
| Return          | Array mới cùng length |
| Time Complexity | O(n)                  |

**Khi nào dùng:** Biến đổi mỗi phần tử thành giá trị mới.

---

### 📌 filter() - Lọc phần tử

```javascript
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const evens = nums.filter((n) => n % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

const greaterThan5 = nums.filter((n) => n > 5);
console.log(greaterThan5); // [6, 7, 8, 9, 10]

// Filter objects
const products = [
  { name: "Phone", price: 999, inStock: true },
  { name: "Laptop", price: 1999, inStock: false },
  { name: "Tablet", price: 599, inStock: true },
];

const available = products.filter((p) => p.inStock);
const affordable = products.filter((p) => p.price < 1000);

// Remove falsy values
const mixed = [0, 1, "", "hello", null, undefined, false, true];
const truthy = mixed.filter(Boolean);
console.log(truthy); // [1, 'hello', true]

// Remove duplicates (simple)
const duplicates = [1, 2, 2, 3, 3, 3, 4];
const unique = duplicates.filter((v, i, arr) => arr.indexOf(v) === i);
console.log(unique); // [1, 2, 3, 4]
// Better: [...new Set(duplicates)]
```

| Đặc điểm        | Giá trị                     |
| --------------- | --------------------------- |
| Thay đổi gốc    | ❌ Không                    |
| Return          | Array mới (có thể ngắn hơn) |
| Time Complexity | O(n)                        |

**Khi nào dùng:** Lọc phần tử theo điều kiện.

---

### 📌 reduce() - Gộp thành một giá trị

```javascript
const nums = [1, 2, 3, 4, 5];

// reduce(callback, initialValue)
// callback(accumulator, currentValue, index, array)

// Tính tổng
const sum = nums.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 15

// Tính tích
const product = nums.reduce((acc, curr) => acc * curr, 1);
console.log(product); // 120

// Tìm max
const max = nums.reduce((acc, curr) => Math.max(acc, curr), -Infinity);
console.log(max); // 5

// Đếm occurrences
const fruits = ["apple", "banana", "apple", "orange", "banana", "apple"];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
console.log(count); // { apple: 3, banana: 2, orange: 1 }

// Group by property
const people = [
  { name: "John", age: 25 },
  { name: "Jane", age: 30 },
  { name: "Bob", age: 25 },
];

const groupedByAge = people.reduce((acc, person) => {
  const key = person.age;
  acc[key] = acc[key] || [];
  acc[key].push(person);
  return acc;
}, {});
// { 25: [{name: 'John'...}, {name: 'Bob'...}], 30: [{name: 'Jane'...}] }

// Flatten array
const nested = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const flat = nested.reduce((acc, curr) => acc.concat(curr), []);
console.log(flat); // [1, 2, 3, 4, 5, 6]

// Pipe functions
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((v, f) => f(v), x);
const add1 = (x) => x + 1;
const double = (x) => x * 2;
const square = (x) => x * x;

const compute = pipe(add1, double, square);
console.log(compute(2)); // ((2 + 1) * 2)² = 36
```

| Đặc điểm        | Giá trị            |
| --------------- | ------------------ |
| Thay đổi gốc    | ❌ Không           |
| Return          | Bất kỳ giá trị nào |
| Time Complexity | O(n)               |

**Khi nào dùng:** Gộp array thành một giá trị (sum, object, array khác).

---

### 📌 reduceRight() - Reduce từ phải sang trái

```javascript
const arr = [
  [1, 2],
  [3, 4],
  [5, 6],
];

const flatLeft = arr.reduce((acc, curr) => acc.concat(curr), []);
console.log(flatLeft); // [1, 2, 3, 4, 5, 6]

const flatRight = arr.reduceRight((acc, curr) => acc.concat(curr), []);
console.log(flatRight); // [5, 6, 3, 4, 1, 2]

// Compose functions (ngược với pipe)
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((v, f) => f(v), x);
```

---

### 📌 every() - Kiểm tra TẤT CẢ

```javascript
const nums = [2, 4, 6, 8, 10];

const allEven = nums.every((n) => n % 2 === 0);
console.log(allEven); // true

const allPositive = nums.every((n) => n > 0);
console.log(allPositive); // true

const allGreaterThan5 = nums.every((n) => n > 5);
console.log(allGreaterThan5); // false

// Empty array returns true
[].every((x) => x > 0); // true (vacuous truth)

// Short-circuit: Dừng ngay khi gặp false
const arr = [1, 2, 3, 4, 5];
arr.every((n) => {
  console.log(n);
  return n < 3;
});
// Logs: 1, 2, 3 (dừng tại 3)
```

| Đặc điểm      | Giá trị                |
| ------------- | ---------------------- |
| Return        | boolean                |
| Short-circuit | ✅ Có (dừng khi false) |

**Khi nào dùng:** Kiểm tra tất cả phần tử thỏa điều kiện.

---

### 📌 some() - Kiểm tra CÓ ÍT NHẤT MỘT

```javascript
const nums = [1, 3, 5, 7, 8, 9];

const hasEven = nums.some((n) => n % 2 === 0);
console.log(hasEven); // true (có 8)

const hasNegative = nums.some((n) => n < 0);
console.log(hasNegative); // false

// Empty array returns false
[].some((x) => x > 0); // false

// Short-circuit: Dừng ngay khi gặp true
const arr = [1, 2, 3, 4, 5];
arr.some((n) => {
  console.log(n);
  return n > 2;
});
// Logs: 1, 2, 3 (dừng tại 3)
```

| Đặc điểm      | Giá trị               |
| ------------- | --------------------- |
| Return        | boolean               |
| Short-circuit | ✅ Có (dừng khi true) |

**Khi nào dùng:** Kiểm tra có ít nhất một phần tử thỏa điều kiện.

---

## 4. Search Methods

### 📌 find() - Tìm phần tử đầu tiên

```javascript
const users = [
  { id: 1, name: "John", age: 25 },
  { id: 2, name: "Jane", age: 30 },
  { id: 3, name: "Bob", age: 25 },
];

const user = users.find((u) => u.id === 2);
console.log(user); // { id: 2, name: 'Jane', age: 30 }

const notFound = users.find((u) => u.id === 999);
console.log(notFound); // undefined

// Tìm số chẵn đầu tiên
const nums = [1, 3, 5, 6, 7, 8];
const firstEven = nums.find((n) => n % 2 === 0);
console.log(firstEven); // 6
```

| Đặc điểm      | Giá trị                |
| ------------- | ---------------------- |
| Return        | Phần tử hoặc undefined |
| Short-circuit | ✅ Có                  |

---

### 📌 findIndex() - Tìm index đầu tiên

```javascript
const users = [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" },
  { id: 3, name: "Bob" },
];

const index = users.findIndex((u) => u.id === 2);
console.log(index); // 1

const notFound = users.findIndex((u) => u.id === 999);
console.log(notFound); // -1
```

---

### 📌 findLast() & findLastIndex() (ES2023)

```javascript
const nums = [1, 2, 3, 4, 5, 4, 3, 2, 1];

const lastGreaterThan3 = nums.findLast((n) => n > 3);
console.log(lastGreaterThan3); // 4 (index 5)

const lastIndex = nums.findLastIndex((n) => n > 3);
console.log(lastIndex); // 5
```

---

### 📌 indexOf() & lastIndexOf()

```javascript
const arr = [1, 2, 3, 2, 1];

console.log(arr.indexOf(2)); // 1 (first occurrence)
console.log(arr.lastIndexOf(2)); // 3 (last occurrence)
console.log(arr.indexOf(99)); // -1 (not found)

// Tìm từ vị trí cụ thể
console.log(arr.indexOf(2, 2)); // 3 (tìm từ index 2)

// ⚠️ Dùng === để so sánh
const mixed = [1, "1", true];
console.log(mixed.indexOf(1)); // 0
console.log(mixed.indexOf("1")); // 1
console.log(mixed.indexOf(true)); // 2
```

---

### 📌 includes() - Kiểm tra tồn tại

```javascript
const arr = [1, 2, 3, NaN];

console.log(arr.includes(2)); // true
console.log(arr.includes(99)); // false
console.log(arr.includes(NaN)); // true ✅

// So sánh với indexOf
console.log(arr.indexOf(NaN)); // -1 ❌ (indexOf không tìm được NaN)

// Tìm từ vị trí cụ thể
console.log(arr.includes(2, 2)); // false (tìm từ index 2)
```

| Method     | Return  | Tìm NaN  | Use case         |
| ---------- | ------- | -------- | ---------------- |
| includes() | boolean | ✅ Có    | Kiểm tra tồn tại |
| indexOf()  | number  | ❌ Không | Cần biết vị trí  |

---

## 5. Transform Methods

### 📌 Array.from() - Tạo array từ iterable

```javascript
// Từ string
Array.from("hello"); // ['h', 'e', 'l', 'l', 'o']

// Từ Set
Array.from(new Set([1, 2, 2, 3])); // [1, 2, 3]

// Từ Map
Array.from(
  new Map([
    ["a", 1],
    ["b", 2],
  ])
); // [['a', 1], ['b', 2]]

// Từ NodeList
Array.from(document.querySelectorAll("div"));

// Từ arguments
function example() {
  return Array.from(arguments);
}
example(1, 2, 3); // [1, 2, 3]

// Với map function
Array.from([1, 2, 3], (x) => x * 2); // [2, 4, 6]

// Tạo array với length
Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]

// Tạo 2D array
Array.from({ length: 3 }, () => Array(3).fill(0));
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

---

### 📌 Array.of() - Tạo array từ arguments

```javascript
Array.of(1, 2, 3); // [1, 2, 3]
Array.of(7); // [7]

// So sánh với Array constructor
Array(7); // [ , , , , , , ] - 7 empty slots
Array.of(7); // [7] - array với 1 phần tử
```

---

### 📌 Array.isArray() - Kiểm tra là array

```javascript
Array.isArray([1, 2, 3]); // true
Array.isArray("hello"); // false
Array.isArray({ length: 3 }); // false
Array.isArray(new Array()); // true

// Tại sao không dùng typeof?
typeof [1, 2, 3]; // 'object' - không phân biệt được!
```

---

## 6. Array Static Methods

### 📊 Tổng hợp Array Methods

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         ARRAY METHODS CHEAT SHEET                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  MUTATING (thay đổi gốc)          NON-MUTATING (không đổi gốc)                 │
│  ─────────────────────            ────────────────────────────                  │
│  push()      pop()                slice()        concat()                       │
│  unshift()   shift()              map()          filter()                       │
│  splice()    sort()               reduce()       flat()                         │
│  reverse()   fill()               flatMap()      join()                         │
│  copyWithin()                     toSorted()     toReversed()                   │
│                                   toSpliced()    with()                         │
│                                                                                 │
│  SEARCH                           ITERATION                                     │
│  ──────                           ─────────                                     │
│  find()      findIndex()          forEach()      map()                          │
│  findLast()  findLastIndex()      filter()       reduce()                       │
│  indexOf()   lastIndexOf()        every()        some()                         │
│  includes()                       reduceRight()                                 │
│                                                                                 │
│  STATIC                                                                         │
│  ──────                                                                         │
│  Array.from()    Array.of()    Array.isArray()                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

# OBJECT METHODS

## 7. Object Static Methods

### 📌 Object.keys() - Lấy tất cả keys

```javascript
const user = {
  name: "John",
  age: 30,
  email: "john@example.com",
};

const keys = Object.keys(user);
console.log(keys); // ['name', 'age', 'email']

// Đếm properties
console.log(Object.keys(user).length); // 3

// Iterate
Object.keys(user).forEach((key) => {
  console.log(`${key}: ${user[key]}`);
});

// ⚠️ Chỉ lấy enumerable own properties
// Không lấy inherited properties
// Không lấy Symbol keys
```

---

### 📌 Object.values() - Lấy tất cả values

```javascript
const user = {
  name: "John",
  age: 30,
  email: "john@example.com",
};

const values = Object.values(user);
console.log(values); // ['John', 30, 'john@example.com']

// Tính tổng
const scores = { math: 90, english: 85, science: 95 };
const total = Object.values(scores).reduce((a, b) => a + b, 0);
console.log(total); // 270
```

---

### 📌 Object.entries() - Lấy [key, value] pairs

```javascript
const user = {
  name: "John",
  age: 30,
  email: "john@example.com",
};

const entries = Object.entries(user);
console.log(entries);
// [['name', 'John'], ['age', 30], ['email', 'john@example.com']]

// Iterate với destructuring
for (const [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}

// Convert to Map
const map = new Map(Object.entries(user));

// Transform object
const doubled = Object.fromEntries(Object.entries({ a: 1, b: 2, c: 3 }).map(([key, value]) => [key, value * 2]));
console.log(doubled); // { a: 2, b: 4, c: 6 }

// Filter object
const filtered = Object.fromEntries(Object.entries({ a: 1, b: 2, c: 3, d: 4 }).filter(([key, value]) => value > 2));
console.log(filtered); // { c: 3, d: 4 }
```

---

### 📌 Object.fromEntries() - Tạo object từ entries

```javascript
// Từ array of arrays
const entries = [
  ["name", "John"],
  ["age", 30],
];
const obj = Object.fromEntries(entries);
console.log(obj); // { name: 'John', age: 30 }

// Từ Map
const map = new Map([
  ["a", 1],
  ["b", 2],
]);
const fromMap = Object.fromEntries(map);
console.log(fromMap); // { a: 1, b: 2 }

// URL search params to object
const params = new URLSearchParams("name=John&age=30");
const paramsObj = Object.fromEntries(params);
console.log(paramsObj); // { name: 'John', age: '30' }
```

---

### 📌 Object.assign() - Copy/Merge objects

```javascript
// Copy object (shallow)
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original);
console.log(copy); // { a: 1, b: 2 }

// Merge objects
const target = { a: 1, b: 2 };
const source1 = { b: 3, c: 4 };
const source2 = { c: 5, d: 6 };

const merged = Object.assign(target, source1, source2);
console.log(merged); // { a: 1, b: 3, c: 5, d: 6 }
console.log(target); // { a: 1, b: 3, c: 5, d: 6 } - target bị thay đổi!

// ✅ Không mutate target
const merged2 = Object.assign({}, target, source1, source2);

// Modern: Spread operator
const merged3 = { ...original, ...source1, ...source2 };

// ⚠️ Shallow copy only!
const nested = { a: { b: 1 } };
const shallowCopy = Object.assign({}, nested);
shallowCopy.a.b = 999;
console.log(nested.a.b); // 999 - nested object bị ảnh hưởng!

// Deep copy
const deepCopy = JSON.parse(JSON.stringify(nested));
// hoặc: structuredClone(nested) (modern)
```

---

### 📌 Object.freeze() - Đóng băng object

```javascript
const user = {
  name: "John",
  age: 30,
};

Object.freeze(user);

user.name = "Jane"; // Silently fails (strict mode: throws)
user.email = "test"; // Silently fails
delete user.age; // Silently fails

console.log(user); // { name: 'John', age: 30 } - không đổi

// Kiểm tra
console.log(Object.isFrozen(user)); // true

// ⚠️ Shallow freeze only!
const nested = { a: { b: 1 } };
Object.freeze(nested);
nested.a.b = 999; // Vẫn thay đổi được!
console.log(nested.a.b); // 999

// Deep freeze
function deepFreeze(obj) {
  Object.keys(obj).forEach((key) => {
    if (typeof obj[key] === "object" && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return Object.freeze(obj);
}
```

---

### 📌 Object.seal() - Seal object

```javascript
const user = {
  name: "John",
  age: 30,
};

Object.seal(user);

user.name = "Jane"; // ✅ Có thể thay đổi value
user.email = "test"; // ❌ Không thể thêm property
delete user.age; // ❌ Không thể xóa property

console.log(user); // { name: 'Jane', age: 30 }
console.log(Object.isSealed(user)); // true
```

| Method              | Thêm property | Xóa property | Thay đổi value |
| ------------------- | ------------- | ------------ | -------------- |
| freeze()            | ❌            | ❌           | ❌             |
| seal()              | ❌            | ❌           | ✅             |
| preventExtensions() | ❌            | ✅           | ✅             |

---

### 📌 Object.create() - Tạo object với prototype

```javascript
// Tạo object với prototype
const personProto = {
  greet() {
    return `Hello, I'm ${this.name}`;
  },
};

const john = Object.create(personProto);
john.name = "John";
console.log(john.greet()); // "Hello, I'm John"

// Tạo object không có prototype
const pureObj = Object.create(null);
console.log(pureObj.toString); // undefined (không có inherited methods)

// Với property descriptors
const user = Object.create(personProto, {
  name: {
    value: "Jane",
    writable: true,
    enumerable: true,
    configurable: true,
  },
  age: {
    value: 30,
    writable: false,
  },
});
```

---

### 📌 Object.getOwnPropertyNames() & Object.getOwnPropertySymbols()

```javascript
const sym = Symbol("id");
const obj = {
  name: "John",
  age: 30,
  [sym]: 123,
};

// Lấy tất cả string keys (kể cả non-enumerable)
console.log(Object.getOwnPropertyNames(obj)); // ['name', 'age']

// Lấy Symbol keys
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]

// Lấy tất cả keys
const allKeys = [...Object.getOwnPropertyNames(obj), ...Object.getOwnPropertySymbols(obj)];
// hoặc: Reflect.ownKeys(obj)
```

---

### 📌 Object.hasOwn() (ES2022) - Kiểm tra own property

```javascript
const user = {
  name: "John",
};

// ✅ Modern way
console.log(Object.hasOwn(user, "name")); // true
console.log(Object.hasOwn(user, "toString")); // false (inherited)

// ❌ Old way (có vấn đề với Object.create(null))
console.log(user.hasOwnProperty("name")); // true

// Vấn đề với hasOwnProperty
const obj = Object.create(null);
obj.name = "John";
// obj.hasOwnProperty('name'); // Error! hasOwnProperty không tồn tại
Object.hasOwn(obj, "name"); // true ✅
```

---

## 8. Object Instance Methods

### 📌 hasOwnProperty()

```javascript
const user = { name: "John" };

user.hasOwnProperty("name"); // true
user.hasOwnProperty("toString"); // false (inherited)

// ⚠️ Có thể bị override
const malicious = {
  hasOwnProperty: () => false,
};
malicious.hasOwnProperty("hasOwnProperty"); // false (sai!)

// ✅ Safe way
Object.prototype.hasOwnProperty.call(malicious, "hasOwnProperty"); // true
// hoặc: Object.hasOwn(malicious, 'hasOwnProperty')
```

---

### 📌 toString() & valueOf()

```javascript
const obj = { name: "John" };

console.log(obj.toString()); // "[object Object]"
console.log(obj.valueOf()); // { name: 'John' }

// Custom toString
const user = {
  name: "John",
  age: 30,
  toString() {
    return `${this.name} (${this.age})`;
  },
};
console.log(String(user)); // "John (30)"
console.log(`User: ${user}`); // "User: John (30)"

// Custom valueOf
const money = {
  amount: 100,
  currency: "USD",
  valueOf() {
    return this.amount;
  },
};
console.log(money + 50); // 150
```

---

## 9. Property Descriptors

### 📌 Object.getOwnPropertyDescriptor()

```javascript
const user = {
  name: "John",
};

const descriptor = Object.getOwnPropertyDescriptor(user, "name");
console.log(descriptor);
// {
//   value: 'John',
//   writable: true,
//   enumerable: true,
//   configurable: true
// }
```

### 📌 Object.defineProperty()

```javascript
const user = {};

Object.defineProperty(user, "name", {
  value: "John",
  writable: false, // Không thể thay đổi
  enumerable: true, // Hiện trong for...in
  configurable: false, // Không thể delete hoặc redefine
});

user.name = "Jane"; // Silently fails
console.log(user.name); // 'John'

// Getter/Setter
const person = {
  firstName: "John",
  lastName: "Doe",
};

Object.defineProperty(person, "fullName", {
  get() {
    return `${this.firstName} ${this.lastName}`;
  },
  set(value) {
    [this.firstName, this.lastName] = value.split(" ");
  },
  enumerable: true,
});

console.log(person.fullName); // 'John Doe'
person.fullName = "Jane Smith";
console.log(person.firstName); // 'Jane'
```

### 📌 Object.defineProperties()

```javascript
const user = {};

Object.defineProperties(user, {
  firstName: {
    value: "John",
    writable: true,
  },
  lastName: {
    value: "Doe",
    writable: true,
  },
  fullName: {
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
  },
});
```

---

## 10. Chaining Methods

### 📌 Method Chaining Pattern

```javascript
const users = [
  { name: "John", age: 25, active: true, score: 85 },
  { name: "Jane", age: 30, active: false, score: 92 },
  { name: "Bob", age: 35, active: true, score: 78 },
  { name: "Alice", age: 28, active: true, score: 95 },
  { name: "Charlie", age: 22, active: false, score: 88 },
];

// Chain multiple operations
const result = users
  .filter((u) => u.active) // Lọc active users
  .filter((u) => u.age >= 25) // Lọc age >= 25
  .map((u) => ({
    // Transform
    name: u.name,
    score: u.score,
  }))
  .sort((a, b) => b.score - a.score) // Sort by score desc
  .slice(0, 3); // Top 3

console.log(result);
// [{ name: 'Alice', score: 95 }, { name: 'John', score: 85 }, { name: 'Bob', score: 78 }]

// Tính average score của active users
const avgScore = users
  .filter((u) => u.active)
  .map((u) => u.score)
  .reduce((sum, score, _, arr) => sum + score / arr.length, 0);

console.log(avgScore); // 86
```

### 📌 Real-world Examples

```javascript
// E-commerce: Tính tổng giỏ hàng
const cart = [
  { name: "Phone", price: 999, quantity: 1, discount: 0.1 },
  { name: "Case", price: 29, quantity: 2, discount: 0 },
  { name: "Charger", price: 49, quantity: 1, discount: 0.2 },
];

const total = cart
  .map((item) => ({
    ...item,
    subtotal: item.price * item.quantity * (1 - item.discount),
  }))
  .reduce((sum, item) => sum + item.subtotal, 0);

console.log(total); // 999 * 0.9 + 29 * 2 + 49 * 0.8 = 996.3

// API Response processing
const apiResponse = {
  data: {
    users: [
      { id: 1, name: "John", email: "john@test.com", role: "admin" },
      { id: 2, name: "Jane", email: "jane@test.com", role: "user" },
      { id: 3, name: "Bob", email: "bob@test.com", role: "user" },
    ],
  },
};

const userEmails = apiResponse.data.users
  .filter((u) => u.role === "user")
  .map((u) => u.email)
  .join(", ");

console.log(userEmails); // "jane@test.com, bob@test.com"

// Text processing
const text = "  Hello   World  How   Are   You  ";
const words = text
  .trim()
  .split(/\s+/)
  .map((w) => w.toLowerCase())
  .filter((w) => w.length > 2);

console.log(words); // ['hello', 'world', 'how', 'are', 'you']
```

---

## 11. Performance Tips

### 📊 Time Complexity

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TIME COMPLEXITY                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  O(1) - Constant                                                        │
│  ├── push(), pop()                                                      │
│  ├── Array access by index: arr[i]                                      │
│  └── Object property access: obj.key                                    │
│                                                                         │
│  O(n) - Linear                                                          │
│  ├── shift(), unshift()                                                 │
│  ├── splice()                                                           │
│  ├── slice(), concat()                                                  │
│  ├── map(), filter(), reduce(), forEach()                              │
│  ├── find(), findIndex(), indexOf(), includes()                        │
│  ├── every(), some()                                                    │
│  └── Object.keys(), Object.values(), Object.entries()                  │
│                                                                         │
│  O(n log n)                                                             │
│  └── sort()                                                             │
│                                                                         │
│  O(n²) - Quadratic (tránh!)                                            │
│  └── Nested loops với array methods                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💡 Performance Best Practices

```javascript
// ❌ BAD: Multiple iterations
const result = arr
  .filter((x) => x > 0)
  .map((x) => x * 2)
  .filter((x) => x < 100);
// 3 lần duyệt array

// ✅ GOOD: Single iteration với reduce
const result2 = arr.reduce((acc, x) => {
  if (x > 0) {
    const doubled = x * 2;
    if (doubled < 100) {
      acc.push(doubled);
    }
  }
  return acc;
}, []);
// 1 lần duyệt array

// ❌ BAD: indexOf trong loop - O(n²)
const unique = arr.filter((v, i) => arr.indexOf(v) === i);

// ✅ GOOD: Set - O(n)
const unique2 = [...new Set(arr)];

// ❌ BAD: Tìm kiếm trong array - O(n)
const users = [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" },
];
const findUser = (id) => users.find((u) => u.id === id); // O(n) mỗi lần

// ✅ GOOD: Convert to Map/Object - O(1) lookup
const userMap = new Map(users.map((u) => [u.id, u]));
const findUser2 = (id) => userMap.get(id); // O(1)

// ❌ BAD: unshift trong loop - O(n²)
const reversed = [];
arr.forEach((x) => reversed.unshift(x));

// ✅ GOOD: push + reverse - O(n)
const reversed2 = [];
arr.forEach((x) => reversed2.push(x));
reversed2.reverse();
// hoặc: arr.slice().reverse()

// ❌ BAD: Spread trong reduce - O(n²)
const flattened = nested.reduce((acc, curr) => [...acc, ...curr], []);

// ✅ GOOD: push trong reduce - O(n)
const flattened2 = nested.reduce((acc, curr) => {
  acc.push(...curr);
  return acc;
}, []);
// hoặc: nested.flat()
```

### 📊 Memory Tips

```javascript
// ❌ BAD: Tạo nhiều intermediate arrays
const result = hugeArray
  .map((x) => x * 2) // Array mới
  .filter((x) => x > 10) // Array mới
  .slice(0, 100); // Array mới

// ✅ GOOD: Generator cho large datasets
function* processLarge(arr) {
  for (const x of arr) {
    const doubled = x * 2;
    if (doubled > 10) {
      yield doubled;
    }
  }
}

const result2 = [...processLarge(hugeArray)].slice(0, 100);

// ✅ GOOD: Early termination
const first100 = [];
for (const x of hugeArray) {
  const doubled = x * 2;
  if (doubled > 10) {
    first100.push(doubled);
    if (first100.length >= 100) break;
  }
}
```

---

## 12. Common Patterns

### 📌 Remove Duplicates

```javascript
const arr = [1, 2, 2, 3, 3, 3, 4];

// Method 1: Set (recommended)
const unique1 = [...new Set(arr)];

// Method 2: filter + indexOf
const unique2 = arr.filter((v, i) => arr.indexOf(v) === i);

// Method 3: reduce
const unique3 = arr.reduce((acc, v) => (acc.includes(v) ? acc : [...acc, v]), []);

// For objects (by property)
const users = [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" },
  { id: 1, name: "John Duplicate" },
];

const uniqueById = [...new Map(users.map((u) => [u.id, u])).values()];
```

### 📌 Group By

```javascript
const people = [
  { name: "John", department: "IT" },
  { name: "Jane", department: "HR" },
  { name: "Bob", department: "IT" },
  { name: "Alice", department: "HR" },
];

// Method 1: reduce
const grouped = people.reduce((acc, person) => {
  const key = person.department;
  acc[key] = acc[key] || [];
  acc[key].push(person);
  return acc;
}, {});

// Method 2: Object.groupBy (ES2024)
const grouped2 = Object.groupBy(people, (p) => p.department);

// Result:
// {
//   IT: [{ name: 'John', ... }, { name: 'Bob', ... }],
//   HR: [{ name: 'Jane', ... }, { name: 'Alice', ... }]
// }
```

### 📌 Chunk Array

```javascript
function chunk(arr, size) {
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

// hoặc với reduce
const chunk2 = (arr, size) => arr.reduce((acc, _, i) => (i % size === 0 ? [...acc, arr.slice(i, i + size)] : acc), []);

console.log(chunk([1, 2, 3, 4, 5, 6, 7], 3));
// [[1, 2, 3], [4, 5, 6], [7]]
```

### 📌 Intersection & Difference

```javascript
const arr1 = [1, 2, 3, 4, 5];
const arr2 = [3, 4, 5, 6, 7];

// Intersection (phần tử chung)
const intersection = arr1.filter((x) => arr2.includes(x));
// [3, 4, 5]

// Difference (có trong arr1, không có trong arr2)
const difference = arr1.filter((x) => !arr2.includes(x));
// [1, 2]

// Symmetric difference (không chung)
const symmetricDiff = [...arr1.filter((x) => !arr2.includes(x)), ...arr2.filter((x) => !arr1.includes(x))];
// [1, 2, 6, 7]

// Union (tất cả, không trùng)
const union = [...new Set([...arr1, ...arr2])];
// [1, 2, 3, 4, 5, 6, 7]
```

### 📌 Deep Clone Object

```javascript
const original = {
  name: "John",
  address: {
    city: "NYC",
    zip: "10001",
  },
  hobbies: ["reading", "gaming"],
};

// Method 1: JSON (không hỗ trợ functions, Date, undefined, etc.)
const clone1 = JSON.parse(JSON.stringify(original));

// Method 2: structuredClone (modern, recommended)
const clone2 = structuredClone(original);

// Method 3: Recursive function
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map((item) => deepClone(item));

  const cloned = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}
```

### 📌 Pick & Omit

```javascript
const user = {
  id: 1,
  name: "John",
  email: "john@test.com",
  password: "secret",
  age: 30,
};

// Pick: Chỉ lấy một số properties
const pick = (obj, keys) => Object.fromEntries(keys.map((k) => [k, obj[k]]));

const publicUser = pick(user, ["id", "name", "email"]);
// { id: 1, name: 'John', email: 'john@test.com' }

// Omit: Loại bỏ một số properties
const omit = (obj, keys) => Object.fromEntries(Object.entries(obj).filter(([k]) => !keys.includes(k)));

const safeUser = omit(user, ["password"]);
// { id: 1, name: 'John', email: 'john@test.com', age: 30 }
```

### 📌 Flatten Object

```javascript
const nested = {
  name: "John",
  address: {
    city: "NYC",
    zip: "10001",
    country: {
      name: "USA",
      code: "US",
    },
  },
};

function flattenObject(obj, prefix = "") {
  return Object.entries(obj).reduce((acc, [key, value]) => {
    const newKey = prefix ? `${prefix}.${key}` : key;
    if (typeof value === "object" && value !== null && !Array.isArray(value)) {
      Object.assign(acc, flattenObject(value, newKey));
    } else {
      acc[newKey] = value;
    }
    return acc;
  }, {});
}

console.log(flattenObject(nested));
// {
//   'name': 'John',
//   'address.city': 'NYC',
//   'address.zip': '10001',
//   'address.country.name': 'USA',
//   'address.country.code': 'US'
// }
```

---

## 📊 Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         QUICK REFERENCE                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  THÊM/XÓA                                                                       │
│  push()/pop()      - Cuối array, O(1)                                          │
│  unshift()/shift() - Đầu array, O(n)                                           │
│  splice()          - Bất kỳ vị trí, O(n)                                       │
│                                                                                 │
│  TÌM KIẾM                                                                       │
│  find()            - Tìm phần tử đầu tiên thỏa điều kiện                       │
│  findIndex()       - Tìm index đầu tiên thỏa điều kiện                         │
│  indexOf()         - Tìm index của giá trị                                     │
│  includes()        - Kiểm tra tồn tại (boolean)                                │
│                                                                                 │
│  BIẾN ĐỔI                                                                       │
│  map()             - Transform mỗi phần tử                                     │
│  filter()          - Lọc theo điều kiện                                        │
│  reduce()          - Gộp thành một giá trị                                     │
│  flat()/flatMap()  - Làm phẳng array                                           │
│                                                                                 │
│  KIỂM TRA                                                                       │
│  every()           - Tất cả thỏa điều kiện?                                    │
│  some()            - Có ít nhất một thỏa?                                      │
│  Array.isArray()   - Có phải array?                                            │
│                                                                                 │
│  OBJECT                                                                         │
│  Object.keys()     - Lấy tất cả keys                                           │
│  Object.values()   - Lấy tất cả values                                         │
│  Object.entries()  - Lấy [key, value] pairs                                    │
│  Object.assign()   - Copy/merge objects                                        │
│  Object.freeze()   - Đóng băng object                                          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Tài liệu tham khảo

- [MDN - Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [MDN - Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [JavaScript.info - Arrays](https://javascript.info/array)
- [JavaScript.info - Object methods](https://javascript.info/object-methods)
