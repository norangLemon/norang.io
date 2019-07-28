---
title: nginx로 shorten link 만들기
date: 2019-07-28 11:50:01
categories: server
tags:
- server
- nginx
---

가끔씩 굉장히 지저분하고 긴 링크를 다른 사람들에게 전해주어야 할 때가 있다.
그럴 때를 위해서 [bit.ly]같은 서비스가 있지만, 일정 갯수 이상으로는 돈을 내야 한다.
물론 이런 shorten link를 생성해주는 서비스가 여러 개 있겠지만, 나로서는 매번 다른 서비스를 찾아서 링크를 만드는 게 귀찮게 느껴졌다.
나에게 서버가 있고, 마침 웹서버도 돌리고 있으니 내가 직접 shorten link를 만들었다.

그 [결과 코드]에 대해서 자세히 풀어 써보려고 한다.
방법을 빠르게 확인하고 싶다면 [결과 코드]에서 `sites-available/link.norang.io`, `redirects-map.conf`를 중심으로 확인하면 된다.

<!--more-->

### Backgrounds

[Nginx]는 유명한 웹서버 중 하나로, 소개 페이지에 따르면:

> nginx [engine x] is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev. For a long time, it has been running on many heavily loaded Russian sites including Yandex, Mail.Ru, VK, and Rambler. According to Netcraft, nginx served or proxied 26.34% busiest sites in June 2019. Here are some of the success stories: Dropbox, Netflix, Wordpress.com, FastMail.FM.

설치는 [공식 튜토리얼](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)을 따라서 금방 할 수 있다.

이 블로그 또한 [Nginx]를 통해 서비스되고 있다.
`/etc/nginx/`에서 세부적인 설정이 가능한데(OS에 따라 경로가 다를 수 있음),
내가 설정한 부분들은 따로 [nginxConf] repository를 만들어서, 해당 경로에 내 파일의 symlink를 만들고 있다.
내 [Nginx] 설정의 주요 뼈대는 아래와 같다.

> sites-available/norang.io
> nginx.conf
> ssl.conf

`nginx.conf`에는 기본 설정이, `sites-available/norang.io`에 블로그 관련 설정이 들어있다.
`nginx.conf`에 선언되지 않고, 다른 파일에 선언된 설정들은 간단하게 `include`해서 사용할 수 있다.
따라서 보기 좋게 기본 설정과 사이트별 설정을 분리했다.
이렇게 `sites-available`에 분리된 설정은 바로 include해서 사용하지 않고,
사용하려고 하는 경우에만 `sites-enabled`에 복사해서 사용한다.


내가 목표로 하는 것은 이런 것이었다:
- `link.norang.io` 도메인을 사용
- 여러 개의 shorten link에서 각각의 target link로 리다이렉트

따라서 우선 해당 도메인에 대한 서버 규칙을 만든 뒤, 매핑을 만들고, 리다이렉트하도록 했다.

### 서버 규칙 만들기

이를 위해서 우선 `link.norang.io`라는 사이트 규칙을 만들었다.
`link.norang.io`에 80번 포트로 요청이 들어올 경우에 대한 규칙이라는 뜻이다.
물론 먼저 웹서버의 IP로 `link.norang.io`를 호스팅하는 걸 잊지 말자.

```
# filename: link.norang.io

server {
  listen 80;                  # 80번 포트로 들어오는 요청
  server_name link.norang.io; # 도메인이 link.norang.io인 요청
}
```

### target link와 shorten link 매핑하기

여기에서 나는,`link.norang.io/some_link`나 `link.norang.io/other_link` 같은 shorten link들을 어떤 다른 링크로 매핑하고 싶다.
다행히도 [Nginx]에서는 [map] 기능을 지원한다!
[map]은 다음과 같이 `$key`라는 변수를 받아 `$value`라는 결과값을 매핑해준다.
아래 코드에서 `$key` 값이 `a`라면, `$value` 값은 1일 것이다.

```
map $key $value {
  a 1;
  b 2;
  default 0;
}
```

[map] 규칙은 위처럼 늘여 쓸수도 있지만, 다른 파일에 분리해 둔 뒤 받아올 수도 있다.
가령 위의 규칙을 `map-rule`이라는 파일에 아래와 같이 분리하면:
```
a 1;
b 2;
```

다음과 같이 파일의 상대 `include`할 수 있다:
```
map $key $value {
  include PATH/TO/map-rule; # map-rule 파일의 절대 경로
  default 0;
}
```

이런 방법으로 `link.norang.io/some_link`, `link.norang.io/other_link`에서 target link를 매핑하려면 `$uri`라는 변수가 필요하다.
[Nginx]에는 다양한 [variables]가 있는데, `$uri`는 request의 URI를 정규화한 것을 의미한다.

여기서의 정규화란, 어떤 URI에 `#paragraph`, `?reqeust=something` 같은 요청을 제외하고 대문자를 소문자로 통합시키는 등의 작업을 일컫는다.
따라서, 도메인 하위의 URI에서 대소문자가 중요한 경우는 정규화를 하지 않는 `$requested_uri`를 사용해야 한다.
나의 경우 특별히 구분하고 싶지는 않았기에 그냥 `$uri`를 사용했다.

URI는 `link.norang.io`의 하위에 있는 부분을 가리킨다.
즉 `/some_link`, `/other_link`가 이에 해당된다.
따라서 이 변수가`$key`가 될 것이고, target link를 그 `$value`로 두면 된다:


```
# link.norang.io
map $uri $redirect_dest {
  include PATH/TO/redirects-map.conf; # map file을 포함
  default https://norang.io;  # 기본적으로는 블로그로 매핑
}

server {
  listen 80;
  server_name link.norang.io;
}
```

```
# redirects-map.conf
/somelink https://some_target_link;
/otherlink https://other_target_link;
```

`link.norang.io` 파일에서 `$uri`에 대한 매핑 규칙을 만들었다.
이 때, 규칙을 만족하지 못하는 요청, 가령 `link.norang.io/wrong_link`가 들어온다면 `https://norang.io`로 매핑해준다.
어떤 다른 페이지를 만들어서, 잘못된 요청임을 보여줄 수도 있을 것이다.

### 리다이렉트 해주기
이제 여러 개의 shorten link에 대한 target link를 매핑하는 것 까지 했으니, 리다이렉트를 하면 된다.
target link가 정해져 있는 경우, 리다이렉트는 정말 간단하다.
[다양한 redirection 방법]이 있지만, 이 경우에는 `return`을 사용하려고 한다.

`server` 블록에 다음 한 줄을 추가하는 것으로 target link로 이동할 수 있다.

```
return CODE url;
```
여기에서 CODE는 [HTTP Status Code]를 의미한다.

리디렉션 코드에도 여러 가지가 존재하지만, 이 경우에는 301 코드를 사용한다.
301 코드는 영구히 target link로 이동한다는 뜻이다.
주로 임시로 이동한다는 뜻의 302 코드와 비교된다.
302 코드의 경우, 다른 페이지에 리다이렉트 하고 있지만 리다이렉트가 임시 조치임을 의미한다.
즉 다시 이 링크에서 무엇인가를 보여줄 것이라고 생각할 수 있다.
shorten link의 경우, 그저 다른 페이지로 리다이렉트할 뿐이므로 301코드를 사용한다.

따라서 `link.norang.io`파일을 다음과 같이 완성할 수 있다.

```
# link.norang.io
map $uri $redirect_dest {
  include PATH/TO/redirects-map.conf; # map file을 포함
  default https://norang.io;  # 기본적으로는 블로그로 매핑
}

server {
  listen 80;
  server_name link.norang.io;
  return 301 $redirect_dest;  # $redirect_dest로 리다이렉트
  }
```

유의해야 하는 점은, 같은 `link.norang.io` 하위의 url으로 `default`를 설정하려면 예외 처리를 잘 해야 한다는 것이다.
이 코드에서는 `link.norang.io`로 들어온 요청은 반드시 리디렉션하고 있기 때문이다.

가령 `link.norang.io/warning`에서 잘못된 url임을 경고하기 위해서  `default link.norang.io/warning;`으로 `default` 값을 바꿀 수 있다.
이 default page에 대해서는 리다이렉트를 하지 않도록 해야 한다.
`return` 구문을 if나 `location` block으로 감싸서 예외 처리를 할 수 있을 것이다.

### Miscellaneous

#### tailing slash 처리해주기

주소창에 `google.com/`, `google.com` 중 어느 것을 입력해도 구글 페이지로 이동한다.
즉 마지막 슬래시는 알아서 잘 처리된다.
하지만 위에서 작성한 매핑 규칙에는 슬래시가 있을 경우 작동하지 않는다!
이는 [정규식]으로 쉽게 처리할 수 있다.
([정규식]에 대해서 알지 못하는 경우, 공부해보기를 강력히 추천한다!)

[정규식]에서 `?`를 붙일 경우, 물음표 앞에 있는 문자가 있거나 없는 경우를 포괄한다.
가령 `ab?c`라는 정규식 표현은 `abc` 및 `ac`를 포괄한다.
따라서 `some_link/?`라는 표현으로 `some_link` 및 `some_link/`를 포괄할 수 있다.

이 때, 어떤 문자열이 정규식으로 표현된 것인지를 구분하기 위해서 [Nginx]에서는 맨 앞에 `~`를 붙이도록 하고 있다.
`~`를 붙인 경우 정규식 표현이고, 없는 경우는 그대로의 문자열을 일컫는다.

따라서 위의 매핑 파일을 다음과 같이 수정할 수 있다:

```
# redirects-map.conf
~/somelink/? https://some_target_link;
~/otherlink/? https://other_target_link;
```

#### 동작 확인하기

[Nginx] 설정을 바꾼 뒤, `systemctl` 혹은 `service` 등의 서비스 매니저로 `reload` 혹은 `restart`를 해야 한다.
그러면 새로운 설정에 맞춰서 [Nginx]가 동작하게 된다.
하지만 이를 확인하기 위해서 웹 브라우저에서 `link.norang.io/some_link`를 입력한다면, 이상하게 동작할 수도 있다.
이 경우, 직전의 라우팅 결과를 브라우저에서 캐싱하고 있기 때문일 수 있다.
크롬의 경우, 개발자 모드를 켠 후 Network 탭에서 Disable cache를 체크해두면 캐시 값을 사용하지 않도록 할 수 있다.

[bit.ly]: https://bitly.com/
[결과 코드]: https://github.com/norangLemon/nginxConf/tree/6c41ed142d1204e864c224d1c4172b87e45d729e
[Nginx]: https://www.nginx.com/
[NginxConf]: https://github.com/norangLemon/nginxConf
[map]: http://nginx.org/en/docs/http/ngx_http_map_module.html
[variables]: http://nginx.org/en/docs/varindex.html
[HTTP Status Code]: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes
[다양한 redirection 방법]: https://www.nginx.com/blog/creating-nginx-rewrite-rules/
[정규식]: https://ko.wikipedia.org/wiki/정규_표현식
