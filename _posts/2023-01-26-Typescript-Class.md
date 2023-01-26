---
layout: post
title:  "Typescript Class"
date:   2022-08-17 20:15:00 +0900
author: ybcho
categories: Typescript Class 객체지향
---
<hr/>

# Typescript Class
Typescript는 Javascript를 객체지향적 프로그램 기법을 사용할 수 있게 해줍니다.
그 중 Class를 설명 드리겠습니다. Typescript의 class는 기존 객체지향언어인 java, C#의 Class와 상당히 비슷하여
사용해보셨다면 익숙한 부분을 많이 느끼실 수 있습니다.

## Class란?
class(클래스)란 객체 지향 프로그래밍에서 객체를 만들기 위한 틀,
객체의 상태(변수들)와 행동(메소드)가 정의된 곳

ex)
붕어빵틀에 재료와 행동에 따라 다른 결과물의 붕어빵(팥붕, 슈붕)을 만든다.
붕어빵틀 -> 클래스
굽다 -> 메소드
붕어빵 -> 객체

예제 코드는 게임에서 적군을 정의할때로 가정하였습니다.

class Enemy{
    hp:number;
    power:number;
    type:string;
    constructor(hp:number, power:number, type:string){
        this.hp=hp;
        this.power=power;
        this.type=type;
    }
}

선언
class Enemy{
    constructor(매개변수){
    }
    ...멤버들
}

생성
const enemy = new Enemy();

## 생성자(constructor)
인스턴스가 생성될 때 호출되는 초기화 메소드 
인스턴스 생성 시에 실행되어야 할 작업이나 초기값을 주기 위해서 사용된다.
인스턴스를 생성할 때 파라미터로 값을 넘겨 받을 수 있다.   

class Enemy{
    hp:number;
    power:number;
    type:string;
    constructor(hp:number, power:number, type:string){
        this.hp=hp;
        this.power=power;
        this.type=type;
        console.log("적군 초기화");
    }
}
const lv1Enemy = new Enemy(30,2, '몬스터'); // "적군 초기화"

클래스의 멤버변수에 접근 하기위해선 this를 사용한다.

## 접근 제어자
Typescript엔 세가지의 접근제어자가 있다. 묵시적 타입은 public이다.

public : 모든 접근 허용
protected : 상속받은 클래스내에서만 접근 허용
private : 클래스 내에서만 접근 허용

class Enemy{
    public hp:number;
    protected power:number;
    private type:string;
    private range:number;
    constructor(hp:number, power:number, type:string){
        this.hp=hp;
        this.power=power;
        this.type=type;
    }
    public attack(){
        console.log(`${this.type}으로 부터 ${this.power} 만큼의 공격을 받았습니다!`);
    }
    private run(){
        console.log("도망감");
    }
}
const lv1Enemy = new Enemy(30,2,'몬스터');
console.log(lv1Enemy.hp) // 30
console.log(lv1Enemy.power) // error : 접근제한 에러
console.log(lv1Enemy.type) // error : 접근제한 에러

class Boss extends Enemy{
    //멤버변수 생략
    constructor(hp:number, power:number, type:string, ultimatePower:number){
        super(hp, power, type);
        super.attack(); "type 으로 부터 power 만큼의 공격을 받았습니다!"
        super.run(); // error : 접근제한 에러
        
    }
}


### 상속
부모클래스로부터 상속을 받아 자식 클래스에서 부모의 멤버를 사용할 수 있다.
부모클래스의 생성자 매개변수는 super()로 전달할 수 있다.
class Enemy{
    private hp:number;
    private power:number;
    private type:string;
    private range:number;
    constructor(hp:number, power:number, type:string){
        this.hp=hp;
        this.power=power;
        this.type=type;
    }
    public attack(){
        console.log(`${this.type}으로 부터 ${this.power} 만큼의 공격을 받았습니다!`);
    }
}

class Boss extends Enemy{
//멤버변수 생략
    constructor(hp:number, power:number, type:string, ultimatePower:number){
        super(hp, power, type);
        this.ultimatePower = ultimatePower;
    }
    public ultimateSkill(){ //보스만의 새로운 행위
        console.log('보스 궁극기 사용! 데미지 : ',this.ultimatePower);
    }
}
const boss = new Boss(1000,100, '보스', 300);
boss.attack(); // "보스로 부터 100 만큼의 공격을 받았습니다!"
boss.ultimateSkill(); // "보스 궁극기 사용! 데미지 : 300"

위와 같이 공통적인 행동은 Enemy 클래스에 보스만의 특별한 행동은 Boss 클래스에 정의하여
Enemy 클래스를 상속받아 Boss 클래스는 부모인 Enemy 클래스에 정의된 attack 메소드를 중복으로 정의할 필요 없이 사용할 수 있습니다.
(private로 선언된 메소드는 사용 X)

### getter setter



### 기타


