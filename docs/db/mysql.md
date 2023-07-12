
## 쿼리문 연습

### 데이터 삽입

```
INSERT INTO `tb_sdk_page_permit` (`pagseq`, `admseq`, `permit`, `delflag`, `reguserid`, `regip`, `regdate`, `moduserid`, `modip`, `moddate`)
VALUES
	(5,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(6,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(7,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(8,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(9,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(10,9,200,0,'admin','127.0.0.1','2022-05-27 11:16:10','user23','127.0.0.1','2022-06-08 15:52:57'),
	(38,9,100,0,'admin','127.0.0.1','2022-10-19 14:19:05','admin','127.0.0.1','2022-10-19 14:19:05');
```

### 날짜별 데이터 조회

```
SELECT*
from table_name
where updated_at between '2023-06-01' and '2023-07-13'
order by updated_at desc
```

```
select *from table_name 
where project = 'mytail' and filedate = '2023-02-17';

select *from tb_sdk_page_permit where admseq=7;
select *from tb_sdk_page_permit where moduserid='chc3484';
```

## [MariaDB/mysql] root 초기 비번 설정
### mariadb 접속
```
$ mysql
```

### mysql db 접근
```
[MariaDB [(none)]> use mysql
```

### root 비번 설정
```
// mysql 5.6 이하
[mysql]> set password for 'root'@'localhost' = PASSWORD('1234');
// mysql 5.7 이상
[mysql]> set authentication_string=password('new password') where user = 'root';
// mysql 8.x 이상
[mysql]> alter user 'root'@'localhost' identified with mysql_native_password by 'new password';

// mariadb 10.4 이상
[MariaDB [mysql]> set password=password('1234');
[MariaDB [mysql]> flush privileges;
```

### root 로그인
```
mysql -uroot -p1234
```

## [MariaDB] port 변경

## 설정파일 편집
```
$ vi /usr/local/etc/my.cnf
```

![port-add](../img/mysql_port.png)

## 저장후 서비스 재시작
```
$ brew services restart mariadb
```

## 서비스 확인
```
$ brew services list
```

## port 변경확인
```
$ sudo lsof -i :8000
```
