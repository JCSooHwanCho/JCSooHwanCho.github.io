---
layout: post
title: Scheduler는 무엇인가?
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트에서는 Rx에서 Scheduler가 무엇인지, RxSwift에서는 어떻게 구현되어 있고 어떻게 사용되는지 알아보도록 하겠습니다.

* **Scheduler의 개념**  
    [Intro to Rx](http://introtorx.com/Content/v1.0.10621.0/15_SchedulingAndThreading.html)에서는 다음과 같이 설명하고 있습니다. 

    > The key concept to understand is that an IScheduler in Rx is used to schedule some action to be performed, either as soon as possible or at a given point in the future. The implementation of the IScheduler defines how that action will be invoked i.e. asynchronously via a thread pool, a new thread or a message pump, or synchronously on the current thread.  
    > 해석: (Scheduler가 무엇인지 이해하기 위한) 핵심 개념은 Rx의 IScheduler(Scheduler의 인터페이스)가 수행되는 동작을 스케쥴링 하는데 쓰인다는 것입니다. 이러한 작업은 가능한 빠르게 실행될 수도 있고, 특정 시점에 수행될 수도 있습니다. IScheduler의 구현은 이러한 동작이 어떻게 호출이 될지를 정의합니다. 다시 말하면 스레드 풀, 새로운 스레드 혹은 메시지 펌프(Windows 프로그램에서 이벤트 루프를 의미합니다)에서 비동기 적으로 수행될지, 현재 스레드에서 동기적으로 수행될 지를 정의한다는 것입니다.  

    즉, Scheduler는 특정 작업이 수행되는 시점과 방법을 결정하는 장치입니다. 여기서 방법이라는 것은 같은 스레드에서 동기적으로 수행되도록 할지, 다른 스레드에서 비동기적으로 수행되도록 할지, 비동기적으로 한다면 어떤 스레드에서 실행할 지 등의 세부 사항을 의미합니다. 그래서 그 구현은 언어와 플랫폼마다 다를 수 밖에 없습니다.  

* **RxSwift에서의 Scheduler**  
  RxSwift는 Scheduler를 구현하는 것에 있어서 플랫폼의 덕을 보고 있다고 볼 수 있습니다. 이미 Scheduler와 비슷한 개념의 프레임워크인 GCD가 플랫폼 차원에서 제공되고 있기 때문입니다. 실제로 RxSwift에서 기본적으로 제공하는 Scheduler들은 GCD에서 제공하는 Queue들을 Rx방식으로 사용할 수 있도록 인터페이스를 바꿔준 것 입니다.

* **Built-In Scheduler**  
  대부분의 경우에는 RxSwift에서 기본적으로 제공하는 Scheduler들만을 사용하게 됩니다. 제공하는 Scheduler는 다음과 같습니다.

  1. CurrentThreadScheduler(Serial): 현재 스레드에서 작업을 수행하도록 하는 Scheduler입니다. Operator들이 element를 내보내는 기본 스레드이기도 합니다. 가끔 trampoline scheduler라고도 불리기도 합니다.
  
  2. SerialDispatchScheduler(Serial): 만들 때 인자로 넘긴 DispatchQueue에서 작업이 수행되도록 하는 Scheduler입니다. 이때 ConcurrentDispatchQueue를 넘기더라도, 내부적으로 Serial로 바꿔줍니다.  
  추가적인 사항으로, Serial Scheduler는 observeOn 연산자에 쓰일 경우 최적화가 가능하기 때문에 observeOn에는 Serial Scheduler를 넘겨 주는 게 좋습니다.
 
  3. ConcurrentDispatchQueueScheduler(Concurrent): SerialDispatchQueueScheduler를 Concurrent로 바꾼 것입니다. 인자로 Serial DispatchQueue를 넘겨도 문제가 발생하지는 않습니다. 특정 작업들을 백그라운드에서 수행할 때 유용합니다.  
  추가적인 사항으로, Concurrent Scheduler는 subscribeOn 연산자와 쓰이는 것에 최적화 되어 있습니다.  

  1. MainScheduler & ConcurrentMainScheduler: 메인 스레드에서 작업이 수행되도록 하는 Scheduler입니다. 메인 스레드에서 사용할 경우 스케쥴링을 하지 않고 바로 작업을 수행합니다.
  
  2. OperationQueueScheduler(Concurrent): OperationQueue를 사용한 Scheduler입니다. OperationQueue의 maxConcurrentOperationCount를 이용해서 백그라운드에서 돌아가는 작업 수를 제한할 때 유용합니다.  
  
  3. VirtualTimeScheduler(Serial): 내부적으로 clock을 가지고 있어서 clock을 기준으로 작업을 수행하는 scheduler입니다. 프로덕션 레벨에서 쓰이기 보다는 주로 Testing을 위해서 사용합니다.(RxTest 프레임워크에서 제공하는 TestScheduler의 부모 클래스입니다.) 실제 시간을 기준으로 하는 HistoricalScheduler라는 하위 클래스가 존재합니다.  