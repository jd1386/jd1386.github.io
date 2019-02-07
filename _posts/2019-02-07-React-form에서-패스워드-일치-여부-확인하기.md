---
layout: post
title: React form에서 패스워드 일치 여부 확인하기
date: 2019-02-07 13:00:00 +0900
categories: blog
tags: react form
---

지난 글 [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})에서 우리는 React 폼에서 유저가 이메일을 입력하면 데이터베이스에 저장되어 있는 유저 정보 중 중복된 이메일이 없는지 확인하고 그 결과를 유저에게 피드백 해주는 기능을 만들었다.

보통의 웹사이트에서 회원가입 폼을 생각해보면 유저에게 패스워드를 입력하고 다시 한번 입력하게 한다. 그 이유는 유저도 실수를 할 수 있기 때문에 유저가 의도한 패스워드를 제대로 입력했는지 재차 확인하고자 하는 의도이다.

이번 글에서는 리액트를 이용해서 폼에서 유저가 입력하는 패스워드가 제대로 입력되었는지 확인하고 그 결과를 유저에게 피드백해주는 기능을 만들어보자.

- [Github repository](https://github.com/jd1386/react-form-password-confirmation-demo)
- [Live demo](https://react-form-password-confirmation-bftg0rvv4.now.sh/)

#### 완성된 패스워드 일치 여부 확인 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-07-01.png">
  <img src="/assets/img/2019-02-07-02.png">
</div>

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 안전한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-안전한-패스워드를-만들도록-유도하기 %})

## Project Setup
새로운 예제 앱을 만들자.

```shell
$ create-react-app react-form-password-confirmation
$ cd react-form-password-confirmation && yarn start
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

## 기본 form

이제 폼에 살을 더해보자. `App.js`에 불필요한 내용은 모두 삭제하고 렌더 함수만 남겨놓자. 그리고 렌더 함수를 다음 내용으로 교체한다.

#### App.js
```jsx
import React, { Component } from 'react';
import './App.css';

render() {
  return (
    <div className="App">
      <form className="my-form">
        <div className="form-row">
          <div className="col-md-12 mb-3">
            <label htmlFor="passwordInput">패스워드</label>
            <input
              type="password"
              className="form-control"
              id="passwordInput"
              onChange={e => this.handleOnPasswordInput(e.target.value)}
            />
          </div>
          <div className="col-md-12 mb-3">
            <label htmlFor="confirmPasswordInput">패스워드 확인</label>
            <input
              type="password"
              className={`form-control ${this.confirmPasswordClassName()}`}
              id="confirmPasswordInput"
              onChange={e =>
                this.handleOnConfirmPasswordInput(e.target.value)
              }
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

위 렌더 함수는 지난 글 [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})와 동일한 구조이다. 몇가지 특이점이 있다면:

1. 두개의 인풋 필드, passwordInput과 confirmPasswordInput이 있고

2. 다른 input field validation 방법과 마찬가지로 유저가 passwordInput 필드에 입력하기 시작하면 onChange 이벤트에 따라 `handleOnPasswordInput` 함수에 입력값을 전달할 것이다.

3. 유저가 confirmPasswordInput 필드에 입력하기 시작하면 `handleOnConfirmationPasswordInput` 함수에 입력값을 전달할 것이다.

4. `renderFeedbackMessage` 함수는 유저가 입력한 패스워드가 일치하는지, 피드백 메세지를 리턴할 것이다.

그럼 먼저 `handleOnPasswordInput` 함수와 `handleOnConfirmPasswordInput` 함수를 완성해보자.

## state, handleOnPasswordInput 함수, handleOnConfirmPasswordInput 함수

`handleOnPasswordInput`와 `handleOnConfirmPasswordInput` 함수는 유저가 각각의 필드에서 입력한 값을 전달받아 state에 저장하는 함수이다. 이를 위해 우선 state에 password와 confirmPassword를 추가해보자.

#### App.js
```jsx
state = {
  password: '',
  confirmPassword: ''
}
```

그 다음 각각의 함수를 완성해보자. 

#### handleOnPasswordInput 함수
```jsx
handleOnPasswordInput(passwordInput) {
  this.setState({ password: passwordInput });
}
```

#### handleOnConfirmPasswordInput 함수
```jsx
handleOnConfirmPasswordInput(confirmPasswordInput) {
  this.setState({ confirmPassword: confirmPasswordInput });
}
```

## doesPasswordMatch 함수
유저가 password를 입력하고나서 confirmPassword를 입력할 때 두 개가 정확히 일치하는지 여부를 확인해야 한다. `doesPasswordMatch` 함수가 일치 여부를 판별해서 true나 false를 반환한다.

#### doesPasswordMatch 함수
```jsx
doesPasswordMatch() {
  const { password, confirmPassword } = this.state;
  return password === confirmPassword;
}
```

## confirmPasswordClassName 함수
confirmPasswordInput 인풋 필드에 녹색(성공)과 빨간색(에러)의 스타일링을 추가하기 위해서는 클래스명을 반환하는 함수가 필요하다. `confirmPasswordClassName` 함수는 `doesPasswordMatch` 함수의 실행 결과에 따라 true면 is-valid, false면 is-invalid 클래스를 반환한다.

#### confirmPasswordClassName 함수
```jsx
confirmPasswordClassName() {
  const { confirmPassword } = this.state;

  if (confirmPassword) {
    return this.doesPasswordMatch() ? 'is-valid' : 'is-invalid';
  }
}
```

## renderFeedbackMessage 함수
`renderFeedbackMessage` 함수는 유저에게 패스워드가 일치하는지 여부를 피드백 주는 기능을 한다. 유저가 confirmPassword를 입력하고 password와 일치하지 않을 때 'invalid-feedback'이라는 클래스를 가진 div안에 메세지를 넣어서 반환한다.

#### renderFeedbackMessage 함수
```jsx
renderFeedbackMessage() {
  const { confirmPassword } = this.state;

  if (confirmPassword) {
    if (!this.doesPasswordMatch()) {
      return (
        <div className="invalid-feedback">패스워드가 일치하지 않습니다</div>
      );
    }
  }
}
```

## 프로젝트 완성
이제 다 되었다. password 필드와 passwordConfirmation 필드를 동일하게 입력해보고 다르게도 입력해보자. 각각의 경우마다 피드백 메세지가 렌더되는 것을 확인할 수 있다.

#### 완성된 패스워드 일치 여부 확인 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-07-01.png">
  <img src="/assets/img/2019-02-07-02.png">
</div>


- [Github repository](https://github.com/jd1386/react-form-password-confirmation-demo)
- [Live demo](https://react-form-password-confirmation-bftg0rvv4.now.sh/)

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 안전한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-안전한-패스워드를-만들도록-유도하기 %})