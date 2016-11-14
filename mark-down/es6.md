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
# export
# import
# iterable protocol
# getter, setter
# Method definitions
