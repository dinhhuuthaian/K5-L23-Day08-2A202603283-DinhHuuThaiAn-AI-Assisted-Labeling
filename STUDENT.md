# Thông tin học viên

- Họ và tên: Đinh Hữu Thái An
- MSHV: 2A202603283
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: dinhhuuthaian
- Repo bài nộp (URL): https://github.com/dinhhuuthaian/K5-L23-Day08-2A202603283-DinhHuuThaiAn-AI-Assisted-Labeling
- Commit chốt bài (hash): 97074aa
- Máy: Windows 10 (build 19045), Python 3.12, không có GPU (chạy CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 454 / 41 / 221 (mẫu số: 729 box máy; 13 box nằm trong ignore_region không tính deleted); added: 49 (mẫu số: 544 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 64.2% / 89.6% (mẫu số: 67 box máy và 48 đối tượng gold, 4 ảnh)
- Bắt bẫy: 11/12 = 91.7%
- Thời gian: gán tay 6.25 phút/ảnh (25 phút / 4 ảnh); duyệt có pre-label 2.5 phút/ảnh (60 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/6), 0.0% theo đối tượng (0/92)
- **Quyết định:** Release — lý do một dòng: 0/6 ảnh lỗi (≤ 2%), bắt bẫy 11/12 = 91.7% (≥ 80%), không có lỗi hệ thống; mẫu 6 ảnh chỉ chứng minh được cận trên 39%, chưa chứng minh được ngưỡng 2%.
