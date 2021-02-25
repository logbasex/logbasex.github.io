---
title: 'Backup and restore mongodb using mongodump and mongorestore with Docker?'
date: 2020-02-23
permalink: /posts/2020/02/backup-and-restore-mongodb-using-mongodump-and-mongorestore-with-docker/
tags:
- mongodb
- mongodump
- mongorestore
- database
---

## Giới thiệu
**I. Vấn đề cần giải quyết** 

Backup dữ liệu từ mongodb trên server về máy local và import dữ liệu đã backup vào mongodb ở máy local.

**II. Kiến thức căn bản**

Trong phạm vi phạm vi bài viết này mình có dùng `docker` để cài đặt `mongodb` ở máy local nên sẽ giới thiệu sơ qua về `docker`.
    
> Đại khái thì Docker là một cái mà cho phép đóng gói phần mềm trong các container. Và container là một hệ thống cô lập (isolated system) sử dụng công nghệ ảo hóa (OS-level virtualization) chứa tất cả những thứ cần thiết để khởi chạy một ứng dụng nào đó.
>> Có thể xem container như một máy ảo để dễ hình dung.
>>> Container thường bao gồm một hệ điều hành tối giản, những ứng dụng cần cài đặt và configs của những ứng dụng đó (software stack).
>
>> Docker có thêm một khái niệm là Docker image, có thể xem image là một template bao gồm những chỉ dẫn cần thiết để tạo ra được container. So sánh với Java thì một image có thể xem như là một class và một container có thể xem như là một object.

Lý do mình sử dụng `docker` cho `mongodb` thì xuất phát từ nguyên nhân là không cài trực tiếp lên máy local được, có thể do tải thiếu dependecies, hoặc trong quá trình cài đặt có conflict với config của ứng dụng khác...Và với công nghệ ảo hóa tạo ra nhiều hệ thống cô lập khác nhau trên máy tính, `docker` cho phép chúng ta có thể tự tin cài đặt các ứng dụng thoải mái mà không phải lo về sự xung đột hay thiếu sót.
> Với những docker image có hàng triệu lượt tải xuống thì sẽ đảm bảo được việc phần mềm chúng ta cài đặt được chuẩn hóa, không có những hành vi bất thường.
> 
> Với những software stack nếu cài đặt trực tiếp khiến việc bật/tắt trở nên tốn thao tác thì docker sẽ giúp chúng ta đơn giản hóa điều đó.

[Xem hướng dẫn cài dặt mongodb với docker ở đây nhé](https://linuxhint.com/setup_mongodb_server_docker/)
## Backup

Để backup dữ liệu của database trên server về máy local chúng ta dùng `mongodump` như sau.

```shell
docker exec <mongodb container> /bin/bash -c 'exec mongodump --host <hostname:port> --db <database> --gzip --archive' > db.dump
```

Như đã giới thiệu ở trên, khi xem `running container` là một máy ảo đang chạy Ubuntu chẳng hạn, thì ý tưởng ở đây là chui vào máy ảo `dump data` ra file rồi chuyển data từ máy ảo sang máy local. Và thật may mắn là Docker hỗ trợ câu lệnh giúp chúng ta làm điều đó với `docker exec`. 

#### Docker Exec Syntax
```shell
docker exec <options> <container> <command>
```


