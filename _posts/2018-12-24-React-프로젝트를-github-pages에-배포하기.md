---
layout: post
title: React 프로젝트를 github pages에 배포하기
date: 2018-12-24 13:00:00 +0900
categories: blog
tags: react dev-ops
---

최근 글 ['react-on-screen을 이용한 컴포넌트 lazy load 전략']({% post_url 2018-12-22-react-on-screen을-이용한-컴포넌트-lazy-load-전략 %})을 위해 간단한 데모용 [React 프로젝트](https://leejungdo.com/react-on-screen-test/){:target="_blank"}를 만들었었다. 데모 사이트를 배포하는 방법은 Heroku도 좋지만 이번 글에서는 [gh-pages](https://www.npmjs.com/package/gh-pages){:target="_blank"}라는 모듈을 이용해 React 프로젝트를 Github Pages에 손쉽게 배포하는 방법을 알아보자. 

Github remote repository를 이미 추가했다면 3번부터 진행하면 된다.

1. React 앱 root directory에서 `git init`을 실행한다.

2. remote repository를 추가한다.
```shell
git remote add origin https://github.com/{YOUR-GITHUB-ID}/{YOUR-REPO-NAME}.git
```

3. `yarn add --dev gh-pages`로 gh-pages를 설치한다.

4. 그 다음 `package.json`을 열어서 다음과 같이 homepage를 추가해준다.
```shell
//...
"homepage": "https://{YOUR-GITHUB-ID}/github.io/{YOUR-REPO-NAME}"
```

5. 같은 `package.json` 파일에서 "scripts" 부분에 다음을 추가해준다.
```shell
//...
"predeploy": "npm run build",
"deploy": "gh-pages -d build",
```

6. 배포한다.
```shell
npm run deploy
```
이 과정에서 gh-pages라는 remote branch에 build된 파일들이 업로드 될 것이다.
<img src="/assets/img/gh-pages-branch.png">

7. 푸시한다. (생략 가능)
```shell
git add .
git commit -m "Deploy to Github Pages"
git push origin master
```

이제 다 되었다. `package.json`의 'homepage'에 설정한 url로 이동하면 다음과 같이 배포가 되었을 것이다.

<img src="/assets/img/react-on-screen-example.gif">