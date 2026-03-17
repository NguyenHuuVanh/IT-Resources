# ES6 (ECMAScript 2015) - Hướng dẫn toàn diện

## 📌 Giới thiệu

**ES6** (ECMAScript 2015) là phiên bản cập nhật lớn nhất trong lịch sử JavaScript, được phát hành chính thức vào tháng 6 năm 2015 bởi tổ chức ECMA International (thông qua TC39 committee). ES6 đánh dấu bước ngoặt quan trọng, đưa JavaScript từ một ngôn ngữ scripting đơn giản trở thành một ngôn ngữ lập trình mạnh mẽ, hiện đại, phù hợp cho cả ứng dụng lớn và phức tạp.

### Tại sao ES6 quan trọng?

- **Code ngắn gọn hơn**: Arrow functions, template literals, destructuring giúp viết ít code hơn
- **Dễ bảo trì hơn**: Classes, modules giúp tổ chức code theo cấu trúc rõ ràng
- **Ít bug hơn**: `let`/`const` thay thế `var` giúp tránh nhiều lỗi phổ biến
- **Xử lý bất đồng bộ tốt hơn**: Promises thay thế callback hell
- **Là nền tảng cho các framework hiện đại**: React, Vue, Angular đều dựa trên ES6+

### Lịch sử ECMAScript

| Version | Năm      | Tên gọi             | Highlights                                      |
| ------- | -------- | -------------------- | ----------------------------------------------- |
| ES1     | 1997     | ECMAScript 1         | First edition                                   |
| ES2     | 1998     | ECMAScript 2         | Editorial changes                               |
| ES3     | 1999     | ECMAScript 3         | Regular expressions, try/catch, switch          |
| ES4     | —        | (Abandoned)          | Quá phức tạp, bị hủy bỏ                        |
| ES5     | 2009     | ECMAScript 5         | Strict mode, JSON, Array methods                |
| ES5.1   | 2011     | ECMAScript 5.1       | Aligned with ISO/IEC 16262:2011                 |
| **ES6** | **2015** | **ECMAScript 2015**  | **Classes, Modules, Promises, Arrow functions** |
| ES7     | 2016     | ECMAScript 2016      | Array.includes(), `**` operator                 |
| ES8     | 2017     | ECMAScript 2017      | async/await, Object.entries()                   |
| ES9     | 2018     | ECMAScript 2018      | Rest/Spread properties, async iteration         |
| ES10    | 2019     | ECMAScript 2019      | flat(), flatMap(), trimStart(), trimEnd()        |
| ES11    | 2020     | ECMAScript 2020      | Optional chaining, nullish coalescing           |
| ES12    | 2021     | ECMAScript 2021      | String.replaceAll(), Promise.any()              |
| ES13    | 2022     | ECMAScript 2022      | Top-level await, private fields (#)             |
| ES14    | 2023     | ECMAScript 2023      | Array findLast(), toSorted(), toReversed()      |

### Tổng quan các tính năng ES6

```
┌─────────────────────────────────────────────────────────────┐
│                    ES6 NEW FEATURES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Variables:        let, const                              │
│  Functions:        Arrow functions, Default params         │
│  Strings:          Template literals, New methods          │
│  Objects:          Enhanced literals, Destructuring        │
│  Arrays:           Spread, Destructuring, New methods      │
│  Classes:          class, extends, super, static           │
│  Modules:          import, export                          │
│  Async:            Promises, Generators                    │
│  Collections:      Map, Set, WeakMap, WeakSet              │
│  Others:           Symbol, Proxy, Reflect, for...of        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Mục lục chi tiết

| # | Chủ đề | File | Mô tả |
|---|--------|------|--------|
| 01 | let & const | [01-let-const.md](./01-let-const.md) | Block scope, TDZ, var vs let vs const |
| 02 | Arrow Functions | [02-arrow-functions.md](./02-arrow-functions.md) | Cú pháp, lexical this, khi nào dùng/không dùng |
| 03 | Template Literals | [03-template-literals.md](./03-template-literals.md) | Interpolation, multi-line, tagged templates |
| 04 | Destructuring | [04-destructuring.md](./04-destructuring.md) | Array/Object destructuring, nested, defaults |
| 05 | Spread & Rest | [05-spread-rest.md](./05-spread-rest.md) | Spread operator, rest parameters, use cases |
| 06 | Default Parameters | [06-default-parameters.md](./06-default-parameters.md) | Default values, expressions, patterns |
| 07 | Enhanced Object Literals | [07-enhanced-object-literals.md](./07-enhanced-object-literals.md) | Shorthand, computed properties |
| 08 | Classes | [08-classes.md](./08-classes.md) | Class syntax, inheritance, static, private |
| 09 | Modules | [09-modules.md](./09-modules.md) | import/export, dynamic imports, patterns |
| 10 | Promises | [10-promises.md](./10-promises.md) | Promise API, chaining, error handling |
| 11 | Symbol | [11-symbol.md](./11-symbol.md) | Unique identifiers, well-known symbols |
| 12 | Iterators & Generators | [12-iterators-generators.md](./12-iterators-generators.md) | Iterator protocol, generator functions |
| 13 | Map & Set | [13-map-set.md](./13-map-set.md) | Map, Set, WeakMap, WeakSet |
| 14 | for...of Loop | [14-for-of-loop.md](./14-for-of-loop.md) | Iterable protocol, so sánh các loại loop |
| 15 | New Built-in Methods | [15-new-methods.md](./15-new-methods.md) | Array, String, Object, Number/Math methods |
| 16 | Proxy & Reflect | [16-proxy-reflect.md](./16-proxy-reflect.md) | Metaprogramming, traps, practical examples |

---

## 🔧 Browser Support

ES6 được hỗ trợ bởi tất cả modern browsers (Chrome 51+, Firefox 54+, Safari 10+, Edge 14+). Với older browsers, sử dụng:

- **[Babel](https://babeljs.io/)** - Transpiler ES6+ → ES5
- **[core-js](https://github.com/zloirock/core-js)** - Polyfills cho các features thiếu
- **[TypeScript](https://www.typescriptlang.org/)** - Superset of JavaScript, tự động transpile

### Cấu hình Babel cơ bản

```json
// .babelrc hoặc babel.config.json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "> 0.25%, not dead",
      "useBuiltIns": "usage",
      "corejs": 3
    }]
  ]
}
```

---

## 📖 Tài liệu tham khảo

- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [ECMAScript 6 Features](http://es6-features.org/)
- [Exploring ES6](https://exploringjs.com/es6/)
- [JavaScript.info](https://javascript.info/)
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)
- [TC39 Proposals](https://github.com/tc39/proposals)
- [Can I Use](https://caniuse.com/) - Kiểm tra browser compatibility
