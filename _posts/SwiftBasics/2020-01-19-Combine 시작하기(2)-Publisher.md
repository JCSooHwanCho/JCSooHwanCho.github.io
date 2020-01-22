---
layout: post
title: Combine 시작하기(2)-Publisher
comments: true
tags: [Swift,Apple,Guideline, Combine]
category: [SwiftBasics]
---  

이번 시간에는 Publisher에 대해서 알아보고, Publisher를 만들고 다룰 때 고려할 것들을 알아보도록 하겠습니다.  

* **Publisher 프로토콜**  
   모든 Publisher는 [Publisher 프로토콜](https://developer.apple.com/documentation/combine/publisher)을 따라야 합니다. 이 Publishers는 3가지 요소를 필수적으로 정의해야 합니다.

   * associatedtype Output: Publisher가 내보내는 값의 타입입니다.
   
   * associatedtype Failure: Publisher가 내보내는 에러의 타입입니다. Error 프로토콜을 채택한 타입이어야 합니다.
   
   * [receive(subscriber:)](https://developer.apple.com/documentation/combine/publisher/3229093-receive): 해당 Publisher에 인자로 주어지는 Subscriber를 전달합니다. Subscriber는 Publisher와 타입이 일치해야만 합니다. receive라는 단어는 Publisher의 입장에서 쓰인 의미이기 때문에, 사용자가 호출할 때는 [subscribe(_:)](https://developer.apple.com/documentation/combine/publisher/3204756-subscribe)메소드를 통해서 사용하게 됩니다.  
     내부적으로는 Publisher는 자신이 사용하는 [Subscription](https://developer.apple.com/documentation/combine/subscription) 객체를 만들어서 Subscriber에게 전달을 하게 되는데, 이 Subscription 객체가 실제적인 작업과 값 전달을 수행하게 됩니다. 즉, Publisher는 Subscription 객체의 팩토리 객체라고 볼 수 있습니다. 

  Publisher 프로토콜은 또한 extension으로 많은 Operator들을 정의하고 있습니다. 이 Operator에 대해서는 다른 포스트에서 좀 더 자세히 알아보도록 하겠습니다.

* Publisher 만들기  
  Apple은 Publisher를 커스텀해서 만드는 것을 권장하지 않습니다. 실제로 Publisher 프로토콜을 구현하기 위해서 신경쓸 부분도 있고, 그에 맞는 Subscription 타입 또한 구현을 제대로 하려면 신경 써줘야 될 부분이 좀 있기 때문입니다. 따라서 다음과 같은 방법을 권장합니다.

  * [Subject](https://developer.apple.com/documentation/combine/subject)의 구체 타입인 [PassthroughSubject](https://developer.apple.com/documentation/combine/passthroughsubject)와 [CurrentValueSubject](https://developer.apple.com/documentation/combine/currentvaluesubject)을 활용합니다. 이 Subject는 외부에서 값을 손쉽게 주입할 수 있는 Publisher의 일종입니다.
  
  * 클래스의 Property인 경우, [@Published](https://developer.apple.com/documentation/combine/published) 프로퍼티 래퍼를 적용하면, 값이 변화할 때 마다 변한 값이 Subscriber들에게 전파됩니다.

* Publisher 사용하기  
  Apple은 여러가지 간편하게 사용할 수 있는 Publiser 타입을 제공해줍니다. 

  * [Just](https://developer.apple.com/documentation/combine/just): 하나의 값을 즉시(동기적으로) 발생시키고자 할 때 사용하는 Publisher입니다. 
  
  * [Future](https://developer.apple.com/documentation/combine/future): 하나의 값을 비동기적으로 발생시키고자 할 때 사용하는 Publisher입니다.
  
  * [Deferred](https://developer.apple.com/documentation/combine/deferred): 구독이 일어나기 전까지 어떤 Publisher가 사용될 지 모르다가 구독이 일어나야지만 실제 사용될 Publisher가 결정되는 Publisher입니다. 외부 조건에 의해 Publisher가 달라져야 할 때 유용합니다.
  
  * [Empty](https://developer.apple.com/documentation/combine/empty): 아무런 값을 발생시키지 않는 Publisher입니다. 즉시 정상 종료하도록 설정할 수도 있고, 영원히 종료하지 않도록 설정할 수도 있습니다.
  
  * [Fail](https://developer.apple.com/documentation/combine/fail): 즉시 에러를 발생시키고 종료되는 Publisher입니다.  
  
  * [Record](https://developer.apple.com/documentation/combine/record): 구독할 때마다 이전에 발생했던 값들을 다시 내보내는 Publisher입니다.  

  또한 Apple은 몇가지 동기 혹은 비동기 인터페이스에 추가적인 publisher 프로퍼티를 제공합니다. 

  ```swift
  // 동기 인터페이스의 publisher 형태
  let syncSub = [1,2,3,4]
            .publisher
            .sink { print($0) }

  // 비동기 인터페이스의 publisher 형태
  let asyncSub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .sink(receiveCompletion: { print ($0) },
          receiveValue: { print ($0) })
  ```

  이어지는 포스트에서는 Combine의 다른 구성요소들에 대해서도 알아보겠습니다. 

---

> 참고 자료  
> [Documentation-Publisher](https://developer.apple.com/documentation/combine/publisher)  
> [Article-Receiving and Handling Events with Combine](https://developer.apple.com/documentation/combine/receiving_and_handling_events_with_combine)