---
layout: post
title: react-on-screen을 이용한 컴포넌트 애니메이션
date: 2018-12-25 13:00:00 +0900
categories: blog
tags: react
---

지난 글 ['react-on-screen을 이용한 컴포넌트 lazy load 전략']({% post_url 2018-12-22-react-on-screen을-이용한-컴포넌트-lazy-load-전략 %})에서 우리는 viewport에 따른 컴포넌트 lazy load를 구현한 바 있다. 유저의 viewport 안에 컴포넌트가 위치하게 되면 해당 컴포넌트를 렌더링해서 퍼포먼스 개선에 상당한 효과를 볼 수 있었다. 

이번 글에서는 동일한 react-on-screen 모듈과 애니메이션 효과에 필요한 react-spring을 이용해서 viewport 안에 컴포넌트가 들어오면 해당 컴포넌트의 내용에 애니메이션을 구현해보자. react-on-screen 모듈 사용법은 [지난 글]({% post_url 2018-12-22-react-on-screen을-이용한-컴포넌트-lazy-load-전략 %})에 나와있으니 이번 글에서는 애니메이션 구현에 집중해보겠다. 

- [데모 사이트](https://leejungdo.com/react-on-screen-animation){:target="_blank"}
- [소스 코드](https://github.com/jd1386/react-on-screen-animation){:target="_blank"}
- [React on Screen](https://github.com/fkhadra/react-on-screen){:target="_blank"}
- [React Spring](https://react-spring.surge.sh/){:target="_blank"}


React에서 애니메이션을 구현하기 위해 여러가지 유명한 라이브러리가 있다. 그 중 우리는 [React Spring](https://react-spring.surge.sh/){:target="_blank"}을 사용해보자.

다양한 종류의 애니메이션 효과가 있지만 대체적으로 사용법은 단순하다. `from`과 `to`에 애니메이션 효과를 넣고자 하는 타겟을 각각 넣어주면 된다. 아래 예시는 0부터 1까지 숫자가 커지는 애니메이션 효과가 생길 것이다.
{% raw %}
```jsx
import { Spring } from 'react-spring';

<Spring
  from={{ number: 0 }}
  to={{ number: 1 }}
>
  {props => <div>{props.number}</div>}
</Spring>
```
{% endraw %}

<img src="/assets/img/spring-animation.gif">

그럼 이제 [데모 사이트](https://leejungdo.com/react-on-screen-animation){:target="_blank"}에서 구현한 viewport별 컴포넌트 애니메이션을 만들어보자.

### 데모 사이트

먼저 우리의 데모 사이트는 다음과 같은 컴포넌트 구조를 가질 것이다.
- App.js
  - FirstSegment.js
  - SecondSegment.js
  - ThirdSegment.js

#### App.js에서 각각의 하위 컴포넌트들을 \<TrackVisibility\>로 감싸준다.
```jsx
import React, { Component } from 'react';
import './App.css';
import TrackVisibility from 'react-on-screen';
import FirstSegment from './FirstSegment';
import SecondSegment from './SecondSegment';
import ThirdSegment from './ThirdSegment';

export default class App extends Component {
  render() {
    return (
      <div className="App">
        <TrackVisibility offset={300}>
          <FirstSegment />
        </TrackVisibility>
        <TrackVisibility offset={300}>
          <SecondSegment />
        </TrackVisibility>
        <TrackVisibility offset={450}>
          <ThirdSegment />
        </TrackVisibility>
      </div>
    );
  }
}
```

App.js에서 \<TrackVisibility\>가 하위 컴포넌트에게 isVisible props을 넘겨준다. 이를 이용해 애니메이션 여부를 구분짓는다. isVisible이 true가 되면, 즉 FirstSegment 컴포넌트가 유저의 viewport 안에 위치하게 되면, Spring이 0부터 4902941까지 숫자를 증가시키는 애니메이션을 실행하게 된다. 

#### FirstSegment.js
{% raw %}
```jsx
import React, { Component } from 'react';
import { Spring, config } from 'react-spring';

export default class FirstSegment extends Component {
  render() {
    const style = this.props.isVisible
      ? { background: '#fff' }
      : { background: '#ddd' };

    return (
      <div className="section" style={style}>
        <h3>First segment {this.props.isVisible && 'is now visible'}</h3>
        {this.props.isVisible && (
          <div style={{ fontSize: '3em' }}>
            <span style={{ marginRight: '5px' }}>Users</span>
            <span>
              <Spring
                from={{ number: 0 }}
                to={{ number: 4902941 }}
                delay={200}
                config={config.slow}
              >
                {springProps => {
                  return Math.round(springProps.number);
                }}
              </Spring>
            </span>
          </div>
        )}
      </div>
    );
  }
}
```
{% endraw %}

마찬가지로 두번째 컴포넌트인 SecondSegment는 0부터 593049402000까지 숫자가 증가하는 애니메이션이 구현될 것이다.

#### SecondSegment.js
{% raw %}
```jsx
import React, { Component } from 'react';
import { Spring, config } from 'react-spring';

export default class SecondSegment extends Component {
  render() {
    const style = this.props.isVisible
      ? { background: '#fff' }
      : { background: '#ddd' };

    return (
      <div className="section" style={style}>
        <h3>Second segment {this.props.isVisible && 'is now visible'}</h3>
        {this.props.isVisible && (
          <div style={{ fontSize: '3em' }}>
            <span>
              Revenue $
              <Spring
                from={{ number: 0 }}
                to={{ number: 593049402000 }}
                delay={200}
                config={config.slow}
              >
                {springProps => {
                  return Math.round(springProps.number);
                }}
              </Spring>
            </span>
          </div>
        )}
      </div>
    );
  }
}
```
{% endraw %}

세번째 컴포넌트 ThirdSegment에서는 72라는 숫자가 0부터 증가할 것이고 순차적으로 Korea, China, Japan, Taiwan, Hong Kong, and many others가 fade in 할 것이다. 순차적으로 fade in 할 수 있는 이유는 각기 갖고 있는 Spring props의 delay값이 200, 400 등으로 나뉘어져 있기 때문이다.

#### ThirdSegment.js
{% raw %}
```jsx
import React, { Component } from 'react';
import { Spring, config } from 'react-spring';

export default class ThirdSegment extends Component {
  render() {
    const style = this.props.isVisible
      ? { background: '#fff' }
      : { background: '#ddd' };

    return (
      <div className="section" style={style}>
        <h3>Third segment {this.props.isVisible && 'is now visible'}</h3>
        {this.props.isVisible && (
          <div style={{ fontSize: '3em' }}>
            <div>
              Popular in{' '}
              <Spring
                from={{ number: 0 }}
                to={{ number: 72 }}
                delay={100}
                config={config.molasses}
              >
                {springProps => {
                  return Math.round(springProps.number);
                }}
              </Spring>{' '}
              <span>Countries</span>
            </div>
            <div>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={200}
                config={config.molasses}
              >
                {props => <div style={props}>Korea</div>}
              </Spring>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={400}
                config={config.molasses}
              >
                {props => <div style={props}>China</div>}
              </Spring>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={600}
                config={config.molasses}
              >
                {props => <div style={props}>Japan</div>}
              </Spring>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={800}
                config={config.molasses}
              >
                {props => <div style={props}>Taiwan</div>}
              </Spring>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={1000}
                config={config.molasses}
              >
                {props => <div style={props}>Hong Kong</div>}
              </Spring>
              <Spring
                from={{ opacity: 0 }}
                to={{ opacity: 1 }}
                delay={1200}
                config={config.molasses}
              >
                {props => <div style={props}>and many others</div>}
              </Spring>
            </div>
          </div>
        )}
      </div>
    );
  }
}

```
{% endraw %}

#### 완성된 데모 사이트
<img src="/assets/img/react-on-screen-animation-demo.gif" width="70%">

### 결론
`react-on-screen`을 이용한 컴포넌트 viewport tracking은 재밌는 개념이기도 하고 쓰임새도 다양할 것 같아서 앞으로 다양하게 활용해보고 싶다. [데모 사이트](https://leejungdo.com/react-on-screen-animation){:target="_blank"}에서 viewport별 컴포넌트 애니메이션을 다시 한번 살펴보고 프로젝트에 활용해보시기 바란다.
