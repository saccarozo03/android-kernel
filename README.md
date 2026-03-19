# Hướng Dẫn Build AOSP Automotive (AAOS) cho Raspberry Pi 4
> **Môi trường:** Google Cloud VM · Ubuntu 22.04 · Android 15

---

## Mục Lục
1. [Tổng quan hệ thống](#1-tổng-quan-hệ-thống)
2. [Chuẩn bị Google Cloud VM](#2-chuẩn-bị-google-cloud-vm)
3. [Cài đặt môi trường build](#3-cài-đặt-môi-trường-build)
4. [Tải AOSP Source Code](#4-tải-aosp-source-code)
5. [Thêm support Raspberry Pi 4](#5-thêm-support-raspberry-pi-4)
6. [Build AOSP](#6-build-aosp)
7. [Đóng gói file Image](#7-đóng-gói-file-image)
8. [Chỉnh sửa Source Code](#8-chỉnh-sửa-source-code)
9. [Quản lý Session và Tiết Kiệm Chi Phí](#9-quản-lý-session-và-tiết-kiệm-chi-phí)
10. [SSH từ Termux (Android)](#10-ssh-từ-termux-android)

---

## 1. Tổng quan hệ thống

### Các khái niệm cần biết

| Khái niệm | Giải thích |
|-----------|------------|
| **AOSP** | Android Open Source Project — mã nguồn gốc của Android |
| **AAOS** | Android Automotive OS — Android chạy trực tiếp trên xe hơi |
| **Raspberry Pi 4** | Board máy tính ARM dùng để giả lập môi trường xe hơi |
| **Google Cloud VM** | Máy chủ ảo thuê theo giờ để build AOSP (máy cá nhân không đủ mạnh) |
| **repo** | Tool của Google để quản lý hàng trăm git repo cùng lúc |
| **ninja** | Build engine của AOSP — chỉ rebuild những gì thay đổi |

### Tại sao cần Google Cloud?
AOSP rất nặng: source ~300GB, cần RAM 32GB+, build lần đầu mất 4–8 tiếng. Máy tính cá nhân thường không đủ tài nguyên.

### Sơ đồ tổng thể

```
Google Cloud VM (Ubuntu 22.04)
    ↓ repo sync (~300GB)
AOSP Source Code + Raspberry Pi 4 device files
    ↓ source build/envsetup.sh && lunch
    ↓ make -j$(nproc)
out/target/product/rpi4_car/*.img
    ↓ rpi4-mkimg.sh
RaspberryVanillaAOSP15-YYYYMMDD-rpi4_car.img
    ↓ flash vào SD card
Raspberry Pi 4 chạy Android Automotive OS 🚗
```

---

## 2. Chuẩn bị Google Cloud VM

### Cấu hình VM khuyến nghị

| Thông số | Giá trị | Lý do |
|----------|---------|-------|
| Machine type | `n2-standard-32` (32 vCPU, 128GB RAM) | AOSP cần ít nhất 32GB RAM |
| OS | Ubuntu 22.04 LTS | Google officially support |
| Boot disk | 500GB SSD | Source ~300GB + build output ~100GB+ |

> ⚠️ **Chi phí:** n2-standard-32 ≈ $1.5/giờ. Tắt VM khi không dùng!

### Tạo và SSH vào VM

```bash
# Từ Google Cloud Console → Compute Engine → VM instances → Create Instance
# Sau khi tạo xong, SSH vào:
gcloud compute ssh YOUR_VM_NAME --zone=YOUR_ZONE
```

---

## 3. Cài đặt môi trường build

### Update hệ thống

```bash
sudo apt update && sudo apt upgrade -y
```

### Cài các dependencies

```bash
sudo apt install -y \
  git-core gnupg flex bison build-essential \
  zip curl zlib1g-dev libc6-dev-i386 \
  libncurses5 lib32ncurses5-dev x11proto-core-dev \
  libx11-dev lib32z1-dev libgl1-mesa-dev \
  libxml2-utils xsltproc unzip fontconfig \
  python3 python-is-python3 \
  openjdk-11-jdk
```

### Cài repo tool

```bash
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Cấu hình Git

```bash
git config --global user.email "email@gmail.com"
git config --global user.name "Ten cua ban"
```

---

## 4. Tải AOSP Source Code

### Tạo thư mục và khởi tạo repo

```bash
mkdir -p /mnt/aaos/android && cd /mnt/aaos/android

# Khởi tạo với Android 15
repo init \
  -u https://android.googlesource.com/platform/manifest \
  -b android-15.0.0_r32 \
  --depth=1
```

> `--depth=1` giúp tải nhanh hơn, không cần toàn bộ git history.

### Thêm local manifest cho Raspberry Pi 4

```bash
mkdir -p .repo/local_manifests

# Tải device manifest cho Pi 4
curl -o .repo/local_manifests/manifest_brcm_rpi.xml \
  -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/manifest_brcm_rpi.xml

# Tải file loại bỏ các project xung đột
curl -o .repo/local_manifests/remove_projects.xml \
  -L https://raw.githubusercontent.com/raspberry-vanilla/android_local_manifest/android-15.0/remove_projects.xml
```

### Sync source code

```bash
# Khuyến nghị: chạy trong tmux (xem mục 9)
repo sync -j$(nproc) --current-branch --no-tags
```

> ☕ Bước này mất 1–3 tiếng. Nên dùng `tmux` để tránh mất tiến độ nếu SSH bị ngắt.

---

## 5. Thêm support Raspberry Pi 4

Raspberry Pi 4 không được AOSP hỗ trợ sẵn. Dự án **raspberry-vanilla** cung cấp device files cần thiết, và đã được tự động tải qua `local_manifests` ở bước trên.

Sau khi `repo sync` xong, kiểm tra:

```bash
ls device/brcm/rpi4/
```

---

## 6. Build AOSP

### Setup môi trường build

```bash
cd /mnt/aaos/android

# Load các function build vào shell session
source build/envsetup.sh
```

> ⚠️ Lệnh `source` phải chạy lại mỗi khi mở terminal mới.

### Chọn target (lunch)

```bash
# aosp_rpi4_car = build cho Pi 4 với giao diện Automotive
# userdebug = bản có thể debug (dùng cho development)
lunch aosp_rpi4_car-userdebug
```

### Build

```bash
# $(nproc) tự động lấy số CPU cores (ví dụ 32 cores → -j32)
make -j$(nproc)
```

> ⏳ Lần đầu build mất 4–8 tiếng. Các lần sau (incremental build) nhanh hơn nhiều.

### Kết quả build thành công

```
#### build completed successfully (47:29 (mm:ss)) ####
```

---

## 7. Đóng gói file Image

### Kiểm tra các file image đã build

```bash
ls -lh out/target/product/rpi4/*.img
```

### Đóng gói thành 1 file .img tổng

```bash
cd /mnt/aaos/android

# Xóa file cũ nếu có
rm -f out/target/product/rpi4/RaspberryVanillaAOSP15-*.img

# Đóng gói
./rpi4-mkimg.sh
```

### Kiểm tra file output

```bash
ls -lh out/target/product/rpi4/RaspberryVanillaAOSP15-*.img
```

### Tải file về máy local

```bash
# Từ máy local có gcloud CLI:
gcloud compute scp \
  YOUR_VM_NAME:/mnt/aaos/android/out/target/product/rpi4/RaspberryVanillaAOSP15-*.img \
  ~/Downloads/ \
  --zone=YOUR_ZONE
```

---

## 8. Chỉnh sửa Source Code

### Hiểu cấu trúc thư mục quan trọng

```
/mnt/aaos/android/
├── system/core/rootdir/init.rc   ← file init chính (Android 12+: nằm trong system image)
├── device/brcm/rpi4/             ← device-specific files cho Pi 4
│   └── aosp_rpi4_car.mk          ← khai báo PRODUCT_COPY_FILES, packages...
└── out/target/product/rpi4/      ← thư mục BUILD OUTPUT (KHÔNG sửa ở đây!)
    ├── system/etc/init/hw/init.rc ← bản copy sau build (do ninja tạo ra)
    └── *.img                      ← các file image
```

> ⚠️ **Quan trọng:** Luôn sửa trong **source gốc**, không bao giờ sửa trong `out/`.  
> Mọi thay đổi trong `out/` sẽ bị **overwrite** khi build lại.

### Ví dụ: Thêm cấu hình ADB qua TCP vào init.rc

```bash
# Mở file đúng — source gốc
nano system/core/rootdir/init.rc
```

Thêm vào block `on property:sys.boot_completed=1`:

```rc
on property:sys.boot_completed=1
    # ...các dòng có sẵn...

    # Thiet lap cong ADB qua TCP
    setprop service.adb.tcp.port 5555
    stop adbd
    start adbd
    write /dev/kmsg "DEBUG: Da thiet lap xong port 5555"
```

### Kiểm tra thay đổi

```bash
grep "5555" system/core/rootdir/init.rc
```

### Rebuild sau khi sửa

```bash
# Android 12+: init.rc nằm trong SYSTEM image, không phải boot image
touch system/core/rootdir/init.rc   # force ninja nhận ra thay đổi
make -j$(nproc)                     # rebuild toàn bộ cho chắc
```

> **Tại sao dùng `touch`?** Ninja dùng timestamp để phát hiện thay đổi. `touch` cập nhật timestamp → ninja biết cần rebuild file này.

> **Tại sao `make -j$(nproc)` thay vì `make bootimage`?**  
> Từ Android 12 trở lên, `init.rc` được đóng gói vào **system partition** (không phải boot partition).  
> `make bootimage` sẽ báo `ninja: no work to do` dù file đã thay đổi.

### Đóng gói lại sau khi rebuild

```bash
rm -f out/target/product/rpi4/RaspberryVanillaAOSP15-*.img
./rpi4-mkimg.sh
```

---

## 9. Quản lý Session và Tiết Kiệm Chi Phí

### Vấn đề

Google Cloud tính tiền theo giờ VM chạy, dù bạn có đang làm việc hay không. Nếu build xong lúc 2am mà không tắt VM → mất tiền oan.

```
VM đang chạy = ~$1.5/giờ
VM đã tắt    = ~$0/giờ (chỉ tốn tiền disk ~$0.04/GB/tháng)
```

### Giải pháp: tmux + tự động tắt VM

**Cài tmux:**

```bash
sudo apt install tmux -y
```

**Workflow hàng ngày:**

```bash
# 1. SSH vào VM
ssh aosp-vm

# 2. Tạo session tmux (như một "căn phòng" trên server)
tmux new -s build

# 3. Chạy lệnh build bình thường bên trong tmux
make -j$(nproc)

# 4. Khi muốn thoát ra (nhấn tổ hợp phím):
#    Ctrl+B, rồi nhấn D   (D = Detach)
#    → Thoát khỏi tmux nhưng giữ session đang chạy

# 5. Quay lại kiểm tra kết quả:
tmux attach -t build
```

### Script tự động tắt VM khi build xong

```bash
nano ~/start_build.sh
```

```bash
#!/bin/bash

echo "=== BAT DAU BUILD AOSP ==="
echo "Thoi gian bat dau: $(date)"

cd /mnt/aaos/android
source build/envsetup.sh
lunch aosp_rpi4_car-userdebug

make -j$(nproc)
BUILD_STATUS=$?

if [ $BUILD_STATUS -eq 0 ]; then
    echo "BUILD THANH CONG luc $(date)"
    echo "VM se tat sau 2 phut..."
    sleep 120
    sudo shutdown -h now
else
    echo "BUILD LOI luc $(date)"
    echo "VM KHONG tat de ban kiem tra loi"
fi
```

```bash
chmod +x ~/start_build.sh
```

**Cách dùng:**

```bash
ssh aosp-vm
tmux new -s build
~/start_build.sh
# Ctrl+B, D để detach
# Đóng Termux, đi ngủ → VM tự tắt khi build xong
```

---

## 10. SSH từ Termux (Android)

Dùng điện thoại Android để điều khiển VM thay vì cần laptop.

### Cài Termux đúng cách

> ⚠️ **Không cài từ Google Play Store** — bản đó đã bỏ rơi từ 2020.

Cài từ **F-Droid**: https://f-droid.org → tìm Termux → cài.

### Cài OpenSSH trong Termux

```bash
pkg update && pkg upgrade -y
pkg install openssh -y
```

### Tạo SSH key

```bash
ssh-keygen -t ed25519 -C "termux-phone"
# Nhấn Enter hết (dùng mặc định)

# Xem public key để copy
cat ~/.ssh/id_ed25519.pub
```

### Thêm SSH key vào Google Cloud VM

Vào **Google Cloud Console → Compute Engine → VM instances → Edit → SSH Keys → Add item** → paste public key → Save.

### Tạo SSH config shortcut

```bash
nano ~/.ssh/config
```

```
Host aosp-vm
    HostName EXTERNAL_IP_CUA_VM
    User ten_user
    IdentityFile ~/.ssh/id_ed25519
```

Từ giờ chỉ cần gõ:

```bash
ssh aosp-vm
```

---

## Tóm Tắt Lệnh Hay Dùng

```bash
# Vào thư mục build
cd /mnt/aaos/android

# Setup môi trường (mỗi lần mở terminal mới)
source build/envsetup.sh && lunch aosp_rpi4_car-ap3a-userdebug

# Build toàn bộ
make -j$(nproc)

# Kiểm tra thay đổi trong init.rc
grep "tu_khoa" system/core/rootdir/init.rc

# Force rebuild sau khi sửa file
touch system/core/rootdir/init.rc && make -j$(nproc)

# Đóng gói image
rm -f out/target/product/rpi4/RaspberryVanillaAOSP15-*.img && ./rpi4-mkimg.sh

# Xem file image đã tạo
ls -lh out/target/product/rpi4/*.img

# tmux: tạo session
tmux new -s build

# tmux: detach (thoát mà không kill)
# Ctrl+B, D

# tmux: quay lại session
tmux attach -t build

# Download file vua build de boot vao usb 
gcloud compute scp saccarozo04@aosp-compiler-15mar:/mnt/aaos/android/out/target/product/rpi4/RaspberryVanillaAOSP15-20260316-rpi4_car.img ./

```

---

*Tài liệu được tổng hợp từ quá trình build AOSP Android 15 cho Raspberry Pi 4*  
*Build environment: Google Cloud VM · n2-standard-32 · Ubuntu 22.04*
