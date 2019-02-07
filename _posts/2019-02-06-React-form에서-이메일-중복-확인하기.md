---
layout: post
title: React form에서 이메일 중복 확인하기
date: 2019-02-06 13:00:00 +0900
categories: blog
tags: react form
---

지난 글 [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})에서 우리는 React 폼에서 이름, 이메일, 핸드폰 번호 등 다양한 인풋 필드를 validate 하는 방법을 연습해 보았다. 지난 주제에 이어 이번 글에서는 폼에서 유저가 입력하는 이메일이 서버에 이미 등록되어 있는 지 확인해서 유저에게 피드백을 주는 기능을 개발해보자. 

대부분의 애플리케이션에서 유저의 이메일 정보는 데이터베이스에 저장되어 있을 것이다. 하지만 이번 예제 프로젝트에서는 데이터베이스와 API를 직접 만들기보다 [JSON Placeholder](https://jsonplaceholder.typicode.com/)에서 제공하는 [유저 목록 API](https://jsonplaceholder.typicode.com/users)를 이용해서 유저 이메일 검색 API를 사용한다고 가정해서 진행해보도록 하자. 

- [Github repository](https://github.com/jd1386/react-form-validation-duplicate-email-demo)
- [Live demo](https://react-form-validation-duplicate-email-demo-nq0viiq3s.now.sh/)

#### 완성된 이메일 중복 확인 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/react-form-duplicate-email-check-2.png">
  <img src="/assets/img/react-form-duplicate-email-check-1.png">
</div>

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 강력한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-강력한-패스워드를-만들도록-유도하기 %})

## Project Setup
새로운 예제 앱을 만들자.

```shell
$ create-react-app react-form-validation-duplicate-email-demo
$ cd react-form-validation-duplicate-email-demo && yarn start
```

먼저 프로젝트 폴더에서 필요하지 않은 모든 파일을 삭제한다.

`index.css`에는 다음 내용만 남겨두고 다 삭제한다.

#### index.css
```css
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

`App.css`에는 폼을 스타일 할 수 있도록 아래와 같이 정의한다.

#### App.css
```css
.my-form {
  max-width: 300px;
  margin: 50px auto 0;
  padding: 20px;
  border: 1px solid lightgrey;
}
```

그 다음 `index.html` 파일의 head 내부에 Bootstrap을 불러오자.

```
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-GJzZqFGwb1QTTN6wy59ffF1BuGJpLSa9DkKMp0DgiMDm4iYMj70gZWKYbI706tWS" crossorigin="anonymous">
```

## Axios 모듈 추가

[JSON Placeholder 유저 목록 API](https://jsonplaceholder.typicode.com/users)에 접근해서 유저 목록을 JSON 파일 형태로 읽고 특정 이메일을 검색하기 위해서는 axios와 같은 모듈을 사용해야 한다. 이를 위해 다음과 같이 axios를 설치한다.

```shell
$ yarn add axios
```

## 기본 form

이제 폼에 살을 더해보자. `App.js`에 불필요한 내용은 모두 삭제하고 렌더 함수만 남겨놓자. 그리고 렌더 함수를 다음 내용으로 교체한다.

#### App.js
```coffee
import React, { Component } from 'react';
import './App.css';
import axios from 'axios';

render() {
  return (
    <div className="App">
      <form className="my-form">
        <div className="form-row">
          <div className="col-md-12 mb-3">
            <label htmlFor="emailInput">이메일</label>
            <input
              type="email"
              className={`form-control ${this.emailInputClassName()}`}
              id="emailInput"
              aria-describedby="emailHelp"
              placeholder="abc@gmail.com"
              onChange={e => this.handleOnChange(e.target.value)}
            />
            {this.renderFeedbackMessage()}
          </div>
        </div>
        <button type="submit" className="btn btn-primary btn-block">
          Submit
        </button>
      </form>
    </div>
  );
}
```

위 렌더 함수에는 3가지 특이점이 있는데:

1. 다른 input field validation 방법과 마찬가지로 유저가 이메일 필드를 입력하기 시작하면 onChange 이벤트에 따라 `handleOnChange` 함수에 입력값을 전달할 것이다.

2. 또한 이메일 인풋의 클래스에는 `emailInputClassName` 함수가 있어서, 폼이 렌더될 때마다 이메일 인풋 필드의 클래스가 `emailInputClassName` 함수의 실행 결과로 업데이트 될 것이다.

3. 마지막으로 `renderFeedbackMessage` 함수는 유저가 입력한 이메일이 중복인 지 아닌 지 피드백 메세지를 리턴할 것이다.

그럼 먼저 `handleOnChange` 함수를 완성해보자.

## state와 handleOnChange 함수

`handleOnChange` 함수는 유저가 이메일 인풋 필드에서 입력한 값을 전달받아 state에 저장하는 함수이다. 이를 위해 우선 state에 typedEmail과 isDuplicateUser를 추가해보자.

#### App.js
```coffee
state = {
  typedEmail: '',
  isDuplicateUser: false
}
```

그 다음 `handleOnChange` 함수를 완성해보자. 유저가 이메일 인풋 필드에 입력한 값을 인자로 전달받아서 해당 이메일을 [JSON Placeholder 유저 목록 API](https://jsonplaceholder.typicode.com/users)에서 검색한다. 해당 이메일이 있으면 중복 유저라는 뜻이므로 isDuplicateUser 상태값을 true로 바꿔준다. 해당 이메일이 없으면 신규 유저이므로 isDuplicateUser 상태값을 false로 바꿔준다.

#### handleOnChange 함수
```coffee
handleOnChange(typedEmail) {
  axios.get('https://jsonplaceholder.typicode.com/users').then(response => {
    const users = response.data;
    const isUserFound = users.filter(user => user.email.toLowerCase() === typedEmail.toLowerCase())
      .length;
    
    isUserFound
    ? this.setState({
        typedEmail,
        isDuplicateUser: true
      })
    : this.setState({
        typedEmail,
        isDuplicateUser: false
      });
  });
}
```

`handleOnChange` 함수를 async 함수로 변환 후 사용하는 것도 가능하다.

#### handleOnChange async 함수
```coffee
async handleOnChange(typedEmail) {
  const response = await axios.get(
      'https://jsonplaceholder.typicode.com/users'
    );

  const users = response.data;
  const isUserFound = users.filter(user => user.email.toLowerCase() === typedEmail.toLowerCase()).length;

  isUserFound
    ? this.setState({
        typedEmail,
        isDuplicateUser: true
      })
    : this.setState({
        typedEmail,
        isDuplicateUser: false
      });
}
```

## emailInputClassName 함수
유저가 이메일을 입력하면 `handleOnChange` 함수가 실행되고 해당 이메일을 [JSON Placeholder 유저 목록 API](https://jsonplaceholder.typicode.com/users)에서 검색한다. 검색 결과에 따라 isDuplicateUser 상태값이 true가 되기도 하고 false가 되기도 한다. 이제는 검색 결과에 따라 유저에게 피드백 메세지를 줄 차례이다.

`emailInputClassName` 함수는 최종적으로 빈 문자열과 is-invalid, is-valid 문자열을 반환하는 함수이다. 유저의 이메일 입력값이 없으면 빈 문자열을 반환하고, 입력값은 있지만 isDuplicateUser인 경우 'is-invalid'를 isDuplicateUser가 아닌 경우 'is-valid'를 반환한다.

#### emailInputClassName 함수
```coffee
emailInputClassName() {
  if (this.state.typedEmail) {
    return this.state.isDuplicateUser ? 'is-invalid' : 'is-valid';
  }
  return '';
}
```

## renderFeedbackMessage 함수
`renderFeedbackMessage` 함수는 `emailInputClassName` 함수와 매우 유사하다. 다른 점은 isDuplicateUser인 경우 이미 등록되어 있는 이메일이라는 메세지를 담은 div를 리턴하고 isDuplicateUser가 아닌 경우 사용할 수 있는 이메일이라는 메세지를 담은 div를 리턴한다는 점이다. invalid-feedback과 valid-feedback 클래스는 모두 Bootstrap에서 지원하는 클래스명이다.

#### renderFeedbackMessage 함수
```coffee
renderFeedbackMessage() {
  if (this.state.typedEmail) {
    return this.state.isDuplicateUser ? (
      <div className="invalid-feedback">이미 등록되어 있는 이메일입니다</div>
    ) : (
      <div className="valid-feedback">사용할 수 있는 이메일입니다</div>
    );
  }
}
```

이제 다 되었다. 데모 프로젝트를 브라우저에서 띄우고 [JSON Placeholder 유저 목록 API](https://jsonplaceholder.typicode.com/users)에 저장되어 있는 이메일 Sincere@april.biz과 그렇지 않은 이메일 Sincere@april.co를 각각 입력해보자. 저장되어 있는 이메일인 경우 이미 등록되어 있는 이메일이라는 메세지와 함께 x자 마크와 빨간색 border가 뜰 것이다. 반대로 저장되어 있지 않은 이메일인 경우 사용할 수 있는 이메일이라는 메세지가 뜰 것이다.

#### 완성된 이메일 중복 확인 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/react-form-duplicate-email-check-2.png">
  <img src="/assets/img/react-form-duplicate-email-check-1.png">
</div>

이번 프로젝트에서 한 가지 유의할 점은 유저가 인풋 필드에 한글자 한글자 입력할 때마다 `handleOnChange` 함수가 실행되어 [JSON Placeholder 유저 목록 API](https://jsonplaceholder.typicode.com/users)에 매 글자마다 검색이 이뤄진다는 점이다. 서버에 부하가 갈 수 있으므로 [lodash의 throttle 함수](https://lodash.com/docs/4.17.11#throttle)를 이용해서 제한을 거는 것이 좋다.


- [Github repository](https://github.com/jd1386/react-form-validation-duplicate-email-demo)
- [Live demo](https://react-form-validation-duplicate-email-demo-nq0viiq3s.now.sh/)

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 강력한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-강력한-패스워드를-만들도록-유도하기 %})