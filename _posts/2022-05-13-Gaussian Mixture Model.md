---
title: Gaussian Mixture Models
author: simon sanghyeon
date: 2022-05-16
categories: [Anomaly Detection]
tags: [Anomaly Detection, Density-based Novelty Detection]
render_with_liquid: true
use_math: true
---
이 포스트는 개인적으로 공부한 내용을 정리하고 필요한 분들에게 지식을 공유하기 위해 작성되었습니다.<br>
지적하실 내용이 있다면, 언제든 댓글 또는 메일로 알려주시기를 바랍니다.

# Gaussian Mixture Model의 개요
`Gaussian Mixture Model`은 `Density-based Novelty Detection` 기법의 일종으로, 정상 데이터로부터 `확률 밀도 함수`를 구하고,
해당 확률 밀도 함수를 기반으로 하여 클러스터링을 수행함으로써 이상 데이터 포인트를 분류하는 비지도학습 기법입니다.

![gaussian_ex](/assets/img/2022-05-13-Gaussian%20Mixture%20Model/gaussian_ex.png)
*위 그림은 고려대 강필성 교수님의 강의에서 가져왔습니다.*

사실 우리는 우리의 데이터가 가우시안 분포를 따르는지 정확하게 알 수 없습니다. 그러나 가우시안 분포로 가정하고 문제를 푸는 것이죠.
위 그림을 보면서 쉽게 이해해보면, 데이터의 분포가 가우시안 분포라고 볼 때, 어떤 새로운 데이터가 앞서 정상 데이터로 구한 가우시안 확률 밀도 함수 영역의 밖에 존재하면 이상치라고 판별할 수 있습니다.
그런데 사실 우리가 바라보는 데이터가 이와 같이 하나의 가우시안 분포만을 따른다는 가정은 현실적이지 않습니다.

![Gaussian Mixture ex](/assets/img/2022-05-13-Gaussian%20Mixture%20Model/Gaussian%20Mixture%20ex.png)
*위 그림은 [여기](https://tinyheero.github.io/2015/10/13/mixture-model.html)에서 가져왔습니다.*

만약 우리의 데이터가 위와 같이 분포되어 있다면, 하나의 가우시안 분포로는 우리의 데이터를 모델링하기 어려울 것이고, 위와 같이 여러 가우시안 분포를 사용해야 할 것입니다.
이렇게 여러 가우시안 분포를 혼합하기 때문에 Gaussian Mixture라고 부릅니다.

# Gaussian Mixture Model의 구성 요소
가우시안 분포를 어떻게 혼합할까요? 바로, 가우시안 분포들의 선형 결합으로 혼합할 수 있습니다.
이는 아래와 같이 표현할 수 있습니다.
<center>$P(\mathbf{x}|\lambda) = \sum_{m=1}^{M}w_{m}g(\mathbf{x}|\boldsymbol{\mu}_m,\boldsymbol{\Sigma}_m)$
</center><br>

이때 $\mathbf{x}$는 다차원으로 이루어진 벡터 데이터이며, $\lambda$는 우리가 추정하고자 하는 미지수들의 집합 $\lambda=\\{w_m, \boldsymbol{\mu}_m,\boldsymbol{\Sigma}_m\\}$을 나타냅니다.
이들 중 첫번째 원소인 $w_m$은 `특정 가우시안 분포를 선택할 일종의 가중치`로 볼 수 있습니다. 이에 대해서는 뒤에서 다시 살펴보겠습니다.

한편, $\boldsymbol{\mu}_m$와 $\boldsymbol{\Sigma}_m$는 각각 Multivariate Gaussian Distribution(다변수 가우시안 분포)의 `평균 벡터`와 `공분산 행렬`을 의미합니다.
이에 따라 혼합되는 복수의 가우시안 중 $m$번째 가우시안 분포 $g(\mathbf{x}\|\boldsymbol{\mu}_m,\boldsymbol{\Sigma}_m)$는 다음과 같이 나타낼 수 있습니다.
<center>
$g(\mathbf{x}|\boldsymbol{\mu}_m,\boldsymbol{\Sigma}_m)=\cfrac{1}{(2\pi)^{d/2}|\boldsymbol{\Sigma_m}|^{1/2}} exp \left[ \cfrac{1}{2}(\mathbf{x}-\boldsymbol{\mu}_m)^{T} \boldsymbol{\Sigma}_m^{-1}(\mathbf{x}-\boldsymbol{\mu}_m) \right]$
</center><br>




# 작성중 ....



# 참고자료
[1] [고려대 강필성 교수님의 강의](https://www.youtube.com/watch?v=kKZM8bxwQbA&list=PLetSlH8YjIfWMdw9AuLR5ybkVvGcoG2EW&index=17)<br>
[2] ['분석가꽁냥이'님의 블로그](https://zephyrus1111.tistory.com/183?category=858748#c1)
