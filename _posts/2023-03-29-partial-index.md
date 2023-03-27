---
layout: post
title:  "Partial Indexes"
date:   2023-03-27 14:00:00 +0900
author: hsyoo
categories: mongodb index partial
---
<hr/>

## Partial Indexes?

저희 개발팀에서는 프로젝트 진행 시 MongoDB를 자주 사용하고 있습니다. 
아시다시피 데이터베이스에서 index 설정은 필수인데 보통은 아래와 같이 생성합니다

```javascript
db.restaurants.createIndex(
    { cuisine: 1, name: 1 },
    { partialFilterExpression: { rating: { $gt: 5 } } }
)
```


mongodb V3.2부터는 **partial index**를 지원합니다. **partial index**는 index를 생성할 때 조건을 걸어서 index를 생성할 수 있습니다.[#](https://www.mongodb.com/docs/manual/core/index-partial/#create-a-partial-index){:target="_blank"}
```javascript
db.restaurants.createIndex(
  { cuisine: 1, name: 1 },
  { partialFilterExpression: { rating: { $gt: 5 } } }
)
```


**partial index**는 아래 연산자들을 지원합니다.

- equality expressions (e.g. { field: value }, { field: { $eq: value } })
- $exists: true expression
- $gt, $gte, $lt, $lte expressions
- $type expressions
- $and operator
- $or operator
- $in operator

```javascript
// index가 작동함
db.restaurants.find( { cuisine: "Italian", rating: { $gte: 8 } } );

// index가 작동하지 않음
db.restaurants.find( { cuisine: "Italian", rating: { $lt: 8 } } );

```

## 장점

**partial index**는 이름처럼 부분적으로 인덱스를 생성하기 때문에 용량을 더 적게 사용하는 장점이 있습니다.
리소스를 적게 사용하는 만큼 속도도 더 빠르다는 장점이 있다고 합니다.

## 마치며
mongodb는 NO SQL의 대표주자 격인 DB입니다. 다만 schemaless한 DB라고 해서 schema가 필요없는 건 절대 아니며,     
적절한 schema를 잘 설계해야 합니다. 위와 같이 index 생성도 충분히 고려되어야 하는 사안이며,
최근 NO SQL와 전통적인 RDBMS를 조합하는 경우를 많이 볼 수 있습니다.   
피앰아이 개발팀에서도 완전 NO SQL보다는 mongodb와 postgreSQL을 조합해서 진행하려고 합니다.

다음 번엔 **sparse indexes**에 대해 알아보는 시간을 갖겠습니다.[#](https://www.mongodb.com/docs/manual/core/index-sparse/){:target="_blank"}


출처
[https://docs.mongodb.com/manual/core/index-partial/](https://docs.mongodb.com/manual/core/index-partial/){:target="_blank"}
