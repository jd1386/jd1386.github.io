---
layout: post
title: React에서 인풋 필드 validate 하기
date: 2019-01-26 13:00:00 +0900
categories: blog
tags: react form
---

폼에서 input field validation이란 무엇일까? 유저는 폼에서 이메일 주소나 핸드폰 번호 같은 문자열을 직접 타이핑하는 경우가 많다. 예를 들어 010-1234-1234로 입력해야 하는 핸드폰 번호를 유저가 01012341234라고 입력할 수도 있고 010-1234-12344와 같이 입력할 수도 있다. 사람이 직접 손으로 친다는 건 많은 실수의 가능성을 담고 있기 때문에 이러한 일을 사전에 클라이언트단에서 (그리고 서버에서도) 막아주는 validation이 꼭 필요하다. 

리액트는 상태(state)와 상위 컴포넌트에서 전달받은 props에 변화가 있을 때마다 해당 컴포넌트를 다시 렌더해준다. 이러한 단순하지만 강력한 리액트의 장점을 살려서 바닐라 자바스크립트나 jQuery로 일일이 구현하기 까다로운 기능을 리액트로는 쉽게 구현할 수 있다. 이번 글에서는 리액트를 이용해 form에서 input field validation을 손쉽게 구현해보자.

- [Github Repo](https://github.com/jd1386/react-form-validation)
- [Live Demo](https://react-form-validation-6i6h1u8xu.now.sh/)

#### 구현하고자 하는 추가 정보 입력 폼
<img src="/assets/img/react-form-validation-complete.png" class="center">


먼저 예제 코드의 폼에는 유저의 이름과 이메일, 휴대폰 총 3가지의 input field가 있다. 모두 유저가 직접 입력해야 하는 필드이고 우리가 구현해야 하는 것은 각각의 validation이다. 또한 유저가 키를 입력할 때마다 validation 결과를 실시간으로 보여주면 좋을 것 같다. 

그럼 우리가 달성해야 하는 목표를 세워보자.

1. 유저의 이름, 이메일, 휴대폰 번호 총 3개의 인풋 필드가 있어야 한다.
2. 각 필드에 하나씩 총 3개의 validation 로직이 있어야 한다.
3. 각 필드에 유저가 입력을 할 때마다 validation 로직을 통해 실시간 피드백을 준다.
4. 모든 validation 로직이 모두 충족될 때만 Submit 버튼을 클릭할 수 있다.

먼저 가장 중요한 각 field의 validation logic은 다음과 같이 정의해볼 수 있겠다.

- 유저가 이름을 입력할 때는 최소 2글자 이상일 때 유효하다.
- 이메일을 입력할 때는 이메일의 고유 형식에 부합해야 한다.
- 휴대폰 번호를 입력할 때 역시 특정 형식 (000-000-0000 또는 000-0000-0000)에 부합해야 한다.

위 validation logic은 사실 정답이 있다고 말할 수는 없고 서비스에서 요구하는 스펙에 부합하기만 하면 되기 때문에 하나의 예제라고 생각하면 좋을 것 같다. 자 그럼 본격적으로 코딩을 시작해보자.

<br>

## 프로젝트 기본 세팅

먼저 `create-react-app`으로 예제 프로젝트를 만들어보자.

`$ create-react-app form-validation`

그 다음 App.js, App.css, index.js를 제외한 필요없는 파일을 모두 지워주자.

폼의 기본적인 스타일링을 위해서 [Bootstrap 폼](https://getbootstrap.com/docs/4.2/components/forms/)을 써보도록 하자. 다음 Bootstrap css 소스를 index.html의 <head> 태그 안에 넣자.

`<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-GJzZqFGwb1QTTN6wy59ffF1BuGJpLSa9DkKMp0DgiMDm4iYMj70gZWKYbI706tWS" crossorigin="anonymous">`

그리고 `App.js`에서 불필요한 부분을 모두 제거하고 다음 부분만 남겨두자.

```coffee
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  render() {
    return <div className="App">Hello World</div>;
  }
}

export default App;
```

그 다음 서버를 시작하고

```
$ cd form-validation && yarn start
```

브라우저에서 localhost:3000에 들어가고, 빈 화면에 Hello World 글자가 보이면 성공이다. 이제 기본적인 boilerplate는 준비되었으니 Bootstrap을 이용해 폼을 만들어볼 차례이다.

<br>
## Bootstrap 폼 만들기

[Bootstrap 홈페이지](https://getbootstrap.com/docs/4.2/components/forms/)에 다양한 폼 정보와 예제 코드가 있다. 우린 유저의 이름과 이메일 그리고 휴대폰 번호를 입력받는 폼을 만들 것이기 때문에 `App.js` 파일에 다음과 같이 폼을 추가해보자.

#### App.js
```coffee
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <form className="myForm">
          <div className="form-group">
            <label htmlFor="nameInput">이름</label>
            <input
              type="text"
              className="form-control"
              id="nameInput"
              placeholder="홍길동"
            />
          </div>
          <div className="form-group">
            <label htmlFor="emailInput">이메일</label>
            <input
              type="email"
              className="form-control"
              id="emailInput"
              aria-describedby="emailHelp"
              placeholder="abc@gmail.com"
            />
          </div>
          <div className="form-group">
            <label htmlFor="phoneNumberInput">휴대폰 번호</label>
            <input
              type="text"
              className="form-control"
              id="phoneNumberInput"
              placeholder="010-1234-1234"
            />
          </div>
          <button type="submit" className="btn btn-primary btn-block">
            Submit
          </button>
        </form>
      </div>
    );
  }
}

export default App;
```

그리고 `App.css` 파일에는 폼 관련 스타일을 추가해보자.

#### App.css
```css
.myForm {
  max-width: 300px;
  margin: 50px auto 0;
  padding: 20px;
  border: 1px solid lightgrey;
}
```

그러면 다음과 같은 폼이 화면에 렌더될 것이다.


<img src="/assets/img/react-form-validation-1.png" class="center">


<br>

## state와 onChange 이벤트

본격적으로 validation logic을 추가하기 전에 선행되어야 하는 작업은 먼저 각 필드에 onChange 이벤트를 추가해서 state에 저장하는 것이다. 그럼 state에는 어떤 정보가 필요할까?

먼저 유저가 입력한 값을 받을 state가 각 필드마다 필요하고 해당 입력 값이 valid한 지 여부를 저장할 state가 하나씩 필요하다. 다음 state 오브젝트를 `App.js`에 추가해보자.

```javascript
state = {
  nameEntered: '',
  isNameValid: false,
  emailEntered: '',
  isEmailValid: false,
  phoneNumberEntered: '',
  isPhoneNumberValid: false
};
```

유저가 이름을 입력하면 onChange 이벤트를 통해 nameEntered에 저장될 것이고 저장된 nameEntered는 validation 로직을 통해 유효 여부가 isNameValid에 저장될 것이다. 그럼 이제 onChange 이벤트를 만들어보자. 먼저 `App.js`에서 이름 인풋 필드에 다음과 같이 onChange 이벤트를 넣어준다.

#### App.js
```coffee
<div className="form-group">
  <label htmlFor="nameInput">이름</label>
  <input
    type="text"
    className="form-control"
    id="nameInput"
    placeholder="홍길동"
    onChange={e => this.validateName(e.target.value)}
    required
  />
</div>
```

이름 인풋 필드에 변화가 발생하면, 즉 유저가 새로운 값을 입력할 때마다 onChange 이벤트를 통해 validateName 함수에 해당 인풋 필드의 입력값이 인자로 전달되어 실행된다. validateName 함수가 가장 중요한 일인 유저의 이름 입력값의 유효 여부를 판단할 것이다. 그럼 validateName 함수를 `App.js`에 만들어보자.

#### App.js
```coffee
validateName = nameEntered => {
  if (nameEntered.length > 1) {
    this.setState({
      isNameValid: true,
      nameEntered
    });
  } else {
    this.setState({
      isNameValid: false,
      nameEntered
    });
  }
};
```

validateName 함수는 onChange 이벤트를 통해 유저의 이름 입력값을 인자로 받아서 입력값이 유효한 지 여부를 판단한다. 여기서는 간단하게 최소 2글자 이상이면 유효하다고 판단하기로 하자. 입력값이 유효하면 isNameValid state를 true로 바꿔주고 nameEntered state에는 넘겨받은 입력값을 그대로 넣어준다. 만약 입력값이 2글자가 되지 않아 유효하지 않다면 isNameValid는 false가 된다.

## validation 결과를 유저에게 보여주자

기본적인 뼈대는 갖추어져서 이제 유저의 이름 입력값이 유효한 지를 코드 내부적으로는 판단할 수 있게 되었다. 하지만 유저에게 유효 여부를 실시간으로 알려주는 작업이 필요하다. 방법이야 정말 다양하겠지만 편의를 위해 Bootstrap이 제공하는 스타일을 활용해보자. 

먼저 유저가 이름 인풋 필드의 입력값이 유효하면 해당 인풋 필드를 초록색으로 감싸주고 유효하지 않다면 빨간색으로 감싸주자. 굳이 border 스타일을 만들지 않고도 Bootstrap은 is-valid와 is-invalid 클래스를 제공한다. 각 경우에 맞는 클래스가 인풋에 추가되도록 해보자.

먼저 이름 인풋의 className을 다음과 같이 수정해주자.

```
className={`form-control ${this.inputClassNameHelper(this.isEnteredNameValid())}`}
```

기존 className에는 "form-control"처럼 클래스명이 하드코드 되어있었지만 실시간으로 유저에게 유효 여부를 알려주기 위해서는 클래스명을 동적으로 지정 해줄 수 밖에 없다. 그래서 string interpolation을 이용해서, `inputClassNameHelper()` 함수를 통해 Bootstrap의 is-valid와 is-invalid 클래스를 넣을 것이다.

그럼 `isEnteredNameValid()` 함수와 `inputClassNameHelper()` 함수를 만들어보자.

#### App.js
```coffee
isEnteredNameValid = () => {
  const { nameEntered, isNameValid } = this.state;

  if (nameEntered) return isNameValid;
};

inputClassNameHelper = boolean => {
  switch (boolean) {
    case true:
      return 'is-valid';
    case false:
      return 'is-invalid';
    default:
      return '';
  }
};
```

`isEnteredNameValid()` 함수는 단순히 nameEntered state가 있을 때 isNameValid state를 반환하는 역할만 한다. 굳이 이 함수를 쓰지 않고 isNameValid state를 쓰면 안될까 싶기도 하지만, 그렇게 하면 isNameValid state는 true와 false, 단 두 개의 상태 밖에 없기 때문에 인풋 필드에 아무런 입력을 하지 않은 상태일 때도 인풋 필드에 is-invalid 클래스가 추가되어 빨간 border가 생긴다.

이제 이름 인풋 필드에 2글자 이상되는 이름을 넣어보자. 1글자일 때는 유효하지 않으므로 빨간색 border와 X자가 생겨서 유저에게 잘못된 입력임을 알려준다. 2글자 이상 입력하면 초록색 border와 체크 마크가 보여지고 유저에게 유효한 입력임을 알려주게 된다. 그리고 한 글자도 입력하지 않은 상태, 그러니까 nameEntered state가 빈 문자열인 상태라면 `isEnteredNameValid()` 함수가 undefined를 반환하고 undefined를 인자로 넘겨받은 `inputClassNameHelper()` 함수는 빈 문자열을 반환해서 이름 인풋의 className에는 부가적인 클래스가 추가되지 않는다.

#### 유저의 이름 인풋 validation 결과에 따라 인풋의 class가 동적으로 변하고 있다
<img src="https://media.giphy.com/media/9JptW6XjMqAvLrWOnD/giphy.gif" class="center">

## 이메일 validation 추가하기

다음은 이메일 주소를 validate 해보자.

이름 validation과 방법은 똑같다. 먼저 폼에 이메일 주소를 입력할 새로운 인풋 필드를 만들어보자. 그리고 onChange 이벤트에 `validateEmail()`이라는 함수가 실행되도록 하자.

#### App.js
```coffee
<div className="form-group">
  <label htmlFor="emailInput">이메일</label>
  <input
    type="email"
    className="form-control"
    id="emailInput"
    aria-describedby="emailHelp"
    placeholder="abc@gmail.com"
    onChange={e => this.validateEmail(e.target.value)}
    required
  />
</div>
```

이메일 인풋에 유저가 입력을 시작하면 onChange 이벤트가 시작되어 인풋 필드의 값이 `validateEmail()` 함수의 인자로 전달되어 실행될 것이다. 그럼 이젠 `validateEmail()` 함수를 만들어보자.

#### App.js
```coffee
validateEmail = emailEntered => {
  const emailRegExp = /^[\w-]+(\.[\w-]+)*@([a-z0-9-]+(\.[a-z0-9-]+)*?\.[a-z]{2,6}|(\d{1,3}\.){3}\d{1,3})(:\d{4})?$/;

  if (emailEntered.match(emailRegExp)) {
    this.setState({
      isEmailValid: true,
      emailEntered
    });
  } else {
    this.setState({
      isEmailValid: false,
      emailEntered
    });
  }
};
```

이름을 validate하는 `validateName()` 함수와의 유일한 차이점은 이메일은 이름과 달리 유저의 입력값이 고유한 이메일 형식인 지 확인해야 한다는 점이다. 이메일 형식인 지 확인할 수 있는 방법은 편의성과 나름의 정확성 때문에 정규표현식을 쓰는 것이 합당하다. 물론 상업 서비스에서는 검증된 알고리즘을 사용하는 것이 바람직하기 때문에 [email-validator](https://www.npmjs.com/package/email-validator)와 같은 검증된 모듈을 사용하는 것도 좋은 방법이다.


`isEnteredEmailValid()` 함수는 `isEnteredNameVaild()` 함수와 같은 구조를 지녔다.

```coffee
isEnteredEmailValid = () => {
  const { emailEntered, isEmailValid } = this.state;

  if (emailEntered) return isEmailValid;
};
```

그 다음 이메일 인풋의 className을 이름 인풋과 마찬가지로 바꿔준다. 이번에는 inputClassNameHelper의 인자로 isEnteredEmailValid() 함수를 실행해서 전달한다.

```
className={`form-control ${this.inputClassNameHelper(this.isEnteredEmailValid())}`}
```

#### 유저의 이메일 인풋 validation 결과에 따라 인풋의 class가 동적으로 변하고 있다
<img src="https://media.giphy.com/media/3BMQEasfoOqAEI91UR/giphy.gif" class="center">

## 휴대폰 번호 validation 추가하기

이메일 validation을 문제없이 했다면 휴대폰 번호도 어렵지 않다. 먼저 다음과 같이 폼에 휴대폰 번호 인풋 필드를 추가하자.

#### App.js
```coffee
<div className="form-group">
  <label htmlFor="phoneNumberInput">휴대폰 번호</label>
  <input
    type="text"
    className={`form-control ${this.inputClassNameHelper(this.isEnteredPhoneNumberValid())}`}
    id="phoneNumberInput"
    placeholder="010-1234-1234"
    onChange={e => this.validatePhoneNumber(e.target.value)}
    required
  />
</div>
```

유저가 휴대폰 번호를 입력할 때마다 실행될 `validatePhoneNumber()` 함수를 만들어보자.

#### App.js
```coffee
validatePhoneNumber = phoneNumberInput => {
  const phoneNumberRegExp = /^\d{3}-\d{3,4}-\d{4}$/;

  if (phoneNumberInput.match(phoneNumberRegExp)) {
    this.setState({
      isPhoneNumberValid: true,
      phoneNumberEntered: phoneNumberInput
    });
  } else {
    this.setState({
      isPhoneNumberValid: false,
      phoneNumberEntered: phoneNumberInput
    });
  }
};
```

#### App.js
```coffee
isEnteredPhoneNumberValid = () => {
  const { phoneNumberEntered, isPhoneNumberValid } = this.state;

  if (phoneNumberEntered) return isPhoneNumberValid;
};
```

#### 유저의 휴대폰 번호 인풋 validation 결과에 따라 인풋의 class가 동적으로 변하고 있다
<img src="https://media.giphy.com/media/BouaHqht7vkZoMtB1q/giphy.gif" class="center">


## conditional submit 버튼 만들기

이제 이름, 이메일, 휴대폰 번호 3개의 인풋 필드를 validation하고 그 결과를 유저가 타이핑할 때마다 실시간으로 피드백하는 것까진 완성했다. 하지만 아직 한 가지 문제가 남아있다. 유저가 모든 필드를 넣기 전에도 폼을 submit 할 수 있다는 것이다. 물론 각 인풋마다 required 프로퍼티를 넣어둬서 브라우저가 폼 제출을 불가능하게 해놨지만 우린 유저에게 조금 더 친절할 필요가 있다. 유저가 모든 인풋 필드를 채워넣고 모든 유효성 검사를 통과할 때만 폼 제출 버튼이 활성화되도록 만들어보자. 

먼저 3개의 인풋 필드의 유효성을 모두 통과했는지 확인할 수 있는 함수 `isEveryFieldValid()`가 필요하다.

```coffee
isEveryFieldValid = () => {
  const { isNameValid, isEmailValid, isPhoneNumberValid } = this.state;
  return isNameValid && isEmailValid && isPhoneNumberValid;
}
```

그리고 `isEveryFieldValid()` 함수 결과에 따라 폼 제출 버튼을 렌더할 또다른 함수 `renderSubmitBtn()`을 만들어보자.

```coffee
renderSubmitBtn = () => {
  if (this.isEveryFieldValid()) {
    return (
      <button type="submit" className="btn btn-primary btn-block">
        Submit
      </button>
    )
  } 

  return (
    <button type="submit" className="btn btn-primary btn-block" disabled>
      Submit
    </button>
  )
}
```

거의 다되었다. 마지막으로 폼 하단 제출 버튼을 지우고 그 자리에 `renderSubmitBtn()` 함수 실행하자.

```coffee
{this.renderSubmitBtn()}
```

## validation 완성

이제 인풋 필드 validation이 완성되었다. `App.js` 파일을 저장하고 이제 인풋 필드를 하나씩 작성해보자. 

#### 완성된 폼 validation
<img src="https://media.giphy.com/media/oVu0Y2iapp0cWHm7QJ/giphy.gif" class="center">

이번 프로젝트의 목표를 달성했는지 확인해보자. 모든 목표를 달성했다.

1. [x] 유저의 이름, 이메일, 휴대폰 번호 총 3개의 인풋 필드가 있어야 한다.
2. [x] 각 필드에 하나씩 총 3개의 validation 로직이 있어야 한다.
3. [x] 각 필드에 유저가 입력을 할 때마다 validation 로직을 통해 실시간 피드백을 준다.
4. [x] 모든 validation 로직이 모두 충족될 때만 Submit 버튼을 클릭할 수 있다.


## recap
이번 글에서 다룬 주요 포인트를 다시 한번 짚어보자.
1. 폼에서 유저 인풋을 받는 경우 client-side와 server-side 모두에서 인풋 validation은 필수이다.
2. 각 인풋 필드마다 onChange 이벤트를 만들어서 유저의 입력값을 받아 state에 저장한다.
3. state에 저장된 입력값을 validation 함수를 만들어서 검사한다.
4. validation 함수 결과에 따라 유저에게 실시간으로 피드백을 줄 수 있다.
5. 더 나아가 모든 인풋 필드의 validation이 통과되어야 폼 제출 버튼을 activate 할 수 있다.
이번 프로젝트의 전체 코드는 [Github](https://github.com/jd1386/react-form-validation)에서 확인할 수 있고 [라이브 데모](https://react-form-validation-6i6h1u8xu.now.sh/) 또한 확인할 수 있다.


## extend 할 만한 이슈
- 유저가 이메일을 입력할 때마다 서버에 중복 이메일이 없는 지 확인해서 피드백을 주는 것도 구현해보면 좋을 것 같다.

### React form 관련 다른 글
* [React에서 인풋 필드 validate하기]({% post_url  2019-01-26-React에서-인풋-필드-validate하기 %})
* [React form에서 이메일 중복 확인하기]({% post_url  2019-02-06-React-form에서-이메일-중복-확인하기 %})
* [React form에서 패스워드 일치 여부 확인하기]({% post_url  2019-02-07-React-form에서-패스워드-일치-여부-확인하기 %})