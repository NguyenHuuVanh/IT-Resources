# SEO cho Frontend Developer - Hướng dẫn toàn diện

## 📌 Mục lục

1. [Tổng quan về SEO](#1-tổng-quan-về-seo)
2. [HTML Semantic & Structure](#2-html-semantic--structure)
3. [Meta Tags](#3-meta-tags)
4. [Open Graph & Social Media](#4-open-graph--social-media)
5. [Structured Data (Schema.org)](#5-structured-data-schemaorg)
6. [Performance & Core Web Vitals](#6-performance--core-web-vitals)
7. [Images & Media Optimization](#7-images--media-optimization)
8. [URL Structure & Routing](#8-url-structure--routing)
9. [Mobile SEO](#9-mobile-seo)
10. [JavaScript SEO & Rendering](#10-javascript-seo--rendering)
11. [Technical SEO](#11-technical-seo)
12. [International SEO](#12-international-seo)
13. [SEO Tools & Testing](#13-seo-tools--testing)

---

## 1. Tổng quan về SEO

### 🎯 SEO là gì?

**SEO (Search Engine Optimization)** là tập hợp các kỹ thuật giúp website xuất hiện cao hơn trên kết quả tìm kiếm.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CÁC YẾU TỐ SEO CHÍNH                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ON-PAGE SEO (Frontend có thể kiểm soát)                              │
│   ├── Content quality & keywords                                        │
│   ├── HTML structure & semantics                                        │
│   ├── Meta tags & Open Graph                                           │
│   ├── URL structure                                                     │
│   ├── Internal linking                                                  │
│   ├── Image optimization                                                │
│   ├── Page speed & Core Web Vitals                                     │
│   └── Mobile responsiveness                                             │
│                                                                         │
│   TECHNICAL SEO                                                         │
│   ├── Crawlability & indexability                                      │
│   ├── XML sitemap                                                       │
│   ├── Robots.txt                                                        │
│   ├── Canonical URLs                                                    │
│   ├── HTTPS security                                                    │
│   ├── Structured data                                                   │
│   └── JavaScript rendering                                              │
│                                                                         │
│   OFF-PAGE SEO (Ngoài tầm kiểm soát Frontend)                          │
│   ├── Backlinks                                                         │
│   ├── Social signals                                                    │
│   └── Brand mentions                                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📊 Cách Search Engine hoạt động

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   CRAWLING  │────►│   PARSING   │────►│  INDEXING   │────►│   RANKING   │
│             │     │             │     │             │     │             │
│ Googlebot   │     │ Đọc HTML,   │     │ Lưu vào     │     │ Xếp hạng    │
│ truy cập    │     │ CSS, JS     │     │ database    │     │ kết quả     │
│ website     │     │             │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

## 2. HTML Semantic & Structure

### 🎯 Tại sao Semantic HTML quan trọng?

- Search engines hiểu nội dung tốt hơn
- Accessibility tốt hơn
- Maintainability cao hơn

### 📋 Document Structure

```html
<!DOCTYPE html>
<html lang="vi">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tiêu đề trang - Tên website</title>
    <meta name="description" content="Mô tả trang trong 150-160 ký tự" />
    <!-- Các meta tags khác -->
  </head>
  <body>
    <header>
      <nav aria-label="Main navigation">
        <!-- Navigation -->
      </nav>
    </header>

    <main>
      <article>
        <header>
          <h1>Tiêu đề chính (chỉ 1 H1 mỗi trang)</h1>
          <time datetime="2024-01-15">15 tháng 1, 2024</time>
        </header>

        <section>
          <h2>Phần 1</h2>
          <p>Nội dung...</p>
        </section>

        <section>
          <h2>Phần 2</h2>
          <h3>Phần con 2.1</h3>
          <p>Nội dung...</p>
        </section>
      </article>

      <aside>
        <!-- Sidebar content -->
      </aside>
    </main>

    <footer>
      <!-- Footer content -->
    </footer>
  </body>
</html>
```

### 📊 Semantic Elements

```html
<!-- ❌ BAD: Không semantic -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="content">
  <div class="article">
    <div class="title">Tiêu đề</div>
  </div>
</div>
<div class="footer">...</div>

<!-- ✅ GOOD: Semantic -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>
    <h1>Tiêu đề</h1>
  </article>
</main>
<footer>...</footer>
```

### 📋 Heading Hierarchy

```html
<!-- ✅ GOOD: Đúng thứ tự -->
<h1>Tiêu đề chính của trang</h1>
<h2>Phần 1</h2>
<h3>Phần con 1.1</h3>
<h3>Phần con 1.2</h3>
<h2>Phần 2</h2>
<h3>Phần con 2.1</h3>
<h4>Chi tiết 2.1.1</h4>

<!-- ❌ BAD: Nhảy cấp -->
<h1>Tiêu đề</h1>
<h3>Bỏ qua h2</h3>
<!-- Sai! -->
<h5>Nhảy cấp</h5>
<!-- Sai! -->
```

### 🔗 Links

```html
<!-- Internal links -->
<a href="/products/laptop">Xem laptop</a>

<!-- External links -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer"> Link ngoài </a>

<!-- Nofollow link (không truyền SEO juice) -->
<a href="https://ads.com" rel="nofollow">Quảng cáo</a>

<!-- Sponsored link -->
<a href="https://sponsor.com" rel="sponsored">Tài trợ</a>

<!-- UGC (User Generated Content) -->
<a href="https://user-link.com" rel="ugc">Link từ user</a>
```

---

## 3. Meta Tags

### 📋 Essential Meta Tags

```html
<head>
  <!-- Charset & Viewport (bắt buộc) -->
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Title (50-60 ký tự) -->
  <title>Từ khóa chính - Mô tả ngắn | Tên Brand</title>

  <!-- Description (150-160 ký tự) -->
  <meta
    name="description"
    content="Mô tả hấp dẫn chứa từ khóa chính, 
        giải thích nội dung trang và kêu gọi hành động."
  />

  <!-- Robots -->
  <meta name="robots" content="index, follow" />
  <!-- Hoặc noindex, nofollow cho trang không muốn index -->

  <!-- Canonical URL (tránh duplicate content) -->
  <link rel="canonical" href="https://example.com/page" />

  <!-- Language -->
  <meta name="language" content="Vietnamese" />
  <link rel="alternate" hreflang="vi" href="https://example.com/vi/page" />
  <link rel="alternate" hreflang="en" href="https://example.com/en/page" />

  <!-- Author -->
  <meta name="author" content="Tên tác giả" />

  <!-- Keywords (ít quan trọng với Google nhưng vẫn nên có) -->
  <meta name="keywords" content="từ khóa 1, từ khóa 2, từ khóa 3" />
</head>
```

### 🤖 Robots Meta Tag

```html
<!-- Cho phép index và follow links -->
<meta name="robots" content="index, follow" />

<!-- Không index nhưng follow links -->
<meta name="robots" content="noindex, follow" />

<!-- Index nhưng không follow links -->
<meta name="robots" content="index, nofollow" />

<!-- Không index, không follow -->
<meta name="robots" content="noindex, nofollow" />

<!-- Các directives khác -->
<meta name="robots" content="noarchive" />
<!-- Không cache -->
<meta name="robots" content="nosnippet" />
<!-- Không hiện snippet -->
<meta name="robots" content="noimageindex" />
<!-- Không index hình -->
<meta name="robots" content="max-snippet:150" />
<!-- Giới hạn snippet -->
<meta name="robots" content="max-image-preview:large" />
<!-- Kích thước preview -->

<!-- Cho specific bot -->
<meta name="googlebot" content="index, follow" />
<meta name="bingbot" content="index, follow" />
```

### 📱 Mobile Meta Tags

```html
<!-- Viewport (bắt buộc cho mobile) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0" />

<!-- Theme color (thanh địa chỉ trên mobile) -->
<meta name="theme-color" content="#4285f4" />

<!-- iOS specific -->
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
<meta name="apple-mobile-web-app-title" content="App Name" />
<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png" />

<!-- Android specific -->
<link rel="manifest" href="/manifest.json" />
```

---

## 4. Open Graph & Social Media

### 📘 Open Graph (Facebook)

```html
<!-- Basic OG Tags -->
<meta property="og:title" content="Tiêu đề bài viết" />
<meta property="og:description" content="Mô tả hấp dẫn cho social media" />
<meta property="og:image" content="https://example.com/image.jpg" />
<meta property="og:url" content="https://example.com/page" />
<meta property="og:type" content="website" />
<meta property="og:site_name" content="Tên Website" />
<meta property="og:locale" content="vi_VN" />

<!-- Image dimensions (recommended: 1200x630) -->
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:alt" content="Mô tả hình ảnh" />

<!-- For articles -->
<meta property="og:type" content="article" />
<meta property="article:published_time" content="2024-01-15T08:00:00+07:00" />
<meta property="article:modified_time" content="2024-01-16T10:00:00+07:00" />
<meta property="article:author" content="https://facebook.com/author" />
<meta property="article:section" content="Technology" />
<meta property="article:tag" content="JavaScript" />
<meta property="article:tag" content="SEO" />
```

### 🐦 Twitter Cards

```html
<!-- Summary Card -->
<meta name="twitter:card" content="summary" />
<meta name="twitter:site" content="@username" />
<meta name="twitter:creator" content="@author" />
<meta name="twitter:title" content="Tiêu đề" />
<meta name="twitter:description" content="Mô tả" />
<meta name="twitter:image" content="https://example.com/image.jpg" />

<!-- Large Image Card (recommended: 1200x600) -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:image:alt" content="Mô tả hình ảnh" />
```

### 📋 Complete Social Meta Template

```html
<head>
  <!-- Primary Meta Tags -->
  <title>Tiêu đề trang - Brand</title>
  <meta name="title" content="Tiêu đề trang - Brand" />
  <meta name="description" content="Mô tả trang" />

  <!-- Open Graph / Facebook -->
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://example.com/page" />
  <meta property="og:title" content="Tiêu đề trang - Brand" />
  <meta property="og:description" content="Mô tả trang" />
  <meta property="og:image" content="https://example.com/og-image.jpg" />

  <!-- Twitter -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:url" content="https://example.com/page" />
  <meta name="twitter:title" content="Tiêu đề trang - Brand" />
  <meta name="twitter:description" content="Mô tả trang" />
  <meta name="twitter:image" content="https://example.com/twitter-image.jpg" />
</head>
```

---

## 5. Structured Data (Schema.org)

### 🎯 Structured Data là gì?

Structured Data giúp search engines hiểu nội dung trang tốt hơn và hiển thị **Rich Snippets** trong kết quả tìm kiếm.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RICH SNIPPETS EXAMPLES                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ⭐⭐⭐⭐⭐ 4.8 (1,234 reviews)     ← Rating                            │
│  $299.00 - In Stock                 ← Product info                     │
│  📍 123 Main St, City               ← Local business                   │
│  🍳 30 mins | 4 servings            ← Recipe                           │
│  ❓ FAQ expandable answers          ← FAQ                              │
│  📅 Event: Jan 20, 2024             ← Event                            │
│  🔗 Sitelinks                       ← Navigation                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📋 JSON-LD Format (Recommended)

```html
<!-- Organization -->
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "Tên Công ty",
    "url": "https://example.com",
    "logo": "https://example.com/logo.png",
    "sameAs": ["https://facebook.com/company", "https://twitter.com/company", "https://linkedin.com/company/company"],
    "contactPoint": {
      "@type": "ContactPoint",
      "telephone": "+84-123-456-789",
      "contactType": "customer service",
      "availableLanguage": ["Vietnamese", "English"]
    }
  }
</script>

<!-- Website with Search -->
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "Tên Website",
    "url": "https://example.com",
    "potentialAction": {
      "@type": "SearchAction",
      "target": "https://example.com/search?q={search_term_string}",
      "query-input": "required name=search_term_string"
    }
  }
</script>
```

### 📝 Article Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Tiêu đề bài viết",
    "description": "Mô tả ngắn về bài viết",
    "image": ["https://example.com/image1.jpg", "https://example.com/image2.jpg"],
    "author": {
      "@type": "Person",
      "name": "Tên tác giả",
      "url": "https://example.com/author"
    },
    "publisher": {
      "@type": "Organization",
      "name": "Tên Publisher",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/logo.png"
      }
    },
    "datePublished": "2024-01-15T08:00:00+07:00",
    "dateModified": "2024-01-16T10:00:00+07:00",
    "mainEntityOfPage": {
      "@type": "WebPage",
      "@id": "https://example.com/article"
    }
  }
</script>
```

### 🛒 Product Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Tên sản phẩm",
    "description": "Mô tả sản phẩm chi tiết",
    "image": ["https://example.com/product1.jpg", "https://example.com/product2.jpg"],
    "brand": {
      "@type": "Brand",
      "name": "Tên thương hiệu"
    },
    "sku": "SKU123",
    "mpn": "MPN456",
    "offers": {
      "@type": "Offer",
      "url": "https://example.com/product",
      "priceCurrency": "VND",
      "price": "2990000",
      "priceValidUntil": "2024-12-31",
      "availability": "https://schema.org/InStock",
      "seller": {
        "@type": "Organization",
        "name": "Tên cửa hàng"
      }
    },
    "aggregateRating": {
      "@type": "AggregateRating",
      "ratingValue": "4.8",
      "reviewCount": "1234"
    },
    "review": [
      {
        "@type": "Review",
        "author": {
          "@type": "Person",
          "name": "Người review"
        },
        "datePublished": "2024-01-10",
        "reviewRating": {
          "@type": "Rating",
          "ratingValue": "5"
        },
        "reviewBody": "Sản phẩm rất tốt!"
      }
    ]
  }
</script>
```

### ❓ FAQ Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
      {
        "@type": "Question",
        "name": "Câu hỏi 1?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Câu trả lời cho câu hỏi 1."
        }
      },
      {
        "@type": "Question",
        "name": "Câu hỏi 2?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Câu trả lời cho câu hỏi 2."
        }
      }
    ]
  }
</script>
```

### 🍳 Recipe Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Recipe",
    "name": "Phở Bò Hà Nội",
    "description": "Công thức nấu phở bò truyền thống",
    "image": "https://example.com/pho.jpg",
    "author": {
      "@type": "Person",
      "name": "Đầu bếp A"
    },
    "datePublished": "2024-01-15",
    "prepTime": "PT30M",
    "cookTime": "PT3H",
    "totalTime": "PT3H30M",
    "recipeYield": "4 servings",
    "recipeCategory": "Main course",
    "recipeCuisine": "Vietnamese",
    "recipeIngredient": ["1kg xương bò", "500g thịt bò", "Bánh phở"],
    "recipeInstructions": [
      {
        "@type": "HowToStep",
        "text": "Ninh xương trong 3 tiếng"
      },
      {
        "@type": "HowToStep",
        "text": "Thái thịt bò mỏng"
      }
    ],
    "nutrition": {
      "@type": "NutritionInformation",
      "calories": "450 calories"
    },
    "aggregateRating": {
      "@type": "AggregateRating",
      "ratingValue": "4.9",
      "ratingCount": "500"
    }
  }
</script>
```

### 📍 Local Business Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "LocalBusiness",
    "name": "Tên cửa hàng",
    "description": "Mô tả cửa hàng",
    "image": "https://example.com/store.jpg",
    "@id": "https://example.com",
    "url": "https://example.com",
    "telephone": "+84-123-456-789",
    "priceRange": "$$",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "123 Đường ABC",
      "addressLocality": "Quận 1",
      "addressRegion": "TP.HCM",
      "postalCode": "70000",
      "addressCountry": "VN"
    },
    "geo": {
      "@type": "GeoCoordinates",
      "latitude": 10.7769,
      "longitude": 106.7009
    },
    "openingHoursSpecification": [
      {
        "@type": "OpeningHoursSpecification",
        "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
        "opens": "08:00",
        "closes": "22:00"
      },
      {
        "@type": "OpeningHoursSpecification",
        "dayOfWeek": ["Saturday", "Sunday"],
        "opens": "09:00",
        "closes": "21:00"
      }
    ]
  }
</script>
```

### 🍞 Breadcrumb Schema

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {
        "@type": "ListItem",
        "position": 1,
        "name": "Trang chủ",
        "item": "https://example.com"
      },
      {
        "@type": "ListItem",
        "position": 2,
        "name": "Danh mục",
        "item": "https://example.com/category"
      },
      {
        "@type": "ListItem",
        "position": 3,
        "name": "Sản phẩm",
        "item": "https://example.com/category/product"
      }
    ]
  }
</script>
```

---

## 6. Performance & Core Web Vitals

### 🎯 Core Web Vitals là gì?

Google sử dụng Core Web Vitals như ranking factors.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      CORE WEB VITALS                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   LCP (Largest Contentful Paint)                                       │
│   ════════════════════════════════                                     │
│   Thời gian load phần tử lớn nhất                                      │
│   ✅ Good: < 2.5s  ⚠️ Needs Improvement: 2.5-4s  ❌ Poor: > 4s         │
│                                                                         │
│   FID (First Input Delay) → INP (Interaction to Next Paint)           │
│   ═══════════════════════════════════════════════════════════          │
│   Thời gian phản hồi tương tác đầu tiên                                │
│   ✅ Good: < 100ms  ⚠️ Needs Improvement: 100-300ms  ❌ Poor: > 300ms  │
│                                                                         │
│   CLS (Cumulative Layout Shift)                                        │
│   ═════════════════════════════                                        │
│   Độ ổn định layout (không nhảy lung tung)                             │
│   ✅ Good: < 0.1  ⚠️ Needs Improvement: 0.1-0.25  ❌ Poor: > 0.25      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### ⚡ Tối ưu LCP

```html
<!-- 1. Preload critical resources -->
<link rel="preload" href="/hero-image.webp" as="image" />
<link rel="preload" href="/critical.css" as="style" />
<link rel="preload" href="/main-font.woff2" as="font" type="font/woff2" crossorigin />

<!-- 2. Inline critical CSS -->
<style>
  /* Critical CSS cho above-the-fold content */
  .hero {
    ...;
  }
  .header {
    ...;
  }
</style>

<!-- 3. Defer non-critical CSS -->
<link rel="preload" href="/non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'" />
<noscript><link rel="stylesheet" href="/non-critical.css" /></noscript>

<!-- 4. Optimize images -->
<img src="hero.webp" alt="Hero image" width="1200" height="600" fetchpriority="high" decoding="async" />
```

```javascript
// 5. Lazy load below-the-fold content
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll("img[data-src]").forEach((img) => {
  observer.observe(img);
});
```

### 🖱️ Tối ưu INP (Interaction to Next Paint)

```javascript
// 1. Break up long tasks
function processLargeArray(items) {
  const CHUNK_SIZE = 100;
  let index = 0;

  function processChunk() {
    const chunk = items.slice(index, index + CHUNK_SIZE);
    chunk.forEach((item) => processItem(item));
    index += CHUNK_SIZE;

    if (index < items.length) {
      // Yield to main thread
      setTimeout(processChunk, 0);
    }
  }

  processChunk();
}

// 2. Use requestIdleCallback for non-urgent work
requestIdleCallback(() => {
  // Analytics, prefetching, etc.
  sendAnalytics();
});

// 3. Debounce/throttle event handlers
function debounce(fn, delay) {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

const handleScroll = debounce(() => {
  // Handle scroll
}, 100);

// 4. Use passive event listeners
document.addEventListener("scroll", handleScroll, { passive: true });
```

### 📐 Tối ưu CLS

```html
<!-- 1. Always set dimensions for images/videos -->
<img src="image.jpg" width="800" height="600" alt="..." />

<video width="1280" height="720" poster="poster.jpg">
  <source src="video.mp4" type="video/mp4" />
</video>

<!-- 2. Reserve space for ads/embeds -->
<div class="ad-container" style="min-height: 250px;">
  <!-- Ad loads here -->
</div>

<!-- 3. Use aspect-ratio CSS -->
<style>
  .video-container {
    aspect-ratio: 16 / 9;
    width: 100%;
  }

  .image-placeholder {
    aspect-ratio: 4 / 3;
    background: #f0f0f0;
  }
</style>
```

```css
/* 4. Avoid inserting content above existing content */
.notification {
  position: fixed; /* Không đẩy content */
  top: 0;
}

/* 5. Use transform instead of changing layout properties */
.animate {
  /* ❌ Bad - causes layout shift */
  /* top: 100px; */

  /* ✅ Good - no layout shift */
  transform: translateY(100px);
}

/* 6. Font loading optimization */
@font-face {
  font-family: "CustomFont";
  src: url("font.woff2") format("woff2");
  font-display: swap; /* hoặc optional */
}
```

### 📊 Performance Budget

```javascript
// performance-budget.json
{
    "timings": {
        "firstContentfulPaint": 1500,
        "largestContentfulPaint": 2500,
        "timeToInteractive": 3500,
        "cumulativeLayoutShift": 0.1
    },
    "resourceSizes": {
        "total": 500,      // KB
        "javascript": 150,
        "css": 50,
        "images": 200,
        "fonts": 50
    },
    "resourceCounts": {
        "total": 50,
        "javascript": 10,
        "css": 3
    }
}
```

---

## 7. Images & Media Optimization

### 🖼️ Image Best Practices

```html
<!-- 1. Modern formats với fallback -->
<picture>
  <source srcset="image.avif" type="image/avif" />
  <source srcset="image.webp" type="image/webp" />
  <img src="image.jpg" alt="Mô tả hình ảnh" width="800" height="600" />
</picture>

<!-- 2. Responsive images -->
<img
  src="image-800.jpg"
  srcset="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  alt="Mô tả chi tiết"
  width="800"
  height="600"
  loading="lazy"
  decoding="async"
/>

<!-- 3. Art direction -->
<picture>
  <source media="(max-width: 600px)" srcset="mobile.webp" />
  <source media="(max-width: 1200px)" srcset="tablet.webp" />
  <img src="desktop.webp" alt="..." />
</picture>
```

### 📋 Image Attributes cho SEO

```html
<img
  src="product-laptop-dell-xps-15.webp"
  alt="Laptop Dell XPS 15 màu bạc, màn hình 15.6 inch, bàn phím backlit"
  title="Dell XPS 15 - Laptop cao cấp cho doanh nhân"
  width="800"
  height="600"
  loading="lazy"
  decoding="async"
  fetchpriority="low"
/>

<!-- Cho hero image (above the fold) -->
<img src="hero.webp" alt="..." fetchpriority="high" loading="eager" />
```

### 📊 Image Format Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SO SÁNH ĐỊNH DẠNG ẢNH                               │
├──────────┬──────────┬──────────┬──────────┬────────────────────────────┤
│  Format  │   Size   │ Quality  │ Support  │         Use Case           │
├──────────┼──────────┼──────────┼──────────┼────────────────────────────┤
│   AVIF   │  Nhỏ nhất│   Tốt    │   85%    │ Modern browsers            │
│   WebP   │   Nhỏ    │   Tốt    │   97%    │ Recommended default        │
│   JPEG   │  Trung   │   Tốt    │   100%   │ Photos, fallback           │
│   PNG    │   Lớn    │ Lossless │   100%   │ Transparency, graphics     │
│   SVG    │  Nhỏ     │  Vector  │   100%   │ Icons, logos               │
│   GIF    │   Lớn    │  256 màu │   100%   │ Simple animations          │
└──────────┴──────────┴──────────┴──────────┴────────────────────────────┘
```

### 🎬 Video Optimization

```html
<!-- Lazy load video -->
<video width="1280" height="720" poster="video-poster.webp" preload="none" controls>
  <source src="video.webm" type="video/webm" />
  <source src="video.mp4" type="video/mp4" />
  <track kind="captions" src="captions-vi.vtt" srclang="vi" label="Tiếng Việt" />
</video>

<!-- Video Schema -->
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "VideoObject",
    "name": "Tên video",
    "description": "Mô tả video",
    "thumbnailUrl": "https://example.com/thumbnail.jpg",
    "uploadDate": "2024-01-15",
    "duration": "PT5M30S",
    "contentUrl": "https://example.com/video.mp4",
    "embedUrl": "https://example.com/embed/video"
  }
</script>
```

---

## 8. URL Structure & Routing

### 📋 URL Best Practices

```
✅ GOOD URLs:
https://example.com/san-pham/laptop-dell-xps-15
https://example.com/blog/huong-dan-seo-2024
https://example.com/danh-muc/dien-thoai

❌ BAD URLs:
https://example.com/product?id=12345&cat=1
https://example.com/p/12345
https://example.com/blog/post/2024/01/15/this-is-a-very-long-url-that-nobody-wants
```

### 🔧 URL Guidelines

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      URL GUIDELINES                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Ngắn gọn, dễ đọc                                                    │
│  ✓ Chứa từ khóa chính                                                  │
│  ✓ Dùng dấu gạch ngang (-) thay vì gạch dưới (_)                      │
│  ✓ Chữ thường (lowercase)                                              │
│  ✓ Không có ký tự đặc biệt                                             │
│  ✓ Không có tham số không cần thiết                                    │
│  ✓ HTTPS                                                                │
│  ✓ Trailing slash nhất quán                                            │
│                                                                         │
│  ✗ Tránh: ID numbers, dates trong URL                                  │
│  ✗ Tránh: Stop words (the, and, or, a, an)                            │
│  ✗ Tránh: Quá nhiều subdirectories                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🔄 Canonical URLs

```html
<!-- Trang chính -->
<link rel="canonical" href="https://example.com/product" />

<!-- Tránh duplicate từ parameters -->
<!-- https://example.com/product?color=red -->
<!-- https://example.com/product?size=large -->
<!-- Cả 2 đều canonical về: -->
<link rel="canonical" href="https://example.com/product" />

<!-- Pagination -->
<!-- Page 1 -->
<link rel="canonical" href="https://example.com/blog" />

<!-- Page 2+ -->
<link rel="canonical" href="https://example.com/blog?page=2" />
<link rel="prev" href="https://example.com/blog" />
<link rel="next" href="https://example.com/blog?page=3" />
```

### 🔀 Redirects

```javascript
// Next.js - next.config.js
module.exports = {
  async redirects() {
    return [
      // 301 Permanent redirect
      {
        source: "/old-page",
        destination: "/new-page",
        permanent: true,
      },
      // 302 Temporary redirect
      {
        source: "/temp-page",
        destination: "/other-page",
        permanent: false,
      },
      // Redirect với wildcard
      {
        source: "/blog/:slug",
        destination: "/articles/:slug",
        permanent: true,
      },
    ];
  },
};
```

```nginx
# Nginx redirects
server {
    # 301 redirect
    rewrite ^/old-page$ /new-page permanent;

    # Redirect www to non-www
    if ($host = 'www.example.com') {
        return 301 https://example.com$request_uri;
    }

    # Redirect HTTP to HTTPS
    if ($scheme = 'http') {
        return 301 https://$host$request_uri;
    }
}
```

---

## 9. Mobile SEO

### 📱 Mobile-First Indexing

Google sử dụng mobile version để index và rank.

```html
<!-- Viewport meta (bắt buộc) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0" />

<!-- Không disable zoom -->
<!-- ❌ BAD -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />

<!-- ✅ GOOD -->
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

### 📋 Mobile SEO Checklist

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MOBILE SEO CHECKLIST                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  RESPONSIVE DESIGN                                                      │
│  □ Viewport meta tag                                                   │
│  □ Responsive images                                                   │
│  □ Flexible layouts (flexbox/grid)                                     │
│  □ Readable font sizes (min 16px)                                      │
│  □ Tap targets đủ lớn (min 48x48px)                                   │
│  □ Không horizontal scroll                                             │
│                                                                         │
│  PERFORMANCE                                                            │
│  □ Fast loading (< 3s on 3G)                                          │
│  □ Optimized images                                                    │
│  □ Minimal JavaScript                                                  │
│  □ Lazy loading                                                        │
│                                                                         │
│  CONTENT                                                                │
│  □ Same content as desktop                                             │
│  □ Không hide content trên mobile                                      │
│  □ Structured data giống desktop                                       │
│                                                                         │
│  UX                                                                     │
│  □ Không intrusive interstitials                                       │
│  □ Easy navigation                                                     │
│  □ Clickable phone numbers                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📞 Mobile-Specific Features

```html
<!-- Clickable phone number -->
<a href="tel:+84123456789">0123 456 789</a>

<!-- Clickable email -->
<a href="mailto:contact@example.com">contact@example.com</a>

<!-- Clickable address (opens maps) -->
<a href="https://maps.google.com/?q=123+Main+St">123 Main St, City</a>

<!-- SMS link -->
<a href="sms:+84123456789">Gửi SMS</a>
```

```css
/* Touch-friendly tap targets */
.button,
.link {
  min-height: 48px;
  min-width: 48px;
  padding: 12px 16px;
}

/* Readable text */
body {
  font-size: 16px;
  line-height: 1.5;
}

/* Avoid horizontal scroll */
* {
  max-width: 100%;
  box-sizing: border-box;
}

img,
video,
iframe {
  max-width: 100%;
  height: auto;
}
```

---

## 10. JavaScript SEO & Rendering

### 🎯 Rendering Strategies

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RENDERING STRATEGIES                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   SSR (Server-Side Rendering)                                          │
│   ═══════════════════════════                                          │
│   Server render HTML → Send to client → Hydrate                        │
│   ✅ SEO tốt nhất                                                      │
│   ✅ Fast FCP                                                          │
│   ❌ Server load cao                                                   │
│   Frameworks: Next.js, Nuxt.js, Remix                                  │
│                                                                         │
│   SSG (Static Site Generation)                                         │
│   ════════════════════════════                                         │
│   Build time render → Static HTML files                                │
│   ✅ SEO tốt                                                           │
│   ✅ Fast, CDN-friendly                                                │
│   ❌ Không dynamic content                                             │
│   Frameworks: Next.js, Gatsby, Astro                                   │
│                                                                         │
│   CSR (Client-Side Rendering)                                          │
│   ═══════════════════════════                                          │
│   Empty HTML → JS loads → Render on client                             │
│   ❌ SEO khó (Google có thể render nhưng chậm)                        │
│   ❌ Slow FCP                                                          │
│   ✅ Interactive                                                       │
│   Frameworks: React SPA, Vue SPA                                       │
│                                                                         │
│   ISR (Incremental Static Regeneration)                                │
│   ═══════════════════════════════════════                              │
│   SSG + Revalidate sau interval                                        │
│   ✅ SEO tốt                                                           │
│   ✅ Fresh content                                                     │
│   Framework: Next.js                                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Next.js SEO Example

```jsx
// pages/product/[slug].js
import Head from "next/head";

export default function ProductPage({ product }) {
  return (
    <>
      <Head>
        <title>{product.name} | Shop</title>
        <meta name="description" content={product.description} />
        <link rel="canonical" href={`https://example.com/product/${product.slug}`} />

        {/* Open Graph */}
        <meta property="og:title" content={product.name} />
        <meta property="og:description" content={product.description} />
        <meta property="og:image" content={product.image} />
        <meta property="og:type" content="product" />

        {/* Twitter */}
        <meta name="twitter:card" content="summary_large_image" />
      </Head>

      {/* Product Schema */}
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
              priceCurrency: "VND",
            },
          }),
        }}
      />

      <main>
        <h1>{product.name}</h1>
        {/* Product content */}
      </main>
    </>
  );
}

// SSR hoặc SSG
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.slug);

  return {
    props: { product },
    revalidate: 3600, // ISR: revalidate mỗi 1 giờ
  };
}

export async function getStaticPaths() {
  const products = await fetchAllProducts();

  return {
    paths: products.map((p) => ({ params: { slug: p.slug } })),
    fallback: "blocking",
  };
}
```

### 🔧 Dynamic Rendering

```javascript
// Detect Googlebot và serve pre-rendered content
// middleware.js (Next.js)
import { NextResponse } from "next/server";

export function middleware(request) {
  const userAgent = request.headers.get("user-agent") || "";

  const isBot =
    /googlebot|bingbot|yandex|baiduspider|facebookexternalhit|twitterbot|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator/i.test(
      userAgent
    );

  if (isBot) {
    // Redirect to pre-rendered version
    // hoặc serve cached HTML
  }

  return NextResponse.next();
}
```

---

## 11. Technical SEO

### 🤖 Robots.txt

```txt
# robots.txt
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /api/
Disallow: /private/
Disallow: /*?*sort=
Disallow: /*?*filter=

# Specific bots
User-agent: Googlebot
Allow: /

User-agent: Bingbot
Crawl-delay: 10

# Sitemap location
Sitemap: https://example.com/sitemap.xml
```

### 🗺️ XML Sitemap

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1"
        xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">

    <url>
        <loc>https://example.com/</loc>
        <lastmod>2024-01-15</lastmod>
        <changefreq>daily</changefreq>
        <priority>1.0</priority>
    </url>

    <url>
        <loc>https://example.com/products</loc>
        <lastmod>2024-01-14</lastmod>
        <changefreq>weekly</changefreq>
        <priority>0.8</priority>
        <image:image>
            <image:loc>https://example.com/images/products.jpg</image:loc>
            <image:title>Products</image:title>
        </image:image>
    </url>

    <url>
        <loc>https://example.com/blog/article</loc>
        <lastmod>2024-01-10</lastmod>
        <changefreq>monthly</changefreq>
        <priority>0.6</priority>
    </url>
</urlset>
```

### 📋 Sitemap Index (cho site lớn)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <sitemap>
        <loc>https://example.com/sitemap-pages.xml</loc>
        <lastmod>2024-01-15</lastmod>
    </sitemap>
    <sitemap>
        <loc>https://example.com/sitemap-products.xml</loc>
        <lastmod>2024-01-14</lastmod>
    </sitemap>
    <sitemap>
        <loc>https://example.com/sitemap-blog.xml</loc>
        <lastmod>2024-01-10</lastmod>
    </sitemap>
</sitemapindex>
```

### 🔒 Security Headers

```nginx
# Nginx security headers
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self';" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

### 📄 HTTP Status Codes cho SEO

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HTTP STATUS CODES                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  200 OK              Trang hoạt động bình thường                       │
│  301 Moved           Redirect vĩnh viễn (chuyển SEO juice)             │
│  302 Found           Redirect tạm thời (không chuyển SEO)              │
│  304 Not Modified    Cache hit                                         │
│  404 Not Found       Trang không tồn tại                               │
│  410 Gone            Trang đã bị xóa vĩnh viễn                         │
│  500 Server Error    Lỗi server                                        │
│  503 Unavailable     Server tạm thời không khả dụng                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 12. International SEO

### 🌍 Hreflang Tags

```html
<head>
  <!-- Trang tiếng Việt -->
  <link rel="alternate" hreflang="vi" href="https://example.com/vi/page" />

  <!-- Trang tiếng Anh -->
  <link rel="alternate" hreflang="en" href="https://example.com/en/page" />

  <!-- Trang tiếng Anh cho US -->
  <link rel="alternate" hreflang="en-US" href="https://example.com/en-us/page" />

  <!-- Trang tiếng Anh cho UK -->
  <link rel="alternate" hreflang="en-GB" href="https://example.com/en-gb/page" />

  <!-- Default fallback -->
  <link rel="alternate" hreflang="x-default" href="https://example.com/page" />

  <!-- Canonical cho trang hiện tại -->
  <link rel="canonical" href="https://example.com/vi/page" />
</head>
```

### 📋 URL Structures cho Multi-language

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    URL STRUCTURES                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. Subdirectories (Recommended)                                       │
│     example.com/vi/                                                    │
│     example.com/en/                                                    │
│     ✅ Dễ setup, share domain authority                                │
│                                                                         │
│  2. Subdomains                                                         │
│     vi.example.com                                                     │
│     en.example.com                                                     │
│     ⚠️ Được coi như separate sites                                    │
│                                                                         │
│  3. ccTLDs (Country Code Top-Level Domains)                           │
│     example.vn                                                         │
│     example.co.uk                                                      │
│     ✅ Strong geo-targeting signal                                     │
│     ❌ Expensive, separate SEO efforts                                 │
│                                                                         │
│  4. Parameters (Not Recommended)                                       │
│     example.com?lang=vi                                                │
│     ❌ Hard to manage, poor SEO                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 🗺️ Sitemap cho Multi-language

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">

    <url>
        <loc>https://example.com/vi/page</loc>
        <xhtml:link rel="alternate" hreflang="vi" href="https://example.com/vi/page"/>
        <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/page"/>
        <xhtml:link rel="alternate" hreflang="x-default" href="https://example.com/page"/>
    </url>

    <url>
        <loc>https://example.com/en/page</loc>
        <xhtml:link rel="alternate" hreflang="vi" href="https://example.com/vi/page"/>
        <xhtml:link rel="alternate" hreflang="en" href="https://example.com/en/page"/>
        <xhtml:link rel="alternate" hreflang="x-default" href="https://example.com/page"/>
    </url>

</urlset>
```

---

## 13. SEO Tools & Testing

### 🔧 Essential Tools

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      SEO TOOLS                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  GOOGLE TOOLS (Free)                                                   │
│  ├── Google Search Console    Indexing, performance, errors            │
│  ├── Google Analytics         Traffic, user behavior                   │
│  ├── PageSpeed Insights       Core Web Vitals, performance             │
│  ├── Mobile-Friendly Test     Mobile compatibility                     │
│  ├── Rich Results Test        Structured data validation               │
│  └── Lighthouse               Comprehensive audit                      │
│                                                                         │
│  THIRD-PARTY TOOLS                                                     │
│  ├── Ahrefs                   Backlinks, keywords, competitors         │
│  ├── SEMrush                  All-in-one SEO suite                     │
│  ├── Moz                      Domain authority, link analysis          │
│  ├── Screaming Frog           Technical SEO crawler                    │
│  └── GTmetrix                 Performance testing                      │
│                                                                         │
│  BROWSER EXTENSIONS                                                    │
│  ├── SEO Meta in 1 Click      Quick meta tag check                     │
│  ├── Lighthouse               Built into Chrome DevTools               │
│  ├── Web Vitals               Core Web Vitals monitoring               │
│  └── Wappalyzer               Technology detection                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 💻 Lighthouse Audit

```bash
# CLI
npm install -g lighthouse
lighthouse https://example.com --output html --output-path ./report.html

# Programmatic
npx lighthouse https://example.com --output=json --chrome-flags="--headless"
```

### 📊 Testing Checklist

```javascript
// SEO Audit Script
const seoChecklist = {
  // Meta Tags
  title: document.title.length >= 30 && document.title.length <= 60,
  description: document.querySelector('meta[name="description"]')?.content.length <= 160,
  canonical: !!document.querySelector('link[rel="canonical"]'),

  // Open Graph
  ogTitle: !!document.querySelector('meta[property="og:title"]'),
  ogDescription: !!document.querySelector('meta[property="og:description"]'),
  ogImage: !!document.querySelector('meta[property="og:image"]'),

  // Structure
  h1Count: document.querySelectorAll("h1").length === 1,
  imgAlt: [...document.querySelectorAll("img")].every((img) => img.alt),

  // Performance
  lazyImages: [...document.querySelectorAll("img")].some((img) => img.loading === "lazy"),

  // Mobile
  viewport: !!document.querySelector('meta[name="viewport"]'),

  // Security
  https: location.protocol === "https:",
};

console.table(seoChecklist);
```

### 📋 Complete SEO Checklist

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SEO CHECKLIST CHO FRONTEND                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  TECHNICAL                                                              │
│  □ HTTPS enabled                                                       │
│  □ Mobile responsive                                                   │
│  □ Fast loading (< 3s)                                                 │
│  □ Core Web Vitals pass                                                │
│  □ XML sitemap                                                         │
│  □ Robots.txt                                                          │
│  □ Canonical URLs                                                      │
│  □ No broken links (404)                                               │
│  □ Proper redirects (301)                                              │
│                                                                         │
│  ON-PAGE                                                                │
│  □ Unique title (50-60 chars)                                          │
│  □ Meta description (150-160 chars)                                    │
│  □ One H1 per page                                                     │
│  □ Proper heading hierarchy                                            │
│  □ Semantic HTML                                                       │
│  □ Alt text for images                                                 │
│  □ Internal linking                                                    │
│  □ Keyword in URL                                                      │
│                                                                         │
│  STRUCTURED DATA                                                        │
│  □ Organization schema                                                 │
│  □ Breadcrumb schema                                                   │
│  □ Article/Product schema (nếu applicable)                             │
│  □ FAQ schema (nếu có FAQ)                                             │
│                                                                         │
│  SOCIAL                                                                 │
│  □ Open Graph tags                                                     │
│  □ Twitter Card tags                                                   │
│  □ Social share images (1200x630)                                      │
│                                                                         │
│  PERFORMANCE                                                            │
│  □ Image optimization (WebP/AVIF)                                      │
│  □ Lazy loading                                                        │
│  □ Code splitting                                                      │
│  □ Minification                                                        │
│  □ Caching headers                                                     │
│  □ CDN                                                                 │
│                                                                         │
│  ACCESSIBILITY                                                          │
│  □ ARIA labels                                                         │
│  □ Keyboard navigation                                                 │
│  □ Color contrast                                                      │
│  □ Focus indicators                                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Quick Reference

### Meta Tags Template

```html
<!DOCTYPE html>
<html lang="vi">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <!-- SEO -->
    <title>Từ khóa chính - Mô tả ngắn | Brand</title>
    <meta name="description" content="Mô tả 150-160 ký tự với từ khóa" />
    <meta name="robots" content="index, follow" />
    <link rel="canonical" href="https://example.com/page" />

    <!-- Open Graph -->
    <meta property="og:type" content="website" />
    <meta property="og:url" content="https://example.com/page" />
    <meta property="og:title" content="Tiêu đề" />
    <meta property="og:description" content="Mô tả" />
    <meta property="og:image" content="https://example.com/og-image.jpg" />
    <meta property="og:locale" content="vi_VN" />

    <!-- Twitter -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content="Tiêu đề" />
    <meta name="twitter:description" content="Mô tả" />
    <meta name="twitter:image" content="https://example.com/twitter-image.jpg" />

    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" />
    <link rel="apple-touch-icon" href="/apple-touch-icon.png" />

    <!-- Preconnect -->
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="dns-prefetch" href="https://analytics.google.com" />
  </head>
  <body>
    <!-- Content -->

    <!-- Structured Data -->
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "WebPage",
        "name": "Tên trang",
        "description": "Mô tả"
      }
    </script>
  </body>
</html>
```

---

## 🔗 Tài liệu tham khảo

- [Google Search Central](https://developers.google.com/search)
- [Schema.org](https://schema.org/)
- [Web.dev - SEO](https://web.dev/learn/seo/)
- [Moz - Beginner's Guide to SEO](https://moz.com/beginners-guide-to-seo)
- [Ahrefs Blog](https://ahrefs.com/blog/)
- [Google PageSpeed Insights](https://pagespeed.web.dev/)
- [Rich Results Test](https://search.google.com/test/rich-results)
