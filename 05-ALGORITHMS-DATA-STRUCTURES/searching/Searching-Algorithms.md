# Thuật toán Tìm kiếm (Searching Algorithms)

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Linear Search](#2-linear-search)
3. [Binary Search](#3-binary-search)
4. [Jump Search](#4-jump-search)
5. [Interpolation Search](#5-interpolation-search)
6. [Exponential Search](#6-exponential-search)
7. [Ternary Search](#7-ternary-search)
8. [So sánh tổng hợp](#8-so-sánh-tổng-hợp)

---

## 1. Tổng quan

### 🎯 Searching là gì?

Tìm kiếm là quá trình **xác định vị trí** của một phần tử trong tập dữ liệu.

### 📊 Phân loại

```
┌─────────────────────────────────────────────────────────────────────┐
│                 PHÂN LOẠI THUẬT TOÁN TÌM KIẾM                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  THEO YÊU CẦU DỮ LIỆU:                                             │
│  ├── Không cần sắp xếp: Linear Search                              │
│  └── Cần sắp xếp: Binary, Jump, Interpolation, Exponential         │
│                                                                     │
│  THEO ĐỘ PHỨC TẠP:                                                 │
│  ├── O(n): Linear Search                                           │
│  ├── O(√n): Jump Search                                            │
│  ├── O(log n): Binary, Exponential                                 │
│  └── O(log log n): Interpolation (best case)                       │
│                                                                     │
│  THEO CÁCH TIẾP CẬN:                                               │
│  ├── Sequential: Linear, Jump                                      │
│  └── Divide & Conquer: Binary, Ternary                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Linear Search (Tìm kiếm tuyến tính)

### 🎯 Ý tưởng

Duyệt **tuần tự** từ đầu đến cuối, so sánh từng phần tử với giá trị cần tìm.

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]    Tìm: 8

Bước 1: [5, 3, 8, 1, 2]   5 == 8? ❌
         ↑
Bước 2: [5, 3, 8, 1, 2]   3 == 8? ❌
            ↑
Bước 3: [5, 3, 8, 1, 2]   8 == 8? ✓ Tìm thấy tại index 2
               ↑
```

### 💻 Code

**JavaScript:**

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i; // Trả về index
    }
  }
  return -1; // Không tìm thấy
}

// Test
console.log(linearSearch([5, 3, 8, 1, 2], 8)); // 2
console.log(linearSearch([5, 3, 8, 1, 2], 9)); // -1
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int linearSearch(vector<int>& arr, int target) {
    for (int i = 0; i < arr.size(); i++) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    cout << linearSearch(arr, 8) << endl;  // 2
    cout << linearSearch(arr, 9) << endl;  // -1
    return 0;
}
```

### 📊 Complexity

| Case                  | Time | Space |
| --------------------- | ---- | ----- |
| Best (đầu mảng)       | O(1) | O(1)  |
| Average               | O(n) | O(1)  |
| Worst (cuối/không có) | O(n) | O(1)  |

### ✅ Đánh giá

- ✅ Đơn giản nhất
- ✅ Không cần mảng đã sắp xếp
- ✅ Hoạt động với mọi kiểu dữ liệu
- ❌ Chậm với mảng lớn

---

## 3. Binary Search (Tìm kiếm nhị phân)

### 🎯 Ý tưởng

**Chia đôi** mảng đã sắp xếp, so sánh với phần tử giữa để quyết định tìm ở nửa trái hay phải.

### ⚠️ Yêu cầu

Mảng **PHẢI được sắp xếp** trước.

### 🖼️ Minh họa

```
Mảng: [1, 2, 3, 5, 8, 10, 15]    Tìm: 8

Bước 1: [1, 2, 3, 5, 8, 10, 15]
         L        M          H
         0        3          6

         arr[3] = 5 < 8 → Tìm bên phải

Bước 2: [1, 2, 3, 5, 8, 10, 15]
                     L   M   H
                     4   5   6

         arr[5] = 10 > 8 → Tìm bên trái

Bước 3: [1, 2, 3, 5, 8, 10, 15]
                     L
                     M
                     H
                     4

         arr[4] = 8 == 8 ✓ Tìm thấy tại index 4
```

### 💻 Code

**JavaScript - Iterative:**

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return -1;
}

console.log(binarySearch([1, 2, 3, 5, 8, 10, 15], 8)); // 4
```

**JavaScript - Recursive:**

```javascript
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1;

  const mid = Math.floor((left + right) / 2);

  if (arr[mid] === target) return mid;
  if (arr[mid] < target) return binarySearchRecursive(arr, target, mid + 1, right);
  return binarySearchRecursive(arr, target, left, mid - 1);
}
```

**C++ - Iterative:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int binarySearch(vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;  // Tránh overflow

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}

int main() {
    vector<int> arr = {1, 2, 3, 5, 8, 10, 15};
    cout << binarySearch(arr, 8) << endl;  // 4
    return 0;
}
```

**C++ - Recursive:**

```cpp
int binarySearchRecursive(vector<int>& arr, int target, int left, int right) {
    if (left > right) return -1;

    int mid = left + (right - left) / 2;

    if (arr[mid] == target) return mid;
    if (arr[mid] < target) return binarySearchRecursive(arr, target, mid + 1, right);
    return binarySearchRecursive(arr, target, left, mid - 1);
}
```

### 📊 Complexity

| Case    | Time     | Space (Iterative) | Space (Recursive) |
| ------- | -------- | ----------------- | ----------------- |
| Best    | O(1)     | O(1)              | O(1)              |
| Average | O(log n) | O(1)              | O(log n)          |
| Worst   | O(log n) | O(1)              | O(log n)          |

### 🔧 Các biến thể Binary Search

**Tìm vị trí đầu tiên (Lower Bound):**

```javascript
function lowerBound(arr, target) {
  let left = 0,
    right = arr.length;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }

  return left;
}

// [1, 2, 2, 2, 3, 4] tìm 2 → index 1 (vị trí đầu tiên)
```

**Tìm vị trí cuối cùng (Upper Bound):**

```javascript
function upperBound(arr, target) {
  let left = 0,
    right = arr.length;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] <= target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }

  return left - 1;
}

// [1, 2, 2, 2, 3, 4] tìm 2 → index 3 (vị trí cuối cùng)
```

### ✅ Đánh giá

- ✅ Rất nhanh O(log n)
- ✅ Hiệu quả với mảng lớn
- ❌ Yêu cầu mảng đã sắp xếp
- ❌ Chỉ hoạt động với random access (array)

---

## 4. Jump Search (Tìm kiếm nhảy)

### 🎯 Ý tưởng

**Nhảy** theo bước √n, khi tìm được block chứa target thì **Linear Search** trong block đó.

### 🖼️ Minh họa

```
Mảng: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]    Tìm: 11
       n = 15, step = √15 ≈ 4

Bước 1: Nhảy từng bước 4
        [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
         ↑        ↑        ↑         ↑
         0        4        8         12

         arr[0]=1 < 11 → Nhảy
         arr[4]=5 < 11 → Nhảy
         arr[8]=9 < 11 → Nhảy
         arr[12]=13 > 11 → Dừng, Linear Search từ 8 đến 12

Bước 2: Linear Search trong [9, 10, 11, 12]
        arr[8]=9 ≠ 11
        arr[9]=10 ≠ 11
        arr[10]=11 == 11 ✓ Tìm thấy tại index 10
```

### 💻 Code

**JavaScript:**

```javascript
function jumpSearch(arr, target) {
  const n = arr.length;
  const step = Math.floor(Math.sqrt(n));

  let prev = 0;
  let curr = step;

  // Nhảy để tìm block
  while (curr < n && arr[curr] < target) {
    prev = curr;
    curr += step;
  }

  // Linear search trong block
  for (let i = prev; i < Math.min(curr + 1, n); i++) {
    if (arr[i] === target) {
      return i;
    }
  }

  return -1;
}

const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15];
console.log(jumpSearch(arr, 11)); // 10
```

**C++:**

```cpp
#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

int jumpSearch(vector<int>& arr, int target) {
    int n = arr.size();
    int step = sqrt(n);

    int prev = 0;
    int curr = step;

    // Nhảy để tìm block
    while (curr < n && arr[curr] < target) {
        prev = curr;
        curr += step;
    }

    // Linear search trong block
    for (int i = prev; i < min(curr + 1, n); i++) {
        if (arr[i] == target) {
            return i;
        }
    }

    return -1;
}

int main() {
    vector<int> arr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15};
    cout << jumpSearch(arr, 11) << endl;  // 10
    return 0;
}
```

### 📊 Complexity

| Case    | Time  | Space |
| ------- | ----- | ----- |
| Best    | O(1)  | O(1)  |
| Average | O(√n) | O(1)  |
| Worst   | O(√n) | O(1)  |

### ✅ Đánh giá

- ✅ Tốt hơn Linear Search
- ✅ Ít so sánh hơn Binary Search trong một số trường hợp
- ✅ Chỉ nhảy tiến (tốt cho linked list)
- ❌ Chậm hơn Binary Search
- ❌ Yêu cầu mảng đã sắp xếp

---

## 5. Interpolation Search (Tìm kiếm nội suy)

### 🎯 Ý tưởng

Cải tiến Binary Search bằng cách **ước lượng vị trí** dựa trên giá trị target (giống tra từ điển).

### 📐 Công thức

```
pos = low + ((target - arr[low]) * (high - low)) / (arr[high] - arr[low])
```

### 🖼️ Minh họa

```
Mảng: [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]    Tìm: 70

Binary Search: Luôn chọn mid = 4 (arr[4] = 50)

Interpolation Search:
pos = 0 + ((70 - 10) * (9 - 0)) / (100 - 10)
    = 0 + (60 * 9) / 90
    = 6

arr[6] = 70 ✓ Tìm thấy ngay lần đầu!
```

### 💻 Code

**JavaScript:**

```javascript
function interpolationSearch(arr, target) {
  let low = 0;
  let high = arr.length - 1;

  while (low <= high && target >= arr[low] && target <= arr[high]) {
    if (low === high) {
      if (arr[low] === target) return low;
      return -1;
    }

    // Công thức nội suy
    const pos = low + Math.floor(((target - arr[low]) * (high - low)) / (arr[high] - arr[low]));

    if (arr[pos] === target) {
      return pos;
    } else if (arr[pos] < target) {
      low = pos + 1;
    } else {
      high = pos - 1;
    }
  }

  return -1;
}

const arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100];
console.log(interpolationSearch(arr, 70)); // 6
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int interpolationSearch(vector<int>& arr, int target) {
    int low = 0;
    int high = arr.size() - 1;

    while (low <= high && target >= arr[low] && target <= arr[high]) {
        if (low == high) {
            if (arr[low] == target) return low;
            return -1;
        }

        // Công thức nội suy
        int pos = low + ((double)(target - arr[low]) * (high - low))
                       / (arr[high] - arr[low]);

        if (arr[pos] == target) {
            return pos;
        } else if (arr[pos] < target) {
            low = pos + 1;
        } else {
            high = pos - 1;
        }
    }

    return -1;
}

int main() {
    vector<int> arr = {10, 20, 30, 40, 50, 60, 70, 80, 90, 100};
    cout << interpolationSearch(arr, 70) << endl;  // 6
    return 0;
}
```

### 📊 Complexity

| Case                      | Time         | Space |
| ------------------------- | ------------ | ----- |
| Best                      | O(1)         | O(1)  |
| Average (phân bố đều)     | O(log log n) | O(1)  |
| Worst (phân bố không đều) | O(n)         | O(1)  |

### ✅ Đánh giá

- ✅ Cực nhanh với dữ liệu phân bố đều
- ✅ O(log log n) - tốt hơn Binary Search
- ❌ Chậm với dữ liệu phân bố không đều
- ❌ Yêu cầu mảng đã sắp xếp
- ❌ Chỉ hoạt động với số

---

## 6. Exponential Search (Tìm kiếm hàm mũ)

### 🎯 Ý tưởng

1. Tìm **range** chứa target bằng cách nhân đôi index (1, 2, 4, 8, 16...)
2. **Binary Search** trong range đó

### 🖼️ Minh họa

```
Mảng: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]    Tìm: 10

Bước 1: Tìm range (nhân đôi)
        i = 1: arr[1] = 2 < 10 → i = 2
        i = 2: arr[2] = 3 < 10 → i = 4
        i = 4: arr[4] = 5 < 10 → i = 8
        i = 8: arr[8] = 9 < 10 → i = 16
        i = 16 > n → Dừng

        Range: [8, 15] (từ i/2 đến min(i, n-1))

Bước 2: Binary Search trong [8, 15]
        → Tìm thấy 10 tại index 9
```

### 💻 Code

**JavaScript:**

```javascript
function exponentialSearch(arr, target) {
  const n = arr.length;

  // Nếu phần tử đầu tiên là target
  if (arr[0] === target) return 0;

  // Tìm range bằng cách nhân đôi
  let i = 1;
  while (i < n && arr[i] <= target) {
    i *= 2;
  }

  // Binary search trong range [i/2, min(i, n-1)]
  return binarySearch(arr, target, Math.floor(i / 2), Math.min(i, n - 1));
}

function binarySearch(arr, target, left, right) {
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }

  return -1;
}

const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15];
console.log(exponentialSearch(arr, 10)); // 9
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int binarySearch(vector<int>& arr, int target, int left, int right) {
    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }

    return -1;
}

int exponentialSearch(vector<int>& arr, int target) {
    int n = arr.size();

    if (arr[0] == target) return 0;

    // Tìm range
    int i = 1;
    while (i < n && arr[i] <= target) {
        i *= 2;
    }

    // Binary search trong range
    return binarySearch(arr, target, i / 2, min(i, n - 1));
}

int main() {
    vector<int> arr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15};
    cout << exponentialSearch(arr, 10) << endl;  // 9
    return 0;
}
```

### 📊 Complexity

| Case    | Time     | Space |
| ------- | -------- | ----- |
| Best    | O(1)     | O(1)  |
| Average | O(log n) | O(1)  |
| Worst   | O(log n) | O(1)  |

### ✅ Đánh giá

- ✅ Tốt khi target ở gần đầu mảng
- ✅ Hoạt động với unbounded/infinite arrays
- ✅ O(log i) với i là vị trí của target
- ❌ Yêu cầu mảng đã sắp xếp

---

## 7. Ternary Search (Tìm kiếm tam phân)

### 🎯 Ý tưởng

Chia mảng thành **3 phần** thay vì 2 như Binary Search.

### 🖼️ Minh họa

```
Mảng: [1, 2, 3, 4, 5, 6, 7, 8, 9]    Tìm: 7

Bước 1: [1, 2, 3, 4, 5, 6, 7, 8, 9]
         L     M1    M2        H
         0     2     5         8

         arr[2] = 3 < 7 và arr[5] = 6 < 7
         → Tìm trong phần 3 (từ M2+1 đến H)

Bước 2: [1, 2, 3, 4, 5, 6, 7, 8, 9]
                           L  M1 M2 H
                           6  6  7  8

         arr[7] = 8 > 7 và arr[6] = 7 == 7 ✓
         Tìm thấy tại index 6
```

### 💻 Code

**JavaScript:**

```javascript
function ternarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid1 = left + Math.floor((right - left) / 3);
    const mid2 = right - Math.floor((right - left) / 3);

    if (arr[mid1] === target) return mid1;
    if (arr[mid2] === target) return mid2;

    if (target < arr[mid1]) {
      right = mid1 - 1;
    } else if (target > arr[mid2]) {
      left = mid2 + 1;
    } else {
      left = mid1 + 1;
      right = mid2 - 1;
    }
  }

  return -1;
}

console.log(ternarySearch([1, 2, 3, 4, 5, 6, 7, 8, 9], 7)); // 6
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int ternarySearch(vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;

    while (left <= right) {
        int mid1 = left + (right - left) / 3;
        int mid2 = right - (right - left) / 3;

        if (arr[mid1] == target) return mid1;
        if (arr[mid2] == target) return mid2;

        if (target < arr[mid1]) {
            right = mid1 - 1;
        } else if (target > arr[mid2]) {
            left = mid2 + 1;
        } else {
            left = mid1 + 1;
            right = mid2 - 1;
        }
    }

    return -1;
}

int main() {
    vector<int> arr = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    cout << ternarySearch(arr, 7) << endl;  // 6
    return 0;
}
```

### 📊 Complexity

| Case    | Time      | Space |
| ------- | --------- | ----- |
| Best    | O(1)      | O(1)  |
| Average | O(log₃ n) | O(1)  |
| Worst   | O(log₃ n) | O(1)  |

### ⚠️ So sánh với Binary Search

```
Binary Search:  log₂(n) comparisons, 1 comparison per iteration
Ternary Search: log₃(n) iterations, 2 comparisons per iteration

Tổng comparisons:
- Binary:  log₂(n) × 1 = log₂(n)
- Ternary: log₃(n) × 2 = 2 × log₂(n)/log₂(3) ≈ 1.26 × log₂(n)

→ Binary Search thực tế NHANH HƠN Ternary Search!
```

### ✅ Đánh giá

- ✅ Ít iterations hơn Binary Search
- ❌ Nhiều comparisons hơn → Chậm hơn Binary Search
- ❌ Thường dùng cho tìm cực trị của hàm unimodal, không phải tìm kiếm

---

## 8. So sánh tổng hợp

### 📊 Bảng so sánh

```
┌─────────────────────┬────────────┬────────────┬────────────┬─────────────────┐
│     Algorithm       │    Best    │  Average   │   Worst    │ Yêu cầu sắp xếp │
├─────────────────────┼────────────┼────────────┼────────────┼─────────────────┤
│ Linear Search       │    O(1)    │    O(n)    │    O(n)    │       ❌        │
│ Binary Search       │    O(1)    │  O(log n)  │  O(log n)  │       ✓         │
│ Jump Search         │    O(1)    │   O(√n)    │   O(√n)    │       ✓         │
│ Interpolation Search│    O(1)    │O(log log n)│    O(n)    │       ✓         │
│ Exponential Search  │    O(1)    │  O(log n)  │  O(log n)  │       ✓         │
│ Ternary Search      │    O(1)    │  O(log n)  │  O(log n)  │       ✓         │
└─────────────────────┴────────────┴────────────┴────────────┴─────────────────┘
```

### 🎯 Khi nào dùng thuật toán nào?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HƯỚNG DẪN CHỌN THUẬT TOÁN                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Mảng KHÔNG sắp xếp:                                                   │
│  └── Linear Search (lựa chọn duy nhất)                                 │
│                                                                         │
│  Mảng đã sắp xếp - General purpose:                                    │
│  └── Binary Search (tốt nhất trong hầu hết trường hợp)                 │
│                                                                         │
│  Mảng đã sắp xếp - Dữ liệu phân bố đều:                               │
│  └── Interpolation Search (O(log log n))                               │
│                                                                         │
│  Target có thể ở gần đầu mảng:                                         │
│  └── Exponential Search                                                 │
│                                                                         │
│  Mảng vô hạn / không biết kích thước:                                  │
│  └── Exponential Search                                                 │
│                                                                         │
│  Linked List đã sắp xếp:                                               │
│  └── Jump Search (chỉ cần forward traversal)                           │
│                                                                         │
│  Tìm cực trị của hàm unimodal:                                         │
│  └── Ternary Search                                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📈 Biểu đồ so sánh trực quan

```
Số phép so sánh với n = 1,000,000 phần tử

Linear Search       ████████████████████████████████████████  1,000,000
Jump Search         ████                                       1,000
Binary Search       █                                          20
Interpolation Search█                                          ~4 (best case)

⚠️ Interpolation có thể tệ hơn Linear nếu dữ liệu không đều
```

### 💡 Tips thực tế

```javascript
// 1. Trong JavaScript, dùng built-in methods
const arr = [1, 2, 3, 4, 5];
arr.indexOf(3);        // Linear Search - O(n)
arr.includes(3);       // Linear Search - O(n)
arr.find(x => x > 2);  // Linear Search - O(n)

// 2. Với mảng đã sắp xếp, tự implement Binary Search
// hoặc dùng thư viện như lodash
_.sortedIndex(arr, 3);  // Binary Search

// 3. Trong C++, dùng STL
#include <algorithm>
std::binary_search(arr.begin(), arr.end(), target);  // true/false
std::lower_bound(arr.begin(), arr.end(), target);    // iterator
std::upper_bound(arr.begin(), arr.end(), target);    // iterator
```

---

## 🔗 Tài liệu tham khảo

- [Visualgo - Searching Visualization](https://visualgo.net/en/sorting)
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [GeeksforGeeks - Searching Algorithms](https://www.geeksforgeeks.org/searching-algorithms/)
