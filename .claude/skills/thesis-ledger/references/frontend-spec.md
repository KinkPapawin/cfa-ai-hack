# Frontend Spec — 4 หน้าจอ, Traffic Engine, Build Order

ที่มา: แชท "Frontend implementation strategy for product"

## หลักคิด

Frontend ไม่ใช่ dashboard สวย ๆ แต่เป็น UI ของ 3 loop:

| Loop | หน้าจอที่ surface |
|---|---|
| Input loop (ข้อมูลใหม่ → ตัดสิน severity) | **Drop & Trace** |
| Severity → footprint → alert (ใครโดนกระทบ) | **Impact Panel** (ฝังใน Trace) + **Verification Queue** |
| Output loop (คนดึง fact ไปใช้ → บันทึก footprint) | ปุ่ม **"cite"** ทุกจุดที่แสดง fact |

มี 4 หน้าจอพอ — หน้าชูโรงคือ Drop & Trace

## หน้า 1: Portfolio Board (เปิดเจอทุกเช้า)

- เรียง 🔴 → 🟡 → 🟢 → ⚪ / ตัวที่ไม่มีอะไรเปลี่ยนพับเก็บ
- **ไม่มีอะไรเปลี่ยน = จอเกือบว่าง** ห้าม spam — ไฟส้มปลอมเยอะ = ความน่าเชื่อของทั้งระบบตาย
- 🟢 ต้องโชว์ด้วย: เราแสดงข่าวดีที่เพิ่ม conviction ไม่ใช่แค่ข่าวร้าย (จุดต่างจาก alert ทั่วไป)
- ตัวอย่างแถว: `🔴 MWG VN — Margin claim ขัดกับ field note ของเรา` / `🟢 BBCA ID — งบ Q2 ยืนยัน loan-growth claim (2 ขา)`

## หน้า 2: Drop & Trace ⭐ (demo moment)

1. ลากไฟล์ (เช่น broker_MWG_Q2.pdf) มาวาง → แสดงสถานะอ่านทีละหน้า
2. ผลสตรีมสด: "สกัดได้ 14 assertions — ⚪9 ไม่แตะ claim ไหน / 🟢3 ยืนยัน claim เดิม (คนละแหล่ง) / 🔴1 ขัด C1 — หน้า 12 บรรทัด 4 / 🟡1 กำกวม → เข้าคิว verify"
3. คลิก 🔴 เปิด **Trace view side-by-side**: ซ้าย = หน้า PDF ไฮไลต์ประโยคจริง / ขวา = Claim + เกณฑ์ + evidence เดิม (รวม field note เสียง)
4. Impact Panel: claim นี้ถูก cite ที่ไหนบ้าง → ใครต้องรู้

## หน้า 3: Thesis Card (ต่อหุ้นหนึ่งตัว) — ⚠️ ใน prototype ล่าสุดเปลี่ยนชื่อเป็น "Thesis Evidence" และ redesign ใหม่ทั้งหน้า (support points แทน claim list เดิม) — ดู `handoff/BRIEF.md` §11–12

สถานะราย claim + evidence ที่ลิงก์ได้ทุกชิ้น + ประวัติ verdict — ใช้เป็นหน้า drill-down จาก Portfolio Board

## หน้า 4: Verification Queue

รายการ 🟡 ที่ adjudicator ไม่มั่นใจ หรือ adversarial checker ขัดกัน → คนตัดสิน แล้วผลป้อนกลับเป็น ground truth

## Traffic Engine (compute สี)

- **ชั้น 1 — Deterministic comparator:** claim ตัวเลข เทียบ operator/threshold ตรง ๆ (Claim เป็น join key)
- **ชั้น 2 — LLM adjudicator + adversarial check:** claim เชิงคุณภาพ; สองโมเดลขัดกัน → 🟡 เข้าคิว
- **Color logic:** 🟢 ต้องมี evidence จาก **2 source tier อิสระ** / 🔴 ต้อง traceback ถึง page + character offset ได้เสมอ / หาไม่เจอ = ⚪ (NO_DATA) ไม่ใช่ 🟢

## Tech stack (เคาะแล้ว)

| ชิ้น | เครื่องมือ | เหตุผล |
|---|---|---|
| PDF parsing | PyMuPDF | เก็บ page + character offset สำหรับ traceback |
| Assertion extraction | Claude API (structured JSON output) | schema-locked, ลด parsing error |
| Embedding + search | bge-m3 + pgvector | multilingual (th/vi/id/zh/en) |
| Backend | FastAPI + background tasks | สตรีมผล Drop & Trace |
| Alert demo | Slack webhook | เห็นภาพ workflow จริง |
| Office add-in / browser extension | ❌ ตัดออกจาก MVP | ใส่ roadmap พร้อมเหตุผล |

## Data contract

ทุก assertion ที่ส่งเข้า frontend ต้องมี provenance fields ครบ: `document_id, page, span (char offsets), source_tier, extracted_text, matched_claim_id, verdict, confidence`

## Build order 5 วัน (Definition of Done ต่อวัน)

| วัน | งาน | DoD |
|---|---|---|
| 1 | Schema + seed DB + entity map | query ได้จาก DB จริง |
| 2 | Ingestion + extraction prompt + entity map | โยน PDF → assertion JSON พร้อม page/บรรทัด |
| 3 | Portfolio Board + Thesis Card ต่อ DB จริง | เปิดจอเห็น 🔴🟡🟢⚪ จาก seed |
| 4 | Drop & Trace (สตรีม + side-by-side) + Verification Queue | ลากไฟล์สด → ไฟติด → คลิกเห็นหน้า/บรรทัด |
| 5 | Footprint panel + Slack alert + วัด precision บน seed set + ซ้อม demo | ตัวเลข precision จริงขึ้นสไลด์ |

## วิธี present ความน่าเชื่อถือ

- อย่าอ้าง "แม่น 95%" ลอย ๆ — รายงาน **precision/recall บน seeded test set** ที่ฝังคำตอบไว้แล้ว
- Positioning: ระบบเป็น **thesis-health nowcast** ที่กระตุ้นให้ revisit valuation — ไม่ใช่ price predictor ไม่ใช่คำแนะนำการลงทุน
