## MVC(Model - View - Controller)

![mvc](https://raw.githubusercontent.com/namgunghyeon/wiki/2a0a02e898426190eca765c853fc8fe2c450db21/images/design_pattern/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-22%20%EC%98%A4%EC%A0%84%2012.50.13.png)

 - 프로그램을 구성하는 요소들을 Model, View, Controller 세 가지로 나누어 각각 역활을 부여하고, 각자의 역할에만 충실해 결합도를 줄이는 패턴
 - Model
   - 데이터를 가진 객체
 - View
   - 사용자에게 제공되는 UI
 - Controller
   - Clinet에게 입력을 받아 해당하는 Model을 선택해 입력을 처리하고, 결과에 따라 View를 선택하여 Model과 View를 연결한다.

 **모든 입력은 Controller가 받아서 처리한다.**

 입력이 들어오면 Controller는 입력에 해당하는 Model를 업데이트하고 Model을 표현해 줄 View를 선택한다.
 하나의 Controller는 여러개의 View를 가질 수 있으며 Controller는 여러개의 View를 선택하여 Model을 표현할 수 있다. 반대로 View는 Controller에서 어떤 동작이 일어나는지 알 수 없다.

 **View의 업데이트 방법**
```
 - View가 Model을 직접 사용하여 업데이트
 - Model에서 View에게 Notify해 주는 방법(Observer Interface)
 - View에서 Polling하여 Model의 변화를 감지하는 방법
```
 View와 Model간의 의존성이 낮으면 낮을 수록 좋은 구성이며 View와 Model간의 의존성을 완전히 없앨 수 없다는 한계를 가지고 있다.


## MVP(Model - View - Presenter)
![mvp](https://raw.githubusercontent.com/namgunghyeon/wiki/0bcf5f0b6a01fc69ddba7f6da3b063fbb72bacec/images/design_pattern/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-22%20%EC%98%A4%ED%9B%84%2010.01.45.png)

MVC모델에서 파생된 아키텍쳐로 MVC에서 Controller를 Presenter로 변경해 Model과 View의 의존도를 없애기 위해 등장

- Model
  - 데이터를 가진 객체

- View
  - UI Layer

- Presenter
  - 사용자의 이벤트에 대한 로직을 처리하고 Model을 관리하며 Model의 상태 변화를 View에게 알려는 역활도 수행한다.

**Presenter는 Model과 View의 상호 작용을 담당한다.**

```
Client에서 들어오는 입력은 View가 받아서 처리하고 이벤트가 들어왔을 때 View는 이벤트를 Presenter에게 전달한다.
Presenter는 이벤트를 바탕으로 Model을 조작하고, 그 결과를 다시 View에게 바인딩하면, View의 각 요소들이 업데이트 되는 구조이다.
```

 Model과 View의 의존성은 사라졌지만 Presenter와 View 간의 1:1관계로 둘의 의존성이 매우 강해진다는 문제가 있다.


## MVVM(Mode - View - ViewModel)
![mvvm](https://github.com/namgunghyeon/wiki/blob/master/images/design_pattern/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202016-11-22%20%EC%98%A4%ED%9B%84%2010.00.13.png?raw=true)

Model과 View 사이의 의존성 뿐만 아니라 View와 Controller간의 의존성도 고려해 Layer가 완전히 독립적으로 작성되고 테스트 될 수 있도록 설계된 패턴

- Model
  - 데이터를 가진 객체

- View
  - UI Layer

- View Model
  - View의 표현을 담당한다.


ViewModel은 View를 나타내기 위한 Model이며, View의 Presentation logic을 담당한다.
MVP에서 Presenter가 View와 의존 관계에 있었다면 view Model은 View와 독립적으로 이루어져 있다는 점이다.

각가의 View는 자신이 사용할 ViewModel을 선택할 수 있으며, ViewModel은 Model을 베이스로 Presentation Logic에 따라 서로 다르게 구현된다.
View의 어떤 ViewModel을 연결하느냐에 따라 로직 처리가 달라지고, 각 로직 처리에 따라 Model이 변경되면 해당 ViewModel을 사용하는 View가 자동으로 업데이트 된다.




예제코드
MVC
http://theholmesoffice.com/getting-ready-for-scalability-creating-an-mvc-framework-for-our-node-js-page/



출처:
http://coding-dragon.tistory.com/4
http://geekswithblogs.net/dlussier/archive/2009/11/21/136454.aspx
