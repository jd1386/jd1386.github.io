---
layout: post
title: react-on-screen을 이용한 컴포넌트 lazy load
date: 2018-12-22 13:00:00 +0900
categories: blog
tags: react
---

### The less, the better
웹사이트 퍼포먼스의 rule no.1이 있다면 뭘까. 아마도 DOM 갯수의 최소화일 것이다. 렌더링 해야하는 DOM node가 적을 수록 브라우저는 DOM tree 전후 비교도 훨씬 빨리할 것이고 그만큼 rendering과 painting에 소요되는 시간도 줄 것이다.

네이버 뉴스의 경우, 유저가 기사 제목을 클릭하면 기사 내용이 렌더된다. 유저는 기사 내용을 마우스로 스크롤 다운하면서 기사를 한줄 한줄 읽게 된다. 이 때까지만 해도 화면 아래에 있던 댓글 목록은 렌더 되지 않은 상태이다. 유저의 viewport가 댓글 목록 가까이 가면 그제서야 댓글이 렌더된다.

<img src="/assets/img/naver-news.gif">

화면 우측 스크롤 바를 보면 화면 상단에서 80%정도 내려간 지점에서 스크롤 바가 갑자기 위로 올라간다. 이건 viewport 아래에 새로운 컴포넌트가 생성되었다는 의미다.

viewport에 따른 컴포넌트 렌더링이라는 건 유저가 화면에서 컴포넌트를 볼 수 있는지, 즉 컴포넌트가 유저의 viewport 안에 위치 하는 지에 따라 컴포넌트를 렌더링한다는 것이다.

그럼 viewport에 따른 컴포넌트 렌더링의 장점은 무엇일까? 최초 화면을 렌더할 때 굉장히 빠를 수 있다. 네이버 뉴스에서 댓글 목록을 기사와 함께 동시에 렌더링하면 그 시간만큼 유저가 기다려야 하고 그만큼 웹사이트는 느리게 느껴질 것이다. 하지만 기사 내용과 댓글 목록 렌더링을 viewport로 구분해서 비동기로 렌더링한다면 유저는 댓글 목록이 렌더되지 않은 만큼 쾌적하게 느낄 것이다.

### react-on-screen 사용하기

유저의 viewport에 특정 컴포넌트의 존재하는지 여부를 쉽게 확인하도록 도와주는 [react-on-screen](https://github.com/fkhadra/react-on-screen)을 활용해서 컴포넌트 lazy load를 구현해보자. 먼저 아래 데모 사이트를 확인해보고 우리가 만들려고 하는 게 뭔지 한번 감을 잡아보자.

- [데모 사이트](https://leejungdo.com/react-on-screen-test/)
- [소스 코드](https://github.com/jd1386/react-on-screen-test)

컴포넌트 구조는 다음과 같다:
- App.js
  - Users.js
  - Posts.js
  - Comments.js

먼저 `create-react-app react-on-screen-test`를 실행해서 새로운 리액트 프로젝트를 만든다.
  
그 다음 `yarn add react-on-screen` 또는 `npm install react-on-screen`으로 react-on-screen을 설치한다.

`App.js`에서 다음과 같이 `TrackVisibility`를 불러와서 Users, Posts, Comments 세 개의 컴포넌트를 감싸준다. 
  - `offset`값으로 해당 컴포넌트의 최상단이 화면에 보이기 시작한 순간부터 얼만큼의 픽셀을 스크롤 다운해야 visible 할 것인지를 설정할 수 있다. 300을 설정했다면 컴포넌트의 최상단에서 300px 지점이 viewport에 들어오면 컴포넌트는 visible하다고 정의할 수 있다.
  - `once`는 컴포넌트가 단 한번이라도 visible해지면 event listener를 삭제해서 더이상의 viewport tracking을 하지 않는다는 것이다. 

#### App.js
```coffee
  import React, { Component } from 'react';
  import TrackVisibility from 'react-on-screen';
  import Users from './Users';
  import Posts from './Posts';
  import Comments from './Comments';

  class App extends Component {
    render() {
      return (
        <div className="App">
          <TrackVisibility offset={500} once>
            <Users />
          </TrackVisibility>
          <TrackVisibility offset={300} once>
            <Posts />
          </TrackVisibility>
          <TrackVisibility offset={450} once>
            <Comments />
          </TrackVisibility>
        </div>
      );
    }
  }

  export default App;
```

<br>

Users 컴포넌트는 10개의 유저 목록을 보여준다 가정하고 [jsonplaceholder](https://jsonplaceholder.typicode.com)를 이용해 10명의 유저 데이터를 axios로 불러온다.

#### Users.js
```coffee
import React, { Component } from 'react';
import axios from 'axios';

const USERS_URL = 'https://jsonplaceholder.typicode.com/users';

class Users extends Component {
  state = {
    users: []
  };

  _loadUsers() {
    if (!this.state.users.length) {
      axios.get(USERS_URL).then(res => {
        this.setState({ users: res.data });
      });
    }
  }

  render() {
    this.props.isVisible && this._loadUsers();

    return (
      <div className="section">
        <div>
          <h1>
            Users section {this.props.isVisible ? 'visible' : 'not visible yet'}{' '}
            && 10 users {this.state.users.length ? 'loaded' : 'not loaded yet'}
          </h1>
        </div>
        <div>
          {this.state.users.map((user, index) => {
            return (
              <ul key={index}>
                <li>User id: {user.id}</li>
                <li>Name: {user.name}</li>
              </ul>
            );
          })}
        </div>
      </div>
    );
  }
}

export default Users;
```

<br>

Posts 컴포넌트는 50개의 포스트 목록을 보여준다 가정하고 [jsonplaceholder](https://jsonplaceholder.typicode.com)를 이용해 50개의 포스트 데이터를 axios로 불러온다.

#### Posts.js
```coffee
import React, { Component } from 'react';
import axios from 'axios';

const POSTS_URL = 'https://jsonplaceholder.typicode.com/posts';

class Posts extends Component {
  state = {
    posts: []
  };

  _loadPosts() {
    if (!this.state.posts.length) {
      axios.get(POSTS_URL).then(res => {
        this.setState({ posts: res.data.slice(0, 50) });
      });
    }
  }

  render() {
    this.props.isVisible && this._loadPosts();

    return (
      <div className="section">
        <div>
          <h1>
            Posts section {this.props.isVisible ? 'visible' : 'not visible yet'}{' '}
            && 50 posts {this.state.posts.length ? 'loaded' : 'not loaded yet'}
          </h1>
        </div>
        <div>
          {this.state.posts.map((post, index) => {
            return (
              <ul key={index}>
                <li>Post id: {post.id}</li>
                <li>Title: {post.title}</li>
                <li>Body: {post.body}</li>
              </ul>
            );
          })}
        </div>
      </div>
    );
  }
}

export default Posts;
```

<br>

Comments 컴포넌트는 500개의 댓글 목록을 보여준다 가정하고 [jsonplaceholder](https://jsonplaceholder.typicode.com)를 이용해 500개의 댓글 데이터를 axios로 불러온다.

#### Comments.js

```coffee
import React, { Component } from 'react';
import axios from 'axios';

const COMMENTS_URL = 'https://jsonplaceholder.typicode.com/comments';

class Comments extends Component {
  state = {
    comments: []
  };

  _loadComments() {
    if (!this.state.comments.length) {
      axios.get(COMMENTS_URL).then(res => {
        this.setState({ comments: res.data });
      });
    }
  }

  render() {
    this.props.isVisible && this._loadComments();

    return (
      <div className="section">
        <div>
          <h1>
            Comments section{' '}
            {this.props.isVisible ? 'visible' : 'not visible yet'} && 500
            comments {this.state.comments.length ? 'loaded' : 'not loaded yet'}
          </h1>
        </div>
        <div>
          {this.state.comments.map((comment, index) => {
            return (
              <ul key={index}>
                <li>Comment id: {comment.id}</li>
                <li>Name: {comment.name}</li>
                <li>Email: {comment.email}</li>
                <li>Body: {comment.body}</li>
              </ul>
            );
          })}
        </div>
      </div>
    );
  }
}

export default Comments;
```

### Recap
기본적인 구현은 끝났다. 다시 정리하자면 [데모 사이트](https://leejungdo.com/react-on-screen-test/)는 총 3개의 컴포넌트를 사용하고 각각 정해진 갯수의 오브젝트를 API로 불러와서 렌더할 것이다.
- Users 10명
- Posts 50개
- Comments 500개

여기서 핵심은 각 컴포넌트가 유저의 viewport에 있는 지 없는 지를 `react-on-screen`을 활용해서, viewport 안에 있다면 API call해서 해당 리스트를 렌더할 것이고 viewport에 밖에 있다면 아무 것도 하지 않는다는 것이다. 

<img src="/assets/img/react-on-screen-example.gif">

User 10명, Post 50개, Comments 500개를 모두 한번에 불러오고 렌더해줘야 한다면 어떨까? 10 + 50 + 500, 총 560번의 API call이 있고 나서야 렌더링이 된다는 것인데, 이건 유저가 감내하기에는 상당히 버거운 시간이다. 그리고 유저를 차치하더라도 엔지니어링적으로도 굳이 560번의 API call이 있고나서 화면에 렌더된다는 건 합리적이지 않다.

### 배운 점
라이브러리를 사용하지 않고 vanilla javascript로 구현할 수 있다는 점은 분명하지만 리액트 프로젝트를 위해 이미 잘 만들어진 `react-on-screen`을 써보는 것도 꽤 괜찮은 옵션같다. 실제 프로젝트에서도 써봐야겠지만 데모 사이트에서 보더라도 사용성이나 속도면에서 크게 떨어지는 것 같지 않아서 추천하고 싶다.