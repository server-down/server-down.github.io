---
layout: post
title:  "Monitoring Redis Performance"
author: halab
categories: [ Monitoring ]
tags: [Monitoring, Redis]
image: "https://dwglogo.com/wp-content/uploads/2017/12/Redis_Logo.png"
featured: true
---
Từ trước đến nay mình không quan tâm lắm đến performace của Redis, 
đơn giản vì mặc định trong đầu nó là thằng xử lý performance quá tốt rồi.
Đáp ứng hàng ngàn request/s, các phép tính toàn là O(1) với O(log n).

Tuy nhiên, đợt vừa rồi mới gặp phải issue liên quan đến performance của nó,
nên đầu óc phải thay đổi 1 chút rồi. 
Sau quãng thời gian "mất ăn mất ngủ" thì cũng thu được 1 ít kinh nghiệm xương máu (như mọi lần),
nên phải note ra cho nhớ.

### Redis Persistence
Redis không chỉ lưu trữ dữ liệu trên RAM mà còn lưu cả vào ổ cứng nữa. 
Mặc định cứ sau 1 khoảng thời gian Redis sẽ lưu lại dữ liệu hiện có trên RAM vào file `/var/lib/redis/dump.rdb`.
Ở trong file config của redis, bạn có thể bắt gặp dòng `save 60 1000`, này có nghĩa là redis sẽ save data mỗi 60s nếu có 1000 thay đổi trong dataset.
VIệc này giúp backup data mới nhất vào ổ đĩa, tránh thất thoát khi có sự cố (redis chết, server restart ...). 
Cơ chế này gọi là `snapshotting`.

Nghe hay đấy chứ, có vấn đề gì đâu nhỉ? 

Quay lại với cái server khốn khổ của mình nào. Bỗng nhiên một ngày đẹp trời, bỗng thấy nó có hiện tượng "nấc cụt" 😂.
App đang dùng bình thường, bỗng nhiên bị đơ khoảng tầm 1", việc này lặp lại khá thường xuyên, có vẻ có tính chu kỳ nữa.
Sau 1 hồi mò mẫm thì phát hiện ra những lúc app bị đơ cũng là lúc Redis tiến hành save date vào file =)).
Những lúc này, redis chiếm hết IO, khiến server app không thể xử lý request được nữa. (Vâng, vẫn là bối cảnh khốn khó, không có server để tách riêng con redis ra).

Vậy xử lý như thế nào giờ?

1. Tắt chức năng persistence của redis đi, vào `redis-cli` và gõ lệnh 
```
CONFIG SET SAVE ""
```
Nhớ sửa cả trong file config nữa nhé

2. Nếu vẫn muốn backup data của redis, thì có thể tạo cronjob chạy vào lúc ít ngừơi dùng, gọi đến lệnh save của redis

3. Nếu có điều kiện, hãy tách redis ra 1 server riêng, không để chung với app server.

### Memory Used
#### Memory Metrics
Để lấy thông tin các memory metric của redis, gõ lệnh `info memory`:
```
# Memory
used_memory:1073643192
used_memory_human:1023.91M
used_memory_rss:1100926976
used_memory_rss_human:1.03G
used_memory_peak:1082025392
used_memory_peak_human:1.01G
used_memory_peak_perc:99.23%
used_memory_overhead:303943696
used_memory_startup:524192
used_memory_dataset:769699496
used_memory_dataset_perc:71.73%
allocator_allocated:1073718776
allocator_active:1075978240
allocator_resident:1104146432
total_system_memory:4147884032
total_system_memory_human:3.86G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:1073741824
maxmemory_human:1.00G
maxmemory_policy:allkeys-lru
allocator_frag_ratio:1.00
allocator_frag_bytes:2259464
allocator_rss_ratio:1.03
allocator_rss_bytes:28168192
rss_overhead_ratio:1.00
rss_overhead_bytes:-3219456
mem_fragmentation_ratio:1.03
mem_fragmentation_bytes:27305704
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:133456
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
```

Ở đây, có 1 số metric cần quan tâm:

- `used_memory_rss`/`used_memory_rss_human` (rss: resident set size): số memory mà HĐH cấp phát cho Redis
- `used_memory`/`used_memory_human`: số memory thực tế mà Redis đang sử dụng
- `mem_fragmentation_ratio`: Tỷ lệ `used_memory_rss/used_memory`. 
Nếu tỉ lệ này lớn hơn 1, có nghĩa là đang có hiện tượng phân mảnh memory, 
nếu từ 1.5 trở lên tức là đang có phân mảnh memory nghiêm trọng.
Nếu tỉ lệ này nhỏ hơn 1, nghĩa là Redis đang cần thêm RAM nhưng không được cấp phát, 
trường hợp này sẽ phải dùng đến swap, nhưng dùng swap thì sẽ ảnh hưởng đến độ trễ khi xử lý.
Thông thường, `mem_fragmentation_ratio` sẽ nằm giữa `1` và `1.5`, 
nếu vượt quá, có thể cân nhắc restart redis để HĐH cấp phát lại bộ nhớ từ đầu.

#### Transparent Huge Page
Ở trường hợp của mình, sau khi restart redis được vài ngày thì thằng `mem_fragmentation_ratio` lại ngóc đầu lên.
Chắc là có cái gì sai sai rồi. Dạo qua [trang chủ](https://redis.io/topics/latency#latency-induced-by-transparent-huge-pages) thì thấy bảo cần tắt setting transparent hupe page đi.
Và quả thật, it works like a charm! 😍. Fragment ratio từ đó rất ổn định ở mức `1` - `1.3`

Ngoài ra thằng này cũng là nguyên nhân gây chậm khi redis tiến hành backup dataset vào hard disk, thế cho nên thằng này phải tắt ngay khi nghĩ đến dùng Redis

### Slow Log
Nếu muốn monitor log bị chậm của redis thì sử dụng [slowlog](https://redis.io/commands/slowlog) command.
Configure slow log với 2 parameter: `slowlog-log-slower-than` - set thời gian xử lí tối thiểu của query sẽ được lưu, và `slowlog-max-len` - số lượng slow log tối đa. Ví dụ như sau: 
```shell
CONFIG GET slowlog-log-slower-than
CONFIG SET slowlog-log-slower-than 10000
CONFIG GET slowlog-max-len
CONFIG SET slowlog-max-len 256
```
Đọc slow log với câu lệnh `SLOWLOG GET`. Mặc định thì câu lệnh này sẽ trả về toàn bộ slow log, nếu muốn giới hạn số lượng trả về, ví dụ là 2 giá trị thì sử dụng `SLOWLOG GET 2`. Sample output như sau:
```shell
redis 127.0.0.1:6379> SLOWLOG GET 2
1) 1) (integer) 14
   2) (integer) 1309448221
   3) (integer) 15
   4) 1) "ping"
2) 1) (integer) 13
   2) (integer) 1309448128
   3) (integer) 30
   4) 1) "slowlog"
      2) "get"
      3) "100"
```


### Latency Monitoring
Để monitoring một cách chi tiết thì redis có cung cấp sẵn công cụ [monitoring latency](https://redis.io/topics/latency-monitor), với các metrics được monitor đa dạng. Để enable tính năng này, run command:
```shell
CONFIG SET latency-monitor-threshold 100
``` 
Sau khi chạy xong, redis sẽ ghi lại các command mà thời gian execute lớn hơn 100ms.
Ta có thể dùng 1 số câu lệnh để lấy ra kết quả như `latency latest`, `latency history command`...

Ngoài ra thì còn có câu lệnh để lấy đc max latency theo thời gian nữa, chạy từ console luôn:
```shell
redis-cli --intrinsic-latency 100
```

### Monitoring command
1 số câu lệnh có thể hữu ích trong quá trình monitoring redis:
#### Monitor
Command `monitor` sẽ show các command đang được thực hiện realtime, có thể biết được câu lệnh/key nào thường được gọi đến.
Từ đó cũng có thể thu được ít manh mối để điều tra, tối ưu.

#### Bigkeys
```shell
$ redis-cli --bigkeys
``` 
Lệnh này sẽ scan toàn bộ dataset và list ra các key có size lớn nhất (theo từng loại string, set, hash).
Lưu ý là nó có thể gây chậm redis nhé.


Trên này là 1 số kinh nghiệm xương máu với redis rồi, khi nào có thêm sẽ lại note tiếp vào đây nhé 😂

### References:
- [Redis latency problems troubleshooting](https://redis.io/topics/latency)
- [Redis latency monitoring framework](https://redis.io/topics/latency-monitor)
- [Redis Persistence](https://redis.io/topics/persistence)
- [How to monitor Redis performance metrics](https://www.datadoghq.com/blog/how-to-monitor-redis-performance-metrics/)