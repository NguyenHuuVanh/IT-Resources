# MVVM Pattern (Model - View - ViewModel)

## 1. Khái niệm

**MVVM (Model-View-ViewModel)** là mô hình kiến trúc phổ biến trong phát triển Frontend, đặc biệt với các framework như Vue.js, Angular, và React (kết hợp state management).

### MVVM khác MVC như thế nào?

| MVC                  | MVVM                    |
| -------------------- | ----------------------- |
| Controller điều phối | ViewModel quản lý state |
| View gọi Controller  | View bind với ViewModel |
| One-way data flow    | Two-way data binding    |
| Phổ biến ở Backend   | Phổ biến ở Frontend     |

### Tại sao cần MVVM?

Trong ứng dụng Frontend hiện đại:

- UI phức tạp với nhiều state
- Cần cập nhật UI realtime khi data thay đổi
- User tương tác nhiều (forms, clicks, drag-drop...)

**Vấn đề với MVC ở Frontend:**

- Phải manually update DOM khi data thay đổi
- Phải manually lấy data từ DOM khi user nhập
- Code dài dòng, dễ bug

**MVVM giải quyết bằng Two-way Data Binding:**

- Data thay đổi → UI tự động cập nhật
- User nhập → Data tự động cập nhật

## 2. Các thành phần

```
┌─────────────┐     Two-way      ┌─────────────┐     ┌─────────────┐
│    View     │◄═══════════════►│  ViewModel  │◄───►│    Model    │
│  (Template) │     Binding      │   (State)   │     │   (Data)    │
└─────────────┘                  └─────────────┘     └─────────────┘
```

### Model - "Nguồn dữ liệu"

**Khái niệm:** Model đại diện cho data và business logic, thường là API calls hoặc local storage.

**Nhiệm vụ:**

- Fetch data từ API
- Lưu data xuống server
- Transform data
- Không biết về View và ViewModel

**Tác dụng:**

- Tách biệt data layer
- Dễ mock khi testing
- Có thể reuse ở nhiều ViewModel

```javascript
// models/userModel.js
const API_URL = "/api/users";

export const userModel = {
  // Lấy tất cả users
  async getAll() {
    const response = await fetch(API_URL);
    if (!response.ok) {
      throw new Error("Failed to fetch users");
    }
    return response.json();
  },

  // Lấy user theo ID
  async getById(id) {
    const response = await fetch(`${API_URL}/${id}`);
    if (!response.ok) {
      throw new Error("User not found");
    }
    return response.json();
  },

  // Tạo user mới
  async create(userData) {
    const response = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }
    return response.json();
  },

  // Cập nhật user
  async update(id, userData) {
    const response = await fetch(`${API_URL}/${id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
    if (!response.ok) {
      throw new Error("Failed to update user");
    }
    return response.json();
  },

  // Xóa user
  async delete(id) {
    const response = await fetch(`${API_URL}/${id}`, {
      method: "DELETE",
    });
    if (!response.ok) {
      throw new Error("Failed to delete user");
    }
  },
};
```

### ViewModel - "Trạng thái và Logic hiển thị"

**Khái niệm:** ViewModel là lớp trung gian chứa state của UI và logic xử lý cho View.

**Nhiệm vụ:**

- Quản lý state (data hiển thị trên UI)
- Chứa computed properties (data được tính toán)
- Xử lý user actions (methods)
- Gọi Model để fetch/save data
- Expose data cho View qua binding

**Tác dụng:**

- Tách logic khỏi View
- State management tập trung
- Dễ unit test (không cần DOM)
- Reactive updates

**Đặc điểm quan trọng:**

- ViewModel KHÔNG biết về View cụ thể
- Chỉ expose data và methods
- View tự bind vào ViewModel

### View - "Giao diện người dùng"

**Khái niệm:** View là template hiển thị UI, bind với ViewModel.

**Nhiệm vụ:**

- Hiển thị data từ ViewModel
- Bind events đến ViewModel methods
- Two-way binding cho form inputs
- Không chứa logic

**Tác dụng:**

- UI thuần túy, dễ đọc
- Designer có thể làm việc độc lập
- Tự động cập nhật khi state thay đổi

## 3. Two-way Data Binding là gì?

**One-way binding (MVC):**

```javascript
// Data → View: Manual update
document.getElementById("name").textContent = user.name;

// View → Data: Manual get
const name = document.getElementById("input").value;
```

**Two-way binding (MVVM):**

```vue
<!-- Data ↔ View: Automatic -->
<input v-model="user.name">

<!-- Khi user gõ → user.name tự động cập nhật -->
<!-- Khi user.name thay đổi → input tự động hiển thị -->
```

## 4. Ví dụ với Vue.js (Options API)

```vue
<!-- UserList.vue -->
<template>
  <!-- VIEW: Chỉ hiển thị và bind events -->
  <div class="user-management">
    <h1>Quản lý người dùng</h1>

    <!-- Search với two-way binding -->
    <div class="search-box">
      <input v-model="searchQuery" placeholder="Tìm kiếm theo tên hoặc email..." class="search-input" />
      <span class="result-count"> Hiển thị {{ filteredUsers.length }} / {{ users.length }} người dùng </span>
    </div>

    <!-- Loading state -->
    <div v-if="loading" class="loading">
      <span class="spinner"></span>
      Đang tải...
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="error">
      <p>{{ error }}</p>
      <button @click="fetchUsers">Thử lại</button>
    </div>

    <!-- User list -->
    <div v-else>
      <table class="user-table">
        <thead>
          <tr>
            <th>Tên</th>
            <th>Email</th>
            <th>Trạng thái</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in filteredUsers" :key="user.id">
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>
              <span :class="['status', user.active ? 'active' : 'inactive']">
                {{ user.active ? "Hoạt động" : "Ngừng" }}
              </span>
            </td>
            <td>
              <button @click="editUser(user)" class="btn-edit">Sửa</button>
              <button @click="confirmDelete(user)" class="btn-delete">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>

      <p v-if="filteredUsers.length === 0" class="no-data">Không tìm thấy người dùng nào.</p>
    </div>

    <!-- Add user form -->
    <div class="add-form">
      <h2>Thêm người dùng mới</h2>
      <form @submit.prevent="addUser">
        <div class="form-group">
          <label>Tên:</label>
          <input v-model="newUser.name" required />
        </div>
        <div class="form-group">
          <label>Email:</label>
          <input v-model="newUser.email" type="email" required />
        </div>
        <button type="submit" :disabled="isSubmitting">
          {{ isSubmitting ? "Đang thêm..." : "Thêm người dùng" }}
        </button>
      </form>
    </div>
  </div>
</template>

<script>
import { userModel } from "@/models/userModel";

export default {
  name: "UserList",

  // ========== VIEWMODEL ==========

  // State - Dữ liệu của component
  data() {
    return {
      users: [], // Danh sách users
      loading: false, // Trạng thái loading
      error: null, // Thông báo lỗi
      searchQuery: "", // Từ khóa tìm kiếm
      isSubmitting: false, // Trạng thái submit form
      newUser: {
        // Data form thêm mới
        name: "",
        email: "",
      },
    };
  },

  // Computed - Dữ liệu được tính toán từ state
  // Tự động cập nhật khi dependencies thay đổi
  computed: {
    // Lọc users theo search query
    filteredUsers() {
      if (!this.searchQuery.trim()) {
        return this.users;
      }

      const query = this.searchQuery.toLowerCase();
      return this.users.filter(
        (user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query)
      );
    },

    // Đếm users active
    activeUsersCount() {
      return this.users.filter((user) => user.active).length;
    },

    // Check form valid
    isFormValid() {
      return this.newUser.name.trim() && this.newUser.email.trim() && this.newUser.email.includes("@");
    },
  },

  // Methods - Xử lý user actions
  methods: {
    // Fetch users từ API
    async fetchUsers() {
      this.loading = true;
      this.error = null;

      try {
        this.users = await userModel.getAll();
      } catch (err) {
        this.error = "Không thể tải danh sách người dùng. " + err.message;
        console.error("Fetch users error:", err);
      } finally {
        this.loading = false;
      }
    },

    // Thêm user mới
    async addUser() {
      if (!this.isFormValid || this.isSubmitting) return;

      this.isSubmitting = true;

      try {
        const user = await userModel.create(this.newUser);

        // Thêm vào danh sách local
        this.users.push(user);

        // Reset form
        this.newUser = { name: "", email: "" };

        // Thông báo thành công
        this.$toast.success("Thêm người dùng thành công!");
      } catch (err) {
        this.error = "Không thể thêm người dùng: " + err.message;
      } finally {
        this.isSubmitting = false;
      }
    },

    // Chuyển đến trang edit
    editUser(user) {
      this.$router.push(`/users/${user.id}/edit`);
    },

    // Xác nhận xóa
    async confirmDelete(user) {
      const confirmed = await this.$confirm(`Bạn có chắc muốn xóa "${user.name}"?`);

      if (confirmed) {
        await this.deleteUser(user.id);
      }
    },

    // Xóa user
    async deleteUser(id) {
      try {
        await userModel.delete(id);

        // Xóa khỏi danh sách local
        this.users = this.users.filter((u) => u.id !== id);

        this.$toast.success("Đã xóa người dùng");
      } catch (err) {
        this.error = "Không thể xóa người dùng: " + err.message;
      }
    },
  },

  // Lifecycle - Gọi khi component được mount
  mounted() {
    this.fetchUsers();
  },
};
</script>
```

## 5. Ví dụ với Vue 3 Composition API

```vue
<script setup>
import { ref, computed, onMounted } from "vue";
import { userModel } from "@/models/userModel";

// ========== STATE ==========
const users = ref([]);
const loading = ref(false);
const error = ref(null);
const searchQuery = ref("");
const isSubmitting = ref(false);
const newUser = ref({ name: "", email: "" });

// ========== COMPUTED ==========
const filteredUsers = computed(() => {
  if (!searchQuery.value.trim()) {
    return users.value;
  }

  const query = searchQuery.value.toLowerCase();
  return users.value.filter(
    (user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query)
  );
});

const activeUsersCount = computed(() => {
  return users.value.filter((user) => user.active).length;
});

// ========== METHODS ==========
async function fetchUsers() {
  loading.value = true;
  error.value = null;

  try {
    users.value = await userModel.getAll();
  } catch (err) {
    error.value = "Không thể tải danh sách: " + err.message;
  } finally {
    loading.value = false;
  }
}

async function addUser() {
  if (isSubmitting.value) return;

  isSubmitting.value = true;

  try {
    const user = await userModel.create(newUser.value);
    users.value.push(user);
    newUser.value = { name: "", email: "" };
  } catch (err) {
    error.value = "Không thể thêm: " + err.message;
  } finally {
    isSubmitting.value = false;
  }
}

async function deleteUser(id) {
  try {
    await userModel.delete(id);
    users.value = users.value.filter((u) => u.id !== id);
  } catch (err) {
    error.value = "Không thể xóa: " + err.message;
  }
}

// ========== LIFECYCLE ==========
onMounted(fetchUsers);
</script>

<template>
  <!-- Template giống như trên -->
</template>
```

## 6. Ví dụ với React + Hooks

Trong React, ViewModel được implement qua Custom Hooks:

```jsx
// hooks/useUsers.js - ViewModel as Custom Hook
import { useState, useEffect, useMemo, useCallback } from "react";
import { userModel } from "../models/userModel";

export function useUsers() {
  // ========== STATE ==========
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");

  // ========== COMPUTED (useMemo) ==========
  const filteredUsers = useMemo(() => {
    if (!searchQuery.trim()) return users;

    const query = searchQuery.toLowerCase();
    return users.filter((user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query));
  }, [users, searchQuery]);

  const activeUsersCount = useMemo(() => {
    return users.filter((user) => user.active).length;
  }, [users]);

  // ========== METHODS (useCallback) ==========
  const fetchUsers = useCallback(async () => {
    setLoading(true);
    setError(null);

    try {
      const data = await userModel.getAll();
      setUsers(data);
    } catch (err) {
      setError("Không thể tải danh sách: " + err.message);
    } finally {
      setLoading(false);
    }
  }, []);

  const addUser = useCallback(async (userData) => {
    const user = await userModel.create(userData);
    setUsers((prev) => [...prev, user]);
    return user;
  }, []);

  const deleteUser = useCallback(async (id) => {
    await userModel.delete(id);
    setUsers((prev) => prev.filter((u) => u.id !== id));
  }, []);

  // ========== LIFECYCLE (useEffect) ==========
  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  // ========== RETURN (Expose to View) ==========
  return {
    // State
    users,
    filteredUsers,
    loading,
    error,
    searchQuery,
    activeUsersCount,

    // Actions
    setSearchQuery,
    addUser,
    deleteUser,
    refetch: fetchUsers,
  };
}

// ========================================
// UserList.jsx - VIEW
// ========================================
import { useState } from "react";
import { useUsers } from "../hooks/useUsers";

function UserList() {
  // Sử dụng ViewModel
  const { filteredUsers, loading, error, searchQuery, setSearchQuery, addUser, deleteUser } = useUsers();

  // Local state cho form
  const [newUser, setNewUser] = useState({ name: "", email: "" });
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Handle submit form
  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      await addUser(newUser);
      setNewUser({ name: "", email: "" });
    } catch (err) {
      alert("Lỗi: " + err.message);
    } finally {
      setIsSubmitting(false);
    }
  };

  // Handle delete
  const handleDelete = async (user) => {
    if (window.confirm(`Xóa "${user.name}"?`)) {
      try {
        await deleteUser(user.id);
      } catch (err) {
        alert("Lỗi: " + err.message);
      }
    }
  };

  // ========== RENDER (VIEW) ==========
  if (loading) return <div className="loading">Đang tải...</div>;
  if (error) return <div className="error">{error}</div>;

  return (
    <div className="user-management">
      <h1>Quản lý người dùng</h1>

      {/* Search */}
      <input
        type="text"
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Tìm kiếm..."
        className="search-input"
      />

      {/* User list */}
      <table className="user-table">
        <thead>
          <tr>
            <th>Tên</th>
            <th>Email</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          {filteredUsers.map((user) => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>
                <button onClick={() => handleDelete(user)}>Xóa</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      {filteredUsers.length === 0 && <p className="no-data">Không có dữ liệu</p>}

      {/* Add form */}
      <form onSubmit={handleSubmit} className="add-form">
        <h2>Thêm người dùng</h2>
        <input
          value={newUser.name}
          onChange={(e) => setNewUser({ ...newUser, name: e.target.value })}
          placeholder="Tên"
          required
        />
        <input
          value={newUser.email}
          onChange={(e) => setNewUser({ ...newUser, email: e.target.value })}
          placeholder="Email"
          type="email"
          required
        />
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Đang thêm..." : "Thêm"}
        </button>
      </form>
    </div>
  );
}

export default UserList;
```

## 7. So sánh MVC vs MVVM

| Khía cạnh      | MVC                      | MVVM                        |
| -------------- | ------------------------ | --------------------------- |
| **Data flow**  | One-way                  | Two-way binding             |
| **View**       | Active, gọi Controller   | Passive, bind với ViewModel |
| **Mediator**   | Controller xử lý request | ViewModel quản lý state     |
| **Update UI**  | Manual                   | Automatic (reactive)        |
| **Testing**    | Test Controller          | Test ViewModel              |
| **Use case**   | Server-side, APIs        | Frontend SPAs               |
| **Complexity** | Đơn giản hơn             | Phức tạp hơn                |

## 8. Ưu điểm của MVVM

| Ưu điểm             | Giải thích                                      |
| ------------------- | ----------------------------------------------- |
| **Two-way binding** | Giảm boilerplate code, UI tự động sync với data |
| **Reactive**        | Data thay đổi → UI tự động cập nhật             |
| **Testable**        | ViewModel không phụ thuộc DOM, dễ unit test     |
| **Separation**      | View chỉ hiển thị, logic ở ViewModel            |
| **Reusable**        | ViewModel có thể dùng cho nhiều Views           |

## 9. Nhược điểm của MVVM

| Nhược điểm           | Giải thích                                     |
| -------------------- | ---------------------------------------------- |
| **Debugging khó**    | Data flow phức tạp, khó trace                  |
| **Performance**      | Two-way binding có thể chậm với large datasets |
| **Learning curve**   | Cần hiểu reactivity system                     |
| **Over-engineering** | Overkill cho simple apps                       |
| **Memory**           | Watchers/observers tốn memory                  |

## 10. Khi nào dùng MVVM?

### ✅ Phù hợp khi:

- Single Page Applications (SPAs)
- UI phức tạp với nhiều state
- Real-time updates (chat, notifications)
- Forms với validation phức tạp
- Dashboard với nhiều widgets
- Vue.js, Angular, React apps

### ❌ Không phù hợp khi:

- Static websites
- Server-rendered pages
- Simple landing pages
- Apps không cần reactive UI

## 11. Best Practices

### 1. Tách ViewModel thành Custom Hooks/Composables

```javascript
// ✅ Good: Tách logic ra hook riêng
const { users, loading, addUser } = useUsers();

// ❌ Bad: Tất cả logic trong component
const [users, setUsers] = useState([]);
const [loading, setLoading] = useState(false);
// ... 100 lines of logic
```

### 2. Keep View dumb

```vue
<!-- ✅ Good: View chỉ hiển thị -->
<template>
  <div>{{ formattedDate }}</div>
</template>

<!-- ❌ Bad: Logic trong template -->
<template>
  <div>{{ new Date(timestamp).toLocaleDateString("vi-VN") }}</div>
</template>
```

### 3. Computed cho derived data

```javascript
// ✅ Good: Dùng computed
const filteredUsers = computed(() => users.filter((u) => u.active));

// ❌ Bad: Filter trong template
// <div v-for="user in users.filter(u => u.active)">
```

## 12. Tổng kết

**MVVM là gì?** Pattern với ViewModel quản lý state, View bind two-way với ViewModel.

**Tại sao dùng?** UI reactive, tự động sync data với view, dễ test.

**Khi nào dùng?** Frontend SPAs, UI phức tạp, cần real-time updates.

**Nhớ:**

- Model: Data layer (API calls)
- ViewModel: State + Logic (hooks/composables)
- View: UI template (bind với ViewModel)
- Two-way binding: Data ↔ UI tự động sync
