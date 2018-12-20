---
layout: post
title: 변수명과 object property명
date: 2018-11-06 13:00:00 +0900
categories: blog
tags: javascript
---

변수 이름과 object property 이름이 곂치지 않게 주의해야 한다. 별 생각없이 하다간 오늘처럼 몇십분동안 시간을 허비하게 된다.

```javascript
const naverURLParser = (url) => {
    const parsed = queryString.parseUrl(url);
    return {
      sid1: parsed.query.sid1,
      oid: parsed.query.oid,
      aid: parsed.query.aid
    };
  }

// 문제가 있었던 부분.. 한참 에러 이유를 모르고 찾아 해맸음..
let aid = naverURLParser(url).aid;

//// 솔루션
// 1. 다른 변수명을 만들거나
let newAid = naverURLParser(url).aid;
// 2. 괄호를 쓰거나
let aid = naverURLParser(url)['aid'];
```

생각보다 눈에 잘 띄지 않아서 디버깅하는데 괜히 오랜 시간 잡아먹었다..
