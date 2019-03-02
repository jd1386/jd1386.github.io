---
layout: post
title: Mocha와 Chai로 Node API 서버를 유닛 테스트 해보자
date: 2019-02-26 00:00:00 +0900
categories: blog
tags: koreanjson.com testing mocha chai
---

지난 글 [KoreanJSON.com을 만들었다]({% post_url  2019-02-24-KoreanJSON.com을-만들었다 %})에서 [Korean JSON](https://koreanjson.com)을 어떤 이유로 만들게 되었는지 설명했었다. 

[Korean JSON](https://koreanjson.com)은 단순한 Rest API이지만 users, posts, comments, todos 등 여러 resource를 제공한다. 더불어 각 endpoint별로 GET, POST, PUT, DELETE 메서드를 지원하고 있다. 그래서 앞으로 resource가 더 늘어나게 될 수록 개발자가 모든 API가 제대로 작동하고 있는지 반복적으로 확인해야만 한다. 이런 작업에는 많은 시간이 소요될 수 밖에 없고, 매번 개발자의 눈으로 정상 작동 여부를 확인하는 방법으로는 그 정확성을 담보할 수도 없는 노릇이다. 따라서 이런 경우에 각 endpoint의 메서드별로, 개발자가 기대하는 값과 API가 제공하는 값이 일치하는지 여부를 자동으로 확인할 수 있는 테스트가 절대적으로 필요하게 된다. 


이번 글에서는 노드 환경에서, 좀 더 정확히는 노드 환경에서 실행되는 Express 앱에서 Rest API 서버를 Mocha와 Chai를 이용해 자동으로 테스트하는 방법을 알아보겠다.

[KoreanJSON 깃헙 리포](https://github.com/jd1386/korean-json)에서 코드를 확인할 수 있다.


### mocha, chai, chai-http 설치
먼저 이번 글에서 가장 중요한 test framework인 [Mocha](https://www.npmjs.com/package/mocha)와 테스트 라이브러리인 [Chai](https://www.npmjs.com/package/chai) 그리고 http 요청을 위한 Chai의 플러그인 [chai-http](https://www.chaijs.com/plugins/chai-http/)를 아래와 같이 "devDependencies"에 추가하자.
```shell
$ npm i --save-dev mocha chai chai-http
```

그럼 `package.json`의 "devDependencies" 부분에 다음과 같이 각 모듈이 설치될 것이다.
#### package.json
```json
"devDependencies": {
  "chai": "^4.2.0",
  "chai-http": "^4.2.1",
  "mocha": "^6.0.1"
}
```

### Babel 설치
KoreanJSON은 현재 모든 .js 파일에서 ES6를 사용중이다. 테스트 또한 ES6로 작성하기 위해서 컴파일러인 Babel이 필요하다. 아래와 같이 Babel 관련 모듈을 설치해주자.

```shell
$ npm i @babel/cli @babel/core @babel/node @babel/polyfill @babel/preset-env @babel/register
```

[@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)과 [@babel/register](https://babeljs.io/docs/en/babel-register)를 설치하지 않으면 테스트 실행시 테스트 파일의 ES6 문법을 인식하지 못하는 에러가 발생하니 꼭 설치해야 한다. 또한 @babel/polyfill 같은 경우 devDependencies가 아닌 dependency에 설치해야 한다.

이제 테스트를 실행할 스크립트를 `package.json` 파일의 `scripts` 부분에 추가해보자.
#### package.json
```json
"test": "NODE_ENV=test npx mocha --require @babel/register --require @babel/polyfill --exit ./test/**.spec.js"
```

위 스크립트는 @babel/register와 @babel/polyfill을 이용해서 test 폴더 내부의 모든 spec.js 파일을 먼저 ES5로 컴파일한 후 node_modules에 설치되어 있는 mocha를 실행하는 스크립트이다. `--exit` 옵션은 테스트가 종료되면 프로세스 또한 종료하라는 내용이다. 더 많은 옵션은 [Mocha 공식 문서](https://mochajs.org/#command-line-usage)에서 확인할 수 있다.

이제 프로젝트 root directory에서 /test 폴더를 만든 후 `users.spec.js` 파일도 만들어주자. 

그리고 mysql 데이터베이스 설정 파일인 config.json의 "test" 데이터베이스 관련 사항에 다음과 같이 `"logging": false` 옵션을 추가해주자. 이 부분을 생략하면 테스트 결과와 데이터베이스 로그가 뒤죽박죽 섞여버려 테스트 결과를 제대로 읽기 어렵다.

#### config.json
```json
"test": {
  "username": "root",
  "password": null,
  "database": "koreanjson_test",
  "host": "127.0.0.1",
  "dialect": "mysql",
  "logging": false
}
```

### 유저 라우터

이제 유저 라우터를 작성할 차례이다. 아래와 같이 /users로 GET 요청이 오면 모든 유저를 찾아서 json 형식으로 반환하는 라우터를 만들어보자.

#### /routes/users.js
```js
import express from 'express';
import { sequelize, User } from '../models';

const router = express.Router();

router.get('/', async (req, res) => {
  const users = await User.findAll();
  res.json(users);
});

module.exports = router;
```

### 테스트 데이터베이스 생성
테스트를 작성하기 전 가장 중요한 데이터베이스가 남았다. 테스트 환경은 development나 production 환경과 다르게 별도의 test 환경이 존재한다. 따라서 mocha 테스트를 실행하는 스크립트에도 별도로 `NODE_ENV=test`라는 환경변수가 추가되어 있었던 것이다. 여기서 중요한 점은 test 환경에는 development, production 환경과 구분되는, 별도의 테스트 데이터베이스가 생성되어 있어야 한다는 점이다. [sequelize-cli](http://docs.sequelizejs.com/manual/tutorial/migrations.html)를 사용해서 migration 하는 방법도 있지만 일단은 mysql 콘솔에서 테스트 데이터베이스를 생성해보자.

`$ mysql -u root`으로 mysql 콘솔에 들어간 후 
`CREATE DATABASE koreanjson_test`를 입력하면 koreanjson_test 데이터베이스를 만들 수 있다.

이제 본격적으로 테스트 케이스를 작성해보자.

### 테스트 케이스 작성
먼저 필요한 각종 모듈들, 서버 설정 및 실행 내용을 담고 있는 app.js파일 그리고 테스트를 실행하고자 하는 User 모델을 불러온다. 그리고 chai 라이브러리에서는 should나 assert 대신 모든 브라우저에 호환이 되는 expect를 사용해보도록 하겠다. 또한 http 요청을 보내기 위해서는 chai-http 모듈을 사용해야 한다.

#### /test/users.spec.js
```coffee
import chai from 'chai';
import chaiHttp from 'chai-http';
import app from '../app';
import User from '../models';
import { type } from 'os';

const expect = chai.expect;
chai.use(chaiHttp);
```


각각의 테스트가 실행되기 전 beforeEach 함수가 실행되는데 그때 유저 테이블을 싹 다 비웠다가 다시 새롭게 샘플 유저 데이터를 채워넣는 작업이 필요하다. 그렇지 않으면 테스트를 통해 변경된 (그리고 테스트 중 잠재적으로 변경될 수 있는) 유저 데이터로 인해 테스트 데이터의 무결성에 손상이 갈 수 있다. 잘못된 데이터 샘플로는 아무리 테스트 케이스를 잘 작성해도 제대로 된 결과가 나오지 않을 테니 말이다.

#### /test/users.spec.js
```js
import chai from 'chai';
import chaiHttp from 'chai-http';
import app from '../app';
import { sequelize, User } from '../models';
import userSeeder from '../seeders/users';
import { users } from '../data';
import { type } from 'os';
import { resolve } from 'url';

describe('Users resource', () => {
  beforeEach(done => {
    // users 테이블의 모든 레코드를 삭제한다
    User.destory({
      where: {},
      force: true
    }).then(async () => {
      // 새로운 유저 데이터를 생성한다
      // User.create({...}) 또는
      // User.bulkCreate([]) 등으로 생성해도 되지만 
      // user seeder를 만들어 놓은 게 있으므로 seeder를 사용하겠다
      await userSeeder();
    }).then(() => {
      done();
    })
  });
});
```

이제 beforeEach 함수 아래에 본격적으로 테스트케이스를 추가해보자.

#### /test/users.spec.js
```js
  // Test GET /users route
  describe('GET /users', () => {
    /* 
      /users로 GET 요청이 오면 
      1. 200 코드를 반환해야 하고
      2. JSON 형식으로 된 res.body는 array로 되어 있어야 하고
      3. array는 10개의 요소를 포함해야 한다
    */
    it('it should GET all the users', done => {
      chai
        .request(app)
        .get('/users')
        .end((err, res) => {
          expect(res).to.have.status(200);
          expect(res.body).to.be.a('array');
          expect(res.body.length).to.equal(10);
          done();
        });
    });

    /* 
      /users로 GET 요청이 오면 
      1. 200 코드를 반환해야 하고
      2. JSON 형식으로 된 res.body array의 첫번째 요소는 오브젝트여야 하고
      3. 오브젝트의 name 프로퍼티는 문자열 형식이어야 하며
      4. 그 값은 '이정도'여야 한다.
    */

    it('it should GET correct users', done => {
      chai
        .request(app)
        .get('/users')
        .end((err, res) => {
          expect(res).to.have.status(200);
          expect(res.body[0]).to.be.an('object');
          expect(res.body[0].name).to.be.a('string');
          expect(res.body[0].name).to.equal('이정도');
          done();
        });
    });
  });
});
```

`$ npm test`를 실행하면 다음과 같이 우리가 작성한 2개의 테스가 모두 통과함을 알 수 있다.

<img src="/assets/img/2019-02-26.png">