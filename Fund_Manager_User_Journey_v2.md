# Persona: Fund Manager User Journey (v2 — updated)

> **หมายเหตุโครงสร้าง (สำคัญ ต้องอ่านก่อน):**
> - Swimlane "PM" และ "Analyst" ในเอกสารนี้คือ **การแบ่งตามหน้าที่ (role-based)** ไม่ใช่ตำแหน่งคน ในทีมขนาดเล็ก (2-3 คน) คนคนเดียวอาจทำหน้าที่ทั้ง PM และ Analyst พร้อมกัน ("Assistant Fund Manager" ที่ทำแทบทุกอย่างเหมือน Fund Manager) — โครงสร้างจริงต่างกันไปตามแต่ละที่ บางที่มี Research แยกออกจาก Fund โดยเฉพาะ บางที่ Fund/Assistant Fund ทำ Research เอง
> - การแบ่งงาน coverage หุ้นในทีมเล็กมักเป็นแบบ **ไว้ใจกัน (trust-based)** ต่างคนต่างดูหุ้นคนละตัว ไม่ใช่การ assign แบบทางการเสมอไป

---

## Phase 1: Research

**Client Goal / Committee**
- 0. Requirement: ส่วนราชการได้แผนไหน หรือเกณฑ์/criteria ต่าง ๆ

**Idea Sourcing (ใหม่)**
- **0.5 Idea Sourcing:** ฟัง broker call / รับข้อมูล (ลักษณะเดียวกับ SCB) + อ่าน research report → เป็นจุดเริ่มต้นจริงของไอเดียส่วนใหญ่ ก่อนจะเข้าสู่การรีเสิร์ชเชิงลึก

**PM**
- 1. Research Assign: PM assign Research ให้ Analyst
  - *(ปรับ: เป็นลูกศร 2 ทาง — PM/manager เองก็เห็นหุ้นน่าสนใจแล้วฝากให้ Analyst ช่วยดูได้เช่นกัน ไม่ใช่ PM สั่งทางเดียวเสมอไป)*
- 2. Submit / Wait for approval Research
- 3. Presenting: Analyst นำเสนอ idea ในที่ประชุมให้ PM
  - *(หมายเหตุ: รับ input จากหลาย track คู่ขนานได้ — เมื่อต่างคนต่างดูหุ้นคนละตัว แต่ละคนเอาหุ้นของตัวเองเข้าที่ประชุมพร้อมกัน)*
- 4. Thesis construction: เอาข้อมูลมาสรุปเป็น plan & thesis, PM เขียนคอนเซ็ป/บทวิเคราะห์
- 4. Processing: Idea ที่ผ่านการกลั่นกรองเข้า Watchlist ของ PM

**Analyst**
- 1.1 Research: Combine Public + External (macro, sector, company, ทุน)
- 1.2 Research: Analyst ทำ Valuation Model / DCF
- 1.3 Research: Analyst ทำ Internal interview
- 1.4 Research: Meeting ผู้บริหาร, Field insight, summary
- **1.5 Informal Discussion (ใหม่):** Analyst + PM (หรือคนในทีมเล็ก) คุยกันแบบปากเปล่า (นั่งด้วยกัน) ว่าหุ้นตัวนี้น่าสนใจไหม
  - ถ้า **น่าสนใจ** → ไปทำ deep-dive ต่อ (overview, financial statement analysis, valuation model) แล้ววนกลับมาคุยกันอีกครั้ง ก่อนเสนอเข้าที่ประชุม
  - ถ้า **ไม่น่าสนใจ** → พักไอเดียไว้ ไม่ไปต่อ
- 4.1 Team construction: คัดเลือกข้อมูลมา construct thesis, หาข้อมูลมา support thesis
- 4.2 Financial model Testing: ทดสอบสมมติฐานต่าง ๆ ของ thesis

**Current use (AI ปัจจุบัน)**
- External + Internal AI: สรุป/ดึงข้อมูล purchased research ให้ง่ายขึ้น, โหลดข้อมูลทำ excel และช่วยสรุปกับโมเดล — เช่น NotebookLM, Claude, RAG
- Internal AI: ช่วยตอบคำถามจากเอกสารผู้บริหาร (background + prompt) — เช่น ChatGPT, Claude
- Internal AI: ช่วยสรุปเอกสารภายใน/ทั่วไป ให้เร็วและแม่นยำขึ้น — เช่น ChatGPT, Claude

**Output**
- รายชื่อหุ้น/ตราสารที่มี investment thesis ชัดเจน พร้อม conviction level

**Gaps**
1. **Gap of Researching** (Analyst usecase): ช่วยเช็คข้อมูล ตรวจข้อมูล หาข้อมูลจากการ research ให้ง่ายขึ้น → Impact: หาง่าย → ลดเวลา
2. **Gap of Reviewing Researching** (Analyst usecase): หาจากช่องทางที่เคยทำไว้แล้ว ให้ละเอียด/แม่นยำมากขึ้น
3. **Gap of Verifying** (PM usecase): ช่วยเช็ค/ตรวจสอบข้อมูลก่อนตัดสินใจ; คำถามเปิด: เกณฑ์ที่มีอยู่เข้าใจยากไหม ต้องปรับปรุงบ่อยแค่ไหน ปฏิบัติถูกต้องตามกฎหมายหรือไม่ → Impact: หาง่าย + ตรวจง่าย → ลดเวลา
4. **Gap of Capturing Informal Discussion (ใหม่)** (Analyst/PM usecase): บทสนทนา/การคุยกันปากเปล่าในทีมเล็กไม่ถูกบันทึกไว้ ความรู้อาจหายไปเมื่อทีมโตขึ้นหรือมีคนออก — โอกาส AI: ช่วยสรุป/บันทึกบทสนทนาทีมหรือสรุป broker call เป็นข้อความที่ retrieve ย้อนหลังได้ → Impact: เก็บความรู้ได้ + หาย้อนหลังง่าย → ลดการสูญเสีย knowledge

---

## Phase 2: Portfolio Construction & Decision

**Committee**
- 4.1 Committee refining: จัดลำดับความสำคัญ high impact จากเหตุการณ์เปลี่ยนแปลง — แนวคิด Investment Committee (IC) เห็นชอบ

**PM**
- 5. Comparing: PM พิจารณา idea จาก Research เทียบกับ IPS/Guideline ของแต่ละกองทุน
- 6. Decision: PM ตัดสินใจกำหนดน้ำหนัก weight/position size และพิจารณาปัจจัยสนับสนุนอื่น ๆ
- 7. Model Test (Risk management): สร้างโมเดลประเมินความเสี่ยง (risk model), pre-trade compliance (limit, VaR) ที่เกี่ยวข้อง

---

## Phase 3: Trade Execution

- 8. Execute: PM สั่งคำสั่งซื้อผ่าน OMS (Order Management System) / Manual
- 9. Broker confirm?: Dealer/Trader ยืนยัน execution (Broker, algo, block trade) เลือก best execution
- 10. Confirmation: ยืนยันการซื้อขาย

**Compliance**
- 11. Security & Checking: ตรวจสอบธุรกรรมให้ถูกต้อง custodian ตรงกับสภาพคล่อง
- Output: ทรัพย์สินอยู่ในบัญชี custodian ถูกต้องครบ บันทึกใน database bank

---

## Phase 4: Operation

- 12. Reviewing & Monitoring: Back Office / Fund Accounting บันทึกรายการทางบัญชี
- 12.1 Reviewing & Monitoring: Tracking position รายวัน เช็คความเรียบร้อย

---

## Phase 5: Monitoring

**Compliance**
- 13. Reviewing Compliance: Fund Administrator (ภายนอก) ตรวจสอบ NAV (independent NAV check)
- 14. Daily Compliance: Compliance ตรวจ trade compliance ทุกวัน และตรวจสอบธุรกรรมรายวันให้ถูกต้อง

**PM**
- 13. Daily Review Position: PM ทบทวนสถานะเทียบ benchmark และ IPS อย่างสม่ำเสมอ

**Current use (AI ปัจจุบัน)**
- Internal AI: AI for monitoring and report — Flow ยังไม่ชัดเจน — เช่น Claude, LLM

**Output**
- NAV ที่เชื่อถือได้ + รายงานความเสี่ยงรายวัน

**Gaps**
5. **Gap of Verifying II** (PM usecase): เช็คข้อมูลที่เกี่ยวข้องว่าตรงกับเป้าหมาย ครบถ้วนหรือไม่ → Impact: หาง่าย + ตรวจง่าย → ลดเวลา
6. **Gap of Monitoring** (PM usecase): จุดที่ต้องเช็คต่อเนื่อง (รายละเอียดยังไม่ชัดเจนในต้นฉบับ ควร follow-up เพิ่ม) → Impact: หาง่าย + ตรวจง่าย → ลดเวลา

---

## Phase 6: Rebalancing & Portfolio Review *(ฉบับร่าง — ไม่มีรายละเอียดในไดอะแกรมต้นฉบับ จึงร่างขึ้นตามกระบวนการมาตรฐานของการบริหารกองทุน โปรดตรวจสอบและปรับแก้กับ flow จริง)*

**PM / Analyst**
- 15. Performance Review: PM/Analyst ทบทวนผลตอบแทนพอร์ตเทียบ benchmark, ทำ performance attribution (ตัวไหนทำให้ได้/เสียผลตอบแทน)
- 16. Thesis Revisit: ทบทวนว่า investment thesis เดิมยังใช้ได้ไหม — ราคาถึงเป้าหมายหรือยัง, ปัจจัยพื้นฐานเปลี่ยนไปหรือไม่, catalyst ที่คาดไว้เกิดขึ้นแล้วหรือยัง
- 17. Rebalancing Decision: PM ตัดสินใจ trim / add / exit position ตาม weight drift, risk limit ที่เกินเกณฑ์ หรือ idea ใหม่ที่เข้ามาแทน

**Committee**
- 18. Portfolio Review Meeting: PM นำเสนอสรุปภาพรวมพอร์ตให้ Committee/IC ทบทวน และขออนุมัติการปรับพอร์ตถ้าจำเป็น

**Loop back**
- คำสั่งปรับพอร์ตที่อนุมัติแล้ว → วนกลับไปที่ **Phase 2 (Decision)** และ **Phase 3 (Trade Execution)** เพื่อดำเนินการซื้อ/ขายจริง

**Output**
- พอร์ตที่ปรับสมดุลแล้วตาม IPS/เป้าหมายความเสี่ยง + thesis/conviction level ที่อัปเดต

**Gap (ร่าง — เสนอเพื่อ validate กับ user เพิ่มเติม)**
7. **Gap of Thesis Revisit** (PM/Analyst usecase): ช่วยเช็คว่า thesis เดิมยัง valid อยู่ไหมเมื่อเทียบกับข้อมูล/ข่าวใหม่ ๆ, แจ้งเตือน (alert) เมื่อมี catalyst หรือปัจจัยพื้นฐานเปลี่ยนแปลงที่กระทบ thesis → Impact (คาดการณ์): ลดโอกาสถือหุ้นตาม thesis ที่ล้าสมัยไปแล้ว, ตรวจพอร์ตได้เร็วขึ้น

---

## สรุป Pattern ของ Gap ทั้งหมด

Gap ส่วนใหญ่ (1, 2, 3, 5, 6) วนอยู่รอบ **"การหาข้อมูล"** และ **"การตรวจสอบ/verify ความถูกต้อง"** ทั้งฝั่ง Analyst (ตอน research) และฝั่ง PM (ตอนตัดสินใจ + monitoring) — เกือบทุก gap ให้ impact แบบเดียวกันคือ "หาง่าย + ตรวจง่าย → ลดเวลา" ส่วน Gap ใหม่ที่เพิ่มเข้ามา (4, 7) เป็นเรื่อง **การรักษาความรู้ (knowledge capture)** และ **การเฝ้าระวัง thesis ที่ล้าสมัย (thesis decay monitoring)** ซึ่งเป็นมิติที่ diagram เดิมยังไม่ครอบคลุม
