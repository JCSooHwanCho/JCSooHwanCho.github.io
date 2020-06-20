---
layout: post
title: dynamicMemberLookup, dynamicCallable
comments: true
tags: [Swift,Apple]
category: [SwiftBasics]
---  

Swift는 기본적으로 정적인 언어입니다. 강력한 타입시스템을 가지고 컴파일 타임에 많은 것을 강제합니다. 이러한 점은 실제 프로그램을 실행하지 않고도 오류를 어느정도 거를 수 있게 만들어 준다는 점에서 분명한 장점이지만, 일부에서 발생하는 컴파일 에러 때문에 프로그램 전체를 실행하지 못하게 되거나 비정형 데이터 처리가 쉽지 않다는 점에서는 단점 또한 있습니다. 또한 스위프트보다 더 많은 사용자를 가지고 있는 동적 언어들이 제공하는 여러 유용한 라이브러리를 함께 사용하기 어려워지는 단점 또한 있었습니다.  
그래서 스위프트에서는 동적 언어의 특성을 정적인 형태로 사용할 수 있도록 돕는 기능들을 여러 버전에 걸쳐서 점진적으로 추가하게 되었습니다. 오늘은 이러한 기능들에 대해서 알아보도록 하겠습니다.

* **DynamicMemberLookup**  
  DynamicMemberLookup은 swift 4.2에 처음으로 추가된 기능입니다. 이 attribute는 타입에 붙이게 되면 다음과 같은 subscript를 구현할 것을 요구합니다.
  
  ```swift
  // 반환 값은 미정, 인자는 ExpressibleByStringLiteral이면 된다.
  // 외부 라벨은 고정이며, 내부 라벨은 이름을 자유롭게 지정할 수 있다.
  // 다만, 외부라벨과 내부라벨을 둘 다 써줘야 하며, 생략해서 하나만 써주면 오류가 난다. 이것은 나중에 수정되면 좋겠다.
  subscript(dynamicMember member: String) -> ??? 
  ```  

  이것을 구현하게 되면 .문법(dot syntax)로 런타임에 동적으로 프로퍼티를 참조하는 기능을 만들 수 있습니다. 아주 간단한 예를 들어보겠습니다.

  ```swift
  @dynamicMemberLookup
  struct Sample {
    var name: String

    subscript(dynamicMember member: String) -> String {
        if member == "myName" {
          return name
        } else {
          return "unknown"
      }
    }
  }

  let s = Sample(name: "Rhyno")

  print(s.myName) // Rhyno
  ```  

  분명 Sample 객체는 myName이라는 프로퍼티를 가지고 있지 않습니다. 하지만 이는 정상적으로 컴파일이 되며, myName이라는 이름을 가지고 Sampe의 name 프로퍼티를 정상적으로 참조하고 있다는 것을 알 수 있습니다. 여기서 myName이 아닌 다른 프로퍼티를 주면 어떻게 될까요?

  ```swift
  print(s.anotherName) // unknown
  ```  

  이 경우에는 조건문에서 else에 해당하는 값을 반환하고 있음을 알 수 있습니다. 주어진 문자열을 가지고, 원하는 값을 동적으로 찾을 수 있게 된것입니다. 그래서 dynamicMemberLookup입니다.

  물론 이런 기능을 구현하는 것이 불가능한 것은 아니였습니다. 하지만 subscript를 이용하는 기존 방법이였다면 다음과 같이 썼어야만 합니다.

  ```swift
    print(s[dynamicMember: "myname"])
  ```  

  위 기능은 이러한 번거로움을 줄이기 위한 문법적 설탕(syntactic sugar)에 해당합니다. 또한 여기에 Swift의 옵셔널을 덧붙이면, 기존 동적인 언어를 좀 더 스위프트스럽게 융화시킬 수 있게 됩니다. 하지만 그래도 여전히 정적인 언어이기 때문에 반환값은 언제나 동일해야 된다는 문제가 있습니다. 그렇다고 동일한 선언을 반환값에 따라 오버로딩하게 되면 타입 추론이 무력화되는 단점이 있습니다. 따라서 이 기능을 좀 더 활용하기 위해서는 서로 다른 타입을 동일하게 활용할 수 있도록 타입을 지운 래퍼(Wrapper)를 활용할 필요가 있습니다. 그런데, 이러한 방식은 Objective-C에서 모든 객체를 id로 다루던 것과 유사한 상황을 만듭니다. 그리고 이것이 실제로 동적 언어들이 객체를 다루는 방식입니다. dynamicMemberLookup을 활용해서 이를 표현하면 대략 이런식입니다.

  ```swift
  // 예제 출저: https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md

  @dynamicMemberLookup
  struct PyVal {
    ...
    subscript(dynamicMember member: String) -> PyVal {
        get {
            let result = PyObject_GetAttrString(borrowedPyObject, member)!
            return PyVal(owned: result)
        }
        set {
            PyObject_SetAttrString(borrowedPyObject, member,
                                newValue.borrowedPyObject)
        }
    }
  }
  ```

  이러한 방식은 동적 언어를 스위프트 사이의 간격을 줄여주어, 스위프트가 동적 언어의 여러 유용한 라이브러리를 쉽게 사용할 수 있는 기회를 제공해줍니다. 그렇지만 스위프트가 가진 강력한 타입시스템을 무력화시킨다는 단점이 있습니다. 그래서 스위프트 5.2에서는 문자열뿐 아니라 키패스로 dynamicMemberLookup을 사용할 수 있는 기능이 추가되었습니다. 키패스는 문자열보다 타입 정보를 많이 가지고 있기 때문에, 좀 더 정적인 방법으로 사용이 가능해집니다.

  ```swift
    // 예제 출저: https://github.com/apple/swift-evolution/blob/master/proposals/0252-keypath-dynamic-member-lookup.md
  @dynamicMemberLookup
  struct Lens<T> {
    let getter: () -> T
    let setter: (T) -> Void

    var value: T {
        get {
        return getter()
        }
        nonmutating set {
        setter(newValue)
        }
    }

    subscript<U>(dynamicMember keyPath: WritableKeyPath<T, U>) -> Lens<U> {
        return Lens<U>(
            getter: { self.value[keyPath: keyPath] },
            setter: { self.value[keyPath: keyPath] = $0 })
    }
  }
  ``` 

* **dynamicCallable**  
  dynamicMemberLookup이 동적으로 프로퍼티를 찾을 수 있는 기능을 제공한다면, dynamicCallable은 해당 객체를 임의의 인자에 대해서 메소드처럼 호출할 수 있는 기능을 제공해줍니다. 즉, 객체지만 함수처럼 사용할 수 있는 것입니다. 이는 Callable Object라는 것으로, C++ 의 Functor, C#의 delegate 등 다른 언어에서 이미 지원하고 있던 기능입니다.  

  어떤 객체가 dynamicCallable이 되기 위해서는 @dynamicCallable attribute를 붙여야 하며, 다음 두개의 메소드 중 최소 한개는 구현해야 합니다.  

  ```swift
  func dynamicallyCall(withArguments: <#Arguments#>) -> <#R1#>
  // `<#Arguments#>`는 ExpressibleByArrayLiteral을 채택한 타입이여야 합니다.
  // `<#Arguments#>.ArrayLiteralElement`와 `<#R1#>` 는 어떤 타입도 올 수 있습니다.

  func dynamicallyCall(withKeywordArguments: <#KeywordArguments#>) -> <#R2#>
  // `<#KeywordArguments#>`는 `ExpressibleByDictionaryLiteral`을 채택해야 합니다.
  // `<#KeywordArguments#>.Key`는 `ExpressibleByStringLiteral`을 채택한 타입이어야 합니다.
  // `<#KeywordArguments#>.Value`와 `<#R2#>`는 어떤 타입도 올 수 있습니다.
  ```  
  첫번째 메소드는 라벨이 없이 인자만 나열된 경우에 사용하고, 두번째 메소드는 인자중 한개 이상이 라벨을 가지는 경우에 사용됩니다. 이때, 라벨이 없는 인자는 빈 문자열("")을 라벨로 가지게 됩니다. 그런데 만약 인자가 Dictionary 타입인 경우, 키의 중복을 허용하지 않으므로 값이 유실될 수 있습니다. 따라서 인자 타입으로 추천되는 타입은 [KeyValuePairs](https://developer.apple.com/documentation/swift/keyvaluepairs)입니다. 또한 저 메소드들에도 @discardableResult이나 throws 수식어를 붙일 수 있는데, 동적으로 호출되는 메소드 역시 이 특성을 그대로 따라갑니다.  

  ```swift
  @dynamicCallable
  struct Callable {
    func dynamicallyCall(withArguments args: [Int]) ->  Int { 
        return args.count 
    }
  }
  let c1 = Callable()
  c1() // `c1.dynamicallyCall(withArguments: [])` 로 해석됩니다.
  c1(1, 2) // `c1.dynamicallyCall(withArguments: [1, 2])` 로 해석됩니다.
  c1(a: 1, 2) // error! 'withKeywordArguments:' 메소드가 구현되지 않았습니다.

  @dynamicCallable
  struct KeywordCallable {
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Int {
        return args.count
    }
  }
  let c2 = KeywordCallable()
  c2() // `c2.dynamicallyCall(withKeywordArguments: [:])` 로 해석됩니다.
  c2(1, 2) //`c2.dynamicallyCall(withKeywordArguments: ["": 1, "": 2])` 로 해석됩니다.
  c2(a: 1, 2) //`c2.dynamicallyCall(withKeywordArguments: ["a": 1, "": 2])` 로 해석됩니다.

  @dynamicCallable
  struct BothCallable {
    func dynamicallyCall(withArguments args: [Int]) -> Int { 
        return args.count 
    }

    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Int {
        return args.count
    }
  }
  let c3 = BothCallable()
  c3() //`c3.dynamicallyCall(withArguments: [])` 로 해석됩니다.
  c3(1, 2) // `c3.dynamicallyCall(withArguments: [1, 2])` 로 해석됩니다.
  c3(a: 1, 2) // `c3.dynamicallyCall(withKeywordArguments: ["a": 1, "": 2])`로 해석됩니다.
  ```
  dynamicCallable 역시 dynamicMemberLookup의 특성을 그대로 따라갑니다. 인자로 들어오는 배열 혹은 키-값 쌍은 모두 동일한 타입이여야 한다는 제한이 걸리기 때문에, 실제 동적 언어와 함께 사용하기 위해서는 타입 정보를 가지고 있지 않은 래퍼를 사용해야 합니다. 이러한 특성은 파이썬이 메소드를 호출하는 방식과 거의 동일합니다.그래서 dynamicMemberLookup을 함께 써서 파이썬의 객체를 Swift로 모델링하면 대략 이렇게 됩니다.

  ```swift
  // 예제 출저: https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md

  @dynamicCallable @dynamicMemberLookup
  struct PythonObject {
    
    // 파이썬은 키워드가 있는 인자와 없는 인자를 섞어 쓸 수 있습니다.
    @discardableResult
    func dynamicallyCall(
        withKeywordArguments: KeyValuePairs<String, PythonObject>
    ) -> PythonObject { ... }

    // 키워드 없는 인자만을 쓸 때는, 다음 메소드를 구현해 놓으면 키-값 쌍으로의 변환을 방지할 수 있습니다.
    @discardableResult
    func dynamicallyCall(withArguments: [PythonObject]) -> PythonObject { ... }

    // `@dynamicMemberLookup`의 요구사항입니다.
    subscript(dynamicMember member: String) -> PythonObject {...}
    
    // ... other stuff ...
  }
  ```  

* **우선순위**  
  dynamicMemberLookup이나 dynamicCallable을 사용할 때, 기존 멤버나 메소드 선언과 충돌하는 경우가 있을 수 있습니다. 이때는 무조건 기존 멤버나 메소드 선언이 우선합니다. 즉 동적으로 멤버나 메소드를 찾는 것은 언제나 가장 마지막 후보라는 것입니다. 따라서 이를 사용할 때, 기존 선언과의 충돌을 최소화할 수 있도록 해야 합니다. 

  ---  

  스위프트는 계속해서 더 많은 영역을 커버할 수 있도록 발전하고 있습니다. 이러한 발전을 통해서 스위프트가 애플 플랫폼을 넘어서 더 많은 곳에서 사용되기를 기대해봅니다.