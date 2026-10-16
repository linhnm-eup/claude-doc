# Báo cáo tình trạng dự án – 15/10/2026

Oct 15, 2026 · @Linh NM eUp

35 task đang mở thuộc 7 epic. Báo cáo nội bộ đã xong 4/4, Chấm công chờ test, còn trọng tâm từ nay đến 28/10 là SSO (19 task, bắt đầu 15/10). SSO chưa thuộc KPI nên đang đề xuất tách KPI tháng 10 thành 4 mục.

## Tổng quan

| Epic | Dự án | KPI đề xuất | Tiến độ | Task mở | Tình trạng | Việc tiếp theo |
| --- | --- | --- | --- | --- | --- | --- |
| EP-005 SSO – baocao làm nơi đăng nhập chung | Staff It | KPI 2 | 0/11 | 11 | Bắt đầu 15/10, KB-18997 đang làm | Nghiệm thu với WeKnora dev 26/10 |
| EP-012 SSO – WeKnora đăng nhập qua baocao | Staff It | KPI 3 | 0/8 | 8 | 3 task đang làm, code có trên nhánh feat/sso-baocao nhưng chưa commit | Deploy lên wiki.eup.ai 28/10 |
| EP-007 Chấm công | Staff It | KPI 2 | 5/9 | 4 (Testing) | Code xong từ 08/10 | Đóng sau khi test xong |
| EP-010 Báo cáo nội bộ | Report | KPI 2 | 4/4 | 0 | Xong; giữ mở cho việc phát sinh | Cập nhật cuối tháng |
| EP-006 ARIA – Trả lời review app | Staff It | KPI 1 | 9/14 | 5 | Chờ key 36 app + server nội bộ | Chưa đặt hạn |
| EP-009 AI Creative Studio | Staff It | KPI 4 | 3/8 | 5 | Feedback đợt 3 hạn 16/10; đã tạo 4 task SaaS | SaaS hạn 27–30/10 |
| EP-008 AI Copilot × WeKnora | Staff It | KPI 3 | 4/11 | 2 | Còn test ngưỡng confidence + gỡ service cũ | Chưa đặt hạn |

## Cần xử lý ngay

Hôm nay và ngày mai có 7 task tới hạn. Ba task WeKnora đã có code, chỉ cần commit là đóng được.

- [ ] KB-19009, 19010, 19011 WeKnora (hạn 15/10): commit code trên nhánh feat/sso-baocao, chuyển sang Testing
- [ ] KB-18997 Cài Passport + OpenID Connect (hạn 15/10, đang làm)
- [ ] KB-18741 Creative Studio feedback đợt 3 (hạn 16/10)
- [ ] KB-18998, 18999 SSO (hạn 16/10): nếu bị dồn việc thì dời KB-18999 sang 19/10
- [ ] KB-18392 → 18395 Chấm công: đang Testing từ 08/10, cần nhắc người test để đóng

## Chi tiết từng dự án

**SSO (EP-005 baocao + EP-012 WeKnora)**: 19 task, bắt đầu 15/10. Phía baocao làm OIDC Provider trước, WeKnora tích hợp sau, deploy lên wiki.eup.ai 28/10.

| Hạn | baocao (EP-005) | WeKnora (EP-012) |
| --- | --- | --- |
| 15/10 | KB-18997 Cài Passport + OIDC | KB-19009 Chặn domain, KB-19010 Chặn user bị khoá, KB-19011 Login tự chuyển sang baocao |
| 16/10 | KB-18998 Key RSA, KB-18999 Cấu hình issuer/scopes |  |
| 19/10 | KB-19000 Claims | KB-19012 Test E2E với Dex local |
| 20/10 | KB-19001 /oauth/authorize |  |
| 21/10 | KB-19002 userinfo, KB-19003 logout |  |
| 22/10 | KB-19004 Trang admin SSO client | KB-19014 Webhook khoá/xoá user |
| 23/10 | KB-19005 Rate limit + chống iframe | KB-19013 Review + merge |
| 26/10 | KB-19007 Nghiệm thu, KB-19006 Tài liệu | KB-19015 Tích hợp baocao dev |
| 28/10 |  | KB-19016 Deploy wiki.eup.ai |

**ARIA (eup\_agents)**: code đăng trả lời lên Google Play / App Store Connect đã xong từ 06/10, còn chờ dữ liệu để chạy thật.

- Đã xong: dashboard giám sát, báo cáo tuần tự chạy, GET /status, Quality Gate chặn duyệt P0; API review của baocao đã trả Store review id + App id (KB-18764).
- Còn lại: KB-17539 chạy API thật (key 36 app), KB-17538 deploy server nội bộ, KB-17552 tự đăng P2/P3, KB-18273 trả lời bổ sung khi đội báo đã sửa, KB-18315 đẩy ticket sang kênh từng đội.

**Chấm công (internal\_report\_daily)**: các quy định HCNS đã chốt đều đã có code, đang chờ test.

- Đã xong (05–08/10): Mobile API, đơn quên chấm công / xin sửa giờ, giờ công theo quy định HCNS, giới hạn đơn từ lần 6/tháng, nhắc chấm vào/ra, xuất bảng công Excel.
- Còn lại: test và đóng KB-18392 → 18395 (OT, sửa đơn đã duyệt, nuôi con nhỏ/thai sản, chấm khi WFH chờ duyệt).

**Báo cáo nội bộ (EP-010)**: 4/4 task đã xong, gồm Báo cáo Product định kỳ, API review cho ARIA, Epic qua API/MCP và các hotfix. Epic giữ mở để nhận việc phát sinh.

**AI Creative Studio (EP-009)**: feedback đợt 3 đã xử lý phần lớn, phần SaaS có 4 task mới.

- Đã xong: trang Terms of Service + Privacy Policy kèm consent, hạn mức chi phí theo người, phần lớn feedback đợt 3.
- Còn lại: KB-18741 feedback đợt 3 (hạn 16/10), KB-19090 Quality Gate CI (27/10), KB-19091 email transactional (29/10), KB-19092 CAPTCHA + xác thực email (30/10), KB-19093 project mẫu cho tenant mới (30/10).

**AI Copilot × WeKnora (EP-008)**: 4/11 task đã xong. Còn KB-18381 test + chỉnh ngưỡng confidence với dữ liệu thật, KB-18382 gỡ service ai-copilot (Node) cũ; cả hai chưa đặt hạn.

## Rủi ro và cần quyết định

| Vấn đề | Ảnh hưởng | Cần gì |
| --- | --- | --- |
| 19 task SSO (hạn 15–28/10) chưa thuộc KPI | Phần việc chính nửa cuối tháng không được tính điểm | Quản lý duyệt phương án 4 KPI (mục dưới) |
| Lịch SSO dày, chồng với Creative Studio (16/10) và SaaS (27–30/10) | Dễ trễ nghiệm thu 26/10 và deploy 28/10 | Giữ ưu tiên: SSO → Creative Studio → ARIA; dời việc ít gấp nếu bị dồn |
| ARIA phụ thuộc key 36 app và server nội bộ | KPI 1 không đạt dù code đã xong | Người cấp credentials Google Play / App Store Connect và server |
| Phía baocao chưa có task gửi webhook khoá/xoá user | KB-19014 (WeKnora nhận webhook) không test đầu-cuối được | Tạo thêm 1 task trong EP-005, hạn 22/10 |
| Chấm công treo ở Testing từ 08/10 | Chưa lên production, KPI 2 chưa có kết quả | Nhắc QA/HCNS test và xác nhận |

## Đề xuất điều chỉnh KPI tháng 10 (15/10)

21/35 task đang mở (60%) nằm ngoài KPI hiện tại, chủ yếu là SSO với 19 task có hạn 15–28/10; đề xuất tách thành 4 KPI để SSO được tính điểm và vẫn giữ Creative Studio SaaS (đã tạo 4 task SaaS KB-19090 → 19093, hạn 27–30/10).

| KPI | Epic | Tỉ trọng cũ | Tỉ trọng mới |
| --- | --- | --- | --- |
| 1. ARIA giai đoạn 2 | EP-006 | 50% | 30% |
| 2. Báo cáo nội bộ – Chấm công, SSO baocao & bảo trì | EP-007, EP-005, EP-010 | 35% | 35% |
| 3. WeKnora – SSO qua baocao & AI Copilot | EP-012, EP-008 | — | 20% |
| 4. Creative Studio – chuẩn bị SaaS | EP-009 | 15% | 15% |
