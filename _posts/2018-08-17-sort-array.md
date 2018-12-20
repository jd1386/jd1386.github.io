---
layout: post
title: Array 정렬하기
date: 2018-08-17 13:00:00 +0900
categories: blog
tags: javascript
---

#### alphabetical order
```javascript
var nonSortedArray = ['hi', 'yo', 'whatup', 'bye', 'lol'];
var sortedArray = nonSortedArray.sort(function (a, b) {
      if (a < b) return -1;
      else if (a > b) return 1;
      return 0;
    });
console.log(sortedArray); // ["bye", "hi", "lol", "whatup", "yo"]
```


#### reverse alphabetical order

```javascript
var nonSortedArray = ['hi', 'yo', 'whatup', 'bye', 'lol'];
var reverseSortedArray = nonSortedArray.sort(function (a, b) {
      if (a > b) return -1;
      else if (a < b) return 1;
      return 0;
    });
console.log(reverseSortedArray); // ["yo", "whatup", "lol", "hi", "bye"]
```
