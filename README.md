---

![macOS](https://img.shields.io/badge/macOS-000000?style=for-the-badge&logo=apple&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Python](https://img.shields.io/badge/python-v3.8+-blue.svg)

---

# 🚀 Cài đặt Odoo ERP trên macOS với Docker Desktop

## 1. Mục tiêu

- ✅ Cài đặt môi trường chạy Odoo ERP trên macOS
- ✅ Sử dụng Docker Desktop để chạy Odoo + PostgreSQL
- ✅ Truy cập hệ thống tại `http://localhost:8069`

---

## 2. Kiến trúc hệ thống

```
macOS
├── Docker Desktop
│   ├── PostgreSQL (container)
│   └── Odoo ERP (container)
└── Project source code (trên máy macOS)
```

---

## 3. Cài đặt & Hướng dẫn

### 3.1. Cài đặt Docker Desktop

1. **Tải Docker Desktop cho macOS**
   - Truy cập: https://www.docker.com/products/docker-desktop
   - Chọn phiên bản phù hợp với chip Mac (Apple Silicon hoặc Intel)

2. **Cài đặt**
   - Kéo **Docker.app** vào thư mục **Applications**
   - Mở **Applications → Docker.app**
   - Nhập password khi được yêu cầu

3. **Kiểm tra cài đặt thành công**
   ```bash
   docker --version
   docker-compose --version
   ```

---

### 3.2. Clone project từ GitHub

```bash
git clone https://github.com/hieupham10032003/odoo.git
cd odoo
```

---

### 3.3. Tạo file `docker-compose.yml`

Nếu chưa có, tạo file `docker-compose.yml` trong thư mục project:

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:14
    container_name: odoo-postgres
    environment:
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo
      POSTGRES_DB: odoo
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - odoo-network

  odoo:
    image: odoo:16.0
    container_name: odoo-app
    depends_on:
      - postgres
    ports:
      - "8069:8069"
    environment:
      HOST: postgres
      USER: odoo
      PASSWORD: odoo
      DB_PORT: 5432
    volumes:
      - ./addons:/mnt/extra-addons
      - ./config:/etc/odoo
    networks:
      - odoo-network
    command: odoo -c /etc/odoo/odoo.conf

volumes:
  postgres_data:

networks:
  odoo-network:
    driver: bridge
```

---

### 3.4. Khởi động Docker containers

```bash
docker-compose up -d
```

**Kiểm tra containers đang chạy:**

```bash
docker-compose ps
```

Expected output:

```
NAME              STATUS
odoo-postgres     Up ...
odoo-app          Up ...
```

---

### 3.5. Tạo file cấu hình `odoo.conf`

Tạo file `config/odoo.conf`:

```ini
[options]
addons_path = /mnt/extra-addons
db_host = postgres
db_port = 5432
db_user = odoo
db_password = odoo
db_name = odoo
xmlrpc_port = 8069
workers = 0
```

---

### 3.6. Truy cập Odoo

Mở trình duyệt và truy cập:

```
http://localhost:8070
```

> ℹ️ Port được thay đổi từ 8069 sang 8070 để tránh xung đột. Nếu muốn dùng port khác, sửa trong `docker-compose.yml`

Màn hình khởi tạo database Odoo xuất hiện → **Thành công! 🎉**

---

## 4. Cài đặt addons (tùy chọn)

Nếu cần cài thêm addons:

```bash
# Vào folder project
cd Business-Internship

# Clone hoặc copy addons vào thư mục ./addons

# Cập nhật Odoo
docker-compose restart odoo
```

---

## 5. Các lệnh Docker hữu ích

**Xem logs của containers:**

```bash
docker-compose logs -f odoo
```

**Tắt containers:**

```bash
docker-compose down
```

**Tắt containers + xóa volumes (xóa database):**

```bash
docker-compose down -v
```

**Khởi động lại:**

```bash
docker-compose restart
```

**Truy cập terminal trong container Odoo:**

```bash
docker-compose exec odoo bash
```

---

## 6. Quản lý Database với pgAdmin (tùy chọn)

Thêm vào `docker-compose.yml`:

```yaml
pgadmin:
  image: dpage/pgadmin4
  container_name: pgadmin
  environment:
    PGADMIN_DEFAULT_EMAIL: admin@admin.com
    PGADMIN_DEFAULT_PASSWORD: admin
  ports:
    - "5050:80"
  depends_on:
    - postgres
  networks:
    - odoo-network
```

Sau đó truy cập: `http://localhost:5050`

- Email: `admin@admin.com`
- Password: `admin`

---

## 7. Lỗi thường gặp

| Vấn đề                      | Nguyên nhân                   | Giải pháp                                          |
| --------------------------- | ----------------------------- | -------------------------------------------------- |
| Port 8069 đã được sử dụng   | Odoo chạy ở cổng khác         | Thay đổi trong `docker-compose.yml`: `"8070:8069"` |
| Docker daemon không chạy    | Docker Desktop chưa khởi động | Mở **Docker.app** từ Applications                  |
| Không kết nối được database | PostgreSQL chưa sẵn sàng      | Đợi vài giây rồi reload page                       |
| Permission denied           | Lỗi quyền hạn                 | Chạy `sudo` hoặc thêm user vào docker group        |

---

## 8. Checklist hoàn thành

- [ ] Docker Desktop cài đặt & chạy
- [ ] `docker --version` & `docker-compose --version` hoạt động
- [ ] Project cloned từ GitHub
- [ ] File `docker-compose.yml` cấu hình đúng
- [ ] Containers chạy thành công (`docker-compose ps`)
- [ ] Truy cập được `http://localhost:8069`
- [ ] Màn hình khởi tạo database Odoo hiển thị

---

## 9. Ghi nhớ quan trọng

⚠️ **Docker Desktop phải luôn chạy để sử dụng Odoo**

✅ **Ưu điểm của phương pháp này:**

- Không cần cài thư viện phức tạp trên macOS
- Dễ dàng quản lý các version
- Có thể lưu trữ toàn bộ cấu hình trong project

---
