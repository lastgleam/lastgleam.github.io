---
layout: post
title:  "2018-02-22 TIL"
author: "lastgleam"
—
#javascript에서의 `==`와 `===`의 차이
다들 처음 자바스크립트를 처음배울 때 자주 듣는 말이 있다.  
> 왠만하면 `===`으로 값 비교해라

입문서나 각종 튜토리얼에서 자주 접할 수 있는 얘기이나, 전혀 와닿지 않는다.
Mozilla Developer Network의 문서를 살펴보니 이렇게 설명하고 있다.
> 간단히 말해서, 이중 등호는 두 값을 비교할 때 형 변환(type conversion)을 수행합니다. 반면에 삼중 등호는 형 변환 없이 같은 비교를 행합니다 (형이 다른 경우 단순하게 항상 false를 반환하여)

예를들어 `==`연산의 경우에는
```
true == 1 //true 
123 == '123' //true
null == undefined //true
```
라는 결과를 얻게된다.
이와 달리 `===`연산의 경우는? 형변환을 거치지 않고 비교하기 때문에
```
true === 1 //false
123 === '123' //false
null === undefined //false
```
와 같은 결과를 얻고 만다.
결론적으로 차이를 이해하고 개발자 자신이 '적절'하게 사용하는 것이 중요하다고 하겠다.

출처 : [개발자를 위한 웹 기술 > JavaScript > 같음 비교 및 똑같음
](https://developer.mozilla.org/ko/docs/Web/JavaScript/Equality_comparisons_and_sameness)
