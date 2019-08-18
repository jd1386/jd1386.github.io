---
layout: post
title: git hook으로 binding.pry 검사하기
date: 2019-08-17 00:00:00 +0900
categories: blog
tags: ruby
---

레일즈나 루비 프로젝트를 진행하면서 코드를 디버깅할 때는 보통 `binding.pry`를 사용한다. `binding.pry`를 코드 중간에 삽입후 파일을 실행하면 `binding.pry`가 위치한 코드 라인에서 프로그램 실행을 일시적으로 멈추고 콜스택과 실행 컨텍스트를 확인할 수 있어서 디버깅시 매우 유용하다. ([pry 공식문서](https://github.com/pry/pry#runtime-invocation))

조심해야할 건 프로그램 내에 `binding.pry` 를 실행한 채로 코드를 배포해버리면 실서버에서도 똑같이 프로그램이 해당 라인에서 멈추게 되어 치명적인 문제를 야기한다.

부끄럽지만 나도 최근 4개월간 이런 실수를 정확히 2번했다. 다행히 실서버의 어드민 기능에 코드가 배포되었기 망정이지 유저가 사용하는 코드였다면 유저는 영문도 모르고 아무것도 실행되지 않는 화면을 멍하니 바라볼 수 밖에 없었을 거다. 

그래서 내 나름대로 대책을 마련하기 위해서 `git hooks`에 작업한 파일 내에서 binding.pry를 scan하는 기능이 있으면 좋겠다고 생각했고, 찾아보니 역시나 이미 스크립트가 있었다. 
([참조](https://gist.github.com/wacko/62560b45c1d191859d6b)) 

이번 글에서는 루비를 이용해서 git hooks에 binding.pry를 스캔하는 스크립트를 추가해보자. 루비가 설치되어 있지 않다면 [bash 스크립트](https://gist.github.com/guilherme/9604324) 또는 [node 스크립트](https://medium.com/@Sergeon/using-javascript-in-your-git-hooks-f0ce09477334)를 이용해도 되겠다. 

(git hooks에 대해 더 알고 싶다면 [이곳](https://githooks.com/)을 확인하면 좋다.)

### 1. pre-commit 스크립트 파일 생성

git init되어 있는 프로젝트 폴더로 이동 후 `touch .git/hooks/pre-commit` 를 이용해 파일을 생성해준다. 에디터로 `pre-commit` 파일을 열어 아래 내용을 붙여넣고 저장한다.

```ruby
#!/usr/bin/env ruby
    
# Validates that you don't commit forbidden keywords to the repo
# You can skip this checking with 'git commit --no-verify'

exit 0 if ARGV.include?('--no-verify')
# Update this list with your own forbidden keywords
KEYWORDS = %w(binding.pry console.log debugger)

def red(text)    "\033[31m#{text}\033[0m"; end
def yellow(text) "\033[33m#{text}\033[0m"; end

# list all the files staged for commit
files_changed = %x(git diff --cached --name-only --).split

# search files for keywords
%x(git grep -q -E "#{KEYWORDS.join('|')}" #{files_changed.join(' ')})
if $?.exitstatus.zero?
  puts "# Check following lines:"
  files_changed.each do |file|
    KEYWORDS.each do |keyword|
      %x(git grep -q #{keyword} #{file})
      if $?.exitstatus.zero?
        line = %x(git grep -n #{keyword} #{file} | awk -F ":" '{print $2}').split.join(', ')
        puts "#\t#{red(file)}:#{line} contains #{yellow keyword}."
      end
    end
  end
  puts "Use 'git commit --no-verify' to skip this validation"
  exit 1
end
```

위 스크립트 내용은 보기보다 단순한데 요약하자면, 스테이지 되었고 (staged) 작업으로 인해 내용이 변경된 (modified) 파일 중에 'binding.pry', 'console.log', 'debugger'라는 내용을 검색한 후, 있으면 `"Use 'git commit --no-verify' to skip this validation"` 라는 메세지를 내고 1번 코드 (에러)를 반환한다. 에러가 반환되었기 때문에 commit이 실행되지 않는다. commit이 되지 않았기 때문에, 당연하게도 remote repository에 push될 수 없고 remote server에도 반영되지 않는 것이다.

### 2. pre-commit 파일 퍼미션 설정

위 스크립트 파일을 생성한 후 코드에서 binding.pry나 console.log 등을 찍고 커밋하려고 하면 기대와는 달리 pre-commit 파일이 실행가능(executable) 하지 않아서 내용이 무시되었다고 하면서 그대로 커밋이 되어 버릴 것이다. 이런 경우 다음과 같이 실행가능하게 만들어준다.

`cd .git/hooks && chmod +x pre-commit`

 
간단하게 위 두단계만 거치면 이제 binding.pry가 코드에 실수로 배포되는 일은 없을 것이다. 루비를 사용하든 노드를 사용하든 git hooks는 사용할 수 있기 때문에 필요에 따라 다르게 적용할 수도 있을 것이다.

<img src="/assets/img/2019-08-18-01.png">
위와 같이 코드에 console.log가 있어서 커밋할 수 없다고 에러를 반환한다.