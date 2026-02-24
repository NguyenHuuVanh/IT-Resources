# Next.js Rendering: SSG, SSR, ISR, CSR - Hướng Dẫn Toàn Diện

## Mục Lục

1. [Giới Thiệu](#giới-thiệu)
2. [Client-Side Rendering (CSR)](#client-side-rendering-csr)
3. [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
4. [Static Site Generation (SSG)](#static-site-generation-ssg)
5. [Incremental Static Regeneration (ISR)](#incremental-static-regeneration-isr)
6. [So Sánh Chi Tiết](#so-sánh-chi-tiết)
7. [Khi Nào Dùng Gì?](#khi-nào-dùng-gì)
8. [Next.js App Router (13+)](#nextjs-app-router-13)
9. [Best Practices](#best-practices)

---

## Giới Thiệu

### Rendering là gì?

**Rendering** là quá trình chuyển đổi code (React components, data) thành HTML để hiển thị trên browser. Đây là bước quan trọng quyết định:

- Tốc độ tải trang
- SEO (Search Engine Optimization)
- User experience
- Server load và chi phí

### Tại Sao Cần Nhiều Phương Thức Rendering?

Không có một phương pháp nào phù hợp cho tất cả các trường hợp:

```javascript
// Ví dụ: E-commerce website
{
  homepage: "Cần SEO + Performance → SSG/ISR",
  productPage: "Cần SEO + Fresh data → ISR",
  userDashboard: "Personalized + No SEO → CSR",
  checkout: "Secure + Real-time → SSR"
}
```

Next.js cho phép bạn chọn phương pháp tối ưu cho từng trang, thậm chí kết hợp nhiều phương pháp trong cùng một ứng dụng.

### Các Phương Thức Rendering

Next.js hỗ trợ 4 phương thức chính:

| Phương Thức                         | Viết Tắt | Render Ở Đâu? | Render Khi Nào?         |
| ----------------------------------- | -------- | ------------- | ----------------------- |
| **Client-Side Rendering**           | CSR      | Browser       | Sau khi JS load         |
| **Server-Side Rendering**           | SSR      | Server        | Mỗi request             |
| **Static Site Generation**          | SSG      | Server        | Build time              |
| **Incremental Static Regeneration** | ISR      | Server        | Build time + Background |

### Khái Niệm Cơ Bản

#### 1. Pre-rendering

Next.js **pre-render** mỗi page thành HTML trước, thay vì để JavaScript render tất cả trên client.

```
Traditional React (CSR):
Browser → Empty HTML → Download JS → Execute JS → Render UI

Next.js (Pre-rendering):
Browser → Full HTML → Download JS → Hydrate → Interactive
```

**Lợi ích**:

- SEO tốt hơn (crawlers thấy content ngay)
- Performance tốt hơn (user thấy content nhanh)
- Hoạt động khi JS disabled

#### 2. Hydration

**Hydration** là quá trình React "attach" event listeners và state vào HTML đã được pre-render.

```javascript
// Server render HTML:
<button>Click me</button>

// Client hydrate (thêm interactivity):
<button onClick={handleClick}>Click me</button>
```

**Flow**:

```
1. Server gửi HTML → User thấy content (không interactive)
2. JS download → React hydrate
3. Page interactive → User có thể click, type, etc.
```

#### 3. Build Time vs Runtime

**Build Time**: Khi chạy `npm run build`

- SSG generate HTML tại đây
- Code được compile, optimize
- Chỉ chạy một lần khi deploy

**Runtime**: Khi user request page

- SSR render HTML tại đây
- CSR fetch data và render tại đây
- Chạy mỗi khi có request

#### 4. Static vs Dynamic

**Static Content**: Không thay đổi giữa các users

```javascript
// Blog post, documentation, marketing pages
<article>
  <h1>How to Learn React</h1>
  <p>React is a JavaScript library...</p>
</article>
```

**Dynamic Content**: Khác nhau cho mỗi user hoặc request

```javascript
// User dashboard, personalized recommendations
<div>
  <h1>Welcome, {user.name}</h1>
  <p>Your orders: {user.orders.length}</p>
</div>
```

#### 5. Data Fetching Timing

```javascript
// Build Time (SSG)
export async function getStaticProps() {
  // Chạy lúc build
  const data = await fetchData();
  return { props: { data } };
}

// Request Time (SSR)
export async function getServerSideProps() {
  // Chạy mỗi request
  const data = await fetchData();
  return { props: { data } };
}

// Client Side (CSR)
useEffect(() => {
  // Chạy trên browser
  fetchData().then(setData);
}, []);
```

### Timeline Evolution

```
2016: React (CSR only)
  ↓
2018: Next.js 7 (SSR support)
  ↓
2019: Next.js 9 (SSG với getStaticProps)
  ↓
2020: Next.js 10 (ISR với revalidate)
  ↓
2022: Next.js 12 (Middleware, Edge Runtime)
  ↓
2023: Next.js 13 (App Router, React Server Components)
  ↓
2024: Next.js 14 (Partial Prerendering, Server Actions)
```

### Kiến Trúc Next.js

```
┌─────────────────────────────────────────┐
│           Next.js Application           │
├─────────────────────────────────────────┤
│  Pages/Routes                           │
│  ├─ page1.js (SSG)                      │
│  ├─ page2.js (SSR)                      │
│  ├─ page3.js (ISR)                      │
│  └─ page4.js (CSR)                      │
├─────────────────────────────────────────┤
│  Data Fetching Layer                    │
│  ├─ getStaticProps (Build time)         │
│  ├─ getServerSideProps (Request time)   │
│  ├─ getStaticPaths (Dynamic routes)     │
│  └─ Client-side fetch (Runtime)         │
├─────────────────────────────────────────┤
│  Rendering Engine                       │
│  ├─ React Server Components             │
│  ├─ React Client Components             │
│  └─ Streaming & Suspense                │
├─────────────────────────────────────────┤
│  Optimization Layer                     │
│  ├─ Image Optimization                  │
│  ├─ Font Optimization                   │
│  ├─ Code Splitting                      │
│  └─ Caching Strategy                    │
└─────────────────────────────────────────┘
```

### Core Concepts

#### Request Lifecycle

```javascript
// 1. User request page
GET / products / 123;

// 2. Next.js router xác định rendering method
if (page.hasGetServerSideProps) {
  // SSR: Render on server
  const html = await renderOnServer();
  return html;
} else if (page.hasGetStaticProps) {
  // SSG/ISR: Serve static file
  if (needsRevalidation) {
    regenerateInBackground();
  }
  return cachedHTML;
} else {
  // CSR: Serve empty HTML + JS
  return emptyHTML + jsBundle;
}

// 3. Browser nhận response
// 4. Hydration (nếu có pre-rendering)
// 5. Page interactive
```

#### Caching Layers

Next.js có nhiều caching layers:

```
┌──────────────────────────────────────┐
│  CDN Cache (Vercel Edge Network)    │ ← Fastest
├──────────────────────────────────────┤
│  Next.js Cache (Static files)       │
├──────────────────────────────────────┤
│  Data Cache (fetch responses)       │
├──────────────────────────────────────┤
│  React Cache (Component level)      │
├──────────────────────────────────────┤
│  Browser Cache                       │ ← Slowest
└──────────────────────────────────────┘
```

#### Performance Metrics

Hiểu các metrics để chọn rendering method phù hợp:

```javascript
// TTFB (Time To First Byte)
// Thời gian từ request đến byte đầu tiên
CSR: ~100ms (static file)
SSG: ~50ms (static file)
SSR: ~500ms (server processing)
ISR: ~50ms (cached) hoặc ~500ms (regenerating)

// FCP (First Contentful Paint)
// Thời gian đến khi user thấy content đầu tiên
CSR: ~2000ms (phải load JS)
SSG: ~300ms (HTML có sẵn)
SSR: ~800ms (HTML có sẵn)
ISR: ~300ms (HTML có sẵn)

// TTI (Time To Interactive)
// Thời gian đến khi page có thể tương tác
CSR: ~3500ms
SSG: ~1000ms
SSR: ~2000ms
ISR: ~1000ms

// LCP (Largest Contentful Paint)
// Thời gian render element lớn nhất
Target: < 2.5s (Good)
```

### Terminology

| Thuật Ngữ             | Giải Thích                  | Ví Dụ                 |
| --------------------- | --------------------------- | --------------------- |
| **Pre-rendering**     | Generate HTML trước         | SSG, SSR              |
| **Hydration**         | Thêm interactivity vào HTML | React attach events   |
| **Code Splitting**    | Chia JS thành chunks nhỏ    | Dynamic import        |
| **Lazy Loading**      | Load component khi cần      | React.lazy()          |
| **Prefetching**       | Load trước khi cần          | Link prefetch         |
| **Revalidation**      | Regenerate static page      | ISR background update |
| **Fallback**          | UI khi page chưa generate   | Loading state         |
| **Edge Runtime**      | Run code gần user           | Vercel Edge Functions |
| **Middleware**        | Intercept requests          | Auth, redirects       |
| **Server Components** | Component chỉ render server | No JS to client       |
| **Client Components** | Component cần interactivity | useState, onClick     |

### Khi Nào Dùng Next.js?

✅ **Nên dùng Next.js khi**:

- Cần SEO tốt
- Muốn performance tối ưu
- Cần flexibility (SSG + SSR + CSR)
- Large-scale applications
- E-commerce, blogs, SaaS platforms

❌ **Không cần Next.js khi**:

- Pure SPA không cần SEO (dùng Create React App)
- Static site đơn giản (dùng Gatsby, Hugo)
- Mobile app (dùng React Native)
- Real-time app thuần (dùng Socket.io + React)

---

## Client-Side Rendering (CSR)

### Khái Niệm

**Client-Side Rendering (CSR)** là phương pháp rendering truyền thống của React, nơi toàn bộ UI được render trên browser bằng JavaScript.

Server chỉ gửi về:

- HTML shell rỗng (minimal HTML)
- JavaScript bundle
- CSS files

Browser sau đó:

- Download và execute JavaScript
- Fetch data từ API
- Render UI động

### Đặc Điểm Chính

```javascript
// HTML ban đầu từ server (rất nhỏ)
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <div id="root"></div>  <!-- Empty! -->
    <script src="bundle.js"></script>
  </body>
</html>

// JavaScript sẽ render tất cả content
ReactDOM.render(<App />, document.getElementById('root'));
```

### Cách Hoạt Động Chi Tiết

```
Timeline:
─────────────────────────────────────────────────────────

t=0ms:    User clicks link
          │
t=50ms:   Browser sends request to server
          │
t=150ms:  Server responds với HTML rỗng + JS bundle URLs
          │ (User thấy blank page)
          │
t=200ms:  Browser bắt đầu download JS bundle (2MB)
          │
t=1500ms: JS download complete
          │
t=1600ms: JS parsing & execution
          │
t=1800ms: React app khởi tạo
          │
t=1900ms: useEffect chạy → Fetch API data
          │ (User thấy loading spinner)
          │
t=2400ms: API response về
          │
t=2500ms: React render UI với data
          │
t=2600ms: ✅ User thấy content đầy đủ
          │
t=2700ms: ✅ Page interactive
```

### Vòng Đời Request-Response

```javascript
// 1. Initial Request
GET /products
→ Server: Trả về HTML rỗng (instant)

// 2. Browser Parse HTML
<div id="root"></div>
<script src="/static/js/main.chunk.js"></script>

// 3. Download JavaScript
GET /static/js/main.chunk.js (2.5MB)
→ Takes 1-2 seconds on 3G

// 4. Execute JavaScript
ReactDOM.render(<App />)
→ React khởi tạo, components mount

// 5. Fetch Data (trong useEffect)
GET /api/products
→ API call, wait for response

// 6. Render với Data
setProducts(data)
→ Re-render với data mới

// 7. Interactive
User có thể click, scroll, interact
```

### Internal Mechanism

```javascript
// Browser's perspective

// Step 1: Parse HTML
document.getElementById("root"); // <div id="root"></div>

// Step 2: Load & Execute JS
const App = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  // Step 3: Component Mount
  useEffect(() => {
    // Step 4: Fetch Data
    fetch("/api/data")
      .then((res) => res.json())
      .then((data) => {
        // Step 5: Update State
        setData(data);
        setLoading(false);
      });
  }, []);

  // Step 6: Render UI
  if (loading) return <Spinner />;
  return <Content data={data} />;
};

// Step 7: Mount to DOM
ReactDOM.render(<App />, document.getElementById("root"));
```

### Network Waterfall

```
CSR Network Waterfall:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HTML (5KB)          ████ (100ms)
├─ CSS (50KB)           ████████ (200ms)
├─ JS Main (500KB)      ████████████████████ (800ms)
│  ├─ Parse & Compile       ████ (150ms)
│  └─ Execute               ████ (150ms)
│
└─ API Call (/api/products)     ████████ (300ms)
   └─ Render với data               ██ (50ms)

Total: ~1800ms đến khi user thấy content
```

### Code Example (Pages Router)

```javascript
// pages/products-csr.js
import { useState, useEffect } from "react";

export default function ProductsCSR() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Fetch data on client side
    fetch("/api/products")
      .then((res) => res.json())
      .then((data) => {
        setProducts(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Products (CSR)</h1>
      {products.map((product) => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### Code Example (App Router)

```javascript
// app/products-csr/page.js
"use client"; // Bắt buộc cho CSR trong App Router

import { useState, useEffect } from "react";

export default function ProductsCSR() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch("/api/products")
      .then((res) => res.json())
      .then((data) => setProducts(data));
  }, []);

  return (
    <div>
      <h1>Products (CSR)</h1>
      {products.map((product) => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### Ưu Điểm

✅ Tương tác nhanh sau khi load xong
✅ Giảm tải cho server
✅ Rich interactions, animations
✅ Phù hợp với SPA
✅ Không cần server mạnh

### Nhược Điểm

❌ SEO kém (content không có trong HTML ban đầu)
❌ First Contentful Paint (FCP) chậm
❌ Phụ thuộc vào JavaScript
❌ Loading state nhiều
❌ Không tốt cho low-end devices

### Khi Nào Dùng CSR?

- Dashboard, admin panel (không cần SEO)
- Trang cần authentication
- Real-time data (chat, notifications)
- Interactive apps (games, tools)

---

## Server-Side Rendering (SSR)

### Khái Niệm

**Server-Side Rendering (SSR)** là phương pháp render HTML hoàn chỉnh trên server cho mỗi request, sau đó gửi về browser.

Khác với CSR:

- HTML đã có đầy đủ content
- User thấy content ngay lập tức
- SEO-friendly (crawlers thấy full content)
- Hydration sau đó để thêm interactivity

### Đặc Điểm Chính

```javascript
// HTML từ server (đầy đủ content)
<!DOCTYPE html>
<html>
  <head>
    <title>Products - My Store</title>
  </head>
  <body>
    <div id="root">
      <!-- Full HTML content -->
      <h1>Products</h1>
      <div class="product">
        <h2>Product 1</h2>
        <p>$99.99</p>
      </div>
      <div class="product">
        <h2>Product 2</h2>
        <p>$149.99</p>
      </div>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>
```

### Cách Hoạt Động Chi Tiết

```
Timeline:
─────────────────────────────────────────────────────────

t=0ms:    User clicks link
          │
t=50ms:   Browser sends request to server
          │
t=100ms:  Server nhận request
          │
          ├─ Execute getServerSideProps()
          │  ├─ Fetch data từ database (200ms)
          │  ├─ Fetch data từ API (150ms)
          │  └─ Process data (50ms)
          │
t=500ms:  Server có đủ data
          │
          ├─ Render React components to HTML
          │  └─ ReactDOMServer.renderToString()
          │
t=650ms:  HTML generation complete
          │
t=700ms:  Server gửi HTML về browser
          │
t=800ms:  Browser nhận HTML
          │ ✅ User thấy content (không interactive)
          │
t=900ms:  Browser download JS bundle
          │
t=1500ms: JS download complete
          │
t=1600ms: React hydration bắt đầu
          │
t=1800ms: ✅ Page fully interactive
```

### Server-Side Process

```javascript
// Server-side execution flow

// 1. Request arrives
app.get("/products/:id", async (req, res) => {
  // 2. Execute getServerSideProps
  const props = await getServerSideProps({
    req,
    res,
    params: { id: req.params.id },
    query: req.query,
  });

  // 3. Fetch data (parallel)
  const [product, reviews, related] = await Promise.all([
    fetchProduct(props.params.id),
    fetchReviews(props.params.id),
    fetchRelatedProducts(props.params.id),
  ]);

  // 4. Render React to HTML string
  const html = ReactDOMServer.renderToString(<ProductPage product={product} reviews={reviews} related={related} />);

  // 5. Inject into HTML template
  const fullHTML = `
    <!DOCTYPE html>
    <html>
      <head>
        <title>${product.name}</title>
        <meta name="description" content="${product.description}">
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          window.__INITIAL_DATA__ = ${JSON.stringify({
            product,
            reviews,
            related,
          })};
        </script>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `;

  // 6. Send response
  res.send(fullHTML);
});
```

### Hydration Process

```javascript
// Client-side hydration

// 1. Browser nhận HTML với content
// User đã thấy content nhưng chưa interactive

// 2. JS bundle download & execute
import { hydrateRoot } from "react-dom/client";

// 3. React "hydrate" thay vì "render"
const root = document.getElementById("root");
const initialData = window.__INITIAL_DATA__;

hydrateRoot(root, <ProductPage {...initialData} />);

// 4. React attach event listeners
// <button onClick={handleClick}> ← Giờ mới hoạt động

// 5. Page fully interactive
```

### Request Flow Diagram

```
Client                    Server                    Database/API
  │                         │                            │
  ├─── GET /products ──────>│                            │
  │                         │                            │
  │                         ├─── Query products ───────>│
  │                         │                            │
  │                         │<──── Return data ──────────┤
  │                         │                            │
  │                         ├─ Render React to HTML     │
  │                         │  (ReactDOMServer)          │
  │                         │                            │
  │<──── HTML + Data ───────┤                            │
  │                         │                            │
  ├─ Display content        │                            │
  │  (not interactive yet)  │                            │
  │                         │                            │
  ├─── GET /bundle.js ─────>│                            │
  │<──── JS bundle ─────────┤                            │
  │                         │                            │
  ├─ Hydration              │                            │
  │  (attach events)        │                            │
  │                         │                            │
  └─ ✅ Fully interactive   │                            │
```

### Memory & CPU Usage

```javascript
// Server resources per request

Request 1: /products/123
├─ Memory: ~50MB (React render)
├─ CPU: ~100ms (data fetch + render)
└─ Response time: ~500ms

Request 2: /products/456 (concurrent)
├─ Memory: ~50MB (separate instance)
├─ CPU: ~100ms
└─ Response time: ~500ms

// 100 concurrent requests
├─ Memory: ~5GB
├─ CPU: High load
└─ Need scaling (load balancer, multiple servers)
```

### Caching Strategies

```javascript
// 1. No cache (always fresh)
export async function getServerSideProps({ res }) {
  res.setHeader("Cache-Control", "no-store");
  return { props: { data } };
}

// 2. Cache với revalidation
export async function getServerSideProps({ res }) {
  res.setHeader("Cache-Control", "public, s-maxage=10, stale-while-revalidate=59");
  // Cache 10s, serve stale trong 59s khi revalidate
  return { props: { data } };
}

// 3. Private cache (user-specific)
export async function getServerSideProps({ res }) {
  res.setHeader("Cache-Control", "private, max-age=60");
  return { props: { data } };
}

// 4. CDN cache
export async function getServerSideProps({ res }) {
  res.setHeader("Cache-Control", "public, s-maxage=3600, stale-while-revalidate=86400");
  // CDN cache 1 hour, serve stale 24h
  return { props: { data } };
}
```

### Network Waterfall

```
SSR Network Waterfall:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Server Processing:
├─ Receive request          ██ (50ms)
├─ Database query           ████████ (200ms)
├─ API calls (parallel)     ██████ (150ms)
├─ Render to HTML           ████ (100ms)
└─ Send response            ██ (50ms)
                            ─────────────────
                            Total: 550ms (TTFB)

Client Processing:
├─ Receive HTML             ██ (50ms)
├─ Parse & Display          ████ (100ms) ← User sees content
├─ Download JS              ████████████ (400ms)
├─ Hydration                ████ (100ms)
└─ Interactive              ✅ (1200ms total)

Total: ~1200ms đến fully interactive
```

### Code Example (Pages Router)

```javascript
// pages/products-ssr.js

// getServerSideProps chạy trên server mỗi request
export async function getServerSideProps(context) {
  // Access request context
  const { req, res, query, params } = context;

  // Fetch data
  const response = await fetch("https://api.example.com/products");
  const products = await response.json();

  // Return props
  return {
    props: {
      products,
      timestamp: new Date().toISOString(),
    },
  };
}

export default function ProductsSSR({ products, timestamp }) {
  return (
    <div>
      <h1>Products (SSR)</h1>
      <p>Rendered at: {timestamp}</p>
      {products.map((product) => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}
```

### Code Example với Error Handling

```javascript
// pages/user/[id].js

export async function getServerSideProps(context) {
  const { params } = context;

  try {
    const res = await fetch(`https://api.example.com/users/${params.id}`);

    if (!res.ok) {
      return {
        notFound: true, // Hiển thị 404 page
      };
    }

    const user = await res.json();

    return {
      props: { user },
    };
  } catch (error) {
    return {
      redirect: {
        destination: "/error",
        permanent: false,
      },
    };
  }
}

export default function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Code Example (App Router - Next.js 13+)

```javascript
// app/products-ssr/page.js

// Không cache, fetch mỗi request
async function getProducts() {
  const res = await fetch("https://api.example.com/products", {
    cache: "no-store", // Tương đương SSR
  });
  return res.json();
}

export default async function ProductsSSR() {
  const products = await getProducts();

  return (
    <div>
      <h1>Products (SSR)</h1>
      {products.map((product) => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### Advanced SSR Features

```javascript
// pages/dashboard.js

export async function getServerSideProps(context) {
  const { req, res } = context;

  // 1. Set cache headers
  res.setHeader("Cache-Control", "public, s-maxage=10, stale-while-revalidate=59");

  // 2. Check authentication
  const session = await getSession(context);
  if (!session) {
    return {
      redirect: {
        destination: "/login",
        permanent: false,
      },
    };
  }

  // 3. Parallel data fetching
  const [userData, statsData] = await Promise.all([
    fetch(`/api/user/${session.userId}`).then((r) => r.json()),
    fetch("/api/stats").then((r) => r.json()),
  ]);

  return {
    props: {
      user: userData,
      stats: statsData,
    },
  };
}

export default function Dashboard({ user, stats }) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <Stats data={stats} />
    </div>
  );
}
```

### Ưu Điểm

✅ SEO tốt (HTML đầy đủ ngay từ đầu)
✅ Fast First Contentful Paint
✅ Data luôn fresh (mỗi request)
✅ Tốt cho dynamic content
✅ Hoạt động khi JS disabled

### Nhược Điểm

❌ Chậm hơn SSG (phải render mỗi request)
❌ Tốn tài nguyên server
❌ TTFB (Time To First Byte) cao
❌ Cần server Node.js
❌ Không thể deploy lên CDN

### Khi Nào Dùng SSR?

- Data thay đổi liên tục
- Personalized content (user-specific)
- Real-time data cần SEO
- Authentication-based pages cần SEO

---

## Static Site Generation (SSG)

### Khái Niệm

**Static Site Generation (SSG)** là phương pháp pre-render tất cả pages thành HTML tĩnh tại build time, trước khi deploy.

Đặc điểm:

- HTML được generate một lần duy nhất khi build
- Mọi user nhận cùng một HTML file
- Serve từ CDN (cực nhanh)
- Không cần server Node.js để serve

### Đặc Điểm Chính

```javascript
// Build Process
$ npm run build

Building pages:
├─ / (SSG)                    ✓ Generated in 234ms
├─ /about (SSG)               ✓ Generated in 123ms
├─ /blog/post-1 (SSG)         ✓ Generated in 456ms
├─ /blog/post-2 (SSG)         ✓ Generated in 389ms
└─ /products/123 (SSG)        ✓ Generated in 567ms

Output:
.next/
├─ static/
│  └─ pages/
│     ├─ index.html           (homepage)
│     ├─ about.html           (about page)
│     └─ blog/
│        ├─ post-1.html
│        └─ post-2.html
```

### Cách Hoạt Động Chi Tiết

```
Build Time (npm run build):
─────────────────────────────────────────────────────────

t=0s:     Start build process
          │
t=1s:     Next.js reads all pages
          │
          ├─ Find pages with getStaticProps
          │  ├─ pages/index.js
          │  ├─ pages/blog/[slug].js
          │  └─ pages/products/[id].js
          │
t=2s:     Execute getStaticPaths() for dynamic routes
          │
          ├─ /blog/[slug].js
          │  └─ Returns: ['post-1', 'post-2', 'post-3']
          │
          ├─ /products/[id].js
          │  └─ Returns: ['1', '2', '3', ..., '100']
          │
t=5s:     For each path, execute getStaticProps()
          │
          ├─ Fetch data from CMS/API/Database
          │  ├─ GET /api/blog/post-1
          │  ├─ GET /api/blog/post-2
          │  └─ GET /api/products/1
          │
t=30s:    Render each page to HTML
          │
          ├─ ReactDOMServer.renderToString()
          │  ├─ <BlogPost data={post1} />
          │  ├─ <BlogPost data={post2} />
          │  └─ <Product data={product1} />
          │
t=45s:    Write HTML files to disk
          │
          ├─ .next/server/pages/blog/post-1.html
          ├─ .next/server/pages/blog/post-2.html
          └─ .next/server/pages/products/1.html
          │
t=50s:    ✅ Build complete
          │
          └─ Ready to deploy

Runtime (User Request):
─────────────────────────────────────────────────────────

t=0ms:    User requests /blog/post-1
          │
t=10ms:   CDN serves pre-built HTML file
          │ (No server processing!)
          │
t=50ms:   ✅ User sees content
          │
t=200ms:  JS downloads
          │
t=400ms:  ✅ Page interactive
```

### Build Process Internals

```javascript
// Next.js build process

// 1. Scan pages directory
const pages = ["pages/index.js", "pages/blog/[slug].js", "pages/products/[id].js"];

// 2. Identify static pages
const staticPages = pages.filter((page) => hasGetStaticProps(page) || hasGetStaticPaths(page));

// 3. For dynamic routes, get all paths
async function buildDynamicRoutes() {
  // pages/blog/[slug].js
  const { paths } = await getStaticPaths();
  // Returns: [
  //   { params: { slug: 'post-1' } },
  //   { params: { slug: 'post-2' } }
  // ]

  // 4. For each path, fetch data and render
  for (const path of paths) {
    const { props } = await getStaticProps({ params: path.params });

    // 5. Render to HTML
    const html = ReactDOMServer.renderToString(<BlogPost {...props} />);

    // 6. Write to file
    fs.writeFileSync(`.next/server/pages/blog/${path.params.slug}.html`, wrapInHTMLTemplate(html, props));
  }
}

// 7. Generate static files
await buildDynamicRoutes();
```

### File System Output

```
After build:
─────────────────────────────────────────────────────────

.next/
├─ server/
│  └─ pages/
│     ├─ index.html                    (5KB)
│     ├─ about.html                    (3KB)
│     ├─ blog/
│     │  ├─ post-1.html               (12KB)
│     │  ├─ post-2.html               (15KB)
│     │  └─ post-3.html               (10KB)
│     └─ products/
│        ├─ 1.html                    (8KB)
│        ├─ 2.html                    (8KB)
│        └─ ...                       (800 files)
│
├─ static/
│  ├─ chunks/
│  │  ├─ main.js                      (200KB)
│  │  └─ pages/
│  │     ├─ index.js                  (50KB)
│  │     └─ blog/[slug].js            (30KB)
│  └─ css/
│     └─ styles.css                   (20KB)
│
└─ cache/
   └─ (build cache for faster rebuilds)

Total size: ~50MB for 1000 pages
```

### Deployment & Serving

```javascript
// Traditional Server (SSR)
const server = express();
server.get("/blog/:slug", async (req, res) => {
  // Render on every request
  const html = await renderPage(req.params.slug);
  res.send(html);
});

// Static Server (SSG)
// Just serve files from disk
const server = express();
server.use(express.static(".next/server/pages"));

// Or deploy to CDN
// Upload .next/server/pages/* to S3/Cloudflare/Vercel
// CDN serves files directly (no server needed!)
```

### CDN Distribution

```
User Request Flow với CDN:
─────────────────────────────────────────────────────────

User (Vietnam)
  │
  ├─ Request: /blog/post-1
  │
  ↓
CDN Edge (Singapore) ← 10ms latency
  │
  ├─ Check cache
  │  └─ HIT! File exists
  │
  ├─ Serve file from edge
  │  └─ .next/server/pages/blog/post-1.html
  │
  ↓
User receives HTML (50ms total)
✅ Extremely fast!


Without CDN (SSR):
─────────────────────────────────────────────────────────

User (Vietnam)
  │
  ├─ Request: /blog/post-1
  │
  ↓
Origin Server (US) ← 200ms latency
  │
  ├─ Execute getServerSideProps (200ms)
  ├─ Render HTML (100ms)
  │
  ↓
User receives HTML (500ms total)
❌ Much slower
```

### Scalability

```javascript
// SSG Scalability

// Scenario: 1 million requests/hour

// SSG (Static files on CDN)
├─ Server load: 0% (CDN handles everything)
├─ Response time: ~50ms (CDN edge)
├─ Cost: $10/month (CDN bandwidth)
└─ Can handle: Unlimited requests

// SSR (Server rendering)
├─ Server load: 100% (need 50+ servers)
├─ Response time: ~500ms (server processing)
├─ Cost: $5000/month (server instances)
└─ Can handle: Limited by server capacity
```

### Limitations & Solutions

```javascript
// Problem 1: Stale Data
// Build: 10:00 AM → Data from 10:00 AM
// User visit: 2:00 PM → Still sees 10:00 AM data

// Solution: ISR (Incremental Static Regeneration)
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

// Problem 2: Long Build Times
// 10,000 products × 500ms = 5000 seconds = 83 minutes!

// Solution: Fallback
export async function getStaticPaths() {
  // Only pre-render popular products
  const popularProducts = await getPopular(100);

  return {
    paths: popularProducts.map((p) => ({ params: { id: p.id } })),
    fallback: "blocking", // Generate others on-demand
  };
}

// Problem 3: Dynamic Content
// User-specific data, real-time updates

// Solution: Hybrid approach
export default function Page({ staticData }) {
  // Static data from SSG
  const [dynamicData, setDynamicData] = useState(null);

  // Dynamic data from CSR
  useEffect(() => {
    fetch("/api/user-data").then(setDynamicData);
  }, []);

  return (
    <>
      <StaticContent data={staticData} />
      <DynamicContent data={dynamicData} />
    </>
  );
}
```

### Network Waterfall

```
SSG Network Waterfall:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Request to CDN:
├─ DNS lookup               ██ (20ms)
├─ CDN connection           ██ (10ms)
├─ HTML file (5KB)          ████ (20ms) ← User sees content
├─ CSS file (20KB)          ████ (30ms)
├─ JS bundle (200KB)        ████████ (200ms)
└─ Hydration                ████ (100ms)
                            ─────────────────
                            Total: ~380ms fully interactive

Compare với SSR: ~1200ms
Compare với CSR: ~2600ms
```

### Code Example (Pages Router)

```javascript
// pages/blog/[slug].js

// 1. Generate paths tại build time
export async function getStaticPaths() {
  const posts = await fetch("https://api.example.com/posts").then((r) => r.json());

  const paths = posts.map((post) => ({
    params: { slug: post.slug },
  }));

  return {
    paths,
    fallback: false, // false | true | 'blocking'
  };
}

// 2. Fetch data cho mỗi path
export async function getStaticProps({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then((r) => r.json());

  return {
    props: {
      post,
    },
  };
}

export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

### Fallback Options

```javascript
// pages/products/[id].js

export async function getStaticPaths() {
  // Chỉ pre-render 100 products phổ biến nhất
  const popularProducts = await getPopularProducts(100);

  const paths = popularProducts.map((product) => ({
    params: { id: product.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // false | true | 'blocking'
  };
}

// fallback: false
// - Chỉ paths được return mới valid
// - Paths khác → 404

// fallback: true
// - Paths không có sẽ render fallback UI
// - Fetch data on-demand
// - Cache sau khi generate

// fallback: 'blocking'
// - Giống true nhưng không có fallback UI
// - Wait cho page generate xong mới show
```

### Fallback True Example

```javascript
// pages/products/[id].js

export async function getStaticPaths() {
  return {
    paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
    fallback: true,
  };
}

export async function getStaticProps({ params }) {
  const product = await fetch(`/api/products/${params.id}`).then((r) => r.json());

  return {
    props: { product },
    revalidate: 60, // ISR
  };
}

export default function Product({ product }) {
  const router = useRouter();

  // Fallback UI khi page chưa generate
  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

### Code Example (App Router)

```javascript
// app/blog/[slug]/page.js

// Generate static params
export async function generateStaticParams() {
  const posts = await fetch("https://api.example.com/posts").then((r) => r.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// Fetch data (cached by default)
async function getPost(slug) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    cache: "force-cache", // Default, tương đương SSG
  });
  return res.json();
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

### Ưu Điểm

✅ Cực kỳ nhanh (serve file tĩnh)
✅ SEO tốt nhất
✅ Deploy lên CDN dễ dàng
✅ Giảm tải server
✅ Chi phí hosting thấp
✅ Highly scalable

### Nhược Điểm

❌ Data có thể outdated
❌ Build time lâu với nhiều pages
❌ Không phù hợp dynamic data
❌ Phải rebuild để update

### Khi Nào Dùng SSG?

- Blog, documentation
- Marketing pages, landing pages
- E-commerce product pages (với ISR)
- Portfolio, company website
- Content không thay đổi thường xuyên

---

## Incremental Static Regeneration (ISR)

### Khái Niệm

**Incremental Static Regeneration (ISR)** là sự kết hợp hoàn hảo giữa SSG và SSR, cho phép:

- Pre-render pages tại build time (như SSG)
- Tự động regenerate pages trong background sau một khoảng thời gian
- Generate pages on-demand cho paths chưa được build
- Cập nhật content mà không cần rebuild toàn bộ site

ISR giải quyết vấn đề lớn nhất của SSG: stale data.

### Đặc Điểm Chính

```javascript
// ISR Configuration
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60, // ⭐ Key difference: revalidate time
  };
}

// revalidate: 60 nghĩa là:
// - Page được cache 60 giây
// - Sau 60 giây, request tiếp theo trigger regeneration
// - User vẫn thấy old page (fast)
// - Background regeneration xảy ra
// - Request sau đó thấy new page
```

### Cách Hoạt Động Chi Tiết

```
Build Time (t=0):
─────────────────────────────────────────────────────────

$ npm run build

├─ Generate /products/123
│  ├─ Fetch data from API
│  ├─ Render to HTML
│  └─ Cache with timestamp: 10:00:00
│
└─ Deploy to production


Runtime - Request Timeline:
─────────────────────────────────────────────────────────

10:00:30 - User A visits /products/123
├─ Check cache: EXISTS (30s old)
├─ Check revalidate: 60s not passed
├─ Serve cached HTML (instant)
└─ ✅ Response time: 50ms

10:01:30 - User B visits /products/123
├─ Check cache: EXISTS (90s old)
├─ Check revalidate: 60s PASSED! ⚠️
├─ Serve cached HTML (stale but fast)
├─ Trigger background regeneration
│  ├─ Fetch fresh data from API
│  ├─ Render new HTML
│  └─ Update cache with timestamp: 10:01:30
└─ ✅ Response time: 50ms (user không wait)

10:01:35 - Background regeneration complete
└─ New HTML cached

10:02:00 - User C visits /products/123
├─ Check cache: EXISTS (30s old, FRESH)
├─ Serve NEW cached HTML
└─ ✅ Response time: 50ms (with fresh data)
```

### Stale-While-Revalidate Pattern

```javascript
// ISR implements "stale-while-revalidate" strategy

function handleRequest(url, revalidateTime) {
  const cached = getFromCache(url);
  const age = Date.now() - cached.timestamp;

  if (!cached) {
    // No cache: Generate on-demand (blocking)
    return generateAndCache(url);
  }

  if (age < revalidateTime) {
    // Fresh: Serve from cache
    return cached.html;
  }

  // Stale: Serve old + regenerate in background
  regenerateInBackground(url);
  return cached.html; // User gets instant response
}

// Visual representation:
// ─────────────────────────────────────────────────────
// 0s        60s       120s      180s
// │─ Fresh ─│─ Stale ─│─ Fresh ─│─ Stale ─│
//           ↑                   ↑
//           Regenerate          Regenerate
//           (background)        (background)
```

### Regeneration Process

```javascript
// Internal ISR mechanism

class ISRCache {
  constructor() {
    this.cache = new Map();
    this.regenerating = new Set();
  }

  async get(path, revalidate) {
    const cached = this.cache.get(path);
    const now = Date.now();

    // Case 1: No cache (first request or after purge)
    if (!cached) {
      console.log("MISS: Generating page...");
      return await this.generate(path);
    }

    const age = (now - cached.timestamp) / 1000;

    // Case 2: Fresh cache
    if (age < revalidate) {
      console.log("HIT: Serving fresh cache");
      return cached.html;
    }

    // Case 3: Stale cache
    console.log("STALE: Serving old + regenerating");

    // Serve stale immediately
    const response = cached.html;

    // Regenerate in background (only once)
    if (!this.regenerating.has(path)) {
      this.regenerating.add(path);

      this.generate(path)
        .then((newHtml) => {
          this.cache.set(path, {
            html: newHtml,
            timestamp: Date.now(),
          });
          this.regenerating.delete(path);
          console.log("REGENERATED: New cache ready");
        })
        .catch((err) => {
          console.error("REGENERATION FAILED:", err);
          this.regenerating.delete(path);
          // Keep serving stale on error
        });
    }

    return response;
  }

  async generate(path) {
    // Fetch data
    const data = await fetchData(path);

    // Render to HTML
    const html = await renderToHTML(path, data);

    // Cache
    this.cache.set(path, {
      html,
      timestamp: Date.now(),
    });

    return html;
  }
}
```

### On-Demand Revalidation

```javascript
// Trigger revalidation manually (không cần đợi revalidate time)

// pages/api/revalidate.js
export default async function handler(req, res) {
  // Verify request (security)
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: "Invalid token" });
  }

  try {
    // Revalidate specific path
    await res.revalidate("/products/123");

    console.log("Revalidation triggered for /products/123");

    return res.json({
      revalidated: true,
      timestamp: new Date().toISOString(),
    });
  } catch (err) {
    return res.status(500).json({
      message: "Error revalidating",
      error: err.message,
    });
  }
}

// Webhook from CMS
// POST /api/revalidate?secret=xxx
// Body: { path: '/products/123' }

// Use cases:
// 1. Content updated in CMS → Webhook → Revalidate
// 2. Product price changed → API call → Revalidate
// 3. Admin updates content → Button click → Revalidate
```

### Fallback Modes

```javascript
// getStaticPaths với ISR

export async function getStaticPaths() {
  // Pre-render only popular pages at build time
  const popularProducts = await getPopularProducts(100);

  return {
    paths: popularProducts.map((p) => ({
      params: { id: p.id.toString() },
    })),
    fallback: "blocking", // or true or false
  };
}

// fallback: false
// ─────────────────────────────────────────────────────
// - Chỉ paths được return mới valid
// - Paths khác → 404
// - Use case: Biết chính xác tất cả paths

// fallback: true
// ─────────────────────────────────────────────────────
// - Paths không có → Show fallback UI
// - Generate page in background
// - Cache sau khi generate
// - Use case: Nhiều pages, muốn show loading

export default function Product({ product }) {
  const router = useRouter();

  if (router.isFallback) {
    return <LoadingSkeleton />; // Fallback UI
  }

  return <ProductDetails product={product} />;
}

// fallback: 'blocking'
// ─────────────────────────────────────────────────────
// - Paths không có → Generate on-demand
// - User wait cho page generate (như SSR)
// - Không có fallback UI
// - Cache sau khi generate
// - Use case: Không muốn show loading, prefer wait
```

### Fallback Flow Comparison

```
fallback: true
─────────────────────────────────────────────────────────

User requests /products/999 (not pre-rendered)
  │
  ├─ Next.js checks cache: NOT FOUND
  │
  ├─ Immediately return fallback page
  │  └─ router.isFallback = true
  │  └─ User sees: <LoadingSkeleton />
  │
  ├─ Background: Generate page
  │  ├─ Execute getStaticProps
  │  ├─ Fetch data
  │  └─ Render HTML
  │
  ├─ Generation complete (2s later)
  │
  ├─ Replace fallback with real content
  │  └─ User sees: <ProductDetails />
  │
  └─ Cache for future requests


fallback: 'blocking'
─────────────────────────────────────────────────────────

User requests /products/999 (not pre-rendered)
  │
  ├─ Next.js checks cache: NOT FOUND
  │
  ├─ User waits... (no response yet)
  │
  ├─ Generate page (SSR-like)
  │  ├─ Execute getStaticProps
  │  ├─ Fetch data
  │  └─ Render HTML
  │
  ├─ Generation complete (2s later)
  │
  ├─ Send HTML to user
  │  └─ User sees: <ProductDetails />
  │
  └─ Cache for future requests
```

### Cache Invalidation Strategies

```javascript
// Strategy 1: Time-based (revalidate)
export async function getStaticProps() {
  return {
    props: { data },
    revalidate: 60, // Every 60 seconds
  };
}

// Strategy 2: On-demand (webhook)
// CMS → Webhook → /api/revalidate
await res.revalidate("/blog/post-1");

// Strategy 3: Tag-based (App Router)
// app/products/[id]/page.js
export async function generateStaticParams() {
  return [{ id: "1" }, { id: "2" }];
}

async function getProduct(id) {
  const res = await fetch(`/api/products/${id}`, {
    next: {
      revalidate: 60,
      tags: ["products", `product-${id}`],
    },
  });
  return res.json();
}

// Revalidate all products
revalidateTag("products");

// Revalidate specific product
revalidateTag("product-123");

// Strategy 4: Path-based
revalidatePath("/products/123");
revalidatePath("/products"); // All products
```

### Performance Characteristics

```javascript
// Performance comparison

// First request (cache miss)
ISR (fallback: 'blocking'):
├─ TTFB: ~500ms (generate on-demand)
├─ FCP: ~700ms
└─ Like SSR

ISR (fallback: true):
├─ TTFB: ~50ms (fallback page)
├─ FCP: ~200ms (skeleton)
├─ Real content: ~2000ms
└─ Progressive loading

// Subsequent requests (cache hit)
ISR:
├─ TTFB: ~50ms (cached)
├─ FCP: ~200ms
└─ Like SSG (very fast!)

// After revalidate time
ISR:
├─ TTFB: ~50ms (serve stale)
├─ FCP: ~200ms (old content)
├─ Background: Regenerate
└─ Next request: Fresh content
```

### Real-World Example: E-commerce

```javascript
// pages/products/[id].js

export async function getStaticPaths() {
  // Pre-render top 1000 products at build time
  const topProducts = await db.products.orderBy("views", "desc").limit(1000).select("id");

  return {
    paths: topProducts.map((p) => ({
      params: { id: p.id.toString() },
    })),
    fallback: "blocking", // Generate others on-demand
  };
}

export async function getStaticProps({ params }) {
  // Fetch product data
  const [product, reviews, related] = await Promise.all([
    db.products.findById(params.id),
    db.reviews.findByProductId(params.id),
    db.products.findRelated(params.id, 4),
  ]);

  if (!product) {
    return { notFound: true };
  }

  return {
    props: {
      product,
      reviews,
      related,
      generatedAt: new Date().toISOString(),
    },
    revalidate: 300, // Revalidate every 5 minutes
  };
}

export default function ProductPage({ product, reviews, related, generatedAt }) {
  return (
    <div>
      <ProductDetails product={product} />
      <Reviews reviews={reviews} />
      <RelatedProducts products={related} />

      {/* Show when page was generated */}
      <small>Last updated: {generatedAt}</small>
    </div>
  );
}

// Result:
// - Top 1000 products: Pre-rendered at build (instant)
// - Other products: Generated on first visit (2s wait)
// - All products: Cached and served fast
// - Auto-update: Every 5 minutes in background
// - Manual update: Webhook from admin panel
```

### Monitoring & Debugging

```javascript
// Check ISR status

// 1. Response headers
HTTP/1.1 200 OK
X-Nextjs-Cache: HIT          // Served from cache
X-Nextjs-Cache: MISS         // Generated on-demand
X-Nextjs-Cache: STALE        // Served stale, regenerating

Age: 45                      // Cache age in seconds
Cache-Control: s-maxage=60, stale-while-revalidate

// 2. Logging
export async function getStaticProps({ params }) {
  console.log(`[ISR] Generating page: /products/${params.id}`);
  console.log(`[ISR] Timestamp: ${new Date().toISOString()}`);

  const data = await fetchData(params.id);

  return {
    props: { data },
    revalidate: 60
  };
}

// 3. Analytics
// Track regeneration frequency
analytics.track('isr_regeneration', {
  path: `/products/${id}`,
  timestamp: Date.now(),
  reason: 'time-based' // or 'on-demand'
});
```

### Network Waterfall

```
ISR Network Waterfall:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Scenario 1: Cache Hit (Fresh)
├─ Request to CDN           ██ (30ms)
├─ Serve cached HTML        ████ (50ms) ← User sees content
├─ Download JS              ████████ (200ms)
└─ Hydration                ████ (100ms)
                            Total: ~380ms ✅ Very fast!

Scenario 2: Cache Hit (Stale)
├─ Request to server        ██ (30ms)
├─ Serve stale HTML         ████ (50ms) ← User sees content
├─ Background regeneration  ████████████ (500ms, async)
├─ Download JS              ████████ (200ms)
└─ Hydration                ████ (100ms)
                            Total: ~380ms ✅ Still fast!
                            (New content ready for next user)

Scenario 3: Cache Miss (fallback: 'blocking')
├─ Request to server        ██ (30ms)
├─ Generate page            ████████████ (500ms)
├─ Serve HTML               ████ (50ms) ← User sees content
├─ Download JS              ████████ (200ms)
└─ Hydration                ████ (100ms)
                            Total: ~880ms (First time only)
```

### Code Example (Pages Router)

```javascript
// pages/products/[id].js

export async function getStaticPaths() {
  const products = await fetch("https://api.example.com/products").then((r) => r.json());

  const paths = products.map((product) => ({
    params: { id: product.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking",
  };
}

export async function getStaticProps({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`).then((r) => r.json());

  return {
    props: {
      product,
      generatedAt: new Date().toISOString(),
    },
    revalidate: 60, // Regenerate sau 60 giây
  };
}

export default function Product({ product, generatedAt }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Price: ${product.price}</p>
      <p>Stock: {product.stock}</p>
      <small>Generated at: {generatedAt}</small>
    </div>
  );
}
```

### ISR Flow Chi Tiết

```javascript
// Giả sử revalidate: 60

// t=0s: Build time
// → Page generated, cached

// t=30s: User A request
// → Serve cached page (fast)

// t=70s: User B request (sau revalidate time)
// → Serve cached page (stale)
// → Trigger regeneration in background

// t=75s: Regeneration complete
// → New page cached

// t=80s: User C request
// → Serve new page
```

### On-Demand Revalidation

```javascript
// pages/api/revalidate.js

export default async function handler(req, res) {
  // Check secret để bảo mật
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: "Invalid token" });
  }

  try {
    // Revalidate specific path
    await res.revalidate("/products/123");
    await res.revalidate("/blog/my-post");

    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send("Error revalidating");
  }
}

// Webhook từ CMS
// POST /api/revalidate?secret=xxx&path=/blog/new-post
```

### Code Example (App Router)

```javascript
// app/products/[id]/page.js

export const revalidate = 60; // ISR với 60 giây

async function getProduct(id) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 60 }, // Hoặc set ở đây
  });
  return res.json();
}

export default async function Product({ params }) {
  const product = await getProduct(params.id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}
```

### On-Demand Revalidation (App Router)

```javascript
// app/api/revalidate/route.js

import { revalidatePath, revalidateTag } from "next/cache";

export async function POST(request) {
  const { path, tag } = await request.json();

  if (path) {
    revalidatePath(path);
  }

  if (tag) {
    revalidateTag(tag);
  }

  return Response.json({ revalidated: true });
}

// Sử dụng tags
// app/products/page.js
async function getProducts() {
  const res = await fetch("https://api.example.com/products", {
    next: { tags: ["products"] },
  });
  return res.json();
}

// Revalidate tất cả pages có tag 'products'
// POST /api/revalidate { "tag": "products" }
```

### Ưu Điểm

✅ Kết hợp tốt nhất của SSG và SSR
✅ Fast như SSG
✅ Data tương đối fresh
✅ Không cần rebuild toàn bộ site
✅ Scalable
✅ On-demand revalidation

### Nhược Điểm

❌ Data vẫn có thể stale trong revalidate window
❌ Phức tạp hơn SSG thuần
❌ Cần hiểu rõ caching behavior

### Khi Nào Dùng ISR?

- E-commerce (product pages)
- Blog với comments/likes
- News websites
- Data thay đổi không quá thường xuyên
- Cần balance giữa performance và freshness

---

## So Sánh Chi Tiết

### Bảng So Sánh Tổng Quan

| Tiêu Chí           | CSR       | SSR                  | SSG                | ISR                |
| ------------------ | --------- | -------------------- | ------------------ | ------------------ |
| **Render Time**    | Client    | Server (mỗi request) | Build time         | Build + Background |
| **Speed**          | Chậm FCP  | Trung bình           | Cực nhanh          | Cực nhanh          |
| **SEO**            | Kém       | Tốt                  | Tốt nhất           | Tốt nhất           |
| **Data Freshness** | Real-time | Real-time            | Stale              | Tương đối fresh    |
| **Server Load**    | Thấp      | Cao                  | Thấp               | Thấp               |
| **Build Time**     | Nhanh     | N/A                  | Chậm (nhiều pages) | Chậm               |
| **Scalability**    | Cao       | Thấp                 | Cao nhất           | Cao                |
| **CDN**            | ✅        | ❌                   | ✅                 | ✅                 |
| **Cost**           | Thấp      | Cao                  | Thấp nhất          | Thấp               |

### Performance Metrics

```
CSR:
├─ TTFB: ~100ms
├─ FCP: ~2000ms (phải load JS)
├─ LCP: ~3000ms
└─ TTI: ~3500ms

SSR:
├─ TTFB: ~500ms (server processing)
├─ FCP: ~800ms
├─ LCP: ~1200ms
└─ TTI: ~2000ms (hydration)

SSG:
├─ TTFB: ~50ms (static file)
├─ FCP: ~300ms
├─ LCP: ~500ms
└─ TTI: ~1000ms

ISR:
├─ TTFB: ~50ms (cached)
├─ FCP: ~300ms
├─ LCP: ~500ms
└─ TTI: ~1000ms
```

### Use Case Matrix

```javascript
// Decision Tree

const chooseRenderingMethod = (requirements) => {
  // SEO không quan trọng?
  if (!requirements.needsSEO) {
    return "CSR";
  }

  // Data thay đổi liên tục?
  if (requirements.dataChangesEverySecond) {
    return "SSR";
  }

  // Data không bao giờ thay đổi?
  if (requirements.staticContent) {
    return "SSG";
  }

  // Data thay đổi định kỳ?
  if (requirements.dataChangesHourly || requirements.dataChangesDaily) {
    return "ISR";
  }

  // Personalized content?
  if (requirements.userSpecific) {
    return "SSR";
  }

  // Default
  return "ISR";
};
```

### Real-World Examples

```javascript
// E-commerce Website

const ecommerceStrategy = {
  // Homepage: Thay đổi thường xuyên, cần SEO
  "/": "ISR (revalidate: 300)", // 5 phút

  // Category pages: Ít thay đổi, cần SEO
  "/category/[slug]": "ISR (revalidate: 3600)", // 1 giờ

  // Product pages: Cần SEO, stock thay đổi
  "/product/[id]": "ISR (revalidate: 60)", // 1 phút

  // User dashboard: Personalized, không cần SEO
  "/dashboard": "CSR",

  // Cart: Real-time, không cần SEO
  "/cart": "CSR",

  // Checkout: Personalized, secure
  "/checkout": "SSR",

  // Blog: Static content
  "/blog/[slug]": "SSG",

  // Search results: Dynamic
  "/search": "CSR",
};
```

---

## Khi Nào Dùng Gì?

### CSR - Client-Side Rendering

```javascript
// ✅ Dùng khi:
- Dashboard, admin panel
- Authenticated pages (không cần SEO)
- Real-time data (chat, notifications)
- Interactive tools (calculators, games)
- Data visualization
- Infinite scroll, filters

// ❌ Không dùng khi:
- Cần SEO
- Cần fast initial load
- Target users có slow devices
- Content-heavy pages
```

### SSR - Server-Side Rendering

```javascript
// ✅ Dùng khi:
- Personalized content + SEO
- Data thay đổi mỗi request
- User-specific pages cần SEO
- Real-time data + SEO
- Authentication-based content + SEO

// ❌ Không dùng khi:
- Static content (dùng SSG)
- High traffic (tốn server)
- Data không thay đổi thường xuyên
```

### SSG - Static Site Generation

```javascript
// ✅ Dùng khi:
- Blog, documentation
- Marketing pages
- Landing pages
- Portfolio
- Content không thay đổi
- Cần performance tốt nhất

// ❌ Không dùng khi:
- Data thay đổi thường xuyên
- Personalized content
- User-specific data
- Real-time updates
```

### ISR - Incremental Static Regeneration

```javascript
// ✅ Dùng khi:
- E-commerce product pages
- News websites
- Blog với engagement metrics
- Data thay đổi định kỳ
- Cần balance performance + freshness
- Large-scale websites

// ❌ Không dùng khi:
- Data phải real-time
- Content hoàn toàn static
- Cần control chính xác cache
```

---

## Next.js App Router (13+)

### Server Components vs Client Components

```javascript
// Server Component (default)
// app/products/page.js

async function getProducts() {
  const res = await fetch('https://api.example.com/products');
  return res.json();
}

export default async function Products() {
  const products = await getProducts();

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Client Component
// app/components/ProductCard.js
'use client';

import { useState } from 'react';

export default function ProductCard({ product }) {
  const [liked, setLiked] = useState(false);

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => setLiked(!liked)}>
        {liked ? '❤️' : '🤍'}
      </button>
    </div>
  );
}
```

### Fetch Options trong App Router

```javascript
// 1. SSG (default)
fetch("https://api.example.com/data", {
  cache: "force-cache",
});

// 2. SSR
fetch("https://api.example.com/data", {
  cache: "no-store",
});

// 3. ISR
fetch("https://api.example.com/data", {
  next: { revalidate: 60 },
});

// 4. ISR với tags
fetch("https://api.example.com/data", {
  next: {
    revalidate: 3600,
    tags: ["products"],
  },
});
```

### Route Segment Config

```javascript
// app/products/page.js

// Force dynamic rendering (SSR)
export const dynamic = "force-dynamic";

// Force static rendering (SSG)
export const dynamic = "force-static";

// ISR
export const revalidate = 60;

// Opt out of caching
export const fetchCache = "force-no-store";

async function getProducts() {
  const res = await fetch("https://api.example.com/products");
  return res.json();
}

export default async function Products() {
  const products = await getProducts();
  return <div>{/* ... */}</div>;
}
```

### Streaming và Suspense

```javascript
// app/dashboard/page.js

import { Suspense } from "react";

async function SlowComponent() {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  return <div>Slow data loaded!</div>;
}

async function FastComponent() {
  return <div>Fast data loaded!</div>;
}

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Fast component render ngay */}
      <FastComponent />

      {/* Slow component stream sau */}
      <Suspense fallback={<div>Loading slow data...</div>}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

### Parallel Data Fetching

```javascript
// app/user/[id]/page.js

async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

async function getUserPosts(id) {
  const res = await fetch(`/api/users/${id}/posts`);
  return res.json();
}

export default async function UserProfile({ params }) {
  // Parallel fetching
  const [user, posts] = await Promise.all([getUser(params.id), getUserPosts(params.id)]);

  return (
    <div>
      <h1>{user.name}</h1>
      <PostList posts={posts} />
    </div>
  );
}
```

---

## Best Practices

### 1. Hybrid Rendering Strategy

```javascript
// Kết hợp nhiều phương pháp trong một app

// next.config.js
module.exports = {
  // Tối ưu cho production
  swcMinify: true,
  compress: true,

  // Image optimization
  images: {
    domains: ["cdn.example.com"],
    formats: ["image/avif", "image/webp"],
  },
};

// pages/index.js - ISR cho homepage
export const getStaticProps = async () => {
  return {
    props: { data: await fetchData() },
    revalidate: 300,
  };
};

// pages/dashboard.js - CSR cho dashboard
export default function Dashboard() {
  const { data } = useSWR("/api/user", fetcher);
  return <div>{/* ... */}</div>;
}

// pages/product/[id].js - ISR cho products
export const getStaticProps = async ({ params }) => {
  return {
    props: { product: await fetchProduct(params.id) },
    revalidate: 60,
  };
};

// pages/checkout.js - SSR cho checkout
export const getServerSideProps = async (context) => {
  const session = await getSession(context);
  return { props: { session } };
};
```

### 2. Data Fetching Best Practices

```javascript
// ✅ GOOD: Parallel fetching
export async function getStaticProps() {
  const [products, categories, featured] = await Promise.all([
    fetch("/api/products").then((r) => r.json()),
    fetch("/api/categories").then((r) => r.json()),
    fetch("/api/featured").then((r) => r.json()),
  ]);

  return {
    props: { products, categories, featured },
    revalidate: 60,
  };
}

// ❌ BAD: Sequential fetching
export async function getStaticProps() {
  const products = await fetch("/api/products").then((r) => r.json());
  const categories = await fetch("/api/categories").then((r) => r.json());
  const featured = await fetch("/api/featured").then((r) => r.json());

  return {
    props: { products, categories, featured },
    revalidate: 60,
  };
}
```

### 3. Error Handling

```javascript
// pages/products/[id].js

export async function getStaticProps({ params }) {
  try {
    const product = await fetch(`/api/products/${params.id}`).then((res) => {
      if (!res.ok) throw new Error("Product not found");
      return res.json();
    });

    return {
      props: { product },
      revalidate: 60,
    };
  } catch (error) {
    return {
      notFound: true, // Hiển thị 404 page
    };
  }
}

export async function getStaticPaths() {
  return {
    paths: [],
    fallback: "blocking", // Generate on-demand
  };
}

export default function Product({ product }) {
  return <div>{product.name}</div>;
}
```

### 4. Caching Strategy

```javascript
// pages/api/products.js

export default async function handler(req, res) {
  // Set cache headers
  res.setHeader("Cache-Control", "public, s-maxage=60, stale-while-revalidate=120");

  const products = await fetchProducts();
  res.json(products);
}

// Giải thích:
// - public: Cache được share giữa users
// - s-maxage=60: CDN cache 60 giây
// - stale-while-revalidate=120: Serve stale content trong 120s khi revalidate
```

### 5. Incremental Adoption

```javascript
// Chuyển từ CSR sang SSG/ISR từng bước

// Bước 1: CSR (hiện tại)
export default function Products() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('/api/products')
      .then(r => r.json())
      .then(setProducts);
  }, []);

  return <div>{/* ... */}</div>;
}

// Bước 2: Thêm SSG cho initial data
export async function getStaticProps() {
  const products = await fetch('/api/products').then(r => r.json());

  return {
    props: { initialProducts: products }
  };
}

export default function Products({ initialProducts }) {
  const [products, setProducts] = useState(initialProducts);

  // Vẫn có thể refetch nếu cần
  const refresh = () => {
    fetch('/api/products')
      .then(r => r.json())
      .then(setProducts);
  };

  return <div>{/* ... */}</div>;
}

// Bước 3: Thêm ISR
export async function getStaticProps() {
  const products = await fetch('/api/products').then(r => r.json());

  return {
    props: { products },
    revalidate: 60 // ISR
  };
}

export default function Products({ products }) {
  return <div>{/* ... */}</div>;
}
```

### 6. Performance Optimization

```javascript
// 1. Code Splitting
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('../components/Heavy'), {
  loading: () => <p>Loading...</p>,
  ssr: false // Không SSR component này
});

// 2. Image Optimization
import Image from 'next/image';

export default function Product({ product }) {
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={500}
      height={500}
      priority // LCP image
      placeholder="blur"
      blurDataURL={product.blurDataURL}
    />
  );
}

// 3. Font Optimization
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function App({ Component, pageProps }) {
  return (
    <main className={inter.className}>
      <Component {...pageProps} />
    </main>
  );
}

// 4. Prefetching
import Link from 'next/link';

export default function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <Link
          key={product.id}
          href={`/product/${product.id}`}
          prefetch={true} // Prefetch khi link visible
        >
          {product.name}
        </Link>
      ))}
    </div>
  );
}
```

### 7. SEO Optimization

```javascript
// pages/product/[id].js

import Head from "next/head";

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);

  return {
    props: { product },
    revalidate: 60,
  };
}

export default function Product({ product }) {
  return (
    <>
      <Head>
        <title>{product.name} | My Store</title>
        <meta name="description" content={product.description} />

        {/* Open Graph */}
        <meta property="og:title" content={product.name} />
        <meta property="og:description" content={product.description} />
        <meta property="og:image" content={product.image} />
        <meta property="og:type" content="product" />

        {/* Twitter Card */}
        <meta name="twitter:card" content="summary_large_image" />
        <meta name="twitter:title" content={product.name} />
        <meta name="twitter:description" content={product.description} />
        <meta name="twitter:image" content={product.image} />

        {/* Structured Data */}
        <script
          type="application/ld+json"
          dangerouslySetInnerHTML={{
            __html: JSON.stringify({
              "@context": "https://schema.org",
              "@type": "Product",
              name: product.name,
              description: product.description,
              image: product.image,
              offers: {
                "@type": "Offer",
                price: product.price,
                priceCurrency: "USD",
              },
            }),
          }}
        />
      </Head>

      <div>
        <h1>{product.name}</h1>
        <p>{product.description}</p>
      </div>
    </>
  );
}
```

### 8. Monitoring và Analytics

```javascript
// pages/_app.js

import { useEffect } from "react";
import { useRouter } from "next/router";

export default function App({ Component, pageProps }) {
  const router = useRouter();

  useEffect(() => {
    // Track page views
    const handleRouteChange = (url) => {
      // Google Analytics
      window.gtag("config", "GA_MEASUREMENT_ID", {
        page_path: url,
      });

      // Custom analytics
      trackPageView(url);
    };

    router.events.on("routeChangeComplete", handleRouteChange);
    return () => {
      router.events.off("routeChangeComplete", handleRouteChange);
    };
  }, [router.events]);

  return <Component {...pageProps} />;
}

// Web Vitals
export function reportWebVitals(metric) {
  console.log(metric);

  // Send to analytics
  if (metric.label === "web-vital") {
    sendToAnalytics({
      name: metric.name,
      value: metric.value,
      id: metric.id,
    });
  }
}
```

### 9. Environment-Specific Rendering

```javascript
// next.config.js

module.exports = {
  env: {
    API_URL: process.env.API_URL,
  },

  // Redirect trong production
  async redirects() {
    return [
      {
        source: "/old-blog/:slug",
        destination: "/blog/:slug",
        permanent: true,
      },
    ];
  },

  // Rewrite cho API proxy
  async rewrites() {
    return [
      {
        source: "/api/:path*",
        destination: "https://api.example.com/:path*",
      },
    ];
  },
};

// pages/index.js
export async function getStaticProps() {
  const isDev = process.env.NODE_ENV === "development";

  return {
    props: { isDev },
    revalidate: isDev ? 1 : 300, // Dev: 1s, Prod: 5min
  };
}
```

### 10. Testing Strategy

```javascript
// __tests__/pages/product.test.js

import { render, screen } from "@testing-library/react";
import Product, { getStaticProps } from "../pages/product/[id]";

// Test component
describe("Product Page", () => {
  it("renders product information", () => {
    const product = {
      id: 1,
      name: "Test Product",
      price: 99.99,
    };

    render(<Product product={product} />);

    expect(screen.getByText("Test Product")).toBeInTheDocument();
    expect(screen.getByText("$99.99")).toBeInTheDocument();
  });
});

// Test getStaticProps
describe("getStaticProps", () => {
  it("fetches product data", async () => {
    const context = {
      params: { id: "1" },
    };

    const result = await getStaticProps(context);

    expect(result.props.product).toBeDefined();
    expect(result.revalidate).toBe(60);
  });
});
```

---

## Common Pitfalls và Solutions

### 1. Hydration Mismatch

```javascript
// ❌ BAD: Server và client render khác nhau
export default function Component() {
  return <div>{new Date().toISOString()}</div>;
}

// ✅ GOOD: Chỉ render trên client
export default function Component() {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) return null;

  return <div>{new Date().toISOString()}</div>;
}
```

### 2. Large Build Times

```javascript
// ❌ BAD: Generate tất cả pages
export async function getStaticPaths() {
  const products = await fetchAllProducts(); // 100,000 products

  const paths = products.map((p) => ({
    params: { id: p.id.toString() },
  }));

  return { paths, fallback: false };
}

// ✅ GOOD: Generate popular pages, fallback cho còn lại
export async function getStaticPaths() {
  const popularProducts = await fetchPopularProducts(100);

  const paths = popularProducts.map((p) => ({
    params: { id: p.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // Generate on-demand
  };
}
```

### 3. Stale Data Issues

```javascript
// ❌ BAD: Không có revalidation strategy
export async function getStaticProps() {
  const data = await fetchData();
  return { props: { data } };
}

// ✅ GOOD: ISR với on-demand revalidation
export async function getStaticProps() {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60, // Background revalidation
  };
}

// API route cho on-demand revalidation
// pages/api/revalidate.js
export default async function handler(req, res) {
  await res.revalidate("/products/123");
  return res.json({ revalidated: true });
}
```

### 4. Memory Leaks

```javascript
// ❌ BAD: Không cleanup
export default function Component() {
  useEffect(() => {
    const interval = setInterval(() => {
      fetchData();
    }, 1000);
  }, []);

  return <div>...</div>;
}

// ✅ GOOD: Cleanup properly
export default function Component() {
  useEffect(() => {
    const interval = setInterval(() => {
      fetchData();
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <div>...</div>;
}
```

---

## Deployment Considerations

### Vercel (Recommended)

```bash
# Tự động detect Next.js
# Zero config deployment
# ISR, Edge Functions support

vercel deploy
```

### Self-Hosted (Node.js)

```bash
# Build
npm run build

# Start production server
npm run start

# Hoặc với PM2
pm2 start npm --name "nextjs-app" -- start
```

### Docker

```dockerfile
# Dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

### Static Export (SSG Only)

```javascript
// next.config.js
module.exports = {
  output: "export",
  images: {
    unoptimized: true,
  },
};

// Build
// npm run build
// → Tạo folder 'out' với static files
// → Deploy lên Netlify, GitHub Pages, S3, etc.
```

---

## Tổng Kết

### Decision Flowchart

```
Cần SEO?
├─ Không → CSR
└─ Có
   └─ Data thay đổi như thế nào?
      ├─ Mỗi request → SSR
      ├─ Không bao giờ → SSG
      ├─ Định kỳ (phút/giờ) → ISR
      └─ User-specific → SSR hoặc CSR + SSG
```

### Key Takeaways

1. **Không có one-size-fits-all**: Mỗi page có thể dùng phương pháp khác nhau
2. **ISR là sweet spot**: Cho hầu hết use cases cần SEO + performance
3. **Measure performance**: Dùng Lighthouse, Web Vitals để đo lường
4. **Start simple**: Bắt đầu với SSG/ISR, optimize sau
5. **Consider costs**: SSR tốn server resources, SSG/ISR rẻ hơn

### Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Web.dev Performance](https://web.dev/performance/)
- [Vercel Analytics](https://vercel.com/analytics)

---

**Lưu Ý**: Tài liệu này dựa trên Next.js 13-14. Luôn check documentation mới nhất vì Next.js update thường xuyên.
