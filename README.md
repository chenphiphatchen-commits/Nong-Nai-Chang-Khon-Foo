[Readme Nai Chang Khon Foo.md](https://github.com/user-attachments/files/27569823/Readme.Nai.Chang.Khon.Foo.md)
# 👷‍♂️ NONG NAI CHANG KHON FOO — LINE + n8n + Gemini + Sheets Construction Bot

บอท **“นายช่างขนฟู”** สำหรับใช้ในงานบริหารโครงการก่อสร้าง (Automated Construction Management System) เพื่อให้โฟร์แมนรายงานความคืบหน้าจากหน้าไซต์งานผ่าน LINE โดยมี AI ช่วยอ่านข้อความ สกัดข้อมูล และอัปเดตลงตาราง Master Plan (Google Sheets) อัตโนมัติ พร้อมวิเคราะห์ความล่าช้าและส่งอีเมลแจ้ง Project Engineer ทันที

ตัวอย่างข้อความที่รองรับ:
```text
สวัสดีครับ (พิมพ์ทักทายทั่วไป)
งานเสาเข็มและฐานราก ความคืบหน้า 50% วันที่ 1 พ.ค. 2026
งานเสาเข็มและฐานราก เสร็จแล้ว 100% วันที่ 7 พ.ค. 2026
```

---

## 1. ระบบนี้ทำอะไรได้บ้าง
* **รับข้อความจาก LINE ผ่าน Webhook:** นำเข้าสู่แพลตฟอร์ม n8n (Workflow Orchestrator)
* **ใช้ Gemini AI สกัดข้อมูล:** แยกข้อมูลจากภาษาพูดทั่วไป ได้แก่ `ชื่องาน`, `เปอร์เซ็นต์ความคืบหน้า`, และ `วันที่`
* **ตรวจสอบข้อมูล (Data Validation):** หากโฟร์แมนพิมพ์ข้อมูลมาไม่ครบ (เช่น ขาดวันที่) ระบบจะตีกลับข้อความทาง LINE ทันทีเพื่อขอข้อมูลเพิ่ม
* **บันทึกงานลง Google Sheets:** อัปเดตตาราง Master Plan โดยอัตโนมัติ
* **วิเคราะห์ความล่าช้า (Gap Analysis):** ประเมินข้อมูลเทียบกับแผนงานหลัก (CPM) หากงานล่าช้ากว่ากำหนด ระบบจะวิเคราะห์ผลกระทบต่องานถัดไป (Predecessor)
* **ส่งอีเมลแจ้งเตือนผู้บริหาร (Gmail API):** ส่งสรุปรายงานประจำวันให้ Project Engineer พร้อมเสนอแนะแนวทางแก้ไขเบื้องต้น (เช่น การเพิ่มเวลาโอที)

---

## 2. สิ่งที่ผู้ใช้ต้องเตรียมเอง
ผู้ใช้ต้องมีบัญชี/ข้อมูลจาก 4 ระบบหลักนี้:

### 2.1 LINE Developers
ต้องเตรียม:
```text
LINE_CHANNEL_ACCESS_TOKEN
```
เอามาจาก LINE Developers > Messaging API
สิ่งที่ต้องเปิด/ตั้งค่าใน LINE Developers:
```text
Use webhook: Enabled
Webhook URL: https://chenphiphat.app.n8n.cloud/webhook/construction (หรือ URL ของ n8n)
```
*บอทที่ใช้ทดสอบสำหรับโปรเจกต์นี้คือ Bot ID: `@219gilhh`*

---

### 2.2 n8n Platform (Workflow Orchestrator)
ใช้เป็นตัวกลางแทนการเขียนโค้ด (Low-code Automation)
ต้องเตรียม Node การทำงานดังนี้:
```text
1. Webhook Node: รับ HTTP POST requests
2. Edit Fields Node: จัดรูปแบบ JSON ให้แบนราบก่อนส่งเข้า AI
3. AI Agent Node: ประมวลผลภาษา
4. IF Node: สร้าง Validation Logic ตรวจสอบข้อมูล
5. Google Sheets Node: อ่าน/เขียน Master Plan
6. Gmail Node: ส่งอีเมลรายงาน
7. LINE Messaging Node: ตอบกลับข้อความ
```

---

### 2.3 Gemini API
ต้องเตรียม:
```text
GEMINI_API_KEY
```
เอามาจาก Google AI Studio
ในระบบ n8n ตั้งค่า Model ไว้ที่:
```text
models/gemini-2.5-flash-preview
```
*อ้างอิงจากการตั้งค่าใน AI Agent Node*

---

### 2.4 Google Workspace (Sheets & Gmail)
ต้องเตรียม:
```text
Google OAuth2 Credentials
```
สิ่งที่ต้องทำ:
1. เปิดใช้งาน Google Sheets API และ Gmail API ใน Google Cloud Console
2. เตรียม Database บน Google Sheets ให้มีคอลัมน์ (Column) ตามที่กำหนด

โครงสร้างคอลัมน์ที่จำเป็น (Master Plan):
| Column Name | Type | จำเป็นไหม |
| :--- | :--- | :--- |
| ชื่องาน (Task Name) | Text | จำเป็น |
| งานที่ต้องเสร็จก่อน (Predecessor) | Text | จำเป็นสำหรับการคำนวณ CPM |
| วันเริ่ม (Start) | Date | จำเป็น |
| วันเสร็จ (End) | Date | จำเป็น |
| สถานะ (Status) | Text | จำเป็น |
| ความคืบหน้า (%) | Number/Percentage | จำเป็น |

---

## 3. การตั้งค่า Credentials ใน n8n
ไปที่เมนู `Credentials` ใน n8n และทำการ Add Credentials ดังนี้:
1. **LINE Messaging API:** ใส่ `Channel Access Token`
2. **Google Gemini API:** ใส่ `API Key`
3. **Google Sheets OAuth2 API:** ทำการ Connect กับบัญชี Google
4. **Gmail OAuth2 API:** ทำการ Connect กับบัญชี Google ที่ต้องการใช้ส่งอีเมล

*ข้อสำคัญ: ห้ามแชร์ไฟล์ Export ของ n8n (JSON) ที่มี Credentials ฝังอยู่เด็ดขาด*

---

## 4. ขั้นตอนติดตั้งตั้งแต่ต้น
**Step 1: สร้าง Workflow**
สามารถ Import ไฟล์ JSON ของ Workflow เข้าสู่ n8n Canvas ได้เลย

**Step 2: ตั้งค่า Webhook URL**
เข้าไปที่ Webhook Node กดเปิดใช้งาน `Production URL` แล้ว Copy URL ไปใส่ใน LINE Developers > Webhook URL

**Step 3: ตั้งค่า Custom Tools ของ AI**
ตรวจสอบในส่วนของ Tools ที่เชื่อมกับ AI Agent:
* `check_master_plan`: สำหรับดึงข้อมูลตารางงานปัจจุบันมาวิเคราะห์
* `update_master_plan`: สำหรับสั่งการให้ AI บันทึกข้อมูลลงตาราง

**Step 4: ทดสอบการทำงาน (Activate)**
กดปุ่ม `Active` มุมขวาบนของ n8n เพื่อเริ่มใช้งานระบบ

---

## 5. รูปแบบสถานการณ์จำลอง (Testing Scenarios)
ระบบถูกออกแบบให้รองรับ 4 สถานการณ์หลัก:

**`ทักทายทั่วไป (System Initialization)`**
* **พิมพ์:** "สวัสดีครับ ขอรายงานความคืบหน้าครับ"
* **บอทตอบ:** รับทราบว่าเป็นคำทักทาย และจะไกด์ให้พิมพ์รายละเอียดงาน

**`งานปกติ / ไวกว่ากำหนด (Nominal Reporting)`**
* **พิมพ์:** "งานเสาเข็มและฐานราก ความคืบหน้า 50% วันที่ 1 พ.ค. 2026"
* **บอทตอบ:** อัปเดตลง Sheets เป็น `In Progress (50%)` และส่งอีเมลแจ้ง Project Engineer ว่างานเร็วกว่ากำหนด

**`งานล่าช้า (Delayed Reporting - Critical Path Impact)`**
* **พิมพ์:** "งานเสาเข็มและฐานราก 100% วันที่ 7 พ.ค. 2026" (กำหนดเดิมคือ 5 พ.ค.)
* **บอทตอบ:** AI ตรวจพบความล่าช้า 2 วัน อัปเดตสถานะเป็น `Done (100%)` และส่งอีเมลแจ้ง PE ถึงผลกระทบต่อ "งานคานคอดิน" พร้อมเสนอ Action Plan (เช่น การอนุมัติ Overtime)

**`ข้อมูลไม่ครบ (Validation Trigger)`**
* **พิมพ์:** ส่งแค่เปอร์เซ็นต์โดยไม่มีชื่องาน หรือไม่มีวันที่
* **บอทตอบ:** IF Node จะดักจับข้อผิดพลาด และตีกลับข้อความทาง LINE ให้โฟร์แมนระบุข้อมูลที่ขาดหายไปทันที

---

## 6. จุดที่ผู้ใช้แก้เพิ่มได้เอง
**6.1 เปลี่ยนเป้าหมายการส่งอีเมล**
เข้าไปที่ `Gmail Node` -> เปลี่ยนพารามิเตอร์ตรงช่อง `To:` เป็นอีเมลของ Project Engineer ที่ต้องการ (ค่าเริ่มต้นถูกตั้งเป็น `chenphiphat.chen@gmail.com`)

**6.2 ปรับแต่ง Prompt ของ AI**
เข้าไปที่ `AI Agent Node` เพื่อแก้คำสั่ง (System Message) หากต้องการให้ AI ตอบกลับด้วยน้ำเสียงที่ต่างไป หรือปรับเกณฑ์การวิเคราะห์ Gap Analysis

**6.3 แก้ไขรูปแบบ Payload ของ LINE**
เข้าไปที่ `Edit Fields Node` หากมีการอัปเดต API จากฝั่ง LINE สามารถปรับการ Map `JSON Body` ใหม่ที่นี่ได้เลย (เช่น การเรียก `events[0].message.text`)

---

## 7. Troubleshooting (ปัญหาที่พบบ่อย)
**LINE ไม่ตอบสนอง (Webhook ไม่ทำงาน)**
* สาเหตุที่พบบ่อย: ยังไม่ได้กด Active Workflow ใน n8n หรือ Webhook URL ใน LINE ไม่ตรงกับ Production URL ของ n8n

**AI สกัดข้อมูลผิดพลาด / ทำงานไม่ได้**
* ตรวจสอบว่าโควต้าการใช้งาน Gemini API เต็มหรือไม่
* เช็คที่ Edit Fields Node ว่ามีการลดรูป JSON ถูกต้องก่อนส่งเข้า AI Agent หรือไม่

**อีเมลส่งไม่ออก**
* ตรวจสอบ OAuth2 Token ของ Gmail ว่าหมดอายุหรือไม่ หรือสิทธิ์ (Scopes) ในการส่งอีเมลไม่เพียงพอ

---

## 8. Flow การทำงานของระบบ (Workflow Pipeline)
```text
LINE Message (โฟร์แมนส่งข้อมูล)
   ↓
Webhook (รับ Payload)
   ↓
Edit Fields (คลีนข้อมูล JSON)
   ↓
AI Agent (Gemini วิเคราะห์ Entity)
   ↓
IF Function (ตรวจสอบความครบถ้วน)
   ├── ข้อมูลไม่ครบ → LINE Messaging (ตีกลับถามโฟร์แมน)
   └── ข้อมูลครบ   → โหลด Custom Tools
          ↓
Google Sheets (อ่านแผน CPM / อัปเดต Master Plan)
          ↓
Gmail API (วิเคราะห์ Action Plan และส่งรายงาน)
```

---

## 9. สิ่งที่ควรพัฒนาต่อ (Future Development)
* เพิ่มการแจ้งเตือนเชิงรุก (Proactive Alerts) ก่อนถึงกำหนดส่งงาน
* สร้าง Dashboard แสดงผลข้อมูลภาพรวมของไซต์งาน
* รองรับการวิเคราะห์รูปภาพความคืบหน้าหน้าไซต์งาน (Image/Vision Processing)

---
**คณะผู้จัดทำโครงการ (Project Members):**
King Mongkut's University of Technology Thonburi (KMUTT)
- 68070700701 KRITTIN SUTTHIJIT
- 68070700703 CHENPHIPHAT SINGHARATTHANON
- 68070700711 PHAWITPHAT TEMUDOM
