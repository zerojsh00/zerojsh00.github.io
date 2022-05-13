# gym-inventory

이 예제는 open AI gym에서 공유되어 있는 재고관리 문제를 발전시켜 재구성하였습니다. 단일 에이전트가 이산적 상태(state)에서 이산적 행동(action)을 취함으로써 재고를 고려하여 최적 주문량을 결정하는 문제입니다.

## Simple Inventory Control with Lost Sales

본 환경은 lost sales 문제가 있는 재고관리 환경으로, 문제에 대한 자세한 내용은 [Algorithms for Reinforcement Learning by Csaba Szepesvari (2010)](https://sites.ualberta.ca/~szepesva/RLBook.html) Example 1.1.을 참고하세요.


## Installation

```
conda create -n inventory python=3.6
source activate inventory

pip install -e .
pip install -r requirements.txt
```

## Contribution
본 데모 시뮬레이션에서의 주요 기여사항은 다음과 같습니다.

본 시뮬레이션에서 활용된 알고리즘인 Q-learning에서는 재고의 상황을 강화학습의 state로, 주문량을 action으로 하는 Q-table로 이루어져 있습니다. 관례적 방법대로라면, Q-table은 행렬을 무작위로 초기화하거나 0으로 초기화하는 방법을 사용합니다. 실제로 본 문제에서도 두 초기화 방법으로 Q-table 행렬을 초기화 하여도 학습 시 reward가 안정적으로 수렴할 수 있기 때문에 agent가 성공적으로 학습했다고 착각할 수 있습니다. 그러나 학습이 완료된 후 policy를 분석해보면, 재고 수준과 무관하게 매우 엉뚱한 양을 주문하는 것이 최적이라고 분석될 것입니다.

그러나, 위와 같은 경우는 재고를 저장할 수 있는 최대 용량(capacity)이 정해져 있음에도 불구하고 최대 용량을 넘어서는 주문까지 허용되는 오류가 존재합니다. 이러한 상황을 해결하기 위해서 NLP 분야 Transformer network의 decoder에서 사용되는 masked self-attention의 컨셉을 적용하였습니다. 즉, 관례적인 초기화 기법을 버리고, Q-table의 하삼각행렬 부분을 매우 작은 음수로 초기화하여 마스킹한 후 학습을 진행하니, reward 수렴이 훨씬 안정화될 뿐 아니라, agent가 재고의 최대 용량까지 고려한 최적 주문량을 결정하는 policy를 학습하는 데 성공했습니다.
