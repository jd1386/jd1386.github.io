---
layout: post
title: node cron과 shell.js로 cron job 실행하기
date: 2019-01-04 13:00:00 +0900
categories: blog
tags: node
---

['뉴스 읽어주는 Sunny 프로젝트']({% post_url 2018-12-19-2주-프로젝트-뉴스-읽어주는-Sunny-회고 %})에서 해결해야 했던 주요 이슈 중 하나는 최신 인기 기사 목록을 자동으로 갱신하는 것이었다. 매시간마다 인기 기사가 네이버 뉴스 사이트에서 업데이트 되는 구조였기 때문에 최신 인기 기사 목록을 보여줘야 하는 Sunny 앱 역시 실시간으로 인기 기사 목록을 개발자의 개입없이 자동으로 갱신할 수 있어야 했다. 

네이버에서 기사 목록을 찾아 기사 제목과 url 등의 정보를 scrape하고 parse해서 데이터베이스에 저장하는 [함수]({% post_url 2018-12-19-2주-프로젝트-뉴스-읽어주는-Sunny-회고 %})는 이미 구현했으니 이 함수를 매 시간마다 자동으로 실행하도록 cron job을 활용해보자. 
<br><br>


### cron job이란
매 시간마다 특정 명령을 수행하도록 하기 위해서 자주 쓰이는 것은 cron job이다 (cron task로 불리기도 함). [cron job](https://en.wikipedia.org/wiki/Cron)이란 scheduled tasks를 의미하는데 정해진 시간 혹은 시간 간격마다 정해진 스크립트를 실행한다. 초단위로 설정 할 수도 있고 매분, 매시간, 또는 매주 등 굉장히 유연하게 단위 시간을 정할 수 있어서, 데이터베이스 백업, 뉴스레터 발송 등 다양한 목적으로 굉장히 유용하게 쓸 수 있다. 다음 예시를 살펴보자.

```
# 출처: wikipedia
#
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command to execute
```

왼쪽부터 각각 순서대로 분, 시간, 일(월), 월, 일(주)를 뜻한다. 만약 Sunny 앱과 같이 매시간 특정 스크립트를 실행해야 한다면 `0 * * * *`으로 매시간을 나타낼 수 있고, 매달 1일의 오전 1시를 원한다면 `0 1 1 * *`와 같이 표현할 수 있다. 매년 12월 14일 0시는 `0 0 14 12 *`과 같이 표현할 수 있겠다. 더 자세한 cron syntax는 [cron tab guru](https://crontab.guru/)에서 확인할 수 있다. 

cron job 내용을 우분투 등과 같은 유닉스 계열 OS에서는 터미널 상에서 `sudo crontab -e`로 crontab 파일을 열어서 내용을 수정하고 저장하면 된다. 하지만 cron job 목록을 서버 코드와 같이 한번에 관리하는 것이 유지보수와 코드 예측 가능성 등의 이점이 있다. 따라서 Node 환경에서 cron job을 관리하고 실행할 수 있게끔 도와주는 `node cron`과 shell script를 직접 실행할 수 있게 해주는 `shell.js`를 활용해보자. 

<br>
### 필요한 모듈
- [node cron](https://www.npmjs.com/package/node-cron) - 자바스크립트로 구현된 태스크 스케줄러. Node 환경에서 crontab syntax를 정의할 수 있다.
- [shell.js](https://www.npmjs.com/package/shelljs) - Node 환경에서 unix shell script를 실행할 수 있다. shell 명령이 필요하지 않으면 생략 가능하다.

```javascript
// Node 서버 설정을 담고 있는 www 파일
const app = require('../app');
const cron = require('node-cron');
const shell = require('shelljs');

if (process.env.NODE_ENV === 'production') {
  cron.schedule('0 * * * *', () => {
    console.log('running a cron task');
    if (shell.exec(`NODE_ENV=${app.settings.env} node tasks/saveArticlesList.js`).code === 0) {
      // 코드 0으로 프로그램이 종료되면 성공적으로 프로그램이 실행되었다는 뜻
      shell.echo('cron task successfully run');
    } else {
      // 에러 발생했다는 메세지 로그
      shell.exit(1);
      shell.echo('error occurred when running cron task');
    }
  });
}
```
<br>
만약 shell 명령이 필요없는 작업이라면 어떨까? `shell.js` 모듈을 생략하고 해당 함수를 바로 실행하면 된다. 예를 들어 매주 토요일 오전 6시 15분마다 주기적으로 `error.log` 파일을 삭제하는 cron job을 만들어보자.

```javascript
// Node 서버 설정을 담고 있는 www 파일
const app = require('../app');
const cron = require('node-cron');
const fs = require('fs');
// shell script를 실행할 게 아니면 필요 없음
// const shell = require('shelljs'); 
if (process.env.NODE_ENV === 'production') {
  cron.schedule('15 6 * * 6', () => {
    fs.unlink("./error.log", err => {
      if (err) throw err;
      console.log("Error file succesfully deleted");
    });
  })
}
```