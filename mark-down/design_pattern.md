## MVC(Model - View - Controller)
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

 View의 업데이트 방법
 - View가 Model을 직접 사용하여 업데이트
 - Model에서 View에게 Notify해 주는 방법(Observer Interface)
 - View에서 Polling하여 Model의 변화를 감지하는 방법

 View와 Model간의 의존성이 낮으면 낮을 수록 좋은 구성이며 View와 Model간의 의존성을 완전히 없앨 수 없다는 한계를 가지고 있다.


## MVP

## MVVM
