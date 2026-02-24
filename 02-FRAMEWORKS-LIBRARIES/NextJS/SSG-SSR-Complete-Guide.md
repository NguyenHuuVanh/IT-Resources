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

Next.js hỗ trợ nhiều phương thức rendering khác nhau, cho phép developer chọn cách tối ưu nhất cho từng trang/component.

### Các Phương Thức Rendering

- **CSR** (Client-Side Rendering): Render trên browser
- **SSR** (Server-Side Rendering): Render trên server mỗi request
- **SSG** (Static Site Generation): Pre-render tại build time
- **ISR** (Incremental Static Regeneration): SSG + revalidation

### Timeline Evolution

```
React (CSR) → Next.js 9 (SSR, SSG) → Next.js 12 (ISR) → Next.js 13+ (App Router, RSC)
```

---

## Client-Side Rendering (CSR)

### Khái Niệm

HTML rỗng được gửi về browser, JavaScript tải về và render UI trên client.

### Cách Hoạt Động

```
1. Browser request → Server
2. Server trả về HTML rỗng + JS bundle
3. Browser download JS
4. JS execute → Fetch data → Render UI
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

HTML được render trên server cho mỗi request, gửi về browser đã có content đầy đủ.

### Cách Hoạt Động

```
1. Browser request → Server
2. Server fetch data
3. Server render HTML với data
4. Server trả về HTML đầy đủ
5. Browser hiển thị (hydration sau)
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

HTML được pre-render tại build time, serve file tĩnh cho mọi request.

### Cách Hoạt Động

```
Build Time:
1. Next.js fetch data
2. Generate HTML files
3. Save to disk

Request Time:
1. Browser request → CDN/Server
2. Serve pre-built HTML (instant)
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

Kết hợp SSG + revalidation: pre-render tại build time, tự động regenerate sau một khoảng thời gian.

### Cách Hoạt Động

```
1. Build time: Generate static pages
2. Request 1: Serve static page (fast)
3. After revalidate time: Background regeneration
4. Request 2: Serve old page, regenerate in background
5. Request 3+: Serve new page
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
