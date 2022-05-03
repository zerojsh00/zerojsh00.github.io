---
title: 공부하며 정리하는 인과추론 02
author: simon sanghyeon
date: 2022-05-04
categories: [Causal Inference]
tags: [Causal Inference, Root Cause Analysis]
render_with_liquid: true
---
이 포스트는 개인적으로 공부한 내용을 정리하고 필요한 분들에게 지식을 공유하기 위해 작성되었습니다.<br>
지적하실 내용이 있다면, 언제든 댓글 또는 메일로 알려주시기를 바랍니다.

본 포스트의 상당 내용은 [`인과추론의 데이터과학`](https://www.youtube.com/channel/UCkEHnPq2T8Vpafk3p-Rk49A) 강의를 참고하였음을 밝힙니다.

# 지난 이야기
복잡한 IT 환경에서 이상징후 및 장애가 발생되었을 때, 이를 인과추론의 방식으로 해결해보고자 하는 큰 꿈을 품고 첫 포스트인 [공부하며 정리하는 인과추론 01](https://zerojsh00.github.io/posts/Causal-Inference-01/)을 올렸습니다.
기계학습 기법에 익숙했던 필자에게는 다소 생소한 데이터 분석 분야라는 것을 알게되었고, 어렵지만 차근차근 공부하며 정복해보고자 하는 욕심이 생겼습니다. 또한 추천시스템 도입 사례와 심슨의 역설 예제를 통해서 인과관계를 어떻게 정의하는지에 따라 인과추론 결과가 달라질 수 있다는 점도 살펴보았습니다.

이번 포스트에서는 인과추론의 양대산맥인 `Potential Outcomes Framework`와 `Structural Causal Model(SCM)`의 개괄을 살펴보려 합니다.
이번 포스트도 역시나 인과추론을 처음 접하는 필자가 [`인과추론의 데이터과학`](https://www.youtube.com/channel/UCkEHnPq2T8Vpafk3p-Rk49A) 강의를 듣고 해당 내용을 중심으로 정리합니다.
따라서 자세한 내용은 해당 강의를 보시길 추천드립니다.

# 인과추론의 양대산맥
인과관계를 알아내기 위한 인간의 고민은 아리스토텔레스 시대까지도 거슬러 올라갈 수 있을 만큼 굉장히 오래된 역사를 가진다고 합니다([참고](https://namu.wiki/w/%EC%9D%B8%EA%B3%BC)).
인간의 사유에 근거한, 철학적인 접근으로 인과관계가 무엇인지 고민했던 것이죠.
현대는 데이터의 시대입니다. 이에 따라, 인간의 사유에만 의존했던 과거의 철학적 접근 방식을 너머, 더욱 시스템적이고 과학적인 방법으로 인과관계를 분석하려는 방식이 개발되었습니다.

![frameworks](/assets/img/2022-05-04-Causal Inference 02/frameworks.png)

위 그림이 바로 그것인데요. 이들은 인과추론의 양대산맥으로, 왼쪽은 Potential Outcomes Framework라는 인과추론 방식이며, 오른쪽은 Structural Causal Model이라는 인과추론 방식입니다.
Potential Outcomes Framework는 하버드의 Donald Rubin 교수님이 만든 방법이며, Structural Causal Model은 UCLA의 Judea Pearl 교수님이 개발했다고 합니다.
무엇이 다를까요? 구체적인 방법론의 상세 내용은 추후에 하나씩 다루겠지만, 지금은 간략하게만 짚고 가봅시다.

Potential Outcomes Framework는 '만약 타임머신을 타고 과거로 돌아갔는데 그 원인이 없었더라면 현재 결과는 어땠을까?'라는 관점에서 인과관계를 정의합니다.
제목 그대로 Potential Outcome, 즉, `잠재적 결과`를 이용하여 인과관계를 보려는 것이죠. 반면에, Structural Causal Model은 인과관계로 표현할 수 있는 변수들을 `인과그래프`로 모델링한 후, 인과관계를 분석해보려는 시도입니다.

이 두 방법 중 어느 방법이 특별히 더 우수하다거나 상호배타적인 관계라고 할 수는 없습니다. 다만, 결이 다른 접근 방법이라고 보면 좋을 것 같습니다.
Potential Outcomes Framework의 경우, 해당 방법론의 초석을 다진 연구자들이 최근 노벨 경제학상을 받았다고 합니다.
한편, Structural Causal Model을 개척한 Judea Pearl 교수님은 컴퓨터과학 분야에서 가장 권위 있는 상인 Turing Award를 수상했다고 합니다.
이러한 단편적인 예만 보아도 두 방식의 차이가 있는데, 전자는 사회과학 분야를 바탕으로 발전되어 왔으며, 후자는 컴퓨터과학 분야를 바탕으로 발전되었다고 합니다.
중요한 것은, 결이 다른 두 방법론을 잘 이해하고, 우리가 풀고자 하는 문제에 더 적합한 방법론이 무엇인지 결정하여 잘 활용하는 데 있습니다.

# 훑어보는 Potential Outcomes Framework
(작업 중)

# 훑어보는 Structural Causal Model
(작업 중)
