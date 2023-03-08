---
layout: post
title:  "Electron GitHub 로 배포 하기"
date:   2023-03-08 14:35:00 +0900
author: ihhwang
categories: vue.js electron javascript
---
<hr/>

![Untitled](https://nklayman.github.io/vue-cli-plugin-electron-builder/hero.png)
# Vue + Electron GitHub 로 배포하기

현재 프로젝트로 진행중인 metasurvey-downloader 프로그램을 git hub로 배포하고 자동 업데이트 기능 구현 까지의 간략한 여정을 담아봤습니다.  

## Git Hub Release 기능이란?
GitHub 에서 제공하는 소프트웨어의 소스코드 뿐만 아니라 빌드한 결과물을 배포할 수 있는 기능 입니다.

![downloader1.png](/assets/images/ihhwang/downloader1.png)

Draft a new release 를 통해 배포할 프로그램을 업로드 할 수 있지만, 자동으로 할 수도 있다.

## Release 설정하기
1. 새로운 버전임을 감지하는 기준은 package.json의 version 항목으로 판단한다.

    새로운 버전을 업데이트 하고 싶으면 숫자를 변경하면 된다.

    변경 후, main branch에 merge를 해준다.

    ex) 1.0.0 -> 1.0.1

2. github의 repository 설정 

    package.json 에서 다음과 같은 항목을 추가해 줍니다.
    
    
    "repository": {
        "type": "git",
        "url": <repository_url>
    }

3. vue.config.js 파일 수정

electronBuilder > builderOption > publish 에 github를 추가해 줍니다.


    module.exports = defineConfig({
        transpileDependencies: true,
        pluginOptions: {
            ...
            electronBuilder: {
                nodeIntegration: true,
                builderOptions: {
                    publish: ['github']
                }
            }
        }
    })


4. script 수정

package.json의 script에서 publish 옵션을 추가해 줍니다.

-p always

    
    "electron:build-prod": "vue-cli-service electron:build -p always --mode prod",


5. private repository 일 경우 환경변수에 git token을 세팅해 줍니다.

![downloader2.png](/assets/images/ihhwang/downloader2.png)


6. build가 완료되면 github release에 자동으로 올라가고 latest 설정을 해줘야 합니다. 

![downloader3.png](/assets/images/ihhwang/downloader3.png)

### 참조

[https://www.electron.build/](https://www.electron.build/)

[https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/recipes.html#table-of-contents](https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/recipes.html#table-of-contents)

[https://programmerk.tistory.com/55](https://programmerk.tistory.com/55)

[https://junglow9.tistory.com/7](https://junglow9.tistory.com/7)

