# MVP Pattern (Model - View - Presenter)

## 1. Tổng quan

MVP phổ biến trong Android, Desktop apps. View hoàn toàn passive, Presenter làm trung gian.

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    View     │◄───►│  Presenter  │◄───►│    Model    │
│  (Passive)  │     │  (Mediator) │     │   (Data)    │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 2. Các thành phần

### Model

- Data và business logic
- Giống MVC

### View

- Hoàn toàn passive (dumb)
- Implement interface
- Chỉ hiển thị, không có logic

### Presenter

- Trung gian giữa View và Model
- Chứa presentation logic
- Gọi View methods để update UI

## 3. Ví dụ Implementation

### View Interface

```typescript
// interfaces/IUserView.ts
interface IUserView {
  showLoading(): void;
  hideLoading(): void;
  showUsers(users: User[]): void;
  showError(message: string): void;
  showSuccess(message: string): void;
  clearForm(): void;
}
```

### Model

```typescript
// models/UserModel.ts
class UserModel {
  private apiUrl = "/api/users";

  async getAll(): Promise<User[]> {
    const response = await fetch(this.apiUrl);
    return response.json();
  }

  async create(userData: CreateUserDTO): Promise<User> {
    const response = await fetch(this.apiUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
    return response.json();
  }

  async delete(id: string): Promise<void> {
    await fetch(`${this.apiUrl}/${id}`, { method: "DELETE" });
  }
}
```

### Presenter

```typescript
// presenters/UserPresenter.ts
class UserPresenter {
  private view: IUserView;
  private model: UserModel;

  constructor(view: IUserView) {
    this.view = view;
    this.model = new UserModel();
  }

  async loadUsers(): Promise<void> {
    this.view.showLoading();
    try {
      const users = await this.model.getAll();
      this.view.showUsers(users);
    } catch (error) {
      this.view.showError("Failed to load users");
    } finally {
      this.view.hideLoading();
    }
  }

  async createUser(userData: CreateUserDTO): Promise<void> {
    this.view.showLoading();
    try {
      await this.model.create(userData);
      this.view.showSuccess("User created successfully");
      this.view.clearForm();
      await this.loadUsers(); // Refresh list
    } catch (error) {
      this.view.showError("Failed to create user");
    } finally {
      this.view.hideLoading();
    }
  }

  async deleteUser(id: string): Promise<void> {
    this.view.showLoading();
    try {
      await this.model.delete(id);
      this.view.showSuccess("User deleted");
      await this.loadUsers();
    } catch (error) {
      this.view.showError("Failed to delete user");
    } finally {
      this.view.hideLoading();
    }
  }
}
```

### View Implementation

```typescript
// views/UserView.ts
class UserView implements IUserView {
  private presenter: UserPresenter;
  private loadingEl: HTMLElement;
  private userListEl: HTMLElement;
  private formEl: HTMLFormElement;
  private messageEl: HTMLElement;

  constructor() {
    this.presenter = new UserPresenter(this);
    this.initElements();
    this.bindEvents();
    this.presenter.loadUsers();
  }

  private initElements(): void {
    this.loadingEl = document.getElementById("loading")!;
    this.userListEl = document.getElementById("user-list")!;
    this.formEl = document.getElementById("user-form") as HTMLFormElement;
    this.messageEl = document.getElementById("message")!;
  }

  private bindEvents(): void {
    this.formEl.addEventListener("submit", (e) => {
      e.preventDefault();
      const formData = new FormData(this.formEl);
      this.presenter.createUser({
        name: formData.get("name") as string,
        email: formData.get("email") as string,
      });
    });
  }

  // IUserView implementation
  showLoading(): void {
    this.loadingEl.style.display = "block";
  }

  hideLoading(): void {
    this.loadingEl.style.display = "none";
  }

  showUsers(users: User[]): void {
    this.userListEl.innerHTML = users
      .map(
        (user) => `
      <div class="user-item">
        <span>${user.name} - ${user.email}</span>
        <button onclick="view.onDeleteClick('${user.id}')">Delete</button>
      </div>
    `
      )
      .join("");
  }

  showError(message: string): void {
    this.messageEl.textContent = message;
    this.messageEl.className = "error";
  }

  showSuccess(message: string): void {
    this.messageEl.textContent = message;
    this.messageEl.className = "success";
  }

  clearForm(): void {
    this.formEl.reset();
  }

  onDeleteClick(id: string): void {
    if (confirm("Are you sure?")) {
      this.presenter.deleteUser(id);
    }
  }
}

// Initialize
const view = new UserView();
```

## 4. MVP trong React

```tsx
// interfaces/IUserListView.ts
interface IUserListView {
  setUsers(users: User[]): void;
  setLoading(loading: boolean): void;
  setError(error: string | null): void;
}

// presenters/UserListPresenter.ts
class UserListPresenter {
  constructor(private view: IUserListView, private userService: UserService) {}

  async loadUsers() {
    this.view.setLoading(true);
    this.view.setError(null);
    try {
      const users = await this.userService.getAll();
      this.view.setUsers(users);
    } catch (error) {
      this.view.setError("Failed to load users");
    } finally {
      this.view.setLoading(false);
    }
  }

  async deleteUser(id: string) {
    try {
      await this.userService.delete(id);
      await this.loadUsers();
    } catch (error) {
      this.view.setError("Failed to delete user");
    }
  }
}

// components/UserList.tsx
function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // View interface implementation
  const view: IUserListView = useMemo(
    () => ({
      setUsers,
      setLoading,
      setError,
    }),
    []
  );

  const presenter = useMemo(() => new UserListPresenter(view, new UserService()), [view]);

  useEffect(() => {
    presenter.loadUsers();
  }, [presenter]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div className="error">{error}</div>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => presenter.deleteUser(user.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

## 5. So sánh MVC vs MVP vs MVVM

| Aspect              | MVC        | MVP       | MVVM              |
| ------------------- | ---------- | --------- | ----------------- |
| View                | Active     | Passive   | Passive + Binding |
| Mediator            | Controller | Presenter | ViewModel         |
| Data binding        | Manual     | Manual    | Automatic         |
| View-Model relation | Indirect   | 1:1       | 1:1               |
| Testability         | Medium     | High      | High              |
| Complexity          | Low        | Medium    | Medium-High       |

## 6. Ưu và Nhược điểm

### Ưu điểm

- ✅ View hoàn toàn passive, dễ test
- ✅ Presenter dễ unit test (mock View)
- ✅ Tách biệt rõ ràng concerns
- ✅ Phù hợp cho UI phức tạp

### Nhược điểm

- ❌ Nhiều boilerplate code
- ❌ Presenter có thể phình to
- ❌ 1:1 relationship giữa View-Presenter
- ❌ Phức tạp hơn MVC

## 7. Khi nào dùng MVP?

- ✅ Android native development
- ✅ Desktop applications
- ✅ Cần testability cao
- ✅ UI logic phức tạp
- ❌ Simple CRUD apps (dùng MVC)
- ❌ SPAs với data binding (dùng MVVM)
