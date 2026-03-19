# ROADMAP — Android Kernel / AOSP Learning Path

> Đánh dấu [x] khi hoàn thành. Ghi ngày để theo dõi tiến độ.

---

## Lesson 1 — AOSP Setup: Build AAOS cho Raspberry Pi 4

> Tài liệu: `lessons/lesson-01-aosp-setup/AOSP_AAOS_RPI4_Huong_Dan.md`

- [ ] Hiểu tổng quan: AOSP, AAOS, RPi4, Google Cloud VM
- [ ] Chuẩn bị Google Cloud VM (n2-standard-32, Ubuntu 22.04)
- [ ] Cài đặt môi trường build (dependencies, repo tool, git)
- [ ] Tải AOSP source code (`repo sync` ~300GB)
- [ ] Thêm support Raspberry Pi 4 (raspberry-vanilla local manifest)
- [ ] Build AOSP lần đầu (`make -j$(nproc)` — 4-8 tiếng)
- [ ] Đóng gói file image (`rpi4-mkimg.sh`)
- [ ] Flash lên SD card và boot
- [ ] Biết cách sửa source code và rebuild
- [ ] Quản lý VM / tmux để tiết kiệm chi phí
- [ ] SSH từ Termux (điện thoại)

---

## Lesson 2 — AOSP Source Code Structure

- [ ] Hiểu cấu trúc thư mục AOSP (system/, device/, frameworks/, packages/...)
- [ ] Cách đọc và tìm kiếm trong AOSP source
- [ ] Android.mk vs Android.bp (Soong build system)
- [ ] Cách thêm 1 module/app đơn giản vào build
- [ ] PRODUCT_COPY_FILES — copy file vào image

---

## Lesson 3 — Android Init System

- [ ] init.rc syntax: actions, services, properties
- [ ] `on property:` trigger
- [ ] Thêm service tùy chỉnh khởi động cùng hệ thống
- [ ] `setprop` / `getprop`
- [ ] Kiểm tra init bằng `adb logcat`

---

## Lesson 4 — Android HAL (Hardware Abstraction Layer)

- [ ] HAL là gì — tại sao cần
- [ ] HIDL vs AIDL
- [ ] Xem HAL có sẵn trên RPi4
- [ ] Implement HAL đơn giản

---

## Lesson 5 — Kernel Module cho Android

- [ ] Viết kernel module chạy trong AOSP
- [ ] Tích hợp module vào build
- [ ] Giao tiếp kernel ↔ user space qua `/dev`

---

## Tiến Độ

| Lesson | Nội dung | % | Bắt đầu | Xong |
|--------|---------|---|---------|------|
| 1 | AOSP Setup — Build AAOS cho RPi4 | 0% | | |
| 2 | AOSP Source Code Structure | 0% | | |
| 3 | Android Init System | 0% | | |
| 4 | Android HAL | 0% | | |
| 5 | Kernel Module cho Android | 0% | | |

---

*Cập nhật % sau mỗi buổi học!*
