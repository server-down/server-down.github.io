---
layout: post
title:  "Bảo mật Elasticsearch và Kibana với X-Pack"
author: halab
categories: [ Tutorial ]
tags: [Kibana, Elasticsearch, X-Pack]
image: https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/blta053a53f023ccc57/5bbedb3a6c9763b95d07abdc/blog-x-pack-thumb.jpg
featured: true
---

Đợt vừa rồi mình vừa mới phải cài lại Elasticsearch với Kibana cho hệ thống monitoring 
(vì sao phải cài lại thì nó lại là 1 câu chuyện buồn vô cùng dài dòng khác 🤣)
Trước đây không hề có tí bảo mật nào cả, tức là chỉ cần có địa chỉ Kibana là vào xem vô tư luôn, 
như thế thì không ổn chút nào, đã mất công cài lại thì phải chuyên nghiệp hơn chứ nhỉ. 

Thực ra trước đây đã lọ mọ 1 lần rồi, biết là Elastic stack có thằng X-Pack có cung cấp các cơ chế login, phân quyền
người dùng... nhưng bỏ cuộc sớm vì document của bọn Elastic quá tệ, 
lần này thì quyết tâm có thành quả để còn làm bài hướng dẫn =))

### So, what is X-Pack?
![](https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/bltfcbfce9a6b3fe7ee/5bbf193b192fad64364a51cb/blog-machine-learning-5-4-release.png)

X-Pack là 1 extension của Elastic Stack, cung cấp các chức năng liên quan đến bảo mật, alert, monitoring, báo cáo, 
machine learning và nhiều thứ khác nữa. Mặc định khi cài Elasticsearch thì X-Pack cũng đã được cài đặt rồi. 

### Enable X-Pack

Rồi, giờ thì mình sẽ enable X-Pack lên nhé.

- Sau khi cài đặt xong Elasticsearch và Kibana, vào file `/etc/elasticsearch/elasticsearch.yml`, thêm dòng config này vào
```yml
xpack.security.enable: true
```

- Tiếp theo thì set password cho 2 user elastic và kibana. Mặc định Elastic đã có sẵn 1 số 
[built-in user](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html), 
trong đó `elastic` là super user còn `kibana` là user mà Kibana dùng để connect và giao tiếp với  Elasticsearch. 
Chạy lệnh sau và set password cho từng user:
```shell
$ cd /usr/share/elasticsearch/
$ bin/elasticsearch-setup-passwords interactive
```

- Thêm các config sau vào file `/etc/kibana/kibana.yml`
```yml
xpack.security.authProviders: [basic]
xpack.security.encryptionKey: <ANY_STRING_WITH_AT_LEAST_32_CHARACTER>
elasticsearch.username: "kibana"
elasticsearch.password: "<KIBANA_USER_PASSWORD>"
```

- Restart Elasticsearch và Kibana
```shell
$ service elasticsearch restart
$ service kibana restart
```

- Truy cập kibana, giờ thì sẽ có 1 login page đẹp đẹp hiện ra :D
![](/assets/images/kibana_login_page.png)

- Đăng nhập với user `elastic`, vào page `Management/Security/User`, với
ở đây thì có thể tạo ra các roles/user phù hợp như cầu của mình nhé (có những quyền hạn gì với index nào)
![](https://www.elastic.co/guide/en/elasticsearch/reference/current/security/images/management-builtin-users.jpg)

Done 😘