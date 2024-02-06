---
layout: post
title:  "TensorFlow MobileNet (이미지분류)"
date:   2024-02-06 07:02:00 +0900
author: 
categories: TensorFlow MobileNet
---
<hr/>

# TensorFlow MobileNet

## TensorFlow란?

 TensorFlow는 구글에서 개발한 오픈 소스 머신러닝 라이브러리입니다. 딥러닝에서 데이터를 의미하는 Tensor와 딥러닝 모델을 구성하는 연산들의 그래프에 따라 수행되는 형태인 Flow를 합쳐 만들어진 이름이며, 주로 심층 신경망을 비롯한 다양한 기계 학습 및 딥러닝 모델을 구축하고 학습하는 데 사용됩니다. 


## MobileNet이란?

![mobilenetImg.png](/assets/images/jwyoo/mobilenetImg.png)

 MobileNet은 2017년 구글이 개발한 경량 심층 신경망 아키텍처 중 하나로 매우 경량화된 CNN 모델입니다. 스마트폰 및 기타 모바일 장치와 같이 리소스가 제한된 환경에서도 효율적으로 동작하도록 설계되어, 이미지 분류, 객체 감지, 시멘틱 세그멘테이션등 다양한 컴퓨터 비전 작업에 널리 사용되고 있습니다.


## MobileNet 구조 및 성능

MobileNet은 Depthwise Separable Convolution이라는 기법을 사용하여 파라미터 수를 줄이고 연산량을 최적화합니다.


![mobilenetImg2.png](/assets/images/jwyoo/mobilenetImg2.png)

Depthwise Separable Convolution은 Depthwise Convolution (깊이별 합성곱)과 Pointwise Convolution (점별 합성곱) 단계로 구성되어있습니다.


각 단계를 강아지 품종 분류로 예시를 들어 설명해보겠습니다.

- Depthwise Convolution (깊이별 합성곱)

 입력 데이터의 각 채널마다 따로따로 합성곱을 수행하므로 각각의 강아지 이미지에 대해 깊이별 합성곱을 적용합니다. 이 과정에서는 입력 이미지의 각 채널마다 특징을 추출합니다. 예를 들어, 각 채널은 강아지의 얼굴, 몸통, 다리 등과 같은 다양한 부분을 나타냅니다.
각 채널에 대해 서로 다른 커널을 적용하여 해당 부분의 특징을 감지합니다. 이를 통해 강아지의 특성을 더 잘 이해할 수 있습니다.


- Pointwise Convolution (점별 합성곱)
  
 깊이별 합성곱을 통해 얻은 특징 맵을 점별 합성곱에 입력합니다. 이 과정에서는 각 특징 맵의 깊이 방향에서 정보를 섞고 조합하여 더욱 풍부한 특징을 추출합니다. 점별 합성곱은 1x1 크기의 필터를 사용하여 각 위치의 특징을 결합합니다. 이를 통해 강아지의 특징들을 공간적으로 조합하고, 다양한 특징을 고려하여 품종을 구분할 수 있습니다.


 따라서, 깊이별 합성곱은 각각의 채널에서 강아지 이미지의 다양한 부분을 감지하고, 점별 합성곱은 이러한 부분들을 조합하여 강아지의 특징을 더욱 효과적으로 추출하여 강아지 품종을 분류하는 데 사용됩니다. 이러한 합성곱 연산을 통해 MobileNet은 강아지 이미지를 효과적으로 분류하고 품종을 식별할 수 있습니다.


## 스크립트를 통해 JavaScript 프로젝트에서 사용하기.

```javascript
<!-- TensorFlow.js 라이브러리를 로드합니다. MobileNet을 사용하기 위해 필요합니다. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.1"></script>

<!-- MobileNet 모델을 로드합니다. -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/mobilenet@1.0.0"></script>

<!-- CORS 설정이 이미지를 읽을 수 있도록 하세요. 이미지를 교체하세요! -->
<img id="img" src="cat.jpg"></img>

<!-- 아래의 script 태그에 코드를 추가하세요. 외부 .js 파일을 사용할 수도 있습니다. -->
<script>
    // 'import' 문이 없습니다. 'mobilenet' 및 'tf'는 script 태그로 인해
    // index 페이지에서 사용 가능합니다.

    // 'img' id를 가진 요소를 가져옵니다.
    const img = document.getElementById('img');

    // 모델을 로드합니다.
    mobilenet.load().then(model => {
    // 이미지를 분류합니다.
    model.classify(img).then(predictions => {
        console.log('Predictions: ');
        console.log(predictions);
    })
});
</script>
```


## NPM을 통해 JavaScript 프로젝트에서 사용하기.

```typescript
// TensorFlow.js의 MobileNet 모델을 가져오기. (사전에 이미 훈련된 이미지 분류 모델)
const mobilenet = require('@tensorflow-models/mobilenet');

// HTML 문서에서 id가 'img'인 요소를 가져와서 img 변수에 할당하기. (여기서 img는 분류할 이미지)
const img = document.getElementById('img');

// MobileNet 모델 로드
const model = await mobilenet.load();

// 로드된 모델을 사용하여 이미지를 분류.
// model.classify() 함수는 이미지를 입력으로 받아 그에 대한 예측 생성
const predictions = await model.classify(img);

// 예측 결과를 콘솔에 출력
console.log('Predictions: ', predictions);
```

## vue에서 활용한 mobileNet 이미지 분류 코드

```vue
// vue에서 만들어보는 간단한 MobileNet 코드
<template>
  <div>
    <input type="file" accept="image/*" @change="previewImage($event)">
    <br><br>
    <img v-if="imageUrl" :src="imageUrl" alt="Preview">
    <div v-if="predictions.length > 0">
      <h2>Predictions:</h2>
      <ul>
        <li v-for="(prediction, index) in predictions" :key="index">
          {{ prediction.className }}: {{ Math.floor(prediction.probability * 100) }}%
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import * as mobilenet from '@tensorflow-models/mobilenet';

export default {
  name: 'ImageClassifier',
  data() {
    return {
      imageUrl: '',
      predictions: []
    };
  },
  methods: {
    async previewImage(event) {
      const input = event.target;
      const file = input.files[0];
      const reader = new FileReader();

      reader.onload = async () => {
        this.imageUrl = reader.result;

        const img = document.createElement('img');
        img.src = reader.result;

        const model = await mobilenet.load();
        const predictions = await model.classify(img);
        this.predictions = predictions;
      };

      reader.readAsDataURL(file);
    }
  }
};
</script>
```

## 결과
![mobilenetImg3.png](/assets/images/jwyoo/mobilenetImg3.png)



### 후기
모델이 특정 이미지를 잘못 분류하는 경우가 종종 있습니다. 분류 오류를 해결하는 가장 효과적인 방법은 데이터셋을 개선하고 미리 학습된 모델 이외에 모델을 조정하는 것입니다. 그러나 완벽한 분류는 어려울 수 있으며, 항상 모델의 한계와 데이터의 제한을 염두에 두어야 합니다.


### 공식사이트
https://www.tensorflow.org/js/models?hl=ko

### 참고
[https://www.researchgate.net/figure/Depthwise-separable-convolutions_fig1_358585116](https://www.researchgate.net/figure/Depthwise-separable-convolutions_fig1_358585116)
[https://velog.io/@woojinn8/LightWeight-Deep-Learning-5.-MobileNet](https://velog.io/@woojinn8/LightWeight-Deep-Learning-5.-MobileNet)
[https://github.com/tensorflow/tfjs-models/tree/master/mobilenet](https://github.com/tensorflow/tfjs-models/tree/master/mobilenet)



