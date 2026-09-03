# การดาวน์โหลดไฟล์ PDF จากเว็บ WordPress

## สถานการณ์
หน้าเว็บ WordPress ที่มีไฟล์ PDF ฝังอยู่ (เช่น หลักสูตรมหาวิทยาลัย) โดยไม่มีลิงก์ดาวน์โหลดตรงๆ

## ขั้นตอน

### 1. หาลิงก์ PDF ในหน้าเว็บ
```bash
curl -s "<url>" | grep -oiE 'href="[^"]*\.pdf[^"]*"' | sed 's/href="//;s/"$//'
```

### 2. เปิดลิงก์ PDF ใน browser
ใช้ `browser_navigate` เปิดลิงก์ที่ได้ (browser จะ URL-encode Thai characters ให้อัตโนมัติ)

### 3. ดึง URL จริง
ใช้ `browser_console` รัน:
```javascript
window.location.href
```

### 4. ดาวน์โหลดด้วย curl
```bash
curl -sL -o "<filename>.pdf" "<url>"
```

### 5. ตรวจสอบไฟล์
```bash
file "<filename>.pdf"
# ควรแสดง: PDF document, version X.X
# ถ้าแสดง: HTML document → URL ผิด ต้องลองใหม่
```

## ปัญหาที่พบบ่อย

| ปัญหา | สาเหตุ | วิธีแก้ |
|-------|--------|--------|
| ได้ไฟล์ HTML แทน PDF | URL encoding ผิด | เปิดใน browser ก่อน แล้วดึง URL จริง |
| 404 Not Found | ไฟล์ถูกย้าย/ลบ | ตรวจสอบลิงก์ใหม่ในหน้าเว็บ |
| ไฟล์เล็กผิดปกติ (< 100KB) | ได้ error page แทน | ใช้ `file` ตรวจสอบ แล้วลองใหม่ |

## ตัวอย่างจริง

**เว็บ:** https://idt.fte.rmuti.ac.th/ปริญญาตรี/
**PDF ที่พบ:** เอกสารหลักสูตร-ฉบับปรับปรุงแก้ไขตามข้อเ.pdf (298 หน้า, ~15MB)
**วิธีที่ใช้:** curl grep → browser navigate → curl download

## หมายเหตุ
- WordPress มักเก็บไฟล์อัปโหลดไว้ที่ `/wp-content/uploads/`
- ชื่อไฟล์อาจมีภาษาไทย → ต้อง URL-encode
- Browser's built-in PDF viewer มีปุ่ม Download อยู่แล้ว (ref จาก snapshot)
