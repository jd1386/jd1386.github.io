---
layout: post
title: user-select로 unselectable text 구현하기
date: 2018-12-26 13:00:00 +0900
categories: blog
tags: css
---
리액트에서 text를 포함하는 div를 클릭할 때 실행되는 onClick 이벤트를 만들었다. onClick이 되면 해당 요소의 숨겨져 있던 sub item들이 expand되는 간단한 코드인데.. 

문제는 div는 button이 아니다보니 클릭을 여러번하다보면 텍스트가 선택되는 문제가 발생한다. 이를 해결하기 위해서는 div를 button으로 변환하던지, 아니면 다음과 같이 CSS의 user-select 프로퍼티를 이용해 div내 텍스트가 selectable하지 않게 하는 방법이 있다.

```css
-webkit-touch-callout: none; /* iOS Safari */
-webkit-user-select: none; /* Safari */
-khtml-user-select: none; /* Konqueror HTML */
-moz-user-select: none; /* Firefox */
-ms-user-select: none; /* Internet Explorer/Edge */
user-select: none; /* Non-prefixed version, currently
                      supported by Chrome and Opera */
```

위 user-select 프로퍼티로 div 내 text는 더이상 selectable하지 않게 되었다.