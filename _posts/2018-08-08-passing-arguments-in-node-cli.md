---
layout: post
title: argv를 이용한 arguments passing
date: 2018-08-08 13:00:00 +0900
categories: blog
tags: node
---

```javascript
// npm i yargs
const arg = require('yargs').argv;

let city = argv.c || 'New York';

// Run program with an argument
node index.js -c Boston
```
