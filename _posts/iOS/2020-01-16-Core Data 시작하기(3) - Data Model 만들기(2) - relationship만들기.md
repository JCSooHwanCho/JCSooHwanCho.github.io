---
layout: post
title: Core Data 시작하기(3) - Data Model 만들기(2) - relationship만들기
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

[지난 포스트](/2020-01-02-Core-Data-시작하기(2)-Data-Model-만들기(1)-entity만들기/)를 통해 Entity를 만들고 Attribute를 설정해보았습니다. 이번 포스트에서는 Entity간의 관계를 설정하는 방법을 알아보겠습니다.  

* Relationship 만들기
  Relationship은 어떤 entity가 변화될 때 다른 entity들에 미치는 변화를 표현한 것입니다. 예제를 위해, Computer라는 entity를 추가적으로 만들겠습니다.

  ![relationship initial State]({{"/img/relationshipInitialState.png"}}){: .center-block :}  

  Relation을 추가하기 위해서는 현재 뷰에서 작업하는 방법도 있지만, 오른쪽 아래에서 에디터 스타일을 바꿔서 작업하면 좀 더 직관적으로 설정할 수 있습니다. 

  ![Tree Style Editor]({{"/img/TreeStyleEditor.png"}}){: .center-block :}  

  에디터 스타일을 바꾼 상태에서, 컨트롤(Ctrl)키를 누른 채로 한 Entity를 클릭하여 다른 Entity로 드래그 하는 것으로 양방향 Relationship을 만들 수 있습니다. 이렇게 만들어진 Relationship은 각 Entity에 Relationship 항목에 추가 되어 있으며, 기본 이름을 가지고 있습니다.

  ![Default Relationship]({{"/img/DefaultRelationship.png"}}){: .center-block :}  

  이 Relationship을 코드로 표현하면 다음과 같습니다.  

  ```swift
    extension Computer {

        @nonobjc public class func fetchRequest() -> NSFetchRequest<Computer> {
            return NSFetchRequest<Computer>(entityName: "Computer")
        }

        @NSManaged public var id: UUID?
        @NSManaged public var model: String?
        @NSManaged public var owner: String?
        @NSManaged public var newRelationship: Person? // To One Relationship

    }

    extension Person {

        @nonobjc public class func fetchRequest() -> NSFetchRequest<Person> {
            return NSFetchRequest<Person>(entityName: "Person")
        }

        @NSManaged public var gender: Bool
        @NSManaged public var id: UUID?
        @NSManaged public var name: String?
        @NSManaged public var newRelationship: NSSet? // To Many Relationship

    }

    // MARK: Generated accessors for newRelationship
    extension Person {

        @objc(addNewRelationshipObject:)
        @NSManaged public func addToNewRelationship(_ value: Computer)

        @objc(removeNewRelationshipObject:)
        @NSManaged public func removeFromNewRelationship(_ value: Computer)

        @objc(addNewRelationship:)
        @NSManaged public func addToNewRelationship(_ values: NSSet)

        @objc(removeNewRelationship:)
        @NSManaged public func removeFromNewRelationship(_ values: NSSet)

    }
  ```  

  보다시피 Destination으로 지정한 객체의 참조(Class 타입이니까요)를 멤버로 가지는 형태로 구현됩니다. 때문에 Attribute와 거의 동일한 설정들을 공유합니다.  

* Relationship 설정하기
  Relationship을 선택하면, 여러가지 옵션들을 설정할 수 있습니다. 이 옵션들의 의미를 알아보도록 하겠습니다.  

  ![relationship Option]({{"/img/RelationshipOption.png"}}){: .center-block :}  

  1. Transient: 해당 Relationship이 임시적인 값인지 여부를 뜻합니다. 여기에 임시적인 속성으로 취급되어 저장소에 들어가지 않습니다.  
  
  2. Optional: 해당 Relationship이 선택적인지 여부를 뜻합니다. 선택하면, 해당 Relationship을 nil로 둘 수 있습니다. 만약 Optional이 아닌데 nil로 설정한 뒤 save를 시도하면 런타임 에러가 발생합니다.

  3. Destination: Relationship의 타입을 의미합니다. 자기 자신을 지정할 수도 있습니다.
  
  4. Inverse: 해당 Relationship의 역방향 Relationship을 의미합니다. 한쪽에서의 변화가 역방향으로도 전파될 수 있도록, 모든 Relationship은 Inverse를 가져야 합니다.
  
  5. Delete Rules: 자기 자신이 지워질 때, Relationship으로 연결된 Entity들에게 어떻게 변화가 전파되는지를 설정합니다. Inverse가 필수이기 때문에 Destination은 Source 참조를 가지고 있습니다. 
     
     * No Action: 아무런 행동을 하지 않습니다. 따라서 Destination에서의 참조는 그대로 유지되며, 이는 수동으로 업데이트 되어야 합니다.
     
     * Nullify : Destination의 Source 참조을 nil로 설정합니다.
     
     * Cascade : Destination 객체들을 연쇄적으로 삭제합니다.   
     
     * Deny: 아무런 Destination을 가리키지 않을 때만 삭제가 가능하며, 하나라도 다른 객체 참조를 가지고 있으면 삭제가 거부됩니다.  
     
  6. Type: 해당 Relationship이 1:1 관계인지, 1:N 관계인지를 결정합니다. 1:N 관계인 경우, 최소 숫자와 최대 숫자를 정할 수 있습니다.  


Relationship을 설정하여 객체 그래프를 만드는 과정은 많은 고민이 필요합니다. Entity와 Relationship에서 설정할 수 있는 것들을 잘 참조해서 그래프를 구성해봅시다.

---  
> 참조 자료
> [Configuring Relationships](https://developer.apple.com/documentation/coredata/modeling_data/configuring_relationships)  
> [Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/HowManagedObjectsarerelated.html#//apple_ref/doc/uid/TP40001075-CH17-SW1)  