# baocao.eup.ai — Đặc tả OIDC Provider (SSO cho các dịch vụ eUp)

Oct 14, 2026 · @Linh NM eUp

## Bối cảnh và mục tiêu

baocao.eup.ai trở thành **nhà cung cấp danh tính (OpenID Connect Provider)** dùng chung cho mọi dịch vụ nội bộ eUp. Người dùng chỉ đăng nhập một lần bằng tài khoản baocao.

Dịch vụ đầu tiên dùng là **WeKnora (wiki.eup.ai)**. WeKnora đã có sẵn OIDC client, nên phía WeKnora chỉ cần cấu hình mà không phải viết luồng riêng.

Hiện baocao (Laravel) đăng nhập theo session riêng. Hai đường dẫn `/.well-known/openid-configuration` và `/.well-known/oauth-authorization-server` đều trả 404, kiểm tra ngày 14/10/2026.

## Phạm vi

**Nguyên tắc:** baocao chỉ trả lời câu hỏi "người này là ai". Việc "được làm gì" do từng dịch vụ tự quản. Thêm dịch vụ mới chỉ cần đăng ký thêm một client, không sửa logic baocao.

**Trong phạm vi:**

- Các endpoint OIDC chuẩn, luồng Authorization Code.
- Dùng lại form đăng nhập và session hiện có của baocao. Đã đăng nhập baocao thì không phải nhập lại mật khẩu.
- Quản lý client (tạo, xoay vòng secret, thu hồi) trong trang admin hoặc bằng lệnh artisan.
- Thông báo cho các dịch vụ khi tài khoản bị khóa.

**Ngoài phạm vi:**

- Role, quyền riêng của từng dịch vụ (ví dụ role workspace trong WeKnora).
- Trang consent kiểu "Ứng dụng X muốn truy cập…". Client nội bộ được tin cậy nên bỏ qua bước này.
- Cho đăng ký client tự động (dynamic client registration).

## Luồng đăng nhập

Dùng luồng chuẩn OIDC Authorization Code. Nét liền là request, nét đứt là phản hồi hoặc redirect.

&#91;embedded content: Luồng đăng nhập SSO · 11 bước, 3 bên\]

baocao chỉ tham gia xác nhận danh tính (bước 3 đến 9). Việc tạo user và phân quyền nằm hoàn toàn ở WeKnora (bước 10).

## Endpoint bắt buộc

Issuer: `https://baocao.eup.ai`. Nên dùng giá trị này cố định, không có dấu `/` ở cuối, và claim `iss` trong token phải trùng từng ký tự. Đường dẫn cụ thể của các endpoint có thể đổi, miễn khai báo đúng trong discovery.

| Endpoint | Method | Mục đích |
| --- | --- | --- |
| `/.well-known/openid-configuration` | GET | Discovery: các client tự đọc cấu hình từ đây |
| `/oauth/authorize` | GET | Hiển thị login (hoặc dùng session sẵn có), rồi redirect về client kèm `code` và `state` |
| `/oauth/token` | POST | Đổi `code` lấy `access_token` và `id_token` |
| `/oauth/userinfo` | GET | Trả claims của user theo access token (Bearer) |
| `/oauth/jwks` | GET | Public key để client kiểm tra chữ ký `id_token` |
| `/oauth/logout` | GET | (Nên có) đăng xuất baocao rồi redirect về `post_logout_redirect_uri` |

Discovery document mẫu:

```json
{
  "issuer": "https://baocao.eup.ai",
  "authorization_endpoint": "https://baocao.eup.ai/oauth/authorize",
  "token_endpoint": "https://baocao.eup.ai/oauth/token",
  "userinfo_endpoint": "https://baocao.eup.ai/oauth/userinfo",
  "jwks_uri": "https://baocao.eup.ai/oauth/jwks",
  "end_session_endpoint": "https://baocao.eup.ai/oauth/logout",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email", "groups"],
  "token_endpoint_auth_methods_supported": ["client_secret_post", "client_secret_basic"],
  "code_challenge_methods_supported": ["S256"],
  "claims_supported": ["sub", "email", "email_verified", "name", "preferred_username", "groups"]
}
```

Bắt buộc hỗ trợ **`client_secret_post`**. WeKnora gửi `client_secret` trong form body khi gọi `/oauth/token`, không dùng header Basic.

## Claims và scopes

`id_token` và `/oauth/userinfo` trả cùng một bộ claims. Chỉ trả những claim thuộc scope mà client đã xin.

| Claim | Scope | Bắt buộc | Ý nghĩa |
| --- | --- | --- | --- |
| `sub` | openid | Có | ID user cố định, không bao giờ đổi và không dùng lại (nên là UUID hoặc ID nội bộ, không dùng email) |
| `email` | email | Có | Email công ty, viết thường. WeKnora dùng email để khớp tài khoản |
| `email_verified` | email | Có | `true` khi email đã được xác minh. WeKnora sẽ từ chối nếu là `false` |
| `name` | profile | Có | Họ tên hiển thị |
| `preferred_username` | profile | Nên có | Tên đăng nhập ngắn |
| `groups` | groups | Tùy chọn | Danh sách phòng ban, nhóm (mã ổn định, ví dụ `dept-sales`). Chỉ là thông tin tham khảo, dịch vụ tự quyết có dùng hay không |

Ví dụ payload của `id_token`:

```json
{
  "iss": "https://baocao.eup.ai",
  "sub": "8f3c2a1e-0b5d-4c7e-9a10-2d6f4e8b1c33",
  "aud": "<client_id>",
  "iat": 1792000000,
  "exp": 1792000300,
  "nonce": "<nếu client gửi>",
  "email": "nguyenvana@eupgroup.net",
  "email_verified": true,
  "name": "Nguyễn Văn A",
  "preferred_username": "nguyenvana"
}
```

Phản hồi của `/oauth/userinfo` có cùng các trường trên, trừ `iss`, `aud`, `iat`, `exp`, `nonce`. Nếu client xin scope `groups` thì thêm `"groups": ["dept-sales"]`.

## Yêu cầu bảo mật

- **Chữ ký:** `id_token` ký bằng RS256. Header có `kid` trùng với một key trong JWKS. WeKnora chỉ chấp nhận RS256/384/512, không chấp nhận HS256.
- **Xoay vòng key:** khi đổi key, để JWKS chứa cả key cũ và key mới ít nhất 24 giờ. Private key lưu trong secret, không commit vào repo.
- **Redirect URI:** so khớp **chính xác** với danh sách đã đăng ký của từng client. Không dùng wildcard, không so tiền tố. URI sai thì báo lỗi, không redirect.
- **`state` và `nonce`:** trả nguyên `state` về client. Nếu client gửi `nonce` thì đưa vào `id_token`.
- **PKCE (S256):** hỗ trợ nhưng không bắt buộc với client có secret. WeKnora hiện chưa gửi PKCE.
- **Authorization code:** dùng một lần, hết hạn sau 60 giây, gắn với `client_id` và `redirect_uri` đã dùng khi xin code.
- **Thời hạn token:** `id_token` 5 phút. `access_token` 10 phút, chỉ để gọi userinfo. Refresh token của baocao là tùy chọn, vì WeKnora tự quản phiên đăng nhập của mình.
- **Tài khoản bị khóa:** `/oauth/authorize` từ chối, `/oauth/token` và `/oauth/userinfo` trả `401`.
- **HTTPS:** mọi endpoint chạy qua HTTPS. Client secret chỉ lưu dạng hash (Passport 13 luôn hash, không cần cấu hình).
- **Chống dò mật khẩu:** form đăng nhập baocao có rate limit. Nên bật 2FA cho tài khoản admin.
- **Chống nhúng trang:** trang login gửi header `X-Frame-Options: DENY` hoặc CSP `frame-ancestors 'none'`.

## Đăng ký client cho WeKnora

Cần tạo một client confidential trên baocao với các thông số sau.

| Thuộc tính | Giá trị |
| --- | --- |
| Tên hiển thị | eUp Knowledge Hub (WeKnora) |
| `client_id` | `<UUID do baocao cấp khi tạo>` |
| Loại client | Confidential, có secret |
| Redirect URI (production) | `https://wiki.eup.ai/api/v1/auth/oidc/callback` |
| Redirect URI (dev, local) | `http://localhost:8080/api/v1/auth/oidc/callback` |
| Post-logout redirect URI | `https://wiki.eup.ai/login` |
| Scopes | `openid profile email` |
| Grant types | `authorization_code` |
| Xác thực ở token endpoint | `client_secret_post` |
| Trang consent | Bỏ qua (client nội bộ được tin cậy) |

Nên tạo client riêng cho từng môi trường (dev, prod), mỗi client một secret riêng. client\_id là UUID do baocao sinh, không đặt được dạng chữ như "weknora" vì Laravel Passport 13 bắt buộc UUID; claim aud trong id\_token cũng là UUID này (cập nhật 15/10/2026).

Sau khi tạo xong, team WeKnora cấu hình như sau. Phần này để baocao tham khảo, không cần làm gì thêm:

```
OIDC_AUTH_ENABLE=true
OIDC_AUTH_ISSUER_URL=https://baocao.eup.ai
OIDC_AUTH_DISCOVERY_URL=https://baocao.eup.ai/.well-known/openid-configuration
OIDC_AUTH_PROVIDER_DISPLAY_NAME=eUp
OIDC_AUTH_CLIENT_ID=<client_id do baocao cấp>
OIDC_AUTH_CLIENT_SECRET=<secret do baocao cấp>
OIDC_AUTH_SCOPES="openid profile email"
OIDC_USER_INFO_MAPPING_USER_NAME=name
OIDC_USER_INFO_MAPPING_EMAIL=email
```

## Khóa tài khoản và nghỉ việc

Khi một tài khoản bị khóa hoặc xóa, baocao gửi webhook tới mọi client đã đăng ký `webhook_url`. Nhờ vậy các dịch vụ cắt truy cập ngay, không phải chờ phiên đăng nhập hết hạn. Nếu chưa kịp làm webhook, WeKnora vẫn buộc người dùng đăng nhập lại qua baocao sau tối đa 7 ngày.

- **Thuộc tính mới của client:** `webhook_url` (ví dụ `https://wiki.eup.ai/api/v1/auth/oidc/webhook`) và `webhook_secret` riêng cho từng client.
- **Sự kiện:** `user.disabled`, `user.enabled`, `user.deleted`, `user.updated` (đổi email hoặc tên).
- **Request:** `POST`, `Content-Type: application/json`, header `X-EUP-Signature: sha256=<hex HMAC-SHA256(webhook_secret, raw body)>` và `X-EUP-Timestamp: <unix giây>`. Client từ chối request lệch giờ quá 5 phút.
- **Gửi lại:** nếu client không trả `2xx` thì retry với backoff (1 phút, 5 phút, 30 phút, 2 giờ), dùng Laravel queue. Ghi log trạng thái gửi để admin xem lại.
- **Chống xử lý trùng:** mỗi sự kiện có một `id` duy nhất. Client bỏ qua sự kiện có `id` đã xử lý.

Ví dụ body:

```json
{
  "id": "evt_01JB7Q8K2M",
  "type": "user.disabled",
  "occurred_at": "2026-10-14T09:30:00Z",
  "user": {
    "sub": "8f3c2a1e-0b5d-4c7e-9a10-2d6f4e8b1c33",
    "email": "nguyenvana@eupgroup.net"
  }
}
```

## Gợi ý triển khai trên Laravel

Cách ít tốn công nhất là dùng **Laravel Passport** kèm một package bổ sung OIDC, ví dụ `jeremy379/laravel-openid-connect`. Team baocao có thể chọn package khác, miễn đáp ứng đúng các endpoint và claims ở trên.

1. Cài Passport. Chạy `php artisan passport:keys` để sinh cặp RSA, private key đưa vào secret của môi trường.
2. Cài package OIDC. Khai báo scopes `openid profile email groups` và issuer cố định `https://baocao.eup.ai`.
3. Cho model `User` trả claims theo bảng ở phần Claims, bao gồm `email_verified` và `sub` cố định.
4. Màn authorize dùng lại middleware `auth` và trang login hiện có. Client nội bộ bỏ qua consent (Passport: `skipsAuthorization()` trên model Client).
5. Passport 13 luôn hash client secret (không còn `Passport::hashClientSecrets()`). Đặt thời hạn token theo phần bảo mật.
6. Thêm endpoint `/oauth/logout`: xóa session, kiểm tra `post_logout_redirect_uri` nằm trong danh sách đã đăng ký rồi mới redirect.
7. Thêm cột `webhook_url`, `webhook_secret` cho bảng client. Bắt event khóa, mở khóa, xóa, cập nhật user và đẩy job gửi webhook vào queue.
8. Thêm lệnh artisan hoặc trang admin để tạo client, xoay vòng secret và thu hồi client.
9. Rate limit route login và `/oauth/token`.

## Checklist nghiệm thu

Có thể kiểm nhanh bằng [OpenID Connect Debugger](https://oidcdebugger.com) hoặc `curl`, trước khi nối với WeKnora.

- [ ] `GET /.well-known/openid-configuration` trả JSON đúng mẫu, `issuer` trùng từng ký tự.
- [ ] `GET /oauth/jwks` trả key RSA có `kid`, và `kid` này trùng header của `id_token`.
- [ ] Luồng authorize → login → redirect về kèm `code` và `state` nguyên vẹn.
- [ ] Đã có session baocao thì authorize redirect ngay, không hỏi lại mật khẩu và không hiện consent.
- [ ] `POST /oauth/token` với `client_secret_post` trả `id_token` và `access_token`.
- [ ] `id_token` qua được kiểm tra tại jwt.io bằng public key lấy từ JWKS. `aud` bằng `client_id` của WeKnora, có `email_verified`.
- [ ] `GET /oauth/userinfo` bằng access token trả đủ claims.
- [ ] Dùng lại một `code` lần thứ hai thì bị từ chối (`invalid_grant`).
- [ ] Redirect URI sai một ký tự thì bị từ chối, không redirect.
- [ ] Tài khoản bị khóa thì không lấy được code, userinfo trả `401`.
- [ ] Khóa user thì webhook `user.disabled` được gửi với chữ ký đúng, và có retry khi client trả `500`.
- [ ] Kiểm tra trọn luồng với WeKnora trên môi trường dev: bấm "Đăng nhập bằng eUp" → vào được workspace chung.

## Thông tin cần baocao gửi lại

- [ ] Issuer và URL discovery của môi trường dev và prod.
- [ ] `client_id` và `client_secret` cho dev và prod, gửi qua kênh bảo mật (không gửi qua chat hoặc email thường).
- [ ] `webhook_secret` của client WeKnora.
- [ ] Một tài khoản test trên môi trường dev.
- [ ] Dự kiến ngày xong môi trường dev.

**Câu hỏi mở:**

- Mọi tài khoản baocao đều dùng email `@eupgroup.net` không, hay có domain khác? WeKnora sẽ chặn theo danh sách domain này.
- baocao đã xác minh email của user chưa? Nếu chưa, có thể coi email do admin tạo là đã xác minh không?
- Port và domain của môi trường dev WeKnora sẽ chốt sau, team WeKnora báo lại redirect URI chính xác.

## Kế hoạch triển khai giai đoạn 1

Giai đoạn 1 đưa baocao thành OIDC Provider cho WeKnora trong 15–22/10/2026, khoảng 25 giờ, gồm cả webhook khoá tài khoản. Task nằm ở dự án Staff It, epic EP-005; phía WeKnora theo dõi ở epic EP-012.

**Quyết định đã chốt (14/10/2026):**

- Laravel Passport `^13` + `jeremy379/laravel-openid-connect` ghim **3.0.1**. Bản 3.0.2 trở đi chỉ hỗ trợ Laravel 12/13; nâng package khi baocao lên Laravel 12.
- `sub` = `users.id` dạng chuỗi. Không đổi về sau, vì mọi dịch vụ gắn tài khoản theo `sub`.
- Webhook khoá tài khoản làm ngay trong giai đoạn 1 (đổi quyết định ngày 15/10/2026), vì WeKnora đã lên lịch phía nhận ở KB-19014, hạn 22/10. Giai đoạn 1 gửi user.disabled, user.enabled, user.updated; chưa có luồng xoá user nên chưa gửi user.deleted.

**Điểm phải xử lý trước, nếu không SSO hỏng:**

1. Package 3.0.1 không có `/oauth/userinfo` → tự viết, route tên `openid.userinfo` để discovery tự khai báo. Logout tương tự với `openid.end_session_endpoint`.
2. Issuer mặc định lấy theo host request, sau Cloudflare có thể lệch → đặt cứng `openid.issuedBy` từ env `OIDC_ISSUER`.
3. Chưa đăng nhập mà vào `/oauth/authorize` sẽ lỗi 500, vì `Authenticate::redirectTo()` trả `route('/')` không tồn tại → lưu `url.intended` rồi về trang login.
4. Guard đăng nhập là `admin` (model `Admin`), không phải `web` → `passport.guard = admin`. User hợp lệ giống hệt lúc login: `admin=1` và `active=1`.

| Mã | Việc | Ưu tiên | Giờ | Hạn |
| --- | --- | --- | --- | --- |
| KB-18997 | Cài Passport 13 + jeremy379 3.0.1 | Cao | 1.5 | 15/10 |
| KB-18998 | Key RSA ký `id_token` qua env + `kid` | Cao | 1 | 15/10 |
| KB-18999 | Issuer cố định, scopes, thời hạn token, hash secret | Cao | 1 | 16/10 |
| KB-19000 | IdentityRepository trả claims | Cao | 1.5 | 16/10 |
| KB-19001 | `/oauth/authorize`: dùng session, bỏ consent, chặn user khoá | Khẩn | 2 | 16/10 |
| KB-19002 | Endpoint `/oauth/userinfo` | Cao | 1.5 | 17/10 |
| KB-19003 | Endpoint `/oauth/logout` | TB | 1 | 17/10 |
| KB-19004 | Trang admin quản lý client + lệnh `oidc:client` | TB | 5 | 20/10 |
| KB-19005 | Rate limit login và `/oauth/token`, chống nhúng iframe | Cao | 1 | 20/10 |
| KB-19006 | Cập nhật README và GUIDE | TB | 1.5 | 21/10 |
| KB-19007 | Feature test + nghiệm thu với WeKnora dev | Cao | 3 | 21/10 |
| KB-19099 | Gửi webhook khoá/mở/cập nhật user tới các client | Cao | 5 | 22/10 |

**Ngoài giai đoạn 1:** claim `groups`, refresh token, trang consent.

**Khi deploy:** thêm env `OIDC_ISSUER`, `PASSPORT_PRIVATE_KEY`, `PASSPORT_PUBLIC_KEY`; chạy `migrate` cho các bảng `oauth_*`; kiểm tra Cloudflare không cache `/.well-known/*` và `/oauth/*`.
