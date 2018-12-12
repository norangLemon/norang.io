---
title: 'grep, sed, xargs'
date: 2017-04-05 17:55:24
categories: server
tags:
- server
- linux
- grep
- sed
- pipe
- xargs
---
바쿠스 작업을 할 때나, 내 서버 관리를 할 때 자주 문자열 치환 작업이 필요하다.
이번에 바쿠스 일을 하면서 문서를 남겼는데,
내 홈페이지에도 같이 남겨본다.

<!-- more -->
## 한 PC의 파일(들)을 수정하는 경우
이 경우, [sed](https://www.gnu.org/software/sed/manual/sed.html)를 사용하여 쉽게 작업을 수행할 수 있다.
이 문서에서는 GNU regex engine을 사용하는 경우를 기준으로 서술하였다.

OS X의 경우 기본 `sed`가 gnu sed가 아니다.
OS X에서는 `man sed`를 참조해서 작업하거나, `brew`를 통해서 `gnu-sed`를 따로 설치하도록 한다.
이때 `gsed`로 접근하여야 함에 유의한다.

다음과 같은 명령어로 `[filename]`의 `[string]`을 `[replacement]`로 변경한다.

```
sed -i -r 's/[string]/[replacement]/g' [filename]
```

* `-i`: 결과를 출력하는 대신, 파일을 직접 바꾼다.
sed 명령어가 잘 동작하는지 확인하려면 `-i`옵션을 제외하고 결과를 출력해본다.
* `-r`: Extended (ERE) Syntax를 사용하도록 한다. 
ERE에서는 괄호, `?`, `+` 등을 정규식 기호로 사용할 때 escaping 하지 않는다.
가령 `a?`는 빈 스트링 혹은 `a`를 의미할 것이다.
* `s`: substitute. 단어를 대체한다.
`s`뒤에 tokenizing unit을 입력할 수도 있다.
단어를 삭제하는 `d` 등등의 옵션도 있다.
* `[string]`: 바꿀 단어를 입력한다. 정규식을 사용할 수 있다.
* `[replacement]`: 대체할 단어를 입력한다.
* `g`: 한 토큰(기본적으로 한 줄) 안에 정규식 매칭이 여러번 될 경우, 모두 대체하도록 한다.

이렇게 한 개의 파일에서 여러 문자열을 대체하게 할 수 있다.

### 여러 파일에서 수정
만약 여러 개의 파일에서 수정해야 하는 경우,
`find`, `grep`, [`ag`](https://github.com/ggreer/the_silver_searcher),
[`rg`](https://github.com/BurntSushi/ripgrep) 등을 사용하여 다음과 같이 파일명을 넘기게 할 수 있다.

특히 어떠한 문자열을 포함하고 있는 파일을 모두 찾은 후,
그 파일에서 문자열을 수정해야 하는 경우에 `grep`, `ag`, `rg` 등이 유용하다.

### grep
아래는 대부분 기본적으로 설치되어있는 검색 프로그램인 `grep`을 사용한 예제이다.

```
grep -rlE "[search string]" * | xargs sed -i -r 's/[string]/[replacement]/g'
```

* `grep -rlE "[search string]" *`: 현재 위치에서 `"[search string]"`을 포함하고 있는 모든 파일을 그 이름만 출력해준다.
    * `-r`: 지정한 경로에 있는 모든 하위 디렉토리까지 반복적으로 검색을 수행한다.
    * `-l`: 매칭된 문자열의 내용은 생략하고, 파일의 이름만 출력한다.
    * `-E`: ERE를 사용하게끔 한다. 정규식 사용 시 괄호 등의 escape가 필요 없다.
    정규식을 사용하지 않는다면 이 옵션을 제외해도 괜찮다.
* `|`: [pipe](http://ryanstutorials.net/linuxtutorial/piping.php#piping)라고도 불린다.
`|` 앞의 명령어에 의해 STDOUT으로 입력된 문자열을, 뒤에 있는 명령어에게 전달한다. 
* `xargs`: pipe(`|`)로 넘어온 인자를 line by line으로 쪼개어 다음 명령어에게 전달해준다.
이 경우, `[search string]`이 포함된 문자열이 여러 개일 수 있기 때문에,
바로 pipe를 통해 넘겨줄 수 없기 때문이다.

### ag
아래는 `ag`를 사용한 예제다.

```
ag [search string] -l | xargs sed -i -r 's/[string]/[replacement]/g'
```

* `ag [search string] -l`: `[search string]`을 포함하고 있는 모든 파일을 그 이름만(`-l`) 출력해준다.


