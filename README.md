# claude-doc

Bản sao lưu artifact từ Claude.ai (xuất ngày 16/10/2026), dùng để import sang tài khoản Claude khác.

## Cấu trúc

- `docs/`: các **Claude Doc**, xuất ra Markdown. Khi import, nhờ Claude tạo lại thành Claude Doc từ file `.md`.
- `pages/`: các **trang HTML** tự chứa (self-contained). Khi import, nhờ Claude publish nguyên file làm artifact.

## Danh sách

| # | Tiêu đề gốc | File | Loại | Cập nhật |
|---|---|---|---|---|
| 1 | WeKnora — Hướng dẫn cấu hình SSO OIDC với baocao | `docs/weknora-sso-oidc-baocao.md` | Claude Doc | 15/10/2026 |
| 2 | baocao.eup.ai — Đặc tả OIDC Provider (SSO cho các dịch vụ eUp) | `docs/baocao-oidc-provider-spec.md` | Claude Doc | 14/10/2026 |
| 3 | Tình trạng xử lý feedback AI Creative Studio | `docs/ai-creative-studio-feedback-status.md` | Claude Doc | 13/10/2026 |
| 4 | Báo cáo tình trạng dự án – 13/10/2026 | `docs/bao-cao-tinh-trang-du-an.md` | Claude Doc | 13/10/2026 |
| 5 | Checklist test feedback đợt 2 | `pages/checklist-test-feedback-dot-2.html` | HTML (dùng db) | 09/10/2026 |
| 6 | Mệnh An — ba hướng thiết kế | `pages/menh-an-ba-huong-thiet-ke.html` | HTML | 08/10/2026 |
| 7 | Hướng dẫn setup tenant demo | `docs/huong-dan-setup-tenant-demo.md` | Claude Doc | 05/10/2026 |
| 8 | MKANJI Go-Live | `pages/mkanji-go-live.html` | HTML | 09/09/2026 |
| 9 | vibecode-kit — Kiến trúc & Cơ chế vận hành | `pages/vibecode-kit-kien-truc-co-che-van-hanh.html` | HTML | 03/09/2026 |
| 10 | vibecode-kit — Đánh giá tiến độ & mức độ trưởng thành | `pages/vibecode-kit-danh-gia-tien-do.html` | HTML | 03/09/2026 |
| 11 | AI_COMPLETE_SUMMARY.md | `pages/ai-complete-summary.html` | HTML | 10/08/2026 |
| 12 | AI_Agentic_Summary_Evaluation.md | `pages/ai-agentic-summary-evaluation.html` | HTML | 10/08/2026 |
| 13 | Xác nhận /clip/le-5 chạy thật | `pages/xac-nhan-clip-le-5.html` | HTML | 08/08/2026 |
| 14 | VibeCode Kit v2 — Kiến trúc Agent-Orchestrated | `pages/vibecode-kit-v2-kien-truc-agent-orchestrated.html` | HTML | 14/07/2026 |
| 15 | AI Agent Bridge — Demo nội bộ | `pages/ai-agent-bridge-demo.html` | HTML | 02/07/2026 |
| 16 | Intelligent CI/CD — Migii JLPT API | `pages/intelligent-cicd-migii-jlpt-api.html` | HTML | 02/07/2026 |

Không xuất: artifact "Untitled" (12/10/2026), là canvas Design trống, không có nội dung.

## Cách import sang tài khoản Claude khác

**Trang HTML (`pages/`)**: tải file lên chat và nhắn:
> Publish nguyên file này thành artifact, giữ nguyên nội dung.

**Claude Doc (`docs/`)**: tải file `.md` lên chat và nhắn:
> Tạo một Claude Doc từ file markdown này, giữ nguyên nội dung và định dạng.

## Lưu ý

- **Checklist test feedback đợt 2** lưu trạng thái tick/ghi chú trong database của artifact (capability `db` + `user`, collection `results`). Lúc xuất, database đang trống. Khi publish lại, nhắc Claude bật capability `db` và `user` để trang lưu được trạng thái.
- **Mệnh An** dùng `localStorage` để nhớ lựa chọn trên từng trình duyệt; phần này không chuyển được và cũng không cần.
- **baocao.eup.ai — Đặc tả OIDC Provider**: sơ đồ nhúng "Luồng đăng nhập SSO · 11 bước, 3 bên" không xuất được sang Markdown, chỉ còn dòng placeholder. Nếu cần, nhờ Claude vẽ lại sơ đồ (ví dụ bằng Mermaid) từ phần mô tả luồng trong tài liệu.
- Các chip ngày và @mention trong Doc được giữ dưới dạng chữ thường.
- Các tài liệu có nhắc đến hạ tầng nội bộ (namespace, host, tên biến secret). Không có giá trị secret thật, nhưng nên để repo ở chế độ **private**.
