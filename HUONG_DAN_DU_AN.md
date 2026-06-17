# 📱 Hướng Dẫn Triển Khai Dự Án PWA to APK Builder

> Hệ thống tự động biến trang web PWA thành file APK Android, sử dụng GitHub Actions + Cloudflare Workers + Bubblewrap Core — hoàn toàn miễn phí.

---

## 🗺️ Tổng Quan Kiến Trúc

```
Người dùng nhập URL
        ↓
Cloudflare Worker (nhận yêu cầu, lưu KV)
        ↓
GitHub Actions (build APK bằng @bubblewrap/core)
        ↓
GitHub Release (lưu file APK tạm thời)
        ↓
Webhook → Cloudflare Worker (cập nhật trạng thái)
        ↓
Frontend polling → Hiển thị link tải APK
```

---

## 📋 Yêu Cầu Trước Khi Bắt Đầu

- Tài khoản **GitHub** (miễn phí)
- Tài khoản **Cloudflare** (miễn phí)
- Trình duyệt web

---

## 🔧 PHẦN 1: Cài Đặt GitHub

### Bước 1.1 — Fork Repo Bubblewrap

1. Truy cập: `https://github.com/googlechromelabs/bubblewrap`
2. Click nút **"Fork"** góc trên phải
3. Bỏ tick **"Copy the main branch only"**
4. Click **"Create fork"**

> Kết quả: Mày có repo `tên-mày/bubblewrap`

---

### Bước 1.2 — Bật Quyền cho GitHub Actions

Vào repo fork → **Settings** → **Actions** → **General**

- Chọn **"Allow all actions and reusable workflows"** → **Save**
- Kéo xuống phần **"Workflow permissions"**
- Chọn **"Read and write permissions"** → **Save**

---

### Bước 1.3 — Tạo File Workflow

Trong repo fork, tạo file `.github/workflows/build-apk.yml`:

1. Click **"Add file"** → **"Create new file"**
2. Gõ đường dẫn: `.github/workflows/build-apk.yml`
3. Paste nội dung workflow (xem file `build-apk.yml` đính kèm)
4. Click **"Commit changes"**

---

### Bước 1.4 — Tạo GitHub Personal Access Token

1. Vào `github.com` → Click **avatar** → **Settings**
2. Kéo xuống cuối → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)**
4. **Generate new token (classic)**
5. Điền **Note**: `pwa-to-apk-builder`
6. **Expiration**: Chọn `No expiration`
7. Tick các quyền: ✅ `repo` và ✅ `workflow`
8. Click **"Generate token"**
9. **Copy token ngay** — chỉ hiện 1 lần!

---

### Bước 1.5 — Thêm Secrets cho Repo

Vào repo fork → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Thêm 3 secrets:

| Tên Secret | Giá Trị |
|---|---|
| `KEYSTORE_PASSWORD` | Mật khẩu tự đặt (VD: `MyApp@2024`) |
| `KEY_PASSWORD` | Giống `KEYSTORE_PASSWORD` |
| `WORKER_WEBHOOK_URL` | Điền sau khi tạo Cloudflare Worker |

---

## ☁️ PHẦN 2: Cài Đặt Cloudflare

### Bước 2.1 — Tạo KV Namespace

1. Vào `dash.cloudflare.com`
2. **Workers & Pages** → **KV**
3. Click **"Create namespace"**
4. Đặt tên: `PWA_BUILDS`
5. Click **"Add"**

---

### Bước 2.2 — Tạo Cloudflare Worker

1. **Workers & Pages** → **Create** → **Create Worker**
2. Đặt tên: `pwa-to-apk-worker`
3. Click **"Deploy"**
4. Copy URL dạng: `https://pwa-to-apk-worker.tên-mày.workers.dev`

---

### Bước 2.3 — Bind KV vào Worker

1. Vào worker vừa tạo → **Settings** → **Bindings**
2. Click **"Add"** → chọn **"KV Namespace"**
3. Điền:
   - Variable name: `PWA_BUILDS`
   - KV Namespace: `PWA_BUILDS`
4. Click **"Save"**

---

### Bước 2.4 — Paste Code vào Worker

1. Vào worker → **Edit Code**
2. Xóa hết code cũ
3. Paste code Worker (xem file `cloudflare-worker.js` đính kèm)
4. Sửa 2 dòng đầu:

```javascript
const GITHUB_REPO = 'tên-github-mày/bubblewrap';
// Token lưu trong biến môi trường Cloudflare (xem bước 2.5)
```

5. Click **"Deploy"**

---

### Bước 2.5 — Thêm GitHub Token vào Cloudflare

Vào worker → **Settings** → **Variables and Secrets** → **Add variable**

| Tên | Giá Trị |
|---|---|
| `GITHUB_TOKEN` | Token GitHub vừa tạo ở bước 1.4 |

---

### Bước 2.6 — Điền WORKER_WEBHOOK_URL vào GitHub

Quay lại GitHub repo → **Settings** → **Secrets** → Sửa secret `WORKER_WEBHOOK_URL`:

```
https://pwa-to-apk-worker.tên-mày.workers.dev
```

---

## 🌐 PHẦN 3: Triển Khai Frontend

### Bước 3.1 — Tạo File index.html

Tạo file `index.html` với code giao diện (xem file đính kèm).

Đảm bảo sửa biến `WORKER_URL` trong file:

```javascript
const WORKER_URL = 'https://pwa-to-apk-worker.tên-mày.workers.dev';
```

---

### Bước 3.2 — Upload lên Cloudflare Pages

1. Vào **Cloudflare Dashboard** → **Workers & Pages** → **Create**
2. Chọn **Pages** → **Upload assets**
3. Đặt tên project: `pwa-to-apk`
4. Upload file `index.html`
5. Click **"Deploy site"**

> Kết quả: Giao diện web chạy tại `https://pwa-to-apk.pages.dev`

---

## ✅ PHẦN 4: Kiểm Tra Hệ Thống

### Bước 4.1 — Test Worker hoạt động

Mở trình duyệt, truy cập:

```
https://pwa-to-apk-worker.tên-mày.workers.dev/status?buildId=test
```

Kết quả mong đợi:
```json
{"error": "Không tìm thấy build"}
```

---

### Bước 4.2 — Test Build APK

1. Mở trang web frontend
2. Nhập URL trang PWA (VD: `https://libvn.pages.dev`)
3. Click **"Build APK"**
4. Theo dõi tiến trình (khoảng 5–10 phút)
5. Download APK khi hoàn tất

---

### Bước 4.3 — Kiểm Tra GitHub Actions

Vào repo → **Actions** → Xem lần chạy mới nhất

Các bước sẽ lần lượt hiển thị ✅:
- Checkout mã nguồn
- Đọc dữ liệu Payload
- Cấu hình Java & Node.js
- Đồng ý SDK Licenses
- Sinh Keystore
- Biên dịch TWA Headless
- Phát hành lên GitHub Release
- Gửi webhook về Worker

---

## 🔒 PHẦN 5: Lưu Ý Bảo Mật

| Mục | Hành Động |
|---|---|
| GitHub Token | Lưu trong Cloudflare Secrets, KHÔNG hardcode trong code |
| Keystore Password | Lưu trong GitHub Secrets |
| Repo GitHub | Để **Public** an toàn vì token không nằm trong code |
| APK | Tồn tại tạm thời trên GitHub Release, xóa sau khi tải |

---

## 🐛 PHẦN 6: Xử Lý Lỗi Thường Gặp

### Lỗi: `androidSdk isn't correct`
**Nguyên nhân:** Đường dẫn SDK sai  
**Cách sửa:** Dùng biến `${ANDROID_SDK_ROOT}` thay vì hardcode

---

### Lỗi: `Invalid URL`
**Nguyên nhân:** URL manifest có dấu `/` thừa hoặc thiếu `https://`  
**Cách sửa:** Đảm bảo URL đầy đủ dạng `https://domain.com`

---

### Lỗi: `Process completed with exit code 130`
**Nguyên nhân:** Bubblewrap CLI hỏi interactive, bị timeout  
**Cách sửa:** Dùng `@bubblewrap/core` thay vì CLI

---

### Lỗi: `cli ERROR The provided androidSdk isn't correct`
**Nguyên nhân:** Config file `~/.bubblewrap/config.json` sai  
**Cách sửa:**
```bash
echo "{\"jdkPath\": \"${JAVA_HOME}\", \"androidSdkPath\": \"${ANDROID_SDK_ROOT}\"}" > ~/.bubblewrap/config.json
```

---

### Lỗi: Webhook không nhận được
**Nguyên nhân:** Secret `WORKER_WEBHOOK_URL` chưa điền hoặc sai  
**Cách sửa:** Kiểm tra lại URL trong GitHub Secrets

---

## 📁 Danh Sách File Dự Án

```
dự-án/
├── .github/
│   └── workflows/
│       └── build-apk.yml        ← Workflow GitHub Actions
├── cloudflare-worker.js          ← Code Cloudflare Worker
└── index.html                    ← Giao diện web
```

---

## 💡 Nâng Cấp Tương Lai

- Thêm **email notification** khi build xong
- Hỗ trợ **custom package ID** cho từng user
- Thêm **lịch sử build** lưu lâu dài bằng database
- Hỗ trợ **tạo AAB** (Android App Bundle) cho Google Play
- Thêm **Digital Asset Links** tự động cho TWA verification

---

*Tài liệu này được tạo dựa trên quá trình triển khai thực tế. Cập nhật lần cuối: 06/2026*
