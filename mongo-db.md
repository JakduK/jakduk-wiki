<!-- TITLE: MongoDB -->
<!-- SUBTITLE: MongoDB, Database, NoSQL -->

# 설치
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/
# 인증
mongodb는 접속 인증이 없다. 접속 후 명령들에 대한 접근 제어만 가능하다.
production에서는  내부 네트워크를 구성하거나 접근 허용할 ip주소 바인딩을 필히 해야한다.
외부 접속을 가능하게 설정하고 싶을때 `/etc/mongod.conf`를 수정하고 계정을 추가한다.
* /etc/mongod.conf
```
net:
  bindIP: 0.0.0.0

security:
  authorization: enabled
```
* 계정 추가
	* `roles`프로퍼티에 적절한 권한을 지정해준다.
	* buildt-in roles : https://docs.mongodb.com/manual/reference/built-in-roles/
* 계정 관리 : https://docs.mongodb.com/manual/reference/command/nav-user-management/
```
#mongodb 접속
mongo
#db 선택
db db_name
#계정 생성 및 권한 설정
db.createUser({user:'id',pwd:'password',roles:['role']})
```

> `mongoose`는 접속 옵션에 `dbName`을 지정해도 `authdb`를 지정하지 않으면 인증에 실패한다. 이것때문에 시간 무지 낭비함.
> * https://stackoverflow.com/a/33642761/4449475
> ```
> options: {
>     auth: {
>         authdb: 'db_name'
>     }
> }
> ```


# database rename 불가
Collection, index, 내부 uid등등 싸그리 변경해야 돼서 기술적 복잡도가 너무 높다고 하였다.

# Import
* mongorestore
```sh
mongorestore --collection people --db accounts dump/
```

# Export
* mongodump
```sh
mongodump --db test --collection collection
mongodump --db test --excludeCollection=users --excludeCollection=salaries
```

# 스크립트 실행
```sh
mongo 127.0.0.1/my-profile-db --username='username' --password='pwd' mongodb-script.js
```
