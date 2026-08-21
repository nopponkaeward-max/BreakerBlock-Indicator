# แพลนการประกอบร่าง Indicator — THE POWER OF BREAKBLOCK (EP.01–EP.05)

เอกสารนี้สรุปสิ่งที่เรียนรู้จากไฟล์คอร์สทีละ EP และวิธีที่แต่ละ EP ถูกแปลงเป็นส่วนประกอบของ
`BreakBlock_Indicator.pine` เพื่อให้ตรวจสอบย้อนกลับได้ว่าโค้ดแต่ละส่วนมาจากกฎข้อไหนของคอร์ส

---

## EP.01 — Market Structure Basic

**สิ่งที่คอร์สสอน**
- นิยาม HH (Higher High), HL (Higher Low), LH (Lower High), LL (Lower Low)
- ลำดับการเปลี่ยนเทรนด์: ขาขึ้น→ขาลง = `HH → HL → HH → L (จุดกลาง) → LH → LL`
  และขาลง→ขาขึ้น = `LL → LH → LL → H (จุดกลาง) → HL → HH`
- โครงสร้าง 2 ชั้น: **External** (โครงสร้างหลัก เส้นทึบ) และ **Internal** (โครงสร้างรอง เส้นประ)
  — Internal ใช้จับจุดกลับตัว (การกลับตัว) และเป็นจุดเข้าเมื่อกลับตัวตามทิศ External
- คำถามที่ EP.01 ทิ้งไว้: "ใช้สวิงไหนกำหนด High/Low และใช้ TF ไหน" → ตอบใน EP.02

**แปลงเป็นโค้ด**
- Label `HH / HL / LH / LL` บนจุดสวิงที่ยืนยันแล้ว (เทียบกับสวิงก่อนหน้า)
- สวิงตามเทรนด์ = label ใหญ่ (External-style), สวิงสวนเทรนด์ = label เล็กสีเทา (Internal)
  เปิด/ปิดได้ด้วย `showInternal`

---

## EP.02 — Market Structure Advance (Valid Pullback + IDM)

**สิ่งที่คอร์สสอน**
- ปัญหา: ทุกคน mark High/Low ไม่เหมือนกัน → ต้องมีกติกากลาง = **Valid Pullback**
- **Checklist Valid Pullback ขาขึ้น** (ขาลงกลับด้าน):
  1. หาแท่งที่ทำ HIGH สูงสุดปัจจุบัน **รวมไส้**
  2. LOW ของแท่งนั้น (รวมไส้) โดนเบรก
  3. ราคากลับขึ้นไปเบรก HIGH ของแท่งนั้น → สวิงยืนยัน: High = ยอดแท่ง, HL = จุดต่ำสุดช่วงพัก
- **IDM (Inducement)** = จุด High/Low ของ Valid Pullback ล่าสุด (จุดล่อรายย่อย มีสภาพคล่อง)
- IDM **ย้ายไปสวิงใหม่เสมอ** เมื่อเกิด Valid Pullback ใหม่ — ใช้ตัวล่าสุดเท่านั้น
- High/Low ของ **External ยืนยันได้ก็ต่อเมื่อ IDM ล่าสุดถูก "เคลียร์" (sweep)** แล้วเท่านั้น

**แปลงเป็นโค้ด**
- ตัวตรวจสวิง 2 ตัว (ขาขึ้น/ขาลง) ทำงานทุกแท่งตาม checklist 3 ขั้นเป๊ะ ๆ
  (ขั้น 1–2 ใช้ไส้, ขั้น 3 เลือกได้ผ่าน `useCloseBreak` — ดู EP.03)
- เส้น IDM จุดประสีส้มที่ pivot ล่าสุด ต่ออายุทุกแท่ง, ถูกแทนที่เมื่อเกิดสวิงใหม่,
  ถูก freeze พร้อม label `IDM ✕` เมื่อโดน sweep (ด้วยไส้) + alert "IDM Swept"

---

## EP.03 — Strong Swing / CHoCH / BOS

**สิ่งที่คอร์สสอน**
- ยืนยันจาก EP.02: swing valid pullback **ใช้ไส้ล้วนได้ทุกขั้น** (ไม่ต้องรอปิดแท่ง)
- **BOS** = เบรกโครงสร้างต่อเทรนด์ (ทำ HH ใหม่/LL ใหม่) — BOS ที่แท้จริงต้องมี
  valid pullback หรือเคลียร์ IDM มาก่อน
- **Strong Swing** = สวิงที่ (1) มี valid pullback และ (2) เคลียร์ IDM แล้วทำ extreme ใหม่
  — ลำดับขาลง: `LL > LH > LL > H > L`, ขาขึ้น: `HH > HL > HH > L > H`
- การเบรกยอด LH/HL ธรรมดา = แค่ IDM sweep **ไม่ใช่ CHoCH** (กับดักที่ทำให้โดน SL ซ้ำ ๆ)
- **CHoCH ที่แท้จริง** = การเบรก **จุดกำเนิดของ Strong Swing** (H ในขาลง / L ในขาขึ้น)
  และ TIP สำคัญ: ควรเป็น **การปิดแท่งทะลุไส้** (ยิ่งชัดเจนยิ่งดี)

**แปลงเป็นโค้ด**
- State machine เต็มรูป: trend dir → IDM → IDM cleared → BOS → Strong Swing origin →
  ระดับ CHoCH (เส้นประเฝ้าระวัง) → CHoCH ยิงเมื่อ **close ทะลุ** ระดับนั้น (+`confClose`
  บังคับรอปิดแท่ง realtime)
- เส้น BOS ที่จุดเบรกทุกครั้ง, เส้น+label CHoCH เมื่อเทรนด์พลิก, alert ครบทุกเหตุการณ์

---

## EP.04 — MSS / SR Shadow / BreakBlock

**สิ่งที่คอร์สสอน**
- **MSS** = เบรก pivot กลางระหว่าง extreme สองลูกท้ายของเทรนด์เดิม (แค่ "เริ่ม" กลับตัว)
  → มีนัยยะก็ต่อเมื่อมี **BOS ตามหลัง** ไม่งั้นให้มองข้าม
- **Zone POI** = กล่องจาก extreme ของ pattern ถึงเส้น MSS + เส้นประกลาง 50% (POI)
- Model entry: MSS → BOS → เข้าที่ POI/IDM, SL พ้นโซน, TP ที่ High/Low ฝั่งตรงข้าม
- **SR Shadow** = FVG ที่มองเห็นบน **line chart** — แถบที่ราคาชน/เด้งจากทั้งสองฝั่ง (S/R flip)
- **BreakBlock** = ช่วงราคา (Low→High) ของ **valid pullback ลูกสุดท้าย** ที่ IDM ของมัน
  **ถูกทำลาย** — ถ้าอยู่ในแถบ SR Shadow ประสิทธิภาพยิ่งสูง (OVERLAP)
- Max RR: หาโซนบน TF เล็ก (M5/M3/M1) ในกรอบ pattern ของ M15 (ตัวอย่างคอร์สได้ ~20R)

**แปลงเป็นโค้ด**
- MSS = close ทะลุ IDM pivot สวนเทรนด์ → เส้น MSS + กล่อง POI พร้อมเส้นกลาง 50%
- สร้างโซน BreakBlock 2 ทาง:
  - **T1**: โครงสร้างเบรกผ่านโซน pullback (เกิดพร้อม BOS) → โซนตามทิศเบรก
  - **T2**: IDM ถูกทำลาย (เหตุการณ์ MSS) → โซนฝั่งตรงข้ามที่ pivot เดิม
- SR Shadow ตีความเป็น **gap ระหว่าง body ของแท่ง** (ลดสัญญาณรบกวนจากไส้แบบ line chart)
  กรองความกว้างขั้นต่ำด้วย ATR — BreakBlock ที่ทับซ้อน SR Shadow ติดป้าย `BB ★`

---

## EP.05 — Real BreakBlock (CHoCH Retest)

**สิ่งที่คอร์สสอน**
- **BreakBlock ที่แท้จริง** = โซน Demand/Supply/Hidden Base/Orderblock ที่
  **ต้องมีแท่ง imbalance สร้าง FVG** (บังคับ — FVG คือ confirm ว่าโซน "มีพลัง"),
  โดนเบรกทะลุ/ราคาตีกลับไปกลับมา และใช้เทรดเมื่อโครงสร้างเปลี่ยน (**CHoCH**) แล้วเท่านั้น
- โบนัสความแรง: โซนที่ทำลายโซนฝั่งตรงข้าม หรือเป็นแท่ง breakout ของ BOS เอง
- **Model Setup**: Structure H1 (สร้างสวิง valid pullback → เคลียร์ IDM → CHoCH) →
  BreakBlock M15 (+FVG, OVERLAP) → Refine M5/M3/M1 (โซนเล็กซ้อนในกรอบ M15)
- Entry = ราคาย่อกลับเข้าโซนหลัง CHoCH ("CHOCH RETEST"), SL พ้นโซน refine
- **TP ด้วย Fibo (CR. NINJATHAI)**: ลากจาก A (สวิงกลางที่ถูกเบรก) → B (extreme):
  **TP1 = 4.123, TP2 = 6** (ระดับรอง 4.618 / 6.444 / 6.918)

**แปลงเป็นโค้ด**
- ฟิลเตอร์ `requireFVG` (เปิดเป็นค่าเริ่มต้น): โซน BreakBlock ต้องมี FVG ใน leg ต้นกำเนิด
  (ตรวจระหว่างช่วง pullback + อีก 3 แท่งหลังทริกเกอร์) ไม่มี FVG = ตัดทิ้ง
- สัญญาณ **BB Retest** (วงกลม + alert): ยิงเมื่อราคากลับเข้าโซนที่ทิศตรงกับเทรนด์ปัจจุบัน
  (คือ retest หลัง CHoCH/BOS ตามคอร์ส) — หนึ่งครั้งต่อโซน
- Fibo อัตโนมัติหลัง CHoCH ทุกครั้ง: A = ระดับ CHoCH ที่ถูกเบรก, B = extreme ของ leg
  → เส้น 0 / 1 / TP1 4.123 / TP2 6 (+ระดับรองเลือกเปิดได้)
- Dashboard แสดง HTF bias (ค่าเริ่มต้น H1 + H4) เพื่อทำ workflow หลาย TF ตาม Model Setup

---

## ข้อสมมติ / การตีความ (จุดที่คอร์สไม่ได้ระบุเป๊ะ จึงเลือกแนวทางเอง)

1. **Swing break ขั้นที่ 3**: EP.2 เขียน "ปิดเหนือไส้" แต่ EP.3 ยืนยันว่า "ใช้ไส้ล้วนได้" —
   ค่าเริ่มต้นใช้ไส้ (ตาม EP.3 ซึ่งมาทีหลัง) และมี `useCloseBreak` ให้สลับ
2. **MSS ใช้ close ทะลุ pivot** เพื่อแยกจาก IDM sweep (ที่ใช้ไส้) — คอร์สวาดเป็นแผนภาพ
   ไม่ระบุ wick/close
3. **โซน BreakBlock** ใช้ช่วงราคาแท่ง pivot ของ pullback (Low→High ของแท่งที่ทำจุดกลับ)
   ตามภาพ "คลุมทั้งโซนของ valid pullback จาก LOW ถึง HIGH"
4. **SR Shadow** ใช้ gap ระหว่าง body แทน "gap บน line chart" (ให้ผลใกล้เคียงและคำนวณได้เสถียร)
5. **Multi-TF**: คอร์สใช้วิธีสลับ TF ด้วยมือ (H1→M15→M5) — อินดิเคเตอร์คำนวณโครงสร้างบน
   TF ของชาร์ต และให้ HTF bias บน dashboard; การ refine โซนบน TF เล็กให้เปิดชาร์ต TF เล็ก
   โดยดู bias จาก dashboard ให้ตรงกัน

## แผนต่อยอด (ยังไม่ทำ)

- เวอร์ชัน `strategy()` สำหรับ backtest ตาม Model Setup EP.5 (entry ที่ BB retest,
  SL พ้นโซน, TP1/TP2 ตาม Fibo)
- วาดโซน M15 อัตโนมัติขณะอยู่บนชาร์ต M5 ผ่าน `request.security`
- แยกความแรงของโซน (ทำลายโซนฝั่งตรงข้าม / เป็นแท่ง breakout เอง) ตาม TIP ของ EP.5
