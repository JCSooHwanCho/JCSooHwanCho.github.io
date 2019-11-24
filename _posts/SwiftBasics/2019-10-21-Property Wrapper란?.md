---
layout: post
title: Property Wrapper란?
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---  

Swift 5.1버젼부터 Property Wrapper라는 기능이 추가 되었습니다. Property Behaviors, Property Delegate라고도 하는 이 기능이 무엇이고 왜 추가 되었는지, 어떻게 사용하는 지를 알아보도록 하겠습니다.

> 이 포스트는 Swift 공식 refernce와 Swift-evolution의 proposal을 기준으로 작성되었습니다.  
> [proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)  
> [reference](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)  
 
* **Property Wrapper의 목적**  

    프로퍼티를 구현할 때 반복적으로 사용되는 패턴이 있습니다. lazy를 가장 대표적인 예시로 들 수 있습니다. Swift는 이를 언어 차원에서 제공하지만, 이를 언어 지원 없이 구현해야 한다면, 필요할 때마다 다음과 같은 코드를 작성해야 할 것입니다. (이를 Boilerplate Code라 합니다.)

    ```swift
    struct Lazy {
    // lazy var foo = 1738
    private var _foo: Int?
        var foo: Int {
            get {
            if let value = _foo { return value }
            let initialValue = 1738
            _foo = initialValue
            return initialValue
            }
            set {
            _foo = newValue
            }
        }
    }
    ```  

    지금은 언어 차원에서 이를 지원하지만, 이러면 컴파일러 구현과 언어가 복잡해지고 여러가지 구현을 유연하게 제공해주기 어렵다는 단점이 있습니다. 컴파일러에 하드코딩을 할 수도 있겠지만 하드 코딩이 좋은 선택이라고 보기는 어렵죠.

    따라서 이러한 코드를 라이브러리로 만들어 사용할 수 있게 함으로 컴파일러의 변경을 최소화하면서 더 많은 매커니즘을 재사용할 수 있도록 만들려는 시도의 결과가 Property Wrapper입니다.

* **Property Wrapper의 사용**  
    proposal에서 제시하는 Lazy의 Property Wrapper 버전 구현은 다음과 같습니다.

    ```swift
    @propertyWrapper
    enum Lazy<Value> {
    case uninitialized(() -> Value)
    case initialized(Value)

    init(wrappedValue: @autoclosure @escaping () -> Value) {
        self = .uninitialized(wrappedValue)
    }

    var wrappedValue: Value {
        mutating get {
            switch self {
            case .uninitialized(let initializer):
                let value = initializer()
                self = .initialized(value)
                return value
            case .initialized(let value):
                return value
                }
            }
        set {
            self = .initialized(newValue)
            }
        }
    }

    @Lazy var foo: Int = 1738 // (Property Wrapper) [var/let] (name) 
    ```  
    
    이렇게 하면 foo 프로퍼티는 Int 타입처럼 사용이 가능하면서, Lazy라는 Wrapper를 통해 값에 접근하게 됩니다. 즉, 다음과 같이 바뀝니다.  

    ```swift
    private var _foo: Lazy<Int> = Lazy<Int>(wrappedValue: 1738) // 프로퍼티명에 _(언더바)가 추가된 형태로 존재한다. 접근 제한자는 private다.
    var foo: Int {
        get { return _foo.wrappedValue }
        set { _foo.wrappedValue = newValue }
    }
    ```  

    이렇게 자동으로 getter와 setter가 제공되기 때문에 프로그래머가 get,set 블록을 제공해 줄 수는 없습니다. 다만, willSet, didSet 블록은 제공할 수 있습니다.

    Property Wrapper는 반드시 wrappedValue라는 인스턴스 프로퍼티를 제공해야 합니다. 이 프로퍼티는 계산 프로퍼티, 저장 프로퍼티 모두 가능합니다.

    또한 projectedValue라는 프로퍼티를 선택적으로 제공해 줄 수 있는데, Property Wrapper의 값을 다른 타입으로 바꿔서 반환하는 역할을 합니다. 이 projectedValue는 $접두사를 앞에 붙여서 사용할 수 있습니다.

    ```swift
    @propertyWrapper
    struct WrapperWithProjection {
        var wrappedValue: Int
        var projectedValue: SomeProjection { 
            return SomeProjection(wrapper: self)
        }
    }
    struct SomeProjection {
        var wrapper: WrapperWithProjection
    }

    struct SomeStruct {
        @WrapperWithProjection var x = 123
    }

    let s = SomeStruct()
    s.x           // Int value
    s.$x          // SomeProjection value
    s.$x.wrapper  // WrapperWithProjection value
    ```  

    또한 이름부터가 'Property' Wrapper이기 때문에, Property가 아닌 지역 변수나 전역 변수에는 사용할 수 없습니다.

* **Property Wrapper초기화**  
    Property Wrapper는 여러가지 초기화 방법을 제공합니다. 예제 코드로 한꺼번에 살펴보도록 하겠습니다.  

    ```swift
    @propertyWrapper
    struct SomeWrapper {
        var wrappedValue: Int
        var someValue: Double
        init() {
            self.wrappedValue = 100
            self.someValue = 12.3
        }
        init(wrappedValue: Int) {
            self.wrappedValue = wrappedValue
            self.someValue = 45.6
        }
        init(wrappedValue value: Int, custom: Double) {
            self.wrappedValue = value
            self.someValue = custom
        }
    }

    struct SomeStruct {
        // init() 호출
        @SomeWrapper var a: Int

        // init(wrappedValue:) 호출
        @SomeWrapper var b = 10

        // init(wrappedValue:custom:) 호출, 둘 다 유효한 호출
        @SomeWrapper(custom: 98.7) var c = 30 // 할당문은 wrapped value를 초기화 하도록 자동으로 사용된다.
        @SomeWrapper(wrappedValue: 30, custom: 98.7) var d
    }
    ```  

* **Property Wrapper합성**  
  Property Wrapper는 두개 이상 적용할 수 있습니다. 이 때 교환법칙은 성립하지 않습니다. 또한 합성시에는 각 Property Wrapper의 제한 조건을 만족하는 방향으로 설정이 되어야 합니다.

  ```swift
  // DelayedMutable<Copying<UIBezierPath>>
  @DelayedMutable @Copying var path1: UIBezierPath 

  // Copying<DelayedMutable<UIBezierPath>>
  @Copying @DelayedMutable var path2: UIBezierPath 
  ```

