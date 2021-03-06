>Contents
>
1. Android Threads   
    1.1. UI, Binder, Background Thread  
    1.2. Blocking UI Thread and ANR  
>
2. MultiThreading  
    2.1. Basic Usage  
    2.2. Multithreading Considerations  
    2.3. Thread Safety  
    2.4. Task Execution Strategies  
    2.5. MultiThreading on Android : UI Thread Only  
>
3. Thread Communication in Java  
    3.1. Pipes  
    3.2. Shared Memory  
    3.3. Blocking Queue  
>
4. Thread Communication in Android  
    4.1. Android Message handling Mechansim  
    4.2. MessageQueue
    4.3. Message
    4.4. Looper  
    4.5. Handler  
    4.6. HandlerThread  
>
5. Asynctask  
    5.1. Overall  
    5.2. Usage  
    5.3. Pitfalls  
>
6. Executor
>
7. IntentService
>
8. Loader
>
9. Select MultiThreading Method

---

안드로이드 애플리케이션을 개발하다 보면 비동기 작업이나 UI 반응을 위해 스레드를 사용할 일이 많이 생긴다. 안드로이드에서는 멀티스레딩을 위해 몇 가지 방법을 제공하고 있으며, API 문서만 읽고도 손쉽게 적용할 수 있게 되어 있으나 자세하게 정리하고자 이 글을 작성하였다.

### 1. Android Threads

안드로이드 애플리케이션이 시작되는 과정을 살펴보면, Linux Process를 만들고, 그 위에 Dalvik VM runtime을 띄워 Application Class의 Instance를 생성, Instance가 가지고 있는 Entry Point Component를 실행한다. 이 과정에서 Linux Process와 Dalvik VM을 관리하는 많은 스레드들이 생성되는데, 이 중 UI thread와 Binder thread는 Application에서 사용되며, Application이 자체적으로 Background thread를 만들어 사용할 수도 있다.

#### 1.1. UI, Binder, Background Thread

안드로이드 애플리케이션에서 사용하는 모든 스레드는 Linux native pthread(POSIX Thread)의 자바 구현체이다. 하지만 Android Platform은 역할에 따라 UI, Binder, Background thread로 나누어 각각에 특별한 속성들을 부여했다.

UI Thread는 간단히 말하면 Main thread이다. 애플리케이션이 시작되어 프로세스가 종료될 때까지 유지되며, UI elements에 접근할 수 있는 유일한 스레드이다. (모든 Thread는 Linux native thread로 Linux에서는 동등하게 취급되기 때문에 Application Framework Layer의 WindowManager에서 UI thread 이외의 접근을 제한한다) Activity에서 실행하는 코드들은 별다른 스레딩을 하지 않았다면 UI thread에서 실행된다. UI elements들의 이벤트들을 순차적으로 처리하기 때문에 시간이 오래 걸리는 이벤트를 실행하면 UI 전체가 멈추게 된다.

Binder Thread는 IPC(InterProcess Communication)를 위한 스레드로, 모든 프로세스는 Binder thread를 위한 thread pool을 가진다. 이 스레드풀은 임의로 제거되거나 생성할 수 없으며, Intent, Content Provider, Service 등 다른 프로세스의 요청을 핸들링하게 된다. Binder를 사용할 경우를 제외하고는 생각하지 않아도 되는 thread로, Binder란 Linux IPC를 대체한 안드로이드 커널만의 프레임워크로, 프로세스 간의 RPC(Remote Procedure Calls)를 제공하는 것인데, 추후에 다른 글에서 다루기로 한다.

Background Thread는 애플리케이션이 필요할 때 생성하는 thread로 UI thread의 속성들을 상속받는다. 자유롭게 생성하고 실행이 가능하며, 일반적인 애플리케이션을 개발할 때 가장 신경써야 할 thread이다. 앞으로 다룰 Threading에 관한 내용은 거의 Background Thread에 대한 내용이라고 볼 수 있다.

#### 1.2. Blocking UI Thread and ANR

사용자와 인터렉션하는 GUI 애플리케이션들의 경우, 사용자경험 측면에서 즉각적인 응답을 주는 것이 매우 중요하다. 사용자는 화면이 멈추고 아무런 입력도 할 수 없는 상태가 자주 일어나는 애플리케이션에 결코 좋은 평가를 하지 않을 것이다. 이런 이유로 윈도우의 ‘응답 없음’과 같이, 어떤 이벤트가 발생한 후 일정 시간동안 응답이 없는 경우 OS단에서 앱을 중지할 것인지 물어보는 (과격한) 정책들이 사용되고 있다.

Android에서는 UI Thread가 UI elements들의 이벤트들을 순차적으로 실행하는 까닭에 어떤 element의 이벤트가 오랜 시간 실행된다면 전체 UI가 다른 이벤트를 받지 못하고 멈추게 된다. 이 상태로 약 5초가 지나면 Android OS는 ANR(Application Not Responding) 메시지를 띄우며 사용자가 앱을 종료할 것인지 묻게 된다. 이 때 사용자가 당신의 애플리케이션이 완벽하다는 믿음을 가지고 종료하지 않고 기다려 주는 경우는 사용자가 당신 혹은 당신의 동료일 경우 뿐이다. 대부분의 사용자는 망설이지 않고 종료 버튼을 누를 것이며, 다시는 당신의 애플리케이션을 이용하지 않을 확률이 높다.

이 경우 사용자에게 비현실적인 것을 기대하기보다는 Background Thread를 사용해 비동기적으로 처리해 주고, 기다리는 동안은 처리되고 있다는 알림을 - 필수는 아니지만 - 띄우는 편이 사용자에게 더 큰 만족감을 줄 것이다. 더 나아가 안드로이드는 시간이 많이 걸리는 작업들 - 네트워크 등(더 찾아보기) - 을 Background Thread에서 처리하도록 (찾아봐야댐)버전 몇부터는 정책적으로 강제하였다. 다시 말해 Background Thread을 사용해야 하는 부분은 안드로이드 개발에 있어 빼놓을 수 없는 부분이며, Android MultiThreading에 대한 깊은 이해가 필요하다고 볼 수 있다.

### 2. MultiThreading

안드로이드의 스레드에 대해 더 살펴보기 전에, 기본적인 스레드에 대해 더 알아보자. 스레드란 프로세스 내에서 실행되는 흐름의 단위를 뜻하는 말로, 코드의 실행 흐름을 새로 만들 때 사용된다. 일반적으로 한 프로세스는 하나 이상의 스레드를 가지는데, 한 스레드 안에서 코드는 항상 순차적으로 실행된다.

(TODO 예시 수정할 것)
이해를 돕기 위해 라면을 끓이는 상황을 생각해 보자. 

> 불을 켜고(Task A) 1초에 한 번씩 물이 끓는 지 보다 물이 끓으면(wait) 라면을 넣는다(Task B)는 코드를 한 스레드 안에서 실행하면 불을 켜고(Task A 실행) 가만히 서서 냄비를 바라보다(wait) 라면을 넣을 것이다(Task B 실행).

위의 예시에서 당신은 물이 끓을 때까지 1초에 한번씩 냄비를 확인하며 가만히 서있어야 할 것이다. 다행스럽게도, 당신에게는 Thread라는 로봇이 있다. 불을 켜고(Task A) Thread를 불러(thread 생성) 물이 끓는 것을 확인한 후(wait) 면을 넣으라(Task B)는 이야기를 해 놓고 쇼파에 누워 페이스북(Task C)을 할 수 있을 것이다.

#### 2.1. Basic Usage

Java의 Thread는 `java.lang.Thread`에 구현되어 있다. Thread는 어떤 task를 실행하면서 시작되고, task의 실행이 끝나 더 이상 실행할 task가 없는 경우 제거된다. task의 구현은 `java.lang.Runnable` interface를 통해 구현할 수 있다. 기본적인 thread의 사용법은 다음과 같다.

```java
class CountTask implements Runnable {
    public void run() {
        count = 0;
        while(count<5) {
            System.out.println("Count: " + count);
            count++;
        }
    }
}

Thread otherThread = new Thread(new CountTask());
otherThread.start();
```

위 코드에서 CountTask의 `run()` 메서드 내부의 변수는 새로 만들어진 otherThread의 local memory stack에 저장된다. 각각의 thread는 각각의 instruction pointer와 stack pointer를 가지며, 이 stack pointer는 thread-local memory stack의 변수를 가리킨다. thread-local memory stack은 다른 스레드에서 접근할 수 없는 영역이다.

#### 2.2. Multithreading Considerations

여러 개의 스레드를 사용함으로서 얻는 이점들도 있지만, 그에 대해 지불해야 하는 cost도 적지 않다. 모든 코드가 순차적으로 실행되는 Single-threaded 애플리케이션에 비해 MultiThreaded 애플리케이션의 경우 스레드 수만큼의 실행 흐름에 대해 고려해야 하며, 그에 따라 에러가 발생할 확률이 높아지고 디버깅이 어려워진다. 서로 다른 스레드 - 실행 흐름 - 상에서 돌아가는 코드들의 순서가 nondeterministic하기 때문에 고려해야 할 문제도 많다. 단순하게 코드가 복잡하고 예측하기 어려워지는 것 뿐 아니라 공유된 자원에 접근할 경우 의도대로 코드가 작동하지 않는 문제가 있다.

예를 들어, 두 스레드 A, B가 있다고 하자. 두 스레드 A, B는 모두 공유된 자원 `count`에 접근하는데, A는 `count`를 증가시키고, B는 감소시킨다. Java 코드로 표현하면 다음과 같다.

```java
public class RaceCondition {
    int count;

    public static void main(String[] args) {
        count = 0;
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                count++;
                System.out.println(count);
            }
        });

        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                count--;
                System.out.println(count);
            }
        });

        threadA.start();
        threadB.start();
        System.out.println(count);
    }
}
```

단순하게 생각했을 때, 출력은 1, 0, 0이 되어야 한다. 그러나 위 코드를 실행해 보면, 출력은 항상 - 같은 환경에서 테스트할 경우 확률적으로 한 개의 결과를 더 많이 보여줄 수는 있으나 - 다른 결과를 보여준다. 스레드 A가 B보다 먼저 실행된느 것을 보장할 수 없다는 것이다. 비단 실행 순서의 문제일까? 세 번째 출력은 항상 0일까?

최종적으로 `count`가 가지는 값은 0일 수도 있지만, 1 혹은 -1이 될 수도 있다. `count`가 Race Condition(경쟁 상태)에 있기 때문이다. Race Condition이란 공유 자원에 대해 여러 개의 접근이 동시에 이루어지는 상태를 말한다. 다행스럽게도 스레드 B가 A의 작업이 끝난 후 - `count`가 0에서 1로 바뀐 후 - 혹은 A가 B의 작업이 끝난 후에 실행된다면 `count`는 최종적으로 0이 되겠지만, 만약 A와 B가 동시에 - count가 0일 때 - `count`에 접근해 작업을 실행한다면 최종적으로 `count`는 -1이나 1의 값을 가지게 된다.

#### 2.3. Thread Safety

멀티스레드 환경에서는 위와 같이 공유된 자원에 여러 스레드에서 접근하는 상황이 자주 발생하게 된다. 여러 스레드에서 하나의 자원을 공유하는 것은 하나의 writer와 여러 개의 reader가 있을 경우 성능적인 측면에서 좋은 선택일 수 있지만, 개발자는 항상 실수를 하기 마련이므로 Thread Safety에 대한 고민이 필요하다. Thread Safety란 어떤 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없음을 뜻하는 말이다. 하나의 함수가 한 스레드에서 호출되어 실행될 때, 다른 스레드에서 같은 함수를 호출해 동시에 실행되더라도 각각의 스레드에서 함수가 올바르게 작동해야 한다는 것이다.

그렇다면 어떻게 Thread Safety하도록 만들 수 있을까? 공유된 자원이 없도록 만들면 된다. Thread-local storage를 사용하는 등 Re-Enterancy(재진입성)를 가지는 함수만을 사용하면 된다. Re-Enterancy를 가지는 함수란 언제나 다시 실행해도 같은 결과를 가지는 것으로, 간단히 이야기하면 호출 시 제공한 파라미터만으로 동작하는 전역 변수와 무관하게 돌아가는 함수를 말한다. 그러나 공유된 자원을 꼭 사용해야 하는 경우도 있다. 이러한 경우 세마포어 등의 락으로 상호 배제를 만들거나, atomically 하게 실행하도록 - 한 번에 하나의 스레드에서만 실행하도록 - 만들면 된다. 이렇게 atomically하게 실행되는 코드 영역을 critical section(임계영역)이라고 한다.

Java에서는 `synchronized` 키워드나 `java.util.concurrent.locks.ReentrantLock` 패키지를 사용하여 atomic execution을 구현할 수 있다. 두 가지 방법 모두 critical section이 atomical하게 실행되도록 다른 모든 thread를 block하는 방식으로 동작한다.

`synchronized` 키워드는 세 가지 방법으로 사용될 수 있는데, 각각에 대한 예시 코드를 보자.

- 해당 함수가 실행되고 있는 동안 동기화를 보장
```java
public synchronized void someMethod() {
    // Do something...
}
```

- 해당 블록 내에서 동기화를 보장
```java
public void someMethod() {
    // Do Something...
    synchronized(this) {
        // Do something...
    }
    // Do Something...
}
```

- 해당 블록 내에서 해당 변수에 대해 동기화를 보장
```java
public void someMethod(int sth) {
    // Do something...
    synchronized(sth) {
        // Do something...
    }
    // Do something...
}
```

`ReentrantLock`의 경우 `synchronized` 키워드에 비해 더 많은 기능을 제공한다. `ReentrantLock`은 Lock 인터페이스의 구현체로서, 타임아웃애 있는 Lock, Polling Lock 등을 지원한다. Lock 인터페이스와 간단한 사용법을 소개하고 넘어가도록 한다. 더 자세한 설명은 [이 포스트](http://ismydream.tistory.com/51)에 잘 설명되어 있다.

- Lock Interface
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException();
    boolean tryLock();
    boolean tryLock( long timeout, TimeUnit unit() throws InterruptedException();
    void unlock();
    Condition newCondition();
}
```

- Example
```java
Lock mLock = new ReentrantLock();
mLock.lock();
try {
    // Do something...
} finally {
    mLock.unlock();
}
```

#### 2.4. Task Execution Strategies

여러 개의 스레드를 사용할 경우 스레드를 적재적소에 사용하는 것이 매우 중요한데, 하나의 스레드에서 모든 일을 처리하는 경우 프로그램은 unresponsive하게 될 것이고, task 하나당 하나의 스레드를 사용하는 경우 context switching, thread communication 등의 overhead가 성능을 떨어뜨릴 것이다. 그렇다면 어떻게 task execution을 수행할 것인가? Sequential execution과 Concurrent execution에 대해 살펴보자.

Sequential execution은 task들이 순차적으로 실행되는 것을 말한다. 이 경우 하나의 스레드에서 실행하는 것이 효율적이며, 당연하게도 Thread Safe하다. 하지만 throughput이 낮고, 중간에 오래 걸리는 task가 있다면 그 후의 모든 task들이 delay되거나 실행되지 않을 수 있다.

Concurrent execution의 경우 task들이 parallel하게 실행되기 때문에 CPU를 효율적으로 사용할 수 있으나, Thread Safety를 보장할 수 없으므로 synchronization이 필요하다. Concurrent execution을 구현할 경우 많은 방법이 있으나 thread를 재사용하거나 너무 과도하게 사용하지 않도록 해야 한다.

효율적인 프로그램을 만드려면 실행하려는 task들에 따라 sequential execution과 concurrent execution을 적절히 사용해야 한다.

#### 2.5. MultiThreading on Android : UI Thread Only

다시 안드로이드로 돌아와 안드로이드의 UI Thread를 살펴보자. 앞서 UI Element에 대한 접근은 Application Framework Layer에서 WindowManager를 통해 제한한다고 이야기했다. 왜 이런 제한을 걸었을까? UI Element를 조작하는 과정을 생각해보자. UI Element들은 단순히 Activity - 혹은 View - 의 인스턴스 필드라고 생각하기 쉬우나, (조사 필요) 실제로는 조금 더 복잡하다. 그러나 UI Element 들에 접근할 때 synchronization에 신경써야 하지는 않는다. Android Runtime이 UI elements들에 대해 single-threaded로 작동하도록 강제함으로서 concurrency problems에 대해 자유로워질 수 있는 것이다.

### 3. Thread Communication in Java

지금까지 안드로이드에서 멀티스레딩이 필요한 이유와 방법에 대해 살펴보았다. 안드로이드만의 스레드 통신 방법인 Handler/Looper에 대해 이야기하기 전에 Java의 Thread 통신 방법들을 먼저 알아보자.

#### 3.1. Pipes

[그림]

`java.io` 패키지의 Pipe는 단방향 데이터 채널을 위해 사용되는 것으로, POSIX의 pipe operator와 비슷한 기능을 하지만, 프로세스 간의 통신을 하는 POSIX pipe와는 달리 VM 위의 스레드 사이에서 output redirecting을 한다. Pipe에 데이터를 쓰는 스레드를 Producer 스레드라 하고, Pipe에서 데이터를 읽는 스레드를 Consumer 스레드라고 한다. Pipe는 circular buffer로서, producer와 consumer thread만 접근 가능한 - 둘 사이에 공유된 - 자원이다. 앞서 Thread Safety 부분에서 언급한 것처럼 한 개의 스레드만 데이터를 조작하고, 나머지 하나의 스레드는 읽기만 하므로 Pipe를 사용하는 것은 Thread Safe한 방법이다.

Pipe는 여러 개의 task를 decouple하기 위한 용도로 많이 쓰이며, 두 개의 long-running task가 있을 경우 한 개의 task가 끝난 후 다음 task가 새로운 스레드에서 실행될 수 있게 해 준다. Pipe는 `PipedInputStream`, `PipedOuputStream`을 통해 binary나 character data를 전달할 수 있게 해 주는데, connection이 형성될 때부터 닫힐 때까지 작동한다. 이 과정을 크게 세 가지로 나누면 setup, data transfer, disconnection으로, 간단한 예시를 보자.

- Set up
```java
PipedInputStream pipedInputStream = new PipedInputStream();
PipedOutputStream pipedOutputStream = new PipedOutputStream();
pipedInputStream.connect(pipedInputStream);
```

- Data transfer
```java
Thread inputThread = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            String string = "Hello Pipe!";
            pipedOutputStream.write(string.getBytes());
        } catch(IOException e) {
            e.printStackTrace();
        }
    }
});

Thread outputThread = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            int data = pipedInputStream.read();
            for(; data != -1; data = pipedInputStream.read()) {
                System.out.print((char)data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});

inputThread.start();
outputThread.start();
```

위 코드에서 `PipedInputStream.read()`와 `PipedOutputStream.write()`는 blocking call이기 때문에 같은 스레드에서 동시에 read와 write를 하면 deadlock 상태가 되는 것에 주의하라.

- Disconnection
```java
pipedInputStream.close();
pipedOutputStream.close();
```

그렇다면 Android에서 이를 적용시키려면 어떻게 해야 할까? 간단한 예시로 액티비티를 실행하는 동안 어떤 이벤트가 발생하면 worker thread에서 처리하는 상황을 생각해 보자. `onCreate()`시에 위의 `PipedInputStream`과 `PipedOutputStream`을 생성 후 연결시켜 주고, worker thread에서 무한 루프를 돌며 `PipedInputStream`에 data가 있는지 계속 확인하도록 만든다. 이벤트 발생 시에 main thread에서 `PipedOutputStream`에 `write()`를 하도록 하면 Pipe가 비어 있지 않으므로 worker thread에서 data를 받아 처리하게 된다. 이 때 `onDestroy()`에서 stream들을 `close()`하고 worker thread를 `inturrupt()`해야 메모리 낭비가 생기지 않는다. 또한 pipe가 가득 차면 UI thread를 blocking 하게 되므로 buffer size를 충분하게 설정해야 한다.

(TODO sample code 추가할 것)


#### 3.2. Shared Memory

[그림]

스레드의 특징 중 하나는 한 프로세스 내의 모든 스레드는 각각의 stack 영역을 제외하고는 다른 모든 부분 - Code, Data, Heap 영역을 공유한다는 것이다. 이 Heap 영역을 이용하여 스레드간의 통신을 할 수 있다. Java에서 모든 객체는 Heap 영역에 저장되며, 이 객체들의 reference들은 각각 thread의 stack에 저장된다. 따라서 이 reference만 thread간에 전달해 주면 전달받은 thread는 전달받은 reference가 가리키는 객체에 접근할 수 있게 된다.

만일 두 스레드가 순서대로 실행되야 하며, 두 스레드 간에 Shared Memory를 사용해 통신한다면 어떻게 해야 할까? 앞서 Pipe의 예시처럼 어떤 state를 polling하여 구현할 수 있다. Shared Memory에 state를 나타내는 변수를 만들고, 무한 루프를 돌며 state 변수가 변하는 것을 체크하는 것이다. 이 방법도 물론 잘 동작하지만, 이러한 busy waiting은 성능 저하를 초래한다. Java의 built-in signaling mechanism을 이용하면 더 효율적으로 작동하게 할 수 있는데, `java.lang.Object`에 정의되어 있는 `wait()`, `notify()`, `notifyAll()` 세 개의 메서드를 사용하는 것이다. 간단하게 예시 코드를 통해 사용법을 소개하고 넘어가겠다. 더 자세한 설명은 [이 포스트](http://tutorials.jenkov.com/java-concurrency/thread-signaling.html)를 참고하라.


```java
public class WaitNotify {
    Object lock = new Object();
    boolean wasSignalled = false;

    public void doWait() {
        synchronized(lock) {
            while (!wasSignalled) {
                lock.wait();
            }
        }
        wasSignalled = false;
    }

    public void doNotify() {
        synchronized(lock) {
            wasSignalled = true;
            lock.notify();
        }
    }
    
    public static void main(String[] args) {
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("threadA: run() called");
                for(int i = 0; i < 5; i++) {
                    System.out.println("threadA: 0." + i + "s");
                    sleep(100);
                }
                doNotify();
            }
        });

        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("threadB: run() called");
                doWait();
                System.out.println("threadB: start working");
            }
        });

        threadA.start();
        threadB.start();
    }
}
```

`doNotify()` 함수가 실행되기 전까지는 `wasSignalled`의 값이 `false`이므로 `doWait()`함수에서 `lock.wait()`을 실행하게 된다. `lock` Object에 대해 synchronized 된 블럭 안에 있는 코드는 `lock`객체에 대한 Lock을 획득하기 전까지는 비활성화된다. `threadA`가 시작된지 0.5초가 지나 `doNotify()`가 호출되면 `lock` 객체의 Lock을 반환하고, 그제서야 threadB의 `doWait();`을 벗어나 다음 코드로 넘어가게 된다.

다른 방법으로는 `java.util.concurrent.CountDownLatch`를 사용하는 방법이 있다. `CountDownLatch.await()`, `CountDownLatch.countDown()` 메서드를 사용하는 방법으로, [공식 API 문서](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CountDownLatch.html)에 예제 코드까지 자세하게 나와 있으니 자세한 설명은 생략하도록 하겠다.


#### 3.3. Blocking Queue

위에서 살펴본 thread signal을 사용하는 방법은 low-level mechansim이기 때문에 use case에 따라 많은 부분을 직접 설정해 사용할 수 있다. 하지만 그만큼 고려해야 할 사항이 많고, 에러를 일으키기 쉽다는 단점이 있기 때문에 Java에서는 단방향 통신에 대해 추상화된 high-level signaling mechansim을 제공한다.

[그림]

`java.util.concurrent.BlockingQueue` 인터페이스들의 구현체로는 여러 가지가 있는데, Array로 구현되어 고정 크기를 가지는 `ArrayBlockingQueue`, Linked List로 구현된 `LinkedBlockingQueue`, Priority를 가지는 `PriorityBlockingQueue`, insert와 remove가 동시에 이루어지는, 크기가 항상 0으로 유지되는 `SynchronousQueue` 등이 있다. [공식 API 문서](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/BlockingQueue.html)의 예제 코드를 살펴보자.

```java
class Producer implements Runnable {
    private final BlockingQueue queue;
    Producer(BlockingQueue q) { queue = q; }
    public void run() {
        try {
            while (true) { queue.put(produce()); }
        } catch (InterruptedException ex) { ... handle ...}
    }
    Object produce() { ... }
}

class Consumer implements Runnable {
    private final BlockingQueue queue;
    Consumer(BlockingQueue q) { queue = q; }
    public void run() {
        try {
            while (true) { consume(queue.take()); }
        } catch (InterruptedException ex) { ... handle ...}
    }
    void consume(Object x) { ... }
}

class Setup {
    void main() {
        BlockingQueue q = new SomeQueueImplementation();
        Producer p = new Producer(q);
        Consumer c1 = new Consumer(q);
        Consumer c2 = new Consumer(q);
        new Thread(p).start();
        new Thread(c1).start();
        new Thread(c2).start();
    }
}
```

Producer는 `BlockingQueue.put()`을 하고, Consumer는 `BlockingQueue.take()`를 하는 것만으로 Thread Safe한 통신을 구현할 수 있다. 내부적으로 atomical하게 작동하도록 lock을 컨트롤 해 주기 때문이다.


### 4. Thread Communication in Android

지금까지 Java의 Thread Communication 방법들에 대해 알아보았다. 앞서 언급한 모든 방법은 Android에서도 같은 방법으로 사용할 수 있으나, 모두 UI 스레드를 block하는 상황 - Queue가 가득 차거나 하는 - 이 발생할 위험이 있다. UI Thread가 block되면 반응성이 저하되고 ANR이 발생 위험이 있으므로, 이 문제를 해결하기 위해서는 nonblocking한 consumer-producer pattern이 필요하다. Android platform에서는 자체적으로 message handling mechanism을 만들어 `android.os` 패키지에서 제공하고 있다.

#### 4.1. Android Message Handling Mechanism

[그림]

안드로이드에서 message handling은 `android.os.Looper`, `android.os.Handler`, `android.os.MessageQueue`, `android.os.Message` 네 가지를 이용해 이루어진다. `Looper`는 message dispatcher로서, 하나의 consumer thread에서 작동한다. `Handler`의 경우 consumer thread의 message processor이자, producer thread가 `Message`를 queue에 넣을 수 있도록 하는 interface 역할을 한다. `MessageQueue`는 크기제한이 없는 Linked List로서, `Looper`에서 dispatch된 `Message`들이 `Handler`를 통해 추가된다. `Message`란 말 그대로 임의의 데이터나 객체를 담는 객체로, consumer thread에서 실행될 것을 담아 보낼 수 있다.

message handling이 이루어지는 과정은 크게 세 부분 - Insert, Retrieve, Dispatch - 으로 이루어진다. 먼저 producer thread가 consumer thread에 연결되어 있는 `Handler`에 `Message`를 insert한다. 한편, consumer thread에서 작동하는 `Looper`는 `MessageQueue`의 `Message`들을 순차적으로 Retrieve한다. Retrieve된 `Message`는 `Looper`에 의해 알맞은 `Handler`로 Dispatch되고, `Handler`에 의해 처리된다.

각각의 클래스들에 대해 더 살펴보기 전에 `android.os.Looper`의 소스 코드에 주석으로 나와 있는 샘플 코드를 간단히 보고 넘어가자.

```java
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();
        
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };

        Looper.loop();
    }
}
```

#### 4.2. MessageQueue

Source Code : [platform_frameworks_base/core/java/android/os/MessageQueue.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/MessageQueue.java)

`android.os.MessageQueue`에 정의된 메시지큐는 단방향 Linked List로 구현되어 있는데, Producer thread가 insert한 `Message`가 차례대로 dispatch되어 consumer thread에 전달된다. insert된 `Message`들은 timestamp에 따라 정렬되며 timestamp를 queue 첫 번째의 timestamp보다 작은 값을 주는 식으로 queue의 첫 번째로 새로운 `Message`를 삽입할 수 있다. `Message`의 timestamp가 현재 시간보다 미래라면 dispatch 하지 않고 기다리는데, Dispatch Barrier를 - 멤버 `mNextBarrierToken`를 이용해 구현됨 - 넘어온 `Message`가 없다면 consumer thread를 block하게 되고, 넘어온 것이 생기면 다시 실행된다. 이렇게 block되는 시간동안 `MessageQueue.IdleHandler`를 사용하면 다른 작업을 처리할 수 있는데, 만약 idle time 없이 어떤 작업들이 수행되는 consumer thread의 MessageQueue라면 `Looper.quit()`를 통해 스레드를 종료하고 메모리를 반환할 수 있다. 단, UI thread의 `Looper`는 정지시킬 수 없다. `IdleHandler`는 `boolean queueIdle()` 하나의 메서드만을 가지는 인터페이스인데, true를 반환할 경우 계속 `IdleHandler`를 active한 상태로 두고, false를 반환할 경우 지금 설정되어 있는 `IdleHandler`를 제거한다. 처음 스레드가 만들어졌을 때의 idle time을 건너뛰고 다음 idle time에 thread를 제거하는 Thread 내부 코드를 간단히 소개하고 넘어가겠다.

```java
private boolean mIsFirstIdle = true;

...

@Override
public void run() {
    ...
    Looper.myQueue().addIdleHandler(this);
    ...
}

@Override
public boolean queueIdle() { 
    if (mIsFirstIdle) {
        mIsFirstIdle = false;
        return true; 
    }
    mConsumerHandler.getLooper().quit();
    return false;
}

...

```

(Remove, Add, Observe 등 Method 추가할 것)


#### 4.3. Message

Source Code : [platform_frameworks_base/core/java/android/os/Message.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/Message.java)

`android.os.Message` 클래스는 container object로서, data item이나 task를 전달하는데 사용된다. `Message`의 파라미터로는 여러 가지가 있는데, 정리하면 다음과 같다.

| Name     | Type   | Description |
|:--------:|:------:|-------------|
| `what`      | `int`    | Message identifier |
| `arg1`, `arg2` | `int` | Simple integer data values |
| `obj` | `Object` | Arbitrary objcet. If it is handed off to another process, it has to implement `Parcelable` |
| `data` | `Bundle` | Container of arbitrary data values |
| `replyTo` | `Messenger` | Reference to `Handler` in other process. Enables 2-Way IPC |
| `callback` | `Runnable` | Task to execute on a thread. Internal instance field holds the `Runnable` object from `Handler.post()` |

이 때 우리가 흔히 Task라고 부르는 `Runnable` 객체인 `callback`을 가지는 `Message`는 다른 모든 데이터를 가질 수 없다는 것에 주의하라. `MessageQueue`는 Task를 전달하는 `Message`와 data를 전달하는 `Message` 모두 가질 수 있다. 타입에 상관없이 모든 `Message`들은 순서대로 실행되는데, Task message는 `Handler.handleMessage(Message)`에서 아무 메시지도 받을 수 없다.

`Message`의 lifecycle은 아주 간단한데, 먼저 producer thread에서 생성되고 초기화된다. 이 과정에서 생성자를 이용해 만들기도 하지만, 보통 Factory method들을 사용해 만든다. 각각 만드려는 메서드에 따라 사용하는 메서드는 다음과 같다.
- 빈 메시지를 만들 때 
    `Message.obtain()`
- Data 메시지를 만들 때 
    `Message.obtain(Handler h, SomeData data)`
- Task 메시지를 만들 때 
    `Message.obtain(Handler h, Runnable task)` 

이렇게 초기화된 메시지는 `MessageQueue`에 enqueue되고, consumer thread로 dispatch될 때까지 Pending 상태가 된다. `Looper`는 루프를 돌며 큐의 메시지들을 차례로 dispatch 시키는데, dispatch된 메시지들은 consumer thread에서 실행된다. 이후 `Message` 인스턴스는 Android Runtime에 의해 message pool로 반환되어 재사용된다.

#### 4.4. Looper

Source Code : [platform_frameworks_base/core/java/android/os/Looper.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/Looper.java)

`android.os.Looper` 클래스는 `MessageQueue`의 `Message`들을 dispatch 시켜 알맞은 `Handler`로 연결해 준다. 한 스레드는 하나의 `Looper`만 가질 수 있고, UI thread는 기본적으로 `Looper`를 가지고 있다. `Looper`가 생성되고 준비되는 과정을 잠시 살펴보자.

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

final MessageQueue mQueue;
final Thread mThread;

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

보다시피 `prepare()`에서는 스레드 로컬 메모리에 `Looper`를 생성해 set한다. `Looper`는 생성될 때 새로운 `MessageQueue`를 생성하고, 생성자가 실행되고 있는 스레드를 저장한다. 위에서 소개한 간단한 예제에서처럼 별다른 코드 없이 간단하게 `LooperThread`를 만들 수 있게 static method로서 thread와 `Looper`를 연결시켜 준 것이다.

이제, 핸들러에 대한 부분은 잠시 미뤄두고, `Looper.loop()` 메서드에 대해 살펴보자.

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```

`Looper`는 말 그대로 `for(;;)`를 통해 무한 루프를 돌며 다음 `Message`를 `msg.target.dispatchMessage(msg)`한다. 이 때 `Message msg = queue.next()`을 통해 `MessageQueue`에서 `Message`를 가져오는데, dispatch barrier를 통과한 `Message`가 없다면 이 부분에서 blocking이 일어나 기다리게 된다.

Looper를 제거하는 것은 간단한데, `quit()` 메서드를 호출하면 dispatch barrier를 통과한 `Message`를 포함, `MessageQueue`의 모든 pending 메시지를 삭제한다. 이 때 `quit()` 대신 API level 18부터 추가된 `quitSafely()` 메서드를 이용하면 dispatch barrier를 통과하지 못한 메시지만 삭제할 수 있다. `Looper`를 제거한다고 thread가 제거되는 것은 아니다. 그러나 `Looper.loop()`를 통해 실행되던 무한루프가 멈춤으로서 그 스레드에서 더 이상의 작업이 없으면 자연스럽게 제거될 것이다. 주의해야 할 것은 `quit()`한 후 다시 `Looper.prepare()`를 할 경우 `RuntimeException`이 발생하고, `Looper.loop()`를 할 경우 메시지가 dispatch되지 않아 thread가 block되게 된다.

UI Thread의 경우 Main Thread인 만큼 Looper도 특이한데, `private static Looper sMainLooper`에서 알 수 있듯 static한 Looper이다. 어디서든 `Looper.getMainLooper()`를 통해 접근할 수 있으며, 새로 `prepare()` 되거나 `quit()` 될 수 없다.


#### 4.5. Handler

Source Code : [platform_frameworks_base/core/java/android/os/Handler.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/Handler.java)

앞서 `MessageQueue`, `Message`, `Looper`를 설명하면서 빠지지 않고 등장한 `android.os.Handler`는 Android Thread Communication에서 핵심적인 역할을 하는 클래스이다. `Handler`는 insertion과 processing 두 가지를 모두 담당하는데, 항상 특정 스레드와 연결되어 있어야 하고, 해당 스레드에는 메시지를 담을 수 있는 `MessageQueue`와 메시지를 전달해줄 Looper가 있어야 한다.
`Handler`의 생성자를 호출하기 전 `Looper.prepare()` 메서드를 통해 `MessageQueue`와 `Looper`를 만들어 주지 않은 채 핸들러를 생성하려 하면 `RuntimeException`을 내며 에러가 난다. `Handler` 생성자들의 코드를 살펴보자.

```java
public Handler() { this(null, false); }
public Handler(Callback callback) { this(callback, false); }
public Handler(boolean async) { this(null, async); }

public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

`Looper`를 인자로 받는 생성자가 아닌 경우, 실행되고 있는 스레드의 루퍼를 가져온다. 만약 스레드에 루퍼가 생성되어 있지 않다면 `RuntimeException`을 띄우는 것을 볼 수 있다. 이러한 경우는 생성자 호출 전에 `Looper.prepare()`를 호출해야 한다.


```java
public Handler(Looper looper) { this(looper, null, false); }
public Handler(Looper looper, Callback callback) { this(looper, callback, false); }

public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

`Looper`를 명시적으로 넘기는 경우 특정 `Looper`와 결합시킬 수 있다. 모든 생성자는 생성시에 `Looper`와 `Looper`가 갖고 있는 `MessageQueue`와 결합되므로 `Handler` 생성 전에 `Looper` 생성이 필요하다.

```java
final MessageQueue mQueue;
final Looper mLooper;
```

하나의 스레드는 하나의 `Looper`와 `MessageQueue`만 가질 수 있다고 이야기했다. 위의 `Looper`에서 멤버 선언 코드의 `final`을 주목하라. 하지만 `Handler`의 개수엔 제한이 없다. 한 스레드 위에 여러 개의 `Handler`가 존재하는 경우 모두 하나의 `Looper`와 상호작용하며, `MessageQueue`에 들어온 순서에 따라 순차적으로 실행된다. 그러나 `Handler` 역시 `MessageQueue`와 `Looper`는 `final`로 선언되어 있으며, Binding이 이루어진 후에는 다른 `Looper`에 결합될 수 없다.

`boolean async` 부분은 Message.setAsynchronous()를 하기 위해 `mAsynchronous`에 저장하는 파라미터로, 자세한 설명은 [공식 소스코드](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/Message.java)의 주석을 인용한다.

>Sets whether the message is asynchronous, meaning that it is not
subject to {@link Looper} synchronization barriers.
>
Certain operations, such as view invalidation, may introduce synchronization barriers into the {@link Looper}'s message queue to prevent subsequent messages from being delivered until some condition is met. In the case of view invalidation, messages which are posted after a call to {@link android.view.View#invalidate} are suspended by means of a synchronization barrier until the next frame is ready to be drawn. The synchronization barrier ensures that the invalidation request is completely handled before resuming.
>
Asynchronous messages are exempt from synchronization barriers.  They typically represent interrupts, input events, and other signals that must be handled independently even while other work has been suspended.
>
Note that asynchronous messages may be delivered out of order with respect to synchronous messages although they are always delivered in order among themselves. If the relative order of these messages matters then they probably should not be asynchronous in the first place. Use with caution.


`Handler`는 위에서 살펴본 `Message` 클래스의 메서드들에 대한 wrapper 메서드들을 가지고 있는데, `Message` 생성에 대한 메서드들을 먼저 살펴보자.

```java
public final Message obtainMessage() { return Message.obtain(this); }

public final Message obtainMessage(int what) { return Message.obtain(this, what); }

public final Message obtainMessage(int what, Object obj) { return Message.obtain(this, what, obj); }

public final Message obtainMessage(int what, int arg1, int arg2) { return Message.obtain(this, what, arg1, arg2); }

public final Message obtainMessage(int what, int arg1, int arg2, Object obj) { return Message.obtain(this, what, arg1, arg2, obj); }
```

이 wrapper 메서드들을 통해 `Message`들은 message pool로부터 `Handler`로 obtain되며, 이와 동시에 불러온 `Handler`와 결합된다. 하지만 이렇게 결합되었다고 해서 `MessageQueue`로 들어가는 것은 아니다. `Message`를 `MessageQueue`에 넣으려면 `send()` 메서드를 호출해야 한다. 자세한 API는 다음과 같다.

```java
boolean sendMessage(Message msg)
boolean sendMessageAtFrontOfQueue(Message msg)
boolean sendMessageAtTime(Message msg, long uptimeMillis)
boolean sendMessageDelayed(Message msg, long delayMillis)
```

간단하게 integer data를 `MessageQueue`에 넣기 위해 빈 `Message`를 `obtain()`해서 `what` 부분에 넣어 보내는 메서드도 제공된다.

```java
boolean sendEmptyMessage(int what)
boolean sendEmptyMessageAtTime(int what, long uptimeMillis) boolean sendEmptyMessageDelayed(int what, long delayMillis)
```

Task - `Runnable` - 를 간단하게 보낼 수 있는 메서드 역시 제공된다.

```java
boolean post(Runnable r)
boolean postAtFrontOfQueue(Runnable r)
boolean postAtTime(Runnable r, Object token, long uptimeMillis) boolean postAtTime(Runnable r, long uptimeMillis)
boolean postDelayed(Runnable r, long delayMillis)
```

간단히 `post(Runnable r)`의 코드를 살펴보자.

```java
public final boolean post(Runnable r) { 
    return sendMessageDelayed(getPostMessage(r), 0);
}

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

파라미터로 받은 `Runnable`은 새로운 `Message`의 `callback`에 저장되어 `enqueueMessage(queue, msg, uptimeMillis)` 메서드에 의해 메시지큐에 들어가게 된다.

메서드 `postAtTime(Runnable r, Object token, long uptimeMillis)`의 `Object token`은 소스코드에도 구글 개발자 문서에도 명확한 설명이 없다. 코드상으로 보면 `Message.obj`에 token을 저장하는데, 메시지를 제거할 때 `token`을 가진 콜백만을 제거하는 데 사용되는 것으로 추정된다.

`Handler`는 `Message` 제거에 대한 `MessageQueue` 메서드들의 wrapper 메서드도 가지고 있다. `Message`의 callback인 `Runnable`, `Object token`, `int what` 등을 가지고 `MessageQueue.removeMessages()`를 호출하는 것이다. `Message` 제거에 대한 API는 다음과 같다.

```java
removeCallbacks(Runnable r)
removeCallbacks(Runnable r, Object token)
removeMessages(int what)
removeMessages(int what, Object object)
removeCallbacksAndMessages(Object token)
```

`MessageQueue`에 들어간 `Message`는 `Looper`에 의해 Consumer thread로 dispatch 되는데, 이 부분의 코드를 보자.

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

private static void handleCallback(Message message) {
    message.callback.run();
}

public interface Callback {
    public boolean handleMessage(Message msg);
}
    
public void handleMessage(Message msg) { }
```

`dispatchMessage(Message msg)`는 `Looper`의 루프에서 자동으로 실행되는 - 위의 `Looper.loop()` 코드 중 `msg.target.dispatchMessage(msg)` 부분을 참고하라 - 메서드로서, 처리하는 `Message`의 `callback`의 존재 여부 - Task Message인지, Data Message인지 여부 - 에 따라 실행하는 방법이 다르다. Task Message의 경우 `handleCallback(msg)`을 실행하는데, 최종적으로 `msg.callback.run()`을 실행한다. `callback`만 가지는 `Message`를 `MessageQueue`에 넣어주기만 하면 들어간 순서에 따라 `callback`이 실행되는 것이다. 앞서 `callback`을 가지는 `Message`는 다른 data값을 가질 수 없다고 했는데, 더 정확하게는 `callback`을 가지고 있는 경우 data를 처리하는 코드를 건너 뛰어 data가 있더라도 사용할 방법이 없는 것이다.

Data Message의 경우 `mCallback`이 있다면 `Handler.mCallback.handleMessage(msg)`를 실행, 없다면 건너뛰고 최종적으로 `Handler.handleMessage(msg)`를 호출한다. 이 때 `mCallback.handleMessage(msg)`의 반환값이 `true`일 경우 `Handler.handleMessage(msg)`는 실행되지 않는다. 즉, Data Message의 처리는 `Handler.handleMessage(msg)`를 오버라이딩하는 것과 `Callback` 인터페이스 - `Callback.handleMessage(msg)` - 를 구현한 것을 `Handler.mCallback`에 넣는 두 가지 방법이 있다. 이에 대한 예시를 살펴보자.

- `Handler.handleMessage(Message msg)` 사용
```java
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();
        
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };

        Looper.loop();
    }
}
```

- `Handler.Callback.handleMessage(Message msg)` 사용
```java
public class HandleMessageActivity extends Activity implements Handler.Callback {
    Handler mUIHandler;
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mUiHandler = new Handler(this);
    }
    
    @Override
    public boolean handleMessage(Message message) {
        // process incoming messages here
        return true;
    }
}
```

두 가지 방법을 동시에 사용할 경우 주의해야 할 것은 `Callback.handleMessage()`가 먼저 실행된다는 것이다. 또한 이 메서드의 반환값에 따라 - `true`일 경우 - `Handler.handleMessage()`는 실행되지 않는다는 것에 유의해야 한다.



(TODO Observing MessageQueue)
Communicating with the UI thread
    runonui code






### 5. HandlerThread

Source Code : [platform_frameworks_base/core/java/android/os/HandlerThread.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/HandlerThread.java)

앞서 `Handler`를 이용한 Android Message Passing Mechansim에 대해 이야기했다. `android.os.HandlerThread`는 `Handler`를 조금 더 간편하게 사용할 수 있도록 해 주는 클래스이며, 일반적인 `Thread`와 같은 방식으로 사용하면 내부적으로 `Looper`와 `MessageQueue`를 세팅해 주는 것이다.

#### 5.1. Basis

안드로이드 core 개발자들은 주석에서 `HandlerThread`를 *Handy class for starting a new thread that has a looper.*이라 소개하고 있다. 말 그대로 `HandlerThread`는 `Looper`에 대해 생각할 필요 없이 `Handler`를 사용할 수 있게 해주는 클래스라고 보면 된다. 간단하게 사용하는 방법을 소개한다.

```java
HandlerThread handlerThread = new HandlerThread("HandlerThread"); handlerThread.start();

mHandler = new Handler(handlerThread.getLooper()) { 
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        // Process incoming messages here
    }
};
```

`HandlerThread`를 생성한 후, `HandlerThread`의 `Looper`를 인자로 넘겨 새로운 `Handler`를 만들면 된다. 앞서 `Handler`의 생성자 설명 부분을 상기하라. `Handler`는 생성 시 `Looper`를 명시적으로 받으면 `Looper`를 가지고 있는 `Thread`에 연결된다.

`HandlerThread`의 소스코드를 보면 정말 간단하게 구현되어 있다는 것을 알 수 있다. 앞서 소개한 `LooperThread`예제와 `HandlerThread`의 구현 코드를 비교해 보자.

- `LooperThread`
```java
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();
        
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };

        Looper.loop();
    }
}
```

- `HandlerThread`
```java
public class HandlerThread extends Thread {
    Looper mLooper;
    
    ...

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
    
    ...
    
}
```

한 눈에 보기에도 비슷하다. `run()` 메서드가 실행될 때 `Looper.prepare()`와 `Looper.loop()`를 호출한다. 위 예제에서 `prepare()`와 `loop()` 사이에 코드를 넣었던 것처럼 오버라이딩을 통해 custom code를 넣을 수 있도록 `protected void onLooperPrepared() { }`이라는 빈 메서드도 제공된다.위 예제처럼 `Thread` 내부에서 `Handler`를 세팅하는 예시를 보자.

```java
class CustomHandlerThread extends HandlerThread {
    public Handler mHandler;

    @Override
    protected void onLooperPrepared() {
        super.onLooperPrepared();
        
        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // process incoming messages here
            }
        };
    }
}
```




(TODO
#### 5.2. LifeCycle
#### 5.3. Use Case(?)

    Repeated Task Execution
    Related Task
        Example - SharedPref
    Task chaining
        Example - chained network calls
    Conditional Task Insertion
)





### 6. AsyncTask

Source Code : [platform_frameworks_base/core/java/android/os/AsyncTask.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/AsyncTask.java)

`AsyncTask`는 안드로이드에서 가장 많이 쓰이는 비동기 처리 방식 중 하나로, 단어의 뜻 그대로 UI thread에서 하기 힘든 오래 걸리는 task들을 비동기적으로 처리하기 위해 만들어졌다. UI thread에서만 사용해야 하며, `Thread`나 `Handler` 등을 고려하지 않고 편하게 background task를 수행할 수 있도록 해 준다.

#### 6.1. Basis

`AsyncTask`의 소스 코드를 보면, `AsyncTask` 역시 `Handler`와 `Looper`를 사용한다. `InternalHandler`이라는 inner class의 소스 코드를 보자.

```java
private static class InternalHandler extends Handler {
    public InternalHandler() {
        super(Looper.getMainLooper());
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```

private로 선언된 `InternalHandler`은 오직 `AsyncTask` 내부에서만 접근 가능하며, Main Looper - UI Thread의 Looper - 와 결합하는 Handler이다. 하지만 `doInBackground()`라는 메서드를 통해 데이터를 주고받음으로서 `Handler` 관련 부분은 내부적으로만 작동하게 된다.

`AsyncTask`는 `AsyncTask<Params, Progress, Result>`를 상속받아 이용하는데, 미리 정의된 `protected` 메서드들을 오버라이딩하여 사용한다. `AsyncTask`에서 재정의할 수 있는 모든 콜백 메서드들을 구현하면 다음과 같은 코드를 볼 수 있다.

```java
public class FullTask extends AsyncTask<Params, Progress, Result> {
    @Override
	protected void onPreExecute() { ... }
	
	@Override
	protected Result doInBackground(Params... params) { ... }
	
	@Override
	protected void onProgressUpdate(Progress... progress) { ... }
	
	@Override
	protected void onPostExecute(Result result) { ... }
	
	@Override
	protected void onCancelled(Result result) { ... } 
}
```

이 중 `doInBackground()` 메서드는 `abstract`로 선언되어 필수적으로 구현해야 하며, 다른 메서드들은 필요에 따라 구현하면 된다. `AsyncTask<Params, Progress, Result>`에서 generic type의 경우, `Params`에는 background thread로 넘겨 줄 input data를, `Progress`에는 background thread에서 UI thread로 넘겨 줄 progress data를, `Result`에는 최종적으로 background thread에서 UI thread로 넘겨 줄 result data를 설정해 주면 된다. 이 때 generic type을 이용하지 않으려면 `void`를 넣어 unused라는 표시를 해 주어야 한다.

[그림]

위의 그림은 `FullTask`가 실행될 때 일어나는 일들을 시간 순으로 그린 것이다. 먼저 `AsyncTask.execute(Params)`가 호출되면 UI thread에서 `onPreExecute()`가 실행된다. 그 후 background thread에서 `doInBackground(Params)`가 실행되는데, 이 동안 `doInBackground(Params)` 내부에서 `publishProgress(Progress)`를 호출하면 UI thread에서는 `onProgressUpdate(Progress)`가 실행되어 UI를 업데이트 할 수 있다. `doInBackground(Params)` 메서드가 실행을 마치면 결과값을 파라미터로 넘겨 UI thread에서 `onPostExecute(Result)`가 실행되어 결과값을 반영할 수 있다. `AsyncTask`가 cancel된 경우, `onCancelled(Result)` 메서드가 대신 실행되어 결과를 처리하게 된다.

참고 : `finish()` 메서드
```java
private void finish(Result result) {
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    mStatus = Status.FINISHED;
}
```

`AsyncTask`는 소스 코드 주석에서 subclass를 만들어 사용하기를 강력히 권장하고 있으며, 사용 방법은 subclass를 만들고, 생성자를 통해 생성, `AsynTask.execute()` 메서드를 통해 실행하는 것이다. 주의할 점은 `AsyncTask` 클래스는 UI thread에 load 되어야 하며 - Jelly Bean 부터는 자동으로 해 준다 - 인스턴스 역시 UI 스레드에 생성되어야 한다. 또한 `execute()` 역시 UI 스레드에서 실행되어야 하고, 한 번만 실행될 수 있다.

`execute(Params)`가 실행될 때, `Params`는 `AsyncTask`로 전달되고, `doInBackground(Params)` 메서드를 통해 Background thread로 전달된다. Background thread에서 처리된 데이터는 UI thread의 `Looper`와 결합되어 있는 `InternalHandler`로 보내지며, dispatch 과정을 거쳐 - 위 코드의 `handleMessage()` 부분을 보라 - UI thread에서 처리된다.

```java
public final boolean cancel(boolean mayInterruptIfRunning) {
    mCancelled.set(true);
    return mFuture.cancel(mayInterruptIfRunning);
}
```

`AsyncTask`를 cancel하는 방법은은 두 가지가 있는데, interrupt를 하는 것과 하지 않는 것이다. `mayInterruptIfRunning`을 true로 하게 되면 `Thread.interrupt()`가 호출되어 `InterruptedException e`가 발생하고, `Thread.isInterrupted()` 플래그가 true가 된다. Interrupt가 일어나게 되면 blocking method call을 벗어나 바로 cancel 작업을 수행하게 된다. 반면, Interrupt를 하지 않을 경우 하던 작업을 계속하며 `mCancelled`만 `true`가 되는데, 내부에 루프가 있을 경우 - 예를 들면 사진 여러 장을 가져오는 등 - `isCancelled()` 값 검사를 통해 일정 작업 단위를 기준으로 cancel할 수 있다. 두 경우 모두 `doInBackground(Params)`에서 - `Result`가 `void`가 아니라면 - `Result`를 반환해 주어야 하며, `doInBackground(Params)` 실행 후 `onPostExecute(Result)` 대신 `onCancelled(Result)`가 호출된다.

`AsyncTask`는 3개의 state를 가지는데, `PENDING`, `RUNNING`, `FINISHED`이다. `AsyncTask` 인스턴스가 만들어지면 `PENDING` state에 있다가 `execute()`가 실행되면 `RUNNING` state로 변하고, `onPostExecute()`나 `onCancelled()`가 실행을 마치면 `FINISHED` state가 된다. 한 번 `RUNNING` state로 진입한 `AsyncTask`는 다시 실행될 수 없으며, 새로운 인스턴스를 만들어 실행해야 한다. `AsyncTask`의 state는 간단히 `AsyncTask.getStatus()` 메서드로 확인할 수 있다.


#### 6.2. Implementation

간단해 보이는 `AsyncTask`이지만, 항상 `Context`의 lifecycle을 염두에 두고 잘 맞추어서 사용해야 한다. 구글이 권장하는 대로 `AsyncTask`를 `Activity`와 같은 view 안의 inner class로 선언되고 레퍼런스 될 경우, view가 없어지더라도 `AsyncTask`가 종료되지 않으면 view의 메모리가 반환되지 않고 남아 있는 문제가 생긴다. 이러한 문제를 해결하기 위해서 static inner class로 선언하는 방식이 많이 쓰인다. [이 스레드](http://stackoverflow.com/questions/3821423/background-task-progress-dialog-orientation-change-is-there-any-100-working/3821998#3821998)에서 자세히 설명하고 있다.
또한 위에서 언급한 cancellation에 대해 policy를 정해 처리해 주어야 한다. 예시 코드를 소개하고 넘어가겠다.

```java
public class AsyncTaskActivity extends Activity {

    private static final String[] URLs = { "url1", "url2", "url3", "url4" };

    DownloadTask downloadTask;
    
    ProgressBar progressBar;
    LinearLayout linearLayout
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        ... findViewByIds, etc ...
        
        downloadTask = new DownloadTask(this);
        downloadTask.execute(URLs);
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        downloadTask.setActivity(null);
        downloadTask.cancel(true);
    }
    
    public static class DownloadTask extends AsyncTask<String, String, Void> {
        private AsyncTaskActivity mActivity;
        private int count = 0;
        
        public DownloadTask(AsyncTaskActivity activity) {
            mActivity = activity;
        }
        
        public void setActivity(AsyncTaskActivity activity) {
            mActivity = activity;
        }
        
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            mActivity.mProgressBar.setVisibility(View.VISIBLE);
            mActivity.mProgressBar.setProgress(0);
        }

        @Override
        protected Void doInBackground(String... urls) { 
            for (String url : urls) {
                if (!isCancelled()) {
                    // Simulate Downloading
                    Thread.sleep(300);
                    String fileName = "FileNameBy"+url;
                    publishProgress(fileName);
                }
            }
        return null;
        }
        
        @Override
		protected void onProgressUpdate(String... fileNames) {
			super.onProgressUpdate(fileNames);
			if (mActivity != null) {
				TextView tv = new TextView(mActivity); 
				tv.setText(fileNames[count]);
		    	mActivity.mProgressBar.setProgress(++count);
				mActivity.linearLayout.addView(tv);
			} 
		}


		@Override
		protected void onPostExecute(Void aVoid) { 
			super.onPostExecute(aVoid);
			if (mActivity != null) {
		        mActivity.mProgressBar.setVisibility(View.GONE);
		    }
		}

		@Override
		protected void onCancelled() { 
			super.onCancelled();
			if (mActivity != null) {
		        mActivity.mProgressBar.setVisibility(View.GONE);
		    }
		}
    }
}
```

(TODO
#### 6.3. Execution Options


#### 6.4. Pitfalls
Common pitfalls
when not to use

)



http://suribada.com/wp/?p=13
http://therne.me/?p=76
http://javacan.tistory.com/entry/maintainable-async-processing-code-based-on-AsyncTask
http://blog.danlew.net/2014/06/21/the-hidden-pitfalls-of-asynctask/
http://bon-app-etit.blogspot.kr/2013/04/the-dark-side-of-asynctask.html








### 7. Executor
http://blog.bsidesoft.com/?p=311&fb_ref=AL2FB

### 8. IntentService
https://realm.io/news/android-threading-background-tasks/

### 9. Loader
http://www.vogella.com/tutorials/AndroidBackgroundProcessing/article.html#concurrency_asynchtask_parallel

### 10. Select MultiThreading Method
http://www.slideshare.net/andersgoransson/efficient-android-threading



---
Ref
http://frontjang.info/443

http://huewu.blog.me/110115454542
http://huewu.blog.me/110116293622

http://blog.nikitaog.me/2014/10/11/android-looper-handler-handlerthread-i/
http://blog.nikitaog.me/2014/10/18/android-looper-handler-handlerthread-ii/
https://corner.squareup.com/2013/10/android-main-thread-1.html
https://corner.squareup.com/2013/12/android-main-thread-2.html
http://codetheory.in/android-handlers-runnables-loopers-messagequeue-handlerthread/

http://ensider.tistory.com/5