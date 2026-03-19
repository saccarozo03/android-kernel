# ERRORS LOG — Android Kernel / AOSP

> Ghi lại mọi lỗi đã gặp. Paste file này cho Claude khi cần debug.

---

## Cách Thêm Lỗi Mới

```
### LỖI #AK[số] — [Tên ngắn gọn]
- **Ngày:** DD/MM/YYYY
- **Lesson:** [số lesson]
- **Loại:** Build Error / Runtime / Flash Error / Boot Loop
- **Bước đang làm:** [repo sync / lunch / make / flash...]

**Lệnh gây lỗi:**
$ [lệnh]

**Output / Error:**
[paste error]

**Nguyên nhân:** ...
**Cách sửa:** ...
**Rule để không mắc lại:** ...
```

---

## Build Errors

_(Chưa có — thêm vào khi gặp!)_

---

## Lunch / envsetup Errors

_(Chưa có)_

---

## Flash / Boot Errors

_(Chưa có)_

---

## Repo Sync Errors

_(Chưa có)_

---

## Checklist Trước Khi Build

- [ ] Đã `source build/envsetup.sh` chưa?
- [ ] Đã `lunch aosp_rpi4_car-userdebug` chưa?
- [ ] Đủ dung lượng disk? (`df -h /mnt/aaos` — cần 400GB+)
- [ ] Đang trong `tmux session` chưa? (tránh mất build khi SSH ngắt)
- [ ] Sửa source file nào? Đã `touch` chưa?
- [ ] **Tắt VM sau khi xong!** (`sudo shutdown -h now`)
