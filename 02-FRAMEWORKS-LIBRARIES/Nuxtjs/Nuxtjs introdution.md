# Giới thiệu về Nuxt.js

## 📌 Mục lục

1. [Nuxt.js là gì?](#nuxtjs-là-gì)
2. [Tự động hóa và Quy ước](#tự-động-hóa-và-quy-ước)
3. [Server-Side Rendering](#server-side-rendering)
4. [Các chế độ Rendering](#các-chế-độ-rendering)
5. [Server Engine (Nitro)](#server-engine)
6. [Cấu trúc thư mục](#cấu-trúc-thư-mục)
7. [Routing trong Nuxt](#routing-trong-nuxt)
8. [Data Fetching](#data-fetching)
9. [State Management](#state-management)
10. [Hệ thống Module](#hệ-thống-module)
11. [Kiến trúc](#kiến-trúc)
12. [So sánh Nuxt vs Vue thuần](#so-sánh-nuxt-vs-vue-thuần)
13. [Khi nào nên dùng Nuxt?](#khi-nào-nên-dùng-nuxt)

---

## Nuxt.js là gì?

Nuxt là một framework miễn phí và mã nguồn mở với cách tiếp cận trực quan và có thể mở rộng để tạo ra các ứng dụng web full-stack an toàn về kiểu dữ liệu, hiệu suất cao và sẵn sàng cho production với Vue.js.

### Tại sao chọn Nuxt?

| Đặc điểm                 | Mô tả                                   |
| ------------------------ | --------------------------------------- |
| **Full-stack**           | Frontend + Backend trong một project    |
| **Type-safe**            | TypeScript support tự động              |
| **SEO-friendly**         | SSR mặc định, tối ưu cho search engines |
| **Developer Experience** | Hot reload, auto-imports, zero config   |
| **Flexible**             | SSR, SSG, SPA, Hybrid rendering         |

Chúng tôi đã làm mọi thứ để bạn có thể bắt đầu viết các file `.vue` ngay từ đầu, đồng thời tận hưởng tính năng hot module replacement trong quá trình phát triển và một ứng dụng hiệu suất cao trong production với server-side rendering mặc định.

Nuxt không có vendor lock-in, cho phép bạn triển khai ứng dụng của mình ở mọi nơi, ngay cả trên edge.

## Tự động hóa và Quy ước

Nuxt sử dụng các quy ước và cấu trúc thư mục có định hướng để tự động hóa các tác vụ lặp đi lặp lại và cho phép các nhà phát triển tập trung vào việc phát triển tính năng. File cấu hình vẫn có thể tùy chỉnh và ghi đè các hành vi mặc định của nó.

- **File-based routing**: Định nghĩa routes dựa trên cấu trúc thư mục `app/pages/` của bạn. Điều này giúp việc tổ chức ứng dụng của bạn dễ dàng hơn và tránh cần phải cấu hình routing thủ công.

- **Code splitting**: Nuxt tự động chia code của bạn thành các chunk nhỏ hơn, giúp giảm thời gian tải ban đầu của ứng dụng.

- **Server-side rendering tích hợp sẵn**: Được xây dựng với khả năng SSR ngay từ đầu, mang lại hiệu suất tốt hơn và tối ưu hóa SEO.

- **Auto-imports**: Viết composables và components trong các thư mục đã được định nghĩa và sử dụng chúng mà không cần import thủ công, với tính năng tree-shaking và các bundle được tối ưu hóa.

- **Data-fetching utilities**: Nuxt cung cấp các composables để xử lý data fetching tương thích với SSR cũng như các chiến lược caching khác nhau.

- **Zero-config TypeScript support**: Viết code an toàn về kiểu dữ liệu mà không cần học TypeScript, với các kiểu được tự động tạo và `tsconfig.json`.

- **Configured build tools**: Chúng tôi sử dụng Vite theo mặc định để hỗ trợ hot module replacement (HMR) trong development và bundle code của bạn cho production với các best practices được tích hợp sẵn.

Nuxt đảm nhiệm những việc này và cung cấp chức năng cả frontend và backend để bạn có thể tập trung vào điều quan trọng: tạo ứng dụng web của bạn.

---

## Server-Side Rendering

Nuxt đi kèm với khả năng server-side rendering (SSR) tích hợp sẵn theo mặc định, mà không cần phải tự cấu hình server, điều này mang lại nhiều lợi ích cho các ứng dụng web:

- **Thời gian tải trang ban đầu nhanh hơn**: Nuxt gửi một trang HTML được render đầy đủ đến trình duyệt, có thể hiển thị ngay lập tức. Điều này có thể cung cấp trải nghiệm người dùng cảm nhận nhanh hơn, đặc biệt trên các mạng hoặc thiết bị chậm hơn.

- **Cải thiện SEO**: Các search engine có thể lập chỉ mục tốt hơn nội dung của trang vì nội dung HTML có sẵn ngay lập tức, thay vì phải đợi JavaScript render nó.

- **Hiệu suất tốt hơn trên các thiết bị yếu**: Giảm lượng JavaScript cần được tải xuống và thực thi ở phía client, điều này có thể có lợi cho các thiết bị yếu có thể gặp khó khăn khi xử lý các ứng dụng JavaScript nặng.

- **Khả năng truy cập tốt hơn**: Nội dung có sẵn ngay lập tức khi tải trang ban đầu, cải thiện khả năng truy cập cho người dùng dựa vào screen readers hoặc các công nghệ hỗ trợ khác.

- **Caching dễ dàng hơn**: Các trang có thể được cache ở phía server, giúp cải thiện hiệu suất bằng cách giảm thời gian cần thiết để tạo và gửi nội dung.

Nhìn chung, server-side rendering có thể cung cấp trải nghiệm người dùng nhanh hơn và hiệu quả hơn, cũng như cải thiện tối ưu hóa công cụ tìm kiếm và khả năng truy cập.

Vì Nuxt là một framework linh hoạt, nó cho bạn khả năng render tĩnh toàn bộ ứng dụng của bạn sang static hosting với `nuxt generate`, vô hiệu hóa SSR toàn cục với tùy chọn `ssr: false` hoặc tận dụng hybrid rendering bằng cách thiết lập tùy chọn `routeRules`.

---

## Các chế độ Rendering

Nuxt hỗ trợ nhiều chế độ rendering khác nhau:

### 1. Server-Side Rendering (SSR) - Mặc định

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true, // Mặc định
});
```

- HTML được render trên server
- Gửi HTML đầy đủ đến client
- Tốt cho SEO và performance

### 2. Static Site Generation (SSG)

```bash
# Generate static files
npx nuxi generate
```

- Pre-render tất cả pages tại build time
- Deploy như static files
- Phù hợp cho blog, documentation

### 3. Single Page Application (SPA)

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: false,
});
```

- Render hoàn toàn ở client
- Giống Vue SPA truyền thống
- Không cần Node.js server

### 4. Hybrid Rendering

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    "/": { prerender: true }, // SSG
    "/api/**": { cors: true }, // API routes
    "/admin/**": { ssr: false }, // SPA
    "/blog/**": { swr: 3600 }, // ISR - revalidate mỗi 1h
    "/products/**": { isr: true }, // ISR
  },
});
```

- Mix các chế độ rendering theo route
- Linh hoạt tối đa

### So sánh các chế độ Rendering

| Chế độ     | SEO          | Performance   | Use Case                    |
| ---------- | ------------ | ------------- | --------------------------- |
| **SSR**    | ✅ Tốt       | ✅ Tốt        | Dynamic content, e-commerce |
| **SSG**    | ✅ Tốt nhất  | ✅ Tốt nhất   | Blog, docs, landing pages   |
| **SPA**    | ❌ Kém       | ⚠️ Trung bình | Admin panels, dashboards    |
| **Hybrid** | ✅ Tùy chỉnh | ✅ Tùy chỉnh  | Large apps với mixed needs  |

---

## Server Engine

Server engine Nuxt là Nitro, mở khóa các khả năng full-stack mới.

Trong development, nó sử dụng Rollup và Node.js workers cho server code của bạn và context isolation. Nó cũng tạo server API của bạn bằng cách đọc các file trong `server/api/` và server middleware từ `server/middleware/`.

Trong production, Nitro build app và server của bạn thành một thư mục `.output` universal. Output này nhẹ: được minified và loại bỏ khỏi bất kỳ Node.js modules nào (ngoại trừ polyfills). Bạn có thể triển khai output này trên bất kỳ hệ thống nào hỗ trợ JavaScript, từ Node.js, Serverless, Workers, Edge-side rendering hoặc hoàn toàn tĩnh.

## Production-Ready

Một ứng dụng Nuxt có thể được triển khai trên Node hoặc Deno server, được pre-render để host trong các môi trường tĩnh, hoặc được triển khai cho các serverless và edge providers.

### Deployment Options

| Platform   | Command         | Preset             |
| ---------- | --------------- | ------------------ |
| Node.js    | `nuxt build`    | `node-server`      |
| Static     | `nuxt generate` | `static`           |
| Vercel     | Auto-detect     | `vercel`           |
| Netlify    | Auto-detect     | `netlify`          |
| Cloudflare | `nuxt build`    | `cloudflare-pages` |
| AWS Lambda | `nuxt build`    | `aws-lambda`       |

---

## Cấu trúc thư mục

```
my-nuxt-app/
├── .nuxt/                 # Build directory (auto-generated)
├── .output/               # Production build output
├── app/                   # Application source (Nuxt 3.14+)
│   ├── components/        # Vue components (auto-imported)
│   ├── composables/       # Composables (auto-imported)
│   ├── layouts/           # Layout components
│   ├── middleware/        # Route middleware
│   ├── pages/             # File-based routing
│   ├── plugins/           # Plugins
│   └── app.vue            # Root component
├── public/                # Static assets (served at /)
├── server/                # Server-side code
│   ├── api/               # API routes
│   ├── middleware/        # Server middleware
│   └── plugins/           # Server plugins
├── nuxt.config.ts         # Nuxt configuration
├── package.json
└── tsconfig.json          # TypeScript config (auto-generated)
```

### Giải thích các thư mục quan trọng

| Thư mục        | Mục đích               | Auto-import |
| -------------- | ---------------------- | ----------- |
| `components/`  | Vue components         | ✅          |
| `composables/` | Reusable logic (hooks) | ✅          |
| `pages/`       | Routes của app         | ✅          |
| `layouts/`     | Page layouts           | ✅          |
| `middleware/`  | Route guards           | ❌          |
| `plugins/`     | Vue plugins            | ❌          |
| `server/api/`  | Backend API endpoints  | ✅          |
| `public/`      | Static files           | -           |

---

## Routing trong Nuxt

Nuxt sử dụng file-based routing - cấu trúc thư mục `pages/` tự động tạo routes.

### Cơ bản

```
pages/
├── index.vue          → /
├── about.vue          → /about
├── contact.vue        → /contact
└── users/
    ├── index.vue      → /users
    └── profile.vue    → /users/profile
```

### Dynamic Routes

```
pages/
├── users/
│   ├── [id].vue       → /users/:id
│   └── [id]/
│       └── posts.vue  → /users/:id/posts
├── posts/
│   └── [...slug].vue  → /posts/* (catch-all)
└── [[optional]].vue   → /:optional? (optional param)
```

### Ví dụ Dynamic Route

```vue
<!-- pages/users/[id].vue -->
<script setup>
const route = useRoute();
const { id } = route.params;

// Hoặc dùng useFetch
const { data: user } = await useFetch(`/api/users/${id}`);
</script>

<template>
  <div>
    <h1>User {{ id }}</h1>
    <p>{{ user?.name }}</p>
  </div>
</template>
```

### Nested Routes

```
pages/
└── parent/
    ├── index.vue      → /parent
    └── child.vue      → /parent/child
```

```vue
<!-- pages/parent.vue -->
<template>
  <div>
    <h1>Parent</h1>
    <NuxtPage />
    <!-- Render child routes -->
  </div>
</template>
```

### Navigation

```vue
<template>
  <!-- Declarative -->
  <NuxtLink to="/about">About</NuxtLink>
  <NuxtLink :to="{ name: 'users-id', params: { id: 1 } }">User 1</NuxtLink>

  <!-- Programmatic -->
  <button @click="navigateTo('/contact')">Contact</button>
</template>

<script setup>
// Programmatic navigation
const router = useRouter();
router.push("/about");

// Or use navigateTo
await navigateTo("/about");
</script>
```

---

## Data Fetching

Nuxt cung cấp các composables mạnh mẽ cho data fetching:

### useFetch

```vue
<script setup>
// Basic usage
const { data, pending, error, refresh } = await useFetch("/api/users");

// With options
const { data: posts } = await useFetch("/api/posts", {
  query: { page: 1, limit: 10 },
  pick: ["id", "title"], // Chỉ lấy fields cần thiết
  transform: (data) => data.items, // Transform response
  default: () => [], // Default value
  watch: [page], // Re-fetch khi page thay đổi
});
</script>
```

### useAsyncData

```vue
<script setup>
// Khi cần custom logic
const { data: user } = await useAsyncData("user", async () => {
  const [profile, posts] = await Promise.all([$fetch("/api/profile"), $fetch("/api/posts")]);
  return { profile, posts };
});
</script>
```

### useLazyFetch / useLazyAsyncData

```vue
<script setup>
// Non-blocking - không chờ data trước khi render
const { data, pending } = useLazyFetch("/api/users");
</script>

<template>
  <div v-if="pending">Loading...</div>
  <div v-else>{{ data }}</div>
</template>
```

### $fetch

```vue
<script setup>
// Direct fetch (không có SSR benefits)
const submitForm = async () => {
  const result = await $fetch("/api/submit", {
    method: "POST",
    body: { name: "John" },
  });
};
</script>
```

### So sánh Data Fetching Methods

| Method         | SSR | Caching | Use Case                   |
| -------------- | --- | ------- | -------------------------- |
| `useFetch`     | ✅  | ✅      | API calls trong components |
| `useAsyncData` | ✅  | ✅      | Custom async logic         |
| `useLazyFetch` | ❌  | ✅      | Non-critical data          |
| `$fetch`       | ❌  | ❌      | Event handlers, mutations  |

---

## State Management

### useState (Built-in)

```vue
<!-- composables/useCounter.ts -->
<script setup>
// Shared state across components
const count = useState("count", () => 0);

const increment = () => count.value++;
</script>
```

```vue
<!-- Sử dụng trong component -->
<script setup>
const count = useState("count");
</script>

<template>
  <p>Count: {{ count }}</p>
</template>
```

### Pinia (Recommended cho complex state)

```bash
npx nuxi module add pinia
```

```ts
// stores/user.ts
export const useUserStore = defineStore("user", {
  state: () => ({
    name: "",
    isLoggedIn: false,
  }),

  getters: {
    greeting: (state) => `Hello, ${state.name}!`,
  },

  actions: {
    async login(credentials) {
      const user = await $fetch("/api/login", {
        method: "POST",
        body: credentials,
      });
      this.name = user.name;
      this.isLoggedIn = true;
    },
  },
});
```

```vue
<script setup>
const userStore = useUserStore();
</script>

<template>
  <p>{{ userStore.greeting }}</p>
  <button @click="userStore.login({ email, password })">Login</button>
</template>
```

---

## Hệ thống Module

Một hệ thống module cho phép bạn mở rộng Nuxt với các tính năng tùy chỉnh và tích hợp với các dịch vụ bên thứ ba.

### Cài đặt Module

```bash
# Sử dụng nuxi
npx nuxi module add @nuxtjs/tailwindcss

# Hoặc cài thủ công
npm install @nuxtjs/tailwindcss
```

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxtjs/tailwindcss", "@pinia/nuxt", "@nuxt/image"],
});
```

### Các Module phổ biến

| Module                | Mục đích                 |
| --------------------- | ------------------------ |
| `@nuxtjs/tailwindcss` | Tailwind CSS integration |
| `@pinia/nuxt`         | State management         |
| `@nuxt/image`         | Image optimization       |
| `@nuxt/content`       | File-based CMS           |
| `@nuxtjs/i18n`        | Internationalization     |
| `@sidebase/nuxt-auth` | Authentication           |
| `@nuxt/ui`            | UI component library     |
| `@vueuse/nuxt`        | VueUse composables       |

### Tạo Module tùy chỉnh

```ts
// modules/my-module.ts
export default defineNuxtModule({
  meta: {
    name: "my-module",
    configKey: "myModule",
  },
  defaults: {
    enabled: true,
  },
  setup(options, nuxt) {
    if (!options.enabled) return;

    // Add composables
    addImports({
      name: "useMyFeature",
      from: resolve("./runtime/composables"),
    });
  },
});
```

---

## Kiến trúc

Nuxt được cấu thành từ các core packages khác nhau:

- **nuxt**: Core engine
- **@nuxt/schema**: Cross-version Nuxt type definitions
- **@nuxt/kit**: Toolkit để phát triển modules
- **@nuxt/vite-builder**: Builder dựa trên Vite
- **@nuxt/webpack-builder**: Builder dựa trên webpack
- **nitro**: Server engine
- **h3**: Minimal HTTP framework
- **nuxi**: CLI commands và utilities

Chúng tôi khuyên bạn nên đọc từng concept để có cái nhìn toàn diện về các khả năng của Nuxt và phạm vi của từng package.

---

## So sánh Nuxt vs Vue thuần

| Tiêu chí           | Vue.js (SPA)      | Nuxt.js               |
| ------------------ | ----------------- | --------------------- |
| **Rendering**      | Client-side only  | SSR, SSG, SPA, Hybrid |
| **SEO**            | ❌ Kém            | ✅ Tốt                |
| **Routing**        | Cấu hình thủ công | File-based (tự động)  |
| **Data Fetching**  | Tự implement      | Built-in composables  |
| **Backend**        | Cần server riêng  | Tích hợp sẵn (Nitro)  |
| **Auto-imports**   | ❌ Không          | ✅ Có                 |
| **TypeScript**     | Cấu hình thủ công | Zero-config           |
| **Build Tool**     | Vite/Webpack      | Vite (mặc định)       |
| **Learning Curve** | Thấp              | Trung bình            |
| **Bundle Size**    | Nhỏ hơn           | Lớn hơn một chút      |

---

## Khi nào nên dùng Nuxt?

### ✅ Nên dùng Nuxt khi:

```
✅ Cần SEO tốt (blog, e-commerce, marketing sites)
✅ Cần SSR hoặc SSG
✅ Muốn full-stack trong một project
✅ Dự án lớn, cần cấu trúc rõ ràng
✅ Team đã quen Vue.js
✅ Cần performance tốt (First Contentful Paint)
```

### ❌ Không cần Nuxt khi:

```
❌ App đơn giản, không cần SEO (admin panel, dashboard)
❌ Prototype nhanh
❌ Team chưa quen Vue
❌ Cần bundle size tối thiểu
❌ Chỉ cần SPA đơn giản
```

### Decision Flowchart

```
                    ┌─────────────────┐
                    │   Cần SEO tốt?  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │ Yes          │              │ No
              ▼              │              ▼
        ┌─────────┐          │      ┌───────────────┐
        │  Nuxt   │          │      │ Cần Backend   │
        │  (SSR)  │          │      │ tích hợp?     │
        └─────────┘          │      └───────┬───────┘
                             │              │
                             │   ┌──────────┼──────────┐
                             │   │ Yes      │          │ No
                             │   ▼          │          ▼
                             │ ┌──────┐     │    ┌──────────┐
                             │ │ Nuxt │     │    │ Vue SPA  │
                             │ └──────┘     │    └──────────┘
                             │              │
                             └──────────────┘
```

---

## 🚀 Quick Start

### Tạo project mới

```bash
# Sử dụng nuxi
npx nuxi@latest init my-app

# Di chuyển vào thư mục
cd my-app

# Cài dependencies
npm install

# Chạy development server
npm run dev
```

### Cấu hình cơ bản

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  // App config
  app: {
    head: {
      title: "My Nuxt App",
      meta: [{ name: "description", content: "My awesome Nuxt app" }],
    },
  },

  // Modules
  modules: ["@nuxtjs/tailwindcss", "@pinia/nuxt"],

  // Runtime config
  runtimeConfig: {
    apiSecret: "", // Server-only
    public: {
      apiBase: "", // Client + Server
    },
  },

  // Dev tools
  devtools: { enabled: true },
});
```

---

## 📚 Tài liệu tham khảo

- [Nuxt Documentation](https://nuxt.com/docs)
- [Nuxt Modules](https://nuxt.com/modules)
- [Nuxt Examples](https://nuxt.com/docs/examples)
- [Vue.js Documentation](https://vuejs.org/)
- [Nitro Documentation](https://nitro.unjs.io/)
