---
layout: post
title:  "Typescript Class"
date:   2023-01-27 20:15:00 +0900
author: ybcho
categories: Typescript Class 객체지향
---
<hr/>

## Typescript Class
Typescript는 Javascript를 객체지향적 프로그램 기법을 사용할 수 있게 해줍니다.  
그 중 class를 설명 드리겠습니다. Typescript의 class는 기존 객체지향언어인 java, C#의 class와 상당히 비슷하여  
사용해보셨다면 익숙한 부분을 많이 느끼실 수 있습니다.

## Class란?
class(클래스)란 객체 지향 프로그래밍에서 객체를 만들기 위한 틀,  
객체의 상태(변수들)와 행동(메소드)가 정의된 곳

ex)
붕어빵틀에 재료와 행동에 따라 다른 결과물의 붕어빵(팥붕, 슈붕)을 만든다.  
붕어빵틀 -> 클래스  
굽다 -> 메소드  
붕어빵 -> 객체  

예제 코드는 게임을 가정하여 작성하였습니다.

- 선언
```tsx
class Enemy{
    //...멤버변수
    constructor(매개변수){
    }
    //...멤버들
}
```
- 생성
```tsx
const enemy:Enemy = new Enemy();
```
## 생성자
인스턴스가 생성될 때 호출되는 초기화 메소드   
인스턴스 생성 시에 실행되어야 할 작업이나 초기값을 주기 위해서 사용된다.  
인스턴스를 생성할 때 파라미터로 값을 넘겨 받을 수 있다.   

```tsx
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
const lv1Enemy = new Enemy(30,2, '몬스터'); // 생성과 동시에 생성자 실행 됨 "적군 초기화"
```
클래스의 멤버에 접근 하기위해선 `this`를 사용한다.

## 접근 제어자
Typescript엔 세가지의 접근제어자가 있다. 묵시적 타입은 `public`이다.

`public` : 모든 접근 허용  
`protected` : 상속받은 클래스내에서만 접근 허용  
`private` : 클래스 내에서만 접근 허용
```tsx
class Enemy{
    public hp:number;
    protected power:number;
    private _type:string;
    private _range:number;
    constructor(hp:number, power:number, type:string){
        this.hp=hp;
        this.power=_power;
        this.type=type;
    }
    public attack(){
        console.log(`${this._type}으로 부터 ${this.power} 만큼의 공격을 받았습니다!`);
    }
    private _run(){
        console.log("도망감");
    }
}

const lv1Enemy = new Enemy(30,2,'몬스터');
console.log(lv1Enemy.hp) // 30
console.log(lv1Enemy.power) // error : 접근제한 에러
console.log(lv1Enemy._type) // error : 접근제한 에러

class Boss extends Enemy{
    //멤버변수 생략
    constructor(hp:number, power:number, type:string, ultimatePower:number){
        super(hp, power, type);
        super.attack(); //"type 으로 부터 power 만큼의 공격을 받았습니다!"
        super._run(); // error : 접근제한 에러
        
    }
}
```
TypeScript 상에서는 접근 제어자에 따라 접근 또는 차단을 제어할 수 있지만,컴파일된 JavaScript는 접근 제어자를 지원하지 않는다.  
때문에 실행시 `private`로 선언된 멤버도 호출 되어 실행된다.

## 상속
부모클래스로부터 상속을 받아 자식 클래스에서 부모의 멤버를 사용할 수 있다.
부모클래스의 생성자 매개변수는 `super()`로 전달할 수 있다.
```tsx
class Enemy{
    private _hp:number;
    private _power:number;
    private _type:string;
    private _range:number;
    constructor(hp:number, power:number, type:string){
        this._hp=hp;
        this._power=power;
        this._type=type;
    }
    public attack(){
        console.log(`${this._type}으로 부터 ${this._power} 만큼의 공격을 받았습니다!`);
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
```

위와 같이 공통적인 행동은 Enemy 클래스에 보스만의 특별한 행동은 Boss 클래스에 정의하여  
Enemy 클래스를 상속받아 Boss 클래스는 부모인 Enemy 클래스에 정의된 attack 메소드를  
중복으로 정의할 필요 없이 사용할 수 있습니다. (`private`로 선언된 메소드는 사용 X)

## getter setter
TypeScript는 객체의 멤버에 대한 접근을 getter/setter로 접근할 수 있다.  
이를 통해 각 객체의 멤버에 접근하는 방법을 세밀하게 제어할 수도 있다.

```tsx
class Enemy{
    private _hp:number;
    private _power:number;
    private _type:string;
    private _range:number;
    constructor(hp:number, power:number, type:string, range:number){
        this._hp=hp;
        this._power=power;
        this._type=type;
        this._range=range;
    }
    public attack(){
        console.log(`${this._type}으로 부터 ${this._power} 만큼의 공격을 받았습니다!`);
    }
    set range(range:number){
        if(range<100) this._range = range;
    }
    get range(){
        return this._range;
    }
}

class Boss extends Enemy{
    //멤버변수 생략
    constructor(hp:number, power:number, type:string, ultimatePower:number){
        super(hp, power, type);
        this.ultimatePower = ultimatePower;
    }
        public ultimateSkill(){ //보스만의 새로운 행위
            super.range = 10;
            console.log('보스 궁극기 사용! 데미지 : ',this._ultimatePower, "사정거리 : ", super.range);
    }
}
const boss = new Boss(1000, 100, '보스', 300, 5);
boss.attack(); // "보스로 부터 100 만큼의 공격을 받았습니다!"
boss.ultimateSkill(); // "보스 궁극기 사용! 데미지 : 300 사정거리 : 10"

boss.range = 99;
console.log(boss.range) // 99
```

## 추상클래스
일부 메소드가 구현되지 않고 선언만 되어있는 클래스로 `abstract`가 선언된 메소드는 자식 클래스에서 반드시 구현되어야 한다.

```tsx
abstract class Character {
    protected constructor(public name: string) {
    }
    printName(): void {
        console.log("캐릭터 아이디 : " + this.name);
    }
    abstract move(): void
    protected abstract attack():void;
    abstract skill():void
}
class Wizard extends Character {
    constructor(name:string) {
        super(name);
    }
    move(): void {
        console.log("캐릭터가 이동합니다.");
    }
    attack(): void {
        console.log("캐릭터가 공격합니다.");
    }
    skill():void{
        console.log("얼음공격을 사용합니다.");
    }
}
class Warrior extends Character {
    constructor(name:string) {
        super(name);
    }
    move(): void {
        console.log("캐릭터가 이동합니다.");
    }
    attack(): void { 
        console.log("캐릭터가 공격합니다.");
    }
    /* skill():void {
        console.log("핵주먹을 사용합니다.");
    }*/ // error 추상클래스에 정의된 abstract메소드는 반드시 정의 해야함
}
const wizard = new Wizard("이은결");
wizard.printName(); // "캐릭터 아이디 : 이은결"
wizard.move(); // "캐릭터가 이동합니다."
wizard.attack(); // "캐릭터가 공격합니다."
wizard.skill(); //"얼음공격을 사용합니다."
const warrior = new Warrior("정찬성")
warrior.printName(); // "캐릭터 아이디 : 이은결"
warrior.move(); // "캐릭터가 이동합니다."
warrior.attack(); // "캐릭터가 공격합니다."
warrior.skill(); //"핵주먹을 사용합니다."
```

참고 사이트   
[https://typescript-kr.github.io/pages/classes.html](https://typescript-kr.github.io/pages/classes.html)  
[https://velog.io/@jun094/TypeScript-Class](https://velog.io/@jun094/TypeScript-Class)
