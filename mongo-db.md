<!-- TITLE: MongoDB -->
<!-- SUBTITLE: MongoDB, Database, NoSQL -->

# 설치
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

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
```
<br>
```sh
mongodump  --db test --excludeCollection=users --excludeCollection=salaries
```

# 스크립트 실행
```sh
mongo 127.0.0.1/my-profile-db --username='username' --password='pwd' mongodb-script.js
```
