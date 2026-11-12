# Kế hoạch phân chia công việc — Nhóm top1build (Track T4: Thương mại điện tử)

> **Lát cắt:** Trong Trò chuyện với Shopee, khi người mua báo "hàng nhận không đúng mô tả", AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên.

---

## 1. Bảng phân công chi tiết theo 3 thành viên

| Thành viên | Vai trò phụ trách | Đầu mục công việc cụ thể | Phụ trách trong `spec.md` |
| :--- | :--- | :--- | :--- |
| **Thành viên 1** | **Product & Risk Lead** | - Xác định 3 việc tính năng **KHÔNG làm** (kèm lý do).<br>- Lập bảng **≥8 kịch bản rủi ro** (phủ 4 lớp ①②③④).<br>- Chọn & biện luận **2 kịch bản đắt nhất** + 1 ca suýt chọn.<br>- Định nghĩa 3 tiêu chí Pass/Fail chấm được. | **§1. Lát cắt & phạm vi**<br>**§2. Kịch bản rủi ro**<br>**§6. Tốt là gì** |
| **Thành viên 2** | **UX & AI Safety Lead** | - Thiết kế xử lý khi AI sai cho 2 kịch bản đắt nhất (câu thoại nguyên văn, fallback, escalation).<br>- Phân loại lỗi (hệ thống / người dùng / ngữ cảnh).<br>- Áp dụng 2–3 nguyên tắc **HAX / PAIR** (chỉ rõ câu từ, vị trí UI).<br>- Thiết kế giải pháp kỹ thuật/UI cho lỗi "không thể cứu bằng câu chữ". | **§3. Khi AI sai**<br>**Ghi chú HAX/PAIR cho §5** |
| **Thành viên 3** | **System & Prototype Lead** | - Phân loại mức tự chủ của từng tool (Tự làm / Cần duyệt / Cấm) theo Cost-of-Error.<br>- Giải quyết tool gây tranh cãi nhất & test trên 3 kịch bản.<br>- Dựng **Prototype** (file HTML chạy được trong thư mục `prototype/`).<br>- Chụp 1–2 ảnh đánh số, đối chiếu với HAX/PAIR và mức tự chủ. | **§4. Mức tự chủ**<br>**§5. Prototype** |

---

## 2. Kế hoạch thời gian thực hiện (Timeline)

```
[00:00 - 00:25] SYNC TOÀN NHÓM
│ ├── Chốt 3 việc KHÔNG làm (§1)
│ └── Chốt danh sách sơ bộ 8 kịch bản rủi ro & 2 kịch bản đắt nhất (§2)
▼
[00:25 - 01:15] LÀM VIỆC ĐỘC LẬP THEO PHÂN CÔNG
│ ├── TV1: Hoàn thiện bảng §1, §2 và §6 vào spec.md
│ ├── TV2: Viết chi tiết §3 (lời thoại nguyên văn, HAX/PAIR, loại lỗi)
│ └── TV3: Viết §4 (ma trận mức tự chủ tool) & code file prototype/index.html
▼
[01:15 - 01:45] TÍCH HỢP & VERIFY
│ ├── TV3: Chụp ảnh prototype, đánh số gắn vào §5
│ ├── TV2 + TV1: Review chéo câu chữ, kịch bản, và logic tool
│ └── Điền thông tin bảng thành viên ở đầu spec.md
▼
[01:45 - 02:00] TỔNG DUYỆT & CHUẨN BỊ DEMO (§7)
  └── Thử nghiệm ca dị (edge case) để chuẩn bị cho phần demo trực tiếp.
```

---

## 3. Checklist nghiệm thu bài nộp

- [ ] Bảng thông tin thành viên ở đầu file `spec.md` đã điền đủ họ tên, mã học viên.
- [ ] §1 có đủ 3 việc KHÔNG làm, mỗi việc có 1 câu giải thích lý do.
- [ ] §2 có đủ ≥8 kịch bản, đủ 4 lớp ①②③④, có 2 kịch bản đắt nhất và 1 kịch bản suýt chọn.
- [ ] §3 có câu thoại nguyên văn, có chỉ định nguyên tắc HAX/PAIR cụ thể.
- [ ] §4 đã phân loại mức tự chủ cho tất cả các Tool liên quan và có phân tích Cost-of-Error.
- [ ] §5 có file trong thư mục `prototype/` và 1–2 ảnh chụp màn hình có đánh số.
- [ ] §6 có 3 tiêu chí Pass/Fail rõ ràng.

