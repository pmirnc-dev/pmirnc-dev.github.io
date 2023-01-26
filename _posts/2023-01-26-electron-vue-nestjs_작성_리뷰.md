---
layout: post
title:  "Electron 초간단 빠른 리뷰"
date:   2023-01-26 16:23:00 +0900
author: mclee
categories: electron vue nestjs
---
<hr/>

## Electron + vue + Nestjs 작성 후기

---

## backend

### nest.js 설치 및 electron 설치

1. backend 폴더내에 electron을 설치
    - npm install -d electron 설치
2. /src/main.ts electron 실행시키는 소스를 작성

```typescript
//...생략
bootstrap();
/**
 * NestJS에 electron 실행 처리
 * */
const { app, BrowserWindow, Menu } = require('electron')

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
  win.loadURL('http://localhost:4000/')
}
/**
 * 메뉴 설정
 * */
const template = [
  {
    label: 'File',
    submenu: [{ role: 'toggleDevTools' }, {role:'quit'}],
  }
];

const menu = Menu.buildFromTemplate(template);
Menu.setApplicationMenu(menu);
/**
* electron 화면 창 생성
*/
app.whenReady().then(() => {
  createWindow();
  app.on('activate', () => {
    if(BrowserWindow.getAllWindows().length === 0){
      createWindow()
    }
  })
})
/**
* electron 종료
*/
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});
```

3. package.json
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
  },
```
- "main": "./dist/main.js" 추가
- scripts 내에 "electron:start": "tsc && npx electron ." 추가


4. tsconfig.json 파일 내에 "include": ["src/**/*.ts"] 추가
```json
{
"include": [
"src/**/*.ts"
],
"compilerOptions": {
//...생략
}
```
※ 이유
- npm run electron:start 실행시 dist 폴더내에 src 폴더 포함해서 생성됩니다.

  dist - src - [파일/폴더 포함]
- npm run start or npm run start:dev 실행시 src 폴더 내에 있는 파일과 폴더들이 dist 폴더에 그대로 생성이 됩니다.

  dist - [파일/폴더 포함]
- 중복되는 파일과 폴더와 용량 증가 되는 부분을 막기 위해서 include 부분을 추가 하였습니다.


5. scripts - electron:start 실행하면 electron 창이 띄워지는걸 확인됩니다.

---

## frontend

### vue 설치

1. backend 폴더에서 화면을 띄우기 위해서 vue.config.js 파일 내용을 추가
```typescript
module.exports = {
  outputDir: '../back/public'
}
```

2. frontend - package.json - scripts - build를 실행 후에는 backend폴더내에 public 폴더가 생성되는걸 볼 수 있습니다.

3. frontend에서 할 일은 다 끝났습니다.
    - 추후에 화면 변경시 build로 실행을 하면 됩니다.

---

## electron 빌드(배포)

### 빌드 설치
1. npm install -d electron-builder 설치 후 backend - package.json - scripts에 스크립트 추가
    - "electron:build" : "electron-builder"

2. backend - src - main.ts내에 소스를 수정 작업을 합니다.
```typescript
async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, '../', 'public'));
  await app.listen(4000);
}
```
#### ※ 빌드 전에 backend 포트 번호를 변경합시다. 다른 개발중인 것과 간섭을 피하기 위해서 입니다.

3. 마지막으로 backend - package.json 소스를 추가를 합시다
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

4. 이걸로 끝입니다.

   electron:build를 실행하게 되면 ~~dist 폴더내에 zip 파일과 win-unpacked 폴더가 생성이 됩니다.~~

   build 폴더가 생성이 되며 빌드한 파일들이 들어가게 됩니다.
    - build -> win-unpacked 폴더내에 들어가시면

      productName에 적혀 있는 이름이 실행프로그램 이름으로 되어 있을겁니다. 클릭해서 실행 하시면 됩니다.

---

### DB 연결 및 .env 경로 설정

1. DB 연결은 기존에 하던거 처럼 작성하면 됩니다.
2. Electron 빌드시 포함시킬 폴더 및 파일 경로를 작성하시면 됩니다. 
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
- .env 파일을 폴더에 넣어서 포함을 시켜도 되고,
- 파일명만 작성 하시면 됩니다.
3. 소스 툴에서 잘 작동하는데 electron 빌드시에 작동이 안됩니다.
```typescript
//test.Module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env'
      //1안 join(__dirname, '../..', 'default/.env'),
      //2안 join(__dirname, '../..', '.env'),
    }),
    //...생략
  ],
})

```
※ 이유 및 해결 방안
- .env 파일을 찾을 수가 없어 DB 연결하는 필요한 MONGO_URI를 불러오지 못해서 정상 작동이 안되는 이슈 입니다.
- 해결 방법은 join(__dirname, '../..', '.env') 
 
  현재 실행중인 파일(test.Module.ts) 경로인 절대경로를 통해서 .env 파일 경로를 찾으시면 됩니다.


---
참조 사이트
- https://qiita.com/A_T_B/items/e68b4c85b3e8e4c4aa98#4-vuejs%E3%81%AE%E7%B5%84%E3%81%BF%E8%BE%BC%E3%81%BF
- https://velog.io/@altmshfkgudtjr/Electron-%EC%A0%81%EC%9A%A9-%EB%B0%8F-%EB%B9%8C%EB%93%9C%ED%95%98%EA%B8%B0