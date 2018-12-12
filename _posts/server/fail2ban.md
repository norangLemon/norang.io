---
title: fail2ban
date: 2018-12-06 08:13:19
categories: server
tags:
- ssh
---

간만에 서버 관리를 하다가, 문득 ssh brute-force 공격이 내 서버에도 들어오나 싶어 조회해봤다.
놀랍게도 아주 많은 횟수의 공격이 들어와 있었다.
물론 root 계정으로는 로그인이 안 되고, 보안에 취약한 admin, ubuntu 등등의 계정은 모두 지운 상태였다.
그렇다고 하더라도, 계속된 공격을 그저 방치하는 게 신경쓰이기는 한다.
마침 [fail2ban]을 추천받아서, 이 기회에 사용해보기로 했다.

<!-- more -->

[fail2ban]은 `/var/log/auth.log`를 살펴서 로그인을 시도했으나 계속 실패한 IP를 차단해준다.
물론 관리자가 직접 로그를 살펴서 차단하는 일을 할 수도 있겠지만 매우 귀찮으므로..
차단할 실패 횟수, 실패시 로그인 잠금 등은 직접 설정하도록 되어있다.

설치는 [fail2ban] 페이지에 설명되어있는 대로 쭉 따라하면 된다.
개인적으로는 tar로 설치하는 것보다 git clone 해오는 것이 더 좋아서, clone 하는 방식을 선택했다.

```
git clone https://github.com/fail2ban/fail2ban.git
cd fail2ban
sudo python setup.py install 
```

잘 설치되었는지는 아래 두 개의 명령어로 각각 확인할 수 있다.

```
fail2ban-client -h
fail2ban-client version
```

service에 등록하기 위해서 아래의 커맨드를 입력한다.
root directory에 접근하므로, sudo 권한이 필요할 수 있다.

```
cp files/debian-initd /etc/init.d/fail2ban
update-rc.d fail2ban defaults
service fail2ban start
```

이렇게 실행하더라도, 세부적인 사항은 직접 config에 설정해주어야 한다.
차단 설정은 `/etc/fail2ban/jail.conf`에 있지만, 이 파일을 직접 수정하지 않고 `/etc/fail2ban/jail.d/jain.local`에 추가하도록 [권장]된다.

`/etc/fail2ban/jail.conf`를 열어 보면 다양한 설정이 주석처리되어 있는 것을 확인할 수 있다.
나는 그 중에서 아래와 같은 일부만 해제해서 추가했다.

```
# [DEFAULT]
bantime = 1h

# sshd 공격을 차단한다
# [sshd]
enabled = true

# 기본 설정이다. 세부 설정으로 덮어씌워질 수 있다고 한다.
[DEFAULT]

#
# MISCELLANEOUS OPTIONS
#

# bantime을 실패에 따라서 증가시키기 위한 옵션.
bantime.increment = true

# 최대 bantime. bantime.rndtime으로 랜덤하게 설정할 수도 있다고 한다.
#bantime.maxtime = 999999

# 지수적으로 bantime을 증가시키는 옵션. 1로 설정한 결과는 1, 2, 4, 8, 16... 이라고 한다.
bantime.factor = 4

# 자기 자신의 IP는 차단하지 않는 옵션.
ignoreself = true
```

이와 같이 설정을 업데이트 한 후 이 내용을 반영하기 위해서는 재시작해주는 것으로 충분하다.

```
service fail2ban restart
```

현재 sshd 차단 상황을 아래와 같이 확인할 수 있다.

```
sudo fail2ban-client status sshd
```

[fail2ban]: https://github.com/fail2ban/fail2ban
[권장]: https://github.com/fail2ban/fail2ban/wiki/Proper-fail2ban-configuration
