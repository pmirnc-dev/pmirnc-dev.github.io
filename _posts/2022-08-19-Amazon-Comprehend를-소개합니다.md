---
layout: post
title:  "Amazon Comprehend를 소개합니다"
date:   2022-08-19 09:30:00 +0900
author: hsyoo
categories: 빅데이터 NLP AWS 자연어처리
---
<hr/>

저희 피앰아이에서는 빅데이터 분석을 위해 자연어처리(NLP)를 하고 있는데요,  
AWS ML 서비스 중에 도움이 될만한 쉽고 간편하게 써볼 수 있는 서비스가 있어 준비해 봤습니다.

**Amazon Comprehend**입니다

> Amazon Comprehend is a natural-language processing (NLP) service that uses machine learning to uncover valuable insights and connections in text.
> 
> _Amazon Comprehend는 기계 학습을 사용하여 텍스트에서 유용한 인사이트 및 관계를 찾아내는 자연어 처리(NLP) 서비스입니다._  
> [https://aws.amazon.com/comprehend/?nc1=h_ls](https://aws.amazon.com/comprehend/?nc1=h_ls){:target="_blank"}

작동방식은 다음과 같습니다

![process](https://d1.awsstatic.com/Comprehend/Use-cases_Customer-analytics.93d8f858f529bec19d1899c9d8a5625d3989e621.png)


### API 예제
간편하게 AWS console에서도 확인이 바로 됩니다

API call
```json

{
  "Text": "아이폰은 좋긴한데 결제가 안되는 게 너무 불편하다",
  "LanguageCode": "ko"
}

```

API response

Entities
```json
{
  "Entities": [
    {
      "Score": 0.9941830039024353,
      "Type": "COMMERCIAL_ITEM",
      "Text": "아이폰",
      "BeginOffset": 0,
      "EndOffset": 3
    }
  ]
}
```

Key phrases
```json
{
  "KeyPhrases": [
    {
      "Score": 0.9898720383644104,
      "Text": "아이폰",
      "BeginOffset": 0,
      "EndOffset": 3
    },
    {
      "Score": 0.999206006526947,
      "Text": "결제",
      "BeginOffset": 10,
      "EndOffset": 12
    },
    {
      "Score": 0.9998993873596191,
      "Text": "게",
      "BeginOffset": 18,
      "EndOffset": 19
    }
  ]
}
```

Sentiment
```json
{
    "Sentiment": {
        "Sentiment": "MIXED",
        "SentimentScore": {
            "Positive": 0.00021102314349263906,
            "Negative": 0.009446796029806137,
            "Neutral": 0.00009593776485417038,
            "Mixed": 0.9902461767196655
        }
    }
}
```



아래는 Python을 통해 간단하게 분석하는 예제입니다. (감성분석)
```python

import boto3
import json

comprehend = boto3.client(service_name='comprehend', region_name='region')
text = "오늘은 하루 종일 비가 내려서 우울하다. 맛있는 부침개가 생각나는 날이다."

print('Calling DetectSentiment')
print(json.dumps(comprehend.detect_sentiment(Text=text, LanguageCode='ko'), sort_keys=True, indent=4))
print('End of DetectSentiment\n')

```

이에 대한 응답은 아래와 같습니다.

```json
{
    "SentimentScore": {
        "Mixed": 0.54585512690246105,
        "Positive": 0.01592071056365967,
        "Neutral": 0.2985543131828308,
        "Negative": 0.7093945890665054
    },
    "Sentiment": "NEGATIVE",
    "LanguageCode": "ko"
}
```

이랜드 이노플에서 Amazon Comprehend 기반 설공 상품평 분석을 통한 트렌드 예측 개선에 활용한 사례
[https://aws.amazon.com/ko/blogs/korea/amazon-comprehend-goods-trend-prediction/](https://aws.amazon.com/ko/blogs/korea/amazon-comprehend-goods-trend-prediction/)){:target="_blank"}

더 자세한 내용은 아래 링크를 참고해 주시길 바랍니다
[https://aws.amazon.com/ko/comprehend/](https://aws.amazon.com/ko/comprehend/){:target="_blank"}

요금은 아래 링크를 확인 해주세요  
[https://aws.amazon.com/ko/comprehend/pricing/?pg=ln&sec=hs](https://aws.amazon.com/ko/comprehend/pricing/?pg=ln&sec=hs){:target="_blank"}

