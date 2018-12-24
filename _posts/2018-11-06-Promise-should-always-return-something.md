---
layout: post
title: Promise는 항상 객체를 반환해야 한다
date: 2018-11-06 13:00:00 +0900
categories: blog
tags: javascript
---

아래와 같은 코드가 있었음.
```javascript
await scraper.getArticleContent(urls[i])
  .then(articleContent => {
    Article.findOne({
      where: { aid: articleContent.aid }
    })
      .then(savedArticle => {
        if (savedArticle.aid === articleContent.aid) {
          savedArticle.update({
            publisher: articleContent.publisher
          });
        }
      })
      .catch(err => { throw err; });
  })
  .catch(err => { throw err; });
```

그리고 다음과 같은 에러 메세지가 떴음.
```
Warning: a promise was created in a handler at /Users/jungdolee/projects/codestates/sunny/server-sunny/tasks/saveArticlesList.js:106:28 but was not returned from it, see http://goo.gl/rRqMUw
    at Function.Promise.attempt.Promise.try (/Users/jungdolee/projects/codestates/sunny/server-sunny/node_modules/bluebird/js/release/method.js:29:9)
```

[여기에서](https://github.com/petkaantonov/bluebird/blob/master/docs/docs/warning-explanations.md#warning-a-promise-was-created-in-a-handler-but-was-not-returned-from-it) 조사를 시작했음.

요약:
Promise 안에서는 다음 .then()에서 인자로 사용할 객체를 return 해주라는 말씀.

위 코드는 아래와 같이 Promise안에서 return null을 해주었음. 특별히 return 해야할 것이 없는 상황이었기 때문. 하지만 다음 .then()에서 해당 객체를 사용하기 위해서는 무조건 그 객체를 반환 해주어야 함.

```javascript
await scraper.getArticleContent(urls[i])
  .then(articleContent => {
    Article.findOne({
      where: { aid: articleContent.aid }
    })
      .then(savedArticle => {
        if (savedArticle.aid === articleContent.aid) {
          savedArticle.update({
            publisher: articleContent.publisher
          });
          return null; // or return savedArticle;
        }
      })
      .catch(err => { throw err; });
  })
  .catch(err => { throw err; });
```
