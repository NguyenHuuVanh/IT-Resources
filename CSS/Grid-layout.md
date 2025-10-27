# Hướng dẫn hoàn chỉnh về CSS Grid

> **Lưu ý**: Bài viết này được dịch và tổng hợp từ [CSS-Tricks](https://css-tricks.com/snippets/css/complete-guide-grid/)

## Mục lục

1. [Background - Giới thiệu](#background)
2. [Cơ bản và thuật ngữ](#co-ban-va-thuat-ngu)
3. [Properties cho Parent (Grid Container)](#properties-cho-parent)
4. [Properties cho Children (Grid Items)](#properties-cho-children)
5. [Ví dụ thực tế](#vi-du-thuc-te)
6. [Browser Support](#browser-support)
7. [Grid Tricks](#grid-tricks)

---

## Background

CSS Grid Layout (hay còn gọi là "Grid") là một hệ thống layout hai chiều mạnh mẽ được thiết kế cho web. Nó cho phép bạn bố trí nội dung theo hàng và cột, và có nhiều tính năng giúp dễ dàng tạo ra các layouts phức tạp. Nó được hỗ trợ bởi tất cả các trình duyệt hiện đại (Safari 10.1+, Chrome 57+, Firefox 52+, Edge 16+).

Grid khác với Flexbox ở chỗ Flexbox là một chiều (hàng hoặc cột), trong khi Grid là hai chiều (hàng và cột cùng lúc). Tuy nhiên, chúng có thể kết hợp với nhau để tạo ra các layouts phức tạp.

---

## Cơ bản và thuật ngữ

### Grid Container

Phần tử cha được áp dụng `display: grid` trở thành grid container. Nó định nghĩa grid context cho tất cả các phần tử con trực tiếp (grid items).

### Grid Item

Các phần tử con trực tiếp của grid container trở thành grid items. Trong ví dụ dưới đây, các phần tử `div` bên trong `.container` là grid items.

```html
<div class="container">
  <div class="item item-1">1</div>
  <div class="item item-2">2</div>
  <div class="item item-3">3</div>
</div>
```

### Grid Line

Các đường kẻ chia grid thành các cột và hàng. Chúng có thể được tham chiếu theo số hoặc tên.

### Grid Track

Không gian giữa hai grid lines liền kề. Bạn có thể nghĩ về chúng như các cột hoặc hàng của grid.

### Grid Cell

Không gian giữa bốn grid lines liền kề. Nó là đơn vị nhỏ nhất của grid.

### Grid Area

Tổng hợp của một hoặc nhiều grid cells. Grid areas được tạo ra khi bạn định nghĩa grid template areas.

### Sơ đồ Grid

```
┌─────────────────── Grid Container ───────────────────┐
│  ┌─────┬─────┬─────┐  ┌─────┬─────┬─────┐           │
│  │  1  │  2  │  3  │  │  4  │  5  │  6  │           │
│  ├─────┼─────┼─────┤  ├─────┼─────┼─────┤           │
│  │  7  │  8  │  9  │  │ 10  │ 11  │ 12  │           │
│  └─────┴─────┴─────┘  └─────┴─────┴─────┘           │
│                                                     │
│  ◀────────── Grid Columns ────────────▶            │
│  ▲                                                 │
│  │ Grid Rows                                       │
│  ▼                                                 │
└─────────────────────────────────────────────────────┘
```

---

## Properties cho Parent

### display

Định nghĩa element là grid container và thiết lập một new grid formatting context cho nội dung của nó.

```css
.container {
  display: grid | inline-grid;
}
```

- `grid` - Tạo một block-level grid
- `inline-grid` - Tạo một inline-level grid

### grid-template-columns, grid-template-rows

Định nghĩa các cột và hàng của grid với một danh sách các giá trị. Các giá trị đại diện cho kích thước track và không gian giữa chúng.

```css
.container {
  grid-template-columns: 40px 50px auto 50px 40px;
  grid-template-rows: 25% 100px auto;
}
```

### grid-template-areas

Định nghĩa một grid template bằng cách tham chiếu đến các tên của grid areas được chỉ định bởi property `grid-area`. Lặp lại tên của một grid area khiến nội dung trải dài qua các cells đó. Một dấu chấm đại diện cho một empty cell. Cú pháp tự nó cung cấp một hình ảnh trực quan của cấu trúc grid.

```css
.container {
  grid-template-areas: 
    "header header header header"
    "main main . sidebar"
    "footer footer footer footer";
}
```

### grid-template

Shorthand cho `grid-template-rows`, `grid-template-columns`, và `grid-template-areas` trong một declaration duy nhất.

```css
.container {
  grid-template:
    "header header header" 50px
    "main main sidebar" 200px
    "footer footer footer" 100px /
    1fr 1fr 200px;
}
```

### column-gap, row-gap

Chỉ định kích thước của gutters giữa các cột và hàng.

```css
.container {
  column-gap: 20px;
  row-gap: 10px;
}
```

### gap

Shorthand cho `row-gap` và `column-gap`.

```css
.container {
  gap: 10px 20px; /* row-gap column-gap */
}
```

### justify-items

Căn chỉnh grid items dọc theo row axis (inline axis). Giá trị này áp dụng cho tất cả grid items bên trong container.

```css
.container {
  justify-items: start | end | center | stretch;
}
```

### align-items

Căn chỉnh grid items dọc theo column axis (block axis). Giá trị này áp dụng cho tất cả grid items bên trong container.

```css
.container {
  align-items: start | end | center | stretch;
}
```

### justify-content

Đôi khi grid tổng thể có không gian còn lại trong container. Property `justify-content` căn chỉnh grid dọc theo inline axis (như flexbox).

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
}
```

### align-content

Đôi khi grid tổng thể có không gian còn lại trong container. Property `align-content` căn chỉnh grid dọc theo block axis (như flexbox).

```css
.container {
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;
}
```

### grid-auto-columns, grid-auto-rows

Chỉ định kích thước của bất kỳ auto-generated grid tracks nào (còn được gọi là implicit grid tracks).

```css
.container {
  grid-auto-columns: 60px;
  grid-auto-rows: 60px;
}
```

### grid-auto-flow

Nếu bạn có grid items không được đặt rõ ràng trong grid, auto-placement algorithm sẽ tự động đặt chúng. Property `grid-auto-flow` kiểm soát cách thức hoạt động của algorithm đó.

```css
.container {
  grid-auto-flow: row | column | row dense | column dense;
}
```

### grid

Shorthand cho tất cả các properties sau: `grid-template-rows`, `grid-template-columns`, `grid-template-areas`, `grid-auto-rows`, `grid-auto-columns`, và `grid-auto-flow`.

```css
.container {
  grid: 100px 300px / 3fr 1fr;
}
```

---

## Properties cho Children

### grid-column-start, grid-column-end, grid-row-start, grid-row-end

Xác định vị trí của grid item trong grid bằng cách tham chiếu đến các line cụ thể. `grid-column-start`/`grid-row-start` là line nơi grid item bắt đầu, và `grid-column-end`/`grid-row-end` là line nơi grid item kết thúc.

```css
.item {
  grid-column-start: 2;
  grid-column-end: 5;
  grid-row-start: 1;
  grid-row-end: 3;
}
```

### grid-column, grid-row

Shorthand cho `grid-column-start` + `grid-column-end`, và `grid-row-start` + `grid-row-end`.

```css
.item {
  grid-column: 2 / 5;
  grid-row: 1 / 3;
}
```

### grid-area

Shorthand cho `grid-row-start`, `grid-column-start`, `grid-row-end`, và `grid-column-end` trong một declaration duy nhất.

```css
.item {
  grid-area: 1 / 2 / 4 / 6;
}
```

Hoặc cung cấp tên cho grid area:

```css
.item {
  grid-area: header;
}
```

### justify-self

Căn chỉnh một grid item bên trong cell dọc theo inline axis (như `justify-items` nhưng chỉ cho item này).

```css
.item {
  justify-self: start | end | center | stretch;
}
```

### align-self

Căn chỉnh một grid item bên trong cell dọc theo block axis (như `align-items` nhưng chỉ cho item này).

```css
.item {
  align-self: start | end | center | stretch;
}
```

---

## Ví dụ thực tế

### Basic Grid

```css
.container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 100px 100px;
  gap: 10px;
}
```

### Named Grid Areas

```css
.container {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar main main"
    "footer footer footer";
  grid-template-columns: 200px 1fr 1fr;
  grid-template-rows: 60px 1fr 60px;
  gap: 10px;
}

.header {
  grid-area: header;
}

.sidebar {
  grid-area: sidebar;
}

.main {
  grid-area: main;
}

.footer {
  grid-area: footer;
}
```

### Responsive Grid

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 10px;
}
```

### 12-Column Grid System

```css
.container {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 20px;
}

.item-1 {
  grid-column: span 1;
}

.item-4 {
  grid-column: span 4;
}

.item-6 {
  grid-column: span 6;
}
```

---

## Browser Support

CSS Grid Layout được hỗ trợ bởi tất cả các trình duyệt hiện đại:

| Desktop | Chrome | Firefox | IE | Edge | Safari |
|---------|--------|---------|-----|------|--------|
| | 57+ | 52+ | 11* | 16+ | 10.1+ |

| Mobile/Tablet | Android Chrome | Android Firefox | iOS Safari |
|---------------|----------------|-----------------|------------|
| | 125+ | 124+ | 10.3+ |

> **Lưu ý**: IE 11 có hỗ trợ hạn chế với `-ms-` prefix. Sử dụng Autoprefixer để xử lý fallbacks.

---

## Grid Tricks

### Implicit vs Explicit Grids

Grid có thể có tracks được định nghĩa rõ ràng (explicit) và tự động tạo (implicit).

### Grid Auto-Placement

Cách thức grid tự động đặt items khi không được chỉ định vị trí.

### Using Grid with Flexbox

Kết hợp Grid và Flexbox để tạo layouts phức tạp.

### Grid and Accessibility

Đảm bảo Grid layouts thân thiện với accessibility.

### Common Grid Patterns

Các pattern phổ biến như Holy Grail, Card Layouts, v.v.

---

## Kết luận

CSS Grid là một công cụ mạnh mẽ cho việc tạo layouts phức tạp trên web. Nó cung cấp khả năng kiểm soát chi tiết và linh hoạt hơn so với các phương pháp layout truyền thống. Khi kết hợp với Flexbox, bạn có thể tạo ra hầu hết mọi loại layout hiện đại.

### Tài nguyên bổ sung

- [CSS Grid Layout Module Level 1 (W3C)](https://www.w3.org/TR/css-grid-1/)
- [Grid Garden](https://cssgridgarden.com/) - Game để học CSS Grid
- [CSS Grid Cheat Sheet](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Grid by Example](https://gridbyexample.com/)

---

**Tác giả**: Chris Coyier (CSS-Tricks)  
**Dịch và biên soạn**: [Việt Anh Nguyễn]  
**Cập nhật lần cuối**: [27/10/2025]

---

> **Lời khuyên**: CSS Grid và Flexbox không phải là đối thủ cạnh tranh mà là công cụ bổ sung cho nhau. Học cả hai sẽ giúp bạn tạo ra các layouts mạnh mẽ và linh hoạt!