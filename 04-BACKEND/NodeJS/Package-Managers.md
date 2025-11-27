# Package Managers

## 📌 Mục lục

1. [Package Manager là gì?](#1-package-manager-là-gì)
2. [Tại sao cần Package Manager?](#2-tại-sao-cần-package-manager)
3. [Các Package Managers phổ biến](#3-các-package-managers-phổ-biến)
   - [npm](#31-npm-node-package-manager)
   - [Yarn](#32-yarn)
   - [pnpm](#33-pnpm)
   - [Bun](#34-bun)
4. [So sánh chi tiết](#4-so-sánh-chi-tiết)
5. [Khi nào nên dùng Package Manager nào?](#5-khi-nào-nên-dùng-package-manager-nào)
6. [Best Practices](#6-best-practices)

---

## 1. Package Manager là gì?

**Package Manager** (Trình quản lý gói) là công cụ tự động hóa quá trình:

- **Cài đặt** (install) các thư viện/dependencies
- **Cập nhật** (update) phiên bản packages
- **Gỡ bỏ** (uninstall) packages không cần thiết
- **Quản lý phiên bản** (version management) của dependencies
- **Chia sẻ** code giữa các dự án

### Hình dung đơn giản

```
📦 Package Manager giống như một "siêu thị phần mềm"
   - Bạn cần gì? → Tìm kiếm → Thêm vào giỏ hàng (install)
   - Tự động xử lý các dependencies (phụ thuộc)
   - Quản lý "hóa đơn" (package.json) để biết đã mua gì
```

---

## 2. Tại sao cần Package Manager?

### ❌ Không có Package Manager

```
1. Tải thủ công từng thư viện
2. Copy vào project
3. Tự quản lý phiên bản
4. Tự xử lý dependencies của dependencies
5. Cập nhật thủ công khi có version mới
→ Mất thời gian, dễ sai sót, khó maintain
```

### ✅ Có Package Manager

```
1. Chạy 1 lệnh: npm install lodash
2. Tự động tải và cài đặt
3. Tự động quản lý dependencies
4. Dễ dàng update: npm update
5. Chia sẻ dự án: chỉ cần package.json
→ Nhanh, chính xác, dễ maintain
```

---

## 3. Các Package Managers phổ biến

### 3.1 npm (Node Package Manager)

#### 📖 Giới thiệu

- **Ra đời**: 2010
- **Mặc định** đi kèm với Node.js
- **Registry**: npmjs.com (>2 triệu packages)
- **Ngôn ngữ**: JavaScript

#### 🔧 Các lệnh cơ bản

```bash
# Khởi tạo project
npm init
npm init -y                    # Khởi tạo với default values

# Cài đặt packages
npm install                    # Cài tất cả dependencies từ package.json
npm install <package>          # Cài package vào dependencies
npm install <package> -D       # Cài vào devDependencies
npm install <package> -g       # Cài global

# Gỡ cài đặt
npm uninstall <package>
npm uninstall <package> -g     # Gỡ global package

# Cập nhật
npm update                     # Update tất cả packages
npm update <package>           # Update package cụ thể

# Kiểm tra
npm list                       # Liệt kê packages đã cài
npm outdated                   # Kiểm tra packages cần update
npm audit                      # Kiểm tra lỗ hổng bảo mật

# Chạy scripts
npm run <script-name>
npm start                      # Chạy script "start"
npm test                       # Chạy script "test"
```

#### ✅ Ưu điểm

| Ưu điểm               | Mô tả                                 |
| --------------------- | ------------------------------------- |
| **Mặc định**          | Đi kèm Node.js, không cần cài thêm    |
| **Cộng đồng lớn**     | Tài liệu phong phú, hỗ trợ tốt        |
| **Registry khổng lồ** | >2 triệu packages                     |
| **npx**               | Chạy packages mà không cần cài global |
| **Workspaces**        | Hỗ trợ monorepo (từ npm 7+)           |

#### ❌ Nhược điểm

| Nhược điểm       | Mô tả                                           |
| ---------------- | ----------------------------------------------- |
| **Tốc độ**       | Chậm hơn yarn, pnpm                             |
| **Disk space**   | Tốn dung lượng (duplicate packages)             |
| **node_modules** | Cấu trúc phẳng, có thể gây phantom dependencies |

#### 📁 Cấu trúc files

```
project/
├── node_modules/          # Thư mục chứa packages
├── package.json           # Manifest file
└── package-lock.json      # Lock file (đảm bảo version consistency)
```

---

### 3.2 Yarn

#### 📖 Giới thiệu

- **Ra đời**: 2016 (bởi Facebook)
- **Mục đích**: Khắc phục nhược điểm của npm
- **Phiên bản**: Yarn Classic (1.x) và Yarn Berry (2.x+)

#### 🔧 Các lệnh cơ bản

```bash
# Khởi tạo project
yarn init
yarn init -y

# Cài đặt packages
yarn                           # = npm install
yarn add <package>             # = npm install <package>
yarn add <package> -D          # = npm install <package> -D
yarn global add <package>      # = npm install <package> -g

# Gỡ cài đặt
yarn remove <package>

# Cập nhật
yarn upgrade                   # Update tất cả
yarn upgrade <package>         # Update package cụ thể
yarn upgrade-interactive       # Interactive mode

# Kiểm tra
yarn list
yarn outdated
yarn audit

# Chạy scripts
yarn <script-name>             # Không cần "run"
yarn start
yarn test
```

#### ✅ Ưu điểm

| Ưu điểm               | Mô tả                                |
| --------------------- | ------------------------------------ |
| **Tốc độ**            | Nhanh hơn npm (parallel downloads)   |
| **Offline mode**      | Cache packages, cài offline được     |
| **Deterministic**     | yarn.lock đảm bảo cài đặt giống nhau |
| **Workspaces**        | Hỗ trợ monorepo tốt                  |
| **Plug'n'Play (PnP)** | Yarn Berry: không cần node_modules   |

#### ❌ Nhược điểm

| Nhược điểm        | Mô tả                                 |
| ----------------- | ------------------------------------- |
| **Cài đặt riêng** | Cần cài thêm, không đi kèm Node.js    |
| **Yarn Berry**    | Breaking changes, learning curve      |
| **Compatibility** | Một số packages không tương thích PnP |

#### 📁 Cấu trúc files

```
# Yarn Classic
project/
├── node_modules/
├── package.json
└── yarn.lock

# Yarn Berry (PnP)
project/
├── .yarn/
│   ├── cache/             # Cached packages
│   └── releases/          # Yarn binary
├── .pnp.cjs               # Plug'n'Play runtime
├── package.json
└── yarn.lock
```

---

### 3.3 pnpm

#### 📖 Giới thiệu

- **Ra đời**: 2017
- **Tên gọi**: Performant npm
- **Đặc điểm**: Content-addressable storage

#### 🔧 Các lệnh cơ bản

```bash
# Khởi tạo project
pnpm init

# Cài đặt packages
pnpm install                   # = npm install
pnpm add <package>             # = npm install <package>
pnpm add <package> -D          # = npm install <package> -D
pnpm add <package> -g          # = npm install <package> -g

# Gỡ cài đặt
pnpm remove <package>

# Cập nhật
pnpm update
pnpm update <package>

# Kiểm tra
pnpm list
pnpm outdated
pnpm audit

# Chạy scripts
pnpm <script-name>
pnpm start
pnpm test
```

#### 🎯 Cơ chế hoạt động đặc biệt

```
📦 Content-addressable storage

Thay vì:
project-a/node_modules/lodash/  (100MB)
project-b/node_modules/lodash/  (100MB)
→ Tổng: 200MB

pnpm làm:
~/.pnpm-store/lodash@4.17.21/   (100MB) ← Lưu 1 lần
project-a/node_modules/lodash   → symlink
project-b/node_modules/lodash   → symlink
→ Tổng: ~100MB (tiết kiệm 50%+)
```

#### ✅ Ưu điểm

| Ưu điểm            | Mô tả                                 |
| ------------------ | ------------------------------------- |
| **Tiết kiệm disk** | Symlinks, không duplicate packages    |
| **Tốc độ**         | Nhanh nhất trong các package managers |
| **Strict**         | Ngăn phantom dependencies             |
| **Monorepo**       | Hỗ trợ workspaces xuất sắc            |
| **Compatible**     | Tương thích npm ecosystem             |

#### ❌ Nhược điểm

| Nhược điểm         | Mô tả                           |
| ------------------ | ------------------------------- |
| **Symlinks**       | Một số tools không hỗ trợ tốt   |
| **Learning curve** | Cấu trúc node_modules khác biệt |
| **Windows**        | Có thể gặp issues với symlinks  |

#### 📁 Cấu trúc files

```
project/
├── node_modules/
│   ├── .pnpm/                 # Actual packages
│   │   └── lodash@4.17.21/
│   └── lodash -> .pnpm/...    # Symlink
├── package.json
└── pnpm-lock.yaml
```

---

### 3.4 Bun

#### 📖 Giới thiệu

- **Ra đời**: 2022
- **Đặc điểm**: All-in-one (runtime + package manager + bundler)
- **Ngôn ngữ**: Viết bằng Zig, tối ưu performance

#### 🔧 Các lệnh cơ bản

```bash
# Khởi tạo project
bun init

# Cài đặt packages
bun install                    # = npm install
bun add <package>              # = npm install <package>
bun add <package> -d           # = npm install <package> -D
bun add <package> -g           # = npm install <package> -g

# Gỡ cài đặt
bun remove <package>

# Cập nhật
bun update
bun update <package>

# Chạy scripts
bun run <script-name>
bun start
bun test

# Chạy file trực tiếp
bun run index.ts               # Chạy TypeScript không cần compile
```

#### ✅ Ưu điểm

| Ưu điểm        | Mô tả                                |
| -------------- | ------------------------------------ |
| **Cực nhanh**  | Nhanh hơn npm 10-100x                |
| **All-in-one** | Runtime + PM + Bundler + Test runner |
| **TypeScript** | Native support, không cần config     |
| **Compatible** | Tương thích npm packages             |
| **Hot reload** | Built-in hot reload                  |

#### ❌ Nhược điểm

| Nhược điểm       | Mô tả                      |
| ---------------- | -------------------------- |
| **Mới**          | Ecosystem chưa mature      |
| **Stability**    | Có thể có bugs             |
| **Windows**      | Hỗ trợ Windows còn hạn chế |
| **Node.js APIs** | Chưa implement đầy đủ 100% |

#### 📁 Cấu trúc files

```
project/
├── node_modules/
├── package.json
└── bun.lockb                  # Binary lock file
```

---

## 4. So sánh chi tiết

### 4.1 Bảng so sánh tổng quan

| Tiêu chí           | npm        | Yarn     | pnpm       | Bun        |
| ------------------ | ---------- | -------- | ---------- | ---------- |
| **Ra đời**         | 2010       | 2016     | 2017       | 2022       |
| **Tốc độ**         | ⭐⭐       | ⭐⭐⭐   | ⭐⭐⭐⭐   | ⭐⭐⭐⭐⭐ |
| **Disk usage**     | ⭐⭐       | ⭐⭐⭐   | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐   |
| **Stability**      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐   | ⭐⭐⭐     |
| **Cộng đồng**      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐     | ⭐⭐       |
| **Monorepo**       | ⭐⭐⭐     | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐     |
| **Learning curve** | Dễ         | Dễ       | Trung bình | Dễ         |

### 4.2 So sánh tốc độ (Benchmark)

```
Cài đặt project với ~1000 dependencies:

npm     ████████████████████████████████  45s
Yarn    ██████████████████████            28s
pnpm    ████████████████                  20s
Bun     ████████                          10s

* Kết quả có thể khác tùy môi trường
```

### 4.3 So sánh Disk Usage

```
10 projects cùng dependencies:

npm     ████████████████████████████████  5GB
Yarn    ████████████████████████████████  5GB
pnpm    ████████████                      1.5GB (symlinks)
Bun     ████████████████████████          3GB
```

### 4.4 So sánh Commands

| Tác vụ      | npm                  | Yarn              | pnpm              | Bun              |
| ----------- | -------------------- | ----------------- | ----------------- | ---------------- |
| Install all | `npm install`        | `yarn`            | `pnpm install`    | `bun install`    |
| Add package | `npm install pkg`    | `yarn add pkg`    | `pnpm add pkg`    | `bun add pkg`    |
| Add dev     | `npm install pkg -D` | `yarn add pkg -D` | `pnpm add pkg -D` | `bun add pkg -d` |
| Remove      | `npm uninstall pkg`  | `yarn remove pkg` | `pnpm remove pkg` | `bun remove pkg` |
| Update      | `npm update`         | `yarn upgrade`    | `pnpm update`     | `bun update`     |
| Run script  | `npm run dev`        | `yarn dev`        | `pnpm dev`        | `bun run dev`    |
| Execute     | `npx pkg`            | `yarn dlx pkg`    | `pnpm dlx pkg`    | `bunx pkg`       |

### 4.5 Lock Files

| Package Manager | Lock File           | Format        |
| --------------- | ------------------- | ------------- |
| npm             | `package-lock.json` | JSON          |
| Yarn            | `yarn.lock`         | Custom format |
| pnpm            | `pnpm-lock.yaml`    | YAML          |
| Bun             | `bun.lockb`         | Binary        |

---

## 5. Khi nào nên dùng Package Manager nào?

### 🎯 Chọn npm khi:

```
✅ Mới học Node.js/JavaScript
✅ Dự án nhỏ, đơn giản
✅ Cần stability cao nhất
✅ Team chưa quen với PM khác
✅ Không muốn cài thêm tools
```

### 🎯 Chọn Yarn khi:

```
✅ Cần tốc độ nhanh hơn npm
✅ Làm việc với monorepo
✅ Cần offline mode
✅ Muốn interactive upgrade
✅ Dự án Facebook/Meta ecosystem
```

### 🎯 Chọn pnpm khi:

```
✅ Disk space hạn chế
✅ Nhiều projects cùng dependencies
✅ Monorepo lớn
✅ Cần strict dependency management
✅ CI/CD cần tối ưu cache
```

### 🎯 Chọn Bun khi:

```
✅ Cần tốc độ cực nhanh
✅ Dự án mới, có thể chấp nhận risk
✅ Muốn all-in-one solution
✅ Làm việc với TypeScript nhiều
✅ Prototyping, side projects
```

### 📊 Decision Flowchart

```
                    ┌─────────────────┐
                    │  Bắt đầu chọn   │
                    │ Package Manager │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ Mới học Node.js?│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │ Yes          │              │ No
              ▼              │              ▼
        ┌─────────┐          │      ┌───────────────┐
        │   npm   │          │      │ Cần tốc độ    │
        └─────────┘          │      │ cực nhanh?    │
                             │      └───────┬───────┘
                             │              │
                             │   ┌──────────┼──────────┐
                             │   │ Yes      │          │ No
                             │   ▼          │          ▼
                             │ ┌─────┐      │   ┌─────────────┐
                             │ │ Bun │      │   │ Disk space  │
                             │ └─────┘      │   │ quan trọng? │
                             │              │   └──────┬──────┘
                             │              │          │
                             │              │   ┌──────┼──────┐
                             │              │   │ Yes  │      │ No
                             │              │   ▼      │      ▼
                             │              │ ┌──────┐ │  ┌──────┐
                             │              │ │ pnpm │ │  │ Yarn │
                             │              │ └──────┘ │  └──────┘
                             │              │          │
                             └──────────────┴──────────┘
```

---

## 6. Best Practices

### 6.1 Chung cho tất cả Package Managers

```bash
# ✅ Luôn commit lock file
git add package-lock.json  # hoặc yarn.lock, pnpm-lock.yaml

# ✅ Sử dụng exact versions trong production
npm install lodash@4.17.21 --save-exact

# ✅ Kiểm tra security thường xuyên
npm audit
yarn audit
pnpm audit

# ✅ Cập nhật dependencies định kỳ
npm outdated
npm update

# ✅ Sử dụng .npmrc để config
# .npmrc
save-exact=true
engine-strict=true
```

### 6.2 Cấu hình package.json tốt

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "lint": "eslint .",
    "build": "tsc"
  },
  "dependencies": {
    "express": "4.18.2"
  },
  "devDependencies": {
    "nodemon": "3.0.1",
    "jest": "29.7.0"
  }
}
```

### 6.3 Quản lý trong Team

```bash
# Thống nhất package manager trong team
# Tạo file .npmrc hoặc tương đương

# Sử dụng corepack (Node.js 16.10+)
corepack enable
corepack prepare pnpm@latest --activate

# Hoặc specify trong package.json
{
  "packageManager": "pnpm@8.10.0"
}
```

### 6.4 CI/CD Optimization

```yaml
# GitHub Actions example với pnpm
- name: Install pnpm
  uses: pnpm/action-setup@v2
  with:
    version: 8

- name: Get pnpm store directory
  id: pnpm-cache
  run: echo "dir=$(pnpm store path)" >> $GITHUB_OUTPUT

- name: Setup pnpm cache
  uses: actions/cache@v3
  with:
    path: ${{ steps.pnpm-cache.outputs.dir }}
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}

- name: Install dependencies
  run: pnpm install --frozen-lockfile
```

---

## 📚 Tổng kết

| Nếu bạn...                       | Chọn     |
| -------------------------------- | -------- |
| Mới bắt đầu                      | **npm**  |
| Cần balance tốc độ + stability   | **Yarn** |
| Cần tiết kiệm disk + strict deps | **pnpm** |
| Muốn bleeding edge + speed       | **Bun**  |

### 🔑 Key Takeaways

1. **npm** - Default, stable, đủ dùng cho hầu hết cases
2. **Yarn** - Nhanh hơn npm, offline mode, monorepo tốt
3. **pnpm** - Tiết kiệm disk nhất, strict, monorepo xuất sắc
4. **Bun** - Nhanh nhất, all-in-one, nhưng còn mới

> 💡 **Tip**: Không có "best" package manager. Chọn dựa trên nhu cầu cụ thể của dự án và team!

---

## 📖 Tài liệu tham khảo

- [npm Documentation](https://docs.npmjs.com/)
- [Yarn Documentation](https://yarnpkg.com/)
- [pnpm Documentation](https://pnpm.io/)
- [Bun Documentation](https://bun.sh/)
