<p align="center">
  <img src="test/test.gif"/>
</p>

# กลยุทธ์ TFEX (POC Volume Profile) [Long Only]

Repository นี้มี Pine Script กลยุทธ์สำหรับการเทรดผลิตภัณฑ์ TFEX โดยใช้ **POC (Point of Control) Volume Profile** และ **EMA200** กลยุทธ์นี้เน้นเฉพาะการเปิดสถานะ Long (ซื้อ) และมีการตั้งกฎแบบไดนามิกสำหรับการเข้า (Entry) การทำกำไร (Take Profit) และการหยุดขาดทุน (Stop Loss) โดยออกแบบให้ทำงานบน **กรอบเวลารายสัปดาห์ (Weekly Timeframe)**

---
## อธิบายcode [YouTube](https://youtu.be/U3OZ6ZTb7V0)
## แนวคิดของกลยุทธ์
### เงื่อนไขการเข้า:
1. **POC รายสัปดาห์เพิ่มขึ้น:**
   - ค่า POC (Point of Control) ของสัปดาห์ปัจจุบันต้องสูงกว่าสัปดาห์ก่อนหน้า
2. **ราคายืนเหนือ EMA200:**
   - ราคาปัจจุบันต้องสูงกว่าเส้นค่าเฉลี่ย EMA 200
3. **การเปิดสถานะ (Entry):**
   - เมื่อทั้งสองเงื่อนไขข้างต้นเป็นจริง ให้เปิดสถานะ Long จำนวน 2 สัญญา

### เงื่อนไขการออก:
1. **ทำกำไร (Take Profit):**
   - **TP1:** ปิดสถานะ 1 สัญญาเมื่อราคาสูงกว่าราคาเข้า 10 จุด
   - **TP2:** รันสถานะที่เหลือ 1 สัญญาต่อไปจนกว่าราคาจะปิดต่ำกว่า EMA200
2. **หยุดขาดทุน (Stop Loss):**
   - ปิดสถานะทั้งหมดหากราคาต่ำกว่า EMA200

---

## อินดิเคเตอร์หลัก

1. **POC (Point of Control):**
   - ระดับราคาที่มีปริมาณการซื้อขายสูงที่สุดในแต่ละสัปดาห์
   - ใช้เพื่อยืนยันแนวโน้ม
2. **EMA200 (Exponential Moving Average):**
   - ใช้เป็นแนวรับ/แนวต้านแบบไดนามิก
   - ช่วยให้การเทรดเป็นไปตามแนวโน้มใหญ่

---

## การคำนวณ POC

POC คำนวณจาก Volume Profile ของแท่งเทียนรายสัปดาห์:
- ระดับราคาที่มีปริมาณการซื้อขายสะสมสูงสุดจะถูกกำหนดเป็น POC
- กลยุทธ์จะเปรียบเทียบ POC ของสัปดาห์ปัจจุบันกับสัปดาห์ก่อนหน้าเพื่อดูว่ามีการเพิ่มขึ้นหรือไม่

---

## การใช้งาน Pine Script

กลยุทธ์นี้เขียนขึ้นโดยใช้ Pine Script (เวอร์ชัน 5) บนแพลตฟอร์ม TradingView โดยใช้:
- **Volume Profile:** สำหรับการคำนวณ POC
- **EMA Indicator:** สำหรับวิเคราะห์แนวโน้ม
- **`strategy.entry` และ `strategy.exit`**: สำหรับจัดการคำสั่งซื้อขาย

---

## อธิบาย code  Pine Script


```python
//@version=6
strategy("TFEX POC + EMA200 [Long Only]", overlay=true)
```
- @version=5: ระบุว่าใช้ Pine Script เวอร์ชัน 6
- strategy(): ตั้งชื่อกลยุทธ์ว่า "TFEX POC + EMA200 [Long Only]" และกำหนดให้แสดงผลบนกราฟ (overlay=true)

### 1. กำหนดพารามิเตอร์ (Parameters)
```python
bins = input.int(50, title="Number of Bins", minval=10)
timeframe = input.timeframe("W", title="POC Timeframe")
emaLength = input.int(200, title="EMA Length", minval=1)

tpPoints = input.int(10, title="Take Profit (Points)")
contracts = input.int(2, title="Number of Contracts", minval=1)
```
- bins: จำนวนบล็อก (Bins) สำหรับคำนวณ Volume Profile (ค่าเริ่มต้น: 50)
- timeframe: กำหนดกรอบเวลา (Timeframe) สำหรับคำนวณ POC (ค่าเริ่มต้น: รายสัปดาห์)
- emaLength: ระยะเวลาของ EMA (ค่าเริ่มต้น: 200)
- tpPoints: ระบุจุดทำกำไร TP1 (ค่าเริ่มต้น: 10 จุด)
- contracts: จำนวนสัญญาที่เปิดเมื่อเข้าเทรด (ค่าเริ่มต้น: 2

### 2. การคำนวณ EMA200
```python
ema200 = ta.ema(close, emaLength)
```
- ใช้ฟังก์ชัน ta.ema เพื่อคำนวณค่า EMA200 จากราคาปิด (Close)

## 3. คำนวณ POC (Point of Control)


## 1. แนวคิดพื้นฐาน
- POC คือระดับราคาที่มี Volume มากที่สุด หรือมีปริมาณการซื้อขายสะสมสูงที่สุดในช่วงเวลาที่กำหนด (เช่น 1 วัน, 1 สัปดาห์, หรือ 1 เดือน)
## 2. ขั้นตอนการคำนวณ
   * 1.กำหนดช่วงเวลา (Time Period)
     * เลือกช่วงเวลาที่ต้องการวิเคราะห์ เช่น สัปดาห์ (Weekly)
     * ดึงข้อมูลราคาสูงสุด (High), ราคาต่ำสุด (Low), และปริมาณการซื้อขาย (Volume)
   * 2 แบ่งราคาออกเป็นช่วง (Bins)
      * ช่วงราคาจะถูกแบ่งเป็น Bin หรือ Price Levels (เช่น 1 จุด หรือ 0.5 จุด ต่อ Bin)
      * ตัวอย่าง: หากราคาช่วง Low-High อยู่ระหว่าง 1000 - 1020 และ Bin Size = 1 จุด
      * Bins = [1000, 1001, 1002, …, 1020]
   * 3 สะสม Volume ในแต่ละระดับราคา
      * ตรวจสอบว่าปริมาณการซื้อขาย (Volume) ในแต่ละ Bin มีค่าเท่าไหร่
      * ตัวอย่าง:
        * ระดับราคา 1000 มี Volume 150
        * ระดับราคา 1001 มี Volume 200
        * ระดับราคา 1002 มี Volume 250
        * ระดับราคา 1003 มี Volume 300
   * 4 ระบุระดับราคาที่มี Volume มากที่สุด (POC)
        * POC = ระดับราคาที่มี Volume สะสมมากที่สุด
        * ตัวอย่าง:
          * ระดับราคา 1003 มี Volume 300 (สูงสุด)
          * ดังนั้น POC = 1003

```python
var float poc = na
var float[] volumeBins = array.new_float(bins, 0)

highTF = request.security(syminfo.tickerid, timeframe, high)
lowTF = request.security(syminfo.tickerid, timeframe, low)
priceRange = highTF - lowTF
binSize = priceRange / bins
```
- ใช้ request.security ดึงข้อมูล High และ Low จากกรอบเวลารายสัปดาห์
- แบ่งราคาออกเป็นช่วงย่อย (binSize) โดยใช้จำนวน Bins ที่ระบุ

```python
if (ta.change(time(timeframe)))
    array.fill(volumeBins, 0)

currentBin = math.floor((close - lowTF) / binSize)
if currentBin >= 0 and currentBin < bins
    array.set(volumeBins, currentBin, array.get(volumeBins, currentBin) + volume)
```
- รีเซ็ตข้อมูล Volume Profile (volumeBins) เมื่อเริ่มกรอบเวลาใหม่
- เพิ่มปริมาณการซื้อขาย (volume) ในแต่ละช่วงราคาที่สัมพันธ์กับ Bin

```python
var float maxVolume = 0
var int maxIndex = 0
for i = 0 to bins - 1
    binVolume = array.get(volumeBins, i)
    if binVolume > maxVolume
        maxVolume := binVolume
        maxIndex := i

poc := lowTF + maxIndex * binSize
```
- ค้นหา Bin ที่มี Volume สูงสุดและคำนวณ POC (ระดับราคาที่มี Volume สูงที่สุด)

### 4. เงื่อนไขการเข้า (Entry Conditions)
```python
previousPoc = request.security(syminfo.tickerid, timeframe, poc[1])
longCondition = poc > previousPoc and close > ema200

if (longCondition and strategy.position_size == 0)
    strategy.entry("Long", strategy.long, contracts)
    entryPrice := na
    tp1Hit := false
```
- ตรวจสอบว่า POC สัปดาห์ปัจจุบันสูงกว่าสัปดาห์ก่อนหน้า และราคาปัจจุบันสูงกว่า EMA200
- เมื่อเข้าเงื่อนไข ให้เปิดสถานะ Long จำนวน 2 สัญญา

### 5. เงื่อนไขการทำกำไรและปิดสถานะ (Take Profit & Exit)
```python
if (strategy.position_size > 0 and close >= tp1Price and not tp1Hit)
    strategy.exit("TP1", from_entry="Long", qty_percent=50, limit=tp1Price)
    tp1Hit := true

if (strategy.position_size > 0 and close < ema200)
    strategy.close("Long")
```
- TP1: ปิดสถานะ 50% เมื่อราคาถึงระดับกำไรที่ตั้งไว้ (tp1Price)
- Stop Loss: ปิดสถานะทั้งหมดเมื่อราคาต่ำกว่า EMA200


###  6.การแสดงผลอินดิเคเตอร์
```python
plot(ema200, color=color.blue, linewidth=2, title="EMA 200")
plot(poc, title="POC Line", color=color.red, style=plot.style_line, linewidth=2)
plot(entryPrice, color=color.yellow, style=plot.style_line, linewidth=1, title="Entry Price")
plot(tp1Price, color=color.green, style=plot.style_line, linewidth=1, title="TP1 Level")
```
- แสดงเส้น EMA200, POC, ราคาที่เข้าเทรด (entryPrice) และระดับ TP1 (tp1Price) บนกราฟ

### 7. แสดงค่า POC เป็นตัวเลข
```python
if (bar_index % 10 == 0)  
    label.new( x=bar_index,y=poc, text=str.tostring(poc, "#.##"),color=color.red,style=label.style_circle,textcolor=color.black)

var table pocTable = table.new(position.top_right, 1, 1, border_width=1, border_color=color.red)
if not na(poc)
    table.cell(pocTable, 0, 0, str.tostring(poc, "#.##"), text_color=color.red, bgcolor=color.white)
```
- แสดงค่า POC เป็นตัวเลขบนกราฟทุกๆ 10 แท่ง
- สร้างตารางแสดงค่า POC มุมบนขวาของหน้าจอ
---

### 8. ผลสรุป backtest ของ สินทรัพย์ต่างๆ 
- s50 set50 index future
<p align="center">
  <img src="รูป backtest/s 50.png"/>
</p>

- go gold online future
<p align="center">
  <img src="รูป backtest/go .png"/>
</p>

- usd us dollar future
<p align="center">
  <img src="รูป backtest/usd .png"/>
</p>

- cp all public company limited future
<p align="center">
  <img src="รูป backtest/cpall.png"/>
</p>




---


## การติดตั้งและการใช้งาน

1. เปิด [TradingView](https://www.tradingview.com/)
2. สร้างไฟล์ Pine Script ใหม่ใน Editor
3. คัดลอกและวางโค้ด Pine Script ที่ให้ไว้ลงใน Editor
4. บันทึกและเพิ่มกลยุทธ์ลงในกราฟของคุณ
5. ตรวจสอบให้แน่ใจว่ากราฟถูกตั้งเป็น **กรอบเวลารายสัปดาห์ (Weekly Timeframe)** เพื่อผลลัพธ์ที่ดีที่สุด

---

## ข้อจำกัดของกลยุทธ์

- **เฉพาะสถานะ Long เท่านั้น:**
  - กลยุทธ์นี้ไม่ได้คำนึงถึงโอกาสในการ Short
- **ขึ้นอยู่กับกรอบเวลารายสัปดาห์:**
  - ทำงานได้ดีที่สุดในกรอบเวลารายสัปดาห์ และอาจไม่เหมาะสมกับกรอบเวลาที่ต่ำกว่า
- **ความแม่นยำของการคำนวณ POC:**
  - ขึ้นอยู่กับข้อมูล Volume ที่มีอยู่

---

## การปรับแต่ง

พารามิเตอร์ต่อไปนี้สามารถปรับเปลี่ยนได้ในสคริปต์:
1. **ระดับ TP1:** ค่าเริ่มต้นตั้งไว้ที่ 10 จุด สามารถปรับเปลี่ยนเพื่อเป้าหมายกำไรที่แตกต่าง
2. **ระยะเวลา EMA:** ค่าเริ่มต้นตั้งไว้ที่ 200 สามารถปรับเพื่อความไวของแนวโน้ม
3. **ขนาดสัญญา:** ปรับจำนวนสัญญาที่เปิดได้ตามความเหมาะสม

---

## การมีส่วนร่วม

สามารถ Fork Repository นี้และส่ง Pull Request เพื่อปรับปรุงหรือแก้ไขข้อบกพร่องได้ ยินดีต้อนรับทุกข้อเสนอแนะ!

---

## ข้อสงวนสิทธิ์

กลยุทธ์นี้จัดทำขึ้นเพื่อวัตถุประสงค์ทางการศึกษาเท่านั้น การเทรดมีความเสี่ยงสูง และผลการดำเนินงานในอดีตไม่ได้บ่งชี้ถึงผลลัพธ์ในอนาคต ใช้กลยุทธ์นี้ด้วยความรอบคอบ
