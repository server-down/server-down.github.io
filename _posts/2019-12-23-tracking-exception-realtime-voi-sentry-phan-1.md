---
layout: post
title:  "Tracking error real time với Sentry <br>Phần 1 — Sentry là gì?"
author: halab
categories: [Monitoring ]
tags: [Sentry, Exception]
image: assets/images/sentry-logo-black.png
featured: true
---
Trong quá trình vận hành hệ thống, không thể tránh khỏi các bug phát sinh. 
Bản thân team của mình cũng thường xuyên nhận được những feedback về các lỗi xảy ra từ phía khách hàng, từ end users. 
Đôi khi những bug này đã xảy ra suốt 1 thời gian dài mà không được phát hiện, và tất nhiên, có thể có nhiều bug khác cũng đang lẩn trốn đâu đó nữa.

Những lúc như vậy, công việc mà dev hay làm là remote lên server, kéo file log về để xem app của mình có “trăn trối” lại cái Exception nào không. 
Nhưng việc tìm kiếm trong đống file log không dễ dàng tý nào, 
chưa kể nếu hệ thống có setting log rotate thì có thể các thông tin cần thiết đã bị dọn đi từ lúc nào rồi không biết.

Thế rồi một ngày đẹp trời thấy ông devops của cty có dùng 1 tool tên là Sentry để monitoring exception và thế là ... ăn trộm ngay 😆. 
Giờ thì mỗi khi app của mình bắn ra exception thì Sentry đều log lại, có thể group các exception giống nhau, 
qua đó ta có thể thấy được exception này đã xảy ra bao nhiêu lần. Có bao nhiêu user đã bị ảnh hưởng bởi nó. 
Và có thể gửi thông báo qua mail, slack … team dev có thể detech và fix bug sớm trước cả khi user nhận ra nũa. 
User vui, khách hàng có doanh thu, sếp nở mày nở mặt, team … được yên ổn 😂.

Cùng điểm qua 1 số nét chính nhé:
![](https://sentry.io/_assets/screenshots/features-page-dash-12c65431808e7d8daf234a096446c1f0da311a0f3bcec5352e28bda60136fb16.jpg)
Danh sách exception được log lại này

![](https://miro.medium.com/max/2662/1*tDbvXZyleARaJJYvHdpHzQ.png)
Chi tiết exception xảy ra ở dòng code nào cũng đc ghi lại rất cụ thể

![](https://sentry.io/_assets/screenshots/slack/error-notification-1ae4a0cc045bd99ade95c0ac37864e37bf2b29585690025454fd4d25cb5e0f1c.png)
Thông báo qua Slack luôn nhé

Nghe hấp dẫn phải không? Thế thì còn chờ gì mà không [install]({% post_url 2019-12-24-tracking-exception-realtime-voi-sentry-phan-2 %}) đi nào.