---
layout: post
title: JSON.parse로 문자열로 된 array 읽기
date: 2018-12-15 13:00:00 +0900
categories: blog
tags: javascript
---

문자열로 된 array가 있다.
```javascript
'[1, 2, 3]'
```

이 문자열을 array 오브젝트로 변환하기 위해서는 다음과 같은 방법이 있다:
1. eval()을 이용한다
```javascript
console.log(eval('[1, 2, 3]'));
// => [1, 2, 3]
```
물론 eval 사용은 지양해야 한다.

2. JSON.parse()를 이용한다
```javascript
console.log(JSON.parse('[1, 2, 3]'));
// => [1, 2, 3]
```

