---
layout: post
title:  "Ổ cứng đầy rồi, làm sao bây giờ?"
author: halab
categories: [ Monitoring ]
tags: [Monitoring, Disk Space]
image: "https://www.servercake.blog/wp-content/uploads/2017/11/df-and-du-command.png"
featured: false
---
- Có alert full disk kìa em, check đi!
- Để em xem nào. `df -h`
```shell
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        7.9G     0  7.9G   0% /dev
tmpfs           7.9G     0  7.9G   0% /dev/shm
tmpfs           7.9G  624K  7.9G   1% /run
tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/xvda1      100G   89G   11G  89% /
```

- Giờ làm thế nào nữa anh?
- Tìm xem thằng nào chiếm nhiều dung lương.
- Như thế nào ạ?
- Về `root` đi. Xong gõ `du -sh *`, à thêm sort nữa cho dễ nhìn
```shell
$ cd /
$ du -sh * | sort -h
du: cannot access ‘proc/19736/task/19736/fd/3’: No such file or directory
du: cannot access ‘proc/19736/task/19736/fdinfo/3’: No such file or directory
du: cannot access ‘proc/19736/fd/3’: No such file or directory
du: cannot access ‘proc/19736/fdinfo/3’: No such file or directory
0       bin
0       dev
0       lib
0       lib64
0       local
0       media
0       mnt
0       proc
0       sbin
0       srv
0       sys
324K    tmp
624K    run
32M     etc
118M    opt
126M    boot
623M    root
8.0G    home
2.3G    usr
84G     var
```
- Đấy chủ yếu ở thằng `/var` này này. Thường thì data sinh ra hay ở `var/lib` với `/var/log` thôi.
`/var/lib` thì là data của các service rồi, không xoá được, tập trung vào thằng `log` xem.

```shell
$ cd /var
$ du -sh * | sort -h
... <many file and folder> ...
20K     spool
352M    cache
52.2G    lib
32G     log
$
$ cd log
$ du -sh * | sort -h
128K    cron-20200712
128K    cron-20200719
128K    cron-20200726
128K    cron-20200802
352K    cloud-init.log
704K    amplify-agent
3.5M    nginx
8.9M    redis
19M     sa
35M     audit
331M    messages
1.4G    messages-20200712
1.4G    messages-20200719
1.5G    messages-20200726
1.8G    messages-20200802
4.1G    journal
6G     mongodb
20G    server-down
```

- Ơ sao app của mình lại tận 20G anh nhỉ?
- Check lại code xem nào, có in log bậy bạ ở đâu không đấy 😱. 
Xem config log level đúng chưa.

....

Few minutes later

...

- Được rồi, xoá được gần 20GB. Xem nốt thằng mongo đi.
- Có mỗi 1 file `mongod.log` thôi anh.
- Thôi bỏ mịa, không setting log rotate rồi 😂
- Log rotate là gì ạ?
- Nhìn thấy mấy file `messages-yyyyMMdd` kia không? 
Mấy cái log cũ nó sẽ để ở đấy, sau 1 khoảng thời gian thì sẽ tự động xoá đi, 
không cần phải xoá bằng tay như này nữa.
- Ồ hay thế!!!
- Đấy tự làm nốt đi nhé. 
À hình như còn mấy chỗ mình export file csv ra rồi để đấy nữa. Cần thiết thì tạo cronjob để nó tự xoá bớt đi nhé
- Yeap!!!

