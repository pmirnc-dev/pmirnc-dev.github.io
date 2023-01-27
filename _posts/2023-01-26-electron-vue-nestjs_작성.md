---
layout: post
title:  "electron+vue+nestjs+typescript 작성"
date:   2023-01-26 16:23:00 +0900
author: mclee
categories: electron vue nestjs typescript
---
<hr/>

## Electron + vue + Nestjs + typescript 작성

---

## backend

### nest.js 설치 및 electron 설치

1). backend 폴더내에 electron을 설치
- npm install -d electron 설치

2). /src/main.ts electron 실행시키는 소스를 작성
```typescript
//...생략
bootstrap();
/**
 * NestJS에 electron 실행 처리
 * */
const { app, BrowserWindow, Menu } = require("electron")

const createWindow = () =>  {
  const win = new BrowserWindow(
    {
      width: 1280,
      height: 1024,
      webPreferences: {
        nodeIntegration: true,
        contextIsolation: false,
      }
    }
  );
  win.removeMenu()  //메뉴 삭제
  win.loadURL("http://localhost:4000/")
}
/**
 * 메뉴 설정
 * */
const template = [
  {
    label: "File",
    submenu: [{ role: "toggleDevTools" }, { role: "quit" }],
  }
];

const menu = Menu.buildFromTemplate(template);
Menu.setApplicationMenu(menu);
/**
* electron 화면 창 생성
*/
app.whenReady().then(() => {
  createWindow();
  app.on("activate", () => {
    if(BrowserWindow.getAllWindows().length === 0){
      createWindow()
    }
  })
})
/**
* electron 종료
*/
app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});
```
3). package.json
```json
{
  "name": "back",
  "version": "0.0.1",
  "main": "./dist/main.js",
  //...생략
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "nest build",
    "start": "nest start",
    //...생략
    "electron:start": "tsc && npx electron ."
  }
```
- "main": "./dist/main.js" 추가를 합니다. (electron 실행시킬 파일을 경로와 같이 적어 춥니다.)
- scripts 내에 "electron:start": "tsc && npx electron ." 추가


4). tsconfig.json 파일 내에 "include": ["src/**/*.ts"] 추가
```json
{
    "include": [
      "src/**/*.ts"
    ]
    //...생략
}
```
※ 이유
- npm run electron:start 실행시 dist 폴더내에 src 폴더도 포함해서 생성됩니다.

  생성된 경로 예) dist - src - [파일/폴더 포함]


- npm run start or npm run start:dev 실행시 src 폴더 내에 있는 파일과 폴더들이 dist 폴더에 그대로 생성이 됩니다.

  생성된 경로 예) dist - [파일/폴더 포함]


- 중복되는 파일과 폴더와 용량 증가 되는 부분을 막기 위해서 include 부분을 추가 하였습니다.


5). scripts - electron:start 실행하면 electron 창이 띄워지는걸 확인됩니다.

---

## frontend

### vue 설치

1). backend 폴더에서 화면을 띄우기 위해서 vue.config.js 파일 내용을 추가
```typescript
module.exports = {
  outputDir: "../back/public"
  //...생략
}
```

2). frontend - package.json - scripts - build를 실행 후에는 backend 폴더 내에 public 폴더가 생성되는걸 볼 수 있습니다.

3). 이제 frontend 에서 해야 하는 일들은 끝났습니다.

- 추후에 화면 변경시 front npm run build로 실행을 하면 됩니다.

---

## electron 빌드(배포)

### 빌드 설치
1). npm install -d electron-builder 설치 후 backend - package.json - scripts에 스크립트 추가

   - "electron:build" : "electron-builder"

2). backend - src - main.ts내에 소스를 수정 작업을 합니다.
```typescript
async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, "../", "public"));
  await app.listen(4000);
}
```
#### ※ 빌드 전에 backend 포트 번호를 변경합시다. 다른 개발중인 것과 간섭 및 포트 충돌을 피하기 위해서 입니다.


3). 마지막으로 backend - package.json 소스를 추가를 합시다
```json
{
   "build": {
      "appId": "com.example.test",
      "productName": "Test",
      "win": {
         "target": {
            "target": "zip",
            "arch": [
               "x64"
            ]
         }
      },
      "files": [
         "dist/**/*",
         "public/**/*",
         "default/*"
      ],
      "directories": {
         "buildResources": "./dist/",
         "output": "./build/"
      }
   }
}
```

- files에 들어가는 값들은 실행시킬 폴더들로 작성해주시면 됩니다.
- [추가 2023-01-26] electron:build 할때 마다 용량이 증가되는 막기 위해서 build - directories 속성에

  buildResources(빌드 대상 폴더 경로) / output(빌드 생성 경로) 폴더 경로들을 추가

4). 이걸로 끝입니다.

   - electron:build를 실행하게 되면 ~~dist 폴더내에 zip 파일과 win-unpacked 폴더가 생성이 됩니다.~~

     build 폴더가 생성됩니다.
   - build -> win-unpacked 폴더내에 들어가시면

     productName에 적혀 있는 값이 실행프로그램 이름으로 생성되어 있습니다. 클릭해서 실행 하시면 됩니다.

---

### DB 연결 및 .env 경로 설정

1). DB 연결은 기존에 하던 것처럼 작성하시면 됩니다.

2). Electron 빌드시 포함시킬 폴더 및 파일 경로들을 작성 합니다. 
```json
//1안
{
    "build": {
        //...생략
        "files": [
            "dist/**/*",
            "public/**/*",
            "default/*"
        ]        
    }
},
//2안
{
    "build": {
        //...생략
        "files": [
          "dist/**/*",
          "public/**/*",
          ".env"
        ]
    }
}
```
- .env 파일을 폴더에 넣어서 포함을 시켜도 되고, 파일명만 작성 하시면 됩니다.

3). WebStrom에서 실행할 때는 잘 작동하는데 electron 빌드하고 나서 작동이 안됩니다.

```javascript
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ".env"
      //1안 join(__dirname, "../..", "default/.env"),
      //2안 join(__dirname, "../..", ".env"),
    }),
    //...생략
  ],
})
```

※ 이유 및 해결 방안
- .env 파일을 찾을 수가 없어 DB 연결하는 필요한 MONGO_URI의 값을 불러오지 못하여 작동이 안되는 이슈 입니다.
- 해결 방법은 스크립트 주석으로 1안, 2안으로 작성을 하였습니다. 
 
  설명: 현재 실행중인 파일(Module.ts) 경로인 절대경로를 통해서 이전 폴더로 이동을 하여 .env 파일 경로를 찾으시면 됩니다.

#### 리뷰

electron 알기 전까지는 vue + nestjs + typescript로 다 작성하고 나서 배포를 어떻게 할까 고민하고 있는 도중에 개발부 단체 미팅 중에서 황인호 대리님이 이것을 사용중에 있다고 말을 듣고 나서 electron 검색을 하여 이걸로 배포하면 되겠구나 생각을 하고 바로 기존 작업하던 프로젝트에 electron을 적용시겼습니다.

로컬 실행까지 잘 작동하다가 빌드(배포)후 실행 할때 마다 렌더러 프로세스(front) 페이지가 나타나지 않아서 많이 당황을 했습니다. 결국에는 원인을 찾아서 해결을 한 상황이지만요.


---
참조 사이트
- [일본 사이트 - electron + vue + nextjs 앱 만들기](https://qiita.com/A_T_B/items/e68b4c85b3e8e4c4aa98#4-vuejs%E3%81%AE%E7%B5%84%E3%81%BF%E8%BE%BC%E3%81%BF)
- [Electron 적용 및 빌드하기 - 빌드 속성](https://velog.io/@altmshfkgudtjr/Electron-%EC%A0%81%EC%9A%A9-%EB%B0%8F-%EB%B9%8C%EB%93%9C%ED%95%98%EA%B8%B0)