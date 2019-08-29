---
layout: post
title: RxSwift연산자-Create
comments: true
tags: [iOS 프로그래밍,Swift,Apple, ReactiveX,RxSwift]
category: [RxSwift]
---  

이번 포스트부터 꾸준히 Rxswift의 여러 연산자들을 살펴보도록 하겠습니다. 원래는 카테고리 별로 묶어서 포스트를 작성하려고 했으나, 포스트의 호흡을 짧게 유지하고, 필요한 연산자를 빠르게 찾아볼 수 있도록 연산자 하나하나 포스트를 작성하도록 하겠습니다.

---  

이번에 살펴볼 연산자는 [Create](http://reactivex.io/documentation/operators/create.html)연산자입니다. 이 연산자는 직접 Observable 시퀀스를 구성할 때 사용합니다.



```swift
public static func create(_ subscribe: @escaping (AnyObserver<Element>) -> Disposable) -> Observable<Element>
```  

제공해줘야 하는 인자는 (AnyObserver<Element>) -> Disposable 타입의 클로저입니다. 

1. AnyObserver는 해당 Observable을 구독하게 될 Observer라고 생각하면 됩니다. Observer의 on 메소드를 통해 해당 Observer에 이벤트를 전달하게 됩니다.

2. Disposable은 해당 Observable과 연결될 Disposable입니다. 이 Observable은 명시적으로 Dispose 메시지를 받거나, DisposeBag에 담겨 있다 Bag이 해제될 때 메모리 해제가 일어나게 됩니다.

실 사용 예를 보겠습니다.

```swift
// 타입 추론이 불가능해서 명시적으로 타입을 Generic으로 써줘야 합니다.
Observable<Int>.create { observable in
    
    //실제 이벤트를 내보내는 부분
    observable.onNext(0)
    observable.onNext(1)
    observable.onNext(2)
    observable.onCompleted()
    
    //Disposable을 만들어 내보냅니다.
    return Disposables.create()
    
    }.subscribe { event in
        switch event {
        case let .next(value):
            print(value)
        default:
            print("finished")
        }
        
}.disposed(by: bag)

//0
//1
//2
//finished
```  

조금만 응용하면 다음과 같이, 비동기적인 요청이 필요할 때도 손쉽게 Observable을 적용할 수 있습니다.  

```swift
func loadImage() {
        let url = URL(string: LARGE_IMAGE_URL)! //이미지를 다운받을 URL
        
        Observable<UIImage?>.create { observable in
                DispatchQueue.global().async { // 비동기적으로 이미지 요청을 합니다.
                    do {
                        let data = try Data(contentsOf: url)
                        let image: UIImage? = UIImage(data: data)
                        
                        observable.onNext(image) // 다운받은 이미지를 내보냅니다.
                        observable.onCompleted()
                    } catch {
                        observable.onError(error)
                    }
                }

                return Disposables.create()
            }.subscribe {event in
                switch event {
                case let .next(image):
                    DispatchQueue.main.async {
                        self.imageView.image = image
                    }
                case let .error(e):
                    print(e.localizedDescription)
                    self.imageView.image = nil
                case .completed:
                    break
                }
            }.disposed(by: bag) // 외부에 선언된 DisposeBag 

    }
```  

이처럼 기존코드를 쉽게 Observable로 바꿀 수 있게 됩니다.