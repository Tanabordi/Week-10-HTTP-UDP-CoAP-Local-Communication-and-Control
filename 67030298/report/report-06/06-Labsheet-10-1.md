# ใบงานการทดลองที่ 10.1
### การพัฒนาระบบค้นหาอุปกรณ์ด้วย mDNS และเว็บเซิร์ฟเวอร์ควบคุมอุปกรณ์ผ่าน HTTP RESTful API บน ESP-IDF

---

## 1. บันทึกผลการทดลอง (กิจกรรมที่ 10-1.5 การทดสอบและตรวจพิสูจน์)

### 1.1 ตรวจสอบ mDNS ด้วยคำสั่ง ping
- **คำสั่ง:**
  ```powershell
  ping esp32-node.local
  ```
- **ผลลัพธ์จาก Terminal:**
  ```text
  Pinging esp32-node.local [192.168.1.8] with 32 bytes of data:
  Reply from 192.168.1.8: bytes=32 time=1412ms TTL=64
  Reply from 192.168.1.8: bytes=32 time=607ms TTL=64
  Reply from 192.168.1.8: bytes=32 time=42ms TTL=64
  Reply from 192.168.1.8: bytes=32 time=1657ms TTL=64

  Ping statistics for 192.168.1.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
  Approximate round trip times in milli-seconds:
    Minimum = 42ms, Maximum = 1657ms, Average = 929ms
  ```
- **หมายเลข IP Address ที่ Resolve ได้:** `192.168.1.8`
- **ภาพบันทึกหน้าจอผลการ Ping:**
  ![alt text](image.png)

---

### 1.2 ทดสอบอ่านค่าเซนเซอร์ผ่าน cURL (`GET /api/status`)
- **คำสั่ง:**
  ```powershell 
  curl.exe -X GET http://esp32-node.local/api/status
  ```
- **ผลลัพธ์ JSON Payload:**
  ```json
  {"pot_raw":4095,"free_heap":213176,"led":false}
  ```

---

### 1.3 ทดสอบสั่งเปิด-ปิด LED ผ่าน cURL หรือ PowerShell (`POST /api/led`)

- **คำสั่งสั่งเปิดไฟ LED (`state: true`):**
  ```powershell
  curl.exe -s -X POST http://esp32-node.local/api/led -H "Content-Type: application/json" -d '{\"state\": true}'
  ```
  - **ข้อความตอบกลับ:** `{"result":"success"}`
  - **สถานะหลอดไฟ LED บนบอร์ดจริง:** [✅] ติด (ON)  [ ] ดับ (OFF)
![alt text](<image (1).png>)

- **คำสั่งสั่งปิดไฟ LED (`state: false`):**
  ```powershell
  curl.exe -s -X POST http://esp32-node.local/api/led -H "Content-Type: application/json" -d '{\"state\": false}'
  ```
  - **ข้อความตอบกลับ:** `{"result":"success"}`
  - **สถานะหลอดไฟ LED บนบอร์ดจริง:** [ ] ติด (ON)  [✅] ดับ (OFF)
![alt text](<image (2).png>)

---

## 2. คำถามท้ายการทดลอง

### ข้อ 1: การวิเคราะห์ HTTP Response Header และขนาดข้อมูลด้วย `curl -i`
นำผลการรัน `curl.exe -i http://esp32-node.local/api/status` มาแปะ พร้อมวิเคราะห์ว่า HTTP Header มีขนาดกี่ไบต์ และข้อมูล JSON มีขนาดกี่ไบต์

- **ผลการรัน `curl -i`:**
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/json
  Content-Length: 47

  {"pot_raw":4095,"free_heap":213176,"led":false}
  ```
- **การวิเคราะห์ขนาดข้อมูล:**
  - **ขนาด HTTP Header:** 68 ไบต์
  - **ขนาดข้อมูล JSON:** 47 ไบต์
  - **บทวิเคราะห์:** ตัวส่วนหัวข้อมูลมีขนาด 68 ไบต์ ซึ่งใหญ่กว่าตัวข้อมูลจริงที่ส่งเพียง 47 ไบต์ ทำให้กินแบนด์วิดท์โดยไม่จำเป็น จึงไม่ค่อยเหมาะกับการส่งข้อมูลบ่อยๆ บนอุปกรณ์ขนาดเล็ก

---

### ข้อ 2: การแก้ไขปัญหาเมื่อไม่สามารถใช้งาน mDNS ได้
หากในเครือข่ายมีคอมพิวเตอร์ที่ไม่รองรับ mDNS หรือปิดกั้นพอร์ต UDP 5353 จะเกิดผลกระทบอย่างไร และแก้ไขได้อย่างไร?

- **ผลกระทบ:** เครื่องคอมพิวเตอร์จะไม่สามารถค้นหาหรือแปลงชื่อเป็นหมายเลขไอพีได้ ทำให้เปิดใช้งานผ่านชื่อไม่ได้
- **แนวทางแก้ไข:** 
  1. ใช้งานผ่านหมายเลขไอพีของบอร์ดโดยตรง
  2. ตั้งค่าจับคู่ชื่อกับไอพีในระบบเครือข่ายหรือในเครื่องคอมพิวเตอร์
  3. เปิดอนุญาตพอร์ตในระบบความปลอดภัยหรือเปลี่ยนเครือข่ายให้เป็นแบบส่วนตัว

---

### ข้อ 3: การจัดการหน่วยความจำของ cJSON
เหตุใดจึงต้องเรียกคำสั่ง `cJSON_Delete(root)` และ `cJSON_free(resp)` เสมอหลังจากประมวลผลเสร็จสิ้น?

- **คำตอบ:** เพราะตัวระบบมีการจองพื้นที่ในแรมตอนสร้างข้อมูลและแปลงเป็นข้อความ ถ้าใช้งานเสร็จแล้วไม่สั่งคืนพื้นที่ แรมจะถูกกินไปเรื่อยๆ จนเต็มและทำให้บอร์ดค้างหรือรีสตาร์ทเองได้
