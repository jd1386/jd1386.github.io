---
layout: post
title: CSS에서 상대적 위치값 조정
date: 2018-07-17 14:00:00 +0900
categories: blog
tags: css
---


```javascript
$('#button').on('click', event => {
  $('.example').animate({
    opacity: 0.25,
    left: "+=50"
  }, 5000); 
});
```

위 예제 코드에서 "+=50" 이라는 value는 relative change를 의미한다. 즉 현재 있는 포지션에서 +50px 만큼의 위치로 animate 하라는 것.

반대로 -50px 만큼의 위치를 나타내는 "-=50" 또한 가능하다.

[이곳](https://www.w3schools.com/jquery/tryit.asp?filename=tryjquery_animation1_relative)에서 실시간으로 code snippet을 확인 할 수 있다.
