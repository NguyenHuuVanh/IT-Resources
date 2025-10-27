# Hướng dẫn hoàn chỉnh về CSS Flexbox

> **Lưu ý**: Bài viết này được dịch và tổng hợp từ nhiều nguồn, bao gồm [CSS-Tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

## Mục lục

1. [Background - Giới thiệu](#background)
2. [Cơ bản và thuật ngữ](#co-ban-va-thuat-ngu)
3. [Properties cho Parent (Flex Container)](#properties-cho-parent)
4. [Properties cho Children (Flex Items)](#properties-cho-children)
5. [Ví dụ thực tế](#vi-du-thuc-te)
6. [Browser Support](#browser-support)
7. [Flexbox Tricks](#flexbox-tricks)

---

## Background

Module **Flexbox Layout** (Flexible Box) nhằm mục đích cung cấp một cách hiệu quả hơn để bố trí, căn chỉnh và phân phối không gian giữa các items trong một container, ngay cả khi kích thước của chúng không xác định và/hoặc động (do đó có từ "flex").

Ý tưởng chính đằng sau flex layout là cho phép container có khả năng thay đổi width/height (và thứ tự) của các items để lấp đầy không gian có sẵn một cách tốt nhất (chủ yếu để phù hợp với tất cả các loại thiết bị hiển thị và kích thước màn hình). Một flex container mở rộng các items để lấp đầy không gian trống có sẵn hoặc thu nhỏ chúng để ngăn chặn tràn.

Quan trọng nhất, flexbox layout là **direction-agnostic** (không phụ thuộc vào hướng) trái ngược với các layout thông thường (block theo chiều dọc và inline theo chiều ngang). Mặc dù những cách đó hoạt động tốt cho các trang, nhưng chúng thiếu tính linh hoạt để hỗ trợ các ứng dụng lớn hoặc phức tạp (đặc biệt khi nói đến thay đổi hướng, thay đổi kích thước, kéo dài, co lại, v.v.).

> **Lưu ý**: Flexbox layout thích hợp nhất cho các components của một ứng dụng và các layouts quy mô nhỏ, trong khi **Grid layout** được dự định cho các layouts quy mô lớn hơn.

---

## Cơ bản và thuật ngữ

Vì flexbox là một module hoàn chỉnh chứ không phải một property đơn lẻ, nó bao gồm rất nhiều thứ bao gồm toàn bộ tập hợp các properties. Một số trong đó được đặt trên **container** (phần tử cha, được gọi là "flex container") trong khi những cái khác được đặt trên **children** (được gọi là "flex items").

Nếu layout "thông thường" dựa trên cả hai hướng luồng block và inline, thì flex layout dựa trên **"flex-flow directions"**. 

### Sơ đồ Flex Container

```
┌─────────────────── Flex Container ───────────────────┐
│                                                       │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐   │
│  │ Item 1 │  │ Item 2 │  │ Item 3 │  │ Item 4 │   │
│  └────────┘  └────────┘  └────────┘  └────────┘   │
│                                                       │
│  ◀────────── Main Axis (trục chính) ────────────▶   │
│  ▲                                                    │
│  │ Cross Axis (trục phụ)                            │
│  ▼                                                    │
└───────────────────────────────────────────────────────┘
```

Items sẽ được bố trí theo **main axis** (từ main-start đến main-end) hoặc **cross axis** (từ cross-start đến cross-end).

### Các thuật ngữ quan trọng

- **main axis** – Trục chính của flex container là trục chính mà các flex items được bố trí theo. Lưu ý, nó không nhất thiết phải là ngang; nó phụ thuộc vào property `flex-direction`.

- **main-start | main-end** – Các flex items được đặt trong container bắt đầu từ main-start và đi đến main-end.

- **main size** – Width hoặc height của flex item, tùy thuộc vào chiều nào là chiều chính, là main size của item đó.

- **cross axis** – Trục vuông góc với main axis được gọi là cross axis. Hướng của nó phụ thuộc vào hướng main axis.

- **cross-start | cross-end** – Các flex lines được lấp đầy bằng items và được đặt vào container bắt đầu từ cross-start và đi về phía cross-end.

- **cross size** – Width hoặc height của flex item, tùy thuộc vào chiều nào là chiều cross, là cross size của item.

---

## Properties cho Parent

### display

Định nghĩa flex container; inline hoặc block tùy thuộc vào giá trị được cung cấp.

```css
.container {
  display: flex; /* hoặc inline-flex */
}
```

### flex-direction

![flex-direction](https://css-tricks.com/wp-content/uploads/2018/10/flex-direction.svg)

Thiết lập main-axis, do đó xác định hướng mà các flex items được đặt trong flex container.

```css
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

- `row` (default): trái sang phải trong `ltr`; phải sang trái trong `rtl`
- `row-reverse`: phải sang trái trong `ltr`; trái sang phải trong `rtl`
- `column`: giống như `row` nhưng từ trên xuống dưới
- `column-reverse`: giống như `row-reverse` nhưng từ dưới lên trên

### flex-wrap

![flex-wrap](https://css-tricks.com/wp-content/uploads/2018/10/flex-wrap.svg)

Theo mặc định, các flex items sẽ cố gắng vừa trên một dòng. Bạn có thể thay đổi điều đó và cho phép các items wrap khi cần thiết với property này.

```css
.container {
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

- `nowrap` (default): tất cả flex items sẽ ở trên một dòng
- `wrap`: các flex items sẽ wrap sang nhiều dòng, từ trên xuống dưới
- `wrap-reverse`: các flex items sẽ wrap sang nhiều dòng từ dưới lên trên

### flex-flow

Đây là shorthand cho `flex-direction` và `flex-wrap`, kết hợp với nhau để định nghĩa main axis và cross axis của flex container. Giá trị mặc định là `row nowrap`.

```css
.container {
  flex-flow: row wrap;
}
```

### justify-content

![justify-content](https://css-tricks.com/wp-content/uploads/2018/10/justify-content.svg)

Định nghĩa alignment dọc theo main axis. Nó giúp phân phối không gian trống còn lại khi tất cả các flex items trên một line không linh hoạt, hoặc linh hoạt nhưng đã đạt đến kích thước tối đa của chúng.

```css
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
}
```

- `flex-start` (default): items được đóng gói về phía start của flex-direction
- `flex-end`: items được đóng gói về phía end của flex-direction
- `center`: items được căn giữa dọc theo line
- `space-between`: items được phân phối đều trong line; item đầu tiên ở start line, item cuối cùng ở end line
- `space-around`: items được phân phối đều với khoảng cách bằng nhau xung quanh chúng
- `space-evenly`: items được phân phối sao cho khoảng cách giữa hai items bất kỳ (và khoảng cách đến các cạnh) là bằng nhau

### align-items

![align-items](https://css-tricks.com/wp-content/uploads/2018/10/align-items.svg)

Định nghĩa hành vi mặc định cho cách các flex items được bố trí dọc theo cross axis trên line hiện tại. Hãy nghĩ về nó như phiên bản `justify-content` cho cross-axis (vuông góc với main-axis).

```css
.container {
  align-items: stretch | flex-start | flex-end | center | baseline;
}
```

- `stretch` (default): kéo dài để lấp đầy container (vẫn tôn trọng min-width/max-width)
- `flex-start`: items được đặt ở start của cross axis
- `flex-end`: items được đặt ở end của cross axis
- `center`: items được căn giữa trong cross-axis
- `baseline`: items được căn chỉnh sao cho baselines của chúng align

### align-content

![align-content](https://css-tricks.com/wp-content/uploads/2018/10/align-content.svg)

Căn chỉnh các lines của flex container khi có không gian thừa trong cross-axis, tương tự như cách `justify-content` căn chỉnh các items riêng lẻ trong main-axis.

> **Lưu ý**: Property này chỉ có hiệu lực trên flexible containers nhiều dòng, nơi `flex-wrap` được đặt thành `wrap` hoặc `wrap-reverse`.

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | space-evenly | stretch;
}
```

- `normal` (default): items được đóng gói ở vị trí mặc định
- `flex-start`: items được đóng gói về phía start của container
- `flex-end`: items được đóng gói về phía end của container
- `center`: items được căn giữa trong container
- `space-between`: items được phân phối đều; line đầu tiên ở start của container trong khi line cuối cùng ở end
- `space-around`: items được phân phối đều với khoảng cách bằng nhau xung quanh mỗi line
- `space-evenly`: items được phân phối đều với khoảng cách bằng nhau xung quanh chúng
- `stretch`: lines kéo dài để chiếm không gian còn lại

### gap, row-gap, column-gap

![gap](https://css-tricks.com/wp-content/uploads/2021/09/gap-1.svg)

Property `gap` kiểm soát rõ ràng không gian giữa các flex items. Nó chỉ áp dụng khoảng cách đó giữa các items chứ không phải ở các cạnh ngoài.

```css
.container {
  display: flex;
  gap: 10px;
  gap: 10px 20px; /* row-gap column gap */
  row-gap: 10px;
  column-gap: 20px;
}
```

Hành vi có thể được coi như một gutter tối thiểu, như thể gutter lớn hơn bằng cách nào đó (vì một cái gì đó như `justify-content: space-between;`) thì gap sẽ chỉ có hiệu lực nếu không gian đó sẽ nhỏ hơn.

> **Lưu ý**: `gap` không chỉ dành riêng cho flexbox, nó cũng hoạt động trong grid và multi-column layout.

---

## Properties cho Children

### order

![order](https://css-tricks.com/wp-content/uploads/2018/10/order.svg)

Theo mặc định, các flex items được bố trí theo thứ tự source. Tuy nhiên, property `order` kiểm soát thứ tự mà chúng xuất hiện trong flex container.

```css
.item {
  order: 5; /* default is 0 */
}
```

Items có cùng `order` sẽ về thứ tự source.

### flex-grow

![flex-grow](https://css-tricks.com/wp-content/uploads/2018/10/flex-grow.svg)

Định nghĩa khả năng cho một flex item phát triển nếu cần thiết. Nó chấp nhận một giá trị không có đơn vị đóng vai trò như một tỷ lệ. Nó quyết định lượng không gian có sẵn bên trong flex container mà item nên chiếm.

Nếu tất cả các items có `flex-grow` được đặt thành `1`, không gian còn lại trong container sẽ được phân phối đều cho tất cả children. Nếu một trong các children có giá trị `2`, child đó sẽ chiếm gấp đôi không gian so với các child khác.

```css
.item {
  flex-grow: 4; /* default 0 */
}
```

Số âm là không hợp lệ.

### flex-shrink

Định nghĩa khả năng cho một flex item co lại nếu cần thiết.

```css
.item {
  flex-shrink: 3; /* default 1 */
}
```

Số âm là không hợp lệ.

### flex-basis

Định nghĩa kích thước mặc định của một element trước khi không gian còn lại được phân phối. Nó có thể là một length (ví dụ: 20%, 5rem, v.v.) hoặc một keyword.

```css
.item {
  flex-basis:  | auto; /* default auto */
}
```

Keyword `auto` có nghĩa là "nhìn vào property width hoặc height của tôi". Keyword `content` có nghĩa là "kích thước nó dựa trên nội dung của item".

Nếu được đặt thành `0`, không gian thừa xung quanh nội dung không được tính đến. Nếu được đặt thành `auto`, không gian thừa được phân phối dựa trên giá trị `flex-grow` của nó.

### flex

![flex](https://css-tricks.com/wp-content/uploads/2018/10/flex-shorthand.svg)

Đây là shorthand cho `flex-grow`, `flex-shrink` và `flex-basis` kết hợp. Tham số thứ hai và thứ ba (`flex-shrink` và `flex-basis`) là tùy chọn. Mặc định là `0 1 auto`.

```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

**Khuyến nghị**: Sử dụng shorthand property này thay vì đặt các properties riêng lẻ. Shorthand đặt các giá trị khác một cách thông minh.

### align-self

![align-self](https://css-tricks.com/wp-content/uploads/2018/10/align-self.svg)

Cho phép alignment mặc định (hoặc alignment được chỉ định bởi `align-items`) được ghi đè cho các flex items riêng lẻ.

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

> **Lưu ý**: `float`, `clear` và `vertical-align` không có hiệu lực trên một flex item.

---

## Ví dụ thực tế

### Perfect Centering

```css
.parent {
  display: flex;
  height: 300px;
}

.child {
  width: 100px;
  height: 100px;
  margin: auto; /* Magic! */
}
```

Điều này dựa vào thực tế là một margin được đặt thành `auto` trong một flex container hấp thụ không gian thừa. Vì vậy, đặt margin của `auto` sẽ làm cho item được căn giữa hoàn hảo trong cả hai trục.

### Navigation

```css
/* Large */
.navigation {
  display: flex;
  flex-flow: row wrap;
  justify-content: flex-end;
}

/* Medium screens */
@media all and (max-width: 800px) {
  .navigation {
    justify-content: space-around;
  }
}

/* Small screens */
@media all and (max-width: 500px) {
  .navigation {
    flex-direction: column;
  }
}
```

### Mobile-first 3-columns layout

```css
.wrapper {
  display: flex;
  flex-flow: row wrap;
}

/* We tell all items to be 100% width */
.wrapper > * {
  flex: 1 100%;
}

/* Medium screens */
@media all and (min-width: 600px) {
  .aside { flex: 1 auto; }
}

/* Large screens */
@media all and (min-width: 800px) {
  .main { flex: 3 0px; }
  .aside-1 { order: 1; }
  .main { order: 2; }
  .aside-2 { order: 3; }
  .footer { order: 4; }
}
```

---

## Browser Support

| Desktop | Chrome | Firefox | IE | Edge | Safari |
|---------|--------|---------|-----|------|--------|
| | 29+ | 28+ | 11 | 12+ | 9+ |

| Mobile/Tablet | Android Chrome | Android Firefox | iOS Safari |
|---------------|----------------|-----------------|------------|
| | 125+ | 124+ | 9.0-9.3+ |

> **Lưu ý**: Flexbox yêu cầu một số vendor prefixing để hỗ trợ nhiều trình duyệt nhất có thể. Cách tốt nhất để xử lý điều này là viết theo cú pháp mới (và cuối cùng) và chạy CSS của bạn qua **Autoprefixer**, xử lý các fallbacks rất tốt.

---

## Flexbox Tricks

### Adaptive Photo Layout with Flexbox

Sử dụng flexbox để tạo photo galleries responsive với các tỷ lệ khác nhau.

### Balancing on a Pivot with Flexbox

Tạo layouts cân bằng với flexbox bằng cách sử dụng `margin: auto`.

### Using Flexbox and text ellipsis together

Kết hợp flexbox với `text-overflow: ellipsis` để tạo text truncation.

### Flexbox and Truncated Text

Xử lý văn bản bị cắt ngắn trong flex containers.

### Filling the Space in the Last Row

Xử lý các items trong hàng cuối cùng của flex layout.

---

## Kết luận

Flexbox là một công cụ mạnh mẽ cho CSS layout hiện đại. Nó giải quyết nhiều vấn đề mà chúng ta đã phải đối mặt với floats và positioning trong nhiều năm. Mặc dù có một số đường cong học tập, việc đầu tư thời gian để hiểu flexbox sẽ làm cho CSS của bạn mạnh mẽ và dễ bảo trì hơn nhiều.

### Tài nguyên bổ sung

- [CSS Flexible Box Layout Module Level 1 (W3C)](https://www.w3.org/TR/css-flexbox-1/)
- [Flexbox Froggy](https://flexboxfroggy.com/) - Game để học Flexbox
- [Flexbox Defense](http://www.flexboxdefense.com/) - Game tower defense với Flexbox
- [Solved by Flexbox](https://philipwalton.github.io/solved-by-flexbox/) - Các ví dụ thực tế

---

**Tác giả**: Chris Coyier (CSS-Tricks)  
**Dịch và biên soạn**: [Việt Anh Nguyễn]  
**Cập nhật lần cuối**: [27/10/2025]

---

> **Lời khuyên**: Nếu bạn muốn tìm hiểu sâu hơn, hãy thử kết hợp Flexbox với CSS Grid để tạo ra các layouts phức tạp và mạnh mẽ hơn!