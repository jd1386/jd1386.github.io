---
layout: post
title: git hook을 이용한 Cloudflare purge cache
date: 2018-12-23 13:00:00 +0900
categories: blog
tags: 
---

Github Pages는 기본적으로 Let's Encrypt를 통한 https를 제공하고 CDN을 통해 컨텐츠를 제공한다. 하지만 나는 여기서 한발 더 나아가서 [Cloudflare](https://cloudflare.com)를 이용해서 블로그 로딩 속도를 좀 더 개선해보고 싶었다. Cloudflare는 html, css, javascript 파일들을 minify해서 컨텐츠를 제공하기 때문에 정적 사이트 같은 경우 속도개선 효과가 있다. 또한 전세계 각지의 edge 서버를 운영해서 전세계 어느 곳에서 사이트에 들어오더라도 꽤나 괜찮은 속도를 보장하는 것으로 알려져있다.

Github Pages와 Cloudflare를 연동하는 방법은 다른 글에서 다루도록 하고 이번 글에서는 Github에 블로그 소스 코드를 push 할 때 git hook을 이용해서 기존 Cloudflare의 cached content를 purge(삭제)하는 방법을 알아보도록 하겠다. 

Cached content를 purge해야 하는 이유는 Cloudflare에 cached content가 만료될 때까지 변경 내용이 보이지 않기 때문이다. Cloudflare 사이트에 들어가서 직접 purge하는 방법도 존재하지만 Cloudflare가 제공하는 API와 git hook을 이용하면 훨씬 더 편리하고 자동화된 방법으로 purge를 구현할 수 있다.

먼저 프로젝트 root directory에서 `ls -a`로 숨겨져있는 모든 폴더까지 리스트업 해보자. 그럼 다음과 같이 숨겨져있던 `.git` 폴더가 나타난다. 참고로 `.git`폴더는 `git init`을 실행할 때 만들어진다.

<img src="/assets/img/git-folder.png">

그 다음 `.git` 폴더로 이동 후 `hooks` 폴더에 들어가면 다음과 같은 샘플 파일들이 보인다.

<img src="/assets/img/git-hooks-content.png">

commit, push, receive 등 git 이벤트가 발생하면 그에 해당하는 파일이 실행될 것이다. 더 자세한 hook 관련 내용은 [githooks.com](https://githooks.com/)에서 확인할 수 있다.

우리는 `git push`이후 실행 될 hook을 만들 것이기 때문에 post-receive 파일을 만들어 내용을 채워넣으면 된다. 

`sudo nano post-receive`를 실행하면 빈 파일이 뜬다. (파일 확장자가 없다는 점에 주의) 그리고 아래 내용을 붙여넣고  \<EMAIL\>, \<API_KEY\>, \<ZONE_ID\>에 해당 내용을 넣고 저장한다.


```shell
export CLOUDFLARE_AUTH_EMAIL="<EMAIL>"
export CLOUDFLARE_API_KEY="<API_KEY>"
export CLOUDFLARE_ZONE_ID="<ZONE_ID>"

curl -X POST "https://api.cloudflare.com/client/v4/zones/${CLOUDFLARE_ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: ${CLOUDFLARE_AUTH_EMAIL}" \
  -H "X-Auth-Key: ${CLOUDFLARE_API_KEY}" \
  -H "Content-Type: application/json" \
  --data '{"purge_everything":true}'
  ```

  위 코드는 Cloudflare API를 이용해서 등록한 웹사이트의 cached content를 purge하라는 POST 명령어이다.

  이제 새로운 포스트를 작성하거나 수정한 후 git push해보자. 그리고 잠시 후 포스트를 브라우저에서 불러오면 컨텐츠가 Cloudflare에서 제공된다는 걸 확인할 수 있다.

  <img src="/assets/img/cloudflare-cached.png">

  curl을 이용해도 확인할 수 있다.
  `curl -svo /dev/null https://leejungdo.com`

  이제 git hook을 이용해 git push 이후 매번 자동으로 Cloudflare cached content가 purge 되고 새롭게 caching 될 것이다.

  Cloudflare API는 [Cloudflare API 공식 문서](https://api.cloudflare.com/)에서 확인할 수 있다.