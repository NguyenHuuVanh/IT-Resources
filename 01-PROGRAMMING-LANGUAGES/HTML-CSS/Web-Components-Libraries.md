# Web Components Libraries & Frameworks

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Lit](#2-lit)
3. [Stencil](#3-stencil)
4. [FAST Element](#4-fast-element)
5. [Shoelace](#5-shoelace)
6. [So sánh các thư viện](#6-so-sánh-các-thư-viện)
7. [Web Components với Frameworks](#7-web-components-với-frameworks)

---

## 1. Tổng quan

### 🎯 Tại sao cần thư viện?

Vanilla Web Components có thể verbose. Các thư viện giúp:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LỢI ÍCH CỦA THƯ VIỆN                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Reactive properties      Tự động re-render khi data thay đổi       │
│  ✓ Declarative templates    HTML templates dễ đọc hơn                  │
│  ✓ Less boilerplate         Ít code hơn cho cùng chức năng             │
│  ✓ Better DX                TypeScript support, tooling tốt hơn        │
│  ✓ Performance              Optimized rendering, batched updates       │
│  ✓ Ecosystem                Components có sẵn, plugins                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📊 So sánh nhanh

| Thư viện     | Size   | Đặc điểm chính              |
| ------------ | ------ | --------------------------- |
| **Lit**      | ~5KB   | Google, reactive, fast      |
| **Stencil**  | ~14KB  | Ionic, compiler, TypeScript |
| **FAST**     | ~10KB  | Microsoft, design system    |
| **Shoelace** | UI lib | Component library sẵn       |

---

## 2. Lit

### 🎯 Giới thiệu

Lit là thư viện nhẹ từ Google để xây dựng Web Components với reactive properties và declarative templates.

### 🔧 Cài đặt

```bash
npm install lit
```

### 💻 Cú pháp cơ bản

```javascript
import { LitElement, html, css } from 'lit';
import { customElement, property, state } from 'lit/decorators.js';

@customElement('my-counter')
export class MyCounter extends LitElement {
    // Scoped styles
    static styles = css`
        :host {
            display: block;
            padding: 16px;
            font-family: sans-serif;
        }

        .count {
            font-size: 48px;
            font-weight: bold;
            color: var(--counter-color, #333);
        }

        button {
            padding: 8px 16px;
            margin: 4px;
            font-size: 16px;
            cursor: pointer;
            border: none;
            border-radius: 4px;
            background: #4CAF50;
            color: white;
        }

        button:hover {
            background: #45a049;
        }
    `;

    // Reactive property (từ attribute)
    @property({ type: Number })
    count = 0;

    // Internal state (không reflect to attribute)
    @state()
    private _lastAction = '';

    render() {
        return html`
            <div class="count">${this.count}</div>
            <p>Last action: ${this._lastAction || 'None'}</p>

            <button @click=${this._decrement}>-</button>
            <button @click=${this._increment}>+</button>
            <button @click=${this._reset}>Reset</button>
        `;
    }

    private _increment() {
        this.count++;
        this._lastAction = 'Increment';
        this._dispatchChange();
    }

    private _decrement() {
        this.count--;
        this._lastAction = 'Decrement';
        this._dispatchChange();
    }

    private _reset() {
        this.count = 0;
        this._lastAction = 'Reset';
        this._dispatchChange();
    }

    private _dispatchChange() {
        this.dispatchEvent(new CustomEvent('count-changed', {
            detail: { count: this.count },
            bubbles: true,
            composed: true
        }));
    }
}
```

```html
<!-- Sử dụng -->
<my-counter count="10"></my-counter>

<script>
  document.querySelector("my-counter").addEventListener("count-changed", (e) => {
    console.log("Count:", e.detail.count);
  });
</script>
```

### 🔧 Lit Directives

```javascript
import { LitElement, html } from "lit";
import { repeat } from "lit/directives/repeat.js";
import { classMap } from "lit/directives/class-map.js";
import { styleMap } from "lit/directives/style-map.js";
import { when } from "lit/directives/when.js";
import { ifDefined } from "lit/directives/if-defined.js";

class DirectivesDemo extends LitElement {
  @property({ type: Array }) items = [];
  @property({ type: Boolean }) isActive = false;
  @property({ type: String }) color = "blue";
  @property({ type: String }) href = undefined;

  render() {
    // classMap - conditional classes
    const classes = {
      card: true,
      active: this.isActive,
      highlighted: this.count > 10,
    };

    // styleMap - dynamic styles
    const styles = {
      color: this.color,
      fontWeight: this.isActive ? "bold" : "normal",
    };

    return html`
      <!-- classMap -->
      <div class=${classMap(classes)}>Content</div>

      <!-- styleMap -->
      <p style=${styleMap(styles)}>Styled text</p>

      <!-- repeat - efficient list rendering -->
      <ul>
        ${repeat(
          this.items,
          (item) => item.id, // key function
          (item, index) => html` <li>${index}: ${item.name}</li> `
        )}
      </ul>

      <!-- when - conditional rendering -->
      ${when(
        this.isActive,
        () => html`<p>Active content</p>`,
        () => html`<p>Inactive content</p>`
      )}

      <!-- ifDefined - only set attribute if defined -->
      <a href=${ifDefined(this.href)}>Link</a>
    `;
  }
}
```

### 🔄 Lifecycle trong Lit

```javascript
import { LitElement, html } from "lit";

class LifecycleDemo extends LitElement {
  @property() name = "";

  // Gọi khi component được tạo
  constructor() {
    super();
    console.log("constructor");
  }

  // Gọi khi connected to DOM
  connectedCallback() {
    super.connectedCallback();
    console.log("connectedCallback");
  }

  // Gọi khi disconnected from DOM
  disconnectedCallback() {
    super.disconnectedCallback();
    console.log("disconnectedCallback");
  }

  // Gọi khi property thay đổi, trước render
  willUpdate(changedProperties) {
    console.log("willUpdate", changedProperties);
  }

  // Render template
  render() {
    console.log("render");
    return html`<p>Hello ${this.name}</p>`;
  }

  // Gọi sau lần render đầu tiên
  firstUpdated(changedProperties) {
    console.log("firstUpdated", changedProperties);
    // Good place to: focus elements, measure DOM, etc.
  }

  // Gọi sau mỗi lần render
  updated(changedProperties) {
    console.log("updated", changedProperties);
  }
}
```

### 📦 Lit Task (Async Data)

```javascript
import { LitElement, html } from 'lit';
import { Task } from '@lit/task';

class UserProfile extends LitElement {
    @property() userId = '';

    private _userTask = new Task(this, {
        task: async ([userId]) => {
            const response = await fetch(`/api/users/${userId}`);
            if (!response.ok) throw new Error('Failed to fetch');
            return response.json();
        },
        args: () => [this.userId]
    });

    render() {
        return this._userTask.render({
            pending: () => html`<p>Loading...</p>`,
            complete: (user) => html`
                <div class="profile">
                    <h2>${user.name}</h2>
                    <p>${user.email}</p>
                </div>
            `,
            error: (e) => html`<p>Error: ${e.message}</p>`
        });
    }
}
```

---

## 3. Stencil

### 🎯 Giới thiệu

Stencil là compiler từ Ionic để build Web Components với TypeScript và JSX.

### 🔧 Cài đặt

```bash
npm init stencil
# Chọn "component" để tạo component library
```

### 💻 Cú pháp cơ bản

```tsx
// my-counter.tsx
import { Component, Prop, State, Event, EventEmitter, h } from "@stencil/core";

@Component({
  tag: "my-counter",
  styleUrl: "my-counter.css",
  shadow: true,
})
export class MyCounter {
  // Public property (từ attribute)
  @Prop() initialCount: number = 0;

  // Internal state
  @State() count: number;

  // Custom event
  @Event() countChanged: EventEmitter<number>;

  // Lifecycle
  componentWillLoad() {
    this.count = this.initialCount;
  }

  private increment = () => {
    this.count++;
    this.countChanged.emit(this.count);
  };

  private decrement = () => {
    this.count--;
    this.countChanged.emit(this.count);
  };

  // JSX render
  render() {
    return (
      <div class="counter">
        <span class="count">{this.count}</span>
        <div class="buttons">
          <button onClick={this.decrement}>-</button>
          <button onClick={this.increment}>+</button>
        </div>
      </div>
    );
  }
}
```

```css
/* my-counter.css */
:host {
  display: block;
  font-family: sans-serif;
}

.count {
  font-size: 48px;
  font-weight: bold;
}

button {
  padding: 8px 16px;
  margin: 4px;
  font-size: 18px;
  cursor: pointer;
}
```

### 🔧 Stencil Decorators

```tsx
import { Component, Prop, State, Event, EventEmitter, Method, Watch, Element, h } from "@stencil/core";

@Component({
  tag: "advanced-component",
  shadow: true,
})
export class AdvancedComponent {
  // Reference to host element
  @Element() el: HTMLElement;

  // Props với options
  @Prop({ reflect: true, mutable: true })
  value: string = "";

  @Prop({ attribute: "is-disabled" })
  disabled: boolean = false;

  // State
  @State() internalState: string = "";

  // Events
  @Event({
    eventName: "valueChange",
    bubbles: true,
    composed: true,
  })
  valueChange: EventEmitter<string>;

  // Watch property changes
  @Watch("value")
  valueChanged(newValue: string, oldValue: string) {
    console.log("Value changed:", oldValue, "->", newValue);
  }

  // Public method (có thể gọi từ bên ngoài)
  @Method()
  async focus() {
    const input = this.el.shadowRoot.querySelector("input");
    input?.focus();
  }

  @Method()
  async getValue(): Promise<string> {
    return this.value;
  }

  render() {
    return <input type="text" value={this.value} disabled={this.disabled} onInput={(e) => this.handleInput(e)} />;
  }

  private handleInput(e: Event) {
    const target = e.target as HTMLInputElement;
    this.value = target.value;
    this.valueChange.emit(this.value);
  }
}
```

### 🔄 Stencil Lifecycle

```tsx
@Component({ tag: "lifecycle-demo", shadow: true })
export class LifecycleDemo {
  // ═══════════════════════════════════════════
  // LOAD PHASE
  // ═══════════════════════════════════════════

  // Trước khi component load (chưa render)
  componentWillLoad() {
    console.log("componentWillLoad");
    // Good for: fetch initial data
  }

  // Sau khi component load lần đầu
  componentDidLoad() {
    console.log("componentDidLoad");
    // Good for: DOM manipulation, third-party libs
  }

  // ═══════════════════════════════════════════
  // UPDATE PHASE
  // ═══════════════════════════════════════════

  // Trước mỗi lần render (trừ lần đầu)
  componentWillUpdate() {
    console.log("componentWillUpdate");
  }

  // Sau mỗi lần render (trừ lần đầu)
  componentDidUpdate() {
    console.log("componentDidUpdate");
  }

  // Trước mỗi lần render (kể cả lần đầu)
  componentWillRender() {
    console.log("componentWillRender");
  }

  // Sau mỗi lần render (kể cả lần đầu)
  componentDidRender() {
    console.log("componentDidRender");
  }

  // ═══════════════════════════════════════════
  // UNLOAD PHASE
  // ═══════════════════════════════════════════

  // Khi component bị remove
  disconnectedCallback() {
    console.log("disconnectedCallback");
    // Good for: cleanup, remove listeners
  }

  render() {
    return <div>Lifecycle Demo</div>;
  }
}
```

---

## 4. FAST Element

### 🎯 Giới thiệu

FAST Element là thư viện từ Microsoft để xây dựng Web Components, là nền tảng của Fluent UI Web Components.

### 🔧 Cài đặt

```bash
npm install @microsoft/fast-element
```

### 💻 Cú pháp cơ bản

```typescript
import { FASTElement, customElement, attr, html, css } from "@microsoft/fast-element";

const template = html<MyCounter>`
  <div class="counter">
    <span class="count">${(x) => x.count}</span>
    <button @click="${(x) => x.decrement()}">-</button>
    <button @click="${(x) => x.increment()}">+</button>
  </div>
`;

const styles = css`
  :host {
    display: block;
    font-family: sans-serif;
  }

  .count {
    font-size: 48px;
    font-weight: bold;
  }

  button {
    padding: 8px 16px;
    margin: 4px;
  }
`;

@customElement({
  name: "my-counter",
  template,
  styles,
})
export class MyCounter extends FASTElement {
  @attr({ mode: "fromView" })
  count: number = 0;

  increment() {
    this.count++;
    this.$emit("count-changed", this.count);
  }

  decrement() {
    this.count--;
    this.$emit("count-changed", this.count);
  }
}
```

### 🔧 FAST Bindings

```typescript
import { html, when, repeat } from "@microsoft/fast-element";

const template = html<MyComponent>`
  <!-- One-way binding -->
  <p>${(x) => x.message}</p>

  <!-- Two-way binding -->
  <input :value="${(x) => x.inputValue}" @input="${(x, c) => (x.inputValue = c.event.target.value)}" />

  <!-- Attribute binding -->
  <button ?disabled="${(x) => x.isDisabled}">Click</button>
  <div class="${(x) => (x.isActive ? "active" : "")}">Content</div>

  <!-- Event binding -->
  <button @click="${(x) => x.handleClick()}">Click</button>
  <button @click="${(x, c) => x.handleClickWithEvent(c.event)}">Click</button>

  <!-- Conditional rendering -->
  ${when((x) => x.isLoading, html`<p>Loading...</p>`)} ${when((x) => !x.isLoading, html`<p>Content loaded</p>`)}

  <!-- List rendering -->
  <ul>
    ${repeat((x) => x.items, html<Item>` <li>${(x) => x.name}</li> `)}
  </ul>

  <!-- Slotted content -->
  <slot></slot>
  <slot name="header"></slot>
`;
```

---

## 5. Shoelace

### 🎯 Giới thiệu

Shoelace là thư viện UI components được xây dựng bằng Web Components, có thể dùng với bất kỳ framework nào.

### 🔧 Cài đặt

```bash
npm install @shoelace-style/shoelace
```

```javascript
// Import components
import "@shoelace-style/shoelace/dist/themes/light.css";
import "@shoelace-style/shoelace/dist/components/button/button.js";
import "@shoelace-style/shoelace/dist/components/input/input.js";
import "@shoelace-style/shoelace/dist/components/dialog/dialog.js";
```

### 💻 Sử dụng

```html
<!-- Button -->
<sl-button variant="primary">Primary</sl-button>
<sl-button variant="success">Success</sl-button>
<sl-button variant="danger" outline>Danger Outline</sl-button>
<sl-button loading>Loading</sl-button>

<!-- Input -->
<sl-input label="Name" placeholder="Enter your name" clearable></sl-input>
<sl-input type="password" label="Password" password-toggle></sl-input>
<sl-input type="email" label="Email" help-text="We'll never share your email"></sl-input>

<!-- Select -->
<sl-select label="Favorite Food" clearable>
  <sl-option value="pizza">Pizza</sl-option>
  <sl-option value="burger">Burger</sl-option>
  <sl-option value="sushi">Sushi</sl-option>
</sl-select>

<!-- Dialog -->
<sl-dialog label="Confirm" class="dialog-confirm">
  Are you sure you want to proceed?
  <sl-button slot="footer" variant="primary">Yes</sl-button>
  <sl-button slot="footer" variant="default">No</sl-button>
</sl-dialog>

<sl-button onclick="document.querySelector('.dialog-confirm').show()"> Open Dialog </sl-button>

<!-- Card -->
<sl-card class="card-overview">
  <img slot="image" src="image.jpg" alt="Card image" />
  <strong>Card Title</strong>
  <p>This is the card content.</p>
  <sl-button slot="footer" variant="primary">Action</sl-button>
</sl-card>

<!-- Tabs -->
<sl-tab-group>
  <sl-tab slot="nav" panel="general">General</sl-tab>
  <sl-tab slot="nav" panel="settings">Settings</sl-tab>
  <sl-tab slot="nav" panel="advanced">Advanced</sl-tab>

  <sl-tab-panel name="general">General content</sl-tab-panel>
  <sl-tab-panel name="settings">Settings content</sl-tab-panel>
  <sl-tab-panel name="advanced">Advanced content</sl-tab-panel>
</sl-tab-group>
```

### 🎨 Theming Shoelace

```css
/* Custom theme với CSS variables */
:root {
  --sl-color-primary-50: #e3f2fd;
  --sl-color-primary-100: #bbdefb;
  --sl-color-primary-500: #2196f3;
  --sl-color-primary-600: #1e88e5;

  --sl-border-radius-medium: 8px;
  --sl-font-sans: "Inter", sans-serif;

  --sl-input-height-medium: 44px;
  --sl-button-font-size-medium: 16px;
}

/* Dark mode */
.sl-theme-dark {
  --sl-color-neutral-0: #1a1a1a;
  --sl-color-neutral-50: #2a2a2a;
  --sl-color-neutral-100: #3a3a3a;
}
```

---

## 6. So sánh các thư viện

### 📊 Bảng so sánh chi tiết

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SO SÁNH CÁC THƯ VIỆN                                    │
├──────────────┬──────────────┬──────────────┬──────────────┬─────────────────────┤
│              │     Lit      │   Stencil    │    FAST      │     Shoelace        │
├──────────────┼──────────────┼──────────────┼──────────────┼─────────────────────┤
│ Maintainer   │   Google     │    Ionic     │  Microsoft   │    Community        │
│ Size (gzip)  │    ~5KB      │    ~14KB     │    ~10KB     │   UI Library        │
│ Language     │  TS/JS       │  TypeScript  │  TypeScript  │   Built on Lit      │
│ Template     │  Tagged      │     JSX      │   Tagged     │       N/A           │
│              │  Template    │              │   Template   │                     │
│ Decorators   │     ✓        │      ✓       │      ✓       │       N/A           │
│ SSR Support  │     ✓        │      ✓       │      ✓       │        ✓            │
│ React Wrapper│   Manual     │   Built-in   │   Built-in   │     Built-in        │
│ Learning     │    Easy      │   Medium     │   Medium     │      Easy           │
│ Use Case     │  General     │  Component   │   Design     │   Ready-to-use      │
│              │              │   Library    │   Systems    │   Components        │
└──────────────┴──────────────┴──────────────┴──────────────┴─────────────────────┘
```

### 🎯 Khi nào dùng thư viện nào?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HƯỚNG DẪN CHỌN THƯ VIỆN                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Lit:                                                                   │
│  └── Khi cần thư viện nhẹ, đơn giản                                    │
│  └── Khi đã quen với tagged template literals                          │
│  └── Khi cần performance tốt nhất                                      │
│                                                                         │
│  Stencil:                                                               │
│  └── Khi build component library cho nhiều frameworks                  │
│  └── Khi team quen với JSX (React background)                          │
│  └── Khi cần auto-generate React/Vue/Angular wrappers                  │
│                                                                         │
│  FAST:                                                                  │
│  └── Khi build design system                                           │
│  └── Khi cần tích hợp với Microsoft ecosystem                          │
│  └── Khi cần adaptive UI patterns                                      │
│                                                                         │
│  Shoelace:                                                              │
│  └── Khi cần UI components sẵn có                                      │
│  └── Khi không muốn build từ đầu                                       │
│  └── Khi cần components đẹp, accessible                                │
│                                                                         │
│  Vanilla Web Components:                                                │
│  └── Khi cần zero dependencies                                         │
│  └── Khi components đơn giản                                           │
│  └── Khi học/hiểu Web Components standards                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Web Components với Frameworks

### ⚛️ React

```jsx
// React 19+ hỗ trợ Web Components tốt hơn
// Với React < 19, cần wrapper

// Cách 1: Sử dụng trực tiếp (React 19+)
function App() {
  const handleCountChange = (e) => {
    console.log("Count:", e.detail.count);
  };

  return <my-counter count={10} onCountChanged={handleCountChange} />;
}

// Cách 2: Wrapper component (React < 19)
import { useRef, useEffect } from "react";

function MyCounterWrapper({ count, onCountChanged }) {
  const ref = useRef(null);

  useEffect(() => {
    const element = ref.current;

    const handler = (e) => onCountChanged?.(e.detail.count);
    element.addEventListener("count-changed", handler);

    return () => {
      element.removeEventListener("count-changed", handler);
    };
  }, [onCountChanged]);

  useEffect(() => {
    if (ref.current) {
      ref.current.count = count;
    }
  }, [count]);

  return <my-counter ref={ref} />;
}

// Cách 3: Dùng @lit/react
import { createComponent } from "@lit/react";
import { MyCounter } from "./my-counter";

export const MyCounterReact = createComponent({
  tagName: "my-counter",
  elementClass: MyCounter,
  react: React,
  events: {
    onCountChanged: "count-changed",
  },
});
```

### 💚 Vue

```vue
<!-- Vue 3 hỗ trợ Web Components tốt -->
<template>
  <my-counter :count="count" @count-changed="handleCountChange" />
</template>

<script setup>
import { ref } from "vue";

// Import Web Component
import "../components/my-counter.js";

const count = ref(10);

const handleCountChange = (e) => {
  console.log("Count:", e.detail.count);
  count.value = e.detail.count;
};
</script>

<!-- Cấu hình Vue để nhận diện custom elements -->
<!-- vite.config.js -->
<script>
export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          isCustomElement: (tag) => tag.includes("-"),
        },
      },
    }),
  ],
};
</script>
```

### 🅰️ Angular

```typescript
// app.module.ts
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from "@angular/core";

@NgModule({
  schemas: [CUSTOM_ELEMENTS_SCHEMA], // Cho phép custom elements
})
export class AppModule {}
```

```typescript
// app.component.ts
import { Component } from "@angular/core";
import "../components/my-counter.js";

@Component({
  selector: "app-root",
  template: ` <my-counter [count]="count" (count-changed)="onCountChange($event)"></my-counter> `,
})
export class AppComponent {
  count = 10;

  onCountChange(event: CustomEvent) {
    console.log("Count:", event.detail.count);
    this.count = event.detail.count;
  }
}
```

### 🟢 Svelte

```svelte
<!-- Svelte hỗ trợ Web Components native -->
<script>
    import '../components/my-counter.js';

    let count = 10;

    function handleCountChange(e) {
        count = e.detail.count;
    }
</script>

<my-counter
    {count}
    on:count-changed={handleCountChange}
/>
```

---

## 🔗 Tài liệu tham khảo

- [Lit Documentation](https://lit.dev/)
- [Stencil Documentation](https://stenciljs.com/)
- [FAST Documentation](https://www.fast.design/)
- [Shoelace Documentation](https://shoelace.style/)
- [Custom Elements Everywhere](https://custom-elements-everywhere.com/) - Framework compatibility
