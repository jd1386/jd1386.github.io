---
layout: post
title: React form에서 zxcvbn 모듈을 이용해 좀더 안전한 패스워드를 만들도록 유도하기
date: 2019-02-08 00:00:00 +0900
categories: blog
tags: react form
---

지난 글 [React form에서 패스워드 일치 여부 확인하기]({% post_url  2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})에서 우리는 React 폼에서 유저가 패스워드와 패스워드 확인 인풋 필드를 입력하면 두 개의 입력값을 비교해서 일치시키도록 유도했다. 패스워드를 한번만 입력하게 하면 입력할 때 실수를 할 수 있으니 한번 더 입력하게끔 해서 패스워드를 잘못 입력하는 실수를 방지하고자 하는 의도였다.

이번 글에서는 리액트에서 [zxcvbn](https://www.npmjs.com/package/zxcvbn) 모듈을 이용해 유저가 입력하는 패스워드가 얼마나 위험한 패스워드인지, 혹은 안전한 패스워드인지 여부를 확인하고 그 결과를 유저에게 피드백해주는 기능을 만들어보자.

- [Github repository](https://github.com/jd1386/github-form-password-strength-feedback-demo)
- [Live demo](https://react-form-password-strength-feedback-jo1lliy8f.now.sh)

#### 완성된 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-08-03.png">
  <img src="/assets/img/2019-02-08-02.png">
  <img src="/assets/img/2019-02-08-04.png">
  <img src="/assets/img/2019-02-08-01.png">
</div>

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 안전한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-안전한-패스워드를-만들도록-유도하기 %})

## Project Setup
새로운 예제 앱을 만들자.

```shell
$ create-react-app react-form-password-strength-feedback
$ cd react-form-password-strength-feedback && yarn start
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

위 렌더 함수는 지난 글 [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})와 동일한 구조이다.

## zxcvbn 모듈로 패스워드의 안전성을 확인해보자
문자열의 길이가 길 수록, 대문자와 소문자 알파벳이 섞여있을 수록, 숫자와 특수 기호가 포함되어 있을 수록 좀 더 안전한 패스워드라고 할 수 있다. 왜냐하면 안전하고 위험한 패스워드를 나누는 기준은 얼마나 알아내기 (guess) 어려운가로 나뉘기 때문이다. 문자를 하나하나 대입해서 가능한 모든 경우의 수를 대조하는 brute force와 같은 해킹에 좀 더 안전하기 위해서는 비밀번호를 털리지 않는 것이 첫번째이고 암호화 되어있는 비밀번호를 해독하지 못하도록 최대한 어렵게 만들어 놓는 것이 그 다음이다.

기본적인 세팅은 끝났으니 이제 유저가 입력한 패스워드가 얼마나 안전한지 알아내야 한다. 이번 글에서는 이미 검증된 바 있는 [zxcvbn 모듈](https://www.npmjs.com/package/zxcvbn)을 사용해서 패스워드의 안전성을 확인해볼 것이다.

사용법은 단순하다. zxcvbn 모듈을 불러와서

```jsx
import zxcvbn from 'zxcvbn';
```

zxcvbn 함수에 테스트하고자 하는 패스워드를 인자로 넣고 실행하면
```jsx
const passwordInfo = zxcvbn('mypassword');
console.log(zxcvpasswordInfobn);
```
다음과 같은 오브젝트가 반환된다.

#### zxcvbn('mypassword')의 실행 결과
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-08-05.png">
</div>

여기서 재밌는 건 인자로 넘긴 'mypassword'를 guess하기까지 누적된 guess count는 3,860회이고 오프라인에서 베스트의 경우는 'mypassword'를 guess 하는데 채 1초도 걸리지 않는다는 점이다. 누가봐도 안전해보이지 않는 'mypassword'는 결국 score 1을 받았다. 

반면 사용자도 기억하기 어려울 것 같은 'alskdfe12@$%'는 어떨까?

#### zxcvbn('alskdfe12@$%')의 실행 결과
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-08-06.png">
</div>

최고 점수인 4점을 받아서 가장 안전한 수준의 패스워드임을 확인할 수 있다. 적당히 긴 문자열에 알파벳, 숫자, 거기다가 여러개의 특수문자까지 적당히 섞여있으니, 기억할 수만 있다면 꽤나 안전할 것 같다.

zxcvbn 모듈은 이름 만큼은 친절하지 않지만 속도도 빠르고 상당히 잘만든 모듈이라는 점에는 이견이 없을 것 같다. [npm 공식 문서](https://www.npmjs.com/package/zxcvbn)를 보고 더 자세한 정보를 확인해보자.

## state와 handleOnPasswordInput 함수
zxcvbn 모듈에 대해서 알게 되었으니 이제 zxcvbn 모듈을 활용해서 유저가 입력한 패스워드의 안전성을 검사해보자. 

먼저 state에는 zxcvbn 함수의 테스트 결과 중에서 score를 담을 것이다. 

#### App.js
```jsx
state = {
  passwordScore: ''
}
```

`handleOnPasswordInput` 함수는 유저가 인풋 필드를 입력할 때마다 실행되어 zxcvbn 함수에 유저의 패스워드 입력값을 인자로 전달해 실행한 후 반환받은 오브젝트의 score 값을 passwordScore state에 저장한다.

#### handleOnPasswordInput 함수
```jsx
handleOnPasswordInput(passwordInput) {
  const { score } = zxcvbn(passwordInput);
  this.setState({ passwordScore: score });
}
```

## renderFeedbackMessage 함수
`renderFeedbackMessage` 함수는 `passwordScore` state의 값에 따라 각기 다른 메세지를 포함한 div 요소를 반환한다. `passwordScore`가 0이면 말도 안되게 위험한 패스워드라는 점과 4점이면 더할 나위없이 훌륭하게 안전한 패스워드라는 점이 부각되어야 한다. 참고로 text-danger나 text-success와 같은 클래스들은 모두 Bootstrap에서 지원하는 클래스이다.

#### renderFeedbackMessage 함수
```jsx
renderFeedbackMessage() {
  const { passwordScore } = this.state;
  let message, className;

  switch (passwordScore) {
    case 0:
      message = 'Way too weak!';
      className = 'text-danger';
      break;
    case 1:
      message = 'Weak strength!';
      className = 'text-danger';
      break;
    case 2:
      message = 'Moderate strength!';
      className = 'text-warning';
      break;
    case 3:
      message = 'Good strength!';
      className = 'text-success';
      break;
    case 4:
      message = 'Powerful strength!';
      className = 'text-primary';
      break;
    default:
      message = '';
      break;
  }

  return (
    <small id="passwordHelp" className={`form-text mt-2 ${className}`}>
      {`${message}`}
    </small>
  );
}
```


## 프로젝트 완성
이제 다 되었다. 패스워드 인풋 필드에 다양한 패스워드를 입력해보자. 당신의 패스워드 안정성를 입력과 동시에 확인할 수 있다.

#### 완성된 기능
<div style="display: flex; flex-direction: row; justify-content: center; flex-wrap: wrap;">
  <img src="/assets/img/2019-02-08-03.png">
  <img src="/assets/img/2019-02-08-02.png">
  <img src="/assets/img/2019-02-08-04.png">
  <img src="/assets/img/2019-02-08-01.png">
</div>


- [Github repository](https://github.com/jd1386/github-form-password-strength-feedback-demo)
- [Live demo](https://react-form-password-strength-feedback-jo1lliy8f.now.sh)

### React form 관련 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url 2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})
* [React form에서 zxcvbn 모듈을 이용해 좀더 안전한 패스워드를 만들도록 유도하기]({% post_url 2019-02-08-React-form에서-zxcvbn-모듈을-이용해-좀더-안전한-패스워드를-만들도록-유도하기 %})