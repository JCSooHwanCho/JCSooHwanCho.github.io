---
layout: post
title: NSAttributedString 분석
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

텍스트 처리는 개발의 모든 영역에서 중요하게 다뤄지는 부분입니다. 그 중에서도 iOS는 문자열을 직접 사용자에게 보여줘야하기 때문에 문자열을 다양한 형태로 보여주기 위한 기능을 제공합니다. 이번 포스트에서는 이러한 기능을 제공해주는 핵심 클래스인 NSAttributedString에 대해서 알아보겠습니다.

* **NSAtrributedString의 구조**  
  [NSAttributedString](https://developer.apple.com/documentation/foundation/nsattributedstring)은 문자열 데이터인 String(NSString)과 이 String에 대한 Attribute들에 대한 정보를 가지고 있는 객체입니다. Foundation에 정의되어 있으며, 내부적으로 CoreFoundation과 긴밀하게 연결되어 있습니다. 깊게 들어가면 어려우므로 간단하게 알아보도록 하겠습니다. 다음은 Foundation 오픈소스에서 발췌한 NSAttributedString의 저장 프로퍼티들입니다. (실제 개발 환경과 다소의 차이가 있을 수 있습니다.)

  ```swift
  open class NSAttributedString: NSObject, NSCopying, NSMutableCopying, NSSecureCoding {
    
    private let _cfinfo = _CFInfo(typeID: CFAttributedStringGetTypeID())
    fileprivate var _string: NSString
    fileprivate var _attributeArray: CFRunArrayRef

    // ... 이하는 모두 메소드들 ...
  }
  ```  

  보다시피 NSString을 제외하고는 모두 CoreFoundation의 객체들을 그대로 사용하고 있습니다. 우선 _CFInfo는 CoreFoundation객체의 [isa](https://developer.apple.com/documentation/objectivec/id/1418809-isa?language=objc)(deprecatede된 것은 애플에서 이 값을 직접 참조하지 못하게 막은 것 뿐입니다.), 레퍼런스 카운트 등의 중요한 정보를 가지고 있는 객체입니다. 이는 곧 이 NSAttributedString이 사실상 CoreFoundation 객체임을 의미합니다. 실제로 NSAttributedString은 CFAttributedString으로 무비용 연결([Toll-free-bridge](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Toll-FreeBridgin/Toll-FreeBridgin.html))이 가능한 객체입니다.  
  string은 보다시피 문자열 데이터이고, 중요한 것은  _attributeArray의 정체입니다. 이 프로퍼티의 타입은 'CFRunArrayRef'라는 굉장히 낮선 타입입니다. 이 타입에 대해서 더 자세히 알아보겠습니다. CFRunArrayRef는 CFRunArray의 참조를 의미하고, CFRunArray 타입은 CoreFoundation에서 __CFRunArray 타입에 매칭되게 됩니다.

  ```c
    struct __CFRunArray {
        CFRuntimeBase base; /* swift 코드에서의 _CFInfo에 해당하는 값. */
        CFRunArrayGuts *guts;
    };

    typedef struct _CFRunArrayGuts {/* 가변 크기 객체 */
        CFIndex numRefs; /* CoW(Copy-on-Write)를 위한 값 */
        CFIndex length; /* list에 담긴 값들의 갯수, 각 CFRunArrayItem의 length의 합이다. */
        CFIndex numBlocks, maxBlocks; /* list에 담긴 CFRunArrayItem의 수 */
        CFIndex cachedBlock, cachedLocation; /* 마지막으로 배열에서 찾았던 값의 캐시값 */
        CFRunArrayItem list[0]; /* 가변 크기 객체를 만들기 위한 테크닉으로, C99 표준에 정의되어 있다. */
    } CFRunArrayGuts;

    typedef struct {
        CFIndex length; /* 해당 아이템이 차지하는 길이*/
        CFTypeRef obj; /* 해당 아이템이 의미하는 실제 객체의 참조 */
    } CFRunArrayItem; /* CFRunArray이 실제로 담고 있는 아이템 */
  ```  

  즉, 각 원소가 차지하는 길이(비중)이 다른 배열입니다. 특정 객체는 1의 길이 만큼을 가질수도 있고, 2 혹은 그 이상의 길이를 가질 수 있습니다. 이는 곧 문자열의 맨 앞에서부터 시작해서, 어느정도의 길이만큼 해당 Attribute를 적용할지를 의미합니다. 이때 한 글자에 여러개의 Attributes가 적용될 수 있기 때문에, CFRunArrayItem이 가지는 객체의 타입은 [NSAttributedString.Key: Any] 타입의 Dictionary(정확히는 CFDictionay)입니다. Any 타입은 Attribute로 사용할 수 있는 객체의 타입이 정해져 있지 않다는 것을 의미합니다. 따라서 어떠한 객체라도 들어갈 수 있습니다. 다만, 표준에서 제공하는 Key들은 특정한 타입의 객체(정확히는 특정 메시지를 이해할 수 있는 객체)를 요구하고, 이를 지키지 않으면 크래시가 나기 때문에 이를 반드시 문서와 검색을 통해 확인해야 합니다. ([참조1](https://developer.apple.com/documentation/foundation/nsattributedstring/key), [참조2](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/AttributedStrings/Articles/standardAttributes.html#//apple_ref/doc/uid/TP40004903-SW2))
  
  지금까지 설명한 구조를 도식화하면 다음과 같습니다.

    ![NSAttributedString]({{"/img/NSAttributedStringStructure.png"}}){: .center-block :} 

  여기서 Range를 표현하는 방식은 NSRange입니다.(익숙하지 않으시다면 [NSRange와 Range](../2019-11-17-NSRange와-Range/) 포스트를 참조해주세요) 같은 Attribute를 가질 경우에는 최대한 한 덩어리로 취급하며, Attribute 구성이 달라지는 경우에 분리가 됩니다. Attribute가 없는 부분은 빈 Dictionary로 나타내어, Attributes 배열이 문자열 전체를 빠짐없이 커버합니다. 이때 Attributes가 동일하다고 판단되기 위해서는 NSObject의 isEqual 메소드로 비교했을 때 true여야 하며, 이때는 동일한 Attribute가 부분부분 적용 되어 있어도 동일한 객체를 공유하여 같은 값을 가진 객체가 중복하여 생기지 않도록 메모리를 최적화합니다. 때문에 커스텀 타입을 attribute로 사용하려면 NSObject를 상속받고 isEqual 메소드를 오버라이드 해야 동일한 값임을 프레임워크가 판단하고 최적화할 수 있습니다. 또한 swift의 값타입을 Attribute로 사용하려고 한다면 반드시 Hashable을 채택해서 불필요한 객체가 여러개 생기지 않도록 해줘야 합니다(그 이유는 [Foundation의 Swift 타입 브릿징](../2020-03-01-Foundation에서의-Swift-타입-브릿징)을 참조해주세요)

* **NSAttributedString의 사용법**  
   NSAttributedString은 String과 Attribute가 담긴 Dictionary를 통해 초기화 하는 방법이 일반적입니다. 다만 이 방법으로는 복잡한 AttritbutedString은 초기화가 안됩니다.

   ```swift
   init(string str: String, attributes attrs: [NSAttributedString.Key : Any]? = nil)
   ```  

   혹은 RTF(Rich Text Format)혹은 RTFA(RTF with Attachment, RTFD) 데이터를 통해서도 초기화가 가능하고, 혹은 HTML데이터를 URL을 통해 다이렉트로 로드할 수도 있습니다. 다만 이 과정에서 WebKit을 사용하기 때문에 반드시 메인 스레드에서만 사용해야 합니다.([참조](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/AttributedStrings/Tasks/CreatingAttributedStrings.html#//apple_ref/doc/uid/20000714-BBCCGGCC))  

   만약 이러한 데이터들 없이 Attribute가 복잡하게 적용된 AttributedString을 초기화 하기 위해서는 NSMutableAttributedString을 만든뒤, AttributedString을 이어 붙이거나 Attribute를 수동으로 적용하는 방법밖에는 없습니다.  

   특정 글자에 적용된 Attribute를 확인하기 위해서는 다음 메소드를 사용합니다.  

   ```swift
   func attributes(at: Int, effectiveRange: NSRangePointer?) -> [NSAttributedString.Key : Any] // 특정 위치에 적용되어있는 Attribute의 Dictionay를 반환합니다.
   func attributes(at: Int, longestEffectiveRange: NSRangePointer?, in: NSRange) -> [NSAttributedString.Key : Any]
   func attribute(NSAttributedString.Key, at: Int, effectiveRange: NSRangePointer?) -> Any? // 특정 위치에 원하는 Attribute가 적용되어 있다면 그 객체를 반환합니다. 없을 경우 nil을 반환합니다.
   func attribute(NSAttributedString.Key, at: Int, longestEffectiveRange: NSRangePointer?, in: NSRange) -> Any?
   ```  

   effectiveRange와 longestEffectiveRange 인자는 NSRangePointer 타입인데, 이는 **inout NSRange**와 동일합니다. 즉, 해당 Attribute가 적용되어 있는 범위를 의미합니다. 다만 effectiveRange는 해당 Attribute가 적용된 범위를 나타내는 것은 맞지만, 최대 범위를 의미하지는 않습니다. 만약 최대 범위를 찾고 싶다면 반드시 longestEffectiveRange 레이블을 가진 메소드를 사용해야 합니다. 
   
   만약 attribute 전체 혹은 특정 attribute를 훑어야 한다면 enumerateAttribute(s)계열의 메소드를 사용할 수 있습니다.  

   ```swift
   func enumerateAttribute(_ attrName: NSAttributedString.Key, 
                     in enumerationRange: NSRange, 
                options opts: NSAttributedString.EnumerationOptions = [], 
                  using block: (Any?, NSRange, UnsafeMutablePointer<ObjCBool>) -> Void)

    func enumerateAttributes(in: NSRange,
                        options: NSAttributedString.EnumerationOptions
                          using: ([NSAttributedString.Key : Any], NSRange, UnsafeMutablePointer<ObjCBool>) -> Void)
   ```  

   using은 attribute를 순회하면서 사용할 클로저인데, 인자로 Attribute Dictionary 혹은 Attribute 객체, 해당 attribute가 적용되는 범위, 순회를 계속할지를 결정하기 위한 inout Bool이 주어집니다. 특정 Attribute를 지정하여 순회할 경우, 파편화 되어 있는 객체들을 비교해서 같다면 하나로 합쳐서 범위를 반환해줍니다.

   NSAttributedString의 Mutable 버전은 여기에 더해서 글자를 바꾸거나([replaceCharacters](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1418451-replacecharacters)), 지우고([deleteCharacters](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1410610-deletecharacters)), Attribute를 덮어씌우거나([setAttributes](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1412179-setattributes)) 추가([addAttributee](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1414304-addattributes)), 삭제([removeAttributes](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1409691-removeattribute)) 등을 할 수 있는 기능을 제공해줍니다. 다만 이러한 작업은 구조를 생각해보면 Dictionary를 자르고 합치는 연산이 많이 필요한 굉장히 번거로고 연산량이 많은 작업입니다. 그렇기 때문에 이러한 변경들을 모아서 한꺼번에 처리할 수 있도록 그룹화 해주는 [beginEditing](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1411853-beginediting), [endEditing](https://developer.apple.com/documentation/foundation/nsmutableattributedstring/1411853-beginediting) 쌍을 제공해줍니다.

* **정리**  
  NSAttributedString은 Apple이 제공하는 Text 처리 프레임워크들에서 핵심적인 위치를 차지하고 있습니다. 이번 포스트에서 소개한 것 이외에도 굉장히 방대한 기능들을 제공해주고 있기 때문에, 이러한 기능들은 차후 기회가 되는대로 살펴보도록 하겠습니다.

    ---  

    > 참조 문서  
    > [Attributed String Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/AttributedStrings/AttributedStrings.html#//apple_ref/doc/uid/10000036-BBCCGDBG)  
    > [NSAttributedString](https://developer.apple.com/documentation/foundation/nsattributedstring)  
    > [NSMutableAttributedString](https://developer.apple.com/documentation/foundation/nsmutableattributedstring)

