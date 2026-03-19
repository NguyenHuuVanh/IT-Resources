# Git Conflict Resolution Guide / Hướng Dẫn Xử Lý Conflict Git

## Understanding Conflicts / Hiểu Về Conflict

### What is a Git Conflict? / Conflict Git là gì?

**English**: A conflict occurs when Git cannot automatically merge changes because two people modified the same lines in a file, or one person deleted a file while another modified it.

**Tiếng Việt**: Conflict xảy ra khi Git không thể tự động merge các thay đổi vì hai người đã sửa cùng một dòng trong file, hoặc một người xóa file trong khi người khác sửa nó.

### Key Git Concepts / Các Khái Niệm Git Quan Trọng

| Khái niệm | Định nghĩa (EN) | Định nghĩa (VI) |
| --- | --- | --- |
| **Repository (Repo)** | A storage location for your project's files and their version history | Nơi lưu trữ các file dự án và lịch sử thay đổi của chúng |
| **Local Repository** | The repo stored on your machine | Repo lưu trên máy của bạn |
| **Remote Repository** | The repo stored on a server (GitHub, GitLab...) | Repo lưu trên server (GitHub, GitLab...) |
| **Branch** | An independent line of development | Một nhánh phát triển độc lập |
| **HEAD** | A pointer to the latest commit on the current branch | Con trỏ đến commit mới nhất trên nhánh hiện tại |
| **Merge** | Combining changes from two branches into one | Kết hợp thay đổi từ hai nhánh thành một |
| **Rebase** | Re-applying commits on top of another base commit | Áp dụng lại các commits trên một commit gốc khác |
| **Cherry-pick** | Picking a specific commit from one branch and applying it to another | Chọn một commit cụ thể từ nhánh này và áp dụng vào nhánh khác |
| **Staging Area (Index)** | A holding area where you prepare changes before committing | Vùng trung gian nơi bạn chuẩn bị thay đổi trước khi commit |
| **Working Directory** | The actual files on your disk that you edit | Các file thực tế trên ổ đĩa mà bạn chỉnh sửa |
| **ORIG_HEAD** | A backup reference that Git creates before dangerous operations (merge, rebase) | Tham chiếu backup mà Git tạo trước các thao tác nguy hiểm (merge, rebase) |

---

## Common Conflict Scenarios / Các Tình Huống Conflict Thường Gặp

### Scenario 1: Push Rejected / Push Bị Từ Chối

**English**:

```bash
# Error message
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'repository-url'
```

**Tiếng Việt**: Lỗi này xảy ra khi remote repository có commits mới mà local chưa có. Git từ chối push vì nếu cho phép, các commits của người khác trên remote sẽ bị ghi đè và mất.

**Solution / Giải pháp**:

```bash
# Step 1: Fetch latest changes / Tải về các thay đổi mới nhất
git fetch origin
```

> 📖 **`git fetch origin`**: Tải về tất cả thay đổi mới từ remote repository (origin) nhưng **KHÔNG tự động merge** vào code của bạn. Đây là cách an toàn để xem remote có gì mới mà không ảnh hưởng đến working directory.
>
> - `origin` là tên mặc định của remote repository (server) mà bạn clone từ đó.
> - So với `git pull`, `fetch` chỉ tải về metadata, không thay đổi code local.

```bash
# Step 2: Pull and merge / Kéo về và merge
git pull origin main
```

> 📖 **`git pull origin main`**: Lệnh này = `git fetch` + `git merge`. Nó tải thay đổi mới từ nhánh `main` trên remote `origin` rồi **tự động merge** vào nhánh hiện tại của bạn.
>
> - Tại sao dùng: Vì remote đã có commits mới, bạn cần đồng bộ local trước khi push.
> - Nếu có conflict, Git sẽ dừng lại và yêu cầu bạn xử lý thủ công.

```bash
# Step 3: Resolve conflicts if any (see below)
# Step 4: Push again / Push lại
git push origin main
```

> 📖 **`git push origin main`**: Đẩy tất cả commits local lên nhánh `main` trên remote `origin`.
>
> - Tại sao dùng: Sau khi đã đồng bộ (pull + resolve conflicts), giờ local của bạn đã "ahead" so với remote → có thể push an toàn.
> - `origin` = remote server, `main` = tên nhánh đích.

---

## Step-by-Step Conflict Resolution / Các Bước Xử Lý Conflict

### Method 1: Manual Resolution / Xử Lý Thủ Công

**Step 1: Identify conflicts / Xác định conflicts**

```bash
git status
```

> 📖 **`git status`**: Hiển thị trạng thái hiện tại của working directory và staging area.
>
> - Tại sao dùng: Khi có conflict, lệnh này cho bạn biết **chính xác file nào bị conflict** (hiển thị là `both modified` hoặc `Unmerged paths`).
> - Nó cũng cho biết file nào đã staged, file nào chưa tracked, giúp bạn nắm rõ tình hình trước khi hành động.

**Step 2: Open conflicted files / Mở các file bị conflict**

Conflict markers look like / Dấu hiệu conflict:

```
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name
```

**English**:

- `<<<<<<< HEAD`: Your current changes (on the branch you are currently on)
- `=======`: Separator between the two versions
- `>>>>>>> branch-name`: Incoming changes (from the branch being merged in)

**Tiếng Việt**:

- `<<<<<<< HEAD`: Thay đổi của bạn (trên nhánh hiện tại mà bạn đang đứng)
- `=======`: Dấu phân cách giữa hai phiên bản
- `>>>>>>> branch-name`: Thay đổi từ remote (từ nhánh đang được merge vào)

> 💡 **Giải thích thêm**: Git chèn các conflict markers này vào file khi nó không biết nên giữ phiên bản nào. Nhiệm vụ của bạn là xóa hết các markers (`<<<`, `===`, `>>>`) và chỉ giữ lại code đúng mà bạn muốn.

**Step 3: Edit the file / Sửa file**

Choose one of these options / Chọn một trong các cách:

1. **Keep your changes / Giữ thay đổi của bạn**:

```javascript
// Remove conflict markers and keep only your code
function myFunction() {
  // Your implementation
}
```

2. **Accept their changes / Chấp nhận thay đổi của họ**:

```javascript
// Remove conflict markers and keep only their code
function myFunction() {
  // Their implementation
}
```

3. **Combine both / Kết hợp cả hai**:

```javascript
// Keep the best parts from both versions
function myFunction() {
  // Combined implementation
}
```

**Step 4: Mark as resolved / Đánh dấu đã giải quyết**

```bash
git add <file-name>
```

> 📖 **`git add <file-name>`**: Đưa file vào **staging area** (vùng chuẩn bị commit).
>
> - Tại sao dùng ở đây: Sau khi bạn đã sửa xong conflict trong file, `git add` báo cho Git biết rằng **conflict đã được giải quyết** và file này sẵn sàng để commit.
> - Trong ngữ cảnh conflict, `git add` có 2 ý nghĩa: (1) đánh dấu conflict đã resolved, (2) stage file để commit.

**Step 5: Complete the merge / Hoàn thành merge**

```bash
git commit -m "Resolve merge conflicts"
```

> 📖 **`git commit -m "message"`**: Tạo một commit mới với tất cả thay đổi đã staged, kèm theo message mô tả.
>
> - Tại sao dùng: Sau khi resolve tất cả conflicts và `git add`, bạn cần commit để hoàn tất quá trình merge.
> - `-m` cho phép viết commit message trực tiếp trên command line thay vì mở editor.

```bash
git push origin main
```

> 📖 **`git push origin main`**: Đẩy commit lên remote.
>
> - Tại sao dùng: Để đồng bộ kết quả merge conflict lên server cho team thấy.

---

### Method 2: Using VS Code / Sử dụng VS Code

**English**: VS Code provides visual tools for conflict resolution.

**Tiếng Việt**: VS Code cung cấp công cụ trực quan để xử lý conflict. Thay vì sửa conflict markers thủ công, VS Code hiển thị các nút bấm trực quan ngay phía trên mỗi vùng conflict.

**Steps / Các bước**:

1. Open conflicted file in VS Code / Mở file bị conflict trong VS Code
2. You'll see options above each conflict / Bạn sẽ thấy các tùy chọn phía trên mỗi conflict:
   - `Accept Current Change` — giữ code của bạn (HEAD)
   - `Accept Incoming Change` — lấy code từ nhánh đang merge vào
   - `Accept Both Changes` — giữ cả hai phiên bản (xếp chồng lên nhau)
   - `Compare Changes` — mở chế độ so sánh side-by-side để xem rõ sự khác biệt

3. Click the appropriate option / Bấm vào tùy chọn phù hợp
4. Save the file / Lưu file
5. Stage and commit:

```bash
git add .
```

> 📖 **`git add .`**: Stage **tất cả** file đã thay đổi trong thư mục hiện tại và các thư mục con.
>
> - Dấu `.` nghĩa là "thư mục hiện tại" (tất cả files).
> - So với `git add <file-name>` (stage từng file), `git add .` nhanh hơn khi bạn đã resolve xong tất cả conflicts.
> - ⚠️ **Cẩn thận**: Nó stage TẤT CẢ thay đổi, kể cả files bạn chưa muốn commit. Hãy chạy `git status` trước để kiểm tra.

```bash
git commit -m "Resolve conflicts using VS Code"
git push
```

> 📖 **`git push`** (không có `origin main`): Khi nhánh local đã được liên kết (tracking) với nhánh remote, bạn có thể bỏ qua `origin main`. Git sẽ tự hiểu push lên nhánh remote tương ứng.

---

### Method 3: Using Git Tools / Sử dụng Git Tools

**Option A: Accept all theirs / Chấp nhận tất cả của họ**

```bash
git checkout --theirs <file-name>
```

> 📖 **`git checkout --theirs <file>`**: Lấy **toàn bộ nội dung file từ nhánh đang merge vào** (nhánh "của họ"), ghi đè lên file hiện tại.
>
> - Tại sao dùng: Khi bạn chắc chắn rằng phiên bản của người khác là đúng và muốn bỏ hết thay đổi của mình trong file đó.
> - `--theirs` = nhánh đang được merge vào (incoming branch).

```bash
git add <file-name>
```

**Option B: Accept all yours / Giữ tất cả của bạn**

```bash
git checkout --ours <file-name>
```

> 📖 **`git checkout --ours <file>`**: Giữ **toàn bộ nội dung file của nhánh hiện tại** (nhánh "của bạn"), bỏ hết thay đổi từ nhánh incoming.
>
> - Tại sao dùng: Khi bạn chắc chắn code của mình là đúng và không cần thay đổi từ người khác.
> - `--ours` = nhánh hiện tại (current branch / HEAD).
> - ⚠️ **Lưu ý**: Trong rebase, `--ours` và `--theirs` bị **đảo ngược** so với merge thông thường!

```bash
git add <file-name>
```

**Option C: Use merge tool / Dùng merge tool**

```bash
git mergetool
```

> 📖 **`git mergetool`**: Mở công cụ merge visual (GUI) đã được cấu hình để xử lý conflict.
>
> - Tại sao dùng: Cung cấp giao diện trực quan (3-way merge) giúp bạn so sánh 3 phiên bản: (1) phiên bản gốc trước khi cả hai sửa, (2) phiên bản của bạn, (3) phiên bản của họ.
> - Cần cấu hình trước qua `git config --global merge.tool <tool-name>` (ví dụ: `meld`, `p4merge`, `kdiff3`).

---

## Advanced Scenarios / Tình Huống Nâng Cao

### Scenario 2: Merge Conflict During Pull / Conflict Khi Pull

**Giải thích**: Khi bạn chạy `git pull`, Git tự động fetch + merge. Nếu cùng file bị sửa trên cả local và remote → conflict xảy ra.

```bash
# You see this error
Auto-merging file.js
CONFLICT (content): Merge conflict in file.js
Automatic merge failed; fix conflicts and then commit the result.
```

**Solution / Giải pháp**:

```bash
# 1. Check which files have conflicts / Kiểm tra file nào bị conflict
git status

# 2. Resolve conflicts in each file (see Method 1 above)
# Xử lý conflict trong từng file (xem Method 1 ở trên)

# 3. Stage resolved files / Stage các file đã resolve
git add <resolved-files>

# 4. Complete the merge / Hoàn thành merge
git commit -m "Merge branch 'main' and resolve conflicts"

# 5. Push / Đẩy lên remote
git push origin main
```

---

### Scenario 3: Rebase Conflict / Conflict Khi Rebase

> 📖 **Rebase là gì?**: Rebase lấy tất cả commits của bạn, "tháo" chúng ra khỏi nhánh, di chuyển nhánh lên đầu mới nhất của nhánh đích, rồi "gắn" lại từng commit một. Điều này tạo ra lịch sử commit **tuyến tính**, sạch đẹp hơn merge.
>
> **Tại sao dùng rebase thay vì merge?**: Rebase giữ lịch sử commit sạch, không tạo merge commit thừa. Nhưng **không nên rebase** trên nhánh public mà nhiều người dùng chung.

```bash
git rebase main
# CONFLICT appears
```

> 📖 **`git rebase main`**: Di chuyển (re-apply) tất cả commits trên nhánh hiện tại lên trên đầu nhánh `main`.
>
> - Tại sao dùng: Để cập nhật nhánh feature của bạn với thay đổi mới nhất từ `main` mà không tạo merge commit.
> - Conflict có thể xảy ra tại **từng commit** trong quá trình rebase (khác với merge chỉ conflict một lần).

**Solution / Giải pháp**:

```bash
# 1. Resolve conflicts in files / Sửa conflict trong files

# 2. Stage resolved files / Stage file đã resolve
git add <resolved-files>

# 3. Continue rebase / Tiếp tục rebase
git rebase --continue
```

> 📖 **`git rebase --continue`**: Sau khi bạn resolve conflict của commit hiện tại, lệnh này **tiếp tục** quá trình rebase với commit tiếp theo. Nếu commit tiếp theo cũng conflict, bạn lại resolve → `--continue` → lặp lại cho đến hết.

```bash
# Or abort if you want to start over / Hoặc hủy bỏ nếu muốn bắt đầu lại
git rebase --abort
```

> 📖 **`git rebase --abort`**: **Hủy bỏ hoàn toàn** quá trình rebase, đưa nhánh về trạng thái trước khi rebase.
>
> - Tại sao dùng: Khi conflict quá phức tạp hoặc bạn nhận ra rebase không phải lựa chọn đúng. An toàn, không mất dữ liệu.

---

### Scenario 4: Cherry-pick Conflict / Conflict Khi Cherry-pick

> 📖 **Cherry-pick là gì?**: "Hái quả cherry" — chọn **một commit cụ thể** từ bất kỳ nhánh nào và áp dụng nó vào nhánh hiện tại, mà không cần merge toàn bộ nhánh.
>
> **Khi nào dùng?**: Khi bạn chỉ cần một tính năng hoặc fix từ nhánh khác, không muốn lấy toàn bộ thay đổi của nhánh đó.

```bash
git cherry-pick <commit-hash>
# CONFLICT appears
```

> 📖 **`git cherry-pick <commit-hash>`**: Áp dụng thay đổi từ commit có mã hash chỉ định vào nhánh hiện tại.
>
> - `<commit-hash>` là mã định danh duy nhất của commit (ví dụ: `a1b2c3d`). Bạn có thể tìm hash bằng `git log`.
> - Tại sao conflict: Nếu commit đó sửa code mà nhánh hiện tại cũng đã sửa → conflict.

**Solution / Giải pháp**:

```bash
# 1. Resolve conflicts / Giải quyết conflicts

# 2. Stage files / Stage file
git add <resolved-files>

# 3. Continue cherry-pick / Tiếp tục cherry-pick
git cherry-pick --continue
```

> 📖 **`git cherry-pick --continue`**: Hoàn tất cherry-pick sau khi resolve conflict. Git sẽ tạo commit mới với thay đổi từ commit gốc.

```bash
# Or abort / Hoặc hủy bỏ
git cherry-pick --abort
```

> 📖 **`git cherry-pick --abort`**: Hủy bỏ cherry-pick, quay về trạng thái trước khi chạy lệnh. An toàn, không mất dữ liệu.

---

## Prevention Strategies / Chiến Lược Phòng Tránh

### English Best Practices:

1. **Pull frequently**: Always pull before starting work

```bash
git pull origin main
```

> 📖 Tại sao: Giảm khả năng conflict bằng cách luôn làm việc trên phiên bản mới nhất. Khoảng cách giữa local và remote càng nhỏ → conflict càng ít và dễ resolve hơn.

2. **Commit often**: Make small, frequent commits

```bash
git add .
git commit -m "Small incremental change"
```

> 📖 Tại sao: Commits nhỏ giúp dễ dàng xác định vị trí conflict và resolve. Một commit lớn sửa 20 files sẽ khó resolve hơn nhiều so với 5 commits nhỏ mỗi cái sửa 4 files.

3. **Communicate**: Coordinate with team on who's working on what

> 📖 Tại sao: Phần lớn conflict nghiêm trọng xảy ra khi 2 người sửa cùng file mà không biết. Một cuộc trao đổi ngắn có thể tránh hàng giờ resolve conflict.

4. **Use feature branches**: Work on separate branches

```bash
git checkout -b feature/my-feature
```

> 📖 **`git checkout -b feature/my-feature`**: Tạo nhánh mới tên `feature/my-feature` và chuyển sang nhánh đó ngay lập tức.
>
> - `-b` = tạo nhánh mới (branch). Không có `-b` thì chỉ chuyển nhánh đã tồn tại.
> - Tại sao dùng: Mỗi tính năng/bug fix trên một nhánh riêng → không ảnh hưởng code trên `main` → giảm conflict.
> - Convention: `feature/` cho tính năng mới, `bugfix/` cho sửa bug, `hotfix/` cho fix khẩn cấp.

5. **Keep branches up to date**:

```bash
git checkout main
git pull
git checkout feature/my-feature
git merge main
```

> 📖 **Giải thích flow**:
>
> 1. `git checkout main` — Chuyển về nhánh `main`
> 2. `git pull` — Tải thay đổi mới nhất từ remote
> 3. `git checkout feature/my-feature` — Chuyển lại nhánh feature
> 4. `git merge main` — Merge code mới từ `main` vào feature branch
>
> Tại sao: Giữ nhánh feature cập nhật với `main` liên tục giúp conflicts nhỏ và dễ xử lý, thay vì đợi đến cuối rồi merge một lần với rất nhiều conflicts.

### Tiếng Việt - Thực Hành Tốt:

1. **Pull thường xuyên**: Luôn pull trước khi bắt đầu làm việc
2. **Commit thường xuyên**: Tạo các commits nhỏ, thường xuyên
3. **Giao tiếp**: Phối hợp với team về ai đang làm gì
4. **Dùng feature branches**: Làm việc trên các nhánh riêng
5. **Giữ branches cập nhật**: Thường xuyên merge main vào branch của bạn

---

## Useful Commands / Lệnh Hữu Ích

### Check conflict status / Kiểm tra trạng thái conflict

```bash
git status
```

> 📖 Hiển thị tổng quan: files bị conflict, files đã staged, files chưa tracked. Đây luôn là lệnh đầu tiên bạn nên chạy khi gặp vấn đề.

```bash
git diff
```

> 📖 **`git diff`**: Hiển thị sự khác biệt giữa **working directory** và **staging area** (dạng line-by-line). Cho bạn thấy chính xác dòng nào đã thay đổi, thêm, hoặc xóa.
>
> - Khi có conflict: hiển thị conflict markers trong file, giúp bạn xem conflict trước khi mở editor.

### View conflicts / Xem conflicts

```bash
git diff --name-only --diff-filter=U
```

> 📖 **`git diff --name-only --diff-filter=U`**: Liệt kê **chỉ tên** các file đang trong trạng thái **Unmerged** (conflict chưa được resolve).
>
> - `--name-only` = chỉ hiển thị tên file, không hiển thị nội dung diff.
> - `--diff-filter=U` = lọc chỉ lấy file có trạng thái `U` (Unmerged/conflict).
> - Tại sao dùng: Nhanh chóng biết còn bao nhiêu file cần resolve mà không bị "ngập" trong chi tiết diff.

### Abort merge / Hủy merge

```bash
git merge --abort
```

> 📖 **`git merge --abort`**: Hủy bỏ hoàn toàn quá trình merge đang dở, đưa tất cả files về trạng thái **trước khi merge**.
>
> - Tại sao dùng: Khi conflict quá nhiều hoặc phức tạp, bạn muốn "quay lại" và thử cách khác (ví dụ: merge từng phần, hoặc rebase thay vì merge).
> - An toàn: Không mất bất kỳ thay đổi nào trước khi merge.

### Abort rebase / Hủy rebase

```bash
git rebase --abort
```

> 📖 Tương tự `merge --abort`, đưa nhánh về trạng thái trước khi bắt đầu rebase. Tất cả commits gốc của bạn vẫn còn nguyên.

### View merge conflicts in a file / Xem conflicts trong file

```bash
git diff <file-name>
```

> 📖 Hiển thị diff cho **một file cụ thể**. Hữu ích khi bạn chỉ muốn tập trung vào conflict trong một file thay vì xem tất cả.

### List all conflicted files / Liệt kê tất cả files bị conflict

```bash
git diff --name-only --diff-filter=U
```

> 📖 (Giống lệnh "View conflicts" ở trên) — Liệt kê tất cả files còn conflict chưa resolve, giúp bạn biết còn bao nhiêu file cần xử lý.

---

## Conflict Resolution Workflow / Quy Trình Xử Lý Conflict

### English Workflow:

```
1. git pull origin main          ← Sync with remote
   ↓
2. CONFLICT detected             ← Git cannot auto-merge
   ↓
3. git status                    ← Identify conflicted files
   ↓
4. Open and edit conflicted files ← Fix the code
   ↓
5. Remove conflict markers (<<<, ===, >>>) ← Clean up markers
   ↓
6. git add <resolved-files>      ← Mark as resolved
   ↓
7. git commit -m "Resolve conflicts" ← Save the resolution
   ↓
8. git push origin main          ← Share with team
```

### Tiếng Việt - Quy Trình:

```
1. git pull origin main          ← Đồng bộ với remote
   ↓
2. Phát hiện CONFLICT            ← Git không thể tự merge
   ↓
3. git status                    ← Xác định files bị conflict
   ↓
4. Mở và sửa các files bị conflict ← Sửa code
   ↓
5. Xóa các dấu conflict (<<<, ===, >>>) ← Dọn dẹp markers
   ↓
6. git add <resolved-files>      ← Đánh dấu đã giải quyết
   ↓
7. git commit -m "Giải quyết conflicts" ← Lưu kết quả
   ↓
8. git push origin main          ← Chia sẻ với team
```

---

## Common Mistakes to Avoid / Lỗi Thường Gặp Cần Tránh

### English:

1. **Don't force push** unless absolutely necessary

```bash
# Dangerous - avoid this
git push -f origin main
```

> 📖 **`git push -f` (force push)**: Ghi đè lịch sử commit trên remote bằng lịch sử local, **bất kể** remote có gì.
>
> - ⚠️ Tại sao nguy hiểm: Nó XÓA commits của người khác trên remote mà không có cách khôi phục dễ dàng. Cả team sẽ bị ảnh hưởng.
> - Khi nào OK: Chỉ dùng trên nhánh **cá nhân** mà chỉ mình bạn làm việc (ví dụ: `feature/my-task`), **TUYỆT ĐỐI KHÔNG** dùng trên `main` hoặc `develop`.
> - Thay thế an toàn hơn: `git push --force-with-lease` — chỉ force push nếu remote chưa có commits mới từ người khác.

2. **Don't ignore conflicts** - they won't go away
3. **Don't delete conflict markers** without resolving
4. **Don't commit without testing** after resolving conflicts
5. **Don't panic** - conflicts are normal and fixable

### Tiếng Việt:

1. **Không force push** trừ khi thực sự cần thiết
2. **Không bỏ qua conflicts** - chúng sẽ không tự biến mất
3. **Không xóa dấu conflict** mà không giải quyết (xóa markers nhưng không chọn code đúng → code lỗi)
4. **Không commit mà không test** sau khi giải quyết conflicts (merge sai có thể gây bug ẩn)
5. **Không hoảng sợ** - conflicts là bình thường và có thể sửa được

---

## Emergency Recovery / Khôi Phục Khẩn Cấp

### If you mess up / Nếu bạn làm hỏng

**Undo last commit / Hoàn tác commit cuối**:

```bash
git reset --soft HEAD~1
```

> 📖 **`git reset --soft HEAD~1`**: Hủy commit cuối cùng nhưng **GIỮ NGUYÊN** tất cả thay đổi trong staging area.
>
> - `--soft` = chỉ di chuyển con trỏ HEAD, không đụng đến staging area hay working directory.
> - `HEAD~1` = commit trước HEAD (tức commit cuối cùng). `HEAD~2` = 2 commits trước, v.v.
> - Tại sao dùng: Khi bạn commit nhầm (sai message, thiếu file...) và muốn commit lại. Code vẫn giữ nguyên, chỉ "bỏ" commit.

**Discard all local changes / Hủy tất cả thay đổi local**:

```bash
git reset --hard HEAD
```

> 📖 **`git reset --hard HEAD`**: Đưa **TẤT CẢ** files về trạng thái của commit cuối cùng (HEAD). **XÓA VĨNH VIỄN** mọi thay đổi chưa commit.
>
> - `--hard` = reset cả working directory + staging area + HEAD pointer.
> - ⚠️ NGUY HIỂM: Không thể undo lệnh này cho các thay đổi chưa commit. Đảm bảo bạn thực sự muốn bỏ hết thay đổi.
> - Tại sao dùng: Khi bạn đã sửa lung tung và muốn bắt đầu lại từ đầu.

> 💡 **So sánh 3 loại reset**:
>
> | Flag | HEAD | Staging Area | Working Directory |
> | --- | --- | --- | --- |
> | `--soft` | ✅ Di chuyển | ❌ Giữ nguyên | ❌ Giữ nguyên |
> | `--mixed` (mặc định) | ✅ Di chuyển | ✅ Reset | ❌ Giữ nguyên |
> | `--hard` | ✅ Di chuyển | ✅ Reset | ✅ Reset (XÓA thay đổi) |

**Go back to before merge / Quay lại trước khi merge**:

```bash
git reset --hard ORIG_HEAD
```

> 📖 **`git reset --hard ORIG_HEAD`**: Quay lại trạng thái **ngay trước** thao tác merge/rebase cuối cùng.
>
> - `ORIG_HEAD` là tham chiếu đặc biệt mà Git tự động lưu trước khi thực hiện merge, rebase, hoặc reset. Nó hoạt động như một "checkpoint" tự động.
> - Tại sao dùng: Khi bạn đã merge xong nhưng nhận ra kết quả sai → quay lại ngay.

**Create backup branch / Tạo branch backup**:

```bash
git branch backup-branch
```

> 📖 **`git branch backup-branch`**: Tạo một nhánh mới tên `backup-branch` tại commit hiện tại, **nhưng không chuyển sang nhánh đó**.
>
> - Tại sao dùng: Tạo một "bản sao lưu" trước khi thực hiện thao tác nguy hiểm (rebase, reset --hard...). Nếu có gì sai, bạn luôn có thể quay lại bằng `git checkout backup-branch`.
> - 💡 **Mẹo hay**: Luôn tạo backup branch trước khi rebase hoặc merge phức tạp!

---

## Tools & Resources / Công Cụ & Tài Nguyên

### Recommended Tools / Công cụ đề xuất:

- **VS Code**: Built-in merge conflict resolver — tích hợp sẵn, dễ dùng, phổ biến nhất
- **GitKraken**: Visual Git client — giao diện đẹp, trực quan cho người mới
- **Sourcetree**: Free Git GUI — miễn phí từ Atlassian, tốt cho Windows/Mac
- **Meld**: Visual diff and merge tool — open source, 3-way merge
- **P4Merge**: Free merge tool — mạnh mẽ, hỗ trợ 3-way merge

### Useful Git Aliases / Git aliases hữu ích:

```bash
# Add to ~/.gitconfig
[alias]
    conflicts = diff --name-only --diff-filter=U
    resolve = !git add $(git conflicts)
```

> 📖 **Git Aliases là gì?**: Cho phép bạn tạo lệnh tắt cho các lệnh Git dài.
>
> - `git conflicts` = chạy `git diff --name-only --diff-filter=U` (liệt kê files conflict)
> - `git resolve` = chạy `git add` cho tất cả files conflict → stage tất cả file đã resolve
> - Thêm vào file `~/.gitconfig` (Linux/Mac) hoặc `C:\Users\<tên>\.gitconfig` (Windows)

---

## Quick Reference / Tham Khảo Nhanh

| Situation / Tình huống | Command / Lệnh | Giải thích |
| --- | --- | --- |
| Check conflicts | `git status` | Xem tổng quan trạng thái, files nào bị conflict |
| Accept theirs | `git checkout --theirs <file>` | Lấy toàn bộ nội dung từ nhánh incoming |
| Accept ours | `git checkout --ours <file>` | Giữ toàn bộ nội dung nhánh hiện tại |
| Abort merge | `git merge --abort` | Hủy merge, quay lại trạng thái trước |
| Continue after fix | `git add . && git commit` | Stage tất cả + tạo commit hoàn tất merge |
| View diff | `git diff <file>` | Xem chi tiết thay đổi trong file cụ thể |
| List conflict files | `git diff --name-only --diff-filter=U` | Liệt kê tất cả files đang conflict |
| Undo last commit | `git reset --soft HEAD~1` | Hủy commit cuối, giữ nguyên code |
| Discard everything | `git reset --hard HEAD` | Xóa hết thay đổi, về commit cuối |
| Safe force push | `git push --force-with-lease` | Force push an toàn (kiểm tra remote trước) |

---

## Tips for Team Collaboration / Mẹo Làm Việc Nhóm

### English:

1. Use pull requests for code review
2. Keep PRs small and focused
3. Merge main into your branch regularly
4. Communicate before working on same files
5. Use `.gitignore` properly
6. Write clear commit messages

### Tiếng Việt:

1. Sử dụng pull requests để review code — giúp phát hiện lỗi trước khi merge
2. Giữ PRs nhỏ và tập trung — PR càng lớn → conflict càng nhiều, review càng khó
3. Thường xuyên merge main vào branch của bạn — tránh "nợ conflict" chồng chất
4. Giao tiếp trước khi làm việc trên cùng files — phòng bệnh hơn chữa bệnh
5. Sử dụng `.gitignore` đúng cách — tránh conflict trên files không cần track (node_modules, .env, build...)
6. Viết commit messages rõ ràng — giúp hiểu tại sao thay đổi, dễ resolve conflict hơn

---

## Merge vs Rebase: Khi Nào Dùng Gì?

| Tiêu chí | `git merge` | `git rebase` |
| --- | --- | --- |
| **Lịch sử commit** | Giữ nguyên, tạo merge commit | Tuyến tính, sạch đẹp |
| **An toàn** | ✅ An toàn hơn | ⚠️ Viết lại lịch sử |
| **Nhánh public** | ✅ Nên dùng | ❌ Không nên |
| **Nhánh cá nhân** | ✅ OK | ✅ Ưu tiên (lịch sử sạch) |
| **Conflict** | Xử lý 1 lần | Có thể xử lý nhiều lần (từng commit) |
| **Khi nào dùng** | Merge feature vào main | Cập nhật feature branch với main |

> 💡 **Quy tắc vàng**: **Không bao giờ rebase** trên nhánh mà người khác cũng đang làm việc. Rebase viết lại lịch sử → gây khổ cho tất cả người dùng chung nhánh đó.
