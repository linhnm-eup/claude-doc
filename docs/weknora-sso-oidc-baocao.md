# WeKnora — Hướng dẫn cấu hình SSO OIDC với baocao

Oct 15, 2026 · @Linh NM eUp

## Điều kiện trước khi bật

SSO chỉ hoạt động khi có đủ ba điều kiện dưới đây. Deploy code mà chưa đặt biến nào thì đăng nhập mật khẩu vẫn chạy như cũ, chỉ khác ở hai điểm luôn bật: phiên tối đa 7 ngày và user bị khóa mất quyền ngay.

- [ ] **baocao production đã có OIDC.** `https://baocao.eup.ai/.well-known/openid-configuration` trả JSON, không phải 404. Ngày 15/10/2026 production vẫn trả 404, baocao mới lên dev.
- [ ] **Đã có client `weknora`** trên baocao production (Bước 1).
- [ ] **Nhánh `feat/sso-baocao` đã merge vào `main`** (commit `9ff4bf7`, `dd3d9bb`).

## Bước 1: Đăng ký client trên baocao

Admin baocao tạo client ở trang quản lý SSO client hoặc bằng lệnh `php artisan oidc:client` (task KB-19004). Hai URI phải khớp **từng ký tự**.

| Thuộc tính | Giá trị |
| --- | --- |
| Tên | eUp Knowledge Hub (WeKnora) |
| `client_id` | `weknora` (phải đúng như vậy: WeKnora kiểm tra `aud` của `id_token` bằng client\_id) |
| Redirect URI | `https://wiki.eup.ai/api/v1/auth/oidc/callback` |
| Post-logout redirect URI | `https://wiki.eup.ai/login` |
| Scopes | `openid profile email` |

Ghi lại `client_secret`. Secret chỉ hiện một lần và chỉ được chuyển qua kênh bảo mật.

## Bước 2: Tạo workspace chung và lấy tenant id

User mới đăng nhập qua SSO sẽ tự được thêm vào workspace này. Làm bước này trên wiki.eup.ai bằng tài khoản admin hiện có, trước khi bật SSO.

1. Đăng nhập bằng tài khoản admin, mở menu user rồi chọn tạo workspace mới, đặt tên `eUp`. Người tạo trở thành Owner.
2. Chuyển sang workspace `eUp`.
3. Mở DevTools → Console, chạy lệnh dưới đây để lấy id:

```javascript
localStorage.getItem('weknora_selected_tenant_id')
```

Số trả về là giá trị của `OIDC_AUTH_DEFAULT_TENANT_ID` ở Bước 3. Nếu muốn có thêm người quản trị workspace, sau khi họ đăng nhập SSO lần đầu, Owner nâng role của họ lên Admin trong phần thành viên.

## Bước 3: Biến môi trường

Các biến này đặt cho container `app` (deployment `weknora-app`). Container frontend không cần đổi gì. Endpoint authorize, token, userinfo và JWKS được WeKnora tự đọc từ discovery, riêng endpoint logout phải khai báo tay.

| Biến | Giá trị | Loại |
| --- | --- | --- |
| `OIDC_AUTH_ENABLE` | `true` | Thường |
| `OIDC_AUTH_ISSUER_URL` | `https://baocao.eup.ai` (không có `/` cuối) | Thường |
| `OIDC_AUTH_DISCOVERY_URL` | `https://baocao.eup.ai/.well-known/openid-configuration` | Thường |
| `OIDC_AUTH_CLIENT_ID` | `weknora` | Thường |
| `OIDC_AUTH_CLIENT_SECRET` | Secret từ Bước 1 | **Secret** |
| `OIDC_AUTH_PROVIDER_DISPLAY_NAME` | `eUp` (chữ trên nút đăng nhập) | Thường |
| `OIDC_AUTH_SCOPES` | `openid profile email` | Thường |
| `OIDC_USER_INFO_MAPPING_USER_NAME` | `name` | Thường |
| `OIDC_USER_INFO_MAPPING_EMAIL` | `email` | Thường |
| `OIDC_AUTH_ALLOWED_EMAIL_DOMAINS` | `eupgroup.net` (nhiều domain thì cách nhau bằng dấu phẩy) | Thường |
| `OIDC_AUTH_REQUIRE_EMAIL_VERIFIED` | `true` | Thường |
| `OIDC_AUTH_DEFAULT_TENANT_ID` | Id từ Bước 2 | Thường |
| `OIDC_AUTH_DEFAULT_ROLE` | `viewer` (hoặc `contributor`, `admin`; không cho `owner`) | Thường |
| `OIDC_AUTH_AUTO_REDIRECT` | `true` | Thường |
| `OIDC_AUTH_END_SESSION_ENDPOINT` | `https://baocao.eup.ai/oauth/logout` | Thường |
| `DISABLE_REGISTRATION` | `true` | Thường |
| `JWT_SECRET` | Chuỗi ngẫu nhiên dài, ví dụ `openssl rand -hex 32` | **Secret** |

Không để trống `JWT_SECRET`, cũng không dùng giá trị mẫu như `CHANGE-ME-jwt-secret`. Thiếu biến này thì mọi phiên đăng nhập mất sau mỗi lần pod khởi động lại. Nếu cluster đã có `JWT_SECRET` hợp lệ thì giữ nguyên, đổi giá trị sẽ làm mọi người bị đăng xuất.

`OIDC_AUTH_AUTO_REDIRECT` và `OIDC_AUTH_END_SESSION_ENDPOINT` phải đi cùng nhau. Thiếu endpoint logout thì bấm đăng xuất xong người dùng bị đăng nhập lại ngay.

## Bước 4: Áp biến vào k8s

**Chỉ làm bước này sau khi Bước 5 đã deploy xong image mới.**

WeKnora chạy trên cluster `k8s-local-dev`, namespace `eup-internal`. Manifest gốc (secret, ingress, deployment) **không nằm trong repo** mà ở `/home/rio/k8s_pod/move_weknora` trên máy của phucnh. Jenkins chỉ đổi image, không đụng tới biến môi trường.

Cách khuyến nghị là sửa thẳng manifest gốc rồi `kubectl apply`, để lần apply sau không ghi đè mất cấu hình. Nếu cần làm nhanh bằng lệnh thì chạy trên host có kubectl (`ansible@192.168.10.137`), sau đó vẫn phải chép các giá trị này vào manifest gốc:

```bash
K="kubectl --context k8s-local-dev -n eup-internal"

# 1. Kiểm tra JWT_SECRET hiện tại (chỉ xem có hay không, không in giá trị)
$K exec deploy/weknora-app -c app -- sh -c '[ -n "$JWT_SECRET" ] && echo JWT_SECRET_SET || echo JWT_SECRET_MISSING'

# 2. Secret cho SSO (thay <...>)
$K create secret generic weknora-sso \
  --from-literal=OIDC_AUTH_CLIENT_SECRET='<client_secret từ baocao>'

# 3. Gắn secret và các biến thường vào deployment (pod sẽ tự khởi động lại)
$K set env deploy/weknora-app -c app --from=secret/weknora-sso
$K set env deploy/weknora-app -c app \
  OIDC_AUTH_ENABLE=true \
  OIDC_AUTH_ISSUER_URL=https://baocao.eup.ai \
  OIDC_AUTH_DISCOVERY_URL=https://baocao.eup.ai/.well-known/openid-configuration \
  OIDC_AUTH_CLIENT_ID=weknora \
  OIDC_AUTH_PROVIDER_DISPLAY_NAME=eUp \
  OIDC_AUTH_SCOPES="openid profile email" \
  OIDC_USER_INFO_MAPPING_USER_NAME=name \
  OIDC_USER_INFO_MAPPING_EMAIL=email \
  OIDC_AUTH_ALLOWED_EMAIL_DOMAINS=eupgroup.net \
  OIDC_AUTH_REQUIRE_EMAIL_VERIFIED=true \
  OIDC_AUTH_DEFAULT_TENANT_ID=<id từ Bước 2> \
  OIDC_AUTH_DEFAULT_ROLE=viewer \
  OIDC_AUTH_AUTO_REDIRECT=true \
  OIDC_AUTH_END_SESSION_ENDPOINT=https://baocao.eup.ai/oauth/logout \
  DISABLE_REGISTRATION=true

# 4. Chờ rollout xong (app chạy migration khi khởi động, có thể mất vài phút)
$K rollout status deploy/weknora-app --timeout=600s
```

Nếu lệnh kiểm tra in ra `JWT_SECRET_MISSING`, thêm `JWT_SECRET` vào cùng secret `weknora-sso` (`openssl rand -hex 32`) trước khi chạy bước 3. Nếu đã có sẵn thì giữ nguyên.

## Bước 5: Merge, tag và deploy

**Thứ tự đúng là deploy code mới trước, đặt biến (Bước 4) sau.** Image cũ đã hỗ trợ OIDC cơ bản nhưng chưa có kiểm tra domain và workspace chung. Nếu bật `OIDC_AUTH_ENABLE` trên image cũ, mỗi user baocao đăng nhập sẽ được tạo một workspace riêng.

1. Merge `feat/sso-baocao` vào `main`. Push vào `main` không tự deploy.
2. Tạo tag `v*` trên `main`, ví dụ:

```bash
git checkout main && git pull
git tag v0.8.1-sso && git push origin v0.8.1-sso
```

3. Jenkins (multibranch, discover tags) tự build. Lần này sẽ build lại hai thành phần `app` và `ui`, `docreader` giữ nguyên. Sau đó Jenkins deploy lên `eup-internal` và smoke test `GET /api/v1/auth/config`.
4. Sau khi Jenkins xanh thì làm Bước 4.

Tag trên chỉ là ví dụ. Hãy đặt theo quy ước version của team.

## Bước 6: Kiểm tra sau deploy

- [ ] `curl -s https://wiki.eup.ai/api/v1/auth/oidc/config` trả `enabled: true`, `auto_redirect: true`, có `end_session_endpoint`.
- [ ] Mở `https://wiki.eup.ai` bằng cửa sổ ẩn danh: tự chuyển sang trang đăng nhập baocao.
- [ ] Đăng nhập bằng tài khoản baocao lần đầu: quay về WeKnora, vào đúng workspace `eUp`, role `viewer`.
- [ ] Tài khoản đã có trên WeKnora (cùng email) đăng nhập SSO: vào đúng tài khoản cũ, giữ nguyên dữ liệu.
- [ ] Bấm đăng xuất: chuyển qua baocao logout rồi về `/login`, không tự đăng nhập lại.
- [ ] `https://wiki.eup.ai/login?local=1` vẫn hiện form mật khẩu, admin đăng nhập được.
- [ ] Không còn tab Đăng ký trên trang login.

## Tắt SSO hoặc rollback

Tắt SSO không cần rollback code, chỉ cần tắt biến. User đã tạo qua SSO vẫn còn, nhưng họ không có mật khẩu, nên phải dùng chức năng đặt lại mật khẩu của admin nếu muốn họ đăng nhập kiểu cũ.

```bash
K="kubectl --context k8s-local-dev -n eup-internal"
# Chỉ bỏ tự chuyển trang, giữ nút Đăng nhập bằng eUp
$K set env deploy/weknora-app -c app OIDC_AUTH_AUTO_REDIRECT=false
# Tắt hẳn SSO
$K set env deploy/weknora-app -c app OIDC_AUTH_ENABLE=false
```

Rollback code: deploy lại tag trước đó qua Jenkins. Trước khi rollback phải tắt `OIDC_AUTH_ENABLE`, vì image cũ không có kiểm tra domain và workspace chung.

## Xử lý sự cố

Lỗi OIDC hiện thành thông báo trên trang login, chi tiết xem trong log: `$K logs deploy/weknora-app -c app | grep -i oidc`.

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
| --- | --- | --- |
| baocao báo `invalid redirect_uri` | Redirect URI đăng ký khác từng ký tự | Đăng ký đúng `https://wiki.eup.ai/api/v1/auth/oidc/callback` |
| `OIDC discovery request failed` | Sai `OIDC_AUTH_DISCOVERY_URL`, hoặc baocao production chưa có OIDC | Mở URL discovery trên trình duyệt, phải ra JSON |
| `failed SSRF validation` | Pod phân giải `baocao.eup.ai` ra IP nội bộ | Thêm `baocao.eup.ai` vào biến `SSRF_WHITELIST` |
| `id_token verification failed` | `iss` khác `OIDC_AUTH_ISSUER_URL` (thường do dấu `/` cuối), `aud` khác `weknora`, hoặc JWKS không có `kid` trùng | So `id_token` trên jwt.io với cấu hình |
| `Email domain is not allowed` | Email không thuộc `OIDC_AUTH_ALLOWED_EMAIL_DOMAINS` | Thêm domain, hoặc cho user đó dùng `/login?local=1` |
| `Email is not verified by the identity provider` | baocao không trả `email_verified: true` | baocao sửa claim (KB-19000) |
| Đăng xuất xong bị đăng nhập lại ngay | Thiếu `OIDC_AUTH_END_SESSION_ENDPOINT`, hoặc baocao chưa đăng ký post-logout URI | Đặt biến và đăng ký `https://wiki.eup.ai/login` |
| User mới vào trang tạo workspace thay vì `eUp` | Sai `OIDC_AUTH_DEFAULT_TENANT_ID`, hoặc thêm thành viên lỗi | Kiểm tra id, Owner thêm user thủ công |
| Login tự chuyển không chạy, chỉ còn nút bấm | Tab đã gặp một lần SSO lỗi nên chủ động dừng để tránh lặp | Đóng tab hoặc đăng nhập thành công một lần |
| Mọi người bị đăng xuất sau khi pod khởi động lại | `JWT_SECRET` trống hoặc là giá trị mẫu | Đặt `JWT_SECRET` cố định |
