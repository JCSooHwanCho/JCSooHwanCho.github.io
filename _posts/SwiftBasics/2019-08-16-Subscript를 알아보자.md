---
layout: post
title: Subscript를 알아보자
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

배열이나 딕셔너리 등의 콜렉션 타입을 쓰다보면 **[index]** 형태의 구문을 많이 쓰게 됩니다. 이러한 것을 Subscript(첨자) 라고 하며, Swift에서는 Subscript를 프로그래머가 이용할 수 있는 간편한 방법을 제공합니다. 오늘은 이 첨자에 대해서 알아보겠습니다.

> 이 글은 다음 문서를 기반으로 정리한 것임을 밝힙니다.
>  [Swift Language Guide - Subscript](https://docs.swift.org/swift-book/LanguageGuide/Subscripts.html)

Subscript는 콜렉션, 리스트 그리고 시퀀스 등에 별도의 getter,setter 메소드가 없이 인덱스를 통해 접근할 수 있도록 해줍니다. 
Subscript는 클래스, 구조체, 열거형에 적용할 수 있고, 입력 값의 타입에 따라 오버로딩도 가능합니다. (즉, 하나의 타입은 여러개의 Subscript를 가질 수 있습니다.) 또 Subscript는 한개 이상의 값을 입력 받도록 정의할 수도 있습니다. 

Subscript 문법은 다음과 같습니다.  

```swift
// T,R : 임의의 타입
subscript(index: T) -> R {
    get {
        // 적절한 값을 반환합니다.
    }
    set(newValue) { // newValue 의 타입은 T'입니다. 
        // 입력받은 값을 적절히 세팅합니다.
    }
}
``` 

> 계산 프로퍼티와 같이 setter에 파라미터를 명시적으로 제공하지 않을 경우, 기본 파라미터 이름은 newValue 입니다.

읽기 전용으로 설정하고 싶다면 다음과 같이 간략하게 쓸 수 있습니다.

```swift
subscript(index:  T) -> R {
    // 적절한 값을 반환합니다.
}
```

다음은 구구단표(Times Table)를 Subscript를 이용해서 구현한 예시입니다.

```swift
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3) 
print("six times three is \(threeTimesTable[6])") // "six times three is 18"
```

Subscript가 보통 콜렉션, 리스트, 시퀀스에서 쓰이긴 하지만, Subscript의 정확한 의미는 프로그램의 현재 상황에 따라 달라집니다. Swift의 Dictionary가 그 좋은 예입니다.

```swift
var numberOfLegs = ["spider": 8, "ant": 6, "cat": 4] 
numberOfLegs["bird"] = 2
```

다음 Dictionary의 Subscript는 String -> Int 형태임을 쉽게 유추할 수 있습니다.

Subscript는 여러개의 인자를 가질 수 있고, 인자 타입에도 제한이 없습니다. 가변 인자(variadic parameter)도 사용할 수 있고 기본 값을 제공할 수도 있습니다. 다만 [inout 파라미터](https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#ID545)는 사용할 수 없습니다.  

앞에서 언급했듯이 Subscript는 여러 개가 있을 수 있고 주어진 인자의 타입에 맞게 자동으로 타입 추론이 이루어 집니다. 

두개 이상의 인자를 받는 경우의 예시는 다음과 같습니다.

```swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}

var matrix: Matrix = Matrix(rows: 2, columns: 2) // 초기화
matrix[0, 1] = 1.5 
matrix[1, 0] = 3.2
```  

내부적으로는 1차원 배열이지만, Subscript를 적절히 이용해 행렬처럼 사용할 수 있게 만들었습니다.

지금까지는 인스턴스에 대한 Subscript만 설명했지만, 타입 자체적으로 호출이 가능한 Subscript도 만들 수 있습니다. 타입 메소드나 프로퍼티를 만들 때 처럼 static만 붙여주면 됩니다. 클래스의 경우에는 class 키워드를 대신 붙일 수도 있으며, 이 경우 서브클래스에서 오버라이드가 가능해집니다.

다음은 타입 Subscript의 예시입니다.  

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
    static subscript(n: Int) -> Planet {
        return Planet(rawValue: n)!
    }
}
let mars = Planet[4]
print(mars)

```

---

지금까지 Subscript에 대해 알아보았습니다. 저는 주로 범위검사를 넣거나, 내부에 컨테이너를 가진 타입의 경우에 Subscript를 사용합니다.