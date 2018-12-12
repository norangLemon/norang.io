---
title: 'ssl 인증서 달기'
date: 2017-04-07 05:52:32
categories: server
tags:
- server
- let's encrypt
- aws
---
오래 전부터 서버에 ssl 인증서를 달려고 했는데,
바쁘다는 이유로 차일피일 미루고 있었다.
그리고 드디어 삽질을 시작했다.

<!-- more -->

## AWS Certificate Manager
AWS에서도 무료로 인증서를 제공한다는 말을 듣고 우선 이 서비스를 이용해보기로 했다.
이용 방법은 몹시 간단하다.
[이 링크](https://aws.amazon.com/ko/blogs/korea/new-aws-certificate-manager-deploy-ssltls-based-apps-on-aws/)의 설명대로만 등록해주면 끝.

### 문제점

#### io 메일 비공개
io 도메인의 경우 whois 했을 때 메일 주소를 공개하지 않는다.
AWS에서 보내는 메일을 받을 수 없었다.
[이 링크](https://aws.amazon.com/ko/premiumsupport/knowledge-center/resend-validation-email-io/)에 나온 문제이다.
https://www.nic.io/cgi-bin/whois 에 직접 가서 계정 메일을 공개하도록 설정해 주어야 한다.
이때 비밀번호를 입력해야 하는데, 어떤 비밀번호인지 몰라 그냥 비밀번호 찾기 후 설정했음.

### 단점
#### 배포가 유료
인증서 생성은 무료지만 배포를 위하여 Elastic Load Balancers 혹은 CloudFront를 사용해야 한다.
두 서비스 모두 유료.
로드 밸런서의 경우 HTTP, HTTPS 요청이 들어올 때 부하가 걸리지 않도록 적절히 조정해주는 서비스인데,
HTTPS 요청이 들어올 경우 80번 포트로 포워딩해주면 된다는 듯.

#### 인증 메일이 잘 오지 않음
위에서 이야기한 io 도메인 문제를 해결하고 나서도,
인증 메일을 10번 보내면 1~2통 정도 오는 문제가 있었다.
두 가지 경우를 생각할 수 있다.

1. 메일이 잘 안 보내진다.
1. 메일이 정말 늦게 보내지는데, 요청이 중첩될 경우 마지막 하나만 보낸다.

수 분 단위로 지연되어서, 해당 페이지에서의 "전송되었습니다!"를 믿을 수가 없다.
다만 잘 오는 경우는 보낸 즉시 도착하니, 알 수 없는 노릇이다.

## Let's encrypt
### 생성
나는 우분투 16에 nginx를 돌리고 있기 때문에, [이 방법]대로만 따라하면 된다.
하는 김에 자동 갱신도 걸어둔다.

### 배포
위에서 생성한 녀석을 찾아, nginx에 [nginx 공식 문서](http://nginx.org/en/docs/http/configuring_https_servers.html#single_http_https_server)가 설명한 대로 설정한다.

let's encrypt는 모든 인증서를 `/etc/letsencrypt/live`에 저장한다.
또한 이 파일을 다른 위치로 옮기는 것을 권장하지 않는다고 한다.
해당 폴더에 들어가면 4 개의 파일과 README를 발견할 수 있다.
리드미에 다음과 같은 설명이 나와 있다.

* `privkey.pem`  : the private key for your certificate.
* `fullchain.pem`: the certificate file used in most server software.
* `chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.
* `cert.pem`     : will break many server configurations, and should not be used without reading further documentation (see link below).

OCSP 체인은 굳이 사용할 필요가 없는 거 같고..
privkey와 cert 파일을 합쳐놓은 것이 fullchain 파일인데, 나는 아래와 같이 설정했다.
아마도 fullchain을 쓰는 방법이 있을 것 같긴 한데, 두 개를 따로 써도 상관은 없으니.

```nginx
server {
›   listen 80;
›   server_name norang.io;
›   listen 443 ssl;
›   ssl_certificate /etc/letsencrypt/live/norang.io/cert.pem;
›   ssl_certificate_key /etc/letsencrypt/live/norang.io/privkey.pem;
›   root /home/norang/norang.io/public;
›   index index.html index.htm;
}
```

이렇게 설정하고 나면 https에서도 접속이 잘 되는 것을 확인할 수 있다!
