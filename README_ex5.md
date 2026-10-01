# Báo cáo Bài 5: Quản lý quyền sở hữu thư mục Web (Directory Permission Management)

- **Môn học:** DevOps in Action — Session 02
- **Đường dẫn nộp:** `homework/session_02/ex5/`
- **Máy chủ:** VPS Ubuntu 22.04 LTS (AZVPS) — IP `160.187.229.73`
- **User thường:** `devops` (uid 1000, thuộc group `sudo`)

---

## 1. Mục tiêu
Phân quyền cho thư mục `/var/www/ptit-web/` sao cho:
- User **`devops`** (owner) được **đọc + ghi** → chỉnh sửa mã nguồn không cần `sudo`.
- Group **`www-data`** (nhóm Nginx) được **đọc** → phục vụ web tĩnh.
- Người dùng khác **không có quyền truy cập**.
- Nginx **không bị lỗi 403 Forbidden**.

---

## 2. Chuẩn bị thư mục web `ptit-web`

Ban đầu máy chủ chỉ có thư mục mặc định `/var/www/html`, chưa có `/var/www/ptit-web/`. Tạo mới cấu trúc web và trỏ Nginx về đó:

```bash
# 2.1. Tạo thư mục web + file index
sudo mkdir -p /var/www/ptit-web/html
echo "<h1>PTIT Web - DevOps</h1>" | sudo tee /var/www/ptit-web/html/index.html

# 2.2. Tạo Nginx server block trỏ root về ptit-web
sudo tee /etc/nginx/sites-available/ptit-web > /dev/null <<'EOF'
server {
    listen 80;
    server_name 160.187.229.73;
    root /var/www/ptit-web/html;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
EOF

# 2.3. Kích hoạt site, gỡ site mặc định, kiểm tra & reload
sudo ln -sf /etc/nginx/sites-available/ptit-web /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

## 3. Phân quyền (phần chính của bài)

```bash
# 3.1. Đổi owner -> devops, group -> www-data (đệ quy)
sudo chown -R devops:www-data /var/www/ptit-web/

# 3.2. Thư mục: owner rwx, group r-x, others không có gì  -> 750
sudo find /var/www/ptit-web/ -type d -exec chmod 750 {} \;

# 3.3. File: owner rw, group r, others không có gì  -> 640
sudo find /var/www/ptit-web/ -type f -exec chmod 640 {} \;
```

> 💡 Thư mục cần quyền `execute` để Nginx (`www-data`) duyệt vào — đó là lý do tách **750 cho thư mục / 640 cho file** thay vì chmod đều, tránh lỗi 403.

---

## 4. Kiểm tra (Verification)

### 4.1. Xem phân quyền
```bash
ls -la /var/www/ptit-web/
```
**Kết quả mong đợi** (owner `devops`, group `www-data`):
```
drwxr-x---  3 devops www-data 4096 Oct  1 20:21 .
drwxr-xr-x  4 root   root     4096 Oct  1 20:21 ..
drwxr-x---  2 devops www-data 4096 Oct  1 20:21 html
```

### 4.2. Ghi file bằng user devops (KHÔNG dùng sudo)
```bash
su - devops
echo "Update" >> /var/www/ptit-web/html/index.html
cat /var/www/ptit-web/html/index.html
exit
```
→ Ghi thành công, không báo `Permission denied`.

### 4.3. Web không bị 403
```bash
curl http://160.187.229.73
```
→ Trả về nội dung trang (có dòng "Update"), không lỗi 403.

---

## 5. Kết quả
- `/var/www/ptit-web/` thuộc sở hữu `devops:www-data`.
- `devops` ghi/sửa được không cần sudo; Nginx đọc và phục vụ web bình thường; user khác bị chặn.
- Website hiển thị nội dung mới, không có lỗi 403.

---

## 6. Output / Ảnh cần nộp
1. Output `ls -la /var/www/ptit-web/`.
2. Các bước ghi file từ user `devops` thành công.

![ls -la output](ls-la.png)
![devops write test](write-test.png)

---

## 7. Cách thức nộp bài
```bash
mkdir -p homework/session_02/ex5
# Copy README.md và ảnh vào thư mục trên, rồi:
git add homework/session_02/ex5
git commit -m "Session 02 - Bai 5: Directory Permission Management"
git push origin main
```
Sau khi push, vào mục **"Nộp bài"** trên portal Rikkei và dán link GitHub tới `homework/session_02/ex5/`.
