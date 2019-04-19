---
title: symlink & dotfiles
date: 2019-04-19 08:58:54
tags:
- server
- dotfiles
- shellscript
---

linux 기반 환경에서 zsh, bash, vim, tmux 등의 파일의 설정 파일은 보통 홈 디렉토리에 있으며, `.`으로 시작한다.
`ls` 명령어로 조회했을 때, 옵션 없이는 `.`으로 시작하는 파일을 보여주지 않기 때문이다.
(`ls -a`로 숨겨진 파일도 확인할 수 있다.)
이런 연유로, 설정 파일들은 `.`으로 시작하고 그것들을 dotfiles라고 부른다.

dotfile은 깃헙으로 관리하면 편하다.
자신의 설정을 따로 만든 경우에, 다른 PC 또는 클라우드 환경에서 갑자기 필요할 때가 있기 때문이다.
미리 github에 올려둔 경우, 인터넷만 연결되어 있으면 쉽게 클론해서 사용할 수 있다.

<!-- more -->
나 또한 아주 예전부터 [dotfile을 업로드해서 관리](https://github.com/norangLemon/.dotfiles)하고 있다.
이 때, 이 파일들을 일일히 홈 디렉토리로 옮기는 것은 매우 귀찮기 때문에, symbolic link(symlink)를 사용하고 있었다.
원래 파일은 깃 폴더에 두되, 그 파일로의 심볼릭 링크를 생성하는 것이다.
간단히 말해서 바로가기를 만들어 두어서 파일을 일일히 복사하지 않아도 되도록 한 것이다.
바로가기와 같기 때문에, 가령 홈 디렉토리의 `.vimrc`를 열어보면 내가 원하는 파일이 열리게 된다.
뿐만 아니라 심링크 파일이 원 파일을 파일 경로에 따라서 링크하기 때문에, 경로를 바꾸지만 않는다면 원 파일의 수정사항을 따로 심링크 파일에 적용해 줄 필요가 없다.

심링크를 생성하는 방법은 아래와 같이 간단하다.
```
ln -s ORIGIANL_FILE SYMLINK_FILE
```
링크할 파일들과 그 경로는 정해뒀기 때문에, 위 명령어를 `make_link.sh`에 넣은 후 `./make_link.sh`로 한 번에 실행하는 식으로 쓰고 있었다.

그런데 다양한 환경에서 작업을 하다보니, 다음과 같은 요구사항이 생겨났다.

1. 이미 존재하는 dotfile은 백업해두기
1. 회사의 dotfile은 숨겨두기

오늘은 아래의 방법으로 이 두 문제를 해결해서 [더 업그레이드 된 `make_link.sh`](https://github.com/norangLemon/.dotfiles/blob/master/make_link.sh)을 만들었다.

#### 이미 존재하는 dotfile 백업하기

예전에는 `ln -sf`로 옵션을 주어 링크로 만드려는 파일이 이미 있는 경우 덮어씌우게 설정했었다.
링크 파일을 덮어씌우는 데에는 두 가지 문제가 있었는데,

1. 디폴트 설정이 있는 경우 사라지게 됨
1. [symlink race condition]이 일어남

2의 경우, 비밀번호 등이 관련된 파일이 아니면 상관없긴 하나, 1의 경우는 큰 문제였다.
간혹 디폴트 설정을 참고하거나, 혹은 다시 돌아가고 싶을 때가 있었기 때문이었다.

따라서 원래 파일이 있는 경우, 무조건 백업하도록 바꾸었다.

```zsh
function backup {
    FILE="$HOME/$1"               # 입력받은 인자 $1을 이용해 파일 경로 설정
    if [ -e "$FILE" ]; then       # 해당 파일이 존재하면
        mv "$FILE" "$FILE.backup" # 백업 파일을 만들어준다.
        echo -e "$FILE \e[1mis moved to\e[0m $FILE.backup"
    fi
}
```

여기서 `-e` 옵션은 해당 파일이 존재하는지 검사한다.
파일이 존재할 경우, 파일을 백업한 뒤 백업했다는 메시지를 `echo` 명령어를 통해 남겼다.
이 때, `\e1m`은 [prompt customization]으로, 뒤따라오는 문자열을 굵은 글씨체로 표시하게 한다.
뒤의 `\e0m`은 뒤의 문자열을 보통 굵기로 표시한다.

그러나 이 코드에도 문제가 있었다.
링크할 대상이 유효하지 않은 경우, `-e` 옵션으로 해당 파일을 검사했을 때 `0`이 나오기 때문이었다.
즉, 깨진 링크 파일은 존재하지 않는 것으로 검사된다.

```console
$ ln -s not_exist_file link_file
$ ls -al link_file
cli> lrwxr-xr-x  1 myusername  mygroup  3 Apr 19 19:51 link_file -> not_exist_file
$ if [ -e "link_file" ]; then
  echo "file exist"
else
  echo "file not exist"
fi
cli> file not exist
```

위와 같이 존재하지 않는 파일에 대해 링크를 생성한 후,
`ls` 명령어로 생성된 파일을 확인하면, 존재하지 않는 파일로 링크하고 있는 것을 확인할 수 있다.
이를 `-e` 옵션을 통해 검사해보면, 파일이 존재하지 않아 `file not exist`를 출력하는 것을 확인할 수 있다.

그런데 **파일 자체의 존재와 더불어, 이것이 링크인지 확인하는 옵션** `-L`을 사용하면 해당 파일의 존재 유무를 잘 가려낸다.

```console
$ if [ -L "link_file" ]; then
  echo "link exist"
else
  echo "link not exist"
fi
cli> link exist
```

따라서 최종적으로는 링크 파일인지 먼저 검사한 뒤, 링크 파일이면 이것을 지우고 그렇지 않은 경우는 백업하는 식으로 수정했다.
dotfile이 링크 파일인 경우에는 내가 건드렸을 것이므로.
그렇지 않은 파일은 원래 있었던 것일 테니까, 안심하고 백업하면 된다.

```zsh
function backup {
  FILE="$HOME/$1"
  if [ -L "$FILE" ]; then     # 링크 파일은 삭제
    rm "$FILE"
    echo -e "$FILE is symlink; \e[1mremoved\e[0m"
  elif [ -e "$FILE" ]; then   # 일반 파일은 백업
    mv "$FILE" "$FILE.backup"
    echo -e "$FILE \e[1mis moved to\e[0m $FILE.backup"
  fi
}
```

#### 회사의 dotfile은 숨겨두기

이 경우, 단순히 깃헙에 올리는 것은 `.gitignore`에 등록해서 막을 수 있다.
하지만 해당 파일을 대신 사용하려고 할 경우 또 다시 일일히 링크를 만들어 주어야 하니까, 최소 두 번의 스크립트를 돌려야 하는 귀찮음이 있다.

나는 이것조차도 너무나 귀찮은 사람이기 때문에, 한 파일에 때려박되 회사 환경에서 작업할 때만 다른 파일에 링크하는 것으로 해결했다.
우선 스크립트 하나에서 `HOST` 환경변수를 조사해서 회사 환경과 아닌 환경을 구분했다.
회사인 경우 `CORP`변수를 세팅하고, 아닌 경우 비워둔다.
이렇게 빈 스트링은 쉘 스크립트에서 `-n`으로 체크하면 `0`을 반환한다.
따라서 이런 식의 코드가 가능해진다.

```zsh
# initialize CORP using corp_set.sh
if [[ -e "corp_set.sh" ]]; then   # corp_set.sh 파일의 존재 여부 확인
  source "corp_set.sh"            # 존재하는 경우 실행
fi

if [[ -e "exclusive_zshrc" && -n "$CORP" ]]; then  # 설정파일이 있고 회사 환경인 경우
  ln -s ~/.dotfiles/exclusive_zshrc ~/.zshrc
  echo -e "  \e[1mUse exclusive zshrc\e[0m"
else
  ln -s ~/.dotfiles/zshrc ~/.zshrc
fi
```

이 때, 중요한 것은 `corp_set.sh` 파일을 `source`로 불러오는 것이다.
만약에 `zsh corp_set.sh`과 같이 파일을 실행하면 환경변수가 제대로 설정되지 않는다.
왜냐하면, 해당 환경 변수가 설정된 쉘 환경이 **종료된 뒤** 원래 환경으로 돌아오기 때문이다.
그러나 `source`를 사용하면 해당 스크립트의 명령을 읽어와서 **현재 환경에서 실행**한다.

특정 환경에서는 다른 설정을 사용하고 싶을 때 이런 방법을 사용할 수 있지 않을까 싶다.

---

참고링크

- [shell script checker](https://www.shellcheck.net/)
- [bash if-test option list](https://www.tldp.org/LDP/Bash-Beginners-Guide/html/sect_07_01.html)
- [bash -e option vs -a option](https://stackoverflow.com/questions/321348/bash-if-a-vs-e-option)

[symlink race condition]: https://en.wikipedia.org/wiki/Symlink_race
[prompt customization]: https://wiki.archlinux.org/index.php/Bash/Prompt_customization#Examples
