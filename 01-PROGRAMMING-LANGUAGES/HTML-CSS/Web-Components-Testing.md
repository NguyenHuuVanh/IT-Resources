# Testing Web Components

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Testing với Web Test Runner](#2-testing-với-web-test-runner)
3. [Testing với Jest](#3-testing-với-jest)
4. [Testing Lit Components](#4-testing-lit-components)
5. [Testing Stencil Components](#5-testing-stencil-components)
6. [Best Practices](#6-best-practices)

---

## 1. Tổng quan

### 🎯 Các công cụ testing phổ biến

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TESTING TOOLS CHO WEB COMPONENTS                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Web Test Runner (@web/test-runner)                                    │
│  └── Chạy tests trong browser thật                                     │
│  └── Recommended cho Web Components                                    │
│  └── Hỗ trợ ES modules native                                          │
│                                                                         │
│  Jest + jsdom                                                           │
│  └── Phổ biến, ecosystem lớn                                           │
│  └── Cần polyfills cho Web Components                                  │
│  └── Không phải browser thật                                           │
│                                                                         │
│  Playwright / Cypress                                                   │
│  └── E2E testing                                                        │
│  └── Browser thật                                                       │
│  └── Chậm hơn unit tests                                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Testing với Web Test Runner

### 🔧 Cài đặt

```bash
npm install --save-dev @web/test-runner @esm-bundle/chai @open-wc/testing
```

### 📁 Cấu hình

```javascript
// web-test-runner.config.js
import { playwrightLauncher } from "@web/test-runner-playwright";

export default {
  files: "src/**/*.test.js",
  nodeResolve: true,
  browsers: [
    playwrightLauncher({ product: "chromium" }),
    playwrightLauncher({ product: "firefox" }),
    playwrightLauncher({ product: "webkit" }),
  ],
  testFramework: {
    config: {
      timeout: 3000,
    },
  },
};
```

```json
// package.json
{
  "scripts": {
    "test": "web-test-runner",
    "test:watch": "web-test-runner --watch"
  }
}
```

### 💻 Viết Tests

```javascript
// my-counter.test.js
import { html, fixture, expect, oneEvent } from "@open-wc/testing";
import "../src/my-counter.js";

describe("MyCounter", () => {
  it("renders with default count of 0", async () => {
    const el = await fixture(html`<my-counter></my-counter>`);

    expect(el.count).to.equal(0);
    expect(el.shadowRoot.querySelector(".count").textContent).to.equal("0");
  });

  it("renders with initial count from attribute", async () => {
    const el = await fixture(html`<my-counter count="5"></my-counter>`);

    expect(el.count).to.equal(5);
  });

  it("increments count when increment button clicked", async () => {
    const el = await fixture(html`<my-counter count="0"></my-counter>`);

    const button = el.shadowRoot.querySelector(".increment");
    button.click();

    await el.updateComplete;

    expect(el.count).to.equal(1);
  });

  it("decrements count when decrement button clicked", async () => {
    const el = await fixture(html`<my-counter count="5"></my-counter>`);

    const button = el.shadowRoot.querySelector(".decrement");
    button.click();

    await el.updateComplete;

    expect(el.count).to.equal(4);
  });

  it("fires count-changed event", async () => {
    const el = await fixture(html`<my-counter></my-counter>`);

    const button = el.shadowRoot.querySelector(".increment");

    setTimeout(() => button.click());

    const { detail } = await oneEvent(el, "count-changed");

    expect(detail.count).to.equal(1);
  });

  it("passes accessibility audit", async () => {
    const el = await fixture(html`<my-counter></my-counter>`);

    await expect(el).to.be.accessible();
  });
});
```

### 🔧 Testing Utilities từ @open-wc/testing

```javascript
import {
  html, // Tagged template cho fixtures
  fixture, // Tạo element trong DOM
  expect, // Chai expect
  oneEvent, // Wait for single event
  aTimeout, // Wait for timeout
  nextFrame, // Wait for next animation frame
  defineCE, // Define custom element
  unsafeStatic, // Unsafe static HTML
  triggerFocusFor, // Trigger focus
  triggerBlurFor, // Trigger blur
} from "@open-wc/testing";

describe("Testing Utilities", () => {
  it("fixture creates element", async () => {
    const el = await fixture(html`<my-element></my-element>`);
    expect(el).to.exist;
    expect(el.tagName.toLowerCase()).to.equal("my-element");
  });

  it("oneEvent waits for event", async () => {
    const el = await fixture(html`<my-button></my-button>`);

    setTimeout(() => el.click());
    const event = await oneEvent(el, "click");

    expect(event).to.exist;
  });

  it("aTimeout waits for specified time", async () => {
    const start = Date.now();
    await aTimeout(100);
    const elapsed = Date.now() - start;

    expect(elapsed).to.be.at.least(100);
  });

  it("nextFrame waits for animation frame", async () => {
    const el = await fixture(html`<my-element></my-element>`);
    el.setAttribute("animate", "");

    await nextFrame();

    // Animation should have started
  });
});
```

---

## 3. Testing với Jest

### 🔧 Cài đặt

```bash
npm install --save-dev jest @testing-library/dom jest-environment-jsdom
```

### 📁 Cấu hình

```javascript
// jest.config.js
module.exports = {
  testEnvironment: "jsdom",
  transform: {
    "^.+\\.js$": "babel-jest",
  },
  setupFilesAfterEnv: ["./jest.setup.js"],
  moduleNameMapper: {
    "^(\\.{1,2}/.*)\\.js$": "$1",
  },
};
```

```javascript
// jest.setup.js
// Polyfill cho Web Components trong jsdom
import "@webcomponents/webcomponentsjs/webcomponents-bundle.js";

// Custom element registry mock nếu cần
if (!window.customElements) {
  window.customElements = {
    define: jest.fn(),
    get: jest.fn(),
    whenDefined: jest.fn(() => Promise.resolve()),
  };
}
```

### 💻 Viết Tests với Jest

```javascript
// my-counter.test.js
import "../src/my-counter.js";

describe("MyCounter", () => {
  let element;

  beforeEach(() => {
    element = document.createElement("my-counter");
    document.body.appendChild(element);
  });

  afterEach(() => {
    element.remove();
  });

  it("should render with default count", () => {
    expect(element.count).toBe(0);
  });

  it("should update count when attribute changes", () => {
    element.setAttribute("count", "10");
    expect(element.count).toBe(10);
  });

  it("should increment count", async () => {
    const button = element.shadowRoot.querySelector(".increment");
    button.click();

    // Wait for update
    await new Promise((resolve) => setTimeout(resolve, 0));

    expect(element.count).toBe(1);
  });

  it("should dispatch event on change", () => {
    const handler = jest.fn();
    element.addEventListener("count-changed", handler);

    const button = element.shadowRoot.querySelector(".increment");
    button.click();

    expect(handler).toHaveBeenCalled();
    expect(handler.mock.calls[0][0].detail.count).toBe(1);
  });
});
```

---

## 4. Testing Lit Components

### 💻 Lit Component Test

```javascript
// user-card.js
import { LitElement, html, css } from "lit";

export class UserCard extends LitElement {
  static properties = {
    name: { type: String },
    email: { type: String },
    loading: { type: Boolean },
  };

  static styles = css`
    :host {
      display: block;
    }
    .card {
      padding: 16px;
      border: 1px solid #ddd;
    }
    .loading {
      opacity: 0.5;
    }
  `;

  constructor() {
    super();
    this.name = "";
    this.email = "";
    this.loading = false;
  }

  render() {
    return html`
      <div class="card ${this.loading ? "loading" : ""}">
        <h2>${this.name || "Unknown"}</h2>
        <p>${this.email || "No email"}</p>
        <button @click=${this._handleClick}>Contact</button>
      </div>
    `;
  }

  _handleClick() {
    this.dispatchEvent(
      new CustomEvent("contact", {
        detail: { name: this.name, email: this.email },
        bubbles: true,
        composed: true,
      })
    );
  }
}

customElements.define("user-card", UserCard);
```

```javascript
// user-card.test.js
import { html, fixture, expect, oneEvent } from "@open-wc/testing";
import "./user-card.js";

describe("UserCard", () => {
  it("renders with default values", async () => {
    const el = await fixture(html`<user-card></user-card>`);

    expect(el.shadowRoot.querySelector("h2").textContent).to.equal("Unknown");
    expect(el.shadowRoot.querySelector("p").textContent).to.equal("No email");
  });

  it("renders with provided name and email", async () => {
    const el = await fixture(html` <user-card name="John Doe" email="john@example.com"></user-card> `);

    expect(el.shadowRoot.querySelector("h2").textContent).to.equal("John Doe");
    expect(el.shadowRoot.querySelector("p").textContent).to.equal("john@example.com");
  });

  it("applies loading class when loading", async () => {
    const el = await fixture(html`<user-card loading></user-card>`);

    const card = el.shadowRoot.querySelector(".card");
    expect(card.classList.contains("loading")).to.be.true;
  });

  it("updates when properties change", async () => {
    const el = await fixture(html`<user-card></user-card>`);

    el.name = "Jane Doe";
    await el.updateComplete;

    expect(el.shadowRoot.querySelector("h2").textContent).to.equal("Jane Doe");
  });

  it("dispatches contact event when button clicked", async () => {
    const el = await fixture(html` <user-card name="John" email="john@test.com"></user-card> `);

    const button = el.shadowRoot.querySelector("button");

    setTimeout(() => button.click());
    const { detail } = await oneEvent(el, "contact");

    expect(detail.name).to.equal("John");
    expect(detail.email).to.equal("john@test.com");
  });

  it("is accessible", async () => {
    const el = await fixture(html` <user-card name="John" email="john@test.com"></user-card> `);

    await expect(el).to.be.accessible();
  });
});
```

---

## 5. Testing Stencil Components

Stencil có built-in testing utilities.

### 🔧 Cài đặt (đã có sẵn với Stencil)

```bash
npm install --save-dev @stencil/core
```

### 💻 Unit Tests

```typescript
// my-button.tsx
import { Component, Prop, Event, EventEmitter, h } from "@stencil/core";

@Component({
  tag: "my-button",
  styleUrl: "my-button.css",
  shadow: true,
})
export class MyButton {
  @Prop() disabled: boolean = false;
  @Prop() variant: "primary" | "secondary" = "primary";

  @Event() buttonClick: EventEmitter<void>;

  private handleClick = () => {
    if (!this.disabled) {
      this.buttonClick.emit();
    }
  };

  render() {
    return (
      <button class={`btn btn-${this.variant}`} disabled={this.disabled} onClick={this.handleClick}>
        <slot></slot>
      </button>
    );
  }
}
```

```typescript
// my-button.spec.ts
import { newSpecPage } from "@stencil/core/testing";
import { MyButton } from "./my-button";

describe("my-button", () => {
  it("renders", async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button>Click me</my-button>`,
    });

    expect(page.root).toEqualHtml(`
            <my-button>
                <mock:shadow-root>
                    <button class="btn btn-primary">
                        <slot></slot>
                    </button>
                </mock:shadow-root>
                Click me
            </my-button>
        `);
  });

  it("renders with secondary variant", async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button variant="secondary">Click</my-button>`,
    });

    const button = page.root.shadowRoot.querySelector("button");
    expect(button.classList.contains("btn-secondary")).toBe(true);
  });

  it("is disabled when disabled prop is true", async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button disabled>Click</my-button>`,
    });

    const button = page.root.shadowRoot.querySelector("button");
    expect(button.disabled).toBe(true);
  });

  it("emits buttonClick event when clicked", async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button>Click</my-button>`,
    });

    const spy = jest.fn();
    page.root.addEventListener("buttonClick", spy);

    const button = page.root.shadowRoot.querySelector("button");
    button.click();

    expect(spy).toHaveBeenCalled();
  });

  it("does not emit event when disabled", async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button disabled>Click</my-button>`,
    });

    const spy = jest.fn();
    page.root.addEventListener("buttonClick", spy);

    const button = page.root.shadowRoot.querySelector("button");
    button.click();

    expect(spy).not.toHaveBeenCalled();
  });
});
```

### 💻 E2E Tests với Stencil

```typescript
// my-button.e2e.ts
import { newE2EPage } from "@stencil/core/testing";

describe("my-button e2e", () => {
  it("renders", async () => {
    const page = await newE2EPage();
    await page.setContent("<my-button>Click me</my-button>");

    const element = await page.find("my-button");
    expect(element).toHaveClass("hydrated");
  });

  it("handles click events", async () => {
    const page = await newE2EPage();
    await page.setContent("<my-button>Click me</my-button>");

    const buttonClick = await page.spyOnEvent("buttonClick");
    const button = await page.find("my-button >>> button");

    await button.click();

    expect(buttonClick).toHaveReceivedEvent();
  });

  it("updates when props change", async () => {
    const page = await newE2EPage();
    await page.setContent('<my-button variant="primary">Click</my-button>');

    const element = await page.find("my-button");
    const button = await page.find("my-button >>> button");

    expect(button).toHaveClass("btn-primary");

    element.setProperty("variant", "secondary");
    await page.waitForChanges();

    expect(button).toHaveClass("btn-secondary");
  });
});
```

---

## 6. Best Practices

### ✅ Testing Checklist

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    WEB COMPONENTS TESTING CHECKLIST                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  □ Test rendering với default props                                    │
│  □ Test rendering với custom props/attributes                          │
│  □ Test property changes trigger re-render                             │
│  □ Test attribute reflection                                           │
│  □ Test event dispatching                                              │
│  □ Test event handling                                                 │
│  □ Test slots content                                                  │
│  □ Test CSS custom properties                                          │
│  □ Test accessibility (a11y)                                           │
│  □ Test keyboard navigation                                            │
│  □ Test disabled states                                                │
│  □ Test loading states                                                 │
│  □ Test error states                                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💡 Tips

```javascript
// 1. Luôn wait for updateComplete (Lit) hoặc waitForChanges (Stencil)
el.someProperty = "new value";
await el.updateComplete;
expect(el.shadowRoot.textContent).to.include("new value");

// 2. Query trong shadowRoot
const button = el.shadowRoot.querySelector("button");
// KHÔNG: el.querySelector('button')

// 3. Test accessibility
await expect(el).to.be.accessible();

// 4. Test slots
const el = await fixture(html`
  <my-card>
    <span slot="title">Title</span>
    <p>Content</p>
  </my-card>
`);
const slot = el.shadowRoot.querySelector('slot[name="title"]');
const assigned = slot.assignedNodes();
expect(assigned[0].textContent).to.equal("Title");

// 5. Mock external dependencies
const mockFetch = jest.fn(() => Promise.resolve({ json: () => Promise.resolve({ data: "test" }) }));
global.fetch = mockFetch;
```

---

## 🔗 Tài liệu tham khảo

- [Open WC Testing](https://open-wc.org/docs/testing/testing-package/)
- [Web Test Runner](https://modern-web.dev/docs/test-runner/overview/)
- [Stencil Testing](https://stenciljs.com/docs/testing-overview)
- [Lit Testing](https://lit.dev/docs/tools/testing/)
