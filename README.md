# MyDmsVn.Docs

Bộ phân tích và đề xuất tổ chức phát triển MyDmsVn, lập ngày **24/10/2026**, theo nhánh `v1a-pro-ui-next` tại commit [`03acacd`](https://github.com/chung6a8m/MyDmsVn/commit/03acacdaea6a750aa88b33d61ea139379edcb326) và các dependency được pin thực tế.

## Đọc theo câu hỏi cần giải quyết

| Câu hỏi | Tài liệu |
|---|---|
| Hiện trạng đang nặng ở đâu? Bằng chứng nào? | [Khảo sát hiện trạng](assessments/2026-10-24-current-state.md) |
| Nên gom/tách repo, server và các tầng như thế nào? | [Mô hình tổ chức đề xuất](proposals/2026-10-24-organization.md) |
| Làm sao phác họa nhanh, thử UI bằng mock và chuẩn bị Web? | [UI Lab và prototype Web](proposals/2026-10-24-ui-prototyping.md) |
| Giữ docs, tests và skills gọn mà đủ tin cậy? | [Docs, tests và Codex skills](proposals/2026-10-24-docs-tests-skills.md) |
| Chọn phương án nào, triển khai theo thứ tự nào? | [Lựa chọn và lộ trình áp dụng](proposals/2026-10-24-adoption.md) |
| Cụ thể triển khai A0–A1 bằng những PR, project và kiểm tra nào? | [Hướng dẫn triển khai A0–A1](proposals/2026-10-24-a0-a1-implementation.md) |

**Đề xuất chính:** một repo sản phẩm với các điểm chạy/build/test độc lập; gom hai thư viện WinForms thường thay đổi cùng nhau vào một repo toolkit nhưng giữ ranh giới assembly; một nơi sở hữu mỗi loại tài liệu; UI Lab native và prototype Web dùng chung ý nghĩa kịch bản; kiểm thử theo hành vi/rủi ro; sáu skills nhỏ gắn với repo thực thi.

## Cách hiểu bộ tài liệu

- `assessments/` ghi nhận source và bằng chứng ở một commit xác định. Đây không phải kết quả chạy lại Windows, SQL hay GUI.
- `proposals/` là phương án để Owner lựa chọn. [Bảng quyết định](proposals/2026-10-24-adoption.md#1-lựa-chọn) là nơi duy nhất ghi trạng thái lựa chọn của bộ đề xuất này.
- Tên repo, project, đường dẫn tương lai và các skill trong đề xuất chưa có nghĩa chúng đã được tạo hoặc hoạt động.
- Bộ tài liệu này không thay thế roadmap UI-NEXT của repo sản phẩm. Quyết định tái tổ chức cần được ghi nhận trước khi chuyển thành công việc triển khai.

Sau khi lựa chọn, chỉ phần được duyệt mới trở thành quy chuẩn hiện hành. Tài liệu kỹ thuật đi cùng source, lệnh test và trạng thái thực thi vẫn do repo sở hữu code quản lý; không sao chép toàn bộ chúng sang đây.

## Đóng góp

Đọc [AGENTS.md](AGENTS.md). Giữ một nguồn cho mỗi sự thật, dùng link commit cố định cho nhận định về code và phân biệt rõ **đã quan sát**, **suy luận**, **đề xuất**, **đã được duyệt**.
