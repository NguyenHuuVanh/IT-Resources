# JWT (Json web token)

## 1. Khái niệm

JWT là một tiêu chuẩn mở (RFC 7519) để truyền thông tin an toàn giữa hai bên, thường là client và server dưới dạng một đối tượng JSON. Dữ kiệu sẽ được ký số để đảm bảo tính toàn vẹn và bảo mật và xác minh được tính xác thực, giúp các ứng dụng xác thực và uỷ quyền mà không cần lưu trạng thái trên server (JWT có thể được ký bằng bí mật (bằng thuật toán HMAC) hoặc cặp khóa công khai/riêng bằng RSA hoặc ECDSA.)

## 2. Các thành phần chính của JWT

Một JWT bao gồm ba phần, được mã hóa bằng base64 và ngăn cách nhau bằng dấu chấm:

- **Header**: Chứa thông tin về loại token và thuật toán ký được sử dụng (ví dụ: HMAC, RSA). Header thông thường sẽ bao gồm 2 thành phần, gồm : Kiểu token, mà với trường hợp này luôn luôn là JWT, và thuật toán mã hoá được sử dụng, ví dụ như HMAC hoặc RSA. Do đó, Header của JWT sẽ có dạng như sau.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Sau đó, đoạn này sẽ được encoding (base64Encode) và trở thành chuỗi đầu tiên trong 3 chuỗi JWT.

- **Payload**: Chứa các "claims" (tuyên bố), bao gồm các thông tin về người dùng và quyền hạn của họ, ví dụ như người phát hành, thời gian hết hạn, các quyền được cấp. 
Payload trong JWT chứa các claims.
Trong lĩnh vực an toàn thông tin, claims đề cập đến sự tuyên bố quyền truy cập hoặc quyền sử dụng tài nguyên.
Claims là tập hợp các thông tin đại diện cho một thực thể (object) (ví dụ : user_id) và một số thông tin đi kèm. Claims sẽ có dạng Key - Value. Do đó, chúng ta có thể hiểu rằng, claims ám chỉ việc yêu cầu truy xuất tài nguyên cho object tương ứng.
Có 3 kiểu claims, bao gồm registered, public, and private claims.

  - **Registered Claims**
  Registered Claims là các thành phần được xác định trước của claims. Thành phần này mặc dù không bắt buộc, nhưng là thành phần nên có để cung cấp một số chức năng và thông tin hữu ích. 
  Một số registered claims bao gồm :
  - iss (issuer): Tổ chức, đơn vị cung cấp, phát hành JWT.
  - sub (subject): Chủ thể của JWT, xác định rằng đây là người sở hữu hoặc có quyền truy cập các resource (tài nguyên).
  - aud (audience): Được hiểu là người nhận thông tin, và có thể xác thực tính hợp lệ của JWT.
  - exp (expiration time): Thời hạn của JWT, vượt quá thời gian này, JWT được coi là không hợp lệ

  - **Public Claims**
  Public claims là các thành phần được xác định bởi người sử dụng JWT, được sử dụng rộng rãi trong JWT. Mặc dù việc sử dụng public claims không phải là bắt buộc, tuy nhiên để tránh xảy ra xung đột, public claims name được xác định theo danh sách dưới đây:

  | Claim Name | Claim Description |
  |------------|-------------------|
  | iss | Issuer |
  | sub | Subject |
  | aud | Audience |
  | exp | Expiration Time |
  | nbf | Not Before |
  | iat | Issued At |
  | jti | JWT ID |
  | name | Full name |
  | given_name | Given name(s) or first name(s) |
  | family_name | Surname(s) or last name(s) |
  | middle_name | Middle name(s) |
  | nickname | Casual name |
  | preferred_username | Shorthand name by which the End-User wishes to be referred to |
  | profile | Profile page URL |
  | picture | Profile picture URL |

  - **Private Claims**
  Các bên sử dụng JWT có thể sẽ cần sử dụng đến claims không phải là Registered Claims, cũng không được định nghĩa trước như Public Claims. Đây là phần thông tin mà các bên tự thoả thuận với nhau, không có tài liệu hay tiêu chuẩn nào dành cho private claims.
  Lưu ý: Tên claims chỉ nên dài ba ký tự vì JWT hướng tới sự nhỏ gọn.

- **Signature**: Được tạo ra bằng cách kết hợp phần header, phần payload và một khóa bí mật, sau đó sử dụng thuật toán đã chỉ định để đảm bảo dữ liệu không bị thay đổi bởi bên thứ ba. Signature là phần cuối cùng của JWT, có chức năng xác thực danh tính người gửi. Để tạo ra một signature chính xác, ta cần encode phần header, phần payload, chọn cryptography (mật mã khoá) thích hợp đã được xác định ở header kèm secret key và thực hiện sign.
- Ví dụ, nếu chúng ta lựa chọn hàm HMACSHA256,  function thực hiện sign sẽ có dạng như sau:
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
Cả bên gửi và bên nhận JWT đều sẽ xử dụng hàm này để xác định phần signature, nếu thông tin này của cả 2 bên khác nhau, JWT này được coi là không hợp lệ.
Lưu ý: Chỉ có người có secret key mới có thể sign được một signature phù hợp. Do đó, danh tính của bên gửi được đảm bảo nhờ có signature.

Vd: xxxxx.yyyyy.zzzzz

## 3. Lý do sử dụng JWT

- **Authentication (Xác thực)**: JWT được sử dụng để xác thực người dùng trước khi họ truy cập đến tài nguyên trên server.
- **Authorization (Uỷ quyền)**: Khi người dùng đăng nhập thành công, application có thể truy cập vào các tài nguyên thay mặt người dùng đó. Các ứng dụng đăng nhập một lần (Single Sign-On SSO) sử dụng JWT thường xuyên vì tính nhỏ gọn và dễ dàng triển khai trên nhiều domain.
- **Trao đổi thông tin an toàn**: JWT được coi là một cách trao đổi thông tin an toàn vì thông tin đã được signed trước khi gửi đi.

## 4. Ưu điểm của JWT

- **Gọn nhẹ**: JWT nhỏ gọn, chi phí truyền tải thấp giúp tăng hiệu suốt của các ứng dụng.
- **Bảo mật**: JWT sử dụng các mật mã khoá để tiến hành xác thực người danh tính người dùng. Ngoài ra, cấu trúc của JWT cho phép chống giả mạo nên thông tin được đảm bảo an toan trong quá trình trao đổi.
- **Phổ thông**: JWT được sử dụng dựa trên JSON, là một dạng dữ liệu phổ biến, có thể sử dụng ở hầu hết các ngôn ngữ lập trình. Ngoài ra, triển khai JWT tương đối dễ dàng và tích hợp được với nhiều thiết bị, vì JWT đã tương đối phổ biến.

## 5. Khuyết điểm của JWT

- **Kích thước**: Mặc dù trong tài liệu không ghi cụ thể giới hạn, nhưng do được truyền trên HTTP Header, vì thế, JWT có giới hạn tương đương với HTTP Header (khoảng 8KB).
- **Rủi ro bảo mật**: Khi sử dụng JWT không đúng cách, ví dụ như không kiểm tra tính hợp lệ của signature, không kiểm tra expire time, kẻ tấn công có thể lợi dụng sơ hở để truy cập vào các thông tin trái phép.

Ngoài ra, việc để thời gian hết hạn của JWT quá dài cũng có thể tạo ra kẽ hở tương tự.

## 6. Một số ứng dụng JWT

- **Single Sign-On (SSO)**: JWT có thể được sử dụng để cung cấp single sign-on cho người dùng. Điều này cho phép họ đăng nhập vào nhiều ứng dụng chỉ với một tài khoản duy nhất.
- **API Authorization**: JWT thường được sử dụng để phân quyền cho người dùng đến những tài nguyên cụ thể, từ những claims chứa trong JWT đó.
- **User Authentication**: JWT cung cấp khả năng xác thực người dùng và cấp quyền cho họ truy cập vào các tài nguyên mong muốn trong hệ thống.
- **Microservices Communication**: JWT còn có thể sử dụng cho việc giao tiếp giữa các service nhỏ trong hệ thống microservices.

## 7. Cách hoạt động

![alt text](images/image16.png)

Trong ví dụ này, trước tiên người dùng đăng nhập vào máy chủ xác thực bằng cách sử dụng hệ thống đăng nhập của máy chủ (ví dụ: username và password, Facebook login, Google login, etc). Server xác thực sau đó tạo JWT (JWT là token) và gửi nó cho người dùng. Khi người dùng thực hiện các lần gọi API đến ứng dụng, người dùng chuyển JWT cùng với cuộc gọi API. Trong thiết lập này, server của ứng dụng sẽ được cấu hình để xác minh rằng JWT đến được tạo ra bởi server xác thực. Vì vậy, khi người dùng thực hiện các cuộc gọi API đính kèm JWT, ứng dụng có thể sử dụng JWT để xác minh rằng cuộc gọi API đến từ một người dùng đã được xác thực.

Ví dụ 1: Chỉ gửi Access Token (JWT)
Server thường gói JWT trong một đối tượng JSON dưới một cái tên (key) như token hoặc accessToken:

```json
{
  "success": true,
  "message": "Đăng nhập thành công!",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

Trong trường hợp này, token chính là JWT.

Trường Hợp Gửi Hai Token (Có Thể Đây Là Điều Bạn Thắc Mắc)
Trong các hệ thống bảo mật hiện đại, để an toàn hơn, server thường gửi về HAI loại token. Đây có thể là nguồn gốc cho câu hỏi của bạn:

1. **Access Token (Thẻ truy cập)**:
   - Đây chính là cái JWT mà bạn đang hỏi.
   - Nó có thời hạn sử dụng ngắn (ví dụ: 15 phút, 1 giờ).
   - Client sẽ đính kèm token này vào mọi yêu cầu (request) gửi lên server để chứng minh mình là ai.

2. **Refresh Token (Thẻ làm mới)**:
   - Đây là một token thứ hai (có thể là JWT hoặc chỉ là một chuỗi ngẫu nhiên, dài hơn).
   - Nó có thời hạn sử dụng rất dài (ví dụ: 7 ngày, 30 ngày).
   - Nó chỉ có một mục đích duy nhất: Khi Access Token (JWT) hết hạn, client sẽ dùng Refresh Token này để bí mật yêu cầu server cấp một Access Token mới mà không cần người dùng phải đăng nhập lại.

Ví dụ 2: Gửi cả Access Token (JWT) và Refresh Token
Khi đăng nhập, server có thể trả về một cấu trúc JSON như thế này:

```json
{
  "success": true,
  "accessToken": "eyJhbGciOiJ...", // Đây là JWT, dùng trong 15 phút
  "refreshToken": "another_very_long_random_string_or_jwt..." // Dùng trong 7 ngày
}
```

Tóm lại: Server luôn gửi về JWT, và cái JWT đó chính là "token" (hay cụ thể hơn là "access token"). Đôi khi, server sẽ gửi kèm thêm một "refresh token" nữa để tăng cường bảo mật và trải nghiệm người dùng

Sự khác nhau giữa Access Token và Refresh Token

| Tiêu chí | Access Token (AT) | Refresh Token (RT) |
|----------|-------------------|-------------------|
| Mục đích | Dùng để truy cập tài nguyên (gọi API, lấy dữ liệu, xem profile...). | Dùng để lấy Access Token mới khi AT cũ hết hạn. |
| Thời gian sống | Rất ngắn (Ví dụ: 15 phút, 1 giờ). | Rất dài (Ví dụ: 7 ngày, 30 ngày, hoặc lâu hơn). |
| Tần suất sử dụng | Liên tục. Được gửi kèm trong mọi yêu cầu (request) cần xác thực. | Rất ít. Chỉ được dùng một lần duy nhất khi AT hết hạn. |
| Nơi lưu trữ (Phía Client) | Thường lưu ở bộ nhớ (RAM) của ứng dụng (ví dụ: biến JavaScript, React State, Vuex). | Phải lưu ở nơi an toàn nhất. Tiêu chuẩn là HttpOnly Cookie. |
| Rủi ro khi bị lộ | Thấp. Kẻ tấn công chỉ có quyền truy cập trong thời gian ngắn (ví dụ: 15 phút) trước khi nó hết hạn. | Rất cao. Kẻ tấn công có thể liên tục tạo AT mới và truy cập hệ thống của bạn trong thời gian dài (ví dụ: 7 ngày). |

Luồng Hoạt Động (Workflow) Của Chúng
Hãy xem chúng phối hợp với nhau như thế nào để bạn không phải đăng nhập lại liên tục:

1. **Đăng nhập**: Bạn gửi username và password cho server.
2. **Nhận Token**: Server xác thực đúng, tạo ra 1 cặp:
   - Một Access Token (AT) - có hạn 15 phút.
   - Một Refresh Token (RT) - có hạn 7 ngày.
   - Server trả về AT về trong JSON, và set RT vào một HttpOnly Cookie (để JavaScript không đọc được).
3. **Sử dụng App**:
   - Bạn vào xem trang "Thông tin cá nhân". Trình duyệt của bạn gọi API /api/profile và đính kèm AT vào Header Authorization.
   - Server nhận AT, thấy hợp lệ, trả về thông tin của bạn.
4. **Khi AT Hết Hạn**:
   - ... 20 phút sau, bạn bấm nút "Xem bài viết".
   - Trình duyệt gọi API /api/posts và vẫn đính kèm cái AT (lúc này đã hết hạn).
   - Server kiểm tra AT, thấy hết hạn, trả về lỗi 401 Unauthorized.
5. **Tự động Làm Mới (Đây là điều kỳ diệu)**:
   - Ứng dụng (client) của bạn thấy lỗi 401, nó không bắt bạn đăng nhập lại.
   - Thay vào đó, nó âm thầm gọi đến một API đặc biệt, ví dụ: /api/refresh.
   - Vì RT được lưu trong HttpOnly Cookie, trình duyệt tự động đính kèm RT này vào yêu cầu /api/refresh.
6. **Cấp Token Mới**:
   - Server nhận RT, kiểm tra xem RT này có hợp lệ và còn hạn không.
   - Nếu OK, server tạo ra một AT mới (lại có hạn 15 phút) và trả về cho client.
7. **Thử Lại**:
   - Client nhận được AT mới, lưu nó vào bộ nhớ.
   - Nó tự động gọi lại yêu cầu /api/posts (đã thất bại ở bước 4) với cái AT mới này.
   - Server nhận AT mới, thấy hợp lệ, trả về danh sách bài viết.

Kết quả: Người dùng không hề biết có lỗi 401 xảy ra. Họ chỉ thấy ứng dụng mượt mà, không bao giờ bị văng ra đăng nhập lại, dù AT chỉ sống có 15 phút.

Tóm lại: AT là "giấy thông hành" dùng hàng ngày, RT là "sổ hộ khẩu" cất kỹ ở nhà để đi gia hạn giấy thông hành khi cần.