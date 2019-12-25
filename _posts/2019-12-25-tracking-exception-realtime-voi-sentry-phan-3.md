---
layout: post
title:  "Tracking error real time với Sentry <br>Phần 3 — Sentry và Spring Boot"
author: halab
categories: [Monitoring, Tutorial]
tags: [Sentry, Spring Boot, Exception]
image: assets/images/spring-sentry.png
featured: true
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
        <artifactId>sentry-spring</artifactId>
        <version>1.7.27</version>
    </dependency>
    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry-logback</artifactId>
        <version>1.7.27</version>
    </dependency>
    ...
</dependencies>
```

## Configuration
### Config sentry option
Tạo file `sentry.properties` trong thư mục `src/resource` với nội dung sau:
```properties
release=1.0.0
tags=service:gateway
stacktrace.app.packages=com.github.halab4dev


async.queuesize=100
timeout=1000
```
Danh sách đầy đủ các option có thể xem tại [đây](https://docs.sentry.io/error-reporting/configuration/?platform=javascript#common-options)

### Config Spring beans
Tạo class `SentryConfiguration` với các nội dung sau:

Khởi tạo Sentry, thiếu dòng này đương nhiên là Sentry sẽ không hoạt động rồi 😀.
`DSN_URL` thì lấy ở phía trên kia nhé.
```java
@PostConstruct
    public void init() {
        Sentry.init("<DSN_URL>?environment=staging");
    }
```
Param `environment` cũng là 1 option của Sentry, 
cái này đi theo môi trường nên cả cái đoạn `<DSN_URL>?environment=staging` nên config linh động theo môi trường,
dùng biến môi trường hay đọc từ file chẳng hạn

Tiếp theo, để bắt các exception được throw ra từ controller, ta cần khai báo 1 `HandlerExceptionResolver`:
```java
    @Bean
    public HandlerExceptionResolver sentryExceptionResolver() {
        return new io.sentry.spring.SentryExceptionResolver();
    }
```

Ngoài ra, để Sentry có thể lấy được các thông tin từ HTTP request (ví dụ như User Agent), thì khai báo thêm bean này nữa nhé:
```java
    @Bean
    public ServletContextInitializer sentryServletContextInitializer() {
        return new io.sentry.spring.SentryServletContextInitializer();
    }
```

Túm cái váy lại, file `SentryConfiguration` trông sẽ như thế này:
```java
@Configuration
public class SentryConfiguration {

    @PostConstruct
    public void init() {
        Sentry.init("<DSN_URL>?environment=staging");
    }

    @Bean
    public HandlerExceptionResolver sentryExceptionResolver() {
        return new io.sentry.spring.SentryExceptionResolver();
    }

    @Bean
    public ServletContextInitializer sentryServletContextInitializer() {
        return new io.sentry.spring.SentryServletContextInitializer();
    }
}
```
### Integrating with logback
Cần lưu ý là sau khi các bean ở trên thì chỉ có thể bắt được các Exception được throw ra phía ngoài cùng thôi nhé.
Nếu có exception nào đã catch lại rồi thì chịu. Để xử lý vấn đề này thì cần phải custom lại logback mặc định của spring boot.
Tạo file `logback-spring.xml` cụng tại thư mục `src/resource`:
```xml
<configuration>
    <!--    Include spring boot default config-->
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <!--    Configure the Sentry appender, overriding the logging threshold to the WARN level -->
    <appender name="Sentry" class="io.sentry.logback.SentryAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <!--    Enable the Console and Sentry appenders, Console is provided as an example
        of a non-Sentry logger that is set to a different logging threshold -->
    <root level="INFO">
        <appender-ref ref="Sentry" />
    </root>
</configuration>
```
Ý nghĩa cái config trên cũng đơn giản thôi. Mỗi khi sử dụng hàm `log.error()` thì exception cũng được tự động gửi lên Sentry nữa.

Đấy, xong rồi. Apply Sentry vào Spring Boot cũng đơn giản mà 😘

Source code tham khảo: `https://github.com/halab4dev/spring-sentry`