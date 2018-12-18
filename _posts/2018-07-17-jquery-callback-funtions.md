---
layout: post
title: jQuery에서 콜백 함수
date: 2018-07-17 14:00:00 +0900
categories: blog
---

# jQuery Callback Functions

자바스크립트에서와 마찬가지로 jQuery에 콜백 함수를 사용할 수 있다. jQuery에서 이펙트가 완료되지 않았는데도 다음 행의 코드가 실행될 수 있기 때문에 이런 경우 explicit하게 콜백 함수를 인자로 넘겨주어야 한다.



#### Without callback parameter

```javascript
$("button").click(function(){
    $("p").hide(1000);
    alert("The paragraph is now hidden");
});
```

위 예시 코드에서:

```javascript
$("p").hide(1000) 
// 라는 이펙트가 끝나기도 전에 (1초의 hide duration이 필요)
// 아래 alert이 실행되어 버림
alert('The paragraph is now hidden');
```

이런 경우 의도와 다르게 이펙트 순서가 꼬이는 문제가 발생할 수 있다. 또한 디버깅도 어려울 수 있다. 따라서 콜백 함수를 인자로 explicit하게 넘겨주는 방법이 필요하다.



#### With callback parameter

```javascript
$('button').click(() => {
  $('p').hide('slow', () => {
    alert('The paragraph is now hidden')
  })
})
```

위 코드를 실행해보면 버튼을 클릭하고 나서 hide 이펙트가 끝나고, 직후 alert이 뜨는 걸 확인할 수 있다. 이런 식으로 sequential하게 코드가 실행되기 때문에 원치않는 에러나 디버깅에 쓸데없이 시간 낭비할 가능성이 줄게 된다.