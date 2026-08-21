# BreakBlock Market Structure Indicator [EP.1-5]

อินดิเคเตอร์ Pine Script v6 สำหรับ TradingView ที่ประกอบร่างจากคอร์ส
**THE POWER OF BREAKBLOCK (EP.01–EP.05)** + **Breakblock_S (Pattern W/M)** +
**BREAKBLOCK STRATEGY DAY TRADE (Liquidity/Range/POI)** + **REAPER MODEL** —
ครอบคลุมตั้งแต่โครงสร้างตลาดพื้นฐานจนถึง Real BreakBlock, Liquidity และ Fibo Target

> รายละเอียดว่าแต่ละฟีเจอร์มาจากกฎข้อไหนของคอร์ส ดูที่ [docs/PLAN.md](docs/PLAN.md)

## ติดตั้ง

1. เปิด TradingView → Pine Editor
2. คัดลอกโค้ดทั้งหมดจาก [`BreakBlock_Indicator.pine`](BreakBlock_Indicator.pine) วางลงไป
3. กด **Add to chart** แล้ว **Save**

## ฟีเจอร์

| ฟีเจอร์ | ที่มา | การแสดงผล |
|---|---|---|
| Label โครงสร้าง HH / HL / LH / LL | EP.1 | label ที่จุดสวิง (ตามเทรนด์ = ใหญ่, Internal = เล็กสีเทา) |
| Valid Pullback swing engine | EP.2–3 | ตรวจสวิงตาม checklist 3 ขั้น (รวมไส้) |
| เส้น IDM (Inducement) | EP.2 | เส้นจุดประสีส้ม ย้ายตามสวิงล่าสุด, `IDM ✕` เมื่อโดน sweep |
| เส้น BOS | EP.3 | เส้นที่จุดเบรกโครงสร้างต่อเทรนด์ |
| CHoCH ที่แท้จริง | EP.3 | เส้นประเฝ้าระวังที่จุดกำเนิด Strong Swing → เส้นทึบ + label เมื่อปิดแท่งทะลุ |
| MSS + POI Zone | EP.4 | เส้น MSS + กล่องโซนพร้อมเส้นกลาง 50% |
| SR Shadow | EP.4 | แถบสีเทา (gap แบบ line chart) — โซน S/R flip |
| BreakBlock Zone | EP.4–5 | กล่องโซน `BB` (ทับซ้อน SR Shadow = `BB ★`) |
| Real BreakBlock filter | EP.5 | ตัดโซนที่ไม่มี FVG ทิ้ง (ปิดได้) |
| สัญญาณ CHoCH Retest | EP.5 | วงกลมเมื่อราคากลับเข้าโซนตามเทรนด์ + alert |
| Fibo Target (เลือกชุดได้) | EP.5 / Breakblock_S | `4.123 / 6` หรือ `300 / 400 / 600%` ลากอัตโนมัติหลัง CHoCH |
| Pattern Reversal W / M | Breakblock_S | ตรวจฟิลเตอร์ไหล่ 50% ที่จังหวะ CHoCH → label `W ✓` / `M ✓` + alert |
| Re-Accumulation / Re-Distribution | Breakblock_S | label เมื่อย่อหลอกแตะ neckline แล้ว BOS ต่อ |
| Liquidity BSL / SSL | Day Trade | เส้นจุดประจากสวิง 3 แท่ง จนโดน sweep (`BSL ✕` / `SSL ✕`) |
| EQH / EQL | Day Trade | label `$$$` เมื่อสวิงเท่ากัน/เกือบเท่ากัน (ค่าเผื่อ xATR) |
| Overlap Counter ×N | Day Trade | โซน BB ไม่ลบเมื่อถูกทะลุ — นับ ×N (≥2 = BREAKBLOCK แท้) |
| Strong / Weak High-Low (ERL) | Day Trade | label จุดที่ควรยัน + เป้าหมายของ range หลัง CHoCH |
| Reaper Model | ภาพประกอบคอร์ส | เส้น Stoploss Retail + สัญญาณเพชร Reaper Buy/Sell (Phase C) |
| จุด Entry / SL / TP อัตโนมัติ | ทุกไฟล์ + SMC | วาดเส้น Entry/SL/TP + RR เมื่อมีสัญญาณเข้า |
| Trade Simulator + สถิติ Win/Loss | Backtest | เดินย้อนหลัง เช็ค TP/SL โดนก่อน → ตารางสถิติ |
| Dashboard | EP.5 workflow | Trend / IDM / CHoCH / HTF bias (H1, H4) / จำนวนโซน / SL Retail |

## วิธีใช้ตาม Model Setup ของคอร์ส (EP.5)

1. **โครงสร้าง — H1**: เปิดชาร์ต H1 ดูให้เกิดลำดับ สร้างสวิง → เคลียร์ IDM → **CHoCH**
   (หรือดู HTF bias บน dashboard ขณะอยู่ TF เล็ก)
2. **หาโซน — M15**: เปิดชาร์ต M15 ดูโซน `BB` ที่ทิศตรงกับ CHoCH ของ H1
   — โซน `BB ★` (ทับซ้อน SR Shadow) น่าสนใจกว่า
3. **Refine — M5/M3/M1**: ลง TF เล็ก หาโซน `BB` ที่ซ้อนอยู่ในกรอบโซน M15
4. **เข้าเทรด**: รอสัญญาณ retest (วงกลม/alert) — SL พ้นโซน refine, โครงสร้างพัง = พ้น extreme
5. **TP**: ใช้เส้น Fibo ที่ลากให้อัตโนมัติ — **TP1 = 4.123**, **TP2 = 6**

## วิธีใช้ Reaper Model (จากภาพประกอบคอร์ส)

1. **Phase A**: รอ BOS ตามเทรนด์ (เส้น BOS ของอินดิเคเตอร์)
2. **Phase B**: เมื่อ Valid Pullback ล่าสุดโดนเคลียร์ อินดิเคเตอร์จะวาดเส้นแดง
   **Stoploss Retail** ที่ระดับนั้น (= จุดที่ SL รายย่อยกองอยู่)
3. **Phase C**: เมื่อราคาลงต่ำกว่า (buy) / ขึ้นเหนือ (sell) เส้นนี้และแตะโซน BreakBlock
   ตามเทรนด์ → สัญญาณ **เพชร Reaper Buy/Sell** + alert

## Entry / SL / TP + ตารางสถิติ Win/Loss (Backtest)

เปิดใช้ที่กลุ่ม **Trade Setup / Backtest** — เมื่อมีสัญญาณเข้า อินดิเคเตอร์จะวาด
เส้น Entry (เทา) / SL (แดง) / TP (เขียว) พร้อม RR และเดินจำลองย้อนหลังว่าไม้นั้น
ชน TP หรือ SL ก่อน แล้วสะสมลง **ตารางสถิติมุมล่างซ้าย**

**กฎที่ใช้ (สังเคราะห์จากทุกไฟล์ + SMC มาตรฐาน):**
- **Entry**: retest โซน BreakBlock ตามเทรนด์ (เลือก "ทุกสัญญาณ" หรือ "Reaper เท่านั้น")
- **SL**: พ้นโซนฝั่งตรงข้าม + buffer (xATR) หรือเลือก "พ้นระดับ CHoCH" เพื่อความปลอดภัย
- **TP**: `Fixed RR` (เริ่มต้น 3R) / `สวิงล่าสุด` (Day Trade) / `Fibo TP1` (EP.5 4.123 หรือ W-M 300%)

**ตารางสถิติแสดง:** จำนวนเทรด, Win/Loss, Win Rate %, Avg Win (R), Profit Factor,
Net R, แยก Long/Short, และ Best Win / Worst Loss Streak

> ⚠️ เป็น simulator เชิงกลไก (1 ไม้/เวลา, ไม่คิด spread/คอมมิชชัน, แท่ง realtime อาจ
> repaint) — ใช้ประเมินระบบคร่าว ๆ ไม่ใช่ผลเทรดจริง; ต้องการ backtest เต็มรูปให้ใช้
> เวอร์ชัน `strategy()` (แผนต่อยอด)

## Alerts ที่มีให้

`BOS Up/Down` · `MSS Bullish/Bearish` · `CHoCH Up/Down` · `IDM Swept` ·
`BB Retest Buy/Sell` · `Reaper Buy/Sell` · `Pattern W/M` · `Trade Win/Loss` —
ตั้งผ่านเมนู Alert ของ TradingView ได้ทันที

## การตั้งค่าสำคัญ

- **Swing break ใช้ราคาปิด** — ค่าเริ่มต้นปิด (ใช้ไส้ตาม EP.3); เปิดถ้าต้องการเข้มแบบ EP.2
- **ยืนยัน MSS/CHoCH เมื่อปิดแท่ง** — เปิดเป็นค่าเริ่มต้น กันสัญญาณกระพริบระหว่างแท่ง
- **Real BreakBlock: ต้องมี FVG** — เปิดเป็นค่าเริ่มต้นตาม EP.5
- **Overlap สูงสุด / อายุโซน** — โซน BB ทนการทะลุและนับ ×N ลบเมื่อเกินลิมิต
- **ชุดเป้าหมาย Fibo** — `4.123 / 6 (EP.5)` หรือ `300 / 400 / 600% (W/M)`
- **ค่าเผื่อ EQH/EQL (xATR14)** — ความหลวมของการนับว่า "เท่ากัน"
- **HTF Bias 1/2** — ค่าเริ่มต้น 60 (H1) และ 240 (H4)

## ข้อจำกัด

- สัญญาณบนแท่ง realtime อาจเปลี่ยนได้จนกว่าแท่งจะปิด (ลดปัญหาด้วยตัวเลือกยืนยันปิดแท่ง)
- HTF bias ใช้ `request.security` ค่าอาจอัปเดตเมื่อแท่ง HTF ปิด
- อินดิเคเตอร์นี้เป็นเครื่องมือช่วยวิเคราะห์ตามแนวทางของคอร์สเท่านั้น ไม่ใช่คำแนะนำการลงทุน
