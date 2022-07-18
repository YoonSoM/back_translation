![Alt text](https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/02_en_de.png)
기본적으로 기계 번역 모델은 Encoder-Decoder 구조를 이루며 Source Sentence가 Encoder에 입력되고, Target Sentence가 Decoder에 입력되며 훈련을 진행한다. 고로 두 문장이 한 쌍을 이루는 병렬 데이터가 불가피하다. 그런데 단일 데이터만으로 훈련을 하겠다니? 잘 상상이 가지 않는다. 저자들은 두 가지 방법을 제안했다.
# Dummy Source Sentence
첫 번째 방법은 Encoder의 입력으로 Dummy 값을 주는 것이다. 저자들은 null 토큰을 생성하여 Target Sentence의 단일 데이터와 한 쌍을 이루게끔 하였다. 그리고 Encoder의 모든 Parameter은 Freeze하여 Dummy 값에 대한 학습은 일절 이루어지지 않도록 하였다.
![Alt text](https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/03_flu.png)
이 경우 Decoder만 새로운 문장에 대해 추가 학습이 진행되는 것과 같은 효과이므로, 저자들이 말한 대로 유창한 번역을 만드는 데에 도움이 될 것 같은 느낌이 든다!
유의할 점은 단일 데이터가 병렬 데이터의 수를 넘어가게 되면 (비율이 1:1을 초과하면) Decoder가 Source Sentence로부터 추출한 정보를 잊어버리고, Target Sentence에 의존적인 양상을 보이게 된다고 한다. 이 문제점을 해결하고자 한 것이 두 번째 방법이고, 바로 Back Translation이다.
# Synthetic Source Sentence (Back Translation)
![Alt text](https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/04_synthetic.png)
아무런 정보도 담고 있지 않은 null 토큰을 입력으로 주느니, 완벽하지는 않더라도 Target Sentence를 보고 인공적인 Source Sentence를 만드는 방법론이다. 생성된 인공 데이터를 Synthetic Source Sentence라 칭한다. 그리고 인공 데이터를 생성하는 과정을 Back Translation이라 정의하며, 그 과정을 다음과 같이 표현하고 있다.
# Understanding Back-Translation at Scale
앞서 말했 듯이 Back Translation을 소개한 논문에는 다소 경험적으로 보이는 부분이 있었다. 즉, 분석이 부족했다. 이에 Back Translation을 자세히 분석하려는 시도가 있었고, 해당 논문은 2018년 EMLNP에서 소개되었다. 무려 Facebook과 Google의 합작품인 <Understanding Back-Translation at Scale>이다.
# Synthetic Data Generation Methods & Analysis
앞선 연구에서는 인공 데이터를 생성하는 데에 Greedy Decoding과 Beam Search를 사용했다. 저 두 방법은 모두 MAP(Maximum A-Posteriori)를 사용하며 가장 높은 확률을 갖는 데이터를 생성하게 된다. 저자들은 두 방법 외에 Non-MAP 방법을 비교해보고자 하였다. Non-MAP 방법으론 Random Sampling과 Top-10 Sampling, Noise를 추가한 Beam Search (이하 Noise Beam)이 있다.
![Alt text](https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/05_ex.png)
Beam Search: Source Sentence와 Target Sentence의 일부가 주어졌을 때, 다음 단어로 가장 높은 확률의 단어를 즉시 택하는 것을 Greedy Decoding이라 한다. Beam Search는 Greedy Decoding의 발전된 형태로, 두 번째, 세 번째(범위는 Beam Size에 의해 결정)로 높은 확률의 단어도 택해 문장을 ‘만들어 본다’. 그리고 최종적으로 만들어진 문장들의 확률의 총합을 구하여 가장 높은 값을 갖는 문장을 생성한다.
* * *
Random Sampling: 모델의 확률 분포에 따라서 랜덤하게 단어를 선택한다. 예를 들어 모델의 확률 분포가 “I”의 다음 단어로 “am”을 90%, “will”을 10%로 가진다면 10문장을 생성했을 때 “I am” 은 9문장, “I will”은 1문장이 나오는 식이다.
* * *
Top-10 Sampling: Random Sampling과 유사하지만, 각 단어들을 확률 내림차순으로 정렬했을 때 Top-10을 제외하곤 생성에 포함하지 않는다. 임의성이 Random Sampling보다 덜하므로 보다 문맥에 맞는 문장이 생성된다. 굳이 Top-10인 이유는 5, 20, 50으로 실험을 해봐도 비슷한 결과가 나오기에 그렇다.
* * *
Beam + Noise: Beam Search로 생성된 결과에 Noise를 추가하였다. 각 단어를 10%의 확률로 지우고, 10%의 확률로 filler token 인 BLANK로 치환하였다.
![Alt text](https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/06_graph_1.png)

