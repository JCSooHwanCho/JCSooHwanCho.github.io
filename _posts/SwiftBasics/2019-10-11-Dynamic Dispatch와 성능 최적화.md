---
layout: post
title: Dynamic Dispatch와 성능 최적화
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

많은 객체 지향 언어들이 메소드와 프로퍼티들을 오버라이드 할 수 있도록 허용합니다. Swift도 예외는 아닙니다. 그런데 이렇게 오버라이드를 할 경우, 프로그램은 실제 호출할 함수가 어떤 것인지 결정하는 과정이 필요합니다.

```swift
class Parent {
    func someMethod() { 
        //... 
    }
}

class Child: Parent {
    override func someMethod() {
        // ...
    }
}


let object: Parent = Child()

object.someMethod() // Parent의 someMethod를 호출할 것인가, Child의 someMethod를 호출할 것인가?
``` 

이를 해결하는 방법에는 두가지가 있습니다.

1) Static Dispatch(Direct Call): 변수의 명목상 타입에 맞춰서 메소드와 프로퍼티를 참조합니다. 이 경우 참조될 요소를 컴파일 타임에 결정할 수 있고, 실제로 그렇게 합니다. 위의 예제에서 Parent의 메소드를 호출하는 경우가 이에 해당합니다. 컴파일 타임에 결정이 끝나기 때문에 성능상의 이점이 있으나, 자식 클래스의 요소 호출하고 싶으면 명시적인 타입 캐스팅으로 변수를 자식 타입으로 만들어줘야 합니다. 따라서 프로그램이 다형성을 활용하기 어렵게 만드는 단점이 있습니다.  

2) Dynamic Dispatch(Indirect Call): 변수의 실제 타입의 맞춰서 메소드와 프로퍼티를 호출합니다. 코드상으로는 이것이 드러나지 않기 때문에 실제 참조될 요소는 런타임에 결정됩니다. 어떤 서브클래스가 들어와도 실제 타입에 맞는 요소를 참조하기 때문에 다형성 활용에 유리합니다. 다만, 런타임에 실제 참조할 요소를 찾는 과정이 있기 때문에(이 과정은 O(1)의 시간복잡도를 가지도록 구현되있습니다.) Static Dispatch보다 성능상에서 손해를 보게 된다는 단점이 있습니다. 

Swift에서는 Dynamic Dispatch를 채택하였습니다.(아닌 경우도 있습니다. 이 경우에 대해서는 이후 포스트를 통해 더 살펴보도록 하겠습니다.) 일반적인 경우에는 Dynamic Dispatch가 편리하긴 하지만, 성능을 신경써야 하는 코드에서는 dynamic Dispatch의 오버헤드도 거슬릴 수 있습니다. 또한 Dynamic Dispatch의 가능성이 있는 코드에서는 컴파일러의 최적화가 제한되어버립니다. 따라서 swift는 Dynamic Dispath가 필요하지 않을 경우에 사용할 수 있는 3가지의 성능 최적화 방법을 제시합니다. 여기서는 이 방법들을 살펴보도록 하겠습니다.  

> 이 포스트는 다음 블로그 포스트를 참고하여 작성되었습니다.  
> [Increasing Performance by Reducing Dynamic Dispatch](https://developer.apple.com/swift/blog/?id=27)  

1. 오버라이드 될 필요가 없는 요소들에는 final 키워드를 붙이세요.  
    final 키워드를 붙여서 선언된 클래스, 메소드, 프로퍼티는 오버라이드 할 수 없게 됩니다. 이렇게 되면 컴파일러는 Dynamic Dispatch를 사용하지 않아도 됨을 알고 이 부분을 최적화 해 줄수 있게 됩니다.

2. private 키워드를 붙여서 선언하게 되면 해당 요소는 한 파일 내에서만 참조되는 것이 자동으로 보장이 됩니다. 따라서 한 파일내에 해당 요소에 대한 오버라이드가 없는 경우 컴파일러가 이를 자동으로 Direct Call로 바꿔줄 수 있습니다. 

3.  WMO(Whole Module Optimization)을 사용한다면 internal 선언만으로 final을 추론해낼 수 있습니다.(물론 모듈 내에 오버라이드가 없는 경우 한정입니다.)  
   swift는 기본적으로 모듈을 이루는 파일들을 개별적으로 컴파일 하기 때문에 다른 파일에서 오버라이딩이 일어났는지 알 수 없습니다. 하지만 WMO를 활성화 하면 모듈 전체가 하나의 덩어리로 취급되어 컴파일 되기 때문에 이를 확인하고 더 강력한 추론을 할 수 있습니다. 이는 모듈 내의 internal 선언은 모듈 바깥에 드러나지 않아 오버라이딩이 불가능 하다는 것이 보장되기 때문입니다.

---  

이 주제는 Swift가 다양한 접근 지정자를 제공하는 것과 연관되어 있습니다. 개인앱 단위에서 접근 지정자를 쓸 이유는 많지 않지만 최적화에 민감한 앱을 작성한다면 이런 것 역시 간과할 수는 없을 것입니다.