---
layout: post
title:  "Tracking error real time với Sentry <br>Phần 2 — Install Sentry với Docker"
author: halab
categories: [Monitoring, Tutorial]
tags: [Sentry, Docker]
image: assets/images/sentry-docker.jpeg
featured: false
---

Updated: 22/6/2021
---

Tại sao phải tự cài Sentry ý hả? Vì mặc dù Sentry.io có cung cấp dịch vụ cloud nhưng gói free chỉ có 5k event 1 tháng. 
Và với cái dự án mà không có nhiều ngân sách + code lởm khởm bắn exception như lá rụng mùa thu này 
thì chỉ có cách là tự cài Sentry mà dùng thôi 😭.

## Requirements

- Docker 19.03.6+
- Compose 1.24.1+
- 4 CPU Cores
- 8 GB RAM
- 20 GB Free Disk Space

## Install

Đầu tiên là clone cái repo này về: `https://github.com/getsentry/onpremise` (ở thời điểm viết bài này là 21.5.1). Trong này mình sẽ cần chú ý 2 file thôi:

- `sentry/config.yml`: dùng để config mail, slack.
- `install.sh`: dùng để cài đặt sentry

### Config mail

Sentry có 1 cái hơi dở là không cho admin tạo account. 
Thế nên để mọi người trong team có tài khoản riêng thì mình phải config mail cho Sentry, dùng để gửi mail invite về cho member. 
Trong cái file config.yml, tìm đến đoạn Mail Server, điền các thông tin sau:
```yml
###############
# Mail Server #
###############

mail.backend: 'smtp'  # Use dummy if you want to disable email entirely
mail.host: '<YOUR_MAIL_HOST>'
mail.port: <YOUR_MAIL_PORT>
mail.username: '<YOUR_MAIL_ADDRESS>'
mail.password: '<YOUR_MAIL_PASSWORD>'
# mail.use-tls: false
# The email address to send on behalf of
mail.from: 'sentry@yourdomai.com'
```

### Install and Run

```shell
$ sh ./install.sh
$ docker-compose up -d
```

Sau bước này thì chúng ta đã có thể truy cập Sentry ở địa chỉ <HOST_ADDRESS>:9000

## Create member account

Để invite member vào sentry, vào `Setting` > `Team` > `Add Member` > `Invite Members`. Điển mail, role và team rồi bấm send là được

Vậy nếu đã config mail ở trên và làm theo các thao tác ở trên mà member trong team vẫn không nhận được mail thì sao?
Nghe vô lý nhưng mình đã bị như vậy thật đấy 🤣. Sau 1 hồi (khá lâu) ngồi mò thì phát hiện ra sentry có support tạo account qua cli.

Vào container docker chạy sentry nhé
```shell
$ docker exec -it <CONTAINER_NAME> bash
```

Tạo account:
```shell
$ sentry createuser
```
Điền mail với password là xong nhé.

OK cài xong nhé 😘. Cài xong thì làm gì? Thì tích hợp vào code thôi. Let's go.
