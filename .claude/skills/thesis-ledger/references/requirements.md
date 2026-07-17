# Requirements — Locked Scope for 3-Day Build

ที่มา: BA elicitation กับ product owner (3 รอบ) ก่อนเข้า design/implementation phase — เอกสารนี้คือ requirement ฉบับสุดท้ายที่ใช้แทน 5-day plan เดิมใน architecture.md / frontend-spec.md (ตัวเลข/สโคปที่ขัดกับ 2 ไฟล์นั้น ให้ยึดไฟล์นี้เป็นหลัก)

**Build window:** 2 คน, 3 วัน. **Client case:** Alpha Fund (InnovestX).

## 1. Scope

- ต้องมี **หน้าจอจริงให้เดโม** เสมอ — ไม่มี fallback แบบ backend-only ไม่ว่ากรณีใด
- ทีมสร้างจริง = 2 คน, 3 วัน (ไม่ใช่ 5 วัน/ทีมใหญ่กว่าตามแผนเดิม)
- **Data Ledger** = out of scope เต็มรูปแบบ ไม่มี schema hook เผื่อไว้ พูดถึงแค่ใน roadmap slide

## 2. Data & Claims Model

**Seed set — 2 บริษัท (ลดจาก 6):**
- **MWG** (เวียดนาม) — money-shot company, full depth ต้องหา data จริง (งบ/ข่าว/ราคา) ช่วงเม.ย. 2025 ("Liberation Day")
- **CP All (CPALL)** — บริษัทที่ 2 (ยืนยันแล้ว) เลือกเพราะเป็นหุ้นไทย blue-chip สภาพคล่องสูง หา public data ง่ายที่สุด (ความเสี่ยงเรื่อง data-sourcing ต่ำสุด)

**Claims:** 2 claims/บริษัท — 1 load-bearing (ตัวเลข, ใช้ deterministic comparator ได้) + 1 supporting

**Document tier:**

| Tier | เนื้อหา | แหล่ง |
|---|---|---|
| `public` | evidence layer — งบ/ข่าว/ราคา | **ข้อมูลจริง** ทั้งสองบริษัท |
| `inhouse` | thesis/research note — เหตุผลที่ทดสอบ | **mock** โดยทีม (ไม่มี note จริงให้ใช้) |
| `licensed` | — | ไม่ใช้ในรอบนี้ (schema รองรับไว้เผื่ออนาคต) |

ข้อดี: ความน่าเชื่อถือสูงขึ้นจาก grounding จริง โดยไม่แตะ guardrail "ห้ามใช้ licensed data" เพราะ public data ไม่มีความเสี่ยง ToS

- **Entity resolution:** เจอ entity ที่ไม่อยู่ใน map → flag ให้คนตรวจ (ไม่ drop เงียบ ไม่เดา)
- **Freshness nudge:** ระดับ **per-claim**

## 3. Guardrails

**Traffic-light rule (ล็อกแล้ว):**

| สถานะ | เกณฑ์ |
|---|---|
| 🟢 INTACT | ค่าจริง satisfy threshold **และ** (≥2 แหล่งอิสระ **หรือ** 1 แหล่งทางการ) **และ** ไม่มี evidence ค้าน |
| 🔴 BROKEN | evidence ที่มี page/span ชัดเจน ขัด threshold ในทิศทางลบ |
| 🟡 AT_RISK | โมเดล adversarial เห็นไม่ตรงกัน **หรือ** ค่าจริงอยู่ในโซน ±20% ของ threshold |
| ⚪ STALE/NO_DATA | ไม่มี evidence ใหม่ภายใน decay window |

**Staleness decay:** claim อิงงบไตรมาส → STALE หลัง **~100 วัน**, claim อิงข่าว/เหตุการณ์ → STALE หลัง **~30 วัน**

**Guardrail เดิมที่ยังคงอยู่:** grounding บังคับ (ไม่มี page/span = ไม่นับ evidence), abstention (`NO_DATA` ไม่เดา), adversarial checker ไม่ตรงกัน → Verification Queue, human veto ทุก decision, `as_of` ≠ `knowledge_date` เสมอ

**Human-in-the-loop ระหว่างเดโม:** pre-baked/scripted ไม่ live

**ประโยคตอบเรื่อง accountability (ใช้คำนี้เป๊ะ ๆ):**
> "Thesis Ledger ไม่ตัดสินใจแทนคน — Portfolio Manager คือผู้รับผิดชอบการตัดสินใจลงทุนขั้นสุดท้ายเสมอ (fiduciary duty ตาม CFA Institute Standard III(A) — Loyalty, Prudence & Care) ระบบทำหน้าที่แค่ยกธงเตือนพร้อม evidence ต้นทางให้ตรวจสอบ"

## 4. Demo & Evaluation

- เดโมไม่ต้อง live-interactive — scripted/pre-loaded replay พอ (ลด engineering risk ของ Drop & Trace ลงมาก)
- ไม่ทำ backtest/eval framework แบบเป็นทางการ — ถ้ามีตัวเลขบนสไลด์ ให้เป็น hand-verified statement จาก scenario เดียว ไม่ใช่ study เต็มรูป
- **ยังไม่ปิด:** ใครเป็นคนยืนยันว่า MWG money-shot ไม่ใช่ hindsight-fitted (ดู §6)

## 5. UX

- **Freshness-confirm** = ต้อง log เป็น decision ที่มี audit trail (`decided_by` + timestamp)
- **คำต้องห้าม** ในทุก verdict/alert copy: "ซื้อ", "ขาย", "แนะนำ", "การันตี" — ใช้ภาษากลางแทน เช่น "thesis อยู่ในสถานะ BROKEN — REVISIT"
- **ปุ่ม "cite"** = แค่ลิงก์ไปต้นฉบับ (ใช้ page/span ที่มีอยู่แล้ว) ไม่มี audit trail/notify แยกจาก freshness-confirm

## 6. Infra

| รายการ | Decision |
|---|---|
| Slack/LINE notify | **ตัดออกทั้งหมด** จาก scope 3 วันนี้ → เป็น Future Requirement (FR-1) เฟสหลัง |
| Demo environment | laptop |
| API budget | ~3,000 บาท (สบายมากเพราะ seed แค่ 2 บริษัท) |

## 7. Build Plan (3 วัน, 2 คน)

หน้าจอหลัก: **Drop & Trace** (hero/demo screen — เป็นไปได้ใน 3 วันเพราะ confirm แล้วว่า scripted ไม่ live) รอง: **Portfolio Board** (ตารางง่าย ต่อ DB จริง) มี **Verification Queue** แบบ static snapshot ไว้โชว์ guardrail เท่านั้น **Thesis Card ตัดออก** ถ้าเวลาไม่พอ

**Day 1 — Foundation (งานหนักสุด: data sourcing)**
1. ก่อนเริ่มโค้ดจริง: รัน **B1 falsifiability test** (ดู §8) — go/no-go co-authoring UI
2. คนที่ 1: schema (thesis/claim/verdict/decision + support tables) + seed DB + entity map (MWG + CPALL) + หา public data จริงของ MWG
3. คนที่ 2: หา public data จริงของ CPALL + เขียน mock thesis note (tier=inhouse) พร้อม claims ตาม §2 ทั้งสองบริษัท + เริ่ม extraction prompt (Claude API, structured JSON)
4. **DoD:** DB seed ครบทั้งสองบริษัท (public evidence จริง + inhouse thesis mock) + entity map + verdict คำนวณได้อย่างน้อย 1 ตัวจากข้อมูลจริง

**Day 2 — Core loop + hero screen**
1. คนที่ 1: Drop & Trace ต่อ pipeline จริง แบบ scripted/replay
2. คนที่ 2: adjudicator — deterministic comparator (claim ตัวเลขทั้งสองบริษัท) + adversarial check จริงบน load-bearing claim ของ MWG + implement traffic-light rule (±20% band + decay) + freshness nudge (per-claim, audit-logged)
3. **DoD:** MWG money-shot scenario ทำงานถูกต้อง (BROKEN ก่อนราคาตกจริง, `knowledge_date` ถูกต้อง) + CPALL โชว์ verdict ตัดกัน (🟢 หรือ ⚪) ให้ Portfolio Board ดูมีความหลากหลาย

**Day 3 — Integration, polish, rehearsal**
1. ทั้งคู่: Portfolio Board (ขั้นต่ำ) + ปุ่ม cite + Verification Queue static + เช็คคำต้องห้ามทุกจุด + ใส่ accountability line ในโปรดักต์/สไลด์
2. ซ้อมเดโมแบบ scripted เต็มรูปแบบ ≥3 รอบ แก้เฉพาะ show-stopper ห้ามเพิ่มสโคปใหม่
3. **ตัดออกชัดเจน:** Slack/LINE (FR-1), live demo path ใด ๆ, live Verification Queue, backtest/eval framework เป็นทางการ, Thesis Card (ถ้าไม่มีเวลาเหลือ)

## 8. Open Items — ต้องปิดก่อน/ระหว่าง Day 1

1. **B1 — Falsifiability test (ความเสี่ยงอันดับ 1)** ยังไม่มีใครลองแกะ claim จาก research note จริงเลย ต้องทำ manual test 30-60 นาที (แกะ 3-5 claims แบบ `metric + operator + threshold + horizon`) **ก่อน** ล็อกงาน Day 1 — ผลจะบอกว่าต้องเพิ่ม co-authoring UI เข้า scope หรือไม่
2. **D2 — เจ้าภาพยืนยัน money-shot** ยังไม่มีคนรับผิดชอบยืนยันว่า MWG BROKEN-ก่อนราคาตก ไม่ใช่ hindsight-fitted ต้องมีเจ้าภาพ (คนสร้าง 1 ใน 2 หรือคนอื่นในทีมใหญ่) + กำหนดเวลาทำ แนะนำ Day 1-2
