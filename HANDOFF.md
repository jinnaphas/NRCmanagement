# HANDOFF — NRC Management System (องค์ความรู้ + วิธีทำงานครบชุด)

> เอกสารส่งต่อสำหรับ Claude/ผู้ดูแลคนถัดไป — อ่านไฟล์นี้ก่อนเริ่มงานทุกครั้ง
> เจ้าของงาน: Earth (จิณนภัสฐ์ ภัสฐาพงษ์) เลขาคณะกรรมการ NRC, บมจ. พรีไซซ คอร์ปอเรชั่น (PCC)
> ภาษาหลัก: ไทย · วันที่เก็บเป็น พ.ศ. (เช่น 2569-04-29) แสดงแบบไทย

---

## 1. ภาพรวมระบบ
- **NRC Management System** = Web App ไฟล์เดียว `NRC Management System.html` (มีช่องว่างในชื่อ) host บน **GitHub Pages**
- repo: `jinnaphas/NRCmanagement` (Public) · default branch `main`
- **URL ใช้งานจริง (case-sensitive!):** https://jinnaphas.github.io/NRCmanagement/
  - `index.html` = redirect ไปยัง `NRC%20Management%20System.html`
- ไม่มี backend / ไม่ล็อกอินสำหรับผู้ชม / ไม่ใช้ localStorage เป็นแหล่งข้อมูลหลัก
- เคยลองแล้ว **เลิกใช้**: GitHub Gist sync, Google OAuth/Drive login (ผู้ใช้บอกว่ายุ่งยาก)

## 2. แหล่งข้อมูล (data flow)
- ข้อมูลหลักทั้งหมดอยู่ในไฟล์ **`data.json`** ที่ repo root (read-only, เสิร์ฟผ่าน Pages)
- แอป `fetch('data.json')` ตอนเปิด; ระหว่างแก้ไขเก็บใน memory (`DB`) มี `beforeunload` เตือนถ้ายังไม่เผยแพร่
- **เผยแพร่:** ปุ่ม **💾 Save & Publish** (บน topbar/หน้าตั้งค่า) → commit `data.json` ขึ้น GitHub ผ่าน Contents API → Pages redeploy ~1 นาที
  - ใช้ **GitHub fine-grained token** (สิทธิ์ **Contents: Read and write** เฉพาะ repo นี้) เก็บใน localStorage ของ "คนแก้" เท่านั้น
  - โค้ด: `publishToGitHub()` (GET sha → PUT, `cache:no-store` + retry เมื่อ 409)
- วิธีสำรอง: ปุ่ม **Export data.json** แล้ว commit เอง

## 3. รูปภาพ & ไฟล์เอกสาร
- ใช้ **Google Drive share link** (ตั้งสิทธิ์ "Anyone with the link") วางลิงก์ในแอป — ไม่อัปโหลดผ่านระบบ
- รูปกรรมการ: ช่อง "รูป (Drive link)" · `driveImageUrl()` แปลง share link → thumbnail URL
- ไฟล์แนบ (ประชุม/วาระ/เอกสาร): เก็บเป็น `{name, link}` · `attAnchor()` แสดงลิงก์ · `promptLinkAttachment()` รับลิงก์

## 4. โครงสร้างข้อมูล (data.json schema)
```
company: { name, abbr, secretary, fiscalYear, signatoryIds[], signCondition }
directors[]: {
  id, name, nameEn, position, type(independent|nonexecutive|executive), gender, startDate,
  boardRole(chair|vice|member|none),            // การเป็นกรรมการบริษัท PCC
  nrcRole|acRole|rmcRole|gcgRole|excoRole (chair|member|none),  // คณะย่อย
  skills{<skillKey>:0|1}, note, photo(Drive link),
  iodCerts{ DAP|DCP|RCP|BNCP|AACP : "เลขรุ่น/ปี" },
  // วาระ (Board Term):
  firstAppoint, convAppoint, nineYearDate, replacedName,
  termStatus(active|resigned), exitDate, rotation{ <ปีพ.ศ.>: reelect|retire|in|exit }
}
meetings[]: { id, year, no, date, type, status, agendaAttachments[], momAttachments[],
              agendaItems[{id,title,type,status,note,attachments[]}], actionItems[], note }
  attachments: { name, link }              // link = Drive share link
actionItems[], succession[], directorChangelog[], remuneration[], selfAssessments[],
documents[]: { id, title, docType, version, effectiveDate, approvedBy, file{name,link}, note }
agmItems[], independenceChecks[], conflictLog[]
controlList: { subcommittees[](legacy ไม่ใช้แล้ว), subsidiaries[]{ id,name,fullName,
               directors[]{name,role,type}, signatories[], signCondition } }
directorTraining[]: { id, directorId, directorName, course, institute,
  courseType(iod|governance|exec|external|internal), dateText, year, hours, batch, result, note }
employees[]: { code, name, nameEn, gender, company(PCC|PEM|PSP|PDE|PSL|PPP|SBP|PCE),
  position, jobRole, level, group, dept, div, serviceDate(CE), birthDate(CE), education, major }
  // Master pool พนักงานระดับวิชาชีพขึ้นไป (import จาก Employee XLS) — ใช้เลือกในหน้า Internal Succession
internalSuccession[]: { id, company, layer(1|2|3), parentId, position, order,
  incumbent, incumbentCode, successors[]{ name, code,
  readiness(ready-now|ready-1y|ready-2y|ready-not), idp }, note }
  // โครงสร้าง tree 3 ชั้น/บริษัท: L1=C-Level · L2=Business Manager · L3=Function Manager
  // parentId ผูกชั้น (L1 = '') · drill-down + breadcrumb · ลบ = cascade ลบลูกด้วย
  // defaultSuccPositions() = ตำแหน่ง L1 ตั้งต้น 25 ตำแหน่ง (8 บริษัท) seed เมื่อ DB ว่าง
skillsList[]: {key,label}                  // Board Skills Matrix (แก้/เพิ่ม/ลบได้)
iodPrograms[]: ['DAP','DCP','RCP','BNCP','AACP']
termRules: { termYears:3, indepLimitYears:9, conversionDate:'2561-05-09', minIndependent:3 }
agmYears[]: [2561..2574]   agmDates{ '2561':'2561-04-27', ... }
```
- `migrateDB()` ใส่ default ฟิลด์ที่ขาดทุกครั้งที่โหลด → data.json เก่าจะถูกเติมเองตอน runtime

## 5. เมนู/ฟีเจอร์
| เมนู | สาระสำคัญ |
|------|-----------|
| Dashboard | สรุปภาพรวม |
| การประชุม | วาระ + action items + ไฟล์แนบ (Drive link) |
| **ทะเบียนคณะกรรมการทั้งหมด** | Master pool ของทุกคน (ไม่ใช่บอร์ด PCC อัตโนมัติ) |
| Control List | แท็บ คณะกรรมการบริษัท (derive จากบอร์ด + CG analysis + Board Skills Matrix + อำนาจลงนาม PCC) / คณะย่อย (derive จาก role flags) / บริษัทย่อย (แก้ชื่อ + อำนาจลงนาม + เลือกคนจาก Master) |
| **วาระกรรมการ (Board Term)** | แท็บ Rotation Matrix (คลิกหมุน P/NEW/IN/OUT + คำนวณวาระอัตโนมัติ hybrid) / สรุป&ธงเตือน (ออกตามวาระ, ใกล้ครบ 9 ปี) / ผังกรรมการอิสระรายปี |
| **Internal Succession** | แผนสืบทอดผู้บริหารบริษัทในเครือ — tree 3 ชั้น/บริษัท (L1 C-Level → L2 Business Manager → L3 Function Manager) drill-down + breadcrumb · แต่ละตำแหน่งมี incumbent + ผู้สืบทอด (ความพร้อม 4 ระดับ + IDP) เลือกชื่อจาก Employee Master (datalist) หรือพิมพ์เอง · ลบ = cascade |
| **พนักงาน (Master)** | ทะเบียนพนักงานระดับวิชาชีพขึ้นไป (399 คน, import จาก XLS) ค้นหา/กรองตามบริษัท-กลุ่ม · เป็น pool เลือกผู้สืบทอด |
| ค่าตอบแทน | — |
| **พัฒนากรรมการ** | 3 แท็บ: ภาพรวม (ชม.พัฒนา + สรุปคณะย่อย+IOD compliance) / IOD Certification Matrix / ประวัติการอบรม (filter) |
| ประเมินผล, คลังเอกสาร, AGM Tracker, ผลประโยชน์ฯ, ตั้งค่า | — |

**แนวคิดสำคัญ:** ทะเบียน = Master pool → จัดคนเข้า บอร์ด PCC (`boardRole`) / คณะย่อย (role flags) / บริษัทย่อย · CG analysis + Skills Matrix คิดเฉพาะ "สมาชิกบอร์ด" (boardRole≠none)

## 6. Deployment (สำคัญมาก — เคยมีปัญหา)
- **วิธีที่ใช้:** GitHub Actions → `.github/workflows/deploy-pages.yml`
  - trigger **เฉพาะ `main`** (+ `workflow_dispatch`) — อย่าใส่ branch อื่น (จะ fail แบบไม่มีผล สร้าง noise)
  - ต้องตั้ง repo → **Settings → Pages → Source = "GitHub Actions"**
- **อาการ 401 "Requires authentication"** ตอน deploy = Pages Source ถูกตั้งเป็น "Deploy from a branch" → สลับกลับเป็น "GitHub Actions"
- **`.nojekyll`** อยู่ที่ root (ให้เสิร์ฟไฟล์ทุกตัวรวมชื่อที่มีช่องว่าง)
- ตรวจผล deploy ได้ด้วย MCP: `actions_list`/`actions_get`/`get_job_logs` (output ใหญ่มาก → save แล้ว parse ด้วย python)
- หมายเหตุ: `WebFetch` ไปยัง github.io มักได้ **403** (GitHub บล็อก bot) — ไม่ได้แปลว่าเว็บล่ม เบราว์เซอร์จริงเปิดได้

## 7. Workflow การพัฒนา (ทำซ้ำได้)
- Dev บน branch `claude/sweet-wozniak-clhf9` → PR เข้า `main` → merge
- **ทุก PR main จะ diverge จาก branch** (เพราะ squash/merge) → ก่อน merge PR ถัดไป:
  ```
  git fetch origin main
  git merge -X ours origin/main -m "merge main"   # คงเวอร์ชัน branch
  git push
  ```
  ⚠️ การ merge -X ours **เคยทำให้ฟังก์ชันซ้ำ** (เช่น cgAnalysisHtml) → ตรวจทุกครั้ง:
  `grep -oE "^(async )?function [a-zA-Z0-9_]+" "NRC Management System.html" | sort | uniq -d`
  (มี `addSCMember` ซ้ำที่เป็น pre-existing/ไม่กระทบ — ตัวอื่นซ้ำต้องลบ)
- **ตรวจ JS syntax ก่อน commit เสมอ** (extract `<script>` แล้วรันผ่าน node vm):
  ```
  node -e 'const fs=require("fs");const h=fs.readFileSync("NRC Management System.html","utf8");
  const re=/<script>([\s\S]*?)<\/script>/g;let m,all="";while((m=re.exec(h)))all+=";(function(){"+m[1]+"})();";
  new (require("vm").Script)(all);console.log("OK")'
  ```
- **data.json คือข้อมูลจริงของผู้ใช้** (เขาแก้ผ่าน Save & Publish ตลอด) → ก่อนแก้ data.json ด้วยสคริปต์ ให้ดึงล่าสุดก่อนเสมอ:
  `git fetch origin main && git checkout origin/main -- data.json` แล้วค่อยแก้ → **ห้ามทับด้วยข้อมูลตัวอย่าง**
- การ merge -X ours จะคง data.json เวอร์ชัน branch — ตรวจว่า data.json บน branch = ของจริงล่าสุดเสมอ

## 8. การ Import ข้อมูลจาก Excel (เคยทำ 2 รอบ)
- ใช้ `python3 + openpyxl`
- จับคู่ชื่อกับ `directors[]` ด้วย **token overlap** (ตัด prefix นาย/นาง/รศ.ดร.ภก./คุณ, ตัดเลข, นับคำที่ตรงกันมากสุด) — แม่นแม้สลับนามสกุล/มีคำนำหน้าต่าง
- แปลงวันที่ CE → BE (+543)
- ไฟล์ที่เคย import: ประวัติอบรม (All_Training) → 216 รายการ + iodCerts ; วาระ (pcc_up) → firstAppoint/convAppoint/9ปี/เข้าแทน/rotation 12 กรรมการ
- เขียนกลับ data.json (base = origin/main) แล้ว commit/publish

## 9. ข้อควรระวัง / งานค้าง
1. **roster มี 44 รายชื่อ** แต่บอร์ด PCC จริง ~12 — ตอน migrate ตั้ง `boardRole='member'` ให้ทุกคน → Rotation Matrix/บอร์ดแสดงเยอะเกิน
   → ควรช่วยผู้ใช้ตั้งคนที่ไม่ใช่บอร์ด PCC เป็น "ไม่อยู่ในบอร์ด"
2. **courseType ของ training** จัดด้วย keyword อัตโนมัติ — อาจผิดบางรายการ แก้รายตัวได้ในแท็บประวัติ
3. กรรมการที่ลาออกแล้ว (นารี/สุริยะ/อธิศานต์) ยังไม่ได้ดึงเข้า (อยู่ในชีต Checklist) — ถ้าต้องการประวัติเต็มค่อยเพิ่ม
4. **Token ที่ผู้ใช้เคยวางในแชต** = รั่ว → เตือนให้ revoke เสมอ (ห้ามนำ token ไปใช้/เก็บ)

## 10. กฎ governance ที่ระบบอิงอยู่
- ออกตามวาระ: 3 ปี/วาระ, ~1/3 ออกทุก AGM แล้วเสนอกลับ
- กรรมการอิสระ: เพดาน 9 ปี (นับจากวันแปรสภาพมหาชน 9 พ.ค. 2561 สำหรับคนก่อน listing)
- เกณฑ์ CG: อิสระ >50%, หญิง ≥30%, ไม่เป็นผู้บริหาร >66%, อายุงาน ≤9 ปี
- IOD cert ต่อคณะ: AC→AACP, NRC→BNCP, ประธาน→RCP, ทุกคน→DAP/DCP

---
อ้างอิงเพิ่ม: `SETUP-GUIDE.md` (คู่มือผู้ใช้), `memory/context/nrc-app.md`, `CLAUDE.md`
