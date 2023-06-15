## 3.1 상황에 따라 달라지는 this

- 자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정된다.
- 실행 컨텍스트는 함수를 호출할 떄 생성되므로, 바꿔 말하면 **this는 함수를 호출할 떄 결정**된다고 할 수 있다.

### 3.1.1 전역 공간에서의 this

```jsx
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

- 전역공간에서의 this는 전역객체를 의미하므로 두 값이 같은 값을 출력하는 것은 당연하지만, 그 값이 1인 것이 의아하다. 그 이유는 **자바스크립트의 모든 변수는 실은 특정 객체의 프로퍼티**로서 동작하기 때문이다.
- **전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당한다.**

### 3.1.2 메서드로서 호출할 떄 크 메서드 내부에서의 this

**함수 vs. 메서드**

- 프로그래밍 언어에서 함수와 메서드는 미리 정의한 동작을 수행하는 코드 뭉치로, 이 둘을 구분하는 유일한 차이는 **독립성**에 있다.
- 함수는 그 자체로 독립적인 기능을 수행하는 반면 메서드는 자신을 호출한 대상 객체에 관한 동작을 수행한다.
- 객체의 메서드로서 호출할 경우에만 메서드로 동작하고, 그렇지 않으면 함수로 동작한다.

```jsx
var func = function (x) {
  console.log(this.x);
};
func(1); // Windew {...} 1

var obj = {
  method: func,
};
obj.method(2); // { method: f } 2
```

- 1번째 줄에서 func라는 변수에 익명함수를 할당했습니다. 4번째 줄에서 func를 호출했더니 this로 전역객체 Window가 출력됩니다.
- 9번째 줄에서 obj의 method를 호출했더니, 이번에는 this가 obj라고 한다.
- 원래의 익명함수는 그대로인데 이를 변수에 담아 호출한 경우와 obj객체의 프로퍼티에 할당해서 호출한 경우에 this가 달라진다.
- 함수 앞에 점(.)이 있는지 여부만으로 함수호출과 메서드호출을 구분할 수 있다.

**메서드 내부에서 this**

- 어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명 앞의 객체이다. 점 표기법의 경우 마지막 점 앞에 명시된 객체가 곧 this가 된다.

### 3.1.3 함수로서 호출할 때 그 함수 내부에서의 this

**함수 내부에서의 this**

- this에는 호출한 주체에 대한 정보가 담긴다고 했습니다. 그런데 함수로서 호출하는 것은 호출 주체를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없다.
- 함수에서의 this는 전역 객체를 가리킨다.

**메서드의 내부 함수에서의 this**

```jsx
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this); // 2, 3
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

- this가 각각 가리키는 것은 (1):obj, (2): 전역객체(Window), (3): obj입니다. 코드 흐름을 생각해며 분석해 보자!
- this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경은 중요하지 않고, 오직 해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기가 있는지 없는지가 관건인 것입니다.

**메서드이 내부 함수에서의 this를 우회하는 방법**

```jsx
var obj = {
  outer: function () {
    console.log(this); // 1.{ outer: f }
    var innerFunc1 = function () {
      console.log(this); // 2. Window {...}
    };
    innerFunc1();

    var self = this;
    var innerFinc2 = function () {
      console.log(self); // 3. { outer: f }
    };
    innerFunc2();
  },
};
obj.outer();
```

**this를 바인딩하지 않는 함수**

- ES6에서 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자, this 바인딩하지 않는 화살표 함수를 새로 도입했습니다.
- 화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 스코프의 this를 그대로 활용할 수 있다. 내부함수를 화살표 함수를 바꾸면 우회법이 불필요해진다.
- 그 밖에도 call, apply등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있다.

### 3.1.4 콜백 함수 호출 시 그 함수 내부에서의 this

```jsx
setTimeout(function () {
  console.log(this);
}, 300);

[1, 2, 3, 4, 5].forEach(function (x) {
  console.log(this, x);
});

document.body.innerHTML += '<button id = "a">클릭</button>';
document.body.quertSelector("#a").addEventListener("click", function (e) {
  console.log(this, e);
});
```

- 콜백 함수의 제어권을 가지는 함수(메서드)가 콜백 함수에서의 this를 무엇으로 할지를 결정하며, 특별히 정의하지 않은 경우에는 기본적으로 함수와 마찬가지로 전역객체를 바라본다

### 3.1.5 생성자 함수 내부에서의 this
