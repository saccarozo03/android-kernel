# QUICK REFERENCE — Android Kernel / AOSP

> Cheat sheet tra nhanh. Paste cho Claude khi hỏi về lệnh cụ thể.

---

## Lệnh Build Hay Dùng

```bash
# Vào thư mục build
cd /mnt/aaos/android

# Setup môi trường (BẮT BUỘC mỗi lần mở terminal mới)
source build/envsetup.sh && lunch aosp_rpi4_car-userdebug

# Build toàn bộ
make -j$(nproc)

# Force rebuild 1 file sau khi sửa
touch path/to/file && make -j$(nproc)

# Đóng gói image
rm -f out/target/product/rpi4/RaspberryVanillaAOSP15-*.img
./rpi4-mkimg.sh

# Xem file image
ls -lh out/target/product/rpi4/*.img

# Kiểm tra disk
df -h /mnt/aaos
```

---

## tmux (Giữ Session Khi SSH Ngắt)

```bash
tmux new -s build        # Tạo session tên "build"
tmux attach -t build     # Quay lại session
tmux ls                  # Xem danh sách session

# Ctrl+B, D → Detach (thoát mà giữ session chạy)
# Ctrl+B, [  → Scroll mode (xem output dài)
# q          → Thoát scroll mode
```

---

## repo Commands

```bash
repo sync -j$(nproc) --current-branch --no-tags   # Sync source
repo status                                         # Xem file đã sửa
repo diff                                           # Xem diff
```

---

## ADB (Debug Trên RPi4)

```bash
adb connect <IP_RPI4>:5555   # Kết nối ADB qua TCP
adb logcat                   # Xem log realtime
adb shell                    # Shell vào thiết bị
adb push file /sdcard/       # Đẩy file lên thiết bị
```

---

## Quản Lý VM

```bash
# Tắt VM (tiết kiệm tiền)
sudo shutdown -h now

# Xem tài nguyên đang dùng
htop
df -h
free -h
```

---

## Cấu Trúc Thư Mục AOSP

```
/mnt/aaos/android/
├── build/envsetup.sh           ← source vào shell mỗi session
├── device/brcm/rpi4/           ← device files RPi4
│   └── aosp_rpi4_car.mk        ← khai báo packages, copy files
├── system/core/rootdir/init.rc ← file init chính (sửa ở đây)
├── frameworks/                 ← Android framework
├── packages/                   ← apps hệ thống
└── out/target/product/rpi4/    ← BUILD OUTPUT (không sửa!)
    ├── system/etc/init/hw/     ← bản copy của init.rc sau build
    └── *.img                   ← các partition image
```

---

> 💸 **Nhắc nhở:** VM n2-standard-32 ≈ $1.5/giờ — TẮT VM KHI KHÔNG DÙNG!
