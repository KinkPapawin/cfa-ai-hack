---
name: thesis-ledger
description: Project context and locked decisions for "Thesis Ledger" — the CFA × AI hackathon product for the Alpha Fund (InnovestX) case. Use this skill whenever the conversation involves Thesis Ledger, Data Ledger, Data Layer, the Alpha Fund case, the hackathon demo/pitch/slides, claim extraction, traffic-light monitoring (INTACT/AT_RISK/BROKEN/STALE), the Drop & Trace screen, or any dev/frontend/backend work for this project — even if the user doesn't name the product explicitly. Read references/architecture.md before backend/data/pitch work, and references/frontend-spec.md before UI/demo work.
---

# Thesis Ledger — Project Skill

## Product ในหนึ่งย่อหน้า

Thesis Ledger เฝ้าดูว่า **เหตุผลตอนซื้อหุ้นแต่ละตัวยังจริงอยู่ไหม** โดยแปลง research paper เป็น claim ที่พิสูจน์เท็จได้ (metric + operator + threshold + horizon) แล้วเก็บ evidence มาตัดสินสถานะต่อเนื่อง: **INTACT / AT_RISK / BROKEN / STALE** ระบบแนะนำเท่านั้น — ทุก decision ต้องมีคนเซ็น

## บริบทลูกค้า (Alpha Fund, InnovestX)

- ทีมลงทุน ~7–8 คน ดูหุ้น ~20–30 ตัว ใน 5 ตลาด (TH, VN, ID, CN, IN) / AUM โต ~4–5 พันล้าน → ~12,000 ล้านบาท โดยแทบไม่เพิ่มคน
- Monitoring ปัจจุบัน: Excel/Google Sheets ผูกราคาปิด, rule-based (ถึง target → revisit, ลง 30% → cut loss ครึ่ง), ไม่ใช้ AI ในการตัดสินใจ
- **ช่องว่างหลัก:** ทุก trigger ยิงเมื่อ "ราคา" เปลี่ยน — ไม่มีตัวไหนยิงเมื่อ "ความจริง" เปลี่ยน ถ้าราคานิ่งแต่ธุรกิจแย่ลงเงียบ ๆ จะไม่มีใครรู้จนราคาตก
- Slide ของลูกค้ามี agent อยู่แล้ว 8 ตัว (content generation, reconciliation, ops mail ฯลฯ) — **ไม่มีตัวไหนทำ thesis monitoring** นี่คือ whitespace ที่เราชิง

## โจทย์ 3 ข้อที่ mentor บอกเองว่ายังไม่มีคำตอบ (= design constraints)

1. **Verify** output ของ AI ยังไง → ทุก evidence ต้องมี provenance (page/span) + conflict check ข้ามแหล่ง
2. **Combine** public data กับ in-house research ยังไง → in-house note เป็น first-class data มี tier แยก
3. **Quantify** ปัจจัย qualitative (CG, ownership) ยังไง → scorecard ที่มี rubric + evidence link

## Decisions ที่เคาะแล้ว (อย่าเปิดใหม่โดยไม่มีเหตุผลใหม่)

| Decision | เหตุผล |
|---|---|
| Pitch **Thesis Ledger แคบ ๆ** — Data Ledger เป็น roadmap item | ถ้า pitch กว้างจะไปชนกับ enterprise KM tools และเสียคะแนน problem ที่เฉพาะเจาะจง |
| **Claim Store ไม่ใช่ Data Lake** — unit of knowledge = structured claim | scale จริง ~3,000 rows; document-level storage ตอบโจทย์ verify ไม่ได้ |
| **Postgres + pgvector ไม่ใช่ Databricks** | ผิด scale, ผิด bottleneck, licensing ห้าม bulk-store purchased data, ไม่มี data engineering team; วางท่าเป็น "Databricks-compatible" สำหรับคุยระดับ production |
| Demo **ไม่แตะ licensed data เลย** | เลี่ยง ToS risk ทั้งก้อน |
| Money shot ของเดโม: **MWG = BROKEN แต่ราคายังไม่ตก** | พิสูจน์ว่าเราเห็นก่อนตลาด |
| Positioning: **thesis-health nowcast** ไม่ใช่ price predictor | เลี่ยงเส้น "คำแนะนำการลงทุน" + ตรง precedent BlackRock Aladdin Copilot |
| Pitch ด้วย **กลไก** ไม่ใช่หัวข้อ | กัน "ก็แค่ AI news alert" — เราเฝ้า *คำสัญญา* ไม่ใช่ *ข่าว* |

## กติกาที่ห้ามละเมิด (guardrails — ใช้ตอบคะแนน feasibility)

- **Grounding บังคับ:** ไม่มี `page_or_span` → ไม่นับเป็น evidence
- **Abstention:** หาไม่เจอ → `NO_DATA` ห้ามเดา
- **Adversarial checker:** โมเดลตัวที่สองหาเหตุผลค้าน — ไม่ตรงกัน = ส่งเข้า Verification Queue
- **Human veto:** ทุก decision ต้องมี `decided_by` เป็นคน / audit log append-only
- **Tier policy:** `tier=licensed` → ห้ามส่งเข้า LLM ภายนอก ห้าม persist raw
- **as_of ≠ knowledge_date:** แยกเสมอ กัน lookahead bias ตอน backtest
- **STALE ≠ INTACT:** ระบบห้ามบอก "ปลอดภัย" ทั้งที่จริงคือ "ไม่มีใครไปเช็ค" — confidence ต้อง decay ตามเวลา

## คำตอบ Q&A ที่ซ้อมไว้แล้ว

> **"ถ้าข้อมูลไม่ update ระบบจะพังไหม?"** — ระบบไม่ "พัง" แต่จะ "เงียบอย่างอันตราย" ซึ่งกันไว้ 2 ชั้น: (1) confidence decay ตามเวลาที่ไม่มีข้อมูลใหม่ (2) สถานะที่สี่ STALE แยกจาก INTACT ชัดเจน

## อ่านต่อ

- **references/requirements.md** — requirement ฉบับล็อกสุดท้าย (3 วัน/2 คน) แทนที่ scope เดิมด้านล่าง: seed set MWG + CPALL, traffic-light rule ที่เป็นตัวเลขจริง, build plan Day 1–3, open items ที่ยังไม่ปิด (falsifiability test, money-shot verification owner) — **อ่านไฟล์นี้ก่อนเสมอเมื่อทำงาน scope/build-plan**
- **references/architecture.md** — schema (thesis/claim/verdict/decision + support tables), field ที่กรรมการ CFA จะถาม, risks + mitigations, กฎ regulator ไทย, Liberation Day backtest (⚠️ build plan Phase 0–3 และ seed 6 บริษัทในไฟล์นี้ถูกแทนที่ด้วย requirements.md แล้ว)
- **references/frontend-spec.md** — 4 หน้าจอ (Portfolio Board, Drop & Trace ⭐, Thesis Card, Verification Queue), traffic engine 2 ชั้น, tech stack, วิธี present ความน่าเชื่อถือ (⚠️ color-logic ตัวเลขจริงและ build order 5 วันในไฟล์นี้ถูกแทนที่ด้วย requirements.md แล้ว)

## Open questions (จาก SKILL.md เดิม — ได้คำตอบแล้วใน requirements.md §8, ยังไม่ปิดสมบูรณ์)

1. ✅ ตอบบางส่วนแล้ว — ยังต้องรัน falsifiability test จริงก่อน Day 1 (ดู requirements.md §8, B1)
2. ✅ ตอบแล้ว — freshness nudge เป็น per-claim (ดู requirements.md §2)
