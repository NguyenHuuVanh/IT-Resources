# Web Components - Hướng dẫn toàn diện

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Custom Elements](#2-custom-elements)
3. [Shadow DOM](#3-shadow-dom)
4. [HTML Templates](#4-html-templates)
5. [Slots](#5-slots)
6. [Lifecycle Callbacks](#6-lifecycle-callbacks)
7. [Attributes và Properties](#7-attributes-và-properties)
8. [Events trong Web Components](#8-events-trong-web-components)
9. [Styling Web Components](#9-styling-web-components)
10. [Ví dụ thực tế](#10-ví-dụ-thực-tế)
11. [Best Practices](#11-best-practices)

---

## 1. Tổng quan

### 🎯 Web Components là gì?

Web Components là tập hợp các **Web APIs** cho phép tạo các **custom HTML elements** có thể tái sử dụng, đóng gói (encapsulated) và độc lập với framework.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        WEB COMPONENTS                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│   │ Custom Elements │  │   Shadow DOM    │  │ HTML Templates  │        │
│   │                 │  │                 │  │                 │        │
│   │ Định nghĩa      │  │ Đóng gói CSS    │  │ Định nghĩa      │        │
│   │ element mới    │  │ và markup       │  │ markup template │        │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│            │                   │                    │                   │
│            └───────────────────┼────────────────────┘                   │
│                                │                                        │
│                    ┌───────────▼───────────┐                           │
│                    │   <my-component>      │                           │
│                    │   Reusable, Isolated  │                           │
│                    │   Framework-agnostic  │                           │
│                    └───────────────────────┘                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📊 3 Công nghệ chính

| Công nghệ           | Mô tả                                         |
| ------------------- | --------------------------------------------- |
| **Custom Elements** | API để định nghĩa HTML elements mới           |
| **Shadow DOM**      | Đóng gói DOM và CSS riêng biệt                |
| **HTML Templates**  | `<template>` và `<slot>` để định nghĩa markup |

### ✅ Ưu điểm

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ƯU ĐIỂM                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Framework-agnostic    Hoạt động với React, Vue, Angular, Vanilla    │
│  ✓ Encapsulation         CSS và DOM được đóng gói, không conflict      │
│  ✓ Reusability           Dùng lại ở bất kỳ đâu như HTML element        │
│  ✓ Native Browser        Không cần thư viện, chạy native               │
│  ✓ Interoperability      Tương thích với mọi web technology            │
│  ✓ Standards-based       Theo chuẩn W3C, được hỗ trợ lâu dài           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🌐 Browser Support

```
Chrome  ████████████████████  ✓ Full support (v67+)
Firefox ████████████████████  ✓ Full support (v63+)
Safari  ████████████████████  ✓ Full support (v10.1+)
Edge    ████████████████████  ✓ Full support (v79+)
```

---

## 2. Custom Elements

### 🎯 Định nghĩa

Custom Elements cho phép tạo **HTML tags mới** với behavior tùy chỉnh.

### 📋 Quy tắc đặt tên

```
✓ my-component      (có dấu gạch ngang)
✓ user-card         (có dấu gạch ngang)
✓ app-header        (có dấu gạch ngang)

✗ mycomponent       (không có gạch ngang)
✗ MyComponent       (không có gạch ngang)
✗ header            (trùng với HTML tag)
```

### 💻 Cú pháp cơ bản

```javascript
// Định nghĩa Custom Element
class MyComponent extends HTMLElement {
  constructor() {
    super(); // Luôn gọi super() đầu tiên
    console.log("Component created");
  }
}

// Đăng ký với browser
customElements.define("my-component", MyComponent);
```

```html
<!-- Sử dụng trong HTML -->
<my-component></my-component>

<!-- Hoặc tạo bằng JavaScript -->
<script>
  const el = document.createElement("my-component");
  document.body.appendChild(el);
</script>
```

### 🔧 Ví dụ: Hello World Component

```javascript
class HelloWorld extends HTMLElement {
  constructor() {
    super();
    this.innerHTML = `<h1>Hello, World!</h1>`;
  }
}

customElements.define("hello-world", HelloWorld);
```

```html
<hello-world></hello-world>
<!-- Output: <hello-world><h1>Hello, World!</h1></hello-world> -->
```

### 📦 Autonomous vs Customized Built-in Elements

```javascript
// 1. Autonomous Custom Element (tạo tag mới hoàn toàn)
class MyButton extends HTMLElement {
  constructor() {
    super();
    this.innerHTML = `<button>Click me</button>`;
  }
}
customElements.define("my-button", MyButton);
// Sử dụng: <my-button></my-button>

// 2. Customized Built-in Element (mở rộng tag có sẵn)
class FancyButton extends HTMLButtonElement {
  constructor() {
    super();
    this.style.background = "linear-gradient(45deg, #ff6b6b, #feca57)";
    this.style.border = "none";
    this.style.padding = "10px 20px";
    this.style.borderRadius = "5px";
    this.style.color = "white";
  }
}
customElements.define("fancy-button", FancyButton, { extends: "button" });
// Sử dụng: <button is="fancy-button">Click me</button>
```

---

## 3. Shadow DOM

### 🎯 Định nghĩa

Shadow DOM tạo một **DOM tree riêng biệt** được đóng gói bên trong element, giúp **cô lập CSS và markup**.

### 🖼️ Minh họa

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SHADOW DOM                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Document DOM (Light DOM)                                              │
│   ┌─────────────────────────────────────────────┐                      │
│   │  <html>                                     │                      │
│   │    <body>                                   │                      │
│   │      <my-card>                              │                      │
│   │        ┌─────────────────────────────┐      │                      │
│   │        │ #shadow-root (Shadow DOM)   │      │                      │
│   │        │   <style>                   │      │  CSS ở đây KHÔNG     │
│   │        │     h1 { color: red; }      │      │  ảnh hưởng ra ngoài  │
│   │        │   </style>                  │      │                      │
│   │        │   <h1>Shadow Content</h1>   │      │                      │
│   │        └─────────────────────────────┘      │                      │
│   │      </my-card>                             │                      │
│   │      <h1>Light DOM Content</h1>  ← Không bị ảnh hưởng             │
│   │    </body>                                  │                      │
│   │  </html>                                    │                      │
│   └─────────────────────────────────────────────┘                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Tạo Shadow DOM

```javascript
class MyCard extends HTMLElement {
  constructor() {
    super();

    // Tạo Shadow DOM
    const shadow = this.attachShadow({ mode: "open" });

    // Thêm content vào Shadow DOM
    shadow.innerHTML = `
            <style>
                /* CSS chỉ áp dụng trong Shadow DOM */
                .card {
                    padding: 20px;
                    border-radius: 8px;
                    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                    background: white;
                }
                h2 {
                    color: #333;
                    margin: 0 0 10px 0;
                }
                p {
                    color: #666;
                    margin: 0;
                }
            </style>
            <div class="card">
                <h2>Card Title</h2>
                <p>Card content goes here</p>
            </div>
        `;
  }
}

customElements.define("my-card", MyCard);
```

### 🔒 Shadow DOM Mode

```javascript
// Mode: 'open' - có thể truy cập từ bên ngoài
const shadow = this.attachShadow({ mode: "open" });
element.shadowRoot; // Trả về shadow root

// Mode: 'closed' - không thể truy cập từ bên ngoài
const shadow = this.attachShadow({ mode: "closed" });
element.shadowRoot; // Trả về null
```

### 📊 So sánh Light DOM vs Shadow DOM

| Đặc điểm      | Light DOM                  | Shadow DOM                           |
| ------------- | -------------------------- | ------------------------------------ |
| CSS Scope     | Global                     | Isolated                             |
| Truy cập      | `document.querySelector()` | `element.shadowRoot.querySelector()` |
| Style leak    | Có                         | Không                                |
| Encapsulation | Không                      | Có                                   |

---

## 4. HTML Templates

### 🎯 Định nghĩa

`<template>` chứa HTML markup **không được render** cho đến khi được clone và thêm vào DOM.

### 💻 Cú pháp cơ bản

```html
<!-- Template không hiển thị trên trang -->
<template id="my-template">
  <style>
    .greeting {
      color: blue;
      font-size: 24px;
    }
  </style>
  <div class="greeting">
    <h1>Hello!</h1>
    <p>Welcome to Web Components</p>
  </div>
</template>

<script>
  // Sử dụng template
  const template = document.getElementById("my-template");
  const clone = template.content.cloneNode(true);
  document.body.appendChild(clone);
</script>
```

### 🔧 Kết hợp Template với Custom Element

```javascript
// Định nghĩa template
const template = document.createElement("template");
template.innerHTML = `
    <style>
        :host {
            display: block;
            padding: 20px;
            background: #f5f5f5;
            border-radius: 8px;
        }
        .title {
            font-size: 1.5em;
            color: #333;
        }
        .content {
            color: #666;
            margin-top: 10px;
        }
    </style>
    <div class="title">Default Title</div>
    <div class="content">Default Content</div>
`;

class InfoBox extends HTMLElement {
  constructor() {
    super();

    // Attach shadow và clone template
    const shadow = this.attachShadow({ mode: "open" });
    shadow.appendChild(template.content.cloneNode(true));
  }
}

customElements.define("info-box", InfoBox);
```

---

## 5. Slots

### 🎯 Định nghĩa

Slots cho phép **chèn content từ Light DOM** vào vị trí cụ thể trong Shadow DOM.

### 🖼️ Minh họa

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            SLOTS                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Light DOM (người dùng viết):                                         │
│   ┌─────────────────────────────────────────────┐                      │
│   │  <user-card>                                │                      │
│   │    <span slot="name">John Doe</span>        │ ──┐                  │
│   │    <span slot="email">john@email.com</span> │ ──┼─┐                │
│   │  </user-card>                               │   │ │                │
│   └─────────────────────────────────────────────┘   │ │                │
│                                                      │ │                │
│   Shadow DOM (component định nghĩa):                │ │                │
│   ┌─────────────────────────────────────────────┐   │ │                │
│   │  <div class="card">                         │   │ │                │
│   │    <h2><slot name="name"></slot></h2>       │ ◄─┘ │                │
│   │    <p><slot name="email"></slot></p>        │ ◄───┘                │
│   │  </div>                                     │                      │
│   └─────────────────────────────────────────────┘                      │
│                                                                         │
│   Kết quả render:                                                      │
│   ┌─────────────────────────────────────────────┐                      │
│   │  <div class="card">                         │                      │
│   │    <h2>John Doe</h2>                        │                      │
│   │    <p>john@email.com</p>                    │                      │
│   │  </div>                                     │                      │
│   └─────────────────────────────────────────────┘                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Ví dụ: User Card với Slots

```javascript
class UserCard extends HTMLElement {
  constructor() {
    super();

    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                .card {
                    display: flex;
                    align-items: center;
                    padding: 20px;
                    background: white;
                    border-radius: 10px;
                    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
                    max-width: 300px;
                }
                .avatar {
                    width: 60px;
                    height: 60px;
                    border-radius: 50%;
                    margin-right: 15px;
                    background: #ddd;
                    overflow: hidden;
                }
                .avatar ::slotted(img) {
                    width: 100%;
                    height: 100%;
                    object-fit: cover;
                }
                .info h3 {
                    margin: 0 0 5px 0;
                    color: #333;
                }
                .info p {
                    margin: 0;
                    color: #666;
                    font-size: 14px;
                }
            </style>
            
            <div class="card">
                <div class="avatar">
                    <slot name="avatar"></slot>
                </div>
                <div class="info">
                    <h3><slot name="name">Anonymous</slot></h3>
                    <p><slot name="email">No email</slot></p>
                </div>
            </div>
        `;
  }
}

customElements.define("user-card", UserCard);
```

```html
<!-- Sử dụng -->
<user-card>
  <img slot="avatar" src="avatar.jpg" alt="Avatar" />
  <span slot="name">John Doe</span>
  <span slot="email">john@example.com</span>
</user-card>

<!-- Với default content (không truyền slot) -->
<user-card>
  <span slot="name">Jane Doe</span>
  <!-- email sẽ hiển thị "No email" (default) -->
</user-card>
```

### 🔧 Default Slot (Unnamed Slot)

```javascript
// Component
this.shadowRoot.innerHTML = `
    <div class="wrapper">
        <slot></slot>  <!-- Default slot -->
    </div>
`;
```

```html
<!-- Sử dụng -->
<my-wrapper>
  <p>This goes into default slot</p>
  <p>This too!</p>
</my-wrapper>
```

### 📡 Slot Events

```javascript
// Lắng nghe khi slot content thay đổi
const slot = this.shadowRoot.querySelector("slot");
slot.addEventListener("slotchange", (e) => {
  const assignedNodes = slot.assignedNodes();
  console.log("Slot content changed:", assignedNodes);
});
```

---

## 6. Lifecycle Callbacks

### 🎯 Định nghĩa

Lifecycle callbacks là các methods được gọi tự động tại các thời điểm khác nhau trong vòng đời của component.

### 🔄 Các Lifecycle Methods

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      LIFECYCLE CALLBACKS                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────┐                                                  │
│   │   constructor   │  ← Element được tạo (chưa trong DOM)             │
│   └────────┬────────┘                                                  │
│            │                                                            │
│            ▼                                                            │
│   ┌─────────────────────┐                                              │
│   │ connectedCallback   │  ← Element được thêm vào DOM                 │
│   └────────┬────────────┘                                              │
│            │                                                            │
│            ▼                                                            │
│   ┌──────────────────────────────┐                                     │
│   │ attributeChangedCallback     │  ← Attribute thay đổi               │
│   │ (name, oldValue, newValue)   │                                     │
│   └────────┬─────────────────────┘                                     │
│            │                                                            │
│            ▼                                                            │
│   ┌─────────────────────┐                                              │
│   │ adoptedCallback     │  ← Element được move sang document khác      │
│   └────────┬────────────┘                                              │
│            │                                                            │
│            ▼                                                            │
│   ┌──────────────────────┐                                             │
│   │ disconnectedCallback │  ← Element bị remove khỏi DOM               │
│   └──────────────────────┘                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Ví dụ đầy đủ

```javascript
class LifecycleDemo extends HTMLElement {
  // Khai báo attributes cần observe
  static get observedAttributes() {
    return ["name", "color"];
  }

  constructor() {
    super();
    console.log("1. constructor - Element created");

    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host { display: block; padding: 20px; }
            </style>
            <div id="content">Loading...</div>
        `;
  }

  connectedCallback() {
    console.log("2. connectedCallback - Added to DOM");

    // Thời điểm tốt để:
    // - Fetch data
    // - Add event listeners
    // - Start animations
    this.render();
  }

  disconnectedCallback() {
    console.log("5. disconnectedCallback - Removed from DOM");

    // Cleanup:
    // - Remove event listeners
    // - Cancel timers/intervals
    // - Abort fetch requests
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log(`3. attributeChangedCallback - ${name}: ${oldValue} → ${newValue}`);

    // Re-render khi attribute thay đổi
    if (oldValue !== newValue) {
      this.render();
    }
  }

  adoptedCallback() {
    console.log("4. adoptedCallback - Moved to new document");
  }

  // Custom method
  render() {
    const name = this.getAttribute("name") || "World";
    const color = this.getAttribute("color") || "black";

    this.shadowRoot.getElementById("content").innerHTML = `
            <h1 style="color: ${color}">Hello, ${name}!</h1>
        `;
  }
}

customElements.define("lifecycle-demo", LifecycleDemo);
```

```html
<!-- Sử dụng -->
<lifecycle-demo name="John" color="blue"></lifecycle-demo>

<script>
  const demo = document.querySelector("lifecycle-demo");

  // Thay đổi attribute → trigger attributeChangedCallback
  demo.setAttribute("name", "Jane");
  demo.setAttribute("color", "red");

  // Remove element → trigger disconnectedCallback
  // demo.remove();
</script>
```

### ⚠️ Lưu ý quan trọng

```javascript
class MyComponent extends HTMLElement {
    constructor() {
        super();

        // ✅ OK trong constructor
        this.attachShadow({ mode: 'open' });
        this.state = {};

        // ❌ KHÔNG nên trong constructor
        // this.getAttribute('...')  // Có thể chưa có
        // this.innerHTML = '...'    // Chưa trong DOM
        // fetch('...')              // Quá sớm
    }

    connectedCallback() {
        // ✅ Thời điểm tốt để:
        this.getAttribute('...');    // Đọc attributes
        this.render();               // Render content
        this.fetchData();            // Fetch data
        this.addEventListener(...);  // Add listeners
    }
}
```

---

## 7. Attributes và Properties

### 🎯 Sự khác biệt

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  ATTRIBUTES vs PROPERTIES                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ATTRIBUTES (HTML)              PROPERTIES (JavaScript)                │
│   ─────────────────              ──────────────────────                 │
│   • Định nghĩa trong HTML        • Định nghĩa trong JS object           │
│   • Luôn là string               • Có thể là bất kỳ type                │
│   • Reflect trong DOM            • Không reflect trong DOM              │
│   • getAttribute/setAttribute    • Truy cập trực tiếp: el.prop          │
│                                                                         │
│   <my-el name="John" active>     element.name = "John"                  │
│                                  element.active = true                  │
│                                  element.data = { id: 1 }               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Sync Attributes và Properties

```javascript
class MyInput extends HTMLElement {
  static get observedAttributes() {
    return ["value", "disabled"];
  }

  constructor() {
    super();
    this._value = "";
    this._disabled = false;

    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                input {
                    padding: 10px;
                    border: 2px solid #ddd;
                    border-radius: 5px;
                    font-size: 16px;
                }
                input:disabled {
                    background: #f5f5f5;
                    cursor: not-allowed;
                }
            </style>
            <input type="text">
        `;

    this._input = this.shadowRoot.querySelector("input");
  }

  // Property getter/setter cho 'value'
  get value() {
    return this._value;
  }

  set value(val) {
    this._value = val;
    this._input.value = val;
    // Reflect to attribute
    this.setAttribute("value", val);
  }

  // Property getter/setter cho 'disabled'
  get disabled() {
    return this._disabled;
  }

  set disabled(val) {
    this._disabled = Boolean(val);
    this._input.disabled = this._disabled;
    // Reflect to attribute
    if (this._disabled) {
      this.setAttribute("disabled", "");
    } else {
      this.removeAttribute("disabled");
    }
  }

  // Sync attribute → property
  attributeChangedCallback(name, oldValue, newValue) {
    switch (name) {
      case "value":
        this._value = newValue || "";
        this._input.value = this._value;
        break;
      case "disabled":
        this._disabled = newValue !== null;
        this._input.disabled = this._disabled;
        break;
    }
  }

  connectedCallback() {
    // Sync initial attributes
    this.value = this.getAttribute("value") || "";
    this.disabled = this.hasAttribute("disabled");

    // Listen for input changes
    this._input.addEventListener("input", (e) => {
      this._value = e.target.value;
      this.dispatchEvent(
        new CustomEvent("change", {
          detail: { value: this._value },
        })
      );
    });
  }
}

customElements.define("my-input", MyInput);
```

```html
<!-- Sử dụng với attributes -->
<my-input value="Hello" disabled></my-input>

<script>
  const input = document.querySelector("my-input");

  // Sử dụng properties
  input.value = "New Value";
  input.disabled = false;

  // Listen for changes
  input.addEventListener("change", (e) => {
    console.log("Value changed:", e.detail.value);
  });
</script>
```

---

## 8. Events trong Web Components

### 🎯 Custom Events

```javascript
class CounterButton extends HTMLElement {
  constructor() {
    super();
    this._count = 0;

    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                button {
                    padding: 10px 20px;
                    font-size: 18px;
                    cursor: pointer;
                    background: #4CAF50;
                    color: white;
                    border: none;
                    border-radius: 5px;
                }
                button:hover {
                    background: #45a049;
                }
            </style>
            <button>Count: 0</button>
        `;

    this._button = this.shadowRoot.querySelector("button");
  }

  connectedCallback() {
    this._button.addEventListener("click", () => {
      this._count++;
      this._button.textContent = `Count: ${this._count}`;

      // Dispatch custom event
      this.dispatchEvent(
        new CustomEvent("count-changed", {
          detail: { count: this._count },
          bubbles: true, // Event bubble lên parent
          composed: true, // Event vượt qua Shadow DOM boundary
        })
      );
    });
  }
}

customElements.define("counter-button", CounterButton);
```

```html
<counter-button></counter-button>

<script>
  const counter = document.querySelector("counter-button");

  // Listen for custom event
  counter.addEventListener("count-changed", (e) => {
    console.log("Count is now:", e.detail.count);
  });

  // Hoặc listen ở document level (vì bubbles: true, composed: true)
  document.addEventListener("count-changed", (e) => {
    console.log("Count changed somewhere:", e.detail.count);
  });
</script>
```

### 📊 Event Options

| Option             | Mô tả                              |
| ------------------ | ---------------------------------- |
| `bubbles: true`    | Event bubble lên parent elements   |
| `composed: true`   | Event vượt qua Shadow DOM boundary |
| `cancelable: true` | Có thể preventDefault()            |

### 🔧 Event Delegation trong Shadow DOM

```javascript
class TodoList extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                ul { list-style: none; padding: 0; }
                li { padding: 10px; border-bottom: 1px solid #eee; }
                button { margin-left: 10px; }
            </style>
            <ul id="list"></ul>
        `;

    this._list = this.shadowRoot.getElementById("list");
  }

  connectedCallback() {
    // Event delegation - listen ở parent
    this._list.addEventListener("click", (e) => {
      if (e.target.matches("button.delete")) {
        const li = e.target.closest("li");
        const id = li.dataset.id;

        this.dispatchEvent(
          new CustomEvent("item-delete", {
            detail: { id },
            bubbles: true,
            composed: true,
          })
        );

        li.remove();
      }
    });
  }

  addItem(id, text) {
    const li = document.createElement("li");
    li.dataset.id = id;
    li.innerHTML = `
            <span>${text}</span>
            <button class="delete">Delete</button>
        `;
    this._list.appendChild(li);
  }
}

customElements.define("todo-list", TodoList);
```

---

## 9. Styling Web Components

### 🎨 :host Selector

```javascript
class StyledBox extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                /* Style cho host element */
                :host {
                    display: block;
                    padding: 20px;
                    background: #f0f0f0;
                    border-radius: 8px;
                }
                
                /* Style khi host có attribute/class cụ thể */
                :host([theme="dark"]) {
                    background: #333;
                    color: white;
                }
                
                :host(.highlighted) {
                    border: 2px solid gold;
                }
                
                /* Style dựa trên context (parent) */
                :host-context(.dark-mode) {
                    background: #222;
                    color: #fff;
                }
                
                /* Style cho slotted content */
                ::slotted(h1) {
                    color: #333;
                    margin: 0;
                }
                
                ::slotted(p) {
                    color: #666;
                }
            </style>
            <slot></slot>
        `;
  }
}

customElements.define("styled-box", StyledBox);
```

```html
<!-- Normal -->
<styled-box>
  <h1>Title</h1>
  <p>Content</p>
</styled-box>

<!-- Dark theme -->
<styled-box theme="dark">
  <h1>Dark Title</h1>
</styled-box>

<!-- With class -->
<styled-box class="highlighted">
  <h1>Highlighted</h1>
</styled-box>
```

### 🎨 CSS Custom Properties (CSS Variables)

CSS Variables **xuyên qua** Shadow DOM boundary!

```javascript
class ThemeableCard extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    /* Default values */
                    --card-bg: white;
                    --card-color: #333;
                    --card-padding: 20px;
                    --card-radius: 8px;
                    
                    display: block;
                    background: var(--card-bg);
                    color: var(--card-color);
                    padding: var(--card-padding);
                    border-radius: var(--card-radius);
                    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                }
                
                h2 {
                    color: var(--card-title-color, var(--card-color));
                    margin: 0 0 10px 0;
                }
            </style>
            <h2><slot name="title">Card Title</slot></h2>
            <div><slot></slot></div>
        `;
  }
}

customElements.define("themeable-card", ThemeableCard);
```

```html
<style>
  /* Override CSS variables từ bên ngoài */
  themeable-card {
    --card-bg: #1a1a2e;
    --card-color: #eee;
    --card-title-color: #00d9ff;
    --card-padding: 30px;
  }

  /* Hoặc global theme */
  :root {
    --card-bg: #f5f5f5;
    --card-radius: 16px;
  }
</style>

<themeable-card>
  <span slot="title">Custom Styled Card</span>
  <p>This card uses CSS custom properties for theming!</p>
</themeable-card>
```

### 🎨 CSS Parts (::part)

```javascript
class FancyButton extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                .button {
                    padding: 10px 20px;
                    border: none;
                    border-radius: 5px;
                    cursor: pointer;
                    display: inline-flex;
                    align-items: center;
                    gap: 8px;
                }
                .icon {
                    width: 20px;
                    height: 20px;
                }
            </style>
            <button class="button" part="button">
                <span class="icon" part="icon">
                    <slot name="icon">🔘</slot>
                </span>
                <span part="label">
                    <slot>Click me</slot>
                </span>
            </button>
        `;
  }
}

customElements.define("fancy-button", FancyButton);
```

```html
<style>
  /* Style parts từ bên ngoài */
  fancy-button::part(button) {
    background: linear-gradient(45deg, #667eea, #764ba2);
    color: white;
    font-size: 16px;
  }

  fancy-button::part(button):hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
  }

  fancy-button::part(icon) {
    font-size: 20px;
  }

  fancy-button::part(label) {
    font-weight: bold;
  }
</style>

<fancy-button>
  <span slot="icon">🚀</span>
  Launch
</fancy-button>
```

---

## 10. Ví dụ thực tế

### 🔧 Ví dụ 1: Modal Dialog Component

```javascript
class ModalDialog extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: none;
                }
                
                :host([open]) {
                    display: block;
                }
                
                .overlay {
                    position: fixed;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 100%;
                    background: rgba(0, 0, 0, 0.5);
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    z-index: 1000;
                }
                
                .modal {
                    background: white;
                    border-radius: 12px;
                    max-width: 500px;
                    width: 90%;
                    max-height: 80vh;
                    overflow: auto;
                    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
                    animation: slideIn 0.3s ease;
                }
                
                @keyframes slideIn {
                    from {
                        opacity: 0;
                        transform: translateY(-20px);
                    }
                    to {
                        opacity: 1;
                        transform: translateY(0);
                    }
                }
                
                .header {
                    padding: 20px;
                    border-bottom: 1px solid #eee;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                }
                
                .header h2 {
                    margin: 0;
                    font-size: 1.25rem;
                }
                
                .close-btn {
                    background: none;
                    border: none;
                    font-size: 24px;
                    cursor: pointer;
                    color: #666;
                    padding: 0;
                    line-height: 1;
                }
                
                .close-btn:hover {
                    color: #333;
                }
                
                .body {
                    padding: 20px;
                }
                
                .footer {
                    padding: 15px 20px;
                    border-top: 1px solid #eee;
                    display: flex;
                    justify-content: flex-end;
                    gap: 10px;
                }
            </style>
            
            <div class="overlay" part="overlay">
                <div class="modal" part="modal">
                    <div class="header" part="header">
                        <h2><slot name="title">Modal Title</slot></h2>
                        <button class="close-btn" part="close-btn">&times;</button>
                    </div>
                    <div class="body" part="body">
                        <slot></slot>
                    </div>
                    <div class="footer" part="footer">
                        <slot name="footer"></slot>
                    </div>
                </div>
            </div>
        `;
  }

  connectedCallback() {
    // Close button
    this.shadowRoot.querySelector(".close-btn").addEventListener("click", () => {
      this.close();
    });

    // Click overlay to close
    this.shadowRoot.querySelector(".overlay").addEventListener("click", (e) => {
      if (e.target.classList.contains("overlay")) {
        this.close();
      }
    });

    // ESC key to close
    this._handleKeydown = (e) => {
      if (e.key === "Escape" && this.hasAttribute("open")) {
        this.close();
      }
    };
    document.addEventListener("keydown", this._handleKeydown);
  }

  disconnectedCallback() {
    document.removeEventListener("keydown", this._handleKeydown);
  }

  open() {
    this.setAttribute("open", "");
    document.body.style.overflow = "hidden";
    this.dispatchEvent(new CustomEvent("modal-open"));
  }

  close() {
    this.removeAttribute("open");
    document.body.style.overflow = "";
    this.dispatchEvent(new CustomEvent("modal-close"));
  }

  // Toggle
  toggle() {
    this.hasAttribute("open") ? this.close() : this.open();
  }
}

customElements.define("modal-dialog", ModalDialog);
```

```html
<!-- Sử dụng -->
<button id="openModal">Open Modal</button>

<modal-dialog id="myModal">
  <span slot="title">Confirm Action</span>

  <p>Are you sure you want to proceed with this action?</p>
  <p>This cannot be undone.</p>

  <div slot="footer">
    <button onclick="myModal.close()">Cancel</button>
    <button onclick="handleConfirm()">Confirm</button>
  </div>
</modal-dialog>

<script>
  const modal = document.getElementById("myModal");

  document.getElementById("openModal").addEventListener("click", () => {
    modal.open();
  });

  modal.addEventListener("modal-close", () => {
    console.log("Modal closed");
  });

  function handleConfirm() {
    console.log("Confirmed!");
    modal.close();
  }
</script>
```

---

### 🔧 Ví dụ 2: Tabs Component

```javascript
class TabGroup extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                }
                
                .tabs {
                    display: flex;
                    border-bottom: 2px solid #eee;
                    gap: 5px;
                }
                
                .tab {
                    padding: 12px 24px;
                    border: none;
                    background: none;
                    cursor: pointer;
                    font-size: 14px;
                    color: #666;
                    border-bottom: 2px solid transparent;
                    margin-bottom: -2px;
                    transition: all 0.2s;
                }
                
                .tab:hover {
                    color: #333;
                    background: #f5f5f5;
                }
                
                .tab.active {
                    color: var(--tab-active-color, #2196F3);
                    border-bottom-color: var(--tab-active-color, #2196F3);
                    font-weight: 500;
                }
                
                .panels {
                    padding: 20px 0;
                }
                
                ::slotted(tab-panel) {
                    display: none;
                }
                
                ::slotted(tab-panel[active]) {
                    display: block;
                }
            </style>
            
            <div class="tabs" role="tablist" part="tabs"></div>
            <div class="panels" part="panels">
                <slot></slot>
            </div>
        `;

    this._tabsContainer = this.shadowRoot.querySelector(".tabs");
  }

  connectedCallback() {
    // Wait for children to be parsed
    requestAnimationFrame(() => {
      this._setupTabs();
    });

    // Listen for slot changes
    this.shadowRoot.querySelector("slot").addEventListener("slotchange", () => {
      this._setupTabs();
    });
  }

  _setupTabs() {
    const panels = this.querySelectorAll("tab-panel");
    this._tabsContainer.innerHTML = "";

    panels.forEach((panel, index) => {
      const tab = document.createElement("button");
      tab.className = "tab";
      tab.textContent = panel.getAttribute("label") || `Tab ${index + 1}`;
      tab.setAttribute("role", "tab");
      tab.setAttribute("part", "tab");

      if (panel.hasAttribute("active") || index === 0) {
        tab.classList.add("active");
        panel.setAttribute("active", "");
      } else {
        panel.removeAttribute("active");
      }

      tab.addEventListener("click", () => {
        this._activateTab(index);
      });

      this._tabsContainer.appendChild(tab);
    });
  }

  _activateTab(index) {
    const tabs = this._tabsContainer.querySelectorAll(".tab");
    const panels = this.querySelectorAll("tab-panel");

    tabs.forEach((tab, i) => {
      tab.classList.toggle("active", i === index);
    });

    panels.forEach((panel, i) => {
      if (i === index) {
        panel.setAttribute("active", "");
      } else {
        panel.removeAttribute("active");
      }
    });

    this.dispatchEvent(
      new CustomEvent("tab-change", {
        detail: { index, panel: panels[index] },
      })
    );
  }
}

class TabPanel extends HTMLElement {
  constructor() {
    super();
  }
}

customElements.define("tab-group", TabGroup);
customElements.define("tab-panel", TabPanel);
```

```html
<style>
  tab-group {
    --tab-active-color: #e91e63;
  }
</style>

<tab-group>
  <tab-panel label="Profile" active>
    <h3>Profile Content</h3>
    <p>This is the profile tab content.</p>
  </tab-panel>

  <tab-panel label="Settings">
    <h3>Settings Content</h3>
    <p>This is the settings tab content.</p>
  </tab-panel>

  <tab-panel label="Notifications">
    <h3>Notifications Content</h3>
    <p>This is the notifications tab content.</p>
  </tab-panel>
</tab-group>

<script>
  document.querySelector("tab-group").addEventListener("tab-change", (e) => {
    console.log("Tab changed to:", e.detail.index);
  });
</script>
```

---

### 🔧 Ví dụ 3: Toast Notification Component

```javascript
class ToastContainer extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    position: fixed;
                    top: 20px;
                    right: 20px;
                    z-index: 9999;
                    display: flex;
                    flex-direction: column;
                    gap: 10px;
                }
            </style>
            <slot></slot>
        `;
  }

  // Static method để show toast từ bất kỳ đâu
  static show(message, type = "info", duration = 3000) {
    let container = document.querySelector("toast-container");

    if (!container) {
      container = document.createElement("toast-container");
      document.body.appendChild(container);
    }

    const toast = document.createElement("toast-notification");
    toast.setAttribute("type", type);
    toast.setAttribute("duration", duration);
    toast.textContent = message;

    container.appendChild(toast);
  }
}

class ToastNotification extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: flex;
                    align-items: center;
                    gap: 12px;
                    padding: 14px 20px;
                    border-radius: 8px;
                    color: white;
                    font-size: 14px;
                    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
                    animation: slideIn 0.3s ease;
                    min-width: 250px;
                    max-width: 400px;
                }
                
                :host([type="success"]) { background: #4CAF50; }
                :host([type="error"]) { background: #f44336; }
                :host([type="warning"]) { background: #ff9800; }
                :host([type="info"]) { background: #2196F3; }
                
                @keyframes slideIn {
                    from {
                        opacity: 0;
                        transform: translateX(100%);
                    }
                    to {
                        opacity: 1;
                        transform: translateX(0);
                    }
                }
                
                @keyframes slideOut {
                    from {
                        opacity: 1;
                        transform: translateX(0);
                    }
                    to {
                        opacity: 0;
                        transform: translateX(100%);
                    }
                }
                
                .icon {
                    font-size: 20px;
                }
                
                .message {
                    flex: 1;
                }
                
                .close {
                    background: none;
                    border: none;
                    color: white;
                    cursor: pointer;
                    padding: 0;
                    font-size: 18px;
                    opacity: 0.8;
                }
                
                .close:hover {
                    opacity: 1;
                }
            </style>
            
            <span class="icon"></span>
            <span class="message"><slot></slot></span>
            <button class="close">&times;</button>
        `;
  }

  connectedCallback() {
    const type = this.getAttribute("type") || "info";
    const duration = parseInt(this.getAttribute("duration")) || 3000;

    // Set icon based on type
    const icons = {
      success: "✓",
      error: "✕",
      warning: "⚠",
      info: "ℹ",
    };
    this.shadowRoot.querySelector(".icon").textContent = icons[type];

    // Close button
    this.shadowRoot.querySelector(".close").addEventListener("click", () => {
      this._close();
    });

    // Auto close
    if (duration > 0) {
      this._timeout = setTimeout(() => {
        this._close();
      }, duration);
    }
  }

  disconnectedCallback() {
    if (this._timeout) {
      clearTimeout(this._timeout);
    }
  }

  _close() {
    this.style.animation = "slideOut 0.3s ease forwards";
    setTimeout(() => {
      this.remove();
    }, 300);
  }
}

customElements.define("toast-container", ToastContainer);
customElements.define("toast-notification", ToastNotification);
```

```html
<!-- Sử dụng -->
<button onclick="ToastContainer.show('Profile saved successfully!', 'success')">Success Toast</button>

<button onclick="ToastContainer.show('Something went wrong!', 'error')">Error Toast</button>

<button onclick="ToastContainer.show('Please check your input', 'warning')">Warning Toast</button>

<button onclick="ToastContainer.show('New update available', 'info', 5000)">Info Toast (5s)</button>
```

---

## 11. Best Practices

### ✅ DO's

```javascript
class GoodComponent extends HTMLElement {
  // ✅ Khai báo observed attributes
  static get observedAttributes() {
    return ["value", "disabled"];
  }

  constructor() {
    super();

    // ✅ Attach shadow DOM trong constructor
    this.attachShadow({ mode: "open" });

    // ✅ Khởi tạo state
    this._value = "";
  }

  connectedCallback() {
    // ✅ Setup trong connectedCallback
    this.render();
    this._addEventListeners();
  }

  disconnectedCallback() {
    // ✅ Cleanup khi remove
    this._removeEventListeners();
  }

  // ✅ Reflect properties to attributes
  get value() {
    return this._value;
  }
  set value(val) {
    this._value = val;
    this.setAttribute("value", val);
  }

  // ✅ Dispatch custom events với composed: true
  _emitChange() {
    this.dispatchEvent(
      new CustomEvent("change", {
        detail: { value: this._value },
        bubbles: true,
        composed: true,
      })
    );
  }

  // ✅ Sử dụng CSS custom properties cho theming
  render() {
    this.shadowRoot.innerHTML = `
            <style>
                :host {
                    --primary-color: #2196F3;
                    display: block;
                }
                .btn {
                    background: var(--primary-color);
                }
            </style>
            <button class="btn"><slot></slot></button>
        `;
  }
}
```

### ❌ DON'Ts

```javascript
class BadComponent extends HTMLElement {
    constructor() {
        super();

        // ❌ Đọc attributes trong constructor
        const value = this.getAttribute('value');

        // ❌ Thêm children trong constructor
        this.innerHTML = '<div>Content</div>';

        // ❌ Fetch data trong constructor
        fetch('/api/data').then(...);
    }

    connectedCallback() {
        // ❌ Không cleanup event listeners
        window.addEventListener('resize', this.handleResize);

        // ❌ Không check nếu đã connected trước đó
        this.innerHTML = '...';  // Có thể gọi nhiều lần
    }

    // ❌ Không observe attributes
    // static get observedAttributes() { ... }

    // ❌ Không reflect properties
    set value(val) {
        this._value = val;
        // Thiếu: this.setAttribute('value', val);
    }
}
```

### 📋 Checklist

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    WEB COMPONENTS CHECKLIST                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  □ Tên element có dấu gạch ngang (my-component)                        │
│  □ Gọi super() đầu tiên trong constructor                              │
│  □ attachShadow() trong constructor                                     │
│  □ Không đọc attributes/children trong constructor                      │
│  □ Setup logic trong connectedCallback                                  │
│  □ Cleanup trong disconnectedCallback                                   │
│  □ Khai báo observedAttributes cho attributes cần watch                │
│  □ Reflect properties to attributes khi cần                            │
│  □ Sử dụng CSS custom properties cho theming                           │
│  □ Export CSS parts cho external styling                               │
│  □ Dispatch events với bubbles và composed                             │
│  □ Handle accessibility (ARIA, keyboard navigation)                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Tài liệu tham khảo

- [MDN Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [Web Components Spec](https://www.webcomponents.org/)
- [Google Web Fundamentals](https://developers.google.com/web/fundamentals/web-components)
- [Open Web Components](https://open-wc.org/)
- [Lit Library](https://lit.dev/) - Thư viện giúp viết Web Components dễ hơn
