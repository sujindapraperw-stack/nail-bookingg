# 📖 คู่มือติดตั้งระบบจองคิวร้านเล็บ LINE Bot

## 🗂 โครงสร้างไฟล์
```
nail-booking/
├── src/
│   └── server.js        ← ตัวหลัก: LINE webhook + API
├── public/
│   └── index.html       ← หน้าจัดการคิว (เจ้าของร้านดู)
├── package.json
├── .env.example         ← ตัวอย่างค่าที่ต้องตั้ง
└── SETUP.md             ← ไฟล์นี้
```

---

## ขั้นตอนที่ 1 — ตั้งค่า LINE Messaging API

### 1.1 เข้า LINE Developers Console
1. ไปที่ https://developers.line.biz
2. Login ด้วยบัญชี LINE ของคุณ
3. เลือก Provider ของร้าน (หรือสร้างใหม่)
4. คลิก **Create a new channel** → เลือก **Messaging API**

### 1.2 กรอกข้อมูล Channel
- Channel type: **Messaging API**
- Provider: เลือก Provider ของร้าน
- Channel name: ชื่อร้านเล็บ
- Channel description: คำอธิบายร้าน
- Category: **Beauty**
- Subcategory: **Nail salon**

### 1.3 เก็บ Key สำคัญ 2 ตัว
เมื่อสร้าง Channel แล้ว ให้เก็บ:

**Channel Secret** (Basic settings tab)
```
LINE_CHANNEL_SECRET=abc123xyz...
```

**Channel Access Token** (Messaging API tab → กดปุ่ม Issue)
```
LINE_CHANNEL_ACCESS_TOKEN=eyJhbGci...
```

### 1.4 ตั้งค่า Channel
ใน Messaging API tab:
- **Allow bot to join group chats**: Off
- **Auto-reply messages**: Disabled ← สำคัญมาก!
- **Greeting messages**: แก้หรือปิดก็ได้

---

## ขั้นตอนที่ 2 — Deploy บน Render (ฟรี)

### 2.1 อัปโหลดโค้ดขึ้น GitHub
```bash
# ถ้ายังไม่มี Git ให้ติดตั้งก่อน
git init
git add .
git commit -m "initial commit"

# สร้าง repo ใหม่บน GitHub แล้ว push
git remote add origin https://github.com/ชื่อคุณ/nail-booking.git
git push -u origin main
```

### 2.2 สร้าง Web Service บน Render
1. ไปที่ https://render.com → Sign up/Login
2. คลิก **New +** → **Web Service**
3. เชื่อมต่อ GitHub → เลือก repo `nail-booking`
4. ตั้งค่าดังนี้:

| ฟิลด์ | ค่า |
|-------|-----|
| Name | nail-booking (หรือชื่อที่ต้องการ) |
| Region | Singapore (ใกล้ไทยที่สุด) |
| Branch | main |
| Build Command | (ว่าง) |
| Start Command | `node src/server.js` |
| Instance Type | **Free** |

5. คลิก **Create Web Service**
6. รอประมาณ 2-3 นาที จะได้ URL เช่น:
   `https://nail-booking.onrender.com`

### 2.3 ตั้ง Environment Variables บน Render
ใน dashboard → **Environment** tab → กด **Add Environment Variable**

```
LINE_CHANNEL_SECRET      = [ค่าจาก LINE Console]
LINE_CHANNEL_ACCESS_TOKEN = [ค่าจาก LINE Console]
SHOP_NAME                = ร้านเล็บสวยงาม
OPEN_HOUR                = 10
CLOSE_HOUR               = 20
SLOT_MINUTES             = 60
NUM_STAFF                = 2
```

---

## ขั้นตอนที่ 3 — เชื่อม Webhook กับ LINE

### 3.1 กรอก Webhook URL
1. กลับไป LINE Developers Console
2. Messaging API tab → **Webhook settings**
3. Webhook URL: `https://[ชื่อของคุณ].onrender.com/webhook`
4. เปิด **Use webhook**: ON
5. กด **Verify** — ถ้าขึ้น Success แปลว่าใช้งานได้!

---

## ขั้นตอนที่ 4 — ทดสอบ

### 4.1 ทดสอบบอทไลน์
1. สแกน QR Code จาก LINE Console → Bot tab
2. กด **Add friend** เพิ่มบอทเป็นเพื่อน
3. ส่งข้อความ "จองคิว" — ควรตอบกลับทันที

### 4.2 ทดสอบหน้าจัดการคิว
เปิด browser ไปที่:
```
https://[ชื่อของคุณ].onrender.com
```
จะเห็นหน้า dashboard แสดงรายการจองทั้งหมด

---

## ⚙️ ปรับแต่งร้าน

### เปลี่ยนเวลาเปิด-ปิด
```
OPEN_HOUR=9      ← เปิด 9 โมง
CLOSE_HOUR=21    ← ปิด 3 ทุ่ม
```

### เปลี่ยนเวลาต่อคิว
```
SLOT_MINUTES=90  ← คิวละ 1.5 ชั่วโมง
```

### เพิ่มจำนวนช่าง
```
NUM_STAFF=3      ← ช่าง 3 คน (รับได้ 3 คนต่อชั่วโมง)
```

---

## 🔍 แก้ปัญหาที่พบบ่อย

### บอทไม่ตอบ
- ตรวจสอบ Auto-reply ใน LINE Console ว่าปิดอยู่
- กด Verify webhook อีกครั้ง
- ดู logs ใน Render dashboard → Logs tab

### Render หยุดทำงานหลัง 15 นาที (free tier)
เป็นเรื่องปกติของ free plan — เซิร์ฟเวอร์จะ wake up เมื่อมีคนส่งข้อความ
อาจช้า 30-60 วินาทีครั้งแรก แก้ได้โดย:
- ใช้ https://uptimerobot.com ping ทุก 10 นาที (ฟรี)
- หรืออัปเกรดเป็น Render paid plan ($7/เดือน)

### ข้อมูลหาย
ระบบนี้เก็บข้อมูลในหน่วยความจำ (in-memory) หากเซิร์ฟเวอร์รีสตาร์ท ข้อมูลจะหาย
สำหรับร้านที่ต้องการเก็บข้อมูลถาวร ควรเพิ่ม database เช่น Supabase (ฟรี)

---

## 💬 สรุปคำสั่งที่ลูกค้าใช้ได้

| พิมพ์ | ผลลัพธ์ |
|-------|---------|
| `จองคิว` หรือ `จอง` | เริ่มกระบวนการจองคิว |
| `ดูคิว` | ดูคิวที่จองไว้ |
| `ยกเลิก BK0001` | ยกเลิกคิวรหัส BK0001 |
| อื่นๆ | แสดงเมนูหลัก |
