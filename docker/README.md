#  VNC desktop trong Docker container
**Mục tiêu:** chạy một container Ubuntu có desktop (XFCE) và VNC, kết nối từ Windows qua SSH tunnel.

---

## Yêu cầu (Chuẩn bị)
- Windows với **Docker Desktop** (Linux containers).
- **PowerShell** và **OpenSSH Client** (kiểm tra `ssh -V`).
- **RealVNC Viewer** (hoặc VNC client khác) trên Windows.

---

## Tóm tắt các bước
1. Build & chạy container  
2. SSH vào container  
3. Cài desktop environment + VNC trong container  
4. Tạo mật khẩu VNC và file `xstartup`  
5. Đảm bảo quyền cho socket X11  
6. Khởi động VNC server  
7. Tạo SSH tunnel trên Windows và kết nối bằng RealVNC

---

## Bước 1 — Build và chạy container
Mở PowerShell, vào thư mục chứa `Dockerfile` và `docker-compose.yml`, chạy:
```powershell
docker compose up -d --build
docker ps
Test-NetConnection 127.0.0.1 -Port 2222
```

---

## Bước 2 — SSH vào container
Trên Windows (PowerShell / cmd / WSL):
```bash
ssh student@127.0.0.1 -p 2222
# password: student
```

---

## Bước 3 — Cài DE (XFCE) + VNC bên trong container
Trong shell Ubuntu (sau khi SSH):
```bash
sudo apt update
sudo apt install -y xfce4 xfce4-goodies tigervnc-standalone-server dbus-x11 xauth x11-xserver-utils xfonts-base
```

---

## Bước 4 — Tạo mật khẩu VNC và file xstartup
Tạo thư mục `.vnc` và đặt mật khẩu:
```bash
mkdir -p ~/.vnc
vncpasswd   # nhập mật khẩu (không cần view-only)
```

Tạo `~/.vnc/xstartup` (toàn bộ nội dung dưới đây):
```sh
cat > ~/.vnc/xstartup << 'EOF'
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

export XDG_RUNTIME_DIR="/run/user/$(id -u)"
mkdir -p "$XDG_RUNTIME_DIR"
chmod 700 "$XDG_RUNTIME_DIR"

[ -f "$HOME/.Xresources" ] && xrdb "$HOME/.Xresources"

exec dbus-launch --exit-with-session startxfce4
EOF
```

Thiết lập quyền:
```bash
chmod 700 ~/.vnc
chmod 600 ~/.vnc/passwd
chmod +x ~/.vnc/xstartup
```

---

## Bước 5 — Đảm bảo socket X11 có quyền đúng
```bash
sudo mkdir -p /tmp/.X11-unix
sudo chown root:root /tmp/.X11-unix
sudo chmod 1777 /tmp/.X11-unix
```

---

## Bước 6 — Khởi động VNC server
Ví dụ:
```bash
vncserver :1 -geometry 1366x768 -depth 24 -localhost yes
```
`-localhost yes` giúp VNC chỉ lắng nghe trên loopback (an toàn khi kết nối qua SSH tunnel).

---

## Bước 7 — Tạo SSH tunnel từ Windows và kết nối RealVNC
Trên Windows (PowerShell hoặc terminal SSH):
```powershell
ssh -N -L 5901:127.0.0.1:5901 -p 2222 student@127.0.0.1
```
Sau đó mở **RealVNC Viewer** → nhập `127.0.0.1:5901` → Continue → nhập mật khẩu VNC.

---