---
layout: post
title: iconv-lite로 한글 인코딩이 깨질 때 수정하기
date: 2018-12-15 16:00:00 +0900
categories: blog
tags: node
---

코드스테이츠의 2주 프로젝트에서 cheerio로 네이버 뉴스 웹사이트 기사 내용을 scrape 할 일이 있었다. 하지만 문제는 인코딩이 euc-KR인지라 utf-8으로 인코딩 변환을 먼저 해야했던 것. 그렇지 않으면 cheerio에서 html을 parse할 때 한글이 깨지는 문제가 생겼다.

이렇게 인코딩 문제로 한글이 깨져서 출력될때 수정하는 방법은 [iconv-lite](https://www.npmjs.com/package/iconv-lite)라는 모듈을 사용해서 utf-8으로 변환해주면 된다.

다음은 그 예제.

```javascript
var request = require("request");
var cheerio = require('cheerio');
var iconv  = require('iconv-lite');

async getContent (url) {
  return new Promise((resolve, reject) => {
    request(url, (err, res, body) => {
      if (err) { throw err; };

      const strContents = Buffer.from(body);
      const html = iconv.decode(strContents, 'utf-8').toString();
      const $ = cheerio.load(html);

      // ... do the rest
    });
  });
}
```