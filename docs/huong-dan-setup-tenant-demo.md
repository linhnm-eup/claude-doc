# Hướng dẫn setup tenant demo

Oct 5, 2026 · @Linh NM eUp

Tenant demo cho phép khách lẻ dùng thử eUp Creative Studio mà không cần tự tạo brand kit: khách đăng nhập là vào thẳng brand demo và gen được ngay.

## 1. Chuẩn bị: biến môi trường trên cụm

Backend của namespace `ai-agent-dev` (domain creative.eup.ai) phải có đủ các biến dưới đây trong ConfigMap/Secret. Thiếu biến nào thì đúng bước tương ứng sẽ hỏng giữa chừng.

| Biến | Dùng ở bước | Thiếu thì gặp |
| --- | --- | --- |
| `CDN_UPLOAD_URL`, `CDN_PUBLIC_BASE_URL` | Upload bảng màu cho brand kit | 502 "Không upload được ảnh lên CDN" |
| `OPENAI_API_KEY` | Gen ảnh | "OPENAI\_API\_KEY không được thiết lập" |
| `OPENAI_IMAGE_EDITS_URL`, `OPENAI_IMAGE_GENERATIONS_URL` | Gen ảnh | "Invalid URL '': No scheme supplied" |
| `CONNECTION_ENCRYPTION_KEY` | Thêm Kết nối AI | 503 "Chưa cấu hình CONNECTION\_ENCRYPTION\_KEY" |

Kiểm tra nhanh, lệnh chỉ in tên biến còn thiếu, không in giá trị:

```bash
kubectl --context k8s-local-dev -n ai-agent-dev exec deploy/design-banner-backend -- python -c "
from src.core.config_check import _required, _optional
print('THIẾU:', [r.name for r in _required()+_optional() if not str(r.value).strip()])"
```

Sau khi thêm biến vào ConfigMap/Secret, phải chạy `kubectl ... rollout restart deploy/design-banner-backend` thì pod mới nhận giá trị mới.

Từ đây trở đi, `BE` là viết tắt của tiền tố lệnh sau:

```bash
kubectl --context k8s-local-dev -n ai-agent-dev exec deploy/design-banner-backend --
```

## 2. Các bước setup

Chỉ làm 1 lần: bước 2.1 đến 2.3. Mỗi khách mới: lặp lại bước 2.4.

### 2.1. Tạo tài khoản admin cho tenant demo

1. Đăng nhập bằng tài khoản admin hiện có của eUp, vào `/admin`.
2. Bấm **Tạo admin**, nhập email (ví dụ `admin-demo@eupgroup.net`) và mật khẩu tạm. Mật khẩu tạm chỉ hiện 1 lần, lưu ngay vào password manager.
3. Lúc này tài khoản đang thuộc tenant eUp. Bước 2.2 sẽ chuyển nó sang tenant demo.

### 2.2. Tạo tenant demo và chuyển admin sang

Tenant chỉ tạo được bằng CLI. Lấy `user_id` của `admin-demo` (dòng mới nhất trong `--list`), rồi chạy:

```bash
BE python -m scripts.assign_user_tenant --list
BE python -m scripts.assign_user_tenant --user-id <id admin-demo> --tenant-id demo --create-tenant
```

- Cờ `--create-tenant` chỉ dùng ở lần đầu. Thiếu cờ này, script từ chối tạo tenant mới để tránh gõ nhầm tên.
- Script sẽ in "CẢNH BÁO: ... vừa được CHUYỂN từ ... sang 'demo'". Đây là kết quả mong muốn.
- Lượt gán được ghi vào nhật ký kiểm toán với method "CLI".

### 2.3. Tạo brand kit demo

Thoát ra, đăng nhập lại bằng `admin-demo` (hệ thống bắt đổi mật khẩu ở lần đầu), rồi vào `/settings`:

1. **Quản lý dự án** → tạo dự án mới, ví dụ `Demo Brand`. Dự án mới tự có sẵn khổ và ngôn ngữ mặc định.
2. **Dự án & Brand kit** → chọn `Demo Brand`:
   - **Brand Identity:** chỉ upload ảnh vào ô **Color** (bảng màu). **Để trống ô Mascot.** Prompt ép model giữ nguyên mascot, nên có mascot thì banner nào của khách cũng mang con mascot đó.
   - Không dùng asset của sản phẩm thật (Mazii, Hanzii...).
3. Thẻ **Brand kit** → **Chỉnh sửa** → đặt Màu chính / Màu đậm.
4. Thẻ **Chi phí & hạn mức** → đặt hạn mức tháng cho cả tenant demo (đề xuất: con số ngân sách demo đã duyệt), hành động khi vượt = **chặn hẳn**.

Tenant demo chỉ nên có **đúng 1 brand kit**. Khi đó khách mở Studio là hệ thống tự chọn brand này, không cần chọn gì.

### 2.4. Tạo tài khoản cho từng khách

Phải đăng nhập bằng **`admin-demo`**. Tài khoản mới luôn vào tenant của người tạo. Tạo bằng admin eUp thì khách vào tenant eUp và nhìn thấy dữ liệu nội bộ.

1. `/admin` → **Tạo admin** → email khách + mật khẩu tạm. Hệ thống chỉ có nút tạo admin, không có nút tạo user thường.
2. **Hạ quyền ngay:** trên dòng vừa tạo → menu ⋯ → **Quyền** → hạ xuống **User**. Bỏ qua bước này thì khách sửa được brand kit, xem được danh sách khách khác và tự tạo thêm tài khoản.
3. Gửi email + mật khẩu tạm cho khách qua kênh riêng. Khách đổi mật khẩu ở lần đăng nhập đầu.

Không hướng dẫn khách dùng chế độ "Đăng ký" trên màn hình đăng nhập: đơn đăng ký hiện cho mọi admin của mọi tenant, admin nào duyệt trước thì khách vào tenant của người đó.

## 3. Kiểm tra sau setup

Đăng nhập bằng 1 tài khoản khách vừa tạo (cửa sổ ẩn danh) và đi hết danh sách sau:

- [ ] Đăng nhập được, bị yêu cầu đổi mật khẩu ở lần đầu.
- [ ] Không thấy menu quản trị `/admin`. Nếu thấy: tài khoản chưa được hạ xuống User.
- [ ] Mở Studio: tiêu đề canvas hiện số lượt (ví dụ `#0001`), không kẹt ở "Đang tải…".
- [ ] Brand đang chọn là `Demo Brand`.
- [ ] Nhập Brief → Ý tưởng → Gen ra được 1 ảnh.
- [ ] Ảnh không có mascot của sản phẩm nào.
- [ ] Đăng nhập lại `admin-demo` → `/admin` → "Tổng chi phí" có ghi nhận lượt gen vừa rồi.

## 4. Lưu ý vận hành

- **Khách trong tenant demo nhìn thấy Library của nhau.** Library lấy theo cả tenant, không tách theo người dùng. Khách nào không muốn lộ brief của mình thì tạo cho họ 1 tenant riêng theo đúng mục 2, đổi tên `demo` thành tên khác.
- **Chi phí tính vào khoá OpenAI của eUp.** Kết nối AI trong Cài đặt hiện chưa được dùng khi gen; mọi tenant đều gen bằng `OPENAI_API_KEY` chung. Vì vậy hạn mức ở bước 2.3 là phanh duy nhất.
- **Hết đợt demo:** vào `/admin` bằng `admin-demo` → menu ⋯ → **Truy cập** → tắt đăng nhập của khách. Khách bị đăng xuất ngay; lịch sử và chi phí vẫn giữ.
- **Khách chốt mua:** tạo tenant riêng cho họ rồi chuyển tài khoản bằng `assign_user_tenant`. Banner đã làm trong tenant demo **không đi theo**, vì dữ liệu nằm lại tenant cũ.
- **Giữ `admin-demo` trong password manager của team.** Mất tài khoản này thì chỉ còn CLI để quản lý tenant demo.

## 5. Xử lý sự cố

| Triệu chứng | Nguyên nhân | Cách xử lý |
| --- | --- | --- |
| Gen báo "Project is required" | Tenant của khách chưa có brand kit, thường do khách bị tạo nhầm bằng admin eUp hoặc vào nhầm tenant | `BE python -m scripts.assign_user_tenant --list`, kiểm tra khách có nằm ở `demo` không; sai thì chạy lại lệnh gán với `--tenant-id demo` |
| Tiêu đề Studio kẹt "Đang tải…", bấm Tạo mới không có tác dụng | Cùng nguyên nhân: không có brand kit | Như dòng trên |
| "Không có ảnh nào để gửi cho project..." | Brand kit không có ảnh nào và khách không upload ảnh tham chiếu | Upload ảnh vào ô Color của `Demo Brand` (bước 2.3) |
| "Invalid URL '': No scheme supplied" | Thiếu `OPENAI_IMAGE_EDITS_URL` | Thêm biến (mục 1), restart backend |
| 502 khi upload ảnh brand kit | Thiếu `CDN_UPLOAD_URL` | Thêm biến (mục 1), restart backend |
| 401 "Chưa đăng nhập." khi gọi API từ Postman/curl | Request không có cookie và không có khoá API | Tạo khoá ở `/api-keys`, gửi header `Authorization: Bearer <khoá>`. Khoá OpenAI trong Kết nối AI không dùng được cho việc này |
| Khách thấy dữ liệu nội bộ eUp | Tài khoản được tạo bằng admin eUp | Chuyển ngay sang `demo` bằng CLI; kiểm tra `/admin/audit` xem khách đã làm gì |
