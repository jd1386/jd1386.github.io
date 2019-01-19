---
layout: post
title: SCSS @extend를 사용해서 CSS 룰 중복 사용을 최소화하기
date: 2019-01-08 14:00:00 +0900
categories: blog
tags: css
---

```css
%transition-effect-1 {
  -webkit-transition: color 0.2s ease-in-out;
  -moz-transition: color 0.2s ease-in-out;
  -o-transition: color 0.2s ease-in-out;
  transition: color 0.2s ease-in-out;
}

%transition-effect-2 {
  -webkit-transition: color 0.2s ease-in-out, background-color 0.2s ease-in-out;
  -moz-transition: color 0.2s ease-in-out, background-color 0.2s ease-in-out;
  -o-transition: color 0.2s ease-in-out, background-color 0.2s ease-in-out;
  transition: color 0.2s ease-in-out, background-color 0.2s ease-in-out;
}

%transition-effect-3 {
  -webkit-transition: font-weight 0.1s ease-in-out, color 0.2s ease-in-out;
  -moz-transition: font-weight 0.1s ease-in-out, color 0.2s ease-in-out;
  -o-transition: font-weight 0.1s ease-in-out, color 0.2s ease-in-out;
  transition: font-weight 0.1s ease-in-out, color 0.2s ease-in-out;
}

%transition-effect-4 {
  -webkit-transition: font-weight 0.2s ease-in-out, color 0.2s ease-in-out,
    font-weight 0.2s ease-in-out;
  -moz-transition: font-weight 0.2s ease-in-out, color 0.2s ease-in-out,
    font-weight 0.2s ease-in-out;
  -o-transition: font-weight 0.2s ease-in-out, color 0.2s ease-in-out,
    font-weight 0.2s ease-in-out;
  transition: font-weight 0.2s ease-in-out, color 0.2s ease-in-out;
}
```

```css
.sidebar-sprint-entry {
    @extend %transition-effect-2;
    &:first-child {
      margin-top: 70px;
    }
    &:hover {
      cursor: pointer;
      background: #eee;
    }
    .left-menu {
      display: flex;
      align-items: center;
    }
```
