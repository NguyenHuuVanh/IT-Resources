# CSS Position - Hướng dẫn chi tiết về Static, Relative, Absolute

## Mục lục

1. [CSS Position là gì?](#1-css-position-là-gì)
2. [Position: Static (Mặc định)](#2-position-static-mặc-định)
3. [Position: Relative (Tương đối)](#3-position-relative-tương-đối)
4. [Position: Absolute (Tuyệt đối)](#4-position-absolute-tuyệt-đối)
5. [So sánh Static vs Relative vs Absolute](#5-so-sánh-static-vs-relative-vs-absolute)
6. [Position: Fixed và Sticky (Bonus)](#6-position-fixed-và-sticky-bonus)
7. [Ví dụ thực tế](#7-ví-dụ-thực-tế)

## 1. CSS Position là gì?

### 1.1. Định nghĩa

`position` là thuộc tính CSS xác định **cách một element được định vị** trên trang web.

### 1.2. Các giá trị của position

```css
position: static;      /* Mặc định */
position: relative;    /* Tương đối */
position: absolute;    /* Tuyệt đối */
position: fixed;       /* Cố định */
position: sticky;      /* Dính */
```

### 1.3. Các thuộc tính đi kèm

Khi dùng position (trừ `static`), bạn có thể sử dụng:

```css
top: 10px;      /* Cách từ trên xuống */
right: 20px;    /* Cách từ phải sang */
bottom: 30px;   /* Cách từ dưới lên */
left: 40px;     /* Cách từ trái sang */
```

**Lưu ý:** `top`, `right`, `bottom`, `left` chỉ hoạt động khi `position` KHÔNG phải `static`.

---

## 2. Position: Static (Mặc định)

### 2.1. Static là gì?

`position: static` là **giá trị mặc định** của mọi element. Element sẽ hiển thị theo **luồng bình thường** (normal flow) của tài liệu.

### 2.2. Đặc điểm

- ✅ Hiển thị theo thứ tự trong HTML (từ trên xuống)
- ✅ Tuân theo box model bình thường
- ❌ **KHÔNG** sử dụng được `top`, `right`, `bottom`, `left`
- ❌ **KHÔNG** thể dùng `z-index`

### 2.3. Ví dụ

```html
<div class="box1">Box 1</div>
<div class="box2">Box 2</div>
<div class="box3">Box 3</div>
```

```css
.box1, .box2, .box3 {
    width: 150px;
    height: 100px;
    margin: 10px;
    padding: 10px;
    position: static; /* Mặc định, có thể bỏ qua */
}

.box1 { background: #ff6b6b; }
.box2 { background: #4ecdc4; }
.box3 { background: #ffe66d; }
```

**Kết quả hiển thị:**

```
┌─────────────┐
│   Box 1     │ ← Đỏ
└─────────────┘

┌─────────────┐
│   Box 2     │ ← Xanh dương
└─────────────┘

┌─────────────┐
│   Box 3     │ ← Vàng
└─────────────┘
```

**Giải thích:**
- Các box xếp theo thứ tự từ trên xuống
- Không thể dịch chuyển bằng `top`, `left`

### 2.4. Thử dùng top/left với static (KHÔNG hoạt động)

```css
.box2 {
    position: static;
    top: 50px;    /* ❌ KHÔNG có tác dụng */
    left: 50px;   /* ❌ KHÔNG có tác dụng */
}
```

**Kết quả:** Box 2 vẫn ở vị trí bình thường, không dịch chuyển.

### 2.5. Khi nào dùng static?

- ✅ Layout bình thường
- ✅ Khi không cần định vị đặc biệt
- ✅ Mặc định của browser (không cần khai báo)

**Khi nào KHÔNG dùng:**
- ❌ Cần dịch chuyển element
- ❌ Cần đặt element chồng lên nhau
- ❌ Cần làm positioning parent cho absolute child

---

## 3. Position: Relative (Tương đối)

### 3.1. Relative là gì?

`position: relative` cho phép element **dịch chuyển tương đối so với vị trí ban đầu** của nó, nhưng vẫn **giữ chỗ** trong luồng tài liệu.

### 3.2. Đặc điểm quan trọng

- ✅ Element giữ nguyên **vị trí ban đầu** trong layout
- ✅ **Không gian trống** được giữ lại (không bị elements khác lấp đầy)
- ✅ Có thể dịch chuyển bằng `top`, `right`, `bottom`, `left`
- ✅ Có thể dùng `z-index`
- ✅ Trở thành **positioning parent** cho `absolute` children

### 3.3. Ví dụ cơ bản

```html
<div class="box1">Box 1</div>
<div class="box2">Box 2 (Relative)</div>
<div class="box3">Box 3</div>
```

```css
.box1, .box2, .box3 {
    width: 150px;
    height: 100px;
    margin: 10px;
    padding: 10px;
}

.box1 { background: #ff6b6b; }

.box2 { 
    background: #4ecdc4;
    position: relative;
    top: 30px;      /* Dịch xuống 30px */
    left: 50px;     /* Dịch sang phải 50px */
}

.box3 { background: #ffe66d; }
```

**Kết quả hiển thị:**

```
┌─────────────┐
│   Box 1     │ 
└─────────────┘
               ┌─────────────┐
   [Khoảng     │   Box 2     │ ← Dịch sang phải + xuống
    trống]     └─────────────┘

┌─────────────┐
│   Box 3     │ ← Vẫn ở vị trí bình thường
└─────────────┘
```

**Giải thích chi tiết:**

1. **Box 2 dịch chuyển:**
   - `top: 30px` → Di chuyển xuống 30px
   - `left: 50px` → Di chuyển sang phải 50px

2. **Khoảng trống:**
   - Vị trí ban đầu của Box 2 vẫn được giữ lại
   - Box 3 KHÔNG lên lấp đầy chỗ trống

### 3.4. So sánh Static vs Relative

```css
/* Static (Mặc định) */
.box-static {
    position: static;
    top: 50px;    /* ❌ Không có tác dụng */
}

/* Relative */
.box-relative {
    position: relative;
    top: 50px;    /* ✅ Dịch xuống 50px */
}
```

**Visual comparison:**

```
Static:
┌───┐
│ A │
└───┘
┌───┐
│ B │ ← Vị trí bình thường, không thể dịch
└───┘

Relative:
┌───┐
│ A │
└───┘
[Trống]
  ┌───┐
  │ B │ ← Đã dịch, nhưng giữ chỗ trống
  └───┘
```

### 3.5. Ví dụ: Dịch chuyển nhiều hướng

```css
/* Lên trên + sang trái */
.move-up-left {
    position: relative;
    top: -20px;      /* Âm = lên trên */
    left: -30px;     /* Âm = sang trái */
}

/* Xuống dưới + sang phải */
.move-down-right {
    position: relative;
    bottom: -20px;   /* Âm = xuống dưới */
    right: -30px;    /* Âm = sang phải */
}
```

**Lưu ý về giá trị âm:**
- `top: -10px` = Lên trên 10px
- `left: -10px` = Sang trái 10px
- `bottom: -10px` = Xuống dưới 10px
- `right: -10px` = Sang phải 10px

### 3.6. Use Case: Positioning Parent

**Vai trò quan trọng nhất của `relative`:**
Làm **parent** cho các element `absolute`!

```html
<div class="parent">
    Parent (Relative)
    <div class="child">
        Child (Absolute)
    </div>
</div>
```

```css
.parent {
    position: relative; /* ✅ Làm parent cho absolute */
    width: 300px;
    height: 200px;
    background: #f0f0f0;
}

.child {
    position: absolute;
    top: 10px;
    right: 10px;
    background: #ff6b6b;
}
```

**Kết quả:**
```
┌─────────────────────────────────┐
│ Parent (Relative)      ┌─────┐  │
│                        │Child│  │ ← Ở góc trên phải
│                        └─────┘  │
│                                 │
└─────────────────────────────────┘
```

### 3.7. Khi nào dùng Relative?

✅ **NÊN dùng:**
- Làm parent cho `absolute` children (use case chính!)
- Dịch chuyển nhẹ element (nudge)
- Điều chỉnh vị trí mà không ảnh hưởng layout

❌ **KHÔNG NÊN dùng:**
- Tạo overlay (dùng `absolute` hoặc `fixed`)
- Layout phức tạp (dùng Flexbox/Grid)

---

## 4. Position: Absolute (Tuyệt đối)

### 4.1. Absolute là gì?

`position: absolute` **loại bỏ element** khỏi luồng tài liệu bình thường và định vị nó **tương đối so với parent gần nhất có position** (không phải `static`).

### 4.2. Đặc điểm quan trọng

- ❌ **Bị loại khỏi** normal flow
- ❌ **KHÔNG giữ chỗ** trong layout (như thể không tồn tại)
- ✅ Có thể dùng `top`, `right`, `bottom`, `left`
- ✅ Có thể dùng `z-index`
- ✅ Định vị **tương đối so với positioned parent**
- ✅ Nếu không có positioned parent → Định vị theo `<body>`

### 4.3. Absolute cần Positioned Parent

**Quy tắc vàng:**
```
Absolute tìm parent gần nhất có position khác static:
- relative ✅
- absolute ✅
- fixed ✅
- sticky ✅
- static ❌
```

### 4.4. Ví dụ: Absolute với Relative Parent

```html
<div class="container">
    Container (Relative)
    <div class="box">Box (Absolute)</div>
</div>
```

```css
.container {
    position: relative; /* ✅ Positioned parent */
    width: 400px;
    height: 300px;
    background: #f0f0f0;
    border: 2px solid #333;
}

.box {
    position: absolute;
    top: 20px;
    right: 30px;
    width: 100px;
    height: 100px;
    background: #ff6b6b;
}
```

**Kết quả:**

```
┌────────────────────────────────────┐
│ Container (Relative)      ┌─────┐  │
│                           │ Box │  │ ← Absolute
│                           └─────┘  │
│                                    │
│                                    │
└────────────────────────────────────┘
```

**Giải thích:**
- `.box` định vị theo `.container` (positioned parent)
- `top: 20px` → Cách container 20px từ trên
- `right: 30px` → Cách container 30px từ phải

### 4.5. Ví dụ: Absolute KHÔNG có Positioned Parent

```html
<div class="container">
    Container (KHÔNG có position)
    <div class="box">Box (Absolute)</div>
</div>
```

```css
.container {
    /* ❌ Không có position, mặc định là static */
    width: 400px;
    height: 300px;
    background: #f0f0f0;
}

.box {
    position: absolute;
    top: 20px;
    right: 30px;
    width: 100px;
    height: 100px;
    background: #ff6b6b;
}
```

**Kết quả:**

```
Browser Window
┌─────────────────────────────────────┐
│                          ┌─────┐    │ ← Box ở góc trên phải
│                          │ Box │    │    của WINDOW
│                          └─────┘    │
│  ┌──────────────────────────────┐  │
│  │ Container                    │  │
│  │                              │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Giải thích:**
- Container không có `position` (mặc định `static`)
- Box bỏ qua container, định vị theo `<body>`
- `top: 20px`, `right: 30px` → Tính từ cạnh window

### 4.6. So sánh Relative vs Absolute

```css
/* Relative: Giữ chỗ */
.relative-box {
    position: relative;
    top: 50px;
}

/* Absolute: Không giữ chỗ */
.absolute-box {
    position: absolute;
    top: 50px;
}
```

**Visual comparison:**

```
HTML:
<div>Box 1</div>
<div class="target">Box 2</div>
<div>Box 3</div>

Relative (giữ chỗ):
┌───────┐
│ Box 1 │
└───────┘
[Trống]
  ┌───────┐
  │ Box 2 │ ← Dịch nhưng giữ chỗ
  └───────┘
┌───────┐
│ Box 3 │ ← Ở vị trí bình thường
└───────┘

Absolute (không giữ chỗ):
┌───────┐
│ Box 1 │
└───────┘
┌───────┐
│ Box 3 │ ← Lên lấp chỗ trống!
└───────┘
  ┌───────┐
  │ Box 2 │ ← Nổi lên trên
  └───────┘
```

### 4.7. Centering với Absolute

**Cách 1: Transform (Modern, recommended)**

```css
.centered {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

**Cách 2: Margin auto (Cần biết width/height)**

```css
.centered {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
    width: 200px;  /* ⚠️ Bắt buộc */
    height: 100px; /* ⚠️ Bắt buộc */
}
```

**Cách 3: Calc (Cần biết width/height)**

```css
.centered {
    position: absolute;
    top: calc(50% - 50px);  /* 50px = height/2 */
    left: calc(50% - 100px); /* 100px = width/2 */
    width: 200px;
    height: 100px;
}
```

### 4.8. Ví dụ thực tế: Overlay

```html
<div class="card">
    <img src="image.jpg" alt="Image">
    <div class="overlay">
        <h3>Tiêu đề</h3>
        <p>Mô tả ngắn</p>
    </div>
</div>
```

```css
.card {
    position: relative; /* ✅ Parent */
    width: 300px;
    height: 400px;
}

.card img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.overlay {
    position: absolute; /* ✅ Child */
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    color: white;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    opacity: 0;
    transition: opacity 0.3s;
}

.card:hover .overlay {
    opacity: 1;
}
```

**Kết quả:**
```
Normal:                Hover:
┌──────────┐          ┌──────────┐
│          │          │ ▓▓▓▓▓▓▓▓ │ ← Overlay
│  Image   │    →     │ Tiêu đề  │
│          │          │ Mô tả    │
└──────────┘          └──────────┘
```

### 4.9. Khi nào dùng Absolute?

✅ **NÊN dùng:**
- Tooltips, dropdowns
- Modal overlays
- Badge, notification badges
- Close buttons (×)
- Icons overlay trên images
- Popup menus

❌ **KHÔNG NÊN dùng:**
- Layout chính của trang (dùng Flexbox/Grid)
- Khi cần element trong flow

---

## 5. So sánh Static vs Relative vs Absolute

### 5.1. Bảng so sánh tổng quan

| Đặc điểm | Static | Relative | Absolute |
|----------|--------|----------|----------|
| **Mặc định** | ✅ Có | ❌ Không | ❌ Không |
| **Normal flow** | ✅ Trong flow | ✅ Trong flow | ❌ Ngoài flow |
| **Giữ chỗ** | ✅ Giữ | ✅ Giữ | ❌ Không giữ |
| **top/left** | ❌ Không dùng được | ✅ Dùng được | ✅ Dùng được |
| **z-index** | ❌ Không dùng được | ✅ Dùng được | ✅ Dùng được |
| **Định vị theo** | - | Vị trí ban đầu | Positioned parent |
| **Ảnh hưởng elements khác** | ✅ Có | ✅ Có | ❌ Không |

### 5.2. Visual comparison

```html
<div class="box static">Static</div>
<div class="box relative">Relative</div>
<div class="box absolute">Absolute</div>
<div class="box normal">Normal</div>
```

```css
.box {
    width: 150px;
    height: 100px;
    margin: 10px;
}

.static {
    position: static;
    top: 50px;     /* ❌ Không có tác dụng */
}

.relative {
    position: relative;
    top: 50px;     /* ✅ Dịch xuống 50px */
    left: 50px;
}

.absolute {
    position: absolute;
    top: 50px;     /* ✅ Theo parent */
    left: 50px;
}
```

**Kết quả:**

```
┌─────────┐
│ Static  │ ← Vị trí bình thường
└─────────┘

  [Trống]
  ┌─────────┐
  │Relative │ ← Dịch nhưng giữ chỗ
  └─────────┘

┌─────────┐
│ Normal  │ ← Lên lấp chỗ của Absolute
└─────────┘

  ┌──────────┐
  │ Absolute │ ← Nổi lên, không giữ chỗ
  └──────────┘
```

### 5.3. So sánh code

```css
/* 1. STATIC - Mặc định */
.element-static {
    position: static;
    /* top, left KHÔNG hoạt động */
    /* z-index KHÔNG hoạt động */
}

/* 2. RELATIVE - Dịch chuyển + Giữ chỗ */
.element-relative {
    position: relative;
    top: 20px;        /* ✅ Dịch xuống 20px */
    left: 30px;       /* ✅ Dịch phải 30px */
    z-index: 10;      /* ✅ Có thể dùng */
    /* Vị trí ban đầu vẫn được giữ */
}

/* 3. ABSOLUTE - Loại khỏi flow */
.element-absolute {
    position: absolute;
    top: 20px;        /* ✅ Theo positioned parent */
    left: 30px;
    z-index: 10;      /* ✅ Có thể dùng */
    /* KHÔNG giữ chỗ trong layout */
}
```

### 5.4. Use Cases

**Static:**
- Tất cả layout bình thường
- Khi không cần positioning đặc biệt

**Relative:**
- Làm parent cho absolute children (★ quan trọng nhất)
- Dịch chuyển nhẹ element
- Tạo stacking context cho z-index

**Absolute:**
- Tooltips, dropdowns
- Modal overlays
- Close buttons, badges
- Popup menus
- Icons overlay

---

## 6. Position: Fixed và Sticky (Bonus)

### 6.1. Position: Fixed

**Định nghĩa:** Element **cố định** theo viewport (cửa sổ browser), không scroll theo trang.

```css
.fixed-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background: white;
    z-index: 1000;
}
```

**Đặc điểm:**
- ❌ Loại khỏi normal flow
- ✅ Cố định khi scroll
- ✅ Định vị theo viewport

**Use cases:**
- Fixed header/navigation
- Back to top button
- Chat widget
- Cookie banner

### 6.2. Position: Sticky

**Định nghĩa:** Kết hợp `relative` và `fixed` - Element "dính" khi scroll đến vị trí nhất định.

```css
.sticky-nav {
    position: sticky;
    top: 0;
    background: white;
    z-index: 100;
}
```

**Đặc điểm:**
- ✅ Ban đầu: `relative` (trong flow)
- ✅ Khi scroll: "Dính" tại vị trí chỉ định
- ✅ Giữ chỗ trong layout

**Use cases:**
- Sticky table headers
- Section headers trong long content
- Sidebar navigation

---

## 7. Ví dụ thực tế

### 7.1. Card với Badge (Absolute)

```html
<div class="product-card">
    <span class="badge">Sale 50%</span>
    <img src="product.jpg" alt="Product">
    <h3>Tên sản phẩm</h3>
    <p class="price">199.000đ</p>
</div>
```

```css
.product-card {
    position: relative; /* ✅ Parent */
    width: 250px;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
}

.badge {
    position: absolute; /* ✅ Child */
    top: 10px;
    right: 10px;
    background: #ff4444;
    color: white;
    padding: 5px 10px;
    border-radius: 4px;
    font-size: 12px;
    font-weight: bold;
}
```

### 7.2. Tooltip (Absolute)

```html
<button class="tooltip-trigger">
    Hover me
    <span class="tooltip">This is a tooltip</span>
</button>
```

```css
.tooltip-trigger {
    position: relative; /* ✅ Parent */
    padding: 10px 20px;
}

.tooltip {
    position: absolute; /* ✅ Child */
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    background: #333;
    color: white;
    padding: 8px 12px;
    border-radius: 4px;
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.3s;
}

.tooltip-trigger:hover .tooltip {
    opacity: 1;
}

/* Arrow */
.tooltip::after {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translateX(-50%);
    border: 6px solid transparent;
    border-top-color: #333;
}
```

### 7.3. Modal Overlay (Fixed + Absolute)

```html
<div class="modal-overlay">
    <div class="modal">
        <button class="close-btn">×</button>
        <h2>Modal Title</h2>
        <p>Modal content...</p>
    </div>
</div>
```

```css
/* Overlay - Fixed */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

/* Modal - Relative (parent cho close button) */
.modal {
    position: relative;
    background: white;
    border-radius: 8px;
    padding: 30px;
    max-width: 500px;
    width: 90%;
}

/* Close button - Absolute */
.close-btn {
    position: absolute;
    top: 10px;
    right: 10px;
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
}
```