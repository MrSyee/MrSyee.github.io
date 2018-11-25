---
title: Show and Tell - A Neural Caption Generator 논문 리뷰
layout: post
categories: [NLP]
tags: [ai, nlp]
---
## Introduction
[Show and Tell: A Neural Caption Generator](https://arxiv.org/abs/1411.4555) 논문을 리뷰한 포스트입니다. 이미지를 보고 설명하는 글을 만들어내는 **Image Captioning** 묹제를 CNN과 LSTM을 조합한 end-to-end 구조의 신경망으로 풀었고 그 당시 SOTA(State-of-the-art)를 갱신했다는 내용입니다.

이 자료는 **모두의연구소** 의 풀잎스쿨반인 **NLP Boot Camp** 에서 발표한 내용입니다.

### Contents
- Introduction
- Model
- Experiments
- Results

---

# Introduction  

- 이미지를 설명하는 문장을 자동으로 만들어 내는것(Image description)은 굉장히 어려운 문제다. (challenging task)
- image classification과 object dection보다 훨씬 어렵다.  

	### $\rightarrow$ 이미지 인식 + 자연어 표현까지 학습해야 되기 때문

---

## Contribution

1. Image description 문제를 푸는 end-to-end 시스템을 제안함.
2. 논문에서 제안하는 모델은 vision과 language 모델 중에 SOTA인 모델들의 구조의 일부를 조합해서 만듬.
3. 기존 SOTA 보다 더 좋은 성능을 보여줌.

---

## Idea
- **encoder** RNN - **decoder** RNN
- Source language $S$ - Target language $T$

<center>

  **maximizing $p(T|S)$** 하도록 학습
   <img src=https://user-images.githubusercontent.com/17582508/48945949-7c436500-ef6f-11e8-982d-32a6c06fa426.png width=100% height="100%">

</center>

---

## Idea
- **encoder** RNN $\rightarrow$ **encoder CNN**
- image $I$
- target sequence of words $S = \{ S_1, S_2, ... \}$  
- **Neural Image Caption** model **(NIC)**

<center>

  **maximizing $p(S|I)$** 하도록 학습
  <img src=https://user-images.githubusercontent.com/17582508/48946312-a0ec0c80-ef70-11e8-8436-e335efd2145a.png width=70% height="70%">

</center>

> 이 논문에서 새롭게 제안하는 모델의 이름은 Neural Image Caption(NIC)입니다. 이전에 나온 논문인 Seq2Seq(또는 RNN Autoencoder)의 구조와 거의 유사합니다. 대신 encoder RNN이 아닌 encoder CNN을 사용해 이미지의 feature를 decoder의 입력으로 합니다. 또 다른 점은 둘 다 RNN 구조를 사용했을 때는 encoder의 hidden state를 decoder의 hidden state로 초기화 해주었는데 NIC에서는 encoder CNN의 출력을 decoder RNN의 첫 번째 입력으로 넣어줍니다.
---

# Model

- Machine translation 모델과 같이
- input **Image**가 주어졌을 때 정답 output  **Sequence**의 확률을 **maximizing**  
- correct transcription $S$, input image $I$, parameter $\theta$

$$ \theta^* = \arg\max_\theta \sum_{(I,S)} log p(S|I; \theta)$$
$$ logp(S|I) = \sum_{t=0}^{N} log p(S_t|I,S_0, ....,S_{t-1}) $$

---

<center>

<img src=https://user-images.githubusercontent.com/17582508/48947066-35f00500-ef73-11e8-94f3-1859e8274d67.png width=90% height="90%">

</center>

> 이 그림은 논문의 그림이고 더 좋은 구조 그림은 아래에 있습니다.
---
## Training
- Encoder로 Deep CNN 사용.
- Decoder로 LSTM 사용.
$$ x_{t-1} = CNN(I) $$
$$ x_t = W_eS_t, t \in \{0, ...., N-1\} $$
$$ p_{t+1} = LSTM(x_t), t \in \{0, ...., N-1\} $$

---

## Training Details
- Over fiiting을 막기 위한 techniques
	1. Pre-Train 된 Deep CNN 사용. (e.g. ImageNet)
	2. Pre-Train 된 Word embedding vetor 사용. (효과 적음)
	3. Dropout & Ensembling

> Pre-Train 된 word embedding vector를 사용하는 것은 효과가 적었기 때문에 실제 실험에서는 random initialize해서 사용했고 NIC 모델의 학습을 통해 embedding vector도 잘 학습됨을 아래 결과에서 보입니다.

---
# Model

<center>

<img src=https://user-images.githubusercontent.com/17582508/48948706-df85c500-ef78-11e8-8261-0c46d57dd3c5.png width=100% height="100%">

</center>

> 구현 입장에서 더 명확하다고 생각되는 구조 그림.

---

## Inference
- Sampling : 가장 확률이 높은 값(단어)을 고른다.
- BeamSearch : k 개의 후보(단어)를 뽑아서 다음 t+1 에서의 단어와의 조합의 확률을 보고 높은 값을 고른다.

> BeamSearch는 단순히 가장 높은 확률을 가지는 단어를 선택하는 것이 아닌 상위 k개를 뽑아 그 단어들로 다음 단어를 예측한 후 각 단어들의 확률을 더해 문장 단위로 높은 확률을 가지는 문장을 결과로 사용합니다.
---

# Experiments
## Evaluation Metrics
- Amazon Mechanical Turk experiment
- BLEU
- METHOR
- CIDER

> 각 지표는 wiki나 google을 통해 검색해보면 잘 나와 있습니다.

---

## Datasets
- 이미지마다 5개의 문장 (SBU 제외)
- Pascal VOC 2008은 test로만 사용 (실험에서는 학습은 MSCOCO로 함)
- Flickr는 사진과 사진에 대한 글을 올리는 사이트
- SBU는 Flicker에 올라온 사진과 글을 그대로 데이터로 사용

> SBU는 정확히 이미지를 설명하는 글이 아닌 사용자들이 올린 글이기 때문에 일종의 noise 역할을 하기를 기대함. (weakly labeled examples)

<center>

<img src=https://user-images.githubusercontent.com/17582508/48950978-52df0500-ef80-11e8-936e-5a891e3dbd01.png width=60% height="60%">

</center>

---

# Results
> 결과를 통해 세 가지 질문에 답하려 함.

- How dataset size affects generalization
- What kinds of transfer learning it would be able to achieve
- How it would deal with weakly labeled examples

---

## Generalization
- 좋은 데이터가 10만개 정도
- 데이터가 많아지면 더 좋은 결과가 나올 것이라 예상
- 데이터가 부족하기 때문에 generalization(overfitting 방지)을 하기 위해 노력함.

> 논문에서 overfitting 방지를 위해 위의 Training Details에서 설명한 3가지 방법을 실험했다고 설명합니다.

---

## Generation Results
- 여러 metric으로 평가해봄.
- 사람보다 점수가 높은 경우가 있지만 실제 결과는 그렇지 않음.
- metric에 대한 연구도 더 필요할 것으로 보임.

---

## Generation Results

<center>

<img src=https://user-images.githubusercontent.com/17582508/48952308-16fa6e80-ef85-11e8-90a5-68f456219ece.png width=70% height="60%">

</center>

---

## Transfer Learning
- 다른 dataset 간의 transfer가 가능한지 실험.
- Flickr30k -> Flickr8k (유사 데이터, 데이터 차이 4배)
	- BLEU 4 증가
- MSCOCO -> Flickr8k (다른 데이터, 데이터 차이 20배)
	- BLEU 10 감소, but 만든 문장은 괜찮음.
- MSCOCO -> SBU
	- BLEU 16 감소

---

## Generation Diversity Discussion
- generating model 모델이 새롭고 다양하고 높은 퀄리티의 문장을 만들어내는지 확인.

<center>

<img src=https://user-images.githubusercontent.com/17582508/48957613-f9390380-ef9c-11e8-9c60-25b44fe2cc27.png width=70% height="60%">

</center>

> 굵게 표시된 문장이 dataset에 없는 문장이다.

---

## Ranking Results
- 다른 연구에서 사용되는 지표인 Ranking Scores에서도 높은 점수를 받음.

<center>

<img src=https://user-images.githubusercontent.com/17582508/48958436-453a7700-efa2-11e8-92a1-ff96e9979c1a.png width=60% height="60%">

</center>

---

## Human Evaluation
- 사람이 직접 평가한 지표를 보여줌.
- BLEU Score는 human보다 높았는데, 여기서는 낮다.
- BLEU 지표가 완벽한 지표는 아님을 보여줌.

<center>

<img src=https://user-images.githubusercontent.com/17582508/48958773-b713c000-efa4-11e8-9831-7f9e81b237a6.png width=60%>

</center>

---

## Human Evaluation

<center>

<img src=https://user-images.githubusercontent.com/17582508/48958804-df9bba00-efa4-11e8-87f0-f6a8472e2921.png width=100%>

</center>

---

## Analysis of Embedding
- Word embedding vector도 유사한 단어들끼리 뭉쳐있도록 잘 학습됨을 확인.
<center>
  
<img src=https://user-images.githubusercontent.com/17582508/48958876-386b5280-efa5-11e8-9e8e-7c0bcf01da00.png width=60%>

</center>

## Reference
- https://arxiv.org/abs/1411.4555
- https://github.com/YBIGTA/DeepNLP-Study/wiki/Show-and-Tell:-A-Neural-Image-Caption-Generator-%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0
- https://github.com/yunjey/pytorch-tutorial/tree/master/tutorials/03-advanced/image_captioning
