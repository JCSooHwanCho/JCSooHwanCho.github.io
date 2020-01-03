---
layout: post
title: Core Data 시작하기(2) - Data Model 만들기(1) - entity만들기
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

이번 포스트에서는 Core Data의 핵심 요소인 Data Model을 만들고 구성하는 방법에 대해서 알아보도록 하겠습니다.  

* Data Model 만들기
  Data Model을 만드는 방법은 2가지가 있습니다.  

  1. 프로젝트를 처음 만들 때 만들어주기
    프로젝트를 처음 만들 때, Core Data 사용 여부를 체크할 수 있습니다.  

    ![Use Core Data]({{"/img/UseCoreData.png"}}){: .center-block :}  

    이렇게 되면 프로젝트 명과 동일한 이름의 Data Model 파일을 자동으로 만들어줍니다. 또한, AppDelegate의 프로퍼티로 Container 초기 설정 코드와 저장용 메소드를 자동으로 만들어 줍니다. 

    ![Initial Core Data Code]({{"/img/InitialCoreDataCode.png"}}){: .center-block :}  
  
  2. 기존 프로젝트에 Data Model 추가하기
    프로젝트 처음 생성할 때 Core Data 사용에 체크하지 않았거나, Core Data를 사용하지 않던 기존 프로젝트에서 Core Data를 사용하고 싶을 경우엔 Data Model을 만들고 Container를 만들고 저장하는 코드를 수동으로 만들어주면 됩니다.  

    ![Create Data Model]({{"/img/CreateDataModel.png"}}){: .center-block :}  

    이렇게 한 뒤, 해당 Data Model을 사용하는 Container를 만들어서 사용하고 저장하면 됩니다.

* Data Model 구성하기 
  이렇게 Data Model 파일이 만들어 졌다면, 이제 Data Model을 직접 구성해야 합니다. 만들어진 DataModel 파일(.xcdatamodel 파일)을 클릭하면, DataModel 편집기가 나타나게 됩니다.  

  ![Data Model Editor]({{"/img/DataModelEditor.png"}}){: .center-block :}  

  1. Entity 설정  
    가장 먼저 해야할 일은 Entity를 만드는 일입니다. Data Model에서 Entity는 Core Data에 의해 관리되는 객체의 단위입니다. 즉, 클래스에 해당합니다. 이는  데이터베이스의 Schema와 유사하며, 객체가 가지는 attribute들과, 다른 Entity와의 관계(Relationship)들로 이루어 집니다. 
    
    Entity를 만들고 이 Entity를 클릭하면, inspector에서 Entity에 대한 설정을 할 수 있습니다. 
    
    ![Data Model Example]({{"/img/DataModelExample.png"}}){: .center-block :}  
    
    주요 항목들의 의미를 살펴보면 다음과 같습니다.  

    ![EntityInspector]({{"/img/EntityInspector.png"}}){: .center-block :}

    1. Entity Name: Entity의 이름입니다. DataModel 편집기에 있는 Entity list에 보이는 Entity이름과 동일한 것으로, 한쪽에서 바꿀 때 마다 다른 쪽에 반영됩니다.
  
    2. Abstract Entity: 해당 entity로 instance를 만들 수 있는 지 여부를 나타냅니다. 이 항목에 체크가 되어 있으면, 해당 entity는 추상 entity 로 사용되어 해당 타입의 인스턴스를 만들 수 없게 됩니다. 만약 Abstract 설정이 되어 있는 entity를 DataModel을 통해 인스턴스화 시키려 한다면 예외가 발생하게 됩니다.
  
    3. Parent Entity: 부모 Entity를 설정합니다. 이를 설정하면 부모 entity의 모든 attribute를 그대로 가지게 됩니다. 즉, 상속의 효과를 가질 수 있습니다.
  
    4. Class Name: 해당 entity를 클래스 선언으로 바꿀 때 사용하는 클래스 이름을 나타냅니다. 기본적으로는 Entity Name을 그대로 따라가지만, 원하면 수정할 수 있습니다. 대신, Class Name을 바꿔도 Entity Name에는 영향을 주지 않습니다.
  
    5. Module: 해당 entity의 클래스 선언이 속한 모듈을 정해줍니다. 기본적으로는 글로벌 이름공간에 속합니다.
  
    6. Codegen: 해당 entity에 대한 클래스 선언을 자동으로 만들어 주는 옵션을 설정합니다. 
     
        * None/Manual: 관련 파일을 자동으로 만들어주지 않습니다. 개발자는 DataModel을 선택한 상태에서 Editor-Create NSManagedObject Subclass 항목을 클릭하여 클래스 선언 파일과 프로퍼티 extension 파일을 빌드시마다 추가시켜 주고, 이를 수동으로 관리해야 합니다.
         
        * Class Definition: 클래스 선언 파일과 프로퍼티 관련 extension 파일을 빌드시마다 자동으로 추가시켜줍니다. 따라서 관련된 파일을 전혀 추가시켜줄 필요가 없습니다.(그래서도 안됩니다. 만약 수동으로 추가시켜준 상태에서 빌드를 시도하면 컴파일 에러가 발생합니다.)
         
        * Category/Extension: 프로퍼티 관련 extension파일만 자동으로 추가시켜 줍니다. 즉, 클래스 선언에는 사용자가 원하는 로직을 자유롭게 추가할 수 있습니다.

    7. Constraint: attribute를 콤마로 구분해서 입력해서 해당 attribute들에 유일성 제약을 겁니다. 만약 제약이 걸린 attribute들의 값이 동일한 값이 들어오면, merge정책에 따라 기존 데이터를 덮어쓰거나 아예 저장이 안 될 수도 있습니다. 
    
  2. attribute설정  
    attribute는 객체가 가지는 데이터의 타입과 제약조건을 나타냅니다. entity가 클래스라면, attribute는 프로퍼티에 해당합니다. attribute를 추가하기 위해서는 attribute 항목의 + 버튼을 누르거나, 맨 아래 Add Attribute 버튼을 누르면 됩니다. attribute는 Entity내에서 고유한 이름을 가져야 하며, 타입 역시 명시해줘야 합니다. 예제에서는 다음과 같이 Person Entity를 선언하고, id와 name이라는 attribute를 만들겠습니다.  

    ![AttributeInspector]({{"/img/AttributeInspector.png"}}){: .center-block :}  

    attribute 설정의 주요 항목을 보면 다음과 같습니다.  

   1. Attribute Name: attribute의 이름입니다. 필수적으로 정해줘야 합니다. 
      * Transient: 해당 옵션을 선택하면, 이 attribute는 저장소에 저장이 되지 않습니다. 즉, 해당 attribute는 임시 데이터가 됩니다. 대신 앱이 실행되는 동안 해당 데이터의 변화를 추적할 수는 있습니다.
      * Optional: 해당 데이터가 저장소에 저장되는데 필수적이지 않음을 나타냅니다. 이는 Swift의 옵셔널과는 다르다는 점을 유의해야 합니다.
   
   2. Type: attribute가 가지는 데이터가 어떤 의미를 가지는지를 나타냅니다. Core Data가 지원하는 타입은 한정적이며, 다음과 같습니다.  
      
      * Integer16, Integer32, Integer64, Double, Float, Boolean: NSNumber에 해당하며, 참조 타입이 아닌 값 타입 자체로도 저장할 수 있습니다.
      
      * Date: Date(NSDate)에 해당합니다.
      
      * Decimal: NSDecimalNumber에 해당합니다.
      
      * UUID: UUID(NSUUID) 타입에 해당합니다.
      
      * URI: URL(NSURL)타입에 해당합니다.
      
      * String: String(NSString)타입에 해당합니다.
      
      * Binary Data: Data(NSData)타입에 해당합니다.
      
      * Transformable: NSObject 타입에 해당합니다.  
   
   3. Default Value: 대부분의 타입은 기본값을 지원합니다. 사용자가 다른 값을 대입하지 않는 이상은, 해당 attribute는 기본값을 가지게 됩니다. non-optional과 결합하면 성능개선을 기대할 수 있습니다.
   
   4. Validation: 숫자 타입이나 문자 타입 등 일부 타입은 가질 수 있는 값의 범위를 제한할 수 있습니다. 
   
   5. Use Scalar Type: 코드 자동생성을 할때, Scalar 타입(Int, Double 등)을 쓸지, NSNumber 같은 Non-Scalar 타입을 쓸지 정하는 옵션입니다.
   
   6. Preserve After Deletion: entity가 삭제된 후에도 undo 등을 위해 일부 데이터를 유지하는데, 해당 attribute를 여기에 포함시키는 옵션입니다.  
  
이어지는 포스트에서 Core Data의 기능에 대해서 좀 더 알아보도록 하겠습니다.

---  

> 참고 자료
> [Creating a Core Data Model](https://developer.apple.com/documentation/coredata/creating_a_core_data_model)  
> [Configuring Entities](https://developer.apple.com/documentation/coredata/modeling_data/configuring_entities)  
> [Configuring Attributes](https://developer.apple.com/documentation/coredata/modeling_data/configuring_attributes)  