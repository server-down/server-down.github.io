---
layout: post
title:  "Tracking error real time với Sentry <br>Phần 3 — Sentry và Spring Boot"
author: halab
categories: [Monitoring, Tutorial]
tags: [Sentry, Spring Boot, Exception]
image: assets/images/spring-sentry.png
featured: false
---

Updated: 22/6/2021
---

Sau khi cài đặt Sentry thành công thì giờ là lúc tích hợp vào project của mình rồi.
## Tạo project trên Sentry
- Vào mục `Project` từ left menu. Chọn `Create Project`
![](https://docs.sentry.io/assets/guides/integrate-frontend/create-new-project-01-065112a0411889cfbf165aebc89e15b3421eb5f522b686b95d49e272440edae5.png)

- Lựa chọn ngôn ngữ và đặt tên cho project.
![](https://docs.sentry.io/assets/guides/integrate-frontend/create-new-project-02-6de02f9c01096e0a9096c1803e7e563dc5f9344a792b7c4fd6bce035438b14ec.png)

- Assign project cho 1 team và bấm `Create Project`
- Sau khi project được tạo, vào `Project Setting` > `Client Key`. Ở đây sẽ lấy được DSN link dùng để config trong project nhé:
![](https://docs.sentry.io/assets/guides/integrate-frontend/create-new-project-04-8a5f184782b4860cd5594c8f7973a1d5df6e9f5923f85fd08d3bffa1c4ff0d0f.png)

- OK, xong. Giờ chuyển qua phần tích hợp vào project nào.

## Dependencies
Đầu tiên là thêm dependencies vào file `pom.xml`:
```xml
<dependencies>
    ...
    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry-spring-boot-starter</artifactId>
        <version>5.0.1</version>
    </dependency>
    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry-logback</artifactId>
        <version>5.0.1</version>
    </dependency>
    ...
</dependencies>
```

## Configuration
### Config sentry option
Trong file `application.yml` trong thư mục `src/resource` thêm các nội dung sau:
```yml
sentry:
  dsn: <DSN_URL>
  environment: <ENVIRONMENT>
  tags:
    <some tag name>: <tag value>
```
Danh sách đầy đủ các option có thể xem tại [đây](https://docs.sentry.io/platforms/java/guides/spring-boot/configuration/)

### Config Spring beans
Tạo class `SentryConfiguration` và thêm đoạn code sau để có thể gắn kèm các thông tin của user khi catch exception:
```java
import org.springframework.stereotype.Component;
import io.sentry.core.protocol.User;
import io.sentry.spring.SentryUserProvider;

@Component
class CustomSentryUserProvider implements SentryUserProvider {
  public User provideUser() {
    User user = User();
    // ... set user information
    return user;
  }
}
```

### Integrating with logback
Cần lưu ý là sau khi các bean ở trên thì chỉ có thể bắt được các Exception được throw ra phía ngoài cùng thôi nhé.
Nếu có exception nào đã catch lại rồi thì chịu. Để xử lý vấn đề này thì cần phải custom lại logback mặc định của spring boot.
Tạo file `logback-spring.xml` cụng tại thư mục `src/resource`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml" />

  <appender name="SENTRY" class="io.sentry.logback.SentryAppender" />

  <root level="info">
    <appender-ref ref="CONSOLE" />
    <appender-ref ref="SENTRY" />
  </root>
</configuration>
```
Ý nghĩa cái config trên cũng đơn giản thôi. Mỗi khi sử dụng hàm `log.error()` thì exception cũng được tự động gửi lên Sentry nữa.

Đấy, xong rồi. Apply Sentry vào Spring Boot cũng đơn giản mà 😘

Source code tham khảo: `https://github.com/halab4dev/spring-sentry`