# MVP Pattern (Model - View - Presenter)

## 1. Khái niệm

**MVP (Model-View-Presenter)** là mô hình kiến trúc phổ biến trong phát triển Mobile (Android) và Desktop applications. Đặc điểm nổi bật là View hoàn toàn "passive" (thụ động), không chứa bất kỳ logic nào.

### MVP khác MVC như thế nào?

| MVC                      | MVP                           |
| ------------------------ | ----------------------------- |
| View có thể gọi Model    | View không biết Model         |
| Controller xử lý request | Presenter điều khiển View     |
| View có thể có logic     | View hoàn toàn passive        |
| Loose coupling           | Tight coupling View-Presenter |

### Tại sao cần MVP?

**Vấn đề với MVC:**

- View có thể chứa logic → khó test
- View biết về Model → coupling cao
- Khó mock View khi unit test

**MVP giải quyết:**

- View chỉ implement interface → dễ mock
- Tất cả logic ở Presenter → dễ unit test
- View không biết Model → decoupled

## 2. Các thành phần

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    View     │◄───►│  Presenter  │◄───►│    Model    │
│  (Passive)  │     │  (Mediator) │     │   (Data)    │
└─────────────┘     └─────────────┘     └─────────────┘
     │                    │
     │    implements      │
     ▼                    │
┌─────────────┐           │
│ IView       │◄──────────┘
│ (Interface) │   uses interface
└─────────────┘
```

### Model - "Nguồn dữ liệu"

**Khái niệm:** Giống MVC, Model quản lý data và business logic.

**Nhiệm vụ:**

- Fetch/Save data
- Business logic
- Không biết về View và Presenter

```javascript
// models/UserModel.js
class UserModel {
  constructor(apiClient) {
    this.apiClient = apiClient;
  }

  async getAll() {
    const response = await this.apiClient.get("/users");
    return response.data;
  }

  async getById(id) {
    const response = await this.apiClient.get(`/users/${id}`);
    return response.data;
  }

  async create(userData) {
    const response = await this.apiClient.post("/users", userData);
    return response.data;
  }

  async update(id, userData) {
    const response = await this.apiClient.put(`/users/${id}`, userData);
    return response.data;
  }

  async delete(id) {
    await this.apiClient.delete(`/users/${id}`);
  }
}

module.exports = UserModel;
```

### View Interface - "Hợp đồng với View"

**Khái niệm:** Interface định nghĩa các methods mà View phải implement. Presenter chỉ giao tiếp với View qua interface này.

**Tại sao cần Interface?**

- Presenter không phụ thuộc vào View cụ thể
- Dễ mock View khi unit test Presenter
- Có thể swap View implementations

**Tác dụng:**

- Decoupling Presenter và View
- Contract rõ ràng
- Testability cao

```typescript
// interfaces/IUserListView.ts
interface IUserListView {
  // Hiển thị loading
  showLoading(): void;
  hideLoading(): void;

  // Hiển thị data
  showUsers(users: User[]): void;
  showUserDetail(user: User): void;

  // Hiển thị messages
  showError(message: string): void;
  showSuccess(message: string): void;

  // Form actions
  clearForm(): void;
  getFormData(): UserFormData;

  // Navigation
  navigateToEdit(userId: string): void;
  navigateToList(): void;
}
```

### Presenter - "Bộ não điều khiển"

**Khái niệm:** Presenter là trung gian giữa View và Model, chứa toàn bộ presentation logic.

**Nhiệm vụ:**

- Nhận events từ View
- Gọi Model để lấy/xử lý data
- Gọi View methods để update UI
- Chứa presentation logic

**Đặc điểm quan trọng:**

- Presenter giữ reference đến View (qua interface)
- Presenter gọi View methods để update UI
- View không tự update, chờ Presenter gọi

```javascript
// presenters/UserListPresenter.js
class UserListPresenter {
  constructor(view, userModel) {
    this.view = view; // IUserListView
    this.model = userModel; // UserModel
  }

  // Được gọi khi View khởi tạo
  async onViewReady() {
    await this.loadUsers();
  }

  // Load danh sách users
  async loadUsers() {
    // 1. Báo View hiển thị loading
    this.view.showLoading();

    try {
      // 2. Gọi Model lấy data
      const users = await this.model.getAll();

      // 3. Báo View hiển thị data
      this.view.showUsers(users);
    } catch (error) {
      // 4. Báo View hiển thị lỗi
      this.view.showError("Không thể tải danh sách người dùng");
      console.error("Load users error:", error);
    } finally {
      // 5. Báo View ẩn loading
      this.view.hideLoading();
    }
  }

  // Xử lý khi user click "Thêm"
  async onAddUserClicked() {
    // Lấy data từ form qua View
    const formData = this.view.getFormData();

    // Validate
    if (!this.validateForm(formData)) {
      this.view.showError("Vui lòng điền đầy đủ thông tin");
      return;
    }

    this.view.showLoading();

    try {
      await this.model.create(formData);

      this.view.showSuccess("Thêm người dùng thành công!");
      this.view.clearForm();

      // Reload list
      await this.loadUsers();
    } catch (error) {
      this.view.showError("Không thể thêm người dùng: " + error.message);
    } finally {
      this.view.hideLoading();
    }
  }

  // Xử lý khi user click "Xóa"
  async onDeleteUserClicked(userId) {
    this.view.showLoading();

    try {
      await this.model.delete(userId);

      this.view.showSuccess("Đã xóa người dùng");
      await this.loadUsers();
    } catch (error) {
      this.view.showError("Không thể xóa người dùng");
    } finally {
      this.view.hideLoading();
    }
  }

  // Xử lý khi user click "Sửa"
  onEditUserClicked(userId) {
    this.view.navigateToEdit(userId);
  }

  // Validate form
  validateForm(formData) {
    if (!formData.name || formData.name.trim() === "") {
      return false;
    }
    if (!formData.email || !formData.email.includes("@")) {
      return false;
    }
    return true;
  }
}

module.exports = UserListPresenter;
```

### View - "Giao diện thụ động"

**Khái niệm:** View implement interface và chỉ làm đúng những gì Presenter yêu cầu.

**Nhiệm vụ:**

- Implement IView interface
- Hiển thị UI
- Forward events đến Presenter
- KHÔNG chứa logic

**Đặc điểm quan trọng:**

- View là "dumb" - không có logic
- Chỉ biết cách hiển thị
- Delegate mọi thứ cho Presenter

```javascript
// views/UserListView.js
class UserListView {
  constructor() {
    // Khởi tạo Presenter với View này
    this.presenter = new UserListPresenter(this, new UserModel(apiClient));

    // Bind DOM elements
    this.loadingEl = document.getElementById("loading");
    this.userListEl = document.getElementById("user-list");
    this.formEl = document.getElementById("user-form");
    this.messageEl = document.getElementById("message");

    // Bind events
    this.bindEvents();

    // Báo Presenter là View đã ready
    this.presenter.onViewReady();
  }

  bindEvents() {
    // Form submit → delegate to Presenter
    this.formEl.addEventListener("submit", (e) => {
      e.preventDefault();
      this.presenter.onAddUserClicked();
    });
  }

  // ========== IMPLEMENT IUserListView ==========

  showLoading() {
    this.loadingEl.style.display = "block";
  }

  hideLoading() {
    this.loadingEl.style.display = "none";
  }

  showUsers(users) {
    this.userListEl.innerHTML = users
      .map(
        (user) => `
      <div class="user-item" data-id="${user.id}">
        <span class="user-name">${user.name}</span>
        <span class="user-email">${user.email}</span>
        <button class="btn-edit" onclick="view.onEditClick('${user.id}')">
          Sửa
        </button>
        <button class="btn-delete" onclick="view.onDeleteClick('${user.id}')">
          Xóa
        </button>
      </div>
    `
      )
      .join("");
  }

  showError(message) {
    this.messageEl.textContent = message;
    this.messageEl.className = "message error";
    this.messageEl.style.display = "block";

    setTimeout(() => {
      this.messageEl.style.display = "none";
    }, 3000);
  }

  showSuccess(message) {
    this.messageEl.textContent = message;
    this.messageEl.className = "message success";
    this.messageEl.style.display = "block";

    setTimeout(() => {
      this.messageEl.style.display = "none";
    }, 3000);
  }

  clearForm() {
    this.formEl.reset();
  }

  getFormData() {
    const formData = new FormData(this.formEl);
    return {
      name: formData.get("name"),
      email: formData.get("email"),
    };
  }

  navigateToEdit(userId) {
    window.location.href = `/users/${userId}/edit`;
  }

  // Event handlers - delegate to Presenter
  onEditClick(userId) {
    this.presenter.onEditUserClicked(userId);
  }

  onDeleteClick(userId) {
    if (confirm("Bạn có chắc muốn xóa?")) {
      this.presenter.onDeleteUserClicked(userId);
    }
  }
}

// Initialize
const view = new UserListView();
```

## 3. Luồng hoạt động

```
1. User click "Thêm" button
   │
   ▼
2. View.onSubmit() → gọi Presenter.onAddUserClicked()
   │
   ▼
3. Presenter:
   - Gọi View.getFormData() lấy data
   - Validate data
   - Gọi View.showLoading()
   - Gọi Model.create(data)
   │
   ▼
4. Model: Gọi API, trả về result
   │
   ▼
5. Presenter:
   - Gọi View.hideLoading()
   - Gọi View.showSuccess() hoặc View.showError()
   - Gọi View.clearForm()
   - Gọi loadUsers() để refresh list
   │
   ▼
6. View: Hiển thị UI theo yêu cầu của Presenter
```

## 4. Unit Testing Presenter

**Đây là lợi ích lớn nhất của MVP** - có thể test Presenter mà không cần DOM thật.

```javascript
// __tests__/UserListPresenter.test.js
describe("UserListPresenter", () => {
  let presenter;
  let mockView;
  let mockModel;

  beforeEach(() => {
    // Mock View - implement interface
    mockView = {
      showLoading: jest.fn(),
      hideLoading: jest.fn(),
      showUsers: jest.fn(),
      showError: jest.fn(),
      showSuccess: jest.fn(),
      clearForm: jest.fn(),
      getFormData: jest.fn(),
      navigateToEdit: jest.fn(),
    };

    // Mock Model
    mockModel = {
      getAll: jest.fn(),
      create: jest.fn(),
      delete: jest.fn(),
    };

    presenter = new UserListPresenter(mockView, mockModel);
  });

  describe("loadUsers", () => {
    it("should show loading, fetch users, then show users", async () => {
      // Arrange
      const users = [
        { id: "1", name: "John", email: "john@test.com" },
        { id: "2", name: "Jane", email: "jane@test.com" },
      ];
      mockModel.getAll.mockResolvedValue(users);

      // Act
      await presenter.loadUsers();

      // Assert
      expect(mockView.showLoading).toHaveBeenCalled();
      expect(mockModel.getAll).toHaveBeenCalled();
      expect(mockView.showUsers).toHaveBeenCalledWith(users);
      expect(mockView.hideLoading).toHaveBeenCalled();
    });

    it("should show error when fetch fails", async () => {
      // Arrange
      mockModel.getAll.mockRejectedValue(new Error("Network error"));

      // Act
      await presenter.loadUsers();

      // Assert
      expect(mockView.showLoading).toHaveBeenCalled();
      expect(mockView.showError).toHaveBeenCalledWith("Không thể tải danh sách người dùng");
      expect(mockView.hideLoading).toHaveBeenCalled();
      expect(mockView.showUsers).not.toHaveBeenCalled();
    });
  });

  describe("onAddUserClicked", () => {
    it("should create user and refresh list on success", async () => {
      // Arrange
      const formData = { name: "New User", email: "new@test.com" };
      const createdUser = { id: "3", ...formData };

      mockView.getFormData.mockReturnValue(formData);
      mockModel.create.mockResolvedValue(createdUser);
      mockModel.getAll.mockResolvedValue([createdUser]);

      // Act
      await presenter.onAddUserClicked();

      // Assert
      expect(mockView.getFormData).toHaveBeenCalled();
      expect(mockModel.create).toHaveBeenCalledWith(formData);
      expect(mockView.showSuccess).toHaveBeenCalledWith("Thêm người dùng thành công!");
      expect(mockView.clearForm).toHaveBeenCalled();
    });

    it("should show error when form is invalid", async () => {
      // Arrange
      mockView.getFormData.mockReturnValue({ name: "", email: "" });

      // Act
      await presenter.onAddUserClicked();

      // Assert
      expect(mockView.showError).toHaveBeenCalledWith("Vui lòng điền đầy đủ thông tin");
      expect(mockModel.create).not.toHaveBeenCalled();
    });

    it("should show error when create fails", async () => {
      // Arrange
      mockView.getFormData.mockReturnValue({
        name: "Test",
        email: "test@test.com",
      });
      mockModel.create.mockRejectedValue(new Error("Email exists"));

      // Act
      await presenter.onAddUserClicked();

      // Assert
      expect(mockView.showError).toHaveBeenCalledWith("Không thể thêm người dùng: Email exists");
    });
  });

  describe("onDeleteUserClicked", () => {
    it("should delete user and refresh list", async () => {
      // Arrange
      mockModel.delete.mockResolvedValue();
      mockModel.getAll.mockResolvedValue([]);

      // Act
      await presenter.onDeleteUserClicked("123");

      // Assert
      expect(mockModel.delete).toHaveBeenCalledWith("123");
      expect(mockView.showSuccess).toHaveBeenCalledWith("Đã xóa người dùng");
    });
  });
});
```

## 5. So sánh MVC vs MVP vs MVVM

| Khía cạnh        | MVC             | MVP                   | MVVM                |
| ---------------- | --------------- | --------------------- | ------------------- |
| **View**         | Active          | Passive               | Passive + Binding   |
| **Mediator**     | Controller      | Presenter             | ViewModel           |
| **View-Model**   | View biết Model | View không biết Model | View bind ViewModel |
| **Data binding** | Manual          | Manual                | Automatic           |
| **Testability**  | Medium          | High                  | High                |
| **Complexity**   | Low             | Medium                | Medium-High         |
| **Use case**     | Web apps        | Mobile, Desktop       | Frontend SPAs       |

## 6. Ưu điểm của MVP

| Ưu điểm              | Giải thích                             |
| -------------------- | -------------------------------------- |
| **High Testability** | Presenter dễ unit test với mock View   |
| **Separation**       | View hoàn toàn passive, không có logic |
| **Decoupling**       | View và Model không biết nhau          |
| **Clear contract**   | Interface định nghĩa rõ ràng           |
| **Maintainable**     | Logic tập trung ở Presenter            |

## 7. Nhược điểm của MVP

| Nhược điểm           | Giải thích                                 |
| -------------------- | ------------------------------------------ |
| **Boilerplate**      | Cần viết nhiều code (interface, presenter) |
| **1:1 relationship** | Mỗi View cần 1 Presenter riêng             |
| **Presenter bloat**  | Presenter có thể phình to                  |
| **Manual updates**   | Phải gọi View methods manually             |
| **Complexity**       | Phức tạp hơn MVC                           |

## 8. Khi nào dùng MVP?

### ✅ Phù hợp khi:

- Android native development
- Desktop applications (WinForms, WPF)
- Cần unit test UI logic
- UI phức tạp với nhiều states
- Team cần clear separation

### ❌ Không phù hợp khi:

- Simple CRUD apps → dùng MVC
- SPAs với data binding → dùng MVVM
- Prototype nhanh
- Small projects

## 9. MVP trong React

```jsx
// interfaces/IUserListView.ts
interface IUserListView {
  setUsers(users: User[]): void;
  setLoading(loading: boolean): void;
  setError(error: string | null): void;
}

// presenters/UserListPresenter.ts
class UserListPresenter {
  constructor(private view: IUserListView, private model: UserModel) {}

  async loadUsers() {
    this.view.setLoading(true);
    this.view.setError(null);

    try {
      const users = await this.model.getAll();
      this.view.setUsers(users);
    } catch (error) {
      this.view.setError('Không thể tải danh sách');
    } finally {
      this.view.setLoading(false);
    }
  }

  async deleteUser(id: string) {
    try {
      await this.model.delete(id);
      await this.loadUsers();
    } catch (error) {
      this.view.setError('Không thể xóa');
    }
  }
}

// components/UserList.tsx
function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // View interface implementation
  const view: IUserListView = useMemo(() => ({
    setUsers,
    setLoading,
    setError
  }), []);

  // Create presenter
  const presenter = useMemo(
    () => new UserListPresenter(view, new UserModel()),
    [view]
  );

  useEffect(() => {
    presenter.loadUsers();
  }, [presenter]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div className="error">{error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => presenter.deleteUser(user.id)}>
            Xóa
          </button>
        </li>
      ))}
    </ul>
  );
}
```

## 10. Tổng kết

**MVP là gì?** Pattern với View passive, Presenter điều khiển mọi thứ qua interface.

**Tại sao dùng?** Testability cao, View không có logic, decoupling tốt.

**Khi nào dùng?** Mobile/Desktop apps, cần unit test UI logic.

**Nhớ:**

- Model: Data layer
- View: Passive, implement interface
- Presenter: Điều khiển View qua interface
- Interface: Contract giữa View và Presenter
- Test: Mock View interface để test Presenter
