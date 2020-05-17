---
layout: post
title: Foundation이 제공하는 특별한 Collection들
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

이번 포스트에서는 Foundation이 제공하는 Collection중에서 특별한 기능을 가진 Collection에 대해서 알아보도록 하겠습니다. 여기서 특별한 Collection이라고 하는 것은 Swift 표준 라이브러리가 제공하는 Collection인 Array와 Set, Dictionary와 이에 대응되는 Foundation 클래스인 NSArray, NSSet, NSDictionary 그리고 이들의 Mutable 버젼 이외의 추가적으로 제공되는 Collection들을 이야기합니다. 특별한 Collection들은 기존 Collection들에서 추가적인 속성을 더한 것들입니다.

* **NSCountedSet**  
  Set은 Set인데, 중복되는 원소가 1개이상 들어갈 수 있는 Set입니다. NSMutableSet을 상속받아 만들어져서 기본적인 속성은 NSMutableSet과 동일합니다. 추가적인 이해를 위해 좀 더 살펴보면, NSSet 계열은 원소의 타입이 Any라 어떠한 원소든 타입에 관계없이 넣을 수 있습니다. 다만 값타입의 경우는 Hashable을 채택하지 않으면 중복 체크가 불가능하여 중복된 원소도 그대로 들어가게 됨에 유의해야 합니다. 특징적인 인터페이스는 [count(for:)](https://developer.apple.com/documentation/foundation/nscountedset/1408658-count)로, 특정 객체가 존재하는 수를 반환합니다. 이때 2 이상의 수가 반환된다고 해서 객체가 2개 이상 존재하는 것이 아님에 유의해야 합니다. 동일한 객체는 여전히 1개만 존재할 수 있습니다. 단지 카운트 개념이 추가되어서 카운트가 0이 되었을때만 실제로 Set에서 원소가 사라지는 것입니다.
  
  ```swift
  let c = NSCountedSet(array: [1,2,3,3, "name"])

  print(c.allObjects) // [3,2,name,1], Set의 원소 순서는 보장되지 않습니다
  print(c.count(for: 3)) // 2

  ```  
  > Foundation의 Collection 객체를 실험하기 위한 코드를 Playground에서 할 때 주의할 점이 하나 있습니다. Foundation의 Collection에 클래스를 집어넣을때, 넣는 클래스가 반드시 NSObject의 서브클래스여야 한다는 것입니다. 실제 코드에서는 이러한 제한이 없는데, 이는 NSObject의 자식타입이 아닌 객체를 NSObject의 서브클래스로 래핑해주는 과정이 들어가기 때문입니다. 플레이그라운드에서는 타입 처리 과정이 실제 프로덕션 코드와 달라, 이를 래핑해줄 수 없어 일어나는 일입니다. 

* **NSOrderedSet, NSMutableOrderedSet**  
   원소의 순서를 가지는 Set입니다. Array처럼 인덱스 기반의 참조와 정렬등이 가능하면서도 Set의 중복원소 방지 특성이 적용됩니다. 어떻게 이런게 가능한가 하면 단순히 Array와 Set을 동시에 유지하면서 둘 사이의 일관성을 유지하는 것입니다. 만약 Array를 사용하면서 원소가 중복되면 안되고, 특정 원소의 존재 여부를 자주 파악해야 한다면 이를 Array 대신 사용하는 것을 고려해볼만 합니다. 대신 Set에 원소가 들어가야 하기 때문에 원소의 타입은 반드시 Hashable해야만 합니다. 마냥 좋아보일 수도 있지만, 정보를 이중으로 유지하기 때문에 메모리 소모량이 좀 더 높고 삽입, 삭제에 어쩔 수 없이 추가 비용이 들 수 밖에 없다는 점과 모든 원소가 Any로 캐스팅되서 들어가기 때문에 컴파일 타임에 타입 안정성을 보장할 수 없다는 단점을 고려할 필요가 있습니다.  

   ```swift
    let c = NSMutableOrderedSet(array: [1,2,3,3, "name"])

    print(c.array) // [1, 2, 3, "name"]
    print(c.set) // [AnyHashable(3), AnyHashable(1), AnyHashable(2), AnyHashable("name")]
   ```  

* **NSCache**  
   NSMutableDictionary와 유사하지만, 용량과 엔트리 수의 제한을 줄 수 있습니다. 계산하거나 로딩하는데 비용이 많이 드는 객체를 여기에 두고 사용하다가, 메모리가 모자랄 때 시스템에 의해 자동적으로 내용물이 사라질 수 있도록 관리해줍니다. 실제로 어떻게 동작하는 지는 시스템이 NSCache를 어떻게 구현했냐에 따라 결정되어 어플리케이션 개발자가 건드릴 수 없으나, 용량제한과 엔트리수 제한을 주어 이에 대한 힌트를 제공할 수 있습니다. 또한 사용자 측에서 lock을 걸지 않고도 스레드 안전하게 사용할 수 있으며, 키값이 복사되지 않아 NSCopying을 채택하지 않은 클래스도 Key로 사용할 수 있다는 장점이 있습니다. 대신, Key와 Value 모두 참조 타입(AnyObject)이여야 한다는 제한이 있습니다. 또한 문서에는 나와있지 않지만, 실제 제대로 된 Key로 활용하기 위해서는 NSObject를 상속 받아서 isEqual을 재정의해야만 합니다. 그렇지 않으면 처음 등록할 때 사용한 key객체 이외에 같은 값을 가진 다른 객체로는 참조가 불가능해집니다.(Hashable을 채택하는 것만으로는 안됩니다.) 보통은 String이나 URL같은 객체를 NSString이나 NSURL등으로 캐스팅해서 사용하기에 문제가 안되겠지만, 커스텀한 key 객체를 만들 경우는 이러한 경우를 잊지 마시기 바랍니다.  

   ```swift
    let c = NSCache<NSString, NSNumber>()

    c.setObject(10 as NSNumber, forKey: "ten" as NSString)
    c.setObject(20 as NSNumber, forKey: "twelve" as NSString)

    print(c.object(forKey: "ten" as NSString)) // Optional(10)
   ```  

* **Pointer Collection**  
  NSPointArray, NSHashTable, NSMapTable은 포인터를 다루는 데 특화된 Collection들입니다. 즉, 참조를 다루는 데 특화되어 있습니다. Array, Set, Dictionary등에 Class를 담아도 되는데 왜 새로운 객체를 사용해야하는 지 궁금할 수 있는데, 이들의 의의는 weak 참조를 저장할 수 있는 능력을 가지고 있다는 것입니다. 다만 NSPointerArray의 경우는 UnsafeMutableRawPointer를 쓰기 때문에 자동 메모리 관리와 타입 안정성을 전혀 기대할 수 없고, NSHashTable, NSMapTable은 역시 NSObject를 상속 받아서 isEqual을 구현해야하는 점이 단점입니다. 즉, Swift스럽지 않습니다. 따라서 대부분 swift 오픈소스에서는 이를 별도로 구현해 놓는 경우가 많습니다.

  ```swift
    let number = 5 as NSNumber

    // NSPointerArray 사용
    let arr = NSPointerArray.weakObjects() // 약한 참조를 가지는 Pointer배열
    let p = Unmanaged.passUnretained(number).toOpaque() // 약한 참조를 가지는 포인터

    arr.addPointer(p)

    print(arr.allObjects) // [5]

    // NSHashTable사용
    let hashTable = NSHashTable<NSNumber>.weakObjects()
    let number = 5 as NSNumber
    hashTable.add(number)

    print(hashTable.allObjects) // [5]

    // NSMapTable 사용
    let key = "five" as NSString
    let mapTable = NSMapTable<NSString, NSNumber>()
    mapTable.setObject(number, forKey: key)

    print(mapTable.object(forKey: key)) // Optional(5)

  ```

* **정리**  
  Foundation이 제공해주는 Collection들은 우리 생각보다 다양하게 존재합니다. 하지만 다른 언어보다 댜앙하지 않을 뿐 아니라 Objective-C와의 호환성 때문에 Swift로 쓰면 쉽게 이해하기 어려운 제한들이 생기기도 합니다. 이러한 자료구조의 필요성과 제한들을 알고 해소할 수 있는 것 또한 개발자에게 필요한 능력일 것이라 생각됩니다.

  ---  

  > 참고 자료  
  > [MacroSantaDev - Swift Arrays Holding Elements With Weak References](https://marcosantadev.com/swift-arrays-holding-elements-weak-references/)
