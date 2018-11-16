---
title: Unity ML Agent - 실습 준비와 Push Block 예제 학습해보기
layout: post
categories: [RL]
tags: [ai, reinforcement learning, unity]
---
## Introduction
강화학습 공부, 더 나아가 강화학습을 실제 문제에 적용을 해야할 때 가장 걸림돌이 되는 것이 바로 강화학습 Agent를 학습(시뮬레이션) 시킬 **환경(envirionment)** 입니다. 로봇을 학습시키려면 직접 로봇을 만들어야 하고 자율주행 자동차를 학습시키려면 자동차를 직접 도로에 주행시켜야 합니다. 하지만 실제 환경을 이용해 시뮬레이션 하는 것은 많은 비용과 위험부담이 따릅니다. 이러한 문제 때문에 실제 환경에 학습하기 보다 현실과 유사한 **가상의 시뮬레이션 환경** 을 만들고 그 환경에서 학습시킨 모델을 실제 환경에 적용합니다. 그러나 이러한 현실과 유사한 가상 공간을 만드는 것도 전문가 수준이 아니면 쉽지 않습니다.  

게임 엔진으로 잘 알려진 **Unity** 는 이런 가상 환경을 쉽게 만들 수 있게 해주는 툴입니다. 물론 정교하게 만들려면 많은 기술이 필요하겠지만 간단한 물리법칙들은 쉽게 적용할 수 있기 때문에 비교적 어렵지 않게 시뮬레이션 환경을 구축 할 수 있습니다. 최근 Unity에서 이러한 장점을 살려 Unity로 만든 가상 환경에 쉽게 인공지능 알고리즘을 적용할 수 있게 해주는 [Unity ML-Agents Toolkit](https://github.com/Unity-Technologies/ml-agents/tree/master/docs) 을 배포하고 업데이트하고 있습니다.  

저는 위 Github에 올라온 Unity ML-Agents Toolkit의 매뉴얼을 보고 실습해본 내용을 공유해보려 합니다. Unity ML-Agents의 스터디는 **모두의연구소** 의 강화학습 랩인 **CTRL Lab** 에서 함께 하고 있습니다.

## 목표
1. 매뉴얼에 올라온 예제들을 학습해보고 test 해본다.
2. 예제 환경의 일부를 고쳐서 새로운 환경을 만들어보고 학습해본다.
3. 강화학습 알고리즘도 예제 코드가 아닌 새로운 코드를 적용해본다.
4. 원하는 환경을 만들어서 강화학습을 적용해본다.

## Settings
[Unity ML-Agents Toolkit](https://github.com/Unity-Technologies/ml-agents/tree/master/docs)에 있는 매뉴얼을 통해 기본적인 Unity의 설치와 ML-Agent를 `clone` 합니다.
- [Unity ML-Agents Toolkit 설치 매뉴얼](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Installation.md)
- [Unity ML-Agents Toolkit 윈도우용 설치 매뉴얼](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Installation-Windows.md)  

```
git clone https://github.com/Unity-Technologies/ml-agents.git
```

저는 아래와 같은 환경에서 세팅했습니다.
- Window 10
- Unity 2018.2.16f1
- conda 4.3.30
- python 3.6.0
- tensorflow-gpu 1.7.1
- Cuda 9.0.176 & cudnn 7.0.5

### 파이썬 세팅

저는 Windows에서 세팅했기 때문에 Anaconda를 사용했습니다. Anaconda에 대한 설명은 이 포스트에서 자세히 다루지는 않겠습니다. 간단하게 얘기하면 파이썬 패키지를 가상환경 단위로 관리할 수 있게 해주는 도구입니다. Anaconda 설치 후에 가상환경을 만듭니다. 저는 `unity_ml` 이라는 이름으로 만들었습니다. 그리고 가상환경을 활성화합니다.
```
conda create -n unity_ml python=3.6
activate unity_ml
```
이제부터 설치하는 파이썬 패키지들은 `unity_ml` 이라는 가상환경에 설치됩니다. 이전에 `clone` 한 디렉토리로 가서 Unity ML-Agents도 설치해봅시다.
```
cd ml-agents
pip install .
```
설치가 완료되면 `mlagents-learn` 명령어를 사용할 수 있습니다.  

![unity_mlagents1](https://user-images.githubusercontent.com/17582508/48579901-0df41680-e961-11e8-990f-e7a1c235f07a.PNG)

### Unity 세팅
Unity 쪽의 세팅과 예제 실행은 다음 문서를 참고합니다.
- [Unity ML-Agents Toolkit Basic Guide](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Basic-Guide.md)

먼저, Unity 프로젝트를 하나 생성합니다. 그 후 생성한 프로젝트 폴더의 `Assets` 폴더에 이전에 `clone` 한 `ml-agents` 폴더의 `UnitySDK/Assets` 폴더 안에 있는 `ML-Agents` 폴더와 `ML-Agents.meta` 파일을 복사합니다. 이 폴더안에 Unity ML-Agent의 주요 코드들과 예제가 담겨 있습니다.  
여기까지가 위 가이드의 3번까지 입니다. 그 후 위 가이드의 4번 부터 차례로 세팅합니다.  

추가로 [Tensorflow sharp 플러그인](https://github.com/Unity-Technologies/ml-agents/blob/master/docs/Basic-Guide.md)을 설치해야합니다. 링크의 플러그인을 Unity 프로젝트를 실행시킨 채로 플러그인을 실행시키면 `Assets/ML-Agents/Plugins` 경로에 설치됩니다.

여기까지 완료하시면 예제 실행을 위한 준비는 끝납니다. 이제 Push Block 예제를 실행해봅시다.

## Push Block 예제
![pushblock](https://user-images.githubusercontent.com/17582508/48623459-0e39f380-e9ed-11e8-867d-107db4dc3bf3.PNG)

Push Block 예제는 파란색 블럭인 Agent가 주황색 블럭을 목표 지점까지 밀어서 옮기는 것을 목표로 합니다.

Push Block 환경이 구현된 Scene 파일은 `Assets/ML-Agents/Examples/PushBlock/Scenes`에 있습니다.

![pushblock_scene](https://user-images.githubusercontent.com/17582508/48618453-c8752f00-e9dc-11e8-82b5-65054680d8bf.PNG)

Scene을 열면 위 그림과 같은 환경이 나옵니다. 똑같은 환경이 32개가 있는데 이는 32개의 시뮬레이션 경험을 모아 학습속도를 빠르게 하기 위함입니다. Unity 환경에 강화학습 알고리즘을 적용하려면 `Brain` object를 설정해야 합니다. Unity 창에서 `Hierarchy` 탭의 `Academy/PushBlockBrain`를 클릭한 후 `Brain` component에서 `Brain Type`을 **External** 로 변경합니다.
- Internal : 학습된 파라미터 파일을 이용해 Agent를 조종합니다.
- External : 외부 Python 파일을 이용해 Agent를 조종합니다. 학습 시 사용합니다.
- Player : 직접 조종합니다.
- Heuristic : 행동을 결정하는 알고리즘을 구현해 Agent를 조종합니다. (직접 해보지 않아 확실하지 않습니다.)

![brain_](https://user-images.githubusercontent.com/17582508/48621822-222f2680-e9e8-11e8-8b95-d9d112d2c6f0.PNG)

이제 강화학습 알고리즘을 이용해 Agent를 학습해봅시다. 저는 Unity ML-Agent Toolkit에서 기본적으로 제공하는 강화학습 알고리즘인 `PPO` 알고리즘의 코드를 사용하려 합니다.  
먼저 이전에 Anaconda 가상환경을 실행시킨 cmd 창(또는 Anaconda Prompt 창)에서 `clone` 한 `ml-agents` 폴더의 경로로 이동합니다. 그리고 다음 명령어를 입력합니다.

```
mlagents-learn config/trainer_config.yaml --run-id=firstRun --train
```

이 명령어를 사용하면 Unity ML-Agent Toolkit에서 제공해주는 `config/trainer_config.yaml` 파일에 미리 정해준 파라미터와 각종 설정들을 불러와 학습을 실행시킵니다. 이 명령어를 통해 제공되는 `PPO` 알고리즘 코드로 현재 Unity 환경의 Agent를 학습할 수 있게 됩니다. `run-id` 는 학습 결과를 구분하기 위한 구분자로 원하는 string을 입력하면 됩니다. 실행시키면 다음과 같은 창이 나옵니다.

![train](https://user-images.githubusercontent.com/17582508/48622363-d7aea980-e9e9-11e8-9121-1aa538854efa.PNG)

이 상태에서 Unity 화면으로 가서 Play(▶) 버튼을 누르면 현재 환경에서 Agent 학습을 시작합니다. 저는 20000 step 정도 학습시켰고 10000 step 정도부터 점점 Agent가 주황색 블럭을 목표 지점으로 잘 미는 것을 확인할 수 있었습니다.

![pushblock_train](https://user-images.githubusercontent.com/17582508/48622448-28be9d80-e9ea-11e8-9356-3a09a146e6a5.PNG)

학습이 어느 정도 완료되면 학습을 멈추고(ctrl+c) `ml-agents` 폴더에서 `models` 폴더에 위에서 설정한 `run-id` 이름의 폴더가 생성된 것을 확인할 수 있습니다. 저는 `firstRun`이라고 지정했기 때문에 그 이름의 폴더가 생겼습니다. 이 폴더 안의 파일들이 학습된 Agent와 신경망의 파라미터를 가지고 있습니다. 이 폴더를 Unity 프로젝트 폴더의 `Assets` 폴더에 `Model` 폴더를 만든 후 그 폴더로 옮깁니다. 그 후 Unity 화면에서 `Academy/PushBlockBrain`를 클릭한 후 `Brain` component에서 `Brain Type`을 **Internal** 로 변경하고 `Graph Model`을 이전에 옮긴 `Model/firstRun' 경로의 'editor_Academy_firstRun-0.bytes` 파일로 지정합니다. 이 작업은 Unity 화면에서 마우스로 Drag & Drop을 통해 쉽게 지정할 수 있습니다.

![interal](https://user-images.githubusercontent.com/17582508/48623215-5f95b300-e9ec-11e8-9c44-bd08009dbd16.PNG)

그 후 Play(▶) 버튼을 누르면 학습한 Agent가 실행됩니다. 아래는 학습한 Agent를 실행한 화면입니다.

![push_block_train](https://user-images.githubusercontent.com/17582508/48623358-c2874a00-e9ec-11e8-808a-6c31978d8e1f.gif)

## 다음에 해볼 것
Push Block 예제를 기본적으로 제공하는 `PPO` 알고리즘으로 학습해보았습니다. 다음으로는 위 Push Block 환경을 조금씩 변화시켜 새로운 환경을 만들고 Agent를 학습시켜 보려합니다.
