---
layout: post
title: RxSwift기초 - Relay
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

Subject에 이어서 오늘은 Relay에 대해서 알아보도록 하겠습니다.

Relay는 기존의 Variable을 대체하기 위한 개념입니다.(정확히는 BehaviorRelay가 Variable을 대체합니다.) Variable이라는 이름에서 알 수 있듯 일반적인 변수를 대체해서 사용하기 위한 타입이였는데, Rx보다는 기존 상태기계(State Machine)에 더 가까워서 이를 대체하기 위해 나온 것이 Relay입니다.

Relay는 RxRelay패키지에 정의되어 있기 때문에, 사용하기 위해서는 RxRelay 패키지를 임포트( import) 해야 합니다. 하지만 RxCocoa 패키지가 내부적으로 이미 임포트 하고 있기 때문에 RxCocoa를 쓰신다면 따로 임포트 할 필요는 없습니다.

Relay는 Subject의 Wrapper 클래스라고 볼 수 있습니다. 대신 Subject와는 다른 특징을 가지고 있는데 바로 **절대 종료이벤트로 인해 종료되지 않는다는 것입니다.** 

Relay는 아예 이벤트 객체를 받는 on() 메소드가 구현되어 있지 않기 때문에 이벤트를 바로 넘길 수는 없습니다. 대신 accept()라는 메소드를 대신 사용하는데, 이 메소드는 **값**을 인자로 받습니다. accept() 메소드는 값을 인자로 받아 next 이벤트로 감싸 내부의 Subject에게 흘려보냅니다.

Subject는 4종류가 있었는데, Relay는 이 중 BehaviorSubject, PublishSubject를 래핑한 BehaviorRelay, PublishRelay를 제공합니다. 기존 Subject의 경우와 마찬가지로 최근의 상태값을 가지고 구독시 이를 방출하는지 여부로 구분해서 사용하시면 됩니다.  

> Relay에 대해서 찾아보신 분들은 'Relay를 bind하는 것은 지양해야 한다'는 말을 들어보신 적이 있을 것입니다. 하지만 RxRelay에서는 옵저버블과 Relay를 bind하는 메소드를 제공해주고 있습니다. 

```swift
//PublishRelay에 대한 bind 메소드. BehaviorRelay도 타입 빼고 같은 내용의 메소드가 있다.
public func bind(to relays: PublishRelay<Element>...) -> Disposable {
        return bind(to: relays)
    }

private func bind(to relays: [PublishRelay<Element>]) -> Disposable {
        return subscribe { e in
            switch e {
            case let .next(element):
                relays.forEach {
                    $0.accept(element) // Relay 내부의 Subject에 next이벤트를 흘려보낸다.
                }
            case let .error(error):
                //디버그 모드면 런타임 에러를, 릴리즈 모드면 그냥 에러 메시지를 print만 한다.
                //즉, 어떻게 해서든 내부 subject에 에러가 흘러가지를 않는다.
                rxFatalErrorInDebug("Binding error to publish relay: \(error)")
            case .completed:
                break
            }
        }
    }
```  
<br>
> 정의를 보면 알 수 있듯, 내부적으로 종료 이벤트를 무시해 버린다는 것을 알 수 있습니다. 이는 옵저버블이 종료되어도, Relay내의 Subject는 종료되지 않는 비일관적인 상태를 만들게 됩니다. 따라서 옵저버블이 절대 종료되지 않는다는 보장이 없는 이상, 옵저버블에 Relay를 bind시키는 것은 바람직하지 않습니다.

---

간단하게 Relay에 대해서 알아보았습니다. 다음에는 Driver와 Signal에 대해서 알아보도록 하겠습니다. 
 