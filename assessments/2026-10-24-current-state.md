# Khảo sát hiện trạng MyDmsVn

**Ngày khảo sát:** 24/10/2026. **Loại tài liệu:** đánh giá source tại một mốc cố định. Phương án xử lý nằm trong [mô hình tổ chức](../proposals/2026-10-24-organization.md); trạng thái lựa chọn nằm trong [lộ trình áp dụng](../proposals/2026-10-24-adoption.md).

## 1. Kết luận chuyên gia

Cảm giác nặng nề có cơ sở. Tuy nhiên, nguyên nhân chính quan sát được không phải một framework đơn lẻ hay việc dùng Clean Architecture. Có bốn loại công việc đang bị ghép quá chặt:

1. **Sửa một control nhưng phải quản lý chuỗi pin/PR qua nhiều repo.**
2. **Thử giao diện nhưng project test kéo cả Local adapter, server và SQL fixture.**
3. **Đọc trạng thái hiện tại nhưng phải đối chiếu nhiều tài liệu chứa trạng thái trùng nhau.**
4. **Kiểm tra hành vi hiện hành nhưng một số test lại bảo vệ dấu vết của lần chuyển đổi trước.**

Đây là chẩn đoán từ source và tài liệu, chưa phải kết quả profiling. Hướng phù hợp là giảm số nơi phải thay đổi cho một việc, làm rõ điểm chạy độc lập và nguồn thông tin chính. Việc chia thêm repo chỉ có ích khi phục vụ được các ranh giới đó.

Nền tảng đáng giữ lại gồm API ports phía desktop, DTO không chứa control, các Application handlers, explicit UnitOfWork, transaction protocol của nhập/bán/tồn, và Respawn fixtures. Xóa rồi viết lại chúng đồng thời sẽ tăng rủi ro mà chưa giải quyết nguyên nhân vừa nêu.

## 2. Mốc khảo sát và giới hạn

| Thành phần | Revision được khảo sát | Cách xác định |
|---|---|---|
| MyDmsVn, `v1a-pro-ui-next` | `03acacdaea6a750aa88b33d61ea139379edcb326` | Branch head chứa merge PR #45 |
| BootstrapSourceGrid | `e311144c27cd79a6c34aa0308ad029651c9e03c9` | Gitlink thực tế trong app |
| Bootstrap5WinFormUI | `8ae217ea8aa9a938ebd558f7071dc737f909e6a5` | Gitlink lồng trong BootstrapSourceGrid |
| SourceGrid fork | `f4e457b43582bf01892f50bdc74aa480531e5944` | Gitlink lồng trong BootstrapSourceGrid |

Nguồn: [app tree][S01], [BSG tree][S02], [UI tree][S03], [SourceGrid tree][S04]. Không dùng một SHA chép trong tài liệu cũ thay cho gitlink của baseline.

Upstream `main` đã được đối chiếu riêng. UI `c1eab308d26e0bda133b80b2cdb08c3bfbc35842` và BSG `e7aaa19a39a344eccc6f73074232ebaa9a06fd35` đều có hai merge commits thêm so với pin nhưng cùng file tree; không có cơ sở nói app đang thiếu bản sửa source mới chỉ vì SHA khác. [UI compare][S05], [BSG compare][S06].

Khảo sát tập trung dependency graph đang dùng. Không suy ra rằng mọi repo cũ có tên Persistence hoặc UnitOfWork đều còn là dependency. Mọi nhận định về runtime dưới đây được ghi rõ là tài liệu dự án đã ghi nhận hay là điều nhìn thấy từ code.

**Không chạy build, test, SQL, WinForms hoặc benchmark trong đợt khảo sát này.** Không suy ra production-ready từ tên task VERIFIED. Nội dung và trạng thái mới trên các branch sau mốc này cần được đối chiếu khi bắt đầu triển khai.

## 3. Kiến trúc thực tế

Đây là ứng dụng desktop chạy nghiệp vụ server **trong cùng process**, theo đường gọi WinForms → Desktop.Application → Local adapter → Server.Application → Server.Infrastructure → SQL Server. `Server.*` hiện là tên các tầng, chưa phải một HTTP server được triển khai độc lập. `Desktop.App/Program.cs` đăng ký cả server application, SQL persistence và Local adapter. [Composition root][S07], [Local registration][S08].

`Desktop.WinForms` chỉ tham chiếu `Desktop.Application` và BSG. `Desktop.Application` chỉ tham chiếu Contracts và CommunityToolkit.Mvvm. Các `IProductApiClient`, `ISalesReceiptApiClient`, `IGoodsReceiptApiClient` đã là vị trí thay implementation phù hợp; chưa cần thêm một framework abstraction nữa. [Project WinForms][S09], [project Desktop.Application][S10], [API ports][S11].

Các project đang hoạt động của sản phẩm và mười test project đều là `net48`; AppCore `net8.0-windows` còn nằm trong tree nhưng được park khỏi solution đang dùng. Server.Application, Domain, Infrastructure, SharedKernel và Contracts cũng là `net48`. Vì vậy, thêm API ASP.NET Core hiện đại sẽ cần xử lý target framework và dependencies của backend, không đơn thuần thêm một executable. [Solution][S12], [Server.Application csproj][S13], [Contracts csproj][S14], [runtime record][S15].

Provider hiện hành là LINQ to DB kết hợp typed native SqlClient; DbUp vẫn quản lý schema. Số liệu trong khảo sát này không chứng minh DbUp hoặc LINQ to DB là nguyên nhân thời gian test. Thay provider không phải bước đầu được khuyến nghị. [Package versions][S16], [architecture][S17].

## 4. Các điểm gây tải và ý nghĩa

### 4.1 Chuỗi thư viện UI tạo thêm việc phát hành

Trong bản sửa DateInput, UI sửa focus highlight; BSG chỉ cập nhật pin UI, không sửa adapter hoặc SourceGrid; app tiếp tục cập nhật pin BSG. Evidence ghi nhận chuỗi UI #73 → BSG #18 → app #45. Một lỗi control vì vậy có ba nơi cần đối chiếu source/trạng thái. [DateInput correction evidence][S18].

**Suy luận:** UI và adapter có quan hệ thay đổi đủ gần để cân nhắc gom chung repo. Vẫn giữ hai assembly và dependency một chiều. SourceGrid là engine/fork riêng, chưa có bằng chứng cần gộp source engine vào toolkit.

Chuyển sang package không phải thao tác tức thời đã sẵn sàng. Tài liệu release của BSG tại pin được khảo sát còn ghi điều kiện về package/source equivalence và license của UI dependency. Đây là trạng thái trong tài liệu dự án; availability trên feed hiện tại chưa được kiểm tra. [Release policy][S19]. Đề xuất phải giữ được đường source ProjectReference trong giai đoạn đầu, rồi xác minh điều kiện package trước khi phát hành.

### 4.2 Test UI gắn với graph rộng hơn nhu cầu

`MyDmsVn.Desktop.Tests.csproj` link hai SQL fixture files và tham chiếu cả Local, Server.Application, Server.Infrastructure. Điều đó không có nghĩa mỗi test đều mở SQL, nhưng build/restore và phân loại công việc đang rộng hơn nhiều test thuần presentation. [Desktop test project][S20].

Có fake API và SalesControlHost dùng control thật. Tuy nhiên, `SalesGuiApi.Calculator()` lấy preview calculator bằng `AddLocalDesktopAdapter()`, còn implementation gọi `Server.Application.Sales.SalesReceiptPreview`. UI giả lập hiện có vì vậy chưa hoàn toàn độc lập với server graph. [Fake calculator][S21], [Local preview][S22].

**Hướng xử lý:** tách composition root UI Lab, tái sử dụng ports, di chuyển đúng các ca pure logic, giữ hosted/native checks ở nhóm Windows và package/SQL checks ở nhóm integration. Mock preview dùng dữ liệu kỳ vọng cố định; nếu cần chia sẻ arithmetic thật thì phải có thiết kế nhỏ cùng tests riêng, không copy thuật toán tính giá vốn vào UI.

SourceGrid hiện tham chiếu cả SqlClient và Windows.Compatibility. Một UI Lab không cần connection string vẫn có thể mang assembly liên quan SQL qua vendor. Cần phân biệt **không kết nối database** với **không tồn tại assembly SqlClient trong toàn bộ closure**; chưa có bằng chứng các dependency engine này không được sử dụng. [SourceGrid csproj][S23].

### 4.3 Trạng thái hiện tại mâu thuẫn ngay trong cùng commit

| Điểm đọc vào | Nội dung tại baseline |
|---|---|
| README, AGENTS, docs index, TEST_POLICY, TEST_STRATEGY, tests/README | DateInput UIN-D01–D03 còn NOT STARTED / NOT RUN |
| AI_CONTEXT, active roadmap, correction plan/evidence | UIN-D01–D03 VERIFIED trong phạm vi được ghi nhận |
| docs/ui-next/README | Còn ghi toàn bộ implementation NOT STARTED / runtime NOT RUN |

Đối chiếu trực tiếp: [README][S24], [AGENTS][S25], [AI_CONTEXT][S26], [roadmap][S27], [UI-NEXT index][S28], [test policy][S29]. Đó là lỗi đồng bộ thông tin có thể khiến Codex lập lại việc đã làm hoặc chọn nhầm điểm xuất phát.

Các pin cũ trong AI_CONTEXT được ghi rõ là *inspected correction baseline*, và roadmap gọi chúng là *initial inspection anchors*. Không nên sửa mất lịch sử hoặc kết luận chúng là pin sai. Vấn đề là người đọc phải tự tìm đâu là pin hiện tại giữa nhiều bối cảnh khác nhau.

**Hướng xử lý:** một trang trạng thái ngắn, một nguồn pin từ source; tài liệu kiến trúc và test policy không chép tiến độ từng task. Evidence giữ nguyên tính lịch sử. Việc source đổi không buộc sửa hàng loạt đoạn tường thuật trạng thái.

### 4.4 Đã có archive và test policy tốt; vấn đề còn lại là thực thi nhất quán

`docs/plans/` hiện chỉ còn README chuyển hướng. Sáu plan cũ đã chuyển vào archive; không có nhiều active plan ở đó để lại phải archive một lần nữa. Test policy đã yêu cầu chạy theo phạm vi thay đổi, không chạy full solution/full SQL/full GUI sau mọi chỉnh sửa. [Plan redirect][S30], [test selection matrix][S29].

Vì vậy, đề xuất mới không khôi phục gate manifest, SqlTestHarness, SQL runner, approval files, hay yêu cầu chạy lại mọi acceptance cũ. Cần làm đường đi hiện hành ngắn và rõ hơn.

## 5. Số liệu định lượng: kích thước, không phải chất lượng

Đếm Git tree đầy đủ, `truncated=false`, không recurse nội dung submodule vào repo cha:

| Chỉ số | Giá trị | Ý nghĩa / giới hạn |
|---|---:|---|
| Tracked blobs trong app | 951 | Bao gồm code, tests, ảnh, docs, lock files |
| Files dưới docs/ | 401 | 113 Markdown, 282 ảnh, 6 files khác |
| Files dưới docs/ui-next/ | 30 | 29 Markdown và 1 PNG |
| Files dưới docs/plans/ | 1 | README chuyển hướng |
| Test projects | 10 | Không phải 10 suite có cùng dependency hoặc chi phí |
| C# files dưới tests/ | 224 | Bao gồm helpers/probes, không phải số test cases |
| Bảy entry guides thường đọc | 101.787 bytes | Tổng kích thước tracked của README, AGENTS, AI_CONTEXT, docs index, TEST_POLICY, TEST_STRATEGY, tests/README |
| PNG trùng nội dung | 10 cặp / 20 đường dẫn | Khoảng 1.054.156 bytes có thể giảm trong working tree; Git đã dùng chung blob giống nhau |

Không phát hiện Markdown trùng hoàn toàn theo blob SHA; vấn đề là lặp ý và trạng thái, không phải các file Markdown giống hệt. Không có `.github/` hoặc YAML được track trong tree khảo sát; **chưa xác minh hệ thống CI ngoài repo**. Không có số đo thời gian mới để khẳng định mức tăng tốc.

## 6. Những test cần xem lại cụ thể

| Nhóm | Ví dụ có thể hành động | Cách làm đúng |
|---|---|---|
| Test dấu vết tài liệu | PermissionRemovalInventoryTests kiểm tên runbook cũ/mới | Bỏ ràng buộc đường dẫn tài liệu khi dọn docs; giữ regression permission thực |
| Test dấu vết di chuyển | DesktopSessionProjectContractTests liệt kê cứng 17 file từng được chuyển | Thay bằng invariant dependency/target có ích; không đóng băng danh sách của một đợt migration |
| Token scan trong C# | OperationalPersistenceBoundaryTests dùng regex/string checks | Giữ boundary có ý nghĩa; thay kiểm chuỗi dễ sai bằng kiểm reference/public API phù hợp |
| GUI smoke oracle yếu | UiDependencyCompatibilityTests đổi theme nhưng chủ yếu assert theme global và grid chưa disposed | Tăng assertion lên observable thực, hoặc bỏ ca trùng sau khi có coverage thay thế |

Nguồn: [permission inventory][S31], [session inventory][S32], [operational boundary][S33], [UI theme check][S34]. Đây là **ứng viên review**, chưa phải danh sách cho phép xóa tự động.

Ngược lại, tên `InventoryLinqMigrationTests` hoặc `SecurityNativeMigrationTests` không làm test hết giá trị: chúng còn kiểm SQL mappings, decimal/Unicode, legacy credentials và atomic replacement. Nên đổi tên theo hành vi hiện tại nếu hữu ích; không xóa vì chữ Migration. [Inventory cases][S35], [security cases][S36].

## 7. Điều cần bảo toàn khi tổ chức lại

Giữ các quy tắc đã hiện hữu: giá bán nhập tay, dòng sản phẩm lặp độc lập, Tặng có doanh thu bằng không nhưng vẫn có lượng/giá vốn, bình quân gia quyền liên hoàn theo Kho+Sản phẩm, giá vốn chốt khi Post, và Sales keyset pagination. Thay folder/repo/transport không làm thay đổi nghiệp vụ. [Sales contract][S37].

Giữ ranh giới quyền hiện tại, transaction ownership, dòng ledger bất biến, no-double-post, rowversion và trạng thái kết quả Post chưa xác định. Không coi mock 403 là bằng chứng server chặn quyền, không coi web screenshot là bằng chứng caret/IME WinForms, không chuyển một test skipped thành PASS. [Contracts][S38], [DateInput native evidence][S18].

Đề xuất tiếp theo xuất phát từ những điểm này: [tổ chức code/repo](../proposals/2026-10-24-organization.md), [prototype](../proposals/2026-10-24-ui-prototyping.md), [docs/tests/skills](../proposals/2026-10-24-docs-tests-skills.md).

## Nguồn source cố định

[S01]: https://github.com/chung6a8m/MyDmsVn/tree/03acacdaea6a750aa88b33d61ea139379edcb326
[S02]: https://github.com/chung6a8m/MyDmsVn.BootstrapSourceGrid/tree/e311144c27cd79a6c34aa0308ad029651c9e03c9
[S03]: https://github.com/chung6a8m/MyDmsVn.Bootstrap5WinFormUI/tree/8ae217ea8aa9a938ebd558f7071dc737f909e6a5
[S04]: https://github.com/chung6a8m/sourcegrid/tree/f4e457b43582bf01892f50bdc74aa480531e5944
[S05]: https://github.com/chung6a8m/MyDmsVn.Bootstrap5WinFormUI/compare/8ae217ea8aa9a938ebd558f7071dc737f909e6a5...c1eab308d26e0bda133b80b2cdb08c3bfbc35842
[S06]: https://github.com/chung6a8m/MyDmsVn.BootstrapSourceGrid/compare/e311144c27cd79a6c34aa0308ad029651c9e03c9...e7aaa19a39a344eccc6f73074232ebaa9a06fd35
[S07]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.App/Program.cs#L63-L77
[S08]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Infrastructure.Local/DependencyInjection.cs#L11-L46
[S09]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.WinForms/MyDmsVn.Desktop.WinForms.csproj#L1-L11
[S10]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Application/MyDmsVn.Desktop.Application.csproj#L1-L11
[S11]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Application/FeatureApiClients.cs#L9-L29
[S12]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/MyDmsVn.sln
[S13]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Server.Application/MyDmsVn.Server.Application.csproj
[S14]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Contracts/MyDmsVn.Contracts.csproj
[S15]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/TECH_STACK.md
[S16]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/Directory.Packages.props
[S17]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ARCHITECTURE.md
[S18]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/evidence/20261024-dateinput-unified-compact-correction.md#L38-L62
[S19]: https://github.com/chung6a8m/MyDmsVn.BootstrapSourceGrid/blob/e311144c27cd79a6c34aa0308ad029651c9e03c9/docs/RELEASE.md#L3-L11
[S20]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/MyDmsVn.Desktop.Tests.csproj#L8-L34
[S21]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/SalesReceiptGridTests.cs#L327-L352
[S22]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Infrastructure.Local/LocalSalesReceiptPreviewCalculator.cs
[S23]: https://github.com/chung6a8m/sourcegrid/blob/f4e457b43582bf01892f50bdc74aa480531e5944/SourceGrid/SourceGrid.csproj#L114-L119
[S24]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/README.md#L7
[S25]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/AGENTS.md#L9
[S26]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/AI_CONTEXT.md#L9-L37
[S27]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/20261023-active-roadmap.md#L4-L22
[S28]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/README.md#L3-L5
[S29]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/TEST_POLICY.md#L5-L30
[S30]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/plans/README.md#L6-L18
[S31]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Architecture.Tests/PermissionRemovalInventoryTests.cs#L32-L40
[S32]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Session.Tests/DesktopSessionProjectContractTests.cs#L11-L56
[S33]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Architecture.Tests/OperationalPersistenceBoundaryTests.cs#L17-L46
[S34]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/UiDependencyCompatibilityTests.cs#L11-L36
[S35]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.Inventory.Tests/InventoryLinqMigrationTests.cs#L22-L49
[S36]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.Security.Tests/SecurityNativeMigrationTests.cs#L23-L46
[S37]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/DIRECT_SALES.md
[S38]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/CONTRACTS.md
