# HTML Semantic Elements - Tổng hợp kiến thức

## 📌 1. Semantic Elements in HTML là gì?

### Định nghĩa
**Semantic Elements** (Phần tử ngữ nghĩa) là các thẻ HTML có tên mô tả rõ ràng ý nghĩa và nội dung của chúng cho cả developer và browser.

### So sánh Non-semantic vs Semantic

```html
<!-- ❌ Non-semantic: không rõ ý nghĩa -->
<div id="header">
  <div id="nav">...</div>
</div>
<div class="article">...</div>

<!-- ✅ Semantic: rõ ràng, có ý nghĩa -->
<header>
  <nav>...</nav>
</header>
<article>...</article>
```

### Các Semantic Elements phổ biến
- `<header>` - Phần đầu của trang/section
- `<nav>` - Navigation links
- `<main>` - Nội dung chính
- `<section>` - Phần của document
- `<article>` - Nội dung độc lập
- `<aside>` - Nội dung phụ (sidebar)
- `<footer>` - Phần chân trang
- `<figure>` & `<figcaption>` - Hình ảnh với chú thích

---

## 📌 2. HTML `<section>` Element

### Định nghĩa
`<section>` dùng để nhóm các nội dung có **cùng chủ đề** hoặc liên quan đến nhau.

### Khi nào dùng?
- Chia trang thành các phần logic
- Mỗi section thường có heading (`<h1>` - `<h6>`)
- Nhóm nội dung theo chủ đề

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<body>
  <!-- Section 1: Giới thiệu -->
  <section>
    <h2>Giới thiệu về công ty</h2>
    <p>Chúng tôi là công ty công nghệ hàng đầu...</p>
  </section>

  <!-- Section 2: Dịch vụ -->
  <section>
    <h2>Dịch vụ của chúng tôi</h2>
    <p>Web development, Mobile app, Cloud computing...</p>
  </section>

  <!-- Section 3: Liên hệ -->
  <section>
    <h2>Liên hệ</h2>
    <p>Email: contact@company.com</p>
  </section>
</body>
</html>
```

### ⚠️ Lưu ý
- `<section>` khác với `<div>`: section có ý nghĩa về mặt cấu trúc
- Nên có heading bên trong mỗi section
- Không dùng `<section>` chỉ để styling → dùng `<div>`

---

## 📌 3. HTML `<article>` Element

### Định nghĩa
`<article>` đại diện cho **nội dung độc lập**, có thể tái sử dụng hoặc phân phối riêng biệt.

### Khi nào dùng?
- Blog posts
- Tin tức
- Bình luận của user
- Sản phẩm
- Forum posts

### Ví dụ thực tế

```html
<!-- Blog post -->
<article>
  <header>
    <h2>Học HTML Semantic Elements trong 30 phút</h2>
    <p>Đăng bởi: <strong>John Doe</strong> | 29/10/2025</p>
  </header>
  
  <section>
    <h3>Giới thiệu</h3>
    <p>Semantic HTML giúp code dễ đọc và SEO tốt hơn...</p>
  </section>
  
  <section>
    <h3>Các thẻ phổ biến</h3>
    <p>Header, nav, main, article, section...</p>
  </section>
  
  <footer>
    <p>Tags: HTML, Web Development, Frontend</p>
  </footer>
</article>

<!-- Danh sách bài viết -->
<section>
  <h2>Bài viết mới nhất</h2>
  
  <article>
    <h3>Bài viết 1: React Hooks</h3>
    <p>Học React Hooks từ cơ bản đến nâng cao...</p>
  </article>
  
  <article>
    <h3>Bài viết 2: CSS Grid</h3>
    <p>Hướng dẫn sử dụng CSS Grid Layout...</p>
  </article>
</section>
```

### ⚠️ Đặc điểm của `<article>`
- Phải có thể đứng độc lập (standalone)
- Có thể lấy ra và đặt ở nơi khác vẫn có nghĩa
- Thường có heading riêng
- Có thể có `<header>`, `<footer>` riêng

---

## 📌 4. Nesting `<article>` in `<section>` hay ngược lại?

### ✅ Cả hai đều hợp lệ, tùy ngữ cảnh!

### Case 1: `<section>` chứa nhiều `<article>`

```html
<!-- Trang blog với nhiều bài viết -->
<section>
  <h2>Bài viết về JavaScript</h2>
  
  <article>
    <h3>Closures trong JavaScript</h3>
    <p>Closures là một khái niệm quan trọng...</p>
  </article>
  
  <article>
    <h3>Promises và Async/Await</h3>
    <p>Xử lý bất đồng bộ trong JS...</p>
  </article>
</section>
```

**Khi nào dùng?**
- Nhiều bài viết cùng chủ đề
- Danh sách sản phẩm
- Tin tức theo category

### Case 2: `<article>` chứa nhiều `<section>`

```html
<!-- Một bài viết dài có nhiều phần -->
<article>
  <h1>Hướng dẫn toàn diện về React</h1>
  
  <section>
    <h2>1. Components</h2>
    <p>React được xây dựng từ components...</p>
  </section>
  
  <section>
    <h2>2. Props và State</h2>
    <p>Props truyền dữ liệu từ parent xuống child...</p>
  </section>
  
  <section>
    <h2>3. Lifecycle Methods</h2>
    <p>Mỗi component có các lifecycle methods...</p>
  </section>
</article>
```

**Khi nào dùng?**
- Bài viết dài có nhiều chương/phần
- Tutorial có nhiều bước
- Documentation có nhiều sections

### 🎯 Quy tắc vàng
| Cấu trúc | Khi nào dùng |
|----------|--------------|
| `<section>` → `<article>` | Nhóm nhiều nội dung độc lập cùng chủ đề |
| `<article>` → `<section>` | Một nội dung độc lập có nhiều phần |

---

## 📌 5. HTML `<header>` Element

### Định nghĩa
`<header>` chứa nội dung giới thiệu hoặc navigation cho phần tử cha của nó.

### Vị trí sử dụng
- Header của toàn trang (trong `<body>`)
- Header của `<article>`
- Header của `<section>`

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<body>
  <!-- Header của toàn trang -->
  <header>
    <img src="logo.png" alt="Company Logo" style="height: 50px;">
    <h1>TechBlog.com</h1>
    <nav>
      <a href="/">Home</a> |
      <a href="/blog">Blog</a> |
      <a href="/about">About</a> |
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main>
    <!-- Header của article -->
    <article>
      <header>
        <h2>Top 10 JavaScript Frameworks 2025</h2>
        <p>By <strong>Jane Smith</strong> | Published: Oct 29, 2025</p>
        <p>Reading time: 5 minutes</p>
      </header>
      
      <p>JavaScript frameworks are evolving rapidly...</p>
      
      <footer>
        <p>Tags: JavaScript, Frameworks, Web Dev</p>
      </footer>
    </article>
  </main>

  <footer>
    <p>&copy; 2025 TechBlog. All rights reserved.</p>
  </footer>
</body>
</html>
```

### ⚠️ Lưu ý
- Có thể có **nhiều `<header>`** trong một trang
- `<header>` không thể nằm trong `<footer>`, `<address>`, hoặc `<header>` khác
- Thường chứa: logo, tiêu đề, navigation, search box

---

## 📌 6. HTML `<footer>` Element

### Định nghĩa
`<footer>` chứa thông tin về phần tử cha của nó (thường là thông tin cuối trang).

### Nội dung thường có trong `<footer>`
- Copyright information
- Contact info
- Sitemap
- Social media links
- Back to top links

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<body>
  <article>
    <h2>Understanding CSS Flexbox</h2>
    <p>Flexbox is a powerful layout module...</p>
    
    <!-- Footer của article -->
    <footer>
      <p>Author: <strong>John Developer</strong></p>
      <p>Last updated: October 29, 2025</p>
      <p>Categories: CSS, Layout, Frontend</p>
    </footer>
  </article>

  <!-- Footer của toàn trang -->
  <footer style="background: #333; color: white; padding: 20px;">
    <section>
      <h3>About Us</h3>
      <p>We are a leading tech education platform</p>
    </section>
    
    <section>
      <h3>Quick Links</h3>
      <nav>
        <a href="/privacy">Privacy Policy</a> |
        <a href="/terms">Terms of Service</a> |
        <a href="/sitemap">Sitemap</a>
      </nav>
    </section>
    
    <section>
      <h3>Contact</h3>
      <address>
        Email: <a href="mailto:info@techblog.com">info@techblog.com</a><br>
        Phone: +84 123 456 789
      </address>
    </section>
    
    <p>&copy; 2025 TechBlog. All rights reserved.</p>
  </footer>
</body>
</html>
```

### ⚠️ Lưu ý
- Có thể có nhiều `<footer>` trong một trang
- `<footer>` không thể chứa `<header>` hoặc `<footer>` khác
- Thường ở cuối nhưng không bắt buộc

---

## 📌 7. HTML `<nav>` Element

### Định nghĩa
`<nav>` dành cho các **navigation links chính** của trang.

### Khi nào dùng?
✅ **NÊN dùng:**
- Main navigation menu
- Table of contents
- Breadcrumbs
- Pagination

❌ **KHÔNG NÊN dùng:**
- Footer links (trừ khi là navigation chính)
- Social media links
- Các links nhỏ lẻ

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<body>
  <!-- Navigation chính -->
  <header>
    <h1>My Website</h1>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/services">Services</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <!-- Breadcrumb navigation -->
  <nav aria-label="Breadcrumb">
    <a href="/">Home</a> &gt;
    <a href="/products">Products</a> &gt;
    <span>Laptops</span>
  </nav>

  <main>
    <article>
      <h2>Long Article with Table of Contents</h2>
      
      <!-- Table of Contents -->
      <nav aria-label="Table of Contents">
        <h3>Contents</h3>
        <ul>
          <li><a href="#intro">Introduction</a></li>
          <li><a href="#features">Features</a></li>
          <li><a href="#conclusion">Conclusion</a></li>
        </ul>
      </nav>
      
      <section id="intro">
        <h3>Introduction</h3>
        <p>...</p>
      </section>
    </article>
  </main>

  <!-- Pagination -->
  <nav aria-label="Pagination">
    <a href="/page/1">Previous</a>
    <a href="/page/2">2</a>
    <a href="/page/3">3</a>
    <a href="/page/4">Next</a>
  </nav>
</body>
</html>
```

### ⚠️ Best Practices
- Dùng `aria-label` để mô tả loại navigation
- Có thể có nhiều `<nav>` trong trang
- Thường dùng với `<ul>` & `<li>` hoặc links đơn giản

---

## 📌 8. HTML `<aside>` Element

### Định nghĩa
`<aside>` chứa nội dung **liên quan gián tiếp** đến nội dung chính (sidebar content).

### Khi nào dùng?
- Sidebar
- Related articles
- Advertisements
- Author bio
- Glossary
- Pull quotes

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<head>
<style>
  body {
    display: flex;
    gap: 20px;
  }
  main {
    flex: 3;
  }
  aside {
    flex: 1;
    background: #f4f4f4;
    padding: 15px;
  }
</style>
</head>
<body>
  <!-- Nội dung chính -->
  <main>
    <article>
      <h1>React vs Vue: Which Framework to Choose?</h1>
      <p>Choosing between React and Vue can be challenging...</p>
      
      <!-- Aside trong article -->
      <aside style="background: #fff3cd; padding: 10px; margin: 20px 0;">
        <h4>💡 Quick Tip</h4>
        <p>If you're coming from jQuery, Vue might feel more intuitive.</p>
      </aside>
      
      <p>React has a larger ecosystem...</p>
    </article>
  </main>

  <!-- Sidebar -->
  <aside>
    <section>
      <h3>Popular Posts</h3>
      <ul>
        <li><a href="#">10 JavaScript Tricks</a></li>
        <li><a href="#">CSS Grid Guide</a></li>
        <li><a href="#">Node.js Best Practices</a></li>
      </ul>
    </section>
    
    <section>
      <h3>Categories</h3>
      <ul>
        <li>JavaScript (45)</li>
        <li>CSS (32)</li>
        <li>React (28)</li>
      </ul>
    </section>
    
    <section>
      <h3>Newsletter</h3>
      <form>
        <input type="email" placeholder="Your email">
        <button>Subscribe</button>
      </form>
    </section>
  </aside>
</body>
</html>
```

### Use Cases cụ thể

```html
<!-- Author bio trong blog post -->
<article>
  <h1>10 Tips for Better Code</h1>
  <p>Writing clean code is essential...</p>
  
  <aside>
    <h3>About the Author</h3>
    <img src="author.jpg" alt="John Doe">
    <p><strong>John Doe</strong> is a senior developer with 10 years experience...</p>
  </aside>
</article>

<!-- Advertisement -->
<aside aria-label="Advertisement">
  <img src="ad-banner.jpg" alt="Special Offer">
</aside>

<!-- Glossary -->
<article>
  <p>Understanding <dfn>closure</dfn> is important...</p>
  
  <aside>
    <h4>Glossary</h4>
    <dl>
      <dt>Closure</dt>
      <dd>A function that has access to variables from outer scope</dd>
    </dl>
  </aside>
</article>
```

---

## 📌 9. HTML `<figure>` và `<figcaption>` Elements

### Định nghĩa
- `<figure>`: Nhóm nội dung độc lập (hình ảnh, code, diagram...)
- `<figcaption>`: Chú thích cho `<figure>`

### Khi nào dùng?
- Images với caption
- Code snippets
- Diagrams
- Illustrations
- Videos
- Audio

### Ví dụ thực tế

```html
<!DOCTYPE html>
<html>
<body>
  <article>
    <h2>The Future of Web Development</h2>
    <p>Web technologies are evolving rapidly...</p>
    
    <!-- Hình ảnh với caption -->
    <figure>
      <img src="web-trends-2025.jpg" 
           alt="Web development trends chart"
           width="600">
      <figcaption>
        Figure 1: Top web development trends in 2025 
        (Source: Stack Overflow Survey)
      </figcaption>
    </figure>
    
    <p>As shown in Figure 1, React continues to dominate...</p>
    
    <!-- Code snippet với caption -->
    <figure>
      <pre><code>
function greet(name) {
  return `Hello, ${name}!`;
}
console.log(greet('World'));
      </code></pre>
      <figcaption>
        Code Example 1: Simple greeting function in JavaScript
      </figcaption>
    </figure>
    
    <!-- Multiple images -->
    <figure>
      <img src="screenshot1.jpg" alt="Step 1" width="300">
      <img src="screenshot2.jpg" alt="Step 2" width="300">
      <img src="screenshot3.jpg" alt="Step 3" width="300">
      <figcaption>
        Figure 2: Three-step installation process
      </figcaption>
    </figure>
    
    <!-- Poem/Quote -->
    <figure>
      <blockquote>
        <p>"Any fool can write code that a computer can understand. 
           Good programmers write code that humans can understand."</p>
      </blockquote>
      <figcaption>
        — Martin Fowler, <cite>Refactoring</cite>
      </figcaption>
    </figure>
    
    <!-- Diagram -->
    <figure>
      <svg width="200" height="100">
        <rect width="200" height="100" style="fill:lightblue"/>
        <text x="100" y="50" text-anchor="middle">Diagram</text>
      </svg>
      <figcaption>Figure 3: System Architecture Diagram</figcaption>
    </figure>
  </article>
</body>
</html>
```

### ⚠️ Best Practices

```html
<!-- ✅ ĐÚNG: Figure có thể di chuyển mà không ảnh hưởng nội dung -->
<p>The data shows significant growth...</p>
<figure>
  <img src="chart.png" alt="Growth chart">
  <figcaption>Annual growth 2020-2025</figcaption>
</figure>
<p>This trend is expected to continue...</p>

<!-- ❌ SAI: Image là phần không thể thiếu của văn bản -->
<p>Click the <img src="button.png" alt="submit button"> to continue</p>
<!-- Không nên dùng <figure> vì image là inline content -->
```

### Styling Figure & Figcaption

```html
<style>
figure {
  margin: 2em 0;
  padding: 1em;
  background: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  text-align: center;
}

figure img {
  max-width: 100%;
  height: auto;
}

figcaption {
  margin-top: 0.5em;
  font-style: italic;
  color: #666;
  font-size: 0.9em;
}
</style>

<figure>
  <img src="laptop.jpg" alt="Modern laptop">
  <figcaption>Latest MacBook Pro 2025</figcaption>
</figure>
```

---

## 📌 10. Why Semantic Elements? (Tại sao dùng Semantic HTML?)

### 🎯 Lợi ích chính

### 1. **SEO (Search Engine Optimization)**

```html
<!-- ❌ BAD: Search engine khó hiểu cấu trúc -->
<div class="header">
  <div class="nav">
    <div class="link">Home</div>
  </div>
</div>
<div class="content">
  <div class="post">...</div>
</div>

<!-- ✅ GOOD: Search engine hiểu rõ cấu trúc -->
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <article>...</article>
</main>
```

**Kết quả:**
- Google dễ dàng index nội dung
- Rich snippets hiển thị đẹp hơn
- Ranking cao hơn trong kết quả tìm kiếm

### 2. **Accessibility (Khả năng tiếp cận)**

```html
<!-- Screen reader có thể nhảy giữa các sections -->
<nav aria-label="Main navigation">
  <ul>
    <li><a href="#main">Skip to content</a></li>
  </ul>
</nav>

<main id="main">
  <article>
    <header>
      <h1>Article Title</h1>
    </header>
    <p>Content...</p>
  </article>
</main>
```

**Lợi ích cho người khuyết tật:**
- Screen readers đọc đúng cấu trúc
- Keyboard navigation dễ dàng hơn
- Voice control hiểu được landmarks

### 3. **Code Maintainability (Dễ bảo trì)**

```html
<!-- ❌ SAI: Sau 6 tháng bạn quên div nào là gì -->
<div class="top">
  <div class="menu">
    <div class="item">...</div>
  </div>
</div>
<div class="mid">
  <div class="box">
    <div class="title">...</div>
    <div class="text">...</div>
  </div>
</div>

<!-- ✅ ĐÚNG: Rõ ràng, dễ hiểu ngay -->
<header>
  <nav>
    <a href="#">...</a>
  </nav>
</header>
<main>
  <article>
    <h2>...</h2>
    <p>...</p>
  </article>
</main>
```

### 4. **Consistency (Nhất quán)**

```html
<!-- Team member mới hiểu ngay cấu trúc -->
<body>
  <header><!-- Always header at top --></header>
  <nav><!-- Always navigation --></nav>
  <main>
    <article><!-- Always article content --></article>
    <aside><!-- Always sidebar --></aside>
  </main>
  <footer><!-- Always footer at bottom --></footer>
</body>
```

### 5. **Mobile & Responsive**

```css
/* Dễ dàng responsive với semantic HTML */
@media (max-width: 768px) {
  nav {
    flex-direction: column;
  }
  
  aside {
    display: none; /* Hide sidebar on mobile */
  }
  
  article {
    width: 100%;
  }
}
```

### 6. **Browser & Device Compatibility**

```html
<!-- Browser và assistive technology hiểu semantic elements -->
<main>
  <article>
    <!-- Browser biết đây là content chính -->
    <!-- Reader mode (Safari, Firefox) tự động detect -->
  </article>
</main>
```

### 📊 So sánh thực tế

| Tiêu chí | Non-Semantic (`<div>`) | Semantic (`<article>`) |
|----------|----------------------|----------------------|
| **SEO Score** | 60/100 | 90/100 |
| **Accessibility** | 50/100 | 95/100 |
| **Code readability** | 40/100 | 95/100 |
| **Maintainability** | 50/100 | 90/100 |
| **File size** | Same | Same |
| **Performance** | Same | Same |

### 🎓 Best Practices

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Semantic HTML Example</title>
</head>
<body>
  <!-- ✅ Cấu trúc semantic đầy đủ -->
  <header>
    <h1>Website Title</h1>
    <nav aria-label="Main">
      <a href="/">Home</a>
      <a href="/blog">Blog</a>
    </nav>
  </header>

  <main>
    <article>
      <header>
        <h2>Article Title</h2>
        <time datetime="2025-10-29">October 29, 2025</time>
      </header>
      
      <section>
        <h3>Introduction</h3>
        <p>Content here...</p>
      </section>
      
      <figure>
        <img src="image.jpg" alt="Description">
        <figcaption>Image caption</figcaption>
      </figure>
      
      <footer>
        <p>Tags: HTML, Web Dev</p>
      </footer>
    </article>
    
    <aside>
      <h3>Related Posts</h3>
      <nav aria-label="Related">
        <a href="#">Post 1</a>
      </nav>
    </aside>
  </main>

  <footer>
    <p>&copy; 2025 Company Name</p>
  </footer>
</body>
</html>
```

---

## ❓ Câu hỏi phỏng vấn thường gặp

### Q1: Sự khác biệt giữa `<section>` và `<div>`?
**A:** 
- `<section>`: Có ý nghĩa semantic, nhóm nội dung theo chủ đề, nên có heading
- `<div>`: Không có ý nghĩa, chỉ dùng để styling hoặc grouping

### Q2: Khi nào dùng `<article>` thay vì `<section>`?
**A:** Dùng `<article>` khi nội dung:
- Có thể đứng độc lập
- Có thể tái sử dụng
- Có thể syndicate (RSS feed)
- VD: blog post, news article, product card

### Q3: Có thể có nhiều `<header>` và `<footer>` trong một trang không?
**A:** **CÓ**. Mỗi `<article>`, `<section>` có thể có `<header>` và `<footer>` riêng.

### Q4: `<nav>` có bắt buộc cho tất cả links không?
**A:** **KHÔNG**. Chỉ dùng cho **navigation chính**. Footer links, social links không cần `<nav>`.

### Q5: Sự khác biệt giữa `<aside>` và `<section>`?
**A:**
- `<aside>`: Nội dung phụ, liên quan gián tiếp (sidebar, related posts)
- `<section>`: Nội dung chính, phần của document

### Q6: Tại sao nên dùng semantic HTML?
**A:** 
1. SEO tốt hơn
2. Accessibility tốt hơn
3. Code dễ đọc, dễ maintain
4. Browser hiểu cấu trúc
5. Consistent across team

### Q7: `<figure>` chỉ dùng cho images thôi à?
**A:** **KHÔNG**. Có thể dùng cho:
- Images
- Code blocks
- Diagrams
- Videos
- Quotes
- Tables

---

## ✅ Checklist học tập

- [ ] Hiểu được semantic elements là gì
- [ ] Phân biệt được `<section>` vs `<article>` vs `<div>`
- [ ] Biết khi nào nest `<section>` trong `<article>` và ngược lại
- [ ] Sử dụng đúng `<header>`, `<footer>`, `<nav>`
- [ ] Biết cách dùng `<aside>` cho sidebar
- [ ] Sử dụng `<figure>` và `<figcaption>` đúng cách
- [ ] Giải thích được lợi ích của semantic HTML
- [ ] Làm được bài tập xây dựng trang blog với semantic HTML
- [ ] Trả lời được các câu hỏi phỏng vấn

---

## 🔗 Resources

- [MDN: Semantic HTML](https://developer.mozilla.org/en