---
layout: post
title: Swift API Design Guidelines 톺아보기 - 기초,이름 짓기
comments: true
tags: [Swift,Apple,Guideline]
category: [SwiftBasics]
---
블로그를 만들어야지 생각만 하다가 이번 [부스트코스 에이스](https://apply.connect.or.kr/connect/applyDetail?annoId=20003033&page=1)에 선정이 된 기념으로 진짜 마음먹고 블로그를 시작해보기로 했다.
시작은 복잡한 것을 하기 보담도 Apple의 문서를 읽은 내용을 정리하거나, 프로젝트를 수행하며 해결했던 여러 문제들에 대한 글을 간결하게 써보고자 한다.

코딩을 하다보면 과거 자신의 코드나 다른 사람의 코드를 읽을 일이 많다. 잘 써진 코드는 배경지식이 있다면 비교적 편안하게 읽을 수 있겠지만, 잘못 써진 코드는 읽기도 힘들고 유지보수는 더더욱 힘들다. 가장 큰 문제는 미래의 나 자신조차 읽기가 힘들면 결국 다시 만들거나 유지보수하지 않은 채 잘 되기만을 기도할 수 밖에 없어진다.

| ![기도메타]({{"/img/prayMeta.jpeg"}}) |
| :--: |
|코드가 잘 돌아가길 기도하는 모습이다.[출처](http://m.inven.co.kr/board/powerbbs.php?come_idx=2097&l=1012127)|

그래서 코딩 컨벤션을 정해놓고 잘 지키는 게 중요하다. 혼자 읽을 코드라면야 혼자서 규칙 정해놓고 잘 하면 되겠지만, 여러 사람이 읽을 코드라면 규칙을 합의해놓고 쓰는 게 중요하다. 이 규칙은 여러가지가 있을 수 있지만 필자는 swift 코딩을하기 때문에, 그 중 가장 기반이 될 만한 문서인 [swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)를 읽어보고자 한다. API 디자인에 관한 문서이지만 API라는 것도 "반복적으로 사용되는 코드의 집합"이라고 생각하면 평소에 코드를 짤 때에도 이 규칙들은 유효하다.

 **이 글은 본문의 내용을 최대한 그대로 옮기는 것을 중점으로 삼으며, 부가적인 설명이나 필자의 의견이 들어가는 부분은 따로 표시하여 서술합니다. 모든 예시는 API 가이드의 예시를 가져왔습니다**

---  

1. ***기초***  

	* 사용할 때의 명확성(Clearity at the point of use) - 가장 중요한 가치이다.  API의 요소들(메소드,프로퍼티 등)은 한번만 선언되지만 반복적으로 사용되기 때문에 명확하고 간결하게 (clear and concise) 디자인되어야 한다. API 디자인을 평가할 때에도 선언만 보는 건 충분치 않고 실제 사용 예제를 가지고 문맥상에서 평가해야 한다.

	* 명확성은 간결성보다 중요하다. - swift가 아무리 압축이 잘 되도, 최소한의 글자로 가장 작은 코드를 만드는 게 목표는 아니다. swift의 간결성은  강력한 타입 시스템과 보일러플레이트 코드를 자연스럽게 줄이는 여러 특징들에서 나오는 것이다.

		* (추가설명)보일러플레이트(boilerplate) : '표준 문안'이라는 뜻인데, 프로그래밍에서는 '수정이 없거나 거의 하지 않은채로 여러곳에서 필수적으로 사용되는 코드'라는 뜻으로 쓰인다. [출저](https://ko.wikipedia.org/wiki/상용구_코드)이러한 코드는 똑같은 코드를 반복적으로 써야하므로 코드의 양을 늘리고 반복적인 노동을 강요한다는 점에서 주로 부정적으로 쓰인다.

	* 모든 선언에 대해서 문서화된 주석을 남기라. - 문서를 작성하면 서 얻는 통찰은 API에 있어서 깊은 영향력을 가진다. 그러니까 미루지말라.(만약 API의 기능에 대해서 쉽게 설명하지 못한다면, **API디자인이 잘못됐을 가능성이 있다.**
		* Swift의 마크다운 방언을 사용하면 Xcode에서 이를 해석해 API 사용시 자동으로 설명과 용법을 띄워준다.[Markup Formatting Reference](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/)
		*  선언된 요소에 대한 요약(summary)으로 시작하라. API는 종종 선언과 요약만으로 완전히 이해되는 경우가 많다.
		* 추가적인 옵션이 필요하다면 한개 이상의 문단으로 서술해라. 문단은 빈 줄로 구분되며 완전한 문장들을 사용한다.  
		* (Xcode가)인식할 수 있는 심볼들을 때에 맞춰 적절히 사용하라. (자세한 것은 [여기](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/MarkupFunctionality.html#//apple_ref/doc/uid/TP40016497-CH54-SW1)를 참조하자)  

	
2. ***이름짓기***  

	1. *명확하게 사용하도록 유도하라*
		* 필요한 모든 단어를 포함하여 그 이름이 사용된 부분에서의 모호함을 피하라
			* (추가설명) swift는 매개변수에 레이블을 지정할 수 있고, 이 역시 함수 시그니처의 일부이다.  
			* 잘못된 예시  

				```swift  
				employees.remove(x) // 불명확하다. x가 값인가 인덱스인가?
				```  

		    * 옳은 예시  

				```swift  
				extension List {
				public mutating func remove(at position: Index) -> Element
				}
				employees.remove(at: x)//x가 인덱스를 지정하는 것이 명확하다.
				```  
		* 필요없는 단어는 빼라 - 이름에 들어가는 모든 단어는 하나하나가 핵심적인 의미를 전달해야 한다. 의도나 의미를 명확히 하기 위해 더 많은 단어를 사용할 수도 있지만, 사용자가 이미 가지고 있는 정보는 빼야한다. 보통 여기서 빠져야하는 정보는 타입에 대한 정보이다.  

		* 변수,매개변수,연관 타입의 이름은 타입 제약이 아니라 그들의 역할에 맞게 지어줘라.  
			* 잘못된 예시  

				```swift  
				//변수명이 곧 타입명이므로, 타입명은 안 헷갈려도 코드의 의도는 명확치 않다.
				var string = "Hello" 
				protocol ViewController {
				associatedtype ViewType : View
				}
				class ProductionLine {
				func restock(from widgetFactory: WidgetFactory)
				}
				```  
  		    * 옳은 예시  

				```swift
				//변수명이 코드의 의도를 드러내어준다.  
				var greeting = "Hello"
				protocol ViewController {
				associatedtype ContentView : View
				}
				class ProductionLine {
				func restock(from supplier: WidgetFactory)
				}
				```  

			* 만약 연관 타입이 프로토콜과 강하게 엮여 있어 프로토콜 이름이 곧 역할이라면,프로토콜 이름에 Protocol이라는 이
			름을 붙여줘서 충돌을 피해주자  

				```swift
				protocol Sequence {
				associatedtype Iterator : IteratorProtocol
				}
				protocol IteratorProtocol { ... }
				```  

		* 약타입의 경우에는 매개변수의 역할을 확실히 해주기 위해 추가적인 정보를 제공하자 - 매개변수가 NSObject,Any,AnyObject등이거나(필자 주 - 거의 모든 타입의 기본이 되는 녀석들로 지나치게 포괄적이다) Int, String등의 기본타입이라면, 타입 정보와 사용 문맥 만으로는 의도가 온전히 전달되지가 않는다. 명확성을 위해 약타입 매개변수의 역할을 설명하는 명사를 앞에다 붙여라  

			* 나쁜 예  

				```swift
				func add(_ observer: NSObject, for keyPath: String)
				grid.add(self, for: graphics) // 함수 사용시의 의미가 명확하지 않다.
				```
			* 좋은 예  

				```swift
				func addObserver(_ observer: NSObject, forKeyPath path: String)
				grid.addObserver(self, forKeyPath: graphics) // 옵저버를 추가한다는 의도가 명확히 드러난다.  
				```  
				<br>

	2. *유창하게 쓰려고 노력해라*  
		* 사용자가 문법에 맞는 영문구처럼 쓸 수 있는 메소드나 함수 이름을 택하라
			* 나쁜 예  

				```swift
				//의미는 전달되지만 문법상으로는 자연스럽지 않다.
				x.insert(y, position: z)
				x.subViews(color: y)
				x.nounCapitalize()
				```  

			* 좋은 예  

				```swift
				x.insert(y, at: z)// y를 x의 z 위치에 넣는다.  
				x.subViews(havingColor: y)  //x의 subViews중 color가 y인 뷰
				x.capitalizingNouns()       //x안의 명사들의 앞글자를 대문자로 만든다.
				```  

			* 첫번째나 두번째 인수 이후에는, 해당 인수가 의미상 중심적이지 않다면 유창성이 좀 떨어져도 괜찮다.  

		* [팩토리 메소드](https://en.wikipedia.org/wiki/Factory_method_pattern) 의 앞에는 'make'를 붙여라  

		* 초기화 메소드와 팩토리 메소드의 첫 인수는 함수의 이름과 구를 맞춰서는 안된다. (필자 주 - 해당 메소드들은 설정 값을 넣어서 무언가를 만들어낸다는 느낌이 강해서 이러한 규칙을 정해놓지 않았나 싶다. 자주 호출되기 때문에 간결할 필요도 있었을 것이고.)  
			* 나쁜 예  

				```swift
				let foreground = Color(havingRGBValuesRed: 32, green: 64, andBlue: 128)
				let newPart = factory.makeWidget(havingGearCount: 42, andSpindleCount: 14)
				let ref = Link(to: destination)
				```  

			
			* 좋은 예  

				```swift
				let foreground = Color(red: 32, green: 64, blue: 128)
				let newPart = factory.makeWidget(gears: 42, spindles: 14)
				let ref = Link(target: destination)	  
				```  

		* 사이드 이펙트를 고려해서 이름을 지어라   

			* 사이드 이펙트가 없으면 명사구로  

				```swift
				x.distance(to: y)
				i.successor()
				```  
			
			* 사이드 이펙트가 있으면 명령조의 동사구로  

				```swift
				print(x)
				x.sort()
				x.append(y)
				```  

			* Mutating/Nonmutating 메소드 쌍에 대해서  
				* Mutating 메소드가 동사로 표현이 되면 ed 나 ing를 붙여서  Nonmutating 함수 이름을 지어라  

				* Mutating 함수가 명사로 표현되면, 앞에 form을 붙여서 Nonmutating 함수를 만들라.  


		* Boolean형 함수는 Nonmutating일 경우, 결과를 받는 개체(receiver)에 대한 단정문(assertion)으로 읽히도록 하라  

		* 무엇인가를(what something is) 설명하는 프로토콜은 명사로 읽혀야 한다.  

		* 가능성(capability)을 나타내는 프로토콜은 able,ible,ing등의 접미사를 붙여 이름을 붙여야 한다.  

		* 나머지 타입,프로퍼티, 변수, 상수 등은 명사로 읽혀야 한다.  

	3. *전문용어를 잘 사용하라*  

		* 더 일반적인 용어가 의미를 그럭저럭 잘 전달한다면, 잘 안알려진 단어는 피하라. 전문 용어는 다른 단어로는 설명하기 어려운 의미를 담는데에만 사용되어야 한다.  

		* 이미 정립된 의미를 고수하라 - 전문용어를 쓰는 것은 오직 다른 단어로 표현하면 모호하거나 명확하지 않은 것을 정확히 설명하기 위해서이다.  

			* 전문가를 당황케 하지말라 - 이미 해당 용어에 익숙한 사람은 우리가 새로운 의미를 발명해낸다면 당황스럽고 화가 날 것이다.
			* 초심자를 혼란스럽게 하지말라 - 해당 용어를 처음 배우려는 사람은 인터넷에서 원래의 뜻을 찾아볼 것이다.  

		* 줄임말을 자제하라 - 줄임말은 줄이기 전의 형태를 올바로 알아야만 이해가 된다는 점에서 사실상 전문용어다.  

			* 줄임말의 원래 의미는 인터넷에서 쉽게 찾을 수 있는 것이여야만 한다.  

		* 선례를 받아들이라 - 기존 문화와의 부합을 포기하면서까지 완전 초보자에게 맞출 필요는 없다.  

	나머지 Conventions, Special Instructions는 다음번 포스트에서 이어서 하도록 하자.
