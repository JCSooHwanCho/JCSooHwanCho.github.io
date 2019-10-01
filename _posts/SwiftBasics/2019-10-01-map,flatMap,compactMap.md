---
layout: post
title: map, flatMap, compactMap
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

swift는 Collection 타입에 사용할 수 있는 여러가지 고차 함수를 제공합니다. 이 중에서 이름이 비슷한 함수인 map, flatMap, compactMap에 대해서 알아보도록 하겠습니다.

세 함수는 모두 swift의 모든 Collection 타입의 기반이 되는 Sequence 프로토콜에 선언되어 있습니다. 따라서 모든 Collection 타입이 이를 이용할 수 있고 Collection의 특성에 따라 추가적인 관련 함수를 제공하기도 합니다. 여기서는 Sequence에 선언된 함수들에 대해서만 다뤄보도록 하겟습니다.

1. [map(_:)](https://developer.apple.com/documentation/swift/sequence/3018373-map) : (Element)->(T) 형태의 클로저를 인자로 받아서 해당 Collection의 모든 원소에 대해 특정 변환 연산을 적용한 Array 반환합니다. 어떤 Collection에 대해 적용되어도 map의 결과값은 Array입니다. 

    map의 선언은 다음과 같습니다.

    ```swift
    func map<T>(_ transform: (Self.Element) throws -> T) rethrows -> [T]
    ```  

    인자로 주어지는 transform은 Collection의 원소 타입을 인자로 받아서, 또 다른 타입의 값을 반환합니다. 즉 Element 타입에서 T타입으로의 변환을 일어나게 합니다. 하지만 원본을 변화시키지는 않습니다.

2. [flatMap(_:)](https://developer.apple.com/documentation/swift/sequence/2905332-flatmap) : map과 비슷하지만, 인자로 주어지는 변환 연산이 Sequence를 채택한 타입을 반환하며, 여기서 반환된 모든 Sequence들을 합쳐 하나의 Array로 만들어 반환합니다.

    flatMap의 선언은 다음과 같습니다.

    ```swift
    func flatMap<SegmentOfResult>(_ transform: (Self.Element) throws -> SegmentOfResult) rethrows -> [SegmentOfResult.Element] where SegmentOfResult : Sequence
    ```  

    인자로 주어진 transform 클로저는 Collection의 원소를 받아서, 또 다른 형태의 Sequence를 만들어 반환하며, 이렇게 반환된 Sequence들은 순서대로 연결되어 하나의 배열이 됩니다. 만약 Element가 배열이였고, 별다른 변환 없이 이를 반환한다면 이는 '배열의 배열'을 '배열'로 만드는 효과가 있습니다. 이는 다차원 배열의 차원을 한단계 낮춘다고 볼 수 있고, 그렇기 때문에 함수의 이름이 flatMap이 된 것입니다.

3. [compactMap(_:)](https://developer.apple.com/documentation/swift/sequence/2950916-compactmap) : 역시 map과 비슷하지만, 변환의 결과가 optional이고 이 중 nil이 아닌 것만 Array에 담겨 반환된다는 점이 차이점입니다.

    compactMap의 선언은 다음과 같습니다.

    ```swift
    func compactMap<ElementOfResult>(_ transform: (Self.Element) throws -> ElementOfResult?) rethrows -> [ElementOfResult]
    ```  

    transform 클로저의 반환 값이 optional이야 하고, 이중에서 변환 결과가 nil일 경우 최종 결과에 포함되지 않는다는 점이 핵심입니다.

    > compactMap은 Swift 4.1 이전까지는 flatMap이라는 이름을 가지고 있었습니다. 2번에서 본 flatMap과는 인자로 구분되는 오버로딩 관계였지만, 명확한 구분을 위해 Swift 4.1에서 nil을 필터링하는 flatMap 함수의 이름이 compactMap으로 변경되었습니다.

보면 세 함수의 차이는 크지 않고, flatMap과 compactMap은 모두 map을 이용하여 구현할 수 있습니다. 하지만 코드를 단순하게 하고 의미를 명확히 하기 위해 flatMap과 compactMap을 적절한 곳에 사용하는 것이 좋습니다.

추가적으로 RxSwift에도 같은 연산자들이 있는데, Observable 역시 Sequence의 일종으로 볼 수 있기 때문에 의미가 동일합니다.