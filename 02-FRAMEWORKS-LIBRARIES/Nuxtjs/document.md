# Tài liệu Nuxt 4 - Hướng dẫn đầy đủ

## 📋 Mục lục

1. [Giới thiệu Nuxt 4](#giới-thiệu-nuxt-4)
2. [Cài đặt](#cài-đặt)
3. [Cấu trúc thư mục mới](#cấu-trúc-thư-mục-mới)
4. [Những thay đổi quan trọng](#những-thay-đổi-quan-trọng)
5. [Data Fetching](#data-fetching)
6. [TypeScript](#typescript)
7. [Cấu hình](#cấu-hình)
8. [Migration từ Nuxt 3](#migration-từ-nuxt-3)
9. [Best Practices](#best-practices)

---

**Nuxt.js** là một framework mã nguồn mở, được xây dựng trên Vue.js, giúp đơn giản hóa việc phát triển các ứng dụng web phức hợp như Server-Side Rendering (SSR), Static Site Generation (SSG) và Single Page Applications (SPA). Nó cung cấp nhiều tính năng và cấu hình được thiết lập sẵn, giúp nhà phát triển tập trung vào việc viết mã logic thay vì xử lý các cấu hình phức tạp của công cụ như Webpack hoặc Vue Router.  

## 🚀 Giới thiệu Nuxt 4

Nuxt 4 là bản phát hành chủ yếu tập trung vào **cải thiện trải nghiệm developer**, được phát hành chính thức sau một năm testing trong môi trường production.

### Điểm nổi bật

- ✅ **Cấu trúc project mới** với thư mục `app/`
- ✅ **Data fetching thông minh hơn** với `useAsyncData` và `useFetch`
- ✅ **TypeScript tốt hơn** với project separation
- ✅ **Performance cải thiện** với CLI nhanh hơn
- ✅ **Backward compatible** - dễ dàng upgrade từ Nuxt 3

---

## 💻 Cài đặt

### Yêu cầu hệ thống

- **Node.js**: v20.x hoặc mới hơn (khuyến nghị LTS)
- **Editor**: VS Code (với Vue extension) hoặc WebStorm
- **Terminal**: để chạy commands

### Tạo project mới

```bash
# NPM
npm create nuxt@latest my-app

# Yarn
yarn create nuxt@latest my-app

# PNPM
pnpm create nuxt@latest my-app

# Bun
bun create nuxt@latest my-app
```

### Chạy development server

```bash
cd my-app
npm run dev
```

Truy cập: `http://localhost:3000`

### Cài đặt vào project có sẵn

```bash
npm install nuxt@latest
```

---

## 📁 Cấu trúc thư mục mới

Nuxt 4 giới thiệu cấu trúc thư mục **`app/`** để tổ chức code tốt hơn:

```
my-nuxt-app/
├── app/                    # ✨ THƯ MỤC MỚI - Code ứng dụng
│   ├── assets/            # CSS, images, fonts
│   ├── components/        # Vue components
│   ├── composables/       # Composable functions
│   ├── layouts/           # Layout components
│   ├── middleware/        # Route middleware
│   ├── pages/            # File-based routing
│   ├── plugins/          # Vue plugins
│   ├── utils/            # Utility functions
│   ├── app.vue           # Root component
│   ├── app.config.ts     # App configuration
│   └── error.vue         # Error page
├── content/              # Nuxt Content files
├── public/               # Static files
├── shared/               # ✨ Code dùng chung client & server
├── server/               # Server-side code
│   ├── api/             # API endpoints
│   ├── middleware/      # Server middleware
│   └── utils/           # Server utilities
├── nuxt.config.ts       # Nuxt configuration
├── tsconfig.json        # ✨ CHỈ MỘT FILE duy nhất
└── package.json
```

### Lợi ích của cấu trúc mới

1. **Performance tốt hơn**: File watching nhanh hơn (đặc biệt trên Windows/Linux)
2. **IDE thông minh hơn**: Biết được context (client vs server code)
3. **Tổ chức rõ ràng**: Tách biệt app code khỏi `node_modules/`, `.git/`

### Cấu trúc cũ vẫn được hỗ trợ

Nếu không muốn dùng thư mục `app/`, có thể giữ nguyên cấu trúc Nuxt 3:

```
my-nuxt-app/
├── assets/
├── components/
├── pages/
├── layouts/
├── plugins/
└── nuxt.config.ts
```

---

## 🔄 Những thay đổi quan trọng

### 1. App Directory (Mặc định)

**Nuxt 3:**
```
components/
pages/
layouts/
```

**Nuxt 4:**
```
app/
  ├── components/
  ├── pages/
  └── layouts/
```

**Tắt app directory** (giữ nguyên cấu trúc cũ):

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  future: {
    compatibilityVersion: 4,
  },
  compatibilityDate: '2024-11-01',
  dir: {
    app: '.', // Tắt app directory
  }
})
```

### 2. TypeScript Configuration

Nuxt 4 chỉ cần **MỘT file `tsconfig.json`** duy nhất ở root:

```json
{
  "extends": "./.nuxt/tsconfig.json"
}
```

Nuxt tự động tạo separate TypeScript projects cho:
- App code (client)
- Server code
- Shared code
- Build configuration

### 3. Shared Directory

Thư mục mới `shared/` để chia sẻ code giữa client và server:

```typescript
// shared/utils/math.ts
export function add(a: number, b: number) {
  return a + b
}
```

Dùng ở cả client và server:

```vue
<!-- app/pages/index.vue -->
<script setup>
import { add } from '~/shared/utils/math'
const result = add(2, 3) // 5
</script>
```

```typescript
// server/api/calculate.ts
import { add } from '~/shared/utils/math'

export default defineEventHandler(() => {
  return add(10, 20) // 30
})
```

### 4. Defaults Changes

**Route Rules Defaults:**

```typescript
// Nuxt 3
routeRules: {
  '/': { prerender: true }
}

// Nuxt 4 - mặc định prerender
routeRules: {
  '/': {} // Tự động prerender
}
```

**Data Fetching Defaults:**

```typescript
// Nuxt 4 - server: false mặc định cho client-only fetching
const { data } = await useFetch('/api/data', {
  server: false // Mặc định trong Nuxt 4
})
```

---

## 🔌 Data Fetching

Nuxt 4 cải thiện đáng kể `useAsyncData` và `useFetch`.

### useAsyncData

**Tính năng mới:**
- ✅ Shared data giữa components cùng key
- ✅ Tự động cleanup khi component unmount
- ✅ Reactive keys để refetch
- ✅ Kiểm soát cache tốt hơn

```vue
<script setup>
// Shared data - nhiều components dùng chung
const { data: user } = await useAsyncData('user', () => 
  $fetch('/api/user')
)

// Reactive key - tự động refetch khi id thay đổi
const route = useRoute()
const { data: post } = await useAsyncData(
  () => `post-${route.params.id}`,
  () => $fetch(`/api/posts/${route.params.id}`)
)

// Cache control
const { data, refresh } = await useAsyncData(
  'products',
  () => $fetch('/api/products'),
  {
    getCachedData(key) {
      // Custom cache logic
      return useNuxtApp().payload.data[key]
    }
  }
)
</script>
```

### useFetch

Wrapper của `useAsyncData` với `$fetch`:

```vue
<script setup>
// Basic usage
const { data, pending, error, refresh } = await useFetch('/api/data')

// With options
const { data } = await useFetch('/api/users', {
  method: 'POST',
  body: { name: 'John' },
  headers: {
    'Authorization': 'Bearer token'
  },
  // Server: false = chỉ fetch ở client
  server: false,
  // Lazy: true = không block navigation
  lazy: true
})

// Reactive URL
const page = ref(1)
const { data } = await useFetch(() => `/api/posts?page=${page.value}`)
</script>
```

### Best Practices

```vue
<script setup>
// ✅ ĐÚNG: Dùng key duy nhất
const { data } = await useAsyncData('unique-key', fetchFunction)

// ❌ SAI: Không có key (auto-generated, có thể trùng)
const { data } = await useAsyncData(fetchFunction)

// ✅ ĐÚNG: Server-side fetching
const { data } = await useFetch('/api/data', { server: true })

// ✅ ĐÚNG: Client-side fetching (không block SSR)
const { data } = await useFetch('/api/data', { server: false })

// ✅ ĐÚNG: Lazy loading (không block navigation)
const { data } = await useLazyFetch('/api/slow-endpoint')
</script>
```

---

## 📘 TypeScript

### Single tsconfig.json

Chỉ cần một file duy nhất:

```json
{
  "extends": "./.nuxt/tsconfig.json",
  "compilerOptions": {
    "strict": true
  }
}
```

### Type-safe Routes

```typescript
// Auto-generated types cho routes
const router = useRouter()

router.push({
  name: 'posts-id', // Type-safe!
  params: { id: '123' }
})
```

### Type-safe API

```typescript
// server/api/user.ts
export default defineEventHandler(() => {
  return {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com'
  }
})

// app/pages/index.vue
const { data } = await useFetch('/api/user')
// data is typed as { id: number, name: string, email: string }
```

### Composable Types

```typescript
// app/composables/useCounter.ts
export const useCounter = () => {
  const count = ref<number>(0)
  
  const increment = () => {
    count.value++
  }
  
  return {
    count: readonly(count),
    increment
  }
}

// Tự động import và type-safe
const { count, increment } = useCounter()
```

---

## ⚙️ Cấu hình

### nuxt.config.ts

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Bật Nuxt 4 features
  future: {
    compatibilityVersion: 4,
  },
  
  // App configuration
  app: {
    head: {
      title: 'My App',
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1' }
      ],