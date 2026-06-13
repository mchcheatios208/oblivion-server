# 🚀 Oblivion Key Server — Hướng dẫn deploy Vercel

## Cấu trúc project

```
oblivion-server/
├── api/
│   ├── _auth.js          ← helper kiểm tra admin password
│   ├── verify.js         ← POST /api/verify  (user gọi để check key)
│   ├── setup.js          ← POST /api/setup   (seed key mặc định)
│   └── admin/
│       ├── keys.js       ← GET  /api/admin/keys  (danh sách keys)
│       ├── key.js        ← POST/DELETE /api/admin/key
│       └── logs.js       ← GET  /api/admin/logs
├── public/
│   ├── index.html        ← App chính (người dùng)
│   └── admin.html        ← Admin panel (chỉ bạn dùng)
├── package.json
└── vercel.json
```

---

## BƯỚC 1 — Tạo tài khoản & cài Vercel CLI

```bash
# Cài Node.js nếu chưa có: https://nodejs.org
npm install -g vercel
vercel login
```

---

## BƯỚC 2 — Tạo Vercel KV (database lưu keys)

1. Vào **https://vercel.com** → Dashboard → **Storage** → **Create Database**
2. Chọn **KV (Redis)** → đặt tên ví dụ `oblivion-kv` → Create
3. Sau khi tạo xong, vào tab **`.env.local`** → copy 3 biến:
   ```
   KV_URL=...
   KV_REST_API_URL=...
   KV_REST_API_TOKEN=...
   ```
   *(Vercel tự inject vào project khi bạn connect)*

---

## BƯỚC 3 — Deploy lên Vercel

```bash
cd oblivion-server
vercel
```

Trả lời các câu hỏi:
- **Set up and deploy?** → `Y`
- **Which scope?** → chọn account của bạn
- **Link to existing project?** → `N`
- **Project name?** → `oblivion` (hoặc tên bạn muốn)
- **In which directory is your code?** → `.` (enter)
- **Override settings?** → `N`

Sau đó sẽ có URL kiểu: `https://oblivion-xxxx.vercel.app`

---

## BƯỚC 4 — Gắn KV vào project

1. Vào **https://vercel.com** → project `oblivion` → **Storage** tab
2. Click **Connect Database** → chọn KV `oblivion-kv` vừa tạo
3. Click **Connect**

---

## BƯỚC 5 — Thêm biến môi trường ADMIN_PASSWORD

1. Vào project → **Settings** → **Environment Variables**
2. Thêm:
   ```
   Name:  ADMIN_PASSWORD
   Value: (mật khẩu bí mật của bạn, ví dụ: MySecret@2024)
   ```
3. Click **Save**

---

## BƯỚC 6 — Redeploy để áp dụng env vars

```bash
vercel --prod
```

Hoặc vào Vercel dashboard → **Deployments** → **Redeploy**

---

## BƯỚC 7 — Seed key mặc định vào database

Gọi API một lần để thêm key `Mc7X-2k9Z-xW3m1` vào KV:

```bash
curl -X POST https://oblivion-xxxx.vercel.app/api/setup \
  -H "x-admin-key: MẬT_KHẨU_ADMIN_CỦA_BẠN"
```

Hoặc dùng Postman / Thunder Client.

---

## BƯỚC 8 — Truy cập

| URL | Dành cho |
|-----|----------|
| `https://oblivion-xxxx.vercel.app` | Người dùng — nhập key để vào |
| `https://oblivion-xxxx.vercel.app/admin` | **Chỉ bạn** — quản lý keys |

---

## Cách dùng Admin Panel

1. Vào `/admin` → nhập **ADMIN_PASSWORD** đã đặt ở bước 5
2. **Tạo key:** 3 chế độ:
   - 🎲 **Tự động** — random key `Xxxx-xxxx-xxxx`
   - ✏️ **Tùy chỉnh** — nhập key bất kỳ
   - 🔣 **Pattern** — ví dụ `VIP-0000-xxxx`
   - 📦 **Số lượng** — tạo 10/20/50 keys cùng lúc
3. **Xem login log** — ai dùng key nào, lúc mấy giờ, IP gì
4. **Xóa key** — click nút đỏ trên từng key
5. **Copy tất cả** — copy toàn bộ danh sách để gửi cho người dùng

---

## API Reference

### `POST /api/verify`
Người dùng gọi để xác thực key.
```json
// Request
{ "key": "Mc7X-2k9Z-xW3m1" }

// Response OK
{ "valid": true }

// Response fail
{ "valid": false, "reason": "invalid_key" }
```

### `GET /api/admin/keys`
Header: `x-admin-key: YOUR_PASSWORD`
```json
{ "keys": [{ "key": "Mc7X-2k9Z-xW3m1", "uses": 5 }], "total": 1 }
```

### `POST /api/admin/key`
Tạo key mới.
```json
// Tự động
{}

// Tùy chỉnh
{ "custom": "mykey-2024" }

// Pattern
{ "pattern": "VIP-0000-xxxx" }

// Hàng loạt
{ "count": 20 }
```

### `DELETE /api/admin/key`
Xóa key.
```json
{ "key": "key-cần-xóa" }
```

### `GET /api/admin/logs`
100 lần đăng nhập gần nhất.

---

## Lưu ý bảo mật

- **ADMIN_PASSWORD** phải mạnh (chữ hoa, số, ký tự đặc biệt)
- Không chia sẻ URL `/admin` với người dùng
- KV free tier: 30,000 requests/tháng — đủ dùng cho vài nghìn users
- Vercel free tier: 100GB bandwidth/tháng

---

## Troubleshooting

**Lỗi `KV_REST_API_URL is not defined`**
→ Chưa connect KV vào project. Làm lại bước 4.

**Lỗi 401 khi vào admin**
→ Sai ADMIN_PASSWORD hoặc chưa set env var. Kiểm tra bước 5.

**Key nhập đúng nhưng báo sai**
→ Chưa seed key vào DB. Làm lại bước 7.

**Deploy thành công nhưng trang trắng**
→ Kiểm tra Vercel logs: Dashboard → project → Deployments → click deployment → Functions tab
