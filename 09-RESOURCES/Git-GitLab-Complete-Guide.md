# Git & GitLab — Hướng Dẫn Kiến Thức Toàn Diện

> Tài liệu này **không chỉ liệt kê lệnh** — nó giải thích **tại sao**, **khi nào**, và **cách Git hoạt động** bên trong. Mục tiêu: đọc xong bạn **hiểu Git**, không chỉ **copy-paste lệnh**.

---

## Mục Lục

1. [Git Là Gì — Hiểu Đúng Bản Chất](#1-git-là-gì--hiểu-đúng-bản-chất)
2. [Kiến Trúc Bên Trong Git](#2-kiến-trúc-bên-trong-git)
3. [Thiết Lập & Cấu Hình](#3-thiết-lập--cấu-hình)
4. [Vòng Đời Của File Trong Git](#4-vòng-đời-của-file-trong-git)
5. [Branching — Linh Hồn Của Git](#5-branching--linh-hồn-của-git)
6. [Merge vs Rebase — Cuộc Tranh Luận Muôn Thuở](#6-merge-vs-rebase--cuộc-tranh-luận-muôn-thuở)
7. [Làm Việc Với Remote](#7-làm-việc-với-remote)
8. [Kỹ Thuật Nâng Cao](#8-kỹ-thuật-nâng-cao)
9. [GitLab — Nền Tảng DevOps](#9-gitlab--nền-tảng-devops)
10. [Git Workflows — Chọn Quy Trình Phù Hợp](#10-git-workflows--chọn-quy-trình-phù-hợp)
11. [Best Practices](#11-best-practices)
12. [Troubleshooting — Xử Lý Sự Cố](#12-troubleshooting--xử-lý-sự-cố)
13. [Cheat Sheet Nhanh](#13-cheat-sheet-nhanh)

---

# 1. Git Là Gì — Hiểu Đúng Bản Chất

## Vấn Đề Mà Git Giải Quyết

Trước Git, lập trình viên quản lý code kiểu:

```
📁 project_v1.zip
📁 project_v2_final.zip
📁 project_v2_final_THẬT.zip
📁 project_v2_final_THẬT_SỬA_LẦN_CUỐI.zip  😱
```

**Hậu quả:**
- Không biết ai sửa gì, khi nào
- Merge code = copy-paste thủ công, dễ đè nhau
- Muốn quay lại phiên bản cũ? Chúc may mắn

## Git Giải Quyết Thế Nào?

**Git là hệ thống quản lý phiên bản phân tán (DVCS).** Mỗi từ đều quan trọng:

| Từ khóa | Ý nghĩa |
|---------|---------|
| **Quản lý phiên bản** | Lưu lại MỌI thay đổi, ai sửa, khi nào, vì sao |
| **Phân tán** | Mỗi máy tính đều có **bản sao đầy đủ** của dự án + lịch sử — không phụ thuộc server trung tâm |

> 💡 **Khác biệt cốt lõi:** SVN (tập trung) cần kết nối server để xem lịch sử. Git thì không — tất cả lịch sử nằm ngay trên máy bạn.

## SVN vs Git — Hiểu Sự Khác Biệt

```
SVN (Tập trung):                    Git (Phân tán):
                                     
   ┌──────────┐                      ┌──────────┐
   │  Server  │                      │  Remote  │
   │ (nguồn   │                      │  (GitHub │
   │  sự thật)│                      │  GitLab) │
   └────┬─────┘                      └────┬─────┘
        │                            ┌────┴─────┐
   ┌────┼────┐                  ┌────┴──┐  ┌────┴──┐
   │    │    │                  │ Clone │  │ Clone │
   ▼    ▼    ▼                  │ ĐẦY   │  │ ĐẦY   │
  Dev  Dev  Dev                 │ ĐỦ    │  │ ĐỦ    │
  (chỉ (chỉ (chỉ               └───────┘  └───────┘
  file  file file                 Dev A      Dev B
  mới   mới  mới                (toàn bộ   (toàn bộ
  nhất) nhất) nhất)              lịch sử)   lịch sử)
```

**Lợi ích thực tế của phân tán:**
- **Làm việc offline:** Commit, tạo branch, xem log — tất cả không cần internet
- **Tốc độ:** Mọi thao tác chạy local → nhanh cực kỳ
- **An toàn:** Server sập? Mỗi dev đều có bản sao đầy đủ

---

# 2. Kiến Trúc Bên Trong Git

## Ba Khu Vực — Mental Model Quan Trọng Nhất

Đây là kiến thức **cốt lõi** — hiểu điều này thì mọi lệnh Git đều trở nên logic:

```
┌─────────────────┐    git add     ┌──────────────┐   git commit   ┌────────────────┐
│  Working        │ ────────────►  │   Staging     │ ────────────►  │   Repository   │
│  Directory      │                │   Area        │                │   (.git/)      │
│                 │  ◄────────────  │   (Index)     │                │                │
│  (Nơi bạn      │  git restore   │  (Nơi chuẩn   │                │  (Nơi lưu      │
│   viết code)   │                │   bị commit)  │                │   lịch sử)     │
└─────────────────┘                └──────────────┘                └────────────────┘
```

### Tại sao cần Staging Area?

Nhiều người mới thắc mắc: "Sao không commit thẳng, thêm bước staging làm gì?"

**Ví dụ thực tế:** Bạn đang fix bug, đồng thời phát hiện thêm 1 lỗi CSS. Bạn sửa cả 2. Nhưng muốn **commit riêng** từng cái để lịch sử rõ ràng:

```bash
# Chỉ add file liên quan đến bug fix
git add src/auth.js
git commit -m "fix(auth): xử lý lỗi null khi login"

# Sau đó add file CSS
git add src/styles.css
git commit -m "style(header): sửa lỗi tràn text"
```

> 💡 Staging Area cho phép bạn **kiểm soát chính xác** cái gì vào commit nào — giống như bạn chọn món trước khi gọi, không phải gọi cả menu.

## Git Lưu Dữ Liệu Thế Nào? — Snapshot, Không Phải Delta

**Hầu hết VCS cũ:** lưu **sự khác biệt (delta)** giữa các phiên bản.
**Git:** chụp **ảnh toàn bộ (snapshot)** trạng thái dự án tại mỗi commit.

```
VCS cũ (Delta):               Git (Snapshot):

v1: File A (original)          Commit 1: [A₁] [B₁] [C₁]
v2: File A (diff +3 lines)     Commit 2: [A₁] [B₂] [C₁]  ← A₁, C₁ không đổi → Git dùng con trỏ
v3: File A (diff -1 line)      Commit 3: [A₂] [B₂] [C₂]  ← toàn bộ thay đổi
```

> 💡 Nếu file không thay đổi, Git **không lưu lại** — chỉ tạo 1 con trỏ đến snapshot trước. Nên Git rất nhanh và tiết kiệm dung lượng.

## Commit — Đơn Vị Cốt Lõi

Mỗi commit là **1 snapshot** + metadata:

```
┌──────────────────────────────┐
│        Commit abc123         │
├──────────────────────────────┤
│ Tree:     (snapshot files)   │ ← Con trỏ đến cây file
│ Parent:   xyz789             │ ← Commit trước đó
│ Author:   Anh <anh@mail.com>│
│ Date:     2026-03-22         │
│ Message:  "fix: login bug"   │
│ SHA-1:    abc123def456...    │ ← Mã hash duy nhất
└──────────────────────────────┘
```

**SHA-1 hash** được tính từ nội dung file + metadata → nếu thay đổi dù 1 bit, hash sẽ khác hoàn toàn. Đây là cách Git đảm bảo **tính toàn vẹn dữ liệu**.

---

# 3. Thiết Lập & Cấu Hình

## Cài Đặt Git

| Hệ điều hành | Cách cài |
|--------------|----------|
| **Windows** | Tải từ [git-scm.com](https://git-scm.com/download/win) hoặc `winget install Git.Git` |
| **macOS** | `brew install git` (cần Homebrew) hoặc `xcode-select --install` |
| **Linux** | `sudo apt install git` (Debian/Ubuntu) / `sudo dnf install git` (Fedora) |

Kiểm tra: `git --version`

## Cấu Hình — 3 Tầng Ưu Tiên

Git có 3 cấp cấu hình, **cấp thấp ghi đè cấp cao**:

```
┌──────────────────────────────────────────┐
│  --system   │ /etc/gitconfig             │  Toàn hệ thống (ít dùng)
├──────────────────────────────────────────┤
│  --global   │ ~/.gitconfig               │  Cho user hiện tại ← thường dùng nhất
├──────────────────────────────────────────┤
│  --local    │ .git/config (trong repo)   │  Cho riêng 1 dự án ← ghi đè global
└──────────────────────────────────────────┘
```

**Cấu hình bắt buộc khi mới cài Git:**

```bash
git config --global user.name "Tên Của Bạn"
git config --global user.email "email@example.com"
```

> ⚠️ **Hai dòng trên là BẮT BUỘC.** Git từ chối commit nếu thiếu. Email này sẽ hiển thị công khai trong lịch sử commit.

**Cấu hình nên thiết lập:**

```bash
# Editor mặc định khi Git cần bạn nhập text (commit message, rebase...)
git config --global core.editor "code --wait"   # VS Code

# Tên branch mặc định khi git init
git config --global init.defaultBranch main

# Bật màu cho output
git config --global color.ui auto

# Xem toàn bộ config hiện tại
git config --list --show-origin   # Show cả file nguồn
```

## SSH Key — Xác Thực Không Cần Mật Khẩu

**Tại sao SSH?** Khi dùng HTTPS, mỗi lần push/pull bạn phải nhập username + password (hoặc token). SSH giải quyết bằng **cặp khóa mật mã**:

```
┌──────────┐     Mã hóa bằng       ┌──────────┐
│Private Key│    Public Key         │  GitLab  │
│(máy bạn, │ ◄──────────────────►  │  (lưu    │
│ GIỮ BÍ MẬT)│                     │  Public  │
└──────────┘                        │  Key)    │
                                    └──────────┘
```

```bash
# 1. Tạo cặp khóa (Ed25519 — thuật toán hiện đại, an toàn hơn RSA)
ssh-keygen -t ed25519 -C "email@example.com"
# Nhấn Enter 3 lần để dùng đường dẫn mặc định và không đặt passphrase

# 2. Copy public key
cat ~/.ssh/id_ed25519.pub
# Copy nội dung → dán vào GitLab/GitHub > Settings > SSH Keys

# 3. Test kết nối
ssh -T git@gitlab.com
ssh -T git@github.com
```

> 💡 **Private Key = chìa khóa vàng.** Tuyệt đối không chia sẻ file `id_ed25519` (không có `.pub`).

---

# 4. Vòng Đời Của File Trong Git

## 4 Trạng Thái

```
    Untracked ───► Staged ───► Committed ───► Modified
        │            │                           │
        │         git add                    (sửa file)
        │                                       │
        └────────────────────────────────────────┘
                     git add (lại)
```

| Trạng thái | Nghĩa là | Trong `git status` |
|-----------|----------|-------------------|
| **Untracked** | File mới, Git chưa biết | Đỏ, dưới "Untracked files" |
| **Modified** | File đã được Git theo dõi, vừa bị sửa | Đỏ, dưới "Changes not staged" |
| **Staged** | Đã đánh dấu sẵn sàng cho commit tiếp theo | Xanh, dưới "Changes to be committed" |
| **Committed** | Đã an toàn trong lịch sử | KHÔNG hiện trong `git status` |

## Các Thao Tác Cơ Bản Theo Workflow

### Khởi tạo Repository

```bash
# Tạo repo mới từ thư mục hiện tại
git init

# HOẶC clone repo có sẵn
git clone https://gitlab.com/user/repo.git

# Clone chỉ lịch sử gần nhất (nhanh hơn cho repo lớn)
git clone --depth 1 https://gitlab.com/user/repo.git
```

### Workflow hàng ngày

```bash
# 1. Xem trạng thái — LUÔN dùng trước mọi thao tác
git status

# 2. Xem cụ thể đã thay đổi GÌ
git diff                    # Thay đổi chưa staged
git diff --staged           # Thay đổi đã staged, sắp commit

# 3. Chọn file để commit
git add file.txt            # Thêm 1 file cụ thể
git add src/                # Thêm cả thư mục
git add .                   # Thêm TẤT CẢ (cẩn thận!)

# 4. Commit — lưu vào lịch sử
git commit -m "feat(auth): thêm chức năng đăng nhập"

# 5. Xem lịch sử
git log --oneline --graph   # Dạng cây, ngắn gọn
```

### Hủy thay đổi — Biết khi nào dùng gì

| Tình huống | Lệnh | Giải thích |
|-----------|------|-----------|
| Sửa file rồi muốn bỏ (chưa add) | `git restore file.txt` | Khôi phục về commit cuối |
| Đã `git add` nhưng muốn unstage | `git restore --staged file.txt` | Đưa ngược về Working Directory |
| Đã commit, muốn sửa message | `git commit --amend -m "message mới"` | Thay đổi commit cuối |
| Đã commit, muốn uncommit (giữ code) | `git reset --soft HEAD~1` | Code vẫn ở Staging |
| Đã commit, muốn xóa sạch | `git reset --hard HEAD~1` | ⚠️ MẤT CODE, không khôi phục |

### Stash — "Tạm Cất" Công Việc Dang Dở

**Tình huống:** Đang code feature, đột nhiên sếp bảo fix bug gấp. Chưa muốn commit code dở → **stash**.

```bash
git stash                       # Cất hết thay đổi vào "ngăn kéo"
# ... chuyển branch, fix bug, commit ...
git stash pop                   # Lấy lại code đã cất

# Xem danh sách các stash
git stash list
# stash@{0}: WIP on feature: abc123 đang làm login
# stash@{1}: WIP on main: def456 thử nghiệm CSS

# Lấy stash cụ thể (không xóa khỏi list)
git stash apply stash@{1}
```

> 💡 `stash pop` = lấy ra + xóa khỏi list. `stash apply` = lấy ra + giữ lại trong list.

---

# 5. Branching — Linh Hồn Của Git

## Branch Là Gì Thực Sự?

Nhiều người nghĩ branch = "copy cả dự án". **Sai.**

> **Branch trong Git chỉ là 1 con trỏ (40 ký tự) trỏ đến 1 commit.** Tạo branch = tạo 1 file 41 byte. Cực kỳ nhẹ.

```
                     main
                       ↓
  A ← B ← C ← D ← E (commit)
                  ↑
               feature
```

Khi bạn tạo branch `feature` từ commit `D`, Git chỉ tạo 1 con trỏ mới trỏ đến `D`. Không copy gì cả.

## HEAD — "Bạn Đang Ở Đâu?"

`HEAD` là con trỏ đặc biệt cho biết **bạn đang đứng ở branch/commit nào**.

```
Trước checkout:          Sau git checkout feature:

HEAD → main                HEAD → feature
         ↓                          ↓
  A ← B ← C                 A ← B ← C
                                    ↑
                                  main
```

## Thao Tác Với Branch

```bash
# Xem tất cả branch
git branch              # Local
git branch -a           # Local + remote

# Tạo + chuyển sang branch mới (1 lệnh)
git switch -c feature/login        # Cách mới (Git 2.23+)
git checkout -b feature/login      # Cách cũ (vẫn hoạt động)

# Chuyển branch
git switch main

# Xóa branch đã merge
git branch -d feature/login

# Xóa branch chưa merge (cẩn thận!)
git branch -D feature/login

# Xóa branch trên remote
git push origin --delete feature/login
```

---

# 6. Merge vs Rebase — Cuộc Tranh Luận Muôn Thuở

Đây là quyết định quan trọng nhất khi làm việc nhóm. Hiểu rõ để chọn đúng.

## Merge — Giữ Nguyên Lịch Sử

```
Trước merge:                    Sau git merge feature:

main:    A ← B ← C             main:    A ← B ← C ← ← ← M (merge commit)
                                                          ↗
feature:      ← D ← E          feature:      ← D ← E ──┘
```

```bash
git checkout main
git merge feature
```

**Ưu điểm:** Lịch sử **trung thực** — thấy rõ ai tạo branch, merge khi nào
**Nhược điểm:** Lịch sử có thể rối với nhiều merge commit

## Rebase — Viết Lại Lịch Sử Cho "Sạch"

```
Trước rebase:                   Sau git rebase main (trên feature):

main:    A ← B ← C             main:    A ← B ← C
                                                    ↖
feature:      ← D ← E          feature:              ← D' ← E'
                                        (D, E được "replay" lên đầu C)
```

```bash
git checkout feature
git rebase main
```

**Ưu điểm:** Lịch sử **thẳng, sạch đẹp** — dễ đọc
**Nhược điểm:** **Viết lại lịch sử** → nguy hiểm nếu branch đã push

## Quy Tắc Vàng

> ⚠️ **KHÔNG BAO GIỜ rebase branch đã push và có người khác đang dùng.** Rebase tạo commit mới (D' ≠ D) → người khác sẽ gặp xung đột.

| Tình huống | Dùng gì |
|-----------|---------|
| Merge feature vào main (đội nhóm) | `merge --no-ff` |
| Cập nhật feature branch với main mới nhất (cá nhân) | `rebase` |
| Branch đã push, có người khác code trên đó | `merge` |
| Branch chỉ mình bạn dùng, chưa push | `rebase` |

## Interactive Rebase — Dọn Dẹp Commit Trước Khi Merge

**Tình huống:** Bạn có 5 commit lộn xộn. Muốn gom thành 2 commit sạch trước khi tạo MR.

```bash
git rebase -i HEAD~5     # Mở editor với 5 commit gần nhất
```

Editor sẽ hiện:

```
pick abc123 WIP: bắt đầu login
pick def456 fix typo
pick ghi789 WIP: tiếp tục login
pick jkl012 fix lỗi nhỏ
pick mno345 hoàn thành login
```

Bạn sửa thành:

```
pick abc123 WIP: bắt đầu login
squash def456 fix typo              ← gom vào commit trên
squash ghi789 WIP: tiếp tục login   ← gom vào commit trên
squash jkl012 fix lỗi nhỏ           ← gom vào commit trên
reword mno345 hoàn thành login      ← đổi message
```

| Lệnh | Tác dụng |
|------|---------|
| `pick` | Giữ nguyên commit |
| `squash` | Gom vào commit phía trên, giữ cả 2 message |
| `fixup` | Gom vào commit phía trên, **bỏ** message |
| `reword` | Giữ commit nhưng đổi message |
| `drop` | Xóa commit |

## Cherry-Pick — "Hái" Commit Cụ Thể

**Tình huống:** Branch `feature` có 10 commit, nhưng bạn chỉ cần **1 commit** cụ thể đưa sang `main`.

```bash
git checkout main
git cherry-pick abc123     # Chỉ copy commit abc123 sang main
```

---

# 7. Làm Việc Với Remote

## Hiểu Remote — "Bản Sao Trên Server"

```
┌─────────────┐                    ┌─────────────┐
│  Máy bạn    │    push ──────►    │   Remote    │
│  (local)    │    ◄────── pull    │  (GitLab/   │
│             │    ◄────── fetch   │   GitHub)   │
└─────────────┘                    └─────────────┘
```

### Fetch vs Pull — Sự Khác Biệt Quan Trọng

| | `git fetch` | `git pull` |
|--|-----------|-----------|
| **Làm gì** | Tải về thay đổi từ remote | Tải về **+ tự động merge** |
| **An toàn** | ✅ Không thay đổi code của bạn | ⚠️ Có thể gây conflict |
| **Khi nào dùng** | Muốn xem trước có gì mới | Chắc chắn muốn cập nhật |

```bash
# Fetch — an toàn, chỉ download
git fetch origin
git log main..origin/main   # Xem những commit mới chưa có ở local
git merge origin/main       # Quyết định merge thủ công

# Pull — nhanh, nhưng cần cẩn thận
git pull origin main         # = fetch + merge
git pull --rebase origin main  # = fetch + rebase (lịch sử sạch hơn)
```

### Push — Đẩy Code Lên Remote

```bash
# Push lần đầu (thiết lập tracking)
git push -u origin main     # -u = --set-upstream

# Push các lần sau
git push                    # Git nhớ remote nhờ -u ở trên

# Force push — KHI NÀO?
git push --force-with-lease  # An toàn hơn --force: kiểm tra không ai push trước
```

> ⚠️ **`--force` = ghi đè lịch sử remote.** Chỉ dùng trên branch **riêng** của bạn, KHÔNG BAO GIỜ dùng trên `main`.

---

# 8. Kỹ Thuật Nâng Cao

## Tags — Đánh Dấu Phiên Bản

Tag khác branch ở chỗ: **tag là con trỏ CỐ ĐỊNH** — luôn trỏ vào 1 commit, không di chuyển.

```bash
# Tạo annotated tag (khuyến khích — có metadata)
git tag -a v1.0.0 -m "Phiên bản 1.0.0 — ra mắt sản phẩm"

# Xem thông tin tag
git show v1.0.0

# Push tag lên remote
git push origin v1.0.0      # 1 tag cụ thể
git push --tags              # Tất cả tags
```

## Reflog — "Mạng Lưới An Toàn"

`reflog` ghi lại **MỌI thao tác** bạn làm trên HEAD — kể cả reset --hard.

```bash
git reflog
# abc123 HEAD@{0}: reset: moving to HEAD~1
# def456 HEAD@{1}: commit: feat: thêm login
# ghi789 HEAD@{2}: checkout: moving from main to feature

# Khôi phục commit đã "xóa" bằng reset --hard
git checkout -b recovered def456
```

> 💡 `reflog` là "tấm lưới" cứu bạn khỏi hầu hết sai lầm. Dữ liệu reflog giữ **90 ngày** mặc định.

## Git Bisect — Tìm Bug Bằng Nhị Phân

**Tình huống:** có bug nhưng không biết commit nào gây ra, từ 100 commit gần đây.

```bash
git bisect start
git bisect bad                    # Commit hiện tại có bug
git bisect good abc123            # Commit cũ không có bug

# Git tự checkout commit giữa → bạn test
git bisect good    # hoặc bad, tùy kết quả test

# Lặp lại cho đến khi Git tìm ra commit gây bug
# Chỉ cần ~7 bước cho 100 commits (log₂100 ≈ 7)

git bisect reset   # Quay lại trạng thái ban đầu
```

## Git Hooks — Tự Động Hóa

Hooks là **script tự động chạy** khi bạn thực hiện thao tác Git.

| Hook | Chạy khi | Dùng để |
|------|---------|---------|
| `pre-commit` | Trước mỗi commit | Chạy lint, format code |
| `commit-msg` | Sau khi nhập message | Kiểm tra format message |
| `pre-push` | Trước khi push | Chạy test |
| `post-merge` | Sau khi merge | Cài dependency mới |

**Ví dụ `pre-commit` hook:**
```bash
#!/bin/sh
# File: .git/hooks/pre-commit
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ Lint failed. Commit bị chặn."
  exit 1
fi
```

> 💡 Thực tế nên dùng **Husky** (Node.js) để quản lý hooks — dễ chia sẻ hooks với cả team.

---

# 9. GitLab — Nền Tảng DevOps

## GitLab vs GitHub — Chọn Cái Nào?

| Tiêu chí | GitLab | GitHub |
|---------|--------|--------|
| **CI/CD** | Tích hợp sẵn, mạnh mẽ | GitHub Actions (mới hơn, đang bắt kịp) |
| **Self-hosted** | ✅ Miễn phí (Community Edition) | ❌ Chỉ Enterprise |
| **DevOps tích hợp** | Đầy đủ: CI/CD, Registry, Monitoring, Security | Cần ghép thêm tools |
| **Cộng đồng open-source** | Nhỏ hơn | Rất lớn, nhiều repo public |
| **Tốt nhất cho** | Doanh nghiệp, self-hosted, DevOps pipeline | Open-source, cá nhân, cộng đồng |

> 💡 **Quy tắc ngón tay cái:** Cần tự host + DevOps tích hợp → GitLab. Cần cộng đồng lớn + open-source → GitHub. Nhiều công ty dùng cả hai.

## GitLab CI/CD — Hiểu Pipeline

Pipeline = **chuỗi công việc tự động** chạy khi bạn push code.

```
Push code → Build → Test → Deploy
               │       │       │
            (biên dịch) (unit, e2e)  (staging/prod)
```

### Cấu trúc file `.gitlab-ci.yml`

```yaml
# Định nghĩa các giai đoạn (chạy tuần tự)
stages:
  - build
  - test
  - deploy

# Mỗi job thuộc 1 stage
build-app:
  stage: build
  script:
    - npm install
    - npm run build
  artifacts:             # File truyền sang job sau
    paths:
      - dist/
    expire_in: 1 hour   # Tự xóa sau 1 giờ

test-unit:
  stage: test
  script:
    - npm run test
  dependencies:
    - build-app          # Lấy artifacts từ job build-app

deploy-production:
  stage: deploy
  script:
    - npm run deploy
  only:
    - main               # Chỉ deploy khi push vào main
  when: manual           # Yêu cầu bấm nút thủ công
  environment:
    name: production
    url: https://example.com
```

### Khái Niệm Quan Trọng

| Thuật ngữ | Ý nghĩa |
|----------|---------|
| **Stage** | Giai đoạn (build, test, deploy). Jobs cùng stage chạy **song song** |
| **Job** | Một tác vụ cụ thể. Chạy trong Docker container riêng |
| **Artifacts** | File/thư mục truyền giữa các jobs |
| **Cache** | Lưu dependencies (node_modules) giữa các lần chạy → nhanh hơn |
| **Runner** | Máy thực thi pipeline (GitLab cung cấp shared runner miễn phí) |
| **Environment** | Môi trường deploy (staging, production) — có URL và lịch sử deploy |

### Cache vs Artifacts — Hay Nhầm Lẫn

| | Cache | Artifacts |
|--|-------|-----------|
| **Mục đích** | Tăng tốc (không cần npm install lại) | Truyền kết quả giữa các jobs |
| **Chia sẻ** | Giữa các **lần chạy pipeline** | Giữa các **jobs trong cùng pipeline** |
| **Ví dụ** | `node_modules/`, `.npm/` | `dist/`, `coverage/` |

## Merge Request (MR) — Code Review Đúng Cách

MR (GitLab) = PR/Pull Request (GitHub) — là **cách chính thức** để đưa code vào main.

**Quy trình chuẩn:**

```
1. Tạo branch → 2. Code & commit → 3. Push → 4. Tạo MR
→ 5. Code review → 6. CI/CD pass → 7. Approve → 8. Merge
```

**MR Template nên có:**

```markdown
## Mô tả thay đổi
Giải thích ngắn gọn: thay đổi gì, tại sao.

## Loại thay đổi
- [ ] Bug fix
- [ ] Tính năng mới
- [ ] Breaking change
- [ ] Cập nhật documentation

## Checklist
- [ ] Code follow convention
- [ ] Tự review code rồi
- [ ] Đã viết test
- [ ] Document đã cập nhật

## Ảnh chụp màn hình (nếu có UI thay đổi)
```

---

# 10. Git Workflows — Chọn Quy Trình Phù Hợp

## So Sánh 3 Workflow Phổ Biến

### Git Flow — Cho Dự Án Release-based

```
main ─────────────────●────────────────●──── (production)
                     ↑ merge          ↑ merge
release/1.0 ────────●                 │
                   ↑ merge            │
develop ──●──●──●──●──●──●──●──●──●──●──── (integration)
          ↑  ↑           ↑
feature/A ●  │       feature/C ●
          feature/B ●
```

| Branch | Vai trò | Tồn tại |
|--------|---------|---------|
| `main` | Production — luôn ổn định | Vĩnh viễn |
| `develop` | Tích hợp feature | Vĩnh viễn |
| `feature/*` | Phát triển tính năng | Tạm thời |
| `release/*` | Chuẩn bị release | Tạm thời |
| `hotfix/*` | Fix bug khẩn cấp trên production | Tạm thời |

**Phù hợp:** Dự án lớn, release theo đợt (v1.0, v2.0), cần kiểm soát chặt.

### GitHub Flow — Đơn Giản & Hiệu Quả

```
main ──●──●──●──●──●──●──── (luôn deploy được)
       ↑     ↑     ↑
    feature feature feature
    (MR)    (MR)    (MR)
```

**Quy tắc duy nhất:** `main` luôn deployable. Mọi thay đổi đều qua MR.

**Phù hợp:** Ứng dụng web, CI/CD tự động, deploy liên tục (SaaS).

### Trunk-Based Development — Tốc Độ Tối Đa

```
main ──●──●──●──●──●──●──── (deploy nhiều lần/ngày)
       ↑  ↑  ↑
      (branch ngắn, < 1 ngày, merge ngay)
```

**Phù hợp:** Team giỏi, CI/CD mạnh, cần ship nhanh. Google & Facebook dùng cách này.

### Cách Chọn?

| Tiêu chí | Git Flow | GitHub Flow | Trunk-Based |
|---------|---------|------------|------------|
| **Độ phức tạp** | Cao | Thấp | Rất thấp |
| **Tốc độ ship** | Chậm | Nhanh | Rất nhanh |
| **Team size** | Lớn | Mọi size | Mọi size (cần giỏi) |
| **Release cycle** | Theo đợt | Liên tục | Nhiều lần/ngày |
| **Yêu cầu CI/CD** | Không bắt buộc | Nên có | **Bắt buộc** |

---

# 11. Best Practices

## Viết Commit Message — Nghệ Thuật Bị Đánh Giá Thấp

### Quy Ước Conventional Commits

```
<type>(<scope>): <subject>

[body - giải thích WHY, không phải WHAT]

[footer - breaking changes, issue references]
```

| Type | Khi nào | Ví dụ |
|------|---------|-------|
| `feat` | Thêm tính năng | `feat(cart): thêm chức năng mã giảm giá` |
| `fix` | Sửa bug | `fix(auth): xử lý lỗi null khi token hết hạn` |
| `refactor` | Tái cấu trúc (không thêm/sửa gì) | `refactor(api): tách service layer` |
| `docs` | Chỉ thay đổi Documentation | `docs(readme): thêm hướng dẫn cài đặt` |
| `style` | Format, dấu chấm phẩy, whitespace | `style(global): chạy prettier` |
| `test` | Thêm/sửa test | `test(auth): thêm test cho login flow` |
| `chore` | Cập nhật tooling, config | `chore(deps): cập nhật React lên 19.1` |
| `perf` | Cải thiện hiệu năng | `perf(query): thêm index cho bảng users` |

**❌ BAD:**
```
git commit -m "fix"
git commit -m "update"
git commit -m "sửa lỗi"
git commit -m "wip"
```

**✅ GOOD:**
```
git commit -m "fix(payment): prevent double charge when user clicks submit twice"
git commit -m "feat(search): add fuzzy matching for Vietnamese diacritics"
```

> 💡 Commit message tốt = **chạy `git log --oneline` vẫn hiểu dự án đang phát triển gì** mà không cần đọc code.

## .gitignore — Biết Cái Gì Không Nên Commit

**Nguyên tắc:** Không commit file **sinh ra tự động** hoặc **chứa thông tin bí mật**.

| Loại file | Ví dụ | Lý do |
|----------|-------|-------|
| **Dependencies** | `node_modules/`, `venv/` | Tái tạo bằng `npm install` |
| **Build output** | `dist/`, `build/`, `*.pyc` | Tái tạo bằng build command |
| **Secrets** | `.env`, `*.key`, `credentials.json` | ⚠️ **BẢO MẬT** |
| **OS files** | `.DS_Store`, `Thumbs.db` | Không liên quan đến code |
| **IDE** | `.vscode/settings.json`, `.idea/` | Config cá nhân |

## Đặt Tên Branch

```
<type>/<ticket-id>-<mô-tả-ngắn>

Ví dụ:
feature/AUTH-42-google-oauth
bugfix/PAY-156-duplicate-charge
hotfix/SEC-001-sql-injection
```

---

# 12. Troubleshooting — Xử Lý Sự Cố

## Merge Conflict — Hiểu Và Xử Lý

**Conflict xảy ra khi:** 2 người sửa **cùng dòng** trong cùng file.

```
<<<<<<< HEAD (thay đổi của bạn)
const color = "blue";
=======
const color = "red";
>>>>>>> feature/new-theme (thay đổi của họ)
```

**Cách xử lý:**
1. Mở file conflict → chọn giữ code nào (hoặc kết hợp cả 2)
2. Xóa các dòng `<<<<`, `====`, `>>>>`
3. `git add file.txt` → `git commit`

**Cách TRÁNH conflict:**
- Pull thường xuyên (`git pull --rebase`)
- Branch nhỏ, merge nhanh
- Giao tiếp với team: "Mình đang sửa file X"

## Khôi Phục Sau Sai Lầm

| Sai lầm | Cách khôi phục |
|---------|---------------|
| Commit nhầm branch | `git cherry-pick <hash>` sang branch đúng, rồi `reset` branch sai |
| Reset --hard mất code | `git reflog` → tìm hash → `git checkout -b recovery <hash>` |
| Push code sai lên remote | `git revert <hash>` (tạo commit đảo ngược, an toàn) |
| Xóa branch nhầm | `git reflog` → tìm commit cuối của branch → tạo lại |
| File quá lớn trong lịch sử | `git filter-repo` hoặc BFG Repo-Cleaner |

> 💡 **`git revert` vs `git reset`:** `revert` tạo commit mới undo, AN TOÀN cho branch đã push. `reset` xóa commit, CHỈ dùng cho branch chưa push.

## Đồng Bộ Fork

```bash
# Thêm upstream (repo gốc)
git remote add upstream https://github.com/original/repo.git

# Cập nhật fork
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

# 13. Cheat Sheet Nhanh

## Lệnh Hàng Ngày

```bash
git status                          # Xem trạng thái
git add .                           # Stage tất cả
git commit -m "type: message"      # Commit
git push                           # Đẩy lên remote
git pull --rebase                  # Cập nhật từ remote
git switch -c feature/name         # Tạo branch mới
git log --oneline --graph -20      # Xem lịch sử 20 commit
```

## Lệnh "Cứu Mạng"

```bash
git stash                          # Tạm cất code
git reflog                         # Xem mọi thao tác đã làm
git reset --soft HEAD~1            # Uncommit (giữ code)
git restore file.txt               # Bỏ thay đổi chưa stage
git revert HEAD                    # Undo commit đã push
git bisect start                   # Tìm commit gây bug
```

## Git Aliases — Tiết Kiệm Thời Gian

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all -20"
git config --global alias.last "log -1 HEAD"
git config --global alias.undo "reset --soft HEAD~1"
```

---

## Tài Nguyên Tham Khảo

| Loại | Link | Mô tả |
|------|------|-------|
| 📖 Sách | [Pro Git Book](https://git-scm.com/book/en/v2) | Miễn phí, đầy đủ nhất |
| 📖 Docs | [Git Documentation](https://git-scm.com/doc) | Tài liệu chính thức |
| 🎮 Trực quan | [Learn Git Branching](https://learngitbranching.js.org/) | Game học Git branching |
| 📖 Docs | [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) | Hướng dẫn CI/CD |
| 📋 Quy ước | [Conventional Commits](https://www.conventionalcommits.org/) | Chuẩn commit message |

---

> **Ghi nhớ cuối cùng:** Git là công cụ mạnh mẽ — nhưng sức mạnh đi kèm trách nhiệm. **Luôn cẩn thận** với `--force`, `--hard`, và `filter-branch`. Và khi hoang mang, `git reflog` là bạn thân nhất của bạn. 🚀
