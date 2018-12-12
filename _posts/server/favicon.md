---
title: favicon 바꾸기
date: 2018-12-12 13:24:15
categories: server
tags:
- server
- favicon
- hexo
---

블로그 등 웹사이트에서 주소창 혹은 페이지 제목 왼쪽에 작게 표시되는 그림을 favicon이라고 한다.
내가 사용하고 있는 hexo [iissnan/next]에서도 파비콘을 자동으로 설정해준다.
[theme-next/hexo-theme-next]에서는 크기별로 이미지를 넣어서 설정하는 기능이 있는데,
버그인건지 몇몇 기능이 제대로 동작하지 않아 옛날 버전이라고 할 수 있는 [iissnan/next]를 사용하고 있다.

<!-- more -->
[iissnan/next]에서는 이미지를 업로드 한 뒤 개별 설정하는 게 불가능해서,
기존에 있던 파비콘 이미지를 백업한 뒤 [flaticon]이라고 하는 프리 아이콘 사이트에서 아이콘을 받았다.
[freepik]이라고 하는 프리 이미지 사이트 운영사에서 함께 운영하는 듯하다.

![bigger](https://norang.io/images/favicon-32x32-next.png)

32픽셀, 16픽셀의 이미지가 각각 쓰이고 있기 때문에, 각각을 [flaticon]에서 받아준다.
이 이미지는 테마가 있는 디렉토리 하위의 `source/images`로 옮겨준다.
이 디렉토리에는 `favicon-32-32-next.png`와 같은 파일이 있는데, 각각 크기에 맞춰 이 이름으로 바꾸어준다.
설정 후 블로그를 갱신하면 파비콘이 잘 반영된 것을 확인할 수 있다.

한편 [flaticon]의 무료 이용자는 규약상 이미지가 쓰이는 모든 페이지에 아래와 같은 코드를 통해 출처와 저작권을 명기해야 한다.

```HTML
<div>Icons made by <a href="https://www.flaticon.com/authors/srip" title="srip">srip</a> from <a href="https://www.flaticon.com/"           title="Flaticon">www.flaticon.com</a> is licensed by <a href="http://creativecommons.org/licenses/by/3.0/"          title="Creative Commons BY 3.0" target="_blank">CC 3.0 BY</a></div>
```

이것은 간단하게 footer에 항상 표시되게 함으로써 해결했다.
footer에 customed message는 [theme의 config]에 항목이 있어서 쉽게 추가할 수 있다.
블로그의 모든 페이지 최하단에서 저작권에 대한 부분을 확인할 수 있다.

[iissnan/next]: https://github.com/iissnan/hexo-theme-next
[theme-next/hexo-theme-next]: https://github.com/theme-next/hexo-theme-next
[freepik]: https://www.freepik.com/
[flaticon]: https://www.flaticon.com/
[theme의 config]: https://github.com/iissnan/hexo-theme-next/blob/master/_config.yml#L66
