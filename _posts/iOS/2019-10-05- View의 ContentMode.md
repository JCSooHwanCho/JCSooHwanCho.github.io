---
layout: post
title: View의 ContentMode
comments: true
tags: [iOS 프로그래밍,Swift,Apple]
category: [iOS]
---  

UIView에는 ContentMode라는 속성이 있습니다. 뷰는 내부적으로 자신이 띄울 컨텐츠의 비트맵 데이터를 캐싱하고 있는데, 뷰의 bounds가 변했을 때(물론 ContentMode는 최초로 컨텐츠를 그릴 때도 영향을 미칩니다.) 매번 컨텐츠를 새로 그리는 것은 비용이 많이 드므로, 캐싱하고 있던 비트맵 데이터를 활용하여 빠르게 반응을 하게 됩니다. 이 때 뷰는 컨텐츠의 사이즈를 어떻게 조절할지, 어디에 배치할지를 결정해야 하는데, 이러한 정책을 결정하는 것이 바로 ContentMode입니다. 

contentMode는 UIView의 인스턴트 프로퍼티이며, UIView 내부에 열거형으로 정의되어 있습니다.

1. ScaleToFill - default값입니다.
   ![ScaleToFill]({{"/img/ContentMode/ScaleToFill.png"}}){: .center-block :}  

   컨텐츠를 뷰의 크기에 딱 맞게 크기를 조절합니다. 이 때, 컨텐츠의 원래 비율이 유지되지 않아 컨텐츠가 찌그러져 보일 수 있습니다. 

2. AspectFit(ScaleAspectFit)
   ![AspectFit]({{"/img/ContentMode/AspectFit.png"}}){: .center-block :}  

   컨텐츠를 원래의 비율을 유지하면서 뷰의 크기에 맞춥니다. 이 때, 컨텐츠가 잘리지 않을 정도로만 컨텐츠 크기를 조절하며, 채워지지 않은 뷰 영역은 투명한 상태가 되어 뷰의 배경색이 그대로 드러나게 됩니다.

3. AspectFill(ScaleAspectFill)
   ![AspectFill]({{"/img/ContentMode/AspectFill.png"}}){: .center-block :}  

   AspectFit처러 컨텐츠의 원래 비율을 유지하지만, 뷰 영역 전체를 덮을수 있을 정도로 크기를 조절합니다. 이때 컨텐츠가 일부 잘릴 수 있습니다.

4. Redraw
   ![redraw]({{"/img/ContentMode/Redraw.png"}}){: .center-block :} 

   캐시된 비트맵을 사용하지 않고, 매번 컨텐츠를 새로 그립니다. 이 때 [setNeedsDisplay()](https://developer.apple.com/documentation/uikit/uiview/1622437-setneedsdisplay)가 호출됩니다.  

이후의 옵션들은 컨텐츠의 크기를 조절하지 않고, 컨텐츠가 배치되는 위치에만 영향을 미칩니다.

1. Center
   ![center]({{"/img/ContentMode/Center.png"}}){: .center-block :} 

   뷰의 중앙에 컨텐츠가 배치됩니다.

2. Top
   ![top]({{"/img/ContentMode/Top.png"}}){: .center-block :} 

   뷰의 상단 중앙에 컨텐츠가 배치됩니다.

3. Bottom
   ![bottom]({{"/img/ContentMode/Bottom.png"}}){: .center-block :} 

   뷰의 하단 중앙에 컨텐츠가 배치됩니다.

4. Left
   ![left]({{"/img/ContentMode/Left.png"}}){: .center-block :}

   뷰의 좌측 중앙에 컨텐츠가 배치됩니다.

5. Right
   ![right]({{"/img/ContentMode/Right.png"}}){: .center-block :}
   
   뷰의 우측 중앙에 컨텐츠가 배치됩니다. 

6. topLeft
   ![topLeft]({{"/img/ContentMode/TopLeft.png"}}){: .center-block :} 

   뷰의 좌측 상단에 컨텐츠가 배치됩니다. 

7. topRight
   ![topRight]({{"/img/ContentMode/TopRight.png"}}){: .center-block :} 

   뷰의 우측 상단에 컨텐츠가 배치됩니다. 

8. bottomLeft
   ![bottomLeft]({{"/img/ContentMode/BottomLeft.png"}}){: .center-block :} 

   뷰의 좌측 하단에 컨텐츠가 배치됩니다. 

9. topLeft
   ![bottomRight]({{"/img/ContentMode/BottomRight.png"}}){: .center-block :} 

   뷰의 우측 하단에 컨텐츠가 배치됩니다. 

---
contentMode는 이미지뷰 등에서 특히 중요하게 사용되므로, 그 차이를 알고 적절하게 사용할 필요가 있습니다.