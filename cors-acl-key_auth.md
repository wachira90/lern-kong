เยี่ยมเลย! 🎯 มาต่อกันกับ 3 Plugin สำคัญใน Kong:

---

## 🌐 1. **CORS Plugin**  
(Cross-Origin Resource Sharing)  
ใช้เพื่ออนุญาตหรือจำกัดการเรียก API จาก domain ภายนอก เช่น frontend ต่าง origin

### ✅ การติดตั้ง Plugin CORS

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=cors \
  --data config.origins='*' \
  --data config.methods='GET, POST, OPTIONS' \
  --data config.headers='Accept, Authorization, Content-Type' \
  --data config.exposed_headers='X-Custom-Header' \
  --data config.credentials=true \
  --data config.max_age=3600
```

📝 ปรับค่า `origins` ถ้าต้องการอนุญาตเฉพาะ domain เช่น `https://myfrontend.com`

---

## 🔒 2. **ACL Plugin**  
(Access Control List)  
ใช้ควบคุมว่า consumer กลุ่มไหนเข้าถึง API นี้ได้

### ✅ การใช้งานร่วมกับ Auth Plugin เช่น Key Auth หรือ JWT

#### 2.1 สร้าง ACL group

```bash
curl -i -X POST http://localhost:8001/consumers/myuser/acls \
  --data group=premium
```

#### 2.2 เปิดใช้งาน Plugin ACL บน Service หรือ Route

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=acl \
  --data config.whitelist=premium \
  --data config.hide_groups_header=true
```

📌 `whitelist=premium` หมายถึงเฉพาะ consumer ที่อยู่ใน group `premium` ถึงเข้าได้

---

## 🔑 3. **Key Auth Plugin**  
(ใช้ API Key เพื่อเข้าถึง API)

### ✅ การเปิดใช้ Plugin Key Auth

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=key-auth \
  --data config.key_names=apikey \
  --data config.hide_credentials=true
```

#### 3.1 สร้าง Consumer

```bash
curl -i -X POST http://localhost:8001/consumers \
  --data username=demo_user
```

#### 3.2 สร้าง Key ให้ consumer

```bash
curl -i -X POST http://localhost:8001/consumers/demo_user/key-auth
```

📌 คุณจะได้ `key` สำหรับใช้เรียก API เช่น:

```bash
curl -i http://localhost:8000/example \
  --header "apikey: <your-key-here>"
```

---

## 🔁 สรุปการใช้งาน:

| Plugin     | หน้าที่                        | ใช้ร่วมกับ              |
|------------|-------------------------------|--------------------------|
| CORS       | อนุญาต cross-origin           | ทุก service              |
| ACL        | จำกัดการเข้าถึงตามกลุ่ม      | JWT, Key Auth, OAuth     |
| Key Auth   | ตรวจสอบ API Key               | ใช้แทนหรือเสริม JWT     |

---
