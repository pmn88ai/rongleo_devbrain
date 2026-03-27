# ⬡ DevBrain

> **Bộ nhớ ngoài cho dev** — Quản lý tiến độ nhiều web/app cá nhân song song, không bị rối, không bị quên.

![DevBrain Dashboard](https://img.shields.io/badge/status-v4%20stable-4caf7a?style=flat-square) ![Stack](https://img.shields.io/badge/stack-HTML%20%2B%20Supabase-e85a20?style=flat-square) ![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

---

## Tại sao DevBrain?

Khi mày chạy 5–10 project cá nhân song song, vấn đề không phải là thiếu tool — mà là **không nhớ được mình đang làm đến đâu**. Mở lại project sau 2 tuần nghỉ, không biết task cuối đang dở là gì, bug nào chưa fix, quyết định kỹ thuật nào đã chốt.

DevBrain không phải project manager. Không phải Jira. Không phải Notion. Nó là **bộ nhớ ngoài của não dev** — ghi nhanh, xem lại nhanh, không overhead.

---

## Tính năng

### Dashboard & Navigation

| Tính năng | Mô tả |
|---|---|
| **3 view modes** | Grid / List / Kanban — chuyển đổi tức thì |
| **Filter theo status** | idea / spec / building / testing / done / improving / dropped |
| **Filter theo tag** | click tag trong sidebar để lọc nhanh |
| **Full-text search** | tìm trong tên, mô tả, use case, keywords, và nội dung notes — dùng pre-built index O(n) |
| **Stats bar** | tổng projects / active / done / zombie đang chờ |

### Project Management

| Tính năng | Mô tả |
|---|---|
| **7 status mặc định** | idea → spec → building → testing → done → improving → dropped |
| **Progress tự động** | tính từ checklist 5 mục, không nhập tay — luôn nhất quán |
| **Tag hệ thống** | normalize lowercase, dedupe tự động, suggest khi nhập |
| **Keywords field** | từ khóa phụ cho search sâu hơn không cần AI |
| **Use case field** | ghi rõ app này giải quyết vấn đề gì |
| **updatedAt chính xác** | update khi: thêm note / tick checklist / đổi status / edit project |

### Return Mode — Tính năng cốt lõi

Khi mở lại project sau thời gian nghỉ, **Return Mode** hiện ngay:

- 📝 **Lần cuối ghi** — note gần nhất là gì, ghi khi nào
- ☐ **Việc chưa xong** — còn bao nhiêu mục trong checklist
- 🚀 **Backlog** — improvement đang ở status todo
- 🐛 **Bug chưa fix** — số bug đang open

Kèm 3 nút hành động nhanh: **Continue** / **Snooze 7 ngày** / **Drop**

### Zombie Detection

Project không được cập nhật hơn **14 ngày** tự động bị highlight màu vàng với badge 🧟. Zombie detection bỏ qua project đã done/dropped và project đang trong thời gian snooze.

**Snooze logic:** snooze KHÔNG thay đổi `updated_at` — phân biệt "nghỉ có ý thức" vs "bỏ bê thật sự".

### Dev Log & Notes

| Tính năng | Mô tả |
|---|---|
| **4 loại note** | 💡 Idea / 🐛 Bug / 📝 Log / ✅ Decision |
| **Note edit được** | click ✎ để sửa nội dung và type — lưu `updatedAt` riêng |
| **Bug lifecycle** | open → ✓ Fix → resolved (gạch ngang) → ↩ Reopen |
| **Note TODO** | đánh dấu note là TODO, tick checkbox khi xong |
| **Decision highlight** | note type decision tự highlight nền xanh — nổi bật khỏi log thường |
| **Filter note** | All / ☐ Todo (có badge đếm) / 💡 Idea / 🐛 Bug / 📝 Log / ✅ Decision |
| **Pagination** | chỉ show 20 note/trang, nút "Xem thêm" — không render cả trăm note cùng lúc |

### Backlog / Improvements

- Sort tự động theo priority: 🔴 High → 🟡 Medium → ⚪ Low
- Border-left màu theo priority — nhìn là biết ngay
- Dropdown đổi status trực tiếp: todo / doing / done

### Links Hub

Lưu tất cả link liên quan tập trung: repo, demo, figma, doc, staging, production — không cần nhớ bookmark.

### Data & Sync

| Tính năng | Mô tả |
|---|---|
| **Supabase backend** | lưu cloud, không mất data khi clear browser |
| **Multi-device sync** | mở trên laptop và phone cùng lúc — Realtime subscription tự đồng bộ |
| **Optimistic UI** | mọi action update local state ngay, gọi DB sau — không có lag |
| **Rollback on error** | nếu DB fail, tự rollback về state trước |
| **Export JSON** | backup toàn bộ data ra file `.json` bất cứ lúc nào |
| **Import JSON + sync** | import file backup → xoá data cũ trên Supabase → insert toàn bộ lên DB → reload |
| **Undo stack** | undo delete note/improvement/link (10 levels, 4.5 giây) bằng ⌘Z |
| **Confirm before delete** | confirm modal trước mọi thao tác xoá — không click nhầm mất data |

### UX & Accessibility

- **Dark / Light mode** — toggle tức thì, nhớ preference
- **Mobile responsive** — bottom nav, FAB, sidebar overlay
- **Keyboard shortcuts** — `Escape` đóng modal, `⌘K` focus search, `⌘Z` undo
- **FAB context-aware** — đang trong project → mở note modal / đang ở dashboard → mở project modal
- **pre-line whitespace** — use case và description giữ nguyên xuống dòng khi nhập

---

## Tech Stack

```
Frontend    HTML + CSS + Vanilla JS    Single file, no build step
Database    Supabase (PostgreSQL)      Cloud, realtime, free tier
SDK         supabase-js v2 (UMD)       Loaded từ cdn.jsdelivr.net
```

**Không dùng:** React, Vue, Node, webpack, npm, hay bất kỳ framework nào. Mở file HTML là chạy.

---

## Database Schema

```sql
projects (
  id            uuid PK,
  name          text,
  description   text,
  use_case      text,
  status        text CHECK (idea|spec|building|testing|done|improving|dropped),
  tags          text[],
  keywords      text,
  checklist     boolean[5],
  snoozed_until timestamptz,
  created_at    timestamptz,
  updated_at    timestamptz
)

notes (
  id          uuid PK,
  project_id  uuid FK → projects,
  content     text,
  type        text CHECK (idea|bug|log|decision),
  resolved    boolean,
  is_todo     boolean,
  is_done     boolean,
  created_at  timestamptz,
  updated_at  timestamptz
)

improvements (
  id         uuid PK,
  project_id uuid FK → projects,
  content    text,
  priority   text CHECK (high|medium|low),
  status     text CHECK (todo|doing|done)
)

links (
  id         uuid PK,
  project_id uuid FK → projects,
  label      text,
  url        text
)
```

---

## Setup

### 1. Supabase

1. Tạo project tại [supabase.com](https://supabase.com)
2. Vào **SQL Editor** → paste nội dung `schema.sql` → Run
3. Copy **Project URL** và **anon/public key** từ Settings → API

### 2. Config trong file HTML

M�� `devbrain-v4.html`, tìm đoạn này và thay bằng credentials của mày:

```js
const SB_URL = 'https://your-project.supabase.co'
const SB_KEY = 'your-anon-public-key'
```

### 3. Mở file

Double-click `devbrain-v4.html` trong browser — xong.

Không cần localhost, không cần server, không cần `npm install`.

### Bật Realtime (tùy chọn)

Supabase Dashboard → **Database → Replication** → bật 4 bảng: `projects`, `notes`, `improvements`, `links`.

---

## Files

```
devbrain-v4.html    App chính — toàn bộ UI + logic + Supabase layer (~79KB)
schema.sql          SQL để tạo database schema trên Supabase
devbrain-test.html  Connection test page — chạy trước để verify Supabase config
README.md           File này
```

---

## Roadmap

- [ ] Auth (Supabase Auth) — mỗi user có data riêng
- [ ] Drag & drop Kanban
- [ ] AI features — suggest next step từ notes, semantic search
- [ ] Export portfolio page — public page showcase các project đã done
- [ ] Custom status CRUD UI
- [ ] Notification khi project sắp zombie

---

## License

MIT — dùng thoải mái, fork thoải mái.
