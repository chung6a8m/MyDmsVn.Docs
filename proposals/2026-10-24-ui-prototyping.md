# Đề xuất UI Lab và prototype Web

**Trạng thái: PROPOSED — phương án để Owner xem xét, chưa phải kế hoạch triển khai đã được duyệt.**

**Ngày:** 2026-10-24. **Baseline:** MyDmsVn tại commit 03acacdaea6a750aa88b33d61ea139379edcb326 trên nhánh v1a-pro-ui-next, sau khi PR #45 được merge.

Tài liệu đề xuất hai môi trường để thiết kế và kiểm tra giao diện nhanh hơn: một UI Lab chạy WinForms thật, và một prototype Web có phạm vi rõ ràng. Tài liệu không triển khai control, mock service, API, website hay skill.

Đọc cùng [hiện trạng](../assessments/2026-10-24-current-state.md), [tổ chức repository](2026-10-24-organization.md) và [lộ trình áp dụng](2026-10-24-adoption.md). Phương án chung là một product monorepo và một WinFormsKit riêng hợp nhất UI library với adapter; tài liệu này chỉ bàn vòng làm việc UI.

## 1. Kết quả cần đạt

Người sửa màn hình có thể mở đúng trạng thái cần sửa, nhập liệu, nhìn lỗi và reset dữ liệu mà không cần khởi động SQL hoặc toàn bộ ứng dụng. Người quyết định UX có thể xem một prototype Web dễ chỉnh để so sánh bố cục và luồng thao tác.

Hai môi trường dùng chung ý nghĩa của kịch bản và dữ liệu mẫu; mỗi môi trường vẫn phải kiểm chứng hành vi của runtime mình.

| Môi trường đề xuất | Vai trò chính | Bằng chứng tạo ra |
|---|---|---|
| UI Lab native | Chạy control/viewmodel WinForms thật với fake ports | Layout, nhập liệu native, focus, caret, command state |
| Prototype Web | Thử bố cục, điều hướng và luồng UX trong trình duyệt | Quyết định thiết kế và hành vi browser trong phạm vi đã dựng |

**Ưu tiên UI Lab native trước**, vì sản phẩm đang giao là WinForms net48. Prototype Web bổ sung giá trị khi có câu hỏi thiết kế hoặc một lát cắt Web cụ thể; không cần dựng lại toàn bộ sản phẩm để có bản xem thử.

Mục tiêu là rút ngắn vòng sửa và phát hiện lỗi sớm. Không suy ra độ nhanh, chất lượng hoặc khả năng chuyển đổi chỉ từ việc tách project hay thêm công cụ.

## 2. Những điểm có thể tái sử dụng ngay trong source

Desktop.Application đã chỉ tham chiếu Contracts và CommunityToolkit.Mvvm; Desktop.WinForms tham chiếu Desktop.Application cùng BootstrapSourceGrid. Đây là điểm tách hợp lý để thay adapter bằng fake khi dựng host thử nghiệm. [E1][E2]

Các interface như ISalesReceiptApiClient trả DTO trong Contracts, có CancellationToken và ranh giới session. IUiDispatcher đã có ImmediateUiDispatcher cho kiểm tra xác định được. Không cần tạo một hệ interface mới bao quanh mọi control. [E3][E4]

CatalogPresentationClient đã có dữ liệu mẫu, trạng thái lỗi và barrier trì hoãn. SalesControlHost đã dựng SalesReceiptControl thật bằng fake API/session và ImmediateUiDispatcher. Đây là vật liệu khởi đầu, chưa phải một UI Lab độc lập hoàn chỉnh. [E5][E6]

Có một dependency cần xử lý rõ: SalesGuiApi.Calculator() lấy preview calculator qua AddLocalDesktopAdapter(); implementation Local gọi Server.Application.Sales.SalesReceiptPreview. Vì vậy harness hiện tại vẫn kéo server graph cho preview calculation. [E7][E8]

Desktop.Tests hiện cũng reference Local, Server.Application, Server.Infrastructure và link SQL fixture files. Chọn một test filter không loại bỏ dependency ở cấp project. Việc tách vòng UI nhẹ cần đi cùng thay đổi project references trong một đợt triển khai được duyệt. [E9]

| Thành phần sẵn có | Cách tận dụng đề xuất |
|---|---|
| Ports và Contracts DTOs | Dựng composition root riêng cho Lab |
| CatalogPresentationClient, SalesGuiApi | Chắt lọc thành scenario fixtures có owner rõ |
| SalesControlHost | Mẫu hosted screen; giảm reflection khi có seam phù hợp |
| NativeLookupTypingDriver | Giữ kiểm tra ký tự, caret, selection, focus |
| Preview calculator interface | Fixture xác định được cho UI; kiểm tra arithmetic riêng |

Không chuyển toàn bộ test helpers thành public production API chỉ để Lab gọi được. Chọn seam nhỏ ở composition, constructor hoặc factory khi triển khai.

## 3. UI Lab native: chạy màn hình thật với dữ liệu kiểm soát được

Lab được đề xuất là executable phát triển riêng trong product monorepo. Nó dùng Desktop.Application, Desktop.WinForms và toolkit đã pin; không gọi production bootstrap hay cần thông tin kết nối SQL. Lab và fake composition không nằm trong package production; không thêm cờ bỏ qua Login vào executable đang giao hàng để tiện demo.

Giữ target net48 cho lát cắt đầu tiên. Việc tổ chức Lab không đồng thời kích hoạt AppCore, đổi runtime đang giao hoặc mở lại kế hoạch nâng .NET.

Lab có danh sách màn hình, chọn scenario, reset scenario và hiển thị tên scenario đang chạy. Cần phân biệt dữ liệu giả ngay trong host thử nghiệm, tránh nhầm với phiên ứng dụng thật.

Composition root của Lab cung cấp fake ports, session thử nghiệm, clock/delay kiểm soát được và các services trình bày cần thiết. Form/control nghiệp vụ tiếp tục dùng dependency injection hiện có.

Các khả năng tối thiểu:

- Mở Sales list/editor của pilot ở trạng thái đã biết. Catalog và Goods Receipt là phần mở rộng sau pilot, không phải điều kiện hoàn thành bước A1.
- Chọn empty, loaded, loading, error, forbidden, conflict hoặc recovery.
- Trì hoãn response rồi chủ động hoàn tất để kiểm tra race và stale callback.
- Reset về cùng dữ liệu đầu vào, không phụ thuộc thứ tự chạy trước đó.
- Chuyển theme/font trong phạm vi sản phẩm hỗ trợ.
- Giữ thao tác nhập và validation trên control thật.

Generic ControlGallery thuộc WinFormsKit; màn hình Sales/Goods/Catalog và fixtures nghiệp vụ thuộc product monorepo. Không đưa receipt DTO hoặc permission policy vào thư viện control dùng chung.

### Preview calculation trong Lab

Phương án đầu tiên là cung cấp preview fixture cho các giá trị đã định trước để kiểm tra tổng tiền hiển thị, lỗi và trạng thái command. Ghi rõ fixture này không xác nhận thuật toán tính toán.

Nếu cần tái sử dụng calculation thật ở nhiều runtime .NET, đánh giá riêng một thư viện pure calculation nhỏ và các tests precision/rounding/overflow. Không kéo Server.Application vào Lab chỉ để duy trì cảm giác “mock nhưng giống production”.

Kết quả Save/Post, giá vốn và tổng tiền có thẩm quyền vẫn đến từ server. Không thay quy tắc nghiệp vụ hiện hữu trong công việc dựng Lab.

## 4. Prototype Web: một lát cắt dễ sửa và dễ bỏ

Đề xuất mặc định là React + TypeScript + Vite cho một prototype có thể chạy độc lập. Đây là lựa chọn triển khai tương lai, chưa phải dependency đã được thêm hay stack bắt buộc của sản phẩm cuối.

Bắt đầu với một lát cắt: Sales list → mở Draft → sửa vài dòng → nhận validation hoặc Save result giả lập. Chọn Catalog nếu mục tiêu trước mắt chỉ là chốt mật độ danh sách và inspector.

Prototype cần thể hiện navigation, hierarchy, empty/loading/error states, validation và phản hồi của các hành động chính. Không cần authentication thật, database hoặc một bản sao server.

Storybook chỉ hữu ích khi cần xem nhiều biến thể component hoặc duyệt trạng thái độc lập. MSW chỉ hữu ích khi muốn giữ cách gọi HTTP của frontend trong lúc thay response bằng fixture. Có thể bắt đầu bằng fixture provider trong bộ nhớ.

Không bắt buộc đưa cả Storybook, MSW, state manager, design system và end-to-end framework vào lát cắt đầu tiên. Mỗi công cụ cần giải quyết một khó khăn đã gặp.

Prototype Web không quyết định tự động cấu trúc API production. Nó cần một contract draft đủ cho lát cắt; các quyết định transport còn mở phải được ghi lại.

Mỗi bản xem thử ghi rõ scenario, phần tương tác đã dựng và phần là minh họa. Một nút giả lập thành công không chứng minh giao dịch hoặc phân quyền ở server.

## 5. Tái sử dụng contracts, fixtures và case IDs

Chia sẻ các khái niệm ổn định trước khi chia sẻ implementation. Nguồn dữ liệu mẫu nên nằm ở product monorepo, có phiên bản và liên kết tới contract/scenario liên quan.

Đề xuất một danh mục scenario gồm mã, mục tiêu, trạng thái ban đầu, hành động, response sequence và kỳ vọng quan sát được. JSON có thể dùng khi thuận tiện, nhưng schema fixture cụ thể cần được chọn trong thiết kế triển khai.

| Nội dung | Cách chia sẻ đề xuất |
|---|---|
| Product/customer/warehouse mẫu | Dữ liệu không nhạy cảm, seed ổn định |
| Draft/Posted/conflict responses | Cùng ý nghĩa và ID nghiệp vụ |
| Error codes và field/cell mapping | Cùng case ID, renderer-specific binding |
| Delay/cancellation/stale responses | Cùng chuỗi sự kiện, driver riêng |
| Visual roles | Cùng ý nghĩa; mapping WinForms/CSS riêng |
| Raw numeric/date input | Cùng chuỗi đầu vào và expected state |

Không ép mọi fixture qua cùng serializer ngay từ đầu. Adapter .NET và frontend có thể đọc cùng dữ liệu qua loader riêng nếu điều đó đơn giản và rõ ràng hơn.

Khi có HTTP API thật, thêm contract tests để phát hiện fixture lệch wire format. Không coi việc frontend và fake cùng đọc một file là bằng chứng server thực hiện đúng contract.

## 6. Bộ scenario ưu tiên

Các mã dưới đây là đề xuất; chưa tồn tại như một bộ test hay chứng nhận acceptance mới.

| ID đề xuất | Tình huống | Kỳ vọng cần giữ |
|---|---|---|
| UI-SALES-01 | Mở Draft có sản phẩm trùng ở nhiều dòng | Giữ identity từng dòng và các giá trị độc lập |
| UI-SALES-02 | Quantity/price đang nhập dở | Raw text không bị projection ghi đè; chưa tự nhận giá trị |
| UI-SALES-03 | Lỗi nhiều ô rồi sửa một ô | Xóa đúng lỗi đã sửa; còn nguyên lỗi ở ô khác |
| UI-SALES-04 | Save bị conflict hoặc lỗi validation | Giữ bản sửa và hướng dẫn phục hồi phù hợp |
| UI-LOOKUP-01 | Gõ từng ký tự, thay selection, mở/cancel popup | Giữ raw text/caret/focus và committed identity đúng |
| UI-LOOKUP-02 | Response cũ đến sau query mới | Kết quả cũ không thay thế trạng thái hiện tại |
| UI-DATE-01 | Nhập ngày ngắn rồi Up/Down Month nhiều lần | Giữ logical segment và accepted value đến lúc commit |
| UI-DATE-02 | F4, di chuyển ngày, Escape hoặc pointer dismiss | Highlight, selection và focus có hành vi nhất quán |
| UI-CLOSE-01 | Đóng tab khi dirty hoặc raw input invalid | Đi qua admission; không mất sửa đổi bị từ chối |
| UI-POST-01 | Post timeout, outcome chưa xác định | Chuyển sang recovery; không tự gửi lại Post |
| UI-SESSION-01 | Đổi session/đóng workspace khi request còn chạy | Callback cũ không khôi phục dữ liệu hoặc thông báo |
| UI-LAYOUT-01 | Tên tiếng Việt dài, font lớn, Light/Dark | Đọc được, thao tác được, không che lỗi quan trọng |

Lab và Web có thể dùng cùng tên scenario. Các kỳ vọng caret native hay browser focus phải có assertion riêng của từng renderer, không sao chép máy móc cách phát event.

Không bổ sung luồng thanh toán, công nợ hoặc pricing mới dưới danh nghĩa làm mẫu màn hình. Dữ liệu và hành động của scenario bám phạm vi nghiệp vụ hiện hữu.

## 7. Kiểm chứng theo đúng loại bằng chứng

| Nhóm đề xuất | Dependency chính | Có thể chứng minh |
|---|---|---|
| Presentation logic | Contracts, Desktop.Application, fake ports | State transitions, errors, cancellation |
| Native hosted interaction | WinForms thật, fake ports | Keyboard, caret, focus, popup, buffer admission |
| Web prototype interaction | Browser components, fixtures | Luồng UX và hành vi browser đã dựng |
| Adapter/contract integration | Adapter thật, contract đã chọn | Mapping và transport behavior trong phạm vi test |
| Packaged acceptance | Executable thực và môi trường được chỉ định | Các luồng tích hợp đã chọn trên package đó |

Fake session không chứng minh authentication hoặc authorization thật. Fake response không chứng minh SQL transaction, rowversion conflict, stock consistency hay Post idempotency.

Native tests vẫn cần Windows, STA, xử lý exception và thời hạn hữu hạn. UI Lab bỏ dependency SQL khỏi vòng sửa presentation; nó không biến WinForms thành một runtime không cần Windows.

NativeLookupTypingDriver hiện phát WM_CHAR/WM_PASTE và kiểm tra focus/selection. Giữ loại proof này cho lookup, decimal và DateInput; gán cả chuỗi Text không thay thế được kiểm tra từng ký tự. [E10]

DateInput correction đã từng có candidate qua automated selections nhưng Owner phát hiện highlight không đi theo phím. Evidence ghi rõ lỗi, sửa ở UI library và acceptance của package cuối. Đây là lý do phải kiểm tra đúng quan sát cần bảo vệ. [E11]

Khi thử DPI, ghi physical monitor scale và window DPI riêng. Evidence hiện tại phân biệt monitor 125% với app window vẫn báo 96 DPI; không suy ra đã có per-monitor awareness mới. [E12]

Mỗi scenario nên có owner test rõ. Chỉ mở rộng suite khi thay đổi chạm shared contract hoặc native seam tương ứng; tránh bắt vòng chỉnh spacing chạy lại toàn bộ SQL/package matrix.

## 8. Ranh giới tái sử dụng khi chuyển WinForms sang Web

| Phần | Triển vọng tái sử dụng | Giới hạn |
|---|---|---|
| DTO semantics, error codes | Cao nếu contract transport được thiết kế rõ | C# type không tự thành wire contract |
| Business scenarios và expected outcomes | Cao | Driver và rendering assertions khác nhau |
| Design tokens/semantic roles | Chia sẻ ý nghĩa | Metrics, fonts, layout engine khác nhau |
| Desktop.Application viewmodels | Nguồn tham khảo về state/commands | React không tự chạy lại C# viewmodel |
| WinForms controls/SourceGrid adapters | Giữ cho desktop | Không chuyển nguyên code sang DOM |
| Exact calculations | Server tiếp tục có thẩm quyền | Browser preview cần policy riêng |

SourceGrid có attach/focus/Validated lifecycle riêng, và editor control được tạo từ constructor. Khi viết Web grid, cần kiểm chứng behavior tương đương ở mức người dùng; không cố giữ cùng event order của SourceGrid. [E13][E14]

Migration nên đi theo một vertical slice có contract và acceptance rõ. Một prototype được duyệt về hình thức chưa cho phép thay renderer của màn hình đang giao hoặc bỏ native regression.

## 9. Các quyết định wire format để dành cho thiết kế API

Các mục dưới đây là vấn đề cần quyết định khi có HTTP/Web client, không phải thay đổi DTO hiện tại trong tài liệu này.

- **Int64 identifiers:** chọn biểu diễn an toàn cho JavaScript; đánh giá string IDs thay vì mặc định dùng JSON number cho mọi ID.
- **Decimal:** xác định precision, scale, rounding và cách serialize; cân nhắc decimal string/typed client handling trước khi dùng number cho tiền.
- **Date-only:** phân biệt business date với timestamp; quyết định format ngày và quy tắc timezone, không âm thầm chuyển ngày nghiệp vụ qua UTC.
- **Rowversion/concurrency tokens:** tiếp tục coi là token opaque; client gửi lại theo contract, không tự diễn giải hoặc tạo mới.
- **Null và partial input:** tách input buffer của UI với giá trị hợp lệ gửi qua wire; không gửi raw text tùy ý như DTO đã được chấp nhận.
- **Preview:** quy định rõ kết quả nào chỉ là ước tính trên client và phản hồi server nào có thẩm quyền.

Các decision records phải có ví dụ request/response và contract tests cho biên dữ liệu. Không dùng việc dựng Web prototype để lén đổi kiểu ID, serialization hoặc arithmetic của net48 hiện tại.

## 10. Trình tự áp dụng và tiêu chí ra khỏi từng bước

1. **Chốt phạm vi Lab:** chọn một màn hình thật, các ports cần fake và ba đến năm scenario đầu; xác nhận không dùng Local/Server bootstrap.
2. **Dựng native host:** mở và reset được cùng trạng thái, sửa UI bằng control thật, có focused interaction checks cho rủi ro đang sửa.
3. **Tách vòng test nhẹ:** project graph của presentation/native fake tests không reference Server.Infrastructure hoặc SQL fixtures.
4. **Thử một Web slice khi có câu hỏi thiết kế:** dùng cùng case IDs/dữ liệu; ghi khác biệt renderer và quyết định UX.
5. **Kết nối API thật ở một đợt riêng:** chốt wire decisions, contract tests và acceptance; chưa gắn bước này vào việc hoàn thành prototype.

Bằng chứng hoàn thành Lab phải gồm dependency graph thực tế, cách chạy trên máy phát triển và các scenario đã kiểm tra. Bằng chứng hoàn thành prototype phải ghi rõ hành vi đã dựng, câu hỏi thiết kế đã trả lời và phần còn giả lập.

**Đề xuất pilot:** Sales list/editor, theo [lộ trình áp dụng](2026-10-24-adoption.md). Owner có thể đổi ưu tiên hoặc phạm vi khi chọn phương án. Cho đến khi được duyệt, toàn bộ nội dung ở đây giữ trạng thái PROPOSED.

## Nguồn tại revision đã kiểm tra

[E1]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Application/MyDmsVn.Desktop.Application.csproj#L1-L11
[E2]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.WinForms/MyDmsVn.Desktop.WinForms.csproj#L1-L11
[E3]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Application/FeatureApiClients.cs#L9-L29
[E4]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Application/UiDispatcher.cs#L7-L25
[E5]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/CatalogPresentationClient.cs#L8-L34
[E6]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/SalesReceiptControlTests.cs#L399-L410
[E7]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/SalesReceiptGridTests.cs#L327-L352
[E8]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/src/MyDmsVn.Desktop.Infrastructure.Local/LocalSalesReceiptPreviewCalculator.cs#L1-L11
[E9]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/MyDmsVn.Desktop.Tests.csproj#L8-L34
[E10]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/NativeLookupTypingDriver.cs#L41-L88
[E11]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/evidence/20261024-dateinput-unified-compact-correction.md#L46-L66
[E12]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/evidence/20261024-dateinput-unified-compact-correction.md#L36-L44
[E13]: https://github.com/chung6a8m/sourcegrid/blob/f4e457b43582bf01892f50bdc74aa480531e5944/SourceGrid/SourceGrid/Cells/Editors/EditorControlBase.cs#L17-L108
[E14]: https://github.com/chung6a8m/sourcegrid/blob/f4e457b43582bf01892f50bdc74aa480531e5944/SourceGrid/SourceGrid/Cells/Editors/EditorControlBase.cs#L243-L252
