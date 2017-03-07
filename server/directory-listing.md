---
title: directory listing
date: 2017-03-03 20:34:19
tags: 
- server
- nginx
categories: server
---
사람들이 자신의 서버에 어떤 파일을 올려놓고, 여기서 이 파일을 받아가라고 할 때가 있다.
혹은 디렉토리 안에 들어있는 파일의 리스트를 보여줄 수도 있다.

<!-- more -->

### 파일 다운로드 가능하게 하기
서버의 `/download/file`을 `download.norang.io/file`에서 다운로드하도록 해보자.
해당 파일을 웹에서 접근한 후 읽을 수 있어야 하므로, 공개된 read 권한을 추가한다.
가령 644 권한이라면 누구든 해당 파일을 읽을 수 있을 것이다.
`chmod 644 file`

그 후, `download.norang.io/file`에서 파일에 접근할 수 있도록 nginx에 설정해준다.
설정 이후에 nginx를 reload하는 것을 잊지 말자.
```nginx
server {
    listen 80;
    server_name download.norang.io;
    root /download;
}
```
해당 링크에 들어가면,  `/download` 폴더 안에 있는 `file`을 다운로드 할 수 있다.

### 디렉토리의 리스트를 확인할 수 있게 하기
해당 디렉토리에 무엇이 있는지 확인하고, 그것을 다운로드 가능하게 하고 싶을 수 있다.
이 경우, 위의 nginx 설정에 대하여 `autoindex on`을 추가한다.

```nginx
server {
    listen 80;
    server_name download.norang.io;
    root /download;
    autoindex on;
}
```

`download.norang.io`에서 해당 폴더 안에 있는 파일 리스트를 확인할 수 있다.
