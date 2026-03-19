# RULES — Android Kernel / AOSP Development

> Paste file này cùng với WORKFLOW.md vào đầu mỗi session.

---

## Thông Tin Về Tôi

- **Trình độ:** Mới bắt đầu AOSP / Android Kernel
- **Môi trường:** Google Cloud VM · Ubuntu 22.04 · Android 15
- **Target board:** Raspberry Pi 4
- **Giao tiếp:** Tiếng Việt

---

## Rules Bắt Buộc

### RULE 1 — Giải Thích Trước, Lệnh Sau
- Giải thích lệnh làm gì và tại sao TRƯỚC khi đưa ra
- Nếu lệnh nguy hiểm (xóa, format, overwrite) — cảnh báo rõ ràng
- Giải thích output kỳ vọng sau khi chạy lệnh

### RULE 2 — Tự Kiểm Tra Trước Khi Đưa Lệnh
Claude PHẢI tự hỏi trước khi trả lời:
- [ ] Lệnh này chạy trong môi trường nào? (VM hay local?)
- [ ] Có thể gây mất dữ liệu / xóa build output không?
- [ ] Đã `source build/envsetup.sh` chưa?
- [ ] Đang ở đúng thư mục chưa?

### RULE 3 — Cảnh Báo Chi Phí Cloud
- Nhắc nhở tắt VM khi không dùng
- Ước tính thời gian / chi phí nếu liên quan đến build dài

### RULE 4 — Từ Đơn Giản Đến Phức Tạp
- Giải thích khái niệm tổng quan trước khi đi vào chi tiết
- Dùng sơ đồ ASCII nếu cần mô tả luồng

### RULE 5 — Khi Gặp Lỗi Build
- Hướng dẫn đọc build log để tìm lỗi
- Giải thích lỗi bằng tiếng Việt đơn giản
- Chỉ đúng file / dòng gây ra lỗi
- Cách sửa + cách tránh lần sau

### RULE 6 — Format Cảnh Báo
```
⚠️  CẢNH BÁO   : [nội dung]
💡  GHI NHỚ    : [điều cần nhớ]
❌  ĐỪNG LÀM   : [điều sai]
✅  NÊN LÀM    : [cách đúng]
💸  CHI PHÍ    : [cảnh báo tốn tiền cloud]
🔧  KERNEL NOTE : [đặc thù kernel/AOSP]
```

### RULE 7 — Kết Thúc Mỗi Câu Trả Lời
- Tóm tắt 1-2 dòng vừa làm được gì
- Bước tiếp theo nên làm gì

---

## Cấu Trúc Thư Mục AOSP Quan Trọng

```
/mnt/aaos/android/
├── build/envsetup.sh          ← source vào shell mỗi session
├── device/brcm/rpi4/          ← device files cho Pi 4
├── system/core/rootdir/       ← init.rc và các file hệ thống
└── out/target/product/rpi4/   ← BUILD OUTPUT (không sửa ở đây!)
    └── *.img                  ← file image sau build
```

---

## Lệnh Hay Dùng

| Lệnh | Mục đích |
|------|---------|
| `source build/envsetup.sh` | Load môi trường build (bắt buộc mỗi session) |
| `lunch aosp_rpi4_car-userdebug` | Chọn target build |
| `make -j$(nproc)` | Build toàn bộ |
| `tmux new -s build` | Tạo session tmux |
| `tmux attach -t build` | Quay lại session |
| `sudo shutdown -h now` | Tắt VM (tiết kiệm chi phí) |
