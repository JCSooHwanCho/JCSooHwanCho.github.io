---
layout: post
title: 5분상식- Self vs self
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

swift를 코드를 보다보면 self(s가 소문자)가 쓰이는 곳이 있고 Self(S가 대문자)가 쓰이는 곳이 있습니다. 이번 포스트에서는 짧게 둘 사이에 무슨 차이가 있는지 알아보겠습니다.

> 이 포스트는 다음 글들을 참조하였음을 밝힙니다.  
> [Self versus self in Swift 4 – capital “S” and lowercase “s”](http://iosbrain.com/blog/2018/09/26/self-versus-self-in-swift-4-capital-s-and-lowercase-s/)  
> [Self vs self - what's the difference?](https://www.hackingwithswift.com/example-code/language/self-vs-self--whats-the-difference)  

소문자로 시작하는 self의 경우는 타입을 정의할 때 지겹게 사용하는 그것인데, 타입 인스턴스에서 자기자신을 나타내는 프로퍼티입니다. 사용자가 직접 정의하지 않아도 컴파일러가 자동으로 추가해줍니다.

self는 해당 타입이 값 타입일 경우에는 똑같이 값 타입처럼 동작하고, 참조 타입일 경우는 포인터로써 힙 영역에 존재하는 클래스 인스턴스의 주소를 가지게 됩니다. 

> 값 타입의 self가 포인터가 아닌것은 아닙니다. 다만 값 타입 '처럼' 동작할 뿐입니다. 만약 self가 값 타입이였다면 참조할 때 마다 복사가 일어나서 제대로 된 변경도 안되고, 성능에 악영향을 줄 것입니다.  

대문자롤 시작하는 Self는 대문자로 시작하는 만큼 타입을 의미합니다. 위 self 프로퍼티의 타입을 보면 Self 타입임을 알 수 있습니다. 다만 프로그래머가 직접 쓸 수 있는 곳은 극도로 제한적이고 문맥에 따라 의미도 조금 다릅니다. 

1. Protocol 내부 : 해당 프로토콜을 채택한 타입을 의미합니다. [enum 더 알아보기](/2019-08-19-enum-더-살펴보기-CaseIterable,-RawPresentable,-메모리구조) 포스트에서 보았던 RawRepresentable 프로토콜의 선언을 예시로 보겠습니다.  

    ```swift
    public protocol RawRepresentable {
    associatedtype RawValue

    init?(rawValue: Self.RawValue)

    var rawValue: Self.RawValue { get }
    }
    ```  

    여기서 Self를 발견할 수 있습니다. 이 경우는 프로토콜 자신이 아니라 프로토콜을 채택한 타입을 뜻하는 것입니다. 또한 실제로 구현을 할때는 Self를 모두 걷어내야만 합니다.

1. Class : 메소드 시그니처에서 결과값으로만 사용이 가능합니다. 이 경우 해당 인스턴스의 타입 자체를 나타냅니다. 메소드 몸체에서는 사용할 수 없습니다.

```swift
func someFunc() -> Self {
    return self
}
```

> 구조체나 열거형에서는 Self를 사용할 수 없습니다.

---  

간단하면서도 궁금할 수도 있는 주제에 대해 알아보았습니다. 다음에 더 흥미로운 주제를 가지고 살펴보도록 하겠습니다!