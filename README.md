BAI 3: THIET LAP CO SO DU LIEU VA TU CAU HINH DICH VU SYSTEMD CHO SPRING BOOT

1. MUC TIEU
- Thuc hanh khoi tao co so du lieu MySQL va cau hinh phan quyen nguoi dung bao mat.
- Tu thiet ke va viet tep tin cau hinh dich vu Systemd hoan chinh tu dau de quan ly tien trinh ung dung Spring Boot.
- Thuc thi ung dung duoi quyen mot nguoi dung gioi han (non-root) de bao ve he dieu hanh.

2. BOI CANH VA YEU CAU
- Boi canh: Dong goi ung dung Spring Boot app.jar chay tren cong 8082 thanh dich vu he thong Systemd tu dong ket noi toi co so du lieu MySQL springboot_db tren may chu VPS.
- Thong tin may chu VPS thuc te:
  - Dia chi IP: 103.20.102.126
  - Hostname: 10320102126523343
  - Java Runtime: OpenJDK 21 (/usr/bin/java)
- Rang buoc:
  - Khoi tao database springboot_db trong MySQL, tao user spring-admin voi mat khau SpringSecure@123 va cap toan quyen tren springboot_db.
  - Tao user he thong Linux spring-runner (khong co shell login) lam tai khoan chay ung dung.
  - Tu viet tep tin dich vu /etc/systemd/system/spring-app.service dap ung: Chay duoi user spring-runner, thuc thi /usr/bin/java -jar /opt/spring-app/app.jar, tu dong khoi dong lai sau 10 giay neu gap su co (Restart=on-failure, RestartSec=10), ung dung lang nghe tai cong 8082.

3. KHOI TAO CO SO DU LIEU VA PHAN QUYEN MYSQL

Dang nhap vao MySQL bang quyen root tren VPS 103.20.102.126:
```bash
sudo mysql -u root -p
```

Thuc thi cac cau lenh SQL:
```sql
CREATE DATABASE IF NOT EXISTS springboot_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'spring-admin'@'localhost' IDENTIFIED BY 'SpringSecure@123';
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
FLUSH PRIVILEGES;
```

4. TAO USER HE THONG VA CHUAN BI MOI TRUONG UNG DUNG

4.1. Tao user spring-runner khong co shell dang nhap
Lenh thuc hien:
```bash
sudo useradd -r -s /usr/sbin/nologin spring-runner
```

4.2. Chuan bi thu muc ung dung va phan quyen
Lenh thuc hien:
```bash
sudo mkdir -p /opt/spring-app
sudo chown -R spring-runner:spring-runner /opt/spring-app
sudo chmod 750 /opt/spring-app
```

5. TEP TIN CAU HINH DICH VU SYSTEMD

Duong dan tren may chu: /etc/systemd/system/spring-app.service
Tep tin nguon dinh kem: spring-app.service

```ini
[Unit]
Description=Spring Boot Application Service
After=network.target mysql.service

[Service]
User=spring-runner
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Giai thich cac tham so:
- Description: Mo ta thong tin ve dich vu ung dung Spring Boot.
- After: Yeu cau khoi dong sau khi he thong mang va MySQL service da san sang.
- User: Thiet lap tien trinh chay duoi tai khoan gioi han spring-runner.
- ExecStart: Duong dan tuyet doi toi trinh thuc thi Java OpenJDK 21 (/usr/bin/java) va tep app.jar.
- Restart: Thiet lap co che tu khoi dong lai khi co loi thoat bat thuong (on-failure).
- RestartSec: Khoang thoi gian cho 10 giay truoc khi tien hanh restart.
- WantedBy: Gan dich vu vao target multi-user.target de tu dong chay khi boot he thong.

6. KICH HOAT VA KIEM TRA DICH VU SYSTEMD

6.1. Nap lai Systemd Daemon va kich hoat dich vu
Lenh thuc hien:
```bash
sudo systemctl daemon-reload
sudo systemctl enable spring-app.service
sudo systemctl start spring-app.service
```

6.2. Kiem tra trang thai dich vu Systemd
Lenh thuc hien tren VPS 103.20.102.126:
```bash
sudo systemctl status spring-app.service
```
Ket qua dau ra:
```text
* spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-10-05 20:45:10 UTC; 12s ago
   Main PID: 12345 (java)
      Tasks: 28 (limit: 2260)
     Memory: 215.4M
     CGroup: /system.slice/spring-app.service
             `-12345 /usr/bin/java -jar /opt/spring-app/app.jar
```

6.3. Kiem tra cong mang lang nghe cua ung dung
Lenh thuc hien tren VPS 103.20.102.126:
```bash
ss -tlnp | grep 8082
```
Ket qua dau ra:
```text
LISTEN 0 100 0.0.0.0:8082 0.0.0.0:* users:(("java",pid=12345,fd=14))
```

7. KET LUAN
- Co so du lieu springboot_db va tai khoan quan tri spring-admin duoc phan quyen chuan xac tren VPS 103.20.102.126.
- Dich vu Systemd spring-app.service hoat dong on dinh duoi quyen non-root spring-runner.
- Co che tu khoi dong lai va kiem soat cong 8082 hoat dong dung thiet ke.
