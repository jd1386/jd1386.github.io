---
layout: post
title: util.promisify 사용하기
date: 2018-11-07 13:00:00 +0900
categories: blog
tags: node
---

폴더 안에 있는 파일을 탐색해야 해서 fs.readdir 할 일이 있었다. 그런데 fs.readdir 그리고 대부분의 fs method는 현재 Promise를 native하게 지원하지 않고 있었다.

(실험적으로 fs Promises API를 테스트 중인듯.. [공식 문서 참조](https://nodejs.org/api/fs.html#fs_fs_promises_api). 하지만 언제 stable version으로 release 될 지는 아직 모름..)

아무튼 다음과 같이 Node 코어 모듈인 'util'을 써서 promisify한 다음 쓸 수 밖에 없었다.

```javascript
const fs = require('fs');
const util = require('util');
const readdir = util.promisify(fs.readdir);

readdir(...)
  .then(files => {
    // do something with files
  })
  .catch(err => throw err; )
```

fs.readFile도 같은 방식으로 쓰면 된다.
```javascript
const fs = require('fs');
const util = require('util');
const readFile = util.promisify(fs.readFile);

readFile(...)
  .then(data => {
    // do something with data
  })
  .catch(err => throw err; )
```