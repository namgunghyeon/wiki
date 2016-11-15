# Class
Class는 사실 함수.
함수 선언과 클래스 선언의 중요한 차이점은 함수 선언의 경우 호이스팅이 일어나지만, 클래스 선선은 그렇지 않다는 것
다른 언어에 있는 class 형태와 같아 이해하는데 어렵운 없어 보인다.

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions
```javascript
class Polygon {
    constructor(height, width) {
        this.height = height;
        this.width = width;
    }
    get area() {
        return this.calcArea();
    }

    calcArea() {
        return this.height * this.width;
    }
}

var polygon = new Polygon(1, 2);
console.log('area : ', polygon.area);
console.log('area : ', polygon.calcArea);
console.log('area : ', polygon.calcArea());

```

### Static methods
```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    static distance(a, b) {
        const dx = a.x - b.y;
        const dy = a.y - b.y;

        return Math.sqrt(dx * dx + dy * dy);
    }
}
var aPoint = new Point(1, 2);
var bPoint = new Point(2, 1);
console.log('distance : ', Point.distance(aPoint, bPoint));

```

### Extends
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    speak() {
        console.log(this.name + ' make a nose');
    }
}

class Dog extends Animal {
    speak() {
        console.log(this.name + ' barks.');
    }
}

var dog = new Dog('choco');
dog.speak();
```

### Supper
```javascript
class Cat {
    constructor(name) {
        this.name = name;
    }
    speak() {
        console.log(this.name + ' make a nose');
    }
}

class Lion extends Cat {
    speak() {
        super.speak();
        console.log(this.name + ' roars.');
    }
}
var lion = new Lion('king');
lion.speak()
```

### Species
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol/species
``` javascript
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

var array = new MyArray(1,2,3);
var mapped = array.map(x => {
    return x * x
});
console.log(mapped)
console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array);   // true
```



# Arrow Function Expression
(화살표 함수 표현)
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98
자신의 this, arguments, super 또는 new.target을 바인딩 하지 않는다
```var that = this```을 하지 않아도 된다. 화살표 함수는 항상 익명이다.

```javascript
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression
// 다음과 동일함:  => { return expression; }

// 매개변수가 하나뿐인 경우 괄호는 선택사항:
(singleParam) => { statements }
singleParam => { statements }

// 매개변수가 없는 함수는 괄호가 필요:
() => { statements }
```

```javascript
var a = [
  "Hydrogen",
  "Helium",
  "Lithium",
  "Beryl­lium"
];

var a2 = a.map(function(s) {
    return s.length;
});
var a3 = a.map(s => s.length);
console.log(a2);
console.log(a3);

```

```javascript
function Person() {
    this.age = 0;
    setInterval(function growUp() {
        //setInterval 위에 선언된 this와 아래 선언된 this는 서로 다른 this
        this.age++;
   }, 1000);
}

function Person2() {
    var that = this;
    this.age = 0;
    setInterval(function growUp() {
        that.age++;
   }, 1000);
}

function Person3(){
  this.age = 0;
  setInterval(() => {
      //setInterval 위에 선언된 this와 아래 선언된 this는 같은 this
      this.age++;
  }, 1000);
}
```

```javascript
 var func = () => ({ foo: 1 });
```

# let
현재 scope에서만 유효한 let 변수
```javascript
function varTest() {
   var x = 1;
   if (true) {
       var x = 2;  // 같은 변수
       console.log(x);
   }
   console.log(x);
}

function letTest() {
   let x = 1;
   if (true) {
       let x = 2;  // 다른 변수
       console.log(x);
   }
   console.log(x);
}
console.log('varTest')
varTest()
console.log('letTest')
letTest()

```

# Const
Read Only refrence to a value
``` javascript
const MY_FAV = 7;

MY_FAV = 20;
console.log('MY_FAV', MY_FAV);

const MY_FAV = 20; //Error

let MY_FAV = 20; //Error

console.log('MY_FAV', MY_FAV);

```

# export
파일 또는 함수에서 외부로 노출할 때 사용
아직 어느곳에서 구현되어 있지 않아 Babel를 사용해야 한다.
```javascript
export { name1, name2, …, nameN };
export { variable1 as name1, variable2 as name2, …, nameN };
export let name1, name2, …, nameN; // 또는 var
export let name1 = …, name2 = …, …, nameN; // 또는 var, const

export default expression;
export default function (…) { … } // 또는 class, function*
export default function name1(…) { … } // 또는 class, function*
export { name1 as default, … };

export * from …;
export { name1, name2, …, nameN } from …;
export { import1 as name1, import2 as name2, …, nameN } from …;

//module "my_module.js"
export function cube(x) {
    return x * x * x;
}

const foo = Math.PI + Math.SQRT2;
export { foo }


//moduel demo.js
import { cube, foo} from 'my_module.js';

console.log(cube(3))
console.log(foo);

```

# import
export로 노출하면 import로 다른 곳에서 사용
import도 아직 어느곳에서 구현되어 있지 않아 Babel를 사용해야 한다.(Phyton import 와 같은 느낌)

``` javascript
import name from "module-name";
import * as name from "module-name";
import { member } from "module-name";
import { member as alias } from "module-name";
import { member1 , member2 } from "module-name";
import { member1 , member2 as alias2 , [...] } from "module-name";
import defaultMember, { member [ , [...] ] } from "module-name";
import defaultMember, * as alias from "module-name";
import defaultMember from "module-name";
import "module-name";

import * as myModule from "my-module.js";
모듈 전체를 가져온다. 그리고 myModule이름으로 바인딩되어 사용한다.

import {myMember} from "my-module.js";
모듈 내에서 myMember 하나만 가져온다.

import {foo, bar} from "my-module.js";
모듈 내에서 foo, bar를 가져온다.

import {reallyReallyLongModuleMemberName as shortName} from "my-module.js";
reallyReallyLongModuleMemberName라는 변수를 shortName으로 별명을 지어 사용한다. (mysql AS와 같은 느낌)
```

# function*
함수 끝에 *가 붙을 경우 Generator 객체를 반환한다.
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Generator
yield 만큼 next를 사용할 수 있다.

``` javascript

function* name([param[, param[, ... param]]]) {
   statements
}

function* idMaker() {
    var index = 0;
    while (index < 3) {
        yield index ++;
    }
}

var gen = idMaker()

console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);
//0
//1
//2
//undefined

function* anotherGenerator(i) {
    yield i + 1;
    yield i + 2;
    yield i + 3;
    console.log(i)
}

function* generator(i) {
    yield i;
    yield* anotherGenerator(i);
    yield i + 10;
}

var gen = generator(10);
console.log(gen.next().value);
//generator 아래 yield
console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);
//위 3개는 anotherGenerator에 있는 yield
console.log(gen.next().value);
//마지막 yield
console.log(gen.next().value);


function* logGenerator() {
    console.log(yield);
    console.log(yield);
    console.log(yield);
}

var gen = logGenerator();

gen.next();
gen.next('pretzel');
gen.next('california');
gen.next('mayonnaise');

```
# for ... of
(Array, Map, Set, String, TypedArray, arguments)
``` javascript
"use strict";
let iterable = [10, 20, 30];

for (let value of iterable) {
   console.log(value);
}

for (const value of iterable) {
   console.log(value);
}

iterable = "boo";
for (let value of iterable) {
   console.log(value);
}

iterable = new Uint8Array([0x00, 0xff]);

for (let value of iterable) {
   console.log(value);
}

iterable = new Map([["a", 1], ["b", 2], ["c", 3]]);

for (let entry of iterable) {
   console.log(entry);
}

iterable = new Set([1, 1, 2, 2, 3, 3]);
//중복 없는 데이터 셋
for (let value of iterable) {
 console.log(value);
}

```
