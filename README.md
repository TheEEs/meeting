# Phần mềm hội nghị trực tuyến - JitsiMeet

## Hướng dẫn cài đặt

1. Trước tiên, hãy đảm bảo bạn đã đăng ký một tên miền và trỏ nó về máy chủ của bạn.
2. Sau đó, sử dụng SSH để login vào máy chủ, cài đặt `docker` và `docker-compose`.
3. Clone thư mục này, sau đó di chuyển vào thư mục bằng cách `cd jitsimeet-docker`.
4. Tạo một file `.env` bằng lệnh `cp env.example .env`.
5. Tạo các thư mục liên quan: `mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}`

6. Chỉnh sửa file `.env`
Bạn cần setup vài thông số trước khi hệ thống có thể hoạt động. Hãy vào file `.env` và thêm các biến môi trường sau:
```env
PUBLIC_URL=https://<tên miền bạn đã đăng ký>
DEFAULT_LANGUAGE=<Ngôn ngữ mặc định mà bạn muốn dùng, tiếng việt là `vi`>
ENABLE_LETSENCRYPT=1 #Sử dụng để tạo chứng chỉ SSL
LETSENCRYPT_DOMAIN=<thay thế bằng tên miền của bạn>
LETSENCRYPT_EMAIL=<thay thế bằng email của bạn>
```

7. Trong thư mục hiện hành, gõ lệnh `docker-compose up -d` để chạy các container đã liệt kê. Chờ khoảng vài phút để hệ thống đăng ký chứng chỉ SSL, sau đó truy cập vào `https://<tên miền của bạn>` và thử tạo một hội nghị. Nếu bạn có thể tạo một cuộc họp mà không gặp vấn đề gì, bạn có thể tiếp tục các bước tiếp theo.

## Cài đặt COTURN SERVER

Khi các máy khách tham gia vào cùng một hội nghị, trình duyệt web sẽ cố gắng kết nối trực tiếp các máy với nhau thông qua một kết nối trực tiếp (không qua trung gian) sử dụng công nghệ WebRTC. Tuy nhiên không phải lúc nào kết nối trực tiếp này cũng là khả thi, vì các máy khách có thể nằm sau tường lửa. Để tránh việc đó, cần có một server làm trung gian kết nối giữa các máy khách. Server đó là TURN Server. Chúng ta sẽ sử dụng Coturn làm máy chủ TURN.

1. Thêm service sau vào trong file `docker-compose.yml`:

```yml
coturn:
        image: coturn/coturn:4.7.0-alpine
        environment:
            - DETECT_EXTERNAL_IP=yes
            - DETECT_RELAY_IP=yes
        network_mode: host
        volumes:
            - ${CONFIG}/coturn/turnserver.conf:/etc/coturn/turnserver.conf:Z
            - ${CONFIG}/web/acme-certs:/etc/acme-certs
        depends_on:
            - web
```

2. Cài đặt tools `pwgen` (eg: `sudo apt-get install -y pwgen).

3. Tạo password: `pwgen -s 64 1`. Một password được in ra terminal.

4. Cập nhật password vào trong file `.jitsi-meet-cfg/coturn/turnserver.conf`: 
```env
static-auth-secret=<password vừa tạo>
```

5. Cập nhật chứng chỉ SSL cho Coturn trong `.jitsi-meet-cfg/coturn/turnserver.conf`:
```env
cert=/etc/acme-certs/<tên miền của bạn>/fullchain.pem
pkey=/etc/acme-certs/<tên miền của bạn>/key.pem
```

6. Cập nhật lại file `.env`:

```env
TURN_CREDENTIALS=replace.with.your.turn.credentials
TURN_HOST=replace.with.your.turn.host
TURNS_HOST=replace.with.your.turn.host
TURN_PORT=3478
TURNS_PORT=5349
JVB_STUN_SERVERS=replace.with.your.stun.servers:3478
```

7. Khởi động lại toàn bộ dịch vụ của bạn
```bash
docker-compose down && docker-compose up -d
```

Chúc mừng, bạn đã setup thành công một ứng dụng họp trực tuyến

## Tuỳ chỉnh ứng dụng

Trong thư mục `.jitsi-meet-cfg`. Bạn sẽ thấy một số thư mục con chứa các file cấu hình hệ thống.

Để thay đổi ngôn ngữ, hãy vào thư mục `.jitsi-meet-cfg/lang`.

Để thay đổi giao diện và hành vi của ứng dụng, hãy vào thư mục `.jitsi-meet-cfg/web`. Bạn sẽ tìm thấy file `custom-config.js` và `custom-interface_config.js`...etc...