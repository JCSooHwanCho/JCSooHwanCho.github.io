---
layout: post
title: Combine 시작하기(3)-Subscriber
comments: true
tags: [Swift,Apple,Guideline, Combine]
category: [SwiftBasics]
---  

이번 포스트에서는 Subscriber에 대해서 알아보고, Subscriber를 만들고 사용할 때 알아둬야 될 것들을 살펴보도록 하겠습니다.  

* **Subscriber 프로토콜**  
  Subscriber 역할을 하려면 반드시 Subscriber 프로토콜을 따라야 합니다. Subscriber 프로토콜은 2개의 associatedType과 3개의 필수 메소드를 구현해야 합니다.  

  * associatedtype Input: Publisher를 통해서 제공받을 값의 타입입니다.
  
  * associatedtype Failure: Publisher를 통해서 제공받을 에러의 타입입니다. 에러는 반드시 Error 프로토콜을 따르는 타입이여야 합니다.
  
  * [receive(subscription:)](https://developer.apple.com/documentation/combine/subscriber/3213655-receive): Subscriber가 Publisher를 구독할 때, Publisher가 내부적으로 호출하는 함수입니다. Publisher는 해당 메소드를 통해서 Subscriber에 Subscription 객체를 제공하고, 이 Subscription 객체를 통해 Subscriber가 이벤트를 요청하게 됩니다.
  
  * [receive(_:)](https://developer.apple.com/documentation/combine/subscriber/3213653-receive): Publisher(정확히는 Subscription)가 값을 만들어서 Subscriber에 전달할 때 사용하는 메소드입니다. 이 함수의 반환 값은 Subscriber가 몇개의 값을 더 받게 될 지를 Subscriber에 알려주는 역할을 합니다. 이 값을 어떻게 사용할지는 Subscriber의 재량입니다.
  
  * [receive(completion:)](https://developer.apple.com/documentation/combine/subscriber/3213654-receive): Publisher(정확히는  Subscription)이 정상적으로 혹은 오류로 인해 종료되었음을 통보하기 위해 사용하는 메소드입니다. 

* **Subscriber 만들기**  
  Apple은 Subscriber 프로토콜을 직접 채택해여 Subscriber를 구현하는 것을 권장하지 않습니다. 대신 다음과 같은 방법을 권장합니다.

  1. AnySubscriber의 생성자는 구현 필수 메소드들을 클로저 형태로 받습니다. Combine을 구현하는 데 필요한 세부사항들을 가려주기 때문에 유용합니다.
  
  2. Combine에서는 [Sink](https://developer.apple.com/documentation/combine/subscribers/sink), [Assign](https://developer.apple.com/documentation/combine/subscribers/assign) 이라는 기본 Subscriber를 제공합니다. 해당 Subscriber는 Cancellable 프로토콜을 채택해서 Subscription을 취소하는 기능도 자동으로 제공해줍니다. 
  
  3. 직접 구현을 해야겠다면 Publisher가 Subscription 객체를 가지도록 하고, [Cancellable](https://developer.apple.com/documentation/combine/cancellable)프로토콜을 반드시 구현하여 중간에 취소가 가능하도록 만드는게 좋습니다. 다만 Publisher가 Subscription에 대한 참조를 가지게 되면 순환 참조가 생기므로, 이에 대한 고려가 반드시 필요합니다.

* **Subscriber 사용하기**  
  타입이 맞는 Publisher에 subscribe 메소드를 호출해서 Publisher를 붙이거나 [sink](https://developer.apple.com/documentation/combine/publisher/3343978-sink)나 [assign](https://developer.apple.com/documentation/combine/publisher/3235801-assign) 등의 메소드를 활용하여 구독을 합니다.

  ```swift
    // Sink 활용 -> AnyCancellable 타입의 값 반환, 주어진 클로저를 실행
    let syncSub = [1,2,3,4]
            .publisher
            .sink { print($0) }

    // Assign 활용 -> AnyCancellable 타입의 값 반환, 주어진 객체에 keypath를 활용하여 값을 바인딩
    let assignSub = [1,2,3,4]
            .publisher
            .assign(to: /.value. on: self)
  ```  

---  

> 참고 자료  
> [Documentation-Subscriber](https://developer.apple.com/documentation/combine/subscriber)  
> [Article-Receiving and Handling Events with Combine](https://developer.apple.com/documentation/combine/receiving_and_handling_events_with_combine)