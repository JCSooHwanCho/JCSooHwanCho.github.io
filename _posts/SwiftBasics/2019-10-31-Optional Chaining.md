---
layout: post
title: Optional Chaining
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

이번 포스트에서는 Optional을 이용한 테크닉인 Optional Chaining의 사용법과 원리에 대해서 알아보도록 하겠습니다.  

* **Optional Chaining**  
  옵셔널 체이닝의 문법은 아래와 같습니다.
  
  ```swift  
  someOptional? // 이 뒤로 여러 Optional이 이어질 수 있습니다.
  ```  

  위와 같이 사용하면, 옵셔널을 언래핑하지 않아도 옵셔널 내부의 값에 접근할 수 있습니다. 만약 옵셔널이 nil이라면, 뒤에 이어지는 모든 식은 실행되지 않고, 전체 식의 값은 nil이 됩니다. 이를 다시 생각해보면, 옵셔널 체이닝이 포함된 식의 전체 값 역시 옵셔널이라는 것입니다.  

  ```swift  
  class someValue {
      func performAction() -> Bool {
          return true  // 기본값으로 true를 반환한다 가정
      }
  }
  class SomeClass {
      let property: someValue // non Optional
  }

  var c: SomeClass?
  var result = c?.property.performAction() // Optional(true)
  ```  

  이를 언래핑해서 계산하기 위해서는 다음과 같이 코드를 짜야합니다.  

  ```swift
    var result: Bool?
  // 1. 강제 해제 코드
   result = c!.property.performAction() // c가 nil일 경우, 런타임 에러가 난다.
  // 2. 선택적 해제 코드

  if let unwrappedC = c {
    result = unwrappedC.property.performAction()
  }
  ```  

  이렇게 옵셔널 체이닝으로 받은 결과를 통해서 원본 값에 직접 대입을 수행하여 원본 값을 바꾸는 작업이 가능합니다. 다만, 체이닝 결과가 nil이면, 우측의 식은 실행되지 않습니다.

  ```swift
  func someFunctionWithSideEffects() -> Int {
    return 42  // 실제 사이드 이펙트가 있지는 않습니다.
    }
    var someDictionary = ["a": [1, 2, 3], "b": [10, 20]]

    someDictionary["not here"]?[0] = someFunctionWithSideEffects()
    // someFunctionWithSideEffects는 실행되지 않습니다.

    someDictionary["a"]?[0] = someFunctionWithSideEffects()
    // someFunctionWithSideEffects가 실행됩니다.
    // someDictionary is now ["a": [42, 2, 3], "b": [10, 20]]
  ```  

* **Optional Chaining의 원리**  
  이러한 체이닝은 어떻게 가능할까요? 필요한 기능은 대략 정리해보면 다음과 같습니다.  

  1. 대상은 옵셔널이다.   
  
  2. 값이 있으면 원래의 값처럼 사용할 수 있어야 한다.  
  
  3. 중간에라도 값이 없으면(nil이면) 이후의 문장들은 전혀 실행되지 않아야 한다.  
  
  4. 여러번 겹쳐서 쓸 수 있어야 한다.  

  [지난 포스트](../2019-10-30-Monad와-Swift/)에서 옵셔널은 모나드의 일종이라고 하였습니다. 모나드에서 사용할 수 있는 도구인 map을 활용하면 이러한 과정을 수행할 수 있을 것 같습니다. 

  ```swift
  var result = c.map({ $0.property }).map({ $0.performAction() }) // Optional(true)
  ```  

  잘 작동하는 것 처럼 보이는데, 문제가 있습니다. 지금 performAction()은 옵셔널 타입을 반환하는 메소드가 아닙니다. 하지만 만약 아래 코드처럼 메소드가 옵셔널을 반환한다면 문제가 생깁니다.

  ```swift
  extension SomeClass {
      func loadDataFromNetwork() -> Data? // 네트워크에서 Data요청을 받아오는 메소드
  }

  var result = c.map({ $0.loadDataFromNetwork() }) // Optional(Optional(Data))
  ```  

  이렇게 옵셔널이 중첩되게 됩니다. 이렇게 되면 원하는 대로 옵셔널 여부를 사용자가 의식하고 써야하고, 자연스러운 방법으로 체이닝이 불가능해지는 문제가 생깁니다. 이를 해결할 수 있는 도구가 바로 flatMap입니다. flatMap은 옵셔널이 중첩되지 않도록 해줍니다. 

  ```swift
  var result = c.flatMap({ $0.loadDataFromNetwork() }) // Optional(Data)  
  ```  

  따라서 옵셔널 프로퍼티나 옵셔널을 반환하는 메소드에는 flatMap을, 그렇지 않은 경우에는 map을 쓰면 됩니다. 하지만 swift는 반환 값이 옵셔널일 경우 자동으로 옵셔널로 래핑을 해주기 때문에, map도 flatMap으로 대체할 수 있습니다. 
  
  ```swift
  var result = c.flatMap({ $0.property }).flatMap({ $0.performAction() }) // Optional(true)
  ```
  결론적으로 옵셔널 체이닝은 flatMap을 통해 옵셔널을 손쉽게 사용하게 해주는 문법적인 부가기능(Syntatic Sugar)입니다. 이렇게 중첩해서 쓸 수 있다는 것 자체도 모나드의 중요한 특성 중 하나입니다.  

  ---  

  > 참고 자료  
  > [Swift Language Guide - Optional Chaining](https://docs.swift.org/swift-book/LanguageGuide/OptionalChaining.html)  
  > [Swift Language Reference - Optional Chaining Expression](https://docs.swift.org/swift-book/ReferenceManual/Expressions.html)  
  > [Optional Binding == flatMap Example](https://gist.github.com/cobalamin/dd88a1755323798a1dfe)  
  >[Swift Optional Implementation](https://github.com/apple/swift/blob/master/stdlib/public/core/Optional.swift)  