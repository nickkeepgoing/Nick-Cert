# ◈ CERT_VAULT

> Personal certificate collection with simple one file.
---

## 📋 Overview

CERT_VAULT เป็น web app สำหรับเก็บและจัดการ certificate ส่วนตัว (PDF / รูปภาพ) ทำงานเป็น single HTML file ที่ใช้ Supabase เป็น backend

**Features:**
- อัปโหลด certificate (PDF, JPG, PNG, WEBP, GIF) ขนาดสูงสุด 50MB
- Preview PDF thumbnail และรูปภาพได้ทันที
- ค้นหา / กรองตาม category / เรียงลำดับ
- Login ด้วย Supabase Auth (email + password)
- ไม่มี API key หรือรหัสผ่านใน source code

---

## 🚀 Getting Started

### 1. ตั้งค่า Supabase

1. สร้าง project ที่ [supabase.com](https://supabase.com)
2. ไปที่ **Table Editor** → สร้าง table ชื่อ `certs` ด้วย schema ด้านล่าง
3. ไปที่ **Storage** → สร้าง bucket ชื่อ `certs` (ตั้งเป็น Private)
4. ไปที่ **Authentication** → สร้าง user ด้วย email + password

### 2. Schema สำหรับ table `certs`

```sql
create table certs (
  id           uuid primary key default gen_random_uuid(),
  title        text not null,
  issuer       text,
  issued_date  date,
  category     text,
  file_path    text not null,
  file_name    text not null,
  file_type    text,
  created_at   timestamptz default now()
);
```

### 3. เปิด Row Level Security (RLS)

```sql
-- เปิด RLS
alter table certs enable row level security;

-- อนุญาตเฉพาะ user ที่ login แล้ว
create policy "auth only" on certs
  for all using (auth.role() = 'authenticated');

-- Storage bucket policy
-- ไปที่ Storage → certs bucket → Policies → เพิ่ม policy สำหรับ authenticated users
```

### 4. เปิดใช้งาน

เปิดไฟล์ `test_secure.html` ในเบราว์เซอร์ กรอก:
- **Supabase URL** — หาได้ที่ Project Settings → API
- **Anon Key** — หาได้ที่ Project Settings → API → `anon` `public`
- **Email + Password** — ที่สร้างไว้ใน Authentication

---

## 🔐 Security Model

| จุด | วิธีป้องกัน |
|-----|------------|
| API Key ใน source | ❌ ไม่มี — user กรอกเองตอน login |
| รหัสผ่านใน source | ❌ ไม่มี — ใช้ Supabase Auth |
| Brute force | Rate limit 5 ครั้ง → ล็อค 5 นาที |
| ไฟล์อันตราย | Whitelist เฉพาะ PDF / JPG / PNG / WEBP / GIF |
| XSS | `escapeHtml()` ทุก user input ก่อน render |
| Database access | Row Level Security (RLS) บน Supabase |

> **หมายเหตุ:** Anon Key ของ Supabase ถูกออกแบบให้ expose ใน client-side ได้ ความปลอดภัยจริงอยู่ที่ RLS บน database — **ต้องเปิด RLS ทุกครั้ง**

---

## 📁 File Structure

```
test_secure.html   ← single-file app (HTML + CSS + JS)
README.md          ← ไฟล์นี้
```

---

## 🗂️ Categories

| ค่า | แสดงผล |
|-----|--------|
| `IT` | IT / Tech |
| `Security` | Security |
| `ภาษา` | Language |
| `การอบรม` | Training |
| `การศึกษา` | Education |
| `รางวัล` | Award |
| `อื่นๆ` | Other |

แก้ไข category ได้ใน `<select id="certCategory">` ใน HTML

---

## 🛠️ Tech Stack

- **Frontend:** Vanilla JS, HTML, CSS (single file)
- **Backend:** [Supabase](https://supabase.com) (Auth + Database + Storage)
- **PDF Preview:** [PDF.js](https://mozilla.github.io/pdf.js/) v3.11
- **Font:** Kanit, JetBrains Mono (Google Fonts)

---

## ⚙️ Configuration

ค่าที่ปรับได้ใน source code:

```js
var LOGIN_MAX_ATTEMPTS = 5;        // จำนวนครั้งที่ผิดได้ก่อนล็อค
var LOGIN_LOCKOUT_MS   = 5 * 60 * 1000;  // ระยะเวลาล็อค (ms)
var BUCKET = 'certs';             // ชื่อ Storage bucket
var ALLOWED_TYPES = [...];        // ประเภทไฟล์ที่อนุญาต (MIME)
var ALLOWED_EXTS  = [...];        // นามสกุลไฟล์ที่อนุญาต
```

