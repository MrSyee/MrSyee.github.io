---
title: Priortized Experience Replay의 Sum tree 구현
layout: post
categories: [RL]
tags: [AI, Reinforcement learning]
---
## Sum tree로 구현하는 PER

강화학습에서 많이 쓰이는 Replay buffer 중 PER는 Sum tree로 구현하는 것이 효율적이라고 논문에서 언급한다. Sum tree는 무엇이고 어떤 식으로 구현하는지 정리한다.

### Segment Tree

Sum tree는 Segment tree의 부분집합이다. 먼저 Segment Tree에 대해 알아본다.

![image](https://user-images.githubusercontent.com/17582508/51464350-b0d57080-1da8-11e9-9438-d1c4cd16ce53.png)

![image](https://user-images.githubusercontent.com/17582508/51464373-c054b980-1da8-11e9-9c79-0056e5bcacf5.png)

Image Ref : https://www.acmicpc.net/blog/view/9

- 완전 이진 트리

- Segment  뜻 : 구획, 구간

- 부모 노드는 각 자식노드들의 **범위(구간)**에 대한 정보를 갖고 있다. (e.g. 그 범위에 대한 합, 그 범위에서의 최소값)

- **실제 데이터는 맨 아래의 leaf 노드들이 가지게 됨.**

- Sum Tree는 Segment tree의 부분집합

- ex.

  - [4, 5, 1, 3] 의 값을 Sum tree로 구성한다면

  - ![image](https://user-images.githubusercontent.com/17582508/51532280-0a11d280-1e83-11e9-8c4e-6839e1a6d971.png)

  - [None, 13, 9, 4, 4, 5, 1, 3]



#### When to use Segment Tree

특정 범위의 element들에 대한 연산값들(e.g. sum, max, min)을 미리 구해 저장해 두기 때문에 빠르게 접근(Find)하거나 수정(Update)할 수 있다.

- Sum all elements in a range.
- Find the min or max value of elements in a range.
- Update all elements in a range.

PER은 버퍼에 보관하고 있는 td error의 총합에 대한 비율로 prior를 결정하기 때문에 합을 계속해서 계산해주어야 한다. Sum tree를 사용하면 update를 하는데 O(log N)의 시간복잡도를 가진다. (일반 List로 구현할 경우 O(N))

### Sampling

![image](https://user-images.githubusercontent.com/17582508/51665898-18d1c400-2000-11e9-88d5-c3cf04d459e4.png)

sampling의 경우도 O(log N)의 시간복잡도를 가진다. 논문에서는 각 transition에 대한 prior $p_i $ 에 대한 확률분포 ${p^{\alpha}_{i}}  \over {\sum_{k}p^{\alpha}_{k}}$ 를 따라 transition을 sampling 한다. 하지만 실제 Sum tree로 구현했을때 분포를 따로 계산하는 것이 아니라 tree를 순회하는 로직으로 확률분포에서 sampling 했을 때와 똑같은 똑같은 확률로 sampling 한다 (TD error가 높은 transition을 더 많이 sampling).  Sum tree가 위 그림과 같다고 가정 했을 때 sampling 하는 로직을 소개한다. [[OpenAI 코드](https://github.com/openai/baselines/blob/master/baselines/common/segment_tree.py) 참조함.]

- hyperparmeter 예시
  - batch size : 4

1. 총 합(Tree의 root)인 24 만큼의 구간을 설정하고  batch size 크기로 나눈다.
   - [0, 24] : [0, 6], [6, 12], [12, 18], [18, 24]
2. 각 구간에서 랜덤한 값을 샘플링 : S
   - 4, 9, 16, 20
3. 샘플링한 값을 이용해 다음의 로직으로 leaf node까지 retrieve
   1. root node 에서 시작 (index: 1)
   2. left node가 S 보다 크면 index = left node index
   3. 작거나 같으면 index = right node, s = s - val(left node)
   4. leaf node에 도착할 때까지 반복
4. 이 작업을 거치면 전체 구간이 leaf node 값의 비율로 나눠진다.
   - 위의 예시를 보면 아래 그림과 같은 비율로 나눠진다.
   - ![image](https://user-images.githubusercontent.com/17582508/51676832-26e10e00-201b-11e9-9ebb-17de98e4cef6.png)
   - 각 leaf node는 prior를 가지고 있으므로 prior에 따라 해당 prior의 node가 선택된다.
   - ex. 10 prior를 갖는 node는 10/24 확률로 선택됨.
5. 위 로직을 따르면 해당 prior가 뽑힐 확률이 ${p^{\alpha}_{i}}  \over {\sum_{k}p^{\alpha}_{k}}​$ 된다.



### PER 코드 검증

PER 코드 : [Link](https://github.com/medipixel/reinforcement_learning_examples/blob/master/algorithms/priortized_replay_buffer.py)

- Sampling Test
  - 0 ~ 100 용량의 PER에 100에 가까울수록 prior를 크게 넣었을 때 각 인덱스 별 sampling 된 횟수

![image](https://user-images.githubusercontent.com/17582508/51665406-0905b000-1fff-11e9-8cb4-8b6f37d77f68.png)

- Update Test
  - - 위와 같은 상태에서 0 ~ 30 구간만 prior를 더 큰 값으로 update 했을 시 sampling 결과

![image](https://user-images.githubusercontent.com/17582508/51665461-289cd880-1fff-11e9-9e0e-9df76eb975da.png)

## Reference

- https://www.acmicpc.net/blog/view/9

- http://snowdeer.github.io/algorithm/2016/03/28/segment-tree/

- https://hackernoon.com/practical-data-structures-for-frontend-applications-when-to-use-segment-trees-9c7cdb7c2819

- https://github.com/openai/baselines/blob/master/baselines/common/segment_tree.py
