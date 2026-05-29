# คู่มือติดตั้ง NRC Management System (v3.0 — Google Drive)

> ระบบเก็บข้อมูลและไฟล์แนบไว้บน **Google Drive** ทีมเข้าใช้ผ่าน URL เดียว
> ล็อกอินด้วยบัญชี Google — ไม่ต้องวาง token ใดๆ อีกต่อไป

## ภาพรวมการทำงาน

| สิ่งที่เก็บ | เก็บที่ไหน | ใครเห็น |
|------------|-----------|---------|
| ข้อมูลหลัก (วาระ, ค่าตอบแทน, Succession, ประเมินผล) | ไฟล์ `nrc-data.json` บน Google Drive | เฉพาะคนที่ถูกแชร์โฟลเดอร์ + ล็อกอิน |
| ไฟล์แนบ & รูปกรรมการ | ไฟล์บน Google Drive (ตั้ง "Anyone with link") | กรรมการคลิกลิงก์เปิดได้ทันที |

ระบบใช้สิทธิ์ Google แบบ `drive.file` เท่านั้น → แอป **เห็นเฉพาะไฟล์ที่ตัวเองสร้าง/เลือก** เข้าถึงไฟล์อื่นใน Drive ของคุณไม่ได้

---

## ขั้นตอนทั้งหมด (ทำครั้งเดียวโดยผู้ดูแลระบบ — Earth)

### Part A — เปิด GitHub Pages เพื่อให้ได้ URL ของแอป

1. ไปที่ repo บน GitHub → **Settings** → **Pages**
2. หัวข้อ **Build and deployment** → **Source** เลือก **GitHub Actions**
3. รอ workflow "Deploy to GitHub Pages" รันเสร็จ (ดูที่แท็บ **Actions**)
4. จะได้ URL ประมาณ: **`https://jinnaphas.github.io/nrcmanagement/`**
   - URL นี้คือที่ทีมจะเข้าใช้งาน — จดไว้

> หมายเหตุ: ถ้า deploy จากสาขาอื่นที่ไม่ใช่ `main` ไม่ได้ ให้ merge โค้ดเข้า `main` ก่อน
> หรือเปลี่ยนเป็น Source = "Deploy from a branch" → เลือกสาขา + โฟลเดอร์ `/ (root)`

### Part B — สร้าง Google OAuth Client ID

1. เข้า **https://console.cloud.google.com/** (ล็อกอินด้วย Google ของคุณ)
2. สร้างโปรเจกต์ใหม่ (เช่นชื่อ `NRC-PCC`) ที่มุมบนซ้าย
3. เปิด API ที่ต้องใช้ — ไปที่ **APIs & Services → Library** แล้ว Enable ทั้ง 2 ตัว:
   - **Google Drive API**
   - **Google Picker API**
4. ตั้งหน้าจอยินยอม — **APIs & Services → OAuth consent screen**
   - User Type: **External** → Create
   - กรอกชื่อแอป `NRC Management System`, อีเมลสนับสนุน, อีเมลนักพัฒนา → Save
   - หน้า **Scopes**: ข้ามได้ (ระบบขอ scope `drive.file` ตอนล็อกอินเอง)
   - หน้า **Test users**: กด **+ ADD USERS** ใส่อีเมล Google ของ **ทุกคนในทีม** ที่จะใช้งาน → Save
     (ขณะอยู่สถานะ Testing เฉพาะอีเมลในรายการนี้เท่านั้นที่ล็อกอินได้)
5. สร้าง Credential — **APIs & Services → Credentials → + CREATE CREDENTIALS → OAuth client ID**
   - Application type: **Web application**
   - ชื่อ: `NRC Web`
   - **Authorized JavaScript origins** → ADD URI ใส่ origin ของ Pages (ไม่มี path ต่อท้าย):
     - `https://jinnaphas.github.io`
   - (ไม่ต้องใส่ Authorized redirect URIs)
   - กด **CREATE** → จะได้ **Client ID** หน้าตา `xxxxxxxx.apps.googleusercontent.com` → คัดลอกไว้

### Part C — ตั้งค่าครั้งแรกในแอป (สร้างข้อมูลกลาง)

1. เปิด URL ของแอป (จาก Part A) → ครั้งแรกจะเข้าโหมด Local
2. ไปเมนู **ตั้งค่า → Google Drive Sync**
3. วาง **Client ID** ที่ได้จาก Part B → กด **บันทึก Client ID**
4. กด **Sign in with Google** → เลือกบัญชี → ถ้าเจอจอ "Google hasn't verified this app"
   ให้กด **Advanced → Go to NRC Management System (unsafe)** (ปลอดภัย เพราะเป็นแอปของเราเอง)
5. กด **สร้างข้อมูลใหม่ในโฟลเดอร์** → เลือก/สร้างโฟลเดอร์ใน Drive (เช่น `NRC`)
   - ระบบจะสร้างไฟล์ `nrc-data.json` ในโฟลเดอร์นั้น พร้อม sync ข้อมูลปัจจุบันขึ้นไป
6. เสร็จแล้ว — แถบบนขวาจะขึ้น ✅ Sync

### Part D — เพิ่มสมาชิกทีม

1. เปิด **Google Drive** → หาโฟลเดอร์ที่ใช้เก็บข้อมูล (เช่น `NRC`)
2. คลิกขวา → **Share / แชร์** → ใส่อีเมล Google ของสมาชิก → ตั้งสิทธิ์ **Editor** → ส่ง
3. แจ้งสมาชิกให้:
   - เปิด URL ของแอป
   - ไปเมนู **ตั้งค่า → Google Drive** → วาง **Client ID เดียวกัน** → บันทึก
   - กด **Sign in with Google** (อีเมลต้องอยู่ใน Test users จาก Part B ข้อ 4)
   - กด **เลือกไฟล์ข้อมูลที่แชร์มา** → เลือกไฟล์ `nrc-data.json` ที่ถูกแชร์ → เสร็จ

> เมื่อ Sign in สำเร็จและเลือกไฟล์แล้ว ครั้งต่อๆ ไปแค่เปิด URL แล้วกด Sign in (หรือเข้าอัตโนมัติถ้ายังล็อกอิน Google อยู่)

---

## ไฟล์แนบ & รูปกรรมการ

- เวลาแนบไฟล์/อัปโหลดรูป ระบบอัปโหลดขึ้นโฟลเดอร์ Drive และตั้งสิทธิ์ **Anyone with link (อ่านได้)** อัตโนมัติ
- กรรมการ/ทีมคลิกชื่อไฟล์เพื่อเปิดได้ทันที (ไม่ต้องล็อกอินก็เปิดไฟล์แนบได้)
- จำกัด: ไฟล์แนบ ≤ 5MB, รูปกรรมการ ≤ 2MB ต่อไฟล์
- ต้อง **Sign in ก่อน** จึงจะอัปโหลด/แนบไฟล์ได้

---

## ความปลอดภัย

- ข้อมูลหลัก (`nrc-data.json`) **ไม่ได้** ตั้ง public — แชร์เฉพาะอีเมลที่คุณเพิ่มในโฟลเดอร์ Drive
- ระบบขอสิทธิ์ `drive.file` เท่านั้น (เห็นเฉพาะไฟล์ที่แอปสร้าง/ผู้ใช้เลือก)
- ถอนสิทธิ์สมาชิก: ลบอีเมลออกจากการแชร์โฟลเดอร์ใน Google Drive
- Client ID ไม่ใช่ความลับ (ฝังในหน้าเว็บได้ตามปกติของเว็บ OAuth) — ความปลอดภัยอยู่ที่ Test users + การแชร์โฟลเดอร์

---

## ปัญหาที่พบบ่อย

| อาการ | วิธีแก้ |
|-------|--------|
| จอ "Google hasn't verified this app" | กด Advanced → Go to ... (ปกติสำหรับแอปภายในที่ยังไม่ verify) |
| ล็อกอินไม่ได้ / error 403 | อีเมลยังไม่ถูกเพิ่มใน **Test users** (Part B ข้อ 4) |
| กด Sign in แล้วไม่มีอะไรเกิด | ตรวจ **Authorized JavaScript origins** ต้องตรงกับ URL ที่เปิด (origin เป๊ะ ๆ ไม่มี / ต่อท้าย) |
| สมาชิกมองไม่เห็นไฟล์ตอนกด "เลือกไฟล์ข้อมูลที่แชร์มา" | ยังไม่ได้แชร์โฟลเดอร์/ไฟล์ให้อีเมลนั้น (Part D) — หรือดูที่แท็บ "Shared with me" ใน Picker |
| Sync ค้าง/หมดอายุ | access token หมดอายุใน ~1 ชม. ระบบจะขอใหม่อัตโนมัติ ถ้าไม่หาย ให้กด Sign in อีกครั้ง |

---

## หากต้องการเปิดให้คนทั่วไป (ไม่ต้อง Test users)

ถ้าทีมมีจำนวนมากและไม่อยากเจอจอ "unverified": ไปที่ OAuth consent screen → **PUBLISH APP**
เนื่องจากใช้ scope `drive.file` (non-sensitive) จึง**ไม่ต้องผ่าน security review** ของ Google
