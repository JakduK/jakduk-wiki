<!-- TITLE: MySQL -->
<!-- SUBTITLE: MySQL, Database, RDBMS -->

# 계정 조회
```sql
select * from mysql.user;
```

# 계정 추가
```sql
CREATE USER '<id>'@'<계정 사용자의 host>' IDENTIFIED BY '<password>';
```

# 권한 할당
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON contacts TO 'smithj'@'localhost';
GRANT ALL ON contacts TO 'smithj'@'localhost';
GRANT SELECT ON contacts TO '*'@'localhost';
```

# 권한 제거
```sql
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'USERNAME'@'%';
REVOKE INSERT ON *.* FROM 'jeffrey'@'localhost';
REVOKE 'role1', 'role2' FROM 'user1'@'localhost', 'user2'@'localhost';
REVOKE SELECT ON world.* FROM 'role3';
```

# 계정 삭제
```sql
DROP USER '<id>'@'<계정 사용자의 host>';
```

# 테이블 생성
https://dev.mysql.com/doc/refman/8.0/en/create-table.html 13.1.18 CREATE TABLE Syntax
```sql
CREATE TABLE (column_name data_type [null | not null] [default value], ...)
```

# 터미널에서 쿼리 실행
옵션으로 쿼리 입력
```sql
mysql -u [username] -p [dbname] -e [query]
```

스크립트 파일로 쿼리 입력
```sql
mysql -u [username] -p [dbname] < query.sql
```

# Dump
* --single-transaction : lock 사용안함
* --databases : database 선택, whitespace로 구분하여 복수 지정 가능
* --no-create-db : create database 구문 제외
* --no-create-info : create table 구문 제외

* database, table 지정
```sql
mysqldump -u [username] -p [dbname] [tableA] [tableB] > dump.sql
```

* database 지정
```sql
mysqldump -u [username] -p --databases [dbname] > dump.sql
```
# JDBC 연동
## 반드시 겪는 오류
* java.sql.SQLException: The server time zone value 'KST' is unrecognized or represents more than one time zone.
  * "KST"는 MySQL이 모르는 값이라 발생. "KST"타임존을 적용하겠다면 "Asia/Seoul"사용 ex) mysql://`<HOST>`/`<DB>`?serverTimezone=Asia/Seoul