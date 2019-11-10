---
layout: post
title: Iterator
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

개발을 할때는 단순한 배열부터 여러 복잡한 트리까지 다양한 자료구조를 사용합니다. 이러한 자료구조들은 내부 구조가 다르기 때문에 사용 방법도 다릅니다. 이렇게 서로 다른 자료구조를 동일한 인터페이스로 순회할 수 있는 방법을 제공해주기 위해 만들어진 것이 반복자(iterator)라는 개념입니다. 반복자는 내부 구조를 노출하지 않고, 원소들을 순차적으로 접근할 수 있도록 만들어줍니다. 이 반복자 개념은 어려 언어에서 각자의 방법으로 구현되어 사용되는데, 여기서는 Swift의 반복자를 살펴보도록 하겠습니다. 

* **IteratorProtocol**  
  Swift는 반복자를 위해 [IteratorProtocol](https://developer.apple.com/documentation/swift/iteratorprotocol)를 제공합니다. Iterator는 Sequence프로토콜과 같이 사용하는 것을 전제로 하고 있는데, Sequence 프로토콜은 실제로 Iterator를 typealias로 필수적으로 지정해주도록 하고 있습니다.  

  IteratorProtocol의 선언은 대략 다음과 같습니다.  

    ```swift
    protocol IteratorProtocol {
        associatedtype Element

        mutating func next() -> Self.Element?
    }
    ```  

    swift의 Iterator는 next() 메소드를 통해서 다음 원소를 얻어내고, 다음 원소가 존재하지 않는다면 nil을 반환합니다. 다음 상태를 찾기 위해서는 현재의 상태를 기록해 놓을 필요가 았고, 이는 IteratorProtocol을 채택하는 타입에서 자체적으로 구현해야 합니다.  

    > 반복자는 타입과 그 안의 값이 같으면 동일한 반복자로 취급할 수 있기 때문에, 주로 구조체로 구현됩니다.
    
    Sequence의 선언부도 살펴보겠습니다. 전체를 볼 수는 없고, 반복자에 관련된  부분만 살펴보겠습니다.  

    ```swift
    protocol Sequence {
        associatedtype Element where Self.Element == Self.Iterator.Element
        associatedtype Iterator: IteratorProtocol

        func makeIterator() -> Self.Iterator
    }
    ```  

    makeIterator()를 통해 반복자 객체를 얻어내고, 이 반복자의 next() 메소드를 통해 Sequence의 원소를 처음부터 끝까지 순회하게 됩니다. 마지막 원소를 얻은 다음에 next()를 호출하게 되면 nil 값을 반환하게 되고, 이것으로 시퀀스의 마지막을 체크할 수 있습니다.  

    이렇게 구현된 반복자가 주로 사용되는 것은 for...in 루프입니다.  

    ```swift
    let animals = ["Antelope", "Butterfly", "Camel", "Dolphin"]
    for animal in animals {
        print(animal)
    }
    // Prints "Antelope"
    // Prints "Butterfly"
    // Prints "Camel"
    // Prints "Dolphin"
    ```  

    이 루프가 다음과 같이 바뀝니다.  

    ```swift
    var animalIterator = animals.makeIterator()
    while let animal = animalIterator.next() {
        print(animal)
    }
    // Prints "Antelope"
    // Prints "Butterfly"
    // Prints "Camel"
    // Prints "Dolphin"
    ```  

    아니면 처음부터 반복자를 직접 사용하게 할 수도 있습니다.

---  
