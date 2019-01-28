---
layout: post
title: sequelize cli로 seed data 생성하기
date: 2019-01-19 13:00:00 +0900
categories: blog
tags: node sequelize
---

루비온레일즈(Ruby on Rails)의 경우 `bundle exec rake db:seed` 명령을 통해 비어있는 데이터베이스에 데이터를 손쉽게 추가할 수 있다. 새롭게 데이터베이스를 만들었다거나 테이블을 새롭게 생성했을 때 당장에 필요한 샘플 데이터를 넣어야 할 일이 있다. 이를 테면 Articles라는 테이블을 만들었으면 기사 목록을 데이터로 넣어줘야 하는데, 테스트 환경이라 할 지라도 하나 하나 INSERT 하는 건 너무나도 귀찮은 일이다. 

Node의 대표적인 ORM 중 하나인 Sequelize는 Cli 프로그램을 제공하는데 `bundle exec rake db:seed`와 동일한 작업을 수행할 수 있다.

먼저 sequelize-cli를 설치하자. <br>
`$ npm install sequelize-cli`

그리고 다음 명령어를 통해 Seed file을 생성해주자. <br>
`$ node_modules/.bin/sequelize seed:generate --name articles` <br>
--name은 만들고자 하는 파일명을 argument로 넘기면 된다.

npx가 설치되어 있다면 다음과 실행 할 수도 있다.<br>
`$ npx sequelize seed:generate --name articles`

명령을 실행하면 프로젝트 루트 폴더 내에 seeders라는 폴더가 생기고 내부에 명령을 실행한 timestamp와 --name 값으로 넘긴 articles를 포함한 파일 (예: `20190119082901-articles.js`)이 생성된다. 내용은 다음과 같다.

```javascript
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkInsert('Person', [{
        name: 'John Doe',
        isBetaMember: false
      }], {});
    */
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkDelete('Person', null, {});
    */
  }
};
```

up 함수에는 추가할 seed data를 bulkInsert해주어야 하고 반대로 down 함수는 up 함수를 통해 추가했던 seed data를 bulkDelete 해주는 내용을 넣어야 한다. down 함수를 추가해주어야 하는 이유는 up 함수를 통해 추가했던 데이터를 삭제할 때 쓰이기 때문이다. 그럼 up과 down 함수를 다음의 예시와 같이 작성해보자.


```javascript
module.exports = {
  up: function (queryInterface, Sequelize) {
    // 시드 데이터를 추가한다.
    return queryInterface.bulkInsert('Articles', [
      {
        title: '테스트 기사 제목 1',
        content: '테스트 기사 내용 테스트 기사 내용 테스트 기사 내용',
        source_url: 'https://m.news.naver.com/rankingRead.nhn?oid=421&aid=0003664078&sid1=102&date=20181029&ntype=RANKING',
        publisher: '중앙일보',
        image_url: 'https://mimgnews.pstatic.net/image/origin/421/2018/10/29/3664078.jpg?type=nf144_144',
        sid: '102',
        oid: '421',
        aid: '0003664078',
        category_id: 1
      },
      {
        title: '테스트 기사 제목 2',
        content: '테스트 기사 내용 테스트 기사 내용 테스트 기사 내용',
        source_url: 'https://m.news.naver.com/rankingRead.nhn?oid=421&aid=0003664078&sid1=102&date=20181029&ntype=RANKING',
        publisher: '조선일보',
        image_url: 'https://mimgnews.pstatic.net/image/origin/421/2018/10/29/3664078.jpg?type=nf144_144',
        sid: '102',
        oid: '421',
        aid: '0003664078',
        category_id: 1
      },
      {
        title: '테스트 기사 제목 3',
        content: '테스트 기사 내용 테스트 기사 내용 테스트 기사 내용',
        source_url: 'https://m.news.naver.com/rankingRead.nhn?oid=421&aid=0003664078&sid1=102&date=20181029&ntype=RANKING',
        publisher: '한겨레신문',
        image_url: 'https://mimgnews.pstatic.net/image/origin/421/2018/10/29/3664078.jpg?type=nf144_144',
        sid: '102',
        oid: '421',
        aid: '0003664078',
        category_id: 1
      },
      {
        title: '테스트 기사 제목 4',
        content: '테스트 기사 내용 테스트 기사 내용 테스트 기사 내용',
        source_url: 'https://m.news.naver.com/rankingRead.nhn?oid=421&aid=0003664078&sid1=102&date=20181029&ntype=RANKING',
        publisher: '경향신문',
        image_url: 'https://mimgnews.pstatic.net/image/origin/421/2018/10/29/3664078.jpg?type=nf144_144',
        sid: '102',
        oid: '421',
        aid: '0003664078',
        category_id: 2
      },
      {
        title: '테스트 기사 제목 5',
        content: '테스트 기사 내용 테스트 기사 내용 테스트 기사 내용',
        source_url: 'https://m.news.naver.com/rankingRead.nhn?oid=421&aid=0003664078&sid1=102&date=20181029&ntype=RANKING',
        publisher: '동아일보',
        image_url: 'https://mimgnews.pstatic.net/image/origin/421/2018/10/29/3664078.jpg?type=nf144_144',
        sid: '102',
        oid: '421',
        aid: '0003664078',
        category_id: 2
      }
    ], {});
  },

  down: function (queryInterface, Sequelize) {
    // 추가했던 모든 데이터를 삭제한다.
    return queryInterface.bulkDelete('Articles', {});
  }
};
```

이제 거의 다되었다. 다음과 같이 seed 명령을 실행하면 데이터베이스에 seed data가 추가될 것이다.

`node_modules/.bin/sequelize db:seed:all`

추가한 seed data 작업을 undo 하고 싶다면 다음 명령을 실행하면 해당 seed 파일의 down 함수에 있는 bulkDelete가 실행되어 Articles 테이블의 모든 데이터가 삭제될 것이다.

`node_modules/.bin/sequelize db:seed:undo`

더 자세한 내용은 Sequelize [공식 문서](http://docs.sequelizejs.com/manual/tutorial/migrations.html)에서 확인할 수 있다.