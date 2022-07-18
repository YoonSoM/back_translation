# Back-Translation
Back-Translation은 에딘버러 대학의 리코 센리치 교수가 제안한 방법으로, 센리치 교수님은 BPE를 통한 subword segmentation을 제안[5]한 분으로도 유명합니다. Parallel corpus의 부족으로 인해 겪는 가장 기본적인 문제중에 하나는, 디코더인 타깃 언어의 언어모델(Language Model, LM)의 성능 저하를 생각해볼 수 있습니다. 즉, 다량의 monlingual corpus를 수집하여 풍부한 표현을 학습할 수 있는 언어모델에 비해, parallel corpus만을 활용한 경우에는 훨씬 빈약한 표현만을 배울 수 밖에 없습니다. 따라서, 소스 언어 문장으로부터 타깃 언어 문장으로 가는 translation model(TM)의 성능 자체도 문제가 될테지만, 번역에 필요한 정보를 바탕으로 완성된 문장을 만들어내는 능력도 부족할 것 입니다.


이때, TM의 성능 저하는 parallel corpus의 부족과 직접적으로 연관이 있지만, LM의 성능 저하는 monolingual corpus를 통해 개선을 꾀해볼 수 있을 것 같습니다. 하지만 예전 Statistical Machine Translation (SMT)의 경우에는 보통 TM과 LM이 명시적으로 따로 존재하였기 때문에 monolingual corpus를 통한 LM의 성능 개선을 쉽게 시도할 수 있었지만, NMT에선 end-to-end 모델로 이루어져 있으므로 LM이 명시적으로 분리되어 있지 않아 어려움이 있습니다. BT는 이러한 상황에서 디코더의 언어모델의 성능을 올리기 위한 (+ 추가적으로 TM의 성능 개선도 약간 기대할 수 있는) 방법을 제안합니다.


보통 번역기를 개발할 경우, 한 쌍의 번역 모델이 자연스럽게 나오게 됩니다. 왜냐하면 우리는 parallel corpus를 통해 번역기를 개발하므로, 두 방향의 번역기를 학습할 수 있기 때문입니다. 이때 Back-Translation이라는 이름에서 볼 수 있듯이, BT는 반대쪽 모델을 타깃 모델을 개선하는데 활용합니다.

예를 들어 아래와 같이 parallel corpus [Math Processing Error] 와 monolingual corpus [Math Processing Error] 을 수집한 상황을 생각해볼 수 있습니다.


그럼 자연스럽게 우리는 일단은 [Math Processing Error] 를 활용하여 두 개의 모델을 얻을 수 있습니다.


이때, 우리는 [Math Processing Error] 를 통해서 [Math Processing Error] 데이터셋에 대한 추론 결과를 얻어, pseudo(or synthetic) corpus를 만들 수 있습니다. 즉, 반대쪽 모델에 monolingual corpus를 집어넣어 [Math Processing Error] 을 구할 수 있습니다.


이제 그럼 우리는 새롭게 얻은 [Math Processing Error] 을 포함하여 [Math Processing Error] 와 함께 다시 [Math Processing Error] 를 학습하면 더 나은 성능의 파라미터를 얻을 수 있다는 것이 BT입니다. 당연히 이것은 반대쪽 모델에도 똑같이 적용 가능할 것 입니다.


이 방법의 핵심은 pseudo sentence가 인코더에 들어가고, 실제 문장이 디코더에 들어가는 것입니다. 이에따라 인코더는 비록 큰 도움을 못받더라도, 디코더는 어쨌든 주어진 인코더의 결과값에 대해서 풍부한 언어모델 디코딩 능력을 학습할 것으로 예상할 수 있습니다.
# 실험 결과 및 한계
아래와 같이 BT는 굉장히 간단한 구현 방법에 비해 준수한 성능 개선 효과를 보여줍니다.

![Alt text](https://kh-kim.github.io/assets/images/20200930/1.png)

그런데 중요한 점은 pseudo corpus의 양이 너무 많아서는 안된다는 것입니다. 비록 우리는 무한대에 가까운 monolingual corpus를 얻어 pseudo corpus를 만들어낼 수 있겠지만, 만약 그럴경우 pseudo corpus가 기존 parallel corpus를 압도해버릴 수 있습니다. Pseudo corpus의 경우에는 인코더에 들어갈 [Math Processing Error] 이 실제 정답과는 일부 다를 수 있기 때문이고, 더욱이 [Math Processing Error] 에 의해 bias가 생겨있는 상태일 것이므로, 너무 많은 양의 pseudo corpus를 활용할 경우 [Math Processing Error] 가 잘못된 bias를 학습할 경우도 생각해볼 수 있습니다. [6] 따라서 우리는 제한된 양의 [Math Processing Error] 만 활용할 수 있으며, 이는 또 하나의 하이퍼파라미터를 추가시킵니다. 그리고 이 하이퍼파라미터는 보통 기존 parallel corpus의 2~3배 정도가 적당하다고 알려져 있습니다.
# Noise 추가를 통한 Back-Translation 개선
실제 모든 [Math Processing Error] 을 활용하지 못하고 제한된 양을 활용할 수 밖에 없기 때문에, 이 제한된 양을 좀 더 늘릴수 없을지 또 다른 연구들이 이어졌습니다. [7]에서는 pseudo corpus를 생성할 때, noise를 섞으면 BT의 성능이 더 향상되는 것을 확인하였습니다. 예를 들어 generation을 하는 과정에서 argmax(or greedy)를 통해 번역 문장을 생성하는 것보다, random sampling을 통해 random noise를 섞어주거나 beam seach 과정에서 약간의 noise를 섞어주는 것이 기존 BT보다 더 나은 성능을 제공한다는 것입니다.
![Alt text](https://kh-kim.github.io/assets/images/20200930/2.png)
이는 인코더에서 [Math Processing Error] 를 학습할 때, 기존 [Math Processing Error] 의 bias를 학습하는 것을 방해하는 일종의 regularization 역할로도 생각해볼 수 있습니다.

# Tagged Back-Translation
여기서 한 발 더 나아가 더 쉬운 방법을 통해 더 높은 성능을 제공하는 방법도 제안되었습니다. [6]에서는 인코더에서의 잘못된 bias 학습으로 인해 번역기 전체 성능이 하락되는 것을 막기 위해, pseudo corpus에 tag를 붙인 상태로 학습하는 것을 제안하였습니다. 좀 더 정확히 말하면 인코더에 입력으로 들어가는 소스 언어의 pseudo sentence의 맨 앞에 pseudo corpus라는 tag를 넣어주어, 네트워크가 pseudo corpus에 대해서는 다르게 행동하여, 실제 테스트 환경에서는 잘못 학습된 bias로 인해 번역 성능이 낮아지는 것을 막고자 하였습니다.
![Alt text](https://kh-kim.github.io/assets/images/20200930/3.png)
이 결과 기존 BT 뿐만 아니라, Noise added BT에 비해서도 더 높은 성능 향상을 이끌어냈으며, 심지어 기존 방법에 비해 더 많은 monolingual corpus를 활용 하였을 때도 성능의 저하가 이루어지지 않는 것을 확인하였습니다. 이것은 전체 monolingual corpus를 활용할 수 없어 아쉬움이 남던 BT의 단점을 획기적으로 개선한 것이라고 볼 수 있을 것입니다.

위에서 언급하였듯이, 우리는 tag를 pseudo sentence의 맨 앞에 달아 인코더에 넣어줌으로써, 아마도 인코더는 pseudo corpus를 처리하기 위한 별도의 mode에 들어갈 것이고, 이는 잘못된 bias를 학습하는 것을 방지하여 기존의 parallel corpus 학습에 지장을 주지 않도록 하지 않을까 예상해볼 수 있습니다.
# 실제 Back-Translation은 효과가 있을까?
이처럼 BT를 활용한 방법들은 간단하면서도 높은 성능을 제공하는 효율성으로 널리 사랑받고 있습니다. 이때, [8]에서는 실제로 BT가 겉으로 보이는 성능 만큼이나 실제로도 번역기의 성능을 개선하는데 도움이 되는지 분석해 보았습니다. 이를 위해 이 논문에서는 아래의 3가지 질문에 대해서 BT에 실제로 어떻게 동작하는지 좀 더 검증해보고자 하였습니다. 

저자는 기존의 논문들에서 사용한 테스트셋(e.g. WMT)의 입력들이 자연스러운 원문장이라기보단 상대적으로 어색한 원문장에 대한 번역문(translationese)들을 포함하고 있으며, BT 방법들이 이러한 테스트셋에서 좋은 성능을 거둘 수 있었던 것은 실제 번역성능이 올랐다기보단 translationese들을 잘 번역했기 때문이라고 주장했습니다. Pseudo sentence [Math Processing Error] 를 인코더에 넣어 학습시키는 것은 bias가 포함되어 있기 때문에, translationese를 잘 번역하도록 할 뿐 실제 원문(본문에서는 original text라고 표현)에 대한 번역 성능은 검증이 필요하다는 것입니다. 이를 위해서 저자는 아래와 같이 기존 각 연도별 WMT 테스트셋에서 original text와 translationese를 구분해서 BT와 Tagged BT의 성능을 각각 검증해보았습니다.
![Alt text](https://kh-kim.github.io/assets/images/20200930/4.png)
그 결과 재미있게도 vanilla BT의 경우에는 translationese가 입력으로 주어졌을 때의 성능 향상만 있었을 뿐, original text에 대해서는 오히려 성능이 하락하는 것을 알 수 있습니다. 특히 translationese에 대한 성능 개선이 두드러지는 바람에, 둘이 섞인 전체 테스트셋에서는 오히려 Tagged BT에 비해 성능 향상이 더 큰 것처럼 착시 현상을 일으키기까지 합니다. 이에 반해 Tagged BT의 경우에는 translationese 뿐만 아니라, original text에서도 미미하지만 성능 개선이 있었음을 확인할 수 있습니다.

결국 BT로 인한 성능 향상의 대부분은 번역문과 같은 translationese를 입력으로 받았을 때 일어난 것으로 해석할 수 있습니다. 이것은 물론 실망스러운 결과일 수 있지만, 그렇다고 해서 실제 deploy환경에서 translationese와 같은 입력들이 전혀 없을 것은 아니기 때문에, 전혀 쓸모없는 성능 개선이라고는 볼 수 없을 것입니다.

물론 아래와 같이 low-resource 환경에서의 번역일 때는 BT와 Tagged BT 모두 번역 성능 개선에 매우 큰 도움을 주는 것을 확인할 수 있습니다.
![Alt text](https://kh-kim.github.io/assets/images/20200930/5.png)
여기에서도 Tagged BT가 기존 BT보다 original text에서 더 나은 성능 개선 폭을 보이는 것을 확인할 수 있습니다.
# Back-Translation 수식으로 풀어보기
사실 BT는 부족한 코퍼스로 인한 디코더 언어모델의 성능개선이라는 미명아래, 직관적인 설명에 의존해서 제안되었습니다. [9]에서는 여기에 수식으로 BT를 설명하기도 하였습니다.

아까와 같이 parallel corpus와 monolingual corpus가 수집되었다고 할 때, importance sampling을 통해 아래와 같이 수식을 전개할 수 있을 것입니다.

그리고 Jensen’s Inequality를 활용하여 부등식을 완성할 수 있습니다. – VAE[10]의 전개와 매우 비슷합니다.

여기서 우리가 구하고자 하는 파라미터 [Math Processing Error] 는 아래의 수식을 최대화 한다고 할 때, 위의 수식을 넣어 볼 수 있을 것입니다
