# Đề xuất tinh gọn tài liệu, tests và Codex skills

**Trạng thái: PROPOSED — chưa phê duyệt triển khai.**
**Ngày:** 2026-10-24.
**Baseline khảo sát:** `MyDmsVn/v1a-pro-ui-next@03acacdaea6a750aa88b33d61ea139379edcb326`.

Tài liệu này đề xuất cách giảm thời gian tìm hiểu và kiểm chứng thay đổi.
Chưa di chuyển tài liệu sản phẩm, xóa test, cài skill hoặc thay đổi CI.
Ứng dụng tiếp tục dùng `net48`; việc tổ chức tài liệu không mở lại lộ trình nâng runtime.
Đọc cùng [hiện trạng](../assessments/2026-10-24-current-state.md), [tổ chức repo](2026-10-24-organization.md), [prototype UI](2026-10-24-ui-prototyping.md) và [lộ trình áp dụng](2026-10-24-adoption.md).

## 1. Kết luận chính

Nên giảm số nơi lặp lại cùng một sự thật trước khi giảm số tài liệu hoặc số test.
Giữ tài liệu quyết định và UX ở `MyDmsVn.Docs`; giữ hợp đồng gắn với code, lệnh test và runbook ở repo thực thi.
Mỗi trạng thái công việc chỉ có một trang sở hữu; các trang vào chỉ liên kết đến trang đó.
Đưa workflow lặp lại vào sáu skill có phạm vi rõ, được nạp khi cần.
Không chuyển toàn bộ kế hoạch vào `AGENTS.md` và không bắt mọi thay đổi đi qua cả sáu skill.

Chính sách test hiện tại đã có lựa chọn theo thay đổi và không yêu cầu chạy toàn bộ solution sau mỗi edit.
Vấn đề là tài liệu dài, trạng thái bị sao chép và một số assertion vẫn kiểm tra dấu vết của lần di chuyển trước.
Có thể làm nhẹ cách làm việc mà vẫn giữ bảo vệ cho transaction, authorization, concurrency, native input và recovery. [Nguồn: chính sách hiện tại][S-policy]

### Bằng chứng cụ thể

| Quan sát tại baseline | Ý nghĩa |
|---|---|
| `docs/` có 401 file, trong đó 113 Markdown | Đây là số file, không phải 401 quy trình đang hoạt động |
| `docs/plans/` chỉ còn một README; archive có tám file | Sáu kế hoạch cũ đã được lưu trữ; không cần làm lại bước này |
| Bảy tài liệu đầu vào thường gặp có tổng 101.787 byte | Nhiều nội dung phải đọc và cập nhật lại ở các trang khác nhau |
| Có 10 cặp PNG trùng Git blob SHA; không có Markdown trùng nguyên blob | Lặp ý và quyền sở hữu thông tin đáng xử lý hơn tìm file Markdown giống hệt |
| 10 test project đều `net48`; 224 file C# dưới `tests/` | 224 bao gồm helper, partial class và probe; không phải số test case |

Các số trên lấy từ recursive tree tại commit đã nêu, không gồm nội dung submodule.
Bảy tài liệu là AGENTS, AI_CONTEXT, README gốc, docs/README, TEST_POLICY, TEST_STRATEGY và tests/README.
Không thực thi build, discovery, SQL hoặc GUI trong khảo sát; không có benchmark mới.
Không thấy cấu hình `.github/` hay YAML trong tree này; cấu hình CI bên ngoài và branch requirements chưa được kiểm tra.

## 2. Một chủ sở hữu cho mỗi loại tài liệu

| Nội dung | Nơi sở hữu đề xuất | Nơi khác giữ gì |
|---|---|---|
| Mục tiêu sản phẩm, phạm vi, ưu tiên nghiệp vụ | MyDmsVn.Docs | Liên kết ngắn từ README sản phẩm |
| Quyết định kiến trúc xuyên repo và lý do chọn | MyDmsVn.Docs | Liên kết quyết định từ phần code liên quan |
| Luồng UX, trạng thái màn hình, acceptance intent | MyDmsVn.Docs | Fixture và test hiện thực hóa tại repo chạy được |
| API/DTO thực tế, dependency graph, schema, transaction | Repo sản phẩm, sát phiên bản code | Docs giải thích mục đích và trỏ đến permalink |
| Public control API và tương thích UI | Repo thư viện/UI Kit | Docs mô tả trải nghiệm và ranh giới ownership |
| Lệnh build/test, biến môi trường, SQL/GUI prerequisites | Repo đang thực thi các lệnh đó | Docs không chép lại lệnh thành một bản thứ hai |
| Runbook migration, vận hành, đóng gói | Repo sản phẩm | Docs có mục lục và giới hạn triển khai |
| Bằng chứng test/acceptance | Theo candidate tại repo tạo bằng chứng hoặc kho evidence đã chọn | Liên kết đến bản gốc; không suy diễn PASS cho candidate mới |
| Trạng thái công việc hiện tại | Một roadmap được chỉ định | README, AGENTS, AI_CONTEXT và index chỉ trỏ tới đó |

“Canonical” nghĩa là chỉ một nơi được sửa nội dung gốc, không phải gom mọi thứ vào Docs.
Không duy trì hai bản CONTRACTS, TEST_POLICY hoặc lệnh `dotnet test` rồi yêu cầu đồng bộ bằng tay.
Tài liệu trong Docs dùng permalink khi mô tả một baseline; dùng liên kết điều hướng khi muốn người đọc đến bản hiện hành.
Mỗi quyết định cần nêu phạm vi, lý do, điều kiện áp dụng và liên kết thay đổi liên quan; không cần thêm sổ phê duyệt.

### Trạng thái hiện tại đang mâu thuẫn

[AGENTS](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/AGENTS.md#L9), [README](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/README.md#L7), [docs index](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/README.md#L3), [TEST_POLICY](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/TEST_POLICY.md#L5), [TEST_STRATEGY](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/TEST_STRATEGY.md#L3) và [test commands](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/README.md#L3) nói DateInput UIN-D01–D03 là NOT STARTED / NOT RUN.
Trong cùng commit, [AI_CONTEXT](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/AI_CONTEXT.md#L9) và [roadmap](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/20261023-active-roadmap.md#L16-L22) nói VERIFIED trong phạm vi đã ghi nhận.
[UI-NEXT index](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/ui-next/README.md#L3-L5) còn ghi toàn bộ task chưa bắt đầu.
Đây là lỗi trạng thái có thể khiến Codex thực hiện lại việc đã xong.

Giữ roadmap UI-NEXT hiện tại làm nguồn trạng thái cho đến khi một lần chuyển quyền sở hữu được thực hiện rõ.
Nếu sau này đưa roadmap sang Docs, chuyển một lần, sửa liên kết và để redirect cần thiết.
Không tạo thêm `STATUS.md` ở root để trở thành bản trạng thái thứ ba.
AI_CONTEXT chỉ cần baseline đang đọc, vấn đề đang xử lý và liên kết đến roadmap/evidence; không chép toàn bộ tiến độ.

Các pin cũ trong [AI_CONTEXT](https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/AI_CONTEXT.md#L31-L37) được ghi rõ là baseline đã kiểm tra.
Chúng là thông tin lịch sử hợp lệ, không được sửa thành HEAD mới rồi mất dấu provenance.
Cần phân biệt “pin của baseline lịch sử” với “pin hiện đang được source tree tiêu thụ”.

## 3. Rút gọn, lưu trữ hay xóa

| Nhóm | Hành động đề xuất | Điều kiện |
|---|---|---|
| Trạng thái lặp trong trang vào | Xóa phần chép lại, giữ liên kết chủ sở hữu | Không mất thông tin blocker hoặc giới hạn chỉ có ở trang đó |
| README và AGENTS quá nhiều diễn tiến | Rút gọn theo trách nhiệm | Bất biến sản phẩm vẫn có đường dẫn rõ |
| Lệnh test hiện tại | Giữ ở repo chạy test | Kiểm tra project/filter thật, không đoán selector |
| Kế hoạch đã hoàn tất hoặc dừng | Lưu trữ, loại khỏi đường đọc thường ngày | Giữ trạng thái cuối, lý do dừng và evidence |
| Prompt giao việc dùng một lần | Gộp thông tin hữu ích rồi bỏ khỏi navigation hiện tại | Không xóa quyết định hoặc acceptance chưa có nơi khác |
| File thay thế hoàn toàn, không còn thông tin riêng | Xóa và sửa inbound links | Git history/permalink vẫn truy xuất được |
| Evidence lỗi gốc, ngoại lệ chấp nhận | Giữ | Không biến archive thành hoàn tất hoặc miễn kiểm chứng |
| Ảnh trùng SHA | Một file gốc, nhiều liên kết | Giữ chú thích ảnh đã được dùng lại tại checkpoint nào |

Sáu kế hoạch cũ đã được chuyển theo [archive inventory][S-archive]; `docs/plans/README.md` hiện là redirect.
Không khôi phục các kế hoạch REL/P6/net8 đã bị loại khỏi phạm vi.
Mười cặp ảnh trùng nằm giữa T04 catalog captures và T06 comparisons của BCF.
Dùng lại một đường dẫn có thể giảm 1.054.156 byte trong working tree.
Git vốn dùng chung object cho blob giống nhau, nên không tuyên bố tiết kiệm bằng đó dung lượng remote.

Không lấy “xóa 50% docs” làm tiêu chí thành công.
Tiêu chí tốt hơn là người đọc tìm được quyết định, code contract, trạng thái và lệnh kiểm chứng mà không phải đối chiếu nhiều bản.

## 4. Dọn tests theo rủi ro được bảo vệ

Giữ test nếu nó bảo vệ hành vi hiện tại hoặc ranh giới lỗi có giá trị.
Tuổi file, tiền tố phase hay tên “Migration/Pilot” không đủ để kết luận test đã lỗi thời.
Trước khi xóa một case, cần chỉ ra hành vi đã bỏ khỏi sản phẩm hoặc case còn lại thực sự bảo vệ cùng rủi ro, cùng lớp quan sát.
Ghi lý do ngay trong PR; không tạo replacement ledger hoặc manifest mới cho mọi test.

### Các ứng viên đã nhìn thấy trong source

| Case hoặc nhóm | Đề xuất cụ thể |
|---|---|
| PermissionRemovalInventoryTests, dòng 39–40 | Bỏ assertion về tên runbook khi tổ chức lại docs; kiểm tra authorization không nên phụ thuộc tên file Markdown |
| DesktopSessionProjectContractTests, danh sách 17 tên file | Bỏ kiểm kê cố định của lần chuyển file cũ; giữ net48, project references và solution inclusion |
| OperationalPersistenceBoundaryTests, regex/token trong C# | Giữ kiểm tra package/assembly; thu hẹp assertion phụ thuộc comment hoặc cách viết `IQueryable<` |
| UiDependencyCompatibilityTests, theme smoke | Kiểm tra observable của grid hoặc gộp khi đã xác định coverage tương đương; hiện test chỉ nhìn theme toàn cục và IsDisposed |
| GoodsReceiptEditorViewModelTests trong GUI project | Tách phần pure fake-API sang Session khi phù hợp; giữ case dispatcher/STA tại Desktop |

Nguồn: [assertion tên docs][S-doc-test], [danh sách file Session][S-session-test], [source text assertions][S-token-test], [theme smoke][S-theme-test], [case STA thực sự][S-sta-test].

Project Session hiện có 26 file C#, nhưng danh sách cũ chỉ chứa 17 tên.
Điều này chứng minh danh sách không còn bao quát project; không chứng minh chín file còn lại sai hoặc đều là test.
Tương tự, Sales và Goods Receipt có numeric draft tests gần giống nhau nhưng bảo vệ hai implementation.
Chỉ gộp khi không mất khác biệt về authorization, raw buffer, notification và state transition.

[InventoryLinqMigrationTests][S-inventory-test] vẫn kiểm tra precision, Unicode, identity, ngày và write allowlist của implementation hiện hành.
[SecurityNativeMigrationTests][S-security-test] vẫn kiểm tra credential, nullable snapshot và atomic password replacement.
Có thể đổi tên cho dễ hiểu; không xóa chỉ vì migration đã xong.

### Các ranh giới cần giữ

| Lớp | Điều nó có thể chứng minh |
|---|---|
| Mock/fixture dùng cho prototype | Bố cục, dữ liệu mẫu, trạng thái loading/empty/error và thao tác dự kiến |
| Pure Application/Session | Validation, quy tắc trạng thái, actor/session generation, recovery và điều phối |
| Contract/API | DTO, error semantics, tương thích serialization và adapter; HTTP chỉ khi thực sự có host |
| Native WinForms/UI | Focus, caret, native keyboard, binding, DPI/theme, accessibility và disposal |
| SQL integration | Transaction, locking, rowversion, schema, grants, reset và dữ liệu thực tế |

Screenshot hoặc mock không thay thế SQL hay native input.
Test-session injection không chứng minh actual Login.
Case concurrency phải quan sát các SQL session độc lập; chạy tuần tự toàn bộ thao tác không còn kiểm chứng tranh chấp.
Coverage cao hoặc lệnh trả exit 0 không chứng minh các case bắt buộc đã được thực thi.

## 5. Giữ fixture đang có và chạy đúng phạm vi

[TEST_POLICY hiện tại][S-policy] đã là nền tảng phù hợp: chọn theo hành vi thay đổi, mở rộng khi chạm hợp đồng dùng chung.
Không khôi phục `tests/test-gates.v1.json`, SqlTestHarness, SQL runner, runner tests, approval file hoặc fingerprint gate.
Không thêm framework chọn test mới chỉ để bọc `dotnet test`.
Các nhãn lớp kiểm chứng ở trên dùng để giải thích phạm vi, không phải sáu cổng phải vượt qua trong mọi PR.

[SqlFixtureRunMode][S-mode] đã mặc định reuse; fresh vẫn được chọn tường minh.
[Inventory csproj][S-linked] đã link helper chung, nên nhiều project không đồng nghĩa nhiều bản implementation.
[ReusableSqlFixture][S-lifecycle] có drain, quarantine và verify-after; cần giữ các bảo vệ này.
Migration, privilege, schema mutation và một số fault/concurrency case tiếp tục dùng fresh theo nguồn hiện tại.
Không chạy lại cả sáu SQL owner ở cả fresh/reuse cho một sửa đổi trình bày.

[SqlServerFact][S-skip] skip khi thiếu cấu hình SQL.
Điều đó hữu ích cho phát triển không liên quan SQL, nhưng kết quả phải ghi SQL NOT RUN nếu SQL nằm trong phạm vi yêu cầu.
Chạy GUI cần Windows/STA và môi trường tương tác phù hợp; không giả định mọi Windows CI agent đều đáp ứng native observations.
Không tạo thêm HTTP host hoặc đổi runtime để làm vừa một sơ đồ test.

### Ngân sách phản hồi đề xuất

**Đây là mục tiêu chưa đo, không phải benchmark, SLA, timeout hay CI đang tồn tại.**

| Loại thay đổi | Mục tiêu tham khảo sau warm build | Phạm vi |
|---|---|---|
| Markdown | Dưới 1 phút | Diff, link, trạng thái và encoding |
| Pure behavior/Session | 1–3 phút | Project/filter liên quan |
| Repository/query/SQL | 5–10 phút cho lát cắt tập trung | Owner bị ảnh hưởng và boundary cần thiết |
| Native UI tự động thường ngày | 5–10 phút | Control/session/host liên quan |
| Package, physical DPI, attended | Xếp lịch theo scenario | Không đặt một giới hạn ngắn chung |

Đo vài thay đổi đại diện trước khi điều chỉnh các mục tiêu này.
Tách thời gian restore/build, test body và cleanup để biết chỗ tốn thời gian.
Không cắt cleanup an toàn để đạt ngân sách.
Một so sánh fresh/reuse phải dùng candidate và điều kiện tương ứng; khảo sát này không đưa ra phần trăm tăng tốc.

## 6. Blueprint sáu Codex skills

Đây là thiết kế skill, chưa phải bộ skill cài đặt được.
Mỗi skill có một việc chính, trigger rõ, đầu vào/đầu ra và vài tình huống đánh giá.
Skill tham chiếu tài liệu chủ sở hữu; không chép PRD, TEST_POLICY và roadmap vào sáu file.
Theo [hướng dẫn chính thức về skills](https://learn.chatgpt.com/docs/build-skills), mô tả giúp chọn skill, nội dung đầy đủ chỉ được nạp khi skill được dùng; nên ưu tiên instruction khi chưa cần script.

### 6.1 ui-prototype

- **Trigger:** cần thử luồng hoặc màn hình trước khi xây feature; không tự chạy cho bugfix thuần SQL.
- **Đầu vào:** mục tiêu người dùng, UX intent, component có sẵn, fixture/state contract và ràng buộc kích thước.
- **Thực hiện:** dựng prototype bằng component thật khi khả thi; cung cấp normal/loading/empty/error/denied; nối tương tác bằng fixture có chủ sở hữu.
- **Đầu ra:** prototype mở được, danh sách trạng thái/interaction và điểm cần kiểm chứng native.
- **Đánh giá:** đổi fixture tái hiện được cùng trạng thái; flow chạy được; không trình bày fake Save/Post như dữ liệu thật.
- **Nơi chạy sau này:** repo chứa prototype thực thi, dùng chung fixtures theo [đề xuất UI](2026-10-24-ui-prototyping.md).

### 6.2 feature-slice

- **Trigger:** một lát cắt nghiệp vụ đã rõ mục tiêu, đi qua một hoặc nhiều lớp hiện có.
- **Đầu vào:** use case, trạng thái trước/sau, contract đang có, baseline và phạm vi chấp nhận.
- **Thực hiện:** truy dấu UI → Application → Local/Server → persistence bị ảnh hưởng; triển khai phần nhỏ có thể review.
- **Đầu ra:** source thay đổi, behavior tests cần thiết và ghi chú kết quả ngắn.
- **Đánh giá:** xử lý cả thành công/lỗi liên quan; không mở rộng feature ngầm; không bắt viết lại kế hoạch cho sửa nhỏ.
- **Nơi chạy sau này:** repo sản phẩm; gọi test-scope khi cần lựa chọn phạm vi kiểm chứng.

### 6.3 contract-change

- **Trigger:** sửa DTO, error semantics, public control API hoặc ranh giới dùng chung có consumer.
- **Đầu vào:** contract hiện tại, consumer đã tìm trong source, yêu cầu tương thích và baseline dependency.
- **Thực hiện:** phân biệt đổi hình dạng với đổi hành vi; cập nhật producer/consumer tương ứng; ghi rõ phần còn chưa có source.
- **Đầu ra:** diff contract và consumer, test contract/adapter liên quan, quyết định tương thích.
- **Đánh giá:** consumer cũ có hành vi xác định; validation/error mapping không mất dữ liệu; không suy ra HTTP coverage từ Local call.
- **Nơi chạy sau này:** repo sở hữu contract; phần generic UI thuộc UI Kit khi phương án tổ chức được thông qua.

### 6.4 native-control

- **Trigger:** sửa control WinForms, editor, popup hoặc rendering có tác động native semantics.
- **Đầu vào:** control source, host thật, trạng thái input, ràng buộc theme/DPI và baseline dependency.
- **Thực hiện:** kiểm tra ownership/disposal và input/focus; chọn STA/native case đúng rủi ro; dùng host nhỏ để tái hiện.
- **Đầu ra:** control/adapter diff, reproducer hoặc test tập trung, evidence phân biệt hosted với physical display.
- **Đánh giá:** caret/range/focus sau thao tác đúng; cancel/dispose không để callback muộn; theme/DPI có phạm vi ghi rõ.
- **Nơi chạy sau này:** UI Kit cho control generic; repo sản phẩm cho policy và adapter nghiệp vụ.

### 6.5 test-scope

- **Trigger:** cần chọn test cho diff, xem một case dư/lỗi thời, hoặc giải thích vì sao phải mở rộng phạm vi.
- **Đầu vào:** diff, source/project graph, risk boundary, test hiện có và môi trường thật.
- **Thực hiện:** chọn owner/filter đã tồn tại; phân biệt pure/native/SQL; kiểm tra selector không rỗng khi chạy.
- **Đầu ra:** lý do chọn scope, lệnh thực tế, kết quả/pass/fail/skip/not-run và tài nguyên chưa cleanup nếu có.
- **Đánh giá:** docs-only không kéo SQL/GUI; transaction change có SQL; missing SQL không được ghi PASS.
- **Nơi chạy sau này:** repo thực thi; dùng policy hiện có, không sinh manifest hoặc runner riêng.

### 6.6 maintain-docs

- **Trigger:** hành vi/quyết định đã đổi, trạng thái không khớp evidence, hoặc người dùng yêu cầu dọn tài liệu.
- **Đầu vào:** thay đổi thực tế, nguồn chủ sở hữu, inbound links và evidence theo candidate.
- **Thực hiện:** sửa đúng tài liệu gốc; gộp/xóa phần lặp; archive nội dung lịch sử; sửa link sau di chuyển.
- **Đầu ra:** diff docs nhỏ, bảng disposition khi có nhiều file và một nguồn trạng thái rõ.
- **Đánh giá:** không còn hai trạng thái trái nhau; baseline lịch sử còn đúng; không tạo thêm roadmap hay ROOT status.
- **Nơi chạy sau này:** Docs cho intent/decision/UX; repo sản phẩm cho code contracts và runbooks.

### Materialize sau khi phạm vi thực thi được chấp thuận

Blueprint có thể được quản lý trong Docs.
Skill thực thi sau này đặt ở `.agents/skills/<tên-skill>/SKILL.md` tại repo nơi Codex làm việc.
Tài liệu chính thức nêu Codex tìm `.agents/skills` từ working directory lên repo root; skill đặt riêng trong Docs không tự được nạp khi làm ở một repo khác.
Chọn owner cho mỗi skill, tránh hai bản cùng tên nhưng hướng dẫn khác nhau.
Đây là đề xuất vị trí cho giai đoạn sau; tài liệu này không tạo `SKILL.md`, script hoặc installer.

Đánh giá mỗi skill bằng một prompt nên trigger, một prompt không nên trigger và một tình huống thiếu thông tin/môi trường.
Đo kết quả công việc và phạm vi đọc; không đo chất lượng bằng độ dài checklist.
Không bắt skill xin lại quyền đã có hoặc mở thêm chuỗi approval cho thao tác thường lệ.
Chỉ thêm script khi có nhu cầu lặp lại rõ và hành vi xác định; không bắt đầu bằng bộ orchestration.

## 7. AGENTS nhỏ và ổn định

Theo [hướng dẫn AGENTS.md chính thức](https://learn.chatgpt.com/docs/agent-configuration/agents-md), Codex đọc instruction theo phạm vi repo/thư mục và kết hợp theo thứ tự.
Vì vậy nội dung root nên ngắn, ổn định và tránh trùng với instructions ở phạm vi nhỏ hơn.
Mục tiêu biên tập đề xuất là khoảng 15–30 dòng rõ nghĩa; không phải một giới hạn kỹ thuật bắt buộc.

AGENTS sản phẩm chỉ cần nêu các bất biến quan trọng và đường dẫn:

- Ứng dụng hiện tại `net48`, Windows x86; modern app runtime vẫn parked.
- Domain/Application không phụ thuộc WinForms/Infrastructure; Local dispatch và explicit UoW giữ ownership.
- Authorization, atomic Post, rowversion, uncertain-write recovery và raw edit buffer không bị phá.
- SQL chỉ dùng instance test được chỉ định, ownership/drain/cleanup được bảo toàn.
- UI generic thuộc thư viện; workspace/session/command policy thuộc sản phẩm.
- Chọn test theo diff tại policy/commands hiện hành; báo đúng NOT RUN.
- Quy tắc encoding, vendor pin và vị trí roadmap/contract chủ sở hữu.

Không nhét lịch sử phase, số case theo candidate, trình tự task và toàn bộ skill vào AGENTS.
README gốc phục vụ bắt đầu dùng/build repo; AGENTS phục vụ hướng dẫn thực thi; roadmap sở hữu trạng thái.
Những nội dung này không cần ba bản sao của cùng một tiến độ.

## 8. Cách áp dụng với ít gián đoạn

1. Sửa mâu thuẫn trạng thái và bỏ các đoạn sao chép trước, vẫn dùng nguồn evidence hiện tại.
2. Chốt bảng quyền sở hữu docs; chuyển theo từng nhóm có inbound links, không di chuyển hàng loạt bằng tên file.
3. Dọn assertion bookkeeping đã xác định; giữ hoặc thay thế coverage hành vi trước khi xóa test.
4. Tách pure/STA thật sự cần tách và giữ fixture dùng chung; không mở một đợt chia project lớn.
5. Thử blueprint skill trên một thay đổi thật; chỉ materialize phần hữu ích ở repo thực thi.
6. Đo thời gian tìm tài liệu, build/test và số lần phải sửa trạng thái lặp; điều chỉnh theo dữ liệu.

Một PR docs-only chỉ cần kiểm tra diff, link, trạng thái và encoding theo policy hiện tại.
Đề xuất này không đóng các acceptance còn mở, không xác nhận readiness production và không phê duyệt đổi code/repo.
Quyết định áp dụng và thứ tự triển khai thuộc [lộ trình chung](2026-10-24-adoption.md).

## Nguồn tại commit đã khảo sát

[S-policy]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/TEST_POLICY.md#L19-L30
[S-archive]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/docs/archive/20261023-pre-ui-next/README.md#L10-L22
[S-doc-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Architecture.Tests/PermissionRemovalInventoryTests.cs#L32-L40
[S-session-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Session.Tests/DesktopSessionProjectContractTests.cs#L11-L56
[S-token-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Architecture.Tests/OperationalPersistenceBoundaryTests.cs#L17-L46
[S-theme-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/UiDependencyCompatibilityTests.cs#L11-L36
[S-sta-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Desktop.Tests/GoodsReceiptEditorViewModelTests.cs#L394-L432
[S-inventory-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.Inventory.Tests/InventoryLinqMigrationTests.cs#L22-L49
[S-security-test]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.Security.Tests/SecurityNativeMigrationTests.cs#L23-L46
[S-mode]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.IntegrationTests/Fixtures/SqlFixtureRunMode.cs#L9-L18
[S-linked]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.Inventory.Tests/MyDmsVn.Server.Infrastructure.Inventory.Tests.csproj#L28-L45
[S-lifecycle]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.IntegrationTests/Fixtures/ReusableSqlFixture.cs#L174-L215
[S-skip]: https://github.com/chung6a8m/MyDmsVn/blob/03acacdaea6a750aa88b33d61ea139379edcb326/tests/MyDmsVn.Server.Infrastructure.IntegrationTests/SqlServerFactAttribute.cs#L16-L27
