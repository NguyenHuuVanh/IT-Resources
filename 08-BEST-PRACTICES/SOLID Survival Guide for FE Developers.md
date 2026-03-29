# 🚀 SOLID Survival Guide cho Front-End Developer

> Đừng cố học thuộc lòng định nghĩa học thuật. Hãy nhớ SOLID thông qua các **"Dấu hiệu cảnh báo"** (Code Smells) và các **"Câu hỏi thần chú"**.

---

## 📋 Mục lục

1. [Bộ "Thần Chú" 5 Giây](#1-bộ-thần-chú-5-giây)
2. [Chi Tiết Từng Nguyên Tắc + Code Ví Dụ](#2-chi-tiết-từng-nguyên-tắc--code-ví-dụ)
3. [Quy Trình 3 Bước Để Không Quên](#3-quy-trình-3-bước-để-không-quên)
4. [Mẹo Ghi Nhớ Qua Hình Ảnh & Ví dụ Thực Tế](#4-mẹo-ghi-nhớ-qua-hình-ảnh--ví-dụ-thực-tế)
5. [Real-World Scenarios](#5-real-world-scenarios)
6. [Công cụ hỗ trợ "Nhắc bài"](#6-công-cụ-hỗ-trợ-nhắc-bài)
7. [Checklist Code Review theo SOLID](#7-checklist-code-review-theo-solid)
8. [Những Sai Lầm Phổ Biến](#8-những-sai-lầm-phổ-biến)

---

## 1. Bộ "Thần Chú" 5 Giây

Mỗi khi viết một Component, hãy tự hỏi:

| Nguyên tắc | Câu hỏi thần chú |
|---|---|
| **S** (Single) | "Component này có đang làm **quá nhiều việc** không?" _(Vừa fetch, vừa tính toán, vừa render)_ |
| **O** (Open) | "Nếu ngày mai thêm một giao diện tương tự, mình có phải **sửa code cũ** không?" |
| **L** (Liskov) | "Component này có **hành xử giống** như thẻ HTML gốc mà nó đại diện không?" |
| **I** (Interface) | "Mình có đang bắt Component nhận những **Props mà nó chẳng bao giờ dùng** tới không?" |
| **D** (Dependency) | "Logic này có bị **'dính chặt'** vào một thư viện/API cụ thể không?" |

---

## 2. Chi Tiết Từng Nguyên Tắc + Code Ví Dụ

### 🔴 S — Single Responsibility Principle (SRP)

> **"Một Component chỉ nên có MỘT lý do để thay đổi."**

#### Dấu hiệu vi phạm SRP:
- Component dài quá **150 dòng**
- Component vừa **fetch data**, vừa **xử lý logic**, vừa **render UI**
- Tên Component quá chung chung: `Dashboard`, `UserPage`, `MainContent`
- Khi sửa 1 bug UI lại ảnh hưởng đến logic nghiệp vụ

#### ❌ Anti-pattern: Component "All-in-one"

```tsx
// ❌ SAI — Component làm quá nhiều việc
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  // Fetch data — trách nhiệm 1
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [userId]);

  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(data => setPosts(data));
  }, [userId]);

  // Tính toán phức tạp — trách nhiệm 2
  const totalLikes = posts.reduce((sum, post) => sum + post.likes, 0);
  const avgLikes = posts.length ? totalLikes / posts.length : 0;
  const topPosts = posts
    .sort((a, b) => b.likes - a.likes)
    .slice(0, 5);

  // Format data — trách nhiệm 3
  const formatDate = (date: string) => {
    return new Date(date).toLocaleDateString("vi-VN");
  };

  // Render UI — trách nhiệm 4
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage msg={error} />;

  return (
    <div>
      <img src={user?.avatar} alt={user?.name} />
      <h1>{user?.name}</h1>
      <p>Tham gia: {formatDate(user?.createdAt)}</p>
      <p>Tổng likes: {totalLikes} | Trung bình: {avgLikes.toFixed(1)}</p>
      <h2>Top bài viết</h2>
      {topPosts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <span>{post.likes} likes</span>
        </div>
      ))}
    </div>
  );
}
```

#### ✅ Best practice: Tách trách nhiệm rõ ràng

```tsx
// ✅ ĐÚNG — Custom Hook cho data fetching (trách nhiệm: lấy dữ liệu)
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser)
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [userId]);

  return { user, loading, error };
}

// ✅ Custom Hook cho posts (trách nhiệm: lấy và tính toán posts)
function useUserPosts(userId: string) {
  const [posts, setPosts] = useState<Post[]>([]);

  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(setPosts);
  }, [userId]);

  const stats = useMemo(() => {
    const totalLikes = posts.reduce((sum, p) => sum + p.likes, 0);
    return {
      totalLikes,
      avgLikes: posts.length ? totalLikes / posts.length : 0,
      topPosts: [...posts].sort((a, b) => b.likes - a.likes).slice(0, 5),
    };
  }, [posts]);

  return { posts, stats };
}

// ✅ Component con: Avatar (trách nhiệm: hiển thị avatar)
function UserAvatar({ src, name }: { src: string; name: string }) {
  return <img src={src} alt={name} className="avatar" />;
}

// ✅ Component con: Stats (trách nhiệm: hiển thị thống kê)
function UserStats({ totalLikes, avgLikes }: UserStatsProps) {
  return (
    <div className="stats">
      <span>Tổng likes: {totalLikes}</span>
      <span>Trung bình: {avgLikes.toFixed(1)}</span>
    </div>
  );
}

// ✅ Component chính chỉ lo "lắp ráp" (composition)
function UserProfile({ userId }: { userId: string }) {
  const { user, loading, error } = useUser(userId);
  const { stats } = useUserPosts(userId);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage msg={error} />;
  if (!user) return null;

  return (
    <div className="profile">
      <UserAvatar src={user.avatar} name={user.name} />
      <h1>{user.name}</h1>
      <UserStats totalLikes={stats.totalLikes} avgLikes={stats.avgLikes} />
      <TopPostsList posts={stats.topPosts} />
    </div>
  );
}
```

---

### 🟠 O — Open/Closed Principle (OCP)

> **"Component nên MỞ cho mở rộng, nhưng ĐÓNG cho sửa đổi."**

#### Dấu hiệu vi phạm OCP:
- Có quá nhiều `if/else` hoặc `switch/case` trong Component để render các loại UI khác nhau
- Mỗi lần thêm tính năng mới phải sửa Component cũ
- Copy-paste code ra rồi sửa vài dòng

#### ❌ Anti-pattern: if/else chồng chất

```tsx
// ❌ SAI — Mỗi lần thêm loại Alert mới phải vào sửa Component này
function Alert({ type, message }: { type: string; message: string }) {
  if (type === "success") {
    return (
      <div style={{ background: "green", color: "white" }}>
        <span>✅</span> {message}
      </div>
    );
  } else if (type === "error") {
    return (
      <div style={{ background: "red", color: "white" }}>
        <span>❌</span> {message}
      </div>
    );
  } else if (type === "warning") {
    return (
      <div style={{ background: "orange", color: "black" }}>
        <span>⚠️</span> {message}
      </div>
    );
  }
  // Nếu thêm type "info"? Lại phải sửa file này... 😩
  return null;
}
```

#### ✅ Best practice: Composition + Configuration

```tsx
// ✅ ĐÚNG — Dùng configuration map (dễ mở rộng, không cần sửa Component)
const ALERT_CONFIG: Record<string, { bg: string; color: string; icon: string }> = {
  success: { bg: "green", color: "white", icon: "✅" },
  error:   { bg: "red",   color: "white", icon: "❌" },
  warning: { bg: "orange", color: "black", icon: "⚠️" },
  info:    { bg: "blue",  color: "white", icon: "ℹ️" },
  // Thêm loại mới? Chỉ cần thêm 1 dòng ở đây!
};

function Alert({ type, message }: { type: keyof typeof ALERT_CONFIG; message: string }) {
  const config = ALERT_CONFIG[type];
  return (
    <div style={{ background: config.bg, color: config.color }}>
      <span>{config.icon}</span> {message}
    </div>
  );
}
```

```tsx
// ✅ ĐÚNG — Dùng Composition Pattern (linh hoạt tối đa)
function AlertBase({ children, className }: { children: ReactNode; className?: string }) {
  return <div className={`alert ${className ?? ""}`}>{children}</div>;
}

// Mở rộng bằng cách TẠO MỚI, không sửa AlertBase
function SuccessAlert({ message }: { message: string }) {
  return (
    <AlertBase className="alert-success">
      <span>✅</span> {message}
    </AlertBase>
  );
}

function ErrorAlert({ message, onRetry }: { message: string; onRetry?: () => void }) {
  return (
    <AlertBase className="alert-error">
      <span>❌</span> {message}
      {onRetry && <button onClick={onRetry}>Thử lại</button>}
    </AlertBase>
  );
}
```

#### ✅ Best practice nâng cao: Render Props & Compound Components

```tsx
// ✅ Compound Components — Cho phép người dùng tự tổ hợp
function Card({ children }: { children: ReactNode }) {
  return <div className="card">{children}</div>;
}

Card.Header = ({ children }: { children: ReactNode }) => (
  <div className="card-header">{children}</div>
);

Card.Body = ({ children }: { children: ReactNode }) => (
  <div className="card-body">{children}</div>
);

Card.Footer = ({ children }: { children: ReactNode }) => (
  <div className="card-footer">{children}</div>
);

// Sử dụng: thoải mái tổ hợp, không cần sửa Card
<Card>
  <Card.Header>Tiêu đề</Card.Header>
  <Card.Body>Nội dung bất kỳ</Card.Body>
  <Card.Footer><button>Xác nhận</button></Card.Footer>
</Card>
```

---

### 🟡 L — Liskov Substitution Principle (LSP)

> **"Nếu Component con kế thừa/wrap Component cha, nó phải hoạt động đúng như Component cha."**

#### Dấu hiệu vi phạm LSP:
- Custom Input component không nhận `onChange`, `value`, `disabled`...
- Wrap một `<button>` nhưng bỏ qua `type`, `onClick`, `disabled`, `aria-label`
- Component "trông giống" một thẻ HTML nhưng hành xử khác hoàn toàn

#### ❌ Anti-pattern: Custom Input "giả"

```tsx
// ❌ SAI — Trông giống input nhưng không chấp nhận các props chuẩn
interface CustomInputProps {
  label: string;
  onUpdate: (val: string) => void; // tên khác chuẩn onChange
  // Quên: value, disabled, placeholder, name, onBlur, onFocus...
}

function CustomInput({ label, onUpdate }: CustomInputProps) {
  const [val, setVal] = useState("");

  return (
    <div>
      <label>{label}</label>
      <input
        value={val}
        onChange={(e) => {
          setVal(e.target.value);
          onUpdate(e.target.value);
        }}
      />
    </div>
  );
}

// Kết quả: Không thể dùng trong <form>, không thể validate bằng
// thư viện form (react-hook-form, formik) vì thiếu ref, name, onChange chuẩn
```

#### ✅ Best practice: Extend HTML element natively

```tsx
// ✅ ĐÚNG — Kế thừa đầy đủ props của <input> gốc
interface CustomInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
  helperText?: string;
}

const CustomInput = forwardRef<HTMLInputElement, CustomInputProps>(
  ({ label, error, helperText, className, id, ...rest }, ref) => {
    const inputId = id ?? useId();

    return (
      <div className={`input-wrapper ${error ? "has-error" : ""}`}>
        <label htmlFor={inputId}>{label}</label>
        <input
          ref={ref}
          id={inputId}
          className={`input ${className ?? ""}`}
          aria-invalid={!!error}
          aria-describedby={error ? `${inputId}-error` : undefined}
          {...rest} // Spread TẤT CẢ props còn lại: onChange, value, name, disabled, placeholder...
        />
        {error && <span id={`${inputId}-error`} className="error" role="alert">{error}</span>}
        {helperText && !error && <span className="helper">{helperText}</span>}
      </div>
    );
  }
);

// Bây giờ có thể dùng ở BẤT KỲ đâu như <input> gốc:
<CustomInput label="Email" type="email" onChange={handleChange} value={email} />
<CustomInput label="Mật khẩu" type="password" disabled={isLoading} />

// Hoàn toàn tương thích với react-hook-form:
<CustomInput label="Email" {...register("email")} error={errors.email?.message} />
```

#### ✅ Best practice: Custom Button

```tsx
// ✅ ĐÚNG — Button mở rộng hoàn toàn từ <button> gốc
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "danger" | "ghost";
  size?: "sm" | "md" | "lg";
  isLoading?: boolean;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = "primary", size = "md", isLoading, leftIcon, rightIcon,
     children, disabled, className, ...rest }, ref) => {
    return (
      <button
        ref={ref}
        className={`btn btn-${variant} btn-${size} ${className ?? ""}`}
        disabled={disabled || isLoading}
        {...rest}
      >
        {isLoading ? <Spinner size="sm" /> : leftIcon}
        {children}
        {rightIcon}
      </button>
    );
  }
);

// ✅ Có thể thay thế <button> ở bất kỳ đâu mà không lo thiếu props
<Button type="submit" onClick={handleSubmit}>Gửi</Button>
<Button variant="danger" disabled={!canDelete}>Xoá</Button>
<Button variant="ghost" aria-label="Đóng" onClick={onClose}>✕</Button>
```

---

### 🟢 I — Interface Segregation Principle (ISP)

> **"Đừng bắt Component phụ thuộc vào những Props mà nó không cần."**

#### Dấu hiệu vi phạm ISP:
- Truyền cả object lớn (User, Order, Product) chỉ để dùng 1-2 fields
- Props interface có **nhiều optional fields** mà mỗi nơi dùng chỉ cần vài cái
- Component bị **re-render** không cần thiết vì nhận quá nhiều data

#### ❌ Anti-pattern: Truyền cả object thừa

```tsx
// ❌ SAI — UserCard chỉ cần name và avatar, nhưng nhận cả User object
interface User {
  id: string;
  name: string;
  email: string;
  avatar: string;
  address: Address;
  creditCard: CreditCard;   // Dữ liệu nhạy cảm!
  loginHistory: LoginLog[];  // Data nặng, không liên quan!
  permissions: Permission[];
  preferences: Preferences;
}

function UserCard({ user }: { user: User }) {
  // Chỉ dùng có 2 fields mà nhận cả cục data to đùng 😩
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
    </div>
  );
}

// Khi dùng: phải fetch TOÀN BỘ User object chỉ để hiển thị card
// Re-render mỗi khi BẤT KỲ field nào của User thay đổi
```

#### ✅ Best practice: Props nhỏ, đúng nhu cầu

```tsx
// ✅ ĐÚNG — Chỉ nhận đúng những gì cần dùng
interface UserCardProps {
  name: string;
  avatar: string;
}

function UserCard({ name, avatar }: UserCardProps) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
    </div>
  );
}

// Dùng: chỉ truyền đúng data cần thiết
<UserCard name={user.name} avatar={user.avatar} />
```

#### ✅ Best practice nâng cao: Tách Interface theo ngữ cảnh

```tsx
// ✅ ĐÚNG — Tách Interface theo "vai trò" sử dụng
interface Identifiable {
  id: string;
}

interface Displayable {
  name: string;
  avatar: string;
}

interface Contactable {
  email: string;
  phone?: string;
}

interface Authorizable {
  permissions: Permission[];
  role: Role;
}

// Mỗi Component chỉ "ăn" interface phù hợp
function UserCard({ name, avatar }: Displayable) { /* ... */ }
function UserContact({ email, phone }: Contactable) { /* ... */ }
function UserPermissions({ permissions, role }: Authorizable) { /* ... */ }

// Component profile tổ hợp nhiều interface
interface UserProfileProps extends Identifiable, Displayable, Contactable {}
function UserProfile({ id, name, avatar, email, phone }: UserProfileProps) {
  return (
    <>
      <UserCard name={name} avatar={avatar} />
      <UserContact email={email} phone={phone} />
    </>
  );
}
```

#### ✅ Best practice: Custom Hook cũng cần ISP

```tsx
// ❌ SAI — Hook trả về quá nhiều thứ không liên quan
function useUser(id: string) {
  // ... fetch user, posts, comments, notifications, settings...
  return { user, posts, comments, notifications, settings, loading, error,
           updateUser, deleteUser, createPost, toggleNotification };
}

// ✅ ĐÚNG — Tách thành nhiều hooks nhỏ, mỗi hook 1 trách nhiệm
function useUserProfile(id: string) {
  return { user, loading, error };
}

function useUserPosts(id: string) {
  return { posts, createPost, loading };
}

function useUserNotifications(id: string) {
  return { notifications, toggleNotification, unreadCount };
}
```

---

### 🔵 D — Dependency Inversion Principle (DIP)

> **"Component/Hook không nên phụ thuộc trực tiếp vào implementation cụ thể. Hãy phụ thuộc vào abstraction (interface)."**

#### Dấu hiệu vi phạm DIP:
- `import axios from 'axios'` trực tiếp trong Component
- Hardcode `localStorage`, `fetch`, hoặc tên API endpoint trong Component
- Không thể viết **Unit Test** mà không mock module
- Thay đổi thư viện HTTP (axios → fetch → ky) phải sửa hàng chục file

#### ❌ Anti-pattern: Hardcode dependency

```tsx
// ❌ SAI — Component dính chặt vào axios và endpoint cụ thể
import axios from "axios";

function useProducts() {
  const [products, setProducts] = useState<Product[]>([]);

  useEffect(() => {
    axios.get("https://api.myshop.com/v2/products", {
      headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
    }).then(res => setProducts(res.data));
  }, []);

  return products;
}

// ❌ Vấn đề:
// 1. Muốn test? Phải mock axios VÀ localStorage
// 2. Đổi sang fetch? Phải sửa TỪNG file import axios
// 3. Đổi API URL? Phải tìm sửa ở TẤT CẢ mọi nơi
```

#### ✅ Best practice: Dependency Injection qua abstraction

```tsx
// ✅ ĐÚNG — Bước 1: Định nghĩa Interface (abstraction)
interface HttpClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: unknown): Promise<T>;
  put<T>(url: string, data: unknown): Promise<T>;
  delete(url: string): Promise<void>;
}

// ✅ Bước 2: Implementation cụ thể (có thể thay đổi bất kỳ lúc nào)
class AxiosHttpClient implements HttpClient {
  private client = axios.create({
    baseURL: import.meta.env.VITE_API_URL,
  });

  constructor() {
    this.client.interceptors.request.use((config) => {
      const token = localStorage.getItem("token");
      if (token) config.headers.Authorization = `Bearer ${token}`;
      return config;
    });
  }

  async get<T>(url: string): Promise<T> {
    const { data } = await this.client.get<T>(url);
    return data;
  }

  async post<T>(url: string, body: unknown): Promise<T> {
    const { data } = await this.client.post<T>(url, body);
    return data;
  }
  // ... put, delete tương tự
}

// ✅ Bước 3: Cung cấp qua Context (Dependency Injection ở React)
const HttpClientContext = createContext<HttpClient | null>(null);

function useHttpClient(): HttpClient {
  const client = useContext(HttpClientContext);
  if (!client) throw new Error("HttpClientProvider is missing");
  return client;
}

// ✅ Bước 4: Sử dụng trong Hook — không biết đang dùng axios hay fetch
function useProducts() {
  const http = useHttpClient();
  const [products, setProducts] = useState<Product[]>([]);

  useEffect(() => {
    http.get<Product[]>("/products").then(setProducts);
  }, [http]);

  return products;
}

// ✅ Bước 5: Trong App
<HttpClientContext.Provider value={new AxiosHttpClient()}>
  <App />
</HttpClientContext.Provider>

// ✅ Bước 6: Trong Test — dùng Mock, không cần mock axios!
const mockClient: HttpClient = {
  get: vi.fn().mockResolvedValue([{ id: 1, name: "Test Product" }]),
  post: vi.fn(),
  put: vi.fn(),
  delete: vi.fn(),
};

<HttpClientContext.Provider value={mockClient}>
  <ProductList />
</HttpClientContext.Provider>
```

#### ✅ Best practice: DIP cho Storage

```tsx
// ✅ Interface cho Storage
interface StorageService {
  get(key: string): string | null;
  set(key: string, value: string): void;
  remove(key: string): void;
}

// Implementation cho browser
const localStorageService: StorageService = {
  get: (key) => localStorage.getItem(key),
  set: (key, value) => localStorage.setItem(key, value),
  remove: (key) => localStorage.removeItem(key),
};

// Implementation cho test (in-memory)
const memoryStorageService: StorageService = (() => {
  const store = new Map<string, string>();
  return {
    get: (key) => store.get(key) ?? null,
    set: (key, value) => store.set(key, value),
    remove: (key) => store.delete(key),
  };
})();

// Inject qua Context hoặc tham số
const StorageContext = createContext<StorageService>(localStorageService);
```

---

## 3. Quy Trình 3 Bước Để Không Quên

### **Bước 1: Viết cho chạy được (Make it work)** 🏃

Đừng áp dụng SOLID ngay từ dòng code đầu tiên. Hãy cứ viết code "xấu" một chút để giải quyết xong tính năng.

> ⚠️ **Lưu ý:** "Code xấu" không có nghĩa là code không chạy. Vẫn phải đảm bảo TypeScript không lỗi, không có `any`, và có xử lý error cơ bản.

### **Bước 2: Giai đoạn Refactor (Make it right)** 🔧

Sau khi tính năng đã chạy, hãy dành **10-15 phút** nhìn lại:

| Dấu hiệu | Hành động | Nguyên tắc |
|---|---|---|
| Component dài quá **150 dòng** | Tách Custom Hook hoặc Component con | SRP |
| Quá nhiều `if/else` cho nhiều loại UI | Chuyển sang Composition / Config map | OCP |
| Custom Component thiếu props của HTML gốc | Extend `React.HTMLAttributes` + `forwardRef` | LSP |
| Truyền cả object lớn chỉ dùng 1-2 fields | Tách props nhỏ, truyền đúng data cần | ISP |
| Import trực tiếp `axios`, `localStorage`... | Wrap qua Interface + Context | DIP |

### **Bước 3: Code Review (Checklist cho team)** 📝

Khi review code cho đồng nghiệp, hãy dùng SOLID làm thước đo:

- 🔴 _"UserComponent này nhận cả object User to đùng chỉ để lấy cái tên à? →_ **ISP**_"_
- 🟠 _"Sao lại hardcode axios ở đây? →_ **DIP**_"_
- 🟡 _"Custom Select này có nhận onChange + value không? →_ **LSP**_"_
- 🟢 _"Thêm loại notification mới phải sửa file cũ? →_ **OCP**_"_
- 🔵 _"Component 300 dòng, vừa fetch vừa render? →_ **SRP**_"_

---

## 4. Mẹo Ghi Nhớ Qua Hình Ảnh & Ví dụ Thực Tế

| Nguyên tắc | Hình ảnh ẩn dụ | Ví dụ FE thực tế |
|---|---|---|
| **SRP** | 🔪 Con dao đa năng Thụy Sĩ | Đừng làm 1 Component "All-in-one". Hãy tách thành: `useUser`, `UserAvatar`, `UserBio`. |
| **OCP** | 🔌 Ổ cắm điện | Thiết kế Component như cái ổ cắm, ai muốn cắm thiết bị gì (`children`/`props`) vào cũng được mà không cần tháo ổ điện ra. |
| **LSP** | 🦆 Con vịt đồ chơi | Nếu nó trông giống `<input>`, nó phải nhận được `onChange` và `value` như một cái `<input>` thật. |
| **ISP** | 🍽️ Thực đơn chọn món (À la carte) | Chỉ gọi những gì bạn ăn. Đừng bắt Component "ăn" cả một cục dữ liệu thừa từ API. |
| **DIP** | 🔋 Sạc dự phòng | Điện thoại (UI) không nên quan tâm điện đến từ ổ cắm hay sạc dự phòng (API/Mock). Nó chỉ cần biết có "nguồn điện" cung cấp qua cổng sạc (Interface). |

---

## 5. Real-World Scenarios

### 📌 Scenario 1: Xây dựng Form phức tạp

**Vấn đề:** Form đăng ký có 15 fields, validation phức tạp, và nhiều bước.

**Áp dụng SOLID:**
- **SRP**: Tách mỗi bước thành Component riêng (`PersonalInfoStep`, `AddressStep`, `PaymentStep`)
- **OCP**: Dùng config array để định nghĩa các steps, thêm step mới không cần sửa `FormWizard`
- **LSP**: Mỗi field component (`CustomInput`, `CustomSelect`) extend HTML element gốc → tương thích `react-hook-form`
- **ISP**: Mỗi step chỉ nhận data của bước đó, không nhận toàn bộ form data
- **DIP**: Validation rules inject qua config, không hardcode trong Component

```tsx
// Config-driven form — Thêm step mới chỉ cần thêm vào array
const REGISTRATION_STEPS: FormStep[] = [
  { id: "personal", title: "Thông tin cá nhân", component: PersonalInfoStep, validation: personalSchema },
  { id: "address",  title: "Địa chỉ",          component: AddressStep,      validation: addressSchema },
  { id: "payment",  title: "Thanh toán",        component: PaymentStep,      validation: paymentSchema },
  // Thêm step mới? Chỉ cần thêm 1 dòng ở đây! (OCP)
];

function FormWizard({ steps }: { steps: FormStep[] }) {
  const [currentStep, setCurrentStep] = useState(0);
  const StepComponent = steps[currentStep].component;

  return (
    <div>
      <StepIndicator steps={steps} current={currentStep} />
      <StepComponent onNext={() => setCurrentStep(s => s + 1)} />
    </div>
  );
}
```

### 📌 Scenario 2: Data Table với nhiều loại column

**Vấn đề:** Table cần hiển thị text, số, ngày tháng, trạng thái, avatar, actions... mỗi loại render khác nhau.

**Áp dụng SOLID:**

```tsx
// ✅ OCP: Column renderers — thêm loại mới không cần sửa Table
const COLUMN_RENDERERS: Record<string, (value: any) => ReactNode> = {
  text:     (value) => <span>{value}</span>,
  number:   (value) => <span>{value.toLocaleString("vi-VN")}</span>,
  date:     (value) => <span>{new Date(value).toLocaleDateString("vi-VN")}</span>,
  currency: (value) => <span>{value.toLocaleString("vi-VN")}₫</span>,
  status:   (value) => <StatusBadge status={value} />,
  avatar:   (value) => <Avatar src={value} />,
  actions:  (value) => <ActionMenu actions={value} />,
};

// ✅ ISP: Column config chỉ cần data liên quan
interface ColumnDef<T> {
  key: keyof T;
  title: string;
  type: keyof typeof COLUMN_RENDERERS;
  sortable?: boolean;
  width?: string;
}

// Sử dụng
const columns: ColumnDef<Order>[] = [
  { key: "id",        title: "Mã đơn",   type: "text" },
  { key: "total",     title: "Tổng tiền", type: "currency", sortable: true },
  { key: "status",    title: "Trạng thái", type: "status" },
  { key: "createdAt", title: "Ngày tạo",  type: "date",    sortable: true },
];
```

### 📌 Scenario 3: Authentication với nhiều providers

```tsx
// ✅ DIP: Auth không phụ thuộc vào provider cụ thể
interface AuthProvider {
  login(credentials: unknown): Promise<AuthResult>;
  logout(): Promise<void>;
  getUser(): Promise<User | null>;
  refreshToken(): Promise<string>;
}

// Implementations
class GoogleAuthProvider implements AuthProvider { /* ... */ }
class EmailAuthProvider implements AuthProvider { /* ... */ }
class MockAuthProvider implements AuthProvider { /* ... cho testing */ }

// ✅ SRP: AuthContext chỉ quản lý trạng thái auth
const AuthContext = createContext<{
  user: User | null;
  login: (credentials: unknown) => Promise<void>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
} | null>(null);

function AuthProvider({ provider, children }: { provider: AuthProvider; children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (credentials: unknown) => {
    const result = await provider.login(credentials);
    setUser(result.user);
  };

  const logout = async () => {
    await provider.logout();
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## 6. Công cụ hỗ trợ "Nhắc bài"

### 🛠️ ESLint Rules

Cấu hình các rule để máy nhắc bạn khi vi phạm SOLID:

```json
{
  "rules": {
    "max-lines-per-function": ["warn", { "max": 150 }],
    "complexity": ["warn", { "max": 10 }],
    "max-params": ["warn", { "max": 4 }],
    "max-depth": ["warn", { "max": 3 }],
    "no-restricted-imports": ["error", {
      "patterns": [{
        "group": ["axios"],
        "message": "Dùng HttpClient interface thay vì import axios trực tiếp (DIP)"
      }]
    }]
  }
}
```

### 🛠️ TypeScript

Ép buộc bạn phải định nghĩa Interface rõ ràng, từ đó dễ nhận ra khi nào Interface quá to (**ISP**).

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noUnusedParameters": true,
    "noUnusedLocals": true
  }
}
```

### 🛠️ Storybook

Nếu bạn không thể viết một Story cho Component mà không phải truyền vào **20 cái Props giả**, đó là dấu hiệu bạn cần áp dụng SOLID.

```tsx
// ❌ Nếu Story phải mock quá nhiều thứ → Component vi phạm SRP + ISP
export const Default: Story = {
  args: {
    user: mockUser,           // object to đùng
    posts: mockPosts,         // thêm data
    comments: mockComments,   // thêm nữa
    onLike: fn(),
    onComment: fn(),
    onShare: fn(),
    onFollow: fn(),
    onBlock: fn(),
    // 20 props... 😱
  },
};

// ✅ Story đơn giản = Component tốt
export const Default: Story = {
  args: {
    name: "Nguyễn Văn A",
    avatar: "/avatar.jpg",
  },
};
```

### 🛠️ Architecture Decision Records (ADR)

Ghi lại quyết định thiết kế trong file `docs/adr/`:

```markdown
# ADR-001: Sử dụng HttpClient Interface thay vì axios trực tiếp

## Bối cảnh
Team đang import axios trực tiếp trong 47 files...

## Quyết định
Áp dụng DIP: tạo HttpClient interface, inject qua React Context.

## Hệ quả
- ✅ Dễ test (mock HttpClient thay vì mock axios)
- ✅ Đổi thư viện HTTP không ảnh hưởng business logic
- ⚠️ Team cần thời gian làm quen pattern mới
```

---

## 7. Checklist Code Review theo SOLID

Sử dụng checklist này khi review Pull Request:

### Single Responsibility (SRP)
- [ ] Component có dưới 150 dòng không?
- [ ] Component có nhiều hơn 1 `useEffect` fetch data không? → cần tách Hook
- [ ] Có thể mô tả Component bằng 1 câu ngắn không? (Nếu phải dùng "và" → tách)
- [ ] Khi sửa logic, có bị ảnh hưởng đến UI không? → cần tách

### Open/Closed (OCP)
- [ ] Có `if/else` hoặc `switch` dài để render UI khác nhau không? → dùng config map hoặc composition
- [ ] Thêm tính năng mới có phải sửa code cũ không? → cần abstract hóa
- [ ] Component có dùng `children` hoặc `render props` cho các phần linh hoạt không?

### Liskov Substitution (LSP)
- [ ] Custom form element có extend `React.HTMLAttributes` không?
- [ ] Có dùng `forwardRef` để expose ref không?
- [ ] Các props có theo naming convention của HTML gốc không? (`onChange`, `value`, `disabled`...)
- [ ] Có hỗ trợ `aria-*` attributes cho accessibility không?

### Interface Segregation (ISP)
- [ ] Có props nào không được sử dụng trong Component không?
- [ ] Có truyền object lớn khi chỉ cần 1-2 fields không?
- [ ] TypeScript interface có quá 7-8 required fields không? → cần tách
- [ ] Hook có trả về quá nhiều values không liên quan không? → cần tách

### Dependency Inversion (DIP)
- [ ] Có import trực tiếp thư viện HTTP (axios, fetch wrapper) trong Component không?
- [ ] Có hardcode API URL, localStorage key trong Component không?
- [ ] Có thể viết Unit Test mà KHÔNG cần mock module không?
- [ ] Có dùng Context/DI để inject dependencies không?

---

## 8. Những Sai Lầm Phổ Biến

### ❌ Over-engineering

> **Sai lầm #1:** Áp dụng SOLID cho MỌI thứ, kể cả Component đơn giản.

```tsx
// ❌ Quá mức — Component chỉ render static text mà tách 5 files
// Không cần tách khi component < 30 dòng và chỉ làm 1 việc đơn giản
```

> **Quy tắc:** Chỉ refactor khi Component bắt đầu "đau" (quá dài, khó test, khó hiểu). Đừng refactor "phòng ngừa".

### ❌ Premature Abstraction

> **Sai lầm #2:** Tạo abstraction khi chỉ có 1 implementation.

```tsx
// ❌ Chưa cần — Chỉ có 1 API client duy nhất, chưa cần Interface
interface HttpClient { /* ... */ }
class AxiosClient implements HttpClient { /* ... */ }
// Nếu chưa bao giờ cần mock hay đổi implementation → YAGNI

// ✅ Quy tắc: "Rule of Three"
// Khi thấy pattern lặp lại lần thứ 3 → mới tạo abstraction
```

### ❌ Tách quá nhỏ

> **Sai lầm #3:** Mỗi `div` là một Component.

```tsx
// ❌ Quá nhỏ
function UserNameText({ name }: { name: string }) {
  return <span className="username">{name}</span>;
}

// ✅ Tách ở mức có ý nghĩa — khi Component có logic hoặc được reuse
function UserBadge({ name, role, isOnline }: UserBadgeProps) {
  return (
    <div className="badge">
      <span className={`status ${isOnline ? "online" : "offline"}`} />
      <span className="name">{name}</span>
      <span className="role">{role}</span>
    </div>
  );
}
```

### Nguyên tắc vàng: **KISS + YAGNI + SOLID**

| Nguyên tắc | Khi nào áp dụng |
|---|---|
| **KISS** (Keep It Simple) | Luôn luôn — ưu tiên giải pháp đơn giản nhất |
| **YAGNI** (You Ain't Gonna Need It) | Khi muốn tạo abstraction "phòng xa" |
| **SOLID** | Khi code bắt đầu "đau" — khó đọc, khó test, khó mở rộng |

---

## 💡 Lời khuyên cuối cùng

> SOLID giống như **tập gym**. Những ngày đầu sẽ thấy mệt và viết code chậm hơn. Nhưng sau một thời gian, bạn sẽ viết code sạch một cách tự nhiên mà không cần phải nhớ tên từng nguyên tắc nữa.

**🎯 Hãy bắt đầu từ 2 nguyên tắc dễ nhất:**

1. **SRP** — Tách Hook khỏi Component (làm ngay hôm nay)
2. **ISP** — Truyền props nhỏ thay vì object lớn (làm ngay hôm nay)

Khi đã quen, mở rộng sang:

3. **OCP** — Dùng composition thay if/else
4. **LSP** — Extend HTML attributes cho custom components
5. **DIP** — Inject dependencies qua Context

> 📌 **Nhớ:** Code tốt không phải code tuân thủ 100% SOLID. Code tốt là code mà **đồng nghiệp đọc hiểu trong 30 giây** và **sửa được trong 5 phút**.
