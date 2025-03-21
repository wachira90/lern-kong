ขั้นตอน **การติดตั้งและใช้งาน Plugin** ยอดฮิตใน Kong ได้แก่:

1. ✅ JWT Authentication  
2. 🚦 Rate Limiting  
3. 📜 Logging (เช่น File Log หรือ HTTP Log)

---

## ✅ 1. การใช้งาน Plugin JWT Authentication

JWT ใช้สำหรับตรวจสอบตัวตนของผู้ใช้ โดยจะให้ client ส่ง token มาพร้อมกับการเรียก API

### 🔹 ขั้นตอน:

#### 1.1 สร้าง service และ route

```bash
curl -i -X POST http://localhost:8001/services \
  --data name=example-service \
  --data url=http://httpbin.org/anything

curl -i -X POST http://localhost:8001/services/example-service/routes \
  --data paths[]=/example
```

#### 1.2 เปิดใช้งาน plugin JWT

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=jwt
```

#### 1.3 สร้าง consumer (ตัวแทน user)

```bash
curl -i -X POST http://localhost:8001/consumers \
  --data username=myuser
```

#### 1.4 สร้าง JWT credential

```bash
curl -i -X POST http://localhost:8001/consumers/myuser/jwt
```

📌 จะได้ response ที่มี `key` และ `secret` สำหรับสร้าง JWT token

#### 1.5 สร้าง JWT token (ตัวอย่างด้วย `jsonwebtoken` บน Node.js):

```js
const jwt = require('jsonwebtoken');
const payload = {
  iss: 'your-key', // จากขั้นตอน 1.4
  exp: Math.floor(Date.now() / 1000) + 3600, // หมดอายุ 1 ชม.
};

const token = jwt.sign(payload, 'your-secret'); // จากขั้นตอน 1.4
```

#### 1.6 เรียก API พร้อม JWT token

```bash
curl -i http://localhost:8000/example \
  -H "Authorization: Bearer <your_token>"
```

---

## 🚦 2. การใช้งาน Plugin Rate Limiting

จำกัดจำนวน request ต่อ user หรือ IP เพื่อลดการโดน spam

### 🔹 เปิดใช้งาน plugin กับ service หรือ route:

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=rate-limiting \
  --data config.minute=5 \
  --data config.policy=local
```

👆 ตัวอย่างนี้ จำกัด 5 requests ต่อนาที

---

## 📜 3. การใช้งาน Plugin Logging (HTTP Log หรือ File Log)

Kong รองรับ logging หลายแบบ เช่น HTTP log, File log, Syslog ฯลฯ

### 🔹 HTTP Log (ส่ง log ไปยัง endpoint ที่รับข้อมูล)

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=http-log \
  --data config.http_endpoint=http://requestbin.net/r/xxxxxx \
  --data config.method=POST \
  --data config.timeout=1000 \
  --data config.keepalive=1000
```

> 💡 ใช้ [https://webhook.site](https://webhook.site) หรือ [https://requestbin.com](https://requestbin.com) เพื่อดู log ได้สด ๆ

### 🔹 File Log (เขียนลงไฟล์ใน container)

```bash
curl -i -X POST http://localhost:8001/services/example-service/plugins \
  --data name=file-log \
  --data config.path=/tmp/kong.log
```

> จากนั้นเข้าไปใน container ด้วย `docker exec -it kong bash` และดู log ได้ที่ `/tmp/kong.log`

---

## 🔁 สรุปการใช้ Plugins:

| Plugin        | ใช้ทำอะไร         | วิธีเรียกใช้ |
|---------------|------------------|----------------|
| JWT           | ตรวจสอบตัวตนผ่าน token | `/plugins?name=jwt` |
| Rate Limiting | จำกัด request    | `/plugins?name=rate-limiting` |
| Logging       | เก็บ log การใช้งาน | `/plugins?name=http-log` หรือ `file-log` |

---
