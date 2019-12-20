---
layout: post
title: Swift API Design Guidelines 톺아보기 - 관례, 특수 규정
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---

[지난번 포스트](https://jcsoohwancho.github.io/2019-07-19-Swift-API-Design-Guidelines-톺아보기-기초,-이름짓기/)에 이어서 계속해서 가이드라인에 대해서 보도록 하자.

---  

1. ***관례***  

    1. 일반적인 관례
        * 계산 프로퍼티(Computed Property)의 경우, O(1)이 아닌 경우에는 반드시 그 복잡도를 문서화하라 - 프로퍼티에 접근할때 사람들은 저장 프로퍼티(Stored Property)라고 생각하기 때문에 큰 연산이 필요하지 않을 것이라 가정한다. 이러한 가정에 위반되는 경우에는, 경고하는 것을 잊지 마라.  

        * 비멤버 함수(free function) 보다 멤버함수와 프로퍼티를 우선시하라 - 특수한 경우에만 사용하라  

            1. 명확한 self 대상이 없는 경우  
                
                ```swift
                min(x, y, z)
                ```  

            2. 제한이 없는 제네릭 함수인 경우  

                ```swift
                print(x)
                ```  

            3. 함수 문법이 이미 해당 영역에서 의미가 부여된 표현일 경우  

                ```swift
                sin(x)
                ```

        * 대소문자 규칙을 준수하라 - 타입명,프로토콜 이름은 UpperCamelCase, 나머지는 모두 lowerCamelCase로 지으라  

            * 미국식 영어에서 모든 글자가 대문자로 등장하는 두문자어(Acronym)의 경우 위의 규칙에 따라 모든 글자를 대문자로 하거나 소문자로 해야한다.  
            
                ```swift
                var utf8Bytes: [UTF8.CodeUnit]
                var isRepresentableAsASCII = true
                var userSMTPServer: SecureSMTPServer
                ```  

            * 그 외의 두문자어는 다른단어와 동일하게 취급한다.  

                ```swift
                var radarDetector: RadarScanner
                var enjoysScubaDiving = true
                ```  
        
        * 메소드들이 같은 의도를 가졌거나, 다른 영역에서 동작하는 경우에는 같은 이름을 공유할 수 있다.  

            * 나쁜 예시  

                ```swift
                //동일한 영역에서 한 단어를 두가지 이상의 의미로 쓰고 있다.
                extension Database {
                // 데이터베이스의 모든 인덱스를 재구축한다.
                func index() { ... }

                // n번째 행을 반환한다.
                func index(_ n: Int, inTable: TableID) -> TableRow { ... }
                }

                //반환 값만으로 오버로딩하면 타입 시스템에 혼란이 오게 된다.
                extension Box {.
                
                func value() -> Int? { ... }
                func value() -> String? { ... }
                }
                ```  

            * 좋은 예시  

                ```swift  
                // 이 함수들은 모두 본질적으로 하는 일이 같다.
                extension Shape {
                // other가 self안에 포함된 경우 true 반환
                func contains(_ other: Point) -> Bool { ... }

                // other가 self안에 완전히 포함된 경우 true 반환
                func contains(_ other: Shape) -> Bool { ... }

                // other가 self안에 포함된 경우 true 반환
                func contains(_ other: LineSegment) -> Bool { ... }

                // Shape와 Collection은 서로 영역이 분리되어 있으므로 같이 써도 무리가 없다.
                extension Collection where Element : Equatable {
                // Returns `true` iff `self` contains an element equal to
                // `sought`.
                func contains(_ sought: Element) -> Bool { ... }
                }
                ```  
    2. 매개변수  
        
        * 문서에 맞춰서 매개변수 명을 정하라. - 비록 사용자가 함수나 메소드를 사용할 때 드러나지는 않지만, 설명할때는 중요한 역할을 한다.  
            * 문서로 작성했을 때 읽기 쉽고 자연스럽게 정하라.  
            
                ```swift

                // 'predicate'라는 이름은 문서로 써도 문법이 맞다.
                // Return an `Array` containing the elements of `self`
                // that satisfy `predicate`.
                func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

                // Replace the given `subRange` of elements with `newElements`.
                mutating func replaceRange(_ subRange: Range, with newElements: [E])

                // 'includeInResult'라는 이름은 문서로 써놓으면 문법이 맞지 않아 어색해보인다.
                // Return an `Array` containing the elements of `self`
                // that satisfy `includedInResult`.
                func filter(_ includedInResult: (Element) -> Bool) -> [Generator.Element]

                // Replace the range of elements indicated by `r` with
                // the contents of `with`.
                mutating func replaceRange(_ r: Range, with: [E])
                ```  

        * 일반적인 사용을 쉽게 만드는 기본 매개변수(defaulted parameters)를 잘 활용하라 - 한가지 값이 일반적으로 사용된다면 기본값으로 고려할 만 하다.  

            * 기본 매개변수는 관련성이 적은 정보들을 감추고, 가독성을 높인다.  

                ```swift  
                //뒤에 붙은 option,range,locale등의 정보는 큰 필요 없는데도 생략되지 않았다.
                let order = lastName.compare(royalFamilyName, options: [], range: nil, locale: nil)
                //반드시 필요한 정보만 들어가 있다.
                let order = lastName.compare(royalFamilyName)
                ```  

            * 기본 매개변수는 메소드 집합(method families)보다 우선시된다. API를 이해하려는 사람의 인지적 부담을 덜어주기 때문이다.  

                * 나쁜 예  

                    ```swift
                    extension String {
                    /// ...description 1...
                    public func compare(_ other: String) -> Ordering
                    /// ...description 2...
                    public func compare(_ other: String, options: CompareOptions) -> Ordering
                    /// ...description 3...
                    public func compare(
                        _ other: String, options: CompareOptions, range: Range) -> Ordering
                    /// ...description 4...
                    public func compare(
                        _ other: String, options: StringCompareOptions,
                        range: Range, locale: Locale) -> Ordering
                    }
                    ```  

                * 좋은 예  

                    ```swift  
                    extension String {
                    /// ...description...
                    public func compare(
                        _ other: String, options: CompareOptions = [],
                        range: Range? = nil, locale: Locale? = nil
                    ) -> Ordering
                    }
                    ```  

                * 이러한 메소드 집합의 모든 함수들은 별도로 문서화 되어야 하고, 유저들도 이를 별도로 알고 있어야 한다. 어떤 것을 쓸지 판단하려면, 모든 메소드를 알아야 함은 물론이고 그 메소드들 간의 미묘한 차이를 알기 위해 비슷비슷하게 생긴 문서들을 뚫어지게 쳐다봐야 한다. 기본 매개변수가 제공되는 단일 메소드는 프로그래머의 경험의 질을 높인다.  

        * 기본 값이 있는 매개변수는 맨 끝에다 놓아라 - 기본값이 없는 매개변수는 메소드의 구조에서 훨씬 더 중요한 의미를 가진 경우가 많고, 처음 쓰기에도 좋은 패턴을 제공해준다.  

    3. 매개변수 레이블  
        
        * 매개변수가 유의미하게 구분이 되지 않는경우에는 모든 레이블을 생략하라.  

            ```swift
            min(number1, number2), zip(sequence1, sequence2)//순서가 바뀌어도 상관 없다.
            ```  
        
        * 값을 보존하는 타입 변환을 수행하는 초기화 메소드는 첫 레이블을 생략하라.  

            * 첫 매개변수는 항상 변환 대상이 되어야 한다.  

                ```swift  
                extension String {
                // Convert `x` into its textual representation in the given radix
                init(_ x: BigInt, radix: Int = 10) //Note the initial underscore
                }

                text = "The value is: "
                text += String(veryLargeNumber)
                text += " and in hexadecimal, it's"
                text += String(veryLargeNumber, radix: 16)
                ```  

        * 값이 잘리는(narrowing) 변환이라면, 값이 잘리는 것을 알려주는 레이블을 권장한다.  

            ```swift  
            extension UInt32 {
            /// Creates an instance having the specified `value`.
            init(_ value: Int16)            ← Widening, so no label
            /// Creates an instance having the lowest 32 bits of `source`.
            init(truncating source: UInt64)
            /// Creates an instance having the nearest representable
            /// approximation of `valueToApproximate`.
            init(saturating valueToApproximate: UInt64)
            }
            ```  

        * 첫번째 매개변수가 전치사구(prepositional phrase)의 일부라면, 레이블을 주어라  
            * 만약 첫 두 매개변수가 하나의 추상화를 이루는 경우는 예외가 발생한다.  

                ```swift
                a.move(toX: b, y: c)
                a.fade(fromRed: b, green: c, blue: d)
                ```  

                이러한 경우는 레이블은 전치사 뒤에서 부터 시작하게 해서 추상화를 깔끔하게 하라.  

                ```swift  
                a.moveTo(x: b, y: c)
                a.fadeFrom(red: b, green: c, blue: d)
                ```  
            * 그 외에도, 첫 매개변수가 문법적인 구의 일부를 구성한다면, 해당 레이블을 생략하고 앞 단어를 메소드명에 붙이라  
                
                * 그 말은 곧 첫 매개변수가 문법적인 구를 구성하지 않는다면 레이블을 가져야 한다는 것이다.
                    ```swift  
                    view.dismiss(animated: false)
                    let text = words.split(maxSplits: 12)
                    let studentsByName = students.sorted(isOrderedBefore: Student.namePrecedes)
                    ```  

                * 그와 동시에 이러한 구가 정확한 의미를 전달하는 것도 중요하다.  

                    ```swift
                    view.dismiss(false)   //dismiss하지 말란 것인가? bool을 dismiss하라는 건가?  
                    words.split(12)      // 12를 쪼개란 것인가?
                    ```  
            * 또한 모든 기본 값을 가지는 매개변수는 생략이 가능하다는 점도 주의하라. 이런 경우는 해당 매개변수가 문법적 구를 구성하지 않기 때문에, 반드시 레이블을 가져야 한다.  

        * 이외의 경우는 모두 레이블을 가져야 한다.  

2. ***특수 규정***  

    * 튜플의 구성요소와 클로져 매개변수에도 레이블을 붙이라 - 이러한 이름들은 설명할 때 유용하여, 문서화 주석을 달때도 참조할 수 있고, 튜플 멤버에 접근하는 것도 쉽게 설명될 수 있게 한다.  

    * 클로저 매개변수에 대해서는 매개변수 이름 짓는 규칙이 동일하게 적용된다.  

    * 제한없는 다형성을 다룰 때는 오버로딩된 함수들의 모호성에 특별히 주의해야 한다.  
    
        ```swift
        struct Array {
        // array의 마지막에 newElement를 삽입힌다.
        public mutating func append(_ newElement: Element)

        // array의 마지막에 newElements의 원소들을 순서대로 삽입힌다.  
        public mutating func append(_ newElements: S)
            where S.Generator.Element == Element
        }

        var values: [Any] = [1, "a"]
        values.append([2, 3, 4]) // [1, "a", [2, 3, 4]] or [1, "a", 2, 3, 4]? 타입으로 구분이 안되기 때문에 어떤 함수를 호출할 지 모호해진다.
        ```  

        따라서 이러한 모호함을 해결하기 위해서 오버로딩되는 함수를 더욱 명시적으로 만들 필요가 있다.  

        ```swift  
        struct Array {
        public mutating func append(_ newElement: Element)

        public mutating func append(contentsOf newElements: S)
            where S.Generator.Element == Element
        }  
        ```  
        
---  

실제로 코드를 작성하고 리뷰를 받아보니 가장 놓치기 쉽고 계속해서 지적하기도 미묘한 부분이 코딩 규칙이다. 시작부터 제대로 습관을 잡아놓지 않으면 스스로의 발전을 저해하는 효과를 가져올 수 있는 만큼 계속 읽고 확실하게 숙지해야겠다.
