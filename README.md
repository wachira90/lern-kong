**Docker Kong คืออะไร?**

Kong คือ API Gateway แบบโอเพ่นซอร์สที่ใช้ในการจัดการ API เช่น การควบคุมการเข้าถึง, การจัดการ rate limiting, การทำ load balancing, การตรวจสอบสิทธิ์ (Authentication) ฯลฯ  
เมื่อใช้งานร่วมกับ **Docker** จะช่วยให้สามารถติดตั้งและจัดการ Kong ได้ง่ายขึ้นผ่าน container โดยไม่ต้องติดตั้งลงในเครื่องโดยตรง

---

### จุดเด่นของ Kong

- รองรับ plugin สำหรับการตรวจสอบสิทธิ์, logging, caching, rate-limiting
- ทำงานเร็วและมีประสิทธิภาพสูง (เขียนด้วยภาษา Lua และรันบน Nginx)
- รองรับ load balancing
- รองรับ clustering (เชื่อมต่อหลาย instance)

---

## การใช้งาน Docker Kong แบบเบื้องต้น

### 1. เตรียม Docker และ Docker Compose

ต้องติดตั้ง Docker และ Docker Compose ให้พร้อมก่อนใช้งาน

---

### 2. สร้าง `docker-compose.yml`

```yaml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    container_name: kong-database
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong
    volumes:
      - kong-data:/var/lib/postgresql/data

  kong:
    image: kong:3.4
    container_name: kong
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"   # Proxy port
      - "8443:8443"   # Proxy SSL
      - "8001:8001"   # Admin API
      - "8444:8444"   # Admin API SSL
    depends_on:
      - kong-database

  kong-migrations:
    image: kong:3.4
    container_name: kong-migrations
    command: kong migrations bootstrap
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
    depends_on:
      - kong-database

volumes:
  kong-data:
```

---

### 3. รัน Kong ด้วย Docker Compose

```bash
docker-compose up -d
```

---

### 4. ทดสอบ Kong Admin API

ลองเรียก API เพื่อตรวจสอบว่า Kong ทำงานได้แล้ว:

```bash
curl http://localhost:8001/
```

---

### 5. เริ่มใช้งาน Kong

คุณสามารถเริ่มเพิ่ม API services, routes และ plugins ผ่าน Kong Admin API เช่น:

**สร้าง service:**

```bash
curl -i -X POST http://localhost:8001/services \
  --data name=example-service \
  --data url=http://example.com
```

**สร้าง route:**

```bash
curl -i -X POST http://localhost:8001/services/example-service/routes \
  --data paths[]=/example
```

**ทดสอบ:**

```bash
curl http://localhost:8000/example
```

---

หากอยากให้ช่วย setup หรือใช้งาน plugin ต่าง ๆ เช่น JWT Auth, Rate Limiting, Logging ฯลฯ ก็บอกได้นะครับ เดี๋ยวช่วยจัดให้! 😄
