---
layout: post
title:  "데이터 가공 도전기 ( with $group )"
date:   2024-05-30 10:30:07 +0900
author: tkkim
categories: nestjs mongodb 
---
<hr/>


# 데이터 가공 도전기 ( with $group)

데이터 가공이라고 해봤자 이관 작업 후 불러온 데이터 값을 기반으로 가공 시키거나 
그저 **`$match`** 를 이용해서 값을 불러오는 것이 전부였기에 해당 데이터를 어떻게 가공했는지 기록할 필요성을 느껴 작성해보았습니다.

<br />

## 짜야 할 코드의 구성 파악

![Untitled](/assets/images/tkkim/20240530/image.png)

위 그림을 보고 대강 흐름을 파악해보자면

관리자 페이지에서는 각 행에 기관 9개의 데이터가 보이고 
기관마다 그룹 별 응답자 수를 구해야 하며 그룹은 총 5그룹으로 나뉘어져 있습니다.

또한 구분값 체크박스를 클릭했을때 해당 구분값 ( 사전, 사후, 전체를 의미 ) 에 맞게 표가 노출되어야합니다.

기관별로 보여지는 페이지에서는 사전 / 사후로 나누어서 데이터를 출력하고 
각각의 응답 시기( 사전 / 사후 )에 따라 그룹 별  응답자 수를 구해야 합니다.

구성 후, 관리자, 기관별 나눠서 코드를 작성할 필요가 있다고 느껴 코드를 작성했습니다.

<br />

## 초기 코드

```tsx
    public async getData(data: DataDto) {
      const$match: Record<string, any> = { $and: [] };
      let$option: Record<string, any> = {};
    
      // admin 여부
      if (data.key === '지역문화진흥원') {
        // 구분값이 없을경우
    
        if (data.option.length === 0) {
          constwholeTable= await this.setAdminData($match);
          return {
            result: true,
            data: [{ type: 'overAll', chartData:wholeTable}],
          };
        } else {
          constoptionTables= [];
          letoptionTable= {};
    
          for (constoofdata.option) {
            constvalue=o['value'];
            // 구분값 중 '전체'를 선택했을경우
    
            if (value=== 'overAll') {
    					optionTable= await this.setAdminData($match);
            } else {
    					$option= { SQ2: `${Option[value]}` };
    					optionTable= await this.setAdminData($match,$option);
            }
    					optionTables.push({ type:value, chartData:optionTable});
          }
    
          return { result: true, data:optionTables};
        }
      } else {
        constwholeTable= await this.setUserData($match,data.key,$option);
    
        return { result: true, data: [{ type: 'user', chartData:wholeTable}] };
      }
    }
    
    // 관리자 테이블
    async setAdminData(
    $match: Record<string, any>,
    $option?: Record<string, any>,
    ) {
      constrowDatas= [];
      for (leti= 0;i< 6;i++) {
        if ($option) {
    				$match=i!== 0 ? { $and: [{ SQ1: `${i}` }] } : { $and: [] };
    				$match.$and.push($option);
        } else {
    				$match=i!== 0 ? { SQ1: `${i}` } : {};
        }
    
        constrowData= await this.getResult($match);
    
        constresult= {
          title:i!== 0 ? `organization${i}` : 'Total',
          ...rowData,
        };
    
    rowDatas.push(result);
      }
      returnrowDatas;
    }
    
    // 기관별 테이블
    async setUserData(
    $match: Record<string, any>,
    userName: string,
    $option?: Record<string, any>,
    ) {
      constrowDatas= [];
      for (leti= 0;i< 3;i++) {
    			$match= { $and: [{ SQ1: `${Organization[userName]}` }] };
    			$option=i!== 0 ? { SQ2: `${i}` } : {};
        if ($option)$match.$and.push($option);
    
        constrowData= await this.getResult($match);
    
        constresult= {
          title:i=== 0 ? 'Total' :i== 1 ? 'before' : 'after',
          ...rowData,
        };
    
    rowDatas.push(result);
      }
      returnrowDatas;
    }
    
    async getResult($match: Record<string, any>) {
      constresultData= await this.dataRepository.getWholeData({$match});
      constcompleteNum=resultData.length;
    
      constgroup1=resultData.filter((r) => {
        returnr['XXM'] === '1';
      }).length;
      constgroup2=resultData.filter((r) => {
        returnr['XXM'] === '2';
      }).length;
      constgroup3=resultData.filter((r) => {
        returnr['XXM'] === '3';
      }).length;
      constgroup4=resultData.filter((r) => {
        returnr['XXM'] === '4';
      }).length;
      constgroup5=resultData.filter((r) => {
        returnr['XXM'] === '5';
      }).length;
    
      return {completeNum,group1,group2,group3,group4,group5};
    }
    
```


목돌아가는 비둘기….

초기 코드 설명(의도)를 잠시 설명하자면

구분 값 체크박스를 염두에 둔 상태로 두개의[ 관리자 / 기관별 ]메소드로 나눠 **`$match`** 에 각각 필요한 조건을 추가 시키고, 
**`$match`** 조건의 맞는 전체의 데이터를 뽑아 일일이 그룹별 응답자 count를 했습니다.

<br />

## 문제점

위 코드 작성하며 발견한 문제점을 보자면….

1. 하나의 메소드로 만들 수 있음에도 두개로 나눠버림

   (작성한 메소드에서 공통으로 사용되는 코드를 발견해 하나의 메소드로 작성할 수 있음을 깨달음)

2. 이후 기관 명이 추가될때마다 수정 해야하는 번거로움을 간과한 반복문의 남용
3. ‘전체의 데이터’를 불러와서 그룹마다 filtering한 후 count 함

이 모든 문제점들을 **`$group`** 하나로 해결할 수 있었습니다.

<br />

## $group 이란?

문서들을 그룹화하고 그룹화 된 문서들의 합계, 평균, 최대, 최소 등의 값을 계산할 수 있는 연산자로,

SQL의 **`GROUP BY`** 와 유사하다고 보면 됩니다.

‘계산 방법’ 영역에는 **`$sum`, `$avg`, `$max`, `$min`, `$push`, `$addToSet`** 등의 연산자를 지정할 수 있습니다.

```tsx
{
  $group: {
     _id: <계산 할 필드명>, // 그룹화할 기준 필드
     <계산 된 결과를 저장할 필드>: { <계산 방법1> : <계산 할 필드명> },
     ...
  }
}
```

<br />

## 개선

```tsx
    public async getData(data: DataDto) {
        let $match: Record<string, any> = {};
        const isAdmin = data.key === '지역문화진흥원';
        const groupSql = isAdmin ? 'SQ1' : 'SQ2';
        const option = data.option;
    
        const $group = {
          _id: `$_id.${groupSql}`,
          total: { $sum: '$count' },
          counts: { $push: { group: '$_id.XXM', count: '$count' } },
        };
    
        // 구분 체크박스 체크 하지않은 경우
        if (option.length === 0) {
          $match = !isAdmin ? { SQ1: `${Organization[data.key]}` } : {};
          const wholeTable = await this.setData($match, $group);
          return {
            result: true,
            data: [{ type: '0', chartData: wholeTable }],
          };
        } else {
          const optionTables = await Promise.all(
            data.option.map(async (opt) => {
              const value = opt['value'];
    
              // 구분값 중 '전체'를 선택여부
              $match = value === '0' ? {} : { SQ2: `${value}` };
              const optionTable = await this.setData($match, $group);
              return { type: value, chartData: optionTable };
            }),
          );
    
          return { result: true, data: optionTables };
        }
      }
    
      // 데이터 가공 메소드
      async setData($match?: Record<string, any>, $group?: Record<string, any>) {
        const rowData = await this.dataRepository.getWholeData($match, $group);
    
        const totalNum = rowData.reduce((acc, row) => acc + row.total, 0);
        const totalGroup = rowData.reduce((acc, item) => {
          item.counts.forEach((count) => {
            const duplicateIndex = acc.findIndex((x) => x.group === count.group);
            if (duplicateIndex !== -1) {
              acc[duplicateIndex].count += count.count;
            } else {
              acc.push({ group: count.group, count: count.count });
            }
          });
          return acc;
        }, []);
    
        rowData.push({ row: '0', total: totalNum, counts: totalGroup });
    
        return rowData;
      }
```


```tsx
db.getCollection("컬렉션 명").aggregate([
    {$match: {필터링할 조건}},
   {
        $group: {
          _id: {
            SQ1: '$SQ1',
            XXM: '$XXM',
            SQ2: '$SQ2',
          },
          count: { $sum: 1 },
        },
      },
      {
        $group: {
          _id: `$_id.${groupSql}`,
          total: { $sum: '$count' },
          counts: { $push: { group: '$_id.XXM', count: '$count' } },
        },
      },
      {
        $project: {
          _id: 0,
          row: '$_id',
          total: 1,
          counts: 1,
        },
      },
])
```

상단에 있는 파이프라인 간략설명 하자면

필터링 한 값을 '$SQ1', '$XXM', '$SQ2' 기준으로 그룹 화 해서 응답자 수를 count합니다.

그룹화 한 데이터를 `groupSql` 관리자 여부에 따라 반영하는 필드 기준으로 그룹 화 시켜서

**각 행의 총 응답자 수( total )와 [각 그룹의 응답자 수, 그룹 이름 ( counts )]을 만들어 가공시킬 데이터를 뽑아냈습니다.**

⇒ 해당 파이프 라인으로 수정 후 무지성 반복문에서 해방되었고 코드도 간략해진 것을 확인 할 수 있었습니다.

<br />

## 끝으로

mongoDB 파이프 라인 형식을 먼저 구상하고 이후 코드를 작성 하는 것, 그리고 mongoDB내 연산자를 사용해 
해결할 수 있는 부분에서는 십분 활용하는 것으로 backend 데이터 가공 시 효율적인 처리를 할 수 있다는 것을 느꼈습니다.

코드를 작성하기 이전, 특히 데이터를 뽑아낼 때 효율적인 처리방법을 고민해서 쓸데없는 시간낭비를 하지 않도록 해야겠습니다.

---
