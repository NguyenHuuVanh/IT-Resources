# MVVM Pattern (Model - View - ViewModel)

## 1. Tổng quan

MVVM phổ biến trong Frontend development, đặc biệt với các framework như Vue.js, Angular, và React (với state management).

```
┌─────────────┐     Two-way      ┌─────────────┐     ┌─────────────┐
│    View     │◄═══════════════►│  ViewModel  │◄───►│    Model    │
│  (Template) │     Binding      │   (State)   │     │   (Data)    │
└─────────────┘                  └─────────────┘     └─────────────┘
```

## 2. Các thành phần

### Model

- Data layer, business logic
- API calls, data transformation
- Không biết về View và ViewModel

```javascript
// models/userModel.js
const API_URL = "/api/users";

export const userModel = {
  async getAll() {
    const response = await fetch(API_URL);
    return response.json();
  },

  async getById(id) {
    const response = await fetch(`${API_URL}/${id}`);
    return response.json();
  },

  async create(userData) {
    const response = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
    return response.json();
  },

  async update(id, userData) {
    const response = await fetch(`${API_URL}/${id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
    return response.json();
  },

  async delete(id) {
    await fetch(`${API_URL}/${id}`, { method: "DELETE" });
  },
};
```

### ViewModel

- Chứa state và presentation logic
- Expose data cho View qua data binding
- Handle user interactions

### View

- UI template
- Bind với ViewModel
- Không chứa logic

## 3. Ví dụ với Vue.js

```vue
<!-- UserList.vue -->
<template>
  <!-- VIEW: Chỉ hiển thị, bind với ViewModel -->
  <div class="user-list">
    <h1>Users</h1>

    <!-- Search -->
    <input v-model="searchQuery" placeholder="Search users..." class="search-input" />

    <!-- Loading state -->
    <div v-if="loading" class="loading">Loading...</div>

    <!-- Error state -->
    <div v-else-if="error" class="error">{{ error }}</div>

    <!-- User list -->
    <div v-else>
      <p>Showing {{ filteredUsers.length }} of {{ users.length }} users</p>

      <ul class="users">
        <li v-for="user in filteredUsers" :key="user.id" class="user-item">
          <span>{{ user.name }}</span>
          <span>{{ user.email }}</span>
          <button @click="editUser(user)">Edit</button>
          <button @click="deleteUser(user.id)">Delete</button>
        </li>
      </ul>

      <p v-if="filteredUsers.length === 0">No users found.</p>
    </div>

    <!-- Add user form -->
    <form @submit.prevent="addUser" class="add-form">
      <input v-model="newUser.name" placeholder="Name" required />
      <input v-model="newUser.email" placeholder="Email" required />
      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? "Adding..." : "Add User" }}
      </button>
    </form>
  </div>
</template>

<script>
import { userModel } from "@/models/userModel";

export default {
  name: "UserList",

  // VIEWMODEL: State
  data() {
    return {
      users: [],
      loading: false,
      error: null,
      searchQuery: "",
      newUser: {
        name: "",
        email: "",
      },
      isSubmitting: false,
    };
  },

  // VIEWMODEL: Computed properties (derived state)
  computed: {
    filteredUsers() {
      if (!this.searchQuery) return this.users;

      const query = this.searchQuery.toLowerCase();
      return this.users.filter(
        (user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query)
      );
    },

    hasUsers() {
      return this.users.length > 0;
    },
  },

  // VIEWMODEL: Methods (actions)
  methods: {
    async fetchUsers() {
      this.loading = true;
      this.error = null;

      try {
        this.users = await userModel.getAll();
      } catch (err) {
        this.error = "Failed to load users";
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    async addUser() {
      if (this.isSubmitting) return;

      this.isSubmitting = true;
      try {
        const user = await userModel.create(this.newUser);
        this.users.push(user);
        this.newUser = { name: "", email: "" };
      } catch (err) {
        this.error = "Failed to add user";
      } finally {
        this.isSubmitting = false;
      }
    },

    editUser(user) {
      this.$router.push(`/users/${user.id}/edit`);
    },

    async deleteUser(id) {
      if (!confirm("Are you sure?")) return;

      try {
        await userModel.delete(id);
        this.users = this.users.filter((u) => u.id !== id);
      } catch (err) {
        this.error = "Failed to delete user";
      }
    },
  },

  // Lifecycle
  mounted() {
    this.fetchUsers();
  },
};
</script>
```

## 4. Ví dụ với Vue 3 Composition API

```vue
<script setup>
import { ref, computed, onMounted } from "vue";
import { userModel } from "@/models/userModel";

// State
const users = ref([]);
const loading = ref(false);
const error = ref(null);
const searchQuery = ref("");
const newUser = ref({ name: "", email: "" });
const isSubmitting = ref(false);

// Computed
const filteredUsers = computed(() => {
  if (!searchQuery.value) return users.value;

  const query = searchQuery.value.toLowerCase();
  return users.value.filter(
    (user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query)
  );
});

// Methods
async function fetchUsers() {
  loading.value = true;
  error.value = null;

  try {
    users.value = await userModel.getAll();
  } catch (err) {
    error.value = "Failed to load users";
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
    error.value = "Failed to add user";
  } finally {
    isSubmitting.value = false;
  }
}

async function deleteUser(id) {
  if (!confirm("Are you sure?")) return;

  try {
    await userModel.delete(id);
    users.value = users.value.filter((u) => u.id !== id);
  } catch (err) {
    error.value = "Failed to delete user";
  }
}

// Lifecycle
onMounted(fetchUsers);
</script>
```

## 5. Ví dụ với React + Hooks

```jsx
// hooks/useUsers.js (ViewModel as Custom Hook)
import { useState, useEffect, useMemo, useCallback } from "react";
import { userModel } from "../models/userModel";

export function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");

  // Fetch users
  const fetchUsers = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const data = await userModel.getAll();
      setUsers(data);
    } catch (err) {
      setError("Failed to load users");
    } finally {
      setLoading(false);
    }
  }, []);

  // Filtered users (computed)
  const filteredUsers = useMemo(() => {
    if (!searchQuery) return users;
    const query = searchQuery.toLowerCase();
    return users.filter((user) => user.name.toLowerCase().includes(query) || user.email.toLowerCase().includes(query));
  }, [users, searchQuery]);

  // Add user
  const addUser = useCallback(async (userData) => {
    const user = await userModel.create(userData);
    setUsers((prev) => [...prev, user]);
    return user;
  }, []);

  // Delete user
  const deleteUser = useCallback(async (id) => {
    await userModel.delete(id);
    setUsers((prev) => prev.filter((u) => u.id !== id));
  }, []);

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  return {
    users,
    filteredUsers,
    loading,
    error,
    searchQuery,
    setSearchQuery,
    addUser,
    deleteUser,
    refetch: fetchUsers,
  };
}

// UserList.jsx (View)
import { useUsers } from "../hooks/useUsers";

function UserList() {
  const { filteredUsers, loading, error, searchQuery, setSearchQuery, addUser, deleteUser } = useUsers();

  const [newUser, setNewUser] = useState({ name: "", email: "" });

  const handleSubmit = async (e) => {
    e.preventDefault();
    await addUser(newUser);
    setNewUser({ name: "", email: "" });
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>{error}</div>;

  return (
    <div>
      <input value={searchQuery} onChange={(e) => setSearchQuery(e.target.value)} placeholder="Search..." />

      <ul>
        {filteredUsers.map((user) => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => deleteUser(user.id)}>Delete</button>
          </li>
        ))}
      </ul>

      <form onSubmit={handleSubmit}>
        <input
          value={newUser.name}
          onChange={(e) => setNewUser({ ...newUser, name: e.target.value })}
          placeholder="Name"
        />
        <input
          value={newUser.email}
          onChange={(e) => setNewUser({ ...newUser, email: e.target.value })}
          placeholder="Email"
        />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```

## 6. So sánh MVC vs MVVM

| Aspect               | MVC                           | MVVM                   |
| -------------------- | ----------------------------- | ---------------------- |
| Data flow            | One-way                       | Two-way binding        |
| View                 | Active, có thể gọi Controller | Passive, chỉ bind data |
| Controller/ViewModel | Handle requests               | Manage state & logic   |
| Testing              | Test Controller               | Test ViewModel         |
| Use case             | Server-side, APIs             | Frontend SPAs          |

## 7. Ưu và Nhược điểm

### Ưu điểm

- ✅ Two-way data binding giảm boilerplate
- ✅ Reactive updates tự động
- ✅ ViewModel dễ unit test
- ✅ Tách biệt UI và logic rõ ràng

### Nhược điểm

- ❌ Debugging khó hơn (data flow phức tạp)
- ❌ Performance issues với large datasets
- ❌ Learning curve cao hơn MVC
- ❌ Over-engineering cho simple apps

## 8. Khi nào dùng MVVM?

- ✅ Single Page Applications (SPAs)
- ✅ Complex UI với nhiều state
- ✅ Real-time updates
- ✅ Forms với validation
- ❌ Không cần cho static pages
- ❌ Server-rendered apps (dùng MVC)
