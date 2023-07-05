---
layout: post
title:  "postcode api"
date:   2023-07-05 10:59:07 +0900
author: tkkim
categories: API Vuetify 
---
<hr/>


# Postcode API
<hr/>

토이 프로젝트를 다루면서 처음 접하게 되었던 기능 중

post api, vuetify 내 v-dialog 의 아름다움에 대해 발표하고자 합니다.

# postcode 저장

**postcode api**는 우리가 회원가입 내 정보 기입 중 주소를 기입할때 사용하는 기능입니다.

![Untitled](/assets/images/tkkim/20230705/1.png)



저는 postcode api 중 우리가 자주 접하는 daum postcode api를 설치했습니다

`npm install vue-daum-postcode --save`

- code

```javascript
    // router > index.ts
    import VueDaumPostcode from "vue-daum-postcode"
    
    Vue.use(VueDaumPostcode)
    
```

```javascript
    // signUp.ts
    <template>
      <v-container class="mx-auto signUpContainer" min-width="100%" min-height="100vh">
              <vue-daum-postcode
                  @complete="oncomplete"
              />
      </v-container>
    </template>
    
    <script lang="ts">
    import { vue-daum-postcode } from "vue-daum-postcode"
    import { Component, Vue } from "vue-property-decorator";
    
    @Component({
      components: { vue-daum-postcode }
    })
    .
    .
    .
    </script>
    
```


위 예시처럼 router에 설정 후 component 형식으로 사용할 수 있습니다

자 이제

vuetify 중 v-text-field를 사용해 input을 만들어 보겠습니다

- code

```javascript
    // signUp.ts
    <template>
      <v-container class="mx-auto signUpContainer" min-width="100%" min-height="100vh">
              <div>
                <v-sheet>
                  <v-text-field
                      label="우편번호"
                      v-model="userInfo.zipcode"
                      prepend-inner-icon="mdi-map-marker"
                      readonly
                  >
                    <template v-slot:append>
                      <v-btn icon small tabindex="-1" @click="openPostcode">
                        <v-icon>mdi-magnify</v-icon>
                      </v-btn>
                    </template>
                  </v-text-field>
                </v-sheet>
                <v-text-field
                    label="주소"
                    v-model="userInfo.addr1"
                    readonly
                ></v-text-field>
                <v-text-field
                    label="상세주소"
                    v-model="userInfo.addr2"
                ></v-text-field>
              </div>
      </v-container>
    </template>
```


![Untitled](/assets/images/tkkim/20230705/2.png)

값을 담을 수 있는 환경을 조성 했으니 값을 넣어줘야겠죠?

→ v-dialog를 이용해 만들어 보겠습니다



## beauty vuetify v-dialog

버튼을 클릭 했을때 openPostcode 함수를 실행시켜 dialog를 띄울 수 있습니다.

- code

```javascript
    <template>
      <v-container class="mx-auto signUpContainer" min-width="100%" min-height="100vh">
              <div>
                <v-sheet>
                    . 
                    . 
                    .
                    <template v-slot:append>
                      <v-btn icon small tabindex="-1" @click="openPostcode">
                        <v-icon>mdi-magnify</v-icon>
                      </v-btn>
                    </template>
                  </v-text-field>
                    .
                    .
                    .
                </v-sheet>
             
              </div>
         
            <v-dialog v-model="dialog" max-width="500" persistent>
              <v-toolbar>
                <v-toolbar-title>주소검색</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon plain @click="closePostcode">
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-toolbar>
              <vue-daum-postcode
                  @complete="oncomplete"
              />
            </v-dialog>
      </v-container>
    </template>
    
    <script lang="ts">
    import { Component, Vue } from "vue-property-decorator";
    
    @Component({
      components: {}
    })
    export default class signUp extends Vue {
        .
    	.
    	.
      openPostcode(){
        this.dialog = true;
      }
    	.
    	.
    	.
    }
    </script>
```


**point!**

1. v-dialog 에서는 v-show or v-if가 아닌  **v-model**를 사용합니다
2. vuetify 사용 시 꼭 공식 문서를 참고하세요!



## postcode 옵션 활용

postcode 사이트 내에 기재되어있는 다양한 옵션들 중

**`zonecode` - 우편번호**

**`roadAddress` - 도로명 주소**

**`jibunAddress` - 지번 주소**

를 사용해 구현을 했습니다.


## 로직 설명

**`userSelectedType`**(값 - R/J) 으로 input값에 반영될 주소 타입을 설정할 수 가 있는데

사용자가 선택한 주소의 타입이 R인 경우 - R( 도로명 )주소

사용자가 선택한 주소의 타입이 J(else)인 경우 - J( 지번 )주소

타입 설정 후 값을 저장하고 this.dialog = false로 dialog 창을 닫아줍니다



이외에도 다양한 옵션들이 많으니  위 사이트를 참고하시길 바랍니다.

---

참조

[https://postcode.map.daum.net/guide](https://postcode.map.daum.net/guide)

[https://www.npmjs.com/package/vue-daum-postcode](https://www.npmjs.com/package/vue-daum-postcode) - **npm 설치 참고 사이트**

[https://vuetifyjs.com/en/components/dialogs/](https://vuetifyjs.com/en/components/dialogs/)
