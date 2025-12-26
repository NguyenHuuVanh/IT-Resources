# AJAX (Asynchronous JavaScript and XML)

## AJAX là gì?

AJAX là một kỹ thuật lập trình web cho phép trang web giao tiếp với server và cập nhật nội dung mà **không cần reload toàn bộ trang**. Mặc dù tên có chứa "XML", ngày nay AJAX thường sử dụng JSON thay vì XML.

## Cách hoạt động của AJAX

```
┌─────────────┐     1. Request      ┌─────────────┐
│             │ ─────────────────▶  │             │
│   Browser   │                     │   Server    │
│             │ ◀─────────────────  │             │
└─────────────┘     2. Response     └─────────────┘
       │
       ▼
  3. Update DOM
  (không reload)
```

## Các cách thực hiện AJAX

### 1. XMLHttpRequest (Cách cũ)

```javascript
// Tạo request
const xhr = new XMLHttpRequest();

// Cấu hình request
xhr.open("GET", "https://api.example.com/data", true);

// Xử lý response
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
      const data = JSON.parse(xhr.responseText);
      console.log(data);
    } else {
      console.error("Error:", xhr.status);
    }
  }
};

// Gửi request
xhr.send();
```

**ReadyState values:**
| Value | State | Mô tả |
|-------|-------|-------|
| 0 | UNSENT | Request chưa được khởi tạo |
| 1 | OPENED | Đã gọi open() |
| 2 | HEADERS_RECEIVED | Đã nhận headers |
| 3 | LOADING | Đang tải response |
| 4 | DONE | Hoàn thành |

### 2. Fetch API (Cách hiện đại)

```javascript
// GET Request
fetch("https://api.example.com/data")
  .then((response) => {
    if (!response.ok) {
      throw new Error("Network response was not ok");
    }
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((error) => console.error("Error:", error));

// POST Request
fetch("https://api.example.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "John Doe",
    email: "john@example.com",
  }),
})
  .then((response) => response.json())
  .then((data) => console.log(data));
```

### 3. Fetch với Async/Await

```javascript
// GET Request
async function fetchData() {
  try {
    const response = await fetch("https://api.example.com/data");

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    console.log(data);
    return data;
  } catch (error) {
    console.error("Fetch error:", error);
  }
}

// POST Request
async function createUser(userData) {
  try {
    const response = await fetch("https://api.example.com/users", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: "Bearer token123",
      },
      body: JSON.stringify(userData),
    });

    const result = await response.json();
    return result;
  } catch (error) {
    console.error("Error:", error);
  }
}
```

## Các HTTP Methods phổ biến

| Method | Mục đích          | Có Body  |
| ------ | ----------------- | -------- |
| GET    | Lấy dữ liệu       | Không    |
| POST   | Tạo mới dữ liệu   | Có       |
| PUT    | Cập nhật toàn bộ  | Có       |
| PATCH  | Cập nhật một phần | Có       |
| DELETE | Xóa dữ liệu       | Không/Có |

## Xử lý các loại Response

```javascript
async function handleResponse(url) {
  const response = await fetch(url);

  // JSON
  const jsonData = await response.json();

  // Text
  const textData = await response.text();

  // Blob (file, image)
  const blobData = await response.blob();

  // ArrayBuffer (binary data)
  const bufferData = await response.arrayBuffer();

  // FormData
  const formData = await response.formData();
}
```

## Error Handling

```javascript
async function fetchWithErrorHandling(url) {
  try {
    const response = await fetch(url);

    // Kiểm tra HTTP status
    if (!response.ok) {
      switch (response.status) {
        case 400:
          throw new Error("Bad Request");
        case 401:
          throw new Error("Unauthorized");
        case 403:
          throw new Error("Forbidden");
        case 404:
          throw new Error("Not Found");
        case 500:
          throw new Error("Internal Server Error");
        default:
          throw new Error(`HTTP Error: ${response.status}`);
      }
    }

    return await response.json();
  } catch (error) {
    if (error.name === "TypeError") {
      // Network error (no internet, CORS, etc.)
      console.error("Network error:", error.message);
    } else {
      console.error("Error:", error.message);
    }
    throw error;
  }
}
```

## Abort Request (Hủy request)

```javascript
const controller = new AbortController();
const signal = controller.signal;

// Tự động hủy sau 5 giây
setTimeout(() => controller.abort(), 5000);

fetch("https://api.example.com/data", { signal })
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => {
    if (error.name === "AbortError") {
      console.log("Request was aborted");
    }
  });

// Hoặc hủy thủ công
// controller.abort();
```

## Request với Timeout

```javascript
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      signal: controller.signal,
    });
    clearTimeout(timeoutId);
    return await response.json();
  } catch (error) {
    clearTimeout(timeoutId);
    if (error.name === "AbortError") {
      throw new Error("Request timeout");
    }
    throw error;
  }
}
```

## CORS (Cross-Origin Resource Sharing)

CORS là cơ chế bảo mật của browser, ngăn chặn request từ domain khác.

```javascript
// Request với CORS
fetch("https://api.other-domain.com/data", {
  method: "GET",
  mode: "cors", // 'cors', 'no-cors', 'same-origin'
  credentials: "include", // 'include', 'same-origin', 'omit'
  headers: {
    "Content-Type": "application/json",
  },
});
```

**Các mode:**

- `cors`: Cho phép cross-origin requests (default)
- `no-cors`: Chỉ cho phép simple requests, không đọc được response
- `same-origin`: Chỉ cho phép same-origin requests

## Upload File với AJAX

```javascript
// HTML: <input type="file" id="fileInput" />

async function uploadFile() {
  const fileInput = document.getElementById("fileInput");
  const file = fileInput.files[0];

  const formData = new FormData();
  formData.append("file", file);
  formData.append("description", "My file");

  try {
    const response = await fetch("https://api.example.com/upload", {
      method: "POST",
      body: formData,
      // Không set Content-Type header, browser tự động set
    });

    const result = await response.json();
    console.log("Upload success:", result);
  } catch (error) {
    console.error("Upload failed:", error);
  }
}
```

## Upload với Progress

```javascript
function uploadWithProgress(file, url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append("file", file);

    // Progress event
    xhr.upload.addEventListener("progress", (e) => {
      if (e.lengthComputable) {
        const percent = Math.round((e.loaded / e.total) * 100);
        console.log(`Upload progress: ${percent}%`);
      }
    });

    xhr.addEventListener("load", () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });

    xhr.addEventListener("error", () => reject(new Error("Upload error")));

    xhr.open("POST", url);
    xhr.send(formData);
  });
}
```

## Retry Logic

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${i + 1} failed, retrying...`);

      // Exponential backoff
      await new Promise((resolve) => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }

  throw lastError;
}
```

## Parallel Requests

```javascript
// Chạy song song với Promise.all
async function fetchMultiple() {
  try {
    const [users, posts, comments] = await Promise.all([
      fetch("/api/users").then((r) => r.json()),
      fetch("/api/posts").then((r) => r.json()),
      fetch("/api/comments").then((r) => r.json()),
    ]);

    console.log({ users, posts, comments });
  } catch (error) {
    console.error("One of the requests failed:", error);
  }
}

// Promise.allSettled - không fail nếu 1 request lỗi
async function fetchMultipleSafe() {
  const results = await Promise.allSettled([
    fetch("/api/users").then((r) => r.json()),
    fetch("/api/posts").then((r) => r.json()),
    fetch("/api/failing-endpoint").then((r) => r.json()),
  ]);

  results.forEach((result, index) => {
    if (result.status === "fulfilled") {
      console.log(`Request ${index} succeeded:`, result.value);
    } else {
      console.log(`Request ${index} failed:`, result.reason);
    }
  });
}
```

## Caching Requests

```javascript
const cache = new Map();

async function fetchWithCache(url, ttl = 60000) {
  const cached = cache.get(url);

  if (cached && Date.now() - cached.timestamp < ttl) {
    console.log("Returning cached data");
    return cached.data;
  }

  const response = await fetch(url);
  const data = await response.json();

  cache.set(url, {
    data,
    timestamp: Date.now(),
  });

  return data;
}
```

## Interceptors Pattern

```javascript
// Tạo wrapper cho fetch với interceptors
class FetchClient {
  constructor(baseURL = "") {
    this.baseURL = baseURL;
    this.requestInterceptors = [];
    this.responseInterceptors = [];
  }

  addRequestInterceptor(fn) {
    this.requestInterceptors.push(fn);
  }

  addResponseInterceptor(fn) {
    this.responseInterceptors.push(fn);
  }

  async request(url, options = {}) {
    let config = { url: this.baseURL + url, ...options };

    // Apply request interceptors
    for (const interceptor of this.requestInterceptors) {
      config = await interceptor(config);
    }

    let response = await fetch(config.url, config);

    // Apply response interceptors
    for (const interceptor of this.responseInterceptors) {
      response = await interceptor(response);
    }

    return response;
  }
}

// Sử dụng
const client = new FetchClient("https://api.example.com");

// Thêm auth token vào mọi request
client.addRequestInterceptor((config) => {
  config.headers = {
    ...config.headers,
    Authorization: `Bearer ${localStorage.getItem("token")}`,
  };
  return config;
});

// Log response
client.addResponseInterceptor((response) => {
  console.log("Response status:", response.status);
  return response;
});
```

## So sánh XMLHttpRequest vs Fetch

| Feature        | XMLHttpRequest | Fetch                        |
| -------------- | -------------- | ---------------------------- |
| Syntax         | Phức tạp       | Đơn giản, Promise-based      |
| Progress       | Hỗ trợ         | Không hỗ trợ native          |
| Abort          | Có             | Có (AbortController)         |
| Cookies        | Tự động gửi    | Cần `credentials: 'include'` |
| Error handling | Callback       | Promise catch                |
| Streaming      | Không          | Có                           |

## Best Practices

1. **Luôn xử lý errors** - Không bao giờ bỏ qua error handling
2. **Sử dụng async/await** - Code dễ đọc hơn callbacks
3. **Implement timeout** - Tránh request treo vô hạn
4. **Validate response** - Kiểm tra status code trước khi parse
5. **Sử dụng AbortController** - Hủy request không cần thiết
6. **Cache khi có thể** - Giảm tải server và tăng performance
7. **Retry với exponential backoff** - Xử lý network không ổn định

## Thư viện phổ biến

- **Axios** - Promise-based HTTP client với nhiều tính năng
- **ky** - Tiny fetch wrapper
- **got** - HTTP request library cho Node.js
- **superagent** - Flexible HTTP library
