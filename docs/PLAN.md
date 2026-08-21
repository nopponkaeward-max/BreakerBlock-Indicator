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

## Breakblock_S — Pattern Reversal W/M + Re-Acc/Re-Dist + BreakBlock A++

**สิ่งที่คอร์สสอน**
- **Pattern Reversal W (กลับตัวขึ้น) / M (กลับตัวลง)** 4 แบบ: W2, W3, M2, M3
  - ทุกแบบ: BOS + Valid Pullback → retest โซน Supply/Demand (Internal หรือ External) →
    **ฟิลเตอร์ไหล่ 50%**: ไหล่ (สวิงที่ไม่ใช่จุด extreme) ต้องอยู่ฝั่ง extreme ของเส้นกลาง 50%
    ของช่วง High↔Low (W: ไหล่ต่ำกว่า 50%, M: ไหล่สูงกว่า 50%) → BOS ทะลุโซน External = ยืนยัน
  - W2/M2 = ไหล่มาก่อนจุด extreme (หัวมาทีหลัง), W3/M3 = ไหล่มาหลัง (reject)
  - Fibo: 0% ที่ extreme, 100% ที่ระดับ retest — **TP ที่ 300% / 400% / 600%**
- **Re-Distribution / Re-Accumulation**: หลังกลับตัว ราคาย่อหลอกกลับไปเคลียร์สวิง/โซน
  imbalance และแตะ **neckline** (Low ของ M / High ของ W) แล้วค่อยทำ external LL/HH ใหม่
  = "BOS ครั้งสุดท้าย" ยืนยันไปต่อ
- **BreakBlock เกรด A++** = Demand RBR / Supply DBD / Hidden + **imbalance** →
  Overlap → BOS (สูตร: `D/S/Hidden + IMBALANCE → OVERLAP → BOS = BREAKBLOCK`)
  retest แล้วทำ High/Low ใหม่ = ยิ่งเพิ่มคุณภาพ

**แปลงเป็นโค้ด**
- ตรวจ W/M อัตโนมัติ ณ จังหวะ CHoCH: หาไหล่ (สวิงโลว์/ไฮลูกที่ไม่ใช่ extreme จาก 2 ลูกล่าสุด)
  เทียบกับเส้นกลาง 50% ของช่วง CHoCH-level ↔ extreme → ผ่าน = label `W ✓` / `M ✓` + alert
- ชุดเป้า Fibo เลือกได้: `4.123 / 6 (EP.5)` หรือ `300 / 400 / 600% (W/M)`
- Re-Acc/Re-Dist: หลัง CHoCH เก็บ neckline = ระดับที่ถูกเบรก; ถ้าราคาย้อนมาแตะ
  แล้วเกิด BOS ใหม่ตามเทรนด์ → label `Re-Acc ✓` / `Re-Dist ✓`

## BREAKBLOCK STRATEGY DAY TRADE — Liquidity / Range / POI

**สิ่งที่คอร์สสอน**
- **Liquidity**: สวิง = รูปแบบ 3 แท่ง (แท่งกลางสูง/ต่ำสุดรวมไส้ สีไม่สำคัญ);
  เหนือสวิงไฮมี Buy Stops = **BSL**, ใต้สวิงโลว์มี Sell Stops = **SSL** — ตลาดมัก
  ไล่เก็บ (Stop Hunt) แล้วกลับตัว; **EQH/EQL** (เท่ากันหรือเกือบเท่า) = แหล่งสภาพคล่องใหญ่;
  trendline ที่แตะ ≥3 ครั้งก็เป็นแหล่งสภาพคล่อง
- **Trading Range**: **QM** = stop hunt + BOS สวนทาง; จุดเริ่ม range = Strong High/Low
  (ควรยัน), ปลาย range = Weak High/Low = **ERL** (เป้าหมาย); สวิงภายใน = **IRL**;
  วัฏจักร: ราคาวิ่งหา IRL → ERL → IRL วนไป; Range Traps 2 แบบ (leg ย่อ = trap,
  failure swing = trap)
- **POI 4 แบบ**: Orderblock (แท่งสวนสีสุดท้าย + ควรมี FVG, ตีกรอบจาก body),
  Breaker Block (OB ที่ถูกทะลุ **1 ครั้ง**), Hidden Base (base ของ LTF ซ่อนใน HTF),
  **BREAKBLOCK = RBR/DBD ที่ Overlap ถูกทะลุ ≥2 ครั้ง** (มองง่ายสุดบน line chart;
  TF ≥M15 ใช้ body, ต่ำกว่า M15 รวมไส้)
- กฎ POI: ต้องมีสภาพคล่องเหนือโซน (ฝั่ง buy) / ใต้โซน (ฝั่ง sell) และเลือกโซนที่
  **ใกล้สภาพคล่องที่สุด**
- Setup ทั้ง 3: HTF POI (M15/M30) + BREAKBLOCK LTF (M5/M3/M1) ข้างใน —
  entry ที่ Breakblock, **SL ปลอดภัยสุด = พ้นโซน HTF POI**, **TP = สวิงล่าสุด**
  (ตัวอย่าง RR 36.4 / 27.2 / 23.36)

**แปลงเป็นโค้ด**
- กลุ่ม **Liquidity**: pivot 3 แท่ง (รวมไส้) → เส้นจุดประ BSL/SSL ต่ออายุจนโดน sweep
  (label `BSL ✕` / `SSL ✕`), ตรวจ **EQH/EQL** ด้วยค่าเผื่อ xATR (label `EQH $$$`)
- โซน BreakBlock เปลี่ยน lifecycle: ไม่ลบเมื่อถูกทะลุ แต่**นับ Overlap ×N**
  (ทะลุ 1 ครั้ง = Breaker Block, ≥2 = BREAKBLOCK แท้ตามคอร์ส) — ลบเมื่อเกิน
  `maxOvl` หรือโซนแก่เกิน `zoneAge` แท่ง
- Label `Strong High/Low` (จุดที่ควรยัน) และ `Weak High/Low (ERL)` (เป้าหมาย) หลัง CHoCH

## REAPER BUY/SELL MODEL (จากภาพประกอบ)

**สิ่งที่โมเดลสอน** (Phase A → B → C)
1. **Phase A**: เบรก HIGH เดิม (buy) / LOW เดิม (sell) — โครงสร้างยืนยันทิศ
2. **Phase B**: สร้าง Valid Pullback แล้ว**โดนเคลียร์** → ระดับ pullback ที่ถูกเคลียร์
   คือ **Stoploss Retail** (จุดที่ SL ของรายย่อยกองอยู่)
3. **Phase C**: ราคาลงต่ำกว่า (buy) / ขึ้นเหนือกว่า (sell) เส้น Stoploss Retail —
   **หา Breakblock ในบริเวณนั้นเป็นจุดเข้า**

**แปลงเป็นโค้ด**
- เมื่อ IDM ถูก sweep → วาดเส้นแดง **Stoploss Retail** ที่ระดับนั้น ต่ออายุจนเกิด CHoCH
  หรือมี sweep ใหม่ (+แสดงบน dashboard)
- สัญญาณ **Reaper Buy/Sell** (รูปเพชร + alert): retest โซน BreakBlock ตามเทรนด์
  โดยราคาอยู่ต่ำกว่า/เหนือกว่าเส้น Stoploss Retail = Phase C entry

## Trade Setup + Backtest (จุด Entry/SL/TP + ตารางสถิติ Win/Loss)

**กฎที่สังเคราะห์จากทุกไฟล์ + ข้อมูลเว็บ SMC** (ตรงกันหมด)
- **Entry** = ราคากลับมา retest โซน BreakBlock ตามเทรนด์ (proximal) — เลือกได้ว่าใช้
  ทุกสัญญาณ BB Retest หรือเฉพาะ Reaper (Phase C: ต่ำกว่า/เหนือ Stoploss Retail)
- **SL** = พ้นโซนฝั่ง distal + buffer (xATR) — ตรงกับ EP.5 "SL พ้นโซน refine",
  Day Trade "SL พ้น HTF POI/Strong Swing", เว็บ "stop beyond the block's range";
  โหมด "พ้นระดับ CHoCH" = เลือกจุดที่ไกลกว่าเพื่อความปลอดภัย (Strong Swing invalidation)
- **TP** 3 โหมด:
  - **Fixed RR** (ค่าเริ่มต้น 3R) — เว็บยืนยัน SMC ใช้ 1:2–1:3 ขึ้นไป
  - **สวิงล่าสุด** — Day Trade: "TP = Swing ล่าสุด" (สภาพคล่องถัดไป)
  - **Fibo TP1** — EP.5: 4.123 / W-M: 300% (ลากจาก extreme→CHoCH level)

**Trade Simulator (เดินย้อนหลัง)**
- เปิดเทรดได้ครั้งละ 1 ไม้ (ไม่ซ้อน) เพื่อสถิติที่อ่านง่าย
- ทุกแท่งหลังเข้า เช็คว่า high/low แตะ TP หรือ SL ก่อน — ถ้าแท่งเดียวแตะทั้งคู่
  นับเป็นแพ้ (conservative, ปรับได้)
- วาดเส้น Entry (เทา) / SL (แดง) / TP (เขียว) + label RR และผลลัพธ์ `✓ +xR` / `✗ -1R`

**การแสดงผล (อ้างอิงสไตล์จาก V9.9_NOPPON_SessionBreak):**
- จุด Entry = วงกลม `label.style_circle` เขียว(buy)/แดง(sell) + label โปร่งใส (กระจกดำ)
  ระบุทิศ + RR; จุด SL = วงกลมแดง; จุดออก = วงกลม + `✓ +xR`/`✗ −1R` + tick line
- **ธีม Luxe**: กรอบทอง (`frame_color` ทอง 42%), พื้นกระจกเข้ม, แถวสลับ (zebra),
  หัวตาราง ◆, แถบ winrate 10 ช่อง `▰▱`, ไฮไลต์แถวดีสุดด้วย ★ + พื้นทอง

**เก็บผลเทรดเป็น array** (`resIsWin`, `resR`, `resTime`, `resDir`) พร้อม timestamp
ของแท่งเข้า → คำนวณสถิติทุกตารางจาก array นี้บนแท่งสุดท้าย (แปลงเดือน/วันด้วย
`month()/dayofmonth()/year()` ตาม Timezone ที่ตั้ง — คีย์เดือน = `year*12+month`)

**3 ตาราง:**
1. **หลัก**: Trades, Win·Loss, Winrate + `▰▱`, Avg Win, Profit Factor, Net R,
   Long/Short W·L, Best/Worst Streak
2. **รายเดือน** (N เดือนย้อนหลัง): MONTH · W-L · WR% · NET R · STREAK + ★เดือนดีสุด + TOTAL
3. **รายวัน** (เดือนเป้าหมาย): วันที่ 1–31 · W-L · WR% · NET R + TOTAL (วันไม่มีเทรด = แถวจาง)

**ข้อจำกัดของ simulator** (บันทึกไว้ให้ชัด)
- เป็นการจำลองแบบ 1 ไม้/เวลา ไม่มีค่าคอมมิชชัน/spread/slippage (ต่างจาก `strategy()`)
- Entry ใช้ราคาปิดของแท่งสัญญาณ (จริง ๆ คอร์สเข้าที่ขอบโซน — ใกล้เคียง)
- แท่ง realtime ล่าสุดอาจ repaint จนกว่าจะปิด; แท่งย้อนหลัง deterministic
- ผลลัพธ์เป็นสถิติเชิงกลไกเพื่อประเมินระบบ ไม่ใช่ผลเทรดจริง

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
6. **ไหล่ของ Pattern W/M**: ใช้สวิงโลว์/ไฮ 2 ลูกล่าสุดก่อน CHoCH — ลูกที่ไม่ใช่ extreme
   คือไหล่; ช่วงวัด 50% = CHoCH-level ↔ extreme (คอร์สวัด High↔Low ของ pattern ซึ่ง
   ใกล้เคียงกัน)
7. **EQH/EQL**: เทียบเฉพาะ pivot ล่าสุดกับลูกก่อนหน้าด้วยค่าเผื่อ xATR (คอร์สใช้สายตา
   แยก "relative equal" กับ "ไม่เท่า") — ปรับค่าเผื่อได้ใน settings
8. **Trendline liquidity**: ยังไม่ implement (วาด trendline อัตโนมัติมีความกำกวมสูง)
9. **Stoploss Retail**: ตีความจากภาพ Reaper Model = ระดับ IDM/Valid Pullback ล่าสุด
   ที่ถูกเคลียร์ระหว่างเทรนด์ยังไม่เปลี่ยน (ภาพไม่ได้ระบุว่าเส้นหมดอายุเมื่อไร —
   เลือกให้หมดอายุเมื่อเกิด CHoCH หรือมี sweep ใหม่แทนที่)

## แผนต่อยอด (ยังไม่ทำ)

- เวอร์ชัน `strategy()` สำหรับ backtest ตาม Model Setup EP.5 (entry ที่ BB retest,
  SL พ้นโซน, TP1/TP2 ตาม Fibo)
- วาดโซน M15 อัตโนมัติขณะอยู่บนชาร์ต M5 ผ่าน `request.security`
- แยกความแรงของโซน (ทำลายโซนฝั่งตรงข้าม / เป็นแท่ง breakout เอง) ตาม TIP ของ EP.5
