# BreakBlock Market Structure Indicator [EP.1-5]

อินดิเคเตอร์ Pine Script v6 สำหรับ TradingView ที่ประกอบร่างจากคอร์ส
**THE POWER OF BREAKBLOCK (EP.01–EP.05)** — ครอบคลุมตั้งแต่โครงสร้างตลาดพื้นฐาน
จนถึง Real BreakBlock + Fibo Target

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
| Fibo Target 4.123 / 6 | EP.5 | ลากอัตโนมัติหลัง CHoCH (A = จุดที่ถูกเบรก, B = extreme) |
| Dashboard | EP.5 workflow | Trend / IDM / ระดับ CHoCH / HTF bias (H1, H4) / จำนวนโซน |

## วิธีใช้ตาม Model Setup ของคอร์ส (EP.5)

1. **โครงสร้าง — H1**: เปิดชาร์ต H1 ดูให้เกิดลำดับ สร้างสวิง → เคลียร์ IDM → **CHoCH**
   (หรือดู HTF bias บน dashboard ขณะอยู่ TF เล็ก)
2. **หาโซน — M15**: เปิดชาร์ต M15 ดูโซน `BB` ที่ทิศตรงกับ CHoCH ของ H1
   — โซน `BB ★` (ทับซ้อน SR Shadow) น่าสนใจกว่า
3. **Refine — M5/M3/M1**: ลง TF เล็ก หาโซน `BB` ที่ซ้อนอยู่ในกรอบโซน M15
4. **เข้าเทรด**: รอสัญญาณ retest (วงกลม/alert) — SL พ้นโซน refine, โครงสร้างพัง = พ้น extreme
5. **TP**: ใช้เส้น Fibo ที่ลากให้อัตโนมัติ — **TP1 = 4.123**, **TP2 = 6**

## Alerts ที่มีให้

`BOS Up/Down` · `MSS Bullish/Bearish` · `CHoCH Up/Down` · `IDM Swept` ·
`BB Retest Buy/Sell` — ตั้งผ่านเมนู Alert ของ TradingView ได้ทันที

## การตั้งค่าสำคัญ

- **Swing break ใช้ราคาปิด** — ค่าเริ่มต้นปิด (ใช้ไส้ตาม EP.3); เปิดถ้าต้องการเข้มแบบ EP.2
- **ยืนยัน MSS/CHoCH เมื่อปิดแท่ง** — เปิดเป็นค่าเริ่มต้น กันสัญญาณกระพริบระหว่างแท่ง
- **Real BreakBlock: ต้องมี FVG** — เปิดเป็นค่าเริ่มต้นตาม EP.5
- **HTF Bias 1/2** — ค่าเริ่มต้น 60 (H1) และ 240 (H4)

## ข้อจำกัด

- สัญญาณบนแท่ง realtime อาจเปลี่ยนได้จนกว่าแท่งจะปิด (ลดปัญหาด้วยตัวเลือกยืนยันปิดแท่ง)
- HTF bias ใช้ `request.security` ค่าอาจอัปเดตเมื่อแท่ง HTF ปิด
- อินดิเคเตอร์นี้เป็นเครื่องมือช่วยวิเคราะห์ตามแนวทางของคอร์สเท่านั้น ไม่ใช่คำแนะนำการลงทุน
