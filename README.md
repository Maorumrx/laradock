# Laradock — POS Project

Docker stack สำหรับ **Restaurant POS + QR Ordering System**

> Laravel 12 app อยู่ที่ `pos/` submodule → mount เป็น `/var/www/pos/` ในทุก container

---

## Services ที่ใช้

| Service      | Port | รายละเอียด                             |
|--------------|------|----------------------------------------|
| Nginx        | 80   | Serve `/var/www/pos/public`            |
| MariaDB      | 3306 | DB: `pos-database`, root: `root`       |
| Redis        | 6379 | password: `secret_redis`               |
| Soketi       | 6001 | WebSocket (Pusher-compatible)          |
| phpMyAdmin   | 8081 | user: `default`, password: `secret`    |
| Workspace    | —    | PHP 8.4 + Composer + Node (dev shell)  |

---

## เริ่มใช้งาน

```bash
cd laradock

# ครั้งแรก — copy .env
cp .env.example .env

# Start services
docker compose up -d nginx mariadb redis soketi phpmyadmin workspace

# เข้า workspace shell
docker compose exec workspace bash
cd /var/www/pos
```

---

## คำสั่งที่ใช้บ่อย

```bash
# ดู logs
docker compose logs -f nginx
docker compose logs -f workspace

# Restart service
docker compose restart nginx

# Stop ทั้งหมด
docker compose down

# Rebuild หลังแก้ Dockerfile หรือ .env
docker compose build workspace
docker compose up -d
```

---

## การตั้งค่า `.env` หลัก

```env
APP_CODE_PATH_HOST=../pos
APP_CODE_PATH_CONTAINER=/var/www/pos
COMPOSE_PROJECT_NAME=pos
PHP_VERSION=8.4

MARIADB_DATABASE=pos-database
MARIADB_ROOT_PASSWORD=root

REDIS_PASSWORD=secret_redis

SOKETI_PORT=6001

PMA_PORT=8081
PMA_DB_ENGINE=mysql
PMA_USER=default
PMA_PASSWORD=secret
PMA_ROOT_PASSWORD=secret
```

---

## Soketi Config

ไฟล์: `soketi/config.json`

```json
{
    "appManager.array.apps": [{
        "id": "myapp-key",
        "key": "myapp-key",
        "secret": "myapp-secret"
    }]
}
```

ตรงกับ `.env` ของ Laravel app:
```env
BROADCAST_CONNECTION=pusher
PUSHER_APP_KEY=myapp-key
PUSHER_APP_SECRET=myapp-secret
PUSHER_HOST=soketi
PUSHER_PORT=6001
PUSHER_SCHEME=http
```

---

## Nginx Site Config

ไฟล์: `nginx/sites/app.conf` — root ชี้ไปที่ `/var/www/pos/public`

> ถ้าต้องการเปลี่ยน domain หรือ SSL แก้ที่ไฟล์นี้แล้ว `docker compose restart nginx`
