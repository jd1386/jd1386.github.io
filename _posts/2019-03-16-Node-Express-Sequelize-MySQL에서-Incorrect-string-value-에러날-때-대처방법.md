---
layout: post
title: Node Express Sequelize MySQL에서 Incorrect string value 에러날 때 대처방법
date: 2019-03-16 00:00:00 +0900
categories: blog
tags: sequelize mysql
---

MySQL과 [Sequelize CLI](https://github.com/sequelize/cli) 를 사용하는 프로젝트에서 한글 값을 칼럼에 넣으려고 할 때 아래와 같이 **"Incorrect string value"**라는 에러가 뜨고 저장이 되지 않았다.

<img src="/assets/img/2019-03-18-error-log.png">


원인을 살펴보니 Sequelize CLI의 `db:create` 를 이용해서 데이터베이스를 생성하면 모든 칼럼의 기본 인코딩이 `cp1252 West European (latin1)`으로 되어있다. 이런 이유로 한글 인코딩을 인식하지 못해서 저장하지 못하는 것이다.

<img src="/assets/img/sequelize-cli-default-charset.png">


재미있는 것은 database config 파일에 아래와 같이 charset과 collate 필드를 추가했음에도 불구하고 Sequelize CLI가 제대로 반영하지 못한다. development 및 production 환경 모두에서 마찬가지였다.

```js
"development": {
  "username": "root",
  "password": null,
  "database": process.env.DATABASE_NAME,
  "host": process.env.DATABASE_HOST,
  "dialect": "mysql",
  "define": {
    "charset": "utf8mb4",
    "dialectOptions": {
      "collate": "utf8mb4_general_ci"
    }
  }
}
```



Sequelize CLI를 이용하지 않고 `sequelize.sync({ force: true})` 로 실행하면 database config 파일의 charset과 collate 필드를 인식하고 각 테이블의 기본 인코딩 값을 설정값으로 변경해준다. 따라서 Sequelize의 문제는 아니고 Sequelize CLI로 `db:create` 실행 할 때 데이터베이스 설정 파일을 pick up하지 못하는 것 같다. (Sequelize CLI 리포에 [관련 Issue](https://github.com/sequelize/cli/issues/590)가 있다.)



## 해결 방법

지금까지 찾은 해결 방법은 여러가지가 있는데 그 중 하나를 선택하면 된다.


### 최초 데이터베이스 생성 후 mysql 콘솔에 들어가서 charset과 collate를 변경해주는 방법

- `mysql -u USERNAME -p` 으로 mysql 콘솔에 접속 후
- `ALTER DATABASE YOUR_DBNAME CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;` 실행한다.
- 단점: 
  - 귀찮다. 
  - 로컬 환경, 테스트 환경, 프로덕션 환경 모두 일일이 해줘야 한다. 
  - 잊어버리기 쉽다.
  - 일일이 다시 지정해주지 않는 이상, 자동화된 rollback 방법이 없다.


### 최초 데이터베이스 생성 후 Sequel Pro에 접속해서 데이터베이스 설정을 바꿔주는 방법

<img src="/assets/img/sequelize-cli-default-charset.png">

- 상단 'Database' 메뉴 > 'Alter Database' 선택

- Database Encoding에 utf8mb4와 Collation에 utf8mb4_unicode_ci 선택

- 단점: 

  - 역시 귀찮다. 
  - 모든 환경에 일일이 적용시켜야 한다.
  - 잊어버리기 쉽고 자동화된 rollback 방법이 없다.


### sequelize db:create 할 때 arg를 넣는 방법

아래와 같이 최초 db:create 할 때 charset을 arg로 넘기는 방법이 있다.

`npx sequelize db:create --charset=utf8mb4 --env=production`

장점:

 - 편하다.

단점:

- 잊기 쉽고 자동화된 rollback 방법이 없다.



### table을 생성하는 migration 파일별로 charset을 지정해주는 방법

```js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable(
      'Posts',
      {
        id: {
          allowNull: false,
          autoIncrement: true,
          primaryKey: true,
          type: Sequelize.INTEGER
        },
        title: {
          type: Sequelize.TEXT
        },
        content: {
          type: Sequelize.TEXT
        },
        createdAt: {
          allowNull: false,
          type: Sequelize.DATE
        },
        updatedAt: {
          allowNull: false,
          type: Sequelize.DATE
        }
      },
      {
        charset: 'utfmb4',
        collate: 'utf8mb4_general_ci'
      }
    );
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Posts');
  }
};

```
장점:

- 테이블별로 인코딩을 다르게 가져갈 수 있다. (=> 장점이 있을까? 여러 언어권을 동시에 서비스한다면 그럴 지도 모르겠다.)

단점: 

- 모든 테이블 생성 마이그레이션마다 추가해줘야해서 잊기 쉽고 역시 귀찮다.



### migrations 폴더에 데이터베이스 charset을 변경해주는 migration 파일 추가

[관련 Issue](https://github.com/sequelize/cli/issues/590)에 있는 내용인데, 다음과 같이 migration 파일을 새로 작성하고

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.query(
      `ALTER DATABASE ${
        queryInterface.sequelize.config.database
      } CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
    );
  },

  down: (queryInterface, Sequelize) => {}
};
```


파일명을 `20190303000000-fix-db-charset.js` 등으로 설정해서 다른 마이그레이션 파일이 실행되기 이전에 최초로 실행되도록 한다. 최초로 실행되어야 한다는 점이 포인트다. 

<img src="/assets/img/2019-03-18-after-migration.png">


장점: 

- 1, 2번 방법에 비해서 신경쓸 일이 줄어들어서 편하다.
- 단순 migration 파일일 뿐이므로 필요에 따라서 Rollback 할 수 있다.


마이그레이션 이후 Encoding과 Collation이 모두 설정한 값으로 지정되었음을 알 수 있다.

<img src="/assets/img/2019-03-18-success.png">

Sequelize CLI는 편해서 잘 쓰고 있긴한데 몇몇 직관적이지 않은 부분이 있어서 조금 아쉽다. 이럴 때면 Ruby on Rails의 [ActiveRecord](https://edgeguides.rubyonrails.org/active_record_migrations.html)가 자체 지원하는 마이그레이션 방식이 돌이켜보면 굉장히 직관적이고 훌륭했다는 생각이 든다. Sequelize CLI도 기본 syntax와 설정 방식 등 많은 부분이 ActiveRecord로부터 영향을 받은 것으로 보인다. 