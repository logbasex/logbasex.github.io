---
title: 'Backup and restore mongodb using mongodump and mongorestore with Docker?'
date: 2021-02-23
permalink: /posts/2021/02/backup-and-restore-mongodb-using-mongodump-and-mongorestore-with-docker/
tags:
- mongodb
- mongodump
- mongorestore
- database
---

## Giới thiệu
**I. Vấn đề cần giải quyết** 

Backup dữ liệu từ mongodb trên server về máy local và import dữ liệu đã backup vào mongodb ở máy local.

## Cài đặt 
```shell
docker run -d --name=mongo44 -p=27017:27017 mongo:latest
```

## Backup

Để backup dữ liệu của database trên server về máy local chúng ta dùng `mongodump` như sau.

```shell
docker exec <mongodb container> /bin/bash -c 'exec mongodump --host <hostname:port> --db <database> --gzip --archive' > db.gz
```

Như đã giới thiệu ở trên, khi xem `running container` là một máy ảo đang chạy Ubuntu chẳng hạn, thì ý tưởng ở đây là chui vào máy ảo `dump data` ra file rồi chuyển data từ máy ảo sang máy local. Và thật may mắn là Docker hỗ trợ câu lệnh giúp chúng ta làm điều đó với `docker exec`. 

#### Docker Exec Syntax
```shell
docker exec <options> <container> <command>
```

## Restore

```shell
docker exec -i <mongodb container> /bin/bash -c 'mongorestore --gzip --archive --nsFrom="db_test.*" --nsTo="db_dev.*"' < db.gz
```

## Export
```shell
sudo docker exec -it mongo44 /bin/bash -c 'mongoexport --host <hostname:port> --db <database-name> --collection <collection-name> --fields <field-name> --type <type/csv/gzip> --noHeaderLine' --query <query> > fields.csv
```

For further reading: https://docs.mongodb.com/database-tools/