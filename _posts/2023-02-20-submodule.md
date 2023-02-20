---
layout: post
title: "[Git] Submodule"
summary: "Git Submodule"
author: hyerin
date: "2023-02-20 22:38:00 +0900"
category: Git
thumbnail: /assets/img/posts/230220_thumbnail.jpg
keywords: Frontend
permalink:
usemathjax: true
---

> 수정해서 쓰고 있는 라이브러리를 메인 프로젝트 repo 내에서 함께 관리하고 싶은 생각에 "하나의 repo 내에서 다수의 프로젝트를 함께 관리할 수 있나요?" 란 질문을 했고, `Git Submodule`이 있단 팀원의 말씀을 들었다! Git의 Submodule에 대해 살펴보자!

# 서브모듈(Submodule)이란?

Git의 레포지토리 하위에 다른 저장소를 관리하기 위한 도구이다. 상위 레포지토리를 `Super project(슈퍼 프로젝트)`, 하위 레포지토리를 `Submodule(서브모듈)`이라고 부른다. (혹은 부모 저장소, 자식 저장소라고 부른다.) 서브 모듈을 사용하면 특정한 Git 레포지토리를 다른 레포지토리의 하위 디렉토리로 사용할 수 있다.<br />
슈퍼 프로젝트에 서브모듈을 추가하면 슈퍼 프로젝트가 하위 모듈의 특정 커밋을 가리키게 된다. 그리고 슈퍼 프로젝트는 현재 가리키고 있는 하위 모듈의 파일을 슈퍼 프로젝트에 추가하게 된다.

# 서브모듈(Submodule) 사용하기

## 레포지토리 생성

Github 또는 Bitbucket에 `super-repository`와 `sub-repository`라는 이름의 레포지토리를 생성한다. 각각은 슈퍼 프로젝트와 서브 모듈이며, 로컬 환경으로 클론한다.<br />
`sub-repository` 레포지토리에 파일을 하나 추가하고 커밋, 푸시해준다.

```bash
$ git add .
$ git commit -m 'first commit'
$ git push origin main
```

## 서브 모듈 등록

`super-repository`의 레포지토리의 `lib` 디렉토리에 `sub-repository`를 서브 모듈로 등록한다. `super-repository` 디렉토리로 이동하여 아래의 명령어를 입력한다.

```bash
$ git submodule add https://github.com/Hailey0930/sub-repository.git lib
```

`super-repository` 디렉토리에 `sub-repository`가 클론되며, 서브모듈이 등록된다.

## .gitmodules

`.gitmodules` 파일이 생성된 것을 확인할 수 있다. 이 파일에는 서브 디렉토리와 서브 모듈 저장소 URL이 매핑된 설정 정보가 담겨있다.

```bash
$ cat .gitmodules
[submodule "lib"]
    path=lib
    url=https://github.com/Hailey0930/sub-repository.git
```

`.gitmodules`도 `.gitignore`과 마찬가지로 버전 관리 대상이다. 이 파일을 통해 프로젝트 참여자들이 이 프로젝트에 어떤 서브 모듈이 있는지 확인할 수 있다.

## lib 디렉토리

```bash
$ git diff --cached lib
diff --git a/lib b/lib
new file mode 160000
index 0000000..e45f25d
--- /dev/null
+++ b/lib
@@ -0,0 +1 @@
+Subproject commit e45f25dc08aabcfa40b666125bdb389a2375e0c7
```

> `git diff` 명령어를 사용해 Git 디렉토리에서 파일이 변경된 내용을 확인할 수 있다.<br /> `--cached` (혹은 `--staged`) 옵션은 아직 커밋되지 않고 스테이지에 올라간 파일 대상으로 변경 내용을 확인한다. `lib`는 변경 사항을 확인할 대상이다.

`lib` 디렉토리에 서브모듈이 클론되고 파일이 생성되었는데, `git diff`의 결과물에는 보이지 않는다.<br />
Git은 `lib` 디렉토리를 서브모듈로 취급하기 때문에 `lib` 하위 파일의 변경사항을 추적하지 않는다. 대신 서브 모듈 디렉토리를 통째로 특별한 커밋으로 취급한다.<br />
마지막 줄에 나와있는 커밋 해시를 기억하고, 서브 모듈 디렉토리로 이동해서 아래와 같은 명령을 입력한다.

```bash
$ git log
commit e45f25dc08aabcfa40b666125bdb389a2375e0c7 (HEAD -> main, origin/main)
Author: devHudi <devhudi@gmail.com>
Date:   Sat Jul 30 00:09:43 2022 +0900

    first commit
```

`git diff` 명령을 사용했을 때 출력된 커밋 해시와 동일한 것을 알 수 있다. 즉 슈퍼 프로젝트는 서브 모듈의 특정 커밋을 가리키고 있다.

## 서브 모듈 변경 후 슈퍼 프로젝트에 반영

서브모듈 디렉토리롤 이동하여 아래 명령을 입력해서 `b`라는 파일을 생성하고 커밋한다.

```bash
$ touch b
$ git add .
$ git commit -m 'second commit'
$ git push origin main
```

다시 슈퍼 프로젝트 디렉토리로 이동하여 pull을 받아온다.

```bash
$ git pull
Your configuration specifies to merge with the ref 'refs/heads/main'
from the remote, but no such ref was fetched.
```

`lib`는 `super-repository`와 독립적인 저장소이기 때문에 pull을 해오지 않는 것을 볼 수 있다. `lib` 디렉토리로 이동한 다음 다시 pull을 해보자!

```bash
$ cd lib
$ git pull
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 0), reused 2 (delta 0), pack-reused 0
Unpacking objects: 100% (2/2), 208 bytes | 208.00 KiB/s, done.
From https://github.com/devHudi/sub-repository
   e45f25d..5eade94  main       -> origin/main
Updating e45f25d..5eade94
Fast-forward
 b | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b
```

정상적으로 pull되고 b라는 파일도 생성됐다.

## 슈퍼 프로젝트에 더 편하게 반영하기

```bash
$ cd super-directory
$ git submodule update --remote
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 0), reused 2 (delta 0), pack-reused 0
Unpacking objects: 100% (2/2), 208 bytes | 104.00 KiB/s, done.
From https://github.com/devHudi/sub-repository
   5eade94..7ced843  main       -> origin/main
Submodule path 'lib': checked out '7ced843b8f637a5bf9096edd56da92a97b99f907'
```

## 서브모듈을 포함한 프로젝트 클론하기

`--recurse-submodules` 옵션을 사용해 서브모듈을 자동으로 함께 가져올 수 있다.

```bash
$ git clone --recurse-submodules https://github.com/Hailey0930/super-repository.git
```

# 서브모듈 사용 케이스

서브모듈을 사용하면 복수의 레포지토리에서 공통으로 사용되는 여러 상수나 로직 등 공통으로 사용되는 부분을 서브 모듈로 분리하여 관리할 수 있게 된다.<br />
또한, 서브모듈을 사용하면 민감한 정보를 프라이빗 레포지토리에 저장하고 해당 레포지토리를 서브 모듈로 프로젝트에 등록할 수 있다. 민감한 정보엔 데이터베이스 연결 정보, JWT 시크릿 키 등이 있을 것이다. 이런 정보를 서브 모듈을 사용하여 관리할 경우 데이터를 보호할 수 있을 뿐만 아니라 민감 정보의 형상관리까지 가능해진다.<br />
프라이빗 레포지토리를 서브모듈로 등록할 경우 서브 모듈 레포지토리에 권한이 없는 사람이 슈퍼 프로젝트를 클론해도 민감 정보를 조회할 수 없다.
