# Tình trạng xử lý feedback AI Creative Studio

Oct 12, 2026 · @Linh NM eUp

Trong 13 đầu việc của file Checklist công việc: **10 đã xong, 2 xong một phần, 1 chưa làm**. Việc chưa làm (tách ô logo / ảnh sản phẩm) đang chờ chốt C1.

Đợt 2 lên develop ngày 09/10, đợt 3 ngày 14/10. Bản develop tự deploy lên creative.eup.ai, nên người test dùng được ngay trên đó.

## Tiến độ

Ngày 14/10 đợt 3 lên develop: thêm ngôn ngữ ngay trong studio (C2), Print tự thêm khổ (C3), nút Xem ảnh trong Thư viện. Còn chờ: chốt C1, C4 và thông báo lỗi Social/Print.

| Ngày | Việc | Kết quả |
| --- | --- | --- |
| 14/10 | Đợt sửa thứ 3 | Merge vào develop: thêm ngôn ngữ trong studio (C2), Print tự thêm khổ (C3), Thư viện xem ảnh của bộ đã lưu. |
| 12/10 | Thư viện: mở lại được ảnh của bộ đã lưu (nút "Xem ảnh") | Đã lên develop 14/10. |
| 12/10 | Đọc lại board FigJam | Có đề xuất mới đổi tên 4 tab, cần xác nhận (cuối doc). Chưa có trả lời cho mục Cần chốt. |
| 12/10 | Kiểm tra ảnh vượt giới hạn bị cắt bớt | Không cần sửa: giao diện mới đã chặn đúng giới hạn 5 ảnh. |
| 12/10 | Tìm nguyên nhân Social và Print chưa gen được | Có nghi vấn (mục Cần thông tin), chờ thông báo lỗi từ lần test tới. |
| 12/10 | Lập doc này và trang checklist cho người test | Xong. |
| 09/10 | Đợt sửa thứ 2 | Merge vào develop: 7 việc trong Checklist công việc và 6 việc ngoài danh sách. |

## Tình trạng từng đầu việc

Thứ tự và mức độ lấy theo file Checklist công việc. Mã C1–C4 trỏ tới mục Cần chốt.

| Khu vực | Việc | Mức độ | Trạng thái | Ghi chú |
| --- | --- | --- | --- | --- |
| AI Design / Brief | Kéo rộng chiều ngang khu vực nhập | Medium | Đã xong | Panel trái rộng hơn. Ô ý tưởng tự cao theo nội dung rồi cuộn trong ô. Thêm giới hạn 300 từ, có bộ đếm. |
| AI Design / Ảnh tham chiếu | Tách riêng logo và ảnh sản phẩm | High | Chưa làm | Chờ chốt C1. |
| AI Design / Đầu ra | Thu gọn ô chọn khổ, bỏ nội dung thừa | Medium | Đã xong | Mỗi thẻ còn một dòng. Sửa lỗi khổ "Popup 600x400" hiện sai tỉ lệ. |
| AI Design / Đầu ra | Cho phép thêm ngôn ngữ | Medium | Đã xong | Nút "+ Thêm ngôn ngữ" ở bước Đầu ra, lưu vào dự án. Ai cũng thêm được, chỉ admin xoá được. |
| AI Design / Output | Hiện ảnh to, đúng kích thước | High | Đã xong | Ảnh kết quả không còn bị cắt. Ô xem lớn hiện đủ ảnh theo tỉ lệ thật. |
| AI Design / Output | Nút chuyển sang Resize khi ảnh đã ổn | High | Đã xong | Nút "Chuyển sang Resize & Translate" hiện ngay khi có ảnh, không cần tick chọn. |
| Resize & Translate / Giữ nguyên | Thay text theo file sheet | Medium | Một phần | Các ô "Giữ từ bản gốc" đã có tác dụng: bỏ tick thì AI được đổi phần đó. Thay text theo sheet chờ chốt C4. |
| Resize & Translate / Đầu ra | Ô chọn kích thước nhỏ lại | Medium | Đã xong | Dùng chung thay đổi với AI Design. |
| Resize & Translate / Ngôn ngữ | Thêm ngôn ngữ | Medium | Đã xong | Như dòng AI Design / Đầu ra (thêm ngôn ngữ). |
| Resize & Translate / Kết quả | Hiện đủ ảnh đã resize | High | Đã xong | Cùng thay đổi với AI Design / Output. |
| Social Media Kit | Chỗ đưa logo, sản phẩm, ảnh tham khảo | Medium | Một phần | Đã có ô ảnh tham khảo ở bước 1. Ô riêng cho logo, sản phẩm chờ chốt C1. |
| Social Media Kit / Mẫu thiết kế | Chưa hiểu chức năng |  | Đã xong | Đổi tên bước thành "Bố cục bài đăng", thêm dòng giải thích: chọn cách xếp ảnh và khổ ảnh, không phải mẫu màu. |
| Print Design / Khổ & quy cách | User tự thêm khổ mong muốn | Medium | Đã xong | Nút "+ Thêm khổ riêng": tên + rộng × cao (mm), lưu theo tài khoản từng người. Đã sửa riêng lỗi tải về: PDF, TIFF, JPEG giờ đúng định dạng, PDF đúng khổ mm kể cả tràn lề. |

## Cần chốt

C2 và C3 đã chốt và làm xong (lên develop 14/10). C1 và C4 vẫn cần PO hoặc người test trả lời trước. Trả lời bằng comment ngay dưới từng câu.

- [ ] **C1. Tách ô logo / ảnh sản phẩm / ảnh tham chiếu** (AI Design, Social, Print). Cần mấy ô, mỗi ô tối đa bao nhiêu ảnh? Hiện tổng tối đa là 5 ảnh, nâng lên được, nhưng mỗi ảnh thêm làm tăng chi phí mỗi lần gen.
- [x] **C2. Thêm ngôn ngữ ngay trong studio** (AI Design, Resize). Ngôn ngữ thêm vào chỉ dùng cho lần gen đó, hay lưu luôn vào dự án? Liên quan feedback ở Cài đặt: bỏ ô thêm khổ bên trong studio, đưa phần thêm ngôn ngữ ra ngoài.
- [x] **C3. Print tự thêm khổ.** Ô nhập chỉ gồm rộng × cao (mm), hay có cả tràn lề và DPI? Khổ tự thêm có lưu lại để lần sau dùng tiếp không?
- [ ] **C4. Thay text theo file sheet trong Resize.** Tính năng gen hàng loạt theo sheet đã có sẵn nhưng đang tắt. Bật nó lên cho người test dùng thử trước, hay làm lại bên trong màn Resize & Translate?

## Cần thông tin: Social và Print chưa gen được

Dev chưa tái hiện được lỗi này, nên cần thông tin từ lần test tiếp theo trên creative.eup.ai. Khi gặp lại, nhờ ghi vào comment:

1. Workspace (Social hay Print) và các lựa chọn đã bấm.
2. Thông báo lỗi hiện trên màn hình, chụp màn hình nếu được.
3. Giờ test (để dev tìm đúng log).

Nghi vấn hiện tại: dự án chưa có ảnh brand (mascot, màu, element) và người dùng không tải ảnh tham khảo thì hệ thống từ chối tạo ảnh. Trước đây Social và Print không có ô ảnh tham khảo, nên dễ rơi vào trường hợp này. Nếu thông báo lỗi bắt đầu bằng "Không có ảnh nào để gửi", hãy thêm ảnh brand ở Cài đặt → Dự án & Brand kit, hoặc tải ít nhất 1 ảnh tham khảo.

## Đã làm thêm ngoài danh sách

Các feedback này có trên board FigJam nhưng không có trong file Checklist công việc. Tất cả đã xong.

| Khu vực | Đã làm |
| --- | --- |
| AI Design / Output | Chuyển sang tab khác không còn mất lượt đang gen. |
| Thư viện | Chế độ "Tất cả ảnh" luôn hiện ảnh đã gen, kể cả khi chưa lưu bộ nào. Bấm ảnh để mở cỡ đầy đủ. |
| Thư viện | Lọc theo khoảng ngày và theo người tạo. Lọc người tạo chỉ có ở chế độ "Theo bộ", áp dụng cho bộ lưu từ 09/10 trở đi. |
| Thư viện | Ô lọc khổ và ngôn ngữ có đủ lựa chọn trong cấu hình dự án. |
| Cài đặt | Thêm chú thích dưới 2 ô prompt: ô "Tạo banner mới" là khung câu lệnh AI Design gửi cho AI; ô "Popup" chỉ dùng khi khổ có chữ "Popup". |
| Quản lý chi phí | Đặt hạn mức chi phí theo tháng cho từng người dùng ở Cài đặt → Người dùng. |

**Đã lên develop 14/10:** mỗi bộ trong Thư viện có nút "Xem ảnh" để mở lại ảnh của bộ đó. Chỉ áp dụng cho bộ lưu sau bản cập nhật này; bộ cũ vẫn xem ảnh ở chế độ "Tất cả ảnh".

## Đề xuất tính năng

Lấy từ bảng "Đề xuất tính năng" trên board FigJam, đọc ngày 12/10.

| Đề xuất | Tình trạng |
| --- | --- |
| 1. Gen hàng loạt trên file sheet theo kế hoạch | Đã có sẵn nhưng đang tắt, chờ chốt C4. |
| 2. Chuyển ảnh từ AI Design sang Resize & Translate | Đã xong, xem bảng trên. |
| 3. Đề xuất chủ đề, có mẫu idea | Chưa làm. Cần mô tả thêm: mẫu theo ngành, theo dịp, hay theo sản phẩm? |
| 4. Tách riêng ô logo, ảnh tham chiếu, ảnh sản phẩm | Chưa làm, chờ chốt C1. |

**Đề xuất mới trên board (12/10), cần xác nhận:** đổi tên 4 tab ở khối "Brief sáng tạo": Nhập tay → Design, Import từ Google Sheet → Auto design theo plan, Upload ảnh & sửa → Remove background, In ấn → Design Print. Ảnh chụp trên board là giao diện cũ; giao diện mới đã tách thành các workspace AI Design, Resize & Translate, Social Media Kit, Print Design. Cần hỏi lại: đây là đề xuất tên cho menu của giao diện mới, hay chỉ cho giao diện cũ? "Remove background" có phải là tính năng tách nền riêng, không phải "sửa ảnh"?
