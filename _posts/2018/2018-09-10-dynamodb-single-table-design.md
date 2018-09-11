---
tag:
title: 단일 테이블 DynamoDB 디자인
---

## 데이터를 어떻게 저장할 것인가

DynamoDB를 처음 접하면서 가장 어려웠던 부분이다. 걍 대충 RDB 형태를 json으로 저장하면 되겠지 했는데, 으잉 

단일 테이블로 1:1, 1:N, N:M

NoSQL 테이블 디자인에 대해 구글링하면서 가장 많이 봤던 문장은

> 서비스를 먼저 디자인하고 어떤 쿼리가 필요한지 파악한 후 테이블을 디자인한다.

타 NoSQL은 어떤지 모르겠다. DynamoDB은 일단 테이블을 만들면 키의 이름과 LSI를 추가-수정-삭제할 수 없다. 그래서 서비스를 먼저 디자인 하는 것이 맞다.

## 하나의 테이블 vs 여러개의 테이블

| id (PK) | createdAt (SK) | email          |
| ------- | -------------- | -------------- |
| USER-1  | createdAt      | user@email.com |

'id'는 이 데이터의 타입도 함꼐 나타낸다.

이메일을 다른 row로 빼면

| id (PK) | createdAt (SK) | email          |
| ------- | -------------- | -------------- |
| USER-1  | createdAt      | EMAIL-1        |
| EMAIL-1 | createdAt      | user@email.com |

이런 1:1 관계가 된다.

'email' 속성명과 내용이 같지 않은 경우가 있기 때문에 컬럼명 일반화

| PK      | SK        | Data           |
| ------- | --------- | -------------- |
| USER-1  | createdAt | EMAIL-1        |
| EMAIL-1 | createdAt | user@email.com |

email로 사용자를 찾을 수 있도록 GSI 추가

| PK (GSI-SK) | SK        | Data (GSI-PK)  |
| ----------- | --------- | -------------- |
| USER-1      | createdAt | EMAIL-1        |
| EMAIL-1     | createdAt | user@email.com |

Find email by user
[ email_id ] = SELECT Data FROM TABLE WHERE PK="USER-1"
[ email ] = SELECT * FROM TABLE WHERE PK="EMAIL-1"

Find user by email
[ user_id ] = SELECT id FROM GSI WHERE Data="user@email.com"

---

| PK (GSI-SK) | SK        | Data (GSI-PK) |
| ----------- | --------- | ------------- |
| USER-1      | createdAt | email         |
| POST-1      | createdAt | post body     |

1:N

| PK (GSI-SK) | SK        | Data (GSI-PK) | Body      |
| ----------- | --------- | ------------- | --------- |
| USER-1      | createdAt | EMAIL-1       |
| POST-1      | createdAt | USER-1        | post body |
| POST-2      | createdAt | USER-1        | post body |

Find user id by post
[ user_id ] = SELECT Data FROM TABLE WHERE PK="POST-1"

Find post ids by user
[ post_ids ] = SELECT PK FROM GSI WHERE Data="USER-1"

다중 종속

| PK (GSI-SK)     | SK        | Data (GSI-PK) | Body |
| --------------- | --------- | ------------- | ---- |
| USER-1          | createdAt | EMAIL-1       |
| POST-1          | createdAt | post body     |
| CATEGORY-1      | createdAt | post body     |
| USER-1:POST     | createdAt | POST-1        |
| CATEGORY-1:POST | createdAt | POST-1        |

Find user id by post
[ user_id ] = SELECT PK FROM GSI WHERE Data="POST-1" AND PK= "USER-1:POST"

Find post ids by user
[ post_ids ] = SELECT Data FROM TABLE WHERE PK="USER-1:POST"

종속 앤티티에 상태 추가

| PK (GSI-SK) | SK                 | Data (GSI-PK) | Body |
| ----------- | ------------------ | ------------- | ---- |
| USER-1      | createdAt          | EMAIL-1       |
| POST-1      | createdAt          | post body     |
| POST-2      | createdAt          | post body     |
| USER-1:POST | DRAFT:createdAt    | POST-1        |
| USER-1:POST | PUBLISHEDcreatedAt | POST-2        |

Find draft post ids by user
[ draft_post_ids ] = SELECT Data FROM TABLE WHERE PK="USER-1:POST" AND SK=begins_with("DRAFT")

다중 소유

| PK (GSI-SK)     | SK                  | Data (GSI-PK) | Body |
| --------------- | ------------------- | ------------- | ---- |
| USER-1          | createdAt           | EMAIL-1       |
| POST-1          | DRAFT:createdAt     | post body     |
| POST-2          | PUBLISHED:createdAt | post body     |
| USER-1:question | DRAFT:createdAt     | POST-1        |
| USER-1:answer   | PUBLISHED:createdAt | POST-2        |

---

N:M

| PK (GSI-SK) | SK        | Data (GSI-PK) | ... |
| ----------- | --------- | ------------- | --- |
| POST-1      | createdAt | post body     |
| TAG-1       | createdAt | tag name      |
| POST-1:tag  | createdAt | TAG-1         |
| POST-1:tag  | createdAt | TAG-2         |
| POST-2:tag  | createdAt | TAG-1         |
| TAG-1:post  | createdAt | POST-1        |
| TAG-1:post  | createdAt | POST-2        |
| TAG-2:post  | createdAt | POST-2        |

---


| PK (GSI-SK) | SK        | Data (GSI-PK) | ... |
| ----------- | --------- | ------------- | --- |
| USER-x      | createdAt | email         |
| USER-x:post | createdAt | POST-x        |
| POST-x      | createdAt | body          |
| TAG-x       | createdAt | name          |
| POST-x:tag  | createdAt | TAG-x         |
| TAG-x:post  | createdAt | POST-x        |