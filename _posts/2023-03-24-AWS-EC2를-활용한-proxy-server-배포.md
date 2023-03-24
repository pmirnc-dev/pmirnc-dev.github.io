---
layout: post
title:  "AWS EC2를 활용한 proxy server 배포"
date:   2023-03-24 09:00:00 +0900
author: hsyoo
categories: AWS EC2 proxy puppeteer
---
<hr/>

## 개요

최근 [puppeteer](https://pptr.dev/){:target="_blank"}를 이용한 개발 프로젝트를 진행하고 있습니다.
특정 웹사이트에서 pptr로 탐색을 하다가(전혀 무리하거나 많은 양이 아니였음에도) 저희 IP가 차단되었습니다.
사실 해당 웹사이트에서 필요한 정보가 크게 중요하진 않았으나 IP 기반으로 차단되는 경우
[proxy server](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%EC%84%9C%EB%B2%84){:target="_blank"}로 해결이 가능 할 것 같아 방법을 찾아보았습니다.

오늘의 키워드는 [AWS AMI](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/AMIs.html){:target="_blank"}와
전설적인 nodejs 프로세스 매니저인 [PM2](https://pm2.keymetrics.io/docs/usage/quick-start/){:target="_blank"}에 대해 알아보겠습니다.
<hr/>

## AMI

AMI는 간단하게 EC2의 현재 상태를 이미지화하는 겁니다.  
이미지화 해놓고 새로운 EC2 인스턴스를 생성할 때 해당 이미지와 똑같이 생성할 수 있습니다.  
proxy server 설치를 위해 저는 다음과 같은 작업을 EC2에 했습니다.  

- EC2 생성

![img.png](/assets/images/hsyoo/ec2-instance.png)

~~t4g 티어 가성비 좋습니다~~

- node.js 설치


```shell

# nvm을 통한 node 16버전 설치
# https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 16
```

- PM2와 [proxy-chain](https://github.com/apify/proxy-chain) 설치

```shell
# proxy-chain은 꽤 인기가 좋은 nodejs proxy server 라이브러리입니다.

npm init
npm i -g pm2
npm i proxy-chain

```

- proxy.app.js 파일 작성

```javascript
const ProxyChain = require('proxy-chain');
const server = new ProxyChain.Server({
    port: 8080,
    // verbose: true,
});

server.listen(async () => {
    console.log(`Proxy server is listening on port ${8080}`);
});
```

- PM2를 통한 proxy server 실행

```shell
pm2 start proxy.app.js --name proxy

# 재부팅 시 실행되도록 
# https://pm2.keymetrics.io/docs/usage/startup/
pm2 startup
# 위 커맨드 입력 시 나오는 스크립트를 실행
sudo su -c "env PATH=$PATH:/home/unitech/.nvm/versions/node/v14.3/bin pm2 startup <distribution> -u <user> --hp <home-path>"
# 현재 설정을 저장
pm2 save
```


- AMI 생성 

![img.png](/assets/images/hsyoo/ami.png)

<hr/>

## 적용

여기까지 하셨다면 앞으로 EC2인스턴스 생성 시 해당 AMI로 시작하실 수 있습니다.  
EC2들은 public IP와 DNS를 가지게 되고 이를 통해 proxy server에 접근할 수 있습니다.

```typescript

  const browser = await puppeteer.launch({ headless: false, args: [
      '--lang="ko-KR"',
      '--proxy-server=http://ec2-000-000-000-000.ap-northeast-2.compute.amazonaws.com:8080'
    ]});
  const [page] = await browser.pages();
  await page.goto('https://www.findip.kr/');
  // proxy server의 아이피로 나오는 것을 확인할 수 있습니다.
```



윈도우에서 프록시 서버로 설정해서 사용해도 잘 적용이 되는 모습입니다.
![img.png](/assets/images/hsyoo/windows-proxy.png)


- 심화편 AWS SDK를 이용해 proxy server를 동적으로 생성

```typescript
class Proxy {
  async createInstance() {
    const instanceParams: AWS.EC2.RunInstancesRequest = {
      TagSpecifications: [
        {
          ResourceType: 'instance',
          Tags: [
            {
              Key: 'Name',
              Value: this.options?.name,
            },
          ]
        }
      ],
      ImageId: this.options?.imageId,
      InstanceType: this.options?.instanceType,
      KeyName: this.options?.keyName,
      MinCount: 1,
      MaxCount: 1,
      SecurityGroupIds: this.options?.securityGroupIds,
    };
    return this.EC2.runInstances(instanceParams).promise();
  }
}

```
<hr/>

## 마치며

생략한 부분이 있지만 이 정도 내용이면 누구나 쉽고 빠르게 proxy server를 구축할 수 있어 보입니다.  
다만 대용량을 프록시 서버를 통해 다운로드 받는다 던가 한다면 AWS의 데이터 전송 비용이 많이 나올 여지가 있어보이네요.  
우회의 목적보다는 IP 차단을 잠시 피하는 정도로만 사용하시는 것을 추천드립니다.
