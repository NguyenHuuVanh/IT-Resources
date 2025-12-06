# Thuật toán Sắp xếp (Sorting Algorithms)

## 📌 Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Bubble Sort](#2-bubble-sort)
3. [Selection Sort](#3-selection-sort)
4. [Insertion Sort](#4-insertion-sort)
5. [Merge Sort](#5-merge-sort)
6. [Quick Sort](#6-quick-sort)
7. [Heap Sort](#7-heap-sort)
8. [Counting Sort](#8-counting-sort)
9. [So sánh tổng hợp](#9-so-sánh-tổng-hợp)

---

## 1. Tổng quan

### 🎯 Sorting là gì?

Sắp xếp là quá trình **tổ chức lại** các phần tử trong một tập hợp theo **thứ tự** nhất định (tăng dần hoặc giảm dần).

### 📊 Phân loại

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHÂN LOẠI THUẬT TOÁN SẮP XẾP                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  THEO ĐỘ PHỨC TẠP:                                                 │
│  ├── O(n²): Bubble, Selection, Insertion                           │
│  ├── O(n log n): Merge, Quick, Heap                                │
│  └── O(n): Counting, Radix, Bucket (với điều kiện đặc biệt)       │
│                                                                     │
│  THEO BỘ NHỚ:                                                      │
│  ├── In-place: Bubble, Selection, Insertion, Quick, Heap          │
│  └── Out-of-place: Merge, Counting, Radix                         │
│                                                                     │
│  THEO TÍNH ỔN ĐỊNH (Stability):                                    │
│  ├── Stable: Bubble, Insertion, Merge, Counting                   │
│  └── Unstable: Selection, Quick, Heap                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 📝 Stability là gì?

```
Stable Sort: Giữ nguyên thứ tự tương đối của các phần tử bằng nhau

Input:  [(A,2), (B,1), (C,2), (D,1)]  (sắp xếp theo số)

Stable:   [(B,1), (D,1), (A,2), (C,2)]  ← B trước D, A trước C (giữ nguyên)
Unstable: [(D,1), (B,1), (C,2), (A,2)]  ← Thứ tự có thể thay đổi
```

---

## 2. Bubble Sort

### 🎯 Ý tưởng

So sánh và đổi chỗ các phần tử **liền kề** nhiều lần. Phần tử lớn "nổi" lên cuối như bong bóng.

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]

Lượt 1: [5,3,8,1,2] → [3,5,8,1,2] → [3,5,1,8,2] → [3,5,1,2,8]
                                                         ↑ 8 nổi lên
Lượt 2: [3,5,1,2,8] → [3,1,5,2,8] → [3,1,2,5,8]
                                         ↑ 5 nổi lên
Lượt 3: [3,1,2,5,8] → [1,3,2,5,8] → [1,2,3,5,8]
                                       ↑ 3 nổi lên
Kết quả: [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function bubbleSort(arr) {
  const n = arr.length;

  for (let i = 0; i < n - 1; i++) {
    let swapped = false;

    for (let j = 0; j < n - 1 - i; j++) {
      if (arr[j] > arr[j + 1]) {
        // Swap
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        swapped = true;
      }
    }

    // Tối ưu: Nếu không có swap nào → đã sắp xếp xong
    if (!swapped) break;
  }

  return arr;
}

// Test
console.log(bubbleSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

void bubbleSort(vector<int>& arr) {
    int n = arr.size();

    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;

        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }

        if (!swapped) break;
    }
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    bubbleSort(arr);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case              | Time  | Space |
| ----------------- | ----- | ----- |
| Best (đã sắp xếp) | O(n)  | O(1)  |
| Average           | O(n²) | O(1)  |
| Worst (ngược)     | O(n²) | O(1)  |

### ✅ Đánh giá

- ✅ Đơn giản, dễ hiểu
- ✅ Stable sort
- ✅ In-place (O(1) space)
- ❌ Chậm với dữ liệu lớn

---

## 3. Selection Sort

### 🎯 Ý tưởng

Tìm phần tử **nhỏ nhất** và đưa về đầu. Lặp lại với phần còn lại.

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]

Lượt 1: Tìm min trong [5,3,8,1,2] → 1 → Đổi với vị trí 0
        [1, 3, 8, 5, 2]
         ↑ đã sắp xếp

Lượt 2: Tìm min trong [3,8,5,2] → 2 → Đổi với vị trí 1
        [1, 2, 8, 5, 3]
         ↑──↑ đã sắp xếp

Lượt 3: Tìm min trong [8,5,3] → 3 → Đổi với vị trí 2
        [1, 2, 3, 5, 8]
         ↑─────↑ đã sắp xếp

Kết quả: [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function selectionSort(arr) {
  const n = arr.length;

  for (let i = 0; i < n - 1; i++) {
    let minIdx = i;

    // Tìm vị trí phần tử nhỏ nhất
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIdx]) {
        minIdx = j;
      }
    }

    // Đổi chỗ
    if (minIdx !== i) {
      [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
    }
  }

  return arr;
}

console.log(selectionSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

void selectionSort(vector<int>& arr) {
    int n = arr.size();

    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;

        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }

        if (minIdx != i) {
            swap(arr[i], arr[minIdx]);
        }
    }
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    selectionSort(arr);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case    | Time  | Space |
| ------- | ----- | ----- |
| Best    | O(n²) | O(1)  |
| Average | O(n²) | O(1)  |
| Worst   | O(n²) | O(1)  |

### ✅ Đánh giá

- ✅ Đơn giản
- ✅ Ít swap hơn Bubble Sort
- ✅ In-place
- ❌ Không stable
- ❌ Luôn O(n²) kể cả khi đã sắp xếp

---

## 4. Insertion Sort

### 🎯 Ý tưởng

Giống như **xếp bài**: Lấy từng phần tử và **chèn** vào đúng vị trí trong phần đã sắp xếp.

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]

Bước 1: [5] | 3, 8, 1, 2     → Chèn 3 vào [5]
        [3, 5] | 8, 1, 2

Bước 2: [3, 5] | 8, 1, 2     → Chèn 8 vào [3,5]
        [3, 5, 8] | 1, 2

Bước 3: [3, 5, 8] | 1, 2     → Chèn 1 vào [3,5,8]
        [1, 3, 5, 8] | 2

Bước 4: [1, 3, 5, 8] | 2     → Chèn 2 vào [1,3,5,8]
        [1, 2, 3, 5, 8]

Kết quả: [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function insertionSort(arr) {
  const n = arr.length;

  for (let i = 1; i < n; i++) {
    const key = arr[i];
    let j = i - 1;

    // Dịch các phần tử lớn hơn key sang phải
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }

    // Chèn key vào đúng vị trí
    arr[j + 1] = key;
  }

  return arr;
}

console.log(insertionSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

void insertionSort(vector<int>& arr) {
    int n = arr.size();

    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;

        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }

        arr[j + 1] = key;
    }
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    insertionSort(arr);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case              | Time  | Space |
| ----------------- | ----- | ----- |
| Best (đã sắp xếp) | O(n)  | O(1)  |
| Average           | O(n²) | O(1)  |
| Worst (ngược)     | O(n²) | O(1)  |

### ✅ Đánh giá

- ✅ Hiệu quả với mảng nhỏ hoặc gần như đã sắp xếp
- ✅ Stable sort
- ✅ In-place
- ✅ Online algorithm (có thể sort khi nhận data)
- ❌ Chậm với mảng lớn

---

## 5. Merge Sort

### 🎯 Ý tưởng

**Chia để trị (Divide and Conquer)**:

1. **Chia** mảng thành 2 nửa
2. **Đệ quy** sắp xếp từng nửa
3. **Trộn** 2 nửa đã sắp xếp

### 🖼️ Minh họa

```
                    [5, 3, 8, 1, 2]
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
        [5, 3, 8]                   [1, 2]
            │                           │
      ┌─────┴─────┐               ┌─────┴─────┐
      ▼           ▼               ▼           ▼
   [5, 3]       [8]             [1]         [2]
      │           │               │           │
   ┌──┴──┐        │               │           │
   ▼     ▼        │               │           │
  [5]   [3]       │               │           │
   │     │        │               │           │
   └──┬──┘        │               └─────┬─────┘
      ▼           │                     ▼
   [3, 5]         │                  [1, 2]
      │           │                     │
      └─────┬─────┘                     │
            ▼                           │
        [3, 5, 8]                       │
            │                           │
            └─────────────┬─────────────┘
                          ▼
                  [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0,
    j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }

  // Thêm phần còn lại
  return result.concat(left.slice(i)).concat(right.slice(j));
}

console.log(mergeSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> L(arr.begin() + left, arr.begin() + mid + 1);
    vector<int> R(arr.begin() + mid + 1, arr.begin() + right + 1);

    int i = 0, j = 0, k = left;

    while (i < L.size() && j < R.size()) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }

    while (i < L.size()) arr[k++] = L[i++];
    while (j < R.size()) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;

    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    mergeSort(arr, 0, arr.size() - 1);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case    | Time       | Space |
| ------- | ---------- | ----- |
| Best    | O(n log n) | O(n)  |
| Average | O(n log n) | O(n)  |
| Worst   | O(n log n) | O(n)  |

### ✅ Đánh giá

- ✅ Luôn O(n log n) - hiệu quả và ổn định
- ✅ Stable sort
- ✅ Tốt cho linked list
- ❌ Cần O(n) bộ nhớ phụ
- ❌ Không in-place

---

## 6. Quick Sort

### 🎯 Ý tưởng

**Chia để trị** với **pivot**:

1. Chọn một phần tử làm **pivot**
2. **Partition**: Đưa các phần tử nhỏ hơn pivot sang trái, lớn hơn sang phải
3. **Đệ quy** sắp xếp 2 phần

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]    (chọn pivot = 5)

Partition: Nhỏ hơn 5 | 5 | Lớn hơn 5
          [3, 1, 2]   [5]   [8]

Đệ quy trái: [3, 1, 2] (pivot = 3)
            [1, 2] [3] []

            [1, 2] (pivot = 1)
            [] [1] [2]

            → [1, 2, 3]

Đệ quy phải: [8] → [8]

Kết hợp: [1, 2, 3] + [5] + [8] = [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
  if (low < high) {
    const pivotIdx = partition(arr, low, high);
    quickSort(arr, low, pivotIdx - 1);
    quickSort(arr, pivotIdx + 1, high);
  }
  return arr;
}

function partition(arr, low, high) {
  const pivot = arr[high]; // Chọn phần tử cuối làm pivot
  let i = low - 1;

  for (let j = low; j < high; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }

  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1;
}

console.log(quickSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;

    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }

    swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pivotIdx = partition(arr, low, high);
        quickSort(arr, low, pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);
    }
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    quickSort(arr, 0, arr.size() - 1);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case               | Time       | Space    |
| ------------------ | ---------- | -------- |
| Best               | O(n log n) | O(log n) |
| Average            | O(n log n) | O(log n) |
| Worst (đã sắp xếp) | O(n²)      | O(n)     |

### ✅ Đánh giá

- ✅ Nhanh nhất trong thực tế (cache-friendly)
- ✅ In-place (O(log n) cho call stack)
- ✅ Được dùng trong nhiều thư viện chuẩn
- ❌ Không stable
- ❌ Worst case O(n²) với pivot tệ

### 💡 Tối ưu chọn Pivot

```javascript
// Median of three - tránh worst case
function medianOfThree(arr, low, high) {
  const mid = Math.floor((low + high) / 2);

  if (arr[low] > arr[mid]) [arr[low], arr[mid]] = [arr[mid], arr[low]];
  if (arr[low] > arr[high]) [arr[low], arr[high]] = [arr[high], arr[low]];
  if (arr[mid] > arr[high]) [arr[mid], arr[high]] = [arr[high], arr[mid]];

  return mid;
}
```

---

## 7. Heap Sort

### 🎯 Ý tưởng

Sử dụng cấu trúc **Max Heap** để sắp xếp:

1. **Build Max Heap** từ mảng
2. Lấy phần tử lớn nhất (root) đưa về cuối
3. **Heapify** lại và lặp lại

### 🖼️ Minh họa

```
Mảng: [5, 3, 8, 1, 2]

Build Max Heap:
        8              Index: [8, 5, 3, 1, 2]
       / \
      5   3
     / \
    1   2

Extract Max:
1. Swap 8 với 2: [2, 5, 3, 1, 8]
2. Heapify: [5, 2, 3, 1 | 8]
3. Swap 5 với 1: [1, 2, 3, 5 | 8]
4. Heapify: [3, 2, 1 | 5, 8]
5. Tiếp tục...

Kết quả: [1, 2, 3, 5, 8] ✓
```

### 💻 Code

**JavaScript:**

```javascript
function heapSort(arr) {
  const n = arr.length;

  // Build max heap
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }

  // Extract elements from heap
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]]; // Swap root với cuối
    heapify(arr, i, 0); // Heapify phần còn lại
  }

  return arr;
}

function heapify(arr, n, i) {
  let largest = i;
  const left = 2 * i + 1;
  const right = 2 * i + 2;

  if (left < n && arr[left] > arr[largest]) {
    largest = left;
  }

  if (right < n && arr[right] > arr[largest]) {
    largest = right;
  }

  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}

console.log(heapSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

void heapify(vector<int>& arr, int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest])
        largest = left;

    if (right < n && arr[right] > arr[largest])
        largest = right;

    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();

    // Build max heap
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    // Extract elements
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);
        heapify(arr, i, 0);
    }
}

int main() {
    vector<int> arr = {5, 3, 8, 1, 2};
    heapSort(arr);

    for (int x : arr) cout << x << " ";  // 1 2 3 5 8
    return 0;
}
```

### 📊 Complexity

| Case    | Time       | Space |
| ------- | ---------- | ----- |
| Best    | O(n log n) | O(1)  |
| Average | O(n log n) | O(1)  |
| Worst   | O(n log n) | O(1)  |

### ✅ Đánh giá

- ✅ Luôn O(n log n)
- ✅ In-place
- ✅ Không cần bộ nhớ phụ
- ❌ Không stable
- ❌ Chậm hơn Quick Sort trong thực tế (không cache-friendly)

---

## 8. Counting Sort

### 🎯 Ý tưởng

**Đếm** số lần xuất hiện của mỗi giá trị, sau đó **xây dựng lại** mảng.
Chỉ áp dụng cho **số nguyên** trong **phạm vi nhỏ**.

### 🖼️ Minh họa

```
Mảng: [4, 2, 2, 8, 3, 3, 1]    Range: 1-8

Bước 1: Đếm số lần xuất hiện
Count: [0, 1, 2, 2, 1, 0, 0, 0, 1]
Index:  0  1  2  3  4  5  6  7  8
            ↑  ↑  ↑           ↑
           2x 2x 1x          1x

Bước 2: Tính vị trí tích lũy
Count: [0, 1, 3, 5, 6, 6, 6, 6, 7]

Bước 3: Xây dựng mảng kết quả
Output: [1, 2, 2, 3, 3, 4, 8]
```

### 💻 Code

**JavaScript:**

```javascript
function countingSort(arr) {
  if (arr.length === 0) return arr;

  const max = Math.max(...arr);
  const min = Math.min(...arr);
  const range = max - min + 1;

  const count = new Array(range).fill(0);
  const output = new Array(arr.length);

  // Đếm số lần xuất hiện
  for (const num of arr) {
    count[num - min]++;
  }

  // Tính vị trí tích lũy
  for (let i = 1; i < range; i++) {
    count[i] += count[i - 1];
  }

  // Xây dựng mảng kết quả (từ cuối để stable)
  for (let i = arr.length - 1; i >= 0; i--) {
    output[count[arr[i] - min] - 1] = arr[i];
    count[arr[i] - min]--;
  }

  return output;
}

console.log(countingSort([4, 2, 2, 8, 3, 3, 1])); // [1, 2, 2, 3, 3, 4, 8]
```

**C++:**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

vector<int> countingSort(vector<int>& arr) {
    if (arr.empty()) return arr;

    int maxVal = *max_element(arr.begin(), arr.end());
    int minVal = *min_element(arr.begin(), arr.end());
    int range = maxVal - minVal + 1;

    vector<int> count(range, 0);
    vector<int> output(arr.size());

    // Đếm
    for (int num : arr) {
        count[num - minVal]++;
    }

    // Tích lũy
    for (int i = 1; i < range; i++) {
        count[i] += count[i - 1];
    }

    // Xây dựng output
    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i] - minVal] - 1] = arr[i];
        count[arr[i] - minVal]--;
    }

    return output;
}

int main() {
    vector<int> arr = {4, 2, 2, 8, 3, 3, 1};
    vector<int> sorted = countingSort(arr);

    for (int x : sorted) cout << x << " ";  // 1 2 2 3 3 4 8
    return 0;
}
```

### 📊 Complexity

| Case      | Time     | Space    |
| --------- | -------- | -------- |
| All cases | O(n + k) | O(n + k) |

_k = range của giá trị (max - min)_

### ✅ Đánh giá

- ✅ O(n) khi k nhỏ - nhanh hơn comparison-based sorts
- ✅ Stable sort
- ❌ Chỉ dùng cho số nguyên
- ❌ Tốn bộ nhớ khi range lớn

---

## 9. So sánh tổng hợp

### 📊 Bảng so sánh

```
┌─────────────────┬───────────────────────────────────────────────────────────┐
│   Algorithm     │  Best      │  Average   │  Worst     │ Space  │ Stable  │
├─────────────────┼────────────┼────────────┼────────────┼────────┼─────────┤
│ Bubble Sort     │   O(n)     │   O(n²)    │   O(n²)    │  O(1)  │   ✓     │
│ Selection Sort  │   O(n²)    │   O(n²)    │   O(n²)    │  O(1)  │   ✗     │
│ Insertion Sort  │   O(n)     │   O(n²)    │   O(n²)    │  O(1)  │   ✓     │
│ Merge Sort      │ O(n log n) │ O(n log n) │ O(n log n) │  O(n)  │   ✓     │
│ Quick Sort      │ O(n log n) │ O(n log n) │   O(n²)    │O(log n)│   ✗     │
│ Heap Sort       │ O(n log n) │ O(n log n) │ O(n log n) │  O(1)  │   ✗     │
│ Counting Sort   │  O(n + k)  │  O(n + k)  │  O(n + k)  │O(n + k)│   ✓     │
└─────────────────┴────────────┴────────────┴────────────┴────────┴─────────┘
```

### 🎯 Khi nào dùng thuật toán nào?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HƯỚNG DẪN CHỌN THUẬT TOÁN                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Mảng nhỏ (n < 50):                                                    │
│  └── Insertion Sort (đơn giản, overhead thấp)                          │
│                                                                         │
│  Mảng gần như đã sắp xếp:                                              │
│  └── Insertion Sort (O(n) best case)                                   │
│                                                                         │
│  Mảng lớn, cần hiệu suất cao:                                          │
│  └── Quick Sort (nhanh nhất trong thực tế)                             │
│                                                                         │
│  Cần đảm bảo O(n log n) worst case:                                    │
│  └── Merge Sort hoặc Heap Sort                                         │
│                                                                         │
│  Cần stable sort:                                                       │
│  └── Merge Sort                                                         │
│                                                                         │
│  Bộ nhớ hạn chế:                                                       │
│  └── Heap Sort (in-place, O(1) space)                                  │
│                                                                         │
│  Số nguyên trong phạm vi nhỏ:                                          │
│  └── Counting Sort (O(n))                                              │
│                                                                         │
│  Linked List:                                                           │
│  └── Merge Sort (không cần random access)                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📈 Biểu đồ so sánh trực quan

```
Thời gian thực thi (n = 10,000 phần tử ngẫu nhiên)

Bubble Sort     ████████████████████████████████████████  ~100ms
Selection Sort  ████████████████████████████████████      ~90ms
Insertion Sort  ██████████████████████████████            ~70ms
Heap Sort       ████                                       ~8ms
Merge Sort      ███                                        ~6ms
Quick Sort      ██                                         ~4ms
Counting Sort   █                                          ~1ms (nếu range nhỏ)

⚠️ Số liệu minh họa, phụ thuộc vào implementation và hardware
```

---

## 🔗 Tài liệu tham khảo

- [Visualgo - Sorting Visualization](https://visualgo.net/en/sorting)
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [GeeksforGeeks - Sorting Algorithms](https://www.geeksforgeeks.org/sorting-algorithms/)
