# Architecture — Claim Store, Guardrails, Build Plan

ที่มา: แชท "AI and finance questions for CFA workshop" (dev spec v0.1)

## 1. หน่วยความรู้ = Claim ไม่ใช่ Document

Claim หนึ่งรายการ = `metric + operator + threshold + horizon` เช่น "BHX revenue growth > 20% YoY ภายใน 4 ไตรมาส" — ทำให้เปรียบเทียบกับ evidence ใหม่ได้แบบ deterministic และ falsify ได้จริง

## 2. Schema หลัก (Postgres + pgvector)

**Core tables**
- `thesis` — เหตุผลตอนซื้อต่อหุ้นหนึ่งตัว
- `claim` — pillar ย่อยของ thesis (metric, operator, threshold, horizon, `is_load_bearing`)
- `verdict` — ผลตัดสินต่อ claim: `status` (INTACT / AT_RISK / BROKEN / STALE), `as_of`, `knowledge_date`, confidence
- `decision` — action ของคน (HOLD / ADD / TRIM / EXIT / REVISIT), `decided_by` ต้องเป็นคนเสมอ, `triggered_by` อ้าง verdict, append-only

**Support tables**
- `document` — `tier` (public / licensed / inhouse), `doc_type`, `language` (th/vi/id/en/zh), `can_send_to_llm` (licensed = FALSE)
- `chunk` — text + embedding (VECTOR 1536, pgvector) + page
- `price_daily` — (ticker, date, close)
- `source_health` — Freshness Monitor: last_sync_at, expected_freq, status (OK / STALE / BROKEN)
- `entity_map` — canonical_id + alias หลายภาษา + parent (เช่น Bach Hoa Xanh → MWG); map มือ ~30 ตัวพอ

## 3. Field ที่ทีมมักลืม (แต่กรรมการ CFA จะถาม)

| Field | ทำไมต้องมี |
|---|---|
| `tier` | ขับ policy: licensed → ห้าม persist raw, ห้ามส่ง LLM ภายนอก |
| `as_of` vs `knowledge_date` | กัน lookahead bias ตอน backtest — backtest ด้วยข้อมูลที่ยังไม่รู้ในตอนนั้น กรรมการจับได้ทันที |
| `page_or_span` | บังคับ grounding — ไม่มี citation = ไม่นับเป็น evidence |
| `days_since_evidence` | ขับ staleness decay |
| `is_load_bearing` | triage ว่าคนต้อง verify claim ไหนก่อน |

## 4. Pipeline 4 stage

1. **Thesis extraction** — LLM + JSON schema จาก research paper + confirm UI (คนยืนยันก่อนเข้าระบบ)
2. **Evidence harvester** — filings + news (+ field notes) เข้าเป็น document/chunk
3. **Adjudicator** — deterministic comparator สำหรับ claim ตัวเลข / LLM + adversarial check สำหรับ qualitative
4. **Decision matrix** — thesis status × price → คำแนะนำให้คนตัดสิน

Manual-export sources (Excel, broker email): pipeline อัตโนมัติทำไม่ได้ → ระบบ nudge "claim นี้อิง broker research อัปเดตล่าสุด 3 สัปดาห์ก่อน ยืนยันว่ายังใช้ได้ไหม?" — เปลี่ยน "ข้อมูลไม่อัปเดต" จาก bug เงียบเป็น workflow ที่มีเจ้าภาพ

## 5. Guardrails (มัดกับ implementation)

| Guardrail | Implementation |
|---|---|
| Grounding บังคับ | ไม่มี `page_or_span` → reject evidence |
| Abstention | หาไม่เจอ → `NO_DATA` ไม่ใช่เดา |
| Adversarial checker | โมเดล 2 หาเหตุผลค้าน → ไม่ตรงกัน = ส่งให้คน |
| Human veto | ทุก `decision.decided_by` = คน |
| Audit log | `decision` table immutable (append-only) |
| Tier policy | `tier=licensed` → `can_send_to_llm = FALSE` |

**อ้างอิงกฎไทย (ใส่สไลด์ได้):** ก.ล.ต. — คู่มือกรอบกำกับดูแล AI/ML ในตลาดทุน (กลต.ตท.(ว) 57/2566) / ธปท. 2568 — แนวนโยบายบริหารความเสี่ยง AI (ถูกต้อง น่าเชื่อถือ โปร่งใส อธิบายได้) / คปภ. 2568 — human-in-the-loop สำหรับ AI ความเสี่ยงสูง
**Precedent:** BlackRock Aladdin Copilot ไม่ให้คำแนะนำการลงทุนและไม่ตอบนอกขอบเขตแพลตฟอร์ม — เราขีดเส้นเดียวกัน

## 6. Risks + mitigations

| # | Risk | ระดับ | Mitigation |
|---|---|---|---|
| 1 | research paper ไม่ falsifiable | 🔴 สูงสุด | co-authoring UI + ถาม mentor ก่อน |
| 2 | claim เฉลยช้า (รายไตรมาส) → demo นิ่ง | 🟡 | เสริม leading evidence (ข่าวคู่แข่ง, ถ้อยคำใน MD&A เปลี่ยน) |
| 3 | ข้อมูลไม่อัปเดต → เงียบอย่างมั่นใจผิด ๆ | 🔴 | STALE state + staleness decay + Freshness Monitor |
| 4 | Entity resolution หลายภาษา/บริษัทลูก | 🟡 | map มือ ~30 ตัว |
| 5 | Lookahead bias ตอน backtest | 🔴 | `as_of` แยกจาก `knowledge_date` |
| 6 | ถูกมองว่าซ้ำ "AI news alert" | 🔴 | pitch ด้วยกลไก — เฝ้าคำสัญญา ไม่ใช่ข่าว |
| 7 | ToS ข้อมูล licensed | 🟡 | demo ไม่แตะ licensed data |

## 7. Build plan

- **Phase 0 — Foundation:** schema + seed DB / entity map 6 บริษัท / research paper จำลอง 6 ฉบับ (อิงข้อมูลจริง) / งบ-ข่าว-ราคาย้อนหลัง 4–6 ไตรมาส
- **Phase 1 — Core loop (ห้ามตัด):** Stage 1–3 ของ pipeline
- **Phase 2 — The demo:** Stage 4 + dashboard + **money shot: MWG = BROKEN แต่ราคายังไม่ตก** + Freshness Monitor
- **Phase 3 — Credibility:** backtest Liberation Day + eval numbers (extraction accuracy, lead time) + guardrail demo (adversarial checker จับ hallucination)
- ถ้าเวลาไม่พอ ตัดจากท้าย: Phase 3 → Phase 2 → ห้ามตัด Phase 1

## 8. Liberation Day backtest (เรื่องเล่าปิดท้าย)

เมษายน 2025: ราคาร่วงทั้งกระดาน แต่พอร์ต Alpha Fund เน้น domestic consumption (MWG, Erajaya, MAP) ที่ไม่ได้ export ไปสหรัฐ → ระบบควรบอก "ราคาตก แต่ thesis INTACT → ซื้อเพิ่ม" และมันจะถูก (S&P เด้ง +9.52% วันถัดมา, all-time high ใน 3 เดือน) ⚠️ ต้องใช้ `knowledge_date` ให้ถูก ไม่งั้นเป็น lookahead bias — เรื่องนี้ยังตอกย้ำ insight ว่า discipline คือ edge ที่ยั่งยืน ไม่ใช่ information advantage
