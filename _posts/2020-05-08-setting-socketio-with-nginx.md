---
layout: post
title:  "Setting SocketIO with Nginx"
author: halab
categories: [ Tutorial ]
tags: [Nginx, SocketIO]
image: "https://seeklogo.com/images/N/nginx-logo-FF65602A76-seeklogo.com.png"
featured: true
---

Project mới lại đến đoạn deploy lên production rồi.
Có mấy cái sub domain: api, file, socketio ... nên chắc lại dùng nginx để làm proxy thôi. 
Trước làm rồi, thôi thì cứ lấy ra copy paster vậy 😜

Thêm cái file config cho từng sub domain này `/etc/nginx/conf.d/myapp-socketio.conf`
```
# Force redirect HTTP to HTTPS
server {
        listen  80;
        listen  [::]:80;
        server_name  socketio.myapp.com;

        return 301 https://socketio.myapp.com$request_uri;
}

# Pass to socketio server
server {
        listen  443 ssl http2;
        listen  [::]:443 ssl http2;
        server_name  socketio.myapp.com;

        ssl_certificate "/path/to/ssl/files/myapp.com.chained.crt";
        ssl_certificate_key "/path/to/ssl/files/myapp.com.key";

        location / {
                proxy_pass      http://127.0.0.1:3001;
        }
}
```
Ngon rồi, nghỉ thôi.

---

Few days later:
- A ơi, giúp em với, bên Android đang không kết nối socketio được. Cứ connect rồi disconnect liên tục thôi.
- Bên iOS thế nào em?
- Bên iOS thì bình thường anh. 
Em thử thay đổi version thư viện rồi mà không được. Tắc mấy ngày rồi.
- Lạ nhỉ, dự án cũ cũng setting như này, chạy ầm ầm mà nhỉ 🤔. Để thử code cái client test xem sao, mình cũng dùng Java mà .
... Oh, không được thật này.🤨

À à nhớ rồi, client cũ là react native, phía Android/ Java client có gì đó vi diệu chăng? Search cái xem có ai bị như mình không?
Ồ, có [bài hướng dẫn trên trang chủ Nginx](https://www.nginx.com/blog/nginx-nodejs-websockets-socketio/) luôn này. 
Để apply xem sao, thêm mấy cái setting loằng ngoằng này vào xem nào:
```
# Force redirect HTTP to HTTPS
server {
        listen  80;
        listen  [::]:80;
        server_name  socketio.myapp.com;

        return 301 https://socketio.myapp.com$request_uri;
}

# Pass to socketio server
server {
        listen  443 ssl http2;
        listen  [::]:443 ssl http2;
        server_name  socketio.myapp.com;

        ssl_certificate "/path/to/ssl/files/myapp.com.chained.crt";
        ssl_certificate_key "/path/to/ssl/files/myapp.com.key";

        location / {
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_http_version 1.1;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;
                proxy_pass      http://127.0.0.1:3001;
        }
}
```

- Chạy rồi, nhưng mà không hiểu lắm, sao lại phải upgrade cái gì ý nhỉ, nhưng mà thôi cứ note vào đã, tìm hiểu sau vậy =)).
À quên, test lại xem Android đc chưa em ơi!!!
- Rồi anh ơi, có ping pong rồi này.
