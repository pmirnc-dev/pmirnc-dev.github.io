---
layout: post
title:  "Let's Git it!"
date:   2024-06-27 09:00:00 +0900
author: ihhwang
categories: ['git', 'github']
---
<hr/>

안녕하세요.

예전에 전략기획실에 Git 강의 자료를 만들었었는데 내용을 좀 더 보강하여 발표하게 되었습니다.

다들 많이 사용하셔서 알겠지만 정리 & 복습 차원으로 같이 보시죠.

# 목차

1. [버전관리란?](#버전관리란?)
2. [Git의 역사?](#git의-역사)
3. [Git의 구조](#git의-구조)
4. [Git 시작하기](#git-시작하기)
5. [Git Clone](#clone)
5. [Git Conflict](#conflict)

# 버전관리란?

> 형상관리 혹은 버전 관리는 프로그램을 개발함에 있어 소프트웨어의 **변경사항을 체계적으로 추적하고 통제하는 것**으로,
> 형상 관리는 일반적인 단순 버전관리 기반의 소프트웨어 운용을 좀 더 포괄적인 학술 분야의 형태로 넓히는 근간을 이야기한다.
by 위키피디아
>

우리가 메모장 혹은 Excel, Power Point 등등 여러 편집기 에서 `Ctrl` + `z` 를 통해 현재 작성 이전 단계로 되돌릴 수 있습니다.

이처럼 내가 원하는 시점으로 다시 이동할 수 있게 해주는 것이 버전관리 입니다.

## 장점
- 처리속도가 빠르다.
- 팀원 간의 협업을 원활하게 하다.
- 변경 이력을 추적 가능하다.
- 문제가 발생했을 때 이전 버전으로 되돌릴 수 있다.

## 단점
- 직관적이지 못해서 서드파티의 의존이 높을 수 있다.

# Git의 역사

아주 짧게 요약하자면, Git은 2005년 **리누스 토르발스**가 개발한 분산 버전 관리 시스템입니다.

# Git의 구조

![structure1.png](/assets/images/ihhwang/git-history/structure1.png)

나무위키에서 가져온 git의 동작 구조 입니다.

최상단에 원격 서버가 있고 하단에는 Local 저장소가 있습니다.

이 그림을 3줄 요약 하면 다음과 같습니다.

1. 로컬 저장소에서 작업 변경사항을 저장합니다.(add, commit)
2. 로컬 저장소의 작업 내용을 원격 저장소에 동기화 요청합니다.(push)
3. 동기화가 완료되면 최신 변경 사항을 가져옵니다.(pull)

앞으로 많이 사용하게 될 주요 키워드는 다음과 같습니다.

- **저장소 (Repository)**: 프로젝트의 모든 파일과 변경 이력이 저장되는 곳입니다.
- **커밋 (Commit)**: 파일의 스냅샷을 저장하는 단위로, 변경 사항을 기록합니다.
- **브랜치 (Branch)**: 독립적으로 작업을 진행할 수 있는 분기점입니다. 기본 브랜치는 `main` 또는 `master`입니다.
- **스테이징 영역 (Staging Area)**: 커밋 될 파일들을 잠시 보관하는 공간입니다.
- **원격 저장소 (Remote Repository)**: 네트워크를 통해 접근할 수 있는 저장소로, 협업 시에 주로 사용됩니다.

# Git 시작하기

## 다운로드 & 설치

1. Git 홈페이지 접속: https://git-scm.com/
2. 내 os에 맞는 버전 다운로드
3. 설치
4. 끝!

## 기본 설정

### git 정보 설정
config 파일에 git 정보가 설정이 되어 있지 않다면 error 사항이 꽃필 것이다.
![만신](/assets/images/ihhwang/git-history/ksm1.jpg)

```
git config --global user.name "username"
git config --global user.email "you@example.com"

>git config user.name
username

```

## 로컬 저장소 생성

원격 저장소에서 생성 후 clone을 해도 되지만 처음부터 보여드리기 위해 로컬저장소 생성부터 보여드리겠습니다.

먼저 프로젝트를 생성 할 폴더를 만들고 IDE를 사용하여 폴더를 엽니다.

그 후 터미널을 열어 git init 을 입력해주면 초기 세팅이 완료됩니다.

![git](/assets/images/ihhwang/git-history/git01.png)

![git](/assets/images/ihhwang/git-history/git02.png)

로컬 git 저장소는 3가지 영역으로 나뉘어 있습니다.

1. 작업 디렉토리(Working Directory)
2. 인덱스(Index)
3. 헤드(HEAD)

**작업 디렉토리**는 

실제 파일들로 이루어져 있습니다. 커밋 전에는 인덱스에 올려놓고 얼마든지 변경할 수 있습니다.

**인덱스**는 

커밋을 할 항목들 입니다. 준비 영역(staging area)은 사용자가 git commit 명령을 실행했을 때 

Git이 처리할 것들이 있는 장소입니다.

**헤드**는 

최종 확정본(commit) 영역입니다. 현재 브랜치를 가리키는 포인터이며, 브랜치에 담긴 커밋 중 가장 마지막 커밋을 가리킵니다. 


## 작업

이제 프로젝트 작업을 시작해보겠습니다. 여기서는 git에 대한 간단한 테스트만 할 예정이라

readme.md 와 .gitignore 정도만 추가해보겠습니다.

.md는 markdown 파일이며 readme.md 는 이 프로젝트를 설명하는 안내문 이라고 볼 수 있습니다.

.gitignore는 프로젝트를 git에 저장할 때 변경사항에 포함하지 않는 파일들 입니다. 주로 시스템이나 라이브러리 파일들은 용량도 크고 무겁기 때문에 포함하지 않습니다. 

https://www.toptal.com/developers/gitignore
해당 사이트에 접속하여 os, ide, 프로그래밍 언어 정보를 입력하면 필요한 ignore 정보를 알아서 만들어줍니다.

```
# Created by https://www.toptal.com/developers/gitignore/api/windows,visualstudiocode
# Edit at https://www.toptal.com/developers/gitignore?templates=windows,visualstudiocode

### VisualStudioCode ###
.vscode/*
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
!.vscode/*.code-snippets

# Local History for Visual Studio Code
.history/

# Built Visual Studio Code Extensions
*.vsix

### VisualStudioCode Patch ###
# Ignore all local history of files
.history
.ionide

### Windows ###
# Windows thumbnail cache files
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db

# Dump file
*.stackdump

# Folder config file
[Dd]esktop.ini

# Recycle Bin used on file shares
$RECYCLE.BIN/

# Windows Installer files
*.cab
*.msi
*.msix
*.msm
*.msp

# Windows shortcuts
*.lnk

# End of https://www.toptal.com/developers/gitignore/api/windows,visualstudiocode

```

## Git 상태 확인하기

### status

터미널에 git status 를 입력하면 현재 git의 상태를 확인할 수 있습니다.

``` text
PS D:\dev\GitStudy> git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        readme.md

nothing added to commit but untracked files present (use "git add" to track)
```

여기서 Untracked files 는 추적되지 않은 파일들을 표시하여 git에서 한번도 상태관리가 되지 않은 파일들을 표시해줍니다.

### add
`git add <file>` 을 사용하여 staging에 추가할 수 있습니다.

편하게 IDE에서 제공하는 기능을 사용하여 추가할 수도 있고 직접 명령어를 입력할 수도 있습니다.

![git](/assets/images/ihhwang/git-history/git03.png)

파일명이 간단해서 터미널에 입력을 했봤습니다.

```
PS D:\dev\GitStudy> git add .gitignore
PS D:\dev\GitStudy> git add readme.md
```

아까와는 달라진 내용을 확인할 수 있습니다.
```
PS D:\dev\GitStudy> git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitignore
        new file:   readme.md

```

변경사항에 있던 파일들이 스테이징된 변경사항으로 옮겨진 것을 볼 수 있습니다.

![git](/assets/images/ihhwang/git-history/git04.png)

### commit

이제 staging에 있는 파일들을 commit 해줍니다. commit은 staging된 파일들의 스냅샷을 로컬 저장소에 저장하게 해줍니다. commit을 통해 변경 이력을 남기고, 이후에 특정 시점으로 되돌아가거나 다른 사람들과 협업할 때 중요한 역할을 합니다.


```
D:\dev\GitStudy> git commit -m "docs: First Commit!"
[master (root-commit) f078e20] docs: First Commit!
 2 files changed, 56 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 readme.md
```


#### 메세지 규칙

여기서 -m은 메세지를 의미하며 커밋에 대한 메세지를 추가할 수 있습니다.
혼자 할때는 아무렇게나 적어도 되지만 팀으로 협업할때는 규칙을 적용하면 보다 편하게 소통할 수 있습니다.


보통의 규칙은 다음과 같습니다.

글 쓰기 규칙에서 서론, 본론, 결론과 같이 Header, Body, Footer 로 나눕니다.

```bash
타입(스코프): 주제(제목) // Header

본문 // Body

바닥글 // Footer
```

**Header**는 필수이며 스코프는 생략 가능 합니다.

타입은 커밋을 하는게 어떤 내용인지 나타내는 라벨 입니다.

솔직히 여기 있는 모두 다 지키진 않고 feat, fix, docs 정도만 사용하는 것 같습니다. 

| 타입 이름 | 내용 |
| --- | --- |
| **feat** | 새로운 기능에 대한 커밋 |
| **fix** | 버그 수정에 대한 커밋 |
| **build** | 빌드 관련 파일 수정 / 모듈 설치 또는 삭제에 대한 커밋 |
| **chore** | 그 외 자잘한 수정에 대한 커밋 |
| **ci** | ci 관련 설정 수정에 대한 커밋 |
| **docs** | 문서 수정에 대한 커밋 |
| **style** | 코드 스타일 혹은 포맷 등에 관한 커밋 |
| **refactor** | 코드 리팩터링에 대한 커밋 |
| **test** | 테스트 코드 수정에 대한 커밋 |
| **perf** | 성능 개선에 대한 커밋 |

**Body** 는 Header의 내용을 뒷받침 한 상세한 내용을 적습니다.

Header에서 충분히 설명 가능하다면 생략해도 됩니다.

**Footer** 는 바닥글로 어떤 이슈에서 왔는지 참조정보들을 추가하는 용도로 사용합니다.

Footer 또한 생략 가능합니다.

**작성 예시**
```bash
// Header
git commit -m "fix(backend): 저장 로직 버그 수정 완료"

// Body
유저 저장 정보 로직에서 나이 필드에 타입 오류가 발생하여 validation 추가 완료했습니다.

// Footer
resolves: #1234

```

![git](/assets/images/ihhwang/git-history/git05.png)

commit을 완료하면 구름 아이콘과 게시 Branch 로 바뀌게 됩니다.
COMMITS 메뉴에 방금 입력했던 commit이 보이는 것을 확인할 수 있습니다.

### push

push 는 로컬에 있는 git의 변경 사항을 원격 서버로 동기화 요청을 하는 것 입니다.

지금은 원격 서버의 정보가 없기 때문에 push 를 하기 앞서 테스트로 사용 할 원격 서버를 추가해보겠습니다.

## GitHub

### GitHub란?

git을 기반으로 한 웹 서비스로, 프로젝트를 관리하고 협업할 수 있는 플랫폼 입니다.
github 외에 GitLab, Bitbucket, SoruceForge, Azure DevOps Repos 등등이 있습니다.

### 저장소(Repository) 생성하기

![git](/assets/images/ihhwang/git-history/git06.png)

git을 원격 저장소에 저장하기 위한 공간을 생성합니다. Repository name 에 저장소의 이름을 적어줍니다.

그 다음은 공개여부를 설정 합니다. 모든 사람에게 공개해도 된다면 public, 나만 볼거라면 private 를 선택해줍니다.

Add a README files 와 Add .gitignore는 이미 추가했기 때문에 체크해주지 않습니다.

마지막으로 Create reposityry  버튼을 클릭하면 저장소가 생성됩니다.


repository 를 생성하면 원격 저장소의 주소가 생성됩니다.

```
https://github.com/pmirnc-dev/my_project.git
```

이 주소를 가지고 다시 vscode 로 이동합니다.

```
git remote add origin https://github.com/username/my_project.git
```

여기서 username은 개인 혹은 팀의 ID, my_project에는 repository name으로 바꿔줍니다.

터미널에 입력해주면 REMOTE 탭에 repositoy name에 입력했던 명칭으로 생성이 됩니다.

```
PS D:\dev\GitStudy> git remote add origin https://github.com/pmirnc-dev/GitStudy.git
```

![git](/assets/images/ihhwang/git-history/git07.png)

### push
이제 로컬 저장소의 커밋을 원격 저장소에 전달해야 합니다.

다음과 같은 커맨드를 사용하며 원격 브랜치는 생략 가능합니다.
```
git push <원격 저장소> <로컬 브랜치>:<원격 브랜치>
git push -u origin master
```

주요 옵션은 다음과 같습니다

* -u, --set-upstream: 로컬 브랜치와 원격 브랜치를 추적 브랜치로 설정합니다.
* -f, --force: 강제로 push를 하여 원격 브랜치의 기록을 덮어씁니다.
* --all: 모든 로컬 브랜치를 원격 저장소에 push 합니다.


```
PS D:\dev\GitStudy> git push -u origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 717 bytes | 358.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/pmirnc-dev/GitStudy.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

push가 완료되면 GitHub는 다음과 같이 변합니다.

![git](/assets/images/ihhwang/git-history/git08.png)


팀으로 협업하기

이제 팀으로 프로젝트를 진행할 때를 살펴보겠습니다.

먼저 확인해야 할 개념은 branch 입니다.

## branch

현재 사용자가 작업하고 있는 작업 공간 입니다.

따로 설정해주지 않는다면 기본으로 main 혹은 master 라는 branch 가 기본으로 생성 됩니다.

PMI 개발부는 **이름/작업명** 으로 각자의 작업 공간을 나눠서 사용하기로 약속했습니다.

예시로 racoi 프로젝트의 git tree 모습 입니다.

![git](/assets/images/ihhwang/git-history/git09.png)

### branch 분기하기

1. 분기하려는 branch 우클릭 -> Create branch(vscode 기준)

![git](/assets/images/ihhwang/git-history/git10.png)

2. 브랜치명 입력

![git](/assets/images/ihhwang/git-history/git11.png)

3. 여기서는 3가지 선택지가 나옵니다. Create & Switch to Branch 를 하면 브랜치 성성 및 새로 생성한 브랜치로 이동이 됩니다.

![git](/assets/images/ihhwang/git-history/git12.png)

4. 새로 생성된 branch

![git](/assets/images/ihhwang/git-history/git13.png)


## Pull Request(PR)

이제 각자가 작업한 내용들을 master(main) branch 에 병합을 해야 합니다.

앞서 설명했던 add, commit, push 까지는 동일하고 Pull Request 라는 새로운 개념이 있습니다.

Pull Request는 개발자들이 자신이 작업한 작업물을 다른 팀원들에게 검토하고 병합을 요청하는 기능 입니다.

ihhwang/branch_test 에서의 작업물을 원격 저장소로 push 하겠습니다.

push를 하면 github에 노란색 공간이 하나 생기고 Compare & pull request 버튼을 클릭합니다.

![git](/assets/images/ihhwang/git-history/git14.png)

Open pull request 화면으로 넘어가면서 내 작업물에 대한 설명을 추가 할 수 있고, 하단에는 변경사항이 나와있습니다.

![git](/assets/images/ihhwang/git-history/git15.png)

Create pull request 버튼을 클릭하면 github 에서 자동으로 conflict 체크를 합니다.

![git](/assets/images/ihhwang/git-history/git16.png)

요청에 대한 코멘트도 추가할 수 있습니다.

![git](/assets/images/ihhwang/git-history/git17.png)

작업물에 이상이 없다면 Merge pull request 버튼을 클릭하여 master branch에 ihhwang/branch_test 를 병합합니다.

![git](/assets/images/ihhwang/git-history/git18.png)

![git](/assets/images/ihhwang/git-history/git19.png)

mrege가 완료되면 보라색으로 Merged 상태로 바뀌며 branch 삭제여부도 나옵니다.
더이상 사용하지 않을거라면 깔끔하게 삭제를 하고 계속 이어서 작업을 하려면 그냥 두시면 됩니다.

![git](/assets/images/ihhwang/git-history/git20.png)

## Pull

이제 master branch의 상태가 변합니다. 새로운 변경사항이 병합됐으므로 master 또한 동기화를 해줘야 합니다.
master branch로 checkout 합니다.

![git](/assets/images/ihhwang/git-history/git21.png)

변경내용 동기화를 하면 master 에도 index.html, main.js가 생성됩니다.

![git](/assets/images/ihhwang/git-history/git22.png)

# clone

프로젝트를 새로 받아올 때 사용합니다.
github 사이트에서 code 버튼을 클릭하면 나오는 url를 복사합니다.

![git](/assets/images/ihhwang/git-history/git23.png)

프로젝트를 생성하는 방법이 두가지가 있습니다.

1. 내 폴더로 clone하기
2. IDE에서 clone하기

## 내 폴더로 clone

![git](/assets/images/ihhwang/git-history/git24.png)

프로젝트를 생성할 폴더로 들어가 마우스 우클릭을 하여 git bash here 를 선택합니다.(git이 설치되어 있어야 보입니다.)

git command 창이 뜨는데 여기에 git clone url주소 를 입력합니다.

![git](/assets/images/ihhwang/git-history/git25.png)

프로젝트 복사가 완료되면 해당 폴더에 Repository name으로 폴더가 하나 생긴 것을 확인할 수 있습니다.

![git](/assets/images/ihhwang/git-history/git26.png)

## IDE 에서 clone하기
vscode 기준으로 설명하겠습니다.
중간에 Git 리포지토리 복제를 클릭하면 상단에 주소를 입력하는 공간이 나옵니다.

![git](/assets/images/ihhwang/git-history/git27.png)

주소를 입력후 복제할 폴더를 선택합니다.

![git](/assets/images/ihhwang/git-history/git28.png)

# Conflict
가끔 작업을 하다보면 같은 코드를 수정하고 branch를 병합하면 충돌이 발생하는 경우가 있습니다.
처음 겪게되면 무척 당황스러운데 걱정 할 필요 없습니다.
(업무 분담 및 소통의 중요성)


충돌 상황을 재연하기 위해 먼저 master 에서 main.js에 코드를 한 줄 수정 후 push까지 진행하겠습니다.
그 다음 다른 branch로 이동하여 똑같은 부분을 다른 코드로 수정해봅니다.

![git](/assets/images/ihhwang/git-history/git29.png)

아까와는 다르게 Conflict 라고 메세지가 나오는 것을 확인할 수 있습니다.

![git](/assets/images/ihhwang/git-history/git30.png)

Resolve conflicts 를 클릭하여 확인해봅니다.


![git](/assets/images/ihhwang/git-history/git31.png)

```
<<<<<<< ihhwang/branch_test
console.log('야 나두!');
=======
console.log('야너두');
>>>>>>> master

```

======= 을 기준으로 위는 ihhwang/branch_test 이고 아래는 master 입니다.


최종적으로 남길 코드를 자유롭게 편집합니다.

```
console.log('야 너두?');
console.log('야 나두!');
```


수정을 하고 우측에 Mark as resolved 를 클릭하면 commit mrege 로 변합니다.

![git](/assets/images/ihhwang/git-history/git32.png)

![git](/assets/images/ihhwang/git-history/git33.png)

이제 conflict test를 통과했고 Merge를 할 수 있습니다.

![git](/assets/images/ihhwang/git-history/git34.png)

끝!!

![그럼이만](/assets/images/ihhwang/img.gif)

---
# 참조

* You Don’t Know JS Yet
* https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures
* https://adjh54.tistory.com/64
* https://ko.javascript.info/closure#ref-1091
* https://poiemaweb.com/js-closure