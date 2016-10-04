

    
##1. MVP model ##


 어플리케이션은 여러 기능들의 유기적인 집합이라고 볼 수 있다.  이런 기능들의 구현을 적절히 분배하는 것은 협업 프로세스를 체계화하여 일관성을 확보하고, 코드 내 문제 발견과 해결 그리고 추가 기능 구현을 효율화하는데 매우 핵심적인 요소이다.

이와 같은 분배를 효율적으로 하기 위해서는, 기능 단위로 책임을 분배하고 각 부분에서 월권행위를 하지 않을 수 있도록 여러 장치를 마련해 주는 것이 좋다.

    
이러한 효율적인 분배에 대한 설계의 노하우가 축적된 것이 MVC, MVVM, MVP 과 같은 **디자인패턴**[^designPattern]인데, 우리는 MapSee 어플리케이션의 구현을 위해 MVP를 이용할 것이다.




####**1)  MVP 패턴이란?**

프로그램이 동작하는 과정에서, 프로그램은 user-interface 상에서 유저의 커맨드와 input을 입력받고, 이를 저장하거나 처리한 뒤 다시 user 에게 결과를 알려준다.
이러한 동작 프로세스를 크게 세가지 부분으로 분리할 수 있는데,   MVP가 제안하는 분리는 바로 **모델, 뷰, 프레젠터**이다.  이 파트에서는 각각에 대해 먼저 개념적으로 알아보도록 하자.  각 개념들이 쉽게 이해되지 않는다고 걱정하지 말자. 회원 가입 안드로이드 프로그램 예제를 통해 개념들을 더욱 쉽게 이해 할 수 있도록 할 것이기 때문에 부담없이 읽어보길 바란다.

>   **모델 (Model)**
> 
> : 모델은 유저 인터페이스를 구현하는 데 참고할 데이터를 정의하는 인터페이스이다. (_The model is an interface defining the data to be displayed or otherwise acted upon in the user interface._) 쉽게 풀어쓰자면, 모델은 유저의 정보를 포함한 다양한 정보들을 실세계의 로직에 따라 관리하기 위한 프로그램의 한 모듈이라고 생각해도 좋다.    
> 
> **뷰(View)**
>
> : 뷰는 모델을 보여주고, 유저에 의해 발생한 이벤트들을 프레젠터에게 라우팅 시켜주는 인터페이스이다. (_The view is a passive interface that displays data (the model) and routes user commands (events) to the presenter to act upon that data._) 이 역시 쉽게 풀어쓰자면, 뷰는 스마트폰에서 우리가 터치하거나 키보드를 통해 글을 입력하는 UI를 제공하고, 여기서 이벤트(터치, 혹은 글자 변동)가 발생했을 때 프레젠터에게 이벤트 발생을 알려주는 역할을 하는 모듈이다.
>
>**프레젠터(Presenter)**

>: 프레젠터는 모델과 뷰에 따라 행동하며, 모델에서 데이터를 인출하여 뷰에게  이를 전달함으로써, UI를 변화시키도록 지시한다. (_presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view._) 즉, 프레젠터는 모델과 뷰 사이에 있는 중간 관리인이라고 할 수 있으며, 정보와 커맨드를 적절히 상호 간에 전달한다. MVP모델에서는 Presenter은 view instance를 갖고 있고, View 역시 그  presenter instance를 갖고 있어 1:1 대응 관계를 이룬다.
>
> ![enter image description here](https://upload.wikimedia.org/wikipedia/commons/d/dc/Model_View_Presenter_GUI_Design_Pattern.png)
> 영문으로 된 내용은 Model–view–presenter, Wikipedia 에서 발췌한 것이다.[^MVP]

이 개념들을 읽다보면 자연스럽게 드는 의문은 'Model, View, Presenter 는 안드로이드 코드에서 어떤 부분에 해당하는가?'일 것이다. View가 Activity 인지, Model 이 하나의 Class를 말하는 것인지 이제 회원가입 안드로이드 프로그램 예제를 통해 MVP 디자인 패턴을 이해해보도록 하자.

####**2) MVP 패턴 예제  - 회원 가입 프로그램**

**2.1) MVP에서 interface 의 사용**

먼저 예제를 보기 전에, interface라는 중요한 주제에 대해 짚고 넘어가자.
interface는 무엇이며, 우리는 왜 MVP 모델에서 interface를 쓰는가?

먼저 interface란 우리가 구현할 클래스에 대한 약속이다.  interface에서는, 우리가 구현할 클래스가 갖기로 약속된 메소드들을 선언할 수 있다.  다시 말해 클래스내 메소드의 접근제어자와 반환형,이름, 인자 타입들을 명시적으로 선언할 수 있다.

```java
public interface SignUpView{
	void enableSignUpButton(boolean isSignUpEnabled);
}
```
위에서 보는 바와 같이 SignUpView라는 인터페이스를 구현한 클래스는 enableSignUpButton라는 메소드를 구현하기로 약속이 되어있다.


```java
public class SignUpActivity extends AppCompatActivity implements SignUpView {

...

  @Override
    public void enableSignUpButton(boolean isSignUpEnabled) {
         Button sign_up_button = (Button)findViewById(R.id.sign_up_button);
        sign_up_button.setEnabled(isSignUpEnabled);
	}
}
```
위와 같이 signUpView를 implement한 SignUpActivity 에서는, signUpButtonEnable 메소드, 즉 signUp 버튼을 클릭 가능한 형태로 만들어주는 함수가 구현되기로 약속되어 있기 때문에 이를 반드시 구현해야한다.

그렇다면 MVP모델에서는 왜 이러한 약속( 여기서 인터페이스)이 필요할까? 먼저 인터페이스라는 이름 속에 답이 있다. 인터페이스라는 것은 서로 다른 두 시스템이 연결되는 곳이다. 그 연결이 제대로 된 기능을 하기 위해서, 우리는 여러 약속을 정하게 된다. 이는 한 컴퓨터와 다른 단말사이의 원격 통신을 하기 위해, 주고 받을 자료의 형식과 오류 검출 방식 등을 정하는 프로토콜을 정하는 것과 비슷하다.

위의 예제코드로부터 좀 더 실질적인 통찰을 얻어보자. 만약 SignUpActivity 외부에서,  SignUpActivity 내의 sign_up 버튼이 눌러질 수 있는 상태가 되기를 바란다고 하자. 그런데 이러한 인터페이스가 없다면, 코드를 일일히 읽기 전까지 SignUpActivity 가  sign up 버튼을 enable하는 메소드를 구현하기는 했는지, 했다고 추측한다해도 어떤 메소드 명으로 어떤 인자들을 주어 그 메소드를 호출해야하는지 도무지 알 수가 없다. 

그렇지만 만약,  signUpActivity가 signUpView 를 implement했다는 사실을 알게 된다면? 그 때엔 signUpActivity 코드를 꼼꼼히 읽지 않아도, interface만 확인함으로써 외부에서 SignUpActivity의 enableSignUpButton(true) 를 제대로 호출할 수 있다.

이와같이 인터페이스에 정의된 약속이 빛을 발하는 또 다른 상황은 메소드 구현이 되지 않은 상황에서도 마치 구현 된 것처럼 코드를 짤 수 있을 때이다. 이 상황은 다음과 같이 SignUpPresenter 코드에서 확인할 수 있다.

``` java
public class SignUpPresenter {

    private SignUpView signUpView;
    
    public void updateID(String ID) {
       if(ID.length > 5) //적합한 ID인지 판별
       {signUpView.enableSignUpButton(true) }
       else{signUpView.enableSignUpButton(false);}
}
```

 위 코드에서 볼 수 있는 바와 같이 비록 아직 enableSignUpButton()이 구현되지 않은 상태라고 하더라도, 이와 같이 Presenter의 로직을 완성할 수 있다.  **다시말해 enableSignUpButton()  구현여부와  구현방식에 상관없이 Presenter의 로직에만 집중해서 개발할 수 있다.** 

두번째로,  각 interface 를 사용하면 클래스의 월권행위를 방지할 수 있다. 다시말해,  사용하기로 약속된 메소드만 사용할 수 있도록 할 수 있다. 

이를 위해서는 interface 참조변수가 그 interface를 구현한 클래스의 객체를 가리킬 수 있다는 사실을 먼저 이해해야한다.
위의 코드에서 생성자 부분을  유의해서 다시 보도록 하자. 
``` java
public class SignUpPresenter {

    private SignUpView signUpView;
    
    public SignUpPresenter(SignUpView signUpView) {
        this.signUpView = signUpView;
    }
    public void updateID(String ID) {
       if(ID.length > 5) //적합한 ID인지 판별
       {signUpView.enableSignUpButton(true) }
       else{signUpView.enableSignUpButton(false);}
}
```
보는 바와 같이 생성자에 SignUpView 참조변수를 인자로 받아, this.signUpview에 이를 연결하고 있다. 실제로 어떤 변수가 이 인자로 올까? SignUpPresenter(SignUpView signUpView) 의 형태 때문에 인자에는 interface 의 객체가 올 것 같지만, interface 의 객체를 만드는 것은 불가능하다. 답은 아래 코드에서 확인해보도록 하자. 

```java
public class SignUpActivity extends AppCompatActivity implements SignUpView {

...
   @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        signUpPresenter = new SignUpPresenter(this);
        
    }
}
```
답을 발견했는가? 답은 signUpPresenter = new SignUpPresenter(this);에서  확인할 수 있다. 여기서는 그 interface 를 구현한 SignUpActivity 의 객체 자신의 reference 가  인자로 가서, signUpView의 참조변수가 이를 가리킬 수 있게 된다. 




 [^designPattern]: [design Pattern](https://stackedit.io/)  소프트웨어 개발 방법에서 사용되는 디자인 패턴은, 프로그램 개발에서 자주 나타나는 과제를 해결하기 위한 방법 중 하나로, 과거의 소프트웨어 개발 과정에서 발견된 설계의 노하우를 축적하여 이름을 붙여, 이후에 재이용하기 좋은 형태로 특정의 규약을 묶어서 정리한 것이다. - 위키피디아.

