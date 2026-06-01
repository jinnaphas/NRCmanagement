# NRC Management System — สถาปัตยกรรม & กระบวนการบริหาร (v3)

> บันทึกการปรับเปลี่ยนกระบวนการบริหาร NRC ด้วย Web Application นี้ (มิ.ย. 2569)

## รูปแบบระบบ
- **Single-file Web App** (`NRC Management System.html`) host บน **GitHub Pages**
  - repo: `jinnaphas/NRCmanagement` · branch `main` · เปิดผ่าน `index.html` (redirect)
- **ไม่ต้องล็อกอิน ไม่พึ่ง server** — กรรมการเปิด URL ดูได้ทันที
- เลิกใช้ทั้ง GitHub Gist และ Google OAuth login (เคยลองแล้วยุ่งยากสำหรับผู้ใช้)

## แหล่งข้อมูล (Data flow)
- ข้อมูลหลักอยู่ในไฟล์ **`data.json`** ใน repo (read-only, เสิร์ฟผ่าน Pages) — ทุกคนเห็นเหมือนกัน
- แอปโหลด `data.json` ตอนเปิด; ระหว่างแก้ไขเก็บใน memory (มีเตือน beforeunload ถ้ายังไม่เผยแพร่)
- **ไม่ใช้ localStorage เป็นแหล่งข้อมูลหลัก**

## การบันทึก/เผยแพร่ (สำคัญต่อกระบวนการ)
1. แก้ข้อมูลในแอป → มุมขวาบนขึ้น ✏️ "แก้ไขแล้ว"
2. กดปุ่ม **💾 Save & Publish** (topbar / หน้าตั้งค่า) → commit `data.json` ขึ้น GitHub โดยตรง (Contents API)
3. GitHub Pages อัปเดตอัตโนมัติ ~1 นาที → กรรมการเห็นข้อมูลใหม่
- ใช้ **GitHub fine-grained token** (สิทธิ์ Contents: Read & write เฉพาะ repo นี้) เก็บใน browser ของ **คนแก้ (Earth) คนเดียว** — กรรมการที่เปิดดูไม่ต้องมี token
- วิธีสำรอง: Export `data.json` แล้ว commit เอง

## รูป & ไฟล์เอกสาร
- เก็บเป็น **Google Drive share link** (ตั้งสิทธิ์ "Anyone with the link") — วางลิงก์ในแอป ไม่อัปโหลดผ่านระบบ
- รองรับทั้งรูปกรรมการ, เอกสารคลัง, ไฟล์ Agenda/MoM, ไฟล์แนบวาระ

## โมเดลข้อมูล (แนวคิดหลัก)
- **ทะเบียนคณะกรรมการทั้งหมด = Master pool** (รายชื่อบุคคลทุกคน) — อยู่ในทะเบียน ≠ อยู่ในบอร์ด PCC อัตโนมัติ
- จาก Master นำไป **จัดเข้า** โครงสร้างต่าง ๆ (อ้างอิงจาก Master):
  - **คณะกรรมการบริษัท (PCC Board):** ฟิลด์ `boardRole` = chair/vice/member/none
  - **คณะกรรมการชุดย่อย (NRC/AC/RMC/GCG/ExCo):** ฟิลด์ role flags (`nrcRole`, `acRole`, …) → Control List แสดงแบบ derive อัตโนมัติ
  - **บริษัทย่อย:** เลือกชื่อจาก Master ได้ (หรือพิมพ์เอง)
- **CG analysis + Board Skills Matrix คิดจากสมาชิกบอร์ด PCC เท่านั้น** (boardRole ≠ none)
- **Board Skills Matrix แก้ไข/เพิ่ม-ลบทักษะได้** (เก็บใน `DB.skillsList`)
- **อำนาจลงนามตามกฎหมาย:** ติ๊กเลือกผู้มีอำนาจ + ข้อความเงื่อนไข — แยกของ PCC และของแต่ละบริษัทย่อย

## เมนูในระบบ
Dashboard · การประชุม · ทะเบียนคณะกรรมการทั้งหมด · Control List (คณะกรรมการบริษัท/ชุดย่อย/บริษัทย่อย) · Succession Plan · ประวัติกรรมการ · ค่าตอบแทน · พัฒนากรรมการ · ประเมินผล · คลังเอกสาร · AGM Tracker · ผลประโยชน์ฯ · ตั้งค่า

## หมายเหตุการดูแล
- คู่มือผู้ใช้/ติดตั้ง: `SETUP-GUIDE.md` (เปิด Pages, สร้าง token, ใส่ลิงก์ Drive, เผยแพร่)
- วันที่เก็บเป็น พ.ศ. แสดงแบบไทย; ภาษาหลักไทย
